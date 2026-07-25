---
read_when:
    - Sie möchten ein vollwertiges Backup-Archiv für den lokalen OpenClaw-Zustand
    - Sie benötigen einen kompakten, verifizierten Snapshot einer OpenClaw-SQLite-Datenbank
    - Sie möchten vor dem Zurücksetzen oder Deinstallieren eine Vorschau der enthaltenen Pfade anzeigen.
summary: CLI-Referenz für `openclaw backup` (Archive und SQLite-Snapshots)
title: Sicherung
x-i18n:
    generated_at: "2026-07-24T20:23:43Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: dfb5a118545589b181cede26dab72e9d029d98a1cac5cfccedd9d9cf2c56d3b5
    source_path: cli/backup.md
    workflow: 16
---

# `openclaw backup`

Erstellen Sie ein lokales Sicherungsarchiv für OpenClaw-Status, Konfiguration, Authentifizierungsprofile, Kanal-/Provider-Anmeldedaten, Sitzungen und optional Arbeitsbereiche.

```bash
openclaw backup create
openclaw backup create --output ~/Backups
openclaw backup create --dry-run --json
openclaw backup create --verify
openclaw backup create --no-include-workspace
openclaw backup create --only-config
openclaw backup verify ./2026-03-09T08-00-00.000+08-00-openclaw-backup.tar.gz
openclaw backup sqlite create --global --repository ~/Backups/openclaw-sqlite
openclaw backup sqlite create --agent main --repository ~/Backups/openclaw-sqlite
openclaw backup sqlite list --repository ~/Backups/openclaw-sqlite
openclaw backup sqlite verify ~/Backups/openclaw-sqlite/<snapshot-id>
openclaw backup sqlite verify ~/Backups/openclaw-sqlite/<snapshot-id> --scratch ~/Private/openclaw-scratch
openclaw backup sqlite restore ~/Backups/openclaw-sqlite/<snapshot-id> --target ./restored/openclaw.sqlite
```

## Hinweise

- Das Archiv enthält eine eingebettete `manifest.json` mit den aufgelösten Quellpfaden und dem Archivlayout.
- Standardmäßig wird im aktuellen Arbeitsverzeichnis ein mit einem Zeitstempel versehenes `.tar.gz`-Archiv erstellt. Dateinamen mit Zeitstempel verwenden die lokale Zeitzone Ihres Rechners und enthalten den UTC-Versatz. Wenn sich das aktuelle Arbeitsverzeichnis innerhalb eines gesicherten Quellbaums befindet, verwendet OpenClaw stattdessen Ihr Home-Verzeichnis als Standardspeicherort für das Archiv.
- Vorhandene Archivdateien werden niemals überschrieben. Ausgabepfade innerhalb der Quellbäume für Status oder Arbeitsbereiche werden abgelehnt, um eine Selbsteinbindung zu vermeiden.
- `openclaw backup verify <archive>` prüft, ob das Archiv genau ein Stammmanifest enthält, lehnt Archivpfade im Traversal-Stil und SQLite-Sidecars ab, bestätigt, dass jede im Manifest deklarierte Nutzlast vorhanden ist, validiert die Dateiform jedes SQLite-Snapshots und führt vollständige Integritäts- und Rollenprüfungen für kanonische OpenClaw-Datenbanken aus. Dedizierte Plugin-Schemas bleiben undurchsichtig, da sie möglicherweise vom Eigentümer definierte SQLite-Funktionen erfordern. `openclaw backup create --verify` führt diese Validierung unmittelbar nach dem Schreiben des Archivs aus.
- `openclaw backup create --only-config` sichert nur die aktive JSON-Konfigurationsdatei.

## SQLite-Snapshots

Verwenden Sie `openclaw backup sqlite`, wenn Sie statt eines umfassenden Statusarchivs ein portables Artefakt für eine einzelne OpenClaw-eigene SQLite-Datenbank benötigen.

Bei der Snapshot-Erstellung wird genau eine benannte Quelle akzeptiert:

