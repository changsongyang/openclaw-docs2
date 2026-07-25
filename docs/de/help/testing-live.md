---
read_when:
    - Ausführen von Smoke-Tests für Live-Modellmatrix / CLI-Backend / ACP / Medien-Provider
    - Fehlerbehebung bei der Anmeldedatenauflösung für Live-Tests
    - Hinzufügen eines neuen Provider-spezifischen Live-Tests
sidebarTitle: Live tests
summary: 'Live-Tests (mit Netzwerkzugriff): Modellmatrix, CLI-Backends, ACP, Medien-Provider, Anmeldedaten'
title: 'Tests: Live-Suiten'
x-i18n:
    generated_at: "2026-07-24T22:21:37Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: ea8279e734e3aa09dd1fa184806c925e0404edfa9acf0f682f73a4955ed90b8b
    source_path: help/testing-live.md
    workflow: 16
---

Für den Schnellstart, QA-Runner, Unit-/Integrationstest-Suites und Docker-Abläufe siehe
[Tests](/de/help/testing). Diese Seite behandelt **Live-Tests** (mit Netzwerkzugriff):
Modellmatrix, CLI-Backends, ACP, Medien-Provider und den Umgang mit Zugangsdaten.

## Live-Tests im Vergleich zu Ihrem echten Gateway

Live-Suites und Ad-hoc-Smoke-Tests dürfen niemals ein Gateway beeinträchtigen, das bereits
echten Datenverkehr verarbeitet (Ihren oder den eines anderen Betreibers):

- Eigenes Gateway verwenden: Verwenden Sie das prozessinterne Gateway (Ebene 2 unten) oder starten Sie eine
  Entwicklungsinstanz mit einem isolierten Zustandsverzeichnis (`OPENCLAW_STATE_DIR=<scratch>`) und einem
  freien Port. Binden Sie nicht den Standard-Gateway-Port (18789), während darauf ein echtes Gateway
  ausgeführt wird.
- Führen Sie `openclaw gateway stop`/`restart` (oder die entsprechenden `launchctl`/`systemctl`/tmux-Befehle)
  nicht für einen Dienst aus, den Sie in dieser Sitzung nicht gestartet haben – dabei handelt es sich um die
  Live-Instanz des Betreibers. Holen Sie zuerst eine ausdrückliche Genehmigung ein.
- Benötigen Sie realistische Daten? Kopieren Sie den Live-Zustand/die Live-Datenbank in Ihr Entwicklungs-Zustandsverzeichnis und testen Sie
  mit der Kopie. Direkte Migrationen des Zustands eines Live-Gateways erfordern ebenfalls
  eine ausdrückliche Genehmigung.

## Live: lokale Smoke-Befehle

Exportieren Sie vor Ad-hoc-Live-Prüfungen den erforderlichen Provider-Schlüssel in die
Prozessumgebung.

Sicherer Medien-Smoke-Test:

```bash
pnpm openclaw infer tts convert --local --json \
  --text "OpenClaw-Live-Smoke-Test." \
  --output /tmp/openclaw-live-smoke.mp3
```

Sicherer Smoke-Test der Anrufbereitschaft:

```bash
pnpm openclaw voicecall setup --json
pnpm openclaw voicecall smoke --to "+15555550123"
```

`voicecall smoke` ist ein Probelauf, sofern nicht auch `--yes` vorhanden ist; verwenden Sie `--yes` nur,
wenn Sie tatsächlich einen Anruf tätigen möchten. Bei Twilio, Telnyx und Plivo erfordert eine
erfolgreiche Bereitschaftsprüfung eine öffentliche Webhook-URL – lokale/private
Loopback-URLs werden abgelehnt, da diese Provider sie nicht erreichen können.

## Live: Funktionsumfang-Prüfung eines Android-Nodes

- Test: `src/gateway/android-node.capabilities.live.test.ts`
- Skript: `pnpm android:test:integration`
- Ziel: **jeden aktuell angekündigten Befehl** eines verbundenen Android-Nodes aufrufen und das Verhalten des Befehlsvertrags prüfen.
- Umfang:
  - Vorbereitete/manuelle Einrichtung (die Suite installiert, startet oder koppelt die App nicht).
  - Befehlsweise Gateway-`node.invoke`-Validierung für den ausgewählten Android-Node.
- Erforderliche Vorbereitung:
  - Die Android-App ist bereits mit dem Gateway verbunden und gekoppelt.
  - Die App bleibt im Vordergrund.
  - Berechtigungen bzw. Zustimmung zur Erfassung wurden für die Funktionen erteilt, deren erfolgreiche Prüfung Sie erwarten.
- Optionale Zielüberschreibungen:
  - `OPENCLAW_ANDROID_NODE_ID` oder `OPENCLAW_ANDROID_NODE_NAME`.
  - `OPENCLAW_ANDROID_GATEWAY_URL` / `OPENCLAW_ANDROID_GATEWAY_TOKEN` / `OPENCLAW_ANDROID_GATEWAY_PASSWORD`.
- Vollständige Details zur Android-Einrichtung: [Android-App](/de/platforms/android)

## Live: Modell-Smoke-Test (Profilschlüssel)

Live-Modelltests sind in zwei Ebenen unterteilt, damit Fehler isoliert werden:

- „Direktes Modell“ zeigt Ihnen, ob der Provider/das Modell mit dem angegebenen Schlüssel überhaupt antworten kann.
- „Gateway-Smoke-Test“ zeigt Ihnen, ob die vollständige Gateway-und-Agent-Pipeline für dieses Modell funktioniert (Sitzungen, Verlauf, Tools, Sandbox-Richtlinie usw.).

Die nachstehenden kuratierten Modelllisten befinden sich in `src/agents/live-model-filter.ts` und
ändern sich im Laufe der Zeit; betrachten Sie die dortigen Arrays als maßgebliche Quelle, nicht diese
Seite.

MiniMax M3 verwendet `minimax/MiniMax-M3` als standardmäßige Provider-/Modellreferenz.

### Ebene 1: Direkte Modellvervollständigung (ohne Gateway)

- Test: `src/agents/models.profiles.live.test.ts`
- Ziel:
  - Erkannte Modelle auflisten
  - Mit `getApiKeyForModel` Modelle auswählen, für die Sie Zugangsdaten besitzen
  - Eine kleine Vervollständigung pro Modell ausführen (sowie bei Bedarf gezielte Regressionstests)
- Aktivierung:
  - `pnpm test:live` (oder `OPENCLAW_LIVE_TEST=1`, wenn Sie Vitest direkt aufrufen)
  - Legen Sie `OPENCLAW_LIVE_MODELS=modern`, `small` oder `all` (Alias für `modern`) fest, um diese Suite tatsächlich auszuführen; andernfalls wird sie übersprungen, sodass sich `pnpm test:live` allein weiterhin auf den Gateway-Smoke-Test konzentriert.
