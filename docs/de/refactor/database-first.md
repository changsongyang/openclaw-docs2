---
read_when:
    - OpenClaw-Laufzeitdaten, Cache, Transkripte, Aufgabenstatus oder temporäre Dateien nach SQLite verschieben
    - Entwurf von Doctor-Migrationen aus veralteten JSON- oder JSONL-Dateien
    - Verhalten von Backup, Wiederherstellung, VFS oder Worker-Speicher ändern
    - Entfernen von Sitzungssperren, Bereinigung, Kürzung oder JSON-Kompatibilitätspfaden
summary: Migrationsplan, um SQLite zur primären dauerhaften Zustands- und Cache-Schicht zu machen, während die Konfiguration dateibasiert bleibt
title: Datenbankzentriertes Zustands-Refactoring
x-i18n:
    generated_at: "2026-07-24T20:41:56Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: ae4d72f04c1228742cc7ea3cc87a96b06aa1e2b750ace23cca5b387844746186
    source_path: refactor/database-first.md
    workflow: 16
---

# Datenbankzentriertes Zustands-Refactoring

## Entscheidung

Verwenden Sie ein zweistufiges SQLite-Layout:

- Globale Datenbank: `~/.openclaw/state/openclaw.sqlite`
- Agentendatenbank: eine SQLite-Datenbank pro Agent für den agenteneigenen Arbeitsbereich,
  das Transkript, VFS, Artefakte und umfangreiche agentenspezifische Laufzeitdaten
- Die Konfiguration bleibt dateibasiert: `openclaw.json` verbleibt außerhalb der
  Datenbank. Laufzeit-Authentifizierungsprofile werden nach SQLite verschoben; externe Provider- oder CLI-
  Anmeldedatendateien werden weiterhin außerhalb der OpenClaw-Datenbank von ihren Eigentümern verwaltet.

Die globale Datenbank ist die Datenbank der Steuerungsebene. Sie verwaltet die Agentenerkennung,
den gemeinsamen Gateway-Zustand, Kopplungen, den Geräte-/Node-Zustand, Aufgaben- und Ablaufprotokolle, den Plugin-
Zustand, den Laufzeitzustand des Schedulers, Sicherungsmetadaten und den Migrationszustand.

Die Agentendatenbank ist die Datenbank der Datenebene. Sie verwaltet die Sitzungsmetadaten
des Agenten, den Transkript-Ereignisstrom, den VFS-Arbeitsbereich oder temporären Namensraum, Tool-
Artefakte, Ausführungsartefakte und durchsuchbare beziehungsweise indizierbare agentenlokale Cache-Daten.

Dies ermöglicht eine dauerhafte globale Sicht, ohne große Agentenarbeitsbereiche,
Transkripte und binäre temporäre Daten in den gemeinsamen Gateway-Schreibkanal zu zwingen.

## Verbindlicher Vertrag

Diese Migration hat eine einzige kanonische Laufzeitform:

- Sitzungszeilen speichern ausschließlich Sitzungsmetadaten. Sie dürfen weder
  `transcriptLocator` noch Transkriptdateipfade, benachbarte JSONL-Pfade, Sperrpfade,
  Bereinigungsmetadaten oder Kompatibilitätsverweise aus der Dateiära speichern.
- Die Transkriptidentität ist immer eine SQLite-Identität: `{agentId, sessionId}` sowie
  optionale Themenmetadaten, sofern das Protokoll sie benötigt.
- `sqlite-transcript://...` ist keine Laufzeit- oder Protokollidentität. Neuer Code darf
  keine Transkript-Locators ableiten, speichern, übergeben, parsen oder migrieren. Laufzeit und
  Tests sollten überhaupt keine Pseudo-Locators enthalten; die Dokumentation darf die Zeichenfolge
  nur erwähnen, um sie zu verbieten.
- Veraltete `sessions.json`, Transkript-JSONL, `.jsonl.lock`, Bereinigung, Kürzung
  und alte Sitzungspfadlogik gehören ausschließlich in den Migrations-/Importpfad von Doctor.
- Veraltete Aliasnamen der Sitzungskonfiguration gehören ausschließlich in die Doctor-Migration. Die Laufzeit
  interpretiert weder `session.idleMinutes` noch `session.resetByType.dm` oder
  agentenübergreifende `agent:main:*`-Aliasnamen für die Hauptsitzung eines anderen konfigurierten Agenten.
- Die Sitzungsrouting-Identität ist typisierter relationaler Zustand. Häufig ausgeführte Laufzeit- und UI-Pfade
  sollten `sessions.session_scope`, `sessions.account_id`,
  `sessions.primary_conversation_id`, `conversations` und
  `session_conversations` lesen; sie dürfen `session_key` nicht parsen oder
  `session_entries.entry_json` nach der Provider-Identität durchsuchen, außer als
  Kompatibilitätsschatten, während alte Aufrufstellen gelöscht werden.
- Direktnachrichtenmarkierungen auf Kanalebene wie `dm` gegenüber `direct` sind Routing-
  Vokabular, keine Transkript-Locators oder Kompatibilitäts-Handles für den Dateispeicher.
- Veraltete Konfiguration für Hook-Handler gehört ausschließlich in die Warnungs-/Migrationsoberflächen von Doctor.
  Die Laufzeit darf `hooks.internal.handlers` nicht laden; Hooks werden ausschließlich über erkannte
  Hook-Verzeichnisse und `HOOK.md`-Metadaten ausgeführt.
- Laufzeitstart, häufig ausgeführte Antwortpfade, Compaction, Zurücksetzen, Wiederherstellung, Diagnose,
  TTS, Speicher-Hooks, Subagenten, Plugin-Befehlsrouting, Protokollgrenzen und
  Hooks müssen `{agentId, sessionId}` durch die Laufzeit übergeben.
- Tests sollten SQLite-Transkriptzeilen über
  `{agentId, sessionId}` anlegen und prüfen. Tests, die ausschließlich die Weiterleitung von JSONL-Pfaden,
  die Beibehaltung von durch Aufrufer bereitgestellten Locators oder die Kompatibilität mit Transkriptdateien belegen,
  sollten gelöscht werden, sofern sie nicht den Doctor-Import, die Materialisierung von Unterstützungs-/Debugmaterial
  außerhalb von Sitzungen oder die Protokollform abdecken.
- `runEmbeddedPiAgent(...)`, vorbereitete Worker-Ausführungen und der innere eingebettete
  Versuch dürfen keine Transkript-Locators akzeptieren. Sie öffnen den SQLite-Transkript-
  Manager anhand von `{agentId, sessionId}` und übergeben diesen Manager an die internalisierte
  PI-kompatible Agentensitzung, sodass veraltete Aufrufer den Runner nicht zum Schreiben von
  JSON-/JSONL-Transkripten veranlassen können.
- Runner-Diagnosen müssen Laufzeit-/Cache-/Payload-Ablaufverfolgungsdatensätze in SQLite speichern.
  Laufzeitdiagnosen dürfen keine Override-Optionen für JSONL-Dateien oder -Pfade oder generische
  Hilfsfunktionen zum Exportieren von Transkript-JSONL bereitstellen; benutzerseitige Exporte können explizite
  Artefakte aus Datenbankzeilen materialisieren, ohne Dateinamen wieder in die Laufzeit einzuspeisen.
- Die Rohdatenstromprotokollierung verwendet `OPENCLAW_RAW_STREAM=1` sowie SQLite-Diagnosezeilen.
  Der alte pi-mono-Vertrag für die Dateiprotokollierer `PI_RAW_STREAM`, `PI_RAW_STREAM_PATH` und
  `raw-openai-completions.jsonl` ist nicht Bestandteil der OpenClaw-
  Laufzeit oder ihrer Tests.
- Die QMD-Speicherindizierung darf SQLite-Transkripte nicht in Markdown-Dateien exportieren.
  QMD indiziert ausschließlich konfigurierte Speicherdateien; die Suche in Sitzungstranskripten bleibt
  SQLite-basiert.
- Der QMD-SDK-Unterpfad ist für neuen Code ausschließlich für QMD vorgesehen. Hilfsfunktionen zur
  Indizierung von SQLite-Sitzungstranskripten befinden sich unter `memory-core-host-engine-session-transcripts`; jeder
  QMD-Reexport dient nur der Kompatibilität und darf nicht von Laufzeitcode verwendet werden.
- Integrierte Speicherindizes befinden sich in der jeweiligen Agentendatenbank. Laufzeitkonfiguration und
  aufgelöste Laufzeitverträge dürfen `memorySearch.store.path` nicht offenlegen; Doctor
  löscht diesen veralteten Konfigurationsschlüssel, und aktueller Code übergibt die
  `databasePath` des Agenten intern.

Bei der Implementierung sollte weiterhin Code gelöscht werden, bis diese Aussagen
ohne Ausnahmen außerhalb der Doctor-/Import-/Export-/Debug-Grenzen zutreffen.

## Zielzustand und Fortschritt

### Verbindliches Ziel

- Eine globale SQLite-Datenbank verwaltet den Zustand der Steuerungsebene:
  `state/openclaw.sqlite`.
- Eine SQLite-Datenbank pro Agent verwaltet den Zustand der Datenebene:
  `agents/<agentId>/agent/openclaw-agent.sqlite`.
- Die Konfiguration bleibt dateibasiert. `openclaw.json` ist nicht Bestandteil dieses Datenbank-
  Refactorings.
- Veraltete Dateien dienen ausschließlich als Migrationseingaben für Doctor.
- Die Laufzeit schreibt oder liest Sitzungs- oder Transkript-JSONL niemals als aktiven Zustand.

### Zielzustände

- `not-started`: Laufzeitcode aus der Dateiära schreibt weiterhin aktiven Zustand.
- `migrating`: Doctor-/Importcode kann Dateidaten nach SQLite verschieben.
- `dual-read`: Eine temporäre Brücke liest sowohl SQLite als auch veraltete Dateien. Dieser Zustand
  ist für dieses Refactoring verboten, sofern er nicht ausdrücklich als
  ausschließlich für Doctor vorgesehen dokumentiert ist.
- `sqlite-runtime`: Die Laufzeit liest und schreibt ausschließlich SQLite.
- `clean`: Veraltete Laufzeit-APIs und Tests wurden entfernt, und die Schutzprüfung verhindert
  Regressionen.
- `done`: Dokumentation, Tests, Sicherung, Doctor-Migration und Prüfungen der Änderungen belegen den
  bereinigten Zustand.

### Aktueller Zustand

- Sitzungen: `clean` für die Laufzeit. Sitzungszeilen befinden sich in der Agentendatenbank,
  Laufzeit-APIs verwenden `{agentId, sessionId}` oder `{agentId, sessionKey}`, und
  `sessions.json` dient ausschließlich als veraltete Eingabe für Doctor.
- Transkripte: `clean` für die Laufzeit. Transkriptereignisse, Identitäten, Snapshots
  und Trajektorien-Laufzeitereignisse befinden sich in der Agentendatenbank. Die Laufzeit
  akzeptiert keine Transkript-Locators oder JSONL-Transkriptpfade mehr.
- Eingebetteter PI-Runner: `clean`. Eingebettete PI-Ausführungen, vorbereitete Worker, Compaction
  und Wiederholungsschleifen verwenden den SQLite-Sitzungsbereich und weisen veraltete Transkript-Handles zurück.
- Cron: `clean` für die Laufzeit. Die Laufzeit verwendet `cron_jobs` und Cron-eigene `task_runs`;
  Laufzeittests verwenden die SQLite-Namensgebung `storeKey`, und Cron-Pfade aus der Dateiära verbleiben
  ausschließlich in Tests der veralteten Doctor-Migration.
- Aufgabenregistrierung: `clean`. Laufzeitzeilen für Aufgaben und TaskFlow befinden sich in
  `state/openclaw.sqlite`; nicht ausgelieferte SQLite-Importer für Sidecar-Dateien wurden gelöscht.
- Plugin-Zustand: `clean`. Zeilen für Plugin-Zustände/-Blobs befinden sich in der gemeinsamen globalen
  Datenbank; Schutzprüfungen verhindern die Nutzung alter SQLite-Hilfsfunktionen für Plugin-Zustands-Sidecars.
- Speicher: `sqlite-runtime` für den integrierten Speicher und die Indizierung von Sitzungstranskripten.
  Tabellen für Speicherindizes befinden sich in der Agentendatenbank, der Plugin-Speicherzustand verwendet
  gemeinsame Plugin-Zustandszeilen, und veraltete Speicherdateien sind Migrationseingaben für Doctor
  oder Inhalte des Benutzerarbeitsbereichs.
- Sicherung: `sqlite-runtime`. Die Sicherung stellt kompakte SQLite-Snapshots bereit, lässt aktive
  WAL-/SHM-Sidecars aus, überprüft die SQLite-Integrität und zeichnet Sicherungsläufe in der
  globalen Datenbank auf.
- Arbeitsbereichseinrichtung: `sqlite-runtime`. Der Abschluss der Einrichtung, Arbeitsbereichsbestätigungen
  und generierte Bootstrap-Hashes befinden sich in typisierten gemeinsamen SQLite-Tabellen. Die Laufzeit
  liest oder schreibt die außer Betrieb genommenen Arbeitsbereichs-JSON- und `.attested`-Sidecars nicht;
  Doctor verwaltet ihren validierten Import und ihre verifizierte Entfernung.
- Doctor-Migration: `migrating`, absichtlich. Doctor importiert veraltete JSON-,
  JSONL- und außer Betrieb genommene Sidecar-Speicher nach SQLite, zeichnet Migrationsläufe/-quellen auf
  und entfernt erfolgreich migrierte Quellen.
- Ausführungsgenehmigungen: `file-runtime`. TypeScript und macOS lesen und schreiben weiterhin
  `exec-approvals.json` des aktiven Zustandsverzeichnisses; das reservierte
  Schema `exec_approvals_config` hat noch keinen Laufzeiteigentümer. Eine zukünftige Umstellung muss
  einen Doctor-Import innerhalb desselben Zustands hinzufügen und beide Laufzeiten gemeinsam umstellen.
- E2E-Skripte: `clean` für die Laufzeitabdeckung. Docker-MCP-Seeding schreibt SQLite-
  Zeilen. Das Docker-Skript für den Laufzeitkontext erstellt veraltete JSONL-Dateien ausschließlich innerhalb des
  Doctor-Migrations-Seeds und benennt den Pfad des veralteten Sitzungsindex ausdrücklich.

### Verbleibende Arbeiten

- [x] Variablen für Cron-Laufzeittest-Speicher umbenennen, sodass sie nicht mehr `storePath` verwenden, außer
      wenn es sich um veraltete Doctor-Eingaben handelt.
      Dateien: `src/cron/service.test-harness.ts`,
      `src/cron/service.runs-one-shot-main-job-disables-it.test.ts`,
      `src/cron/service/timer.regression.test.ts`,
      `src/cron/service/ops.test.ts`, `src/cron/service/store.test.ts`,
      `src/cron/service.heartbeat-ok-summary-suppressed.test.ts`,
      `src/cron/service.main-job-passes-heartbeat-target-last.test.ts`,
      `src/cron/store.test.ts`.
      Nachweis: `pnpm check:database-first-legacy-stores`; `rg -n 'storePath' src/cron --glob '!**/commands/doctor/**'`.
- [x] Veraltete Export-Test-Mocks aus der Dateiära entfernen oder umbenennen.
      Datei: `src/auto-reply/reply/commands-export-test-mocks.ts`.
      Nachweis: `rg -n 'resolveSessionFilePath|sessionFile|storePath|transcriptLocator' src/auto-reply/reply`.
- [x] Den veralteten JSONL-Seed des Docker-Laufzeitkontexts eindeutig als ausschließlich für Doctor vorgesehen kennzeichnen.
      Datei: `scripts/e2e/session-runtime-context-docker-client.ts`.
      Nachweis: `rg -n 'sessions\\.json|sessionFile|\\.jsonl' scripts/e2e/session-runtime-context-docker-client.ts` zeigt ausschließlich
      `seedBrokenLegacySessionForDoctorMigration`.
- [x] Die generierten Kysely-Typen nach jeder Schemaänderung synchron halten.
      Dateien: `src/state/openclaw-state-schema.sql`,
      `src/state/openclaw-agent-schema.sql`,
      `src/state/*generated*`.
      Nachweis: keine Schemaänderung in diesem Durchlauf; `pnpm db:kysely:check`;
      `pnpm lint:kysely`.
- [x] Fokussierte Tests für betroffene Speicher, Befehle und Skripte erneut ausführen.
      Nachweis: `pnpm test src/cron/service/store.test.ts src/cron/store.test.ts src/cron/service.heartbeat-ok-summary-suppressed.test.ts src/cron/service.main-job-passes-heartbeat-target-last.test.ts src/cron/service.every-jobs-fire.test.ts src/cron/service.persists-delivered-status.test.ts src/cron/service.runs-one-shot-main-job-disables-it.test.ts src/cron/service/ops.test.ts src/cron/service/timer.regression.test.ts src/auto-reply/reply/commands-export-session.test.ts extensions/telegram/src/thread-bindings.test.ts extensions/slack/src/monitor/message-handler/prepare.test.ts src/acp/translator.session-lineage-meta.test.ts`; `git diff --check`.
- [x] Vor der Erklärung von `done` die Prüfung der Änderungen oder einen umfassenden Remote-Nachweis ausführen.
      Nachweis: `pnpm check:changed --timed -- <changed extension paths>` wurde im
      Hetzner-Crabbox-Lauf `run_3f1cabf6b25c` nach einer temporären Node-24-/pnpm-Einrichtung und
      explizitem Pfadrouting für den synchronisierten Arbeitsbereich ohne `.git` erfolgreich ausgeführt.

### Regressionen verhindern

- Keine Transkript-Locators.
- Keine aktiven Sitzungsdateien.
- Keine fingierten JSONL-Test-Fixtures, außer in Tests veralteter Doctor-Migrationen.
- Kein roher SQLite-Zugriff, wo Kysely erwartet wird.
- Keine neuen Datenbankmigrationen aus der Dateiära. Das globale Schema bleibt bei Version `1`.
  Das ausgelieferte Agentenschema der Version `1` verfügt über eine begrenzte Laufzeitmigration auf
  Version `2` für stabile Identitäten von Speicherquellen.

## Annahmen aus der Codeanalyse

Keine nachgelagerten Produktentscheidungen blockieren diesen Plan. Die Implementierung sollte
unter folgenden Annahmen fortgesetzt werden:

- Verwenden Sie `node:sqlite` direkt und setzen Sie für diesen Speicherpfad eine WAL-Reset-sichere Node-Laufzeit
  (22.22.3+, 24.15+ oder 25.9+) voraus.
- Behalten Sie genau eine normale Konfigurationsdatei bei. Verschieben Sie bei diesem Refactoring weder die Konfiguration noch Plugin-
  Manifeste oder Git-Arbeitsbereiche nach SQLite.
- Kompatibilitätsdateien für die Laufzeit sind nicht erforderlich. Ältere JSON- und JSONL-Dateien dienen
  ausschließlich als Migrationseingaben. Die zweiglokalen SQLite-Begleitdateien wurden nie veröffentlicht und werden
  gelöscht statt importiert.
- `openclaw doctor --fix` ist für die Migration älterer Dateien in die Datenbank zuständig. Der Start der Laufzeit
  ist ausschließlich für begrenzte Upgrades zwischen veröffentlichten SQLite-Schemaversionen zuständig;
  er darf keinen Zustand aus der Dateiära importieren.
- Für die Kompatibilität von Anmeldedaten gilt dieselbe Regel: Laufzeitanmeldedaten befinden sich in
  SQLite. Alte `auth-profiles.json`-, agentenspezifische `auth.json`- und gemeinsam genutzte
  `credentials/oauth.json`-Dateien dienen als Migrationseingaben für Doctor und werden
  nach dem Import entfernt.
- Der generierte Modellkatalogzustand wird in der Datenbank gespeichert. Laufzeitcode darf
  `agents/<agentId>/agent/models.json` nicht schreiben; vorhandene `models.json`-Dateien sind ältere
  Eingaben für Doctor und werden nach dem Import in `agent_model_catalogs` entfernt.
- Die Laufzeit darf Transkript-Locators weder migrieren, normalisieren noch überbrücken. Die aktive
  Transkriptidentität ist `{agentId, sessionId}` in SQLite. Dateipfade sind
  ausschließlich ältere Eingaben für Doctor, und `sqlite-transcript://...` muss aus
  Laufzeit-, Protokoll-, Hook- und Plugin-Oberflächen verschwinden, statt als
  Grenz-Handle behandelt zu werden.
- SQLite-Transkriptlesevorgänge der Laufzeit führen keine Migrationen alter JSONL-Eintragsformen aus und
  schreiben nicht aus Kompatibilitätsgründen ganze Transkripte neu. Die Normalisierung älterer Einträge verbleibt in
  expliziten Doctor-/Import-Dienstprogrammen. Doctor normalisiert ältere JSONL-Transkriptdateien,
  bevor SQLite-Zeilen eingefügt werden; aktuelle Laufzeitzeilen werden
  bereits im aktuellen Transkriptschema geschrieben. Der Trajektorien-/Sitzungsexport
  liest diese Zeilen unverändert und darf beim Export keine älteren Migrationen durchführen.
- Parser-/Migrationshelfer für ältere Transkript-JSONL-Dateien sind ausschließlich für Doctor bestimmt. Der
  Transkriptformatcode der Laufzeit erstellt nur den aktuellen SQLite-Transkriptkontext; Doctor
  ist für Upgrades alter JSONL-Einträge vor dem Einfügen der Zeilen zuständig.
- Der alte, von der Laufzeit verwaltete Streaming-Helfer für JSONL-Transkripte wurde gelöscht. Der
  Doctor-Importcode ist für explizite Lesevorgänge älterer Dateien zuständig; der Laufzeit-Sitzungsverlauf liest
  SQLite-Zeilen.
- Codex-App-Server-Bindungen verwenden den OpenClaw-`sessionId` als kanonischen
  Schlüssel im Codex-Plugin-Zustandsnamensraum. `sessionKey` sind Metadaten für
  Routing/Anzeige und dürfen weder die dauerhafte Sitzungs-ID ersetzen noch
  die Identität der Transkriptdatei wiederbeleben.
- Kontext-Engines erhalten den aktuellen Laufzeitvertrag direkt. Die Registry
  darf Engines nicht mit Wiederholungs-Shims umschließen, die `sessionKey`,
  `transcriptScope` oder `prompt` löschen; Engines, die die aktuellen
  datenbankorientierten Parameter nicht akzeptieren können, sollten mit einem eindeutigen Fehler abbrechen, statt überbrückt zu werden.
- Die Sicherungsausgabe sollte aus einer einzigen Archivdatei bestehen. Datenbankinhalte sollten
  als kompakte SQLite-Snapshots in dieses Archiv aufgenommen werden, nicht als unverarbeitete aktive WAL-Begleitdateien.
- Die Transkriptsuche ist nützlich, aber für die erste datenbankorientierte
  Umsetzung nicht erforderlich. Gestalten Sie das Schema so, dass FTS später hinzugefügt werden kann.
- Die Worker-Ausführung sollte während der Stabilisierung der Datenbankgrenze hinter Einstellungen
  weiterhin experimentell bleiben.

## Erkenntnisse aus der Codeanalyse

Der aktuelle Zweig hat die Proof-of-Concept-Phase bereits hinter sich. Die gemeinsam genutzte
Datenbank ist vorhanden, Node-`node:sqlite` ist über einen kleinen Laufzeithelfer eingebunden und
frühere Speicher schreiben nun in `state/openclaw.sqlite` oder die jeweils zuständige
`openclaw-agent.sqlite`-Datenbank.

Bei den verbleibenden Arbeiten geht es nicht um die Entscheidung für SQLite, sondern darum, die neue Grenze sauber zu halten
und alle kompatibilitätsorientierten Schnittstellen zu löschen, die noch der alten
Dateiwelt ähneln:

- Sitzungs-`storePath` ist weder eine Laufzeitidentität noch eine Test-Fixture-Form oder
  ein Statusnutzlastfeld. Laufzeit- und Brückentests enthalten den
  Vertragsnamen `storePath` nicht mehr; Doctor-/Migrationscode ist für dieses ältere Vokabular zuständig.
- Sitzungsschreibvorgänge werden nicht mehr über die alte prozessinterne `store-writer.ts`-
  Warteschlange geleitet. SQLite-Patch-Schreibvorgänge werden außerhalb der Transaktion vorbereitet und verwenden anschließend eine kurze,
  synchrone Validierungs-/Anwendungstransaktion mit expliziter Konflikterkennung.
- Die Ermittlung älterer Pfade hat weiterhin gültige Migrationszwecke, aber der Laufzeitcode sollte
  `sessions.json` und Transkript-JSONL-Dateien nicht mehr als mögliche Schreibziele
  behandeln.
- Agenteneigene Tabellen befinden sich in agentenspezifischen SQLite-Datenbanken. Die globale Datenbank enthält
  Registry-/Steuerungsebenenzeilen; die Transkriptidentität ist `{agentId, sessionId}` in
  den agentenspezifischen Transkriptzeilen. Laufzeitcode darf weder Transkriptdateipfade
  speichern noch Transkript-Locators migrieren.
- Doctor importiert bereits mehrere ältere Dateien. Bei der Bereinigung geht es darum, daraus eine
  einzige explizite Migrationsimplementierung zu machen, die Doctor aufruft und die einen dauerhaften
  Migrationsbericht erstellt.

Keine weiteren Produktfragen blockieren die Implementierung.

## Aktuelle Codestruktur

Der Zweig verfügt bereits über eine echte gemeinsam genutzte SQLite-Basis:

- Die Mindestanforderung an die Laufzeit erfordert jetzt einen WAL-Reset-sicheren Node-Build: 22.22.3+,
  24.15+ oder 25.9+. `package.json`, die Laufzeitprüfung der CLI, die Standardwerte des Installationsprogramms,
  die macOS-Laufzeitsuche, die CI und die öffentliche Installationsdokumentation stimmen alle überein.
- `src/state/openclaw-state-db.ts` öffnet `openclaw.sqlite`, legt WAL fest,
  `synchronous=NORMAL`, `busy_timeout=30000`, `foreign_keys=ON` und wendet
  das generierte Schemamodul an, das aus
  `src/state/openclaw-state-schema.sql` abgeleitet wurde.
- Kysely-Tabellentypen und Laufzeit-Schemamodule werden aus temporären
  SQLite-Datenbanken generiert, die aus den eingecheckten `.sql`-Dateien erstellt werden; der Laufzeitcode
  enthält keine kopierten Schemazeichenfolgen mehr für globale, agentenspezifische oder Proxy-
  Erfassungsdatenbanken.
- Laufzeitspeicher leiten Typen ausgewählter und eingefügter Zeilen aus diesen generierten
  Kysely-`DB`-Schnittstellen ab, statt SQLite-Zeilenstrukturen manuell nachzubilden. Rohes SQL
  bleibt auf Schemaanwendung, Pragmas und ausschließlich für Migrationen verwendetes DDL beschränkt.
- Das globale SQLite-Schema bleibt bei `user_version = 1`. Das agentenspezifische Schema
  hat Version `2`; seine Öffnungsroutine migriert den in Version `1` ausgelieferten
  Speicherdatenquellenschlüssel atomar zu einer stabilen Ganzzahlidentität. Der Import aus Dateien
  in die Datenbank verbleibt im Doctor-Code.
- Relationale Eigentumsverhältnisse werden dort durchgesetzt, wo die Eigentumsgrenze maßgeblich ist:
  Zeilen der Quellenmigration werden aus `migration_runs` kaskadiert, der Task-Zustellungsstatus
  aus `task_runs` und Zeilen der Transkriptidentität aus
  Transkriptereignissen.
