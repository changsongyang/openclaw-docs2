---
read_when:
    - OpenClaw-runtimegegevens, cache, transcripties, taakstatus of tijdelijke bestanden naar SQLite verplaatsen
    - Doctor-migraties ontwerpen voor verouderde JSON- of JSONL-bestanden
    - Gedrag voor back-ups, herstel, VFS of workeropslag wijzigen
    - Sessievergrendelingen verwijderen, opschonen, afkappen of JSON-compatibiliteitspaden verwijderen
summary: Migratieplan om SQLite de primaire duurzame opslaglaag voor status en cache te maken, terwijl configuratie in bestanden blijft opgeslagen
title: Databasegerichte statusrefactor
x-i18n:
    generated_at: "2026-07-27T05:20:50Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: ae4d72f04c1228742cc7ea3cc87a96b06aa1e2b750ace23cca5b387844746186
    source_path: refactor/database-first.md
    workflow: 16
---

# Database-eerst-statusrefactor

## Besluit

Gebruik een SQLite-indeling met twee niveaus:

- Globale database: `~/.openclaw/state/openclaw.sqlite`
- Agentdatabase: één SQLite-database per agent voor de werkruimte,
  het transcript, VFS, artefacten en omvangrijke runtime-status die eigendom zijn van de agent
- Configuratie blijft bestandsgebaseerd: `openclaw.json` blijft buiten de
  database. Runtime-authenticatieprofielen worden naar SQLite verplaatst; referentiebestanden van
  externe providers of de CLI blijven buiten de database van OpenClaw onder beheer van hun eigenaar.

De globale database is de database van het besturingsvlak. Deze beheert agentdetectie,
gedeelde Gateway-status, koppeling, apparaat-/Node-status, taak- en flowlogboeken, Plugin-
status, runtime-status van de planner, back-upmetagegevens en migratiestatus.

De agentdatabase is de database van het gegevensvlak. Deze beheert de sessie-
metagegevens van de agent, de stroom transcriptgebeurtenissen, de VFS-werkruimte of tijdelijke naamruimte,
toolartefacten, uitvoeringsartefacten en doorzoekbare/indexeerbare agentlokale cachegegevens.

Dit biedt één duurzaam globaal overzicht zonder omvangrijke agentwerkruimten,
transcripten en binaire tijdelijke gegevens in het gedeelde schrijfpad van de Gateway te dwingen.

## Hard contract

Deze migratie heeft één canonieke runtimevorm:

- Sessierijen slaan uitsluitend sessiemetagegevens op. Ze mogen geen
  `transcriptLocator`, bestandspaden van transcripten, paden van verwante JSONL-bestanden, vergrendelingspaden,
  opschoningsmetagegevens of compatibiliteitsverwijzingen uit het bestandstijdperk opslaan.
- Transcriptidentiteit is altijd SQLite-identiteit: `{agentId, sessionId}` plus
  optionele onderwerpmetagegevens waar het protocol die nodig heeft.
- `sqlite-transcript://...` is geen runtime- of protocolidentiteit. Nieuwe code mag
  geen transcriptlocators afleiden, opslaan, doorgeven, parseren of migreren. Runtime en
  tests mogen helemaal geen pseudolocators bevatten; documentatie mag de tekenreeks
  alleen vermelden om deze te verbieden.
- Verouderde `sessions.json`, transcript-JSONL, `.jsonl.lock`, opschoning, afkapping
  en oude sessiepadlogica horen uitsluitend bij het migratie-/importpad van doctor.
- Verouderde aliassen voor sessieconfiguratie horen uitsluitend bij doctormigratie. Runtime
  interpreteert geen `session.idleMinutes`, `session.resetByType.dm` of
  agentoverschrijdende `agent:main:*`-aliassen voor de hoofdsessie van een andere geconfigureerde agent.
- Routeringsidentiteit van sessies is getypeerde relationele status. Intensief gebruikte runtime- en UI-paden
  moeten `sessions.session_scope`, `sessions.account_id`,
  `sessions.primary_conversation_id`, `conversations` en
  `session_conversations` lezen; ze mogen `session_key` niet parseren of
  `session_entries.entry_json` niet doorzoeken naar provideridentiteit, behalve als compatibiliteits-
  schaduw terwijl oude aanroeplocaties worden verwijderd.
- Markeringen voor directe berichten op kanaalniveau, zoals `dm` tegenover `direct`, zijn routerings-
  vocabulaire, geen transcriptlocators of compatibiliteitshandvatten voor bestandsopslag.
- Verouderde configuratie van hookhandlers hoort uitsluitend bij waarschuwings-/migratieoppervlakken van doctor.
  Runtime mag `hooks.internal.handlers` niet laden; hooks worden uitsluitend uitgevoerd via ontdekte
  hookmappen en `HOOK.md`-metagegevens.
- Runtime-opstart, intensief gebruikte antwoordpaden, Compaction, reset, herstel, diagnostiek,
  TTS, geheugenhooks, subagents, routering van Plugin-opdrachten, protocolgrenzen en
  hooks moeten `{agentId, sessionId}` door de runtime doorgeven.
- Tests moeten SQLite-transcriptrijen instellen en controleren via
  `{agentId, sessionId}`. Tests die uitsluitend het doorgeven van JSONL-paden,
  het behouden van door de aanroeper aangeleverde locators of compatibiliteit met transcriptbestanden aantonen,
  moeten worden verwijderd, tenzij ze betrekking hebben op doctorimport, niet-sessiegebonden
  materialisatie voor ondersteuning/debugging of protocolvorm.
- `runEmbeddedPiAgent(...)`, voorbereide workeruitvoeringen en de binnenste ingebedde
  poging mogen geen transcriptlocators accepteren. Ze openen de SQLite-transcript-
  manager op basis van `{agentId, sessionId}` en geven die manager door aan de geïnternaliseerde
  PI-compatibele agentsessie, zodat verouderde aanroepers de runner geen
  JSON/JSONL-transcripten kunnen laten schrijven.
- Runnerdiagnostiek moet runtime-/cache-/payloadtracerecords in SQLite opslaan.
  Runtimediagnostiek mag geen opties voor het overschrijven van JSONL-bestanden of algemene
  exporthelpers voor transcript-JSONL beschikbaar stellen; gebruikersgerichte exports kunnen expliciete
  artefacten uit databaserijen materialiseren zonder bestandsnamen terug de runtime in te voeren.
- Ruwe streamlogregistratie gebruikt `OPENCLAW_RAW_STREAM=1` plus SQLite-diagnostiekrijen.
  Het oude pi-mono-contract voor bestandsloggers met `PI_RAW_STREAM`, `PI_RAW_STREAM_PATH` en
  `raw-openai-completions.jsonl` maakt geen deel uit van de OpenClaw-
  runtime of tests.
- QMD-geheugenindexering mag SQLite-transcripten niet naar Markdown-bestanden exporteren.
  QMD indexeert uitsluitend geconfigureerde geheugenbestanden; het zoeken in sessietranscripten blijft
  SQLite-gebaseerd.
- Het SDK-subpad van QMD is voor nieuwe code uitsluitend voor QMD. Helpers voor het indexeren
  van SQLite-sessietranscripten bevinden zich in `memory-core-host-engine-session-transcripts`; elke
  QMD-herexport is uitsluitend voor compatibiliteit en mag niet door runtimecode worden gebruikt.
- Ingebouwde geheugenindexen bevinden zich in de database van de betreffende agent. Runtimeconfiguratie en
  opgeloste runtimecontracten mogen `memorySearch.store.path` niet beschikbaar stellen; doctor
  verwijdert die verouderde configuratiesleutel en de huidige code geeft de
  `databasePath` van de agent intern door.

Bij de implementatie moet code worden verwijderd totdat deze uitspraken zonder
uitzonderingen buiten de grenzen voor doctor/import/export/debugging waar zijn.

## Doelstatus en voortgang

### Hard doel

- Eén globale SQLite-database beheert de status van het besturingsvlak:
  `state/openclaw.sqlite`.
- Eén SQLite-database per agent beheert de status van het gegevensvlak:
  `agents/<agentId>/agent/openclaw-agent.sqlite`.
- Configuratie blijft bestandsgebaseerd. `openclaw.json` maakt geen deel uit van deze database-
  refactor.
- Verouderde bestanden dienen uitsluitend als invoer voor doctormigratie.
- Runtime schrijft of leest sessie- of transcript-JSONL nooit als actieve status.

### Doelstatussen

- `not-started`: runtimecode uit het bestandstijdperk schrijft nog steeds actieve status.
- `migrating`: doctor-/importcode kan bestandsgegevens naar SQLite verplaatsen.
- `dual-read`: tijdelijke brug leest zowel SQLite als verouderde bestanden. Deze status
  is voor deze refactor verboden, tenzij deze expliciet als uitsluitend voor doctor
  is gedocumenteerd.
- `sqlite-runtime`: runtime leest en schrijft uitsluitend SQLite.
- `clean`: verouderde runtime-API's en tests zijn verwijderd en de bewaking voorkomt
  regressies.
- `done`: documentatie, tests, back-up, doctormigratie en controles van wijzigingen bewijzen de
  opgeschoonde status.

### Huidige status

- Sessies: `clean` voor runtime. Sessierijen bevinden zich in de database per agent,
  runtime-API's gebruiken `{agentId, sessionId}` of `{agentId, sessionKey}`, en
  `sessions.json` is uitsluitend verouderde invoer voor doctor.
- Transcripten: `clean` voor runtime. Transcriptgebeurtenissen, identiteiten, momentopnamen
  en runtimegebeurtenissen van trajecten bevinden zich in de database per agent. Runtime
  accepteert geen transcriptlocators of paden naar JSONL-transcripten meer.
- Ingebedde PI-runner: `clean`. Ingebedde PI-uitvoeringen, voorbereide workers, Compaction
  en herhalingslussen gebruiken SQLite-sessiebereik en weigeren verouderde transcripthandvatten.
- Cron: `clean` voor runtime. Runtime gebruikt `cron_jobs` en `task_runs` van Cron;
  runtimetests gebruiken SQLite-naamgeving met `storeKey`, en Cron-paden uit het bestandstijdperk blijven
  uitsluitend aanwezig in tests voor verouderde doctormigratie.
- Taakregister: `clean`. Runtime-rijen voor taken en TaskFlow bevinden zich in
  `state/openclaw.sqlite`; niet-uitgebrachte SQLite-importers voor sidecars zijn verwijderd.
- Plugin-status: `clean`. Rijen voor Plugin-status/blobs bevinden zich in de gedeelde globale
  database; oude SQLite-helpers voor sidecars met Plugin-status worden tegengehouden.
- Geheugen: `sqlite-runtime` voor ingebouwd geheugen en indexering van sessietranscripten.
  Geheugenindextabellen bevinden zich in de database per agent, geheugenstatus van Plugins gebruikt
  gedeelde rijen voor Plugin-status en verouderde geheugenbestanden zijn invoer voor doctormigratie
  of inhoud van de gebruikerswerkruimte.
- Back-up: `sqlite-runtime`. Back-up verwerkt compacte SQLite-momentopnamen, laat actieve
  WAL-/SHM-sidecars weg, verifieert de integriteit van SQLite en registreert back-upuitvoeringen in de
  globale database.
- Werkruimte-instelling: `sqlite-runtime`. Voltooiing van de instelling, attestaties van de werkruimte
  en gegenereerde bootstrap-hashes bevinden zich in getypeerde gedeelde SQLite-tabellen. Runtime
  leest of schrijft de buiten gebruik gestelde werkruimte-JSON en `.attested`-sidecars niet;
  doctor beheert de gevalideerde import en geverifieerde verwijdering ervan.
- Doctormigratie: `migrating`, met opzet. Doctor importeert verouderde JSON,
  JSONL en buiten gebruik gestelde sidecaropslag in SQLite, registreert migratie-uitvoeringen/-bronnen
  en verwijdert bronnen na succesvolle verwerking.
- Uitvoeringsgoedkeuringen: `file-runtime`. TypeScript en macOS lezen en schrijven nog steeds
  `exec-approvals.json` van de actieve statusmap; het gereserveerde
  `exec_approvals_config`-schema heeft nog geen runtime-eigenaar. Een toekomstige omschakeling moet
  doctorimport binnen dezelfde status toevoegen en beide runtimes samen verplaatsen.
- E2E-scripts: `clean` voor runtimedekking. Docker MCP-seeding schrijft SQLite-
  rijen. Het Docker-script voor runtimecontext maakt alleen verouderde JSONL aan binnen de
  doctormigratieseed en benoemt het pad van de verouderde sessie-index expliciet.

### Resterend werk

- [x] Hernoem opslagvariabelen in Cron-runtimetests zodat ze niet langer `storePath` gebruiken, tenzij
      het verouderde invoer voor doctor betreft.
      Bestanden: `src/cron/service.test-harness.ts`,
      `src/cron/service.runs-one-shot-main-job-disables-it.test.ts`,
      `src/cron/service/timer.regression.test.ts`,
      `src/cron/service/ops.test.ts`, `src/cron/service/store.test.ts`,
      `src/cron/service.heartbeat-ok-summary-suppressed.test.ts`,
      `src/cron/service.main-job-passes-heartbeat-target-last.test.ts`,
      `src/cron/store.test.ts`.
      Bewijs: `pnpm check:database-first-legacy-stores`; `rg -n 'storePath' src/cron --glob '!**/commands/doctor/**'`.
- [x] Verwijder of hernoem verouderde exporttestmocks uit het bestandstijdperk.
      Bestand: `src/auto-reply/reply/commands-export-test-mocks.ts`.
      Bewijs: `rg -n 'resolveSessionFilePath|sessionFile|storePath|transcriptLocator' src/auto-reply/reply`.
- [x] Maak duidelijk dat de verouderde JSONL-seed voor de Docker-runtimecontext uitsluitend voor doctor is.
      Bestand: `scripts/e2e/session-runtime-context-docker-client.ts`.
      Bewijs: `rg -n 'sessions\\.json|sessionFile|\\.jsonl' scripts/e2e/session-runtime-context-docker-client.ts` toont uitsluitend
      `seedBrokenLegacySessionForDoctorMigration`.
- [x] Houd door Kysely gegenereerde typen uitgelijnd na elke schemawijziging.
      Bestanden: `src/state/openclaw-state-schema.sql`,
      `src/state/openclaw-agent-schema.sql`,
      `src/state/*generated*`.
      Bewijs: geen schemawijziging in deze ronde; `pnpm db:kysely:check`;
      `pnpm lint:kysely`.
- [x] Voer gerichte tests voor aangeraakte opslaglagen, opdrachten en scripts opnieuw uit.
      Bewijs: `pnpm test src/cron/service/store.test.ts src/cron/store.test.ts src/cron/service.heartbeat-ok-summary-suppressed.test.ts src/cron/service.main-job-passes-heartbeat-target-last.test.ts src/cron/service.every-jobs-fire.test.ts src/cron/service.persists-delivered-status.test.ts src/cron/service.runs-one-shot-main-job-disables-it.test.ts src/cron/service/ops.test.ts src/cron/service/timer.regression.test.ts src/auto-reply/reply/commands-export-session.test.ts extensions/telegram/src/thread-bindings.test.ts extensions/slack/src/monitor/message-handler/prepare.test.ts src/acp/translator.session-lineage-meta.test.ts`; `git diff --check`.
- [x] Voer de gewijzigde controle of uitgebreid extern bewijs uit voordat `done` wordt verklaard.
      Bewijs: `pnpm check:changed --timed -- <changed extension paths>` is geslaagd tijdens
      Hetzner Crabbox-uitvoering `run_3f1cabf6b25c` na tijdelijke instelling van Node 24/pnpm en
      expliciete padroutering voor de gesynchroniseerde werkruimte zonder `.git`.

### Voorkom regressies

- Geen transcriptlocators.
- Geen actieve sessiebestanden.
- Geen nagebootste JSONL-testfixtures, behalve tests voor verouderde doctormigratie.
- Geen rechtstreekse SQLite-toegang waar Kysely wordt verwacht.
- Geen nieuwe databasemigraties uit het bestandstijdperk. Het globale schema blijft op versie `1`.
  Het uitgebrachte schema per agent van versie `1` heeft één begrensde runtimemigratie naar
  versie `2` voor stabiele identiteiten van geheugenbronnen.

## Aannamen bij het lezen van de code

Er zijn geen aanvullende productbeslissingen die dit plan blokkeren. De implementatie moet
doorgaan met deze aannamen:

- Gebruik `node:sqlite` rechtstreeks en vereis een WAL-resetveilige Node-runtime
  (22.22.3+, 24.15+ of 25.9+) voor dit opslagpad.
- Behoud precies één normaal configuratiebestand. Verplaats bij deze refactor de configuratie, Plugin-
  manifesten of Git-werkruimten niet naar SQLite.
- Compatibiliteitsbestanden voor de runtime zijn niet vereist. Verouderde JSON- en JSONL-bestanden zijn
  uitsluitend migratie-invoer. De branchlokale SQLite-sidecars zijn nooit uitgebracht en worden
  verwijderd in plaats van geïmporteerd.
- `openclaw doctor --fix` beheert de migratie van verouderde bestanden naar de database. Het opstarten van de runtime
  beheert uitsluitend begrensde upgrades tussen uitgebrachte SQLite-schemaversies;
  daarbij mag geen status uit het bestandstijdperk worden geïmporteerd.
- Voor compatibiliteit van aanmeldgegevens geldt dezelfde regel: runtime-aanmeldgegevens bevinden zich in
  SQLite. Oude `auth-profiles.json`-, `auth.json`-bestanden per agent en gedeelde
  `credentials/oauth.json`-bestanden zijn migratie-invoer voor doctor en worden vervolgens
  na de import verwijderd.
- De gegenereerde status van de modelcatalogus wordt door de database ondersteund. Runtimecode mag niet naar
  `agents/<agentId>/agent/models.json` schrijven; bestaande `models.json`-bestanden zijn verouderde
  invoer voor doctor en worden verwijderd nadat ze in `agent_model_catalogs` zijn geïmporteerd.
- De runtime mag transcriptlocators niet migreren, normaliseren of overbruggen. De identiteit van het actieve
  transcript is `{agentId, sessionId}` in SQLite. Bestandspaden zijn uitsluitend verouderde invoer
  voor doctor en `sqlite-transcript://...` moet uit runtime-, protocol-, hook- en
  Plugin-oppervlakken verdwijnen in plaats van als grens-handle te worden
  behandeld.
- Bij het lezen van SQLite-transcripten door de runtime worden geen oude migraties van JSONL-itemvormen uitgevoerd en
  worden geen volledige transcripten herschreven voor compatibiliteit. Normalisatie van verouderde items blijft in
  expliciete doctor-/importhulpprogramma's. Doctor normaliseert verouderde JSONL-transcriptbestanden
  voordat SQLite-rijen worden ingevoegd; huidige runtimerijen zijn
  al volgens het huidige transcriptschema geschreven. Traject-/sessie-export
  leest die rijen zoals ze zijn en mag tijdens het exporteren geen verouderde migraties uitvoeren.
- Helpers voor het parseren/migreren van verouderde JSONL-transcripten zijn uitsluitend voor doctor. Runtimecode
  voor transcriptindelingen bouwt alleen de huidige SQLite-transcriptcontext; doctor
  beheert upgrades van oude JSONL-items voordat rijen worden ingevoegd.
- De oude, door de runtime beheerde helper voor het streamen van JSONL-transcripten is verwijderd. Doctor-
  importcode beheert expliciete leesbewerkingen van verouderde bestanden; de sessiegeschiedenis van de runtime leest
  SQLite-rijen.
- Codex-app-serverbindingen gebruiken de OpenClaw `sessionId` als canonieke
  sleutel in de Plugin-statusnaamruimte van Codex. `sessionKey` is metadata voor
  routering/weergave en mag de duurzame sessie-id niet vervangen of de identiteit van
  transcriptbestanden opnieuw invoeren.
- Contextengines ontvangen het huidige runtimecontract rechtstreeks. Het register
  mag engines niet omwikkelen met nieuwe-poging-shims die `sessionKey`,
  `transcriptScope` of `prompt` verwijderen; engines die de huidige
  database-eerst-parameters niet kunnen accepteren, moeten duidelijk mislukken in plaats van te worden overbrugd.
- De back-upuitvoer moet één archiefbestand blijven. Database-inhoud moet dat
  archief binnengaan als compacte SQLite-snapshots, niet als onbewerkte actieve WAL-sidecars.
- Zoeken in transcripten is nuttig, maar niet vereist voor de eerste
  database-eerst-versie. Ontwerp het schema zo dat FTS later kan worden toegevoegd.
- Workeruitvoering moet experimenteel blijven en achter instellingen verborgen zijn terwijl de database-
  grens zich stabiliseert.

## Bevindingen uit de codeanalyse

De huidige branch is het proof-of-conceptstadium al voorbij. De gedeelde
database bestaat, Node `node:sqlite` is via een kleine runtimehelper aangesloten en
voormalige opslagplaatsen schrijven nu naar `state/openclaw.sqlite` of de bijbehorende
`openclaw-agent.sqlite`-database.

Het resterende werk gaat niet om het kiezen van SQLite, maar om het schoonhouden van de nieuwe grens
en het verwijderen van interfaces met een compatibiliteitsvorm die nog op de oude
bestandswereld lijken:

- Sessie-`storePath` is niet langer een runtime-identiteit, vorm van een testfixture of
  veld in een statuspayload. Runtime- en bridgetests bevatten niet langer de
  contractnaam `storePath`; doctor-/migratiecode beheert die verouderde terminologie.
- Sessieschrijfbewerkingen lopen niet langer via de oude in-process `store-writer.ts`-
  wachtrij. SQLite-patchschrijfbewerkingen worden buiten de transactie voorbereid en gebruiken vervolgens een korte
  synchrone validatie-/toepassingstransactie met expliciete conflictdetectie.
- Detectie van verouderde paden heeft nog steeds geldige migratietoepassingen, maar runtimecode moet
  `sessions.json` en JSONL-transcriptbestanden niet langer als mogelijke schrijfdoelen
  behandelen.
- Tabellen die eigendom zijn van agents bevinden zich in SQLite-databases per agent. De globale database bevat
  register-/control-plane-rijen; de transcriptidentiteit is `{agentId, sessionId}` in
  de transcriptrijen per agent. Runtimecode mag geen transcriptbestandspaden
  opslaan of transcriptlocators migreren.
- Doctor importeert al verschillende verouderde bestanden. De opschoning bestaat erin dit tot één
  expliciete migratie-implementatie te maken die door doctor wordt aangeroepen, met een duurzaam
  migratierapport.

Er zijn geen aanvullende productvragen die de implementatie blokkeren.

## Huidige codestructuur

De branch heeft al een echte gedeelde SQLite-basis:

- De minimale runtimeversie vereist nu een Node-build die veilig is bij een WAL-reset: 22.22.3+,
  24.15+ of 25.9+. `package.json`, de runtimecontrole van de CLI, de standaardinstellingen van het installatieprogramma,
  de runtimezoeker voor macOS, CI en de openbare installatiedocumentatie zijn allemaal op elkaar afgestemd.