| Befehl                                                          | Datenbank                    |
| --------------------------------------------------------------- | ---------------------------- |
| `openclaw backup sqlite create --global --repository <dir>`     | Gemeinsamer OpenClaw-Status |
| `openclaw backup sqlite create --agent <id> --repository <dir>` | Eine Datenbank pro Agent    |

Das Repository enthält ein Verzeichnis für jeden festgeschriebenen Snapshot. Jedes Snapshot-Verzeichnis enthält genau:

- `manifest.json`
- `database.sqlite`

Die Snapshot-Erstellung überprüft die aktive Datenbank vor dem Lesen, verwendet die Online-Sicherungs-API von SQLite, um den festgeschriebenen WAL-Status zu erfassen, ohne eine lange Lesetransaktion offen zu halten, schließt die aktive Datenbank, komprimiert die private Kopie mit `VACUUM`, überprüft die erzeugte Datenbank erneut und veröffentlicht das fertiggestellte Verzeichnis, ohne vorhandene Pfade zu überschreiben. Globale Snapshots entfernen vor der Compaction vorübergehende Zeilen der Zustellungswarteschlange, damit gelöschte Warteschlangen-Nutzlasten nicht in freien Seiten erhalten bleiben.

Kopieren Sie aktive `.sqlite`-, `-wal`-, `-shm`- oder `-journal`-Dateien nicht als Portabilitätsartefakt. Kopieren Sie ausschließlich fertiggestellte Snapshot-Verzeichnisse.

SQLite-Snapshots können Authentifizierungsprofile, Sitzungsstatus, Plugin-Status und andere sensible Datensätze enthalten. Schützen Sie Repositorys mit denselben Berechtigungen, derselben Verschlüsselung, derselben Aufbewahrungsrichtlinie und denselben Zielbeschränkungen wie das aktive OpenClaw-Statusverzeichnis.

### Überprüfen und wiederherstellen

```bash
openclaw backup sqlite verify <snapshot-directory>
openclaw backup sqlite restore <snapshot-directory> --target <new-database-path>
```

Die Überprüfung prüft die strikte Manifestform, Artefaktgröße und SHA-256, SQLite-Integrität, Fremdschlüssel, Schemaversion, Datenbankrolle und -eigentümer sowie OpenClaw-eigene Indexdefinitionen.

Die Überprüfung validiert eine private, an den Inhalt gebundene Kopie, sodass Pfadnamen-Wettläufe die von SQLite untersuchten Bytes nicht austauschen können. Standardmäßig wird diese temporäre Kopie neben dem Snapshot-Repository erstellt und entfernt, bevor der Befehl zurückkehrt. Das Staging-Stammverzeichnis und seine Vorfahrenkette müssen verhindern, dass andere Benutzer es ersetzen. POSIX-Stammverzeichnisse müssen dem aktuellen Benutzer gehören und dürfen weder für die Gruppe noch für alle Benutzer beschreibbar sein; Sticky-Vorfahren wie `/tmp` werden für benutzereigene untergeordnete Elemente akzeptiert. macOS-ACL-Berechtigungen, die das Staging offenlegen oder ersetzbar machen, werden abgelehnt. Windows-Stammverzeichnisse und deren Vorfahren müssen dem aktuellen Benutzer oder einem vertrauenswürdigen Betriebssystemprinzipal gehören und ACLs aufweisen, die nicht vertrauenswürdigem Zugriff auf das Staging verweigern. Geben Sie für eine schreibgeschützte Einbindung oder Netzwerkfreigabe `--scratch <existing-private-directory>` auf einem Speicher mit gleichwertiger Verschlüsselung und gleichwertigen Zielkontrollen an.

Die Snapshot-Erstellung wendet vor dem Staging oder der Veröffentlichung von Datenbankbytes dieselben Prüfungen für Eigentümer, ACLs, Vorfahren und Pfadidentität auf das Repository an.

Die Wiederherstellung wiederholt die Überprüfung und schreibt ausschließlich in ein neues Ziel. Sie lehnt ein vorhandenes Ziel sowie `-wal`-, `-shm`- oder `-journal`-Sidecars ab und ersetzt niemals eine aktive OpenClaw-Datenbank direkt. Für das übergeordnete Zielverzeichnis gelten dieselben Pfadsicherheitsanforderungen wie für das temporäre Überprüfungsverzeichnis. Die Aktivierung einer wiederhergestellten Datenbank bleibt ein expliziter Offline-Schritt für den Betreiber.

