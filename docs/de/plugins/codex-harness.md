---
read_when:
    - Sie möchten das offizielle Codex-App-Server-Harness verwenden
    - Sie benötigen Konfigurationsbeispiele für das Codex-Harness
    - Sie möchten, dass reine Codex-Bereitstellungen fehlschlagen, anstatt auf OpenClaw zurückzufallen
summary: Führen Sie eingebettete OpenClaw-Agentenrunden über das offizielle Codex-App-Server-Testsystem aus
title: Codex-Harness
x-i18n:
    generated_at: "2026-07-24T13:48:00Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: e016a1689af65c5520d529ce22a87bd25ee29369f7aedca77b27f943a7f21b0f
    source_path: plugins/codex-harness.md
    workflow: 16
---

Das offizielle `codex`-Plugin führt eingebettete OpenAI-Agent-Durchläufe über den Codex
app-server statt über das integrierte OpenClaw-Harness aus. Codex verwaltet die
Low-Level-Agent-Sitzung: natives Fortsetzen von Threads, native Fortsetzung von Tools,
native Compaction und app-server-Ausführung. OpenClaw verwaltet weiterhin Chat-
Kanäle, Sitzungsdateien, Modellauswahl, dynamische OpenClaw-Tools, Genehmigungen,
Medienübermittlung und die sichtbare Transkriptspiegelung.

Verwenden Sie kanonische OpenAI-Modellreferenzen wie `openai/gpt-5.6-sol`. Konfigurieren Sie keine
veralteten Codex-GPT-Referenzen; legen Sie die OpenAI-Agent-Authentifizierungsreihenfolge unter `auth.order.openai` ab.
Veraltete Codex-Authentifizierungsprofil-IDs und veraltete Einträge der Codex-Authentifizierungsreihenfolge werden
durch `openclaw doctor --fix` repariert.

Wenn die Provider-/Modell-Laufzeitrichtlinie nicht gesetzt ist oder `auto` lautet, wählt das Präfix `openai/*` allein
dieses Harness niemals aus. OpenAI kann Codex nur für eine
exakte offizielle HTTPS-Platform-Responses- oder ChatGPT-Responses-Route ohne
benutzerdefinierte Anfrageüberschreibung implizit auswählen. Siehe
[Implizite OpenAI-Agent-Laufzeit](/de/providers/openai#implicit-agent-runtime).
Wenn Codex die Authentifizierung verwaltet, bevor das Routing zwischen Platform und ChatGPT bekannt ist, verlangt OpenClaw
weiterhin, dass jede infrage kommende Route Codex-Kompatibilität deklariert. Allein die native
Verwaltung der Authentifizierung umgeht diese Routenprüfung niemals.

Wenn keine OpenClaw-Sandbox aktiv ist, startet OpenClaw Codex-app-server-Threads
mit aktiviertem nativen Codex-Codemodus (nur Codemodus bleibt standardmäßig deaktiviert), sodass
native Arbeitsbereichs-/Codefunktionen zusammen mit dynamischen OpenClaw-
Tools verfügbar bleiben, die über die app-server-`item/tool/call`-Bridge geleitet werden. Eine
aktive OpenClaw-Sandbox oder eingeschränkte Tool-Richtlinie deaktiviert den nativen Codemodus
vollständig, sofern Sie nicht den experimentellen Sandbox-exec-server-Pfad aktivieren.

Mit dem standardmäßigen `tools.exec.host: "auto"` und ohne aktive OpenClaw-Sandbox
erhält Codex außerdem die Tools `node_exec` und `node_process` für Befehle auf gekoppelten
Nodes. Die native Shell verbleibt auf dem Host und im Arbeitsbereich des Codex app-server
(bei der standardmäßigen stdio-Bereitstellung lokal zum Gateway); `node_exec` wählt einen Node anhand
seines Namens oder seiner ID aus und behält die Node-Genehmigungsrichtlinie von OpenClaw bei. Wenn eine endliche
Laufzeit-Zulassungsliste den nativen Codemodus deaktiviert und für den Durchlauf keine
Ausführungsumgebung verbleibt, hält OpenClaw stattdessen seine richtliniengefilterten Tools `exec` und `process`
für die direkte Ausführung ohne Sandbox verfügbar.

Diese Codex-native Funktion unterscheidet sich vom
[OpenClaw-Codemodus](/de/tools/code-mode), einer optionalen QuickJS-WASI-Laufzeit
für allgemeine OpenClaw-Durchläufe mit einer anderen `exec`-Eingabeform. Einen Überblick über die
umfassendere Trennung von Modell, Provider und Laufzeit finden Sie unter
[Agent-Laufzeiten](/de/concepts/agent-runtimes): `openai/gpt-5.6-sol` ist die Modell-
referenz, `codex` ist die Laufzeit, und Telegram, Discord, Slack oder ein anderer
Kanal ist die Kommunikationsoberfläche.

## Voraussetzungen

- Das offizielle `@openclaw/codex`-Plugin muss installiert sein. Nehmen Sie `codex` in
  `plugins.allow` auf, wenn Ihre Konfiguration eine Zulassungsliste verwendet.
- Ein stabiler Codex app-server von `0.143.0` bis `0.145.0`. Das Plugin verwaltet standardmäßig eine kompatible
  Binärdatei, sodass ein `codex`-Befehl unter `PATH` den normalen
  Start nicht beeinflusst.
- Codex-Authentifizierung über `openclaw models auth login --provider openai`, ein
  bereits im Codex-Home des Agenten vorhandenes app-server-Konto oder ein
  explizites Codex-API-Schlüssel-Authentifizierungsprofil.

Informationen zur Authentifizierungspriorität, Umgebungsisolierung, zu benutzerdefinierten app-server-Befehlen,
zur Modellerkennung und zur vollständigen Liste der Konfigurationsfelder finden Sie in der
[Codex-Harness-Referenz](/de/plugins/codex-harness-reference).

## Schnellstart

Installieren Sie das offizielle Plugin und melden Sie sich anschließend mit Codex OAuth an:

```bash
openclaw plugins install @openclaw/codex
openclaw models auth login --provider openai
```

Aktivieren Sie das `codex`-Plugin und wählen Sie ein OpenAI-Agent-Modell aus:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
      },
    },
  },
  agents: {
    defaults: {
      model: "openai/gpt-5.6-sol",
    },
  },
}
```

Wenn Ihre Konfiguration `plugins.allow` verwendet, fügen Sie dort auch `codex` hinzu:

```json5
{
  plugins: {
    allow: ["codex"],
    entries: {
      codex: {
        enabled: true,
      },
    },
  },
}
```

Starten Sie das Gateway nach Änderungen an der Plugin-Konfiguration neu. Wenn ein Chat bereits über eine
Sitzung verfügt, führen Sie zuerst `/new` oder `/reset` aus, damit der nächste Durchlauf das Harness
anhand der aktuellen Konfiguration bestimmt.

## Threads mit Codex Desktop und der CLI teilen

Das standardmäßige `appServer.homeScope: "agent"` isoliert jeden OpenClaw-Agenten vom
nativen Codex-Zustand des Betreibers. Damit ein Eigentümer dieselben nativen Threads einsehen und verwalten kann,
die in Codex Desktop und der Codex-CLI angezeigt werden, aktivieren Sie das
Codex-Home des Benutzers:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          appServer: {
            homeScope: "user",
          },
        },
      },
    },
  },
}
```

Der Benutzer-Home-Modus unterstützt einen lokal verwalteten stdio-Prozess oder den gemeinsam genutzten Unix-Socket-
Transport. Er verwendet `$CODEX_HOME`, wenn gesetzt, andernfalls `~/.codex`, einschließlich
der nativen Codex-Authentifizierung, Konfiguration, Plugins und des Thread-Speichers dieses Homes. OpenClaw
injiziert kein OpenClaw-Authentifizierungsprofil in diesen app-server.

Eigentümer-Durchläufe erhalten das Tool `codex_threads`: native Threads auflisten, durchsuchen, lesen, forken, umbenennen,
archivieren und wiederherstellen. Forken Sie einen Thread, um ihn in
OpenClaw fortzusetzen; der Fork wird an die aktuelle OpenClaw-Sitzung angehängt und bleibt
für andere native Codex-Clients sichtbar. Die Archivierung erfordert eine ausdrückliche
Bestätigung, dass der Thread andernorts geschlossen ist. Wenn außerdem die Überwachung
aktiviert ist, erfordern Transkriptfelder und Änderungen die entsprechende
Aktivierung von `supervision.allowRawTranscripts` oder `supervision.allowWriteControls`.

Setzen Sie denselben Thread nicht gleichzeitig über unabhängige verwaltete
stdio-App-Server fort und schreiben Sie nicht gleichzeitig darüber. Codex koordiniert aktive Schreibvorgänge innerhalb eines App Servers, nicht
prozessübergreifend. Das Forken ist für gewöhnliche
Benutzer-Home-stdio-Sitzungen der sichere Weg zur Koexistenz.

`appServer.homeScope: "user"` allein steuert den Flottenkatalog nicht. Die native
Sitzungserkennung ist aktiviert, solange das Plugin aktiv ist; setzen Sie
`sessionCatalog.enabled: false`, um sie aus der OpenClaw-Seitenleiste zu entfernen, ohne
Codex zu deaktivieren. Der Katalog verwendet eine separate Überwachungsverbindung; ohne
explizite `appServer`-Verbindungseinstellungen verwendet diese Verbindung standardmäßig verwaltetes
Benutzer-Home-stdio, während das gewöhnliche Harness agentenspezifisch bleibt. Explizite
`appServer`-Einstellungen werden von beiden Pfaden berücksichtigt. Setzen Sie `homeScope: "user"`
wie oben ausdrücklich, wenn auch das gewöhnliche Harness den nativen Zustand teilen soll.

## Codex-Sitzungen überwachen

