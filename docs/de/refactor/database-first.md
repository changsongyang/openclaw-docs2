---
read_when:
    - OpenClaw-Laufzeitdaten, Cache, Transkripte, Aufgabenstatus oder temporäre Dateien nach SQLite verschieben
    - Entwerfen von Doctor-Migrationen aus veralteten JSON- oder JSONL-Dateien
    - Backup-, Wiederherstellungs-, VFS- oder Worker-Speicherverhalten ändern
    - Entfernen von Sitzungssperren, Bereinigung, Kürzung oder JSON-Kompatibilitätspfaden
summary: Migrationsplan, um SQLite zur primären dauerhaften Zustands- und Cache-Schicht zu machen, während die Konfiguration dateibasiert bleibt
title: Datenbankorientierte Zustandsrefaktorierung
x-i18n:
    generated_at: "2026-07-26T18:03:05Z"
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

Verwenden Sie eine zweistufige SQLite-Struktur:

- Globale Datenbank: `~/.openclaw/state/openclaw.sqlite`
- Agentendatenbank: eine SQLite-Datenbank pro Agent für den agenteneigenen Arbeitsbereich,
  das Transkript, VFS, Artefakte und umfangreiche agentenspezifische Laufzeitdaten
- Die Konfiguration bleibt dateibasiert: `openclaw.json` verbleibt außerhalb der
  Datenbank. Laufzeit-Authentifizierungsprofile werden nach SQLite verschoben; externe Provider- oder CLI-
  Anmeldedatendateien bleiben außerhalb der OpenClaw-Datenbank unter Verwaltung des jeweiligen Eigentümers.

Die globale Datenbank ist die Datenbank der Steuerungsebene. Sie verwaltet die Agentenerkennung,
den gemeinsam genutzten Gateway-Zustand, Kopplungen, den Geräte-/Node-Zustand, Aufgaben- und Ablaufregister, den Plugin-
Zustand, den Scheduler-Laufzeitzustand, Sicherungsmetadaten und den Migrationszustand.

Die Agentendatenbank ist die Datenbank der Datenebene. Sie verwaltet die Sitzungsmetadaten
des Agenten, den Transkriptereignisstrom, den VFS-Arbeitsbereich oder Scratch-Namensraum, Tool-
Artefakte, Ausführungsartefakte und durchsuchbare beziehungsweise indizierbare agentenlokale Cache-Daten.

Dies bietet eine einheitliche dauerhafte globale Sicht, ohne große Agentenarbeitsbereiche,
Transkripte und binäre Scratch-Daten in den gemeinsam genutzten Gateway-Schreibpfad zu zwingen.

## Verbindlicher Vertrag

Diese Migration hat genau eine kanonische Laufzeitstruktur:

- Sitzungszeilen speichern ausschließlich Sitzungsmetadaten. Sie dürfen weder
  `transcriptLocator` noch Transkriptdateipfade, benachbarte JSONL-Pfade, Sperrpfade,
  Bereinigungsmetadaten oder Kompatibilitätszeiger aus der Dateiära speichern.
- Die Transkriptidentität ist immer eine SQLite-Identität: `{agentId, sessionId}` sowie
  optionale Themenmetadaten, sofern das Protokoll diese benötigt.
- `sqlite-transcript://...` ist keine Laufzeit- oder Protokollidentität. Neuer Code darf
  keine Transkript-Locators ableiten, speichern, übergeben, analysieren oder migrieren. Laufzeit und
  Tests sollten überhaupt keine Pseudo-Locators enthalten; die Dokumentation darf die Zeichenfolge
  ausschließlich erwähnen, um sie zu verbieten.
- Alte `sessions.json`, Transkript-JSONL, `.jsonl.lock`, Bereinigung, Kürzung
  und alte Sitzungspfadlogik gehören ausschließlich in den Doctor-Migrations-/Importpfad.
- Alte Aliase der Sitzungskonfiguration gehören ausschließlich in die Doctor-Migration. Die Laufzeit
  interpretiert weder `session.idleMinutes` noch `session.resetByType.dm` oder
  agentenübergreifende `agent:main:*`-Hauptsitzungsaliase für einen anderen konfigurierten Agenten.
- Die Sitzungsroutingidentität ist typisierter relationaler Zustand. Häufig durchlaufene Laufzeit- und UI-Pfade
  sollten `sessions.session_scope`, `sessions.account_id`,
  `sessions.primary_conversation_id`, `conversations` und
  `session_conversations` lesen; sie dürfen weder `session_key` analysieren noch
  `session_entries.entry_json` nach der Provideridentität durchsuchen, außer vorübergehend
  als Kompatibilitätsschatten, während alte Aufrufstellen entfernt werden.
- Direktnachrichtenmarkierungen auf Kanalebene wie `dm` gegenüber `direct` sind Routing-
  Vokabular und keine Transkript-Locators oder Kompatibilitäts-Handles für den Dateispeicher.
- Die alte Hook-Handler-Konfiguration gehört ausschließlich in Doctor-Warnungs-/Migrationsoberflächen.
  Die Laufzeit darf `hooks.internal.handlers` nicht laden; Hooks werden ausschließlich über erkannte
  Hook-Verzeichnisse und `HOOK.md`-Metadaten ausgeführt.
- Laufzeitstart, häufig durchlaufene Antwortpfade, Compaction, Zurücksetzung, Wiederherstellung, Diagnose,
  TTS, Speicher-Hooks, Unteragenten, Plugin-Befehlsrouting, Protokollgrenzen und
  Hooks müssen `{agentId, sessionId}` durch die Laufzeit weiterreichen.
- Tests sollten SQLite-Transkriptzeilen über
  `{agentId, sessionId}` anlegen und prüfen. Tests, die lediglich die Weiterleitung von JSONL-Pfaden,
  die Beibehaltung vom Aufrufer bereitgestellter Locators oder die Kompatibilität mit Transkriptdateien nachweisen, sollten
  gelöscht werden, sofern sie nicht den Doctor-Import, die Materialisierung von Support-/Debug-
  Material außerhalb von Sitzungen oder die Protokollstruktur abdecken.
- `runEmbeddedPiAgent(...)`, vorbereitete Worker-Ausführungen und der innere eingebettete
  Versuch dürfen keine Transkript-Locators akzeptieren. Sie öffnen den SQLite-Transkript-
  Manager über `{agentId, sessionId}` und übergeben diesen Manager an die internalisierte
  PI-kompatible Agentensitzung, sodass veraltete Aufrufer den Runner nicht zum Schreiben
  von JSON-/JSONL-Transkripten veranlassen können.
- Runner-Diagnosen müssen Laufzeit-, Cache- und Nutzdaten-Trace-Datensätze in SQLite speichern.
  Laufzeitdiagnosen dürfen keine Überschreibungsoptionen für JSONL-Dateien oder generischen
  Exporthilfen für Transkript-JSONL bereitstellen; benutzerseitige Exporte können explizite
  Artefakte aus Datenbankzeilen materialisieren, ohne Dateinamen wieder in die Laufzeit einzuspeisen.
- Die Rohdatenstromprotokollierung verwendet `OPENCLAW_RAW_STREAM=1` sowie SQLite-Diagnosezeilen.
  Der alte pi-mono-Dateiprotokollierungsvertrag aus `PI_RAW_STREAM`, `PI_RAW_STREAM_PATH` und
  `raw-openai-completions.jsonl` gehört weder zur OpenClaw-
  Laufzeit noch zu deren Tests.
- Die QMD-Speicherindizierung darf SQLite-Transkripte nicht in Markdown-Dateien exportieren.
  QMD indiziert ausschließlich konfigurierte Speicherdateien; die Suche in Sitzungstranskripten bleibt
  SQLite-basiert.
- Der QMD-SDK-Unterpfad ist bei neuem Code ausschließlich für QMD bestimmt. Hilfsfunktionen zur Indizierung
  von SQLite-Sitzungstranskripten befinden sich unter `memory-core-host-engine-session-transcripts`; jeder
  QMD-Reexport dient ausschließlich der Kompatibilität und darf nicht von Laufzeitcode verwendet werden.
- Integrierte Speicherindizes befinden sich in der zugehörigen Agentendatenbank. Laufzeitkonfiguration und
  aufgelöste Laufzeitverträge dürfen `memorySearch.store.path` nicht bereitstellen; Doctor
  löscht diesen alten Konfigurationsschlüssel, und aktueller Code übergibt intern die
  agenteneigene `databasePath`.

Bei der Implementierung sollte weiterhin Code entfernt werden, bis diese Aussagen
ohne Ausnahmen außerhalb der Doctor-/Import-/Export-/Debug-Grenzen zutreffen.

## Zielzustand und Fortschritt

### Verbindliches Ziel

- Eine globale SQLite-Datenbank verwaltet den Zustand der Steuerungsebene:
  `state/openclaw.sqlite`.
- Eine SQLite-Datenbank pro Agent verwaltet den Zustand der Datenebene:
  `agents/<agentId>/agent/openclaw-agent.sqlite`.
- Die Konfiguration bleibt dateibasiert. `openclaw.json` ist nicht Teil dieses Datenbank-
  Refactorings.
- Alte Dateien dienen ausschließlich als Eingaben für die Doctor-Migration.
- Die Laufzeit schreibt oder liest Sitzungs- oder Transkript-JSONL niemals als aktiven Zustand.

### Zielzustände

- `not-started`: Laufzeitcode aus der Dateiära schreibt weiterhin aktiven Zustand.
- `migrating`: Doctor-/Importcode kann Dateidaten nach SQLite verschieben.
- `dual-read`: Eine temporäre Brücke liest sowohl SQLite als auch alte Dateien. Dieser Zustand
  ist für dieses Refactoring verboten, sofern er nicht ausdrücklich als ausschließlich
  für Doctor bestimmt dokumentiert ist.
- `sqlite-runtime`: Die Laufzeit liest und schreibt ausschließlich SQLite.
- `clean`: Alte Laufzeit-APIs und Tests wurden entfernt, und die Schutzprüfung verhindert
  Regressionen.
- `done`: Dokumentation, Tests, Sicherung, Doctor-Migration und Prüfungen der Änderungen weisen den
  bereinigten Zustand nach.

### Aktueller Zustand

- Sitzungen: `clean` für die Laufzeit. Sitzungszeilen befinden sich in der Datenbank pro Agent,
  Laufzeit-APIs verwenden `{agentId, sessionId}` oder `{agentId, sessionKey}`, und
  `sessions.json` dient ausschließlich als alte Doctor-Eingabe.
- Transkripte: `clean` für die Laufzeit. Transkriptereignisse, Identitäten, Snapshots
  und Laufzeitereignisse von Trajektorien befinden sich in der Datenbank pro Agent. Die Laufzeit
  akzeptiert keine Transkript-Locators oder JSONL-Transkriptpfade mehr.
- Eingebetteter PI-Runner: `clean`. Eingebettete PI-Ausführungen, vorbereitete Worker, Compaction
  und Wiederholungsschleifen verwenden den SQLite-Sitzungsbereich und lehnen veraltete Transkript-Handles ab.
- Cron: `clean` für die Laufzeit. Die Laufzeit verwendet `cron_jobs` und Cron-eigene `task_runs`;
  Laufzeittests verwenden die SQLite-Namensgebung `storeKey`, und Cron-Pfade aus der Dateiära verbleiben
  ausschließlich in Tests der alten Doctor-Migration.
- Aufgabenregister: `clean`. Laufzeitzeilen für Aufgaben und Task Flow befinden sich in
  `state/openclaw.sqlite`; nicht veröffentlichte Importer für SQLite-Sidecars wurden gelöscht.
- Plugin-Zustand: `clean`. Plugin-Zustands-/Blob-Zeilen befinden sich in der gemeinsam genutzten globalen
  Datenbank; Schutzprüfungen verhindern alte SQLite-Hilfsfunktionen für Plugin-Zustands-Sidecars.
- Speicher: `sqlite-runtime` für den integrierten Speicher und die Indizierung von Sitzungstranskripten.
  Speicherindextabellen befinden sich in der Datenbank pro Agent, der Plugin-Speicherzustand verwendet
  gemeinsam genutzte Plugin-Zustandszeilen, und alte Speicherdateien sind Doctor-Migrationseingaben
  oder Inhalte des Benutzerarbeitsbereichs.
- Sicherung: `sqlite-runtime`. Die Sicherung stellt kompakte SQLite-Snapshots bereit, lässt aktive
  WAL-/SHM-Sidecars aus, überprüft die SQLite-Integrität und zeichnet Sicherungsläufe in der
  globalen Datenbank auf.
- Arbeitsbereichseinrichtung: `sqlite-runtime`. Der Abschluss der Einrichtung, Arbeitsbereichsbestätigungen
  und erzeugte Bootstrap-Hashes befinden sich in typisierten gemeinsam genutzten SQLite-Tabellen. Die Laufzeit
  liest oder schreibt weder die eingestellte Arbeitsbereichs-JSON noch `.attested`-Sidecars;
  Doctor ist für deren validierten Import und verifizierte Entfernung zuständig.
- Doctor-Migration: absichtlich `migrating`. Doctor importiert alte JSON-,
  JSONL- und eingestellte Sidecar-Speicher nach SQLite, zeichnet Migrationsläufe und -quellen auf
  und entfernt erfolgreich migrierte Quellen.
- Ausführungsgenehmigungen: `file-runtime`. TypeScript und macOS lesen und schreiben weiterhin
  `exec-approvals.json` des aktiven Zustandsverzeichnisses; das reservierte
  Schema `exec_approvals_config` hat noch keinen Laufzeiteigentümer. Eine zukünftige Umstellung muss
  einen Doctor-Import innerhalb desselben Zustands hinzufügen und beide Laufzeiten gemeinsam umstellen.
- E2E-Skripte: `clean` für die Laufzeitabdeckung. Das Docker-MCP-Seeding schreibt SQLite-
  Zeilen. Das Docker-Laufzeitkontextskript erstellt alte JSONL ausschließlich innerhalb des
  Doctor-Migrations-Seeds und benennt den alten Sitzungsindexpfad ausdrücklich.

### Verbleibende Arbeiten

- [x] Variablen für Cron-Laufzeittest-Speicher von `storePath` weg umbenennen, sofern
      sie keine alten Doctor-Eingaben sind.
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
- [x] Den alten JSONL-Seed des Docker-Laufzeitkontexts eindeutig als ausschließlich für Doctor bestimmt kennzeichnen.
      Datei: `scripts/e2e/session-runtime-context-docker-client.ts`.
      Nachweis: `rg -n 'sessions\\.json|sessionFile|\\.jsonl' scripts/e2e/session-runtime-context-docker-client.ts` zeigt ausschließlich
      `seedBrokenLegacySessionForDoctorMigration`.
- [x] Die von Kysely generierten Typen nach jeder Schemaänderung synchron halten.
      Dateien: `src/state/openclaw-state-schema.sql`,
      `src/state/openclaw-agent-schema.sql`,
      `src/state/*generated*`.
      Nachweis: keine Schemaänderung in diesem Durchlauf; `pnpm db:kysely:check`;
      `pnpm lint:kysely`.
- [x] Fokussierte Tests für betroffene Speicher, Befehle und Skripte erneut ausführen.
      Nachweis: `pnpm test src/cron/service/store.test.ts src/cron/store.test.ts src/cron/service.heartbeat-ok-summary-suppressed.test.ts src/cron/service.main-job-passes-heartbeat-target-last.test.ts src/cron/service.every-jobs-fire.test.ts src/cron/service.persists-delivered-status.test.ts src/cron/service.runs-one-shot-main-job-disables-it.test.ts src/cron/service/ops.test.ts src/cron/service/timer.regression.test.ts src/auto-reply/reply/commands-export-session.test.ts extensions/telegram/src/thread-bindings.test.ts extensions/slack/src/monitor/message-handler/prepare.test.ts src/acp/translator.session-lineage-meta.test.ts`; `git diff --check`.
- [x] Vor der Erklärung von `done` die Änderungsprüfung oder einen umfassenden Remote-Nachweis ausführen.
      Nachweis: `pnpm check:changed --timed -- <changed extension paths>` wurde beim
      Hetzner-Crabbox-Lauf `run_3f1cabf6b25c` nach einer temporären Node-24-/pnpm-Einrichtung und
      explizitem Pfadrouting für den synchronisierten Arbeitsbereich ohne `.git` erfolgreich ausgeführt.

### Regressionen vermeiden

- Keine Transkript-Locators.
- Keine aktiven Sitzungsdateien.
- Keine fingierten JSONL-Test-Fixtures außer in Tests alter Doctor-Migrationen.
- Kein direkter SQLite-Zugriff, wo Kysely erwartet wird.
- Keine neuen Datenbankmigrationen aus der Dateiära. Das globale Schema bleibt bei Version `1`.
  Das veröffentlichte Schema der Version `1` pro Agent besitzt genau eine begrenzte Laufzeitmigration auf
  Version `2` für stabile Speicherquellenidentitäten.

## Annahmen für die Codeanalyse

Keine ausstehenden Produktentscheidungen blockieren diesen Plan. Die Implementierung sollte
unter folgenden Annahmen fortgesetzt werden:

- Verwenden Sie `node:sqlite` direkt und setzen Sie für diesen Speicherpfad eine WAL-Reset-sichere Node-Laufzeit voraus
  (22.22.3+, 24.15+ oder 25.9+).
- Behalten Sie genau eine normale Konfigurationsdatei bei. Verschieben Sie bei diesem Refactoring weder die Konfiguration noch Plugin-
  Manifeste oder Git-Arbeitsbereiche nach SQLite.
- Laufzeitkompatibilitätsdateien sind nicht erforderlich. Alte JSON- und JSONL-Dateien dienen
  ausschließlich als Migrationseingaben. Die branch-lokalen SQLite-Sidecars wurden nie veröffentlicht und werden
  gelöscht, statt importiert.
- `openclaw doctor --fix` ist für die Migration alter Dateien in die Datenbank zuständig. Der Laufzeit-
  start ist nur für begrenzte Upgrades zwischen veröffentlichten SQLite-Schemaversionen zuständig;
  er darf keinen Zustand aus der Dateiära importieren.
- Für die Anmeldeinformationen gilt dieselbe Kompatibilitätsregel: Laufzeitanmeldeinformationen befinden sich in
  SQLite. Alte `auth-profiles.json`-, agentenspezifische `auth.json`- und gemeinsam genutzte
  `credentials/oauth.json`-Dateien dienen als Migrationseingaben für Doctor und werden
  nach dem Import entfernt.
- Der generierte Zustand des Modellkatalogs ist datenbankgestützt. Laufzeitcode darf
  `agents/<agentId>/agent/models.json` nicht schreiben; vorhandene `models.json`-Dateien sind alte
  Eingaben für Doctor und werden nach dem Import in `agent_model_catalogs` entfernt.
- Die Laufzeit darf Transkript-Locators weder migrieren noch normalisieren oder überbrücken. Die aktive
  Transkriptidentität ist `{agentId, sessionId}` in SQLite. Dateipfade dienen
  ausschließlich als alte Eingaben für Doctor, und `sqlite-transcript://...` muss aus
  Laufzeit-, Protokoll-, Hook- und Plugin-Oberflächen verschwinden, statt als
  Boundary-Handle behandelt zu werden.
- Beim Lesen von SQLite-Transkripten führt die Laufzeit keine alten Migrationen der JSONL-Eintragsform aus und
  schreibt nicht aus Kompatibilitätsgründen ganze Transkripte neu. Die Normalisierung alter Einträge verbleibt in
  expliziten Doctor-/Import-Dienstprogrammen. Doctor normalisiert alte JSONL-Transkript-
  dateien vor dem Einfügen der SQLite-Zeilen; aktuelle Laufzeitzeilen werden
  bereits im aktuellen Transkriptschema geschrieben. Der Trajektorien-/Sitzungsexport
  liest diese Zeilen unverändert und darf beim Export keine Altmigrationen durchführen.
- Hilfsfunktionen zum Parsen und Migrieren alter JSONL-Transkripte sind ausschließlich Doctor vorbehalten. Der Laufzeitcode
  für das Transkriptformat erstellt nur den aktuellen SQLite-Transkriptkontext; Doctor
  ist für Upgrades alter JSONL-Einträge vor dem Einfügen der Zeilen zuständig.
- Die alte, von der Laufzeit verwaltete Hilfsfunktion zum Streamen von JSONL-Transkripten wurde gelöscht. Der Doctor-
  Importcode ist für explizite Lesevorgänge alter Dateien zuständig; der Laufzeit-Sitzungsverlauf liest
  SQLite-Zeilen.
- Codex-App-Server-Bindings verwenden den OpenClaw-`sessionId` als kanonischen
  Schlüssel im Codex-Plugin-Zustandsnamensraum. `sessionKey` sind Metadaten für
  Routing/Anzeige und dürfen weder die dauerhafte Sitzungs-ID ersetzen noch
  die Identität der Transkriptdatei wiederbeleben.
- Kontext-Engines erhalten den aktuellen Laufzeitvertrag direkt. Die Registry
  darf Engines nicht mit Wiederholungs-Shims umschließen, die `sessionKey`,
  `transcriptScope` oder `prompt` löschen; Engines, die die aktuellen
  datenbankorientierten Parameter nicht akzeptieren können, sollten mit einem deutlichen Fehler abbrechen, statt überbrückt zu werden.
- Die Sicherungsausgabe sollte weiterhin aus einer einzelnen Archivdatei bestehen. Datenbankinhalte sollten
  als kompakte SQLite-Snapshots in dieses Archiv aufgenommen werden, nicht als rohe Live-WAL-Sidecars.
- Die Transkriptsuche ist nützlich, aber für den ersten datenbankorientierten
  Schnitt nicht erforderlich. Gestalten Sie das Schema so, dass FTS später hinzugefügt werden kann.
- Die Worker-Ausführung sollte hinter Einstellungen experimentell bleiben, während sich die Datenbank-
  grenze stabilisiert.

## Erkenntnisse aus der Codeanalyse

Der aktuelle Branch hat die Proof-of-Concept-Phase bereits hinter sich. Die gemeinsam genutzte
Datenbank ist vorhanden, Node `node:sqlite` ist über eine kleine Laufzeit-Hilfsfunktion eingebunden, und
frühere Speicher schreiben nun in `state/openclaw.sqlite` oder die zugehörige
`openclaw-agent.sqlite`-Datenbank.

Bei den verbleibenden Arbeiten geht es nicht um die Wahl von SQLite, sondern darum, die neue Grenze sauber zu halten
und alle kompatibilitätsorientierten Schnittstellen zu löschen, die noch der alten
Dateiwelt ähneln:

- Sitzungs-`storePath` ist weder eine Laufzeitidentität noch die Form einer Test-Fixture oder
  ein Feld der Statusnutzlast. Laufzeit- und Bridge-Tests enthalten den
  Vertragsnamen `storePath` nicht mehr; Doctor-/Migrationscode ist für dieses alte Vokabular zuständig.
- Sitzungsschreibvorgänge durchlaufen nicht mehr die alte prozessinterne `store-writer.ts`-
  Warteschlange. SQLite-Patch-Schreibvorgänge werden außerhalb der Transaktion vorbereitet und verwenden anschließend eine kurze
  synchrone Validierungs-/Anwendungstransaktion mit expliziter Konflikterkennung.
- Die Erkennung alter Pfade hat weiterhin gültige Migrationszwecke, der Laufzeitcode sollte
  `sessions.json` und JSONL-Transkriptdateien jedoch nicht mehr als mögliche Schreib-
  ziele behandeln.
- Agenteneigene Tabellen befinden sich in agentenspezifischen SQLite-Datenbanken. Die globale Datenbank enthält
  Registry-/Steuerungsebenenzeilen; die Transkriptidentität ist `{agentId, sessionId}` in
  den agentenspezifischen Transkriptzeilen. Laufzeitcode darf weder Transkriptdateipfade
  persistieren noch Transkript-Locators migrieren.
- Doctor importiert bereits mehrere alte Dateien. Die Bereinigung besteht darin, daraus eine
  einzelne explizite Migrationsimplementierung zu machen, die Doctor aufruft und die einen dauerhaften
  Migrationsbericht erstellt.

Keine weiteren Produktfragen blockieren die Implementierung.

## Aktuelle Codestruktur

Der Branch verfügt bereits über eine echte gemeinsam genutzte SQLite-Basis:

- Die Mindestanforderung an die Laufzeit setzt nun einen WAL-Reset-sicheren Node-Build voraus: 22.22.3+,
  24.15+ oder 25.9+. `package.json`, die Laufzeitprüfung der CLI, die Standardwerte des Installationsprogramms,
  die macOS-Laufzeitsuche, die CI und die öffentliche Installationsdokumentation stimmen überein.
- `src/state/openclaw-state-db.ts` öffnet `openclaw.sqlite`, legt WAL fest,
  `synchronous=NORMAL`, `busy_timeout=30000`, `foreign_keys=ON` und wendet
  das generierte Schemamodul an, das aus
  `src/state/openclaw-state-schema.sql` abgeleitet wurde.
- Kysely-Tabellentypen und Laufzeit-Schemamodule werden aus temporären
  SQLite-Datenbanken generiert, die anhand der eingecheckten `.sql`-Dateien erstellt werden; der Laufzeitcode
  enthält keine kopierten Schemastrings mehr für globale, agentenspezifische oder Proxy-
  Erfassungsdatenbanken.
- Laufzeitspeicher leiten ausgewählte und eingefügte Zeilentypen aus diesen generierten
  Kysely-`DB`-Schnittstellen ab, statt SQLite-Zeilenstrukturen manuell nachzubilden. Rohes SQL
  bleibt auf die Schemaanwendung, Pragmas und ausschließlich für Migrationen verwendete DDL beschränkt.