- Zu den aktuellen gemeinsamen Tabellen gehören `agent_databases`,
  `auth_profile_stores`, `auth_profile_state`,
  `plugin_state_entries`, `plugin_blob_entries`, `media_blobs`,
  `skill_uploads`, `capture_sessions`, `capture_events`, `capture_blobs`,
  `sandbox_registry_entries`, `cron_jobs`, `commitments`,
  `delivery_queue_entries`, `model_capability_cache`,
  `workspace_setup_state`, `workspace_path_aliases`, `workspace_attestations`,
  `workspace_generated_bootstrap_hashes`, `native_hook_relay_bridges`,
  `current_conversation_bindings`, `plugin_binding_approvals`,
  `tui_last_sessions`, `acp_sessions`, `acp_replay_sessions`,
  `acp_replay_events`, `task_runs`, `task_delivery_state`, `flow_runs`,
  `subagent_runs`, `migration_runs` und `backup_runs`.
- Beliebiger Plugin-eigener Zustand erhält keine hosteigenen typisierten Tabellen. Installierte
  Plugins verwenden `plugin_state_entries` für versionierte JSON-Nutzdaten und
  `plugin_blob_entries` für Bytes, einschließlich Eigentumszuordnung über Namensraum und Schlüssel, TTL-Bereinigung,
  Sicherung und Plugin-Migrationsdatensätzen. Vom Host verwalteter Plugin-Orchestrierungszustand kann
  weiterhin typisierte Tabellen erhalten, wenn der Host den Abfragevertrag besitzt, beispielsweise
  `plugin_binding_approvals`.
- Plugin-Migrationen sind Datenmigrationen über Plugin-eigene Namensräume und keine
  Host-Schemamigrationen. Ein Plugin kann seine eigenen versionierten Zustands-/Blob-Einträge
  über einen Migrations-Provider migrieren, und der Host erfasst Quellen-/Ausführungsstatus im
  normalen Migrationsjournal. Neue Plugin-Installationen erfordern keine Änderung von
  `openclaw-state-schema.sql`, sofern nicht der Host selbst die Verantwortung für einen
  neuen Plugin-übergreifenden Vertrag übernimmt.
- `src/state/openclaw-agent-db.ts` öffnet
  `agents/<agentId>/agent/openclaw-agent.sqlite`, registriert die Datenbank in der
  globalen DB und verwaltet agentenlokale Sitzungs-, Transkript-, VFS-, Artefakt-, Cache-
  und Speicherindextabellen. Die gemeinsame Laufzeiterkennung liest jetzt die generiert typisierte
  `agent_databases`-Registry, statt diese Abfrage an jeder Aufrufstelle neu zu implementieren.
- Globale und agentenspezifische Datenbanken erfassen eine `schema_meta`-Zeile mit Datenbankrolle,
  Schemaversion, Zeitstempeln und Agenten-ID für Agentendatenbanken. Die globale DB
  bleibt bei `user_version = 1`; agentenspezifische DBs verwenden nach der begrenzten
  Migration der Speicherdatenquellenidentität Version `2`.
- Die agentenspezifische Sitzungsidentität verfügt jetzt über eine maßgebliche `sessions`-Stammtabelle mit
  `session_id` als Schlüssel sowie `session_key`, `session_scope`, `account_id`,
  `primary_conversation_id`, Zeitstempeln, Anzeigefeldern, Modellmetadaten,
  Harness-ID und Eltern-/Erzeugungsverknüpfung als abfragbare Spalten. `session_routes`
  ist der eindeutige aktive Routenindex von `session_key` zur aktuellen
  `session_id`, sodass ein Routenschlüssel zu einer neuen dauerhaften Sitzung wechseln kann, ohne
  dass häufige Lesezugriffe zwischen doppelten `sessions.session_key`-Zeilen wählen müssen. Die alte
  kompatibilitätsgeformte `session_entries.entry_json`-Nutzlast ist über einen Fremdschlüssel an die
  dauerhafte `session_id`-Stammtabelle angehängt; sie ist nicht mehr die einzige
  Darstellung einer Sitzung auf Schemaebene.
- Auch die agentenspezifische externe Konversationsidentität ist relational:
  `conversations` speichert die normalisierte Provider-/Konto-/Konversationsidentität und
  `session_conversations` verknüpft eine OpenClaw-Sitzung mit einer oder mehreren externen
  Konversationen. Dies deckt gemeinsam genutzte Haupt-DM-Sitzungen ab, bei denen mehrere Kommunikationspartner
  absichtlich einer Sitzung zugeordnet werden können, ohne falsche Angaben in `session_key` zu erzeugen. SQLite
  erzwingt außerdem die Eindeutigkeit der natürlichen Provider-Identität, sodass dasselbe
  Kanal-/Konto-/Art-/Kommunikationspartner-/Thread-Tupel nicht auf mehrere Konversations-IDs aufgeteilt werden kann.
  Direkte Kommunikationspartner in gemeinsam genutzten Hauptsitzungen werden mit einer `participant`-Rolle verknüpft, sodass eine
  OpenClaw-Sitzung mehrere externe DM-Kommunikationspartner darstellen kann, ohne
  ältere Kommunikationspartner zu unspezifischen zugehörigen Zeilen herabzustufen. `sessions.primary_conversation_id`
  verweist weiterhin auf das aktuelle typisierte Zustellungsziel. Geschlossene Routing-/Statusspalten
  werden mit SQLite-`CHECK`-Constraints erzwungen, statt sich ausschließlich auf
  TypeScript-Unions zu verlassen.
  Die Laufzeitprojektion der Sitzung entfernt kompatibilitätsbezogene Routing-Schatten aus
  `session_entries.entry_json`, bevor typisierte Sitzungs-/Konversationsspalten
  angewendet werden, sodass veraltete JSON-Nutzdaten keine Zustellungsziele wiederherstellen können.
  Das Ankündigungs-Routing von Subagenten erfordert ebenfalls den typisierten SQLite-Zustellungskontext;
  es greift nicht mehr auf kompatibilitätsbezogene `SessionEntry`-Routenfelder zurück.
  Die explizite Zustellungsvererbung von Gateway-`chat.send` liest den typisierten SQLite-
  Zustellungskontext statt der kompatibilitätsbezogenen Felder `origin`/`last*`.
  `tools.effective` leitet den Provider-/Konto-/Thread-Kontext ebenfalls aus typisierten
  SQLite-Zustellungs-/Routingzeilen ab, nicht aus veralteten Schatten von `last*`-Sitzungseinträgen.
  Der Prompt-Kontext für Systemereignisse erstellt Kanal-/Ziel-/Konto-/Thread-Felder aus
  typisierten Zustellungsfeldern statt aus `origin`-Schatten neu.
  Der gemeinsame `deliveryContextFromSession`-Helper und die Zuordnung von Sitzungen zu Konversationen
  ignorieren `SessionEntry.origin` jetzt vollständig; nur typisierte Zustellungsfelder
  und relationale Konversationszeilen können eine aktive Routenidentität erzeugen.
  Die Normalisierung von Laufzeitsitzungseinträgen entfernt `origin`, bevor
  `entry_json` gespeichert oder projiziert wird, und eingehende Metadaten schreiben typisierte Kanal-/Chat-
  Felder sowie relationale Konversationszeilen, statt neue Ursprungsschatten
  zu erzeugen.
- Transkriptereignisse, Transkript-Snapshots und Trajektorien-Laufzeitereignisse
  verweisen jetzt auf die maßgebliche agentenspezifische `sessions`-Stammtabelle und werden beim Löschen der Sitzung
  kaskadiert. Zeilen für Transkriptidentität/Idempotenz werden weiterhin aus der
  exakten Transkriptereigniszeile kaskadiert.
- Memory-Core-Indizes verwenden jetzt die expliziten Agentendatenbanktabellen
  `memory_index_meta`, `memory_index_sources`, `memory_index_chunks` und
  `memory_embedding_cache`, wobei `memory_index_state` Revisionsänderungen verfolgt.
  Optionale FTS-/Vektor-Nebenindizes heißen `memory_index_chunks_fts` und
  `memory_index_chunks_vec` statt der generischen Tabellen `meta`, `files`, `chunks`,
  `chunks_fts` oder `chunks_vec`. Die maßgeblichen Namen behalten die aktuelle
  Pfad-/Quellenzeilenstruktur und die Kompatibilität serialisierter Einbettungen bei. Diese Tabellen
  sind abgeleitete Such-Caches und kein maßgeblicher Transkriptspeicher; sie können
  gelöscht und aus Speicherarbeitsbereichsdateien und konfigurierten Quellen neu erstellt werden.
  Beim Öffnen eines ausgelieferten Speicherindex mit generischen Namen werden dessen Metadaten, Quellen,
  Segmente und Einbettungs-Cache in die maßgeblichen Tabellen migriert; abgeleitete FTS-/Vektor-
  Tabellen werden unter ihren maßgeblichen Namen neu erstellt.
- Der Wiederherstellungszustand von Subagent-Ausführungen befindet sich jetzt in typisierten gemeinsamen `subagent_runs`-Zeilen
  mit indizierten Schlüsseln für untergeordnete, anfordernde und steuernde Sitzungen. Die alte
  `subagents/runs.json`-Datei dient nur noch als Eingabe für die Doctor-Bereinigung. Ihre Ausführungseinträge sind
  flüchtiger Wiederherstellungszustand, daher erfasst Doctor den Ausmusterungsnachweis und
  verwirft die Datei, ohne sie zu importieren. Da anhand einer Datei nicht feststellbar ist, ob
  ihre Einträge aktiv oder veraltet sind, nachdem SQLite-Zeilen bereinigt wurden, müssen
  Betreiber aktive Ausführungen aus der Dateiära abschließen lassen, bevor sie diese Versionsgrenze überschreitend aktualisieren.
- Aktuelle Konversationsbindungen befinden sich jetzt in typisierten gemeinsamen
  `current_conversation_bindings`-Zeilen mit der normalisierten Konversations-ID als Schlüssel sowie
  Zielagenten-/Sitzungsspalten, Konversationsart, Status, Ablauf und Metadaten,
  die als relationale Spalten statt als doppelter undurchsichtiger Bindungsdatensatz gespeichert werden.
  Der dauerhafte Bindungsschlüssel umfasst die normalisierte Konversationsart, sodass
  Direkt-/Gruppen-/Kanalreferenzen nicht kollidieren können, und SQLite lehnt ungültige Werte für Bindungsart
  und -status ab. Die alte
  `bindings/current-conversations.json`-Datei dient nur noch als Eingabe für die Doctor-Migration.
- Die Wiederherstellung der Zustellungswarteschlange überlagert die Wiedergabe-JSON jetzt mit typisierten Warteschlangenspalten für Kanal, Ziel,
  Konto, Sitzung, Wiederholung, Fehler, Plattformversand und Wiederherstellungszustand.
  `entry_json` enthält weiterhin die Wiedergabenutzdaten, Hooks und Formatierungsnutzdaten,
  aber typisierte Spalten sind für das aktive Warteschlangen-Routing und den Warteschlangenstatus maßgeblich.
- Die TUI-Zeiger zur Wiederherstellung der letzten Sitzung befinden sich jetzt in typisierten gemeinsamen
  `tui_last_sessions`-Zeilen mit dem gehashten TUI-Verbindungs-/Sitzungsbereich als Schlüssel.
  Die Laufzeit liest und schreibt ausschließlich SQLite, führt für jeden Bereich atomare Upserts durch und
  schließt Heartbeat-Sitzungen aus. `openclaw doctor --fix` validiert die
  alte TUI-JSON-Datei strikt, behält neuere SQLite-Zeilen bei, überprüft das maßgebliche Ergebnis
  und entfernt die unveränderte Legacy-Datei, statt ein Archiv zurückzulassen.
- Hashes für die Bereitstellung von Discord-Befehlen befinden sich jetzt im gemeinsamen SQLite-
  Speicher für den Plugin-Zustand. Die Laufzeit liest und schreibt ausschließlich exakte anwendungsbezogene Schlüssel. Doctor
  löscht die wiederherstellbare Legacy-Datei `discord/command-deploy-cache.json`,
  ohne sie zu importieren, sodass beim nächsten Start ein einmaliger maßgeblicher Abgleich erfolgt.
- Standardmäßige TTS-Einstellungen befinden sich jetzt in gemeinsamen SQLite-Zeilen für den Plugin-Zustand mit Schlüsseln unter dem
  Plugin `speech-core`. Die alte Datei `settings/tts.json` dient nur noch als Eingabe für die Doctor-Migration;
  die Laufzeit liest oder schreibt keine JSON-Dateien für TTS-Einstellungen mehr, und die
  Legacy-Pfadauflösung befindet sich im Doctor-Migrationsmodul.
- Metadaten für Geheimnisziele sprechen jetzt von Speichern, statt vorzugeben, jedes
  Anmeldedatenziel sei eine Konfigurationsdatei. `openclaw.json` bleibt der Konfigurationsspeicher;
  Authentifizierungsprofilziele verwenden typisierte SQLite-`auth_profile_stores`-Zeilen, wobei
  Provider-spezifische Anmeldedaten als JSON-Nutzdaten gespeichert werden.
- Die Geheimnisprüfung durchsucht keine ausgemusterten agentenspezifischen `auth.json`-Dateien mehr. Doctor ist für
  Warnung, Import und Entfernung dieser Legacy-Datei zuständig.
- Legacy-Pfad-Helper für Authentifizierungsprofile befinden sich jetzt im Legacy-Code von Doctor. Die zentralen Pfad-
  Helper für Authentifizierungsprofile stellen die Identität und Anzeigeorte des SQLite-Authentifizierungsspeichers bereit,
  nicht die Laufzeitpfade `auth-profiles.json` oder `auth-state.json`.
- Laufzeitmodule für die Wiederherstellung von Subagent-Ausführungen und den OpenRouter-Modellfähigkeits-Cache
  halten SQLite-Snapshot-Lese-/Schreibfunktionen jetzt von ausschließlich für Doctor vorgesehenen Legacy-JSON-
  Import-Helpern getrennt. OpenRouter-Modellfähigkeiten verwenden die typisierten generischen
  `model_capability_cache`-Zeilen unter `provider_id = "openrouter"` statt
  eines einzelnen undurchsichtigen Cache-Blobs oder einer Provider-spezifischen Host-Tabelle. Der `taskName`-Wert
  einer Subagent-Ausführung wird in der typisierten Spalte `subagent_runs.task_name` gespeichert; die
  Kopie `payload_json` enthält Wiedergabe-/Debugdaten und ist nicht die Quelle für häufig verwendete Anzeige- oder
  Suchfelder.
- `src/agents/filesystem/virtual-agent-fs.sqlite.ts` implementiert ein SQLite-VFS
  über der Agentendatenbanktabelle `vfs_entries`. Verzeichnislesevorgänge, rekursive
  Exporte, Löschvorgänge und Umbenennungen verwenden indizierte `(namespace, path)`-Präfixbereiche,
  statt einen gesamten Namensraum zu durchsuchen oder sich auf die `LIKE`-Pfadübereinstimmung zu verlassen.
- `src/agents/runtime-worker.entry.ts` erstellt für jeden Lauf SQLite-VFS-, Tool-Artefakt-, Laufartefakt- und bereichsgebundene Cache-Speicher für Worker.
- Der Abschluss des Workspace-Bootstrappings, die Aktualität der Attestierung und die generierten Bootstrap-Hashes befinden sich jetzt in typisierten gemeinsamen `workspace_setup_state`-, `workspace_path_aliases`-, `workspace_attestations`- und `workspace_generated_bootstrap_hashes`-Zeilen mit der kanonischen Workspace-Identität als Schlüssel. Persistierte lexikalische Aliase und Aliase für reale Pfade sorgen dafür, dass der Schutz für verschwundene Workspaces stabil bleibt, nachdem ein konfigurierter symbolischer Link verschwindet; neu ausgerichtete Aliase schlagen nach dem Fail-Closed-Prinzip fehl. Die Runtime liest oder schreibt nicht mehr `openclaw-workspace-state.json`, `.openclaw/workspace-state.json`, `workspace-attestations/*.attested` im Zustandsverzeichnis oder benachbarte `<workspace>.attested`-Sidecars. `openclaw doctor --fix` validiert und beansprucht Legacy-Quellen, importiert sie mit Migrationsbelegen in SQLite, überprüft die kanonischen Zeilen und entfernt erst danach die beanspruchten Dateien.
- Das gemeinsame Schema reserviert eine `exec_approvals_config`-Singleton-Zeile, die Runtime-Umstellung steht jedoch noch aus. TypeScript und die macOS-Begleitanwendung verwenden weiterhin die zustandsbezogene JSON-Datei und müssen gemeinsam zu SQLite migriert werden.
- Die TypeScript-Geräteidentität verwendet jetzt typisierte `device_identities`-Zeilen, wobei der ausschließlich dem Doctor vorbehaltene Import von Legacy-JSON außerhalb des Runtime-Owners verbleibt. Die Geräteauthentifizierung bleibt bis zu einer koordinierten Schema- und Runtime-übergreifenden Migration dateibasiert; `device_auth_tokens` bleibt für diese Nachfolgearbeit reserviert.
- Der Cache für den GitHub-Copilot-Tokenaustausch verwendet die gemeinsame SQLite-Tabelle für den Plugin-Zustand unter `github-copilot/token-cache/default`. Es handelt sich um einen Provider-eigenen Cache-Zustand, daher wird bewusst keine Host-Schematabelle hinzugefügt.
- Die GitHub-Copilot-Compaction schreibt keine `openclaw-compaction-*.json`-Workspace-Sidecars mehr. Der Harness ruft den SDK-RPC zur Verlaufs-Compaction für die nachverfolgte SDK-Sitzung auf, und OpenClaw speichert dauerhafte Sitzungs- und Transkriptzustände in SQLite statt in Kompatibilitäts-Markierungsdateien.
- Die gemeinsame Swift-Runtime (`OpenClawKit`) verwendet dieselbe `state/openclaw.sqlite#table/device_identities`-Form und dieselben Zeilenschlüssel für die Geräteidentität. Legacy-Dateien aus Apple-Containern werden vom Swift-Migrations-Owner importiert, da der TypeScript-Doctor nicht auf diese Container zugreifen kann. Die Swift-Geräteauthentifizierung bleibt bis zur koordinierten Authentifizierungs-Nachfolgearbeit dateibasiert.
- Die Android-Geräteidentität und die zwischengespeicherte Geräteauthentifizierung bleiben anwendungsinterne Speicher. Sie erfordern eine separate, Android-eigene Migration; die Host-SQLite-Ansprüche beschreiben nicht das aktuelle Android-Verhalten.
- Der Verlauf der zuletzt verwendeten Pakete für Android-Benachrichtigungen verwendet typisierte `android_notification_recent_packages`-Zeilen. Die Runtime migriert oder liest die alten SharedPreferences-CSV-Schlüssel nicht mehr.
- Die Erstellung der Geräteidentität schlägt nach dem Fail-Closed-Prinzip fehl, wenn die Legacy-Datei `identity/device.json` vorhanden ist, wenn die SQLite-Identitätszeile ungültig ist oder wenn der SQLite-Identitätsspeicher nicht geöffnet werden kann. Der Doctor importiert und entfernt diese Datei zuerst, sodass der Runtime-Start die Kopplungsidentität vor der Migration nicht unbemerkt wechseln kann.
- Die Auswahl der Geräteidentität ist ein SQLite-Zeilenschlüssel und kein JSON-Dateispeicherort. Tests und Gateway-Hilfsfunktionen übergeben explizite Identitätsschlüssel; nur die Doctor-Migration und die Fail-Closed-Startprüfung kennen den außer Betrieb genommenen Dateinamen `identity/device.json`.
- Die Kompatibilität für das Zurücksetzen von Sitzungen befindet sich jetzt in der Doctor-Konfigurationsmigration: `session.idleMinutes` wird nach `session.reset.idleMinutes` verschoben, `session.resetByType.dm` wird nach `session.resetByType.direct` verschoben, und die Runtime-Richtlinie zum Zurücksetzen liest nur kanonische Zurücksetzungsschlüssel.
- Die Legacy-Konfigurationskompatibilität befindet sich jetzt unter `src/commands/doctor/`. Die normale `readConfigFileSnapshot()`-Validierung importiert weder Legacy-Detektoren des Doctors noch versieht sie Legacy-Probleme mit Anmerkungen; `runDoctorConfigPreflight()` fügt diese Probleme für die Reparatur und Berichterstattung durch den Doctor hinzu. Der Doctor-Konfigurationsablauf importiert `src/commands/doctor/legacy-config.ts`, und die Reparatur alter OAuth-Profil-IDs befindet sich unter `src/commands/doctor/legacy/oauth-profile-ids.ts`.
- Befehle außerhalb des Doctors führen die Reparatur von Legacy-Konfigurationen nicht automatisch aus. Beispielsweise schlägt `openclaw update --channel` jetzt bei einer ungültigen Legacy-Konfiguration fehl und fordert den Benutzer auf, den Doctor auszuführen, statt unbemerkt Doctor-Migrationscode zu importieren.
- Web-Push, APNs, Voice Wake, Aktualisierungsprüfungen und der Konfigurationszustand verwenden jetzt typisierte gemeinsame SQLite-Tabellen für Abonnements, VAPID-Schlüssel, Node-Registrierungen, Triggerzeilen, Routingzeilen, den Zustand von Aktualisierungsbenachrichtigungen und Einträge zum Konfigurationszustand statt vollständiger undurchsichtiger JSON-Blobs. Schreibvorgänge von Web-Push und APNs führen nur für die betroffene Primärschlüsselzeile ein Upsert aus; der Konfigurationszustand wird anhand des Konfigurationspfads abgeglichen. Ihre Runtime-Module bleiben von den ausschließlich dem Doctor vorbehaltenen Hilfsfunktionen für den Import von Legacy-JSON getrennt.
- Die APNs-Runtime liest und schreibt ausschließlich `apns_registrations`. Das explizite `openclaw doctor --fix` importiert die außer Betrieb genommene Datei `push/apns-registrations.json` strikt, bewahrt bestehende kanonische Zeilen, überprüft die Transaktion, zeichnet einen Beleg auf und entfernt die geheimnishaltige JSON-Datei. Beleggestützte Wiederholungsversuche führen nur die Bereinigung durch, während `apns_registration_tombstones` Invalidierungen vor der ersten Reparatur abdeckt, sodass veraltete Relay-Berechtigungen oder Geräte-Token nicht wiederhergestellt werden können.
- Die Node-Host-Konfiguration verwendet jetzt eine typisierte Singleton-Zeile in der gemeinsamen SQLite-Datenbank. Die Runtime schlägt nach dem Fail-Closed-Prinzip fehl, solange die alte Datei `node.json` oder ein unterbrochener Beanspruchungsvorgang bestehen bleibt; das explizite `openclaw doctor --fix` importiert und entfernt sie strikt vor der normalen Runtime-Nutzung.
- Geräte-/Node-Kopplung, Kanalkopplung, Kanal-Zulassungslisten und Bootstrap-Zustand verwenden jetzt typisierte SQLite-Zeilen statt vollständiger undurchsichtiger JSON-Blobs. Genehmigungen für Plugin-Bindungen und der Zustand von Cron-Aufträgen folgen derselben Aufteilung: Runtime-Module stellen SQLite-gestützte Operationen und neutrale Snapshot-Hilfsfunktionen bereit, und Snapshot-Schreibvorgänge für Kopplung/Bootstrap sowie Genehmigungen von Plugin-Bindungen gleichen Zeilen anhand des Primärschlüssels ab, statt Tabellen zu leeren, während der Doctor die alten JSON-Dateien über `src/commands/doctor/legacy/*`-Module importiert und entfernt.
- Datensätze installierter Plugins befinden sich jetzt im SQLite-Index installierter Plugins. Das Lesen und Schreiben der Runtime-Konfiguration migriert oder bewahrt alte `plugins.installs`-Daten der erstellten Konfiguration nicht mehr; der Doctor importiert diese Legacy-Konfigurationsform vor der normalen Runtime-Nutzung in SQLite.
- Snapshots zur Wiederherstellung von QQBot-Anmeldedaten befinden sich jetzt im SQLite-Plugin-Zustand unter `qqbot/credential-backups`. Die Runtime schreibt `qqbot/data/credential-backup*.json` nicht mehr; der QQBot-Doctor-Vertrag importiert und archiviert diese Legacy-Sicherungsdateien aus dem aktiven Zustandsverzeichnis.
- Die Gateway-Neuladeplanung vergleicht Snapshots des SQLite-Indexes installierter Plugins unter einem internen `installedPluginIndex.installRecords.*`-Diff-Namensraum. Runtime-Neuladeentscheidungen verpacken diese Zeilen nicht mehr in vorgetäuschte `plugins.installs`-Konfigurationsobjekte.
- Die Anmeldedaten von Matrix-Konten befinden sich jetzt im SQLite-Plugin-Zustand. Die Runtime liest nur aus diesem kanonischen Speicher; der Doctor importiert, überprüft und archiviert außer Betrieb genommene `credentials/matrix/credentials*.json`-Dateien, wenn das zugehörige Konto aufgelöst werden kann.
- Die zentralen Runtime-Module für Kopplung und Cron verwenden keine Legacy-JSON-Pfadgeneratoren mehr. Die veraltete SDK-Hilfsfunktion für Kopplungspfade bleibt als ausschließlich für Migrationen bestimmte Kompatibilitätsfunktion erhalten; die Doctor-Zustandsmigration übernimmt ihre Dateilese- und Importvorgänge. Doctor-eigene Legacy-Module erstellen die Quellpfade `pending.json`, `paired.json`, `bootstrap.json` und `cron/jobs.json` ausschließlich für Importtests und Migrationen. Die Normalisierung von Legacy-Formen für Cron-Aufträge und der JSONL-Verlaufsimport befinden sich unter `src/commands/doctor/cron/`; die Finalisierung des Legacy-SQLite-Verlaufs wird beim Öffnen der Zustandsdatenbank ausgeführt.
- `src/commands/doctor/legacy/runtime-state.ts` importiert Legacy-JSON-Zustandsdateien, einschließlich der Node-Host-Konfiguration, über den Doctor in SQLite. Neue Importfunktionen für Legacy-Dateien verbleiben unter `src/commands/doctor/legacy/`.
- `src/commands/doctor/state-migrations.ts` importiert Legacy-Transkripte aus `sessions.json` und `*.jsonl` direkt in SQLite und entfernt erfolgreich importierte Quellen. Root-Legacy-Transkripte werden nicht mehr über `agents/<agentId>/sessions/*.jsonl` zwischengelagert, und vor dem Import wird kein kanonisches JSONL-Ziel mehr erstellt.
- Doctor-Prüfungen der Zustandsintegrität durchsuchen keine Legacy-Sitzungsverzeichnisse mehr und bieten keine Löschung verwaister JSONL-Dateien mehr an. Legacy-Transkriptdateien sind ausschließlich Migrationseingaben, und der Migrationsschritt ist für den Import und die Entfernung der Quelle verantwortlich.
- Der Import der Legacy-Sandbox-Registry befindet sich unter `src/commands/doctor/legacy/sandbox-registry.ts`; aktive Lese- und Schreibvorgänge der Sandbox-Registry bleiben ausschließlich SQLite-basiert.
- Die Integritätsprüfung und Importreparatur für Legacy-Sitzungstranskripte befindet sich unter `src/commands/doctor/legacy/session-transcript-health.ts`; Runtime-Befehlsmodule enthalten keine JSONL-Transkriptanalyse oder Reparatur aktiver Branches mehr.

