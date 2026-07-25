---
read_when:
    - Sie möchten, dass OpenClaw-Agenten im Codex-Modus Codex Computer Use verwenden
    - Sie entscheiden sich zwischen Codex Computer Use, PeekabooBridge und direktem cua-driver-MCP.
    - Sie konfigurieren computerUse für das gebündelte Codex-Plugin
    - Sie beheben Probleme mit dem Status oder der Installation von /codex computer-use
summary: Codex Computer Use für OpenClaw-Agenten im Codex-Modus einrichten
title: Codex-Computernutzung
x-i18n:
    generated_at: "2026-07-24T21:12:29Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 6b11d00c74bc2990a4e33b6ffe23209ed76a1e10180ce5950dbb5073ea57ad05
    source_path: plugins/codex-computer-use.md
    workflow: 16
---

Computer Use ist ein Codex-natives MCP-Plugin zur lokalen Desktop-Steuerung. OpenClaw
liefert die Desktop-App nicht mit, führt Desktop-Aktionen nicht selbst aus und umgeht
keine Codex-Berechtigungen. Das gebündelte `codex`-Plugin bereitet lediglich den Codex app-server vor:
Es aktiviert die Codex-Plugin-Unterstützung, findet oder installiert das konfigurierte Computer Use-
Plugin, prüft, ob der `computer-use`-MCP-Server verfügbar ist, und überlässt
Codex anschließend während Durchläufen im Codex-Modus die nativen MCP-Tool-Aufrufe.

Verwenden Sie diese Seite, wenn OpenClaw bereits das native Codex-Harness verwendet. Informationen zur
Einrichtung der Runtime selbst finden Sie unter [Codex-Harness](/de/plugins/codex-harness).

Dies unterscheidet sich vom integrierten [Node-gestützten Computer-Tool](/de/nodes/computer-use) von OpenClaw. Verwenden Sie das integrierte Tool, wenn derselbe Agent-Vertrag einen gekoppelten Mac steuern soll, unabhängig davon, ob der Agent auf dem Gateway oder einem anderen Node ausgeführt wird. Verwenden Sie Codex Computer Use, wenn Codex app-server die lokale MCP-Installation, Berechtigungen und nativen Tool-Aufrufe verwalten soll.

## OpenClaw.app und Peekaboo

Die Peekaboo-Integration von OpenClaw.app ist von Codex Computer Use getrennt. Die
macOS-App kann einen PeekabooBridge-Socket bereitstellen, damit die `peekaboo`-CLI die
lokalen Berechtigungen der App für Bedienungshilfen und Bildschirmaufnahme für Peekaboos eigene
Automatisierungstools wiederverwenden kann. Diese Bridge installiert oder vermittelt Codex Computer Use nicht, und
Codex Computer Use ruft den PeekabooBridge-Socket nicht auf.

Verwenden Sie die [Peekaboo-Bridge](/de/platforms/mac/peekaboo), wenn OpenClaw.app als
berechtigungsbewusster Host für die Peekaboo-CLI-Automatisierung dienen soll. Verwenden Sie diese Seite, wenn einem
OpenClaw-Agenten im Codex-Modus das native `computer-use`-MCP-Plugin von Codex
vor Beginn des Durchlaufs zur Verfügung stehen soll.

## iOS-App

Die iOS-App ist von Codex Computer Use getrennt. Sie installiert oder vermittelt
den Codex-`computer-use`-MCP-Server nicht und ist kein Backend zur Desktop-Steuerung.
Stattdessen verbindet sich die iOS-App als OpenClaw-Node und stellt mobile
Funktionen über Node-Befehle wie `canvas.*`, `camera.*`, `screen.*`,
`location.*` und `talk.*` bereit.

Verwenden Sie [iOS](/de/platforms/ios), wenn ein Agent einen iPhone-Node
über das Gateway steuern soll. Verwenden Sie diese Seite, wenn ein Agent im Codex-Modus den
lokalen macOS-Desktop über das native Computer Use-Plugin von Codex steuern soll.

## Direkter cua-driver-MCP

Codex Computer Use ist nicht die einzige Möglichkeit, Desktop-Steuerung bereitzustellen. Wenn
von OpenClaw verwaltete Runtimes den Treiber von TryCua direkt aufrufen sollen, verwenden Sie den vorgelagerten
`cua-driver mcp`-Server über die MCP-Registry von OpenClaw anstelle des
Codex-spezifischen Marketplace-Ablaufs.

