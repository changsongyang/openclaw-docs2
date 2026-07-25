---
read_when:
    - Sie erstellen ein neues Plugin für einen Nachrichtenkanal
    - Sie möchten OpenClaw mit einer Messaging-Plattform verbinden
    - Sie müssen die Adapteroberfläche von ChannelPlugin verstehen
sidebarTitle: Channel Plugins
summary: Schritt-für-Schritt-Anleitung zum Erstellen eines Messaging-Kanal-Plugins für OpenClaw
title: Channel-Plugins entwickeln
x-i18n:
    generated_at: "2026-07-24T20:53:32Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 0ff8ad04346babf3eece7e10bd38946ee290947b2e504b6b5ca438865531bf38
    source_path: plugins/sdk-channel-plugins.md
    workflow: 16
---

Dieser Leitfaden erstellt ein Kanal-Plugin, das OpenClaw mit einer Messaging-
Plattform verbindet: DM-Sicherheit, Kopplung, Antwort-Threads und ausgehende Nachrichten.

<Info>
  Neu bei OpenClaw-Plugins? Lesen Sie zuerst [Erste Schritte](/de/plugins/building-plugins),
  um sich mit Paketstruktur und Manifest-Einrichtung vertraut zu machen.
</Info>

## Zuständigkeiten Ihres Plugins

Kanal-Plugins implementieren keine Tools zum Senden, Bearbeiten oder Reagieren; der Kern stellt ein
gemeinsames `message`-Tool bereit. Ihr Plugin ist zuständig für:

- **Konfiguration** - Kontoauflösung und Einrichtungsassistent
- **Sicherheit** - DM-Richtlinie und Zulassungslisten
- **Kopplung** - DM-Genehmigungsablauf
- **Sitzungsgrammatik** - Zuordnung Provider-spezifischer Konversations-IDs zu Basis-
  Chats, Thread-IDs und übergeordneten Rückfalloptionen
- **Ausgehend** - Senden von Text, Medien und Umfragen an die Plattform
- **Thread-Verknüpfung** - Verknüpfung von Antworten in Threads
- **Heartbeat-Tippanzeige** - optionale Tipp-/Beschäftigt-Signale für Heartbeat-Zustellungs-
  ziele

Der Kern ist zuständig für das gemeinsame Nachrichtentool, die Prompt-Anbindung, die äußere Form des Sitzungsschlüssels,
die allgemeine `:thread:`-Verwaltung und die Weiterleitung.

## Nachrichtenadapter

Stellen Sie einen `message`-Adapter mit `defineChannelMessageAdapter` aus
`openclaw/plugin-sdk/channel-outbound` bereit. Deklarieren Sie nur die dauerhaften Funktionen für den endgültigen Versand,
die Ihr nativer Transport tatsächlich unterstützt, abgesichert durch einen Vertragstest,
der den nativen Nebeneffekt und den zurückgegebenen Beleg nachweist. Leiten Sie Text-/Medien-
sendungen an dieselben Transportfunktionen weiter, die der bisherige `outbound`-Adapter verwendet. Den
vollständigen API-Vertrag, die Funktionsmatrix, Belegregeln, die Finalisierung der Live-Vorschau,
die Richtlinie für Empfangsbestätigungen, Tests und die Migrationstabelle finden Sie unter
[API für ausgehende Kanalnachrichten](/de/plugins/sdk-channel-outbound).

Wenn Ihr vorhandener `outbound`-Adapter bereits die richtigen Sendemethoden und
Funktionsmetadaten besitzt, leiten Sie den `message`-Adapter mit
`createChannelMessageAdapterFromOutbound(...)` ab, anstatt eine weitere
Brücke manuell zu schreiben. Adapter-Sendevorgänge geben `MessageReceipt`-Werte zurück. Leiten Sie ältere IDs
mit `listMessageReceiptPlatformIds(...)` oder
`resolveMessageReceiptPrimaryId(...)` ab, anstatt parallele `messageIds`-
Felder beizubehalten.

Deklarieren Sie Live- und Finalisierungsfunktionen präzise - der Kern entscheidet anhand dieser Angaben,
welche Aktionen ein Kanal ausführen kann, und Abweichungen zwischen deklariertem und tatsächlichem Verhalten führen zum
Fehlschlagen eines Vertragstests:

| Oberfläche                            | Werte                                                                                            |
| ------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `message.live.capabilities`           | `draftPreview`, `previewFinalization`, `progressUpdates`, `nativeStreaming`, `quietFinalization` |
| `message.live.finalizer.capabilities` | `finalEdit`, `normalFallback`, `discardPending`, `previewReceipt`, `retainOnAmbiguousFailure`    |

Kanäle, die einen Vorschauentwurf direkt finalisieren, sollten die Laufzeitlogik
über `defineFinalizableLivePreviewAdapter(...)` zusammen mit
`deliverWithFinalizableLivePreviewAdapter(...)` führen und die deklarierten
Funktionen durch `verifyChannelMessageLiveCapabilityAdapterProofs(...)`-
und `verifyChannelMessageLiveFinalizerProofs(...)`-Tests absichern, damit das Verhalten von nativer Vorschau,
Fortschritt, Bearbeitung, Rückfalloption/Aufbewahrung, Bereinigung und Belegen nicht unbemerkt
abweichen kann.

Eingangsempfänger, die Plattformbestätigungen verzögern, sollten
`message.receive.defaultAckPolicy` und `supportedAckPolicies` deklarieren, anstatt
den Zeitpunkt der Bestätigung in einem lokalen Monitorstatus zu verbergen. Decken Sie jede deklarierte Richtlinie mit
`verifyChannelMessageReceiveAckPolicyAdapterProofs(...)` ab.

Ältere Antworthilfen wie `dispatchInboundReplyWithBase` und
`recordInboundSessionAndDispatchReply` bleiben für kompatible
Dispatcher verfügbar. Verwenden Sie sie nicht für neuen Kanalcode; beginnen Sie stattdessen mit dem `message`-
Adapter, Belegen und Lebenszyklushilfen für Empfang und Versand auf
`openclaw/plugin-sdk/channel-outbound`.

### Eingangsverarbeitung (experimentell)

Kanäle, die ihre Autorisierung eingehender Nachrichten migrieren, können den experimentellen
`openclaw/plugin-sdk/channel-ingress-runtime`-Unterpfad aus den Empfangspfaden der Laufzeit
verwenden. Er akzeptiert Plattformfakten, unaufbereitete Zulassungslisten, Routendeskriptoren, Befehls-
fakten und die Konfiguration von Zugriffsgruppen und gibt anschließend Absender-/Routen-/Befehls-/Aktivierungs-
projektionen sowie den geordneten Eingangsgraphen zurück, während Plattformabfragen und Neben-
effekte im Plugin verbleiben. Belassen Sie die Normalisierung der Plugin-Identität in dem
Deskriptor, den Sie an den Resolver übergeben; serialisieren Sie keine unaufbereiteten Vergleichswerte aus
dem aufgelösten Status oder der Entscheidung. Unter
[API für Kanaleingänge](/de/plugins/sdk-channel-ingress) finden Sie das API-Design,
die Zuständigkeitsgrenze und die Testerwartungen.

### Dauerhafte Eingangsverarbeitung und Deduplizierung bei Wiederholung

