---
read_when:
    - De levenscyclus van ACP-sessies of het opruimen van ACPX-processen refactoren
    - Foutopsporing voor verweesde ACPX-processen, hergebruik van PID's of veilige opschoning met meerdere Gateways
    - De zichtbaarheid van sessions_list wijzigen voor gestarte ACP- of subagentsessies
    - Eigenaarschapsmetadata ontwerpen voor achtergrondtaken, ACP-sessies of procesleases
sidebarTitle: ACP lifecycle refactor
summary: Migratieplan om het eigenaarschap van ACP-sessies en ACPX-processen expliciet te maken
title: Refactor van de ACP-levenscyclus
x-i18n:
    generated_at: "2026-07-27T06:32:13Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bda66f0acc93216c3d9386ca3ebf7f544efd306cd7f53386391f0c48e5dc8f06
    source_path: refactor/acp.md
    workflow: 16
---

De ACP-levenscyclus werkt momenteel, maar te veel ervan wordt achteraf afgeleid.
Procesopruiming reconstrueert eigenaarschap uit PID's, opdrachtreeksen, wrapper-
paden en de live procestabel. Sessiezichtbaarheid reconstrueert eigenaarschap
uit sessiesleutelreeksen plus secundaire `sessions.list({ spawnedBy })`-zoekopdrachten.
Daardoor zijn gerichte oplossingen mogelijk, maar kunnen randgevallen ook gemakkelijk worden gemist:
hergebruik van PID's, opdrachten tussen aanhalingstekens, kleinkindprocessen van adapters, statusroots met meerdere Gateways,
`cancel` versus `close`, en zichtbaarheid van `tree` versus `all` worden allemaal afzonderlijke
plaatsen waar dezelfde eigenaarschapsregels opnieuw moeten worden achterhaald.

Deze refactor maakt eigenaarschap een eersteklasconcept. Het doel is geen nieuw ACP-product-
oppervlak, maar een veiliger intern contract voor het bestaande ACP- en ACPX-gedrag.

## Doelen

- Opruiming stuurt nooit een signaal naar een proces tenzij actueel live bewijs overeenkomt met een
  lease die eigendom is van OpenClaw.
- `cancel`, `close` en opruiming bij het opstarten hebben verschillende intenties voor de levenscyclus.
- `sessions_list`, `sessions_history`, `sessions_send` en statuscontroles gebruiken
  hetzelfde model voor sessies die eigendom zijn van de aanvrager.
- Installaties met meerdere Gateways kunnen elkaars ACPX-wrappers niet opruimen.
- Oude ACPX-sessierecords blijven tijdens de migratie werken.
- De runtime blijft eigendom van de plugin; core krijgt geen kennis van ACPX-pakketdetails.

## Geen doelen

- ACPX vervangen of het openbare opdrachtoppervlak van `/acp` wijzigen.
- Leveranciersspecifiek gedrag van ACP-adapters naar core verplaatsen.
- Van gebruikers vereisen dat ze vóór een upgrade handmatig de status opruimen.
- Herbruikbare ACP-sessies laten sluiten door `cancel`.

## Doelmodel

### Identiteit van Gateway-instantie

Elk Gateway-proces moet een stabiele runtime-instantie-id hebben:

```ts
type GatewayInstanceId = string;
```

Deze kan bij het opstarten van de Gateway worden gegenereerd en gedurende de levensduur van
die installatie in de status worden opgeslagen. Het is geen beveiligingsgeheim, maar een onderscheidingskenmerk voor eigenaarschap dat wordt gebruikt
om te voorkomen dat de ACP-processen van de ene Gateway worden verward met de processen van een andere Gateway.

### Eigenaarschap van ACP-sessies

Elke gestarte ACP-sessie moet genormaliseerde eigenaarschapsmetadata hebben:

```ts
type AcpSessionOwner = {
  sessionKey: string;
  spawnedBy?: string;
  parentSessionKey?: string;
  ownerSessionKey: string;
  agentId: string;
  backend: "acpx";
  gatewayInstanceId: GatewayInstanceId;
  createdAt: number;
};
```

De Gateway moet deze velden retourneren voor sessierijen waar ze bekend zijn.
Zichtbaarheidsfiltering moet een zuivere controle op rijmetadata zijn:

```ts
canSeeSessionRow({
  row,
  requesterSessionKey,
  visibility,
  a2aPolicy,
});
```

Dit verwijdert verborgen secundaire `sessions.list({ spawnedBy })`-aanroepen uit
zichtbaarheidscontroles. Een gestart ACP-kindproces van een andere agent is eigendom van de aanvrager omdat
dit in de rij staat, niet omdat een tweede query het toevallig vindt.

### ACPX-procesleases

Elke gegenereerde wrapperstart moet een leaserecord aanmaken:

```ts
type AcpxProcessLease = {
  leaseId: string;
  gatewayInstanceId: GatewayInstanceId;
  sessionKey: string;
  wrapperRoot: string;
  wrapperPath: string;
  rootPid: number;
  processGroupId?: number;
  commandHash: string;
  startedAt: number;
  state: "open" | "closing" | "closed" | "lost";
};
```

Het wrapperproces ontvangt de lease-id en Gateway-instantie-id als overdraagbare
argumenten:

```sh
--openclaw-acpx-lease-id ... --openclaw-gateway-instance-id ...
```

Wanneer het platform dit toestaat, moet verificatie de voorkeur geven aan live procesmetadata
die niet door aanhalingstekens in opdrachten kan worden verward:

- root-PID bestaat nog
- live wrapperpad bevindt zich onder `wrapperRoot`
- procesgroep komt overeen met de lease wanneer deze beschikbaar is
- argumenten bevatten de verwachte lease-id
- opdrachthash of pad naar uitvoerbaar bestand komt overeen met de lease

Als het live proces niet kan worden geverifieerd, wordt opruiming veiligheidshalve afgebroken.

## Levenscycluscontroller

Introduceer één ACPX-levenscycluscontroller die eigenaar is van procesleases en het opruimings-
beleid:

```ts
interface AcpxLifecycleController {
  ensureSession(input: AcpRuntimeEnsureInput): Promise<AcpRuntimeHandle>;
  cancelTurn(handle: AcpRuntimeHandle): Promise<void>;
  closeSession(input: {
    handle: AcpRuntimeHandle;
    discardPersistentState?: boolean;
    reason?: string;
  }): Promise<void>;
  reapStartupOrphans(): Promise<void>;
  verifyOwnedTree(lease: AcpxProcessLease): Promise<OwnedProcessTree | null>;
}
```

`cancelTurn` vraagt alleen om annulering van de beurt. Het mag herbruikbare wrapper-
of adapterprocessen niet opruimen.

`closeSession` mag opruimen, maar alleen nadat het sessierecord is geladen,
de lease is geladen en is geverifieerd dat de live processtructuur nog bij die
lease hoort.

`reapStartupOrphans` begint bij open leases in de status. Het mag de proces-
tabel gebruiken om afstammelingen te vinden, maar het moet niet eerst willekeurige op ACP lijkende
opdrachten scannen en vervolgens besluiten dat die waarschijnlijk van ons zijn.

## Wrappercontract

Gegenereerde wrappers moeten klein blijven. Ze moeten:

- de adapter starten in een procesgroep waar dit wordt ondersteund
- normale beëindigingssignalen doorsturen naar de procesgroep
- overlijden van het bovenliggende proces detecteren
- bij overlijden van het bovenliggende proces SIGTERM sturen en vervolgens de wrapper actief houden totdat de SIGKILL-
  terugval wordt uitgevoerd
- root-PID en procesgroep-id terugrapporteren aan de levenscycluscontroller wanneer
  die beschikbaar zijn

Wrappers mogen niet over sessiebeleid beslissen. Ze dwingen alleen lokale opruiming van de processtructuur
voor hun eigen adaptergroep af.

## Contract voor sessiezichtbaarheid

Zichtbaarheid moet genormaliseerd rijeigenaarschap gebruiken:

```ts
type SessionVisibilityInput = {
  requesterSessionKey: string;
  row: {
    key: string;
    agentId: string;
    ownerSessionKey?: string;
    spawnedBy?: string;
    parentSessionKey?: string;
  };
  visibility: "self" | "tree" | "agent" | "all";
  a2aPolicy: AgentToAgentPolicy;
};
```

Regels:

- `self`: alleen de sessie van de aanvrager.
- `tree`: de sessie van de aanvrager plus rijen die eigendom zijn van of gestart zijn vanuit de aanvrager.
- `all`: alle rijen van dezelfde agent, door a2a toegestane rijen van andere agents en door de aanvrager beheerde
  gestarte rijen van andere agents, zelfs wanneer algemene a2a is uitgeschakeld.
- `agent`: alleen dezelfde agent, tenzij een expliciete eigenaarsrelatie aangeeft dat de rij
  bij de aanvrager hoort.

Dit maakt `tree` en `all` monotoon: `all` mag geen eigen kindproces verbergen dat
`tree` wel zou tonen.

## Migratieplan

### Fase 1: identiteit en leases toevoegen

- Voeg `gatewayInstanceId` toe aan de Gateway-status.
- Voeg een ACPX-leasestore toe onder de ACPX-statusmap.
- Schrijf een lease voordat een gegenereerde wrapper wordt gestart.
- Sla `leaseId` op in nieuwe ACPX-sessierecords.
- Behoud bestaande PID- en opdrachtvelden voor oude records.

### Fase 2: opruiming met leases als uitgangspunt

