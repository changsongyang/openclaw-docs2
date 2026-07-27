---
read_when:
    - Sie weisen die Umstellung des SQLite-Speichers für Pfad 3 anhand eines aktiven Gateways nach.
    - Sie müssen erwartete Abweichungen in Legacy-JSONL von Laufzeitfehlern unterscheiden
    - Sie erstellen oder überprüfen das agentengesteuerte Live-SQLite-E2E-Testsystem
summary: Konzept für den Live-Gateway-Nachweis der Umstellung von Sitzungen/Transkripten auf SQLite in Pfad 3
title: 'Pfad 3: Live-SQLite-E2E-Testsystem'
x-i18n:
    generated_at: "2026-07-26T18:45:31Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 2749bf47cb4967bc80a5ed37a12f2a553f3b388ed8cd90cfb3217e1b5e8afae9
    source_path: reference/path3-live-sqlite-e2e-harness.md
    workflow: 16
---

Der Path-3-Live-SQLite-E2E-Harness weist nach, dass das Gateway SQLite als
kanonischen Sitzungs- und Transkriptspeicher verwendet, während ältere JSONL-Dateien
Migrationseingaben oder Archivmaterial bleiben. Es handelt sich um einen Nachweis-Harness für Maintainer, nicht um ein
normales Benutzerdiagnosewerkzeug.

Nachdem ein Gateway Datenverkehr nach der Migration verarbeitet hat, ist die Parität mit älteren JSONL-Dateien
kein gültiges Signal für den Zustand der Laufzeit mehr. Bei einem fehlerfrei migrierten Gateway können
sich die SQLite-Transkriptzeilen von der Anzahl in älteren JSONL-Dateien unterscheiden, da neue Interaktionen
nur SQLite fortschreiben sollten. Der Live-Harness muss daher bei jedem
Schritt das Gateway-Verhalten, die Veränderung der SQLite-Zeilen, den Ruhezustand der älteren Dateien und den Protokollzustand messen.

## Befehlsform

Der vorgesehene Live-Befehl lautet:

```bash
node scripts/path3-live-sqlite-e2e.mjs \
  --url http://127.0.0.1:18789 \
  --agent main \
  --session-key agent:main:path3-live-e2e:<timestamp> \
  --json
```

Der Befehl stellt eine Verbindung zu einem bereits laufenden Gateway her. Er startet oder stoppt
die Migration nicht, importiert sie nicht und führt sie auch nicht erneut aus, sofern später nicht
ein expliziter Migrationsmodus hinzugefügt wird. Eine CI- oder isolierte lokale Variante kann
`test/helpers/openclaw-test-instance.ts` verwenden, aber der Live-Nachweispfad sollte
das tatsächliche Betreiber-Gateway und dessen echte agentenspezifische SQLite-Datenbank untersuchen.

## Isolierter Nachweis mit der gebauten CLI

Der Nachweis-Runner für die gebaute CLI legt einen isolierten älteren Sitzungsspeicher an, startet das
neu gebaute Gateway und weist nach, dass der Start aktive ältere Sitzungen in
SQLite importiert, bevor die Laufzeit mit Lesezugriffen beginnt. Er darf `openclaw doctor --fix`
nicht vor dem ersten Gateway-Start ausführen, da dies den manuellen Migrationspfad
anstelle des Upgrade-Pfads nachweisen würde, den Benutzer beim ersten Start nach der Umstellung erhalten.

Nach dem Startimport darf der isolierte Nachweis
`openclaw doctor --session-sqlite inspect` und
`openclaw doctor --session-sqlite validate` als Diagnosebelege ausführen. Diese
doctor-Befehle sind nicht der Migrationstreiber für den Nachweis des Start-Upgrades.
Separate doctor-Importszenarien sollten ältere Transkriptdateien samt
Trajektorien-Sidecars anlegen und überprüfen, dass doctor diese Artefakte archiviert, während SQLite
kanonisch bleibt.

## Vorabprüfung

Die Vorabprüfung erfasst einen Ausgangszustand und schlägt vor dem Senden einer Nachweisinteraktion fehl, wenn das
Gateway nicht verwendbar ist:

- `GET /health` und der detaillierte Gateway-Status müssen ein laufendes, erreichbares
  Gateway melden.