- Das globale SQLite-Schema verbleibt bei `user_version = 1`. Das agentenspezifische Schema
  hat Version `2`; seine Öffnungsroutine migriert den ausgelieferten Memory-Source-Schlüssel der Version `1`
  atomar zu einer stabilen ganzzahligen Identität. Der Import von Dateien in die Datenbank
  verbleibt im Doctor-Code.
- Die relationale Eigentümerschaft wird dort durchgesetzt, wo die Eigentumsgrenze maßgeblich ist:
  Zeilen der Quellenmigration werden beim Löschen aus `migration_runs` kaskadiert, der Zustellungsstatus von Aufgaben
  wird beim Löschen aus `task_runs` kaskadiert und Zeilen der Transkriptidentität werden beim Löschen von
  Transkriptereignissen kaskadiert.
- Zu den aktuellen gemeinsam genutzten Tabellen gehören `agent_databases`,
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
- Beliebiger Plugin-eigener Zustand erhält keine typisierten Tabellen im Eigentum des Hosts. Installierte
  Plugins verwenden `plugin_state_entries` für versionierte JSON-Nutzdaten und
  `plugin_blob_entries` für Bytes, mit Eigentümerschaft an Namespace und Schlüssel, TTL-Bereinigung,
  Sicherung und Plugin-Migrationsdatensätzen. Vom Host verwalteter Plugin-Orchestrierungszustand kann
  weiterhin typisierte Tabellen besitzen, wenn der Host für den Abfragevertrag verantwortlich ist, beispielsweise
  `plugin_binding_approvals`.
- Plugin-Migrationen sind Datenmigrationen über Plugin-eigene Namespaces und keine
  Host-Schemamigrationen. Ein Plugin kann seine eigenen versionierten Zustands-/Blob-Einträge
  über einen Migrations-Provider migrieren, und der Host zeichnet den Quellen-/Ausführungsstatus im
  regulären Migrationsprotokoll auf. Neue Plugin-Installationen erfordern keine Änderung an
  `openclaw-state-schema.sql`, es sei denn, der Host übernimmt selbst die Verantwortung für einen
  neuen Plugin-übergreifenden Vertrag.
- `src/state/openclaw-agent-db.ts` öffnet
  `agents/<agentId>/agent/openclaw-agent.sqlite`, registriert die Datenbank in der
  globalen Datenbank und verwaltet agentenlokale Sitzungs-, Transkript-, VFS-, Artefakt-, Cache-
  und Speicherindextabellen. Die Ermittlung zur Laufzeit aus gemeinsam genutztem Code liest nun die generiert typisierte
  `agent_databases`-Registrierung, statt diese Abfrage an jeder Aufrufstelle
  neu zu implementieren.
- Globale und agentenspezifische Datenbanken zeichnen eine `schema_meta`-Zeile mit Datenbankrolle,
  Schemaversion, Zeitstempeln und Agenten-ID für Agentendatenbanken auf. Die globale Datenbank
  verbleibt bei `user_version = 1`; agentenspezifische Datenbanken verwenden nach der begrenzten
  Migration der Memory-Source-Identität Version `2`.
- Die agentenspezifische Sitzungsidentität besitzt nun eine maßgebliche `sessions`-Stammtabelle mit
  `session_id` als Schlüssel sowie `session_key`, `session_scope`, `account_id`,
  `primary_conversation_id`, Zeitstempeln, Anzeigefeldern, Modellmetadaten,
  Harness-ID und Eltern-/Erzeugungsverknüpfung als abfragbaren Spalten. `session_routes`
  ist der eindeutige aktive Routenindex von `session_key` zur aktuellen
  `session_id`, sodass ein Routenschlüssel zu einer neuen dauerhaften Sitzung wechseln kann, ohne
  dass häufige Lesezugriffe zwischen doppelten `sessions.session_key`-Zeilen wählen müssen. Die alte,
  wie `session_entries.entry_json` geformte Kompatibilitätsnutzlast ist per Fremdschlüssel an die
  dauerhafte `session_id`-Wurzel angehängt; sie ist nicht mehr die einzige
  Darstellung einer Sitzung auf Schemaebene.
- Auch die agentenspezifische externe Konversationsidentität ist relational:
  `conversations` speichert die normalisierte Provider-/Konto-/Konversationsidentität und
  `session_conversations` verknüpft eine OpenClaw-Sitzung mit einer oder mehreren externen
  Konversationen. Dies deckt gemeinsam genutzte Haupt-DM-Sitzungen ab, bei denen mehrere Kommunikationspartner
  absichtlich derselben Sitzung zugeordnet werden können, ohne in `session_key` falsche Angaben zu machen. SQLite
  erzwingt außerdem die Eindeutigkeit der natürlichen Provider-Identität, sodass dasselbe Tupel aus
  Kanal/Konto/Art/Kommunikationspartner/Thread nicht über mehrere Konversations-IDs verteilt werden kann.
  Direkte Kommunikationspartner der gemeinsam genutzten Hauptsitzung werden mit der Rolle `participant` verknüpft, sodass eine
  OpenClaw-Sitzung mehrere externe DM-Kommunikationspartner darstellen kann, ohne
  ältere Kommunikationspartner zu unspezifischen verknüpften Zeilen herabzustufen. `sessions.primary_conversation_id`
  verweist weiterhin auf das aktuelle typisierte Zustellungsziel. Geschlossene Routing-/Statusspalten
  werden durch SQLite-`CHECK`-Constraints erzwungen, statt sich ausschließlich auf
  TypeScript-Unions zu verlassen.
  Die Laufzeitprojektion der Sitzung entfernt Routing-Schatten für die Kompatibilität aus
  `session_entries.entry_json`, bevor sie typisierte Sitzungs-/Konversationsspalten
  anwendet, sodass veraltete JSON-Nutzdaten keine Zustellungsziele wiederherstellen können.
  Auch das Ankündigungs-Routing von Subagenten erfordert den typisierten SQLite-Zustellungskontext;
  es greift nicht mehr auf Routenfelder für die Kompatibilität in `SessionEntry` zurück.
  Die explizite Zustellungsvererbung von Gateway `chat.send` liest den typisierten SQLite-
  Zustellungskontext anstelle der Kompatibilitätsfelder `origin`/`last*`.
  `tools.effective` leitet den Provider-/Konto-/Thread-Kontext ebenfalls aus typisierten
  SQLite-Zustellungs-/Routingzeilen ab und nicht aus veralteten Schatten der Sitzungseinträge in `last*`.
  Der Prompt-Kontext für Systemereignisse erstellt Kanal-/Ziel-/Konto-/Thread-Felder aus
  typisierten Zustellungsfeldern neu, statt Schatten in `origin` zu verwenden.
  Der gemeinsam genutzte `deliveryContextFromSession`-Helfer und die Zuordnung von Sitzungen zu Konversationen
  ignorieren `SessionEntry.origin` nun vollständig; nur typisierte Zustellungsfelder
  und relationale Konversationszeilen können eine Laufzeit-Routenidentität erzeugen.
  Die Normalisierung von Laufzeit-Sitzungseinträgen entfernt `origin`, bevor
  `entry_json` gespeichert oder projiziert wird, und eingehende Metadaten schreiben typisierte Kanal-/Chat-
  Felder sowie relationale Konversationszeilen, statt neue Ursprungsschatten
  zu erzeugen.
- Transkriptereignisse, Transkript-Snapshots und Laufzeitereignisse von Trajektorien
  verweisen nun auf die maßgebliche agentenspezifische `sessions`-Wurzel und werden beim Löschen der Sitzung
  kaskadiert. Zeilen für Transkriptidentität/Idempotenz werden weiterhin beim Löschen der
  exakten Transkriptereigniszeile kaskadiert.
- Memory-Core-Indizes verwenden nun die expliziten Agentendatenbanktabellen
  `memory_index_meta`, `memory_index_sources`, `memory_index_chunks` und
  `memory_embedding_cache`, wobei `memory_index_state` Revisionsänderungen verfolgt.
  Optionale FTS-/Vektor-Nebenindizes heißen `memory_index_chunks_fts` und
  `memory_index_chunks_vec` statt der generischen Tabellen `meta`, `files`, `chunks`,
  `chunks_fts` oder `chunks_vec`. Die maßgeblichen Namen behalten die aktuelle
  Pfad-/Quellenzeilenstruktur und die Kompatibilität serialisierter Einbettungen bei. Diese Tabellen
  sind abgeleitete Such-Caches und kein maßgeblicher Transkriptspeicher; sie können
  gelöscht und aus Dateien des Memory-Arbeitsbereichs und konfigurierten Quellen neu erstellt werden.
  Beim Öffnen eines ausgelieferten Memory-Indexes mit generischem Namen werden dessen Metadaten, Quellen,
  Chunks und Einbettungs-Cache in die maßgeblichen Tabellen migriert; abgeleitete FTS-/Vektor-
  Tabellen werden unter ihren maßgeblichen Namen neu erstellt.
- Der Wiederherstellungszustand von Subagent-Ausführungen befindet sich nun in typisierten gemeinsam genutzten `subagent_runs`-Zeilen
  mit indizierten Sitzungsschlüsseln für untergeordneten Agenten, Anforderer und Controller. Die alte
  `subagents/runs.json`-Datei dient nur noch als Eingabe für die Doctor-Bereinigung. Ihre Ausführungseinträge sind
  vorübergehender Wiederherstellungszustand, daher zeichnet Doctor den Stilllegungsbeleg auf und
  verwirft die Datei, ohne sie zu importieren. Da eine Datei nicht belegen kann, ob
  ihre Einträge nach dem Bereinigen von SQLite-Zeilen aktiv oder veraltet sind, müssen Betreiber
  aktive Ausführungen aus der Dateiära abschließen lassen, bevor sie über diese Grenze hinweg aktualisieren.
- Aktuelle Konversationsbindungen befinden sich nun in typisierten gemeinsam genutzten
  `current_conversation_bindings`-Zeilen, deren Schlüssel die normalisierte Konversations-ID ist und deren
  Zielagenten-/Sitzungsspalten, Konversationsart, Status, Ablaufzeit und Metadaten
  als relationale Spalten statt als doppelter opaker Bindungsdatensatz gespeichert werden.
  Der dauerhafte Bindungsschlüssel enthält die normalisierte Konversationsart, sodass
  Direkt-/Gruppen-/Kanalreferenzen nicht kollidieren können, und SQLite weist ungültige Werte für
  Bindungsart/-status zurück. Die alte
  `bindings/current-conversations.json`-Datei dient nur noch als Eingabe für die Doctor-Migration.
- Die Wiederherstellung der Zustellungswarteschlange legt nun typisierte Warteschlangenspalten für Kanal, Ziel,
  Konto, Sitzung, Wiederholungsversuch, Fehler, Plattformversand und Wiederherstellungsstatus über das
  Replay-JSON. `entry_json` behält die Replay-Nutzdaten, Hooks und Formatierungs-
  nutzlast bei, doch die typisierten Spalten sind für das Laufzeit-Routing und den Zustand der Warteschlange maßgeblich.
- Zeiger zum Wiederherstellen der letzten TUI-Sitzung befinden sich nun in typisierten gemeinsam genutzten
  `tui_last_sessions`-Zeilen, deren Schlüssel der Hash des TUI-Verbindungs-/Sitzungsbereichs ist.
  Die Laufzeit liest und schreibt ausschließlich SQLite, führt für jeden Bereich ein atomares Upsert durch und
  schließt Heartbeat-Sitzungen aus. `openclaw doctor --fix` validiert die
  alte TUI-JSON-Datei strikt, behält neuere SQLite-Zeilen bei, überprüft das maßgebliche Ergebnis
  und entfernt die unveränderte Legacy-Datei, statt ein Archiv zurückzulassen.
- Hashes für die Bereitstellung von Discord-Befehlen befinden sich nun im gemeinsam genutzten SQLite-
  Plugin-Zustandsspeicher. Die Laufzeit liest und schreibt ausschließlich exakte anwendungsspezifische Schlüssel. Doctor
  löscht die wiederherstellbare Legacy-Datei `discord/command-deploy-cache.json`,
  ohne sie zu importieren, sodass beim nächsten Start genau ein maßgeblicher Abgleich erfolgt.
- Standardmäßige TTS-Einstellungen befinden sich nun in gemeinsam genutzten SQLite-Zeilen des Plugin-Zustands, die dem
  Plugin `speech-core` zugeordnet sind. Die alte Datei `settings/tts.json` dient nur noch als Eingabe für die Doctor-Migration;
  die Laufzeit liest oder schreibt keine JSON-Dateien mit TTS-Einstellungen mehr und der
  Legacy-Pfadauflöser befindet sich im Doctor-Migrationsmodul.
- Metadaten für Secret-Ziele beziehen sich nun auf Speicher, statt so zu tun, als sei jedes
  Ziel für Anmeldedaten eine Konfigurationsdatei. `openclaw.json` bleibt der Konfigurationsspeicher;
  Authentifizierungsprofilziele verwenden typisierte SQLite-`auth_profile_stores`-Zeilen, wobei
  Provider-spezifisch strukturierte Anmeldedaten als JSON-Nutzdaten gespeichert werden.
- Die Secret-Prüfung durchsucht eingestellte agentenspezifische `auth.json`-Dateien nicht mehr. Doctor ist
  für die Warnung vor dieser Legacy-Datei sowie ihren Import und ihre Entfernung verantwortlich.
- Legacy-Pfadhelfer für Authentifizierungsprofile befinden sich nun im Doctor-Legacy-Code. Die Pfadhelfer für
  Kern-Authentifizierungsprofile stellen SQLite-Identitäten und Anzeigeorte des Authentifizierungsspeichers bereit,
  nicht die Laufzeitpfade `auth-profiles.json` oder `auth-state.json`.
- Die Laufzeitmodule für die Wiederherstellung von Subagent-Ausführungen und den OpenRouter-Modellfunktions-Cache
  halten SQLite-Snapshot-Lese-/Schreibvorgänge nun von ausschließlich für Doctor bestimmten Legacy-JSON-
  Importhelfern getrennt. OpenRouter-Funktionen verwenden die typisierten generischen
  `model_capability_cache`-Zeilen unter `provider_id = "openrouter"` statt
  eines opaken Cache-Blobs oder einer Provider-spezifischen Hosttabelle. Der `taskName` einer Subagent-Ausführung
  wird in der typisierten Spalte `subagent_runs.task_name` gespeichert; die
  Kopie in `payload_json` sind Replay-/Debugdaten und nicht die Quelle für häufig verwendete Anzeige- oder
  Suchfelder.
- `src/agents/filesystem/virtual-agent-fs.sqlite.ts` implementiert ein SQLite-VFS
  über der Agentendatenbanktabelle `vfs_entries`. Verzeichnislesevorgänge, rekursive
  Exporte, Löschvorgänge und Umbenennungen verwenden indizierte `(namespace, path)`-Präfixbereiche,
  statt einen gesamten Namespace zu durchsuchen oder sich auf die Pfadübereinstimmung von `LIKE` zu verlassen.
- `src/agents/runtime-worker.entry.ts` erstellt für jeden Lauf SQLite-VFS-, Tool-Artefakt-, Laufartefakt- und bereichsbezogene Cache-Speicher für Worker.
- Der Abschluss des Workspace-Bootstraps, die Aktualität der Attestierung und die generierten Bootstrap-Hashes befinden sich jetzt in typisierten, gemeinsam genutzten `workspace_setup_state`-, `workspace_path_aliases`-, `workspace_attestations`- und `workspace_generated_bootstrap_hashes`-Zeilen, die nach der kanonischen Workspace-Identität verschlüsselt sind. Persistierte lexikalische Aliasse und Aliasse für reale Pfade sorgen dafür, dass der Schutz vor verschwundenen Workspaces stabil bleibt, nachdem ein konfigurierter symbolischer Link verschwindet; neu ausgerichtete Aliasse führen zu einem sicheren Abbruch. Die Laufzeit liest oder schreibt `openclaw-workspace-state.json`, `.openclaw/workspace-state.json`, `workspace-attestations/*.attested` im Zustandsverzeichnis oder gleichgeordnete `<workspace>.attested`-Sidecars nicht mehr. `openclaw doctor --fix` validiert und beansprucht Legacy-Quellen, importiert sie mit Migrationsbelegen in SQLite, überprüft die kanonischen Zeilen und entfernt erst dann die beanspruchten Dateien.
- Das gemeinsam genutzte Schema reserviert eine `exec_approvals_config`-Singleton-Zeile, die Laufzeitumstellung steht jedoch noch aus. TypeScript und die macOS-Begleitanwendung verwenden weiterhin die zustandsbezogene JSON-Datei und müssen gemeinsam zu SQLite migriert werden.
- Die TypeScript-Geräteidentität verwendet jetzt typisierte `device_identities`-Zeilen, wobei der ausschließlich Doctor vorbehaltene Import von Legacy-JSON außerhalb des Laufzeit-Owners verbleibt. Die Geräteauthentifizierung bleibt bis zu einer koordinierten Schema- und laufzeitübergreifenden Migration dateibasiert; `device_auth_tokens` bleibt für diese Folgearbeit reserviert.
- Der Cache für den GitHub-Copilot-Tokenaustausch verwendet die gemeinsam genutzte SQLite-Tabelle für den Plugin-Zustand unter `github-copilot/token-cache/default`. Es handelt sich um Provider-eigenen Cache-Zustand, daher wird absichtlich keine Host-Schematabelle hinzugefügt.
- Die GitHub-Copilot-Compaction schreibt keine `openclaw-compaction-*.json`-Workspace-Sidecars mehr. Das Testsystem ruft für die nachverfolgte SDK-Sitzung den RPC zur SDK-Verlaufs-Compaction auf, und OpenClaw speichert dauerhafte Sitzungs- und Transkriptzustände in SQLite statt in Kompatibilitäts-Markierungsdateien.
- Die gemeinsam genutzte Swift-Laufzeit (`OpenClawKit`) verwendet dieselbe `state/openclaw.sqlite#table/device_identities`-Struktur und dieselben Zeilenschlüssel für die Geräteidentität. Legacy-Dateien in Apple-Containern werden vom Swift-Migrations-Owner importiert, da TypeScript Doctor nicht auf diese Container zugreifen kann. Die Swift-Geräteauthentifizierung bleibt für die koordinierte Folgearbeit an der Authentifizierung dateibasiert.
- Die Android-Geräteidentität und die zwischengespeicherte Geräteauthentifizierung bleiben anwendungsinterne Speicher. Sie erfordern eine separate, Android-eigene Migration; die Host-SQLite-Ansprüche beschreiben nicht das aktuelle Android-Verhalten.
- Der Verlauf der zuletzt verwendeten Pakete für Android-Benachrichtigungen verwendet typisierte `android_notification_recent_packages`-Zeilen. Die Laufzeit migriert oder liest die alten CSV-Schlüssel in SharedPreferences nicht mehr.
- Die Erstellung der Geräteidentität bricht sicher ab, wenn das Legacy-Element `identity/device.json` vorhanden ist, wenn die SQLite-Identitätszeile ungültig ist oder wenn der SQLite-Identitätsspeicher nicht geöffnet werden kann. Doctor importiert und entfernt diese Datei zuerst, sodass der Laufzeitstart die Kopplungsidentität vor der Migration nicht unbemerkt wechseln kann.
- Die Auswahl der Geräteidentität ist ein SQLite-Zeilenschlüssel und kein Locator für eine JSON-Datei. Tests und Gateway-Hilfsfunktionen übergeben explizite Identitätsschlüssel; nur die Doctor-Migration und die Startprüfung mit sicherem Abbruch kennen den stillgelegten Dateinamen `identity/device.json`.
- Die Kompatibilität beim Zurücksetzen von Sitzungen befindet sich jetzt in der Doctor-Konfigurationsmigration: `session.idleMinutes` wird nach `session.reset.idleMinutes` verschoben, `session.resetByType.dm` wird nach `session.resetByType.direct` verschoben, und die Laufzeitrichtlinie zum Zurücksetzen liest nur kanonische Rücksetzungsschlüssel.
- Die Kompatibilität mit Legacy-Konfigurationen befindet sich jetzt unter `src/commands/doctor/`. Die normale `readConfigFileSnapshot()`-Validierung importiert weder Doctor-Detektoren für Legacy-Konfigurationen noch versieht sie Legacy-Probleme mit Anmerkungen; `runDoctorConfigPreflight()` fügt diese Probleme für die Reparatur und Berichterstattung durch Doctor hinzu. Der Doctor-Konfigurationsablauf importiert `src/commands/doctor/legacy-config.ts`, und die Reparatur alter OAuth-Profil-IDs befindet sich unter `src/commands/doctor/legacy/oauth-profile-ids.ts`.
- Befehle außerhalb von Doctor führen die Reparatur von Legacy-Konfigurationen nicht automatisch aus. Beispielsweise schlägt `openclaw update --channel` jetzt bei einer ungültigen Legacy-Konfiguration fehl und fordert den Benutzer auf, Doctor auszuführen, statt den Doctor-Migrationscode unbemerkt zu importieren.
- Web-Push, APNs, Voice Wake, Aktualisierungsprüfungen und der Konfigurationszustand verwenden jetzt typisierte, gemeinsam genutzte SQLite-Tabellen für Abonnements, VAPID-Schlüssel, Node-Registrierungen, Triggerzeilen, Routingzeilen, den Zustand von Aktualisierungsbenachrichtigungen und Einträge zum Konfigurationszustand statt vollständiger undurchsichtiger JSON-Blobs. Schreibvorgänge von Web Push und APNs führen nur für die betroffene Primärschlüsselzeile ein Upsert aus; der Konfigurationszustand wird anhand des Konfigurationspfads abgeglichen. Ihre Laufzeitmodule bleiben von den ausschließlich Doctor vorbehaltenen Hilfsfunktionen zum Import von Legacy-JSON getrennt.
- Die APNs-Laufzeit liest und schreibt ausschließlich `apns_registrations`. `openclaw doctor --fix` importiert explizit und strikt das stillgelegte `push/apns-registrations.json`, behält vorhandene kanonische Zeilen bei, überprüft die Transaktion, zeichnet einen Beleg auf und entfernt die JSON-Datei mit vertraulichen Daten. Beleggestützte Wiederholungsversuche führen nur die Bereinigung durch, während `apns_registration_tombstones` Invalidierungen vor der ersten Reparatur abdecken, sodass veraltete Relay-Berechtigungen oder Geräte-Token nicht wiederhergestellt werden können.
- Die Node-Host-Konfiguration verwendet jetzt eine typisierte Singleton-Zeile in der gemeinsam genutzten SQLite-Datenbank. Die Laufzeit bricht sicher ab, solange die alte Datei `node.json` oder ein unterbrochener Anspruch vorhanden ist; `openclaw doctor --fix` importiert und entfernt sie explizit und strikt vor der normalen Laufzeitverwendung.
- Geräte-/Node-Kopplung, Kanalkopplung, Kanal-Zulassungslisten und Bootstrap-Zustand verwenden jetzt typisierte SQLite-Zeilen statt vollständiger undurchsichtiger JSON-Blobs. Genehmigungen für Plugin-Bindungen und der Zustand von Cron-Aufträgen folgen derselben Aufteilung: Laufzeitmodule stellen SQLite-gestützte Operationen und neutrale Snapshot-Hilfsfunktionen bereit, und Snapshot-Schreibvorgänge für Kopplung/Bootstrap sowie Genehmigungen von Plugin-Bindungen gleichen Zeilen anhand des Primärschlüssels ab, statt Tabellen zu leeren, während Doctor die alten JSON-Dateien über `src/commands/doctor/legacy/*`-Module importiert und entfernt.
- Datensätze installierter Plugins befinden sich jetzt im SQLite-Index installierter Plugins. Das Lesen und Schreiben der Laufzeitkonfiguration migriert oder bewahrt keine alten `plugins.installs`-Daten aus der manuell erstellten Konfiguration mehr; Doctor importiert diese Legacy-Konfigurationsstruktur vor der normalen Laufzeitverwendung in SQLite.
- Snapshots zur Wiederherstellung von QQBot-Anmeldedaten befinden sich jetzt im SQLite-Plugin-Zustand unter `qqbot/credential-backups`. Die Laufzeit schreibt `qqbot/data/credential-backup*.json` nicht mehr; der QQBot-Doctor-Vertrag importiert und archiviert diese Legacy-Sicherungsdateien aus dem aktiven Zustandsverzeichnis.
- Die Gateway-Neuladeplanung vergleicht Snapshots des SQLite-Indexes installierter Plugins unter einem internen `installedPluginIndex.installRecords.*`-Diff-Namensraum. Entscheidungen zum Neuladen der Laufzeit verpacken diese Zeilen nicht mehr in vorgetäuschte `plugins.installs`-Konfigurationsobjekte.
- Die Anmeldedaten für Matrix-Konten befinden sich jetzt im SQLite-Plugin-Zustand. Die Laufzeit liest nur diesen kanonischen Speicher; Doctor importiert, überprüft und archiviert stillgelegte `credentials/matrix/credentials*.json`-Dateien, wenn das zugehörige Konto aufgelöst werden kann.
- Die Kernlaufzeitmodule für Kopplung und Cron verwenden keine Legacy-JSON-Pfadgeneratoren mehr. Die veraltete SDK-Hilfsfunktion für Kopplungspfade bleibt ausschließlich als Migrationskompatibilität erhalten; die Doctor-Zustandsmigration ist für ihre Dateilesevorgänge und Importe zuständig. Doctor-eigene Legacy-Module erstellen die Quellpfade `pending.json`, `paired.json`, `bootstrap.json` und `cron/jobs.json` ausschließlich für Importtests und die Migration. Die Normalisierung der Legacy-Struktur von Cron-Aufträgen und der JSONL-Verlaufsimport befinden sich unter `src/commands/doctor/cron/`; die Finalisierung des Legacy-SQLite-Verlaufs erfolgt beim Öffnen der Zustandsdatenbank.
- `src/commands/doctor/legacy/runtime-state.ts` importiert Legacy-JSON-Zustandsdateien einschließlich der Node-Host-Konfiguration über Doctor in SQLite. Neue Importfunktionen für Legacy-Dateien verbleiben unter `src/commands/doctor/legacy/`.
- `src/commands/doctor/state-migrations.ts` importiert Legacy-Transkripte aus `sessions.json` und `*.jsonl` direkt in SQLite und entfernt erfolgreich importierte Quellen. Root-Legacy-Transkripte werden nicht mehr über `agents/<agentId>/sessions/*.jsonl` zwischengelagert, und vor dem Import wird kein kanonisches JSONL-Ziel mehr erstellt.
- Doctor-Prüfungen der Zustandsintegrität durchsuchen keine Legacy-Sitzungsverzeichnisse mehr und bieten keine Löschung verwaister JSONL-Dateien mehr an. Legacy-Transkriptdateien dienen ausschließlich als Migrationseingaben, und der Migrationsschritt ist sowohl für den Import als auch für die Entfernung der Quelle zuständig.
- Der Import der Legacy-Sandbox-Registry befindet sich unter `src/commands/doctor/legacy/sandbox-registry.ts`; aktive Lese- und Schreibvorgänge der Sandbox-Registry erfolgen weiterhin ausschließlich über SQLite.
- Die Reparatur für Zustandsprüfung und Import von Legacy-Sitzungstranskripten befindet sich unter `src/commands/doctor/legacy/session-transcript-health.ts`; Laufzeitbefehlsmodule enthalten keine Logik mehr zum Parsen von JSONL-Transkripten oder zum Reparieren des aktiven Branches.

