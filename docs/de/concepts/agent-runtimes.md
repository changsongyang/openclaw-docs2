---
read_when:
    - Sie wählen zwischen OpenClaw, Codex, ACP oder einer anderen nativen Agenten-Runtime.
    - Sie sind durch Provider-/Modell-/Runtime-Bezeichnungen im Status oder in der Konfiguration verwirrt
    - Sie dokumentieren die Funktionsparität für ein natives Harness.
summary: Wie OpenClaw Modell-Provider, Modelle, Kanäle und Agent-Runtimes voneinander trennt
title: Agentenlaufzeitumgebungen
x-i18n:
    generated_at: "2026-07-24T22:11:05Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 980d112946535df1566f2df4e3e71abacc2b073b51717c1e85fbb678691d39cb
    source_path: concepts/agent-runtimes.md
    workflow: 16
---

Eine **Agent-Runtime** besitzt eine vorbereitete Modellschleife: Sie empfängt den Prompt,
steuert die Modellausgabe, verarbeitet native Tool-Aufrufe und gibt den abgeschlossenen Turn
an OpenClaw zurück.

Runtimes werden leicht mit Providern verwechselt, da beide im Umfeld der Modellkonfiguration
auftauchen. Sie bilden unterschiedliche Ebenen:

| Ebene         | Beispiele                                     | Bedeutung                                                             |
| ------------- | -------------------------------------------- | ------------------------------------------------------------------- |
| Provider      | `anthropic`, `github-copilot`, `openai`      | Wie OpenClaw sich authentifiziert, Modelle erkennt und Modellreferenzen benennt. |
| Modell         | `claude-opus-4-6`, `gpt-5.6-sol`             | Das für den Agent-Turn ausgewählte Modell.                              |
| Agent-Runtime | `claude-cli`, `codex`, `copilot`, `openclaw` | Die Low-Level-Schleife oder das Backend, das den vorbereiteten Turn ausführt.      |
| Kanal       | Discord, Slack, Telegram, WhatsApp           | Wo Nachrichten in OpenClaw eingehen und es verlassen.                            |

Ein **Harness** ist die Implementierung, die eine Agent-Runtime bereitstellt
(Codebegriff). Beispielsweise implementiert das mitgelieferte Codex-Harness die Runtime `codex`.
Die öffentliche Konfiguration verwendet `agentRuntime.id` für Provider- oder Modelleinträge; Runtime-Schlüssel
für den gesamten Agent sind veraltet und werden ignoriert. `openclaw doctor --fix` entfernt alte
Runtime-Festlegungen für den gesamten Agent und schreibt veraltete Runtime-Modellreferenzen in kanonische
Provider-/Modellreferenzen sowie, falls erforderlich, in eine modellspezifische Runtime-Richtlinie um.

Zwei Runtime-Familien:

- **Eingebettete Harnesses** werden innerhalb der vorbereiteten Agent-Schleife von OpenClaw ausgeführt: die
  integrierte Runtime `openclaw` sowie registrierte Plugin-Harnesses wie
  `codex` und `copilot`.
- **CLI-Backends** führen einen lokalen CLI-Prozess aus, während die Modellreferenz
  kanonisch bleibt. Beispielsweise bedeutet `anthropic/claude-opus-5` mit einem modellspezifischen
  `agentRuntime.id: "claude-cli"`: „Das Anthropic-Modell auswählen und
  über die Claude CLI ausführen.“ `claude-cli` ist keine eingebettete Harness-ID und darf nicht
  an die AgentHarness-Auswahl übergeben werden.

Das Harness `copilot` ist ein separates, optionales externes Plugin-Harness für die
GitHub Copilot CLI; die benutzerorientierte Entscheidung zwischen PI, Codex und der
GitHub-Copilot-Agent-Runtime finden Sie unter [GitHub-Copilot-Agent-Runtime](/de/plugins/copilot).

## Codex-Oberflächen

Mehrere Oberflächen verwenden den Namen Codex:

| Oberfläche                                          | OpenClaw-Name/-Konfiguration                 | Funktion                                                                                                   |
| ------------------------------------------------ | ------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| Native Codex-App-Server-Runtime                  | Modellreferenzen `openai/*`                | Führt eingebettete OpenAI-Agent-Turns über den Codex-App-Server aus. Dies ist die übliche Einrichtung für ein ChatGPT-/Codex-Abonnement. |
| Codex-OAuth-Authentifizierungsprofile                        | OAuth-Profile `openai`              | Speichert die ChatGPT-/Codex-Abonnementauthentifizierung, die das Codex-App-Server-Harness verwendet.                             |
| Codex-ACP-Adapter                                | `runtime: "acp"`, `agentId: "codex"` | Führt Codex über die externe ACP-/acpx-Steuerungsebene aus. Nur verwenden, wenn ACP/acpx ausdrücklich angefordert wird.        |
| Nativer Codex-Befehlssatz zur Chat-Steuerung            | `/codex ...`                         | Bindet Codex-App-Server-Threads aus dem Chat, setzt sie fort, steuert und beendet sie und zeigt Informationen dazu an.                                |
| OpenAI-Platform-API-Route für Nicht-Agent-Oberflächen | `openai/*` plus API-Schlüssel-Authentifizierung         | Direkte OpenAI-APIs wie Bilder, Einbettungen, Sprache und Echtzeit.                                           |

Diese Oberflächen sind absichtlich unabhängig voneinander. Durch Aktivieren des Plugins `codex`
werden native App-Server-Funktionen verfügbar; `openclaw doctor --fix` ist für die
Reparatur veralteter Codex-Routen und die Bereinigung veralteter Sitzungsfestlegungen zuständig. Die Auswahl von `openai/*`
für ein Agent-Modell bedeutet nun „dies über Codex ausführen“, sofern keine
Nicht-Agent-Oberfläche der OpenAI-API verwendet wird.

Die übliche Einrichtung für ein ChatGPT-/Codex-Abonnement verwendet Codex OAuth zur Authentifizierung,
behält jedoch `openai/*` als Modellreferenz bei und wählt die Runtime `codex` aus:

```json5
{
  agents: {
    defaults: {
      model: "openai/gpt-5.6-sol",
    },
  },
}
```

Das bedeutet, dass OpenClaw eine OpenAI-Modellreferenz auswählt und anschließend die
Codex-App-Server-Runtime auffordert, den eingebetteten Agent-Turn auszuführen. Es bedeutet weder
„API-Abrechnung verwenden“, noch bedeutet es, dass der Kanal, der Modell-Provider-Katalog oder
der OpenClaw-Sitzungsspeicher zu Codex wird.

Wenn das mitgelieferte Plugin `codex` aktiviert ist, verwenden Sie für die natürlichsprachliche
Codex-Steuerung anstelle von ACP die native Befehlsoberfläche `/codex` (`/codex bind`,
`/codex threads`, `/codex resume`, `/codex steer`,
`/codex stop`). Verwenden Sie ACP für Codex nur, wenn ausdrücklich ACP/acpx angefordert wird oder der
ACP-Adapterpfad getestet wird. Claude Code, Gemini CLI, OpenCode, Cursor und ähnliche externe
Harnesses verwenden weiterhin ACP.

Entscheidungsbaum:

1. **Codex binden/steuern/Thread fortsetzen/lenken/beenden** -> native Befehlsoberfläche `/codex`, wenn das mitgelieferte Plugin `codex` aktiviert ist.
2. **Codex als eingebettete Runtime** oder die normale abonnementgestützte Codex-Agent-Nutzung -> `openai/<model>`.
3. **OpenClaw wurde ausdrücklich für ein OpenAI-Modell ausgewählt** -> Modellreferenz als `openai/<model>` beibehalten und Provider-/Modell-Runtime-Richtlinie auf `agentRuntime.id: "openclaw"` setzen. Ein ausgewähltes OAuth-Profil `openai` wird intern über den Codex-Authentifizierungstransport von OpenClaw geroutet.
4. **Veraltete Codex-Modellreferenzen in der Konfiguration** -> mit `openclaw doctor --fix` zu `openai/<model>` reparieren; Doctor behält die Codex-Authentifizierungsroute bei, indem bei Bedarf das Provider-/modellspezifische `agentRuntime.id: "codex"` hinzugefügt wird, wenn die alte Modellreferenz dies implizierte. Veraltete **`codex-cli/*`**-Modellreferenzen werden zur selben Codex-App-Server-Route `openai/<model>` repariert; OpenClaw behält kein mitgeliefertes Codex-CLI-Backend mehr bei.
5. **ACP, acpx oder Codex-ACP-Adapter ausdrücklich angefordert** -> `runtime: "acp"` und `agentId: "codex"`.
6. **Claude Code, Gemini CLI, OpenCode, Cursor, Droid oder ein anderes externes Harness** -> ACP/acpx, nicht die native Sub-Agent-Runtime.

