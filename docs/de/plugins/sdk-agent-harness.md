---
read_when:
    - Sie ändern die eingebettete Agentenlaufzeit oder die Harness-Registry
    - Sie registrieren ein Agent-Harness aus einem gebündelten oder vertrauenswürdigen Plugin
    - Sie müssen verstehen, wie das Codex-Plugin mit Modell-Providern zusammenhängt.
sidebarTitle: Agent Harness
summary: Experimentelle SDK-Schnittstelle für Plugins, die den eingebetteten Agent-Executor auf niedriger Ebene ersetzen
title: Agent-Harness-Plugins
x-i18n:
    generated_at: "2026-07-24T22:25:04Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 4ff4e41a46ba0074fc6c8bf46da813b58d074f5e6c5c1d236d7ab78e824bdc02
    source_path: plugins/sdk-agent-harness.md
    workflow: 16
---

Ein **Agent-Harness** ist der Low-Level-Executor für einen vorbereiteten OpenClaw-Agent-
Durchlauf. Er ist weder ein Modell-Provider noch ein Kanal oder eine Tool-Registry. Das
benutzerorientierte mentale Modell finden Sie unter [Agent-Runtimes](/de/concepts/agent-runtimes).

Verwenden Sie diese Oberfläche nur für gebündelte oder vertrauenswürdige native Plugins. Der Vertrag ist
weiterhin experimentell, da die Parametertypen absichtlich den
aktuellen eingebetteten Runner abbilden.

## Wann ein Harness verwendet werden sollte

Registrieren Sie ein Agent-Harness, wenn eine Modellfamilie über eine eigene native Sitzungs-
Runtime verfügt und der normale OpenClaw-Provider-Transport die falsche Abstraktion darstellt:

- ein nativer Coding-Agent-Server, der Threads und Compaction verwaltet
- eine lokale CLI oder ein Daemon, die bzw. der native Planungs-, Reasoning- und Tool-Ereignisse streamen muss
- eine Modell-Runtime, die zusätzlich zum OpenClaw-
  Sitzungstranskript eine eigene Fortsetzungs-ID benötigt

Registrieren Sie **kein** Harness, nur um eine neue LLM-API hinzuzufügen. Erstellen Sie für normale HTTP-
oder WebSocket-Modell-APIs ein [Provider-Plugin](/de/plugins/sdk-provider-plugins).

## Wofür der Core weiterhin zuständig ist

Bevor ein Harness ausgewählt wird, hat OpenClaw bereits Folgendes aufgelöst:

- Provider und Modell
- Runtime-Authentifizierungsstatus, sofern das Harness nicht angibt, dass es den Authentifizierungs-Bootstrap übernimmt
- Thinking-Level und Kontextbudget
- die OpenClaw-Transkript-/Sitzungsdatei
- Workspace-, Sandbox- und Tool-Richtlinie
- Callbacks für Kanalantworten und Streaming
- Richtlinie für Modell-Fallback und Live-Modellwechsel

Ein Harness führt einen vorbereiteten Versuch aus; es wählt keine Provider aus, ersetzt nicht die
Kanalauslieferung und wechselt nicht unbemerkt das Modell.

### Harness-eigener Authentifizierungs-Bootstrap

Standardmäßig löst der Core die Provider-Anmeldedaten auf, bevor er ein Harness aufruft. Ein
vertrauenswürdiges Harness, das sich über seine eigene native Runtime authentifizieren kann, darf
`authBootstrap: "harness"` in seiner statischen `AgentHarness`-Registrierung festlegen. Der Core
überspringt dann seinen generischen Bootstrap für Provider-Anmeldedaten und den Fehler wegen fehlender Anmeldedaten
bei jedem Versuch, den dieses Harness übernimmt.

Der Core leitet weiterhin ein kompatibles, ausdrücklich ausgewähltes oder geordnetes OpenClaw-Authentifizierungs-
profil und dessen bereichsgebundenen Store weiter, sofern eines vorhanden ist. Das Harness muss dieses
Profil oder seine nativen Anmeldedaten auflösen, bevor es Modellanfragen sendet, Geheimnisse
auf den Versuch beschränken und aussagekräftige Authentifizierungsfehler ausgeben. Legen Sie
diese Fähigkeit nicht für ein Harness fest, das die Authentifizierung nur gelegentlich übernimmt.

### Verifizierte Runtime-Artefakte für die Einrichtung

