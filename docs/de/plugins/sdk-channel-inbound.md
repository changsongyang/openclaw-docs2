---
read_when:
    - Sie erstellen oder refaktorieren den Empfangspfad eines Messaging-Kanal-Plugins
    - Sie benötigen eine gemeinsame Erstellung des eingehenden Kontexts, Sitzungsaufzeichnung oder vorbereitete Antwortübermittlung
    - Sie migrieren alte Hilfsfunktionen für Kanal-Turns zu Inbound-/Message-APIs
summary: 'Hilfsfunktionen für eingehende Ereignisse in Kanal-Plugins: Kontexterstellung, gemeinsame Runner-Orchestrierung, Sitzungsdatensatz und vorbereiteter Antwortversand'
title: API für eingehende Kanalnachrichten
x-i18n:
    generated_at: "2026-07-24T20:32:46Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 854408ca42cfe1e1b48e4fd223b176f438e1db28deb9a5aa33eea8238127d9df
    source_path: plugins/sdk-channel-inbound.md
    workflow: 16
---

Empfangspfade von Kanälen folgen einem Ablauf:

```text
Plattformereignis -> eingehende Fakten/Kontext -> Agentenantwort -> Nachrichtenzustellung
```

Verwenden Sie `openclaw/plugin-sdk/channel-inbound` für die Normalisierung eingehender Ereignisse,
Formatierung, Roots und Orchestrierung. Verwenden Sie
`openclaw/plugin-sdk/channel-outbound` für nativen Versand, Empfangsbestätigung, dauerhafte
Zustellung und Live-Vorschauverhalten.

## Kern-Hilfsfunktionen

```ts
import {
  buildChannelInboundEventContext,
  runChannelInboundEvent,
  dispatchChannelInboundReply,
} from "openclaw/plugin-sdk/channel-inbound";
```

- `buildChannelInboundEventContext(...)`: projiziert normalisierte Kanalfakten
  in den Prompt-/Sitzungskontext. Übergeben Sie kanaleigene Absender-/Chat-Metadaten
  über `channelContext`, die Plugin-Hooks als `ctx.channelContext` sehen.
  Erweitern Sie `PluginHookChannelSenderContext` oder `PluginHookChannelChatContext`
  aus diesem Unterpfad um kanalspezifische Felder.
- `runChannelInboundEvent(...)`: führt Erfassung, Klassifizierung, Vorprüfung, Auflösung,
  Aufzeichnung, Weiterleitung und Abschluss für ein eingehendes Plattformereignis aus.
- `dispatchChannelInboundReply(...)`: zeichnet eine bereits
  zusammengestellte eingehende Antwort auf und leitet sie über einen Zustellungsadapter weiter.

Lassen Sie bei eingehenden Ereignissen, die ausschließlich Medien enthalten, den Nachrichtentext und Befehlstext leer und
übergeben Sie pro nativem Anhang einen `ChannelInboundMediaInput`-Fakt. Wenn eine umgebende
Verlaufszeile oder ein anderer reiner Textträger diese Fakten beschreiben muss, verwenden Sie
`formatMediaPlaceholderText(media)`. Die Funktion klassifiziert jeden Fakt anhand von `kind`, dem MIME-
Typ und anschließend der Pfad- oder URL-Erweiterung; nicht heruntergeladene native Anhänge sollten weiterhin
jeweils einen Fakt enthalten, der nur den Typ angibt. Verwenden Sie den Formatierer nicht, um den
primären eingehenden Nachrichtentext zu erzeugen.

Normalisieren Sie Plugin-eigene Anhangsdatensätze mit `toInboundMediaFacts(...)` und
übergeben Sie anschließend das resultierende geordnete Array über das Feld `media` des Kontexts:

```ts
const media = toInboundMediaFacts([
  { path: saved.path, url: nativeUrl, contentType: saved.contentType, messageId },
]);

const ctx = finalizeInboundContext({ Body: caption, media });
```

