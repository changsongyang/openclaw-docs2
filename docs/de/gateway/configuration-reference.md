---
read_when:
    - Sie benötigen genaue Konfigurationssemantik oder Standardwerte auf Feldebene
    - Sie validieren Konfigurationsblöcke für Kanäle, Modelle, Gateways oder Tools
summary: Gateway-Konfigurationsreferenz für zentrale OpenClaw-Schlüssel, Standardwerte und Links zu Referenzen der jeweiligen Subsysteme
title: Konfigurationsreferenz
x-i18n:
    generated_at: "2026-07-24T19:40:45Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 7135554fda444fd1b8c072af5768c53a165f7be2dcd12a7909fc7fd4bd864428
    source_path: gateway/configuration-reference.md
    workflow: 16
---

Feldbezogene Referenz für `~/.openclaw/openclaw.json`: Schlüssel, Standardwerte und Links zu ausführlicheren Subsystemseiten. Aufgabenorientierte Anleitungen zur Einrichtung finden Sie unter [Konfiguration](/de/gateway/configuration). Kanal- und Plugin-eigene Befehlskataloge sowie detaillierte Speicher-/QMD-Optionen befinden sich auf den jeweiligen Seiten, nicht hier.

Das Konfigurationsformat ist **JSON5** (Kommentare und abschließende Kommas sind zulässig). Alle Felder sind optional; OpenClaw verwendet bei Auslassung sichere Standardwerte.

Der Code ist maßgeblicher als diese Seite:

- `openclaw config schema` gibt das für die Validierung und die Control UI verwendete aktuelle JSON-Schema aus, einschließlich zusammengeführter Metadaten von gebündelten Plugins und Kanälen.
- Agenten sollten vor dem Bearbeiten der Konfiguration die Tool-Aktion `config.schema.lookup` des Tools `gateway` für genau einen pfadbezogenen Schemaknoten aufrufen.
- `pnpm config:docs:check` / `pnpm config:docs:gen` validieren den Baseline-Hash dieser Dokumentation anhand der aktuellen Schemaoberfläche.

Schema-`uiHints` enthalten außerdem für jeden Pfad einen aufgelösten booleschen Wert `advanced`.
Die Control UI verwendet ihn, um häufig verwendete Felder zuerst anzuzeigen und erweiterte Felder pro
Abschnitt einzuklappen; die Suche umfasst weiterhin beide Ebenen. Ebenenmetadaten dienen nur der Darstellung.
Wenn Sie einen Schlüssel hinzufügen, deklarieren Sie seine Ebene am Blatt oder lassen Sie ihn die Deklaration des nächsten
Vorfahren erben. Ein Pfad ohne deklarierten Vorfahren gilt standardmäßig als erweitert.

Ausführliche Spezialreferenzen:

- [Referenz zur Speicherkonfiguration](/de/reference/memory-config) für `memory.search.*`, `memory.qmd.*`, `memory.citations` und die Dreaming-Konfiguration unter `plugins.entries.memory-core.config.dreaming`.
- [Slash-Befehle](/de/tools/slash-commands) für den aktuellen integrierten und gebündelten Befehlskatalog.
- Die zuständigen Kanal-/Plugin-Seiten für kanalspezifische Befehlsoberflächen.

---

## Kanäle

Kanalspezifische Konfigurationsschlüssel finden Sie unter [Konfiguration – Kanäle](/de/gateway/config-channels): `channels.*` für Slack, Discord, Telegram, WhatsApp, Matrix, iMessage und andere gebündelte Kanäle (Authentifizierung, Zugriffskontrolle, mehrere Konten, Erwähnungsbeschränkung).

## Agentenstandardwerte, Multi-Agenten, Sitzungen und Nachrichten

Unter [Konfiguration – Agenten](/de/gateway/config-agents) finden Sie Informationen zu:

- `agents.defaults.*` (Arbeitsbereich, Modell, Denkprozess, Heartbeat, Speicher, Medien, Skills, Sandbox)
- `multiAgent.*` (Multi-Agenten-Routing und Bindungen)
- `session.*` (Sitzungslebenszyklus, Compaction, Bereinigung)
- `messages.*` (Nachrichtenzustellung, TTS, Markdown-Darstellung)
- `talk.*` (Sprechmodus)
  - `talk.consultThinkingLevel`: Überschreibung der Denkstufe für den vollständigen OpenClaw-Agentenlauf hinter Echtzeitberatungen des Sprechmodus in der Control UI
  - `talk.consultFastMode`: einmalige Überschreibung des Schnellmodus für Echtzeitberatungen des Sprechmodus in der Control UI
  - `talk.speechLocale`: optionale BCP-47-Gebietsschema-ID für die Spracherkennung des Sprechmodus unter Android, iOS und macOS
  - `talk.silenceTimeoutMs`: wenn nicht festgelegt, behält der Sprechmodus vor dem Senden des Transkripts das standardmäßige Pausenfenster der Plattform bei (`700 ms on macOS and Android, 900 ms on iOS`)
  - `talk.realtime.consultRouting`: Gateway-Relay-Fallback für abgeschlossene Echtzeittranskripte des Sprechmodus, die `openclaw_agent_consult` überspringen

## Tools und benutzerdefinierte Provider

Tool-Richtlinien, experimentelle Umschalter, Provider-gestützte Tool-Konfigurationen und die Einrichtung benutzerdefinierter
Provider bzw. Basis-URLs finden Sie unter
[Konfiguration – Tools und benutzerdefinierte Provider](/de/gateway/config-tools).

## Modelle