Ein lokales Harness, das Inferenz für die Ersteinrichtung bereitstellen kann, muss die
Implementierung bestätigen, die die Prüfung abgeschlossen hat. Wenn
`params.captureRuntimeArtifact` den Wert „true“ hat, geben Sie ein opakes
`result.runtimeArtifact` mit einer stabilen ID und einem Inhaltsfingerabdruck zurück. Registrieren Sie eine
passende `runtimeArtifact.validate(...)`-Fähigkeit, die diese Bindung erneut prüft,
ohne ein anderes Harness zu laden oder nicht zugehörige Plugins zu durchsuchen.

Verifizierte OpenClaw-Fortsetzungen übergeben außerdem `params.expectedRuntimeArtifact`.
Das Harness muss diesen Wert mit dem exakten nativen Prozess vergleichen, den es übernommen hat, und einen Fehler
ausgeben, bevor es einen nativen Thread startet oder fortsetzt, falls sie voneinander abweichen. Bei gewöhnlichen Agent-
Durchläufen fehlen beide Felder, sodass die Inhalts-Hash-Bildung nicht im normalen Hot Path der Anfrage
erfolgt. Remote-/WebSocket-Harnesses benötigen einen Server-Attestierungsvertrag, bevor
sie teilnehmen können; eine Versionszeichenfolge allein ist keine Artefaktidentität.

Der vorbereitete Versuch enthält außerdem `params.runtimePlan`, ein OpenClaw-eigenes
Richtlinienpaket für Runtime-Entscheidungen, die zwischen OpenClaw und
nativen Harnesses einheitlich bleiben müssen:

- `runtimePlan.tools.normalize(...)` und `runtimePlan.tools.logDiagnostics(...)`
  für Provider-bezogene Richtlinien für Tool-Schemas
- `runtimePlan.transcript.resolvePolicy(...)` für die Bereinigung von Transkripten und
  die Richtlinie zur Reparatur von Tool-Aufrufen
- `runtimePlan.delivery.isSilentPayload(...)` für gemeinsame `NO_REPLY` und die Unterdrückung
  der Medienauslieferung
- `runtimePlan.outcome.classifyRunResult(...)` für die Klassifizierung von
  Modell-Fallbacks
- `runtimePlan.observability` für aufgelöste Provider-/Modell-/Harness-Metadaten

Harnesses dürfen den Plan für Entscheidungen verwenden, die dem Verhalten von OpenClaw
entsprechen müssen, sollten ihn jedoch als Host-eigenen Versuchszustand behandeln: Verändern Sie ihn nicht und verwenden
Sie ihn nicht, um innerhalb eines Durchlaufs Provider oder Modelle zu wechseln.

### Vertrag für den Anfragetransport

`supports(ctx)` empfängt den aufgelösten Modelltransport in `ctx.modelProvider`.
Zwei geheimnisfreie, Provider-eigene Fakten beschreiben die ausgewählte Route:

- `runtimePolicy.compatibleIds` führt die Runtime-IDs auf, die der Provider
  als mit dieser konkreten Route kompatibel deklariert. Eine fehlende Richtlinie bedeutet, dass der Provider
  keine Kompatibilität auf Routenebene deklariert hat; sie ist keine Erlaubnis, Unterstützung anzunehmen.
- `requestTransportOverrides: "none"` bedeutet, dass keine ausdrücklich definierte Provider-/Modell-
  Überschreibung der Anfrage reproduziert werden muss. `"present"` bedeutet, dass ausdrücklich definierte Header, ein Authentifizierungs-
  transport, Proxy-, TLS-, lokaler Dienst-, privates Netzwerk-Verhalten oder Anfrage-
  parameter vorhanden sind. Das Faktum legt diese Werte nicht offen.

Geben Sie `{ supported: false, reason }` zurück, wenn das Harness den
vorbereiteten Transport nicht reproduzieren kann. Leiten Sie die Unterstützung nach der Auswahl nicht durch das Lesen der Rohkonfiguration ab.
Wenn die Authentifizierungsvorbereitung mehrere Wiederholungsrouten ergibt, muss ein Harness
alle unterstützen, bevor die Ausführung erfolgt. Bei impliziter Auswahl wird OpenClaw verwendet, wenn kein Plugin
den vollständigen Satz übernehmen kann; eine ausdrückliche oder persistierte Plugin-Auswahl schlägt sicher fehl.

## Harness registrieren

**Import:** `openclaw/plugin-sdk/agent-harness`

