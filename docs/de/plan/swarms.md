---
x-i18n:
    generated_at: "2026-07-26T17:52:38Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 90c6c85a837448f4e5ceccdccf73489db801ad502cbbb2f3eb04d6aff7e902f0
    source_path: plan/swarms.md
    workflow: 16
---

# Swarms — Agent-Fan-out und Orchestrierung im Code-Modus

Status: Ausgeliefert — ersetzt durch `docs/tools/swarm.md`. Dieses Dokument bleibt als
Entwurfsdokument der Implementierung erhalten.

## 1. Was und warum

Ein **Swarm** besteht aus vielen Subagents, die deterministisch von einem Skript
im Code-Modus orchestriert werden: N Leser auffächern, Ergebnisse adversariell
überprüfen, durch einen zustandsbehafteten Priorisierer synthetisieren und
Entscheidungsschranken wiederholt durchlaufen. Der Kontrollfluss
(`Promise.all`, `while`, `if`) _ist_ die Orchestrierung — es gibt bewusst **keine Graph-DSL,
keinen neuen Modus und keine neue Top-Level-Tool-Oberfläche**.

Der OpenClaw-Code-Modus (QuickJS-WASI, Snapshot/Fortsetzung, Bridge-Anfragen)
bildet die Grundlage. Ein geparkter Bridge-Aufruf übersteht VM-Snapshots und
Gateway-Neustarts und wird exakt an der Stelle fortgesetzt, an der er angehalten
wurde — stärker als Journal-Replay-Entwürfe und ohne Determinismusanforderungen
an Skripte.

Benennung: Der Name in Produkt und Dokumentation lautet **Swarm**. Code-Bezeichner bleiben unverändert:
`agents.*`-Gast-API, `tools.swarm`-Konfiguration, `swarm`-Gruppenspalten.

## 2. Entscheidungen (Maintainer, 2026-07-17)

- Kosten: erzwungene Konfigurationsobergrenzen; Token-Budget pro Swarm optional. Kein verpflichtendes Budget.
- Genehmigungen: untergeordnete Agents werden **fehlersicher geschlossen / nicht interaktiv** ausgeführt. Aktionen,
  die eine Genehmigung erfordern, werden abgelehnt; die Ablehnung wird im Ergebnis
  des untergeordneten Agents gemeldet; das Skript entscheidet. Keine Flut von Bedienerabfragen durch Fan-out.
- v1 umfasst nur ad hoc vom Modell geschriebene Skripte. Gespeicherte/benannte Workflows, CLI-/Cron-
  Einstieg: später (ein Headless-Code-Modus ist für Cron bereits vorhanden).
- Identität des untergeordneten Agents: standardmäßig ein dedizierter Worker-Agent über die
  `tools.swarm.defaultAgentId`-Konfiguration (gegen die bestehende Zulassungsliste für Subagent-Ziele validiert);
  `agentId`-Überschreibung pro Spawn. Core liefert keine gebündelte Agent-ID aus; die Dokumentation empfiehlt
  eine schlanke `worker`-Agent-Konfiguration.
- Keine Änderungen am Codex-Quellcode. Das Codex-Harness verwendet das Spawn/Wait-Idiom (§8).

## 3. Architekturübersicht

```
Code-Modus-Skript (QuickJS-VM, Gateway)          Codex-V8-Skript (Codex-Prozess)
  agents.run(...) ── geparkter Bridge-Aufruf       tools.sessions_spawn / tools.agents_wait
        │                                                │ Element-/Tool-/Aufruf-RPC (jeweils ≤600s)
        ▼                                                ▼
             CORE (Harness-unabhängig, dieses Repository)
  sessions_spawn {collect:true, outputSchema, fastMode, groupId}
  agents_wait {ids, timeoutSeconds}
        │
  Subagent-Registry (SQLite): Collector-Abschlussdatensätze, Swarm-Gruppen-ID
        │
  untergeordnete Agents = gewöhnliche Subagent-Sitzungen (Lane-begrenzt, fehlersicher geschlossene Genehmigungen)
        │
  sessions.changed SSE ──► Punkte der Control UI / Seitenleiste / Kanalstatusmeldung
```

