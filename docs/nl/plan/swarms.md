---
x-i18n:
    generated_at: "2026-07-27T05:10:33Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 90c6c85a837448f4e5ceccdccf73489db801ad502cbbb2f3eb04d6aff7e902f0
    source_path: plan/swarms.md
    workflow: 16
---

# Swarms — agent-fan-out en orkestratie in codemodus

Status: Uitgebracht — vervangen door `docs/tools/swarm.md`. Dit document blijft behouden als
het ontwerpverslag van de implementatie.

## 1. Wat en waarom

Een **swarm** bestaat uit meerdere subagents die deterministisch worden georkestreerd vanuit een
script in codemodus: waaier uit naar N lezers, verifieer bevindingen kritisch, synthetiseer via een
statusbehoudende prioriteerder en herhaal op beslissingspoorten. De besturingsstroom (`Promise.all`,
`while`, `if`) _is_ de orkestratie — er is bewust **geen grafiek-DSL,
geen nieuwe modus en geen nieuw tooloppervlak op topniveau**.

De codemodus van OpenClaw (QuickJS-WASI, momentopname/hervatten, bridge-verzoeken) vormt de
basis. Een geparkeerde bridge-aanroep overleeft een VM-momentopname en herstart van de Gateway,
en wordt precies hervat waar die is gestopt — krachtiger dan ontwerpen met journal-replay, zonder
determinismebeperkingen voor scripts.

Naamgeving: de naam in het product en de documentatie is **Swarm**. Code-identificatoren blijven letterlijk:
`agents.*`-gast-API, `tools.swarm`-configuratie, `swarm`-groepskolommen.

## 2. Besluiten (maintainer, 2026-07-17)

- Kosten: afgedwongen configuratielimieten; tokenbudget per swarm optioneel. Geen verplicht budget.
- Goedkeuringen: kinderen worden **fail-closed/niet-interactief** uitgevoerd. Acties waarvoor
  goedkeuring vereist is, worden geweigerd; de weigering wordt gemeld in het resultaat van het kind; het script
  beslist. Geen overvloed aan operatorprompts door fan-out.
- v1 bestaat uitsluitend uit ad-hocscripts die door het model zijn geschreven. Opgeslagen/benoemde workflows, invoer via
  CLI/Cron: later (headless codemodus bestaat al voor Cron).
- Identiteit van het kind: standaard een speciale worker-agent via de `tools.swarm.defaultAgentId`-
  configuratie (gevalideerd tegen de bestaande allowlist voor subagentdoelen); overschrijving per spawn via
  `agentId`. Core levert geen gebundelde agent-id; de documentatie beveelt een gestroomlijnde
  `worker`-agentconfiguratie aan.
- Geen wijzigingen aan de Codex-broncode. De Codex-harness gebruikt het spawn/wait-idioom (§8).

## 3. Architectuuroverzicht

```
script in codemodus (QuickJS-VM, gateway)       Codex V8-script (Codex-proces)
  agents.run(...) ── geparkeerde bridge-aanroep    tools.sessions_spawn / tools.agents_wait
        │                                                │ item/tool/call-RPC (elk ≤600s)
        ▼                                                ▼
             CORE (harness-agnostisch, deze repository)
  sessions_spawn {collect:true, outputSchema, fastMode, groupId}
  agents_wait {ids, timeoutSeconds}
        │
  subagentregister (SQLite): collector-voltooiingsrecords, swarmgroep-id
        │
  kinderen = gewone subagentsessies (beperkt per lane, fail-closed-goedkeuringen)
        │
  sessions.changed SSE ──► stippen in Control UI / zijbalk / kanaalstatusbericht
```

Eén canonieke eigenaar van de semantiek voor spawnen/voltooien/afhandelen (core-tools + register).
Twee wachttransports: QuickJS parkeert onbeperkt een bridge-aanroep (momentopname);
Codex pollt `agents_wait` in begrensde RPC's.

## 4. Configuratiepoort (v1)

Nieuwe `tools.swarm` (globaal + overschrijving per agent, hetzelfde samenvoegpatroon als
`tools.codeMode`):

```jsonc
"tools": {
  "swarm": {
    "enabled": false,            // hoofdpoort, standaard UIT
    "maxConcurrent": 8,          // tegelijk actieve kinderen (lane-limiet voor swarm)
    "maxChildrenPerGroup": 50,   // actieve kinderen per swarmgroep
    "maxTotalPerGroup": 200,     // totaal aantal spawns gedurende de levensduur per groep (noodrem tegen ontsporing)
    "waitTimeoutSecondsMax": 600,
    "defaultAgentId": ""         // optioneel; agent-id van kind wanneer agentId bij spawn ontbreekt
  }
}
```

