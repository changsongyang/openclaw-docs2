---
read_when: You want agent sessions to run on ephemeral cloud machines instead of the Gateway host, or you are configuring cloudWorkers profiles.
sidebarTitle: Cloud Workers
status: active
summary: 'Sessies naar tijdelijke cloudmachines sturen: provisioning, worker-runtime, inference via proxy en streamingresultaten'
title: Cloudworkers
x-i18n:
    generated_at: "2026-07-27T05:45:39Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 5620be5957a20019d4687b3ec935ec1116fdef6ea05e42ab766508d2b54322a2
    source_path: gateway/cloud-workers.md
    workflow: 16
---

Cloudworkers laten een sessie de agentlus uitvoeren op een tijdelijke cloudmachine, terwijl alles over de sessie blijft waar het altijd was: zichtbaar in de zijbalk, live streamend, met het transcript in beheer van de Gateway. De Gateway leaset een machine, installeert daarop een vastgezette kopie van OpenClaw, synchroniseert de werkruimte van de sessie en draagt de beurtlus over aan een beperkt `openclaw worker`-proces. Modelaanroepen worden via de Gateway geproxyd, zodat providerreferenties je machine nooit verlaten. Promptcaching blijft werken omdat de provider één continue stroom ziet.

Wanneer het werk klaar is (of de machine uitvalt), wordt de machine verwijderd. De duurzame status — transcript, werkruimtecommits, plaatsingsrecords — blijft bij de Gateway.

<Note>
Cloudworkers zijn opt-in en onzichtbaar totdat je een profiel configureert. Niet-geconfigureerde installaties zien geen nieuwe RPC's, configuratie of UI.
</Note>

## Wat waar wordt uitgevoerd

| Onderdeel                                               | Locatie                                                                          |
| ------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Agentlus + tools (`exec`, `read`, `write`, `edit`, …) | Cloudworkermachine                                                               |
| Modelinferentie en providerreferenties                  | Gateway (geproxyd via `{provider, model}`-referentie)                            |
| Transcript (duurzaam, sessieopslag)                     | Gateway                                                                          |
| Live streamen naar de zijbalk                           | Gateway-fan-out, gevoed door de herhaalbare gebeurtenisstroom van de worker      |
| Git-geschiedenis van de werkruimte                      | Zonder referenties aangemaakt op de machine; de Gateway neemt commits over en beheert push/PR |

De machine heeft behalve `sshd` geen inkomende poorten nodig: de Gateway maakt via vastgezette SSH een uitgaande verbinding en een reverse tunnel brengt de WebSocket van de worker terug. De gebundelde Crabbox-provider dwingt de openbare SSH-route af en schakelt beheerde Tailscale-inschrijving uit. Uitgaande internettoegang valt onder het providerbeleid; het standaard AWS-profiel heeft internettoegang, tenzij je het netwerk of de beveiligingsgroep ervan beperkt.

## Vereisten

