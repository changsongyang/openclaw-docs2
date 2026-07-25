---
read_when:
    - Sie warten ein OpenClaw-Plugin.
    - Sie sehen eine Plugin-Kompatibilitätswarnung
    - Sie planen eine Migration des Plugin-SDKs oder Manifests
summary: Plugin-Kompatibilitätsverträge, Metadaten zu veralteten Funktionen und Migrationserwartungen
title: Plugin-Kompatibilität
x-i18n:
    generated_at: "2026-07-24T20:30:54Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 80cf1dfce9e0538e78138ff80a6807ee36267a07d3eee6f19bd8e56e5c0c9cd3
    source_path: plugins/compatibility.md
    workflow: 16
---

OpenClaw bindet ältere Plugin-Verträge über benannte Kompatibilitätsadapter
weiterhin ein, bevor sie entfernt werden. Dies schützt bestehende gebündelte und externe
Plugins, während sich die Verträge für SDK, Manifest, Einrichtung, Konfiguration und Agent-Laufzeit
weiterentwickeln.

## Kompatibilitätsregister

Plugin-Kompatibilitätsverträge werden im Kernregister unter
`src/plugins/compat/registry.ts` erfasst. Jeder Eintrag enthält:

- einen stabilen Kompatibilitätscode
- Status: `active`, `deprecated`, `removal-pending` oder `removed`
- Verantwortlicher: `sdk`, `config`, `setup`, `channel`, `provider`, `plugin-execution`,
  `agent-runtime` oder `core`
- Einführungs- und Einstellungsdaten, sofern zutreffend
- ein genaues Entfernungsdatum, sobald der zuständige Maintainer es genehmigt; ein fehlendes
  `removeAfter` bewirkt, dass eine eingestellte Oberfläche nicht entfernt werden darf
- Hinweise zum Ersatz
- Dokumentation, Diagnosen und Tests, die das alte und neue Verhalten abdecken

Das Register dient als Grundlage für die Planung der Maintainer und zukünftige Prüfungen durch den Plugin-
Inspektor. Wenn sich ein Plugin-bezogenes Verhalten ändert, fügen Sie den
Kompatibilitätseintrag in derselben Änderung hinzu oder aktualisieren Sie ihn, mit der auch der Adapter hinzugefügt wird.

Kompatibilität für Doctor-Reparaturen und -Migrationen wird separat unter
`src/commands/doctor/shared/deprecation-compat.ts` erfasst. Diese Einträge decken alte
Konfigurationsformen, Installationsprotokoll-Layouts und Reparatur-Shims ab, die möglicherweise
auch nach Entfernung des Laufzeit-Kompatibilitätspfads verfügbar bleiben müssen.

Release-Prüfungen sollten beide Register prüfen. Löschen Sie eine Doctor-
Migration nicht nur deshalb, weil der zugehörige Laufzeit- oder Konfigurationskompatibilitätseintrag
abgelaufen ist; vergewissern Sie sich zunächst, dass kein unterstützter Upgrade-Pfad die
Reparatur weiterhin benötigt. Validieren Sie bei der Release-Planung außerdem jede Ersatzangabe
erneut, da sich die Plugin-Zuständigkeit und der Konfigurationsumfang ändern können, wenn Provider
und Kanäle aus dem Kern verlagert werden.

## Einstellungsrichtlinie

OpenClaw sollte einen dokumentierten Plugin-Vertrag nicht in demselben Release
entfernen, in dem sein Ersatz eingeführt wird. Migrationsreihenfolge:

1. Fügen Sie den neuen Vertrag hinzu.
2. Binden Sie das alte Verhalten weiterhin über einen benannten Kompatibilitätsadapter ein.
3. Geben Sie Diagnosen oder Warnungen aus, sobald Plugin-Autoren handeln können.
4. Dokumentieren Sie den Ersatz und den Zeitplan.
5. Testen Sie sowohl den alten als auch den neuen Pfad.
6. Warten Sie das angekündigte Migrationsfenster ab.
7. Entfernen Sie den Vertrag nur mit ausdrücklicher Genehmigung für ein inkompatibles Release.