Highlights der abgeschlossenen Konsolidierungen/Löschungen:

- Der Plugin-Zustand verwendet jetzt die gemeinsam genutzte `state/openclaw.sqlite`-Datenbank. Der alte
  branch-lokale `plugin-state/state.sqlite`-Sidecar-Importer wurde entfernt, weil
  dieses SQLite-Layout nie ausgeliefert wurde. Prüf-/Test-Hilfsfunktionen melden die gemeinsam genutzte
  `databasePath`, statt einen Plugin-Zustands-spezifischen SQLite-Pfad offenzulegen.
- Die Runtime-Tabellen für Tasks und TaskFlow befinden sich jetzt in der gemeinsam genutzten
  `state/openclaw.sqlite`-Datenbank statt in `tasks/runs.sqlite` und
  `tasks/flows/registry.sqlite`; die alten Sidecar-Importer wurden aus demselben Grund
  des nicht ausgelieferten Layouts entfernt.
- `src/config/sessions/store.ts` benötigt `storePath` nicht mehr für eingehende
  Metadaten, Routenaktualisierungen oder Lesezugriffe auf den Aktualisierungszeitpunkt. Befehlspersistenz, CLI-
  Sitzungsbereinigung, Subagent-Tiefe, Authentifizierungsüberschreibungen und die Sitzung-
  Identität des Transkripts verwenden Agent-/Sitzungszeilen-APIs. Schreibvorgänge werden als SQLite-Zeilen-Patches
  mit optimistischer Konfliktwiederholung angewendet.
- Die Auflösung von Sitzungszielen stellt jetzt Datenbankziele pro Agent bereit, nicht veraltete
  `sessions.json`-Pfade. Gemeinsam genutztes Gateway, ACP-Metadaten, Doctor-Routenreparatur und
  `openclaw sessions` führen `agent_databases` sowie konfigurierte Agenten auf.
- Das Gateway-Sitzungsrouting verwendet jetzt `resolveGatewaySessionDatabaseTarget`; das
  zurückgegebene Ziel enthält `databasePath` und mögliche SQLite-Zeilenschlüssel statt
  eines veralteten Dateipfads zum Sitzungsspeicher.
- Die Runtime-Typen für Channel-Sitzungen stellen jetzt `{agentId, sessionKey}` für
  Lesezugriffe auf den Aktualisierungszeitpunkt, eingehende Metadaten und Aktualisierungen der letzten Route bereit. Der alte
  Kompatibilitätstyp `saveSessionStore(storePath, store)` wurde entfernt.
- Die Sitzungsoberflächen der Plugin-Runtime, Erweiterungs-API und des Plugin-SDK stellen jetzt
  SQLite-gestützte Hilfsfunktionen für Sitzungszeilen statt datei-/gesamtspeicherbezogener
  Kompatibilitätshilfen für aktive Sitzungen bereit. Kompatibilitätsexporte der Root-Bibliothek bleiben
  nur außerhalb des Plugin-SDK für veraltete interne Aufrufer und Migrationsaufrufer verfügbar. Die alte
  Hilfsfunktion `resolveLegacySessionStorePath` wurde entfernt; die veraltete Konstruktion von
  `sessions.json`-Pfaden erfolgt jetzt lokal in Migrations- und Test-Fixtures.
- `src/config/sessions/session-entries.sqlite.ts` speichert kanonische Sitzungs-
  einträge jetzt in der Datenbank pro Agent und unterstützt Lesen, Upsert, Löschen und Patchen
  auf Zeilenebene. Upsert-, Patch- und Löschvorgänge der Runtime suchen nicht mehr nach Varianten der Groß-/Kleinschreibung und
  bereinigen keine veralteten Alias-Schlüssel mehr; Doctor ist für die Kanonisierung zuständig. Die
  eigenständige JSON-Importhilfsfunktion wurde entfernt, und die Migration führt neuere Zeilen per Upsert zusammen,
  statt die gesamte Sitzungstabelle zu ersetzen. Öffentliche Hilfsfunktionen zum Lesen, Auflisten und Laden
  projizieren häufig benötigte Sitzungsmetadaten aus typisierten `sessions`- und `conversations`-Zeilen;
  `entry_json` ist eine Kompatibilitäts-/Debug-Schattenkopie und kann veraltet oder ungültig sein,
  ohne dass die typisierte Sitzungsidentität oder der Zustellungskontext verloren geht.
- `src/config/sessions/delivery-info.ts` löst den Zustellungskontext jetzt aus den
  typisierten agentenspezifischen Zeilen `sessions` + `conversations` + `session_conversations` auf.
  Die Runtime-Zustellungsidentität wird nicht mehr aus
  `session_entries.entry_json` rekonstruiert; eine fehlende typisierte Konversationszeile ist ein
  Doctor-Migrations-/Reparaturproblem und kein Runtime-Fallback.
- Entscheidungen zum Zurücksetzen gespeicherter Sitzungen bevorzugen jetzt typisierte Metadaten aus `sessions.session_scope`,
  `sessions.chat_type` und `sessions.channel`. Das Parsen von `sessionKey`
  bleibt nur für explizite Thread-/Themen-Suffixe an Befehlszielen bestehen; die Klassifizierung von Zurücksetzungen als Gruppe oder
  Direktnachricht wird nicht mehr aus der Schlüsselform abgeleitet.
- Die Klassifizierung der Sitzungslisten-/Statusanzeige verwendet jetzt typisierte Chat-Metadaten und
  die Gateway-Sitzungsart. Teilzeichenfolgen `:group:` oder `:channel:`
  innerhalb von `session_key` werden nicht mehr als dauerhafte Aussage über Gruppe oder Direktnachricht behandelt.
- Die Auswahl der Richtlinie für stille Antworten verwendet jetzt ausschließlich einen expliziten Konversationstyp oder Oberflächen-
  Metadaten. Die Direkt-/Gruppenrichtlinie wird nicht mehr anhand von
  `session_key`-Teilzeichenfolgen geschätzt.
- Die Auflösung des Modells für die Sitzungsanzeige erhält die Agent-ID jetzt vom SQLite-
  Sitzungsdatenbankziel, statt sie aus `session_key` herauszutrennen.
- Das Hydratisieren des Ankündigungsziels zwischen Agenten verwendet jetzt ausschließlich die typisierten `sessions.list`-
  `deliveryContext`. Channel-/Konto-/Thread-Routing wird nicht mehr
  aus dem veralteten `origin`, gespiegelten `last*`-Feldern oder der Form von `session_key` wiederhergestellt.
- Die Ablehnung von Thread-Zielen durch `sessions_send` liest jetzt typisierte SQLite-Routing-
  Metadaten. Ziele werden nicht mehr durch das Parsen von Thread-Suffixen
  aus dem Zielschlüssel abgelehnt oder akzeptiert.
- Die Validierung gruppenbezogener Tool-Richtlinien liest jetzt typisiertes SQLite-Konversations-
  routing für die aktuelle oder erzeugte Sitzung. Der Gruppen-/Channel-
  Identität wird nicht mehr durch Dekodieren von `sessionKey` vertraut; vom Aufrufer bereitgestellte Gruppen-IDs werden verworfen, wenn
  keine typisierte Sitzungszeile sie bestätigt.
- Der Abgleich von Channel-Modellüberschreibungen verwendet jetzt explizite Gruppen- und übergeordnete
  Konversationsmetadaten. Übergeordnete Konversations-IDs werden nicht mehr aus
  `parentSessionKey` dekodiert.
- Die Vererbung gespeicherter Modellüberschreibungen erfordert jetzt einen expliziten Schlüssel der übergeordneten Sitzung
  aus einem typisierten Sitzungskontext. Übergeordnete Überschreibungen werden nicht mehr aus
  `:thread:`- oder `:topic:`-Suffixen in `sessionKey` abgeleitet.
- Der alte Wrapper für Sitzungs-Thread-Informationen und der Thread-Parser für geladene Plugins wurden entfernt;
  kein Runtime-Code importiert mehr `config/sessions/thread-info`.
- Die Channel-Konversationshilfsfunktion stellt keine Parsing-
  Brücken für vollständige Sitzungsschlüssel mehr bereit. Der Core normalisiert weiterhin Provider-eigene rohe Konversations-IDs über
  `resolveSessionConversation(...)`, rekonstruiert jedoch keine Routeninformationen
  aus `sessionKey`.
- Abschlusszustellung, Senderichtlinie und Task-Wartung leiten den Chat-
  Typ nicht mehr aus der Form von `session_key` ab. Der alte Schlüsselparser für Chat-Typen wurde gelöscht;
  diese Pfade benötigen typisierte Sitzungsmetadaten, einen typisierten Zustellungskontext oder
  ein explizites Vokabular für Zustellungsziele.
- Sitzungsliste/-status, Diagnose, Kontobindung für Genehmigungen, TUI-Heartbeat-
  Filterung und Nutzungsübersichten durchsuchen `SessionEntry.origin` nicht mehr nach
  Provider-/Konto-/Thread-/Anzeige-Routing. Die einzigen verbleibenden Runtime-
  Lesezugriffe auf `origin` betreffen Konzepte außerhalb von Sitzungen oder Zustellungsobjekte des aktuellen Durchlaufs.
- Die native Konversationssuche für Genehmigungsanfragen liest jetzt typisierte agentenspezifische Sitzungs-
  Routingzeilen. Die Channel-/Gruppen-/Thread-Konversationsidentität wird nicht mehr
  aus `sessionKey` geparst; fehlende typisierte Metadaten sind ein Migrations-/Reparaturproblem.
- Die Nutzdaten der Gateway-Ereignisse „Sitzung geändert“, „Chat“ und „Sitzung“ spiegeln
  keine Routenschatten `SessionEntry.origin` oder `last*` mehr; Clients erhalten typisierte
  `channel`, `chatType` und `deliveryContext`.
- Die Auflösung der Heartbeat-Zustellung kann jetzt die typisierte SQLite-
  `deliveryContext` direkt empfangen, und die Heartbeat-Runtime übergibt die agentenspezifische
  Sitzungszustellungszeile, statt sich für das aktuelle Routing auf Kompatibilitäts-
  Schattenkopien von `session_entries` zu verlassen.
- Die Auflösung des Zustellungsziels für isolierte Cron-Agenten hydratisiert ihre aktuelle
  Route ebenfalls aus der typisierten agentenspezifischen Sitzungszustellungszeile, bevor auf die
  Nutzdaten des Kompatibilitätseintrags zurückgegriffen wird.
- Die Auflösung des Ursprungs von Subagent-Ankündigungen reicht jetzt den typisierten Zustellungs-
  kontext der anfordernden Sitzung durch `loadRequesterSessionEntry` weiter und bevorzugt diese Zeile gegenüber
  den Kompatibilitätsschatten `last*`/`deliveryContext`.
- Aktualisierungen eingehender Sitzungsmetadaten werden jetzt zuerst mit der typisierten agentenspezifischen
  Zustellungszeile zusammengeführt; alte `SessionEntry`-Zustellungsfelder dienen nur als Fallback,
  wenn keine typisierte Konversationszeile vorhanden ist.
- Bei der Extraktion der Neustart-/Aktualisierungszustellung hat die typisierte SQLite-Zustellung
  `threadId` jetzt Vorrang vor Themen-/Thread-Fragmenten, die aus `sessionKey` geparst wurden; das Parsen
  dient nur als Fallback für veraltete Schlüssel in Thread-Form.
- Channel-IDs im Hook-Agentenkontext bevorzugen jetzt die typisierte SQLite-Konversationsidentität,
  danach explizite Nachrichtenmetadaten. Provider-/Gruppen-/Channel-
  Fragmente werden nicht mehr aus `sessionKey` geparst.
- Die Vererbung externer Routen durch Gateway `chat.send` liest jetzt typisierte SQLite-Sitzungs-
  Routingmetadaten, statt Channel-/Direkt-/Gruppenbereich aus
  Teilen von `sessionKey` abzuleiten. Channel-bezogene Sitzungen erben nur, wenn der typisierte
  Sitzungs-Channel und Chat-Typ mit dem gespeicherten Zustellungskontext übereinstimmen; gemeinsam genutzte Haupt-
  sitzungen behalten ihre strengere Regel für CLI/fehlende Client-Metadaten bei.
- Das Aufwecken durch Neustart-Sentinels und das Routing von Fortsetzungen lesen jetzt typisierte SQLite-
  Zustellungs-/Routingzeilen, bevor Heartbeat-Aufweckvorgänge oder geroutete Fortsetzungen von Agenten-Durchläufen
  in die Warteschlange gestellt werden. Der Zustellungskontext wird nicht mehr aus der
  JSON-Schattenkopie des Sitzungseintrags rekonstruiert.
- Die Kontextauflösung von Gateway `tools.effective` liest jetzt typisierte SQLite-
  Zustellungs-/Routingzeilen für Provider, Konto, Ziel, Thread und Antwortmodus-
  Eingaben. Diese häufig benötigten Routingfelder werden nicht mehr aus veralteten
  `session_entries.entry_json`-Ursprungsschatten wiederhergestellt.
- Das Routing für Echtzeit-Sprachkonsultationen löst die Zustellung für übergeordnete Sitzung/Anruf jetzt aus typisierten
  agentenspezifischen SQLite-Sitzungszeilen auf. Bei der Auswahl der Route für die eingebettete Agenten-
  nachricht wird nicht mehr auf Kompatibilitätsschatten von `SessionEntry.deliveryContext`
  zurückgegriffen.
- Das Heartbeat-Relay beim ACP-Spawn und das Routing des übergeordneten Streams lesen die Zustellung der übergeordneten Sitzung jetzt
  aus typisierten SQLite-Sitzungszeilen. Der Zustellungs-
  kontext der übergeordneten Sitzung wird nicht mehr aus Kompatibilitätsschatten des Sitzungseintrags rekonstruiert.
- Die Beibehaltung der Sitzungszustellungsroute folgt jetzt typisierten Chat-Metadaten und
  persistenten Zustellungsspalten. Channel-Hinweise, Direkt-/Haupt-
  markierungen oder Thread-Form werden nicht mehr aus `sessionKey` extrahiert; interne Webchat-Routen
  erben ein externes Ziel nur, wenn SQLite bereits eine typisierte/persistente Zustellungs-
  identität für die Sitzung enthält.
- Die generische Extraktion der Sitzungszustellung liest jetzt ausschließlich die exakte typisierte SQLite-
  Sitzungszustellungszeile. Thread-/Themen-Suffixe werden nicht mehr geparst, und es wird nicht mehr
  von einem Schlüssel in Thread-Form auf einen Basissitzungsschlüssel zurückgegriffen.
- Antwortversand, Wiederherstellung durch Neustart-Sentinels und Routing von Echtzeit-Sprachkonsultationen
  verwenden jetzt exakte typisierte SQLite-Sitzungs-/Konversationszeilen für das Thread-Routing. Sie
  stellen Thread-IDs oder den Zustellungskontext der Basissitzung nicht mehr durch Parsen
  von Sitzungsschlüsseln in Thread-Form wieder her.
- Die Begrenzung des eingebetteten PI-Verlaufs verwendet jetzt die typisierte SQLite-Sitzungsrouting-
  projektion (`sessions` + primäres `conversations`) für Provider, Chat-Typ
  und Gegenstellenidentität. Provider-, Direktnachrichten-, Gruppen- oder Thread-Form wird nicht mehr
  aus `sessionKey` geparst.
- Die Ableitung der Cron-Tool-Zustellung verwendet jetzt ausschließlich eine explizite Zustellung oder den aktuellen typisierten
  Zustellungskontext. Channel-, Gegenstellen-, Konto- oder Thread-
  Ziele werden nicht mehr aus `agentSessionKey` dekodiert.
- Runtime-Sitzungszeilen enthalten nicht mehr den alten Routenalias `lastProvider`.
  Hilfsfunktionen und Tests verwenden typisierte Felder `lastChannel` und `deliveryContext`;
  die Doctor-Migration ist die einzige Stelle, an der ältere Routenaliase
  oder persistente `origin`-Schatten übersetzt werden sollten.
- Transkriptereignisse, VFS-Zeilen und Zeilen für Tool-Artefakte werden jetzt in die agentenspezifische
  Datenbank geschrieben. Die nicht ausgelieferte globale Zuordnungstabelle für Transkriptdateien wurde entfernt; Doctor
  zeichnet veraltete Quellpfade stattdessen in dauerhaften Migrationszeilen auf.
- Die Runtime-Transkriptsuche durchsucht keine JSONL-Byte-Offsets mehr und prüft keine veralteten
  Transkriptdateien. Gateway-Pfade für Chat/Medien/Verlauf lesen Transkriptzeilen aus
  SQLite; Sitzungs-JSONL dient jetzt nur noch als veraltete Doctor-Eingabe, nicht als Runtime-Zustand
  oder Exportformat.
- Übergeordnete und Verzweigungsbeziehungen von Transkripten verwenden strukturierte
  `parentTranscriptScope: {agentId, sessionId}`-Metadaten in SQLite-Transkript-
  headern, nicht pfadähnliche `agent-db:...transcript_events...`-Locator-Zeichenfolgen.
- Der Vertrag des Transkriptmanagers stellt keine impliziten persistenten
  `create(cwd)`- oder `continueRecent(cwd)`-Konstruktoren mehr bereit. Persistente Transkript-
  manager werden mit einem expliziten `{agentId, sessionId}`-Gültigkeitsbereich geöffnet; nur
  In-Memory-Manager bleiben für Tests und reine Transkripttransformationen ohne Scope.
- APIs des Runtime-Transkriptspeichers lösen SQLite-Scopes auf, keine Dateisystempfade. Der
  alte `resolve...ForPath`-Helper und die nicht verwendeten `transcriptPath`-Schreiboptionen sind
  aus Runtime-Aufrufern entfernt.
- Die Runtime-Sitzungsauflösung verwendet jetzt `{agentId, sessionId}` und darf keine
  `sqlite-transcript://<agent>/<session>`-Zeichenfolgen für externe Grenzen ableiten.
  Veraltete absolute JSONL-Pfade dienen ausschließlich als Eingaben für die Doctor-Migration.
- Direct-Bridge-Datensätze der nativen Hook-Weiterleitung befinden sich jetzt in typisierten gemeinsamen
  `native_hook_relay_bridges`-Zeilen, die nach Relay-ID indiziert sind. Die Runtime schreibt für diese kurzlebigen Bridge-
  Datensätze weder eine `/tmp`-JSON-Registry noch undurchsichtige generische Datensätze mehr.
- `runEmbeddedPiAgent(...)` besitzt keinen Transkript-Locator-Parameter mehr.
  Vorbereitete Worker-Deskriptoren lassen Transkript-Locators ebenfalls weg. Der Runtime-Sitzungs-
  status und in die Warteschlange gestellte Folgeläufe führen `{agentId, sessionId}` statt
  abgeleiteter Transkript-Handles mit.
- Eingebettete Compaction übernimmt den SQLite-Scope jetzt von `agentId` und `sessionId`.
  Compaction-Hooks, Aufrufe der Kontext-Engine, CLI-Delegierung und Protokollantworten
  dürfen keine abgeleiteten `sqlite-transcript://...`-Handles erhalten. Export-/Debug-Code
  kann explizite Benutzerartefakte aus Zeilen materialisieren, stellt jedoch keinen
  generischen JSONL-Exportpfad für Sitzungen bereit und speist Dateinamen nicht zurück in die Runtime-
  Identität ein.
- `/export-session` liest Transkriptzeilen aus SQLite und schreibt ausschließlich die angeforderte
  eigenständige HTML-Ansicht. Der eingebettete Viewer rekonstruiert oder
  lädt Sitzungs-JSONL aus diesen Zeilen nicht mehr herunter.
- Die Delegierung an die Kontext-Engine analysiert keinen Transkript-Locator mehr, um
  die Agentenidentität wiederherzustellen. Der vorbereitete Runtime-Kontext übergibt den aufgelösten `agentId`
  an den integrierten Compaction-Adapter.
- Das Umschreiben von Transkripten und die Live-Kürzung von Tool-Ergebnissen lesen und speichern
  den Transkriptstatus jetzt anhand von `{agentId, sessionId}` und leiten keine temporären
  Locators für Ereignis-Payloads von Transkriptaktualisierungen ab.
- Die Oberfläche der Transkriptstatus-Helper besitzt keine Locator-basierten
  Varianten `readTranscriptState`, `replaceTranscriptStateEvents` oder
  `persistTranscriptStateMutation` mehr. Runtime-Aufrufer müssen die
  `{agentId, sessionId}`-APIs verwenden. Der Doctor-Import liest veraltete Dateien über explizite Datei-
  pfade und schreibt SQLite-Zeilen; Locator-Zeichenfolgen werden nicht migriert.
- Der Runtime-Vertrag des Sitzungs-Managers stellt `open(locator)`,
  `forkFrom(locator)` oder `setTranscriptLocator(...)` nicht mehr bereit. Persistierte Sitzungs-
  Manager öffnen ausschließlich anhand von `{agentId, sessionId}`; Listen-/Fork-Helper befinden sich
  auf zeilenorientierten Sitzungs- und Checkpoint-APIs statt auf der Fassade des Transkript-Managers.
- Die Transkriptleser-APIs des Gateway sind Scope-first. Sie akzeptieren
  `{agentId, sessionId}` und keinen positionellen Transkript-Locator, der
  versehentlich zur Runtime-Identität werden könnte. Das Parsen aktiver Transkript-Locators
  entfällt; veraltete Quellpfade werden ausschließlich vom Doctor-Importcode gelesen.
- Transkriptaktualisierungsereignisse sind ebenfalls Scope-first. `emitSessionTranscriptUpdate`
  akzeptiert keine bloße Locator-Zeichenfolge mehr, und Listener routen anhand von
  `{agentId, sessionId}`, ohne ein Handle zu parsen.
- Die Übertragung von Gateway-Sitzungsnachrichten löst Sitzungsschlüssel aus dem Agenten-/Sitzungs-
  Scope auf, nicht aus einem Transkript-Locator. Der alte Resolver/Cache von Transkript-Locators zu Sitzungs-
  schlüsseln wurde entfernt.
- SSE für den Gateway-Sitzungsverlauf filtert Live-Aktualisierungen nach Agenten-/Sitzungs-Scope. Es
  kanonisiert keine Kandidaten für Transkript-Locators, Realpaths oder dateiförmigen
  Transkriptidentitäten mehr, um zu entscheiden, ob ein Stream eine Aktualisierung erhalten soll.
- Hooks des Sitzungslebenszyklus leiten keine Transkript-Locators für
  `session_end` mehr ab und stellen sie auch nicht bereit. Hook-Konsumenten erhalten `sessionId`, `sessionKey`, IDs der nächsten Sitzung
  und Agentenkontext; Transkriptdateien sind nicht Teil des Lebenszyklus-
  vertrags.
- Reset-Hooks leiten ebenfalls keine Transkript-Locators mehr ab und stellen sie nicht bereit. Der
  `before_reset`-Payload enthält wiederhergestellte SQLite-Nachrichten sowie den Reset-
  Grund, während die Sitzungsidentität im Hook-Kontext verbleibt.
- Der Reset des Agent-Harness akzeptiert keinen Transkript-Locator mehr. Der Reset-Versand ist
  durch `sessionId`/`sessionKey` sowie den Grund begrenzt.
- Sitzungstypen von Agentenerweiterungen stellen `transcriptLocator` nicht mehr bereit; Erweiterungen
  sollten Sitzungskontext und Runtime-APIs verwenden, statt auf eine
  dateiförmige Transkriptidentität zuzugreifen.
- Compaction-Hooks von Plugins stellen keine Transkript-Locators mehr bereit. Der Hook-Kontext
  enthält bereits die Sitzungsidentität, und Transkriptlesevorgänge müssen über SQLite-
  Scope-bezogene APIs statt über dateiförmige Handles erfolgen.
