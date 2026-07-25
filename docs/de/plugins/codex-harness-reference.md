---
read_when:
    - Sie benötigen jedes Konfigurationsfeld des Codex-Harnesses
    - Sie ändern das Transport-, Authentifizierungs-, Erkennungs- oder Zeitüberschreitungsverhalten des App-Servers
    - Sie debuggen den Start des Codex-Harness, die Modellerkennung oder die Umgebungsisolierung
summary: Konfigurations-, Authentifizierungs-, Ermittlungs- und App-Server-Referenz für das Codex-Harness
title: Codex-Harness-Referenz
x-i18n:
    generated_at: "2026-07-24T13:17:37Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 149f065f5bef18d0f491c97facc4b5991afc3f7e1077abdc7a4b49f506eac3e0
    source_path: plugins/codex-harness-reference.md
    workflow: 16
---

Diese Referenz behandelt die detaillierte Konfiguration für das offizielle Plugin `codex`.
Beginnen Sie für die Einrichtung und Routing-Entscheidungen mit dem
[Codex-Harness](/de/plugins/codex-harness).

## Plugin-Konfigurationsoberfläche

Alle Einstellungen des Codex-Harness befinden sich unter `plugins.entries.codex.config`.

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          discovery: {
            enabled: true,
            timeoutMs: 2500,
          },
          appServer: {
            mode: "guardian",
          },
        },
      },
    },
  },
}
```

Felder der obersten Ebene:

| Feld                       | Standard                 | Bedeutung                                                                                                                                      |
| -------------------------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `discovery`                | aktiviert                | Einstellungen zur Modellerkennung für Codex-App-Server `model/list`.                                                                           |
| `appServer`                | verwalteter stdio-App-Server | Einstellungen für Transport, Befehl, Authentifizierung, Genehmigung, Sandbox und Zeitüberschreitung. Der reguläre Harness verwendet standardmäßig agentenspezifischen Zustand. |
| `codexDynamicToolsLoading` | `"searchable"`           | Verwenden Sie `"direct"`, um dynamische OpenClaw-Tools direkt in den anfänglichen Codex-Toolkontext aufzunehmen.                              |
| `codexDynamicToolsExclude` | `[]`                     | Zusätzliche Namen dynamischer OpenClaw-Tools, die bei Codex-App-Server-Durchläufen ausgelassen werden sollen.                                   |
| `codexPlugins`             | deaktiviert              | Native Unterstützung für Codex-Plugins/-Apps, einschließlich optionalem Zugriff auf Apps verbundener Konten. Siehe [Native Codex-Plugins](/de/plugins/codex-native-plugins). |
| `computerUse`              | deaktiviert              | Einrichtung von Codex Computer Use. Siehe [Codex Computer Use](/de/plugins/codex-computer-use).                                                    |
| `sessionCatalog`           | aktiviert                | Native Codex-Sitzungserkennung für die Seitenleiste. Setzen Sie `enabled: false`, um die Erkennung zu deaktivieren, ohne den Provider oder Harness zu deaktivieren. |
| `supervision`              | deaktiviert              | Agentenseitige Richtlinie für Transkripte und Schreibsteuerung nativer Sitzungen. Siehe [Codex-Überwachung](/de/plugins/codex-supervision).         |

## Überwachung

Die native Sitzungserkennung listet standardmäßig nicht archivierte Codex-Sitzungen vom Gateway-
Computer und von entsprechend aktivierten gekoppelten Nodes auf. Deaktivieren Sie nur diesen Katalog mit:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          sessionCatalog: {
            enabled: false,
          },
        },
      },
    },
  },
}
```

`supervision` steuert agentenseitige Tools separat:

| Feld                  | Standard                | Bedeutung                                                                                                                                                                                                                                |
| --------------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enabled`             | `false`                 | Agentenseitige Codex-Überwachungstools aktivieren. Dies steuert nicht den authentifizierten Sitzungskatalog für Bediener.                                                                                                                 |
| `endpoints`           | integrierter lokaler Endpunkt | Kompatibilitäts- und erweiterte Endpunktziele für den beibehaltenen Codex-Überwachungsagenten und eigenständige MCP-Tools. Der menschliche Katalog und der Branch-Ablauf ignorieren diese Ziele und verwenden den über `appServer` aufgelösten Überwachungs-App-Server. |
| `allowRawTranscripts` | `false`                 | Bei aktivierter Überwachung autonome Transkriptlesevorgänge durch Agenten oder eigenständige MCP-Tools sowie aus Transkripten abgeleitete Listenfelder zulassen. Reine Metadaten-Lesevorgänge über `codex_threads` bleiben verfügbar. Steuert nicht die authentifizierte Fortsetzung in der Control UI. |
| `allowWriteControls`  | `false`                 | Bei aktivierter Überwachung autonome `codex_threads`-Mutationen zum Forken, Umbenennen, Archivieren und Wiederherstellen sowie eigenständige MCP-Vorgänge zum Senden, Steuern und Unterbrechen zulassen. Umgeht keine anderen Prüfungen auf Bindung, Host, Status oder Bestätigung. |

Endpunkteinträge akzeptieren diese Felder:

| Feld           | Gilt für      | Bedeutung                                                             |
| -------------- | ------------- | --------------------------------------------------------------------- |
| `id`           | alle          | Stabile Endpunkt-ID.                                                  |
| `label`        | alle          | Optionale Anzeigebezeichnung.                                         |
| `transport`    | alle          | `"stdio-proxy"` oder `"websocket"`.                                  |
| `command`      | `stdio-proxy` | Optionaler App-Server-Befehl.                                         |
| `args`         | `stdio-proxy` | Optionale Befehlsargumente.                                           |
| `cwd`          | `stdio-proxy` | Optionales Arbeitsverzeichnis des untergeordneten Prozesses.          |
| `url`          | `websocket`   | Erforderliche WebSocket- oder unterstützte lokale Socket-URL.         |
| `authTokenEnv` | `websocket`   | Optionale Umgebungsvariable, deren Wert den Endpunkt authentifiziert. |

Die Seite **Codex-Sitzungen** verwendet den Überwachungs-App-Server des Plugins und zeigt
nur nicht archivierte Sitzungen an. Ohne explizite Verbindungseinstellungen unter `appServer`
wird diese Verbindung als verwaltetes stdio im Benutzer-Home betrieben. Gespeicherte oder inaktive lokale Zeilen können
einen modellgebundenen Chat mit begrenztem Benutzer- und Assistentenverlauf bis zum letzten
persistierten abschließenden Quelldurchlauf erstellen. Seine private Bindung hält den Snapshot-Fork,
den kanonischen Branch aus der `appServer`-Quelle, die Verlaufseinspeisung und spätere Durchläufe auf dieser
Verbindung. Beim ersten kanonischen Start wird das vom Fork zurückgegebene Paar verwendet. Bei späteren
Fortsetzungen werden OpenClaw-Modell- und Provider-Überschreibungen ausgelassen, damit Codex das
persistierte Paar des kanonischen Threads wiederherstellt; eine separate native Änderung kann dieses
Paar aktualisieren, aber das äußere Modell und die Fallback-Kette ersetzen es niemals. Gespeicherte und inaktive
Zeilen können nach Bestätigung, dass kein anderer Runner vorhanden ist, archiviert werden, sofern nicht eine andere aktive
OpenClaw-Bindung das genaue Ziel oder einen seiner nicht archivierten erzeugten
Nachfolger besitzt. OpenClaw folgt der Nachfolger-Paginierung von Codex und bricht bei
Aufzählungsfehlern, Zyklen oder Ausschöpfung des Sicherheitslimits sicher ab. Die Bestätigung deckt weiterhin
unbekannte native Clients und das Zeitfenster zwischen Statusprüfung und Archivierung ab. Ein überwachter
modellgebundener Chat kann nicht gelöscht werden, solange er die native Bindung schützt.
Aktive Quellen können keinen Branch erstellen oder archiviert werden, ein vorhandener überwachter
Chat kann jedoch weiterhin geöffnet werden. Jede Zeile eines gekoppelten Nodes bleibt schreibgeschützt; der Node-
Transport stellt den vom Harness benötigten Streaming-Lebenszyklus noch nicht bereit.

`appServer.homeScope: "user"` allein ändert, welches Codex-Home ein verwalteter Harness-
Prozess verwendet; dadurch wird der Flottenkatalog nicht veröffentlicht. Das Aktivieren der Überwachung ändert
den Harness-Standard nicht. Stattdessen verwendet die separate Überwachungsverbindung
standardmäßig verwaltetes stdio im Benutzer-Home, wenn keine expliziten Verbindungseinstellungen unter `appServer`
vorhanden sind. Explizite Einstellungen werden für diese Verbindung berücksichtigt.
Ausstehende und festgeschriebene überwachte Bindungen behalten diese Verbindung für jeden Durchlauf bei;
eine deaktivierte Überwachung oder eine Abweichung bei Verbindung bzw. Lebenszyklus bricht sicher ab, anstatt
auf den Agenten-Home-Harness zurückzufallen. Die Standardverbindung teilt gespeicherte
Sitzungen mit nativen Codex-Clients, nicht deren prozesslokalen Aktivitätsstatus.

Veraltete Einstellungen unter `plugins.entries.codex-supervisor` werden nicht mehr unterstützt. Führen Sie
`openclaw doctor --fix` aus, um den alten Eintrag, die Endpunktdefinitionen, Richtlinien-
Flags und Plugin-Zulassungs-/Sperrverweise in diesen Block zu migrieren. Explizite kanonische
Werte unter `codex.config.supervision` haben bei Konflikten Vorrang.

## App-Server-Transport

Für reguläre Harness-Durchläufe startet OpenClaw die verwaltete Codex-Binärdatei, die
mit dem offiziellen Plugin ausgeliefert wird (derzeit `@openai/codex` `0.145.0`):

```bash
codex app-server --listen stdio://
```

Dadurch bleibt die App-Server-Version an das offizielle Plugin `codex` gebunden, statt
an eine beliebige separat lokal installierte Codex-CLI. Setzen Sie
`appServer.command` nur, wenn Sie bewusst eine andere ausführbare Datei verwenden möchten.
Reguläre verwaltete Durchläufe mit dem standardmäßig isolierten Agenten-Home bevorzugen dieses angeheftete
Paket auch dann, wenn ein macOS-Desktop-Bundle installiert ist. Wenn
[Computer Use](/de/plugins/codex-computer-use) aktiviert ist oder `homeScope`
den Wert `"user"` hat und nativen Computer-Use-Zustand laden kann, bevorzugt der verwaltete Start stattdessen
die Binärdatei der Desktop-App, die über die erforderlichen macOS-Berechtigungen verfügt. Dieselbe
Desktop-zuerst-Regel gilt, wenn die wirksame Codex-Konfiguration eines isolierten Agenten-Homes
native Computer Use aktiviert. Wenn kein Desktop-App-Bundle installiert ist, greift OpenClaw
auf die Binärdatei des angehefteten Pakets zurück.

Die Übergabe ausführbarer Dateien und die Abschirmung nativer Konfiguration koordinieren Clients innerhalb eines
laufenden Gateway-Prozesses. Starten Sie das Gateway neu, nachdem ein anderer Prozess die
native Codex-Plugin-Konfiguration geändert hat.

Die Überwachung löst eine separate Verbindung auf. Ohne explizite
Verbindungseinstellungen unter `appServer` verwendet sie verwaltetes stdio mit `homeScope: "user"`;
der reguläre Harness bleibt bei verwaltetem stdio mit `homeScope: "agent"`. Explizite
Verbindungseinstellungen werden von beiden Pfaden berücksichtigt. Setzen Sie `homeScope: "user"`
explizit, wenn der reguläre Harness `$CODEX_HOME` (oder `~/.codex`)
mit nativen Clients teilen soll. Eine private überwachte Bindung verwendet unabhängig vom Standard
des regulären Harness die Überwachungsverbindung. Unabhängige App-Server-
Prozesse behalten getrennte Live-Status- und Genehmigungszustände bei.

Für nicht produktive Tests mit einem bereits laufenden App-Server ist der WebSocket-
Transport verfügbar:

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
            requestTimeoutMs: 60000,
          },
        },
      },
    },
  },
}
```

Codex stuft den WebSocket-Transport als experimentell und nicht unterstützt ein. Bevorzugen Sie
für Produktions-Workloads verwaltetes stdio oder den lokalen Unix-Steuerungssocket.

Felder unter `appServer`:

