---
read_when:
    - Sie möchten Anthropic-Modelle in OpenClaw verwenden
    - Sie möchten Claude-CLI- oder Claude-Desktop-Sitzungen auf gekoppelten Computern durchsuchen
summary: Anthropic Claude über API-Schlüssel oder die Claude CLI in OpenClaw verwenden
title: Anthropic
x-i18n:
    generated_at: "2026-07-24T22:25:43Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 08b34794352a559d549f7cf0cb88aca9cb537984049367f55be371bd8e0c10f0
    source_path: providers/anthropic.md
    workflow: 16
---

Anthropic entwickelt die **Claude**-Modellfamilie. OpenClaw unterstützt zwei Authentifizierungswege:

- **API-Schlüssel** – direkter Zugriff auf die Anthropic-API mit nutzungsbasierter Abrechnung (`anthropic/*`-Modelle)
- **Claude CLI** – eine vorhandene Claude-Code-Anmeldung auf demselben Host wiederverwenden

## Nutzungs- und Kostenverfolgung

OpenClaw erkennt die verfügbare Anthropic-Anmeldeinformation und wählt die passende Nutzungsansicht aus:

- Claude-Abonnement-/Einrichtungsanmeldeinformationen zeigen Kontingentzeiträume und ein optionales Budget für zusätzliche Nutzung an.
- `ANTHROPIC_ADMIN_KEY` oder `ANTHROPIC_ADMIN_API_KEY` zeigt in der Control UI unter **Nutzung** die von Anthropic gemeldeten Organisationskosten und die Nutzung der Messages API für 30 Tage an, einschließlich täglicher Ausgaben, Token-/Cache-Gesamtsummen, meistgenutzter Modelle und Kostenkategorien.
- Eine im Anthropic-Providerprofil gespeicherte `sk-ant-admin...`-Anmeldeinformation wird automatisch als Admin-API-Schlüssel erkannt.