Die Arrayposition ist die Identität des Anhangs. Die faktenbezogenen Felder `transcribed`, `messageId` und
`workspaceDir` ersetzen die veralteten parallelen Index-/Workspace-Felder. Die Kontextfelder
`MediaPath`, `MediaPaths`, `MediaUrl`, `MediaUrls`, `MediaType`, `MediaTypes`,
`MediaTranscribedIndexes`, `MediaWorkspaceDir` und `MediaStaged`
sowie `buildChannelInboundMediaPayload(...)` bleiben nur als veraltete
Kompatibilitätsfunktionen verfügbar. Neue Plugins sollten sie weder erstellen noch lesen.

Gebündelte/native Kanäle, die bereits das injizierte Plugin-Laufzeitobjekt
erhalten, können dieselben Hilfsfunktionen unter `runtime.channel.inbound.*` aufrufen, anstatt
diesen Unterpfad direkt zu importieren:

```ts
await runtime.channel.inbound.run({
  channel: "demo",
  accountId,
  raw: platformEvent,
  adapter: {
    ingest: normalizePlatformEvent,
    resolveTurn: resolveInboundReply,
  },
});
```

Stellen Sie `dispatchChannelInboundReply(...)`-Eingaben für Kompatibilitäts-
Dispatcher zusammen, bei denen die Plattformzustellung im Zustellungsadapter verbleibt. Neue Versandpfade
sollten stattdessen Nachrichtenadapter und Hilfsfunktionen für dauerhafte Nachrichten aus
`channel-outbound` verwenden.

## Vertrag für den Zustellungsabschluss

`ChannelInboundTurnPlan.delivery` übernimmt den nativen Versand für jede logische Antwort-
Nutzlast. Der Kern übernimmt die Reihenfolge ausgehender Hooks und, wenn der Adapter dies aktiviert,
die abschließende `message_sent`-Beobachtung. Halten Sie diese Zuständigkeiten getrennt, damit
eine Nutzlast keine doppelten Abschlussereignisse erzeugen kann.

Die Felder des Zustellungsergebnisses haben folgende Bedeutung:

| Feld                    | Vertrag                                                                                                                                                                                                                     |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `content`                | Vom Provider akzeptierter sichtbarer Text für die logische Nutzlast nach nativer Formatierung oder Finalisierung. Lassen Sie das Feld weg, um den vorbereiteten Nutzlasttext für die abschließende Beobachtung zu verwenden. Bei reinen Medien-Sendungen kann es entfallen.                             |
| `messageIds` / `receipt` | Tatsächliche Provider-Identitäten für die sichtbare Sendung. Bevorzugen Sie einen `MessageReceipt`; der Kern verwendet dessen primäre Provider-ID für `message_sent`.                                                                                            |
| `visibleReplySent`       | Setzen Sie dies nur dann auf `false`, wenn der Provider keine sichtbare Vorschau oder endgültige Nachricht erzeugt hat. Der Kern gibt für dieses Ergebnis kein erfolgreiches `message_sent` aus.                                                                          |
| `finalization`           | Ein Promise für den verzögerten nativen Abschluss derselben logischen Nutzlast, etwa zum Schließen oder Bearbeiten einer direkt aktualisierten Streaming-Karte. Seine aufgelösten Felder überschreiben das unmittelbare Ergebnis vor der abschließenden Beobachtung und `onDelivered`. |

Setzen Sie die Option `observeMessageSent` des Zustellungsadapters auf `true`, wenn der Kern
die kanonischen Plugin- und internen `message_sent`-Ereignisse für die
nicht dauerhaften Sendungen dieses Adapters ausgeben soll. Geben Sie diese Option nicht aus `deliver` zurück und geben Sie
diese Ereignisse nicht zusätzlich im Plugin aus. Dauerhafte Sendungen werden bereits über
den gemeinsamen Eigentümer für ausgehende Vorgänge ausgegeben und nicht dupliziert.

