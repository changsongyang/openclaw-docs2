---
x-i18n:
    generated_at: "2026-07-27T04:46:31Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: a8712b1aeb2e605055c22cf308049e5e74fdf33061870026be20bd55cb0c3d1d
    source_path: AGENTS.md
    workflow: 16
---

# Documentatiehandleiding

Deze map beheert het schrijven van documentatie, Mintlify-linkregels en het i18n-beleid voor documentatie.

## Mintlify-regels

- Documentatie wordt gehost op Mintlify (`https://docs.openclaw.ai`).
- Interne documentatielinks in `docs/**/*.md` moeten hoofdmaprelatief blijven zonder het achtervoegsel `.md` of `.mdx` (voorbeeld: `[Config](/gateway/configuration)`).
- Kruisverwijzingen naar secties moeten ankers op hoofdmaprelatieve paden gebruiken (voorbeeld: `[Hooks](/gateway/configuration-reference#hooks)`).
- Documentatiekoppen moeten gedachtestreepjes en apostroffen vermijden, omdat Mintlify daarbij onbetrouwbaar ankers genereert.
- README en andere door GitHub weergegeven documentatie moeten absolute documentatie-URL's behouden, zodat links buiten Mintlify werken.
- De inhoud van documentatie moet generiek blijven: geen persoonlijke apparaatnamen, hostnamen of lokale paden; gebruik tijdelijke aanduidingen zoals `user@gateway-host`.

## Regels voor documentatie-inhoud

- Sorteer services/providers in documentatie, UI-tekst en keuzelijsten alfabetisch, tenzij de sectie expliciet de uitvoeringsvolgorde of volgorde van automatische detectie beschrijft.
- Houd de naamgeving van gebundelde plugins consistent met de terminologieregels voor plugins in de hele repository in het `AGENTS.md` in de hoofdmap.
- Gegenereerde documentatie mag nooit handmatig worden bewerkt: `docs/plugins/reference/**`, `docs/plugins/reference.md` en `docs/plugins/plugin-inventory.md` zijn afkomstig van `pnpm plugins:inventory:gen`; `docs/docs_map.md` van `pnpm docs:map:gen`; `docs/maturity/**` van `pnpm maturity:render`.

## Interne documentatie

- Langdurig gebruikte privédocumentatie voor operators hoort thuis in `~/Projects/manager/docs/`.
- Interne klad- of spiegeldocumentatie die lokaal bij de repository hoort, mag onder de genegeerde map `docs/internal/` staan.
- Voeg pagina's uit `docs/internal/**` nooit toe aan de navigatie van `docs/docs.json` en link er niet naar vanuit openbare documentatie.
- `scripts/docs-sync-publish.mjs` sluit `docs/internal/**` uit en verwijdert deze uit de openbare publicatierepository `openclaw/docs` als een pagina later geforceerd wordt toegevoegd.
- Interne documentatie mag repositorypaden, namen van privé-apps, namen van 1Password-items en draaiboeken vermelden, maar mag nooit geheime waarden bevatten.

## De volwassenheidsscorekaart bewerken

`taxonomy.yaml` en `qa/maturity-scores.yaml` zijn de broninvoer; gegenereerde volwassenheidsdocumentatie onder `docs/maturity/` bestaat uit projecties en mag niet handmatig worden bewerkt voor scores, LTS, taxonomie, QA-profielen of bewijstabellen.
`scripts/qa/render-maturity-docs.ts` beheert het genereren; gebruik `pnpm maturity:render` om vastgelegde documentatie te vernieuwen en `pnpm maturity:check` om deze te verifiëren.
`.github/workflows/maturity-scorecard.yml` geeft voorbeelden van artefacten weer en kan pull requests voor gegenereerde documentatie openen; `.github/workflows/openclaw-release-checks.yml` start dit voor release-QA.
Bewaar deterministische `qa-evidence.json.scorecard`-gegevens in GitHub Actions-artefacten, tenzij een maintainer expliciet om een opgeschoonde vastgelegde projectie vraagt.
Menselijke overschrijvingen moeten de bronstatus in een pull request wijzigen en de reden plus openbaar of geredigeerd bewijs toelichten.

## i18n voor documentatie

- Documentatie in andere talen wordt niet in deze repository onderhouden. De gegenereerde publicatie-uitvoer staat in de afzonderlijke repository `openclaw/docs` (vaak lokaal gekloond als `../openclaw-docs`).
- Voeg hier geen gelokaliseerde documentatie onder `docs/<locale>/**` toe en bewerk deze niet.
- Beschouw de Engelse documentatie in deze repository plus de woordenlijstbestanden als de bron van waarheid.
- Pijplijn: werk hier de Engelse documentatie bij, werk indien nodig `docs/.i18n/glossary.<locale>.json` bij en laat vervolgens de synchronisatie van de publicatierepository en `scripts/docs-i18n` in `openclaw/docs` uitvoeren.
- Voeg voordat je `scripts/docs-i18n` opnieuw uitvoert woordenlijstvermeldingen toe voor nieuwe technische termen, paginatitels of korte navigatielabels die in het Engels moeten blijven of een vaste vertaling moeten gebruiken.
- `pnpm docs:check-i18n-glossary` is de controle voor gewijzigde Engelse documentatietitels en korte interne documentatielabels.
- Het vertaalgeheugen staat in gegenereerde `docs/.i18n/*.tm.jsonl`-bestanden in de publicatierepository.
- Zie `docs/.i18n/README.md`.