Provider-Definitionen, Modell-Zulassungslisten und die Einrichtung benutzerdefinierter Provider finden Sie unter
[Konfiguration – Tools und benutzerdefinierte Provider](/de/gateway/config-tools#custom-providers-and-base-urls).
Der Stamm `models` steuert außerdem das globale Verhalten des Modellkatalogs.

```json5
{
  models: {
    // Optional. Standardwert: true. Erfordert bei einer Änderung einen Neustart des Gateways.
    pricing: { enabled: false },
  },
}
```

- `models.mode`: Verhalten des Provider-Katalogs (`merge` oder `replace`).
- `models.providers`: benutzerdefinierte Provider-Zuordnung mit der Provider-ID als Schlüssel.
- `models.providers.*.localService`: optionaler bedarfsgesteuerter Prozessmanager für
  lokale Modellserver. OpenClaw prüft den konfigurierten Integritätsendpunkt, startet bei
  Bedarf die absolute `command`, wartet auf die Betriebsbereitschaft und sendet anschließend die Modellanfrage.
  Siehe [Lokale Modelldienste](/de/gateway/local-model-services).
- `models.pricing.enabled`: steuert die im Hintergrund ausgeführte Initialisierung der Preisdaten, die
  beginnt, nachdem Sidecars und Kanäle den Bereitschaftspfad des Gateways erreicht haben. Bei `false`
  überspringt das Gateway das Abrufen der Preiskataloge von OpenRouter und LiteLLM; konfigurierte
  `models.providers.*.models[].cost`-Werte funktionieren weiterhin für lokale Kostenschätzungen.

## MCP

Von OpenClaw verwaltete MCP-Serverdefinitionen befinden sich unter `mcp.servers` und werden
von eingebettetem OpenClaw sowie anderen Laufzeitadaptern verwendet. Die Befehle `openclaw mcp list`,
`show`, `set` und `unset` verwalten diesen Block, ohne während der
Konfigurationsbearbeitung eine Verbindung zum Zielserver herzustellen.

```json5
{
  mcp: {
    servers: {
      docs: {
        command: "npx",
        args: ["-y", "@modelcontextprotocol/server-fetch"],
      },
      remote: {
        url: "https://example.com/mcp",
        transport: "streamable-http", // streamable-http | sse
        requestTimeoutMs: 20000,
        connectionTimeoutMs: 5000,
        supportsParallelToolCalls: true,
        headers: {
          Authorization: "Bearer ${MCP_REMOTE_TOKEN}",
        },
        auth: "oauth",
        oauth: {
          scope: "docs.read",
        },
        sslVerify: true,
        clientCert: "/path/to/client.crt",
        clientKey: "/path/to/client.key",
        toolFilter: {
          include: ["search_*"],
          exclude: ["admin_*"],
        },
        // Optionale Projektionssteuerung für den Codex-App-Server.
        codex: {
          agents: ["main"],
          defaultToolsApprovalMode: "approve", // auto | prompt | approve
        },
      },
    },
  },
}
```

- `mcp.servers`: benannte stdio- oder Remote-MCP-Serverdefinitionen für Laufzeiten, die
  konfigurierte MCP-Tools bereitstellen.
  Remote-Einträge verwenden `transport: "streamable-http"` oder `transport: "sse"`;
  `type: "http"` ist ein CLI-nativer Alias, den `openclaw mcp set` und
  `openclaw doctor --fix` in das kanonische Feld `transport` normalisieren.
- `mcp.servers.<name>.enabled`: Legen Sie `false` fest, um eine gespeicherte Serverdefinition
  beizubehalten, sie jedoch von der eingebetteten MCP-Erkennung und Tool-Projektion von OpenClaw auszuschließen.
- `mcp.servers.<name>.requestTimeoutMs`: MCP-Anfragezeitlimit pro Server in Millisekunden.
- `mcp.servers.<name>.connectionTimeoutMs`: Verbindungszeitlimit pro Server in Millisekunden.
- `mcp.servers.<name>.supportsParallelToolCalls`: optionaler Parallelitätshinweis für
  Adapter, die entscheiden können, ob sie parallele MCP-Tool-Aufrufe ausführen.
- `mcp.servers.<name>.auth`: Legen Sie für HTTP-MCP-Server, die
  OAuth erfordern, `"oauth"` fest. Führen Sie `openclaw mcp login <name>` aus, um Token im OpenClaw-Zustand zu speichern.
- `mcp.servers.<name>.oauth`: optionale Überschreibungen für OAuth-Bereich, Weiterleitungs-URL und
  Clientmetadaten-URL.
- `mcp.servers.<name>.sslVerify`, `clientCert`, `clientKey`: HTTP-TLS-Steuerungen
  für private Endpunkte und gegenseitiges TLS.
- `mcp.servers.<name>.toolFilter`: optionale Tool-Auswahl pro Server. `include`
  beschränkt die erkannten MCP-Tools auf übereinstimmende Namen; `exclude` blendet übereinstimmende
  Namen aus. Einträge sind exakte MCP-Tool-Namen oder einfache `*`-Glob-Muster. Server mit
  Ressourcen oder Prompts erzeugen außerdem Namen für Hilfstools (`resources_list`,
  `resources_read`, `prompts_list`, `prompts_get`), für die derselbe Filter gilt.
- `mcp.servers.<name>.codex`: optionale Projektionssteuerung für den Codex-App-Server.
  Dieser Block enthält ausschließlich OpenClaw-Metadaten für Codex-App-Server-Threads; er wirkt sich weder
  auf ACP-Sitzungen noch auf die generische Codex-Harness-Konfiguration oder andere Laufzeitadapter aus.
  Ein nicht leeres `codex.agents` beschränkt den Server auf die aufgeführten OpenClaw-Agenten-IDs.
  Leere, aus leeren Einträgen bestehende oder ungültige Listen eingeschränkter Agenten werden von der Konfigurationsvalidierung abgelehnt
  und vom Laufzeitprojektionspfad ausgelassen, anstatt global zu werden.
  `codex.defaultToolsApprovalMode` gibt das native
  `default_tools_approval_mode` von Codex für diesen Server aus. OpenClaw entfernt den Block `codex`,
  bevor die native `mcp_servers`-Konfiguration an Codex übergeben wird. Lassen Sie den Block aus, damit
  der Server für jeden Codex-App-Server-Agenten mit dem standardmäßigen MCP-Genehmigungsverhalten
  von Codex projiziert bleibt.
- Sitzungsbezogene gebündelte MCP-Laufzeiten verwenden eine integrierte Leerlauf-TTL von 10 Minuten.
  Einmalige eingebettete Läufe fordern eine Bereinigung am Laufende an; die TTL dient als Absicherung für langlebige Sitzungen und zukünftige Aufrufer.
- Änderungen unter `mcp.*` werden direkt angewendet, indem zwischengespeicherte sitzungsbezogene MCP-Laufzeiten verworfen werden.
  Bei der nächsten Tool-Erkennung/-Verwendung werden sie aus der neuen Konfiguration neu erstellt, sodass entfernte
  `mcp.servers`-Einträge sofort bereinigt werden, statt auf die Leerlauf-TTL zu warten.
- Die Laufzeiterkennung berücksichtigt außerdem Benachrichtigungen über Änderungen an MCP-Tool-Listen, indem
  der zwischengespeicherte Katalog für diese Sitzung verworfen wird. Server, die Ressourcen oder
  Prompts anbieten, erhalten Hilfstools zum Auflisten/Lesen von Ressourcen und zum Auflisten/Abrufen von
  Prompts. Wiederholte Fehler bei Tool-Aufrufen pausieren den betroffenen Server kurzzeitig, bevor
  ein weiterer Aufruf versucht wird.

Informationen zum Laufzeitverhalten finden Sie unter [MCP](/de/cli/mcp#openclaw-as-an-mcp-client-registry) und
[CLI-Backends](/de/gateway/cli-backends#bundle-mcp-overlays).

## Skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
      allowSymlinkTargets: ["~/Projects/manager/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun
      allowUploadedArchives: false,
    },
    workshop: {
      allowSymlinkTargetWrites: false,
    },
    entries: {
      "image-lab": {
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // oder Klartextzeichenfolge
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`: optionale Zulassungsliste nur für gebündelte Skills (verwaltete Skills und Arbeitsbereich-Skills sind nicht betroffen).
- `load.extraDirs`: zusätzliche gemeinsam genutzte Skill-Stammverzeichnisse (niedrigste Priorität).
- `load.allowSymlinkTargets`: vertrauenswürdige reale Zielstammverzeichnisse, in die Skill-Symlinks
  aufgelöst werden dürfen, wenn sich der Link außerhalb seines konfigurierten Quellstammverzeichnisses befindet.
- `workshop.allowSymlinkTargetWrites`: erlaubt Skill Workshop Apply, über
  bereits vertrauenswürdige Symlink-Ziele zu schreiben (Standardwert: false).
- `install.preferBrew`: Wenn true, werden Homebrew-Installationsprogramme bevorzugt, sofern `brew`
  verfügbar ist, bevor auf andere Installationsprogrammtypen zurückgegriffen wird.
- `install.nodeManager`: bevorzugtes Node-Installationsprogramm für `metadata.openclaw.install`-
  Spezifikationen (`npm` | `pnpm` | `yarn` | `bun`).
- `install.allowUploadedArchives`: erlaubt vertrauenswürdigen `operator.admin`-Gateway-
  Clients, private ZIP-Archive zu installieren, die über `skills.upload.*` bereitgestellt wurden
  (Standardwert: false). Dies aktiviert nur den Pfad für hochgeladene Archive; normale ClawHub-
  Installationen benötigen ihn nicht.
- `entries.<skillKey>.enabled: false` deaktiviert einen Skill, selbst wenn er gebündelt/installiert ist.
- `entries.<skillKey>.apiKey`: Komfortoption für Skills, die eine primäre Umgebungsvariable deklarieren (Klartextzeichenfolge oder SecretRef-Objekt).
- `limits.maxCandidatesPerRoot`, `limits.maxSkillsLoadedPerSource`, `limits.maxSkillsInPrompt`, `limits.maxSkillsPromptChars`, `limits.maxSkillFileBytes`: begrenzen die Skill-Erkennung und den modellseitigen Skills-Prompt.
- Die Autonomie-/Genehmigungseinstellungen von Skill Workshop (`workshop.autonomous.enabled`, `workshop.approvalPolicy`, `workshop.maxPending`, `workshop.maxSkillBytes`) sind unter [Skills-Konfiguration](/de/tools/skills-config) dokumentiert.

---

## Plugins

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-plugin"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        hooks: {
          allowPromptInjection: false,
        },
        config: { provider: "twilio" },
      },
    },
  },
}
```

- Wird aus Paket- oder Bundle-Verzeichnissen unter `~/.openclaw/extensions` und `<workspace>/.openclaw/extensions` sowie aus den in `plugins.load.paths` aufgeführten Dateien oder Verzeichnissen geladen.
- Legen Sie eigenständige Plugin-Dateien in `plugins.load.paths` ab; automatisch erkannte Erweiterungsstammverzeichnisse ignorieren `.js`-, `.mjs`- und `.ts`-Dateien der obersten Ebene, damit Hilfsskripte in diesen Stammverzeichnissen den Start nicht blockieren.
- Die Erkennung akzeptiert native OpenClaw-Plugins sowie kompatible Codex-Bundles und Claude-Bundles, einschließlich manifestloser Claude-Bundles mit Standardlayout.
- **Konfigurationsänderungen erfordern einen Neustart des Gateways.**
- `allow`: optionale Positivliste (nur aufgeführte Plugins werden geladen). `deny` hat Vorrang.
- `plugins.entries.<id>.apiKey`: praktisches Feld für einen API-Schlüssel auf Plugin-Ebene (sofern vom Plugin unterstützt).
- `plugins.entries.<id>.env`: Plugin-spezifische Zuordnung von Umgebungsvariablen.
- `plugins.entries.<id>.hooks.allowPromptInjection`: Wenn `false`, blockiert der Kern Prompt-verändernde Hooks wie `before_prompt_build`. Dies gilt für native Plugin-Hooks und unterstützte, von Bundles bereitgestellte Hook-Verzeichnisse.
- `plugins.entries.<id>.hooks.allowConversationAccess`: Wenn `true`, dürfen vertrauenswürdige, nicht gebündelte Plugins rohe Gesprächsinhalte aus typisierten Hooks wie `llm_input`, `llm_output`, `before_model_resolve`, `before_agent_reply`, `before_agent_run`, `before_agent_finalize` und `agent_end` lesen.
- `plugins.entries.<id>.subagent.allowModelOverride`: Vertraut diesem Plugin ausdrücklich, für Subagent-Hintergrundläufe laufbezogene Überschreibungen von `provider` und `model` anzufordern.
- `plugins.entries.<id>.subagent.allowedModels`: optionale Positivliste kanonischer `provider/model`-Ziele für vertrauenswürdige Subagent-Überschreibungen. Verwenden Sie `"*"` nur, wenn Sie bewusst jedes Modell zulassen möchten.
- `plugins.entries.<id>.llm.allowModelOverride`: Vertraut diesem Plugin ausdrücklich, Modellüberschreibungen für `api.runtime.llm.complete` anzufordern.
- `plugins.entries.<id>.llm.allowedModels`: optionale Positivliste kanonischer `provider/model`-Ziele für vertrauenswürdige Überschreibungen von Plugin-LLM-Vervollständigungen. Verwenden Sie `"*"` nur, wenn Sie bewusst jedes Modell zulassen möchten.
- `plugins.entries.<id>.llm.allowAgentIdOverride`: Vertraut diesem Plugin ausdrücklich, `api.runtime.llm.complete` mit einer nicht standardmäßigen Agent-ID auszuführen.
- `plugins.entries.<id>.config`: vom Plugin definiertes Konfigurationsobjekt (wird anhand des Schemas des nativen OpenClaw-Plugins validiert, sofern verfügbar).
- Konto- und Laufzeiteinstellungen von Kanal-Plugins befinden sich unter `channels.<id>` und sollten durch die `channelConfigs`-Metadaten im Manifest des zuständigen Plugins beschrieben werden, nicht durch ein zentrales OpenClaw-Optionsregister.

### Konfiguration des Codex-Harness-Plugins

Das gebündelte Plugin `codex` verwaltet die Einstellungen des nativen Codex-App-Server-Harness unter
`plugins.entries.codex.config`. Die vollständige Konfigurationsoberfläche finden Sie in der
[Codex-Harness-Referenz](/de/plugins/codex-harness-reference), das Laufzeitmodell unter
[Codex-Harness](/de/plugins/codex-harness).

`codexPlugins` gilt nur für Sitzungen, die das native Codex-Harness auswählen.
Es aktiviert keine Codex-Plugins für OpenClaw-Provider-Läufe, ACP-
Gesprächsbindungen oder andere Harnesses als Codex.

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          codexPlugins: {
            enabled: true,
            allow_all_plugins: true,
            allow_destructive_actions: "auto",
            plugins: {
              "google-calendar": {
                enabled: true,
                marketplaceName: "openai-curated",
                pluginName: "google-calendar",
                allow_destructive_actions: false,
              },
            },
          },
        },
      },
    },
  },
}
```

- `plugins.entries.codex.config.codexPlugins.enabled`: aktiviert native Codex-
  Plugin-/App-Unterstützung für das Codex-Harness. Standard: `false`.
- `plugins.entries.codex.config.codexPlugins.allow_all_plugins`: stellt in
  jedem neuen nativen Codex-Thread alle derzeit zugänglichen Apps bereit, die mit dem authentifizierten Codex-Konto verbunden sind. Standard: `false`.
- `plugins.entries.codex.config.codexPlugins.allow_destructive_actions`:
  standardmäßige Richtlinie für destruktive Aktionen bei konfigurierten Plugin-App-Abfragen.
  Verwenden Sie `true`, um sichere Codex-Genehmigungsschemas ohne Nachfrage zu akzeptieren, `false`,
  um sie abzulehnen, `"auto"`, um von Codex benötigte Genehmigungen über OpenClaw-
  Plugin-Genehmigungen zu leiten, oder `"ask"`, um bei jeder schreibenden/destruktiven
  Plugin-Aktion ohne dauerhafte Genehmigung nachzufragen. Der Modus `"ask"` löscht dauerhafte Codex-
  Genehmigungsüberschreibungen pro Tool für die betroffene App und wählt den menschlichen
  Genehmigungsprüfer für diese App aus, bevor der Codex-Thread beginnt.
  Standard: `true`.
- `plugins.entries.codex.config.codexPlugins.plugins.<key>.enabled`: aktiviert einen
  konfigurierten Plugin-Eintrag, wenn das globale `codexPlugins.enabled` ebenfalls wahr ist.
  Standard: `true` für explizite Einträge.
- `plugins.entries.codex.config.codexPlugins.plugins.<key>.marketplaceName`:
  stabile Marketplace-Identität, zusammen mit `pluginName` für jeden aufgelösten
  Eintrag erforderlich. Unterstützt `"openai-curated"` und `"workspace-directory"`. Einträge,
  denen eines der Identitätsfelder fehlt, werden ignoriert.
- `plugins.entries.codex.config.codexPlugins.plugins.<key>.pluginName`: stabile
  Codex-Plugin-Identität, zusammen mit `marketplaceName` erforderlich. Ein
  `workspace-directory`-Eintrag muss den exakten, Marketplace-qualifizierten
  `summary.id` verwenden, der von `plugin/list` zurückgegeben wird, zum Beispiel
  `"example-plugin@workspace-directory"`.
- `plugins.entries.codex.config.codexPlugins.plugins.<key>.allow_destructive_actions`:
  Plugin-spezifische Überschreibung für destruktive Aktionen. Wenn sie fehlt, wird der globale
  Wert `allow_destructive_actions` verwendet. Der Plugin-spezifische Wert akzeptiert dieselben
  Richtlinien `true`, `false`, `"auto"` oder `"ask"`.

Jede zugelassene Plugin-App, die `"ask"` verwendet, leitet die Genehmigungsanfragen dieser App
an den menschlichen Prüfer weiter. Andere Apps und Genehmigungen für Threads, die nicht zu Apps gehören, behalten ihren
konfigurierten Prüfer, sodass gemischte Plugin-Richtlinien das Verhalten von `"ask"` nicht übernehmen.

`codexPlugins.enabled` ist die globale Aktivierungsdirektive. Durch Migration geschriebene explizite Plugin-
Einträge bilden die dauerhafte kuratierte Eignungsmenge für Installation und Reparatur.
Manuell konfigurierte `workspace-directory`-Einträge müssen bereits
installiert und aktiviert sein, und die zugehörigen Apps müssen zugänglich sein; OpenClaw
installiert oder authentifiziert sie nicht. Wenn Codex die explizite Anfrage an den Arbeitsbereichskatalog
ablehnt, werden aktivierte Arbeitsbereichseinträge geschlossen abgewiesen und melden
`marketplace_missing`, während kuratierte Einträge aus dem Standardkatalog weiterhin
verfügbar bleiben. `plugins["*"]` wird nicht unterstützt, es gibt keinen `install`-Schalter, und
lokale `marketplacePath`-Werte sind bewusst keine Konfigurationsfelder, da sie
hostspezifisch sind. Anforderungen an App-Server-Version und
Bereitschaft finden Sie unter [Native Codex-Plugins](/de/plugins/codex-native-plugins).

Bereitschaftsprüfungen für `app/list` werden eine Stunde lang zwischengespeichert und bei
Veraltung asynchron aktualisiert. Die App-Konfiguration des Codex-Threads wird beim Aufbau der Codex-Harness-
Sitzung berechnet, nicht bei jedem Turn; verwenden Sie nach Änderungen an der nativen Plugin-Konfiguration `/new`, `/reset` oder einen Neustart des
Gateways.

`codexPlugins.allow_all_plugins` übernimmt jede derzeit zugängliche Konto-
App als Momentaufnahme in jeden neuen nativen Codex-Thread. Es installiert keine Plugins oder Apps, und
nicht zugängliche Apps bleiben ausgeschlossen. Konto-Apps verwenden die globale
Richtlinie `codexPlugins.allow_destructive_actions`. Explizite Plugin-Einträge haben
Vorrang, wenn dieselbe App über beide Pfade vorhanden ist. Wenn `app/list` nicht gelesen werden kann,
wird die kontoweite Bereitstellung geschlossen abgewiesen.

- `plugins.entries.firecrawl.config.webFetch`: Einstellungen des Firecrawl-Webabruf-Providers.
  - `apiKey`: Optionaler Firecrawl-API-Schlüssel für höhere Grenzwerte (akzeptiert SecretRef). Fällt auf die Umgebungsvariable `plugins.entries.firecrawl.config.webSearch.apiKey` oder `FIRECRAWL_API_KEY` zurück.
  - `baseUrl`: Basis-URL der Firecrawl-API (Standard: `https://api.firecrawl.dev`; selbst gehostete Überschreibungen müssen auf private/interne Endpunkte verweisen).
  - `onlyMainContent`: extrahiert nur den Hauptinhalt von Seiten (Standard: `true`).
  - `maxAgeMs`: maximales Cache-Alter in Millisekunden (Standard: `172800000` / 2 Tage).
  - `timeoutSeconds`: Zeitüberschreitung für Scraping-Anfragen in Sekunden (Standard: `60`).
- `plugins.entries.xai.config.xSearch`: Einstellungen für xAI X Search (Grok-Websuche).
  - `enabled`: aktiviert den X-Search-Provider.
  - `model`: für die Suche zu verwendendes Grok-Modell (z. B. `"grok-4.3"`).
- `plugins.entries.memory-core.config.dreaming`: Einstellungen für Memory-Dreaming. Phasen und Schwellenwerte finden Sie unter [Dreaming](/de/concepts/dreaming).
  - `enabled`: übergeordneter Dreaming-Schalter (Standard: `false`).
  - `frequency`: Cron-Intervall für jeden vollständigen Dreaming-Durchlauf (standardmäßig `"0 3 * * *"`).
  - `model`: optionale Modellüberschreibung für den Dream-Diary-Subagent. Erfordert `plugins.entries.memory-core.subagent.allowModelOverride: true`; kombinieren Sie dies mit `allowedModels`, um Ziele einzuschränken. Fehler wegen nicht verfügbarer Modelle führen zu einem erneuten Versuch mit dem Standardsitzungsmodell; Fehler bei Vertrauen oder Positivliste führen nicht stillschweigend zu einem Rückfall.
  - Phasenrichtlinien und Schwellenwerte sind Implementierungsdetails (keine benutzerseitigen Konfigurationsschlüssel).
- Die vollständige Memory-Konfiguration finden Sie in der [Referenz zur Memory-Konfiguration](/de/reference/memory-config):
  - `memory.search.*`
  - `agents.entries.*.memory.search.*` für agentenspezifische Überschreibungen
  - `memory.backend`
  - `memory.citations`
  - `memory.qmd.*`
  - `plugins.entries.memory-core.config.dreaming`