Ein kanonischer Verantwortlicher für Spawn-/Abschluss-/Erledigungssemantik
(Core-Tools + Registry). Zwei Await-Transporte: QuickJS parkt einen Bridge-Aufruf
unbegrenzt (Snapshot); Codex fragt `agents_wait` in begrenzten RPCs ab.

## 4. Konfigurationsschranke (v1)

Neue `tools.swarm` (global + Überschreibung pro Agent, dasselbe Zusammenführungsmuster wie
`tools.codeMode`):

```jsonc
"tools": {
  "swarm": {
    "enabled": false,            // Hauptschranke, standardmäßig AUS
    "maxConcurrent": 8,          // gleichzeitig ausgeführte untergeordnete Agents (Swarm-Lane-Obergrenze)
    "maxChildrenPerGroup": 50,   // aktive untergeordnete Agents pro Swarm-Gruppe
    "maxTotalPerGroup": 200,     // lebenslange Spawn-Anzahl pro Gruppe (Rückfallsicherung gegen außer Kontrolle geratene Ausführung)
    "waitTimeoutSecondsMax": 600,
    "defaultAgentId": ""         // optional; ID des untergeordneten Agents, wenn beim Spawn agentId fehlt
  }
}
```

- Zod: Union `boolean | strict object` wie `CodeModeSchema`
  (`src/config/zod-schema.agent-runtime.ts`); `swarm: true` → `{enabled: true}`.
- Typen in `src/config/types.tools.ts` (sowohl pro Agent als auch Top-Level-`tools`),
  Bezeichnungen in `schema.labels.ts`, Hilfe in `schema.help.runtime.ts`.
- Auflösungshelfer `resolveSwarmConfig(cfg, agentId)` analog zu
  `resolveCodeModeConfig` (`src/agents/code-mode.ts:215`), der alle Zahlen begrenzt.
- Auswirkungen der Schranke bei Deaktivierung: Das `agents_wait`-Tool fehlt in Katalogen;
  die Parameter `collect`/`outputSchema`/`fastMode`/`groupId` für `sessions_spawn`
  werden mit einer eindeutigen Fehlermeldung abgelehnt, die den Konfigurationsschlüssel nennt. Keine weitere Verhaltensänderung.
- `defaultAgentId` wird über `resolveSubagentAllowedTargetIds`
  (`src/agents/subagent-target-policy.ts`) validiert; unbekannte ID → Spawn-Fehler, kein Fallback.

## 5. Core: Spawn im Collector-Modus + `agents_wait` (v1)

### 5.1 Ergänzungen für `sessions_spawn` (alle von aktiviertem Swarm abhängig)

- `collect: boolean` — wenn wahr, wird der Lauf des untergeordneten Agents mit
  `expectsCompletionMessage: false` und einem **Collector-Abschlussdatensatz**
  anstelle einer Ankündigungs-/Steuerungszustellung registriert. Das Tool gibt sofort
  `{ runId, sessionKey }` zurück. Keine Kanal-/Thread-Bindung.
- `outputSchema: object` — JSON Schema. Der untergeordnete Agent erhält ein synthetisches
  `structured_output`-Tool, das seiner Tool-Oberfläche hinzugefügt wird; ein System-Prompt-Zusatz
  weist ihn an, es genau einmal mit seinem Endergebnis aufzurufen. Bei einem
  Validierungsfehler erhält der untergeordnete Agent eine einmalige Aufforderung zur Wiederholung; danach enthält
  der Abschlussdatensatz `structured: undefined` sowie den Rohtext und einen `schemaError`.
- `fastMode: true | "auto" | false` — wird zusammen mit Modell/Denken über
  `resolveSubagentModelAndThinkingPlan` (`src/agents/subagent-spawn-plan.ts`) in den Sitzungs-Patch des untergeordneten Agents
  eingefügt und verwendet die bestehende `FastMode`-Achse
  (`src/shared/fast-mode.ts`). Weggelassen = erben.
- `groupId: string` — Stempel der Swarm-Gruppe. Standardmäßig
  `swarm:<requesterSessionKey>:<runId-of-requesting-run>`. Wird im Registry-Datensatz und in der
  Sitzungszeile des untergeordneten Agents persistiert. Wird für Obergrenzen, Auflistung, Batch-
  Archivierung und die Punkte verwendet.