| Feld                                          | Standard                                               | Bedeutung                                                                                                                                                                                                                                                                                                                                                                                       |
| --------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `transport`                                   | `"stdio"`                                              | `"stdio"` startet Codex; ein explizites `"unix"` stellt eine Verbindung zum lokalen Steuerungs-Socket her; `"websocket"` stellt eine Verbindung zu `url` her.                                                                                                                                                                                                                                     |
| `homeScope`                                   | `"agent"`                                              | `"agent"` isoliert den gewöhnlichen Harness-Zustand je OpenClaw-Agent. `"user"` ist eine explizite Opt-in-Option, die das native `$CODEX_HOME` oder `~/.codex` gemeinsam nutzt, die native Authentifizierung verwendet und die Thread-Verwaltung ausschließlich für den Eigentümer aktiviert. Der Benutzerbereich unterstützt lokales stdio oder Unix-Transport. Für die separate Überwachungsverbindung wird ein nicht festgelegter Wert bei stdio oder Unix als `"user"` und bei WebSocket als `"agent"` aufgelöst. |
| `command`                                     | verwaltete Codex-Binärdatei                            | Ausführbare Datei für den stdio-Transport. Lassen Sie den Wert nicht festgelegt, um die verwaltete Binärdatei zu verwenden.                                                                                                                                                                                                                                                                      |
| `args`                                        | `["app-server", "--listen", "stdio://"]`               | Argumente für den stdio-Transport.                                                                                                                                                                                                                                                                                                                                                              |
| `url`                                         | nicht festgelegt                                      | WebSocket-App-Server-URL oder `unix://`-URL. Ein explizit leerer Unix-Pfad wählt den kanonischen Steuerungs-Socket im Benutzerverzeichnis aus.                                                                                                                                                                                                                                           |
| `authToken`                                   | nicht festgelegt                                      | Bearer-Token für den WebSocket-Transport. Akzeptiert eine Literalzeichenfolge oder SecretInput wie `${CODEX_APP_SERVER_TOKEN}`.                                                                                                                                                                                                                                                                          |
| `headers`                                     | `{}`                                                   | Zusätzliche WebSocket-Header. Headerwerte akzeptieren Literalzeichenfolgen oder SecretInput-Werte, beispielsweise `x-codex-client-session-token: "${CODEX_CLIENT_SESSION_TOKEN}"`.                                                                                                                                                                                                                                                            |
| `clearEnv`                                    | `[]`                                                   | Zusätzliche Namen von Umgebungsvariablen, die aus dem gestarteten stdio-App-Server-Prozess entfernt werden, nachdem OpenClaw dessen geerbte Umgebung erstellt hat.                                                                                                                                                                                                                                |
| `remoteWorkspaceRoot`                         | nicht festgelegt                                      | Stammverzeichnis des Remote-Codex-App-Server-Arbeitsbereichs. Wenn festgelegt, leitet OpenClaw das lokale Arbeitsbereich-Stammverzeichnis aus dem aufgelösten OpenClaw-Arbeitsbereich ab, behält das aktuelle cwd-Suffix unter diesem Remote-Stammverzeichnis bei und sendet nur das endgültige App-Server-cwd an Codex. Wenn sich das cwd außerhalb des aufgelösten OpenClaw-Arbeitsbereich-Stammverzeichnisses befindet, verweigert OpenClaw den Vorgang, statt einen Gateway-lokalen Pfad an den Remote-App-Server zu senden. |
| `loopDetectionPreToolUseRelay`                | `true`                                                 | Installiert den Codex-Unterprozess `PreToolUse`, der ausschließlich für die OpenClaw-Schleifenerkennung und deren explizite Markierung ohne Richtlinie verwendet wird. Legen Sie `false` fest, um die Prozessauffächerung pro Tool zu reduzieren. Plugin-Hooks vor der Tool-Ausführung und die Richtlinie für vertrauenswürdige Tools installieren weiterhin das erforderliche Relay. |
| `requestTimeoutMs`                            | `60000`                                                | Zeitüberschreitung für Aufrufe der App-Server-Steuerungsebene.                                                                                                                                                                                                                                                                                                                                  |
| `turnCompletionIdleTimeoutMs`                 | `60000`                                                | Ruhefenster, nachdem Codex einen Turn akzeptiert hat oder nach einer Turn-bezogenen App-Server-Anfrage, während OpenClaw auf `turn/completed` wartet.                                                                                                                                                                                                                                           |
| `turnAssistantCompletionIdleTimeoutMs`        | `10000`                                                | Ruhefenster, nachdem ein endgültiges/nicht kommentierendes Assistentenelement oder der Abschluss einer rohen Assistentenausgabe vor einem Tool die Freigabe der Assistentenausgabe aktiviert, während OpenClaw weiterhin auf `turn/completed` wartet. Eine Erhöhung gibt Codex mehr Zeit, `turn/completed` auszugeben, bevor OpenClaw unterbricht und die Sitzungsspur freigibt.                     |
| `postToolRawAssistantCompletionIdleTimeoutMs` | `300000`                                               | Abschlussleerlauf- und Fortschrittsüberwachung, die nach einer Tool-Übergabe, dem Abschluss eines nativen Tools, dem Fortschritt einer rohen Assistentenausgabe nach einem Tool, dem Abschluss einer rohen Schlussfolgerung oder einem Schlussfolgerungsfortschritt verwendet wird, während OpenClaw auf `turn/completed` wartet. Verwenden Sie dies für vertrauenswürdige oder rechenintensive Arbeitslasten, bei denen die Synthese nach einem Tool berechtigterweise länger ruhig bleiben kann als das Freigabebudget des endgültigen Assistenten. |
| `mode`                                        | `"yolo"`, es sei denn, lokale Codex-Anforderungen lassen YOLO nicht zu | Voreinstellung für YOLO- oder durch einen Guardian geprüfte Ausführung.                                                                                                                                                                                                                                                                                                                         |
| `approvalPolicy`                              | `"never"` oder eine zulässige Guardian-Genehmigungsrichtlinie | Native Codex-Genehmigungsrichtlinie, die beim Start und Fortsetzen des Threads sowie beim Turn gesendet wird.                                                                                                                                                                                                                                                                                    |
| `sandbox`                                     | `"danger-full-access"` oder eine zulässige Guardian-Sandbox | Nativer Codex-Sandbox-Modus, der beim Start und Fortsetzen des Threads gesendet wird. Aktive OpenClaw-Sandboxes beschränken `danger-full-access`-Turns auf Codex `workspace-write`; das Netzwerk-Flag des Turns folgt dem ausgehenden Netzwerkverkehr der OpenClaw-Sandbox.                                                                                                                            |
| `approvalsReviewer`                           | `"user"` oder ein zulässiger Guardian-Prüfer           | Verwenden Sie `"auto_review"`, damit Codex native Genehmigungsaufforderungen prüft, sofern dies zulässig ist.                                                                                                                                                                                                                                                                                 |
| `defaultWorkspaceDir`                         | aktuelles Prozessverzeichnis                            | Von `/codex bind` verwendeter Arbeitsbereich, wenn `--cwd` nicht angegeben ist.                                                                                                                                                                                                                                                                                                 |
| `serviceTier`                                 | nicht festgelegt                                      | Optionale Dienststufe des Codex-App-Servers. `"priority"` aktiviert das Fast-Mode-Routing, `"flex"` fordert die Flex-Verarbeitung an und `null` entfernt die Überschreibung. Das veraltete `"fast"` wird als `"priority"` akzeptiert.                                                                                                                     |
| `networkProxy`                                | deaktiviert                                            | Aktiviert optional das Netzwerk des Codex-Berechtigungsprofils für App-Server-Befehle. OpenClaw definiert die ausgewählte `permissions.<profile>.network`-Konfiguration und wählt sie mit `default_permissions` aus, anstatt `sandbox` zu senden.                                                                                                                                                          |
| `experimental.sandboxExecServer`              | `false`                                                | Optionale Vorschaufunktion, die eine durch die OpenClaw-Sandbox gestützte Codex-Umgebung beim unterstützten Codex-App-Server registriert, sodass die native Codex-Ausführung innerhalb der aktiven OpenClaw-Sandbox erfolgen kann.                                                                                                                                                                                                            |