- Wijzig opruiming bij sluiten zodat eerst `leaseId` wordt geladen.
- Verifieer eigenaarschap van het live proces aan de hand van de lease voordat een signaal wordt gestuurd.
- Behoud de huidige terugval op root-PID en wrapperroot alleen voor verouderde records.
- Markeer leases als `closed` na geverifieerde opruiming.
- Markeer leases als `lost` wanneer het proces vóór de opruiming verdwenen is.

### Fase 3: opruiming bij opstarten met leases als uitgangspunt

- Opruiming bij het opstarten scant open leases.
- Verifieer voor elke lease het rootproces en verzamel afstammelingen.
- Ruim geverifieerde structuren op, te beginnen bij de kindprocessen.
- Laat oude `closed`- en `lost`-leases verlopen met een begrensde bewaartermijn.
- Behoud het scannen van opdrachtmarkeringen alleen als tijdelijke terugval voor verouderde records, waar mogelijk beschermd door
  wrapperroot en Gateway-instantie.

### Fase 4: rijen voor sessie-eigenaarschap

- Voeg eigenaarschapsmetadata toe aan Gateway-sessierijen.
- Leer schrijvers van ACPX, subagents, achtergrondtaken en sessiestores om
  `ownerSessionKey` of `spawnedBy` in te vullen.
- Zet controles voor sessiezichtbaarheid om zodat ze rijmetadata gebruiken.
- Verwijder secundaire `sessions.list({ spawnedBy })`-zoekopdrachten tijdens zichtbaarheidscontroles.

### Fase 5: verouderde heuristieken verwijderen

Na één releaseperiode:

- stop met vertrouwen op opgeslagen rootopdrachtreeksen voor niet-verouderde ACPX-opruiming
- verwijder scans van opdrachtmarkeringen bij het opstarten
- verwijder terugvalzoekopdrachten in lijsten voor zichtbaarheid
- behoud defensief, veiligheidshalve afbrekend gedrag voor ontbrekende of niet-verifieerbare leases

## Tests

Voeg twee tabelgestuurde suites toe.

Simulator voor proceslevenscyclus:

- PID hergebruikt door een niet-gerelateerd proces
- PID hergebruikt door de wrapperroot van een andere Gateway
- opgeslagen wrapperopdracht is door de shell tussen aanhalingstekens geplaatst, live `ps`-opdracht niet
- adapterkindproces stopt, kleinkindproces blijft in de procesgroep
- SIGTERM-terugval bij overlijden van bovenliggend proces bereikt SIGKILL
- proceslijst niet beschikbaar
- verouderde lease met ontbrekend proces
- weesproces bij opstarten met wrapper, adapterkindproces en kleinkindproces

Matrix voor sessiezichtbaarheid:

- `self`, `tree`, `agent`, `all`
- a2a ingeschakeld en uitgeschakeld
- rij van dezelfde agent
- rij van een andere agent
- door de aanvrager beheerde gestarte ACP-rij van een andere agent
- aanvrager in sandbox beperkt tot `tree`
- acties voor lijst, geschiedenis, verzenden en status

De belangrijke invariant: een door de aanvrager beheerd gestart kindproces is overal zichtbaar waar
de geconfigureerde zichtbaarheid de sessiestructuur van de aanvrager omvat, en `all` is niet
minder krachtig dan `tree`.

## Compatibiliteitsopmerkingen

Oude sessierecords hebben mogelijk geen `leaseId`. Ze moeten het verouderde
veiligheidshalve afbrekende opruimingspad gebruiken:

- vereis een live rootproces
- vereis eigenaarschap van de wrapperroot wanneer een gegenereerde wrapper wordt verwacht
- vereis overeenstemming van opdrachten voor roots zonder wrapper
- stuur nooit een signaal uitsluitend op basis van verouderde opgeslagen PID-metadata

Als een verouderd record niet kan worden geverifieerd, laat het dan met rust. Opruiming van leases bij het opstarten en
de volgende releaseperiode moeten de terugval uiteindelijk uitfaseren.

## Succescriteria

- Het sluiten van een oude of verouderde ACPX-sessie kan het proces van een andere Gateway niet beëindigen.
- Overlijden van het bovenliggende proces laat geen hardnekkige kleinkindprocessen van adapters actief.
- `cancel` breekt de actieve beurt af zonder herbruikbare sessies te sluiten.
- `sessions_list` kan ACP-kindprocessen van andere agents die eigendom zijn van de aanvrager tonen onder zowel
  `tree` als `all`.
- Opruiming bij het opstarten wordt aangestuurd door leases, niet door brede scans van opdrachtreeksen.
- De gerichte tests voor de proces- en zichtbaarheidsmatrix dekken elk randgeval
  waarvoor voorheen eenmalige reviewoplossingen nodig waren.
