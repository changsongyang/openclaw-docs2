---
read_when:
    - Heartbeat-Intervall oder Nachrichtenübermittlung anpassen
    - Entscheidung zwischen Heartbeat und Cron für geplante Aufgaben
sidebarTitle: Heartbeat
summary: Heartbeat-Abfragenachrichten und Benachrichtigungsregeln
title: Heartbeat
x-i18n:
    generated_at: "2026-07-24T17:18:01Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 44c78e797987d8dccab910cd82fc1f482df86afce40677846d8f26522d33f6fa
    source_path: gateway/heartbeat.md
    workflow: 16
---

<Note>
**Heartbeat oder Cron?** Unter [Automatisierung](/de/automation) finden Sie Hinweise dazu, wann Sie was verwenden sollten.
</Note>

Heartbeat führt **regelmäßige Agent-Durchläufe** in der Hauptsitzung aus, damit das Modell alles ansprechen kann, was Aufmerksamkeit erfordert, ohne Sie mit Nachrichten zu überfluten.

Heartbeat ist ein geplanter Durchlauf in der Hauptsitzung – dabei werden **keine** Datensätze für [Hintergrundaufgaben](/de/automation/tasks) erstellt. Aufgabendatensätze sind für entkoppelte Arbeiten vorgesehen (ACP-Durchläufe, Subagenten, isolierte Cron-Aufträge).

Intern wird der Heartbeat-Takt vom Cron-Scheduler verwaltet: Das Gateway verwaltet für jeden Agenten mit aktiviertem Heartbeat einen systemeigenen Cron-Auftrag (in `openclaw cron list --all` als `Heartbeat (agent-id)` sichtbar). Die Heartbeat-Konfiguration bleibt die Eingabe für den gewünschten Zustand, während der persistierte Überwachungszeitplan den tatsächlichen Ausführungszeitpunkt und die anschließende Abkühlzeit des Runners steuert. Das Gateway überträgt Konfigurationsänderungen beim Start und beim Neuladen der Konfiguration; `openclaw doctor --fix` kann fehlende oder veraltete Überwachungszeilen vor dem nächsten Gateway-Start materialisieren. Bearbeiten Sie `agents.*.heartbeat`, nicht den Cron-Auftrag.

Geplante Heartbeats erfordern Cron. Wenn `cron.enabled` auf `false` oder `OPENCLAW_SKIP_CRON=1` gesetzt ist, protokolliert das Gateway beim Start eine Warnung und führt keine geplanten Heartbeats aus; manuelle und ereignisgesteuerte Heartbeat-Aktivierungen bleiben verfügbar. Es gibt keinen separaten Heartbeat-Ersatztimer.