- `src/state/openclaw-state-db.ts` opent `openclaw.sqlite`, stelt WAL in,
  `synchronous=NORMAL`, `busy_timeout=30000`, `foreign_keys=ON` en past
  de gegenereerde schemamodule toe die is afgeleid van
  `src/state/openclaw-state-schema.sql`.
- Kysely-tabeltypen en runtimeschemamodules worden gegenereerd vanuit tijdelijke
  SQLite-databases die zijn gemaakt op basis van de vastgelegde `.sql`-bestanden; runtimecode
  bevat niet langer gekopieerde schematekenreeksen voor globale databases, databases per agent of
  proxy-opnamedatabases.
- Runtimestores leiden geselecteerde en ingevoegde rijtypen af van die gegenereerde
  Kysely-interfaces `DB` in plaats van SQLite-rijstructuren handmatig te dupliceren. Ruwe SQL
  blijft beperkt tot het toepassen van schema's, pragma's en DDL die uitsluitend voor migraties dient.
- Het globale SQLite-schema blijft op `user_version = 1`. Het schema per agent
  heeft versie `2`; de opener migreert atomair de meegeleverde memory-source-sleutel van versie `1`
  naar een stabiele gehele identiteit. Import van bestand naar database
  blijft onderdeel van de doctor-code.
- Relationeel eigenaarschap wordt afgedwongen waar de eigendomsgrens canoniek is:
  bronmigratierijen worden trapsgewijs verwijderd vanaf `migration_runs`, de afleveringsstatus van taken
  vanaf `task_runs` en transcriptidentiteitsrijen vanaf
  transcriptgebeurtenissen.
- De huidige gedeelde tabellen omvatten `agent_databases`,
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
  `subagent_runs`, `migration_runs` en `backup_runs`.
- Willekeurige status die eigendom is van plugins krijgt geen getypeerde tabellen die eigendom zijn van de host. Geïnstalleerde
  plugins gebruiken `plugin_state_entries` voor JSON-payloads met versiebeheer en
  `plugin_blob_entries` voor bytes, met eigenaarschap van naamruimte/sleutel, TTL-opschoning,
  back-ups en pluginmigratierecords. Pluginorkestratiestatus die eigendom is van de host kan
  nog steeds getypeerde tabellen hebben wanneer de host eigenaar is van het querycontract, zoals
  `plugin_binding_approvals`.
- Pluginmigraties zijn datamigraties binnen naamruimten die eigendom zijn van plugins, geen
  migraties van het hostschema. Een plugin kan zijn eigen status- en blobvermeldingen met versiebeheer
  migreren via een migratieprovider, waarna de host de bron- en uitvoeringsstatus vastlegt in het
  normale migratielogboek. Nieuwe plugininstallaties vereisen geen wijziging van
  `openclaw-state-schema.sql`, tenzij de host zelf eigenaar wordt van een
  nieuw contract tussen plugins.
- `src/state/openclaw-agent-db.ts` opent
  `agents/<agentId>/agent/openclaw-agent.sqlite`, registreert de database in de
  globale database en beheert agentlokale tabellen voor sessies, transcripten, VFS, artefacten, caches
  en geheugenindexen. Gedeelde runtime-detectie leest nu het gegenereerde, getypeerde
  register `agent_databases` in plaats van die query op elke aanroeplocatie
  opnieuw te implementeren.
- Globale databases en databases per agent leggen een `schema_meta`-rij vast met de databaserol,
  schemaversie, tijdstempels en agent-id voor agentdatabases. De globale database
  blijft op `user_version = 1`; databases per agent gebruiken versie `2` na de begrensde
  migratie van de memory-source-identiteit.
- De sessie-identiteit per agent heeft nu een canonieke hoofdtabel `sessions` met
  `session_id` als sleutel, en `session_key`, `session_scope`, `account_id`,
  `primary_conversation_id`, tijdstempels, weergavevelden, modelmetadata,
  harness-id en bovenliggende/spawn-koppelingen als opvraagbare kolommen. `session_routes`
  is de unieke actieve route-index van `session_key` naar de huidige
  `session_id`, zodat een routesleutel naar een nieuwe duurzame sessie kan worden verplaatst zonder
  dat snelle leesbewerkingen tussen dubbele `sessions.session_key`-rijen moeten kiezen. De oude,
  op compatibiliteit gerichte payload `session_entries.entry_json` hangt via een refererende sleutel onder
  de duurzame hoofdstructuur `session_id`; deze is niet langer de enige
  representatie van een sessie op schemaniveau.
- De externe gespreksidentiteit per agent is eveneens relationeel:
  `conversations` slaat genormaliseerde provider-/account-/gespreksidentiteit op en
  `session_conversations` koppelt één OpenClaw-sessie aan een of meer externe
  gesprekken. Dit omvat gedeelde hoofdsessies voor privéberichten waarbij meerdere peers
  bewust aan één sessie kunnen worden gekoppeld zonder onjuiste gegevens in `session_key`. SQLite
  dwingt ook uniciteit af voor de natuurlijke provideridentiteit, zodat dezelfde
  combinatie van kanaal/account/type/peer/thread niet over meerdere gespreks-id's kan worden gesplitst.
  Rechtstreekse peers van de gedeelde hoofdsessie worden gekoppeld met de rol `participant`, zodat één
  OpenClaw-sessie meerdere externe peers voor privéberichten kan vertegenwoordigen zonder
  oudere peers te degraderen tot vage gerelateerde rijen. `sessions.primary_conversation_id` verwijst nog steeds
  naar het huidige getypeerde afleveringsdoel. Afgesloten routerings-/statuskolommen
  worden afgedwongen met SQLite-beperkingen `CHECK` in plaats van uitsluitend te vertrouwen op
  TypeScript-unions.
  De runtimeprojectie van sessies wist compatibiliteitsschaduwen voor routering uit
  `session_entries.entry_json` voordat getypeerde sessie-/gesprekskolommen worden
  toegepast, zodat verouderde JSON-payloads geen afleveringsdoelen opnieuw kunnen activeren.
  Voor routering van subagentmeldingen is eveneens de getypeerde SQLite-afleveringscontext vereist;
  deze valt niet langer terug op compatibele routevelden van `SessionEntry`.
  Expliciete afleveringsovererving van Gateway `chat.send` leest de getypeerde SQLite-
  afleveringscontext in plaats van de compatibiliteitsvelden `origin`/`last*`.
  `tools.effective` leidt de provider-/account-/threadcontext eveneens af uit getypeerde
  SQLite-rijen voor aflevering/routering, niet uit verouderde schaduwvelden van sessievermeldingen in `last*`.
  De promptcontext voor systeemgebeurtenissen reconstrueert kanaal-, doel-, account- en threadvelden uit
  getypeerde afleveringsvelden in plaats van schaduwvelden van `origin`.
  De gedeelde helper `deliveryContextFromSession` en de koppeling van sessie naar gesprek
  negeren `SessionEntry.origin` nu volledig; alleen getypeerde afleveringsvelden
  en relationele gespreksrijen kunnen tijdens runtime route-identiteit creëren.
  Normalisatie van runtimesessievermeldingen verwijdert `origin` voordat
  `entry_json` wordt opgeslagen of geprojecteerd, en inkomende metadata schrijft getypeerde kanaal-/chatvelden
  plus relationele gespreksrijen in plaats van nieuwe oorsprongsschaduwen
  te maken.
- Transcriptgebeurtenissen, transcripmomentopnamen en runtimegebeurtenissen van trajecten
  verwijzen nu naar de canonieke hoofdstructuur `sessions` per agent en worden trapsgewijs verwijderd wanneer de sessie
  wordt verwijderd. Rijen voor transcriptidentiteit/idempotentie worden nog steeds trapsgewijs verwijderd vanaf de
  exacte transcriptgebeurtenisrij.
- Memory-core-indexen gebruiken nu expliciete agentdatabasetabellen
  `memory_index_meta`, `memory_index_sources`, `memory_index_chunks` en
  `memory_embedding_cache`, waarbij `memory_index_state` revisiewijzigingen bijhoudt.
  Optionele FTS-/vector-nevenindexen heten `memory_index_chunks_fts` en
  `memory_index_chunks_vec` in plaats van generieke tabellen `meta`, `files`, `chunks`,
  `chunks_fts` of `chunks_vec`. De canonieke namen behouden de huidige
  rijstructuur voor pad/bron en compatibiliteit met geserialiseerde embeddings. Deze tabellen
  zijn afgeleide zoekcaches, geen canonieke transcriptopslag; ze kunnen worden
  verwijderd en opnieuw opgebouwd vanuit bestanden in de geheugenwerkruimte en geconfigureerde bronnen.
  Bij het openen van een meegeleverde geheugenindex met generieke namen worden de metadata, bronnen,
  fragmenten en embeddingcache naar de canonieke tabellen gemigreerd; afgeleide FTS-/vectortabellen
  worden opnieuw opgebouwd onder hun canonieke namen.
- De herstelstatus van subagentuitvoeringen bevindt zich nu in getypeerde gedeelde `subagent_runs`-rijen
  met geïndexeerde sessiesleutels voor de onderliggende sessie, aanvrager en controller. Het oude
  bestand `subagents/runs.json` dient uitsluitend als invoer voor opschoning door Doctor. De uitvoeringsvermeldingen daarin zijn
  tijdelijke herstelstatus, dus Doctor legt het bewijs van uitfasering vast en
  verwijdert het bestand zonder het te importeren. Omdat een bestand niet kan aantonen of
  de vermeldingen nog actief of verouderd zijn nadat SQLite-rijen zijn opgeschoond, moeten
  operators actieve uitvoeringen uit het bestandstijdperk laten afronden voordat ze over deze grens heen upgraden.
- Huidige gesprekskoppelingen bevinden zich nu in getypeerde gedeelde
  `current_conversation_bindings`-rijen met de genormaliseerde gespreks-id als sleutel, waarbij
  doelagent-/sessiekolommen, gesprekstype, status, vervaldatum en metadata
  als relationele kolommen worden opgeslagen in plaats van als een gedupliceerde ondoorzichtige koppelingsrecord.
  De duurzame koppelingssleutel bevat het genormaliseerde gesprekstype, zodat
  verwijzingen naar rechtstreekse gesprekken, groepen en kanalen niet kunnen botsen, en SQLite wijst ongeldige waarden voor
  koppelingssoort/status af. Het oude
  bestand `bindings/current-conversations.json` dient uitsluitend als invoer voor doctormigratie.
- Herstel van de afleveringswachtrij legt nu getypeerde wachtrijkolommen voor kanaal, doel,
  account, sessie, nieuwe poging, fout, platformverzending en herstelstatus over de
  JSON voor opnieuw afspelen. `entry_json` bewaart de payloads voor opnieuw afspelen, hooks en opmaakpayload,
  maar getypeerde kolommen zijn leidend voor snelle routering/status van de wachtrij.
- Herstelverwijzingen naar de laatste TUI-sessie bevinden zich nu in getypeerde gedeelde
  `tui_last_sessions`-rijen met het gehashte bereik van de TUI-verbinding/sessie als sleutel.
  De runtime leest en schrijft uitsluitend SQLite, voert voor elk bereik atomair een upsert uit en
  sluit Heartbeat-sessies uit. `openclaw doctor --fix` valideert het
  oude TUI-JSON-bestand strikt, behoudt nieuwere SQLite-rijen, verifieert het canonieke resultaat
  en verwijdert het ongewijzigde verouderde bestand in plaats van een archief achter te laten.
- Hashes voor de implementatie van Discord-opdrachten bevinden zich nu in de gedeelde SQLite-
  opslag voor pluginstatus. De runtime leest en schrijft uitsluitend exacte toepassingsgebonden sleutels. Doctor
  verwijdert het opnieuw opbouwbare verouderde bestand `discord/command-deploy-cache.json`
  zonder het te importeren, zodat bij de volgende start één canonieke afstemming wordt uitgevoerd.
- Standaard-TTS-voorkeuren bevinden zich nu in gedeelde SQLite-rijen voor pluginstatus, met sleutels onder de
  plugin `speech-core`. Het oude bestand `settings/tts.json` dient uitsluitend als invoer voor doctormigratie;
  de runtime leest of schrijft geen JSON-bestanden voor TTS-voorkeuren meer en de
  resolver voor het verouderde pad bevindt zich in de doctormigratiemodule.
- Metadata voor geheime doelen spreekt nu over stores in plaats van te doen alsof elk
  doel voor inloggegevens een configuratiebestand is. `openclaw.json` blijft de configuratiestore;
  doelen voor authenticatieprofielen gebruiken getypeerde SQLite-rijen `auth_profile_stores`, waarbij
  providerspecifieke inloggegevens als JSON-payloads worden bewaard.
- De controle op geheimen scant niet langer uitgefaseerde `auth.json`-bestanden per agent. Doctor is verantwoordelijk
  voor het waarschuwen voor, importeren en verwijderen van dat verouderde bestand.
- Helpers voor verouderde paden van authenticatieprofielen bevinden zich nu in verouderde doctor-code. Kernhelpers voor
  paden van authenticatieprofielen bieden SQLite-identiteit voor de authenticatiestore en weergavelocaties,
  geen runtimepaden `auth-profiles.json` of `auth-state.json`.
- Runtimemodules voor herstel van subagentuitvoeringen en de cache voor OpenRouter-modelmogelijkheden
  houden SQLite-lezers/-schrijvers voor momentopnamen nu gescheiden van uitsluitend voor doctor bestemde helpers voor het importeren van verouderde JSON.
  OpenRouter-mogelijkheden gebruiken de getypeerde generieke
  rijen `model_capability_cache` onder `provider_id = "openrouter"` in plaats van
  één ondoorzichtige cacheblob of een providerspecifieke hosttabel. `taskName` van een subagentuitvoering
  wordt opgeslagen in de getypeerde kolom `subagent_runs.task_name`; de
  kopie `payload_json` is data voor opnieuw afspelen/foutopsporing, niet de bron voor snelle weergave- of
  opzoekvelden.
- `src/agents/filesystem/virtual-agent-fs.sqlite.ts` implementeert een SQLite-VFS
  bovenop de tabel `vfs_entries` van de agentdatabase. Het lezen van mappen, recursief
  exporteren, verwijderen en hernoemen gebruikt geïndexeerde prefixbereiken van `(namespace, path)`
  in plaats van een volledige naamruimte te scannen of te vertrouwen op padvergelijking met `LIKE`.
- `src/agents/runtime-worker.entry.ts` maakt per uitvoering SQLite-VFS-, toolartefact-,
  uitvoeringsartefact- en bereikgebonden cacheopslag voor workers.
- Voltooiing van workspace-bootstrap, recentheid van attestatie en gegenereerde bootstrap-
  hashes bevinden zich nu in getypeerde gedeelde `workspace_setup_state`-,
  `workspace_path_aliases`-, `workspace_attestations`- en
  `workspace_generated_bootstrap_hashes`-rijen, gesleuteld op canonieke workspace-
  identiteit. Gepersistente lexicale en reële-padaliassen houden de bescherming
  tegen verdwenen workspaces stabiel nadat een geconfigureerde symlink verdwijnt;
  opnieuw toegewezen aliassen sluiten bij fouten af. De runtime leest of schrijft niet langer
  `openclaw-workspace-state.json`, `.openclaw/workspace-state.json`, `workspace-attestations/*.attested`
  in de statusmap of aangrenzende `<workspace>.attested`-
  sidecars. `openclaw doctor --fix` valideert en claimt verouderde bronnen,
  importeert ze met migratiebewijzen in SQLite, verifieert de canonieke
  rijen en verwijdert pas daarna de geclaimde bestanden.
- Het gedeelde schema reserveert een singletonrij `exec_approvals_config`, maar de
  runtime-omschakeling is nog niet uitgevoerd. TypeScript en de macOS-begeleidende app gebruiken nog
  het statusgebonden JSON-bestand en moeten gezamenlijk naar SQLite worden overgezet.
- De TypeScript-apparaatidentiteit gebruikt nu getypeerde `device_identities`-rijen, waarbij
  de import van verouderde JSON uitsluitend door Doctor buiten de runtime-eigenaar blijft. Apparaatauthenticatie
  blijft in afwachting van een gecoördineerde schema- en runtime-overschrijdende migratie
  bestandsgebaseerd; `device_auth_tokens` blijft voor die vervolgactie gereserveerd.
- De cache voor GitHub Copilot-tokenuitwisseling gebruikt de gedeelde SQLite-tabel voor Pluginstatus
  onder `github-copilot/token-cache/default`. Dit is cachestatus die eigendom is van de provider,
  en daarom wordt bewust geen hostschema-tabel toegevoegd.
- GitHub Copilot Compaction schrijft niet langer `openclaw-compaction-*.json`-
  workspacesidecars. De harnascode roept de SDK-RPC voor geschiedeniscompactie aan voor de
  gevolgde SDK-sessie, en OpenClaw bewaart duurzame sessie-/transcriptstatus in
  SQLite in plaats van compatibiliteitsmarkeringsbestanden.
- De gedeelde Swift-runtime (`OpenClawKit`) gebruikt dezelfde
  `state/openclaw.sqlite#table/device_identities`-vorm en rijsleutels voor apparaat-
  identiteit. Verouderde Apple-containerbestanden worden door de eigenaar van de Swift-migratie
  geïmporteerd, omdat TypeScript Doctor geen toegang heeft tot die containers. Swift-
  apparaatauthenticatie blijft bestandsgebaseerd voor de gecoördineerde authenticatievervolgactie.
- Android-apparaatidentiteit en gecachte apparaatauthenticatie blijven lokale appopslag. Ze
  vereisen een afzonderlijke migratie die eigendom is van Android; de SQLite-claims van de host
  beschrijven niet het huidige Android-gedrag.
- De geschiedenis van recente pakketten voor Android-meldingen gebruikt getypeerde
  `android_notification_recent_packages`-rijen. De runtime migreert of
  leest de oude CSV-sleutels van SharedPreferences niet langer.
- Het maken van een apparaatidentiteit sluit bij fouten af wanneer verouderde `identity/device.json`
  bestaat, wanneer de SQLite-identiteitsrij ongeldig is of wanneer de SQLite-identiteitsopslag
  niet kan worden geopend. Doctor importeert en verwijdert dat bestand eerst, zodat het starten van de runtime
  de koppelingsidentiteit niet ongemerkt vóór de migratie kan vervangen.
- De selectie van een apparaatidentiteit is een SQLite-rijsleutel, geen locator van een JSON-bestand. Tests
  en Gateway-helpers geven expliciete identiteitssleutels door; alleen de Doctor-migratie en de
  opstartpoort die bij fouten afsluit kennen de uitgefaseerde bestandsnaam `identity/device.json`.
- Compatibiliteit voor sessieresets bevindt zich nu in de Doctor-configuratiemigratie:
  `session.idleMinutes` wordt verplaatst naar `session.reset.idleMinutes`,
  `session.resetByType.dm` wordt verplaatst naar `session.resetByType.direct`, en het
  runtime-resetbeleid leest alleen canonieke resetsleutels.
- Compatibiliteit met verouderde configuratie bevindt zich nu onder `src/commands/doctor/`. Normale
  validatie van `readConfigFileSnapshot()` importeert geen verouderde Doctor-detectoren
  en annoteert geen verouderde problemen; `runDoctorConfigPreflight()` voegt die problemen toe voor
  reparatie/rapportage door Doctor. De Doctor-configuratiestroom importeert
  `src/commands/doctor/legacy-config.ts`, en reparatie van oude OAuth-profiel-id's bevindt zich
  onder
  `src/commands/doctor/legacy/oauth-profile-ids.ts`.
- Niet-Doctor-opdrachten voeren reparatie van verouderde configuratie niet automatisch uit. Zo
  mislukt `openclaw update --channel` nu bij ongeldige verouderde configuratie en vraagt deze
  de gebruiker Doctor uit te voeren, in plaats van ongemerkt Doctor-migratiecode te importeren.
- Webpush, APNs, Voice Wake, updatecontroles en configuratiestatus gebruiken nu getypeerde gedeelde SQLite-
  tabellen voor abonnementen, VAPID-sleutels, Node-registraties, triggerrijen,
  routeringsrijen, status van updatemeldingen en configuratiestatusitems in plaats van
  volledige ondoorzichtige JSON-blobs. Schrijfbewerkingen van Web Push en APNs voeren alleen een upsert uit op de betrokken
  primaire-sleutelrij; configuratiestatus wordt op configuratiepad afgestemd. Hun runtime-
  modules blijven gescheiden van helpers die uitsluitend voor Doctor verouderde JSON importeren.
- De APNs-runtime leest en schrijft alleen `apns_registrations`. Expliciete
  `openclaw doctor --fix` importeert strikt de uitgefaseerde
  `push/apns-registrations.json`, behoudt bestaande canonieke rijen, verifieert
  de transactie, registreert een bewijs en verwijdert de JSON die geheimen bevat.
  Nieuwe pogingen op basis van bewijzen voeren alleen opschoning uit, terwijl
  `apns_registration_tombstones` ongeldigverklaringen vóór de eerste reparatie afdekken, zodat
  verouderde relaytoekenningen of apparaattokens niet opnieuw actief kunnen worden.
- De configuratie van de Node-host gebruikt nu een getypeerde singletonrij in de gedeelde SQLite-database.
  De runtime sluit bij fouten af zolang het oude bestand `node.json` of een onderbroken claim
  aanwezig blijft; expliciete `openclaw doctor --fix` importeert en verwijdert dit strikt
  vóór normaal runtimegebruik.
- Apparaat-/Node-koppeling, kanaalkoppeling, kanaaltoelatingslijsten en bootstrapstatus
  gebruiken nu getypeerde SQLite-rijen in plaats van volledige ondoorzichtige JSON-blobs. Goedkeuringen van Pluginbindingen
  en Cron-taakstatus volgen dezelfde opsplitsing: runtimemodules bieden
  SQLite-gebaseerde bewerkingen en neutrale snapshothelpers, en schrijfbewerkingen van snapshots voor koppeling/bootstrap
  plus goedkeuring van Pluginbindingen stemmen rijen af op primaire sleutel
  in plaats van tabellen af te kappen, terwijl Doctor de oude JSON-bestanden importeert/verwijdert via
  `src/commands/doctor/legacy/*`-modules.
- Records van geïnstalleerde Plugins bevinden zich nu in de SQLite-index voor geïnstalleerde Plugins.
  Het lezen/schrijven van runtimeconfiguratie migreert of behoudt niet langer oude
  door `plugins.installs` aangemaakte configuratiegegevens; Doctor importeert die verouderde configuratievorm
  in SQLite vóór normaal runtimegebruik.
- Snapshots voor herstel van QQBot-inloggegevens bevinden zich nu in SQLite-Pluginstatus onder
  `qqbot/credential-backups`. De runtime schrijft niet langer
  `qqbot/data/credential-backup*.json`; het QQBot-Doctor-contract importeert en
  archiveert die verouderde back-upbestanden uit de actieve statusmap.