- Zod: union `boolean | strict object` zoals `CodeModeSchema`
  (`src/config/zod-schema.agent-runtime.ts`); `swarm: true` → `{enabled: true}`.
- Typen in `src/config/types.tools.ts` (zowel per agent als `tools` op topniveau),
  labels in `schema.labels.ts`, hulptekst in `schema.help.runtime.ts`.
- Resolutiehelper `resolveSwarmConfig(cfg, agentId)`, naar het voorbeeld van
  `resolveCodeModeConfig` (`src/agents/code-mode.ts:215`), met begrenzing van alle getallen.
- Effecten van de poort wanneer uitgeschakeld: de tool `agents_wait` ontbreekt in catalogi;
  `collect`-/`outputSchema`-/`fastMode`-/`groupId`-parameters voor `sessions_spawn`
  worden geweigerd met een duidelijke foutmelding die de configuratiesleutel noemt. Geen andere gedragswijziging.
- `defaultAgentId` wordt gevalideerd via `resolveSubagentAllowedTargetIds`
  (`src/agents/subagent-target-policy.ts`); onbekende id → spawnfout, geen fallback.

## 5. Core: spawn in collectormodus + `agents_wait` (v1)

### 5.1 Toevoegingen aan `sessions_spawn` (allemaal afhankelijk van ingeschakelde swarm)

- `collect: boolean` — wanneer true, wordt de uitvoering van het kind geregistreerd met
  `expectsCompletionMessage: false` en een **collector-voltooiingsrecord**
  in plaats van levering via aankondiging/sturing. De tool retourneert onmiddellijk `{ runId, sessionKey }`.
  Geen binding aan kanaal/thread.
- `outputSchema: object` — JSON Schema. Het kind krijgt een synthetische
  `structured_output`-tool toegevoegd aan zijn tooloppervlak; een aanvulling op de systeemprompt
  instrueert het om die precies eenmaal aan te roepen met het eindresultaat. Bij een validatiefout
  krijgt het kind één aansporing om het opnieuw te proberen; daarna bevat het voltooiingsrecord
  `structured: undefined`, plus de onbewerkte tekst en een `schemaError`.
- `fastMode: true | "auto" | false` — wordt naast model/denkniveau doorgegeven aan de patch van de kindsessie
  via `resolveSubagentModelAndThinkingPlan`
  (`src/agents/subagent-spawn-plan.ts`), met gebruik van de bestaande `FastMode`-as
  (`src/shared/fast-mode.ts`). Weggelaten = overnemen.
- `groupId: string` — stempel voor de swarmgroep. Standaard
  `swarm:<requesterSessionKey>:<runId-of-requesting-run>`. Wordt opgeslagen in het
  registerrecord en de rij van de kindsessie. Wordt gebruikt voor limieten, lijsten, batcharchivering
  en de stippen.
- `label: string` bestaat al — wordt weergegeven in de stippen en `subagents list`.
- Agent-id van kind: `params.agentId` → anders `tools.swarm.defaultAgentId` → anders
  aanvragende agent (bestaand gedrag).

### 5.2 Goedkeuringen fail-closed

Collectorkinderen worden uitgevoerd met een niet-interactieve goedkeuringscontext: elke toolaanroep
waarvoor goedkeuring van een operator vereist zou zijn, resulteert in een gestructureerde weigering
(`approval_required`) die zichtbaar is voor het kind, dat geacht wordt de
blokkering in zijn resultaat te melden. Implementatie: hergebruik de bestaande infrastructuur voor het
goedkeuringsbeleid van exec/tools met een afgedwongen `deny`-resolver voor uitvoeringen van kinderen in collectormodus.
Vanuit collectorkinderen worden geen goedkeuringsgebeurtenissen naar operatoroppervlakken verzonden.

### 5.3 Tool `agents_wait` (nieuw, achter poort)

```
agents_wait({ ids: string[], timeoutSeconds?: number })
→ {
    completed: [{ runId, status: "done"|"failed"|"killed"|"timeout",
                  result: string, structured?: unknown, schemaError?: string,
                  sessionKey, label?, usage?: {inputTokens, outputTokens} }],
    pending: string[]
  }
```

- Retourneert zodra **ten minste één** id is voltooid (semantiek van eerste voltooiing/race,
  maakt pijplijnen mogelijk), of bij een time-out met `completed: []`.
- `timeoutSeconds` standaard 30, begrensd tot `waitTimeoutSecondsMax`.
- Idempotent: reeds voltooide id's retourneren hun records opnieuw (records worden
  bewaard tot de groepsarchivering). Onbekende id → foutitem per id, geen exception.
