---
read_when:
    - Den richtigen plugin-sdk-Unterpfad für einen Plugin-Import auswählen
    - Prüfen von Unterpfaden gebündelter Plugins und Hilfsoberflächen
summary: 'Plugin-SDK-Unterpfadkatalog: Welche Importe wo zu finden sind, gruppiert nach Bereich'
title: Plugin-SDK-Unterpfade
x-i18n:
    generated_at: "2026-07-24T20:39:07Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 58df43436d0e26f1ffa1383be47fd108655e57d61cf5534d650a4fa2fb7b364c
    source_path: plugins/sdk-subpaths.md
    workflow: 16
---

Das Plugin-SDK enthält eng abgegrenzte öffentliche Unterpfade und nur für das Repository bestimmte gebündelte
Hilfsfunktionen unter `openclaw/plugin-sdk/`. Diese Seite katalogisiert beide und kennzeichnet
private-lokale Einträge ausdrücklich. Drei Dateien definieren die Grenze:

- `scripts/lib/plugin-sdk-entrypoints.json`: das gepflegte Inventar der Einstiegspunkte,
  das der Build kompiliert.
- `scripts/lib/plugin-sdk-private-local-only-subpaths.json`: interne Unterpfade,
  die vom typisierten, dokumentierten SDK ausgeschlossen sind. Produktionseinträge bleiben
  als reine JavaScript-Exporte der Host-Laufzeit für separat veröffentlichte offizielle
  Plugins verfügbar; reine Testeinträge bleiben nicht exportiert.
- `src/plugin-sdk/entrypoints.ts`: Klassifizierungsmetadaten für veraltete
  Unterpfade, reservierte gebündelte Hilfsfunktionen, unterstützte gebündelte Fassaden und
  Plugin-eigene öffentliche Oberflächen.

Maintainer prüfen die Anzahl öffentlicher Exporte mit `pnpm plugin-sdk:surface` und
aktive reservierte Hilfsunterpfade mit `pnpm plugins:boundary-report:summary`;
ungenutzte reservierte Hilfsexporte lassen den CI-Bericht fehlschlagen, statt als
inaktive Kompatibilitätsschuld im öffentlichen SDK zu verbleiben.

Den Leitfaden zur Plugin-Erstellung finden Sie unter [Übersicht zum Plugin-SDK](/de/plugins/sdk-overview).

## Plugin-Einstieg

| Unterpfad                      | Wichtige Exporte                                                                                                                                                                                         |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `plugin-sdk/plugin-entry`      | `definePluginEntry`                                                                                                                                                                                     |
| `plugin-sdk/core`              | `defineChannelPluginEntry`, `createChatChannelPlugin`, `createChannelPluginBase`, `defineSetupPluginEntry`, `buildChannelConfigSchema`, `buildJsonChannelConfigSchema`, `resolveTailscalePublishedHost` |
| `plugin-sdk/provider-entry`    | Seit Juli 2026 private-lokal; `defineSingleProviderPluginEntry`                                                                                                                                        |
| `plugin-sdk/migration`         | Seit Juli 2026 private-lokal; Hilfsfunktionen für Migration-Provider-Elemente wie `createMigrationItem`, Ursachenkonstanten, Elementstatusmarkierungen, Schwärzungshilfen und `summarizeMigrationItems`                   |
| `plugin-sdk/migration-runtime` | Seit Juli 2026 private-lokal; Laufzeit-Migrationshilfen wie `copyMigrationFileItem`, `resolvePlannedMigrationTargets`, `withCachedMigrationConfigRuntime` und `writeMigrationReport`              |
| `plugin-sdk/health`            | Registrierung, Erkennung, Reparatur, Auswahl sowie Schweregrad- und Befundtypen für Doctor-Zustandsprüfungen gebündelter Zustandsverbraucher                                                             |

### Kompatibilität und private-lokale Hilfsfunktionen

Nur die in einem späteren Zeitfenster veralteten Unterpfade bleiben exportiert. Aliasse vom
Juli 2026 und ungenutzte Unterpfade wurden gelöscht, während ausschließlich gebündelte Hilfsfunktionen aus dem
öffentlichen Paket entfernt wurden und nachfolgend als private-lokal gekennzeichnet sind. Die gepflegte Liste ist
`scripts/lib/plugin-sdk-deprecated-public-subpaths.json`; die CI weist in gebündeltem Code verwendete
Importe aus dieser Liste zurück. `plugin-sdk/text-runtime` ist ausschließlich zur Kompatibilität vorgesehen, und `plugin-sdk/zod` ist ein
Kompatibilitäts-Reexport: Importieren Sie `zod` direkt aus `zod`. Die breiten Domänen-
Barrels `plugin-sdk/agent-runtime`, `plugin-sdk/channel-lifecycle`,
`plugin-sdk/conversation-runtime`, `plugin-sdk/hook-runtime`,
`plugin-sdk/media-runtime`, `plugin-sdk/plugin-runtime` und
`plugin-sdk/security-runtime` sind zugunsten fokussierter
Unterpfade ebenfalls veraltet.

Die Vitest-basierten Testhilfs-Unterpfade von OpenClaw sind ausschließlich repository-lokal und werden nicht
mehr als Paketexporte bereitgestellt: `agent-runtime-test-contracts`,
`channel-contract-testing`, `channel-target-testing`, `channel-test-helpers`,
`plugin-state-test-runtime`, `plugin-test-api`, `plugin-test-contracts`,
`plugin-test-runtime`, `provider-http-test-mocks`, `provider-test-contracts`,
`reply-payload-testing`, `sqlite-runtime-testing`, `test-env`, `test-fixtures`,
`test-live`, `test-live-auth`, `test-media-generation`,
`test-media-understanding`, `test-node-mocks` und `testing`. Die privaten gebündelten Hilfsoberflächen
`ssrf-runtime-internal` und `codex-native-task-runtime` sind ebenfalls ausschließlich
repository-lokal.

### Hilfsunterpfade gebündelter Plugins

Ausschließlich gebündelte Hilfsmodule sind seit der Bereinigung im Juli 2026 private-lokal. Besitzerübergreifende Importe werden durch Schutzmechanismen des Paketvertrags blockiert. `src/plugin-sdk/entrypoints.ts` erfasst separat die unterstützten gebündelten Fassaden, die öffentlich bleiben, also SDK-
Einstiegspunkte, die von ihrem gebündelten Plugin bereitgestellt werden, bis generische Verträge
`plugin-sdk/qa-runner-runtime`, `plugin-sdk/telegram-account` ersetzen,
für neuen Code veraltet; beachten Sie die Hinweise in den einzelnen Zeilen unten.