`appServer.networkProxy` ist explizit, da es den Sandbox-Vertrag von Codex
ändert. Wenn diese Option aktiviert ist, setzt OpenClaw außerdem `features.network_proxy.enabled` und
`default_permissions` in der Codex-Thread-Konfiguration, damit das generierte Berechtigungsprofil
die von Codex verwaltete Netzwerkfunktion starten kann. OpenClaw generiert standardmäßig einen
kollisionsresistenten `openclaw-network-<fingerprint>`-Profilnamen aus dem
Profilinhalt; verwenden Sie `profileName` nur, wenn ein stabiler lokaler Name
erforderlich ist.

```js
export default {
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
              allowUpstreamProxy: true,
              proxyUrl: "http://127.0.0.1:3128",
            },
          },
        },
      },
    },
  },
};
```

Wenn die normale App-Server-Laufzeit `danger-full-access` wäre, verwendet die Aktivierung von
`networkProxy` stattdessen einen Dateisystemzugriff im Workspace-Stil für das generierte
Berechtigungsprofil. Die von Codex verwaltete Netzwerkdurchsetzung ist eine Sandbox-
Netzwerkfunktion, daher würde ein Profil mit Vollzugriff ausgehenden Datenverkehr nicht schützen.

Das Plugin blockiert ältere, neuere noch nicht validierte, Vorabversions-, mit Build-Zusatz versehene oder
unversionierte App-Server-Handshakes. Der Codex-App-Server muss eine stabile Version
von `0.143.0` bis einschließlich der gebündelten Version `0.145.0` melden.

OpenClaw behandelt WebSocket-App-Server-URLs, die nicht auf Loopback verweisen, als remote und erfordert
identitätstragende WebSocket-Authentifizierung über `appServer.authToken` oder einen
`Authorization`-Header. `appServer.authToken` und jeder `appServer.headers.*`-
Wert können ein SecretInput sein; die Secrets-Laufzeit löst SecretRefs und die
Kurzschreibweise für Umgebungsvariablen auf, bevor OpenClaw App-Server-Startoptionen erstellt, und nicht aufgelöste
strukturierte SecretRefs führen zu einem Fehler, bevor ein Token oder Header gesendet wird. Wenn native
Codex-Plugins konfiguriert sind, verwendet OpenClaw die Plugin-Steuerungsebene des verbundenen App-Servers,
um diese Plugins zu installieren oder zu aktualisieren, und aktualisiert anschließend das App-
Inventar, damit Plugin-eigene Apps für den Codex-Thread sichtbar sind. `app/list` bleibt
die maßgebliche Quelle für Inventar und Metadaten, aber die OpenClaw-Richtlinie
entscheidet, ob `thread/start` für eine
aufgeführte zugängliche App `config.apps[appId].enabled = true` sendet, selbst wenn Codex sie derzeit als deaktiviert kennzeichnet. Unbekannte oder
fehlende App-IDs führen weiterhin zu einem geschlossenen Fehlerzustand; dieser Pfad aktiviert ausschließlich Marketplace-
Plugins über `plugin/install` und aktualisiert das Inventar. Verbinden Sie OpenClaw nur mit
Remote-App-Servern, denen Sie zutrauen, von OpenClaw verwaltete Plugin-Installationen
und Aktualisierungen des App-Inventars zu akzeptieren.

## Genehmigungs- und Sandbox-Modi

Lokale stdio-App-Server-Sitzungen verwenden standardmäßig den YOLO-Modus:
`approvalPolicy: "never"`, `approvalsReviewer: "user"` und
`sandbox: "danger-full-access"`. Diese vertrauenswürdige lokale Betreiberkonfiguration ermöglicht es
unbeaufsichtigten OpenClaw-Durchläufen und Heartbeats, ohne native Genehmigungsabfragen,
die niemand beantworten kann, Fortschritte zu erzielen.

Wenn die lokale Systemanforderungsdatei von Codex implizite YOLO-Werte für Genehmigung,
Prüfer oder Sandbox nicht zulässt, behandelt OpenClaw den impliziten Standard stattdessen als Guardian
und wählt zulässige Guardian-Berechtigungen aus. `tools.exec.mode: "auto"`
erzwingt außerdem durch Guardian geprüfte Codex-Genehmigungen und behält unsichere
veraltete Überschreibungen für `approvalPolicy: "never"` oder `sandbox: "danger-full-access"` nicht bei;
setzen Sie `tools.exec.mode: "full"` für eine bewusst gewählte Konfiguration ohne Genehmigungen.
Einträge in `[[remote_sandbox_config]]`, deren Hostname übereinstimmt, werden in derselben Anforderungsdatei
bei der Entscheidung über den Sandbox-Standard berücksichtigt.

