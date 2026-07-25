---
read_when:
    - Sie sehen die Warnung OPENCLAW_PLUGIN_SDK_COMPAT_DEPRECATED
    - Sie sehen die Warnung OPENCLAW_EXTENSION_API_DEPRECATED
    - Sie haben api.registerEmbeddedExtensionFactory vor OpenClaw 2026.4.25 verwendet
    - Sie aktualisieren ein Plugin auf die moderne Plugin-Architektur
    - Sie pflegen ein externes OpenClaw-Plugin
sidebarTitle: Migrate to SDK
summary: Von der veralteten Abwärtskompatibilitätsschicht zum modernen Plugin-SDK migrieren
title: Plugin-SDK-Migration
x-i18n:
    generated_at: "2026-07-24T20:35:17Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: a483f9c0f8409505fc2688872995382944e002520ceb651214dbc5ad8e3554fb
    source_path: plugins/sdk-migration.md
    workflow: 16
---

OpenClaw hat eine umfassende Abwärtskompatibilitätsschicht durch eine moderne Plugin-
Architektur ersetzt, die aus kleinen, gezielten Importen besteht. Wenn Ihr Plugin vor dieser
Änderung entstanden ist, bringt dieser Leitfaden es auf die aktuellen Verträge.

## Was sich geändert hat

Mehrere weitreichende Importoberflächen ermöglichten Plugins früher den Zugriff auf nahezu alles
über einen einzigen Einstiegspunkt:

- **`openclaw/plugin-sdk`** und **`openclaw/plugin-sdk/compat`** – re-exportierten
  während der Entwicklung des gezielten SDK Dutzende Hilfsfunktionen. Beide Wurzelpfade wurden
  inzwischen entfernt; importieren Sie stattdessen einen dokumentierten Unterpfad.
- **`openclaw/plugin-sdk/infra-runtime`** – ein umfangreiches Barrel, das Systemereignisse,
  Heartbeat-Status, Zustellungswarteschlangen, Fetch-/Proxy-Hilfsfunktionen, Dateihilfen,
  Genehmigungstypen und nicht zusammengehörige Dienstprogramme vermischte.
- **`openclaw/plugin-sdk/config-runtime`** – ein umfangreiches Konfigurations-Barrel, das
  nur für sein späteres Kompatibilitätsfenster beibehalten wurde; direkte Hilfsfunktionen zum
  Laden und Schreiben während der Laufzeit wurden entfernt.
- **`openclaw/extension-api`** – eine entfernte Brücke, die Plugins direkten
  Zugriff auf hostseitige Hilfsfunktionen wie den eingebetteten Agent-Runner gewährte.
