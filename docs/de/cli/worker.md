---
read_when:
    - Betrieb oder Fehlerbehebung von durch das Gateway gestarteten Cloud-Workern
    - Überprüfung der Worker-Zulassung, Sitzungszuweisung oder lokalen Tool-Isolierung
summary: Interne Betreiberreferenz für die eingeschränkte Cloud-Worker-Laufzeit
title: Worker
x-i18n:
    generated_at: "2026-07-24T07:17:59Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 0c4749e2abaf4fca00d903114b0661454d67207547fe17711dc5315656e0cd14
    source_path: cli/worker.md
    workflow: 16
---

# `openclaw worker`

`openclaw worker` ist der eingeschränkte Laufzeit-Einstiegspunkt, den ein Cloud-Worker-
Orchestrator innerhalb einer vorbereiteten Worker-Umgebung startet. Er ist kein
allgemeiner Befehl für die manuelle Worker-Registrierung.

Das Gateway installiert das passende OpenClaw-Bundle und öffnet den an den Hostschlüssel gebundenen
Reverse-SSH-Tunnel. Der Worker-Launcher startet diesen Befehl mit einer vorbereiteten
Zuweisung. Der Befehl stellt über den durch den Tunnel weitergeleiteten lokalen Socket eine Verbindung her und
wird mit der dedizierten Rolle `worker` zugelassen.

## Startvertrag

Der Befehl liest genau einen begrenzten JSON-Start-Umschlag von der Standardeingabe.
Der Umschlag enthält den Speicherort des lokalen Sockets, die ausgestellte Worker-Anmeldeinformation, die Bundle-
und Protokollidentität, die Owner-Epoche, die einzelne zugewiesene Sitzung und Interaktion sowie die
exakten Namen der lokalen Worker-Tools, die für diese Interaktion autorisiert sind. Das Gateway löst diesen
endgültigen Toolsatz vor der Übergabe anhand der aktuellen Richtlinie auf; die Rohkonfiguration und die Identität
des geplanten Owners gelangen niemals in den Worker-Umschlag.
Die Anmeldeinformation wird niemals über Befehlszeilenargumente akzeptiert, und diese Seite
enthält bewusst kein Beispiel für Anmeldeinformationen oder einen manuell erstellten Umschlag.

Die Zulassung schlägt nach dem Fail-Closed-Prinzip fehl, wenn der Umschlag ungültig ist, die Anmeldeinformation abgelehnt wird,
die Bundle- oder Protokollfunktionen nicht übereinstimmen oder die Sitzung und Owner-Epoche nicht
mehr aktuell sind. Fehlende, doppelte oder unbekannte Toolnamen machen den
Umschlag ebenfalls ungültig. Operatoren sollten Worker über den Cloud-Worker-
Orchestrator starten, anstatt diesen Einstiegspunkt direkt aufzurufen.

## Laufzeitgrenze

Der Prozess führt die normale eingebettete Agentenschleife mit einem eingeschränkten Backend aus:

- Die Coding-Tools `read`, `write`, `edit`, `apply_patch`, `exec` und `process`
  werden lokal im Arbeitsbereich des Workers ausgeführt, sofern sie in der vom Gateway für den Turn
  erteilten Berechtigung enthalten sind. Bei einer leeren Berechtigung wird das Modell ohne Tools ausgeführt.
- Modellaufrufe verwenden den Inferenz-Proxy des Gateways. Es wird kein lokales
  Modellauthentifizierungsprofil geladen.
- Transkriptschreibvorgänge verwenden den Transcript-Commit-RPC des Gateways.
- Streaming- und Tool-Lebenszyklusaktualisierungen verwenden den Live-Event-RPC des Gateways.
- Nur die zugewiesene Sitzung und der zugewiesene Turn werden akzeptiert.

Der Worker-Modus startet keine Kanäle, Gateway-HTTP-Oberflächen oder den automatischen Start von Plugins
über den zugewiesenen Sitzungstoolsatz hinaus. Er verwendet ein temporäres Zustandsverzeichnis und verfügt
über keine dauerhaft hinterlegten Provider- oder Forge-Anmeldedaten.

Der Session-Dispatch von Worker zu Worker ist in diesem Modus nicht verfügbar. Platzierung und
Dispatch verbleiben in der Zuständigkeit des Gateways: Ein Operator kann eine bestehende lokale
Session in einem verwalteten Worktree über das Gateway dispatchen, während ein Worker-Prozess
weder sich selbst noch einen anderen Worker dispatchen kann.

Die vorbereitete Zuweisung enthält den Transkriptkontext, das akzeptierte Basis-Leaf,
die Commit-Sequenz und den Live-Event-Cursor. Bei einer erneuten Tunnelverbindung wird der Prozess
mit denselben Anmeldedaten und derselben Owner-Epoche erneut zugelassen, behält die akzeptierte
Transkriptbasis bei, spielt das noch nicht bestätigte Ende seiner Live-Events erneut ab und hängt einen
laufenden Inferenzdurchlauf mit derselben Identität wieder an. Die abschließende Inferenznachricht
ist maßgeblich, wenn gestreamte Deltas verpasst wurden. Eine ablösende Owner-Epoche
grenzt den Prozess aus und bewirkt ein ordnungsgemäßes Beenden.

Eine `stale-base-leaf`-Transkriptablehnung stoppt den aktuellen Lauf unmittelbar. Der Worker-
Modus versucht die abgelehnte Sequenz nicht erneut mit einem anderen Leaf, sodass kein
doppelter Commit erzeugt wird; ein noch nicht committetes, im Arbeitsspeicher befindliches Ende dieses
Laufs geht verloren. Der Neustart liegt in der Zuständigkeit des Platzierungs-Owners von Meilenstein 3, der
aus dem maßgeblichen Transkript und Commit-Ledger des Gateways eine neue Zuweisung
erstellen muss. Ebenso beendet ein Neustart des Gateway-Prozesses einen ausstehenden
Inferenzdurchlauf mit einem Provider-Fehler; nur eine erneute Verbindung des Tunnels oder Worker-WebSockets
kann sich wieder an einen aktiven Inferenzstream desselben Prozesses anhängen.

Informationen zur geschlossenen Worker-RPC-Oberfläche finden Sie unter [Gateway-Protokoll](/de/gateway/protocol#worker-role-and-closed-protocol) und
Informationen zum Architektur- und Sicherheitsmodell unter [Plan für Cloud-Worker](/de/plan/cloud-workers).
