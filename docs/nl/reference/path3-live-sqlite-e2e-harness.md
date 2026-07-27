---
read_when:
    - Je bewijst de omschakeling van Path 3 naar SQLite-opslag met een live Gateway
    - Je moet verwachte afwijkingen in verouderde JSONL onderscheiden van runtimefouten
    - Je bouwt of beoordeelt de agentgestuurde live SQLite-E2E-harnasomgeving
summary: Ontwerp voor live Gateway-bewijs van de Path 3-omschakeling van sessies/transcripten naar SQLite
title: Pad 3 live SQLite-E2E-harnas
x-i18n:
    generated_at: "2026-07-27T06:11:38Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 2749bf47cb4967bc80a5ed37a12f2a553f3b388ed8cd90cfb3217e1b5e8afae9
    source_path: reference/path3-live-sqlite-e2e-harness.md
    workflow: 16
---

De Path 3 live SQLite E2E-harness bewijst dat de Gateway SQLite als de
canonieke opslag voor sessies en transcripten gebruikt, terwijl verouderde JSONL-bestanden
invoer voor migratie of archiefmateriaal blijven. Het is een bewijsharness voor beheerders, geen
normaal diagnostisch hulpmiddel voor gebruikers.

Nadat een Gateway verkeer na de migratie heeft verwerkt, is pariteit met verouderde JSONL-bestanden
niet langer een geldig signaal voor de runtime-status. Een gezonde gemigreerde Gateway kan
SQLite-transcriptrijen hebben die afwijken van de aantallen in verouderde JSONL-bestanden, omdat nieuwe beurten
alleen SQLite horen bij te werken. De live harness moet daarom bij elke
stap het gedrag van de Gateway, wijzigingen in SQLite-rijen, de inactiviteit van verouderde bestanden en de logboekstatus meten.

## Opdrachtvorm

De bedoelde live opdracht is:

```bash
node scripts/path3-live-sqlite-e2e.mjs \
  --url http://127.0.0.1:18789 \
  --agent main \
  --session-key agent:main:path3-live-e2e:<timestamp> \
  --json
```

De opdracht maakt verbinding met een Gateway die al actief is. De opdracht start, stopt,
importeert of herhaalt de migratie niet, tenzij later een expliciete migratiemodus wordt
toegevoegd. Een CI-variant of geïsoleerde lokale variant kan
`test/helpers/openclaw-test-instance.ts` gebruiken, maar het live bewijspad hoort
de daadwerkelijke Gateway van de beheerder en de echte SQLite-database per agent te inspecteren.

## Geïsoleerd bewijs met gebouwde CLI

De bewijsrunner voor de gebouwde CLI vult een geïsoleerde verouderde sessieopslag, start de
opnieuw gebouwde Gateway en bewijst dat bij het opstarten actieve verouderde sessies in
SQLite worden geïmporteerd voordat runtime-lezingen beginnen. Deze mag `openclaw doctor --fix`
niet uitvoeren vóór de eerste start van de Gateway, omdat daarmee het handmatige migratiepad
zou worden bewezen in plaats van het upgradepad dat gebruikers bij de eerste opstart na de omschakeling krijgen.

Na de opstartimport mag het geïsoleerde bewijs
`openclaw doctor --session-sqlite inspect` en
`openclaw doctor --session-sqlite validate` uitvoeren als diagnostisch bewijs. Deze
doctor-opdrachten sturen de migratie niet aan voor het bewijs van de opstartupgrade.
Afzonderlijke doctor-importscenario's horen verouderde transcriptbestanden plus
traject-sidecars te vullen en te verifiëren dat doctor die artefacten archiveert terwijl SQLite
canoniek blijft.

## Voorcontrole

De voorcontrole verzamelt een nulmeting en mislukt voordat een bewijsbeurt wordt verzonden als de
Gateway niet bruikbaar is:

- `GET /health` en de diepgaande status van de Gateway moeten een actieve, bereikbare
  Gateway melden.
- De versies van de CLI en Gateway moeten overeenkomen met de geteste branch.
- De harness registreert een logboekcursor voor het actieve bestandslogboek van de Gateway.
- De harness registreert per agent de aantallen in SQLite-tabellen voor `sessions`,
  `session_entries`, `transcript_events`, `transcript_event_identities` en
  `session_routes`.
- De harness registreert `mtime`, `size` en het bestaan van verouderde
  `sessions.json`, JSONL-bestanden waarnaar wordt verwezen en mogelijke JSONL-paden
  voor bewijssessies.
- `lsof -p <gateway-pid>` moet SQLite DB/WAL/SHM-handles tonen en geen actieve
  `.jsonl`- of `sessions.json`-handles.

`openclaw doctor --session-sqlite validate` is in live modus uitsluitend informatief.
Na verkeer na de omschakeling kan dit verwachte afwijkingen ten opzichte van verouderde bestanden melden. De
harness hoort doctor-uitvoer te gebruiken voor classificatie en migratie-inventarisatie,
niet als het runtime-orakel voor slagen of mislukken.

## Agentgestuurd scenario

Het live scenario gebruikt een speciale sessiesleutel voor bewijs en stuurt de Gateway
waar mogelijk via openbare RPC-paden aan. Eén agentbeurt hoort voldoende te zijn om
normale persistentie uit te oefenen, maar het volledige bewijs hoort de 3.1b-overgangen
te dekken waarvoor eerder afzonderlijke live controles nodig waren:

- Normale chatbeurt: maak de bewijssessie of hergebruik deze, verzend een echte agentprompt,
  wacht op het uiteindelijke assistentresultaat en verifieer `chat.history` of
  een gelijkwaardige Gateway-projectie.