- `before_agent_finalize`-Hooks stellen `transcriptPath` nicht mehr bereit, einschließlich
  Payloads der nativen Hook-Weiterleitung. Finalisierungs-Hooks verwenden ausschließlich den Sitzungskontext.
- Gateway-Reset-Antworten synthetisieren keinen Transkript-Locator mehr im
  zurückgegebenen Eintrag. Der Reset erstellt SQLite-Transkriptzeilen, gibt den bereinigten
  Sitzungseintrag zurück und überlässt den Transkriptzugriff Scope-bezogenen Lesern.
- Ergebnisse eingebetteter Läufe und von Compaction stellen keine Transkript-Locators mehr für die
  Sitzungsabrechnung bereit. Automatische Compaction aktualisiert ausschließlich den aktiven `sessionId`,
  Compaction-Zähler und Token-Metadaten.
- Ergebnisse eingebetteter Versuche geben `transcriptLocatorUsed` nicht mehr zurück, und
  `compact()`-Ergebnisse der Kontext-Engine geben keine Transkript-Locators mehr zurück.
  Runtime-Wiederholungsschleifen akzeptieren ausschließlich einen nachfolgenden `sessionId`.
- Ergebnisse beim Anhängen an das Transkript des Delivery-Mirror geben keine Transkript-
  Locators mehr zurück. Aufrufer erhalten den angehängten `messageId`; Signale für Transkriptaktualisierungen verwenden
  den SQLite-Scope.
- Fork-Helper für übergeordnete Sitzungen geben ausschließlich den geforkten `sessionId` zurück. Die Vorbereitung von Subagents
  übergibt den Scope des untergeordneten Agenten/der untergeordneten Sitzung an Engines.
- Parameter des CLI-Runners und das erneute Einspeisen des Verlaufs akzeptieren keine Transkript-Locators mehr.
  CLI-Verlaufslesevorgänge lösen den SQLite-Transkript-Scope aus `{agentId,
sessionId}` und dem Sitzungsschlüsselkontext auf.
- Test-Fixtures für CLI und eingebettete Runner befüllen und lesen SQLite-Transkriptzeilen jetzt
  anhand der Sitzungs-ID, statt vorzugeben, aktive Sitzungen seien `*.jsonl`-Dateien, oder
  eine `sqlite-transcript://...`-Zeichenfolge über Runtime-Parameter weiterzureichen.
- Guard-Ereignisse für Tool-Ergebnisse einer Sitzung werden aus dem bekannten Sitzungs-Scope ausgegeben, selbst wenn ein
  In-Memory-Manager keinen abgeleiteten Locator besitzt. Die Tests simulieren keine aktiven
  `/tmp/*.jsonl`-Transkriptdateien mehr.
- BTW- und Compaction-Checkpoint-Helper lesen und forken Transkriptzeilen jetzt anhand des
  SQLite-Scope. Checkpoint-Metadaten speichern jetzt ausschließlich Sitzungs-IDs sowie Blatt-/Eintrags-IDs;
  abgeleitete Locators werden nicht mehr in Checkpoint-Payloads geschrieben.
- Die Transkriptschlüssel-Suche des Gateway verwendet an Protokollgrenzen den SQLite-Transkript-
  Scope und löst bei Transkriptdateinamen weder Realpaths noch Stat-Abfragen aus.
- Die automatische Transkriptrotation bei Compaction schreibt nachfolgende Transkriptzeilen
  direkt über den SQLite-Transkriptspeicher. Sitzungszeilen enthalten ausschließlich die
  nachfolgende Sitzungsidentität, keinen dauerhaften JSONL-Pfad oder persistierten Locator.
- Die Compaction der eingebetteten Kontext-Engine verwendet SQLite-benannte Helper zur Transkriptrotation.
  Die Rotationstests konstruieren keine nachfolgenden JSONL-Pfade mehr und
  modellieren aktive Sitzungen nicht mehr als Dateien.
- Die verwaltete Aufbewahrung ausgehender Bilder indiziert ihren Cache für Transkriptnachrichten anhand von
  SQLite-Transkriptstatistiken statt über Dateisystem-Stat-Aufrufe.
- Runtime-Sitzungssperren und der eigenständige veraltete `.jsonl.lock`-Doctor-
  Pfad wurden entfernt.
- Das Runtime-Barrel von Microsoft Teams und das öffentliche Plugin-SDK re-exportieren
  den alten Dateisperren-Helper nicht mehr; dauerhafte Plugin-Statuspfade sind SQLite-gestützt.
- Das Bereinigen nach Sitzungsalter/-anzahl und die explizite Sitzungsbereinigung wurden entfernt.
  Doctor ist für den Legacy-Import zuständig; veraltete Sitzungen werden explizit zurückgesetzt oder gelöscht.
- Doctor-Integritätsprüfungen zählen eine veraltete JSONL-Datei nicht mehr als gültiges aktives
  Transkript für eine SQLite-Sitzungszeile. Die Integrität aktiver Transkripte basiert ausschließlich auf SQLite;
  veraltete JSONL-Dateien werden als Eingaben für Migration bzw. Bereinigung verwaister Daten gemeldet.
- Doctor behandelt `agents/<agent>/sessions/` nicht mehr als erforderlichen Runtime-
  Status. Das Verzeichnis wird nur durchsucht, wenn es bereits existiert, und dient dann als Eingabe für den Legacy-Import
  oder die Bereinigung verwaister Daten.
- Gateway-`sessions.resolve`, Pfade für Sitzungspatches/-resets/-Compaction, das Starten von Subagents,
  schnelles Abbrechen, ACP-Metadaten, Heartbeat-isolierte Sitzungen und TUI-
  Patching migrieren oder bereinigen veraltete Sitzungsschlüssel nicht mehr als Nebeneffekt
  normaler Runtime-Arbeit.
- Die Sitzungsauslösung für CLI-Befehle gibt jetzt den zuständigen `agentId` statt eines
  `storePath` zurück und kopiert bei der normalen
  Auflösung von `--to` oder `--session-id` keine veralteten Hauptsitzungszeilen mehr. Die Kanonisierung
  veralteter Hauptzeilen gehört ausschließlich in Doctor.
- Die Runtime-Auflösung der Subagent-Tiefe liest `sessions.json` oder JSON5-
  Sitzungsspeicher nicht mehr. Sie liest SQLite-`session_entries` anhand der Agenten-ID, und veraltete
  Tiefen-/Sitzungsmetadaten können ausschließlich über den Doctor-Importpfad eingebracht werden.
- Sitzungsüberschreibungen von Authentifizierungsprofilen werden durch direkte Upserts von
  `{agentId, sessionKey}`-Zeilen persistiert, statt eine dateiförmige Sitzungsspeicher-Runtime verzögert zu laden.
- Verbose-Gating für automatische Antworten und Helper für Sitzungsaktualisierungen lesen/aktualisieren jetzt SQLite-
  Sitzungszeilen anhand der Sitzungsidentität und benötigen keinen veralteten Speicherpfad mehr,
  bevor sie den persistierten Zeilenstatus ändern.
- Helper für Sitzungsmetadaten von Befehlsläufen verwenden jetzt eintragsorientierte Namen und Modul-
  pfade; die alte `session-store`-Oberfläche für Befehls-Helper wurde entfernt.
- Das Einspeisen von Bootstrap-Headern und die Härtung manueller Compaction-Grenzen verändern
  SQLite-Transkriptzeilen jetzt direkt. Runtime-Aufrufer übergeben die Sitzungsidentität, keine
  schreibbaren `.jsonl`-Pfade.
- Die stille Wiedergabe bei Sitzungsrotation kopiert aktuelle Benutzer-/Assistenten-Dialogschritte anhand von
  `{agentId, sessionId}` aus SQLite-Transkriptzeilen. Sie akzeptiert keine
  Quell- oder Ziel-Transkript-Locators mehr.
- Neue Runtime-Sitzungszeilen speichern keine Transkript-Locators mehr. Aufrufer verwenden
  `{agentId, sessionId}` direkt; Export-/Debug-Befehle können Ausgabedateinamen
  auswählen, wenn sie Zeilen materialisieren.
- Beim Starten einer neuen persistierten Transkriptsitzung werden SQLite-Zeilen jetzt immer anhand des
  Scope geöffnet. Der Sitzungs-Manager verwendet keinen vorherigen Transkript-
  pfad oder Locator aus der Dateiära mehr als Identität der neuen Sitzung.
- Persistierte Transkriptsitzungen verwenden die explizite
  `openTranscriptSessionManagerForSession({agentId, sessionId})`-API. Die alten
  statischen `SessionManager.create/openForSession/list/forkFromSession`-Fassaden sind
  entfernt, sodass Tests und Runtime-Code nicht versehentlich die Sitzungserkennung
  aus der Dateiära wiederherstellen können.
- Die Plugin-Runtime stellt `api.runtime.agent.session.resolveTranscriptLocatorPath` nicht mehr bereit;
  Plugin-Code verwendet SQLite-Zeilen-Helper und Scope-Werte.
- Die öffentliche `session-store-runtime`-SDK-Oberfläche exportiert jetzt ausschließlich Sitzungszeilen-
  und Transkriptzeilen-Helper. Spezialisierte SQLite-Schema-/Pfad-/Transaktions-Helper
  befinden sich in `sqlite-runtime`; rohe Helper zum Öffnen/Schließen/Zurücksetzen bleiben ausschließlich lokal
  für Erstanbieter-Tests verfügbar.
- Veraltete Klassifizierer für `.jsonl`-Trajektorien-/Checkpoint-Dateinamen befinden sich jetzt im
  Doctor-Modul für veraltete Sitzungsdateien. Die Kernsitzungsvalidierung importiert keine
  Helper für Datei-Artefakte mehr, um normale SQLite-Sitzungs-IDs zu bestimmen.
- Blockierende Subagent-Läufe von Active Memory verwenden SQLite-Transkriptzeilen, statt
  temporäre oder persistierte `session.jsonl`-Dateien im Plugin-Status zu erstellen. Die
  alte `transcriptDir`-Option wurde entfernt.
- Einmalige Slug-Erzeugung und Planner-Läufe des Systemagenten verwenden SQLite-Transkriptzeilen,
  statt temporäre `session.jsonl`-Dateien zu erstellen.
- `llm-task`-Hilfsläufe und die Extraktion verborgener Festlegungen verwenden ebenfalls SQLite-
  Transkriptzeilen, sodass diese ausschließlich modellinternen Hilfssitzungen keine
  temporären JSON-/JSONL-Transkriptdateien mehr erstellen.
- `TranscriptSessionManager` ist jetzt nur noch ein geöffneter SQLite-Transkriptbereich.
  Laufzeitcode öffnet ihn mit `openTranscriptSessionManagerForSession({agentId,
sessionId})`; Abläufe zum Erstellen, Verzweigen, Fortsetzen, Auflisten und Forken befinden sich in den
  jeweils zuständigen SQLite-Zeilenhelfern statt in statischen Manager-Fassaden.
  Doctor-/Import-/Debug-Code verarbeitet explizite Legacy-Quelldateien außerhalb des
  Laufzeit-Sitzungsmanagers.
- Die veralteten Fassadenmethoden `SessionManager.newSession()` und
  `SessionManager.createBranchedSession()` wurden entfernt. Neue
  Sitzungen und Transkriptnachfolger werden durch den jeweils zuständigen SQLite-
  Workflow erstellt, statt einen bereits geöffneten Manager in eine andere
  persistierte Sitzung umzuwandeln.
- Entscheidungen zum Forken übergeordneter Transkripte und die Fork-Erstellung akzeptieren
  `storePath` oder `sessionsDir` nicht mehr; sie verwenden den SQLite-
  Transkriptbereich `{agentId, sessionId}` statt beibehaltener Dateisystem-Pfadmetadaten.
- Memory-Host exportiert keine wirkungslosen Hilfsfunktionen mehr zur
  Transkriptklassifizierung anhand des Sitzungsverzeichnisses; die Transkriptfilterung wird jetzt während der Eintragserstellung aus
  SQLite-Zeilenmetadaten abgeleitet.
- Memory-Host- und QMD-Sitzungsexporttests verwenden SQLite-Transkriptbereiche. Alte
  `agents/<agentId>/sessions/*.jsonl`-Pfade werden nur noch dort abgedeckt, wo ein Test
  bewusst die Kompatibilität von Doctor, Import oder Export nachweist.
- Die Rohdateninspektion von Sitzungen in QA Lab verwendet jetzt `sessions.list` über das Gateway,
  statt `agents/qa/sessions/sessions.json` zu lesen; MSteams-Feedback
  wird direkt an SQLite-Transkripte angehängt, ohne einen JSONL-Pfad vorzutäuschen.
- Gemeinsam verarbeitete eingehende Kanalinteraktionen führen jetzt `{agentId, sessionKey}` statt eines
  veralteten `storePath`. Die Aufzeichnungspfade von LINE, WhatsApp, Slack, Discord, Telegram, Matrix, Signal,
  iMessage, BlueBubbles, Feishu, Google Chat, IRC, Nextcloud Talk, Zalo,
  Zalo Personal, QA Channel, Microsoft Teams, Mattermost, Synology Chat, Tlon,
  Twitch und QQBot lesen jetzt Aktualisierungszeit-Metadaten und erfassen
  eingehende Sitzungszeilen über die SQLite-Identität.
- Die Persistierung von Transkript-Locators wurde aus aktiven Sitzungszeilen entfernt.
  `resolveSessionTranscriptTarget` gibt `agentId`, `sessionId` und optionale
  Themenmetadaten zurück; Doctor ist der einzige Code, der Namen veralteter Transkriptdateien
  importiert.
- Laufzeit-Transkript-Header beginnen mit SQLite-Version `1`. Upgrades alter JSONL-V1-/V2-/V3-
  Formate erfolgen ausschließlich beim Doctor-Import und normalisieren importierte Header auf
  die aktuelle SQLite-Transkriptversion, bevor Zeilen gespeichert werden.
- Die Database-first-Schutzprüfung verbietet jetzt `SessionManager.listAll` und
  `SessionManager.forkFromSession`; Sitzungsauflistung sowie Fork-/Wiederherstellungsworkflows
  müssen zeilen- beziehungsweise bereichsbasierte SQLite-APIs verwenden.
- Die Schutzprüfung verbietet außerhalb des Doctor-/Import-Codes außerdem Namen veralteter Hilfsfunktionen zum
  Parsen von Transkript-JSONL und Reparieren aktiver Zweige, sodass die Laufzeit keinen zweiten Legacy-
  Transkriptmigrationspfad entwickeln kann.
- Eingebettete PI-Läufe lehnen eingehende Transkript-Handles ab. Sie verwenden die SQLite-
  Identität `{agentId, sessionId}` vor dem Start des Workers und erneut, bevor der
  Versuch auf den Transkriptzustand zugreift. Eine veraltete `/tmp/*.jsonl`-Eingabe kann kein
  Laufzeit-Schreibziel auswählen.
- Cache-Trace-, Anthropic-Payload-, Rohdatenstream- und Diagnose-Zeitleisteneinträge
  werden jetzt in typisierte SQLite-Zeilen vom Typ `diagnostic_events` geschrieben. Gateway-Stabilitätspakete
  werden jetzt in typisierte SQLite-Zeilen vom Typ `diagnostic_stability_bundles` geschrieben. Die alten
  JSONL-Überschreibungspfade `diagnostics.cacheTrace.filePath`, `OPENCLAW_CACHE_TRACE_FILE`,
  `OPENCLAW_ANTHROPIC_PAYLOAD_LOG_FILE` und
  `OPENCLAW_DIAGNOSTICS_TIMELINE_PATH` wurden entfernt, und
  die normale Stabilitätserfassung schreibt keine `logs/stability/*.json`-Dateien mehr.
- Die Cron-Persistierung gleicht jetzt SQLite-Zeilen vom Typ `cron_jobs` ab, statt
  bei jedem Speichern die gesamte Job-Tabelle zu löschen und neu einzufügen. Rückschreibungen von Plugin-Zielen
  aktualisieren passende Cron-Zeilen direkt und halten den Cron-Laufzeitzustand in
  derselben Zustandsdatenbanktransaktion.
- Cron-Laufzeitaufrufer verwenden jetzt einen stabilen SQLite-Cron-Speicherschlüssel. Veraltete
  `cron.store`-Pfade dienen nur noch als Doctor-Importeingaben; Produktions-Gateway, Aufgaben-
  wartung, Status, Ausführungsverlauf und Telegram-Zielrückschreibungen verwenden
  `resolveCronStoreKey` und normalisieren den Schlüssel nicht mehr als Pfad. Der Cron-Status
  meldet jetzt `storeKey` statt des alten dateiförmigen Felds `storePath`.
- Das Laden und Planen der Cron-Laufzeit normalisiert keine veralteten persistierten Job-
  Formate mehr, etwa `jobId`, `schedule.cron`, numerisches `atMs`, boolesche Zeichenfolgen oder
  fehlendes `sessionTarget`. Der Doctor-Legacy-Import übernimmt diese Reparaturen, bevor Zeilen
  in SQLite eingefügt werden.
- ACP-Spawn löst keine Pfade zu Transkript-JSONL-Dateien mehr auf und persistiert sie nicht mehr. Die Spawn-
  und Thread-Bind-Einrichtung persistiert die SQLite-Sitzungszeile direkt und behält die
  Sitzungs-ID als Transkriptidentität bei.
- ACP-Sitzungsmetadaten-APIs lesen, listen und aktualisieren beziehungsweise erstellen jetzt SQLite-Zeilen anhand von `agentId`
  und stellen `storePath` nicht mehr als Teil des ACP-Sitzungseintragsvertrags bereit.
- Die Abrechnung der Sitzungsnutzung und die Gateway-Nutzungsaggregation lösen Transkripte jetzt
  ausschließlich anhand von `{agentId, sessionId}` auf. Der Kosten-/Nutzungs-Cache und Zusammenfassungen erkannter Sitzungen
  erzeugen keine Transkript-Locator-Zeichenfolgen mehr und geben sie nicht mehr zurück.
- Gateway-Chat-Anhänge, die Persistierung abgebrochener Teilantworten, `/sessions.send` und
  Webchat-Medien-Transkriptschreibvorgänge hängen Inhalte direkt über den SQLite-Transkriptbereich
  an. Die Gateway-Hilfsfunktion zur Transkriptinjektion akzeptiert keinen
  `transcriptLocator`-Parameter mehr.
- Die SQLite-Transkripterkennung listet jetzt nur Transkriptbereiche und Statistiken auf:
  `{agentId, sessionId, updatedAt, eventCount}`. Die nicht mehr verwendete
  Kompatibilitätshilfsfunktion `listSqliteSessionTranscriptLocators` und das zeilenbezogene
  Feld `locator` wurden entfernt.
- Die Laufzeit für Transkriptreparaturen stellt jetzt nur noch
  `repairTranscriptSessionStateIfNeeded({agentId, sessionId})` bereit. Die alte
  Locator-basierte Reparaturhilfsfunktion wurde gelöscht; Doctor-/Debug-Code liest explizite
  Quelldateipfade und migriert niemals Locator-Zeichenfolgen.
- Die ACP-Replay-Ledger-Laufzeit speichert sitzungsbezogene Replay-Zeilen jetzt in der gemeinsamen
  SQLite-Zustandsdatenbank statt in `acp/event-ledger.json`; Doctor importiert und
  entfernt die Legacy-Datei.
- Gateway-Hilfsfunktionen zum Lesen von Transkripten befinden sich jetzt in
  `src/gateway/session-transcript-readers.ts` statt im alten Modul
  `session-utils.fs`. Die Prüfung des Fallback-Wiederholungsverlaufs ist nach
  SQLite-Transkriptinhalten statt nach der alten Datei-Hilfsoberfläche benannt.
- Gateway-Hilfsfunktionen für injizierte Chats und Compaction übergeben jetzt den SQLite-Transkriptbereich
  über interne Hilfs-APIs, statt Werte als Transkriptpfade oder
  Quelldateien zu bezeichnen.
- Die Erkennung von Bootstrap-Fortsetzungen prüft jetzt SQLite-Transkriptzeilen über
  `hasCompletedBootstrapTranscriptTurn`; sie stellt keinen dateiförmigen
  Hilfsfunktionsnamen mehr bereit.
- Tests des eingebetteten Runners verwenden jetzt die SQLite-Transkriptidentität, und das Öffnen eines neuen
  Transkriptmanagers erfordert immer ein explizites `sessionId`.
- Hilfsfunktionen zur Speicherindizierung verwenden jetzt durchgängig SQLite-Transkriptterminologie:
  Der Host exportiert `listSessionTranscriptScopesForAgent` und
  `sessionTranscriptKeyForScope`, die gezielte Synchronisierung stellt `sessionTranscripts` in die Warteschlange,
  öffentliche Sitzungssuchtreffer stellen opake `transcript:<agent>:<session>`-Pfade bereit,
  und der interne DB-Quellschlüssel lautet `session:<session>` unter
  `source_kind='sessions'` statt eines vorgetäuschten Dateipfads.
- Die generische Hilfsfunktion des Plugin-SDK für persistente Deduplizierung stellt keine dateiförmigen
  Optionen mehr bereit. Aufrufer geben SQLite-Bereichsschlüssel an, und dauerhafte Deduplizierungszeilen befinden sich im
  gemeinsamen Plugin-Zustand.
- Microsoft Teams-SSO-Token wurden von gesperrten JSON-Dateien in den SQLite-Plugin-
  Zustand verschoben. Doctor importiert `msteams-sso-tokens.json`, erstellt kanonische SSO-Token-
  Schlüssel aus den Payloads neu und entfernt die Quelldatei. Delegierte OAuth-Token verbleiben
  an ihrer bestehenden Grenze für private Anmeldedatendateien.
- Der Matrix-Synchronisierungs-Cachezustand wurde von `bot-storage.json` in den SQLite-Plugin-
  Zustand verschoben. Doctor importiert veraltete rohe oder umschlossene Synchronisierungs-Payloads und entfernt die
  Quelldatei. Aktive Matrix- und QA-Lab-Matrix-Adapterclients übergeben ein SQLite-Synchronisierungsspeicher-
  Stammverzeichnis statt eines vorgetäuschten `sync-store.json`- oder `bot-storage.json`-Pfads.
- Der Status der Matrix-Legacy-Kryptomigration wurde von
  `legacy-crypto-migration.json` in den SQLite-Plugin-Zustand verschoben. Doctor importiert die
  alte Statusdatei; Matrix-SDK-IndexedDB-Snapshots wurden von
  `crypto-idb-snapshot.json` in SQLite-Plugin-Blobs verschoben. Matrix-Wiederherstellungsschlüssel und
  Anmeldedaten sind Zeilen im SQLite-Plugin-Zustand; ihre alten JSON-Dateien dienen nur noch als Doctor-
  Migrationseingaben.
- Memory-Wiki-Aktivitätsprotokolle verwenden jetzt SQLite-Plugin-Zustand statt
  `.openclaw-wiki/log.jsonl`. Der Memory-Wiki-Migrations-Provider importiert alte
  JSONL-Protokolle; Wiki-Markdown und Inhalte des Benutzer-Vaults bleiben als
  Workspace-Inhalte dateibasiert.
- Memory Wiki erstellt `.openclaw-wiki/state.json` oder das ungenutzte Verzeichnis
  `.openclaw-wiki/locks` nicht mehr. Der Migrations-Provider entfernt diese ausgemusterten
  Plugin-Metadatendateien, wenn sie in einem älteren Vault noch vorhanden sind.
- Audit-Einträge des System-Agenten verwenden jetzt den zentralen SQLite-Plugin-Zustand statt
  `audit/crestodian.jsonl`. Doctor importiert das veraltete JSONL-Auditprotokoll und
  entfernt es nach erfolgreichem Import.
- Audit-Einträge für das Schreiben und Beobachten der Konfiguration verwenden jetzt den zentralen SQLite-Plugin-Zustand statt
  `logs/config-audit.jsonl`. Doctor importiert das veraltete JSONL-Auditprotokoll und
  entfernt es nach erfolgreichem Import.
- Die macOS-Begleitanwendung schreibt beim Bearbeiten von `openclaw.json` keine anwendungslokalen
  Sidecar-Dateien `logs/config-audit.jsonl` oder `logs/config-health.json` mehr. Die Konfigurationsdatei
  bleibt dateibasiert, Wiederherstellungs-Snapshots verbleiben neben der Konfigurationsdatei,
  und dauerhafter Konfigurationsaudit- und Integritätszustand gehört in den SQLite-Speicher des Gateways.
- Ausstehende Genehmigungen für die Rettungsfunktion des System-Agenten verwenden jetzt den zentralen SQLite-Plugin-Zustand statt
  `crestodian/rescue-pending/*.json` oder `openclaw/rescue-pending/*.json`.
  Diese kurzlebigen Sicherheitsberechtigungen werden niemals importiert; Doctor verwirft
  beide ausgemusterten Verzeichnisse, damit ein Upgrade keinen veralteten Schreibvorgang reaktivieren kann.
- Der temporäre Aktivierungszustand von Phone Control verwendet jetzt SQLite-Plugin-Zustand statt
  `plugins/phone-control/armed.json`. Doctor importiert die veraltete Aktivierungszustandsdatei
  in den Namespace `phone-control/arm-state` und entfernt die Datei.
- Doctor repariert JSONL-Transkripte nicht mehr direkt und erstellt keine JSONL-
  Sicherungsdateien mehr. Er importiert den aktiven Zweig in SQLite und entfernt die Legacy-Quelle.
- Die Transkriptsuche des Sitzungsspeicher-Hooks verwendet bereichsbeschränkte SQLite-Lesevorgänge über
  `{agentId, sessionId}`. Seine Hilfsfunktion akzeptiert oder ermittelt keine Transkript-Locators,
  Legacy-Dateilesevorgänge oder Optionen zum Neuschreiben von Dateien mehr.
