---
summary: Doorverwijzen naar /plugins/sdk-channel-outbound
title: API voor kanaalberichten
x-i18n:
    generated_at: "2026-07-27T05:27:55Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bf0d607bd3287233cbb1fe47c15958bf57a81267ae1e37e45a1881f56e1370cb
    source_path: plugins/sdk-channel-message.md
    workflow: 16
---

Deze pagina is verplaatst naar [API voor uitgaande kanalen](/nl/plugins/sdk-channel-outbound).

`openclaw/plugin-sdk/channel-message` blijft een verouderd compatibiliteits-
subpad voor oudere plugins. Nieuwe kanaalplugins moeten
`openclaw/plugin-sdk/channel-outbound` gebruiken voor de levenscyclus van berichten, ontvangstbevestigingen,
duurzaam verzenden en helpers voor livevoorbeelden, in plaats van nieuwe helpers toe te voegen aan het
verouderde subpad.

Verwijderingsplan: behoud deze aliassen gedurende de migratieperiode voor externe plugins
en verwijder ze vervolgens tijdens de volgende grote opschoning van de SDK, nadat aanroepers
zijn overgestapt op `channel-outbound`.