Highlights der abgeschlossenen Konsolidierungen/Löschungen:

- Der Plugin-Zustand verwendet jetzt die gemeinsam genutzte Datenbank `state/openclaw.sqlite`. Der alte
  branch-lokale Sidecar-Importer `plugin-state/state.sqlite` wurde entfernt, weil
  dieses SQLite-Layout nie veröffentlicht wurde. Prüf-/Test-Helfer melden die gemeinsam genutzte
  `databasePath`, statt einen Plugin-Zustands-spezifischen SQLite-Pfad offenzulegen.
- Die Laufzeittabellen für Aufgaben und Task Flow befinden sich jetzt in der gemeinsam genutzten
  Datenbank `state/openclaw.sqlite` statt in `tasks/runs.sqlite` und
  `tasks/flows/registry.sqlite`; die alten Sidecar-Importer wurden aus demselben Grund
  des unveröffentlichten Layouts entfernt.
- `src/config/sessions/store.ts` benötigt `storePath` nicht mehr für eingehende
  Metadaten, Routenaktualisierungen oder Lesezugriffe auf den Aktualisierungszeitpunkt. Befehlspersistenz, Bereinigung von CLI-
  Sitzungen, Subagent-Tiefe, Authentifizierungsüberschreibungen und die Sitzungsidentität
  von Transkripten verwenden APIs für Agenten-/Sitzungszeilen. Schreibvorgänge werden als SQLite-Zeilen-Patches
  mit optimistischer Konfliktwiederholung angewendet.
- Die Auflösung von Sitzungszielen stellt jetzt Datenbankziele pro Agent bereit, keine veralteten
  `sessions.json`-Pfade. Das gemeinsam genutzte Gateway, ACP-Metadaten, die Reparatur von Doctor-Routen und
  `openclaw sessions` führen `agent_databases` sowie konfigurierte Agenten auf.
- Das Sitzungsrouting des Gateways verwendet jetzt `resolveGatewaySessionDatabaseTarget`; das
  zurückgegebene Ziel enthält `databasePath` und mögliche SQLite-Zeilenschlüssel
  statt eines veralteten Dateipfads zum Sitzungsspeicher.
- Die Laufzeittypen für Kanalsitzungen stellen jetzt `{agentId, sessionKey}` für
  Lesezugriffe auf den Aktualisierungszeitpunkt, eingehende Metadaten und Aktualisierungen der letzten Route bereit. Der alte
  Kompatibilitätstyp `saveSessionStore(storePath, store)` ist entfallen.
- Die Sitzungsoberflächen der Plugin-Laufzeit, Erweiterungs-API und des Plugin-SDK stellen jetzt
  SQLite-gestützte Helfer für Sitzungszeilen bereit statt Kompatibilitätshelfern
  für den gesamten Speicher bzw. Dateien aktiver Sitzungen. Kompatibilitätsexporte der Root-Bibliothek bleiben nur
  außerhalb des Plugin-SDK für veraltete interne Aufrufer und Migrationsaufrufer verfügbar. Der alte
  Helfer `resolveLegacySessionStorePath` ist entfallen; die veraltete Konstruktion von
  `sessions.json`-Pfaden erfolgt jetzt lokal in Migrations- und Test-Fixtures.
- `src/config/sessions/session-entries.sqlite.ts` speichert jetzt kanonische Sitzungseinträge
  in der Datenbank des jeweiligen Agenten und unterstützt Lesen, Upsert, Löschen und Patches
  auf Zeilenebene. Upsert-, Patch- und Löschvorgänge der Laufzeit suchen nicht mehr nach Varianten der Groß-/Kleinschreibung und
  bereinigen keine veralteten Alias-Schlüssel mehr; der Doctor ist für die Kanonisierung zuständig. Der
  eigenständige JSON-Importhelfer ist entfallen, und die Migration führt neuere Zeilen per Upsert zusammen,
  statt die gesamte Sitzungstabelle zu ersetzen. Öffentliche Lese-, Auflistungs- und Ladehelfer
  projizieren häufig verwendete Sitzungsmetadaten aus typisierten `sessions`- und `conversations`-Zeilen;
  `entry_json` ist eine Kompatibilitäts-/Debug-Schattenkopie und kann veraltet oder ungültig sein,
  ohne dass die typisierte Sitzungsidentität oder der Zustellungskontext verloren geht.
- `src/config/sessions/delivery-info.ts` löst den Zustellungskontext jetzt aus den
  typisierten agentenspezifischen Zeilen `sessions` + `conversations` + `session_conversations` auf.
  Die Laufzeit-Zustellungsidentität wird nicht mehr aus
  `session_entries.entry_json` rekonstruiert; eine fehlende typisierte Konversationszeile ist ein
  Migrations-/Reparaturproblem des Doctors, kein Laufzeit-Fallback.
- Entscheidungen zum Zurücksetzen gespeicherter Sitzungen bevorzugen jetzt typisierte Metadaten aus `sessions.session_scope`,
  `sessions.chat_type` und `sessions.channel`. Das Parsen von `sessionKey`
  bleibt nur für explizite Thread-/Themen-Suffixe an Befehlszielen bestehen; die Klassifizierung von Zurücksetzungen als Gruppe oder
  Direktverbindung wird nicht mehr aus der Schlüsselform abgeleitet.
- Die Anzeigeklassifizierung von Sitzungslisten und -status verwendet jetzt typisierte Chat-Metadaten und
  die Gateway-Sitzungsart. Teilzeichenfolgen `:group:` oder `:channel:`
  innerhalb von `session_key` werden nicht mehr als dauerhafte Aussage über Gruppe oder Direktverbindung behandelt.
- Die Auswahl der Richtlinie für stille Antworten verwendet jetzt ausschließlich den expliziten Konversationstyp oder
  Oberflächenmetadaten. Die Richtlinie für Direktverbindungen/Gruppen wird nicht mehr anhand von
  `session_key`-Teilzeichenfolgen erraten.
- Die Auflösung des Modells für die Sitzungsanzeige erhält die Agenten-ID jetzt aus dem Ziel der SQLite-
  Sitzungsdatenbank, statt sie aus `session_key` herauszutrennen.
- Die Hydrierung des Ankündigungsziels zwischen Agenten verwendet jetzt ausschließlich typisierte
  `sessions.list` `deliveryContext`. Das Routing für Kanal/Konto/Thread wird nicht mehr
  aus dem veralteten `origin`, gespiegelten `last*`-Feldern oder der Form von `session_key`
  wiederhergestellt.
- Die Ablehnung von Thread-Zielen durch `sessions_send` liest jetzt typisierte SQLite-Routing-
  Metadaten. Ziele werden nicht mehr durch das Parsen von Thread-Suffixen
  aus dem Zielschlüssel abgelehnt oder akzeptiert.
- Die Validierung gruppenbezogener Tool-Richtlinien liest jetzt typisiertes SQLite-Konversations-
  Routing für die aktuelle oder erzeugte Sitzung. Sie vertraut der Gruppen-/Kanalidentität nicht mehr,
  indem `sessionKey` dekodiert wird; vom Aufrufer bereitgestellte Gruppen-IDs werden verworfen, wenn
  keine typisierte Sitzungszeile sie bestätigt.
- Der Abgleich von Kanalmodell-Überschreibungen verwendet jetzt explizite Gruppen- und übergeordnete
  Konversationsmetadaten. Übergeordnete Konversations-IDs werden nicht mehr aus
  `parentSessionKey` dekodiert.
- Die Vererbung gespeicherter Modellüberschreibungen erfordert jetzt einen expliziten Schlüssel der übergeordneten Sitzung
  aus dem typisierten Sitzungskontext. Übergeordnete Überschreibungen werden nicht mehr aus
  `:thread:`- oder `:topic:`-Suffixen in `sessionKey` abgeleitet.
- Der alte Wrapper für Sitzungs-Thread-Informationen und der Thread-Parser für geladene Plugins sind entfallen;
  kein Laufzeitcode importiert `config/sessions/thread-info`.
- Der Helfer für Kanalkonversationen stellt keine Parsing-
  Brücken für vollständige Sitzungsschlüssel mehr bereit. Der Core normalisiert weiterhin Provider-eigene rohe Konversations-IDs über
  `resolveSessionConversation(...)`, rekonstruiert daraus jedoch keine Routendaten
  aus `sessionKey`.
- Abschlusszustellung, Senderichtlinie und Aufgabenwartung leiten den Chat-
  Typ nicht mehr aus der Form von `session_key` ab. Der alte Parser für Chattyp-Schlüssel wurde gelöscht;
  diese Pfade erfordern typisierte Sitzungsmetadaten, einen typisierten Zustellungskontext oder
  explizites Zustellungsziel-Vokabular.
- Sitzungsliste/-status, Diagnose, Kontobindung für Genehmigungen, TUI-Heartbeat-
  Filterung und Nutzungszusammenfassungen durchsuchen `SessionEntry.origin` nicht mehr nach
  Provider-/Konto-/Thread-/Anzeige-Routing. Die einzigen verbleibenden Laufzeit-
  Lesezugriffe auf `origin` betreffen sitzungsfremde Konzepte oder Zustellungsobjekte des aktuellen Turns.
- Die native Konversationssuche für Genehmigungsanfragen liest jetzt typisierte agentenspezifische Sitzungs-
  Routingzeilen. Die Identität von Kanal-/Gruppen-/Thread-Konversationen wird nicht mehr
  aus `sessionKey` geparst; fehlende typisierte Metadaten sind ein Migrations-/Reparaturproblem.
- Die Ereignis-Payloads des Gateways für Sitzungsänderungen, Chats und Sitzungen geben keine
  `SessionEntry.origin`- oder `last*`-Routenschatten mehr zurück; Clients erhalten typisierte
  `channel`, `chatType` und `deliveryContext`.
- Die Auflösung der Heartbeat-Zustellung kann jetzt direkt die typisierte SQLite-
  `deliveryContext` empfangen, und die Heartbeat-Laufzeit übergibt die agentenspezifische
  Sitzungszustellungszeile, statt sich für das aktuelle Routing auf Kompatibilitäts-
  Schatten `session_entries` zu verlassen.
- Die Auflösung des Zustellungsziels isolierter Cron-Agenten hydriert ihre aktuelle
  Route ebenfalls aus der typisierten agentenspezifischen Sitzungszustellungszeile, bevor sie auf den
  Kompatibilitätseintrags-Payload zurückfällt.
- Die Auflösung des Ursprungs von Subagent-Ankündigungen reicht jetzt den typisierten Zustellungskontext
  der anfragenden Sitzung durch `loadRequesterSessionEntry` weiter und bevorzugt diese Zeile gegenüber
  den Kompatibilitätsschatten `last*`/`deliveryContext`.
- Aktualisierungen eingehender Sitzungsmetadaten werden jetzt zuerst mit der typisierten agentenspezifischen
  Zustellungszeile zusammengeführt; alte Zustellungsfelder von `SessionEntry` dienen nur als Fallback,
  wenn keine typisierte Konversationszeile vorhanden ist.
- Die Extraktion der Zustellung für Neustart/Aktualisierung lässt jetzt die typisierte SQLite-Zustellung
  `threadId` gegenüber Themen-/Thread-Fragmenten aus `sessionKey` Vorrang haben; das Parsen
  dient nur als Fallback für veraltete Schlüssel in Thread-Form.
- Kanal-IDs des Agentenkontexts von Hooks bevorzugen jetzt die typisierte SQLite-Konversationsidentität
  und danach explizite Nachrichtenmetadaten. Provider-/Gruppen-/Kanal-
  Fragmente werden nicht mehr aus `sessionKey` geparst.
- Die Vererbung externer Routen durch Gateway `chat.send` liest jetzt typisierte SQLite-Sitzungs-
  Routingmetadaten, statt Kanal-/Direkt-/Gruppenumfang aus Teilen von
  `sessionKey` abzuleiten. Kanalbezogene Sitzungen erben nur, wenn der typisierte
  Sitzungskanal und der Chattyp mit dem gespeicherten Zustellungskontext übereinstimmen; gemeinsam genutzte Haupt-
  sitzungen behalten ihre strengere Regel für CLI/fehlende Client-Metadaten bei.
- Das Aufwecken durch Neustart-Sentinels und das Routing von Fortsetzungen liest jetzt typisierte SQLite-
  Zustellungs-/Routingzeilen, bevor Heartbeat-Aufweckvorgänge oder geroutete Fortsetzungen von Agentenläufen
  in die Warteschlange gestellt werden. Der Zustellungskontext wird nicht mehr aus der
  JSON-Schattenkopie des Sitzungseintrags rekonstruiert.
- Die Kontextauflösung von Gateway `tools.effective` liest jetzt typisierte SQLite-
  Zustellungs-/Routingzeilen für Provider-, Konto-, Ziel-, Thread- und Antwortmodus-
  Eingaben. Diese häufig verwendeten Routingfelder werden nicht mehr aus veralteten
  Ursprungsschatten von `session_entries.entry_json` wiederhergestellt.
- Das Routing für Sprachkonsultationen in Echtzeit löst die Zustellung für übergeordnete Sitzung/Anruf jetzt aus typisierten
  agentenspezifischen SQLite-Sitzungszeilen auf. Bei der Auswahl der Route für die eingebettete Agenten-
  nachricht wird nicht mehr auf Kompatibilitätsschatten von `SessionEntry.deliveryContext`
  zurückgegriffen.
- Das Heartbeat-Relay für ACP-Erzeugung und das Routing des übergeordneten Streams lesen die übergeordnete Zustellung jetzt
  aus typisierten SQLite-Sitzungszeilen. Der Zustellungskontext der übergeordneten Sitzung wird nicht mehr
  aus Kompatibilitätsschatten von Sitzungseinträgen rekonstruiert.
- Die Beibehaltung der Sitzungszustellungsroute folgt jetzt typisierten Chat-Metadaten und
  persistierten Zustellungsspalten. Kanalhinweise, Direkt-/Haupt-
  Markierungen oder die Thread-Form werden nicht mehr aus `sessionKey` extrahiert; interne Webchat-Routen
  erben ein externes Ziel nur, wenn SQLite bereits über eine typisierte/persistierte Zustellungs-
  identität für die Sitzung verfügt.
- Die generische Extraktion der Sitzungszustellung liest nur noch die exakte typisierte SQLite-
  Sitzungszustellungszeile. Thread-/Themen-Suffixe werden nicht mehr geparst, und es erfolgt kein Fallback
  von einem threadförmigen Schlüssel auf einen Basissitzungsschlüssel.
- Antwortversand, Wiederherstellung durch Neustart-Sentinels und Routing von Sprachkonsultationen in Echtzeit
  verwenden jetzt exakte typisierte SQLite-Sitzungs-/Konversationszeilen für das Thread-Routing. Sie
  stellen Thread-IDs oder den Zustellungskontext der Basissitzung nicht mehr durch Parsen
  threadförmiger Sitzungsschlüssel wieder her.
- Die Begrenzung des eingebetteten PI-Verlaufs verwendet jetzt die typisierte SQLite-Sitzungsrouting-
  Projektion (`sessions` + primäre `conversations`) für Provider, Chattyp
  und Peer-Identität. Provider-, DM-, Gruppen- oder Thread-Form werden nicht mehr
  aus `sessionKey` geparst.
- Die Ableitung der Cron-Tool-Zustellung verwendet jetzt nur noch eine explizite Zustellung oder den aktuellen typisierten
  Zustellungskontext. Kanal-, Peer-, Konto- oder Thread-
  Ziele werden nicht mehr aus `agentSessionKey` dekodiert.
- Laufzeit-Sitzungszeilen enthalten den alten Routenalias `lastProvider` nicht mehr.
  Helfer und Tests verwenden typisierte Felder `lastChannel` und `deliveryContext`;
  nur die Doctor-Migration sollte ältere Routenaliase oder persistierte
  Schatten von `origin` übersetzen.
- Transkriptereignisse, VFS-Zeilen und Tool-Artefaktzeilen werden jetzt in die agentenspezifische
  Datenbank geschrieben. Die unveröffentlichte globale Zuordnungstabelle für Transkriptdateien ist entfallen; der Doctor
  zeichnet veraltete Quellpfade stattdessen in dauerhaften Migrationszeilen auf.
- Die Laufzeitsuche nach Transkripten durchsucht keine JSONL-Byte-Offsets und prüft keine veralteten
  Transkriptdateien mehr. Gateway-Pfade für Chat/Medien/Verlauf lesen Transkriptzeilen aus
  SQLite; Sitzungs-JSONL dient jetzt nur als veraltete Eingabe für den Doctor, nicht als Laufzeitzustand
  oder Exportformat.
- Übergeordnete und Verzweigungsbeziehungen von Transkripten verwenden strukturierte
  `parentTranscriptScope: {agentId, sessionId}`-Metadaten in SQLite-Transkript-
  Headern, keine pfadähnlichen `agent-db:...transcript_events...`-Locator-Zeichenfolgen.
- Der Vertrag des Transkriptmanagers stellt keine implizit persistierten
  Konstruktoren `create(cwd)` oder `continueRecent(cwd)` mehr bereit. Persistierte Transkript-
  manager werden mit einem expliziten `{agentId, sessionId}`-Gültigkeitsbereich geöffnet; nur
  In-Memory-Manager bleiben für Tests und reine Transkripttransformationen frei von Gültigkeitsbereichen.
- APIs des Runtime-Transkriptspeichers lösen SQLite-Gültigkeitsbereiche auf, keine Dateisystempfade. Der
  alte `resolve...ForPath`-Helper und die nicht verwendeten `transcriptPath`-Schreiboptionen sind
  aus den Runtime-Aufrufern entfernt.
- Die Runtime-Sitzungsauflösung verwendet jetzt `{agentId, sessionId}` und darf keine
  `sqlite-transcript://<agent>/<session>`-Zeichenfolgen für externe Grenzen ableiten.
  Veraltete absolute JSONL-Pfade dienen ausschließlich als Eingaben für die Doctor-Migration.
- Direkt-Bridge-Datensätze der nativen Hook-Weiterleitung befinden sich jetzt in typisierten gemeinsamen
  `native_hook_relay_bridges`-Zeilen, deren Schlüssel die Relay-ID ist. Die Runtime schreibt für diese kurzlebigen Bridge-
  Datensätze weder eine `/tmp`-JSON-Registry noch undurchsichtige generische Datensätze mehr.
- `runEmbeddedPiAgent(...)` besitzt keinen Transkript-Locator-Parameter mehr.
  Vorbereitete Worker-Deskriptoren lassen Transkript-Locators ebenfalls weg. Runtime-Sitzungs-
  zustand und in die Warteschlange eingereihte Folgeläufe führen `{agentId, sessionId}` statt
  abgeleiteter Transkript-Handles mit.
- Eingebettete Compaction erhält den SQLite-Gültigkeitsbereich jetzt aus `agentId` und `sessionId`.
  Compaction-Hooks, Kontext-Engine-Aufrufe, CLI-Delegierung und Protokollantworten
  dürfen keine abgeleiteten `sqlite-transcript://...`-Handles erhalten. Export-/Debug-Code
  kann explizite Benutzerartefakte aus Zeilen materialisieren, stellt jedoch keinen
  generischen JSONL-Exportpfad für Sitzungen bereit und speist keine Dateinamen zurück in die Runtime-
  Identität.
- `/export-session` liest Transkriptzeilen aus SQLite und schreibt ausschließlich die angeforderte
  eigenständige HTML-Ansicht. Der eingebettete Viewer rekonstruiert oder
  lädt Sitzungs-JSONL nicht mehr aus diesen Zeilen herunter.
- Die Kontext-Engine-Delegierung analysiert keinen Transkript-Locator mehr, um die
  Agentenidentität wiederherzustellen. Der vorbereitete Runtime-Kontext führt den aufgelösten `agentId`
  in den integrierten Compaction-Adapter mit.
- Transkriptumschreibung und Live-Kürzung von Tool-Ergebnissen lesen und speichern
  den Transkriptzustand jetzt anhand von `{agentId, sessionId}` und leiten keine temporären
  Locators für Ereignis-Payloads von Transkriptaktualisierungen ab.
- Die Helper-Oberfläche für den Transkriptzustand besitzt keine Locator-basierten
  Varianten `readTranscriptState`, `replaceTranscriptStateEvents` oder
  `persistTranscriptStateMutation` mehr. Runtime-Aufrufer müssen die
  `{agentId, sessionId}`-APIs verwenden. Der Doctor-Import liest veraltete Dateien anhand expliziter Datei-
  pfade und schreibt SQLite-Zeilen; Locator-Zeichenfolgen werden nicht migriert.
- Der Vertrag des Runtime-Sitzungsmanagers stellt `open(locator)`,
  `forkFrom(locator)` oder `setTranscriptLocator(...)` nicht mehr bereit. Persistierte Sitzungs-
  manager werden ausschließlich anhand von `{agentId, sessionId}` geöffnet; Auflistungs-/Fork-Helper befinden sich
  auf zeilenorientierten Sitzungs- und Checkpoint-APIs statt auf der Fassade des Transkript-
  managers.
- Gateway-APIs zum Lesen von Transkripten verwenden zuerst den Gültigkeitsbereich. Sie akzeptieren
  `{agentId, sessionId}` und keinen positionellen Transkript-Locator, der
  versehentlich zur Runtime-Identität werden könnte. Die Analyse aktiver Transkript-Locators
  ist entfernt; veraltete Quellpfade werden ausschließlich vom Doctor-Importcode gelesen.
- Ereignisse für Transkriptaktualisierungen verwenden ebenfalls zuerst den Gültigkeitsbereich. `emitSessionTranscriptUpdate`
  akzeptiert keine bloße Locator-Zeichenfolge mehr, und Listener leiten anhand von
  `{agentId, sessionId}` weiter, ohne ein Handle zu analysieren.
- Die Gateway-Übertragung von Sitzungsnachrichten löst Sitzungsschlüssel aus dem Agenten-/Sitzungs-
  gültigkeitsbereich auf, nicht aus einem Transkript-Locator. Der alte Resolver/Cache für die Zuordnung von Transkript-Locators zu Sitzungs-
  schlüsseln ist entfernt.
- SSE-Filter für den Gateway-Sitzungsverlauf filtern Live-Aktualisierungen nach Agenten-/Sitzungsgültigkeitsbereich. Sie
  kanonisieren keine möglichen Transkript-Locators, realen Pfade oder dateiförmigen
  Transkriptidentitäten mehr, um zu entscheiden, ob ein Stream eine Aktualisierung erhalten soll.
- Hooks für den Sitzungslebenszyklus leiten keine Transkript-Locators mehr auf
  `session_end` ab oder stellen sie dort bereit. Hook-Konsumenten erhalten `sessionId`, `sessionKey`, IDs
  der nächsten Sitzung und Agentenkontext; Transkriptdateien sind kein Bestandteil des Lebenszyklus-
  vertrags.
- Reset-Hooks leiten ebenfalls keine Transkript-Locators mehr ab oder stellen sie bereit. Der
  `before_reset`-Payload enthält wiederhergestellte SQLite-Nachrichten sowie den Reset-
  Grund, während die Sitzungsidentität im Hook-Kontext verbleibt.
- Der Reset des Agenten-Harness akzeptiert keinen Transkript-Locator mehr. Die Reset-Ausführung wird
  anhand von `sessionId`/`sessionKey` sowie dem Grund eingegrenzt.
- Sitzungstypen von Agentenerweiterungen stellen `transcriptLocator` nicht mehr bereit; Erweiterungen
  sollten Sitzungskontext und Runtime-APIs verwenden, statt auf eine
  dateiförmige Transkriptidentität zuzugreifen.
- Plugin-Compaction-Hooks stellen keine Transkript-Locators mehr bereit. Der Hook-Kontext
  enthält bereits die Sitzungsidentität, und Transkripte müssen über SQLite-
  APIs mit Gültigkeitsbereich gelesen werden, nicht über dateiförmige Handles.
- `before_agent_finalize`-Hooks stellen `transcriptPath` nicht mehr bereit, einschließlich
  Payloads nativer Hook-Weiterleitungen. Finalisierungs-Hooks verwenden ausschließlich den Sitzungskontext.
- Gateway-Reset-Antworten erzeugen auf dem zurückgegebenen Eintrag keinen Transkript-Locator mehr
  künstlich. Der Reset erstellt SQLite-Transkriptzeilen, gibt den bereinigten
  Sitzungseintrag zurück und überlässt den Transkriptzugriff Readern mit Gültigkeitsbereich.
- Ergebnisse eingebetteter Läufe und Compaction-Ergebnisse stellen für die
  Sitzungsabrechnung keine Transkript-Locators mehr bereit. Automatische Compaction aktualisiert nur den aktiven `sessionId`,
  die Compaction-Zähler und Token-Metadaten.
- Ergebnisse eingebetteter Versuche geben `transcriptLocatorUsed` nicht mehr zurück, und
  `compact()`-Ergebnisse der Kontext-Engine geben keine Transkript-Locators mehr zurück.
  Runtime-Wiederholungsschleifen akzeptieren ausschließlich einen nachfolgenden `sessionId`.
- Ergebnisse des Anhängens an Delivery-Mirror-Transkripte geben keine Transkript-
  Locators mehr zurück. Aufrufer erhalten den angehängten `messageId`; Signale für Transkriptaktualisierungen verwenden
  den SQLite-Gültigkeitsbereich.
- Helper zum Forken übergeordneter Sitzungen geben ausschließlich den geforkten `sessionId` zurück. Die Vorbereitung von Subagenten
  übergibt den Engines den Gültigkeitsbereich des untergeordneten Agenten bzw. der untergeordneten Sitzung.
- Parameter des CLI-Runners und das erneute Einspeisen des Verlaufs akzeptieren keine Transkript-Locators mehr.
  CLI-Verlaufslesevorgänge lösen den SQLite-Transkriptgültigkeitsbereich aus `{agentId,
sessionId}` und dem Sitzungsschlüsselkontext auf.
- Test-Fixtures für CLI und eingebettete Runner speisen und lesen SQLite-Transkriptzeilen jetzt
  anhand der Sitzungs-ID ein bzw. aus, statt vorzugeben, aktive Sitzungen seien `*.jsonl`-Dateien, oder
  eine `sqlite-transcript://...`-Zeichenfolge über Runtime-Parameter weiterzureichen.
- Guard-Ereignisse für Tool-Ergebnisse einer Sitzung werden aus einem bekannten Sitzungsgültigkeitsbereich ausgegeben, selbst wenn ein
  In-Memory-Manager keinen abgeleiteten Locator besitzt. Die zugehörigen Tests simulieren keine aktiven
  `/tmp/*.jsonl`-Transkriptdateien mehr.
- BTW- und Compaction-Checkpoint-Helper lesen und forken Transkriptzeilen jetzt anhand des
  SQLite-Gültigkeitsbereichs. Checkpoint-Metadaten speichern jetzt ausschließlich Sitzungs-IDs sowie Blatt-/Eintrags-IDs;
  abgeleitete Locators werden nicht mehr in Checkpoint-Payloads geschrieben.
- Die Suche nach Gateway-Transkriptschlüsseln verwendet an Protokollgrenzen den SQLite-Transkriptgültigkeitsbereich
  und löst für Transkriptdateinamen weder reale Pfade auf noch ruft sie Dateistatusinformationen ab.
- Die automatische Transkriptrotation bei Compaction schreibt nachfolgende Transkriptzeilen
  direkt über den SQLite-Transkriptspeicher. Sitzungszeilen enthalten ausschließlich die
  nachfolgende Sitzungsidentität, keinen dauerhaften JSONL-Pfad oder persistierten Locator.
