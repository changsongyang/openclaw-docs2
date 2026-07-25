---
read_when:
    - Ändern der Audiotranskription oder Medienverarbeitung
summary: Wie eingehende Audio- und Sprachnachrichten heruntergeladen, transkribiert und in Antworten eingefügt werden
title: Audio- und Sprachnachrichten
x-i18n:
    generated_at: "2026-07-24T20:27:53Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 4076e3e55eb5c6dcc94cfdd842619697c8d756b924956d7b266d18446b4dd9be
    source_path: nodes/audio.md
    workflow: 16
---

## Funktionsweise

Wenn die Audioerkennung aktiviert ist (oder automatisch erkannt wird), führt OpenClaw Folgendes aus:

1. Sucht den ersten Audioanhang (lokaler Pfad oder URL) und lädt ihn bei Bedarf herunter.
2. Erzwingt `maxBytes`, bevor der Anhang an jeden Modelleintrag gesendet wird.
3. Führt den ersten geeigneten Modelleintrag der Reihe nach aus (Provider oder CLI); wenn ein Eintrag fehlschlägt oder übersprungen wird (Größe/Timeout), wird der nächste Eintrag versucht.
4. Ersetzt bei Erfolg `Body` durch einen `[Audio]`-Block und setzt `{{Transcript}}`.

Wenn die Transkription erfolgreich ist, werden `CommandBody`/`RawBody` ebenfalls auf das Transkript gesetzt, damit Slash-Befehle weiterhin funktionieren. Mit `--verbose` zeigen die Protokolle, wann die Transkription ausgeführt wird und wann sie den Nachrichtentext ersetzt.

## Automatische Erkennung (Standard)

Wenn Sie keine Modelle konfiguriert haben und `tools.media.audio.enabled` nicht `false` ist, führt OpenClaw die automatische Erkennung in dieser Reihenfolge durch und stoppt bei der ersten funktionierenden Option:

1. **Aktives Antwortmodell**, wenn dessen Provider Audioerkennung unterstützt.
2. **Konfigurierte Provider-Authentifizierung** – jeder `models.providers.*`-Eintrag mit verfügbarer Authentifizierung für einen Provider, der Audiotranskription unterstützt. Dies wird vor lokalen CLIs geprüft, sodass ein konfigurierter API-Schlüssel stets Vorrang vor einer lokalen Binärdatei auf `PATH` hat.
   Provider-Priorität bei mehreren konfigurierten Providern: Groq, OpenAI, xAI, Deepgram, Google, SenseAudio, ElevenLabs, Mistral.
3. **Lokale CLIs** (nur wenn keine Provider-Authentifizierung ermittelt wurde). OpenClaw erstellt eine geordnete Fallback-Liste:
   - `whisper-cli`, vor den CPU-Standardeinstellungen nur dann, wenn ein früherer Modellaufruf im aktuellen Prozess Metal oder CUDA erkannt hat
   - `sherpa-onnx-offline` mit seinem standardmäßigen CPU-Provider (erfordert `SHERPA_ONNX_MODEL_DIR` mit `tokens.txt`, `encoder.onnx`, `decoder.onnx` und `joiner.onnx`)
   - `whisper-cli`, wenn Metal/CUDA lediglich beim Build unterstützt wird oder das ausgewählte Backend anderweitig nicht erkannt wurde
   - `parakeet-mlx` auf Apple Silicon (MLX-fähig; Gerätenutzung bleibt unerkannt)
   - `whisper` (Python-CLI; lädt Modelle automatisch herunter)

Die Herkunft der Installation bzw. Verknüpfung ist ein Nachweis der Fähigkeit, kein Nachweis der Ausführung. Dadurch wird ein Kandidat niemals von selbst vor CPU-sherpa eingeordnet. OpenClaw lädt während der Einrichtung oder bei Statusprüfungen kein Modell, nur um ein Backend zu testen.
Das automatisch erkannte whisper.cpp behält seine normalen Protokolle für Modellausführungen bei, damit OpenClaw die vorgelagerte `using … backend`-Zeile erfassen kann. Explizite CLI-Einträge behalten ihre konfigurierten Ausgabeoptionen bei.