```typescript
import type { AgentHarness } from "openclaw/plugin-sdk/agent-harness";
import { definePluginEntry } from "openclaw/plugin-sdk/plugin-entry";

const myHarness: AgentHarness = {
  id: "my-harness",
  label: "Mein natives Agent-Harness",

  supports(ctx) {
    const routeSupportsHarness =
      ctx.modelProvider?.runtimePolicy?.compatibleIds.includes("my-harness") === true;
    const canReproduceRequest = ctx.modelProvider?.requestTransportOverrides !== "present";
    return ctx.provider === "my-provider" && routeSupportsHarness && canReproduceRequest
      ? { supported: true, priority: 100 }
      : { supported: false, reason: "die effektive Route ist nicht mit dem Harness kompatibel" };
  },

  async runAttempt(params) {
    // Starten Sie Ihren nativen Thread oder setzen Sie ihn fort.
    // Verwenden Sie params.prompt, params.tools, params.images, params.onPartialReply,
    // params.onAgentEvent und die anderen Felder des vorbereiteten Versuchs.
    return await runMyNativeTurn(params);
  },
};

export default definePluginEntry({
  id: "my-native-agent",
  name: "Mein nativer Agent",
  description: "Führt ausgewählte Modelle über einen nativen Agent-Daemon aus.",
  register(api) {
    api.registerAgentHarness(myHarness);
  },
});
```

`authBootstrap` fehlt in diesem generischen Beispiel absichtlich. Fügen Sie
`authBootstrap: "harness"` nur hinzu, wenn das Harness den oben beschriebenen Vertrag erfüllt.

### Delegierte Ausführung

Der Eigentümer eines Harnesses darf `delegatedExecutionPluginIds` auf die IDs vertrauenswürdiger
Plugins festlegen, die eine vorhandene, an ein Modell gebundene Sitzung ausführen müssen, beispielsweise ein Sprach-
transport, der eine Codex-gestützte Unterhaltung fortsetzt. Dies ist eine statische Zustimmung des Eigentümers
und keine Core-Zulassungsliste. Halten Sie sie eng begrenzt.

Delegierte erhalten nur die Arbeitszulassung und eingebettete Ausführung. OpenClaw verlangt
den exakt gespeicherten Sitzungsschlüssel, Store-Pfad und die Sitzungs-ID; `modelSelectionLocked:
true`; sowie übereinstimmende Werte für `agentHarnessId` und `agentHarnessRuntimeOverride`.
Die Ausführung wird anschließend über den Eigentümer des Harnesses bereichsgebunden. Erstellung, Änderung,
Zurücksetzung, Löschung und Archivierung von Sitzungen sowie Gateway-Mutationen bleiben ausschließlich dem Eigentümer vorbehalten.

## Auswahlrichtlinie

OpenClaw wählt nach der Auflösung von Provider und Modell ein Harness aus:

1. Die modellbezogene Runtime-Richtlinie hat Vorrang.
2. Danach folgt die Provider-bezogene Runtime-Richtlinie.
3. `auto` fragt registrierte Harnesses, ob sie die aufgelöste effektive
   Route unterstützen. Provider-/Modellpräfixe allein wählen niemals ein Harness aus.
4. Wenn kein registriertes Harness übereinstimmt, verwendet OpenClaw seine eingebettete Runtime.

Fehler von Plugin-Harnesses werden als Ausführungsfehler ausgegeben. Im Modus `auto`
gilt der eingebettete Fallback nur, wenn kein registriertes Plugin-Harness den aufgelösten
Provider bzw. das Modell unterstützt. Sobald ein Plugin-Harness eine Ausführung übernommen hat, spielt OpenClaw
denselben Durchlauf nicht über eine andere Runtime erneut ab, da dies
die Authentifizierungs-/Runtime-Semantik verändern oder Nebeneffekte duplizieren kann.

Die konfigurierte Runtime-Richtlinie bleibt für die gewünschte Runtime maßgeblich. Eine
persistierte Sitzung `agentHarnessId` behält die Zuständigkeit für ihr natives Transkript,
während die Routen-/Authentifizierungsvorbereitung noch aussteht. Keines von beiden macht eine inkompatible
Route kompatibel: Sobald vorbereitete Fakten vorhanden sind, muss das ausgewählte oder fixierte Harness
sie unterstützen, andernfalls schlägt die Ausführung sicher fehl. `/status` zeigt die effektive Runtime,
die anhand von Richtlinie, persistierter Zuständigkeit und Routenunterstützung ausgewählt wurde.
Der vorbereitete Status ist ausdrücklich angegeben: Ein fehlendes `runtimePolicy` bleibt undeklariert,
anstatt aus den zufällig vorhandenen Transportfeldern abgeleitet zu werden.
Wenn bei Harness-eigener Authentifizierung mehrere physische Routen unaufgelöst bleiben, entspricht
das vorbereitete Unterstützungsfaktum der Schnittmenge ihrer kompatiblen Runtime-IDs und
meldet Anfrageüberschreibungen, falls irgendein Kandidat solche enthält. Ein undeklarierter Kandidat
führt daher dazu, dass die native Kompatibilität leer ist; `preparedAuth.source: "harness"`
ist ein Authentifizierungseigentümer und keine Erlaubnis, Routenunterstützung abzuleiten.