<AccordionGroup>
  <Accordion title="Kanal-Unterpfade">
    | Unterpfad | Wichtige Exporte |
    | --- | --- |
    | `plugin-sdk/channel-core` | `defineChannelPluginEntry`, `defineSetupPluginEntry`, `createChatChannelPlugin`, `createChannelPluginBase`, `createChannelConfigUiHints` |
    | `plugin-sdk/json-schema-runtime` | Seit Juli 2026 private-lokal; zwischengespeicherte JSON-Schema-Validierungshilfe für Plugin-eigene Schemas |
    | `plugin-sdk/channel-setup` | `defineChannelSetupContract`, kanaleigene Typen für Einrichtungsfelder/-eingaben, `createOptionalChannelSetupSurface`, `createOptionalChannelSetupAdapter`, `createOptionalChannelSetupWizard` sowie `DEFAULT_ACCOUNT_ID`, `createTopLevelChannelDmPolicy`, `setSetupChannelEnabled`, `splitSetupEntries` |
    | `plugin-sdk/setup` | Gemeinsame Hilfsfunktionen für den Einrichtungsassistenten, Einrichtungsübersetzer, Eingabeaufforderungen für Zulassungslisten, Builder für den Einrichtungsstatus |
    | `plugin-sdk/setup-runtime` | `defineChannelSetupContract`, `createSetupTranslator`, `createPatchedAccountSetupAdapter`, `createEnvPatchedAccountSetupAdapter`, `createSetupInputPresenceValidator`, `noteChannelLookupFailure`, `noteChannelLookupSummary`, `promptResolvedAllowFrom`, `splitSetupEntries`, `createAllowlistSetupWizardProxy`, `createDelegatedSetupWizardProxy` |
    | `plugin-sdk/setup-tools` | `formatCliCommand`, `detectBinary`, `extractArchive`, `resolveBrewExecutable`, `formatDocsLink`, `CONFIG_DIR` |
    | `plugin-sdk/account-core` | Hilfsfunktionen für Mehrkontokonfiguration/Aktionssperren, Hilfsfunktionen für den Fallback auf das Standardkonto |
    | `plugin-sdk/account-id` | `DEFAULT_ACCOUNT_ID`, Hilfsfunktionen zur Normalisierung von Konto-IDs |
    | `plugin-sdk/account-resolution` | Hilfsfunktionen für Kontosuche und Standard-Fallback |
    | `plugin-sdk/account-helpers` | Eng abgegrenzte Hilfsfunktionen für Kontolisten/Kontoaktionen |
    | `plugin-sdk/access-groups` | Seit Juli 2026 private-lokal; Hilfsfunktionen zum Parsen von Zugriffsgruppen-Zulassungslisten und für geschwärzte Gruppendiagnosen |
    | `plugin-sdk/channel-pairing` | `createChannelPairingController` |
    | `plugin-sdk/channel-reply-pipeline` | Veraltete Kompatibilitätsfassade. Verwenden Sie `plugin-sdk/channel-outbound`. |
    | `plugin-sdk/channel-config-helpers` | `createHybridChannelConfigAdapter`, `resolveChannelDmAccess`, `resolveChannelDmAllowFrom`, `resolveChannelDmPolicy`, `normalizeChannelDmPolicy`, `normalizeLegacyDmAliases` |
    | `plugin-sdk/channel-config-schema` | Gemeinsame Primitive für Kanalkonfigurationsschemas sowie Zod- und direkte JSON-/TypeBox-Builder |
    | `plugin-sdk/bundled-channel-config-schema` | Seit Juli 2026 private-lokal; Konfigurationsschemas gebündelter OpenClaw-Kanäle ausschließlich für gepflegte gebündelte Plugins |
    | `plugin-sdk/chat-channel-ids` | Seit Juli 2026 private-lokal; `BUNDLED_CHAT_CHANNEL_IDS`, `BUNDLED_CHAT_CHANNEL_ENVELOPE_PREFIXES`, `ChatChannelId`. Kanonische IDs gebündelter/offizieller Chatkanäle sowie Formatiererbezeichnungen/-aliasse für Plugins, die Text mit vorangestelltem Umschlagpräfix erkennen müssen, ohne eine eigene Tabelle fest zu codieren. |
    | `plugin-sdk/channel-policy` | `resolveChannelGroupRequireMention` |
    | `plugin-sdk/channel-ingress-runtime` | Experimenteller übergeordneter Laufzeit-Resolver für eingehende Kanalnachrichten, Resolver für Richtlinien zu impliziten Erwähnungen und Builder für Routing-Fakten für migrierte Empfangspfade von Kanälen. Bevorzugen Sie dies gegenüber dem Zusammenstellen effektiver Zulassungslisten, Befehlszulassungslisten und veralteter Projektionen in jedem Plugin. Siehe [API für eingehende Kanalnachrichten](/de/plugins/sdk-channel-ingress). |
    | `plugin-sdk/channel-lifecycle` | Veraltete Kompatibilitätsfassade. Verwenden Sie `plugin-sdk/channel-outbound`. |
    | `plugin-sdk/channel-outbound` | Verträge für den Nachrichtenlebenszyklus sowie Optionen für die Antwort-Pipeline, Empfangsbestätigungen, Live-Vorschau/Streaming, Lebenszyklus-Hilfsfunktionen, ausgehende Identität, Nutzlastplanung, dauerhafte Sendevorgänge und Hilfsfunktionen für den Nachrichtenversandkontext. Siehe [API für ausgehende Kanalnachrichten](/de/plugins/sdk-channel-outbound). |
    | `plugin-sdk/channel-message` | Veralteter Kompatibilitätsalias für `plugin-sdk/channel-outbound`. |
    | `plugin-sdk/inbound-envelope` | Gemeinsame Hilfsfunktionen zum Erstellen eingehender Routen und Umschläge |
    | `plugin-sdk/inbound-reply-dispatch` | Veraltete Kompatibilitätsfassade. Verwenden Sie `plugin-sdk/channel-inbound` für Runner eingehender Nachrichten und Dispatch-Prädikate sowie `plugin-sdk/channel-outbound` für Hilfsfunktionen zur Nachrichtenzustellung. |
    | `plugin-sdk/messaging-targets` | Veralteter Alias zum Parsen von Zielen; verwenden Sie `plugin-sdk/channel-targets` |
    | `plugin-sdk/outbound-media` | Seit Juli 2026 private-lokal; gemeinsame Hilfsfunktionen zum Laden ausgehender Medien und für den Zustand gehosteter Medien |
    | `plugin-sdk/poll-runtime` | Seit Juli 2026 private-lokal; eng abgegrenzte Hilfsfunktionen zur Umfragenormalisierung |
    | `plugin-sdk/thread-bindings-runtime` | Seit Juli 2026 private-lokal; Hilfsfunktionen für den Lebenszyklus und Adapter von Thread-Bindungen |
    | `plugin-sdk/agent-media-payload` | Veraltete Kompatibilitätsfassade für die veraltete Nutzlastprojektion `Media*`. Übergeben Sie geordnete Fakten über `MsgContext.media` / `toInboundMediaFacts(...)`; importieren Sie die Richtlinie für lokale Wurzeln aus `plugin-sdk/media-local-roots`. |
    | `plugin-sdk/conversation-runtime` | Veraltetes breites Barrel für Konversations-/Thread-Bindung, Kopplung und Hilfsfunktionen für konfigurierte Bindungen; bevorzugen Sie fokussierte Bindungsunterpfade wie `plugin-sdk/thread-bindings-runtime` und `plugin-sdk/session-binding-runtime` |
    | `plugin-sdk/runtime-group-policy` | Hilfsfunktionen zur Auflösung von Gruppenrichtlinien zur Laufzeit |
    | `plugin-sdk/channel-status` | Gemeinsame Hilfsfunktionen für Momentaufnahmen/Zusammenfassungen des Kanalstatus |
    | `plugin-sdk/channel-config-primitives` | Eng abgegrenzte Primitive für Kanalkonfigurationsschemas |
    | `plugin-sdk/channel-config-writes` | Seit Juli 2026 private-lokal; Autorisierungshilfen zum Schreiben der Kanalkonfiguration |
    | `plugin-sdk/channel-plugin-common` | Gemeinsame Prelude-Exporte für Kanal-Plugins |
    | `plugin-sdk/allowlist-config-edit` | Hilfsfunktionen zum Bearbeiten/Lesen der Zulassungslistenkonfiguration |
    | `plugin-sdk/group-access` | Veraltete Hilfsfunktionen für Entscheidungen zum Gruppenzugriff; verwenden Sie `resolveChannelMessageIngress` aus `plugin-sdk/channel-ingress-runtime` |
    | `plugin-sdk/direct-dm-guard-policy` | Seit Juli 2026 private-lokal; eng abgegrenzte Richtlinienhilfen für die direkte DM-Prüfung vor der Kryptografie |
    | `plugin-sdk/discord` | Veraltete Discord-Kompatibilitätsfassade für das veröffentlichte `@openclaw/discord@2026.3.13` und nachverfolgte Besitzerkompatibilität; neue Plugins sollten generische Kanal-SDK-Unterpfade verwenden |
    | `plugin-sdk/telegram-account` | Veraltete Telegram-Kompatibilitätsfassade für die Kontoauflösung und nachverfolgte Besitzerkompatibilität; neue Plugins sollten injizierte Laufzeithilfen oder generische Kanal-SDK-Unterpfade verwenden |
    | `plugin-sdk/interactive-runtime` | Semantische Nachrichtendarstellung, Zustellung und veraltete Hilfsfunktionen für interaktive Antworten. Siehe [Nachrichtendarstellung](/de/plugins/message-presentation) |
    | `plugin-sdk/question-gateway-runtime` | Von der Laufzeit erstellte `ask_user`-Auswahlmöglichkeiten aus Interaktionshandlern von Kanälen über das Gateway auflösen |
    | `plugin-sdk/channel-inbound` | Gemeinsame Hilfsfunktionen für eingehende Nachrichten zur Ereignisklassifizierung, Kontexterstellung, Formatierung, Wurzeln, Entprellung, Erwähnungsabgleich, Erwähnungsrichtlinie und Protokollierung eingehender Nachrichten |
    | `plugin-sdk/channel-inbound-debounce` | Eng abgegrenzte Entprellungshilfen für eingehende Nachrichten |
    | `plugin-sdk/channel-mention-gating` | Seit Juli 2026 private-lokal; eng abgegrenzte Hilfsfunktionen für Erwähnungsrichtlinien, Erwähnungsmarkierungen und Erwähnungstext ohne die breitere Laufzeitoberfläche für eingehende Nachrichten |
    | `plugin-sdk/channel-streaming` | Veraltete Kompatibilitätsfassade. Verwenden Sie `plugin-sdk/channel-outbound`. |
    | `plugin-sdk/channel-send-result` | Typen für Antwortergebnisse |
    | `plugin-sdk/channel-actions` | Hilfsfunktionen für Kanalnachrichtenaktionen sowie veraltete native Schemahilfen, die zur Plugin-Kompatibilität beibehalten werden |
    | `plugin-sdk/channel-route` | Seit Juli 2026 private-lokal; gemeinsame Hilfsfunktionen für Routing-Normalisierung, parsergestützte Zielauflösung, Umwandlung von Thread-IDs in Zeichenfolgen, deduplizierte/kompakte Routenschlüssel, Typen geparster Ziele sowie Routen-/Zielvergleiche |
    | `plugin-sdk/channel-targets` | Seit Juli 2026 private-lokal; Hilfsfunktionen zum Parsen von Zielen; Aufrufer für Routenvergleiche sollten `plugin-sdk/channel-route` verwenden |
    | `plugin-sdk/channel-contract` | Typen für Kanalverträge |
    | `plugin-sdk/channel-feedback` | Verknüpfung von Feedback/Reaktionen |
  </Accordion>

