---
read_when:
    - Exec-Genehmigungen oder Positivlisten konfigurieren
    - Implementierung der UX für Ausführungsgenehmigungen in der macOS-App
    - Überprüfung von Prompts zur Umgehung der Sandbox und ihrer Auswirkungen
sidebarTitle: Exec approvals
summary: 'Host-Ausführungsgenehmigungen: Richtlinienoptionen, Positivlisten und der YOLO-/strikte Workflow'
title: Ausführungsgenehmigungen
x-i18n:
    generated_at: "2026-07-24T15:17:31Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 2bd09746375061232e9094b8803d33859cac4c13c7bde14a059b7d52e48b5de8
    source_path: tools/exec-approvals.md
    workflow: 16
---

Ausführungsgenehmigungen sind die **Schutzvorkehrung der Begleit-App / des Node-Hosts**, mit der ein
Sandbox-Agent Befehle auf einem realen Host ausführen darf (`gateway` oder `node`). Befehle
werden nur ausgeführt, wenn Richtlinie + Zulassungsliste + (optionale) Benutzergenehmigung übereinstimmen.
Genehmigungen gelten **zusätzlich zu** Tool-Richtlinie und Elevated-Prüfung (Elevated
`full` umgeht sie).

Eine nach Modi gegliederte Übersicht über `deny`, `allowlist`, `ask`, `auto`, `full`,
die Zuordnung zu Codex Guardian und ACPX-Harness-Berechtigungen finden Sie unter
[Berechtigungsmodi](/de/tools/permission-modes).

<Note>
Die effektive Richtlinie ist die **strengere** aus `tools.exec.*` und den
Genehmigungsstandards: Genehmigungen können die aus der Konfiguration abgeleiteten Sicherheits- und Abfrageeinstellungen nur
verschärfen, niemals lockern. Wenn ein Genehmigungsfeld ausgelassen wird, wird der Wert
`tools.exec` verwendet. Die Host-Ausführung verwendet außerdem den lokalen Genehmigungsstatus dieses Rechners – ein
hostlokales `ask: "always"` in der Genehmigungsdatei des Ausführungs-Hosts fragt
weiterhin nach, selbst wenn Sitzungs- oder Konfigurationsstandards `ask: "on-miss"` anfordern.
</Note>

## Geltungsbereich

Ausführungsgenehmigungen werden lokal auf dem Ausführungs-Host durchgesetzt:

- **Gateway-Host** -> `openclaw`-Prozess auf dem Gateway-Rechner.
- **Node-Host** -> Node-Runner (macOS-Begleit-App oder monitorloser Node-Host).

### Vertrauensmodell

- Vom Gateway authentifizierte Aufrufer sind vertrauenswürdige Operatoren für dieses Gateway.
- Gekoppelte Nodes erweitern diese vertrauenswürdige Operatorfunktion auf den Node-Host.
- Genehmigungen verringern das Risiko einer versehentlichen Ausführung, sind aber **keine** benutzerspezifische Authentifizierungsgrenze oder schreibgeschützte Dateisystemrichtlinie.
- Nach der Genehmigung kann ein Befehl Dateien gemäß den ausgewählten Dateisystemberechtigungen des Hosts oder der Sandbox verändern.
- Genehmigte Ausführungen auf dem Node-Host binden den kanonischen Ausführungskontext: Arbeitsverzeichnis, exakte Argumentfolge, Umgebungsbindung, sofern vorhanden, und gegebenenfalls den festgelegten Pfad der ausführbaren Datei.
- Bei Shell-Skripten und direkten Datei-Aufrufen über Interpreter oder Laufzeiten versucht OpenClaw außerdem, einen konkreten lokalen Dateioperanden zu binden. Ändert sich diese Datei nach der Genehmigung, aber vor der Ausführung, wird die Ausführung abgelehnt, statt den veränderten Inhalt auszuführen.
- Die Dateibindung erfolgt nach bestem Bemühen und bildet nicht jeden möglichen Ladepfad eines Interpreters oder einer Laufzeit vollständig ab. Wenn nicht genau eine konkrete lokale Datei identifiziert werden kann, verweigert OpenClaw die Ausstellung einer genehmigungsgestützten Ausführung, statt eine vollständige Abdeckung vorzutäuschen.