- Eigendom: alleen de sessie die een uitvoering heeft gespawnd (of de bovenliggende keten ervan) mag erop wachten
  — dezelfde eigendomsregel als `wait` in codemodus (`code-mode.ts:1684`).
- Register: voltooiingsrecords worden opgeslagen in de bestaande SQLite-opslag van het subagentregister
  (`subagent-registry.store.sqlite.ts`) — nieuwe velden, geen nieuwe opslag en
  geen verhoging van de schemaversie (uitsluitend aanvullende kolommen; zie beperking in §9).

### 5.4 Afdwingen van limieten

- `maxConcurrent`: collectorkinderen worden uitgevoerd op de bestaande subagentlane, maar
  per swarmgroep geteld; spawns boven de limiet worden FIFO in de wachtrij geplaatst (aan de hostzijde, in het
  spawnpad — retourneer runId onmiddellijk, uitvoering start zodra een plaats vrijkomt).
- `maxChildrenPerGroup` / `maxTotalPerGroup`: spawn weigert met een getypeerde foutmelding
  zodra de limiet is overschreden; de fouttekst noemt de configuratiesleutel.
- Diepte: collectorkinderen behouden de semantiek van `DEFAULT_SUBAGENT_MAX_SPAWN_DEPTH`
  (kinderen zijn bladeren, tenzij nesting expliciet is geconfigureerd).

## 6. Testcontract (v1, lane A)

- Unit: configuratieresolutie/-begrenzing; poortweigeringen wanneer uitgeschakeld; standaardwaarde voor groupId;
  afdwingen van limieten (wachtrij + weigering); racesemantiek van wait; idempotentie van wait;
  eigendomsweigering; validatie van gestructureerde uitvoer + aansporing om opnieuw te proberen +
  schemaError-pad; doorvoer van fastMode naar sessiepatch; validatie van defaultAgentId.
- Integratie (vitest, nagebootste modelruntime): spawn 3 collectorkinderen, wacht
  in een lus, controleer de volgorde van eerste voltooiingen en de uiteindelijke leegloop; simulatie van
  Gateway-herstart: register opnieuw laden → wait wordt afgehandeld vanuit opgeslagen voltooiing.
- Alle tests staan bij elkaar in `*.test.ts`; geen live modelaanroepen.

## 7. QuickJS-gastoppervlak (lane B, na core)

- Gastglobals geïnstalleerd in `CONTROLLER_SOURCE`
  (`src/agents/code-mode.worker.ts:190-374`), gereserveerde namen toegevoegd in
  `code-mode-namespaces.ts`:
  - `agents.run(prompt, opts) → Promise<result|structured>` — syntactische suiker:
    collectorspawn + geparkeerd wachten via een speciale bridge-methode (`agentWait`)
    die de host bij voltooiing afhandelt (geen polling; veilig voor momentopnamen).
  - `agents.session(system, opts) → Promise<handle>`;
    `handle.send(input, opts) → Promise<...>`; `handle.close()`. (v1.1 —
    wordt na run() uitgebracht; gebruikt `mode:"session"` + collectorrecords per beurt.)
  - `phase(title)`, `log(message)` — bridge-meldingen zonder wachten op resultaat →
    voortgangsgebeurtenissen van de swarm.
- Bridge-methoden toegevoegd aan `CodeModeBridgeMethod` (`code-mode.ts:91`):
  `agentSpawn`, `agentWait`, `swarmNote`. `agentSpawn`/`agentWait` zijn
  **door hun constructie** veilig voor replay: idempotentiesleutel `(codeModeRunId, bridgeId)`
  opgeslagen in het registerrecord; bij herstart wordt opnieuw afgehandeld vanuit opgeslagen voltooiingen
  en worden nooit dubbele spawns uitgevoerd.
- Openstaande `agentWait`-bridge-aanroepen verlengen de TTL van de momentopname van de uitvoering (de set openstaande
  agents is het signaal; geen vlag).
- Het virtuele bestand `API.read("agents.d.ts")` documenteert het getypeerde oppervlak en de
  idiomen voor fan-out/poort/cyclus (`createCodeModeApiVirtualFiles`,
  `code-mode-namespaces.ts:876`).

## 8. Projectie van de Codex-harness (latere lane)

- `sessions_spawn` (met nieuwe parameters) en `agents_wait` lopen via de
  bestaande bridge voor dynamische tools; binnen Codex-scripts in codemodus verschijnen ze automatisch als
  `tools.*` (geverifieerd: `codex-rs/code-mode/src/runtime/globals.rs:14-65`,
  `codex-rs/core/src/tools/spec_plan.rs:448-507`).