- Konversationsbindungen des Codex-App-Servers verschlüsseln den SQLite-Plugin-Zustand jetzt anhand des
  OpenClaw-Sitzungsschlüssels oder eines expliziten `{agentId, sessionId}`-Bereichs. Sie dürfen keine
  Fallback-Bindungen über Transkriptpfade beibehalten.
- Lesevorgänge des gespiegelten Verlaufs im Codex-App-Server verwenden ausschließlich den SQLite-Transkriptbereich;
  sie dürfen die Identität nicht aus Transkriptdateipfaden wiederherstellen.
- Pfade zum Zurücksetzen der Rollenreihenfolge und der Compaction löschen keine alten Transkriptdateien
  mehr; beim Zurücksetzen werden nur die SQLite-Sitzungszeile und die Transkriptidentität rotiert.
- Gateway-Antworten zum Zurücksetzen und für Checkpoints geben bereinigte Sitzungszeilen sowie Sitzungs-
  IDs zurück. Sie erzeugen für Clients keine SQLite-Transkript-Locators mehr.
- Dreaming in Memory Core bereinigt Sitzungszeilen nicht mehr durch Prüfung auf fehlende
  JSONL-Dateien. Die Subagent-Bereinigung erfolgt über die Sitzungslaufzeit-API statt über
  Existenzprüfungen im Dateisystem. Die Tests zur Transkriptaufnahme legen SQLite-Zeilen
  direkt an, statt `agents/<id>/sessions`-Fixtures oder Locator-
  Platzhalter zu erstellen.
- Die Memory-Transkriptindizierung kann `transcript:<agentId>:<sessionId>` als
  virtuellen Suchtrefferpfad für Zitations-/Lesehilfen bereitstellen. Die dauerhafte Indexquelle ist
  relational (`source_kind='sessions'`, `source_key='session:<sessionId>'`,
  `session_id=<sessionId>`), daher ist der Wert kein Locator für ein Laufzeittranskript,
  kein Dateisystempfad und darf niemals an Sitzungs-Laufzeit-APIs zurückgegeben werden.
- Der Speicherstatus von Gateway doctor liest die Anzahl der Kurzzeitabrufe und Phasensignale
  aus SQLite-Plugin-Zustandszeilen statt aus `memory/.dreams/*.json`; die Ausgaben der CLI und
  von doctor bezeichnen diesen Speicher nun als SQLite-Speicher und nicht als Pfad.
- Die Memory-Core-Laufzeit, der CLI-Status, die Gateway-doctor-Methoden und die Fassaden des Plugin SDK
  prüfen oder archivieren veraltete `.dreams/session-corpus`-Dateien nicht mehr.
  Diese Dateien dienen nur als Migrationseingaben; doctor importiert sie in SQLite und
  löscht die Quelle nach der Überprüfung. Beweiszeilen für die aktive Sitzungserfassung
  verwenden nun den virtuellen SQLite-Pfad `memory/session-ingestion/<day>.txt`; die Laufzeit
  schreibt niemals Zustand in `.dreams/session-corpus` und leitet daraus auch keinen Zustand ab.
- Öffentliche Memory-Core-Artefakte stellen SQLite-Hostereignisse als virtuelles JSON-Artefakt
  `memory/events/memory-host-events.json` bereit; sie verwenden den
  veralteten Quellpfad `.dreams/events.jsonl` nicht mehr erneut.
- Sandbox-Container-/Browser-Registrierungen verwenden nun die gemeinsame
  SQLite-Tabelle `sandbox_registry_entries` mit typisierten Spalten für Sitzung, Image, Zeitstempel,
  Backend/Konfiguration und Browser-Port. Doctor importiert veraltete monolithische und
  aufgeteilte JSON-Registrierungsdateien und entfernt erfolgreich importierte Quellen. Laufzeitlesevorgänge
  verwenden die typisierten Zeilenspalten als maßgebliche Datenquelle; `entry_json` ist nur eine
  Wiedergabe-/Debug-Kopie.
- Zusagen verwenden nun eine typisierte gemeinsame Tabelle `commitments` anstelle eines
  JSON-Blobs für den gesamten Speicher. Die Laufzeit verwendet indizierte Abfragen für Geltungsbereich,
  Zustellfenster, rollierendes Limit, Status und Versuche sowie synchrone SQLite-Transaktionen;
  `record_json` ist nur eine Wiedergabe-/Debug-Kopie. Eine explizite doctor-Reparatur validiert
  die vollständige veraltete Datei `commitments.json`, behält neuere SQLite-Zeilen bei, überprüft das
  Ergebnis und entfernt erst dann die unveränderte Quelle. Die Laufzeit liest oder
  schreibt die außer Betrieb genommene Datei niemals.
- Web-Push-Abonnements und die generierte VAPID-Identität verwenden nun typisierte gemeinsame
  Zeilen `web_push_subscriptions` und `web_push_vapid_keys`. Laufzeitregistrierung,
  Ablaufbereinigung und Schlüsselerzeugung bei der ersten Verwendung nutzen SQLite-Transaktionen
  auf Zeilenebene. Eine explizite Doctor-Reparatur validiert beide außer Betrieb genommenen JSON-Speicher,
  beansprucht sie vor dem SQLite-Schreibvorgang, importiert sie atomar, weist
  widersprüchliche VAPID-Identitäten zurück, überprüft das Ergebnis und entfernt erst dann die
  Beanspruchungen. Doctor hält während des vollständigen Imports die Wartungssperre
  des Zustandsverzeichnisses, damit ein älterer Gateway die außer Betrieb genommenen Dateien nicht neu erstellen kann.
  Registrierung, Zustellung, Löschung und Schlüsselauflösung schlagen geschlossen fehl, bis Doctor
  ausstehende veraltete Quellen oder unterbrochene Beanspruchungen aufgelöst hat.
- Cron-Auftragsdefinitionen, Zeitplanzustand und Ausführungsverlauf verfügen nicht mehr über
  JSON-Schreib- oder -Lesevorgänge zur Laufzeit. Die Laufzeit verwendet `cron_jobs`-Zeilen mit typisierten Spalten
  für Zeitplan, Nutzlast, Zustellung, Fehlerwarnung, Sitzung, Status und Laufzeitzustand sowie
  Cron-eigene `task_runs`-Details für Diagnose, Zustellung, Sitzung/Ausführung, Modell
  und Token-Gesamtwerte. `job_json` ist nur eine Wiedergabe-/Debug-Kopie; `state_json` enthält verschachtelte
  Laufzeitdiagnosen, für die noch keine häufig abgefragten Felder vorhanden sind, während die Laufzeit
  häufig verwendete Zustandsfelder aus typisierten Spalten rehydriert. Doctor importiert
  veraltete Dateien `jobs.json`, `jobs-state.json` und `runs/*.jsonl` und entfernt
  die importierten Quellen. Rückschreibvorgänge für Plugin-Ziele aktualisieren passende `cron_jobs`-Zeilen,
  anstatt den gesamten Cron-Speicher zu laden und zu ersetzen.
- Der Gateway-Start ignoriert veraltete `notify: true`-Markierungen in der Laufzeitprojektion.
  Doctor liest die außer Betrieb genommenen Rohdaten `cron.webhook` nur, während
  diese Markierungen in eine explizite SQLite-Zustellung übersetzt werden, und entfernt anschließend den Konfigurationsschlüssel.
- Warteschlangen für ausgehende Nachrichten und Sitzungszustellungen speichern nun Warteschlangenstatus, Eintragstyp,
  Sitzungsschlüssel, Kanal, Ziel, Konto-ID, Wiederholungsanzahl, letzten Versuch/Fehler,
  Wiederherstellungszustand und Plattform-Sendemarkierungen als typisierte Spalten in der gemeinsamen
  Tabelle `delivery_queue_entries`. Die Laufzeitwiederherstellung liest diese häufig verwendeten Felder aus
  den typisierten Spalten, und Wiederholungs-/Wiederherstellungsmutationen aktualisieren diese Spalten direkt,
  ohne Wiedergabe-JSON neu zu schreiben. Die vollständige JSON-Nutzlast bleibt nur als
  Wiedergabe-/Debug-Blob für Nachrichtentexte und andere selten verwendete Wiedergabedaten erhalten.
- Verwaltete Datensätze ausgehender Bilder verwenden nun typisierte gemeinsame
  `managed_outgoing_image_records`-Zeilen. Die Laufzeit liest ausschließlich typisierte Spalten; die
  JSON-Spalte ist eine Wiedergabe-/Debug-Kopie. Die ursprünglichen Bildbytes bleiben benannte
  Anhangsartefakte im Verzeichnis für verwaltete Medien.
- Discord-Einstellungen für die Modellauswahl, Hashes für die Befehlsbereitstellung und Thread-Bindungen
  verwenden nun den gemeinsamen SQLite-Plugin-Zustand. Ihre Importpläne für veraltetes JSON befinden sich in der
  Einrichtungs-/doctor-Migrationsoberfläche des Discord-Plugins und nicht im Kernmigrationscode.
- Detektoren für den Import veralteter Plugin-Daten verwenden nach doctor benannte Module wie
  `doctor-legacy-state.ts` oder `doctor-state-imports.ts`; normale Kanallaufzeitmodule
  dürfen keine Detektoren für veraltetes JSON importieren.
- BlueBubbles-Aufholcursor und Markierungen zur Deduplizierung eingehender Nachrichten verwenden nun den gemeinsamen
  SQLite-Plugin-Zustand. Ihre Importpläne für veraltetes JSON befinden sich in der
  Einrichtungs-/doctor-Migrationsoberfläche des BlueBubbles-Plugins und nicht im Kernmigrationscode.
- Telegram-Aktualisierungs-Offsets, Sticker-Cache-Zeilen, Cache-Zeilen gesendeter Nachrichten,
  Cache-Zeilen für Themennamen und Thread-Bindungen verwenden nun den gemeinsamen SQLite-Plugin-
  Zustand. Ihre Importpläne für veraltetes JSON befinden sich in der
  Einrichtungs-/doctor-Migrationsoberfläche des Telegram-Plugins und nicht im Kernmigrationscode.
- iMessage-Aufholcursor, Zuordnungen kurzer Antwort-IDs und Deduplizierungszeilen für Sende-Echos
  verwenden nun den gemeinsamen SQLite-Plugin-Zustand. Die alten Dateien `imessage/catchup/*.json`,
  `imessage/reply-cache.jsonl` und `imessage/sent-echoes.jsonl` dienen
  nur als doctor-Eingaben.
- Feishu-Deduplizierungszeilen verwenden nun die beanspruchbare Kerndeduplizierung
  (`feishu.dedup.*`-Namensräume im gemeinsamen SQLite-Plugin-Zustand) anstelle von
  `feishu/dedup/*.json`-Dateien oder dem außer Betrieb genommenen, manuell implementierten `dedup.*`-Speicher,
  ohne Import veralteter Daten, da der Cache zum Schutz vor Wiederholungen nach dem Upgrade neu aufgebaut wird.
- Microsoft Teams-Unterhaltungen, Umfragen, ausstehende Upload-Puffer und Feedback-
  Erkenntnisse verwenden nun gemeinsame SQLite-Tabellen für Plugin-Zustand/Blobs. Der Pfad für ausstehende Uploads
  verwendet `plugin_blob_entries`, sodass Medienpuffer als SQLite-BLOBs
  statt als Base64-JSON gespeichert werden. Die Namen der Laufzeithelfer verwenden nun SQLite-/Zustandsbezeichnungen
  statt der `*-fs`-Dateispeicherbezeichnungen, und der alte `storePath`-Shim ist
  aus diesen Speichern entfernt. Der Importplan für veraltetes JSON befindet sich in der
  Einrichtungs-/doctor-Migrationsoberfläche des Microsoft Teams-Plugins.
- Von Zalo gehostete ausgehende Medien verwenden nun das gemeinsame SQLite `plugin_blob_entries`
  anstelle temporärer JSON-/bin-Begleitdateien `openclaw-zalo-outbound-media`.
- HTML und Metadaten des Diff-Viewers verwenden nun das gemeinsame SQLite `plugin_blob_entries`
  anstelle temporärer Dateien `meta.json`/`viewer.html`. Viewer-HTML wird als
  gzip-Blob gespeichert, und nur der Hash des URL-Tokens wird persistiert. Gerenderte PNG-/PDF-Ausgaben
  bleiben temporäre Materialisierungen, da die Kanalzustellung weiterhin einen Dateipfad benötigt;
  ihre Ablaufmetadaten werden von SQLite verwaltet, ohne JSON-Begleitdateien.
- Verwaltete Canvas-Dokumente verwenden nun das gemeinsame SQLite `plugin_blob_entries` anstelle
  eines standardmäßigen Verzeichnisses `state/canvas/documents`. Der Canvas-Host stellt diese
  Blobs direkt bereit; lokale Dateien werden nur für explizite `host.root`-
  Operatorinhalte oder zur temporären Materialisierung erstellt, wenn ein nachgelagerter Medienleser
  einen Pfad benötigt.
- Auditentscheidungen für Dateiübertragungen verwenden nun das gemeinsame SQLite `plugin_state_entries`
  anstelle des unbegrenzten Laufzeitprotokolls `audit/file-transfer.jsonl`. Doctor
  importiert die veraltete JSONL-Auditdatei in den Plugin-Zustand und entfernt die Quelle
  nach einem fehlerfreien Import.
- ACPX-Prozess-Leases und die Identität der Gateway-Instanz verwenden nun den gemeinsamen SQLite-Plugin-
  Zustand. Doctor importiert die veraltete Datei `gateway-instance-id` in den Plugin-Zustand
  und entfernt die Quelle.
- Von ACPX generierte Wrapper-Skripte und das isolierte Codex-Ausgangsverzeichnis sind temporäre
  Materialisierungen unter dem temporären OpenClaw-Stammverzeichnis und kein dauerhafter OpenClaw-Zustand. Die
  dauerhaften ACPX-Laufzeitdatensätze sind die SQLite-Lease- und Gateway-Instanzzeilen;
  die alte ACPX-Konfigurationsoberfläche `stateDir` wurde entfernt, da dort kein Laufzeitzustand
  mehr geschrieben wird.
- Gateway-Medienanhänge verwenden nun die gemeinsame SQLite-Tabelle `media_blobs` als
  kanonischen Bytespeicher. Lokale Pfade, die an Kompatibilitätsoberflächen für Kanäle und die Sandbox
  zurückgegeben werden, sind temporäre Materialisierungen der Datenbankzeile und nicht der
  dauerhafte Medienspeicher. Laufzeit-Zulassungslisten für Medien enthalten die veralteten Stammverzeichnisse
  `$OPENCLAW_STATE_DIR/media` oder `media` im Konfigurationsverzeichnis nicht mehr; diese Verzeichnisse dienen
  nur als doctor-Importquellen.
- Die Shell-Vervollständigung schreibt keine `$OPENCLAW_STATE_DIR/completions/*`-Cache-
  Dateien mehr. Installations-, doctor-, Aktualisierungs- und Release-Smoke-Pfade verwenden generierte
  Vervollständigungsausgaben oder das Einlesen von Profilen statt dauerhafter Vervollständigungs-Cache-
  Dateien.
- Die Bereitstellung hochgeladener Gateway-Skills verwendet nun gemeinsame Zeilen `skill_uploads` und
  `skill_upload_chunks`. Chunks bleiben während des Uploads einzeln transaktional;
  beim Commit wird anschließend ein einzelnes verifiziertes Archiv-BLOB zusammengesetzt, und die Chunk-
  Zeilen werden entfernt. Das Installationsprogramm erhält nur während einer laufenden Installation
  einen temporär materialisierten Archivpfad. Doctor verwirft den außer Betrieb genommenen einstündigen
  Dateisystem-Bereitstellungsbaum, anstatt vorübergehende Uploads zu importieren.
- Inline-Anhänge von Subagenten werden nicht mehr unter dem Workspace-Pfad
  `.openclaw/attachments/*` materialisiert. Der Erzeugungspfad bereitet SQLite-VFS-Starteinträge vor,
  Inline-Ausführungen übertragen diese Einträge in den Laufzeit-Scratch-Namensraum des jeweiligen Agenten,
  und datenträgergestützte Tools überlagern diesen SQLite-Scratch für Anhangspfade. Die
  alten Spalten der Anhangsverzeichnisregistrierung für Subagentenausführungen und die Bereinigungs-Hooks wurden entfernt.
- Die CLI-Bildhydratisierung verwaltet keine stabilen `openclaw-cli-images`-Cache-
  Dateien mehr. Externe CLI-Backends erhalten weiterhin Dateipfade, diese Pfade sind jedoch
  temporäre Materialisierungen pro Ausführung mit anschließender Bereinigung.
- Cache-Trace-Diagnosen, Anthropic-Nutzlastdiagnosen, Diagnosen des rohen Modellstreams,
  Ereignisse der Diagnosezeitleiste und Gateway-Stabilitätspakete schreiben nun
  SQLite-Zeilen anstelle von Dateien `logs/*.jsonl` oder
  `logs/stability/*.json`.
  Flags und Umgebungsvariablen zum Überschreiben von Laufzeitpfaden wurden entfernt; Export-/Debug-
  Befehle können Dateien explizit aus Datenbankzeilen materialisieren.
- Die macOS-Begleitanwendung verfügt nicht mehr über einen rollierenden `diagnostics.jsonl`-Schreiber. App-
  Protokolle werden in das einheitliche Protokollierungssystem geschrieben, und dauerhafte Gateway-Diagnosen bleiben SQLite-basiert.
- Die Datensatzliste des macOS-Portwächters verwendet nun typisierte gemeinsame SQLite-
  Zeilen `macos_port_guardian_records` anstelle einer JSON-Datei unter Application Support
  oder eines undurchsichtigen Singleton-Blobs. Alle macOS-App-Profile verwenden dieselbe hostglobale native
  Datenbank, da sie lokale Ports des Computers koordinieren. Jeder Ledger-Vorgang
  wird blockiert, solange eine ältere JSON-schreibende App-Kopie ausgeführt wird. Die Migration bindet das stabile
  Dateisperrprotokoll des alten Ledgers nur ein, um eine Momentaufnahme der Quelle zu erstellen und diese später
  erneut zu validieren. Sie löst jede veraltete Zeile anhand aktueller Befehls- und Prozessstartfakten auf,
  ohne diese Sperre zu halten, liest anschließend maßgebliche SQLite-Zeilen erneut ein, wendet den
  Plan an, überprüft jeden Beleg und entfernt die Quelle. Wiederholte Entfernungsversuche planen
  fehlende Zeilen neu, damit außer Betrieb genommene veraltete Belege nicht wiederhergestellt werden können. Die Sperre bleibt
  kurzlebig, damit sie einen älteren Schreiber nach dem Start durch SSH nicht blockieren kann. Die Umstellung erfolgt
  absichtlich nur in eine Richtung: Die Laufzeit im Normalbetrieb liest, projiziert oder schreibt niemals JSON,
  und ein Rollback auf reine JSON-Builds erhält neuere SQLite-Belege nicht.
- Gateway-Singleton-Sperren verwenden nun typisierte gemeinsame SQLite-Zeilen `state_leases` im
  Geltungsbereich `gateway_locks` anstelle von Sperrdateien im temporären Verzeichnis. Die Dokumentation zur Fehlerbehebung
  für Fly und OAuth verweist nun auf die SQLite-Lease-/Authentifizierungsaktualisierungssperre statt
  auf die Bereinigung veralteter Dateisperren.
- Der Status des Gateway-Neustart-Sentinels verwendet jetzt typisierte Zeilen in der gemeinsam genutzten SQLite-Datenbank
  `gateway_restart_sentinel` anstelle von `restart-sentinel.json`; die Laufzeit
  liest Sentinel-Art, Status, Routing, Nachricht, Fortsetzung und Statistiken aus
  typisierten Spalten. Diese Spalten sind maßgeblich; `payload_json` ist nur ein
  Schatten für Wiedergabe und Debugging. Die Laufzeitpfade zum Lesen, Schreiben und Löschen verwenden ausschließlich SQLite.
  Ein begrenztes Zustandsmigrationsmodul wird beim Start und durch Doctor ausgeführt, um vor der normalen Neustartwiederherstellung ein
  validiertes älteres Sentinel nach einem Update zu importieren, die
  typisierte Zeile zu verifizieren und die Quelldatei zu entfernen. Kein Laufzeitmodul im
  regulären Betrieb liest, schreibt oder bereinigt die Legacy-Datei.
- Gateway-Neustartabsicht und Status der Supervisor-Übergabe verwenden jetzt typisierte gemeinsam genutzte
  SQLite-Zeilen `gateway_restart_intent` und `gateway_restart_handoff` anstelle der
  Sidecar-Dateien `gateway-restart-intent.json` und
  `gateway-supervisor-restart-handoff.json`.
- Die Koordination der Gateway-Singleton-Instanz verwendet jetzt typisierte Zeilen `state_leases` unter
  `gateway_locks`, anstatt Dateien `gateway.<hash>.lock` zu schreiben. Die Lease-Zeile
  verwaltet den Sperrinhaber, den Ablaufzeitpunkt, den Heartbeat und die Debug-Nutzlast; SQLite verwaltet die
  atomare Grenze für Erwerb und Freigabe. Die eingestellte Option für das Dateisperrverzeichnis
  wurde entfernt; Tests verwenden direkt die Identität der SQLite-Zeile.
- Der alte, nicht referenzierte Hilfsmechanismus für Cron-Nutzungsberichte, der Dateien unter `cron/runs/*.jsonl`
  durchsuchte, wurde gelöscht. Berichte zum Cron-Ausführungsverlauf lesen Cron-eigene Zeilen `task_runs`.
- Die Neustartwiederherstellung der Hauptsitzung ermittelt mögliche Agenten jetzt über die
  SQLite-Registry `agent_databases`, anstatt Verzeichnisse unter `agents/*/sessions`
  zu durchsuchen.
- Die Wiederherstellung nach einer beschädigten Gemini-Sitzung löscht jetzt ausschließlich die SQLite-Sitzungszeile;
  sie benötigt kein Legacy-Gate `storePath` mehr und versucht nicht mehr, einen abgeleiteten
  JSONL-Pfad des Transkripts zu entfernen.
- Die Verarbeitung von Pfadüberschreibungen behandelt literale Umgebungswerte `undefined`/`null`
  jetzt als nicht gesetzt und verhindert dadurch bei Tests oder Shell-Übergaben versehentlich im Repository-Stamm erstellte
  Datenbanken `undefined/state/*.sqlite`.
- Fingerabdrücke des Konfigurationszustands verwenden jetzt typisierte gemeinsam genutzte SQLite-Zeilen `config_health_entries`
  anstelle von `logs/config-health.json`, sodass die normale Konfigurationsdatei
  das einzige Konfigurationsdokument ohne Anmeldedaten bleibt. Die macOS-Begleitanwendung behält nur
  prozesslokalen Zustandsstatus und erstellt die alte JSON-Sidecar-Datei nicht erneut.
- Die Laufzeit für Authentifizierungsprofile importiert oder schreibt keine JSON-Dateien mit Anmeldedaten mehr. Der
  kanonische Speicher für Anmeldedaten ist SQLite; `auth-profiles.json`, agentenspezifische
  `auth.json` und gemeinsam genutzte `credentials/oauth.json` dienen Doctor als Migrationseingaben
  und werden nach dem Import entfernt.
- Tests zum Speichern und Status von Authentifizierungsprofilen prüfen jetzt direkt typisierte SQLite-Authentifizierungstabellen
  und verwenden Legacy-Dateinamen für Authentifizierungsprofile nur als Migrationseingaben für Doctor.
- `openclaw secrets apply` bereinigt nur die Konfigurationsdatei, die Umgebungsdatei und den SQLite-Speicher
  für Authentifizierungsprofile. Die Funktion enthält keine Kompatibilitätslogik mehr, die eingestellte agentenspezifische
  `auth.json` bearbeitet; Doctor ist für den Import und das Löschen dieser Datei zuständig.
- Hermes-Pläne zur Geheimnismigration importieren API-Schlüsselprofile direkt
  in den SQLite-Speicher für Authentifizierungsprofile und wenden sie dort an. Sie schreiben oder verifizieren
  `auth-profiles.json` nicht mehr als Zwischenziel.
- Benutzerorientierte Dokumentation zur Authentifizierung beschreibt jetzt
  `state/openclaw.sqlite#table/auth_profile_stores/<agentDir>`, anstatt
  Benutzer anzuweisen, `auth-profiles.json` zu prüfen oder zu kopieren; Legacy-Namen für OAuth-/Authentifizierungs-JSON-Dateien
  bleiben nur als Doctor-Importeingaben dokumentiert.
- MCP-OAuth-Sitzungen verwenden jetzt versionierte Zeilen `mcp_oauth_stores` in der gemeinsam genutzten
  `state/openclaw.sqlite`. SDK-eigene Token-, Clientregistrierungs- und Ermittlungsobjekte
  bleiben eine einzige validierte JSON-Nutzlast, damit Erweiterungsfelder von Abhängigkeiten
  erhalten bleiben, während jeder Lese-/Änderungs-/Schreibvorgang in einer kurzen Kysely-
  Transaktion festgeschrieben wird. Eine gemeinsam genutzte SQLite-Lease serialisiert Aktualisierung, Anmeldung und Abmeldung;
  eingebettete MCP-Transporte lassen das MCP SDK Aktualisierungen nicht mehr außerhalb dieser
  Lease durchführen. Doctor importiert und entfernt ausschließlich eingestellte Speicher `mcp-oauth/*.json`
  mit Quellbelegen; die Laufzeit besitzt keinen Datei-Fallback.