Kanal-Kompatibilitätsunterpfade aus späteren Zeitfenstern bleiben nur bis zu ihren
Registrierungsdaten öffentlich. Juli-Aliasse wie direkter DM-Zugriff, Antwortoptionen, Kopplungs-
pfade und Aufspaltungen der Kanallaufzeit wurden entfernt; ausschließlich gebündelte Hilfsfunktionen
sind private-lokal.

  <Accordion title="Provider-Unterpfade">
    | Unterpfad | Wichtige Exporte |
    | --- | --- |
    | `plugin-sdk/provider-entry` | Seit Juli 2026 nur noch privat-lokal; `defineSingleProviderPluginEntry` |
    | `plugin-sdk/provider-setup` | Seit Juli 2026 nur noch privat-lokal; kuratierte Hilfsfunktionen für die Einrichtung lokaler/selbst gehosteter Provider |
    | `plugin-sdk/cli-backend` | Seit Juli 2026 nur noch privat-lokal; CLI-Backend-Standardwerte und Watchdog-Konstanten |
    | `plugin-sdk/provider-auth-runtime` | Seit Juli 2026 nur noch privat-lokal; Laufzeithilfen für die Provider-Authentifizierung: OAuth-Loopback-Ablauf, Token-Austausch, Authentifizierungspersistenz und API-Schlüssel-Auflösung |
    | `plugin-sdk/provider-oauth-runtime` | Seit Juli 2026 nur noch privat-lokal; generische OAuth-Callback-Typen für Provider, Darstellung der Callback-Seite, PKCE-/State-Hilfsfunktionen, Parsing der Autorisierungseingabe, Hilfsfunktionen für den Token-Ablauf und Abbruchhilfen |
    | `plugin-sdk/provider-auth-api-key` | Seit Juli 2026 nur noch privat-lokal; Hilfsfunktionen für das API-Schlüssel-Onboarding und das Schreiben von Profilen wie `upsertApiKeyProfile` |
    | `plugin-sdk/provider-auth-result` | Seit Juli 2026 nur noch privat-lokal; Standard-Builder für OAuth-Authentifizierungsergebnisse |
    | `plugin-sdk/provider-env-vars` | Seit Juli 2026 nur noch privat-lokal; Hilfsfunktionen zum Nachschlagen von Umgebungsvariablen für die Provider-Authentifizierung |
    | `plugin-sdk/provider-auth` | `createProviderApiKeyAuthMethod`, `ensureApiKeyFromOptionEnvOrPrompt`, `upsertAuthProfile`, `upsertApiKeyProfile`, `writeOAuthCredentials`, Hilfsfunktionen zum Importieren der OpenAI-Codex-Authentifizierung, veralteter Kompatibilitätsexport `resolveOpenClawAgentDir` |
    | `plugin-sdk/provider-model-shared` | Seit Juli 2026 nur noch privat-lokal; `ProviderReplayFamily`, `buildProviderReplayFamilyHooks`, `selectPreferredLocalModelId`, `normalizeModelCompat`, gemeinsam genutzte Builder für Wiederholungsrichtlinien, Hilfsfunktionen für Provider-Endpunkte und gemeinsam genutzte Hilfsfunktionen zur Normalisierung von Modell-IDs |
    | `plugin-sdk/provider-catalog-live-runtime` | Seit Juli 2026 nur noch privat-lokal; Hilfsfunktionen für Live-Modellkataloge von Providern zur abgesicherten Erkennung im Stil von `/models`: `buildLiveModelProviderConfig`, `fetchLiveProviderModelRows`, `getCachedLiveProviderModelRows`, `fetchLiveProviderModelIds`, `LiveModelCatalogHttpError`, `clearLiveCatalogCacheForTests`, Modell-ID-Filterung, TTL-Cache und statischer Fallback |
    | `plugin-sdk/provider-catalog-runtime` | Laufzeit-Hook zur Erweiterung des Provider-Katalogs und Schnittstellen der Plugin-Provider-Registry für Vertragstests |
    | `plugin-sdk/provider-catalog-shared` | Seit Juli 2026 nur noch privat-lokal; `findCatalogTemplate`, `buildSingleProviderApiKeyCatalog`, `buildManifestModelProviderConfig`, `supportsNativeStreamingUsageCompat`, `applyProviderNativeStreamingUsageCompat` |
    | `plugin-sdk/provider-http` | Seit Juli 2026 nur noch privat-lokal; generische HTTP-/Endpunkt-Fähigkeitshilfen für Provider, Provider-HTTP-Fehler und Hilfsfunktionen für Multipart-Formulare zur Audiotranskription |
    | `plugin-sdk/provider-web-fetch-contract` | Seit Juli 2026 nur noch privat-lokal; eng gefasste Vertragshilfen für Webabruf-Konfiguration und -Auswahl wie `enablePluginInConfig` und `WebFetchProviderPlugin` |
    | `plugin-sdk/provider-web-fetch` | Seit Juli 2026 nur noch privat-lokal; Hilfsfunktionen für Registrierung und Cache von Webabruf-Providern |
    | `plugin-sdk/provider-web-search-config-contract` | Seit Juli 2026 nur noch privat-lokal; eng gefasste Konfigurations-/Anmeldedatenhilfen für Websuche-Provider, die keine Verknüpfung zur Plugin-Aktivierung benötigen |
    | `plugin-sdk/provider-web-search-contract` | Seit Juli 2026 nur noch privat-lokal; eng gefasste Vertragshilfen für Websuche-Konfiguration und -Anmeldedaten wie `createWebSearchProviderContractFields`, `enablePluginInConfig`, `resolveProviderWebSearchPluginConfig` sowie bereichsgebundene Setter/Getter für Anmeldedaten |
    | `plugin-sdk/provider-web-search` | Seit Juli 2026 nur noch privat-lokal; Hilfsfunktionen für Registrierung, Cache und Laufzeit von Websuche-Providern |
    | `plugin-sdk/embedding-providers` | Seit Juli 2026 nur noch privat-lokal; allgemeine Typen und Lesehilfen für Embedding-Provider, einschließlich `EmbeddingProviderAdapter`, `getEmbeddingProvider(...)` und `listEmbeddingProviders(...)`; Plugins registrieren Provider über `api.registerEmbeddingProvider(...)`, sodass die Manifest-Zuständigkeit durchgesetzt wird |
    | `plugin-sdk/provider-tools` | Seit Juli 2026 nur noch privat-lokal; `ProviderToolCompatFamily`, `buildProviderToolCompatFamilyHooks` sowie Schema-Bereinigung und Diagnose für DeepSeek/Gemini/OpenAI |
    | `plugin-sdk/provider-usage` | Seit Juli 2026 nur noch privat-lokal; Typen für Provider-Nutzungsübersichten, gemeinsam genutzte Hilfsfunktionen zum Abrufen der Nutzung und Provider-Abruffunktionen wie `fetchClaudeUsage` |
    | `plugin-sdk/provider-stream` | Seit Juli 2026 nur noch privat-lokal; `ProviderStreamFamily`, `buildProviderStreamFamilyHooks`, `composeProviderStreamWrappers`, Stream-Wrapper-Typen, Kompatibilität für Nur-Text-Tool-Aufrufe und gemeinsam genutzte Wrapper-Hilfsfunktionen für Anthropic/Google/Kilocode/MiniMax/Moonshot/OpenAI/OpenRouter/Z.AI |
    | `plugin-sdk/provider-stream-shared` | Seit Juli 2026 nur noch privat-lokal; öffentliche gemeinsam genutzte Provider-Stream-Wrapper-Hilfsfunktionen, einschließlich `composeProviderStreamWrappers`, `createOpenAICompatibleCompletionsThinkingOffWrapper`, `createPlainTextToolCallCompatWrapper`, `createPayloadPatchStreamWrapper`, `createToolStreamWrapper`, `normalizeOpenAICompatibleReasoningPayload`, `setQwenChatTemplateThinking` sowie Anthropic-/DeepSeek-/OpenAI-kompatible Stream-Dienstprogramme |
    | `plugin-sdk/provider-transport-runtime` | Seit Juli 2026 nur noch privat-lokal; native Provider-Transporthilfen wie abgesicherter Abruf, Textextraktion aus Tool-Ergebnissen, Transformationen von Transportnachrichten und beschreibbare Transportereignis-Streams |
    | `plugin-sdk/provider-onboard` | Seit Juli 2026 nur noch privat-lokal; Hilfsfunktionen zum Patchen der Onboarding-Konfiguration |
    | `plugin-sdk/global-singleton` | Seit Juli 2026 nur noch privat-lokal; prozesslokale Hilfsfunktionen für Singletons, Maps und Caches |
    | `plugin-sdk/group-activation` | Seit Juli 2026 nur noch privat-lokal; eng gefasste Hilfsfunktionen für den Gruppenaktivierungsmodus und das Befehlsparsing |
  </Accordion>

