---
read_when: You want agent sessions to run on ephemeral cloud machines instead of the Gateway host, or you are configuring cloudWorkers profiles.
sidebarTitle: Cloud Workers
status: active
summary: 'Sitzungen an temporäre Cloud-Maschinen weiterleiten: Bereitstellung, Worker-Laufzeit, weitergeleitete Inferenz und Streaming-Ergebnisse'
title: Cloud-Worker
x-i18n:
    generated_at: "2026-07-24T07:47:26Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 5620be5957a20019d4687b3ec935ec1116fdef6ea05e42ab766508d2b54322a2
    source_path: gateway/cloud-workers.md
    workflow: 16
---

Cloud-Worker ermöglichen es einer Sitzung, ihre Agent-Schleife auf einer kurzlebigen Cloud-Maschine auszuführen, während alles zur Sitzung dort bleibt, wo es schon immer war: in der Seitenleiste sichtbar, live gestreamt und mit dem Transkript im Besitz des Gateways. Das Gateway mietet eine Maschine, installiert darauf eine gepinnte Kopie von OpenClaw, synchronisiert den Workspace der Sitzung dorthin und übergibt die Turn-Schleife an einen eingeschränkten `openclaw worker`-Prozess. Modellaufrufe werden über das Gateway zurückgeleitet, sodass Provider-Anmeldedaten Ihre Maschine nie verlassen. Prompt-Caching funktioniert weiterhin, da der Provider einen einzigen kontinuierlichen Datenstrom sieht.

Wenn die Arbeit abgeschlossen ist (oder die Maschine ausfällt), wird die Maschine verworfen. Der dauerhafte Zustand – Transkript, Workspace-Commits, Platzierungsdatensätze – verbleibt beim Gateway.

<Note>
Cloud-Worker sind optional und bleiben unsichtbar, bis Sie ein Profil konfigurieren. Bei nicht konfigurierten Installationen werden keine neuen RPCs, Konfigurationen oder UI-Elemente angezeigt.
</Note>

## Was wo ausgeführt wird

| Aspekt                                                  | Ort                                                                              |
| ------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Agent-Schleife + Tools (`exec`, `read`, `write`, `edit`, …) | Cloud-Worker-Maschine                                                            |
| Modellinferenz und Provider-Anmeldedaten                | Gateway (über `{provider, model}`-Referenz weitergeleitet)                        |
| Transkript (dauerhaft, Sitzungsspeicher)                | Gateway                                                                          |
| Live-Streaming in die Seitenleiste                      | Gateway-Fan-out, gespeist durch den wiederabspielbaren Ereignisstrom des Workers |
| Git-Verlauf des Workspace                               | Ohne Anmeldedaten auf der Maschine erstellt; das Gateway übernimmt Commits und ist für Push/PR verantwortlich |

Die Maschine benötigt außer `sshd` keine eingehenden Ports: Das Gateway stellt über gepinntes SSH eine ausgehende Verbindung her, und ein Reverse-Tunnel führt den WebSocket des Workers zurück. Der mitgelieferte Crabbox-Provider erzwingt die öffentliche SSH-Route und deaktiviert die verwaltete Tailscale-Registrierung. Ausgehender Internetzugriff wird durch die Provider-Richtlinie bestimmt; das standardmäßige AWS-Profil kann auf das Internet zugreifen, sofern Sie sein Netzwerk oder seine Sicherheitsgruppe nicht einschränken.

## Anforderungen

