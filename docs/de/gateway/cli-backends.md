---
read_when:
    - Sie möchten einen zuverlässigen Fallback, wenn API-Provider ausfallen
    - Sie führen lokale KI-CLIs aus und möchten sie wiederverwenden
    - Sie möchten die MCP-Loopback-Bridge für den Tool-Zugriff auf das CLI-Backend verstehen
summary: 'CLI-Backends: lokaler KI-CLI-Fallback mit optionaler MCP-Tool-Bridge'
title: CLI-Backends
x-i18n:
    generated_at: "2026-07-24T22:13:33Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 0ce0427c587bf2a1e0a2ff24b5e76952eecae059e6f900af777b897b2d8d4210
    source_path: gateway/cli-backends.md
    workflow: 16
---

OpenClaw kann eine lokale KI-CLI als reine Textausweichlösung ausführen, wenn API-Provider ausgefallen, ratenbegrenzt oder fehlerhaft sind. Diese Funktion ist bewusst konservativ ausgelegt:

- OpenClaw-Tools werden nicht direkt eingebunden, aber ein Backend mit `bundleMcp: true` kann Gateway-Tools über eine Loopback-MCP-Bridge empfangen.
- JSONL-Streaming für CLIs, die es unterstützen.
- Sitzungen werden unterstützt, sodass Folgeinteraktionen kohärent bleiben.
- Bilder werden weitergereicht, wenn die CLI Bildpfade akzeptiert.

Verwenden Sie dies als Sicherheitsnetz für Textantworten, die „immer funktionieren“ sollen, nicht als primären Pfad. Verwenden Sie für eine vollständige Harness-Laufzeit mit ACP-Sitzungssteuerung, Hintergrundaufgaben, Thread-/Konversationsbindung und persistenten externen Coding-Sitzungen stattdessen [ACP-Agenten](/de/tools/acp-agents); CLI-Backends sind kein ACP.

<Tip>
  Sie erstellen ein neues Backend-Plugin? Siehe [CLI-Backend-Plugins](/de/plugins/cli-backend-plugins). Diese Seite behandelt die Konfiguration und den Betrieb eines bereits registrierten Backends.
</Tip>

## Schnellstart

Das gebündelte Anthropic-Plugin registriert ein standardmäßiges `claude-cli`-Backend, sodass es ohne weitere Konfiguration funktioniert, sofern Claude Code installiert und angemeldet ist:

```bash
openclaw agent --agent main --message "hi" --model claude-cli/claude-sonnet-4-6
```

`main` ist die Standard-Agenten-ID, wenn keine explizite Agentenliste konfiguriert ist; verwenden Sie andernfalls Ihre eigene Agenten-ID.

Der Gateway-Dienst muss die CLI in seinem `PATH` verfügbar haben. Wenn eine Bereitstellung einen
nicht standardmäßigen Pfad zur ausführbaren Datei oder nicht standardmäßige Argumente benötigt, registrieren Sie diesen Adapter stattdessen in einem
[CLI-Backend-Plugin](/de/plugins/cli-backend-plugins), anstatt Startmechanismen in
`openclaw.json` abzulegen.

OpenClaw lädt automatisch ein zuständiges gebündeltes Plugin, wenn die Modellauswahl oder ein
modellbezogenes `agentRuntime.id` auf dessen Backend verweist.

## Verwendung als Ausweichlösung