- Aktivierte Claude-Bundle-Plugins können außerdem eingebettete OpenClaw-Standardwerte aus `settings.json` beitragen; OpenClaw wendet diese als bereinigte Agent-Einstellungen an, nicht als rohe Patches der OpenClaw-Konfiguration.
- `plugins.slots.memory`: wählt die ID des aktiven Memory-Plugins oder `"none"`, um Memory-Plugins zu deaktivieren.
- `plugins.slots.contextEngine`: wählt die ID des aktiven Kontext-Engine-Plugins; standardmäßig `"legacy"`, sofern Sie keine andere Engine installieren und auswählen.

Siehe [Plugins](/de/tools/plugin).

---

## Browser

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "user",
    ssrfPolicy: {
      // dangerouslyAllowPrivateNetwork: true, // nur für vertrauenswürdigen Zugriff auf private Netzwerke aktivieren
      // allowPrivateNetwork: true, // veralteter Alias
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    tabCleanup: {
      enabled: true,
      idleMinutes: 120,
      maxTabsPerSession: 8,
      sweepMinutes: 5,
    },
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: {
        cdpPort: 18801,
        color: "#0066CC",
        executablePath: "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome",
      },
      user: { driver: "existing-session", attachOnly: true, color: "#00AA00" },
      brave: {
        driver: "existing-session",
        attachOnly: true,
        userDataDir: "~/Library/Application Support/BraveSoftware/Brave-Browser",
        color: "#FB542B",
      },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // extraArgs: [],
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false` deaktiviert `act:evaluate` und `wait --fn`.
- `tabCleanup` steuert die nach bestem Bemühen regelmäßig ausgeführte Bereinigung nachverfolgter Tabs des primären Agenten
  nach einer Leerlaufzeit oder wenn eine Sitzung ihr Limit überschreitet. Die Nachverfolgung gilt nur
  für Tabs, die vom Browser-Tool `action: "open"` erstellt wurden; vom Benutzer geöffnete Tabs oder
  Tabs mit unbekannter Eigentümerschaft werden niemals übernommen. Das Deaktivieren von `tabCleanup` deaktiviert nicht die explizite Bereinigung des Sitzungslebenszyklus.
- Hostlokale Öffnungen mit einem stabilen nativen CDP-Ziel und einer stabilen Browseridentität werden
  im gemeinsamen SQLite-Zustand gespeichert und bleiben über Gateway-Neustarts hinweg für
  `/new` und die Bereinigung des Sitzungslebenszyklus berechtigt. Native, für Tools bestimmte CDP-Ziele
  bleiben nach einem Neustart ebenfalls für die Leerlauf- und Limitbereinigung berechtigt. Chrome MCP verwendet
  prozesslokale Ziel-Handles, daher warten bestehende Sitzungsdatensätze nach einem Kaltstart auf die
  Lebenszyklusbereinigung, statt eine Leerlaufbereinigung bei nicht zuordenbarer
  Aktivität nach dem Neustart zu riskieren. OpenClaw überprüft das Profil und die Browserinstanz,
  bevor sie geschlossen werden. Die automatische Verbindung von Chrome MCP, eine fehlende
  Browseridentität `/json/version` und nicht aufgelöste native Ziele bleiben vollständig prozesslokal,
  sodass sie nach einem Neustart nicht automatisch geschlossen werden. Ältere, nicht nachverfolgte Tabs müssen
  manuell geschlossen werden. Vorübergehende Fehler bleiben für einen späteren erneuten Versuch ausstehend. Siehe
  [Eigentümerschaft bei der Tab-Bereinigung](/de/tools/browser#tab-cleanup-ownership).
- `ssrfPolicy.dangerouslyAllowPrivateNetwork` ist im nicht gesetzten Zustand deaktiviert, sodass die Browsernavigation standardmäßig strikt bleibt.
- Setzen Sie `ssrfPolicy.dangerouslyAllowPrivateNetwork: true` nur, wenn Sie der Browsernavigation in privaten Netzwerken bewusst vertrauen.
- Im strikten Modus unterliegen Endpunkte entfernter CDP-Profile (`profiles.*.cdpUrl`) bei Erreichbarkeits- und Erkennungsprüfungen derselben Blockierung privater Netzwerke.
- `ssrfPolicy.allowPrivateNetwork` wird weiterhin als Legacy-Alias unterstützt.
- Verwenden Sie im strikten Modus `ssrfPolicy.hostnameAllowlist` und `ssrfPolicy.allowedHostnames` für explizite Ausnahmen.
- Entfernte Profile können nur angehängt werden (Start/Stopp/Zurücksetzen deaktiviert).
- `profiles.*.cdpUrl` akzeptiert `http://`, `https://`, `ws://` und `wss://`.
  Verwenden Sie HTTP(S), wenn OpenClaw `/json/version` ermitteln soll; verwenden Sie WS(S),
  wenn Ihr Provider Ihnen eine direkte DevTools-WebSocket-URL bereitstellt.
- Wenn ein extern verwalteter CDP-Dienst über Loopback erreichbar ist, setzen Sie für dieses
  Profil `attachOnly: true`; andernfalls behandelt OpenClaw den Loopback-Port als
  lokal verwaltetes Browserprofil und meldet möglicherweise Fehler zur lokalen Port-Eigentümerschaft.
- `existing-session`-Profile verwenden Chrome MCP anstelle von CDP und können auf
  dem ausgewählten Host oder über eine verbundene Browser-Node angehängt werden.
- `existing-session`-Profile können `userDataDir` festlegen, um ein bestimmtes
  Chromium-basiertes Browserprofil wie Brave oder Edge als Ziel zu verwenden.
- `existing-session`-Profile können `cdpUrl` festlegen, wenn Chrome bereits
  hinter einem DevTools-HTTP(S)-Erkennungsendpunkt oder einem direkten WS(S)-Endpunkt ausgeführt wird. In diesem
  Modus übergibt OpenClaw den Endpunkt an Chrome MCP, statt die automatische Verbindung zu verwenden;
  `userDataDir` wird für Chrome-MCP-Startargumente ignoriert.
- `existing-session`-Profile behalten die aktuellen Beschränkungen der Chrome-MCP-Route bei:
  Snapshot-/Referenz-gesteuerte Aktionen statt Zielauswahl über CSS-Selektoren, Upload-Hooks
  für eine einzelne Datei, keine Überschreibungen des Dialog-Zeitlimits, kein `wait --load networkidle` und keine
  `responsebody`, kein PDF-Export, kein Abfangen von Downloads und keine Stapelaktionen.
- Lokal verwaltete `openclaw`-Profile weisen `cdpPort` und `cdpUrl` automatisch zu; legen Sie
  `cdpUrl` nur für entfernte CDP-Profile oder beim Anhängen an den Endpunkt einer bestehenden Sitzung explizit fest.
- Lokal verwaltete Profile können `executablePath` festlegen, um das globale
  `browser.executablePath` für dieses Profil zu überschreiben. Verwenden Sie dies, um ein Profil in
  Chrome und ein anderes in Brave auszuführen.
- Reihenfolge der automatischen Erkennung: Standardbrowser, falls Chromium-basiert → Chrome → Brave → Edge → Chromium → Chrome Canary.
- `browser.executablePath` und `browser.profiles.<name>.executablePath`
  akzeptieren beide `~` und `~/...` für das Home-Verzeichnis Ihres Betriebssystems vor dem Chromium-Start.
  Profilbezogenes `userDataDir` für `existing-session`-Profile wird ebenfalls per Tilde erweitert.
- Steuerungsdienst: nur Loopback (Port wird von `gateway.port` abgeleitet, Standardwert `18791`).
- `extraArgs` hängt zusätzliche Start-Flags an den lokalen Chromium-Start an (zum Beispiel
  `--disable-gpu`, Fenstergröße oder Debug-Flags).

---

## Benutzeroberfläche

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // Emoji, kurzer Text, Bild-URL oder Daten-URI
    },
    prefs: {
      theme: "claw", // claw | knot | dash | custom
      themeMode: "system", // light | dark | system
      locale: "en",
      chatShowThinking: true,
      chatShowToolCalls: true,
      chatPersistCommentary: true, // Kommentare nach Ausführungen in der Control UI beibehalten; sie werden nicht an Kanäle übermittelt
      chatSendShortcut: "enter", // enter | modifier-enter
      chatFollowUpMode: "steer", // steer | queue; weglassen, um den Warteschlangenmodus des Servers zu verwenden
      showAdvancedSettings: false, // Jede Gruppe „Advanced“ in den Einstellungen aufklappen
    },
  },
}
```

- `seamColor`: Akzentfarbe für die UI-Elemente nativer Apps (Farbton der Talk-Mode-Blase usw.).
- `assistant`: Überschreibung der Identität in der Control UI. Fällt auf die Identität des aktiven Agenten zurück.
- `prefs`: geräteübergreifende Bedienereinstellungen. Dies ist der kanonische Speicherort, damit Agenten
  sie über die Genehmigungsschranke ändern können und alle Control-UI-Clients
  synchron bleiben; Browser spiegeln die Werte für einen sofortigen Start in den lokalen Speicher und behalten
  eine gerätelokale Kopie, wenn sie die Konfiguration nicht schreiben können (Betrachterbereich, offline).
  `chatPersistCommentary` verwendet standardmäßig `true`. Wenn Sie es auf `false` setzen, bleiben Live-
  Kommentare während einer Ausführung sichtbar, werden jedoch bei deren Abschluss entfernt, und neue
  Codex-Kommentare gelangen nicht in die dauerhafte Transkriptspiegelung. Die Übermittlung an
  Nachrichtenkanäle bleibt davon getrennt und unverändert.
  `showAdvancedSettings` verwendet standardmäßig `false`; die Einstellungssuche kann vorübergehend
  eine passende erweiterte Gruppe öffnen, ohne diese Einstellung zu ändern.
  Rein darstellungsbezogene Einstellungen wie Textskalierung, Chatbreite und Live-
  Aktivität in der Seitenleiste bleiben browserlokal und werden unter Settings konfiguriert.
  Verbundene Clients wenden serverseitige Änderungen live an: Das Gateway sendet nach jedem
  persistenten Schreiben der Konfiguration ein ausschließlich den Hash enthaltendes `config.changed`-Ereignis, und
  die Clients aktualisieren ihren Snapshot (wird übersprungen, solange ein lokaler Einstellungsentwurf
  ungespeicherte Änderungen enthält). Clients gleichen ihren Zustand beim erneuten Verbinden ab.

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // none | token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // oder OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // für mode=trusted-proxy; siehe /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // toolTitles: false, // optionale KI-Zwecktitel für Tool-Aufrufe (verbraucht Tokens des Utility-Modells)
      // embedSandbox: "scripts", // strict | scripts | trusted
      // allowExternalEmbedUrls: false, // gefährlich: absolute externe HTTP(S)-Einbettungs-URLs zulassen
      // allowedOrigins: ["https://control.example.com"], // für eine Control UI außerhalb von Loopback erforderlich
      // dangerouslyAllowHostHeaderOriginFallback: false, // gefährlicher Fallback-Modus für den Ursprung über den Host-Header
    },
    terminal: {
      enabled: false,
      // shell: "/bin/zsh",
    },
    remote: {
      url: "ws://127.0.0.1:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    // Optional. Standardwert false.
    allowRealIpFallback: false,
    nodes: {
      pairing: {
        // Optional. Standardmäßig nicht gesetzt/deaktiviert.
        autoApproveCidrs: ["192.168.1.0/24", "fd00:1234:5678::/64"],
        // SSH-verifizierte automatische Genehmigung. Standardmäßig aktiviert (true).
        // Auf false setzen, um nur die SSH-Verifizierung zu deaktivieren; dies wirkt sich nicht
        // auf autoApproveCidrs oben aus. Für eine ausschließlich manuelle Node-Kopplung false setzen UND
        // autoApproveCidrs nicht setzen. Zur Feinabstimmung ein Objekt übergeben: { user, identity,
        // timeoutMs, cidrs }.
        sshVerify: true,
      },
      commands: {
        allow: ["canvas.navigate"],
        deny: ["system.run"],
      },
    },
    tools: {
      // Zusätzliche HTTP-Sperren für /tools/invoke
      deny: ["browser"],
      // Tools für Eigentümer-/Administrator-Aufrufer aus der standardmäßigen HTTP-Sperrliste entfernen
      allow: ["gateway"],
    },
    push: {
      apns: {
        relay: {
          baseUrl: "https://relay.example.com",
          timeoutMs: 10000,
        },
      },
    },
  },
}
```

<Accordion title="Details zu Gateway-Feldern">