Dasselbe `codex`-Plugin kann nicht archivierte Codex-Sitzungen vom Gateway-
Computer und von dafür aktivierten gekoppelten Nodes auflisten. Eine gespeicherte oder inaktive Gateway-lokale Sitzung kann
einen modellgebundenen Chat erstellen, der ihren begrenzten, dauerhaft gespeicherten Benutzer- und Assistenten-
verlauf spiegelt. Seine private Bindung verwendet die Überwachungsverbindung für den nativen
Snapshot, den kanonischen Branch und spätere Durchläufe, während gewöhnliche Codex-Sitzungen
agentenspezifisch bleiben. Der erste kanonische Start verwendet exakt das Modell und den Provider, die
Codex für den Snapshot-Fork zurückgibt. Bei späteren Fortsetzungen bleibt die Auswahl der nativen
Codex-Konfiguration überlassen; das äußere OpenClaw-Modell und die Fallback-Kette ersetzen
sie niemals. Gespeicherte und inaktive Zeilen können nach ausdrücklicher Bestätigung,
dass kein anderer Runner aktiv ist, archiviert werden. Aktive Quellen können keinen Branch erstellen und nicht archiviert werden; ein vorhandener
überwachter Chat kann weiterhin geöffnet werden. Sitzungen gekoppelter Nodes bleiben auf Metadaten beschränkt.

Informationen zur Einrichtung, zu Branching-
Regeln, Beschränkungen gekoppelter Nodes, zur Offenlegung von Metadaten und zur Fehlerbehebung finden Sie unter [Codex-Sitzungen überwachen](/de/plugins/codex-supervision).

## Konfiguration

| Bedarf                                              | Einstellung                                                                                      | Ort                                |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ---------------------------------- |
| Harness aktivieren                                  | `plugins.entries.codex.enabled: true`                                                            | OpenClaw-Konfiguration             |
| Native Codex-Sitzungserkennung ausblenden           | `plugins.entries.codex.config.sessionCatalog.enabled: false`                                     | Codex-Plugin-Konfiguration         |
| Eine Plugin-Installation mit Zulassungsliste beibehalten | `codex` in `plugins.allow` aufnehmen                                                      | OpenClaw-Konfiguration             |
| Berechtigten OpenAI-Durchläufen die implizite Verwendung von Codex erlauben | Exakte offizielle HTTPS-Responses-/ChatGPT-Route, keine benutzerdefinierte Anfrageüberschreibung, Laufzeit nicht gesetzt/`auto` | OpenAI-Provider-/Modellkonfiguration |
| Mit ChatGPT/Codex OAuth anmelden                    | `openclaw models auth login --provider openai`                                                   | CLI-Authentifizierungsprofil       |
| API-Schlüssel-Backup für Codex-Durchläufe hinzufügen | `openai:*`-API-Schlüsselprofil, das in `auth.order.openai` nach der Abonnementauthentifizierung aufgeführt ist | CLI-Authentifizierungsprofil + OpenClaw-Konfiguration |
| Geschlossen fehlschlagen, wenn Codex nicht verfügbar ist | Provider oder Modell `agentRuntime.id: "codex"`                                                     | OpenClaw-Modell-/Provider-Konfiguration |
| Direkten OpenAI-API-Datenverkehr verwenden          | Provider oder Modell `agentRuntime.id: "openclaw"` mit normaler OpenAI-Authentifizierung                         | OpenClaw-Modell-/Provider-Konfiguration |
| app-server-Verhalten abstimmen                      | `plugins.entries.codex.config.appServer.*`                                                       | Codex-Plugin-Konfiguration         |
| Native Codex-Plugin-Apps aktivieren                 | `plugins.entries.codex.config.codexPlugins.*`                                                    | Codex-Plugin-Konfiguration         |
| Codex Computer Use aktivieren                       | `plugins.entries.codex.config.computerUse.*`                                                     | Codex-Plugin-Konfiguration         |

Bevorzugen Sie `auth.order.openai` für die Reihenfolge „Abonnement zuerst, API-Schlüssel als Backup“.
Vorhandene veraltete Codex-Authentifizierungsprofil-IDs und die veraltete Codex-Authentifizierungsreihenfolge sind
Altzustände, die ausschließlich durch doctor behandelt werden; schreiben Sie keine neuen veralteten Codex-GPT-Referenzen.

```json5
{
  auth: {
    order: {
      openai: ["openai:user@example.com", "openai:api-key-backup"],
    },
  },
}
```

Für eine effektive Codex-kompatible Route bleiben beide obigen Profile Kandidaten
für denselben Codex-Durchlauf. Die Profilreihenfolge wählt die Anmeldedaten, nicht die Laufzeit.
Eine Änderung der Authentifizierungsreihenfolge macht eine benutzerdefinierte, Completions-, HTTP- oder
anfrageüberschriebene Route nicht Codex-kompatibel.

### Compaction

Setzen Sie `compaction.model` oder `compaction.provider` nicht für Codex-gestützte
Agenten. Codex führt Compaction über seinen nativen app-server-Thread-Zustand durch, sodass
OpenClaw diese lokalen Überschreibungen des Zusammenfassers zur Laufzeit ignoriert und
`openclaw doctor --fix` sie entfernt, wenn der Agent Codex verwendet.

Lossless wird weiterhin als Kontext-Engine für Zusammenstellung, Aufnahme und
Wartung rund um Codex-Durchläufe unterstützt und über
`plugins.slots.contextEngine: "lossless-claw"` und
`plugins.entries.lossless-claw.config.summaryModel` konfiguriert, nicht über
`agents.defaults.compaction.provider`. `openclaw doctor --fix` migriert die
alte `compaction.provider: "lossless-claw"`-Form in den Lossless-
Kontext-Engine-Slot, wenn Codex die aktive Laufzeit ist, aber natives Codex verwaltet weiterhin
Compaction. Das native app-server-Harness unterstützt Kontext-Engines,
die eine Zusammenstellung vor dem Prompt benötigen; allgemeine CLI-Backends, einschließlich `codex-cli`,
stellen diese Host-Funktion nicht bereit.

Für Codex-gestützte Agenten startet `/compact` die native Codex-app-server-
Compaction auf dem gebundenen Thread und wartet auf ihr Endergebnis. Das gemeinsame
`agents.defaults.compaction.timeoutSeconds`-Budget gilt; bei einem Timeout
fordert OpenClaw Codex auf, den nativen Durchlauf zu unterbrechen, und behält die threadbezogene Sperre
bei, bis die Beendigung bestätigt ist. Es erfolgt niemals ein Fallback auf eine Kontext-Engine oder
einen öffentlichen OpenAI-Zusammenfasser. Wenn die native Codex-Thread-Bindung fehlt oder
veraltet ist, schlägt der Befehl geschlossen fehl, statt stillschweigend den Compaction-
Backend zu wechseln.

### Direkte API mit langem Kontext

Codex-Abonnement und direkter OpenAI-API-Datenverkehr sind separate Verträge. Der
Live-Katalog von ChatGPT/Codex stellt üblicherweise ein Modellfenster von `272000` Token bereit,
während OpenAI für GPT-5.5 und GPT-5.6 ein Platform-API-Fenster von `1050000` Token und
eine maximale Ausgabe von `128000` dokumentiert. Wird die vollständige Ausgabekapazität
reserviert, verbleibt ein daraus abgeleitetes Eingabebudget von `922000` Token. Für Anfragen mit
mehr als `272000` Eingabe-Token gilt die höhere OpenAI-Preisgestaltung für lange Kontexte.

Beginnen Sie mit einem vollständigen Codex-Modellkatalog, der mit der installierten Codex-Version
kompatibel ist. Behalten Sie für jeden direkten GPT-5.5- oder GPT-5.6-Eintrag, der einen langen Kontext
verwenden soll, den restlichen Deskriptor bei und legen Sie Folgendes fest:

```json
{
  "context_window": 922000,
  "max_context_window": 922000,
  "auto_compact_token_limit": 700000
}
```

Codex wendet seine normale Reserve von 95 % für das effektive Fenster auf den Katalogwert
`922000` an und meldet daher etwa `875900` nutzbare Token. Eine Compaction bei
`700000` lässt `175900` Token bis zu dieser effektiven Schutzgrenze und
`222000` bis zur für den Provider sicheren Eingabekapazität. Dieser größere Puffer ist
beabsichtigt: Codex prüft den bereits aufgezeichneten Kontext, bevor die nächste Benutzernachricht und
Kontextaktualisierungen hinzugefügt werden. Daher muss der Schwellenwert sowohl einen großen eingehenden
Turn als auch Tools, Anweisungen, Serialisierung und den Compaction-Turn selbst abdecken.

Für die eigenständige Verwendung der Codex CLI oder Desktop-Anwendung kann ein benutzerdefinierter
Provider mit Befehlsauthentifizierung den API-Schlüssel aus einem Systemschlüsselbund oder
Secret-Manager lesen, während die normale ChatGPT-Anmeldung weiterhin für Konnektoren verfügbar bleibt:

```toml
model = "gpt-5.6-terra"
model_provider = "openai_api_direct"
model_context_window = 922000
model_auto_compact_token_limit = 700000
model_auto_compact_token_limit_scope = "total"
model_catalog_json = "/absolute/path/to/models-api-1m.json"

[model_providers.openai_api_direct]
name = "OpenAI API direct"
base_url = "https://api.openai.com/v1"
wire_api = "responses"
requires_openai_auth = false

[model_providers.openai_api_direct.auth]
command = "/absolute/path/to/read-openai-inference-key"
timeout_ms = 5000
refresh_interval_ms = 300000
```

Das Authentifizierungshilfsprogramm darf ausschließlich den Schlüssel auf stdout ausgeben. Tragen Sie
ihn nicht in TOML ein.

Behalten Sie für das OpenClaw-Codex-App-Server-Harness das standardmäßige agentenbezogene Codex-Home
bei und lassen Sie OpenClaw ein API-Schlüsselprofil `openai` einschleusen. Übergeben Sie den
Katalog und die Kontextgrenzen als native Codex-App-Server-Argumente:

```json5
{
  auth: {
    order: {
      openai: ["openai:api-key"],
    },
  },
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          appServer: {
            args: [
              "app-server",
              "--listen",
              "stdio://",
              "-c",
              'model_catalog_json="/absolute/path/to/models-api-1m.json"',
              "-c",
              "model_context_window=922000",
              "-c",
              "model_auto_compact_token_limit=700000",
              "-c",
              "model_auto_compact_token_limit_scope=total",
            ],
          },
        },
      },
    },
  },
  agents: {
    defaults: {
      model: "openai/gpt-5.6-terra",
      models: {
        "openai/gpt-5.6-terra": { agentRuntime: { id: "codex" } },
      },
    },
  },
}
```