Der Kostenverlauf der Admin API stammt aus der [Usage and Cost API](https://platform.claude.com/docs/en/manage-claude/usage-cost-api) von Anthropic. Dabei handelt es sich um die tatsächliche Providerabrechnung, getrennt von den aus Sitzungen abgeleiteten geschätzten Kosten von OpenClaw.

<Warning>
Das Claude-CLI-Backend von OpenClaw führt die installierte Claude Code CLI im
nicht interaktiven Druckmodus (`claude -p`) aus. Die aktuelle Dokumentation von Anthropic zu Claude Code
beschreibt diesen Modus als Agent-SDK-/programmatische Nutzung. Mit dem Support-Update von Anthropic vom 15. Juni 2026
wurde die angekündigte separate Änderung der Agent-SDK-Abrechnung ausgesetzt: Claude
Agent SDK, `claude -p` und die Nutzung durch Drittanbieter-Apps werden weiterhin auf die
Nutzungslimits eines angemeldeten Abonnements angerechnet, und das zuvor angekündigte monatliche
Agent-SDK-Guthaben ist nicht verfügbar, während Anthropic diesen Plan überarbeitet.

Das interaktive Claude Code wird weiterhin auf die Limits des angemeldeten Claude-Tarifs angerechnet.
Die Authentifizierung per API-Schlüssel wird direkt nutzungsabhängig abgerechnet und ist nicht von diesem Tarif abhängig.
Verwenden Sie für langlebige Gateway-Hosts, gemeinsam genutzte Automatisierung und planbare
Produktionsausgaben einen Anthropic-API-Schlüssel.

Die aktuellen Support-Artikel von Anthropic können dieses Verhalten ohne ein
OpenClaw-Release ändern:

- [Claude Code CLI-Referenz](https://code.claude.com/docs/en/cli-usage)
- [Claude Agent SDK mit Ihrem Claude-Tarif verwenden](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan)
- [Claude Code mit Ihrem Pro- oder Max-Tarif verwenden](https://support.claude.com/en/articles/11145838-use-claude-code-with-your-pro-or-max-plan)
- [Claude Code mit Ihrem Team- oder Enterprise-Tarif verwenden](https://support.claude.com/en/articles/11845131-using-claude-code-with-your-team-or-enterprise-plan)
- [Claude-Code-Kosten verwalten](https://code.claude.com/docs/en/costs)

</Warning>

## Erste Schritte

<Tabs>
  <Tab title="API-Schlüssel">
    **Am besten geeignet für:** standardmäßigen API-Zugriff und nutzungsbasierte Abrechnung.

    <Steps>
      <Step title="API-Schlüssel abrufen">
        Erstellen Sie in der [Anthropic Console](https://console.anthropic.com/) einen API-Schlüssel.
      </Step>
      <Step title="Onboarding ausführen">
        ```bash
        openclaw onboard
        # auswählen: Anthropic API key
        ```

        Alternativ können Sie den Schlüssel direkt übergeben:

        ```bash
        openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
        ```
      </Step>
      <Step title="Verfügbarkeit des Modells überprüfen">
        ```bash
        openclaw models list --provider anthropic
        ```
      </Step>
    </Steps>

    ### Konfigurationsbeispiel

    ```json5
    {
      env: { ANTHROPIC_API_KEY: "example-anthropic-key-not-real" },
      agents: { defaults: { model: { primary: "anthropic/claude-opus-5" } } },
    }
    ```

  </Tab>

  <Tab title="Claude CLI">
    **Am besten geeignet für:** Wiederverwenden einer vorhandenen Claude-CLI-Anmeldung ohne separaten API-Schlüssel.

    <Steps>
      <Step title="Sicherstellen, dass Claude CLI installiert und angemeldet ist">
        Überprüfen Sie dies mit:

        ```bash
        claude --version
        ```
      </Step>
      <Step title="Onboarding ausführen">
        ```bash
        openclaw onboard
        # auswählen: Claude CLI
        ```

        OpenClaw erkennt die vorhandenen Claude-CLI-Anmeldeinformationen und verwendet sie wieder.
      </Step>
      <Step title="Verfügbarkeit des Modells überprüfen">
        ```bash
        openclaw models list --provider anthropic
        ```
      </Step>
    </Steps>

    <Note>
    Details zur Einrichtung und Laufzeit des Claude-CLI-Backends finden Sie unter [CLI-Backends](/de/gateway/cli-backends).
    </Note>

    <Warning>
    Die Wiederverwendung der Claude CLI setzt voraus, dass der OpenClaw-Prozess auf demselben Host wie die
    Claude-CLI-Anmeldung ausgeführt wird. Bei Docker-Installationen kann ein Container-Home-Verzeichnis persistent gespeichert und dort eine Anmeldung bei
    Claude Code vorgenommen werden; siehe
    [Claude-CLI-Backend in Docker](/de/install/docker#claude-cli-backend-in-docker).
    Andere Container-Installationen wie [Podman](/de/install/podman) binden das Host-
    `~/.claude` weder bei der Einrichtung noch zur Laufzeit ein; verwenden Sie dort einen Anthropic-API-Schlüssel oder wählen Sie
    einen Provider mit von OpenClaw verwaltetem OAuth, beispielsweise
    [OpenAI Codex](/de/providers/openai).
    </Warning>

    ### Einrichtungstoken abrufen

    Führen Sie `claude setup-token` auf einem beliebigen Computer aus, auf dem Claude Code installiert ist. Der Befehl gibt
    ein langlebiges Token aus, das mit `sk-ant-oat01-` beginnt.

    Fügen Sie das Token während des Onboardings in der macOS-App ein, indem Sie
    **Anthropic setup-token** unter **Connect with an API key or token** auswählen, oder verwenden Sie:

    ```bash
    openclaw models auth login --provider anthropic --method setup-token
    ```

    ### Konfigurationsbeispiel

    Bevorzugen Sie die kanonische Anthropic-Modellreferenz zusammen mit einer CLI-Laufzeitüberschreibung:

    ```json5
    {
      agents: {
        defaults: {
          model: { primary: "anthropic/claude-opus-5" },
          models: {
            "anthropic/claude-opus-5": {
              agentRuntime: { id: "claude-cli" },
            },
          },
        },
      },
    }
    ```

    Ältere `claude-cli/claude-opus-4-7`-Modellreferenzen funktionieren aus
    Kompatibilitätsgründen weiterhin, neue Konfigurationen sollten die Provider-/Modellauswahl jedoch als
    `anthropic/*` beibehalten und das Ausführungs-Backend in der Laufzeitrichtlinie des Providers/Modells festlegen.

    ### Abrechnung und `claude -p`

    OpenClaw verwendet für Claude-CLI-Ausführungen den nicht interaktiven `claude -p`-Pfad von Claude Code.
    Anthropic behandelt diesen Pfad derzeit als Agent-SDK-/programmatische Nutzung:

    - Mit dem Support-Update von Anthropic vom 15. Juni 2026 wurde der zuvor angekündigte
      separate Agent-SDK-Guthabenplan ausgesetzt.
    - Die Nutzung von Claude Agent SDK, `claude -p` und Drittanbieter-Apps im Rahmen eines Abonnementtarifs
      wird weiterhin auf die Nutzungslimits des angemeldeten Abonnements angerechnet.
    - Das zuvor angekündigte monatliche Agent-SDK-Guthaben ist nicht verfügbar, während
      Anthropic diesen Plan überarbeitet.
    - Anmeldungen per Console/API-Schlüssel verwenden die nutzungsabhängige API-Abrechnung und erhalten
      kein Agent-SDK-Guthaben des Abonnements.

    Den Hinweis zur Aussetzung finden Sie im [Artikel zum Agent-SDK-Tarif
    von Anthropic](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan). Informationen zum Abonnementverhalten finden Sie in den Claude-Code-Tarifartikeln für
    [Pro/Max](https://support.claude.com/en/articles/11145838-use-claude-code-with-your-pro-or-max-plan)
    und
    [Team/Enterprise](https://support.claude.com/en/articles/11845131-use-claude-code-with-your-team-or-enterprise-plan).

    Anthropic kann das Abrechnungs- und Ratenbegrenzungsverhalten von Claude Code ohne ein
    OpenClaw-Release ändern. Prüfen Sie `claude auth status`, `/status` und
    die verlinkte Anthropic-Dokumentation, wenn eine planbare Abrechnung wichtig ist.

    <Tip>
    Verwenden Sie für gemeinsam genutzte Produktionsautomatisierung einen Anthropic-API-Schlüssel anstelle der
    Claude CLI. OpenClaw unterstützt außerdem abonnementähnliche Optionen von
    [OpenAI Codex](/de/providers/openai), [Qwen Cloud](/de/providers/qwen),
    [MiniMax](/de/providers/minimax) und [Z.AI / GLM](/de/providers/zai).
    </Tip>

  </Tab>
</Tabs>

## Claude-Sitzungen auf mehreren Computern

Das gebündelte Anthropic-Plugin fügt der normalen Sitzungsseitenleiste eine Gruppe **Claude Code** hinzu.
Zeilen werden im normalen Chat-Bereich geöffnet. Das Plugin erkennt nicht archivierte Claude-
Code-Sitzungen auf dem Gateway und auf verbundenen Node-Hosts:

- Claude-CLI-Sitzungen stammen aus gültigen Projektindex-Datensätzen. Bei nicht indizierten
  Transkripten erkennt ein begrenzter Metadaten-Fallback gleichzeitig laufende interaktive Nicht-Sidechain-
  Sitzungen (`cli`) und Headless-Agent-SDK-CLI-Sitzungen (`sdk-cli`) unter
  `~/.claude/projects/`.
- Claude-Desktop-Sitzungen verwenden den Desktop-Titel, die Aktivitätszeit und den
  Archivstatus, wenn ihre Metadaten auf dieselbe Claude-Code-Sitzungs-ID verweisen.
- Eine reine CLI-Sitzung besitzt kein Archivkennzeichen und bleibt daher sichtbar, solange ihr
  Transkript vorhanden ist.

Für die Erkennung ist keine zusätzliche OpenClaw-Konfiguration erforderlich. Das Anthropic-Plugin
ist gebündelt und standardmäßig aktiviert; eine native macOS-Node kündigt die schreibgeschützten
Claude-Sitzungsbefehle an, wenn das lokale Verzeichnis `~/.claude/projects/` vorhanden ist.
Genehmigen Sie das Upgrade der Node-Kopplung, wenn diese Befehle erstmals angezeigt werden.

Die Seitenleiste gruppiert Zeilen nach ihrem Gateway- oder gekoppelten Node-Host und zeigt die
neueste begrenzte Seite jedes Hosts an, sobald der jeweilige Computer antwortet. Ein erneuter Abgleich erfolgt
nach Änderungen der Host-Konnektivität, wenn die Seite wieder den Fokus erhält, sowie während der Anzeige höchstens alle
30 Sekunden, sodass außerhalb von OpenClaw erstellte Claude-Sitzungen
ohne Neuladen erscheinen. Bei einem geänderten Katalog erfolgt schneller ein weiterer Durchlauf. Verwenden Sie **Weitere
Sitzungen laden** unter einer Kataloggruppe, um für jeden Host mit
weiterem Verlauf die nächste Seite anzuhängen. Angehängte Zeilen bleiben sichtbar und werden bei
Aktualisierungen erneut bis zur gleichen Tiefe abgerufen. Katalogclients verwenden `sessions.catalog.list`; beim Öffnen einer Zeile wird
`sessions.catalog.read` verwendet.

Die Terminalübernahme löst `claude` aus dem PATH der Anmelde-Shell des besitzenden Host-Benutzers
vor dem PATH des Dienstes/Daemons auf. Dadurch bleiben von Apps gestartete Sitzungen auf die
Claude CLI abgestimmt, die der Operator in einem normalen Terminal erhält.

Beim Auswählen einer Zeile wird zuerst die neueste Transkriptseite gelesen. **Ältere Transkriptelemente
laden** folgt einem undurchsichtigen Byte-Cursor und liest einen weiteren begrenzten Abschnitt aus der
JSONL-Datei, anstatt den gesamten Verlauf zu laden. Normale Benutzer-, Assistenten-,
Reasoning-, Toolaufruf- und Toolergebnisinhalte bleiben erhalten. Ein einzelnes Element,
das die Sicherheitsobergrenze von Node/Gateway überschreitet, wird deutlich als abgeschnitten gekennzeichnet.

Bei einer Gateway-lokalen `claude-cli`-Zeile ruft eine Eingabe im normalen Eingabefeld
`sessions.catalog.continue` auf. OpenClaw löst den lokalen Katalogeintrag erneut auf,
erstellt eine modellgebundene native Sitzung oder verwendet sie erneut, importiert höchstens 200 sichtbare
Elemente oder 512 KiB und initialisiert die Claude-CLI-Bindung. Der erste Durchlauf wird mit
`--fork-session` fortgesetzt; Claude weist dem Fork eine neue Sitzungs-ID zu, sodass spätere Durchläufe
den Fork verwenden und die Quellsitzung unverändert bleibt.

Ein Headless-Node-Host kann die Fortsetzung seiner Claude-CLI-Zeilen ebenfalls ermöglichen, indem
die folgende Node-lokale Einstellung aktiviert und der Node-Host neu gestartet wird:

```json5
{
  nodeHost: {
    agentRuns: {
      claude: { enabled: true },
    },
  },
}
```

Die Node kündigt `agent.cli.claude.run.v1` nur an, wenn die Einstellung aktiviert ist
und ihre lokale ausführbare Datei `claude` aufgelöst werden kann. OpenClaw löst den Katalogeintrag
auf dieser Node erneut auf, importiert denselben begrenzten Verlauf und bindet die übernommene
Sitzung an die Node sowie an das vom Katalog gemeldete Arbeitsverzeichnis. Jeder Durchlauf führt den
echten `claude -p`-Prozess der Node mit den Claude-Dateien und der Anmeldung dieser Node aus. Die
Richtlinie der Node für Ausführungsgenehmigungen gilt weiterhin; das Gateway kann die Zustimmung nicht erzwingen.

Node-Fortsetzung v1 arbeitet ausschließlich im One-Shot-Modus. Sie lässt die Gateway-Loopback-MCP-Konfiguration und
die Gateway-Skills-Plugin-Argumente weg, führt keine erneute Initialisierung aus einem Gateway-Transkript durch und
lehnt Anhänge und Bilder ab. Claude-Desktop-Zeilen bleiben schreibgeschützt. Native
macOS-App-Nodes bleiben ebenfalls schreibgeschützt, bis die App den Ausführungsbefehl ankündigt.

<Note>
Claude-Sitzungen gekoppelter Nodes bleiben schreibgeschützt, sofern der Headless-Node nicht ausdrücklich
`agent.cli.claude.run.v1` ankündigt. OpenClaw verändert niemals Claude-Desktop-
Metadaten und archiviert keine Claude-Sitzungen. Die Seite erfordert eine Operatorverbindung
mit Schreibberechtigung, da sie authentifiziertes `node.invoke` verwendet; Auflisten und Lesen
bleiben selbst auf einer Node mit aktivierter Fortsetzung schreibgeschützt.
</Note>

Siehe [Nodes: Claude-Sitzungen und -Transkripte](/de/nodes#claude-sessions-and-transcripts)
für den Node-Befehl und die Sicherheitsgrenze.

## Standardwerte für das Denken (Claude Opus 5, Sonnet 5, Mythos 5, Fable 5, 4.8 und 4.6)

`anthropic/claude-opus-5` verwendet standardmäßig adaptives Denken mit dem Aufwand `high`.
Verwenden Sie `/think off`, um das Denken zu deaktivieren, oder `/think xhigh|max` für die
höheren nativen Aufwandsstufen des Modells. OpenClaw lässt manuelle Denkbudgets, benutzerdefinierte
Sampling-Parameter, Assistant-Prefills und Priority Tier für Opus 5 aus, da
Anthropic diese Anfragefunktionen bei diesem Modell nicht unterstützt. Der Katalog
weist dessen Kontextfenster mit 1.000.000 Token, das Ausgabelimit von 128.000 Token, die
Bildeingabe und die Preise von `$5/$25` für Ein- und Ausgabe aus.

`anthropic/claude-sonnet-5` verwendet dieselben Standardwerte für adaptives Denken und dieselben
Anfragebeschränkungen. Der Katalog verwendet bis zum 31. August 2026 Anthropics Einführungspreise von `$2/$10` für Ein- und Ausgabe;
ab dem 1. September 2026 gelten die Standardpreise von `$3/$15`.

`anthropic/claude-fable-5` verwendet immer adaptives Denken und standardmäßig den Aufwand `high`.
Anthropic erlaubt bei diesem Modell nicht, das Denken zu deaktivieren, daher werden
`/think off` und `/think minimal` stattdessen dem Aufwand `low` zugeordnet. OpenClaw lässt außerdem
benutzerdefinierte Temperaturwerte bei Anfragen an Fable 5 aus, da Anthropic
eine Temperaturüberschreibung bei jeder Anfrage mit aktiviertem Denken ablehnt.

`anthropic/claude-mythos-5` ist ein Modell mit eingeschränktem Zugriff und demselben Vertrag für
stets aktives adaptives Denken. OpenClaw verwendet standardmäßig `high`, ordnet `/think off` und
`/think minimal` `low` zu und lässt vom Aufrufer gewählte Sampling-Parameter aus.
Der Katalog weist dessen Kontextfenster mit 1.000.000 Token, das Ausgabelimit von 128.000 Token,
die Bildeingabe und die Preise von `$10/$50` für Ein- und Ausgabe aus.

Bei Claude Opus 4.8 bleibt das Denken in OpenClaw standardmäßig deaktiviert. Wenn Sie
adaptives Denken ausdrücklich mit `/think high|xhigh|max` aktivieren, sendet OpenClaw
Anthropics Aufwandswerte für Opus 4.8; Claude-4.6-Modelle (Opus 4.6 und Sonnet 4.6)
verwenden standardmäßig `adaptive`.

Überschreiben Sie dies pro Nachricht mit `/think:<level>` oder in den Modellparametern:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-5": {
          params: { thinking: "high" },
        },
      },
    },
  },
}
```

<Note>
Zugehörige Anthropic-Dokumentation:
- [Adaptives Denken](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
- [Erweitertes Denken](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

</Note>

## Fallback bei sicherheitsbedingter Ablehnung (Claude Fable 5)

<Warning>
Die Verwendung von Claude Fable 5 bedeutet, dass auch Claude Opus 4.8 verwendet wird. Fable 5 enthält
Sicherheitsklassifikatoren, die eine Anfrage ablehnen können, und Anthropics zugelassener
Wiederherstellungsweg besteht darin, `claude-opus-4-8` diese Anfrage beantworten zu lassen. OpenClaw aktiviert dies
bei direkten Anfragen mit API-Schlüssel automatisch, sodass einige Fable-Anfragen
von Claude Opus 4.8 beantwortet und als solche abgerechnet werden. Wenn Ihre Richtlinien oder Ihr Budget
keine von Opus beantworteten Anfragen zulassen, wählen Sie `anthropic/claude-fable-5` nicht aus.
</Warning>

### Warum dies erforderlich ist

Die Klassifikatoren von Fable 5 geben bei Anfragen in eingeschränkten
Domänen `stop_reason: "refusal"` zurück und erzeugen auch Fehlalarme bei an sich unbedenklichen angrenzenden Aufgaben
(Sicherheitswerkzeuge, Biowissenschaften oder sogar die Aufforderung an das Modell, seine unverarbeiteten
Gedankengänge wiederzugeben). Ohne einen Fallback schlägt die Anfrage mit einem Fehler fehl, obwohl
ein anderes Claude-Modell sie problemlos bearbeiten würde – Anthropics eigene Ablehnungsmeldung
weist API-Integratoren an, ein Fallback-Modell zu konfigurieren.

### Funktionsweise

1. Bei jeder direkten Anfrage mit API-Schlüssel an `anthropic/claude-fable-5` sendet OpenClaw
   Anthropics serverseitige Fallback-Aktivierung: den
   Beta-Header `server-side-fallback-2026-06-01` sowie
   `fallbacks: [{"model": "claude-opus-4-8"}]`. Claude Opus 4.8 ist das einzige
   Fallback-Ziel, das Anthropic für Fable 5 zulässt.
2. Nur eine Ablehnung durch den Sicherheitsklassifikator löst den Fallback aus. Ratenbegrenzungen,
   Überlastungen und Serverfehler verhalten sich genau wie zuvor und durchlaufen
   OpenClaws normalen [Modell-Failover](/de/concepts/model-failover).
3. Die Wiederherstellung erfolgt innerhalb desselben Aufrufs. Eine Ablehnung vor einer Ausgabe ist
   abgesehen von der Latenz nicht sichtbar; die gesamte Antwort stammt von Opus 4.8. Bei einer
   Ablehnung während des Streams bleibt der Teiltext als Präfix erhalten, an das das Fallback-Modell
   anknüpft, während die Gedankengänge und Tool-Aufrufe des ablehnenden Modells
   gemäß Anthropics Regeln für die Wiedergabe verworfen werden (sie dürfen weder zurückgegeben noch
   ausgeführt werden).
4. Wenn auch Claude Opus 4.8 ablehnt, wird die Ablehnung für die Anfrage
   als Fehler ausgegeben, genau wie vor dieser Funktion.

Der Fallback erfolgt auf Ebene der Anthropic-API, daher muss `claude-opus-4-8`
nicht in Ihrer konfigurierten Modellliste oder Fallback-Kette enthalten sein – ein
Fable-fähiger API-Schlüssel kann Opus immer verwenden.

### Beobachtbarkeit und Abrechnung

- Eine durch den Fallback beantwortete Anfrage zeichnet in der Assistant-Nachricht eine Diagnose vom Typ `provider_fallback` auf,
  die `fromModel` und `toModel` nennt, und `responseModel` der Nachricht
  meldet `claude-opus-4-8`.
- Anthropic rechnet pro Versuch ab: Eine Ablehnung vor der Ausgabe ist kostenlos, und die Wiederherstellung
  wird nach den Tarifen von Claude Opus 4.8 abgerechnet (derzeit halb so hoch wie die Tarife von Fable 5). OpenClaws
  Kostenschätzung pro Anfrage berechnet durch den Fallback beantwortete Anfragen entsprechend den Opus-Tarifen.
- Bei einer Ablehnung während des Streams berechnet Anthropic zusätzlich den bereits gestreamten Fable-Teil;
  dieser Anteil wird in der nutzungsbezogenen Aufschlüsselung pro Versuch der API ausgewiesen,
  aber nicht in OpenClaws Kostenschätzung pro Anfrage einbezogen.

### Geltungsbereich

Gilt für `anthropic/claude-fable-5` mit API-Schlüssel-Authentifizierung gegenüber
`api.anthropic.com`. OAuth (Wiederverwendung eines Claude-CLI-Abonnements), Proxy-Basis-URLs,
Bedrock-, Vertex- und Foundry-Anfragen bleiben unverändert und geben
Ablehnungen weiterhin als Fehler aus.

Live verifiziert: Eine unbedenkliche Eingabeaufforderung, die Fable 5 auffordert, seine unverarbeitete Gedankenkette
wiederzugeben, wird ohne Fallbacks mit `category: "reasoning_extraction"` abgelehnt,
während dieselbe Eingabeaufforderung über OpenClaw eine normale, von Opus beantwortete
Antwort mit angehängter Diagnose `provider_fallback` zurückgibt.

Siehe Anthropics [Leitfaden zu Ablehnungen und Fallbacks](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback)
zum zugrunde liegenden Verhalten.

## Prompt-Caching

OpenClaw unterstützt Anthropics Prompt-Caching-Funktion für die Authentifizierung mit API-Schlüssel.

| Wert               | Cache-Dauer | Beschreibung                            |
| ------------------- | -------------- | -------------------------------------- |
| `"short"` (Standard) | 5 Minuten      | Wird bei Authentifizierung mit API-Schlüssel automatisch angewendet |
| `"long"`            | 1 Stunde         | Erweiterter Cache                         |
| `"none"`            | Kein Caching     | Prompt-Caching deaktivieren                 |

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

<AccordionGroup>
  <Accordion title="Cache-Überschreibungen pro Agent">
    Verwenden Sie Parameter auf Modellebene als Ausgangsbasis und überschreiben Sie dann bestimmte Agenten über `agents.entries.*.params`:

    ```json5
    {
      agents: {
        defaults: {
          model: { primary: "anthropic/claude-opus-4-6" },
          models: {
            "anthropic/claude-opus-4-6": {
              params: { cacheRetention: "long" },
            },
          },
        },
        list: [
          { id: "research", default: true },
          { id: "alerts", params: { cacheRetention: "none" } },
        ],
      },
    }
    ```

    Reihenfolge der Konfigurationszusammenführung:

    1. `agents.defaults.models["provider/model"].params`
    2. `agents.entries.*.params` (übereinstimmendes `id`, Überschreibung nach Schlüssel)

    Dadurch kann ein Agent einen langlebigen Cache behalten, während ein anderer Agent mit demselben Modell das Caching für stoßartigen Datenverkehr mit geringer Wiederverwendung deaktiviert.

  </Accordion>

  <Accordion title="Hinweise zu Claude auf Bedrock">
    - Anthropic-Claude-Modelle auf Bedrock (`amazon-bedrock/*anthropic.claude*`) akzeptieren die Durchleitung von `cacheRetention`, wenn dies konfiguriert ist.
    - Nicht von Anthropic stammende Bedrock-Modelle werden zur Laufzeit auf `cacheRetention: "none"` erzwungen.
    - Intelligente Standardwerte für API-Schlüssel setzen außerdem `cacheRetention: "short"` für Claude-auf-Bedrock-Referenzen, wenn kein expliziter Wert festgelegt ist.

  </Accordion>
</AccordionGroup>

## Erweiterte Konfiguration

<AccordionGroup>
  <Accordion title="Schnellmodus">
    OpenClaws gemeinsam verwendeter Schalter `/fast` setzt Anthropics Feld `service_tier` für direkten Datenverkehr mit API-Schlüssel an `api.anthropic.com`.

    | Befehl | Zuordnung zu |
    |---------|---------|
    | `/fast on` | `service_tier: "auto"` |
    | `/fast off` | `service_tier: "standard_only"` |

    ```json5
    {
      agents: {
        defaults: {
          models: {
            "anthropic/claude-sonnet-4-6": {
              params: { fastMode: true },
            },
          },
        },
      },
    }
    ```

    <Note>
    - Gilt nur für direkte `api.anthropic.com`-Anfragen mit einem API-Schlüssel. OAuth-/Abonnement-Token-Anfragen und Proxy-Routen erhalten niemals ein Feld `service_tier`.
    - Explizite Parameter `serviceTier` oder `service_tier` überschreiben `/fast`, wenn beide festgelegt sind.
    - Claude Opus 5 und Sonnet 5 unterstützen Priority Tier nicht, daher lässt OpenClaw `service_tier` bei diesen Modellen aus.
    - Bei Konten ohne Priority-Tier-Kapazität kann `service_tier: "auto"` zu `standard` aufgelöst werden.

    </Note>

  </Accordion>

  <Accordion title="Medienverständnis (Bild und PDF)">
    Das mitgelieferte Anthropic-Plugin registriert das Verständnis von Bildern und PDFs. OpenClaw
    ermittelt Medienfunktionen automatisch anhand der konfigurierten Anthropic-Authentifizierung;
    es ist keine zusätzliche Konfiguration erforderlich.

    | Eigenschaft        | Wert                 |
    | --------------- | --------------------- |
    | Standardmodell   | `claude-opus-5`       |
    | Unterstützte Eingabe | Bilder, PDF-Dokumente |

    Wenn ein Bild oder eine PDF-Datei an eine Unterhaltung angehängt wird, leitet OpenClaw
    sie automatisch über den Anthropic-Provider für Medienverständnis weiter.

  </Accordion>

  <Accordion title="1M-Kontextfenster">
    Claude Opus 5, Sonnet 5, Mythos 5 und Fable 5 verfügen über ein exaktes
    Eingabefenster von 1.000.000 Token und unterstützen bis zu 128.000 Ausgabetoken.
    Anthropics 1M-Kontextfenster ist außerdem allgemein verfügbar für Claude-4.x-Modelle mit adaptivem
    Denken: Opus 4.8,
    Opus 4.7, Opus 4.6 und Sonnet 4.6. OpenClaw dimensioniert diese Modelle
    automatisch; `params.context1m` ist nicht erforderlich:

    ```json5
    {
      agents: {
        defaults: {
          models: {
            "anthropic/claude-opus-5": {},
            "anthropic/claude-sonnet-5": {},
            "anthropic/claude-mythos-5": {},
            "anthropic/claude-opus-4-6": {},
          },
        },
      },
    }
    ```

    Ältere Konfigurationen können `params.context1m: true` beibehalten; für
    diese Modelle ist dies ein harmloser No-Op, und OpenClaw sendet den eingestellten
    Beta-Header `context-1m-2025-08-07` grundsätzlich nicht mehr. Ältere `anthropicBeta`-Konfigurationseinträge
    mit diesem Wert werden bei der Auflösung der Anfrage-Header verworfen, und
    nicht unterstützte ältere Claude-Modelle behalten ihr normales Kontextfenster.

    `params.context1m: true` verhält sich beim Claude-CLI-Backend
    (`claude-cli/*`) ebenso: Geeignete, allgemein verfügbare Opus- und Sonnet-Modelle erhalten das
    1M-Fenster bereits automatisch, sodass der Parameter auch dort optional ist.

    <Warning>
    Erfordert Zugriff auf lange Kontexte für Ihre Anthropic-Anmeldedaten. Die Authentifizierung per OAuth-/Abonnement-Token behält die erforderlichen Anthropic-Beta-Header bei, aber OpenClaw entfernt den eingestellten 1M-Beta-Header, falls er noch in einer älteren Konfiguration vorhanden ist.
    </Warning>

  </Accordion>

  <Accordion title="Claude Opus 5: 1M-Kontext">
    `anthropic/claude-opus-5` und dessen Variante `claude-cli` verfügen standardmäßig über ein
    1M-Kontextfenster; `params.context1m: true` ist nicht erforderlich.
  </Accordion>
</AccordionGroup>

## Fehlerbehebung

<AccordionGroup>
  <Accordion title="401-Fehler / Token plötzlich ungültig">
    Die Anthropic-Token-Authentifizierung läuft ab und kann widerrufen werden. Verwenden Sie für neue Einrichtungen stattdessen einen Anthropic-API-Schlüssel.
  </Accordion>

  <Accordion title='Kein API-Schlüssel für Provider "anthropic" gefunden'>
    Die Anthropic-Authentifizierung erfolgt **pro Agent**; neue Agenten übernehmen die Schlüssel des Hauptagenten nicht. Führen Sie das Onboarding für diesen Agenten erneut aus (oder konfigurieren Sie einen API-Schlüssel auf dem Gateway-Host) und überprüfen Sie die Konfiguration anschließend mit `openclaw models status`.
  </Accordion>

  <Accordion title='Keine Anmeldedaten für Profil "anthropic:default" gefunden'>
    Führen Sie `openclaw models status` aus, um zu sehen, welches Authentifizierungsprofil aktiv ist. Führen Sie das Onboarding erneut aus oder konfigurieren Sie einen API-Schlüssel für diesen Profilpfad.
  </Accordion>

  <Accordion title="Kein verfügbares Authentifizierungsprofil (alle in der Abklingzeit)">
    Prüfen Sie `openclaw models status --json` auf `auth.unusableProfiles`. Abklingzeiten aufgrund von Anthropic-Ratenbegrenzungen können modellspezifisch sein, sodass ein anderes Anthropic-Modell derselben Familie möglicherweise weiterhin verwendet werden kann. Fügen Sie ein weiteres Anthropic-Profil hinzu oder warten Sie, bis die Abklingzeit abgelaufen ist.
  </Accordion>
</AccordionGroup>

<Note>
Weitere Hilfe: [Fehlerbehebung](/de/help/troubleshooting) und [Häufig gestellte Fragen](/de/help/faq).
</Note>

## Verwandte Themen

<CardGroup cols={2}>
  <Card title="Modellauswahl" href="/de/concepts/model-providers" icon="layers">
    Auswahl von Providern, Modellreferenzen und Failover-Verhalten.
  </Card>
  <Card title="CLI-Backends" href="/de/gateway/cli-backends" icon="terminal">
    Einrichtung des Claude-CLI-Backends und Details zur Laufzeit.
  </Card>
  <Card title="Prompt-Caching" href="/de/reference/prompt-caching" icon="database">
    Funktionsweise des Prompt-Cachings bei verschiedenen Providern.
  </Card>
  <Card title="OAuth und Authentifizierung" href="/de/gateway/authentication" icon="key">
    Details zur Authentifizierung und Regeln für die Wiederverwendung von Anmeldedaten.
  </Card>
</CardGroup>