Snapshot-Repositorys sind lokale Verzeichnisse. Zeitplanung, Upload, Aufbewahrung, inkrementelle WAL-Pakete, Failover und Wiederherstellung beim Systemstart liegen absichtlich außerhalb dieses Befehls.

## Was gesichert wird

`openclaw backup create` plant Quellen aus Ihrer lokalen OpenClaw-Installation:

- Das Statusverzeichnis (normalerweise `~/.openclaw`)
- Der Pfad der aktiven Konfigurationsdatei
- Das aufgelöste `credentials/`-Verzeichnis, wenn es außerhalb des Statusverzeichnisses vorhanden ist
- Aus der aktuellen Konfiguration ermittelte Arbeitsbereichsverzeichnisse, sofern Sie nicht `--no-include-workspace` übergeben

Authentifizierungsprofile und anderer agentenspezifischer Laufzeitstatus befinden sich in SQLite unterhalb des Statusverzeichnisses (`agents/<agentId>/agent/openclaw-agent.sqlite`) und werden daher automatisch durch den Statussicherungseintrag abgedeckt.

`--only-config` überspringt die Ermittlung von Status, Anmeldedatenverzeichnis und Arbeitsbereichen und archiviert ausschließlich den Pfad der aktiven Konfigurationsdatei.

OpenClaw kanonisiert Pfade vor dem Erstellen des Archivs: Wenn sich die Konfiguration, das Anmeldedatenverzeichnis oder ein Arbeitsbereich bereits innerhalb des Statusverzeichnisses befindet, wird er nicht als separate Sicherungsquelle auf oberster Ebene dupliziert. Fehlende Pfade werden übersprungen.

Während der Archiverstellung schließt OpenClaw bekannte Pfade mit aktiven Änderungen aus, bevor `tar` sie liest. Dadurch werden Wettläufe zwischen der aufgezeichneten Größe einer Datei und gleichzeitigen Schreibvorgängen vermieden. Der Filter wendet unter jedem gesicherten Statusverzeichnis die folgenden relativ zum Statusverzeichnis definierten Regeln an:

| Relativer Geltungsbereich im Statusverzeichnis    | Übersprungene Dateiendungen    |
| ------------------------------------------------- | ------------------------------ |
| `sessions/**`                                | `.jsonl`, `.log`              |
| `agents/<agentId>/sessions/**`               | `.jsonl`, `.log`              |
| `cron/runs/**`                               | `.jsonl`, `.log`              |
| `logs/**`                                    | `.jsonl`, `.log`              |
| `delivery-queue/**`                          | `.json`, `.delivered`, `.tmp` |
| `session-delivery-queue/**`                  | `.json`, `.delivered`, `.tmp` |
| Jeder Pfad unterhalb des gesicherten Statusverzeichnisses | `.sock`, `.pid`, `.tmp`       |

Diese Regeln filtern keine Arbeitsbereichsdateien außerhalb des Statusverzeichnisses. Sie lassen außerdem abgeschlossene Transkript- und Protokolldateien aus, die der Tabelle entsprechen; bewahren Sie diese Datensätze daher bei Bedarf separat auf. `skippedVolatileCount` im JSON-Ergebnis gibt an, wie viele Dateien absichtlich ausgelassen wurden.

SQLite-Datenbanken unterhalb des Statusverzeichnisses werden mit der Online-Sicherungs-API von SQLite erfasst und offline mit `VACUUM` komprimiert, damit Reste gelöschter Seiten nicht in das Archiv gelangen und aktive WAL-/SHM-Dateien nicht kopiert werden. Bei einer Plugin-eigenen Datenbank, die nicht verfügbare, vom Eigentümer definierte SQLite-Funktionen erfordert, wird sicher abgebrochen, anstatt auf ein direktes Kopieren der Datei zurückzugreifen. SQLite-Dateien, die über Arbeitsbereichssicherungen einbezogen werden, werden als Arbeitsbereichsdateien kopiert und fallen nicht unter die Compaction-Garantie.