- Een provider-Plugin voor workers. De gebundelde `crabbox`-Plugin stuurt de [Crabbox](https://github.com/openclaw/crabbox)-CLI aan, die leases tussen cloudbackends (AWS, Hetzner en andere) bemiddelt. Het binaire bestand `crabbox` moet zich op `PATH` bevinden (of stel `settings.binary` in), met reeds geconfigureerde providerreferenties. Voor toelating tot AWS is Crabbox 0.38.1 of nieuwer vereist.
- Voor Crabbox AWS-workers moet de effectieve `aws.instanceProfile` leeg zijn. De provider controleert `crabbox config show --json` vóór toewijzing en vereist vervolgens dat `crabbox inspect --json` `providerMetadata.instanceProfileAttached: false` rapporteert vanuit EC2 `DescribeInstances`. Leases met een instancerol of zonder gezaghebbende metadata worden gestopt en geweigerd.
- Node.js op de geleasete machine. Kale cloudimages bevatten dit doorgaans niet — installeer het met de opdracht `setup` van het profiel.
- Een sessie met een door de sessie beheerde worktree (maak er een met `worktree: true`). Verzending verplaatst de inhoud van die worktree; gewone mappen worden als manifestspiegel gesynchroniseerd.

## Configuratie

Voeg in `openclaw.json` een profiel toe onder `cloudWorkers.profiles`:

```json
{
  "cloudWorkers": {
    "profiles": {
      "aws": {
        "provider": "crabbox",
        "install": "bundle",
        "settings": {
          "provider": "aws",
          "class": "standard",
          "ttl": "8h",
          "idleTimeout": "45m",
          "setup": "test -x /usr/bin/node || (curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash - && sudo apt-get install -y nodejs)"
        }
      }
    }
  }
}
```

Profielvelden:

| Sleutel    | Betekenis                                                                                                                                                                                                                                      |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `provider` | ID van de workerprovider die door een Plugin is geregistreerd (`crabbox` voor de gebundelde Plugin).                                                                                                                                          |
| `install`  | `bundle` (standaard) levert de build van de actieve Gateway; `npm` installeert de exacte uitgebrachte Gateway-versie met vastgezette integriteit. `npm` vereist dat de Gateway vanuit een verpakte release wordt uitgevoerd. |
| `settings` | JSON in beheer van de provider. Voor crabbox: `provider` (backend), `class` (machineklasse), `ttl`, `idleTimeout` (Go-tijdsduurwaarden), optioneel `setup` en het absolute pad `binary`. OpenClaw dwingt openbare SSH af en schakelt beheerde Tailscale voor deze leases uit. |
| `lifetime` | Optioneel opgeslagen beleid (`idleTimeoutMinutes`, `maxLifetimeMinutes`).                                                                                                                                                                           |

### De setup-opdracht

`settings.setup` wordt uitgevoerd op de geleasete machine nadat deze gereed is voor SSH en voordat OpenClaw wordt geïnstalleerd. Deze opdracht wordt bij **elke** inrichtingspoging uitgevoerd (ook bij herhalingen na een onderbroken verzending) en moet daarom idempotent zijn — beveilig installaties met een `command -v`- of `test -x`-controle, zoals in het voorbeeld. Als de setup mislukt, stopt de provider de lease en wordt de verzending veilig afgebroken; er blijft geen half-geconfigureerde machine actief.

### Installatiekanalen

- **`bundle`** verpakt de `dist` van de actieve Gateway, een opgeschoonde `package.json` en alle werkruimtepakketten waarnaar de build verwijst, allemaal gedekt door een inhoudshash. De machine verifieert de oorspronkelijke bundel aan de hand van die hash en installeert vervolgens npm-productieafhankelijkheden (scripts uitgeschakeld). Zo voer je een ontwikkelbuild uit op een worker.
- **`npm`** verifieert dat de release in het openbare register bestaat, zet de SHA-512-integriteit ervan vast en installeert `openclaw@<version>` dat exact met de Gateway overeenkomt.

## Een sessie verzenden

Open in de Control UI **Nieuwe sessie**, kies een agent waarvan de geconfigureerde runtime OpenClaw is, selecteer in het menu **Waar** een geconfigureerd doel **Cloud · profiel** en start de taak. Als je cloud selecteert, wordt de vereiste beheerde worktree automatisch ingeschakeld; de Gateway maakt de sessie, voltooit de verzending en verzendt pas daarna de eerste beurt. De serverbadge in de sessiezijbalk toont de duurzame plaatsingsstatus. Clouddoelen worden niet aangeboden voor sessiecatalogi van externe CLI's.

De equivalente RPC-stroom is:

Maak een sessie met een beheerde worktree en verzend deze vervolgens (de RPC vereist `operator.admin` en bestaat alleen wanneer profielen zijn geconfigureerd):

Cloudworkers voeren de OpenClaw-agentruntime uit. Kies een `openai/*` of ander model dat naar die runtime wordt herleid; sessies die zijn geconfigureerd voor een externe CLI-runtime, zoals `claude-cli`, kunnen niet worden verzonden.

```bash
openclaw gateway call sessions.create \
  --params '{"key":"agent:main:big-refactor","worktree":true,"cwd":"/path/to/repo","worktreeName":"big-refactor"}'

openclaw gateway call sessions.dispatch \
  --timeout 1500000 \
  --params '{"key":"agent:main:big-refactor","profileId":"aws"}'
```

`sessions.dispatch` sluit lokale toelating van beurten, laat actief werk uitlopen, richt de lease in, voert de setup uit, initialiseert OpenClaw, synchroniseert de werkruimte en keert terug zodra de plaatsing `active`-workereigendom bereikt. Reken voor de eerste verzending op enkele minuten; leases en installaties worden gecachet waar de provider dit ondersteunt. Daarna communiceer je zoals gewoonlijk met de sessie — beurten worden automatisch naar de worker gerouteerd.

Voltooide workerbeurten brengen in aanmerking komende werkruimtebestanden binnen de maximale grootte terug naar de beheerde worktree van de sessie voordat de beurtclaim wordt vrijgegeven. De terminale workergebeurtenis maakt een duurzame pending-result-afscherming voordat deze wordt bevestigd. De Gateway staged vervolgens het volledige cloudresultaat als Git-ref onder `refs/openclaw/worker-results/` voordat het wordt toegepast, zodat de cloudversie herstelbaar blijft, zelfs als de Gateway tijdens het toepassen stopt. Werkruimteresultaten gebruiken de bestandssemantiek van Git: gewone bestanden, uitvoerbare bits, symbolische koppelingen, toevoegingen, wijzigingen en verwijderingen blijven behouden, terwijl lege mappen en andere mapmodi dat niet blijven. De resulterende bestandswijzigingen blijven in de beheerde worktree staan voor normale beoordeling en commit.

Bij het toepassen wordt het manifest van het verzendingstijdstip als samenvoegbasis gebruikt. Wijzigingen die alleen in de cloud bestaan, worden toegepast, wijzigingen die alleen lokaal bestaan, blijven behouden en voor paden die aan beide zijden zijn gewijzigd geldt een driewegbeleid dat de lokale versie behoudt. Een beurt met conflicten wordt toch voltooid: het transcript rapporteert het begrensde padoverzicht en de gestagede resultaatref, de plaatsing stelt hetzelfde conflict beschikbaar aan de Control UI en niet-conflicterende cloudwijzigingen blijven toegepast. De melding bevat `git show <ref>:<path>` om een aanwezig cloudbestand te inspecteren en een opdracht `git checkout <ref> -- <path>` met een letterlijk pathspec op het hoogste niveau om het vanuit elke werkruimtemap over te nemen. Voer de opdrachten uit in Bash of zsh (Git Bash op Windows). Als de inspectie meldt dat het pad niet bestaat, heeft het cloudresultaat het verwijderd; controleer dit en verwijder het behouden lokale pad handmatig. Als checkout een blokkade door een bestand/map meldt, verplaats of verwijder dan het blokkerende lokale pad en probeer het opnieuw. Als de gestagede ref zelf verdwenen is, behandel de melding dan als verouderd en wijzig het lokale pad niet. Gestagede refs met conflicten blijven beschikbaar nadat de normale beurt-afscherming is vrijgegeven; een later schoon resultaat wist de melding en trekt de oude ref in, terwijl expliciete verwijdering van de afscherming de laatste opschoningsgrens vormt.

Terwijl een afgeschermd resultaat nog wordt verwerkt, wacht een nieuwe beurt maximaal 15 seconden totdat de vorige claim wordt vrijgegeven. Als deze nog steeds bezig is, mislukt de beurt met een bruikbare melding “het werkruimteresultaat van de vorige cloudbeurt wordt nog verwerkt” en kan deze kort daarna opnieuw worden geprobeerd. Na een herstart ontdekt herstel openstaande en gestagede resultaten vóór het opschonen van verouderde claims, voltooit of herhaalt het lokaal toepassen daarvan en eist dode omgevingen pas terug nadat het resultaat is behouden. Het begrensde SQLite-rollbackjournaal maakt een onderbroken toepassing op het bestandssysteem herstelbaar zonder reeds geaccepteerde mutaties opnieuw af te spelen.

Wanneer het werk is voltooid en er geen beurt wordt uitgevoerd, open je het sessiemenu en kies je **Cloudworker stoppen…**. De Gateway voert nog één laatste synchronisatie van de werkruimte uit voordat de omgeving wordt vernietigd. Een plaatsing die zich al in `draining` of `reconciling` bevindt, rondt de afbouw af; wacht tot de badge `reclaimed` wordt voordat je de sessie verwijdert.

Voor een defecte of op hol geslagen gekoppelde worker kan een operator als laatste redmiddel `environments.destroy` aanroepen met `{ "force": true }`. Geforceerde afbouw markeert de plaatsing duurzaam als mislukt en laat elk niet-verwerkt extern resultaat achter voordat de omgeving wordt vernietigd.

De equivalente administratieve RPC is:

```bash
openclaw gateway call sessions.reclaim \
  --timeout 600000 \
  --params '{"key":"agent:main:big-refactor"}'
```

Plaatsing verloopt via een duurzame toestandsmachine (`local → requested → provisioning → syncing → starting → active`), zodat bij een herstart van de Gateway tijdens de dispatch machines worden gereconcilieerd in plaats van dat ze achterblijven. Bij een mislukte modelbeurt blijft de actieve plaatsing beschikbaar voor een nieuwe poging. Bij conflicten met werkruimtepaden blijft de lokale versie behouden, wordt de rest van het cloudresultaat toegepast en blijft de klaargezette cloud-ref behouden voor inspectie; bij andere reconciliatie- of levenscyclusfouten blijven de duurzame herstelbarrière en het diagnostische uiteinde behouden totdat het herstel veilig opnieuw kan worden geprobeerd of de omgeving kan worden teruggevorderd.

## Beveiligingsmodel

- **Gesloten inkomend verkeer voor workers.** Workers communiceren via een speciaal protocol op de getunnelde socket met een gesloten lijst van toegestane methoden — een worker kan geen RPC's voor operators aanroepen.
- **Door de Gateway beheerde toolbevoegdheid.** Vóór elke beurt projecteert de Gateway het actuele beleid voor profiel, provider, agent, groep, afzender, sandbox, delegatie, overerving en runtimebeperkingen op de vaste catalogus met codeertools van de worker. De startenvelop bevat uitsluitend die definitieve subset met een gesloten vocabulaire. Expliciet begrensde geplande beurten hergebruiken hun vertrouwde context van de eigenaarsgroep zonder die identiteit naar de box te sturen of opnieuw een nieuwe afzenderoverlay toe te passen. Tools buiten de catalogus van de worker blijven niet beschikbaar; met een leeg resultaat wordt zonder tools gewerkt.
- **Uitgegeven inloggegevens, gehasht opgeslagen.** Elke dispatch geeft inloggegevens voor een worker uit; de Gateway slaat alleen de hash ervan op. Rotatie van inloggegevens en afscherming op basis van het eigenaarstijdperk garanderen maximaal één actieve eigenaar per sessie — een verouderde worker die opnieuw verbinding maakt, wordt afgeschermd en nooit samengevoegd.
- **Vastzetten van hostsleutels.** De provider moet tijdens het beschikbaar stellen de SSH-hostsleutel van de box beschikbaar maken; bootstrap maakt verbinding met strikte sleutelvastzetting en stopt veilig als de sleutel ontbreekt.
- **Geen permanent aanwezige model-, forge- of cloudinloggegevens op de box.** Modelauthenticatie blijft op de Gateway (inferentie verloopt via de `{provider, model}`-referentie), Git-commits in de werkruimte worden zonder forge-inloggegevens aangemaakt en de metadata van Crabbox AWS-leases wordt vóór de configuratie gezaghebbend gecontroleerd op een instantierol. Houd ook configuratiecommando's vrij van inloggegevens.
- **Door de provider beheerd uitgaand verkeer.** Door de omgekeerde tunnel heeft OpenClaw geen directe modeltoegang nodig, maar OpenClaw herschrijft geen providerfirewalls. Beperk uitgaand verkeer bij de workerprovider wanneer de taak dit vereist.
- **Duurzame transcripten die exact één keer worden vastgelegd.** De worker legt transcriptbatches vast via een compare-and-swap-protocol tegen het blad van de sessie; bij een verouderde basis wordt de uitvoering onmiddellijk gestopt in plaats van betaalde uitvoer te dupliceren of te rebasen.

## Probleemoplossing

- **`sessions.dispatch` is een onbekende methode** — er zijn geen `cloudWorkers.profiles` geconfigureerd of de aanroeper beschikt niet over `operator.admin`.
- **"Cloudworkerbeurten vereisen de OpenClaw-runtime"** — kies een model waarvan OpenClaw de geconfigureerde runtime is. Externe CLI-runtimes zoals `claude-cli` ondersteunen geen workerinferentie.
- **"Voor de bootstrap van een worker is Node.js vereist op de geleasete host"** — voeg een Node-installatie toe aan `settings.setup` (zie hierboven).
- **Attestatie van de AWS-instantierol mislukt** — wis `aws.instanceProfile` (en `CRABBOX_AWS_INSTANCE_PROFILE`, indien ingesteld). Installeer Crabbox 0.38.1 of nieuwer; oudere binaire bestanden bieden niet het gezaghebbende `providerMetadata.instanceProfileAttached`-contract dat voor AWS-toelating vereist is.
- **Dispatch mislukt met een providerfout** — de plaatsingsrecord en `environments.list` bewaren de laatste fout, inclusief het laatste deel van stderr van de configuratie/bootstrap. Boxen worden bij een fout vernietigd, dus dat laatste deel is de primaire bron voor forensisch onderzoek.
- **Clienttime-out tijdens de dispatch** — `openclaw gateway call` gebruikt standaard een time-out van 10s; geef `--timeout` ruim mee (de dispatch blijft in beide gevallen aan de serverzijde actief en een nieuwe poging tijdens het beschikbaar stellen wordt geweigerd met `session cannot dispatch from placement provisioning`).
- **Worker teruggevorderd na een upgrade vanaf een bèta van 2026.7.2** — die bèta's gebruikten het oudere startcontract voor workers. Bij een herstart vernietigt OpenClaw een inactieve incompatibele worker, behoudt het de sessie en werkruimte, markeert het de plaatsing als teruggevorderd en stelt het bij de volgende dispatch of beurt een actuele worker beschikbaar. Een bèta-worker die wordt onderbroken terwijl deze nog wordt gestart, wordt na het opschonen als mislukt gemarkeerd; probeer de dispatch opnieuw om de worker met het actuele contract beschikbaar te stellen.
- **Melding van een conflict in de cloudwerkruimte** — de beurt is voltooid en de lokale versie van elk vermeld pad is behouden. Gebruik de commando's voor de klaargezette ref in de melding om de cloudversie te inspecteren of over te nemen; voor de niet-conflicterende wijzigingen, die al zijn toegepast, is geen nieuwe poging nodig.
- **“Het werkruimteresultaat van de vorige cloudbeurt wordt nog gereconcilieerd”** — de Gateway heeft kort gewacht op de duurzame barrière van het vorige resultaat en kon de sessieclaim niet verkrijgen. Wacht totdat de reconciliatie is voltooid en probeer de beurt vervolgens opnieuw; het herstarten van de Gateway is veilig, omdat het herstel klaargezette resultaten behoudt voordat een niet-werkende worker wordt teruggevorderd.
- **Leasebeheer** — `crabbox list --provider <backend>` toont actieve leases; `crabbox stop --provider <backend> --id <lease>` geeft er handmatig één vrij. Inactieve leases verlopen op basis van `idleTimeout` van het profiel.

## Gerelateerd

- [Sandboxing](/nl/gateway/sandboxing) — de impact van lokale tooluitvoering beperken
- [Sessies-CLI](/nl/cli/sessions) — opgeslagen sessies inspecteren
- [Configuratiereferentie](/nl/gateway/configuration-reference)