- Hilfsfunktionen für Kernzustandspfade stellen die eingestellte Datei `credentials/oauth.json`
  nicht mehr bereit. Der Legacy-Dateiname ist lokal auf den Doctor-Importpfad für Authentifizierung beschränkt.
- Dokumentation zu Installation, Sicherheit, Onboarding, Modellauthentifizierung und SecretRef beschreibt jetzt
  SQLite-Zeilen für Authentifizierungsprofile sowie Sicherung und Migration des Gesamtzustands anstelle von
  agentenspezifischen JSON-Dateien für Authentifizierungsprofile.
- Die PI-Modellermittlung übergibt jetzt kanonische Anmeldedaten an den speicherinternen
  Authentifizierungsspeicher `pi-coding-agent`. Sie erstellt, bereinigt oder schreibt während der Ermittlung
  keine agentenspezifische `auth.json` mehr.
- Auslöse- und Routing-Einstellungen für Voice Wake verwenden jetzt typisierte gemeinsam genutzte SQLite-Tabellen
  anstelle von `settings/voicewake.json`, `settings/voicewake-routing.json` oder
  undurchsichtigen generischen Zeilen; Doctor importiert die Legacy-JSON-Dateien und entfernt sie nach einer
  erfolgreichen Migration.
- Der Status der Updateprüfung verwendet jetzt eine typisierte gemeinsam genutzte Zeile `update_check_state` anstelle von
  `update-check.json` oder einem undurchsichtigen generischen Blob; Doctor importiert
  die Legacy-JSON-Datei und entfernt sie nach einer erfolgreichen Migration.
- Der Konfigurationszustand verwendet jetzt typisierte gemeinsam genutzte Zeilen `config_health_entries` anstelle
  von `logs/config-health.json` oder einem undurchsichtigen generischen Blob; Doctor
  importiert die Legacy-JSON-Datei und entfernt sie nach einer erfolgreichen Migration.
- Genehmigungen für Plugin-Unterhaltungsbindungen verwenden jetzt typisierte
  Zeilen `plugin_binding_approvals` anstelle eines undurchsichtigen gemeinsam genutzten SQLite-Zustands oder
  `plugin-binding-approvals.json`; die Legacy-Datei dient Doctor als Migrationseingabe.
- Generische Bindungen der aktuellen Unterhaltung speichern jetzt typisierte
  Zeilen `current_conversation_bindings`, anstatt
  `bindings/current-conversations.json` neu zu schreiben; Doctor importiert die Legacy-JSON-Datei und
  entfernt sie nach einer erfolgreichen Migration.
- Synchronisierungsjournale importierter Quellen von Memory Wiki speichern jetzt eine SQLite-Plugin-Zustandszeile
  pro Vault-/Quellschlüssel, anstatt `.openclaw-wiki/source-sync.json` neu zu schreiben;
  der Migrations-Provider importiert und entfernt das Legacy-JSON-Journal.
- Datensätze zu ChatGPT-Importausführungen von Memory Wiki speichern jetzt eine SQLite-Plugin-Zustandszeile
  pro Vault-/Ausführungs-ID, anstatt `.openclaw-wiki/import-runs/*.json` zu schreiben.
  Rollback-Snapshots bleiben explizite Vault-Dateien, bis die Archivierung von Snapshots
  der Importausführung in den Blob-Speicher verschoben wird.
- Kompilierte Digests von Memory Wiki speichern jetzt komprimierte SQLite-Plugin-Blob-Zeilen,
  anstatt `.openclaw-wiki/cache/agent-digest.json` und
  `.openclaw-wiki/cache/claims.jsonl` zu schreiben. Der Cache kann neu aufgebaut werden, daher
  löscht Doctor alte Cache-Dateien, ohne sie zu importieren.
- Die Nachverfolgung installierter ClawHub-Skills speichert jetzt eine SQLite-Plugin-Zustandszeile pro
  Workspace/Skill, anstatt die Sidecar-Dateien `.clawhub/lock.json` und
  `.clawhub/origin.json` zur Laufzeit zu schreiben oder zu lesen. Der Laufzeitcode verwendet Zustandsobjekte
  für nachverfolgte Installationen anstelle dateiförmiger Lockfile-/Ursprungsabstraktionen. Doctor
  importiert die Legacy-Sidecar-Dateien aus konfigurierten Agenten-Workspaces und entfernt sie
  nach einem fehlerfreien Import.
- Der Index installierter Plugins liest und schreibt jetzt die typisierte gemeinsam genutzte SQLite-
  Singleton-Zeile `installed_plugin_index` anstelle von `plugins/installs.json`; die
  Legacy-JSON-Datei dient nur als Doctor-Migrationseingabe und wird nach dem Import entfernt.
- Die Legacy-Pfadhilfsfunktion `plugins/installs.json` befindet sich jetzt im Legacy-Code
  von Doctor. Laufzeitmodule für den Plugin-Index stellen ausschließlich SQLite-gestützte Persistenzoptionen
  bereit, keinen JSON-Dateipfad.
- Gateway-Neustart-Sentinel, Neustartabsicht und Status der Supervisor-Übergabe verwenden jetzt
  typisierte gemeinsam genutzte SQLite-Zeilen (`gateway_restart_sentinel`,
  `gateway_restart_intent` und `gateway_restart_handoff`) anstelle generischer
  undurchsichtiger Blobs. Der Neustartcode der Laufzeit besitzt keinen dateiförmigen Vertrag für Sentinel, Absicht oder Übergabe.
- Matrix-Synchronisierungscache, Speichermetadaten, Thread-Bindungen, Marker für die Deduplizierung eingehender Nachrichten,
  Cooldown-Status der Startverifizierung, IndexedDB-Kryptografie-Snapshots des SDK,
  Anmeldedaten und Wiederherstellungsschlüssel verwenden jetzt gemeinsam genutzte SQLite-Tabellen für Plugin-Zustand und -Blobs.
  Laufzeit-Pfadstrukturen stellen keinen Metadatenpfad `storage-meta.json` mehr
  bereit; dieser Dateiname dient nur als Legacy-Migrationseingabe. Der Plan zum Import ihrer Legacy-JSON-Daten
  befindet sich in der Einrichtungs-/Doctor-Migrationsoberfläche des Matrix-Plugins. Marker für die
  Deduplizierung eingehender Nachrichten verwenden die beanspruchbare Kerndeduplizierung (Namensräume `matrix.inbound-dedupe.*`
  in der gemeinsam genutzten Zustandsdatenbank); die Matrix-Doctor-Zustandsmigration importiert
  die eingestellten root-spezifischen Zeilen `inbound-dedupe` und `inbound-dedupe.json` einmalig,
  anschließend liest die Laufzeit ausschließlich den Speicher für beanspruchbare Deduplizierung.
- Beim Start durchsucht, meldet oder vervollständigt Matrix keinen Legacy-Matrix-Dateizustand mehr.
  Erkennung von Matrix-Dateien, Erstellung von Legacy-Kryptografie-Snapshots, Migrationsstatus für die Wiederherstellung von Raumschlüsseln,
  Import und Entfernung der Quelle liegen vollständig in der Verantwortung von Doctor.
- Matrix-Laufzeit-Barrels für Migrationen wurden entfernt. Hilfsfunktionen zur Erkennung
  und Änderung von Legacy-Zustand und -Kryptografie werden direkt von Matrix Doctor importiert, statt
  Teil der Laufzeit-API-Oberfläche zu sein.
- Marker für die Wiederverwendung von Matrix-Migrations-Snapshots befinden sich jetzt im SQLite-Plugin-Zustand
  anstelle von `matrix/migration-snapshot.json`; Doctor kann dasselbe
  verifizierte Archiv vor der Migration weiterhin wiederverwenden, ohne eine Sidecar-Zustandsdatei zu schreiben.
- Nostr-Bus-Cursor und Status der Profilveröffentlichung verwenden jetzt den gemeinsam genutzten SQLite-Plugin-
  Zustand. Ihr Plan zum Import von Legacy-JSON-Daten befindet sich in der Einrichtungs-/Doctor-
  Migrationsoberfläche des Nostr-Plugins.
- Active Memory-Sitzungsschalter verwenden jetzt den gemeinsam genutzten SQLite-Plugin-Zustand anstelle von
  `session-toggles.json`; beim erneuten Aktivieren des Speichers wird die Zeile gelöscht, anstatt
  ein JSON-Objekt neu zu schreiben.
- Vorschläge und Überprüfungszähler von Skill Workshop verwenden jetzt den gemeinsam genutzten SQLite-Plugin-
  Zustand anstelle Workspace-spezifischer Speicher `skill-workshop/<workspace>.json`. Jeder
  Vorschlag ist eine separate Zeile unter `skill-workshop/proposals`, und der
  Überprüfungszähler ist eine separate Zeile unter `skill-workshop/reviews`.
- Ausführungen von Skill-Workshop-Prüfer-Subagenten verwenden jetzt die Laufzeitauflösung für Sitzungstranskripte,
  anstatt Sidecar-Sitzungspfade `skill-workshop/<sessionId>.json` zu erstellen.
- ACPX-Prozess-Leases verwenden jetzt den gemeinsam genutzten SQLite-Plugin-Zustand unter
  `acpx/process-leases` anstelle einer dateiweiten Registry `process-leases.json`.
  Jede Lease wird als eigene Zeile gespeichert, wodurch die Entfernung veralteter Prozesse beim Start
  ohne einen Laufzeitpfad zum Neuschreiben von JSON erhalten bleibt.
- ACPX-Wrapper-Skripte und das isolierte Codex-Basisverzeichnis werden im
  temporären OpenClaw-Stammverzeichnis generiert. Sie werden bei Bedarf neu erstellt und dienen weder als Sicherungs-
  noch als Migrationseingaben.
- Die Persistenz der Registry für Subagentenausführungen verwendet typisierte gemeinsam genutzte Zeilen `subagent_runs`. Der
  alte Pfad `subagents/runs.json` dient jetzt nur noch Doctor als Bereinigungseingabe. Doctor
  beansprucht ihn unter der Sperre für die Zustandswartung, zeichnet die Verwerfungsentscheidung in
  SQLite auf und entfernt ihn, ohne vorübergehenden Ausführungsstatus zu importieren. Es verbleiben keine JSON-
  Leser, -Schreiber, -Caches oder -Fallbacks in der Laufzeit; die versionsübergreifende Wiederherstellung ausschließlich dateibasierter
  laufender Ausführungen wird an dieser Ablösungsgrenze bewusst nicht unterstützt.
  Laufzeittests erstellen keine ungültigen oder leeren Fixtures `runs.json` mehr, um das
  Registry-Verhalten nachzuweisen; sie legen SQLite-Zeilen direkt an und lesen sie direkt.
- Die Sicherung stellt das Zustandsverzeichnis vor der Archivierung bereit, kopiert Nicht-Datenbankdateien,
  erstellt Datenbank-Snapshots mittels Online-Sicherung plus Offline-`VACUUM`, lässt aktive WAL-/SHM-Sidecar-Dateien aus, zeichnet
  Snapshot-Metadaten im Archivmanifest auf und erfasst
  abgeschlossene Sicherungsausführungen zusammen mit dem Archivmanifest in SQLite. `openclaw backup
create` validiert das geschriebene Archiv standardmäßig; `--no-verify` ist der
  explizite schnelle Pfad.
- `openclaw backup restore` validiert das Archiv vor dem Extrahieren, verwendet das
  normalisierte Manifest des Prüfers erneut und stellt verifizierte Manifest-Assets an ihren
  aufgezeichneten Quellpfaden wieder her. Für Schreibvorgänge ist `--yes` erforderlich; `--dry-run`
  wird für einen Wiederherstellungsplan unterstützt.
- Der alte Filter für flüchtige Sicherungspfade wurde gelöscht. Die Sicherung benötigt keine
  Überspringliste für eine laufende tar-Archivierung von Legacy-JSON-/JSONL-Dateien für Sitzungen oder Cron mehr, da SQLite-
  Snapshots vor der Erstellung des Archivs bereitgestellt werden.
- Bei der einfachen Einrichtung und der Vorbereitung des Arbeitsbereichs im Onboarding werden keine
  `agents/<agentId>/sessions/`-Verzeichnisse mehr erstellt. Dabei werden nur Konfiguration und Arbeitsbereich erstellt;
  SQLite-Sitzungszeilen und Transkriptzeilen werden bei Bedarf in der
  agentenspezifischen Datenbank erstellt.
- Die Reparatur von Sicherheitsberechtigungen gilt jetzt für die globale und die agentenspezifischen SQLite-
  Datenbanken sowie die WAL/SHM-Begleitdateien statt für `sessions.json` und Transkript-
  JSONL-Dateien.
- Die Laufzeitnamen der Sandbox-Registrierung beschreiben SQLite-Registrierungsarten jetzt direkt,
  statt Legacy-Terminologie der JSON-Registrierung im aktiven Speicher weiterzuführen.
- `openclaw reset --scope config+creds+sessions` entfernt agentenspezifische
  `openclaw-agent.sqlite`-Datenbanken einschließlich WAL/SHM-Begleitdateien und nicht nur veraltete
  `sessions/`-Verzeichnisse.
- Die aggregierten Sitzungshilfsfunktionen des Gateways verwenden jetzt eintragsorientierte Namen:
  `loadCombinedSessionEntriesForGateway` gibt `{ databasePath, entries }` zurück.
  Die alte Benennung des kombinierten Speichers wurde aus den Laufzeitaufrufern entfernt.
- Das Seeding des Docker-MCP-Kanals schreibt jetzt die Hauptsitzungszeile und Transkript-
  ereignisse in die agentenspezifische SQLite-Datenbank, statt
  `sessions.json` und ein JSONL-Transkript zu erstellen.
- Der mitgelieferte session-memory-Hook löst den Kontext der vorherigen Sitzung jetzt anhand von
  `{agentId, sessionId}` aus SQLite auf. Er durchsucht, speichert oder synthetisiert keine
  Transkriptpfade oder `workspace/sessions`-Verzeichnisse mehr.
- Der mitgelieferte command-logger-Hook schreibt Befehlsauditzeilen jetzt in die gemeinsam genutzte
  SQLite-Tabelle `command_log_entries`, statt sie an
  `logs/commands.log` anzuhängen.
- Kanal-Kopplungs-Zulassungslisten stellen zur Laufzeit jetzt ausschließlich SQLite-gestützte Lese-/Schreibhilfsfunktionen
  bereit. Der veraltete Pfadauflöser des Plugin-SDK bleibt aus Gründen der Migrations-
  kompatibilität erhalten; Dateileser sind ausschließlich im Doctor-Code für die Zustandsmigration enthalten.
- `migration_runs` zeichnet Ausführungen der Legacy-Zustandsmigration mit Status,
  Zeitstempeln und JSON-Berichten auf.
- `migration_sources` zeichnet jede importierte Legacy-Dateiquelle mit Hash, Größe,
  Datensatzanzahl, Zieltabelle, Ausführungs-ID, Status und Zustand der Quellentfernung auf.
- `backup_runs` zeichnet Pfade von Sicherungsarchiven, Status und JSON-Manifeste auf.
- Das globale Schema enthält keine ungenutzte Registrierungstabelle `agents`. Die Erkennung von Agenten-
  datenbanken ist die kanonische `agent_databases`-Registrierung, bis die Laufzeit
  einen echten Eigentümer für Agentendatensätze hat.
- Die Konfiguration des generierten Modellkatalogs wird in typisierten globalen SQLite-
  Zeilen `agent_model_catalogs` gespeichert, die nach Agentenverzeichnis geschlüsselt sind. Laufzeitaufrufer verwenden
  `ensureOpenClawModelCatalog`; im Laufzeitcode gibt es keine Kompatibilitäts-API
  `models.json`. Die Implementierung schreibt in SQLite, und die eingebettete PI-Registrierung wird
  aus dieser gespeicherten Nutzlast geladen, ohne eine Datei `models.json` zu erstellen.
- Der optionale Export `memory.qmd.sessions` liest kanonische Transkriptzeilen aus
  der agentenspezifischen Datenbank und materialisiert bereinigtes Markdown unter dem QMD-Ausgangsverzeichnis
  als explizites QMD-Eingabeartefakt. QMD-Sitzungssammlungen und Zuordnungen von Artefakt-
  identitäten bleiben daher Teil der konfigurierten Brücke zum externen Tool;
  sie sind kein zweiter kanonischer Transkriptspeicher.
- QMDs eigene `index.sqlite`, die YAML-Sammlungskonfiguration und Modelldownloads bleiben
  Artefakte des externen Tools unter `~/.openclaw/agents/<agentId>/qmd`; sie werden nicht
  nach `plugin_blob_entries` gespiegelt. Die OpenClaw-eigene QMD-Koordination ist
  datenbankorientiert: Gemeinsam genutzte `state_leases` serialisieren Einbettungen global und agentenspezifische
  `state_leases` serialisieren Schreibvorgänge für Sammlungen, Aktualisierungen und Einbettungen. Die Laufzeit erstellt keine
  QMD-Sperr-Begleitdateien.
- Das optionale Plugin `memory-lancedb` erstellt
  `~/.openclaw/memory/lancedb` nicht mehr als impliziten, von OpenClaw verwalteten Speicher. Es ist ein
  externes LanceDB-Backend und bleibt deaktiviert, bis der Betreiber einen
  expliziten `dbPath` konfiguriert.
- `check:database-first-legacy-stores` schlägt bei neuem Laufzeitquellcode fehl, der
  Legacy-Speichernamen mit schreibenden Dateisystem-APIs kombiniert. Die Prüfung schlägt außerdem bei Laufzeit-
  quellcode fehl, der die außer Betrieb genommenen Transkriptbrücken-Markierungen
  `transcriptLocator` oder `sqlite-transcript://...` wieder einführt. Migrations-, Doctor-, Import-
  und expliziter Exportcode außerhalb von Sitzungen bleibt zulässig. Umfassendere Namen von Legacy-Verträgen
  wie `sessionFile`, `storePath` und alte Dateizeitalter-Fassaden von `SessionManager`
  haben weiterhin aktuelle Eigentümer und benötigen separate Schutzmaßnahmen für die Migration,
  bevor sie zu einer erforderlichen Vorabprüfung werden können. Die Schutzmaßnahme deckt jetzt außerdem
  Laufzeitspeicher `cache/*.json`, generische
  `thread-bindings.json`-Begleitdateien, Cron-Zustands-/Ausführungsprotokoll-JSON, Konfigurationszustands-JSON,
  Neustart- und Sperr-Begleitdateien, Voice-Wake-Einstellungen, Genehmigungen für Plugin-Bindungen,
  das JSON-Verzeichnis installierter Plugins, File-Transfer-Audit-JSONL, Memory-Wiki-Aktivitäts-
  protokolle, das alte mitgelieferte Textprotokoll `command-logger` und JSONL-
  Diagnoseoptionen für den pi-mono-Rohdatenstrom ab. Sie verbietet außerdem alte Legacy-Modulnamen des Doctors auf Stammebene, sodass
  Kompatibilitätscode unter `src/commands/doctor/` verbleibt. Android-Debug-Handler
  verwenden außerdem logcat-/In-Memory-Ausgaben, statt `camera_debug.log`- oder
  `debug_logs.txt`-Cache-Dateien zwischenzuspeichern.

## Form des Zielschemas

Halten Sie Schemas explizit. Vom Host verwalteter Laufzeitstatus verwendet typisierte Tabellen. Vom Plugin verwalteter
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
Tabelle `agents` hinzu, bis ein tatsächlicher Eigentümer für Agent-Datensätze vorhanden ist; die Agent-Konfiguration verbleibt in
`openclaw.json`.

## Form der Doctor-Migration

Doctor sollte einen expliziten Migrationsschritt aufrufen, der berichtsfähig ist und
sicher erneut ausgeführt werden kann:

```bash
openclaw doctor --fix
```

`openclaw doctor --fix` ruft die Implementierung der Statusmigration nach
der gewöhnlichen Konfigurationsvorprüfung auf und erstellt vor dem Import eine verifizierte Sicherung. Der Start der Laufzeit
und `openclaw migrate` dürfen keine veralteten OpenClaw-Statusdateien importieren.

Migrationseigenschaften:

- Ein Migrationsdurchlauf erkennt alle Quellen veralteter Dateien und erstellt einen Plan,
  bevor Änderungen vorgenommen werden.
- Doctor erstellt vor dem Import
  veralteter Dateien ein verifiziertes Sicherungsarchiv vor der Migration.
- Importe sind idempotent und werden anhand von Quellpfad, mtime, Größe, Hash und Zieltabelle
  identifiziert.
- Erfolgreich verarbeitete Quelldateien werden entfernt oder archiviert, nachdem die Zieldatenbank
  den Commit abgeschlossen hat.
- Fehlgeschlagene Importe lassen die Quelle unverändert und erfassen eine Warnung in
  `migration_runs`.
- Der Laufzeitcode liest nur SQLite, nachdem die Migration vorhanden ist.
- Ein Pfad zum Downgrade oder Export in Laufzeitdateien ist nicht erforderlich.

## Migrationsinventar

Verschieben Sie Folgendes in die globale Datenbank:

- Laufzeitschreibvorgänge der Task-Registry verwenden jetzt die gemeinsame Datenbank; der nicht ausgelieferte
  `tasks/runs.sqlite`-Sidecar-Importer wurde gelöscht. Snapshot-Speicherungen führen Upserts anhand der Task-
  ID durch und löschen nur fehlende Task-/Delivery-Zeilen.
- Laufzeitschreibvorgänge von Task Flow verwenden jetzt die gemeinsame Datenbank; der nicht ausgelieferte
  `tasks/flows/registry.sqlite`-Sidecar-Importer wurde gelöscht. Snapshot-Speicherungen
  führen Upserts anhand der Flow-ID durch und löschen nur fehlende Flow-Zeilen.
- Laufzeitschreibvorgänge des Plugin-Zustands verwenden jetzt die gemeinsame Datenbank; der nicht ausgelieferte
  `plugin-state/state.sqlite`-Sidecar-Importer wurde gelöscht.
- Die integrierte Speichersuche verwendet nicht mehr standardmäßig `memory/<agentId>.sqlite`; ihre
  Indextabellen befinden sich in der Datenbank des zuständigen Agenten, und die explizite
  Aktivierung des `memorySearch.store.path`-Sidecars wurde in die
  Doctor-Konfigurationsmigration verlagert.
- Die Neuindizierung des integrierten Speichers setzt nur speichereigene Tabellen in der Agentendatenbank zurück.
  Sie darf nicht die gesamte SQLite-Datei ersetzen, da dieselbe Datenbank
  Sitzungen, Transkripte, VFS-Zeilen, Artefakte und Laufzeit-Caches enthält.
- Sandbox-Container-/Browser-Registrys aus monolithischem und fragmentiertem JSON. Laufzeitschreibvorgänge
  verwenden jetzt die gemeinsame Datenbank; der Import von Legacy-JSON bleibt bestehen.
- Cron-Auftragsdefinitionen, Zeitplanzustand und Ausführungsverlauf verwenden jetzt gemeinsames SQLite;
  Doctor importiert/entfernt die Legacy-Dateien `jobs.json`, `jobs-state.json` und
  `cron/runs/*.jsonl`
- Geräteidentität/-authentifizierung, Push, Update-Prüfung, Zusagen, OpenRouter-Modell-
  Cache, Index installierter Plugins und App-Server-Bindungen
- Kopplungs- und Bootstrap-Datensätze für Geräte/Nodes verwenden jetzt typisierte SQLite-Tabellen
- Abonnenten von Gerätekopplungsbenachrichtigungen und Markierungen zugestellter Anfragen verwenden jetzt die
  gemeinsame SQLite-Tabelle für Plugin-Zustände anstelle von `device-pair-notify.json`.
- Anrufdatensätze für Sprachanrufe verwenden jetzt die gemeinsame SQLite-Tabelle für Plugin-Zustände unter dem
  Namensraum `voice-call` / `calls` anstelle von `calls.jsonl`; die Plugin-CLI
  verfolgt und fasst den SQLite-basierten Anrufverlauf zusammen.
- QQBot-Gateway-Sitzungen, Datensätze bekannter Benutzer und der Ref-Index-Zitat-Cache verwenden jetzt
  den SQLite-Plugin-Zustand unter den `qqbot`-Namensräumen (`gateway-sessions`,
  `known-users`, `ref-index`) anstelle von `session-*.json`, `known-users.json`
  und `ref-index.jsonl`. Diese Legacy-Dateien sind Caches und werden nicht migriert.
- Discord-Einstellungen für die Modellauswahl, Hashes der Befehlsbereitstellung und Thread-Bindungen
  verwenden jetzt den SQLite-Plugin-Zustand unter den `discord`-Namensräumen
  (`model-picker-preferences`, `command-deploy-hashes`, `thread-bindings`)
  anstelle von `model-picker-preferences.json`, `command-deploy-cache.json` und
  `thread-bindings.json`; die Discord-Doctor-/Einrichtungsmigration importiert und
  entfernt die Legacy-Dateien.
- BlueBubbles-Aufholcursor und Markierungen zur Deduplizierung eingehender Daten verwenden jetzt den SQLite-Plugin-
  Zustand unter den `bluebubbles`-Namensräumen (`catchup-cursors`, `inbound-dedupe`)
  anstelle von `bluebubbles/catchup/*.json` und
  `bluebubbles/inbound-dedupe/*.json`; die BlueBubbles-Doctor-/Einrichtungsmigration
  importiert und entfernt die Legacy-Dateien.
