---
read_when:
    - Sie müssen Core-Hilfsfunktionen aus einem Plugin aufrufen (TTS, STT, Bildgenerierung, Websuche, Gateway, Unteragent, Nodes)
    - Sie möchten verstehen, was `api.runtime` bereitstellt
    - Sie greifen aus Plugin-Code auf Konfigurations-, Agenten- oder Medien-Hilfsfunktionen zu
sidebarTitle: Runtime helpers
summary: api.runtime -- die injizierten Laufzeit-Hilfsfunktionen, die Plugins zur Verfügung stehen
title: Hilfsfunktionen für die Plugin-Laufzeit
x-i18n:
    generated_at: "2026-07-24T20:38:46Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: ff1d901de8ec70011eeaafbab7b3cc30709fc95894c7ba4f4346c026de682cd0
    source_path: plugins/sdk-runtime.md
    workflow: 16
---

Referenz für das `api.runtime`-Objekt, das bei der Registrierung in jedes Plugin injiziert wird. Verwenden Sie diese Hilfsfunktionen, anstatt Host-Interna direkt zu importieren.

<CardGroup cols={2}>
  <Card title="Channel-Plugins" href="/de/plugins/sdk-channel-plugins">
    Schritt-für-Schritt-Anleitung, die diese Hilfsfunktionen im Kontext von Channel-Plugins verwendet.
  </Card>
  <Card title="Provider-Plugins" href="/de/plugins/sdk-provider-plugins">
    Schritt-für-Schritt-Anleitung, die diese Hilfsfunktionen im Kontext von Provider-Plugins verwendet.
  </Card>
</CardGroup>

```typescript
register(api) {
  const runtime = api.runtime;
}
```

`api.runtime.version` ist die aktuelle Produktversion von OpenClaw und stammt aus dem gemeinsamen Versionsresolver, sodass Plugins denselben Wert sehen, den die CLI meldet.

## Laden und Schreiben der Konfiguration

Bevorzugen Sie die Konfiguration, die bereits an den aktiven Aufrufpfad übergeben wurde, beispielsweise `api.config` während der Registrierung oder ein `cfg`-Argument in Channel-/Provider-Callbacks. Dadurch wird ein einzelner Prozess-Snapshot durch den Arbeitsablauf weitergereicht, anstatt die Konfiguration in häufig ausgeführten Pfaden erneut zu parsen.

Verwenden Sie `api.runtime.config.current()` nur, wenn ein langlebiger Handler den aktuellen Prozess-Snapshot benötigt und dieser Funktion keine Konfiguration übergeben wurde. Der zurückgegebene Wert ist schreibgeschützt; klonen Sie ihn oder verwenden Sie vor der Bearbeitung eine Mutationshilfsfunktion.

Tool-Factories erhalten `ctx.runtimeConfig` sowie `ctx.getRuntimeConfig()`. Verwenden Sie den Getter im `execute`-Callback eines langlebigen Tools, wenn sich die Konfiguration nach Erstellung der Tool-Definition ändern kann.

Persistieren Sie Änderungen mit `api.runtime.config.mutateConfigFile(...)` oder `api.runtime.config.replaceConfigFile(...)`. Jeder Schreibvorgang muss eine explizite `afterWrite`-Richtlinie auswählen:

- `afterWrite: { mode: "auto" }` überlässt die Entscheidung dem Planer für Gateway-Neuladungen.
- `afterWrite: { mode: "restart", reason: "..." }` erzwingt einen sauberen Neustart, wenn der Schreibende weiß, dass ein Hot Reload unsicher ist.
- `afterWrite: { mode: "none", reason: "..." }` unterdrückt das automatische Neuladen bzw. den automatischen Neustart nur, wenn der Aufrufer die Folgemaßnahme übernimmt.

Die Mutationshilfsfunktionen geben `afterWrite` sowie eine typisierte `followUp`-Zusammenfassung zurück, sodass Aufrufer protokollieren oder testen können, ob sie einen Neustart angefordert haben. Das Gateway bestimmt weiterhin, wann dieser Neustart tatsächlich erfolgt.

Verwenden Sie `current()`, ein übergebenes `cfg`, `mutateConfigFile(...)` oder
`replaceConfigFile(...)` für den Zugriff auf die Laufzeitkonfiguration und für Schreibvorgänge.

Bevorzugen Sie bei direkten SDK-Importen die gezielten Konfigurations-Unterpfade gegenüber dem allgemeinen `openclaw/plugin-sdk/config-runtime`-Kompatibilitäts-Barrel: `config-contracts` für Typen, `runtime-config-snapshot` für aktuelle Prozess-Snapshots und `config-mutation` für Schreibvorgänge. Lesen Sie eintragsspezifische Werte aus `api.pluginConfig`; verwenden Sie einen bereitgestellten Tool-Kontext nur für dessen laufzeitweiten Konfigurations-Snapshot und führen Sie Plugin-spezifische Zusammenführungen an dieser Grenze durch. Tests gebündelter Plugins sollten diese gezielten Unterpfade direkt mocken, anstatt das allgemeine Kompatibilitäts-Barrel zu mocken.

Interner OpenClaw-Laufzeitcode folgt demselben Ansatz: Die Konfiguration wird einmal an der CLI-, Gateway- oder Prozessgrenze geladen und anschließend weitergereicht. Erfolgreiche Mutationsschreibvorgänge aktualisieren den Prozess-Laufzeit-Snapshot und erhöhen dessen interne Revision; langlebige Caches sollten den laufzeiteigenen Cache-Schlüssel verwenden, anstatt die Konfiguration lokal zu serialisieren. Ein Scanner für langlebige Laufzeitmodule toleriert keine umgebungsabhängigen `loadConfig()`-Aufrufe; verwenden Sie ein übergebenes `cfg`, ein Anfrage-`context.getRuntimeConfig()` oder `getRuntimeConfig()` an einer expliziten Prozessgrenze.

Ausführungspfade von Providern und Channels müssen den aktiven Laufzeitkonfigurations-Snapshot verwenden, nicht einen Datei-Snapshot, der zum Zurücklesen oder Bearbeiten der Konfiguration zurückgegeben wurde. Datei-Snapshots bewahren Quellwerte wie SecretRef-Markierungen für die Benutzeroberfläche und Schreibvorgänge; Provider-Callbacks benötigen die aufgelöste Laufzeitansicht. Wenn eine Hilfsfunktion entweder mit dem aktiven Quell-Snapshot oder mit dem aktiven Laufzeit-Snapshot aufgerufen werden kann, leiten Sie den Zugriff vor dem Lesen von Anmeldedaten über `selectApplicableRuntimeConfig()`.

## Wiederverwendbare Laufzeit-Dienstprogramme

Verwenden Sie eingehende `botLoopProtection`-Fakten für eingehende Nachrichten, die von Bots verfasst wurden. Der Core wendet den gemeinsamen gleitenden In-Memory-Zeitfensterschutz vor Sitzungserfassung und Dispatch an, ohne die Richtlinie an einen einzelnen Channel zu binden. Der Schutz verfolgt `(scopeId, conversationId, participant pair)`-Schlüssel, zählt beide Richtungen eines Paars gemeinsam, aktiviert nach Überschreitung des Zeitfensterbudgets eine Abkühlphase und entfernt inaktive Einträge bei Gelegenheit.

Channel-Plugins, die dieses Verhalten für Betreibende verfügbar machen, sollten für Basisbudgets bevorzugt die gemeinsame `channels.defaults.botLoopProtection`-Struktur verwenden und anschließend Channel-/Provider-spezifische Überschreibungen darauf anwenden. Die gemeinsame Konfiguration verwendet Sekunden, da sie benutzerseitig sichtbar ist:

```typescript
type ChannelBotLoopProtectionConfig = {
  enabled?: boolean;
  maxEventsPerWindow?: number;
  windowSeconds?: number;
  cooldownSeconds?: number;
};
```

Übergeben Sie normalisierte Bot-Paar-Fakten zusammen mit dem aufgelösten Turn. Der Core löst Standardwerte, Einheitenumrechnung und die `enabled`-Semantik auf:

```typescript
return {
  channel: "example",
  routeSessionKey,
  storePath,
  ctxPayload,
  recordInboundSession,
  runDispatch,
  botLoopProtection: {
    scopeId: "account-1",
    conversationId: "channel-1",
    senderId: "bot-a",
    receiverId: "bot-b",
    config: channelConfig.botLoopProtection,
    defaultsConfig: runtimeConfig.channels?.defaults?.botLoopProtection,
    defaultEnabled: allowBotsMode !== "off",
  },
};
```