Fehlerbehebung: [Geplante Aufgaben](/de/automation/cron-jobs#troubleshooting)

## Schnellstart (Einsteiger)

<Steps>
  <Step title="Takt auswählen">
    Lassen Sie Heartbeats aktiviert (Standard ist `30m` beziehungsweise `1h`, wenn Anthropic-OAuth-/Token-Authentifizierung konfiguriert ist, einschließlich der Wiederverwendung der Claude CLI), oder legen Sie einen eigenen Takt fest.
  </Step>
  <Step title="Überwachungsnotizen hinzufügen (optional)">
    Speichern Sie mit `openclaw cron scratch <jobId> --set "..."` eine kurze Checkliste in den Notizen der Heartbeat-Überwachung.
  </Step>
  <Step title="Ziel für Heartbeat-Nachrichten festlegen">
    `target: "none"` ist die Standardeinstellung; legen Sie `target: "last"` fest, um Nachrichten an den letzten Kontakt weiterzuleiten.
  </Step>
  <Step title="Optionale Feinabstimmung">
    - Verwenden Sie einen schlanken Bootstrap-Kontext, wenn Heartbeat-Durchläufe nur die Überwachungsnotizen benötigen.
    - Aktivieren Sie isolierte Sitzungen, damit nicht bei jedem Heartbeat der vollständige Gesprächsverlauf gesendet wird.
    - Beschränken Sie Heartbeats auf aktive Zeiten (Ortszeit).

  </Step>
</Steps>

Beispielkonfiguration:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // explizite Zustellung an den letzten Kontakt (Standard ist "none")
        directPolicy: "allow", // Standard: direkte/DM-Ziele zulassen; zum Unterdrücken auf "block" setzen
        lightContext: true, // optional: Workspace-Bootstrap-Dateien für Heartbeat-Durchläufe überspringen
        isolatedSession: true, // optional: neuer Sitzungskontext für jeden Durchlauf (kein Gesprächsverlauf)
        // activeHours: { start: "08:00", end: "24:00" },
      },
    },
  },
}
```

## Standardeinstellungen

- Intervall: `30m`. Durch Anwenden der Standardeinstellungen des Anthropic-Providers wird dieser Wert auf `1h` erhöht, wenn der ermittelte Authentifizierungsmodus OAuth/Token ist (einschließlich der Wiederverwendung der Claude CLI), jedoch nur, solange `heartbeat.every` nicht festgelegt ist. Legen Sie `agents.defaults.heartbeat.every` oder agentenspezifisch `agents.entries.*.heartbeat.every` fest; verwenden Sie zum Deaktivieren `0m`.
- Prompt-Text (über `agents.defaults.heartbeat.prompt` konfigurierbar): `Follow the heartbeat monitor scratch context when provided. Recurring tasks are cron jobs; create or change their schedules with cron tools or the openclaw cron CLI, not heartbeat scratch. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
- Zeitüberschreitung: Heartbeat-Durchläufe ohne festgelegten Wert verwenden `agents.defaults.timeoutSeconds`, sofern dieser Wert gesetzt ist. Andernfalls verwenden sie den auf 600 Sekunden begrenzten Heartbeat-Takt. Legen Sie für längere Heartbeat-Arbeiten `agents.defaults.heartbeat.timeoutSeconds` oder agentenspezifisch `agents.entries.*.heartbeat.timeoutSeconds` fest.
- Der Heartbeat-Prompt wird **unverändert** als Benutzernachricht gesendet. Der System-Prompt enthält einen Abschnitt „Heartbeats“, wenn Heartbeats für den Standardagenten aktiviert sind, und der Durchlauf wird intern gekennzeichnet.
- Wenn Heartbeats mit `0m` deaktiviert werden, bleibt der Überwachungs-Cron-Auftrag bestehen, wird jedoch deaktiviert. Seine Notizen bleiben erhalten, bis Sie den Takt erneut aktivieren.
- Wenn Cron selbst deaktiviert ist, werden geplante Heartbeats nicht ausgeführt, auch wenn der Heartbeat-Takt aktiviert bleibt.
- Aktive Zeiten (`heartbeat.activeHours`) werden in der konfigurierten Zeitzone geprüft. Außerhalb des Zeitfensters werden Heartbeats bis zum nächsten Ausführungszeitpunkt innerhalb des Fensters übersprungen.
- Heartbeats werden automatisch zurückgestellt, während Cron-Arbeiten aktiv sind oder sich in der Warteschlange befinden oder während sitzungsschlüsselgebundene Subagenten- oder verschachtelte Befehlsausführungsspuren dieses Agenten beschäftigt sind. Andere Agenten pausieren sich nicht gegenseitig.

## Zweck des Heartbeat-Prompts

Der Standard-Prompt ist bewusst allgemein gehalten:

- **Hintergrundaufgaben**: „Ausstehende Aufgaben berücksichtigen“ regt den Agenten dazu an, Folgeaufgaben (Posteingang, Kalender, Erinnerungen, Arbeiten in der Warteschlange) zu prüfen und auf dringende Punkte hinzuweisen.
- **Nachfrage beim Menschen**: „Erkundigen Sie sich tagsüber gelegentlich nach Ihrem Menschen“ regt zu einer gelegentlichen kurzen Nachricht wie „Benötigen Sie etwas?“ an, vermeidet aber mithilfe Ihrer konfigurierten lokalen Zeitzone nächtliche Nachrichten (siehe [Zeitzone](/de/concepts/timezone)).

Heartbeat kann auf abgeschlossene [Hintergrundaufgaben](/de/automation/tasks) reagieren, ein Heartbeat-Durchlauf selbst erstellt jedoch keinen Aufgabendatensatz.

Wenn ein Heartbeat eine ganz bestimmte Aufgabe ausführen soll (z. B. „Gmail-PubSub-Statistiken prüfen“ oder „Zustand des Gateways überprüfen“), legen Sie `agents.defaults.heartbeat.prompt` (oder `agents.entries.*.heartbeat.prompt`) auf einen benutzerdefinierten Text fest, der unverändert gesendet wird.

## Antwortvertrag