Ersetzen Sie `openai:api-key` bei Bedarf durch die tatsächliche ID des API-Schlüsselprofils. Der
agentenbezogene App-Server erhält nur diesen vorbereiteten Schlüssel; die native
ChatGPT-Anmeldung `~/.codex` des Betreibers, Plugins, Konnektoren und der Thread-Speicher bleiben
unverändert. Der Codex-App-Server `0.144.6` fügt den Bearer eines benutzerdefinierten Providers
mit Befehlsauthentifizierung nicht an App-Server-Turns an. Verwenden Sie für diese Route daher den oben
beschriebenen eingeschleusten API-Schlüsselpfad anstelle von `homeScope: "user"`.

Starten Sie nach Änderungen am Katalog oder an den App-Server-Argumenten den Gateway neu und beginnen
Sie einen neuen Chat. Vorhandene native Threads behalten ihre aufgezeichneten Provider- und
Modelleinstellungen bei. Überprüfen Sie die Laufzeit mit `/status` und `/codex status` und
senden Sie anschließend einen harmlosen direkten API-Turn, bevor Sie eine lange Sitzung beginnen.

<Warning>
Lange Kontexte müssen bewusst aktiviert werden. OpenAI berechnet die gesamte Anfrage mit dem
2-Fachen Eingabe- und dem 1,5-Fachen Ausgabetarif, sobald die Eingabe `272000` Token
überschreitet. Die API ist weiterhin die maßgebliche Instanz für Zugriff, tatsächliche Grenzwerte und
Abrechnung. Siehe [OpenAI-Modellgrenzen](https://developers.openai.com/api/docs/models/compare) und
[API-Preisgestaltung](https://developers.openai.com/api/docs/pricing).
</Warning>

Der Rest dieser Seite behandelt die Bereitstellungsstruktur, Fail-Closed-Routing, die
Guardian-Genehmigungsrichtlinie, native Codex-Plugins und Computer Use. Vollständige Listen der Optionen,
Standardwerte, Enums, Erkennung, Umgebungsisolierung, Zeitüberschreitungen und
App-Server-Transportfelder finden Sie in der
[Codex-Harness-Referenz](/de/plugins/codex-harness-reference).

## Codex-Laufzeit überprüfen

Verwenden Sie `/status` in dem Chat, in dem Sie Codex erwarten. Ein von Codex gestützter
OpenAI-Agenten-Turn zeigt:

```text
Laufzeit: OpenAI Codex
```

Prüfen Sie anschließend den Zustand des Codex-App-Servers:

```text
/codex status
/codex models
/codex binding
```

`/codex binding` meldet den verknüpften nativen Thread und die aktuellen Modelleinstellungen.
`/codex status` meldet App-Server-Konnektivität, Konto, Ratenbegrenzungen, MCP-Server und Skills.
`/codex models` listet den Live-Katalog des Codex-App-Servers für das Harness und das Konto auf.
Falls `/status` unerwartet ist, lesen Sie die
[Fehlerbehebung](#troubleshooting).

## Routing und Modellauswahl

Halten Sie Provider-Referenzen und Laufzeitrichtlinien getrennt:

- Verwenden Sie `openai/gpt-*` für die kanonische OpenAI-Modellauswahl. Das Präfix allein
  wählt niemals Codex aus.
- Wenn keine Laufzeit oder `auto` festgelegt ist, kann nur eine exakte offizielle
  HTTPS-Route für Platform Responses oder ChatGPT Responses ohne selbst definierte Anfrageüberschreibung
  Codex implizit auswählen.
- Verwenden Sie keine veralteten Codex-GPT-Referenzen in der Konfiguration; führen Sie
  `openclaw doctor --fix` aus, um veraltete Referenzen und überholte Sitzungs-Routenfixierungen zu reparieren.
- `agentRuntime.id: "codex"` macht Codex für eine kompatible Route zu einer Fail-Closed-Anforderung.
  Dadurch wird eine inkompatible effektive Route nicht kompatibel.
- `agentRuntime.id: "openclaw"` aktiviert für einen Provider oder ein Modell bewusst die eingebettete
  OpenClaw-Laufzeit.
- `/codex ...` steuert native Unterhaltungen des Codex-App-Servers aus dem Chat.
- ACP/acpx ist ein separater externer Harness-Pfad. Verwenden Sie ihn nur, wenn der Benutzer
  ACP/acpx oder einen externen Harness-Adapter anfordert.

| Benutzerabsicht                                            | Verwendung                                                                                            |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Aktuellen Chat verknüpfen                                  | `/codex bind [thread-id] [--cwd <path>] [--model <model>] [--provider <provider>]`                    |
| Vorhandenen Codex-Thread fortsetzen                        | `/codex resume <thread-id>`                                                                           |
| Codex-Threads auflisten oder filtern                       | `/codex threads [filter]`                                                                             |
| Natives Ziel des verknüpften Threads lesen oder aktualisieren | `/codex goal [status\|set <objective>\|pause\|resume\|block\|complete\|clear]`                        |
| Native Codex-Plugins auflisten                             | `/codex plugins list`                                                                                 |
| Konfiguriertes natives Codex-Plugin aktivieren oder deaktivieren | `/codex plugins enable <name>`, `/codex plugins disable <name>`                                       |
| Gespeicherte Codex-CLI-Sitzung als Paired-Node-Turn fortsetzen | `/codex sessions --host <node> [filter]`, dann `/codex resume <session-id> --host <node> --bind here` |
| Nicht archivierte Codex-Sitzungen computerübergreifend anzeigen | Codex-Überwachung aktivieren und **Codex-Sitzungen** öffnen                                           |
| Modell, Schnellmodus oder Berechtigungen des verknüpften Threads ändern | `/codex model <model>`, `/codex fast [on\|off\|status]`, `/codex permissions [default\|yolo\|status]` |
| Aktiven Turn stoppen oder steuern                          | `/codex stop`, `/codex steer <text>`                                                                  |
| Aktuelle Verknüpfung lösen                                 | `/codex detach` (Alias `/codex unbind`)                                                               |
| Nur Codex-Feedback senden                                  | `/codex diagnostics [note]`                                                                           |
| ACP/acpx-Aufgabe starten                                   | ACP/acpx-Sitzungsbefehle, nicht `/codex`                                                               |

| Anwendungsfall                                  | Konfiguration                                                                                               | Überprüfung                              | Hinweise                                   |
| ----------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ---------------------------------------- | ------------------------------------------ |
| Geeignete OpenAI-Route mit nativer Codex-Laufzeit | Exakte offizielle HTTPS-Responses-/ChatGPT-Route ohne selbst definierte Anfrageüberschreibung sowie aktiviertes Plugin `codex` | `/status` zeigt `Runtime: OpenAI Codex` | Impliziter Pfad, wenn keine Laufzeit/`auto` festgelegt ist |
| Fail-Closed, wenn Codex nicht verfügbar ist     | Provider oder Modell `agentRuntime.id: "codex"`                                                                     | Turn schlägt statt eines eingebetteten Fallbacks fehl | Für reine Codex-Bereitstellungen verwenden |
| Direkter OpenAI-API-Schlüssel-Datenverkehr über OpenClaw | Provider oder Modell `agentRuntime.id: "openclaw"` und normale OpenAI-Authentifizierung                                | `/status` zeigt die OpenClaw-Laufzeit | Nur verwenden, wenn OpenClaw beabsichtigt ist |
| Veraltete Konfiguration                         | veraltete Codex-GPT-Referenzen                                                                               | `openclaw doctor --fix` schreibt sie um       | Neue Konfiguration nicht auf diese Weise erstellen |
| ACP/acpx-Codex-Adapter                          | ACP `sessions_spawn({ runtime: "acp" })`                                                                                       | ACP-Aufgaben-/Sitzungsstatus             | Vom nativen Codex-Harness getrennt         |

`agents.defaults.imageModel` folgt derselben Präfixaufteilung. Verwenden Sie `openai/gpt-*`
für die normale OpenAI-Route und `codex/gpt-*` nur, wenn das Bildverständnis
über einen begrenzten Codex-App-Server-Turn erfolgen soll. Doctor schreibt veraltete
Codex-GPT-Referenzen in `openai/gpt-*` um.

## Bereitstellungsmuster

### Einfache Codex-Bereitstellung

Verwenden Sie die Schnellstartkonfiguration für ein OpenAI-Modell, dessen effektive offizielle
HTTPS-Route geeignet ist, Codex implizit auszuwählen:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
      },
    },
  },
  agents: {
    defaults: {
      model: "openai/gpt-5.6-sol",
    },
  },
}
```

### Bereitstellung mit mehreren Providern

Behalten Sie Claude als Standardagenten bei und fügen Sie einen benannten Codex-Agenten hinzu:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
      },
    },
  },
  agents: {
    defaults: {
      model: "anthropic/claude-opus-4-6",
    },
    list: [
      {
        id: "main",
        default: true,
        model: "anthropic/claude-opus-4-6",
      },
      {
        id: "codex",
        name: "Codex",
        model: "openai/gpt-5.6-sol",
      },
    ],
  },
}
```

Der Agent `main` verwendet seinen normalen Provider-Pfad. Der Agent
`codex` verwendet den Codex-App-Server, solange seine effektive OpenAI-Route kompatibel bleibt;
fügen Sie explizit modellbezogen `agentRuntime.id: "codex"` hinzu, wenn dies eine Fail-Closed-Anforderung sein soll.

### Fail-Closed-Codex-Bereitstellung

Eine geeignete, exakt offizielle HTTPS-OpenAI-Route kann zu Codex aufgelöst werden, wenn das
gebündelte Plugin verfügbar ist. Fügen Sie eine explizite Laufzeitrichtlinie für eine festgelegte
Fail-Closed-Regel hinzu:

```json5
{
  models: {
    providers: {
      openai: {
        agentRuntime: {
          id: "codex",
        },
      },
    },
  },
  agents: {
    defaults: {
      model: "openai/gpt-5.6-sol",
    },
  },
  plugins: {
    entries: {
      codex: {
        enabled: true,
      },
    },
  },
}
```

Wenn Codex erzwungen wird, bricht OpenClaw frühzeitig ab, falls die effektive Route nicht als
Codex-kompatibel deklariert ist, das Plugin deaktiviert ist, der App-Server zu alt ist oder der
App-Server nicht gestartet werden kann.

## App-Server-Richtlinie