Fügen Sie das CLI-Backend Ihrer Ausweichliste hinzu, damit es nur ausgeführt wird, wenn primäre Modelle fehlschlagen:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["claude-cli/claude-sonnet-4-6"],
      },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "claude-cli/claude-sonnet-4-6": {},
      },
    },
  },
}
```

Konfigurierte Ausweichmodelle bleiben auswählbar, wenn der primäre Provider fehlschlägt (Authentifizierung, Ratenbegrenzungen, Zeitüberschreitungen), selbst wenn sie nicht in `agents.defaults.modelPolicy.allow` enthalten sind. Fügen Sie dieser Richtlinie ein CLI-Backend-Modell nur hinzu, wenn Benutzer es auch direkt über `/model`, eine Sitzungsüberschreibung oder `--model` auswählen können sollen. `agents.defaults.models` verwaltet nur modellspezifische Aliase, Parameter und Metadaten.

## Konfiguration

Benutzer wählen ein registriertes Backend über die Modell- und Laufzeitrichtlinie aus. Behalten Sie
die kanonische Modellreferenz bei und wählen Sie die CLI-Laufzeit pro Modell aus:

```json5
{
  agents: {
    defaults: {
      model: "anthropic/claude-opus-5",
      models: {
        "anthropic/claude-opus-5": {
          agentRuntime: { id: "claude-cli" },
        },
      },
    },
  },
}
```

Anmeldedaten verbleiben in OpenClaw-Authentifizierungsprofilen oder in der Konfiguration des zuständigen Plugins.
Mechanismen für Befehl, argv, Umgebung, Parsing, Sitzung, Bilder und Watchdog sind
Plugin-Code, der mit `api.registerCliBackend(...)` registriert wird.

## Funktionsweise

1. Wählt ein Backend anhand des Provider-Präfixes (`claude-cli/...`) aus.
2. Erstellt einen System-Prompt mit demselben OpenClaw-Prompt und Workspace-Kontext.
3. Führt die CLI mit einer Sitzungs-ID aus (sofern unterstützt), damit der Verlauf konsistent bleibt. Das gebündelte `claude-cli`-Backend hält pro OpenClaw-Sitzung einen Claude-stdio-Prozess aktiv und sendet Folgeinteraktionen über stream-json-stdin.
4. Parst die Ausgabe (JSON oder Klartext) und gibt den endgültigen Text zurück.
5. Speichert Sitzungs-IDs pro Backend dauerhaft, damit Folgeinteraktionen dieselbe CLI-Sitzung wiederverwenden.

## Zeitüberschreitungen und lang laufende Aufgaben

CLI-Backends haben zwei unabhängige Begrenzungen:

- `agents.defaults.timeoutSeconds` begrenzt die gesamte Agenteninteraktion. Normale Gateway-Interaktionen übernehmen den Standardwert von 48 Stunden; `0` hebt die Begrenzung des Interaktionsbudgets auf. Eine gespeicherte Überschreibung wie `600` ersetzt diesen Standardwert.
- Der Watchdog für ausbleibende CLI-Ausgaben beendet einen Unterprozess, der keine Ausgabe erzeugt. Jedes Backend-Plugin verwaltet separate Profile für neue und fortgesetzte Sitzungen, und der Watchdog bleibt auch dann aktiv, wenn das gesamte Interaktionsbudget unbegrenzt ist.

Entfernen Sie eine kurze Überschreibung der Gesamtzeitüberschreitung, um zum Standardwert von 48 Stunden zurückzukehren, oder legen Sie ein explizites Budget von beispielsweise 12 Stunden fest:

```bash
# Zum Standardwert von 48 Stunden zurückkehren:
openclaw config unset agents.defaults.timeoutSeconds