- Wenn nichts Aufmerksamkeit erfordert, antworten Sie mit **`HEARTBEAT_OK`**.
- Heartbeat-Durchläufe können stattdessen `heartbeat_respond` mit `notify: false` aufrufen, wenn keine sichtbare Aktualisierung erforderlich ist, oder `notify: true` zusammen mit `notificationText` für eine Warnmeldung. Falls vorhanden, hat die strukturierte Werkzeugantwort Vorrang vor dem textbasierten Rückgriff.
- Ein aussagekräftiges `heartbeat_respond`-Ergebnis mit `notify: false` bleibt unsichtbar, wird jedoch als begrenzter interner Kontext für den nächsten Benutzerdurchlauf in dieser Sitzung gespeichert. `no_change`-Bestätigungen und sichtbare Benachrichtigungen werden nicht auf diese Weise gespeichert.
- Während Heartbeat-Durchläufen behandelt OpenClaw `HEARTBEAT_OK` als Bestätigung, wenn es am **Anfang oder Ende** der Antwort erscheint. Das Token wird entfernt, und die Antwort wird verworfen, wenn der verbleibende Inhalt höchstens 300 Zeichen umfasst.
- Wenn `HEARTBEAT_OK` in der **Mitte** einer Antwort erscheint, wird es nicht besonders behandelt.
- Fügen Sie bei Warnmeldungen `HEARTBEAT_OK` **nicht** ein; geben Sie ausschließlich den Warntext zurück.

Außerhalb von Heartbeats wird ein unbeabsichtigtes `HEARTBEAT_OK` am Anfang oder Ende einer Nachricht entfernt und protokolliert; eine Nachricht, die ausschließlich aus `HEARTBEAT_OK` besteht, wird verworfen.

## Konfiguration

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // Standard: 30m (0m deaktiviert)
        model: "anthropic/claude-opus-4-6",
        lightContext: false, // Standard: false; true überspringt Workspace-Bootstrap-Dateien für Heartbeat-Durchläufe
        isolatedSession: false, // Standard: false; true führt jeden Heartbeat in einer neuen Sitzung aus (kein Gesprächsverlauf)
        target: "last", // Standard: none | Optionen: last | none | <channel id> (Kern oder Plugin, z. B. "imessage")
        to: "+15551234567", // optionale kanalspezifische Überschreibung
        accountId: "ops-bot", // optionale Kanal-ID bei mehreren Konten
        prompt: "Befolgen Sie den Notizkontext der Heartbeat-Überwachung, wenn er bereitgestellt wird. Wiederkehrende Aufgaben sind Cron-Aufträge; erstellen oder ändern Sie ihre Zeitpläne mit Cron-Werkzeugen oder der OpenClaw-Cron-CLI, nicht mit Heartbeat-Notizen. Leiten Sie keine alten Aufgaben aus früheren Chats ab und wiederholen Sie sie nicht. Wenn nichts Aufmerksamkeit erfordert, antworten Sie mit HEARTBEAT_OK.",
      },
    },
  },
}
```

### Geltungsbereich und Rangfolge

- `agents.defaults.heartbeat` legt das globale Heartbeat-Verhalten fest.
- `agents.entries.*.heartbeat` wird darübergelegt; wenn ein Agent einen `heartbeat`-Block besitzt, führen **nur diese Agenten** Heartbeats aus.
- `channels.defaults.heartbeatVisibility` legt die standardmäßige Sichtbarkeit für alle Kanäle fest.
- `channels.<channel>.heartbeatVisibility` überschreibt die Kanaleinstellungen.
- `channels.<channel>.accounts.<id>.heartbeatVisibility` (Kanäle mit mehreren Konten) überschreibt die kanalspezifischen Einstellungen.

### Agentenspezifische Heartbeats

Wenn ein `agents.entries.*`-Eintrag einen `heartbeat`-Block enthält, führen **nur diese Agenten** Heartbeats aus. Der agentenspezifische Block wird über `agents.defaults.heartbeat` gelegt, sodass Sie gemeinsame Standardeinstellungen einmalig festlegen und für einzelne Agenten überschreiben können.

Beispiel: zwei Agenten, von denen nur der zweite Heartbeats ausführt.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // explizite Zustellung an den letzten Kontakt (Standard ist "none")
      },
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          timeoutSeconds: 45,
          prompt: "Befolgen Sie den Notizkontext der Heartbeat-Überwachung, wenn er bereitgestellt wird. Wiederkehrende Aufgaben sind Cron-Aufträge; erstellen oder ändern Sie ihre Zeitpläne mit Cron-Werkzeugen oder der OpenClaw-Cron-CLI, nicht mit Heartbeat-Notizen. Leiten Sie keine alten Aufgaben aus früheren Chats ab und wiederholen Sie sie nicht. Wenn nichts Aufmerksamkeit erfordert, antworten Sie mit HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

### Beispiel für aktive Zeiten

Beschränken Sie Heartbeats auf Geschäftszeiten in einer bestimmten Zeitzone:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // explizite Zustellung an den letzten Kontakt (Standard ist "none")
        activeHours: {
          start: "09:00",
          end: "22:00",
          timezone: "America/New_York", // optional; verwendet userTimezone, falls festgelegt, andernfalls die Zeitzone des Hosts
        },
      },
    },
  },
}
```

