---
read_when:
    - Sie implementieren clawdbot-d63.2 / clawdbot-04b
    - Sie bearbeiten die SQLite-Sitzungsaufbewahrung, das Zurücksetzen, Löschen oder die Archivierung bei der Agentenlöschung
    - Sie müssen Artefaktfamilien aus der SQLite-Ära von veralteten JSONL-Sidecars unterscheiden
summary: Plan für Pfad 3 zur Archivierung aller SQLite-Transkriptartefakte, die zu einer Sitzung gehören
title: 'Pfad 3: Familie der SQLite-Sitzungsartefakte'
x-i18n:
    generated_at: "2026-07-26T17:55:06Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 29f4d541b2df5a06468fd0cee620b4340ee33eea1064f7d3ee823580c7b5760e
    source_path: plan/path3-sqlite-session-artifact-family.md
    workflow: 16
---

# Pfad 3: SQLite-Sitzungsartefaktfamilie

Dieser Hinweis grenzt `clawdbot-d63.2` ab, während `clawdbot-d63.1` für den überlappenden
Archivierungshelfer zum Zurücksetzen/Löschen in `src/config/sessions/session-accessor.sqlite.ts` zuständig ist.
Die Implementierungsdatei enthielt während dieses Durchlaufs nicht übertragene Änderungen, daher hält dieses Artefakt
den genauen Vertrag und die Änderungspunkte fest, ohne mit dem parallel arbeitenden Worker in Konflikt zu geraten.

## Maßgebliche Familie

Nach der Umstellung auf SQLite sind aktive Sitzungstranskripte SQLite-Zeilen. Die
Archivfamilie einer Sitzung umfasst:

- Die Zeilen `transcript_events`, `transcript_event_identities` und `sessions`
  für die aktuelle `sessionId` des Eintrags.
- Dieselbe Gruppe von SQLite-Transkriptzeilen für jede `sessionId`, auf die
  `entry.compactionCheckpoints[*].preCompaction.sessionId` verweist.
- Dieselbe Gruppe von SQLite-Transkriptzeilen für jede `sessionId`, auf die
  `entry.compactionCheckpoints[*].postCompaction.sessionId` verweist.
- Dieselbe Gruppe von SQLite-Transkriptzeilen für jede `sessionId` in
  `entry.usageFamilySessionIds`.

Archivieren Sie nur Zeilen, auf die weder eine verbleibende
`session_entries`-Zeile noch die Compaction- oder Nutzungsfamilienmetadaten
eines verbleibenden Eintrags verweisen. Dadurch bleiben der Zustand für Checkpoint-Verzweigung/-Wiederherstellung und Nutzungsaggregation erhalten, bis
die letzte aktive Referenz entfernt wurde.

## Nicht zur Familie gehörende Artefakte nach der Umstellung

Generierte themenspezifische Transkriptdateivarianten und Trajektorien-Sidecars sind kein aktiver
SQLite-Laufzeitzustand. Es handelt sich um veraltete Dateiartefakte:

- Themenvarianten wie `<sessionId>-topic-<thread>.jsonl` existieren nur für das
  dateibasierte Transkriptformat. SQLite verwendet die kanonische Sitzungs-ID sowie
  `session_routes`/Eintragszustellungsmetadaten anstelle themenspezifischer JSONL-Dateien.
- Trajektorien-Sidecars wie `.trajectory.jsonl` und `.trajectory-path.json`
  werden anhand tatsächlicher JSONL-`sessionFile`-Pfade benannt. SQLite-`sessionFile`-Werte sind
  `sqlite:<agentId>:<sessionId>:<storePath>`-Marker und bezeichnen keine Sidecar-Dateien.
- Leser der Archivierungsebene müssen weiterhin ältere archivierte JSONL-Dateien lesen, aber
  die Laufzeitaufbewahrung darf weder aktive Sitzungsverzeichnisse durchsuchen noch JSONL-
  Transkriptdateien für SQLite-Sitzungen erneut öffnen.

Der Doctor-Import bleibt für ältere primäre JSONL-Dateien und
deren benachbarte Trajektorien-Sidecars zuständig. Die SQLite-Laufzeitaufbewahrung darf keinen
zweiten Importer oder Datei-Fallback hinzufügen.

## Änderungspunkte

Erweitern Sie den durch `clawdbot-d63.1` eingeführten SQLite-Archivierungshelfer, statt
einen parallelen Pfad hinzuzufügen.

1. Fügen Sie nahe `deleteSqliteSessionStateIfUnreferenced` einen lokalen Collector hinzu:
   - `collectSqliteSessionArtifactFamily(entry: SessionEntry): Set<string>`
   - Beziehen Sie `entry.sessionId`, die Sitzungs-IDs vor und nach dem Checkpoint sowie
     `usageFamilySessionIds` ein.
   - Filtern Sie leere Zeichenfolgen und entfernen Sie Duplikate deterministisch.

2. Fügen Sie einen Referenz-Collector für den Speicher nach dem Entfernen hinzu:
   - `readReferencedSqliteSessionArtifactFamilyIds(database): Set<string>`
   - Durchlaufen Sie die aktuellen `session_entries`, parsen Sie jede `entry_json` und erfassen Sie
     dieselben Familien-IDs aus jedem verbleibenden Eintrag.

3. Ändern Sie die Aufrufer für Zurücksetzung, Löschung und Wartung, die derzeit eine
   entfernte `sessionId` archivieren, sodass sie die vollständige Familie des entfernten Eintrags übergeben.

4. Archivieren Sie für jede Familien-ID die SQLite-Transkriptzeilen mit dem vom Aufrufer angegebenen
   Grund (`reset` oder `deleted`) und löschen Sie anschließend die `sessions`-Zeile nur, wenn die
   Familien-ID nicht in der Referenzmenge nach dem Entfernen enthalten ist.

5. Lassen Sie das Löschen von Transkriptereignissen über den bestehenden SQLite-
   Bereinigungspfad für Sitzungszeilen zentralisiert. Fügen Sie keine aktiven JSONL-Lesevorgänge hinzu.

## Fokussierte Tests

Fügen Sie reine SQLite-Tests zu `src/config/sessions/session-accessor.conformance.test.ts`
oder zum benachbarten Lebenszyklustest hinzu, nachdem `clawdbot-d63.1` seine Änderungen übertragen hat:

- Beim Löschen eines Eintrags mit einem Transkript vor der Compaction werden sowohl die aktuelle
  Sitzung als auch die Sitzung vor der Compaction archiviert und anschließend beide SQLite-Zeilengruppen entfernt.
- Beim Löschen eines von zwei Einträgen, die eine Sitzung vor der Compaction gemeinsam verwenden, wird
  für die gemeinsam verwendete vorherige Sitzung nichts archiviert, bis der letzte darauf verweisende Eintrag
  entfernt wurde.
- Beim Löschen eines Eintrags mit `usageFamilySessionIds` werden die SQLite-
  Transkriptzeilen des Vorgängers archiviert, wenn kein anderer Eintrag auf diese Nutzungsfamilie verweist.
- Ein themenförmiger Sitzungsschlüssel mit einem SQLite-Marker verursacht weder das Lesen einer generierten
  themenspezifischen JSONL-Datei noch eine Sidecar-Suche.

Für den fokussierten Nachweis sollte Folgendes verwendet werden:

```bash
node scripts/run-vitest.mjs src/config/sessions/session-accessor.conformance.test.ts
```

Umfassende `pnpm`-Gates sollten für diesen Codex-Worktree auf Crabbox/Testbox verbleiben.
