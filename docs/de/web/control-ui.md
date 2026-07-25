---
read_when:
    - Sie möchten das Gateway über einen Browser bedienen
    - Sie möchten Tailnet-Zugriff ohne SSH-Tunnel
sidebarTitle: Control UI
summary: Browserbasierte Bedienoberfläche für den Gateway (Chat, Aktivität, Nodes, Konfiguration)
title: Steuerungsoberfläche
x-i18n:
    generated_at: "2026-07-24T20:44:21Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 069bad7f3c8fce46759893e16d2dac86047c0929d6d866d25ce3b080204c1180
    source_path: web/control-ui.md
    workflow: 16
---

Die Control UI ist eine kleine **Vite + Lit**-Single-Page-App, die vom Gateway bereitgestellt wird:

- Standard: `http://<host>:18789/`
- optionales Präfix: `gateway.controlUi.basePath` festlegen (z. B. `/openclaw`)

Sie kommuniziert **direkt mit dem Gateway-WebSocket** am selben Port.

Während Sie eine laufende Sitzung beobachten, kann das Gateway das Utility-Modell dieses Agenten verwenden, um eine kompakte Statuszusammenfassung zu erstellen. Im Chat wird sie als einzeilige Statusanzeige dargestellt, die sich zu einer Karte mit Bewertung, Planfortschritt, Pull Requests und verstrichener Zeit erweitern lässt. Die Karte kann einmal automatisch aufgeklappt werden, wenn eine Ausführung feststeckt oder Eingaben benötigt; der Seitenchat `/btw` hat Vorrang vor der aufgeklappten Karte.

Die aufgeklappte Karte akzeptiert außerdem kurze Fragen zur Ausführung. Antworten verwenden ausschließlich die aktuelle Zusammenfassung des Beobachters und bereinigte, begrenzte Notizen, verbleiben für diese Sitzung im Browser und gelangen niemals in die Hauptausführung des Agenten oder unterbrechen sie. Wenn die Beobachtungen keine Antwort enthalten, gibt der Beobachter an, dass er sie nicht kennen kann.

Nach dem Eintreffen der ersten Zusammenfassung übernimmt sie anstelle der heuristischen Live-Aktivität den Untertitel der Seitenleiste für diese Ausführung. Eine abschließende Zusammenfassung über den erfolgreichen oder fehlgeschlagenen Abschluss bleibt sichtbar, solange die Sitzung ungelesen ist; anschließend zeigt die Zeile wieder ihren normalen Arbeitsuntertitel.

Die Sitzungsbeobachtung ist standardmäßig aktiviert. Unter **Einstellungen > Darstellung > Seitenleiste** können Sie sie Gateway-weit deaktivieren, das ermittelte kleine Modell und seine Herkunft prüfen, automatisches Routing auswählen, Utility-Aufgaben deaktivieren oder explizit ein `agents.defaults.utilityModel` auswählen. Die entsprechenden Konfigurationseinstellungen sind `gateway.controlUi.sessionObserver: false` und `agents.defaults.utilityModel: ""`.

## Schnell öffnen (lokal)