Die automatische Erkennung der Gemini CLI für Medienerkennung wurde für Bilder/Videos durch einen Sandbox-Fallback der Antigravity CLI (`agy`) ersetzt; für Audio wird über die oben genannten lokalen Binärdateien hinaus kein CLI-Fallback verwendet.

Um die automatische Erkennung zu deaktivieren, setzen Sie `tools.media.audio.enabled: false`. Fügen Sie zur Anpassung Einträge mit Fähigkeits-Tags zu `tools.media.models` hinzu.

<Note>
Die Binärdateierkennung erfolgt unter macOS/Linux/Windows nach bestem Bemühen. Stellen Sie sicher, dass sich die CLI in `PATH` befindet (`~` wird aufgelöst), oder legen Sie ein explizites CLI-Modell mit einem vollständigen Befehlspfad fest.
</Note>

Prüfen Sie die lokale Auswahl, ohne Audio zu transkribieren:

```bash
openclaw capability audio providers
openclaw doctor --lint --only core/doctor/local-audio-acceleration --severity-min info
```

Das Provider-Inventar meldet den Gewinner des lokalen Fallbacks getrennt von der globalen Provider-Auswahl sowie Felder für fähige, angeforderte und beobachtete Backends. Nachdem die Transkription ausgeführt wurde, meldet `/status` das angeforderte oder bei der Ausführung beobachtete Backend in der Medienzeile. Explizite audiofähige `tools.media.models`-CLI-Einträge umgehen weiterhin die automatische Auswahl; verwenden Sie deren Backend-spezifische Optionen wie sherpa `--provider=cuda` oder whisper.cpp `--no-gpu`/`--device`.

## Konfigurationsbeispiele

### Provider + CLI-Fallback (OpenAI + Whisper CLI)

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-4o-transcribe", capabilities: ["audio"] },
        {
          type: "cli",
          command: "whisper",
          args: ["--model", "base", "{{AttachmentPath}}"],
          timeoutSeconds: 45,
          capabilities: ["audio"],
        },
      ],
      audio: { enabled: true, preferredModel: "openai/gpt-4o-transcribe" },
    },
  },
}
```

### Nur Provider (Deepgram)

```json5
{
  tools: {
    media: {
      models: [{ provider: "deepgram", model: "nova-3", capabilities: ["audio"] }],
      audio: { enabled: true },
    },
  },
}
```

### Nur Provider (Mistral Voxtral)

```json5
{
  tools: {
    media: {
      models: [{ provider: "mistral", model: "voxtral-mini-latest", capabilities: ["audio"] }],
      audio: { enabled: true },
    },
  },
}
```

### Nur Provider (SenseAudio)

```json5
{
  tools: {
    media: {
      models: [
        {
          provider: "senseaudio",
          model: "senseaudio-asr-pro-1.5-260319",
          capabilities: ["audio"],
        },
      ],
      audio: { enabled: true },
    },
  },
}
```

### Transkript im Chat wiedergeben (Opt-in)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        echoTranscript: true,
        echoFormat: '📝 "{transcript}"',
      },
    },
  },
}
```

## Hinweise und Einschränkungen

