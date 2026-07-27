---
read_when:
    - Je implementeert clawdbot-d63.2 / clawdbot-04b
    - Je werkt aan het bewaren, resetten, verwijderen of archiveren bij agentverwijdering van SQLite-sessies
    - Je moet artefactfamilies uit het SQLite-tijdperk onderscheiden van verouderde JSONL-sidecars
summary: Plan voor pad 3 voor het archiveren van alle SQLite-transcriptartefacten die bij een sessie horen
title: Pad 3 SQLite-sessieartefactfamilie
x-i18n:
    generated_at: "2026-07-27T05:05:19Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 29f4d541b2df5a06468fd0cee620b4340ee33eea1064f7d3ee823580c7b5760e
    source_path: plan/path3-sqlite-session-artifact-family.md
    workflow: 16
---

# Pad 3 SQLite-sessieartefactfamilie

Deze notitie bakent `clawdbot-d63.2` af, terwijl `clawdbot-d63.1` de overlappende
helper voor het resetten/verwijderen van archieven in `src/config/sessions/session-accessor.sqlite.ts` beheert.
Het implementatiebestand bevatte tijdens deze ronde niet-vastgelegde wijzigingen, dus dit artefact legt
het exacte contract en de patchpunten vast zonder de parallelle worker te hinderen.

## Gezaghebbende familie

Na de overstap naar SQLite zijn actieve sessietranscripten SQLite-rijen. De
archieffamilie van een sessie is:

- De rijen `transcript_events`, `transcript_event_identities` en `sessions`
  voor de huidige `sessionId` van de vermelding.
- Dezelfde verzameling SQLite-transcriptrijen voor elke `sessionId` waarnaar wordt verwezen door
  `entry.compactionCheckpoints[*].preCompaction.sessionId`.
- Dezelfde verzameling SQLite-transcriptrijen voor elke `sessionId` waarnaar wordt verwezen door
  `entry.compactionCheckpoints[*].postCompaction.sessionId`.
- Dezelfde verzameling SQLite-transcriptrijen voor elke `sessionId` in
  `entry.usageFamilySessionIds`.

Archiveer alleen rijen waarnaar niet langer wordt verwezen door een resterende
`session_entries`-rij of door metadata van de Compaction- of gebruiksfamilie van een resterende
vermelding. Dit behoudt de status van checkpointvertakkingen/-herstel en gebruiksaggregatie totdat
de laatste actieve verwijzing verdwenen is.

## Artefacten buiten de familie na de overstap

Gegenereerde varianten van topictranscriptbestanden en trajectory-sidecars zijn geen actieve
SQLite-runtimestatus. Het zijn verouderde bestandsartefacten:

- Topicvarianten zoals `<sessionId>-topic-<thread>.jsonl` bestaan alleen voor de
  bestandsgebaseerde transcriptindeling. SQLite gebruikt de canonieke sessie-id plus
  `session_routes`/afleveringsmetadata van de vermelding in plaats van JSONL-bestanden per topic.
- Trajectory-sidecars zoals `.trajectory.jsonl` en `.trajectory-path.json`
  worden benoemd op basis van echte JSONL-paden van `sessionFile`. SQLite-waarden van `sessionFile` zijn
  `sqlite:<agentId>:<sessionId>:<storePath>`-markeringen en benoemen geen
  sidecarbestanden.
- Lezers van de archieflaag moeten verouderde gearchiveerde JSONL-bestanden blijven lezen, maar
  runtimebewaring mag geen actieve sessiemappen scannen of JSONL-
  transcriptbestanden voor SQLite-sessies opnieuw openen.

Doctor-import blijft de migratie-eigenaar voor verouderde primaire JSONL-bestanden en
de aangrenzende trajectory-sidecars. Runtimebewaring in SQLite mag geen
tweede importer of bestandsterugval toevoegen.

## Patchpunten

Breid de SQLite-archiefhelper uit die door `clawdbot-d63.1` is geïntroduceerd, in plaats van
een parallel pad toe te voegen.

1. Voeg een lokale collector toe nabij `deleteSqliteSessionStateIfUnreferenced`:
   - `collectSqliteSessionArtifactFamily(entry: SessionEntry): Set<string>`
   - Neem `entry.sessionId`, sessie-id's vóór/na checkpoints en
     `usageFamilySessionIds` op.
   - Filter lege tekenreeksen en dedupliceer deterministisch.

2. Voeg een verwijzingscollector toe voor de opslag na verwijdering:
   - `readReferencedSqliteSessionArtifactFamilyIds(database): Set<string>`
   - Doorloop de huidige `session_entries`, parse elke `entry_json` en verzamel
     dezelfde familie-id's uit elke resterende vermelding.

3. Wijzig de aanroepers voor resetten/verwijderen/onderhoud die momenteel één
   verwijderde `sessionId` archiveren, zodat ze de volledige familie van de verwijderde vermelding doorgeven.

4. Archiveer voor elke familie-id de SQLite-transcriptrijen met de reden van de aanroeper
   (`reset` of `deleted`) en verwijder vervolgens de `sessions`-rij alleen wanneer de
   familie-id niet voorkomt in de verzameling verwijzingen na verwijdering.

5. Houd het verwijderen van transcriptgebeurtenissen gecentraliseerd via het bestaande SQLite-
   opschoonpad voor sessierijen. Voeg geen actieve JSONL-leesbewerkingen toe.

## Gerichte tests

Voeg alleen-voor-SQLite-tests toe aan `src/config/sessions/session-accessor.conformance.test.ts`
of aan de parallelle levenscyclustest nadat `clawdbot-d63.1` is vastgelegd:

- Het verwijderen van een vermelding met een transcript van vóór Compaction archiveert zowel de huidige
  sessie als de sessie van vóór Compaction en verwijdert vervolgens beide verzamelingen SQLite-rijen.
- Het verwijderen van één van twee vermeldingen die een Compaction-voorsessie delen, archiveert
  niets voor de gedeelde voorsessie totdat de laatste verwijzende vermelding is
  verwijderd.
- Het verwijderen van een vermelding met `usageFamilySessionIds` archiveert voorgaande SQLite-
  transcriptrijen wanneer geen andere vermelding naar die gebruiksfamilie verwijst.
- Een topicvormige sessiesleutel met een SQLite-markering veroorzaakt geen leesbewerking van een gegenereerd
  topic-JSONL-bestand of zoekactie naar sidecars.

Gebruik voor het gerichte bewijs:

```bash
node scripts/run-vitest.mjs src/config/sessions/session-accessor.conformance.test.ts
```

Brede `pnpm`-controles moeten voor deze Codex-worktree op Crabbox/Testbox blijven.