Außerhalb dieses Zeitfensters (vor 9 Uhr oder nach 22 Uhr Eastern Time) werden Heartbeats übersprungen. Der nächste geplante Ausführungszeitpunkt innerhalb des Fensters wird normal ausgeführt.

### Einrichtung von 24/7

Wenn Heartbeats den ganzen Tag ausgeführt werden sollen, verwenden Sie eines der folgenden Muster:

- Lassen Sie `activeHours` vollständig weg (keine Zeitfensterbeschränkung; dies ist das Standardverhalten).
- Legen Sie ein ganztägiges Zeitfenster fest: `activeHours: { start: "00:00", end: "24:00" }`.

<Warning>
Legen Sie für `start` und `end` nicht dieselbe Uhrzeit fest (beispielsweise `08:00` bis `08:00`). Dies wird als Zeitfenster ohne Dauer behandelt, sodass Heartbeats immer übersprungen werden.
</Warning>

### Beispiel für mehrere Konten

Verwenden Sie `accountId`, um ein bestimmtes Konto auf Kanälen mit mehreren Konten wie Telegram anzusprechen:

```json5
{
  agents: {
    list: [
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "telegram",
          to: "12345678:topic:42", // optional: an ein bestimmtes Thema/einen bestimmten Thread weiterleiten
          accountId: "ops-bot",
        },
      },
    ],
  },
  channels: {
    telegram: {
      accounts: {
        "ops-bot": { botToken: "YOUR_TELEGRAM_BOT_TOKEN" },
      },
    },
  },
}
```

### Feldhinweise

<ParamField path="every" type="string">
  Heartbeat-Intervall (Dauerzeichenfolge; Standardeinheit = Minuten).
</ParamField>
<ParamField path="model" type="string">
  Optionale Modellüberschreibung für Heartbeat-Durchläufe (`provider/model`).
</ParamField>
<ParamField path="lightContext" type="boolean" default="false">
  Bei true verwenden Heartbeat-Durchläufe einen schlanken Bootstrap-Kontext und überspringen Workspace-Bootstrap-Dateien. Die Überwachungsnotizen werden in beiden Fällen vom Heartbeat-Runner eingefügt.
</ParamField>
<ParamField path="isolatedSession" type="boolean" default="false">
  Bei true wird jeder Heartbeat in einer neuen Sitzung ohne vorherigen Gesprächsverlauf ausgeführt. Verwendet dasselbe Isolationsmuster wie Cron `sessionTarget: "isolated"`. Reduziert die Token-Kosten pro Heartbeat erheblich. Kombinieren Sie dies mit `lightContext: true`, um maximale Einsparungen zu erzielen. Die Zustellungsweiterleitung verwendet weiterhin den Kontext der Hauptsitzung.
</ParamField>
<ParamField path="session" type="string">
  Optionaler Sitzungsschlüssel für Heartbeat-Durchläufe.

- `main` (Standard): Hauptsitzung des Agenten.
- Expliziter Sitzungsschlüssel (aus `openclaw sessions --json` oder der [Sitzungs-CLI](/de/cli/sessions) kopieren).
- Formate für Sitzungsschlüssel: siehe [Sitzungen](/de/concepts/session) und [Gruppen](/de/channels/groups).

</ParamField>
<ParamField path="target" type="string">
- `last`: an den zuletzt verwendeten externen Kanal zustellen.
- Expliziter Kanal: eine beliebige konfigurierte Kanal- oder Plugin-ID, zum Beispiel `discord`, `matrix`, `telegram` oder `whatsapp`.
- `none` (Standard): den Heartbeat ausführen, aber **nicht extern zustellen**.

</ParamField>
<ParamField path="directPolicy" type='"allow" | "block"' default="allow">
  Steuert das Zustellungsverhalten für direkte Nachrichten/DMs. `allow`: Heartbeat-Zustellung per Direktnachricht/DM zulassen. `block`: Zustellung per Direktnachricht/DM unterdrücken (`reason=dm-blocked`).

