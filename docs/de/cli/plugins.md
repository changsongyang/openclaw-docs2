---
read_when:
    - Sie möchten Gateway-Plugins oder kompatible Bundles installieren oder verwalten
    - Sie möchten ein einfaches Tool-Plugin erstellen oder validieren
    - Sie möchten Fehler beim Laden von Plugins debuggen
sidebarTitle: Plugins
summary: CLI-Referenz für `openclaw plugins` (initialisieren, erstellen, validieren, auflisten, installieren, Marketplace, deinstallieren, aktivieren/deaktivieren, Doctor)
title: Plugins
x-i18n:
    generated_at: "2026-07-24T16:47:56Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: a1acba76fb1bc0ddae75e51fe573d3c2ac8f694607836e0c072ec7ca8fc0e262
    source_path: cli/plugins.md
    workflow: 16
---

Verwalten Sie Gateway-Plugins, Hook-Pakete und kompatible Bundles.

<CardGroup cols={2}>
  <Card title="Plugin-System" href="/de/tools/plugin">
    Benutzerhandbuch zum Installieren, Aktivieren und Beheben von Problemen bei Plugins.
  </Card>
  <Card title="Plugins verwalten" href="/de/plugins/manage-plugins">
    Kurze Beispiele für Installation, Auflistung, Aktualisierung, Deinstallation und Veröffentlichung.
  </Card>
  <Card title="Plugin-Bundles" href="/de/plugins/bundles">
    Kompatibilitätsmodell für Bundles.
  </Card>
  <Card title="Plugin-Manifest" href="/de/plugins/manifest">
    Manifestfelder und Konfigurationsschema.
  </Card>
  <Card title="Sicherheit" href="/de/gateway/security">
    Sicherheitshärtung für Plugin-Installationen.
  </Card>
</CardGroup>

## Befehle

```bash
openclaw plugins list [--enabled] [--verbose] [--json]
openclaw plugins search <query> [--limit <n>] [--json]
openclaw plugins install <path-or-spec> [--link] [--force] [--pin] [--marketplace <source>]
openclaw plugins inspect <id> [--runtime] [--json]
openclaw plugins inspect --all [--runtime] [--json]
openclaw plugins info <id>                    # Alias für inspect
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id> [--dry-run] [--keep-files] [--force]
openclaw plugins update <id-or-npm-spec> | --all [--dry-run]
openclaw plugins registry [--refresh] [--json]
openclaw plugins doctor
openclaw plugins init <id> [--name <name>] [--type tool|provider] [--directory <path>]
openclaw plugins build [--entry <path>] [--check]
openclaw plugins validate [--entry <path>]
openclaw plugins marketplace entries [--offline] [--feed-profile <name>] [--json]
openclaw plugins marketplace list <source> [--json]
openclaw plugins marketplace refresh [--feed-profile <name>] [--expected-sha256 <sha256>] [--json]
```