- Die Compaction der eingebetteten Kontext-Engine verwendet SQLite-benannte Helper zur Transkriptrotation.
  Die Rotationstests erstellen keine nachfolgenden JSONL-Pfade mehr und
  modellieren aktive Sitzungen nicht mehr als Dateien.
- Die verwaltete Aufbewahrung ausgehender Bilder erzeugt ihre Schlüssel für den Transkript-Nachrichten-Cache aus
  SQLite-Transkriptstatistiken statt durch Dateisystem-Statusaufrufe.
- Runtime-Sitzungssperren und der eigenständige veraltete `.jsonl.lock`-Doctor-
  Pfad wurden entfernt.
- Das Runtime-Barrel von Microsoft Teams und das öffentliche Plugin-SDK exportieren
  den alten Helper für Dateisperren nicht mehr erneut; dauerhafte Plugin-Zustandspfade werden durch SQLite gestützt.
- Das Bereinigen von Sitzungen nach Alter/Anzahl sowie die explizite Sitzungsbereinigung wurden entfernt.
  Doctor ist für den veralteten Import zuständig; überholte Sitzungen werden explizit zurückgesetzt oder gelöscht.
- Doctor-Integritätsprüfungen zählen eine veraltete JSONL-Datei nicht mehr als gültiges aktives
  Transkript für eine SQLite-Sitzungszeile. Der Zustand aktiver Transkripte basiert ausschließlich auf SQLite;
  veraltete JSONL-Dateien werden als Eingaben für Migration/Bereinigung verwaister Daten gemeldet.
- Doctor behandelt `agents/<agent>/sessions/` nicht mehr als erforderlichen Runtime-
  Zustand. Dieses Verzeichnis wird nur durchsucht, wenn es bereits vorhanden ist, und dient dann als Eingabe für den veralteten Import
  oder die Bereinigung verwaister Daten.
- Gateway-`sessions.resolve`, Pfade zum Patchen/Zurücksetzen/Komprimieren von Sitzungen, Erzeugen von Subagenten,
  schneller Abbruch, ACP-Metadaten, Heartbeat-isolierte Sitzungen und TUI-
  Patching migrieren oder bereinigen veraltete Sitzungsschlüssel nicht mehr als Nebeneffekt
  normaler Runtime-Arbeit.
- Die Auflösung von CLI-Befehlssitzungen gibt jetzt den zuständigen `agentId` statt eines
  `storePath` zurück und kopiert während der normalen Auflösung von
  `--to` oder `--session-id` keine veralteten Hauptsitzungszeilen mehr. Die Kanonisierung veralteter Hauptzeilen ist
  ausschließlich Aufgabe von Doctor.
- Die Runtime-Auflösung der Subagententiefe liest `sessions.json` oder JSON5-
  Sitzungsspeicher nicht mehr. Sie liest SQLite-`session_entries` anhand der Agenten-ID, und veraltete
  Tiefen-/Sitzungsmetadaten können ausschließlich über den Doctor-Importpfad übernommen werden.
- Sitzungsüberschreibungen für Authentifizierungsprofile werden durch direkte Upserts von
  `{agentId, sessionKey}`-Zeilen persistiert, statt verzögert eine dateiförmige Sitzungsspeicher-Runtime zu laden.
- Verbose-Steuerung für automatische Antworten und Helper für Sitzungsaktualisierungen lesen bzw. führen Upserts von SQLite-
  Sitzungszeilen jetzt anhand der Sitzungsidentität aus und benötigen keinen veralteten Speicherpfad mehr,
  bevor sie den persistierten Zeilenzustand bearbeiten.
- Helper für Sitzungsmetadaten von Befehlsläufen verwenden jetzt eintragsorientierte Namen und Modul-
  pfade; die alte `session-store`-Befehls-Helper-Oberfläche wurde entfernt.
- Das Einspeisen des Bootstrap-Headers und die Absicherung der manuellen Compaction-Grenze verändern
  SQLite-Transkriptzeilen jetzt direkt. Runtime-Aufrufer übergeben die Sitzungsidentität, keine
  beschreibbaren `.jsonl`-Pfade.
- Die stille Wiedergabe bei Sitzungsrotation kopiert die jüngsten Benutzer-/Assistenteninteraktionen anhand von
  `{agentId, sessionId}` aus SQLite-Transkriptzeilen. Sie akzeptiert keine
  Quell- oder Ziel-Transkript-Locators mehr.
- Neue Runtime-Sitzungszeilen speichern keine Transkript-Locators mehr. Aufrufer verwenden
  `{agentId, sessionId}` direkt; Export-/Debug-Befehle können beim Materialisieren von Zeilen
  Ausgabedateinamen auswählen.
- Beim Starten einer neuen persistierten Transkriptsitzung werden SQLite-Zeilen jetzt immer anhand des
  Gültigkeitsbereichs geöffnet. Der Sitzungsmanager verwendet weder einen früheren Transkriptpfad aus der Dateiära
  noch einen solchen Locator als Identität für die neue Sitzung wieder.
- Persistierte Transkriptsitzungen verwenden die explizite
  `openTranscriptSessionManagerForSession({agentId, sessionId})`-API. Die alten
  statischen `SessionManager.create/openForSession/list/forkFromSession`-Fassaden sind
  entfernt, sodass Tests und Runtime-Code die Sitzungserkennung aus der Dateiära nicht versehentlich
  wiederherstellen können.
- Die Plugin-Runtime stellt `api.runtime.agent.session.resolveTranscriptLocatorPath` nicht mehr bereit;
  Plugin-Code verwendet SQLite-Zeilen-Helper und Gültigkeitsbereichswerte.
- Die öffentliche `session-store-runtime`-SDK-Oberfläche exportiert jetzt ausschließlich Helper für Sitzungszeilen
  und Transkriptzeilen. Spezifische SQLite-Helper für Schema/Pfad/Transaktionen
  befinden sich in `sqlite-runtime`; rohe Helper zum Öffnen/Schließen/Zurücksetzen bleiben ausschließlich lokal für
  interne Tests.
- Veraltete `.jsonl`-Klassifikatoren für Trajektorien-/Checkpoint-Dateinamen befinden sich jetzt im
  Doctor-Modul für veraltete Sitzungsdateien. Die zentrale Sitzungsvalidierung importiert keine
  Helper für Dateiartefakte mehr, um normale SQLite-Sitzungs-IDs zu bestimmen.
- Blockierende Active-Memory-Subagentenläufe verwenden SQLite-Transkriptzeilen, statt
  temporäre oder persistierte `session.jsonl`-Dateien im Plugin-Zustand anzulegen. Die
  alte `transcriptDir`-Option wurde entfernt.
- Einmalige Slug-Erzeugung und Planerläufe des Systemagenten verwenden SQLite-Transkriptzeilen,
  statt temporäre `session.jsonl`-Dateien anzulegen.
- `llm-task`-Hilfsläufe und die Extraktion verborgener Commitments verwenden ebenfalls SQLite-Transkriptzeilen, sodass diese ausschließlich modellbezogenen Hilfssitzungen keine temporären JSON-/JSONL-Transkriptdateien mehr erstellen.
- `TranscriptSessionManager` ist jetzt nur noch ein geöffneter SQLite-Transkript-Scope.
  Der Runtime-Code öffnet ihn mit `openTranscriptSessionManagerForSession({agentId,
sessionId})`; Abläufe zum Erstellen, Verzweigen, Fortsetzen, Auflisten und Forken befinden sich in den jeweils zuständigen SQLite-Zeilen-Hilfsfunktionen statt in statischen Manager-Fassaden.
  Doctor-/Import-/Debug-Code verarbeitet explizite Legacy-Quelldateien außerhalb des Runtime-Sitzungsmanagers.
- Die veralteten Fassadenmethoden `SessionManager.newSession()` und
  `SessionManager.createBranchedSession()` wurden entfernt. Neue
  Sitzungen und Transkript-Nachfolger werden von ihrem jeweils zuständigen SQLite-Workflow erstellt, statt einen bereits geöffneten Manager in eine andere
  persistierte Sitzung umzuwandeln.
- Entscheidungen zum Forken übergeordneter Transkripte und die Fork-Erstellung akzeptieren
  `storePath` oder `sessionsDir` nicht mehr; sie verwenden den SQLite-
  Transkript-Scope `{agentId, sessionId}` statt beibehaltener Dateisystempfad-Metadaten.
- Memory-Host exportiert keine wirkungslosen Hilfsfunktionen zur Klassifizierung von Transkripten anhand des Sitzungsverzeichnisses mehr; die Transkriptfilterung wird jetzt beim Erstellen von Einträgen aus SQLite-Zeilen-Metadaten abgeleitet.
- Memory-Host- und QMD-Sitzungsexporttests verwenden SQLite-Transkript-Scopes. Alte
  `agents/<agentId>/sessions/*.jsonl`-Pfade werden nur noch dort abgedeckt, wo ein Test
  absichtlich die Kompatibilität von Doctor, Import oder Export nachweist.
- Die Rohsitzungsprüfung von QA Lab verwendet jetzt `sessions.list` über das Gateway,
  statt `agents/qa/sessions/sessions.json` zu lesen; MSteams-Feedback
  wird direkt an SQLite-Transkripte angehängt, ohne einen JSONL-Pfad vorzutäuschen.
- Gemeinsam genutzte eingehende Kanal-Turns enthalten jetzt `{agentId, sessionKey}` statt eines
  veralteten `storePath`. Die Aufzeichnungspfade von LINE, WhatsApp, Slack, Discord, Telegram, Matrix, Signal,
  iMessage, BlueBubbles, Feishu, Google Chat, IRC, Nextcloud Talk, Zalo,
  Zalo Personal, QA Channel, Microsoft Teams, Mattermost, Synology Chat, Tlon,
  Twitch und QQBot lesen jetzt Aktualisierungszeit-Metadaten und zeichnen
  eingehende Sitzungszeilen über die SQLite-Identität auf.
- Die Persistierung des Transkript-Locators wurde aus aktiven Sitzungszeilen entfernt.
  `resolveSessionTranscriptTarget` gibt `agentId`, `sessionId` und optionale
  Themenmetadaten zurück; Doctor ist der einzige Code, der Legacy-Transkriptdateinamen
  importiert.
- Runtime-Transkript-Header beginnen bei SQLite-Version `1`. Upgrades alter JSONL-V1-/V2-/V3-
  Strukturen finden nur beim Doctor-Import statt und normalisieren importierte Header auf
  die aktuelle SQLite-Transkriptversion, bevor Zeilen gespeichert werden.
- Der Database-first-Guard verbietet jetzt `SessionManager.listAll` und
  `SessionManager.forkFromSession`; das Auflisten von Sitzungen sowie Fork-/Wiederherstellungs-Workflows
  müssen zeilen- bzw. Scope-basierte SQLite-APIs verwenden.
- Der Guard verbietet außerhalb von Doctor-/Import-Code außerdem Namen veralteter Hilfsfunktionen zum Parsen von Transkript-JSONL und Reparieren aktiver Branches, sodass die Runtime keinen zweiten Legacy-
  Transkriptmigrationspfad entwickeln kann.
- Eingebettete PI-Läufe lehnen eingehende Transkript-Handles ab. Sie verwenden die SQLite-
  Identität `{agentId, sessionId}` vor dem Start des Workers und erneut, bevor der
  Versuch auf den Transkriptzustand zugreift. Eine veraltete `/tmp/*.jsonl`-Eingabe kann kein
  Runtime-Schreibziel auswählen.
- Cache-Trace-, Anthropic-Payload-, Rohdatenstrom- und Diagnostik-Zeitleistendatensätze
  werden jetzt in typisierte SQLite-Zeilen des Typs `diagnostic_events` geschrieben. Gateway-Stabilitätspakete
  werden jetzt in typisierte SQLite-Zeilen des Typs `diagnostic_stability_bundles` geschrieben. Die alten
  JSONL-Überschreibungspfade `diagnostics.cacheTrace.filePath`, `OPENCLAW_CACHE_TRACE_FILE`,
  `OPENCLAW_ANTHROPIC_PAYLOAD_LOG_FILE` und
  `OPENCLAW_DIAGNOSTICS_TIMELINE_PATH` wurden entfernt, und
  die normale Stabilitätserfassung schreibt keine `logs/stability/*.json`-Dateien mehr.
- Die Cron-Persistierung gleicht jetzt SQLite-Zeilen des Typs `cron_jobs` ab, statt
  bei jedem Speichern die gesamte Auftragstabelle zu löschen und neu einzufügen. Rückschreibungen von Plugin-Zielen
  aktualisieren übereinstimmende Cron-Zeilen direkt und halten den Runtime-Cron-Zustand in
  derselben Zustandsdatenbanktransaktion.
- Cron-Runtime-Aufrufer verwenden jetzt einen stabilen Schlüssel für den SQLite-Cron-Speicher. Legacy-
  Pfade vom Typ `cron.store` dienen nur als Doctor-Importeingaben; die Pfade für Produktions-Gateway, Aufgabenwartung,
  Status, Ausführungsverlauf und Telegram-Zielrückschreibung verwenden
  `resolveCronStoreKey` und normalisieren den Schlüssel nicht mehr als Pfad. Der Cron-Status
  meldet jetzt `storeKey` statt des alten dateiförmigen Felds `storePath`.
- Das Laden und Planen der Cron-Runtime normalisiert keine veralteten persistierten Auftragsstrukturen
  wie `jobId`, `schedule.cron`, numerisches `atMs`, boolesche Zeichenketten oder
  fehlendes `sessionTarget` mehr. Der Legacy-Import von Doctor führt diese Reparaturen durch, bevor Zeilen
  in SQLite eingefügt werden.
- ACP-Spawn löst oder persistiert keine JSONL-Dateipfade für Transkripte mehr. Die Einrichtung
  von Spawn und Thread-Bindung persistiert die SQLite-Sitzungszeile direkt und behält die
  Sitzungs-ID als Transkriptidentität bei.
- ACP-Sitzungsmetadaten-APIs lesen, listen und aktualisieren bzw. fügen SQLite-Zeilen jetzt anhand von `agentId` ein und
  stellen `storePath` nicht mehr als Teil des ACP-Sitzungseintragsvertrags bereit.
- Die Abrechnung der Sitzungsnutzung und die Gateway-Nutzungsaggregation lösen Transkripte jetzt
  ausschließlich anhand von `{agentId, sessionId}` auf. Der Kosten-/Nutzungscache und Zusammenfassungen erkannter Sitzungen
  erzeugen oder liefern keine Transkript-Locator-Zeichenketten mehr.
- Das Anhängen an Gateway-Chats, die Persistierung teilweise abgebrochener Vorgänge, `/sessions.send` und
  Webchat-Medien-Transkriptschreibvorgänge hängen direkt über den SQLite-Transkript-
  Scope an. Die Gateway-Hilfsfunktion zur Transkriptinjektion akzeptiert keinen
  `transcriptLocator`-Parameter mehr.
- Die SQLite-Transkripterkennung listet jetzt nur noch Transkript-Scopes und Statistiken auf:
  `{agentId, sessionId, updatedAt, eventCount}`. Die nicht mehr verwendete
  Kompatibilitäts-Hilfsfunktion `listSqliteSessionTranscriptLocators` und das zeilenbezogene
  Feld `locator` wurden entfernt.
- Die Runtime für Transkriptreparaturen stellt jetzt nur noch
  `repairTranscriptSessionStateIfNeeded({agentId, sessionId})` bereit. Die alte
  Locator-basierte Reparatur-Hilfsfunktion wurde gelöscht; Doctor-/Debug-Code liest explizite
  Quelldateipfade und migriert niemals Locator-Zeichenketten.
- Die ACP-Replay-Ledger-Runtime speichert sitzungsbezogene Replay-Zeilen jetzt in der gemeinsam genutzten
  SQLite-Zustandsdatenbank statt in `acp/event-ledger.json`; Doctor importiert und
  entfernt die Legacy-Datei.
- Gateway-Hilfsfunktionen zum Lesen von Transkripten befinden sich jetzt in
  `src/gateway/session-transcript-readers.ts` statt im alten Modul
  `session-utils.fs`. Die Prüfung des Fallback-Wiederholungsverlaufs ist nach
  SQLite-Transkriptinhalten statt nach der alten Datei-Hilfsfunktionsoberfläche benannt.
- Gateway-Hilfsfunktionen für injizierte Chats und Compaction übergeben jetzt den SQLite-Transkript-Scope
  über interne Hilfs-APIs, statt Werte als Transkriptpfade oder
  Quelldateien zu bezeichnen.
- Die Erkennung von Bootstrap-Fortsetzungen prüft jetzt SQLite-Transkriptzeilen über
  `hasCompletedBootstrapTranscriptTurn`; sie stellt keinen dateiförmigen
  Hilfsfunktionsnamen mehr bereit.
- Tests für den eingebetteten Runner verwenden jetzt die SQLite-Transkriptidentität, und das Öffnen eines neuen
  Transkriptmanagers erfordert immer ein explizites `sessionId`.
- Hilfsfunktionen zur Speicherindizierung verwenden jetzt durchgängig SQLite-Transkriptterminologie:
  Der Host exportiert `listSessionTranscriptScopesForAgent` und
  `sessionTranscriptKeyForScope`, die gezielte Synchronisierung reiht `sessionTranscripts` ein,
  Treffer der öffentlichen Sitzungssuche stellen undurchsichtige `transcript:<agent>:<session>`-Pfade bereit,
  und der interne DB-Quellschlüssel lautet `session:<session>` unter
  `source_kind='sessions'` statt eines vorgetäuschten Dateipfads.
- Die generische Persistent-Dedupe-Hilfsfunktion des Plugin-SDK stellt keine dateiförmigen
  Optionen mehr bereit. Aufrufer geben SQLite-Scope-Schlüssel an, und dauerhafte Dedupe-Zeilen befinden sich im
  gemeinsam genutzten Plugin-Zustand.
- Microsoft Teams-SSO-Token wurden aus gesperrten JSON-Dateien in den SQLite-Plugin-
  Zustand verschoben. Doctor importiert `msteams-sso-tokens.json`, rekonstruiert kanonische SSO-Token-
  Schlüssel aus Payloads und entfernt die Quelldatei. Delegierte OAuth-Token verbleiben
  an ihrer bestehenden privaten Grenze für Anmeldedatendateien.
- Der Zustand des Matrix-Synchronisierungscaches wurde von `bot-storage.json` in den SQLite-Plugin-
  Zustand verschoben. Doctor importiert rohe oder umschlossene Legacy-Synchronisierungs-Payloads und entfernt die
  Quelldatei. Aktive Matrix- und QA-Lab-Matrix-Adapterclients übergeben ein Stammverzeichnis für den SQLite-Synchronisierungsspeicher,
  nicht einen vorgetäuschten Pfad vom Typ `sync-store.json` oder `bot-storage.json`.
- Der Status der Legacy-Kryptomigration von Matrix wurde von
  `legacy-crypto-migration.json` in den SQLite-Plugin-Zustand verschoben. Doctor importiert die
  alte Statusdatei; IndexedDB-Snapshots des Matrix-SDK wurden von
  `crypto-idb-snapshot.json` in SQLite-Plugin-Blobs verschoben. Matrix-Wiederherstellungsschlüssel und
  Anmeldedaten sind SQLite-Plugin-Zustandszeilen; ihre alten JSON-Dateien dienen nur als Doctor-
  Migrationseingaben.
- Memory-Wiki-Aktivitätsprotokolle verwenden jetzt den SQLite-Plugin-Zustand statt
  `.openclaw-wiki/log.jsonl`. Der Migrations-Provider von Memory Wiki importiert alte
  JSONL-Protokolle; Wiki-Markdown und Inhalte des Benutzer-Vaults bleiben als
  Workspace-Inhalte dateibasiert.
- Memory Wiki erstellt `.openclaw-wiki/state.json` oder das ungenutzte
  Verzeichnis `.openclaw-wiki/locks` nicht mehr. Der Migrations-Provider entfernt diese ausgemusterten
  Plugin-Metadatendateien, falls ein älterer Vault sie noch enthält.
- Audit-Einträge des System-Agenten verwenden jetzt den zentralen SQLite-Plugin-Zustand statt
  `audit/crestodian.jsonl`. Doctor importiert das Legacy-JSONL-Auditprotokoll und
  entfernt es nach erfolgreichem Import.
- Audit-Einträge für das Schreiben/Beobachten der Konfiguration verwenden jetzt den zentralen SQLite-Plugin-Zustand statt
  `logs/config-audit.jsonl`. Doctor importiert das Legacy-JSONL-Auditprotokoll und
  entfernt es nach erfolgreichem Import.
- Die macOS-Begleitanwendung schreibt beim Bearbeiten von `openclaw.json` keine anwendungslokalen Sidecars vom Typ `logs/config-audit.jsonl` oder
  `logs/config-health.json` mehr. Die Konfigurationsdatei
  bleibt dateibasiert, Wiederherstellungs-Snapshots verbleiben neben der Konfigurationsdatei,
  und der dauerhafte Audit-/Integritätszustand der Konfiguration gehört in den SQLite-Speicher des Gateways.
- Ausstehende Rettungsgenehmigungen des System-Agenten verwenden jetzt den zentralen SQLite-Plugin-Zustand statt
  `crestodian/rescue-pending/*.json` oder `openclaw/rescue-pending/*.json`.
  Diese kurzlebigen Sicherheitsberechtigungen werden niemals importiert; Doctor verwirft
  beide ausgemusterten Verzeichnisse, sodass ein Upgrade einen veralteten Schreibvorgang nicht reaktivieren kann.
- Der temporäre Aktivierungszustand von Phone Control verwendet jetzt den SQLite-Plugin-Zustand statt
  `plugins/phone-control/armed.json`. Doctor importiert die Legacy-Datei mit dem Aktivierungszustand
  in den Namensraum `phone-control/arm-state` und entfernt die Datei.
- Doctor repariert JSONL-Transkripte nicht mehr direkt und erstellt keine JSONL-
  Sicherungsdateien mehr. Er importiert den aktiven Branch in SQLite und entfernt die Legacy-Quelle.
- Die Transkriptsuche des Sitzungsspeicher-Hooks verwendet ausschließlich Scope-basierte
  SQLite-Lesevorgänge über `{agentId, sessionId}`. Die Hilfsfunktion akzeptiert oder ermittelt keine Transkript-Locators,
  Legacy-Dateilesevorgänge oder Optionen zum Umschreiben von Dateien mehr.
- Konversationsbindungen des Codex-App-Servers verwenden jetzt den OpenClaw-Sitzungsschlüssel oder einen expliziten `{agentId, sessionId}`-
  Scope als Schlüssel für den SQLite-Plugin-Zustand. Sie dürfen keine Fallback-Bindungen für Transkriptpfade
  beibehalten.
- Lesevorgänge des gespiegelten Verlaufs im Codex-App-Server verwenden ausschließlich den SQLite-Transkript-Scope;
  sie dürfen die Identität nicht aus Transkriptdateipfaden wiederherstellen.
- Pfade für Rollensortierung und Compaction-Zurücksetzung entfernen alte Transkriptdateien nicht mehr;
  beim Zurücksetzen werden nur die SQLite-Sitzungszeile und die Transkriptidentität rotiert.
- Gateway-Antworten auf Zurücksetzungen und Checkpoints geben bereinigte Sitzungszeilen sowie Sitzungs-
  IDs zurück. Sie erzeugen keine SQLite-Transkript-Locators mehr für Clients.
- Dreaming von Memory-Core bereinigt Sitzungszeilen nicht mehr durch die Prüfung auf fehlende
  JSONL-Dateien. Die Bereinigung von Subagenten erfolgt über die Sitzungs-Runtime-API statt über
  Existenzprüfungen im Dateisystem. Die Tests zur Transkriptaufnahme legen SQLite-Zeilen
  direkt an, statt `agents/<id>/sessions`-Fixtures oder Locator-
  Platzhalter zu erstellen.
- Die Memory-Transkriptindizierung kann `transcript:<agentId>:<sessionId>` als
  virtuellen Suchtrefferpfad für Zitier-/Lese-Hilfsfunktionen bereitstellen. Die dauerhafte Indexquelle ist
  relational (`source_kind='sessions'`, `source_key='session:<sessionId>'`,
  `session_id=<sessionId>`), daher ist der Wert kein Locator für ein Laufzeittranskript,
  kein Dateisystempfad und darf niemals wieder an Sitzungs-Laufzeit-APIs übergeben werden.
- Der Speicherstatus von Gateway Doctor liest Kurzzeitabruf- und Phasensignalanzahlen
  aus SQLite-Zeilen des Plugin-Status statt aus `memory/.dreams/*.json`; die Ausgabe der CLI und
  von Doctor bezeichnet diesen Speicher nun als SQLite-Speicher und nicht als Pfad.
- Die Memory-Core-Laufzeit, der CLI-Status, Gateway-Doctor-Methoden und Plugin-SDK-
  Fassaden prüfen oder archivieren keine veralteten `.dreams/session-corpus`-Dateien mehr.
  Diese Dateien sind ausschließlich Migrationseingaben; Doctor importiert sie in SQLite und
  löscht die Quelle nach der Verifizierung. Nachweiszeilen für die aktive Sitzungserfassung
  verwenden nun den virtuellen SQLite-Pfad `memory/session-ingestion/<day>.txt`; die Laufzeit
  schreibt niemals Status in `.dreams/session-corpus` und leitet daraus auch keinen Status ab.
- Öffentliche Memory-Core-Artefakte stellen SQLite-Hostereignisse als virtuelles JSON-
  Artefakt `memory/events/memory-host-events.json` bereit; sie verwenden nicht mehr den
  veralteten Quellpfad `.dreams/events.jsonl`.
- Sandbox-Container-/Browser-Registrierungen verwenden nun die gemeinsame
  SQLite-Tabelle `sandbox_registry_entries` mit typisierten Spalten für Sitzung, Image, Zeitstempel,
  Backend/Konfiguration und Browser-Port. Doctor importiert veraltete monolithische und
  aufgeteilte JSON-Registrierungsdateien und entfernt erfolgreich importierte Quellen. Laufzeitlesevorgänge
  verwenden die typisierten Zeilenspalten als maßgebliche Datenquelle; `entry_json` ist nur eine
  Kopie für Wiedergabe/Debugging.
- Zusagen verwenden nun eine typisierte gemeinsame Tabelle `commitments` statt eines
  JSON-Blobs für den gesamten Speicher. Die Laufzeit verwendet indizierte Abfragen für Geltungsbereich,
  Zustellfenster, gleitende Obergrenze, Status und Versuche sowie synchrone SQLite-Transaktionen;
  `record_json` ist nur eine Kopie für Wiedergabe/Debugging. Eine explizite Doctor-Reparatur validiert
  die vollständige veraltete Datei `commitments.json`, behält neuere SQLite-Zeilen bei, verifiziert das
  Ergebnis und entfernt erst danach die unveränderte Quelle. Die Laufzeit liest oder
  schreibt die außer Betrieb genommene Datei niemals.
- Web-Push-Abonnements und die generierte VAPID-Identität verwenden nun typisierte gemeinsame
  Zeilen `web_push_subscriptions` und `web_push_vapid_keys`. Laufzeitregistrierung,
  Ablaufbereinigung und Schlüsselerzeugung bei der ersten Verwendung nutzen SQLite-
  Transaktionen auf Zeilenebene. Eine explizite Doctor-Reparatur validiert beide außer Betrieb genommenen JSON-Speicher,
  beansprucht sie vor dem SQLite-Schreibvorgang, importiert sie atomar, weist
  widersprüchliche VAPID-Identitäten zurück, verifiziert das Ergebnis und entfernt erst danach die
  Beanspruchungen. Doctor hält während des vollständigen Imports die Wartungssperre des Statusverzeichnisses,
  damit ein älterer Gateway die außer Betrieb genommenen Dateien nicht neu erstellen kann. Registrierung,
  Zustellung, Löschung und Schlüsselauflösung schlagen sicher fehl, bis Doctor
  ausstehende veraltete Quellen oder unterbrochene Beanspruchungen auflöst.