| Gemeint ist ...                             | Verwenden Sie ...                                       |
| --------------------------------------- | -------------------------------------------- |
| Chat-/Thread-Steuerung des Codex-App-Servers    | `/codex ...` aus dem mitgelieferten Plugin `codex` |
| Eingebettete Agent-Runtime des Codex-App-Servers | Agent-Modellreferenzen `openai/*`                  |
| OpenAI Codex OAuth                      | OAuth-Profile `openai`                      |
| Claude Code oder ein anderes externes Harness   | ACP/acpx                                     |

Zur Aufteilung der Präfixe der OpenAI-Familie siehe [OpenAI](/de/providers/openai) und
[Modell-Provider](/de/concepts/model-providers). Den Supportvertrag für die Codex-Runtime
finden Sie unter [Codex-Harness-Runtime](/de/plugins/codex-harness-runtime#v1-support-contract).

## Runtime-Zuständigkeit

Verschiedene Runtimes sind für unterschiedliche Teile der Schleife zuständig:

| Oberfläche                     | Eingebettet in OpenClaw                              | Codex-App-Server                                                            |
| --------------------------- | ---------------------------------------------- | --------------------------------------------------------------------------- |
| Eigentümer der Modellschleife            | OpenClaw, über den eingebetteten OpenClaw-Runner | Codex-App-Server                                                            |
| Kanonischer Thread-Zustand      | OpenClaw-Transkript                            | Codex-Thread plus OpenClaw-Transkriptspiegelung                               |
| Dynamische OpenClaw-Tools      | Native OpenClaw-Tool-Schleife                      | Über den Codex-Adapter überbrückt                                           |
| Native Shell- und Datei-Tools | OpenClaw-Pfad                                  | Codex-native Tools, sofern unterstützt über native Hooks überbrückt            |
| Kontext-Engine              | Native OpenClaw-Kontextzusammenstellung               | OpenClaw projiziert den zusammengestellten Kontext in den Codex-Turn                     |
| Compaction                  | OpenClaw oder ausgewählte Kontext-Engine            | Codex-native Compaction mit OpenClaw-Benachrichtigungen und Spiegelpflege |
| Kanalzustellung            | OpenClaw                                       | OpenClaw                                                                    |

Designregel: Wenn OpenClaw für die Oberfläche zuständig ist, kann es das normale Verhalten von
Plugin-Hooks bereitstellen. Wenn die native Runtime für die Oberfläche zuständig ist, benötigt OpenClaw
Runtime-Ereignisse oder native Hooks. Wenn die native Runtime für den kanonischen Thread-Zustand zuständig ist,
spiegelt OpenClaw den Kontext und projiziert ihn, anstatt nicht unterstützte interne Strukturen
umzuschreiben.

## Runtime-Auswahl

OpenClaw löst eine eingebettete Runtime nach der Auflösung von Provider und Modell in
dieser Reihenfolge auf:

1. **Modellspezifische Runtime-Richtlinie** hat Vorrang. Sie befindet sich in einem konfigurierten
   Provider-Modelleintrag oder in `agents.defaults.models["provider/model"].agentRuntime`
   / `agents.entries.*.models["provider/model"].agentRuntime`. Ein Provider-Platzhalter wie `agents.defaults.models["vllm/*"].agentRuntime` greift
   nach der exakten Modellrichtlinie, sodass dynamisch erkannte Provider-Modelle
   eine Runtime gemeinsam nutzen können, ohne exakte modellspezifische Ausnahmen zu
   überschreiben.
2. **Providerspezifische Runtime-Richtlinie**: `models.providers.<provider>.agentRuntime`.
3. **Modus `auto`**: Registrierte Plugin-Runtimes können unterstützte Provider-/Modellpaare für sich beanspruchen.
4. Wenn im Modus `auto` keine Runtime den Turn übernimmt, fällt OpenClaw auf
   `openclaw` als Kompatibilitäts-Runtime zurück. Verwenden Sie eine explizite Runtime-ID, wenn
   die Ausführung strikt sein muss.

Runtime-Festlegungen für die gesamte Sitzung und den gesamten Agent werden ignoriert: `OPENCLAW_AGENT_RUNTIME`,
der Sitzungszustand `agentHarnessId`/`agentRuntimeOverride`, `agents.defaults.agentRuntime`
und `agents.entries.*.agentRuntime`. Führen Sie `openclaw doctor --fix` aus, um veraltete
Runtime-Konfigurationen für den gesamten Agent zu entfernen und veraltete Runtime-Modellreferenzen umzuwandeln, wenn die
Absicht erhalten werden kann.

Explizite Provider-/Modell-Plugin-Runtimes schlagen geschlossen fehl: `agentRuntime.id: "codex"`
für einen Provider oder ein Modell bedeutet Codex oder einen eindeutigen Auswahl-/Runtime-Fehler – es wird
niemals stillschweigend zurück zu OpenClaw geroutet. Nur `auto` darf einen nicht zugeordneten
Turn an OpenClaw weiterleiten.

CLI-Backend-Aliasse unterscheiden sich von IDs eingebetteter Harnesses. Bevorzugte Form für die Claude CLI:

```json5
{
  agents: {
    defaults: {
      model: "anthropic/claude-opus-5",
      models: {
        "anthropic/claude-opus-5": {
          agentRuntime: { id: "claude-cli" },
        },
      },
    },
  },
}
```

Veraltete Referenzen wie `claude-cli/claude-opus-4-7` werden aus
Kompatibilitätsgründen weiterhin unterstützt, neue Konfigurationen sollten jedoch Provider und Modell kanonisch halten und
das Ausführungs-Backend in der Provider-/Modell-Runtime-Richtlinie angeben.

Veraltete Referenzen vom Typ `codex-cli/*` unterscheiden sich davon: Doctor migriert sie zu `openai/*`, damit
sie über das Codex-App-Server-Harness ausgeführt werden, anstatt ein Codex-
CLI-Backend beizubehalten.

Der Modus `auto` ist für die meisten Provider bewusst konservativ. OpenAI-Agent-
Modelle sind die Ausnahme: Sowohl eine nicht gesetzte Runtime als auch `auto` werden zum Codex-
Harness aufgelöst. Eine explizite OpenClaw-Runtime-Konfiguration bleibt eine optionale Kompatibilitätsroute
für Agent-Turns vom Typ `openai/*`; zusammen mit einem ausgewählten OAuth-
Profil `openai` routet OpenClaw diesen Pfad intern über den Codex-Authentifizierungstransport,
während die öffentliche Modellreferenz `openai/*` bleibt. Veraltete OpenAI-
Runtime-Sitzungsfestlegungen werden von der Runtime-Auswahl ignoriert und können mit
`openclaw doctor --fix` bereinigt werden.

Wenn `openclaw doctor` davor warnt, dass das Plugin `codex` aktiviert ist, während noch veraltete
Codex-Modellreferenzen in der Konfiguration vorhanden sind, behandeln Sie dies als veralteten Routing-Zustand und führen Sie
`openclaw doctor --fix` aus, um ihn für die Codex-Runtime in `openai/*` umzuschreiben.

## GitHub-Copilot-Agent-Runtime

Das externe Plugin `@openclaw/copilot` registriert eine optionale `copilot`-Runtime,
die auf der GitHub Copilot CLI (`@github/copilot-sdk`) basiert. Sie beansprucht den
kanonischen Abonnement-Provider `github-copilot` und wird von
`auto` **niemals** ausgewählt. Aktivieren Sie sie pro Modell oder pro Provider über `agentRuntime.id`:

```json5
{
  agents: {
    defaults: {
      model: "github-copilot/gpt-5.5",
      models: {
        "github-copilot/gpt-5.5": {
          agentRuntime: { id: "copilot" },
        },
      },
    },
  },
}
```

Das Harness beansprucht seinen Provider, seine Runtime, seinen CLI-Sitzungsschlüssel und sein Authentifizierungsprofil-
präfix in `extensions/copilot/doctor-contract-api.ts`, das `openclaw doctor`
automatisch lädt. Informationen zur Konfiguration, Authentifizierung, Transkriptspiegelung, Compaction, zum
deklarativen Doctor-Vertrag und zur umfassenderen SDK-Entscheidung zwischen PI, Codex und Copilot
finden Sie unter [GitHub-Copilot-Agent-Runtime](/de/plugins/copilot).

## Kompatibilitätsvertrag

Wenn eine Runtime nicht OpenClaw ist, sollte ihre Dokumentation angeben, welche OpenClaw-Oberflächen
sie unterstützt:

| Frage                                  | Warum dies wichtig ist                                                                              |
| -------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Wem gehört die Modellschleife?         | Bestimmt, wo Wiederholungsversuche, Tool-Fortsetzungen und Entscheidungen über die endgültige Antwort stattfinden. |
| Wem gehört der kanonische Threadverlauf? | Bestimmt, ob OpenClaw den Verlauf bearbeiten oder nur spiegeln kann.                              |
| Funktionieren dynamische OpenClaw-Tools? | Messaging, Sitzungen, Cron und OpenClaw-eigene Tools sind darauf angewiesen.                      |
| Funktionieren dynamische Tool-Hooks?   | Plugins erwarten `before_tool_call`, `after_tool_call` und Middleware um OpenClaw-eigene Tools.   |
| Funktionieren native Tool-Hooks?       | Shell-, Patch- und Runtime-eigene Tools benötigen native Hook-Unterstützung für Richtlinien und Beobachtung. |
| Wird der Lebenszyklus der Kontext-Engine ausgeführt? | Speicher- und Kontext-Plugins sind vom Lebenszyklus für Zusammenstellung, Aufnahme, Nachbearbeitung und Compaction abhängig. |
| Welche Compaction-Daten werden bereitgestellt? | Einige Plugins benötigen nur Benachrichtigungen, andere Metadaten zu beibehaltenen und verworfenen Inhalten. |
| Was wird absichtlich nicht unterstützt? | Benutzer sollten nicht von einer Gleichwertigkeit mit OpenClaw ausgehen, wenn die native Runtime mehr Zustand verwaltet. |

Der Unterstützungsvertrag der Codex-Runtime ist unter
[Codex-Harness-Runtime](/de/plugins/codex-harness-runtime#v1-support-contract) dokumentiert.

## Statusbezeichnungen

Die Statusausgabe kann sowohl die Bezeichnungen `Execution` als auch `Runtime` anzeigen. Verstehen Sie sie als
Diagnoseangaben, nicht als Provider-Namen:

- Eine Modellreferenz wie `openai/gpt-5.6-sol` bezeichnet den ausgewählten Provider und das ausgewählte Modell.
- Eine Runtime-ID wie `codex` bezeichnet die Schleife, die den Turn ausführt.
- Eine Kanalbezeichnung wie Telegram oder Discord gibt an, wo die Unterhaltung stattfindet.

Wenn ein Lauf eine unerwartete Runtime anzeigt, prüfen Sie zuerst die Runtime-Richtlinie
des ausgewählten Providers und Modells. Veraltete Runtime-Festlegungen der Sitzung bestimmen das Routing nicht mehr.

## Verwandte Themen

- [Codex-Harness](/de/plugins/codex-harness)
- [Codex-Harness-Runtime](/de/plugins/codex-harness-runtime)
- [GitHub-Copilot-Agent-Runtime](/de/plugins/copilot)
- [OpenAI](/de/providers/openai)
- [Agent-Harness-Plugins](/de/plugins/sdk-agent-harness)
- [Agentenschleife](/de/concepts/agent-loop)
- [Modelle](/de/concepts/models)
- [Status](/de/cli/status)