- Modellauswahl:
  - `OPENCLAW_LIVE_MODELS=modern` führt die kuratierte Prioritätsliste mit hoher Aussagekraft aus (siehe [Live: Modellmatrix](#live-model-matrix-what-we-cover))
  - `OPENCLAW_LIVE_MODELS=small` führt die kuratierte Prioritätsliste kleiner Modelle aus
  - `OPENCLAW_LIVE_MODELS=all` ist ein Alias für `modern`
  - oder `OPENCLAW_LIVE_MODELS="openai/gpt-5.6-luna,anthropic/claude-opus-4-6,..."` (durch Kommas getrennte Positivliste)
  - Lokale Ollama-Ausführungen kleiner Modelle verwenden standardmäßig `http://127.0.0.1:11434`; legen Sie `OPENCLAW_LIVE_OLLAMA_BASE_URL` nur für LAN-, benutzerdefinierte oder Ollama-Cloud-Endpunkte fest.
  - Moderne/alle Prüfungen und Prüfungen kleiner Modelle verwenden standardmäßig die Länge ihrer kuratierten Liste als Obergrenze; legen Sie `OPENCLAW_LIVE_MAX_MODELS=0` für eine vollständige Prüfung der ausgewählten Profile oder eine positive Zahl für eine niedrigere Obergrenze fest.
  - Vollständige Prüfungen verwenden `OPENCLAW_LIVE_TEST_TIMEOUT_MS` als Zeitlimit für den gesamten direkten Modelltest. Standard: 60 Minuten.
  - Direkte Modellprüfungen werden standardmäßig mit einer Parallelität von 20 ausgeführt; legen Sie zum Überschreiben `OPENCLAW_LIVE_MODEL_CONCURRENCY` fest.
- Provider-Auswahl:
  - `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (durch Kommas getrennte Positivliste)
- Herkunft der Schlüssel:
  - Standardmäßig: Profilspeicher und Umgebungs-Fallbacks
  - Legen Sie `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` fest, um ausschließlich den **Profilspeicher** zu erzwingen
- Zweck:
  - Trennt „Provider-API ist defekt/Schlüssel ist ungültig“ von „Gateway-Agent-Pipeline ist defekt“
  - Enthält kleine, isolierte Regressionstests (Beispiel: Reasoning-Replay bei OpenAI Responses/Codex Responses und Abläufe mit Tool-Aufrufen)

### Ebene 2: Gateway- und Entwicklungsagent-Smoke-Test (was „@openclaw“ tatsächlich tut)

- Test: `src/gateway/gateway-models.profiles.live.test.ts`
- Ziel:
  - Ein prozessinternes Gateway starten
  - Eine `agent:dev:*`-Sitzung erstellen/patchen (Modellüberschreibung pro Ausführung)
  - Modelle mit Schlüsseln durchlaufen und Folgendes prüfen:
    - „aussagekräftige“ Antwort (ohne Tools)
    - ein echter Tool-Aufruf funktioniert (Leseprüfung)
    - optionale zusätzliche Tool-Prüfungen (Ausführungs- und Leseprüfung)
    - OpenAI-Regressionspfade (nur Tool-Aufruf -> Folgeanfrage) funktionieren weiterhin
- Prüfdetails (damit Sie Fehler schnell erklären können):
  - `read`-Prüfung: Der Test schreibt eine Nonce-Datei in den Arbeitsbereich und fordert den Agenten auf, sie mit `read` zu lesen und die Nonce zurückzugeben.
  - `exec+read`-Prüfung: Der Test fordert den Agenten auf, mit `exec` eine Nonce in eine temporäre Datei zu schreiben und sie anschließend mit `read` zurückzulesen.
  - Bildprüfung: Der Test hängt eine generierte PNG-Datei (Katze + zufälliger Code) an und erwartet, dass das Modell `cat <CODE>` zurückgibt.
  - Implementierungsreferenz: `src/gateway/gateway-models.profiles.live.test.ts` und `test/helpers/live-image-probe.ts`.
- Aktivierung:
  - `pnpm test:live` (oder `OPENCLAW_LIVE_TEST=1`, wenn Sie Vitest direkt aufrufen)
- Modellauswahl:
  - Standard: die kuratierte `modern`-Prioritätsliste mit hoher Aussagekraft
  - `OPENCLAW_LIVE_GATEWAY_MODELS=small` führt die kuratierte Liste kleiner Modelle durch die vollständige Gateway-und-Agent-Pipeline aus
  - `OPENCLAW_LIVE_GATEWAY_MODELS=all` ist ein Alias für `modern`
  - Oder legen Sie zur Einschränkung `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (oder eine durch Kommas getrennte Liste) fest
  - Moderne/alle Gateway-Prüfungen und Gateway-Prüfungen kleiner Modelle verwenden standardmäßig die Länge ihrer kuratierten Liste als Obergrenze; legen Sie `OPENCLAW_LIVE_GATEWAY_MAX_MODELS=0` für eine vollständige ausgewählte Prüfung oder eine positive Zahl für eine niedrigere Obergrenze fest.
- Provider-Auswahl (vermeidet „alles über OpenRouter“):
  - `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (durch Kommas getrennte Positivliste)
- Tool- und Bildprüfungen sind in diesem Live-Test immer aktiviert:
  - `read`-Prüfung + `exec+read`-Prüfung (Tool-Belastung)
  - Die Bildprüfung wird ausgeführt, wenn das Modell die Unterstützung von Bildeingaben angibt
  - Ablauf (Übersicht):
    - Der Test generiert eine kleine PNG-Datei mit „CAT“ und einem zufälligen Code (`test/helpers/live-image-probe.ts`)
    - Sendet sie über `agent` `attachments: [{ mimeType: "image/png", content: "<base64>" }]`
    - Das Gateway verarbeitet Anhänge zu `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
    - Der eingebettete Agent leitet eine multimodale Benutzernachricht an das Modell weiter
    - Prüfung: Die Antwort enthält `cat` und den Code (OCR-Toleranz: geringfügige Fehler sind zulässig)

<Tip>
Um zu sehen, was Sie auf Ihrem Rechner testen können (und welche exakten `provider/model`-IDs verfügbar sind), führen Sie Folgendes aus:

```bash
openclaw models list
openclaw models list --json
```

</Tip>

## Live: CLI-Backend-Smoke-Test (Claude, Gemini oder andere lokale CLIs)

- Test: `src/gateway/gateway-cli-backend.live.test.ts`
- Ziel: Die Gateway-und-Agent-Pipeline mit einem lokalen CLI-Backend validieren, ohne Ihre Standardkonfiguration zu verändern.
- Backend-spezifische Smoke-Standardeinstellungen befinden sich in der `cli-backend.ts`-Definition des zuständigen Plugins.
- Aktivierung:
  - `pnpm test:live` (oder `OPENCLAW_LIVE_TEST=1`, wenn Sie Vitest direkt aufrufen)
  - `OPENCLAW_LIVE_CLI_BACKEND=1`
- Standardeinstellungen:
  - Standard-Provider/-Modell: `claude-cli/claude-sonnet-4-6`
  - Befehls-, Argument- und Bildverhalten stammen aus den Metadaten des zuständigen CLI-Backend-Plugins.
- Überschreibungen (optional):
  - `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-6"`
  - `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
  - `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json"]'`
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1`, um einen echten Bildanhang zu senden (Pfade werden in den Prompt eingefügt). In Docker-Rezepten standardmäßig deaktiviert.
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"`, um Bilddateipfade als CLI-Argumente statt durch Einfügen in den Prompt zu übergeben.
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (oder `"list"`), um zu steuern, wie Bildargumente übergeben werden, wenn `IMAGE_ARG` festgelegt ist.
  - `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1`, um eine zweite Gesprächsrunde zu senden und den Fortsetzungsablauf zu validieren.
  - `OPENCLAW_LIVE_CLI_BACKEND_MODEL_SWITCH_PROBE=1`, um die sitzungsinterne Kontinuitätsprüfung Claude Sonnet -> Opus zu aktivieren, wenn das ausgewählte Modell ein Wechselziel unterstützt. Standardmäßig deaktiviert, auch in Docker-Rezepten.
  - `OPENCLAW_LIVE_CLI_BACKEND_MCP_PROBE=1`, um die MCP-/Tool-Loopback-Prüfung zu aktivieren. In Docker-Rezepten standardmäßig deaktiviert.

Beispiel:

```bash
  OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-6" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

Kostengünstiger Smoke-Test der Gemini-MCP-Konfiguration:

```bash
OPENCLAW_LIVE_TEST=1 \
  pnpm test:live src/agents/cli-runner/bundle-mcp.gemini.live.test.ts
```

Dabei wird Gemini nicht aufgefordert, eine Antwort zu generieren. Der Test schreibt dieselben System-
einstellungen, die OpenClaw an Gemini übergibt, und führt anschließend `gemini --debug mcp list` aus, um nachzuweisen, dass ein
gespeicherter `transport: "streamable-http"`-Server in Geminis HTTP-MCP-
Format normalisiert wird und eine Verbindung zu einem lokalen streamfähigen HTTP-MCP-Server herstellen kann.

Docker-Rezept:

```bash
pnpm test:docker:live-cli-backend
```

Docker-Rezepte für einzelne Provider:

```bash
pnpm test:docker:live-cli-backend:claude
pnpm test:docker:live-cli-backend:claude-subscription
pnpm test:docker:live-cli-backend:gemini
```

Hinweise:

- Der Docker-Runner befindet sich unter `scripts/test-live-cli-backend-docker.sh`.
- Er führt den Live-Smoke-Test des CLI-Backends innerhalb des Docker-Images des Repositorys als Nicht-root-Benutzer `node` aus.
- Er ermittelt die CLI-Smoke-Test-Metadaten aus dem zuständigen Plugin und installiert anschließend das passende Linux-CLI-Paket (`@anthropic-ai/claude-code` oder `@google/gemini-cli`) in einem zwischengespeicherten, beschreibbaren Präfix unter `OPENCLAW_DOCKER_CLI_TOOLS_DIR` (Standard: `~/.cache/openclaw/docker-cli-tools`).
- `codex-cli` ist kein gebündeltes CLI-Backend mehr; verwenden Sie stattdessen `openai/*` mit der Codex-App-Server-Runtime (siehe [Live: Smoke-Test des Codex-App-Server-Harnesses](#live-codex-app-server-harness-smoke)).
- `pnpm test:docker:live-cli-backend:claude-subscription` erfordert portables OAuth für das Claude-Code-Abonnement, entweder über `~/.claude/.credentials.json` mit `claudeAiOauth.subscriptionType` oder über `CLAUDE_CODE_OAUTH_TOKEN` aus `claude setup-token`. Zunächst wird der direkte Aufruf von `claude -p` in Docker nachgewiesen, anschließend werden zwei Turns des Gateway-CLI-Backends ausgeführt, ohne Umgebungsvariablen für Anthropic-API-Schlüssel beizubehalten. Dieser Abonnement-Testpfad deaktiviert die Claude-MCP-/Tool- und Bildprüfungen standardmäßig, da er die Nutzungslimits des angemeldeten Abonnements beansprucht und Anthropic das Abrechnungs- und Ratenbegrenzungsverhalten des Claude Agent SDK bzw. von `claude -p` ohne ein OpenClaw-Release ändern kann.
- Claude und Gemini unterstützen über die obigen Flags denselben Prüfsatz (Text-Turn, Bildklassifizierung, MCP-Tool-Aufruf `cron`, Kontinuität beim Modellwechsel), aber keine dieser Prüfungen wird standardmäßig ausgeführt – aktivieren Sie sie bei Bedarf jeweils über das entsprechende Flag.

## Live: Erreichbarkeit des APNs-HTTP/2-Proxys

- Test: `src/infra/push-apns-http2.live.test.ts`
- Ziel: einen Tunnel durch einen lokalen HTTP-CONNECT-Proxy zum Sandbox-APNs-Endpunkt von Apple aufzubauen, die APNs-HTTP/2-Validierungsanfrage zu senden und sicherzustellen, dass Apples tatsächliche Antwort `403 InvalidProviderToken` über den Proxy-Pfad zurückkommt.
- Aktivieren:
  - `OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_APNS_REACHABILITY=1 pnpm test:live src/infra/push-apns-http2.live.test.ts`
- Optionales Zeitlimit:
  - `OPENCLAW_LIVE_APNS_TIMEOUT_MS=30000`

## Live: ACP-Bind-Smoke-Test (`/acp spawn ... --bind here`)

- Test: `src/gateway/gateway-acp-bind.live.test.ts`
- Ziel: den tatsächlichen ACP-Ablauf zur Konversationsbindung mit einem aktiven ACP-Agenten zu validieren:
  - `/acp spawn <agent> --bind here` senden
  - eine synthetische Nachrichtenkanal-Konversation direkt binden
  - eine normale Folgenachricht in derselben Konversation senden
  - überprüfen, dass die Folgenachricht im Transkript der gebundenen ACP-Sitzung ankommt
- Aktivieren:
  - `pnpm test:live src/gateway/gateway-acp-bind.live.test.ts`
  - `OPENCLAW_LIVE_ACP_BIND=1`
- Standardwerte:
  - ACP-Agenten in Docker: `claude,codex,gemini`
  - ACP-Agent für direkten Aufruf von `pnpm test:live ...`: `claude`
  - Synthetischer Kanal: Konversationskontext im Stil einer Slack-Direktnachricht
  - ACP-Backend: `acpx`
- Überschreibungen:
  - `OPENCLAW_LIVE_ACP_BIND_AGENT=claude`
  - `OPENCLAW_LIVE_ACP_BIND_AGENT=codex`
  - `OPENCLAW_LIVE_ACP_BIND_AGENT=droid`
  - `OPENCLAW_LIVE_ACP_BIND_AGENT=gemini`
  - `OPENCLAW_LIVE_ACP_BIND_AGENT=opencode`
  - `OPENCLAW_LIVE_ACP_BIND_AGENTS=claude,codex,gemini`
  - `OPENCLAW_LIVE_ACP_BIND_AGENT_COMMAND='npx -y @agentclientprotocol/claude-agent-acp@<version>'`
  - `OPENCLAW_LIVE_ACP_BIND_CODEX_MODEL=gpt-5.6-luna`
  - `OPENCLAW_LIVE_ACP_BIND_OPENCODE_MODEL=opencode/kimi-k2.6`
  - `OPENCLAW_LIVE_ACP_BIND_IMAGE_PROBE=1` (oder `on`/`true`/`yes`), um die Bildprüfung zu erzwingen; jeder andere Wert deaktiviert sie. Sie wird standardmäßig für jeden Agenten außer `opencode` ausgeführt.
  - `OPENCLAW_LIVE_ACP_BIND_REQUIRE_CRON=1`
  - `OPENCLAW_LIVE_ACP_BIND_PARENT_MODEL=openai/gpt-5.6-luna`
- Hinweise:
  - Dieser Testpfad verwendet die Gateway-Oberfläche `chat.send` mit ausschließlich Administratoren vorbehaltenen synthetischen Feldern für die Ursprungsroute, damit Tests einen Nachrichtenkanal-Kontext anhängen können, ohne eine externe Zustellung vorzutäuschen.
  - Wenn `OPENCLAW_LIVE_ACP_BIND_AGENT_COMMAND` nicht gesetzt ist, verwendet der Test die integrierte Agentenregistrierung des eingebetteten Plugins `acpx` für den ausgewählten ACP-Harness-Agenten.
  - Die Erstellung eines Cron-MCP für die gebundene Sitzung erfolgt standardmäßig nach bestem Bemühen, da externe ACP-Harnesse MCP-Aufrufe abbrechen können, nachdem der Bindungs-/Bildnachweis erfolgreich war; setzen Sie `OPENCLAW_LIVE_ACP_BIND_REQUIRE_CRON=1`, um diese Cron-Prüfung nach der Bindung strikt zu machen.

Beispiel:

```bash
OPENCLAW_LIVE_ACP_BIND=1 \
  OPENCLAW_LIVE_ACP_BIND_AGENT=claude \
  pnpm test:live src/gateway/gateway-acp-bind.live.test.ts
```

Docker-Rezept:

```bash
pnpm test:docker:live-acp-bind
```

Docker-Rezepte für einzelne Agenten:

```bash
pnpm test:docker:live-acp-bind:claude
pnpm test:docker:live-acp-bind:codex
pnpm test:docker:live-acp-bind:droid
pnpm test:docker:live-acp-bind:gemini
pnpm test:docker:live-acp-bind:opencode
```

Docker-Hinweise:

- Der Docker-Runner befindet sich unter `scripts/test-live-acp-bind-docker.sh`.
- Standardmäßig führt er den ACP-Bind-Smoke-Test nacheinander mit den zusammengefassten Live-CLI-Agenten aus: `claude`, `codex`, dann `gemini`.
- Verwenden Sie `OPENCLAW_LIVE_ACP_BIND_AGENTS=claude`, `OPENCLAW_LIVE_ACP_BIND_AGENTS=codex`, `OPENCLAW_LIVE_ACP_BIND_AGENTS=droid`, `OPENCLAW_LIVE_ACP_BIND_AGENTS=gemini` oder `OPENCLAW_LIVE_ACP_BIND_AGENTS=opencode`, um die Matrix einzugrenzen.
- Er stellt das passende CLI-Authentifizierungsmaterial im Container bereit und installiert anschließend bei Bedarf die angeforderte Live-CLI (`@anthropic-ai/claude-code`, `@openai/codex`, Factory Droid über `https://app.factory.ai/cli`, `@google/gemini-cli` oder `opencode-ai`). Das ACP-Backend selbst ist das eingebettete Paket `acpx/runtime` aus dem offiziellen Plugin `acpx`.
- Die Droid-Docker-Variante stellt `~/.factory` für Einstellungen bereit, leitet `FACTORY_API_KEY` weiter und benötigt diesen API-Schlüssel, da die lokale Factory-OAuth-/Schlüsselbund-Authentifizierung nicht portabel in den Container übertragen werden kann. Sie verwendet den integrierten Registrierungseintrag `droid exec --output-format acp` von ACPX.
- Die OpenCode-Docker-Variante ist ein strikter Regressions-Testpfad für einen einzelnen Agenten. Sie schreibt ein temporäres Standardmodell `OPENCODE_CONFIG_CONTENT` aus `OPENCLAW_LIVE_ACP_BIND_OPENCODE_MODEL` (Standard: `opencode/kimi-k2.6`).
- Direkte Aufrufe der CLI `acpx` dienen nur als manueller Ausweichpfad zum Vergleichen des Verhaltens außerhalb des Gateways. Der Docker-ACP-Bind-Smoke-Test führt das eingebettete Runtime-Backend `acpx` von OpenClaw aus.

## Live: Smoke-Test des Codex-App-Server-Harnesses

- Ziel: den Plugin-eigenen Codex-Harness über die normale Gateway-Methode
  `agent` zu validieren:
  - das gebündelte Plugin `codex` laden
  - über `/model <ref> --runtime codex` ein OpenAI-Modell auswählen
  - einen ersten Gateway-Agentendurchlauf mit der angeforderten Denkstufe senden
  - einen zweiten Durchlauf an dieselbe OpenClaw-Sitzung senden und überprüfen, dass der
    App-Server-Thread fortgesetzt werden kann
  - `/codex status` und `/codex models` über denselben Gateway-Befehlspfad
    ausführen
  - optional zwei von Guardian geprüfte Shell-Prüfungen mit erhöhten Berechtigungen ausführen: einen harmlosen
    Befehl, der genehmigt werden sollte, und einen Upload mit vorgetäuschtem Geheimnis, der
    abgelehnt werden sollte, sodass der Agent rückfragt
- Test: `src/gateway/gateway-codex-harness.live.test.ts`
- Aktivieren: `OPENCLAW_LIVE_CODEX_HARNESS=1`
- Basismodell des Harnesses: `openai/gpt-5.6-luna`
- Standardauswahl für einen neuen OpenAI-API-Schlüssel: `openai/gpt-5.6`
- Standardmäßige Denkstufe: `low`
- Modellüberschreibung: `OPENCLAW_LIVE_CODEX_HARNESS_MODEL=openai/<model>`
- Überschreibung der Denkstufe: `OPENCLAW_LIVE_CODEX_HARNESS_THINKING=<level>`
- Aufwandsprüfung für ein vom Standard abweichendes Modell:
  `OPENCLAW_LIVE_CODEX_HARNESS_EXPECTED_EFFORT=<level>`
- Matrixüberschreibung: `OPENCLAW_LIVE_CODEX_HARNESS_TARGETS=<model>=<thinking>,...`
- Authentifizierungsmodus: `OPENCLAW_LIVE_CODEX_HARNESS_AUTH=codex-auth` (Standard) verwendet die
  kopierte Codex-Anmeldung; `api-key` verwendet `OPENAI_API_KEY` über den Codex-App-Server.
- Optionale Bildprüfung: `OPENCLAW_LIVE_CODEX_HARNESS_IMAGE_PROBE=1`
- Optionale MCP-/Tool-Prüfung: `OPENCLAW_LIVE_CODEX_HARNESS_MCP_PROBE=1`
- Optionale Guardian-Prüfung: `OPENCLAW_LIVE_CODEX_HARNESS_GUARDIAN_PROBE=1`
- Optionaler Fortsetzungs-Stresstest: `OPENCLAW_LIVE_CODEX_HARNESS_RESUME_STRESS=1` fügt
  vier Verlaufsdurchläufe hinzu, schließt und startet anschließend das Gateway und den Codex-App-Server
  dreimal neu und verlangt dabei dieselbe native Thread-ID und denselben
  Konversationsverlauf. Überschreiben Sie die begrenzten Anzahlen mit
  `OPENCLAW_LIVE_CODEX_HARNESS_RESUME_STRESS_HISTORY_TURNS` (1-20) und
  `OPENCLAW_LIVE_CODEX_HARNESS_RESUME_STRESS_RESTARTS` (1-10).
- Optionaler Fan-out-Stresstest: Setzen Sie `OPENCLAW_LIVE_CODEX_HARNESS_SUBAGENT_PROBE=1`
  und `OPENCLAW_LIVE_CODEX_HARNESS_SUBAGENT_COUNT` (1-12). Der Harness startet
  alle untergeordneten Agenten gleichzeitig, wartet auf jeden abgeschlossenen Lauf und überprüft jede
  eindeutige Antwort der untergeordneten Agenten sowie deren native Thread-Identität.
- Optionaler Compaction-Stresstest: `OPENCLAW_LIVE_CODEX_HARNESS_COMPACTION_STRESS=1`
  erzeugt begrenzte native Tool-Ausgaben, verlangt automatische Compaction-Ereignisse,
  überprüft die persistierte Compaction-Anzahl sowie das Erinnern verborgener Markierungen, startet
  das Gateway und den physischen Codex-App-Server neu und wiederholt anschließend die Ausgabe- und
  Compaction-Welle. Passen Sie den begrenzten Arbeitsumfang mit
  `OPENCLAW_LIVE_CODEX_HARNESS_COMPACTION_STRESS_TURNS` (1-8) und
  `OPENCLAW_LIVE_CODEX_HARNESS_LARGE_OUTPUT_BYTES` (100000-800000) an.
- Vollständiger Direkt-API-Kontext: `OPENCLAW_LIVE_CODEX_HARNESS_FULL_CONTEXT=1` wendet
  die Kontextgrenze `922000` und die gesamte Compaction-Grenze `700000` an, sendet dichte, begrenzte
  Benutzerdurchläufe, führt pro Welle zwei explizite native Compaction-Prüfpunkte aus und
  fährt nach jedem Prüfpunkt mit weiteren Durchläufen fort. Dies erfordert
  `OPENCLAW_LIVE_CODEX_HARNESS_AUTH=api-key` sowie einen absoluten
  Pfad `OPENCLAW_LIVE_CODEX_HARNESS_MODEL_CATALOG`. Der Katalog muss das
  ausgewählte Modell mit `max_context_window: 922000` bereitstellen, damit Codex die
  Überschreibung nicht wieder auf sein normales Katalogfenster begrenzt. Der gewöhnliche Stresstest mit reduzierten Schwellenwerten
  oben behält die strengeren Prüfungen der automatischen Compaction und der
  Beibehaltung verborgener Markierungen bei.
- Optionale Prüfung zum Deaktivieren der Schleifenweiterleitung:
  `OPENCLAW_LIVE_CODEX_HARNESS_DISABLE_LOOP_RELAY=1`
- Die angeforderte Denkpräferenz kann dem nächstgelegenen Aufwand zugeordnet werden,
  den Codex für dieses Modell ausweist. Luna ordnet beispielsweise `minimal` `low` zu.
- Bekannte Codex-Katalogmodelle leiten diesen exakten nativen Aufwand automatisch ab.
  Bei Überschreibungen mit unbekannten Modellen muss der erwartete zugeordnete Aufwand angegeben werden.
- Der Smoke-Test erzwingt Provider/Modell `agentRuntime.id: "codex"`, damit ein defekter Codex-
  Harness nicht durch einen stillen Rückfall auf OpenClaw bestehen kann.
- Authentifizierung: Codex-App-Server-Authentifizierung aus der lokalen Codex-Abonnementanmeldung oder
  `OPENAI_API_KEY`, wenn `OPENCLAW_LIVE_CODEX_HARNESS_AUTH=api-key`. Docker kann
  `~/.codex/auth.json` und `~/.codex/config.toml` für Abonnementdurchläufe kopieren.

Lokales Rezept:

```bash
OPENCLAW_LIVE_CODEX_HARNESS=1 \
  OPENCLAW_LIVE_CODEX_HARNESS_IMAGE_PROBE=1 \
  OPENCLAW_LIVE_CODEX_HARNESS_MCP_PROBE=1 \
  OPENCLAW_LIVE_CODEX_HARNESS_GUARDIAN_PROBE=1 \
  OPENCLAW_LIVE_CODEX_HARNESS_MODEL=openai/gpt-5.6-luna \
  pnpm test:live -- src/gateway/gateway-codex-harness.live.test.ts
```

Docker-Rezept:

```bash
pnpm test:docker:live-codex-harness
```

Neustart- und Verlaufs-Stresstest:

```bash
OPENCLAW_LIVE_CODEX_HARNESS_RESUME_STRESS=1 \
pnpm test:docker:live-codex-harness
```

Fan-out-, Großausgabe-, Compaction- und Neustart-Stresstest:

```bash
OPENCLAW_LIVE_CODEX_HARNESS_AUTH=api-key \
  OPENCLAW_LIVE_CODEX_HARNESS_SUBAGENT_PROBE=1 \
  OPENCLAW_LIVE_CODEX_HARNESS_SUBAGENT_COUNT=8 \
  OPENCLAW_LIVE_CODEX_HARNESS_RESUME_STRESS=1 \
  OPENCLAW_LIVE_CODEX_HARNESS_COMPACTION_STRESS=1 \
  pnpm test:docker:live-codex-harness
```

Vollständiger nativer Codex-Compaction-Stresstest für das Eingabebudget `922000`:

```bash
OPENCLAW_LIVE_CODEX_HARNESS=1 \
  OPENCLAW_LIVE_CODEX_HARNESS_AUTH=api-key \
  OPENCLAW_LIVE_CODEX_HARNESS_FULL_CONTEXT=1 \
  OPENCLAW_LIVE_CODEX_HARNESS_MODEL_CATALOG=/absolute/path/to/models-api-1m.json \
  OPENCLAW_LIVE_CODEX_HARNESS_MODEL=openai/gpt-5.6-terra \
  OPENCLAW_LIVE_CODEX_HARNESS_THINKING=medium \
  OPENCLAW_LIVE_CODEX_HARNESS_COMPACTION_STRESS_TURNS=8 \
  OPENCLAW_LIVE_CODEX_HARNESS_LARGE_OUTPUT_BYTES=800000 \
  pnpm test:live -- src/gateway/gateway-codex-harness.live.test.ts
```

Native Codex-Matrix für GPT-5.6:

```bash
OPENCLAW_LIVE_CODEX_HARNESS_AUTH=api-key \
  OPENCLAW_LIVE_CODEX_HARNESS_TARGETS='openai/gpt-5.6-sol=ultra,openai/gpt-5.6-terra=ultra,openai/gpt-5.6-luna=max' \
  pnpm test:docker:live-codex-harness
```

## Live: Wiederholte OpenAI-Compaction

- Ziel: die eingebettete OpenClaw-`openai-responses`-Agentenschleife über
  mindestens zwei echte automatische Compactions ausführen und anschließend prüfen, ob eine dauerhafte Markierung erhalten bleibt.
- Test: `src/agents/sessions/agent-session.openai-compaction.live.test.ts`
- Aktivieren: `OPENCLAW_LIVE_OPENAI_COMPACTION=1`
- Standardmodell: `gpt-5.6-luna`
- Modellüberschreibung: `OPENCLAW_LIVE_OPENAI_COMPACTION_MODEL=<model>`
- Der normale Belastungsmodus verwendet ein reduziertes Client-Kontextbudget, um denselben
  echten Compaction-Pfad mit begrenzten API-Ausgaben zu erreichen.
- Der Vollkontextmodus setzt das Client-Budget auf `922000` und die Compaction-Reserve auf
  `222000`, sodass die automatische Compaction bei `700000` beginnt. Außerdem ist eine
  beobachtete Provider-Eingabeanzahl oberhalb der Long-Context-Preisgrenze von `272000` erforderlich.

Begrenztes Live-Rezept:

```bash
OPENCLAW_LIVE_TEST=1 \
  OPENCLAW_LIVE_OPENAI_COMPACTION=1 \
  pnpm test:live -- src/agents/sessions/agent-session.openai-compaction.live.test.ts
```

Rezept mit vollständigem `922000`-Eingabebudget:

```bash
OPENCLAW_LIVE_TEST=1 \
  OPENCLAW_LIVE_OPENAI_COMPACTION=1 \
  OPENCLAW_LIVE_OPENAI_COMPACTION_FULL=1 \
  OPENCLAW_LIVE_OPENAI_COMPACTION_MODEL=gpt-5.6-terra \
  pnpm test:live -- src/agents/sessions/agent-session.openai-compaction.live.test.ts
```

<Warning>
Der Vollmodus überschreitet absichtlich die Long-Context-Preisgrenze von OpenAI und
kann mehrere große API-Aufrufe ausführen. Verwenden Sie ihn nur mit ausdrücklicher Ausgabengenehmigung.
</Warning>

Standard für einen neuen OpenAI-API-Schlüssel:

```bash
OPENCLAW_LIVE_GATEWAY_OPENAI_API_DEFAULT=1 \
  OPENCLAW_LIVE_GATEWAY_PROVIDERS=openai \
  OPENCLAW_LIVE_GATEWAY_THINKING=off \
  pnpm test:live -- src/gateway/gateway-models.profiles.live.test.ts
```

Dieser Nachweis lässt `OPENCLAW_LIVE_GATEWAY_MODELS` nicht gesetzt, löst das Modell über
die Inferenz-Auswahl-Schnittstelle des neuen Onboardings auf, prüft `openai/gpt-5.6` und führt anschließend
einen echten Gateway-Durchlauf mit diesem aufgelösten Modell aus.

GPT-5.6-Matrix für eingebettetes OpenClaw:

```bash
OPENCLAW_LIVE_GATEWAY_THINKING=ultra \
  OPENCLAW_LIVE_GATEWAY_PROVIDERS=openai \
  OPENCLAW_LIVE_GATEWAY_MODELS='openai/gpt-5.6-sol,openai/gpt-5.6-terra,openai/gpt-5.6-luna' \
  pnpm test:live -- src/gateway/gateway-models.profiles.live.test.ts
```

Docker-Hinweise:

- Der Docker-Runner befindet sich unter `scripts/test-live-codex-harness-docker.sh`.
- Er übergibt `OPENAI_API_KEY`, kopiert vorhandene Authentifizierungsdateien der Codex CLI, installiert
  `@openai/codex` in ein beschreibbares, eingebundenes npm-
  Präfix, stellt den Quellbaum bereit und führt anschließend ausschließlich den Live-Test des Codex-Harnesses aus.
- Docker aktiviert standardmäßig die Image-, MCP/Tool- und Guardian-Prüfungen. Setzen Sie
  `OPENCLAW_LIVE_CODEX_HARNESS_IMAGE_PROBE=0` oder
  `OPENCLAW_LIVE_CODEX_HARNESS_MCP_PROBE=0` oder
  `OPENCLAW_LIVE_CODEX_HARNESS_GUARDIAN_PROBE=0`, wenn Sie einen enger eingegrenzten Debug-
  Durchlauf benötigen.
- Docker verwendet dieselbe explizite Codex-Laufzeitkonfiguration, sodass veraltete Aliasse oder ein OpenClaw-
  Fallback keine Regression des Codex-Harnesses verbergen können.
- Matrixziele werden nacheinander in einem Container ausgeführt. Das Docker-Skript skaliert sein
  standardmäßiges Zeitlimit von 35 Minuten anhand der Anzahl der Ziele; jedes äußere Shell- oder CI-Zeitlimit muss
  dieselbe Gesamtdauer zulassen. Die kanonische CI führt jedes GPT-5.6-Ziel in einem separaten Shard aus.

### Empfohlene Live-Rezepte

Eng gefasste, explizite Zulassungslisten sind am schnellsten und am wenigsten fehleranfällig:

- Einzelnes Modell, direkt (ohne Gateway):
  - `OPENCLAW_LIVE_MODELS="openai/gpt-5.6-luna" pnpm test:live src/agents/models.profiles.live.test.ts`

- Direktes Profil für kleine Modelle:
  - `OPENCLAW_LIVE_MODELS=small pnpm test:live src/agents/models.profiles.live.test.ts`

- Gateway-Profil für kleine Modelle:
  - `OPENCLAW_LIVE_GATEWAY_MODELS=small pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

- Ollama-Cloud-API-Smoke-Test:
  - `OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_OLLAMA=1 OPENCLAW_LIVE_OLLAMA_BASE_URL=https://ollama.com OPENCLAW_LIVE_OLLAMA_MODEL=glm-5.1:cloud OPENCLAW_LIVE_OLLAMA_WEB_SEARCH=0 pnpm test:live -- extensions/ollama/ollama.live.test.ts`

- Einzelnes Modell, Gateway-Smoke-Test:
  - `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.6-luna" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

- Tool-Aufrufe über mehrere Provider hinweg:
  - `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.6-luna,anthropic/claude-opus-4-6,google/gemini-3.5-flash,deepseek/deepseek-v4-flash,zai/glm-5.1,minimax/MiniMax-M3" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

- Direkter Smoke-Test für den Z.AI Coding Plan GLM-5.2:
  - `ZAI_CODING_LIVE_TEST=1 pnpm test:live src/agents/zai.live.test.ts`

- Google-Schwerpunkt (Gemini-API-Schlüssel + Antigravity):
  - Gemini (API-Schlüssel): `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3.5-flash" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
  - Antigravity (OAuth): `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

- Google-Smoke-Test für adaptives Denken (`qa manual` aus der privaten QA-CLI – erfordert `OPENCLAW_ENABLE_PRIVATE_QA_CLI=1` und einen Quellcode-Checkout; siehe [QA-Übersicht](/de/concepts/qa-e2e-automation)):
  - Dynamischer Standard für Gemini 3: `OPENCLAW_ENABLE_PRIVATE_QA_CLI=1 pnpm openclaw qa manual --provider-mode live-frontier --model google/gemini-3.1-pro-preview --alt-model google/gemini-3.1-pro-preview --message '/think adaptive Reply exactly: GEMINI_ADAPTIVE_OK' --timeout-ms 180000`
  - Dynamisches Budget für Gemini 2.5: `OPENCLAW_ENABLE_PRIVATE_QA_CLI=1 pnpm openclaw qa manual --provider-mode live-frontier --model google/gemini-2.5-flash --alt-model google/gemini-2.5-flash --message '/think adaptive Reply exactly: GEMINI25_ADAPTIVE_OK' --timeout-ms 180000`

Hinweise:

- `google/...` verwendet die Gemini API (API-Schlüssel).
- `google-antigravity/...` verwendet die Antigravity-OAuth-Bridge (Agentenendpunkt im Stil von Cloud Code Assist).
- `google-gemini-cli/...` verwendet die lokale Gemini CLI auf Ihrem Rechner (separate Authentifizierung und Besonderheiten der Tools).
- Gemini API im Vergleich zur Gemini CLI:
  - API: OpenClaw ruft Googles gehostete Gemini API über HTTP auf (API-Schlüssel/Profilauthentifizierung); dies ist das, was die meisten Benutzer mit „Gemini“ meinen.
  - CLI: OpenClaw führt eine lokale `gemini`-Binärdatei über die Shell aus; sie verfügt über eine eigene Authentifizierung und kann sich anders verhalten (Streaming-/Tool-Unterstützung und Versionsabweichungen).

## Live: Modellmatrix (was abgedeckt wird)

Live ist optional, daher gibt es keine feste „CI-Modellliste“. `OPENCLAW_LIVE_MODELS=modern` / `OPENCLAW_LIVE_GATEWAY_MODELS=modern` (und ihr Alias `all`) führen die kuratierte Prioritätsliste aus `HIGH_SIGNAL_LIVE_MODEL_PRIORITY` in `src/agents/live-model-filter.ts` in dieser Prioritätsreihenfolge aus:

| Provider/Modell                                | Hinweise   |
| --------------------------------------------- | ---------- |
| `anthropic/claude-opus-5`                     |            |
| `anthropic/claude-opus-4-8`                   |            |
| `anthropic/claude-sonnet-5`                   |            |
| `anthropic/claude-sonnet-4-6`                 |            |
| `anthropic/claude-opus-4-7`                   |            |
| `google/gemini-3.1-pro-preview`               | Gemini API |
| `google/gemini-3.5-flash`                     | Gemini API |
| `cohere/command-a-plus-05-2026`               |            |
| `moonshot/kimi-k3`                            |            |
| `anthropic/claude-opus-4-6`                   |            |
| `deepseek/deepseek-v4-flash`                  |            |
| `deepseek/deepseek-v4-pro`                    |            |
| `minimax/MiniMax-M3`                          |            |
| `openai/gpt-5.5`                              |            |
| `openrouter/openai/gpt-5.2-chat`              |            |
| `openrouter/minimax/minimax-m2.7`             |            |
| `opencode-go/glm-5`                           |            |
| `openrouter/ai21/jamba-large-1.7`             |            |
| `xai/grok-4.5`                                |            |
| `xai/grok-4.20-0309-reasoning`                |            |
| `zai/glm-5.1`                                 |            |
| `fireworks/accounts/fireworks/models/glm-5p1` |            |
| `minimax-portal/minimax-m3`                   |            |

Die kuratierte Liste **kleiner Modelle** (`OPENCLAW_LIVE_MODELS=small` / `OPENCLAW_LIVE_GATEWAY_MODELS=small`) aus `SMALL_LIVE_MODEL_PRIORITY`:

| Provider/Modell               |
| ---------------------------- |
| `lmstudio/qwen/qwen3.5-9b`   |
| `vllm/qwen/qwen3-8b`         |
| `sglang/qwen/qwen3-8b`       |
| `ollama/gemma3:4b`           |
| `openrouter/qwen/qwen3.5-9b` |
| `openrouter/z-ai/glm-5.1`    |
| `openrouter/z-ai/glm-5`      |
| `zai/glm-5.1`                |

Hinweise zur modernen Liste:

- Die Provider `codex` und `codex-cli` sind vom standardmäßigen modernen Durchlauf ausgeschlossen (sie decken das Verhalten des CLI-Backends/ACP ab, das oben separat getestet wird). `openai/gpt-5.5` selbst wird standardmäßig über den Codex-App-Server-Harness geleitet; siehe [Live: Smoke-Test des Codex-App-Server-Harnesses](#live-codex-app-server-harness-smoke).
- `fireworks`, `google`, `openrouter` und `xai` führen im modernen Durchlauf nur ihre explizit kuratierten Modell-IDs aus (keine automatische Erweiterung auf „jedes Modell dieses Providers“).
- Nehmen Sie mindestens ein bildfähiges Modell (Vision-Varianten der Claude-/Gemini-/OpenAI-Familie usw.) in `OPENCLAW_LIVE_GATEWAY_MODELS` auf, um die Bildprüfung auszuführen.

Führen Sie den Gateway-Smoke-Test mit Tools und Bildern für eine ausgewählte Provider-übergreifende Gruppe aus:

```bash
OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.6-luna,anthropic/claude-opus-4-6,google/gemini-3.1-pro-preview,google/gemini-3.5-flash,google-antigravity/claude-opus-4-6-thinking,deepseek/deepseek-v4-flash,zai/glm-5.1,minimax/MiniMax-M3" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts
```

Optionale zusätzliche Abdeckung außerhalb der kuratierten Listen (wünschenswert; wählen Sie ein „Tools“-fähiges Modell, das Sie aktiviert haben):

- Mistral: `mistral/...`
- Cerebras: `cerebras/...` (wenn Sie Zugriff haben)
- LM Studio: `lmstudio/...` (lokal; Tool-Aufrufe hängen vom API-Modus ab)

### Aggregatoren / alternative Gateways

Wenn entsprechende Schlüssel aktiviert sind, können Sie auch Folgendes testen:

- OpenRouter: `openrouter/...` (Hunderte Modelle; verwenden Sie `openclaw models scan`, um Tool- und bildfähige Kandidaten zu finden)
- OpenCode: `opencode/...` für Zen und `opencode-go/...` für Go (Authentifizierung über `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

Weitere Provider, die Sie in die Live-Matrix aufnehmen können (wenn Anmeldedaten/Konfiguration vorhanden sind):

- Integriert: `anthropic`, `cerebras`, `github-copilot`, `google`, `google-antigravity`, `google-gemini-cli`, `google-vertex`, `groq`, `mistral`, `openai`, `openrouter`, `opencode`, `opencode-go`, `xai`, `zai`
- Über `models.providers` (benutzerdefinierte Endpunkte): `minimax` (Cloud/API) sowie jeder OpenAI-/Anthropic-kompatible Proxy (LM Studio, vLLM, LiteLLM usw.)

<Tip>
Kodieren Sie in der Dokumentation nicht „alle Modelle“ fest. Maßgeblich ist die Liste, die `discoverModels(...)` auf Ihrem Rechner zurückgibt, zusammen mit den verfügbaren Schlüsseln.
</Tip>

## Anmeldedaten (niemals committen)

Live-Tests ermitteln Anmeldedaten auf dieselbe Weise wie die CLI. Praktische Auswirkungen:

- Wenn die CLI funktioniert, sollten Live-Tests dieselben Schlüssel finden.
- Wenn ein Live-Test „keine Anmeldedaten“ meldet, führen Sie die Fehlersuche auf dieselbe Weise wie bei `openclaw models list` bzw. der Modellauswahl durch.

- Agentenspezifische Authentifizierungsprofile: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (dies ist mit „Profilschlüsseln“ in den Live-Tests gemeint)
- Konfiguration: `~/.openclaw/openclaw.json` (oder `OPENCLAW_CONFIG_PATH`)
- Veraltetes OAuth-Verzeichnis: `~/.openclaw/credentials/` (wird, sofern vorhanden, in das bereitgestellte Live-Home kopiert, ist jedoch nicht der primäre Speicher für Profilschlüssel)
- Lokale Live-Durchläufe kopieren die aktive Konfiguration (wobei die Überschreibungen `agents.*.workspace` / `agentDir` entfernt werden) sowie die Datei `auth-profiles.json` jedes Agenten – nicht den Rest des Verzeichnisses dieses Agenten, sodass Daten aus `workspace/` und `sandboxes/` niemals in das bereitgestellte Home gelangen – sowie das veraltete Verzeichnis `credentials/` und unterstützte Authentifizierungsdateien/-verzeichnisse externer CLIs (`.claude.json`, `.claude/.credentials.json`, `.claude/settings*.json`, `.claude/backups`, `.codex/auth.json`, `.codex/config.toml`, `.gemini`, `.minimax`) in ein temporäres Test-Home.

Wenn Sie Umgebungsschlüssel verwenden möchten, exportieren Sie sie vor lokalen Tests oder verwenden Sie die
nachfolgenden Docker-Runner mit einem expliziten `OPENCLAW_PROFILE_FILE`.

## Deepgram Live (Audiotranskription)

- Test: `extensions/deepgram/audio.live.test.ts`
- Aktivieren: `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live extensions/deepgram/audio.live.test.ts`

## BytePlus-Coding-Plan Live

- Test: `extensions/byteplus/live.test.ts`
- Aktivieren: `BYTEPLUS_API_KEY=... BYTEPLUS_LIVE_TEST=1 pnpm test:live extensions/byteplus/live.test.ts`
- Optionale Modellüberschreibung: `BYTEPLUS_CODING_MODEL=ark-code-latest`

## ComfyUI-Workflow-Medien Live

- Test: `extensions/comfy/comfy.live.test.ts`
- Aktivieren: `OPENCLAW_LIVE_TEST=1 COMFY_LIVE_TEST=1 pnpm test:live -- extensions/comfy/comfy.live.test.ts`
- Umfang:
  - Führt die gebündelten comfy-Pfade für Bilder, Videos und `music_generate` aus
  - Überspringt jede Funktion, sofern `plugins.entries.comfy.config.<capability>` nicht konfiguriert ist
  - Nützlich nach Änderungen an der Übermittlung von comfy-Workflows, Abfragen, Downloads oder Plugin-Registrierung

## Live-Bilderzeugung

- Test: `test/image-generation.runtime.live.test.ts`
- Befehl: `pnpm test:live test/image-generation.runtime.live.test.ts`
- Testumgebung: `pnpm test:live:media image`
- Umfang:
  - Listet jedes registrierte Provider-Plugin für die Bildgenerierung auf
  - Verwendet vor der Prüfung bereits exportierte Provider-Umgebungsvariablen
  - Verwendet standardmäßig Live-/Umgebungs-API-Schlüssel vor gespeicherten Authentifizierungsprofilen, damit veraltete Testschlüssel in `auth-profiles.json` echte Shell-Anmeldedaten nicht verdecken
  - Überspringt Provider ohne verwendbare Authentifizierung, verwendbares Profil oder Modell
  - Führt jeden konfigurierten Provider über die gemeinsame Laufzeit für die Bildgenerierung aus:
    - `<provider>:generate`
    - `<provider>:edit`, wenn der Provider Bearbeitungsunterstützung deklariert
- Derzeit abgedeckte gebündelte Provider:
  - `deepinfra`
  - `fal`
  - `google`
  - `minimax`
  - `openai`
  - `openrouter`
  - `vydra`
  - `xai`
- Optionale Eingrenzung:
  - `OPENCLAW_LIVE_IMAGE_GENERATION_PROVIDERS="openai,google,openrouter,xai"`
  - `OPENCLAW_LIVE_IMAGE_GENERATION_PROVIDERS="deepinfra"`
  - `OPENCLAW_LIVE_IMAGE_GENERATION_MODELS="openai/gpt-image-2,google/gemini-3.1-flash-image,openrouter/google/gemini-3.1-flash-image-preview,xai/grok-imagine-image"`
  - `OPENCLAW_LIVE_IMAGE_GENERATION_CASES="google:flash-generate,google:pro-edit,openrouter:generate,xai:default-generate,xai:default-edit"`
- Optionales Authentifizierungsverhalten:
  - `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1`, um die Authentifizierung über den Profilspeicher zu erzwingen und ausschließlich über Umgebungsvariablen gesetzte Überschreibungen zu ignorieren

Fügen Sie für den ausgelieferten CLI-Pfad einen `infer`-Smoke-Test hinzu, nachdem der Live-Test für Provider und Laufzeit
erfolgreich war:

```bash
OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_INFER_CLI_TEST=1 pnpm test:live -- test/image-generation.infer-cli.live.test.ts
openclaw infer image providers --json
openclaw infer image generate \
  --model google/gemini-3.1-flash-image \
  --prompt "Minimales flaches Testbild: ein blaues Quadrat auf weißem Hintergrund, kein Text." \
  --output ./openclaw-infer-image-smoke.png \
  --json
```

Dies deckt die Verarbeitung der CLI-Argumente, die Auflösung der Konfiguration und des Standard-Agenten, die Aktivierung gebündelter
Plugins, die gemeinsame Laufzeit für die Bildgenerierung und die Live-Anfrage an den Provider
ab. Plugin-Abhängigkeiten müssen vor dem Laden der Laufzeit vorhanden sein.

## Live-Musikgenerierung

- Test: `extensions/music-generation-providers.live.test.ts`
- Aktivierung: `OPENCLAW_LIVE_TEST=1 pnpm test:live -- extensions/music-generation-providers.live.test.ts`
- Testumgebung: `pnpm test:live:media music`
- Umfang:
  - Testet den gemeinsamen Pfad für gebündelte Provider zur Musikgenerierung
  - Deckt derzeit `fal`, `google`, `minimax` und `openrouter` ab
  - Verwendet vor der Prüfung bereits exportierte Provider-Umgebungsvariablen
  - Verwendet standardmäßig Live-/Umgebungs-API-Schlüssel vor gespeicherten Authentifizierungsprofilen, damit veraltete Testschlüssel in `auth-profiles.json` echte Shell-Anmeldedaten nicht verdecken
  - Überspringt Provider ohne verwendbare Authentifizierung, verwendbares Profil oder Modell
  - Führt beide deklarierten Laufzeitmodi aus, sofern verfügbar:
    - `generate` mit ausschließlicher Prompt-Eingabe
    - `edit`, wenn der Provider `capabilities.edit.enabled` deklariert
  - `comfy` verfügt über eine eigene separate Live-Datei und ist nicht Teil dieses gemeinsamen Durchlaufs
- Optionale Eingrenzung:
  - `OPENCLAW_LIVE_MUSIC_GENERATION_PROVIDERS="google,minimax"`
  - `OPENCLAW_LIVE_MUSIC_GENERATION_MODELS="google/lyria-3-clip-preview,minimax/music-2.6"`
- Optionales Authentifizierungsverhalten:
  - `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1`, um die Authentifizierung über den Profilspeicher zu erzwingen und ausschließlich über Umgebungsvariablen gesetzte Überschreibungen zu ignorieren

## Live-Videogenerierung

- Test: `extensions/video-generation-providers.live.test.ts`
- Aktivierung: `OPENCLAW_LIVE_TEST=1 pnpm test:live -- extensions/video-generation-providers.live.test.ts`
- Testumgebung: `pnpm test:live:media video`
- Umfang:
  - Testet den gemeinsamen Pfad für gebündelte Provider zur Videogenerierung über `alibaba`, `byteplus`, `deepinfra`, `fal`, `google`, `minimax`, `openai`, `openrouter`, `pixverse`, `qwen`, `runway`, `together`, `vydra`, `xai` hinweg
  - Verwendet standardmäßig den releasesicheren Smoke-Test-Pfad: eine Text-zu-Video-Anfrage pro Provider, einen einsekündigen Hummer-Prompt und eine Obergrenze für Vorgänge pro Provider aus `OPENCLAW_LIVE_VIDEO_GENERATION_TIMEOUT_MS` (standardmäßig `180000`)
  - Überspringt FAL standardmäßig, da die providerseitige Warteschlangenlatenz die Release-Dauer dominieren kann; übergeben Sie `OPENCLAW_LIVE_VIDEO_GENERATION_PROVIDERS="fal"` (oder leeren Sie die Ausschlussliste), um FAL ausdrücklich auszuführen
  - Verwendet vor der Prüfung bereits exportierte Provider-Umgebungsvariablen
  - Verwendet standardmäßig Live-/Umgebungs-API-Schlüssel vor gespeicherten Authentifizierungsprofilen, damit veraltete Testschlüssel in `auth-profiles.json` echte Shell-Anmeldedaten nicht verdecken
  - Überspringt Provider ohne verwendbare Authentifizierung, verwendbares Profil oder Modell
  - Führt standardmäßig nur `generate` aus
  - Setzen Sie `OPENCLAW_LIVE_VIDEO_GENERATION_FULL_MODES=1`, um zusätzlich die deklarierten Transformationsmodi auszuführen, sofern verfügbar:
    - `imageToVideo`, wenn der Provider `capabilities.imageToVideo.enabled` deklariert und der ausgewählte Provider beziehungsweise das ausgewählte Modell im gemeinsamen Durchlauf eine puffergestützte lokale Bildeingabe akzeptiert
    - `videoToVideo`, wenn der Provider `capabilities.videoToVideo.enabled` deklariert und der ausgewählte Provider beziehungsweise das ausgewählte Modell im gemeinsamen Durchlauf eine puffergestützte lokale Videoeingabe akzeptiert
  - Derzeit im gemeinsamen Durchlauf deklarierter, aber übersprungener `imageToVideo`-Provider:
    - `vydra` (puffergestützte lokale Bildeingabe wird in diesem Testpfad nicht unterstützt)
  - Providerspezifische Vydra-Abdeckung:
    - `OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_VYDRA_VIDEO=1 pnpm test:live -- extensions/vydra/vydra.live.test.ts`
    - Diese Datei führt `veo3` für Text-zu-Video sowie einen `kling`-Bild-zu-Video-Testpfad aus, der standardmäßig eine entfernte Bild-URL als Fixture verwendet (zum Überschreiben `OPENCLAW_LIVE_VYDRA_KLING_IMAGE_URL`).
  - Providerspezifische xAI-Abdeckung:
    - `OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_XAI_VIDEO=1 pnpm test:live -- extensions/xai/xai.live.test.ts -t "classic Grok Imagine"`
    - Der klassische Fall generiert zunächst ein quadratisches lokales PNG als erstes Einzelbild, lässt die Geometrie weg, fordert einen einsekündigen Bild-zu-Video-Clip an, fragt den Status bis zum Abschluss ab und überprüft den heruntergeladenen Puffer.
    - `OPENCLAW_LIVE_TEST=1 OPENCLAW_LIVE_XAI_VIDEO=1 pnpm test:live -- extensions/xai/xai.live.test.ts -t "Grok Imagine Video 1.5"`
    - Der 1.5-Fall generiert ein lokales PNG als erstes Einzelbild, fordert einen einsekündigen 1080P-Bild-zu-Video-Clip an, fragt den Status bis zum Abschluss ab und überprüft den heruntergeladenen Puffer.
  - Aktuelle `videoToVideo`-Live-Abdeckung:
    - `runway` nur, wenn das ausgewählte Modell zu `gen4_aleph` aufgelöst wird
  - Derzeit im gemeinsamen Durchlauf deklarierte, aber übersprungene `videoToVideo`-Provider:
    - `alibaba`, `google`, `openai`, `qwen`, `xai`, da diese Pfade derzeit entfernte `http(s)`-Referenz-URLs statt puffergestützter lokaler Eingaben erfordern
- Optionale Eingrenzung:
  - `OPENCLAW_LIVE_VIDEO_GENERATION_PROVIDERS="deepinfra,google,openai,runway"`
  - `OPENCLAW_LIVE_VIDEO_GENERATION_MODELS="google/veo-3.1-fast-generate-preview,openai/sora-2,runway/gen4_aleph"`
  - `OPENCLAW_LIVE_VIDEO_GENERATION_SKIP_PROVIDERS=""`, um alle Provider in den Standarddurchlauf einzubeziehen, einschließlich FAL
  - `OPENCLAW_LIVE_VIDEO_GENERATION_TIMEOUT_MS=60000`, um die Vorgangsobergrenze jedes Providers für einen aggressiven Smoke-Test zu reduzieren
- Optionales Authentifizierungsverhalten:
  - `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1`, um die Authentifizierung über den Profilspeicher zu erzwingen und ausschließlich über Umgebungsvariablen gesetzte Überschreibungen zu ignorieren

## Live-Testumgebung für Medien

- Befehl: `pnpm test:live:media`
- Einstiegspunkt: `test/e2e/qa-lab/media/hosted-media-provider-live.ts`, der `pnpm test:live -- <suite-test-file>` für jede ausgewählte Suite ausführt, sodass das Heartbeat- und Ruhemodusverhalten mit anderen `pnpm test:live`-Ausführungen konsistent bleibt.
- Zweck:
  - Führt die gemeinsamen Live-Suites für Bild, Musik und Video über einen einzigen repo-eigenen Einstiegspunkt aus
  - Lädt fehlende Provider-Umgebungsvariablen automatisch aus `~/.profile`
  - Grenzt jede Suite standardmäßig automatisch auf Provider ein, für die derzeit eine verwendbare Authentifizierung vorliegt
- Flags:
  - `--providers <csv>` als globaler Provider-Filter; `--image-providers` / `--music-providers` / `--video-providers` begrenzen einen Filter auf eine Suite
  - `--all-providers` überspringt den authentifizierungsbasierten automatischen Filter
  - `--allow-empty` beendet den Vorgang mit `0`, wenn nach der Filterung keine ausführbaren Provider verbleiben
  - `--quiet` / `--no-quiet` werden an `test:live` weitergegeben
- Beispiele:
  - `pnpm test:live:media`
  - `pnpm test:live:media image video --providers openai,google,minimax`
  - `pnpm test:live:media video --video-providers openai,runway --all-providers`
  - `pnpm test:live:media music --quiet`

## Verwandte Themen

- [Tests](/de/help/testing) – Unit-, Integrations-, QA- und Docker-Suites