- Cron-Auftragsdefinitionen, Zeitplanstatus und Ausführungsverlauf haben keine JSON-
  Schreiber oder -Leser zur Laufzeit mehr. Die Laufzeit verwendet `cron_jobs`-Zeilen mit typisierten Spalten für Zeitplan,
  Nutzlast, Zustellung, Fehlerwarnung, Sitzung, Status und Laufzeitstatus sowie
  Cron-eigene `task_runs`-Details für Diagnose, Zustellung, Sitzung/Ausführung, Modell
  und Token-Gesamtzahlen. `job_json` ist nur eine Kopie für Wiedergabe/Debugging; `state_json` enthält verschachtelte
  Laufzeitdiagnosen, die noch keine häufig abgefragten Felder besitzen, während die Laufzeit
  häufig verwendete Statusfelder aus typisierten Spalten rehydriert. Doctor importiert
  veraltete Dateien `jobs.json`, `jobs-state.json` und `runs/*.jsonl` und entfernt
  die importierten Quellen. Rückschreibungen von Plugin-Zielen aktualisieren passende `cron_jobs`-
  Zeilen, statt den gesamten Cron-Speicher zu laden und zu ersetzen.
- Der Gateway-Start ignoriert veraltete `notify: true`-Markierungen in der Laufzeit-
  projektion. Doctor liest den außer Betrieb genommenen Rohwert `cron.webhook` nur, während
  diese Markierungen in eine explizite SQLite-Zustellung übersetzt werden, und entfernt anschließend den Konfigurationsschlüssel.
- Ausgehende Zustellwarteschlangen und Sitzungszustellwarteschlangen speichern nun Warteschlangenstatus, Eintragstyp,
  Sitzungsschlüssel, Kanal, Ziel, Konto-ID, Wiederholungsanzahl, letzten Versuch/Fehler,
  Wiederherstellungsstatus und Plattform-Sendemarkierungen als typisierte Spalten in der gemeinsamen
  Tabelle `delivery_queue_entries`. Die Laufzeitwiederherstellung liest diese häufig verwendeten Felder aus
  den typisierten Spalten, und Wiederholungs-/Wiederherstellungsmutationen aktualisieren diese Spalten direkt,
  ohne Wiedergabe-JSON neu zu schreiben. Die vollständige JSON-Nutzlast bleibt nur als
  Wiedergabe-/Debugging-Blob für Nachrichtentexte und andere selten verwendete Wiedergabedaten erhalten.
- Verwaltete Datensätze ausgehender Bilder verwenden nun typisierte gemeinsame
  `managed_outgoing_image_records`-Zeilen. Die Laufzeit liest ausschließlich typisierte Spalten; die
  JSON-Spalte ist eine Kopie für Wiedergabe/Debugging. Die ursprünglichen Bildbytes bleiben benannte
  Anhangsartefakte im Verzeichnis für verwaltete Medien.
- Discord-Einstellungen für die Modellauswahl, Hashes der Befehlsbereitstellung und Thread-Bindungen
  verwenden nun den gemeinsamen SQLite-Plugin-Status. Ihre Importpläne für veraltetes JSON befinden sich in der
  Einrichtungs-/Doctor-Migrationsoberfläche des Discord-Plugins, nicht im Kernmigrationscode.
- Importdetektoren für veraltete Plugin-Daten verwenden nach Doctor benannte Module wie
  `doctor-legacy-state.ts` oder `doctor-state-imports.ts`; normale Kanallaufzeit-
  module dürfen keine Detektoren für veraltetes JSON importieren.
- BlueBubbles-Nachholcursor und Deduplizierungsmarkierungen für eingehende Daten verwenden nun den gemeinsamen SQLite-
  Plugin-Status. Ihre Importpläne für veraltetes JSON befinden sich in der Einrichtungs-/Doctor-Migrationsoberfläche
  des BlueBubbles-Plugins, nicht im Kernmigrationscode.
- Telegram-Aktualisierungs-Offsets, Sticker-Cache-Zeilen, Cache-Zeilen gesendeter Nachrichten,
  Themennamen-Cache-Zeilen und Thread-Bindungen verwenden nun den gemeinsamen SQLite-Plugin-
  Status. Ihre Importpläne für veraltetes JSON befinden sich in der Einrichtungs-/Doctor-Migrationsoberfläche
  des Telegram-Plugins, nicht im Kernmigrationscode.
- iMessage-Nachholcursor, Zuordnungen kurzer Antwort-IDs und Deduplizierungszeilen gesendeter Echos
  verwenden nun den gemeinsamen SQLite-Plugin-Status. Die alten Dateien `imessage/catchup/*.json`,
  `imessage/reply-cache.jsonl` und `imessage/sent-echoes.jsonl` sind
  ausschließlich Doctor-Eingaben.
- Feishu-Zeilen zur Nachrichtendeduplizierung verwenden nun die beanspruchbare Deduplizierung des Kerns
  (`feishu.dedup.*`-Namensräume im gemeinsamen SQLite-Plugin-Status) statt
  `feishu/dedup/*.json`-Dateien oder des außer Betrieb genommenen, individuell implementierten `dedup.*`-Speichers,
  ohne Altimport, da der Cache zum Schutz vor Wiedergaben nach dem Upgrade neu aufgebaut wird.
- Microsoft Teams-Unterhaltungen, Umfragen, ausstehende Upload-Puffer und Feedback-
  Erkenntnisse verwenden nun gemeinsame SQLite-Tabellen für Plugin-Status/Blobs. Der Pfad für ausstehende Uploads
  verwendet `plugin_blob_entries`, sodass Medienpuffer als SQLite-BLOBs
  statt als Base64-JSON gespeichert werden. Die Namen der Laufzeithelfer verwenden nun SQLite-/Status-
  Benennungen statt der `*-fs`-Dateispeicherbenennung, und der alte `storePath`-Shim wurde
  aus diesen Speichern entfernt. Der Importplan für veraltetes JSON befindet sich in der
  Einrichtungs-/Doctor-Migrationsoberfläche des Microsoft Teams-Plugins.
- Von Zalo gehostete ausgehende Medien verwenden nun das gemeinsame SQLite-Objekt `plugin_blob_entries`
  statt temporärer JSON-/Binär-Sidecars `openclaw-zalo-outbound-media`.
- HTML und Metadaten des Diff-Viewers verwenden nun das gemeinsame SQLite-Objekt `plugin_blob_entries`
  statt temporärer Dateien `meta.json`/`viewer.html`. Viewer-HTML wird als
  Gzip-Blob gespeichert, und nur der Hash des URL-Tokens wird persistiert. Gerenderte PNG-/PDF-Ausgaben
  bleiben temporäre Materialisierungen, weil die Kanalzustellung weiterhin einen Dateipfad benötigt;
  ihre Ablaufmetadaten werden von SQLite verwaltet, ohne JSON-Sidecars.
- Verwaltete Canvas-Dokumente verwenden nun das gemeinsame SQLite-Objekt `plugin_blob_entries` statt
  eines standardmäßigen Verzeichnisses `state/canvas/documents`. Der Canvas-Host stellt diese
  Blobs direkt bereit; lokale Dateien werden nur für explizite `host.root`-
  Betreiberinhalte oder zur temporären Materialisierung erstellt, wenn ein nachgelagerter Medienleser
  einen Pfad benötigt.
- Auditentscheidungen für Dateiübertragungen verwenden nun das gemeinsame SQLite-Objekt `plugin_state_entries`
  statt des unbegrenzten Laufzeitprotokolls `audit/file-transfer.jsonl`. Doctor
  importiert die veraltete JSONL-Auditdatei in den Plugin-Status und entfernt die Quelle
  nach einem fehlerfreien Import.
- ACPX-Prozess-Leases und die Gateway-Instanzidentität verwenden nun den gemeinsamen SQLite-Plugin-
  Status. Doctor importiert die veraltete Datei `gateway-instance-id` in den Plugin-Status
  und entfernt die Quelle.
- Von ACPX generierte Wrapper-Skripte und das isolierte Codex-Basisverzeichnis sind temporäre
  Materialisierungen unterhalb des OpenClaw-Temporärstammverzeichnisses und kein dauerhafter OpenClaw-Status. Die
  dauerhaften ACPX-Laufzeitdatensätze sind die SQLite-Zeilen für Lease und Gateway-Instanz;
  die alte ACPX-Konfigurationsoberfläche `stateDir` wurde entfernt, da dort kein Laufzeitstatus
  mehr geschrieben wird.
- Gateway-Medienanhänge verwenden nun die gemeinsame SQLite-Tabelle `media_blobs` als
  kanonischen Bytespeicher. Lokale Pfade, die an Kompatibilitätsoberflächen für Kanäle und Sandboxen
  zurückgegeben werden, sind temporäre Materialisierungen der Datenbankzeile und nicht der
  dauerhafte Medienspeicher. Laufzeit-Medien-Zulassungslisten enthalten nicht mehr die veralteten
  Stammverzeichnisse `$OPENCLAW_STATE_DIR/media` oder `media` aus dem Konfigurationsverzeichnis; diese Verzeichnisse sind
  ausschließlich Doctor-Importquellen.
- Die Shell-Vervollständigung schreibt keine Cache-
  Dateien `$OPENCLAW_STATE_DIR/completions/*` mehr. Installations-, Doctor-, Aktualisierungs- und Release-Smoke-Pfade verwenden generierte
  Vervollständigungsausgaben oder das Einlesen von Profilen statt dauerhafter Vervollständigungs-Cache-
  Dateien.
- Das Staging für Gateway-Skill-Uploads verwendet nun gemeinsame Zeilen `skill_uploads` und
  `skill_upload_chunks`. Chunks bleiben während des Uploads einzeln transaktional;
  beim Commit wird anschließend ein einzelnes verifiziertes Archiv-BLOB zusammengesetzt und die Chunk-
  Zeilen werden entfernt. Das Installationsprogramm erhält nur während einer laufenden Installation einen
  temporär materialisierten Archivpfad. Doctor verwirft den außer Betrieb genommenen einstündigen
  Dateisystem-Staging-Baum, statt vorübergehende Uploads zu importieren.
- Inline-Anhänge von Subagenten werden nicht mehr unterhalb des Workspace-Verzeichnisses
  `.openclaw/attachments/*` materialisiert. Der Spawn-Pfad bereitet SQLite-VFS-Starteinträge vor,
  Inline-Ausführungen übertragen diese Einträge in den Laufzeit-Scratch-Namensraum des jeweiligen Agenten,
  und datenträgergestützte Tools legen diesen SQLite-Scratch für Anhangspfade darüber. Die
  alten Registrierungsspalten für Anhangsverzeichnisse von Subagentenausführungen und die Bereinigungs-Hooks wurden entfernt.
- Die CLI-Bildhydratisierung verwaltet keine stabilen Cache-
  Dateien `openclaw-cli-images` mehr. Externe CLI-Backends erhalten weiterhin Dateipfade, diese Pfade sind jedoch
  temporäre Materialisierungen pro Ausführung mit anschließender Bereinigung.
- Cache-Trace-Diagnosen, Anthropic-Nutzlastdiagnosen, Rohmodellstream-
  Diagnosen, Diagnose-Zeitleistenereignisse und Gateway-Stabilitätspakete
  schreiben nun SQLite-Zeilen statt Dateien `logs/*.jsonl` oder
  `logs/stability/*.json`.
  Laufzeit-Flags und Umgebungsvariablen zur Pfadüberschreibung wurden entfernt; Export-/Debugging-
  Befehle können Dateien explizit aus Datenbankzeilen materialisieren.
- Die macOS-Begleitanwendung besitzt keinen rollierenden Schreiber für `diagnostics.jsonl` mehr. App-
  Protokolle werden in das einheitliche Protokollierungssystem geschrieben, und dauerhafte Gateway-Diagnosen bleiben SQLite-gestützt.
- Die macOS-Port-Guardian-Datensatzliste verwendet nun typisierte gemeinsame SQLite-
  Zeilen `macos_port_guardian_records` statt einer JSON-Datei unter Application Support
  oder eines undurchsichtigen Singleton-Blobs. Alle macOS-App-Profile verwenden dieselbe hostglobale native
  Datenbank, da sie lokale Ports des Rechners koordinieren. Jeder Ledger-Vorgang
  blockiert, solange eine ältere App-Version mit JSON-Schreibzugriff ausgeführt wird. Die Migration tritt dem stabilen
  Dateisperrprotokoll des alten Ledgers nur bei, um die Quelle als Snapshot zu erfassen und später erneut zu validieren.
  Sie löst jede veraltete Zeile anhand aktueller Befehls- und Prozessstartfakten auf,
  ohne diese Sperre zu halten, liest anschließend die maßgeblichen SQLite-Zeilen erneut, wendet den
  Plan an, verifiziert jeden Beleg und entfernt die Quelle. Wiederholte Entfernungsversuche planen
  fehlende Zeilen neu, sodass außer Betrieb genommene veraltete Belege nicht wiederhergestellt werden können. Die Sperre bleibt
  kurzlebig, damit sie einen älteren Schreiber nach dem Start von SSH nicht blockiert. Die Umstellung ist
  bewusst unidirektional: Die Laufzeit im Regelbetrieb liest, projiziert oder schreibt niemals JSON,
  und ein Rollback auf Builds, die ausschließlich JSON verwenden, bewahrt neuere SQLite-Belege nicht.
- Gateway-Singleton-Sperren verwenden nun typisierte gemeinsame SQLite-Zeilen `state_leases` im
  Geltungsbereich `gateway_locks` statt Sperrdateien im temporären Verzeichnis. Die Dokumentation zur Fehlerbehebung
  für Fly und OAuth verweist nun auf die SQLite-Lease-/Authentifizierungsaktualisierungssperre statt
  auf die Bereinigung veralteter Dateisperren.
- Der Gateway-Neustart-Sentinel-Status verwendet jetzt typisierte Zeilen in der gemeinsam genutzten SQLite-Datenbank
  `gateway_restart_sentinel` anstelle von `restart-sentinel.json`; die Laufzeit
  liest Sentinel-Art, Status, Routing, Nachricht, Fortsetzung und Statistiken aus
  typisierten Spalten. Diese Spalten sind maßgeblich; `payload_json` ist lediglich ein
  Replay-/Debug-Abbild. Die Laufzeitpfade zum Lesen, Schreiben und Löschen verwenden ausschließlich SQLite.
  Ein begrenztes Statusmigrationsmodul wird beim Start und durch Doctor ausgeführt, um einen
  validierten älteren Post-Update-Sentinel vor der normalen Neustartwiederherstellung zu importieren, die
  typisierte Zeile zu überprüfen und die Quelldatei zu entfernen. Kein Laufzeitmodul im
  regulären Betrieb liest, schreibt oder bereinigt die Legacy-Datei.
- Gateway-Neustartabsicht und Status der Supervisor-Übergabe verwenden jetzt typisierte gemeinsam genutzte
  SQLite-Zeilen `gateway_restart_intent` und `gateway_restart_handoff` anstelle der
  Sidecar-Dateien `gateway-restart-intent.json` und
  `gateway-supervisor-restart-handoff.json`.
- Die Gateway-Singleton-Koordination verwendet jetzt typisierte `state_leases`-Zeilen unter
  `gateway_locks`, statt `gateway.<hash>.lock`-Dateien zu schreiben. Die Lease-Zeile
  enthält Lock-Eigentümer, Ablaufzeit, Heartbeat und Debug-Nutzlast; SQLite verwaltet die
  atomare Grenze für Erwerb und Freigabe. Die eingestellte Option für das Datei-Lock-Verzeichnis
  wurde entfernt; Tests verwenden direkt die Identität der SQLite-Zeile.
- Der alte, nicht referenzierte Hilfsmechanismus für Cron-Nutzungsberichte, der `cron/runs/*.jsonl`-Dateien
  durchsuchte, wurde gelöscht. Berichte zum Verlauf von Cron-Ausführungen lesen Cron-eigene `task_runs`-Zeilen.
- Die Neustartwiederherstellung der Hauptsitzung ermittelt mögliche Agenten jetzt über die
  SQLite-Registry `agent_databases`, statt `agents/*/sessions`-Verzeichnisse
  zu durchsuchen.
- Die Wiederherstellung bei beschädigten Gemini-Sitzungen löscht jetzt nur die SQLite-Sitzungszeile;
  sie benötigt kein Legacy-Gate `storePath` mehr und versucht nicht mehr, einen abgeleiteten
  JSONL-Transkriptpfad zu entfernen.
- Die Verarbeitung von Pfadüberschreibungen behandelt literale Umgebungswerte `undefined`/`null`
  jetzt als nicht gesetzt. Dadurch werden bei Tests oder Shell-Übergaben versehentliche
  `undefined/state/*.sqlite`-Datenbanken im Repository-Stammverzeichnis verhindert.
- Fingerabdrücke für den Konfigurationszustand verwenden jetzt typisierte gemeinsam genutzte SQLite-Zeilen `config_health_entries`
  anstelle von `logs/config-health.json`, sodass die normale Konfigurationsdatei
  das einzige Konfigurationsdokument ohne Anmeldedaten bleibt. Die macOS-Begleitanwendung behält nur
  prozesslokalen Zustandsstatus und erstellt die alte JSON-Sidecar-Datei nicht erneut.
- Die Laufzeit für Authentifizierungsprofile importiert oder schreibt keine JSON-Dateien mit Anmeldedaten mehr. Der
  kanonische Speicher für Anmeldedaten ist SQLite; `auth-profiles.json`, agentenspezifische
  `auth.json` und gemeinsam genutzte `credentials/oauth.json` sind Migrationseingaben für Doctor,
  die nach dem Import entfernt werden.
- Tests zum Speichern und Status von Authentifizierungsprofilen prüfen jetzt direkt typisierte SQLite-Authentifizierungstabellen
  und verwenden Legacy-Dateinamen für Authentifizierungsprofile ausschließlich als Migrationseingaben für Doctor.
- `openclaw secrets apply` bereinigt ausschließlich die Konfigurationsdatei, die Umgebungsdatei und den
  SQLite-Speicher für Authentifizierungsprofile. Es enthält keine Kompatibilitätslogik mehr, die
  eingestellte agentenspezifische `auth.json` bearbeitet; Doctor ist für den Import und das Löschen dieser Datei zuständig.
- Hermes-Pläne zur Secret-Migration planen und übernehmen importierte API-Schlüsselprofile direkt
  in den SQLite-Speicher für Authentifizierungsprofile. `auth-profiles.json` wird nicht mehr
  als Zwischenziel geschrieben oder überprüft.
- Benutzerorientierte Authentifizierungsdokumentation beschreibt jetzt
  `state/openclaw.sqlite#table/auth_profile_stores/<agentDir>`, statt
  Benutzer anzuweisen, `auth-profiles.json` zu prüfen oder zu kopieren; Legacy-Bezeichnungen für OAuth-/Authentifizierungs-JSON
  bleiben ausschließlich als Doctor-Importeingaben dokumentiert.
- MCP-OAuth-Sitzungen verwenden jetzt versionierte `mcp_oauth_stores`-Zeilen in der gemeinsam genutzten
  `state/openclaw.sqlite`. SDK-eigene Token-, Clientregistrierungs- und Discovery-
  Objekte bleiben eine einzige validierte JSON-Nutzlast, damit Erweiterungsfelder von Abhängigkeiten
  erhalten bleiben, während jeder Lese-/Änderungs-/Schreibvorgang in einer kurzen Kysely-
  Transaktion festgeschrieben wird. Eine gemeinsam genutzte SQLite-Lease serialisiert Aktualisierung, Anmeldung und Abmeldung;
  eingebettete MCP-Transporte lassen das MCP SDK Aktualisierungen nicht mehr außerhalb dieser
  Lease durchführen. Doctor importiert und entfernt ausschließlich eingestellte `mcp-oauth/*.json`-
  Speicher mit Quellbelegen, und die Laufzeit verfügt über keinen Datei-Fallback.
- Hilfsfunktionen für zentrale Statuspfade stellen die eingestellte Datei `credentials/oauth.json`
  nicht mehr bereit. Der Legacy-Dateiname ist lokal auf den Doctor-Importpfad für Authentifizierung beschränkt.
- Dokumentation zu Installation, Sicherheit, Onboarding, Modellauthentifizierung und SecretRef beschreibt jetzt
  SQLite-Zeilen für Authentifizierungsprofile sowie Sicherung und Migration des Gesamtstatus anstelle
  agentenspezifischer JSON-Dateien für Authentifizierungsprofile.
- Die PI-Modellerkennung übergibt kanonische Anmeldedaten jetzt an den speicherinternen
  `pi-coding-agent`-Authentifizierungsspeicher. Sie erstellt, bereinigt oder schreibt während der Erkennung
  keine agentenspezifischen `auth.json` mehr.
- Auslöse- und Routing-Einstellungen für Voice Wake verwenden jetzt typisierte gemeinsam genutzte SQLite-Tabellen
  anstelle von `settings/voicewake.json`, `settings/voicewake-routing.json` oder
  undurchsichtigen generischen Zeilen; Doctor importiert die Legacy-JSON-Dateien und entfernt sie nach einer
  erfolgreichen Migration.
- Der Status der Update-Prüfung verwendet jetzt eine typisierte gemeinsam genutzte `update_check_state`-Zeile anstelle von
  `update-check.json` oder einem undurchsichtigen generischen Blob; Doctor importiert
  die Legacy-JSON-Datei und entfernt sie nach einer erfolgreichen Migration.
- Der Konfigurationszustand verwendet jetzt typisierte gemeinsam genutzte `config_health_entries`-Zeilen anstelle
  von `logs/config-health.json` oder einem undurchsichtigen generischen Blob; Doctor
  importiert die Legacy-JSON-Datei und entfernt sie nach einer erfolgreichen Migration.
- Genehmigungen für Plugin-Konversationsbindungen verwenden jetzt typisierte
  `plugin_binding_approvals`-Zeilen anstelle eines undurchsichtigen gemeinsam genutzten SQLite-Status oder
  `plugin-binding-approvals.json`; die Legacy-Datei ist eine Migrationseingabe für Doctor.
- Generische Bindungen der aktuellen Konversation speichern jetzt typisierte
  `current_conversation_bindings`-Zeilen, statt
  `bindings/current-conversations.json` neu zu schreiben; Doctor importiert die Legacy-JSON-Datei und
  entfernt sie nach einer erfolgreichen Migration.
- Synchronisationsregister für importierte Quellen von Memory Wiki speichern jetzt eine SQLite-Plugin-Statuszeile
  pro Vault-/Quellschlüssel, statt `.openclaw-wiki/source-sync.json` neu zu schreiben;
  der Migrations-Provider importiert und entfernt das Legacy-JSON-Register.
- Datensätze zu ChatGPT-Importausführungen von Memory Wiki speichern jetzt eine SQLite-Plugin-Statuszeile
  pro Vault-/Ausführungs-ID, statt `.openclaw-wiki/import-runs/*.json` zu schreiben.
  Rollback-Snapshots bleiben explizite Vault-Dateien, bis die Archivierung von Snapshots
  der Importausführung in den Blob-Speicher verlagert wird.
- Kompilierte Digests von Memory Wiki speichern jetzt komprimierte SQLite-Plugin-Blob-Zeilen,
  statt `.openclaw-wiki/cache/agent-digest.json` und
  `.openclaw-wiki/cache/claims.jsonl` zu schreiben. Der Cache kann neu aufgebaut werden, daher löscht Doctor
  alte Cache-Dateien, ohne sie zu importieren.
- Die Nachverfolgung installierter ClawHub-Skills speichert jetzt eine SQLite-Plugin-Statuszeile pro
  Workspace/Skill, statt zur Laufzeit die Sidecar-Dateien `.clawhub/lock.json` und
  `.clawhub/origin.json` zu schreiben oder zu lesen. Laufzeitcode verwendet Statusobjekte
  für nachverfolgte Installationen anstelle dateiförmiger Lockfile-/Ursprungsabstraktionen. Doctor
  importiert die Legacy-Sidecar-Dateien aus konfigurierten Agenten-Workspaces und entfernt sie
  nach einem fehlerfreien Import.
- Der Index installierter Plugins liest und schreibt jetzt die typisierte gemeinsam genutzte SQLite-
  Singleton-Zeile `installed_plugin_index` anstelle von `plugins/installs.json`; die
  Legacy-JSON-Datei dient ausschließlich als Migrationseingabe für Doctor und wird nach dem Import entfernt.
- Die Legacy-Pfadhilfsfunktion `plugins/installs.json` befindet sich jetzt im Legacy-
  Code von Doctor. Laufzeitmodule für den Plugin-Index stellen nur noch SQLite-gestützte Persistenzoptionen
  bereit, keinen JSON-Dateipfad.
- Gateway-Neustart-Sentinel, Neustartabsicht und Status der Supervisor-Übergabe verwenden jetzt
  typisierte gemeinsam genutzte SQLite-Zeilen (`gateway_restart_sentinel`,
  `gateway_restart_intent` und `gateway_restart_handoff`) anstelle generischer
  undurchsichtiger Blobs. Der Neustartcode der Laufzeit besitzt keinen dateiförmigen Vertrag für Sentinel, Absicht oder Übergabe.
- Matrix-Synchronisationscache, Speichermetadaten, Thread-Bindungen, Markierungen zur Deduplizierung eingehender Daten,
  Cooldown-Status der Startüberprüfung, IndexedDB-Krypto-Snapshots des SDK,
  Anmeldedaten und Wiederherstellungsschlüssel verwenden jetzt gemeinsam genutzte SQLite-Tabellen für Plugin-Status und -Blobs.
  Laufzeit-Pfadstrukturen stellen keinen `storage-meta.json`-Metadatenpfad mehr bereit;
  dieser Dateiname dient ausschließlich als Legacy-Migrationseingabe. Ihr Legacy-JSON-Importplan
  befindet sich in der Setup-/Doctor-Migrationsoberfläche des Matrix-Plugins. Markierungen zur
  Deduplizierung eingehender Daten verwenden die beanspruchbare zentrale Deduplizierung (`matrix.inbound-dedupe.*`-
  Namespaces in der gemeinsam genutzten Statusdatenbank); die Matrix-Doctor-Statusmigration importiert
  einmalig die eingestellten root-spezifischen `inbound-dedupe`-Zeilen und `inbound-dedupe.json`,
  anschließend liest die Laufzeit ausschließlich den Speicher für beanspruchbare Deduplizierung.
- Beim Start durchsucht oder meldet Matrix keinen Legacy-Matrix-Dateistatus mehr und
  vervollständigt diesen auch nicht. Matrix-Dateierkennung, Erstellung von Legacy-Krypto-Snapshots, Migrationsstatus
  für die Wiederherstellung von Raumschlüsseln, Import und Entfernung der Quelle liegen vollständig in der Verantwortung von Doctor.
- Matrix-Barrels für Laufzeitmigrationen wurden entfernt. Hilfsfunktionen zur Erkennung und
  Änderung von Legacy-Status und -Kryptodaten werden direkt von Matrix Doctor importiert, statt
  Teil der Laufzeit-API-Oberfläche zu sein.