Standardmäßig startet das Plugin die von OpenClaw verwaltete Codex-Binärdatei lokal mit
stdio-Transport. Setzen Sie `appServer.command` nur, wenn absichtlich eine
andere ausführbare Datei verwendet werden soll. Codex stuft den WebSocket-Transport als experimentell
und nicht unterstützt ein; verwenden Sie ihn nur für Tests außerhalb der Produktion mit einem App-Server,
der bereits an anderer Stelle ausgeführt wird:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          appServer: {
            transport: "websocket",
            url: "ws://gateway-host:39175",
            authToken: "${CODEX_APP_SERVER_TOKEN}",
          },
        },
      },
    },
  },
}
```

Lokale stdio-App-Server-Sitzungen verwenden standardmäßig die vertrauenswürdige Haltung für lokale Bediener:
`approvalPolicy: "never"`, `approvalsReviewer: "user"` und
`sandbox: "danger-full-access"`. Wenn lokale Codex-Anforderungen diese
implizite YOLO-Haltung nicht zulassen, wählt OpenClaw stattdessen zulässige Guardian-Berechtigungen
aus. Wenn für die Sitzung eine OpenClaw-Sandbox aktiv ist, deaktiviert OpenClaw
für diesen Turn den nativen Code Mode von Codex, benutzerdefinierte MCP-Server und die Ausführung
App-gestützter Plugins, anstatt sich auf das hostseitige Sandboxing von Codex zu verlassen.
Der Shell-Zugriff erfolgt stattdessen über dynamische, durch die OpenClaw-Sandbox abgesicherte Tools wie
`sandbox_exec` und `sandbox_process`, sofern die normalen exec-/process-Tools
verfügbar sind.

Verwenden Sie für die native automatische Überprüfung von Codex den normalisierten exec-Modus von OpenClaw,
bevor Sie Sandbox-Ausnahmen oder zusätzliche Berechtigungen einsetzen:

```json5
{
  tools: {
    exec: {
      mode: "auto",
    },
  },
  plugins: {
    entries: {
      codex: {
        enabled: true,
      },
    },
  },
}
```

Bei Codex-App-Server-Sitzungen wird `tools.exec.mode: "auto"` auf von Codex
Guardian geprüfte Genehmigungen abgebildet: üblicherweise `approvalPolicy: "on-request"`,
`approvalsReviewer: "auto_review"` und `sandbox: "workspace-write"`, sofern
die lokalen Anforderungen diese Werte zulassen. In `tools.exec.mode: "auto"`
behält OpenClaw veraltete unsichere Codex-Überschreibungen für `approvalPolicy: "never"` oder
`sandbox: "danger-full-access"` nicht bei; verwenden Sie `tools.exec.mode: "full"` für
eine beabsichtigte Codex-Haltung ohne Genehmigungen. Die veraltete Voreinstellung
`plugins.entries.codex.config.appServer.mode: "guardian"` funktioniert weiterhin,
aber `tools.exec.mode: "auto"` ist die normalisierte OpenClaw-Oberfläche.

Den Vergleich auf Modusebene mit hostseitigen exec-Genehmigungen und ACPX-
Berechtigungen finden Sie unter [Berechtigungsmodi](/de/tools/permission-modes). Einzelheiten zu allen
App-Server-Feldern, zur Authentifizierungsreihenfolge, zur Umgebungsisolierung und zum Timeout-Verhalten
finden Sie in der [Codex-Harness-Referenz](/de/plugins/codex-harness-reference).

## Befehle und Diagnose

Das Plugin `codex` registriert `/codex` als Slash-Befehl auf jedem Kanal, der
OpenClaw-Textbefehle unterstützt.

Native Ausführung und Steuerung erfordern einen Eigentümer oder einen `operator.admin`-
Gateway-Client: Threads binden oder fortsetzen, Turns senden oder stoppen,
Modell, Schnellmodus oder Berechtigungsstatus ändern, Compaction oder Überprüfungen ausführen und
eine Bindung lösen. Andere autorisierte Absender behalten schreibgeschützte Befehle zur Status-, Hilfe-,
Konto-, Modell-, Thread-, nativen Ziel-, MCP-Server-, Skill- und Bindungsprüfung.

Häufige Formen:

- `/codex status` prüft App-Server-Konnektivität, Modelle, Konto, Ratenbegrenzungen,
  MCP-Server und Skills.
- `/codex models` listet aktive Codex-App-Server-Modelle auf.
- `/codex threads [filter]` listet kürzlich verwendete Codex-App-Server-Threads auf.
- `/codex goal` liest oder aktualisiert das native Codex-Ziel des angehängten Threads. Die automatische Zielfortsetzung durch Codex bleibt deaktiviert; OpenClaw übernimmt noch keine autonomen Folge-Turns.
- `/codex resume <thread-id>` hängt die aktuelle OpenClaw-Sitzung an einen
  bestehenden Codex-Thread an.
- `/codex bind [thread-id] [--cwd <path>] [--model <model>] [--provider <provider>]`
  hängt den aktuellen Chat an.
- `/codex detach` (oder `/codex unbind`) löst die aktuelle Bindung.
- `/codex binding` beschreibt die aktuelle Bindung.
- `/codex stop` stoppt den aktiven Turn; `/codex steer <text>` steuert ihn.
- `/codex model <model>`, `/codex fast [on|off|status]` und
  `/codex permissions [default|yolo|status]` ändern den Zustand pro Unterhaltung.
- `/codex compact` weist den Codex-App-Server an, für den angehängten Thread eine Compaction auszuführen.
- `/codex review` startet eine native Codex-Überprüfung für den angehängten Thread.
- `/codex diagnostics [note]` fragt vor dem Senden von Codex-Feedback für den
  angehängten Thread nach.
- `/codex account` zeigt Konto- und Ratenbegrenzungsstatus an.
- `/codex mcp` listet den Status der MCP-Server des Codex-App-Servers auf.
- `/codex skills` listet die Skills des Codex-App-Servers auf.
- `/codex plugins list`, `/codex plugins enable <name>` und
  `/codex plugins disable <name>` verwalten konfigurierte native Codex-Plugins.
- `/codex computer-use [status|install]` verwaltet Codex Computer Use.
- `/codex help` listet den vollständigen Befehlsbaum auf.

Beginnen Sie bei den meisten Supportmeldungen mit `/diagnostics [note]` in der
Unterhaltung, in der der Fehler aufgetreten ist. Dadurch wird ein Gateway-Diagnosebericht
erstellt und bei Codex-Harness-Sitzungen um Genehmigung zum Senden des
relevanten Codex-Feedback-Pakets gebeten. Informationen zum Datenschutzmodell und zum Verhalten in
Gruppenchats finden Sie unter [Diagnoseexport](/de/gateway/diagnostics).
Verwenden Sie `/codex diagnostics [note]` nur, wenn Sie ausdrücklich
das Codex-Feedback für den derzeit angehängten Thread ohne das
vollständige Gateway-Diagnosepaket hochladen möchten.

### Codex-Threads lokal untersuchen

Die schnellste Möglichkeit, einen fehlerhaften Codex-Lauf zu untersuchen, besteht häufig darin, den nativen
Codex-Thread direkt zu öffnen:

```bash
codex resume <thread-id>
```

Die Thread-ID erhalten Sie aus der abgeschlossenen Antwort von `/diagnostics`, aus `/codex binding`
oder aus `/codex threads [filter]`.

Informationen zum Upload-Mechanismus und zu Diagnosegrenzen auf Runtime-Ebene finden Sie unter
[Codex-Harness-Runtime](/de/plugins/codex-harness-runtime#codex-feedback-upload).

### Authentifizierungsreihenfolge

Im standardmäßigen agentenspezifischen Home-Verzeichnis wird die Authentifizierung in dieser Reihenfolge ausgewählt:

1. Geordnete OpenAI-Authentifizierungsprofile für den Agenten, vorzugsweise unter
   `auth.order.openai`. Führen Sie `openclaw doctor --fix` aus, um ältere veraltete
   Codex-Authentifizierungsprofil-IDs und die veraltete Codex-Authentifizierungsreihenfolge zu migrieren.
2. Das vorhandene Konto des App-Servers im Codex-Home-Verzeichnis dieses Agenten.
3. Nur für lokale stdio-App-Server-Starts: `CODEX_API_KEY`, danach
   `OPENAI_API_KEY`, wenn kein App-Server-Konto vorhanden ist und weiterhin eine OpenAI-Authentifizierung
   erforderlich ist.

Wenn OpenClaw ein Codex-Authentifizierungsprofil im Stil eines ChatGPT-Abonnements erkennt,
entfernt es `CODEX_API_KEY` und `OPENAI_API_KEY` aus dem erzeugten untergeordneten Codex-
Prozess. Dadurch bleiben API-Schlüssel auf Gateway-Ebene für Embeddings oder
direkte OpenAI-Modelle verfügbar, ohne dass native Turns des Codex-App-Servers
versehentlich über die API abgerechnet werden. Explizite Codex-API-Schlüsselprofile und die lokale
stdio-Rückfalloption für Umgebungsschlüssel verwenden die App-Server-Anmeldung anstelle einer geerbten
Umgebung des untergeordneten Prozesses. WebSocket-App-Server-Verbindungen erhalten keine Gateway-
Rückfalloption für API-Schlüssel aus der Umgebung; verwenden Sie ein explizites Authentifizierungsprofil oder das eigene
Konto des entfernten App-Servers.

Wenn ein Abonnementprofil ein Codex-Nutzungslimit erreicht, zeichnet OpenClaw die
Rücksetzzeit auf, sofern Codex eine meldet, und versucht für denselben Codex-Lauf das nächste geordnete
Authentifizierungsprofil. Nach Ablauf der Rücksetzzeit ist das Abonnementprofil
wieder verfügbar, ohne dass das ausgewählte Modell `openai/gpt-*`
oder die Codex-Runtime geändert wird.

Wenn native Codex-Plugins konfiguriert sind, installiert oder aktualisiert OpenClaw
diese Plugins über den verbundenen App-Server, bevor Plugin-eigene
Apps für den Codex-Thread verfügbar gemacht werden. `app/list` bleibt die maßgebliche Quelle für App-
IDs, Zugänglichkeit und Metadaten, doch OpenClaw ist für die Aktivierungsentscheidung
pro Thread zuständig: Wenn die Richtlinie eine aufgeführte zugängliche App zulässt, sendet OpenClaw
`thread/start.config.apps[appId].enabled = true`, selbst wenn `app/list`
diese App derzeit als deaktiviert meldet. Dieser Pfad erfindet keine App-
Installation für unbekannte IDs; OpenClaw aktiviert nur Marketplace-Plugins
mit `plugin/install` und aktualisiert anschließend das Inventar.

### Umgebungsisolierung

Bei lokalen stdio-App-Server-Starts setzt OpenClaw `CODEX_HOME` auf ein
agentenspezifisches Verzeichnis, sodass Codex-Konfiguration, Authentifizierungs-/Kontodateien, Plugin-Cache/-Daten
und nativer Thread-Zustand standardmäßig nicht das persönliche
`~/.codex` des Bedieners lesen oder beschreiben. OpenClaw behält das normale Prozess-`HOME` bei;
von Codex ausgeführte Unterprozesse können daher weiterhin Konfigurationen und Tokens im Benutzer-Home finden, und
Codex kann gemeinsam genutzte Einträge in `$HOME/.agents/skills` und
`$HOME/.agents/plugins/marketplace.json` erkennen. Mit
`appServer.homeScope: "user"` verwendet OpenClaw stattdessen das native Codex-
Home des Benutzers und dessen bestehendes Konto, ohne ein OpenClaw-Authentifizierungsprofil einzuschleusen.

Wenn eine Bereitstellung zusätzliche Umgebungsisolierung benötigt, fügen Sie diese
Variablen zu `appServer.clearEnv` hinzu:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          appServer: {
            clearEnv: ["CODEX_API_KEY", "OPENAI_API_KEY"],
          },
        },
      },
    },
  },
}
```