- De planning voor het herladen van de Gateway vergelijkt snapshots van de SQLite-index voor geïnstalleerde Plugins onder
  een interne `installedPluginIndex.installRecords.*`-diffnaamruimte. Runtime-
  herlaadbeslissingen verpakken die rijen niet langer in neppe `plugins.installs`-configuratie-
  objecten.
- Matrix-accountinloggegevens bevinden zich nu in SQLite-Pluginstatus. De runtime leest
  alleen die canonieke opslag; Doctor importeert, verifieert en archiveert uitgefaseerde
  `credentials/matrix/credentials*.json`-bestanden wanneer hun account kan worden herleid.
- Kernmodules voor koppeling en Cron-runtime gebruiken niet langer verouderde JSON-padbouwers.
  De verouderde SDK-helper voor koppelingspaden blijft uitsluitend als migratiecompatibiliteit bestaan;
  de Doctor-statusmigratie beheert het lezen en importeren van bestanden. Verouderde modules die eigendom zijn
  van Doctor construeren bronpaden voor `pending.json`, `paired.json`, `bootstrap.json` en
  `cron/jobs.json` uitsluitend voor importtests en migratie. Normalisatie van verouderde Cron-
  taakvormen en import van JSONL-geschiedenis bevinden zich onder
  `src/commands/doctor/cron/`; afronding van verouderde SQLite-geschiedenis vindt plaats tijdens
  het openen van de statusdatabase.
- `src/commands/doctor/legacy/runtime-state.ts` importeert verouderde JSON-status-
  bestanden, waaronder de configuratie van de Node-host, vanuit Doctor in SQLite. Nieuwe importfuncties voor verouderde bestanden
  blijven onder `src/commands/doctor/legacy/`.
- `src/commands/doctor/state-migrations.ts` importeert verouderde `sessions.json`- en
  `*.jsonl`-transcripten rechtstreeks in SQLite en verwijdert succesvol geïmporteerde bronnen. Deze
  plaatst verouderde hoofdtranscripten niet langer tijdelijk via
  `agents/<agentId>/sessions/*.jsonl` en maakt vóór de import geen canoniek JSONL-doel
  meer.
- Doctor-controles voor statusintegriteit scannen niet langer verouderde sessiemappen en
  bieden geen verwijdering van verweesde JSONL-bestanden meer aan. Verouderde transcriptbestanden zijn uitsluitend migratie-invoer,
  en de migratiestap beheert zowel de import als de verwijdering van de bron.
- De import van het verouderde sandboxregister bevindt zich onder
  `src/commands/doctor/legacy/sandbox-registry.ts`; het actieve sandboxregister
  blijft uitsluitend in SQLite lezen en schrijven.
- De reparatie voor statuscontrole/import van verouderde sessietranscripten bevindt zich onder
  `src/commands/doctor/legacy/session-transcript-health.ts`; runtime-opdrachtmodules
  bevatten niet langer parsing van JSONL-transcripten of reparatiecode voor de actieve branch.

Hoogtepunten van voltooide consolidatie/verwijdering:

- Pluginstatus gebruikt nu de gedeelde `state/openclaw.sqlite`-database. De oude
  branch-lokale `plugin-state/state.sqlite`-sidecarimporteur is verwijderd omdat
  die SQLite-indeling nooit is uitgebracht. Probe-/testhelpers rapporteren de gedeelde
  `databasePath` in plaats van een pluginspecifiek SQLite-pad voor status bloot te stellen.
- Runtimetabellen voor taken en TaskFlow bevinden zich nu in de gedeelde
  `state/openclaw.sqlite`-database in plaats van `tasks/runs.sqlite` en
  `tasks/flows/registry.sqlite`; de oude sidecarimporteurs zijn verwijderd om dezelfde reden:
  de indeling is nooit uitgebracht.
- `src/config/sessions/store.ts` heeft `storePath` niet langer nodig voor inkomende
  metadata, route-updates of het uitlezen van het tijdstip van de laatste update. Opdrachtpersistentie, het
  opschonen van CLI-sessies, de diepte van subagents, authenticatie-overschrijvingen en de sessie-identiteit
  van transcripties gebruiken rij-API's voor agents/sessies. Schrijfbewerkingen worden toegepast als patches op SQLite-rijen
  met optimistische herhaling bij conflicten.
- Het oplossen van sessiedoelen stelt nu databasebestemmingen per agent beschikbaar, niet verouderde
  `sessions.json`-paden. De gedeelde Gateway, ACP-metadata, routereparatie door doctor en
  `openclaw sessions` inventariseren `agent_databases` plus geconfigureerde agents.
- Sessieroutering van de Gateway gebruikt nu `resolveGatewaySessionDatabaseTarget`; het
  geretourneerde doel bevat `databasePath` en mogelijke SQLite-rijsleutels in plaats
  van een verouderd bestandspad naar de sessieopslag.
- Runtimetypen voor kanaalsessies stellen nu `{agentId, sessionKey}` beschikbaar voor
  het uitlezen van het tijdstip van de laatste update, inkomende metadata en updates van de laatste route. Het oude
  compatibiliteitstype `saveSessionStore(storePath, store)` is verdwenen.
- Sessieoppervlakken van de Plugin-runtime, extensie-API en Plugin-SDK stellen nu
  SQLite-ondersteunde helpers voor sessierijen beschikbaar in plaats van compatibiliteitshelpers
  voor volledige opslag/bestanden van actieve sessies. Compatibiliteitsexports van de hoofdbibliotheek blijven
  alleen buiten de Plugin-SDK beschikbaar voor verouderde interne aanroepers en migratieaanroepers. De oude
  helper `resolveLegacySessionStorePath` is verdwenen; de constructie van verouderde `sessions.json`-paden
  is nu lokaal beperkt tot migratie- en testfixtures.
- `src/config/sessions/session-entries.sqlite.ts` slaat canonieke sessie-items nu op
  in de database per agent en ondersteunt lezen, upsert, verwijderen en patchen op rijniveau.
  Runtime-upserts/-patches/-verwijderingen scannen niet langer op varianten in hoofdlettergebruik en
  verwijderen geen verouderde aliassleutels meer; doctor beheert de canonicalisatie. De
  zelfstandige JSON-importhelper is verdwenen en migratie voegt nieuwere rijen met upsert samen
  in plaats van de volledige sessietabel te vervangen. Openbare helpers voor lezen/inventariseren/laden
  projecteren veelgebruikte sessiemetadata uit getypeerde rijen `sessions` en `conversations`;
  `entry_json` is een compatibiliteits-/debugschaduw en kan verouderd of ongeldig zijn
  zonder verlies van de getypeerde sessie-identiteit of leveringscontext.
- `src/config/sessions/delivery-info.ts` bepaalt de leveringscontext nu uit de
  getypeerde `sessions`- + `conversations`- + `session_conversations`-rijen per agent.
  De leveringsidentiteit van de runtime wordt niet langer gereconstrueerd uit
  `session_entries.entry_json`; een ontbrekende getypeerde gespreksrij is een probleem voor
  migratie/reparatie door doctor, geen runtimefallback.
- Beslissingen over het opnieuw instellen van opgeslagen sessies geven nu de voorkeur aan getypeerde metadata
  `sessions.session_scope`, `sessions.chat_type` en `sessions.channel`. Het parseren van `sessionKey`
  blijft alleen bestaan voor expliciete thread-/onderwerpsachtervoegsels bij opdrachtdoelen; de classificatie
  voor groepsgewijs versus rechtstreeks opnieuw instellen wordt niet langer uit de sleutelvorm afgeleid.
- De weergaveclassificatie van sessielijsten/-statussen gebruikt nu getypeerde chatmetadata en
  het Gateway-sessietype. Subtekenreeksen `:group:` of `:channel:`
  binnen `session_key` worden niet langer beschouwd als blijvende waarheid over groep/rechtstreeks.
- De selectie van beleid voor stille antwoorden gebruikt nu alleen het expliciete gesprekstype of
  expliciete oppervlaktedata. Beleid voor rechtstreeks/groep wordt niet langer geraden op basis van
  subtekenreeksen in `session_key`.
- Het oplossen van het weergavemodel voor sessies ontvangt de agent-id nu van het
  SQLite-databasedoel voor de sessie in plaats van deze uit `session_key` te splitsen.
- Het aanvullen van aankondigingsdoelen tussen agents gebruikt nu alleen de getypeerde
  `sessions.list` `deliveryContext`. Routering voor kanaal/account/thread wordt niet langer
  hersteld uit de verouderde `origin`, gespiegelde `last*`-velden of de vorm van `session_key`.
- De afwijzing van thread-doelen door `sessions_send` leest nu getypeerde SQLite-routeringsmetadata.
  Doelen worden niet langer afgewezen of geaccepteerd door threadachtervoegsels
  uit de doelsleutel te parseren.
- Validatie van groepsgewijs toolbeleid leest nu getypeerde SQLite-gespreksroutering
  voor de huidige of voortgebrachte sessie. De groeps-/kanaalidentiteit wordt niet langer vertrouwd
  door `sessionKey` te decoderen; door de aanroeper verstrekte groeps-id's worden verwijderd wanneer
  geen getypeerde sessierij deze bevestigt.
- Het matchen van kanaalspecifieke modeloverschrijvingen gebruikt nu expliciete metadata
  van het groeps- en bovenliggende gesprek. Id's van bovenliggende gesprekken worden niet langer
  uit `parentSessionKey` gedecodeerd.
- Overerving van opgeslagen modeloverschrijvingen vereist nu een expliciete sleutel van de bovenliggende sessie
  uit getypeerde sessiecontext. Bovenliggende overschrijvingen worden niet langer afgeleid uit
  de achtervoegsels `:thread:` of `:topic:` in `sessionKey`.
- De oude wrapper voor sessie-threadinformatie en de threadparser voor geladen Plugins zijn verdwenen;
  geen runtimecode importeert nog `config/sessions/thread-info`.
- De helper voor kanaalgesprekken stelt niet langer parseerbruggen voor volledige sessiesleutels
  beschikbaar. Core normaliseert nog steeds onbewerkte gespreks-id's van providers via
  `resolveSessionConversation(...)`, maar reconstrueert geen routegegevens
  uit `sessionKey`.
- Levering na voltooiing, verzendbeleid en taakonderhoud leiden het chattype
  niet langer af uit de vorm van `session_key`. De oude parser voor chattypesleutels is verwijderd;
  deze paden vereisen getypeerde sessiemetadata, een getypeerde leveringscontext of
  expliciete terminologie voor leveringsdoelen.
- Sessielijst/-status, diagnostiek, koppeling van goedkeuringsaccounts, Heartbeat-filtering
  in de TUI en gebruiksoverzichten doorzoeken `SessionEntry.origin` niet langer op
  routering voor provider/account/thread/weergave. De enige resterende runtime-uitlezingen
  van `origin` betreffen niet-sessieconcepten of leveringsobjecten van de huidige beurt.
- Het opzoeken van native gesprekken voor goedkeuringsverzoeken leest nu getypeerde routeringsrijen
  voor sessies per agent. De gespreksidentiteit voor kanaal/groep/thread wordt niet langer
  uit `sessionKey` geparseerd; ontbrekende getypeerde metadata is een migratie-/reparatieprobleem.
- Eventpayloads van de Gateway voor gewijzigde sessies/chat/sessies herhalen niet langer
  de routeschaduwen `SessionEntry.origin` of `last*`; clients ontvangen getypeerde
  `channel`, `chatType` en `deliveryContext`.
- Het oplossen van Heartbeat-levering kan de getypeerde SQLite-
  `deliveryContext` nu rechtstreeks ontvangen en de Heartbeat-runtime geeft de leveringsrij
  van de sessie per agent door in plaats van voor de huidige routering te vertrouwen op compatibiliteitsschaduwen
  van `session_entries`.
- Het oplossen van leveringsdoelen voor geïsoleerde Cron-agents vult de huidige
  route eveneens eerst aan vanuit de getypeerde leveringsrij van de sessie per agent, voordat wordt teruggevallen op de
  compatibiliteitspayload van het item.
- Het oplossen van de oorsprong van subagentaankondigingen voert de getypeerde leveringscontext
  van de aanvragende sessie nu door via `loadRequesterSessionEntry` en geeft de voorkeur aan die rij boven
  compatibiliteitsschaduwen van `last*`/`deliveryContext`.
- Updates van inkomende sessiemetadata worden nu eerst samengevoegd met de getypeerde leveringsrij
  per agent; oude leveringsvelden van `SessionEntry` dienen alleen als fallback
  wanneer er geen getypeerde gespreksrij bestaat.
- Bij het extraheren van levering voor herstarten/bijwerken krijgt de getypeerde SQLite-leverings-
  `threadId` nu voorrang op onderwerp-/threadfragmenten die uit `sessionKey` zijn geparseerd; parseren
  is alleen een fallback voor verouderde sleutels met een threadvorm.
- Kanaal-id's in de agentcontext van hooks geven nu de voorkeur aan getypeerde SQLite-gespreksidentiteit
  en daarna aan expliciete berichtmetadata. Provider-/groep-/kanaalfragmenten
  worden niet langer uit `sessionKey` geparseerd.
- Overerving van externe routes door Gateway `chat.send` leest nu getypeerde SQLite-routeringsmetadata
  voor sessies in plaats van kanaal-/rechtstreeks-/groepsbereik af te leiden uit
  delen van `sessionKey`. Kanaalspecifieke sessies erven alleen wanneer het getypeerde
  sessiekanaal en chattype overeenkomen met de opgeslagen leveringscontext; gedeelde hoofdsessies
  behouden hun strengere regel voor CLI/geen clientmetadata.
- Activering door herstartsentinels en vervolgroutering lezen nu getypeerde SQLite-
  leverings-/routeringsrijen voordat Heartbeat-activeringen of gerouteerde voortzettingen
  van agentbeurten in de wachtrij worden geplaatst. De leveringscontext wordt niet langer gereconstrueerd uit de
  JSON-schaduw van het sessie-item.
- Contextresolutie van Gateway `tools.effective` leest nu getypeerde SQLite-
  leverings-/routeringsrijen voor invoerwaarden voor provider, account, doel, thread en antwoordmodus.
  Deze veelgebruikte routeringsvelden worden niet langer hersteld uit verouderde
  oorsprongsschaduwen van `session_entries.entry_json`.
- Routering voor realtime spraakconsultatie bepaalt de levering van bovenliggende sessies/oproepen nu uit getypeerde
  SQLite-sessierijen per agent. Er wordt niet langer teruggevallen op compatibiliteitsschaduwen
  van `SessionEntry.deliveryContext` bij het kiezen van de berichtroute voor de ingebedde agent.
- Heartbeat-doorgifte bij ACP-spawning en routering van de bovenliggende stream lezen de levering
  van de bovenliggende sessie nu uit getypeerde SQLite-sessierijen. De leveringscontext van de bovenliggende sessie
  wordt niet langer gereconstrueerd uit compatibiliteitsschaduwen van sessie-items.
- Het behouden van leveringsroutes voor sessies volgt nu getypeerde chatmetadata en
  persistente leveringskolommen. Kanaalhints, rechtstreeks-/hoofdmarkeringen
  en threadvorm worden niet langer uit `sessionKey` geëxtraheerd; interne webchatroutes
  erven alleen een extern doel wanneer SQLite al getypeerde/persistente leveringsidentiteit
  voor de sessie bevat.
- Generieke extractie van sessielevering leest nu alleen de exacte getypeerde SQLite-
  leveringsrij voor de sessie. Thread-/onderwerpsachtervoegsels worden niet langer geparseerd en er wordt niet meer teruggevallen
  van een sleutel met threadvorm naar een basissessiesleutel.
- Antwoordverzending, herstel door herstartsentinels en routering voor realtime spraakconsultatie
  gebruiken nu exacte getypeerde SQLite-sessie-/gespreksrijen voor threadroutering. Ze
  herstellen geen thread-id's of leveringscontext van basissessies meer door
  sessiesleutels met threadvorm te parseren.
- Geschiedenisbeperking voor ingebedde PI gebruikt nu de getypeerde SQLite-projectie
  voor sessieroutering (`sessions` + primaire `conversations`) voor provider, chattype
  en peer-identiteit. Provider-, DM-, groeps- of threadvorm wordt niet langer
  uit `sessionKey` geparseerd.
- Het afleiden van levering door de Cron-tool gebruikt nu alleen expliciete levering of de huidige getypeerde
  leveringscontext. Kanaal-, peer-, account- of threaddoelen worden niet langer
  uit `agentSessionKey` gedecodeerd.
- Runtime-sessierijen bevatten niet langer de oude routealias `lastProvider`.
  Helpers en tests gebruiken getypeerde velden `lastChannel` en `deliveryContext`;
  doctormigratie is de enige plaats waar oudere routealiassen
  of persistente schaduwen van `origin` mogen worden vertaald.
- Transcriptie-events, VFS-rijen en rijen voor toolartefacten worden nu naar de database
  per agent geschreven. De nooit uitgebrachte globale toewijzingstabel voor transcriptiebestanden is verdwenen; doctor
  legt verouderde bronpaden voortaan vast in duurzame migratierijen.
- Runtime-opzoekingen van transcripties scannen niet langer JSONL-byteoffsets en controleren geen verouderde
  transcriptiebestanden meer. Gateway-paden voor chat/media/geschiedenis lezen transcriptierijen uit
  SQLite; sessie-JSONL is nu alleen nog verouderde invoer voor doctor, geen runtime-status
  of exportindeling.
- Bovenliggende en vertakkingsrelaties van transcripties gebruiken gestructureerde
  `parentTranscriptScope: {agentId, sessionId}`-metadata in SQLite-transcriptieheaders,
  geen padachtige `agent-db:...transcript_events...`-locatorreeksen.
- Het contract van de transcriptiemanager stelt niet langer impliciete persistente
  constructors `create(cwd)` of `continueRecent(cwd)` beschikbaar. Persistente transcriptie-
  managers worden geopend met een expliciet `{agentId, sessionId}`-bereik; alleen
  in-memory managers blijven vrij van scope voor tests en pure transcripttransformaties.
- API's voor runtime-transcriptopslag bepalen de SQLite-scope, niet bestandssysteempaden. De
  oude helper `resolve...ForPath` en ongebruikte schrijfopties van `transcriptPath` zijn
  verwijderd uit runtime-aanroepers.
- Runtime-sessiebepaling gebruikt nu `{agentId, sessionId}` en mag geen
  `sqlite-transcript://<agent>/<session>`-tekenreeksen afleiden voor externe grenzen.
  Verouderde absolute JSONL-paden dienen alleen als invoer voor doctormigratie.
- Direct-bridge-records van de native-hookrelay bevinden zich nu in getypeerde gedeelde
  `native_hook_relay_bridges`-rijen met de relay-id als sleutel. De runtime schrijft niet langer een
  `/tmp`-JSON-register of ondoorzichtige generieke records voor die kortstondige bridge-
  records.
- `runEmbeddedPiAgent(...)` heeft geen transcriptlocatorparameter meer.
  Voorbereide workerdescriptors bevatten evenmin transcriptlocators. Runtime-sessie-
  status en in de wachtrij geplaatste vervolgruns bevatten `{agentId, sessionId}` in plaats van
  afgeleide transcripthandles.
- Ingebedde Compaction ontvangt nu SQLite-scope van `agentId` en `sessionId`.
  Compaction-hooks, context-engine-aanroepen, CLI-delegatie en protocolantwoorden
  mogen geen afgeleide `sqlite-transcript://...`-handles ontvangen. Export-/debugcode
  kan expliciete gebruikersartefacten uit rijen materialiseren, maar biedt geen
  generiek exportpad voor sessie-JSONL en voert bestandsnamen niet terug naar de runtime-
  identiteit.
- `/export-session` leest transcriptrijen uit SQLite en schrijft alleen de gevraagde
  zelfstandige HTML-weergave. De ingebedde viewer reconstrueert of
  downloadt niet langer sessie-JSONL uit die rijen.
- Context-engine-delegatie parseert niet langer een transcriptlocator om de
  agentidentiteit te achterhalen. De voorbereide runtimecontext geeft de bepaalde `agentId`
  door aan de ingebouwde Compaction-adapter.
- Het herschrijven van transcripten en live afkappen van toolresultaten lezen en bewaren
  transcriptstatus nu via `{agentId, sessionId}` en leiden geen tijdelijke
  locators af voor de payloads van transcriptupdate-events.
- Het helperoppervlak voor transcriptstatus heeft niet langer locatorgebaseerde
  varianten van `readTranscriptState`, `replaceTranscriptStateEvents` of
  `persistTranscriptStateMutation`. Runtime-aanroepers moeten de
  `{agentId, sessionId}`-API's gebruiken. Doctor-import leest verouderde bestanden via een expliciet bestands-
  pad en schrijft SQLite-rijen; locator-tekenreeksen worden niet gemigreerd.
- Het runtimecontract van de sessiemanager stelt `open(locator)`,
  `forkFrom(locator)` of `setTranscriptLocator(...)` niet langer beschikbaar. Persistente sessie-
  managers openen alleen via `{agentId, sessionId}`; helpers voor weergeven/forken bevinden zich in
  rijgeoriënteerde sessie- en checkpoint-API's in plaats van in de facade van de transcript-
  manager.
- API's voor de Gateway-transcriptlezer stellen scope voorop. Ze ontvangen
  `{agentId, sessionId}` en accepteren geen positionele transcriptlocator die
  per ongeluk runtime-identiteit kan worden. Het parsen van actieve transcriptlocators
  is verwijderd; verouderde bronpaden worden alleen door doctor-importcode gelezen.
- Transcriptupdate-events stellen eveneens scope voorop. `emitSessionTranscriptUpdate`
  accepteert niet langer een losse locator-tekenreeks en listeners routeren via
  `{agentId, sessionId}` zonder een handle te parsen.
- De broadcast van Gateway-sessieberichten bepaalt sessiesleutels vanuit de agent-/sessie-
  scope, niet vanuit een transcriptlocator. De oude resolver/cache van transcriptlocator naar sessie-
  sleutel is verwijderd.
- SSE-filters voor Gateway-sessiegeschiedenis filteren live-updates op agent-/sessiescope. Ze
  canonicaliseren niet langer mogelijke transcriptlocators, realpaths of bestandsvormige
  transcriptidentiteiten om te bepalen of een stream een update moet ontvangen.
- Hooks voor de sessielevenscyclus leiden niet langer transcriptlocators af en stellen deze niet meer beschikbaar op
  `session_end`. Hookgebruikers ontvangen `sessionId`, `sessionKey`, ids van volgende sessies
  en agentcontext; transcriptbestanden maken geen deel uit van het levenscyclus-
  contract.
- Resethooks leiden evenmin transcriptlocators af of stellen deze beschikbaar. De
  `before_reset`-payload bevat herstelde SQLite-berichten plus de reset-
  reden, terwijl de sessie-identiteit in de hookcontext blijft.
- De reset van het agentharnas accepteert niet langer een transcriptlocator. Resetdispatch is
  beperkt tot `sessionId`/`sessionKey` plus de reden.
- Sessietypen voor agentextensies stellen `transcriptLocator` niet langer beschikbaar; extensies
  moeten sessiecontext en runtime-API's gebruiken in plaats van rechtstreeks een
  bestandsvormige transcriptidentiteit te benaderen.