- Markierungen zur Wiederverwendung von Matrix-Migrations-Snapshots befinden sich jetzt im SQLite-Plugin-Status
  anstelle von `matrix/migration-snapshot.json`; Doctor kann dasselbe
  überprüfte Archiv vor der Migration weiterhin wiederverwenden, ohne eine Sidecar-Statusdatei zu schreiben.
- Nostr-Bus-Cursor und Status der Profilveröffentlichung verwenden jetzt gemeinsam genutzten SQLite-Plugin-
  Status. Ihr Legacy-JSON-Importplan befindet sich in der Setup-/Doctor-
  Migrationsoberfläche des Nostr-Plugins.
- Active Memory-Sitzungsumschalter verwenden jetzt gemeinsam genutzten SQLite-Plugin-Status anstelle von
  `session-toggles.json`; beim erneuten Aktivieren des Speichers wird die Zeile gelöscht, statt
  ein JSON-Objekt neu zu schreiben.
- Vorschläge und Prüfungszähler von Skill Workshop verwenden jetzt gemeinsam genutzten SQLite-Plugin-
  Status anstelle Workspace-spezifischer `skill-workshop/<workspace>.json`-Speicher. Jeder
  Vorschlag ist eine separate Zeile unter `skill-workshop/proposals`, und der
  Prüfungszähler ist eine separate Zeile unter `skill-workshop/reviews`.
- Ausführungen des Prüfer-Subagenten von Skill Workshop verwenden jetzt den Laufzeit-Resolver für Sitzungstranskripte,
  statt `skill-workshop/<sessionId>.json`-Sidecar-Sitzungspfade zu erstellen.
- ACPX-Prozess-Leases verwenden jetzt gemeinsam genutzten SQLite-Plugin-Status unter
  `acpx/process-leases` anstelle einer dateiweiten `process-leases.json`-Registry.
  Jede Lease wird als eigene Zeile gespeichert, wodurch die Bereinigung veralteter Prozesse
  beim Start erhalten bleibt, ohne dass zur Laufzeit ein JSON-Neuschreibpfad erforderlich ist.
- ACPX-Wrapper-Skripte und das isolierte Codex-Home werden im temporären
  Stammverzeichnis von OpenClaw generiert. Sie werden bei Bedarf neu erstellt und sind keine Sicherungs-
  oder Migrationseingaben.
- Die Persistenz der Registry für Subagent-Ausführungen verwendet typisierte gemeinsam genutzte `subagent_runs`-Zeilen. Der
  alte Pfad `subagents/runs.json` dient jetzt ausschließlich als Bereinigungseingabe für Doctor. Doctor
  beansprucht ihn unter dem Lock für die Statuswartung, zeichnet die Verwerfungsentscheidung in
  SQLite auf und entfernt ihn, ohne den transienten Ausführungsstatus zu importieren. Es verbleiben weder
  JSON-Leser oder -Schreiber noch Cache oder Fallback in der Laufzeit; die versionsübergreifende Wiederherstellung von ausschließlich
  dateibasierten laufenden Ausführungen wird an dieser Einstellungsgrenze absichtlich nicht unterstützt.
  Laufzeittests erstellen keine ungültigen oder leeren `runs.json`-Fixtures mehr, um
  das Registry-Verhalten nachzuweisen; sie befüllen und lesen SQLite-Zeilen direkt.
- Die Sicherung stellt das Statusverzeichnis vor der Archivierung bereit, kopiert Dateien, die keine Datenbanken sind,
  erstellt Datenbank-Snapshots mittels Online-Sicherung plus Offline-`VACUUM`, lässt aktive WAL-/SHM-Sidecar-Dateien aus, zeichnet
  Snapshot-Metadaten im Archivmanifest auf und speichert
  abgeschlossene Sicherungsläufe zusammen mit dem Archivmanifest in SQLite. `openclaw backup
create` validiert das geschriebene Archiv standardmäßig; `--no-verify` ist der
  explizite Schnellpfad.
- `openclaw backup restore` validiert das Archiv vor dem Extrahieren, verwendet das
  normalisierte Manifest des Prüfers erneut und stellt verifizierte Manifest-Ressourcen an ihren
  aufgezeichneten Quellpfaden wieder her. Für Schreibvorgänge ist `--yes` erforderlich, und `--dry-run`
  wird für einen Wiederherstellungsplan unterstützt.
- Der alte Filter für flüchtige Sicherungspfade wurde gelöscht. Die Sicherung benötigt keine
  Skip-Liste für Live-Tar mehr, um Legacy-JSON-/JSONL-Dateien von Sitzungen oder Cron auszuschließen, da SQLite-
  Snapshots vor der Archiverstellung bereitgestellt werden.
- Die einfache Einrichtung und die Workspace-Vorbereitung beim Onboarding erstellen keine
  `agents/<agentId>/sessions/`-Verzeichnisse mehr. Sie erstellen nur die Konfiguration und den Workspace;
  SQLite-Sitzungszeilen und Transkriptzeilen werden bei Bedarf in der
  agentenspezifischen Datenbank erstellt.
- Die Reparatur von Sicherheitsberechtigungen betrifft jetzt die globale und die agentenspezifischen SQLite-
  Datenbanken einschließlich WAL/SHM-Begleitdateien statt `sessions.json` und Transkript-
  JSONL-Dateien.
- Die Laufzeitnamen der Sandbox-Registry beschreiben jetzt direkt die SQLite-Registry-Arten,
  statt veraltete JSON-Registry-Terminologie im aktiven Speicher weiterzuführen.
- `openclaw reset --scope config+creds+sessions` entfernt agentenspezifische
  `openclaw-agent.sqlite`-Datenbanken einschließlich WAL/SHM-Begleitdateien, nicht nur veraltete
  `sessions/`-Verzeichnisse.
- Die Gateway-Hilfsfunktionen für aggregierte Sitzungen verwenden jetzt eintragsorientierte Namen:
  `loadCombinedSessionEntriesForGateway` gibt `{ databasePath, entries }` zurück.
  Die alte Benennung des kombinierten Speichers wurde aus den Laufzeitaufrufern entfernt.
- Das Seeding des Docker-MCP-Kanals schreibt jetzt die Hauptsitzungszeile und Transkript-
  ereignisse in die agentenspezifische SQLite-Datenbank, statt
  `sessions.json` und ein JSONL-Transkript zu erstellen.
- Der gebündelte Session-Memory-Hook löst den Kontext der vorherigen Sitzung jetzt anhand von
  `{agentId, sessionId}` aus SQLite auf. Er durchsucht, speichert oder synthetisiert keine
  Transkriptpfade oder `workspace/sessions`-Verzeichnisse mehr.
- Der gebündelte Command-Logger-Hook schreibt Befehlsauditzeilen jetzt in die gemeinsam genutzte
  SQLite-Tabelle `command_log_entries`, statt sie an
  `logs/commands.log` anzuhängen.
- Kanal-Kopplungs-Positivlisten stellen zur Laufzeit jetzt nur noch SQLite-gestützte Lese-/Schreibhilfen
  bereit. Der veraltete Pfadauflöser des Plugin-SDK bleibt für die Migrationskompatibilität
  erhalten; Dateileser existieren nur noch im Doctor-Code für die Zustandsmigration.
- `migration_runs` zeichnet Ausführungen der Migration veralteter Zustände mit Status,
  Zeitstempeln und JSON-Berichten auf.
- `migration_sources` zeichnet jede importierte Quelle einer veralteten Datei mit Hash, Größe,
  Datensatzanzahl, Zieltabelle, Ausführungs-ID, Status und Zustand der Quellentfernung auf.
- `backup_runs` zeichnet Pfade von Sicherungsarchiven, Status und JSON-Manifeste auf.
- Das globale Schema enthält keine ungenutzte `agents`-Registry-Tabelle. Die Erkennung von
  Agentendatenbanken ist die kanonische `agent_databases`-Registry, bis die Laufzeit
  einen echten Eigentümer für Agentendatensätze besitzt.
- Die generierte Modellkatalogkonfiguration wird in typisierten globalen SQLite-
  `agent_model_catalogs`-Zeilen mit dem Agentenverzeichnis als Schlüssel gespeichert. Laufzeitaufrufer verwenden
  `ensureOpenClawModelCatalog`; im Laufzeitcode gibt es keine `models.json`-Kompatibilitäts-API.
  Die Implementierung schreibt in SQLite, und die eingebettete PI-Registry wird
  aus dieser gespeicherten Nutzlast initialisiert, ohne eine `models.json`-Datei zu erstellen.
- Der optionale `memory.qmd.sessions`-Export liest kanonische Transkriptzeilen aus
  der agentenspezifischen Datenbank und materialisiert bereinigtes Markdown im QMD-Basisverzeichnis
  als explizites QMD-Eingabeartefakt. QMD-Sitzungssammlungen und Zuordnungen von
  Artefaktidentitäten bleiben daher Teil der konfigurierten Brücke zum externen Tool;
  sie sind kein zweiter kanonischer Transkriptspeicher.
- QMDs eigene `index.sqlite`, YAML-Sammlungskonfiguration und Modelldownloads bleiben
  Artefakte des externen Tools unter `~/.openclaw/agents/<agentId>/qmd`; sie werden nicht
  nach `plugin_blob_entries` gespiegelt. Die OpenClaw-eigene QMD-Koordination ist
  datenbankzentriert: gemeinsam genutzte `state_leases` serialisieren Einbettungen global und agentenspezifische
  `state_leases` serialisieren Schreibvorgänge für Sammlung/Aktualisierung/Einbettung. Die Laufzeit erstellt keine
  QMD-Sperr-Begleitdateien.
- Das optionale Plugin `memory-lancedb` erstellt
  `~/.openclaw/memory/lancedb` nicht mehr als impliziten, von OpenClaw verwalteten Speicher. Es ist ein
  externes LanceDB-Backend und bleibt deaktiviert, bis der Betreiber einen
  expliziten `dbPath` konfiguriert.
- `check:database-first-legacy-stores` schlägt bei neuem Laufzeitquellcode fehl, der
  veraltete Speichernamen mit schreibenden Dateisystem-APIs kombiniert. Die Prüfung schlägt außerdem bei Laufzeit-
  quellcode fehl, der die eingestellten Transkriptbrücken-Markierungen
  `transcriptLocator` oder `sqlite-transcript://...` wieder einführt. Migrations-, Doctor-, Import-
  und expliziter sitzungsfremder Exportcode bleiben zulässig. Allgemeinere Namen veralteter Verträge
  wie `sessionFile`, `storePath` und alte `SessionManager`-Fassaden aus der Dateiära
  haben weiterhin aktuelle Eigentümer und benötigen separate Schutzmaßnahmen für die Migration,
  bevor sie zu einer erforderlichen Vorabprüfung werden können. Die Schutzprüfung deckt jetzt außerdem
  laufzeitbezogene `cache/*.json`-Speicher, generische
  `thread-bindings.json`-Begleitdateien, Cron-Zustands-/Ausführungsprotokoll-JSON, Konfigurationszustands-JSON,
  Neustart- und Sperr-Begleitdateien, Voice-Wake-Einstellungen, Genehmigungen für Plugin-Bindungen,
  das JSON-Verzeichnis installierter Plugins, File-Transfer-Audit-JSONL, Memory-Wiki-Aktivitäts-
  protokolle, das alte gebündelte `command-logger`-Textprotokoll und pi-mono-Konfigurationsoptionen für
  JSONL-Rohstromdiagnosen ab. Sie verbietet außerdem alte, auf Stammebene liegende Namen veralteter Doctor-Module,
  sodass Kompatibilitätscode unter `src/commands/doctor/` verbleibt. Android-Debug-Handler
  verwenden außerdem logcat-/speicherinterne Ausgaben, statt `camera_debug.log`- oder
  `debug_logs.txt`-Cache-Dateien zwischenzuspeichern.

## Form des Zielschemas

Halten Sie Schemas explizit. Vom Host verwalteter Laufzeitstatus verwendet typisierte Tabellen. Plugin-eigener
opaker Status verwendet `plugin_state_entries` / `plugin_blob_entries`; es gibt keine
generische Host-Tabelle `kv`.

Globale Datenbank:

```text
state_leases(scope, lease_key, owner, expires_at, heartbeat_at, payload_json, created_at, updated_at)
exec_approvals_config(config_key, raw_json, socket_path, has_socket_token, default_security, default_ask, default_ask_fallback, auto_allow_skills, agent_count, allowlist_count, updated_at_ms)
schema_meta(meta_key, role, schema_version, agent_id, app_version, created_at, updated_at)
agent_databases(agent_id, path, schema_version, last_seen_at, size_bytes)
task_runs(...)
task_delivery_state(...)
flow_runs(...)
subagent_runs(run_id, child_session_key, requester_session_key, controller_session_key, created_at, ended_at, cleanup_handled, payload_json)
current_conversation_bindings(binding_key, binding_id, target_agent_id, target_session_id, target_session_key, channel, account_id, conversation_kind, parent_conversation_id, conversation_id, target_kind, status, bound_at, expires_at, metadata_json, updated_at)
plugin_binding_approvals(plugin_root, channel, account_id, plugin_id, plugin_name, approved_at)
tui_last_sessions(scope_key, session_key, updated_at)
plugin_state_entries(plugin_id, namespace, entry_key, value_json, created_at, expires_at)
plugin_blob_entries(plugin_id, namespace, entry_key, metadata_json, blob, created_at, expires_at)
media_blobs(subdir, id, content_type, size_bytes, blob, created_at, updated_at)
skill_uploads(upload_id, kind, slug, force, size_bytes, sha256, actual_sha256, received_bytes, archive_blob, created_at, expires_at, committed, committed_at, idempotency_key_hash)
skill_upload_chunks(upload_id, byte_offset, size_bytes, chunk_blob)
web_push_subscriptions(endpoint_hash, subscription_id, endpoint, p256dh, auth, created_at_ms, updated_at_ms)
web_push_vapid_keys(key_id, public_key, private_key, subject, updated_at_ms)
apns_registrations(node_id, transport, token, relay_handle, send_grant, installation_id, relay_origin, topic, environment, distribution, token_debug_suffix, updated_at_ms)
apns_registration_tombstones(node_id, deleted_at_ms)
node_host_config(config_key, version, node_id, token, display_name, gateway_host, gateway_port, gateway_tls, gateway_tls_fingerprint, gateway_context_path, updated_at_ms)
device_identities(identity_key, device_id, public_key_pem, private_key_pem, created_at_ms, updated_at_ms)
device_auth_tokens(device_id, role, token, scopes_json, updated_at_ms)
macos_port_guardian_records(pid, port, command, mode, timestamp)
workspace_setup_state(workspace_key, workspace_path, version, bootstrap_seeded_at, setup_completed_at, updated_at)
workspace_path_aliases(alias_key, alias_path, workspace_key, workspace_path, updated_at_ms)
workspace_attestations(workspace_key, attested_at_ms, updated_at_ms)
workspace_generated_bootstrap_hashes(workspace_key, filename, sha256)
native_hook_relay_bridges(relay_id, pid, hostname, port, token, expires_at_ms, updated_at_ms)
model_capability_cache(provider_id, model_id, name, input_text, input_image, reasoning, supports_tools, context_window, max_tokens, cost_input, cost_output, cost_cache_read, cost_cache_write, updated_at_ms)
agent_model_catalogs(catalog_key, agent_dir, raw_json, updated_at)
managed_outgoing_image_records(attachment_id, session_key, agent_id, message_id, created_at, updated_at, retention_class, alt, original_media_id, original_media_subdir, original_content_type, original_width, original_height, original_size_bytes, original_filename, record_json, cleanup_pending)
gateway_restart_sentinel(sentinel_key, version, kind, status, ts, session_key, thread_id, delivery_channel, delivery_to, delivery_account_id, message, continuation_json, doctor_hint, stats_json, payload_json, updated_at_ms)
channel_pairing_requests(channel_key, account_id, request_id, code, created_at, last_seen_at, meta_json)
channel_pairing_allow_entries(channel_key, account_id, entry, sort_order, updated_at)
voicewake_triggers(config_key, position, trigger, updated_at_ms)
voicewake_routing_config(config_key, version, default_target_mode, default_target_agent_id, default_target_session_key, updated_at_ms)
voicewake_routing_routes(config_key, position, trigger, target_mode, target_agent_id, target_session_key, updated_at_ms)
update_check_state(state_key, last_checked_at, last_notified_version, last_notified_tag, last_available_version, last_available_tag, auto_install_id, auto_first_seen_version, auto_first_seen_tag, auto_first_seen_at, auto_last_attempt_version, auto_last_attempt_at, auto_last_success_version, auto_last_success_at, updated_at_ms)
config_health_entries(config_path, last_known_good_json, last_promoted_good_json, last_observed_suspicious_signature, updated_at_ms)
sandbox_registry_entries(registry_kind, container_name, session_key, backend_id, runtime_label, image, created_at_ms, last_used_at_ms, config_label_kind, config_hash, cdp_port, no_vnc_port, entry_json, updated_at)
cron_jobs(store_key, job_id, name, description, enabled, delete_after_run, created_at_ms, agent_id, session_key, schedule_kind, schedule_expr, schedule_tz, every_ms, anchor_ms, at, stagger_ms, session_target, wake_mode, payload_kind, payload_message, payload_model, payload_fallbacks_json, payload_thinking, payload_timeout_seconds, payload_allow_unsafe_external_content, payload_external_content_source_json, payload_light_context, payload_tools_allow_json, delivery_mode, delivery_channel, delivery_to, delivery_thread_id, delivery_account_id, delivery_best_effort, failure_delivery_mode, failure_delivery_channel, failure_delivery_to, failure_delivery_account_id, failure_alert_disabled, failure_alert_after, failure_alert_channel, failure_alert_to, failure_alert_cooldown_ms, failure_alert_include_skipped, failure_alert_mode, failure_alert_account_id, next_run_at_ms, running_at_ms, last_run_at_ms, last_run_status, last_error, last_duration_ms, consecutive_errors, consecutive_skipped, schedule_error_count, last_delivery_status, last_delivery_error, last_delivered, last_failure_alert_at_ms, job_json, state_json, runtime_updated_at_ms, schedule_identity, sort_order, updated_at)
delivery_queue_entries(queue_name, id, status, entry_kind, session_key, channel, target, account_id, retry_count, last_attempt_at, last_error, recovery_state, platform_send_started_at, entry_json, enqueued_at, updated_at, failed_at)
commitments(id, agent_id, session_key, channel, account_id, recipient_id, thread_id, sender_id, kind, sensitivity, source, status, reason, suggested_text, dedupe_key, confidence, due_earliest_ms, due_latest_ms, due_timezone, source_message_id, source_run_id, created_at_ms, updated_at_ms, attempts, last_attempt_at_ms, sent_at_ms, dismissed_at_ms, snoozed_until_ms, expired_at_ms, record_json)
migration_runs(id, started_at, finished_at, status, report_json)
migration_sources(source_key, migration_kind, source_path, target_table, source_sha256, source_size_bytes, source_record_count, last_run_id, status, imported_at, removed_source, report_json)
backup_runs(id, created_at, archive_path, status, manifest_json)
```

Agent-Datenbank:

```text
schema_meta(meta_key, role, schema_version, agent_id, app_version, created_at, updated_at)
sessions(session_id, session_key, session_scope, created_at, updated_at, started_at, ended_at, status, chat_type, channel, account_id, primary_conversation_id, model_provider, model, agent_harness_id, parent_session_key, spawned_by, display_name)
conversations(conversation_id, channel, account_id, kind, peer_id, parent_conversation_id, thread_id, native_channel_id, native_direct_user_id, label, metadata_json, created_at, updated_at)
session_conversations(session_id, conversation_id, role, first_seen_at, last_seen_at)
session_routes(session_key, session_id, updated_at)
session_entries(session_id, session_key, entry_json, updated_at)
transcript_events(session_id, seq, event_json, created_at)
transcript_event_identities(session_id, event_id, seq, event_type, has_parent, parent_id, message_idempotency_key, created_at)
transcript_snapshots(session_id, snapshot_id, reason, event_count, created_at, metadata_json)
vfs_entries(namespace, path, kind, content_blob, metadata_json, updated_at)
tool_artifacts(run_id, artifact_id, kind, metadata_json, blob, created_at)
run_artifacts(run_id, path, kind, metadata_json, blob, created_at)
trajectory_runtime_events(session_id, run_id, seq, event_json, created_at)
memory_index_meta(key, value)
memory_index_sources(id, path, source, hash, mtime, size)
memory_index_chunks(id, path, source, start_line, end_line, hash, model, text, embedding, updated_at)
memory_embedding_cache(provider, model, provider_key, hash, embedding, dims, updated_at)
memory_index_state(id, revision)
cache_entries(scope, key, value_json, blob, expires_at, updated_at)
```

`memory_index_sources.id` ist der stabile ganzzahlige Primärschlüssel; `(path, source)` bleibt eindeutig.

Eine zukünftige Suche kann FTS-Tabellen hinzufügen, ohne die kanonischen Ereignistabellen zu ändern:

```text
transcript_events_fts(session_id, seq, text)
vfs_entries_fts(namespace, path, text)
```

Große Werte sollten `blob`-Spalten statt einer JSON-Zeichenfolgenkodierung verwenden. Behalten Sie
`value_json` für kleine strukturierte Daten bei, die mit einfachen
SQLite-Werkzeugen einsehbar bleiben müssen.

`agent_databases` ist die kanonische Registry für diesen Branch. Fügen Sie keine
Tabelle `agents` hinzu, bis ein tatsächlicher Eigentümer für Agent-Datensätze existiert; die Agent-Konfiguration verbleibt in
`openclaw.json`.

## Form der Doctor-Migration

Doctor sollte einen expliziten Migrationsschritt aufrufen, der protokollierbar ist und
gefahrlos erneut ausgeführt werden kann:

```bash
openclaw doctor --fix
```

`openclaw doctor --fix` ruft die Implementierung der Statusmigration nach
der regulären Konfigurationsvorprüfung auf und erstellt vor dem Import eine verifizierte Sicherung. Der Start der Laufzeit
und `openclaw migrate` dürfen keine alten OpenClaw-Statusdateien importieren.

Migrationseigenschaften:

- Ein Migrationsdurchlauf ermittelt alle alten Dateiquellen und erstellt einen Plan,
  bevor Änderungen vorgenommen werden.
- Doctor erstellt vor dem Import
  alter Dateien ein verifiziertes Sicherungsarchiv für den Zustand vor der Migration.
- Importe sind idempotent und werden anhand von Quellpfad, mtime, Größe, Hash und Zieltabelle
  identifiziert.
- Erfolgreich verarbeitete Quelldateien werden entfernt oder archiviert, nachdem die Zieldatenbank
  den Commit abgeschlossen hat.
- Fehlgeschlagene Importe lassen die Quelle unverändert und zeichnen eine Warnung in
  `migration_runs` auf.
- Der Laufzeitcode liest ausschließlich SQLite, nachdem die Migration vorhanden ist.
- Ein Downgrade- oder Exportpfad zurück zu Laufzeitdateien ist nicht erforderlich.

## Migrationsinventar

Verschieben Sie Folgendes in die globale Datenbank:

- Laufzeitschreibvorgänge der Aufgabenregistrierung verwenden jetzt die gemeinsame Datenbank; der nicht ausgelieferte
  `tasks/runs.sqlite`-Sidecar-Importer wurde gelöscht. Snapshot-Speicherungen führen Upserts anhand der Aufgaben-ID
  aus und löschen nur fehlende Aufgaben-/Auslieferungszeilen.
- Laufzeitschreibvorgänge von Task Flow verwenden jetzt die gemeinsame Datenbank; der nicht ausgelieferte
  `tasks/flows/registry.sqlite`-Sidecar-Importer wurde gelöscht. Snapshot-Speicherungen
  führen Upserts anhand der Flow-ID aus und löschen nur fehlende Flow-Zeilen.
- Laufzeitschreibvorgänge des Plugin-Zustands verwenden jetzt die gemeinsame Datenbank; der nicht ausgelieferte
  `plugin-state/state.sqlite`-Sidecar-Importer wurde gelöscht.
- Die integrierte Speichersuche verwendet nicht mehr standardmäßig `memory/<agentId>.sqlite`; ihre
  Indextabellen befinden sich in der Datenbank des zuständigen Agenten, und die explizite
  Aktivierung des `memorySearch.store.path`-Sidecars wurde durch die Doctor-Konfigurationsmigration
  abgeschafft.
- Die integrierte Neuindizierung des Speichers setzt nur speichereigene Tabellen in der Agentendatenbank zurück.
  Sie darf nicht die gesamte SQLite-Datei ersetzen, da dieselbe Datenbank
  Sitzungen, Transkripte, VFS-Zeilen, Artefakte und Laufzeit-Caches enthält.
- Sandbox-Container-/Browserregistrierungen aus monolithischem und aufgeteiltem JSON. Laufzeitschreibvorgänge
  verwenden jetzt die gemeinsame Datenbank; der Import von Legacy-JSON bleibt bestehen.
- Cron-Aufgabendefinitionen, Zeitplanzustand und Ausführungsverlauf verwenden jetzt gemeinsames SQLite;
  doctor importiert/entfernt die Legacy-Dateien `jobs.json`, `jobs-state.json` und
  `cron/runs/*.jsonl`
- Geräteidentität/-authentifizierung, Push, Aktualisierungsprüfung, Verpflichtungen, OpenRouter-Modell-
  Cache, Index installierter Plugins und App-Server-Bindungen
- Geräte-/Node-Kopplungs- und Bootstrap-Datensätze verwenden jetzt typisierte SQLite-Tabellen
- Abonnenten von Benachrichtigungen zur Gerätekopplung und Markierungen zugestellter Anfragen verwenden jetzt die
  gemeinsame SQLite-Tabelle für den Plugin-Zustand anstelle von `device-pair-notify.json`.
- Anrufdatensätze für Sprachanrufe verwenden jetzt die gemeinsame SQLite-Tabelle für den Plugin-Zustand im
  Namensraum `voice-call` / `calls` anstelle von `calls.jsonl`; die Plugin-CLI
  verfolgt und fasst den SQLite-gestützten Anrufverlauf zusammen.
- QQBot-Gateway-Sitzungen, Datensätze bekannter Benutzer und der Ref-Index-Zitat-Cache verwenden jetzt
  den SQLite-Plugin-Zustand in `qqbot`-Namensräumen (`gateway-sessions`,
  `known-users`, `ref-index`) anstelle von `session-*.json`, `known-users.json`
  und `ref-index.jsonl`. Diese Legacy-Dateien sind Caches und werden nicht migriert.
- Discord-Modellauswahlpräferenzen, Hashes der Befehlsbereitstellung und Thread-Bindungen
  verwenden jetzt den SQLite-Plugin-Zustand in `discord`-Namensräumen
  (`model-picker-preferences`, `command-deploy-hashes`, `thread-bindings`)
  anstelle von `model-picker-preferences.json`, `command-deploy-cache.json` und
  `thread-bindings.json`; die Discord-doctor-/Einrichtungsmigration importiert und
  entfernt die Legacy-Dateien.
- BlueBubbles-Aufholcursor und Markierungen zur Deduplizierung eingehender Daten verwenden jetzt den SQLite-Plugin-
  Zustand in `bluebubbles`-Namensräumen (`catchup-cursors`, `inbound-dedupe`)
  anstelle von `bluebubbles/catchup/*.json` und
  `bluebubbles/inbound-dedupe/*.json`; die BlueBubbles-doctor-/Einrichtungsmigration
  importiert und entfernt die Legacy-Dateien.
