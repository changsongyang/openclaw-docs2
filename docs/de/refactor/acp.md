---
read_when:
    - Refactoring des ACP-Sitzungslebenszyklus oder der ACPX-Prozessbereinigung
    - Fehlerbehebung bei verwaisten ACPX-Prozessen, PID-Wiederverwendung oder sicherer Bereinigung mehrerer Gateways
    - Ändern der Sichtbarkeit von sessions_list für erzeugte ACP- oder Subagent-Sitzungen
    - Entwurf von Eigentümermetadaten für Hintergrundaufgaben, ACP-Sitzungen oder Prozess-Leases
sidebarTitle: ACP lifecycle refactor
summary: Migrationsplan zur expliziten Festlegung der Eigentümerschaft von ACP-Sitzungen und ACPX-Prozessen
title: Refaktorierung des ACP-Lebenszyklus
x-i18n:
    generated_at: "2026-07-26T19:13:14Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bda66f0acc93216c3d9386ca3ebf7f544efd306cd7f53386391f0c48e5dc8f06
    source_path: refactor/acp.md
    workflow: 16
---

Der ACP-Lebenszyklus funktioniert derzeit, aber zu viele Aspekte davon werden erst im Nachhinein abgeleitet.
Die Prozessbereinigung rekonstruiert die Eigentümerschaft anhand von PIDs, Befehlszeichenfolgen, Wrapper-
Pfaden und der aktuellen Prozesstabelle. Die Sitzungssichtbarkeit rekonstruiert die Eigentümerschaft
anhand von Sitzungsschlüssel-Zeichenfolgen sowie sekundärer `sessions.list({ spawnedBy })`-Abfragen.
Das ermöglicht gezielte Korrekturen, führt aber auch dazu, dass Grenzfälle leicht übersehen werden:
PID-Wiederverwendung, Befehle mit Anführungszeichen, untergeordnete Adapterprozesse, Zustandswurzeln mehrerer Gateways,
`cancel` im Vergleich zu `close` sowie die Sichtbarkeit von `tree` im Vergleich zu `all` werden zu separaten
Stellen, an denen dieselben Eigentümerschaftsregeln erneut ermittelt werden.

Dieses Refactoring macht die Eigentümerschaft zu einem grundlegenden Konzept. Ziel ist keine neue ACP-Produktoberfläche,
sondern ein sichererer interner Vertrag für das bestehende Verhalten von ACP und ACPX.

## Ziele

- Die Bereinigung sendet niemals ein Signal an einen Prozess, sofern aktuelle Live-Nachweise nicht mit einem
  OpenClaw-eigenen Lease übereinstimmen.
- `cancel`, `close` und die Bereinigung beim Start haben unterschiedliche Lebenszyklusabsichten.
- `sessions_list`, `sessions_history`, `sessions_send` und Statusprüfungen verwenden
  dasselbe Modell anfragereigener Sitzungen.
- Installationen mit mehreren Gateways können die ACPX-Wrapper der jeweils anderen nicht bereinigen.
- Alte ACPX-Sitzungsdatensätze funktionieren während der Migration weiterhin.
- Die Laufzeit bleibt Eigentum des Plugins; der Kern erhält keine Kenntnis von ACPX-Paketdetails.

## Nichtziele

- ACPX zu ersetzen oder die öffentliche Befehlsoberfläche `/acp` zu ändern.
- Anbieterspezifisches Verhalten von ACP-Adaptern in den Kern zu verschieben.
- Von Benutzern zu verlangen, den Zustand vor einem Upgrade manuell zu bereinigen.
- Durch `cancel` wiederverwendbare ACP-Sitzungen schließen zu lassen.

## Zielmodell

### Identität der Gateway-Instanz

Jeder Gateway-Prozess sollte eine stabile Laufzeitinstanz-ID besitzen:

```ts
type GatewayInstanceId = string;
```

Sie kann beim Start des Gateways erzeugt und für die Lebensdauer
dieser Installation im Zustand gespeichert werden. Sie ist kein Sicherheitsgeheimnis, sondern ein Unterscheidungsmerkmal der Eigentümerschaft,
das verhindert, dass die ACP-Prozesse eines Gateways mit den Prozessen eines anderen Gateways verwechselt werden.