Setzen Sie `appServer.mode: "guardian"` für durch Codex Guardian geprüfte Genehmigungen:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          appServer: {
            mode: "guardian",
            serviceTier: "priority",
          },
        },
      },
    },
  },
}
```

Die Voreinstellung `guardian` wird zu `approvalPolicy: "on-request"`,
`approvalsReviewer: "auto_review"` und `sandbox: "workspace-write"` erweitert, wenn diese
Werte zulässig sind. Einzelne Richtlinienfelder überschreiben `mode`. Der ältere
Prüferwert `guardian_subagent` wird weiterhin als Kompatibilitätsalias akzeptiert,
neue Konfigurationen sollten jedoch `auto_review` verwenden.

Wenn eine OpenClaw-Sandbox aktiv ist, wird der lokale Codex-App-Server-Prozess weiterhin
auf dem Gateway-Host ausgeführt. OpenClaw deaktiviert daher für diesen Durchlauf den nativen Code Mode von Codex,
benutzerdefinierte MCP-Server und die durch Apps gestützte Plugin-Ausführung, anstatt
die hostseitige Sandbox von Codex als gleichwertig mit dem OpenClaw-Sandbox-
Backend zu behandeln. Shell-Zugriff wird über von der OpenClaw-Sandbox gestützte dynamische Tools
wie `sandbox_exec` und `sandbox_process` bereitgestellt, wenn die normalen exec-/process-Tools
verfügbar sind.

<Note>
Auf Docker-basierten OpenClaw-Sandbox-Hosts (`agents.defaults.sandbox.mode` auf
ein Docker-Backend gesetzt) prüft `openclaw doctor`, ob der Host die
User-Namespaces für nicht privilegierte Benutzer (und, wenn der Netzwerk-Egress der Docker-Sandbox deaktiviert ist,
die Network-Namespaces) zulässt, die der verschachtelte Codex-`bwrap`-Prozess für die `workspace-write`-
Shell-Ausführung innerhalb des Sandbox-Containers benötigt. Eine fehlgeschlagene Prüfung äußert sich
auf Ubuntu-/AppArmor-Hosts üblicherweise als `bwrap: setting up uid map: Permission denied` oder
`bwrap: loopback: Failed RTM_NEWADDR: Operation not permitted`.
Korrigieren Sie die gemeldete Host-Namespace-Richtlinie für den OpenClaw-
Dienstbenutzer und starten Sie das Gateway neu; bevorzugen Sie ein auf den
Dienstprozess beschränktes AppArmor-Profil gegenüber dem hostweiten
Fallback `kernel.apparmor_restrict_unprivileged_userns=0`, und gewähren Sie nicht
nur zur Unterstützung des verschachtelten `bwrap`-Prozesses weiterreichende Docker-Container-Berechtigungen.
</Note>

## Native Ausführung in der Sandbox

Der stabile Standard ist ein geschlossener Fehlerzustand: Eine aktive OpenClaw-Sandbox deaktiviert native
Codex-Ausführungsoberflächen, die andernfalls auf dem Host des Codex-App-Servers
ausgeführt würden. Verwenden Sie `appServer.experimental.sandboxExecServer: true` nur, wenn Sie
die Unterstützung für Remote-Umgebungen von Codex mit dem Sandbox-Backend von OpenClaw testen möchten.
Dieser Vorschaupfad funktioniert mit jeder unterstützten Codex-App-Server-Version.

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          appServer: {
            experimental: {
              sandboxExecServer: true,
            },
          },
        },
      },
    },
  },
}
```

Wenn das Flag aktiviert und die aktuelle OpenClaw-Sitzung in einer Sandbox ausgeführt wird, startet OpenClaw
einen lokalen Loopback-exec-Server, der von der aktiven Sandbox gestützt wird, registriert ihn
beim Codex-App-Server und startet den Codex-Thread und -Durchlauf mit dieser
OpenClaw-eigenen Umgebung. Wenn der App-Server die Umgebung nicht registrieren kann,
schlägt der Lauf geschlossen fehl, statt stillschweigend auf die Host-Ausführung zurückzufallen.

Dieser Vorschaupfad ist ausschließlich lokal verfügbar. Ein Remote-WebSocket-App-Server kann
den Loopback-exec-Server nur erreichen, wenn er auf demselben Host ausgeführt wird, daher lehnt OpenClaw
diese Kombination ab.

## Authentifizierungs- und Umgebungsisolation

Im standardmäßigen agentenspezifischen Home-Verzeichnis wird die Authentifizierung in dieser Reihenfolge ausgewählt:

1. Ein explizites OpenClaw-Codex-Authentifizierungsprofil für den Agenten.
2. Das vorhandene Konto des App-Servers im Codex-Home-Verzeichnis dieses Agenten.
3. Nur für lokale stdio-App-Server-Starts: `CODEX_API_KEY`, danach
   `OPENAI_API_KEY`, wenn kein App-Server-Konto vorhanden ist und weiterhin eine OpenAI-Authentifizierung
   erforderlich ist.

Wenn OpenClaw ein Codex-Authentifizierungsprofil im Stil eines ChatGPT-Abonnements erkennt (OAuth oder
Anmeldedatentyp „Token“), entfernt es `CODEX_API_KEY` und `OPENAI_API_KEY` aus
dem gestarteten Codex-Unterprozess. Dadurch bleiben API-Schlüssel auf Gateway-Ebene
für Embeddings oder direkte OpenAI-Modelle verfügbar, ohne dass native Codex-App-Server-
Durchläufe versehentlich über die API abgerechnet werden.

Explizite Codex-API-Schlüsselprofile und der lokale stdio-Fallback auf Umgebungsvariablenschlüssel verwenden
die App-Server-Anmeldung statt geerbter Unterprozess-Umgebungsvariablen. WebSocket-App-Server-
Verbindungen erhalten keinen Fallback auf API-Schlüssel aus der Gateway-Umgebung; verwenden Sie ein explizites Authentifizierungsprofil
oder das eigene Konto des Remote-App-Servers.

Starts von stdio-App-Servern erben standardmäßig die Prozessumgebung von OpenClaw.
OpenClaw verwaltet die Kontobrücke des Codex-App-Servers und setzt `CODEX_HOME` auf ein
agentenspezifisches Verzeichnis im OpenClaw-Zustand dieses Agenten. Dadurch bleiben Codex-
Konfiguration, Konten, Plugin-Cache/-Daten und Thread-Zustand auf den OpenClaw-
Agenten beschränkt, statt aus dem persönlichen Home-Verzeichnis `~/.codex` des Betreibers
einzufließen.

Setzen Sie `appServer.homeScope: "user"`, um den nativen Codex-Zustand mit Codex
Desktop und der CLI zu teilen. Dieser lokale Benutzer-Home-Modus unterstützt verwaltetes stdio und
expliziten Unix-Transport. Er verwendet `$CODEX_HOME`, wenn gesetzt, andernfalls `~/.codex`,
einschließlich nativer Authentifizierung, Konfiguration, Plugins und Threads.
OpenClaw überspringt seine Authentifizierungsprofilbrücke für den App-Server. Verifizierte Durchläufe des Besitzers
können `codex_threads` verwenden, um diese Threads aufzulisten (mit einem optionalen `search`-Filter),
zu lesen, zu forken, umzubenennen, zu archivieren und aus dem Archiv wiederherzustellen. Forken Sie einen Thread, bevor
Sie ihn in OpenClaw fortsetzen; unabhängige Codex-Prozesse koordinieren keine
gleichzeitigen Schreibzugriffe auf denselben Thread.

Diese Zustimmung zu `homeScope` gilt für gewöhnliche Harness-Sitzungen. Ein über
Codex Sessions erstellter Chat verwendet stattdessen seine private Überwachungsverbindung, die
die Authentifizierungs- und Provider-Konfiguration der nativen Verbindung für den
kanonischen Branch und zukünftige Wiederaufnahmen beibehält.