Verwenden Sie `openclaw/plugin-sdk/pair-loop-guard-runtime` nur für benutzerdefinierte
Ereignisschleifen zwischen zwei Parteien direkt, die nicht über den gemeinsamen Runner für eingehende Antworten laufen.

## Laufzeit-Namespaces

<AccordionGroup>
  <Accordion title="api.runtime.agent">
    Agentenidentität, Verzeichnisse und Sitzungsverwaltung.

    ```typescript
    // Arbeitsverzeichnis des Agenten auflösen (agentId ist erforderlich)
    const agentDir = api.runtime.agent.resolveAgentDir(cfg, agentId);

    // Agenten-Workspace auflösen
    const workspaceDir = api.runtime.agent.resolveAgentWorkspaceDir(cfg, agentId);

    // Agentenidentität abrufen
    const identity = api.runtime.agent.resolveAgentIdentity(cfg);

    // Standard-Denkniveau abrufen
    const thinking = api.runtime.agent.resolveThinkingDefault({
      cfg,
      provider,
      model,
    });

    // Ein benutzerdefiniertes Denkniveau anhand des aktiven Provider-Profils validieren
    const policy = api.runtime.agent.resolveThinkingPolicy({ provider, model });
    const level = api.runtime.agent.normalizeThinkingLevel("extra high");
    if (level && policy.levels.some((entry) => entry.id === level)) {
      // Denkniveau an einen eingebetteten Lauf übergeben
    }

    // Agenten-Timeout abrufen
    const timeoutMs = api.runtime.agent.resolveAgentTimeoutMs(cfg);

    // Sicherstellen, dass der Workspace vorhanden ist
    await api.runtime.agent.ensureAgentWorkspace(cfg);

    // Einen eingebetteten Agenten-Turn ausführen
    const result = await api.runtime.agent.runEmbeddedAgent({
      sessionId: "my-plugin:task-1",
      runId: crypto.randomUUID(),
      workspaceDir: api.runtime.agent.resolveAgentWorkspaceDir(cfg, agentId),
      prompt: "Fassen Sie die neuesten Änderungen zusammen",
      timeoutMs: api.runtime.agent.resolveAgentTimeoutMs(cfg),
    });
    ```

    `runEmbeddedAgent(...)` ist die neutrale Hilfsfunktion zum Starten eines normalen OpenClaw-Agenten-Turns aus Plugin-Code. Sie verwendet dieselbe Provider-/Modellauflösung und Auswahl des Agenten-Harness wie durch Channels ausgelöste Antworten.

    `runEmbeddedPiAgent(...)` bleibt als veralteter Kompatibilitätsalias für bestehende Plugins erhalten. Neuer Code sollte `runEmbeddedAgent(...)` verwenden.

    `resolveCliBackendDispatchEligibility({ provider, model, agentId, authProfileId, config, agentDir, workspaceDir })` stellt Aufrufern, die eingebettete Läufe für `cliBackendDispatch: "subscription-auth"` aktivieren, die Dispatch-Entscheidung des eingebetteten Runners für das CLI-Backend bereit (Route, die deklarierte `subscriptionAuthDispatch`-Fähigkeit des Backends, gespeicherter Anmeldedatenmodus – unter Beachtung eines explizit festgelegten `authProfileId`). Es gibt `{ provider }` zurück, wenn der Lauf über das CLI-Backend ausgeführt würde, und `undefined`, wenn er beim direkten Passthrough verbleibt, sodass Aufrufer Timeouts für den Lauf einplanen können, der tatsächlich ausgeführt wird.

    `resolveThinkingPolicy(...)` gibt die vom Provider/Modell unterstützten Denkniveaus sowie einen optionalen Standardwert zurück. Provider-Plugins verwalten das modellspezifische Profil über ihre Thinking-Hooks, daher sollten Tool-Plugins diese Laufzeithilfsfunktion aufrufen, anstatt Provider-Listen zu importieren oder zu duplizieren.

    `normalizeThinkingLevel(...)` konvertiert Benutzereingaben wie `on`, `x-high` oder `extra high` in das kanonisch gespeicherte Niveau, bevor es anhand der aufgelösten Richtlinie geprüft wird.

    **Hilfsfunktionen für den Sitzungsspeicher** befinden sich unter `api.runtime.agent.session`:

    ```typescript
    const entry = api.runtime.agent.session.getSessionEntry({ agentId, sessionKey });
    for (const { sessionKey, entry } of api.runtime.agent.session.listSessionEntries({ agentId })) {
      // Sitzungszeilen durchlaufen, ohne von der veralteten sessions.json-Struktur abhängig zu sein.
    }
    await api.runtime.agent.session.patchSessionEntry({
      agentId,
      sessionKey,
      update: (entry) => ({ thinkingLevel: "high" }),
    });

    const created = await api.runtime.agent.session.createSessionEntry({
      cfg,
      key: "agent:main:my-plugin:task-1",
      initialEntry: {
        agentHarnessId: "my-harness",
        modelSelectionLocked: true,
        pluginExtensions: { "my-plugin": { phase: "initializing" } },
      },
      afterCreate: async () => ({
        pluginExtensions: { "my-plugin": { phase: "ready" } },
      }),
    });

    const storePath = api.runtime.agent.session.resolveStorePath(cfg.session?.store, { agentId });
    await api.runtime.agent.session.runWithWorkAdmission(
      { storePath, sessionKey },
      async (signal) => {
        // Sitzung erstellen oder aktualisieren und anschließend signal an den zugelassenen Agentenlauf übergeben.
      },
    );
    ```

    Bevorzugen Sie `getSessionEntry(...)`, `listSessionEntries(...)`, `patchSessionEntry(...)` oder `upsertSessionEntry(...)` für Sitzungsabläufe. Diese Hilfsfunktionen adressieren Sitzungen über die Agenten-/Sitzungsidentität, sodass Plugins nicht von der veralteten `sessions.json`-Speicherstruktur abhängen. Verwenden Sie `preserveActivity: true` für reine Metadaten-Patches, die die Sitzungsaktivität nicht aktualisieren sollen, und `replaceEntry: true` nur, wenn der Callback einen vollständigen Eintrag zurückgibt und gelöschte Felder gelöscht bleiben müssen. Doctor- und Migrationspfade können `fallbackEntry`, `skipMaintenance` und `requireWriteSuccess` für eine einzelne atomare Reparatur des kanonischen Speichers kombinieren.

    `createSessionEntry(...)` erstellt eine neue kanonische Sitzungszeile und ein Transkript. Die vertrauenswürdige `initialEntry`-Oberfläche ist bewusst eng gefasst: ein nicht leeres `agentHarnessId`, ein optionales `modelSelectionLocked: true` und ein optionales `pluginExtensions`. Die injizierte Laufzeit akzeptiert über `registerAgentHarness(...)` ausschließlich Harness-IDs, die dem aufrufenden Plugin gehören; dies ist eine Eigentumsinvariante und keine Sandbox zwischen prozessinternen Plugins. Eine bereits vorhandene Zeile wird abgelehnt; `label` und `spawnedCwd` sind eigenständige Erstellungsfelder und keine vertrauenswürdigen Eintrags-Patches.

    Während der Erstellung wird die Mutationssperre für den Sitzungslebenszyklus über `afterCreate` gehalten, sodass neue Arbeit auf den Abschluss der Plugin-eigenen Initialisierung wartet und bereits zuvor zugelassene Arbeit die Erstellung fehlschlagen lässt. Der Callback erhält einen Klon des erstellten Zustands. Wenn er einen Patch zurückgibt, darf dieser Patch ausschließlich `pluginExtensions` enthalten, und dessen Wert ist das vollständige endgültige `pluginExtensions`-Feld. Ein Fehler im Callback oder bei der endgültigen Persistierung setzt die unveränderte neue Zeile und das Transkript zurück; ein geschützter Rollback bewahrt eine Zeile, die zwischenzeitlich geändert oder beansprucht wurde. `recoverMatchingInitialEntry: true` dient ausschließlich zum erneuten Versuch einer unterbrochenen Initialisierung, wenn die persistierten vertrauenswürdigen Felder exakt übereinstimmen, und die Wiederherstellung erfordert, dass `afterCreate` einen endgültigen Patch zurückgibt.

    Verwenden Sie `runWithWorkAdmission(...)`, wenn ein Plugin Arbeit an einer persistierten Sitzung beginnt. Der Callback lehnt archivierte oder gleichzeitig ersetzte Sitzungen ab, koordiniert Archivierungs-, Zurücksetzungs- und Löschmutationen bis zum Abschluss und erhält ein `AbortSignal`, das an den Agentenlauf weitergegeben werden muss. Ein Harness kann über sein experimentelles `delegatedExecutionPluginIds`-Registrierungsfeld explizit vertrauenswürdige Ausführungsdelegierte benennen. Delegierte können nur eine exakt vorhandene, modellgesperrte Sitzung zulassen und ausführen; sämtliche Sitzungsmutationen bleiben auf den Eigentümer des Harness beschränkt. Siehe [Agenten-Harness-Plugins](/de/plugins/sdk-agent-harness#delegated-execution).

    Wartungs- und Reparatur-Plugins können `deleteSessionEntry(...)` für einen einzelnen bereichsgebundenen Sitzungseintrag, `cleanupSessionLifecycleArtifacts(...)` für lebenszyklusverwaltete temporäre Sitzungen und `resolveSessionStoreBackupPaths(...)` vor der Änderung eines Speichers verwenden. Übergeben Sie `expectedSessionId` und `expectedUpdatedAt`, wenn das Löschen nicht mit einer gleichzeitigen Sitzungsaktualisierung kollidieren darf; verwenden Sie `expectedSessionId: null`, wenn der frühere Snapshot keine Sitzungs-ID enthielt. Diese Hilfsfunktionen sind eng begrenzte Reparatur-/Lebenszyklus-Schnittstellen und keine allgemeine API zum Löschen von Speichern.

    `resolveStorePath(...)` und `updateSessionStoreEntry(...)` vervollständigen die Sitzungshilfsfunktionen: `resolveStorePath` ermittelt den Pfad des Sitzungsspeichers für einen bestimmten Gültigkeitsbereich, und `updateSessionStoreEntry({ storePath, sessionKey, update })` aktualisiert einen Eintrag direkt über den Speicherpfad, wenn der Aufrufer diesen bereits kennt.

    `loadTranscriptEventsSync(...)` steht für synchrone Doctor- und Reparaturpfade zur Verfügung, die die asynchrone Transkript-Laufzeit nicht verwenden können. Die Funktion gibt unverarbeitete `SessionStoreTranscriptEvent`-Datensätze zurück. Normaler Plugin-Laufzeitcode sollte `openclaw/plugin-sdk/session-transcript-runtime` bevorzugen.

    `formatSqliteSessionFileMarker(...)`, `parseSqliteSessionFileMarker(...)` und `sqliteSessionFileMarkerMatchesSession(...)` sind Übergangshilfsfunktionen für Code, der noch ein Legacy-Feld namens `sessionFile` empfängt. Eine geparste SQLite-Markierung bezeichnet ein aktives SQLite-Transkriptziel; sie ist kein Dateisystempfad. Neue APIs sollten eine typisierte Sitzungsidentität anstelle von Markierungszeichenfolgen übertragen.

    Importieren Sie für Lese- und Schreibvorgänge an Transkripten `openclaw/plugin-sdk/session-transcript-runtime` und verwenden Sie `resolveSessionTranscriptIdentity(...)`, `resolveSessionTranscriptTarget(...)`, `readSessionTranscriptEvents(...)`, `readSessionTranscriptRawDelta(...)`, `readSessionTranscriptVisibleMessageDelta(...)`, `readVisibleSessionTranscriptMessageEntries(...)`, `appendSessionTranscriptMessageByIdentity(...)`, `publishSessionTranscriptUpdateByIdentity(...)` oder `withSessionTranscriptWriteLock(...)` mit `{ agentId, sessionKey, sessionId }`. Mit diesen APIs können Plugins ein Transkript identifizieren, unverarbeitete Ereignisse oder sichtbare, verzweigungssichere Nachrichteneinträge lesen, Nachrichten anhängen, Aktualisierungen veröffentlichen und zugehörige Vorgänge unter derselben Transkript-Schreibsperre ausführen, ohne von aktiven Transkriptdateipfaden abhängig zu sein. `readVisibleSessionTranscriptMessageEntries(...)` gibt geordnete Lesemetadaten zurück; das Feld `seq` ist kein fortsetzbarer Cursor.

    `appendSessionTranscriptMessageByIdentity(...)` ist eine Low-Level-Funktion zum Anhängen einer bereits kanonischen Nachricht. Plugins dürfen keine Benutzerzeilen mit Medieninhalten und `MediaPath`, `MediaPaths`, `MediaUrl`, `MediaUrls`, `MediaType` oder `MediaTypes` auf oberster Ebene erzeugen. Der Kanaleingang sollte geordnete Fakten über `MsgContext.media` übergeben und die Persistierung des Benutzerzugs dem Host überlassen. Eine vom Host vorbereitete persistierte Benutzernachricht enthält kanonische geordnete Fakten unter `message.__openclaw.media`; die generische Anhänge-API leitet parallele Legacy-Arrays weder ab noch repariert sie diese.

    `readSessionTranscriptRawDelta(...)` gibt ein begrenztes Ergebnis vom Typ `page`, `reset` oder `missing` zurück. Übergeben Sie den opaken `page.cursor` an den nächsten Aufruf. Reine Anhängevorgänge behalten den Cursor bei, während das Ersetzen des Transkripts `reset` mit einem neuen Bootstrap-Cursor zurückgibt. Seiten umfassen standardmäßig 1.000 Ereignisse und 1.000.000 serialisierte Byte; Aufrufer können bis zu 10.000 Ereignisse und 64 MiB anfordern. Wenn bereits das nächste Ereignis `maxBytes` überschreitet, ist die Seite leer und meldet `requiredBytes`; versuchen Sie es erneut mit mindestens diesem Byte-Limit, sofern es nicht größer als 64 MiB ist. Größere Einzelereignisse erfordern die API zum vollständigen Lesen. Ein Cursor bezeichnet nur eine Position und gewährt niemals Zugriff auf eine andere Sitzung.

    `readSessionTranscriptVisibleMessageDelta(...)` bietet dieselbe begrenzte Bootstrap-und-Fortsetzungsstruktur für die hostverwaltete aktive Nachrichtenprojektion. Die Funktion gibt Nachrichten von der ältesten zur neuesten zurück, sodass Kontext-Engines den anfänglichen Verlauf vollständig verarbeiten und den opaken Cursor als ihre Fortschrittsmarke persistieren können. Speichern Sie den Cursor unverändert und geben Sie ihn unverändert zurück; er ist ein Fortsetzungshinweis und kein Autorisierungsnachweis. Lineare Anhängevorgänge werden nach der zuletzt zurückgegebenen Nachricht fortgesetzt. Das Ersetzen des Transkripts, ein Cursor, dessen Anker den aktiven Zweig verlassen hat oder innerhalb dieses Zweigs verschoben wurde, fehlerhafte Cursor und sitzungsübergreifende Cursor geben `reset` mit einem neuen Bootstrap-Cursor zurück. Die Standardwerte und Obergrenzen für Anzahl und Byte entsprechen denen der Rohdaten-Delta-API. Während die aktive Projektion nach einer Verzweigungsänderung neu aufgebaut wird, lautet das Ergebnis `unavailable` mit dem Grund `projection_rebuilding`; versuchen Sie es später erneut, anstatt auf eine aktive Transkriptdatei zurückzugreifen.

    Die Legacy-Hilfsfunktionen für den gesamten Speicher und für aktive Transkriptdateien werden nicht mehr aus dem Plugin-SDK exportiert. Verwenden Sie die bereichsgebundenen Eintragshilfsfunktionen für Sitzungsmetadaten und die Hilfsfunktionen zur Transkriptidentität für aktive Transkriptvorgänge. Archivierungs-/Support-Workflows, die Dateiartefakte benötigen, sollten ihre dedizierten Archivschnittstellen anstelle der Laufzeit-APIs für aktive Sitzungen verwenden.

  </Accordion>
  <Accordion title="api.runtime.agent.defaults">
    Konstanten für das Standardmodell und den Standard-Provider:

    ```typescript
    const model = api.runtime.agent.defaults.model; // z. B. "gpt-5.6-sol"
    const provider = api.runtime.agent.defaults.provider; // z. B. "openai"
    ```

  </Accordion>

  <Accordion title="api.runtime.llm">
    Führen Sie eine hostverwaltete Textvervollständigung aus, ohne interne Provider-Komponenten zu importieren oder
    die Vorbereitung von OpenClaw-Modell, Authentifizierung und Basis-URL zu duplizieren.

    ```typescript
    const result = await api.runtime.llm.complete({
      messages: [{ role: "user", content: "Fassen Sie dieses Transkript zusammen." }],
      purpose: "my-plugin.summary",
      maxTokens: 512,
      temperature: 0.2,
      reasoning: "high",
    });
    ```

    Die Provider-Orchestrierung kann außerdem den Lebenszyklus des konfigurierten lokalen Dienstes
    übernehmen, bevor eine HTTP-Anfrage gesendet wird:

    ```typescript
    const lease = await api.runtime.llm.acquireLocalService(
      {
        providerId,
        baseUrl,
        headers,
      },
      signal,
    );
    try {
      // Senden und verarbeiten Sie die Provider-Anfrage vollständig.
    } finally {
      await lease?.release();
    }
    ```

    `acquireLocalService(...)` ist ein stabiler, generischer SDK-Vertrag für Provider-Dienste.
    Der Host ermittelt die Prozesskonfiguration aus
    `models.providers.<providerId>.localService`; Aufrufer können keinen
    Befehl, keine Argumente, keine Umgebung und keine Lebenszyklusrichtlinie angeben. Prozesserzeugung,
    Bereitschaft, Diagnose und Richtlinie für das Beenden bei Inaktivität bleiben hostintern.

    Übergeben Sie die genaue konfigurierte Provider-ID und die aufgelöste Basis-URL der Anfrage. Ersetzen Sie
    Aliasse nicht durch eine Adapter-ID: Unterschiedliche Aliasse können auf unterschiedliche
    lokale GPU-Hosts verweisen. Der Host lehnt Endpunkte ab, die nicht mit der konfigurierten
    Provider-Basis-URL übereinstimmen, abgesehen von der durch Ollama- und LM-Studio-Adapter verwendeten
    `/v1`-Normalisierung. Der Host verwaltet die Serialisierung des Starts, Bereitschaftsprüfungen,
    Anfrage-Leases, Abbruchbehandlung und das Herunterfahren bei Inaktivität.

    Die Hilfsfunktion verwendet denselben Vorbereitungsablauf für einfache Vervollständigungen wie die
    integrierte OpenClaw-Laufzeit sowie den hostverwalteten Snapshot der Laufzeitkonfiguration. Kontext-Engines
    erhalten eine sitzungsgebundene `llm.complete`-Fähigkeit, sodass Modellaufrufe den Agenten
    der aktiven Sitzung verwenden und nicht stillschweigend auf den Standardagenten zurückfallen. Das
    Ergebnis enthält die Zuordnung zu Provider, Modell und Agent sowie normalisierte Angaben zu Token-
    und Cache-Nutzung und, sofern verfügbar, geschätzten Kosten.

    Setzen Sie `reasoning`, um einen Reasoning-Aufwand für das ausgewählte Modell anzufordern. Der
    Host normalisiert die kanonischen Denkstufen (`off`, `minimal`, `low`,
    `medium`, `high`, `xhigh`, `adaptive`, `max` und `ultra`) für den ausgewählten
    Provider und das ausgewählte Modell, bevor die Vervollständigung weitergeleitet wird. `adaptive` wird zu
    `medium`; `max` und `ultra` werden zu `max`, sofern unterstützt, andernfalls zu `xhigh`.

    <Warning>
    Modellüberschreibungen erfordern die Zustimmung des Betreibers über `plugins.entries.<id>.llm.allowModelOverride: true` in der Konfiguration. Verwenden Sie `plugins.entries.<id>.llm.allowedModels`, um vertrauenswürdige Plugins auf bestimmte kanonische `provider/model`-Ziele zu beschränken. Agentenübergreifende Vervollständigungen erfordern `plugins.entries.<id>.llm.allowAgentIdOverride: true`.
    </Warning>

  </Accordion>
  <Accordion title="api.runtime.gateway">
    Rufen Sie prozessintern eine andere Gateway-Methode auf und behalten Sie dabei die vertrauenswürdige Laufzeitidentität
    des aktuellen Plugins bei. Dies ist für gebündelte oder vertrauenswürdige offizielle Plugins vorgesehen, die Plugin-eigene
    Gateway-Fähigkeiten kombinieren, ohne eine Loopback-WebSocket-Verbindung zu öffnen.

    ```typescript
    if (await api.runtime.gateway.isAvailable()) {
      const result = await api.runtime.gateway.request<{ callId: string }>(
        "voicecall.start",
        { to: "+15550001234", mode: "conversation" },
        { timeoutMs: 60_000 },
      );
    }
    ```

    Anfragen verwenden den Gültigkeitsbereich `operator.write` und gewähren keinen Administrator-Gültigkeitsbereich. Aufrufe von beliebigen externen
    Plugins werden abgelehnt. Fehlgeschlagene Methoden lösen einen `GatewayClientRequestError` aus und bewahren dabei strukturierte
    `details`, Metadaten für Wiederholungsversuche und den Gateway-Fehlercode für Wiederherstellungsabläufe. Verwenden Sie `isAvailable()`,
    bevor Sie diesen Pfad in Tools auswählen, die auch in eigenständigen Agentenprozessen ausgeführt werden können.

  </Accordion>
  <Accordion title="api.runtime.subagent">
    Starten und verwalten Sie Subagentenläufe im Hintergrund.

    ```typescript
    // Einen Subagentenlauf starten
    const { runId } = await api.runtime.subagent.run({
      sessionKey: "agent:main:subagent:search-helper",
      message: "Erweitern Sie diese Anfrage zu gezielten Folgesuchen.",
      toolsAlsoAllow: ["my_plugin_progress"],
      provider: "openai", // optionale Überschreibung
      model: "gpt-5.6-sol", // optionale Überschreibung
      deliver: false,
    });

    // Auf den Abschluss warten
    const result = await api.runtime.subagent.waitForRun({ runId, timeoutMs: 30000 });

    // Sitzungsnachrichten lesen
    const { messages } = await api.runtime.subagent.getSessionMessages({
      sessionKey: "agent:main:subagent:search-helper",
      limit: 10,
    });

    // Eine Sitzung löschen
    await api.runtime.subagent.deleteSession({
      sessionKey: "agent:main:subagent:search-helper",
    });
    ```

    <Warning>
    Modellüberschreibungen (`provider`/`model`) erfordern die Zustimmung des Betreibers über `plugins.entries.<id>.subagent.allowModelOverride: true` in der Konfiguration. Nicht vertrauenswürdige Plugins können weiterhin Subagenten ausführen, Überschreibungsanfragen werden jedoch abgelehnt.
    </Warning>

    `toolsAlsoAllow` fügt der normalen Tool-Oberfläche des Workers exakt benannte Tools hinzu, die eindeutig dem aufrufenden Plugin gehören und von ihm registriert wurden. Die Laufzeit lehnt Kern-Tools und Namen ab, die mit einem anderen Plugin geteilt werden. Profile und Tool-Richtlinien des Betreibers gelten weiterhin, einschließlich expliziter Zulassungs- und Sperrlisten.

    `deleteSession(...)` kann Sitzungen löschen, die dasselbe Plugin über `api.runtime.subagent.run(...)` erstellt hat. Das Löschen beliebiger Benutzer- oder Betreibersitzungen erfordert weiterhin eine Gateway-Anfrage mit Administrator-Gültigkeitsbereich.

  </Accordion>
  <Accordion title="api.runtime.sandbox">
    Prüfen Sie die effektive Sandbox-Arbeitsbereichsberechtigung für eine Agentensitzung.

    ```typescript
    const authority = api.runtime.sandbox.resolveWorkspaceAuthority({
      config: cfg,
      agentId,
      sessionKey,
    });

    const liveAuthority = await api.runtime.sandbox.prepareWorkspaceAuthority({
      config: cfg,
      agentId,
      sessionKey,
      workspaceDir,
      confinedToolNames: ["my_plugin_safe_tool"],
    });
    ```

    Das Ergebnis gibt an, ob diese Sitzung in einer Sandbox ausgeführt wird, ob ihr Arbeitsbereich
    nicht verfügbar, schreibgeschützt oder beschreibbar ist, sowie optional `confinementError`,
    wenn die effektive Docker-, Tool-, Sitzungs-, Browser- oder erhöhte Richtlinie
    aus diesem Arbeitsbereich ausbrechen kann. Verwenden Sie dies für hostverwaltete Delegierungsentscheidungen, die
    einem Worker nicht mehr Berechtigungen gewähren dürfen als seinem Aufrufer. Es handelt sich um eine
    Attestierungshilfe und nicht um einen Ersatz für die Prüfung der eigenen Autorisierung des Aufrufers.

    `prepareWorkspaceAuthority(...)` führt dieselbe Richtlinienprüfung durch und
    bereitet außerdem die Docker-Sandbox für `workspaceDir` vor. Die Funktion lehnt einen aktiven Container ab,
    dessen Hash der Live-Konfiguration nicht mit den angeforderten Mounts oder der Richtlinie übereinstimmt. Übergeben Sie
    nur exakte Tool-Namen, deren registrierte Implementierungen das aufrufende Plugin
    einschränkt; Platzhalterpräfixe belegen keine Tool-Eigentümerschaft.

  </Accordion>
  <Accordion title="api.runtime.nodes">
    Listen Sie verbundene Nodes auf und rufen Sie einen vom Node gehosteten Befehl aus vom Gateway geladenem Plugin-Code oder aus Plugin-CLI-Befehlen auf. Verwenden Sie dies, wenn ein Plugin lokale Aufgaben auf einem gekoppelten Gerät übernimmt, beispielsweise eine Browser- oder Audio-Bridge auf einem anderen Mac.

    ```typescript
    const { nodes } = await api.runtime.nodes.list({ connected: true });

    const result = await api.runtime.nodes.invoke({
      nodeId: "mac-studio",
      command: "my-plugin.command",
      params: { action: "start" },
      timeoutMs: 30000,
    });
    ```

    `nodes.list(...)` enthält die von jedem verbundenen Node angekündigten
    `nodePluginTools`-Deskriptoren, wenn dieser Node dem Agenten Plugin- oder MCP-gestützte
    Tools bereitstellt. Diese Deskriptoren bilden den aktuellen Verbindungsstatus ab: Das Gateway
    entfernt sie, wenn der Node die Verbindung trennt, und ein Node kann sie nach Änderungen am
    lokalen Plugin-/MCP-Inventar durch `node.pluginTools.update` ersetzen.

    Innerhalb des Gateways wird diese Runtime prozessintern ausgeführt. In Plugin-CLI-Befehlen ruft sie das konfigurierte Gateway über RPC auf, sodass Befehle wie `openclaw googlemeet recover-tab` gekoppelte Nodes vom Terminal aus prüfen können. Node-Befehle durchlaufen weiterhin die normale Gateway-Node-Kopplung, Befehls-Positivlisten, Richtlinien von Plugins für Node-Aufrufe sowie die lokale Befehlsverarbeitung des Nodes.

    Plugins, die auf Nodes gehostete Agenten-Tools bereitstellen, können `agentTool.defaultPlatforms` für ungefährliche Befehle festlegen, die standardmäßig in die Positivliste aufgenommen werden sollen. Lassen Sie es weg, wenn Betreiber dies mit `gateway.nodes.commands.allow` ausdrücklich aktivieren müssen. Gefährliche Befehle auf dem Node-Host sollten mit `api.registerNodeInvokePolicy(...)` eine Richtlinie für Node-Aufrufe registrieren; die Richtlinie wird im Gateway nach den Prüfungen der Befehls-Positivliste und vor der Weiterleitung des Befehls an den Node ausgeführt. Dadurch verwenden direkte `node.invoke`-Aufrufe, auf Nodes gehostete Plugin-Tools und übergeordnete Plugin-Tools denselben Durchsetzungspfad.

    <Warning>
    Das optionale Feld `scopes` fordert für den Aufruf Gateway-Betreiberbereiche an. OpenClaw berücksichtigt es nur für gebündelte Plugins und vertrauenswürdige Installationen offizieller Plugins; Anforderungen anderer Plugins erweitern die Berechtigungen des Aufrufs nicht. Verwenden Sie es nur, wenn ein vertrauenswürdiges Plugin einen Node-Befehl mit einem strengeren Gateway-Bereich wie `operator.admin` aufrufen muss.
    </Warning>

  </Accordion>
  <Accordion title="api.runtime.tasks">
    Binden Sie den Zustand von Task Flow und Task Run an einen vorhandenen OpenClaw-Sitzungsschlüssel oder einen vertrauenswürdigen Tool-Kontext.

    - `api.runtime.tasks.managedFlows` unterstützt Mutationen: Task Flows erstellen, fortsetzen und abbrechen.
    - `api.runtime.tasks.flows` und `api.runtime.tasks.runs` sind schreibgeschützte DTO-Ansichten für Auflistungen und Statusabfragen; beide stellen `bindSession(...)` / `fromToolContext(...)` sowie `get`, `list`, `findLatest` und `resolve` bereit.

    Task Flow verfolgt den dauerhaften Zustand mehrstufiger Workflows. Es ist kein Planer:
    Verwenden Sie Cron oder `api.session.workflow.scheduleSessionTurn(...)` für zukünftige
    Aktivierungen und anschließend `managedFlows` im geplanten Durchlauf, wenn diese Arbeit
    Flow-Zustand, untergeordnete Tasks, Wartevorgänge oder Abbruchmöglichkeiten benötigt.

    ```typescript
    const taskFlow = api.runtime.tasks.managedFlows.fromToolContext(ctx);

    const created = taskFlow.createManaged({
      controllerId: "my-plugin/review-batch",
      goal: "Neue Pull Requests prüfen",
    });

    const child = taskFlow.runTask({
      flowId: created.flowId,
      runtime: "acp",
      childSessionKey: "agent:main:subagent:reviewer",
      task: "PR #123 prüfen",
      status: "running",
      startedAt: Date.now(),
    });

    const waiting = taskFlow.setWaiting({
      flowId: created.flowId,
      expectedRevision: created.revision,
      currentStep: "await-human-reply",
      waitJson: { kind: "reply", channel: "telegram" },
    });
    ```

    Verwenden Sie `bindSession({ sessionKey, requesterOrigin })`, wenn Sie bereits über einen vertrauenswürdigen OpenClaw-Sitzungsschlüssel aus Ihrer eigenen Bindungsschicht verfügen. Binden Sie ihn nicht aus unverarbeiteten Benutzereingaben.

  </Accordion>
  <Accordion title="api.runtime.tts">
    Text-zu-Sprache-Synthese.

    ```typescript
    // Standard-TTS
    const clip = await api.runtime.tts.textToSpeech({
      text: "Hallo von OpenClaw",
      cfg: api.config,
    });

    // Für Telefonie optimierte TTS
    const telephonyClip = await api.runtime.tts.textToSpeechTelephony({
      text: "Hallo von OpenClaw",
      cfg: api.config,
    });

    // Verfügbare Stimmen auflisten
    const voices = await api.runtime.tts.listVoices({
      provider: "elevenlabs",
      cfg: api.config,
    });
    ```

    Verwendet die zentrale `tts`-Konfiguration und Provider-Auswahl. Gibt einen PCM-Audiopuffer und die Abtastrate zurück. `textToSpeechStream` ist ebenfalls für die Streaming-Synthese verfügbar.

  </Accordion>
  <Accordion title="api.runtime.mediaUnderstanding">
    Bild-, Audio- und Videoanalyse.

    ```typescript
    // Ein Bild beschreiben
    const image = await api.runtime.mediaUnderstanding.describeImageFile({
      filePath: "/tmp/inbound-photo.jpg",
      cfg: api.config,
      agentDir: "/tmp/agent",
    });

    // Audio transkribieren
    const { text } = await api.runtime.mediaUnderstanding.transcribeAudioFile({
      filePath: "/tmp/inbound-audio.ogg",
      cfg: api.config,
      mime: "audio/ogg", // optional, falls der MIME-Typ nicht abgeleitet werden kann
    });

    // Ein Video beschreiben
    const video = await api.runtime.mediaUnderstanding.describeVideoFile({
      filePath: "/tmp/inbound-video.mp4",
      cfg: api.config,
    });

    // Generische Dateianalyse
    const result = await api.runtime.mediaUnderstanding.runFile({
      filePath: "/tmp/inbound-file.pdf",
      cfg: api.config,
    });

    // Strukturierte Bildextraktion über einen bestimmten Provider/ein bestimmtes Modell.
    // Fügen Sie mindestens ein Bild ein; Texteingaben dienen als ergänzender Kontext.
    const evidence = await api.runtime.mediaUnderstanding.extractStructuredWithModel({
      provider: "codex",
      model: "gpt-5.6-sol",
      input: [
        {
          type: "image",
          buffer: receiptImageBuffer,
          fileName: "receipt.png",
          mime: "image/png",
        },
        { type: "text", text: "Bevorzugen Sie den gedruckten Gesamtbetrag gegenüber handschriftlichen Notizen." },
      ],
      instructions: "Extrahieren Sie den Händler, den Gesamtbetrag und durchsuchbare Tags.",
      schemaName: "receipt.evidence",
      jsonSchema: {
        type: "object",
        properties: {
          vendor: { type: "string" },
          total: { type: "number" },
          tags: { type: "array", items: { type: "string" } },
        },
        required: ["vendor", "total"],
      },
      cfg: api.config,
    });
    ```

    Gibt `{ text: undefined }` zurück, wenn keine Ausgabe erzeugt wird (z. B. bei übersprungener Eingabe).

    `describeImageFileWithModel(...)` beschreibt ein bereits bekanntes Bild über einen bestimmten Provider/ein bestimmtes Modell und umgeht dabei die standardmäßige Auflösung des aktiven Modells, die `describeImageFile(...)` verwendet.

  </Accordion>
  <Accordion title="api.runtime.imageGeneration">
    Bilderzeugung.

    ```typescript
    const result = await api.runtime.imageGeneration.generate({
      prompt: "Ein Roboter, der einen Sonnenuntergang malt",
      cfg: api.config,
    });

    const providers = api.runtime.imageGeneration.listProviders({ cfg: api.config });
    ```

  </Accordion>
  <Accordion title="api.runtime.videoGeneration">
    Videoerzeugung entsprechend der Struktur der Bilderzeugung.

    ```typescript
    const result = await api.runtime.videoGeneration.generate({
      prompt: "Eine Drohnenaufnahme, die bei Sonnenaufgang über eine Küste fliegt",
      cfg: api.config,
    });

    const providers = api.runtime.videoGeneration.listProviders({ cfg: api.config });
    ```

  </Accordion>
  <Accordion title="api.runtime.musicGeneration">
    Musikerzeugung entsprechend der Struktur der Bilderzeugung.

    ```typescript
    const result = await api.runtime.musicGeneration.generate({
      prompt: "Ein beschwingter Lo-Fi-Track für eine Programmiersitzung",
      cfg: api.config,
    });

    const providers = api.runtime.musicGeneration.listProviders({ cfg: api.config });
    ```

  </Accordion>
  <Accordion title="api.runtime.webSearch">
    Websuche.

    ```typescript
    const providers = api.runtime.webSearch.listProviders({ config: api.config });

    const result = await api.runtime.webSearch.search({
      config: api.config,
      args: { query: "OpenClaw Plugin-SDK", count: 5 },
    });
    ```

  </Accordion>
  <Accordion title="api.runtime.media">
    Medien-Dienstprogramme auf niedriger Ebene.

    ```typescript
    const webMedia = await api.runtime.media.loadWebMedia(url);
    const mime = await api.runtime.media.detectMime(buffer);
    const kind = api.runtime.media.mediaKindFromMime("image/jpeg"); // "image"
    const isVoice = api.runtime.media.isVoiceCompatibleAudio(filePath);
    const metadata = await api.runtime.media.getImageMetadata(filePath);
    const resized = await api.runtime.media.resizeToJpeg(buffer, { maxWidth: 800 });
    const terminalQr = await api.runtime.media.renderQrTerminal("https://openclaw.ai");
    const pngQr = await api.runtime.media.renderQrPngBase64("https://openclaw.ai", {
      scale: 6, // 1-12
      marginModules: 4, // 0-16
    });
    const pngQrDataUrl = await api.runtime.media.renderQrPngDataUrl("https://openclaw.ai");
    const tmpRoot = resolvePreferredOpenClawTmpDir();
    const pngQrFile = await api.runtime.media.writeQrPngTempFile("https://openclaw.ai", {
      tmpRoot,
      dirPrefix: "my-plugin-qr-",
      fileName: "qr.png",
    });
    ```

  </Accordion>
  <Accordion title="api.runtime.config">
    Aktueller Snapshot der Runtime-Konfiguration und transaktionale Konfigurationsschreibvorgänge. Bevorzugen Sie
    die Konfiguration, die bereits an den aktiven Aufrufpfad übergeben wurde; verwenden Sie
    `current()` nur, wenn der Handler den Prozess-Snapshot direkt benötigt.

    ```typescript
    const cfg = api.runtime.config.current();
    await api.runtime.config.mutateConfigFile({
      afterWrite: { mode: "auto" },
      mutate(draft) {
        draft.plugins ??= {};
      },
    });
    ```

    `mutateConfigFile(...)` und `replaceConfigFile(...)` geben einen `followUp`-Wert
    zurück, beispielsweise `{ mode: "restart", requiresRestart: true, reason }`,
    der die Absicht des Schreibers erfasst, ohne dem Gateway die Kontrolle über den Neustart zu entziehen.

  </Accordion>
  <Accordion title="api.runtime.system">
    Dienstprogramme auf Systemebene.

    ```typescript
    await api.runtime.system.enqueueSystemEvent(event);
    api.runtime.system.requestHeartbeat({
      source: "other",
      intent: "event",
      reason: "plugin-event",
    });
    api.runtime.system.requestHeartbeatNow({ reason: "plugin-event" }); // Veralteter Kompatibilitätsalias.
    const heartbeatResult = await api.runtime.system.runHeartbeatOnce({
      reason: "plugin-triggered-check",
    });
    const output = await api.runtime.system.runCommandWithTimeout(cmd, args, opts);
    const hint = api.runtime.system.formatNativeDependencyHint(pkg);
    ```

    `runHeartbeatOnce(...)` führt sofort einen einzelnen Heartbeat-Zyklus aus und umgeht dabei den normalen Koaleszenz-Timer. Übergeben Sie `{ heartbeat: { target: "last" } }`, um die Zustellung an den zuletzt aktiven Kanal zu erzwingen, anstatt die standardmäßige `target: "none"`-Unterdrückung zu verwenden.

    `runCommandWithTimeout(...)` gibt erfasste `stdout` und `stderr`, optionale
    Kürzungsanzahlen, `code`, `signal`, `killed`, `termination` und
    `noOutputTimedOut` zurück. Ergebnisse bei Zeitüberschreitung und bei Zeitüberschreitung ohne Ausgabe melden `code: 124`,
    wenn der untergeordnete Prozess keinen von null verschiedenen Exit-Code bereitstellt. Signalbedingte Beendigungen
    ohne Zeitüberschreitung können dennoch `code: null` zurückgeben. Verwenden Sie daher `termination` und
    `noOutputTimedOut`, um die Gründe für Zeitüberschreitungen zu unterscheiden.

  </Accordion>
  <Accordion title="api.runtime.events">
    Ereignisabonnements.

    ```typescript
    api.runtime.events.onAgentEvent((event) => {
      /* ... */
    });
    api.runtime.events.onSessionTranscriptUpdate((update) => {
      /* ... */
    });
    ```

  </Accordion>
  <Accordion title="api.runtime.logging">
    Protokollierung.

    ```typescript
    const verbose = api.runtime.logging.shouldLogVerbose();
    const childLogger = api.runtime.logging.getChildLogger({ plugin: "my-plugin" }, { level: "debug" });
    ```

  </Accordion>
  <Accordion title="api.runtime.modelAuth">
    Auflösung der Modell- und Provider-Authentifizierung.

    ```typescript
    const auth = await api.runtime.modelAuth.getApiKeyForModel({ model, cfg });

    // Anforderungsbereite Authentifizierung, einschließlich Provider-Laufzeitaustausch (z. B. OAuth-Aktualisierung)
    const runtimeAuth = await api.runtime.modelAuth.getRuntimeAuthForModel({ model, cfg });

    const providerAuth = await api.runtime.modelAuth.resolveApiKeyForProvider({
      provider: "openai",
      cfg,
    });
    ```

  </Accordion>
  <Accordion title="api.runtime.state">
    Auflösung des Zustandsverzeichnisses und SQLite-gestützter schlüsselbasierter Speicher.

    ```typescript
    const stateDir = api.runtime.state.resolveStateDir(process.env);
    const store = api.runtime.state.openKeyedStore<MyRecord>({
      namespace: "my-feature",
      maxEntries: 200,
      defaultTtlMs: 15 * 60_000,
    });

    await store.register("key-1", { value: "hello" });
    const claimed = await store.registerIfAbsent("dedupe-key", { value: "first" });
    const value = await store.lookup("key-1");
    await store.deleteIf?.("key-1", (current) => current.value === "hello");
    await store.consume("key-1");
    await store.clear();

    const blobs = api.runtime.state.openBlobStore<MyBlobMetadata>({
      namespace: "rendered-artifacts",
      maxEntries: 100,
      maxBytesPerEntry: 4 * 1024 * 1024,
      maxBytesPerNamespace: 64 * 1024 * 1024,
      defaultTtlMs: 15 * 60_000,
    });
    await blobs.register(
      "artifact-1",
      new TextEncoder().encode("binary or text payload"),
      { contentType: "text/plain" },
    );
    const blob = await blobs.lookup("artifact-1");

    await api.runtime.state.withLease(
      {
        namespace: "my-feature",
        key: "writer",
        database: { scope: "agent", agentId },
        leaseMs: 5 * 60_000,
        waitMs: 30_000,
      },
      async ({ signal, assertOwned }) => {
        await runExternalWriter({ signal });
        assertOwned();
      },
    );
    ```

    Schlüsselbasierte Speicher überstehen Neustarts und werden anhand der zur Laufzeit gebundenen Plugin-ID isoliert. Verwenden Sie `registerIfAbsent(...)` für atomare Deduplizierungsansprüche: Die Methode gibt `true` zurück, wenn der Schlüssel fehlte oder abgelaufen war und registriert wurde, oder `false`, wenn bereits ein aktiver Wert vorhanden ist, ohne dessen Wert, Erstellungszeit oder TTL zu überschreiben. Verwenden Sie `deleteIf(...)`, wenn eine Bereinigung nur den zuvor beobachteten Wert entfernen darf; das synchrone Prädikat und die Löschung werden in einer einzigen SQLite-Transaktion ausgeführt. Grenzwerte: `maxEntries` pro Namensraum, 50,000 aktive Zeilen pro Plugin, JSON-Werte unter 64KB und optionaler TTL-Ablauf. Standardmäßig entfernt ein Schreibvorgang beim Erreichen eines der Zeilengrenzwerte die ältesten aktiven Zeilen aus dem Namensraum, in den geschrieben wird; gleichgeordnete Namensräume werden für diesen Schreibvorgang nicht verdrängt, und der Schreibvorgang schlägt weiterhin fehl, wenn der Namensraum nicht genügend Zeilen freigeben kann. Legen Sie `overflowPolicy: "reject-new"` für dauerhafte Eigentumsdatensätze fest, die niemals verdrängt werden dürfen: Neue Schlüssel schlagen bei beiden Grenzwerten fehl, während vorhandene Schlüssel weiterhin aktualisiert werden können.

    `openSyncKeyedStore<T>(...)` gibt dieselbe Speicherstruktur mit synchronen Methoden zurück (`register`, `registerIfAbsent`, `deleteIf`, `lookup`, `consume`, `clear` geben alle Werte direkt statt als Promises zurück) und ist für Aufrufer vorgesehen, die nicht warten können.

    `openBlobStore<TMetadata>(...)` speichert begrenzte Binärnutzlasten ohne Base64 oder Datei-Sidecars in gemeinsam genutztem SQLite. Dies erfordert Byte- und Zeilengrenzwerte pro Eintrag und pro Namensraum, kopiert Byte-Arrays an der API-Grenze und listet Metadaten auf, ohne jedes BLOB zu laden. `register(...)` ist ein explizites Upsert, auch für abgelaufene Schlüssel. `registerIfAbsent(...)` ermöglicht eine kollisionssichere Erstellung: Ein abgelaufener Schlüssel bleibt belegt, bis sein Eigentümer ihn mit `deleteExpiredKey(key)` oder `deleteExpired()` beansprucht. Dadurch bleiben Metadaten erhalten, die erforderlich sind, um zugehörige benannte Artefakte nach dem SQLite-Commit zu entfernen. Jede Zeile mit einer TTL ist temporär und wird selbst vor ihrem Ablauf von Sicherung und Wiederherstellung ausgeschlossen; lassen Sie die TTL für einen dauerhaften, wiederherstellbaren Zustand weg. Host-Sicherungen begrenzen jedes BLOB auf 100 MiB, jedes Plugin auf 512 MiB physisch gespeicherter BLOBs und jedes Plugin auf 50,000 physisch gespeicherte Zeilen, einschließlich abgelaufener Zeilen, die auf eine Bereinigung durch den Eigentümer warten. Verwenden Sie `registerIfAbsent(...)` mit `overflowPolicy: "reject-new"`, wenn externe Materialisierungen durch Ersetzung oder Verdrängung nicht unbemerkt verwaisen dürfen.

    `openChannelIngressQueue<TPayload>(...)` öffnet eine persistierte Eingangswarteschlange im Geltungsbereich des aufrufenden Plugins, um eingehende Ereignisse zu puffern, die über Neustarts hinweg mindestens einmal verarbeitet werden müssen. Wenn die Wiederherstellung veralteter Ansprüche `shouldRecover` verwendet, geben Sie außerdem `shouldRecoverCorrupt` an, falls beschädigte beanspruchte Nutzlasten unter Quarantäne gestellt werden sollen: Die von der Nutzlast unabhängige Anspruchsidentität ermöglicht es dem Plugin, die Richtlinien des aktiven Eigentümers und der Lane beizubehalten, bevor die Warteschlange die Zeile mit einem Tombstone markiert.

    `withLease(...)` serialisiert kooperative Plugin-Arbeit über OpenClaw-Prozesse hinweg. Wählen Sie `database: { scope: "shared" }` für einen globalen Eigentümer oder `{ scope: "agent", agentId }` für eine unabhängige Eigentümerschaft pro Agent. Leiten Sie das `AbortSignal` des Callbacks an jede Operation weiter, die fehlschlagen kann. `assertOwned()` ist ein punktueller Prüfpunkt vor dem Beginn eines weiteren wichtigen Schritts; der Host überprüft die Eigentümerschaft außerdem nach dem Callback. Bei Verlust des Leases oder Abbruch durch den Aufrufer wird das Signal abgebrochen. Erwerbswartezeiten und Heartbeats erfolgen außerhalb kurzer synchroner SQLite-Transaktionen; Plugins erhalten niemals Datenbankpfade oder Handles. Dies ist eine kooperative Abbruchsteuerung, kein Fencing-Token und keine Autorisierung für nicht abgesicherte externe Schreibvorgänge.

    `openChannelIngressDrain(...)` öffnet den kanalunabhängigen Kern-Worker über dieser Warteschlange (oder erstellt eine Warteschlange, wenn keine angegeben wird). Der Drain verwaltet die Wiederherstellung veralteter Ansprüche, die Serialisierung von Ansprüchen pro Lane, den Abschluss bei Übernahme oder bei Rückkehr des Dispatches, die Wiederholungs-/Dead-Letter-Disposition, optionales Ersetzen vor der Übernahme sowie das Zeitlimit für einen Stillstand zwischen Anspruch und Übernahme. Binden Sie die Anspruchseigentümerschaft mit `turnAdoptionLifecycle` in die Antwortgenerierung ein (über `bindIngressLifecycleToReplyOptions` aus `plugin-sdk/channel-outbound`). Kanal-Plugins verwalten das Einreihen auf der Annahmeseite, die Lane-Ableitung, die Klassifizierung als nicht wiederholbar und alle Autorisierungsrichtlinien für das Ersetzen.

    <Warning>
    `openBlobStore`, `openKeyedStore`, `openSyncKeyedStore`, `withLease`, `openChannelIngressQueue` und `openChannelIngressDrain` sind in dieser Version nur für gebündelte Plugins und vertrauenswürdige offizielle Plugin-Installationen verfügbar.
    </Warning>

  </Accordion>
  <Accordion title="api.runtime.channel">
    Kanalspezifische Laufzeit-Hilfsfunktionen (verfügbar, wenn ein Kanal-Plugin geladen ist). Nach Aufgabenbereich gruppiert:

    | Gruppe | Zweck |
    | --- | --- |
    | `text` | Aufteilung (`chunkText`, `chunkMarkdownText`, `resolveChunkMode`), Erkennung von Steuerungsbefehlen, Konvertierung von Markdown-Tabellen. |
    | `reply` | Gepufferter blockweiser Antwortversand, Umschlagformatierung, Auflösung der effektiven Nachrichten-/Verzögerungskonfiguration für Menschen. |
    | `routing` | `buildAgentSessionKey`, `resolveAgentRoute`. |
    | `pairing` | `buildPairingReply`, Lesen/Entfernen von Positivlisten, Upserts für Kopplungsanfragen und aus Anfragen abgeleitete Genehmigungseinträge. |
    | `media` | Herunterladen/Speichern entfernter Medien (siehe unten). |
    | `activity` | Letzte Kanalaktivität aufzeichnen/lesen. |
    | `session` | Sitzungsmetadaten aus eingehenden Ereignissen, Aktualisierungen der letzten Route. |
    | `mentions` | Hilfsfunktionen für Erwähnungsrichtlinien (siehe unten). |
    | `reactions` | Handles für Bestätigungsreaktionen als Verarbeitungsindikatoren für laufende Vorgänge. |
    | `groups` | Auflösung von Gruppenrichtlinie und Erwähnungspflicht. |
    | `debounce` | Entprellung eingehender Nachrichten. |
    | `commands` | Befehlsautorisierung und Steuerung von Textbefehlen. |
    | `outbound` | Ausgangsadapter eines Kanals laden. |
    | `inbound` | Kontext für eingehende Ereignisse erstellen und den gemeinsam genutzten Kern für eingehende Ereignisse/Antworten ausführen. |
    | `threadBindings` | Leerlaufzeitlimit/maximales Alter für gebundene Sitzungsthreads anpassen. |
    | `runtimeContexts` | Prozesslokalen Kontext pro Kanal/Konto/Fähigkeit registrieren, lesen und überwachen. |

    `api.runtime.channel.media` ist die bevorzugte Schnittstelle für das Herunterladen und Speichern von Kanalmedien:

    ```typescript
    const saved = await api.runtime.channel.media.saveRemoteMedia({
      url,
      subdir: "inbound",
      maxBytes,
      filePathHint: fileName,
    });
    ```

    Verwenden Sie `saveRemoteMedia(...)`, wenn eine entfernte URL zu OpenClaw-Medien werden soll. Verwenden Sie `saveResponseMedia(...)`, wenn das Plugin bereits ein `Response` mit Plugin-eigener Authentifizierung sowie eigener Umleitungs- oder Positivlistenbehandlung abgerufen hat. Verwenden Sie `readRemoteMediaBuffer(...)` nur, wenn das Plugin Rohbytes zur Prüfung, Transformation, Entschlüsselung oder zum erneuten Hochladen benötigt. `fetchRemoteMedia(...)` bleibt ein veralteter Kompatibilitätsalias für `readRemoteMediaBuffer(...)`.

    `api.runtime.channel.mentions` ist die gemeinsam genutzte Schnittstelle für Richtlinien zu eingehenden Erwähnungen für gebündelte Kanal-Plugins, die Laufzeitinjektion verwenden:

    ```typescript
    const mentionMatch = api.runtime.channel.mentions.matchesMentionWithExplicit(text, {
      mentionRegexes,
      mentionPatterns,
    });

    const decision = api.runtime.channel.mentions.resolveInboundMentionDecision({
      facts: {
        canDetectMention: true,
        wasMentioned: mentionMatch.matched,
        implicitMentionKinds: api.runtime.channel.mentions.implicitMentionKindWhen(
          "reply_to_bot",
          isReplyToBot,
        ),
      },
      policy: {
        isGroup,
        requireMention,
        allowTextCommands,
        hasControlCommand,
        commandAuthorized,
      },
    });
    ```

    Verfügbare Hilfsfunktionen für Erwähnungen:

    - `buildMentionRegexes`
    - `matchesMentionPatterns`
    - `matchesMentionWithExplicit`
    - `implicitMentionKindWhen`
    - `resolveInboundMentionDecision`

    Verwenden Sie den normalisierten `{ facts, policy }`-Pfad für Erwähnungsentscheidungen.

    Mehrere Felder unter `reply`, `session` und `inbound` enthalten feldspezifische `@deprecated`-Hinweise, die auf den aktuellen Kernel für Kanalvorgänge oder die Ausgangsadapter des Kanals verweisen; prüfen Sie die Inline-JSDoc der jeweiligen Hilfsfunktion, bevor Sie darauf neuen Code aufbauen.

  </Accordion>
</AccordionGroup>

## Speichern von Laufzeitreferenzen

Verwenden Sie `createPluginRuntimeStore`, um die Laufzeitreferenz für die Verwendung außerhalb des `register`-Callbacks zu speichern:

<Steps>
  <Step title="Speicher erstellen">
    ```typescript
    import { createPluginRuntimeStore } from "openclaw/plugin-sdk/runtime-store";
    import type { PluginRuntime } from "openclaw/plugin-sdk/runtime-store";

    const store = createPluginRuntimeStore<PluginRuntime>({
      pluginId: "my-plugin",
      errorMessage: "my-plugin runtime not initialized",
    });
    ```

  </Step>
  <Step title="Mit dem Einstiegspunkt verbinden">
    ```typescript
    export default defineChannelPluginEntry({
      id: "my-plugin",
      name: "My Plugin",
      description: "Example",
      plugin: myPlugin,
      setRuntime: store.setRuntime,
    });
    ```
  </Step>
  <Step title="Aus anderen Dateien darauf zugreifen">
    ```typescript
    export function getRuntime() {
      return store.getRuntime(); // throws if not initialized
    }

    export function tryGetRuntime() {
      return store.tryGetRuntime(); // returns null if not initialized
    }
    ```

  </Step>
</Steps>

<Note>
Bevorzugen Sie `pluginId` für die Identität des Laufzeitspeichers. Die untergeordnete Form `key` ist für seltene Fälle vorgesehen, in denen ein Plugin absichtlich mehr als einen Laufzeit-Slot benötigt.
</Note>

## Weitere `api`-Felder der obersten Ebene

Zusätzlich zu `api.runtime` stellt das API-Objekt außerdem Folgendes bereit:

<ParamField path="api.id" type="string">
  Plugin-ID.
</ParamField>
<ParamField path="api.name" type="string">
  Anzeigename des Plugins.
</ParamField>
<ParamField path="api.config" type="OpenClawConfig">
  Aktueller Konfigurations-Snapshot (sofern verfügbar, der aktive In-Memory-Laufzeit-Snapshot).
</ParamField>
<ParamField path="api.pluginConfig" type="Record<string, unknown>">
  Pluginspezifische Konfiguration aus `plugins.entries.<id>.config`.
</ParamField>
<ParamField path="api.logger" type="PluginLogger">
  Bereichsspezifischer Logger (`debug`, `info`, `warn`, `error`).
</ParamField>
<ParamField path="api.registrationMode" type="PluginRegistrationMode">
  Aktueller Lademodus: `"full"` (Live-Aktivierung), `"discovery"` / `"tool-discovery"` (schreibgeschützte Funktionsermittlung), `"setup-only"` (leichtgewichtiger Einrichtungseinstieg), `"setup-runtime"` (Einrichtungsablauf, der zusätzlich den Laufzeit-Kanaleinstieg benötigt) oder `"cli-metadata"` (Erfassung von CLI-Befehlsmetadaten).
</ParamField>
<ParamField path="api.resolvePath(input)" type="(string) => string">
  Einen Pfad relativ zum Plugin-Stammverzeichnis auflösen.
</ParamField>

## Verwandte Themen

- [Plugin-Interna](/de/plugins/architecture) — Funktionsmodell und Registry
- [SDK-Einstiegspunkte](/de/plugins/sdk-entrypoints) — Optionen für `definePluginEntry`
- [SDK-Übersicht](/de/plugins/sdk-overview) — Unterpfadreferenz