Installierte Plugin-Quell- und Manifestdateien unterhalb des `extensions/`-Baums des Statusverzeichnisses werden einbezogen, ihre verschachtelten `node_modules/`-Abhängigkeitsbäume werden jedoch als erneut erstellbare Installationsartefakte übersprungen. Verwenden Sie nach der Wiederherstellung eines Archivs `openclaw plugins update <id>` oder führen Sie mit `openclaw plugins install <spec> --force` eine Neuinstallation durch, falls ein wiederhergestelltes Plugin fehlende Abhängigkeiten meldet.

Vom Installationsprogramm verwaltete und erneut erstellbare Laufzeit-Stammverzeichnisse unterhalb des Statusverzeichnisses werden ebenfalls übersprungen: `dev/`, `git/`, `npm/`, das veraltete `npm-runtime/` und `tools/`. Diese enthalten verwaltete Checkouts, Paketbäume und heruntergeladene Laufzeiten statt maßgeblichen Benutzerstatus; installieren oder aktualisieren Sie nach der Wiederherstellung die entsprechende Laufzeit oder das entsprechende Plugin. Eine explizit konfigurierte Konfigurationsdatei, ein Anmeldedatenverzeichnis oder ein Arbeitsbereich innerhalb eines dieser Stammverzeichnisse bleibt enthalten.

## Verhalten bei ungültiger Konfiguration

`openclaw backup` umgeht die normale Vorabprüfung der Konfiguration, damit es auch bei einer Wiederherstellung helfen kann. Die Ermittlung von Arbeitsbereichen hängt von einer gültigen Konfiguration ab. Daher bricht `openclaw backup create` sofort ab, wenn die Konfigurationsdatei vorhanden, aber ungültig ist und die Sicherung von Arbeitsbereichen weiterhin aktiviert ist.

Führen Sie für eine Teilsicherung in dieser Situation den Befehl erneut mit `--no-include-workspace` aus: Status, Konfiguration und das externe Anmeldedatenverzeichnis bleiben im Umfang enthalten, während die Ermittlung von Arbeitsbereichen vollständig übersprungen wird.

`--only-config` funktioniert ebenfalls bei einer fehlerhaften Konfiguration, da die Konfiguration nicht zur Ermittlung von Arbeitsbereichen analysiert wird.

## Größe und Leistung

OpenClaw erzwingt weder eine integrierte maximale Sicherungsgröße noch eine Größenbeschränkung pro Datei. Ein Archivschreibvorgang, der fünf Minuten lang keine Daten erzeugt, schlägt fehl und entfernt seine unvollständige temporäre Datei, anstatt unbegrenzt zu hängen. Praktische Grenzen ergeben sich ansonsten aus:

- Verfügbarem Speicherplatz für das Schreiben des temporären Archivs und das endgültige Archiv
- Der benötigten Zeit, um große Arbeitsbereichsbäume zu durchlaufen und in ein `.tar.gz` zu komprimieren
- Der benötigten Zeit, um das Archiv mit `--verify` oder `openclaw backup verify` erneut zu durchsuchen
- Dem Verhalten des Ziel-Dateisystems: OpenClaw erfordert eine Veröffentlichung über nicht überschreibende Hardlinks, damit ein endgültiger Archivpfad niemals eine noch laufende Kopie offenlegt; nicht unterstützte Dateisysteme schlagen mit einer umsetzbaren Fehlermeldung fehl

Wenn die Bestätigung der Dauerhaftigkeit des endgültigen Verzeichnisses nach der Veröffentlichung fehlschlägt, meldet der Befehl einen Fehler, behält jedoch den vollständigen endgültigen Eintrag bei, anstatt zu riskieren, einen parallel erstellten Ersatz zu löschen.

Große Arbeitsbereiche sind normalerweise der wichtigste Faktor für die Archivgröße. Verwenden Sie `--no-include-workspace` für eine kleinere und schnellere Sicherung oder `--only-config` für das kleinste Archiv.

## Verwandte Themen

- [CLI-Referenz](/de/cli)
