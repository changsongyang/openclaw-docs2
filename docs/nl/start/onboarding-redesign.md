---
read_when:
    - Je implementeert of beoordeelt een fase van het herontwerp van de onboarding
summary: Implementatieplan voor het herontwerp van de onboarding voor beheerders (levend document)
title: Herontwerp van onboarding
x-i18n:
    generated_at: "2026-07-27T06:12:39Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: f892991583d0b77a670e9bf7aa5a0c74af3b3eac9e7b0448706486254eb7e2a0
    source_path: start/onboarding-redesign.md
    workflow: 16
---

# Implementatieplan voor het herontwerp van de onboarding

> **Levend document.** Deze pagina volgt het herontwerp van de onboarding door de beheerder
> op implementatieniveau en wordt bijgewerkt wanneer elke fase wordt opgeleverd. Wanneer de laatste fase
> is samengevoegd, wordt deze pagina herschreven als de gebruikersgerichte onboardinghandleiding en toegevoegd
> aan de documentatienavigatie. Tot die tijd staat deze bewust niet in `docs.json`.

## Hoofddoel

Een niet-technische gebruiker typt `openclaw onboard` (of opent de app) en wordt begroet
door één gesprekspartner — OpenClaw, de systeembeheerder ("beheerder" is
alleen de interne naam; de gebruiker ziet altijd "OpenClaw") — die de AI van de gebruiker vindt,
alles instelt met aangekondigde standaardwaarden in plaats van vragen, de agent
tot leven brengt tijdens een zichtbaar identiteitsmoment en daarna voor altijd bereikbaar blijft als
beheerder van het systeem. Standaard magisch, één toestemmingsgrens, geen doodlopende wegen.

Ontwerpprincipes (besloten, niet zomaar opnieuw ter discussie stellen):

- **Aangekondigde standaardwaarden die eenvoudig ongedaan kunnen worden gemaakt** vervangen blokkerende vragen. De enige
  harde vereiste is werkende inferentie; al het andere is een aanbod.
- **Vraag nul is de toestemmingsgrens**: "Volledige toegang" (aanbevolen) betekent
  dat detectie stil en automatisch verloopt; bij "Eerst vragen" vereist elke vorm van detectie — het
  scannen van AI, apps en geheugenbronnen — één
  expliciet ja, met een volledig handmatig traject waarin nooit wordt gescand.
- **Gesprek als gebruikersinterface met progressieve intelligentie**: het beheerdersoppervlak
  bestaat al voordat AI werkt (gescript dialoog), wordt door een model aangestuurd zodra
  een route is geverifieerd en meldt dit zichtbaar. Het simuleert nooit intelligentie:
  vrije tekstinvoer voordat een route is geverifieerd krijgt een gepast "laat me eerst
  mijn brein aan de praat krijgen".
- **Het uitkomen is een ceremonie**: dezelfde thread, wisseling van avatar, de agent geeft zichzelf
  een naam en kiest een eigen gezicht. De beheerder legt de hiërarchie één keer uit: "vraag mij
  naar het systeem, of vraag het gewoon aan je agent — die geeft het door."
- **Vertrouwen is ingedeeld op basis van de bron**: vermeldingen uit de officiële catalogus mogen vooraf zijn geselecteerd;
  Skills van derden uit ClawHub worden nooit vooraf geselecteerd, ongeacht de
  rangschikking door het model, en hun labels vermelden dat ze de code van de uitgever installeren.
- **Geconfigureerde installaties zijn heilig**: het opnieuw uitvoeren van de onboarding is een verificatieronde.
  De installatie wordt nooit opnieuw toegepast en de Gateway-service wordt nooit opnieuw gestart.
- **De terminal is de terugvaloptie, geen vraag**: geef de voorkeur aan het browserdashboard
  wanneer een Gateway bereikbaar is; vraag nooit "terminal of browser?".
- **Zwakke modellen krijgen een beperkt oppervlak** (automatisch `localModelLean`), uitgelegd in
  gewone taal — nooit in termen van tools, codemodus of contextvensters.

## Huidige uitgebrachte flow (na fasen 1-3)

`openclaw onboard` bij een nieuwe macOS-installatie, ideaal traject — in totaal viermaal Enter:

1. Beveiligingsmelding → eenmaal Enter ter bevestiging (opgeslagen; wordt nooit opnieuw gevraagd).
2. **Vraag nul**: "Hoe moet ik alles instellen?" — Volledige toegang (aanbevolen)
   of Eerst vragen. Opgeslagen als `wizard.accessMode`; bij herhaalde uitvoeringen wordt standaard de opgeslagen
   keuze gebruikt. Beveiligd + "handmatig configureren" opent de providerkiezer zonder
   te scannen en slaat ook het scannen van geheugenbronnen over.
3. **Detectietheater**: detecteert programmeer-CLI's, omgevingssleutels en lokale runtimes;
   maakt een kwinkslag wanneer programmeeragents worden gevonden; test kandidaten live op volgorde en
   verzamelt mislukkingen stilletjes in één samenvattingsregel (details achter "Andere
   opties bekijken"). De eerste werkende route wordt aangekondigd als standaardwaarde met een
   sneltoets naar de volledige kiezer; bij verkennen en overslaan blijft de
   werkende route behouden.
4. Aanbod om geheugen te importeren (Claude Code / Codex / Hermes), overgeslagen wanneer detectie
   is geweigerd.
5. Alleen nieuwe installaties: het standaardinstallatieplan wordt automatisch toegepast
   (werkruimte, Gateway-service, sessies — hetzelfde plan dat door het "ja" in het gesprek
   wordt uitgevoerd). Geconfigureerde installaties tonen "al ingesteld" en wijzigen de
   service nooit.
6. **App-aanbevelingen**: geïnstalleerde apps worden door het geverifieerde model gekoppeld
   aan officiële catalogi + ClawHub; officiële kanaalplugins zijn
   vooraf aangevinkt, Skills van derden vereisen aanmelding en hebben een waarschuwingslabel. Kan worden overgeslagen;
   uitschakeloptie `wizard.appRecommendations`.
7. **Uitkomen**: wanneer een Gateway bereikbaar is, wordt de browseroverdracht geopend (GUI) of
   wordt (headless/SSH) de dashboard-URL getoond en gewacht tot de Control UI
   verbinding maakt — "Dashboard verbonden — doorgaan in je browser." Anders, of
   met `--tui`, wordt de terminal-TUI geopend met het bootstrap-uitkomstbericht
   als begintekst en stelt de agent zichzelf voor.

Onboarding voor een externe Gateway behoudt de bestaande gespreksoverdracht
(`handoffMode: "chat"`); de installatie moet op de externe Gateway worden toegepast.

## Fasen

| #   | Fase                                                                                                                                                      | Oppervlak             | Status                                                                                                                            |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Plugin-aanbevelingen voor geïnstalleerde apps (scan, kandidaten, AI-koppeling, wizardstap, `device.apps`-Node-opdracht)                                   | klassieke + begeleide CLI | samengevoegd ([#109668](https://github.com/openclaw/openclaw/pull/109668))                                                     |
| 2   | Hoofdstructuur van de CLI-beheerder (vraag nul, detectietheater, automatisch toepassen + uitkomen)                                                        | begeleide CLI         | samengevoegd ([`a83ed13204f1`](https://github.com/openclaw/openclaw/commit/a83ed13204f118adf1009e5ac88d5afe1905b86c))              |
| 3   | Browsergerichte overdracht (detectie van GUI-sessie, wachten op dashboardverbinding, TUI als terugvaloptie)                                               | CLI → web             | samengevoegd ([#110054](https://github.com/openclaw/openclaw/pull/110054))                                                         |
| 4   | Weboppervlak voor de beheerder (optiekaarten, getypt `question`-veld op `openclaw.chat`, spiegeling van wizardstappen, overdracht bij eerste gebruik) | Control UI            | samengevoegd ([#110141](https://github.com/openclaw/openclaw/pull/110141), [#110242](https://github.com/openclaw/openclaw/pull/110242)) |
| 5   | Uitkomen en bootstrap (opslag van aanbevelingen met eenmalige semantiek, geboortesequentie voor zelfbenoeming, automatische uitkomstoverdracht na nieuwe installatie; avatarladder uitgesteld) | agentbootstrap        | samengevoegd ([#110173](https://github.com/openclaw/openclaw/pull/110173), [#110331](https://github.com/openclaw/openclaw/pull/110331)) |
| 6   | Aanwezigheid van de beheerder PR1 (vastgezet item in zijbalk, OpenClaw vragen in Instellingen, beheerdersbegroeting met normale interface; gebeurteniscommentaar en oproep via kanaal volgen in PR2) | web + kanalen         | samengevoegd ([#110269](https://github.com/openclaw/openclaw/pull/110269))                                                         |
| 7   | Veerkracht (beheerder bereikbaar bij defecte configuratie, herstel van gedeeltelijk werkende oppervlakken, automatische doctor)                           | Gateway               | vervolg                                                                                                                           |

## Implementatienotities per fase

### Fase 1 — app-aanbevelingen (PR #109668)

- Scanner: `src/infra/installed-apps.ts` (macOS-inventarisatie zonder TCC; volgt
  symbolisch gekoppelde `.app`-bundels).
- Kandidaten: officiële catalogi + zoeken in ClawHub, totaalbudget van 20s, soepele
  offline degradatie naar uitsluitend cataloguskandidaten. Catalogusvermeldingen zijn pakketmanifesten
  zonder `id` op het hoogste niveau — kandidaten worden geïndexeerd op de opgeloste
  Plugin-id (regressiegetest met de echte gebundelde catalogi; indexering op
  `entry.id` liet ooit de hele catalogus samenvallen en verwijderde elke officiële
  aanbeveling).
- AI-koppeling: één voltooiing op de geverifieerde route
  (`src/system-agent/setup-app-recommendations.ts`); geen beheerde toewijzing van bundel-id's —
  het model wijst toevallige naamovereenkomsten af. De uitvoer wordt begrensd door het eigen
  `maxTokens`-budget van het opgeloste model (de streaminglaag past dit toe wanneer geen
  expliciete limiet wordt doorgegeven).
- **Beveiliging van de toeleveringsketen**: de tekst van ClawHub-vermeldingen wordt door de uitgever beheerd en
  bereikt de prompt van de koppelaar, zodat een vermelding zichzelf als
  "aanbevolen" kan promoten. Alleen vermeldingen uit de officiële catalogus mogen vooraf worden geselecteerd; ClawHub-
  Skills vereisen altijd expliciet aanvinken en krijgen het label "Skill van een derde uit ClawHub;
  installeert de code van de uitgever".
- Node-opdracht `device.apps` (TS-nodehost, gelijkwaardigheid met Android-envelope), delen
  standaard uitgeschakeld; uitschakeloptie voor Gateway `wizard.appRecommendations`.
- De levering bevindt zich in de klassieke wizard en de begeleide beheerdersflow
  (`src/wizard/setup.app-recommendations.ts`); opnieuw richten op het einde van de bootstrap
  blijft onderdeel van fase 5 (de service accepteert al een injecteerbare inventarisbron).
  Eenmalige semantiek (alleen aanbieden totdat het wordt geaccepteerd, opgeslagen scan) wordt ook
  met de opslag van fase 5 opgeleverd; momenteel wordt het aanbod bij herhaalde uitvoering opnieuw gedaan.
- Ook opgelost: aangepaste `completeSetupInference`-prompts nemen niet langer de
  uitvoerlimiet van 32 tokens van de verificatieprobe over (`SETUP_INFERENCE_TEST_MAX_TOKENS`
  geldt alleen voor de probe "antwoord OK").

### Fase 2 — hoofdstructuur van de CLI-beheerder (PR #109841)

- Herwerking van de flow in `src/commands/onboard-guided.ts`; onboarding voor een externe Gateway
  behoudt de bestaande chatoverdracht via `handoffMode: "chat"`.
- Vraag nul slaat `wizard.accessMode` ("full" | "guarded") op; herhaalde uitvoeringen
  gebruiken standaard de opgeslagen keuze (het accepteren van de standaardwaarde kan beveiligd nooit ongemerkt
  verlagen naar volledig). Beveiligd + handmatig gebruikt
  `listManualSetupInferenceOptions` (alleen configuratie/manifesten, geen probes) en
  slaat het scannen van geheugenbronnen over.
- Detectie: stille verzameling van mislukkingen (één samenvattingsregel; details achter
  "Andere opties bekijken"), kwinkslag over programmeeragent, aangekondigde standaardroute. Sessietellingen
  in de kwinkslag worden uitgesteld (voorlopig alleen kwalitatief) totdat er een goedkope
  interface voor sessietellingen bestaat.
- Nieuwe installaties: `applySystemAgentSetup` (het deterministische "ja" in het gesprek),
  daarna uitkomen via `launchTuiCli`, vooraf ingevuld met het bootstrapbericht.
  Geconfigureerde installaties (bestaand model of bestaande Gateway-configuratie — tijdstempels van de wizard
  bewijzen niets, omdat ze worden gedeeld met configureren/doctor):
  alleen verificatie — niet toepassen, geen herstart van de Gateway-service. Bij mislukking van het toepassen
  wordt teruggevallen op de gesprekschat.

### Fase 3 — browsergerichte overdracht (PR #110054, samengevoegd)

- `src/commands/onboard-browser-handoff.ts` is verantwoordelijk voor zuivere detectie van grafische sessies
  (`SSH_CONNECTION`/`SSH_TTY`; `DISPLAY`/`WAYLAND_DISPLAY` op Linux)
  en de wachttijd van 60 seconden voor de GUI / 300 seconden voor SSH. Begeleide onboarding
  schakelt de overdracht momenteel alleen in op macOS; `--tui` en andere platforms behouden de
  terminaluitweg. Inschakeling voor Linux/Windows volgt later.
- Dashboardlinks gebruiken dezelfde helpers `resolveAdvertisedControlUiLinks`,
  `resolveLocalControlUiProbeLinks` en `buildOnboardingControlUiUrl`
  als de klassieke afronding. Voor het starten van de browser wordt de gedeelde helper `openUrl` gebruikt.
- De gereedheidscontrole peilt de bestaande RPC `system-presence` als een **loopbackclient
  in CLI-modus die het geconfigureerde gedeelde geheim presenteert** — het vertrouwde pad dat elke
  opdracht `openclaw` gebruikt. Een onbewerkte Control UI-client met gedeelde authenticatie wordt
  op SecretRef-gateways geweigerd met "apparaatidentiteit vereist". De voorafgaande
  bereikbaarheidscontrole herleidt hetzelfde doel (en geheim) als de wachtlus, zodat de
  poort en de wachtroutine het nooit oneens kunnen zijn over authenticatie. De overdracht wordt alleen voltooid
  wanneer een verbonden aanwezigheidsrij voor `openclaw-control-ui`/`webchat` nieuw is
  ten opzichte van de uitgangssituatie vóór het starten (een reeds geopend dashboard kan
  de overdracht niet voltooien).
- `gateway.controlUi.enabled: false` breekt af voordat een URL wordt getoond.
- End-to-end bewezen met een geïsoleerde gateway met dezelfde configuratie: URL afdrukken → echte
  browserverbinding → "Dashboard verbonden — ga verder in je browser" → geen
  terminaluitweg. Een eerdere blokkering wegens "token komt niet overeen" was een artefact van de
  testopstelling — zie het testdraaiboek hieronder.

### Fase 4 — webinterface voor de beheerder (samengevoegd: #110141, #110242)

- Pagina `/custodian` via `openclaw.chat` met het optiekaartonderdeel
  (2-4 kaarten, maximaal één aanbevolen, altijd over te slaan); onboardingkader via
  `?onboarding=1`; na voltooiing van de eerste modelconfiguratie volgt de overdracht ernaartoe.
- Gestructureerde vragen zijn een getypeerd, aanvullend veld `question` op
  `SystemAgentChatResult` (tekst `reply` per optie; proza staat altijd zelfstandig
  voor de macOS-app/TUI). Producenten: beide welkomstvarianten van de onboarding en
  selectie-/bevestigingsstappen van de gehoste wizard met 2-4 gesloten opties — echte kanaalwizards
  worden als kaarten weergegeven. De tijdelijke oplossing met tekenreeksmarkeringen uit PR1 is verwijderd.
- Sessie-eigendom is beperkt tot de gateway-URL + elke aangeboden referentie
  (token, wachtwoord, bootstrap-token, opgeslagen apparaattoken — blijft behouden bij
  tijdelijke uitval van hello); mislukte gebruikersbeurten kunnen nooit opnieuw worden afgespeeld; gevoelige
  invoer wordt letterlijk verzonden en in het transcript gemaskeerd.

### Fase 5 — uitweg en bootstrap (samengevoegd: #110173, #110331)

- De beheerder maakt een naamloze agent aan (toolaanroep); de bootstrap van de agent begint
  met het kiezen van een eigen naam. PR1 levert de ceremonie, beperkt tot drie stappen (naam → zielsregel
  → vraag over Skills), en stelt de ladder voor een zelfgetekende avatar/beeldgeneratie
  (door het model gegenereerde kandidaten → vooraf ingestelde tekens → logo behouden) uit tot later. Dezelfde
  thread, avatarwissel; het klauwteken blijft voorbehouden aan de beheerder. De
  overeengekomen identiteit wordt tweemaal opgeslagen: in `IDENTITY.md`/`SOUL.md` (wat de agent
  leest) en via `openclaw agents set-identity` (wat kanalen en de UI
  weergeven).
- Aanbevelingen (service uit fase 1, opgeslagen scan met eenmalige semantiek) verschijnen als
  de laatste bootstrapstap voordat het bootstrapbestand wordt verwijderd: "minimale set
  of maximaal gemak?" De bootstrap leest het opgeslagen aanbod via
  `openclaw onboard recommendations --json` (alleen ondoorzichtige installatie-ID's) en
  bevestigt het nadat de keuze is verwerkt, zodat er nooit opnieuw naar wordt gevraagd. Knoppen voor
  kanaalverbinding bevatten configuratiedraaiboeken per kanaal; de agent verzamelt
  referenties in een gesprek en geeft configuratieschrijfbewerkingen door aan de beheerder
  ("OpenClaw vragen…" is de canonieke formulering).
- Zelfleren wordt gevraagd, niet aangekondigd, en geldt tevens als toestemming voor de Skills-workshop;
  beschrijf de controles van ClawHub op releasevertrouwen, scans, verificatie en integriteit,
  plus de waarschuwing over uitgeverscode — suggereer nooit dat elke release is ondertekend.
- Automatische uitweg uitgebracht: het toepassen van de configuratie bij een nieuwe installatie kondigt de uitweg aan en
  draagt over (terminal-TUI / `open-agent` voor gatewayclients); de webpagina
  opent in de agentchat met het concept "Word wakker, mijn vriend!" vooraf ingevuld. De
  overdracht vindt alleen plaats na een geslaagde verificatie na het schrijven. Het aanbieden
  van nul agents na verwijdering (in plaats van automatisch) blijft een latere verfijning.

### Fase 6 — aanwezigheid van de beheerder (PR1 samengevoegd: #110269; commentaar/oproepen vallen onder PR2)

- Uitgebracht in PR1: standaard vastgemaakt item "OpenClaw" in de zijbalk (nieuwe profielen;
  bestaande gebruikers behouden opgeslagen vastgemaakte items en bereiken het via aanpassen/More), "OpenClaw
  vragen" als eerste item onder Instellingen en bezoeken aan `/custodian` met het normale kader
  die om de begroeting van de beheerder vragen (geen welkomstvariant van de onboarding), waarbij
  Configuratie afsluiten alleen in de onboardingmodus wordt weergegeven. Een vastgezet inlinevenster voor Instellingen
  vereist extractie van de gedeelde gespreksweergave (volgt later).
- Gebeurtenisgestuurd commentaar met anti-Clippy-beperkingen: alleen ingrijpende of
  mislukte wijzigingen, maximaal eenmaal per bezoek aan Instellingen tenzij erom wordt gevraagd. Via dezelfde
  gebeurtenisnaad wordt de beheerder later de stem voor verslechterde authenticatie of defecte
  kanalen.
- Kanalen: onzichtbaar bij dagelijks gebruik (de agent geeft door); bereikbaar via een expliciete
  oproep en bij gebeurtenissen waarbij de agent uitvalt in dezelfde thread, met een eigen naam en
  klauwavatar waar het platform dit toestaat.
- Zwak model gedetecteerd tijdens de configuratie: stel `localModelLean` automatisch in en de beheerder
  meldt dit in duidelijke woorden met een aanbod om te upgraden.
- De beheerder kent zijn interne bijnaam ("sommige mensen noemen me de
  beheerder — OpenClaw is ook goed") en verwijst altijd met de naam naar de agent.

### Fase 7 — veerkracht (vereist een beslissing van een eigenaar vóór de bouw)

De oorspronkelijke schets — "de beheerder moet bereikbaar zijn, hoe defect
de configuratie ook is" — botst met het beveiligingsbeleid van de repository: de hoofdhandleiding
stelt dat de Gateway **weigert te starten** wanneer de configuratie structureel ongeldig is,
en alleen fouten van SecretRef-eigenaren leiden tot geconfigureerde maar niet-beschikbare
mogelijkheden. Een interface aanbieden vanuit een ongeldige configuratie is een beleidswijziging,
geen implementatiedetail. Twee bereiken, kies er één:

- **Optie A (aanbevolen, in overeenstemming met het beleid): automatische doctor aan CLI-zijde.** Wanneer het
  starten van een gateway of CLI mislukt vanwege een ongeldige configuratie met een bekende vorm, biedt de CLI
  `openclaw doctor --fix` aan (of voert dit met toestemming uit), probeert vervolgens eenmaal opnieuw en
  rapporteert dit duidelijk. Het gatewaygedrag verandert niet; de beheerder blijft bereikbaar
  via het bestaande pad voor verslechterde SecretRef-functionaliteit en de terminal.
- **Optie B (vereist expliciete goedkeuring van de eigenaar + beveiligingsreview): modus met minimale
  gatewayinterface.** Start bij een structureel ongeldige configuratie een vergrendelde
  interface die alleen het gesprek met de beheerder en doctor-acties aanbiedt. Dit
  herschrijft het op gesloten falen gerichte opstartcontract en moet vóór het schrijven van code
  een eigen beschermingsaanpak voor inkomend verkeer definiëren.

Resterende vervolgpunten uit fasen 4-6 (bijgehouden, niet gepland): ladder voor avatar/beeldgeneratie
voor de uitweg; weergave van het getypeerde veld `question` in de macOS-app; een
vastgezet inlinevenster voor Instellingen voor de beheerder (vereist extractie van de gedeelde gespreksweergave);
gebeurtenisgestuurd commentaar en kanaaloproepen/herstel bij uitval van de agent
(fase 6 PR2); automatische `localModelLean` voor zwakke modellen; of opgeslagen
vastgemaakte zijbalkitems van bestaande gebruikers het OpenClaw-item moeten overnemen.

## Draaiboek voor testen en landen (met vallen en opstaan verkregen; lees vóór fasen 4-6)

- **`OPENCLAW_STATE_DIR` isoleert de Gateway-service niet.** Het
  LaunchAgent-label (`ai.openclaw.gateway`) is machinebreed: een onboardingtest voor een nieuwe installatie
  met een geïsoleerde statusmap zal de service van de echte machine HERSCHRIJVEN en HERSTARTEN
  (wrapperscripts komen in de geïsoleerde map terecht; de volgende
  servicestart mislukt wanneer die map wordt opgeruimd). Herstel na elke test voor een nieuwe installatie
  met `openclaw gateway install --force && openclaw gateway
restart` vanuit de echte omgeving en controleer de plist. Productvervolg:
  servicelabels die aan de statusmap zijn gebonden, of onboarding die een externe service detecteert.
- **Veilige end-to-end-testopstelling**: vul de geïsoleerde configuratie vooraf met een sectie `gateway`
  (zodat onboarding het pad voor een geconfigureerde installatie neemt en de service nooit aanraakt)
  en voer `openclaw gateway run` uit als een gewoon voorgrondproces op
  een vrije poort met een eenvoudig token. Met die opstelling is de lus uit fase 3 bewezen,
  inclusief een echte browserverbinding.
- **Authenticatiepaden verschillen per clientidentiteit, niet alleen per referentie.** Aanwezigheidsgegevens en
  andere operatorleesbewerkingen gebruiken een loopbackclient in CLI-modus met referenties uit dezelfde
  configuratie. Gateways met tokenauthenticatie vereisen het gedeelde geheim; gateways met SecretRef/geen
  authenticatie kunnen zonder token terugvallen op vertrouwde loopbackauthenticatie. Een browserclient
  die als Control UI is geïdentificeerd, heeft een apparaatidentiteit of de loopbacktoekenning
  voor een beveiligde context nodig. Een sonde die zich authenticeert bij een gateway die een
  ANDERE configuratie aanbiedt (zie het LaunchAgent-probleem) mislukt met "token komt niet overeen" — dat
  artefact hield fase 3 kort tegen.
- **Voltooiingssondes**: `runSetupInferenceTest` beperkt de verificatiesonde tot
  32 uitvoertokens; aangepaste prompts omzeilen de limiet en worden begrensd door de eigen
  `maxTokens` van het model. Redeneermodellen verbruiken dat budget eerst met verborgen
  redenering — een beurt zonder tekst betekent meestal dat het budget daar is opgebruikt.
- **Het landen van agents vereist gehoste CI op exact de head.** De zware workflow `CI` wordt
  bij belasting van de organisatie mogelijk niet in de wachtrij geplaatst na pushes; de terugvaloptie voor beheerders is een
  releasepoortdispatch op de PR-branch:

  ```bash
  gh workflow run ci.yml --ref <branch> -f target_ref=<head-sha> -f release_gate=true -f pull_request_number=<pr>
  ```

  De uitvoering moet plaatsvinden op de
  branchreferentie zodat `head_sha` overeenkomt, en de titel wordt
  `CI release gate <sha>`, wat `scripts/verify-pr-hosted-gates.mjs`
  accepteert. Voer daarna `scripts/pr` uit voor voorbereiden/samenvoegen zoals gebruikelijk.

- **Poorten die CI naast gerichte tests afdwingt**: documentatiekaart
  (`pnpm docs:map:gen` na het toevoegen van een documentatiepagina), oxlint (`no-map-spread`,
  `max-lines` — splits bestanden, onderdruk nooit), `check:test-types`, dode code volgens knip
  (exporteer alleen wat productiecode gebruikt; laat tests via openbare API's lopen),
  en de classificatie van live-testshards
  (`test/scripts/test-live-shard.test.ts` moet elke nieuwe `*.live.test.ts` vermelden).

## Beslissingslogboek

- Magische scan met noodstop, niet met toestemming vooraf (fase 1; permanente uitvoer
  meldt het gebruik van het model en ClawHub vóór het scannen, en de opmerking bij de resultaten herhaalt dit).
- Volledige verticale flow inclusief de Node-opdracht `device.apps` (fase 1).
- Skills van derden uit ClawHub zijn nooit vooraf geselecteerd en worden aangeduid als
  het installeren van de code van de uitgever; officiële vermeldingen mogen vooraf zijn aangevinkt
  (fase 1, uitgebrachte beveiligingsstandaard).
- Twee toegangskaarten, niet drie; toestemming is vooraf in de keuze verwerkt (fase 2).
- Automatisch uitkomen met aankondiging, niet via een blokkerende knop (fasen 2/5).
- Browser eerst: het uitkomen via de terminal is de terugvaloptie, nooit een vraag
  „terminal of browser?” (fase 3).
- De beheerder krijgt aanwezigheid in het kanaal (oproepen + herstel), niet alleen via web/CLI
  (fase 6).
- Het uitkomen gebeurt in dezelfde thread met een avatarwissel; na voltooiing gaat de
  app over naar de normale gebruikersinterface (fase 5).
- Het instellingenscherm behoudt de naam "Settings"; de beheerder bevindt zich daar
  (en in de zijbalk) in plaats van dit scherm te vervangen (fase 6).
- Optiekaarten zijn beperkt: 2-4 opties, precies één aanbevolen, altijd
  over te slaan; hetzelfde component wordt gebruikt voor onboarding en de vraagtool van de agent
  (fase 4).
- „OpenClaw vragen…” is de canonieke formulering voor delegeren; souls mogen sfeer toevoegen,
  maar de toelichting van de tool blijft zakelijk (fase 5).
- Gebruikersgerichte tekst gebruikt nooit „codemodus”, „tools” of „contextvenster” bij
  het uitleggen van inkorting voor zwakke modellen (fase 6).

## Bekende hiaten en vervolgacties

- Het LaunchAgent-label is niet beperkt tot de statusmap (testvalkuil hierboven; ook een
  echt producthiaat voor meerdere instanties).
- Eenmalige semantiek voor aanbevelingen en de opgeslagen scan (fase 5); bij herhaalde uitvoeringen
  worden ze momenteel opnieuw aangeboden.
- Browseroverdracht werkt alleen op macOS; ondersteuning voor Linux/Windows volgt nog.
- De kwinkslag over het aantal sessies is kwalitatief; aantallen vereisen een lichte koppeling voor het tellen van sessies.
- Browseroverdracht komt uit op het normale dashboard; een deep-link naar de beheerder
  in onboardingmodus volgt in fase 4.