Eingestellte Einträge müssen ein Startdatum für Warnungen, einen Ersatz, einen Dokumentations-
link und ein endgültiges Entfernungsdatum enthalten, das höchstens drei Monate nach Beginn der Warnungen
liegt. Fügen Sie keinen eingestellten Kompatibilitätspfad mit einem unbefristeten
Entfernungsfenster hinzu, es sei denn, die Maintainer entscheiden ausdrücklich, dass es sich um dauerhafte
Kompatibilität handelt, und kennzeichnen ihn stattdessen als `active`.

## Aktuelle Kompatibilitätsbereiche

Bei der Prüfung im Juli 2026 wurden die abgelaufenen Aliasse für Root-SDK, Manifest, Provider, Laufzeit,
Register-Flags und Plugin-eigene Webkonfiguration entfernt. Doctor-Migrationen bleiben
separat erfasst, damit unterstützte Upgrade-Pfade alte Konfigurationen weiterhin reparieren können.

Die verbleibenden datierten Kompatibilitätsbereiche sind:

- die im Migrationsleitfaden aufgeführten SDK-Unterpfadfenster für August und September
- die Hook-Aliasse `api.on("deactivate", ...)` und `api.on("subagent_spawning", ...)`
- speicherspezifische Embedding-Registrierung und die Sitzungsspeicher-Bridge aus beta.5
- die unten beschriebenen Aliasse für eingehende WhatsApp-Callbacks
- explizites Parsen von Kanalzielen und `openclaw/plugin-sdk/messaging-targets`
- eingebettete Pi-Agent-Aliasse
- die ausgelieferten SDK-Aliasse des Agent-Harness, deren Entfernung noch von einer neuen
  extern dokumentierten Migrationsentscheidung abhängt

Aktive, undatierte Registereinträge decken unterstütztes Verhalten statt
Entfernungsrückstände ab, darunter Aktivierungshinweise, Plugin-Erfassung, Aktivierung gebündelter Plugins
und der generierte Fallback für die Kanalkonfiguration.

### Flache Aliasse für eingehende WhatsApp-Callbacks

WhatsApp-Laufzeit-Callbacks liefern `WebInboundMessage`: die kanonischen
verschachtelten Kontexte `event`, `payload`, `quote`, `group` und `platform` sowie
eingestellte flache Aliasse für die ausgelieferten Callback-Felder. Neuer Callback-Code
sollte die verschachtelten Kontexte lesen. Code, der bereinigte verschachtelte Callback-
Nachrichten erstellt, kann `WebInboundCallbackMessage` verwenden; Kompatibilitäts-Listener, die
weiterhin alte flache Test- oder Plugin-Nachrichten einspeisen, sollten
`LegacyFlatWebInboundMessage` oder `WebInboundMessageInput` verwenden.

Die flachen Aliasse bleiben bis zum **2026-08-30** verfügbar; dieses Zeitfenster gilt
nur für den Zugriff über flache Aliasse, nicht für die verschachtelte Form, die den kanonischen
Laufzeitvertrag darstellt. Die TypeScript-Annotation `@deprecated` jedes flachen Alias
nennt dessen genauen verschachtelten Ersatz. Häufige Beispiele:

- `id`, `timestamp` und `isBatched` werden unter `event` verschoben.
- `body`, `mediaPath`, `mediaType`, `mediaFileName`, `mediaUrl`, `location`
  und `untrustedStructuredContext` werden unter `payload` verschoben.
- `to`, `chatId`, Absender-/Selbstfelder, `sendComposing`, `reply(...)` und
  `sendMedia(...)` werden unter `platform` verschoben.
- Felder von `replyTo*` werden unter `quote` verschoben; Felder für Gruppenbetreff, Teilnehmer und Erwähnungen
  werden unter `group` verschoben.

`payload.untrustedStructuredContext` wird aus eingehenden Provider-
Nutzdaten extrahiert. Plugins sollten `label`, `source` und `type` prüfen, bevor
sie dessen `payload` als maßgeblich behandeln.