`appServer.clearEnv` wirkt sich nur auf den erzeugten untergeordneten Codex-App-Server-
Prozess aus. OpenClaw entfernt `CODEX_HOME` und `HOME` während
der Normalisierung des lokalen Starts aus dieser Liste: `CODEX_HOME` verweist weiterhin auf den ausgewählten
Agenten- oder Benutzerbereich, und `HOME` wird weiterhin geerbt, damit Unterprozesse den
normalen Zustand im Benutzer-Home verwenden können.

### Dynamische Tools und Websuche

Dynamische Codex-Tools verwenden standardmäßig das Laden über `searchable`. OpenClaw stellt normalerweise
keine dynamischen Tools bereit, die native Codex-Arbeitsbereichsoperationen duplizieren:
`read`, `write`, `edit`, `apply_patch`, `exec`, `process`, `update_plan`,
`get_goal`, `create_goal`, `update_goal`, `tool_call`, `tool_describe`,
`tool_search` und `tool_search_code`. Zieloperationen bleiben nativ in Codex,
daher projiziert OpenClaw keinen zweiten Zielspeicher in Codex-Turns. Die meisten
übrigen OpenClaw-Integrationstools, etwa für Nachrichten, Medien, Cron,
Browser, Nodes, Gateway und `heartbeat_respond`, sind über
die Codex-Tool-Suche im Namespace `openclaw` verfügbar, wodurch der anfängliche Modellkontext
kleiner bleibt. Die Shell-Rückfalloption für eingeschränkte Turns bildet eine Ausnahme für
`exec` und `process`, wenn eine endliche Positivliste den nativen Code Mode deaktiviert;
Runtime-Positivlisten und `codexDynamicToolsExclude` gelten weiterhin.

Mit `catalogMode: "direct-only"` gekennzeichnete Tools, einschließlich des OpenClaw-Tools `computer`,
verwenden stattdessen den Namespace `openclaw_direct`. Codex behandelt diesen Namespace
als `DirectModelOnly`, sodass diese Tools in normalen und auf den Code Mode beschränkten
Threads direkt für das Modell sichtbar bleiben, statt verschachtelte Code-Mode-Aufrufe von `tools.*` zu durchlaufen.

Die Websuche verwendet standardmäßig das gehostete Codex-Tool `web_search`, wenn die Suche
aktiviert und kein verwalteter Provider ausgewählt ist. Die native gehostete Suche und
das verwaltete dynamische OpenClaw-Tool `web_search` schließen sich gegenseitig aus, damit
die verwaltete Suche native Domainbeschränkungen nicht umgehen kann. OpenClaw verwendet das
verwaltete Tool, wenn die gehostete Suche nicht verfügbar, ausdrücklich deaktiviert oder
durch einen ausgewählten verwalteten Provider ersetzt ist. OpenClaw lässt die eigenständige
Codex-Erweiterung `web.run` deaktiviert, da der Produktionsdatenverkehr des App-Servers
deren benutzerdefinierten Namespace `web` ablehnt. `tools.web.search.enabled: false`
deaktiviert beide Pfade, ebenso wie Tool-deaktivierte reine LLM-Läufe. Codex behandelt
`"cached"` als Präferenz und löst sie bei uneingeschränkten App-Server-Turns in
aktiven externen Zugriff auf. Die automatische verwaltete Rückfalloption schlägt geschlossen fehl, wenn
native `allowedDomains` gesetzt sind, damit die Positivliste nicht umgangen werden kann.
Dauerhafte Änderungen der effektiven Suchrichtlinie rotieren den gebundenen Codex-Thread
vor dem nächsten Turn; vorübergehende Einschränkungen pro Turn verwenden einen temporären
eingeschränkten Thread und bewahren die bestehende Bindung für eine spätere Fortsetzung auf.