- `label: string` ist bereits vorhanden — wird in den Punkten und in `subagents list` angezeigt.
- ID des untergeordneten Agents: `params.agentId` → andernfalls `tools.swarm.defaultAgentId` → andernfalls
  anfragender Agent (bestehendes Verhalten).

### 5.2 Genehmigungen fehlersicher geschlossen

Untergeordnete Collector-Agents werden mit einem nicht interaktiven Genehmigungskontext ausgeführt:
Jeder Tool-Aufruf, der eine Bedienergenehmigung erfordern würde, wird als strukturierte
Ablehnung (`approval_required`) aufgelöst, die für den untergeordneten Agent sichtbar ist; von ihm wird
erwartet, dass er die Blockierung in seinem Ergebnis meldet. Implementierung: Die bestehende
Weiterleitung der Ausführungs-/Tool-Genehmigungsrichtlinie wird mit einem erzwungenen
`deny`-Resolver für Läufe untergeordneter Agents im Collector-Modus wiederverwendet.
Von untergeordneten Collector-Agents werden keine Genehmigungsereignisse an Bedieneroberflächen ausgegeben.

### 5.3 `agents_wait`-Tool (neu, durch Schranke geschützt)

```
agents_wait({ ids: string[], timeoutSeconds?: number })
→ {
    completed: [{ runId, status: "done"|"failed"|"killed"|"timeout",
                  result: string, structured?: unknown, schemaError?: string,
                  sessionKey, label?, usage?: {inputTokens, outputTokens} }],
    pending: string[]
  }
```

- Gibt zurück, sobald **mindestens eine** ID abgeschlossen ist (Semantik des ersten Abschlusses /
  Race-Semantik, ermöglicht Pipelines), oder bei Zeitüberschreitung mit `completed: []`.
- `timeoutSeconds` standardmäßig 30, begrenzt auf `waitTimeoutSecondsMax`.
- Idempotent: Bereits abgeschlossene IDs geben ihre Datensätze erneut zurück (Datensätze werden
  bis zur Gruppenarchivierung aufbewahrt). Unbekannte ID → Fehlereintrag pro ID, kein Throw.
- Eigentümerschaft: Nur die Sitzung, die einen Lauf erzeugt hat (oder deren Elternkette), darf
  auf ihn warten — dieselbe Eigentümerschaftsregel wie für `wait` im Code-Modus (`code-mode.ts:1684`).
- Registry: Abschlussdatensätze befinden sich im bestehenden SQLite-Speicher
  der Subagent-Registry (`subagent-registry.store.sqlite.ts`) — neue Felder, kein neuer Speicher,
  keine Erhöhung der Schemaversion (nur additive Spalten; siehe Einschränkung in §9).

### 5.4 Durchsetzung der Obergrenzen

- `maxConcurrent`: Untergeordnete Collector-Agents laufen auf der bestehenden Subagent-Lane, werden aber
  pro Swarm-Gruppe gezählt; Spawns oberhalb der Obergrenze werden FIFO eingereiht (hostseitig im
  Spawn-Pfad — runId wird sofort zurückgegeben, der Lauf startet, sobald ein Platz frei wird).
- `maxChildrenPerGroup` / `maxTotalPerGroup`: Nach Überschreitung wird der Spawn mit einem typisierten Fehler
  abgelehnt; der Fehlertext nennt den Konfigurationsschlüssel.
- Tiefe: Untergeordnete Collector-Agents behalten die `DEFAULT_SUBAGENT_MAX_SPAWN_DEPTH`-Semantik bei
  (untergeordnete Agents sind Blätter, sofern Verschachtelung nicht ausdrücklich konfiguriert ist).

## 6. Testvertrag (v1, Lane A)

- Unit: Auflösung/Begrenzung der Konfiguration; Ablehnungen durch die Schranke bei Deaktivierung;
  Standardwert für groupId; Durchsetzung der Obergrenzen (Warteschlange + Ablehnung);
  Race-Semantik beim Warten; Idempotenz beim Warten; Verweigerung aufgrund der Eigentümerschaft;
  Validierung strukturierter Ausgabe + Aufforderung zur Wiederholung + schemaError-Pfad;
  Weiterleitung von fastMode in den Sitzungs-Patch; Validierung von defaultAgentId.