In einem modellgebundenen überwachten Chat kann `codex_threads` keinen anderen
Fork anhängen oder den an den Chat gebundenen nativen Thread archivieren. Auflisten und
ausschließlich Metadaten lesender Zugriff bleiben verfügbar. Das Lesen von Rohtranskripten erfordert `allowRawTranscripts`;
wenn diese Option deaktiviert ist, wird auch die Listensuche abgelehnt, da die native Suche
Übereinstimmungen in Transkriptvorschauen finden kann. Umbenennen, Wiederherstellen aus dem Archiv, abgelöstes Forken und
Archivieren eines nicht zugehörigen Threads, der keinem anderen OpenClaw-Chat gehört, erfordern
`allowWriteControls`. Keine der beiden Optionen umgeht eine gesperrte Bindung.

OpenClaw schreibt `HOME` für normale lokale App-Server-Starts nicht um.
Von Codex ausgeführte Unterprozesse wie `openclaw`, `gh`, `git`, Cloud-CLIs und Shell-
Befehle sehen das normale Prozess-Home-Verzeichnis und können Konfigurationen und
Token aus dem Benutzer-Home-Verzeichnis finden. Codex kann außerdem `$HOME/.agents/skills` und
`$HOME/.agents/plugins/marketplace.json` erkennen; diese Erkennung von `.agents` wird
absichtlich mit dem Home-Verzeichnis des Betreibers geteilt und ist vom isolierten
Zustand `~/.codex` getrennt.

Im standardmäßigen Agentenbereich werden OpenClaw-Plugins und Snapshots von OpenClaw-Skills
weiterhin über die eigene Plugin-Registry und den Skill-Loader von OpenClaw bereitgestellt; persönliche
Codex-Assets aus `~/.codex` dagegen nicht. Wenn Sie nützliche Codex-CLI-Skills oder
Plugins aus einem Codex-Home-Verzeichnis besitzen, die Teil eines isolierten OpenClaw-
Agenten werden sollen, inventarisieren Sie sie explizit:

```bash
openclaw migrate codex --dry-run
openclaw migrate apply codex --yes
```

Wenn eine Bereitstellung zusätzliche Umgebungsisolation erfordert, fügen Sie diese Variablen
zu `appServer.clearEnv` hinzu:

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

`appServer.clearEnv` wirkt sich nur auf den gestarteten Unterprozess des Codex-App-Servers aus.
OpenClaw entfernt `CODEX_HOME` und `HOME` während der lokalen Startnormalisierung
aus dieser Liste: `CODEX_HOME` verweist weiterhin auf den ausgewählten Agenten- oder Benutzerbereich,
und `HOME` wird weiterhin vererbt, damit Unterprozesse den normalen Zustand im Benutzer-Home-Verzeichnis verwenden können.

## Dynamische Tools

Dynamische Codex-Tools verwenden standardmäßig das Laden mit `searchable` und werden im
Namespace `openclaw` mit `deferLoading: true` bereitgestellt. OpenClaw stellt normalerweise keine
dynamischen Tools bereit, die native Workspace-Operationen von Codex oder
die eigene Tool-Suchoberfläche von Codex duplizieren:

- `read`
- `write`
- `edit`
- `apply_patch`
- `exec`
- `process`
- `update_plan`
- `tool_call`
- `tool_describe`
- `tool_search`
- `tool_search_code`

Wenn eine endliche Laufzeit-Zulassungsliste den nativen Code Mode deaktiviert, sendet OpenClaw eine
leere Auswahl der Ausführungsumgebung. In diesem direkten Fall ohne Sandbox
behält OpenClaw seine richtliniengefilterten Tools `exec` und `process` als Shell-
Fallback bei. Laufzeit-Zulassungslisten und `codexDynamicToolsExclude` gelten weiterhin.

Die meisten verbleibenden OpenClaw-Integrationstools, etwa für Messaging, Medien, Cron,
Browser, Nodes, Gateway, `heartbeat_respond` und `web_search`, sind
über die Codex-Tool-Suche unter diesem Namespace verfügbar. Dadurch bleibt der anfängliche
Modellkontext kleiner. Eine kleine Gruppe von Tools bleibt unabhängig von
`codexDynamicToolsLoading` direkt aufrufbar, da die Codex-Tool-Suche möglicherweise nicht verfügbar ist oder
nur eine Connector-Welt auflösen kann: `agents_list`, `sessions_spawn` und
`sessions_yield`. Entwickleranweisungen lenken normale Codex-Subagents
für Codex-native Subagent-Aufgaben weiterhin zu nativem `spawn_agent`, während
`sessions_spawn` für die ausdrückliche Delegation an OpenClaw oder ACP verfügbar bleibt.
Auch Quellantworten, die ausschließlich das Nachrichten-Tool verwenden, bleiben direkt, da dies ein
Vertrag zur Ablaufsteuerung ist.

Der Codex Code Mode stellt generische Ergebnisse dynamischer OpenClaw-Tools als Text dar. Parsen Sie ein
JSON-Ergebnis, bevor Sie Felder auslesen. Verschachtelte dynamische Aufrufe werden von der
Codex-Laufzeit serialisiert, daher sendet `Promise.all` sie nicht gleichzeitig; verwenden Sie beim
Starten untergeordneter Collector-Prozesse eine begrenzte sequenzielle Startschleife.

Mit `catalogMode: "direct-only"` markierte Tools, einschließlich des OpenClaw-Tools
`computer`, werden unter `openclaw_direct` gruppiert. OpenClaw fügt diesen Namespace
zur Liste `code_mode.direct_only_tool_namespaces` von Codex hinzu, ohne
vom Betreiber bereitgestellte Einträge zu ersetzen. Codex stellt diese Tools daher in
normalen und ausschließlich für den Code Mode vorgesehenen Threads als `DirectModelOnly` bereit, statt sie
durch verschachtelte Code-Mode-Aufrufe von `tools.*` zu leiten. Diese Abgrenzung ist für
Ergebnisse mit Bildern erforderlich: Die verschachtelte Code-Mode-Serialisierung reduziert die Bildausgabe auf
Text, wodurch der für die nächste Computeraktion benötigte Screenshot verloren ginge.

Setzen Sie `codexDynamicToolsLoading: "direct"` nur, wenn Sie eine Verbindung zu einem benutzerdefinierten
Codex-App-Server herstellen, der zurückgestellte dynamische Tools nicht durchsuchen kann, oder wenn Sie
die vollständige Tool-Nutzlast debuggen.

## Zeitüberschreitungen

Dynamische Tool-Aufrufe im Besitz von OpenClaw werden unabhängig von
`appServer.requestTimeoutMs` begrenzt. Jede Codex-Anfrage `item/tool/call` verwendet die
erste verfügbare Zeitüberschreitung in dieser Reihenfolge:

- Ein positives aufrufspezifisches Argument `timeoutMs`.
- Für `image_generate`: `agents.defaults.mediaModels.image.timeoutMs`.
- Für `image_generate` ohne konfigurierte Zeitüberschreitung: der Standardwert von 120 Sekunden
  für die Bilderzeugung.
- Für das Medienanalyse-Tool `image`: der Wert `timeoutSeconds`
  des ausgewählten bildfähigen Eintrags `tools.media.models[]`, in Millisekunden umgerechnet,
  oder der Medienstandardwert von 60 Sekunden. Bei der Bildanalyse gilt dies für die Anfrage
  selbst und wird nicht durch vorangegangene Vorbereitungsarbeiten reduziert.
- Für das Tool `message`: ein festes äußeres Budget von 600 Sekunden, das die Gateway-Zustellung und die begrenzte Abstimmung mit demselben Schlüssel abdeckt.
- Der Standardwert von 90 Sekunden für dynamische Tools.

Dieser Watchdog ist das äußere dynamische Budget für `item/tool/call`. Providerspezifische
Anfragezeitüberschreitungen laufen innerhalb dieses Aufrufs und behalten ihre eigene Zeitüberschreitungssemantik.
Budgets für dynamische Tools sind auf 600000 ms begrenzt. `agents_wait` fügt 30000 ms
äußeren Spielraum für den Abschluss hinzu, und der App-Server-Client erlaubt 660000 ms, damit
das strukturierte Warteergebnis Codex erreichen kann. Bei einer Zeitüberschreitung bricht OpenClaw das Tool-Signal
nach Möglichkeit ab und gibt eine fehlgeschlagene Antwort des dynamischen Tools an Codex zurück, sodass
der Turn fortgesetzt werden kann, statt die Sitzung in `processing` zu belassen.

Nachdem Codex einen Turn akzeptiert hat und nachdem OpenClaw auf eine Turn-bezogene
App-Server-Anfrage geantwortet hat, erwartet das Harness, dass Codex im aktuellen Turn Fortschritte erzielt
und den nativen Turn schließlich mit `turn/completed` abschließt. Wenn der
App-Server für `appServer.turnCompletionIdleTimeoutMs` inaktiv bleibt, unterbricht OpenClaw
den Codex-Turn nach bestem Bemühen, zeichnet eine diagnostische Zeitüberschreitung auf und
gibt die OpenClaw-Sitzungsspur frei, damit nachfolgende Chatnachrichten nicht
hinter einem veralteten nativen Turn eingereiht werden.

Die meisten nicht abschließenden Benachrichtigungen für denselben Turn deaktivieren diesen kurzen Watchdog,
da Codex damit nachgewiesen hat, dass der Turn noch aktiv ist. Tool-Übergaben verwenden ein längeres
Inaktivitätsbudget nach einem Tool: nachdem OpenClaw eine Antwort `item/tool/call` zurückgibt,
nachdem native Tool-Elemente wie `commandExecution` abgeschlossen sind, nach rohen
Abschlüssen von `custom_tool_call_output` sowie nach rohem Assistentenfortschritt,
rohen Reasoning-Abschlüssen oder Reasoning-Fortschritt nach einem Tool. Der Schutzmechanismus verwendet
`appServer.postToolRawAssistantCompletionIdleTimeoutMs`, wenn dies konfiguriert ist, und
standardmäßig andernfalls fünf Minuten. Dasselbe Budget nach einem Tool verlängert außerdem
den Fortschritts-Watchdog für das stille Synthesefenster, bevor Codex das
nächste Ereignis des aktuellen Turns ausgibt. Auf Reasoning-Abschlüsse, Abschlüsse von
Commentary-`agentMessage` sowie rohen Reasoning- oder Assistentenfortschritt vor einem Tool
kann eine automatische endgültige Antwort folgen; daher verwenden sie den Antwortschutz nach Fortschritt,
statt die Sitzungsspur sofort freizugeben. Nur abgeschlossene endgültige bzw. nicht als Commentary
gekennzeichnete `agentMessage`-Elemente und rohe Assistentenabschlüsse vor einem Tool aktivieren die
Freigabe nach Assistentenausgabe: Wenn Codex anschließend ohne `turn/completed` inaktiv bleibt,
unterbricht OpenClaw den nativen Turn nach bestem Bemühen und gibt die Sitzungsspur
frei. Wiederholungssichere stdio-App-Server-Fehler, einschließlich Zeitüberschreitungen bei
inaktivem Turn-Abschluss ohne Hinweise auf Assistenten-, Tool-, aktive Element- oder Nebenwirkungsaktivität,
werden einmal mit einem neuen App-Server-Versuch wiederholt. Unsichere Zeitüberschreitungen setzen den
festgefahrenen App-Server-Client dennoch außer Betrieb und geben die OpenClaw-Sitzungsspur frei. Außerdem
löschen sie die veraltete Bindung des nativen Threads, statt automatisch
wiederholt zu werden. Zeitüberschreitungen der Abschlussüberwachung zeigen Codex-spezifischen Text zur
Zeitüberschreitung an: Wiederholungssichere Fälle weisen darauf hin, dass die Antwort unvollständig sein könnte,
während unsichere Fälle den Benutzer auffordern, vor einem erneuten Versuch den aktuellen Zustand zu prüfen.
Öffentliche Zeitüberschreitungsdiagnosen enthalten strukturelle Felder wie die Methode der letzten
App-Server-Benachrichtigung, ID/Typ/Rolle des rohen Assistentenantwortelements, die Anzahl aktiver
Anfragen und Elemente sowie den aktivierten Überwachungsstatus. Wenn die letzte Benachrichtigung ein
rohes Assistentenantwortelement ist, enthalten sie außerdem eine begrenzte Vorschau des Assistententextes.
Sie enthalten weder rohe Prompt- noch Tool-Inhalte.

## Modellerkennung

Standardmäßig fragt das Codex-Plugin den App-Server nach verfügbaren Modellen. Die
Modellverfügbarkeit liegt in der Verantwortung des Codex-App-Servers, daher kann sich die Liste ändern, wenn
OpenClaw die gebündelte Version `@openai/codex` aktualisiert oder wenn eine Bereitstellung
`appServer.command` auf eine andere Codex-Binärdatei verweist. Die Verfügbarkeit kann auch
kontospezifisch sein. Verwenden Sie `/codex models` auf einem laufenden Gateway, um den aktuellen
Katalog für dieses Harness und Konto anzuzeigen.

Wenn die Erkennung fehlschlägt oder eine Zeitüberschreitung auftritt, verwendet OpenClaw einen gebündelten Ersatzkatalog:

| Modell-ID       | Anzeigename | Reasoning-Stufen         |
| --------------- | ----------- | ------------------------ |
| `gpt-5.5`      | gpt-5.5      | low, medium, high, xhigh |
| `gpt-5.4-mini` | GPT-5.4-Mini | low, medium, high, xhigh |

<Note>
Das derzeit gebündelte Harness ist `@openai/codex` `0.145.0`. Eine Abfrage mit `model/list`
gegen diesen gebündelten App-Server lieferte die folgenden öffentlichen Auswahlzeilen:

| Modell-ID        | Eingabemodalitäten | Reasoning-Stufen                     |
| ---------------- | ------------------ | ------------------------------------ |
| `gpt-5.6-sol`   | Text, Bild         | low, medium, high, xhigh, max, ultra |
| `gpt-5.6-terra` | Text, Bild         | low, medium, high, xhigh, max, ultra |
| `gpt-5.6-luna`  | Text, Bild         | low, medium, high, xhigh, max        |
| `gpt-5.5`       | Text, Bild         | low, medium, high, xhigh             |
| `gpt-5.2`       | Text, Bild         | low, medium, high, xhigh             |

Der App-Server-Katalog kann `ultra` melden; die Reasoning-Steuerung von OpenClaw stellt derzeit
Stufen bis `max` bereit.

Aktuelle Auswahlzeilen sind kontospezifisch und können sich je nach Konto, Codex-Katalog
oder gebündelter Version ändern; führen Sie `/codex models` aus, um die aktuelle Liste zu erhalten,
statt sich auf eine Momentaufnahme zu verlassen. Ausgeblendete Modelle können ebenfalls im
App-Server-Katalog für interne oder spezialisierte Abläufe erscheinen, ohne normale
Optionen der Modellauswahl zu sein.
</Note>

Passen Sie die Erkennung unter `plugins.entries.codex.config.discovery` an:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          discovery: {
            enabled: true,
            timeoutMs: 2500,
          },
        },
      },
    },
  },
}
```

Deaktivieren Sie die Erkennung, wenn beim Start keine Codex-Abfrage erfolgen und ausschließlich
der Ersatzkatalog verwendet werden soll:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          discovery: {
            enabled: false,
          },
        },
      },
    },
  },
}
```

## Bootstrap-Dateien des Arbeitsbereichs

Codex verarbeitet `AGENTS.md` selbst über die native Erkennung von Projektdokumenten.
OpenClaw schreibt keine synthetischen Codex-Projektdokumentdateien und ist für Persona-Dateien
nicht von Codex-Ersatzdateinamen abhängig, da Codex-Ersatzdateien nur gelten, wenn
`AGENTS.md` fehlt.

Für die Übereinstimmung mit dem OpenClaw-Arbeitsbereich leitet das Codex-Harness die anderen
Bootstrap-Dateien als Entwickleranweisungen weiter, jedoch nicht auf identische Weise:

- `TOOLS.md` wird als **vererbte** Codex-Entwickleranweisung weitergeleitet, sodass
  native Codex-Subagents, die während des Turns gestartet werden, sie ebenfalls sehen.
- `SOUL.md`, `IDENTITY.md` und `USER.md` werden als **Turn-bezogene**
  Zusammenarbeitsanweisungen weitergeleitet. Native Codex-Subagents erben sie nicht,
  wodurch verhindert wird, dass Subagent-Turns die Persona und das
  Benutzerprofil des übergeordneten Agents übernehmen.
- Die kompakte Liste geladener OpenClaw-Skills wird ebenfalls als Turn-bezogene
  Entwickleranweisung zur Zusammenarbeit weitergeleitet, sodass native Codex-Subagents
  sie ebenfalls nicht erben.
- Der Inhalt von `HEARTBEAT.md` wird nicht eingefügt; Heartbeat-Turns erhalten einen
  Hinweis im Zusammenarbeitsmodus, die Datei zu lesen, wenn sie vorhanden und
  nicht leer ist.
- Der Inhalt von `MEMORY.md` aus dem konfigurierten Agent-Arbeitsbereich wird nicht in
  die native Codex-Turn-Eingabe eingefügt, wenn für diesen
  Arbeitsbereich Speicher-Tools verfügbar sind; wenn die Datei vorhanden ist, fügt das Harness den Turn-bezogenen
  Entwickleranweisungen zur Zusammenarbeit einen kurzen Hinweis auf den Arbeitsbereichsspeicher hinzu, und Codex
  sollte `memory_search` oder `memory_get` verwenden, wenn dauerhafter Speicher relevant ist.
  Wenn Tools deaktiviert sind, die Speichersuche nicht verfügbar ist oder der aktive
  Arbeitsbereich vom Agent-Speicherarbeitsbereich abweicht, verwendet `MEMORY.md` stattdessen den
  normalen begrenzten Turn-Kontextpfad.
- `BOOTSTRAP.md` wird, sofern vorhanden, als Referenzkontext für die OpenClaw-Turn-Eingabe
  weitergeleitet.

## Umgebungsüberschreibungen

Umgebungsüberschreibungen bleiben für lokale Tests verfügbar:

- `OPENCLAW_CODEX_APP_SERVER_BIN`
- `OPENCLAW_CODEX_APP_SERVER_ARGS`
- `OPENCLAW_CODEX_APP_SERVER_MODE=yolo|guardian`
- `OPENCLAW_CODEX_APP_SERVER_APPROVAL_POLICY`
- `OPENCLAW_CODEX_APP_SERVER_SANDBOX`

`OPENCLAW_CODEX_APP_SERVER_BIN` umgeht die verwaltete Binärdatei, wenn
`appServer.command` nicht gesetzt ist.

`OPENCLAW_CODEX_APP_SERVER_GUARDIAN=1` wurde entfernt. Verwenden Sie stattdessen
`plugins.entries.codex.config.appServer.mode: "guardian"` oder
`OPENCLAW_CODEX_APP_SERVER_MODE=guardian` für einmalige lokale Tests. Für
reproduzierbare Bereitstellungen wird die Konfiguration bevorzugt, da das Plugin-Verhalten dadurch in
derselben geprüften Datei wie die übrige Einrichtung des Codex-Harness verbleibt.

## Verwandte Themen

- [Codex-Harness](/de/plugins/codex-harness)
- [Codex-Harness-Laufzeit](/de/plugins/codex-harness-runtime)
- [Codex-Überwachung](/de/plugins/codex-supervision)
- [Native Codex-Plugins](/de/plugins/codex-native-plugins)
- [Codex-Computernutzung](/de/plugins/codex-computer-use)
- [OpenAI-Provider](/de/providers/openai)
- [Konfigurationsreferenz](/de/gateway/configuration-reference)