- Plugin-hooks voor Compaction stellen transcriptlocators niet langer beschikbaar. De hookcontext
  bevat de sessie-identiteit al en transcripten moeten worden gelezen via SQLite-
  scopebewuste API's in plaats van bestandsvormige handles.
- `before_agent_finalize`-hooks stellen `transcriptPath` niet langer beschikbaar, ook niet in
  payloads van de native-hookrelay. Finalisatiehooks gebruiken alleen sessiecontext.
- Gateway-resetantwoorden synthetiseren niet langer een transcriptlocator op het
  geretourneerde item. De reset maakt SQLite-transcriptrijen, retourneert het opgeschoonde
  sessie-item en laat transcripttoegang over aan scopebewuste lezers.
- Resultaten van ingebedde runs en Compaction tonen niet langer transcriptlocators voor
  sessieadministratie. Automatische Compaction werkt alleen de actieve `sessionId`,
  Compaction-tellers en tokenmetadata bij.
- Resultaten van ingebedde pogingen retourneren `transcriptLocatorUsed` niet langer en
  `compact()`-resultaten van de context-engine retourneren geen transcriptlocators meer.
  Runtime-herhaallussen accepteren alleen een opvolgende `sessionId`.
- Resultaten van het toevoegen aan het delivery-mirror-transcript retourneren niet langer transcript-
  locators. Aanroepers ontvangen de toegevoegde `messageId`; transcriptupdatesignalen gebruiken
  SQLite-scope.
- Forkhelpers voor bovenliggende sessies retourneren alleen de geforkte `sessionId`. De voorbereiding van subagents
  geeft de scope van de onderliggende agent/sessie door aan engines.
- Parameters van de CLI-runner en het opnieuw vullen van geschiedenis accepteren niet langer transcriptlocators.
  Het lezen van CLI-geschiedenis bepaalt de SQLite-transcriptscope vanuit `{agentId,
sessionId}` en de context van de sessiesleutel.
- Testfixtures voor de CLI en ingebedde runner vullen en lezen SQLite-transcriptrijen nu
  via de sessie-id in plaats van te doen alsof actieve sessies `*.jsonl`-bestanden zijn of
  een `sqlite-transcript://...`-tekenreeks via runtimeparameters door te geven.
- Events van de sessietoolresultaatbeveiliging worden vanuit de bekende sessiescope verzonden, zelfs wanneer een
  in-memory manager geen afgeleide locator heeft. De tests simuleren niet langer actieve
  `/tmp/*.jsonl`-transcriptbestanden.
- BTW- en Compaction-checkpointhelpers lezen en forken transcriptrijen nu via
  SQLite-scope. Checkpointmetadata bewaart nu alleen sessie-id's en leaf-/item-id's;
  afgeleide locators worden niet langer naar checkpointpayloads geschreven.
- Gateway-transcriptsleutelzoekopdrachten gebruiken SQLite-transcriptscope bij protocol-
  grenzen en voeren niet langer realpath of stat uit op transcriptbestandsnamen.
- Automatische transcriptrotatie bij Compaction schrijft opvolgende transcriptrijen
  rechtstreeks via de SQLite-transcriptopslag. Sessierijen bewaren alleen de
  opvolgende sessie-identiteit, niet een duurzaam JSONL-pad of persistente locator.
- Ingebedde context-engine-Compaction gebruikt naar SQLite genoemde helpers voor transcriptrotatie.
  De rotatietests construeren niet langer JSONL-opvolgerpaden en modelleren
  actieve sessies niet meer als bestanden.
- Beheerde retentie van uitgaande afbeeldingen baseert de sleutel van de cache voor transcriptberichten op
  SQLite-transcriptstatistieken in plaats van stat-aanroepen naar het bestandssysteem.
- Runtime-sessievergrendelingen en de zelfstandige verouderde `.jsonl.lock`-doctor-
  lane zijn verwijderd.
- De runtimebarrel van Microsoft Teams en de openbare Plugin-SDK exporteren
  de oude helper voor bestandsvergrendeling niet langer opnieuw; duurzame paden voor Plugin-status worden door SQLite ondersteund.
- Opschoning op sessieleeftijd/-aantal en expliciete sessieopschoning zijn verwijderd.
  Doctor beheert verouderde import; verlopen sessies worden expliciet gereset of verwijderd.
- Doctor-integriteitscontroles tellen een verouderd JSONL-bestand niet langer als geldig actief
  transcript voor een SQLite-sessierij. De gezondheid van actieve transcripten is uitsluitend op SQLite gebaseerd;
  verouderde JSONL-bestanden worden gerapporteerd als invoer voor migratie/weesbestandsopschoning.
- Doctor beschouwt `agents/<agent>/sessions/` niet langer als vereiste runtime-
  status. Die map wordt alleen gescand als deze al bestaat, als invoer voor verouderde import
  of weesbestandsopschoning.
- Gateway-`sessions.resolve`, paden voor sessiepatch/reset/Compaction, het starten van subagents,
  snel afbreken, ACP-metadata, door Heartbeat geïsoleerde sessies en TUI-
  patching migreren of verwijderen niet langer verouderde sessiesleutels als neveneffect van
  normale runtimewerkzaamheden.
- CLI-opdrachtsessiebepaling retourneert nu de bijbehorende `agentId` in plaats van een
  `storePath` en kopieert niet langer verouderde rijen van de hoofdsessie tijdens normale
  bepaling van `--to` of `--session-id`. Canonicalisatie van verouderde hoofdrijen hoort
  uitsluitend bij doctor.
- De runtimebepaling van subagentdiepte leest `sessions.json` of JSON5-
  sessieopslag niet langer. Deze leest SQLite-`session_entries` via agent-id en verouderde
  diepte-/sessiemetadata kan alleen binnenkomen via het doctor-importpad.
- Overschrijvingen van auth-profielsessies worden bewaard via rechtstreekse upserts van
  `{agentId, sessionKey}`-rijen in plaats van een bestandsvormige sessieopslagruntime lazy te laden.
- Verbose-filtering voor automatische antwoorden en helpers voor sessie-updates lezen/upserten nu SQLite-
  sessierijen via sessie-identiteit en vereisen niet langer een verouderd opslagpad
  voordat ze persistente rijstatus aanpassen.
- Helpers voor sessiemetadata van opdrachtruns gebruiken nu itemgeoriënteerde namen en module-
  paden; het oude opdracht-helperoppervlak van `session-store` is verwijderd.
- Het vullen van bootstrapheaders en versterken van de grens voor handmatige Compaction wijzigen nu
  rechtstreeks SQLite-transcriptrijen. Runtime-aanroepers geven sessie-identiteit door, geen
  beschrijfbare `.jsonl`-paden.
- Stille replay bij sessierotatie kopieert recente beurten van gebruiker/assistent via
  `{agentId, sessionId}` uit SQLite-transcriptrijen. Deze accepteert niet langer
  bron- of doeltranscriptlocators.
- Nieuwe runtime-sessierijen slaan niet langer transcriptlocators op. Aanroepers gebruiken
  `{agentId, sessionId}` rechtstreeks; export-/debugopdrachten kunnen uitvoerbestands-
  namen kiezen wanneer ze rijen materialiseren.
- Het starten van een nieuwe persistente transcriptsessie opent nu altijd SQLite-rijen via
  scope. De sessiemanager hergebruikt niet langer een eerder transcriptpad
  of locator uit het bestandstijdperk als identiteit voor de nieuwe sessie.
- Persistente transcriptsessies gebruiken de expliciete
  `openTranscriptSessionManagerForSession({agentId, sessionId})`-API. De oude statische
  `SessionManager.create/openForSession/list/forkFromSession`-facades zijn
  verwijderd, zodat tests en runtimecode niet per ongeluk sessiedetectie uit het bestandstijdperk
  opnieuw kunnen creëren.
- De Plugin-runtime stelt `api.runtime.agent.session.resolveTranscriptLocatorPath` niet langer beschikbaar;
  Plugincode gebruikt SQLite-rijhelpers en scopewaarden.
- Het openbare `session-store-runtime`-SDK-oppervlak exporteert nu alleen helpers voor sessierijen
  en transcriptrijen. Gerichte helpers voor SQLite-schema/pad/transactie
  bevinden zich in `sqlite-runtime`; ruwe helpers voor openen/sluiten/resetten blijven uitsluitend lokaal voor
  tests van de eerste partij.
- Verouderde `.jsonl`-classificaties voor bestandsnamen van trajecten/checkpoints bevinden zich nu in de
  verouderde sessiebestandsmodule van doctor. Kernvalidatie van sessies importeert niet langer
  helpers voor bestandsartefacten om normale SQLite-sessie-id's te bepalen.
- Blokkerende Active Memory-subagentruns gebruiken SQLite-transcriptrijen in plaats van
  tijdelijke of persistente `session.jsonl`-bestanden onder Plugin-status te maken. De
  oude optie `transcriptDir` is verwijderd.
- Eenmalige sluggeneratie en planner-runs van systeemagents gebruiken SQLite-transcriptrijen
  in plaats van tijdelijke `session.jsonl`-bestanden te maken.
- `llm-task`-hulpruns en extractie van verborgen toezeggingen gebruiken ook SQLite-
  transcriptierijen, zodat deze helpersessies die alleen voor het model dienen geen
  tijdelijke JSON/JSONL-transcriptiebestanden meer maken.
- `TranscriptSessionManager` is nu alleen een geopend SQLite-transcriptiebereik.
  Runtimecode opent dit met `openTranscriptSessionManagerForSession({agentId,
sessionId})`; flows voor maken, vertakken, voortzetten, weergeven en forken bevinden zich in de
  bijbehorende SQLite-rijhelpers in plaats van statische beheerfacades.
  Doctor-/import-/debugcode verwerkt expliciete verouderde bronbestanden buiten de
  runtimesessiebeheerder.
- De verouderde facademethoden `SessionManager.newSession()` en
  `SessionManager.createBranchedSession()` zijn verwijderd. Nieuwe
  sessies en afstammelingen van transcripties worden door hun bijbehorende SQLite-
  workflow gemaakt, in plaats van een reeds geopende beheerder te muteren tot een andere
  persistente sessie.
- Beslissingen over het forken van bovenliggende transcripties en het maken van forks accepteren
  `storePath` of `sessionsDir` niet meer; ze gebruiken het SQLite-
  transcriptiebereik `{agentId, sessionId}` in plaats van bewaarde metagegevens over bestandssysteempaden.
- Memory-host exporteert niet langer no-op helpers voor transcriptieclassificatie
  van sessiemappen; transcriptiefiltering wordt nu tijdens het opbouwen van vermeldingen afgeleid uit metagegevens van SQLite-
  rijen.
- Memory-host- en QMD-tests voor sessie-export gebruiken SQLite-transcriptiebereiken. Oude
  `agents/<agentId>/sessions/*.jsonl`-paden blijven alleen gedekt waar een test
  opzettelijk compatibiliteit met doctor/import/export bewijst.
- Ruwe sessie-inspectie in QA-lab gebruikt nu `sessions.list` via de Gateway
  in plaats van `agents/qa/sessions/sessions.json` te lezen; MSteams-feedback
  wordt rechtstreeks aan SQLite-transcripties toegevoegd zonder een JSONL-pad te verzinnen.
- Gedeelde inkomende kanaalbeurten bevatten nu `{agentId, sessionKey}` in plaats van een
  verouderde `storePath`. Registratiepaden voor LINE, WhatsApp, Slack, Discord, Telegram, Matrix, Signal,
  iMessage, BlueBubbles, Feishu, Google Chat, IRC, Nextcloud Talk, Zalo,
  Zalo Personal, QA Channel, Microsoft Teams, Mattermost, Synology Chat, Tlon,
  Twitch en QQBot lezen nu metagegevens over de bijwerkingstijd en registreren
  inkomende sessierijen via de SQLite-identiteit.
- Persistentie van transcriptielocators is verwijderd uit actieve sessierijen.
  `resolveSessionTranscriptTarget` retourneert `agentId`, `sessionId` en optionele
  onderwerpmetagegevens; doctor is de enige code die verouderde namen van transcriptiebestanden
  importeert.
- Runtime-transcriptiekoppen beginnen bij SQLite-versie `1`. Upgrades van oude JSONL V1/V2/V3-
  structuren bevinden zich alleen in doctor-import en normaliseren geïmporteerde koppen naar
  de huidige SQLite-transcriptieversie voordat rijen worden opgeslagen.
- De database-first-beveiliging verbiedt nu `SessionManager.listAll` en
  `SessionManager.forkFromSession`; workflows voor sessieweergave en fork/herstel
  moeten SQLite-API's op rij-/bereikniveau blijven gebruiken.
- De beveiliging verbiedt buiten doctor-/importcode ook namen van verouderde helpers voor het parseren van transcriptie-JSONL en
  herstel van actieve vertakkingen, zodat de runtime geen tweede verouderd
  migratiepad voor transcripties kan krijgen.
- Ingebedde PI-runs weigeren inkomende transcriptiehandles. Ze gebruiken de SQLite-
  identiteit `{agentId, sessionId}` vóór het starten van de worker en opnieuw voordat de
  poging de transcriptiestatus aanraakt. Verouderde `/tmp/*.jsonl`-invoer kan geen
  schrijfdoel voor de runtime selecteren.
- Registraties voor cachetracering, Anthropic-payloads, ruwe streams en diagnostische tijdlijnen
  worden nu naar getypeerde SQLite-rijen van `diagnostic_events` geschreven. Gateway-stabiliteitsbundels
  worden nu naar getypeerde SQLite-rijen van `diagnostic_stability_bundles` geschreven. De oude
  JSONL-overschrijvingspaden `diagnostics.cacheTrace.filePath`, `OPENCLAW_CACHE_TRACE_FILE`,
  `OPENCLAW_ANTHROPIC_PAYLOAD_LOG_FILE` en
  `OPENCLAW_DIAGNOSTICS_TIMELINE_PATH` zijn verwijderd en
  normale stabiliteitsregistratie schrijft geen `logs/stability/*.json`-bestanden meer.
- Cron-persistentie synchroniseert nu SQLite-rijen van `cron_jobs` in plaats van
  bij elke opslag de volledige taaktabel te verwijderen en opnieuw in te voegen. Terugschrijfbewerkingen voor Plugin-doelen
  werken overeenkomende Cron-rijen rechtstreeks bij en houden de Cron-runtimestatus binnen
  dezelfde statusdatabasetransactie.
- Cron-runtimeaanroepers gebruiken nu een stabiele SQLite-sleutel voor de Cron-opslag. Verouderde
  `cron.store`-paden dienen alleen als invoer voor doctor-import; productie-Gateway, taakonderhoud,
  status, uitvoeringsgeschiedenis en terugschrijfpaden voor Telegram-doelen gebruiken
  `resolveCronStoreKey` en normaliseren de sleutel niet langer als pad. De Cron-status
  rapporteert nu `storeKey` in plaats van het oude, bestandsvormige veld `storePath`.
- Het laden en plannen van de Cron-runtime normaliseert niet langer verouderde persistente taakstructuren,
  zoals `jobId`, `schedule.cron`, numerieke `atMs`, booleans als tekenreeksen of
  ontbrekende `sessionTarget`. De import van verouderde gegevens door doctor beheert die reparaties voordat rijen
  in SQLite worden ingevoegd.
- ACP-spawn lost geen paden naar transcriptie-JSONL-bestanden meer op en bewaart ze niet meer persistent. De configuratie voor spawn
  en threadbinding bewaart de SQLite-sessierij rechtstreeks persistent en behoudt de
  sessie-id als transcriptie-identiteit.
- API's voor ACP-sessiemetagegevens lezen/vermelden/upserten SQLite-rijen nu op `agentId` en
  stellen `storePath` niet langer beschikbaar als onderdeel van het contract voor ACP-sessievermeldingen.
- De boekhouding van sessiegebruik en aggregatie van Gateway-gebruik lossen transcripties nu
  uitsluitend op via `{agentId, sessionId}`. De kosten-/gebruikscache en samenvattingen van ontdekte sessies
  maken of retourneren geen transcriptielocatortekenreeksen meer.
- Gateway-chattoevoeging, persistentie van gedeeltelijke resultaten bij afbreken, `/sessions.send` en
  het schrijven van webchatmedia naar transcripties voegen rechtstreeks toe via het SQLite-transcriptiebereik.
  De Gateway-helper voor transcriptie-injectie accepteert niet langer een
  parameter `transcriptLocator`.
- SQLite-transcriptiedetectie vermeldt nu alleen transcriptiebereiken en statistieken:
  `{agentId, sessionId, updatedAt, eventCount}`. De ongebruikte
  compatibiliteitshelper `listSqliteSessionTranscriptLocators` en het veld
  `locator` per rij zijn verwijderd.
- De runtime voor transcriptieherstel stelt nu alleen
  `repairTranscriptSessionStateIfNeeded({agentId, sessionId})` beschikbaar. De oude
  locatorgebaseerde herstelhelper is verwijderd; doctor-/debugcode leest expliciete
  bronbestandspaden en migreert nooit locatortekenreeksen.
- De runtime voor het ACP-replaylogboek bewaart replayrijen per sessie nu in de gedeelde
  SQLite-statusdatabase in plaats van `acp/event-ledger.json`; doctor importeert en
  verwijdert het verouderde bestand.
- Gateway-helpers voor het lezen van transcripties bevinden zich nu in
  `src/gateway/session-transcript-readers.ts` in plaats van onder de oude
  modulenaam `session-utils.fs`. De controle van de fallback-pogingsgeschiedenis is genoemd naar
  SQLite-transcriptie-inhoud in plaats van het oude oppervlak van de bestandshelper.
- Gateway-helpers voor geïnjecteerde chats en Compaction geven het SQLite-transcriptiebereik nu
  door via interne helper-API's, in plaats van waarden transcriptiepaden of
  bronbestanden te noemen.
- Detectie van bootstrap-voortzetting controleert SQLite-transcriptierijen nu via
  `hasCompletedBootstrapTranscriptTurn`; er wordt geen
  bestandsvormige helpernaam meer beschikbaar gesteld.
- Tests van de ingebedde runner gebruiken nu SQLite-transcriptie-identiteit en voor het openen van een nieuwe
  transcriptiebeheerder is altijd een expliciete `sessionId` vereist.
- Helpers voor geheugenindexering gebruiken nu van begin tot eind SQLite-transcriptieterminologie:
  de host exporteert `listSessionTranscriptScopesForAgent` en
  `sessionTranscriptKeyForScope`, gerichte synchronisatie plaatst `sessionTranscripts` in de wachtrij,
  treffers van openbare sessiezoekopdrachten stellen ondoorzichtige `transcript:<agent>:<session>`-paden beschikbaar
  en de interne DB-bronsleutel is `session:<session>` onder
  `source_kind='sessions'` in plaats van een fictief bestandspad.
- De generieke helper voor persistente deduplicatie van de Plugin-SDK stelt geen bestandsvormige
  opties meer beschikbaar. Aanroepers leveren SQLite-bereiksleutels en duurzame deduplicatierijen bevinden zich in
  de gedeelde Plugin-status.
- Microsoft Teams SSO-tokens zijn verplaatst van vergrendelde JSON-bestanden naar de SQLite-status van de Plugin.
  Doctor importeert `msteams-sso-tokens.json`, bouwt canonieke SSO-tokensleutels
  opnieuw op uit payloads en verwijdert het bronbestand. Gedelegeerde OAuth-tokens blijven
  binnen hun bestaande grens van privéreferentiebestanden.
- De status van de Matrix-synchronisatiecache is verplaatst van `bot-storage.json` naar de SQLite-status van de Plugin.
  Doctor importeert verouderde ruwe of ingepakte synchronisatiepayloads en verwijdert het
  bronbestand. Actieve adapterclients voor Matrix en QA Lab Matrix geven een SQLite-hoofdmap voor synchronisatieopslag door,
  niet een fictief pad `sync-store.json` of `bot-storage.json`.
- De status van de verouderde Matrix-cryptomigratie is verplaatst van
  `legacy-crypto-migration.json` naar de SQLite-status van de Plugin. Doctor importeert het
  oude statusbestand; Matrix SDK IndexedDB-snapshots zijn verplaatst van
  `crypto-idb-snapshot.json` naar SQLite-Plugin-blobs. Matrix-herstelsleutels en
  referenties zijn SQLite-rijen van de Plugin-status; hun oude JSON-bestanden dienen alleen als invoer voor
  doctor-migratie.
- Activiteitslogboeken van Memory Wiki gebruiken nu de SQLite-status van de Plugin in plaats van
  `.openclaw-wiki/log.jsonl`. De migratieprovider van Memory Wiki importeert oude
  JSONL-logboeken; wiki-Markdown en inhoud van gebruikerskluizen blijven als
  werkruimte-inhoud in bestanden opgeslagen.
- Memory Wiki maakt niet langer `.openclaw-wiki/state.json` of de ongebruikte
  map `.openclaw-wiki/locks` aan. De migratieprovider verwijdert die uitgefaseerde
  metagegevensbestanden van de Plugin als een oudere kluis ze nog bevat.
- Auditvermeldingen van de systeemagent gebruiken nu de SQLite-status van de kern-Plugin in plaats van
  `audit/crestodian.jsonl`. Doctor importeert het verouderde JSONL-auditlogboek en
  verwijdert dit na een geslaagde import.
- Auditvermeldingen voor het schrijven/observeren van configuratie gebruiken nu de SQLite-status van de kern-Plugin in plaats
  van `logs/config-audit.jsonl`. Doctor importeert het verouderde JSONL-auditlogboek en
  verwijdert dit na een geslaagde import.
- De macOS-begeleidende app schrijft niet langer app-lokale sidecars `logs/config-audit.jsonl` of
  `logs/config-health.json` tijdens het bewerken van `openclaw.json`. Het configuratiebestand
  blijft in bestanden opgeslagen, herstelsnapshots blijven naast het configuratiebestand staan
  en duurzame configuratie-audit-/statusgegevens horen thuis in de SQLite-opslag van de Gateway.
- Openstaande goedkeuringen voor reddingsacties van de systeemagent gebruiken nu de SQLite-status van de kern-Plugin in plaats
  van `crestodian/rescue-pending/*.json` of `openclaw/rescue-pending/*.json`.
  Deze kortlevende beveiligingsmogelijkheden worden nooit geïmporteerd; doctor verwijdert
  beide uitgefaseerde mappen, zodat een upgrade geen verouderde schrijfactie opnieuw kan activeren.
- De tijdelijke inschakelstatus van Phone Control gebruikt nu de SQLite-status van de Plugin in plaats van
  `plugins/phone-control/armed.json`. Doctor importeert het verouderde bestand met de ingeschakelde status
  in de naamruimte `phone-control/arm-state` en verwijdert het bestand.
- Doctor herstelt JSONL-transcripties niet langer ter plaatse en maakt geen JSONL-
  back-upbestanden meer. Het importeert de actieve vertakking in SQLite en verwijdert de verouderde bron.