</ParamField>
<ParamField path="to" type="string">
  Optionale Empfängerüberschreibung (kanalspezifische ID, z. B. E.164 für WhatsApp oder eine Telegram-Chat-ID). Verwenden Sie für Telegram-Themen/Threads `<chatId>:topic:<messageThreadId>`.

</ParamField>
<ParamField path="accountId" type="string">
  Optionale Konto-ID für Kanäle mit mehreren Konten. Bei `target: "last"` gilt die Konto-ID für den ermittelten letzten Kanal, sofern dieser Konten unterstützt; andernfalls wird sie ignoriert. Wenn die Konto-ID keinem konfigurierten Konto des ermittelten Kanals entspricht, wird die Zustellung übersprungen.

</ParamField>
<ParamField path="prompt" type="string">
  Überschreibt den Standard-Prompt-Inhalt (wird nicht zusammengeführt).

</ParamField>
<ParamField path="timeoutSeconds" type="number" default="global timeout or min(every, 600)">
  Maximale Anzahl von Sekunden, die ein Heartbeat-Agent-Durchlauf dauern darf, bevor er abgebrochen wird. Lassen Sie die Einstellung leer, um `agents.defaults.timeoutSeconds` zu verwenden, sofern festgelegt; andernfalls wird die auf 600 Sekunden begrenzte Heartbeat-Frequenz verwendet.

</ParamField>
<ParamField path="activeHours" type="object">
  Beschränkt Heartbeat-Ausführungen auf ein Zeitfenster. Objekt mit `start` (HH:MM, einschließlich; verwenden Sie `00:00` für den Tagesbeginn), `end` (HH:MM, ausschließlich; `24:00` ist für das Tagesende zulässig) und optional `timezone`.

- Nicht angegeben oder `"user"`: Verwendet Ihre Einstellung `agents.defaults.userTimezone`, sofern festgelegt; andernfalls wird auf die Zeitzone des Hostsystems zurückgegriffen.
- `"local"`: Verwendet immer die Zeitzone des Hostsystems.
- Beliebige IANA-Kennung (z. B. `America/New_York`): wird direkt verwendet; ist sie ungültig, wird auf das oben beschriebene Verhalten von `"user"` zurückgegriffen.
- `start` und `end` dürfen für ein aktives Zeitfenster nicht identisch sein; identische Werte gelten als Fenster ohne Breite (immer außerhalb des Zeitfensters).
- Außerhalb des aktiven Zeitfensters werden Heartbeats bis zum nächsten Tick innerhalb des Zeitfensters übersprungen.

</ParamField>

## Zustellungsverhalten

<AccordionGroup>
  <Accordion title="Sitzungs- und Zielweiterleitung">
    - Heartbeats werden standardmäßig in der Hauptsitzung des Agenten ausgeführt (`agent:<id>:<mainKey>`) oder in `global`, wenn `session.scope = "global"`. Legen Sie `session` fest, um stattdessen eine bestimmte Kanalsitzung (Discord/WhatsApp/usw.) zu verwenden.
    - `session` wirkt sich nur auf den Ausführungskontext aus; die Zustellung wird durch `target` und `to` gesteuert.
    - Legen Sie für die Zustellung an einen bestimmten Kanal/Empfänger `target` + `to` fest. Mit `target: "last"` wird für die Zustellung der letzte externe Kanal dieser Sitzung verwendet.
    - Heartbeat-Zustellungen erlauben standardmäßig Ziele für Direktnachrichten/DMs. Legen Sie `directPolicy: "block"` fest, um Sendungen an direkte Ziele zu unterdrücken, während der Heartbeat-Durchlauf weiterhin ausgeführt wird.
    - Wenn die Hauptwarteschlange, die Ziel-Sitzungsspur, die Cron-Spur oder ein aktiver Cron-Job ausgelastet ist, wird der Heartbeat übersprungen und später erneut versucht.
    - Wenn `target` kein externes Ziel ergibt, wird der Durchlauf dennoch ausgeführt, aber keine ausgehende Nachricht gesendet.

  </Accordion>
  <Accordion title="Sichtbarkeit und Überspringverhalten">
    - Wenn `showOk`, `showAlerts` und `useIndicator` alle deaktiviert sind, wird der Durchlauf vorab als `reason=alerts-disabled` übersprungen.
    - Wenn nur die Warnungszustellung deaktiviert ist, kann OpenClaw den Heartbeat weiterhin ausführen, die Zeitstempel fälliger Aufgaben aktualisieren, den Leerlaufzeitstempel der Sitzung wiederherstellen und die nach außen gerichtete Warnungsnutzlast unterdrücken.
    - Wenn das ermittelte Heartbeat-Ziel Tippanzeigen unterstützt, zeigt OpenClaw während des aktiven Heartbeat-Durchlaufs eine Tippanzeige an. Hierfür wird dasselbe Ziel verwendet, an das der Heartbeat Chat-Ausgaben senden würde; diese Funktion wird durch `typingMode: "never"` deaktiviert.

  </Accordion>
  <Accordion title="Sitzungslebenszyklus und Audit">
    - Reine Heartbeat-Antworten halten die Sitzung **nicht** aktiv. Heartbeat-Metadaten können die Sitzungszeile aktualisieren, aber für den Ablauf wegen Inaktivität wird `lastInteractionAt` aus der letzten echten Benutzer-/Kanalnachricht verwendet und für den täglichen Ablauf `sessionStartedAt`.
    - Im Verlauf der Control UI und von WebChat werden Heartbeat-Prompts und reine OK-Bestätigungen ausgeblendet. Das zugrunde liegende Sitzungstranskript kann diese Durchläufe weiterhin für Audit/Wiedergabe enthalten.
    - Abgetrennte [Hintergrundaufgaben](/de/automation/tasks) können ein Systemereignis in die Warteschlange stellen und den Heartbeat wecken, wenn die Hauptsitzung schnell auf etwas aufmerksam gemacht werden soll. Durch dieses Wecken wird der Heartbeat-Durchlauf nicht zu einer Hintergrundaufgabe.

  </Accordion>