# Oder eine explizite Begrenzung von 12 Stunden festlegen:
openclaw config set agents.defaults.timeoutSeconds 43200
```

Innerhalb einer CLI gestartete Hintergrundarbeit bleibt Teil dieses CLI-Unterprozesses. Wenn die übergeordnete Interaktion ihre Gesamtbegrenzung erreicht, beendet OpenClaw den Unterprozess zusammen mit dessen CLI-internen Hintergrundaufgaben. Verwenden Sie für dauerhafte, lang laufende Aufgaben einen getrennten OpenClaw-[Sub-Agenten](/de/tools/subagents) oder [ACP-Agenten](/de/tools/acp-agents); getrennte Sub-Agenten haben standardmäßig keine Laufzeitbegrenzung.

Der Befehl `openclaw agent` besitzt außerdem eine eigene Anfragefrist. Sein Ausweichstandardwert von 600 Sekunden gilt für diesen Befehlsaufruf, nicht für gewöhnliche Gateway-Interaktionen; siehe [`openclaw agent`](/de/cli/agent).

### Besonderheiten der Claude CLI

Das gebündelte `claude-cli`-Backend bevorzugt die native Skill-Auflösung von Claude Code. Wenn der aktuelle Skills-Snapshot mindestens einen ausgewählten Skill mit einem materialisierten Pfad enthält, übergibt OpenClaw über `--plugin-dir` ein temporäres Claude-Code-Plugin und lässt den duplizierten OpenClaw-Skills-Katalog aus dem angehängten System-Prompt weg. Ohne einen materialisierten Plugin-Skill behält OpenClaw den Prompt-Katalog als Ausweichlösung bei. Überschreibungen von Skill-Umgebungsvariablen/API-Schlüsseln gelten für die Ausführung weiterhin in der Umgebung des untergeordneten Prozesses.

Die Claude CLI verfügt über einen eigenen nicht interaktiven Berechtigungsmodus; OpenClaw ordnet diesen der vorhandenen Ausführungsrichtlinie zu, anstatt Claude-spezifische Konfiguration hinzuzufügen. Für von OpenClaw verwaltete aktive Claude-Sitzungen ist die effektive Ausführungsrichtlinie maßgeblich: YOLO (`tools.exec.mode: "full"`) startet Claude normalerweise mit `--permission-mode bypassPermissions`, während eine restriktive Richtlinie es mit `--permission-mode default` startet. Als Root ausgeführte Gateways verwenden ebenfalls `default`, da Claude Code den Umgehungsmodus für Root ablehnt. Agentenspezifische `agents.entries.*.tools.exec`-Einstellungen überschreiben für diesen Agenten die globale Einstellung `tools.exec`. Das Anthropic-Plugin normalisiert die Berechtigungsflags von Claude entsprechend der effektiven Richtlinie und der Hostbeschränkung.

Unter einer restriktiven Richtlinie fragt Claude OpenClaw über stdio um Erlaubnis, bevor eines seiner nativen oder Erweiterungstools verwendet wird (seine eigenen Bash-, WebFetch- oder Claude-in-Chrome-Browsertools). Wenn die effektive Rückfrageeinstellung für die Ausführung `on-miss` oder `always` lautet, leitet OpenClaw jede Anfrage als interaktive Genehmigung an den Kanal der Sitzung weiter: **Einmal zulassen** erlaubt den einzelnen Aufruf, **Immer zulassen** erlaubt diesen Toolnamen für den Rest der aktiven Claude-Sitzung (nur im Arbeitsspeicher, niemals dauerhaft gespeichert), und **Ablehnen**, eine Zeitüberschreitung oder ein nicht erreichbarer Genehmigungspfad lehnen den Aufruf jeweils ab. Richtlinien, die niemals nachfragen, behalten ihr bisheriges Verhalten bei: `security: "deny"` lehnt jede Anfrage ab, und die Rückfrageeinstellung `off` mit weniger als vollständiger Sicherheit (Ausführungsmodus `allowlist`) lehnt ohne Nachfrage ab.

### Claude-Browsertools und Anmeldung bei 1Password

Claude Code kann über die [Claude-in-Chrome-Erweiterung](https://code.claude.com/docs/en/chrome) einen Chrome-Browser steuern, einschließlich des automatischen Ausfüllens von Anmeldedaten mit [1Password für Claude](/de/gateway/1password#browser-sign-in-with-1password-for-claude). Das gebündelte Backend aktiviert diese Funktion nicht; registrieren Sie ein [CLI-Backend-Plugin](/de/plugins/cli-backend-plugins), das `--chrome` an die Startargumente eines Backends mit `claude-stream-json`-Dialekt anhängt. OpenClaw behält ein konfiguriertes `--chrome` bei normalen Ausführungen bei und erzwingt bei Ausführungen mit einer eingeschränkten Tool-Richtlinie, etwa Nebenfragen, immer `--no-chrome`. Das Chrome-Fenster, die Erweiterung und alle Genehmigungsaufforderungen von 1Password befinden sich auf dem Gateway-Host, daher muss jemand an diesem Rechner anwesend sein, um die Verwendung von Anmeldedaten zu genehmigen.

Das Backend ordnet außerdem OpenClaw-`/think`-Stufen dem nativen `--effort`-Flag von Claude Code zu: `minimal`/`low` -> `low`, `medium` -> `medium`, und `high`/`xhigh`/`max` werden direkt weitergereicht. Dadurch bleiben die unterstützten Fable-5-Aufwandsstufen für abonnementbasierte Claude-CLI- und API-Schlüssel-Routen identisch. `adaptive` entfernt konfigurierte `--effort`-Flags und stellt keinen Ersatz bereit, sodass Claude Code den effektiven Aufwand aus seiner eigenen Umgebung, seinen Einstellungen und den Modellstandardwerten ermittelt. Bei anderen CLI-Backends muss das jeweils zuständige Plugin einen entsprechenden argv-Mapper deklarieren, bevor `/think` die gestartete CLI beeinflusst.

Bevor OpenClaw `claude-cli` verwenden kann, muss Claude Code selbst auf demselben Host angemeldet sein:

```bash
claude auth login
claude auth status --text
openclaw models auth login --provider anthropic --method cli --set-default
```

Bei Docker-Installationen muss Claude Code innerhalb des persistenten Container-Home-Verzeichnisses installiert und angemeldet sein, nicht nur auf dem Host; siehe [Claude-CLI-Backend in Docker](/de/install/docker#claude-cli-backend-in-docker).

Der Gateway-Dienst muss `claude` über `PATH` auflösen können. Registrieren Sie für einen nicht standardmäßigen Pfad
ein kleines Wrapper-Backend-Plugin.

## Sitzungen

- Wenn die CLI Sitzungen unterstützt, legen Sie `sessionArgs` mit einem `{sessionId}`-Platzhalter fest (zum Beispiel `["--session-id", "{sessionId}"]`).
- Wenn die CLI einen Fortsetzungsunterbefehl mit anderen Flags verwendet, legen Sie `resumeArgs` fest (ersetzt beim Fortsetzen `args`) und optional `resumeOutput` für Fortsetzungen ohne JSON.
- `sessionMode`:
  - `always`: Immer eine Sitzungs-ID senden (neue UUID, wenn keine gespeichert ist).
  - `existing`: Eine Sitzungs-ID nur senden, wenn zuvor eine gespeichert wurde.
  - `none`: Niemals eine Sitzungs-ID senden.
- `claude-cli` verwendet standardmäßig `liveSession: "claude-stdio"`, `output: "jsonl"` und `input: "stdin"`, sodass Folgeinteraktionen den aktiven Claude-Prozess wiederverwenden, solange er läuft. Dies gilt auch für benutzerdefinierte Konfigurationen, in denen Transportfelder fehlen. Wenn das Gateway neu gestartet oder der inaktive Prozess beendet wird, setzt OpenClaw die Sitzung anhand der gespeicherten Claude-Sitzungs-ID fort. Gespeicherte Sitzungs-IDs werden vor dem Fortsetzen anhand eines lesbaren Projekttranskripts überprüft; bei einem fehlenden Transkript wird die Bindung aufgehoben (protokolliert als `reason=transcript-missing`), anstatt unbemerkt eine neue Sitzung unter `--resume` zu starten.
- Aktive Claude-Sitzungen verwenden begrenzte Schutzmechanismen für JSONL-Ausgaben: 8 MiB und 20,000 rohe JSONL-Zeilen pro Interaktion.
- Gespeicherte CLI-Sitzungen stellen eine vom Provider verwaltete Kontinuität dar. Das automatische Zurücksetzen ist standardmäßig deaktiviert; `/reset` sowie explizite tägliche oder inaktivitätsbasierte `session.reset`-Richtlinien unterbrechen sie weiterhin.
- Neue CLI-Sitzungen werden normalerweise nur anhand der Compaction-Zusammenfassung von OpenClaw und des Abschnitts nach der Compaction neu initialisiert. Zur Wiederherstellung kurzer Sitzungen, die vor der Compaction ungültig wurden, kann ein Backend dies mit `reseedFromRawTranscriptWhenUncompacted: true` aktivieren. Die Neuinitialisierung anhand des Rohtranskripts bleibt begrenzt und auf sichere Ungültigkeitsfälle beschränkt, etwa ein fehlendes CLI-Transkript, ein verwaistes Ende einer Toolverwendung, Änderungen an Nachrichtenrichtlinie/System-Prompt/cwd/MCP oder ein Wiederholungsversuch nach Ablauf einer Sitzung; Änderungen am Authentifizierungsprofil oder an der Anmeldedatenepoche führen niemals zu einer Neuinitialisierung anhand des Rohtranskriptverlaufs.

Serialisierung: `serialize: true` hält Ausführungen innerhalb derselben Lane in Reihenfolge (die meisten CLIs serialisieren auf einer Provider-Lane). OpenClaw verwirft außerdem die Wiederverwendung gespeicherter CLI-Sitzungen, wenn sich die ausgewählte Authentifizierungsidentität ändert, einschließlich einer geänderten Authentifizierungsprofil-ID, eines statischen API-Schlüssels, eines statischen Tokens oder einer OAuth-Kontoidentität, sofern die CLI eine solche bereitstellt; allein die Rotation von OAuth-Zugriffs-/Aktualisierungstokens unterbricht die Sitzung nicht. Wenn eine CLI keine stabile OAuth-Konto-ID besitzt, überlässt OpenClaw dieser CLI die Durchsetzung ihrer eigenen Fortsetzungsberechtigungen.

## Ausweichpräambel aus claude-cli-Sitzungen

Wenn ein `claude-cli`-Versuch zu einem Nicht-CLI-Kandidaten in [`agents.defaults.model.fallbacks`](/de/concepts/model-failover) wechselt, versieht OpenClaw den nächsten Versuch mit einer Kontexteinleitung, die aus dem lokalen JSONL-Transkript von Claude Code übernommen wird (unter `~/.claude/projects/`, nach Workspace aufgeschlüsselt). Ohne diese Ausgangsbasis startet der Fallback-Provider ohne Kontext, da das sitzungseigene Transkript von OpenClaw bei `claude-cli`-Ausführungen leer ist.

- Die Einleitung bevorzugt die neueste `/compact`-Zusammenfassung oder `compact_boundary`-Markierung und fügt anschließend die jüngsten Dialogbeiträge nach der Grenze bis zum Zeichenbudget an. Dialogbeiträge vor der Grenze werden verworfen, da sie bereits durch die Zusammenfassung repräsentiert werden.
- Tool-Blöcke werden zu kompakten `(tool call: name)`- und `(tool result: …)`-Hinweisen zusammengeführt, damit das Prompt-Budget eingehalten wird; eine zu umfangreiche Zusammenfassung wird gekürzt und mit `(truncated)` gekennzeichnet.
- Fallbacks desselben Providers von `claude-cli` zu `claude-cli` nutzen Claudes eigenes `--resume` und überspringen die Einleitung.
- Die Ausgangsbasis verwendet die bestehende Validierung des Claude-Sitzungsdateipfads erneut, sodass keine beliebigen Pfade gelesen werden können.

## Bilder

Plugin-Autoren deklarieren die Unterstützung für Bildpfade mit `imageArg`:

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw schreibt Base64-Bilder in temporäre Dateien. Wenn `imageArg` festgelegt ist, werden diese Pfade als CLI-Argumente übergeben; andernfalls hängt OpenClaw die Dateipfade an den Prompt an (Pfadinjektion). Dies funktioniert bei CLIs, die lokale Dateien automatisch anhand einfacher Pfade laden.

## Ein- und Ausgaben

- `output: "text"` (Standard) behandelt stdout als endgültige Antwort.
- `output: "json"` versucht, JSON zu parsen und Text sowie eine Sitzungs-ID zu extrahieren.
- `output: "jsonl"` parst einen JSONL-Stream und extrahiert die abschließende Agent-Nachricht sowie, sofern vorhanden, Sitzungskennungen.
- Bei der JSON-Ausgabe der Gemini CLI liest OpenClaw den Antworttext aus `response` und die Nutzung aus `stats`, wenn `usage` fehlt oder leer ist. Der mitgelieferte Gemini-CLI-Adapter verwendet `stream-json`.

Eingabemodi:

- `input: "arg"` (Standard) übergibt den Prompt als letztes CLI-Argument.
- `input: "stdin"` sendet den Prompt über stdin.
- Wenn der Prompt sehr lang und `maxPromptArgChars` festgelegt ist, wird stattdessen stdin verwendet.

## Plugin-eigene Standardwerte

Die Standardwerte des CLI-Backends sind Teil der Plugin-Oberfläche:

- Plugins registrieren sie mit `api.registerCliBackend(...)`.
- Die Backend-`id` wird zum Provider-Präfix in Modellreferenzen.
- Das Verhalten von Befehl, argv, Umgebung, Parser, Sitzung und Watchdog verbleibt im Plugin-Code.
- Die Backend-spezifische Normalisierung verbleibt über den optionalen `normalizeConfig`-Hook im Besitz des Plugins.

Anthropic ist für `claude-cli` und Google für `google-gemini-cli` zuständig. OpenAI-Codex-Agent-Ausführungen verwenden über `openai/*` das Codex-App-Server-Harness; OpenClaw registriert kein mitgeliefertes `codex-cli`-Backend mehr.

Das mitgelieferte Anthropic-Plugin registriert für `claude-cli`:

| Schlüssel              | Wert                                                                                                                                                                                                          |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `command`             | `claude`                                                                                                                                                                                                      |
| `args`                | `-p --output-format stream-json --include-partial-messages --verbose --setting-sources user --allowedTools mcp__openclaw__* --disallowedTools ScheduleWakeup,CronCreate,Bash(run_in_background:true),Monitor` |
| `output`              | `jsonl`                                                                                                                                                                                                       |
| `input`               | `stdin`                                                                                                                                                                                                       |
| `modelArg`            | `--model`                                                                                                                                                                                                     |
| `sessionArgs`         | `["--session-id", "{sessionId}"]`                                                                                                                                                                             |
| `sessionMode`         | `always`                                                                                                                                                                                                      |
| `imageArg`            | `@`                                                                                                                                                                                                           |
| `imagePathScope`      | `workspace`                                                                                                                                                                                                   |
| `systemPromptFileArg` | `--append-system-prompt-file`                                                                                                                                                                                 |
| `systemPromptMode`    | `append`                                                                                                                                                                                                      |

Das mitgelieferte Google-Plugin registriert für `google-gemini-cli`:

| Schlüssel                  | Wert                                                                                   |
| -------------------------- | -------------------------------------------------------------------------------------- |
| `command`                 | `gemini`                                                                               |
| `args`                    | `--skip-trust --approval-mode auto_edit --output-format stream-json --prompt {prompt}` |
| `resumeArgs`              | ebenso, mit `--resume {sessionId}`                                                     |
| `output` / `resumeOutput` | `jsonl`                                                                                |
| `jsonlDialect`            | `gemini-stream-json`                                                                   |
| `imageArg`                | `@`                                                                                    |
| `imagePathScope`          | `workspace`                                                                            |
| `modelArg`                | `--model`                                                                              |
| `sessionMode`             | `existing`                                                                             |
| `sessionIdFields`         | `["session_id", "sessionId"]`                                                          |

Voraussetzung: Die lokale Gemini CLI muss installiert und unter `PATH` als `gemini` verfügbar sein (`brew install gemini-cli` oder `npm install -g @google/gemini-cli`).

Hinweise zur Ausgabe der Gemini CLI:

- Der standardmäßige `stream-json`-Parser liest Assistenten-`message`-Ereignisse, Tool-Ereignisse, die abschließende `result`-Nutzung und schwerwiegende Gemini-Fehlerereignisse.
- Die Nutzung greift auf `stats` zurück, wenn `usage` fehlt oder leer ist; `stats.cached` wird in OpenClaw-`cacheRead` normalisiert, und wenn `stats.input` fehlt, werden Eingabe-Token aus `stats.input_tokens - stats.cached` abgeleitet.

## Texttransformations-Overlays

Plugins, die kleine Kompatibilitätsanpassungen für Prompts oder Nachrichten benötigen, können bidirektionale Texttransformationen deklarieren, ohne einen Provider oder ein CLI-Backend zu ersetzen:

```typescript
api.registerTextTransforms({
  input: [{ from: /red basket/g, to: "blue basket" }],
  output: [{ from: /blue basket/g, to: "red basket" }],
});
```

`input` schreibt den an die CLI übergebenen System-Prompt und Benutzer-Prompt um. `output` schreibt gestreamten Assistententext und geparsten endgültigen Text um, bevor OpenClaw seine eigenen Steuerungsmarkierungen und die Kanalauslieferung verarbeitet; bei Provider-gestützten Modellaufrufen stellt es außerdem Zeichenfolgenwerte innerhalb strukturierter Tool-Aufrufargumente nach der Stream-Reparatur und vor der Tool-Ausführung wieder her. Unverarbeitete Provider-JSON-Fragmente bleiben unverändert; Verbraucher sollten die strukturierte Teil-, End- oder Ergebnispayload verwenden.

Legen Sie für CLIs, die Provider-spezifische JSONL-Ereignisse ausgeben, `jsonlDialect` in der Konfiguration dieses Backends fest: `claude-stream-json` für Claude-Code-kompatible Streams, `gemini-stream-json` für Gemini-CLI-`stream-json`-Ereignisse.

## Zuständigkeit für native Compaction

Einige CLI-Backends führen einen Agenten aus, der sein eigenes Transkript komprimiert. Daher darf OpenClaw seinen Sicherheits-Zusammenfasser nicht auf sie anwenden – andernfalls arbeitet dieser gegen die eigene Compaction des Backends und kann den Dialogbeitrag mit einem schweren Fehler abbrechen.

`claude-cli` hat keinen Harness-Endpunkt (Claude Code führt die Compaction intern durch), daher deklariert es `ownsNativeCompaction: true`, und der Compaction-Pfad von OpenClaw gibt den Sitzungseintrag unverändert zurück. OpenClaw übergibt das effektive Kontextbudget der Ausführung über die dokumentierte [`CLAUDE_CODE_AUTO_COMPACT_WINDOW`](https://code.claude.com/docs/en/env-vars) von Claude Code, sodass die native automatische Compaction an den konfigurierten Anthropic-`contextTokens`-Grenzwerten ausgerichtet bleibt. Sitzungen mit nativem Harness wie Codex werden stattdessen weiterhin an den Compaction-Endpunkt ihres Harnesses weitergeleitet.

```typescript
api.registerCliBackend({ id: "my-cli", ownsNativeCompaction: true /* ... */ });
```

Deklarieren Sie `ownsNativeCompaction` nur für ein Backend, das die Compaction tatsächlich selbst übernimmt: Es muss sein eigenes Transkript zuverlässig in der Nähe des Kontextfensters begrenzen und eine fortsetzbare Sitzung beibehalten (z. B. `--resume` / `--session-id`), andernfalls kann eine zurückgestellte Sitzung das Budget weiterhin überschreiten.

## MCP-Overlays des Bundles

CLI-Backends empfangen OpenClaw-Tool-Aufrufe nicht direkt, ein Backend kann jedoch mit `bundleMcp: true` ein generiertes MCP-Konfigurations-Overlay aktivieren. Aktuelles mitgeliefertes Verhalten:

- `claude-cli`: generierte strikte MCP-Konfigurationsdatei.
- `google-gemini-cli`: generierte Gemini-Systemeinstellungsdatei.

Wenn Bundle-MCP aktiviert ist, führt OpenClaw folgende Schritte aus:

- Es startet einen Loopback-HTTP-MCP-Server, der Gateway-Tools für den CLI-Prozess verfügbar macht und mit einer ausführungsbezogenen Kontextberechtigung (`OPENCLAW_MCP_TOKEN`) authentifiziert wird, die nur für den aktuellen Ausführungsversuch aktiv ist.
- Es bindet den Tool-Zugriff an den vom Gateway ausgewählten Sitzungs-, Konto- und Kanalkontext, anstatt den Headern des untergeordneten Prozesses zu vertrauen.
- Es lädt aktivierte Bundle-MCP-Server für den aktuellen Workspace und führt sie mit der vorhandenen MCP-Konfigurations- beziehungsweise Einstellungsstruktur des Backends zusammen.
- Es schreibt die Startkonfiguration unter Verwendung des Backend-eigenen Integrationsmodus aus dem zuständigen Plugin um.

Eingeschränkte Ausführungen wie Cron-Aufträge mit `toolsAllow` erfordern eine exakte
Backend-eigene Übersetzung. Das gebündelte `claude-cli`-Backend deaktiviert Claudes
native Tools sowie benutzer-, projekt- und lokale Anpassungen, einschließlich Hooks,
Plugins, Agenten, Skills und `CLAUDE.md`. Anschließend stellt es jedes zulässige
OpenClaw-Tool über den auf die Berechtigung beschränkten MCP-Server bereit. Dadurch verbleiben Richtlinien für Dateisystem,
Prozesse, Ausführung, Genehmigungen und Sandbox innerhalb von OpenClaw, anstatt die
Befugnisse auf Claudes native Tools oder Anpassungsprozesse auszuweiten. Dieselbe MCP-
Liste wird in Claudes generierter Konfiguration und erneut vom Gateway bei der Tool-
Auflistung und -Ausführung durchgesetzt. Vor dem Ausstellen der Berechtigung lehnt der Kern Backend-
Übersetzungen ab, die MCP-Berechtigungen außerhalb der ursprünglichen Zulassungsliste
nennen. Backends ohne exakte Übersetzung werden weiterhin nach dem Fail-Closed-Prinzip abgelehnt.

Wenn keine MCP-Server aktiviert sind, fügt OpenClaw dennoch eine strikte Konfiguration ein, wenn sich ein Backend für gebündeltes MCP entscheidet, sodass Hintergrundausführungen isoliert bleiben.

Sitzungsgebundene gebündelte MCP-Laufzeitumgebungen werden zur Wiederverwendung innerhalb einer Sitzung zwischengespeichert und nach 10 Minuten Inaktivität beendet. Einmalige eingebettete Ausführungen wie Authentifizierungsprüfungen, Slug-Generierung und Active-Memory-Abruf fordern am Ende der Ausführung eine Bereinigung an, damit stdio-Kindprozesse und streamfähige HTTP-/SSE-Streams die Ausführung nicht überdauern.

Für `claude-cli` wird ein kompatibles ausgewähltes oder geordnetes OpenClaw-OAuth-/Token-Profil
an diesen untergeordneten Claude-Prozess weitergeleitet. Dadurch sind agentenspezifische Profile
für den Turn maßgeblich, während Claudes native Host-Anmeldung erhalten bleibt, wenn kein kompatibles
Profil vorhanden ist.

## Obergrenze für den erneuten Verlaufskontext

Wenn eine neue CLI-Sitzung mit einem früheren OpenClaw-Transkript initialisiert wird (beispielsweise nach einem `session_expired`-Wiederholungsversuch), wird der gerenderte `<conversation_history>`-Block begrenzt, damit Prompts zur erneuten Initialisierung nicht übermäßig anwachsen. Der Standardwert beträgt 12,288 Zeichen (etwa 3,000 Token).

Claude-CLI-Backends skalieren diese Obergrenze stattdessen anhand des ermittelten Claude-Kontextfensters: Größere Kontextfenster erhalten einen größeren Ausschnitt des vorherigen Verlaufs bis zu einer festen Obergrenze; andere CLI-Backends behalten den konservativen Standardwert bei. Diese Obergrenze gilt ausschließlich für den Block mit dem vorherigen Verlauf im Prompt zur erneuten Initialisierung.

## Einschränkungen

- OpenClaw fügt keine Tool-Aufrufe in das CLI-Backend-Protokoll ein. Backends sehen Gateway-Tools nur, wenn sie sich für `bundleMcp: true` entscheiden.
- Streaming ist Backend-spezifisch: Einige Backends streamen JSONL, andere puffern bis zum Beenden.
- Strukturierte Ausgaben hängen vom eigenen JSON-Format der CLI ab.

## Fehlerbehebung

| Symptom               | Lösung                                                                                            |
| --------------------- | ---------------------------------------------------------------------------------------------- |
| CLI nicht gefunden         | Fügen Sie die CLI zu `PATH` des Gateway-Dienstes hinzu oder aktualisieren Sie den registrierten Befehl des zuständigen Plugins. |
| Falscher Modellname      | Aktualisieren Sie die `modelAliases`-Zuordnung des Plugins.                                                    |
| Keine Sitzungskontinuität | Prüfen Sie `sessionArgs` und `sessionMode` des Plugins.                                            |
| Bilder werden ignoriert        | Prüfen Sie `imageArg` des Plugins und die Unterstützung der CLI für Dateipfade.                                 |

## Verwandte Themen

- [Gateway-Betriebshandbuch](/de/gateway)
- [Lokale Modelle](/de/gateway/local-models)