### macOS-Aufteilung

- Der **Node-Host-Dienst** leitet `system.run` über lokale IPC an die **macOS-App** weiter.
- Die **macOS-App** setzt Genehmigungen durch und führt den Befehl im UI-Kontext aus.

## Effektive Richtlinie prüfen

| Befehl                                                           | Angezeigte Informationen                                                                |
| ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `openclaw approvals get` / `--gateway` / `--node <id\|name\|ip>` | Angeforderte Richtlinie, Quellen der Host-Richtlinie und das effektive Ergebnis.          |
| `openclaw exec-policy show`                                      | Zusammengeführte Ansicht des lokalen Rechners.                                            |
| `openclaw exec-policy set` / `preset`                            | Synchronisiert die lokal angeforderte Richtlinie in einem Schritt mit der lokalen Host-Genehmigungsdatei. |

<Note>
Sitzungsspezifische Überschreibungen von `/exec` sind nicht enthalten. Führen Sie `/exec` in der betreffenden Sitzung aus, um deren aktuelle Standardwerte zu prüfen. Siehe [Sitzungsüberschreibungen](/de/tools/exec#session-overrides-exec).
</Note>

Vollständige CLI-Referenz (Flags, JSON-Ausgabe, Hinzufügen/Entfernen aus der Zulassungsliste): [Genehmigungs-CLI](/de/cli/approvals).

Wenn ein lokaler Geltungsbereich `host=node` anfordert, meldet `exec-policy show` diesen
Geltungsbereich zur Laufzeit als Node-verwaltet, statt die lokale Genehmigungsdatei
als maßgebliche Quelle zu behandeln.

Wenn die Benutzeroberfläche der Begleit-App **nicht verfügbar** ist, wird jede Anfrage, die
normalerweise eine Abfrage auslösen würde, durch den **Abfrage-Rückfall** aufgelöst (Standard: `deny`).

<Tip>
Native Chat-Genehmigungsclients können die ausstehende Genehmigungsnachricht mit
kanalspezifischen Interaktionsmöglichkeiten versehen. Matrix fügt Reaktionskurzbefehle hinzu (`✅` einmal zulassen,
`♾️` immer zulassen, `❌` ablehnen), während `/approve ...` weiterhin als
Rückfall in der Nachricht verbleibt.
</Tip>

## Einstellungen und Speicherung

Genehmigungen befinden sich in einer lokalen JSON-Datei auf dem Ausführungs-Host. Wenn
`OPENCLAW_STATE_DIR` festgelegt ist, folgt die Datei diesem Zustandsverzeichnis;
andernfalls verwendet sie das standardmäßige OpenClaw-Zustandsverzeichnis:

```text
$OPENCLAW_STATE_DIR/exec-approvals.json
# andernfalls
~/.openclaw/exec-approvals.json
```

Der standardmäßige Genehmigungs-Socket folgt demselben Stammverzeichnis:
`$OPENCLAW_STATE_DIR/exec-approvals.sock` oder
`~/.openclaw/exec-approvals.sock`, wenn die Variable nicht gesetzt ist.

Zustandsverzeichnisse sind voneinander unabhängige Vertrauensbereiche. Wenn `OPENCLAW_STATE_DIR`
auf einen anderen Ort verweist, importiert oder archiviert OpenClaw niemals
`~/.openclaw/exec-approvals.json`; konfigurieren Sie Genehmigungen separat für das
benutzerdefinierte Zustandsverzeichnis. Doctor importiert außerdem das veraltete
`plugin-binding-approvals.json` nur, wenn es zum aktiven Zustandsverzeichnis
gehört.

Beispielschema:

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "argPattern": "sha256:argv:...",
          "source": "allow-always",
          "lastUsedAt": 1737150000000,
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        },
        {
          "pattern": "~/Projects/**/bin/git"
        }
      ]
    }
  }
}
```

## Richtlinienoptionen

### `tools.exec.mode`

`tools.exec.mode` ist die bevorzugte normalisierte Richtlinienoberfläche für die Host-Ausführung:

| Wert        | Verhalten                                                                                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `deny`      | Host-Ausführung blockieren.                                                                                                                                                                 |
| `allowlist` | Nur Befehle aus der Zulassungsliste ohne Nachfrage ausführen.                                                                                                                               |
| `ask`       | Zulassungslistenrichtlinie verwenden und bei fehlenden Treffern nachfragen.                                                                                                                 |
| `auto`      | Zulassungslistenrichtlinie verwenden, deterministische Treffer direkt ausführen und fehlende Genehmigungen zunächst durch den nativen automatischen Prüfer von OpenClaw prüfen lassen, bevor auf eine menschliche Genehmigungsroute zurückgegriffen wird. |
| `full`      | Host-Ausführung ohne Genehmigungsabfragen ausführen.                                                                                                                                        |

Doctor migriert das außer Gebrauch genommene persistierte Paar `tools.exec.security` / `tools.exec.ask`
zu `tools.exec.mode`.

### `exec.security`

<ParamField path="security" type='"deny" | "allowlist" | "full"'>
  - `deny` – alle Host-Ausführungsanfragen blockieren.
  - `allowlist` – nur Befehle aus der Zulassungsliste erlauben.
  - `full` – alles erlauben (entspricht Elevated).

Standard ist `full` für Gateway-/Node-Hosts; ein `sandbox`-Host verwendet stattdessen standardmäßig
`deny`.
</ParamField>

### `exec.ask`

<ParamField path="ask" type='"off" | "on-miss" | "always"'>
  Konfigurierte Abfragerichtlinie für die Host-Ausführung. Steuert das grundlegende Verhalten der
  Genehmigungsabfrage aus `tools.exec.ask` und den Host-Genehmigungsstandards.
  Standard ist `off`. Der Tool-Parameter `ask` pro Aufruf (siehe
  [Ausführungs-Tool](/de/tools/exec#parameters)) kann diese Basis nur verschärfen, und
  vom Kanal stammende Modellaufrufe ignorieren ihn, wenn die effektive Host-Abfrage `off` ist.

- `off` – niemals nachfragen.
- `on-miss` – nur nachfragen, wenn die Zulassungsliste nicht übereinstimmt.
- `always` – bei jedem Befehl nachfragen. Dauerhaftes Vertrauen durch `allow-always` unterdrückt Abfragen **nicht**, wenn der effektive Abfragemodus `always` ist.

</ParamField>

### `askFallback`

<ParamField path="askFallback" type='"deny" | "allowlist" | "full"'>
  Auflösung, wenn eine Abfrage erforderlich ist, aber keine Benutzeroberfläche erreichbar ist (oder die
  Abfrage das Zeitlimit überschreitet). Wenn nicht angegeben, ist der Standard `deny`.

- `deny` – blockieren.
- `allowlist` – nur zulassen, wenn die Zulassungsliste übereinstimmt.
- `full` – zulassen.

</ParamField>

### `tools.exec.strictInlineEval`

<ParamField path="strictInlineEval" type="boolean">
  Wenn `true`, werden Formen der Inline-Codeauswertung nur nach Genehmigung zugelassen, selbst wenn die
  Interpreter-Binärdatei selbst auf der Zulassungsliste steht. Zusätzliche Absicherung für
  Interpreter-Loader, die sich nicht eindeutig einem stabilen Dateioperanden zuordnen lassen.
</ParamField>

Beispiele, die der strikte Modus erfasst: `python -c`, `node -e`/`--eval`/`-p`,
`ruby -e`, `perl -e`/`-E`, `php -r`, `lua -e`, `osascript -e` (sowie die Inline-Formen
`awk`, `sed`, `make`, `find -exec` und `xargs`).

Im strikten Modus benötigen diese Befehle eine Prüfer- oder ausdrückliche Genehmigung. Mit
`tools.exec.mode: "auto"` kann der Prüfer eine einzelne risikoarme Ausführung genehmigen, wenn
der Befehl einen durchsetzbaren Plan besitzt; andernfalls fragt OpenClaw einen Menschen.
`Codex app-server`-Befehlsfreigaben, die den Prüfer-Rückfall erreichen, fragen einen
Menschen, da ihre Genehmigungsanfragen keine durchsetzbare aufgelöste
ausführbare Datei offenlegen.
`allow-always` speichert keine neuen Zulassungslisteneinträge für Inline-Auswertungsbefehle.

### `tools.exec.commandHighlighting`

<ParamField path="commandHighlighting" type="boolean" default="false">
  Nur Darstellung: Wenn aktiviert, kann OpenClaw vom Parser abgeleitete
  Befehlsspannen anhängen, damit Web-Genehmigungsabfragen Befehlstoken hervorheben können. Dies
  ändert **nicht** `security`, `ask`, den Abgleich mit der Zulassungsliste, das Verhalten bei strikter Inline-Auswertung,
  die Weiterleitung von Genehmigungen oder die Befehlsausführung.
</ParamField>

Global unter `tools.exec.commandHighlighting` oder pro Agent unter
`agents.entries.*.tools.exec.commandHighlighting` festlegen.

## YOLO-Modus (ohne Genehmigung)

Um Host-Ausführungen ohne Genehmigungsabfragen auszuführen, müssen **beide** Richtlinienebenen geöffnet werden:
die angeforderte Ausführungsrichtlinie in der OpenClaw-Konfiguration (`tools.exec.*`) **und**
die hostlokale Genehmigungsrichtlinie in der Genehmigungsdatei des Ausführungs-Hosts.

Ein ausgelassenes `askFallback` verwendet standardmäßig `deny`. Setzen Sie Host-`askFallback` ausdrücklich auf `full`,
wenn eine Genehmigungsabfrage ohne verfügbare Benutzeroberfläche auf „Zulassen“ zurückfallen soll.

| Ebene              | YOLO-Einstellung            |
| ------------------ | --------------------------- |
| `tools.exec.mode`  | `full` für `gateway`/`node` |
| Host-`askFallback` | `full`                     |

<Warning>
**Wichtige Unterschiede:**

- `tools.exec.host=auto` bestimmt, **wo** exec ausgeführt wird: sofern verfügbar in der Sandbox, andernfalls auf dem Gateway.
- YOLO bestimmt, **wie** Host-exec genehmigt wird: `security=full` plus `ask=off`.
- YOLO fügt **kein** separates heuristisches Genehmigungs-Gate für Befehlsverschleierung oder eine Skript-Preflight-Ablehnungsebene zusätzlich zur konfigurierten Host-exec-Richtlinie hinzu.
- `auto` macht Gateway-Routing nicht zu einer frei wählbaren Überschreibung aus einer Sandbox-Sitzung. Eine `host=node`-Anforderung pro Aufruf ist von `auto` aus zulässig; `host=gateway` ist von `auto` aus nur zulässig, wenn keine Sandbox-Laufzeit aktiv ist. Für einen stabilen, nicht automatischen Standardwert legen Sie `tools.exec.host` fest oder verwenden Sie ausdrücklich `/exec host=...`.

</Warning>

CLI-gestützte Provider, die einen eigenen nicht interaktiven Berechtigungsmodus bereitstellen,
können dieser Richtlinie folgen. Die Claude CLI fügt
`--permission-mode bypassPermissions` hinzu, wenn die effektive exec-
Richtlinie von OpenClaw YOLO ist. Bei von OpenClaw verwalteten Claude-Live-Sitzungen ist die
effektive exec-Richtlinie von OpenClaw gegenüber dem nativen Berechtigungsmodus von Claude maßgeblich:
YOLO normalisiert Live-Starts auf `--permission-mode bypassPermissions`, und
eine restriktive effektive exec-Richtlinie normalisiert Live-Starts auf
`--permission-mode default`, selbst wenn rohe Claude-Backend-Argumente einen anderen
Modus angeben.

Wenn Sie eine konservativere Einrichtung wünschen, verschärfen Sie die exec-Richtlinie von OpenClaw wieder auf
`allowlist` / `on-miss` oder `deny`.

### Dauerhafte Gateway-Host-Einrichtung „niemals nachfragen“

<Steps>
  <Step title="Angeforderte Konfigurationsrichtlinie festlegen">
    ```bash
    openclaw config set tools.exec.host gateway
    openclaw config set tools.exec.mode full
    openclaw gateway restart
    ```
  </Step>
  <Step title="Host-Genehmigungsdatei abgleichen">
    ```bash
    openclaw approvals set --stdin <<'EOF'
    {
      version: 1,
      defaults: {
        security: "full",
        ask: "off",
        askFallback: "full"
      }
    }
    EOF
    ```
  </Step>
</Steps>

### Lokale Kurzform

```bash
openclaw exec-policy preset yolo
```

Aktualisiert sowohl die lokale `tools.exec.host/security/ask` als auch die Standardwerte der lokalen
Genehmigungsdatei (einschließlich `askFallback: "full"`). Dies ist absichtlich
auf die lokale Umgebung beschränkt. Um Gateway-Host- oder Node-Host-Genehmigungen remote zu ändern, verwenden Sie
`openclaw approvals set --gateway` oder `openclaw approvals set --node
<id|name|ip>`.

Weitere integrierte Voreinstellungen: `cautious` (`host=gateway`, `security=allowlist`,
`ask=on-miss`, `askFallback=deny`) und `deny-all` (`host=gateway`,
`security=deny`, `ask=off`, `askFallback=deny`). Wenden Sie sie auf dieselbe Weise an:
`openclaw exec-policy preset cautious`.

Um einzelne Felder statt einer vollständigen Voreinstellung festzulegen, verwenden Sie
`openclaw exec-policy set --host <auto|sandbox|gateway|node> --security
<deny|allowlist|full> --ask <off|on-miss|always> --ask-fallback
<deny|allowlist|full>` mit einer beliebigen Teilmenge dieser Flags.

### Node-Host

Wenden Sie stattdessen dieselbe Genehmigungsdatei auf der Node an:

```bash
openclaw approvals set --node <id|name|ip> --stdin <<'EOF'
{
  version: 1,
  defaults: {
    security: "full",
    ask: "off",
    askFallback: "full"
  }
}
EOF
```

<Note>
**Nur lokal geltende Einschränkungen:**

- `openclaw exec-policy` synchronisiert keine Node-Genehmigungen.
- `openclaw exec-policy set --host node` wird abgelehnt.
- Node-exec-Genehmigungen werden zur Laufzeit von der Node abgerufen, daher müssen Node-bezogene Aktualisierungen `openclaw approvals --node ...` verwenden.

</Note>

### Nur für die Sitzung geltende Kurzform

- `/exec security=full ask=off` ändert nur die aktuelle Sitzung.
- `/elevated full` ist eine Notfall-Kurzform, die exec-Genehmigungen nur dann umgeht,
  wenn sowohl die angeforderte Richtlinie als auch die Host-Genehmigungsdatei zu
  `security: "full"` und `ask: "off"` aufgelöst werden. Bei einer strengeren Host-Datei wie `ask:
"always"` wird weiterhin nachgefragt.

Wenn die Host-Genehmigungsdatei strenger als die Konfiguration bleibt, setzt sich weiterhin die strengere Host-
Richtlinie durch.

## Positivliste (pro Agent)

Positivlisten gelten **pro Agent**. Wenn mehrere Agenten vorhanden sind, wechseln Sie in der
macOS-App zu dem Agenten, den Sie bearbeiten. Muster sind Glob-Übereinstimmungen.

Muster können Globs für aufgelöste Binärpfade oder Globs für reine Befehlsnamen sein.
Reine Namen stimmen nur mit Befehlen überein, die über `PATH` aufgerufen werden. Daher kann `rg` mit
`/opt/homebrew/bin/rg` übereinstimmen, wenn der Befehl `rg` lautet, aber **nicht** mit `./rg` oder
`/tmp/rg`. Verwenden Sie einen Pfad-Glob, um einem bestimmten Speicherort einer Binärdatei zu vertrauen.

Veraltete `agents.default`-Einträge werden beim Laden zu `agents.main` migriert.
Shell-Ketten wie `echo ok && pwd` erfordern weiterhin, dass jedes Segment der obersten Ebene
die Positivlistenregeln erfüllt.

Beispiele:

- `rg`
- `~/Projects/**/bin/peekaboo`
- `~/.local/bin/*`
- `/opt/homebrew/bin/rg`

### Argumente mit argPattern einschränken

Fügen Sie `argPattern` hinzu, wenn ein Positivlisteneintrag mit einer Binärdatei und einer
bestimmten Argumentstruktur übereinstimmen soll. OpenClaw verwendet auf jedem Host die Semantik regulärer
ECMAScript-Ausdrücke (JavaScript) und wertet den Ausdruck anhand
der analysierten Befehlsargumente unter Ausschluss des ausführbaren Tokens (`argv[0]`) aus.
Bei manuell erstellten Einträgen werden Argumente mit einem einzelnen Leerzeichen verbunden. Verankern Sie daher
das Muster, wenn eine exakte Übereinstimmung erforderlich ist.

```json
{
  "version": 1,
  "agents": {
    "main": {
      "allowlist": [
        {
          "pattern": "python3",
          "argPattern": "^safe\\.py$"
        }
      ]
    }
  }
}
```

Dieser Eintrag erlaubt `python3 safe.py`; `python3 other.py` stimmt mit der Positivliste
nicht überein. Wenn auch ein reiner Pfadeintrag für dieselbe Binärdatei vorhanden ist, können nicht übereinstimmende
Argumente weiterhin auf diesen reinen Pfadeintrag zurückfallen. Lassen Sie den reinen Pfadeintrag
weg, wenn die Binärdatei auf die deklarierten Argumente beschränkt werden soll.

Von Genehmigungsabläufen gespeicherte Einträge verwenden ein internes Trennzeichenformat für die exakte
argv-Übereinstimmung. Verwenden Sie vorzugsweise die Benutzeroberfläche oder den Genehmigungsablauf, um diese Einträge neu zu erzeugen,
statt den codierten Wert manuell zu bearbeiten. Wenn OpenClaw argv
für ein Befehlssegment nicht analysieren kann, stimmen Einträge mit `argPattern` nicht überein.

Erzeugte `allow-always`-Einträge sind an argv gebunden. Neue erzeugte Einträge enthalten
`argPattern`; ältere erzeugte reine Pfadeinträge werden ignoriert und benötigen eine neue
Genehmigung. Lassen Sie für eine manuelle reine Pfadregel sowohl `source` als auch `argPattern` weg.

Jeder Positivlisteneintrag unterstützt:

| Feld               | Bedeutung                                                                      |
| ------------------ | ------------------------------------------------------------------------------ |
| `pattern`          | Glob für einen aufgelösten Binärpfad oder einen reinen Befehlsnamen                      |
| `argPattern`       | ECMAScript-argv-Regex oder erzeugter exakter argv-Hash; nicht angegeben bedeutet nur Pfad |
| `id`               | Stabile undurchsichtige ID; wird bei Fehlen als UUID erzeugt                        |
| `source`           | Quelle des erzeugten Eintrags, beispielsweise `allow-always`; bei manuellen Einträgen weglassen  |
| `commandText`      | Veraltete Klartexteingabe; wird beim Laden verworfen                            |
| `lastUsedAt`       | Zeitstempel der letzten Verwendung                                                      |
| `lastUsedCommand`  | Letzter übereinstimmender Befehl; bei erzeugten gehashten argv-Einträgen nicht angegeben     |
| `lastResolvedPath` | Zuletzt aufgelöster Binärpfad                                                |

## Skill-CLIs automatisch erlauben

Wenn **Skill-CLIs automatisch erlauben** (`autoAllowSkills`) aktiviert ist, werden ausführbare Dateien,
auf die bekannte Skills verweisen, auf Nodes als in der Positivliste enthalten behandelt (macOS-Node
oder monitorloser Node-Host). Dabei wird `skills.bins` über den Gateway-RPC verwendet, um
die Liste der Skill-Binärdateien abzurufen. Deaktivieren Sie dies, wenn Sie strikt manuelle
Positivlisten wünschen.

<Warning>
- Dies ist eine **implizite Komfort-Positivliste**, die von manuellen Pfad-Positivlisteneinträgen getrennt ist.
- Sie ist für vertrauenswürdige Betreiberumgebungen vorgesehen, in denen Gateway und Node derselben Vertrauensgrenze angehören.
- Wenn Sie strikt ausdrückliches Vertrauen benötigen, behalten Sie `autoAllowSkills: false` bei und verwenden Sie ausschließlich manuelle Pfad-Positivlisteneinträge.

</Warning>

## Sichere Binärdateien und Weiterleitung von Genehmigungen

Einzelheiten zu sicheren Binärdateien (dem schnellen Nur-stdin-Pfad), zur Interpreter-Bindung und
zur Weiterleitung von Genehmigungsaufforderungen an Slack/Discord/Telegram (oder ihrer Ausführung als
native Genehmigungsclients) finden Sie unter
[Exec-Genehmigungen – erweitert](/de/tools/exec-approvals-advanced).

## Bearbeitung in der Control UI

Verwenden Sie die Karte **Control UI -> Nodes -> Exec approvals**, um Standardwerte,
agentenspezifische Überschreibungen und Positivlisten zu bearbeiten. Wählen Sie einen Geltungsbereich (Defaults oder einen Agenten),
passen Sie die Richtlinie an, fügen Sie Positivlistenmuster hinzu oder entfernen Sie sie und wählen Sie dann **Save**. Die Benutzeroberfläche
zeigt Metadaten zur letzten Verwendung jedes Musters an, damit Sie die Liste übersichtlich halten können.

Mit der Zielauswahl wählen Sie **Gateway** (lokale Genehmigungen) oder eine **Node**.
Nodes müssen `system.execApprovals.get/set` bekannt geben (macOS-App oder monitorloser
Node-Host). Wenn eine Node exec-Genehmigungen noch nicht bekannt gibt, bearbeiten Sie ihre
lokale Genehmigungsdatei direkt.

Einige Node-Hosts, darunter die Windows-Begleitanwendung, verwenden ein anderes Format für Genehmigungsrichtlinien.
Die Control UI zeigt diese hostnativen Richtlinien schreibgeschützt an. Verwenden Sie die
Begleitanwendung oder `openclaw approvals set --node <id|name|ip>` mit der nativen
Richtlinienstruktur, um sie zu bearbeiten; siehe [Genehmigungs-CLI](/de/cli/approvals).

CLI: `openclaw approvals` unterstützt die Bearbeitung von Gateway oder Node – siehe
[Genehmigungs-CLI](/de/cli/approvals).

## Genehmigungsablauf

Wenn eine Aufforderung erforderlich ist, sendet das Gateway
`exec.approval.requested` an Betreiberclients. Die Control UI und die macOS-
App lösen sie über `exec.approval.resolve` auf. Anschließend leitet das Gateway die
genehmigte Anforderung an den Node-Host weiter.

Für `host=node` enthalten Genehmigungsanforderungen eine kanonische `systemRunPlan`-
Nutzlast. Das Gateway verwendet diesen Plan als maßgeblichen Befehls-/cwd-/Sitzungs-
kontext, wenn genehmigte `system.run`-Anforderungen weitergeleitet werden:

- Der Node-exec-Pfad bereitet vorab einen kanonischen Plan vor.
- Der Genehmigungsdatensatz speichert diesen Plan und seine Bindungsmetadaten.
- Nach der Genehmigung verwendet der endgültige weitergeleitete `system.run`-Aufruf den gespeicherten Plan erneut, statt späteren Änderungen durch den Aufrufer zu vertrauen.
- Wenn der Aufrufer `command`, `rawCommand`, `cwd`, `agentId` oder `sessionKey` ändert, nachdem die Genehmigungsanforderung erstellt wurde, lehnt das Gateway die weitergeleitete Ausführung wegen einer Genehmigungsabweichung ab.

## Systemereignisse und Ablehnungen

Der exec-Lebenszyklus veröffentlicht eine `Exec finished`-Systemnachricht in der Sitzung des Agenten,
nachdem die Node den Abschluss meldet. OpenClaw kann außerdem einen
Hinweis auf den laufenden Vorgang ausgeben, sobald eine Genehmigung erteilt wurde und
`tools.exec.approvalRunningNoticeMs` verstrichen ist (Standardwert `10000`, `0` deaktiviert
dies). Abgelehnte exec-Genehmigungen sind für den Host-Befehl endgültig: Der Befehl
wird nicht ausgeführt.

- Bei asynchronen Genehmigungen für den Hauptagenten mit einer Ursprungssitzung veröffentlicht OpenClaw
  die Ablehnung als interne Folgenachricht in dieser Sitzung, damit der
  Agent nicht länger auf den asynchronen Befehl wartet und eine Reparatur
  wegen eines fehlenden Ergebnisses vermeidet.
- Wenn keine Sitzung vorhanden ist oder die Sitzung nicht fortgesetzt werden kann, kann OpenClaw
  dem Betreiber oder dem direkten Chat-Kanal weiterhin eine knappe Ablehnung melden.
- Ablehnungen für Subagenten- und Cron-Sitzungen werden nicht in dieser
  Sitzung veröffentlicht.

Gateway-Host-exec-Genehmigungen geben dasselbe Abschluss-Lebenszyklusereignis aus.
Genehmigungspflichtige exec-Ausführungen verwenden die Genehmigungs-ID erneut, um die ausstehende
Anforderung ihrer Abschluss-/Ablehnungsnachricht zuzuordnen (`Exec finished (gateway
id=...)` / `Exec denied (gateway id=...)`).

## Auswirkungen

- **`full`** ist leistungsfähig; verwenden Sie nach Möglichkeit Positivlisten.
- **`ask`** hält Sie auf dem Laufenden und ermöglicht zugleich schnelle Genehmigungen.
- Agentenspezifische Positivlisten verhindern, dass die Genehmigungen eines Agenten auf andere übergreifen.
- Genehmigungen gelten nur für Host-exec-Anforderungen von **autorisierten Absendern**. Nicht autorisierte Absender können `/exec` nicht ausgeben.
- `/exec security=full` ist eine Komfortfunktion auf Sitzungsebene für autorisierte Betreiber und umgeht Genehmigungen absichtlich. Um Host-exec vollständig zu sperren, setzen Sie die Genehmigungssicherheit auf `deny` oder verweigern Sie das Tool `exec` über die Tool-Richtlinie.

## Verwandte Themen

<CardGroup cols={2}>
  <Card title="Exec-Genehmigungen – erweitert" href="/de/tools/exec-approvals-advanced" icon="gear">
    Sichere Binärdateien, Interpreter-Bindung und Weiterleitung von Genehmigungen an den Chat.
  </Card>
  <Card title="Exec-Tool" href="/de/tools/exec" icon="terminal">
    Tool zum Ausführen von Shell-Befehlen.
  </Card>
  <Card title="Erweiterter Modus" href="/de/tools/elevated" icon="shield-exclamation">
    Notfallzugriff, der auch Genehmigungen überspringt.
  </Card>
  <Card title="Sandboxing" href="/de/gateway/sandboxing" icon="box">
    Sandbox-Modi und Arbeitsbereichszugriff.
  </Card>
  <Card title="Sicherheit" href="/de/gateway/security" icon="lock">
    Sicherheitsmodell und Härtung.
  </Card>
  <Card title="Sandbox vs. Tool-Richtlinie vs. erweiterter Modus" href="/de/gateway/sandbox-vs-tool-policy-vs-elevated" icon="sliders">
    Wann welche Steuerungsmöglichkeit eingesetzt werden sollte.
  </Card>
  <Card title="Skills" href="/de/tools/skills" icon="sparkles">
    Durch Skills gestütztes automatisches Zulassungsverhalten.
  </Card>
</CardGroup>