- Telegram-Aktualisierungs-Offsets, Sticker-Cache-Einträge, Cache-Einträge für Nachrichten in Antwortketten,
  Cache-Einträge gesendeter Nachrichten, Cache-Einträge für Themennamen und Thread-
  Bindungen verwenden jetzt den SQLite-Plugin-Zustand in `telegram`-Namensräumen
  (`update-offsets`, `sticker-cache`, `message-cache`, `sent-messages`,
  `topic-names`, `thread-bindings`) anstelle von `update-offset-*.json`,
  `sticker-cache.json`, `*.telegram-messages.json`,
  `*.telegram-sent-messages.json`, `*.telegram-topic-names.json` und
  `thread-bindings-*.json`; die Telegram-doctor-/Einrichtungsmigration importiert und
  entfernt die Legacy-Dateien.
- iMessage-Aufholcursor, Zuordnungen kurzer Antwort-IDs und Deduplizierungszeilen gesendeter Echos
  verwenden jetzt den SQLite-Plugin-Zustand in `imessage`-Namensräumen (`catchup-cursors`,
  `reply-cache`, `sent-echoes`) anstelle von `imessage/catchup/*.json`,
  `imessage/reply-cache.jsonl` und `imessage/sent-echoes.jsonl`; die iMessage-
  doctor-/Einrichtungsmigration importiert und entfernt die Legacy-Dateien.
- Microsoft Teams-Unterhaltungen, Umfragen, SSO-Token und Feedback-Erkenntnisse
  verwenden jetzt Namensräume des SQLite-Plugin-Zustands (`conversations`, `polls`, `sso-tokens`,
  `feedback-learnings`) anstelle von `msteams-conversations.json`,
  `msteams-polls.json`, `msteams-sso-tokens.json` und `*.learnings.json`; die
  Microsoft Teams-doctor-/Einrichtungsmigration importiert und archiviert die Legacy-Dateien.
  Ausstehende Uploads sind ein kurzlebiger SQLite-Cache, und alte JSON-Cache-Dateien werden
  nicht migriert.
- Matrix-Synchronisierungs-Cache, Speichermetadaten, Thread-Bindungen, Markierungen zur Deduplizierung eingehender Daten,
  Abklingzustand der Startüberprüfung, Anmeldedaten, Wiederherstellungsschlüssel und kryptografische SDK-
  IndexedDB-Snapshots verwenden jetzt Namensräume für SQLite-Plugin-Zustand/-Blobs unter
  `matrix` (`sync-store`, `storage-meta`, `thread-bindings`,
  `matrix.inbound-dedupe.*` über die beanspruchbare Kerndeduplizierung,
  `startup-verification`, `credentials`, `recovery-key`, `idb-snapshots`)
  anstelle von `bot-storage.json`, `storage-meta.json`, `thread-bindings.json`,
  `inbound-dedupe.json`, `startup-verification.json`, `credentials.json`,
  `recovery-key.json` und `crypto-idb-snapshot.json`; die Matrix-doctor-/Einrichtungsmigration
  importiert und entfernt diese Legacy-Dateien (sowie die stillgelegten wurzelspezifischen
  SQLite-Zeilen `inbound-dedupe`) aus kontobezogenen Matrix-Speicherwurzeln.
- Nostr-Bus-Cursor und der Veröffentlichungszustand von Profilen verwenden jetzt den SQLite-Plugin-Zustand in
  `nostr`-Namensräumen (`bus-state`, `profile-state`) anstelle von
  `bus-state-*.json` und `profile-state-*.json`; die Nostr-doctor-/Einrichtungsmigration
  importiert und entfernt die Legacy-Dateien.
- Active Memory-Sitzungsumschalter verwenden jetzt den SQLite-Plugin-Zustand unter
  `active-memory/session-toggles` anstelle von `session-toggles.json`.
- Vorschlagswarteschlangen und Überprüfungszähler von Skill Workshop verwenden jetzt den SQLite-Plugin-Zustand
  unter `skill-workshop/proposals` und `skill-workshop/reviews` anstelle von
  arbeitsbereichsspezifischen `skill-workshop/<workspace>.json`-Dateien.
- Warteschlangen für ausgehende Zustellungen und Sitzungszustellungen verwenden jetzt gemeinsam die globale SQLite-
  Tabelle `delivery_queue_entries` unter separaten Warteschlangennamen
  (`outbound-delivery`, `session-delivery`) anstelle dauerhafter
  Dateien `delivery-queue/*.json`, `delivery-queue/failed/*.json` und
  `session-delivery-queue/*.json`. Der doctor-Schritt für den Legacy-Zustand importiert
  ausstehende und fehlgeschlagene Zeilen, entfernt veraltete Zustellmarkierungen und löscht nach dem Import die alten
  JSON-Dateien. Häufig verwendete Routing- und Wiederholungsfelder sind typisierte Spalten; die
  JSON-Nutzlast bleibt nur für Wiedergabe/Debugging erhalten.
- ACPX-Prozess-Leases verwenden jetzt den SQLite-Plugin-Zustand unter `acpx/process-leases`
  anstelle von `process-leases.json`.
- Metadaten zu Sicherungs- und Migrationsläufen

Diese in Agentendatenbanken verschieben:

- Wurzeleinträge von Agentensitzungen und kompatibilitätsgeformte Nutzlasten von Sitzungseinträgen. Für
  Laufzeitschreibvorgänge abgeschlossen: Häufig verwendete Sitzungsmetadaten können in `sessions` abgefragt werden, während die
  vollständige Legacy-geformte Nutzlast `SessionEntry` in `session_entries` verbleibt.
- Agenten-Transkriptereignisse. Für Laufzeitschreibvorgänge abgeschlossen.
- Compaction-Prüfpunkte und Transkript-Snapshots. Für Laufzeitschreibvorgänge abgeschlossen:
  Transkriptkopien von Prüfpunkten sind SQLite-Transkriptzeilen, und Prüfpunkt-
  Metadaten werden in `transcript_snapshots` erfasst. Gateway-Prüfpunkt-Hilfsfunktionen
  bezeichnen diese Werte jetzt als Transkript-Snapshots statt als Quelldateien.
- Scratch-/Arbeitsbereichs-Namensräume des Agenten-VFS. Für Laufzeit-VFS-Schreibvorgänge abgeschlossen.
- Nutzlasten von Subagent-Anhängen. Für Laufzeitschreibvorgänge abgeschlossen: Sie sind SQLite-VFS-
  Starteinträge und niemals dauerhafte Arbeitsbereichsdateien.
- Werkzeugartefakte. Für Laufzeitschreibvorgänge abgeschlossen.
- Ausführungsartefakte. Für Laufzeitschreibvorgänge von Workern über die agentenspezifische
  Tabelle `run_artifacts` abgeschlossen.
- Agentenlokale Laufzeit-Caches. Für bereichsbezogene Laufzeit-Cache-Schreibvorgänge von Workern über
  die agentenspezifische Tabelle `cache_entries` abgeschlossen. Gateway-weite Modell-Caches verbleiben in der
  globalen Datenbank, sofern sie nicht agentenspezifisch werden.
- Protokolle übergeordneter ACP-Streams. Für Laufzeitschreibvorgänge abgeschlossen.
- Sitzungen des ACP-Wiedergabeledgers. Für Laufzeitschreibvorgänge über
  `acp_replay_sessions` und `acp_replay_events` abgeschlossen; das Legacy-Element `acp/event-ledger.json`
  bleibt nur als doctor-Eingabe bestehen.
- ACP-Sitzungsmetadaten. Für Laufzeitschreibvorgänge über `acp_sessions` abgeschlossen; Legacy-
  Blöcke `entry.acp` in `sessions.json` dienen nur als Eingabe für die doctor-Migration.
- Trajektorien-Sidecars, wenn es sich nicht um explizite Exportdateien handelt. Für Laufzeit-
  schreibvorgänge abgeschlossen: Die Trajektorienerfassung schreibt `trajectory_runtime_events`-Zeilen in die Agentendatenbank
  und spiegelt ausführungsbezogene Artefakte in SQLite. Legacy-Sidecars dienen nur als doctor-
  Importeingaben; der Export kann neue JSONL-Ausgaben für Support-Bundles materialisieren,
  liest oder migriert jedoch alte Trajektorien-/Transkript-Sidecars nicht zur Laufzeit.
  Die Laufzeit-Trajektorienerfassung stellt den SQLite-Bereich bereit; JSONL-Pfadhilfsfunktionen sind
  auf Export-/Debugging-Unterstützung beschränkt und werden nicht erneut aus dem Laufzeitmodul exportiert.
  Trajektorienmetadaten des eingebetteten Runners erfassen die Identität `{agentId, sessionId, sessionKey}`,
  anstatt einen Transkript-Locator dauerhaft zu speichern.

Diese vorerst dateibasiert belassen:

- `openclaw.json`
- Provider- oder CLI-Anmeldedatendateien
- Plugin-/Paketmanifeste
- Benutzerarbeitsbereiche und Git-Repositorys, wenn der Festplattenmodus ausgewählt ist
- für die Nachverfolgung durch Operatoren vorgesehene Protokolle, sofern keine bestimmte Protokolloberfläche verschoben wird

## Migrationsplan

### Phase 0: Grenze festschreiben

Die Grenze für dauerhaften Zustand explizit festlegen, bevor weitere Zeilen verschoben werden:

- Eine Tabelle `migration_runs` zur globalen Datenbank hinzufügen.
  Für Ausführungsberichte der Legacy-Zustandsmigration abgeschlossen.
- Einen einzelnen doctor-eigenen Zustandsmigrationsdienst für den Import von Dateien in die Datenbank hinzufügen.
  Abgeschlossen: `openclaw doctor --fix` verwendet die Implementierung der Legacy-Zustandsmigration.
- `plan` schreibgeschützt machen und `apply` eine Sicherung erstellen, importieren und überprüfen sowie
  anschließend alte Dateien löschen oder unter Quarantäne stellen lassen.
  Abgeschlossen: doctor erstellt eine verifizierte Sicherung vor der Migration, übergibt den Sicherungspfad
  an `migration_runs` und verwendet die Importer-/Entfernungspfade erneut.
- Statische Verbote hinzufügen, damit neuer Laufzeitcode keine Legacy-Zustandsdateien schreiben kann,
  während Migrationscode und Tests sie weiterhin anlegen/lesen können.
  Für die derzeit migrierten Legacy-Speicher abgeschlossen; die Schutzprüfung durchsucht außerdem verschachtelte
  Tests nach verbotenen Laufzeitverträgen für Transkript-Locators.

### Phase 1: Globale Steuerungsebene abschließen

Gemeinsamen Koordinierungszustand in `state/openclaw.sqlite` belassen:

- Agenten und Registrierung der Agentendatenbanken
- Ledger für Aufgaben und Task Flow
- Plugin-Zustand
- Sandbox-Container-/Browserregistrierung
- Ausführungsverlauf von Cron/Zeitplaner
- Kopplung, Gerät, Push, Aktualisierungsprüfung, TUI, OpenRouter-/Modell-Caches und anderer
  kleiner Gateway-bezogener Laufzeitzustand
- Sicherungs- und Migrationsmetadaten
- Bytes von Gateway-Medienanhängen. Für Laufzeitschreibvorgänge abgeschlossen; direkte Dateipfade
  sind temporäre Materialisierungen zur Kompatibilität mit Kanalabsendern und Sandbox-
  Staging. Laufzeit-Zulassungslisten akzeptieren SQLite-Materialisierungspfade, nicht Legacy-
  Stammverzeichnisse für Zustands-/Konfigurationsmedien. doctor importiert Legacy-Mediendateien in
  `media_blobs` und entfernt die Quelldateien nach erfolgreichen Zeilenschreibvorgängen.
- Debug-Proxy-Erfassungssitzungen, Ereignisse und Nutzlast-Blobs. Abgeschlossen: Erfassungen befinden sich
  in der gemeinsamen Zustandsdatenbank und werden über Bootstrap, Schema,
  WAL- und Busy-Timeout-Einstellungen der gemeinsamen Zustandsdatenbank geöffnet. Nutzlastbytes werden in
  `capture_blobs.data` gzip-komprimiert; es gibt keine Laufzeitüberschreibung einer Sidecar-Datenbank für den Debug-Proxy,
  kein Blob-Verzeichnis und kein ausschließlich für Proxy-Erfassungen generiertes Schema-/Codegen-Ziel.
  Die doctor-/Startmigration importiert ausgelieferte `debug-proxy/capture.sqlite`-Zeilen
  und referenzierte Nutzlast-Blobs einschließlich aktiver Legacy-Umgebungsüberschreibungen für Datenbank/Blobs
  und archiviert anschließend diese Quellen, während CA-Zertifikate unverändert bleiben.

Diese Phase löscht außerdem doppelte Sidecar-Öffnungsfunktionen, Berechtigungshelfer, die WAL-
Einrichtung, Dateisystembereinigung und Kompatibilitäts-Writer aus diesen Subsystemen.

### Phase 2: Datenbanken pro Agent einführen

Erstellen Sie eine Datenbank pro Agent und registrieren Sie sie in der globalen DB:

```text
~/.openclaw/state/openclaw.sqlite
~/.openclaw/agents/<agentId>/agent/openclaw-agent.sqlite
```

Die globale `agent_databases`-Zeile speichert Pfad, Schemaversion, Zeitstempel
der letzten Sichtung sowie grundlegende Metadaten zu Größe und Integrität. Der Laufzeitcode fragt die Registry nach
der Agent-DB, statt Dateipfade direkt abzuleiten.

Die Agent-DB verwaltet:

- `sessions` als kanonischen Sitzungsstamm, wobei `session_entries` als
  Nutzdatentabelle mit Kompatibilitätsstruktur an diesen Stamm angehängt ist und
  `session_routes` als eindeutige aktive `session_key`-Suche dient
- `conversations` und `session_conversations` als normalisierte Provider-
  Routing-Identität, die Sitzungen zugeordnet ist
- `transcript_events`
- Transkript-Snapshots und Compaction-Prüfpunkte. Für Laufzeitschreibvorgänge abgeschlossen.
- `vfs_entries`
- `tool_artifacts` und Ausführungsartefakte
- agent-lokale Laufzeit-/Cache-Zeilen. Für Worker-spezifische Caches abgeschlossen.
- Ereignisse des übergeordneten ACP-Streams
- Trajektorien-Laufzeitereignisse, sofern sie keine expliziten Exportartefakte sind

### Phase 3: Sitzungs-Store-APIs ersetzen

Für die Laufzeit abgeschlossen. Die dateiförmige Oberfläche des Sitzungs-Stores ist kein aktiver
Laufzeitvertrag:

- Die Laufzeit ruft `loadSessionStore(storePath)` nicht mehr auf und behandelt `storePath` nicht mehr als
  Sitzungsidentität.
- Laufzeit-Zeilenoperationen sind `getSessionEntry`, `upsertSessionEntry`,
  `patchSessionEntry`, `deleteSessionEntry` und `listSessionEntries`.
- Hilfsfunktionen zum Neuschreiben des gesamten Stores, Datei-Writer, Warteschlangentests, Alias-Bereinigung und
  Parameter zum Löschen veralteter Schlüssel wurden aus der Laufzeit entfernt.
- Veraltete Kompatibilitätsexporte des Root-Pakets delegieren bis zum 2026-10-12 an den ausschließlich für Doctor vorgesehenen
  `sessions.json`-Importer; Kompatibilitätslesevorgänge des Plugin SDK
  projizieren weiterhin kanonische SQLite-Zeilen.
- Das Parsen von `sessions.json` verbleibt nur im Doctor-Migrations-/Importcode und
  in Doctor-Tests.
- Der Laufzeit-Lebenszyklus-Fallback liest SQLite-Transkript-Header und nicht die ersten
  JSONL-Zeilen.

Löschen Sie weiterhin alles, was Parameter für Dateisperren,
Vokabular für Bereinigung/Kürzung als Dateiwartung, Store-Pfade als Identität oder Tests
wiedereinführt, deren einzige Zusicherung die JSON-Persistenz ist.

### Phase 4: Transkripte, ACP-Streams, Trajektorien und VFS verschieben

Machen Sie jeden Agent-Datenstrom datenbanknativ:

- Transkript-Anfügevorgänge laufen über eine einzige SQLite-Transaktion, die den
  Sitzungs-Header sicherstellt, die Nachrichtenidempotenz prüft, das übergeordnete Ende auswählt, Daten
  in `transcript_events` einfügt und abfragbare Identitätsmetadaten in
  `transcript_event_identities` aufzeichnet. Für direkte Anfügevorgänge von Transkriptnachrichten und
  normale persistierte `TranscriptSessionManager`-Anfügevorgänge abgeschlossen; explizite Verzweigungs-
  operationen behalten ihre explizite Auswahl des übergeordneten Elements bei und schreiben weiterhin SQLite-Zeilen,
  ohne einen Dateilokator abzuleiten.
- Protokolle übergeordneter ACP-Streams werden zu Zeilen statt zu `.acp-stream.jsonl`-Dateien. Abgeschlossen.
- Die ACP-Spawn-Einrichtung persistiert keine Transkript-JSONL-Pfade mehr. Abgeschlossen.
- Die Trajektorienerfassung der Laufzeit schreibt Ereigniszeilen/Artefakte direkt. Der explizite
  Support-/Exportbefehl kann weiterhin Support-Bundle-JSONL-Artefakte als
  Exportformat erzeugen, aber der Sitzungsexport erstellt keine Sitzungs-JSONL-Dateien neu. Abgeschlossen.
- Datenträger-Workspaces verbleiben auf dem Datenträger, wenn der Datenträgermodus konfiguriert ist.
- VFS-Scratch und der experimentelle reine VFS-Workspace-Modus verwenden die Agent-DB.

Die Migration importiert alte JSONL-Dateien einmalig, zeichnet Anzahlen/Hashes in
`migration_runs` auf und entfernt importierte Dateien nach Integritätsprüfungen.

### Phase 5: Sicherung, Wiederherstellung, Vacuum und Verifizierung

Sicherungen bleiben eine einzelne Archivdatei:

- Führen Sie für jede globale und jede Agent-Datenbank einen SQLite-/WAL-Checkpoint durch.
- Erstellen Sie von jeder DB einen Snapshot mit der SQLite-Online-Sicherung und anschließendem Offline-`VACUUM`.
- Archivieren Sie kompakte DB-Snapshots, Konfiguration, externe Zugangsdaten und angeforderte
  Workspace-Exporte.
- Lassen Sie unverarbeitete Live-Dateien `*.sqlite-wal` und `*.sqlite-shm` aus.
- Verifizieren Sie, indem Sie jeden DB-Snapshot öffnen und `PRAGMA integrity_check` ausführen.
  `openclaw backup create` führt diese Archivverifizierung standardmäßig durch;
  `--no-verify` überspringt nur den Archivdurchlauf nach dem Schreiben, nicht die Integritätsprüfung
  bei der Snapshot-Erstellung.
- Die Wiederherstellung kopiert Snapshots zurück an ihre Zielpfade. Wiederhergestellte globale DBs verwenden
  Version `1`; wiederhergestellte Agent-DBs verwenden Version `2`, wobei Snapshots der Version `1`
  beim Öffnen atomar aktualisiert werden.

### Phase 6: Worker-Laufzeit

Behalten Sie den Worker-Modus als experimentell bei, während die Datenbankaufteilung umgesetzt wird:

- Worker erhalten Agent-ID, Ausführungs-ID, Dateisystemmodus und DB-Registry-Identität.
- Jeder Worker öffnet seine eigene SQLite-Verbindung.
- Der übergeordnete Prozess behält die Autorität über Kanalauslieferung, Genehmigungen, Konfiguration und Abbruch.
- Beginnen Sie mit einem Worker pro aktiver Ausführung; fügen Sie Pooling erst hinzu, nachdem Lebenszyklus und Eigentümerschaft der DB-
  Verbindung stabil sind.

### Phase 7: Die alte Welt löschen

Für die Laufzeit-Sitzungsverwaltung abgeschlossen. Die alte Welt ist nur als explizite
Doctor-Eingabe oder Support-/Exportausgabe zulässig:

- Keine Laufzeitschreibvorgänge für `sessions.json`, Transkript-JSONL, Sandbox-Registry-JSON, Task-
  Sidecar-SQLite oder Plugin-Status-Sidecar-SQLite.
- Keine JSON-/Sitzungsdateibereinigung, Kürzung von Transkriptdateien, Sitzungsdateisperren
  oder sitzungssperrenförmigen Tests.
- Keine Laufzeit-Kompatibilitätsexporte, deren Zweck darin besteht, alte Sitzungsdateien
  aktuell zu halten.
- Explizite Support-Exporte bleiben vom Benutzer angeforderte Archiv-/Materialisierungs-
  formate und dürfen Dateinamen nicht wieder in die Laufzeitidentität einspeisen.

## Sicherung und Wiederherstellung

Sicherungen sollten eine einzelne Archivdatei sein, die Datenbankerfassung sollte jedoch
SQLite-nativ erfolgen:

1. Halten Sie Schreibtransaktionen begrenzt, damit die Online-Sicherung Fortschritte erzielen kann.
2. Verifizieren Sie jede aktive globale und jede Agent-Datenbank vor der Erfassung.
3. Erfassen Sie jede Datenbank mit der SQLite-Online-Sicherung in einem temporären Sicherungs-
   verzeichnis, schließen Sie dann die aktive Verbindung und führen Sie `VACUUM` für die private Kopie aus.
   Plugin-Schemas, die vom Eigentümer definierte SQLite-Capabilities benötigen, schlagen sicher geschlossen fehl,
   bis der Eigentümer einen sicheren Snapshot-Vertrag bereitstellt.
4. Archivieren Sie die Datenbank-Snapshots, die Konfigurationsdatei, das Verzeichnis für Zugangsdaten, ausgewählte
   Workspaces und ein Manifest.
5. Verifizieren Sie die Dateistruktur jedes SQLite-Snapshots, öffnen Sie anschließend kanonische OpenClaw-
   Datenbanken und führen Sie `PRAGMA integrity_check` sowie eine Rollenvalidierung aus. Dedizierte
   Plugin-Schemas bleiben undurchsichtig, sofern ihr Eigentümer keinen Verifizierer bereitstellt.
   `openclaw backup create` führt dies standardmäßig durch; `--no-verify` dient nur dazu,
   den Archivdurchlauf nach dem Schreiben absichtlich zu überspringen.

Verlassen Sie sich nicht auf unverarbeitete Live-Kopien von `*.sqlite`, `*.sqlite-wal` und `*.sqlite-shm` als
primäres Sicherungsformat. Das Archivmanifest sollte Datenbankrolle,
Agent-ID, Schemaversion, Quellpfad, Snapshot-Pfad, Byte-Größe und Integritäts-
status aufzeichnen.

Die Wiederherstellung sollte die globale Datenbank und die Agent-Datenbankdateien aus den
Archiv-Snapshots neu erstellen. Das globale Schema bleibt auf Version `1`; Agent-Snapshots der Version `1`
erhalten das begrenzte Laufzeit-Upgrade auf Version `2`. Doctor bleibt
der einzige Eigentümer des Datei-zu-Datenbank-Imports. Der Wiederherstellungsbefehl validiert zuerst das
Archiv und ersetzt anschließend jedes Manifest-Asset durch die verifizierten extrahierten
Nutzdaten.

## Plan zur Laufzeit-Refaktorierung

1. Fügen Sie Datenbank-Registry-APIs hinzu.
   - Lösen Sie die Pfade der globalen DB und der Agent-DBs auf.
   - Behalten Sie das globale Schema bei `user_version = 1`. Agent-DBs verwenden Version `2`
     mit einer atomaren Migration von der ausgelieferten Memory-Source-Struktur der Version `1`.
   - Fügen Sie Hilfsfunktionen zum Schließen, für Prüfpunkte und für Integritätsprüfungen hinzu, die von Tests, Sicherung und Doctor verwendet werden.

2. Führen Sie Sidecar-SQLite-Stores zusammen.
   - Verschieben Sie Plugin-Statustabellen in die globale Datenbank. Für Laufzeit-
     schreibvorgänge abgeschlossen; der nicht ausgelieferte Importer für veraltete Sidecars wurde gelöscht.
   - Verschieben Sie Task-Registry-Tabellen in die globale Datenbank. Für Laufzeit-
     schreibvorgänge abgeschlossen; der nicht ausgelieferte Importer für veraltete Sidecars wurde gelöscht.
   - Verschieben Sie TaskFlow-Tabellen in die globale Datenbank. Für Laufzeitschreibvorgänge abgeschlossen;
     der nicht ausgelieferte Importer für veraltete Sidecars wurde gelöscht.
   - Verschieben Sie integrierte Memory-Search-Tabellen in jede Agent-Datenbank. Abgeschlossen; benutzerdefiniertes
     `memorySearch.store.path` wird nun durch die Doctor-Konfigurationsmigration entfernt.
     Die vollständige Neuindizierung erfolgt direkt und ausschließlich für Memory-Tabellen; der alte Austauschpfad für die gesamte Datei
     und die Hilfsfunktion zum Austauschen des Sidecar-Index wurden gelöscht.
   - Löschen Sie doppelte Datenbank-Öffnungsfunktionen, WAL-Einrichtung, Berechtigungshelfer und
     Schließpfade aus diesen Subsystemen.

3. Verschieben Sie agent-eigene Tabellen in Agent-Datenbanken.
   - Erstellen Sie die Agent-DB bei Bedarf über die Registry der globalen Datenbank. Abgeschlossen.
   - Verschieben Sie Laufzeit-Sitzungseinträge, Transkriptereignisse, VFS-Zeilen und Tool-
     Artefakte in Agent-DBs. Abgeschlossen.
   - Migrieren Sie keine branch-lokalen Sitzungseinträge der gemeinsam genutzten DB, Transkriptereignisse,
     VFS-Zeilen oder Tool-Artefakte; dieses Layout wurde nie ausgeliefert. Behalten Sie nur den Import
     von Dateien in die Datenbank in Doctor bei.

4. Ersetzen Sie die Sitzungs-Store-APIs.
   - Entfernen Sie `storePath` als Laufzeitidentität. Für die Laufzeit abgeschlossen und durch
     `check:database-first-legacy-stores` abgesichert: Sitzungsmetadaten, Routenaktualisierungen,
     Befehlspersistenz, CLI-Sitzungsbereinigung, Feishu-Vorschauen des Reasonings,
     Persistenz des Transkriptstatus, Subagent-Tiefe, sitzungsbezogene Überschreibungen von
     Authentifizierungsprofilen, Parent-Fork-Logik und QA-Lab-Inspektion lösen nun die
     Datenbank anhand kanonischer Agent-/Sitzungsschlüssel auf.
     Gateway-/TUI-/UI-/macOS-Antworten für Sitzungslisten stellen nun `databasePath`
     statt des veralteten `path` bereit; macOS-Debugoberflächen zeigen die Agent-Datenbank
     als schreibgeschützten Status an, statt die `session.store`-Konfiguration zu schreiben.
     `/status`, der chatgesteuerte Trajektorienexport und CLI-Abhängigkeits-Proxys
     geben keine veralteten Store-Pfade mehr weiter; der Fallback für die Transkriptnutzung liest
     SQLite anhand der Agent-/Sitzungsidentität. Laufzeit- und Bridge-Tests stellen
     `storePath` nicht mehr bereit; Doctor-/Migrationseingaben verwalten diesen veralteten Feldnamen.
     Das kombinierte Laden von Sitzungen im Gateway besitzt keinen speziellen Laufzeitzweig mehr für
     nicht vorlagenbasierte `session.store`-Werte; stattdessen werden Agent-SQLite-Zeilen aggregiert.
     Der veraltete Doctor-Pfad für Sitzungssperren und seine `.jsonl.lock`-Bereinigungshilfsfunktion
     wurden entfernt; SQLite bildet nun die Nebenläufigkeitsgrenze für Sitzungen.
     Häufig aufgerufene Laufzeitstellen verwenden zeilenorientierte Hilfsfunktionsnamen wie
     `resolveSessionRowEntry`; der alte Kompatibilitäts-
     alias `resolveSessionStoreEntry` wurde aus Laufzeit- und Plugin-SDK-Exporten entfernt.