- Het opzoeken van transcripties door de sessiegeheugenhook gebruikt alleen SQLite-lezingen binnen het bereik
  `{agentId, sessionId}`. De helper accepteert of bepaalt geen transcriptielocators,
  verouderde bestandslezingen of opties voor het herschrijven van bestanden meer.
- Gespreksbindingen van de Codex-app-server gebruiken nu de OpenClaw-sessiesleutel of een expliciet
  `{agentId, sessionId}`-bereik als sleutel voor de SQLite-status van de Plugin. Ze mogen geen
  fallbackbindingen voor transcriptiepaden behouden.
- Lezingen van gespiegelde geschiedenis door de Codex-app-server gebruiken uitsluitend het SQLite-transcriptiebereik;
  ze mogen de identiteit niet herstellen uit paden naar transcriptiebestanden.
- Paden voor rolordening en Compaction-reset ontkoppelen geen oude transcriptiebestanden meer;
  reset roteert alleen de SQLite-sessierij en transcriptie-identiteit.
- Reacties op Gateway-reset en controlepunten retourneren schone sessierijen plus sessie-
  id's. Ze maken niet langer SQLite-transcriptielocators voor clients.
- Dreaming van memory-core verwijdert sessierijen niet langer door te controleren op ontbrekende
  JSONL-bestanden. Opschoning van subagents verloopt via de sessieruntime-API in plaats van
  controles op het bestaan van bestanden. De transcriptie-invoertests vullen SQLite-rijen
  rechtstreeks in, in plaats van `agents/<id>/sessions`-fixtures of locator-
  tijdelijke aanduidingen te maken.
- Geheugentranscriptie-indexering kan `transcript:<agentId>:<sessionId>` beschikbaar stellen als een
  virtueel pad voor zoekresultaten ten behoeve van citatie-/leeshelpers. De duurzame indexbron is
  relationeel (`source_kind='sessions'`, `source_key='session:<sessionId>'`,
  `session_id=<sessionId>`), dus de waarde is geen locator voor een runtimetranscript,
  geen bestandssysteempad en mag nooit worden teruggegeven aan sessieruntime-API's.
- De geheugenstatus van Gateway doctor leest aantallen voor kortetermijnherinneringen en fasesignalen
  uit SQLite-rijen met Plugin-status in plaats van `memory/.dreams/*.json`; de uitvoer van de CLI en
  doctor duidt die opslag nu aan als een SQLite-opslag, niet als een pad.
- De memory-core-runtime, CLI-status, Gateway doctor-methoden en Plugin SDK-
  façades controleren of archiveren niet langer verouderde `.dreams/session-corpus`-bestanden.
  Die bestanden dienen alleen als migratie-invoer; doctor importeert ze in SQLite en
  verwijdert de bron na verificatie. Bewijsrijen voor actieve sessie-invoer
  gebruiken nu het virtuele SQLite-pad `memory/session-ingestion/<day>.txt`; de runtime
  schrijft nooit status naar of leidt deze af uit `.dreams/session-corpus`.
- Openbare artefacten van memory-core bieden SQLite-hostgebeurtenissen aan als het virtuele JSON-
  artefact `memory/events/memory-host-events.json`; ze hergebruiken niet langer het
  verouderde bronpad `.dreams/events.jsonl`.
- Sandbox-registers voor containers/browsers gebruiken nu de gedeelde
  SQLite-tabel `sandbox_registry_entries` met getypeerde kolommen voor sessie, image, tijdstempel,
  backend/configuratie en browserpoort. Doctor importeert verouderde monolithische en
  gesharde JSON-registerbestanden en verwijdert succesvol geïmporteerde bronnen. Runtime-lezingen gebruiken
  de getypeerde rijkolommen als enige bron van waarheid; `entry_json` is alleen een kopie
  voor opnieuw afspelen/foutopsporing.
- Toezeggingen gebruiken nu een getypeerde gedeelde tabel `commitments` in plaats van een
  JSON-blob voor de volledige opslag. De runtime gebruikt geïndexeerde query's voor bereik, bezorgvenster, voortschrijdend
  maximum, status en pogingen, plus synchrone SQLite-transacties;
  `record_json` is alleen een kopie voor opnieuw afspelen/foutopsporing. Expliciet herstel door doctor valideert
  de volledige verouderde `commitments.json`, behoudt nieuwere SQLite-rijen, verifieert het
  resultaat en verwijdert pas daarna de ongewijzigde bron. De runtime leest of
  schrijft het uitgefaseerde bestand nooit.
- Web Push-abonnementen en de gegenereerde VAPID-identiteit gebruiken nu getypeerde gedeelde
  rijen `web_push_subscriptions` en `web_push_vapid_keys`. Runtimeregistratie,
  opschoning na vervaldatum en sleutelgeneratie bij eerste gebruik maken gebruik van SQLite-
  transacties op rijniveau. Expliciet herstel door Doctor valideert beide uitgefaseerde JSON-opslagen,
  claimt ze vóór het schrijven naar SQLite, importeert ze atomair, weigert
  conflicterende VAPID-identiteiten, verifieert het resultaat en verwijdert pas daarna de
  claims. Doctor houdt de onderhoudsvergrendeling van de statusmap gedurende de volledige
  import vast, zodat een oudere Gateway de uitgefaseerde bestanden niet opnieuw kan aanmaken. Registratie,
  bezorging, verwijdering en sleutelresolutie worden veilig geweigerd totdat Doctor
  wachtende verouderde bronnen of onderbroken claims heeft afgehandeld.
- Cron-taakdefinities, planningsstatus en uitvoeringsgeschiedenis hebben niet langer runtime-
  JSON-schrijvers of -lezers. De runtime gebruikt rijen `cron_jobs` met getypeerde kolommen voor planning,
  payload, bezorging, storingswaarschuwing, sessie, status en runtimestatus, plus
  door Cron beheerde details in `task_runs` voor diagnostiek, bezorging, sessie/uitvoering, model
  en tokentotalen. `job_json` is alleen een kopie voor opnieuw afspelen/foutopsporing; `state_json` bewaart geneste
  runtimediagnostiek die nog geen velden voor snelle query's heeft, terwijl de runtime
  snelle statusvelden opnieuw hydrateert vanuit getypeerde kolommen. Doctor importeert
  verouderde bestanden `jobs.json`, `jobs-state.json` en `runs/*.jsonl` en verwijdert
  de geïmporteerde bronnen. Terugschrijvingen van Plugin-doelen werken overeenkomende rijen `cron_jobs`
  bij in plaats van de volledige Cron-opslag te laden en te vervangen.
- Bij het opstarten negeert de Gateway verouderde markeringen `notify: true` in de runtime-
  projectie. Doctor leest de uitgefaseerde onbewerkte `cron.webhook` alleen tijdens het vertalen
  van die markeringen naar expliciete SQLite-bezorging en verwijdert daarna de configuratiesleutel.
- Uitgaande en sessiebezorgingswachtrijen slaan nu wachtrijstatus, invoertype,
  sessiesleutel, kanaal, doel, account-id, aantal nieuwe pogingen, laatste poging/fout,
  herstelstatus en markeringen voor platformverzending op als getypeerde kolommen in de gedeelde
  tabel `delivery_queue_entries`. Runtimeherstel leest die snelle velden uit
  de getypeerde kolommen en mutaties voor nieuwe pogingen/herstel werken die kolommen rechtstreeks bij
  zonder JSON voor opnieuw afspelen te herschrijven. De volledige JSON-payload blijft alleen behouden als
  blob voor opnieuw afspelen/foutopsporing voor berichtinhoud en andere niet-frequente gegevens voor opnieuw afspelen.
- Beheerde records voor uitgaande afbeeldingen gebruiken nu getypeerde gedeelde
  rijen `managed_outgoing_image_records`. De runtime leest alleen getypeerde kolommen; de
  JSON-kolom is een kopie voor opnieuw afspelen/foutopsporing. De oorspronkelijke afbeeldingsbytes blijven benoemde
  bijlageartefacten in de map voor beheerde media.
- Discord-voorkeuren voor de modelkiezer, hashes voor opdrachtimplementatie en threadkoppelingen
  gebruiken nu gedeelde SQLite-Plugin-status. Hun plannen voor het importeren van verouderde JSON bevinden zich in het
  setup-/doctor-migratieoppervlak van de Discord-Plugin, niet in de kernmigratiecode.
- Detectiemodules voor verouderde Plugin-import gebruiken door doctor benoemde modules zoals
  `doctor-legacy-state.ts` of `doctor-state-imports.ts`; normale kanaalruntime-
  modules mogen geen detectiemodules voor verouderde JSON importeren.
- BlueBubbles-inhaalcursors en markeringen voor het dedupliceren van inkomende berichten gebruiken nu gedeelde SQLite-
  Plugin-status. Hun plannen voor het importeren van verouderde JSON bevinden zich in het
  setup-/doctor-migratieoppervlak van de BlueBubbles-Plugin, niet in de kernmigratiecode.
- Telegram-update-offsets, stickercacherijen, cache-rijen voor verzonden berichten,
  cache-rijen voor onderwerpnamen en threadkoppelingen gebruiken nu gedeelde SQLite-Plugin-
  status. Hun plannen voor het importeren van verouderde JSON bevinden zich in het
  setup-/doctor-migratieoppervlak van de Telegram-Plugin, niet in de kernmigratiecode.
- iMessage-inhaalcursors, toewijzingen van korte antwoord-id's en deduplicatierijen voor verzonden echo's
  gebruiken nu gedeelde SQLite-Plugin-status. De oude bestanden `imessage/catchup/*.json`,
  `imessage/reply-cache.jsonl` en `imessage/sent-echoes.jsonl` dienen
  alleen als invoer voor doctor.
- Rijen voor deduplicatie van Feishu-berichten maken nu gebruik van de claimbare kerndeduplicatie
  (`feishu.dedup.*`-naamruimten in gedeelde SQLite-Plugin-status) in plaats van
  `feishu/dedup/*.json`-bestanden of de uitgefaseerde handmatig gebouwde opslag `dedup.*`, zonder
  verouderde import omdat de cache voor bescherming tegen opnieuw afspelen na de upgrade opnieuw wordt opgebouwd.
- Microsoft Teams-gesprekken, peilingen, wachtende uploadbuffers en geleerde
  feedback gebruiken nu gedeelde SQLite-tabellen voor Plugin-status/blobs. Het pad voor wachtende uploads
  gebruikt `plugin_blob_entries`, zodat mediabuffers als SQLite-BLOB's worden opgeslagen
  in plaats van als base64-JSON. De namen van runtimehelpers gebruiken nu SQLite-/statusnaamgeving
  in plaats van `*-fs`-naamgeving voor bestandsopslag, en de oude shim `storePath` is verdwenen
  uit deze opslagen. Het plan voor het importeren van verouderde JSON bevindt zich in het setup-/doctor-
  migratieoppervlak van de Microsoft Teams-Plugin.
- Door Zalo gehoste uitgaande media gebruiken nu gedeelde SQLite `plugin_blob_entries`
  in plaats van tijdelijke JSON-/bin-nevenbestanden `openclaw-zalo-outbound-media`.
- HTML en metadata van de diffviewer gebruiken nu gedeelde SQLite `plugin_blob_entries`
  in plaats van tijdelijke bestanden `meta.json`/`viewer.html`. Viewer-HTML wordt opgeslagen als een
  gzip-blob en alleen de hash van het URL-token blijft bewaard. Gegenereerde PNG-/PDF-uitvoer
  blijft een tijdelijke materialisatie omdat kanaalbezorging nog steeds een bestandspad vereist;
  de vervalmetadata ervan wordt beheerd door SQLite, zonder JSON-nevenbestanden.
- Beheerde Canvas-documenten gebruiken nu gedeelde SQLite `plugin_blob_entries` in plaats
  van een standaardmap `state/canvas/documents`. De Canvas-host biedt die
  blobs rechtstreeks aan; lokale bestanden worden alleen aangemaakt voor expliciete `host.root`-
  operatorinhoud of tijdelijke materialisatie wanneer een stroomafwaartse medialezer
  een pad vereist.
- Auditbeslissingen voor File Transfer gebruiken nu gedeelde SQLite `plugin_state_entries`
  in plaats van het onbegrensde runtimelogboek `audit/file-transfer.jsonl`. Doctor
  importeert het verouderde JSONL-auditbestand in de Plugin-status en verwijdert de bron
  na een foutloze import.
- ACPX-procesleases en Gateway-instantie-identiteit gebruiken nu gedeelde SQLite-Plugin-
  status. Doctor importeert het verouderde bestand `gateway-instance-id` in de Plugin-status
  en verwijdert de bron.
- Door ACPX gegenereerde wrapperscripts en de geïsoleerde Codex-home zijn tijdelijke
  materialisaties onder de tijdelijke hoofdmap van OpenClaw, geen duurzame OpenClaw-status. De
  duurzame ACPX-runtimerecords zijn de SQLite-lease- en Gateway-instantierijen;
  het oude ACPX-configuratieoppervlak `stateDir` is verwijderd omdat daar geen runtimestatus
  meer wordt geschreven.
- Gateway-mediabijlagen gebruiken nu de gedeelde SQLite-tabel `media_blobs` als
  canonieke byteopslag. Lokale paden die worden teruggegeven aan compatibiliteitsoppervlakken voor kanalen en de Sandbox
  zijn tijdelijke materialisaties van de databaserij, niet de
  duurzame mediaopslag. Runtime-toelatingslijsten voor media bevatten niet langer verouderde
  hoofdlocaties `$OPENCLAW_STATE_DIR/media` of `media` uit de configuratiemap; die mappen dienen
  alleen als importbronnen voor doctor.
- Shell-aanvulling schrijft niet langer cachebestanden
  `$OPENCLAW_STATE_DIR/completions/*`. Smoke-paden voor installatie, doctor, updates en releases gebruiken gegenereerde
  aanvullingsuitvoer of het inladen van profielen in plaats van duurzame cachebestanden voor
  aanvulling.
- Gateway-staging voor Skills-uploads gebruikt nu gedeelde rijen `skill_uploads` en
  `skill_upload_chunks`. Chunks blijven tijdens
  het uploaden afzonderlijk transactioneel, waarna de commit één geverifieerde archief-BLOB samenstelt en de chunkrijen
  verwijdert. Het installatieprogramma ontvangt alleen een tijdelijk gematerialiseerd archiefpad terwijl
  een installatie wordt uitgevoerd. Doctor verwijdert de uitgefaseerde stagingstructuur van één uur in het bestandssysteem
  in plaats van tijdelijke uploads te importeren.
- Inlinebijlagen van subagents worden niet langer gematerialiseerd onder
  `.openclaw/attachments/*` van de werkruimte. Het spawnpad bereidt SQLite VFS-seeditems voor,
  inline-uitvoeringen plaatsen die items in de scratchnaamruimte van de runtime per agent
  en schijfgebaseerde tools leggen die SQLite-scratch over de bijlagepaden heen. De
  oude registerkolommen en opschoningshooks voor bijlagemappen van subagentuitvoeringen zijn verwijderd.
- CLI-afbeeldingshydratie onderhoudt niet langer stabiele cachebestanden
  `openclaw-cli-images`. Externe CLI-backends ontvangen nog steeds bestandspaden, maar die paden zijn
  tijdelijke materialisaties per uitvoering met opschoning.
- Diagnostiek voor cachetracering, Anthropic-payloaddiagnostiek, onbewerkte modelstream-
  diagnostiek, diagnostische tijdlijngebeurtenissen en Gateway-stabiliteitsbundels schrijven nu
  SQLite-rijen in plaats van bestanden `logs/*.jsonl` of
  `logs/stability/*.json`.
  Runtimevlaggen en omgevingsvariabelen voor padoverschrijving zijn verwijderd; export-/foutopsporingsopdrachten
  kunnen expliciet bestanden materialiseren vanuit databaserijen.
- De macOS-begeleidende app heeft niet langer een voortschrijdende schrijver voor `diagnostics.jsonl`. App-
  logboeken gaan naar geïntegreerde logging en duurzame Gateway-diagnostiek blijft door SQLite ondersteund.
- De recordlijst van de macOS-poortbewaker gebruikt nu getypeerde gedeelde SQLite-
  rijen `macos_port_guardian_records` in plaats van een JSON-bestand in Application Support
  of een ondoorzichtige singleton-blob. Alle macOS-appprofielen gebruiken dezelfde systeembrede native
  database omdat ze lokale machinepoorten coördineren. Elke grootboekbewerking
  blokkeert zolang een oudere appversie die JSON schrijft actief is. De migratie neemt alleen deel aan het stabiele
  bestandsvergrendelingsprotocol van het oude grootboek om een momentopname te maken en later de
  bron opnieuw te valideren. Elke verouderde rij wordt afgeleid uit actuele opdracht- en processtartgegevens
  zonder die vergrendeling vast te houden; daarna worden gezaghebbende SQLite-rijen opnieuw gelezen, wordt het
  plan toegepast, elk ontvangstbewijs geverifieerd en de bron verwijderd. Nieuwe verwijderingspogingen maken opnieuw een plan voor
  ontbrekende rijen, zodat uitgefaseerde verouderde ontvangstbewijzen niet opnieuw tot leven kunnen komen. De vergrendeling blijft
  kortstondig, zodat een oudere schrijver niet kan blijven vastzitten nadat SSH een proces heeft gestart. De omschakeling is
  opzettelijk eenrichtingsverkeer: de normale runtime leest, projecteert of schrijft nooit JSON,
  en terugdraaien naar builds die alleen JSON ondersteunen behoudt nieuwere SQLite-ontvangstbewijzen niet.
- Singletonvergrendelingen van de Gateway gebruiken nu getypeerde gedeelde SQLite-rijen `state_leases` onder
  het bereik `gateway_locks` in plaats van vergrendelingsbestanden in de tijdelijke map. Documentatie voor het oplossen van problemen met Fly en OAuth
  verwijst nu naar de SQLite-lease-/vergrendeling voor het vernieuwen van authenticatie in plaats
  van het opschonen van verouderde bestandsvergrendelingen.
- De status van de herstartsentinel van de Gateway gebruikt nu getypeerde gedeelde SQLite-
  `gateway_restart_sentinel`-rijen in plaats van `restart-sentinel.json`; de runtime
  leest het sentineltype, de status, routering, het bericht, de voortzetting en statistieken uit
  getypeerde kolommen. Die kolommen zijn leidend; `payload_json` is alleen een
  schaduwkopie voor opnieuw afspelen/debuggen. De runtimepaden voor lezen, schrijven en wissen gebruiken uitsluitend SQLite.
  Eén begrensde statusmigratiemodule wordt tijdens het opstarten en Doctor uitgevoerd om een
  gevalideerde oudere sentinel van na een update te importeren vóór het normale herstartherstel, de
  getypeerde rij te verifiëren en het bronbestand te verwijderen. Geen enkele runtime-module voor
  stabiele toestand leest, schrijft of ruimt het verouderde bestand op.
- De herstartintentie van de Gateway en de overdrachtsstatus van de supervisor gebruiken nu getypeerde gedeelde
  SQLite-rijen `gateway_restart_intent` en `gateway_restart_handoff` in plaats van
  de sidecars `gateway-restart-intent.json` en
  `gateway-supervisor-restart-handoff.json`.
- De coördinatie van de Gateway-singleton gebruikt nu getypeerde `state_leases`-rijen onder
  `gateway_locks` in plaats van `gateway.<hash>.lock`-bestanden te schrijven. De leaserij
  bevat de vergrendelingseigenaar, vervaltijd, Heartbeat en debugpayload; SQLite beheert de
  atomaire grens voor verkrijgen/vrijgeven. De buiten gebruik gestelde optie voor een bestandsvergrendelingsmap is
  verwijderd; tests gebruiken rechtstreeks de identiteit van de SQLite-rij.
- De oude, niet-gerefereerde helper voor Cron-gebruiksrapportage die `cron/runs/*.jsonl`-
  bestanden scande, is verwijderd. Rapporten over de uitvoeringsgeschiedenis van Cron lezen door Cron beheerde `task_runs`-rijen.
- Herstartherstel van de hoofdsessie vindt kandidaat-agents nu via het
  SQLite-register `agent_databases` in plaats van `agents/*/sessions`-
  mappen te scannen.
- Herstel van beschadigde Gemini-sessies verwijdert nu alleen de SQLite-sessierij;
  een verouderde `storePath`-poort is niet langer nodig en er wordt niet meer geprobeerd een afgeleid
  JSONL-transcriptpad te ontkoppelen.
- De verwerking van padoverschrijvingen behandelt letterlijke omgevingswaarden
  `undefined`/`null` nu als niet ingesteld, waardoor onbedoelde
  `undefined/state/*.sqlite`-databases in de hoofdmap van de repository tijdens tests of shelloverdrachten worden voorkomen.
- Vingerafdrukken voor de configuratiestatus gebruiken nu getypeerde gedeelde SQLite-rijen
  `config_health_entries` in plaats van `logs/config-health.json`, zodat het normale configuratiebestand
  het enige configuratiedocument zonder referenties blijft. De macOS-begeleidende app bewaart alleen
  proceslokale statusinformatie en maakt de oude JSON-sidecar niet opnieuw aan.
- De runtime voor authenticatieprofielen importeert of schrijft geen JSON-bestanden met referenties meer. De
  canonieke opslag voor referenties is SQLite; `auth-profiles.json`, `auth.json`
  per agent en gedeelde `credentials/oauth.json` zijn migratie-invoer voor Doctor
  die na het importeren wordt verwijderd.
- Tests voor het opslaan en de status van authenticatieprofielen controleren nu rechtstreeks getypeerde SQLite-authenticatietabellen
  en gebruiken verouderde bestandsnamen voor authenticatieprofielen alleen als migratie-invoer voor Doctor.
- `openclaw secrets apply` schoont alleen het configuratiebestand, omgevingsbestand en de SQLite-
  opslag voor authenticatieprofielen op. Deze bevat niet langer compatibiliteitslogica die de buiten gebruik gestelde
  `auth.json` per agent bewerkt; Doctor beheert het importeren en verwijderen van dat bestand.
- Migratieplannen voor Hermes-geheimen passen geïmporteerde API-sleutelprofielen rechtstreeks
  toe op de SQLite-opslag voor authenticatieprofielen. `auth-profiles.json` wordt niet langer
  geschreven of geverifieerd als tussentijds doel.
- Gebruikersgerichte documentatie over authenticatie beschrijft nu
  `state/openclaw.sqlite#table/auth_profile_stores/<agentDir>` in plaats van
  gebruikers te vertellen `auth-profiles.json` te inspecteren of kopiëren; verouderde namen voor OAuth-/authenticatie-JSON
  blijven alleen gedocumenteerd als invoer voor import door Doctor.
- MCP OAuth-sessies gebruiken nu geversioneerde `mcp_oauth_stores`-rijen in gedeelde
  `state/openclaw.sqlite`. Door de SDK beheerde token-, clientregistratie- en detectieobjecten
  blijven één gevalideerde JSON-payload, zodat uitbreidingsvelden van afhankelijkheden
  behouden blijven, terwijl elke lees-/wijzig-/schrijfbewerking in één korte Kysely-
  transactie wordt vastgelegd. Eén gedeelde SQLite-lease serialiseert vernieuwen, aanmelden en afmelden;
  ingebedde MCP-transports staan niet langer toe dat de MCP SDK buiten die
  lease vernieuwt. Doctor importeert en verwijdert exclusief buiten gebruik gestelde `mcp-oauth/*.json`-
  opslag met bronbewijzen, en de runtime heeft geen bestandsterugvaloptie.