- Ein Worker-Provider-Plugin. Das mitgelieferte `crabbox`-Plugin steuert die [Crabbox](https://github.com/openclaw/crabbox)-CLI, die Mietvorgänge über Cloud-Backends hinweg vermittelt (AWS, Hetzner und andere). Die Binärdatei `crabbox` muss sich in `PATH` befinden (oder setzen Sie `settings.binary`), und die Provider-Anmeldedaten müssen bereits konfiguriert sein. Die AWS-Zulassung erfordert Crabbox 0.38.1 oder neuer.
- Bei Crabbox-AWS-Workern muss der effektive Wert von `aws.instanceProfile` leer sein. Der Provider prüft `crabbox config show --json` vor der Zuweisung und verlangt anschließend, dass `crabbox inspect --json` den Wert `providerMetadata.instanceProfileAttached: false` aus EC2-`DescribeInstances` meldet. Mietvorgänge mit einer Instanzrolle oder ohne maßgebliche Metadaten werden beendet und abgelehnt.
- Node.js auf der gemieteten Maschine. Unveränderte Cloud-Images enthalten es normalerweise nicht – installieren Sie es über den `setup`-Befehl des Profils.
- Eine Sitzung mit einem sitzungseigenen verwalteten Worktree (erstellen Sie einen mit `worktree: true`). Beim Dispatch wird der Inhalt dieses Worktrees übertragen; gewöhnliche Verzeichnisse werden als Manifest-Spiegel synchronisiert.

## Konfiguration

Fügen Sie unter `cloudWorkers.profiles` in `openclaw.json` ein Profil hinzu:

```json
{
  "cloudWorkers": {
    "profiles": {
      "aws": {
        "provider": "crabbox",
        "install": "bundle",
        "settings": {
          "provider": "aws",
          "class": "standard",
          "ttl": "8h",
          "idleTimeout": "45m",
          "setup": "test -x /usr/bin/node || (curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash - && sudo apt-get install -y nodejs)"
        }
      }
    }
  }
}
```

Profilfelder:

| Schlüssel  | Bedeutung                                                                                                                                                                                                                                      |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `provider` | Von einem Plugin registrierte Worker-Provider-ID (`crabbox` für das mitgelieferte Plugin).                                                                                                                                             |
| `install`  | `bundle` (Standard) überträgt den Build des laufenden Gateways; `npm` installiert die exakte veröffentlichte Gateway-Version mit gepinnter Integrität. `npm` setzt voraus, dass das Gateway aus einer paketierten Veröffentlichung ausgeführt wird. |
| `settings` | Provider-eigenes JSON. Für Crabbox: `provider` (Backend), `class` (Maschinenklasse), `ttl`, `idleTimeout` (Go-Zeitspannen), optional `setup` und absoluter `binary`-Pfad. OpenClaw erzwingt öffentliches SSH und deaktiviert verwaltetes Tailscale für diese Mietvorgänge. |
| `lifetime` | Optional gespeicherte Richtlinie (`idleTimeoutMinutes`, `maxLifetimeMinutes`).                                                                                                                                                                      |

### Der Einrichtungsbefehl

`settings.setup` wird auf der gemieteten Maschine ausgeführt, nachdem sie über SSH erreichbar ist und bevor OpenClaw installiert wird. Er wird bei **jedem** Bereitstellungsversuch ausgeführt (einschließlich Wiederholungen nach einem unterbrochenen Dispatch), daher muss er idempotent sein – sichern Sie Installationen wie im Beispiel mit einer `command -v`-/`test -x`-Prüfung ab. Wenn die Einrichtung fehlschlägt, beendet der Provider den Mietvorgang und der Dispatch schlägt nach dem Fail-Closed-Prinzip fehl; es bleibt keine halb konfigurierte Maschine aktiv.

### Installationskanäle

- **`bundle`** paketiert die `dist` des laufenden Gateways, eine bereinigte `package.json` sowie alle Workspace-Pakete, auf die der Build verweist; alles ist durch einen Inhalts-Hash abgedeckt. Die Maschine prüft das unveränderte Bundle anhand dieses Hashes und installiert anschließend die produktiven npm-Abhängigkeiten (Skripte deaktiviert). So führen Sie einen Entwicklungs-Build auf einem Worker aus.
- **`npm`** weist nach, dass die Veröffentlichung in der öffentlichen Registry vorhanden ist, pinnt ihre SHA-512-Integrität und installiert `openclaw@<version>` exakt passend zum Gateway.

## Eine Sitzung versenden

Öffnen Sie in der Control UI **New Session**, wählen Sie einen Agenten, dessen konfigurierte Runtime OpenClaw ist, wählen Sie im Menü **Where** ein konfiguriertes Ziel des Typs **Cloud · profile** aus und starten Sie die Aufgabe. Die Cloud-Auswahl aktiviert den erforderlichen verwalteten Worktree automatisch; das Gateway erstellt die Sitzung, schließt den Dispatch ab und sendet erst danach den ersten Turn. Das Server-Badge in der Sitzungsseitenleiste zeigt den dauerhaften Platzierungsstatus an. Cloud-Ziele werden für externe CLI-Sitzungskataloge nicht angeboten.

Der entsprechende RPC-Ablauf lautet:

Erstellen Sie eine Sitzung mit einem verwalteten Worktree und versenden Sie sie anschließend (der RPC erfordert `operator.admin` und ist nur vorhanden, wenn Profile konfiguriert sind):

Cloud-Worker führen die OpenClaw-Agent-Runtime aus. Wählen Sie ein `openai/*`- oder anderes Modell, das zu dieser Runtime aufgelöst wird; für eine externe CLI-Runtime wie `claude-cli` konfigurierte Sitzungen können nicht versendet werden.

```bash
openclaw gateway call sessions.create \
  --params '{"key":"agent:main:big-refactor","worktree":true,"cwd":"/path/to/repo","worktreeName":"big-refactor"}'

openclaw gateway call sessions.dispatch \
  --timeout 1500000 \
  --params '{"key":"agent:main:big-refactor","profileId":"aws"}'
```

`sessions.dispatch` sperrt die lokale Turn-Annahme, lässt aktive Arbeit auslaufen, stellt den Mietvorgang bereit, führt die Einrichtung aus, bootstrapt OpenClaw, synchronisiert den Workspace und kehrt zurück, sobald die Platzierung den Status `active` unter Worker-Verantwortung erreicht. Planen Sie für den ersten Dispatch mehrere Minuten ein; Mietvorgänge und Installationen werden zwischengespeichert, sofern der Provider dies unterstützt. Danach interagieren Sie wie gewohnt mit der Sitzung – Turns werden automatisch an den Worker weitergeleitet.

Abgeschlossene Worker-Turns gleichen geeignete Workspace-Dateien innerhalb der Größenbegrenzung zurück in den verwalteten Worktree der Sitzung ab, bevor der Turn-Anspruch freigegeben wird. Das abschließende Worker-Ereignis erstellt eine dauerhafte Sperre für ausstehende Ergebnisse, bevor es bestätigt wird. Anschließend stellt das Gateway das vollständige Cloud-Ergebnis als Git-Ref unter `refs/openclaw/worker-results/` bereit, bevor es dieses anwendet, sodass die Cloud-Version auch dann wiederherstellbar bleibt, wenn das Gateway während der Anwendung beendet wird. Workspace-Ergebnisse verwenden die Git-Dateisemantik: reguläre Dateien, Ausführbarkeits-Bits, symbolische Links, Hinzufügungen, Änderungen und Löschungen bleiben erhalten, leere Verzeichnisse und andere Verzeichnismodi dagegen nicht. Die resultierenden Dateiänderungen verbleiben zur normalen Überprüfung und zum Commit im verwalteten Worktree.

Bei der Anwendung wird das Manifest zum Dispatch-Zeitpunkt als Merge-Basis verwendet. Ausschließlich in der Cloud vorgenommene Änderungen werden angewendet, ausschließlich lokal vorgenommene Änderungen bleiben bestehen, und für auf beiden Seiten geänderte Pfade gilt bei einem Drei-Wege-Merge die Richtlinie, die lokale Version beizubehalten. Ein Turn mit Konflikten wird dennoch abgeschlossen: Das Transkript meldet die begrenzte Pfadzusammenfassung und den bereitgestellten Ergebnis-Ref, die Platzierung stellt denselben Konflikt für die Control UI bereit, und konfliktfreie Cloud-Änderungen bleiben angewendet. Der Hinweis enthält `git show <ref>:<path>` zur Prüfung einer vorhandenen Cloud-Datei sowie einen `git checkout <ref> -- <path>`-Befehl mit einem literalen Pathspec auf oberster Ebene, um sie aus einem beliebigen Workspace-Verzeichnis zu übernehmen. Führen Sie die Befehle in Bash oder zsh aus (Git Bash unter Windows). Wenn die Prüfung meldet, dass der Pfad nicht vorhanden ist, wurde er durch das Cloud-Ergebnis gelöscht; überprüfen und entfernen Sie den beibehaltenen lokalen Pfad manuell. Wenn Checkout eine Datei-/Verzeichnisblockade meldet, verschieben oder entfernen Sie den blockierenden lokalen Pfad und versuchen Sie es erneut. Wenn der bereitgestellte Ref selbst nicht mehr vorhanden ist, behandeln Sie den Hinweis als veraltet und ändern Sie den lokalen Pfad nicht. Bereitgestellte Refs mit Konflikten bleiben verfügbar, nachdem die normale Turn-Sperre freigegeben wurde; ein späteres konfliktfreies Ergebnis entfernt den Hinweis und zieht den alten Ref zurück, während die explizite Entfernung der Sperre die endgültige Bereinigungsgrenze darstellt.

Während ein gesperrtes Ergebnis noch abgeglichen wird, wartet ein neuer Turn bis zu 15 Sekunden darauf, dass der vorherige Anspruch freigegeben wird. Ist er danach weiterhin belegt, schlägt der Turn mit einer handlungsorientierten Meldung „Das Workspace-Ergebnis des vorherigen Cloud-Turns wird noch abgeglichen“ fehl und kann kurz darauf erneut versucht werden. Nach einem Neustart erkennt die Wiederherstellung ausstehende und bereitgestellte Ergebnisse vor der Bereinigung veralteter Ansprüche, schließt deren lokale Anwendung ab oder versucht sie erneut und fordert verwaiste Umgebungen erst zurück, nachdem das Ergebnis gesichert wurde. Das begrenzte SQLite-Rollback-Journal macht eine unterbrochene Dateisystemanwendung wiederherstellbar, ohne bereits akzeptierte Mutationen erneut auszuführen.

Wenn die Arbeit abgeschlossen ist und kein Turn ausgeführt wird, öffnen Sie das Sitzungsmenü und wählen Sie **Stop cloud worker…**. Das Gateway führt einen letzten Workspace-Abgleich durch, bevor es die Umgebung zerstört. Eine Platzierung, die sich bereits in `draining` oder `reconciling` befindet, schließt den Abbau ab; warten Sie, bis ihr Badge `reclaimed` anzeigt, bevor Sie die Sitzung löschen.

Bei einem defekten oder außer Kontrolle geratenen angehängten Worker kann ein Operator als letztes Mittel `environments.destroy` mit `{ "force": true }` aufrufen. Ein erzwungener Abbau markiert die Platzierung dauerhaft als fehlgeschlagen und verwirft alle nicht abgeglichenen Remote-Ergebnisse, bevor die Umgebung zerstört wird.

Der entsprechende administrative RPC lautet:

```bash
openclaw gateway call sessions.reclaim \
  --timeout 600000 \
  --params '{"key":"agent:main:big-refactor"}'
```

Die Platzierung durchläuft eine dauerhafte Zustandsmaschine (`local → requested → provisioning → syncing → starting → active`), sodass bei einem Neustart des Gateways während des Dispatch-Vorgangs eine Abstimmung erfolgt, anstatt Maschinen zurückzulassen. Bei einem fehlgeschlagenen Modelldurchlauf bleibt die aktive Platzierung für einen erneuten Versuch verfügbar. Bei Konflikten mit Workspace-Pfaden wird die lokale Version beibehalten, der übrige Teil des Cloud-Ergebnisses angewendet und die bereitgestellte Cloud-Referenz zur Überprüfung bewahrt; bei anderen Abstimmungs- oder Lebenszyklusfehlern bleiben die dauerhafte Wiederherstellungssperre und das Ende der Diagnoseausgabe erhalten, bis die Wiederherstellung einen erneuten Versuch sicher durchführen oder die Umgebung zurückfordern kann.

## Sicherheitsmodell

- **Geschlossener Worker-Eingang.** Worker kommunizieren über ein dediziertes Protokoll auf dem getunnelten Socket mit einer geschlossenen Methoden-Zulassungsliste — ein Worker kann keine Operator-RPCs aufrufen.
- **Vom Gateway verwaltete Tool-Berechtigung.** Vor jedem Durchlauf projiziert das Gateway die aktuellen Richtlinien für Profil, Provider, Agent, Gruppe, Absender, Sandbox, Delegierung, Vererbung und Laufzeitbegrenzungen auf den festen Katalog der Coding-Tools des Workers. Der Start-Umschlag enthält nur diese endgültige Teilmenge mit geschlossenem Vokabular. Explizit begrenzte geplante Durchläufe verwenden ihren vertrauenswürdigen Kontext der Eigentümergruppe erneut, ohne diese Identität an die Box zu senden oder eine neue Absenderüberlagerung anzuwenden. Tools außerhalb des Worker-Katalogs bleiben nicht verfügbar; bei einem leeren Ergebnis erfolgt die Ausführung ohne Tools.
- **Ausgestellte Anmeldedaten, im Ruhezustand gehasht.** Jeder Dispatch stellt Worker-Anmeldedaten aus; das Gateway speichert nur deren Hash. Die Rotation der Anmeldedaten und die Begrenzung anhand der Eigentümer-Epoche gewährleisten höchstens einen aktiven Eigentümer pro Sitzung — ein veralteter Worker, der die Verbindung wiederherstellt, wird abgegrenzt und niemals zusammengeführt.
- **Hostschlüssel-Pinning.** Der Provider muss bei der Bereitstellung den SSH-Hostschlüssel der Box bereitstellen; der Bootstrap stellt die Verbindung mit striktem Pinning her und schlägt ohne diesen sicher geschlossen fehl.
- **Keine dauerhaft hinterlegten Modell-, Forge- oder Cloud-Anmeldedaten auf der Box.** Die Modellauthentifizierung verbleibt auf dem Gateway (die Inferenz wird über die Referenz `{provider, model}` übertragen), Git-Commits im Workspace werden ohne Forge-Anmeldedaten erstellt und die Metadaten von Crabbox-AWS-Leases werden vor der Einrichtung verbindlich auf eine Instanzrolle geprüft. Auch Einrichtungsbefehle müssen frei von Anmeldedaten bleiben.
- **Vom Provider verwalteter ausgehender Datenverkehr.** Der Reverse-Tunnel beseitigt für OpenClaw die Notwendigkeit eines direkten Modellzugriffs, OpenClaw schreibt jedoch keine Provider-Firewalls um. Beschränken Sie den ausgehenden Datenverkehr beim Worker-Provider, wenn die Aufgabe dies erfordert.
- **Dauerhafte, genau einmal gespeicherte Transkripte.** Der Worker schreibt Transkript-Batches über ein Compare-and-Swap-Protokoll gegen das Blatt der Sitzung fest; eine veraltete Basis stoppt den Durchlauf sicher, anstatt kostenpflichtige Ausgaben zu duplizieren oder neu zu basieren.

## Fehlerbehebung

- **`sessions.dispatch` ist eine unbekannte Methode** — es sind keine `cloudWorkers.profiles` konfiguriert oder dem Aufrufer fehlt `operator.admin`.
- **„Cloud-Worker-Durchläufe erfordern die OpenClaw-Laufzeit“** — wählen Sie ein Modell, dessen konfigurierte Laufzeit OpenClaw ist. Externe CLI-Laufzeiten wie `claude-cli` unterstützen keine Worker-Inferenz.
- **„Der Worker-Bootstrap erfordert Node.js auf dem geleasten Host“** — fügen Sie `settings.setup` eine Node-Installation hinzu (siehe oben).
- **Die Bescheinigung der AWS-Instanzrolle schlägt fehl** — entfernen Sie `aws.instanceProfile` (und `CRABBOX_AWS_INSTANCE_PROFILE`, falls festgelegt). Installieren Sie Crabbox 0.38.1 oder neuer; ältere Binärdateien stellen den verbindlichen Vertrag `providerMetadata.instanceProfileAttached` nicht bereit, der für die AWS-Zulassung erforderlich ist.
- **Der Dispatch schlägt mit einem Provider-Fehler fehl** — der Platzierungsdatensatz und `environments.list` bewahren den letzten Fehler einschließlich des Endes der stderr-Ausgabe von Einrichtung und Bootstrap auf. Boxen werden bei einem Fehler zerstört, daher ist diese Ausgabe die wichtigste forensische Informationsquelle.
- **Client-Zeitüberschreitung während des Dispatch-Vorgangs** — `openclaw gateway call` verwendet standardmäßig eine Zeitüberschreitung von 10s; bemessen Sie `--timeout` großzügig (der Dispatch läuft in jedem Fall serverseitig weiter, und ein erneuter Versuch während der Bereitstellung wird mit `session cannot dispatch from placement provisioning` abgelehnt).
- **Worker nach einem Upgrade von einer 2026.7.2-Betaversion zurückgefordert** — diese Betaversionen verwendeten den älteren Worker-Startvertrag. Bei einem Neustart zerstört OpenClaw einen inaktiven inkompatiblen Worker, behält die Sitzung und den Workspace bei, markiert die Platzierung als zurückgefordert und stellt beim nächsten Dispatch oder Durchlauf einen aktuellen Worker bereit. Ein Beta-Worker, dessen noch laufender Start unterbrochen wird, wird nach der Bereinigung als fehlgeschlagen markiert; wiederholen Sie den Dispatch, um ihn mit dem aktuellen Vertrag bereitzustellen.
- **Hinweis zu einem Cloud-Workspace-Konflikt** — der Durchlauf wurde abgeschlossen und hat für jeden aufgeführten Pfad die lokale Version beibehalten. Verwenden Sie die Befehle für die bereitgestellte Referenz im Hinweis, um die Cloud-Version zu prüfen oder zu übernehmen; für die konfliktfreien Änderungen ist kein erneuter Versuch erforderlich, da sie bereits angewendet wurden.
- **„Das Workspace-Ergebnis des vorherigen Cloud-Durchlaufs wird noch abgestimmt“** — das Gateway wartete kurz auf die dauerhafte Sperre des vorherigen Ergebnisses und konnte den Anspruch auf die Sitzung nicht erwerben. Warten Sie, bis die Abstimmung abgeschlossen ist, und wiederholen Sie dann den Durchlauf; ein Neustart des Gateways ist sicher, da die Wiederherstellung bereitgestellte Ergebnisse bewahrt, bevor ein nicht mehr aktiver Worker zurückgefordert wird.
- **Lease-Verwaltung** — `crabbox list --provider <backend>` zeigt aktive Leases an; `crabbox stop --provider <backend> --id <lease>` gibt eines manuell frei. Inaktive Leases laufen gemäß `idleTimeout` des Profils ab.

## Verwandte Themen

- [Sandboxing](/de/gateway/sandboxing) — Verringerung des Schadensradius bei lokaler Tool-Ausführung
- [Sitzungs-CLI](/de/cli/sessions) — Überprüfen gespeicherter Sitzungen
- [Konfigurationsreferenz](/de/gateway/configuration-reference)
