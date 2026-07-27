---
read_when:
    - Sie implementieren oder überprüfen eine Phase der Neugestaltung des Onboardings.
summary: Implementierungsplan für die Neugestaltung des Custodian-Onboardings (fortlaufend aktualisiertes Dokument)
title: Neugestaltung des Onboardings
x-i18n:
    generated_at: "2026-07-26T18:49:41Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: f892991583d0b77a670e9bf7aa5a0c74af3b3eac9e7b0448706486254eb7e2a0
    source_path: start/onboarding-redesign.md
    workflow: 16
---

# Implementierungsplan für die Neugestaltung des Onboardings

> **Lebendes Dokument.** Diese Seite verfolgt die Neugestaltung des Custodian-Onboardings auf
> Implementierungsebene und wird aktualisiert, sobald die einzelnen Phasen umgesetzt werden. Wenn die letzte Phase
> zusammengeführt wurde, wird diese Seite als benutzerorientierter Onboarding-Leitfaden neu verfasst und in
> die Dokumentationsnavigation aufgenommen. Bis dahin ist sie absichtlich nicht in `docs.json` enthalten.

## Leitbild

Ein technisch nicht versierter Benutzer gibt `openclaw onboard` ein (oder öffnet die App) und wird
von einer einzigen dialogorientierten Präsenz begrüßt — OpenClaw, dem System-Custodian („Custodian“ ist
nur die interne Bezeichnung; dem Benutzer wird immer „OpenClaw“ angezeigt) —, die seine KI findet,
alles mit angekündigten Standardwerten statt durch Fragen einrichtet, seinen
Agenten in einem sichtbaren Identitätsmoment schlüpfen lässt und danach für immer als
Betreuer des Systems erreichbar bleibt. Standardmäßig magisch, eine einzige Einwilligungsgrenze, keine Sackgassen.

Designprinzipien (beschlossen, nicht beiläufig erneut zur Diskussion stellen):

- **Angekündigte Standardwerte mit einfacher Rückgängigmachung** ersetzen blockierende Fragen. Die einzige
  zwingende Voraussetzung ist eine funktionierende Inferenz; alles andere ist ein Angebot.
- **Frage null ist die Einwilligungsgrenze**: „Vollzugriff“ (empfohlen) bedeutet,
  dass die Erkennung still und automatisch erfolgt; „Zuerst fragen“ stellt jeder Erkennung — dem KI-
  Scan, dem App-Scan und dem Scan von Speicherquellen gleichermaßen — ein
  ausdrückliches Ja voran, mit einem vollständig manuellen Pfad, der niemals scannt.
- **Konversation als Benutzeroberfläche mit progressiver Intelligenz**: Die Custodian-Oberfläche
  existiert bereits, bevor eine KI funktioniert (skriptgesteuerter Dialog), wird in dem
  Moment modellgestützt, in dem eine Route verifiziert wurde, und weist sichtbar darauf hin. Sie täuscht niemals Intelligenz vor:
  Freitexteingaben, bevor eine Route verifiziert wurde, erhalten ein freundliches „Lassen Sie mich zuerst
  mein Gehirn zum Laufen bringen“.
- **Das Schlüpfen ist eine Zeremonie**: derselbe Thread, Wechsel des Avatars, der Agent gibt sich selbst
  einen Namen und wählt sein eigenes Gesicht. Der Custodian erklärt die Hierarchie einmal: „Fragen Sie mich
  nach dem System oder fragen Sie einfach Ihren Agenten — er leitet es weiter.“
- **Vertrauen ist nach Quelle abgestuft**: Einträge aus dem offiziellen Katalog dürfen vorausgewählt sein;
  Skills von Drittanbietern aus ClawHub werden unabhängig vom Modell-
  Ranking niemals vorausgewählt, und ihre Beschriftungen weisen darauf hin, dass sie den Code des Herausgebers installieren.
- **Konfigurierte Installationen sind unantastbar**: Eine erneute Ausführung des Onboardings ist ein
  Verifizierungsdurchlauf. Sie wendet die Einrichtung niemals erneut an und startet den Gateway-Dienst niemals neu.
- **Das Terminal ist die Rückfallebene, keine Frage**: Bevorzugen Sie das Browser-
  Dashboard, wenn ein Gateway erreichbar ist; fragen Sie niemals „Terminal oder Browser?“.