- Helpers voor kernstatuspaden stellen het buiten gebruik gestelde bestand `credentials/oauth.json`
  niet langer beschikbaar. De verouderde bestandsnaam is lokaal voor het importpad voor Doctor-authenticatie.
- Documentatie over installatie, beveiliging, onboarding, modelauthenticatie en SecretRef beschrijft nu
  SQLite-rijen voor authenticatieprofielen en back-up/migratie van de volledige status in plaats van
  JSON-bestanden voor authenticatieprofielen per agent.
- PI-modeldetectie geeft canonieke referenties nu door aan de
  authenticatieopslag `pi-coding-agent` in het geheugen. Tijdens detectie wordt
  `auth.json` per agent niet langer aangemaakt, opgeschoond of geschreven.
- Instellingen voor de trigger en routering van Voice Wake gebruiken nu getypeerde gedeelde SQLite-tabellen
  in plaats van `settings/voicewake.json`, `settings/voicewake-routing.json` of
  ondoorzichtige generieke rijen; Doctor importeert de verouderde JSON-bestanden en verwijdert ze na een
  geslaagde migratie.
- De status van updatecontroles gebruikt nu een getypeerde gedeelde `update_check_state`-rij in plaats van
  `update-check.json` of een ondoorzichtige generieke blob; Doctor importeert
  het verouderde JSON-bestand en verwijdert het na een geslaagde migratie.
- De status van de configuratiegezondheid gebruikt nu getypeerde gedeelde `config_health_entries`-rijen in plaats
  van `logs/config-health.json` of een ondoorzichtige generieke blob; Doctor
  importeert het verouderde JSON-bestand en verwijdert het na een geslaagde migratie.
- Goedkeuringen voor gespreksbindingen van Plugins gebruiken nu getypeerde
  `plugin_binding_approvals`-rijen in plaats van ondoorzichtige gedeelde SQLite-status of
  `plugin-binding-approvals.json`; het verouderde bestand is migratie-invoer voor Doctor.
- Generieke bindingen voor het huidige gesprek slaan nu getypeerde
  `current_conversation_bindings`-rijen op in plaats van
  `bindings/current-conversations.json` te herschrijven; Doctor importeert het verouderde JSON-bestand en
  verwijdert het na een geslaagde migratie.
- Synchronisatielogboeken voor geïmporteerde bronnen van Memory Wiki slaan nu één SQLite-pluginstatusrij
  per kluis-/bronsleutel op in plaats van `.openclaw-wiki/source-sync.json` te herschrijven;
  de migratieprovider importeert en verwijdert het verouderde JSON-logboek.
- Records van ChatGPT-importuitvoeringen van Memory Wiki slaan nu één SQLite-pluginstatusrij
  per kluis-/uitvoerings-id op in plaats van `.openclaw-wiki/import-runs/*.json` te schrijven.
  Terugdraaisnapshots blijven expliciete kluisbestanden totdat de archivering van snapshots
  van importuitvoeringen naar blobopslag wordt verplaatst.
- Gecompileerde samenvattingen van Memory Wiki slaan nu gecomprimeerde SQLite-pluginblobrijen
  op in plaats van `.openclaw-wiki/cache/agent-digest.json` en
  `.openclaw-wiki/cache/claims.jsonl` te schrijven. De cache kan opnieuw worden opgebouwd, dus Doctor
  verwijdert oude cachebestanden zonder ze te importeren.
- Het bijhouden van Skills-installaties door ClawHub slaat nu één SQLite-pluginstatusrij per
  werkruimte/Skill op in plaats van tijdens runtime de sidecars `.clawhub/lock.json` en
  `.clawhub/origin.json` te schrijven of lezen. Runtimecode gebruikt statusobjecten voor bijgehouden installaties
  in plaats van bestandsvormige abstracties voor lockfiles/oorsprong. Doctor
  importeert de verouderde sidecars uit geconfigureerde agentwerkruimten en verwijdert ze
  na een foutloze import.
- De index van geïnstalleerde Plugins leest en schrijft nu de getypeerde gedeelde SQLite-
  singletonrij `installed_plugin_index` in plaats van `plugins/installs.json`; het
  verouderde JSON-bestand is alleen migratie-invoer voor Doctor en wordt na het importeren verwijderd.
- De verouderde helper voor het pad `plugins/installs.json` bevindt zich nu in verouderde
  Doctor-code. Runtime-modules voor de Plugin-index stellen alleen door SQLite ondersteunde persistentieopties
  beschikbaar, geen pad naar een JSON-bestand.
- De herstartsentinel, herstartintentie en overdrachtsstatus van de supervisor van de Gateway gebruiken nu
  getypeerde gedeelde SQLite-rijen (`gateway_restart_sentinel`,
  `gateway_restart_intent` en `gateway_restart_handoff`) in plaats van generieke
  ondoorzichtige blobs. Runtimecode voor herstarten heeft geen bestandsvormig contract voor sentinel/intentie/overdracht.
- De Matrix-synchronisatiecache, opslagmetadata, threadbindingen, deduplicatiemarkeringen voor inkomende berichten,
  afkoelstatus voor opstartverificatie, cryptografische SDK IndexedDB-snapshots,
  referenties en herstelsleutels gebruiken nu gedeelde SQLite-tabellen voor Plugin-status/blob.
  Runtime-padstructuren stellen niet langer een metadatapad `storage-meta.json`
  beschikbaar; die bestandsnaam is alleen invoer voor verouderde migratie. Het importplan voor hun verouderde JSON
  bevindt zich in het migratieoppervlak voor installatie/Doctor van de Matrix-Plugin. Deduplicatiemarkeringen voor
  inkomende berichten maken gebruik van de opeisbare kerndeduplicatie (`matrix.inbound-dedupe.*`-
  naamruimten in de gedeelde statusdatabase); de Matrix-statusmigratie van Doctor importeert
  de buiten gebruik gestelde `inbound-dedupe`-rijen per hoofdmap en `inbound-dedupe.json` één keer,
  waarna de runtime alleen de opslag voor opeisbare deduplicatie leest.
- Matrix scant, rapporteert of voltooit tijdens het opstarten geen verouderde Matrix-bestandsstatus
  meer. Detectie van Matrix-bestanden, aanmaak van verouderde cryptografische snapshots, migratiestatus voor
  herstel van kamersleutels, import en verwijdering van bronnen worden allemaal door Doctor beheerd.
- Runtime-migratiebarrels van Matrix zijn verwijderd. Helpers voor detectie en mutatie
  van verouderde status/cryptografie worden rechtstreeks door Matrix Doctor geïmporteerd in plaats van deel uit te maken
  van het runtime-API-oppervlak.
- Markeringen voor hergebruik van Matrix-migratiesnapshots bevinden zich nu in SQLite-pluginstatus
  in plaats van `matrix/migration-snapshot.json`; Doctor kan hetzelfde
  geverifieerde archief van vóór de migratie nog steeds hergebruiken zonder een sidecar-statusbestand te schrijven.
- Nostr-buscursors en de publicatiestatus van profielen gebruiken nu gedeelde SQLite-pluginstatus.
  Hun importplan voor verouderde JSON bevindt zich in het migratieoppervlak voor installatie/Doctor
  van de Nostr-Plugin.
- Sessieschakelaars van Active Memory gebruiken nu gedeelde SQLite-pluginstatus in plaats van
  `session-toggles.json`; wanneer geheugen weer wordt ingeschakeld, wordt de rij verwijderd in plaats van
  een JSON-object te herschrijven.
- Voorstellen en reviewtellers van Skill Workshop gebruiken nu gedeelde SQLite-pluginstatus
  in plaats van `skill-workshop/<workspace>.json`-opslag per werkruimte. Elk
  voorstel is een afzonderlijke rij onder `skill-workshop/proposals`, en de reviewteller
  is een afzonderlijke rij onder `skill-workshop/reviews`.
- Uitvoeringen van reviewer-subagents van Skill Workshop gebruiken nu de runtime-resolver voor sessietranscripten
  in plaats van sidecar-sessiepaden `skill-workshop/<sessionId>.json` aan te maken.
- ACPX-procesleases gebruiken nu gedeelde SQLite-pluginstatus onder
  `acpx/process-leases` in plaats van een volledig bestandsregister `process-leases.json`.
  Elke lease wordt als een eigen rij opgeslagen, waardoor het opruimen van verouderde processen bij het opstarten behouden blijft
  zonder een runtimepad dat JSON herschrijft.
- ACPX-wrapperscripts en de geïsoleerde Codex-hoofdmap worden gegenereerd in de
  tijdelijke hoofdmap van OpenClaw. Ze worden indien nodig opnieuw aangemaakt en zijn geen invoer voor
  back-up of migratie.
- Persistentie van het register voor subagentuitvoeringen gebruikt getypeerde gedeelde `subagent_runs`-rijen. Het
  oude pad `subagents/runs.json` is nu alleen invoer voor opschoning door Doctor. Doctor
  eist het op onder de vergrendeling voor statusonderhoud, legt het besluit tot verwijderen vast in
  SQLite en verwijdert het zonder tijdelijke uitvoeringsstatus te importeren. Er blijft geen runtime-JSON-
  lezer, -schrijver, -cache of -terugvaloptie over; herstel tussen versies van uitsluitend in bestanden
  aanwezige actieve uitvoeringen wordt op deze buitengebruikstellingsgrens bewust niet ondersteund.
  Runtimetests maken niet langer ongeldige of lege `runs.json`-fixtures aan om
  registergedrag aan te tonen; ze vullen/lezen rechtstreeks SQLite-rijen.
- Back-up plaatst de statusmap in een tijdelijke opslag voordat deze wordt gearchiveerd, kopieert bestanden die geen database zijn,
  maakt snapshots van databases met online back-up plus offline `VACUUM`, laat actieve WAL/SHM-sidecars weg, legt
  snapshotmetadata vast in het archiefmanifest en registreert
  voltooide back-upuitvoeringen in SQLite met het archiefmanifest. `openclaw backup
create` valideert het geschreven archief standaard; `--no-verify` is het
  expliciete snelle pad.
- `openclaw backup restore` valideert het archief vóór extractie, hergebruikt het
  genormaliseerde manifest van de verificator en herstelt geverifieerde manifestassets naar hun
  vastgelegde bronpaden. Voor schrijfbewerkingen is `--yes` vereist en `--dry-run`
  wordt ondersteund voor een herstelplan.
- Het oude filter voor vluchtige back-uppaden is verwijderd. Back-up heeft niet langer een
  overslalijst voor live-tar nodig voor verouderde JSON-/JSONL-bestanden van sessies of Cron, omdat SQLite-
  snapshots vóór het maken van het archief in een tijdelijke opslag worden geplaatst.
- Eenvoudige installatie en voorbereiding van de onboardingwerkruimte maken niet langer
  `agents/<agentId>/sessions/`-mappen aan. Ze maken alleen config/werkruimte aan;
  SQLite-sessierijen en transcriptrijen worden op aanvraag aangemaakt in de
  database per agent.
- Herstel van beveiligingsmachtigingen richt zich nu op de globale SQLite-database
  en de SQLite-databases per agent, plus WAL/SHM-sidecars, in plaats van `sessions.json`- en transcript-
  JSONL-bestanden.
- Runtimenamen voor het sandboxregister beschrijven nu rechtstreeks de soorten SQLite-registers,
  in plaats van verouderde JSON-registerterminologie mee te nemen naar de actieve opslag.
- `openclaw reset --scope config+creds+sessions` verwijdert
  `openclaw-agent.sqlite`-databases per agent plus WAL/SHM-sidecars, niet alleen verouderde
  `sessions/`-mappen.
- Gateway-helpers voor samengevoegde sessies gebruiken nu itemgerichte namen:
  `loadCombinedSessionEntriesForGateway` retourneert `{ databasePath, entries }`.
  De oude naamgeving voor gecombineerde opslag is verwijderd uit runtime-aanroepers.
- Het vullen van Docker MCP-kanalen schrijft nu de hoofdsessierij en transcript-
  gebeurtenissen naar de SQLite-database per agent, in plaats van
  `sessions.json` en een JSONL-transcript aan te maken.
- De gebundelde session-memory-hook haalt context uit de vorige sessie nu op uit
  SQLite via `{agentId, sessionId}`. Deze scant, bewaart of genereert niet langer
  transcriptpaden of `workspace/sessions`-mappen.
- De gebundelde command-logger-hook schrijft commando-auditrijen nu naar de gedeelde
  SQLite-tabel `command_log_entries`, in plaats van ze toe te voegen aan
  `logs/commands.log`.
- Toelatingslijsten voor kanaalkoppeling stellen tijdens runtime nu alleen door SQLite ondersteunde
  lees-/schrijfhelpers beschikbaar. De verouderde padresolver van de Plugin-SDK blijft behouden voor
  migratiecompatibiliteit; bestandslezers bevinden zich alleen in de migratiecode voor doctor-status.
- `migration_runs` registreert uitvoeringen van migraties van verouderde status met status,
  tijdstempels en JSON-rapporten.
- `migration_sources` registreert elke geïmporteerde bron van een verouderd bestand met hash, grootte,
  aantal records, doeltabel, uitvoerings-id, status en de verwijderingsstatus van de bron.
- `backup_runs` registreert paden van back-uparchieven, status en JSON-manifesten.
- Het globale schema bevat geen ongebruikte registertabel `agents`. Het ontdekken van
  agentdatabases is het canonieke `agent_databases`-register totdat de runtime
  een echte eigenaar voor agentrecords heeft.
- De gegenereerde configuratie van de modelcatalogus wordt opgeslagen in getypeerde globale SQLite-
  `agent_model_catalogs`-rijen, geïndexeerd op agentmap. Runtime-aanroepers gebruiken
  `ensureOpenClawModelCatalog`; er is geen compatibiliteits-API `models.json` in
  runtimecode. De implementatie schrijft naar SQLite en het ingebedde PI-register wordt
  gevuld vanuit die opgeslagen payload zonder een `models.json`-bestand aan te maken.
- De optionele export `memory.qmd.sessions` leest canonieke transcriptrijen uit
  de database per agent en materialiseert opgeschoonde Markdown onder de QMD-hoofdmap
  als een expliciet QMD-invoerartefact. QMD-sessieverzamelingen en
  identiteitstoewijzingen voor artefacten blijven daarom deel uitmaken van de geconfigureerde bridge
  naar externe tools; ze vormen geen tweede canonieke transcriptopslag.
- QMD's eigen `index.sqlite`, YAML-verzamelingsconfiguratie en modeldownloads blijven
  artefacten van externe tools onder `~/.openclaw/agents/<agentId>/qmd`; ze worden niet
  gespiegeld naar `plugin_blob_entries`. QMD-coördinatie die eigendom is van OpenClaw is
  database-first: gedeelde `state_leases` serialiseren embeddings globaal en
  `state_leases` per agent serialiseren schrijfbewerkingen voor verzamelen/bijwerken/embedding. De runtime maakt geen
  QMD-locksidecars aan.
- De optionele Plugin `memory-lancedb` maakt niet langer
  `~/.openclaw/memory/lancedb` aan als impliciete door OpenClaw beheerde opslag. Het is een
  externe LanceDB-backend en blijft uitgeschakeld totdat de beheerder een expliciete
  `dbPath` configureert.
- `check:database-first-legacy-stores` keurt nieuwe runtimebroncode af die
  verouderde opslagnamen koppelt aan schrijfgerichte bestandssysteem-API's. Ook wordt runtimebroncode afgekeurd
  die de uitgefaseerde markers voor de transcriptbridge
  `transcriptLocator` of `sqlite-transcript://...` opnieuw invoert. Code voor migratie, doctor, import
  en expliciete export buiten sessies blijft toegestaan. Bredere namen van verouderde contracten,
  zoals `sessionFile`, `storePath` en oude facades uit het `SessionManager`-bestandstijdperk,
  hebben nog steeds huidige eigenaren en vereisen afzonderlijk werk aan migratiecontroles
  voordat ze een verplichte preflightcontrole kunnen worden. De controle omvat nu ook
  runtime-opslag `cache/*.json`, generieke
  `thread-bindings.json`-sidecars, JSON voor Cron-status/uitvoeringslogboeken, JSON voor configuratiestatus,
  sidecars voor herstarten en vergrendelen, Voice Wake-instellingen, goedkeuringen voor Plugin-koppelingen,
  JSON voor de index van geïnstalleerde Plugins, File Transfer-audit-JSONL, Memory Wiki-activiteitenlogboeken,
  het oude gebundelde tekstlogboek `command-logger` en diagnostische opties voor onbewerkte pi-mono-streams in JSONL.
  Ook worden oude namen van verouderde doctor-modules op hoofdniveau verboden, zodat
  compatibiliteitscode onder `src/commands/doctor/` blijft. Android-debughandlers
  gebruiken ook logcat/uitvoer in het geheugen in plaats van `camera_debug.log`- of
  `debug_logs.txt`-cachebestanden klaar te zetten.

## Vorm van het doelschema

Houd schema's expliciet. Runtime-status die eigendom is van de host gebruikt getypeerde tabellen. Ondoorzichtige status die eigendom is van een Plugin gebruikt `plugin_state_entries` / `plugin_blob_entries`; er is geen
generieke hosttabel `kv`.

Globale database:

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

Agentdatabase:

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

`memory_index_sources.id` is de stabiele gehele primaire sleutel; `(path, source)` blijft uniek.

Toekomstige zoekfunctionaliteit kan FTS-tabellen toevoegen zonder de canonieke gebeurtenistabellen te wijzigen:

```text
transcript_events_fts(session_id, seq, text)
vfs_entries_fts(namespace, path, text)
```

Grote waarden moeten `blob`-kolommen gebruiken, niet codering als JSON-tekenreeks. Behoud
`value_json` voor kleine gestructureerde gegevens die met gewone
SQLite-hulpmiddelen inspecteerbaar moeten blijven.

`agent_databases` is het canonieke register voor deze branch. Voeg geen
`agents`-tabel toe totdat er een echte eigenaar voor agentrecords bestaat; agentconfiguratie blijft in
`openclaw.json`.

## Vorm van de Doctor-migratie

Doctor moet één expliciete migratiestap aanroepen waarover kan worden gerapporteerd en die veilig opnieuw kan worden
uitgevoerd:

```bash
openclaw doctor --fix
```

`openclaw doctor --fix` roept de implementatie van de statusmigratie aan na
de gebruikelijke configuratiecontrole vooraf en maakt vóór het importeren een geverifieerde back-up. Het opstarten van de runtime
en `openclaw migrate` mogen geen verouderde OpenClaw-statusbestanden importeren.

Migratie-eigenschappen:

- Eén migratieronde detecteert alle verouderde bestandsbronnen en stelt een plan op
  voordat er iets wordt gewijzigd.
- Doctor maakt vóór het importeren van
  verouderde bestanden een geverifieerd back-uparchief van vóór de migratie.
- Imports zijn idempotent en worden geïdentificeerd aan de hand van bronpad, mtime, grootte, hash en doeltabel.
- Bronbestanden die met succes zijn verwerkt, worden verwijderd of gearchiveerd nadat de doeldatabase de transactie heeft
  vastgelegd.
- Bij mislukte imports blijft de bron ongewijzigd en wordt een waarschuwing vastgelegd in
  `migration_runs`.
- Runtime-code leest alleen SQLite nadat de migratie bestaat.
- Er is geen pad vereist voor downgraden/exporteren naar runtimebestanden.

## Migratie-inventaris

Verplaats deze naar de globale database:

- Runtimeschrijfbewerkingen van het taakregister gebruiken nu de gedeelde database; de niet-uitgebrachte
  `tasks/runs.sqlite`-sidecarimporteur is verwijderd. Bij het opslaan van snapshots worden upserts uitgevoerd op taak-
  id en worden alleen ontbrekende taak-/leveringsrijen verwijderd.
- Runtimeschrijfbewerkingen van Task Flow gebruiken nu de gedeelde database; de niet-uitgebrachte
  `tasks/flows/registry.sqlite`-sidecarimporteur is verwijderd. Bij het opslaan van snapshots
  worden upserts uitgevoerd op flow-id en worden alleen ontbrekende flowrijen verwijderd.
- Runtimeschrijfbewerkingen van de Pluginstatus gebruiken nu de gedeelde database; de niet-uitgebrachte
  `plugin-state/state.sqlite`-sidecarimporteur is verwijderd.
- Ingebouwd geheugenzoeken gebruikt niet langer standaard `memory/<agentId>.sqlite`; de
  indextabellen bevinden zich in de database van de beherende agent en de expliciete
  opt-in voor de `memorySearch.store.path`-sidecar is overgeheveld naar de
  doctor-configuratiemigratie.
- Bij het opnieuw indexeren van het ingebouwde geheugen worden alleen tabellen van het geheugen in de agentdatabase opnieuw ingesteld.
  Dit mag niet het volledige SQLite-bestand vervangen, omdat dezelfde database
  sessies, transcripties, VFS-rijen, artefacten en runtimecaches bevat.
- Sandbox-container-/browserregisters uit monolithische en geshardde JSON. Runtime-
  schrijfbewerkingen gebruiken nu de gedeelde database; import van verouderde JSON blijft bestaan.
- Cron-taakdefinities, planningsstatus en uitvoeringsgeschiedenis gebruiken nu gedeelde SQLite;
  doctor importeert/verwijdert verouderde bestanden `jobs.json`, `jobs-state.json` en
  `cron/runs/*.jsonl`
- Apparaatidentiteit/-authenticatie, push, updatecontrole, toezeggingen, OpenRouter-model-
  cache, index van geïnstalleerde plugins en app-serverbindingen
- Koppelings- en bootstraprecords van apparaten/Nodes gebruiken nu getypeerde SQLite-tabellen
- Abonnees op meldingen over apparaatkoppeling en markeringen voor geleverde aanvragen gebruiken nu de
  gedeelde SQLite-tabel voor Pluginstatus in plaats van `device-pair-notify.json`.
- Gespreksrecords voor spraakoproepen gebruiken nu de gedeelde SQLite-tabel voor Pluginstatus binnen de
  naamruimte `voice-call` / `calls` in plaats van `calls.jsonl`; de Plugin-CLI
  volgt en vat de door SQLite ondersteunde gespreksgeschiedenis samen.
- Gateway-sessies, records van bekende gebruikers en de citaatcache met referentie-index van QQBot gebruiken nu
  SQLite-Pluginstatus binnen `qqbot`-naamruimten (`gateway-sessions`,
  `known-users`, `ref-index`) in plaats van `session-*.json`, `known-users.json`
  en `ref-index.jsonl`. Die verouderde bestanden zijn caches en worden niet gemigreerd.