Geben Sie pro logischer Nutzlast ein Ergebnis zurück. `finalization` ist kein zweiter Versand und
darf `reply_payload_sending` oder `message_sending` nicht erneut ausführen. Sobald
`deliver` zurückkehrt, beobachtet der Kern die Ablehnung des Finalisierungs-Promise, damit sie
nicht unbehandelt bleiben kann; der Kern wartet weiterhin auf das ursprüngliche Promise, nachdem die
Antwortweiterleitung abgeschlossen ist. Anschließend gibt er pro Nutzlast höchstens eine abschließende Beobachtung
mit dem finalisierten Inhalt und der Provider-ID aus. `onDelivered` erhält, sofern vorhanden,
nach dieser Beobachtung das abgeschlossene Ergebnis.

Lehnen Sie `deliver` oder `finalization` ab, wenn die native Zustellung fehlschlägt. Wenn kein Provider-
Versand versucht wurde, lösen Sie `PlatformMessageNotDispatchedError` aus
`openclaw/plugin-sdk/error-runtime` aus; der Kern unterdrückt ein fälschliches
`message_sent`-Ereignis. Wenn ein nativer Versand sichtbar wurde, bevor ein späterer Vorgang fehlschlug,
bewahren Sie die sichtbare Teilmenge im Fehler auf:

```ts
import { createChannelPartialDeliveryError } from "openclaw/plugin-sdk/channel-inbound";

throw createChannelPartialDeliveryError(cause, {
  visibleReplySent: true,
  content: finalizedVisibleText,
  receipt,
});
```

Der Kern gibt eine fehlgeschlagene Abschlussbeobachtung mit diesem für den Provider sichtbaren Inhalt und
dieser Identität aus und behandelt die Zustellung weiterhin als fehlgeschlagen, damit Aufrufer einen teilweisen
Erfolg nicht mit einem fehlerfreien Versand verwechseln. Melden Sie `visibleReplySent: false` nicht, nachdem eine
Vorschau, ein Entwurf, ein Anhang oder eine endgültige Nachricht sichtbar wurde.

Wenn `reply_payload_sending` oder `message_sending` registriert ist, müssen diese Hooks
abgeschlossen sein, bevor etwas für den Provider Sichtbares erstellt wird, da jeder der Hooks
die logische Nutzlast umschreiben oder abbrechen kann. Eine voreilig erstellte native Vorschau würde
Inhalt vor dem Umschreiben offenlegen oder einen abgebrochenen Entwurf hinterlassen. Puffern Sie Vorschauinhalte,
bis die akzeptierte Nutzlast `deliver` erreicht; Kompatibilitäts-Dispatcher, die
Vorschauen früher starten, müssen diese voreilige Vorschau unterdrücken, solange einer der Hooks
registriert ist. Verwenden Sie für neue Vorschaupfade die finalisierbaren Live-Vorschau-Hilfsfunktionen aus der
[API für ausgehende Kanalnachrichten](/de/plugins/sdk-channel-outbound).

## Migration

`runtime.channel.turn.*`-Laufzeitaliase wurden entfernt. Verwenden Sie:

- `runtime.channel.inbound.run(...)` für rohe eingehende Ereignisse.
- `runtime.channel.inbound.dispatchReply(...)` für zusammengestellte Antwortkontexte.
- `runtime.channel.inbound.buildContext(...)` für eingehende Kontextnutzlasten.
- `runtime.channel.inbound.runPreparedReply(...)`, veraltet, nur für
  kanaleigene vorbereitete Weiterleitungspfade, die bereits ihre eigene
  Weiterleitungs-Closure zusammenstellen.

Neuer Plugin-Code sollte keine Kanal-APIs mit `turn` im Namen einführen. Beschränken Sie Modell- oder
Agenten-Turn-Terminologie auf Agenten-/Provider-Code; Kanal-Plugins verwenden Begriffe für Eingang,
Nachricht, Zustellung und Antwort.