- Integration (vitest, simulierte Modell-Runtime): 3 untergeordnete Collector-Agents erzeugen, in
  einer Schleife warten, Reihenfolge des ersten Abschlusses und endgültige Leerung bestätigen;
  Simulation eines Gateway-Neustarts: Registry neu laden → Warten wird aus persistiertem Abschluss aufgelöst.
- Alle Tests gemeinsam unter `*.test.ts`; keine Live-Modellaufrufe.

## 7. QuickJS-Gastoberfläche (Lane B, nach Core)

- Gast-Globals werden in `CONTROLLER_SOURCE`
  (`src/agents/code-mode.worker.ts:190-374`) installiert, reservierte Namen werden in
  `code-mode-namespaces.ts` hinzugefügt:
  - `agents.run(prompt, opts) → Promise<result|structured>` — Komfortfunktion:
    Collector-Spawn + geparktes Warten über eine dedizierte Bridge-Methode (`agentWait`),
    die der Host bei Abschluss erledigt (kein Polling; Snapshot-sicher).
  - `agents.session(system, opts) → Promise<handle>`;
    `handle.send(input, opts) → Promise<...>`; `handle.close()`. (v1.1 —
    wird nach run() ausgeliefert; verwendet `mode:"session"` + Collector-Datensätze pro Turn.)
  - `phase(title)`, `log(message)` — Fire-and-forget-Bridge-Benachrichtigungen →
    Swarm-Fortschrittsereignisse.
- Bridge-Methoden werden zu `CodeModeBridgeMethod` (`code-mode.ts:91`) hinzugefügt:
  `agentSpawn`, `agentWait`, `swarmNote`. `agentSpawn`/`agentWait` sind
  **konstruktionsbedingt** Replay-sicher: Der Idempotenzschlüssel `(codeModeRunId, bridgeId)`
  wird im Registry-Datensatz gespeichert; ein Neustart erledigt aus persistierten Abschlüssen erneut
  und erzeugt niemals doppelte Spawns.
- Ausstehende `agentWait`-Bridge-Aufrufe verlängern die Snapshot-TTL des Laufs (die Menge
  ausstehender Agents ist das Signal; kein Flag).
- Die virtuelle `API.read("agents.d.ts")`-Datei dokumentiert die typisierte Oberfläche und die
  Fan-out-/Schranken-/Zyklus-Idiome (`createCodeModeApiVirtualFiles`,
  `code-mode-namespaces.ts:876`).

## 8. Codex-Harness-Projektion (spätere Lane)

- `sessions_spawn` (mit neuen Parametern) und `agents_wait` durchlaufen die
  bestehende dynamische Tool-Bridge; innerhalb von Codex-Code-Modus-Skripten erscheinen sie automatisch
  als `tools.*` (verifiziert: `codex-rs/code-mode/src/runtime/globals.rs:14-65`,
  `codex-rs/core/src/tools/spec_plan.rs:448-507`).
- `agents_wait` erhält die lange Zeitüberschreitungsklasse für dynamische Tools
  (Obergrenze 600s; `extensions/codex/src/app-server/dynamic-tool-execution.ts:37-39`) und wird
  als Zeitüberschreitungs-/Replay-sicher markiert.
- Gruppenschlüssel für Codex-Eltern: `swarm:<parentSessionKey>:<turnId>`.
- Codex-native `spawn_agent`-Subagents bestehen parallel; ihre Task-Spiegelzeilen speisen
  dieselbe Fortschrittsoberfläche.

## 9. Persistenz und Aufbewahrung

- Keine neuen Speicher. Registry-Datensätze erweitern die bestehenden SQLite-Tabellen
  der Subagent-Registry; untergeordnete Agents sind gewöhnliche `sessions`-Zeilen. Nur additive Spalten
  — **jede Änderung, die eine Erhöhung der SQLite-Schemaversion erfordert,
  benötigt zuerst die ausdrückliche Zustimmung eines Maintainers** (Repository-Richtlinie).