### Zulassungsfelder für eingehende WhatsApp-Nachrichten

Akzeptierte WhatsApp-Callback-Nachrichten enthalten `admission`, eine öffentlich unbedenkliche
Hülle für die Zugriffskontrollentscheidung, durch die die Nachricht zugelassen wurde. Neuer
Callback-Code sollte Zulassungsinformationen aus `msg.admission` statt aus
den älteren Zulassungsfeldern der obersten Ebene lesen.

Die Felder der obersten Ebene bleiben bis zum **2026-08-30** verfügbar. Die
TypeScript-Annotation `@deprecated` jedes Feldes nennt dessen Ersatz:

- `from` und `conversationId` werden nach `admission.conversation.id` verschoben.
- `accountId` wird nach `admission.accountId` verschoben.
- `accessControlPassed` ist eine abgeleitete Kompatibilitätsansicht von
  `admission.ingress.decision === "allow"`; bei Nachrichten, die bereits
  `admission` enthalten, schreibt das Setzen des Legacy-Booleschen Werts den Eingangs-
  graphen nicht neu.
- `chatType` wird nach `admission.conversation.kind` verschoben.

## Plugin-Inspektorpaket

Der Plugin-Inspektor sollte außerhalb des OpenClaw-Kern-Repositorys als
separates Paket/Repository angesiedelt sein und auf den versionierten Kompatibilitäts- und
Manifestverträgen basieren. Die CLI für den ersten Tag sollte wie folgt lauten:

```sh
openclaw-plugin-inspector ./my-plugin
```

Sie sollte Manifest-/Schemavalidierung, die geprüfte Version der Vertragskompatibilität,
Prüfungen der Installations-/Quellmetadaten, Importprüfungen für selten ausgeführte Pfade
sowie Einstellungs-/Kompatibilitätswarnungen ausgeben. Verwenden Sie `--json` für stabile
maschinenlesbare Ausgaben in CI-Annotationen. Der OpenClaw-Kern sollte
Verträge und Fixtures bereitstellen, die der Inspektor verwenden kann, aber die
Inspektor-Binärdatei nicht über das Hauptpaket `openclaw` veröffentlichen.

### Abnahmestrecke für Maintainer

Verwenden Sie eine Crabbox-gestützte Blacksmith Testbox für die Abnahmestrecke
installierbarer Pakete, wenn der externe Inspektor mit OpenClaw-Plugin-
Paketen validiert wird. Führen Sie sie nach dem Erstellen des Pakets aus einem sauberen OpenClaw-Checkout aus:

```sh
pnpm crabbox:run -- --provider blacksmith-testbox --timing-json --shell -- "pnpm install && pnpm build && npm exec --yes @openclaw/plugin-inspector@0.1.0 -- ./extensions/telegram --json"
pnpm crabbox:run -- --provider blacksmith-testbox --timing-json --shell -- "npm exec --yes @openclaw/plugin-inspector@0.1.0 -- ./extensions/discord --json"
pnpm crabbox:run -- --provider blacksmith-testbox --timing-json --shell -- "npm exec --yes @openclaw/plugin-inspector@0.1.0 -- <clawhub-plugin-dir> --json"
```

Diese Strecke sollte für Maintainer optional bleiben, da sie ein externes npm-
Paket installiert und möglicherweise außerhalb des Repositorys geklonte Plugin-Pakete prüft. Die lokalen
Repository-Schutzprüfungen decken die SDK-Exportzuordnung, die Metadaten des Kompatibilitätsregisters,
den Abbau eingestellter SDK-Importe und die Importgrenzen gebündelter Erweiterungen ab;
der Testbox-Nachweis des Inspektors deckt das Paket so ab, wie externe Plugin-Autoren
es verwenden.

## Release-Hinweise

Release-Hinweise sollten bevorstehende Plugin-Einstellungen mit Zieldaten
und Links zur Migrationsdokumentation enthalten, bevor ein Kompatibilitätspfad in
`removal-pending` oder `removed` übergeht.