Wenn die Auswahl des Harnesses überraschend ist, aktivieren Sie das Debug-Logging `agents/harness`
und prüfen Sie den strukturierten `agent harness selected`-Datensatz des Gateways: Er
enthält die ID des ausgewählten Harnesses, den Auswahlgrund, die Runtime-/Fallback-Richtlinie
und im Modus `auto` das Unterstützungsergebnis jedes Plugin-Kandidaten.

Das gebündelte Codex-Plugin registriert `codex` als seine Harness-ID. Der Core behandelt diese
wie eine gewöhnliche Plugin-Harness-ID; Codex-spezifische Aliasse gehören in das Plugin
oder die Betreiberkonfiguration und nicht in den gemeinsamen Runtime-Selektor.

## Kopplung von Provider und Harness

Die meisten Harnesses sollten außerdem einen Provider registrieren. Der Provider macht Modellreferenzen,
Authentifizierungsstatus, Modellmetadaten und die Auswahl `/model` für den Rest von
OpenClaw sichtbar. Das Harness übernimmt diesen Provider anschließend in `supports(...)`.

Das gebündelte Codex-Plugin folgt diesem Muster:

- bevorzugte Modellreferenzen für Benutzer: `openai/gpt-5.6-sol`
- Kompatibilitätsreferenzen: Veraltete `codex/gpt-*`-Referenzen werden weiterhin akzeptiert, neue
  Konfigurationen sollten sie jedoch nicht als normale Provider-/Modellreferenzen verwenden
- Harness-ID: `codex`
- Authentifizierung: synthetische Provider-Verfügbarkeit, da das Codex-Harness die
  native Codex-Anmeldung/-Sitzung verwaltet
- App-Server-Anfrage: OpenClaw sendet die reine Modell-ID an Codex und überlässt dem
  Harness die Kommunikation mit dem nativen App-Server-Protokoll