- Swarm-Gruppen-ID im Registry-Datensatz + Sitzungsmetadaten des untergeordneten Agents.
- Aufbewahrung: Abgeschlossene Collector-Datensätze bleiben bis zur **Gruppenarchivierung** erhalten:
  Wenn der Elternlauf endet (oder die TTL abläuft), werden die untergeordneten Agents der Gruppe
  als Batch archiviert (der bestehende `DEFAULT_SUBAGENT_ARCHIVE_AFTER_MINUTES`-Sweep wird erweitert,
  sodass er pro Gruppe arbeitet).

## 10. Fortschrittsoberfläche („die Punkte“) — spätere Lane

- Implizit, vom Harness gesteuert. Abgeleitet aus bestehendem `sessions.changed`-SSE +
  Registry; `phase`-/`log`-Hinweise ergänzen die Semantik. Kein vom Agent gesteuertes Rendering.
- Control UI: `swarm`-Renderer in der Workspace-Widget-Familie
  (`ui/src/lib/workspace/widgets/`) — nach Phase gruppiertes Punkteraster, Erzählerzeile,
  Status/Bezeichnung/Modell pro Punkt; untergeordneter Baum in der Seitenleiste unverändert.
- Kanäle: eine gedrosselte, bearbeitete Statusmeldung pro Gruppe (gemäß
  `docs/concepts/streaming.md`; niemals Meldungen pro untergeordnetem Agent).

## 11. Labs-Seite (Control UI, unabhängiger Entwicklungsstrang)

Settings → **Labs**: Schalter für experimentelle Funktionen, als erste Einträge **Code Mode**
und **Swarm**. Jede Zeile: Name, einzeilige Beschreibung, Link zur Dokumentation, über
den vorhandenen `config.patch`-RPC angebundener Schalter (RFC-7396-Merge-Patch —
`tools.codeMode.enabled` / `tools.swarm.enabled` setzen) sowie gegebenenfalls ein Hinweis
„Neustart erforderlich“. Auffindbar, wobei der Text den experimentellen Status
deutlich macht. i18n: alle Zeichenfolgen über die normale `en.ts`- und Synchronisierungs-Pipeline.

## 12. Platzierung (später)

- `placement`-Option beim Starten: `"local"` (Standard) | `"cloud:<profile>"` über
  die vorhandene Weiterleitung an die Worker-Umgebung (`sessions.dispatch`); gepoolte Platzierung
  später, falls SSH-Sandbox-Kindprozesse auf gemeinsam genutzten Systemen sich als unzureichend erweisen.
- Die Orchestrator-VM verbleibt immer auf dem Gateway; Settle/Dots/Budget sind
  unabhängig von der Platzierung.

## 13. Nichtziele

- Keine Graph-DSL — der Kontrollfluss ist der Graph (bewusst so gewählt und dokumentiert).
- Keine Änderungen am Codex-Quellcode; keine Wiederverwendung der Interna des Codex Code Mode.
- Keine gespeicherten/benannten Workflows in v1; kein CLI-Einstiegspunkt.
- Keine Weiterleitung von Bedienerfreigaben einzelner Kindprozesse nach oben.
- Keine 1:1-Cloud-Bereitstellung im Umfang einer Auffächerung.
- Keine Kompatibilitäts-Shims im regulären Runtime-Betrieb; Swarm ist eine neue, abgesicherte Oberfläche.

## 14. Build-Phasen / PR-Aufteilung

1. **Entwicklungsstrang A (Kern)**: §4 Konfiguration + §5 Starten/Warten/Obergrenzen/Freigaben + §6 Tests.
2. **Entwicklungsstrang C (Labs-Seite)**: §11 — unabhängig, kann zuerst integriert werden.
3. **Entwicklungsstrang B (QuickJS-Oberfläche)**: §7 — nachdem die Verträge aus A integriert wurden.
4. Dots-Renderer (§10), Codex-Projektion (§8), `agents.session` (§7 v1.1),
   Platzierung (§12), Überarbeitung der Benutzerdokumentation — Folge-PRs in dieser Reihenfolge.

Jeder PR: grüne CI, `$autoreview` sauber, standardmäßig deaktiviert, main auslieferbar.