</AccordionGroup>

## Sichtbarkeitssteuerung

Standardmäßig werden `HEARTBEAT_OK`-Bestätigungen unterdrückt, während Warnungsinhalte zugestellt werden. Sie können dies pro Kanal oder Konto anpassen:

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false # HEARTBEAT_OK ausblenden (Standard)
      showAlerts: true # Warnmeldungen anzeigen (Standard)
      useIndicator: true # Indikatorereignisse ausgeben (Standard)
  telegram:
    heartbeat:
      showOk: true # OK-Bestätigungen auf Telegram anzeigen
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Warnungszustellung für dieses Konto unterdrücken
```

Priorität: pro Konto → pro Kanal → Kanalstandards → integrierte Standardwerte.

### Funktion der einzelnen Flags

- `showOk`: Sendet eine `HEARTBEAT_OK`-Bestätigung, wenn das Modell eine reine OK-Antwort zurückgibt.
- `showAlerts`: Sendet den Warnungsinhalt, wenn das Modell eine Antwort zurückgibt, die nicht nur aus OK besteht.
- `useIndicator`: Gibt Indikatorereignisse für UI-Statusoberflächen aus.

Wenn **alle drei** auf „false“ gesetzt sind, überspringt OpenClaw den Heartbeat-Durchlauf vollständig (kein Modellaufruf).

### Beispiele für kanal- und kontospezifische Konfiguration

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # alle Slack-Konten
    accounts:
      ops:
        heartbeat:
          showAlerts: false # Warnungen nur für das ops-Konto unterdrücken
  telegram:
    heartbeat:
      showOk: true
```

### Häufige Muster