Kanäle, die eine dauerhafte Eingangsverarbeitung einführen, sollten `createChannelIngressMonitor`
aus `openclaw/plugin-sdk/channel-outbound` verwenden, sofern sie keinen wesentlich
anderen Zulassungs- oder Pumpvertrag benötigen. Stellen Sie den unaufbereiteten Transportumschlag an einem
einzelnen Empfangsengpass in die Warteschlange (keine Normalisierung zum Empfangszeitpunkt), machen Sie bei Webhook-Transporten
die Transportbestätigung vom dauerhaften Anhängen abhängig, leiten Sie pro Konversation eine
serialisierte Spur ab und markieren Sie das Ereignis bei der Übernahme durch die Weiterleitung als abgeschlossen.
Der Primärschlüssel der Warteschlange lautet `(queue_name, event_id)`; beim Abschluss
wird der Datensatz mit einem Tombstone versehen, statt ihn zu löschen, sodass eine verspätete erneute Plattformzustellung
derselben `event_id` für das Aufbewahrungsfenster des Tombstones dauerhaft abgelehnt wird.
Die Monitor-API und den Vertrag zum Herunterfahren finden Sie unter [API für ausgehende Kanalnachrichten](/de/plugins/sdk-channel-outbound#durable-ingress-monitors).

Dieser Tombstone ist die Schichtungsregel für Wiederholungsschutzmechanismen
(`openclaw/plugin-sdk/persistent-dedupe`): Ein geleerter Kanal behält nur dann einen separaten
Wiederholungsschutz bei, wenn dessen Identität oder Aufbewahrungsdauer die der Warteschlange
überschreitet — also bei einem logischen Nachrichtenschlüssel, der sich von der Transportzustellungs-ID unterscheidet (Telegram
dedupliziert `chat_id:message_id`, da Debounce-Zusammenführungen eine Nachricht
unter einer neuen `update_id` erneut sichtbar machen können), oder bei einem längeren Fenster als der Tombstone-
Aufbewahrung des Kanals. Wenn Ihr Schutzschlüssel der `event_id` des Leervorgangs entspräche, löschen Sie den
Schutz bei der Einführung des Leervorgangs und dimensionieren Sie `completedTtlMs`/`completedMaxEntries`
stattdessen so, dass sie das bisherige Schutzfenster abdecken. Schutzmechanismen ohne Deduplizierung, etwa Alters-
grenzen, stehen mit dieser Regel nicht in Zusammenhang. Stabile IDs ausgehender Nachrichten verwenden die gemeinsame
Registrierung für ausgehende Echos aus `openclaw/plugin-sdk/channel-outbound` statt eines
kanallokalen TTL-Caches.

#### Transportklassen und Aufbewahrung

Klassifizieren Sie einen Transport anhand der Wiederherstellungsgarantie an seiner Empfangsgrenze:

- **Bestätigungsabhängige Webhook- oder Ereigniszustellung:** Bestätigen Sie die Zustellung oder geben Sie Erfolg erst
  nach dem dauerhaften Anhängen zurück. Ein Fehler beim Anhängen muss dafür sorgen, dass die Zustellung weiterhin
  wiederholt werden kann, oder die Empfangsgrenze muss fehlschlagen. Zu dieser Klasse gehören Slack, SMS, Zalo,
  Microsoft Teams, Google Chat, LINE und Synology Chat.
- **Abgewartete Polling- oder Stream-Zustellung:** Bewegen Sie den entfernten Cursor erst weiter oder senden Sie die
  Transportbestätigung erst nach dem Anhängen. Wenn kein expliziter Cursor vorhanden ist, muss der
  Empfangs-Callback serialisiert und abgewartet bleiben, damit ein Fehler beim Anhängen die
  Empfangsschleife nicht vorauseilen lässt. Telegram-Polling, Signal und Tlon verwenden diese Klasse;
  für die Telegram-Webhook-Zustellung gilt die oben beschriebene bestätigungsabhängige Regel.
- **Sockets ohne Wiederholungsmöglichkeit:** IRC, Mattermost, Twitch und Zalo Personal können die
  Plattform nicht auffordern, ein angenommenes Ereignis erneut zuzustellen. Ihre dauerhafte Warteschlange schützt das
  Zeitfenster eines Prozessabsturzes und unterstützt die lokale Wiederherstellung nach einem Neustart; Abschluss-
  Tombstones sind gegenüber Plattformwiederholungen nahezu wirkungslos.

Verwenden Sie 30 Tage als flottenweite Konvention für die Tombstone-TTL, nicht als SDK-Standardwert. Ein
Wiederholungsfenster mit hohem Volumen verwendet normalerweise eine Obergrenze von 20,000 abgeschlossenen Einträgen;
abgewartete Transporte mit geringerem Volumen und Transporte ohne Wiederholungsmöglichkeit verwenden normalerweise 1,000-2,000.
Zu den aktuellen Ausnahmen gehören die Obergrenzen von 4,096 Einträgen bei LINE, die 24-stündige Abschluss-
TTL bei SMS und die ausschließlich auf einer Obergrenze basierende Aufbewahrung abgeschlossener Einträge bei Tlon. Obergrenzen für fehlgeschlagene Datensätze können ebenfalls niedriger
als die Obergrenzen abgeschlossener Datensätze sein. Sowohl TTL als auch Obergrenze entfernen Datensätze, sodass die effektive Aufbewahrung
endet, sobald die erste Grenze erreicht wird. Weichen Sie nur aufgrund eines dokumentierten Wiederholungshorizonts der Plattform,
eines beibehaltenen ausgelieferten Wiederholungsschutzfensters, des erwarteten Volumens oder Speicherbudgets
oder eines Transports ohne Wiederholungsmöglichkeit davon ab und sichern Sie den Aufbewahrungsvertrag durch Tests ab.

#### Mindestens einmal ausgeführte Nebeneffekte

Die Weiterleitung beim Leeren führt Befehlsnebeneffekte aus, bevor der Eingangsdatensatz seinen
Abschluss-Tombstone erreicht. Ein Prozessabsturz zwischen diesen Schritten wiederholt den Datensatz und
kann den Nebeneffekt erneut ausführen. Dieses Absturzfenster mit mindestens einmaliger Ausführung ist der
Standardvertrag. Verwenden Sie für nicht idempotente Vorgänge wie Konfigurationsschreibvorgänge, Speicher-
bereinigungen oder sichtbare Bestätigungen außerhalb der Antwortspur
`createIngressEffectOnce(...)` aus
`openclaw/plugin-sdk/ingress-effect-once`. Übergeben Sie jedem Aufruf die stabile Eingangs-
`eventId` sowie einen Effektnamen. Erstellen Sie eine Hilfsfunktion pro Eingangswarteschlange/Konto und
verwenden Sie eine stabile, eindeutige `namespacePrefix` für diesen Geltungsbereich, da Transportereignis-
IDs warteschlangenlokal sein können. Die Hilfsfunktion schreibt ihren dauerhaften Anspruch erst fest, nachdem der
Effekt erfolgreich war; ein ausgelöster Effektfehler gibt den Anspruch frei, sodass ein erneuter Leerversuch
ihn nochmals ausführen kann, während gleichzeitige Aufrufer auf den aktiven Anspruch warten. Fehler des dauerhaften
Status rufen `onDiskError` auf, sofern angegeben, und führen zur Ablehnung, anstatt auf den
Prozessspeicher zurückzugreifen.

Setzen Sie `ttlMs` der Hilfsfunktion mindestens auf die Aufbewahrungsdauer der Eingangs-Tombstones des Kanals
zuzüglich der maximalen Verzögerung zwischen dem Festschreiben des Effekts und dem Abschluss des Datensatzes, einschließlich
begrenzter Ausfallzeiten und erneuter Leerversuche. Die TTL des Effektdatensatzes beginnt beim Festschreiben,
während die Tombstone-Aufbewahrung erst später beim Abschluss beginnt; wenn die Lebensdauer ausstehender Datensätze
unbegrenzt ist, kann keine endliche TTL beliebige Ausfallzeiten abdecken. Nachdem der Tombstone
den Datensatz nicht mehr wiederholen kann, sind ältere Effektdatensätze überflüssig. Dimensionieren Sie
`stateMaxEntries` für jeden eindeutigen Ereignis-/Effektschlüssel, der in diesem
Aufbewahrungsfenster vorhanden sein kann, und berücksichtigen Sie dabei die Obergrenze abgeschlossener Einträge der Warteschlange sowie die
maximale Anzahl von Effekten pro Ereignis. Eine niedrigere Obergrenze entfernt den ältesten Datensatz vor Ablauf seiner TTL
und ermöglicht die erneute Ausführung dieses Effekts. Verbleibende Fenster mit mindestens einmaliger Ausführung bestehen,
wenn der Prozess beendet wird oder die Persistierung fehlschlägt, nachdem der Effekt erfolgreich war, aber bevor
der Anspruch festgeschrieben wird, oder wenn der Datensatz abläuft, während sein Eingangsdatensatz noch
aussteht.

#### Vertrag für kontobezogene Neustarts

Änderungen an der Kanalkonfiguration starten standardmäßig den gesamten Kanal neu. Ein Kanal mit mehreren Konten
darf `reload.accountScopedRestart: true` nur festlegen, wenn die Konfigurations-
auflösung kanalweite gemeinsame Felder sowie das ausgewählte Konto liest, niemals ein
benachbartes Konto, und der Gateway eine einzelne `(channel, accountId)`-
Laufzeit stoppen und starten kann, ohne benachbarte Laufzeiten zu ersetzen.

Der begrenzte Pfad gilt nur für Änderungen unter
`channels.<channel>.accounts.<non-default-id>.*`. Änderungen an gemeinsamen Kanal-
feldern, `accounts.default`, entfernten oder nicht auflösbaren Konten sowie gemischte Änderungen,
die sich auf die Vererbung auswirken können, werden zu einem Neustart des gesamten Kanals hochgestuft. Plugins,
die sich nicht explizit dafür entscheiden, verwenden stets den Pfad für den gesamten Kanal.

Bei Kanälen, die den dauerhaften Eingangs-Leervorgang verwenden, muss der Stopppfad des Kontomonitors
zunächst alle angenommenen Transportzulassungen abschließen und anschließend seinen
Leervorgang freigeben und abwarten. Beim Starten des Kontos wird dieselbe kontoschlüsselbezogene Warteschlange geöffnet, deren anfänglicher
Leervorgang noch nicht weitergeleitete dauerhafte Datensätze wiederherstellt. Fügen Sie keinen zweiten, ausschließlich für das erneute Laden bestimmten
Wiederholungsdurchlauf hinzu; die Wiederherstellung der Warteschlange ist der kanonische Neustartpfad.

Behandeln Sie dieses Flag als Funktionszusage und nicht als Leistungsvorliebe. Vertrags-
tests sollten nachweisen, dass das Hinzufügen und Bearbeiten eines benannten Kontos die aufgelöste
Konfiguration eines benachbarten Kontos unverändert lässt, dass das Stoppen eines Kontos nur den Monitor und Leervorgang
dieses Kontos abschließt und dass ein neuer Monitor die Datensätze dieses Kontos genau
einmal wiederherstellt. Wenn eine Garantie nicht nachgewiesen werden kann, lassen Sie das Flag weg.

### Tippanzeigen

Wenn Ihr Kanal Tippanzeigen außerhalb eingehender Antworten unterstützt, stellen Sie
`heartbeat.sendTyping(...)` im Kanal-Plugin bereit. Der Kern ruft sie mit dem
aufgelösten Heartbeat-Zustellungsziel auf, bevor der Heartbeat-Modelllauf beginnt, und
verwendet den gemeinsamen Lebenszyklus für Aufrechterhaltung und Bereinigung der Tippanzeige. Fügen Sie
`heartbeat.clearTyping(...)` hinzu, wenn die Plattform ein explizites Stoppsignal benötigt.

### Parameter für Medienquellen

Wenn Ihr Kanal dem Nachrichtentool Parameter hinzufügt, die Medienquellen enthalten, stellen Sie
diese Parameternamen über `plugin.actions.describeMessageTool(...).mediaSourceParams` bereit.
Der Kern verwendet diese explizite Liste für die Normalisierung von Sandbox-Pfaden und die Richtlinie für den Zugriff auf
ausgehende Medien, sodass Plugins keine Sonderfälle im gemeinsamen Kern für
Provider-spezifische Avatar-, Anhangs- oder Titelbildparameter benötigen.

Bevorzugen Sie eine aktionsschlüsselbasierte Zuordnung wie `{ "set-profile": ["avatarUrl", "avatarPath"] }`,
damit nicht zusammenhängende Aktionen nicht die Medienargumente einer anderen Aktion übernehmen. Ein flaches Array
funktioniert weiterhin für Parameter, die absichtlich von jeder bereitgestellten Aktion gemeinsam verwendet werden.

Kanäle, die eine temporäre öffentliche URL für einen plattformseitigen
Medienabruf bereitstellen müssen, können `createHostedOutboundMediaStore(...)` aus
`openclaw/plugin-sdk/outbound-media` mit Plugin-Zustandsspeichern verwenden. Belassen Sie die
Analyse der Plattformroute und die Token-Durchsetzung im Kanal-Plugin; der gemeinsame Helfer
ist nur für das Laden von Medien, Ablaufmetadaten, Chunk-Zeilen und die Bereinigung zuständig.

Eingehende Anhänge verwenden geordnete Fakten, keine parallelen `Media*`-Felder. Normalisieren Sie
Kanaldatensätze mit `toInboundMediaFacts(...)` aus
`openclaw/plugin-sdk/channel-inbound` und übergeben Sie sie beim Erstellen des
eingehenden Kontexts als `media`. Wenn ein Plugin lokale Medienlesevorgänge autorisieren muss, importieren Sie
`getAgentScopedMediaLocalRoots(...)` oder
`getAgentScopedMediaLocalRootsForSources(...)` aus dem gezielten
`openclaw/plugin-sdk/media-local-roots`-Unterpfad. Die alte
`agent-media-payload`-Builder-/Root-Fassade dient nur noch der veralteten Kompatibilität.

### Native Nutzlastgestaltung

Wenn Ihr Kanal eine providerspezifische Gestaltung für `message(action="send")` benötigt,
bevorzugen Sie `actions.prepareSendPayload(...)`. Legen Sie native Karten, Blöcke, Einbettungen oder
andere persistente Daten unter `payload.channelData.<channel>` ab und lassen Sie den Kern
über den Ausgangs-/Nachrichtenadapter senden. Verwenden Sie `actions.handleAction(...)` für den Versand
nur als Kompatibilitätsrückfall für Nutzlasten, die nicht serialisiert und
erneut versucht werden können.

### Grammatik der Sitzungskonversation

Wenn Ihre Plattform zusätzlichen Geltungsbereich in Konversations-IDs speichert, belassen Sie diese Analyse
mit `messaging.resolveSessionConversation(...)` im Plugin. Dies ist der
kanonische Hook für die Zuordnung von `rawId` zur grundlegenden Konversations-ID, zur optionalen
Thread-ID, zu einem expliziten `baseConversationId` und zu beliebigen
`parentConversationCandidates`. Wenn Sie `parentConversationCandidates` zurückgeben,
ordnen Sie sie vom engsten übergeordneten Element bis zur allgemeinsten/Basiskonversation.

`messaging.resolveParentConversationCandidates(...)` ist ein veralteter
Kompatibilitätsrückfall für Plugins, die lediglich übergeordnete Rückfälle zusätzlich zur
generischen/rohen ID benötigen. Wenn beide Hooks vorhanden sind, verwendet der Kern zuerst
`resolveSessionConversation(...).parentConversationCandidates` und greift nur dann
auf `resolveParentConversationCandidates(...)` zurück, wenn der kanonische
Hook sie auslässt.

Gebündelte Plugins, die dieselbe Analyse benötigen, bevor die Kanalregistrierung startet,
können eine `session-key-api.ts`-Datei auf oberster Ebene mit einem passenden
`resolveSessionConversation(...)`-Export bereitstellen (siehe die Plugins
Feishu und Telegram). Der Kern verwendet diese bootstrap-sichere Oberfläche nur, wenn die Laufzeit-Plugin-
Registrierung noch nicht verfügbar ist.

Verwenden Sie `openclaw/plugin-sdk/channel-route`, wenn Plugin-Code routenähnliche
Felder normalisieren, einen untergeordneten Thread mit seiner übergeordneten Route vergleichen oder einen
stabilen Deduplizierungsschlüssel aus `{ channel, to, accountId, threadId }` erstellen muss. Der Helfer
normalisiert numerische Thread-IDs auf dieselbe Weise wie der Kern; bevorzugen Sie ihn daher gegenüber
Ad-hoc-Vergleichen mit `String(threadId)`. Plugins mit providerspezifischer Zielgrammatik
sollten `messaging.resolveOutboundSessionRoute(...)` bereitstellen, damit der Kern
providernative Sitzungs- und Thread-Identität ohne Parser-Shims erhält.

### Unterstützung für kontobezogene Konversationsbindungen

Setzen Sie `conversationBindings.supportsCurrentConversationBinding`, wenn der Kanal
generische Bindungen für die aktuelle Konversation unterstützt. `createChatChannelPlugin(...)`
setzt diese statische Fähigkeit standardmäßig auf `true`.

Wenn sich die Unterstützung je nach konfiguriertem Konto unterscheidet, implementieren Sie zusätzlich
`conversationBindings.isCurrentConversationBindingSupported({ accountId })`.
Der Kern wertet diesen synchronen Hook erst aus, nachdem die statische Fähigkeit
aktiviert wurde. Die Rückgabe von `false` macht generische Fähigkeiten für die aktuelle Konversation sowie
Binde-, Such-, Auflistungs-, Aktualisierungs- und Aufhebungsvorgänge für dieses Konto nicht verfügbar.
Wenn der Hook ausgelassen wird, gilt die statische Fähigkeit für jedes Konto.

Ermitteln Sie die Antwort aus der bereits geladenen Kontokonfiguration oder dem Laufzeitzustand. Dieser
Hook steuert nur generische Bindungen für die aktuelle Konversation; er ersetzt weder
konfigurierte Bindungsregeln noch das Plugin-eigene Sitzungsrouting. Vertragstests
sollten über den von
`openclaw/plugin-sdk/channel-core` exportierten Vertrag `ChannelPlugin["conversationBindings"]` mindestens ein unterstütztes und ein nicht unterstütztes Konto abdecken.

## Genehmigungen und Kanalfähigkeiten

Die meisten Kanal-Plugins benötigen keinen genehmigungsspezifischen Code. Der Kern ist für
`/approve` im selben Chat, gemeinsam verwendete Nutzlasten für Genehmigungsschaltflächen und die generische Rückfallzustellung zuständig.
`ChannelPlugin.approvals` wurde entfernt; legen Sie Fakten zu Genehmigungszustellung, nativer Darstellung, Rendering und Autorisierung
stattdessen in einem einzigen `approvalCapability`-Objekt ab. `plugin.auth` dient
nur der An- und Abmeldung – der Kern liest aus diesem Objekt keine Hooks für die Genehmigungsautorisierung mehr.

Verwenden Sie `approvalCapability.delivery` nur für natives Genehmigungsrouting oder die Unterdrückung von Rückfällen
und `approvalCapability.render` nur, wenn ein Kanal tatsächlich
benutzerdefinierte Genehmigungsnutzlasten anstelle des gemeinsamen Renderers benötigt.

### Genehmigungsautorisierung

- `approvalCapability.authorizeActorAction` und
  `approvalCapability.getActionAvailabilityState` bilden die kanonische
  Nahtstelle für die Genehmigungsautorisierung.
- Verwenden Sie `getActionAvailabilityState` für die Verfügbarkeit der Genehmigungsautorisierung im selben Chat.
  Halten Sie konfigurierte Genehmigende für `/approve` verfügbar, selbst wenn die native Zustellung
  deaktiviert ist; verwenden Sie für Hinweise zur Zustellung/Einrichtung stattdessen den nativen Zustand der auslösenden Oberfläche.
- Wenn Ihr Kanal native Ausführungsgenehmigungen bereitstellt, verwenden Sie
  `approvalCapability.getExecInitiatingSurfaceState` für den
  Zustand der auslösenden Oberfläche/des nativen Clients, wenn dieser von der Genehmigungsautorisierung
  im selben Chat abweicht. Der Kern verwendet diesen ausführungsspezifischen Hook, um zwischen `enabled` und
  `disabled` zu unterscheiden, zu entscheiden, ob der auslösende Kanal native
  Ausführungsgenehmigungen unterstützt, und den Kanal in Rückfallhinweise für native Clients aufzunehmen.
  `createApproverRestrictedNativeApprovalCapability(...)` füllt dies für
  den üblichen Fall aus.
- Wenn ein Kanal aus der vorhandenen Konfiguration stabile, eigentümerähnliche DM-Identitäten ableiten kann,
  verwenden Sie `createResolvedApproverActionAuthAdapter` aus
  `openclaw/plugin-sdk/approval-runtime`, um `/approve` im selben Chat einzuschränken,
  ohne genehmigungsspezifische Kernlogik hinzuzufügen.
- Wenn die benutzerdefinierte Genehmigungsautorisierung absichtlich nur den Rückfall im selben Chat zulässt, geben Sie
  `markImplicitSameChatApprovalAuthorization({ authorized: true })` aus
  `openclaw/plugin-sdk/approval-auth-runtime` zurück; andernfalls behandelt der Kern das
  Ergebnis als ausdrückliche Autorisierung des Genehmigenden.
- Wenn ein kanaleigener nativer Callback Genehmigungen direkt auflöst, verwenden Sie
  vor dem Auflösen `isImplicitSameChatApprovalAuthorization(...)`, damit der implizite
  Rückfall weiterhin die normale Akteursautorisierung des Kanals durchläuft.

### Nutzlastlebenszyklus und Einrichtungshinweise

- Verwenden Sie `outbound.shouldSuppressLocalPayloadPrompt` oder
  `outbound.beforeDeliverPayload` für kanalspezifisches Verhalten im Nutzlastlebenszyklus,
  etwa das Ausblenden doppelter lokaler Genehmigungsaufforderungen oder das Senden von
  Tippindikatoren vor der Zustellung.
- Verwenden Sie `approvalCapability.describeExecApprovalSetup`, wenn der Kanal möchte,
  dass die Antwort für den deaktivierten Pfad die genauen Konfigurationsoptionen erläutert, die zum Aktivieren
  nativer Ausführungsgenehmigungen erforderlich sind. Der Hook erhält `{ channel, channelLabel, accountId }`;
  Kanäle mit benannten Konten sollten kontobezogene Pfade wie
  `channels.<channel>.accounts.<id>.execApprovals.*` anstelle von
  Standardwerten auf oberster Ebene darstellen.
- Verwenden Sie `approvalCapability.describePluginApprovalSetup`, wenn Hinweise zu Fehlern bei Plugin-Genehmigungen
  bei Fehlern ohne Route und bei Zeitüberschreitungen von Plugin-Genehmigungen sicher angezeigt werden können.
  `createApproverRestrictedNativeApprovalCapability(...)` leitet dies nicht
  aus `describeExecApprovalSetup` ab; übergeben Sie denselben Helfer nur dann ausdrücklich,
  wenn Plugin- und Ausführungsgenehmigungen tatsächlich dieselbe native Einrichtung verwenden.

### Native Genehmigungszustellung

Wenn ein Kanal native Genehmigungszustellung benötigt, konzentrieren Sie den Kanalcode auf
Zielnormalisierung sowie Transport-/Darstellungsfakten. Verwenden Sie
`createChannelExecApprovalProfile`, `createChannelNativeOriginTargetResolver`,
`createChannelApproverDmTargetResolver` und
`createApproverRestrictedNativeApprovalCapability` aus
`openclaw/plugin-sdk/approval-runtime`. Kapseln Sie die kanalspezifischen Fakten hinter
`approvalCapability.nativeRuntime`, idealerweise über
`createChannelApprovalNativeRuntimeAdapter(...)` oder
`createLazyChannelApprovalNativeRuntimeAdapter(...)`, damit der Kern den
Handler zusammenstellen und Anfragenfilterung, Routing, Deduplizierung, Ablauf, Gateway-
Abonnement sowie Hinweise zur anderweitigen Weiterleitung verwalten kann.

`nativeRuntime` ist in einige kleinere Nahtstellen aufgeteilt:

- `availability` – ob das Konto konfiguriert ist und ob eine Anfrage
  verarbeitet werden soll
- `presentation` – das gemeinsame Ansichtsmodell für Genehmigungen auf
  ausstehende/aufgelöste/abgelaufene native Nutzlasten oder abschließende Aktionen abbilden
- `transport` – Ziele vorbereiten sowie native Genehmigungsnachrichten senden/aktualisieren/löschen
- `interactions` – optionale Hooks zum Binden/Aufheben/Löschen von Aktionen für native Schaltflächen
  oder Reaktionen sowie ein optionaler `cancelDelivered`-Hook. Implementieren Sie
  `cancelDelivered`, wenn `deliverPending` prozessinternen oder persistenten
  Zustand registriert (beispielsweise einen Speicher für Reaktionsziele), damit dieser Zustand freigegeben werden kann, wenn
  das Stoppen eines Handlers die Zustellung abbricht, bevor `bindPending` ausgeführt wird, oder wenn
  `bindPending` kein Handle zurückgibt
- `observe` – optionale Hooks für Zustellungsdiagnosen

Weitere Genehmigungshelfer:

- Verwenden Sie `createNativeApprovalChannelRouteGates` aus
  `openclaw/plugin-sdk/approval-native-runtime`, wenn ein Kanal sowohl
  sitzungsursprüngliche native Zustellung als auch explizite Weiterleitungsziele für Genehmigungen unterstützt. Der
  Helfer zentralisiert die Auswahl der Genehmigungskonfiguration, die Verarbeitung von `mode`, Agenten-/Sitzungs-
  filter, Kontobindung, Sitzungszielabgleich und Ziellistenabgleich,
  während die Aufrufer weiterhin für Kanal-ID, Standardweiterleitungsmodus, Konto-
  suche, Prüfung auf aktivierten Transport, Zielnormalisierung und die Auflösung des
  Ziels aus der Quelle des aktuellen Turns zuständig sind. Verwenden Sie ihn nicht, um kerneigene Standardwerte für Kanalrichtlinien
  zu erstellen; übergeben Sie den dokumentierten Standardmodus des Kanals ausdrücklich.
- `createChannelNativeOriginTargetResolver` verwendet für `{ to, accountId, threadId }`-Ziele standardmäßig
  den gemeinsamen Kanalroutenabgleich. Übergeben Sie
  `targetsMatch` nur, wenn ein Kanal providerspezifische Äquivalenzregeln besitzt,
  etwa den Abgleich von Slack-Zeitstempelpräfixen. Übergeben Sie `normalizeTargetForMatch`, wenn
  der Kanal Provider-IDs kanonisieren muss, bevor der standardmäßige Routenabgleich
  oder ein benutzerdefinierter `targetsMatch`-Callback ausgeführt wird, während das
  ursprüngliche Ziel für die Zustellung erhalten bleibt. Verwenden Sie `normalizeTarget` nur, wenn das aufgelöste
  Zustellungsziel selbst kanonisiert werden soll.
- Wenn der Kanal laufzeiteigene Objekte wie einen Client, ein Token, eine Bolt-
  App oder einen Webhook-Empfänger benötigt, registrieren Sie diese über
  `openclaw/plugin-sdk/channel-runtime-context`. Die generische Registrierung des Laufzeitkontexts
  ermöglicht es dem Kern, fähigkeitsgesteuerte Handler aus dem Startzustand des Kanals zu initialisieren,
  ohne genehmigungsspezifischen Wrapper-Verbindungscode hinzuzufügen.
- Greifen Sie nur dann auf die niedriger angesiedelten `createChannelApprovalHandler` oder
  `createChannelNativeApprovalRuntime` zurück, wenn die fähigkeitsgesteuerte Nahtstelle
  noch nicht ausdrucksstark genug ist.
- Kanäle mit nativer Genehmigungszustellung müssen sowohl `accountId` als auch `approvalKind`
  durch diese Helfer leiten. `accountId` beschränkt die Genehmigungsrichtlinie für mehrere Konten
  auf das richtige Bot-Konto, und `approvalKind` stellt dem Kanal das Verhalten für Ausführungs- gegenüber Plugin-
  Genehmigungen zur Verfügung, ohne hartcodierte Verzweigungen im Kern.
- Der Kern ist auch für Hinweise zur Umleitung von Genehmigungen zuständig. Kanal-Plugins sollten
  aus `createChannelNativeApprovalRuntime` keine eigenen Folgemeldungen wie „Genehmigung wurde an DMs/einen anderen Kanal gesendet“
  senden; stellen Sie stattdessen über die gemeinsamen Helfer für Genehmigungsfähigkeiten präzise Ursprungsinformationen
  sowie korrektes Routing zu den DMs der Genehmigenden bereit und lassen Sie den Kern die tatsächlichen Zustellungen aggregieren, bevor
  er einen Hinweis an den auslösenden Chat zurücksendet.
- Bewahren Sie die Art der zugestellten Genehmigungs-ID durchgängig. Native Clients sollten
  das Routing für Ausführungs- gegenüber Plugin-Genehmigungen nicht anhand des kanallokalen
  Zustands erraten oder umschreiben.
- Übergeben Sie dieses explizite `approvalKind` an `resolveApprovalOverGateway`. Dies verwendet
  den kanonischen `approval.resolve`-Dienst und gibt den aufgezeichneten Gewinner zurück, wenn
  eine andere Oberfläche zuerst antwortet. Die ältere explizite `resolveMethod`-Eingabe
  bleibt für befehlsbasierte Steuerelemente bestehen; neue native Aktionen dürfen sie weder verwenden noch
  die Art aus einer ID ableiten.
- Verschiedene Genehmigungsarten können absichtlich unterschiedliche native
  Oberflächen bereitstellen. Aktuelle gebündelte Beispiele: Matrix behält für Ausführungs- und Plugin-Genehmigungen
  dasselbe native DM-/Kanalrouting und dieselbe Reaktions-UX bei, lässt aber weiterhin
  eine je nach Genehmigungsart unterschiedliche Autorisierung zu; Slack hält natives Genehmigungsrouting
  sowohl für Ausführungs- als auch für Plugin-IDs verfügbar.
- `createApproverRestrictedNativeApprovalAdapter` existiert weiterhin als
  Kompatibilitäts-Wrapper, neuer Code sollte jedoch den Fähigkeits-Builder bevorzugen
  und `approvalCapability` im Plugin bereitstellen.

### Engere Unterpfade der Genehmigungslaufzeit

Bevorzugen Sie für häufig ausgeführte Kanaleinstiegspunkte diese engeren Unterpfade gegenüber dem breiteren
`approval-runtime`-Barrel, wenn Sie nur einen Teil dieser Familie benötigen:

- `openclaw/plugin-sdk/approval-auth-runtime`
- `openclaw/plugin-sdk/approval-client-runtime`
- `openclaw/plugin-sdk/approval-delivery-runtime`
- `openclaw/plugin-sdk/approval-gateway-runtime`
- `openclaw/plugin-sdk/approval-reference-runtime`
- `openclaw/plugin-sdk/approval-handler-adapter-runtime`
- `openclaw/plugin-sdk/approval-handler-runtime`
- `openclaw/plugin-sdk/approval-native-runtime`
- `openclaw/plugin-sdk/approval-reply-runtime`
- `openclaw/plugin-sdk/channel-runtime-context`

Bevorzugen Sie ebenso `openclaw/plugin-sdk/reply-runtime`,
`openclaw/plugin-sdk/reply-dispatch-runtime`,
`openclaw/plugin-sdk/reply-reference` und
`openclaw/plugin-sdk/reply-chunking` gegenüber breiteren, übergreifenden Oberflächen, wenn Sie
nicht alle davon benötigen.

### Unterpfade für die Einrichtung

- `openclaw/plugin-sdk/setup-runtime` umfasst die laufzeitsicheren Einrichtungshilfen:
  `createSetupTranslator`, importsichere Adapter für Einrichtungs-Patches
  (`createPatchedAccountSetupAdapter`, `createEnvPatchedAccountSetupAdapter`,
  `createSetupInputPresenceValidator`), die Ausgabe von Suchhinweisen,
  `promptResolvedAllowFrom`, `splitSetupEntries` und die delegierten
  Builder für Einrichtungs-Proxys.
- `openclaw/plugin-sdk/channel-setup` umfasst die Builder zur Einrichtung optionaler Installationen
  sowie einige einrichtungssichere Grundelemente: `createOptionalChannelSetupSurface`,
  `createOptionalChannelSetupAdapter`, `createOptionalChannelSetupWizard`,
  `DEFAULT_ACCOUNT_ID`, `createTopLevelChannelDmPolicy`,
  `setSetupChannelEnabled` und `splitSetupEntries`.
- Verwenden Sie die breitere `openclaw/plugin-sdk/setup`-Schnittstelle nur, wenn Sie außerdem
  die umfangreicheren gemeinsamen Einrichtungs-/Konfigurationshilfen wie
  `moveSingleAccountChannelSectionToDefaultAccount(...)` benötigen.

Wenn Ihr Kanal auf Einrichtungsoberflächen nur darauf hinweisen soll, dass „zuerst dieses Plugin installiert werden muss“,
bevorzugen Sie `createOptionalChannelSetupSurface(...)`. Der generierte
Adapter/Assistent verweigert bei Konfigurationsschreibvorgängen und der Finalisierung standardmäßig den Vorgang und verwendet
dieselbe Meldung zur erforderlichen Installation bei Validierung, Finalisierung und
Text für den Dokumentationslink.

Wenn Ihr Kanal eine umgebungsvariablengesteuerte Einrichtung oder Authentifizierung unterstützt, stellen Sie diese über das
Kanalkonfigurationsschema und die Einrichtungsdeskriptoren bereit. Verwenden Sie `envVars` der Kanallaufzeit oder
lokale Konstanten ausschließlich für Texte für Betreiber.

Wenn Ihr Kanal in `status`, `channels list`, `channels status` oder
SecretRef-Scans erscheinen kann, bevor die Plugin-Laufzeit startet, fügen Sie `openclaw.setupEntry` in
`package.json` hinzu. Dieser Einstiegspunkt sollte in schreibgeschützten Befehlspfaden gefahrlos importiert
werden können und die Kanalmetadaten, den einrichtungssicheren Konfigurationsadapter,
den Statusadapter sowie die für diese Zusammenfassungen erforderlichen Kanal-Secret-Zielmetadaten
zurückgeben. Starten Sie über den Einrichtungseinstieg keine Clients, Listener oder Transportlaufzeiten.

Halten Sie auch den Importpfad des Hauptkanaleinstiegs schlank. Die Erkennung kann
den Einstieg und das Kanal-Plugin-Modul auswerten, um Fähigkeiten zu registrieren, ohne
den Kanal zu aktivieren. Dateien wie `channel-plugin-api.ts` sollten
das Kanal-Plugin-Objekt exportieren, ohne Einrichtungsassistenten, Transport-
Clients, Socket-Listener, Starter für Unterprozesse oder Module zum Starten von Diensten zu importieren.
Legen Sie diese Laufzeitkomponenten in Modulen ab, die über `registerFull(...)`, Laufzeit-
Setter oder verzögert geladene Fähigkeitsadapter geladen werden.

### Weitere schlanke Kanalunterpfade

Bevorzugen Sie für andere häufig verwendete Kanalpfade die schlanken Hilfen gegenüber breiteren älteren
Oberflächen:

- `openclaw/plugin-sdk/account-core`, `openclaw/plugin-sdk/account-id`,
  `openclaw/plugin-sdk/account-resolution` und
  `openclaw/plugin-sdk/account-helpers` für Konfigurationen mit mehreren Konten und
  den Rückgriff auf das Standardkonto
- `openclaw/plugin-sdk/inbound-envelope` und
  `openclaw/plugin-sdk/channel-inbound` für eingehende Routen/Umschläge sowie die
  Verdrahtung von Aufzeichnung und Weiterleitung
- `openclaw/plugin-sdk/channel-targets` für Hilfen zum Parsen von Zielen
- `openclaw/plugin-sdk/channel-outbound` für Delegaten für ausgehende Identitäten/Sendevorgänge
  und typisierte Nutzlastplanung
- `buildThreadAwareOutboundSessionRoute(...)` aus
  `openclaw/plugin-sdk/channel-core`, wenn eine ausgehende Route
  ein explizites `replyToId`/`threadId` beibehalten oder die aktuelle `:thread:`-
  Sitzung wiederherstellen soll, nachdem der Basissitzungsschlüssel weiterhin übereinstimmt. Provider-Plugins können
  die Priorität, das Suffixverhalten und die Normalisierung der Thread-ID überschreiben, wenn
  ihre Plattform über eine native Semantik für die Thread-Zustellung verfügt.
- `openclaw/plugin-sdk/thread-bindings-runtime` für den Lebenszyklus von Thread-Bindungen
  und die Adapterregistrierung

Kanäle, die ausschließlich Authentifizierung bereitstellen, können sich in der Regel auf den Standardpfad beschränken: Der Kern verarbeitet
Genehmigungen, und das Plugin stellt lediglich Fähigkeiten für ausgehende Vorgänge und Authentifizierung bereit. Native
Genehmigungskanäle wie Matrix, Slack, Telegram und benutzerdefinierte Chat-Transporte
sollten die gemeinsamen nativen Hilfen verwenden, statt einen eigenen Genehmigungs-
lebenszyklus zu implementieren.

## Richtlinie für Erwähnungen in eingehenden Nachrichten

Halten Sie die Verarbeitung von Erwähnungen in eingehenden Nachrichten in zwei Ebenen getrennt:

- plugin-eigene Erfassung von Nachweisen
- gemeinsame Richtlinienauswertung

Verwenden Sie `openclaw/plugin-sdk/channel-mention-gating` für Entscheidungen zur Erwähnungsrichtlinie.
Verwenden Sie `openclaw/plugin-sdk/channel-inbound` nur, wenn Sie das breitere
Exportmodul für eingehende Hilfen benötigen.

Gut geeignet für Plugin-lokale Logik:

- Erkennung von Antworten an den Bot
- Erkennung zitierter Bot-Nachrichten
- Prüfungen der Thread-Beteiligung
- Ausschlüsse von Dienst-/Systemnachrichten
- plattformnative Caches, die zum Nachweis der Bot-Beteiligung erforderlich sind

Gut geeignet für die gemeinsame Hilfe:

- `requireMention`
- Ergebnis einer expliziten Erwähnung
- Positivliste impliziter Erwähnungen
- Umgehung für Befehle
- abschließende Entscheidung zum Überspringen

Bevorzugter Ablauf:

1. Berechnen Sie lokale Erwähnungsfakten.
2. Übergeben Sie diese Fakten an `resolveInboundMentionDecision({ facts, policy })`.
3. Verwenden Sie `decision.effectiveWasMentioned`, `decision.shouldBypassMention` und
   `decision.shouldSkip` in Ihrer Eingangssperre.

```typescript
import {
  implicitMentionKindWhen,
  matchesMentionWithExplicit,
  resolveInboundMentionDecision,
} from "openclaw/plugin-sdk/channel-inbound";
import { resolveChannelImplicitMentions } from "openclaw/plugin-sdk/channel-ingress-runtime";

const wasMentioned = matchesMentionWithExplicit({
  text,
  mentionRegexes,
  explicit: {
    hasAnyMention,
    isExplicitlyMentioned,
    canResolveExplicit,
  },
});

const facts = {
  canDetectMention: true,
  wasMentioned,
  hasAnyMention,
  implicitMentionKinds: [
    ...implicitMentionKindWhen("reply_to_bot", isReplyToBot),
    ...implicitMentionKindWhen("quoted_bot", isQuoteOfBot),
  ],
};

const implicitMentions = resolveChannelImplicitMentions({
  cfg,
  channel: channelId,
  accountId,
});

const decision = resolveInboundMentionDecision({
  facts,
  policy: {
    isGroup,
    requireMention,
    implicitMentions,
    allowTextCommands,
    hasControlCommand,
    commandAuthorized,
  },
});

if (decision.shouldSkip) return;
```

`matchesMentionWithExplicit(...)` gibt einen booleschen Wert zurück. `hasAnyMention`,
`isExplicitlyMentioned` und `canResolveExplicit` stammen aus den eigenen
nativen Erwähnungsmetadaten des Kanals (Nachrichtenentitäten, Kennzeichnungen für Antworten an den Bot und Ähnliches);
geben Sie die Werte `false`/`undefined` an, wenn Ihre Plattform sie nicht erkennen kann.

`api.runtime.channel.mentions` stellt dieselben gemeinsamen Erwähnungshilfen für
gebündelte Kanal-Plugins bereit, die bereits von Laufzeitinjektion abhängen:
`buildMentionRegexes`, `matchesMentionPatterns`, `matchesMentionWithExplicit`,
`implicitMentionKindWhen`, `resolveInboundMentionDecision`.

Wenn Sie nur `implicitMentionKindWhen` und `resolveInboundMentionDecision` benötigen,
importieren Sie diese aus `openclaw/plugin-sdk/channel-mention-gating`, um das Laden
nicht zugehöriger Laufzeithilfen für eingehende Nachrichten zu vermeiden.

## Anleitung

<Steps>
  <a id="step-1-package-and-manifest"></a>
  <Step title="Paket und Manifest">
    Erstellen Sie die standardmäßigen Plugin-Dateien. Das Feld `channels` in
    `openclaw.plugin.json` (nicht ein Feld `kind`) kennzeichnet ein Manifest als
    Eigentümer eines Kanals. Die vollständige Oberfläche der Paketmetadaten finden Sie unter
    [Plugin-Einrichtung und -Konfiguration](/de/plugins/sdk-setup#openclaw-channel):

    <CodeGroup>
    ```json package.json
    {
      "name": "@myorg/openclaw-acme-chat",
      "version": "1.0.0",
      "type": "module",
      "openclaw": {
        "extensions": ["./index.ts"],
        "setupEntry": "./setup-entry.ts",
        "channel": {
          "id": "acme-chat",
          "label": "Acme Chat",
          "blurb": "OpenClaw mit Acme Chat verbinden."
        }
      }
    }
    ```

    ```json openclaw.plugin.json
    {
      "id": "acme-chat",
      "channels": ["acme-chat"],
      "name": "Acme Chat",
      "description": "Kanal-Plugin für Acme Chat",
      "configSchema": {
        "type": "object",
        "additionalProperties": false,
        "properties": {}
      },
      "channelConfigs": {
        "acme-chat": {
          "schema": {
            "type": "object",
            "additionalProperties": false,
            "properties": {
              "token": { "type": "string" },
              "allowFrom": {
                "type": "array",
                "items": { "type": "string" }
              }
            }
          },
          "uiHints": {
            "token": {
              "label": "Bot-Token",
              "sensitive": true
            }
          }
        }
      }
    }
    ```
    </CodeGroup>

    `configSchema` validiert `plugins.entries.acme-chat.config`. Verwenden Sie es für
    Plugin-eigene Einstellungen, die nicht zur Konfiguration des Kanalkontos gehören.
    `channelConfigs.acme-chat.schema` validiert `channels.acme-chat` und ist die
    Quelle im selten ausgeführten Pfad, die von Konfigurationsschema, Einrichtung und UI-Oberflächen verwendet wird, bevor die
    Plugin-Laufzeit geladen wird. Die vollständige Referenz der Felder auf oberster Ebene finden Sie unter [Plugin-Manifest](/de/plugins/manifest).

  </Step>

  <Step title="Kanal-Plugin-Objekt erstellen">
    Die Schnittstelle `ChannelPlugin` verfügt über viele optionale Adapteroberflächen. Beginnen Sie mit
    dem Minimum – `id`, `config` und `setup` – und fügen Sie nach Bedarf
    Adapter hinzu.

    Erstellen Sie `src/channel.ts`:

    ```typescript src/channel.ts
    import {
      createChatChannelPlugin,
      createChannelPluginBase,
    } from "openclaw/plugin-sdk/channel-core";
    import type { OpenClawConfig } from "openclaw/plugin-sdk/channel-core";
    import { acmeChatApi } from "./client.js"; // your platform API client

    type ResolvedAccount = {
      accountId: string | null;
      token: string;
      allowFrom: string[];
      dmPolicy: string | undefined;
    };

    function resolveAccount(
      cfg: OpenClawConfig,
      accountId?: string | null,
    ): ResolvedAccount {
      const section = (cfg.channels as Record<string, any>)?.["acme-chat"];
      const token = section?.token;
      if (!token) throw new Error("acme-chat: token is required");
      return {
        accountId: accountId ?? null,
        token,
        allowFrom: section?.allowFrom ?? [],
        dmPolicy: section?.dmSecurity,
      };
    }

    export const acmeChatPlugin = createChatChannelPlugin<ResolvedAccount>({
      base: createChannelPluginBase({
        id: "acme-chat",
        // Account resolution/inspection belongs on `config`, not `setup`.
        // `setup` covers onboarding writes (applyAccountConfig, validateInput).
        config: {
          listAccountIds: () => ["default"],
          resolveAccount,
          inspectAccount(cfg, accountId) {
            const section =
              (cfg.channels as Record<string, any>)?.["acme-chat"];
            return {
              enabled: Boolean(section?.token),
              configured: Boolean(section?.token),
              tokenStatus: section?.token ? "available" : "missing",
            };
          },
        },
        setup: {
          applyAccountConfig: ({ cfg, input }) => ({
            ...cfg,
            channels: {
              ...cfg.channels,
              "acme-chat": { ...(cfg.channels as any)?.["acme-chat"], ...input },
            },
          }),
        },
      }),

      // DM security: who can message the bot
      security: {
        dm: {
          channelKey: "acme-chat",
          resolvePolicy: (account) => account.dmPolicy,
          resolveAllowFrom: (account) => account.allowFrom,
          defaultPolicy: "allowlist",
        },
      },

      // Pairing: approval flow for new DM contacts
      pairing: {
        text: {
          idLabel: "Acme Chat username",
          message: "Send this code to verify your identity:",
          notify: async ({ target, code }) => {
            await acmeChatApi.sendDm(target, `Pairing code: ${code}`);
          },
        },
      },

      // Threading: how replies are delivered
      threading: { topLevelReplyToMode: "reply" },

      // Outbound: send messages to the platform
      outbound: {
        attachedResults: {
          channel: "acme-chat",
          sendText: async (params) => {
            const result = await acmeChatApi.sendMessage(
              params.to,
              params.text,
            );
            return { messageId: result.id };
          },
        },
        base: {
          sendMedia: async (params) => {
            await acmeChatApi.sendFile(params.to, params.filePath);
          },
        },
      },
    });
    ```

    Verwenden Sie für Kanäle, die sowohl kanonische DM-Schlüssel auf oberster Ebene als auch verschachtelte Legacy-Schlüssel akzeptieren, die Hilfsfunktionen aus `plugin-sdk/channel-config-helpers`: `resolveChannelDmAccess`, `resolveChannelDmPolicy`, `resolveChannelDmAllowFrom` und `normalizeChannelDmPolicy` sorgen dafür, dass kontolokale Werte Vorrang vor geerbten Stammwerten haben. Kombinieren Sie denselben Resolver über `normalizeLegacyDmAliases` mit der Doctor-Reparatur, damit Laufzeit und Migration denselben Vertrag lesen.

    <Accordion title="Was createChatChannelPlugin für Sie übernimmt">
      Statt Low-Level-Adapterschnittstellen manuell zu implementieren, übergeben Sie
      deklarative Optionen, aus denen der Builder die Komponenten zusammensetzt:

      | Option | Was dadurch verbunden wird |
      | --- | --- |
      | `security.dm` | Bereichsbezogener Resolver für DM-Sicherheit aus Konfigurationsfeldern |
      | `pairing.text` | Textbasierter DM-Kopplungsablauf mit Codeaustausch |
      | `threading` | Resolver für den Antwortmodus (fest, kontobezogen oder benutzerdefiniert) |
      | `outbound.attachedResults` | Sendefunktionen, die Ergebnismetadaten (Nachrichten-IDs) zurückgeben; erfordert eine gleichgeordnete `channel`-ID, damit der Kern das zurückgegebene Zustellungsergebnis kennzeichnen kann |

      Wenn Sie vollständige Kontrolle benötigen, können Sie statt der deklarativen
      Optionen auch unverarbeitete Adapterobjekte übergeben.

      Unverarbeitete ausgehende Adapter können eine `chunker(text, limit, ctx)`-Funktion definieren.
      Die optionale `ctx.formatting` enthält Formatierungsentscheidungen zum Zustellungszeitpunkt,
      beispielsweise `maxLinesPerMessage`; wenden Sie diese vor dem Senden an, damit Antwortverkettung
      und Abschnittsgrenzen durch die gemeinsame ausgehende Zustellung nur einmal aufgelöst werden.
      Sendekontexte enthalten außerdem `replyToIdSource` (`implicit` oder `explicit`),
      wenn ein natives Antwortziel aufgelöst wurde, sodass Payload-Hilfsfunktionen
      explizite Antwort-Tags beibehalten können, ohne einen impliziten, einmalig nutzbaren Antwortplatz zu verbrauchen.
    </Accordion>

    ### Adapter für Gruppen-Tool-Richtlinien

    Ein Kanal, der `group.resolveToolPolicy` implementiert und
    `toolsBySender` unterstützt, muss die vollständige `ChannelGroupContext` an seinen
    gemeinsamen Richtlinien-Resolver weiterleiten. Berücksichtigen Sie insbesondere `senderPolicyMode: "never"`,
    indem Sie absenderspezifische Überlagerungen sowohl im Bereich der übereinstimmenden Gruppe als auch im Platzhalterbereich
    überspringen und dabei weiterhin die grundlegende `tools`-Richtlinie anwenden.

    OpenClaw setzt diesen Modus ausschließlich für vertrauenswürdige Nicht-Ingress-Ausführungen, bei denen die
    Absenderautorität bereits in einem serverseitig verwalteten Umschlag erfasst wurde, etwa bei einem
    ausdrücklich begrenzten geplanten Lauf. Plugins dürfen den Modus nicht aus
    eingehenden Metadaten ableiten, als Kanalstatus speichern oder als Konfiguration verfügbar machen. Fügen Sie
    einen Adaptertest hinzu, der nachweist, dass der Modus einen Platzhaltereintrag `toolsBySender`
    überspringt, ohne die übereinstimmende grundlegende Einschränkung `tools` zu verwerfen.

  </Step>

  <Step title="Einstiegspunkt verbinden">
    Erstellen Sie `index.ts`:

    ```typescript index.ts
    import { defineChannelPluginEntry } from "openclaw/plugin-sdk/channel-core";
    import { acmeChatPlugin } from "./src/channel.js";

    export default defineChannelPluginEntry({
      id: "acme-chat",
      name: "Acme Chat",
      description: "Acme Chat channel plugin",
      plugin: acmeChatPlugin,
      registerCliMetadata(api) {
        api.registerCli(
          ({ program }) => {
            program
              .command("acme-chat")
              .description("Acme Chat management");
          },
          {
            descriptors: [
              {
                name: "acme-chat",
                description: "Acme Chat management",
                hasSubcommands: false,
              },
            ],
          },
        );
      },
      registerFull(api) {
        api.registerGatewayMethod(/* ... */);
      },
    });
    ```

    Platzieren Sie kanaleigene CLI-Deskriptoren in `registerCliMetadata(...)`, damit OpenClaw
    sie in der Stammhilfe anzeigen kann, ohne die vollständige Kanallaufzeit zu aktivieren,
    während normale vollständige Ladevorgänge dieselben Deskriptoren weiterhin für die tatsächliche
    Befehlsregistrierung übernehmen. Behalten Sie `registerFull(...)` für reine Laufzeitaufgaben bei.
    `defineChannelPluginEntry` verarbeitet die Aufteilung der Registrierungsmodi automatisch.
    Wenn `registerFull(...)` Gateway-RPC-Methoden registriert, verwenden Sie ein
    Plugin-spezifisches Präfix. Die Kern-Administrationsnamensräume (`config.*`,
    `exec.approvals.*`, `wizard.*`, `update.*`) bleiben reserviert und werden immer
    zu `operator.admin` aufgelöst. Alle Optionen finden Sie unter
    [Einstiegspunkte](/de/plugins/sdk-entrypoints#definechannelpluginentry).

  </Step>

  <Step title="Setup-Einstieg hinzufügen">
    Erstellen Sie `setup-entry.ts` für leichtgewichtiges Laden während des Onboardings:

    ```typescript setup-entry.ts
    import { defineSetupPluginEntry } from "openclaw/plugin-sdk/channel-core";
    import { acmeChatPlugin } from "./src/channel.js";

    export default defineSetupPluginEntry(acmeChatPlugin);
    ```

    OpenClaw lädt diesen anstelle des vollständigen Einstiegspunkts, wenn der Kanal deaktiviert
    oder nicht konfiguriert ist. Dadurch wird vermieden, dass während der Einrichtungsabläufe umfangreicher Runtime-Code geladen wird.
    Weitere Einzelheiten finden Sie unter [Einrichtung und Konfiguration](/de/plugins/sdk-setup#setup-entry).

    Gebündelte Workspace-Kanäle, die einrichtungssichere Exporte in Sidecar-
    Module aufteilen, können `defineBundledChannelSetupEntry(...)` aus
    `openclaw/plugin-sdk/channel-entry-contract` verwenden, wenn sie außerdem einen
    expliziten Runtime-Setter für die Einrichtungsphase benötigen.

  </Step>

  <Step title="Eingehende Nachrichten verarbeiten">
    Ihr Plugin muss Nachrichten von der Plattform empfangen und an
    OpenClaw weiterleiten. Das typische Muster ist ein Webhook, der die Anfrage verifiziert und
    sie über den Handler für eingehende Nachrichten Ihres Kanals weiterleitet:

    ```typescript
    registerFull(api) {
      api.registerHttpRoute({
        path: "/acme-chat/webhook",
        auth: "plugin", // vom Plugin verwaltete Authentifizierung (Signaturen selbst verifizieren)
        handler: async (req, res) => {
          const event = parseWebhookPayload(req);

          // Ihr Handler für eingehende Nachrichten leitet die Nachricht an OpenClaw weiter.
          // Die genaue Verknüpfung hängt von Ihrem Plattform-SDK ab –
          // ein praxisnahes Beispiel finden Sie im gebündelten Plugin-Paket für Microsoft Teams oder Google Chat.
          await handleAcmeChatInbound(api, event);

          res.statusCode = 200;
          res.end("ok");
          return true;
        },
      });
    }
    ```

    <Note>
      Die Verarbeitung eingehender Nachrichten ist kanalspezifisch. Jedes Kanal-Plugin verwaltet
      seine eigene Pipeline für eingehende Nachrichten. Praxisnahe Muster finden Sie in gebündelten Kanal-Plugins
      (zum Beispiel im Plugin-Paket für Microsoft Teams oder Google Chat).
    </Note>

  </Step>

<a id="step-6-test"></a>
<Step title="Testen">
Schreiben Sie zusammen mit dem Code abgelegte Tests in `src/channel.test.ts`:

    ```typescript src/channel.test.ts
    import { describe, it, expect } from "vitest";
    import { acmeChatPlugin } from "./channel.js";

    describe("acme-chat plugin", () => {
      it("resolves account from config", () => {
        const cfg = {
          channels: {
            "acme-chat": { token: "test-token", allowFrom: ["user1"] },
          },
        } as any;
        const account = acmeChatPlugin.config.resolveAccount(cfg, undefined);
        expect(account.token).toBe("test-token");
      });

      it("inspects account without materializing secrets", () => {
        const cfg = {
          channels: { "acme-chat": { token: "test-token" } },
        } as any;
        const result = acmeChatPlugin.config.inspectAccount!(cfg, undefined);
        expect(result.configured).toBe(true);
        expect(result.tokenStatus).toBe("available");
      });

      it("reports missing config", () => {
        const cfg = { channels: {} } as any;
        const result = acmeChatPlugin.config.inspectAccount!(cfg, undefined);
        expect(result.configured).toBe(false);
      });
    });
    ```

    ```bash
    pnpm test <bundled-plugin-root>/acme-chat/
    ```

    Gemeinsam verwendete Testhilfen finden Sie unter [Testen](/de/plugins/sdk-testing).

</Step>
</Steps>

## Dateistruktur

```text
<bundled-plugin-root>/acme-chat/
├── package.json              # openclaw.channel-Metadaten
├── openclaw.plugin.json      # Manifest mit Konfigurationsschema
├── index.ts                  # defineChannelPluginEntry
├── setup-entry.ts            # defineSetupPluginEntry
├── api.ts                    # Öffentliche Exporte (optional)
├── runtime-api.ts            # Interne Runtime-Exporte (optional)
└── src/
    ├── channel.ts            # ChannelPlugin über createChatChannelPlugin
    ├── channel.test.ts       # Tests
    ├── client.ts             # Plattform-API-Client
    └── runtime.ts            # Runtime-Speicher (falls erforderlich)
```

## Fortgeschrittene Themen

<CardGroup cols={2}>
  <Card title="Threading-Optionen" icon="git-branch" href="/de/plugins/sdk-entrypoints#registration-mode">
    Feste, kontobezogene oder benutzerdefinierte Antwortmodi
  </Card>
  <Card title="Integration des Nachrichten-Tools" icon="puzzle" href="/de/plugins/architecture#channel-plugins-and-the-shared-message-tool">
    describeMessageTool und Aktionserkennung
  </Card>
  <Card title="Zielauflösung" icon="crosshair" href="/de/plugins/architecture-internals#channel-target-resolution">
    inferTargetChatType, looksLikeId, reservedLiterals, resolveTarget
  </Card>
  <Card title="Runtime-Hilfsfunktionen" icon="settings" href="/de/plugins/sdk-runtime">
    TTS, STT, Medien und Subagent über api.runtime
  </Card>
  <Card title="API für eingehende Kanalereignisse" icon="bolt" href="/de/plugins/sdk-channel-inbound">
    Gemeinsamer Lebenszyklus eingehender Ereignisse: erfassen, auflösen, aufzeichnen, weiterleiten, abschließen
  </Card>
</CardGroup>

<Note>
Einige gebündelte Hilfsschnittstellen bestehen weiterhin für die Wartung und
Kompatibilität gebündelter Plugins. Sie sind nicht das empfohlene Muster für neue Kanal-Plugins;
bevorzugen Sie die generischen Unterpfade für Kanal, Einrichtung, Antworten und Runtime der gemeinsamen SDK-
Oberfläche, sofern Sie nicht direkt diese gebündelte Plugin-Familie pflegen.
</Note>

## Nächste Schritte

- [Provider-Plugins](/de/plugins/sdk-provider-plugins) - wenn Ihr Plugin auch Modelle bereitstellt
- [SDK-Übersicht](/de/plugins/sdk-overview) - vollständige Referenz der Unterpfadimporte
- [SDK-Tests](/de/plugins/sdk-testing) - Testhilfsprogramme und Vertragstests
- [Plugin-Manifest](/de/plugins/manifest) - vollständiges Manifest-Schema

## Verwandte Themen

- [Einrichtung des Plugin-SDK](/de/plugins/sdk-setup)
- [Plugins erstellen](/de/plugins/building-plugins)
- [Plugins für den Agent-Harness](/de/plugins/sdk-agent-harness)