- Verwenden Sie `{ agentId, sessionKey }`-Zeilenoperationen.
  Erledigt: `getSessionEntry`, `upsertSessionEntry`, `deleteSessionEntry`,
  `patchSessionEntry` und `listSessionEntries` sind SQLite-zuerst-APIs, die
  keinen Pfad zum Sitzungsspeicher benötigen. Statusübersicht, lokaler Agentenstatus, Integritätsstatus
  und der Auflistungsbefehl `openclaw sessions` lesen jetzt agentenspezifische Zeilen direkt
  und zeigen agentenspezifische SQLite-Datenbankpfade anstelle von `sessions.json`-Pfaden an.
- Ersetzen Sie das Löschen/Einfügen des gesamten Speichers durch `upsertSessionEntry`,
  `deleteSessionEntry`, `listSessionEntries` und SQL-Bereinigungsabfragen.
  Für die Laufzeit erledigt: Häufig ausgeführte Pfade verwenden jetzt Zeilen-APIs und Zeilen-Patches mit
  Konfliktwiederholung; die verbleibenden Hilfsfunktionen zum Importieren/Ersetzen des gesamten Speichers sind auf
  den Migrationsimportcode und Tests des SQLite-Backends beschränkt.
  - Löschen Sie `store-writer.ts` und die Tests der Schreibwarteschlange. Erledigt.
  - Entfernen Sie die laufzeitbezogene Bereinigung veralteter Schlüssel und Parameter zum Löschen von Aliasen aus
    Upserts/Patches von Sitzungszeilen. Erledigt.

5. Entfernen Sie das laufzeitbezogene Verhalten der JSON-Registry.
   - Stellen Sie Lese- und Schreibvorgänge der Sandbox-Registry vollständig auf SQLite um. Erledigt.
   - Importieren Sie monolithisches und fragmentiertes JSON ausschließlich im Migrationsschritt. Erledigt.
   - Entfernen Sie Sperren der fragmentierten Registry und JSON-Schreibvorgänge. Erledigt.

- Verwenden Sie eine typisierte Registry-Tabelle, statt Registry-Zeilen als generisches
  undurchsichtiges JSON zu speichern, wenn die Struktur weiterhin betrieblicher Zustand eines häufig ausgeführten Pfads ist. Erledigt.

6. Entfernen Sie Sitzungsmutationen nach Art von Dateisperren.
   - Für die Erstellung von Laufzeitsperren und Laufzeit-Sperr-APIs erledigt.
   - Der eigenständige veraltete Doctor-Bereinigungspfad `.jsonl.lock` wurde entfernt.
   - Die Zustandsintegrität verfügt nicht mehr über einen separaten Bereinigungspfad für verwaiste
     Transkriptdateien; die Doctor-Migration importiert/entfernt veraltete JSONL-Quellen zentral.
   - Die Gateway-Singleton-Koordination verwendet typisierte SQLite-Zeilen `state_leases` unter
     `gateway_locks` und stellt keine Schnittstelle für ein Dateisperrverzeichnis mehr bereit.
   - Die generische Deduplizierungspersistenz des Plugin-SDK verwendet keine Dateisperren oder JSON-
     Dateien mehr; sie schreibt gemeinsam genutzte SQLite-Plugin-Zustandszeilen. Erledigt.
   - Die QMD-Koordination verwendet eine gemeinsam genutzte SQLite-Lease für Einbettungen und eine agentenspezifische
     SQLite-Lease für jeden Schreiber von Sammlungen/Aktualisierungen/Einbettungen. Die Laufzeit erstellt nicht mehr
     `qmd/embed.lock.lock` oder `agents/<agentId>/qmd-write.lock.lock`;
     Doctor entfernt nur eindeutig veraltete, außer Betrieb genommene Sidecar-Dateien. Erledigt.

7. Machen Sie Worker datenbankfähig.
   - Worker öffnen ihre eigenen SQLite-Verbindungen.
   - Der übergeordnete Prozess ist für Zustellung, Kanal-Callbacks und Konfiguration zuständig.
   - Der Worker erhält Agenten-ID, Ausführungs-ID, Dateisystemmodus und DB-Registry-
     Identität, keine aktiven Handles.
   - `vfs-only` bleibt experimentell und verwendet die Agentendatenbank als Speicher-
     Stammverzeichnis.
   - Verwenden Sie zunächst einen Worker pro aktiver Ausführung. Pooling kann warten, bis die Lebensdauer
     von DB-Verbindungen und das Abbruchverhalten unspektakulär sind.

8. Backup-Integration.
   - Erweitern Sie das Backup so, dass globale, agentenspezifische und Plugin-Datenbanken durch ein Online-
     Backup mit anschließendem Offline-`VACUUM` gesichert werden. Für erkannte `*.sqlite`-Dateien unter dem Zustandsobjekt erledigt;
     Plugin-Schemas, die nicht verfügbare Besitzerfunktionen erfordern, schlagen sicher geschlossen fehl.
   - Fügen Sie eine Backup-Verifizierung für kanonische SQLite-Integrität und Schemaidentität
     sowie eine generische Validierung der Dateistruktur für dedizierte Plugin-Snapshots hinzu. Für
     die Backup-Erstellung und die standardmäßige Archivverifizierung erledigt.
   - Zeichnen Sie Metadaten der Backup-Ausführung in SQLite auf. Über die gemeinsam genutzte Tabelle `backup_runs`
     mit Archivpfad, Status und Manifest-JSON erledigt.
   - Fügen Sie die Wiederherstellung aus verifizierten Archiv-Snapshots hinzu. Erledigt: `openclaw backup
restore` validiert vor dem Extrahieren, verwendet das normalisierte
     Manifest der Verifizierungsfunktion, unterstützt `--dry-run` und erfordert `--yes`, bevor
     aufgezeichnete Quellpfade ersetzt werden.
   - Schließen Sie den VFS-/Workspace-Export nur auf Anforderung ein; exportieren Sie Sitzungs-
     interne Daten nicht als JSON oder JSONL.

9. Entfernen Sie veraltete Tests und veralteten Code. Für die bekannten Laufzeit-Sitzungsoberflächen erledigt.

- Entfernen Sie Tests, die die laufzeitbezogene Erstellung von `sessions.json`- oder Transkript-
  JSONL-Dateien voraussetzen. Erledigt für den zentralen Sitzungsspeicher, Chat, Gateway-Transkriptereignisse,
  Vorschau, Lebenszyklus, Aktualisierungen von Befehlssitzungseinträgen, automatisches Antwort-Zurücksetzen/Tracing und
  Dreaming-Fixtures von memory-core, Routing von Genehmigungszielen, Reparatur von Sitzungstranskripten,
  Reparatur von Sicherheitsberechtigungen, Trajektorienexport und Sitzungsexport.
  Active-Memory-Transkripttests prüfen jetzt SQLite-Gültigkeitsbereiche und stellen sicher, dass keine temporären oder
  persistenten JSONL-Dateien erstellt werden.
  Die alte Heartbeat-Regression zur Transkriptbereinigung wurde entfernt, weil
  die Laufzeit JSONL-Transkripte nicht mehr kürzt.
  Tests des Agentenwerkzeugs zur Sitzungsauflistung modellieren veraltete `sessions.json`-Pfade
  nicht mehr als Form der Gateway-Antwort; App-/UI-/macOS-Tests verwenden `databasePath`.
  Die Transkriptnutzungstests von `/status` befüllen SQLite-Transkriptzeilen jetzt direkt,
  statt JSONL-Dateien zu schreiben.
  Tests des Gateway-Sitzungslebenszyklus verwenden jetzt direkt Hilfsfunktionen zum Befüllen von SQLite-Transkripten;
  die alte Struktur des Fixtures mit einer einzeiligen Sitzungsdatei wurde aus der Abdeckung für Zurücksetzen
  und Löschen entfernt.
  `sessions.delete` gibt das aus der Dateiära stammende Feld `archived: []` nicht mehr zurück; Löschvorgänge
  melden nur das Ergebnis der Zeilenmutation. Auch die alte Option `deleteTranscript` wurde
  entfernt: Beim Löschen einer Sitzung wird das kanonische Stammobjekt `sessions` entfernt, und
  SQLite löscht über Kaskaden die sitzungseigenen Transkript-, Snapshot- und Trajektorienzeilen, sodass kein
  Aufrufer Transkriptwaisen hinterlassen oder einen Bereinigungszweig vergessen kann.
  Tests zur Erfassung von Trajektorien der Kontext-Engine lesen jetzt `trajectory_runtime_events`-
  Zeilen aus einer isolierten Agentendatenbank, statt
  `session.trajectory.jsonl` zu lesen.
  Docker-MCP-Kanal-Befüllungsskripte befüllen SQLite-Zeilen jetzt direkt. Direkte
  Schreibvorgänge in `sessions.json` sind auf Doctor-Fixtures beschränkt.
  Tool Search Gateway E2E liest Nachweise von Werkzeugaufrufen aus SQLite-Transkriptzeilen,
  statt `agents/<agentId>/sessions/*.jsonl`-Dateien zu durchsuchen.
  Host-Ereignisse von memory-core und temporäre Sitzungskorpuszeilen befinden sich jetzt im gemeinsam genutzten
  SQLite-Plugin-Zustand; `events.jsonl` und `session-corpus/*.txt` sind ausschließlich Legacy-Eingaben für die
  Doctor-Migration. Aktive Zeilen verwenden virtuelle `memory/session-ingestion/`-
  Pfade, nicht `.dreams/session-corpus`. Das alte Reparaturmodul für Dreaming von memory-core
  und seine CLI-/Gateway-Tests wurden entfernt, da die Laufzeit nicht mehr
  für die Reparatur von Dateiarchiven dieses Korpus zuständig ist. Tests der Bridge/öffentlichen Artefakte
  von memory-core stellen `.dreams/events.jsonl` nicht mehr bereit; sie
  verwenden den SQLite-gestützten virtuellen JSON-Artefaktnamen.
  Öffentliche SDK-/Codex-Testdokumentation spricht jetzt von SQLite-Sitzungszustand statt von Sitzungs-
  dateien, und das Beispiel für eine Kanalrunde stellt kein Argument `storePath` mehr bereit.
  Der Matrix-Synchronisierungszustand verwendet jetzt direkt den SQLite-Plugin-Zustandsspeicher. Aktive
  Client-/Laufzeitverträge übergeben ein Kontospeicher-Stammverzeichnis, keinen `bot-storage.json`-
  Pfad, und Doctor importiert veraltetes `bot-storage.json` in SQLite, bevor
  die Quelle gelöscht wird. Neustart-/destruktive Matrix-Szenarien im QA Lab mutieren jetzt direkt die SQLite-Synchronisierungs-
  zeile, statt fingierte `bot-storage.json`-Dateien zu erstellen oder zu löschen, und
  die E2EE-Grundlage übergibt ein Stammverzeichnis des Synchronisierungsspeichers statt eines fingierten
  `sync-store.json`-Pfads.
  Die Auswahl des Matrix-Speicher-Stammverzeichnisses bewertet Stammverzeichnisse nicht mehr anhand veralteter JSON-
  Dateien für Synchronisierung/Threads; sie verwendet dauerhafte Stammmetadaten und echten Kryptografiezustand.
  Die Testsuite des SQLite-Laufzeit-Sitzungsbackends erzeugt kein fingiertes
  `sessions.json` mehr; veraltete Quell-Fixtures befinden sich jetzt in den Doctor-
  Tests, die sie importieren.
  Gateway-Sitzungstests stellen keine Hilfsfunktion `createSessionStoreDir` und
  keine ungenutzte Einrichtung temporärer Sitzungsspeicherpfade mehr bereit; Fixture-Verzeichnisse sind explizit, und die direkte
  Zeileneinrichtung verwendet die Benennung von SQLite-Sitzungszeilen.
  Die ausschließlich für Doctor vorgesehene Abdeckung des JSON5-Sitzungsspeicher-Parsers wurde aus Infrastrukturtests
  in Doctor-Migrationstests verschoben, sodass Laufzeit-Testsuites nicht mehr für das Parsen veralteter
  Sitzungsdateien zuständig sind.
  Laufzeit-SSO-/Tests ausstehender Uploads von Microsoft Teams führen keine JSON-Sidecar-
  Fixtures oder Parser mehr mit; das Parsen veralteter SSO-Token befindet sich ausschließlich im Plugin-
  Migrationsmodul. Telegram-Tests befüllen keine fingierten `/tmp/*.json`-Speicher-
  pfade mehr; sie setzen den SQLite-gestützten Nachrichten-Cache direkt zurück. Die generische
  OpenClaw-Testzustands-Hilfsfunktion stellt keinen veralteten `auth-profiles.json`-
  Schreiber mehr bereit; Doctor-Tests zur Authentifizierungsmigration verwalten dieses Fixture lokal.
  Laufzeittests für TUI-Zeiger auf die letzte Sitzung, Ausführungsgenehmigungen, Active-Memory-
  Umschalter, Matrix-Deduplizierung/Startverifizierung, Memory-Wiki-Quellsynchronisierung,
  Bindungen aktueller Unterhaltungen, Onboarding-Authentifizierung und Hermes-Geheimnisimporte
  erzeugen keine alten Sidecar-Dateien mehr und prüfen nicht mehr, ob alte Dateinamen fehlen. Sie
  weisen das Verhalten über SQLite-Zeilen und öffentliche Speicher-APIs nach; Doctor-/Migrations-
  tests sind der einzige Ort, an den veraltete Quelldateinamen gehören.
  Laufzeittests für Geräte-/Node-Kopplung, allowFrom von Kanälen, Neustartabsichten,
  Neustartübergabe, Einträge der Sitzungszustellungswarteschlange, Konfigurationsintegrität, iMessage-
  Caches, Cron-Aufträge, PI-Transkriptkopfzeilen, Subagent-Registrys und verwaltete
  Bildanhänge erstellen ebenfalls keine außer Betrieb genommenen JSON-/JSONL-Dateien mehr, nur um
  nachzuweisen, dass sie ignoriert werden oder fehlen.
  Die PI-Überlaufwiederherstellung verfügt nicht mehr über einen SessionManager-Fallback zum Umschreiben/Kürzen:
  Die Kürzung von Werkzeugergebnissen und das Umschreiben von Transkripten durch die Kontext-Engine mutieren
  SQLite-Transkriptzeilen und aktualisieren anschließend den aktiven Prompt-Zustand aus der Datenbank.
  Persistierte Operationen zum Anhängen von SessionManager-Nachrichten delegieren die Auswahl des übergeordneten Elements
  und die Idempotenz an die atomare SQLite-Hilfsfunktion zum Anhängen von Transkripten. Normale
  Operationen zum Anhängen von Metadaten oder benutzerdefinierten Einträgen wählen das aktuelle übergeordnete Element ebenfalls innerhalb von SQLite aus, sodass
  veraltete Manager-Instanzen keine Elternketten-Wettläufe aus der Zeit vor SQLite wieder aufleben lassen.
  Die Bereinigung des synthetischen PI-Endes für Prüfungen während einer Runde und `sessions_yield` kürzt
  den SQLite-Transkriptzustand jetzt direkt; die alte SessionManager-Bridge zur Entfernung des Endes
  und ihre Tests wurden entfernt.
  Auch die Erfassung von Compaction-Prüfpunkten erstellt Snapshots ausschließlich aus SQLite; Aufrufer
  übergeben keinen aktiven SessionManager mehr als alternative Transkriptquelle.
- Behalten Sie Tests, die veraltete Dateien befüllen, ausschließlich für Migrationen bei.
- Der JSON-Dateinachweis wurde für aktive Laufzeitoberflächen durch den Nachweis über SQL-Zeilen ersetzt.

- Fügen Sie statische Verbote für Laufzeitschreibvorgänge in veraltete JSON-Pfade für Sitzungen/Caches hinzu.
  Für die Repository-Prüfung erledigt.

10. Machen Sie den Migrationsbericht auditierbar.
    - Zeichnen Sie Migrationsausführungen in SQLite mit Start-/Endzeitstempeln, Quell-
      pfaden, Quell-Hashes, Anzahlen, Warnungen und Backup-Pfad auf.
      Erledigt: Ausführungen der Migration veralteter Zustände speichern jetzt einen `migration_runs`-
      Bericht mit Quellpfad-/Tabelleninventar, SHA-256 der Quelldatei, Größen,
      Datensatzanzahlen, Warnungen und Backup-Pfad.
      Erledigt: Ausführungen der Migration veralteter Zustände speichern außerdem `migration_sources`-
      Zeilen für Audits auf Quellenebene und zukünftige Entscheidungen zum Überspringen/Nachfüllen.
    - Machen Sie die Anwendung idempotent. Eine erneute Ausführung nach einem Teilimport sollte entweder
      eine bereits importierte Quelle überspringen oder anhand eines stabilen Schlüssels zusammenführen.
      Erledigt: Sitzungsindizes, Transkripte, Zustellungswarteschlangen, Plugin-Zustand, Aufgaben-
      register und agenteneigene globale SQLite-Zeilen werden über stabile Schlüssel oder
      Upsert-/Ersetzungssemantik importiert, sodass erneute Ausführungen zusammenführen, ohne dauerhafte
      Zeilen zu duplizieren.
    - Fehlgeschlagene Importe müssen die ursprüngliche Quelldatei unverändert an ihrem Speicherort belassen.
      Erledigt: Fehlgeschlagene Transkriptimporte belassen die ursprüngliche JSONL-Quelle jetzt an
      ihrem erkannten Pfad, und `migration_sources` zeichnet die Quelle als
      `warning` mit `removed_source=0` für die nächste Doctor-Ausführung auf.

## Leistungsregeln

- Eine Verbindung pro Thread/Prozess ist in Ordnung; Handles dürfen nicht zwischen
  Workern geteilt werden.
- Verwenden Sie WAL, `foreign_keys=ON`, ein Busy-Timeout von 5s und kurze `BEGIN IMMEDIATE`-
  Schreibtransaktionen. Legen Sie keine synchronen Wiederholungsversuche für Sperren über
  die einmalige Busy-Wartezeit von SQLite.
- Halten Sie Hilfsfunktionen für Schreibtransaktionen synchron, sofern und bis eine asynchrone Transaktions-
  API explizite Mutex-/Backpressure-Semantik bereitstellt.
- Halten Sie Schreibvorgänge für die übergeordnete Zustellung klein und transaktional.
- Vermeiden Sie vollständige Neuschreibungen des Speichers; verwenden Sie Upsert/Löschen auf Zeilenebene.
- Fügen Sie Indizes für die Auflistung nach Agent, die Auflistung nach Sitzung, den Aktualisierungszeitpunkt, die Ausführungs-ID und
  Ablaufpfade hinzu, bevor Sie häufig ausgeführten Code verschieben.
- Speichern Sie große Artefakte, Medien und Vektoren als BLOBs oder in aufgeteilten BLOB-Zeilen, nicht
  als base64 oder JSON mit numerischen Arrays.
- Halten Sie undurchsichtige Plugin-Zustandseinträge klein und klar abgegrenzt.
- Fügen Sie eine SQL-Bereinigung für TTL/Ablauf hinzu, anstatt das Dateisystem zu bereinigen.
  Für datenbankeigene Laufzeitspeicher erledigt: Medien, Plugin-Zustand, Plugin-BLOBs,
  persistente Deduplizierung und Agent-Cache laufen sämtlich über SQLite-Zeilen ab. Die verbleibende
  Dateisystembereinigung ist auf temporäre Materialisierungen oder explizite
  Entfernungsbefehle beschränkt.

## Statische Verbote

Fügen Sie eine Repository-Prüfung hinzu, bei der neue Laufzeitschreibvorgänge in veraltete Zustandspfade fehlschlagen:

- `sessions.json`
- `*.trajectory.jsonl` außer materialisierten Ausgaben von Support-Paketen
- `.acp-stream.jsonl`
- `acp/event-ledger.json`
- `cache/*.json` Laufzeit-Cache-Dateien
- `agents/<agentId>/agent/auth.json`
- `agents/<agentId>/agent/models.json`
- `credentials/oauth.json`
- `github-copilot.token.json`
- `openrouter-models.json`
- `auth-profiles.json`
- `auth-state.json`
- `exec-approvals.json`
- `openclaw-workspace-state.json`
- `workspace-state.json`
- `workspace-attestations/*.attested`
- benachbarte `<workspace>.attested`
- Matrix `credentials*.json` und `recovery-key.json`
- `cron/runs/*.jsonl`
- `cron/jobs.json`
- `jobs-state.json`
- `device-pair-notify.json`
- `devices/pending.json` / `devices/paired.json` / `devices/bootstrap.json`
  (seit 2026.7 außer Betrieb: Laufzeitspeicher ist `device_pairing_*` /
  `device_bootstrap_tokens` in der gemeinsamen Zustandsdatenbank; gekoppelte Datensätze werden beim
  Gateway-Start importiert, vorübergehende ausstehende/Bootstrap-Zeilen werden verworfen)
- `nodes/pending.json` / `nodes/paired.json` (seit 2026.7 außer Betrieb: beim Gateway-Start in gekoppelte Gerätedatensätze integriert)
- `identity/device.json`
- `identity/device-auth.json` (außer Betrieb; Import ausschließlich durch Doctor in `device_auth_tokens`)
- `push/web-push-subscriptions.json` (außer Betrieb; Import ausschließlich durch Doctor in `web_push_subscriptions`)
- `push/vapid-keys.json` (außer Betrieb; Import ausschließlich durch Doctor in `web_push_vapid_keys`)
- `push/apns-registrations.json` (außer Betrieb; Import ausschließlich durch Doctor in `apns_registrations`)
- `process-leases.json`
- `gateway-instance-id`
- `session-toggles.json`
- Memory-core `.dreams/events.jsonl`
- Memory-core `.dreams/session-corpus/`
- Memory-core `.dreams/daily-ingestion.json`
- Memory-core `.dreams/session-ingestion.json`
- Memory-core `.dreams/short-term-recall.json`
- Memory-core `.dreams/phase-signals.json`
- Memory-core `.dreams/short-term-promotion.lock`
- Skill Workshop `skill-workshop/<workspace>.json`
- Skill Workshop `skill-workshop/skill-workshop-review-*.json`
- Nostr `bus-state-*.json`
- Nostr `profile-state-*.json`
- `calls.jsonl`
- `known-users.json`
- `ref-index.jsonl`
- QQBot `session-*.json`
- BlueBubbles `bluebubbles/catchup/*.json`
- BlueBubbles `bluebubbles/inbound-dedupe/*.json`
- Telegram `update-offset-*.json`
- Telegram `sticker-cache.json`
- Telegram `*.telegram-messages.json`
- Telegram `*.telegram-sent-messages.json`
- Telegram `*.telegram-topic-names.json`
- Telegram `thread-bindings-*.json`
- iMessage `catchup/*.json`
- iMessage `reply-cache.jsonl`
- iMessage `sent-echoes.jsonl`
- Microsoft Teams `msteams-conversations.json`
- Microsoft Teams `msteams-polls.json`
- Microsoft Teams `msteams-sso-tokens.json`
- Microsoft Teams `*.learnings.json`
- Matrix `bot-storage.json`
- Matrix `sync-store.json`
- Matrix `thread-bindings.json`
- Matrix `inbound-dedupe.json`
- Matrix `startup-verification.json`
- Matrix `storage-meta.json`
- Matrix `crypto-idb-snapshot.json`
- Discord `model-picker-preferences.json`
- Discord `command-deploy-cache.json`
- JSON-Dateien für Sandbox-Registrierungs-Shards
- `plugin-state/state.sqlite`
- Ad-hoc-Laufzeit-Sidecars `openclaw-state.sqlite`
- `tasks/runs.sqlite`
- `tasks/flows/registry.sqlite`
- `bindings/current-conversations.json`
- `restart-sentinel.json`
- `gateway-restart-intent.json`
- `gateway-supervisor-restart-handoff.json`
- `gateway.<hash>.lock`
- `qmd/embed.lock.lock`
- `agents/<agentId>/qmd-write.lock.lock`
- `commands.log`
- `config-health.json`
- `port-guard.json`
- `settings/voicewake.json`
- `settings/voicewake-routing.json`
- `plugin-binding-approvals.json`
- `plugins/installs.json`
- `audit/file-transfer.jsonl`
- `audit/crestodian.jsonl`
- `crestodian/rescue-pending/*.json`
- `openclaw/rescue-pending/*.json`
- `plugins/phone-control/armed.json`
- Memory Wiki `.openclaw-wiki/log.jsonl`
- Memory Wiki `.openclaw-wiki/state.json`
- Memory Wiki `.openclaw-wiki/locks/`
- Memory Wiki `.openclaw-wiki/source-sync.json`
- Memory Wiki `.openclaw-wiki/import-runs/*.json`
- Memory Wiki `.openclaw-wiki/cache/agent-digest.json`
- Memory Wiki `.openclaw-wiki/cache/claims.jsonl`
- ClawHub `.clawhub/lock.json`
- ClawHub `.clawhub/origin.json`
- Browserprofil-Dekoration `.openclaw-profile-decorated`
- `SessionManager.open(...)` dateibasierte Sitzungsöffner
- `SessionManager.listAll(...)` und `TranscriptSessionManager.listAll(...)`
  Fassaden zur Transkriptauflistung
- `SessionManager.forkFromSession(...)` und
  `TranscriptSessionManager.forkFromSession(...)` Fassaden zum Abzweigen von Transkripten
- `SessionManager.newSession(...)` und `TranscriptSessionManager.newSession(...)`
  Fassaden zum veränderbaren Ersetzen von Sitzungen
- `SessionManager.createBranchedSession(...)` und
  `TranscriptSessionManager.createBranchedSession(...)` Fassaden für Zweigsitzungen

Das Verbot sollte Tests das Erstellen veralteter Fixtures gestatten und Migrationscode das
Lesen/Importieren/Entfernen veralteter Dateiquellen erlauben. Nicht veröffentlichte SQLite-Sidecars bleiben verboten
und erhalten keine Doctor-Importausnahmen.

## Abschlusskriterien

- Laufzeitdaten und Cache-Schreibvorgänge werden in die globale oder Agent-SQLite-Datenbank geschrieben.
- Die Laufzeit schreibt keine Sitzungsindizes, Transkript-JSONL, Sandbox-Registrierungs-
  JSON-Dateien, Aufgaben-Sidecar-SQLite oder Plugin-Zustands-Sidecar-SQLite mehr. Die nicht veröffentlichten
  SQLite-Importer für Aufgaben- und Plugin-Zustands-Sidecars werden gelöscht.
- Der Import veralteter Dateien erfolgt ausschließlich durch Doctor.
- Die Sicherung erzeugt ein Archiv mit kompakten SQLite-Snapshots und Integritätsnachweis.
- Agent-Worker können mit Festplatten-, VFS-Scratch- oder experimentellem reinem VFS-
  Speicher ausgeführt werden.
- Konfigurations- und explizite Anmeldedatendateien bleiben die einzigen erwarteten persistenten
  Steuerdateien außerhalb der Datenbank.
- Repository-Prüfungen verhindern die Wiedereinführung veralteter Laufzeit-Dateispeicher.