- Die Versionen von CLI und Gateway müssen dem getesteten Branch entsprechen.
- Der Harness zeichnet einen Protokollcursor für das aktive Gateway-Dateiprotokoll auf.
- Der Harness zeichnet die agentenspezifischen SQLite-Tabellenanzahlen für `sessions`,
  `session_entries`, `transcript_events`, `transcript_event_identities` und
  `session_routes` auf.
- Der Harness zeichnet `mtime`, `size` und die Existenz älterer
  `sessions.json`-Dateien, referenzierter JSONL-Dateien sowie möglicher JSONL-Pfade
  für Nachweissitzungen auf.
- `lsof -p <gateway-pid>` muss SQLite-DB-/WAL-/SHM-Handles und keine aktiven
  `.jsonl`- oder `sessions.json`-Handles anzeigen.

`openclaw doctor --session-sqlite validate` dient im Live-Modus nur zur Information.
Nach Datenverkehr im Anschluss an die Umstellung kann es erwartete Abweichungen gegenüber älteren Dateien melden. Der
Harness sollte die doctor-Ausgabe zur Klassifizierung und für das Migrationsinventar verwenden,
nicht als Laufzeitinstanz für die Entscheidung über Bestehen oder Fehlschlagen.

## Agentengesteuertes Szenario

Das Live-Szenario verwendet einen dedizierten Sitzungsschlüssel für den Nachweis und steuert das Gateway,
wo immer möglich, über öffentliche RPC-Pfade. Eine Agenteninteraktion sollte ausreichen, um
die gewöhnliche Persistenz abzudecken, aber der vollständige Nachweis sollte die 3.1b-Schnittstellen abdecken,
die zuvor einzelne Live-Prüfungen erforderten:

- Gewöhnliche Chat-Interaktion: Erstellen oder Wiederverwenden der Nachweissitzung, Senden einer echten Agenten-
  Eingabeaufforderung, Warten auf das endgültige Assistentenergebnis und Überprüfen von `chat.history` oder
  einer gleichwertigen Gateway-Projektion.
- Transkriptidentität: Überprüfen, dass dieselbe Markierung im Gateway-Verlauf und in den
  SQLite-Transkriptzeilen erscheint, einschließlich Zeilen mit stabiler Ereignisidentität, sofern vorhanden.
- Zugriffsfunktionen für Sitzungsmetadaten: Lesen der Nachweissitzung und ausgewählter vorhandener Live-
  Sitzungen über Gateway-/Sitzungszugriffsfunktionen und Vergleichen mit den SQLite-Zeilen.
- Projektion eines Sitzungs-Patches: Anwenden einer umkehrbaren Änderung an Modell-/Sitzungsmetadaten auf
  die Nachweissitzung und anschließendes Überprüfen, dass die projizierte Zeile und die Gateway-Antwort übereinstimmen.
- Lebenszyklus eines Compaction-Prüfpunkts: Auflisten, Verzweigen und Wiederherstellen eines Prüfpunkts ausschließlich
  für die Nachweissitzung oder eine vom Harness erstellte synthetische Testsitzung.
- Wiederherstellung nach Neustart: Ausführen des sicheren Wiederherstellungspfads mit Markierung für eine kontrollierte Nachweis-
  sitzung oder eine isolierte Testinstanz; im Live-Modus darf dieser Schritt nur ausgeführt werden, wenn
  die Zielsitzungsmenge explizit angegeben und die Änderung umkehrbar ist.
- Bereinigungslebenszyklus: Löschen oder Zurücksetzen der Nachweissitzung und anschließendes Überprüfen der SQLite-
  Lebenszykluszeilen sowie des archivierten Transkriptzustands.

Transportspezifische Schnittstellen, die sich auf dem Live-Gateway des Betreibers nicht sicher ausführen lassen,
etwa der Eingang über WhatsApp oder Sprachanrufe, sollten Laufzeitprüfungen auf Besitzerebene
anhand desselben SQLite-Vertrags verwenden, statt einen externen Transport vorzutäuschen.

## Assertions pro Schritt

Jeder Schritt erstellt Momentaufnahmen des Zustands davor und danach und schreibt einen strukturierten Assertions-
datensatz:

- Die Anzahl der SQLite-Zeilen erhöht sich nur an den erwarteten Stellen.
- Trajektorien-Laufzeitzeilen werden für markierungsgestützte Nachweissitzungen fortgeschrieben, die
  Laufzeitereignisse aufzeichnen.
