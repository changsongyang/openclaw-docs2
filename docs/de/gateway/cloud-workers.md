---
read_when: You want agent sessions to run on ephemeral cloud machines instead of the Gateway host, or you are configuring cloudWorkers profiles.
sidebarTitle: Cloud Workers
status: active
summary: 'Sitzungen an temporäre Cloud-Maschinen verteilen: Bereitstellung, Worker-Laufzeit, weitergeleitete Inferenz und Streaming-Ergebnisse'
title: Cloud-Worker
x-i18n:
    generated_at: "2026-07-26T18:26:26Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 5620be5957a20019d4687b3ec935ec1116fdef6ea05e42ab766508d2b54322a2
    source_path: gateway/cloud-workers.md
    workflow: 16
---

Cloud-Worker ermöglichen es einer Sitzung, ihre Agentenschleife auf einer kurzlebigen Cloud-Maschine auszuführen, während alles rund um die Sitzung dort bleibt, wo es schon immer war: in der Seitenleiste sichtbar, live gestreamt und mit dem Gateway als Eigentümer des Transkripts. Das Gateway least eine Box, installiert darauf eine gepinnte Kopie von OpenClaw, synchronisiert den Arbeitsbereich der Sitzung dorthin und übergibt die Ausführungsschleife an einen eingeschränkten `openclaw worker`-Prozess. Modellaufrufe werden über das Gateway zurückgeleitet, sodass Provider-Anmeldedaten niemals Ihre Maschine verlassen. Prompt-Caching funktioniert weiterhin, weil der Provider einen einzigen kontinuierlichen Datenstrom sieht.

Wenn die Arbeit abgeschlossen ist (oder die Box ausfällt), wird die Maschine verworfen. Der dauerhafte Zustand – Transkript, Arbeitsbereich-Commits, Platzierungsdatensätze – verbleibt beim Gateway.

<Note>
Cloud-Worker sind optional und unsichtbar, bis Sie ein Profil konfigurieren. Bei nicht konfigurierten Installationen sind keine neuen RPCs, Konfigurationen oder UI-Elemente sichtbar.
</Note>

## Was wo ausgeführt wird

| Bereich                                                 | Ort                                                                              |
| ------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Agentenschleife + Tools (`exec`, `read`, `write`, `edit`, …) | Cloud-Worker-Box                                                                 |
| Modellinferenz und Provider-Anmeldedaten                | Gateway (über `{provider, model}`-Referenz weitergeleitet)                        |
| Transkript (dauerhaft, Sitzungsspeicher)                | Gateway                                                                          |
| Live-Streaming in die Seitenleiste                      | Gateway-Fan-out, gespeist vom wiederholbaren Ereignisstrom des Workers            |
| Git-Verlauf des Arbeitsbereichs                         | Wird auf der Box ohne Anmeldedaten erstellt; das Gateway übernimmt Commits und Push/PR |

Die Box benötigt außer `sshd` keine eingehenden Ports: Das Gateway stellt über gepinntes SSH eine ausgehende Verbindung her, und ein Reverse-Tunnel führt die WebSocket-Verbindung des Workers zurück. Der mitgelieferte Crabbox-Provider erzwingt die öffentliche SSH-Route und deaktiviert die verwaltete Tailscale-Registrierung. Ausgehender Internetzugriff unterliegt den Richtlinien des Providers; das standardmäßige AWS-Profil kann auf das Internet zugreifen, sofern Sie dessen Netzwerk oder Sicherheitsgruppe nicht einschränken.

## Anforderungen

