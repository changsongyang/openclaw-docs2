---
read_when:
    - Änderung des Genehmigungslebenszyklus, der Speicherung, des Protokolls oder der Autorisierung für exec oder Plugins
    - Hinzufügen von Genehmigungslinks oder nativen Genehmigungssteuerelementen zu einem Kanal
    - Genehmigungen untergeordneter Sitzungen in übergeordneten oder Orchestratoransichten abbilden
summary: Konzept für dauerhafte, direkt verlinkbare Genehmigungen in der Control UI, nativen Apps, Kanälen und übergeordneten Sitzungen
title: Bedienerfreigaben über mehrere Oberflächen hinweg
x-i18n:
    generated_at: "2026-07-26T18:08:00Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 9defdaada1911df1184f64429e1787c4881e735c433d6dbc30a5946e11cc7cce
    source_path: refactor/operator-approvals.md
    workflow: 16
---

# Bedienerfreigaben über mehrere Oberflächen

Dieser Entwurf verfolgt [#103505](https://github.com/openclaw/openclaw/issues/103505). Er ersetzt die prozesslokale Freigabeautorität durch einen einzigen, dem Gateway zugeordneten und SQLite-gestützten Lebenszyklus. Jede dem Gateway zugeordnete Ausführungs- oder Plugin-/Tool-Freigabe erhält eine stabile ID, eine authentifizierte Control-UI-Route, eine atomare Auflösung nach dem Prinzip „die erste Antwort gewinnt“ sowie ausschließlich für Bediener bestimmte Projektionen in die Sitzungsstreams ihrer Quell- und übergeordneten Sitzungen.

Inline-Aktionen und Deep Links bestehen nebeneinander. Es gibt keinen Umschalter für den Freigabemodus.

## Ziele

- Ein dauerhaftes Freigabeobjekt für Ausführungs- und Plugin-/Tool-Sperren.
- Stabile `${controlUiBasePath}/approve/{approvalId}`-Route.
- Auflösung über jede autorisierte Control UI, native App oder Kanaloberfläche.
- Atomares Verhalten nach dem Prinzip „die erste Antwort gewinnt“ über gleichzeitig verwendete Oberflächen hinweg.
- Idempotente identische Wiederholungen; widersprüchliche verspätete Antworten können die siegreiche Antwort nicht überschreiben.
- Zeitüberschreitungen, fehlerhafte vertrauenswürdige Entscheidungen, fehlende Routen, Abbrüche und Neustarts schlagen sicher fehl.
- Anforderungs- und Abschlussereignisse erreichen die Quellsitzung und alle relevanten übergeordneten bzw. Orchestrator-Eigentümer.
- Kanäle erhalten typisierte Freigabe- und Navigationsaktionen; Callback-Daten des Transports bleiben kanalprivat.
- Bestehende Gateway-Methoden für Ausführungen und Plugins bleiben kompatibel, während ihre Implementierung in einem einzigen Dienst zusammengeführt wird.

## Nicht-Ziele

- Persistieren oder Fortsetzen der blockierten Tool-Ausführung selbst über einen Gateway-Neustart hinweg.
- Verwenden einer Freigabe-ID oder URL als Bearer-Zugangsdaten.
- Anhängen von Freigabeaufforderungen an für das Modell sichtbare Transkripte oder Aktivieren übergeordneter Agenten.
- Verlagern von Freigaberichtlinien, Produktbefehlen oder der Autorisierung von Prüfern in Kanal-Plugins.
- Klonen des Freigabestatus pro Kanal, Gerät oder übergeordneter Instanz.
- Neugestalten von Ausführungs-Zulassungslisten, der Zusammensetzung von Plugin-Richtlinien oder der Persistierung von `allow-always`, außer soweit dies erforderlich ist, um Endergebnisse eindeutig zu machen.
- Remote-Zugriff auf eine eingebettete TUI ohne Gateway im ersten Ausbauschritt. Sie bleibt ausschließlich lokal und muss sicher fehlschlagen, wenn kein Prüfer vorhanden ist.

## Ausgangsbasis vor der Einführung und Evidenzübersicht

Diese Tabelle dokumentiert den Implementierungsstand zum Zeitpunkt der Eröffnung von #103505. Die folgenden Einführungsabschnitte beschreiben die darauf aufbauenden Ausbauschritte für die dauerhafte Registry, typisierte Aktionen, die Deep-Link-Seite und native Clients.

| Oberfläche           | Ursprünglicher Einstiegspunkt und Eigentümer                                                                                                                                  | Ursprüngliches Verhalten und Lücke                                                                                                                                                                    |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Agentenausführung        | `src/agents/bash-tools.exec-approval-request.ts`, `src/agents/bash-tools.exec-host-shared.ts`                                                                   | Die zweiphasige Registrierung von `exec.approval.*` verhindert ein frühes Race bei `/approve`, aber eine Zeitüberschreitung kann über `askFallback` weiterhin zu einer Zulassung führen.                                                        |
| Plugin-Tool-Sperre  | `src/agents/agent-tools.before-tool-call.ts`                                                                                                                    | Fordert `plugin.approval.*` an; `timeoutBehavior: "allow"` kann eine bereits abgelaufene Sperre freigeben. Der eingebettete Modus besitzt in `src/infra/embedded-plugin-approval-broker.ts` eine separate prozesslokale Autorität. |
| Plugin-Node-Sperre  | `src/gateway/node-invoke-plugin-policy.ts`                                                                                                                      | Erstellt und überträgt direkt über den Plugin-Manager und dupliziert dadurch einen Teil des Lebenszyklus der Servermethode.                                                                                 |
| Gateway-Autorität | `src/gateway/server-aux-handlers.ts`, `src/gateway/exec-approval-manager.ts`, `src/gateway/server-methods/approval-shared.ts`                                   | Separate Manager für Ausführungen und Plugins verwenden prozesslokale Maps. Abschlusseinträge bleiben 15 Sekunden lang erhalten. Das Prinzip „die erste Antwort gewinnt“ gilt nur innerhalb eines einzelnen Prozesses.                                          |
| Gateway-Protokoll  | `packages/gateway-protocol/src/schema/exec-approvals.ts`, `packages/gateway-protocol/src/schema/plugin-approvals.ts`, `src/gateway/methods/core-descriptors.ts` | Für Ausführungen gibt es das ausschließlich ausstehende `get`; für Plugins gibt es kein `get`; für einen Deep Link existiert keine vom Typ unabhängige Suche nach Abschlussergebnissen.                                                                                   |
| Zustellung          | `src/infra/exec-approval-channel-runtime.ts`, `src/infra/approval-native-runtime.ts`, `src/infra/approval-handler-runtime.ts`                                   | Unterstützt Ursprungsrouting, Direktnachrichten an Freigabeberechtigte, Wiedergabe ausstehender Anfragen, native Handler und prozessinterne Abschlussbereinigung. Eine separate Folgeänderung ergänzt einen dauerhaften Abgleich von Abschlussergebnissen.                          |
| Portable Aktionen  | `src/interactive/payload.ts`, `src/plugin-sdk/interactive-runtime.ts`, `src/plugin-sdk/approval-reply-runtime.ts`                                               | Freigabeschaltflächen sind Befehlsaktionen, die `/approve ...` enthalten; URL- und Web-App-Ziele sind nicht typisierte Schaltflächenfelder.                                                                           |
| Telegram          | `extensions/telegram/src/approval-handler.runtime.ts`, `extensions/telegram/src/button-types.ts`                                                                | Der Renderer analysiert den Befehlstext, um die Freigabesemantik zu erkennen, bevor er private Callback-Daten erzeugt.                                                                                     |
| Control UI        | `ui/src/app/exec-approval.ts`, `ui/src/app/overlays.ts`, `ui/src/components/exec-approval.ts`                                                                   | Die Freigabeoberfläche ist ein globaler modaler Dialog. `ui/src/app-route-paths.ts` und `ui/src/app-routes.ts` verwenden exakte Routen und leiten unbekannte Pfade zu Chat um.                                                    |
| Sitzungseigentümerschaft | `src/agents/subagent-registry.types.ts`, `src/agents/subagent-registry-read.ts`, `src/config/sessions/types.ts`                                                 | Eigentümerschaft durch Controller, Anforderer, explizite übergeordnete Instanz und Legacy-Erzeugung ist vorhanden, aber Freigabeereignisse werden nicht in diese Sitzungsstreams projiziert.                                                    |
| Gemeinsamer Status      | `src/state/openclaw-state-schema.sql`, `src/state/openclaw-state-db.ts`                                                                                         | Vorhandene unmittelbare Transaktionen und bedingte Kysely-Aktualisierungen unterstützen dauerhaftes Compare-and-Set in `state/openclaw.sqlite`.                                                                   |

Zu den repräsentativen aktuellen Tests gehören `src/gateway/exec-approval-manager.test.ts`, `src/gateway/server-methods/approval-shared.test.ts`, `src/agents/bash-tools.exec-gateway-approval.e2e.test.ts`, `extensions/telegram/src/approval-handler.runtime.test.ts` und `ui/src/e2e/approval-flow.e2e.test.ts`.

Das Plugin-SDK bleibt die einzige Grenze für Kanäle und Plugins. Änderungen an Freigabelaufzeit und Darstellung müssen über die vorhandenen Unterpfade `src/plugin-sdk/approval-*.ts` und `src/plugin-sdk/interactive-runtime.ts` exportiert werden; Produktionscode von Plugins darf keine Gateway-Interna importieren.

## Bestehende Ansätze

Omnigent bietet nützliche UX- und Fehlersemantik:

- [`approval.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/runtime/policies/approval.py) hält ASK an, wendet Zeitüberschreitungen pro Richtlinie an und behandelt nur eine exakte Annahme als Freigabe.
- [`sessions.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/server/routes/sessions.py) enthält die serverseitige Sperre für das native Testsystem sowie die Projektion von Anforderungen und Auflösungen auf übergeordnete Instanzen.
- [`ApprovePage.tsx`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/web/src/pages/ApprovePage.tsx) stellt die eigenständige mobile Freigabeseite bereit.

Übernehmen Sie die Aussage zur Speicherung nicht unkritisch. Der derzeit aktive ausstehende Status ist in [`_elicitation_registry.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/server/_elicitation_registry.py) prozesslokal, und die ungenutzte Tabelle für ausstehende Einträge wird durch [`e3b1f2a4c9d7_drop_pending_tool_calls_table.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/db/migrations/versions/e3b1f2a4c9d7_drop_pending_tool_calls_table.py) entfernt. OpenClaw geht bewusst weiter: SQLite ist maßgeblich, und jeder Abschlussübergang ist ein Compare-and-Set in der Datenbank.

## Architektur und Eigentümerschaft

Das Gateway ist für den Lebenszyklus verantwortlich:

1. Ein Agent, Plugin-Hook oder eine Node-Richtlinie liefert eine typspezifische Anfrage und eine prozesslokale Ausführungsbindung.
2. Das Gateway validiert sie und erstellt eine bereinigte Projektion für Prüfer.
3. Der Freigabedienst berechnet die Zielgruppe aus Quelle und Eigentümern, fügt die kanonische Zeile ein und registriert anschließend den prozessinternen Wartenden.
4. Nach dem dauerhaften Einfügen veröffentlicht das Gateway vorhandene Freigabeereignisse, Sitzungsprojektionen, Kanalbenachrichtigungen und native Push-Nachrichten.
5. Jede Oberfläche führt die Auflösung über denselben Dienst durch.
6. Der Dienst schreibt einen Abschlussübergang fest, aktiviert den wartenden Laufzeitprozess und veröffentlicht Abschlussprojektionen.
7. Eine fehlgeschlagene Ereigniszustellung macht die festgeschriebene Entscheidung niemals rückgängig; Clients stellen den Status über `approval.get` oder die Wiedergabe der Liste wieder her.

Eigentumsgrenzen:

- `src/gateway/`: Freigabedienst, Autorisierung, RPC-Adapter, URL-Erstellung, Lebenszyklus der Wartenden und Ereignisveröffentlichung.
- `src/state/`: gemeinsames Schema und generierte Kysely-Typen.
- `src/infra/`: bereinigte Freigabe-View-Models und Erstellung portabler Darstellungen.
- `src/agents/`: fordert die Entscheidung an, wartet darauf und wendet sie an; keine Persistierung.
- `src/channels/` und `extensions/*`: stellen typisierte Aktionen dar, autorisieren Kanalbenutzer, codieren private Callbacks und aktualisieren zugestellte Steuerelemente.
- `src/plugin-sdk/`: ausschließlich öffentliche Freigabe- und Darstellungsverträge.
- `ui/`: eigenständige Seite und vorhandene Clients für Warteschlange und modalen Dialog.

Der prozessinterne Wartende ist ein Benachrichtigungsmechanismus und keine Autorität. Die Registrierung fügt die Zeile ein und installiert den Wartenden synchron, bevor die Anfrage veröffentlicht wird, sodass sich zwischen diesen Schritten kein Auflöser einschalten kann. Jeder spätere Auflöser schreibt die Entscheidung über SQLite fest, bevor er diesen Wartenden abschließt.

## Persistenter Datensatz

Fügen Sie der gemeinsamen Statusdatenbank eine Tabelle `operator_approvals` hinzu.

| Spalte                                             | Zweck                                                                                                                                       |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `approval_id`                                      | Global eindeutige kanonische ID. Vorhandene Ausführungs-IDs und `plugin:`-IDs aus Gründen der Protokollkompatibilität beibehalten, aber die Art niemals aus dem Präfix ableiten.      |
| `resolution_ref`                                   | Eindeutiger vollständiger SHA-256-base64url-Locator für Transport-Callbacks, die die kanonische ID nicht übertragen können. Er ist weder eine Autorisierung noch eine öffentliche URL-ID. |
| `kind`                                             | Geschlossener `exec \| plugin`-Diskriminator.                                                                                                        |
| `status`                                           | Geschlossener `pending \| allowed \| denied \| expired \| cancelled`-Status.                                                                          |
| `presentation_json`                                | Validierte, nach Art gekennzeichnete Prüferprojektion. Unverarbeitete Laufzeitanfragen, Befehlsbindungen und Callback-Nutzlasten bleiben prozesslokal.               |
| `source_agent_id`, `source_session_key`            | Quellidentität und Anker der Sitzungsprojektion. Der Sitzungsschlüssel ist dauerhaft, die rotierende Sitzungs-UUID nicht.                                          |
| `audience_session_keys_json`                       | Geordnetes, dedupliziertes JSON-Array, das durch die begrenzte Breitensuche durch die Besitzstruktur erzeugt wird. Anforderungs- und Abschlussereignisse verwenden denselben Snapshot. |
| `requested_by_device_id`, `requested_by_client_id` | Dauerhafte Metadaten zu Anforderer und Audit. Die Verbindungs-ID verbleibt im Arbeitsspeicher und ist kein oberflächenübergreifender Principal.                                         |
| `reviewer_device_ids_json`                         | Optionale, explizit adressierte Prüfergeräte, die ausschließlich von der vertrauenswürdigen Genehmigungslaufzeit bereitgestellt werden.                                                  |
| `runtime_epoch`                                    | Prozessepoche, der die angehaltene Ausführung gehört; wird verwendet, um verwaiste Zeilen nach einem Neustart abzubrechen.                                                     |
| `created_at_ms`, `expires_at_ms`, `updated_at_ms`  | Maßgebliche Zeitangaben.                                                                                                                         |
| `decision`                                         | Explizite Benutzerentscheidung, sofern eine vorliegt.                                                                                                       |
| `terminal_reason`                                  | Geschlossener Grund wie `user`, `timeout`, `malformed-verdict`, `no-route`, `run-aborted` oder `gateway-restart`.                                |
| `resolved_at_ms`, `resolver_kind`, `resolver_id`   | Gewinner- und Auditidentität werden serverseitig aufbewahrt. Prüferprojektionen lassen unverarbeitete Resolver-Bezeichner aus.                                           |
| `consumed_at_ms`, `consumed_by`                    | Separate Wiederholungssperre für `allow-once`; die Verwendung darf die aufgezeichnete Entscheidung nicht löschen.                                                       |

Erforderliche Indizes:

| Index                                      | Zweck                                                                     |
| ------------------------------------------ | --------------------------------------------------------------------------- |
| eindeutiger `(resolution_ref)`                  | Spaltenübergreifende Mehrdeutigkeit von `approval_id`/`resolution_ref` beim Einfügen ablehnen. |
| `(status, expires_at_ms)`                  | Ausstehende Genehmigungen finden und maßgebliche Fristen abgleichen.               |
| `(source_session_key, created_at_ms DESC)` | Kürzlich erfolgte Genehmigungen für eine Quellsitzung wiedergeben.                             |
| `(resolved_at_ms)`                         | Aufbewahrte abgeschlossene Genehmigungen gemäß der festen Aufbewahrungsrichtlinie bereinigen.  |

Zielgruppen-Arrays sind klein und begrenzt. Die nach Sitzung gefilterte Wiedergabe wählt zunächst über Kysely sichtbare ausstehende Zeilen aus und dekodiert und filtert anschließend die begrenzten Zielgruppen-Arrays im Anwendungscode; sie verwendet weder Zeichenfolgenabgleich noch JSON-Abfragen mit unverarbeitetem SQL.

Abgeschlossene Zeilen werden 30 Tage lang aufbewahrt, entsprechend der Aufbewahrung von Audit-Metadaten in `src/audit/audit-event-store.ts`. Die Bereinigung ist eine feste Wartungsrichtlinie und keine neue Konfigurationsoberfläche. Die Datenbank enthält privaten lokalen Zustand der Steuerungsebene, aber Prüfer-APIs dürfen niemals die vollständige gespeicherte Anfrage oder Laufzeitbindung offenlegen.

## Zustandsautomat und Compare-and-Set

Nur diese Übergänge sind gültig:

- `pending -> allowed`: explizites `allow-once` oder `allow-always`.
- `pending -> denied`: explizite Ablehnung, vertrauenswürdiges fehlerhaftes Abschlussurteil oder kein Zustellweg.
- `pending -> expired`: maßgebliche Frist erreicht.
- `pending -> cancelled`: Abbruch der Ausführung, ordnungsgemäßes Herunterfahren oder Wiederherstellung verwaister Einträge nach einem Neustart.

Jeder nicht zulässige Abschlussstatus hat als wirksames Urteil eine Ablehnung.

Die Auflösung verwendet eine unmittelbare SQLite-Transaktion und eine bedingte Kysely-Aktualisierung entsprechend:

```sql
UPDATE operator_approvals
SET status = ?, decision = ?, terminal_reason = ?, resolved_at_ms = ?
WHERE approval_id = ?
  AND status = 'pending'
  AND expires_at_ms > ?;
```

Wenn die Aktualisierung keine Zeile betrifft, liest dieselbe Transaktion den Datensatz:

- Fehlend oder nicht autorisiert: Nicht gefunden zurückgeben; die Existenz nicht offenlegen.
- Noch ausstehend, aber Frist erreicht: per Compare-and-Set auf `expired` setzen und anschließend diese Abschlusszeile zurückgeben.
- Dieselbe aufgezeichnete Entscheidung: idempotenten Erfolg mit dem aufgezeichneten Gewinner zurückgeben.
- Andere Entscheidung: Die einheitliche API gibt `applied: false` mit dem aufgezeichneten Gewinner zurück; Legacy-Adapter behalten `APPROVAL_ALREADY_RESOLVED` bei, wenn ihr ausgelieferter Vertrag dies erfordert.
- Jeder Abschlussstatus: niemals ändern.

`now == expires_at_ms` ist abgelaufen. Die Gateway-Zeit ist maßgeblich.

Die Ausführung von `allow-once` verwendet ein zweites CAS über `consumed_at_ms IS NULL`, das an den bestehenden exakten Befehls-/Systemausführungskontext gebunden ist. Die Genehmigungszeile bleibt nach der Verwendung als Audit-Datensatz bestehen.

Fehlerhafte HTTP-/RPC-Eingaben, die nicht authentifiziert werden können oder keine Genehmigung identifizieren, werden ohne Änderung abgelehnt und können niemals genehmigen. Ein fehlerhaftes Abschlussurteil, das von einem vertrauenswürdigen Harness/Waiter für eine bekannte Genehmigung empfangen wird, führt zum Übergang zu `denied`.

## Gateway-API

Artunabhängige Prüfermethoden hinzufügen:

| Methode                                    | Vertrag                                                                                                                                                                                                            |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `approval.get { id }`                     | Gibt eine sichtbare ausstehende oder aufbewahrte abgeschlossene Projektion zurück.                                                                                                                                                          |
| `approval.resolve { id, kind, decision }` | Akzeptiert die kanonische ID oder die Transportreferenz fester Größe und führt anschließend Autorisierung, Validierung der Art und der zulässigen Entscheidung, Fristabgleich sowie abschließendes CAS durch. Die Antwort enthält immer die kanonische ID. |

Nach einem erfolgreichen CAS die festgeschriebene Projektion sofort zurückgeben. Legacy-Ereignisse, Channel-Weiterleitungen und Push-Abschlussverarbeitung sind Best-Effort-Nachfolgeaktionen; eine langsame oder fehlgeschlagene Oberfläche darf die erfolgreiche Antwort weder verzögern noch zurückrollen.

Die artspezifische Anfragevalidierung verbleibt in `exec.approval.request` und `plugin.approval.request`. Die bestehenden `exec.approval.get/list/waitDecision/resolve` und `plugin.approval.list/waitDecision/resolve` werden zu Protokollgrenzen-Adaptern für den kanonischen Dienst, da sie eine ausgelieferte Gateway-API sind. Interne Aufrufer werden in derselben Änderung auf den Dienst migriert.

Eine Prüferprojektion ist eine gekennzeichnete Union:

```ts
type OperatorApproval = {
  id: string;
  status: OperatorApprovalStatus;
  presentation:
    | { kind: "exec"; commandText: string /* sichere Ausführungsvorschau */ }
    | { kind: "plugin"; title: string; description: string /* sichere Plugin-Vorschau */ };
  // gemeinsame Lebenszyklusfelder
};
```

Der stabile Pfad wird abgeleitet und nicht persistiert. `approval.get` gibt `urlPath` zurück; Oberflächen, die einen genehmigten öffentlichen Ursprung kennen, können zusätzlich ein absolutes `url` erhalten. Prüfer-Snapshots lassen Quell- und Zielgruppen-Sitzungsschlüssel aus. Das Gateway verwahrt diese Routing-Schlüssel serverseitig für die separate `session.approval`-Projektion.

## Ereignisse und portable Aktionen

PR 1 behält die ausgelieferten Ereignisnamen, Nutzlasten und vorhandenen empfängerbezogenen Filter auf Datensatzebene bei:

- `exec.approval.requested`
- `exec.approval.resolved`
- `plugin.approval.requested`
- `plugin.approval.resolved`

Diese Legacy-Ereignisse können die vollständige Laufzeitanfrage enthalten und dürfen daher nicht an alle genehmigungsbezogenen Clients verteilt werden. PR 5 fügt gekennzeichnete Lebenszyklusfelder (`status`, `sourceSessionKey`, `urlPath`, Abschlussmetadaten und ein `kind` auf Präsentationsebene) über die bereinigte Lebenszyklusprojektion hinzu, anstatt die Zustellung von Legacy-Ereignissen auszuweiten.

Ein genehmigungsbezogenes `session.approval`-Projektionsereignis hinzufügen. Das kanonische Ereignis wird einmal mit den persistierten Zielgruppenschlüsseln veröffentlicht; Abonnenten der exakten Sitzung erhalten dasselbe Ereignis für jeden übereinstimmenden Schlüssel:

- `sessionKey`: Datenstrom, der die Projektion empfängt.
- `sourceSessionKey`: untergeordnetes Element/Quelle, das bzw. die die Sperre ausgelöst hat.
- `phase`: `pending \| terminal`, anhand des Genehmigungsstatus diskriminiert.
- eine sichere `OperatorApproval`-Projektion.

Clients melden sich mit `sessions.messages.subscribe { key, agentId?, includeApprovals: true }` an. Die erfolgreiche Antwort fügt ein `approvalReplay` hinzu, das bis zu 1.000 aktuell ausstehende Genehmigungen für genau diesen Datenstromschlüssel enthält, zu deren Prüfung der abonnierende Client auch auf Datensatzebene autorisiert ist. `truncated: false` macht die gefilterte Wiedergabe maßgeblich, und Clients, die erneut eine Verbindung herstellen, ersetzen ihren lokalen Satz ausstehender Einträge damit; `truncated: true` ist ein Überlastungssignal, und Clients müssen noch nicht gesehene lokale Einträge beibehalten, bis eine kanonische Abfrage oder spätere Lebenszyklusereignisse sie klären. Ein späterer dauerhafter Timeout, der während der Wiedergabe festgestellt wird, sendet Abschluss-Tombstones ausschließlich an abonnierte, auf Datensatzebene autorisierte Zielgruppen, bevor der neue Snapshot zurückgegeben wird. `operator.admin` kann sich direkt anmelden; stärker eingeschränkte Clients benötigen sowohl eine gekoppelte Geräteidentität als auch `operator.approvals`. Ein Sitzungsabonnement allein gewährt niemals Sichtbarkeit von Genehmigungen.

Das Ereignis unter `operator.approvals` in `src/gateway/server-broadcast.ts` registrieren. Die Projektion dient nur der Beobachtung: Sie hängt niemals Transkriptzeilen an, gibt kein `sessions.changed` aus und weckt keinen Agenten.

`MessagePresentationAction` in `src/interactive/payload.ts` erweitern:

```ts
type MessagePresentationAction =
  | { type: "command"; command: string }
  | { type: "callback"; value: string }
  | {
      type: "approval";
      approvalId: string;
      approvalKind: "exec" | "plugin";
      decision: ExecApprovalDecision;
    }
  | { type: "url"; url: string }
  | { type: "web-app"; url: string };
```

Der Kern erstellt typisierte Entscheidungsaktionen und einen separaten Review-Link, wenn ein genehmigter absoluter Control-UI-Ursprung verfügbar ist. Channels codieren eine Genehmigungsaktion in ihrem eigenen Callback-Format und senden die Auflösung an den kanonischen Dienst. Ein Callback verwendet die exakte kanonische ID, wenn sie passt; andernfalls verwendet er den eindeutigen vollständigen Digest `resolution_ref` der Zeile. Die Referenz ist nur ein kompakter Suchschlüssel: Die normale Gateway-Authentifizierung, die Autorisierung des Datensatzes, die explizite Art, die Validierung zulässiger Entscheidungen, der Abgleich der Frist und das CAS für die erste Antwort gelten weiterhin. Channels dürfen IDs nicht kürzen, Hash-Präfixe auflösen, `/approve`-Text parsen oder die Art aus einem ID-Präfix ableiten.

Behalten Sie `button.url`, `button.webApp` und befehlsbasierte Genehmigungssteuerelemente als veraltete Kompatibilitätseingaben für das Plugin SDK bei. Normalisieren Sie sie an der SDK-Grenze; migrieren Sie jeden gebündelten internen Aufrufer im selben PR. `/approve {id} {decision}` bleibt ein Text-Fallback und ein CLI-/Chatbefehl, nicht der semantische Vertrag für Schaltflächen.

## Control UI

Die Route lautet `${basePath}/approve/{approvalId}`. Die ID ist der einzige Pfadparameter; die Identität der Quellsitzung stammt aus dem Datensatz.

Da der aktuelle Router exakte statische Routen hat und unbekannte Pfade zu Chat umschreibt, muss dieser Deep Link in `ui/src/app/bootstrap.ts` vor der normalen Routennormalisierung erkannt werden. Verwenden Sie die normale Gateway-/Authentifizierungseinrichtung erneut, stellen Sie jedoch eine eigenständige Genehmigungsseite außerhalb der Seitenleisten-Shell und des globalen Modals dar.

Das Dokument gehört dem Gateway, das seine URL bereitgestellt hat. Seine anfängliche Verbindung ignoriert die persistierte Auswahl eines Remote-Gateways der vollständigen App, ohne die Einstellungen dieser Auswahl zu ändern oder zu kopieren; nur die Authentifizierung bleibt auf die Sitzung des bereitstellenden Gateways beschränkt. Vertrauenswürdige native Authentifizierung oder eine separat bestätigte `gatewayUrl`-Überschreibung kann das Ziel ändern. Der Kern reserviert den einsegmentigen Namespace `/approve` vor Plugin-HTTP-Routen und der Erkennung statischer Erweiterungen, einschließlich IDs, die auf `.json` oder `.js` enden; wenn die Bereitstellung der Control UI deaktiviert ist, schlägt die reservierte Route mit `404` sicher geschlossen fehl. Behalten Sie die Seite im Haupt-Bundle der Control UI, damit ein fehlgeschlagener Lazy Chunk eine Sicherheitsentscheidung nicht dauerhaft an einen Ladeindikator bindet.

Seitenzustände:

- wird geladen
- Authentifizierung erforderlich
- ausstehend
- wird aufgelöst
- hier genehmigt oder abgelehnt
- an anderer Stelle aufgelöst
- abgelaufen
- abgebrochen
- verboten/nicht gefunden
- Verbindungsfehler mit Wiederholungsoption

Die Seite ruft Gateway-RPC auf, keine zweite nicht authentifizierte REST-API. Beim Aktualisieren des Browsers wird der persistente Zustand erneut gelesen. Gateway-Anmeldedaten werden niemals in der URL, der Abfrage oder dem Fragment platziert.

## Autorisierung und Datenschutz

Die URL ist ein Locator, keine Autorität. Die Auflösung erfordert:

1. eine authentifizierte Gateway-Verbindung;
2. `operator.approvals` oder `operator.admin`;
3. Autorisierung des Prüfers auf Datensatzebene.

Regeln auf Datensatzebene:

- `operator.admin` darf prüfen.
- `reviewer_device_ids` ist maßgeblich, wenn vorhanden. Nur ein aufgeführtes gekoppeltes
  `operator.approvals`-Gerät darf prüfen; das anfordernde Gerät hat keinen impliziten
  Zugriff, sofern es nicht ebenfalls aufgeführt ist.
- Ohne eine explizite Prüferliste darf das anfordernde gekoppelte
  `operator.approvals`-Gerät seinen eigenen Datensatz prüfen.
- Tatsächlich veraltete Datensätze ohne Bindung an Anforderer oder Prüfer behalten eine breite
  Sichtbarkeit für gekoppelte Geräte, damit Upgrades bereits ausstehende Arbeit nicht unerreichbar machen.
- Gerätelose interne Laufzeiten dürfen über die bereichsbeschränkte
  Genehmigungslaufzeitverbindung auflösen, aber nicht lesen. Diese Autorität stammt ausschließlich aus dem
  serverseitig authentifizierten Laufzeittoken; öffentliche `approval.resolve`-Felder können
  es nicht erzeugen.
- Die Eigentümerschaft einer aktiven Anfordererverbindung bleibt für veraltete Adapter gültig; sie wird
  niemals aus einem übereinstimmenden Clientnamen abgeleitet.
- Die Zielgruppenzugehörigkeit ändert nur die Darstellung. Sie erweitert niemals die Autorisierung.

`approval.get` stellt nur die bereinigte Prüferprojektion bereit und lässt interne Routing-Schlüssel für Quelle und Zielgruppe aus. Das `session.approval`-Ereignis aus PR 5 enthält sein einzelnes Ziel `sessionKey` sowie `sourceSessionKey`, nachdem das Gateway den persistierten Zielgruppen-Snapshot serverseitig angewendet hat. Bestehende Exec-/Plugin-Ereignisse behalten ihre bisherigen Nutzdaten und eingeschränkten Empfänger, bis die Verbraucher migriert sind. Die ausführbare Anfrage, die Befehlsbindung und die Fortsetzung verbleiben ausschließlich im prozesslokalen Waiter. Die persistente Zeile enthält die sichere Darstellung sowie Lebenszyklus-, Routing- und Audit-Metadaten; sie speichert niemals rohe Umgebungswerte, Anmeldedaten, Authentifizierungsheader oder Channel-Callback-Daten.

## Zielgruppenprojektion

Berechnen Sie die Zielgruppe einmal vor dem Einfügen und persistieren Sie den geordneten Snapshot. Eigentümerschaft ist ein Graph und nicht immer eine einzelne übergeordnete Kette: Ein untergeordnetes Element kann sowohl einen aktuellen Controller als auch einen ursprünglichen Anforderer haben, und diese Eigentümer können zu verschiedenen Wurzeln führen.

Verwenden Sie eine deterministische Breitensuche:

1. Initialisieren Sie die Warteschlange mit dem Schlüssel der Quellsitzung.
2. Lesen Sie für jeden aus der Warteschlange entnommenen Schlüssel die neueste Zeile der Subagenten-Registry und fügen Sie beide unterschiedlichen Eigentümerschaftskanten in fester Reihenfolge zur Warteschlange hinzu: `controllerSessionKey`, dann `requesterSessionKey`.
3. Wenn eine nutzbare Registry-Zeile vorhanden ist, folgen Sie nicht zusätzlich der Abstammung des Sitzungseintrags, die nach einer Steuerungsänderung veraltet sein kann. Fügen Sie andernfalls die einzelne aktuelle Fallback-Kante `parentSessionKey ?? spawnedBy` zur Warteschlange hinzu.
4. Normalisieren und deduplizieren Sie beim Einreihen, sodass der erste, kürzeste Pfad gewinnt.
5. Beenden Sie bei 64 eindeutigen Schlüsseln; diese Begrenzung der Zielgruppengröße begrenzt auch die Traversierungstiefe.

Die Registry-Quelle ist `src/agents/subagent-registry-read.ts`; Eigentümerschaftsfelder sind in `src/agents/subagent-registry.types.ts` definiert. Sitzungs-Fallback-Felder sind in `src/config/sessions/types.ts` definiert.

Anfrage- und Abschlussprojektionen verwenden dieselbe persistierte Zielgruppe, selbst wenn sich der Fokus oder die Eigentümerschaft des Controllers ändert, während die Genehmigung aussteht. Dies gewährleistet die abschließende Bereinigung für jeden Sitzungsstream der Zielgruppe, der die Anfrageprojektion erhalten hat. Die Auflösung zielt immer auf die ID der Quellgenehmigung; Zielgruppensitzungen erhalten niemals einen geklonten Genehmigungszustand. Die Bereinigung weitergeleiteter Channel-Nachrichten bleibt die unten beschriebene separate Nacharbeit für Zustellungs-Locators.

Schreiben Sie nicht ausschließlich wegen einer Genehmigung Transkriptnachrichten, injizieren Sie keine System-Prompts, starten Sie keine Eigentümer-Turns und geben Sie kein `sessions.changed` aus.

## Konvergenz zugestellter Oberflächen

Native Genehmigungshandler bewahren ihre Einträge zugestellter Nachrichten bereits lange genug auf, um aktive Steuerelemente zu ersetzen oder außer Betrieb zu nehmen. Generische weitergeleitete Genehmigungsnachrichten verwerfen derzeit `MessageReceipt`, sodass eine Entscheidung auf einer anderen Oberfläche dazu führen kann, dass ihre alten Steuerelemente weiterhin als ausstehend erscheinen. Eine separate Nacharbeit schließt diese Lücke mit einer untergeordneten Tabelle `operator_approval_deliveries` in der gemeinsamen Zustandsdatenbank.

Jede Zeile speichert die Genehmigungs-ID, eine eindeutige Zustellungs-ID, Channel/Konto/exakte Route, einen größenbegrenzten JSON-validierten Channel-privaten Nachrichten-Locator, Zustellungszeitstempel und den Abschlusszustand. Sie speichert niemals Callback-Daten, Entscheidungstoken oder rohe Genehmigungsanfragen. Der Channel ist für die Locator-Codierung und Nachrichtenmutation zuständig; der Kern ist für kanonischen Status, Zielauswahl, Wiederholungsrichtlinie und abschließenden Fallback-Text zuständig.

Zustellungsregistrierung und abschließende Auflösung behandeln Race Conditions sicher:

1. Nachdem ein ausstehender Sendevorgang seinen Beleg zurückgegeben hat, fügen Sie den Zustellungs-Locator ein und lesen Sie den Status der übergeordneten Genehmigung in einer Transaktion.
2. Wenn das übergeordnete Element bereits abgeschlossen ist, planen Sie eine sofortige Finalisierung, statt die verspätete Zustellung ausstehend zu lassen.
3. Jeder festgeschriebene Abschlussübergang plant separat alle nicht finalisierten Zustellungszeilen; verwerfbare Broadcasts sind nicht der Auslöser.
4. Ein Channel-Terminalizer meldet `replaced`, `retired` oder `unsupported`. „Ersetzt“ unterdrückt eine doppelte Abschlussnachricht; „außer Betrieb genommen“ sendet die vorhandene abschließende Folgenachricht; „nicht unterstützt“ oder ein Fehler greift auf den Fallback zurück, ohne das Genehmigungs-CAS zurückzusetzen.
5. Beim Start werden abgeschlossene Genehmigungen mit nicht abgeschlossenen Zustellungen erneut versucht, wodurch die Bereinigung gegenüber einem Gateway-Neustart resilient wird.

Dieser Transportlebenszyklus ist ein optionaler Hook für Zustellungsadapter, kein Renderer und keine modellseitige Nachrichtenaktion. QQ-C2C-/Gruppennachrichten verfügen derzeit über keine API zum Bearbeiten, Löschen oder Entfernen der Tastatur; dieser Adapter bleibt nicht unterstützt und kann die kanonische Wahrheit erst nach einem späteren Klick anzeigen, bis der Transport eine Mutations-API erhält.

## Neustart-, Zeitüberschreitungs- und Routensemantik

SQLite-Persistenz impliziert keine Fortsetzung der Ausführung. Befehls-/Tool-Bindungen bleiben im Arbeitsspeicher, da sie sicherheitskritische Laufzeitfakten enthalten können und keinen Vertrag für einen fortsetzbaren Auftrag darstellen.

Beim Start des Gateways:

- eine neue Laufzeitepoche erzeugen;
- ausstehende Zeilen aus älteren Epochen atomar mit dem Grund `gateway-restart` in `cancelled` überführen;
- Zeilen beibehalten, damit ihre URLs erklären, was geschehen ist;
- eine spätere Genehmigung niemals gegen eine fehlende Laufzeitbindung ausführen.

Timer sind Optimierungen zum Aufwecken. Die maßgebliche Frist wird in `expires_at_ms` gespeichert; Lese-, Warte- und Auflösungsvorgänge führen alle einen Ablaufabgleich aus.

Endgültiges striktes Verhalten:

- Zeitüberschreitung -> `expired`, ablehnen;
- keine Route -> `denied`, ablehnen;
- Abbruch des Laufs -> `cancelled`, ablehnen;
- fehlerhaftes vertrauenswürdiges Urteil -> `denied`, ablehnen;
- nur eine zulässige ausdrückliche Erlaubnisentscheidung -> `allowed`.

Das derzeit ausgelieferte Exec-Verhalten steht noch im Konflikt mit diesem Vertrag:

- `src/agents/bash-tools.exec-host-shared.ts` kann `askFallback` anwenden.
- `docs/tools/exec-approvals.md` und `docs/cli/approvals.md` dokumentieren diese Oberfläche.

Plugin-Genehmigungen schlagen nun bei Zeitüberschreitungen und fehlerhaften Urteilen sicher geschlossen fehl; das veraltete
Feld `timeoutBehavior` wird weiterhin akzeptiert, aber ignoriert. Die Nacharbeit
für die strikte Exec-Semantik muss Code, Typen, Dokumentation, Tests und Changelog gemeinsam aktualisieren und
ausdrücklich von Eigentümern und Sicherheitsexperten geprüft werden. `askFallback` darf während
der Migration weiterhin die Richtlinienauswahl vor dem Gate beschreiben, darf jedoch die Zeitüberschreitung
eines erstellten ausstehenden Datensatzes nicht in eine Genehmigung umwandeln.

## Kompatibilitätsplan

- Additives Gateway-Protokoll; keine Erhöhung der Protokollversion.
- Bestehende Exec-/Plugin-Methoden und -Ereignisse an der externen Grenze beibehalten.
- Bestehende IDs einschließlich der Präfixe `plugin:` beibehalten, Präfixe jedoch nicht mehr als Typinformationen verwenden.
- Das Verhalten des Textbefehls `/approve` beibehalten.
- Veraltete URL-/Web-App-Felder für Schaltflächen und Befehlsaktionen als Kompatibilitätseingaben für das Plugin SDK beibehalten; neue Kernausgaben sind typisiert.
- Alle gebündelten Channels und internen Aufrufer in derselben Änderung für typisierte Aktionen migrieren.
- Einen Changelog-Eintrag für die neue URL/Seite und für die spätere Änderung des Zeitüberschreitungsverhaltens hinzufügen.
- Keine Einstellung für den Elicitation-Modus hinzufügen.

## Einführung

### PR 1: Persistenter Lebenszyklus

- Dieser Designhinweis.
- Gemeinsames SQLite-Schema, Kysely-Generierung, Speicher und Bereinigung nach 30 Tagen.
- Gateway-Genehmigungsdienst, Laufzeit-Waiter-Brücke und Behandlung verwaister Datensätze nach einem Neustart.
- Vereinheitlichtes `approval.get/resolve`.
- Adapter für Exec-/Plugin-Methoden.
- Tests für „erste Antwort gewinnt“, Idempotenz, Ablauf, Autorisierung und Verbrauch.
- Noch keine Änderung des UI- oder Channel-Verhaltens.

### PR 2: Typisierte Aktionen und Channel-Callbacks

- Typisierte Genehmigungs-, URL- und Web-App-Aktionen.
- Zentrale Darstellungs-Builder und Exporte des Plugin-SDK.
- Transportprivate Callback-Codierung mit expliziter Eigentümerart.
- Dauerhafte Callback-Referenzen fester Größe für kanonische IDs, die Transportlimits überschreiten.
- Migration gebündelter Kanäle weg von der Ableitung aus Befehlstext und Genehmigungs-ID.
- Kanonische Wahrheit der ersten Antwort auf der angeklickten Oberfläche und bestmögliche Aktualisierungen des aktiven nativen Endzustands; die dauerhafte Überführung von Kanalnachrichten in den Endzustand bleibt eine Folgeaufgabe.
- Tests für das SDK und gebündelte Kanäle.

### PR 3: Deep Link für die Control UI

- Eigenständige authentifizierte Genehmigungsseite und Basispfad berücksichtigendes Start-Routing.
- Bindung an den bereitstellenden Gateway, ohne die gespeicherte Remote-Auswahl des Operators zu verändern.
- Vom Kern verwalteter HTTP-Namensraum für Genehmigungen, einschließlich assetähnlicher IDs.
- Vom Gateway erstellte URL-Nutzlast und Abfrage des ausstehenden Zustands, bis Lebenszyklusereignisse bereitgestellt werden.
- Nachweise für mobile Breite, Wiederverbindung, konkurrierende Antworten, Neuladen und eingebundenen Pfad.

### PR 4: native Clients

- iOS- und Android-Prüfoberflächen verwenden artbewusst `approval.get/resolve`; watchOS leitet prüfersichere Aufforderungen und Entscheidungen über das gekoppelte iPhone weiter.
- Die Watch bietet die von ihrem kompakten Weiterleitungsvertrag unterstützten Ausführungsentscheidungen: einmalig zulassen und ablehnen.
- Die kanonische Endzustandswahrheit der ersten Antwort ersetzt den lokalen Zustand des versuchten Entscheids.
- Verlorene oder mehrdeutige Bestätigungen der Auflösung sperren die Steuerelemente bis zum kanonischen Rücklesen.
- Bisherige ausgelieferte Gateway-v4-Instanzen behalten die Ausführungsprüfung durch einen eng begrenzten Fallback auf die Legacy-Methode bei; ein oberflächenübergreifend beibehaltener Endzustand erfordert die vereinheitlichten Methoden.
- Warnungen für Prüfer und Eigentümerkontext bleiben auf iPhone, Watch und Android sichtbar.
- Nachweise durch native Unit-Tests, Builds und Plattformtests.

### PR 5: Weitergabe des Lebenszyklus an übergeordnete Elemente

- `session.approval`-Übermittlung ausstehender und endgültiger Zustände aus dem in PR 1 persistierten Zielgruppen-Snapshot.
- Exaktes Sitzungsabonnement, Wiedergabe nach Wiederverbindung und Endzustands-Tombstones ohne Transkriptänderung oder Aktivierung des Agenten.
- Lebenszyklus-Callbacks werden nach dem dauerhaften Einfügen/CAS ausgeführt und erlangen niemals Genehmigungsautorität.
- Nachweise für verschachtelte Subagenten und Wiederverbindungen.

### PR 6: Fail-Closed-Verhalten

- `node-invoke-plugin-policy.ts` und den eingebetteten Plugin-Broker von doppelter Autorität weg migrieren.
- Strikte Semantik für Zeitüberschreitungen, fehlerhafte Daten, fehlende Routen, Bindungen und den einmaligen Verbrauch von Genehmigungen.
- Ausgelieferte permissive Zeitüberschreitungseinstellungen als veraltet markieren, ohne sie nach dem Ausstehen einer Anfrage zu berücksichtigen.
- Nachweise für Konkurrenz zwischen mehreren Oberflächen und Fehlerinjektion.

### Folgeaufgabe: dauerhafte Bereinigung von Remote-Nachrichten

- Weitergeleitete Zustellungs-Locators persistieren und jede zugestellte Kanalnachricht nach einem Neustart in den Endzustand überführen.
- Diesen Transportlebenszyklus von der kanonischen Genehmigungsautorität und typisierten Darstellungsaktionen getrennt halten.

## Tests

Erforderliche fokussierte Abdeckung:

- Das erneute Öffnen von SQLite erhält ausstehende und endgültige Projektionen.
- Zwei gleichzeitige Auflöser erzeugen genau einen CAS-Gewinner.
- Eine Wiederholung derselben Entscheidung ist idempotent erfolgreich; eine widersprüchliche Wiederholung gibt den aufgezeichneten Gewinner zurück.
- Eine Auflösung zum oder nach dem Stichtag kann nicht genehmigen.
- `allow-once` kann genau einmal verbraucht werden, ohne den endgültigen Audit-Zustand zu löschen.
- Beim Start werden ältere Laufzeitepochen abgebrochen.
- Nicht autorisierte Abfragen und Auflösungen legen die Existenz des Datensatzes nicht offen.
- Explizite Prüfer-Zulassungsliste und allgemeines Verhalten gekoppelter `operator.approvals`.
- Legacy-Methoden für Ausführung und Plugins verwenden denselben Speicher.
- Gateway-Schemas für Anfrage, Auflistung, Abruf und Auflösung sowie additive Ereignisnutzlasten.
- Normalisierung typisierter Aktionen, Fallback-Darstellung, SDK-Exporte und Umstellungen gebündelter Kanäle.
- Die Telegram-Callback-Codierung enthält transportprivate Daten und keine Ableitung aus Befehlszeichenfolgen.
- Direktes untergeordnetes Element, verzweigte Controller-/Anforderer-Eigentümer, verschachtelte Eigentümer, Neuzuweisung, Fallback für Sitzungsfelder, Zyklus und Obergrenze der Zielgruppengröße.
- Die Zielgruppen-Arrays für Anforderung und Endzustand sind identisch.
- Eigentümerprojektionen verursachen weder Transkriptänderungen noch eine Aktivierung des Agenten.
- Die Control-UI-Route funktioniert unter `/` und einem konfigurierten Basispfad; nach dem Aktualisieren wird die ausstehende oder endgültige Wahrheit angezeigt.
- Gleichzeitige Antworten in Control UI und Telegram zeigen einen Gewinner und beim Verlierer „anderweitig aufgelöst“.
- Native Genehmigungskennungen und Gateway-Eigentümerkennungen bewahren beim Routing und Abgleich die exakten UTF-8-Bytes.
- Die Aushandlung der nativen RPC-Familie bindet jede zugelassene Gateway-Route an genau eine kanonische oder Legacy-Familie und führt nach der Verwendung niemals stillschweigend ein Downgrade durch.
- Verlorene native Bestätigungen der Auflösung sperren Aktionen bis zum kanonischen Rücklesen; ein fehlgeschlagenes Rücklesen kann weder einen Gewinner vortäuschen noch eine Watch-Aktualisierung bestätigen.
- Die Korrelation von Watch-Snapshot-Anfragen wird nur für den exakt gekoppelten Gateway-Eigentümer und ein abgeschlossenes kanonisches Rücklesen auf dem iPhone akzeptiert.
- Nachweis des Benutzerpfads durch Testbox/Crabbox, einschließlich einer Genehmigungsseite für mobile Breite, der Bereinigung von Telegram-Aktionen und eines vollständigen Ablaufs mit ausstehender Anfrage, Auflösung und verspätetem Verlierer über Android, iPhone und Watch hinweg.

## Beobachtbarkeit

Strukturierte, inhaltsfreie Übergangsprotokolle mit Genehmigungs-ID, Art, Quellsitzungsschlüssel, Status, Grund und Latenz ausgeben. Niemals die Vorschau oder die rohe Bindung protokollieren.

Erfassen:

- Anzahl der Anforderungen nach Art;
- Anzahl der Endzustände nach Art/Status/Grund;
- Messwert für ausstehende Vorgänge;
- Latenz von der Anforderung bis zum Endzustand;
- Ergebnisse von Auflösungsrennen: Gewinner, idempotente Wiederholung, Konflikt, abgelaufen;
- Anzahl der Zustellrouten und Ablehnungen wegen fehlender Route;
- Abbrüche verwaister Vorgänge beim Start;
- Zielgruppengröße.

Ein bestätigter Übergang gilt als Erfolg, auch wenn die spätere Ereigniszustellung fehlschlägt. Lebenszyklusabonnenten stellen den Zustand durch die Wiedergabe aus PR 5 und kanonische Abfragen wieder her. Die dauerhafte Überführung von Kanalnachrichten in den Endzustand bleibt die oben genannte separate Folgeaufgabe.

## Offene Entscheidungen

1. **Extern erreichbarer Ursprung der Control UI.** Jeder Snapshot enthält den stabilen relativen `urlPath`. Eine absolute URL darf erst aus einer zwischengespeicherten Tailscale-Serve-/Funnel-Adresse bekannt gegeben werden, nachdem die Gateway-Freigabe erfolgreich war; `allowedOrigins`, Host-Header von Anfragen, `gateway.remote.url` sowie ausschließlich zur Anzeige bestimmte Loopback-/LAN-Kandidaten sind keine kanonischen Ursprünge. Telegram kann seinen authentifizierten Mini-App-Wrapper verwenden, um den Genehmigungspfad beim Bootstrap beizubehalten. Beliebige Reverse-Proxys bleiben ausschließlich relativ, bis ein separat geprüfter expliziter Vertrag für öffentliche URLs existiert. Ein Kanal darf den Ursprung niemals erraten.
2. **Kompatibilitätsumstellung für strikte Zeitüberschreitungen bei Ausführungen.** Zeitüberschreitungen bei Plugin-Genehmigungen schlagen jetzt geschlossen fehl und `timeoutBehavior` ist veraltet. Der verbleibende ausgelieferte `askFallback`-Vertrag erfordert eine explizite Prüfung durch Eigentümer und Sicherheitsverantwortliche, einen Changelog-Eintrag, Dokumentation sowie eine Migrations-/Veraltungsentscheidung, bevor er nach einer Zeitüberschreitung einer ausstehenden Anfrage keine Ausführung mehr autorisiert.
3. **Eingebetteter Modus ohne Gateway.** Empfehlung: zunächst ausschließlich lokal halten und ihn anschließend zu einem Client des kanonischen Dienstes machen, wenn ein Gateway vorhanden ist. Keinen Deep Link bekannt geben, den kein Server auflösen kann.