- Telegram-Update-Offsets, Sticker-Cache-Einträge, Nachrichten-Cache-
  Einträge für Antwortketten, Cache-Einträge gesendeter Nachrichten, Cache-Einträge für Themennamen und Thread-
  Bindungen verwenden jetzt den SQLite-Plugin-Zustand unter den `telegram`-Namensräumen
  (`update-offsets`, `sticker-cache`, `message-cache`, `sent-messages`,
  `topic-names`, `thread-bindings`) anstelle von `update-offset-*.json`,
  `sticker-cache.json`, `*.telegram-messages.json`,
  `*.telegram-sent-messages.json`, `*.telegram-topic-names.json` und
  `thread-bindings-*.json`; die Telegram-Doctor-/Einrichtungsmigration importiert und
  entfernt die Legacy-Dateien.
- iMessage-Aufholcursor, Zuordnungen kurzer Antwort-IDs und Deduplizierungszeilen für gesendete Echos
  verwenden jetzt den SQLite-Plugin-Zustand unter den `imessage`-Namensräumen (`catchup-cursors`,
  `reply-cache`, `sent-echoes`) anstelle von `imessage/catchup/*.json`,
  `imessage/reply-cache.jsonl` und `imessage/sent-echoes.jsonl`; die iMessage-
  Doctor-/Einrichtungsmigration importiert und entfernt die Legacy-Dateien.
- Microsoft Teams-Unterhaltungen, Umfragen, SSO-Token und Feedback-Lerndaten
  verwenden jetzt Namensräume des SQLite-Plugin-Zustands (`conversations`, `polls`, `sso-tokens`,
  `feedback-learnings`) anstelle von `msteams-conversations.json`,
  `msteams-polls.json`, `msteams-sso-tokens.json` und `*.learnings.json`; die
  Microsoft Teams-Doctor-/Einrichtungsmigration importiert und archiviert die Legacy-Dateien.
  Ausstehende Uploads sind ein kurzlebiger SQLite-Cache, und alte JSON-Cache-Dateien werden
  nicht migriert.
- Matrix-Synchronisierungscache, Speichermetadaten, Thread-Bindungen, Markierungen zur Deduplizierung eingehender Daten,
  Abklingzustand der Startüberprüfung, Anmeldedaten, Wiederherstellungsschlüssel und kryptografische SDK-
  IndexedDB-Snapshots verwenden jetzt SQLite-Plugin-Zustands-/Blob-Namensräume unter
  `matrix` (`sync-store`, `storage-meta`, `thread-bindings`,
  `matrix.inbound-dedupe.*` über die zentrale beanspruchbare Deduplizierung,
  `startup-verification`, `credentials`, `recovery-key`, `idb-snapshots`)
  anstelle von `bot-storage.json`, `storage-meta.json`, `thread-bindings.json`,
  `inbound-dedupe.json`, `startup-verification.json`, `credentials.json`,
  `recovery-key.json` und `crypto-idb-snapshot.json`; die Matrix-Doctor-/Einrichtungsmigration
  importiert und entfernt diese Legacy-Dateien (und die eingestellten SQLite-Zeilen je Stammverzeichnis
  `inbound-dedupe`) aus kontobezogenen Matrix-Speicherstammverzeichnissen.
- Nostr-Bus-Cursor und der Veröffentlichungszustand von Profilen verwenden jetzt den SQLite-Plugin-Zustand unter den
  `nostr`-Namensräumen (`bus-state`, `profile-state`) anstelle von
  `bus-state-*.json` und `profile-state-*.json`; die Nostr-Doctor-/Einrichtungsmigration
  importiert und entfernt die Legacy-Dateien.
- Active Memory-Sitzungsumschalter verwenden jetzt den SQLite-Plugin-Zustand unter
  `active-memory/session-toggles` anstelle von `session-toggles.json`.
- Vorschlagswarteschlangen und Überprüfungszähler von Skill Workshop verwenden jetzt den SQLite-Plugin-Zustand
  unter `skill-workshop/proposals` und `skill-workshop/reviews` anstelle von
  arbeitsbereichsspezifischen `skill-workshop/<workspace>.json`-Dateien.
- Warteschlangen für ausgehende Zustellungen und Sitzungszustellungen nutzen jetzt gemeinsam die globale SQLite-
  Tabelle `delivery_queue_entries` unter separaten Warteschlangennamen
  (`outbound-delivery`, `session-delivery`) anstelle dauerhafter
  Dateien `delivery-queue/*.json`, `delivery-queue/failed/*.json` und
  `session-delivery-queue/*.json`. Der Doctor-Schritt für Legacy-Zustände importiert
  ausstehende und fehlgeschlagene Zeilen, entfernt veraltete Zustellmarkierungen und löscht die alten
  JSON-Dateien nach dem Import. Felder für Hot-Routing und Wiederholungsversuche sind typisierte Spalten; die
  JSON-Nutzlast bleibt nur für Wiedergabe/Debugging erhalten.
- ACPX-Prozess-Leases verwenden jetzt den SQLite-Plugin-Zustand unter `acpx/process-leases`
  anstelle von `process-leases.json`.
- Metadaten von Sicherungs- und Migrationsläufen

Diese in Agentendatenbanken verschieben:

- Stammverzeichnisse von Agentensitzungen und kompatibilitätsförmige Nutzlasten von Sitzungseinträgen. Für
  Laufzeitschreibvorgänge erledigt: Häufig verwendete Sitzungsmetadaten sind in `sessions` abfragbar, während die
  vollständige Legacy-förmige `SessionEntry`-Nutzlast in `session_entries` verbleibt.
- Transkriptereignisse von Agenten. Für Laufzeitschreibvorgänge erledigt.
- Compaction-Prüfpunkte und Transkript-Snapshots. Für Laufzeitschreibvorgänge erledigt:
  Transkriptkopien von Prüfpunkten sind SQLite-Transkriptzeilen, und Prüfpunkt-
  Metadaten werden in `transcript_snapshots` erfasst. Gateway-Prüfpunkt-Hilfsfunktionen
  bezeichnen diese Werte jetzt als Transkript-Snapshots statt als Quelldateien.
- VFS-Scratch-/Arbeitsbereichsnamensräume von Agenten. Für VFS-Laufzeitschreibvorgänge erledigt.
- Nutzlasten von Subagent-Anhängen. Für Laufzeitschreibvorgänge erledigt: Sie sind SQLite-VFS-
  Seed-Einträge und niemals dauerhafte Arbeitsbereichsdateien.
- Tool-Artefakte. Für Laufzeitschreibvorgänge erledigt.
- Ausführungsartefakte. Für Worker-Laufzeitschreibvorgänge über die agentenspezifische
  Tabelle `run_artifacts` erledigt.
- Agentenlokale Laufzeit-Caches. Für bereichsbezogene Cache-Schreibvorgänge der Worker-Laufzeit über
  die agentenspezifische Tabelle `cache_entries` erledigt. Gateway-weite Modell-Caches verbleiben in der
  globalen Datenbank, sofern sie nicht agentenspezifisch werden.
- ACP-Protokolle übergeordneter Streams. Für Laufzeitschreibvorgänge erledigt.
- Sitzungen des ACP-Wiedergabe-Ledgers. Für Laufzeitschreibvorgänge über
  `acp_replay_sessions` und `acp_replay_events` erledigt; das Legacy-Element `acp/event-ledger.json`
  bleibt nur als Doctor-Eingabe bestehen.
- ACP-Sitzungsmetadaten. Für Laufzeitschreibvorgänge über `acp_sessions` erledigt; Legacy-
  Blöcke `entry.acp` in `sessions.json` dienen nur als Eingabe für die Doctor-Migration.
- Trajektorien-Sidecars, wenn es sich nicht um explizite Exportdateien handelt. Für Laufzeit-
  schreibvorgänge erledigt: Die Trajektorienerfassung schreibt `trajectory_runtime_events`-
  Zeilen in die Agentendatenbank und spiegelt ausführungsbezogene Artefakte in SQLite. Legacy-Sidecars dienen nur als Doctor-
  Importeingaben; der Export kann neue JSONL-Ausgaben für Support-Bundles materialisieren,
  liest oder migriert jedoch zur Laufzeit keine alten Trajektorien-/Transkript-Sidecars.
  Die Laufzeit-Trajektorienerfassung stellt den SQLite-Geltungsbereich bereit; JSONL-Pfadhilfsfunktionen sind
  auf Export-/Debug-Unterstützung beschränkt und werden nicht aus dem Laufzeitmodul erneut exportiert.
  Trajektorienmetadaten des eingebetteten Runners erfassen die `{agentId, sessionId, sessionKey}`-
  Identität, anstatt einen Transkript-Locator dauerhaft zu speichern.

Diese vorerst dateibasiert belassen:

- `openclaw.json`
- Provider- oder CLI-Anmeldedatendateien
- Plugin-/Paketmanifeste
- Benutzerarbeitsbereiche und Git-Repositorys, wenn der Festplattenmodus ausgewählt ist
- Protokolle, die für die laufende Überwachung durch Betreiber vorgesehen sind, sofern keine bestimmte Protokolloberfläche verschoben wird

## Migrationsplan

### Phase 0: Grenze festschreiben

Die Grenze für dauerhaften Zustand explizit festlegen, bevor weitere Zeilen verschoben werden:

- Der globalen Datenbank eine Tabelle `migration_runs` hinzufügen.
  Für Ausführungsberichte zur Migration von Legacy-Zuständen erledigt.
- Einen einzelnen, von Doctor verwalteten Zustandsmigrationsdienst für den Import von Dateien in die Datenbank hinzufügen.
  Erledigt: `openclaw doctor --fix` verwendet die Implementierung zur Migration von Legacy-Zuständen.
- `plan` schreibgeschützt machen und `apply` eine Sicherung erstellen, importieren und überprüfen lassen
  und anschließend alte Dateien löschen oder unter Quarantäne stellen lassen.
  Erledigt: Doctor erstellt eine überprüfte Sicherung vor der Migration, übergibt den Sicherungspfad
  an `migration_runs` und verwendet die Importer-/Entfernungspfade erneut.
- Statische Verbote hinzufügen, damit neuer Laufzeitcode keine Legacy-Zustandsdateien schreiben kann,
  während Migrationscode und Tests sie weiterhin anlegen/lesen können.
  Für die derzeit migrierten Legacy-Speicher erledigt; die Schutzprüfung durchsucht außerdem verschachtelte
  Tests nach verbotenen Laufzeitverträgen für Transkript-Locators.

### Phase 1: Globale Steuerungsebene abschließen

Gemeinsamen Koordinationszustand in `state/openclaw.sqlite` halten:

- Agenten und Registry der Agentendatenbanken
- Ledger für Tasks und Task Flow
- Plugin-Zustand
- Sandbox-Container-/Browser-Registry
- Ausführungsverlauf von Cron/Planer
- Kopplung, Geräte, Push, Update-Prüfung, TUI, OpenRouter-/Modell-Caches und weiterer
  kleiner Gateway-bezogener Laufzeitzustand
- Sicherungs- und Migrationsmetadaten
- Bytes von Gateway-Medienanhängen. Für Laufzeitschreibvorgänge erledigt; direkte Dateipfade
  sind temporäre Materialisierungen zur Kompatibilität mit Channel-Sendern und für das Sandbox-
  Staging. Laufzeit-Zulassungslisten akzeptieren SQLite-Materialisierungspfade, nicht Legacy-
  Zustands-/Konfigurationsstammverzeichnisse für Medien. Doctor importiert Legacy-Mediendateien in
  `media_blobs` und entfernt die Quelldateien nach erfolgreichen Zeilenschreibvorgängen.
- Debug-Proxy-Erfassungssitzungen, -Ereignisse und -Nutzlast-Blobs. Erledigt: Erfassungen befinden sich
  in der gemeinsamen Zustandsdatenbank und werden über Bootstrap, Schema,
  WAL- und Busy-Timeout-Einstellungen der gemeinsamen Zustandsdatenbank geöffnet. Nutzlastbytes werden in
  `capture_blobs.data` mit gzip komprimiert; es gibt keine Laufzeitüberschreibung für eine Debug-Proxy-Sidecar-Datenbank,
  kein Blob-Verzeichnis und kein ausschließlich für Proxy-Erfassungen vorgesehenes generiertes Schema-/Codegen-Ziel.
  Die Doctor-/Startmigration importiert ausgelieferte `debug-proxy/capture.sqlite`-Zeilen
  und referenzierte Nutzlast-Blobs einschließlich aktiver Legacy-Umgebungsüberschreibungen für Datenbank/Blobs
  und archiviert anschließend diese Quellen, während CA-Zertifikate unverändert bleiben.

Diese Phase entfernt außerdem doppelte Sidecar-Öffnungsfunktionen, Berechtigungshelfer, die WAL-
Einrichtung, Dateisystembereinigung und Kompatibilitäts-Writer aus diesen Subsystemen.

### Phase 2: Datenbanken pro Agent einführen

Erstellen Sie eine Datenbank pro Agent und registrieren Sie sie über die globale DB:

```text
~/.openclaw/state/openclaw.sqlite
~/.openclaw/agents/<agentId>/agent/openclaw-agent.sqlite
```

Die globale `agent_databases`-Zeile speichert den Pfad, die Schemaversion, den Zeitstempel
der letzten Sichtung sowie grundlegende Metadaten zu Größe und Integrität. Der Laufzeitcode fragt die Registry nach
der Agent-DB, statt Dateipfade direkt abzuleiten.

Die Agent-DB verwaltet:

- `sessions` als kanonischen Sitzungsstamm, wobei `session_entries` als
  Payload-Tabelle in Kompatibilitätsform an diesen Stamm angehängt ist und
  `session_routes` als eindeutige aktive `session_key`-Suche dient
- `conversations` und `session_conversations` als normalisierte Provider-
  Routing-Identität, die Sitzungen zugeordnet ist
- `transcript_events`
- Transkript-Snapshots und Compaction-Checkpoints. Für Laufzeitschreibvorgänge abgeschlossen.
- `vfs_entries`
- `tool_artifacts` und Ausführungsartefakte
- agent-lokale Laufzeit-/Cache-Zeilen. Für Worker-bezogene Caches abgeschlossen.
- Ereignisse des übergeordneten ACP-Streams
- Trajektorien-Laufzeitereignisse, sofern sie keine expliziten Exportartefakte sind

### Phase 3: Sitzungsspeicher-APIs ersetzen

Für die Laufzeit abgeschlossen. Die dateiförmige Oberfläche des Sitzungsspeichers ist kein aktiver
Laufzeitvertrag:

- Die Laufzeit ruft `loadSessionStore(storePath)` nicht mehr auf und behandelt `storePath` nicht als
  Sitzungsidentität.
- Zeilenoperationen der Laufzeit sind `getSessionEntry`, `upsertSessionEntry`,
  `patchSessionEntry`, `deleteSessionEntry` und `listSessionEntries`.
- Hilfsfunktionen zum Umschreiben des gesamten Speichers, Datei-Writer, Warteschlangentests, Alias-Bereinigung und
  Parameter zum Löschen von Legacy-Schlüsseln wurden aus der Laufzeit entfernt.
- Veraltete Kompatibilitätsexporte des Root-Pakets delegieren bis zum 2026-10-12 an den ausschließlich für Doctor vorgesehenen
  `sessions.json`-Importer; Kompatibilitätslesevorgänge des Plugin SDK
  projizieren weiterhin kanonische SQLite-Zeilen.
- Das Parsen von `sessions.json` verbleibt ausschließlich im Doctor-Migrations-/Importcode und
  in Doctor-Tests.
- Der Laufzeit-Lebenszyklus-Fallback liest SQLite-Transkript-Header und nicht die ersten
  JSONL-Zeilen.

Entfernen Sie weiterhin alles, was Dateisperrparameter,
Vokabular für Bereinigung/Verkürzung als Dateiwartung, Speicherpfadidentität oder Tests wieder einführt,
deren einzige Assertion die JSON-Persistenz ist.

### Phase 4: Transkripte, ACP-Streams, Trajektorien und VFS verschieben

Gestalten Sie jeden Agent-Datenstream datenbanknativ:

- Transkript-Anfügevorgänge werden über eine einzelne SQLite-Transaktion geschrieben, die den
  Sitzungs-Header sicherstellt, die Nachrichtenidempotenz prüft, das übergeordnete Ende auswählt, Daten
  in `transcript_events` einfügt und abfragbare Identitätsmetadaten in
  `transcript_event_identities` erfasst. Für direkte Transkript-Nachrichtenanhänge und
  normale persistierte `TranscriptSessionManager`-Anhänge abgeschlossen; explizite Verzweigungs-
  operationen behalten ihre explizite Auswahl des übergeordneten Elements bei und schreiben weiterhin SQLite-Zeilen,
  ohne einen Dateilokator abzuleiten.
- Protokolle des übergeordneten ACP-Streams werden zu Zeilen statt zu `.acp-stream.jsonl`-Dateien. Abgeschlossen.
- Die Einrichtung von ACP-Spawn-Vorgängen persistiert keine Transkript-JSONL-Pfade mehr. Abgeschlossen.
- Die Laufzeit-Trajektorienerfassung schreibt Ereigniszeilen/-artefakte direkt. Der explizite
  Support-/Exportbefehl kann weiterhin Support-Bundle-JSONL-Artefakte als
  Exportformat erzeugen, aber der Sitzungsexport erstellt keine Sitzungs-JSONL-Dateien neu. Abgeschlossen.
- Datenträger-Arbeitsbereiche verbleiben auf dem Datenträger, wenn der Datenträgermodus konfiguriert ist.
- VFS-Scratch und der experimentelle reine VFS-Arbeitsbereichsmodus verwenden die Agent-DB.

Die Migration importiert alte JSONL-Dateien einmalig, zeichnet Anzahlen/Hashes in
`migration_runs` auf und entfernt importierte Dateien nach Integritätsprüfungen.

### Phase 5: Sicherung, Wiederherstellung, Vacuum und Verifizierung

Sicherungen bleiben eine einzelne Archivdatei:

- Erstellen Sie einen Checkpoint für jede globale und jede Agent-Datenbank.
- Erstellen Sie einen Snapshot jeder DB mittels SQLite-Online-Sicherung, gefolgt von einem Offline-`VACUUM`.
- Archivieren Sie kompakte DB-Snapshots, Konfiguration, externe Zugangsdaten und angeforderte
  Arbeitsbereichsexporte.
- Lassen Sie rohe Live-Dateien `*.sqlite-wal` und `*.sqlite-shm` aus.
- Verifizieren Sie, indem Sie jeden DB-Snapshot öffnen und `PRAGMA integrity_check` ausführen.
  `openclaw backup create` führt diese Archivverifizierung standardmäßig durch;
  `--no-verify` überspringt nur den Archivdurchlauf nach dem Schreiben, nicht die Integritätsprüfung
  bei der Snapshot-Erstellung.
- Bei der Wiederherstellung werden Snapshots zurück an ihre Zielpfade kopiert. Wiederhergestellte globale DBs verwenden
  Version `1`; wiederhergestellte Agent-DBs verwenden Version `2`, wobei Snapshots der Version `1`
  beim Öffnen atomar aktualisiert werden.

### Phase 6: Worker-Laufzeit

Behalten Sie den Worker-Modus im experimentellen Stadium, während die Datenbankaufteilung umgesetzt wird:

- Worker erhalten Agent-ID, Ausführungs-ID, Dateisystemmodus und DB-Registry-Identität.
- Jeder Worker öffnet seine eigene SQLite-Verbindung.
- Der übergeordnete Prozess behält die Zuständigkeit für Kanalauslieferung, Genehmigungen, Konfiguration und Abbruch.
- Beginnen Sie mit einem Worker pro aktiver Ausführung; fügen Sie Pooling erst hinzu, nachdem Lebenszyklus und
  Eigentümerschaft der DB-Verbindungen stabil sind.

### Phase 7: Die alte Welt löschen

Für die Laufzeit-Sitzungsverwaltung abgeschlossen. Die alte Welt ist nur als explizite
Doctor-Eingabe oder Support-/Exportausgabe zulässig:

- Keine Laufzeitschreibvorgänge für `sessions.json`, Transkript-JSONL, Sandbox-Registry-JSON, Task-
  Sidecar-SQLite oder Plugin-Zustands-Sidecar-SQLite.
- Keine Bereinigung von JSON-/Sitzungsdateien, Verkürzung von Transkriptdateien, Sitzungsdateisperren
  oder sperrenförmige Sitzungstests.
- Keine Laufzeit-Kompatibilitätsexporte, deren Zweck darin besteht, alte Sitzungsdateien
  aktuell zu halten.
- Explizite Supportexporte bleiben vom Benutzer angeforderte Archiv-/Materialisierungsformate
  und dürfen Dateinamen nicht zurück in die Laufzeitidentität einspeisen.

## Sicherung und Wiederherstellung

Sicherungen sollten aus einer einzelnen Archivdatei bestehen, die Datenbankerfassung sollte jedoch
SQLite-nativ sein:

1. Halten Sie Schreibtransaktionen begrenzt, damit die Online-Sicherung Fortschritte erzielen kann.
2. Verifizieren Sie vor der Erfassung jede aktive globale und jede Agent-Datenbank.
3. Erfassen Sie jede Datenbank per SQLite-Online-Sicherung in einem temporären Sicherungs-
   verzeichnis, schließen Sie anschließend die aktive Verbindung und wenden Sie `VACUUM` auf die private Kopie an.
   Plugin-Schemata, die vom Eigentümer definierte SQLite-Funktionen erfordern, schlagen sicher fehl,
   bis der Eigentümer einen sicheren Snapshot-Vertrag bereitstellt.
4. Archivieren Sie die Datenbank-Snapshots, die Konfigurationsdatei, das Verzeichnis mit Zugangsdaten, ausgewählte
   Arbeitsbereiche und ein Manifest.
5. Verifizieren Sie die Dateistruktur jedes SQLite-Snapshots, öffnen Sie anschließend kanonische OpenClaw-
   Datenbanken und führen Sie `PRAGMA integrity_check` sowie eine Rollenvalidierung aus. Dedizierte
   Plugin-Schemata bleiben undurchsichtig, sofern ihr Eigentümer keinen Verifizierer bereitstellt.
   `openclaw backup create` führt dies standardmäßig aus; `--no-verify` dient nur dazu,
   den Archivdurchlauf nach dem Schreiben absichtlich zu überspringen.

Verlassen Sie sich nicht auf rohe Live-Kopien von `*.sqlite`, `*.sqlite-wal` und `*.sqlite-shm` als
primäres Sicherungsformat. Das Archivmanifest sollte Datenbankrolle,
Agent-ID, Schemaversion, Quellpfad, Snapshot-Pfad, Bytegröße und Integritätsstatus
aufzeichnen.

Bei der Wiederherstellung sollten die globale Datenbank und die Agent-Datenbankdateien aus den
Archiv-Snapshots neu aufgebaut werden. Das globale Schema verbleibt bei Version `1`; Agent-Snapshots der Version `1`
erhalten das begrenzte Laufzeit-Upgrade auf Version `2`. Doctor bleibt
der einzige Eigentümer des Datei-zu-Datenbank-Imports. Der Wiederherstellungsbefehl validiert zunächst das
Archiv und ersetzt anschließend jedes Manifest-Asset durch die verifizierte extrahierte
Payload.

## Plan zur Laufzeitrefaktorierung

1. Fügen Sie Datenbank-Registry-APIs hinzu.
   - Lösen Sie die Pfade der globalen DB und der Agent-DBs auf.
   - Belassen Sie das globale Schema bei `user_version = 1`. Agent-DBs verwenden Version `2`
     mit einer atomaren Migration von der ausgelieferten speicherquellenbasierten Struktur der Version `1`.
   - Fügen Sie Hilfsfunktionen zum Schließen, für Checkpoints und zur Integritätsprüfung hinzu, die von Tests, Sicherung und Doctor verwendet werden.

2. Fassen Sie Sidecar-SQLite-Speicher zusammen.
   - Verschieben Sie Plugin-Zustandstabellen in die globale Datenbank. Für Laufzeit-
     schreibvorgänge abgeschlossen; der nicht ausgelieferte Legacy-Sidecar-Importer wurde gelöscht.
   - Verschieben Sie Task-Registry-Tabellen in die globale Datenbank. Für Laufzeit-
     schreibvorgänge abgeschlossen; der nicht ausgelieferte Legacy-Sidecar-Importer wurde gelöscht.
   - Verschieben Sie TaskFlow-Tabellen in die globale Datenbank. Für Laufzeitschreibvorgänge abgeschlossen;
     der nicht ausgelieferte Legacy-Sidecar-Importer wurde gelöscht.
   - Verschieben Sie integrierte Speichersuchtabellen in jede Agent-Datenbank. Abgeschlossen; ein expliziter
     benutzerdefinierter `memorySearch.store.path` wird nun durch die Doctor-Konfigurationsmigration entfernt.
     Eine vollständige Neuindizierung erfolgt direkt nur für Speichertabellen; der alte Austauschpfad für die gesamte Datei
     und die Sidecar-Index-Austauschhilfe wurden gelöscht.
   - Löschen Sie doppelte Datenbank-Öffnungsfunktionen, WAL-Einrichtung, Berechtigungshelfer und
     Schließpfade aus diesen Subsystemen.

3. Verschieben Sie agent-eigene Tabellen in Agent-Datenbanken.
   - Erstellen Sie die Agent-DB bei Bedarf über die Registry der globalen Datenbank. Abgeschlossen.
   - Verschieben Sie Laufzeit-Sitzungseinträge, Transkriptereignisse, VFS-Zeilen und Tool-
     Artefakte in Agent-DBs. Abgeschlossen.
   - Migrieren Sie keine Branch-lokalen Sitzungseinträge, Transkriptereignisse,
     VFS-Zeilen oder Tool-Artefakte aus der gemeinsam genutzten DB; dieses Layout wurde nie ausgeliefert. Behalten Sie ausschließlich den Legacy-
     Datei-zu-Datenbank-Import in Doctor bei.