- Die Provider-Authentifizierung folgt der Standardreihenfolge für die Modellauthentifizierung (Authentifizierungsprofile, Umgebungsvariablen, `models.providers.*.apiKey`).
- Einrichtungsdetails für Groq: [Groq](/de/providers/groq).
- Deepgram übernimmt `DEEPGRAM_API_KEY`, wenn `provider: "deepgram"` verwendet wird. Einrichtungsdetails: [Deepgram](/de/providers/deepgram).
- Einrichtungsdetails für Mistral: [Mistral](/de/providers/mistral).
- SenseAudio übernimmt `SENSEAUDIO_API_KEY`, wenn `provider: "senseaudio"` verwendet wird. Einrichtungsdetails: [SenseAudio](/de/providers/senseaudio).
- Audio-Provider können Standardeinstellungen unter `tools.media.audio` verwenden oder `baseUrl`, `headers`, `providerOptions` sowie Grenzwerte in ihrem `tools.media.models[]`-Eintrag überschreiben.
- Die integrierte Größenbegrenzung für Audio beträgt 20MB. Eine Überschreibung über `maxBytes` auf Eintragsebene kann sie ändern; zu große Audiodateien werden für dieses Modell übersprungen und der nächste Eintrag wird versucht.
- Audiodateien unter 1024 Byte werden vor der Transkription durch den Provider/die CLI übersprungen.
- Der standardmäßige `maxChars` für Audio ist **nicht gesetzt** (vollständiges Transkript). Setzen Sie `tools.media.audio.maxChars` oder `maxChars` pro Eintrag, um die Ausgabe zu kürzen.
- Der Standard für die automatische OpenAI-Erkennung ist `gpt-4o-transcribe`; setzen Sie `model: "gpt-4o-mini-transcribe"` für eine günstigere/schnellere Option.
- Das Transkript ist für Vorlagen als `{{Transcript}}` verfügbar.
- `tools.media.audio.echoTranscript` ist standardmäßig deaktiviert; `echoFormat` akzeptiert einen `{transcript}`-Platzhalter.
- Die CLI-Standardausgabe ist auf 5MB begrenzt; halten Sie die CLI-Ausgabe knapp.
- CLI-`args` sollte `{{AttachmentPath}}` für den lokalen Audiodateipfad verwenden. Führen Sie `openclaw doctor --fix` aus, um veraltete `{input}`-Platzhalter aus älteren `audio.transcription.command`-Konfigurationen zu migrieren (entfernter Schlüssel: `audio.transcription`, ersetzt durch `tools.media.models`). `{{MediaPath}}` bleibt ein veralteter Kompatibilitätsalias.
- `tools.media.concurrency` begrenzt Medienaufgaben; es handelt sich nicht um einen GPU-Scheduler.

### Dauerhaft laufende lokale Spracherkennung

Automatisch erkannte lokale Spracherkennung bleibt bei einem Prozess pro Anfrage. OpenClaw verwaltet derzeit keinen dauerhaft laufenden whisper.cpp-Server, da das standardmäßige Homebrew-Paket `whisper-cpp` diesen Server deaktiviert, während das vorgelagerte Beispiel keine konfigurierte, begrenzte Zulassungswarteschlange besitzt. Ein Plugin-eigener dauerhafter Lebenszyklus benötigt einen gepflegten, paketierten Worker mit Zustands-/Startprüfung, Modellresidenz, begrenzter Warteschlange, Abbruch/Timeout, ausschließlich an Loopback gebundenem Betrieb ohne Authentifizierung und ohne Cloud-Fallback, bevor er sicher aktiviert werden kann.

### Unterstützung für Proxy-Umgebungsvariablen

Provider-basierte Audiotranskription berücksichtigt standardmäßige Umgebungsvariablen für ausgehende Proxys entsprechend der `EnvHttpProxyAgent`-Semantik von undici:

- `HTTPS_PROXY` / `https_proxy`
- `HTTP_PROXY` / `http_proxy`
- `ALL_PROXY` / `all_proxy`

Variablen in Kleinbuchstaben haben Vorrang vor solchen in Großbuchstaben; `NO_PROXY`/`no_proxy`-Einträge (Hostnamen, `*.suffix` oder `host:port`) umgehen den Proxy. Wenn keine Proxy-Umgebungsvariablen gesetzt sind, wird ein direkter ausgehender Zugriff verwendet. Wenn die Proxy-Einrichtung fehlschlägt (fehlerhafte URL), protokolliert OpenClaw eine Warnung und greift auf direkten Abruf zurück.

## Erwähnungserkennung in Gruppen