### Eigentümerschaft von ACP-Sitzungen

Jede gestartete ACP-Sitzung sollte normalisierte Eigentümerschaftsmetadaten besitzen:

```ts
type AcpSessionOwner = {
  sessionKey: string;
  spawnedBy?: string;
  parentSessionKey?: string;
  ownerSessionKey: string;
  agentId: string;
  backend: "acpx";
  gatewayInstanceId: GatewayInstanceId;
  createdAt: number;
};
```

Das Gateway sollte diese Felder in Sitzungszeilen zurückgeben, sofern sie bekannt sind.
Die Sichtbarkeitsfilterung sollte eine reine Prüfung der Zeilenmetadaten sein:

```ts
canSeeSessionRow({
  row,
  requesterSessionKey,
  visibility,
  a2aPolicy,
});
```

Dadurch entfallen verborgene sekundäre `sessions.list({ spawnedBy })`-Aufrufe aus
Sichtbarkeitsprüfungen. Ein gestartetes agentenübergreifendes ACP-Kind gehört dem Anfragenden, weil
dies in der Zeile angegeben ist, und nicht, weil eine zweite Abfrage es zufällig findet.

### ACPX-Prozess-Leases

Jeder Start eines generierten Wrappers sollte einen Lease-Datensatz erstellen:

```ts
type AcpxProcessLease = {
  leaseId: string;
  gatewayInstanceId: GatewayInstanceId;
  sessionKey: string;
  wrapperRoot: string;
  wrapperPath: string;
  rootPid: number;
  processGroupId?: number;
  commandHash: string;
  startedAt: number;
  state: "open" | "closing" | "closed" | "lost";
};
```

Der Wrapper-Prozess erhält die Lease-ID und die Gateway-Instanz-ID als portable
Argumente:

```sh
--openclaw-acpx-lease-id ... --openclaw-gateway-instance-id ...
```

Wenn die Plattform dies zulässt, sollte die Verifizierung Live-Prozessmetadaten bevorzugen,
die nicht durch unterschiedliche Befehlsquotierung verwechselt werden können:

- Die Root-PID ist weiterhin vorhanden
- Der Live-Wrapper-Pfad befindet sich unter `wrapperRoot`
- Die Prozessgruppe stimmt, sofern verfügbar, mit dem Lease überein
- Die Argumente enthalten die erwartete Lease-ID
- Der Befehlshash oder der Pfad der ausführbaren Datei stimmt mit dem Lease überein

Wenn der Live-Prozess nicht verifiziert werden kann, schlägt die Bereinigung nach dem Fail-Closed-Prinzip fehl.

## Lebenszyklus-Controller

Es wird ein einzelner ACPX-Lebenszyklus-Controller eingeführt, der Prozess-Leases und die Bereinigungsrichtlinie
verwaltet:

```ts
interface AcpxLifecycleController {
  ensureSession(input: AcpRuntimeEnsureInput): Promise<AcpRuntimeHandle>;
  cancelTurn(handle: AcpRuntimeHandle): Promise<void>;
  closeSession(input: {
    handle: AcpRuntimeHandle;
    discardPersistentState?: boolean;
    reason?: string;
  }): Promise<void>;
  reapStartupOrphans(): Promise<void>;
  verifyOwnedTree(lease: AcpxProcessLease): Promise<OwnedProcessTree | null>;
}
```

`cancelTurn` fordert ausschließlich den Abbruch des aktuellen Durchlaufs an. Es darf keine wiederverwendbaren Wrapper-
oder Adapterprozesse bereinigen.

`closeSession` darf eine Bereinigung durchführen, jedoch erst nach dem Laden des Sitzungsdatensatzes,
dem Laden des Lease und der Verifizierung, dass der aktuelle Prozessbaum weiterhin zu diesem
Lease gehört.

`reapStartupOrphans` beginnt mit offenen Leases im Zustand. Es darf die Prozesstabelle
verwenden, um Nachfahren zu finden, sollte jedoch nicht zuerst beliebige ACP-ähnliche
Befehle durchsuchen und anschließend entscheiden, dass sie wahrscheinlich zu uns gehören.