Das Codex-Plugin ist additiv. Wenn die Runtime-Richtlinie nicht festgelegt ist oder `auto` entspricht, darf OpenAI
Codex nur auswählen, wenn sein Provider-eigener Routenvertrag `codex`
als kompatibel deklariert: eine exakte offizielle HTTPS-Route für Platform Responses oder ChatGPT Responses
ohne ausdrücklich definierte Anfrageüberschreibung. Das Präfix `openai/*` allein
wählt Codex niemals aus. Benutzerdefinierte Endpunkte, Completions-Adapter und ausdrücklich definiertes Anfrage-
verhalten verbleiben bei OpenClaw. Offizielle Klartext-HTTP-Endpunkte werden abgelehnt. Ältere `codex/gpt-*`-
Referenzen bleiben Kompatibilitätseingaben. Siehe
[Implizite OpenAI-Agent-Runtime](/de/providers/openai#implicit-agent-runtime).

Informationen zur Einrichtung durch Betreiber, Beispiele für Modellpräfixe und reine Codex-Konfigurationen finden Sie unter
[Codex-Harness](/de/plugins/codex-harness).

Das Codex-Plugin erzwingt die unter
[Codex-Harness](/de/plugins/codex-harness) dokumentierte Mindestversion des App-Servers. Es prüft den Initialisierungs-Handshake und
blockiert ältere Server oder Server ohne Versionsangabe, sodass OpenClaw nur mit der
getesteten Protokolloberfläche arbeitet.

### Middleware für Tool-Ergebnisse

Gebündelte Plugins und ausdrücklich aktivierte installierte Plugins mit übereinstimmenden
Manifestverträgen können über
`api.registerAgentToolResultMiddleware(...)` Runtime-neutrale Middleware für Tool-Ergebnisse einbinden, wenn ihr Manifest die
betroffenen Runtime-IDs in `contracts.agentToolResultMiddleware` deklariert. Diese vertrauenswürdige
Schnittstelle ist für asynchrone Transformationen von Tool-Ergebnissen vorgesehen, die ausgeführt werden müssen, bevor OpenClaw oder
Codex Tool-Ausgaben an das Modell zurückgibt.

Legacy-gebündelte Plugins können weiterhin
`api.registerCodexAppServerExtensionFactory(...)` für reine Codex-App-Server-
Middleware verwenden, neue Ergebnistransformationen sollten jedoch die laufzeitneutrale API verwenden. Der
nur für den eingebetteten Runner bestimmte Hook `api.registerEmbeddedExtensionFactory(...)` wurde
entfernt; eingebettete Tool-Ergebnistransformationen müssen laufzeitneutrale Middleware verwenden.

### Klassifizierung des Terminalergebnisses

Native Harnesses, die ihre eigene Protokollprojektion verwalten, können
`classifyAgentHarnessTerminalOutcome(...)` aus
`openclaw/plugin-sdk/agent-harness-runtime` verwenden, wenn ein abgeschlossener Turn keinen
sichtbaren Assistententext erzeugt hat. Der Helfer gibt `empty`, `reasoning-only` oder
`planning-only` zurück, damit die Fallback-Richtlinie von OpenClaw entscheiden kann, ob ein erneuter Versuch mit einem
anderen Modell erfolgen soll. `planning-only` erfordert das explizite Feld `planText`
des Harnesses; OpenClaw leitet es nicht aus Assistentenprosa ab. Der Helfer
lässt Prompt-Fehler, laufende Turns und absichtlich stille
Antworten wie `NO_REPLY` bewusst unklassifiziert.

### Nebeneffekte am Agentenende

Native Harnesses müssen `runAgentEndSideEffects(...)` aus
`openclaw/plugin-sdk/agent-harness-runtime` aufrufen, nachdem sie einen Versuch abgeschlossen haben. Die Funktion
löst den portablen Hook `agent_end` und die Forschungserfassung von OpenClaw aus,
ohne interaktive Antworten zu verzögern. Verwenden Sie `awaitAgentEndSideEffects(...)` für
lokale, nicht interaktive Ausführungen, bei denen der Versuch erst abgeschlossen werden darf, nachdem diese
Nebeneffekte beendet sind. Beide Helfer akzeptieren dieselbe `{ event, ctx }`-Nutzlast wie
`runAgentHarnessAgentEndHook(...)`; ihre Fehler ändern das Ergebnis des abgeschlossenen
Versuchs nicht.

### Benutzereingabe- und Tool-Oberflächen

Native Harnesses, die eine Benutzereingabeanforderung auf Laufzeitebene bereitstellen, sollten die
Benutzereingabe-Helfer aus `openclaw/plugin-sdk/agent-harness-runtime` verwenden, um
den Prompt zu formatieren, ihn über den blockierenden Antwortpfad von OpenClaw zuzustellen und
Auswahl- beziehungsweise Freitextantworten zurück in die native Antwortstruktur der Laufzeit zu normalisieren. Der
Helfer hält die Darstellung in Kanal und TUI konsistent, während jedes Harness sein
eigenes Protokollparsing und den Lebenszyklus ausstehender Anforderungen verwaltet.

Native Harnesses, die ein PI-ähnliches kompaktes Tool-Routing benötigen, sollten
`createAgentHarnessToolSurfaceRuntime(...)` aus
`openclaw/plugin-sdk/agent-harness-tool-runtime` verwenden. Die Funktion verwaltet
die Auswahl der Tool-Suche beziehungsweise Code-Modus-Steuerung, schlanke Standardwerte für lokale Modelle,
laufzeitkompatible Schemafilterung, verborgene Katalogausführung, Verzeichnis-
Hydratisierung und Katalogbereinigung. Harnesses bleiben weiterhin für ihre SDK-spezifische Tool-
Konvertierung und den nativen Ausführungs-Callback zuständig.

### Nativer Codex-Harness-Modus

Das gebündelte Harness `codex` ist der native Codex-Modus für eingebettete OpenClaw-
Agenten-Turns. Aktivieren Sie zuerst das gebündelte Plugin `codex` und nehmen Sie `codex` in
`plugins.allow` auf, wenn Ihre Konfiguration eine restriktive Zulassungsliste verwendet. Native App-Server-
Konfigurationen sollten `openai/gpt-*` verwenden; OpenAI-Agenten-Turns wählen das Codex-Harness
nur aus, wenn die effektive Route Codex-Kompatibilität deklariert. Legacy-Codex-Modell-
Referenzen sollten mit `openclaw doctor --fix` repariert werden, und Legacy-Modellreferenzen vom Typ `codex/*`
bleiben Kompatibilitätsaliase für das native Harness.

Wenn dieser Modus ausgeführt wird, verwaltet Codex die native Thread-ID, das Fortsetzungsverhalten,
Compaction und die App-Server-Ausführung. OpenClaw verwaltet weiterhin den Chatkanal,
die sichtbare Transkriptspiegelung, Tool-Richtlinien, Genehmigungen, Medienzustellung und Sitzungs-
auswahl. Verwenden Sie Provider/Modell `agentRuntime.id: "codex"`, wenn Sie
nachweisen müssen, dass ausschließlich der Codex-App-Server-Pfad die Ausführung übernehmen kann. Explizite Plugin-
Laufzeiten schlagen geschlossen fehl; Auswahlfehler und Laufzeitfehler des Codex-App-Servers
werden nicht über eine andere Laufzeit erneut versucht.

## Laufzeitstrenge

Standardmäßig verwendet OpenClaw die Provider-/Modell-Laufzeitrichtlinie `auto`: Registrierte
Plugin-Harnesses können kompatible effektive Routen übernehmen, und die eingebettete
Laufzeit verarbeitet den Turn, wenn keine Übereinstimmung vorliegt. Ein Provider-/Modellpräfix allein
wählt niemals ein Harness aus. Verwenden Sie eine explizite Provider-/Modell-Plugin-Laufzeit wie
`agentRuntime.id: "codex"`, wenn eine fehlende Harness-Auswahl fehlschlagen soll,
anstatt über die eingebettete Laufzeit geleitet zu werden. Eine explizite Auswahl macht eine
inkompatible Route nicht kompatibel. Fehler ausgewählter Plugin-Harnesses führen immer zu einem
harten Fehlschlag. Dies blockiert kein explizites Provider-/Modell-
`agentRuntime.id: "openclaw"`.

Für ausschließlich Codex verwendende eingebettete Ausführungen:

```json
{
  "models": {
    "providers": {
      "openai": {
        "agentRuntime": {
          "id": "codex"
        }
      }
    }
  },
  "agents": {
    "defaults": {
      "model": "openai/gpt-5.6-sol"
    }
  }
}
```

Wenn Sie ein CLI-Backend für ein kanonisches Modell wünschen, legen Sie die Laufzeit in diesem
Modelleintrag fest:

```json
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-5",
      "models": {
        "anthropic/claude-opus-5": {
          "agentRuntime": {
            "id": "claude-cli"
          }
        }
      }
    }
  }
}
```

Überschreibungen pro Agent verwenden dieselbe modellbezogene Struktur:

```json
{
  "agents": {
    "list": [
      {
        "id": "codex-only",
        "model": "openai/gpt-5.6-sol",
        "models": {
          "openai/gpt-5.6-sol": {
            "agentRuntime": { "id": "codex" }
          }
        }
      }
    ]
  }
}
```

Legacy-Beispiele für eine Laufzeit auf Ebene des gesamten Agenten wie dieses werden ignoriert:

```json
{
  "agents": {
    "defaults": {
      "agentRuntime": {
        "id": "codex"
      }
    }
  }
}
```

Bei einer expliziten Plugin-Laufzeit schlägt eine Sitzung frühzeitig fehl, wenn das angeforderte
Harness nicht registriert ist, den aufgelösten Provider beziehungsweise das Modell nicht unterstützt oder
fehlschlägt, bevor Turn-Nebeneffekte erzeugt werden. Dies ist bei reinen Codex-
Bereitstellungen und bei Live-Tests beabsichtigt, die nachweisen müssen, dass der Codex-App-Server-Pfad
tatsächlich verwendet wird.

Diese Einstellung steuert nur das eingebettete Agenten-Harness. Sie deaktiviert nicht
das Provider-spezifische Modell-Routing für Bilder, Videos, Musik, TTS, PDF oder andere Formate.

## Native Sitzungen und Transkriptspiegelung

Ein Harness kann eine native Sitzungs-ID, Thread-ID oder ein daemonseitiges Fortsetzungs-
Token verwalten. Ordnen Sie diese Bindung explizit der OpenClaw-Sitzung zu und
spiegeln Sie für Benutzer sichtbare Assistenten-/Tool-Ausgaben weiterhin in das OpenClaw-
Transkript.

Das OpenClaw-Transkript bleibt die Kompatibilitätsschicht für:

- kanalsichtbaren Sitzungsverlauf
- Transkriptsuche und -indizierung
- den Wechsel zurück zum integrierten OpenClaw-Harness bei einem späteren Turn
- generisches Verhalten für `/new`, `/reset` und das Löschen von Sitzungen

Wenn Ihr Harness eine Sidecar-Bindung speichert, implementieren Sie `reset(...)`, damit OpenClaw
sie löschen kann, wenn die zugehörige OpenClaw-Sitzung zurückgesetzt wird.

## Tool- und Medienergebnisse

Der Kern erstellt die OpenClaw-Tool-Liste und übergibt sie an den vorbereiteten
Versuch. Wenn ein Harness einen dynamischen Tool-Aufruf ausführt, geben Sie das Tool-Ergebnis
über die Harness-Ergebnisstruktur zurück, anstatt Kanalmedien
selbst zu senden.

Dadurch verbleiben Text-, Bild-, Video-, Musik-, TTS-, Genehmigungs- und Messaging-Tool-
Ausgaben auf demselben Zustellpfad wie von OpenClaw unterstützte Ausführungen.

Setzen Sie `AgentHarnessAttemptResult.hostOwnedToolMediaUrls` nur für native Artefakte,
die die vertrauenswürdige Harness-Laufzeit selbst erstellt und persistiert hat. Jeder Eintrag muss
auch in `toolMediaUrls` enthalten sein. Nehmen Sie niemals durch das Modell ausgewählte Medien dynamischer Tools oder
OpenClaw-Tools auf. Bei `message_tool_only`-Routen ermöglicht diese enge Herkunftsdefinition,
dass Artefakte der nativen Laufzeit die Unterdrückung von Quellantworten überstehen; die normale Senderichtlinie
und die Zulassung für Umgebungsräume gelten weiterhin.

### Terminalergebnisse von Tools

`AgentHarnessAttemptParams.observeToolTerminal` ist der vom Host verwaltete Akkumulator für Terminal-
ergebnisse. Ein Harness, das dynamische OpenClaw-Tools oder native
Tools ausführt, muss ihn aufrufen, sobald jedes Tool genau ein Terminalergebnis erreicht, bevor das
Versuchsergebnis abgeschlossen wird. Harnesses, die keine Tools ausführen, müssen ihn nicht
aufrufen.

Melden Sie Fakten von der Ausführungsgrenze:

- Übergeben Sie die Protokollaufruf-ID, sofern vorhanden, den kanonischen Tool-Namen und die
  Argumente, die das Tool nach Vorbereitung oder Hook-Umschreibungen tatsächlich erreicht haben.
- Setzen Sie `executionStarted: false`, wenn Validierung, Genehmigung oder eine andere Schutzmaßnahme
  den Aufruf gestoppt hat, bevor die Tool-Implementierung begann. Sobald eine Übergabe möglicherweise
  stattgefunden hat, melden Sie vorsichtshalber `true`.
- Melden Sie `outcome: "success"` oder `outcome: "failure"`. Fügen Sie die strukturierten
  Fehlerfelder ein, die von der Laufzeit verfügbar sind, anstatt einen Fehler aus
  Anzeigetext abzuleiten.
- Verwenden Sie `nativeMutation` nur für native Tools, die keine OpenClaw-Tool-
  Definition verwenden. Geben Sie dort protokolleigene Mutations- und Replay-Fakten an; kopieren Sie nicht
  den Mutationsklassifizierer von OpenClaw in das Harness.

Der Callback gibt die kanonische Auflösung für diesen Aufruf zurück. Übernehmen Sie dessen
`lastToolError` in `AgentHarnessAttemptResult` und verwenden Sie dessen Ausführungs-,
Argument- und Nebeneffektfakten in der Harness-Projektion, anstatt
parallelen Zustand abzuleiten. Der Host behält einen ungelösten mutierenden Fehler über nicht zusammenhängende
erfolgreiche Tools hinweg bei und löscht ihn erst, nachdem die entsprechende Aktion erfolgreich ausgeführt wurde.

Der Callback bleibt für die Quellkompatibilität mit älteren experimentellen
Harnesses optional. Optional bedeutet für ein Harness, das Tools ausführt, nicht, dass er ignoriert werden kann:
Ohne Terminalberichte kann OpenClaw den Wahrheitswert eines Fehlers eines mutierenden Tools
über spätere Tool-Aufrufe hinweg nicht bewahren, einschließlich eines stillen Heartbeat-Abschlusses.

### Finalisierung abgeschlossener Tools

OpenClaw benötigt möglicherweise eine letzte sichtbare Antwort, nachdem ein Harness jeden
Tool-Aufruf abgeschlossen hat, sein nativer Turn jedoch ohne Assistententext endete. Ein Harness kann
diese Wiederherstellung durch Implementierung von `finalizeSettledTurn({ attempt,
settledAttempt })` aktivieren.

Der Callback ist eine separate Fähigkeit und kein weiterer gewöhnlicher Versuch. Er muss:

- entweder das exakt eingeschränkte native Transkript oder ein vollständiges Anwendungs-
  transkript verwenden, das bis einschließlich der Grenze des abgeschlossenen Tool-Ergebnisses eingefroren ist;
- keine Tools, Fähigkeiten zur Erteilung von Berechtigungen oder Benutzereingabe, nativen Ausführungs-
  Hooks, Agenten, Skills, Speicher, Zeitplanung, Erweiterungen oder Fernsteuerung bereitstellen;
- ausschließlich den vom Host bereitgestellten Finalisierungs-Prompt senden; und
- geschlossen fehlschlagen, wenn die ausgewählte Transkript-/Isolationsstrategie
  diese Einschränkungen nicht durchsetzen kann.

OpenClaw ruft den Callback einmal als abschließende Unteroperation außerhalb des
gewöhnlichen Versuchs- und Wiederholungszyklus auf. Ein Fehler beendet die Ausführung mit der
nebenwirkungsbezogenen Warnung vor einem unvollständigen Turn; er kann nicht in gewöhnliche
Authentifizierungs-/Profilrotation, Modell-Fallback, Kontextwiederherstellung, Compaction-
Fortsetzung oder durch Hooks angeforderte Überarbeitungspfade eintreten. Die Finalisierung überspringt außerdem Plugin-
Prompt-Mutation, `before_agent_run`, LLM-Eingabe/-Ausgabe, Terminalüberarbeitung und
`agent_end`-Hooks. Die Kerndiagnose zeichnet die Operation und ihren Fehler weiterhin auf.

Der Callback gibt `AgentHarnessSettledTurnFinalizationResult` und kein
gewöhnliches Versuchsergebnis zurück. Seine öffentlichen Felder sind auf die abgeschlossene
Assistentennachricht, die Nutzung des Finalisierungsaufrufs, Metadaten zur Transkriptinhaberschaft und
die Diagnosespur beschränkt. Tool-, Zustellungs-, Medien-, Spawn-, Lebenszyklus-, Replay-, Sitzungs- und
Fallback-Zustand können diese Ergebnisgrenze nicht überschreiten. Unbekannte Felder und Tool-Aufrufe des Assistenten
schlagen geschlossen fehl.

Ein Harness, das intern seine vollständige Versuchs-Engine wiederverwendet, kann vor der Rückgabe
`projectSettledTurnFinalizationAttemptResult(...)` aufrufen. Der Helfer
weist kanonische Fehler-, Tool-, Zustellungs-, Replay- und Lebenszyklusnachweise zurück und
projiziert anschließend nur das enge Ergebnis. Dies ist Tiefenverteidigung nach der nativen Isolation
und kein Ersatz für das Entfernen der nativen Fähigkeitsoberfläche.

Ein projektionsgestütztes Harness muss den vollständigen Kontext in
`settledAttempt.settledTurnFinalizationContext` mit
`source: "openclaw-transcript"` ablegen. Es muss den aktiven Zweig erfassen, nachdem der
abgeschlossene Turn gespiegelt wurde, nachweisen, dass der aktuelle Prompt und jeder aktuelle Tool-
Aufruf sowie jedes Ergebnis bis zu dieser Grenze vorhanden sind, und das resultierende Nachrichten-
Array einfrieren, bevor der Versuch zurückgegeben wird. Der Finalisierer muss einen fehlenden,
nicht unterstützten, mehrdeutigen oder übergroßen Kontext zurückweisen. Er darf Nachrichten nicht abschneiden,
frühere Verlaufsdaten verwerfen oder dieses Anwendungstranskript als exakten nativen
Verlauf bezeichnen. Harnesses, die eine einzige eingeschränkte native Sitzung fortsetzen, benötigen dieses
Projektionsfeld nicht.

Implementieren Sie diesen Callback nicht, indem Sie `runAttempt` mit einem Best-Effort-
Hinweis `disableTools` aufrufen. Der Eigentümer des Harnesses muss die vollständige native
Fähigkeitsgrenze durchsetzen. OpenClaw stellt keinen generischen Fallback bereit, da es
nicht bestätigen kann, dass eine beliebige native Laufzeit diese Einschränkungen eingehalten hat.

Der Callback bleibt für die Kompatibilität mit experimentellen Harnesses von Drittanbietern optional. Wenn der ausgewählte Harness ihn nicht bereitstellt, behält OpenClaw den bestehenden Fehler für unvollständige Turns bei, statt wiederholte Nebeneffekte zu riskieren.

## Aktuelle Einschränkungen

- Der öffentliche Importpfad ist generisch, einige Typaliase für Versuche/Ergebnisse tragen aus Kompatibilitätsgründen jedoch weiterhin veraltete Namen.
- Die Installation von Harnesses von Drittanbietern ist experimentell. Bevorzugen Sie Provider-Plugins, bis Sie eine native Sitzungs-Runtime benötigen.
- Der Wechsel des Harnesses zwischen Turns wird unterstützt. Wechseln Sie den Harness nicht mitten in einem Turn, nachdem native Tools, Genehmigungen, Assistententext oder das Senden von Nachrichten begonnen haben.

## Verwandte Themen

- [SDK-Übersicht](/de/plugins/sdk-overview)
- [Runtime-Hilfsfunktionen](/de/plugins/sdk-runtime)
- [Provider-Plugins](/de/plugins/sdk-provider-plugins)
- [Codex-Harness](/de/plugins/codex-harness)
- [Modell-Provider](/de/concepts/model-providers)