- `mode`: `local` (Gateway ausführen) oder `remote` (mit Remote-Gateway verbinden). Der Gateway verweigert den Start, sofern nicht `local`.
- `port`: einzelner Multiplex-Port für WS + HTTP. Rangfolge: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (Standard), `lan` (`0.0.0.0`), `tailnet` (Tailscale-IPv4, falls verfügbar, andernfalls Loopback) oder `custom` (eine IPv4-Adresse). Eine aufgelöste `tailnet`-Adresse und jede `custom`-Adresse außer `127.0.0.1` oder `0.0.0.0` erfordern für Clients auf demselben Host `127.0.0.1` auf demselben Port; der Start schlägt fehl, wenn einer der Listener nicht gebunden werden kann. Die Erreichbarkeit außerhalb von Loopback bleibt auf die ausgewählte Schnittstelle beschränkt.
- **Veraltete Bind-Aliasse**: Verwenden Sie Bind-Moduswerte in `gateway.bind` (`auto`, `loopback`, `lan`, `tailnet`, `custom`) und keine Host-Aliasse (`0.0.0.0`, `127.0.0.1`, `localhost`, `::`, `::1`).
- **Docker-Hinweis**: Die standardmäßige `loopback`-Bindung lauscht innerhalb des Containers auf `127.0.0.1`. Bei Docker-Bridge-Netzwerken (`-p 18789:18789`) trifft der Datenverkehr auf `eth0` ein, sodass der Gateway nicht erreichbar ist. Verwenden Sie `--network host` oder legen Sie `bind: "lan"` (oder `bind: "custom"` mit `customBindHost: "0.0.0.0"`) fest, damit auf allen Schnittstellen gelauscht wird.
- **Authentifizierung**: standardmäßig erforderlich. Bindungen außerhalb von Loopback erfordern eine Gateway-Authentifizierung. In der Praxis bedeutet dies ein gemeinsames Token/Passwort oder einen identitätsbewussten Reverse-Proxy mit `gateway.auth.mode: "trusted-proxy"`. Der Einrichtungsassistent generiert standardmäßig ein Token.
- Wenn sowohl `gateway.auth.token` als auch `gateway.auth.password` konfiguriert sind (einschließlich SecretRefs), legen Sie `gateway.auth.mode` ausdrücklich auf `token` oder `password` fest. Start sowie Installation und Reparatur des Dienstes schlagen fehl, wenn beide konfiguriert sind und kein Modus festgelegt ist.
- `gateway.auth.mode: "none"`: expliziter Modus ohne Authentifizierung. Verwenden Sie ihn ausschließlich für vertrauenswürdige lokale Loopback-Konfigurationen; er wird in den Einrichtungsabfragen bewusst nicht angeboten.
- `gateway.auth.mode: "trusted-proxy"`: Delegiert die Browser-/Benutzerauthentifizierung an einen identitätsbewussten Reverse-Proxy und vertraut Identitäts-Headern von `gateway.trustedProxies` (siehe [Authentifizierung über einen vertrauenswürdigen Proxy](/de/gateway/trusted-proxy-auth)). Dieser Modus erwartet standardmäßig eine Proxy-Quelle **außerhalb von Loopback**; Loopback-Reverse-Proxys auf demselben Host erfordern ausdrücklich `gateway.auth.trustedProxy.allowLoopback = true`. Interne Aufrufer auf demselben Host können `gateway.auth.password` als lokalen direkten Fallback verwenden; `gateway.auth.token` bleibt mit dem Modus für vertrauenswürdige Proxys gegenseitig ausschließend.
- `gateway.auth.allowTailscale`: Wenn `true`, können Identitäts-Header von Tailscale Serve die Authentifizierung der Control UI/WebSocket-Verbindung erfüllen (über `tailscale whois` verifiziert). HTTP-API-Endpunkte verwenden diese Tailscale-Header-Authentifizierung **nicht**; stattdessen folgen sie dem normalen HTTP-Authentifizierungsmodus des Gateways. Dieser tokenlose Ablauf setzt voraus, dass der Gateway-Host vertrauenswürdig ist. Standardmäßig `true`, wenn `tailscale.mode = "serve"`.
- `gateway.auth.rateLimit`: optionale Begrenzung fehlgeschlagener Authentifizierungsversuche. Gilt pro Client-IP und Authentifizierungsbereich (gemeinsames Geheimnis und Geräte-Token werden unabhängig voneinander erfasst). Blockierte Versuche geben `429` + `Retry-After` zurück.
  - Im asynchronen Tailscale-Serve-Pfad der Control UI werden fehlgeschlagene Versuche für denselben `{scope, clientIp}` vor dem Schreiben des Fehlschlags serialisiert. Gleichzeitige ungültige Versuche desselben Clients können die Begrenzung daher bereits bei der zweiten Anfrage auslösen, statt beide aufgrund eines Wettlaufs lediglich als Nichtübereinstimmungen passieren zu lassen.
  - `gateway.auth.rateLimit.exemptLoopback` ist standardmäßig `true`; legen Sie `false` fest, wenn auch der Localhost-Datenverkehr bewusst begrenzt werden soll (für Testkonfigurationen oder strikte Proxy-Bereitstellungen).
- WS-Authentifizierungsversuche mit Browser-Ursprung werden immer gedrosselt, wobei die Loopback-Ausnahme deaktiviert ist (mehrschichtiger Schutz gegen browserbasierte Brute-Force-Angriffe auf Localhost).
- Bei Loopback werden diese Sperren für Browser-Ursprünge pro normalisiertem `Origin`-Wert
  getrennt, sodass wiederholte Fehlschläge von einem Localhost-Ursprung nicht automatisch
  einen anderen Ursprung sperren.
- `tailscale.mode`: `serve` (nur Tailnet, Loopback-Bindung) oder `funnel` (öffentlich, erfordert Authentifizierung).
- `tailscale.serviceName`: optionaler Tailscale-Dienstname für den Serve-Modus, zum
  Beispiel `svc:openclaw`. Wenn festgelegt, übergibt OpenClaw ihn an `tailscale serve
--service`, sodass die Control UI über einen benannten Dienst statt
  über den Geräte-Hostnamen bereitgestellt werden kann. Der Wert muss dem `svc:<dns-label>`-Format
  für Tailscale-Dienstnamen entsprechen; beim Start wird die abgeleitete Dienst-URL ausgegeben.
- `tailscale.preserveFunnel`: Wenn `true` und `tailscale.mode = "serve"`, prüft OpenClaw
  vor dem erneuten Anwenden von Serve beim Start `tailscale funnel status` und überspringt
  dies, wenn eine extern konfigurierte Funnel-Route den Gateway-Port bereits abdeckt.
  Standardwert: `false`.