- Voorkeuren voor de Discord-modelkiezer, hashes voor opdrachtimplementatie en threadbindingen
  gebruiken nu SQLite-Pluginstatus binnen `discord`-naamruimten
  (`model-picker-preferences`, `command-deploy-hashes`, `thread-bindings`)
  in plaats van `model-picker-preferences.json`, `command-deploy-cache.json` en
  `thread-bindings.json`; de Discord doctor-/installatiemigratie importeert en
  verwijdert de verouderde bestanden.
- Inhaalcursors en deduplicatiemarkeringen voor inkomende berichten van BlueBubbles gebruiken nu SQLite-Pluginstatus
  binnen `bluebubbles`-naamruimten (`catchup-cursors`, `inbound-dedupe`)
  in plaats van `bluebubbles/catchup/*.json` en
  `bluebubbles/inbound-dedupe/*.json`; de BlueBubbles doctor-/installatiemigratie
  importeert en verwijdert de verouderde bestanden.
- Telegram-update-offsets, stickercache-items, cache-items voor berichten in antwoordketens,
  cache-items voor verzonden berichten, cache-items voor onderwerpnamen en thread-
  bindingen gebruiken nu SQLite-Pluginstatus binnen `telegram`-naamruimten
  (`update-offsets`, `sticker-cache`, `message-cache`, `sent-messages`,
  `topic-names`, `thread-bindings`) in plaats van `update-offset-*.json`,
  `sticker-cache.json`, `*.telegram-messages.json`,
  `*.telegram-sent-messages.json`, `*.telegram-topic-names.json` en
  `thread-bindings-*.json`; de Telegram doctor-/installatiemigratie importeert en
  verwijdert de verouderde bestanden.
- Inhaalcursors, toewijzingen van korte antwoord-id's en deduplicatierijen voor verzonden echo's van iMessage
  gebruiken nu SQLite-Pluginstatus binnen `imessage`-naamruimten (`catchup-cursors`,
  `reply-cache`, `sent-echoes`) in plaats van `imessage/catchup/*.json`,
  `imessage/reply-cache.jsonl` en `imessage/sent-echoes.jsonl`; de iMessage
  doctor-/installatiemigratie importeert en verwijdert de verouderde bestanden.
- Gesprekken, peilingen, SSO-tokens en geleerde feedback van Microsoft Teams
  gebruiken nu naamruimten voor SQLite-Pluginstatus (`conversations`, `polls`, `sso-tokens`,
  `feedback-learnings`) in plaats van `msteams-conversations.json`,
  `msteams-polls.json`, `msteams-sso-tokens.json` en `*.learnings.json`; de
  Microsoft Teams doctor-/installatiemigratie importeert en archiveert de verouderde bestanden.
  Uploads in behandeling vormen een kortstondige SQLite-cache en oude JSON-cachebestanden worden
  niet gemigreerd.
- Matrix-synchronisatiecache, opslagmetadata, threadbindingen, deduplicatiemarkeringen voor inkomende berichten,
  afkoelstatus voor opstartverificatie, aanmeldgegevens, herstelsleutels en cryptografische
  IndexedDB-snapshots van de SDK gebruiken nu naamruimten voor SQLite-Pluginstatus/-blobs binnen
  `matrix` (`sync-store`, `storage-meta`, `thread-bindings`,
  `matrix.inbound-dedupe.*` via de opeisbare kerndeduplicatie,
  `startup-verification`, `credentials`, `recovery-key`, `idb-snapshots`)
  in plaats van `bot-storage.json`, `storage-meta.json`, `thread-bindings.json`,
  `inbound-dedupe.json`, `startup-verification.json`, `credentials.json`,
  `recovery-key.json` en `crypto-idb-snapshot.json`; de Matrix doctor-/installatie-
  migratie importeert en verwijdert die verouderde bestanden (en de buiten gebruik gestelde SQLite-rijen
  `inbound-dedupe` per hoofdmap) uit accountgebonden Matrix-opslaghoofdmappen.
- Nostr-buscursors en publicatiestatus van profielen gebruiken nu SQLite-Pluginstatus binnen
  `nostr`-naamruimten (`bus-state`, `profile-state`) in plaats van
  `bus-state-*.json` en `profile-state-*.json`; de Nostr doctor-/installatie-
  migratie importeert en verwijdert de verouderde bestanden.
- Sessieschakelaars van Active Memory gebruiken nu SQLite-Pluginstatus binnen
  `active-memory/session-toggles` in plaats van `session-toggles.json`.
- Voorstelwachtrijen en beoordelingstellers van Skill Workshop gebruiken nu SQLite-Pluginstatus
  binnen `skill-workshop/proposals` en `skill-workshop/reviews` in plaats van
  `skill-workshop/<workspace>.json`-bestanden per werkruimte.
- Wachtrijen voor uitgaande levering en sessielevering delen nu de globale SQLite-
  tabel `delivery_queue_entries` onder afzonderlijke wachtrijnamen
  (`outbound-delivery`, `session-delivery`) in plaats van duurzame
  bestanden `delivery-queue/*.json`, `delivery-queue/failed/*.json` en
  `session-delivery-queue/*.json`. De stap voor verouderde status van doctor importeert
  wachtende en mislukte rijen, verwijdert verouderde leveringsmarkeringen en verwijdert de oude
  JSON-bestanden na import. Velden voor directe routering en nieuwe pogingen zijn getypeerde kolommen; de
  JSON-payload blijft alleen behouden voor opnieuw afspelen/foutopsporing.
- ACPX-procesleases gebruiken nu SQLite-Pluginstatus binnen `acpx/process-leases`
  in plaats van `process-leases.json`.
- Metadata van back-up- en migratie-uitvoeringen

Verplaats deze naar agentdatabases:

- Hoofdmappen van agentsessies en sessie-itempayloads in compatibiliteitsvorm. Voltooid voor
  runtimeschrijfbewerkingen: actuele sessiemetadata kan worden opgevraagd in `sessions`, terwijl de
  volledige verouderde `SessionEntry`-payload behouden blijft in `session_entries`.
- Transcriptiegebeurtenissen van agents. Voltooid voor runtimeschrijfbewerkingen.
- Compaction-controlepunten en transcriptiesnapshots. Voltooid voor runtimeschrijfbewerkingen:
  transcriptiekopieën van controlepunten zijn SQLite-transcriptierijen en controlepunt-
  metadata wordt vastgelegd in `transcript_snapshots`. Gateway-controlepunthulpprogramma's
  noemen deze waarden nu transcriptiesnapshots in plaats van bronbestanden.
- VFS-klad-/werkruimtenaamruimten van agents. Voltooid voor VFS-runtimeschrijfbewerkingen.
- Bijlagepayloads van subagents. Voltooid voor runtimeschrijfbewerkingen: dit zijn SQLite VFS-
  initiële items en nooit duurzame werkruimtebestanden.
- Toolartefacten. Voltooid voor runtimeschrijfbewerkingen.
- Uitvoeringsartefacten. Voltooid voor runtimeschrijfbewerkingen van workers via de
  `run_artifacts`-tabel per agent.
- Lokale runtimecaches van agents. Voltooid voor bereikgebonden cache-schrijfbewerkingen van de workerruntime via
  de `cache_entries`-tabel per agent. Gateway-brede modelcaches blijven in de
  globale database, tenzij ze agentspecifiek worden.
- Logboeken van bovenliggende ACP-streams. Voltooid voor runtimeschrijfbewerkingen.
- ACP-sessies van het afspeellogboek. Voltooid voor runtimeschrijfbewerkingen via
  `acp_replay_sessions` en `acp_replay_events`; verouderde `acp/event-ledger.json`
  blijft alleen als invoer voor doctor bestaan.
- ACP-sessiemetadata. Voltooid voor runtimeschrijfbewerkingen via `acp_sessions`; verouderde
  `entry.acp`-blokken in `sessions.json` dienen alleen als invoer voor doctor-migratie.
- Traject-sidecars wanneer dit geen expliciete exportbestanden zijn. Voltooid voor runtime-
  schrijfbewerkingen: trajectregistratie schrijft `trajectory_runtime_events`-rijen naar de agentdatabase
  en spiegelt uitvoeringsgebonden artefacten naar SQLite. Verouderde sidecars dienen alleen als invoer
  voor doctor-import; export kan nieuwe JSONL-uitvoer voor ondersteuningsbundels materialiseren,
  maar leest of migreert tijdens runtime geen oude traject-/transcriptiesidecars.
  Runtime-trajectregistratie maakt het SQLite-bereik beschikbaar; JSONL-padhulpprogramma's zijn
  geïsoleerd tot ondersteuning voor export/foutopsporing en worden niet opnieuw geëxporteerd vanuit de runtimemodule.
  Trajectmetadata van de ingesloten runner legt de identiteit `{agentId, sessionId, sessionKey}`
  vast in plaats van een transcriptielocator permanent op te slaan.

Houd deze voorlopig bestandsgebaseerd:

- `openclaw.json`
- aanmeldingsgegevensbestanden van providers of de CLI
- Plugin-/pakketmanifesten
- gebruikerswerkruimten en Git-opslagplaatsen wanneer schijfmodus is geselecteerd
- logboeken die bedoeld zijn om door operators te worden gevolgd, tenzij een specifiek logboekoppervlak wordt verplaatst

## Migratieplan

### Fase 0: leg de grens vast

Maak de grens voor duurzame status expliciet voordat meer rijen worden verplaatst:

- Voeg een `migration_runs`-tabel toe aan de globale database.
  Voltooid voor uitvoeringsrapporten van verouderde-statusmigraties.
- Voeg één door doctor beheerde statusmigratieservice toe voor import van bestand naar database.
  Voltooid: `openclaw doctor --fix` gebruikt de implementatie voor verouderde-statusmigratie.
- Maak `plan` alleen-lezen en laat `apply` een back-up maken, importeren, verifiëren en
  vervolgens oude bestanden verwijderen of in quarantaine plaatsen.
  Voltooid: doctor maakt een geverifieerde back-up vóór migratie, geeft het back-uppad
  door aan `migration_runs` en hergebruikt de importeur-/verwijderingspaden.
- Voeg statische verboden toe, zodat nieuwe runtimecode geen verouderde statusbestanden kan schrijven, terwijl
  migratiecode en tests ze nog wel kunnen initialiseren/lezen.
  Voltooid voor de momenteel gemigreerde verouderde opslaglocaties; de controle scant ook geneste
  tests op verboden runtimecontracten voor transcriptielocators.

### Fase 1: voltooi het globale besturingsvlak

Bewaar gedeelde coördinatiestatus in `state/openclaw.sqlite`:

- Agents en agentdatabaseregister
- Taak- en Task Flow-logboeken
- Pluginstatus
- Sandbox-container-/browserregister
- Uitvoeringsgeschiedenis van Cron/planner
- Koppeling, apparaat, push, updatecontrole, TUI, OpenRouter-/modelcaches en andere
  kleine runtimestatus op Gateway-niveau
- Back-up- en migratiemetadata
- Bytes van Gateway-mediabijlagen. Voltooid voor runtimeschrijfbewerkingen; directe bestandspaden
  zijn tijdelijke materialisaties voor compatibiliteit met kanaalafzenders en Sandbox-
  staging. Runtime-toegestane-lijsten accepteren SQLite-materialisatiepaden, niet verouderde
  status-/configuratiehoofdmappen voor media. Doctor importeert verouderde mediabestanden in
  `media_blobs` en verwijdert de bronbestanden nadat rijen met succes zijn geschreven.
- Sessies, gebeurtenissen en payloadblobs van foutopsporingsproxyregistraties. Voltooid: registraties bevinden zich
  in de gedeelde statusdatabase en worden geopend via de bootstrap, het schema,
  WAL en de instellingen voor bezettime-out van de gedeelde statusdatabase. Payloadbytes worden met gzip gecomprimeerd in
  `capture_blobs.data`; er is geen runtime-sidecardatabase-override voor de foutopsporingsproxy,
  blobmap of alleen voor proxyregistratie bedoeld gegenereerd schema/codegeneratiedoel.
  Doctor-/opstartmigratie importeert uitgebrachte `debug-proxy/capture.sqlite`-rijen
  en blobs met payloads waarnaar wordt verwezen, inclusief actieve verouderde omgevings-
  overrides voor databases/blobs, en archiveert vervolgens die bronnen terwijl CA-certificaten intact blijven.

Deze fase verwijdert ook dubbele openers voor sidecars, machtigingshelpers, WAL-
instellingen, opschoning van het bestandssysteem en compatibiliteitsschrijvers uit die subsystemen.

### Fase 2: Databases per agent introduceren

Maak één database per agent en registreer deze vanuit de globale DB:

```text
~/.openclaw/state/openclaw.sqlite
~/.openclaw/agents/<agentId>/agent/openclaw-agent.sqlite
```

De globale `agent_databases`-rij slaat het pad, de schemaversie, het tijdstempel
van de laatste waarneming en basismetadata over grootte/integriteit op. Runtimecode vraagt het register om
de agent-DB in plaats van bestandspaden rechtstreeks af te leiden.

De agent-DB beheert:

- `sessions` als de canonieke sessiehoofdstructuur, met `session_entries` als de
  tabel met een compatibiliteitsvormige payload die aan die hoofdstructuur is gekoppeld, en
  `session_routes` als de unieke actieve `session_key`-zoekopdracht
- `conversations` en `session_conversations` als de genormaliseerde routeringsidentiteit
  van de provider die aan sessies is gekoppeld
- `transcript_events`
- transcriptsnapshots en Compaction-controlepunten. Voltooid voor runtimeschrijfbewerkingen.
- `vfs_entries`
- `tool_artifacts` en uitvoeringsartefacten
- agentlokale runtime-/cacherijen. Voltooid voor caches met workerscope.
- gebeurtenissen van bovenliggende ACP-streams
- gebeurtenissen van de trajectieruntime wanneer het geen expliciete exportartefacten zijn

### Fase 3: API's voor sessieopslag vervangen

Voltooid voor runtime. Het als bestand vormgegeven oppervlak voor sessieopslag is geen actief
runtimecontract:

- Runtime roept niet langer `loadSessionStore(storePath)` aan en behandelt `storePath` niet als
  sessie-identiteit.
- Rijbewerkingen van de runtime zijn `getSessionEntry`, `upsertSessionEntry`,
  `patchSessionEntry`, `deleteSessionEntry` en `listSessionEntries`.
- Helpers voor het herschrijven van de volledige opslag, bestandsschrijvers, wachtrijtests, opschoning van aliassen en
  parameters voor het verwijderen van verouderde sleutels zijn uit de runtime verwijderd.
- Verouderde compatibiliteitsexports van het hoofdpakket delegeren tot en met 2026-10-12 aan de uitsluitend voor doctor bestemde
  `sessions.json`-importer; compatibiliteitsleesbewerkingen van de Plugin-SDK
  blijven canonieke SQLite-rijen projecteren.
- Het parseren van `sessions.json` blijft uitsluitend aanwezig in migratie-/importcode van doctor en
  doctortests.
- De fallback voor de runtimelevenscyclus leest SQLite-transcriptheaders, niet de eerste
  regels van JSONL.

Blijf alles verwijderen wat parameters voor bestandsvergrendeling,
terminologie voor opschonen/afkappen als bestandsonderhoud, identiteit op basis van opslagpaden of tests
waarvan de enige bewering JSON-persistentie is, opnieuw introduceert.

### Fase 4: Transcripten, ACP-streams, trajecten en VFS verplaatsen

Maak elke agentgegevensstroom database-eigen:

- Schrijfbewerkingen voor het toevoegen aan transcripten verlopen via één SQLite-transactie die de
  sessieheader waarborgt, de idempotentie van berichten controleert, het uiteinde van de bovenliggende tak selecteert, gegevens
  in `transcript_events` invoegt en doorzoekbare identiteitsmetadata vastlegt in
  `transcript_event_identities`. Voltooid voor het rechtstreeks toevoegen van transcriptberichten en
  normaal persistent opgeslagen toevoegingen van `TranscriptSessionManager`; expliciete vertakkingsbewerkingen
  behouden hun expliciete keuze van de bovenliggende tak en schrijven nog steeds SQLite-rijen
  zonder een bestandslocator af te leiden.
- Logboeken van bovenliggende ACP-streams worden rijen, geen `.acp-stream.jsonl`-bestanden. Voltooid.
- De configuratie voor het starten van ACP slaat niet langer JSONL-paden van transcripten persistent op. Voltooid.
- De runtime legt trajecten rechtstreeks vast als gebeurtenisrijen/artefacten. De expliciete
  ondersteunings-/exportopdracht kan nog steeds JSONL-artefacten voor ondersteuningsbundels produceren als
  exportindeling, maar sessie-export maakt sessie-JSONL niet opnieuw aan. Voltooid.
- Werkruimten op schijf blijven op schijf wanneer de schijfmodus is geconfigureerd.
- VFS-kladruimte en de experimentele uitsluitend-VFS-werkruimtemodus gebruiken de agent-DB.

De migratie importeert oude JSONL-bestanden eenmaal, legt aantallen/hashes vast in
`migration_runs` en verwijdert geïmporteerde bestanden na integriteitscontroles.

### Fase 5: Back-up, herstel, Vacuum en verificatie

Back-ups blijven één archiefbestand:

- Maak een controlepunt voor elke globale en agentdatabase.
- Maak van elke DB een snapshot met de online back-up van SQLite, gevolgd door offline `VACUUM`.
- Archiveer compacte DB-snapshots, configuratie, externe referenties en aangevraagde
  werkruimte-exports.
- Laat onbewerkte live `*.sqlite-wal`- en `*.sqlite-shm`-bestanden weg.
- Verifieer door elke DB-snapshot te openen en `PRAGMA integrity_check` uit te voeren.
  `openclaw backup create` voert deze archiefverificatie standaard uit;
  `--no-verify` slaat uitsluitend de archiefcontrole na het schrijven over, niet de integriteitscontrole
  tijdens het maken van de snapshot.
- Herstel kopieert snapshots terug naar hun doelpaden. Herstelde globale DB's gebruiken
  versie `1`; herstelde DB's per agent gebruiken versie `2`, waarbij snapshots van versie `1`
  bij het openen atomair worden bijgewerkt.

### Fase 6: Workerruntime

Houd de workermodus experimenteel terwijl de databasesplitsing wordt ingevoerd:

- Workers ontvangen de agent-id, uitvoerings-id, bestandssysteemmodus en identiteit van het DB-register.
- Elke worker opent een eigen SQLite-verbinding.
- De bovenliggende entiteit behoudt de bevoegdheid over kanaalbezorging, goedkeuringen, configuratie en annulering.
- Begin met één worker per actieve uitvoering; voeg pooling pas toe nadat de levenscyclus en het eigendom van
  DB-verbindingen stabiel zijn.

### Fase 7: De oude wereld verwijderen

Voltooid voor runtimesessiebeheer. De oude wereld is uitsluitend toegestaan als expliciete
invoer voor doctor of uitvoer voor ondersteuning/export:

- Geen runtimeschrijfbewerkingen naar `sessions.json`, transcript-JSONL, JSON van het sandboxregister, SQLite-sidecars voor taken
  of SQLite-sidecars voor Plugin-status.
- Geen opschoning van JSON-/sessiebestanden, afkapping van bestandstranscripten, vergrendelingen van sessiebestanden
  of sessietests in de vorm van vergrendelingen.
- Geen runtimecompatibiliteitsexports die bedoeld zijn om oude sessiebestanden
  actueel te houden.
- Expliciete ondersteuningsexports blijven door de gebruiker aangevraagde archief-/materialisatie-
  indelingen en mogen bestandsnamen niet terugvoeren naar de runtime-identiteit.

## Back-up en herstel

Back-ups moeten één archiefbestand zijn, maar het vastleggen van databases moet
SQLite-eigen zijn:

1. Houd schrijftransacties begrensd zodat de online back-up voortgang kan boeken.
2. Verifieer elke live globale en agentdatabase vóór het vastleggen.
3. Leg elke database met de online back-up van SQLite vast in een tijdelijke back-upmap,
   sluit vervolgens de live verbinding en voer `VACUUM` uit op de privékopie.
   Pluginschema's die door de eigenaar gedefinieerde SQLite-mogelijkheden vereisen, weigeren standaard
   totdat de eigenaar een veilig snapshotcontract levert.
4. Archiveer de databasesnapshots, het configuratiebestand, de map met referenties, geselecteerde
   werkruimten en een manifest.
5. Verifieer de bestandsvorm van elke SQLite-snapshot, open vervolgens canonieke OpenClaw-
   databases en voer `PRAGMA integrity_check` plus rolvalidatie uit. Specifieke
   Pluginschema's blijven ondoorzichtig tenzij hun eigenaar een verificatieprogramma levert.
   `openclaw backup create` doet dit standaard; `--no-verify` is uitsluitend bedoeld om
   de archiefcontrole na het schrijven bewust over te slaan.

Vertrouw niet op onbewerkte live kopieën van `*.sqlite`, `*.sqlite-wal` en `*.sqlite-shm` als
primaire back-upindeling. Het archiefmanifest moet de databaserol,
agent-id, schemaversie, het bronpad, snapshotpad, aantal bytes en de integriteitsstatus
vastleggen.

Herstel moet de globale database en agentdatabasebestanden opnieuw opbouwen vanuit de
archiefsnapshots. Het globale schema blijft versie `1`; snapshots per agent met versie `1`
krijgen de begrensde runtime-upgrade naar versie `2`. Doctor blijft
de enige eigenaar van import van bestand naar database. De herstelopdracht valideert eerst het
archief en vervangt vervolgens elk manifestartefact door de geverifieerde uitgepakte
payload.

## Refactorplan voor de runtime

1. Voeg API's voor het databaseregister toe.
   - Bepaal paden voor de globale DB en DB's per agent.
   - Houd het globale schema op `user_version = 1`. DB's per agent gebruiken versie `2`
     met één atomaire migratie vanuit de uitgebrachte geheugensbronvorm van versie `1`.
   - Voeg helpers toe voor sluiten, controlepunten en integriteit die door tests, back-up en doctor worden gebruikt.

2. Voeg SQLite-sidecaropslagen samen.
   - Verplaats tabellen voor Plugin-status naar de globale database. Voltooid voor runtime-
     schrijfbewerkingen; de niet-uitgebrachte importer voor verouderde sidecars is verwijderd.
   - Verplaats tabellen van het takenregister naar de globale database. Voltooid voor runtime-
     schrijfbewerkingen; de niet-uitgebrachte importer voor verouderde sidecars is verwijderd.
   - Verplaats TaskFlow-tabellen naar de globale database. Voltooid voor runtimeschrijfbewerkingen;
     de niet-uitgebrachte importer voor verouderde sidecars is verwijderd.
   - Verplaats ingebouwde tabellen voor geheugenzoekopdrachten naar elke agentdatabase. Voltooid; aangepaste
     `memorySearch.store.path` wordt nu verwijderd door de configuratiemigratie van doctor.
     Volledige herindexering wordt ter plaatse uitsluitend op geheugentabellen uitgevoerd; het oude pad voor het
     omwisselen van het volledige bestand en de helper voor het omwisselen van sidecarindexen zijn verwijderd.
   - Verwijder dubbele databaseopeners, WAL-instellingen, machtigingshelpers en
     sluitpaden uit die subsystemen.