- **`api.registerEmbeddedExtensionFactory(...)`** – ein entfernter, ausschließlich für den
  eingebetteten Runner bestimmter Hook, der Ereignisse des eingebetteten Runners wie
  `tool_result` beobachtete. Verwenden Sie stattdessen Middleware für Agent-
  Werkzeugergebnisse (siehe [Erweiterungen für eingebettete Werkzeugergebnisse auf
  Middleware migrieren](#how-to-migrate)).

Das Stamm-SDK, das Kompatibilitäts-Barrel, die Erweiterungsbrücke und die Factory für
eingebettete Erweiterungen wurden entfernt. `infra-runtime` und `config-runtime` bleiben
nur für ihre separat erfassten späteren Zeitfenster erhalten; neue Plugins sollten gezielte
Unterpfade verwenden.

<Warning>
  Plugins, die die entfernten Stamm-, Kompatibilitäts- oder Erweiterungsoberflächen
  importieren, werden nicht mehr geladen. Befolgen Sie vor dem Upgrade die nachstehenden
  Zuordnungen.
</Warning>

OpenClaw entfernt oder interpretiert dokumentiertes Plugin-Verhalten nicht in derselben
Änderung neu, mit der ein Ersatz eingeführt wird. Inkompatible Vertragsänderungen durchlaufen
zunächst einen Kompatibilitätsadapter, Diagnosen, Dokumentation und ein Veraltungsfenster. Dies
gilt für SDK-Importe, Manifestfelder, Einrichtungs-APIs, Hooks und das Registrierungsverhalten
während der Laufzeit.

### Warum

- **Langsamer Start** – der Import einer Hilfsfunktion lud Dutzende nicht
  zusammengehörige Module.
- **Zirkuläre Abhängigkeiten** – umfangreiche Re-Exporte erleichterten das
  Erzeugen von Importzyklen.
- **Unklare API-Oberfläche** – stabile Exporte konnten nicht von internen
  Exporten unterschieden werden.

Jedes `openclaw/plugin-sdk/<subpath>` ist nun ein kleines, eigenständiges Modul mit
einem dokumentierten Vertrag.

Alte Provider-Komfortschnittstellen für gebündelte Kanäle wurden ebenfalls entfernt –
kanalspezifische Hilfsabkürzungen waren private Annehmlichkeiten des Monorepos und keine
stabilen Plugin-Verträge. Verwenden Sie stattdessen schmale, generische SDK-Unterpfade. Behalten
Sie im Arbeitsbereich des gebündelten Plugins Provider-eigene Hilfsfunktionen im eigenen
`api.ts` oder `runtime-api.ts` dieses Plugins:

- Anthropic behält Claude-spezifische Stream-Hilfsfunktionen in seiner eigenen
  `api.ts`- / `contract-api.ts`-Schnittstelle.
- OpenAI behält Provider-Builder, Hilfsfunktionen für Standardmodelle und
  Echtzeit-Provider-Builder im eigenen `api.ts`.
- OpenRouter behält den Provider-Builder sowie Hilfsfunktionen für Onboarding
  und Konfiguration im eigenen `api.ts`.

## Kompatibilitätsrichtlinie

Kompatibilitätsarbeiten für externe Plugins erfolgen in dieser Reihenfolge:

1. Fügen Sie den neuen Vertrag hinzu.
2. Erhalten Sie das alte Verhalten über einen Kompatibilitätsadapter.
3. Geben Sie eine Diagnose oder Warnung aus, die den alten Pfad und seinen Ersatz nennt.
4. Decken Sie beide Pfade durch Tests ab.
5. Dokumentieren Sie die Veraltung und den Migrationspfad.
6. Entfernen Sie den alten Pfad erst nach dem angekündigten Migrationsfenster,
   üblicherweise in einer Hauptversion.

Wenn ein Manifestfeld weiterhin akzeptiert wird, verwenden Sie es, bis Dokumentation und
Diagnosen etwas anderes angeben. Neuer Code sollte den dokumentierten Ersatz bevorzugen;
bestehende Plugins dürfen bei gewöhnlichen Nebenversionen nicht ausfallen.

### Kompatibilität der Einrichtung veröffentlichter Kanäle

Über `2026.7.1` veröffentlichte Pakete für Slack, Discord, Signal und Microsoft Teams
importieren kanalspezifische Konfigurationsschemas aus `openclaw/plugin-sdk/bundled-channel-config-schema`. Die veröffentlichten
Pakete für Slack und Discord importieren außerdem `createLegacyCompatChannelDmPolicy` und
`promptLegacyChannelAllowFromForAccount` aus
`openclaw/plugin-sdk/setup-runtime`.

Diese Exporte bleiben als veraltete Laufzeit-Kompatibilitätsadapter verfügbar. Neue und erneut
veröffentlichte Plugins sollten ihre Konfigurationsschemas und Einrichtungsrichtlinien lokal
verwalten und dafür generische Primitive aus `channel-config-schema` und
`setup-runtime` verwenden. Die Kompatibilitätsexporte können erst entfernt werden, wenn sie
von den mindestens unterstützten Versionen der veröffentlichten Pakete nicht mehr importiert
werden.

### Kompatibilität der Eingabefelder für die Kanaleinrichtung

`ChannelSetupInput` hält nun nur noch den kanalübergreifenden Einrichtungsumschlag dauerhaft
typisiert. Kanalspezifische Felder bleiben in einer veralteten Kompatibilitätsstufe typisiert,
damit vorhandene externe Plugins weiterhin kompiliert werden, während Plugin-Autoren diese
Felder in Plugin-lokale Eingabetypen für die Einrichtung verschieben.

OpenClaw veröffentlicht keine Hauptversionen. Eine Registry-Prüfung am 2026-07-22 untersuchte
426 veröffentlichte, außerhalb des Quellbaums verwaltete Kanal-Plugins und entfernte 21 Felder
ohne Leser. Für jedes der 22 beibehaltenen Felder ist ein veröffentlichter Leser bekannt. Jedes
weitere Feld wird gelöscht, sobald es von keinem veröffentlichten Plugin mehr gelesen wird; die
beibehaltene Menge schrumpft, während Plugin-Autoren zu Plugin-lokalen Eingabetypen für die
Einrichtung migrieren.

Dieselbe Prüfung entfernte 23 alte, nicht deklarierte Adapter-Promotion-Schlüssel ohne
veröffentlichte Abhängige. Sechs häufig verwendete Schlüssel und der ausschließlich für die
Einrichtung bestimmte Schlüssel `rooms` bleiben erhalten. Auch diese Menge schrumpft,
während veröffentlichte Plugins `singleAccountKeysToMove` deklarieren.

Der gemeinsame Typ hat keine Indexsignatur. Plugin-eigene Schlüssel können weiterhin in
Laufzeit-Eingabeobjekten enthalten sein; deklarieren Sie sie in einer Plugin-lokalen
Schnittmenge oder grenzen Sie sie über das Einrichtungsschema des zuständigen Plugins ein.

| `code`                                  | `owner`   | `replacement`                                                                                    | Bedingung für die Entfernung                                          |
| --------------------------------------- | --------- | ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| `plugin-sdk-channel-setup-input-fields` | `channel` | Bilden Sie eine Schnittmenge aus `ChannelSetupInput` und einem Plugin-lokalen Typ, der die Felder des zuständigen Kanals deklariert | Löschen Sie ein Feld, wenn die Registry-Prüfung veröffentlichter Plugins keinen Leser findet |

Für die alte Promotion-Stufe nicht deklarierter Adapter gilt dieselbe lesergesteuerte
Richtlinie. Deklarieren Sie `singleAccountKeysToMove`, einschließlich eines leeren Arrays, wenn das
Plugin keine zusätzlichen Promotion-Schlüssel benötigt, damit der gemeinsame Fallback
schlüsselweise außer Betrieb genommen werden kann.

#### Leser überprüfen

1. Durchlaufen Sie `https://clawhub.ai/api/v1/packages?family=code-plugin&limit=100` seitenweise mit jedem `nextCursor`, und behalten Sie Pakete bei, deren `categories` `channels` enthält.
2. Fügen Sie npm-Kandidaten aus `npm search --json --searchlimit=1000 "openclaw channel plugin"` hinzu. Fügen Sie ausschließlich im Quellcode vorhandene Kandidaten aus GitHub-Codesuchen nach `openclaw/plugin-sdk/channel-setup`, `openclaw/plugin-sdk/setup` und `openclaw/plugin-sdk/core` hinzu.
3. Ermitteln Sie für jeden Kandidaten die neueste veröffentlichte Version. Führen Sie `npm pack <package>@<version> --json --pack-destination <temp-dir>` aus, entpacken Sie das Ergebnis und untersuchen Sie das ausgelieferte `dist`-JavaScript sowie die Deklarationen auf direkte oder destrukturierte Feldzugriffe. Laden Sie das ClawHub-Artefakt herunter, wenn für ein Paket keine npm-Veröffentlichung vorhanden ist.
4. Erfassen Sie Paket, Version, Feld oder Promotion-Schlüssel und die übereinstimmende Datei. Ein Feld oder Schlüssel darf nur gelöscht werden, wenn es von keinem veröffentlichten Plugin-Artefakt gelesen wird. Halten Sie die Lesernamen in den Codekommentaren neben den Listen der beibehaltenen Felder und Schlüssel mit der Prüfung synchron.

Dies ist ausschließlich ein Kompatibilitätsdatensatz für Quellcode und Typen. Er enthält keinen
Laufzeitadapter oder Eintrag in der Kompatibilitäts-Registry, da Laufzeit-Eingabeobjekte und
Verhalten der Einrichtung unverändert bleiben.

Prüfen Sie die aktuelle Migrationswarteschlange mit `pnpm plugins:boundary-report`:

| Flag                                                    | Wirkung                                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `--summary` (oder `pnpm plugins:boundary-report:summary`) | Kompakte Anzahlen anstelle vollständiger Details.                               |
| `--json`                                                | Maschinenlesbarer Bericht.                                                      |
| `--owner <id>`                                          | Auf ein Plugin oder einen Kompatibilitätsverantwortlichen filtern.              |
| `--fail-on-cross-owner`                                 | Bei reservierten SDK-Importen über Verantwortlichkeitsgrenzen hinweg mit einem Exit-Code ungleich null beenden. |
| `--fail-on-eligible-compat`                             | Mit einem Exit-Code ungleich null beenden, wenn das Datum `removeAfter` eines veralteten Kompatibilitätsdatensatzes überschritten wurde. |
| `--fail-on-unclassified-unused-reserved`                | Bei ungenutzten reservierten SDK-Shims mit einem Exit-Code ungleich null beenden. |

`pnpm plugins:boundary-report:ci` wird mit allen drei Fehler-Flags ausgeführt. Veraltete
Datensätze haben normalerweise ein explizites Datum `removeAfter` statt einer vagen Angabe
wie „nächste Hauptversion“. Bei einem Datensatz, dessen Verantwortlicher kein Datum genehmigt
hat, fehlt `removeAfter`; er erscheint als `no-date` und kann nie entfernt werden.
Der Bericht gruppiert veraltete Datensätze nach Datum, zählt lokale Code- und
Dokumentationsreferenzen, zeigt reservierte SDK-Importe über Verantwortlichkeitsgrenzen hinweg
an und fasst die private SDK-Brücke des Memory-Hosts zusammen. Für reservierte SDK-Unterpfade
muss die Nutzung durch den Verantwortlichen nachverfolgt werden; ungenutzte reservierte Exporte
sollten aus dem öffentlichen SDK entfernt werden.

### Alte Medienprojektion

Der Kompatibilitätsdatensatz `media-legacy-projection` umfasst die alten parallelen
Medienfelder, Payload-Builder, Metadaten-Aliasse für Hooks und Namen von Medienvorlagen. Das
genehmigte Datum `removeAfter` ist **2026-10-01** (zwei Veröffentlichungszyklen nach
Auslieferung der Facts-first-Ersetzungen). Die Entfernung erfordert zu diesem Zeitpunkt
zusätzlich eine saubere Prüfung veröffentlichter Plugin-Artefakte; migrieren Sie vor diesem
Datum.

Ersetzen Sie beim Kanaleingang die Singular-/Pluralfelder `MediaPath`,
`MediaUrl`, `MediaType`, `MediaPaths`, `MediaUrls`,
`MediaTypes`, `MediaTranscribedIndexes`, `MediaWorkspaceDir` und `MediaStaged` durch
geordnete Fakten:

```ts
import { toInboundMediaFacts } from "openclaw/plugin-sdk/channel-inbound";

const media = toInboundMediaFacts([
  { path: saved.path, url: nativeUrl, contentType: saved.contentType, messageId },
]);

const ctx = finalizeInboundContext({ Body: caption, media });
```

Verwenden Sie `event.media` in den Hooks `inbound_claim` und
`message_received`. Wenn entfernte Medien nicht lokal bereitgestellt wurden, verwenden Sie
`event.originalMedia` für Identität und Diagnosen und warten Sie auf `event.media`;
`event.mediaStagingPending` kennzeichnet diesen Zustand. Lesen Sie die veralteten Singular-/
Plural-Eigenschaften nicht aus `event.metadata`.

Ersetzen Sie für CLI-Medienmodelle `{{MediaPath}}`, `{{MediaUrl}}`,
`{{MediaType}}` und `{{MediaDir}}` durch `{{AttachmentPath}}`,
`{{AttachmentUrl}}`, `{{AttachmentContentType}}` und `{{AttachmentDir}}`. Verwenden Sie
`{{AttachmentIndex}}`, wenn die Position des Anhangs relevant ist.

Importieren Sie für die Richtlinie zum Lesen lokaler Medien `getAgentScopedMediaLocalRoots(...)` oder
`getAgentScopedMediaLocalRootsForSources(...)` aus
`openclaw/plugin-sdk/media-local-roots`. Die
`openclaw/plugin-sdk/agent-media-payload`-Fassade und ihre
`buildAgentMediaPayload(...)`-Projektion sind veraltet.

## So migrieren Sie

<Steps>
  <Step title="Hilfsfunktionen zum Laden und Schreiben der Laufzeitkonfiguration migrieren">
    Gebündelte Plugins sollten `api.runtime.config.loadConfig()` und
    `api.runtime.config.writeConfigFile(...)` nicht mehr direkt aufrufen. Bevorzugen Sie die
    Konfiguration, die bereits an den aktiven Aufrufpfad übergeben wurde. Langlebige Handler,
    die den aktuellen Prozess-Snapshot benötigen, können `api.runtime.config.current()` verwenden.
    Langlebige Agent-Werkzeuge sollten `ctx.getRuntimeConfig()` innerhalb von
    `execute` lesen, damit ein vor einem Konfigurationsschreibvorgang erstelltes
    Werkzeug weiterhin die aktualisierte Konfiguration sieht.

    Konfigurationsschreibvorgänge erfolgen über die transaktionale Hilfsfunktion mit einer
    expliziten Richtlinie für die Zeit nach dem Schreiben:

    ```typescript
    await api.runtime.config.mutateConfigFile({
      afterWrite: { mode: "auto" },
      mutate(draft) {
        draft.plugins ??= {};
      },
    });
    ```

    Verwenden Sie `afterWrite: { mode: "restart", reason: "..." }`, wenn die Änderung
    einen sauberen Gateway-Neustart erfordert, und `afterWrite: { mode: "none", reason: "..." }`
    nur, wenn der Aufrufer für die Folgemaßnahme verantwortlich ist und den
    Neuplaner bewusst unterdrückt. Mutationsergebnisse enthalten eine typisierte `followUp`-Zusammenfassung für
    Tests und Protokollierung; das Gateway bleibt dafür verantwortlich, den
    Neustart durchzuführen oder zu planen.

    `loadConfig` und `writeConfigFile` wurden aus der Plugin-
    Laufzeit entfernt. Gebündelte Plugins und Laufzeitcode im Repository werden durch
    `pnpm check:deprecated-api-usage` und
    `pnpm check:no-runtime-action-load-config` geschützt: Eine neue Verwendung in
    Produktions-Plugins schlägt unmittelbar fehl, direkte Konfigurationsschreibvorgänge schlagen fehl, Gateway-Servermethoden müssen
    den Laufzeit-Snapshot der Anfrage verwenden, Laufzeithelfer für Kanalversand, Aktionen und Clients
    müssen die Konfiguration von ihrer Grenze erhalten, und langlebige Laufzeitmodule
    erlauben keine umgebungsbezogenen `loadConfig()`-Aufrufe.

    Neuer Plugin-Code sollte das breite `openclaw/plugin-sdk/config-runtime`-
    Barrel vermeiden. Verwenden Sie den schmalen Unterpfad für die jeweilige Aufgabe:

    | Bedarf | Import |
    | --- | --- |
    | Konfigurationstypen wie `OpenClawConfig` | `openclaw/plugin-sdk/config-contracts` |
    | Konfigurationssuche am Plugin-Einstiegspunkt | `api.pluginConfig` |
    | Zusammenführen von Konfigurationen | Plugin-lokale Logik an der Konfigurationsgrenze |
    | Lesen des aktuellen Laufzeit-Snapshots | `openclaw/plugin-sdk/runtime-config-snapshot` |
    | Schreiben der Konfiguration | `openclaw/plugin-sdk/config-mutation` |
    | Hilfsfunktionen für den Sitzungsspeicher | `openclaw/plugin-sdk/session-store-runtime` |
    | Markdown-Tabellenkonfiguration | `openclaw/plugin-sdk/markdown-table-runtime` |
    | Laufzeithelfer für Gruppenrichtlinien | `openclaw/plugin-sdk/runtime-group-policy` |
    | Auflösung geheimer Eingaben | `openclaw/plugin-sdk/secret-input-runtime` |
    | Modell-/Sitzungsüberschreibungen | `openclaw/plugin-sdk/model-session-runtime` |

    Gebündelte Plugins und ihre Tests werden durch Scanner vor dem breiten
    Barrel geschützt, damit Importe und Mocks lokal auf das benötigte Verhalten
    beschränkt bleiben. Das Barrel besteht weiterhin für externe Kompatibilität, neuer Code sollte
    jedoch nicht davon abhängen.

  </Step>

  <Step title="Eingebettete Erweiterungen für Werkzeugergebnisse auf Middleware migrieren">
    Gebündelte Plugins müssen die ausschließlich für eingebettete Runner vorgesehenen
    `api.registerEmbeddedExtensionFactory(...)`-Handler für Werkzeugergebnisse durch
    laufzeitneutrale Middleware ersetzen:

    ```typescript
    // OpenClaw-Laufzeitwerkzeuge und dynamische Werkzeuge der Codex-Laufzeit (das Ergebnis kann
    // transformiert werden). Codex-native Werkzeugergebnisse werden zur Beobachtung ebenfalls weitergeleitet,
    // ihre transformierte Ausgabe erreicht das Modell jedoch nie: Der Codex-
    // PostToolUse-Hook-Vertrag kann eine native Werkzeugantwort nicht ersetzen.
    api.registerAgentToolResultMiddleware(async (event) => {
      return compactToolResult(event);
    }, {
      runtimes: ["openclaw", "codex"],
    });
    ```

    Aktualisieren Sie gleichzeitig das Plugin-Manifest:

    ```json
    {
      "contracts": {
        "agentToolResultMiddleware": ["openclaw", "codex"]
      }
    }
    ```

    Installierte Plugins können ebenfalls Middleware für Werkzeugergebnisse registrieren, wenn sie ausdrücklich
    aktiviert ist und jede adressierte Laufzeit in
    `contracts.agentToolResultMiddleware` deklariert wurde. Nicht deklarierte Middleware-
    Registrierungen installierter Plugins werden abgelehnt.

  </Step>

  <Step title="Native Genehmigungshandler auf Fähigkeitsfakten migrieren">
    Kanäle-Plugins mit Genehmigungsunterstützung stellen natives Genehmigungsverhalten über
    `approvalCapability.nativeRuntime` sowie die gemeinsame Laufzeitkontext-
    Registry bereit:

    - Ersetzen Sie `approvalCapability.handler.loadRuntime(...)` durch
      `approvalCapability.nativeRuntime`.
    - Verschieben Sie genehmigungsspezifische Authentifizierung/Zustellung aus der veralteten `plugin.auth`-/
      `plugin.approvals`-Verdrahtung nach `approvalCapability`.
    - `ChannelPlugin.approvals` wurde aus dem öffentlichen
      Kanal-Plugin-Vertrag entfernt; verschieben Sie Felder für Zustellung, native Verarbeitung und Darstellung nach
      `approvalCapability`.
    - `plugin.auth` bleibt ausschließlich für Anmelde-/Abmeldeabläufe von Kanälen bestehen; der Kern
      liest dort keine Genehmigungs-Authentifizierungshooks mehr.
    - Registrieren Sie kanaleigene Laufzeitobjekte (Clients, Tokens, Bolt-Apps)
      über `openclaw/plugin-sdk/channel-runtime-context`.
    - Senden Sie keine Plugin-eigenen Hinweise auf Umleitung aus nativen Genehmigungshandlern;
      der Kern ist für Hinweise auf eine anderweitige Weiterleitung anhand der tatsächlichen Zustellungsergebnisse verantwortlich.
    - Wenn Sie `channelRuntime` an `createChannelManager(...)` übergeben, stellen Sie eine
      echte `createPluginRuntime().channel`-Oberfläche bereit – unvollständige Stubs werden
      abgelehnt.

    Unter [Kanal-Plugins](/de/plugins/sdk-channel-plugins) finden Sie die aktuelle
    Struktur der Genehmigungsfähigkeiten.

  </Step>

  <Step title="Fallback-Verhalten von Windows-Wrappern prüfen">
    Wenn Ihr Plugin `openclaw/plugin-sdk/windows-spawn` verwendet, schlagen nicht aufgelöste Windows-
    `.cmd`-/`.bat`-Wrapper jetzt standardmäßig sicher fehl, sofern Sie nicht ausdrücklich
    `allowShellFallback: true` übergeben:

    ```typescript
    // Vorher
    const program = applyWindowsSpawnProgramPolicy({ candidate });

    // Nachher
    const program = applyWindowsSpawnProgramPolicy({
      candidate,
      // Legen Sie dies nur für vertrauenswürdige Kompatibilitätsaufrufer fest, die bewusst
      // einen über die Shell vermittelten Fallback akzeptieren.
      allowShellFallback: true,
    });
    ```

    Wenn Ihr Aufrufer nicht bewusst auf einen Shell-Fallback angewiesen ist, setzen Sie
    `allowShellFallback` nicht und behandeln Sie stattdessen den ausgelösten Fehler.

  </Step>

  <Step title="Veraltete Importe finden">
    ```bash
    grep -r "plugin-sdk/compat" my-plugin/
    grep -r "plugin-sdk/infra-runtime" my-plugin/
    grep -r "plugin-sdk/config-runtime" my-plugin/
    grep -r "openclaw/extension-api" my-plugin/
    ```
  </Step>

  <Step title="Durch gezielte Importe ersetzen">
    Jeder Export der alten Oberfläche ist einem bestimmten modernen Importpfad zugeordnet:

    ```typescript
    // Vorher (veraltete Abwärtskompatibilitätsschicht)
    import {
      createChannelReplyPipeline,
      createPluginRuntimeStore,
      resolveControlCommandGate,
    } from "openclaw/plugin-sdk/compat";

    // Nachher (moderne gezielte Importe)
    import { createChannelReplyPipeline } from "openclaw/plugin-sdk/channel-reply-pipeline";
    import { createPluginRuntimeStore } from "openclaw/plugin-sdk/runtime-store";
    import { resolveControlCommandGate } from "openclaw/plugin-sdk/command-auth";
    ```

    Verwenden Sie für hostseitige Hilfsfunktionen die injizierte Plugin-Laufzeit, anstatt
    direkt zu importieren:

    ```typescript
    // Vorher (veraltete extension-api-Brücke)
    import { runEmbeddedAgent } from "openclaw/extension-api";
    const result = await runEmbeddedAgent({ sessionId, prompt });

    // Nachher (injizierte Laufzeit)
    const result = await api.runtime.agent.runEmbeddedAgent({ sessionId, prompt });
    ```

    Dasselbe Muster gilt für weitere veraltete Brückenhilfsfunktionen:

    | Alter Import | Moderne Entsprechung |
    | --- | --- |
    | `resolveAgentDir` | `api.runtime.agent.resolveAgentDir` |
    | `resolveAgentWorkspaceDir` | `api.runtime.agent.resolveAgentWorkspaceDir` |
    | `resolveAgentIdentity` | `api.runtime.agent.resolveAgentIdentity` |
    | `resolveThinkingDefault` | `api.runtime.agent.resolveThinkingDefault` |
    | `resolveAgentTimeoutMs` | `api.runtime.agent.resolveAgentTimeoutMs` |
    | `ensureAgentWorkspace` | `api.runtime.agent.ensureAgentWorkspace` |
    | Hilfsfunktionen für den Sitzungsspeicher | `api.runtime.agent.session.*` |

  </Step>

  <Step title="Breite infra-runtime-Importe ersetzen">
    `openclaw/plugin-sdk/infra-runtime` besteht weiterhin für externe
    Kompatibilität, neuer Code sollte jedoch die tatsächlich
    benötigte gezielte Oberfläche importieren:

    | Bedarf | Import |
    | --- | --- |
    | Hilfsfunktionen für die Systemereigniswarteschlange | `openclaw/plugin-sdk/system-event-runtime` |
    | Hilfsfunktionen für Heartbeat-Aktivierung, Ereignisse und Sichtbarkeit | `openclaw/plugin-sdk/heartbeat-runtime` |
    | Abarbeitung der Warteschlange ausstehender Zustellungen | `openclaw/plugin-sdk/delivery-queue-runtime` |
    | Telemetrie der Kanalaktivität | `openclaw/plugin-sdk/channel-activity-runtime` |
    | Im Arbeitsspeicher und persistent gespeicherte Deduplizierungs-Caches | `openclaw/plugin-sdk/dedupe-runtime` |
    | Sichere Hilfsfunktionen für lokale Datei-/Medienpfade | `openclaw/plugin-sdk/file-access-runtime` |
    | Dispatcher-berücksichtigender Abruf | `openclaw/plugin-sdk/runtime-fetch` |
    | Proxy- und geschützte Abrufhilfsfunktionen | `openclaw/plugin-sdk/fetch-runtime` |
    | Richtlinientypen für SSRF-Dispatcher | `openclaw/plugin-sdk/ssrf-dispatcher` |
    | Typen für Genehmigungsanfragen/-auflösungen | `openclaw/plugin-sdk/approval-runtime` |
    | Hilfsfunktionen für Genehmigungsantwort-Nutzlasten und Befehle | `openclaw/plugin-sdk/approval-reply-runtime` |
    | Hilfsfunktionen zur Fehlerformatierung | `openclaw/plugin-sdk/error-runtime` |
    | Warten auf Transportbereitschaft | `openclaw/plugin-sdk/transport-ready-runtime` |
    | Hilfsfunktionen für sichere Tokens | `openclaw/plugin-sdk/secure-random-runtime` |
    | Begrenzte Nebenläufigkeit asynchroner Aufgaben | `openclaw/plugin-sdk/concurrency-runtime` |
    | Pflichtwertprüfungen für beweisbare Invarianten | `openclaw/plugin-sdk/expect-runtime` |
    | Numerische Umwandlung | `openclaw/plugin-sdk/number-runtime` |
    | Prozesslokale asynchrone Sperre | `openclaw/plugin-sdk/async-lock-runtime` |
    | Dateisperren | `openclaw/plugin-sdk/file-lock` |

    Gebündelte Plugins werden durch Scanner vor `infra-runtime` geschützt, sodass Repository-Code
    nicht zum breiten Barrel zurückkehren kann.

  </Step>

  <Step title="Hilfsfunktionen für Kanalrouten migrieren">
    Neuer Code für Kanalrouten verwendet `openclaw/plugin-sdk/channel-route`. Die älteren
    Routenschlüsselnamen bleiben als Kompatibilitätsaliase erhalten:

    | Alte Hilfsfunktion | Moderne Hilfsfunktion |
    | --- | --- |
    | `channelRouteIdentityKey(...)` | `channelRouteDedupeKey(...)` |
    | `channelRouteKey(...)` | `channelRouteCompactKey(...)` |

    Die modernen Routenhilfsfunktionen normalisieren `{ channel, to, accountId, threadId }`
    einheitlich über native Genehmigungen, Antwortunterdrückung, Deduplizierung eingehender Nachrichten,
    Cron-Zustellung und Sitzungsrouting hinweg.

    Fügen Sie keine neuen Verwendungen von `ChannelMessagingAdapter.parseExplicitTarget` oder
    `resolveChannelRouteTargetWithParser(...)` aus
    `plugin-sdk/channel-route` hinzu – diese sind veraltet und bleiben nur für ältere
    Plugins bestehen. Neue Kanal-Plugins sollten
    `messaging.targetResolver.resolveTarget(...)` für die Normalisierung der Ziel-ID
    und den Fallback bei fehlendem Verzeichniseintrag,
    `messaging.inferTargetChatType(...)`, wenn der Kern frühzeitig eine Peer-Art benötigt,
    und `messaging.resolveOutboundSessionRoute(...)` für Provider-native
    Sitzungs- und Thread-Identität verwenden.

  </Step>

  <Step title="Erstellen und testen">
    ```bash
    pnpm build
    pnpm test my-plugin/
    ```
  </Step>
</Steps>

## Referenz für Importpfade

Die öffentliche Exportzuordnung des Pakets ist die maßgebliche Quelle für importierbare SDK-
Unterpfade. Verwenden Sie die thematischen SDK-Leitfäden, die in der [SDK-Übersicht](/de/plugins/sdk-overview)
verlinkt sind, und bevorzugen Sie den schmalsten dokumentierten öffentlichen Unterpfad. Das Compiler-Inventar in
`scripts/lib/plugin-sdk-entrypoints.json` enthält außerdem private lokale Einträge, die
zum Erstellen gebündelter Plugins verwendet werden; ihre dortige Existenz macht sie nicht zu öffentlichen Paketexporten.

Diese Tabelle stellt die häufig verwendete Migrationsuntermenge dar, nicht die vollständige SDK-Oberfläche. Das
Inventar der Compiler-Einstiegspunkte befindet sich in `scripts/lib/plugin-sdk-entrypoints.json`;
Paketexporte werden aus der öffentlichen Untermenge generiert.

Reservierte Hilfsfunktionsschnittstellen für gebündelte Plugins wurden aus der öffentlichen SDK-
Exportzuordnung entfernt, mit Ausnahme ausdrücklich dokumentierter Kompatibilitätsfassaden wie dem
veralteten `plugin-sdk/discord`-Shim, der für externe Plugins beibehalten wird, die weiterhin
das veröffentlichte `@openclaw/discord`-Paket direkt importieren. Eigentümerspezifische
Hilfsfunktionen befinden sich innerhalb des zuständigen Plugin-Pakets; gemeinsames Hostverhalten wird
über generische SDK-Verträge wie `plugin-sdk/gateway-runtime`,
`plugin-sdk/security-runtime` und die injizierte Plugin-API bereitgestellt.

Verwenden Sie den schmalsten Import, der zur Aufgabe passt. Wenn Sie keinen Export finden,
prüfen Sie die Quelle unter `src/plugin-sdk/` oder fragen Sie die Maintainer, welchem generischen
Vertrag er zugeordnet werden sollte.

## Entfernte Kompatibilitätsoberflächen

Bei der Bereinigung im Juli 2026 wurden die Root-SDK- und Kompatibilitäts-Barrels, die Extension-API-
Brücke, die abgelaufenen SDK-Unterpfadaliase, ungenutzte SDK-Unterpfade und die öffentlichen
Exporte für ausschließlich gebündelt verwendete SDK-Module entfernt. Ausschließlich gebündelt verwendete Module bleiben für
ihre Repository-Eigentümer über private lokale Build-Zuordnungen verfügbar; sie können nicht
aus dem veröffentlichten Paket importiert werden.

### Prozessglobale Veröffentlichung von API-Providern

`registerApiProvider(...)` und `unregisterApiProviders(...)` wurden aus
`openclaw/plugin-sdk/llm` entfernt. Sie veröffentlichten API-Transporte im prozessglobalen
Zustand, die lebenszyklusverwaltete Modelllaufzeiten anschließend in jede vorbereitete
Registry kopieren mussten.

Provider-Plugins sollten Provider für Textinferenz über
`api.registerProvider(...)` registrieren. Hosteigener Code und Tests, die eine
`ApiRegistry` erstellen, sollten die Registrierung direkt in dieser Registry vornehmen, damit Eigentümerschaft
und Abbau des Providers auf die vorbereitete Laufzeit beschränkt bleiben.

### Privates Test-Barrel

`openclaw/plugin-sdk/testing` war Repository-lokal und von ausgelieferten Paket-
Artefakten ausgeschlossen, daher wurde es vor seinem `removeAfter`-Datum am 2026-07-28 entfernt. Repository-
Tests verwenden gezielte Unterpfade wie `plugin-sdk/plugin-test-runtime`,
`plugin-sdk/channel-test-helpers`, `plugin-sdk/channel-target-testing`,
`plugin-sdk/test-env` und `plugin-sdk/test-fixtures`.

## Migrationsreferenz

  Diese Zuordnungen decken sowohl die im Juli 2026 entfernten Oberflächen als auch aktive
  Deprecations in späteren Zeitfenstern ab. Eine Zuordnung ist eine Migrationsanleitung und kein Nachweis dafür, dass die alte
  Oberfläche weiterhin verfügbar ist; den aktuellen Status finden Sie im Kompatibilitätsregister und in der
  Zeitleiste für Entfernungen.

  <AccordionGroup>
  <Accordion title="Builder für command-auth-Hilfe -> command-status">
    **Alt (`openclaw/plugin-sdk/command-auth`)**: `buildCommandsMessage`,
    `buildCommandsMessagePaginated`, `buildHelpMessage`.

    **Neu (`openclaw/plugin-sdk/command-status`)**: dieselben Signaturen, importiert
    aus dem enger gefassten Unterpfad. Die Kompatibilitäts-Re-Exports von `command-auth`
    wurden entfernt.

    ```typescript
    // Vorher
    import { buildHelpMessage } from "openclaw/plugin-sdk/command-auth";

    // Nachher
    import { buildHelpMessage } from "openclaw/plugin-sdk/command-status";
    ```

  </Accordion>

  <Accordion title="Hilfsfunktionen für Mention-Gating -> resolveInboundMentionDecision">
    **Alt**: `resolveMentionGating(params)` und
    `resolveMentionGatingWithBypass(params)` aus
    `openclaw/plugin-sdk/channel-inbound` oder
    `openclaw/plugin-sdk/channel-mention-gating`.

    **Neu**: `resolveInboundMentionDecision({ facts, policy })` – ein Entscheidungsobjekt
    anstelle zweier getrennter Aufrufformen.

    Übernommen für Discord, iMessage, Matrix, MS Teams, QQBot, Signal,
    Telegram, WhatsApp und Zalo. Slacks eigenes `app_mention`-Ereignismodell
    verwendet diese Hilfsfunktion nicht.

  </Accordion>

  <Accordion title="Channel-Runtime-Shim und Hilfsfunktionen für Channel-Aktionen">
    `openclaw/plugin-sdk/channel-runtime` wurde entfernt. Verwenden Sie
    `openclaw/plugin-sdk/channel-runtime-context`, um Runtime-Objekte zu
    registrieren.

    Die Hilfsfunktionen für das native Nachrichtenschema in `openclaw/plugin-sdk/channel-actions`
    wurden zusammen mit den rohen Channel-Exports für „actions“ entfernt. Stellen Sie Fähigkeiten
    stattdessen über die semantische `presentation`-Oberfläche bereit – Channel-Plugins
    deklarieren, was sie darstellen (Karten, Schaltflächen, Auswahlelemente), statt welche rohen
    Aktionsnamen sie akzeptieren.

  </Accordion>

  <Accordion title="Websuche-Provider-Hilfsfunktion tool() -> createTool() im Plugin">
    **Alt**: `tool()`-Factory aus `openclaw/plugin-sdk/provider-web-search`.

    **Neu**: Implementieren Sie `createTool(...)` direkt im Provider-Plugin.
    OpenClaw benötigt die SDK-Hilfsfunktion nicht mehr, um den Tool-Wrapper zu registrieren.

  </Accordion>

  <Accordion title="Klartext-Channel-Envelopes -> BodyForAgent">
    **Alt**: `api.runtime.channel.reply.formatInboundEnvelope(...)` (und das
    Feld `channelEnvelope` in eingehenden Nachrichtenobjekten), um aus eingehenden
    Channel-Nachrichten ein flaches Klartext-Prompt-Envelope zu erstellen.

    **Neu**: `BodyForAgent` plus strukturierte Benutzerkontextblöcke. Channel-
    Plugins fügen Routing-Metadaten (Thread, Thema, Antwortbezug, Reaktionen) als
    typisierte Felder hinzu, statt sie zu einem Prompt-String zu verketten. Die
    Hilfsfunktion `formatAgentEnvelope(...)` wird für synthetisch erzeugte
    assistentenseitige Envelopes weiterhin unterstützt, eingehende Klartext-Envelopes werden jedoch
    schrittweise abgeschafft.

    Betroffene Bereiche: `inbound_claim`, `message_received` und jedes benutzerdefinierte
    Channel-Plugin, das den alten Envelope-Text nachverarbeitet hat.

  </Accordion>

  <Accordion title="Hook deactivate -> gateway_stop">
    **Alt**: `api.on("deactivate", handler)`.

    **Neu**: `api.on("gateway_stop", handler)`. Derselbe Vertrag für die Bereinigung beim Herunterfahren;
    nur der Hook-Name ändert sich.

    ```typescript
    // Vorher
    api.on("deactivate", async (event, ctx) => {
      await stopPluginService(ctx);
    });

    // Nachher
    api.on("gateway_stop", async (event, ctx) => {
      await stopPluginService(ctx);
    });
    ```

    `deactivate` bleibt als veralteter Kompatibilitätsalias eingebunden, bis er
    nach dem 2026-08-16 entfernt wird.

  </Accordion>

  <Accordion title="Hook subagent_spawning -> Core-Thread-Bindung">
    **Alt**: `api.on("subagent_spawning", handler)` mit Rückgabe von
    `threadBindingReady` oder `deliveryOrigin`.

    **Neu**: Lassen Sie den Core `thread: true`-Subagent-Bindungen über den
    Adapter für Channel-Sitzungsbindungen vorbereiten. Verwenden Sie `api.on("subagent_spawned", handler)`
    nur zur Beobachtung nach dem Start.

    ```typescript
    // Vorher
    api.on("subagent_spawning", async () => ({
      status: "ok",
      threadBindingReady: true,
      deliveryOrigin: { channel: "discord", to: "channel:123", threadId: "456" },
    }));

    // Nachher
    api.on("subagent_spawned", async (event) => {
      await observeSubagentLaunch(event);
    });
    ```

    `subagent_spawning`, `PluginHookSubagentSpawningEvent`,
    `PluginHookSubagentSpawningResult` und
    `SubagentLifecycleHookRunner.runSubagentSpawning(...)` bleiben nur als
    veraltete Kompatibilitätsoberflächen bestehen, während externe Plugins migrieren, und werden
    nach dem 2026-08-30 entfernt.

  </Accordion>

  <Accordion title="Provider-Discovery-Typen -> Provider-Katalogtypen">
    Vier Discovery-Typaliase sind jetzt dünne Wrapper um die Typen der
    Katalogära:

    | Alter Alias                 | Neuer Typ                  |
    | ------------------------- | ------------------------- |
    | `ProviderDiscoveryOrder`  | `ProviderCatalogOrder`    |
    | `ProviderDiscoveryContext`| `ProviderCatalogContext`  |
    | `ProviderDiscoveryResult` | `ProviderCatalogResult`   |
    | `ProviderPluginDiscovery` | `ProviderPluginCatalog`   |

    Die Aliase und der alte statische `ProviderCapabilities`-Container wurden
    entfernt. Provider-Plugins
    sollten explizite Provider-Hooks wie `buildReplayPolicy`,
    `normalizeToolSchemas` und `wrapStreamFn` statt eines statischen Objekts verwenden.

  </Accordion>

  <Accordion title="Hooks für Denkregeln -> resolveThinkingProfile">
    **Alt** (drei separate Hooks in `ProviderThinkingPolicy`):
    `isBinaryThinking(ctx)`, `supportsXHighThinking(ctx)` und
    `resolveDefaultThinkingLevel(ctx)`.

    **Neu**: ein einzelnes `resolveThinkingProfile(ctx)`, das ein
    `ProviderThinkingProfile` mit dem kanonischen `id`, optionalem `label` und einer
    nach Rang geordneten Stufenliste zurückgibt. OpenClaw stuft veraltete gespeicherte Werte automatisch anhand des Profilrangs
    herunter.

    Der Kontext enthält `provider`, `modelId`, optional zusammengeführte `reasoning`-
    und optional zusammengeführte Modell-`compat`-Fakten. Provider-Plugins können diese
    Katalogfakten verwenden, um ein modellspezifisches Profil nur dann bereitzustellen, wenn der konfigurierte
    Anfragevertrag dies unterstützt.

    Implementieren Sie einen Hook statt drei. Die alten Hooks wurden entfernt.

  </Accordion>

  <Accordion title="Externe Authentifizierungs-Provider -> contracts.externalAuthProviders">
    **Alt**: Implementierung externer Authentifizierungs-Hooks, ohne den Provider
    im Plugin-Manifest zu deklarieren.

    **Neu**: Deklarieren Sie `contracts.externalAuthProviders` im Plugin-Manifest
    **und** implementieren Sie `resolveExternalAuthProfiles(...)`.

    ```json
    {
      "contracts": {
        "externalAuthProviders": ["anthropic", "openai"]
      }
    }
    ```

  </Accordion>

  <Accordion title="Suche nach Provider-Umgebungsvariablen -> setup.providers[].envVars">
    **Altes** Manifestfeld: `providerAuthEnvVars: { anthropic: ["ANTHROPIC_API_KEY"] }`.

    **Neu**: Spiegeln Sie dieselbe Suche nach Umgebungsvariablen in `setup.providers[].envVars`
    im Manifest. Dadurch werden Umgebungsmetadaten für Einrichtung und Status an einer Stelle zusammengeführt,
    und es ist nicht mehr nötig, die Plugin-Runtime nur für die Beantwortung von Umgebungsvariablenabfragen zu starten.

    `providerAuthEnvVars` wird nicht mehr akzeptiert.

  </Accordion>

  <Accordion title="Registrierung des Memory-Plugins -> registerMemoryCapability">
    **Alt**: drei separate Aufrufe – `api.registerMemoryPromptSection(...)`,
    `api.registerMemoryFlushPlan(...)`, `api.registerMemoryRuntime(...)`.

    **Neu**: ein Aufruf in der Memory-State-API –
    `registerMemoryCapability(pluginId, { promptBuilder, flushPlanResolver, runtime })`.

    Dieselben Slots, ein einzelner Registrierungsaufruf. Additive Prompt- und Korpus-Hilfsfunktionen
    (`registerMemoryPromptSupplement`, `registerMemoryCorpusSupplement`) sind
    nicht betroffen.

  </Accordion>

  <Accordion title="API für Memory-Embedding-Provider">
    **Alt**: `api.registerMemoryEmbeddingProvider(...)` plus
    `contracts.memoryEmbeddingProviders`.

    **Neu**: `api.registerEmbeddingProvider(...)` plus
    `contracts.embeddingProviders`.

    Der generische Vertrag für Embedding-Provider ist außerhalb von Memory wiederverwendbar und
    der unterstützte Pfad für neue Provider. Die Memory-spezifische Registrierungs-API
    bleibt als veraltete Kompatibilitätsoberfläche eingebunden, während bestehende Provider
    migrieren. Die Plugin-Inspektion meldet eine nicht gebündelte Nutzung als
    Kompatibilitätsschuld.

  </Accordion>

  <Accordion title="Rohe Channel-Sendeergebnisse -> OutboundDeliveryResult">
    **Alt**: `{ ok, messageId, error }` über
    `ChannelSendRawResult` zurückgeben und mit
    `createRawChannelSendResultAdapter(...)` normalisieren.

    **Neu**: Geben Sie `OutboundDeliveryResult`-Felder zurück und fügen Sie den Channel mit
    `createAttachedChannelResultAdapter(...)` hinzu. Fehlgeschlagene Sendevorgänge sollten eine Ausnahme auslösen,
    statt einen Fehler-String zurückzugeben. Der rohe Ergebnistyp bleibt bis
    zur nächsten Hauptversion des Plugin-SDK verfügbar.

  </Accordion>

  <Accordion title="Typen für Subagent-Sitzungsnachrichten umbenannt">
    Zwei alte Typaliase werden weiterhin aus `src/plugins/runtime/types.ts` exportiert:

    | Alt                           | Neu                             |
    | ----------------------------- | ------------------------------- |
    | `SubagentReadSessionParams`   | `SubagentGetSessionMessagesParams` |
    | `SubagentReadSessionResult`   | `SubagentGetSessionMessagesResult` |

    Die Runtime-Methode `readSession` ist zugunsten von
    `getSessionMessages` veraltet. Dieselbe Signatur; die alte Methode delegiert an die
    neue.

  </Accordion>

  <Accordion title="Entfernte APIs für Sitzungs- und Transkriptdateien">
    Die Umstellung von Sitzungen und Transkripten auf SQLite entfernt oder verwirft schrittweise Plugin-seitige APIs,
    die aktive `sessions.json`-Speicher, JSONL-Transkriptpfade oder Listen
    von Sitzungsdateien bereitgestellt haben. Runtime-Plugins sollten Sitzungsidentitäten und SDK-Runtime-
    Hilfsfunktionen verwenden, statt aktive Dateien aufzulösen oder zu verändern.

    | Zu migrierende Oberfläche | Ersatz |
    | ----------------- | ----------- |
    | Veraltete `loadSessionStore(...)`, `updateSessionStore(...)` und `resolveSessionStoreEntry(...)` | `getSessionEntry(...)`, `listSessionEntries(...)` und Sitzungsmutationen auf Zeilenebene. |
    | Veraltetes `resolveSessionFilePath(...)` | Sitzungsidentität (`sessionKey`, `sessionId` und SDK-Hilfsfunktionen für Runtime-Ziele) sowie Gateway-Methoden, die auf der aktuellen Sitzung arbeiten. |
    | Entferntes `saveSessionStore(...)` | Gateway-eigene Sitzungs-Runtime-APIs; Plugin-Code sollte Sitzungszustand über dokumentierte Runtime-/Kontexthilfsfunktionen abrufen oder ändern, statt in die aktive Speicherdatei zu schreiben. |
    | Entfernte `resolveSessionTranscriptPathInDir(...)` und `resolveAndPersistSessionFile(...)` | Sitzungsidentität und Gateway-Methoden, die auf der aktuellen Sitzung arbeiten. |
    | `readLatestAssistantTextFromSessionTranscript(...)` | Identitätsgestützte Transkriptleser, die vom aktuellen Runtime-Kontext bereitgestellt werden, oder Gateway-Verlaufs-/Sitzungsmethoden, wenn sich das Plugin außerhalb des Eigentümerpfads des Transkripts befindet. |
    | `SessionTranscriptUpdate.sessionFile` | `SessionTranscriptUpdate.target` mit `agentId`, `sessionKey` und `sessionId`. |
    | Memory-Synchronisierungseingaben wie `sessionFiles` | Vom Host bereitgestellte identitätsgestützte Transkript-/Sitzungsquellen; durchsuchen Sie für Live-Sitzungen keine aktiven JSONL-Dateien. |
    | Runtime-Optionen namens `transcriptPath` oder `sessionFile` für aktive Sitzungen | `sessionTarget`/Runtime-Zielobjekte, die eine speicherneutrale Sitzungsidentität enthalten. |

    Alte JSONL-Transkriptdateien bleiben als Import-, Archiv-, Export- und
    Support-Artefakte gültig. Sie sind nicht länger der Vertrag für den dauerhaften Runtime-Betrieb
    aktiver Sitzungen.

    Offizielle Plugins, die mit `v2026.7.1-beta.5` veröffentlicht wurden, importierten die vier
    oben genannten veralteten Hilfsfunktionen. `openclaw/plugin-sdk/session-store-runtime` erhält
    genau diese Brücke bis zum 2026-10-12 aufrecht; neue Plugins müssen die Ersatzlösungen verwenden.
    `resolveStorePath(...)` bleibt eine unterstützte SDK-Hilfsfunktion und ist nicht Teil
    dieser Deprecation.

    `openclaw plugins inspect --all --runtime` meldet nicht gebündelte Plugins, deren
    Ladefehler oder Diagnosen noch auf diese entfernten Datei-APIs verweisen. Der
    `@openclaw/plugin-inspector`-Advisory-Durchlauf muss Version `0.3.17` oder
    neuer verwenden, damit Scans externer Pakete vor der Veröffentlichung auch Hilfsfunktionen für vollständige Sitzungsspeicher,
    Hilfsfunktionen für Sitzungsdateipfade, alte Transkriptdateiziele und Low-Level-
    Transkripthilfsfunktionen kennzeichnen.

  </Accordion>

  <Accordion title="runtime.tasks.flow -> runtime.tasks.managedFlows">
    **Alt**: `runtime.tasks.flow` (Singular) gab einen Live-Accessor für TaskFlow
    zurück.

    **Neu**: `runtime.tasks.managedFlows` behält die verwaltete TaskFlow-Mutations-
    Runtime für Plugins bei, die untergeordnete Aufgaben aus einem
    Ablauf erstellen, aktualisieren, abbrechen oder ausführen. Verwenden Sie `runtime.tasks.flows`, wenn das Plugin nur DTO-basierte
    Lesezugriffe benötigt.

    ```typescript
    // Vorher
    const flow = api.runtime.tasks.flow.fromToolContext(ctx);
    // Nachher
    const flow = api.runtime.tasks.managedFlows.fromToolContext(ctx);
    ```

    Die Legacy-Aliasse wurden im Juli 2026 entfernt.

  </Accordion>

  <Accordion title="Eingebettete Erweiterungs-Factorys -> Middleware für Agent-Tool-Ergebnisse">
    Dies wird oben unter [Migration](#how-to-migrate) behandelt. Der
    Vollständigkeit halber ist es hier ebenfalls aufgeführt: Der entfernte, ausschließlich
    für eingebettete Runner bestimmte Pfad `api.registerEmbeddedExtensionFactory(...)` wird durch
    `api.registerAgentToolResultMiddleware(...)` mit einer expliziten Runtime-Liste
    in `contracts.agentToolResultMiddleware` ersetzt.
  </Accordion>

  <Accordion title="OpenClawSchemaType-Alias -> OpenClawConfig">
    Der Root-SDK-Alias `OpenClawSchemaType` wurde entfernt. Verwenden Sie den
    kanonischen Namen `OpenClawConfig`.

    ```typescript
    // Vorher
    import type { OpenClawSchemaType } from "openclaw/plugin-sdk";
    // Nachher
    import type { OpenClawConfig } from "openclaw/plugin-sdk/config-contracts";
    ```

  </Accordion>
</AccordionGroup>

<Note>
Veraltete Funktionen auf Erweiterungsebene (innerhalb gebündelter Kanal-/Provider-Plugins unter
`extensions/`) werden in ihren eigenen `api.ts`- und `runtime-api.ts`-
Barrels nachverfolgt. Sie wirken sich nicht auf Verträge von Drittanbieter-Plugins aus und sind
hier nicht aufgeführt. Wenn Sie das lokale Barrel eines gebündelten Plugins direkt verwenden,
lesen Sie vor dem Upgrade die Hinweise zu veralteten Funktionen in diesem Barrel.
</Note>

## Migration von Talk und Echtzeitsprachfunktionen

Echtzeitsprach-, Telefonie-, Meeting- und Browser-Talk-Code verwendet gemeinsam einen Talk-
Sitzungscontroller, der von `openclaw/plugin-sdk/realtime-voice` exportiert wird. Der
Controller verwaltet die gemeinsame Talk-Ereignishülle, den Zustand der aktiven Gesprächsrunde, den Erfassungszustand,
den Audioausgabezustand, den Verlauf der letzten Ereignisse und die Zurückweisung veralteter Gesprächsrunden.
Provider-Plugins verwalten anbieterspezifische Echtzeitsitzungen. Browser-Meeting-Plugins
verwenden `openclaw/plugin-sdk/meeting-runtime` für Sitzungs-, Browser-, Audio-, Node-Host-,
Agent-Konsultations- und Sprachanrufmechanismen und implementieren anschließend `MeetingPlatformAdapter`
für URL-Regeln, DOM-Skripte, die Zuordnung manueller Aktionen, Untertitel, Erstellung und Einwahlpläne.
Plattform-REST-APIs, OAuth, Artefakte, Selektoren und Wire-Namen verbleiben im
Plugin. Browser-Berechtigungspläne erhalten die angeforderte Meeting-URL, damit jede
Plattform ausschließlich ihre exakt unterstützten Ursprünge freigeben kann. Sitzungs-Runtimes müssen außerdem
den plattformspezifischen Live-Funktionszustand nach dem bestätigten Verlassen des Browsers normalisieren;
historische Transkriptfelder dürfen erhalten bleiben, aber die Bereitschaft für Untertitel und Audio darf
nach dem Verlassen nicht aktiv bleiben.

Alle gebündelten Oberflächen verwenden den gemeinsamen Controller: Browser-Relay,
Übergabe verwalteter Räume, Echtzeit-Sprachanrufe, Streaming-STT für Sprachanrufe, Google
Meet in Echtzeit und natives Push-to-Talk. Der Gateway kündigt in
`hello-ok.features.events` einen einzigen Live-Talk-Ereigniskanal an: `talk.event`.

Neuer Code sollte `createTalkEventSequencer(...)` nicht direkt aufrufen, außer
bei der Implementierung eines Low-Level-Adapters oder einer Test-Fixture. Verwenden Sie den gemeinsamen Controller, damit
Ereignisse im Geltungsbereich einer Gesprächsrunde nicht ohne Gesprächsrunden-ID ausgegeben werden können, veraltete Aufrufe von `turnEnd` /
`turnCancel` keine neuere aktive Gesprächsrunde löschen können und Ereignisse des
Audioausgabe-Lebenszyklus über Telefonie, Meetings, Browser-Relay,
Übergabe verwalteter Räume und native Talk-Clients hinweg konsistent bleiben.

Die Form der öffentlichen API:

```typescript
// Gateway-eigene Talk-Sitzungs-API.
await gateway.request("talk.session.create", {
  mode: "realtime",
  transport: "gateway-relay",
  brain: "agent-consult",
  sessionKey: "main",
});
await gateway.request("talk.session.appendAudio", { sessionId, audioBase64 });
await gateway.request("talk.session.cancelOutput", { sessionId, reason: "barge-in" });
await gateway.request("talk.session.submitToolResult", {
  sessionId,
  callId,
  result: { status: "working" },
  options: { willContinue: true },
});
await gateway.request("talk.session.submitToolResult", {
  sessionId,
  callId,
  result: { status: "already_delivered" },
  options: { suppressResponse: true },
});
await gateway.request("talk.session.submitToolResult", { sessionId, callId, result });
await gateway.request("talk.session.close", { sessionId });

// Client-eigene Provider-Sitzungs-API.
await gateway.request("talk.client.create", {
  mode: "realtime",
  transport: "webrtc",
  brain: "agent-consult",
  sessionKey: "main",
});
await gateway.request("talk.client.toolCall", { sessionKey, callId, name, args });
await gateway.request("talk.client.steer", { sessionKey, text, mode: "steer" });
```

Browsereigene WebRTC-/Provider-WebSocket-Sitzungen verwenden `talk.client.create`,
da der Browser die Provider-Aushandlung und den Medientransport verwaltet, während der
Gateway Anmeldedaten, Anweisungen und Tool-Richtlinien verwaltet. `talk.session.*` ist
die gemeinsame vom Gateway verwaltete Oberfläche für Gateway-Relay-Echtzeit,
Gateway-Relay-Transkription und native STT-/TTS-Sitzungen in verwalteten Räumen.

Legacy-Konfigurationen, die Echtzeitselektoren neben `talk.provider` /
`talk.providers` platzieren, sollten mit `openclaw doctor --fix` repariert werden; Talk
interpretiert zur Laufzeit die Sprach-/TTS-Provider-Konfiguration nicht als Echtzeit-Provider-Konfiguration neu.

Die unterstützten Kombinationen von `talk.session.create` sind bewusst begrenzt:

| Modus           | Transport       | Verarbeitung     | Eigentümer         | Hinweise                                                                                                                     |
| --------------- | --------------- | ---------------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| `realtime`      | `gateway-relay` | `agent-consult` | Gateway            | Vollduplex-Provider-Audio wird über den Gateway übertragen; Tool-Aufrufe werden durch das Agent-Konsultations-Tool geleitet.   |
| `transcription` | `gateway-relay` | `none`          | Gateway            | Ausschließlich Streaming-STT; Aufrufer senden Eingangsaudio und empfangen Transkriptereignisse.                              |
| `stt-tts`       | `managed-room`  | `agent-consult` | Nativer/Client-Raum | Räume im Push-to-Talk- und Walkie-Talkie-Stil, in denen der Client Erfassung/Wiedergabe und der Gateway den Gesprächsrundenzustand verwaltet. |
| `stt-tts`       | `managed-room`  | `direct-tools`  | Nativer/Client-Raum | Nur für Administratoren vorgesehener Raummodus für vertrauenswürdige Erstanbieteroberflächen, die Gateway-Tool-Aktionen direkt ausführen. |

Methodenzuordnung für Leser, die von den älteren Familien `talk.realtime.*` /
`talk.transcription.*` / `talk.handoff.*` migrieren (alle entfernt):

| Alt                              | Neu                                                      |
| -------------------------------- | -------------------------------------------------------- |
| `talk.realtime.session`          | `talk.client.create`                                     |
| `talk.realtime.toolCall`         | `talk.client.toolCall`                                   |
| `talk.realtime.relayAudio`       | `talk.session.appendAudio`                               |
| `talk.realtime.relayCancel`      | `talk.session.cancelOutput` oder `talk.session.cancelTurn` |
| `talk.realtime.relayToolResult`  | `talk.session.submitToolResult`                          |
| `talk.realtime.relayStop`        | `talk.session.close`                                     |
| `talk.transcription.session`     | `talk.session.create({ mode: "transcription" })`         |
| `talk.transcription.relayAudio`  | `talk.session.appendAudio`                               |
| `talk.transcription.relayCancel` | `talk.session.cancelTurn`                                |
| `talk.transcription.relayStop`   | `talk.session.close`                                     |
| `talk.handoff.create`            | `talk.session.create({ transport: "managed-room" })`     |
| `talk.handoff.join`              | `talk.session.join`                                      |
| `talk.handoff.revoke`            | `talk.session.close`                                     |

Das vereinheitlichte Steuerungsvokabular ist ebenfalls bewusst begrenzt:

| Methode                         | Gilt für                                                | Vertrag                                                                                                                                                                                                                                 |
| ------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `talk.session.appendAudio`      | `realtime/gateway-relay`, `transcription/gateway-relay` | Einen base64-kodierten PCM-Audioblock an die Provider-Sitzung anhängen, die derselben Gateway-Verbindung zugeordnet ist.                                                                                                                  |
| `talk.session.startTurn`        | `stt-tts/managed-room`                                  | Eine Benutzer-Gesprächsrunde in einem verwalteten Raum starten.                                                                                                                                                                         |
| `talk.session.endTurn`          | `stt-tts/managed-room`                                  | Die aktive Gesprächsrunde nach der Validierung auf veraltete Gesprächsrunden beenden.                                                                                                                                                    |
| `talk.session.cancelTurn`       | alle Gateway-eigenen Sitzungen                         | Aktive Erfassungs-, Provider-, Agent- oder TTS-Arbeit für eine Gesprächsrunde abbrechen.                                                                                                                                                 |
| `talk.session.cancelOutput`     | `realtime/gateway-relay`                                | Die Audioausgabe des Assistenten stoppen, ohne zwingend die Benutzer-Gesprächsrunde zu beenden.                                                                                                                                          |
| `talk.session.submitToolResult` | `realtime/gateway-relay`                                | Einen Provider-Tool-Aufruf nach einem von seiner Bridge bereitgestellten asynchronen Abschluss abschließen; `options.willContinue` für eine Zwischenausgabe oder, sofern unterstützt, `options.suppressResponse` übergeben, um eine weitere Assistentenantwort zu vermeiden. |
| `talk.session.steer`            | agentengestützte Talk-Sitzungen                         | Die gesprochene Steuerung `status`, `steer`, `cancel` oder `followup` an den aktiven eingebetteten Lauf senden, der aus der Talk-Sitzung aufgelöst wurde.                                          |
| `talk.session.close`            | alle vereinheitlichten Sitzungen                        | Relay-Sitzungen stoppen oder den Zustand verwalteter Räume widerrufen und anschließend die vereinheitlichte Sitzungs-ID verwerfen.                                                                                                      |

Führen Sie keine Provider- oder Plattformsonderfälle im Kern ein, um dies zu ermöglichen.
Der Kern verwaltet die Semantik von Talk-Sitzungen. Provider-Plugins verwalten die Einrichtung von Anbietersitzungen.
Sprachanrufe und Google Meet verwalten Telefonie-/Meeting-Adapter. Browser und native
Apps verwalten die UX für Geräteerfassung und -wiedergabe.

## Zeitplan für die Entfernung

| Wann                                        | Was geschieht                                                                                                                              |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **Jetzt**                                     | Veraltete Oberflächen, die Warnungen unterstützen, geben Laufzeitwarnungen aus; Repository-Prüfungen lehnen veraltete SDK-Importe aus dem Kern und gebündelten Plugins ab. |
| **Ausstehende Entscheidung des Verantwortlichen**                  | Datumslose Einträge bleiben veraltet und können nicht entfernt werden, bis ihr Verantwortlicher ein `removeAfter`-Datum veröffentlicht.                          |
| **Das `removeAfter`-Datum jedes Kompatibilitätseintrags** | Diese spezifische Oberfläche kann entfernt werden; `pnpm plugins:boundary-report --fail-on-eligible-compat` lässt die CI nach Ablauf des Datums fehlschlagen.    |
| **Nächste Hauptversion**                      | Datierte Oberflächen dürfen erst nach ihrem `removeAfter`-Datum entfernt werden; datumslose Einträge erfordern weiterhin die Genehmigung des Verantwortlichen und ein veröffentlichtes Datum.   |

Für die nachstehenden verbleibenden öffentlichen SDK-Unterpfade gelten registry-gestützte Entfernungsfristen.
Die Zeilen vom 30. Juli wurden nach ihrer vorzeitigen, von den Maintainern genehmigten Bereinigung entfernt:
Nicht verwendete Unterpfade wurden gelöscht, frühere Kompatibilitätsaliase wurden gelöscht und
ausschließlich für gebündelte Plugins vorgesehene Module wurden zu privaten lokalen Build-Zuordnungen herabgestuft.

| `removeAfter` | Stufe                               | SDK-Unterpfade                                                                                                                                                                        |
| ------------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `2026-08-15`  | Frühere Kompatibilitätsveraltungen | `agent-config-primitives`, `channel-logging`, `channel-secret-runtime`, `channel-streaming`, `group-access`, `inbound-reply-dispatch`, `matrix`, `text-runtime`, `zod`              |
| `2026-09-01`  | Frühere Kompatibilitätsveraltungen | `channel-lifecycle`, `channel-message`, `channel-reply-pipeline`, `config-runtime`, `infra-runtime`                                                                                 |
| `2026-10-01`  | Veraltete Medienprojektion            | `agent-media-payload` sowie die `MsgContext Media*`-Felder, die keine Unterpfade sind, Builder für eingehende Medien-Nutzlasten von Kanälen, `buildMediaPayload`, Medienaliase für Hooks und `{{Media*}}`-Vorlagen |

Alle Kern-Plugins wurden bereits migriert. Externe Plugins sollten
vor der nächsten Hauptversion migriert werden. Führen Sie `pnpm plugins:boundary-report` aus, um zu sehen, welche
Kompatibilitätseinträge für die von Ihrem Plugin verwendeten Oberflächen als Nächstes fällig sind.

## Warnungen vorübergehend unterdrücken

```bash
OPENCLAW_SUPPRESS_PLUGIN_SDK_COMPAT_WARNING=1 openclaw gateway run
OPENCLAW_SUPPRESS_EXTENSION_API_WARNING=1 openclaw gateway run
```

Dies ist ein vorübergehender Ausweg, keine dauerhafte Lösung.

## Verwandte Themen

- [Erste Schritte](/de/plugins/building-plugins) - erstellen Sie Ihr erstes Plugin
- [SDK-Übersicht](/de/plugins/sdk-overview) - vollständige Referenz für Unterpfadimporte
- [Kanal-Plugins](/de/plugins/sdk-channel-plugins) - Kanal-Plugins erstellen
- [Provider-Plugins](/de/plugins/sdk-provider-plugins) - Provider-Plugins erstellen
- [Plugin-Interna](/de/plugins/architecture) - detaillierter Einblick in die Architektur
- [Plugin-Manifest](/de/plugins/manifest) - Referenz zum Manifest-Schema