Führen Sie zur Untersuchung langsamer Installations-, Prüf-, Deinstallations- oder Registry-Aktualisierungsvorgänge den
Befehl mit `OPENCLAW_PLUGIN_LIFECYCLE_TRACE=1` aus. Der Trace schreibt die Zeitmessungen der einzelnen Phasen
nach stderr und hält die JSON-Ausgabe analysierbar. Siehe [Fehlerbehebung](/de/help/debugging#plugin-lifecycle-trace).

<Note>
Im Nix-Modus (`OPENCLAW_NIX_MODE=1`) ist `openclaw.json` unveränderlich. `install`, `update`, `uninstall`, `enable` und `disable` verweigern alle die Ausführung. Bearbeiten Sie stattdessen die Nix-Quelle für diese Installation (`programs.openclaw.config` oder `instances.<name>.config` für nix-openclaw) und erstellen Sie sie anschließend neu. Siehe den agentenorientierten [Schnellstart](https://github.com/openclaw/nix-openclaw#quick-start).
</Note>

<Note>
Gebündelte Plugins werden mit OpenClaw ausgeliefert. Einige sind standardmäßig aktiviert (zum Beispiel gebündelte Modell-Provider, gebündelte Sprach-Provider und das gebündelte Browser-Plugin); andere erfordern `plugins enable`.

Native OpenClaw-Plugins liefern `openclaw.plugin.json` mit einem eingebetteten JSON-Schema (`configSchema`, auch wenn es leer ist) aus. Kompatible Bundles verwenden stattdessen ihre eigenen Bundle-Manifeste.

`plugins list` zeigt `Format: openclaw` oder `Format: bundle` an. Die ausführliche Listen-/Infoausgabe zeigt außerdem den Bundle-Untertyp (`codex`, `claude` oder `cursor`) sowie erkannte Bundle-Funktionen an.
</Note>

## Entwicklung

```bash
openclaw plugins init stock-quotes --name "Stock Quotes"
cd stock-quotes
npm run plugin:build
npm run plugin:validate
```

`plugins init` erstellt standardmäßig ein minimales TypeScript-Tool-Plugin. Das erste
Argument ist die Plugin-ID; `--name` legt den Anzeigenamen fest. OpenClaw verwendet die
ID für das standardmäßige Ausgabeverzeichnis und die Paketbenennung. Tool-Gerüste verwenden
`defineToolPlugin` und erzeugen `package.json`-Skripte `plugin:build` und
`plugin:validate`, die zunächst den Build durchführen und anschließend `openclaw plugins build`/`validate` aufrufen.

`plugins build` importiert den erstellten Einstiegspunkt, liest dessen statische Tool-Metadaten, schreibt
`openclaw.plugin.json` und hält `openclaw.extensions` von `package.json` synchron.
`plugins validate` prüft, ob das generierte Manifest, die Paketmetadaten und
der aktuelle Export des Einstiegspunkts weiterhin übereinstimmen. Den vollständigen Entwicklungsablauf finden Sie unter [Tool-Plugins](/de/plugins/tool-plugins).

Das Gerüst schreibt TypeScript-Quellcode, generiert die Metadaten jedoch aus dem erstellten
`./dist/index.js`-Einstiegspunkt, sodass der Ablauf auch mit der veröffentlichten CLI funktioniert. Verwenden Sie
`--entry <path>`, wenn der Einstiegspunkt nicht dem standardmäßigen Paketeinstiegspunkt entspricht. Verwenden Sie
`plugins build --check` in der CI, damit der Vorgang fehlschlägt, wenn generierte Metadaten veraltet sind, ohne
Dateien neu zu schreiben.

### Provider-Gerüst

```bash
openclaw plugins init acme-models --name "Acme Models" --type provider
cd acme-models
npm install
npm run build
npm test
npm run validate
```

Provider-Gerüste erstellen ein generisches OpenAI-kompatibles Modell-Provider-Plugin
mit API-Schlüssel-Authentifizierungsanbindung, einem `npm run validate`-Skript, das
`clawhub package validate` ausführt, ClawHub-Paketmetadaten und einem manuell
ausgelösten GitHub-Actions-Workflow für eine zukünftige vertrauenswürdige Veröffentlichung über GitHub
OIDC. Provider-Gerüste generieren keine Skills und verwenden nicht
`openclaw plugins build`/`validate`; diese Befehle dienen dem Pfad für generierte Metadaten
des Tool-Gerüsts.

Ersetzen Sie vor der Veröffentlichung die Platzhalter für die API-Basis-URL, den Modellkatalog, die Dokumentationsroute,
den Anmeldedatentext und den README-Text durch echte Provider-Details. Verwenden Sie die
generierte README für die erstmalige Veröffentlichung auf ClawHub und die Einrichtung eines vertrauenswürdigen Herausgebers.

## Installation

```bash
openclaw plugins search "calendar"                      # ClawHub-Plugins durchsuchen
openclaw plugins install @openclaw/<package>            # vertrauenswürdiger offizieller Katalog
openclaw plugins install <package>                       # beliebiges npm-Paket
openclaw plugins install clawhub:<package>                # nur ClawHub
openclaw plugins install npm:<package>                    # nur npm
openclaw plugins install npm-pack:<path.tgz>               # lokales npm-pack-Tarball
openclaw plugins install git:github.com/<owner>/<repo>     # Git-Repository
openclaw plugins install git:github.com/<owner>/<repo>@<ref>
openclaw plugins install <path>                            # lokaler Pfad oder Archiv
openclaw plugins install -l <path>                         # verknüpfen statt kopieren
openclaw plugins install <plugin>@<marketplace>             # Marketplace-Kurzform
openclaw plugins install <plugin> --marketplace <name>      # Marketplace (explizit)
openclaw plugins install <package> --force                  # Quelle bestätigen / vorhandene Installation überschreiben
openclaw plugins install <package> --pin                    # aufgelöste npm-Version fixieren
openclaw plugins install clawhub:<package> --acknowledge-clawhub-risk
openclaw plugins install <package> --dangerously-force-unsafe-install
```

Maintainer, die Installationen während der Einrichtung testen, können automatische Quellen für die Plugin-Installation
mit geschützten Umgebungsvariablen überschreiben. Siehe
[Überschreibungen für Plugin-Installationen](/de/plugins/install-overrides).

<Warning>
Während der Umstellung beim Start werden reine Paketnamen standardmäßig von npm installiert, sofern sie nicht mit einer gebündelten oder offiziellen Plugin-ID übereinstimmen. In diesem Fall verwendet OpenClaw die lokale/offizielle Kopie, statt auf die npm-Registry zuzugreifen. Verwenden Sie stattdessen `npm:<package>`, wenn Sie ausdrücklich ein externes npm-Paket wünschen. Verwenden Sie `clawhub:<package>` für ClawHub. Behandeln Sie Plugin-Installationen wie die Ausführung von Code; bevorzugen Sie festgelegte Versionen.
</Warning>

<Warning>
ClawHub-Pakete und der gebündelte/offizielle Katalog von OpenClaw sind vertrauenswürdige Installationsquellen. Bei einem neuen beliebigen npm-, `npm-pack:`-, Git-, lokalen Pfad-/Archiv- oder Marketplace-Ursprung wird eine Warnung angezeigt und vor dem Fortfahren nachgefragt. Nicht interaktive beliebige Installationen müssen nach Prüfung und Vertrauensbestätigung der Quelle `--force` übergeben. Dasselbe
Flag überschreibt bei Bedarf ein vorhandenes Installationsziel. Normale Aktualisierungen einer
bereits nachverfolgten Installation benötigen es nicht. Diese Bestätigung ist unabhängig von
`--acknowledge-clawhub-risk`, das nur für Vertrauenswarnungen bei riskanten ClawHub-Veröffentlichungen gilt.
`--force` umgeht weder `security.installPolicy` noch die verbleibenden
Sicherheitsprüfungen der Installation.
</Warning>

`plugins search` fragt ClawHub nach installierbaren `code-plugin`- und
`bundle-plugin`-Paketen ab (nicht nach Skills; verwenden Sie dafür `openclaw skills search`).
Der Standardwert für `--limit` ist 20, begrenzt auf 100. Dabei wird ausschließlich der Remote-Katalog gelesen: keine
Prüfung des lokalen Zustands, keine Änderung der Konfiguration, keine Paketinstallation und kein Laden der Plugin-Laufzeit.
Die Ergebnisse enthalten den ClawHub-Paketnamen, die Familie, den Kanal, die Version,
eine Zusammenfassung und einen Installationshinweis wie `openclaw plugins install clawhub:<package>`.

<Note>
ClawHub ist für die meisten Plugins die primäre Oberfläche für Verteilung und Suche. Npm
bleibt ein unterstützter Ausweich- und Direktinstallationspfad. OpenClaw-eigene
`@openclaw/*`-Plugin-Pakete werden wieder auf npm veröffentlicht; die aktuelle Liste finden Sie
auf [npmjs.com/org/openclaw](https://www.npmjs.com/org/openclaw) oder im
[Plugin-Inventar](/de/plugins/plugin-inventory). Stabile Installationen verwenden `latest`.
Installationen und Aktualisierungen im Beta-Kanal bevorzugen, sofern verfügbar, das npm-Dist-Tag `beta`,
und greifen andernfalls auf `latest` zurück. Im Extended-Stable-Kanal werden offizielle npm-Plugins
mit reiner/standardmäßiger oder `latest`-Absicht auf die exakt installierte Core-Version
aufgelöst. Exakte Fixierungen und explizite Tags, die nicht `latest` sind, Drittanbieterpakete und
Nicht-npm-Quellen werden nicht umgeschrieben.
</Note>

<AccordionGroup>
  <Accordion title="Konfigurations-Includes und Reparatur ungültiger Konfigurationen">
    Wenn Ihr Abschnitt `plugins` durch ein einzelnes Datei-Include `$include` bereitgestellt wird, schreibt `plugins install/update/enable/disable/uninstall` direkt in diese eingebundene Datei und lässt `openclaw.json` unverändert. Root-Includes, Include-Arrays und Includes mit parallelen Überschreibungen schlagen sicher fehl, statt die Struktur zu verflachen. Die unterstützten Formen finden Sie unter [Konfigurations-Includes](/de/gateway/configuration).

    Wenn die Konfiguration vor der Installation ungültig ist, schlägt `plugins install` normalerweise sicher fehl und fordert Sie auf, zuerst `openclaw doctor --fix` auszuführen. Während des Gateway-Starts und beim Hot Reload schlägt eine ungültige Plugin-Konfiguration wie jede andere ungültige Konfiguration sicher fehl; `openclaw doctor --fix` kann den ungültigen Plugin-Eintrag isolieren. Die einzige Ausnahme für eine bereits vorhandene Konfiguration ist ein eng begrenzter Wiederherstellungspfad für gebündelte Plugins, die sich ausdrücklich für `openclaw.install.allowInvalidConfigRecovery` anmelden.

    Wenn die vorhandene Host-Konfiguration gültig ist, aber die eigene Konfiguration des neu installierten Plugins fehlt, zeichnet OpenClaw die Installation als deaktiviert auf, statt einen ungültigen aktivierten Eintrag zu schreiben. Konfigurieren Sie `plugins.entries.<id>.config` und führen Sie anschließend `openclaw plugins enable <id>` aus. Wenn ein vorhandener Plugin-Konfigurationseintrag existiert, aber ungültig ist, schlägt die Installation fehl, ohne ihn umzuschreiben.

  </Accordion>
  <Accordion title="--force-Bestätigung und Neuinstallation im Vergleich zur Aktualisierung">
    `--force` bestätigt eine Nicht-ClawHub-Quelle ohne Rückfrage. Es umgeht weder `security.installPolicy` noch die verbleibenden Sicherheitsprüfungen der Installation. Wenn das Plugin oder Hook-Paket bereits installiert ist, verwendet es außerdem das vorhandene Ziel erneut und überschreibt es an Ort und Stelle. Verwenden Sie es nach der Prüfung einer beliebigen npm-, lokalen, Archiv-, Git- oder Marketplace-Quelle oder wenn Sie dieselbe ID absichtlich neu installieren. Bevorzugen Sie für routinemäßige Upgrades eines bereits nachverfolgten npm-Plugins `openclaw plugins update <id-or-npm-spec>`.

    Wenn Sie `plugins install` für eine bereits installierte Plugin-ID ausführen, hält OpenClaw an und verweist für ein normales Upgrade auf `plugins update <id-or-npm-spec>` oder auf `plugins install <package> --force`, wenn Sie die aktuelle Installation tatsächlich aus einer anderen Quelle überschreiben möchten. Bei beliebigen Quellen wird weiterhin die interaktive Herkunftswarnung angezeigt; nicht interaktive Installationen müssen nach der Prüfung `--force` übergeben. Vertrauenswürdige Quellen aus ClawHub und dem OpenClaw-Katalog benötigen dies nicht. Mit `--link` bestätigt `--force` die Quelle, ändert jedoch nicht den Installationsmodus mit verknüpftem Pfad.

  </Accordion>
  <Accordion title="Geltungsbereich von --pin">
    `--pin` gilt nur für npm-Installationen und zeichnet die aufgelöste exakte `<name>@<version>` auf. Es wird weder bei Installationen mit `git:` (heften Sie stattdessen die Referenz in der Spezifikation an, z. B. `git:github.com/acme/plugin@v1.2.3`) noch mit `--marketplace` unterstützt (Marketplace-Installationen speichern statt einer npm-Spezifikation die Marketplace-Quellmetadaten).
  </Accordion>
  <Accordion title="--dangerously-force-unsafe-install">
    `--dangerously-force-unsafe-install` ist veraltet und hat jetzt keine Wirkung. OpenClaw führt bei Plugin-Installationen keine integrierte Blockierung gefährlichen Codes während der Installation mehr aus.

    Verwenden Sie die vom Betreiber verwaltete Oberfläche `security.installPolicy`, wenn eine hostspezifische Installationsrichtlinie erforderlich ist. Plugin-Hooks vom Typ `before_install` sind Lebenszyklus-Hooks der Plugin-Laufzeit und nicht die primäre Richtliniengrenze für CLI-Installationen.

    Wenn ein von Ihnen auf ClawHub veröffentlichtes Plugin durch einen Registry-Scan ausgeblendet oder blockiert wird, führen Sie die Schritte für Herausgeber unter [Veröffentlichung auf ClawHub](/de/clawhub/publishing) aus. `--dangerously-force-unsafe-install` fordert ClawHub weder dazu auf, das Plugin erneut zu scannen, noch dazu, eine blockierte Version öffentlich zugänglich zu machen.

  </Accordion>
  <Accordion title="--acknowledge-clawhub-risk">
    Bei Community-Installationen aus ClawHub wird vor dem Herunterladen der Vertrauensdatensatz der ausgewählten Version geprüft. Wenn ClawHub den Download für die Version deaktiviert, schädliche Scanergebnisse meldet oder die Version in einen blockierenden Moderationsstatus versetzt (unter Quarantäne gestellt, widerrufen), lehnt OpenClaw sie unabhängig von diesem Flag vollständig ab. Bei nicht blockierenden riskanten Scan- oder Moderationsstatus zeigt OpenClaw die Vertrauensdetails an und fordert vor dem Fortfahren eine Bestätigung an.

    Verwenden Sie `--acknowledge-clawhub-risk` nur, nachdem Sie die ClawHub-Warnung geprüft und entschieden haben, ohne interaktive Eingabeaufforderung fortzufahren. Ausstehende oder veraltete (noch nicht als unbedenklich eingestufte) Scanergebnisse lösen eine Warnung aus, erfordern jedoch keine Bestätigung. Offizielle ClawHub-Pakete und gebündelte OpenClaw-Plugin-Quellen umgehen diese Vertrauensprüfung der Version vollständig.

  </Accordion>
  <Accordion title="Hook-Pakete und npm-Spezifikationen">
    `plugins install` ist auch die Installationsoberfläche für Hook-Pakete, die `openclaw.hooks` in `package.json` bereitstellen. Verwenden Sie `openclaw hooks` für die gefilterte Sichtbarkeit von Hooks und die Aktivierung einzelner Hooks, nicht für die Paketinstallation.

    Npm-Spezifikationen sind **ausschließlich für die Registry** bestimmt (Paketname plus optionale **exakte Version** oder optionales **dist-tag**). Git-/URL-/Dateispezifikationen und Semver-Bereiche werden abgelehnt. Abhängigkeitsinstallationen werden aus Sicherheitsgründen in einem verwalteten npm-Projekt pro Plugin mit `--ignore-scripts` ausgeführt, selbst wenn Ihre Shell globale npm-Installationseinstellungen verwendet. Verwaltete npm-Projekte für Plugins übernehmen die npm-Einstellung `overrides` auf Paketebene von OpenClaw, sodass Sicherheits-Pins des Hosts auch für hochgezogene Plugin-Abhängigkeiten gelten.

    Verwenden Sie `npm:<package>`, um die npm-Auflösung explizit festzulegen. Während der Umstellung beim Start werden auch einfache Paketspezifikationen direkt von npm installiert, sofern sie nicht mit einer offiziellen Plugin-ID übereinstimmen.

    Unverarbeitete `@openclaw/*`-Spezifikationen, die gebündelten Plugins entsprechen, werden vor dem npm-Fallback in die zum Image gehörende gebündelte Kopie aufgelöst. Beispielsweise verwendet `openclaw plugins install @openclaw/discord@2026.5.20 --pin` das gebündelte Discord-Plugin aus dem aktuellen OpenClaw-Build, anstatt eine verwaltete npm-Überschreibung zu erstellen. Um die Verwendung des externen npm-Pakets zu erzwingen, verwenden Sie `openclaw plugins install npm:@openclaw/discord@2026.5.20 --pin`.

    Einfache Spezifikationen und `@latest` verbleiben im stabilen Kanal. Mit einem Datum versehene Korrekturversionen von OpenClaw wie `2026.5.3-1` gelten bei dieser Prüfung als stabil. Wenn npm eine der beiden Formen in eine Vorabversion auflöst, bricht OpenClaw ab und fordert Sie auf, sich ausdrücklich mit einem Vorabversions-Tag (`@beta`/`@rc`) oder einer exakten Vorabversion (`@1.2.3-beta.4`) dafür zu entscheiden.

    Bei npm-Installationen ohne exakte Version (`npm:<package>` oder `npm:<package>@latest`) prüft OpenClaw vor der Installation die aufgelösten Paketmetadaten. Wenn das neueste stabile Paket eine neuere OpenClaw-Plugin-API oder eine höhere Mindestversion des Hosts erfordert, prüft OpenClaw ältere stabile Versionen und installiert stattdessen die neueste kompatible Version. Exakte Versionen und explizite dist-tags bleiben strikt: Eine inkompatible Auswahl schlägt fehl und fordert Sie auf, OpenClaw zu aktualisieren oder eine kompatible Version auszuwählen.

    Wenn eine einfache Installationsspezifikation mit einer offiziellen Plugin-ID übereinstimmt (beispielsweise `diffs`), installiert OpenClaw den Katalogeintrag direkt. Um ein npm-Paket mit demselben Namen zu installieren, verwenden Sie eine explizite bereichsbezogene Spezifikation (beispielsweise `@scope/diffs`).

  </Accordion>
  <Accordion title="Git-Repositorys">
    Verwenden Sie `git:<repo>`, um direkt aus einem Git-Repository zu installieren. Unterstützte Formen: `git:github.com/owner/repo`, `git:owner/repo`, vollständige `https://`-, `ssh://`-, `git://`-, `file://`- und `git@host:owner/repo.git`-Klon-URLs. Fügen Sie `@<ref>` oder `#<ref>` hinzu, um vor der Installation einen Branch, ein Tag oder einen Commit auszuchecken.

    Git-Installationen klonen in ein temporäres Verzeichnis, checken die angeforderte Referenz aus, sofern vorhanden, und verwenden anschließend das normale Installationsprogramm für Plugin-Verzeichnisse. Dadurch verhalten sich Manifestvalidierung, Installationsrichtlinie des Betreibers, Installationsvorgänge des Paketmanagers und Installationsdatensätze wie bei npm-Installationen. Aufgezeichnete Git-Installationen enthalten die Quell-URL/-Referenz sowie den aufgelösten Commit, damit `openclaw plugins update` die Quelle später erneut auflösen kann.

    Verwenden Sie nach der Installation aus Git `openclaw plugins inspect <id> --runtime --json`, um Laufzeitregistrierungen wie Gateway-Methoden und CLI-Befehle zu überprüfen. Wenn das Plugin mit `api.registerCli` einen CLI-Stammbefehl registriert hat, führen Sie diesen Befehl direkt über die Stamm-CLI von OpenClaw aus, beispielsweise `openclaw demo-plugin ping`.

  </Accordion>
  <Accordion title="Archive">
    Unterstützte Archive: `.zip`, `.tgz`, `.tar.gz`, `.tar`. Native OpenClaw-Plugin-Archive müssen im Stammverzeichnis des extrahierten Plugins eine gültige `openclaw.plugin.json` enthalten; Archive, die nur `package.json` enthalten, werden abgelehnt, bevor OpenClaw Installationsdatensätze schreibt.

    Verwenden Sie `npm-pack:<path.tgz>`, wenn es sich bei der Datei um einen npm-pack-Tarball handelt und Sie
    denselben verwalteten npm-Projektpfad pro Plugin wie bei Registry-Installationen verwenden möchten,
    einschließlich der Überprüfung von `package-lock.json`, des Scannens hochgezogener Abhängigkeiten
    und der npm-Installationsdatensätze. Einfache Archivpfade werden weiterhin als lokale
    Archive unter dem Stammverzeichnis der Plugin-Erweiterungen installiert.

    Installationen aus dem Claude Marketplace werden ebenfalls unterstützt.

  </Accordion>
</AccordionGroup>

ClawHub-Installationen verwenden einen expliziten `clawhub:<package>`-Locator:

```bash
openclaw plugins install clawhub:openclaw-codex-app-server
openclaw plugins install clawhub:openclaw-codex-app-server@1.2.3
```

Einfache npm-sichere Plugin-Spezifikationen werden während der Umstellung beim Start standardmäßig von npm installiert, sofern sie nicht mit einer offiziellen Plugin-ID übereinstimmen:

```bash
openclaw plugins install openclaw-codex-app-server
```

Verwenden Sie `npm:`, um eine ausschließliche npm-Auflösung explizit festzulegen:

```bash
openclaw plugins install npm:openclaw-codex-app-server
openclaw plugins install npm:@openclaw/discord@2026.5.20
openclaw plugins install npm:@scope/plugin-name@1.0.1
```

OpenClaw prüft vor der Installation die angegebene Kompatibilität mit der Plugin-API bzw. die Mindestkompatibilität mit dem Gateway. Wenn die ausgewählte ClawHub-Version ein ClawPack-Artefakt veröffentlicht, lädt OpenClaw das versionierte npm-pack-Archiv `.tgz` herunter, überprüft den ClawHub-Digest-Header und den Artefakt-Digest und installiert es anschließend über den normalen Archivpfad. Ältere ClawHub-Versionen ohne ClawPack-Metadaten werden weiterhin über den bisherigen Prüfpfad für Paketarchive installiert. Aufgezeichnete Installationen bewahren ihre ClawHub-Quellmetadaten, den Artefakttyp, die npm-Integrität, die npm-Prüfsumme, den Tarball-Namen und die ClawPack-Digest-Daten für spätere Aktualisierungen auf.
ClawHub-Installationen ohne Versionsangabe behalten eine nicht versionierte aufgezeichnete Spezifikation bei, damit `openclaw plugins update` neueren ClawHub-Versionen folgen kann; explizite Versions- oder Tag-Selektoren wie `clawhub:pkg@1.2.3` und `clawhub:pkg@beta` bleiben an diesen Selektor angeheftet.

### Marketplace-Kurzform

Verwenden Sie die Kurzform `plugin@marketplace`, wenn der Marketplace-Name im lokalen Registry-Cache von Claude unter `~/.claude/plugins/known_marketplaces.json` vorhanden ist:

```bash
openclaw plugins marketplace list <marketplace-name>
openclaw plugins install <plugin-name>@<marketplace-name>
```

Verwenden Sie `--marketplace`, um die Marketplace-Quelle explizit anzugeben:

```bash
openclaw plugins install <plugin-name> --marketplace <marketplace-name>
openclaw plugins install <plugin-name> --marketplace <owner/repo>
openclaw plugins install <plugin-name> --marketplace https://github.com/<owner>/<repo>
openclaw plugins install <plugin-name> --marketplace ./my-marketplace
```

<Tabs>
  <Tab title="Marketplace-Quellen">
    - ein Claude-Name für einen bekannten Marketplace aus `~/.claude/plugins/known_marketplaces.json`
    - ein lokales Marketplace-Stammverzeichnis oder ein `marketplace.json`-Pfad
    - eine Kurzform für ein GitHub-Repository wie `owner/repo`
    - eine GitHub-Repository-URL wie `https://github.com/owner/repo`
    - eine Git-URL

  </Tab>
  <Tab title="Regeln für Remote-Marketplaces">
    Bei Remote-Marketplaces, die von GitHub oder über Git geladen werden, müssen Plugin-Einträge innerhalb des geklonten Marketplace-Repositorys verbleiben. OpenClaw akzeptiert Quellen mit relativen Pfaden aus diesem Repository und lehnt HTTP(S)-, absolute Pfad-, Git-, GitHub- und andere Nicht-Pfad-Plugin-Quellen aus Remote-Manifesten ab.
  </Tab>
</Tabs>

Bei lokalen Pfaden und Archiven erkennt OpenClaw automatisch:

- native OpenClaw-Plugins (`openclaw.plugin.json`)
- Codex-kompatible Bundles (`.codex-plugin/plugin.json`)
- Claude-kompatible Bundles (`.claude-plugin/plugin.json` oder das standardmäßige Komponentenlayout von Claude, wenn diese Manifestdatei fehlt)
- Cursor-kompatible Bundles (`.cursor-plugin/plugin.json`)

Verwaltete lokale Installationen müssen Plugin-Verzeichnisse oder Archive sein. Eigenständige Plugin-Dateien vom Typ `.js`,
`.mjs`, `.cjs` und `.ts` werden von `plugins install` nicht in das verwaltete
Plugin-Stammverzeichnis kopiert und auch nicht geladen, wenn sie direkt in
`~/.openclaw/extensions` oder `<workspace>/.openclaw/extensions` abgelegt werden; diese
automatisch erkannten Stammverzeichnisse laden Plugin-Paket- oder Bundle-Verzeichnisse und überspringen
Skriptdateien der obersten Ebene als lokale Hilfsdateien. Führen Sie eigenständige Dateien stattdessen explizit in
`plugins.load.paths` auf.

<Note>
Kompatible Bundles werden im normalen Plugin-Stammverzeichnis installiert und durchlaufen denselben Ablauf zum Auflisten, Anzeigen von Informationen, Aktivieren und Deaktivieren. Derzeit werden Bundle-Skills, Claude-Befehls-Skills, Claude-Standardwerte für `settings.json`, Claude-Standardwerte für `.lsp.json` bzw. im Manifest deklarierte Standardwerte für `lspServers`, Cursor-Befehls-Skills und kompatible Codex-Hook-Verzeichnisse unterstützt; andere erkannte Bundle-Funktionen werden in Diagnose/Informationen angezeigt, sind jedoch noch nicht mit der Laufzeitausführung verbunden.
</Note>

Verwenden Sie `-l`/`--link`, um ohne Kopieren auf ein lokales Plugin-Verzeichnis zu verweisen (fügt es
zu `plugins.load.paths` hinzu):

```bash
openclaw plugins install -l ./my-plugin
```

`--link` wird bei Installationen mit `--marketplace` oder `git:` nicht unterstützt und
erfordert einen bereits vorhandenen lokalen Pfad. Übergeben Sie für eine nicht interaktive lokale Verknüpfung
nach Prüfung der Quelle `--force`; dies bestätigt die Herkunft, kopiert oder überschreibt das verknüpfte
Verzeichnis jedoch nicht.

<Note>
Aus dem Workspace stammende Plugins, die in einem Erweiterungs-Stammverzeichnis des Workspace erkannt werden, werden erst
importiert oder ausgeführt, nachdem sie explizit aktiviert wurden. Führen Sie für die lokale Entwicklung
`openclaw plugins enable <plugin-id>` aus oder setzen Sie
`plugins.entries.<plugin-id>.enabled: true`; wenn Ihre Konfiguration
`plugins.allow` verwendet, nehmen Sie dort ebenfalls dieselbe Plugin-ID auf. Diese ausfallsichere Regel
gilt auch, wenn die Kanaleinrichtung ein aus dem Workspace stammendes Plugin ausdrücklich für das
ausschließliche Laden zur Einrichtung auswählt. Daher wird der Einrichtungscode eines lokalen Kanal-Plugins nicht ausgeführt, solange dieses
Workspace-Plugin deaktiviert oder von der Zulassungsliste ausgeschlossen bleibt. Verknüpfte Installationen
und explizite Einträge in `plugins.load.paths` folgen der normalen Richtlinie für ihren
aufgelösten Plugin-Ursprung. Siehe
[Plugin-Richtlinie konfigurieren](/de/tools/plugin#configure-plugin-policy)
und [Konfigurationsreferenz](/de/gateway/configuration-reference#plugins).

Verwenden Sie `--pin` bei npm-Installationen, um die aufgelöste exakte Spezifikation (`name@version`) im verwalteten Plugin-Index zu speichern, während das Standardverhalten nicht angeheftet bleibt.
</Note>

## Auflisten

```bash
openclaw plugins list
openclaw plugins list --enabled
openclaw plugins list --verbose
openclaw plugins list --json
```

<ParamField path="--enabled" type="boolean">
  Nur aktivierte Plugins anzeigen.
</ParamField>
<ParamField path="--verbose" type="boolean">
  Von der Tabellenansicht zu Detailzeilen pro Plugin mit Metadaten zu Format/Quelle/Ursprung/Version/Aktivierung wechseln.
</ParamField>
<ParamField path="--json" type="boolean">
  Maschinenlesbares Inventar sowie Registry-Diagnosen und Installationsstatus der Paketabhängigkeiten.
</ParamField>

<Note>
`plugins list` liest zuerst die persistierte lokale Plugin-Registry und verwendet eine ausschließlich aus dem Manifest abgeleitete Ausweichlösung, wenn die Registry fehlt oder ungültig ist. Dies ist nützlich, um zu prüfen, ob ein Plugin installiert, aktiviert und für die Planung eines Kaltstarts sichtbar ist, stellt jedoch keine Live-Laufzeitprüfung eines bereits ausgeführten Gateway-Prozesses dar. Starten Sie nach Änderungen an Plugin-Code, Aktivierung, Hook-Richtlinie oder `plugins.load.paths` das Gateway neu, das den Kanal bereitstellt, bevor Sie erwarten, dass neuer `register(api)`-Code oder neue Hooks ausgeführt werden. Stellen Sie bei Remote-/Container-Bereitstellungen sicher, dass Sie den tatsächlichen untergeordneten `openclaw gateway run`-Prozess neu starten und nicht nur einen Wrapper-Prozess.

`plugins list --json` enthält den jeweiligen `dependencyStatus` jedes Plugins aus `package.json`,
`dependencies` und `optionalDependencies`. OpenClaw prüft, ob diese Paketnamen
entlang des normalen Node-`node_modules`-Suchpfads des Plugins vorhanden sind; es
importiert weder Plugin-Laufzeitcode noch führt es einen Paketmanager aus oder repariert
fehlende Abhängigkeiten.
</Note>

Wenn beim Start `plugins.allow is empty; discovered non-bundled plugins may auto-load: ...` protokolliert wird,
führen Sie `openclaw plugins list --enabled --verbose` oder
`openclaw plugins inspect <id>` mit einer aufgeführten Plugin-ID aus, um die Plugin-
IDs zu bestätigen, und kopieren Sie vertrauenswürdige IDs nach `plugins.allow` in `openclaw.json`. Wenn die
Warnung alle erkannten Plugins aufführen kann, gibt sie ein direkt einfügbares
`plugins.allow`-Snippet aus, das diese IDs bereits enthält. Wenn ein Plugin
ohne Herkunftsnachweis für Installation/Ladepfad geladen wird, prüfen Sie diese Plugin-ID und verankern Sie
anschließend entweder die vertrauenswürdige ID in `plugins.allow` oder installieren Sie das Plugin erneut aus einer vertrauenswürdigen Quelle,
damit OpenClaw den Installationsherkunftsnachweis erfasst.

Binden Sie für Arbeiten an gebündelten Plugins innerhalb eines paketierten Docker-Images das Plugin-
Quellverzeichnis über dem entsprechenden paketierten Quellpfad ein, beispielsweise
`/app/extensions/synology-chat`. OpenClaw erkennt dieses eingebundene Quell-Overlay
vor `/app/dist/extensions/synology-chat`; ein lediglich kopiertes Quellverzeichnis
bleibt inaktiv, sodass normale paketierte Installationen weiterhin die kompilierte Distribution verwenden.

Zur Fehlersuche bei Laufzeit-Hooks:

- `openclaw plugins inspect <id> --runtime --json` zeigt registrierte Hooks und Diagnosen aus einem Prüfungsdurchlauf mit geladenem Modul. Die Laufzeitprüfung installiert niemals Abhängigkeiten; verwenden Sie `openclaw doctor --fix`, um veralteten Abhängigkeitsstatus zu bereinigen oder fehlende herunterladbare Plugins wiederherzustellen, auf die in der Konfiguration verwiesen wird.
- `openclaw gateway status --deep --require-rpc` bestätigt die erreichbare Gateway-URL bzw. das Profil, Hinweise zu Dienst/Prozess, den Konfigurationspfad und den RPC-Zustand.
- Nicht gebündelte Konversations-Hooks (`llm_input`, `llm_output`, `before_model_resolve`, `before_agent_reply`, `before_agent_run`, `before_agent_finalize`, `agent_end`) erfordern `plugins.entries.<id>.hooks.allowConversationAccess=true`.

### Plugin-Index

Plugin-Installationsmetadaten sind maschinell verwalteter Status und keine Benutzerkonfiguration. Installationen und Aktualisierungen schreiben sie in die gemeinsame SQLite-Statusdatenbank unter dem aktiven OpenClaw-Statusverzeichnis. Die `installed_plugin_index`-Zeile speichert dauerhafte `installRecords`-Metadaten, einschließlich Datensätzen für fehlerhafte oder fehlende Plugin-Manifeste, sowie einen aus Manifesten abgeleiteten Cache der Kaltstart-Registry, der von `openclaw plugins update`, der Deinstallation, der Diagnose und der Plugin-Registry für Kaltstarts verwendet wird.

`plugins.installs` ist eine eingestellte Oberfläche für manuell erstellte Konfigurationen. Laufzeit- und Aktualisierungsbefehle lesen ausschließlich den SQLite-Index der installierten Plugins. Führen Sie `openclaw doctor --fix` aus, um alte Konfigurationsdatensätze in den Index zu importieren und den eingestellten Schlüssel vor der normalen Laufzeitnutzung zu entfernen.

## Deinstallation

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
openclaw plugins uninstall <id> --force
```

`uninstall` entfernt Plugin-Datensätze aus `plugins.entries`, dem persistierten Plugin-Index, Einträge in Plugin-Zulassungs-/Sperrlisten und gegebenenfalls verknüpfte `plugins.load.paths`-Einträge. Sofern `--keep-files` nicht gesetzt ist, entfernt die Deinstallation außerdem das erfasste verwaltete Installationsverzeichnis, jedoch nur, wenn es innerhalb des Plugin-Erweiterungsstammverzeichnisses von OpenClaw aufgelöst wird. Wenn das Plugin derzeit den `memory`- oder `contextEngine`-Slot belegt, wird dieser Slot auf seinen Standardwert zurückgesetzt (`memory-core` für den Speicher, `legacy` für die Kontext-Engine).

`uninstall` gibt eine Vorschau der zu entfernenden Elemente aus und fragt anschließend `Uninstall plugin "<id>"?` ab, bevor Änderungen vorgenommen werden. Übergeben Sie `--force`, um die Bestätigungsabfrage zu überspringen (nützlich für Skripte und nicht interaktive Ausführungen); ohne diese Option erfordert die Deinstallation ein interaktives TTY. `--dry-run` gibt dieselbe Vorschau aus und beendet den Vorgang, ohne eine Abfrage anzuzeigen oder Änderungen vorzunehmen.

<Note>
`--keep-config` wird als veralteter Alias für `--keep-files` unterstützt.
</Note>

## Aktualisierung

```bash
openclaw plugins update <id-or-npm-spec>
openclaw plugins update --all
openclaw plugins update <id-or-npm-spec> --dry-run
openclaw plugins update @openclaw/voice-call
openclaw plugins update @acme/demo
openclaw plugins update openclaw-codex-app-server --acknowledge-clawhub-risk
openclaw plugins update openclaw-codex-app-server --dangerously-force-unsafe-install
```

Aktualisierungen gelten für erfasste Plugin-Installationen im verwalteten Plugin-Index und erfasste Hook-Paket-Installationen im gemeinsamen SQLite-Status. Sie verwenden erneut die Quelle, die der Benutzer bereits bei der Installation des Plugins ausgewählt hat, und erfordern daher keine zweite Bestätigung der Quelle.

<AccordionGroup>
  <Accordion title="Auflösen von Plugin-ID und npm-Spezifikation">
    Wenn Sie eine Plugin-ID übergeben, verwendet OpenClaw die für dieses Plugin erfasste Installationsspezifikation erneut. Das bedeutet, dass zuvor gespeicherte Dist-Tags wie `@beta` und exakt verankerte Versionen bei späteren `update <id>`-Ausführungen weiterhin verwendet werden.

    Während `update <id> --dry-run` bleiben exakt verankerte npm-Installationen verankert. Wenn OpenClaw außerdem die Standardlinie der Registry für das Paket auflösen kann und diese Standardlinie neuer als die installierte verankerte Version ist, meldet der Testlauf die Verankerung und gibt den expliziten `@latest`-Paketaktualisierungsbefehl aus, mit dem der Standardlinie der Registry gefolgt werden kann.

    Diese Regel für gezielte Aktualisierungen unterscheidet sich vom Massenwartungspfad `openclaw plugins update --all`. Massenaktualisierungen berücksichtigen weiterhin gewöhnliche erfasste Installationsspezifikationen, vertrauenswürdige offizielle OpenClaw-Plugin-Datensätze können jedoch mit dem aktuellen Ziel des offiziellen Katalogs synchronisiert werden, statt bei einem veralteten exakten offiziellen Paket zu verbleiben. Verwenden Sie gezielt `update <id>`, wenn Sie eine exakte oder mit einem Tag versehene offizielle Spezifikation bewusst unverändert beibehalten möchten.

    Bei npm-Installationen können Sie außerdem eine explizite npm-Paketspezifikation mit einem Dist-Tag oder einer exakten Version übergeben. OpenClaw löst diesen Paketnamen zum erfassten Plugin-Datensatz zurück auf, aktualisiert das installierte Plugin und erfasst die neue npm-Spezifikation für zukünftige ID-basierte Aktualisierungen.

    Wenn Sie den npm-Paketnamen ohne Version oder Tag übergeben, wird er ebenfalls zum erfassten Plugin-Datensatz zurück aufgelöst. Verwenden Sie dies, wenn ein Plugin auf eine exakte Version festgelegt war und Sie es wieder auf die Standard-Veröffentlichungslinie der Registry umstellen möchten.

  </Accordion>
  <Accordion title="Aktualisierungen im Beta-Kanal">
    Das gezielte `openclaw plugins update <id-or-npm-spec>` verwendet die erfasste Plugin-Spezifikation erneut, sofern Sie keine neue Spezifikation übergeben. Das gebündelte `openclaw plugins update --all` verwendet das konfigurierte `update.channel`, wenn es vertrauenswürdige offizielle Plugin-Datensätze mit dem offiziellen Katalogziel synchronisiert, sodass Installationen im Beta-Kanal in der Beta-Veröffentlichungslinie verbleiben können, statt stillschweigend auf stable/latest normalisiert zu werden.

    `openclaw update` kennt außerdem den aktiven OpenClaw-Aktualisierungskanal: Im Beta-Kanal versuchen npm- und ClawHub-Plugin-Datensätze der Standardlinie zuerst `@beta`. Sie greifen auf die erfasste Standard-/latest-Spezifikation zurück, wenn keine Beta-Veröffentlichung des Plugins vorhanden ist; npm-Plugins greifen außerdem zurück, wenn das Beta-Paket vorhanden ist, aber die Installationsvalidierung nicht besteht. Dieser Rückgriff wird als Warnung gemeldet und lässt die Kernaktualisierung nicht fehlschlagen. Exakte Versionen und explizite Tags bleiben bei gezielten Aktualisierungen an diesen Selektor gebunden.

  </Accordion>
  <Accordion title="Versionsprüfungen und Integritätsabweichungen">
    Vor einer Live-npm-Aktualisierung prüft OpenClaw die installierte Paketversion anhand der Metadaten der npm-Registry. Wenn die installierte Version und die erfasste Artefaktidentität bereits mit dem aufgelösten Ziel übereinstimmen, wird die Aktualisierung übersprungen, ohne `openclaw.json` herunterzuladen, neu zu installieren oder neu zu schreiben.

    Wenn ein gespeicherter Integritäts-Hash vorhanden ist und sich der Hash des abgerufenen Artefakts ändert, behandelt OpenClaw dies als npm-Artefaktabweichung. Der interaktive Befehl `openclaw plugins update` gibt den erwarteten und den tatsächlichen Hash aus und fordert vor dem Fortfahren eine Bestätigung an. Nicht interaktive Aktualisierungshilfen brechen sicher ab, sofern der Aufrufer keine explizite Fortsetzungsrichtlinie bereitstellt.

  </Accordion>
  <Accordion title="--dangerously-force-unsafe-install bei Aktualisierungen">
    `--dangerously-force-unsafe-install` wird aus Kompatibilitätsgründen auch bei `plugins update` akzeptiert, ist jedoch veraltet und ändert das Verhalten bei Plugin-Aktualisierungen nicht mehr. Betreiberseitiges `security.installPolicy` kann Aktualisierungen weiterhin blockieren; Plugin-`before_install`-Hooks gelten nur in Prozessen, in denen Plugin-Hooks geladen sind.
  </Accordion>
  <Accordion title="--acknowledge-clawhub-risk bei Aktualisierungen">
    Community-Plugin-Aktualisierungen auf ClawHub-Basis durchlaufen vor dem Herunterladen des Ersatzpakets dieselbe Vertrauensprüfung der exakten Veröffentlichung wie Installationen. Verwenden Sie `--acknowledge-clawhub-risk` für geprüfte Automatisierungen, die fortfahren sollen, wenn für die ausgewählte ClawHub-Veröffentlichung eine riskante Vertrauenswarnung vorliegt. Offizielle ClawHub-Pakete und gebündelte OpenClaw-Plugin-Quellen umgehen diese Vertrauensabfrage für Veröffentlichungen.
  </Accordion>
</AccordionGroup>

## Prüfen

```bash
openclaw plugins inspect <id>
openclaw plugins inspect <id> --runtime
openclaw plugins inspect <id> --json
openclaw plugins inspect --all
```

Die Prüfung zeigt Identität, Ladestatus, Quelle, Manifestfunktionen, Richtlinienflags, Diagnosen, Installationsmetadaten, Paketfunktionen und jegliche erkannte Unterstützung für MCP- oder LSP-Server, ohne standardmäßig die Plugin-Laufzeit zu importieren. Die JSON-Ausgabe enthält die Verträge des Plugin-Manifests, beispielsweise `contracts.agentToolResultMiddleware` und `contracts.trustedToolPolicies`, sodass Betreiber Deklarationen vertrauenswürdiger Oberflächen prüfen können, bevor sie ein Plugin aktivieren oder neu starten. Fügen Sie `--runtime` hinzu, um das Plugin-Modul zu laden und registrierte Hooks, Werkzeuge, Befehle, Dienste, Gateway-Methoden und HTTP-Routen einzubeziehen. Die Laufzeitprüfung meldet fehlende Plugin-Abhängigkeiten direkt; Installationen und Reparaturen verbleiben in `openclaw plugins install`, `openclaw plugins update` und `openclaw doctor --fix`.

Plugin-eigene CLI-Befehle werden üblicherweise als `openclaw`-Befehlsgruppen auf der obersten Ebene installiert, Plugins können jedoch auch verschachtelte Befehle unter einem Kern-Elternelement wie `openclaw nodes` registrieren. Nachdem `inspect --runtime` einen Befehl unter `cliCommands` anzeigt, führen Sie ihn unter dem aufgeführten Pfad aus; beispielsweise kann ein Plugin, das `demo-git` registriert, mit `openclaw demo-git ping` überprüft werden.

Jedes Plugin wird danach klassifiziert, was es tatsächlich zur Laufzeit registriert:

| Form                | Bedeutung                                                                    |
| ------------------- | ---------------------------------------------------------------------------- |
| `plain-capability`  | genau ein Funktionstyp (z. B. ein reines Provider-Plugin)                    |
| `hybrid-capability` | mehr als ein Funktionstyp (z. B. Text + Sprache + Bilder)                    |
| `hook-only`         | nur Hooks, keine Funktionen, Werkzeuge, Befehle, Dienste oder Routen         |
| `non-capability`    | Werkzeuge/Befehle/Dienste, aber keine Funktionen                            |

Weitere Informationen zum Funktionsmodell finden Sie unter [Plugin-Formen](/de/plugins/architecture#plugin-shapes).

<Note>
Das Flag `--json` gibt einen maschinenlesbaren Bericht aus, der für Skripterstellung und Audits geeignet ist. `inspect --all` stellt eine systemweite Tabelle mit Spalten für Form, Funktionsarten, Kompatibilitätshinweise, Paketfunktionen und Hook-Zusammenfassung dar. `info` ist ein Alias für `inspect`.
</Note>

## Diagnose

```bash
openclaw plugins doctor
```

`doctor` meldet Fehler beim Laden von Plugins, Manifest-/Discovery-Diagnosen, Kompatibilitätshinweise und veraltete Plugin-Konfigurationsreferenzen wie fehlende Plugin-Slots. Wenn der Installationsbaum und die Plugin-Konfiguration sauber sind, gibt es `No plugin issues detected.` aus. Wenn weiterhin veraltete Konfiguration vorhanden ist, der Installationsbaum jedoch ansonsten fehlerfrei ist, weist die Zusammenfassung darauf hin, statt einen vollständig fehlerfreien Plugin-Zustand zu suggerieren.

Wenn ein konfiguriertes Plugin auf dem Datenträger vorhanden ist, aber durch die Pfadsicherheitsprüfungen des Loaders blockiert wird, behält die Konfigurationsvalidierung den Plugin-Eintrag bei und meldet ihn als `present but blocked`. Beheben Sie die vorherige Diagnose zum blockierten Plugin, etwa hinsichtlich des Pfadeigentümers oder der Berechtigungen für weltweiten Schreibzugriff, statt die Konfiguration `plugins.entries.<id>` oder `plugins.allow` zu entfernen.

Führen Sie den Befehl bei Fehlern der Modulstruktur, etwa fehlenden Exporten `register`/`activate`, erneut mit `OPENCLAW_PLUGIN_LOAD_DEBUG=1` aus, um eine kompakte Zusammenfassung der Exportstruktur in die Diagnoseausgabe aufzunehmen.

## Registry

```bash
openclaw plugins registry
openclaw plugins registry --refresh
openclaw plugins registry --json
```

Die lokale Plugin-Registry ist das persistierte Cold-Read-Modell von OpenClaw für die Identität installierter Plugins, deren Aktivierungsstatus, Quellmetadaten und die Zuständigkeit für Beiträge. Der normale Startvorgang, die Suche nach dem zuständigen Provider, die Klassifizierung der Kanaleinrichtung und die Plugin-Bestandsaufnahme können darauf zugreifen, ohne Plugin-Laufzeitmodule zu importieren.

Verwenden Sie `plugins registry`, um zu prüfen, ob die persistierte Registry vorhanden, aktuell oder veraltet ist. Verwenden Sie `--refresh`, um sie aus dem persistierten Plugin-Index, der Konfigurationsrichtlinie und den Manifest-/Paketmetadaten neu zu erstellen. Dies ist ein Reparaturpfad und kein Pfad zur Laufzeitaktivierung.

`openclaw doctor --fix` behebt außerdem Abweichungen bei verwaltetem npm im Umfeld der Registry. Wenn ein verwaistes oder wiederhergestelltes `@openclaw/*`-Paket unter einem verwalteten Plugin-npm-Projekt oder im veralteten flachen Stammverzeichnis für verwaltetes npm ein gebündeltes Plugin überschattet, entfernt Doctor dieses veraltete Paket und erstellt die Registry neu, sodass der Startvorgang anhand des gebündelten Manifests validiert. Wenn ein maßgeblicher Installationsdatensatz eine verwaltete Generation auswählt, aber ältere flache Verzeichnisse oder Generationsverzeichnisse verbleiben, nimmt Doctor diese veralteten Verzeichnisbäume außer Betrieb, damit sie nach dem Neustart des Gateways bereinigt werden können. Doctor verknüpft außerdem das `openclaw`-Paket des Hosts erneut mit verwalteten npm-Plugins, die `peerDependencies.openclaw` deklarieren, sodass paketlokale Laufzeitimporte wie `openclaw/plugin-sdk/*` nach Aktualisierungen oder npm-Reparaturen aufgelöst werden.

## Marketplace

```bash
openclaw plugins marketplace entries
openclaw plugins marketplace entries --offline
openclaw plugins marketplace entries --json
openclaw plugins marketplace entries --feed-profile <name>
openclaw plugins marketplace entries --feed-url <url>
openclaw plugins marketplace list <source>
openclaw plugins marketplace list <source> --json
openclaw plugins marketplace refresh
openclaw plugins marketplace refresh --feed-profile <name>
openclaw plugins marketplace refresh --feed-url <url>
openclaw plugins marketplace refresh --expected-sha256 <sha256> --json
```

`plugins marketplace entries` listet Einträge aus dem konfigurierten OpenClaw-Marketplace-Feed auf. Standardmäßig versucht der Befehl, den gehosteten Feed abzurufen, und greift ersatzweise auf den zuletzt akzeptierten Snapshot oder die gebündelten Daten zurück. Verwenden Sie `--feed-profile <name>`, um ein bestimmtes konfiguriertes Profil zu lesen, `--feed-url <url>`, um eine explizite URL eines gehosteten Feeds zu lesen, und `--offline`, um den zuletzt akzeptierten Snapshot zu lesen, ohne den Feed abzurufen.

`plugins marketplace refresh` aktualisiert den konfigurierten Snapshot des gehosteten Feeds und meldet, ob OpenClaw gehostete Daten, einen gehosteten Snapshot oder gebündelte Ausweichdaten akzeptiert hat. Verwenden Sie `--expected-sha256`, wenn der Befehl für einen Aufrufer fehlschlagen soll, sofern keine aktuelle gehostete Nutzlast mit einer festgelegten Prüfsumme übereinstimmt.

Marketplace `list` akzeptiert einen lokalen Marketplace-Pfad, einen `marketplace.json`-Pfad, eine GitHub-Kurzform wie `owner/repo`, eine GitHub-Repository-URL oder eine Git-URL. `--json` gibt die Bezeichnung der aufgelösten Quelle sowie das geparste Marketplace-Manifest und die Plugin-Einträge aus.

Die Marketplace-Aktualisierung lädt einen gehosteten OpenClaw-Marketplace-Feed und persistiert die
validierte Antwort als lokalen Snapshot des gehosteten Feeds. Ohne Optionen verwendet sie
das konfigurierte Standard-Feed-Profil. Verwenden Sie `--feed-profile <name>`, um ein
bestimmtes konfiguriertes Profil zu aktualisieren, `--feed-url <url>`, um eine explizite URL eines gehosteten
Feeds zu aktualisieren, `--expected-sha256 <sha256>`, um eine übereinstimmende Prüfsumme der Nutzlast zu verlangen
(`sha256:<hex>` oder einen reinen 64-stelligen Hexadezimal-Digest), und `--json` für
maschinenlesbare Ausgabe. Explizite URLs gehosteter Feeds dürfen keine
Anmeldedaten, Abfragezeichenfolgen oder Fragmente enthalten. Aktualisierungen ohne festgelegte Prüfsumme können ein
Ergebnis aus einem gehosteten Snapshot oder aus gebündelten Ausweichdaten melden, ohne dass der Befehl fehlschlägt. Aktualisierungen
mit festgelegter Prüfsumme schlagen fehl, sofern sie keine aktuelle gehostete Nutzlast akzeptieren, und erfolgreiche gehostete
Aktualisierungen schlagen fehl, wenn OpenClaw den validierten Snapshot nicht persistieren kann.

Das integrierte Profil `clawhub-public` erwartet die Nutzlastidentität
`clawhub-official`. OpenClaw wird den öffentlichen Produktionsschlüssel von ClawHub bündeln, nachdem
ClawHub diesen Schlüssel generiert und übergeben hat. Bis dahin erteilt das integrierte Profil
keine Installationsberechtigung für signierte Feeds. Öffentliche Schlüssel müssen aus einem vertrauenswürdigen
Release- oder Betreiberkanal stammen, nicht von einem Schlüssel-Endpunkt auf dem Feed-Host.

OpenClaw verifiziert die DSSE-Hülle und verlangt, wenn ein Profil `feedId`
deklariert, dass die dekodierte Nutzlast-ID damit übereinstimmt. Das integrierte Profil `clawhub-public`
deklariert stets seine Identität und verhindert so, dass ein gültiges Dokument eines anderen
Feeds über dieses Profil erneut eingespielt wird.

Während der stufenweisen Einführung behalten bestehende benutzerdefinierte signierte Profile, die `feedId`
weglassen, die Signaturprüfung ohne Bindung an die Nutzlastidentität bei. Neue benutzerdefinierte
Profile sollten `feedId` deklarieren. Die Konfigurationsoberfläche für Feed-Profile wird
separat mit den für die Control UI erforderlichen Darstellungsmetadaten eingeführt; ihre
Doctor-Diagnose muss den Betreiber auffordern, eine fehlende Identität anzugeben, und darf
keine aus der Feed-URL ableiten. Diese Vertrauensbindung stellt den außer Betrieb genommenen
Root-Schlüssel `marketplaces` nicht wieder her.

## Verwandte Themen

- [Plugins erstellen](/de/plugins/building-plugins)
- [CLI-Referenz](/de/cli)
- [ClawHub](/de/clawhub)