`sessions_yield`, `sessions_spawn` und reine Quellantworten des Nachrichten-Tools bleiben
direkt, da sie Verträge zur Ablaufsteuerung oder Delegation darstellen. Die Anleitung
bevorzugt weiterhin Codex' natives `spawn_agent` als primäre Codex-Subagent-Oberfläche,
während eine explizite Delegation über OpenClaw oder ACP weiterhin direkt über
`sessions_spawn` aufgerufen werden kann. Im Codex Code Mode sind generische Ergebnisse
dynamischer OpenClaw-Tools JSON-Text statt JavaScript-Objekte; parsen Sie daher
JSON-ähnliche Ergebnisse, bevor Sie Felder auslesen. Codex serialisiert außerdem verschachtelte
dynamische Aufrufe; übermitteln Sie mehrere `sessions_spawn`-Aufrufe in einer begrenzten Schleife,
statt zu erwarten, dass `Promise.all` sie gleichzeitig startet. Bereits angenommene
untergeordnete Prozesse können sich dennoch überschneiden, während spätere Aufrufe übermittelt werden. Ein vollständiges Muster finden Sie unter
[Swarm](/de/tools/swarm#use-swarm-from-other-harnesses).
Anweisungen zur Heartbeat-Zusammenarbeit
weisen Codex an, vor dem Beenden eines Heartbeat-Durchlaufs nach `heartbeat_respond` zu suchen,
wenn das Tool noch nicht geladen ist.

Legen Sie `codexDynamicToolsLoading: "direct"` nur fest, wenn Sie eine Verbindung zu einem benutzerdefinierten
Codex-App-Server herstellen, der nicht nach zurückgestellten dynamischen Tools suchen kann, oder wenn Sie
die vollständige Tool-Nutzlast debuggen.

### Konfigurationsfelder

Unterstützte Codex-Plugin-Felder auf oberster Ebene:

| Feld                       | Standardwert   | Bedeutung                                                                                |
| -------------------------- | -------------- | ---------------------------------------------------------------------------------------- |
| `codexDynamicToolsLoading` | `"searchable"` | Verwenden Sie `"direct"`, um dynamische OpenClaw-Tools direkt in den anfänglichen Codex-Tool-Kontext aufzunehmen. |
| `codexDynamicToolsExclude` | `[]`           | Zusätzliche Namen dynamischer OpenClaw-Tools, die in Codex-App-Server-Durchläufen ausgelassen werden sollen. |
| `codexPlugins`             | deaktiviert    | Native Codex-Plugin-/App-Unterstützung für migrierte, aus dem Quellcode installierte kuratierte Plugins. |
| `sessionCatalog`           | aktiviert      | Erkennung in der Seitenleiste für native Codex-Sitzungen auf diesem Gateway und auf geeigneten gekoppelten Nodes. |
| `supervision`              | deaktiviert    | Agentenseitiges Transkript nativer Sitzungen und Richtlinie zur Schreibsteuerung.         |

Unterstützte `appServer`-Felder:

| Feld                                          | Standard                                                | Bedeutung                                                                                                                                                                                                                                                                                                                                                                                         |
| --------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `transport`                                   | `"stdio"`                                              | `"stdio"` startet Codex; ein explizites `"unix"` stellt eine Verbindung zum lokalen Steuerungs-Socket her; `"websocket"` stellt eine Verbindung zu `url` her.                                                                                                                                                                                                                                                                                |
| `homeScope`                                   | `"agent"`                                              | `"agent"` isoliert den gewöhnlichen Harness-Zustand pro OpenClaw-Agent. `"user"` ist eine explizite Aktivierung, die das native `$CODEX_HOME` oder `~/.codex` gemeinsam nutzt, die native Authentifizierung verwendet und die Thread-Verwaltung ausschließlich durch den Eigentümer ermöglicht. Der Benutzerbereich unterstützt lokales stdio oder Unix-Transport. Für die separate Überwachungsverbindung wird ein nicht gesetzter Wert für stdio oder Unix zu `"user"` und für WebSocket zu `"agent"` aufgelöst.     |
| `command`                                     | verwaltete Codex-Binärdatei                                   | Ausführbare Datei für den stdio-Transport. Lassen Sie die Einstellung leer, um die verwaltete Binärdatei zu verwenden; legen Sie sie nur für eine explizite Überschreibung fest.                                                                                                                                                                                                                                                                                    |
| `args`                                        | `["app-server", "--listen", "stdio://"]`               | Argumente für den stdio-Transport.                                                                                                                                                                                                                                                                                                                                                                  |
| `url`                                         | nicht gesetzt                                                  | WebSocket-App-Server-URL oder `unix://`-URL. Ein explizit leerer Unix-Pfad wählt den kanonischen Steuerungs-Socket im Benutzerverzeichnis aus.                                                                                                                                                                                                                                                                          |
| `authToken`                                   | nicht gesetzt                                                  | Bearer-Token für den WebSocket-Transport. Akzeptiert eine Literalzeichenfolge oder SecretInput wie `${CODEX_APP_SERVER_TOKEN}`.                                                                                                                                                                                                                                                                              |
| `headers`                                     | `{}`                                                   | Zusätzliche WebSocket-Header. Headerwerte akzeptieren Literalzeichenfolgen oder SecretInput-Werte, beispielsweise `x-codex-client-session-token: "${CODEX_CLIENT_SESSION_TOKEN}"`.                                                                                                                                                                                                                               |
| `clearEnv`                                    | `[]`                                                   | Namen zusätzlicher Umgebungsvariablen, die aus dem gestarteten stdio-App-Server-Prozess entfernt werden, nachdem OpenClaw dessen geerbte Umgebung erstellt hat. OpenClaw behält die ausgewählte Variable `CODEX_HOME` und die geerbte Variable `HOME` für lokale Starts bei.                                                                                                                                                                           |
| `codeModeOnly`                                | `false`                                                | Aktiviert ausschließlich die Codex-Tool-Oberfläche für den Code-Modus. Gewöhnliche dynamische OpenClaw-Tools bleiben über verschachtelte `tools.*`-Aufrufe verfügbar; `openclaw_direct`-Tools bleiben für das Modell direkt sichtbar.                                                                                                                                                                                                             |
| `remoteWorkspaceRoot`                         | nicht gesetzt                                                  | Stammverzeichnis des Remote-Codex-App-Server-Arbeitsbereichs. Wenn es festgelegt ist, leitet OpenClaw das lokale Arbeitsbereichs-Stammverzeichnis aus dem aufgelösten OpenClaw-Arbeitsbereich ab, behält das Suffix des aktuellen Arbeitsverzeichnisses unter diesem Remote-Stammverzeichnis bei und sendet nur das endgültige App-Server-Arbeitsverzeichnis an Codex. Befindet sich das Arbeitsverzeichnis außerhalb des aufgelösten OpenClaw-Arbeitsbereichs-Stammverzeichnisses, bricht OpenClaw sicher ab, anstatt einen Gateway-lokalen Pfad an den Remote-App-Server zu senden. |
| `requestTimeoutMs`                            | `60000`                                                | Zeitüberschreitung für Aufrufe der App-Server-Steuerungsebene.                                                                                                                                                                                                                                                                                                                                                     |
| `turnCompletionIdleTimeoutMs`                 | `60000`                                                | Ruhefenster, nachdem Codex einen Turn akzeptiert hat oder nach einer Turn-bezogenen App-Server-Anfrage, während OpenClaw auf `turn/completed` wartet.                                                                                                                                                                                                                                                                    |
| `turnAssistantCompletionIdleTimeoutMs`        | `10000`                                                | Ruhefenster, nachdem ein endgültiges Assistant-Element beziehungsweise ein Assistant-Element ohne Kommentar oder ein Abschluss des unformatierten Assistant vor einem Tool die Freigabe der Assistant-Ausgabe aktiviert, während OpenClaw weiterhin auf `turn/completed` wartet. Eine Erhöhung gibt Codex mehr Zeit, `turn/completed` auszugeben, bevor OpenClaw unterbricht und die Sitzungsspur freigibt.                                                                                            |
| `postToolRawAssistantCompletionIdleTimeoutMs` | `300000`                                               | Abschlussleerlauf- und Fortschrittswächter, der nach einer Tool-Übergabe, dem Abschluss eines nativen Tools, einem Fortschritt des unformatierten Assistant nach einem Tool, dem Abschluss der unformatierten Schlussfolgerung oder dem Fortschritt der Schlussfolgerung verwendet wird, während OpenClaw auf `turn/completed` wartet. Verwenden Sie dies für vertrauenswürdige oder rechenintensive Arbeitslasten, bei denen die Synthese nach einem Tool berechtigterweise länger still bleiben kann als das Freigabebudget des endgültigen Assistant.                                |
| `mode`                                        | `"yolo"`, sofern lokale Codex-Anforderungen YOLO nicht verbieten | Voreinstellung für YOLO oder eine durch einen Guardian geprüfte Ausführung. Lokale stdio-Anforderungen, die `danger-full-access`, die Genehmigung `never` oder den Prüfer `user` auslassen, machen den Guardian zum impliziten Standard.                                                                                                                                                                                                           |
| `approvalPolicy`                              | `"never"` oder eine zulässige Guardian-Genehmigungsrichtlinie       | Native Codex-Genehmigungsrichtlinie, die beim Starten/Fortsetzen eines Threads beziehungsweise bei einem Turn gesendet wird. Guardian-Standards bevorzugen `"on-request"`, wenn dies zulässig ist.                                                                                                                                                                                                                                                                            |
| `sandbox`                                     | `"danger-full-access"` oder eine zulässige Guardian-Sandbox  | Nativer Codex-Sandbox-Modus, der beim Starten/Fortsetzen eines Threads gesendet wird. Guardian-Standards bevorzugen `"workspace-write"`, wenn dies zulässig ist, andernfalls `"read-only"`. Wenn eine OpenClaw-Sandbox aktiv ist, verwenden `danger-full-access`-Turns Codex `workspace-write` mit Netzwerkzugriff, der aus der Egress-Einstellung der OpenClaw-Sandbox abgeleitet wird.                                                                                     |
| `approvalsReviewer`                           | `"user"` oder ein zulässiger Guardian-Prüfer               | Verwenden Sie `"auto_review"`, damit Codex native Genehmigungsaufforderungen prüft, sofern dies zulässig ist, andernfalls `guardian_subagent` oder `user`. `guardian_subagent` bleibt ein veralteter Alias.                                                                                                                                                                                                                              |
| `serviceTier`                                 | nicht gesetzt                                                  | Optionale Dienstklasse des Codex-App-Servers. `"priority"` aktiviert das Fast-Mode-Routing, `"flex"` fordert eine flexible Verarbeitung an, `null` entfernt die Überschreibung und das veraltete `"fast"` wird als `"priority"` akzeptiert.                                                                                                                                                                                                 |
| `networkProxy`                                | deaktiviert                                               | Aktiviert das Netzwerk des Codex-Berechtigungsprofils für App-Server-Befehle. OpenClaw definiert die ausgewählte Konfiguration `permissions.<profile>.network` und wählt sie mit `default_permissions` aus, anstatt `sandbox` zu senden.                                                                                                                                                                             |
| `experimental.sandboxExecServer`              | `false`                                                | Vorschau-Aktivierung, die eine durch die OpenClaw-Sandbox gestützte Codex-Umgebung beim unterstützten Codex-App-Server registriert, sodass die native Codex-Ausführung innerhalb der aktiven OpenClaw-Sandbox ausgeführt werden kann.                                                                                                                                                                                                            |

`appServer.networkProxy` ist explizit, da es den Sandbox-Vertrag von Codex
ändert. Wenn diese Option aktiviert ist, setzt OpenClaw außerdem `features.network_proxy.enabled`
und `default_permissions` in der Codex-Thread-Konfiguration, damit das generierte
Berechtigungsprofil das von Codex verwaltete Netzwerk starten kann. Standardmäßig
generiert OpenClaw aus dem Profilinhalt einen kollisionsresistenten
`openclaw-network-<fingerprint>`-Profilnamen; verwenden Sie `profileName` nur, wenn ein
stabiler lokaler Name erforderlich ist.

```json5
{
  plugins: {
    entries: {
      codex: {
        config: {
          appServer: {
            sandbox: "workspace-write",
            networkProxy: {
              enabled: true,
              domains: {
                "api.openai.com": "allow",
                "blocked.example.com": "deny",
              },
              unixSockets: {
                "/tmp/proxy.sock": "allow",
                "/tmp/blocked.sock": "none",
              },
              allowUpstreamProxy: true,
              proxyUrl: "http://127.0.0.1:3128",
            },
          },
        },
      },
    },
  },
}
```

Wenn die normale App-Server-Laufzeit `danger-full-access` wäre, verwendet die
Aktivierung von `networkProxy` für das generierte Berechtigungsprofil einen
Dateisystemzugriff im Workspace-Stil: Die von Codex verwaltete
Netzwerkdurchsetzung ist Sandbox-Netzwerkzugriff, daher würde ein Profil mit
Vollzugriff ausgehenden Datenverkehr nicht schützen. Domäneneinträge verwenden
`allow` oder `deny`; Unix-Socket-Einträge verwenden die
Codex-Werte `allow` oder `none`.

### Dynamische Zeitüberschreitungen für Tool-Aufrufe

OpenClaw-eigene dynamische Tool-Aufrufe werden unabhängig von
`appServer.requestTimeoutMs` begrenzt: Codex-Anfragen vom Typ `item/tool/call`
verwenden standardmäßig einen OpenClaw-Watchdog von 90 Sekunden. Ein positiver
`timeoutMs`-Wert pro Aufruf verlängert oder verkürzt das jeweilige
Tool-Budget, begrenzt auf 600000 ms. Das Tool `image_generate` verwendet
`agents.defaults.mediaModels.image.timeoutMs`, wenn der Tool-Aufruf keine eigene Zeitüberschreitung
angibt, andernfalls gilt für die Bildgenerierung standardmäßig ein Wert von
120 Sekunden. Das Medienanalyse-Tool `image` verwendet
`timeoutSeconds` des ausgewählten bildfähigen Eintrags
`tools.media.models[]` oder seinen Medienstandardwert von 60 Sekunden; bei der
Bildanalyse gilt diese Zeitüberschreitung für die Anfrage selbst und wird
nicht durch vorherige Vorbereitungsarbeiten verkürzt. Bei einer
Zeitüberschreitung bricht OpenClaw das Tool-Signal ab, sofern dies unterstützt
wird, und gibt eine fehlgeschlagene dynamische Tool-Antwort an Codex zurück,
damit der Turn fortgesetzt werden kann, statt die Sitzung im Zustand
`processing` zu belassen. Dieser Watchdog ist das äußere dynamische
`item/tool/call`-Budget; providerspezifische Anfragezeitüberschreitungen
laufen innerhalb dieses Aufrufs und behalten ihre eigene
Zeitüberschreitungssemantik bei.

Nachdem Codex einen Turn akzeptiert hat und nachdem OpenClaw auf eine
turnbezogene App-Server-Anfrage geantwortet hat, erwartet der Harness, dass
Codex im aktuellen Turn Fortschritte erzielt und den nativen Turn schließlich
mit `turn/completed` abschließt. Wenn der App-Server für
`appServer.turnCompletionIdleTimeoutMs` inaktiv bleibt, unterbricht OpenClaw nach bestem Bemühen
den Codex-Turn, zeichnet eine diagnostische Zeitüberschreitung auf und gibt
die OpenClaw-Sitzungsspur frei, damit nachfolgende Chatnachrichten nicht hinter
einem veralteten nativen Turn in die Warteschlange eingereiht werden. Die
meisten nicht terminalen Benachrichtigungen für denselben Turn deaktivieren
diesen kurzen Watchdog, da Codex damit nachgewiesen hat, dass der Turn noch
aktiv ist.

Tool-Übergaben verwenden ein längeres Inaktivitätsbudget nach dem Tool: nachdem
OpenClaw eine `item/tool/call`-Antwort zurückgibt, nachdem native
Tool-Elemente wie `commandExecution` abgeschlossen sind, nach rohen
`custom_tool_call_output`-Abschlüssen sowie nach rohem Assistentenfortschritt,
rohen Reasoning-Abschlüssen oder Reasoning-Fortschritt nach einem Tool. Die
Schutzvorrichtung verwendet `appServer.postToolRawAssistantCompletionIdleTimeoutMs`, wenn dies konfiguriert ist,
und andernfalls standardmäßig fünf Minuten; dasselbe Budget verlängert auch
den Fortschritts-Watchdog für das stille Synthesefenster, bevor Codex das
nächste Ereignis des aktuellen Turns ausgibt. Globale
App-Server-Benachrichtigungen wie Aktualisierungen von Ratenbegrenzungen setzen
den Fortschritt für Turn-Inaktivität nicht zurück. Auf Reasoning-Abschlüsse,
Abschlüsse vom Typ `agentMessage` im Commentary-Kanal sowie rohen
Reasoning- oder Assistentenfortschritt vor einem Tool kann eine automatische
abschließende Antwort folgen; daher verwenden sie die Antwortschutzvorrichtung
nach Fortschritt, statt die Sitzungsspur sofort freizugeben.

Nur abgeschlossene finale, nicht dem Commentary-Kanal zugehörige
`agentMessage`-Elemente und rohe Assistentenabschlüsse vor einem Tool
aktivieren die Freigabe nach Assistentenausgabe: Wenn Codex anschließend ohne
`turn/completed` inaktiv bleibt, unterbricht OpenClaw nach bestem Bemühen
den nativen Turn und gibt die Sitzungsspur frei. Wenn eine andere
Turn-Überwachung dieses Freigaberennen gewinnt, akzeptiert OpenClaw das
abgeschlossene finale Assistentenelement dennoch, sobald keine native Anfrage,
kein Element und kein dynamischer Tool-Abschluss mehr aktiv ist, die Freigabe
nach Assistentenausgabe weiterhin zum zuletzt abgeschlossenen Element gehört
und kein späterer Elementabschluss vorliegt. Dadurch kann die endgültige
Antwort nach abgeschlossener Tool-Arbeit erhalten bleiben, ohne den Turn
erneut abzuspielen. Teilweise Assistenten-Deltas, veraltete frühere Antworten
und leere spätere Abschlüsse erfüllen die Voraussetzungen nicht.

Wiederholungssichere Fehler des stdio-App-Servers, einschließlich
Inaktivitätszeitüberschreitungen beim Turn-Abschluss ohne Hinweise auf
Assistentenaktivität, Tools, aktive Elemente oder Nebeneffekte, werden bei
einem neuen App-Server-Versuch einmal wiederholt. Unsichere
Zeitüberschreitungen setzen den blockierten App-Server-Client dennoch außer
Betrieb und geben die OpenClaw-Sitzungsspur frei; außerdem löschen sie die
veraltete native Thread-Bindung, statt automatisch erneut ausgeführt zu
werden. Zeitüberschreitungen der Abschlussüberwachung zeigen
Codex-spezifischen Zeitüberschreitungstext an: Bei wiederholungssicheren Fällen
wird darauf hingewiesen, dass die Antwort möglicherweise unvollständig ist,
während Nutzer bei unsicheren Fällen aufgefordert werden, vor einem erneuten
Versuch den aktuellen Zustand zu überprüfen. Öffentliche
Zeitüberschreitungsdiagnosen enthalten strukturelle Felder wie die Methode der
letzten App-Server-Benachrichtigung, ID/Typ/Rolle des rohen
Assistentenantwortelements, die Anzahl aktiver Anfragen und Elemente sowie den
aktivierten Überwachungsstatus; wenn die letzte Benachrichtigung ein rohes
Assistentenantwortelement ist, enthalten sie außerdem eine begrenzte Vorschau
des Assistententexts. Sie enthalten weder rohe Prompt- noch Tool-Inhalte.

### Lokale Überschreibungen durch Test-Umgebungsvariablen

- `OPENCLAW_CODEX_APP_SERVER_BIN` umgeht die verwaltete Binärdatei, wenn
  `appServer.command` nicht gesetzt ist.
- `OPENCLAW_CODEX_APP_SERVER_ARGS`
- `OPENCLAW_CODEX_APP_SERVER_MODE=yolo|guardian`
- `OPENCLAW_CODEX_APP_SERVER_APPROVAL_POLICY`
- `OPENCLAW_CODEX_APP_SERVER_SANDBOX`

`OPENCLAW_CODEX_APP_SERVER_GUARDIAN=1` wurde entfernt. Verwenden Sie stattdessen
`plugins.entries.codex.config.appServer.mode: "guardian"` oder `OPENCLAW_CODEX_APP_SERVER_MODE=guardian` für einmalige lokale Tests. Für
wiederholbare Bereitstellungen wird die Konfiguration bevorzugt, da sie das
Plugin-Verhalten in derselben geprüften Datei wie die übrige Einrichtung des
Codex-Harness festhält.

## Native Codex-Plugins

Die native Unterstützung für Codex-Plugins nutzt die eigenen App- und
Plugin-Funktionen des Codex-App-Servers im selben Codex-Thread wie der
OpenClaw-Harness-Turn. OpenClaw übersetzt Codex-Plugins nicht in synthetische
dynamische OpenClaw-Tools vom Typ `codex_plugin_*`.

`codexPlugins` betrifft nur Sitzungen, die den nativen Codex-Harness
auswählen. Die Einstellung hat keine Auswirkungen auf Ausführungen mit dem
integrierten Harness, normale OpenAI-Provider-Ausführungen,
ACP-Konversationsbindungen oder andere Harnesses.

Minimale migrierte Konfiguration:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          codexPlugins: {
            enabled: true,
            allow_destructive_actions: true,
            plugins: {
              "google-calendar": {
                enabled: true,
                marketplaceName: "openai-curated",
                pluginName: "google-calendar",
              },
            },
          },
        },
      },
    },
  },
}
```

Die Thread-App-Konfiguration wird berechnet, wenn OpenClaw eine
Codex-Harness-Sitzung einrichtet oder eine veraltete Codex-Thread-Bindung
ersetzt; sie wird nicht bei jedem Turn neu berechnet. Verwenden Sie nach einer
Änderung an `codexPlugins` `/new`, `/reset` oder
starten Sie den Gateway neu, damit zukünftige Codex-Harness-Sitzungen mit dem
aktualisierten App-Satz beginnen.

Informationen zur Migrationseignung, zum App-Inventar, zu Richtlinien für
destruktive Aktionen, zu Rückfragen und zur Diagnose nativer Plugins finden
Sie unter [Native Codex-Plugins](/de/plugins/codex-native-plugins).

Der App- und Plugin-Zugriff auf OpenAI-Seite wird durch das angemeldete
Codex-Konto und bei Business- und Enterprise/Edu-Workspaces durch die
App-Steuerung des Workspace kontrolliert. Eine Übersicht von OpenAI zu Konten
und Workspace-Steuerung finden Sie unter
[Codex mit Ihrem ChatGPT-Tarif verwenden](https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan).

## Computer Use

Für Computer Use gibt es eine eigene Einrichtungsanleitung:
[Codex Computer Use](/de/plugins/codex-computer-use).

Kurzfassung: OpenClaw liefert die Desktop-Steuerungs-App nicht mit und führt
Desktop-Aktionen nicht selbst aus. Es bereitet den Codex-App-Server vor,
überprüft, ob der MCP-Server `computer-use` verfügbar ist, und überlässt
Codex anschließend während Turns im Codex-Modus die nativen MCP-Tool-Aufrufe.

## Laufzeitgrenzen

Der Codex-Harness ändert nur den eingebetteten Low-Level-Agent-Executor.

- Dynamische OpenClaw-Tools werden unterstützt. Codex fordert OpenClaw zur
  Ausführung dieser Tools auf, sodass OpenClaw Teil des Ausführungspfads
  bleibt.
- Codex-native Shell-, Patch-, MCP- und native App-Tools gehören zu Codex.
  OpenClaw kann ausgewählte native Ereignisse über das unterstützte Relay
  beobachten oder blockieren, schreibt die Argumente nativer Tools jedoch
  nicht um.
- Codex ist für native Compaction zuständig. OpenClaw verwaltet einen
  Transkriptspiegel für den Kanalverlauf, die Suche, `/new`,
  `/reset` und zukünftige Modell- oder Harness-Wechsel, ersetzt die
  Codex-Compaction jedoch nicht durch eine Zusammenfassung von OpenClaw oder
  der Kontext-Engine.
- Mediengenerierung, Medienanalyse, TTS, Genehmigungen und die Ausgabe von
  Messaging-Tools werden weiterhin über die entsprechenden
  OpenClaw-Provider-/Modelleinstellungen verarbeitet.
- `tool_result_persist` gilt für OpenClaw-eigene
  Transkript-Tool-Ergebnisse, nicht für Codex-native
  Tool-Ergebnisdatensätze.

Informationen zu Hook-Ebenen, unterstützten V1-Oberflächen, der nativen
Berechtigungsverarbeitung, der Warteschlangensteuerung, den Mechanismen zum
Hochladen von Codex-Feedback und Details zur Compaction finden Sie unter
[Laufzeit des Codex-Harness](/de/plugins/codex-harness-runtime).

## Fehlerbehebung

**Codex wird nicht als normaler `/model`-Provider angezeigt:** Dies
ist bei neuen Konfigurationen zu erwarten. Wählen Sie ein
`openai/gpt-*`-Modell aus, aktivieren Sie `plugins.entries.codex.enabled` und prüfen
Sie, ob `plugins.allow` den Wert `codex` ausschließt.

**OpenClaw verwendet statt Codex den integrierten Harness:** Vergewissern Sie
sich, dass die effektive Route exakt einer offiziellen
HTTPS-Platform-Responses- oder ChatGPT-Responses-Route entspricht, keine
selbst definierte Anfrageüberschreibung enthält und das Codex-Plugin
installiert und aktiviert ist. Das Präfix `openai/gpt-*` allein reicht
nicht aus. Um beim Testen einen strikten Nachweis zu erhalten, setzen Sie
`agentRuntime.id: "codex"` für den Provider oder das Modell; bei erzwungenem Codex
schlägt die Ausführung fehl, statt auf eine Alternative zurückzugreifen, wenn
die Route oder der Harness inkompatibel ist.

**Die OpenAI-Codex-Laufzeit greift auf den API-Schlüsselpfad zurück:** Erfassen
Sie einen redigierten Gateway-Auszug, der das Modell, die Laufzeit, den
ausgewählten Provider und den Fehler zeigt. Bitten Sie betroffene
Mitwirkende, diesen schreibgeschützten Befehl auf ihrem OpenClaw-Host
auszuführen:

```bash
(
  pattern='openai/gpt-5\.[45]|openai[-]codex|agentRuntime(\.id)?|harnessRuntime|Runtime: OpenAI Codex|legacy OpenAI Codex prefix|resolveSelectedOpenAIRuntimeProvider|candidateProvider[": ]+openai|status[": ]+401|Incorrect API key|No API key|api-key path|API-key path|OAuth'

  if ls /tmp/openclaw/openclaw-*.log >/dev/null 2>&1; then
    grep -E -i -n "$pattern" /tmp/openclaw/openclaw-*.log 2>/dev/null || true
  else
    journalctl --user -u openclaw-gateway --since today --no-pager 2>/dev/null \
      | grep -E -i "$pattern" || true
  fi
) | sed -E \
    -e 's/(Authorization: Bearer )[A-Za-z0-9._~+\/-]+/\1[REDACTED]/Ig' \
    -e 's/(Bearer )[A-Za-z0-9._~+\/-]+/\1[REDACTED]/Ig' \
    -e 's/(api[_ -]?key[=: ]+)[^ ,}"]+/\1[REDACTED]/Ig' \
    -e 's/(OPENAI_API_KEY[=: ]+)[^ ,}"]+/\1[REDACTED]/Ig' \
    -e 's/sk-[A-Za-z0-9_-]{12,}/sk-[REDACTED]/g' \
    -e 's/[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}/[EMAIL-REDACTED]/g' \
  | tail -200