- Ein Worker-Provider-Plugin. Das mitgelieferte `crabbox`-Plugin steuert die [Crabbox](https://github.com/openclaw/crabbox)-CLI, die Leases über verschiedene Cloud-Backends hinweg vermittelt (AWS, Hetzner und andere). Die `crabbox`-Binärdatei muss sich in `PATH` befinden (oder legen Sie `settings.binary` fest), und die Provider-Anmeldedaten müssen bereits konfiguriert sein. Für die AWS-Zulassung ist Crabbox 0.38.1 oder neuer erforderlich.
- Für Crabbox-AWS-Worker muss der effektive Wert von `aws.instanceProfile` leer sein. Der Provider prüft `crabbox config show --json` vor der Zuweisung und verlangt anschließend, dass `crabbox inspect --json` den Wert `providerMetadata.instanceProfileAttached: false` aus EC2-`DescribeInstances` meldet. Leases mit einer Instanzrolle oder ohne maßgebliche Metadaten werden gestoppt und abgelehnt.
- Node.js auf der geleasten Maschine. Unveränderte Cloud-Images enthalten es üblicherweise nicht – installieren Sie es mit dem `setup`-Befehl des Profils.
- Eine Sitzung mit einem sitzungseigenen verwalteten Worktree (erstellen Sie einen mit `worktree: true`). Beim Dispatch wird der Inhalt dieses Worktrees verschoben; einfache Verzeichnisse werden als Manifestspiegel synchronisiert.

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
| `provider` | Von einem Plugin registrierte Worker-Provider-ID (`crabbox` für das mitgelieferte Plugin).                                                                                                                                                    |
| `install`  | `bundle` (Standard) liefert den Build des laufenden Gateways aus; `npm` installiert die exakte veröffentlichte Gateway-Version mit gepinnter Integrität. `npm` setzt voraus, dass das Gateway aus einer paketierten Version ausgeführt wird. |
| `settings` | Provider-eigenes JSON. Für crabbox: `provider` (Backend), `class` (Maschinenklasse), `ttl`, `idleTimeout` (Go-Zeitangaben), optional `setup` und absoluter `binary`-Pfad. OpenClaw erzwingt öffentliches SSH und deaktiviert verwaltetes Tailscale für diese Leases. |
| `lifetime` | Optional gespeicherte Richtlinie (`idleTimeoutMinutes`, `maxLifetimeMinutes`).                                                                                                                                                                      |

### Der Einrichtungsbefehl

`settings.setup` wird auf der geleasten Box ausgeführt, nachdem sie über SSH erreichbar ist und bevor OpenClaw installiert wird. Er wird bei **jedem** Bereitstellungsversuch ausgeführt (einschließlich Wiederholungen nach einem unterbrochenen Dispatch) und muss daher idempotent sein – sichern Sie Installationen wie im Beispiel mit einer `command -v`- bzw. `test -x`-Prüfung ab. Wenn die Einrichtung fehlschlägt, stoppt der Provider die Lease und der Dispatch schlägt nach dem Fail-Closed-Prinzip fehl; es bleibt keine teilweise konfigurierte Box in Betrieb.

### Installationskanäle

- **`bundle`** bündelt `dist` des laufenden Gateways, eine bereinigte `package.json` und alle Arbeitsbereichspakete, auf die der Build verweist; alles wird durch einen Inhalts-Hash abgedeckt. Die Box prüft das unveränderte Bundle anhand dieses Hashs und installiert anschließend die npm-Produktionsabhängigkeiten (Skripte deaktiviert). So führen Sie einen Entwicklungs-Build auf einem Worker aus.
- **`npm`** weist nach, dass die Version in der öffentlichen Registry vorhanden ist, pinnt ihre SHA-512-Integrität und installiert `openclaw@<version>` exakt passend zum Gateway.

## Eine Sitzung dispatchen

Öffnen Sie in der Control UI **New Session**, wählen Sie einen Agenten, dessen konfigurierte Runtime OpenClaw ist, wählen Sie im Menü **Where** ein konfiguriertes Ziel vom Typ **Cloud · profile** aus und starten Sie die Aufgabe. Durch die Cloud-Auswahl wird der erforderliche verwaltete Worktree automatisch aktiviert; das Gateway erstellt die Sitzung, schließt den Dispatch ab und sendet erst danach die erste Ausführung. Das Server-Badge in der Sitzungsseitenleiste zeigt den dauerhaften Platzierungsstatus. Cloud-Ziele werden für externe CLI-Sitzungskataloge nicht angeboten.

Der entsprechende RPC-Ablauf lautet:

Erstellen Sie eine Sitzung mit einem verwalteten Worktree und dispatchen Sie sie anschließend (der RPC erfordert `operator.admin` und ist nur vorhanden, wenn Profile konfiguriert sind):

Cloud-Worker führen die OpenClaw-Agenten-Runtime aus. Wählen Sie ein `openai/*`- oder anderes Modell, das zu dieser Runtime aufgelöst wird; Sitzungen, die für eine externe CLI-Runtime wie `claude-cli` konfiguriert sind, können nicht dispatcht werden.

```bash
openclaw gateway call sessions.create \
  --params '{"key":"agent:main:big-refactor","worktree":true,"cwd":"/path/to/repo","worktreeName":"big-refactor"}'

openclaw gateway call sessions.dispatch \
  --timeout 1500000 \
  --params '{"key":"agent:main:big-refactor","profileId":"aws"}'
```

`sessions.dispatch` sperrt die lokale Annahme neuer Ausführungen, lässt aktive Arbeit auslaufen, stellt die Lease bereit, führt die Einrichtung aus, bootstrapt OpenClaw, synchronisiert den Arbeitsbereich und kehrt zurück, sobald die Platzierung die `active`-Worker-Eigentümerschaft erreicht. Planen Sie für den ersten Dispatch mehrere Minuten ein; Leases und Installationen werden zwischengespeichert, sofern der Provider dies unterstützt. Danach können Sie wie gewohnt mit der Sitzung interagieren – Ausführungen werden automatisch an den Worker weitergeleitet.

Abgeschlossene Worker-Ausführungen gleichen geeignete, größenbeschränkte Arbeitsbereichsdateien wieder mit dem verwalteten Worktree der Sitzung ab, bevor der Anspruch auf die Ausführung freigegeben wird. Das abschließende Worker-Ereignis erstellt eine dauerhafte Sperre für ausstehende Ergebnisse, bevor es bestätigt wird. Das Gateway legt anschließend das vollständige Cloud-Ergebnis als Git-Ref unter `refs/openclaw/worker-results/` ab, bevor es dieses anwendet. Dadurch bleibt die Cloud-Version wiederherstellbar, selbst wenn das Gateway während der Anwendung stoppt. Arbeitsbereichsergebnisse verwenden die Dateisemantik von Git: Reguläre Dateien, Ausführbarkeitsbits, symbolische Links, Ergänzungen, Änderungen und Löschungen bleiben erhalten; leere Verzeichnisse und andere Verzeichnismodi dagegen nicht. Die resultierenden Dateiänderungen verbleiben zur normalen Prüfung und zum Commit im verwalteten Worktree.

Bei der Anwendung dient das Manifest zum Zeitpunkt des Dispatchs als Merge-Basis. Ausschließlich in der Cloud vorgenommene Änderungen werden angewendet, ausschließlich lokal vorgenommene Änderungen bleiben bestehen, und für auf beiden Seiten geänderte Pfade gilt eine Drei-Wege-Richtlinie, die die lokale Version beibehält. Eine Ausführung mit Konflikten wird dennoch abgeschlossen: Das Transkript meldet die begrenzte Pfadzusammenfassung und die Ref des bereitgestellten Ergebnisses, die Platzierung stellt denselben Konflikt für die Control UI bereit und konfliktfreie Cloud-Änderungen bleiben angewendet. Der Hinweis enthält `git show <ref>:<path>` zum Prüfen einer vorhandenen Cloud-Datei sowie einen `git checkout <ref> -- <path>`-Befehl mit einem Literal-Pathspec auf oberster Ebene, um sie aus einem beliebigen Arbeitsbereichsverzeichnis zu übernehmen. Führen Sie die Befehle in Bash oder zsh aus (Git Bash unter Windows). Wenn die Prüfung meldet, dass der Pfad nicht vorhanden ist, wurde er durch das Cloud-Ergebnis gelöscht; prüfen und entfernen Sie den beibehaltenen lokalen Pfad manuell. Wenn der Checkout eine Datei-/Verzeichnisblockade meldet, verschieben oder entfernen Sie den blockierenden lokalen Pfad und versuchen Sie es erneut. Wenn die bereitgestellte Ref selbst nicht mehr vorhanden ist, behandeln Sie den Hinweis als veraltet und ändern Sie den lokalen Pfad nicht. Bereitgestellte Refs mit Konflikten bleiben verfügbar, nachdem die normale Ausführungssperre freigegeben wurde; ein späteres konfliktfreies Ergebnis entfernt den Hinweis und setzt die alte Ref außer Betrieb, während das explizite Entfernen der Sperre die endgültige Bereinigungsgrenze darstellt.

Während ein gesperrtes Ergebnis noch abgeglichen wird, wartet eine neue Ausführung bis zu 15 Sekunden darauf, dass der vorherige Anspruch freigegeben wird. Ist er danach weiterhin belegt, schlägt die Ausführung mit der handlungsorientierten Meldung „das Arbeitsbereichsergebnis der vorherigen Cloud-Ausführung wird noch abgeglichen“ fehl und kann kurz darauf erneut versucht werden. Nach einem Neustart erkennt die Wiederherstellung ausstehende und bereitgestellte Ergebnisse vor der Bereinigung veralteter Ansprüche, schließt deren lokale Anwendung ab oder versucht sie erneut und gibt inaktive Umgebungen erst zurück, nachdem das Ergebnis gesichert wurde. Das begrenzte SQLite-Rollback-Journal ermöglicht die Wiederherstellung einer unterbrochenen Dateisystemanwendung, ohne bereits akzeptierte Mutationen erneut abzuspielen.

Wenn die Arbeit abgeschlossen ist und keine Ausführung läuft, öffnen Sie das Sitzungsmenü und wählen Sie **Stop cloud worker…**. Das Gateway führt einen letzten Arbeitsbereichsabgleich durch, bevor es die Umgebung zerstört. Eine Platzierung, die sich bereits im Status `draining` oder `reconciling` befindet, schließt den Abbau ab; warten Sie, bis ihr Badge `reclaimed` anzeigt, bevor Sie die Sitzung löschen.

Bei einem defekten oder außer Kontrolle geratenen verbundenen Worker kann ein Operator als letztes Mittel `environments.destroy` mit `{ "force": true }` aufrufen. Ein erzwungener Abbau markiert die Platzierung dauerhaft als fehlgeschlagen und verwirft alle nicht abgeglichenen Remote-Ergebnisse, bevor die Umgebung zerstört wird.

Der entsprechende administrative RPC lautet:

```bash
openclaw gateway call sessions.reclaim \
  --timeout 600000 \
  --params '{"key":"agent:main:big-refactor"}'
```

Die Platzierung durchläuft einen dauerhaften Zustandsautomaten (`local → requested → provisioning → syncing → starting → active`), sodass ein Gateway-Neustart während der Verteilung einen Abgleich durchführt, statt Maschinen unkontrolliert weiterlaufen zu lassen. Bei einem fehlgeschlagenen Modelldurchlauf bleibt die aktive Platzierung für einen erneuten Versuch verfügbar. Bei Konflikten mit Workspace-Pfaden wird die lokale Version beibehalten, der übrige Teil des Cloud-Ergebnisses angewendet und die bereitgestellte Cloud-Referenz zur Überprüfung erhalten; bei anderen Abgleichs- oder Lebenszyklusfehlern bleiben die dauerhafte Wiederherstellungssperre und das Ende der Diagnoseausgabe erhalten, bis die Wiederherstellung einen erneuten Versuch sicher durchführen oder die Umgebung zurückfordern kann.

## Sicherheitsmodell

- **Geschlossener Worker-Eingang.** Worker kommunizieren über ein dediziertes Protokoll auf dem getunnelten Socket mit einer geschlossenen Methoden-Positivliste – ein Worker kann keine Operator-RPCs aufrufen.
- **Gateway-eigene Tool-Berechtigung.** Vor jedem Durchlauf projiziert das Gateway die aktuellen Richtlinien für Profil, Provider, Agent, Gruppe, Absender, Sandbox, Delegierung, Vererbung und Laufzeitbegrenzung auf den festen Katalog der Coding-Tools des Workers. Der Start-Umschlag enthält nur diese endgültige Teilmenge aus einem geschlossenen Vokabular. Explizit begrenzte geplante Durchläufe verwenden ihren vertrauenswürdigen Kontext der Eigentümergruppe erneut, ohne diese Identität an die Box zu senden oder ein neues Absender-Overlay anzuwenden. Tools außerhalb des Worker-Katalogs bleiben nicht verfügbar; bei einem leeren Ergebnis erfolgt die Ausführung ohne Tools.
- **Ausgestellte Anmeldedaten, gehasht gespeichert.** Jede Verteilung stellt Worker-Anmeldedaten aus; das Gateway speichert nur deren Hash. Die Rotation der Anmeldedaten und die Absicherung durch Eigentümer-Epochen gewährleisten höchstens einen aktiven Eigentümer pro Sitzung – ein veralteter Worker, der sich erneut verbindet, wird abgegrenzt und niemals zusammengeführt.
- **Fixierung des Host-Schlüssels.** Der Provider muss bei der Bereitstellung den SSH-Host-Schlüssel der Box bereitstellen; der Bootstrap stellt die Verbindung mit strikter Fixierung her und schlägt ohne ihn sicher geschlossen fehl.
- **Keine dauerhaft hinterlegten Modell-, Forge- oder Cloud-Anmeldedaten auf der Box.** Die Modellauthentifizierung verbleibt auf dem Gateway (die Inferenz wird über eine `{provider, model}`-Referenz übertragen), Git-Commits im Workspace werden ohne Forge-Anmeldedaten erstellt und die AWS-Lease-Metadaten von Crabbox werden vor der Einrichtung verbindlich auf eine Instanzrolle geprüft. Auch Einrichtungsbefehle müssen frei von Anmeldedaten bleiben.
- **Provider-eigener ausgehender Datenverkehr.** Der Rückwärtstunnel beseitigt für OpenClaw die Notwendigkeit eines direkten Modellzugriffs, aber OpenClaw schreibt Provider-Firewalls nicht um. Beschränken Sie den ausgehenden Datenverkehr beim Worker-Provider, wenn die Aufgabe dies erfordert.
- **Dauerhafte, genau einmal gespeicherte Transkripte.** Der Worker schreibt Transkript-Batches über ein Compare-and-Swap-Protokoll gegen das Blatt der Sitzung fest; bei einer veralteten Basis wird der Durchlauf sicher angehalten, statt kostenpflichtige Ausgabe zu duplizieren oder neu zu basieren.

## Fehlerbehebung

- **`sessions.dispatch` ist eine unbekannte Methode** – es sind keine `cloudWorkers.profiles` konfiguriert oder dem Aufrufer fehlt `operator.admin`.
- **„Cloud-Worker-Durchläufe erfordern die OpenClaw-Laufzeit“** – wählen Sie ein Modell, dessen konfigurierte Laufzeit OpenClaw ist. Externe CLI-Laufzeiten wie `claude-cli` unterstützen keine Worker-Inferenz.
- **„Der Worker-Bootstrap erfordert Node.js auf dem geleasten Host“** – fügen Sie `settings.setup` eine Node-Installation hinzu (siehe oben).
- **Die Attestierung der AWS-Instanzrolle schlägt fehl** – leeren Sie `aws.instanceProfile` (und `CRABBOX_AWS_INSTANCE_PROFILE`, falls festgelegt). Installieren Sie Crabbox 0.38.1 oder neuer; ältere Binärdateien stellen den für die AWS-Zulassung erforderlichen verbindlichen `providerMetadata.instanceProfileAttached`-Vertrag nicht bereit.
- **Die Verteilung schlägt mit einem Provider-Fehler fehl** – der Platzierungsdatensatz und `environments.list` speichern den letzten Fehler einschließlich des Endes der stderr-Ausgabe von Einrichtung und Bootstrap. Boxen werden bei einem Fehler zerstört, daher ist dieses Ausgabeende die primäre forensische Informationsquelle.
- **Client-Zeitüberschreitung während der Verteilung** – `openclaw gateway call` verwendet standardmäßig eine Zeitüberschreitung von 10s; bemessen Sie `--timeout` großzügig (die Verteilung läuft in jedem Fall serverseitig weiter, und ein erneuter Versuch während der Bereitstellung wird mit `session cannot dispatch from placement provisioning` abgelehnt).
- **Worker nach einem Upgrade von einer Beta-Version 2026.7.2 zurückgefordert** – diese Beta-Versionen verwendeten den älteren Worker-Startvertrag. Bei einem Neustart zerstört OpenClaw einen inaktiven inkompatiblen Worker, behält Sitzung und Workspace bei, markiert die Platzierung als zurückgefordert und stellt bei der nächsten Verteilung oder beim nächsten Durchlauf einen aktuellen Worker bereit. Ein Beta-Worker, der während des Starts unterbrochen wurde, wird nach der Bereinigung als fehlgeschlagen markiert; wiederholen Sie die Verteilung, um ihn mit dem aktuellen Vertrag bereitzustellen.
- **Hinweis auf einen Cloud-Workspace-Konflikt** – der Durchlauf wurde abgeschlossen und hat für jeden aufgeführten Pfad die lokale Version beibehalten. Verwenden Sie die Befehle für die bereitgestellte Referenz im Hinweis, um die Cloud-Version zu prüfen oder zu übernehmen; für die konfliktfreien Änderungen ist kein erneuter Versuch erforderlich, da sie bereits angewendet wurden.
- **„Das Workspace-Ergebnis des vorherigen Cloud-Durchlaufs wird noch abgeglichen“** – das Gateway hat kurz auf die dauerhafte Sperre des vorherigen Ergebnisses gewartet und konnte den Anspruch auf die Sitzung nicht erwerben. Warten Sie, bis der Abgleich abgeschlossen ist, und wiederholen Sie dann den Durchlauf; ein Neustart des Gateways ist sicher, da die Wiederherstellung bereitgestellte Ergebnisse erhält, bevor sie einen ausgefallenen Worker zurückfordert.
- **Lease-Verwaltung** – `crabbox list --provider <backend>` zeigt aktive Leases an; `crabbox stop --provider <backend> --id <lease>` gibt eine Lease manuell frei. Inaktive Leases laufen entsprechend dem `idleTimeout` des Profils ab.

## Verwandte Themen

- [Sandboxing](/de/gateway/sandboxing) – Verringerung des Schadensradius bei lokaler Tool-Ausführung
- [Sitzungs-CLI](/de/cli/sessions) – Überprüfung gespeicherter Sitzungen
- [Konfigurationsreferenz](/de/gateway/configuration-reference)