- `agents_wait` krijgt de lange time-outklasse voor dynamische tools (limiet van 600s;
  `extensions/codex/src/app-server/dynamic-tool-execution.ts:37-39`) en wordt
  gemarkeerd als veilig voor time-out/replay.
- Groepssleutel voor Codex-ouders: `swarm:<parentSessionKey>:<turnId>`.
- Codex-eigen `spawn_agent`-subagents bestaan naast elkaar; hun taakspiegelrijen voeden
  hetzelfde voortgangsoppervlak.

## 9. Persistentie en bewaring

- Geen nieuwe opslagplaatsen. Registerrecords breiden de bestaande SQLite-tabellen van het subagentregister
  uit; kinderen zijn gewone `sessions`-rijen. Uitsluitend aanvullende kolommen
  — **voor elke wijziging waarvoor de versie van het SQLite-schema moet worden verhoogd, is eerst
  expliciete goedkeuring van een maintainer vereist** (repositorybeleid).
- Swarmgroep-id in registerrecord + metadata van kindsessie.
- Bewaring: voltooide collectorrecords blijven bestaan tot **groepsarchivering**:
  wanneer de uitvoering van de ouder is voltooid (of de TTL verloopt), worden de kinderen van de groep
  als batch gearchiveerd (breid de bestaande `DEFAULT_SUBAGENT_ARCHIVE_AFTER_MINUTES`-
  opschoning uit zodat deze per groep werkt).

## 10. Voortgangsoppervlak ("de stippen") — latere lane

- Impliciet, aangestuurd door de harness. Afgeleid van bestaande `sessions.changed`-SSE +
  register; `phase`-/`log`-notities voegen semantiek toe. Geen door agents aangestuurde rendering.
- Control UI: `swarm`-renderer in de familie van workspace-widgets
  (`ui/src/lib/workspace/widgets/`) — stippenraster gegroepeerd per fase, vertellerregel,
  status/label/model per stip; kindboom in zijbalk ongewijzigd.
- Kanalen: één beperkt bijgewerkt statusbericht per groep (volg
  `docs/concepts/streaming.md`; nooit berichten per kind).

## 11. Labs-pagina (Control UI, onafhankelijk traject)

Settings → **Labs**: schakelaars voor experimentele functies, met als eerste opties **Code Mode**
en **Swarm**. Elke rij: naam, beschrijving van één regel, documentatielink, schakelaar gekoppeld
via de bestaande `config.patch`-RPC (RFC 7396 merge-patch — stel
`tools.codeMode.enabled` / `tools.swarm.enabled` in), plus indien van toepassing de
hint "opnieuw opstarten vereist". Vindbaar, maar de tekst maakt de experimentele status
duidelijk. i18n: alle tekenreeksen via de normale `en.ts` + synchronisatiepijplijn.

## 12. Plaatsing (later)

- `placement`-keuze bij starten: `"local"` (standaard) | `"cloud:<profile>"` via
  de bestaande dispatch voor werkomgevingen (`sessions.dispatch`); gepoolde plaatsing
  later als SSH-sandboxkinderen op een gedeelde box onvoldoende blijken.
- De orchestrator-VM blijft altijd op de Gateway; settle/dots/budget zijn
  plaatsingsonafhankelijk.

## 13. Niet-doelstellingen

- Geen grafiek-DSL — de besturingsstroom is de grafiek (bewust, gedocumenteerd).
- Geen wijzigingen aan de Codex-broncode; geen hergebruik van interne onderdelen van Codex Code Mode.
- Geen opgeslagen/benoemde workflows in v1; geen CLI-ingangspunt.
- Geen escalatie van operatorgoedkeuring per kind.
- Geen 1:1-cloudprovisioning op fan-out-schaal.
- Geen compatibiliteitsshims in de stabiele runtime; swarm is een nieuw, afgeschermd oppervlak.

## 14. Bouwfasen / opsplitsing van PR's

1. **Traject A (core)**: §4-configuratie + §5 starten/wachten/limieten/goedkeuringen + §6-tests.
2. **Traject C (Labs-pagina)**: §11 — onafhankelijk, kan als eerste worden samengevoegd.
3. **Traject B (QuickJS-oppervlak)**: §7 — nadat de contracten van A zijn samengevoegd.
4. Dots-renderer (§10), Codex-projectie (§8), `agents.session` (§7 v1.1),
   plaatsing (§12), herschrijving van gebruikersdocumentatie — vervolg-PR's in die volgorde.

Elke PR: groene CI, `$autoreview` schoon, standaard uitgeschakeld, main gereed voor uitgifte.