| Ziel                                              | Konfiguration                                                                            |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Standardverhalten (stille OKs, Warnungen aktiv)   | _(keine Konfiguration erforderlich)_                                                     |
| Vollständig still (keine Nachrichten, kein Indikator) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Nur Indikator (keine Nachrichten)                 | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }`  |
| OKs nur in einem Kanal                            | `channels.telegram.heartbeat: { showOk: true }`                                          |

## Monitor-Notizblock (optional)

Jeder Cron-Job des Heartbeat-Monitors besitzt ein privates Notizdokument, das in der gemeinsamen Zustandsdatenbank gespeichert ist. Betrachten Sie es als Ihre „Heartbeat-Checkliste“: klein, stabil und dafür geeignet, alle 30 Minuten berücksichtigt zu werden. Wenn ein Notizdokument vorhanden ist, wird sein Inhalt an den Heartbeat-Prompt angehängt.

Verwalten Sie es mit der Cron-CLI (die Job-ID stammt aus `openclaw cron list --all`):

```bash
openclaw cron scratch <jobId>                 # aktuellen Notizinhalt ausgeben
openclaw cron scratch <jobId> --set "..."     # durch den exakten Text ersetzen
openclaw cron scratch <jobId> --file notes.md # durch den Inhalt einer Datei ersetzen (- für stdin)
openclaw cron scratch <jobId> --unset         # entfernen
```

Schreibvorgänge sind durch Compare-and-Swap geschützt: Übergeben Sie `--expected-revision <n>`, damit der Vorgang bei einer gleichzeitigen Bearbeitung fehlschlägt, statt diese zu überschreiben. Das Notizdokument ist auf 256 KiB begrenzt und erscheint nie in der Ausgabe von `cron list`/`cron runs`.

Der Agent kann auch sein eigenes Notizdokument aktualisieren: Während eines Heartbeat-Durchlaufs akzeptiert `heartbeat_respond` eine optionale Zeichenfolge `scratch`, die das Notizdokument des Monitors für zukünftige Heartbeats vollständig ersetzt.

<Note>
**Migration von HEARTBEAT.md oder einer rein konfigurationsbasierten Frequenz?** Führen Sie `openclaw doctor --fix` aus. Doctor erstellt oder aktualisiert zunächst die systemeigenen Monitorzeilen aus `agents.*.heartbeat`, importiert anschließend die Datei `HEARTBEAT.md` aus dem Arbeitsbereich jedes Agenten in das Notizdokument des Monitors, konvertiert alle gültigen älteren `tasks:`-Einträge in Cron-Jobs, archiviert das Original im Zustandsverzeichnis (`backups/heartbeat-migration/`) und entfernt die Datei. Heartbeat-Anweisungen für die Laufzeit stammen ausschließlich aus dem Notizdokument der Datenbank; die Laufzeit liest `HEARTBEAT.md` niemals.
</Note>

Wenn ein Notizdokument vorhanden, aber praktisch leer ist (nur Leerzeilen, Markdown-/HTML-Kommentare, Markdown-Überschriften wie `# Heading`, Begrenzungsmarkierungen für Codeblöcke oder leere Checklisten-Einträge), überspringt OpenClaw den Heartbeat-Durchlauf, um API-Aufrufe zu sparen. Dieses Überspringen wird als `reason=empty-heartbeat-file` gemeldet. Wenn kein Notizdokument vorhanden ist, wird der Heartbeat dennoch ausgeführt und das Modell entscheidet, was zu tun ist.

Halten Sie es klein (kurze Checkliste oder Erinnerungen), um ein Aufblähen des Prompts zu vermeiden.

Beispiel für ein Notizdokument:

```md
# Heartbeat-Checkliste

- Kurz prüfen: Gibt es etwas Dringendes in den Posteingängen?
- Wenn es tagsüber ist, kurz und ressourcenschonend nachfragen, sofern nichts anderes aussteht.
- Wenn eine Aufgabe blockiert ist, notieren, _was fehlt_, und Peter beim nächsten Mal fragen.
```

### Wiederkehrende Prüfungen mit Cron planen

Das Heartbeat-Notizdokument ist Prompt-Kontext, kein Planer. Erstellen Sie jede wiederkehrende Prüfung als [Cron-Job](/de/automation/cron-jobs), damit sie eine eigene Frequenz, einen eigenen Aktivierungs-/Deaktivierungsstatus und einen eigenen Ausführungsverlauf besitzt. Cron-Jobs können weiterhin auf die Hauptsitzung abzielen, wenn die Prüfung den normalen Gesprächskontext verwenden soll.

Ältere Notizdokumente können einen strukturierten `tasks:`-Block enthalten. Führen Sie nach dem Upgrade einmal `openclaw doctor --fix` aus: Doctor konvertiert jeden gültigen Eintrag in einen unabhängig geplanten Cron-Job, behält dessen Intervall und den vorherigen Zeitpunkt der letzten Ausführung bei und entfernt den außer Betrieb genommenen Block, während der umgebende Notiztext erhalten bleibt. Heartbeat-Durchläufe der Laufzeit interpretieren `tasks:`-Text nicht als Zeitpläne.

Von Doctor erstellte Heartbeat-Aufgaben-Jobs behalten die aktiven Heartbeat-Zeiten sowie die Abkühlungs-, Überflutungs- und Auslastungsschutzmechanismen bei. Gleichzeitig fällige Jobs können zu einem Heartbeat-Durchlauf zusammengefasst werden. Ein Vorkommen außerhalb der aktiven Zeiten wird übersprungen und beim nächsten Cron-Vorkommen erneut versucht.

### Kann der Agent sein Notizdokument aktualisieren?