Provider-Nutzungsübersichten melden normalerweise ein oder mehrere Kontingent-`windows`, jeweils mit
einer Bezeichnung, dem verbrauchten Prozentsatz und einer optionalen Rücksetzzeit. Provider, die anstelle
zurücksetzbarer Kontingentfenster einen Kontostand oder Kontostatus-Text bereitstellen, sollten
`summary` mit einem leeren `windows`-Array zurückgeben, statt Prozentwerte zu erfinden.
OpenClaw zeigt diesen Zusammenfassungstext in der Statusausgabe an; verwenden Sie `error` nur, wenn der
Nutzungsendpunkt fehlgeschlagen ist oder keine verwendbaren Nutzungsdaten zurückgegeben hat.

  <Accordion title="Unterpfade für Authentifizierung und Sicherheit">
    | Unterpfad | Wichtige Exporte |
    | --- | --- |
    | `plugin-sdk/command-auth` | Veraltete breite Oberfläche zur Befehlsautorisierung (`resolveControlCommandGate`, Hilfsfunktionen der Befehls-Registry einschließlich dynamischer Formatierung von Argumentmenüs, Hilfsfunktionen zur Absenderautorisierung); verwenden Sie stattdessen Autorisierung am Kanaleingang bzw. in der Laufzeit oder Hilfsfunktionen für den Befehlsstatus |
    | `plugin-sdk/command-status` | Builder für Befehls-/Hilfenachrichten wie `buildCommandsMessagePaginated` und `buildHelpMessage` |
    | `plugin-sdk/approval-auth-runtime` | Hilfsfunktionen zur Auflösung von Genehmigenden und zur Aktionsautorisierung innerhalb desselben Chats |
    | `plugin-sdk/approval-client-runtime` | Hilfsfunktionen für native Ausführungsgenehmigungsprofile und -filter |
    | `plugin-sdk/approval-delivery-runtime` | Adapter für native Genehmigungsfähigkeiten und -zustellung |
    | `plugin-sdk/approval-gateway-runtime` | Gemeinsam genutzter Resolver für das Genehmigungs-Gateway |
    | `plugin-sdk/approval-reference-runtime` | Seit Juli 2026 nur noch privat-lokal; deterministische Hilfsfunktion für dauerhafte Locator-Werte bei transportbedingt eingeschränkten Genehmigungs-Callbacks |
    | `plugin-sdk/approval-handler-adapter-runtime` | Leichtgewichtige Hilfsfunktionen zum Laden nativer Genehmigungsadapter für häufig genutzte Kanaleinstiegspunkte |
    | `plugin-sdk/approval-handler-runtime` | Umfassendere Laufzeithilfen für Genehmigungs-Handler; bevorzugen Sie die enger gefassten Adapter-/Gateway-Schnittstellen, wenn diese ausreichen |
    | `plugin-sdk/approval-native-runtime` | Hilfsfunktionen für native Genehmigungsziele, Kontobindung, Routing-Gates, Weiterleitungs-Fallback und Unterdrückung lokaler nativer Ausführungsaufforderungen |
    | `plugin-sdk/approval-reaction-runtime` | Seit Juli 2026 nur noch privat-lokal; fest codierte Bindungen für Genehmigungsreaktionen, Nutzlasten für Reaktionsaufforderungen, Speicher für Reaktionsziele, Hilfsfunktionen für Reaktionshinweistexte und Kompatibilitätsexport zur Unterdrückung lokaler nativer Ausführungsaufforderungen |
    | `plugin-sdk/approval-reply-runtime` | Hilfsfunktionen für Antwortnutzlasten bei Ausführungs-/Plugin-Genehmigungen |
    | `plugin-sdk/approval-runtime` | Hilfsfunktionen für Nutzlasten bei Ausführungs-/Plugin-Genehmigungen, Builder für Genehmigungsfähigkeiten, Hilfsfunktionen für Genehmigungsautorisierung und -profile, Routing-/Laufzeithilfen für native Genehmigungen und Hilfsfunktionen für strukturierte Genehmigungsanzeigen wie `formatApprovalDisplayPath` |
    | `plugin-sdk/command-auth-native` | Native Befehlsautorisierung, dynamische Formatierung von Argumentmenüs und Hilfsfunktionen für native Sitzungsziele |
    | `plugin-sdk/command-detection` | Gemeinsam genutzte Hilfsfunktionen zur Befehlserkennung |
    | `plugin-sdk/command-primitives-runtime` | Leichtgewichtige Prädikate für Befehlstext in häufig genutzten Kanalpfaden |
    | `plugin-sdk/command-surface` | Seit Juli 2026 nur noch privat-lokal; Normalisierung von Befehlskörpern und Hilfsfunktionen für Befehlsoberflächen |
    | `plugin-sdk/allow-from` | `formatAllowFromLowercase` |
    | `plugin-sdk/provider-auth-login-flow-runtime` | Seit Juli 2026 nur noch privat-lokal; Hilfsfunktionen für verzögert geladene Provider-Authentifizierungsabläufe zur Gerätecode-Kopplung in privaten Kanälen und der Web UI |
    | `plugin-sdk/channel-secret-runtime` | Veraltete breite Oberfläche für Secret-Verträge (`collectSimpleChannelFieldAssignments`, `getChannelSurface`, `pushAssignment`, Secret-Zieltypen); bevorzugen Sie die nachfolgenden fokussierten Unterpfade |
    | `plugin-sdk/channel-secret-basic-runtime` | Eng gefasste Secret-Vertragsexporte und Builder für Ziel-Registrys für Secret-Oberflächen von Kanälen/Plugins außerhalb von TTS |
    | `plugin-sdk/channel-secret-tts-runtime` | Seit Juli 2026 nur noch privat-lokal; eng gefasste Hilfsfunktionen zur Zuweisung verschachtelter TTS-Secrets für Kanäle |
    | `plugin-sdk/secret-ref-runtime` | Eng gefasste Typisierung und Auflösung von SecretRef sowie Nachschlagen von Plan-Zielpfaden für das Parsing von Secret-Verträgen und Konfigurationen |
    | `plugin-sdk/security-runtime` | Veraltetes breites Barrel für Vertrauen, DM-Gating, auf das Stammverzeichnis begrenzte Datei-/Pfadhilfen einschließlich ausschließlich erstellender Schreibvorgänge, synchrone/asynchrone atomare Dateiersetzung, Schreiben temporärer Geschwisterdateien, Fallback für geräteübergreifendes Verschieben, Hilfsfunktionen für private Dateispeicher, Schutzvorrichtungen für Symlink-Elternpfade, externe Inhalte, Schwärzung sensibler Texte, Secret-Vergleich in konstanter Zeit und Hilfsfunktionen zur Secret-Erfassung; bevorzugen Sie fokussierte Unterpfade für Sicherheit/SSRF/Secrets |
    | `plugin-sdk/ssrf-policy` | Hilfsfunktionen für Host-Zulassungslisten und SSRF-Richtlinien für private Netzwerke |
    | `plugin-sdk/ssrf-dispatcher` | Seit Juli 2026 nur noch privat-lokal; eng gefasste Hilfsfunktionen für gebundene Dispatcher ohne die breite Infrastruktur-Laufzeitoberfläche |
    | `plugin-sdk/ssrf-runtime` | Hilfsfunktionen für gebundene Dispatcher, SSRF-abgesicherte Abrufe, SSRF-Fehler und SSRF-Richtlinien |
    | `plugin-sdk/secret-input` | Hilfsfunktionen zum Parsen von Secret-Eingaben |
    | `plugin-sdk/webhook-ingress` | Hilfsfunktionen für Webhook-Anfragen/-Ziele und Rohdaten-Konvertierung von WebSockets und Anfragekörpern |
    | `plugin-sdk/webhook-request-guards` | Hilfsfunktionen für Größenlimits und Zeitüberschreitungen von Anfragekörpern sowie `runDetachedWebhookWork` für nachverfolgte Verarbeitung nach der Bestätigung |
  </Accordion>

  <Accordion title="Runtime and storage subpaths">
    | Unterpfad | Wichtige Exporte |
    | --- | --- |
    | `plugin-sdk/runtime` | Hilfsfunktionen für Laufzeit/Protokollierung/Sicherung, Warnungen zu Plugin-Installationspfaden und Prozesshilfen |
    | `plugin-sdk/runtime-env` | Schlanke Hilfsfunktionen für Laufzeitumgebung, Logger, Zeitüberschreitung, Wiederholungsversuche und Backoff |
    | `plugin-sdk/browser-config` | Nach Juli 2026 nur noch privat-lokal; unterstützte Browserkonfigurationsfassade für normalisierte Profile/Standardwerte, CDP-URL-Parsing und Hilfsfunktionen zur Authentifizierung der Browsersteuerung |
    | `plugin-sdk/agent-harness-task-runtime` | Nach Juli 2026 nur noch privat-lokal; generische Hilfsfunktionen für Aufgabenlebenszyklus und Abschlusszustellung für Harness-gestützte Agenten mit einem vom Host ausgegebenen Aufgabenbereich |
    | `plugin-sdk/codex-mcp-projection` | Nach Juli 2026 nur noch privat-lokal; reservierte gebündelte Codex-Hilfsfunktion zur Übertragung der MCP-Serverkonfiguration des Benutzers in die Codex-Threadkonfiguration; nicht für Plugins von Drittanbietern |
    | `plugin-sdk/codex-native-task-runtime` | Repo-lokale gebündelte Codex-Hilfsfunktion für die native Aufgaben-Spiegelung/Laufzeitverdrahtung; kein Paketexport |
    | `plugin-sdk/channel-runtime-context` | Generische Hilfsfunktionen zur Registrierung und Suche des Laufzeitkontexts von Kanälen |
    | `plugin-sdk/matrix` | Veraltete Matrix-Kompatibilitätsfassade für ältere Kanalpakete von Drittanbietern; neue Plugins sollten `plugin-sdk/run-command` direkt importieren |
    | `plugin-sdk/runtime-store` | `createPluginRuntimeStore` |
    | `plugin-sdk/plugin-runtime` | Veralteter umfassender Barrel für Hilfsfunktionen zu Plugin-Befehlen, Hooks, HTTP und Interaktionen; bevorzugen Sie fokussierte Plugin-Laufzeitunterpfade |
    | `plugin-sdk/hook-runtime` | Veralteter umfassender Barrel für Hilfsfunktionen der Webhook-/internen Hook-Pipeline; bevorzugen Sie fokussierte Hook-/Plugin-Laufzeitunterpfade |
    | `plugin-sdk/lazy-runtime` | Hilfsfunktionen für verzögerten Laufzeitimport und verzögerte Bindung wie `createLazyRuntimeModule`, `createLazyRuntimeMethod` und `createLazyRuntimeSurface` |
    | `plugin-sdk/process-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen zur Prozessausführung |
    | `plugin-sdk/node-host` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen zur Auflösung von ausführbaren Node-Host-Dateien und zur PTY-Fortsetzung |
    | `plugin-sdk/cli-runtime` | Nach Juli 2026 nur noch privat-lokal; veralteter umfassender Barrel für CLI-Formatierung, Warten, Versionen, Argumentaufrufe und verzögert geladene Befehlsgruppen; bevorzugen Sie fokussierte CLI-/Laufzeitunterpfade |
    | `plugin-sdk/qa-runner-runtime` | Nach Juli 2026 nur noch privat-lokal; unterstützte Fassade, die Plugin-QA-Szenarien über die CLI-Befehlsoberfläche bereitstellt |
    | `plugin-sdk/tts-runtime` | Nach Juli 2026 nur noch privat-lokal; unterstützte Fassade für Text-zu-Sprache-Konfigurationsschemas und Laufzeithilfen |
    | `plugin-sdk/gateway-method-runtime` | Reservierte Gateway-Hilfsfunktion zur Methodendisposition für Plugin-HTTP-Routen, die `contracts.gatewayMethodDispatch: ["authenticated-request"]` deklarieren |
    | `plugin-sdk/gateway-runtime` | Gateway-Client, Hilfsfunktion zum Starten des Clients bei bereiter Ereignisschleife, Gateway-CLI-RPC, Gateway-Protokollfehler, Auflösung angekündigter LAN-Hosts und Hilfsfunktionen für Kanalstatus-Patches |
    | `plugin-sdk/config-contracts` | Fokussierte reine Typkonfigurationsoberfläche für Plugin-Konfigurationsformen wie `OpenClawConfig` und Kanal-/Provider-Konfigurationstypen |
    | `plugin-sdk/plugin-config-runtime` | Veraltete Kompatibilitätsfassade für Laufzeithilfen zur Plugin-Konfiguration; neue Plugins verwenden `api.pluginConfig` sowie fokussierte Konfigurationsverträge, Snapshots und Mutationshilfen |
    | `plugin-sdk/config-mutation` | Hilfsfunktionen für transaktionale Konfigurationsmutationen wie `mutateConfigFile`, `replaceConfigFile` und `logConfigUpdated` |
    | `plugin-sdk/message-tool-delivery-hints` | Nach Juli 2026 nur noch privat-lokal; gemeinsame Hinweiszeichenfolgen für Zustellungsmetadaten von Nachrichtenwerkzeugen |
    | `plugin-sdk/runtime-config-snapshot` | Hilfsfunktionen für Snapshots der aktuellen Prozesskonfiguration wie `getRuntimeConfig`, `getRuntimeConfigSnapshot` und Test-Snapshot-Setter |
    | `plugin-sdk/text-autolink-runtime` | Nach Juli 2026 nur noch privat-lokal; Erkennung von Autolinks für Dateiverweise ohne den umfassenden Text-Barrel |
    | `plugin-sdk/reply-runtime` | Gemeinsame Laufzeithilfen für eingehende Nachrichten/Antworten, Aufteilung, Disposition, Heartbeat und Antwortplanung |
    | `plugin-sdk/reply-dispatch-runtime` | Schlanke Hilfsfunktionen für Antwortdisposition/-abschluss und Konversationsbezeichnungen |
    | `plugin-sdk/reply-history` | Gemeinsame Hilfsfunktionen für den Antwortverlauf in einem kurzen Zeitfenster. Neuer Code für Nachrichtendurchläufe sollte `createChannelHistoryWindow` verwenden; untergeordnete Map-Hilfsfunktionen bleiben ausschließlich veraltete Kompatibilitätsexporte |
    | `plugin-sdk/reply-reference` | Nach Juli 2026 nur noch privat-lokal; `createReplyReferencePlanner` |
    | `plugin-sdk/reply-chunking` | Schlanke Hilfsfunktionen zur Aufteilung von Text/Markdown |
    | `plugin-sdk/session-store-runtime` | Hilfsfunktionen für Sitzungsabläufe (`getSessionEntry`, `listSessionEntries`, `patchSessionEntry`, `upsertSessionEntry`), Reparatur-/Lebenszyklushilfen (`deleteSessionEntry`, `cleanupSessionLifecycleArtifacts`, `resolveSessionStoreBackupPaths`), Marker-Hilfsfunktionen für vorübergehende `sessionFile`-Werte, begrenztes Lesen des Texts aktueller Benutzer-/Assistententranskripte anhand der Sitzungsidentität, Hilfsfunktionen für Sitzungsspeicherpfad/Sitzungsschlüssel und Lesen des Aktualisierungszeitpunkts, ohne umfassende Importe für Konfigurationsschreibvorgänge/-wartung |
    | `plugin-sdk/session-transcript-runtime` | Nach Juli 2026 nur noch privat-lokal; Transkriptidentität, begrenzte Rohdaten- und Sichtbarkeitscursor, bereichsgebundene Ziel-/Lese-/Schreibhilfen, Projektion sichtbarer Nachrichteneinträge, Veröffentlichung von Aktualisierungen, Schreibsperren und Trefferschlüssel für den Transkriptspeicher |
    | `plugin-sdk/sqlite-runtime` | Nach Juli 2026 nur noch privat-lokal; fokussierte Hilfsfunktionen für SQLite-Agentenschema, Pfade und Transaktionen für die Erstanbieterlaufzeit, ohne Steuerelemente für den Datenbanklebenszyklus |
    | `plugin-sdk/cron-store-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen für Pfad/Laden/Speichern des Cron-Speichers |
    | `plugin-sdk/state-paths` | Hilfsfunktionen für Pfade von Zustands-/OAuth-Verzeichnissen |
    | `plugin-sdk/plugin-state-runtime` | Nach Juli 2026 nur noch privat-lokal; Plugin-bereichsgebundene Verträge für Schlüsselzustände, BLOBs und kooperative SQLite-Leases sowie Verbindungs-Pragmas, verifizierte WAL-Wartung und atomare STRICT-Schemamigrationshilfen. Lease-Callbacks erhalten ein Abbruchsignal, und typisierte Fehler unterscheiden Zeitüberschreitung, Abbruch, verlorene Eigentümerschaft, ungültige Eingabe und Speicherfehler |
    | `plugin-sdk/routing` | Hilfsfunktionen für Routen-/Sitzungsschlüssel-/Kontobindungen wie `resolveAgentRoute`, `buildAgentSessionKey` und `resolveDefaultAgentBoundAccountId` |
    | `plugin-sdk/status-helpers` | Gemeinsame Hilfsfunktionen für Kanal-/Kontostatuszusammenfassungen, Standardwerte des Laufzeitzustands und Problemmetadaten |
    | `plugin-sdk/target-resolver-runtime` | Nach Juli 2026 nur noch privat-lokal; gemeinsame Hilfsfunktionen zur Zielauflösung |
    | `plugin-sdk/string-normalization-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen zur Normalisierung von Slugs/Zeichenfolgen |
    | `plugin-sdk/request-url` | Nach Juli 2026 nur noch privat-lokal; Extrahieren von URL-Zeichenfolgen aus Fetch-/Request-ähnlichen Eingaben |
    | `plugin-sdk/run-command` | Zeitgesteuerter Befehls-Runner mit normalisierten stdout-/stderr-Ergebnissen |
    | `plugin-sdk/param-readers` | Allgemeine Parameterleser für Werkzeuge/CLI |
    | `plugin-sdk/tool-plugin` | Definieren eines einfachen typisierten Agentenwerkzeug-Plugins und Bereitstellen statischer Metadaten für die Manifestgenerierung |
    | `plugin-sdk/tool-payload` | Nach Juli 2026 nur noch privat-lokal; Extrahieren normalisierter Nutzdaten aus Werkzeugergebnisobjekten |
    | `plugin-sdk/tool-send` | Extrahieren kanonischer Sendeziel-Felder aus Werkzeugargumenten |
    | `plugin-sdk/sandbox` | Nach Juli 2026 nur noch privat-lokal; Typen für Sandbox-Backends und SSH-/OpenShell-Befehlshilfen, einschließlich Vorabprüfung von Ausführungsbefehlen mit sofortigem Abbruch bei Fehlern |
    | `plugin-sdk/temp-path` | Gemeinsame Hilfsfunktionen für temporäre Downloadpfade und private sichere temporäre Arbeitsbereiche |
    | `plugin-sdk/logging-core` | Hilfsfunktionen für Subsystem-Logger und Schwärzung |
    | `plugin-sdk/markdown-table-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen für Markdown-Tabellenmodus und -konvertierung |
    | `plugin-sdk/model-session-runtime` | Hilfsfunktionen für Modell-/Sitzungsüberschreibungen wie `applyModelOverrideToSessionEntry` und `resolveAgentMaxConcurrent` |
    | `plugin-sdk/talk-config-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen zur Auflösung der Talk-Provider-Konfiguration |
    | `plugin-sdk/json-store` | Kleine Hilfsfunktionen zum Lesen/Schreiben von JSON-Zuständen |
    | `plugin-sdk/json-unsafe-integers` | Nach Juli 2026 nur noch privat-lokal; JSON-Parsing-Hilfsfunktionen, die unsichere Ganzzahlliterale als Zeichenfolgen beibehalten |
    | `plugin-sdk/file-lock` | Nach Juli 2026 nur noch privat-lokal; wiedereintrittsfähige Dateisperrhilfen sowie Doctor-sichere Rückgewinnung definitiv veralteter, unveränderter, ausgemusterter Sperr-Sidecars |
    | `plugin-sdk/persistent-dedupe` | Hilfsfunktionen für einen festplattenbasierten Deduplizierungscache |
    | `plugin-sdk/ingress-effect-once` | Dauerhafter Anspruchs-/Commit-Schutz für nicht idempotente Nebeneffekte beim Eingang |
    | `plugin-sdk/acp-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen für ACP-Laufzeit/-Sitzungen und Antwortdisposition |
    | `plugin-sdk/acp-runtime-backend` | Nach Juli 2026 nur noch privat-lokal; leichtgewichtige Hilfsfunktionen für ACP-Backendregistrierung und Antwortdisposition für beim Start geladene Plugins |
    | `plugin-sdk/acp-binding-resolve-runtime` | Nach Juli 2026 nur noch privat-lokal; schreibgeschützte ACP-Bindungsauflösung ohne Importe für den Lebenszyklusstart |
    | `plugin-sdk/agent-config-primitives` | Veraltete Primitive für Agentenlaufzeit-Konfigurationsschemas; importieren Sie Schemaprimitiven aus einer gepflegten, Plugin-eigenen Oberfläche |
    | `plugin-sdk/boolean-param` | Toleranter Leser für boolesche Parameter |
    | `plugin-sdk/dangerous-name-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen zur Auflösung von Übereinstimmungen gefährlicher Namen |
    | `plugin-sdk/device-bootstrap` | Hilfsfunktionen für Geräte-Bootstrap und Kopplungstoken, einschließlich `BOOTSTRAP_HANDOFF_OPERATOR_SCOPES` |
    | `plugin-sdk/extension-shared` | Gemeinsame Primitive für passive Kanäle, Status und umgebungsbezogene Proxy-Hilfen |
    | `plugin-sdk/models-provider-runtime` | Hilfsfunktionen für Befehls-/Provider-Antworten von `/models` |
    | `plugin-sdk/skill-commands-runtime` | Hilfsfunktionen zur Auflistung von Skill-Befehlen |
    | `plugin-sdk/native-command-registry` | Hilfsfunktionen für Registrierung, Aufbau und Serialisierung nativer Befehle |
    | `plugin-sdk/agent-harness` | Experimentelle Oberfläche für vertrauenswürdige Plugins zur Verwendung mit Low-Level-Agenten-Harnesses: Harness-Typen, Hilfsfunktionen zum Steuern/Abbrechen aktiver Ausführungen, Hilfsfunktionen für die OpenClaw-Werkzeugbrücke, Hilfsfunktionen für Werkzeugrichtlinien des Laufzeitplans, Klassifizierung terminaler Ergebnisse, Hilfsfunktionen für Formatierung/Details des Werkzeugfortschritts und Dienstprogramme für Versuchsergebnisse |
    | `plugin-sdk/async-lock-runtime` | Nach Juli 2026 nur noch privat-lokal; prozesslokale asynchrone Sperrhilfsfunktion für kleine Laufzeitzustandsdateien |
    | `plugin-sdk/channel-activity-runtime` | Nach Juli 2026 nur noch privat-lokal; Telemetriehilfsfunktion für Kanalaktivität |
    | `plugin-sdk/concurrency-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktion für begrenzte Nebenläufigkeit asynchroner Aufgaben |
    | `plugin-sdk/dedupe-runtime` | Hilfsfunktionen für speicherinterne und persistent gespeicherte Deduplizierungscaches |
    | `plugin-sdk/delivery-queue-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktion zum Abarbeiten ausstehender ausgehender Zustellungen |
    | `plugin-sdk/file-access-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen für sichere lokale Datei- und Medienquellenpfade |
    | `plugin-sdk/heartbeat-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen für Heartbeat-Aktivierung, -Ereignisse und -Sichtbarkeit |
    | `plugin-sdk/expect-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktion zur Prüfung erforderlicher Werte für beweisbare Laufzeitinvarianten |
    | `plugin-sdk/number-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktion zur numerischen Typumwandlung |
    | `plugin-sdk/secure-random-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen für sichere Token/UUIDs |
    | `plugin-sdk/system-event-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen für die Systemereigniswarteschlange |
    | `plugin-sdk/transport-ready-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktion zum Warten auf Transportbereitschaft |
    | `plugin-sdk/exec-approvals-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen für Richtliniendateien zur Ausführungsgenehmigung ohne den umfassenden Infrastruktur-Laufzeit-Barrel |
    | `plugin-sdk/infra-runtime` | Veralteter Kompatibilitäts-Shim; verwenden Sie die fokussierten Laufzeitunterpfade oben |
    | `plugin-sdk/collection-runtime` | Kleine Hilfsfunktionen für begrenzte Caches |
    | `plugin-sdk/diagnostic-runtime` | Hilfsfunktionen für Diagnosekennzeichen, Ereignisse und Ablaufverfolgungskontext |
    | `plugin-sdk/error-runtime` | Hilfsfunktionen für Fehlergraphen, Formatierung, Typumwandlung unbekannter Werte und gemeinsame Fehlerklassifizierung, `PlatformMessageNotDispatchedError`, `isApprovalNotFoundError` |
    | `plugin-sdk/fetch-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen für gekapseltes Fetch, Proxy, EnvHttpProxyAgent-Optionen und festgelegte Lookups |
    | `plugin-sdk/runtime-fetch` | Nach Juli 2026 nur noch privat-lokal; Dispatcher-fähiges Laufzeit-Fetch ohne Importe für Proxy/abgesichertes Fetch |
    | `plugin-sdk/inline-image-data-url-runtime` | Nach Juli 2026 nur noch privat-lokal; Hilfsfunktionen zur Bereinigung von Daten-URLs eingebetteter Bilder und zur Signaturerkennung ohne die umfassende Medienlaufzeitoberfläche |
    | `plugin-sdk/response-limit-runtime` | Nach Juli 2026 nur noch privat-lokal; durch Byte-Anzahl, Leerlauf und Frist begrenzte Leser für Antworttexte ohne die umfassende Medienlaufzeitoberfläche |
    | `plugin-sdk/session-binding-runtime` | Nach Juli 2026 nur noch privat-lokal; aktueller Konversationsbindungszustand ohne konfigurierte Bindungsweiterleitung oder Kopplungsspeicher |
    | `plugin-sdk/context-visibility-runtime` | Nach Juli 2026 nur noch privat-lokal; Auflösung der Kontextsichtbarkeit und Filterung ergänzender Kontexte ohne umfassende Konfigurations-/Sicherheitsimporte |
    | `plugin-sdk/string-coerce-runtime` | Schlanke primitive Hilfsfunktionen zur Typumwandlung und Normalisierung von Datensätzen/Zeichenfolgen ohne Markdown-/Protokollierungsimporte |
    | `plugin-sdk/html-entity-runtime` | Nach Juli 2026 nur noch privat-lokal; einmalige Dekodierung von mit Semikolon abgeschlossenen HTML5-Entitäten ohne umfassende Textdienstprogramme |
    | `plugin-sdk/text-utility-runtime` | Ab Juli 2026 nur noch privat-lokal; Low-Level-Hilfsfunktionen für Text und Pfade, einschließlich HTML-Escaping für fünf Entitäten |
    | `plugin-sdk/widget-html` | Erkennung vollständiger Dokumente, Größenvalidierung und Fehler bei Tool-Eingaben für eigenständige HTML-Widgets |
    | `plugin-sdk/host-runtime` | Ab Juli 2026 nur noch privat-lokal; Hilfsfunktionen zur Normalisierung von Hostnamen und SCP-Hosts |
    | `plugin-sdk/retry-runtime` | Ab Juli 2026 nur noch privat-lokal; Hilfsfunktionen für Wiederholungskonfiguration und Wiederholungsausführung |
    | `plugin-sdk/agent-runtime` | Veralteter umfassender Barrel-Export für Hilfsfunktionen zu Agentenverzeichnis, -identität und -arbeitsbereich, einschließlich `resolveAgentDir`, `resolveDefaultAgentDir` und des veralteten Kompatibilitätsexports `resolveOpenClawAgentDir`; bevorzugen Sie gezielte Agenten-/Runtime-Unterpfade |
    | `plugin-sdk/directory-runtime` | Konfigurationsgestützte Verzeichnisabfrage und Deduplizierung |
    | `plugin-sdk/keyed-async-queue` | Ab Juli 2026 nur noch privat-lokal; `KeyedAsyncQueue` |
  </Accordion>

  <Accordion title="Unterpfade für Funktionen und Tests">
    | Unterpfad | Wichtige Exporte |
    | --- | --- |
    | `plugin-sdk/media-runtime` | Veraltetes umfassendes Medien-Barrel einschließlich `saveRemoteMedia`, `saveResponseMedia`, `readRemoteMediaBuffer` und des veralteten `fetchRemoteMedia`; bevorzugen Sie `plugin-sdk/media-store`, `plugin-sdk/media-mime`, `plugin-sdk/outbound-media` und Unterpfade der Funktions-Runtime sowie Speicher-Hilfsfunktionen vor Pufferlesevorgängen, wenn eine URL zu OpenClaw-Medien werden soll |
    | `plugin-sdk/media-local-roots` | Fokussierte `getAgentScopedMediaLocalRoots(...)`- und richtlinienbewusste `getAgentScopedMediaLocalRootsForSources(...)`-Hilfsfunktionen für Plugin-eigene Lesevorgänge lokaler Medien |
    | `plugin-sdk/media-mime` | Eng gefasste MIME-Normalisierung, Zuordnung von Dateierweiterungen, MIME-Erkennung und Hilfsfunktionen für Medienarten |
    | `plugin-sdk/media-store` | Eng gefasste Medienspeicher-Hilfsfunktionen wie `saveMediaBuffer` und `saveMediaStream` |
    | `plugin-sdk/media-generation-runtime` | Nach Juli 2026 nur noch lokal-privat; gemeinsame Failover-Hilfsfunktionen für die Mediengenerierung, Kandidatenauswahl und Meldungen bei fehlendem Modell |
    | `plugin-sdk/media-understanding` | Veraltete Kompatibilitätsfassade für Provider-Typen und Hilfsfunktionen zur Medienerkennung; neue Provider registrieren sich über die injizierte Plugin-API und behalten Anfrage-Hilfsfunktionen im Besitz des Plugins |
    | `plugin-sdk/text-chunking` | Ausgehender Text und bereichsweise Segmentierung unter Beibehaltung von Offsets, Markdown-Segmentierung und Rendering-Hilfsfunktionen, zitatbewusste Tokenisierung von HTML-Tags, Konvertierung von Markdown-Tabellen, Entfernung von Direktiven-Tags und Hilfsfunktionen für sicheren Text |
    | `plugin-sdk/speech` | Nach Juli 2026 nur noch lokal-privat; Typen für Sprach-Provider sowie Exporte für Provider-seitige Direktiven, Registry, Validierung, einen OpenAI-kompatiblen TTS-Builder und Sprach-Hilfsfunktionen |
    | `plugin-sdk/speech-core` | Nach Juli 2026 nur noch lokal-privat; gemeinsame Typen für Sprach-Provider sowie Exporte für Registry, Direktiven, Normalisierung und Sprach-Hilfsfunktionen |
    | `plugin-sdk/speech-settings` | Leichtgewichtige Primitive zur Auflösung und Normalisierung der TTS-Konfiguration ohne Provider-Registrys oder Synthese-Runtime |
    | `plugin-sdk/realtime-transcription` | Nach Juli 2026 nur noch lokal-privat; Provider-Typen für Echtzeittranskription, Registry-Hilfsfunktionen und gemeinsame Hilfsfunktion für WebSocket-Sitzungen |
    | `plugin-sdk/realtime-bootstrap-context` | Nach Juli 2026 nur noch lokal-privat; Hilfsfunktion zur Initialisierung von Echtzeitprofilen für die begrenzte Kontextinjektion von `IDENTITY.md`, `USER.md` und `SOUL.md` |
    | `plugin-sdk/realtime-voice` | Nach Juli 2026 nur noch lokal-privat; Provider-Typen für Echtzeitsprache, Registry-Hilfsfunktionen, gemeinsame Schwellenlogik für Audioenergie und Sprachbeginn sowie Hilfsfunktionen für das Echtzeitsprachverhalten, einschließlich des transportunabhängigen Sitzungsharnesses und der Nachverfolgung von Ausgabeaktivitäten |
    | `plugin-sdk/meeting-runtime` | Sitzungs-Runtime für Browserbesprechungen, Echtzeit-Audio-Engines und -Transporte, `MeetingPlatformAdapter`, Browser-/Node-Steuerung, Agent-Konsultation, Delegierung von Sprachanrufen, Einrichtungsprüfungen und Hilfsfunktionen für SoX-Befehle |
    | `plugin-sdk/image-generation` | Nach Juli 2026 nur noch lokal-privat; Provider-Typen für Bildgenerierung sowie Hilfsfunktionen für Bildressourcen und Daten-URLs und der OpenAI-kompatible Bild-Provider-Builder |
    | `plugin-sdk/image-generation-core` | Nach Juli 2026 nur noch lokal-privat; gemeinsame Typen und Hilfsfunktionen für Failover, Authentifizierung und Registry der Bildgenerierung |
    | `plugin-sdk/music-generation` | Nach Juli 2026 nur noch lokal-privat; Provider-, Anfrage- und Ergebnistypen für Musikgenerierung |
    | `plugin-sdk/video-generation` | Nach Juli 2026 nur noch lokal-privat; Provider-, Anfrage- und Ergebnistypen für Videogenerierung |
    | `plugin-sdk/video-generation-core` | Nach Juli 2026 nur noch lokal-privat; gemeinsame Typen für Videogenerierung, Failover-Hilfsfunktionen, Provider-Suche und Analyse von Modellreferenzen |
    | `plugin-sdk/transcripts` | Nach Juli 2026 nur noch lokal-privat; gemeinsame Provider-Typen für Transkriptquellen, Registry-Hilfsfunktionen, Bridge-Factory für Besprechungs-Provider, Sitzungsdeskriptoren und Äußerungsmetadaten |
    | `plugin-sdk/webhook-targets` | Nach Juli 2026 nur noch lokal-privat; Webhook-Ziel-Registry und Hilfsfunktionen zur Routeninstallation |
    | `plugin-sdk/web-media` | Gemeinsame Hilfsfunktionen zum Laden entfernter/lokaler Medien |
    | `plugin-sdk/zod` | Veralteter Kompatibilitäts-Reexport; importieren Sie `zod` direkt aus `zod` |
    | `plugin-sdk/plugin-test-api` | Repo-lokale minimale `createTestPluginApi`-Hilfsfunktion für Unit-Tests der direkten Plugin-Registrierung, ohne Brücken zu Repo-Testhilfsfunktionen zu importieren |
    | `plugin-sdk/agent-runtime-test-contracts` | Repo-lokale Fixtures für den nativen Agent-Runtime-Adaptervertrag für Tests von Authentifizierung, Zustellung, Fallback, Tool-Hooks, Prompt-Overlays, Schemas und Transkriptprojektionen |
    | `plugin-sdk/channel-test-helpers` | Repo-lokale, kanalorientierte Testhilfsfunktionen für generische Aktions-, Einrichtungs- und Statusverträge, Verzeichniszusicherungen, den Startlebenszyklus von Konten, die Weitergabe der Sendekonfiguration, Runtime-Mocks, Statusprobleme, ausgehende Zustellung und Hook-Registrierung |
    | `plugin-sdk/channel-target-testing` | Repo-lokale gemeinsame Suite für Fehlerfälle bei der Zielauflösung in Kanaltests |
    | `plugin-sdk/channel-contract-testing` | Repo-lokale, eng gefasste Testhilfsfunktionen für Kanalverträge ohne das umfassende Test-Barrel |
    | `plugin-sdk/plugin-test-contracts` | Repo-lokale Hilfsfunktionen für Plugin-Pakete, Registrierung, öffentliche Artefakte, direkte Importe, Runtime-API und Verträge zu Import-Nebeneffekten |
    | `plugin-sdk/plugin-state-test-runtime` | Repo-lokale Testhilfsfunktionen für Plugin-Zustandsspeicher, Eingangs-Queue und Zustandsdatenbank |
    | `plugin-sdk/provider-test-contracts` | Repo-lokale Hilfsfunktionen für Provider-Runtime, Authentifizierung, Erkennung, Onboarding, Katalog, Assistenten, Medienfunktionen, Wiedergaberichtlinien, Echtzeit-STT mit Live-Audio, Websuche/-abruf und Streaming-Verträge |
    | `plugin-sdk/provider-http-test-mocks` | Nach Juli 2026 nur noch lokal-privat; repo-lokale, optional aktivierbare Vitest-HTTP-/Authentifizierungs-Mocks für Provider-Tests, die `plugin-sdk/provider-http` ausführen |
    | `plugin-sdk/reply-payload-testing` | Repo-lokale Hilfsfunktionen zum Anhängen von Metadaten an Fixtures für Antwort-Payloads |
    | `plugin-sdk/sqlite-runtime-testing` | Repo-lokale SQLite-Lebenszyklus-Hilfsfunktionen für Erstanbieter-Tests |
    | `plugin-sdk/test-fixtures` | Repo-lokale Fixtures für generische Erfassung der CLI-Runtime, Sandbox-Kontext, Skill-Writer, Agent-Nachrichten, Systemereignisse, Modulneuladung, Pfade gebündelter Plugins, Terminaltext, Segmentierung, Authentifizierungstoken und typisierte Fälle |
    | `plugin-sdk/test-node-mocks` | Repo-lokale fokussierte Mock-Hilfsfunktionen für integrierte Node-Module zur Verwendung innerhalb von Vitest-`vi.mock("node:*")`-Factories |
  </Accordion>

  <Accordion title="Speicher-Unterpfade">
    | Unterpfad | Wichtige Exporte |
    | --- | --- |
    | `plugin-sdk/memory-core-host-embedding-registry` | Nach Juli 2026 nur noch lokal-privat; leichtgewichtige Registry-Hilfsfunktionen für Provider von Speicher-Embeddings |
    | `plugin-sdk/memory-core-host-engine-foundation` | Engine-Exporte der Speicher-Host-Grundlage |
    | `plugin-sdk/memory-core-host-engine-embeddings` | Nach Juli 2026 nur noch lokal-privat; Embedding-Verträge des Speicher-Hosts, Registry-Zugriff, lokaler Provider und generische Batch-/Remote-Hilfsfunktionen. `registerMemoryEmbeddingProvider` ist auf dieser Oberfläche veraltet; verwenden Sie für neue Provider die generische Embedding-Provider-API. |
    | `plugin-sdk/memory-core-host-engine-qmd` | Nach Juli 2026 nur noch lokal-privat; QMD-Engine-Exporte des Speicher-Hosts |
    | `plugin-sdk/memory-core-host-engine-storage` | Nach Juli 2026 nur noch lokal-privat; Speicher-Engine-Exporte des Speicher-Hosts |
    | `plugin-sdk/memory-core-host-secret` | Nach Juli 2026 nur noch lokal-privat; Hilfsfunktionen für Geheimnisse des Speicher-Hosts |
    | `plugin-sdk/memory-core-host-status` | Nach Juli 2026 nur noch lokal-privat; Status-Hilfsfunktionen des Speicher-Hosts |
    | `plugin-sdk/memory-core-host-runtime-cli` | Nach Juli 2026 nur noch lokal-privat; CLI-Runtime-Hilfsfunktionen des Speicher-Hosts |
    | `plugin-sdk/memory-core-host-runtime-core` | Nach Juli 2026 nur noch lokal-privat; Kern-Runtime-Hilfsfunktionen des Speicher-Hosts |
    | `plugin-sdk/memory-core-host-runtime-files` | Nach Juli 2026 nur noch lokal-privat; Datei-/Runtime-Hilfsfunktionen des Speicher-Hosts |
    | `plugin-sdk/memory-host-core` | Veraltete Kompatibilitätsfassade für herstellerneutrale Hilfsfunktionen des Speicher-Hosts. Neue Speicher-Plugins verwenden injizierte Speicherfunktionen und vom Host vorbereitete Prompts; Begleit-Plugins verwenden weiterhin die beibehaltene Fassade zur Erkennung öffentlicher Artefakte, bis eine fokussierte Leseschnittstelle verfügbar ist. |
    | `plugin-sdk/memory-host-events` | Nach Juli 2026 nur noch lokal-privat; herstellerneutraler Alias für Ereignisjournal-Hilfsfunktionen des Speicher-Hosts |
    | `plugin-sdk/memory-host-markdown` | Nach Juli 2026 nur noch lokal-privat; gemeinsame Hilfsfunktionen für verwaltetes Markdown in Speicher-nahen Plugins |
    | `plugin-sdk/memory-host-search` | Nach Juli 2026 nur noch lokal-privat; Active-Memory-Runtime-Fassade für den Zugriff auf den Suchmanager |
  </Accordion>

  <Accordion title="Reservierte Unterpfade für gebündelte Hilfsfunktionen">
    Reservierte SDK-Unterpfade für gebündelte Hilfsfunktionen sind eng gefasste, eigentümerspezifische Oberflächen für
    gebündelten Plugin-Code. Sie werden im SDK-Inventar erfasst, damit Paket-
    Builds und Aliasing deterministisch bleiben, sind jedoch keine allgemeinen APIs
    zur Plugin-Entwicklung. Neue wiederverwendbare Host-Verträge sollten generische SDK-Unterpfade
    wie `plugin-sdk/gateway-runtime` und `plugin-sdk/ssrf-runtime` verwenden.

    | Unterpfad | Eigentümer und Zweck |
    | --- | --- |
    | `plugin-sdk/codex-mcp-projection` | Nach Juli 2026 nur noch lokal-privat; Hilfsfunktion des gebündelten Codex-Plugins zur Projektion der Benutzerkonfiguration von MCP-Servern in die Thread-Konfiguration des Codex-App-Servers (reservierter Paketexport) |
    | `plugin-sdk/codex-native-task-runtime` | Hilfsfunktion des gebündelten Codex-Plugins zur Spiegelung nativer Subagenten des Codex-App-Servers in den OpenClaw-Aufgabenstatus (nur repo-lokal, kein Paketexport) |

  </Accordion>
</AccordionGroup>

## Verwandte Themen

- [Übersicht über das Plugin SDK](/de/plugins/sdk-overview)
- [Einrichtung des Plugin SDK](/de/plugins/sdk-setup)
- [Plugins erstellen](/de/plugins/building-plugins)
