---
read_when:
    - De levenscyclus, opslag, het protocol of de autorisatie van exec- of Plugin-goedkeuringen wijzigen
    - Goedkeuringslinks of systeemeigen goedkeuringsbedieningselementen aan een kanaal toevoegen
    - Goedkeuringen van onderliggende sessies weergeven in bovenliggende of orchestratorweergaven
summary: Ontwerp voor duurzame, rechtstreeks linkbare goedkeuringen in de Control UI, native apps, kanalen en bovenliggende sessies
title: Goedkeuringen door operators via meerdere interfaces
x-i18n:
    generated_at: "2026-07-27T05:32:20Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 9defdaada1911df1184f64429e1787c4881e735c433d6dbc30a5946e11cc7cce
    source_path: refactor/operator-approvals.md
    workflow: 16
---

# Operatorgoedkeuringen voor meerdere oppervlakken

Dit ontwerp volgt [#103505](https://github.com/openclaw/openclaw/issues/103505). Het vervangt proceslokale goedkeuringsbevoegdheid door één door de Gateway beheerde, op SQLite gebaseerde levenscyclus. Elke door de Gateway beheerde goedkeuring voor exec of een plugin/tool krijgt één stabiele ID, één geauthenticeerde Control UI-route, atomische afhandeling waarbij het eerste antwoord wint, en alleen voor operators bestemde projecties naar de bron- en bovenliggende sessiestromen.

Inline-acties en deep links bestaan naast elkaar. Er is geen schakelaar voor de goedkeuringsmodus.

## Doelen

- Eén duurzaam goedkeuringsobject voor exec- en plugin/toolpoorten.
- Stabiele `${controlUiBasePath}/approve/{approvalId}`-route.
- Afhandeling vanuit elke geautoriseerde Control UI, native app of elk kanaaloppervlak.
- Atomisch gedrag waarbij het eerste antwoord wint op gelijktijdige oppervlakken.
- Idempotente identieke nieuwe pogingen; conflicterende late antwoorden kunnen de winnaar niet overschrijven.
- Time-out, ongeldige vertrouwde uitspraken, ontbrekende routes, annulering en herstart weigeren standaard toegang.
- Aanvraag- en terminale gebeurtenissen bereiken de bronsessie en alle relevante bovenliggende/orchestrator-eigenaren.
- Kanalen ontvangen getypeerde goedkeurings- en navigatieacties; callbackgegevens van het transport blijven privé voor het kanaal.
- Bestaande Gateway-methoden voor exec/plugins blijven compatibel terwijl hun implementatie samenkomt in één service.

## Niet-doelen

- De geblokkeerde tooluitvoering zelf permanent opslaan of hervatten na een herstart van de Gateway.
- Een goedkeurings-ID of URL tot bearer-referentie maken.
- Goedkeuringsprompts toevoegen aan voor het model zichtbare transcripten of bovenliggende agents activeren.
- Goedkeuringsbeleid, productopdrachten of autorisatie van beoordelaars naar kanaalplugins verplaatsen.
- De goedkeuringsstatus per kanaal, apparaat of bovenliggende instantie klonen.
- Exec-toelatingslijsten, samenstelling van pluginbeleid of permanente opslag van `allow-always` opnieuw ontwerpen, behalve waar dat nodig is om terminale uitkomsten ondubbelzinnig te maken.
- Een ingebedde TUI zonder Gateway in de eerste uitbreiding op afstand bereikbaar maken. Deze blijft uitsluitend lokaal en moet standaard toegang weigeren wanneer er geen beoordelaar bestaat.

## Basislijn vóór uitrol en bewijsoverzicht

Deze tabel legt de implementatiestatus vast van het moment waarop #103505 werd geopend. De onderstaande uitrolsecties volgen het duurzame register, getypeerde acties, de deep-linkpagina en uitbreidingen voor native clients die boven op die basislijn zijn gebouwd.

| Oppervlak         | Basisingangspunt en eigenaar                                                                                                                                   | Basisgedrag en tekortkoming                                                                                                                                                                  |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Agent-exec        | `src/agents/bash-tools.exec-approval-request.ts`, `src/agents/bash-tools.exec-host-shared.ts`                                                                   | Tweefasige registratie van `exec.approval.*` voorkomt een vroege race met `/approve`, maar een time-out kan via `askFallback` nog steeds in toestaan veranderen.                             |
| Plugintoolpoort   | `src/agents/agent-tools.before-tool-call.ts`                                                                                                                    | Vraagt `plugin.approval.*` aan; `timeoutBehavior: "allow"` kan een verlopen poort goedkeuren. De ingebedde modus heeft afzonderlijke proceslokale bevoegdheid in `src/infra/embedded-plugin-approval-broker.ts`. |
| Plugin-Node-poort | `src/gateway/node-invoke-plugin-policy.ts`                                                                                                                      | Maakt rechtstreeks via de pluginmanager aan en zendt uit, waardoor een deel van de levenscyclus van servermethoden wordt gedupliceerd.                                                       |
| Gateway-bevoegdheid | `src/gateway/server-aux-handlers.ts`, `src/gateway/exec-approval-manager.ts`, `src/gateway/server-methods/approval-shared.ts`                                   | Afzonderlijke exec- en pluginmanagers gebruiken proceslokale maps. Terminale vermeldingen blijven 15 seconden bestaan. Het eerste antwoord wint alleen binnen één proces.                   |
| Gateway-protocol  | `packages/gateway-protocol/src/schema/exec-approvals.ts`, `packages/gateway-protocol/src/schema/plugin-approvals.ts`, `src/gateway/methods/core-descriptors.ts` | Exec heeft alleen voor openstaande items bestemde `get`; plugin heeft geen `get`; er bestaat geen soortonafhankelijke terminale zoekopdracht voor een deep link.                     |
| Levering          | `src/infra/exec-approval-channel-runtime.ts`, `src/infra/approval-native-runtime.ts`, `src/infra/approval-handler-runtime.ts`                                   | Ondersteunt routering naar de oorsprong, privéberichten aan goedkeurders, herhaling van openstaande items, native handlers en terminale opschoning binnen het proces. Een afzonderlijke vervolgwijziging voegt duurzame terminale reconciliatie toe. |
| Draagbare acties  | `src/interactive/payload.ts`, `src/plugin-sdk/interactive-runtime.ts`, `src/plugin-sdk/approval-reply-runtime.ts`                                               | Goedkeuringsknoppen zijn opdrachtacties die `/approve ...` bevatten; URL- en Web App-doelen zijn niet-getypeerde knopvelden.                                                                  |
| Telegram          | `extensions/telegram/src/approval-handler.runtime.ts`, `extensions/telegram/src/button-types.ts`                                                                | De renderer parseert opdrachttekst om goedkeuringssemantiek te herkennen voordat privé-callbackgegevens worden geproduceerd.                                                                |
| Control UI        | `ui/src/app/exec-approval.ts`, `ui/src/app/overlays.ts`, `ui/src/components/exec-approval.ts`                                                                   | De goedkeurings-UI is een globale modal. `ui/src/app-route-paths.ts` en `ui/src/app-routes.ts` gebruiken exacte routes en herschrijven onbekende paden naar Chat.                                           |
| Sessie-eigenaarschap | `src/agents/subagent-registry.types.ts`, `src/agents/subagent-registry-read.ts`, `src/config/sessions/types.ts`                                                 | Eigenaarschap voor controller, aanvrager, expliciete bovenliggende instantie en verouderde spawn bestaat, maar goedkeuringsgebeurtenissen worden niet naar die sessiestromen geprojecteerd. |
| Gedeelde status   | `src/state/openclaw-state-schema.sql`, `src/state/openclaw-state-db.ts`                                                                                         | Bestaande directe transacties en voorwaardelijke Kysely-updates ondersteunen duurzame compare-and-set in `state/openclaw.sqlite`.                                                               |

Representatieve huidige tests zijn onder meer `src/gateway/exec-approval-manager.test.ts`, `src/gateway/server-methods/approval-shared.test.ts`, `src/agents/bash-tools.exec-gateway-approval.e2e.test.ts`, `extensions/telegram/src/approval-handler.runtime.test.ts` en `ui/src/e2e/approval-flow.e2e.test.ts`.

De plugin-SDK blijft de enige grens voor kanalen/plugins. Wijzigingen in de goedkeuringsruntime en -presentatie moeten via de bestaande subpaden `src/plugin-sdk/approval-*.ts` en `src/plugin-sdk/interactive-runtime.ts` worden geëxporteerd; productiecode van plugins mag geen interne Gateway-onderdelen importeren.

## Eerdere oplossingen

Omnigent biedt nuttige UX- en foutsemantiek:

- [`approval.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/runtime/policies/approval.py) parkeert ASK, past time-outs per beleid toe en behandelt alleen een exacte acceptatie als goedkeuring.
- [`sessions.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/server/routes/sessions.py) bevat de serverpoort voor het native harnas en de projectie van aanvragen/afhandelingen naar bovenliggende instanties.
- [`ApprovePage.tsx`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/web/src/pages/ApprovePage.tsx) biedt de zelfstandige mobiele goedkeuringspagina.

Neem de claim over opslag niet kritiekloos over. De huidige actieve openstaande status is proceslokaal in [`_elicitation_registry.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/server/_elicitation_registry.py) en de ongebruikte tabel voor openstaande items wordt verwijderd door [`e3b1f2a4c9d7_drop_pending_tool_calls_table.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/db/migrations/versions/e3b1f2a4c9d7_drop_pending_tool_calls_table.py). OpenClaw gaat bewust verder: SQLite is gezaghebbend en elke terminale overgang is een compare-and-set in de database.

## Architectuur en eigenaarschap

De Gateway beheert de levenscyclus:

1. Een agent, pluginhook of Node-beleid levert een soortspecifieke aanvraag en proceslokale uitvoeringsbinding.
2. De Gateway valideert deze en bouwt een opgeschoonde projectie voor beoordelaars.
3. De goedkeuringsservice berekent een publiek van bron/eigenaren, voegt de canonieke rij in en registreert vervolgens de waiter binnen het proces.
4. Na duurzame invoeging publiceert de Gateway bestaande goedkeuringsgebeurtenissen, sessieprojecties, kanaalmeldingen en native pushmeldingen.
5. Elk oppervlak handelt af via dezelfde service.
6. De service commit één terminale overgang, activeert de runtime-waiter en publiceert terminale projecties.
7. Een mislukte levering van een gebeurtenis draait de gecommitte beslissing nooit terug; clients herstellen via `approval.get` of herhaling van de lijst.

Eigenaarsgrenzen:

- `src/gateway/`: goedkeuringsservice, autorisatie, RPC-adapters, URL-constructie, waiterlevenscyclus en publicatie van gebeurtenissen.
- `src/state/`: gedeeld schema en gegenereerde Kysely-typen.
- `src/infra/`: opgeschoonde goedkeuringsweergavemodellen en constructie van draagbare presentaties.
- `src/agents/`: vraagt het geretourneerde oordeel aan, wacht erop en past het toe; geen permanente opslag.
- `src/channels/` en `extensions/*`: renderen getypeerde acties, autoriseren kanaalgebruikers, coderen privé-callbacks en werken geleverde bedieningselementen bij.
- `src/plugin-sdk/`: alleen openbare goedkeurings- en presentatiecontracten.
- `ui/`: zelfstandige pagina en bestaande clients voor wachtrijen/modals.

De waiter binnen het proces is een meldingsmechanisme, geen bevoegdheid. Registratie voegt de rij in en installeert de waiter synchroon voordat de aanvraag wordt gepubliceerd, zodat een resolver niet tussen deze stappen kan komen. Elke latere resolver commit via SQLite voordat die waiter wordt afgehandeld.

## Permanente record

Voeg één tabel `operator_approvals` toe aan de gedeelde statusdatabase.

| Kolom                                             | Doel                                                                                                                                       |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `approval_id`                                      | Wereldwijd unieke canonieke ID. Behoud bestaande exec-ID's en `plugin:`-ID's voor protocolcompatibiliteit, maar leid het type nooit af uit het voorvoegsel.      |
| `resolution_ref`                                   | Unieke volledige SHA-256-base64url-locator voor transportcallbacks die de canonieke ID niet kunnen bevatten. Dit is geen autorisatie of ID van een openbare URL. |
| `kind`                                             | Gesloten `exec \| plugin`-discriminator.                                                                                                        |
| `status`                                           | Gesloten `pending \| allowed \| denied \| expired \| cancelled`-status.                                                                          |
| `presentation_json`                                | Gevalideerde, van een typetag voorziene reviewerprojectie. Onbewerkte runtimeverzoeken, opdrachtbindingen en callbackpayloads blijven proceslokaal.               |
| `source_agent_id`, `source_session_key`            | Bronidentiteit en anker voor de sessieprojectie. De sessiesleutel is duurzaam; de roterende sessie-UUID niet.                                          |
| `audience_session_keys_json`                       | Geordende, gededupliceerde JSON-array die wordt geproduceerd door de begrensde breedte-eerst-doorloop van eigenaarschap. Aangevraagde en terminale gebeurtenissen gebruiken dezelfde momentopname. |
| `requested_by_device_id`, `requested_by_client_id` | Duurzame metadata van de aanvrager en voor audits. De verbindings-ID blijft in het geheugen en is geen principal voor meerdere oppervlakken.                                         |
| `reviewer_device_ids_json`                         | Optionele, expliciet geselecteerde reviewerapparaten die uitsluitend door de vertrouwde goedkeuringsruntime worden opgegeven.                                                  |
| `runtime_epoch`                                    | Procesepoch die eigenaar is van de geparkeerde uitvoering; wordt gebruikt om verweesde rijen na een herstart te annuleren.                                                     |
| `created_at_ms`, `expires_at_ms`, `updated_at_ms`  | Gezaghebbende timing.                                                                                                                         |
| `decision`                                         | Expliciete gebruikersbeslissing wanneer die bestaat.                                                                                                       |
| `terminal_reason`                                  | Gesloten reden, zoals `user`, `timeout`, `malformed-verdict`, `no-route`, `run-aborted` of `gateway-restart`.                                |
| `resolved_at_ms`, `resolver_kind`, `resolver_id`   | Winnaar en auditidentiteit die aan de serverzijde worden bewaard. Reviewerprojecties laten onbewerkte resolver-ID's weg.                                           |
| `consumed_at_ms`, `consumed_by`                    | Afzonderlijke replaybeveiliging voor `allow-once`; verbruik mag de vastgelegde beslissing niet wissen.                                                       |

Vereiste indexen:

| Index                                      | Doel                                                                     |
| ------------------------------------------ | --------------------------------------------------------------------------- |
| unieke `(resolution_ref)`                  | Weiger ambiguïteit tussen kolommen voor `approval_id`/`resolution_ref` tijdens het invoegen. |
| `(status, expires_at_ms)`                  | Zoek openstaande goedkeuringen en stem gezaghebbende deadlines af.               |
| `(source_session_key, created_at_ms DESC)` | Speel recente goedkeuringen voor één bronsessie opnieuw af.                             |
| `(resolved_at_ms)`                         | Verwijder bewaarde terminale goedkeuringen volgens het vaste bewaarbeleid.  |

Doelgroep-arrays zijn klein en begrensd. Op sessie gefilterde replay selecteert eerst zichtbare openstaande rijen via Kysely en decodeert en filtert vervolgens de begrensde doelgroep-arrays in de applicatiecode; hiervoor worden geen tekenreeksovereenkomsten of onbewerkte SQL-JSON-query's gebruikt.

Bewaar terminale rijen 30 dagen, in overeenstemming met de bewaartermijn voor metadata-audits in `src/audit/audit-event-store.ts`. Opschoning is vast onderhoudsbeleid, geen nieuw configuratieoppervlak. De database is privéstatus van het lokale besturingsvlak, maar reviewer-API's mogen nooit het volledige opgeslagen verzoek of de runtimebinding beschikbaar stellen.

## Toestandsmachine en compare-and-set

Alleen deze overgangen zijn geldig:

- `pending -> allowed`: expliciete `allow-once` of `allow-always`.
- `pending -> denied`: expliciete weigering, vertrouwd ongeldig terminaal oordeel of geen afleverroute.
- `pending -> expired`: gezaghebbende deadline bereikt.
- `pending -> cancelled`: afbreken van uitvoering, correcte afsluiting of herstel van verweesde gegevens na herstart.

Elke niet-toegestane terminale status heeft als effectief oordeel weigeren.

Afhandeling gebruikt één onmiddellijke SQLite-transactie en een voorwaardelijke Kysely-update die gelijkwaardig is aan:

```sql
UPDATE operator_approvals
SET status = ?, decision = ?, terminal_reason = ?, resolved_at_ms = ?
WHERE approval_id = ?
  AND status = 'pending'
  AND expires_at_ms > ?;
```

Als de update geen enkele rij beïnvloedt, leest dezelfde transactie de record:

- Ontbrekend of niet-geautoriseerd: retourneer niet gevonden; maak het bestaan niet bekend.
- Nog steeds openstaand, maar deadline bereikt: zet de status via compare-and-set op `expired` en retourneer vervolgens die terminale rij.
- Dezelfde vastgelegde beslissing: retourneer idempotent succes met de vastgelegde winnaar.
- Andere beslissing: de uniforme API retourneert `applied: false` met de vastgelegde winnaar; verouderde adapters behouden `APPROVAL_ALREADY_RESOLVED` waar hun uitgeleverde contract dit vereist.
- Elke terminale status: wijzig deze nooit.

`now == expires_at_ms` is verlopen. De tijd van de Gateway is gezaghebbend.

De uitvoering van `allow-once` gebruikt een tweede CAS voor `consumed_at_ms IS NULL`, gebonden aan de bestaande exacte context van de opdracht/systeemuitvoering. De goedkeuringsrij blijft na verbruik als auditrecord behouden.

Ongeldige HTTP-/RPC-invoer die niet kan worden geauthenticeerd of geen goedkeuring kan identificeren, wordt zonder wijziging geweigerd en kan nooit goedkeuren. Een ongeldig terminaal oordeel dat voor een bekende goedkeuring van een vertrouwde harness/waiter wordt ontvangen, leidt tot een overgang naar `denied`.

## Gateway-API

Voeg type-onafhankelijke reviewermethoden toe:

| Methode                                    | Contract                                                                                                                                                                                                            |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `approval.get { id }`                     | Retourneert een zichtbare openstaande of bewaarde terminale projectie.                                                                                                                                                          |
| `approval.resolve { id, kind, decision }` | Accepteert de canonieke ID of transportreferentie met vaste grootte en voert vervolgens autorisatie, validatie van type en toegestane beslissing, afstemming van de deadline en terminale CAS uit. Het antwoord bevat altijd de canonieke ID. |

Retourneer na een geslaagde CAS onmiddellijk de vastgelegde projectie. Verouderde gebeurtenissen, kanaaldoorstuurders en push-terminaliseerders zijn vervolgstappen op basis van beste inspanning; een langzaam of mislukt oppervlak mag het winnende antwoord niet vertragen of terugdraaien.

Typespecifieke verzoekvalidatie blijft in `exec.approval.request` en `plugin.approval.request`. Bestaande `exec.approval.get/list/waitDecision/resolve` en `plugin.approval.list/waitDecision/resolve` worden adapters op de protocolgrens voor de canonieke service, omdat ze onderdeel zijn van de uitgeleverde Gateway-API. Interne aanroepers migreren in dezelfde wijziging naar de service.

Een reviewerprojectie is een getagde union:

```ts
type OperatorApproval = {
  id: string;
  status: OperatorApprovalStatus;
  presentation:
    | { kind: "exec"; commandText: string /* veilige exec-voorvertoning */ }
    | { kind: "plugin"; title: string; description: string /* veilige Plugin-voorvertoning */ };
  // gemeenschappelijke levenscyclusvelden
};
```

Het stabiele pad wordt afgeleid, niet opgeslagen. `approval.get` retourneert `urlPath`; oppervlakken die een goedgekeurde openbare oorsprong kennen, kunnen ook een absolute `url` ontvangen. Reviewermomentopnamen laten bron- en doelgroepsessiesleutels weg. De Gateway bewaart die routeringssleutels aan de serverzijde voor de afzonderlijke `session.approval`-projectie.

## Gebeurtenissen en overdraagbare acties

PR 1 behoudt de uitgeleverde gebeurtenisnamen, payloads en bestaande ontvangersfilters op recordniveau:

- `exec.approval.requested`
- `exec.approval.resolved`
- `plugin.approval.requested`
- `plugin.approval.resolved`

Die verouderde gebeurtenissen kunnen het volledige runtimeverzoek bevatten en mogen daarom niet naar elke op goedkeuring gerichte client worden uitgewaaierd. PR 5 voegt getagde levenscyclusvelden (`status`, `sourceSessionKey`, `urlPath`, terminale metadata en een `kind` op presentatieniveau) toe via de opgeschoonde levenscyclusprojectie, in plaats van de levering van verouderde gebeurtenissen uit te breiden.

Voeg een op goedkeuring gerichte `session.approval`-projectiegebeurtenis toe. Publiceer de canonieke gebeurtenis eenmaal met de opgeslagen doelgroepsleutels; abonnees van de exacte sessie ontvangen dezelfde gebeurtenis voor elke overeenkomende sleutel:

- `sessionKey`: stream die de projectie ontvangt.
- `sourceSessionKey`: kind/bron waardoor de poort werd geactiveerd.
- `phase`: `pending \| terminal`, gediscrimineerd op basis van de goedkeuringsstatus.
- één veilige `OperatorApproval`-projectie.

Clients melden zich aan met `sessions.messages.subscribe { key, agentId?, includeApprovals: true }`. Het geslaagde antwoord voegt een `approvalReplay` toe met maximaal 1.000 huidige openstaande goedkeuringen voor die exacte streamsleutel waarvoor de abonnerende client ook op recordniveau als reviewer is geautoriseerd. `truncated: false` maakt de gefilterde replay gezaghebbend en opnieuw verbindende clients vervangen hun lokale set met openstaande items door deze set; `truncated: true` is een overbelastingssignaal en clients moeten ongeziene lokale vermeldingen behouden totdat een canonieke opzoekactie of latere levenscyclusgebeurtenissen ze afhandelen. Een latere duurzame time-out die tijdens replay wordt ontdekt, verzendt terminale tombstones uitsluitend naar geabonneerde, op recordniveau geautoriseerde doelgroepen voordat de nieuwe momentopname wordt geretourneerd. `operator.admin` kan zich rechtstreeks aanmelden; beperktere clients vereisen zowel een gekoppelde apparaatidentiteit als `operator.approvals`. Een sessieabonnement alleen verleent nooit zichtbaarheid van goedkeuringen.

Registreer de gebeurtenis onder `operator.approvals` in `src/gateway/server-broadcast.ts`. De projectie is observerend: deze voegt nooit transcriptierijen toe, verzendt geen `sessions.changed` en wekt geen agent.

Breid `MessagePresentationAction` uit in `src/interactive/payload.ts`:

```ts
type MessagePresentationAction =
  | { type: "command"; command: string }
  | { type: "callback"; value: string }
  | {
      type: "approval";
      approvalId: string;
      approvalKind: "exec" | "plugin";
      decision: ExecApprovalDecision;
    }
  | { type: "url"; url: string }
  | { type: "web-app"; url: string };
```

Core bouwt getypeerde beslissingsacties en een afzonderlijke Review-link wanneer een goedgekeurde absolute Control UI-oorsprong beschikbaar is. Kanalen coderen een goedkeuringsactie in hun eigen callback-indeling en sturen de afhandeling naar de canonieke service. Een callback gebruikt de exacte canonieke ID wanneer die past; anders gebruikt deze de unieke volledige digest `resolution_ref` van de rij. De referentie is slechts een compacte opzoeksleutel: normale Gateway-authenticatie, recordautorisatie, expliciet type, validatie van toegestane beslissingen, deadlineafstemming en de CAS waarbij het eerste antwoord wint, blijven van toepassing. Kanalen mogen ID's niet afkappen, hashvoorvoegsels herleiden, `/approve`-tekst parseren of het type afleiden uit een ID-voorvoegsel.

Behoud `button.url`, `button.webApp` en opdrachtgestuurde goedkeuringsbesturingselementen als verouderde compatibiliteitsinvoer voor de plugin-SDK. Normaliseer ze bij de SDK-grens; migreer elke gebundelde interne aanroeper in dezelfde PR. `/approve {id} {decision}` blijft een tekstuele terugvaloptie en CLI-/chatopdracht, niet het semantische contract voor knoppen.

## Control UI

De route is `${basePath}/approve/{approvalId}`. De ID is de enige padparameter; de identiteit van de bronsessie komt uit het record.

Omdat de huidige router exacte statische routes heeft en onbekende paden herschrijft naar Chat, moet deze deeplink in `ui/src/app/bootstrap.ts` worden gedetecteerd vóór de normale routenormalisatie. Hergebruik de normale Gateway-/authenticatie-instellingen, maar render een zelfstandige goedkeuringspagina buiten de zijbalkshell en het algemene modale venster.

Het document is eigendom van de Gateway die de URL heeft aangeboden. De initiële verbinding negeert de opgeslagen externe Gateway-selectie van de volledige app zonder de instellingen van die selectie te wijzigen of te kopiëren; alleen de authenticatie blijft sessiegebonden aan de aanbiedende Gateway. Vertrouwde native authenticatie of een afzonderlijk bevestigde `gatewayUrl`-overschrijving mag de verbinding naar een ander doel leiden. Core reserveert de eensegmentnaamruimte `/approve` vóór plugin-HTTP-routes en detectie van statische extensies, inclusief ID's die eindigen op `.json` of `.js`; wanneer het aanbieden van Control UI is uitgeschakeld, wordt de gereserveerde route fail-closed afgesloten met `404`. Houd de pagina in de hoofd-bundle van Control UI, zodat een mislukte lazy chunk een beveiligingsbeslissing niet laat stranden bij een laadindicator.

Paginastatussen:

- laden
- authenticatie vereist
- in behandeling
- wordt afgehandeld
- hier goedgekeurd of geweigerd
- elders afgehandeld
- verlopen
- geannuleerd
- verboden/niet gevonden
- verbindingsfout met nieuwe poging

De pagina roept Gateway-RPC aan, niet een tweede niet-geverifieerde REST-API. Bij het vernieuwen van de browser wordt de duurzame status opnieuw gelezen. Gateway-aanmeldgegevens worden nooit in de URL, query of het fragment geplaatst.

## Autorisatie en privacy

De URL is een locatieaanwijzer, geen bevoegdheid. Afhandeling vereist:

1. een geauthenticeerde Gateway-verbinding;
2. `operator.approvals` of `operator.admin`;
3. autorisatie van de beoordelaar op recordniveau.

Regels op recordniveau:

- `operator.admin` mag beoordelen.
- `reviewer_device_ids` is gezaghebbend indien aanwezig. Alleen een vermeld gekoppeld
  `operator.approvals`-apparaat mag beoordelen; het verzoekende apparaat heeft geen impliciete
  toegang, tenzij het ook wordt vermeld.
- Zonder een expliciete beoordelaarslijst mag het verzoekende gekoppelde
  `operator.approvals`-apparaat zijn eigen record beoordelen.
- Echt verouderde records zonder binding aan een verzoeker of beoordelaar behouden brede
  zichtbaarheid voor gekoppelde apparaten, zodat upgrades reeds lopend werk niet laten stranden.
- Interne runtimes zonder apparaat mogen via de bereikgebonden
  goedkeuringsruntimeverbinding afhandelen, maar niet lezen. Die bevoegdheid komt uitsluitend van het
  door de server geauthenticeerde runtimetoken; openbare `approval.resolve`-velden kunnen
  dit niet uitgeven.
- Eigenaarschap van de actieve verzoekersverbinding blijft geldig voor verouderde adapters; dit wordt
  nooit afgeleid uit een overeenkomende clientnaam.
- Lidmaatschap van het publiek verandert alleen de presentatie. Het verruimt nooit de autorisatie.

`approval.get` stelt alleen de opgeschoonde beoordelaarsprojectie beschikbaar en laat interne routeringssleutels voor bron en publiek weg. De PR 5-`session.approval`-gebeurtenis bevat de ene bestemming `sessionKey` plus `sourceSessionKey` nadat de Gateway de opgeslagen momentopname van het publiek server-side heeft toegepast. Bestaande exec-/plugingebeurtenissen behouden hun historische payload en beperkte ontvangers totdat consumenten migreren. Het uitvoerbare verzoek, de opdrachtbinding en de voortzetting blijven uitsluitend in de proceslokale waiter. De duurzame rij bevat de veilige presentatie plus levenscyclus-, routerings- en auditmetadata; deze slaat nooit onbewerkte omgevingswaarden, aanmeldgegevens, authenticatieheaders of callbackgegevens van kanalen op.

## Publieksprojectie

Bereken het publiek eenmaal vóór het invoegen en sla de geordende momentopname op. Eigenaarschap is een graaf, niet altijd één bovenliggende keten: een child kan zowel een huidige controller als een oorspronkelijke verzoeker hebben, en die eigenaars kunnen naar verschillende roots leiden.

Gebruik een deterministische breedte-eerstdoorloop:

1. Initialiseer de wachtrij met de sleutel van de bronsessie.
2. Lees voor elke uit de wachtrij gehaalde sleutel de nieuwste registerrij van de subagent en voeg beide afzonderlijke eigenaarschapsranden in vaste volgorde toe aan de wachtrij: `controllerSessionKey`, gevolgd door `requesterSessionKey`.
3. Als er een bruikbare registerrij bestaat, volg dan niet ook de afstamming van de sessievermelding, die na bijsturing verouderd kan zijn. Voeg anders de ene huidige terugvalrand `parentSessionKey ?? spawnedBy` toe aan de wachtrij.
4. Normaliseer en dedupliceer bij het toevoegen aan de wachtrij, zodat het eerste, kortste pad wint.
5. Stop bij 64 unieke sleutels; deze limiet voor de publieksgrootte begrenst ook de doorloopdiepte.

De registerbron is `src/agents/subagent-registry-read.ts`; eigenschapsvelden zijn gedefinieerd in `src/agents/subagent-registry.types.ts`. Terugvalvelden voor sessies zijn gedefinieerd in `src/config/sessions/types.ts`.

Aangevraagde en terminale projecties gebruiken hetzelfde opgeslagen publiek, zelfs als het eigenaarschap van focus/controller verandert terwijl de goedkeuring in behandeling is. Dit garandeert terminale opschoning voor elke publieksessiestroom die de verzoekprojectie heeft ontvangen. Afhandeling is altijd gericht op de bron-ID van de goedkeuring; publieksessies ontvangen nooit gekloonde goedkeuringsstatus. De opschoning van doorgestuurde kanaalberichten blijft de afzonderlijke vervolgactie voor de afleveringslocatie hieronder.

Schrijf niet uitsluitend vanwege een goedkeuring transcriptberichten, injecteer geen systeemprompts, start geen eigenaarbeurten en emitteer geen `sessions.changed`.

## Convergentie van afgeleverde oppervlakken

Native goedkeuringshandlers bewaren hun afgeleverde berichtvermeldingen al lang genoeg om actieve besturingselementen te vervangen of buiten gebruik te stellen. Algemene doorgestuurde goedkeuringsberichten verwijderen momenteel de `MessageReceipt`, waardoor een beslissing op een ander oppervlak hun oude besturingselementen ten onrechte als in behandeling kan blijven weergeven. Een afzonderlijke vervolgactie dicht dit gat met een childtabel `operator_approval_deliveries` in de gedeelde statusdatabase.

Elke rij bevat de goedkeurings-ID, een unieke afleverings-ID, het kanaal/account/de exacte route, een begrensde en met JSON gevalideerde kanaalprivé-berichtlocatie, afleveringstijdstempels en de terminalisatiestatus. De rij bevat nooit callbackgegevens, beslissingstokens of onbewerkte goedkeuringsverzoeken. Het kanaal is eigenaar van de codering van locaties en de wijziging van berichten; core is eigenaar van de canonieke status, doelselectie, het beleid voor nieuwe pogingen en de terminale terugvaltekst.

Afleveringsregistratie en terminale afhandeling verwerken races veilig:

1. Nadat het verzenden van een bericht in behandeling een ontvangstbewijs oplevert, voeg je de afleveringslocatie in en lees je de status van de bovenliggende goedkeuring in één transactie.
2. Als de bovenliggende goedkeuring al terminaal is, plan je onmiddellijke terminalisatie in in plaats van de late aflevering in behandeling te laten.
3. Elke vastgelegde terminale overgang plant afzonderlijk alle nog niet afgeronde afleveringsrijen in; broadcasts die mogen worden genegeerd zijn niet de trigger.
4. Een kanaalterminalisator rapporteert `replaced`, `retired` of `unsupported`. Vervangen onderdrukt een dubbel terminaal bericht; buiten gebruik stellen verzendt de bestaande terminale vervolgmelding; niet ondersteund of een fout leidt tot terugval zonder de CAS van de goedkeuring terug te draaien.
5. Bij het opstarten worden terminale goedkeuringen met onafgeronde afleveringen opnieuw geprobeerd, waardoor opschoning bestand is tegen een herstart van de Gateway.

Deze transportlevenscyclus is een optionele hook voor afleveringsadapters, geen renderer of modelgerichte berichtactie. QQ C2C-/groepsberichten hebben momenteel geen API voor bewerken, verwijderen of het wissen van het toetsenbord; die adapter blijft niet ondersteund en kan de canonieke waarheid pas na een latere klik tonen totdat het transport een mutatie-API krijgt.

## Semantiek voor herstarten, time-outs en routes

SQLite-persistentie impliceert niet dat de uitvoering wordt hervat. Opdracht-/toolbindingen blijven in het geheugen, omdat ze beveiligingsgevoelige runtimefeiten kunnen bevatten en geen hervatbaar taakcontract vormen.

Bij het opstarten van de Gateway:

- genereer je een nieuw runtime-epoch;
- zet je rijen die in behandeling zijn uit oudere epochs atomair om naar `cancelled` met reden `gateway-restart`;
- behoud je rijen zodat hun URL's uitleggen wat er is gebeurd;
- voer je nooit een latere goedkeuring uit tegen een ontbrekende runtimebinding.

Timers zijn optimalisaties voor het ontwaken. De deadlinebevoegdheid wordt opgeslagen in `expires_at_ms`; bij lezen, wachten en afhandelen wordt altijd een verloopafstemming uitgevoerd.

Definitief strikt gedrag:

- time-out -> `expired`, weigeren;
- geen route -> `denied`, weigeren;
- afbreken van uitvoering -> `cancelled`, weigeren;
- ongeldig vertrouwd oordeel -> `denied`, weigeren;
- alleen een toegestane expliciete toestemmingsbeslissing -> `allowed`.

Het momenteel uitgebrachte exec-gedrag is nog steeds in strijd met dit contract:

- `src/agents/bash-tools.exec-host-shared.ts` kan `askFallback` toepassen.
- `docs/tools/exec-approvals.md` en `docs/cli/approvals.md` documenteren dat oppervlak.

Plugin-goedkeuringen worden bij time-outs en ongeldige oordelen nu fail-closed geweigerd; het verouderde
veld `timeoutBehavior` blijft geaccepteerd, maar wordt genegeerd. De vervolgactie voor strikte exec-semantiek
moet code, typen, documentatie, tests en changelog gezamenlijk bijwerken, met
expliciete beoordeling door de eigenaar en beveiliging. `askFallback` mag tijdens
de migratie het beleid voor selectie vóór de gate blijven beschrijven, maar mag de time-out
van een aangemaakt record dat in behandeling is niet omzetten in goedkeuring.

## Compatibiliteitsplan

- Additief Gateway-protocol; geen verhoging van de protocolversie.
- Behoud bestaande exec-/pluginmethoden en -gebeurtenissen aan de externe grens.
- Behoud bestaande ID's, inclusief `plugin:`-voorvoegsels, maar gebruik voorvoegsels niet langer als type-informatie.
- Behoud het gedrag van de tekstopdracht `/approve`.
- Behoud verouderde URL-/Web App-velden voor knoppen en opdrachtacties als compatibiliteitsinvoer voor de plugin-SDK; nieuwe core-uitvoer is getypeerd.
- Migreer alle gebundelde kanalen en interne aanroepers in dezelfde wijziging voor getypeerde acties.
- Voeg een changelogvermelding toe voor de nieuwe URL/pagina en voor de latere wijziging in het time-outgedrag.
- Voeg geen instelling voor de elicitationmodus toe.

## Uitrol

### PR 1: duurzame levenscyclus

- Deze ontwerpnotitie.
- Gedeeld SQLite-schema, Kysely-generatie, opslag en opschoning na 30 dagen.
- Gateway-goedkeuringsservice, bridge voor de runtime-waiter en afhandeling van verweesde records na een herstart.
- Uniforme `approval.get/resolve`.
- Adapters voor exec-/pluginmethoden.
- Tests voor eerste-antwoord-wint, idempotentie, verloop, autorisatie en verbruik.
- Nog geen wijzigingen in UI- of kanaalgedrag.

### PR 2: getypeerde acties en kanaalcallbacks

- Getypeerde goedkeurings-, URL- en Web App-acties.
- Presentatiebouwers in de kern en exports van de plugin-SDK.
- Transportprivé codering van callbacks met expliciet eigenaartype.
- Duurzame callbackverwijzingen met vaste grootte voor canonieke ID's die transportlimieten overschrijden.
- Migratie van gebundelde kanalen weg van afleiding op basis van opdrachttekst en goedkeurings-ID.
- Canonieke waarheid van het eerste antwoord op het aangeklikte oppervlak en best-effort updates van actieve systeemeigen eindstatussen; duurzame terminalisering van kanaalberichten blijft vervolgwerk.
- Tests voor de SDK en gebundelde kanalen.

### PR 3: deeplink voor de Control UI

- Zelfstandige geauthenticeerde goedkeuringspagina en op het basispad afgestemde routering bij het opstarten.
- Binding aan de actieve Gateway zonder de opgeslagen externe selectie van de beheerder te wijzigen.
- Door de kern beheerde HTTP-naamruimte voor goedkeuringen, inclusief assetachtige ID's.
- Door de Gateway opgestelde URL-payload en polling van de status in behandeling totdat levenscyclusgebeurtenissen worden geleverd.
- Bewijs voor mobiele breedte, opnieuw verbinden, concurrerende antwoorden, opnieuw laden en gekoppelde paden.

### PR 4: systeemeigen clients

- Beoordelingsoppervlakken voor iOS en Android gebruiken typebewuste `approval.get/resolve`; watchOS stuurt voor beoordelaars veilige prompts en beslissingen door via de gekoppelde iPhone.
- Watch biedt de uitvoeringsbeslissingen die door het compacte doorstuurcontract worden ondersteund: eenmaal toestaan en weigeren.
- De canonieke eindstatuswaarheid van het eerste antwoord vervangt de lokale status van de gepoogde beslissing.
- Verloren of dubbelzinnige bevestigingen van oplossingen blokkeren bedieningselementen totdat de canonieke status is teruggelezen.
- Eerder uitgebrachte Gateway v4-instanties behouden uitvoeringsbeoordeling via een beperkte terugval op verouderde methoden; behoud van eindstatus tussen oppervlakken vereist de uniforme methoden.
- Waarschuwingen voor beoordelaars en eigenaarscontext blijven zichtbaar op iPhone, Watch en Android.
- Bewijs via systeemeigen unittests, builds en platformtests.

### PR 5: propagatie van de levenscyclus naar voorouders

- Levering van `session.approval` met status in behandeling/eindstatus vanuit de doelgroepsnapshot die in PR 1 is opgeslagen.
- Abonnement op de exacte sessie, herhaling na opnieuw verbinden en eindstatustombstones zonder transcriptmutatie of activering van de agent.
- Levenscyclusterugroepen worden uitgevoerd na duurzame invoeging/CAS en worden nooit autoriteit voor goedkeuring.
- Bewijs voor geneste subagents en opnieuw verbinden.

### PR 6: fail-closed-gedrag

- Migreer `node-invoke-plugin-policy.ts` en de ingebedde pluginbroker weg van dubbele autoriteit.
- Strikte semantiek voor time-outs, misvormde invoer, ontbrekende routes, binding en eenmalige consumptie van toestemmingen.
- Verouder uitgebrachte tolerante time-outinstellingen zonder ze te respecteren nadat een verzoek in behandeling is.
- Bewijs voor conflicten tussen meerdere oppervlakken en foutinjectie.

### Vervolg: duurzame opschoning van externe berichten

- Sla locators voor doorgestuurde leveringen op en terminaliseer elk geleverd kanaalbericht na een herstart.
- Houd deze transportlevenscyclus gescheiden van canonieke goedkeuringsautoriteit en getypeerde presentatieacties.

## Tests

Vereiste gerichte dekking:

- SQLite opnieuw openen behoudt projecties met status in behandeling en eindstatus.
- Twee gelijktijdige oplossers leveren precies één CAS-winnaar op.
- Opnieuw proberen met dezelfde beslissing slaagt idempotent; een conflicterende nieuwe poging retourneert de vastgelegde winnaar.
- Oplossen op of na de deadline kan geen goedkeuring verlenen.
- `allow-once` kan precies eenmaal worden verbruikt zonder de auditstatus van de eindstatus te wissen.
- Bij het opstarten worden oudere runtime-epochs geannuleerd.
- Ongeautoriseerd opzoeken en oplossen onthullen het bestaan van een record niet.
- Expliciete toestemmingslijst voor beoordelaars en algemeen gedrag van gekoppelde `operator.approvals`.
- Verouderde methoden voor uitvoering en plugins delen dezelfde opslag.
- Gateway-schema's voor aanvragen, weergeven, ophalen en oplossen, plus additieve gebeurtenispayloads.
- Normalisatie van getypeerde acties, terugvalweergave, SDK-exports en omschakelingen van gebundelde kanalen.
- Telegram-callbackcodering bevat transportprivégegevens en geen afleiding uit opdrachttekenreeksen.
- Direct kind, vertakte controller-/aanvragereigenaars, geneste eigenaars, hertoewijzing, terugval op sessievelden, cycli en limiet voor doelgroepgrootte.
- Doelgroepmatrices voor aangevraagd en eindstatus zijn identiek.
- Eigenaarsprojecties veroorzaken geen transcriptmutatie of activering van de agent.
- De Control UI-route werkt op `/` en een geconfigureerd basispad; vernieuwen toont de waarheid van de status in behandeling of de eindstatus.
- Gelijktijdige antwoorden via de Control UI en Telegram tonen één winnaar en 'elders opgelost' bij de verliezer.
- Systeemeigen goedkeurings-ID's en Gateway-eigenaars-ID's behouden exacte UTF-8-bytes tijdens routering en reconciliatie.
- Onderhandeling over systeemeigen RPC-families legt per toegestane Gateway-route één canonieke of verouderde familie vast en schakelt na gebruik nooit stilzwijgend terug.
- Verloren systeemeigen bevestigingen van oplossingen blokkeren acties totdat de canonieke status is teruggelezen; een mislukte teruglezing kan geen winnaar verzinnen of een Watch-vernieuwing bevestigen.
- Correlatie van Watch-snapshotaanvragen wordt alleen geaccepteerd voor de exacte gekoppelde Gateway-eigenaar en een voltooide canonieke teruglezing op de iPhone.
- Bewijs van het gebruikerstraject via Testbox/Crabbox, inclusief een goedkeuringspagina op mobiele breedte, opschoning van Telegram-acties en één volledige cyclus van in behandeling/oplossen/late verliezer op Android, iPhone en Watch.

## Observeerbaarheid

Genereer gestructureerde, inhoudsvrije overgangslogboeken met goedkeurings-ID, type, bronsessiesleutel, status, reden en latentie. Log nooit het voorbeeld of de onbewerkte binding.

Volg:

- aantal aangevraagd per type;
- aantal eindstatussen per type/status/reden;
- meter voor in behandeling;
- latentie van aanvraag tot eindstatus;
- uitkomsten van oplossingsraces: winnaar, idempotente nieuwe poging, conflict, verlopen;
- aantal leveringsroutes en weigeringen wegens ontbrekende route;
- annuleringen van verweesde aanvragen bij het opstarten;
- doelgroepgrootte.

Een vastgelegde overgang geldt als geslaagd, zelfs als latere levering van gebeurtenissen mislukt. Levenscyclusabonnees herstellen via herhaling uit PR 5 en canonieke opzoeking. Duurzame terminalisering van kanaalberichten blijft het afzonderlijke vervolgwerk hierboven.

## Openstaande beslissingen

1. **Extern bereikbare oorsprong van de Control UI.** Elke snapshot bevat de stabiele relatieve `urlPath`. Een absolute URL mag alleen worden gepubliceerd vanuit een gecachte Tailscale Serve/Funnel-locatie nadat blootstelling van de Gateway is geslaagd; `allowedOrigins`, Host-headers van aanvragen, `gateway.remote.url` en alleen voor weergave bedoelde loopback-/LAN-kandidaten zijn geen canonieke oorsprongen. Telegram kan zijn geauthenticeerde Mini App-wrapper gebruiken om het goedkeuringspad tijdens het opstarten te behouden. Willekeurige reverse proxy's blijven uitsluitend relatief totdat er een afzonderlijk beoordeeld expliciet contract voor een openbare URL bestaat. Laat een kanaal de oorsprong nooit raden.
2. **Compatibiliteitsomschakeling voor strikte uitvoeringstime-outs.** Time-outs voor plugingoedkeuringen zijn nu fail-closed en `timeoutBehavior` is verouderd. Het resterende uitgebrachte `askFallback`-contract vereist expliciete beoordeling door de eigenaar en beveiliging, een changelog, documentatie en een migratie-/verouderingsbesluit voordat het stopt met het autoriseren van uitvoering nadat een verzoek in behandeling een time-out bereikt.
3. **Ingebedde modus zonder Gateway.** Aanbevolen: houd deze aanvankelijk uitsluitend lokaal en maak er daarna een client van de canonieke service van wanneer er een Gateway bestaat. Publiceer geen deeplink die door geen enkele server kan worden opgelost.