Bitten Sie `cua-driver` nach der Installation entweder um den OpenClaw-Befehl:

```bash
cua-driver mcp-config --client openclaw
```

oder registrieren Sie den stdio-Server direkt:

```bash
openclaw mcp set cua-driver '{"command":"cua-driver","args":["mcp"]}'
```

Dieser Pfad erhält die vorgelagerte MCP-Tool-Oberfläche vollständig, einschließlich der Treiber-
Schemas und strukturierten MCP-Antworten. Verwenden Sie ihn, wenn der CUA-Treiber
als normaler OpenClaw-MCP-Server verfügbar sein soll. Verwenden Sie die Einrichtung von Codex Computer Use auf
dieser Seite, wenn Codex app-server die Plugin-Installation, MCP-Neuladungen
und nativen Tool-Aufrufe innerhalb von Durchläufen im Codex-Modus verwalten soll.

Der Treiber von CUA liefert Vorabversionen für macOS, Windows (x64 und ARM64) sowie
Linux (x64 und ARM64, Vorschau-Stufe). Er erfordert weiterhin die lokalen Betriebssystem-
berechtigungen, zu deren Erteilung seine App auffordert, beispielsweise Bedienungshilfen und Bildschirmaufnahme unter
macOS. OpenClaw installiert `cua-driver` nicht, erteilt diese Berechtigungen nicht und
umgeht das Sicherheitsmodell des vorgelagerten Treibers nicht.

## Schnelleinrichtung

Legen Sie `plugins.entries.codex.config.computerUse` fest, wenn für Durchläufe im Codex-Modus
Computer Use vor dem Start eines Threads verfügbar sein muss. `autoInstall: true` aktiviert
Computer Use und ermöglicht OpenClaw, es vor dem Durchlauf zu installieren oder erneut zu aktivieren:

```json5
{
  plugins: {
    entries: {
      codex: {
        enabled: true,
        config: {
          computerUse: {
            autoInstall: true,
          },
        },
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

Mit dieser Konfiguration prüft OpenClaw Codex app-server vor jedem Durchlauf im Codex-
Modus. Wenn Computer Use fehlt, Codex app-server aber bereits
einen installierbaren Marketplace gefunden hat, fordert OpenClaw Codex app-server auf, das Plugin zu installieren oder
erneut zu aktivieren und die MCP-Server neu zu laden. Vor dem Start eines isolierten
Codex app-server unter macOS kopiert die automatische Installation außerdem die offizielle signierte
Computer Use-Dienst-App aus dem ausgewählten Desktop-App-Bundle in das
`computer-use`-Verzeichnis dieses Codex-Homes, wenn der native Client fehlt.
Wenn unter macOS kein passender
Marketplace registriert ist und ein standardmäßiges Desktop-App-Bundle vorhanden ist, versucht OpenClaw
außerdem, den gebündelten Codex-Marketplace aus
`/Applications/ChatGPT.app/Contents/Resources/plugins/openai-bundled` zu registrieren, wobei
`/Applications/Codex.app/Contents/Resources/plugins/openai-bundled` als
Fallback für ältere eigenständige Installationen beibehalten wird. Wenn die Einrichtung den
MCP-Server weiterhin nicht verfügbar machen kann, schlägt der Durchlauf vor dem Start des Threads fehl.
Strikte Bereitschaftsfehler sind Fehler der Harness-Vorabprüfung, sodass der Modell-Fallback
dieselbe lokale Bereitschaftssequenz nicht für jeden Modellkandidaten wiederholt.

Verwenden Sie nach einer Änderung der Computer Use-Konfiguration `/new` oder `/reset` im betroffenen
Chat, bevor Sie testen, falls bereits ein vorhandener Codex-Thread gestartet wurde.

Unter macOS bevorzugt der verwaltete Start für Computer Use die Binärdatei der Desktop-App unter
`/Applications/ChatGPT.app/Contents/Resources/codex` und greift anschließend
für ältere eigenständige Installationen auf `/Applications/Codex.app/Contents/Resources/codex` zurück.
Dies gilt auch für einmalige Computer Use-Status- und
Installationsbefehle, die einen eigenen Client starten. Dadurch bleibt die Desktop-Steuerung unter
dem App-Bundle, das die lokalen macOS-Berechtigungen besitzt. Wenn die Desktop-App nicht
installiert ist, greift OpenClaw auf die verwaltete Codex-Binärdatei zurück, die neben dem
Plugin installiert ist. Normale verwaltete Codex-Durchläufe mit dem standardmäßigen isolierten Agent-Home bevorzugen
zunächst dieses angeheftete Paket, damit eine ältere Desktop-App die aktuelle Modell-
unterstützung nicht überschreiben kann. Benutzerbezogene Homes bleiben Desktop-zuerst, da sie nativen
Computer Use-Zustand laden können. Ein isoliertes Agent-Home, dessen wirksame Codex-Konfiguration
Computer Use aktiviert, bleibt ebenfalls Desktop-zuerst. Eine explizite
`appServer.command`-Konfiguration oder `OPENCLAW_CODEX_APP_SERVER_BIN` setzt sich weiterhin über
diese verwaltete Auswahl hinweg.

OpenClaw serialisiert native Codex-Konfigurationslesevorgänge und die Installation von Computer Use
innerhalb eines laufenden Gateways. Ein separater Codex-Prozess oder ein anderes Gateway ist
nicht Teil dieser Sperre. Starten Sie nach einer Änderung der nativen Codex-Plugin-Konfiguration außerhalb des
Gateways das Gateway neu und beginnen Sie einen neuen Chat, bevor Sie sich auf die neue
Auswahl verlassen.

## Befehle

Verwenden Sie die `/codex computer-use`-Befehle auf jeder Chat-Oberfläche, auf der die
Befehlsoberfläche des `codex`-Plugins verfügbar ist. Dies sind OpenClaw-Chat-/Runtime-
Befehle, keine `openclaw codex ...`-CLI-Unterbefehle:

```text
/codex computer-use status
/codex computer-use install
/codex computer-use install --source <marketplace-source>
/codex computer-use install --marketplace-path <path>
/codex computer-use install --marketplace <name>
```

`status` ist die Standardaktion und schreibgeschützt: Sie fügt keine Marketplace-
Quellen hinzu, installiert keine Plugins und aktiviert keine Codex-Plugin-Unterstützung. Wenn keine Konfiguration
Computer Use aktiviert, kann `status` auch nach einem einmaligen Installations-
befehl „deaktiviert“ melden.

`install` aktiviert die Plugin-Unterstützung von Codex app-server, fügt optional eine
konfigurierte Marketplace-Quelle hinzu, installiert oder reaktiviert das konfigurierte Plugin
über Codex app-server, lädt MCP-Server neu und überprüft, ob der MCP-
Server Tools bereitstellt. Da die Installation vertrauenswürdige Host-Ressourcen ändert,
kann nur ein Eigentümer oder ein `operator.admin`-Gateway-Client `install` ausführen. Andere
autorisierte Absender können weiterhin den schreibgeschützten `status`-Befehl verwenden,
auch mit Überschreibungen.

Ältere Versionen akzeptierten einmalige Identitätsüberschreibungen über `--plugin`, `--server` und `--mcp-server`.
Konfigurieren Sie stattdessen `computerUse.pluginName` und
`computerUse.mcpServerName` dauerhaft. Wenn ein älteres Identitäts-Flag
verwendet wird, benennt der Befehl die genaue Einstellung, die dauerhaft gespeichert werden muss, und wiederholt die
angeforderte Aktion sowie alle unterstützten Marketplace-Flags in seiner Migrationsanleitung.

## Marketplace-Auswahl

OpenClaw verwendet dieselbe app-server-API, die Codex selbst bereitstellt. Die
Marketplace-Felder bestimmen, wo Codex `computer-use` finden soll.

| Feld                 | Verwenden, wenn                                                   | Installationsunterstützung                                    |
| -------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------- |
| Kein Marketplace-Feld | Codex app-server soll bereits bekannte Marketplaces verwenden.   | Ja, wenn app-server einen lokalen Marketplace zurückgibt.     |
| `marketplaceSource`  | Sie haben eine Codex-Marketplace-Quelle, die app-server hinzufügen kann. | Ja, für explizites `/codex computer-use install`.             |
| `marketplacePath`    | Sie kennen bereits den lokalen Pfad zur Marketplace-Datei auf dem Host. | Ja, für explizite Installation und automatische Installation beim Durchlaufstart. |
| `marketplaceName`    | Sie möchten einen bereits registrierten Marketplace anhand seines Namens auswählen. | Nur, wenn der ausgewählte Marketplace einen lokalen Pfad hat. |

Neue Codex-Homes benötigen möglicherweise einen kurzen Moment, um ihre offiziellen
Marketplaces einzurichten. Während der Installation fragt OpenClaw `plugin/list` bis zu
`marketplaceDiscoveryTimeoutMs` Millisekunden lang ab (Standard: 60 Sekunden).

Wenn mehrere bekannte Marketplaces Computer Use enthalten, bevorzugt OpenClaw
`openai-bundled`, dann `openai-curated` und anschließend `local`. Unbekannte mehrdeutige
Übereinstimmungen führen zu einem sicheren Abbruch und fordern Sie auf, `marketplaceName` oder
`marketplacePath` festzulegen.

## Gebündelter macOS-Marketplace

Aktuelle ChatGPT-Desktop-Builds bündeln Computer Use hier; ältere eigenständige
Codex-Desktop-Builds verwenden dieselbe Struktur unter `Codex.app`:

```text
/Applications/ChatGPT.app/Contents/Resources/plugins/openai-bundled/plugins/computer-use
/Applications/Codex.app/Contents/Resources/plugins/openai-bundled/plugins/computer-use
```

Wenn `computerUse.autoInstall` auf „true“ gesetzt ist und kein Marketplace mit
`computer-use` registriert ist, versucht OpenClaw, den ersten vorhandenen standardmäßigen
gebündelten Marketplace-Stamm hinzuzufügen:

```text
/Applications/ChatGPT.app/Contents/Resources/plugins/openai-bundled
/Applications/Codex.app/Contents/Resources/plugins/openai-bundled
```

Sie können ihn auch explizit über eine Shell mit Codex registrieren:

```bash
codex plugin marketplace add /Applications/ChatGPT.app/Contents/Resources/plugins/openai-bundled
```

Wenn Sie einen nicht standardmäßigen Codex-App-Pfad verwenden, führen Sie `/codex computer-use install
--source <marketplace-root>` einmal aus oder legen Sie `computerUse.marketplacePath` auf einen
lokalen Pfad zu einer Marketplace-Datei fest. Verwenden Sie `--marketplace-path` nur, wenn Ihnen der
Pfad zur Marketplace-JSON-Datei vorliegt, nicht der Stamm des gebündelten Marketplace.

### Gemeinsamer Plugin-Cache

Der Standardwert `pluginCacheMode: "independent"` lässt jedes Codex-Home und dessen
Plugin-Cache von OpenClaw unverwaltet. Legen Sie `pluginCacheMode: "shared"` fest, um das gebündelte
Computer Use-Plugin vor dem Start von app-server in den auffindbaren Plugin-Cache
des aktiven Codex-Homes zu kopieren. Der gemeinsame Modus behält ältere zwischengespeicherte Versionen bei, da
laufende Codex-Clients weiterhin auf ihre versionierten Plugin-Verzeichnisse verweisen können; auch bei einem
fehlgeschlagenen Ersetzungskopiervorgang bleibt der aktive Cache erhalten. Eine explizite
`marketplaceName`- oder `marketplacePath`-Konfiguration deaktiviert diesen
Abgleich, damit OpenClaw diese Auswahl nicht überschreibt.

## Einschränkung des Remote-Katalogs

Codex app-server kann ausschließlich remote verfügbare Katalogeinträge auflisten und lesen, unterstützt
derzeit jedoch kein Remote-`plugin/install`. Das bedeutet, dass `marketplaceName`
für Statusprüfungen einen ausschließlich remote verfügbaren Marketplace auswählen kann, Installationen und
Reaktivierungen jedoch weiterhin einen lokalen Marketplace über `marketplaceSource` oder
`marketplacePath` benötigen.

Wenn der Status meldet, dass das Plugin in einem Remote-Codex-Marketplace verfügbar ist, die
Remote-Installation jedoch nicht unterstützt wird, führen Sie die Installation mit einer lokalen Quelle oder einem lokalen Pfad aus:

```text
/codex computer-use install --source <marketplace-source>
/codex computer-use install --marketplace-path <path>
```

## Konfigurationsreferenz

| Feld                            | Standardwert   | Bedeutung                                                                                 |
| ------------------------------- | -------------- | ----------------------------------------------------------------------------------------- |
| `enabled`                       | abgeleitet     | Computer Use ist erforderlich. Der Standardwert ist „true“, wenn ein anderes Computer-Use-Feld festgelegt ist. |
| `autoInstall`                   | false          | Stellt den nativen Client bereit und installiert oder reaktiviert das Plugin zu Beginn des Durchlaufs. |
| `marketplaceDiscoveryTimeoutMs` | 60000          | Gibt an, wie lange die Installation auf die Marketplace-Erkennung des Codex-App-Servers wartet. |
| `liveTestTimeoutMs`             | 60000          | Zeitüberschreitung für den temporären Bereitschafts-Thread und seine Bereinigungsanfragen. |
| `toolCallTimeoutMs`             | 60000          | Zeitüberschreitung für den Aufruf des Computer-Use-Bereitschaftstools `list_apps`. |
| `healthCheckEnabled`            | false          | Führt regelmäßige Bereitschaftsprüfungen aus, während der zuständige App-Server-Client aktiv ist. |
| `healthCheckIntervalMinutes`    | 60             | Prüfintervall; zulässige Werte sind 30, 60, 120 oder 240 Minuten.                          |
| `pluginCacheMode`               | `independent`  | Verwendet `shared`, um den Codex-Home-Cache über das gebündelte Desktop-Plugin zu aktualisieren. |
| `strictReadiness`               | false          | Bricht den Start bei einer fehlgeschlagenen Live-Prüfung ab, statt mit einer Warnung fortzufahren. |
| `autoRepair`                    | false          | Beendet veraltete, bereichsgebundene untergeordnete Computer-Use-MCP-Prozesse und wiederholt eine fehlgeschlagene Prüfung einmal. |
| `marketplaceSource`             | nicht festgelegt | An den Codex-App-Server `marketplace/add` übergebene Quellzeichenfolge.                    |
| `marketplacePath`               | nicht festgelegt | Lokaler Dateipfad des Codex-Marketplace, der das Plugin enthält.                           |
| `marketplaceName`               | nicht festgelegt | Name des registrierten Codex-Marketplace, der ausgewählt werden soll.                      |
| `pluginName`                    | `computer-use` | Plugin-Name im Codex-Marketplace.                                                          |
| `mcpServerName`                 | `computer-use` | Name des MCP-Servers, den das installierte Plugin bereitstellt.                            |

Die automatische Installation zu Beginn des Durchlaufs lehnt konfigurierte `marketplaceSource`-Werte
absichtlich ab. Das Hinzufügen einer neuen Quelle ist ein expliziter Einrichtungsvorgang. Verwenden Sie daher
einmal `/codex computer-use install --source <marketplace-source>` und lassen Sie anschließend
`autoInstall` zukünftige Reaktivierungen aus erkannten lokalen Marketplaces durchführen.
Die automatische Installation zu Beginn des Durchlaufs kann einen konfigurierten `marketplacePath` verwenden, da dies
bereits ein lokaler Pfad auf dem Host ist.

Jedes Feld akzeptiert außerdem eine Überschreibung durch eine Umgebungsvariable, die geprüft wird, wenn der
entsprechende Konfigurationsschlüssel nicht festgelegt ist:

| Feld                            | Umgebungsvariable                                               |
| ------------------------------- | --------------------------------------------------------------- |
| `enabled`                       | `OPENCLAW_CODEX_COMPUTER_USE`                                  |
| `autoInstall`                   | `OPENCLAW_CODEX_COMPUTER_USE_AUTO_INSTALL`                     |
| `marketplaceDiscoveryTimeoutMs` | `OPENCLAW_CODEX_COMPUTER_USE_MARKETPLACE_DISCOVERY_TIMEOUT_MS` |
| `liveTestTimeoutMs`             | `OPENCLAW_CODEX_COMPUTER_USE_LIVE_TEST_TIMEOUT_MS`             |
| `toolCallTimeoutMs`             | `OPENCLAW_CODEX_COMPUTER_USE_TOOL_CALL_TIMEOUT_MS`             |
| `healthCheckEnabled`            | `OPENCLAW_CODEX_COMPUTER_USE_HEALTH_CHECK_ENABLED`             |
| `healthCheckIntervalMinutes`    | `OPENCLAW_CODEX_COMPUTER_USE_HEALTH_CHECK_INTERVAL_MINUTES`    |
| `pluginCacheMode`               | `OPENCLAW_CODEX_COMPUTER_USE_PLUGIN_CACHE_MODE`                |
| `strictReadiness`               | `OPENCLAW_CODEX_COMPUTER_USE_STRICT_READINESS`                 |
| `autoRepair`                    | `OPENCLAW_CODEX_COMPUTER_USE_AUTO_REPAIR`                      |
| `marketplaceSource`             | `OPENCLAW_CODEX_COMPUTER_USE_MARKETPLACE_SOURCE`               |
| `marketplacePath`               | `OPENCLAW_CODEX_COMPUTER_USE_MARKETPLACE_PATH`                 |
| `marketplaceName`               | `OPENCLAW_CODEX_COMPUTER_USE_MARKETPLACE_NAME`                 |
| `pluginName`                    | `OPENCLAW_CODEX_COMPUTER_USE_PLUGIN_NAME`                      |
| `mcpServerName`                 | `OPENCLAW_CODEX_COMPUTER_USE_MCP_SERVER_NAME`                  |

## Was OpenClaw prüft

OpenClaw meldet intern einen stabilen Einrichtungsgrund und formatiert den
benutzerseitigen Status für den Chat:

| Grund                        | Bedeutung                                              | Nächster Schritt                                      |
| ---------------------------- | ------------------------------------------------------ | ----------------------------------------------------- |
| `disabled`                   | `computerUse.enabled` wurde als „false“ aufgelöst.     | Legen Sie `enabled` oder ein anderes Computer-Use-Feld fest. |
| `marketplace_missing`        | Kein passender Marketplace war verfügbar.              | Konfigurieren Sie Quelle, Pfad oder Marketplace-Namen. |
| `plugin_not_installed`       | Der Marketplace ist vorhanden, das Plugin jedoch nicht installiert. | Führen Sie die Installation aus oder aktivieren Sie `autoInstall`. |
| `plugin_disabled`            | Das Plugin ist installiert, aber in der Codex-Konfiguration deaktiviert. | Führen Sie die Installation aus, um es zu reaktivieren. |
| `remote_install_unsupported` | Der ausgewählte Marketplace ist ausschließlich remote verfügbar. | Verwenden Sie `marketplaceSource` oder `marketplacePath`. |
| `mcp_missing`                | Das Plugin ist aktiviert, aber der MCP-Server ist nicht verfügbar. | Prüfen Sie Codex Computer Use und die Betriebssystemberechtigungen. |
| `ready`                      | Das Plugin und die MCP-Tools sind verfügbar.           | Starten Sie den Durchlauf im Codex-Modus.             |
| `check_failed`               | Eine Anfrage an den Codex-App-Server ist während der Statusprüfung fehlgeschlagen. | Prüfen Sie die App-Server-Verbindung und die Protokolle. |
| `auto_install_blocked`       | Die Einrichtung zu Beginn des Durchlaufs müsste eine neue Quelle hinzufügen. | Führen Sie zuerst eine explizite Installation aus.    |

Die Chat-Ausgabe enthält den Plugin-Status, den MCP-Server-Status, den Marketplace,
die verfügbaren Tools und die spezifische Meldung für den fehlgeschlagenen Einrichtungsschritt.

## macOS-Berechtigungen

Dieser Codex-eigene Computer-Use-Pfad wird unter macOS ausgeführt, wo der MCP-Server möglicherweise
lokale Betriebssystemberechtigungen benötigt, bevor er Apps untersuchen oder steuern kann. (Informationen zur plattformübergreifenden
Desktop-Steuerung auf Windows- und Linux-Node-Hosts finden Sie unter
[cua-computer fulfiller](/de/nodes/computer-use#windows-and-linux-experimental-via-cua-driver).)
Wenn OpenClaw meldet, dass Computer Use installiert ist, der MCP-Server jedoch nicht verfügbar ist,
überprüfen Sie zuerst die Computer-Use-Einrichtung auf Codex-Seite:

- Der Codex-App-Server wird auf demselben Host ausgeführt, auf dem die Desktop-Steuerung
  erfolgen soll.
- Das Computer-Use-Plugin ist in der Codex-Konfiguration aktiviert.
- Der MCP-Server `computer-use` erscheint im MCP-Status des Codex-App-Servers.
- macOS hat der Desktop-Steuerungs-App die erforderlichen Berechtigungen erteilt.
- Die aktuelle Host-Sitzung kann auf den gesteuerten Desktop zugreifen.

OpenClaw bricht absichtlich sicher ab, wenn `computerUse.enabled` „true“ ist. Ein
Durchlauf im Codex-Modus darf nicht unbemerkt ohne die nativen Desktop-Tools fortgesetzt werden,
die von der Konfiguration vorausgesetzt werden.

## Fehlerbehebung

**Der Status meldet, dass das Plugin nicht installiert ist.** Führen Sie `/codex computer-use install` aus. Wenn der
Marketplace nicht erkannt wird, übergeben Sie `--source` oder `--marketplace-path`.

**Der Status meldet, dass das Plugin installiert, aber deaktiviert ist.** Führen Sie `/codex computer-use install`
erneut aus. Die Installation über den Codex-App-Server schreibt die Plugin-Konfiguration wieder als aktiviert.

**Der Status meldet, dass die Remote-Installation nicht unterstützt wird.** Verwenden Sie eine lokale Marketplace-
Quelle oder einen lokalen Pfad. Ausschließlich remote verfügbare Katalogeinträge können untersucht, aber nicht
über die aktuelle App-Server-API installiert werden.

**Der Status meldet, dass der MCP-Server nicht verfügbar ist.** Führen Sie die Installation einmal erneut aus, damit die MCP-
Server neu geladen werden. Falls er weiterhin nicht verfügbar ist, beheben Sie die Probleme mit der Codex-Computer-Use-App,
dem MCP-Status des Codex-App-Servers oder den macOS-Berechtigungen.

**Beim Status oder bei einer Prüfung tritt für `computer-use.list_apps` eine Zeitüberschreitung auf.** Das Plugin und der
MCP-Server sind vorhanden, aber die lokale Computer-Use-Bridge hat nicht geantwortet.
Beenden Sie Codex Computer Use oder starten Sie es neu, starten Sie Codex Desktop bei Bedarf neu und
versuchen Sie es dann in einer neuen OpenClaw-Sitzung erneut. Wenn auf dem Host Computer Use zuvor
über einen älteren verwalteten Codex-App-Server ausgeführt wurde, aktualisieren Sie das installierte Plugin über
den mit der Desktop-App gebündelten Marketplace (verwenden Sie für eigenständige
Codex-Desktop-Installationen den Pfad `Codex.app`):

```text
/codex computer-use install --source /Applications/ChatGPT.app/Contents/Resources/plugins/openai-bundled
```

**Ein Computer-Use-Tool meldet `Native hook relay unavailable`.** Der
Codex-native Tool-Hook konnte über die lokale Bridge oder den Gateway-Fallback kein aktives OpenClaw-Relay erreichen.
Starten Sie eine neue OpenClaw-Sitzung mit `/new`
oder `/reset`. Wenn es einmal funktioniert und bei einem späteren Tool-Aufruf erneut fehlschlägt,
bereinigt `/new` nur den aktuellen Versuch. Starten Sie den Codex-App-Server oder das
OpenClaw Gateway neu, damit alte Threads und Hook-Registrierungen verworfen werden, und
versuchen Sie es anschließend in einer neuen Sitzung erneut.

**Die automatische Installation zu Beginn des Durchlaufs lehnt eine Quelle ab.** Dies ist beabsichtigt. Fügen Sie die
Quelle zuerst explizit mit `/codex computer-use install --source
<marketplace-source>` hinzu. Danach kann die automatische Installation zu Beginn zukünftiger Durchläufe den
erkannten lokalen Marketplace verwenden.

## Verwandte Themen

- [Codex-Harness](/de/plugins/codex-harness)
- [Peekaboo-Bridge](/de/platforms/mac/peekaboo)
- [iOS-App](/de/platforms/ios)