## Wrapper-Vertrag

Generierte Wrapper sollten klein bleiben. Sie sollten:

- den Adapter, sofern unterstützt, in einer Prozessgruppe starten
- normale Beendigungssignale an die Prozessgruppe weiterleiten
- den Tod des übergeordneten Prozesses erkennen
- beim Tod des übergeordneten Prozesses SIGTERM senden und anschließend den Wrapper aktiv halten, bis die SIGKILL-
  Rückfallmaßnahme ausgeführt wird
- die Root-PID und die Prozessgruppen-ID an den Lebenszyklus-Controller zurückmelden, sofern
  dies verfügbar ist

Wrapper sollten keine Sitzungsrichtlinien festlegen. Sie erzwingen lediglich die lokale Bereinigung des Prozessbaums
ihrer eigenen Adaptergruppe.

## Vertrag für die Sitzungssichtbarkeit

Die Sichtbarkeit sollte die normalisierte Zeileneigentümerschaft verwenden:

```ts
type SessionVisibilityInput = {
  requesterSessionKey: string;
  row: {
    key: string;
    agentId: string;
    ownerSessionKey?: string;
    spawnedBy?: string;
    parentSessionKey?: string;
  };
  visibility: "self" | "tree" | "agent" | "all";
  a2aPolicy: AgentToAgentPolicy;
};
```

Regeln:

- `self`: nur die Sitzung des Anfragenden.
- `tree`: die Sitzung des Anfragenden sowie Zeilen, die dem Anfragenden gehören oder von ihm gestartet wurden.
- `all`: alle Zeilen desselben Agenten, durch a2a erlaubte agentenübergreifende Zeilen und vom Anfragenden gestartete
  agentenübergreifende Zeilen, selbst wenn allgemeines a2a deaktiviert ist.
- `agent`: nur derselbe Agent, sofern keine explizite Eigentümerbeziehung angibt, dass die Zeile
  dem Anfragenden gehört.

Dadurch werden `tree` und `all` monoton: `all` darf kein eigenes Kind ausblenden, das
`tree` anzeigen würde.

## Migrationsplan

### Phase 1: Identität und Leases hinzufügen

- `gatewayInstanceId` zum Gateway-Zustand hinzufügen.
- Einen ACPX-Lease-Speicher unter dem ACPX-Zustandsverzeichnis hinzufügen.
- Vor dem Start eines generierten Wrappers einen Lease schreiben.
- `leaseId` in neuen ACPX-Sitzungsdatensätzen speichern.
- Bestehende PID- und Befehlsfelder für alte Datensätze beibehalten.

### Phase 2: Lease-zuerst-Bereinigung

- Die Bereinigung beim Schließen so ändern, dass zuerst `leaseId` geladen wird.
- Vor dem Senden von Signalen die Eigentümerschaft des Live-Prozesses anhand des Lease verifizieren.
- Den aktuellen Rückfallmechanismus über Root-PID und Wrapper-Wurzel nur für Altdatensätze beibehalten.
- Leases nach verifizierter Bereinigung als `closed` markieren.
- Leases als `lost` markieren, wenn der Prozess bereits vor der Bereinigung beendet ist.

### Phase 3: Lease-zuerst-Bereinigung beim Start

- Die Bereinigung beim Start durchsucht offene Leases.
- Für jeden Lease den Root-Prozess verifizieren und Nachfahren erfassen.
- Verifizierte Bäume von den Kindern zur Wurzel bereinigen.
- Alte Leases vom Typ `closed` und `lost` innerhalb eines begrenzten Aufbewahrungszeitraums verfallen lassen.
- Das Scannen nach Befehlsmarkierungen nur als vorübergehenden Rückfallmechanismus für Altdatensätze beibehalten, soweit möglich abgesichert durch
  Wrapper-Wurzel und Gateway-Instanz.

### Phase 4: Zeilen für Sitzungseigentümerschaft

- Eigentümerschaftsmetadaten zu Gateway-Sitzungszeilen hinzufügen.
- ACPX-, Subagent-, Hintergrundaufgaben- und Sitzungsspeicher-Writer so anpassen, dass sie
  `ownerSessionKey` oder `spawnedBy` eintragen.