- **Schwache Modelle erhalten eine reduzierte Oberfläche** (automatisch `localModelLean`), erklärt in
  einfachen Worten — niemals anhand von Tools, Codemodus oder Kontextfenstern.

## Aktuell ausgelieferter Ablauf (nach den Phasen 1–3)

`openclaw onboard` bei einer neuen macOS-Installation, idealer Ablauf — insgesamt viermal Eingabetaste:

1. Sicherheitshinweis → einmal Eingabetaste zur Bestätigung (dauerhaft gespeichert; wird nie wieder abgefragt).
2. **Frage null**: „Wie soll ich alles einrichten?“ — Vollzugriff (empfohlen)
   oder Zuerst fragen. Wird als `wizard.accessMode` gespeichert; bei erneuter Ausführung wird standardmäßig die gespeicherte
   Auswahl verwendet. Abgesichert + „manuell konfigurieren“ führt ohne
   jeglichen Scan zur Provider-Auswahl und überspringt auch das Scannen von Speicherquellen.
3. **Inszenierte Erkennung**: erkennt Coding-CLIs, Umgebungsschlüssel und lokale Laufzeitumgebungen;
   gibt bei gefundenen Coding-Agenten einen kurzen Hinweis aus; testet Kandidaten der Reihe nach live und
   fasst Fehlschläge unauffällig in einer einzigen Übersichtszeile zusammen (Details unter „Weitere
   Optionen anzeigen“). Die erste funktionierende Route wird als Standardwert angekündigt, mit einem
   Ein-Tasten-Pfad zur vollständigen Auswahl; das Erkunden und Überspringen behält die
   funktionierende Route bei.
4. Angebot zum Speicherimport (Claude Code / Codex / Hermes), wird übersprungen, wenn die Erkennung
   abgelehnt wurde.
5. Nur bei neuen Installationen: Der Standard-Einrichtungsplan wird automatisch angewendet
   (Arbeitsbereich, Gateway-Dienst, Sitzungen — derselbe Plan, den das dialogorientierte
   „Ja“ ausführt). Konfigurierte Installationen geben „bereits eingerichtet“ aus und verändern den
   Dienst niemals.
6. **App-Empfehlungen**: Installierte Apps werden vom verifizierten Modell
   mit offiziellen Katalogen + ClawHub abgeglichen; offizielle Channel-Plugins sind
   vorausgewählt, Skills von Drittanbietern sind optional und mit einem Warnhinweis versehen. Überspringbar;
   Deaktivierungsschalter `wizard.appRecommendations`.
7. **Schlüpfen**: Wenn ein Gateway erreichbar ist, öffnet die Browser-Übergabe (GUI) das
   Dashboard oder gibt (headless/SSH) dessen URL aus und wartet darauf, dass sich die Control UI
   verbindet — „Dashboard verbunden — Fortsetzung im Browser.“ Andernfalls oder
   mit `--tui` wird die Terminal-TUI mit der Bootstrap-Schlüpf-
   Nachricht vorbelegt geöffnet, und der Agent stellt sich vor.

Das Onboarding für ein Remote-Gateway behält seine bisherige dialogorientierte Übergabe
(`handoffMode: "chat"`) bei; die Einrichtung muss auf dem Remote-Gateway angewendet werden.

## Phasen