Ja. Während eines Heartbeat-Durchlaufs kann der Agent einen `scratch`-Wert an `heartbeat_respond` übergeben, um den Monitor-Text für zukünftige Heartbeats vollständig zu ersetzen. Sie können ihn auch in einem normalen Chat auffordern, `openclaw cron scratch <jobId> --set ...` auszuführen, oder das Notizdokument selbst mit demselben Befehl bearbeiten. Verwalten Sie wiederkehrende Zeitpläne mit Cron, anstatt Planersyntax in das Notizdokument zu schreiben.

<Warning>
Speichern Sie keine Geheimnisse (API-Schlüssel, Telefonnummern, private Tokens) im Monitor-Notizdokument – es wird Teil des Prompt-Kontexts.
</Warning>

## Manuelles Wecken (bei Bedarf)

Verwenden Sie `openclaw system event`, um ein Systemereignis in die Warteschlange zu stellen und optional sofort einen Heartbeat auszulösen:

```bash
openclaw system event --text "Auf dringende Nachfassaktionen prüfen" --mode now
```

| Flag                         | Beschreibung                                                                                      |
| ---------------------------- | ------------------------------------------------------------------------------------------------ |
| `--text <text>`              | Systemereignistext (erforderlich).                                                                    |
| `--mode <mode>`              | `now` führt sofort einen Heartbeat aus; `next-heartbeat` (Standard) wartet auf den nächsten geplanten Takt. |
| `--session-key <sessionKey>` | Richtet das Ereignis an eine bestimmte Sitzung; standardmäßig wird die Hauptsitzung des Agenten verwendet.                   |
| `--json`                     | Gibt JSON aus.                                                                                     |

Wenn kein `--session-key` angegeben ist und für mehrere Agenten `heartbeat` konfiguriert ist, führt `--mode now` die Heartbeats all dieser Agenten sofort aus.

Zugehörige Heartbeat-Steuerelemente in derselben CLI-Gruppe:

```bash
openclaw system heartbeat last     # letztes Heartbeat-Ereignis anzeigen
openclaw system heartbeat enable   # Heartbeats aktivieren
openclaw system heartbeat disable  # Heartbeats deaktivieren
```

## Kostenbewusstsein

Heartbeats führen vollständige Agentendurchläufe aus. Kürzere Intervalle verbrauchen mehr Tokens. So reduzieren Sie die Kosten:

- Verwenden Sie `isolatedSession: true`, um das Senden des vollständigen Gesprächsverlaufs zu vermeiden (von ~100K Tokens auf ~2-5K pro Durchlauf).
- Verwenden Sie `lightContext: true`, um Workspace-Bootstrap-Dateien bei Heartbeat-Durchläufen zu überspringen.
- Legen Sie ein kostengünstigeres `model` fest (z. B. `ollama/llama3.2:1b`).
- Halten Sie das Monitor-Notizdokument klein.
- Verwenden Sie `target: "none"`, wenn Sie nur interne Statusaktualisierungen wünschen.

## Kontextüberlauf nach einem Heartbeat

Heartbeats behalten nach Abschluss des Durchlaufs das bestehende Laufzeitmodell der gemeinsam genutzten Sitzung bei. Daher kann ein Heartbeat, der eine Sitzung auf ein kleineres lokales Modell umgestellt hat (beispielsweise ein Ollama-Modell mit einem 32k-Kontextfenster), dieses Modell für den nächsten Durchlauf der Hauptsitzung beibehalten. Wenn dieser nächste Durchlauf dann einen Kontextüberlauf meldet und das zuletzt verwendete Laufzeitmodell der Sitzung dem konfigurierten `heartbeat.model` entspricht, nennt die Wiederherstellungsmeldung von OpenClaw das Übergreifen des Heartbeat-Modells als wahrscheinliche Ursache und schlägt eine Lösung vor.

Um dies zu vermeiden, verwenden Sie `isolatedSession: true`, damit Heartbeats in einer neuen Sitzung ausgeführt werden (optional zusammen mit `lightContext: true` für den kleinstmöglichen Prompt), oder wählen Sie ein Heartbeat-Modell mit einem Kontextfenster, das groß genug für die gemeinsam genutzte Sitzung ist.

## Verwandte Themen

- [Automatisierung](/de/automation) – alle Automatisierungsmechanismen auf einen Blick
- [Hintergrundaufgaben](/de/automation/tasks) – wie abgekoppelte Aufgaben nachverfolgt werden
- [Zeitzone](/de/concepts/timezone) – wie sich die Zeitzone auf die Heartbeat-Planung auswirkt
- [Fehlerbehebung](/de/automation/cron-jobs#troubleshooting) – Automatisierungsprobleme diagnostizieren