- `controlUi.allowedOrigins`: explizite Zulassungsliste für Browser-Ursprünge bei Gateway-WebSocket-Verbindungen. Für öffentliche Browser-Ursprünge außerhalb von Loopback erforderlich. Private UI-Aufrufe desselben Ursprungs im LAN/Tailnet von Loopback-, RFC1918-/Link-Local-, `.local`-, `.ts.net`- oder Tailscale-CGNAT-Hosts werden akzeptiert, ohne den Host-Header-Fallback zu aktivieren.
- `controlUi.toolTitles`: Aktiviert KI-generierte Zweckbezeichnungen für Tool-Aufrufe im Chat der Control UI. Standard: `false` (die Tool-Darstellung bleibt vollständig deterministisch und erfolgt ohne Modellaufrufe im Hintergrund). Wenn aktiviert, versieht die Methode `chat.toolTitles` komplexe Aufrufe über das standardmäßige Utility-Modell-Routing mit Bezeichnungen – über `utilityModel` des Agenten (eine Betreiberentscheidung, durch die begrenzte Tool-Argumente an den ausgewählten Provider gesendet werden können, wie bei jeder Utility-Aufgabe) oder den vom Sitzungs-Provider deklarierten Standard für kleine Modelle (OpenAI → `gpt-5.6-luna`, Anthropic → `claude-haiku-4-5`) – und speichert die Ergebnisse in der agentenspezifischen Zustandsdatenbank zwischen, sodass wiederholte Ansichten nie erneut abgerechnet werden. `utilityModel: \"\"` deaktiviert Bezeichnungen wie bei jeder anderen Utility-Aufgabe; Bezeichnungen greifen nie auf das primäre Modell zurück.
- `controlUi.dangerouslyAllowHostHeaderOriginFallback`: gefährlicher Modus, der den Host-Header-Ursprungs-Fallback für Bereitstellungen aktiviert, die bewusst auf einer Host-Header-Ursprungsrichtlinie basieren.
- `terminal.enabled`: Aktiviert das auf Administratoren beschränkte Betreiberterminal. Standard: `false`. Das Terminal startet ein Host-PTY im ausgewählten Arbeitsbereich des Agenten, übernimmt die Umgebung des Gateway-Prozesses und wird für Agenten mit `sandbox.mode: "all"` verweigert. Aktivieren Sie es ausschließlich für vertrauenswürdige Betreiberbereitstellungen; eine Änderung startet den Gateway neu und aktualisiert die Content-Security-Policy der Control UI.
- `terminal.shell`: optionale ausführbare Shell-Datei. Wenn nicht festgelegt, verwendet OpenClaw unter Unix `$SHELL` und unter Windows `%ComSpec%`.
- `terminal.detachedSessionTimeoutSeconds`: Gibt an, wie lange eine Terminalsitzung nach dem Abbruch ihrer Verbindung (Neuladen der Seite, Ruhezustand des Laptops) bestehen bleibt und über `terminal.attach` wieder verbunden werden kann, wobei die jüngsten Ausgaben erneut angezeigt werden. Standard: `300`. Legen Sie `0` fest, um Sitzungen sofort zu beenden, sobald ihre Verbindung abbricht. Getrennte Sitzungen führen ihre Befehle weiter aus; verkürzen Sie diesen Zeitraum daher auf gemeinsam genutzten oder öffentlich erreichbaren Hosts.
- `remote.transport`: `ssh` (Standard) oder `direct` (ws/wss). Für `direct` muss `remote.url` bei öffentlichen Hosts `wss://` sein; unverschlüsseltes `ws://` wird ausschließlich für Loopback-, LAN-, Link-Local-, `.local`-, `.ts.net`- und Tailscale-CGNAT-Hosts akzeptiert.
- `remote.remotePort`: Gateway-Port auf dem Remote-SSH-Host. Standardmäßig `18789`; verwenden Sie dies, wenn sich der lokale Tunnelport vom Remote-Gateway-Port unterscheidet.
- `remote.tlsFingerprint`: erwarteter SHA-256-Zertifikat-Fingerabdruck für einen Remote-Gateway vom Typ `wss://`. Die macOS-App wendet ihn sowohl auf Betreiber-/Steuerungsverbindungen als auch auf Verbindungen zu Begleit-Nodes an. Ohne expliziten Wert zeichnet macOS eine PIN bei der ersten Verwendung erst auf, nachdem die normale Systemvertrauensprüfung erfolgreich war.
- `remote.sshHostKeyPolicy`: Hostschlüsselrichtlinie des macOS-SSH-Tunnels. `strict` ist der Standard und erfordert einen bereits vertrauenswürdigen Schlüssel. `openssh` ist eine explizite Zustimmung zur effektiven OpenSSH-Konfiguration für verwaltete Aliasse; prüfen Sie vor der Verwendung die entsprechenden SSH-Einstellungen des Benutzers und des Systems. Die macOS-App und `configure-remote` setzen diese Richtlinie beim Wechseln von Zielen auf `strict` zurück, sofern nicht erneut ausdrücklich zugestimmt wird.
- `gateway.remote.token` / `.password` sind Anmeldedatenfelder für Remote-Clients. Sie konfigurieren die Gateway-Authentifizierung nicht eigenständig.
- `gateway.push.apns.relay.baseUrl`: HTTPS-Basis-URL für das externe APNs-Relay, das verwendet wird, nachdem Relay-gestützte iOS-Builds Registrierungen am Gateway veröffentlicht haben. Öffentliche App-Store-Builds verwenden das gehostete OpenClaw-Relay. Benutzerdefinierte Relay-URLs müssen zu einem bewusst separaten iOS-Build-/Bereitstellungspfad passen, dessen Relay-URL auf dieses Relay verweist.
- `gateway.push.apns.relay.timeoutMs`: Zeitüberschreitung für das Senden vom Gateway zum Relay in Millisekunden. Standardmäßig `10000`.
- Relay-gestützte Registrierungen werden an eine bestimmte Gateway-Identität delegiert. Die gekoppelte iOS-App ruft `gateway.identity.get` ab, fügt diese Identität der Relay-Registrierung hinzu und leitet eine auf die Registrierung beschränkte Sendeberechtigung an den Gateway weiter. Ein anderer Gateway kann diese gespeicherte Registrierung nicht wiederverwenden.
- `OPENCLAW_APNS_RELAY_BASE_URL` / `OPENCLAW_APNS_RELAY_TIMEOUT_MS`: temporäre Umgebungsüberschreibungen für die obige Relay-Konfiguration.
- `OPENCLAW_APNS_RELAY_ALLOW_HTTP=true`: ausschließlich für die Entwicklung vorgesehene Ausnahmemöglichkeit für Loopback-HTTP-Relay-URLs. Produktions-Relay-URLs sollten HTTPS verwenden.
- `OPENCLAW_HANDSHAKE_TIMEOUT_MS`: optionale Umgebungsüberschreibung für das integrierte Zeitlimit des Gateway-WebSocket-Handshakes vor der Authentifizierung.
- `channels.<provider>.healthMonitor.enabled`: kanalspezifische Deaktivierung von Neustarts durch die Zustandsüberwachung, während die globale Überwachung aktiviert bleibt.
- `channels.<provider>.accounts.<accountId>.healthMonitor.enabled`: kontospezifische Überschreibung für Kanäle mit mehreren Konten. Wenn festgelegt, hat sie Vorrang vor der Überschreibung auf Kanalebene.
- Lokale Gateway-Aufrufpfade können `gateway.remote.*` nur als Fallback verwenden, wenn `gateway.auth.*` nicht festgelegt ist.
- Wenn `gateway.auth.token` / `gateway.auth.password` explizit über SecretRef konfiguriert und nicht aufgelöst ist, schlägt die Auflösung sicher geschlossen fehl (keine Verschleierung durch einen Remote-Fallback).
- `trustedProxies`: IP-Adressen von Reverse-Proxys, die TLS terminieren oder weitergeleitete Client-Header einfügen. Führen Sie ausschließlich Proxys auf, die Sie kontrollieren. Loopback-Einträge bleiben für Proxy-/Lokalerkennungskonfigurationen auf demselben Host gültig (beispielsweise Tailscale Serve oder ein lokaler Reverse-Proxy), sie machen Loopback-Anfragen jedoch **nicht** für `gateway.auth.mode: "trusted-proxy"` berechtigt.
- `allowRealIpFallback`: Wenn `true`, akzeptiert der Gateway `X-Real-IP`, falls `X-Forwarded-For` fehlt. Standardmäßig `false` für ein sicher geschlossenes Verhalten.
- `gateway.nodes.pairing.autoApproveCidrs`: optionale CIDR-/IP-Zulassungsliste für die automatische Genehmigung der erstmaligen Kopplung eines Node-Geräts ohne angeforderte Geltungsbereiche. Wenn nicht festgelegt, ist sie deaktiviert. Dadurch wird die Kopplung von Betreiber/Browser/Control UI/WebChat nicht automatisch genehmigt; ebenso wenig werden Upgrades von Rolle, Geltungsbereich, Metadaten oder öffentlichem Schlüssel automatisch genehmigt.
- `gateway.nodes.pairing.sshVerify`: SSH-verifizierte automatische Genehmigung der erstmaligen Kopplung eines Node-Geräts (Standard: aktiviert). Der Gateway stellt per SSH eine Rückverbindung zum Kopplungs-Host her (BatchMode, strikte Hostschlüssel) und genehmigt ausschließlich bei exakter Übereinstimmung des `openclaw node identity`-Geräteschlüssels. Es gilt dieselbe Mindestberechtigung wie bei `autoApproveCidrs`; Prüfungen sind auf private/CGNAT-Quelladressen beschränkt, sofern `cidrs` diese nicht überschreibt. Legen Sie `false` zum Deaktivieren oder `{ user, identity, timeoutMs, cidrs }` zur Feinabstimmung fest. Siehe [Node-Kopplung](/de/gateway/pairing#ssh-verified-device-auto-approval-default).
- `gateway.nodes.commands.allow` / `gateway.nodes.commands.deny`: globale Zulassungs-/Sperrsteuerung für deklarierte Node-Befehle nach der Kopplung und der Auswertung der Plattform-Zulassungsliste. Verwenden Sie `commands.allow`, um gefährliche Node-Befehle wie `camera.snap`, `camera.clip`, `screen.record`, `health.summary`, `sms.search` und `sms.send` zuzulassen; `commands.deny` entfernt einen Befehl, selbst wenn er andernfalls durch eine Plattformvorgabe oder eine explizite Zulassung eingeschlossen würde. Die iOS-Health-Berechtigung, die Android-SMS-Berechtigung und die Gateway-Befehlsautorisierung sind voneinander unabhängig. Nachdem ein Node seine deklarierte Befehlsliste geändert hat, lehnen Sie die Kopplung dieses Geräts ab und genehmigen Sie sie erneut, damit das Gateway den aktualisierten Befehlssnapshot speichert.
- `gateway.tools.deny`: zusätzliche Toolnamen, die für HTTP `POST /tools/invoke` gesperrt sind (erweitert die standardmäßige Sperrliste).
- `gateway.tools.allow`: entfernt Toolnamen aus der standardmäßigen HTTP-Sperrliste für
  Aufrufer mit Eigentümer-/Administratorrechten. Dadurch erhalten identitätstragende `operator.write`-Aufrufer
  keinen Eigentümer-/Administratorzugriff; `cron`, `gateway` und `nodes` bleiben
  für Aufrufer ohne Eigentümerrechte auch dann nicht verfügbar, wenn sie in der Zulassungsliste aufgeführt sind.

</Accordion>

### OpenAI-kompatible Endpunkte

- Admin-HTTP-RPC: standardmäßig deaktiviert, ebenso wie das `admin-http-rpc`-Plugin. Aktivieren Sie das Plugin, um `POST /api/v1/admin/rpc` zu registrieren. Siehe [Admin-HTTP-RPC](/de/plugins/admin-http-rpc).
- Chat Completions: standardmäßig deaktiviert. Aktivieren Sie sie mit `gateway.http.endpoints.chatCompletions.enabled: true`.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Absicherung der URL-Eingabe für Responses:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`
    Leere Zulassungslisten werden als nicht festgelegt behandelt; verwenden Sie `gateway.http.endpoints.responses.files.allowUrl=false`
    und/oder `gateway.http.endpoints.responses.images.allowUrl=false`, um den URL-Abruf zu deaktivieren.
- Optionaler Header zur Absicherung von Antworten:
  - `gateway.http.securityHeaders.strictTransportSecurity` (nur für von Ihnen kontrollierte HTTPS-Ursprünge festlegen; siehe [Authentifizierung über vertrauenswürdige Proxys](/de/gateway/trusted-proxy-auth#tls-termination-and-hsts))

### Isolierung mehrerer Instanzen

Führen Sie mehrere Gateways auf einem Host mit eindeutigen Ports und Zustandsverzeichnissen aus:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Komfortoptionen: `--dev` (verwendet `~/.openclaw-dev` + Port `19001`), `--profile <name>` (verwendet `~/.openclaw-<name>`).

Siehe [Mehrere Gateways](/de/gateway/multiple-gateways).

### `gateway.tls`

```json5
{
  gateway: {
    tls: {
      enabled: false,
      autoGenerate: false,
      certPath: "/etc/openclaw/tls/server.crt",
      keyPath: "/etc/openclaw/tls/server.key",
      caPath: "/etc/openclaw/tls/ca-bundle.crt",
    },
  },
}
```

- `enabled`: aktiviert die TLS-Terminierung am Gateway-Listener (HTTPS/WSS) (Standard: `false`).
- `autoGenerate`: generiert automatisch ein lokales, selbstsigniertes Zertifikat-Schlüsselpaar, wenn keine expliziten Dateien konfiguriert sind; nur für lokale Nutzung oder Entwicklungszwecke.
- `certPath`: Dateisystempfad zur TLS-Zertifikatsdatei.
- `keyPath`: Dateisystempfad zur privaten TLS-Schlüsseldatei; beschränken Sie die Zugriffsberechtigungen.
- `caPath`: optionaler Pfad zum CA-Bundle für die Clientverifizierung oder benutzerdefinierte Vertrauensketten.

### `gateway.reload`

```json5
{
  gateway: {
    reload: {
      mode: "hybrid", // off | restart | hot | hybrid
      debounceMs: 500,
      deferralTimeoutMs: 300000,
    },
  },
}
```

- `mode`: steuert, wie Konfigurationsänderungen zur Laufzeit angewendet werden.
  - `"off"`: ignoriert Änderungen im laufenden Betrieb; Änderungen erfordern einen expliziten Neustart.
  - `"restart"`: startet den Gateway-Prozess bei einer Konfigurationsänderung immer neu.
  - `"hot"`: wendet Änderungen prozessintern ohne Neustart an.
  - `"hybrid"` (Standard): versucht zuerst ein Hot Reload; greift bei Bedarf auf einen Neustart zurück.
- `debounceMs`: Entprellzeitraum in ms, bevor Konfigurationsänderungen angewendet werden (nicht negative Ganzzahl; Standard: `300`).
- `deferralTimeoutMs`: optionale maximale Wartezeit in ms für laufende Vorgänge, bevor ein Neustart oder ein Hot Reload des Kanals erzwungen wird. Lassen Sie den Wert weg, um die standardmäßige begrenzte Wartezeit (`300000`) zu verwenden; setzen Sie `0`, um unbegrenzt zu warten und regelmäßig Warnungen über weiterhin ausstehende Vorgänge zu protokollieren.

---

## Cloud-Worker-Umgebungen

Cloud-Worker sind optional. Wenn `cloudWorkers` fehlt oder `profiles` leer ist, akzeptiert OpenClaw keine Erstellung neuer Worker. Zuvor erstellte dauerhafte Datensätze werden weiterhin abgeglichen und bleiben sichtbar; die vorhandene Gateway-/Node-Projektion bleibt unverändert.

Jeder Worker-Provider muss aus der vertrauenswürdigen Bereitstellungsausgabe einen SSH-`hostKey` exakt als `algorithm base64` zurückgeben, ohne Hostnamen oder Kommentar. Der Bootstrap schreibt diesen Schlüssel in eine isolierte `known_hosts`-Datei, verwendet `StrictHostKeyChecking=yes` und schlägt vor dem Öffnen einer Verbindung fehl, wenn der Provider ihn nicht bereitstellt. Es gibt keinen Trust-on-first-use-Fallback.

Die Einrichtung des Tunnels erfolgt bei Bedarf und nicht im Rahmen der Bereitstellung. Beim Start leitet das Gateway einen Worker-lokalen Unix-Socket rückwärts an seinen Loopback-WebSocket-Endpunkt weiter. Der Socket befindet sich in einem zufällig zugewiesenen Remote-Verzeichnis, auf das nur der Eigentümer zugreifen kann; anders als ein Loopback-TCP-Port ist er für andere Konten auf einem Worker mit mehreren Benutzern nicht erreichbar und kann nicht mit dem Port einer anderen Umgebung kollidieren. SSH-Keepalives und ein begrenzter Backoff für Neuverbindungen laufen nur, solange der Tunnel-Eigentümer aktuell bleibt. Beim Stoppen des Tunnels werden Neuverbindungen unterbunden, bevor der SSH-Prozess geschlossen wird.

Steuerungsdatenverkehr und Workspace-Übertragung verwenden separate SSH-Verbindungen. Beide verwenden dieselbe aufgelöste Identität und dieselbe isolierte, fixierte `known_hosts`-Datei, aber die Workspace-Übertragung teilt sich das SSH-Verbindungs-Multiplexing nicht mit dem langlebigen Tunnel, sodass rsync den Steuerungsdatenverkehr nicht blockieren kann.

### Crabbox-Profil

Der gebündelte `crabbox`-Provider stellt über die lokale Crabbox-CLI eine SSH-fähige Lease bereit. Der innere `settings.provider` wählt das Crabbox-Backend aus; er ist von der äußeren OpenClaw-Provider-ID getrennt.

```json5
{
  cloudWorkers: {
    profiles: {
      production: {
        provider: "crabbox",
        install: "bundle", // Standard; "npm" nur für eine veröffentlichte Gateway-Version verwenden.
        settings: {
          provider: "aws",
          class: "standard",
          ttl: "24h",
          idleTimeout: "60m",
          // Optionaler absoluter Pfad. Standard: benachbartes ../crabbox/bin/crabbox, dann PATH.
          binary: "/usr/local/bin/crabbox",
        },
        lifetime: {
          idleTimeoutMinutes: 60,
          maxLifetimeMinutes: 1440,
        },
      },
    },
  },
}
```

- `settings.provider` (erforderlich): Crabbox-Backend, das über `--provider` übergeben wird. Verwenden Sie ein Backend, dessen Inspektionsausgabe einen SSH-Endpunkt enthält; `aws` wählt das direkte AWS-Backend aus.
- `settings.class` (erforderlich): Crabbox-Maschinenklasse, die an `--class` übergeben wird.
- `settings.ttl` und `settings.idleTimeout` (erforderlich): positive Go-Zeitdauerzeichenfolgen, die an `--ttl` und `--idle-timeout` übergeben werden. Diese Provider-seitigen Ausfallsicherungen unterscheiden sich von der unten beschriebenen gespeicherten `lifetime`-Richtlinie von OpenClaw.
- `settings.binary`: optionaler absoluter Pfad zur ausführbaren Crabbox-Datei. Ohne diesen prüft OpenClaw zunächst den benachbarten Crabbox-Checkout, dann ausführbare Einträge in `PATH` und ruft schließlich `crabbox` auf, sodass eine fehlende CLI als sichtbarer Provider-Fehler erhalten bleibt.

Unbekannte Einstellungen werden abgelehnt. Crabbox-Anmeldedaten und Backend-spezifische Kontokonfigurationen verbleiben im Besitz von Crabbox; legen Sie sie nicht in `settings` ab. OpenClaw ruft nur die lokale CLI auf und führt von diesem Plugin aus keine Provider-Netzwerkaufrufe durch. Bei der Bereitstellung wird immer `--keep=true` übergeben; OpenClaw verwaltet den externen Lebenszyklus und zerstört die Lease mit `crabbox stop`.

<Note>
  OpenClaw löst den Lease-lokalen `sshKey`-Pfad von Crabbox über den Provider-eigenen Resolver für Geheimnisse auf und fixiert den von `crabbox inspect --json` zurückgegebenen maßgeblichen `sshHostKey`. Die AWS-Zulassung erfordert außerdem `providerMetadata.instanceProfileAttached`. Installieren Sie Crabbox 0.38.1 oder neuer für diesen geschlossenen Inspektionsvertrag.
</Note>

### Statisches SSH-Entwicklungsprofil

```json5
{
  cloudWorkers: {
    profiles: {
      development: {
        provider: "static-ssh",
        settings: {
          host: "worker.example.test",
          port: 22,
          user: "openclaw",
          hostKey: "ssh-ed25519 <base64-public-host-key>",
          keyRef: {
            source: "env",
            provider: "default",
            id: "OPENCLAW_WORKER_SSH_KEY",
          },
        },
        lifetime: {
          idleTimeoutMinutes: 60,
          maxLifetimeMinutes: 1440,
        },
      },
    },
  },
}
```

- `profiles`: benannte Worker-Profile mit nicht leeren, um Leerraum bereinigten IDs. Jedes Profil wählt einen von einem Plugin registrierten Provider aus.
- `provider`: nicht leere Worker-Provider-ID. Die Beispiele verwenden den gebündelten `crabbox`-Provider und den QA-Lab-Provider `static-ssh`.
- `install`: Installationsmethode des Workers. `"bundle"` (Standard) überträgt ein inhaltsgehashtes Bundle des installierten Gateway-Builds und unterstützt veröffentlichte, Entwicklungs- und unveröffentlichte Versionen. `"npm"` ist eine optionale Optimierung für eine unveränderte paketierte Veröffentlichung; sie installiert `openclaw@<exact gateway version>` aus der öffentlichen npm-Registry und installiert niemals `latest`.
- Gebündelte Provider-Plugins werden bei entsprechender Konfiguration automatisch ausgewählt, explizite Deaktivierungen und `plugins.allow` gelten jedoch weiterhin. Nehmen Sie die Provider-ID (zum Beispiel `crabbox`) auf, wenn eine Zulassungsliste konfiguriert ist. Externe Provider-Plugins müssen ebenfalls installiert und explizit aktiviert werden.
- `settings`: Provider-eigenes, größenbegrenztes JSON. Das ausgewählte Plugin definiert und validiert seine Schlüssel; verwenden Sie für geheimnistragende Werte [SecretRef-Objekte](/de/gateway/secrets). Der statische SSH-Provider erfordert `host`, `user`, `hostKey` und `keyRef`; `port` ist standardmäßig `22`. `hostKey` muss eine einzelne Zeile mit einem öffentlichen OpenSSH-Hostschlüssel (`algorithm base64`) sein, die vom bekannten Host oder über einen anderen vertrauenswürdigen Kanal bezogen wurde, ohne Optionspräfix.
- `lifetime.idleTimeoutMinutes`: positive ganzzahlige Minuten, die für die spätere Richtlinie zur Freigabe bei Inaktivität gespeichert werden.
- `lifetime.maxLifetimeMinutes`: positive ganzzahlige Minuten, die für die spätere Lebenszyklusrichtlinie gespeichert werden.

Eine unterstützte Node-Laufzeit (22.22.3+, 24.15+ oder 25.9+) mit WAL-Reset-sicherem SQLite muss bereits auf dem Worker installiert sein. Die optionale `"npm"`-Methode erfordert außerdem `npm` und ausgehenden HTTPS-Zugriff auf die öffentliche npm-Registry. Die Einrichtung netzwerkgebundener Toolchains ist eine Provider-Richtlinie; der Bootstrap meldet einen konkret behebbaren Fehler, statt selbst Toolchains zu installieren.

Diese Grundlage installiert und verifiziert den Gateway-Build und stellt den Lebenszyklus zum Starten und Stoppen des Tunnels bereit, startet jedoch nicht die allgemeine OpenClaw-CLI. Der eigenständige Worker-Einstiegspunkt und die Schleife folgen im nächsten Cloud-Worker-Meilenstein.

Jeder dauerhafte Umgebungsdatensatz bewahrt seine validierten Provider-Einstellungen, die aufgelöste Installationsmethode und die Lebenszyklusrichtlinie in einem zum Erstellungszeitpunkt angelegten Profil-Snapshot auf. Das Ändern oder Entfernen eines benannten Profils wirkt sich auf neue Erstellungen aus; vorhandene Datensätze setzen den Lebenszyklusabgleich mit diesem Snapshot fort, sofern das besitzende Plugin weiterhin verfügbar ist.

Lebensdauerwerte sind in der ersten Cloud-Worker-Veröffentlichung lediglich Daten; die automatische Durchsetzung folgt mit späteren Lebenszyklusarbeiten. Profiländerungen erfordern einen Neustart des Gateways.

<Warning>
  Der `static-ssh`-Provider ist ein QA-Lab-Entwicklungsharness für den Quellbaum und von paketierten Distributionen ausgeschlossen. Ein Worker, der auf seinem gemeinsam genutzten Host ausgeführt wird, kann nicht zugehörige Hostdaten lesen; verwenden Sie diesen Provider daher nicht als Isolationsgrenze für die Produktion.
  Der Betreiber muss den erwarteten `hostKey` bereitstellen; OpenClaw lernt oder akzeptiert bei der ersten Verbindung keinen Schlüssel.
  Beim Zerstören seiner Lease wird nur der logische Datensatz von OpenClaw freigegeben; der Host wird weder gestoppt noch bereinigt.
</Warning>

---

## Hooks

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:", "hook:gmail:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.6-sol",
      },
    ],
  },
}
```

Authentifizierung: `Authorization: Bearer <token>` oder `x-openclaw-token: <token>`.
Hook-Token in der Abfragezeichenfolge werden abgelehnt.

Hinweise zu Validierung und Sicherheit:

- `hooks.enabled=true` erfordert einen nicht leeren `hooks.token`.
- `hooks.token` sollte sich von der aktiven Shared-Secret-Authentifizierung des Gateways (`gateway.auth.token` / `OPENCLAW_GATEWAY_TOKEN` oder `gateway.auth.password` / `OPENCLAW_GATEWAY_PASSWORD`) unterscheiden; beim Start wird eine nicht schwerwiegende Sicherheitswarnung protokolliert, wenn eine Wiederverwendung erkannt wird.
- `openclaw security audit` kennzeichnet die Wiederverwendung der Hook-/Gateway-Authentifizierung als kritischen Befund, einschließlich einer Gateway-Passwortauthentifizierung, die nur zum Prüfzeitpunkt bereitgestellt wird (`--auth password --password <password>`). Führen Sie `openclaw doctor --fix` aus, um einen dauerhaft gespeicherten, wiederverwendeten `hooks.token` zu rotieren, und aktualisieren Sie anschließend externe Hook-Absender, damit sie das neue Hook-Token verwenden.
- `hooks.path` darf nicht `/` sein; verwenden Sie einen dedizierten Unterpfad wie `/hooks`.
- Wenn `hooks.allowRequestSessionKey=true`, schränken Sie `hooks.allowedSessionKeyPrefixes` ein (zum Beispiel `["hook:"]`).
- Wenn eine Zuordnung oder Voreinstellung einen vorlagenbasierten `sessionKey` verwendet, legen Sie `hooks.allowedSessionKeyPrefixes` und `hooks.allowRequestSessionKey=true` fest. Statische Zuordnungsschlüssel erfordern diese explizite Aktivierung nicht.

**Endpunkte:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - `sessionKey` aus der Anfrage-Nutzlast wird nur akzeptiert, wenn `hooks.allowRequestSessionKey=true` (Standard: `false`).
- `POST /hooks/<name>` → wird über `hooks.mappings` aufgelöst
  - Durch Vorlagen gerenderte `sessionKey`-Werte der Zuordnung werden als extern bereitgestellt behandelt und erfordern ebenfalls `hooks.allowRequestSessionKey=true`.

<Accordion title="Zuordnungsdetails">

- `match.path` stimmt mit dem Unterpfad nach `/hooks` überein (z. B. `/hooks/gmail` → `gmail`).
- `match.source` stimmt bei generischen Pfaden mit einem Nutzlastfeld überein.
- Vorlagen wie `{{messages[0].subject}}` lesen aus der Nutzlast.
- `transform` kann auf ein JS-/TS-Modul verweisen, das eine Hook-Aktion zurückgibt.
  - `transform.module` muss ein relativer Pfad sein und innerhalb von `hooks.transformsDir` bleiben (absolute Pfade und Verzeichnisüberschreitungen werden abgelehnt).
  - Bewahren Sie `hooks.transformsDir` unter `~/.openclaw/hooks/transforms` auf; Skill-Verzeichnisse des Arbeitsbereichs werden abgelehnt. Wenn `openclaw doctor` diesen Pfad als ungültig meldet, verschieben Sie das Transformationsmodul in das Hook-Transformationsverzeichnis oder entfernen Sie `hooks.transformsDir`.
- `agentId` leitet an einen bestimmten Agenten weiter; unbekannte IDs greifen auf den Standardagenten zurück.
- `allowedAgentIds`: schränkt die effektive Agentenweiterleitung ein, einschließlich des Standardagentenpfads, wenn `agentId` ausgelassen wird (`*` oder ausgelassen = alle zulassen, `[]` = alle verweigern).
- `defaultSessionKey`: optionaler fester Sitzungsschlüssel für Hook-Agentenläufe ohne expliziten `sessionKey`.
- `allowRequestSessionKey`: erlaubt `/hooks/agent`-Aufrufern und vorlagengesteuerten Sitzungsschlüsseln der Zuordnung, `sessionKey` festzulegen (Standard: `false`).
- `allowedSessionKeyPrefixes`: optionale Präfix-Zulassungsliste für explizite `sessionKey`-Werte (Anfrage + Zuordnung), z. B. `["hook:"]`. Sie wird erforderlich, sobald eine Zuordnung oder Voreinstellung einen vorlagenbasierten `sessionKey` verwendet.
- `deliver: true` sendet die endgültige Antwort an einen Kanal; `channel` verwendet standardmäßig `last`.
- `model` überschreibt das LLM für diesen Hook-Lauf (muss zulässig sein, wenn der Modellkatalog festgelegt ist).

</Accordion>

### Gmail-Integration

- Die integrierte Gmail-Voreinstellung verwendet `sessionKey: "hook:gmail:{{messages[0].id}}"`.
- Dieser nachrichtenspezifische Schlüssel isoliert den Konversationskontext, nicht die Tools oder den Zugriff auf den Arbeitsbereich. Ohne eine benutzerdefinierte Zuordnung, die `agentId` festlegt, verwendet die Voreinstellung den Standardagenten.
- Leiten Sie Gmail bei nicht vertrauenswürdigen Posteingängen an einen dedizierten Leseagenten weiter und schränken Sie diesen Agenten mit einer [agentenspezifischen Sandbox- und Tool-Richtlinie](/de/tools/multi-agent-sandbox-tools) ein. Wenn der Leseagent den Hauptagenten benachrichtigen muss, schränken Sie die Übergabe mit [`tools.agentToAgent`](/de/gateway/config-tools#toolsagenttoagent) ein. Das empfohlene Bedrohungsmodell und die Modellstufe finden Sie unter [Prompt-Injection](/de/gateway/security#prompt-injection).
- Wenn Sie diese nachrichtenspezifische Weiterleitung beibehalten, legen Sie `hooks.allowRequestSessionKey: true` fest und schränken Sie `hooks.allowedSessionKeyPrefixes` so ein, dass es dem Gmail-Namensraum entspricht, zum Beispiel `["hook:", "hook:gmail:"]`.
- Wenn Sie `hooks.allowRequestSessionKey: false` benötigen, überschreiben Sie die Voreinstellung mit einem statischen `sessionKey` anstelle des vorlagenbasierten Standards.

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },
      model: "openai/gpt-5.6-sol",
      thinking: "high",
    },
  },
}
```

- Beim Start startet das Gateway automatisch `gog gmail watch serve`, wenn es konfiguriert ist. Legen Sie `OPENCLAW_SKIP_GMAIL_WATCHER=1` fest, um dies zu deaktivieren.
- Führen Sie nicht gleichzeitig mit dem Gateway einen separaten `gog gmail watch serve` aus.

---

## Host des Canvas-Plugins

```json5
{
  plugins: {
    entries: {
      canvas: {
        config: {
          host: {
            root: "~/.openclaw/workspace/canvas",
            liveReload: true,
            // enabled: false, // oder OPENCLAW_SKIP_CANVAS_HOST=1
          },
        },
      },
    },
  },
}
```

- Stellt durch Agenten bearbeitbares HTML/CSS/JS und A2UI über HTTP unter dem Gateway-Port bereit:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Nur lokal: Behalten Sie `gateway.bind: "loopback"` bei (Standard).
- Bei Bindungen außerhalb der Loopback-Schnittstelle erfordern Canvas-Routen eine Gateway-Authentifizierung (Token/Passwort/vertrauenswürdiger Proxy), genau wie andere HTTP-Oberflächen des Gateways.
- Node-WebViews senden üblicherweise keine Authentifizierungsheader; nachdem ein Node gekoppelt und verbunden wurde, stellt das Gateway Node-spezifische Capability-URLs für den Canvas-/A2UI-Zugriff bereit.
- Capability-URLs sind an die aktive WS-Sitzung des Nodes gebunden und laufen schnell ab. Ein IP-basierter Rückgriff wird nicht verwendet.
- Fügt den Live-Reload-Client in bereitgestelltes HTML ein.
- Erstellt automatisch eine `index.html`-Startdatei, wenn das Verzeichnis leer ist.
- Stellt außerdem A2UI unter `/__openclaw__/a2ui/` bereit.
- Änderungen erfordern einen Neustart des Gateways.
- Deaktivieren Sie Live-Reload bei großen Verzeichnissen oder `EMFILE`-Fehlern.

---

## Erkennung

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal` (Standard): `cliPath` + `sshPort` aus TXT-Einträgen auslassen.
- `full`: `cliPath` + `sshPort` einschließen; die LAN-Multicast-Ankündigung erfordert weiterhin, dass das mitgelieferte `bonjour`-Plugin aktiviert ist.
- `off`: unterdrückt die LAN-Multicast-Ankündigung, ohne die Plugin-Aktivierung zu ändern.
- Das mitgelieferte `bonjour`-Plugin startet auf macOS-Hosts automatisch und muss unter Linux, Windows sowie bei containerisierten Gateway-Bereitstellungen explizit aktiviert werden.
- Der Hostname entspricht standardmäßig dem System-Hostnamen, wenn dieser eine gültige DNS-Bezeichnung ist; andernfalls wird `openclaw` verwendet. Überschreiben Sie ihn mit `OPENCLAW_MDNS_HOSTNAME`.
- `OPENCLAW_DISABLE_BONJOUR=1` deaktiviert mDNS-Ankündigungen vollständig und überschreibt `discovery.mdns.mode`.