3. Verplaats tabellen die eigendom zijn van agents naar databases per agent.
   - Maak de agent-DB naar behoefte aan via het globale databaseregister. Voltooid.
   - Verplaats runtimesessie-items, transcriptgebeurtenissen, VFS-rijen en tool-
     artefacten naar agent-DB's. Voltooid.
   - Migreer geen gedeelde DB-sessie-items, transcriptgebeurtenissen,
     VFS-rijen of toolartefacten die lokaal bij een vertakking horen; die indeling is nooit uitgebracht. Behoud uitsluitend verouderde
     import van bestand naar database in doctor.

4. Vervang API's voor sessieopslag.
   - Verwijder `storePath` als runtime-identiteit. Voltooid voor runtime en afgeschermd
     door `check:database-first-legacy-stores`: sessiemetadata, route-updates,
     opdrachtpersistentie, opschoning van CLI-sessies, Feishu-redeneervoorbeelden,
     persistentie van transcriptstatus, diepte van subagents, sessieoverschrijvingen
     van authenticatieprofielen, logica voor forks van bovenliggende processen en QA-lab-inspectie bepalen nu de
     database vanuit canonieke agent-/sessiesleutels.
     Gateway-/TUI-/UI-/macOS-antwoorden met sessielijsten tonen nu `databasePath`
     in plaats van het verouderde `path`; macOS-foutopsporingsoppervlakken tonen de database per agent
     als alleen-lezenstatus in plaats van `session.store`-configuratie te schrijven.
     `/status`, trajectexport vanuit chat en CLI-afhankelijkheidsproxy's geven
     verouderde opslagpaden niet langer door; de fallback voor transcriptgebruik leest
     SQLite op basis van agent-/sessie-identiteit. Runtime- en bridgetests tonen
     `storePath` niet langer; invoer voor doctor/migratie beheert die verouderde veldnaam.
     Het gecombineerd laden van sessies door de Gateway heeft niet langer een speciale runtimetak voor
     niet-getemplate `session.store`-waarden; het voegt SQLite-rijen per agent samen.
     De verouderde doctorroute voor sessievergrendeling en de bijbehorende `.jsonl.lock`-opschoonhelper
     zijn verwijderd; SQLite vormt nu de grens voor gelijktijdige sessietoegang.
     Hot runtime-aanroeppunten gebruiken rijgerichte helpernamen zoals
     `resolveSessionRowEntry`; de oude compatibiliteitsalias `resolveSessionStoreEntry`
     is uit runtime- en Plugin-SDK-exports verwijderd.

- Gebruik `{ agentId, sessionKey }`-rijbewerkingen.
  Gereed: `getSessionEntry`, `upsertSessionEntry`, `deleteSessionEntry`,
  `patchSessionEntry` en `listSessionEntries` zijn SQLite-first-API's waarvoor
  geen pad naar een sessieopslag nodig is. Het statusoverzicht, de lokale agentstatus, de gezondheid
  en de lijstingsopdracht `openclaw sessions` lezen nu rechtstreeks rijen per agent
  en tonen SQLite-databasepaden per agent in plaats van `sessions.json`-paden.
- Vervang verwijderen/invoegen van de volledige opslag door `upsertSessionEntry`,
  `deleteSessionEntry`, `listSessionEntries` en SQL-opruimquery's.
  Gereed voor runtime: hot paths gebruiken nu rij-API's en rijpatches met nieuwe pogingen bij conflicten;
  de resterende helpers voor import/vervanging van de volledige opslag zijn beperkt tot
  migratie-importcode en tests van de SQLite-backend.
  - Verwijder `store-writer.ts` en tests voor de schrijfwachtrij. Gereed.
  - Verwijder het tijdens runtime opschonen van verouderde sleutels en parameters voor het verwijderen van aliassen uit
    upserts/patches van sessierijen. Gereed.

5. Verwijder runtimegedrag voor het JSON-register.
   - Maak lees- en schrijfbewerkingen van het sandboxregister uitsluitend SQLite-gebaseerd. Gereed.
   - Importeer monolithische en geshardde JSON alleen vanuit de migratiestap. Gereed.
   - Verwijder vergrendelingen van het geshardde register en JSON-schrijfbewerkingen. Gereed.

- Behoud één getypeerde registertabel in plaats van registerrijen als generieke
  ondoorzichtige JSON op te slaan als de vorm operationele status voor hot paths blijft. Gereed.

6. Verwijder sessiemutatie in de vorm van bestandsvergrendeling.
   - Gereed voor het tijdens runtime maken van vergrendelingen en runtime-API's voor vergrendelingen.
   - Het zelfstandige verouderde `.jsonl.lock`-opruimtraject van doctor is verwijderd.
   - Statusintegriteit heeft niet langer een afzonderlijk pad voor het opschonen van verweesde transcriptbestanden;
     de doctormigratie importeert/verwijdert verouderde JSONL-bronnen op één plaats.
   - Coördinatie van Gateway-singletons gebruikt getypeerde SQLite-rijen `state_leases` onder
     `gateway_locks` en stelt niet langer een naad voor een map met bestandsvergrendelingen beschikbaar.
   - Generieke deduplicatiepersistentie van de plugin-SDK gebruikt niet langer bestandsvergrendelingen of JSON-
     bestanden; deze schrijft gedeelde SQLite-rijen voor pluginstatus. Gereed.
   - QMD-coördinatie gebruikt een gedeelde SQLite-lease voor embeddings en een SQLite-lease
     per agent voor elke schrijver voor verzamelen/bijwerken/insluiten. Runtime maakt niet langer
     `qmd/embed.lock.lock` of `agents/<agentId>/qmd-write.lock.lock` aan;
     Doctor verwijdert alleen met zekerheid verouderde buiten gebruik gestelde sidecars. Gereed.

7. Maak workers databasebewust.
   - Workers openen hun eigen SQLite-verbindingen.
   - De ouder beheert levering, kanaalcallbacks en configuratie.
   - De worker ontvangt agent-id, run-id, bestandssysteemmodus en de identiteit van het DB-register,
     geen live handles.
   - `vfs-only` blijft experimenteel en gebruikt de agentdatabase als opslagroot.
   - Behoud eerst één worker per actieve run. Pooling kan wachten totdat de levensduur
     en het annuleringsgedrag van DB-verbindingen voorspelbaar zijn.

8. Back-upintegratie.
   - Leer back-up globale, agent- en plugindatabases vast te leggen met een online
     back-up gevolgd door offline `VACUUM`. Gereed voor ontdekte `*.sqlite`-bestanden onder het statusartefact;
     pluginschema's waarvoor vereiste eigenaarsmogelijkheden niet beschikbaar zijn, worden bij twijfel geweigerd.
   - Voeg back-upverificatie toe voor canonieke SQLite-integriteit en schema-identiteit,
     plus generieke validatie van de bestandsvorm voor speciale pluginsnapshots. Gereed voor
     het maken van back-ups en standaardverificatie van archieven.
   - Leg metadata van back-upruns vast in SQLite. Gereed via de gedeelde tabel `backup_runs`
     met archiefpad, status en manifest-JSON.
   - Voeg herstel vanuit geverifieerde archiefsnapshots toe. Gereed: `openclaw backup
restore` valideert vóór extractie, gebruikt het genormaliseerde
     manifest van de verifier, ondersteunt `--dry-run` en vereist `--yes` voordat
     vastgelegde bronpaden worden vervangen.
   - Neem VFS-/werkruimte-export alleen op wanneer daarom wordt gevraagd; exporteer interne sessiegegevens
     niet als JSON of JSONL.

9. Verwijder verouderde tests en code. Gereed voor de bekende runtimesessie-oppervlakken.

- Verwijder tests die het tijdens runtime aanmaken van `sessions.json`- of transcript-
  JSONL-bestanden verifiëren. Gereed voor de kernsessieopslag, chat, Gateway-transcriptgebeurtenissen,
  voorbeeldweergave, levenscyclus, updates van sessievermeldingen door opdrachten, reset/tracering van automatische antwoorden en
  dreaming-fixtures van memory-core, routering van goedkeuringsdoelen, herstel van sessietranscripten,
  herstel van beveiligingsmachtigingen, trajectexport en sessie-export.
  Transcripttests voor Active Memory verifiëren nu SQLite-bereiken en dat er geen tijdelijke of
  persistente JSONL-bestanden worden aangemaakt.
  De oude regressietest voor het opschonen van Heartbeat-transcripten is verwijderd omdat
  runtime JSONL-transcripten niet langer afkapt.
  Tests van het hulpmiddel voor agent-sessielijsten modelleren verouderde `sessions.json`-paden
  niet langer als de Gateway-antwoordvorm; app-/UI-/macOS-tests gebruiken `databasePath`.
  Transcriptgebruiktests van `/status` vullen SQLite-transcriptrijen nu rechtstreeks
  in plaats van JSONL-bestanden te schrijven.
  Levenscyclustests van Gateway-sessies gebruiken nu rechtstreeks helpers voor het vullen van SQLite-transcripten;
  de oude fixturevorm voor sessiebestanden met één regel is verdwenen uit de dekking
  voor resetten en verwijderen.
  `sessions.delete` retourneert niet langer een `archived: []`-veld uit het bestandstijdperk; verwijdering
  rapporteert alleen het resultaat van de rijmutatie. De oude optie `deleteTranscript` is
  ook verdwenen: bij het verwijderen van een sessie wordt de canonieke `sessions`-root verwijderd en
  kan SQLite sessiegebonden transcript-, snapshot- en trajectrijen trapsgewijs verwijderen, zodat geen
  aanroeper verweesde transcripten kan achterlaten of een opruimvertakking kan vergeten.
  Tests voor trajectvastlegging door de context-engine lezen nu `trajectory_runtime_events`-
  rijen uit een geïsoleerde agentdatabase in plaats van
  `session.trajectory.jsonl` te lezen.
  Seed-scripts voor Docker MCP-kanalen vullen nu rechtstreeks SQLite-rijen. Rechtstreekse
  schrijfbewerkingen naar `sessions.json` zijn beperkt tot doctor-fixtures.
  Tool Search Gateway E2E leest bewijs van toolaanroepen uit SQLite-transcriptrijen
  in plaats van `agents/<agentId>/sessions/*.jsonl`-bestanden te scannen.
  Hostgebeurtenissen van memory-core en tijdelijke sessiecorpusrijen bevinden zich nu in gedeelde
  SQLite-pluginstatus; `events.jsonl` en `session-corpus/*.txt` zijn alleen invoer voor verouderde
  doctormigraties. Actieve rijen gebruiken virtuele `memory/session-ingestion/`-
  paden, niet `.dreams/session-corpus`. De oude herstelmodule voor dreaming van memory-core
  en de bijbehorende CLI-/Gateway-tests zijn verwijderd omdat runtime niet langer
  verantwoordelijk is voor herstel van bestandsarchieven voor dat corpus. Tests voor de bridge/publieke artefacten
  van memory-core tonen `.dreams/events.jsonl` niet langer; ze
  gebruiken de door SQLite ondersteunde virtuele JSON-artefactnaam.
  Publieke SDK-/Codex-testdocumentatie spreekt nu over SQLite-sessiestatus in plaats van sessie-
  bestanden, en het voorbeeld voor een kanaalbeurt stelt niet langer een argument `storePath` beschikbaar.
  Matrix-synchronisatiestatus gebruikt nu rechtstreeks de SQLite-opslag voor pluginstatus. Actieve
  client-/runtimecontracten geven een opslagroot voor een account door, niet een `bot-storage.json`-
  pad, en doctor importeert verouderde `bot-storage.json` in SQLite voordat
  de bron wordt verwijderd. Matrix-scenario's voor herstarten/destructieve bewerkingen in QA Lab muteren nu rechtstreeks de SQLite-synchronisatie-
  rij in plaats van nepbestanden `bot-storage.json` aan te maken of te verwijderen, en
  de E2EE-onderlaag geeft een synchronisatieopslagroot door in plaats van een nep-
  pad `sync-store.json`.
  De selectie van Matrix-opslagroots beoordeelt roots niet langer op basis van verouderde JSON-bestanden voor synchronisatie/threads;
  deze gebruikt duurzame rootmetadata plus echte cryptografische status.
  De runtimetestsuite voor de SQLite-sessiebackend maakt niet langer een
  `sessions.json` na; verouderde bronfixtures bevinden zich nu in de doctor-
  tests die ze importeren.
  Tests voor Gateway-sessies stellen niet langer een helper `createSessionStoreDir` of
  ongebruikte tijdelijke instellingen voor het pad van de sessieopslag beschikbaar; fixturemappen zijn expliciet en rechtstreekse
  rijinstellingen gebruiken de naamgeving van SQLite-sessierijen.
  Dekking voor de alleen door doctor gebruikte JSON5-parser voor sessieopslag is verplaatst uit infrastructuurtests
  naar tests voor doctormigratie, zodat runtimetestsuites niet langer verantwoordelijk zijn voor het parseren van verouderde
  sessiebestanden.
  Runtimetests voor SSO/wachtende uploads van Microsoft Teams bevatten niet langer JSON-sidecar-
  fixtures of parsers; het parseren van verouderde SSO-tokens bevindt zich alleen in de plugin-
  migratiemodule. Telegram-tests vullen niet langer nep-opslagpaden `/tmp/*.json`;
  ze resetten rechtstreeks de door SQLite ondersteunde berichtencache. De generieke
  OpenClaw-helper voor teststatus stelt niet langer een verouderde `auth-profiles.json`-
  schrijver beschikbaar; doctortests voor authmigratie beheren die fixture lokaal.
  Runtimetests voor TUI-aanwijzers naar de laatste sessie, uitvoeringsgoedkeuringen, Active Memory-
  schakelaars, Matrix-deduplicatie/opstartverificatie, bronsynchronisatie van Memory Wiki,
  bindingen van huidige gesprekken, onboardingauthenticatie en import van Hermes-geheimen maken
  niet langer oude sidecarbestanden aan en verifiëren niet langer dat oude bestandsnamen ontbreken. Ze
  bewijzen gedrag via SQLite-rijen en publieke opslag-API's; doctor-/migratie-
  tests zijn de enige plaats waar verouderde bronbestandsnamen thuishoren.
  Runtimetests voor koppeling van apparaten/Nodes, allowFrom van kanalen, herstartintenties,
  overdracht bij herstart, vermeldingen in de leveringswachtrij voor sessies, configuratiegezondheid, iMessage-
  caches, Cron-taken, PI-transcriptheaders, subagentregisters en beheerde
  afbeeldingsbijlagen maken ook niet langer buiten gebruik gestelde JSON-/JSONL-bestanden aan alleen om te bewijzen
  dat ze worden genegeerd of ontbreken.
  PI-herstel bij overflow heeft niet langer een terugval naar herschrijven/afkappen door SessionManager:
  het afkappen van toolresultaten en het herschrijven van transcripten door de context-engine muteren
  SQLite-transcriptrijen en vernieuwen vervolgens de actieve promptstatus vanuit de database.
  Gepersisteerde toevoegingen van SessionManager-berichten delegeren aan de atomaire SQLite-
  helper voor het toevoegen van transcripten voor ouderselectie en idempotentie. Normale
  toevoegingen van metadata/aangepaste vermeldingen selecteren de huidige ouder eveneens binnen SQLite, zodat
  verouderde managerinstanties geen races in ouderketens van vóór SQLite opnieuw introduceren.
  Synthetische opschoning van de PI-staart voor controles halverwege een beurt en `sessions_yield` kort
  nu rechtstreeks SQLite-transcriptstatus in; de oude SessionManager-bridge voor staartverwijdering
  en de bijbehorende tests zijn verwijderd.
  Ook het vastleggen van Compaction-controlepunten maakt uitsluitend snapshots vanuit SQLite; aanroepers
  geven niet langer een live SessionManager door als alternatieve transcriptbron.
- Behoud tests die verouderde bestanden vullen uitsluitend voor migratie.
- Bewijs via JSON-bestanden is vervangen door bewijs via SQL-rijen voor actieve runtime-
  oppervlakken.

- Voeg statische verboden toe voor runtimeschrijfbewerkingen naar verouderde JSON-paden voor sessies/caches.
  Gereed voor de repositorycontrole.

10. Maak het migratierapport controleerbaar.
    - Leg migratieruns vast in SQLite met begin-/eindtijdstempels, bron-
      paden, bronhashes, aantallen, waarschuwingen en back-uppad.
      Gereed: uitvoeringen van migratie van verouderde status bewaren nu een `migration_runs`-
      rapport met inventaris van bronpaden/-tabellen, SHA-256 van bronbestanden, groottes,
      recordaantallen, waarschuwingen en back-uppad.
      Gereed: uitvoeringen van migratie van verouderde status bewaren ook `migration_sources`-
      rijen voor controle op bronniveau en toekomstige beslissingen over overslaan/aanvullen.
    - Maak toepassen idempotent. Opnieuw uitvoeren na een gedeeltelijke import moet een
      reeds geïmporteerde bron overslaan of samenvoegen op basis van een stabiele sleutel.
      Gereed: sessie-indexen, transcripten, leveringswachtrijen, pluginstatus, taak-
      registers en globale SQLite-rijen die eigendom zijn van agents worden geïmporteerd via stabiele sleutels of
      upsert-/vervangingssemantiek, zodat nieuwe uitvoeringen samenvoegen zonder duurzame
      rijen te dupliceren.
    - Bij mislukte imports moet het oorspronkelijke bronbestand op zijn plaats blijven.
      Gereed: bij mislukte transcriptimports blijft de oorspronkelijke JSONL-bron nu op
      het gedetecteerde pad staan en `migration_sources` registreert de bron als
      `warning` met `removed_source=0` voor de volgende doctoruitvoering.

## Prestatieregels

- Eén verbinding per thread/proces is prima; deel geen handles tussen
  workers.
- Gebruik WAL, `foreign_keys=ON`, een busy-time-out van 5s en korte `BEGIN IMMEDIATE`
  schrijftransacties. Voeg boven op SQLite's enkele busy-wachttijd geen synchrone
  nieuwe vergrendelingspogingen toe.
- Houd helpers voor schrijftransacties synchroon, tenzij/totdat een asynchrone transactie-
  API expliciete semantiek voor mutex/backpressure toevoegt.
- Houd schrijfbewerkingen voor levering aan het bovenliggende proces klein en transactioneel.
- Vermijd het volledig herschrijven van de opslag; gebruik upsert/delete op rijniveau.
- Voeg indexen toe voor lijsten per agent, lijsten per sessie, bijgewerkt-op, uitvoerings-id en
  vervalpaden voordat je hot code verplaatst.
- Sla grote artefacten, media en vectoren op als BLOB's of opgedeelde BLOB-rijen, niet
  als base64 of JSON met numerieke arrays.
- Houd ondoorzichtige vermeldingen voor pluginstatus klein en afgebakend.
- Voeg SQL-opschoning voor TTL/verval toe in plaats van opschoning van het bestandssysteem.
  Voltooid voor runtimeopslag die eigendom is van de database: media, pluginstatus, plugin-BLOB's,
  permanente deduplicatie en agentcache verlopen allemaal via SQLite-rijen. De resterende
  opschoning van het bestandssysteem is beperkt tot tijdelijke materialisaties of expliciete
  verwijderingsopdrachten.

## Statische verboden

Voeg een repocontrole toe die nieuwe runtimeschrijfbewerkingen naar verouderde statuspaden afkeurt:

- `sessions.json`
- `*.trajectory.jsonl` behalve gematerialiseerde uitvoer van ondersteuningsbundels
- `.acp-stream.jsonl`
- `acp/event-ledger.json`
- `cache/*.json` runtimecachebestanden
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
- naastliggende `<workspace>.attested`
- Matrix `credentials*.json` en `recovery-key.json`
- `cron/runs/*.jsonl`
- `cron/jobs.json`
- `jobs-state.json`
- `device-pair-notify.json`
- `devices/pending.json` / `devices/paired.json` / `devices/bootstrap.json`
  (buiten gebruik gesteld in 2026.7: runtimeopslag is `device_pairing_*` /
  `device_bootstrap_tokens` in de gedeelde statusdatabase; gekoppelde records worden bij
  het starten van de Gateway geïmporteerd, tijdelijke pending-/bootstrap-rijen worden verwijderd)
- `nodes/pending.json` / `nodes/paired.json` (buiten gebruik gesteld in 2026.7: bij het starten van de Gateway samengevoegd met gekoppelde apparaatrecords)
- `identity/device.json`
- `identity/device-auth.json` (buiten gebruik gesteld; import alleen via Doctor naar `device_auth_tokens`)
- `push/web-push-subscriptions.json` (buiten gebruik gesteld; import alleen via Doctor naar `web_push_subscriptions`)
- `push/vapid-keys.json` (buiten gebruik gesteld; import alleen via Doctor naar `web_push_vapid_keys`)
- `push/apns-registrations.json` (buiten gebruik gesteld; import alleen via Doctor naar `apns_registrations`)
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
- JSON-bestanden voor shards van het sandboxregister
- `plugin-state/state.sqlite`
- ad-hoc `openclaw-state.sqlite` runtime-sidecars
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
- Browserprofieldecoratie `.openclaw-profile-decorated`
- `SessionManager.open(...)` bestandsgebaseerde sessieopeners
- `SessionManager.listAll(...)` en `TranscriptSessionManager.listAll(...)`
  façades voor transcriptlijsten
- `SessionManager.forkFromSession(...)` en
  `TranscriptSessionManager.forkFromSession(...)` façades voor transcriptforks
- `SessionManager.newSession(...)` en `TranscriptSessionManager.newSession(...)`
  façades voor vervanging van veranderlijke sessies
- `SessionManager.createBranchedSession(...)` en
  `TranscriptSessionManager.createBranchedSession(...)` façades voor vertakte sessies

Het verbod moet toestaan dat tests verouderde fixtures maken en dat migratiecode
verouderde bestandsbronnen leest/importeert/verwijdert. Niet-uitgebrachte SQLite-sidecars blijven verboden
en krijgen geen importtoestemmingen voor Doctor.

## Voltooiingscriteria

- Schrijfbewerkingen van runtimegegevens en caches gaan naar de globale SQLite-database of die van de agent.
- De runtime schrijft niet langer sessie-indexen, transcript-JSONL, JSON voor het sandboxregister,
  SQLite-taaksidecars of SQLite-sidecars voor pluginstatus. De niet-uitgebrachte importfuncties voor SQLite-sidecars
  van taken en pluginstatus zijn verwijderd.
- Import van verouderde bestanden vindt uitsluitend via Doctor plaats.
- Een back-up produceert één archief met compacte SQLite-snapshots en bewijs van integriteit.
- Agentworkers kunnen werken met schijfopslag, tijdelijke VFS-opslag of experimentele
  opslag die uitsluitend VFS gebruikt.
- Configuratiebestanden en expliciete referentiebestanden blijven de enige verwachte permanente
  niet-databasebesturingsbestanden.
- Repocontroles voorkomen dat verouderde runtimebestandsopslag opnieuw wordt ingevoerd.