Auf Kanälen, die eine Audio-Vorabprüfung unterstützen, transkribiert OpenClaw Audio **vor** der Prüfung auf Erwähnungen, wenn `requireMention: true` für einen Gruppenchat gesetzt ist. Dadurch kann eine Sprachnachricht ohne Beschriftung die Erwähnungsschranke passieren, wenn ihr Transkript ein konfiguriertes Erwähnungsmuster enthält. Kanalspezifische Dokumentation beschreibt Übertragungswege, die stattdessen eine eingegebene Erwähnung erfordern.

**Funktionsweise:**

1. Wenn eine Sprachnachricht keinen Text enthält und die Gruppe Erwähnungen erfordert, führt OpenClaw eine Vorabtranskription des ersten Audioanhangs durch.
2. Das Transkript wird auf Erwähnungsmuster geprüft (zum Beispiel `@BotName`, Emoji-Auslöser).
3. Wenn eine Erwähnung gefunden wird, durchläuft die Nachricht die vollständige Antwort-Pipeline.

**Fallback-Verhalten:** Wenn die Vorabtranskription fehlschlägt (Timeout, API-Fehler usw.), greift die Nachricht auf die reine Texterkennung von Erwähnungen zurück, sodass gemischte Nachrichten (Text + Audio) nie verworfen werden.

**Opt-out pro Telegram-Gruppe/-Thema:**

- Setzen Sie `channels.telegram.groups.<chatId>.disableAudioPreflight: true`, um Vorabprüfungen des Transkripts auf Erwähnungen für diese Gruppe zu überspringen.
- Setzen Sie `channels.telegram.groups.<chatId>.topics.<threadId>.disableAudioPreflight`, um die Einstellung pro Thema zu überschreiben (`true` zum Überspringen, `false` zum Erzwingen der Aktivierung).
- Der Standardwert ist `false` (Vorabprüfung aktiviert, wenn die Bedingungen der Erwähnungsschranke erfüllt sind).

**Beispiel:** Eine Person sendet in einer Telegram-Gruppe mit `requireMention: true` eine Sprachnachricht mit dem Inhalt „Hey @Claude, wie ist das Wetter?“. Die Sprachnachricht wird transkribiert, die Erwähnung wird erkannt und der Agent antwortet.

## Fallstricke

- Bereichsregeln verwenden den ersten Treffer; `chatType` wird zu `direct`, `group` oder `channel` normalisiert.
- Stellen Sie sicher, dass Ihre CLI mit 0 beendet wird und Klartext ausgibt; die JSON-Ausgabe muss über `jq -r .text` aufbereitet werden.
- Bekannte Dateiausgabemodi sind maßgeblich: Eine leere oder fehlende abgeleitete Transkriptdatei erzeugt kein Transkript, statt auf die Fortschrittsausgabe der CLI zurückzugreifen.
- Verwenden Sie für `parakeet-mlx` `--output-format txt` (oder `all`) mit `--output-dir` und der standardmäßigen `{filename}`-Ausgabevorlage. Die vorgelagerten Umgebungsvariablen `PARAKEET_OUTPUT_FORMAT` und `PARAKEET_OUTPUT_TEMPLATE` werden ebenfalls berücksichtigt. OpenClaw liest `<output-dir>/<media-basename>.txt`; das standardmäßige `srt`-Format, andere Formate und benutzerdefinierte Ausgabevorlagen verwenden weiterhin die Standardausgabe.
- Verwenden Sie angemessene Timeouts (`timeoutSeconds`, standardmäßig 60s), um eine Blockierung der Antwortwarteschlange zu vermeiden.
- Die Vorabtranskription verarbeitet zur Erwähnungserkennung nur den **ersten** Audioanhang. Zusätzliche Audioanhänge werden während der Hauptphase der Medienerkennung verarbeitet.

## Verwandte Themen

- [Medienerkennung](/de/nodes/media-understanding)
- [Sprechmodus](/de/nodes/talk)
- [Sprachaktivierung](/de/nodes/voicewake)