- Sichtbarkeitsprüfungen für Sitzungen auf die Verwendung von Zeilenmetadaten umstellen.
- Sekundäre `sessions.list({ spawnedBy })`-Abfragen während der Sichtbarkeitsprüfung entfernen.

### Phase 5: Veraltete Heuristiken entfernen

Nach einem Release-Zeitraum:

- bei der Bereinigung nicht veralteter ACPX-Datensätze nicht mehr auf gespeicherte Root-Befehlszeichenfolgen zurückgreifen
- Befehlsmarkierungs-Scans beim Start entfernen
- Rückfallabfragen von Listen für die Sichtbarkeit entfernen
- defensives Fail-Closed-Verhalten für fehlende oder nicht verifizierbare Leases beibehalten

## Tests

Zwei tabellengesteuerte Testsuiten hinzufügen.

Simulator für den Prozesslebenszyklus:

- PID wird von einem nicht zugehörigen Prozess wiederverwendet
- PID wird von der Wrapper-Wurzel eines anderen Gateways wiederverwendet
- der gespeicherte Wrapper-Befehl ist Shell-quotiert, der Live-Befehl `ps` dagegen nicht
- der untergeordnete Adapterprozess wird beendet, ein weiterer Nachfahre verbleibt in der Prozessgruppe
- die SIGTERM-Rückfallmaßnahme beim Tod des übergeordneten Prozesses erreicht SIGKILL
- die Prozessauflistung ist nicht verfügbar
- veralteter Lease mit fehlendem Prozess
- verwaister Prozess beim Start mit Wrapper, untergeordnetem Adapterprozess und weiterem Nachfahren

Matrix für die Sitzungssichtbarkeit:

- `self`, `tree`, `agent`, `all`
- a2a aktiviert und deaktiviert
- Zeile desselben Agenten
- agentenübergreifende Zeile
- vom Anfragenden gestartete agentenübergreifende ACP-Zeile
- auf `tree` begrenzter Anfragender in einer Sandbox
- Aktionen zum Auflisten, Anzeigen des Verlaufs, Senden und Prüfen des Status

Die wichtige Invariante: Ein vom Anfragenden gestartetes Kind ist überall dort sichtbar,
wo die konfigurierte Sichtbarkeit den Sitzungsbaum des Anfragenden einschließt, und `all` ist nicht
weniger leistungsfähig als `tree`.

## Kompatibilitätshinweise

Alte Sitzungsdatensätze enthalten möglicherweise kein `leaseId`. Sie sollten den veralteten
Fail-Closed-Bereinigungspfad verwenden:

- einen aktiven Root-Prozess voraussetzen
- die Eigentümerschaft der Wrapper-Wurzel voraussetzen, wenn ein generierter Wrapper erwartet wird
- bei Wurzeln ohne Wrapper eine Übereinstimmung der Befehle voraussetzen
- niemals ausschließlich anhand veralteter gespeicherter PID-Metadaten ein Signal senden

Wenn ein Altdatensatz nicht verifiziert werden kann, bleibt er unverändert. Die Lease-Bereinigung beim Start und
der nächste Release-Zeitraum sollten den Rückfallmechanismus letztlich außer Betrieb nehmen.

## Erfolgskriterien

- Das Schließen einer alten oder veralteten ACPX-Sitzung kann keinen Prozess eines anderen Gateways beenden.
- Der Tod des übergeordneten Prozesses hinterlässt keine hartnäckigen untergeordneten Adapterprozesse.
- `cancel` bricht den aktiven Durchlauf ab, ohne wiederverwendbare Sitzungen zu schließen.
- `sessions_list` kann vom Anfragenden gestartete agentenübergreifende ACP-Kinder sowohl unter
  `tree` als auch unter `all` anzeigen.
- Die Bereinigung beim Start wird durch Leases gesteuert, nicht durch breit angelegte Scans von Befehlszeichenfolgen.
- Die gezielten Tests für die Prozess- und Sichtbarkeitsmatrix decken jeden Grenzfall ab, der
  zuvor einmalige Korrekturen bei Reviews erforderte.