Wenn das Gateway auf demselben Computer ausgeführt wird, öffnen Sie [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (oder [http://localhost:18789/](http://localhost:18789/)).

Wenn die Seite nicht geladen werden kann, starten Sie zuerst das Gateway: `openclaw gateway`.

<Note>
Bei nativen Windows-LAN-Bindungen können die Windows-Firewall oder eine von der Organisation verwaltete Gruppenrichtlinie die angekündigte LAN-URL weiterhin blockieren, selbst wenn `127.0.0.1` auf dem Gateway-Host funktioniert. Führen Sie `openclaw gateway status --deep` auf dem Windows-Host aus; der Befehl meldet wahrscheinlich blockierte Ports, nicht übereinstimmende Profile und lokale Firewallregeln, die von Richtlinien möglicherweise ignoriert werden.
</Note>

Die Authentifizierung wird während des WebSocket-Handshakes bereitgestellt über:

- `connect.params.auth.token`
- `connect.params.auth.password`
- Tailscale-Serve-Identitätsheader, wenn `gateway.auth.allowTailscale: true`
- Identitätsheader eines vertrauenswürdigen Proxys, wenn `gateway.auth.mode: "trusted-proxy"`

Die Gateway-Authentifizierung erfolgt vor der Gerätekopplung. Eine direkte Loopback-Verbindung umgeht weder die Token- noch die Passwortauthentifizierung. Der Einstellungsbereich des Dashboards bewahrt ein Token für die aktuelle Sitzung des Browser-Tabs und die ausgewählte Gateway-URL auf; Passwörter werden nicht dauerhaft gespeichert. Nach der Kopplung kann der Browser bei späteren Verbindungen sein gespeichertes gerätespezifisches Token verwenden.

Das Onboarding konfiguriert normalerweise ein Gateway-Token für die Authentifizierung über ein gemeinsames Geheimnis. Wenn das Gateway im Token-Modus ohne konfiguriertes Token startet, erzeugt es stattdessen für diesen Prozess ein flüchtiges Laufzeit-Token. Das Laufzeit-Token wird nicht in die Konfiguration geschrieben, sodass `openclaw config get gateway.auth.token` es nicht abrufen kann und ein Loopback-Browser ohne dieses Token abgewiesen wird. Führen Sie `openclaw doctor --generate-gateway-token` aus, starten Sie das Gateway neu und fügen Sie anschließend das konfigurierte Token in den Einstellungen der Control UI ein. Stattdessen funktioniert die Passwortauthentifizierung, wenn `gateway.auth.mode` auf `"password"` gesetzt ist.

## Gerätekopplung (erste Verbindung)

Nachdem die Gateway-Authentifizierung erfolgreich war, erfordert die Verbindung über einen neuen Browser oder ein neues Gerät normalerweise eine **einmalige Kopplungsgenehmigung**, die als `disconnected (1008): pairing required` angezeigt wird.

<Warning>
Bei einem direkten Upgrade von einer Version, die die eingestellte
Notfalloption `gateway.controlUi.dangerouslyDisableDeviceAuth=true` verwendete,
hält OpenClaw den per Token, Passwort oder vertrauenswürdigem Proxy authentifizierten Zugriff auf die Control UI
für die ausschließliche Behebung der Kopplung verfügbar. Wenn der Browser einfaches HTTP verwendet und keine Geräteidentität erstellen kann,
öffnen Sie ihn zunächst erneut über HTTPS oder localhost. Klicken Sie dann im
Warnbanner auf **Diesen Browser absichern**. Das Gateway kehrt erst zur normalen Durchsetzung der Geräteauthentifizierung zurück,
nachdem ein signierter Browser ausdrücklich gekoppelt wurde; es erstellt oder genehmigt niemals eine
Identität für einen Browser ohne Geräteidentität. Der Übergang ist nicht verfügbar, wenn
bereits ein anderes Gerät eines Bedieners gekoppelt ist. Sowohl der Gateway-Start als auch
`openclaw doctor --fix` melden diese Migration ausdrücklich, anstatt
den alten Schlüssel stillschweigend zu verwerfen.
</Warning>

<Steps>
  <Step title="Ausstehende Anfragen auflisten">
    ```bash
    openclaw devices list
    ```
  </Step>
  <Step title="Anhand der Anfrage-ID genehmigen">
    ```bash
    openclaw devices approve <requestId>
    ```
  </Step>
</Steps>

Wenn der Browser die Kopplung mit geänderten Authentifizierungsdetails (Rolle/Bereiche/öffentlicher Schlüssel) erneut versucht, wird die vorherige ausstehende Anfrage ersetzt und ein neuer `requestId` erstellt; führen Sie `openclaw devices list` vor der Genehmigung erneut aus.

Wenn ein bereits gekoppelter Remote-Browser vom Lesezugriff auf Schreib- oder Administratorzugriff umgestellt wird, gilt dies als Erweiterung der Genehmigung und nicht als stillschweigende Neuverbindung: OpenClaw lässt die alte Genehmigung aktiv, blockiert die Neuverbindung mit erweiterten Rechten und fordert Sie auf, den neuen Satz von Bereichen ausdrücklich zu genehmigen. Eine geeignete direkte Loopback-Verbindung der Control UI kann die Erweiterung nach erfolgreicher Authentifizierung stillschweigend genehmigen.

Nach der Genehmigung wird das Gerät gespeichert und muss nicht erneut genehmigt werden, sofern Sie seine Genehmigung nicht mit `openclaw devices revoke --device <id> --role <role>` widerrufen. Informationen zur Token-Rotation, zum Widerruf und zum Genehmigungsablauf beim ersten Start von Paperclip / `openclaw_gateway` finden Sie unter [Geräte-CLI](/de/cli/devices).

<Note>
- Direkte lokale Verbindungen der Control UI von einem Loopback-TCP-Peer (`127.0.0.1` oder `::1`, typischerweise erreichbar als `localhost`) ohne Weiterleitungs-/Proxy-Header können die Gerätekopplung nur dann automatisch genehmigen, wenn die Gateway-Authentifizierung erfolgreich war und der Browser eine Geräteidentität vorlegt. Im Token-/Passwortmodus benötigt die erste Verbindung weiterhin das konfigurierte gemeinsame Geheimnis; diese automatische Genehmigung umgeht das Token nicht.
- Für eine direkte Loopback-Verbindung ist nur dann kein gemeinsames Geheimnis erforderlich, wenn `gateway.auth.mode: "none"` ausdrücklich konfiguriert ist. Dadurch wird die Gateway-Authentifizierung deaktiviert, und dies ist nicht die empfohlene Einrichtung der Control UI. In den Modi Tailscale Serve und vertrauenswürdiger Proxy kann auf das Einfügen eines gemeinsamen Geheimnisses nur verzichtet werden, wenn die jeweiligen Identitätsprüfungen erfolgreich sind.
- Tailscale Serve kann den Kopplungsschritt für Bedienersitzungen der Control UI überspringen, wenn `gateway.auth.allowTailscale: true`, die Tailscale-Identität verifiziert wurde und der Browser seine Geräteidentität vorlegt. Browser ohne Geräteidentität und Verbindungen mit Node-Rolle unterliegen weiterhin den normalen Geräteprüfungen.
- Direkte Tailnet-Bindungen und Browserverbindungen über das LAN erfordern weiterhin eine ausdrückliche Genehmigung. Browserprofile ohne Geräteidentität können die automatische Loopback-Genehmigung nicht verwenden.
- Jedes Browserprofil erzeugt eine eindeutige Geräte-ID. Ein Browserwechsel oder das Löschen der Browserdaten erfordert daher eine erneute Kopplung.

</Note>

## Mobilgerät koppeln

Ein bereits gekoppelter Administrator kann den QR-Code für die iOS-/Android-Verbindung erstellen, ohne ein Terminal zu öffnen:

<Steps>
  <Step title="Kopplung für Mobilgeräte öffnen">
    Wählen Sie **Geräte** aus und klicken Sie anschließend in der Karte **Geräte** auf **Mobilgerät koppeln**.
  </Step>
  <Step title="Telefon verbinden">
    Öffnen Sie in der mobilen OpenClaw-App **Einstellungen** → **Gateway** und scannen Sie den QR-Code. Alternativ können Sie den Einrichtungscode kopieren und einfügen.
  </Step>
  <Step title="Verbindung bestätigen">
    Die offizielle iOS-/Android-App stellt die Verbindung automatisch her. Wenn unter **Ausstehende Genehmigung** eine Anfrage angezeigt wird, prüfen Sie deren Rolle und Bereiche, bevor Sie sie genehmigen.
  </Step>
</Steps>

Zum Erstellen eines Einrichtungscodes ist `operator.admin` erforderlich; für Sitzungen ohne diese Berechtigung ist die Schaltfläche deaktiviert. Ein Einrichtungscode enthält kurzzeitig gültige Bootstrap-Anmeldedaten. Behandeln Sie daher den QR-Code und den kopierten Code wie ein Passwort, solange sie gültig sind. Für die Remote-Kopplung muss das Gateway zu `wss://` aufgelöst werden (beispielsweise über Tailscale Serve/Funnel); einfaches `ws://` ist auf Loopback- und private LAN-Adressen beschränkt. Vollständige Informationen zu Sicherheit und Ausweichverfahren finden Sie unter [Kopplung](/de/channels/pairing#pair-from-the-control-ui-recommended).

## Persönliche Identität (browserlokal)

Die Control UI unterstützt eine browserbezogene persönliche Identität (Anzeigename und Avatar), die ausgehenden Nachrichten zugeordnet wird, um die Urheberschaft in gemeinsam genutzten Sitzungen anzugeben. Sie befindet sich im Browserspeicher, ist auf das aktuelle Browserprofil beschränkt und wird weder mit anderen Geräten synchronisiert noch serverseitig über die normalen Metadaten zur Urheberschaft des Transkripts für von Ihnen gesendete Nachrichten hinaus dauerhaft gespeichert. Durch das Löschen der Websitedaten oder einen Browserwechsel wird sie zurückgesetzt.

Das Überschreiben des Assistenten-Avatars folgt demselben browserlokalen Muster: Hochgeladene Überschreibungen überlagern lokal die vom Gateway ermittelte Identität und werden niemals über `config.patch` übertragen. Das gemeinsame Konfigurationsfeld `ui.assistant.avatar` ist weiterhin für Nicht-UI-Clients verfügbar, die direkt in dieses Feld schreiben.

## Endpunkt für die Laufzeitkonfiguration

Die Control UI ruft ihre Laufzeiteinstellungen von `/control-ui-config.json` ab, relativ zum Basispfad der Control UI des Gateways aufgelöst (beispielsweise `/__openclaw__/control-ui-config.json` unter dem Basispfad `/__openclaw__/`). Dieser Endpunkt ist durch dieselbe Gateway-Authentifizierung wie die übrige HTTP-Oberfläche geschützt: Nicht authentifizierte Browser können ihn nicht abrufen, und ein erfolgreicher Abruf erfordert ein gültiges Gateway-Token/-Passwort, eine Tailscale-Serve-Identität oder eine Identität eines vertrauenswürdigen Proxys.

## Status des Gateway-Hosts

Öffnen Sie **Einstellungen → Allgemein**, um die Karte **Gateway-Host** mit dem Gateway-Computer, der LAN-Adresse, dem Betriebssystem, der Laufzeit, der Betriebszeit, der CPU-Auslastung, dem Arbeitsspeicher und dem Speicherplatz des Zustandsvolumes anzuzeigen. Solange die Karte sichtbar ist, wird sie alle 10 Sekunden über den Gateway-RPC `system.info` aktualisiert, der den Bereich `operator.read` erfordert. Bei älteren Gateways und Verbindungen ohne diesen Bereich wird die Karte nicht angezeigt.

## Sprachunterstützung

Die Control UI lokalisiert sich beim ersten Laden anhand der Spracheinstellung Ihres Browsers. Um diese später zu überschreiben, öffnen Sie **Einstellungen -> Allgemein -> Sprache** (die Auswahl befindet sich auf der Seite „Allgemein“, nicht unter „Darstellung“).

- Unterstützte Gebietsschemata: `en`, `ar`, `de`, `es`, `fa`, `fr`, `hi`, `id`, `it`, `ja-JP`, `ko`, `nl`, `pl`, `pt-BR`, `ru`, `th`, `tr`, `uk`, `vi`, `zh-CN`, `zh-TW`
- Nicht englische Übersetzungen werden im Browser verzögert geladen.
- Das ausgewählte Gebietsschema wird im Browserspeicher gespeichert und bei zukünftigen Besuchen wiederverwendet.
- Bei fehlenden Übersetzungsschlüsseln wird auf Englisch zurückgegriffen.

Dokumentationsübersetzungen werden für dieselben nicht englischen Gebietsschemata erzeugt, aber die integrierte Mintlify-Sprachauswahl der Dokumentationswebsite führt nur Gebietsschemacodes auf, die Mintlify akzeptiert. Die Dokumentation auf Thai (`th`) und Persisch (`fa`) wird weiterhin im Veröffentlichungs-Repository erzeugt; möglicherweise erscheint sie erst in dieser Auswahl, wenn Mintlify diese Codes unterstützt.

## Darstellungsthemen

Der Darstellungsbereich enthält die integrierten Themen Claw, Knot und Dash (Claw ist die Standardeinstellung) sowie einen browserlokalen Importplatz für tweakcn. Um ein Thema zu importieren, öffnen Sie den [tweakcn-Editor](https://tweakcn.com/editor/theme), wählen oder erstellen Sie ein Thema, klicken Sie auf **Share** und fügen Sie den kopierten Link unter „Darstellung“ ein. Der Import akzeptiert außerdem `https://tweakcn.com/r/themes/<id>`-Registry-URLs, Editor-URLs wie `https://tweakcn.com/editor/theme?theme=amethyst-haze`, relative `/themes/<id>`-Pfade, unformatierte Themen-IDs und Namen von Standardthemen wie `amethyst-haze`.

Importierte Themen werden ausschließlich im aktuellen Browserprofil gespeichert; sie werden nicht in die Gateway-Konfiguration geschrieben und nicht geräteübergreifend synchronisiert. Beim Ersetzen des importierten Themas wird dieser eine lokale Platz aktualisiert; wenn das importierte Thema aktiv war, wird nach dem Löschen wieder zu Claw gewechselt.

Unter „Darstellung“ gibt es außerdem die Einstellung „Textgröße“. Sie gilt für Chattext, Eingabetext, Werkzeugkarten und Chatseitenleisten und hält die Schriftgröße in Texteingabefeldern bei mindestens 16 px, damit Mobile Safari beim Fokussieren nicht automatisch zoomt.

Design, Designmodus, Textgröße, Sprache und Einstellungen für die Chatdarstellung werden über die Gateway-Konfiguration (`ui.prefs`) synchronisiert, sodass sie Ihnen geräteübergreifend folgen und Agenten sie über die Genehmigungsschranke ändern können — verbundene Clients übernehmen Änderungen über den `config.changed`-Hinweis des Gateways sofort. Jeder Browser hält für einen sofortigen Start eine lokale Kopie vor; Clients, die die Konfiguration nicht schreiben können (Betrachterbereich, offline), behalten Änderungen lokal auf dem Gerät. Siehe [Konfigurationsreferenz](/de/gateway/configuration-reference#ui).

## OpenClaw-Systempflege

Öffnen Sie **Einstellungen → OpenClaw fragen**, um mit dem Agenten für Systemeinrichtung und -reparatur zu sprechen. Außerhalb des Onboardings kann diese Seite pro Besuch höchstens einen ausblendbaren Ereignis-Chip anzeigen. Bei routinemäßigem Gateway-Datenverkehr bleibt sie stumm und reagiert nur auf Zustands-Snapshots, die einen deaktivierten Konfigurations-Neulader, eine Trennung oder Beeinträchtigung eines konfigurierten Kanals, eine fehlgeschlagene Kanalprüfung oder nicht verfügbare Kanal-Anmeldedaten melden. Ein neueres Ereignis ersetzt den ausstehenden Chip nur, wenn es schwerwiegender ist; durch Ausblenden oder Verwenden des Chips werden Ereignisaufforderungen für diesen Besuch unterdrückt. Beim Anklicken des Chips wird dessen Diagnosefrage als echte `openclaw.chat`-Nachricht gesendet, sodass das Transkript die Anfrage aufzeichnet und OpenClaw die Diagnose durchführt. Während des Onboardings werden diese Ereignis-Chips nie angezeigt.

## Plugins verwalten

Öffnen Sie **Plugins** in der Seitenleiste oder verwenden Sie `/settings/plugins` relativ zum
konfigurierten Basispfad der Control UI, um Plugins zu durchsuchen und zu verwalten, ohne
die Control UI zu verlassen. Beispielsweise verwendet ein Basispfad von `/openclaw`
den Pfad `/openclaw/settings/plugins`. Die Seite ist immer verfügbar, selbst wenn alle
optionalen Plugins deaktiviert sind.

Plugins ist eine Zentrale mit vier Registerkarten: **Installiert** und **Entdecken** verwalten Plugin-
Code unter `/settings/plugins`, **Skills** enthält die agentenspezifische Skills-Verwaltung unter
`/skills`, und **Workshop** enthält die Prüfung von Vorschlägen aus dem Skill Workshop unter
`/skills/workshop`. Jede Registerkarte behält ihre eigene URL, und die Seitenleiste zeigt für
alle Registerkarten den gemeinsamen Eintrag Plugins an.

Die Registerkarte **Installiert** zeigt den vollständigen lokalen Bestand nach Kategorien gruppiert und mit
Übersichtszahlen. Jede Zeile öffnet eine Detailansicht; ihr Überlaufmenü (`…`)
aktiviert oder deaktiviert das Plugin und bietet für extern installierte Plugins **Entfernen** an.
Es listet außerdem konfigurierte [MCP-Server](/de/cli/mcp) auf und unterstützt deren Hinzufügen, Deaktivieren
und Entfernen direkt in der Ansicht. Dieselben Server-Steuerelemente sind unter **Einstellungen → MCP** verfügbar.
Die Registerkarte **Entdecken** ist der Store: vorgestellte, in OpenClaw enthaltene Plugins,
offizielle externe Plugins und MCP-Konnektoren für beliebte Dienste mit nur einem Klick.
Eine Eingabe in das Suchfeld durchsucht
[ClawHub](https://clawhub.ai/plugins) direkt und fügt einen Abschnitt **Von ClawHub**
mit Downloadzahlen und Kennzeichnungen zur Quellverifizierung hinzu. Deep Links können
mit `/settings/plugins?tab=discover` direkt auf den Store verweisen.

Die Registerkarte **Skills** enthält den Skills-Statusbericht, Schalter zum Aktivieren und Deaktivieren, die Eingabe von API-
Schlüsseln und die direkte ClawHub-Skills-Suche, jeweils beschränkt auf den ausgewählten Agenten. Die
Registerkarte **Workshop** enthält das Skill-Workshop-Board und den heutigen Prüfablauf für
[Skills-Vorschläge](/de/tools/skill-workshop). **Ideen für Skills finden** prüft ein begrenztes
Fenster umfangreicher Sitzungen von der neuesten bis zur ältesten und hinterlässt alle Ergebnisse als
ausstehende Vorschläge. Das Panel zeigt die kumulative Abdeckung; **Frühere Arbeit durchsuchen**
setzt die Suche ab dem gespeicherten Cursor fort und wird zu **Neue Arbeit durchsuchen**, nachdem der ältere
Verlauf vollständig verarbeitet wurde. Die manuelle Verlaufsprüfung funktioniert auch bei deaktiviertem autonomen Selbstlernen
und verwendet das konfigurierte Modell des ausgewählten Agenten.

Enthaltene Plugins sind bereits auf dem Gateway vorhanden und zeigen **Aktivieren** oder
**Deaktivieren** statt **Installieren** an. Workboard ist beispielsweise in
OpenClaw enthalten, aber standardmäßig deaktiviert, weshalb die Aktion **Aktivieren** lautet. Gebündelte Plugins
können nicht entfernt, sondern nur deaktiviert werden.

Das Lesen des Katalogs und die Suche in ClawHub erfordern `operator.read`. Das Installieren,
Aktivieren, Deaktivieren oder Entfernen eines Plugins sowie das Ändern von MCP-Servern erfordern
`operator.admin`; für schreibgeschützte Operatoren bleiben diese Aktionen deaktiviert.

ClawHub-Installationen werden über das Gateway ausgeführt und unterliegen denselben Prüfungen für Vertrauen, Integrität
und Plugin-Installationsrichtlinien wie andere durch das Gateway vermittelte Installationen. Das Installieren
oder Entfernen von Plugin-Code erfordert einen Neustart des Gateways. Das Aktivieren oder Deaktivieren eines
installierten Plugins kann ohne Neustart angewendet werden, wenn das Plugin und die aktuelle
Gateway-Laufzeit dies unterstützen; andernfalls meldet die UI, dass ein Neustart
erforderlich ist. OAuth-gestützte MCP-Konnektoren benötigen nach dem Hinzufügen einmalig
`openclaw mcp login <name>` über die CLI.

Die Seite konzentriert sich bewusst auf Bestand, Entdeckung, Installation, Aktivierung
und Entfernung. Verwenden Sie [`openclaw plugins`](/de/cli/plugins) für beliebige npm-, Git- oder
lokale Pfadquellen, Aktualisierungen und die erweiterte Plugin-Konfiguration.

## Apps und Erweiterungen

Öffnen Sie **Apps** über das Menü **Mehr** in der Seitenleiste, die Befehlspalette oder das
Agentenmenü der Seitenleiste (**Apps herunterladen**), oder verwenden Sie `/apps` relativ zum
konfigurierten Basispfad der Control UI. Die Seite bündelt Installationslinks für alle
OpenClaw-Begleitoberflächen: die Apps für [iOS](/de/platforms/ios) und
[Android](/de/platforms/android), die darin enthaltenen Begleit-Apps für Apple Watch und Wear OS,
die Desktop-Apps für [macOS](/de/platforms/macos), [Windows](/de/platforms/windows)
und [Linux](/de/platforms/linux), die
[Chrome-Erweiterung](/de/tools/chrome-extension), die integrierte Plugins-Zentrale mit
[ClawHub](https://clawhub.ai) sowie die Discord-Community und die Dokumentation.

## Navigation in der Seitenleiste

Die Seitenleiste ordnet alles um den Agenten herum an. Die Identitätszeile oben zeigt den aktiven Agenten; darunter beginnt der Abschnitt **Seiten** mit **Startseite** — der fortlaufenden Hauptsitzung des Agenten, versehen mit einer Kennzeichnung für ihren ungelesenen oder laufenden Status — gefolgt von den angehefteten Zielen (standardmäßig **Automatisierungen** und **Plugins**). Das Anpassungssteuerelement in der Kopfzeile von Seiten öffnet ein Menü mit allen weiteren Zielen, darunter **Nutzung** und von Plugins bereitgestellte Registerkarten, sowie **Angeheftete Einträge bearbeiten**; ein Rechtsklick auf den Navigationsbereich öffnet den Editor für angeheftete Einträge direkt. Die darunterliegende Sitzungsliste ist in Bereiche unterteilt: **Threads** für die Chatsitzungen des Agenten (die Hauptsitzung bleibt hinter der Startseite; von ihr gestartete Sitzungen erscheinen hier als Threads der obersten Ebene, und benannte Threads werden ohne Typpräfix angezeigt), **Gruppen** für Gruppen- und Raumunterhaltungen und **Programmierung** für Sitzungen, die an einen verwalteten Worktree oder Ausführungs-Node gebunden sind (Zeilen zeigen eine `repo ⎇ branch`-Zeile sowie den Node-Host), ACP-gestützte Harness-Sitzungen und die CLI-Kataloge von Codex und Claude. Programmierung ist beim ersten Start eingeklappt und merkt sich Ihre Auswahl; die eingeklappte Kopfzeile behält die tatsächliche Anzahl bei und zeigt einen Laufindikator, während enthaltene Sitzungen arbeiten. Benutzerdefinierte Gruppen (die Sitzungs-`category`) und **Angeheftet**-Zeilen stehen über Threads, und die Zuordnung einer Sitzung zu einer benutzerdefinierten Gruppe hat immer Vorrang vor der automatischen Bereichsklassifizierung. Die Kopfzeile von Threads enthält die Sortiersteuerung (Erstellt oder Zuletzt aktualisiert, Gruppieren nach sowie einen gespeicherten **Status**-Filter für Aktiv, Archiviert oder Alle) und das **+**, das die Seite Neue Sitzung öffnet. Archivierte Zeilen bleiben abgeblendet und mit einem Archivsymbol versehen in der Liste; sie tragen weder zum ungelesenen Status noch zum Aufmerksamkeitsstatus bei und bleiben von der Heraufstufung innerhalb der Abstammung ausgeschlossen. Beim Öffnen einer Sitzung wird die Auswahlmarkierung verschoben, ohne die Zeilen neu zu ordnen. Übergeordnete Sitzungen mit kürzlich ausgeführten untergeordneten Sitzungen zeigen ein Aufklappsymbol und die Anzahl der untergeordneten Sitzungen; klappen Sie den Eintrag auf, um verschachtelte untergeordnete Sitzungen, ihren laufenden oder abgeschlossenen Status und die Laufzeit zu prüfen, ohne die Seitenleiste zu verlassen. Bei Auswahl einer untergeordneten Sitzung wird deren Chat geöffnet und der Pfad ihrer übergeordneten Sitzungen automatisch eingeblendet. Untergeordnete Zeilen bleiben von der Gruppierung auf Stammebene, dem Anheften, Ziehen, der Mehrfachauswahl und der Seitennummerierung ausgeschlossen; eingeklappte Bereiche verbrauchen das Budget der sichtbaren Seite nicht. Sitzungen mit neuen Aktivitäten seit dem letzten Lesen zeigen einen Punkt für ungelesene Inhalte, und beim Öffnen werden sie als gelesen markiert. Ein Agent kann außerdem eine kurze, ablaufende Statuszeile veröffentlichen und optional mit einem ausgewählten bernsteinfarbenen Symbol um Aufmerksamkeit bitten; diese Angabe wird gelöscht, wenn Sie die Sitzung öffnen, die nächste Nachricht senden, sie ausdrücklich löschen oder ihre TTL abläuft. Lebenszykluszustände von Cloud-Workern verwenden eine Globus-Kennzeichnung; lokale und zurückgeholte Sitzungen zeigen keine Platzierungskennzeichnung, da die lokale Ausführung die Standardeinstellung ist. Jede Stammsitzungszeile besitzt ein Kontextmenü (Dreipunktschaltfläche oder Rechtsklick) mit Anheften/Lösen, Als ungelesen/gelesen markieren, Umbenennen, Abspalten, In Gruppe verschieben (einschließlich Neue Gruppe und Aus Gruppe entfernen), Archivieren oder Dearchivieren und Löschen; auf Touch-Geräten bleiben die direkten Steuerelemente zum Anheften und für das Menü sichtbar. Cmd-/Strg-Klick schaltet Stammzeilen in eine Mehrfachauswahl, und Umschalt-Klick erweitert sie über die sichtbare Reihenfolge; beim Öffnen des Menüs einer ausgewählten Zeile werden anschließend Stapelaktionen angeboten (N als ungelesen/gelesen markieren, N in Gruppe verschieben, N archivieren, N löschen), die auf alle ausgewählten Sitzungen angewendet werden, wobei für das Löschen im Stapel eine einzige Bestätigung erforderlich ist. Ziehen Sie eine Stammsitzung auf **Angeheftet**, um sie anzuheften, oder auf eine benutzerdefinierte Gruppe, um sie zu verschieben. Kopfzeilen benutzerdefinierter Gruppen können eingeklappt, ausgeklappt oder zum Ändern ihrer Reihenfolge gezogen werden; Gruppennamen und ihre Reihenfolge werden im Gateway (`sessions.groups.*`) gespeichert, sodass sie Ihnen browserübergreifend folgen, während der eingeklappte Zustand im Browserprofil verbleibt. Gruppenkopfzeilen besitzen außerdem ein Menü (Dreipunktschaltfläche oder Rechtsklick) mit Gruppe umbenennen, Neue Gruppe und Gruppe löschen; beim Umbenennen oder Löschen einer Gruppe werden alle zugehörigen Sitzungen serverseitig aktualisiert, einschließlich archivierter Sitzungen, und beim Löschen einer Gruppe bleiben deren Sitzungen erhalten und werden zurück zu Threads verschoben.

## Seite „Neue Sitzung“

Das **+** in der Kopfzeile der Sitzungsliste in der Seitenleiste öffnet unter `/new` einen ganzseitigen Entwurf: Erst mit dem Senden der ersten Nachricht wird etwas erstellt. Eine einheitliche Auswahl **Ort** legt den Arbeitsordner und für Administratoren das Ausführungsziel fest: **Gateway · lokal**, einen gekoppelten Node, der `system.run` bereitstellt, oder ein verfügbares Cloud-Profil. Der Ordner ist standardmäßig der Arbeitsbereich des Agenten; ein anderer absoluter Gateway-Pfad erfordert `operator.admin`, kann jedoch direkt ausgeführt werden, ohne ein Git-Checkout zu sein. Wenn der ausgewählte Gateway-Ordner ein Git-Checkout ist, bietet dieselbe Auswahl eine optionale **Worktree**-Isolation mit einer durch `worktrees.branches` gestützten Auswahl des Basis-Branches (kein Abruf) und einem optionalen Worktree-Namen (der Branch wird zu `openclaw/<name>`). Cloud-Worker erfordern diesen verwalteten Worktree-Pfad; gekoppelte Nodes stellen ihn nie bereit. In der Fußzeile des Eingabefelds werden das Modell und die Reasoning-Stufe der neuen Sitzung ausgewählt. Der Schalter **Inkognito** erstellt einen ausschließlich webbasierten Thread, dessen Sitzungseintrag, Transkript und Compaction-Zustand bis zum Neustart des Gateways im Arbeitsspeicher verbleiben; OpenClaw überspringt außerdem die automatische Speicherung des Speichers. Der Agent behält seine normalen Werkzeuge, sodass eine ausdrückliche Speicheranforderung oder ein werkzeuggestützter Dateischreibvorgang weiterhin Daten dauerhaft speichern kann. Der Modell-Provider verarbeitet die Nachrichten weiterhin, und inhaltsfreie Audit-Metadaten werden weiterhin aufgezeichnet. Bei Cloud-Starts werden die Modell- und Reasoning-Auswahl gespeichert, bevor die Sitzung an den Worker übermittelt wird.

Auf Gateways mit mehreren Benutzern können nur Verbindungen mit Administratorbereich Inkognito-Threads erstellen oder anzeigen, und andere Sitzungen können weder über Sitzungstools des Agenten noch über die Transkriptsuche darauf zugreifen. Inkognito schützt vor Speicherung und anderen durch das Gateway vermittelten Benutzern, nicht vor dem Eigentümer des Gateways oder dem Prozessoperator, die laufende Sitzungen jederzeit beobachten können.

**Ordner durchsuchen** öffnet den integrierten Verzeichnisbrowser der Ortsauswahl, der auf der ausschließlich Administratoren vorbehaltenen Methode `fs.listDir` basiert und auf das ausgewählte Gateway oder den ausgewählten Node beschränkt ist. Das Gateway und durchsuchungsfähige Nodes listen ihr Dateisystem auf; ein ausführungsfähiger Node ohne `fs.listDir` akzeptiert weiterhin einen eingegebenen absoluten Pfad. Zuletzt verwendete Orte können einen Ordner und den zugehörigen Node gemeinsam wiederherstellen, ohne Pfade zwischen Hosts zu übertragen. Beim Absenden wird `sessions.create` mit der ersten Nachricht aufgerufen, sodass der Lauf innerhalb desselben Roundtrips beginnt und die UI zum Chat der neuen Sitzung wechselt. Wenn das Gateway die Sitzung erstellt, aber diesen ersten Sendevorgang ablehnt, bleiben die Eingabeaufforderung und der Fehler auch nach einem Neuladen im Chat erhalten; **Erneut versuchen** sendet sie über die bereits erstellte Sitzung, statt eine weitere zu erstellen.

Innerhalb der **Einstellungen** enthält die eigene Seitenleiste **OpenClaw fragen** und beginnt mit einem Feld **Einstellungen durchsuchen**, mit dem Einstellungsabschnitte schnell gefunden werden können.

Im Desktop-Web befindet sich oben links im Inhaltsbereich eine feste Steuerungsgruppe – das Web-Gegenstück zur macOS-Titelleistenleiste – mit dem Umschalter zum Einklappen der Seitenleiste (⌘B) und der Suchschaltfläche der Befehlspalette (⌘K). Durch Klicken auf die Agentenidentitätszeile oben in der Seitenleiste wird das Agentenmenü geöffnet; **Startseite** öffnet die Hauptsitzung. Wenn Handlungsbedarf besteht – bei fehlgeschlagenen oder überfälligen Cron-Jobs oder einer bald ablaufenden beziehungsweise abgelaufenen Modellauthentifizierung –, erscheinen kompakte Hinweis-Chips über der Fußzeile der Seitenleiste, die beim Anklicken zur jeweils zuständigen Seite führen. Die Identitätszeile zeigt den Avatar des Agenten (Identitätsbild oder Emoji), seinen Namen, den Verbindungsstatuspunkt und einen live aktualisierten Untertitel. Das agentenspezifische Menü enthält den eingebetteten Agentenumschalter (bei Konfigurationen mit mehreren Agenten), **Neuer Agent**, „Was kann dieser Agent?“ und **Agenteneinstellungen**. Bei mehr als zehn Agenten erhalten die Listen ein Filterfeld und zeigen angeheftete Agenten zuerst an; Agenten können auf der Einstellungsseite „Agenten“ angeheftet oder gelöst werden, wobei die angeheftete Auswahl im Browserprofil gespeichert wird. Durch Auswahl eines Agenten werden Chat sowie Nutzung, Automatisierungen, Aufgaben, Workboard und Sitzungen auf diesen Agenten beschränkt. Jede entsprechend beschränkte Seite bietet ein Steuerelement **Agent** mit **Alle Agenten** als Ausweichoption; dadurch wird der Geltungsbereich der gemeinsamen Seite erweitert, ohne den konkreten Chat-Agenten zu ändern, während direkte Sitzungslinks weiterhin ihr jeweiliges Ziel öffnen. Die Einstellungsseite „Agenten“ behält ihre eigene Auswahl `?agent=` bei und folgt nicht dem gemeinsamen Seitengeltungsbereich. Die Fußzeile besteht aus einer Identitätskarte über die gesamte Breite, die auch offline verfügbar bleibt und unter dem zuletzt bekannten Kontonamen **Verbindung wird wiederhergestellt…** anzeigt. Sie öffnet das App-/Kontomenü, in dem auf die Profilidentitäts-Kopfzeile **Einstellungen**, **Nutzung**, die Kopplung mit Mobilgeräten, **Apps herunterladen**, **Hilfe** (Hilfe, Discord, Dokumentation und Änderungsprotokoll), bei Bedarf eine Offline-Aktion zum erneuten Versuch, der Versions-/Build-Chip und der Umschalter für den Farbmodus folgen. Der Build-Chip öffnet die Infoseite. Wenn das Gateway aus einem Quellcode-Checkout auf einem anderen Branch als `main` ausgeführt wird, zeigt die Fußzeile zusätzlich den Namen dieses Branches in Rot an, sodass ein Nicht-Release-Gateway auf einen Blick erkennbar ist (bei Release-Installationen wird er nie angezeigt). Umschalt-Befehl-Komma auf Apple-Plattformen beziehungsweise Strg-Umschalt-Komma auf anderen Plattformen öffnet **Einstellungen**, ohne den einfachen Browserkurzbefehl Befehl-Komma zu überschreiben. Beim Einklappen der Seitenleiste (⌘B oder über den Umschalter der Steuerungsgruppe) wird sie vollständig ausgeblendet, sodass ein Arbeitsbereich über die gesamte Breite entsteht; im eingeklappten Zustand behält die Steuerungsgruppe oben links den Umschalter zum Ausklappen und die Suche bei und erhält zusätzlich eine Schaltfläche für einen neuen Thread – entsprechend den Bedienelementen, die die macOS-App nativ in ihrer Titelleiste bereitstellt. Die Seitenleiste ist auf dem Desktop das einzige Navigationselement; eine obere Leiste gibt es nicht. Bei schmalen Ansichtsbereichen wird die Seitenleiste durch ein einblendbares Seitenpanel hinter einer kompakten Kopfzeile ersetzt, die den Panel-Umschalter, die Marke und die Suche der Befehlspalette enthält; auf Smartphones übernimmt der Chat diese Navigationszeile in seine Titelleiste, wobei sich die Menü- und Suchbedienelemente neben dem Sitzungstitel befinden. In der macOS-App integriert die separate Kopfzeile den Freiraum der Titelleiste in eine einzige kompakte Leiste neben den Fenstersteuerelementen. Die Navigation verwendet den regulären Browserverlauf, sodass sie mit den Vorwärts-/Zurück-Schaltflächen des Browsers durchlaufen werden kann; die macOS-App ergänzt neben den Fenstersteuerelementen einen nativen Umschalter für die Seitenleiste sowie Trackpad-Wischgesten. Bei ausgeklappter Seitenleiste befinden sich Vorwärts-/Zurück-Schaltflächen an ihrem rechten Rand, bei eingeklappter Seitenleiste native Schaltflächen für die Suche (Befehlspalette) und eine neue Sitzung.

Ausstehende Genehmigungen erzeugen ebenfalls einen Hinweis-Chip über der Fußzeile der Seitenleiste;
wählen Sie ihn aus, um die zuständige Seite „Genehmigungen“ zu öffnen.

## Was es kann (heute)

<AccordionGroup>
  <Accordion title="Chat und Sprechen">
    - Chatten Sie über Gateway WS mit dem Modell (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`). Bei archivierten Sitzungen bleibt das Eingabefeld deaktiviert und ein Banner mit der Aktion **Dearchivieren** wird angezeigt, bevor die Unterhaltung fortgesetzt werden kann.
    - Beim Aktualisieren des Chatverlaufs wird ein begrenztes Fenster der letzten Nachrichten mit Textobergrenzen pro Nachricht angefordert, sodass große Sitzungen den Browser nicht dazu zwingen, die Nutzlast eines vollständigen Transkripts darzustellen, bevor der Chat verwendet werden kann.
    - Wenn der Mauszeiger über einem öffentlichen GitHub-Issue- oder Pull-Request-Link verweilt oder dieser per Tastatur fokussiert wird, werden Status, Titel, Autor, letzte Aktivitäten, Kommentare und Änderungsstatistiken angezeigt. Das verbundene Gateway ruft öffentliche Metadaten ab und speichert sie im Cache, ohne das Linkziel zu ändern – auch wenn die Benutzeroberfläche ein entferntes Gateway verwendet. Das Gateway verwendet `GH_TOKEN` oder `GITHUB_TOKEN`, sofern verfügbar, nachdem bestätigt wurde, dass das Repository öffentlich ist; andernfalls verwendet es die anonyme GitHub-API mit einer längeren Cache-Dauer.
    - Sprechen Sie über Echtzeitsitzungen im Browser. OpenAI verwendet direktes WebRTC, Google Live verwendet ein eingeschränktes Browser-Token zur einmaligen Nutzung über WebSocket und ausschließlich backendseitige Echtzeit-Sprach-Plugins verwenden den Gateway-Relay-Transport. Videofähige Browsersitzungen können in den Einstellungen eine lokale Gerätekamera auswählen oder über die Live-Vorschau zwischen Kameras wechseln; der Browser erfasst JPEG-Einzelbilder für den Echtzeit-Provider, ohne Kameravideo durch das Gateway zu streamen. Vom Client verwaltete Provider-Sitzungen beginnen mit `talk.client.create`; Gateway-Relay-Sitzungen beginnen mit `talk.session.create`. Das Relay bewahrt die Provider-Anmeldedaten auf dem Gateway auf, während der Browser Mikrofon-PCM über `talk.session.appendAudio` streamt, `openclaw_agent_consult`-Tool-Aufrufe des Providers über `talk.client.toolCall` an die Gateway-Richtlinien und das größere konfigurierte OpenClaw-Modell weiterleitet und die Sprachsteuerung aktiver Ausführungen über `talk.client.steer` oder `talk.session.steer` überträgt.
    - Streamen Sie Tool-Aufrufe und Karten mit Live-Tool-Ausgaben im Chat (Agentenereignisse). Tool-Aktivitäten werden als nach Art differenzierte Zeilen dargestellt: Shell-Befehle zeigen den syntaxhervorgehobenen Befehl mit einer Ausgabe im Terminalstil; unterstützte Bearbeitungs- und Schreibaufrufe zeigen begrenzte eingebettete Diffs, sofern verfügbar Zeilennummern und `+added -removed`-Statistiken; aufeinanderfolgende Aufrufe werden zu einer Zusammenfassung wie „13 Befehle ausgeführt, 6 Dateien gelesen, 9 Dateien bearbeitet“ zusammengefasst. Während eine Ausführung aktiv ist, gibt der neueste laufende Aufruf der Gruppenkopfzeile ihren Namen. Klappen Sie eine Zeile aus, um die verbleibenden Argumente und die Rohausgabe zu prüfen.
    - Optionale KI-Zwecktitel für komplexe Tool-Aufrufe (lange Shell-Befehle, Plugin-Tools mit vielen Argumenten), aktiviert mit `gateway.controlUi.toolTitles: true` (standardmäßig deaktiviert). Die Titel stammen aus der gebündelten Methode `chat.toolTitles` über das standardmäßige Routing für Hilfsmodelle – entweder ein explizites `utilityModel` (vom Betreiber ausgewählter Provider, wie bei anderen Hilfsaufgaben) oder andernfalls der deklarierte Standard des Sitzungs-Providers für kleine Modelle – und werden auf Gateway-Seite pro Agent im Cache gespeichert. Wenn die optionale Funktion deaktiviert ist oder kein kostengünstiges Modell verwendet werden kann, behalten die Zeilen ihre deterministischen Beschriftungen und es erfolgt kein Modellaufruf.
    - Starten oder verwerfen Sie kurzlebige, vom Modell vorgeschlagene Folgeaufgaben; angenommene Vorschläge öffnen eine neue Sitzung in einem verwalteten Worktree mit dem vorgeschlagenen Prompt.
    - Aktivitätsregisterkarte mit browserlokalen, vorrangig geschwärzten Zusammenfassungen der Live-Tool-Aktivitäten aus der bestehenden Bereitstellung von `session.tool`- bzw. Tool-Ereignissen.

  </Accordion>
  <Accordion title="Kanäle, Sitzungen, Speicher">
    - Kanäle: Status integrierter sowie gebündelter/externer Plugin-Kanäle, QR-Anmeldung und kanalbezogene Konfiguration (`channels.status`, `web.login.*`, `config.patch`).
    - Bei Aktualisierungen der Kanalprüfung bleibt der vorherige Snapshot sichtbar, während langsame Provider-Prüfungen abgeschlossen werden; partielle Snapshots werden gekennzeichnet, wenn eine Prüfung oder ein Audit das vorgesehene Zeitbudget der Benutzeroberfläche überschreitet.
    - Threads (eine Arbeitsbereichsseite unter `/sessions`, mit einer danebenliegenden Registerkarte **Worktrees**): standardmäßig Sitzungen konfigurierter Agenten auflisten, häufig verwendete Sitzungen anheften, sie umbenennen, inaktive Sitzungen archivieren oder wiederherstellen, bei veralteten Sitzungsschlüsseln nicht mehr konfigurierter Agenten auf eine Alternative zurückgreifen und sitzungsbezogene Überschreibungen für Modell, Denkmodus, Schnellmodus, Ausführlichkeit, Ablaufverfolgung und Schlussfolgerung anwenden (`sessions.list`, `sessions.patch`). Ein dreistufiger Filter **Aktiv / Archiviert / Alle** steuert sowohl diese Seite als auch die Seitenleiste; „Alle“ stellt archivierte Zeilen abgeblendet dar und kennzeichnet sie ausdrücklich. Archivierte Sitzungen behalten ihre Transkripte, werden nie automatisch bereinigt und bleiben zurückgestellt, bis sie ausdrücklich dearchiviert oder gelöscht werden. Zeilen zeigen bei aktiven Sitzungen mit Aktivitäten seit dem letzten Lesen einen Ungelesen-Punkt sowie Aktionen zum Markieren als ungelesen/gelesen an (`sessions.patch { unread }`) und bieten eine Aktion zum Abzweigen, die das Transkript in eine neue Sitzung verzweigt (`sessions.create { parentSessionKey, fork: true }`). Übersichtskacheln über der Tabelle fassen die geladene Liste zusammen (Anzahl der Sitzungen, aktive Ausführungen, ungelesene Sitzungen, Gesamtzahl der Token und, sofern verfügbar, Anzahl der archivierten Sitzungen). Jede Zeile enthält ein Symbol für ihre Art mit einem Punkt für eine aktive Ausführung, der Status wird als einfacher Punkt mit Beschriftung dargestellt und die Spalte „Token“ zeigt eine Auslastungsanzeige für das Kontextfenster, wenn die Sitzung Token- und Kontextgrößen meldet. Aktionen zur Zeilenverwaltung befinden sich in einem zeilenbezogenen Menü (Dreipunkt-Schaltfläche oder Rechtsklick), das dem Sitzungsmenü der Seitenleiste entspricht. Das Zeilenpanel zeigt neben den übrigen Sitzungsdetails auch die Agentenlaufzeitumgebung und die Ausführungsdauer an.
    - Native Seitenleistenkataloge von Claude und Codex streamen jeweils einen Host und werden anschließend nach Änderungen der Node-Konnektivität, beim Fokussieren der Seite und während der Sichtbarkeit höchstens alle 30 Sekunden abgeglichen. Katalogänderungen lösen einen schnelleren Nachlauf aus, sodass in den nativen Tools erstellte Sitzungen ohne Neuladen der Control UI erscheinen. Zeilen von Claude Desktop behalten außerdem ihre lokale benutzerdefinierte Gruppenbeschriftung bei, sofern vorhanden; OpenClaw liest diese Zuordnung aus dem lokalen Speicher von Desktop und schreibt niemals dorthin.
    - Sitzungsgruppierung: Ein Steuerelement „Gruppieren nach“ ordnet die Sitzungstabelle in Abschnitte nach benutzerdefinierten Gruppen, Kanal, Art, Agent oder Datum. Benutzerdefinierte Gruppen bleiben pro Sitzung über `sessions.patch` (`category`) erhalten, sodass auch Sitzungen kategorisiert werden können, die über Nachrichtenkanäle (Discord, Telegram, WhatsApp, ...) gestartet wurden; weisen Sie Gruppen zu, indem Sie Zeilen auf einen Abschnitt ziehen oder die Gruppenauswahl der jeweiligen Zeile verwenden, und erstellen Sie Gruppen mit der Aktion „Neue Gruppe“.
    - Speicher (eine auf den ausgewählten Agenten beschränkte Registerkarte auf der Seite „Agenten“): Dreaming-Status, Umschalter zum Aktivieren/Deaktivieren und Leser für das Traumtagebuch (`doctor.memory.status`, `doctor.memory.dreamDiary`, `config.patch`).
    - Speicher importieren (`/memory-import`, erreichbar über die Registerkarte „Speicher“ auf der Seite „Agenten“): lokale Dateien des automatischen Speichers von Claude Code, des konsolidierten Speichers von Codex oder des Hermes-Speichers in der Vorschau anzeigen und in den Arbeitsbereich des ausgewählten Agenten kopieren (`migrations.memory.plan`, `migrations.memory.apply`).
    - Speicherangebot beim Onboarding: Wenn die Control UI im Onboarding-Modus geöffnet wird (`?onboarding=1`, von der Linux-Begleit-App nach ihrer Erstinstallation verwendet), bietet ein einseitiger Dialog an, erkannte Speicher mit demselben Plan-/Anwendungsablauf zu importieren; beim Überspringen bleibt die Einstellungsseite als späterer Einstiegspunkt verfügbar.

  </Accordion>
  <Accordion title="Cron, Aufgaben, Plugins, Skills, Geräte, Ausführungsgenehmigungen">
    - Automatisierungen (Cron-Jobs): Statuskarten (Anzahl der Automatisierungen, Anzahl der fehlgeschlagenen Automatisierungen, Scheduler-Status, nächstes Aufwachen) über einer Umschaltung zwischen den Tabs „Automatisierungen“ und „Ausführungsverlauf“; der Tab „Automatisierungen“ listet Jobs in einer filterbaren Tabelle auf (Alle/Aktiv/Pausiert, Suche, Zeitplan- und Letzte-Ausführung-Filter, Aktionsmenü pro Zeile) und zeigt darunter Vorschläge für den Einstieg, während der Tab „Ausführungsverlauf“ die letzten Ausführungen aller Automatisierungen anzeigt (`cron.*`).
    - Aufgaben: fortlaufendes Verzeichnis aktiver und kürzlich ausgeführter Hintergrundaufgaben mit verknüpften Sitzungen und Abbruchmöglichkeit (`tasks.*`). Die Leiste „Hintergrundaufgaben“ des Chats gruppiert laufende und abgeschlossene Arbeiten; wählen Sie eine Zeile aus, um den größenbegrenzten Prompt und die Ausgabe oder Fehlerzusammenfassung zu prüfen.
    - Plugins: Durchsuchen Sie das installierte Inventar und den kuratierten Store, durchsuchen Sie ClawHub, installieren und entfernen Sie Plugin-Code und aktivieren oder deaktivieren Sie installierte Plugins (`plugins.*`); in den Zeilen der MCP-Server wird `mcp.servers` über die Konfigurationsmethoden bearbeitet.
    - Skills: Status, Aktivierung/Deaktivierung, Installation und Aktualisierung von API-Schlüsseln (`skills.*`).
    - Geräte: Ein gemeinsames Inventar führt Datensätze gekoppelter Geräte, den Node-Katalog und die Live-Anwesenheit zusammen (`device.pair.list`, `node.list`, `system-presence`). Der Gateway-Host ist an erster Stelle angeheftet; gekoppelte Clients zeigen Verbindungsstatus, Rollen, Token, Funktionen und Befehle an. Doppelte Kopplungen werden zu einer erweiterbaren Gruppe zusammengefasst, und **N veraltete bereinigen** entfernt gesammelt und nach Administratorbestätigung Offline-Duplikate, die automatisch genehmigt wurden (stille lokale, vertrauenswürdige CIDR- oder SSH-verifizierte Kopplung) oder aus der Zeit vor der Erfassung der Genehmigungsherkunft stammen. Einträge können entfernt werden (`node.pair.remove`, `device.pair.remove`), Gerätekopplungen und erneute Node-Genehmigungen werden direkt verarbeitet (`device.pair.*`, `node.pair.approve`/`reject`), und Codes für die mobile Einrichtung werden über dieselbe Karte erstellt.
    - Ausführungsgenehmigungen: Bearbeiten Sie Gateway- oder Node-Zulassungslisten und die Abfragerichtlinie für `exec host=gateway/node` (`exec.approvals.*`).

  </Accordion>
  <Accordion title="Konfiguration">
    - `~/.openclaw/openclaw.json` anzeigen/bearbeiten (`config.get`, `config.set`).
    - Die Einstellungsnavigation beginnt mit „OpenClaw fragen“ und gruppiert die Seiten anschließend nach Aufmerksamkeit: Allgemein, Erscheinungsbild und Benachrichtigungen oben; Verbindungen (Verbindung, Kanäle, Kommunikation, Geräte); Agenten und Tools (Agenten, KI und Agenten, Modell-Provider, MCP, Automatisierung, Labs); Datenschutz und Sicherheit (Sicherheit, Genehmigungen); sowie System (Infrastruktur, Erweitert, Debugging, Protokolle, Info). „Allgemein“ ist eine schlanke Übersichtsseite mit Modellstandards, Sprache und Gateway-Hoststatistiken; jede andere Einstellung befindet sich auf genau einer Seite.
    - Datenschutz und Sicherheit: kuratierte Zeilen für Gateway-Authentifizierung, Ausführungsrichtlinie, Browseraktivierung, Tool-Profil, Geräteauthentifizierung und mobile Kopplung über den schemagestützten Abschnitten `security`/`approvals`.
    - „Genehmigungen“ enthält einen 30-tägigen Verlauf erledigter Ausführungs-, Plugin- und Systemagent-Anfragen, wobei die neuesten zuerst erscheinen. Filtern Sie nach Art oder blättern Sie durch ältere Zeilen, um die vom Gateway erfasste Entscheidung, Begründung, Quellsitzung und Zuordnung des Bearbeiters zu prüfen.
    - „Labs“ stellt ausgelieferte experimentelle Schalter bereit. Code Mode und Swarm sind die aktuellen Einträge und speichern `tools.codeMode.enabled` und `tools.swarm.enabled` sofort; nicht ausgelieferte Experimente werden weder angezeigt noch schreiben sie spekulative Konfigurationsschlüssel.
    - Benachrichtigungen: Browser-Web-Push-Status, Abonnieren/Abbestellen und ein Testversand.
    - Erweitert: alle Konfigurationsabschnitte ohne eigene kuratierte Seite sowie der rohe JSON5-Editor (zuvor der erweiterte Modus der Seite „Allgemein“).
    - Die Modelleinrichtung (`/settings/model-setup`) ist eine Unterseite von „Modell-Provider“ und wird über deren Kopfzeile geöffnet.
    - Agenten: eine Einstellungsseite (**Einstellungen → Agenten**, `/settings/agents`) mit Tabs pro Agent (Übersicht, Dateien, Tools, Skills, Kanäle, Automatisierungen, Speicher). Im Tab „Übersicht“ wird die Identität des Agenten bearbeitet – Anzeigename, Emoji und ein Avatarbild, das im Browser vor `agents.update` herunterskaliert und größenbegrenzt wird. Beim Speichern werden die konfigurierten Identitätsfelder gespeichert und in `IDENTITY.md` des Arbeitsbereichs gespiegelt; konfigurierte Werte haben Vorrang vor manuellen Änderungen an denselben Dateifeldern.
    - Profil: eine Einstellungsseite, die die Identität des Standardagenten mit Nutzungsstatistiken über den gesamten Zeitraum anzeigt – Token über die gesamte Laufzeit, Spitzentag, längste Sitzung, Aktivitätsserien, eine Token-Heatmap für ein ganzes Jahr, meistgenutzte Tools und Kanal-Highlights (`usage.cost`, `sessions.usage`).
    - MCP verfügt über eine eigene Einstellungsseite mit Serverzeilen (Transport, Aktivierung, Zusammenfassungen zu OAuth/Filtern/Parallelität), direkten Steuerelementen zum Hinzufügen/Aktivieren/Deaktivieren/Entfernen, gängigen Operatorbefehlen und dem bereichsbezogenen Konfigurationseditor `mcp`. Die Seite „Plugins“ bleibt die zentrale Stelle für Ein-Klick-Konnektoren und deren Erkennung.
    - Modell-Provider: eine Einstellungsseite, die jeden konfigurierten Modell-Provider mit seinem Markensymbol, Authentifizierungsstatus (`models.authStatus`), seiner Modellverfügbarkeit (`models.list`), aktuellen Plan-, Kontingent- und Abrechnungsdaten, sofern der Provider diese meldet (`usage.status`), sowie den lokalen Sitzungsausgaben der letzten 30 Tage (`sessions.usage`) auflistet. Die Aktion „Aktualisieren“ liest den Anmeldedatenstatus und die Provider-Nutzung erneut ein.
    - Verbindung: eine Einstellungsseite (unter **Verbindungen**), die die eigene Gateway-Verbindung des Dashboards verwaltet – WebSocket-URL, Gateway-Token, Passwort und Standardsitzungsschlüssel – sowie den neuesten Handshake-Schnappschuss (Status, Betriebszeit, Tick-Intervall, letzte Kanalaktualisierung). Die Offline-Anmeldesperre behandelt den getrennten Zustand; diese Seite bearbeitet die Verbindung im verbundenen Zustand.
    - Mit Validierung anwenden und neu starten (`config.apply`), anschließend die zuletzt aktive Sitzung aufwecken.
    - Schreibvorgänge enthalten eine Basishash-Schutzprüfung, um das Überschreiben gleichzeitiger Änderungen zu verhindern.
    - Schreibvorgänge (`config.set`/`config.apply`/`config.patch`) prüfen vorab die aktive SecretRef-Auflösung für Referenzen in der übermittelten Konfigurationsnutzlast; nicht aufgelöste aktive übermittelte Referenzen werden vor dem Schreiben abgelehnt.
    - Beim Speichern von Formularen werden veraltete geschwärzte Platzhalter verworfen, die nicht aus der gespeicherten Konfiguration wiederhergestellt werden können; geschwärzte Werte, die weiterhin gespeicherten Geheimnissen zugeordnet sind, bleiben erhalten.
    - Schema und Formulardarstellung stammen aus `config.schema` / `config.schema.lookup`, einschließlich der Felder `title`/`description`, passender UI-Hinweise, unmittelbarer Zusammenfassungen untergeordneter Elemente, Dokumentationsmetadaten für verschachtelte Objekt-, Platzhalter-, Array- und Kompositionsknoten sowie Plugin- und Kanalschemas, sofern verfügbar. Der rohe JSON-Editor ist nur verfügbar, wenn der Schnappschuss einen sicheren unveränderten Rücklauf zulässt; andernfalls erzwingt die Control UI den Formularmodus.
    - „Auf gespeicherten Stand zurücksetzen“ im rohen JSON-Editor bewahrt die roh erstellte Form (Formatierung, Kommentare, `$include`-Layout), anstatt einen abgeflachten Schnappschuss neu darzustellen. Dadurch überstehen externe Änderungen eine Zurücksetzung, wenn der Schnappschuss sicher unverändert durchlaufen werden kann.
    - Strukturierte SecretRef-Objektwerte werden in Texteingaben des Formulars schreibgeschützt dargestellt, um eine versehentliche Beschädigung bei der Umwandlung von Objekten in Zeichenfolgen zu verhindern.

  </Accordion>
  <Accordion title="Nutzung">
    - Die aus Sitzungen abgeleitete Analyse von Token und geschätzten Kosten bleibt von der Provider-Abrechnung getrennt.
    - Provider-Karten rufen `usage.status` auf und zeigen aktuelle Plannamen, Kontingentzeiträume, Guthaben, Ausgaben und Budgets an, die von konfigurierten Provider-Plugins gemeldet werden.
    - Ein Fehler bei der Provider-Nutzung blockiert das Sitzungs-/Kosten-Dashboard nicht; nicht verfügbare Provider-Karten zeigen ihren eigenen Fehlerstatus an.

  </Accordion>
  <Accordion title="Debugging, Protokolle, Aktualisierung">
    - Debugging: Schnappschüsse von Status/Systemzustand/Modellen, Ereignisprotokoll und manuelle RPC-Aufrufe (`status`, `health`, `models.list`).
    - Das Ereignisprotokoll enthält Zeitmessungen für Control-UI-Aktualisierungen/RPCs, Zeitmessungen für langsame Chat-/Konfigurationsdarstellungen sowie Einträge zur Browser-Reaktionsfähigkeit bei langen Animationsframes oder lang laufenden Aufgaben, wenn der Browser diese PerformanceObserver-Eintragstypen bereitstellt.
    - Protokolle: fortlaufende Live-Anzeige der Gateway-Dateiprotokolle mit Filter/Export (`logs.tail`).
    - Aktualisierung: eine Paket-/Git-Aktualisierung mit anschließendem Neustart ausführen (`update.run`), einschließlich Neustartbericht, und nach der Wiederverbindung `update.status` abfragen, um die laufende Gateway-Version zu überprüfen.

  </Accordion>
  <Accordion title="Hinweise zum Automatisierungsbereich">
    - Durch Auswahl einer Zeile wird eine ganzseitige Detailansicht mit einem Aktiv/Pausiert-Schalter und „Jetzt ausführen“ in der Kopfzeile geöffnet („Ausführen, falls fällig“, Klonen und Entfernen befinden sich im zugehörigen Menü); der Tab „Einstellungen“ bearbeitet die Automatisierung direkt (Prompt, Details, Häufigkeit, erweiterte Überschreibungen), und der Tab „Ausführungsverlauf“ zeigt die Ausführungen dieser Automatisierung.
    - Die Einstiegsautomatisierungen unter der Tabelle füllen das Erstellungsformular mit einem bearbeitbaren Prompt und Zeitplan vorab aus.
    - Für isolierte Aufgaben ist die Zustellung standardmäßig auf die Ankündigung einer Zusammenfassung eingestellt; wählen Sie „Keine“ für ausschließlich interne Ausführungen.
    - Felder für Kanal/Ziel werden angezeigt, wenn „Ankündigen“ ausgewählt ist.
    - Der Webhook-Modus verwendet `delivery.mode = "webhook"`, wobei `delivery.to` auf eine gültige HTTP(S)-Webhook-URL gesetzt ist.
    - Für Aufgaben der Hauptsitzung stehen die Zustellungsmodi „Webhook“ und „Keine“ zur Verfügung.
    - Die erweiterten Bearbeitungssteuerelemente umfassen das Löschen nach der Ausführung, das Aufheben der Agentenüberschreibung, genaue/versetzte Cron-Optionen, Überschreibungen für Agentenmodell/Denkmodus und Umschalter für die Best-Effort-Zustellung.
    - Die Formularvalidierung erfolgt direkt mit Fehlern auf Feldebene; ungültige Werte deaktivieren die Schaltfläche „Speichern“, bis sie korrigiert wurden.
    - Legen Sie `cron.webhookToken` fest, um ein dediziertes Bearer-Token zu senden; bei Auslassung wird der Webhook ohne Authentifizierungs-Header gesendet.
    - `cron.webhook` ist ein eingestellter Legacy-Fallback, der von der aktuellen Konfigurationsvalidierung abgelehnt wird. Führen Sie `openclaw doctor --fix` aus, um gespeicherte Jobs, die weiterhin `notify: true` verwenden, auf eine explizite Webhook- oder Abschlusszustellung pro Job zu migrieren und den alten Schlüssel zu entfernen.

  </Accordion>
</AccordionGroup>

## Assistentenspeicher importieren

Öffnen Sie **Settings** → **Import Memory**, um lokalen Speicher von Codex oder Claude Code
in einen OpenClaw-Agenten zu übertragen. Das Gateway erkennt unterstützten lokalen Speicher selbstständig
auf seinem Host. Daher importiert eine entfernte Control UI vom Gateway-Computer und nicht vom
Browser-Computer.

1. Wählen Sie den Zielagenten aus.
2. Prüfen Sie die erkannten Quellsammlungen und Markdown-Dateinamen. Dateiinhalte
   werden weder in der Planantwort gesendet noch auf der Seite angezeigt.
3. Wählen Sie die zu importierenden Sammlungen aus und bestätigen Sie. Beim Anwenden wird der Plan vor dem
   Schreiben neu erstellt, sodass veraltete Auswahlen sicher fehlschlagen.
4. Wenn Dateien bereits vorhanden sind, aktivieren Sie **Replace existing imports**, aktualisieren Sie die
   Vorschau und bestätigen Sie die Ersetzung.

Codex importiert nur seine konsolidierten `MEMORY.md` und `memory_summary.md`. Claude
Code importiert Markdown aus den automatischen Speicherverzeichnissen von Projekten und einem konfigurierten
`autoMemoryDirectory`; Sitzungen, Einstellungen, Anweisungen oder
Anmeldedaten werden über diese Seite nicht importiert. Dateien werden unter `memory/imports/` in den
ausgewählten Arbeitsbereich kopiert, wo das aktive Speicher-Plugin sie indizieren kann. Quellen werden
niemals geändert.

Planung und Anwendung erfordern `operator.admin`. Jede Anwendung erstellt eine verifizierte
OpenClaw-Sicherung, sofern ein Zustand vorhanden ist, schreibt einen geschwärzten Migrationsbericht und bewahrt
Sicherungen auf Elementebene auf, bevor vorhandene Zieldateien ersetzt werden. Unter
[Speicherübersicht](/de/concepts/memory#import-from-coding-assistants) finden Sie Pfade und
Abrufverhalten.

## MCP-Seite

Die spezielle MCP-Seite ist eine Operatoransicht für von OpenClaw verwaltete MCP-Server unter `mcp.servers`. Sie startet MCP-Transporte nicht selbstständig; verwenden Sie sie, um die gespeicherte Konfiguration zu prüfen und zu bearbeiten, und verwenden Sie anschließend `openclaw mcp doctor --probe`, wenn Sie einen Nachweis des laufenden Servers benötigen.

Typischer Arbeitsablauf:

1. Öffnen Sie **MCP** über die Seitenleiste.
2. Prüfen Sie die Übersichtskarten auf die Gesamtzahl sowie die Anzahl aktivierter, OAuth-basierter und gefilterter Server.
3. Prüfen Sie jede Serverzeile auf Transport, Aktivierung, Authentifizierung, Filter, Zeitüberschreitungen und Befehlshinweise.
4. Fügen Sie Server direkt auf der MCP-Seite hinzu, aktivieren oder deaktivieren Sie sie oder entfernen Sie sie. Wählen Sie ausdrücklich Streamable HTTP, SSE oder stdio aus; stdio-Befehlszeilen akzeptieren in Anführungszeichen gesetzte Argumente, beispielsweise Pfade mit Leerzeichen. Verwenden Sie die Seite **Plugins** für Konnektoren mit einem Klick und die Erkennung.
5. Bearbeiten Sie den bereichsspezifischen Konfigurationsabschnitt `mcp` für erweiterte Serverfelder wie Umgebungsvariablen, Arbeitsverzeichnisse, Header, TLS-/mTLS-Pfade, OAuth-Metadaten, Toolfilter und Codex-Projektionsmetadaten.
6. Verwenden Sie **Save**, um die Konfiguration zu schreiben, oder **Save & Publish**, wenn der laufende Gateway die geänderte Konfiguration anwenden soll.
7. Führen Sie `openclaw mcp status --verbose`, `openclaw mcp doctor --probe` oder `openclaw mcp reload` in einem Terminal aus, um statische Diagnosen, einen Live-Nachweis oder die Bereinigung der zwischengespeicherten Laufzeitumgebung durchzuführen.

Die Seite schwärzt URL-ähnliche Werte mit Zugangsdaten vor der Darstellung und setzt Servernamen in Befehlsausschnitten in Anführungszeichen, damit kopierte Befehle auch bei Leerzeichen oder Shell-Metazeichen funktionieren. Vollständige CLI- und Konfigurationsreferenz: [MCP](/de/cli/mcp).

## Registerkarte „Aktivität“

Die Registerkarte „Aktivität“ befindet sich unter **Settings › System** neben Logs und Debug. Sie ist ein flüchtiger, browserlokaler Beobachter für Live-Toolaktivitäten und wird aus demselben Gateway-Ereignisstream für `session.tool`-/Tool-Ereignisse abgeleitet, der die Toolkarten im Chat speist. Sie fügt weder eine weitere Gateway-Ereignisfamilie noch einen Endpunkt, einen dauerhaften Aktivitätsspeicher, einen Metrikfeed oder einen externen Beobachterstream hinzu.

Aktivitätseinträge enthalten nur bereinigte Zusammenfassungen sowie geschwärzte, gekürzte Ausgabevorschauen. Werte von Toolargumenten werden nicht im Aktivitätsstatus gespeichert; die Benutzeroberfläche zeigt an, dass Argumente ausgeblendet sind, und erfasst lediglich die Anzahl der Argumentfelder. Die speicherinterne Liste ist an die aktuelle Browserregisterkarte gebunden, bleibt bei der Navigation innerhalb der Control UI erhalten und wird beim Neuladen der Seite, beim Sitzungswechsel oder über **Clear** zurückgesetzt.

## Operator-Terminal

Das andockbare Operator-Terminal ist standardmäßig deaktiviert. Um es zu aktivieren, legen Sie `gateway.terminal.enabled: true` fest und starten Sie den Gateway neu. Das Terminal erfordert eine `operator.admin`-Verbindung und öffnet ein Host-PTY im Arbeitsbereich des aktiven Agenten. Neue Registerkarten folgen dem derzeit ausgewählten Chat-Agenten.

<Warning>
Das Terminal ist eine nicht eingeschränkte Host-Shell und übernimmt die Umgebung des Gateway-Prozesses. Aktivieren Sie es nur für vertrauenswürdige Operator-Bereitstellungen. OpenClaw verweigert Terminalsitzungen für Agenten mit `sandbox.mode: "all"`; wenn ein aktiver Agent in diesen Modus versetzt wird, werden seine bestehenden und laufenden Terminalsitzungen geschlossen.
</Warning>

Verwenden Sie **Ctrl + backtick**, um das Dock ein- oder auszublenden. Das Layout unterstützt das Andocken unten und rechts, passt seine Größe an den Browser-Viewport an und verwaltet mehrere Shell-Registerkarten. Informationen zu `gateway.terminal.enabled` und der optionalen Überschreibung `gateway.terminal.shell` finden Sie unter [Gateway-Konfiguration](/de/gateway/configuration-reference#gateway).

Vom Eigentümer autorisierte, nicht sandboxierte Agenten können das Tool `terminal` für lang andauernde oder interaktive Arbeiten verwenden, die der Operator beobachten soll. Jeder Toolaufruf kann die eigenen Gateway-PTYs des Agenten öffnen, lesen, beschreiben, in der Größe ändern, schließen oder auflisten. Neue Sitzungen öffnen standardmäßig eine mitverbundene Registerkarte der Control UI, sodass Agent und Operator dieselbe Ausgabe sehen und beide Eingaben vornehmen oder die Größe ändern können. Der Agentenzugriff ist exakt auf die jeweilige Sitzung beschränkt: Ein Agent kann weder vom Operator erstellte Terminals noch Terminals, die von einer anderen Agentensitzung geöffnet wurden, lesen oder steuern.

Ziehen Sie eine oder mehrere Dateien auf das aktive Terminal oder wählen Sie Dateien über die Büroklammer-Schaltfläche aus. OpenClaw stellt jede Datei auf dem Computer bereit, dem das PTY gehört, und fügt Shell-kompatibel in Anführungszeichen gesetzte absolute Pfade an der Cursorposition ein; es betätigt niemals die Eingabetaste und führt die Eingabe nicht aus. Eine kompakte Stapelanzeige zeigt die aktuelle Datei und die Anzahl der abgeschlossenen Dateien. Durch Abbrechen wird der verbleibende Stapel gestoppt, ohne Pfade einzufügen; eine fehlgeschlagene Übertragung bleibt sichtbar, sodass Sie den Vorgang ab dieser Datei wiederholen können, ohne bereits abgeschlossene Dateien erneut hochzuladen. Bilder, PDFs, Archive und andere Dateitypen werden bis zu 16 MiB pro Datei akzeptiert. Bereitgestellte Dateien verwenden auf POSIX-Hosts ein privates temporäres Systemverzeichnis (Verzeichnismodus `0700`, Dateimodus `0600`) oder unter Windows ein Verzeichnis innerhalb der ACL-Grenze des Benutzerprofils sowie einen Bereinigungstimer von 24 Stunden. Verschieben oder kopieren Sie daher alle Dateien, die Sie behalten möchten.

Das Einfügen von Pfaden unterstützt PowerShell, `cmd.exe` und erkannte POSIX-Shells (`sh`, Bash, Dash, Ash, Ksh, Zsh und Fish), einschließlich Git Bash unter Windows. Andere Shell-Überschreibungen werden abgelehnt, da ihre Regeln für Anführungszeichen nicht sicher abgeleitet werden können; führen Sie den Gateway innerhalb von WSL aus, um ein natives WSL-Terminal und Linux-Uploadpfade zu erhalten. `cmd.exe`-Pfade, die `%` oder `!` enthalten, werden ebenfalls abgelehnt, da diese Shell die betreffenden Zeichen selbst innerhalb doppelter Anführungszeichen expandiert.

Codex- und Claude-Code-Sitzungen, die in der Sitzungsseitenleiste erkannt werden, können in ihrer nativen CLI innerhalb desselben Terminalbereichs geöffnet werden. Legen Sie unter **Settings › Chat** für **Open Codex/Claude threads in** den Wert **Terminal** fest, damit ein normaler Klick auf eine Zeile `codex resume` oder `claude --resume` öffnet; standardmäßig bleibt die schreibgeschützte OpenClaw-Ansicht erhalten. Das Kontext- oder Drei-Punkte-Menü einer Zeile bietet stets beide Optionen an, und der Kopfbereich der Ansicht enthält **Open in terminal**, wenn die Sitzung dafür geeignet ist.

Die Eignung wird pro Sitzung und Host bestimmt. Gateway-lokale Sitzungen starten den Provider-eigenen Fortsetzungsbefehl auf dem Gateway-Host. Sitzungen auf gekoppelten Nodes starten einen zulässigen Provider-Befehl auf dem besitzenden Node und übertragen ausschließlich Ausgabe, Eingabe und Größenänderungsereignisse dieses PTYs; dadurch wird weder eine allgemeine Node-Shell verfügbar gemacht noch werden vom Browser bereitgestellte Befehle akzeptiert. Datei-Uploads verwenden den separaten, größenbeschränkten Node-Befehl `terminal.upload` und bleiben an die bereits geöffnete Terminalsitzung gebunden. Genehmigen Sie das Upgrade der Node-Kopplung, wenn dieser Befehl erstmals angezeigt wird. Nodes, die den passenden Befehl zum Fortsetzen des Terminals nicht ankündigen, einschließlich eingebetteter Worker-Bridges ohne Duplex-Streaming, halten die Ansicht verfügbar und zeigen das Öffnen des Terminals als nicht verfügbar an; ältere Nodes können weiterhin ein Terminal ausführen, jedoch keine hineingezogenen Dateien empfangen.

Verbindungseigene Sitzungen überstehen Verbindungsabbrüche: Beim Neuladen einer Seite, im Ruhezustand des Laptops oder bei einer kurzen Netzwerkunterbrechung wird die Sitzung am Gateway getrennt, statt beendet zu werden. Dieselbe Browserregisterkarte verbindet sich nach der Wiederherstellung der Verbindung erneut, wobei die letzte Ausgabe wiedergegeben wird. Getrennte verbindungseigene Sitzungen werden nach `gateway.terminal.detachedSessionTimeoutSeconds` beendet (Standardwert: 300 Sekunden; `0` stellt das Beenden bei Verbindungsabbruch wieder her). Das Verbinden mit einer dieser Sitzungen bleibt eine Übernahme nach Art von tmux.

Agenteneigene Sitzungen sind nicht an eine Browserverbindung gebunden. `terminal.attach` fügt jeden Browser als Betrachter hinzu, ohne die Eigentümerschaft zu übernehmen, und beim Schließen einer Betrachterregisterkarte wird nur dieser Browser getrennt. Das PTY bleibt bestehen, bis der besitzende Agent es schließt, sein Prozess beendet wird, eine Richtlinie es deaktiviert oder der Gateway heruntergefahren wird. `terminal.list` kennzeichnet jeden Eintrag als verbindungs- oder agenteneigen, und `terminal.text` ermöglicht einer Administratorverbindung, die jüngste Klartextausgabe zu lesen, ohne eine Verbindung mit der Sitzung herzustellen.

Das Terminal ist außerdem als bildschirmfüllendes Dokument verfügbar, das ausschließlich das Terminal enthält: `/?view=terminal`. Die iOS- und Android-Apps betten diese Seite in ihre Terminalansichten ein und verwenden dabei die gespeicherten Gateway-Zugangsdaten erneut; die Verfügbarkeit unterliegt denselben Schranken `gateway.terminal.enabled` und `operator.admin`, und die Seite zeigt einen Hinweis an, wenn der verbundene Gateway kein Terminal bereitstellt.

## Browserbereich

Die Control UI enthält einen andockbaren Browserbereich, der den vom Gateway gesteuerten Browser – denselben Browser, den Agenten über das [Browser-Tool](/de/tools/browser-control) steuern – in jedem regulären Webbrowser darstellt; eine native Webview ist nicht erforderlich. Er wird angezeigt, wenn der verbundene Gateway einer `operator.admin`-Verbindung `browser.request` ankündigt; die Globus-Schaltfläche in der Arbeitsbereichsleiste des Threads blendet ihn ein oder aus. Der Bereich zeigt eine Live-Momentaufnahme der Seite mit Registerkarten, einer bearbeitbaren URL-Leiste, Zurück-/Vorwärts-/Neu-laden-Steuerelementen und einer Option zum Öffnen im eigenen Browser. Er lässt sich rechts oder unten andocken und leitet Klicks, Scrollen mit dem Mausrad sowie grundlegende Texteingaben an die Remote-Seite weiter.

Zwei Erfassungsmodi stellen Seitenkontext für den Agenten zusammen:

- **Annotieren (Stift)**: Zeichnen Sie freihändige Markierungen über die Seite. **Send to chat** fügt die Striche in den Screenshot ein, hängt das Bild an den Editor des aktiven Chats an und füllt eine Eingabeaufforderung mit einer Beschreibung der Seiten-URL, des Titels und jedes markierten Bereichs vorab aus, sodass der Agent genau weiß, was Sie eingekreist haben.
- **Untersuchen (Zeiger)**: Bewegen Sie den Mauszeiger über ein Element, um das darunterliegende Element anzuzeigen (Selektor, barrierefreier Name, Rolle, Größe); klicken Sie, um die Details dieses Elements sowie einen hervorgehobenen Screenshot über denselben Editorablauf zu senden. Untersuchen, Scrollen mit dem Mausrad und Zurück/Vorwärts benötigen `browser.evaluateEnabled` (standardmäßig aktiviert).

Die macOS-App behält ihre native Link-Browser-Seitenleiste für Links bei, die im Dashboard angeklickt werden; der Browserbereich funktioniert dort ebenfalls und ermöglicht auf allen anderen Plattformen das Annotieren von Seiten.

## Chatverhalten

<AccordionGroup>
  <Accordion title="Send and history semantics">
    - `chat.send` ist **nicht blockierend**: Die Anfrage wird sofort mit `{ runId, status: "started" }` bestätigt, und die Antwort wird über `chat`-Ereignisse gestreamt. Vertrauenswürdige Control-UI-Clients können außerdem optionale ACK-Zeitmetadaten für die lokale Diagnose erhalten.
    - Chat-Uploads akzeptieren Bilder sowie Dateien, die keine Videos sind. Bilder behalten den nativen Bildpfad; andere Dateien werden als verwaltete Medien gespeichert und im Verlauf als Anhangslinks angezeigt.
    - Erneutes Senden mit demselben `idempotencyKey` gibt während der Ausführung `{ status: "in_flight" }` und nach Abschluss `{ status: "ok" }` zurück.
    - Antworten von `chat.history` sind zur Sicherheit der Benutzeroberfläche größenbeschränkt. Wenn Transkripteinträge zu groß sind, kann der Gateway lange Textfelder kürzen, umfangreiche Metadatenblöcke auslassen und übergroße Nachrichten durch einen Platzhalter (`[chat.history omitted: message too large]`) ersetzen.
    - Wenn eine sichtbare Assistentennachricht in `chat.history` gekürzt wurde, kann die Seitenansicht den vollständigen, für die Anzeige normalisierten Transkripteintrag bei Bedarf über `chat.message.get` anhand von `sessionKey`, bei Bedarf aktivem `agentId` und dem Transkript-`messageId` abrufen. Falls der Gateway weiterhin keine weiteren Inhalte zurückgeben kann, zeigt die Ansicht ausdrücklich einen Nicht-verfügbar-Zustand an, statt stillschweigend erneut die gekürzte Vorschau anzuzeigen.
    - Vom Assistenten bzw. generierte Bilder werden als verwaltete Medienreferenzen dauerhaft gespeichert und über authentifizierte Gateway-Medien-URLs wieder bereitgestellt. Dadurch hängen erneute Ladevorgänge nicht davon ab, dass Base64-Rohbilddaten in der Antwort des Chatverlaufs erhalten bleiben.
    - Beim Rendern von `chat.history` entfernt die Control UI aus dem sichtbaren Assistententext Inline-Direktiven-Tags, die ausschließlich der Anzeige dienen (zum Beispiel `[[reply_to_*]]` und `[[audio_as_voice]]`), Nur-Text-XML-Nutzlasten von Tool-Aufrufen (einschließlich `<tool_call>...</tool_call>`, `<function_call>...</function_call>`, `<tool_calls>...</tool_calls>`, `<function_calls>...</function_calls>` und gekürzter Tool-Aufrufblöcke) sowie offengelegte ASCII-/vollbreite Modellsteuerungs-Token. Assistenteneinträge werden ausgelassen, wenn ihr gesamter sichtbarer Text ausschließlich aus dem exakten Stille-Token `NO_REPLY` / `no_reply` oder dem Heartbeat-Bestätigungs-Token `HEARTBEAT_OK` besteht.
    - Während eines aktiven Sendevorgangs und der abschließenden Aktualisierung des Verlaufs hält die Chatansicht lokale, optimistisch dargestellte Benutzer- und Assistentennachrichten sichtbar, falls `chat.history` kurzzeitig einen älteren Snapshot zurückgibt. Das kanonische Transkript ersetzt diese lokalen Nachrichten, sobald der Gateway-Verlauf aufgeholt hat.
    - Live-Ereignisse von `chat` stellen den Zustellstatus dar, während `chat.history` aus dem dauerhaften Sitzungstranskript neu aufgebaut wird. Nach abschließenden Tool-Ereignissen lädt die Control UI den Verlauf neu und führt nur ein kleines optimistisches Ende zusammen; die Transkriptgrenze ist unter [WebChat](/de/web/webchat) dokumentiert.
    - `chat.inject` hängt eine Assistentennotiz an das Sitzungstranskript an und sendet ein `chat`-Ereignis für reine UI-Aktualisierungen (keine Agentenausführung, keine Kanalzustellung).
    - Die Seitenleiste führt jede geladene aktive Sitzung nach Agentenabschnitt sowie in den Gruppen „Angeheftet“, „Kanal“, „Arbeit“, benutzerdefiniert und „Chats“ auf. Eine einzige Aktion „Neue Sitzung“ öffnet den Entwurfsdialog. Das Öffnen einer sichtbaren Zeile verschiebt lediglich die Hervorhebung. Sitzungen können auf „Angeheftet“ gezogen werden, um sie anzuheften, oder auf eine benutzerdefinierte Gruppe bzw. „Chats“, um sie zu verschieben. Benutzerdefinierte Gruppen lassen sich ein- und ausklappen und durch Ziehen neu anordnen; Gruppennamen und Reihenfolge werden über den Gateway synchronisiert, während der eingeklappte Zustand im Browser verbleibt. Eine neue Dashboard-Sitzung erhält asynchron einen prägnanten generierten Titel aus ihrer ersten Nachricht, die kein Befehl ist. Explizite Namen und die authentifizierte Absenderidentität bleiben davon getrennt, sodass Kontonamen niemals als generierte Titel verwendet werden. Legen Sie `agents.defaults.utilityModel` (oder `agents.entries.*.utilityModel`) fest, um diesen separaten Modellaufruf an ein kostengünstigeres Modell weiterzuleiten. Falls dieses andere Modell fehlschlägt, wird die Titelgenerierung einmal mit dem primären Modell wiederholt. Durch Erweitern eines anderen Agentenabschnitts können die Sitzungen dieses Agenten durchsucht werden, ohne den geöffneten Chat zu verlassen.
    - Die Thread-Suche befindet sich in der Befehlspalette (⌘K oder die Suchschaltfläche in der Steuerungsgruppe oben links): Bei Eingabe einer Suchanfrage wird agentenübergreifend eine begrenzte Anzahl übereinstimmender Seiten durchsucht, interne untergeordnete Zeilen und Cron-Zeilen werden herausgefiltert und sichtbare Treffer neben Navigationsbefehlen aufgeführt. Die Seite „Threads“ enthält weiterhin die vollständige durchsuchbare Liste mit Filtern.
    - Jede Seitenleistenzeile bietet direkten Zugriff auf das Anheften sowie ein vollständiges Kontextmenü für Ungelesen-Status, Umbenennen, Forken, Gruppieren, Archivieren und Löschen. Für mehrfach ausgewählte Zeilen (Cmd-/Strg-Klick, Umschalt-Klick für Bereiche) steht ein Stapelmenü für Ungelesen-Status, Gruppierung, Archivierung und Löschung zur Verfügung. Stapelarchivierung und -löschung bleiben deaktiviert, sofern nicht jede ausgewählte Sitzung archiviert werden kann. Eine aktive Ausführung und die Hauptsitzung eines Agenten können nicht archiviert werden. Beim Archivieren oder Löschen der derzeit ausgewählten Sitzung wechselt der Chat zurück zur Hauptsitzung dieses Agenten.
    - In der macOS-App nutzt das OpenClaw-Logo den ansonsten leeren nativen Titelleistenbereich neben den Fenstersteuerelementen, statt eine Zeile der Seitenleiste zu belegen.
    - Bei Desktop-Breiten bleiben die Chat-Steuerelemente in einer kompakten Zeile und werden beim Abwärtsscrollen im Transkript eingeklappt. Beim Aufwärtsscrollen, bei der Rückkehr zum Anfang oder beim Erreichen des Endes werden die Steuerelemente wieder eingeblendet.
    - Der Sitzungskopf zeigt neben dem Arbeitsbereich-Chip eine kleine Avatargruppe an, wenn andere Personen dieselbe Sitzung betrachten. Sie enthält bis zu vier Betrachter-Avatare mit einer Anzahl für weitere Personen und verschwindet, wenn Sie allein sind.
    - Aufeinanderfolgende identische Nachrichten, die ausschließlich Text enthalten, werden als eine Sprechblase mit einem Zähler dargestellt. Nachrichten mit Bildern, Anhängen, Tool-Ausgaben oder Canvas-Vorschauen werden nicht zusammengefasst.
    - Sprechblasen von Benutzernachrichten enthalten Transkriptaktionen: eine beim Darüberfahren sichtbare Zurückspulen-Schaltfläche (Bestätigungs-Popover mit der Option „Don't ask again“) sowie per Rechtsklick **Bis hierher zurückspulen** und **Ab hier forken**. Das Zurückspulen setzt die Sitzung auf den Zustand unmittelbar vor dieser Nachricht zurück und überträgt deren Text zum Bearbeiten und erneuten Senden in den Editor (`sessions.rewind`, `operator.admin`). Das Forken erstellt aus dem Präfix des aktiven Pfads vor der Nachricht eine neue Sitzung, öffnet sie und befüllt ihren Editor mit demselben Text (`sessions.fork`, `operator.write`). Beide Aktionen sind mit einem erklärenden Tooltip deaktiviert, während der Agent arbeitet, gelten nur für dauerhaft gespeicherte Benutzernachrichten und werden für Sitzungen abgelehnt, deren Unterhaltung einem externen Agenten-Harness gehört. Das Zurückspulen verschiebt nur den Chatkontext – Dateien und andere Nebenwirkungen von Tools werden nicht rückgängig gemacht – und das Transkript vor dem Zurückspulen bleibt im ausschließlich anhängenden Sitzungsspeicher erhalten. Wenn dieser Speicher mehrere Transkriptzweige enthält, zeigt die Chat-Titelleiste ein Zweigmenü mit der jeweils neuesten Nachricht, Nachrichtenanzahl und Aktualität jedes Zweigs an. Durch Auswahl eines inaktiven Zweigs wird die aktuelle Sitzung auf diesen erhaltenen Pfad zurückgesetzt (`sessions.branches.list`, `operator.read`; `sessions.branches.switch`, `operator.admin`). Der Zweigwechsel ist ebenfalls nicht verfügbar, während der Agent arbeitet. Die Auswahl des bereits aktiven Zweigs führt an der RPC-Grenze zu einem typisierten No-op-Fehler. Die separate Aktion zum Ausblenden an Benutzersprechblasen blendet eine Nachricht nur im aktuellen Browser aus; die Nachricht verbleibt im Transkript und ist weiterhin für den Agenten sichtbar.
    - Wenn sich der Checkout einer Sitzung in einem GitHub-Repository auf einem Nicht-Standard-Branch befindet, heftet die Chatansicht Pull-Request-Chips oberhalb des Editors an: PR-Nummer, Repository, Branch, Diff-Anzahlen, eine CI-Kennzeichnung sowie Entwurfs-, Zusammengeführt- oder Geschlossen-Status, jeweils mit einem Link zum PR. Die Zeile zeigt höchstens zwei Chips – Live-PRs (offen/Entwurf) zuerst – und eine Schaltfläche „Show more“ blendet den eingeklappten Verlauf zusammengeführter bzw. geschlossener PRs ein. Die CI-Kennzeichnung öffnet ein kleines Popover zur CI-Überwachung mit Anzahlen bestandener, fehlgeschlagener, laufender und übersprungener Prüfungen sowie einem Link zur Prüfungsseite des PRs. Die Erkennung erfolgt serverseitig über `controlUi.sessionPullRequests`, das die `GH_TOKEN`/`GITHUB_TOKEN` des Gateways wiederverwendet, sofern sie festgelegt sind. Wenn das Ratenlimit der GitHub-API erreicht ist, behalten die Chips den zuletzt bekannten Status bei und zeigen eine Warnung an, dass der Status möglicherweise veraltet ist. Durch Schließen eines Chips wird er für diese Sitzung im aktuellen Browserprofil ausgeblendet. Bevor ein PR vorhanden ist, zeigt die Zeile den Branch selbst an – Repository, Branch-Name und die +/−-Größe des Diffs gegenüber der Merge-Basis des Standard-Branchs (committete und nicht committete Arbeit). Sobald der gepushte Branch vergleichbare Commits enthält, fügt die Zeile eine Schaltfläche „Create PR“ hinzu, die GitHubs Seite für neue Pull Requests öffnet. Davor wird die Zeile für eine Sitzung mit geänderten Dateien (committet, nicht committet oder nicht verfolgt) weiterhin angezeigt, jedoch ohne diese Schaltfläche. Die Zeile blendet sich aus, solange ein offener PR oder PR-Entwurf vorhanden ist. Die Branch-Zeile basiert ausschließlich auf lokalem Git und bleibt daher auch während einer GitHub-Ratenbegrenzung verfügbar. Sie zeigt dieselbe Warnung vor einem möglicherweise veralteten Status an, da „kein PR gefunden“ bis zum Zurücksetzen des Limits nicht verlässlich ist.
    - Das Sitzungs-Diff-Panel zeigt, was der Checkout einer Sitzung tatsächlich geändert hat: Die Branch-Schaltfläche in der Arbeitsbereichsleiste oder der Chat-Titelleiste öffnet das Detailpanel mit einem Diff pro Datei für Branch-Arbeit, nicht committete und nicht verfolgte Arbeit gegenüber der Merge-Basis des Standard-Branchs des Checkouts – Statuspunkt, Umbenennungspfeil, +/−-Anzahlen pro Datei, einklappbare Dateien und Markierungen für „N unveränderte Zeilen“ zwischen Änderungen. Diffs werden serverseitig über die Gateway-Methode `sessions.diff` (`operator.read`-Geltungsbereich) berechnet. Binäre und übergroße Dateien werden auf Einträge beschränkt, die nur Statistiken enthalten, und die Schaltfläche erscheint nur, wenn der verbundene Gateway `sessions.diff` bekannt gibt.
    - Jeder Chat-Bereich besitzt eine Titelleiste. Klicken Sie auf den Sitzungstitel, um ihn umzubenennen. Der Arbeitsbereich-Chip kopiert den Checkout-Pfad oder Branch und kann lokale Gateway-Arbeitsbereiche im Dateimanager des Hosts anzeigen. Sitzungen auf Remote- und Ausführungs-Nodes behalten die Kopieraktionen bei, blenden die Anzeigeaktion jedoch aus.
    - Die Thread-Arbeitsbereichsleiste in jedem Chat-Bereich führt Thread-Dateien, Projektdateien und Artefakte auf. Standardmäßig ist sie am rechten Rand des Bereichs angedockt. Ziehen Sie ihre Kopfzeile (oder verwenden Sie die Andocken-Schaltfläche), um sie nach unten zu verschieben; die Auswahl wird im aktuellen Browserprofil gespeichert. Eine eingeklappte Leiste belegt keinerlei Platz: Öffnen Sie sie mit ⇧⌘B oder über den Datei-Umschalter in der Titelleiste erneut, der eine Kennzeichnung mit der Anzahl geänderter Dateien trägt. Das separate Detailpanel für Dateien, Tools und Canvas bleibt davon unberührt.
    - Durch Klicken auf eine Dateireferenz im Chat, einen Dateipfad in einer erweiterten Tool-Karte zum Lesen/Bearbeiten/Schreiben oder eine Dateizeile in der Arbeitsbereichsleiste wird das Datei-Detailpanel geöffnet: eine CodeMirror-basierte Codeansicht mit Syntaxhervorhebung, Zeilennummern, Sprung zu einer Zeile, Suche innerhalb der Datei, Kopieraktionen und einem Menü zum Öffnen in einem externen Editor. Wenn der Gateway gegenüber einer `operator.admin`-Verbindung `sessions.files.set` bekannt gibt, fügt das Panel einen Bearbeitungsmodus mit Änderungsverfolgung und Speichern über Cmd-/Strg-S hinzu. Nicht gespeicherte Entwürfe bleiben beim Navigieren zwischen Dateien, Panels und Sitzungen im aktuellen Browser-Tab erhalten, bis sie ausdrücklich gespeichert oder verworfen werden. Das Speichern erfolgt per Compare-and-Swap anhand eines von `sessions.files.get` zurückgegebenen Inhalts-Hashes. Falls sich die Datei seit dem Laden auf dem Datenträger geändert hat (beispielsweise weil der Agent weitergearbeitet hat), zeigt das Panel einen Konflikthinweis mit den Aktionen Reload (neuesten Inhalt übernehmen) und Overwrite (lokale Bearbeitung beibehalten) an. Schreibvorgänge durchlaufen dieselben dateisystemsicheren Arbeitsbereichs-Schutzmechanismen wie Lesevorgänge – Beschränkung der Pfade auf den Arbeitsbereich, Ablehnung von symbolischen und harten Links sowie eine UTF-8-Obergrenze von 256 KB – und überschreiben ausschließlich vorhandene Dateien. Der Editor erstellt oder löscht niemals Dateien.
    - Die Leiste für Hintergrundaufgaben in jedem Chat-Bereich führt die Hintergrundaufgaben und Subagenten des aktuellen Agenten auf (`tasks.list`, nach Agent begrenzt und durch `task`-Ereignisse aktuell gehalten): Laufende Arbeit zeigt einen live aktualisierten Zeitgeber für die verstrichene Zeit, die Anzahl der Tool-Verwendungen, das aktuell verwendete Tool und ein Steuerelement zum Stoppen. Der einklappbare Abschnitt für abgeschlossene Aufgaben ergänzt die Ausführungsdauer. Ein Link „View transcript“ öffnet die untergeordnete Sitzung der Aufgabe im Bereich. Öffnen Sie die Leiste über den Aktivitäts-Umschalter in der Titelleiste. Der Aufgaben-Snapshot wird vorab geladen, sodass der Umschalter bereits vor dem ersten Öffnen der Leiste eine Kennzeichnung mit der Anzahl laufender Aufgaben trägt. Die Seite „Aufgaben“ bleibt das vollständige agentenübergreifende Verzeichnis.
    - Die Arbeitsbereichsleiste, die Leiste für Hintergrundaufgaben und das Detailpanel passen sich jeweils an die Breite ihres eigenen Bereichs statt an die des Fensters an: In einem schmalen Bereich oder kompakten Fenster werden beide Leisten als Streifen am unteren Rand dargestellt (die Steuerelemente zum seitlichen Andocken bleiben ausgeblendet, bis der Bereich breiter wird; die Arbeitsbereichsleiste hat Vorrang für den seitlichen Platz, wenn nur eine Spalte hineinpasst), und das Detailpanel wird unterhalb des Threads mit einem horizontalen Größenänderungsgriff angeordnet, statt sich mit ihm eine Zeile zu teilen. Bei Viewports in Smartphone-Größe wird das Detailpanel weiterhin im Vollbildmodus geöffnet.
    - Die Modell- und Thinking-Auswahlfelder im Chat-Header aktualisieren die aktive Sitzung sofort über `sessions.patch`; es handelt sich um dauerhafte Sitzungsüberschreibungen, nicht um nur für einen einzelnen Sendevorgang geltende Optionen.
    - **Geteilte Ansicht:** Öffnen Sie sie über die Titelleiste des Chats (neben den Umschaltern für Thread-Diff, Hintergrundaufgaben und Thread-Dateien) und teilen Sie dann den aktiven Bereich nach rechts oder unten in so viele Bereiche, wie Platz finden. Jeder Bereich verfügt über einen eigenen Thread, ein eigenes Transkript, einen eigenen Composer und einen eigenen Tool-Stream.
    - Agenten mit dem Tool `screen` können dieselben Änderungen an Bereichen, Seitenleiste, Terminal, Browser, Fokus und Navigation anfordern, während eine kompatible Control UI verbunden ist. Protokoll v1 wendet den Befehl auf jede verbundene kompatible Control UI an; siehe [Bildschirm](/de/tools/screen).
    - Ziehen Sie eine Sitzung aus der Seitenleiste in den Chat, um sie in einem Bereich zu öffnen. Eine animierte Ablagevorschau gleitet zwischen den Zonen und kennzeichnet das Ergebnis — „Teilen“ über genau der Hälfte, die ein neuer Bereich einnehmen wird, „Hier öffnen“ über einem vollständigen Bereich — und das Ablegen funktioniert auch im Modus mit nur einem Bereich.
    - Der aktive geteilte Bereich bestimmt die Auswahl in der Seitenleiste und die URL. Seine Titelleiste enthält zusätzliche Steuerelemente zum Teilen und Schließen; Trennlinien ändern die Größe von Spalten und übereinander angeordneten Bereichen, und der Browser speichert das Layout lokal über Neuladevorgänge hinweg.
    - Auf schmalen Bildschirmen behält die geteilte Ansicht das Layout bei, stellt jedoch nur den aktiven Bereich dar, einschließlich seines Headers mit der Schließen-Schaltfläche.
    - Wenn Sie eine Nachricht senden, während eine Änderung der Modellauswahl für dieselbe Sitzung noch gespeichert wird, wartet der Composer auf diese Sitzungsaktualisierung, bevor `chat.send` aufgerufen wird, damit für den Sendevorgang das ausgewählte Modell verwendet wird.
    - Die Eingabe von `/new` erstellt dieselbe neue Dashboard-Sitzung wie „Neuer Chat“, wechselt zu ihr und verwendet sie anschließend, außer wenn `session.dmScope: "main"` konfiguriert ist und das aktuelle übergeordnete Element die Hauptsitzung des Agenten ist; dann wird die Hauptsitzung direkt zurückgesetzt. Die Eingabe von `/reset` behält das explizite direkte Zurücksetzen des Gateways für die aktuelle Sitzung bei.
    - Die Modellauswahl im Chat fordert die konfigurierte Modellansicht des Gateways an. Wenn `agents.defaults.modelPolicy.allow` nicht leer ist, bestimmt diese Richtlinie die Auswahl, einschließlich Einträgen vom Typ `provider/*`, durch die Provider-spezifische Kataloge dynamisch bleiben. Andernfalls zeigt die Auswahl konfigurierte Einträge sowie Provider mit verwendbarer Authentifizierung an; Aliasse und Einstellungen unter `agents.defaults.models` schränken sie nicht ein. Der vollständige Katalog bleibt über den Debug-RPC `models.list` mit `view: "all"` verfügbar.
    - Wenn aktuelle Nutzungsberichte der Gateway-Sitzung die derzeitigen Kontext-Token enthalten, zeigt die Symbolleiste des Chat-Composers einen kleinen Ring zur Kontextnutzung mit dem verwendeten Prozentsatz an. Öffnen Sie den Ring, um das aktuelle Kontextfenster, die Token-Anzahlen des letzten Laufs und die geschätzten Gesamtkosten, die Provider-/Modellidentität sowie, sofern gemeldet, die Aufschlüsselung der Eingabe-, Ausgabe- und Cache-Kosten der neuesten Provider-Antwort anzuzeigen. Bei hoher Kontextauslastung wechselt der Ring zu einem Warnstil und zeigt bei empfohlenen Compaction-Stufen eine kompakte Schaltfläche an, die den normalen Compaction-Pfad der Sitzung ausführt. Veraltete Token-Momentaufnahmen bleiben ausgeblendet, bis das Gateway erneut aktuelle Nutzungsdaten meldet.

  </Accordion>
  <Accordion title="Sprechmodus (Browser-Echtzeit)">
    Der Sprechmodus verwendet einen registrierten Echtzeit-Sprach-Provider. Konfigurieren Sie OpenAI mit `talk.realtime.provider: "openai"` sowie einem API-Schlüsselprofil für `openai`, `talk.realtime.providers.openai.apiKey` oder `OPENAI_API_KEY`. OpenAI Realtime verwendet die öffentliche Platform API und erfordert einen Platform-API-Schlüssel; eine Codex-OAuth-Anmeldung genügt für diese Schnittstelle nicht. Konfigurieren Sie Google mit `talk.realtime.provider: "google"` sowie `talk.realtime.providers.google.apiKey`. Der Browser erhält niemals einen regulären Provider-API-Schlüssel: OpenAI erhält für WebRTC ein kurzlebiges Realtime-Client-Secret, und Google Live erhält für eine Browser-WebSocket-Sitzung ein einmalig verwendbares, eingeschränktes Live-API-Authentifizierungstoken, wobei Anweisungen und Tool-Deklarationen durch das Gateway fest im Token verankert werden. Provider, die nur eine Echtzeit-Backend-Bridge bereitstellen, werden über den Gateway-Relay-Transport ausgeführt, sodass Anmeldedaten und Anbieter-Sockets serverseitig bleiben, während Browser-Audio über authentifizierte Gateway-RPCs übertragen wird. Der Prompt der Realtime-Sitzung wird vom Gateway zusammengestellt; `talk.client.create` akzeptiert keine vom Aufrufer bereitgestellten Überschreibungen der Anweisungen.

    Dauerhafte Standardwerte für Provider, Modell, Stimme, Transport, Reasoning-Aufwand, exakten VAD-Schwellenwert, Stilledauer und Präfixauffüllung befinden sich unter **Settings → Communications → Talk**; für Änderungen ist Zugriff auf `operator.admin` erforderlich. Die Konfiguration des Gateway-Relays erzwingt den Backend-Relay-Pfad; bei der Konfiguration von WebRTC bleibt die Sitzung im Besitz des Clients und schlägt fehl, statt stillschweigend auf Relay zurückzufallen, wenn der Provider keine Browsersitzung erstellen kann.

    Das Steuerelement für Talk ist die Mikrofonschaltfläche in der Werkzeugleiste des Eingabefelds. Das zugehörige Caret-Menü listet **System default** und jedes vom Browser bereitgestellte Mikrofon auf, einschließlich USB-, Bluetooth- und virtueller Eingänge. Die ID des ausgewählten Geräts verbleibt lokal im Browser und wird niemals an das Gateway gesendet. Wenn genau dieses Gerät nicht mehr verfügbar ist, fordert Talk Sie zur Auswahl eines anderen Eingangs auf, statt unbemerkt über ein anderes Mikrofon aufzunehmen. Während Talk aktiv ist, wird die Mikrofonschaltfläche zu einer Pillenanzeige mit dem Live-Eingangspegel; ein Klick darauf beendet die Spracheingabe, und beim Darüberfahren wird das Stoppsymbol angezeigt. Screenreader geben `Connecting voice input...`, `Listening...` oder `Asking OpenClaw...` aus, während ein Echtzeit-Tool-Aufruf über `talk.client.toolCall` das konfigurierte größere Modell konsultiert. Das Beenden einer laufenden Agentenantwort bleibt ein separates quadratisches **Stop**-Steuerelement neben der Pillenanzeige.

    **Video-Talk** ist für OpenAI-Realtime-WebRTC- und Google-Live-Browsersitzungen verfügbar. Klicken Sie auf die Kameraschaltfläche, erlauben Sie den Zugriff auf Kamera und Mikrofon und bestätigen Sie die lokale Vorschau. OpenAI sendet einen größenbegrenzten JPEG-Frame über seinen Browser-Datenkanal, wenn `describe_view` visuellen Kontext anfordert. Google Live sendet größenbegrenzte JPEG-Frames direkt vom Browser mit dem unterstützten Maximum von einem Frame pro Sekunde an den Provider und beantwortet `describe_view`-Funktionsaufrufe mit dem Status des Kamerastreams. Kameraframes werden niemals über das Gateway übertragen. Beim Beenden von Talk wird die Vorschau geschlossen, und beide Medienspuren werden freigegeben. Informationen zu den Übertragungsverträgen des Providers finden Sie in Googles Dokumentation zu den [Funktionen der Live API](https://ai.google.dev/gemini-api/docs/live-api/capabilities#video) und im [Leitfaden zu Funktionsaufrufen](https://ai.google.dev/gemini-api/docs/live-api/tools).

    Live-Smoke-Test für Maintainer: `OPENAI_API_KEY=... GEMINI_API_KEY=... node --import tsx scripts/dev/realtime-talk-live-smoke.ts` überprüft die OpenAI-Backend-WebSocket-Bridge, den WebRTC-SDP-Austausch des OpenAI-Browsers, die Browser-Einrichtung von Google Live mit eingeschränktem Token, einem JPEG-Frame und einem `describe_view`-Funktions-Roundtrip sowie den Gateway-Relay-Browseradapter mit simulierten Mikrofonmedien. Der Befehl gibt nur den Providerstatus aus und protokolliert keine Secrets.

  </Accordion>
  <Accordion title="Stoppen und Abbrechen">
    - Klicken Sie auf **Stop**. Ausführungen mit einer exakten lokalen Ausführungs-ID rufen `chat.abort` auf; wenn der Status der ausgewählten Sitzung aktive Arbeit meldet, die Control UI jedoch keine lokale Ausführungs-ID hat, ruft sie stattdessen `sessions.abort` auf. Bei nicht globalen Sitzungen verwirft dieser Pfad der ausgewählten Sitzung außerdem in der Warteschlange befindliche Folgeanfragen, damit diese die Arbeit nach dem Stopp nicht erneut starten können.
    - Während eine Ausführung aktiv ist, verwenden normale Folgeanfragen den effektiven `messages.queue`-Modus des Gateways. `steer` schleust sie in den laufenden Turn ein; andere Modi behalten die dauerhafte Warteschlangenzustellung des Browsers bei. Auch eine Ablehnung der Steuerung fällt auf diese Warteschlange zurück. Klicken Sie bei einer Nachricht in der Warteschlange auf **Steer**, um sie manuell einzuschleusen.
    - **Settings → Appearance → Chat → Follow-ups while the agent is working** kann diesen Serverstandard für den aktuellen Browser überschreiben. Die Seite kennzeichnet eine Überschreibung ausdrücklich und bietet **Reset to server default** an. `Steer into the active run` sendet Folgeanfragen sofort, während `Queue until the run ends` sie zurückhält, bis die Ausführung abgeschlossen ist.
    - Geben Sie `/stop` (oder eigenständige Abbruchformulierungen wie `stop`, `stop action`, `stop run`, `stop openclaw`, `please stop`) ein, um außerhalb des regulären Ablaufs abzubrechen.
    - `chat.abort` unterstützt `{ sessionKey }` (ohne `runId`), um alle aktiven Ausführungen für diese Sitzung abzubrechen. Die Control UI verwendet `sessions.abort`, wenn keine lokale Ausführungs-ID vorhanden ist.

  </Accordion>
  <Accordion title="Beibehalten von Teilergebnissen bei Abbruch">
    - Wenn eine Ausführung abgebrochen wird, kann unvollständiger Assistententext weiterhin in der UI angezeigt werden.
    - Das Gateway speichert bei einem Abbruch unvollständigen Assistententext im Transkriptverlauf, sofern gepufferte Ausgabe vorhanden ist.
    - Gespeicherte Einträge enthalten Abbruchmetadaten, sodass Transkriptkonsumenten unvollständige Abbruchausgaben von regulär abgeschlossenen Ausgaben unterscheiden können.

  </Accordion>
</AccordionGroup>

## Verbindungsverlust und erneute Verbindung

Sobald eine Sitzung hergestellt wurde, führt eine unterbrochene Gateway-Verbindung nicht zu Ihrer Abmeldung. Das Dashboard
bleibt sichtbar und zeigt unter der oberen Leiste eine schwebende bernsteinfarbene Pillenanzeige „Gateway-Verbindung verloren — Verbindung wird wiederhergestellt…“, während der Client Wiederholungsversuche mit Backoff
automatisch durchführt (800 ms bis zu 15 s). Live-Aktualisierungen und
Echtzeit-/Sitzungsaktionen werden angehalten, bis die Verbindung wiederhergestellt ist; **Retry now** in der Pillenanzeige erzwingt einen
sofortigen Versuch. Der Chat bleibt bearbeitbar: Gewöhnliche Text- und Anhangsendungen werden im Gateway-/sitzungsbezogenen Browserspeicher
des aktuellen Tabs aufbewahrt, als auf die erneute Verbindung wartend angezeigt und
automatisch gesendet, sobald das Gateway wieder verfügbar ist. Live-Steuerelemente und Slash-Befehle bleiben offline
nicht verfügbar; ausgenommen ist **Stop**, das eine exakte lokale Ausführungs-ID zur Wiederholung des Abbruchaufrufs nach der erneuten Verbindung in die Warteschlange stellen kann. Ein nur auf die Sitzung bezogener Stopp
wird nicht erneut ausgeführt, da in dieser Sitzung möglicherweise neue Arbeit beginnt, bevor die Verbindung zurückkehrt.

Wenn dieser Browser bereits über Anmeldedaten verfügt (ein konfiguriertes Token/Passwort oder ein genehmigtes Geräte-
Token), zeigen das erstmalige Öffnen und Neuladen ein kleines animiertes OpenClaw-Symbol, während die Verbindung
hergestellt wird, statt kurzzeitig die Anmeldesperre einzublenden. Die Anmeldesperre erscheint nur, wenn noch keine Anmeldedaten
gespeichert sind oder wenn das Gateway sie aktiv ablehnt (falsches Token/Passwort, widerrufene Kopplung) —
also in Zuständen, die Ihre Eingabe erfordern, statt bloßes Warten.

## PWA-Installation und Web Push

Die Control UI wird mit einer `manifest.webmanifest` und einem Service Worker ausgeliefert, sodass moderne Browser sie als eigenständige PWA installieren können. Web Push ermöglicht dem Gateway, die installierte PWA mit Benachrichtigungen zu aktivieren, selbst wenn der Tab oder das Browserfenster nicht geöffnet ist.

In der macOS-App zeigt die Seite mit den Benachrichtigungseinstellungen die native Benachrichtigungsberechtigung der App statt Browser-Push an, da die App Benachrichtigungen nativ zustellt.

Wenn die Seite direkt nach einem OpenClaw-Update **Protocol mismatch** anzeigt, öffnen Sie das Dashboard zunächst erneut mit `openclaw dashboard` und führen Sie eine vollständige Aktualisierung durch. Wenn der Fehler weiterhin auftritt, löschen Sie die Websitedaten für den Ursprung des Dashboards oder testen Sie in einem privaten Browserfenster. Ein alter Tab oder Browser-Service-Worker-Cache kann weiterhin ein vor dem Update stammendes Control-UI-Bundle mit dem neueren Gateway ausführen.

| Oberfläche                                         | Funktion                                                                     |
| -------------------------------------------------- | ---------------------------------------------------------------------------- |
| `ui/public/manifest.webmanifest`                   | PWA-Manifest. Browser bieten „Install app“ an, sobald es erreichbar ist.     |
| `ui/public/sw.js`                                  | Service Worker, der `push`-Ereignisse und Klicks auf Benachrichtigungen verarbeitet.           |
| `state/openclaw.sqlite` → `web_push_vapid_keys`    | Automatisch generiertes VAPID-Schlüsselpaar zum Signieren von Web-Push-Nutzdaten.                 |
| `state/openclaw.sqlite` → `web_push_subscriptions` | Gespeicherte Browser-Abonnementendpunkte, Schlüssel und Registrierungszeitstempel. |

Upgrades aus den stillgelegten Speichern `push/vapid-keys.json` und `push/web-push-subscriptions.json` werden von `openclaw doctor --fix` importiert. Stoppen Sie das Gateway, bevor Sie diese Reparatur ausführen, damit ein älterer Prozess während des Imports keinen stillgelegten Zustand neu erstellen kann. Führen Sie die Reparatur nach einem Upgrade durch, bevor Sie Web Push verwenden; Registrierung, Zustellung, Löschung und Schlüsselauflösung verweigern die Ausführung, solange noch eine stillgelegte Quelle oder ein unterbrochener Doctor-Claim vorhanden ist. Die Gateway-Laufzeit liest und schreibt ausschließlich SQLite.

Überschreiben Sie das VAPID-Schlüsselpaar über Umgebungsvariablen im Gateway-Prozess, wenn Sie Schlüssel fest vorgeben möchten (Bereitstellungen auf mehreren Hosts, Secret-Rotation oder Tests):

- `OPENCLAW_VAPID_PUBLIC_KEY`
- `OPENCLAW_VAPID_PRIVATE_KEY`
- `OPENCLAW_VAPID_SUBJECT` (Standardwert: `https://openclaw.ai`)

Die Control UI verwendet diese bereichsbeschränkten Gateway-Methoden, um Browser-Abonnements zu registrieren und zu testen:

- `push.web.vapidPublicKey` ruft den aktiven öffentlichen VAPID-Schlüssel ab.
- `push.web.subscribe` registriert einen `endpoint` sowie `keys.p256dh`/`keys.auth`.
- `push.web.unsubscribe` entfernt einen registrierten Endpunkt.
- `push.web.test` sendet eine Testbenachrichtigung an das Abonnement des Aufrufers.

<Note>
Web Push ist unabhängig vom iOS-APNS-Relay-Pfad (siehe [Konfiguration](/de/gateway/configuration) für Relay-gestützte Push-Benachrichtigungen) und von der Methode `push.test`, die auf die native Mobilgerätekopplung abzielt.
</Note>

## Gehostete Einbettungen

Assistentennachrichten können mit dem Shortcode `[embed ...]` gehostete Webinhalte inline darstellen. Die iframe-Sandbox-Richtlinie wird durch `gateway.controlUi.embedSandbox` gesteuert:

Das Core-Tool [`show_widget`](/de/tools/show-widget) rendert eigenständiges SVG oder HTML direkt aus einem Tool-Aufruf. Der Browser und unterstützte native Chat-Clients melden die Gateway-Fähigkeit `inline-widgets`, und das resultierende Canvas-Dokument bleibt verfügbar, wenn der Chatverlauf neu geladen wird. Discord Activities stellt auf Discord denselben Tool-Namen bereit; Ausführungen, die aus anderen Kanälen stammen, erhalten ihn nicht.

<Tabs>
  <Tab title="strict">
    Deaktiviert die Skriptausführung innerhalb gehosteter Einbettungen.
  </Tab>
  <Tab title="scripts (default)">
    Ermöglicht interaktive Einbettungen unter Beibehaltung der Ursprungsisolation; in der Regel ausreichend für eigenständige Browserspiele und Widgets.
  </Tab>
  <Tab title="trusted">
    Fügt `allow-same-origin` zusätzlich zu `allow-scripts` für Dokumente derselben Website hinzu, die bewusst stärkere Berechtigungen benötigen.
  </Tab>
</Tabs>

```json5
{
  gateway: {
    controlUi: {
      embedSandbox: "scripts",
    },
  },
}
```

<Warning>
Verwenden Sie `trusted` nur, wenn das eingebettete Dokument tatsächlich Same-Origin-Verhalten benötigt. Für die meisten von Agenten generierten Spiele und interaktiven Canvas-Inhalte ist `scripts` die sicherere Wahl.
</Warning>

Absolute externe `http(s)`-Einbettungs-URLs bleiben standardmäßig blockiert. Damit `[embed url="https://..."]` Seiten von Drittanbietern laden kann, setzen Sie `gateway.controlUi.allowExternalEmbedUrls: true`.

## Layout des Chattranskripts

Das Chatprotokoll verwendet einen zentrierten, gut lesbaren Rahmen, der am Eingabebereich ausgerichtet ist. Ausgaben des Assistenten und von Tools bleiben linksbündig, während Ihre eigenen Nachrichten innerhalb dieses Rahmens rechtsbündig bleiben. In Sitzungen mit mehreren Benutzern (beispielsweise einem Gruppenchat, der über ein Kanal-Plugin weitergeleitet wird) werden Nachrichten anderer zugeordneter Teilnehmer linksbündig mit Avatar und Name des Autors sowie einer stabilen Farbe je Identität dargestellt, sodass nur die Nachrichten des angemeldeten Betrachters als „meine“ erscheinen. Wenn zwei oder mehr zugeordnete Teilnehmer vorhanden sind, enthalten Antworten des Assistenten eine kleine Markierung „Antwort an Name“, die den Teilnehmer nennt, dessen Nachricht den Durchlauf ausgelöst hat. Systemeinträge wie die lokale Ausgabe von Slash-Befehlen werden ohne Avatar als zentrierte Hinweiszeilen dargestellt.

## Breite der Chatnachrichten

Benutzer mit breiten Monitoren können die Breite des Protokolls unter **Einstellungen → Chat →
Nachrichtenbreite** überschreiben. Die Einstellung bleibt im lokalen Speicher dieses Browsers erhalten. Unterstützte
Formen umfassen einfache Längen und Prozentwerte wie `960px` oder `82%` sowie
beschränkte Breitenausdrücke mit `min(...)`, `max(...)`, `clamp(...)`, `calc(...)` und
`fit-content(...)`.

## Tailnet-Zugriff (empfohlen)

<Tabs>
  <Tab title="Integriertes Tailscale Serve (bevorzugt)">
    Lassen Sie das Gateway an Loopback gebunden und über Tailscale Serve mit HTTPS als Proxy bereitstellen:

    ```bash
    openclaw gateway --tailscale serve
    ```

    Öffnen Sie `https://<magicdns>/` (oder Ihr konfiguriertes `gateway.controlUi.basePath`).

    Standardmäßig können sich Serve-Anfragen der Control UI bzw. des WebSockets über Tailscale-Identitätsheader (`tailscale-user-login`) authentifizieren, wenn `gateway.auth.allowTailscale` auf `true` gesetzt ist. OpenClaw überprüft die Identität, indem es die Adresse `x-forwarded-for` mit `tailscale whois` auflöst und mit dem Header abgleicht. Diese Anfragen werden nur akzeptiert, wenn sie Loopback mit den `x-forwarded-*`-Headern von Tailscale erreichen. Bei Control-UI-Operatorsitzungen mit Browsergeräteidentität überspringt dieser verifizierte Serve-Pfad außerdem den Umlauf für die Gerätekopplung; Browser ohne Geräteidentität und Verbindungen mit Node-Rolle durchlaufen weiterhin die normalen Geräteprüfungen. Setzen Sie `gateway.auth.allowTailscale: false`, wenn Sie selbst für Serve-Datenverkehr explizite Anmeldedaten in Form eines gemeinsamen Geheimnisses verlangen möchten, und verwenden Sie anschließend `gateway.auth.mode: "token"` oder `"password"`.

    Bei diesem asynchronen Serve-Identitätspfad werden fehlgeschlagene Authentifizierungsversuche für dieselbe Client-IP und denselben Authentifizierungsbereich serialisiert, bevor Schreibvorgänge für die Ratenbegrenzung erfolgen. Bei gleichzeitigen fehlerhaften Wiederholungsversuchen desselben Browsers kann die zweite Anfrage daher `retry later` anzeigen, anstatt dass zwei einfache Nichtübereinstimmungen parallel miteinander konkurrieren.

    <Warning>
    Die tokenlose Serve-Authentifizierung setzt voraus, dass der Gateway-Host vertrauenswürdig ist. Wenn auf diesem Host nicht vertrauenswürdiger lokaler Code ausgeführt werden kann, muss eine Token-/Passwortauthentifizierung verlangt werden.
    </Warning>

  </Tab>
  <Tab title="An Tailnet binden + Token">
    ```bash
    openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
    ```

    Öffnen Sie `http://<tailscale-ip>:18789/` (oder Ihr konfiguriertes `gateway.controlUi.basePath`).

    Fügen Sie das passende gemeinsame Geheimnis in die UI-Einstellungen ein (wird als `connect.params.auth.token` oder `connect.params.auth.password` gesendet).

  </Tab>
</Tabs>

## Unsicheres HTTP

Wenn Sie das Dashboard über unverschlüsseltes HTTP öffnen (`http://<lan-ip>` oder `http://<tailscale-ip>`), wird der Browser in einem **unsicheren Kontext** ausgeführt und blockiert WebCrypto. Standardmäßig **blockiert** OpenClaw Control-UI-Verbindungen ohne Geräteidentität.

Die unterstützte Ausnahme ohne Geräteidentität ist eine erfolgreiche Operatorauthentifizierung der Control UI
über `gateway.auth.mode: "trusted-proxy"`. Es gibt keinen dauerhaften Konfigurationsschalter,
der die Geräteidentität deaktiviert.

**Empfohlene Lösung:** Verwenden Sie HTTPS (Tailscale Serve) oder öffnen Sie die UI lokal unter `https://<magicdns>/` (Serve) oder `http://127.0.0.1:18789/` (auf dem Gateway-Host).

<AccordionGroup>
  <Accordion title="Hinweis zu vertrauenswürdigen Proxys">
    - Eine erfolgreiche Authentifizierung über einen vertrauenswürdigen Proxy kann **Operator**-Control-UI-Sitzungen ohne Geräteidentität zulassen.
    - Dies gilt **nicht** für Control-UI-Sitzungen mit Node-Rolle.
    - Loopback-Reverse-Proxys auf demselben Host erfüllen die Anforderungen für die Authentifizierung über vertrauenswürdige Proxys weiterhin nicht; siehe [Authentifizierung über vertrauenswürdige Proxys](/de/gateway/trusted-proxy-auth).

  </Accordion>
</AccordionGroup>

Anleitungen zur HTTPS-Einrichtung finden Sie unter [Tailscale](/de/gateway/tailscale).

## Content-Security-Policy

Die Control UI wird mit einer strengen `img-src`-Richtlinie ausgeliefert: Zulässig sind nur Ressourcen vom **selben Ursprung**, `data:`-URLs und lokal erzeugte `blob:`-URLs. Entfernte `http(s)`- und protokollrelative Bild-URLs werden vom Browser abgelehnt und lösen keine Netzwerkanfragen aus.

In der Praxis:

- Avatare und Bilder, die unter relativen Pfaden bereitgestellt werden (beispielsweise `/avatars/<id>`), werden weiterhin dargestellt. Dies gilt auch für authentifizierte Avatar-Routen, welche die UI abruft und in lokale `blob:`-URLs umwandelt.
- Inline-`data:image/...`-URLs werden weiterhin dargestellt.
- Von der Control UI erstellte lokale `blob:`-URLs werden weiterhin dargestellt.
- Avatare für GitHub-Linkvorschauen werden vom Gateway vom festen Avatar-Host von GitHub abgerufen und als begrenzte `data:`-URLs zurückgegeben; der Browser des Operators kontaktiert den entfernten Avatar-Host nie.
- Von Kanalmetadaten ausgegebene entfernte Avatar-URLs werden von den Avatar-Hilfsfunktionen der Control UI entfernt und durch das integrierte Logo bzw. Abzeichen ersetzt. Somit kann ein kompromittierter oder bösartiger Kanal keine beliebigen Abrufe entfernter Bilder durch den Browser eines Operators erzwingen.

Dies ist immer aktiviert und nicht konfigurierbar.

## Authentifizierung der Avatar-Route

Wenn die Gateway-Authentifizierung konfiguriert ist, erfordert der Avatar-Endpunkt der Control UI dasselbe Gateway-Token wie der Rest der API:

- `GET /avatar/<agentId>` gibt das Avatarbild nur an authentifizierte Aufrufer zurück. `GET /avatar/<agentId>?meta=1` gibt die Avatarmetadaten nach derselben Regel zurück.
- Nicht authentifizierte Anfragen an eine der beiden Routen werden abgelehnt (entsprechend der benachbarten Route für Assistentenmedien), sodass die Avatar-Route auf ansonsten geschützten Hosts keine Agentenidentität preisgeben kann.
- Die Control UI leitet das Gateway-Token beim Abrufen von Avataren als Bearer-Header weiter und verwendet authentifizierte Blob-URLs, damit das Bild weiterhin in Dashboards dargestellt wird.

Wenn Sie die Gateway-Authentifizierung deaktivieren (auf gemeinsam genutzten Hosts nicht empfohlen), wird entsprechend dem übrigen Gateway auch die Avatar-Route nicht authentifiziert.

## Authentifizierung der Route für Assistentenmedien

Wenn die Gateway-Authentifizierung konfiguriert ist, verwenden lokale Medienvorschauen des Assistenten eine zweistufige Route:

- `GET /__openclaw__/assistant-media?meta=1&source=<path>` erfordert die normale Operatorauthentifizierung der Control UI; beim Prüfen der Verfügbarkeit sendet der Browser das Gateway-Token als Bearer-Header.
- Erfolgreiche Metadatenantworten enthalten ein kurzlebiges `mediaTicket`, das auf genau diesen Quellpfad beschränkt ist.
- Vom Browser dargestellte URLs für Bilder, Audio, Video und Dokumente verwenden `mediaTicket=<ticket>` anstelle des aktiven Gateway-Tokens oder Passworts. Das Ticket läuft schnell ab und kann keine andere Quelle autorisieren.

Dadurch bleibt die Mediendarstellung mit nativen Medienelementen des Browsers kompatibel, ohne wiederverwendbare Gateway-Anmeldedaten in sichtbaren Medien-URLs offenzulegen.

## Genehmigungslinks

Benachrichtigungen über Operatorgenehmigungen können per Deep-Link auf ein eigenständiges Genehmigungsdokument verweisen, das unter dem reservierten Namensraum `${controlUiBasePath}/approve/{approvalId}` bereitgestellt wird (beispielsweise `/approve/<approvalId>` oder `/openclaw/approve/<approvalId>` mit einem konfigurierten Basispfad). Die URL bleibt für die Lebensdauer der Genehmigung stabil und kann sicher zwischen Ihren eigenen Geräten weitergeleitet werden: Sie identifiziert die Genehmigung, autorisiert sie jedoch niemals.

- Der aus einem Segment bestehende Namensraum `/approve/<approvalId>` wird vom Gateway für **alle** HTTP-Methoden vor den HTTP-Routen von Plugins reserviert, sodass eine Plugin-Route ein Genehmigungsdokument niemals überlagern oder abfangen kann.
- Das Öffnen eines Genehmigungsdokuments erfordert dieselbe Gateway-Authentifizierung wie der Rest der Control UI (Token/Passwort, Tailscale-Serve-Identität oder Identität über einen vertrauenswürdigen Proxy); Anmeldedaten sind niemals Teil der Genehmigungs-URL.
- Wenn die Bereitstellung der Control UI deaktiviert ist, geben Anfragen an den Namensraum `404` zurück, anstatt an Plugin-Handler weitergereicht zu werden.
- Die Anmeldung bei einem Genehmigungsdokument gilt nur vorübergehend für diese Seite: Sie überschreibt weder die Gateway-Auswahl noch die Einstellungen, die von der vollständigen Control UI im selben Browser gespeichert wurden.

Das Gateway stellt statische Dateien aus `dist/control-ui` bereit:

```bash
pnpm ui:build
```

Optionale absolute Basis (feste Ressourcen-URLs):

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

Lokale Entwicklung (separater Entwicklungsserver):

```bash
pnpm ui:dev
```

Richten Sie die UI anschließend auf die WebSocket-URL Ihres Gateways (z. B. `ws://127.0.0.1:18789`).

## Leere Control-UI-Seite

Wenn der Browser ein leeres Dashboard lädt und die Entwicklertools keinen hilfreichen Fehler anzeigen, hat möglicherweise eine Erweiterung oder ein früh ausgeführtes Inhaltsskript die Auswertung der JavaScript-Modulanwendung verhindert. Die statische Seite enthält ein einfaches HTML-Wiederherstellungsfenster, das erscheint, wenn `<openclaw-app>` nach dem Start nicht registriert wurde.

Verwenden Sie nach einer Änderung der Browserumgebung die Aktion **Erneut versuchen** des Fensters oder laden Sie die Seite nach diesen Prüfungen manuell neu:

- Deaktivieren Sie Erweiterungen, die Code in alle Seiten einschleusen, insbesondere Erweiterungen mit `<all_urls>`-Inhaltsskripten.
- Probieren Sie ein privates Fenster, ein sauberes Browserprofil oder einen anderen Browser aus.
- Lassen Sie das Gateway laufen und überprüfen Sie nach der Browseränderung dieselbe Dashboard-URL.

## Debugging/Tests: Entwicklungsserver + entferntes Gateway

Die Control UI besteht aus statischen Dateien; das WebSocket-Ziel ist konfigurierbar und kann sich vom HTTP-Ursprung unterscheiden. Dies ist praktisch, wenn Sie den Vite-Entwicklungsserver lokal verwenden möchten, das Gateway jedoch an einem anderen Ort ausgeführt wird.

<Steps>
  <Step title="UI-Entwicklungsserver starten">
    ```bash
    pnpm ui:dev
    ```
  </Step>
  <Step title="Mit gatewayUrl öffnen">
    ```text
    http://localhost:5173/?gatewayUrl=ws%3A%2F%2F<gateway-host>%3A18789
    ```

    Optionale einmalige Authentifizierung (falls erforderlich):

    ```text
    http://localhost:5173/?gatewayUrl=wss%3A%2F%2F<gateway-host>%3A18789#token=<gateway-token>
    ```

  </Step>
</Steps>

<AccordionGroup>
  <Accordion title="Hinweise">
    - `gatewayUrl` wird nach dem Laden in localStorage gespeichert und aus der URL entfernt.
    - Wenn Sie über `gatewayUrl` einen vollständigen `ws://`- oder `wss://`-Endpunkt übergeben, prozentkodieren Sie den Wert für die URL, damit der Browser die Abfragezeichenfolge korrekt analysiert.
    - `token` sollte nach Möglichkeit über das URL-Fragment (`#token=...`) übergeben werden. Fragmente werden nicht an den Server gesendet, wodurch eine Offenlegung über Anfrageprotokolle und den Referer vermieden wird. Veraltete `?token=`-Abfrageparameter werden aus Kompatibilitätsgründen weiterhin einmalig importiert, jedoch nur als Fallback, und unmittelbar nach dem Bootstrap entfernt.
    - `password` wird nur im Arbeitsspeicher vorgehalten.
    - Wenn `gatewayUrl` festgelegt ist, greift die UI nicht auf Anmeldedaten aus der Konfiguration oder Umgebung zurück. Geben Sie `token` (oder `password`) explizit an; fehlende explizite Anmeldedaten gelten als Fehler.
    - Verwenden Sie `wss://`, wenn sich das Gateway hinter TLS befindet (Tailscale Serve, HTTPS-Proxy usw.).
    - `gatewayUrl` wird nur in einem Fenster der obersten Ebene akzeptiert (nicht eingebettet), um Clickjacking zu verhindern.
    - Öffentliche Control-UI-Bereitstellungen außerhalb von Loopback müssen `gateway.controlUi.allowedOrigins` explizit festlegen (vollständige Ursprünge). Private LAN-/Tailnet-Ladevorgänge desselben Ursprungs von Loopback-, RFC1918-/Link-Local-, `.local`-, `.ts.net`- oder Tailscale-CGNAT-Hosts werden akzeptiert, ohne den Host-Header-Fallback zu aktivieren.
    - Beim Start des Gateways können lokale Ursprünge wie `http://localhost:<port>` und `http://127.0.0.1:<port>` anhand der effektiven Runtime-Bindung und des Ports vorbelegt werden; entfernte Browserursprünge benötigen jedoch weiterhin explizite Einträge.
    - Verwenden Sie `gateway.controlUi.allowedOrigins: ["*"]` ausschließlich für streng kontrollierte lokale Tests; dies bedeutet, jeden Browserursprung zuzulassen, nicht „mit dem jeweils von mir verwendeten Host abgleichen“.
    - `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` aktiviert den Fallback-Modus für den Host-Header-Ursprung, stellt jedoch einen gefährlichen Sicherheitsmodus dar.

  </Accordion>
</AccordionGroup>

```json5
{
  gateway: {
    controlUi: {
      allowedOrigins: ["http://localhost:5173"],
    },
  },
}
```

Details zur Einrichtung des Fernzugriffs: [Fernzugriff](/de/gateway/remote).

## Verwandte Themen

- [Dashboard](/de/web/dashboard) — Gateway-Dashboard
- [Integritätsprüfungen](/de/gateway/health) — Überwachung des Gateway-Zustands
- [TUI](/de/web/tui) — Terminal-Benutzeroberfläche
- [WebChat](/de/web/webchat) — browserbasierte Chat-Oberfläche