- Transcriptidentiteit: verifieer dat dezelfde markering in de Gateway-geschiedenis en in
  SQLite-transcriptrijen voorkomt, inclusief rijen met stabiele gebeurtenisidentiteiten indien aanwezig.
- Accessors voor sessiemetadata: lees de bewijssessie en geselecteerde bestaande live
  sessies via Gateway-/sessie-accessors en vergelijk ze met SQLite-rijen.
- Projectie van sessiepatch: pas een omkeerbare wijziging in model-/sessiemetadata toe op
  de bewijssessie en verifieer vervolgens dat de geprojecteerde rij en het Gateway-antwoord overeenkomen.
- Levenscyclus van het Compaction-controlepunt: vermeld, vertak en herstel een controlepunt uitsluitend
  in de bewijssessie of een synthetische fixturesessie die door de harness is gemaakt.
- Herstel na opnieuw starten: voer het veilige pad voor herstelmarkeringen uit op een gecontroleerde
  bewijssessie of een geïsoleerde testinstantie; in live modus mag deze stap alleen worden uitgevoerd wanneer
  de doelverzameling sessies expliciet en omkeerbaar is.
- Opschoningslevenscyclus: verwijder of reset de bewijssessie en verifieer vervolgens de SQLite-
  levenscyclusrijen en de gearchiveerde transcriptstatus.

Transportspecifieke overgangen die niet veilig op de live Gateway van de beheerder
kunnen worden uitgevoerd, zoals inkomend WhatsApp- of spraakoproepverkeer, horen runtime-
probes op eigenaarsniveau tegen hetzelfde SQLite-contract te gebruiken in plaats van extern transport te simuleren.

## Asserties per stap

Elke stap maakt momentopnamen van de toestand vóór en na de stap en schrijft een gestructureerde
assertierecord:

- Aantallen SQLite-rijen nemen alleen toe waar dat wordt verwacht.
- Runtime-rijen voor trajecten nemen toe voor op markeringen gebaseerde bewijssessies die
  runtime-gebeurtenissen registreren.
- De rij van de bewijssessie heeft de verwachte `session_id`, status, tijdstempels,
  metadata en routerijen.
- De geschiedenis-/sessieprojectie van de Gateway komt overeen met het einde van het SQLite-transcript.
- Er wordt geen JSONL-bestand voor de bewijssessie gemaakt of gewijzigd.
- Er wordt geen `.trajectory.jsonl`-, `.trajectory-path.json`- of
  van de markering afgeleide `trajectory/<session>.jsonl`-sidecar voor de bewijssessie gemaakt.
- Bestaande verouderde JSONL-bestanden en `sessions.json` blijven ongewijzigd, tenzij de
  stap expliciet een offline migratie- of archiefbewerking is.
- Het Gateway-proces opent geen `.jsonl`- of `sessions.json`-handles.
- Logboeken sinds de vorige cursor bevatten geen `ERROR`, `FATAL`, `SQLITE_`,
  `no such column`, onbeschikbare sessieopslag, mislukking bij herstel na opnieuw starten of
  waarschuwing voor transcriptreconciliatie, tenzij het scenario dit expliciet toestaat.

De logboekscan maakt deel uit van het contract voor slagen of mislukken. Een Gateway die
statuscontroles beantwoordt maar SQLite-schemafouten of herhaalde mislukkingen bij
transcriptreconciliatie meldt, is niet groen voor Path 3.

## Bewijsartefact

De harness hoort bewijs te schrijven onder `.artifacts/path3-live-e2e/<timestamp>/`
en dit buiten git te houden:

- `summary.json`: opdrachtargumenten, Gateway-versie, resultaat, mislukte assertie en
  artefactpaden.
- `sqlite-before.json` en `sqlite-after.json`: aantallen rijen en geselecteerde bewijsrijen.
- `legacy-files.json`: bestaan van verouderde bestanden, `mtime`, grootte en of elk
  bestand is gewijzigd.
- `gateway-log-scan.json`: cursorbereik, overeenkomende logboekregels en
  beslissingen over de toelatingslijst.
- `events.jsonl`: geordende waarnemingen per stap die geschikt zijn voor bewijsreacties bij een PR.

Het PR-bewijs hoort deze artefacten samen te vatten in plaats van volledige
transcripten of privéberichtinhoud te plakken.

## Veiligheidsregels

- In live modus mogen verouderde JSONL-bestanden nooit opnieuw worden geïmporteerd terwijl de Gateway actief is.
- Live modus mag niet-bewijssessies niet wijzigen, behalve voor expliciet geselecteerde,
  omkeerbare herstelprobes.
- Elke destructieve of brede migratiestap vereist een nieuwe back-up van de
  betrokken SQLite-DB en verouderde sessiemap.
- Back-ups horen beperkt te blijven tot de betrokken agent-DB/sessiemap en tijdens
  één bewijsuitvoering te worden hergebruikt om onbeperkte groei van schijfgebruik te voorkomen.
- De opschoningsstap mag geen bewijssessie, bewijs-JSONL of gewijzigd verouderd
  bestand achterlaten, tenzij de aanroeper `--keep-artifacts` doorgeeft.

## Geslaagd resultaat

Een geslaagde live uitvoering betekent dat de Gateway een echte agentgestuurde sessiestroom heeft geaccepteerd,
alle waargenomen canonieke toestand zich in SQLite bevond, verouderde runtimebestanden
inactief bleven en de logboekstatus gedurende het gemeten venster schoon bleef. Dit betekent niet
dat pariteit met verouderde JSONL-bestanden na live verkeer schoon blijft; live afwijkingen worden verwacht
zodra SQLite de canonieke opslag is.