### Weitbereich (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

Schreibt eine Unicast-DNS-SD-Zone unter `~/.openclaw/dns/`. Kombinieren Sie dies für eine netzwerkübergreifende Erkennung mit einem DNS-Server (CoreDNS empfohlen) und Tailscale Split-DNS.

Einrichtung: `openclaw dns setup --apply`.

---

## Umgebung

### `env` (Inline-Umgebungsvariablen)

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- Inline-Umgebungsvariablen werden nur angewendet, wenn der Schlüssel in der Prozessumgebung fehlt.
- `.env`-Dateien: CWD `.env` + `~/.openclaw/.env` (keine davon überschreibt vorhandene Variablen).
- `shellEnv`: importiert fehlende erwartete Schlüssel aus Ihrem Anmelde-Shell-Profil.
- Die vollständige Rangfolge finden Sie unter [Umgebung](/de/help/environment).

### Ersetzung von Umgebungsvariablen

Referenzieren Sie Umgebungsvariablen in einer beliebigen Konfigurationszeichenfolge mit `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Nur Namen in Großbuchstaben werden berücksichtigt: `[A-Z_][A-Z0-9_]*`.
- Fehlende/leere Variablen lösen beim Laden der Konfiguration einen Fehler aus.
- Maskieren Sie mit `$${VAR}`, um ein literales `${VAR}` zu erhalten.
- Funktioniert mit `$include`.

---

## Secrets

Secret-Referenzen sind additiv: Klartextwerte funktionieren weiterhin.

### `SecretRef`

Verwenden Sie eine einheitliche Objektstruktur:

```json5
{ source: "env" | "file" | "exec", provider: "default", id: "..." }
```

Validierung:

- `provider`-Muster: `^[a-z][a-z0-9_-]{0,63}$`
- `source: "env"`-ID-Muster: `^[A-Z][A-Z0-9_]{0,127}$`
- `source: "file"`-ID: absoluter JSON-Zeiger (zum Beispiel `"/providers/openai/apiKey"`)
- `source: "exec"`-ID-Muster: `^[A-Za-z0-9][A-Za-z0-9._:/#-]{0,255}$` (unterstützt AWS-artige `secret#json_key`-Selektoren)
- `source: "exec"`-IDs dürfen weder `.` noch `..` als durch Schrägstriche getrennte Pfadsegmente enthalten (zum Beispiel wird `a/../b` abgelehnt)

### Unterstützte Anmeldedatenoberfläche

- Kanonische Matrix: [Anmeldedatenoberfläche für SecretRef](/de/reference/secretref-credential-surface)
- `secrets apply` zielt auf unterstützte `openclaw.json`-Anmeldedatenpfade.
- `auth-profiles.json`-Referenzen sind in der Laufzeitauflösung und Prüfungsabdeckung enthalten.

### Konfiguration der Secret-Provider

```json5
{
  secrets: {
    providers: {
      default: { source: "env" }, // optional explicit env provider
      filemain: {
        source: "file",
        path: "~/.openclaw/secrets.json",
        mode: "json",
        timeoutMs: 5000,
      },
      vault: {
        source: "exec",
        command: "/usr/local/bin/openclaw-vault-resolver",
        passEnv: ["PATH", "VAULT_ADDR"],
      },
    },
    defaults: {
      env: "default",
      file: "filemain",
      exec: "vault",
    },
  },
}
```

Hinweise:

- Der `file`-Provider unterstützt `mode: "json"` und `mode: "singleValue"` (`id` muss im singleValue-Modus `"value"` sein).
- Pfade von Datei- und Exec-Providern schlagen sicher fehl, wenn die Überprüfung der Windows-ACL nicht verfügbar ist. Legen Sie `allowInsecurePath: true` nur für vertrauenswürdige Pfade fest, die nicht überprüft werden können.
- Der `exec`-Provider erfordert einen absoluten `command`-Pfad und verwendet Protokollnutzlasten über stdin/stdout.
- Standardmäßig werden symbolische Links in Befehlspfaden abgelehnt. Legen Sie `allowSymlinkCommand: true` fest, um Pfade mit symbolischen Links zuzulassen und dabei den aufgelösten Zielpfad zu validieren.
- Wenn `trustedDirs` konfiguriert ist, gilt die Prüfung des vertrauenswürdigen Verzeichnisses für den aufgelösten Zielpfad.
- Die untergeordnete Umgebung von `exec` ist standardmäßig minimal; übergeben Sie erforderliche Variablen explizit mit `passEnv`.
- Secret-Referenzen werden zum Aktivierungszeitpunkt in einen speicherinternen Snapshot aufgelöst; anschließend lesen Anfragepfade ausschließlich aus dem Snapshot.
- Während der Aktivierung wird nach aktiven Oberflächen gefiltert: Nicht aufgelöste Referenzen auf aktivierten Oberflächen führen dazu, dass Start oder Neuladen fehlschlägt, während inaktive Oberflächen mit Diagnosemeldungen übersprungen werden.

---

## Authentifizierungsspeicher

```json5
{
  auth: {
    profiles: {
      "anthropic:default": { provider: "anthropic", mode: "api_key" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
      "openai:personal": { provider: "openai", mode: "oauth" },
    },
    order: {
      anthropic: ["anthropic:default", "anthropic:work"],
      openai: ["openai:personal"],
    },
  },
}
```

- Agent-spezifische Profile werden unter `<agentDir>/auth-profiles.json` gespeichert.
- `auth-profiles.json` unterstützt für statische Anmeldedatenmodi Referenzen auf Wertebene (`keyRef` für `api_key`, `tokenRef` für `token`).
- Veraltete flache `auth-profiles.json`-Zuordnungen wie `{ "provider": { "apiKey": "..." } }` sind kein Laufzeitformat; `openclaw doctor --fix` schreibt sie mit einer `.legacy-flat.*.bak`-Sicherung in kanonische `provider:default`-API-Schlüsselprofile um.
- Profile im OAuth-Modus (`auth.profiles.<id>.mode = "oauth"`) unterstützen keine SecretRef-basierten Anmeldedaten für Authentifizierungsprofile.
- Statische Laufzeitanmeldedaten stammen aus im Arbeitsspeicher aufgelösten Snapshots; veraltete statische `auth.json`-Einträge werden beim Auffinden bereinigt.
- Veraltete OAuth-Importe aus `~/.openclaw/credentials/oauth.json`.
- Siehe [OAuth](/de/concepts/oauth).
- Laufzeitverhalten von Geheimnissen und `audit/configure/apply`-Werkzeuge: [Geheimnisverwaltung](/de/gateway/secrets).

---

## Audit

```json5
{
  audit: {
    enabled: true,
    messages: "off", // off | direct | all
  },
}
```

Das Gateway zeichnet Audit-Ereignisse **ausschließlich mit Metadaten** für Agent-Ausführungen und Werkzeugaktionen in der gemeinsam genutzten Zustandsdatenbank auf. Metadaten zum Nachrichtenlebenszyklus sind eine separate Opt-in-Funktion. Das Protokoll speichert Identität, Zeitangaben, Werkzeugnamen und normalisierte Ergebnisstatus, jedoch niemals Prompts, Nachrichteninhalte, Werkzeugargumente, Werkzeugergebnisse oder unbereinigten Fehlertext. Nachrichtenzeilen speichern keine unbereinigten Plattformkonto-, Konversations-, Nachrichten- und Ziel-IDs. Sitzungsschlüssel für Ausführungen und Werkzeuge bleiben zur Korrelation verfügbar und können selbst Plattformkonto- oder Peer-IDs enthalten. Datensätze laufen nach 30 Tagen ab, und das Protokoll ist auf 100.000 Zeilen begrenzt. Fragen Sie sie mit [`openclaw audit`](/de/cli/audit) oder dem Gateway-RPC [`audit.activity.list`](/de/gateway/protocol#audit-ledger-rpc) ab. Unter [Audit-Verlauf](/de/gateway/audit) finden Sie das vollständige Datenmodell, die Datenschutzsemantik und die Abdeckungsgrenzen.

- `enabled`: Neue Audit-Ereignisse aufzeichnen (Standard: `true`). Das Protokoll ist standardmäßig aktiviert, da ein erst nach einem Vorfall aktivierter Audit-Trail den Vorfall nicht erklären kann. Wird `false` festgelegt, werden nach dem Neustart des Gateways keine neuen Ereignisse mehr eingefügt; vorhandene Datensätze bleiben bis zu ihrem Ablauf lesbar. Durch erneutes Aktivieren wird die Aufzeichnung ab diesem Zeitpunkt fortgesetzt – die Lücke wird nicht nachträglich gefüllt.
- `messages`: Umfang der Nachrichtenmetadaten (Standard: `"off"`). `"direct"` zeichnet nur bekannte direkte Konversationen auf. `"all"` zeichnet zusätzlich Gruppen, Kanäle und unbekannte Konversationstypen auf. Beide Modi bleiben inhaltsfrei und ersetzen unbereinigte Kennungen, sofern eine Korrelation möglich ist, durch installationslokale, schlüsselbasierte Pseudonyme. Diese dienen als Korrelationshilfen und nicht zur Anonymisierung; die Zustandsdatenbank speichert den Ableitungsschlüssel, RPC- und CLI-Exporte jedoch nicht.

Das laufende Gateway erfasst `audit.enabled` und `audit.messages` beim Start; starten Sie es nach einer Änderung dieser Einstellungen neu. Die Nachrichtenabdeckung umfasst derzeit akzeptierte eingehende Nachrichten, die die zentrale Weiterleitung erreichen, sowie eine abschließende Zeile pro ursprünglicher logischer Nutzlast einer ausgehenden Antwort, die die gemeinsam genutzte dauerhafte Zustellung erreicht. Plugin-lokale und direkte Sendepfade, die diese gemeinsamen Grenzen umgehen, werden noch nicht abgedeckt. Der begrenzte Hintergrundschreiber arbeitet nach bestem Bemühen und ist kein verlustfreies Compliance-Archiv.

---

## Protokollierung

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
```

- Standardprotokolldatei: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`; benannte Profile verwenden `/tmp/openclaw/openclaw-<profile>-YYYY-MM-DD.log`.
- Legen Sie `logging.file` für einen stabilen Pfad fest.
- `consoleLevel` wird auf `debug` erhöht, wenn `--verbose`.
- `maxFileBytes`: Maximale Größe der aktiven Protokolldatei in Byte vor der Rotation (positive Ganzzahl; Standard: `104857600` = 100 MB). OpenClaw bewahrt neben der aktiven Datei bis zu fünf nummerierte Archive auf.
- `redactSensitive` / `redactPatterns`: Maskierung nach bestem Bemühen für Konsolenausgaben, Dateiprotokolle, OTLP-Protokolldatensätze und gespeicherten Text von Sitzungstranskripten. `redactSensitive: "off"` deaktiviert nur diese allgemeine Richtlinie für Protokolle und Transkripte; Sicherheitsoberflächen für Benutzeroberfläche, Werkzeuge und Diagnose schwärzen Geheimnisse weiterhin vor der Ausgabe.

---

## Diagnose

```json5
{
  diagnostics: {
    enabled: true,
    flags: ["telegram.*"],

    otel: {
      enabled: false,
      endpoint: "https://otel-collector.example.com:4318",
      tracesEndpoint: "https://traces.example.com/v1/traces",
      metricsEndpoint: "https://metrics.example.com/v1/metrics",
      logsEndpoint: "https://logs.example.com/v1/logs",
      protocol: "http/protobuf", // http/protobuf | grpc
      headers: { "x-tenant-id": "my-org" },
      serviceName: "openclaw-gateway",
      traces: true,
      metrics: true,
      logs: false,
      logsExporter: "otlp",
      sampleRate: 1.0,
      flushIntervalMs: 5000,
      captureContent: {
        enabled: false,
        inputMessages: false,
        outputMessages: false,
        toolInputs: false,
        toolOutputs: false,
        systemPrompt: false,
        toolDefinitions: false,
      },
    },

    cacheTrace: {
      enabled: false,
      filePath: "~/.openclaw/logs/cache-trace.jsonl",
      includeMessages: true,
      includePrompt: true,
      includeSystem: true,
    },
  },
}
```

- `enabled`: Hauptschalter für Instrumentierungsausgaben (Standard: `true`).
- `flags`: Array aus Flag-Zeichenfolgen zur Aktivierung gezielter Protokollausgaben (unterstützt Platzhalter wie `"telegram.*"` oder `"*"`).
- `otel.enabled`: Aktiviert die OpenTelemetry-Exportpipeline (Standard: `false`). Die vollständige Konfiguration, den Signalkatalog und das Datenschutzmodell finden Sie unter [OpenTelemetry-Export](/de/gateway/opentelemetry).
- `otel.endpoint`: Collector-URL für den OTel-Export.
- `otel.tracesEndpoint` / `otel.metricsEndpoint` / `otel.logsEndpoint`: Optionale signalspezifische OTLP-Endpunkte. Wenn festgelegt, überschreiben sie `otel.endpoint` nur für das jeweilige Signal.
- `otel.protocol`: `"http/protobuf"` (Standard) oder `"grpc"`.
- `otel.headers`: Zusätzliche HTTP-/gRPC-Metadaten-Header, die mit OTel-Exportanfragen gesendet werden.
- `otel.serviceName`: Dienstname für Ressourcenattribute.
- `otel.traces` / `otel.metrics` / `otel.logs`: Aktivieren den Export von Traces, Metriken oder Protokollen.
- `otel.logsExporter`: Ziel für den Protokollexport: `"otlp"` (Standard), `"stdout"` für ein JSON-Objekt pro Standardausgabezeile oder `"both"`.
- `otel.sampleRate`: Trace-Abtastrate `0`–`1`.
- `otel.flushIntervalMs`: Periodisches Telemetrie-Flush-Intervall in ms.
- `otel.captureContent`: Optionale Erfassung von Rohinhalten für OTEL-Span-Attribute. Standardmäßig deaktiviert. Der boolesche Wert `true` erfasst Nachrichten- und Werkzeuginhalte außer Systeminhalten; mit der Objektform können Sie `inputMessages`, `outputMessages`, `toolInputs`, `toolOutputs`, `systemPrompt` und `toolDefinitions` ausdrücklich aktivieren.
- `OTEL_SEMCONV_STABILITY_OPT_IN=gen_ai_latest_experimental`: Umgebungsschalter für die neueste experimentelle Form von GenAI-Inferenz-Spans, einschließlich `{gen_ai.operation.name} {gen_ai.request.model}`-Span-Namen, `CLIENT`-Span-Art und `gen_ai.provider.name` anstelle des veralteten `gen_ai.system`. Standardmäßig behalten Spans aus Kompatibilitätsgründen `openclaw.model.call` und `gen_ai.system`; GenAI-Metriken verwenden begrenzte semantische Attribute.
- `OPENCLAW_OTEL_PRELOADED=1`: Umgebungsschalter für Hosts, die bereits ein globales OpenTelemetry-SDK registriert haben. OpenClaw überspringt dann den Plugin-eigenen SDK-Start und -Stopp, während die Diagnose-Listener aktiv bleiben.
- `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT`, `OTEL_EXPORTER_OTLP_METRICS_ENDPOINT` und `OTEL_EXPORTER_OTLP_LOGS_ENDPOINT`: Signalspezifische Endpunkt-Umgebungsvariablen, die verwendet werden, wenn der entsprechende Konfigurationsschlüssel nicht festgelegt ist.
- `cacheTrace.enabled`: Cache-Trace-Snapshots für eingebettete Ausführungen protokollieren (Standard: `false`).
- `cacheTrace.filePath`: Ausgabepfad für Cache-Trace-JSONL (Standard: `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`).
- `cacheTrace.includeMessages` / `includePrompt` / `includeSystem`: Steuern, was in der Cache-Trace-Ausgabe enthalten ist (alle standardmäßig: `true`).

---

## Aktualisierung

```json5
{
  update: {
    channel: "stable", // stable | extended-stable | beta | dev
    checkOnStart: true,

    auto: {
      enabled: false,
    },
  },
}
```

- `channel`: Veröffentlichungskanal – `"stable"`, `"extended-stable"`, `"beta"` oder `"dev"`. Extended-stable ist ausschließlich paketbasiert: Vordergrundbefehle übernehmen die Installation, während das Gateway rein informative Aktualisierungshinweise ausgeben kann.
- `checkOnStart`: Beim Start des Gateways nach npm-Aktualisierungen suchen (Standard: `true`). Gespeicherte Extended-stable-Auswahlen verwenden denselben rein informativen Hinweis und einen Hinweiszeitplan von 24 Stunden.
- `auto.enabled`: Automatische Hintergrundaktualisierungen für Stable- und Beta-Paketinstallationen aktivieren (Standard: `false`). Extended-stable wird niemals automatisch angewendet.

---

## ACP

```json5
{
  acp: {
    enabled: true,
    dispatch: { enabled: true },
    backend: "acpx",
    fallbacks: ["acpx-secondary"],
    defaultAgent: "main",
    allowedAgents: ["main", "ops"],
    stream: {
      repeatSuppression: true,
      deliveryMode: "live", // live | final_only
    },
  },
}
```

- `enabled`: Globaler ACP-Funktionsschalter (Standard: `true`; legen Sie `false` fest, um ACP-Weiterleitung und Optionen zum Erzeugen auszublenden).
- `dispatch.enabled`: Unabhängiger Schalter für die Weiterleitung von ACP-Sitzungsdurchläufen (Standard: `true`). Legen Sie `false` fest, damit ACP-Befehle verfügbar bleiben, während ihre Ausführung blockiert wird.
- `backend`: ID des standardmäßigen ACP-Laufzeit-Backends (muss mit einem registrierten ACP-Laufzeit-Plugin übereinstimmen).
  Installieren Sie zuerst das Backend-Plugin. Wenn `plugins.allow` festgelegt ist, nehmen Sie außerdem die ID des Backend-Plugins auf (zum Beispiel `acpx`), andernfalls wird das ACP-Backend nicht geladen.
- `fallbacks`: Geordnete Liste von IDs alternativer ACP-Backends, die ausprobiert werden, wenn das primäre Backend frühzeitig mit einem wahrscheinlich vorübergehenden Fehler fehlschlägt (nicht verfügbar, ratenbegrenzt, Kontingent ausgeschöpft oder überlastet), bevor es eine Ausgabe erzeugt hat. Jeder Eintrag muss mit einem registrierten ACP-Laufzeit-Plugin-Backend übereinstimmen.
- `defaultAgent`: ID des alternativen ACP-Ziel-Agenten, wenn beim Erzeugen kein ausdrückliches Ziel angegeben wird.
- `allowedAgents`: Zulassungsliste der Agent-IDs, die für ACP-Laufzeitsitzungen zugelassen sind; leer bedeutet keine zusätzliche Einschränkung.
- `stream.repeatSuppression`: Wiederholte Status-/Werkzeugzeilen pro Durchlauf unterdrücken (Standard: `true`).
- `stream.deliveryMode`: `"live"` streamt schrittweise; `"final_only"` puffert bis zu den abschließenden Ereignissen des Durchlaufs.
- `stream.tagVisibility`: Zuordnung von Tag-Namen zu booleschen Sichtbarkeitsüberschreibungen für gestreamte Ereignisse.
- `runtime.installCommand`: Optionaler Installationsbefehl, der beim Bootstrapping einer ACP-Laufzeitumgebung ausgeführt wird.

---

## Assistent

Verhalten und Metadaten für geführte CLI-Einrichtungsabläufe (`onboard`, `configure`, `doctor`):

```json5
{
  wizard: {
    accessMode: "full",
    appRecommendations: true,
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
    securityAcknowledgedAt: "2026-01-01T00:00:00.000Z",
  },
}
```

- `wizard.accessMode`: Zu Beginn des geführten Onboardings ausgewählte Einwilligung zur Erkennung. `"full"` (empfohlen) ermöglicht dem Einrichtungsprozess, automatisch nach KI-Anwendungen, Schlüsseln und lokalen Laufzeitumgebungen zu suchen; bei `"guarded"` fragt der Einrichtungsprozess einmal nach, bevor er die Umgebung durchsucht, und bietet stattdessen eine manuelle Konfiguration an.

- `wizard.appRecommendations` ist standardmäßig `true`. Setzen Sie den Wert auf `false`, um Empfehlungen für installierte Anwendungen während des geführten oder klassischen Onboardings zu deaktivieren und den Gateway-Zugriff auf `device.apps` zu sperren. Node-Hosts benötigen weiterhin ihr separates, standardmäßig deaktiviertes Flag zur Freigabe installierter Anwendungen, bevor sie den Befehl bekannt geben.

---

## Identität

Siehe die Identitätsfelder unter `agents.entries` in [Agent-Standardeinstellungen](/de/gateway/config-agents#agent-defaults).

---

## Bridge (veraltet, entfernt)

Aktuelle Builds enthalten die TCP-Bridge nicht mehr. Nodes stellen die Verbindung über den Gateway-WebSocket her. `bridge.*`-Schlüssel sind nicht mehr Teil des Konfigurationsschemas (die Validierung schlägt fehl, bis sie entfernt wurden; `openclaw doctor --fix` kann unbekannte Schlüssel entfernen).

<Accordion title="Veraltete Bridge-Konfiguration (historische Referenz)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    webhook: "https://example.invalid/legacy", // veralteter Fallback für gespeicherte notify:true-Jobs
    webhookToken: "replace-with-dedicated-token", // optionales Bearer-Token für die Authentifizierung ausgehender Webhooks
    sessionRetention: "24h", // Dauerangabe oder false
  },
}
```

- `sessionRetention`: Gibt an, wie lange abgeschlossene isolierte Cron-Ausführungssitzungen aufbewahrt werden, bevor die SQLite-Sitzungszeilen bereinigt werden. Steuert außerdem die Bereinigung archivierter Transkripte gelöschter Cron-Jobs. Standard: `24h`; setzen Sie `false`, um dies zu deaktivieren.
- Der Ausführungsverlauf behält automatisch die neuesten 2000 Abschlusszeilen pro Job bei. Verlorene Zeilen behalten ihr 24-stündiges Bereinigungsfenster.
- `webhookToken`: Bearer-Token für die POST-Zustellung von Cron-Webhooks (`delivery.mode = "webhook"`); wenn es weggelassen wird, wird kein Authentifizierungsheader gesendet.
- `webhook`: Veraltete Legacy-Fallback-Webhook-URL (http/https), die von `openclaw doctor --fix` verwendet wird, um gespeicherte Jobs zu migrieren, die noch `notify: true` enthalten; die Laufzeitzustellung verwendet `delivery.mode="webhook"` pro Job zusammen mit `delivery.to` oder `delivery.completionDestination`, wenn die Ankündigungszustellung beibehalten wird.

### `cron.failureAlert`

```json5
{
  cron: {
    failureAlert: {
      enabled: false,
      after: 3,
      cooldownMs: 3600000,
      includeSkipped: false,
      mode: "announce",
      accountId: "main",
    },
  },
}
```

- `enabled`: Aktiviert Fehlerwarnungen für Cron-Jobs (Standard: `false`).
- `after`: Anzahl aufeinanderfolgender Fehler, bevor eine Warnung ausgelöst wird (positive Ganzzahl, Minimum: `1`).
- `cooldownMs`: Mindestanzahl an Millisekunden zwischen wiederholten Warnungen für denselben Job (nicht negative Ganzzahl).
- `includeSkipped`: Zählt aufeinanderfolgende übersprungene Ausführungen für den Warnschwellenwert mit (Standard: `false`). Übersprungene Ausführungen werden separat erfasst und wirken sich nicht auf den Backoff bei Ausführungsfehlern aus.
- `mode`: Zustellungsmodus – `"announce"` sendet über eine Kanalnachricht; `"webhook"` sendet eine POST-Anfrage an den konfigurierten Webhook.
- `accountId`: Optionale Konto- oder Kanal-ID zur Eingrenzung der Warnungszustellung.

### `cron.failureDestination`

```json5
{
  cron: {
    failureDestination: {
      mode: "announce",
      channel: "last",
      to: "channel:C1234567890",
      accountId: "main",
    },
  },
}
```

- Standardziel für Cron-Fehlerbenachrichtigungen über alle Jobs hinweg.
- `mode`: `"announce"` oder `"webhook"`; ist standardmäßig `"announce"`, wenn genügend Zieldaten vorhanden sind.
- `channel`: Kanalüberschreibung für die Ankündigungszustellung. `"last"` verwendet den zuletzt bekannten Zustellungskanal erneut.
- `to`: Explizites Ankündigungsziel oder explizite Webhook-URL. Für den Webhook-Modus erforderlich.
- `accountId`: Optionale Kontoüberschreibung für die Zustellung.
- `delivery.failureDestination` pro Job überschreibt diesen globalen Standardwert.
- Wenn weder ein globales noch ein jobspezifisches Fehlerziel festgelegt ist, greifen Jobs, die bereits über `announce` zustellen, bei einem Fehler auf dieses primäre Ankündigungsziel zurück.
- `delivery.failureDestination` wird nur für `sessionTarget="isolated"`-Jobs unterstützt, sofern der primäre `delivery.mode` des Jobs nicht `"webhook"` ist.

Siehe [Cron-Jobs](/de/automation/cron-jobs). Isolierte Cron-Ausführungen werden als [Hintergrundaufgaben](/de/automation/tasks) erfasst.

## Vorlagenvariablen für Medienmodelle

In `tools.media.models[].args` erweiterte Vorlagenplatzhalter:

| Variable                    | Beschreibung                                      |
| --------------------------- | ------------------------------------------------- |
| `{{Body}}`                  | Vollständiger Text der eingehenden Nachricht      |
| `{{RawBody}}`               | Rohtext (ohne Verlaufs-/Absender-Wrapper)          |
| `{{BodyStripped}}`          | Text ohne Gruppenerwähnungen                       |
| `{{From}}`                  | Absenderkennung                                    |
| `{{To}}`                    | Zielkennung                                        |
| `{{MessageSid}}`            | Kanalnachrichten-ID                                |
| `{{SessionId}}`             | UUID der aktuellen Sitzung                         |
| `{{IsNewSession}}`          | `"true"`, wenn eine neue Sitzung erstellt wurde |
| `{{AttachmentUrl}}`         | URL des aktuellen Anhangs oder Provider-Referenz   |
| `{{AttachmentPath}}`        | Lokaler Pfad des aktuellen Anhangs                 |
| `{{AttachmentContentType}}` | MIME-Inhaltstyp des aktuellen Anhangs              |
| `{{AttachmentDir}}`         | Verzeichnis, das `AttachmentPath` enthält        |
| `{{AttachmentIndex}}`       | Nullbasierter Index des Quellfakts                  |
| `{{Transcript}}`            | Audiotranskript                                    |
| `{{Prompt}}`                | Aufgelöster Medien-Prompt für CLI-Einträge         |
| `{{MaxChars}}`              | Aufgelöste maximale Ausgabezeichenzahl für CLI-Einträge |
| `{{ChatType}}`              | `"direct"` oder `"group"`         |
| `{{GroupSubject}}`          | Gruppenthema (nach bestem Bemühen)                  |
| `{{GroupMembers}}`          | Vorschau der Gruppenmitglieder (nach bestem Bemühen) |
| `{{SenderName}}`            | Anzeigename des Absenders (nach bestem Bemühen)    |
| `{{SenderE164}}`            | Telefonnummer des Absenders (nach bestem Bemühen)  |
| `{{Provider}}`              | Provider-Hinweis (whatsapp, telegram, discord usw.) |

Die veralteten Namen `{{MediaPath}}`, `{{MediaUrl}}`, `{{MediaType}}` und `{{MediaDir}}`
bleiben während des Kompatibilitätszeitraums des Plugin-SDK verfügbar, sind jedoch
veraltet. Neue Konfigurationen sollten die `Attachment*`-Variablen verwenden.

---

## Konfigurationseinbindungen (`$include`)

Teilen Sie die Konfiguration auf mehrere Dateien auf:

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

**Zusammenführungsverhalten:**

- Einzelne Datei: Ersetzt das umgebende Objekt.
- Datei-Array: Wird der Reihe nach tief zusammengeführt (spätere Werte überschreiben frühere).
- Benachbarte Schlüssel: Werden nach den Einbindungen zusammengeführt (und überschreiben eingebundene Werte).
- Verschachtelte Einbindungen: Bis zu 10 Ebenen tief.
- Pfade: Werden relativ zur einbindenden Datei aufgelöst, müssen jedoch innerhalb des obersten Konfigurationsverzeichnisses bleiben (`dirname` von `openclaw.json`). Absolute/`../`-Formen sind nur zulässig, wenn sie weiterhin innerhalb dieser Grenze aufgelöst werden. Legen Sie `OPENCLAW_INCLUDE_ROOTS` (absolute Pfade) fest, um zusätzliche Stammverzeichnisse außerhalb des Konfigurationsverzeichnisses zuzulassen.
- Grenzwerte: Pfade dürfen keine Nullbytes enthalten und müssen vor und nach der Auflösung strikt kürzer als 4096 Zeichen sein; jede eingebundene Datei ist auf 2 MB begrenzt.
- Von OpenClaw ausgeführte Schreibvorgänge, die nur einen einzelnen obersten Abschnitt ändern, der durch eine Einbindung einer einzelnen Datei bereitgestellt wird, schreiben die Änderung in diese eingebundene Datei. Beispielsweise aktualisiert `plugins install` den Wert `plugins: { $include: "./plugins.json5" }` in `plugins.json5` und lässt `openclaw.json` unverändert.
- Stammeinbindungen, Einbindungs-Arrays und Einbindungen mit benachbarten Überschreibungen sind für von OpenClaw ausgeführte Schreibvorgänge schreibgeschützt; diese Schreibvorgänge schlagen sicher fehl, anstatt die Konfiguration zu verflachen.
- Fehler: Klare Meldungen für fehlende Dateien, Parsing-Fehler, zirkuläre Einbindungen, ungültige Pfadformate und übermäßige Länge.

---

## Verwandte Themen

- [Konfiguration](/de/gateway/configuration)
- [Konfigurationsbeispiele](/de/gateway/configuration-examples)
- [Doctor](/de/gateway/doctor)