- Die Zeile der Nachweissitzung enthält die erwarteten Werte für `session_id`, Status, Zeitstempel,
  Metadaten und Routingzeilen.
- Die Gateway-Verlaufs-/Sitzungsprojektion entspricht dem Ende des SQLite-Transkripts.
- Es wird keine JSONL-Datei für die Nachweissitzung erstellt oder geändert.
- Es wird kein(e) `.trajectory.jsonl`, `.trajectory-path.json` oder
  aus der Markierung abgeleitete(s) `trajectory/<session>.jsonl`-Sidecar für die Nachweissitzung erstellt.
- Vorhandene ältere JSONL-Dateien und `sessions.json` bleiben unverändert, sofern es sich bei dem
  Schritt nicht ausdrücklich um eine Offline-Migration oder einen Archivierungsvorgang handelt.
- Der Gateway-Prozess öffnet keine `.jsonl`- oder `sessions.json`-Handles.
- Die Protokolle seit dem vorherigen Cursor enthalten weder `ERROR`, `FATAL`, `SQLITE_`
  noch `no such column`, „Sitzungsspeicher nicht verfügbar“, einen Fehler bei der Wiederherstellung nach einem Neustart oder
  eine Warnung beim Transkriptabgleich, sofern das Szenario dies nicht ausdrücklich in der Zulassungsliste erlaubt.

Die Protokollprüfung ist Teil des Vertrags für Bestehen oder Fehlschlagen. Ein Gateway, das Zustandsprüfungen
beantwortet, aber SQLite-Schemafehler oder wiederholte Fehler beim Transkriptabgleich ausgibt,
gilt für Path 3 nicht als fehlerfrei.

## Belegartefakt

Der Harness sollte Belege unter `.artifacts/path3-live-e2e/<timestamp>/`
schreiben und sie nicht in Git aufnehmen:

- `summary.json`: Befehlsargumente, Gateway-Version, Ergebnis, fehlgeschlagene Assertion und
  Artefaktpfade.
- `sqlite-before.json` und `sqlite-after.json`: Zeilenanzahlen und ausgewählte Nachweis-
  zeilen.
- `legacy-files.json`: Existenz älterer Dateien, `mtime`, Größe und Angabe, ob die jeweilige
  Datei geändert wurde.
- `gateway-log-scan.json`: Cursorbereich, übereinstimmende Protokollzeilen und Entscheidungen der Zulassungsliste.
- `events.jsonl`: geordnete Beobachtungen pro Schritt, die sich für PR-Nachweiskommentare eignen.

Der PR-Nachweis sollte diese Artefakte zusammenfassen, statt vollständige
Transkripte oder private Nachrichteninhalte einzufügen.

## Sicherheitsregeln

- Der Live-Modus darf ältere JSONL-Dateien niemals erneut importieren, während das Gateway ausgeführt wird.
- Der Live-Modus darf Sitzungen, die keine Nachweissitzungen sind, nicht verändern, außer für ausdrücklich ausgewählte,
  umkehrbare Reparaturprüfungen.
- Jeder destruktive oder umfassende Migrationsschritt erfordert eine aktuelle Sicherung der
  betroffenen SQLite-Datenbank und des älteren Sitzungsverzeichnisses.
- Sicherungen sollten auf die betroffene Agenten-Datenbank bzw. das betroffene Sitzungsverzeichnis beschränkt und
  während eines Nachweislaufs wiederverwendet werden, um unbegrenztes Wachstum des Speicherplatzbedarfs zu vermeiden.
- Der Bereinigungsschritt darf keine Nachweissitzung, Nachweis-JSONL-Datei oder geänderte ältere
  Datei zurücklassen, sofern der Aufrufer nicht `--keep-artifacts` übergibt.

## Bestandenes Ergebnis

Ein bestandener Live-Lauf bedeutet, dass das Gateway einen echten agentengesteuerten Sitzungsablauf akzeptiert hat,
sich der gesamte beobachtete kanonische Zustand in SQLite befand, ältere Laufzeitdateien
im Ruhezustand blieben und der Protokollzustand im gemessenen Zeitfenster fehlerfrei blieb. Er bedeutet nicht,
dass die Parität mit älteren JSONL-Dateien nach Live-Datenverkehr weiterhin gegeben ist; Abweichungen im Live-Betrieb werden erwartet,
sobald SQLite der kanonische Speicher ist.