| #   | Phase                                                                                                                                                     | Oberfläche              | Status                                                                                                                            |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Plugin-Empfehlungen für installierte Apps (Scan, Kandidaten, KI-Abgleich, Assistentenschritt, Node-Befehl `device.apps`)                                   | klassische + geführte CLI | zusammengeführt ([#109668](https://github.com/openclaw/openclaw/pull/109668))                                                   |
| 2   | CLI-Custodian-Grundgerüst (Frage null, inszenierte Erkennung, automatische Anwendung + Schlüpfen)                                                          | geführte CLI            | zusammengeführt ([`a83ed13204f1`](https://github.com/openclaw/openclaw/commit/a83ed13204f118adf1009e5ac88d5afe1905b86c))              |
| 3   | Browserorientierte Übergabe (Erkennung der GUI-Sitzung, Warten auf Dashboard-Verbindung, TUI als Rückfallebene)                                           | CLI → Web               | zusammengeführt ([#110054](https://github.com/openclaw/openclaw/pull/110054))                                                   |
| 4   | Web-Custodian-Oberfläche (Optionskarten, typisiertes Feld `question` in `openclaw.chat`, Spiegelung der Assistentenschritte, Übergabe beim ersten Start) | Control UI              | zusammengeführt ([#110141](https://github.com/openclaw/openclaw/pull/110141), [#110242](https://github.com/openclaw/openclaw/pull/110242)) |
| 5   | Schlüpfen und Bootstrap (Empfehlungsspeicher mit Einmal-Semantik, selbstbenennende Geburtssequenz, automatische Schlüpf-Übergabe nach neuer Einrichtung; Avatar-Abstufung zurückgestellt) | Agent-Bootstrap         | zusammengeführt ([#110173](https://github.com/openclaw/openclaw/pull/110173), [#110331](https://github.com/openclaw/openclaw/pull/110331)) |
| 6   | Custodian-Präsenz PR1 (angehefteter Seitenleisteneintrag, OpenClaw fragen in den Einstellungen, Betreuerbegrüßung in der normalen Oberfläche; Ereigniskommentare und Channel-Aufruf folgen in PR2) | Web + Channels          | zusammengeführt ([#110269](https://github.com/openclaw/openclaw/pull/110269))                                                   |
| 7   | Ausfallsicherheit (Custodian bei fehlerhafter Konfiguration erreichbar, Rettung teilweise verfügbarer Oberflächen, automatischer Doctor)                  | Gateway                 | Folgearbeit                                                                                                                       |

## Implementierungshinweise je Phase

### Phase 1 — App-Empfehlungen (PR #109668)

- Scanner: `src/infra/installed-apps.ts` (macOS-Aufzählung ohne TCC; folgt
  symbolisch verknüpften `.app`-Bundles).
- Kandidaten: offizielle Kataloge + ClawHub-Suche, insgesamt 20s Zeitbudget, kontrollierte
  Offline-Degradierung auf reine Katalogkandidaten. Katalogeinträge sind Paket-
  Manifeste ohne `id` auf oberster Ebene — Kandidaten werden anhand der aufgelösten
  Plugin-ID indiziert (Regressionstest mit den tatsächlichen gebündelten Katalogen; eine Indizierung nach
  `entry.id` fasste einst den gesamten Katalog zusammen und verwarf jede offizielle
  Empfehlung).
- KI-Abgleich: eine Vervollständigung über die verifizierte Route
  (`src/system-agent/setup-app-recommendations.ts`); keine kuratierte Bundle-ID-Zuordnung —
  das Modell verwirft zufällige Namensüberschneidungen. Die Ausgabe wird durch das eigene
  `maxTokens`-Budget des aufgelösten Modells begrenzt (die Streaming-Schicht wendet es an, wenn keine
  ausdrückliche Obergrenze übergeben wird).
- **Schutz der Lieferkette**: Der Beschreibungstext eines ClawHub-Eintrags wird vom Herausgeber kontrolliert und
  gelangt in den Abgleichs-Prompt, sodass ein Eintrag sich selbst als
  „empfohlen“ bewerben kann. Nur Einträge aus dem offiziellen Katalog dürfen vorausgewählt werden; ClawHub-
  Skills erfordern immer eine ausdrückliche Auswahl und sind mit „ClawHub-Skill eines Drittanbieters;
  installiert den Code seines Herausgebers“ gekennzeichnet.
- Node-Befehl `device.apps` (TS-Node-Host, Parität mit Android-Envelope), Freigabe
  standardmäßig deaktiviert; Gateway-Deaktivierungsschalter `wizard.appRecommendations`.
- Die Bereitstellung erfolgt im klassischen Assistenten und im geführten Custodian-Ablauf
  (`src/wizard/setup.app-recommendations.ts`); die Verlagerung an das Ende des Bootstraps
  bleibt Phase 5 vorbehalten (der Dienst akzeptiert bereits eine injizierbare Inventarquelle).
  Die Einmal-Semantik (Angebot nur bis zur Annahme, gespeicherter Scan) wird ebenfalls
  mit dem Speicher aus Phase 5 umgesetzt; derzeit wird das Angebot bei einer erneuten Ausführung erneut angezeigt.
- Ebenfalls behoben: Benutzerdefinierte `completeSetupInference`-Prompts übernehmen nicht mehr die
  Ausgabebegrenzung der Verifizierungsprüfung auf 32 Token (`SETUP_INFERENCE_TEST_MAX_TOKENS`
  gilt nur für die „reply OK“-Prüfung).

### Phase 2 — CLI-Custodian-Grundgerüst (PR #109841)

- Überarbeitung des Ablaufs in `src/commands/onboard-guided.ts`; das Onboarding für Remote-Gateways
  behält seine bisherige Chat-Übergabe über `handoffMode: "chat"` bei.
- Frage null speichert `wizard.accessMode` („full“ | „guarded“); bei erneuter Ausführung
  wird standardmäßig die gespeicherte Auswahl verwendet (durch Annahme des Standardwerts kann „guarded“ niemals unbemerkt
  auf „full“ herabgestuft werden). Abgesichert + manuell verwendet
  `listManualSetupInferenceOptions` (nur Konfiguration/Manifeste, keine Prüfungen) und
  überspringt das Scannen von Speicherquellen.
- Erkennung: unauffällige Sammlung von Fehlschlägen (eine einzige Übersichtszeile; Details unter
  „Weitere Optionen anzeigen“), Hinweis auf Coding-Agenten, angekündigter Routenstandard. Sitzungs-
  anzahlen im Hinweis sind zurückgestellt (nur qualitativ), bis eine kostengünstige
  Schnittstelle zur Sitzungszählung existiert.
- Neue Installationen: `applySystemAgentSetup` (das deterministische dialogorientierte
  „Ja“), anschließend Schlüpfen über `launchTuiCli`, vorbelegt mit der Bootstrap-Nachricht.
  Konfigurierte Installationen (bereits vorhandene Modell- oder Gateway-Konfiguration — Assistenten-
  Zeitstempel beweisen nichts, da sie mit Konfiguration/Doctor geteilt werden):
  nur Verifizierung — keine Anwendung, kein Neustart des Gateway-Dienstes. Falls die Anwendung
  fehlschlägt, wird auf den dialogorientierten Chat zurückgegriffen.

### Phase 3 — browserorientierte Übergabe (PR #110054, zusammengeführt)

- `src/commands/onboard-browser-handoff.ts` ist für die reine Erkennung grafischer Sitzungen
  zuständig (`SSH_CONNECTION`/`SSH_TTY`; `DISPLAY`/`WAYLAND_DISPLAY` unter Linux)
  sowie für die Wartezeit von 60 Sekunden für die GUI bzw. 300 Sekunden für SSH. Das geführte Onboarding
  aktiviert die Übergabe derzeit nur unter macOS; `--tui` und andere Plattformen behalten den
  Ausweg über das Terminal. Die Aktivierung unter Linux/Windows ist eine Folgeaufgabe.
- Dashboard-Links verwenden dieselben Hilfsfunktionen `resolveAdvertisedControlUiLinks`,
  `resolveLocalControlUiProbeLinks` und `buildOnboardingControlUiUrl`
  wie der klassische Abschluss. Zum Starten des Browsers wird die gemeinsame Hilfsfunktion `openUrl` verwendet.
- Die Bereitschaftsprüfung fragt den vorhandenen RPC `system-presence` als **Loopback-Client
  im CLI-Modus ab, der das konfigurierte gemeinsame Secret vorlegt** — der vertrauenswürdige Pfad, den jeder
  Befehl `openclaw` verwendet. Ein Control-UI-Client mit einfacher gemeinsamer Authentifizierung wird
  auf SecretRef-Gateways mit „device identity required“ abgewiesen. Die Vorabprüfung
  der Erreichbarkeit löst dasselbe Ziel (und Secret) wie die Warteschleife auf, sodass
  Gate und Warteschleife bei der Authentifizierung niemals zu unterschiedlichen Ergebnissen kommen können. Die Übergabe wird erst abgeschlossen,
  wenn eine verbundene Präsenzzeile `openclaw-control-ui`/`webchat` gegenüber
  dem Ausgangszustand vor dem Start neu ist (ein bereits geöffnetes Dashboard kann
  sie nicht abschließen).
- `gateway.controlUi.enabled: false` bricht ab, bevor eine URL angezeigt wird.
- End-to-End gegen ein isoliertes Gateway mit derselben Konfiguration nachgewiesen: URL-Ausgabe → echte
  Browserverbindung → „Dashboard verbunden — Fortsetzung in Ihrem Browser“ → kein
  Ausweg über das Terminal. Ein früheres Anhalten wegen „token mismatch“ war ein Artefakt
  des Test-Harness — siehe das Test-Playbook unten.

### Phase 4 — Web-Oberfläche des Custodian (zusammengeführt: #110141, #110242)

- `/custodian`-Seite über `openclaw.chat` mit der Optionskarten-Komponente
  (2–4 Karten, höchstens eine empfohlen, stets überspringbar); Onboarding-Rahmen über
  `?onboarding=1`; der Abschluss der Model-Ersteinrichtung übergibt dorthin.
- Strukturierte Fragen sind ein typisiertes additives Feld `question` auf
  `SystemAgentChatResult` (Text `reply` je Option; Prosa steht für
  die macOS-App/TUI stets eigenständig). Erzeuger: beide Varianten der Onboarding-Begrüßung und
  Auswahl-/Bestätigungsschritte des gehosteten Assistenten mit 2–4 geschlossenen Optionen — echte Kanalassistenten
  stellen sie als Karten dar. Die vorläufige Lösung mit String-Markern aus PR1 wurde entfernt.
- Der Sitzungsbesitz ist auf die Gateway-URL und alle vorgelegten Anmeldedaten
  beschränkt (Token, Passwort, Bootstrap-Token, gespeichertes Geräte-Token — bleibt
  über vorübergehende Hello-Verbindungsabbrüche hinweg bestehen); fehlgeschlagene Benutzereingaben können niemals erneut abgespielt werden; sensible
  Eingaben werden unverändert gesendet und im Transkript maskiert.

### Phase 5 — Ausstieg und Bootstrap (zusammengeführt: #110173, #110331)

- Der Custodian erstellt einen namenlosen Agenten (Tool-Aufruf); der Bootstrap des Agenten beginnt
  mit der eigenen Namensgebung. PR1 liefert die Zeremonie, begrenzt auf drei Schritte (Name → Seelenzeile
  → Skills-Frage), und verschiebt die Stufenfolge für selbst gezeichnete Avatare/Bilderzeugung
  (vom Model generierte Kandidaten → voreingestellte Zeichen → Logo beibehalten) auf eine Folgeaufgabe. Derselbe
  Thread, Austausch des Avatars; das Krallenzeichen bleibt dem Custodian vorbehalten. Die
  vereinbarte Identität wird zweifach gespeichert: in `IDENTITY.md`/`SOUL.md` (was der Agent
  liest) und über `openclaw agents set-identity` (was Kanäle und die UI
  anzeigen).
- Empfehlungen (Dienst aus Phase 1, gespeicherter Scan mit Einmal-Semantik) werden als
  letzter Bootstrap-Schritt umgesetzt, bevor die Bootstrap-Datei entfernt wird: „Minimale Auswahl
  oder maximaler Komfort?“ Der Bootstrap liest das gespeicherte Angebot über
  `openclaw onboard recommendations --json` (nur undurchsichtige Installations-IDs) und
  bestätigt es, nachdem die Auswahl verarbeitet wurde, sodass die Frage nie erneut gestellt wird. Schaltflächen zum
  Verbinden von Kanälen enthalten kanalspezifische Einrichtungs-Playbooks; der Agent erfasst
  Anmeldedaten im Dialog und leitet Konfigurationsschreibvorgänge an den Custodian weiter
  („OpenClaw wird gefragt …“ ist die kanonische Formulierung).
- Selbstlernen wird erfragt, nicht angekündigt, und dient zugleich als Zustimmung zum Skills-Workshop;
  beschreiben Sie die Prüfungen von ClawHub zu Release-Vertrauen, Scan, Verifizierung und Integrität
  sowie den Warnhinweis zum Publisher-Code — erwecken Sie niemals den Eindruck, jedes Release sei signiert.
- Der automatische Ausstieg wurde ausgeliefert: Das Anwenden einer Neueinrichtung kündigt den Ausstieg an und
  übergibt (Terminal-TUI / `open-agent` für Gateway-Clients); die Webseite
  wechselt in den Agenten-Chat, wobei der Entwurf „Wach auf, mein Freund!“ vorausgefüllt ist. Die
  Übergabe erfolgt nur nach einer fehlerfreien Überprüfung nach dem Schreiben. Nach
  dem Löschen bei null Agenten eine Option anzubieten (statt automatisch zu handeln), bleibt eine nachgelagerte Verfeinerung.

### Phase 6 — Präsenz des Custodian (PR1 zusammengeführt: #110269; Kommentare/Aufruf folgen in PR2)

- In PR1 ausgeliefert: standardmäßig angehefteter Seitenleisteneintrag „OpenClaw“ (neue Profile;
  bestehende Benutzer behalten gespeicherte Anheftungen und erreichen ihn über Anpassen/Mehr), „OpenClaw
  fragen“ als erster Eintrag in den Einstellungen sowie Besuche von `/custodian` im normalen Rahmen,
  die die Begrüßung durch den Betreuer anfordern (keine Variante der Onboarding-Begrüßung), wobei
  „Einrichtung beenden“ nur im Onboarding-Modus dargestellt wird. Ein angedockter, eingebetteter Einstellungsbereich
  erfordert die Extraktion einer gemeinsamen Konversationsansicht (Folgeaufgabe).
- Ereignisreaktive Kommentare mit Anti-Clippy-Leitplanken: nur bei folgenreichen oder
  fehlgeschlagenen Änderungen, höchstens einmal je Besuch der Einstellungen, sofern nicht angefordert. Dieselbe
  Ereignisschnittstelle macht den Custodian später zur Stimme für beeinträchtigte Authentifizierung oder defekte
  Kanäle.
- Kanäle: im Alltag unsichtbar (der Agent leitet weiter); erreichbar durch expliziten
  Aufruf und bei Ausfallereignissen des Agenten im selben Thread, mit eigenem Namen und
  Krallen-Avatar, sofern die Plattform dies zulässt.
- Bei der Einrichtung wird ein schwaches Model erkannt: `localModelLean` automatisch setzen, und der Custodian
  erklärt dies in klaren Worten und bietet ein Upgrade an.
- Der Custodian kennt seinen internen Spitznamen („Manche nennen mich den
  Custodian — OpenClaw ist auch in Ordnung“) und bezeichnet den Agenten stets mit seinem Namen.

### Phase 7 — Resilienz (erfordert vor der Umsetzung eine Entscheidung des Owners)

Der ursprüngliche Entwurf — „Der Custodian muss erreichbar sein, unabhängig davon, wie defekt
die Konfiguration ist“ — kollidiert mit der Sicherheitsrichtlinie des Repositorys: Der Root-Leitfaden
besagt, dass das Gateway den **Start verweigert**, wenn die Konfiguration strukturell ungültig ist,
und nur Fehler bei SecretRef-Ownern zu konfiguriert nicht verfügbaren
Fähigkeiten führen. Bei einer ungültigen Konfiguration irgendeine Oberfläche bereitzustellen, ist eine Richtlinienänderung,
kein Implementierungsdetail. Zwei Umfänge, wählen Sie einen:

- **Option A (empfohlen, richtlinienkonform): Automatischer Doctor auf CLI-Seite.** Wenn der
  Start eines Gateways oder der CLI aufgrund einer ungültigen Konfiguration mit bekannter Struktur fehlschlägt, bietet die CLI
  `openclaw doctor --fix` an (oder führt es mit Zustimmung aus), versucht den Start anschließend einmal erneut und
  meldet das Ergebnis verständlich. Das Verhalten des Gateways ändert sich nicht; der Custodian bleibt
  über den vorhandenen Pfad für beeinträchtigte SecretRefs und das Terminal erreichbar.
- **Option B (erfordert ausdrückliche Zustimmung des Owners und eine Sicherheitsprüfung): Modus mit minimaler
  Gateway-Oberfläche.** Bei einer strukturell ungültigen Konfiguration wird eine gesperrte
  Oberfläche gestartet, die ausschließlich die Konversation mit dem Custodian und Doctor-Aktionen bereitstellt. Dies
  ändert den Fail-Closed-Startvertrag und muss vor jeglicher Implementierung ein eigenes Konzept
  zum Schutz des Zugangs definieren.

Verbleibende Folgeaufgaben aus den Phasen 4–6 (erfasst, nicht terminiert): Stufenfolge für Avatare/Bilderzeugung
für den Ausstieg; Darstellung des typisierten Feldes `question` in der macOS-App; ein
angedockter, eingebetteter Einstellungsbereich für den Custodian (erfordert die Extraktion einer gemeinsamen
Konversationsansicht); ereignisreaktive Kommentare und Kanalaufruf/Wiederherstellung bei Agentenausfall
(PR2 von Phase 6); automatisches `localModelLean` für schwache Models; ob gespeicherte
Seitenleisten-Anheftungen bestehender Benutzer den OpenClaw-Eintrag übernehmen sollen.

## Test- und Landing-Playbook (mühsam erarbeitet; vor den Phasen 4–6 lesen)

- **`OPENCLAW_STATE_DIR` isoliert den Gateway-Dienst nicht.** Das
  LaunchAgent-Label (`ai.openclaw.gateway`) gilt für die gesamte Maschine: Ein Onboarding-Test
  für eine Neuinstallation mit isoliertem Zustandsverzeichnis schreibt den echten Dienst der
  Maschine NEU und STARTET ihn NEU (Wrapper-Skripte landen im isolierten Verzeichnis; der nächste
  Dienststart schlägt fehl, wenn dieses Verzeichnis bereinigt wird). Stellen Sie nach jedem Test einer Neuinstallation
  mit `openclaw gateway install --force && openclaw gateway
restart` aus der echten Umgebung den Zustand wieder her und überprüfen Sie die plist. Produkt-Folgeaufgabe:
  auf das Zustandsverzeichnis beschränkte Dienst-Labels oder Erkennung eines fremden Dienstes durch das Onboarding.
- **Sicherer End-to-End-Harness**: Füllen Sie die isolierte Konfiguration vorab mit einem Abschnitt `gateway`
  (damit das Onboarding den Pfad für eine konfigurierte Installation verwendet und den Dienst niemals berührt), und führen Sie
  `openclaw gateway run` als einfachen Vordergrundprozess an einem freien Port mit einem einfachen Token aus.
  Dieser Harness hat die Schleife aus Phase 3 einschließlich einer echten Browserverbindung nachgewiesen.
- **Authentifizierungspfade unterscheiden sich nach Clientidentität, nicht nur nach Anmeldedaten.** Präsenz-
  und andere Operator-Lesezugriffe verwenden einen Loopback-Client im CLI-Modus mit Anmeldedaten aus derselben
  Konfiguration. Gateways mit Token-Authentifizierung benötigen das gemeinsame Secret; SecretRef-/None-
  Gateways können ohne Token auf die vertrauenswürdige Loopback-Authentifizierung zurückgreifen. Ein als Control
  UI identifizierter Browser-Client benötigt eine Geräteidentität oder die Loopback-Freigabe
  im sicheren Kontext. Ein Probezugriff, der sich gegenüber einem Gateway authentifiziert, das eine
  ANDERE Konfiguration bereitstellt (siehe LaunchAgent-Fallstrick), schlägt mit „token mismatch“ fehl — dieses
  Artefakt hielt Phase 3 kurzzeitig auf.
- **Abschluss-Probes**: `runSetupInferenceTest` begrenzt den Verifizierungs-Probezugriff auf
  32 Ausgabetoken; benutzerdefinierte Prompts umgehen die Begrenzung und werden durch das
  eigene `maxTokens` des Models begrenzt. Reasoning-Models verbrauchen dieses Budget zuerst mit verborgenem
  Reasoning — eine Ausgabe ohne Text bedeutet normalerweise, dass das Budget dort aufgebraucht wurde.
- **Das Landing eines Agenten erfordert gehostete CI am exakten Head.** Der aufwendige Workflow `CI` wird
  bei hoher Organisationslast möglicherweise nicht für Pushes eingereiht; der Fallback für Maintainer ist eine
  Ausführung des Release-Gates auf dem PR-Branch:

  ```bash
  gh workflow run ci.yml --ref <branch> -f target_ref=<head-sha> -f release_gate=true -f pull_request_number=<pr>
  ```

  Der Lauf muss auf dem
  Branch-Ref erfolgen, damit `head_sha` übereinstimmt, und der Titel wird zu
  `CI release gate <sha>`, was `scripts/verify-pr-hosted-gates.mjs`
  akzeptiert. Anschließend wie üblich mit `scripts/pr` vorbereiten/zusammenführen.

- **Gates, die CI zusätzlich zu fokussierten Tests durchsetzt**: Dokumentationszuordnung
  (`pnpm docs:map:gen` nach dem Hinzufügen einer beliebigen Dokumentationsseite), oxlint (`no-map-spread`,
  `max-lines` — Dateien aufteilen, niemals unterdrücken), `check:test-types`, knip-
  Deadcode (nur exportieren, was die Produktion verwendet; Tests über öffentliche APIs leiten)
  und der Shard-Klassifikator für Live-Tests
  (`test/scripts/test-live-shard.test.ts` muss jedes neue `*.live.test.ts` aufführen).

## Entscheidungsprotokoll

- Magischer Scan mit Abbruchschalter, nicht nach dem Consent-first-Prinzip (Phase 1; die persistente Ausgabe
  legt die Nutzung des Modells und von ClawHub vor dem Scan offen, und der Ergebnishinweis wiederholt dies).
- Vollständiger vertikaler Ablauf einschließlich des Node-Befehls `device.apps` (Phase 1).
- Skills von Drittanbietern aus ClawHub sind niemals vorausgewählt und werden als
  Installation des Codes des Herausgebers gekennzeichnet; offizielle Einträge können vorausgewählt sein
  (Phase 1, ausgelieferte Sicherheitskonfiguration).
- Zwei Zugriffskarten, nicht drei; die Einwilligung ist der Auswahl vorangestellt (Phase 2).
- Automatisches Schlüpfen mit Ankündigung statt einer blockierenden Schaltfläche (Phasen 2/5).
- Browser-first: Das Schlüpfen im Terminal ist die Ausweichlösung, niemals eine Frage „Terminal oder
  Browser?“ (Phase 3).
- Der Kustos erhält Kanalpräsenz (Herbeirufen + Wiederherstellung), nicht nur Web/CLI
  (Phase 6).
- Das Schlüpfen erfolgt im selben Thread mit einem Avatarwechsel; nach Abschluss wechselt die
  App zur regulären Benutzeroberfläche (Phase 5).
- Der Einstellungsbereich behält den Namen „Einstellungen“; der Kustos befindet sich dort
  (und in der Seitenleiste), statt ihn zu ersetzen (Phase 6).
- Optionskarten unterliegen Beschränkungen: 2–4 Optionen, genau eine Empfehlung, stets
  überspringbar; dieselbe Komponente dient dem Onboarding und dem Fragetool des Agenten
  (Phase 4).
- „OpenClaw wird gefragt …“ ist die kanonische Formulierung für Delegierung; Souls dürfen für mehr Charakter sorgen,
  die Beschreibung der Tool-Aktivität bleibt sachlich (Phase 5).
- Benutzerseitige Texte verwenden bei der Erklärung der Kürzung für schwache Modelle niemals
  „Codemodus“, „Tools“ oder „Kontextfenster“ (Phase 6).

## Bekannte Lücken und Folgearbeiten

- Das LaunchAgent-Label ist nicht auf das Zustandsverzeichnis beschränkt (oben beschriebene Testfalle; außerdem eine
  echte Produktlücke bei mehreren Instanzen).
- Einmalige Semantik der Empfehlungen und der gespeicherte Scan (Phase 5); bei erneuten Ausführungen
  werden sie derzeit erneut angeboten.
- Die Browser-Übergabe ist nur unter macOS verfügbar; die Unterstützung für Linux/Windows steht noch aus.
- Der Kommentar zur Sitzungsanzahl ist qualitativ; für Anzahlen ist eine ressourcenschonende Schnittstelle zur Sitzungszählung erforderlich.
- Die Browser-Übergabe führt zum normalen Dashboard; der Deep-Link zum Kustos im
  Onboarding-Modus folgt mit Phase 4.