```

Nützliche Auszüge enthalten üblicherweise `openai/gpt-5.6-sol` oder
`openai/gpt-5.6-luna`, `Runtime: OpenAI Codex`, `agentRuntime.id` oder
`harnessRuntime`, `candidateProvider: "openai"` sowie ein Ergebnis vom Typ
`401`, `Incorrect API key` oder `No API key`. Eine
korrigierte Ausführung sollte statt eines einfachen Fehlers mit einem
OpenAI-API-Schlüssel den OpenAI-OAuth-Pfad anzeigen.

**Die Konfiguration enthält weiterhin veraltete Codex-Modellreferenzen:** Führen Sie `openclaw doctor --fix` aus.
Doctor schreibt veraltete Modellreferenzen in `openai/*` um, entfernt überholte Laufzeit-Pins für Sitzungen und
den gesamten Agenten und behält vorhandene Überschreibungen von Authentifizierungsprofilen bei.

**Der App-Server wird abgelehnt:** Verwenden Sie einen stabilen Codex-App-Server aus `0.143.0`
über das gebündelte `0.145.0`. Vorabversionen, Versionen mit Build-Suffix und neuere,
nicht validierte Releases werden abgelehnt, da OpenClaw generierte Schemas
gegen die gebündelte App-Server-Version validiert.

**`/codex status` kann keine Verbindung herstellen:** Prüfen Sie, ob das Plugin `codex`
aktiviert ist, ob `plugins.allow` es enthält, wenn eine Positivliste
konfiguriert ist, und ob benutzerdefinierte `appServer.command`, `url`, `authToken` oder
Header gültig sind.

**Der Codex-App-Server verwendet zu viel Arbeitsspeicher:** Unterscheiden Sie zunächst die beiden Prozesse.
OpenClaw führt den lokalen Codex-App-Server als separaten Rust-Kindprozess aus.
`NODE_OPTIONS=--max-old-space-size=...` ändert nur den Node.js-V8-Heap
des Gateways; Codex wird dadurch weder begrenzt noch vergrößert. Verwaltete Gateway-Installationen wählen bereits
einen adaptiven V8-Heap, und dessen Vergrößerung kann weniger Host-Arbeitsspeicher für Codex übrig lassen. Verwenden Sie
[Fehlerbehebung bei Gateway-Arbeitsspeicherproblemen](/de/gateway/troubleshooting#gateway-exits-during-high-memory-use)
bei Speicherdruck auf das Gateway und prüfen Sie den Host- oder Container-Arbeitsspeicher für den Codex-Kindprozess.

Das gebündelte Codex hat weder eine Heap- noch eine RSS-Begrenzung und keine konfigurierbare
Verzögerung für das Entladen im Leerlauf. Nachdem der letzte Client sein Abonnement beendet hat, kann ein inaktiver Thread
bis zu 30 Minuten geladen bleiben. Reduzieren Sie auf Hosts mit begrenzten Ressourcen zunächst die Auffächerung nativer Codex-Subagenten,
bevor Sie den Gateway-Heap vergrößern:

```json5
{
  plugins: {
    entries: {
      codex: {
        config: {
          appServer: {
            args: ["-c", "agents.max_threads=3", "app-server", "--listen", "stdio://"],
          },
        },
      },
    },
  },
}
```

Diese Einstellung begrenzt native Kind-Threads für das standardmäßig gebündelte
Multi-Agent-Backend von Codex. Wenn Sie Codex Multi-Agent v2 ausdrücklich aktivieren, verwenden Sie stattdessen
`features.multi_agent_v2.max_concurrent_threads_per_session=3`; die v2-
Begrenzung schließt den Root-Thread ein und kann nicht mit `agents.max_threads` kombiniert werden.
Um Codex mehr Spielraum zu geben, erhöhen Sie die Arbeitsspeicherzuweisung
des Hosts, Containers oder der cgroup. Eine harte Betriebssystembegrenzung kann Codex beenden, anstatt Gegendruck auszuüben.

**Die Modellerkennung ist langsam:** Verringern Sie
`plugins.entries.codex.config.discovery.timeoutMs` oder deaktivieren Sie die Erkennung.
Siehe [Codex-Harness-Referenz](/de/plugins/codex-harness-reference#model-discovery).

**Der WebSocket-Transport schlägt sofort fehl:** Prüfen Sie `appServer.url`,
`authToken`, die Header und ob der entfernte App-Server dieselbe Codex-
App-Server-Protokollversion verwendet. Der Codex-WebSocket-Transport bleibt experimentell
und wird nicht unterstützt; bevorzugen Sie verwaltetes stdio oder den lokalen Unix-Steuerungssocket.

**Native Shell- oder Patch-Tools werden mit `Native hook relay
unavailable` blockiert:** Der Codex-Thread versucht weiterhin, eine native Hook-Relay-
ID zu verwenden, die bei OpenClaw nicht mehr registriert ist. Dies ist ein Transportproblem des nativen Codex-Hooks
und kein Fehler des ACP-Backends, Providers, von GitHub oder eines Shell-Befehls.
Starten Sie mit `/new` oder `/reset` eine neue Sitzung im betroffenen Chat
und versuchen Sie anschließend einen harmlosen Befehl erneut. Wenn dies einmal funktioniert, aber der nächste native Tool-
Aufruf erneut fehlschlägt, behandeln Sie `/new` nur als vorübergehende Problemumgehung: Kopieren Sie den
Prompt in eine neue Sitzung, nachdem Sie den Codex-App-Server oder das
OpenClaw Gateway neu gestartet haben, damit alte Threads verworfen und native Hook-Registrierungen
neu erstellt werden.

**Codex-Tool-Aufrufe erzeugen zu viele kurzlebige Hook-Prozesse:** Legen Sie
`plugins.entries.codex.config.appServer.loopDetectionPreToolUseRelay: false`
fest und starten Sie das Gateway neu. Dadurch wird nur der Codex-Unterprozess `PreToolUse`
deaktiviert, der für die OpenClaw-Schleifenerkennung und deren Marker ohne Richtlinie verwendet wird. Erforderliche
`before_tool_call` und Richtlinien-Relays für vertrauenswürdige Tools bleiben aktiviert.

**Ein Nicht-Codex-Modell verwendet den integrierten Harness:** Dies ist zu erwarten, sofern die Laufzeitrichtlinie
des Providers oder Modells es nicht an einen anderen Harness weiterleitet. Einfache Referenzen von Nicht-OpenAI-
Providern verbleiben im Modus `auto` auf ihrem normalen Provider-Pfad.

**Computer Use ist installiert, aber die Tools werden nicht ausgeführt:** Prüfen Sie
`/codex computer-use status` in einer neuen Sitzung. Wenn ein Tool
`Native hook relay unavailable` meldet, verwenden Sie die oben beschriebene Wiederherstellung des nativen Hook-Relays.
Siehe [Codex Computer Use](/de/plugins/codex-computer-use#troubleshooting).

## Verwandte Themen

- [Codex-Harness-Referenz](/de/plugins/codex-harness-reference)
- [Codex-Harness-Laufzeit](/de/plugins/codex-harness-runtime)
- [Codex-Überwachung](/de/plugins/codex-supervision)
- [Native Codex-Plugins](/de/plugins/codex-native-plugins)
- [Codex Computer Use](/de/plugins/codex-computer-use)
- [Agentenlaufzeiten](/de/concepts/agent-runtimes)
- [Modell-Provider](/de/concepts/model-providers)
- [OpenAI-Provider](/de/providers/openai)
- [Hilfe zu OpenAI Codex](https://help.openai.com/en/collections/14937394-codex)
- [Agent-Harness-Plugins](/de/plugins/sdk-agent-harness)
- [Plugin-Hooks](/de/plugins/hooks)
- [Diagnoseexport](/de/gateway/diagnostics)
- [Status](/de/cli/status)
- [Tests](/de/help/testing-live#live-codex-app-server-harness-smoke)