4. Ersetzen Sie die Sitzungsspeicher-APIs.
   - Entfernen Sie `storePath` als Laufzeitidentität. Für die Laufzeit abgeschlossen und durch
     `check:database-first-legacy-stores` abgesichert: Sitzungsmetadaten, Routenaktualisierungen,
     Befehlspersistenz, CLI-Sitzungsbereinigung, Feishu-Reasoning-Vorschauen,
     Transkriptzustandspersistenz, Subagent-Tiefe, sitzungsbezogene Überschreibungen von Authentifizierungsprofilen,
     Parent-Fork-Logik und QA-Lab-Inspektion lösen die
     Datenbank nun anhand kanonischer Agent-/Sitzungsschlüssel auf.
     Gateway-/TUI-/UI-/macOS-Sitzungslistenantworten stellen nun `databasePath`
     anstelle des Legacy-Felds `path` bereit; macOS-Debug-Oberflächen zeigen die Agent-Datenbank
     als schreibgeschützten Zustand an, statt die `session.store`-Konfiguration zu schreiben.
     `/status`, chatgesteuerter Trajektorienexport und CLI-Abhängigkeits-Proxys
     geben keine Legacy-Speicherpfade mehr weiter; der Fallback für die Transkriptnutzung liest
     SQLite anhand der Agent-/Sitzungsidentität. Laufzeit- und Bridge-Tests legen
     `storePath` nicht mehr offen; Doctor-/Migrationseingaben besitzen diesen Legacy-Feldnamen.
     Das kombinierte Laden von Sitzungen im Gateway enthält keinen speziellen Laufzeitzweig mehr für
     nicht vorlagenbasierte `session.store`-Werte; stattdessen aggregiert es Agent-SQLite-Zeilen.
     Der Legacy-Doctor-Pfad für Sitzungssperren und seine `.jsonl.lock`-Bereinigungshilfe
     wurden entfernt; SQLite bildet nun die Nebenläufigkeitsgrenze für Sitzungen.
     Häufig aufgerufene Laufzeitstellen verwenden zeilenorientierte Hilfsfunktionsnamen wie
     `resolveSessionRowEntry`; der alte `resolveSessionStoreEntry`-Kompatibilitäts-
     alias wurde aus Laufzeit- und Plugin-SDK-Exporten entfernt.

- Verwenden Sie `{ agentId, sessionKey }`-Zeilenoperationen.
  Erledigt: `getSessionEntry`, `upsertSessionEntry`, `deleteSessionEntry`,
  `patchSessionEntry` und `listSessionEntries` sind SQLite-zentrierte APIs, die
  keinen Sitzungsspeicherpfad erfordern. Statusübersicht, lokaler Agentenstatus, Integritätsstatus
  und der Auflistungsbefehl `openclaw sessions` lesen jetzt agentenspezifische Zeilen direkt
  und zeigen anstelle von `sessions.json`-Pfaden agentenspezifische SQLite-Datenbankpfade an.
- Ersetzen Sie das Löschen/Einfügen des gesamten Speichers durch `upsertSessionEntry`,
  `deleteSessionEntry`, `listSessionEntries` und SQL-Bereinigungsabfragen.
  Für die Laufzeit erledigt: Stark frequentierte Pfade verwenden jetzt Zeilen-APIs und bei Konflikten
  wiederholte Zeilen-Patches; die verbleibenden Hilfsfunktionen zum Importieren/Ersetzen des gesamten
  Speichers sind auf Migrationsimportcode und Tests des SQLite-Backends beschränkt.
  - Löschen Sie `store-writer.ts` und die Tests der Schreibwarteschlange. Erledigt.
  - Entfernen Sie die laufzeitseitige Bereinigung veralteter Schlüssel und Alias-Löschparameter aus
    Upserts/Patches von Sitzungszeilen. Erledigt.

5. Entfernen Sie das laufzeitseitige Verhalten der JSON-Registry.
   - Stellen Sie Lese- und Schreibvorgänge der Sandbox-Registry vollständig auf SQLite um. Erledigt.
   - Importieren Sie monolithisches und fragmentiertes JSON nur im Migrationsschritt. Erledigt.
   - Entfernen Sie Sperren der fragmentierten Registry und JSON-Schreibvorgänge. Erledigt.

- Verwenden Sie eine einzige typisierte Registry-Tabelle, statt Registry-Zeilen als generisches
  undurchsichtiges JSON zu speichern, wenn die Struktur weiterhin operativer Zustand eines stark frequentierten Pfads ist. Erledigt.

6. Entfernen Sie die dateisperrenbasierte Sitzungsmutation.
   - Für die Erstellung von Laufzeitsperren und Laufzeitsperr-APIs erledigt.
   - Der eigenständige veraltete Doctor-Bereinigungspfad `.jsonl.lock` wurde entfernt.
   - Die Zustandsintegrität verfügt nicht mehr über einen separaten Bereinigungspfad für verwaiste
     Transkriptdateien; die Doctor-Migration importiert/entfernt veraltete JSONL-Quellen zentral.
   - Die Gateway-Singleton-Koordination verwendet typisierte SQLite-Zeilen `state_leases` unter
     `gateway_locks` und stellt keine Schnittstelle für ein Dateisperrverzeichnis mehr bereit.
   - Die generische Deduplizierungspersistenz des Plugin-SDK verwendet keine Dateisperren oder JSON-
     Dateien mehr; sie schreibt Zeilen für den gemeinsam genutzten SQLite-Plugin-Zustand. Erledigt.
   - Die QMD-Koordination verwendet eine gemeinsam genutzte SQLite-Lease für Einbettungen und eine agentenspezifische
     SQLite-Lease für jeden Writer für Sammlungen/Aktualisierungen/Einbettungen. Die Laufzeit erstellt
     `qmd/embed.lock.lock` oder `agents/<agentId>/qmd-write.lock.lock` nicht mehr;
     Doctor entfernt nur eindeutig veraltete, außer Betrieb genommene Sidecars. Erledigt.

7. Machen Sie Worker datenbankfähig.
   - Worker öffnen eigene SQLite-Verbindungen.
   - Der übergeordnete Prozess verwaltet Zustellung, Kanal-Callbacks und Konfiguration.
   - Der Worker erhält Agenten-ID, Ausführungs-ID, Dateisystemmodus und DB-Registry-
     Identität, keine aktiven Handles.
   - `vfs-only` bleibt experimentell und verwendet die Agentendatenbank als Speicher-
     Stammverzeichnis.
   - Verwenden Sie zunächst einen Worker pro aktiver Ausführung. Pooling kann warten, bis Lebensdauer
     und Abbruchverhalten der DB-Verbindung zuverlässig und unspektakulär sind.

8. Backup-Integration.
   - Erweitern Sie Backups um Snapshots globaler, agentenspezifischer und Plugin-Datenbanken mit einem Online-
     Backup, gefolgt von einem Offline-`VACUUM`. Für erkannte `*.sqlite`-Dateien unter dem Zustands-Asset erledigt;
     Plugin-Schemas, die nicht verfügbare Eigentümerfunktionen erfordern, schlagen sicher geschlossen fehl.
   - Fügen Sie eine Backup-Verifizierung für die kanonische SQLite-Integrität und Schemaidentität
     sowie eine generische Dateiformatvalidierung für dedizierte Plugin-Snapshots hinzu. Für
     die Backup-Erstellung und die standardmäßige Archivverifizierung erledigt.
   - Zeichnen Sie Metadaten zu Backup-Ausführungen in SQLite auf. Erledigt über die gemeinsam genutzte Tabelle `backup_runs`
     mit Archivpfad, Status und Manifest-JSON.
   - Fügen Sie die Wiederherstellung aus verifizierten Archiv-Snapshots hinzu. Erledigt: `openclaw backup
restore` validiert vor der Extraktion, verwendet das normalisierte
     Manifest des Verifizierers, unterstützt `--dry-run` und erfordert `--yes`, bevor
     aufgezeichnete Quellpfade ersetzt werden.
   - Schließen Sie den VFS-/Arbeitsbereichsexport nur auf Anforderung ein; exportieren Sie keine Sitzungs-
     interna als JSON oder JSONL.

9. Entfernen Sie veraltete Tests und veralteten Code. Für die bekannten Laufzeit-Sitzungsoberflächen erledigt.

- Entfernen Sie Tests, die die laufzeitseitige Erstellung von `sessions.json`- oder Transkript-
  JSONL-Dateien voraussetzen. Erledigt für den zentralen Sitzungsspeicher, Chat, Gateway-Transkriptereignisse,
  Vorschau, Lebenszyklus, Aktualisierungen von Befehlssitzungseinträgen, Auto-Reply-Zurücksetzung/-Ablaufverfolgung und
  Dreaming-Fixtures von memory-core, Zielweiterleitung für Genehmigungen, Reparatur von Sitzungstranskripten,
  Reparatur von Sicherheitsberechtigungen, Trajektorienexport und Sitzungsexport.
  Active-Memory-Transkripttests prüfen jetzt SQLite-Geltungsbereiche und stellen sicher, dass keine temporären oder
  persistenten JSONL-Dateien erstellt werden.
  Die alte Heartbeat-Regression zur Transkriptbereinigung wurde entfernt, da
  die Laufzeit JSONL-Transkripte nicht mehr kürzt.
  Tests des Werkzeugs zur Auflistung von Agentensitzungen modellieren veraltete `sessions.json`-Pfade
  nicht mehr als Form der Gateway-Antwort; App-/UI-/macOS-Tests verwenden `databasePath`.
  Transkriptnutzungstests für `/status` befüllen SQLite-Transkriptzeilen jetzt direkt,
  statt JSONL-Dateien zu schreiben.
  Tests des Gateway-Sitzungslebenszyklus verwenden jetzt direkt Hilfsfunktionen zum Befüllen von SQLite-Transkripten;
  die alte Fixture-Struktur einer einzeiligen Sitzungsdatei wurde aus der Abdeckung
  für Zurücksetzen und Löschen entfernt.
  `sessions.delete` gibt kein aus der Dateiära stammendes Feld `archived: []` mehr zurück; Löschvorgänge
  melden nur das Ergebnis der Zeilenmutation. Auch die alte Option `deleteTranscript` wurde
  entfernt: Beim Löschen einer Sitzung wird das kanonische Stammverzeichnis `sessions` entfernt und
  SQLite löscht sitzungseigene Transkript-, Snapshot- und Trajektorienzeilen kaskadierend, sodass kein
  Aufrufer verwaiste Transkripte hinterlassen oder einen Bereinigungszweig vergessen kann.
  Tests zur Trajektorienerfassung der Kontext-Engine lesen jetzt `trajectory_runtime_events`-
  Zeilen aus einer isolierten Agentendatenbank, statt
  `session.trajectory.jsonl` zu lesen.
  Docker-MCP-Kanal-Seeding-Skripte befüllen SQLite-Zeilen jetzt direkt. Direkte
  Schreibvorgänge in `sessions.json` sind auf Doctor-Fixtures beschränkt.
  Das Tool-Search-Gateway-E2E liest Nachweise für Werkzeugaufrufe aus SQLite-Transkriptzeilen,
  statt `agents/<agentId>/sessions/*.jsonl`-Dateien zu durchsuchen.
  Host-Ereignisse von memory-core und Scratch-Zeilen des Sitzungskorpus befinden sich jetzt im gemeinsam genutzten
  SQLite-Plugin-Zustand; `events.jsonl` und `session-corpus/*.txt` sind nur noch Eingaben
  für veraltete Doctor-Migrationen. Aktive Zeilen verwenden virtuelle `memory/session-ingestion/`-
  Pfade, nicht `.dreams/session-corpus`. Das alte Dreaming-
  Reparaturmodul von memory-core und seine CLI-/Gateway-Tests wurden entfernt, da die Laufzeit
  die Reparatur von Dateiarchiven für diesen Korpus nicht mehr verwaltet. Tests für
  Bridges/öffentliche Artefakte von memory-core stellen `.dreams/events.jsonl` nicht mehr bereit; sie
  verwenden den SQLite-gestützten virtuellen JSON-Artefaktnamen.
  Öffentliche SDK-/Codex-Testdokumentation spricht jetzt von SQLite-Sitzungszustand statt Sitzungs-
  dateien, und das Beispiel für einen Kanaldurchlauf stellt kein Argument `storePath` mehr bereit.
  Der Matrix-Synchronisierungszustand verwendet jetzt direkt den SQLite-Plugin-Zustandsspeicher. Aktive
  Client-/Laufzeitverträge übergeben ein Speicherstammverzeichnis des Kontos, keinen `bot-storage.json`-
  Pfad, und Doctor importiert das veraltete `bot-storage.json` in SQLite, bevor
  die Quelle gelöscht wird. Neustart-/destruktive Matrix-Szenarien von QA Lab mutieren jetzt direkt die SQLite-Synchronisierungs-
  zeile, statt fingierte `bot-storage.json`-Dateien zu erstellen oder zu löschen, und
  die E2EE-Grundlage übergibt ein Stammverzeichnis für den Synchronisierungsspeicher statt eines fingierten
  `sync-store.json`-Pfads.
  Die Auswahl des Matrix-Speicherstammverzeichnisses bewertet Stammverzeichnisse nicht mehr anhand veralteter Synchronisierungs-/Thread-JSON-
  Dateien, sondern verwendet dauerhafte Stammverzeichnis-Metadaten sowie echten Kryptografiezustand.
  Die Testsuite für das laufzeitseitige SQLite-Sitzungs-Backend erzeugt kein
  `sessions.json` mehr; Fixtures für veraltete Quellen befinden sich jetzt in den Doctor-
  Tests, die sie importieren.
  Gateway-Sitzungstests stellen keine Hilfsfunktion `createSessionStoreDir` oder
  ungenutzte Einrichtung eines temporären Sitzungsspeicherpfads mehr bereit; Fixture-Verzeichnisse sind explizit, und die direkte
  Zeileneinrichtung verwendet die Benennung von SQLite-Sitzungszeilen.
  Die ausschließlich für Doctor bestimmte Abdeckung des JSON5-Sitzungsspeicher-Parsers wurde aus Infrastrukturtests
  in Doctor-Migrationstests verschoben, sodass Laufzeittestsuites nicht mehr für das Parsen veralteter
  Sitzungsdateien zuständig sind.
  Microsoft-Teams-Laufzeittests für SSO/ausstehende Uploads verwenden keine JSON-Sidecar-
  Fixtures oder Parser mehr; die Analyse veralteter SSO-Token erfolgt nur noch im Plugin-
  Migrationsmodul. Telegram-Tests befüllen keine fingierten `/tmp/*.json`-Speicher-
  pfade mehr; sie setzen den SQLite-gestützten Nachrichtencache direkt zurück. Die generische
  OpenClaw-Testzustands-Hilfsfunktion stellt keinen veralteten `auth-profiles.json`-
  Writer mehr bereit; Doctor-Authentifizierungsmigrationstests verwalten diese Fixture lokal.
  Laufzeittests für Zeiger auf die letzte TUI-Sitzung, Ausführungsgenehmigungen, Active-Memory-
  Umschalter, Matrix-Deduplizierung/Startverifizierung, Memory-Wiki-Quellsynchronisierung,
  Bindungen der aktuellen Konversation, Onboarding-Authentifizierung und Hermes-Secret-Importe
  erstellen keine alten Sidecar-Dateien mehr und prüfen nicht mehr, ob alte Dateinamen fehlen. Sie
  weisen das Verhalten über SQLite-Zeilen und öffentliche Speicher-APIs nach; Doctor-/Migrations-
  tests sind der einzige Ort, an den veraltete Quelldateinamen gehören.
  Laufzeittests für Geräte-/Node-Kopplung, kanalbezogenes allowFrom, Neustartabsichten,
  Neustartübergabe, Einträge in der Sitzungszustellungswarteschlange, Konfigurationsintegrität, iMessage-
  Caches, Cron-Aufträge, PI-Transkriptkopfzeilen, Subagenten-Registries und verwaltete
  Bildanhänge erstellen ebenfalls keine außer Betrieb genommenen JSON-/JSONL-Dateien mehr, nur um
  nachzuweisen, dass sie ignoriert werden oder nicht vorhanden sind.
  Die PI-Überlaufwiederherstellung verfügt nicht mehr über einen SessionManager-Fallback zum Umschreiben/Kürzen:
  Die Kürzung von Werkzeugergebnissen und Transkriptumschreibungen der Kontext-Engine mutieren
  SQLite-Transkriptzeilen und aktualisieren anschließend den aktiven Prompt-Zustand aus der Datenbank.
  Persistierte SessionManager-Nachrichtenanhänge delegieren die Auswahl des übergeordneten Elements und
  die Idempotenz an die atomare Hilfsfunktion zum Anhängen von SQLite-Transkripten. Auch normale
  Anhänge von Metadaten/benutzerdefinierten Einträgen wählen das aktuelle übergeordnete Element innerhalb von SQLite aus, sodass
  veraltete Manager-Instanzen keine Konkurrenzsituationen in der übergeordneten Kette aus der Zeit vor SQLite wiederaufleben lassen.
  Die synthetische PI-Endbereinigung für Prüfungen während eines Durchlaufs und `sessions_yield` kürzt jetzt
  den SQLite-Transkriptzustand direkt; die alte SessionManager-Bridge zur Entfernung des Endes
  und ihre Tests wurden gelöscht.
  Auch die Erfassung von Compaction-Prüfpunkten erstellt Snapshots ausschließlich aus SQLite; Aufrufer
  übergeben keinen aktiven SessionManager mehr als alternative Transkriptquelle.
- Behalten Sie Tests, die veraltete Dateien befüllen, ausschließlich für Migrationen bei.
- Der Nachweis über JSON-Dateien wurde für aktive Laufzeitoberflächen durch
  den Nachweis über SQL-Zeilen ersetzt.

- Fügen Sie statische Verbote für Laufzeitschreibvorgänge in veraltete JSON-Pfade für Sitzungen/Caches hinzu.
  Für die Repository-Prüfung erledigt.

10. Machen Sie den Migrationsbericht auditierbar.
    - Zeichnen Sie Migrationsausführungen in SQLite mit Start-/Endzeitstempeln, Quell-
      pfaden, Quell-Hashes, Anzahlen, Warnungen und Backup-Pfad auf.
      Erledigt: Ausführungen der Migration veralteter Zustände persistieren jetzt einen `migration_runs`-
      Bericht mit Inventar der Quellpfade/-tabellen, SHA-256 der Quelldatei, Größen,
      Datensatzanzahlen, Warnungen und Backup-Pfad.
      Erledigt: Ausführungen der Migration veralteter Zustände persistieren außerdem `migration_sources`-
      Zeilen für quellbezogene Audits und künftige Entscheidungen zum Überspringen/Nachfüllen.
    - Machen Sie die Anwendung idempotent. Eine erneute Ausführung nach einem Teilimport sollte eine
      bereits importierte Quelle entweder überspringen oder anhand eines stabilen Schlüssels zusammenführen.
      Erledigt: Sitzungsindizes, Transkripte, Zustellungswarteschlangen, Plugin-Zustand, Aufgaben-
      ledger und agenteneigene globale SQLite-Zeilen werden über stabile Schlüssel oder
      Upsert-/Ersetzungssemantik importiert, sodass erneute Ausführungen zusammenführen, ohne dauerhafte
      Zeilen zu duplizieren.
    - Fehlgeschlagene Importe müssen die ursprüngliche Quelldatei an ihrem Speicherort belassen.
      Erledigt: Fehlgeschlagene Transkriptimporte belassen die ursprüngliche JSONL-Quelle jetzt
      an ihrem erkannten Pfad, und `migration_sources` zeichnet die Quelle als
      `warning` mit `removed_source=0` für die nächste Doctor-Ausführung auf.

## Leistungsregeln

- Eine Verbindung pro Thread/Prozess ist in Ordnung; verwenden Sie Handles nicht gemeinsam über
  Worker hinweg.
- Verwenden Sie WAL, `foreign_keys=ON`, ein Busy-Timeout von 5s und kurze `BEGIN IMMEDIATE`
  Schreibtransaktionen. Legen Sie keine synchronen Wiederholungsversuche für Sperren über
  die einmalige Busy-Wartezeit von SQLite.
- Halten Sie Hilfsfunktionen für Schreibtransaktionen synchron, solange keine asynchrone Transaktions-API
  explizite Mutex-/Backpressure-Semantik bereitstellt.
- Halten Sie Schreibvorgänge für die übergeordnete Zustellung klein und transaktional.
- Vermeiden Sie das Neuschreiben des gesamten Speichers; verwenden Sie Upsert/Löschen auf Zeilenebene.
- Fügen Sie Indizes für Auflistungen nach Agent, Auflistungen nach Sitzung, Aktualisierungszeitpunkt, Ausführungs-ID und
  Ablaufpfade hinzu, bevor Sie häufig ausgeführten Code verschieben.
- Speichern Sie große Artefakte, Medien und Vektoren als BLOBs oder aufgeteilte BLOB-Zeilen, nicht
  als base64 oder JSON mit numerischen Arrays.
- Halten Sie undurchsichtige Plugin-Zustandseinträge klein und klar abgegrenzt.
- Fügen Sie eine SQL-Bereinigung für TTL/Ablauf hinzu, statt das Dateisystem zu bereinigen.
  Für datenbankeigene Laufzeitspeicher abgeschlossen: Medien, Plugin-Zustand, Plugin-BLOBs,
  persistente Deduplizierung und Agent-Cache laufen sämtlich über SQLite-Zeilen ab. Die verbleibende
  Dateisystembereinigung ist auf temporäre Materialisierungen oder explizite
  Entfernungsbefehle beschränkt.

## Statische Verbote

Fügen Sie eine Repository-Prüfung hinzu, bei der neue Laufzeitschreibvorgänge in veraltete Zustandspfade fehlschlagen:

- `sessions.json`
- `*.trajectory.jsonl` außer materialisierten Ausgaben von Support-Bundles
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
- benachbartes `<workspace>.attested`
- Matrix `credentials*.json` und `recovery-key.json`
- `cron/runs/*.jsonl`
- `cron/jobs.json`
- `jobs-state.json`
- `device-pair-notify.json`
- `devices/pending.json` / `devices/paired.json` / `devices/bootstrap.json`
  (seit 2026.7 eingestellt: Laufzeitspeicher ist `device_pairing_*` /
  `device_bootstrap_tokens` in der gemeinsamen Zustandsdatenbank; gekoppelte Datensätze werden beim
  Gateway-Start importiert, temporäre ausstehende/Bootstrap-Zeilen werden verworfen)
- `nodes/pending.json` / `nodes/paired.json` (seit 2026.7 eingestellt: beim Gateway-Start in gekoppelte Gerätedatensätze integriert)
- `identity/device.json`
- `identity/device-auth.json` (eingestellt; Import ausschließlich durch Doctor in `device_auth_tokens`)
- `push/web-push-subscriptions.json` (eingestellt; Import ausschließlich durch Doctor in `web_push_subscriptions`)
- `push/vapid-keys.json` (eingestellt; Import ausschließlich durch Doctor in `web_push_vapid_keys`)
- `push/apns-registrations.json` (eingestellt; Import ausschließlich durch Doctor in `apns_registrations`)
- `process-leases.json`
- `gateway-instance-id`
- `session-toggles.json`
- Memory-Core `.dreams/events.jsonl`
- Memory-Core `.dreams/session-corpus/`
- Memory-Core `.dreams/daily-ingestion.json`
- Memory-Core `.dreams/session-ingestion.json`
- Memory-Core `.dreams/short-term-recall.json`
- Memory-Core `.dreams/phase-signals.json`
- Memory-Core `.dreams/short-term-promotion.lock`
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
- JSON-Dateien für Sandbox-Registry-Shards
- `plugin-state/state.sqlite`
- Ad-hoc-`openclaw-state.sqlite`-Laufzeit-Sidecars
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
  Fassaden für Transkriptauflistungen
- `SessionManager.forkFromSession(...)` und
  `TranscriptSessionManager.forkFromSession(...)` Fassaden zum Verzweigen von Transkripten
- `SessionManager.newSession(...)` und `TranscriptSessionManager.newSession(...)`
  Fassaden zum veränderlichen Ersetzen von Sitzungen
- `SessionManager.createBranchedSession(...)` und
  `TranscriptSessionManager.createBranchedSession(...)` Fassaden für Zweigsitzungen

Das Verbot sollte Tests das Erstellen veralteter Fixtures und Migrationscode das
Lesen/Importieren/Entfernen veralteter Dateiquellen erlauben. Nicht veröffentlichte SQLite-Sidecars bleiben verboten
und erhalten keine Doctor-Importausnahmen.

## Abschlusskriterien

- Laufzeitdaten und Cache-Schreibvorgänge erfolgen in die globale oder die Agent-SQLite-Datenbank.
- Die Laufzeit schreibt keine Sitzungsindizes, Transkript-JSONL, Sandbox-Registry-
  JSON, Aufgaben-Sidecar-SQLite oder Plugin-Zustands-Sidecar-SQLite mehr. Die nicht veröffentlichten Importer für Aufgaben-
  und Plugin-Zustands-Sidecar-SQLite werden gelöscht.
- Der Import veralteter Dateien erfolgt ausschließlich durch Doctor.
- Die Sicherung erzeugt ein Archiv mit kompakten SQLite-Snapshots und Integritätsnachweis.
- Agent-Worker können mit Datenträger-, VFS-Scratch- oder experimentellem reinem VFS-
  Speicher ausgeführt werden.
- Konfigurations- und explizite Anmeldedatendateien bleiben die einzigen erwarteten persistenten
  Steuerdateien außerhalb der Datenbank.
- Repository-Prüfungen verhindern die Wiedereinführung veralteter Laufzeit-Dateispeicher.
