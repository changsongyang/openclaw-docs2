---
read_when:
    - Endpunkte hinzufügen/ändern
    - Debugging von CLI-↔-Registry-Anfragen
summary: HTTP-API-Referenz (öffentliche Endpunkte + CLI-Endpunkte + Authentifizierung).
x-i18n:
    generated_at: "2026-07-24T20:23:40Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 5b180bbd56d20a3d88c1fe74ccab0fd0ecbe0e8c9624cd1afd2070a2ca1f7fb3
    source_path: clawhub/http-api.md
    workflow: 16
---

# HTTP-API

Basis-URL: `https://clawhub.ai` (Standard).

Alle v1-Pfade befinden sich unter `/api/v1/...`.
Die veralteten Pfade `/api/...` und `/api/cli/...` bleiben aus Kompatibilitätsgründen bestehen (siehe `DEPRECATIONS.md`).
OpenAPI: `/api/v1/openapi.json`.

## Wiederverwendung des öffentlichen Katalogs

Verzeichnisse von Drittanbietern dürfen die öffentlichen Leseendpunkte verwenden, um ClawHub-Skills aufzulisten oder zu durchsuchen. Zwischenspeichern Sie die Ergebnisse, beachten Sie `429`/`Retry-After`, verlinken Sie Benutzer zurück zum kanonischen ClawHub-Eintrag (`https://clawhub.ai/<owner>/skills/<slug>`) und vermeiden Sie den Eindruck, ClawHub unterstütze die Drittanbieter-Website. Versuchen Sie nicht, verborgene, private oder durch Moderation gesperrte Inhalte außerhalb der öffentlichen API-Oberfläche zu spiegeln.

Web-Slug-Kurzformen werden über Registry-Familien hinweg aufgelöst, API-Clients sollten jedoch
die von Leseendpunkten zurückgegebenen kanonischen URLs verwenden, anstatt die
Routenpriorität zu rekonstruieren.

## Ratenbegrenzungen

Durchsetzungsmodell:

- Anonyme Anfragen: Durchsetzung pro IP-Adresse.
- Authentifizierte Anfragen (gültiges Bearer-Token): Durchsetzung pro Benutzer-Bucket.
- Wenn das Token fehlt oder ungültig ist, wird ersatzweise die Durchsetzung pro IP-Adresse verwendet.
- Authentifizierte Schreibendpunkte sollten nicht nur `Unauthorized` zurückgeben, wenn
  dem Server der Grund bekannt ist. Fehlende Tokens, ungültige/widerrufene Tokens und
  gelöschte/gesperrte/deaktivierte Konten sollten jeweils einen aussagekräftigen Text erhalten, damit CLI-
  Clients den Benutzern mitteilen können, wodurch sie blockiert wurden.

- Lesen: 3000/min pro IP-Adresse, 12000/min pro Schlüssel
- Schreiben: 300/min pro IP-Adresse, 3000/min pro Schlüssel
- Download: 1200/min pro IP-Adresse, 6000/min pro Schlüssel (Download-Endpunkte)

Header:

- Abwärtskompatibilität: `X-RateLimit-Limit`, `X-RateLimit-Reset`
- Standardisiert: `RateLimit-Limit`, `RateLimit-Reset`
- Bei `429`: `X-RateLimit-Remaining: 0` und `RateLimit-Remaining: 0`
- Bei `429`: `Retry-After`

Header-Semantik:

- `X-RateLimit-Reset`: absolute Sekunden seit der Unix-Epoche
- `RateLimit-Reset`: Sekunden bis zur Zurücksetzung (Verzögerung)
- `X-RateLimit-Remaining` / `RateLimit-Remaining`: exaktes verbleibendes Kontingent, sofern vorhanden.
  Erfolgreiche verteilte Anfragen lassen diesen Header aus, anstatt einen ungefähren globalen Wert zurückzugeben.
- `Retry-After`: Wartezeit in Sekunden vor einem erneuten Versuch (Verzögerung) bei `429`

Beispielantwort für `429`:

```http
HTTP/2 429
content-type: text/plain; charset=utf-8
x-ratelimit-limit: 20
x-ratelimit-remaining: 0
x-ratelimit-reset: 1771404540
ratelimit-limit: 20
ratelimit-remaining: 0
ratelimit-reset: 34
retry-after: 34

Ratenbegrenzung überschritten
```

Hinweise für Clients:

- Wenn `Retry-After` vorhanden ist, warten Sie vor dem erneuten Versuch entsprechend viele Sekunden.
- Verwenden Sie einen Backoff mit Zufallskomponente, um synchronisierte Wiederholungsversuche zu vermeiden.
- Wenn `Retry-After` fehlt, verwenden Sie ersatzweise `RateLimit-Reset` (oder berechnen Sie den Wert aus `X-RateLimit-Reset`).

IP-Quelle:

- Vertrauenswürdige Client-IP-Header, einschließlich `cf-connecting-ip`, werden nur verwendet, wenn die
  Bereitstellung vertrauenswürdige weitergeleitete Header ausdrücklich aktiviert.
- ClawHub verwendet vertrauenswürdige Weiterleitungs-Header, um Client-IP-Adressen am Netzwerkrand zu identifizieren.
- Wenn keine vertrauenswürdige Client-IP-Adresse verfügbar ist, verwenden anonyme Anfragen Ersatz-Buckets,
  deren Geltungsbereich ausschließlich durch die Art der Ratenbegrenzung bestimmt wird. Diese Ersatz-Buckets enthalten keine
  vom Aufrufer bereitgestellten Pfade, Slugs, Paketnamen, Versionen, Abfragezeichenfolgen oder sonstigen
  Artefaktparameter.

## Fehlerantworten

Öffentliche v1-Fehlerantworten sind Klartext mit `content-type: text/plain; charset=utf-8`.
Dies umfasst Validierungsfehler (`400`), fehlende öffentliche Ressourcen (`404`), Authentifizierungs- und
Berechtigungsfehler (`401`/`403`), Ratenbegrenzungen (`429`) und blockierte Downloads. Clients
sollten den Antworttext als menschenlesbare Zeichenfolge auslesen. Unbekannte Abfrageparameter werden
aus Kompatibilitätsgründen ignoriert, erkannte Abfrageparameter mit ungültigen Werten geben jedoch
`400` zurück.

## Öffentliche Endpunkte (keine Authentifizierung)

### `GET /api/v1/search`

Abfrageparameter:

- `q` (erforderlich): Abfragezeichenfolge
- `limit` (optional): Ganzzahl
- `highlightedOnly` (optional): `true`, um auf hervorgehobene Skills zu filtern
- `nonSuspiciousOnly` (optional): `true`, um verdächtige (`flagged.suspicious`) Skills auszublenden
- `nonSuspicious` (optional): veralteter Alias für `nonSuspiciousOnly`

Antwort:

```json
{
  "results": [
    {
      "score": 0.123,
      "slug": "gifgrep",
      "displayName": "GifGrep",
      "summary": "…",
      "version": "1.2.3",
      "updatedAt": 1730000000000,
      "ownerHandle": "openclaw",
      "owner": {
        "handle": "openclaw",
        "displayName": "OpenClaw",
        "image": "https://example.com/avatar.png"
      }
    }
  ]
}
```

Hinweise:

- Ergebnisse werden nach Relevanz sortiert zurückgegeben (Ähnlichkeit der Einbettungen + Verstärkungen für exakte Slug-/Namenstoken + eine geringe vorherige Gewichtung der Popularität).
- Die Relevanz wird stärker gewichtet als die Popularität. Eine genaue Übereinstimmung mit einem Slug- oder Anzeigenamen-Token kann eine weniger genaue Übereinstimmung mit deutlich stärkerer Interaktion übertreffen.
- ASCII-Text wird an Wort- und Interpunktionsgrenzen tokenisiert. Beispielsweise enthält `personal-map` ein eigenständiges `map`-Token, während `amap-jsapi-skill` die Tokens `amap`, `jsapi` und `skill` enthält; eine Suche nach `map` ergibt daher für `personal-map` eine stärkere lexikalische Übereinstimmung als für `amap-jsapi-skill`.
- Die Popularität wird logarithmisch skaliert und begrenzt. Skills mit hoher Interaktion können niedriger eingestuft werden, wenn der Abfragetext weniger gut übereinstimmt.
- Ein verdächtiger oder verborgener Moderationsstatus kann einen Skill abhängig von den Filtern des Aufrufers und dem aktuellen Moderationsstatus aus der öffentlichen Suche entfernen.

Hinweise zur Auffindbarkeit für Herausgeber:

- Fügen Sie die Begriffe, nach denen Benutzer tatsächlich suchen werden, in den Anzeigenamen, die Zusammenfassung und die Tags ein. Verwenden Sie ein eigenständiges Slug-Token nur, wenn es zugleich eine stabile Identität ist, die Sie beibehalten möchten.
- Benennen Sie einen Slug nicht nur deshalb um, um bei einer einzelnen Abfrage besser abzuschneiden, es sei denn, der neue Slug ist langfristig ein besserer kanonischer Name. Alte Slugs werden zu Weiterleitungsaliasen, die kanonische URL, der angezeigte Slug und zukünftige Suchzusammenfassungen verwenden jedoch den neuen Slug.
- Umbenennungsaliase erhalten die Auflösung für alte URLs und Installationen, die über die Registry aufgelöst werden. Die Suchrangfolge basiert jedoch auf den kanonischen Skill-Metadaten, nachdem die Umbenennung indiziert wurde. Vorhandene Statistiken bleiben dem Skill zugeordnet.
- Wenn ein Skill unerwartet nicht sichtbar ist, prüfen Sie im angemeldeten Zustand zunächst den Moderationsstatus mit `clawhub inspect @owner/slug`, bevor Sie für die Rangfolge relevante Metadaten ändern.

### `GET /api/v1/skills`

Abfrageparameter:

- `limit` (optional): Ganzzahl (1–200)
- `cursor` (optional): Paginierungscursor für jede Sortierung außer `trending`
- `sort` (optional): `updated` (Standard), `recommended` (Alias: `default`), `createdAt` (Alias: `newest`), `downloads`, `stars` (Alias: `rating`), veraltete Installationsaliase `installsCurrent`/`installs`/`installsAllTime` werden `downloads` zugeordnet, `trending`
- `nonSuspiciousOnly` (optional): `true`, um verdächtige (`flagged.suspicious`) Skills auszublenden
- `nonSuspicious` (optional): veralteter Alias für `nonSuspiciousOnly`

Ungültige Werte für `sort` geben `400` zurück.

Hinweise:

- `recommended` verwendet Interaktions- und Aktualitätssignale.
- `trending` sortiert nach Installationen innerhalb der letzten 7 Tage (telemetriebasiert).
- `createdAt` ist für das Crawlen neuer Skills stabil; `updated` ändert sich, wenn vorhandene Skills erneut veröffentlicht werden.
- Bei `nonSuspiciousOnly=true` können cursorbasierte Sortierungen auf einer Seite weniger als `limit` Einträge zurückgeben, da verdächtige Skills nach dem Abrufen der Seite herausgefiltert werden.
- Verwenden Sie `nextCursor`, um die Paginierung fortzusetzen, sofern vorhanden. Eine kurze Seite bedeutet für sich allein nicht, dass das Ende der Ergebnisse erreicht ist.

Antwort:

```json
{
  "items": [
    {
      "slug": "gifgrep",
      "displayName": "GifGrep",
      "summary": "…",
      "topics": ["Productivity"],
      "tags": { "latest": "1.2.3" },
      "stats": {},
      "createdAt": 0,
      "updatedAt": 0,
      "latestVersion": { "version": "1.2.3", "createdAt": 0, "changelog": "…" },
      "metadata": { "os": ["macos"], "systems": ["aarch64-darwin"] }
    }
  ],
  "nextCursor": null
}
```

### `GET /api/v1/skills/{slug}`

Antwort:

```json
{
  "skill": {
    "slug": "gifgrep",
    "displayName": "GifGrep",
    "summary": "…",
    "topics": ["Productivity"],
    "tags": { "latest": "1.2.3" },
    "stats": {},
    "createdAt": 0,
    "updatedAt": 0
  },
  "latestVersion": { "version": "1.2.3", "createdAt": 0, "changelog": "…" },
  "metadata": { "os": ["macos"], "systems": ["aarch64-darwin"] },
  "owner": { "handle": "steipete", "displayName": "Peter", "image": null },
  "moderation": {
    "isSuspicious": false,
    "isMalwareBlocked": false,
    "verdict": "clean",
    "reasonCodes": [],
    "summary": null,
    "engineVersion": "v2.0.0",
    "updatedAt": 0
  }
}
```

Hinweise:

- Alte Slugs, die durch Umbenennungs-/Zusammenführungsabläufe des Eigentümers erstellt wurden, werden zum kanonischen Skill aufgelöst.
- `metadata.os`: im Frontmatter des Skills deklarierte Betriebssystemeinschränkungen (z. B. `["macos"]`, `["linux"]`). `null`, wenn nicht deklariert.
- `metadata.systems`: Nix-Systemziele (z. B. `["aarch64-darwin", "x86_64-linux"]`). `null`, wenn nicht deklariert.
- `metadata` ist `null`, wenn der Skill keine Plattformmetadaten besitzt.
- `moderation` ist nur enthalten, wenn der Skill gekennzeichnet ist oder sein Eigentümer ihn betrachtet.

### `GET /api/v1/skills/{slug}/moderation`

Gibt einen strukturierten Moderationsstatus zurück.

Antwort:

```json
{
  "moderation": {
    "isSuspicious": true,
    "isMalwareBlocked": false,
    "verdict": "suspicious",
    "reasonCodes": ["suspicious.dynamic_code_execution"],
    "summary": "Detected: suspicious.dynamic_code_execution",
    "engineVersion": "v2.0.0",
    "updatedAt": 0,
    "legacyReason": null,
    "evidence": [
      {
        "code": "suspicious.dynamic_code_execution",
        "severity": "critical",
        "file": "index.ts",
        "line": 3,
        "message": "Dynamic code execution detected.",
        "evidence": ""
      }
    ]
  }
}
```

Hinweise:

- Eigentümer und Moderatoren können auf Moderationsdetails für verborgene Skills zugreifen.
- Öffentliche Aufrufer erhalten `200` nur für bereits gekennzeichnete, sichtbare Skills.
- Belege werden für öffentliche Aufrufer geschwärzt und enthalten nur für Eigentümer/Moderatoren unverarbeitete Ausschnitte.

### `POST /api/v1/skills/{slug}/report`

Meldet einen Skill zur Überprüfung durch Moderatoren. Meldungen beziehen sich auf den gesamten Skill, können optional
mit einer Version verknüpft sein und werden in die Warteschlange für Skill-Meldungen aufgenommen.

Authentifizierung:

- Erfordert ein API-Token.

Anfrage:

```json
{ "reason": "Verdächtiger Installationsschritt", "version": "1.2.3" }
```

Antwort:

```json
{
  "ok": true,
  "reported": true,
  "alreadyReported": false,
  "reportId": "skillReports:...",
  "skillId": "skills:...",
  "reportCount": 1
}
```

### `GET /api/v1/skills/-/reports`

Moderator-/Administratorendpunkt für die Annahme von Skill-Meldungen.

Abfrageparameter:

- `status` (optional): `open` (Standard), `confirmed`, `dismissed` oder `all`
- `limit` (optional): Ganzzahl (1-200)
- `cursor` (optional): Paginierungscursor

Antwort:

```json
{
  "items": [
    {
      "reportId": "skillReports:...",
      "skillId": "skills:...",
      "skillVersionId": "skillVersions:...",
      "slug": "gifgrep",
      "displayName": "GifGrep",
      "version": "1.2.3",
      "reason": "Verdächtiger Installationsschritt",
      "status": "open",
      "createdAt": 1730000000000,
      "reporter": {
        "userId": "users:...",
        "handle": "reporter",
        "displayName": "Meldende Person"
      },
      "triagedAt": null,
      "triagedBy": null,
      "triageNote": null
    }
  ],
  "nextCursor": null,
  "done": true
}
```

### `POST /api/v1/skills/-/reports/{reportId}/triage`

Moderator-/Admin-Endpunkt zum Abschließen oder erneuten Öffnen von Skill-Meldungen.

Anfrage:

```json
{ "status": "confirmed", "note": "Geprüft und betroffene Version ausgeblendet.", "finalAction": "hide" }
```

`note` ist für `confirmed` und `dismissed` erforderlich; der Wert kann weggelassen werden, wenn
`status` wieder auf `open` gesetzt wird. Übergeben Sie `finalAction: "hide"` zusammen mit einer triagierten
Meldung, um den Skill im selben auditierbaren Workflow auszublenden.

### `GET /api/v1/skills/{slug}/versions`

Abfrageparameter:

- `limit` (optional): Ganzzahl
- `cursor` (optional): Paginierungscursor

### `GET /api/v1/skills/{slug}/versions/{version}`

Gibt Versionsmetadaten und eine Dateiliste zurück.

- `version.security` enthält, sofern verfügbar, den normalisierten Status der Scanverifizierung und Scannerdetails
  (VirusTotal + LLM).

### `GET /api/v1/skills/{slug}/scan`

Gibt Details zur Verifizierung des Sicherheitsscans für eine Skill-Version zurück.

Abfrageparameter:

- `version` (optional): bestimmte Versionszeichenfolge.
- `tag` (optional): eine mit einem Tag versehene Version auflösen (zum Beispiel `latest`).

Hinweise:

- Wenn weder `version` noch `tag` angegeben ist, wird die neueste Version verwendet.
- Enthält den normalisierten Verifizierungsstatus sowie scannerspezifische Details.
- `security.hasScanResult` ist nur dann `true`, wenn ein Scanner ein eindeutiges Urteil erzeugt hat (`clean`, `suspicious` oder `malicious`).
- `moderation` ist eine aktuelle, aus der neuesten Version abgeleitete Moderationsmomentaufnahme auf Skill-Ebene.
- Prüfen Sie bei der Abfrage einer historischen Version `moderation.matchesRequestedVersion` und `moderation.sourceVersion`, bevor Sie `moderation` und `security` als denselben Versionskontext behandeln.

### `POST /api/v1/skills/-/scan`

Authentifizierter Übermittlungsendpunkt für neue ClawScan-Aufträge.

Scans lokaler Uploads werden nicht mehr unterstützt. Anfragen mit
`multipart/form-data` oder `{ "source": { "kind": "upload" } }` geben `410` zurück.

Veröffentlichte Scans verwenden JSON:

```json
{
  "source": { "kind": "published", "slug": "gifgrep", "version": "1.2.3" },
  "update": false
}
```

Hinweise:

- Nutzdaten von Scananfragen und herunterladbare Berichte verfallen nach Ablauf des Aufbewahrungszeitraums im Speicher für Scananfragen.
- Veröffentlichte Scans erfordern Verwaltungszugriff als Eigentümer/Publisher oder Moderator-/Admin-Berechtigungen der Plattform.
- Veröffentlichte Scans schreiben Ergebnisse nur zurück, wenn `update: true` gilt und der Scan erfolgreich abgeschlossen wird.
- Die Antwort lautet `202` mit `{ "ok": true, "scanId": "...", "jobId": "...", "status": "queued", "sourceKind": "published", "update": false, "queue": { "queuedAhead": 0, "queuedAheadIsEstimate": false, "position": 1, "running": 0, "runningIsEstimate": false, "note": "Scans are asynchronous and may take time to complete." } }`.
- Scanaufträge werden asynchron ausgeführt. Manuelle Scananfragen werden gegenüber normalen Veröffentlichungs-/Backfill-Aufträgen priorisiert, der Abschluss hängt jedoch weiterhin von der Verfügbarkeit der Worker ab.

### `GET /api/v1/skills/-/scan/{scanId}`

Authentifizierter Abfrageendpunkt für einen übermittelten Scan.

- Gibt den Status „in Warteschlange“, „wird ausgeführt“, „erfolgreich“ oder „fehlgeschlagen“ zurück.
- Gibt während der Wartezeit `queue.queuedAhead` und `queue.position` zurück, damit Clients anzeigen können, wie viele priorisierte manuelle Scans der Anfrage vorausgehen. Sehr große Warteschlangen werden begrenzt und mit `queuedAheadIsEstimate: true` gemeldet.
- Sofern verfügbar, enthält `report` die Abschnitte `clawscan`, `skillspector`, `staticAnalysis` und `virustotal`.
- Fehlgeschlagene Scanaufträge geben `status: "failed"` mit `lastError` zurück.

### `GET /api/v1/skills/-/scan/{scanId}/download`

Authentifizierter Endpunkt für Berichtsarchive.

- Erfordert einen erfolgreichen Scan; noch nicht abgeschlossene Scans geben `409` zurück.
- Gibt eine ZIP-Datei mit `manifest.json`, `clawscan.json`, `skillspector.json`, `static-analysis.json`, `virustotal.json` und `README.md` zurück.

### `GET /api/v1/skills/-/scan/download/{name}?version=<version>&kind=skill|plugin`

Authentifizierter Endpunkt für gespeicherte Berichtsarchive übermittelter Versionen.

- Erfordert Verwaltungszugriff als Eigentümer/Publisher auf den Skill oder das Plugin oder Moderator-/Admin-Berechtigungen der Plattform.
- Gibt gespeicherte Scanergebnisse für die exakt übermittelte Version zurück, einschließlich blockierter oder ausgeblendeter Versionen.
- `kind` ist standardmäßig `skill`; verwenden Sie `kind=plugin` für Plugin-/Paketscans.
- Gibt dieselbe ZIP-Struktur wie Downloads von Scananfragen zurück.

### `POST /api/v1/skills/-/scan/batch`

Nur für Admins zugängliche kanonische Route für erneute Batch-Scans. Sie akzeptiert dieselbe Nutzdatenstruktur wie das veraltete `POST /api/v1/skills/-/rescan-batch`.

### `POST /api/v1/skills/-/scan/batch/status`

Nur für Admins zugängliche kanonische Route für den Batch-Status. Sie akzeptiert `{ "jobIds": ["..."] }` und gibt dieselben aggregierten Zähler wie das veraltete `POST /api/v1/skills/-/rescan-batch/status` zurück.

### `GET /api/v1/skills/{slug}/verify`

Gibt den von `clawhub skill verify` verwendeten Verifizierungsumschlag der Skill Card zurück.

Abfrageparameter:

- `version` (optional): bestimmte Versionszeichenfolge.
- `tag` (optional): eine mit einem Tag versehene Version auflösen (zum Beispiel `latest`).

Hinweise:

- `ok` ist nur dann `true`, wenn die ausgewählte Version über eine generierte Skill Card verfügt, nicht durch die Malware-Moderation blockiert ist und die ClawScan-Verifizierung unbedenklich ist.
- Skill-Identität, Publisher-Identität und Metadaten der ausgewählten Version sind Felder der obersten Umschlagebene (`slug`, `displayName`, `publisherHandle`, `version`, `resolvedFrom`, `tag`, `createdAt`), damit Shell-Automatisierungen sie lesen können, ohne verschachtelte Wrapper entpacken zu müssen.
- `security` ist das ClawScan-/Sicherheitsurteil der obersten Ebene. Automatisierungen sollten sich an `ok`, `decision`, `reasons` und `security.status` orientieren.
- `security.signals` enthält unterstützende Scannerbelege wie `staticScan`, `virusTotal` und `skillSpector`.
- `security.signals.dependencyRegistry` bleibt zur Kompatibilität mit v1-Antworten erhalten, der Scanner zur Existenzprüfung in Abhängigkeitsregistrys wurde jedoch eingestellt und dieser Schlüssel ist immer `null`.
- `provenance` ist nur dann `server-resolved-github-import`, wenn ClawHub während der Veröffentlichung oder des Imports ein GitHub-Repository/eine Referenz/einen Commit/einen Pfad aufgelöst und gespeichert hat; andernfalls ist der Wert `unavailable`.

### `POST /api/v1/skills/-/security-verdicts`

Gibt aktuelle kompakte Sicherheitsurteile für exakte Skill-Versionen zurück. Dieser
Sammlungsendpunkt ist für Clients vorgesehen, die bereits wissen, welche installierten
ClawHub-Skill-Versionen sie anzeigen müssen, beispielsweise die OpenClaw Control UI.

Anfrage:

```json
{
  "items": [{ "slug": "gifgrep", "version": "1.2.3" }]
}
```

Hinweise:

- `items` muss 1–100 eindeutige `{ slug, version }`-Paare enthalten.
- Die Ergebnisse werden pro Element zurückgegeben; ein fehlender Skill oder eine fehlende Version lässt nicht die gesamte Antwort fehlschlagen.
- Die Antwort enthält ausschließlich Sicherheitsinformationen. Sie enthält keine Skill-Card-Daten, keinen Status der generierten Karte, keine Artefaktdateilisten und keine detaillierten Scanner-Nutzdaten.
- `security.signals` enthält nur unterstützende Belege auf Statusebene; verwenden Sie `/scan` oder die ClawHub-Seite für Sicherheitsaudits, um vollständige Scannerdetails zu erhalten.
- `security.signals.dependencyRegistry` bleibt zur Kompatibilität mit v1-Antworten erhalten, der Scanner zur Existenzprüfung in Abhängigkeitsregistrys wurde jedoch eingestellt und dieser Schlüssel ist immer `null`.
- Das Fehlen einer Skill Card wirkt sich nicht auf `ok`, `decision` oder `reasons` dieses Endpunkts aus; Clients sollten das installierte `skill-card.md` lokal lesen, wenn sie Karteninhalte benötigen.
- Verwenden Sie `/verify`, wenn Sie den Verifizierungsumschlag der Skill Card für einen einzelnen Skill benötigen, `/card`, wenn Sie das generierte Karten-Markdown benötigen, und `/scan`, wenn Sie detaillierte Scannerdaten benötigen.

Antwort:

```json
{
  "schema": "clawhub.skill.security-verdicts.v1",
  "items": [
    {
      "ok": true,
      "decision": "pass",
      "reasons": [],
      "requestedSlug": "gifgrep",
      "slug": "gifgrep",
      "displayName": "GifGrep",
      "publisherHandle": "steipete",
      "publisherDisplayName": "Peter",
      "requestedVersion": "1.2.3",
      "version": "1.2.3",
      "createdAt": 0,
      "checkedAt": 0,
      "skillUrl": "https://clawhub.ai/steipete/skills/gifgrep",
      "securityAuditUrl": "https://clawhub.ai/steipete/skills/gifgrep/security-audit?version=1.2.3",
      "security": {
        "status": "clean",
        "passed": true,
        "signals": {
          "staticScan": { "status": "clean", "reasonCodes": [] },
          "virusTotal": null,
          "skillSpector": null,
          "dependencyRegistry": null
        }
      }
    },
    {
      "ok": false,
      "decision": "fail",
      "reasons": ["version.not_found"],
      "requestedSlug": "missing-version",
      "requestedVersion": "1.0.0",
      "error": { "code": "version_not_found", "message": "Version nicht gefunden" },
      "security": null
    }
  ]
}
```

### `GET /api/v1/skills/{slug}/file`

Gibt exakt gespeicherte Dateibytes als Download zurück. Fügen Sie `preview=1` hinzu, um eine begrenzte Vorschau
mit maskiertem Text anzufordern; jede Datei mit gültigen UTF-8-Bytes kann unabhängig von ihrer Erweiterung oder ihren
MIME-Metadaten in der Vorschau angezeigt werden.

Abfrageparameter:

- `path` (erforderlich)
- `version` (optional)
- `tag` (optional)
- `preview=1` (optional; gibt `text/plain` oder `415` zurück, wenn die Bytes kein gültiges UTF-8 darstellen)

Hinweise:

- Verwendet standardmäßig die neueste Version.
- Limit für Rohdownloads: 10MB.
- Limit für Textvorschauen: 200KB.

### `GET /api/v1/packages`

Einheitlicher Katalogendpunkt für:

- Skills
- Code-Plugins
- Bundle-Plugins

Abfrageparameter:

- `limit` (optional): Ganzzahl (1–100)
- `cursor` (optional): Paginierungscursor
- `family` (optional): `skill`, `code-plugin` oder `bundle-plugin`
- `channel` (optional): `official`, `community` oder `private`
- `isOfficial` (optional): `true` oder `false`
- `sort` (optional): `updated` (Standard), `recommended`, `trending`, `downloads`, veralteter Alias `installs`
- `category` (optional): Plugin-Kategoriefilter. Wird nur unterstützt, wenn die
  Anfrage auf Plugin-Pakete beschränkt ist (`/api/v1/plugins`,
  `/api/v1/code-plugins`, `/api/v1/bundle-plugins` oder Paketendpunkte mit
  `family=code-plugin`/`family=bundle-plugin`). Kontrollierte Kategorien und
  veraltete v1-Filteraliase sind unter `GET /api/v1/plugins` dokumentiert.

Hinweise:

- Ungültige Werte für `family`, `channel`, `isOfficial`, `featured`,
  `highlightedOnly` oder `sort` geben `400` zurück. Unbekannte Abfrageparameter werden ignoriert.
- `GET /api/v1/code-plugins` und `GET /api/v1/bundle-plugins` bleiben Aliase für feste Familien.
- Skill-Einträge werden weiterhin durch die Skill-Registry bereitgestellt und können nach wie vor nur über `POST /api/v1/skills` veröffentlicht werden.
- `POST /api/v1/packages` ist weiterhin ausschließlich für Veröffentlichungen von Code-Plugins und Bundle-Plugins vorgesehen.
- Anonyme Aufrufer sehen nur öffentliche Paketkanäle.
- Authentifizierte Aufrufer können in Listen-/Suchergebnissen private Pakete von Publishern sehen, denen sie angehören.
- `channel=private` gibt nur Pakete zurück, die der authentifizierte Aufrufer lesen darf.

### `GET /api/v1/packages/search`

Einheitliche Katalogsuche über Skills und Plugin-Pakete hinweg.

Abfrageparameter:

- `q` (erforderlich): Abfragezeichenfolge
- `limit` (optional): Ganzzahl (1–100)
- `family` (optional): `skill`, `code-plugin` oder `bundle-plugin`
- `channel` (optional): `official`, `community` oder `private`
- `isOfficial` (optional): `true` oder `false`
- `category` (optional): Filter für Plugin-Kategorien. Wird nur unterstützt, wenn die
  Anfrage auf Plugin-Pakete beschränkt ist. Kontrollierte Kategorien und ältere v1-
  Filteraliase sind unter `GET /api/v1/plugins` dokumentiert.

Hinweise:

- Ungültige Werte für `family`, `channel`, `isOfficial`, `featured` oder
  `highlightedOnly` geben `400` zurück. Unbekannte Abfrageparameter werden ignoriert.
- Anonyme Aufrufer sehen nur öffentliche Paketkanäle.
- Authentifizierte Aufrufer können private Pakete von Herausgebern durchsuchen, denen sie angehören.
- `channel=private` gibt nur Pakete zurück, die der authentifizierte Aufrufer lesen kann.

### `GET /api/v1/plugins`

Nur Plugins umfassende Katalogsuche in Code-Plugin- und Bundle-Plugin-Paketen.

Abfrageparameter:

- `limit` (optional): Ganzzahl (1-100)
- `cursor` (optional): Paginierungscursor
- `isOfficial` (optional): `true` oder `false`
- `sort` (optional): `recommended` (Standard), `trending`, `downloads`, `updated`, älterer Alias `installs`
- `category` (optional): Filter für Plugin-Kategorien. Aktuelle Werte:
  `channels`, `models`, `memory`, `context`, `voice`, `media`, `web`,
  `tools`, `runtime`, `gateway`, `security`, `other`.

Ältere v1-Filteraliase werden an Leseendpunkten weiterhin akzeptiert:

- `mcp-tooling`, `data` und `automation` werden zu `tools` aufgelöst.
- `observability` und `deployment` werden zu `gateway` aufgelöst.
- `dev-tools` wird zu `runtime` aufgelöst.

`trending` ist eine Rangliste der Installationen/Downloads über sieben Tage und verwendet keine Gesamtwerte über den gesamten Zeitraum.
Am vereinheitlichten Endpunkt `/api/v1/packages` gilt sie nur für Plugins; verwenden Sie
`/api/v1/skills?sort=trending` für den Skills-Katalog.

Ältere Aliase werden nicht als gespeicherte oder von Autoren deklarierte Kategoriewerte akzeptiert.

### `GET /api/v1/skills/export`

Massenexport der neuesten öffentlichen Skills zur Offlineanalyse.

Authentifizierung:

- API-Token erforderlich.

Abfrageparameter:

- `startDate` (erforderlich): Untergrenze in Unix-Millisekunden für Skill-`updatedAt`.
- `endDate` (erforderlich): Obergrenze in Unix-Millisekunden für Skill-`updatedAt`.
- `limit` (optional): Ganzzahl (1-250), Standardwert `250`.
- `cursor` (optional): Paginierungscursor aus der vorherigen Antwort.

Antwort:

- Textkörper: ZIP-Archiv.
- Jeder exportierte Skill hat sein Stammverzeichnis unter `{publisher}/{slug}/`.
- Gehostete Skills enthalten die Dateien der neuesten gespeicherten Version und sind in
  `_manifest.json` mit `sourceRef: "public-clawhub"` aufgeführt.
- Aktuelle GitHub-gestützte Skills mit einem `clean`- oder `suspicious`-Scan enthalten
  `_source_handoff.json` mit `sourceRef: "public-github"`, Repository, Commit, Pfad,
  Inhalts-Hash und Archiv-URL. Sie enthalten keine von ClawHub gehosteten Quelldateien.
- Jeder Skill enthält `_export_skill_meta.json`.
- `_manifest.json` ist immer im Stammverzeichnis der ZIP-Datei enthalten.
- `_errors.json` ist enthalten, wenn einzelne Skills oder Dateien nicht
  exportiert werden konnten.

Header:

- `X-Next-Cursor`
- `X-Has-More`
- `X-Total-Returned`
- `X-Date-Range`
- `X-Export-Errors`

### `GET /api/v1/plugins/export`

Massenexport der neuesten öffentlichen Plugin-Releases zur Offlineanalyse.

Authentifizierung:

- API-Token erforderlich.

Abfrageparameter:

- `startDate` (erforderlich): Untergrenze in Unix-Millisekunden für Plugin-`updatedAt`.
- `endDate` (erforderlich): Obergrenze in Unix-Millisekunden für Plugin-`updatedAt`.
- `limit` (optional): Ganzzahl (1-250), Standardwert `250`.
- `cursor` (optional): Paginierungscursor aus der vorherigen Antwort.
- `family` (optional): `code-plugin` oder `bundle-plugin`. Wenn weggelassen, sind beide
  Plugin-Familien gemeint.

Antwort:

- Textkörper: ZIP-Archiv.
- Jedes exportierte Plugin hat sein Stammverzeichnis unter `{family}/{packageName}/`.
- Jedes exportierte Plugin enthält die gespeicherten Dateien des neuesten Releases.
- Die Exportmetadaten jedes Plugins werden unter
  `__clawhub_export/{family}/{packageName}/plugin_meta.json` gespeichert.
- `_manifest.json` ist immer im Stammverzeichnis der ZIP-Datei enthalten.
- `_errors.json` ist enthalten, wenn einzelne Plugins oder Dateien nicht
  exportiert werden konnten.

Header:

- `X-Next-Cursor`
- `X-Has-More`
- `X-Total-Returned`
- `X-Date-Range`
- `X-Export-Errors`

### `GET /api/v1/plugins/search`

Nur Plugins umfassende Suche in Code-Plugin- und Bundle-Plugin-Paketen.

Abfrageparameter:

- `q` (erforderlich): Abfragezeichenfolge
- `limit` (optional): Ganzzahl (1-100)
- `isOfficial` (optional): `true` oder `false`
- `category` (optional): Filter für Plugin-Kategorien. Aktuelle Werte:
  `channels`, `models`, `memory`, `context`, `voice`, `media`, `web`,
  `tools`, `runtime`, `gateway`, `security`, `other`.

Hinweise:

- Die unter `GET /api/v1/plugins` dokumentierten älteren v1-Filteraliase werden ebenfalls
  akzeptiert.
- Die Kategoriefilterung ist ein echter API-Filter, der auf Digest-Zeilen für Plugin-Kategorien
  basiert, und keine Umschreibung der Suchabfrage.
- Die Ergebnisse werden nach Relevanz sortiert zurückgegeben und derzeit nicht paginiert.
- Die Sortiersteuerelemente der Browser-Benutzeroberfläche für die Plugin-Suche ordnen die geladenen Relevanzergebnisse neu,
  entsprechend dem aktuellen Suchverhalten von `/skills`.

### `GET /api/v1/packages/{name}`

Gibt Detailmetadaten des Pakets zurück.

Hinweise:

- Skills können im vereinheitlichten Katalog ebenfalls über diese Route aufgelöst werden.
- Private Pakete geben `404` zurück, sofern der Aufrufer nicht den besitzenden Herausgeber lesen kann.

### `DELETE /api/v1/packages/{name}`

Löscht ein Paket und alle Releases vorläufig.

Hinweise:

- Erfordert ein API-Token für den Paketbesitzer, einen Besitzer/Administrator der herausgebenden Organisation,
  einen Plattformmoderator oder einen Plattformadministrator.

### `GET /api/v1/packages/{name}/versions`

Gibt den Versionsverlauf zurück.

Abfrageparameter:

- `limit` (optional): Ganzzahl (1–100)
- `cursor` (optional): Paginierungscursor

Hinweise:

- Private Pakete geben `404` zurück, sofern der Aufrufer nicht den besitzenden Herausgeber lesen kann.

### `GET /api/v1/packages/{name}/versions/{version}`

Gibt eine Paketversion zurück, einschließlich Dateimetadaten, Kompatibilität,
Verifizierung, Artefaktmetadaten und Scandaten.

Hinweise:

- `version.artifact.kind` ist `legacy-zip` für Paketarchive der alten Welt oder
  `npm-pack` für ClawPack-gestützte Releases.
- ClawPack-Releases enthalten npm-kompatible Felder `npmIntegrity`, `npmShasum` und
  `npmTarballName`.
- `version.sha256hash` sind veraltete Kompatibilitätsmetadaten für alte Clients. Sie
  bilden einen Hash über die exakten ZIP-Bytes, die von `/api/v1/packages/{name}/download` zurückgegeben werden.
  Moderne Clients sollten `version.artifact.sha256` verwenden, das das
  kanonische Release-Artefakt identifiziert.
- `version.vtAnalysis`, `version.llmAnalysis` und `version.staticScan` sind
  enthalten, wenn Scandaten vorhanden sind.
- Private Pakete geben `404` zurück, sofern der Aufrufer nicht den besitzenden Herausgeber lesen kann.

### `GET /api/v1/packages/{name}/versions/{version}/security`

Gibt für Installationsclients die genaue Sicherheits- und Vertrauenszusammenfassung des Paket-Releases
zurück. Dies ist die öffentliche OpenClaw-Nutzungsoberfläche für die Entscheidung, ob ein
aufgelöstes Release installiert werden darf.

Authentifizierung:

- Öffentlicher Leseendpunkt. Es ist kein Token eines Besitzers, Herausgebers, Moderators oder Administrators
  erforderlich.

Antwort:

```json
{
  "package": {
    "name": "@openclaw/example-plugin",
    "displayName": "Example Plugin",
    "family": "code-plugin"
  },
  "release": {
    "releaseId": "packageReleases:...",
    "version": "1.2.3",
    "artifactKind": "npm-pack",
    "artifactSha256": "0123456789abcdef...",
    "npmIntegrity": "sha512-...",
    "npmShasum": "0123456789abcdef0123456789abcdef01234567",
    "npmTarballName": "example-plugin-1.2.3.tgz",
    "createdAt": 1730000000000
  },
  "trust": {
    "scanStatus": "malicious",
    "moderationState": "quarantined",
    "blockedFromDownload": true,
    "reasons": ["manual:quarantined", "scan:malicious"],
    "pending": false,
    "stale": false
  }
}
```

Antwortfelder:

- `package.name`, `package.displayName` und `package.family` identifizieren das
  aufgelöste Registry-Paket.
- `release.releaseId`, `release.version` und `release.createdAt` identifizieren das
  exakt ausgewertete Release.
- `release.artifactKind`, `release.artifactSha256`, `release.npmIntegrity`,
  `release.npmShasum` und `release.npmTarballName` sind vorhanden, wenn sie für
  das Release-Artefakt bekannt sind.
- `trust.scanStatus` ist der effektive Vertrauensstatus, der aus Scanner-Eingaben
  und der manuellen Release-Moderation abgeleitet wird.
- `trust.moderationState` kann null sein. Der Wert ist `null`, wenn keine manuelle Release-
  Moderation vorhanden ist.
- `trust.blockedFromDownload` ist das Signal zur Installationssperre. OpenClaw und andere
  Installationsclients sollten die Installation blockieren, wenn dieser Wert `true` ist, statt
  die Sperrregeln aus Scanner- oder Moderationsfeldern erneut abzuleiten.
- `trust.reasons` ist die benutzerseitige und für Audits bestimmte Erklärungsliste. Ursachencodes
  sind stabile, kompakte Zeichenfolgen wie `manual:quarantined`, `scan:malicious`
  und `package:malicious`.
- `trust.pending` bedeutet, dass mindestens eine Vertrauenseingabe noch auf ihren Abschluss wartet.
- `trust.stale` bedeutet, dass die Vertrauenszusammenfassung aus veralteten Eingaben berechnet wurde und
  vor einer Freigabeentscheidung mit hoher Sicherheit als aktualisierungsbedürftig behandelt werden sollte.

Hinweise:

- Dieser Endpunkt ist versionsgenau. Clients sollten ihn nach der Auflösung der
  zu installierenden Paketversion aufrufen, nicht bereits nach dem Lesen der neuesten
  Paketmetadaten.
- Private Pakete geben `404` zurück, sofern der Aufrufer nicht den besitzenden Herausgeber lesen kann.
- Dieser Endpunkt ist absichtlich enger gefasst als Moderationsendpunkte für Besitzer/Moderatoren.
  Er stellt die Installationsentscheidung und die öffentliche Erklärung bereit, nicht
  die Identitäten der Meldenden, Meldungsinhalte, privaten Beweise oder internen
  Prüfungszeitleisten.

### `GET /api/v1/packages/{name}/versions/{version}/artifact`

Gibt die expliziten Metadaten des Artefakt-Resolvers für eine Paketversion zurück.

Hinweise:

- Ältere Paketversionen geben ein `legacy-zip`-Artefakt und eine ältere ZIP-
  `downloadUrl` zurück.
- ClawPack-Versionen geben ein `npm-pack`-Artefakt, npm-Integritätsfelder, eine
  `tarballUrl` und die ältere ZIP-Kompatibilitäts-URL zurück.
- Dies ist die Resolver-Oberfläche von OpenClaw; sie vermeidet das Erraten des Archivformats anhand
  einer gemeinsam verwendeten URL.

### `GET /api/v1/packages/{name}/versions/{version}/artifact/download`

Lädt das Versionsartefakt über den expliziten Resolver-Pfad herunter.

Hinweise:

- ClawPack-Versionen streamen exakt die Bytes des hochgeladenen npm-Packs `.tgz`.
- Ältere ZIP-Versionen leiten zu `/api/v1/packages/{name}/download?version=` weiter.
- Verwendet das Ratenlimit-Kontingent für Downloads.

### `GET /api/v1/packages/{name}/readiness`

Gibt die berechnete Bereitschaft für die zukünftige Nutzung durch OpenClaw zurück.

Die Bereitschaftsprüfungen umfassen:

- Status des offiziellen Kanals
- Verfügbarkeit der neuesten Version
- Verfügbarkeit des ClawPack-npm-Pack-Artefakts
- Artefakt-Digest
- Herkunft von Quell-Repository und Commit
- OpenClaw-Kompatibilitätsmetadaten
- Host-Ziele
- Scanstatus

Antwort:

```json
{
  "package": {
    "name": "@openclaw/example-plugin",
    "displayName": "Beispiel-Plugin",
    "family": "code-plugin",
    "isOfficial": true,
    "latestVersion": "1.2.3"
  },
  "ready": false,
  "checks": [
    {
      "id": "clawpack",
      "label": "ClawPack-Artefakt",
      "status": "fail",
      "message": "Die neueste Version ist eine ältere, ausschließlich als ZIP verfügbare Version."
    }
  ],
  "blockers": ["clawpack"]
}
```

### `GET /api/v1/packages/migrations`

Moderator-Endpunkt zum Auflisten der Migrationszeilen offizieller OpenClaw-Plugins.

Authentifizierung:

- Erfordert ein API-Token für einen Moderator oder Administrator.

Abfrageparameter:

- `phase` (optional): `planned`, `published`, `clawpack-ready`,
  `legacy-zip-only`, `metadata-ready`, `blocked`, `ready-for-openclaw` oder
  `all` (Standardwert).
- `limit` (optional): Ganzzahl (1-100)
- `cursor` (optional): Paginierungscursor

Antwort:

```json
{
  "items": [
    {
      "migrationId": "officialPluginMigrations:...",
      "bundledPluginId": "core.search",
      "packageName": "@openclaw/search-plugin",
      "packageId": "packages:...",
      "owner": "platform",
      "sourceRepo": "openclaw/openclaw",
      "sourcePath": "plugins/search",
      "sourceCommit": "abc123",
      "phase": "blocked",
      "blockers": ["fehlendes ClawPack"],
      "hostTargetsComplete": true,
      "scanClean": false,
      "moderationApproved": false,
      "runtimeBundlesReady": false,
      "notes": null,
      "createdAt": 1760000000000,
      "updatedAt": 1760000000000
    }
  ],
  "nextCursor": null,
  "done": true
}
```

### `POST /api/v1/packages/migrations`

Administrator-Endpunkt zum Erstellen oder Aktualisieren einer Migrationszeile für ein offizielles Plugin.

Authentifizierung:

- Erfordert ein API-Token für einen Administrator.

Anfragetext:

```json
{
  "bundledPluginId": "core.search",
  "packageName": "@openclaw/search-plugin",
  "owner": "platform",
  "sourceRepo": "openclaw/openclaw",
  "sourcePath": "plugins/search",
  "sourceCommit": "abc123",
  "phase": "blocked",
  "blockers": ["fehlendes ClawPack"],
  "hostTargetsComplete": true,
  "scanClean": false,
  "moderationApproved": false,
  "runtimeBundlesReady": false,
  "notes": "Warten auf den Upload durch den Herausgeber"
}
```

Hinweise:

- `bundledPluginId` wird in Kleinbuchstaben normalisiert und ist der stabile Upsert-Schlüssel.
- `packageName` wird als npm-Name normalisiert; bei geplanten
  Migrationen kann das Paket fehlen.
- Dies verfolgt ausschließlich die Migrationsbereitschaft. Es verändert OpenClaw nicht und generiert
  keine ClawPacks.

### `GET /api/v1/packages/moderation/queue`

Moderator-/Administrator-Endpunkt für Warteschlangen zur Prüfung von Paketveröffentlichungen.

Authentifizierung:

- Erfordert ein API-Token für einen Moderator oder Administrator.

Abfrageparameter:

- `status` (optional): `open` (Standardwert), `blocked`, `manual` oder `all`
- `limit` (optional): Ganzzahl (1-100)
- `cursor` (optional): Paginierungscursor

Bedeutung der Statuswerte:

- `open`: verdächtige, bösartige, ausstehende, unter Quarantäne gestellte, widerrufene oder gemeldete Veröffentlichungen.
- `blocked`: unter Quarantäne gestellte, widerrufene oder bösartige Veröffentlichungen.
- `manual`: jede Veröffentlichung mit einer manuellen Moderationsüberschreibung.
- `all`: jede Veröffentlichung mit manueller Überschreibung, nicht sauberem Scanstatus oder Paketmeldung.

Antwort:

```json
{
  "items": [
    {
      "packageId": "packages:...",
      "releaseId": "packageReleases:...",
      "name": "@openclaw/example-plugin",
      "displayName": "Beispiel-Plugin",
      "family": "code-plugin",
      "channel": "community",
      "isOfficial": false,
      "version": "1.2.3",
      "createdAt": 1730000000000,
      "artifactKind": "npm-pack",
      "scanStatus": "malicious",
      "moderationState": "quarantined",
      "moderationReason": "manuelle Prüfung",
      "sourceRepo": "openclaw/example-plugin",
      "sourceCommit": "abc123",
      "reportCount": 2,
      "lastReportedAt": 1730000001000,
      "reasons": ["manual:quarantined", "scan:malicious", "reports:2"]
    }
  ],
  "nextCursor": null,
  "done": true
}
```

### `POST /api/v1/packages/{name}/report`

Meldet ein Paket zur Prüfung durch einen Moderator. Meldungen gelten für das gesamte Paket und können optional
mit einer Version verknüpft werden. Sie werden der Moderationswarteschlange hinzugefügt, blenden das Paket jedoch nicht automatisch aus und
blockieren Downloads nicht von selbst. Moderatoren sollten die Veröffentlichungsmoderation verwenden, um
Artefakte zu genehmigen, unter Quarantäne zu stellen oder zu widerrufen.

Authentifizierung:

- Erfordert ein API-Token.

Anfrage:

```json
{ "reason": "Verdächtige native Binärdatei", "version": "1.2.3" }
```

Antwort:

```json
{
  "ok": true,
  "reported": true,
  "alreadyReported": false,
  "packageId": "packages:...",
  "releaseId": "packageReleases:...",
  "reportCount": 1
}
```

### `GET /api/v1/packages/reports`

Moderator-/Administrator-Endpunkt zur Annahme von Paketmeldungen.

Authentifizierung:

- Erfordert ein API-Token für einen Moderator oder Administrator.

Abfrageparameter:

- `status` (optional): `open` (Standardwert), `confirmed`, `dismissed` oder `all`
- `limit` (optional): Ganzzahl (1-100)
- `cursor` (optional): Paginierungscursor

Antwort:

```json
{
  "items": [
    {
      "reportId": "packageReports:...",
      "packageId": "packages:...",
      "releaseId": "packageReleases:...",
      "name": "@openclaw/example-plugin",
      "displayName": "Beispiel-Plugin",
      "family": "code-plugin",
      "version": "1.2.3",
      "reason": "Verdächtige native Binärdatei",
      "status": "open",
      "createdAt": 1730000000000,
      "reporter": {
        "userId": "users:...",
        "handle": "reporter",
        "displayName": "Meldende Person"
      },
      "triagedAt": null,
      "triagedBy": null,
      "triageNote": null
    }
  ],
  "nextCursor": null,
  "done": true
}
```

### `GET /api/v1/packages/{name}/moderation`

Eigentümer-/Moderator-Endpunkt für die Sichtbarkeit der Paketmoderation.

Authentifizierung:

- Erfordert ein API-Token für den Paketeigentümer, ein Mitglied des Herausgebers, einen Moderator oder
  Administrator.

Antwort:

```json
{
  "package": {
    "packageId": "packages:...",
    "name": "@openclaw/example-plugin",
    "displayName": "Beispiel-Plugin",
    "family": "code-plugin",
    "channel": "community",
    "isOfficial": false,
    "reportCount": 2,
    "lastReportedAt": 1730000001000,
    "scanStatus": "malicious"
  },
  "latestRelease": {
    "releaseId": "packageReleases:...",
    "version": "1.2.3",
    "artifactKind": "npm-pack",
    "scanStatus": "malicious",
    "moderationState": "quarantined",
    "moderationReason": "manuelle Prüfung",
    "blockedFromDownload": true,
    "reasons": ["manual:quarantined", "scan:malicious", "reports:2"],
    "createdAt": 1730000000000
  }
}
```

### `POST /api/v1/packages/reports/{reportId}/triage`

Moderator-/Administrator-Endpunkt zum Abschließen oder erneuten Öffnen von Paketmeldungen.

Anfrage:

```json
{
  "status": "confirmed",
  "note": "Betroffene Veröffentlichung geprüft und unter Quarantäne gestellt.",
  "finalAction": "quarantine"
}
```

`note` ist für `confirmed` und `dismissed` erforderlich; beim
Zurücksetzen von `status` auf `open` kann es weggelassen werden. Übergeben Sie `finalAction: "quarantine"` oder
`finalAction: "revoke"` zusammen mit einer bestätigten Meldung, um im selben
prüfbaren Arbeitsablauf die Veröffentlichungsmoderation anzuwenden.

Antwort:

```json
{
  "ok": true,
  "reportId": "packageReports:...",
  "packageId": "packages:...",
  "status": "confirmed",
  "reportCount": 0
}
```

### `POST /api/v1/packages/{name}/versions/{version}/moderation`

Moderator-/Administrator-Endpunkt zur Prüfung von Paketveröffentlichungen.

Anfrage:

```json
{ "state": "quarantined", "reason": "Verdächtige native Nutzlast." }
```

Unterstützte Statuswerte:

- `approved`: manuell geprüft und zugelassen.
- `quarantined`: bis zur weiteren Klärung blockiert.
- `revoked`: blockiert, nachdem einer Veröffentlichung zuvor vertraut wurde.

Unter Quarantäne gestellte und widerrufene Veröffentlichungen geben über Artefakt-Downloadrouten `403` zurück.
Jede Änderung schreibt einen Eintrag in das Auditprotokoll.

### `GET /api/v1/packages/{name}/file`

Gibt exakt die gespeicherten Bytes einer Paketdatei als Download zurück. Fügen Sie `preview=1` hinzu, um dieselbe begrenzte
UTF-8-Textvorschau anzufordern, die für Skill-Dateien verwendet wird.

Abfrageparameter:

- `path` (erforderlich)
- `version` (optional)
- `tag` (optional)
- `preview=1` (optional; gibt `text/plain` oder `415` zurück, wenn die Bytes kein gültiges UTF-8 darstellen)

Hinweise:

- Verwendet standardmäßig die neueste Veröffentlichung.
- Verwendet das Ratenlimit-Kontingent für Lesezugriffe, nicht das für Downloads.
- Limit für Rohdownloads: 10MB.
- Limit für Textvorschauen: 200KB; undurchsichtige Dateien geben `415` nur bei Vorschauanfragen zurück.
- Ausstehende VirusTotal-Scans blockieren Lesezugriffe nicht; bösartige Veröffentlichungen können dennoch an anderer Stelle zurückgehalten werden.
- Private Pakete geben `404` zurück, sofern der Aufrufer nicht zum Lesen des besitzenden Herausgebers berechtigt ist.

### `GET /api/v1/packages/{name}/download`

Lädt das ältere deterministische ZIP-Archiv für eine Paketveröffentlichung herunter.

Abfrageparameter:

- `version` (optional)
- `tag` (optional)

Hinweise:

- Verwendet standardmäßig die neueste Veröffentlichung.
- Skills leiten zu `GET /api/v1/download` weiter.
- Plugin-/Paketarchive sind ZIP-Dateien mit einem Stammverzeichnis `package/`, damit ältere OpenClaw-
  Clients weiterhin funktionieren.
- Diese Route bleibt auf ZIP beschränkt. Sie streamt keine ClawPack-Dateien vom Typ `.tgz`.
- Antworten enthalten die Header `ETag`, `Digest`, `X-ClawHub-Artifact-Type` und
  `X-ClawHub-Artifact-Sha256` für Integritätsprüfungen durch den Resolver.
- Reine Registry-Metadaten werden nicht in das heruntergeladene Archiv eingefügt.
- Ausstehende VirusTotal-Scans blockieren Downloads nicht; bösartige Veröffentlichungen geben `403` zurück.
- Private Pakete geben `404` zurück, sofern der Aufrufer nicht der Eigentümer ist.

### `GET /api/npm/{package}`

Gibt ein npm-kompatibles Packument für ClawPack-gestützte Paketversionen zurück.

Hinweise:

- Es werden nur Versionen mit hochgeladenen ClawPack-npm-Pack-Tarballs aufgeführt.
- Ältere, ausschließlich als ZIP verfügbare Versionen werden absichtlich ausgelassen.
- `dist.tarball`, `dist.integrity` und `dist.shasum` verwenden npm-kompatible
  Felder, sodass Benutzer npm bei Bedarf auf den Mirror verweisen können.
- Packuments für Pakete mit Namensbereich unterstützen sowohl `/api/npm/@scope/name` als auch den von npm
  codierten Anfragepfad `/api/npm/@scope%2Fname`.

### `GET /api/npm/{package}/-/{tarball}.tgz`

Streamt exakt die Bytes des hochgeladenen ClawPack-Tarballs für npm-Mirror-Clients.

Hinweise:

- Verwendet das Ratenlimit-Kontingent für Downloads.
- Download-Header enthalten ClawHub-SHA-256 sowie npm-Integritäts-/Shasum-Metadaten.
- Prüfungen der Moderation und des Zugriffs auf private Pakete gelten weiterhin.

### `GET /api/v1/resolve`

Wird von der CLI verwendet, um einen lokalen Fingerabdruck einer bekannten Version zuzuordnen.

Abfrageparameter:

- `slug` (erforderlich)
- `hash` (erforderlich): hexadezimale SHA-256-Zeichenfolge mit 64 Zeichen für den Bundle-Fingerabdruck

Antwort:

```json
{ "slug": "gifgrep", "match": { "version": "1.2.2" }, "latestVersion": { "version": "1.2.3" } }
```

### `GET /api/v1/download`

Lädt eine ZIP-Datei einer gehosteten Skill-Version herunter oder gibt eine Übergabe an den GitHub-Quellcode für einen
aktuellen GitHub-basierten Skill mit einem `clean`- oder `suspicious`-Scan und ohne gehostete
Version zurück.

Abfrageparameter:

- `slug` (erforderlich)
- `version` (optional): SemVer-Zeichenfolge
- `tag` (optional): Tag-Name (z. B. `latest`)

Hinweise:

- Wenn weder `version` noch `tag` angegeben ist, wird die neueste Version verwendet.
- Als vorläufig gelöscht markierte Versionen geben `410` zurück.
- Übergaben GitHub-basierter Skills verwenden weder Proxying noch Spiegelung der Bytes. Die JSON-Antwort
  enthält `sourceRef: "public-github"`, `repo`, `commit`, `path`, `contentHash`
  und `archiveUrl`; der Scan-/Aktualitätsstatus dient als Sperre und ist nicht als Metadaten
  der erfolgreichen Nutzlast enthalten.
- Downloadstatistiken zählen eindeutige Identitäten pro UTC-Tag (`userId` bei gültigem API-Token, andernfalls die IP-Adresse).

## Authentifizierungsendpunkte (Bearer-Token)

Alle Endpunkte erfordern:

```
Authorization: Bearer clh_...
```

### `GET /api/v1/whoami`

Validiert das Token und gibt den Benutzer-Handle zurück.

### `POST /api/v1/skills`

Veröffentlicht eine neue Version.

- Bevorzugt: `multipart/form-data` mit `payload`-JSON und `files[]`-Blobs.
- Ein JSON-Body mit `files` (auf Basis der Speicher-ID) wird ebenfalls akzeptiert.
- Optionales Nutzlastfeld: `ownerHandle`. Falls vorhanden, löst die API diesen
  Publisher serverseitig auf und verlangt, dass der Akteur Publisher-Zugriff besitzt.
- Optionales Nutzlastfeld: `migrateOwner`. Bei `true` zusammen mit `ownerHandle` kann
  ein bestehender Skill zu diesem Eigentümer verschoben werden, wenn der Akteur sowohl beim
  aktuellen als auch beim Ziel-Publisher Administrator/Eigentümer ist. Ohne diese ausdrückliche Zustimmung werden
  Eigentümeränderungen abgelehnt.

### `POST /api/v1/packages`

Veröffentlicht ein Code-Plugin- oder Bundle-Plugin-Release.

- Erfordert eine Authentifizierung mit Bearer-Token.
- Erfordert `multipart/form-data`.
- Zulässige Formularfelder sind `payload`, wiederholte `files`-Blobs oder eine `clawpack`-
  Tarball-Referenz. `clawpack` kann ein `.tgz`-Blob oder eine vom
  Upload-URL-Ablauf zurückgegebene Speicher-ID sein. Veröffentlichungen mit bereitgestellter Speicher-ID müssen außerdem das
  mit dieser Upload-URL zurückgegebene `clawpackUploadTicket` enthalten.
- Verwenden Sie entweder `files` oder `clawpack`, niemals beide in derselben Anfrage.
- JSON-Bodys und vom Aufrufer bereitgestellte `payload.files`- / `payload.artifact`-
  Metadaten werden abgelehnt.
- Direkte mehrteilige Veröffentlichungsanfragen sind auf 18MB begrenzt. ClawPack-Tarballs können
  den Upload-URL-Ablauf bis zur Tarball-Obergrenze von 120MB verwenden.
- Optionales Nutzlastfeld: `ownerHandle`. Falls vorhanden, dürfen nur Administratoren im Namen dieses Eigentümers veröffentlichen.

Wichtige Validierungsregeln:

- `family` muss `code-plugin` oder `bundle-plugin` sein.
- Plugin-Pakete erfordern `openclaw.plugin.json`. ClawPack-`.tgz`-Uploads müssen
  es unter `package/openclaw.plugin.json` enthalten.
- Code-Plugins erfordern `package.json`, Metadaten zum Quell-Repository, Metadaten zum Quell-Commit,
  Metadaten zum Konfigurationsschema, `openclaw.compat.pluginApi` und
  `openclaw.build.openclawVersion`.
- `openclaw.hostTargets` und `openclaw.environment` sind optionale Metadaten.
- Nur der Organisations-Publisher `openclaw` und die persönlichen Publisher aktueller Mitglieder
  der Organisation `openclaw` dürfen im Kanal `official` veröffentlichen.
- Veröffentlichungen im Namen anderer prüfen die Berechtigung für den offiziellen Kanal weiterhin anhand des Ziel-Eigentümerkontos.

### `DELETE /api/v1/skills/{slug}` / `POST /api/v1/skills/{slug}/undelete`

Einen Skill vorläufig löschen/wiederherstellen (Eigentümer, Moderator oder Administrator).

Optionaler JSON-Body:

```json
{ "reason": "Zur Moderation bis zum Abschluss der rechtlichen Prüfung zurückgehalten." }
```

Falls vorhanden, wird `reason` als Moderationsnotiz des Skills gespeichert und in das Auditprotokoll kopiert.
Vom Eigentümer veranlasste vorläufige Löschungen reservieren den Slug für 30 Tage; danach kann der Slug von
einem anderen Publisher beansprucht werden. Die Löschantwort enthält `slugReservedUntil`, wenn dieser Ablauf gilt.
Ausblendungen durch Moderatoren/Administratoren und sicherheitsbedingte Entfernungen laufen nicht auf diese Weise ab.

Löschantwort:

```json
{ "ok": true, "slugReservedUntil": 1730000000000 }
```

Statuscodes:

- `200`: erfolgreich
- `401`: nicht authentifiziert
- `403`: nicht autorisiert
- `404`: Skill/Benutzer nicht gefunden
- `500`: interner Serverfehler

### `POST /api/v1/users/publisher`

Nur für Administratoren. Stellt sicher, dass für einen Handle ein Organisations-Publisher vorhanden ist. Wenn der Handle noch auf einen
veralteten gemeinsamen Benutzer-/persönlichen Publisher verweist, migriert der Endpunkt ihn zunächst zu einem Organisations-Publisher.
Geben Sie für eine neu erstellte Organisation `memberHandle` an; der ausführende Administrator wird nicht als Mitglied hinzugefügt.
`memberRole` verwendet standardmäßig `owner`.

- Body: `{ "handle": "openclaw", "displayName": "OpenClaw", "memberHandle": "alice", "memberRole": "owner", "trusted": true }`
- Antwort: `{ "ok": true, "publisherId": "...", "handle": "openclaw", "created": true, "migrated": false, "trusted": true, "member": { "userId": "...", "handle": "alice", "role": "owner" } }`

### `POST /api/v1/publishers`

Authentifizierte Self-Service-Erstellung eines Organisations-Publishers. Erstellt einen neuen Organisations-Publisher und fügt den
Aufrufer als Eigentümer hinzu. Dieser Endpunkt migriert keine vorhandenen Benutzer-/persönlichen Handles und
markiert den Publisher nicht als vertrauenswürdig/offiziell.

- Body: `{ "handle": "opik", "displayName": "Opik" }`
- Antwort: `{ "ok": true, "publisherId": "...", "handle": "opik", "created": true, "trusted": false }`
- Gibt `409` zurück, wenn der Handle bereits von einem Publisher, Benutzer oder persönlichen Publisher verwendet wird.

### `POST /api/v1/users/reserve`

Nur für Administratoren. Reserviert Stamm-Slugs und Paketnamen für einen rechtmäßigen Eigentümer, ohne ein
Release zu veröffentlichen. Paketnamen werden zu privaten Platzhalterpaketen ohne Release-Zeilen, sodass derselbe
Eigentümer später das tatsächliche Code-Plugin- oder Bundle-Plugin-Release unter diesem Namen veröffentlichen kann.

- Body: `{ "handle": "openclaw", "slugs": ["diffs"], "packageNames": ["@openclaw/diffs"], "reason": "reserved for official OpenClaw plugin" }`
- Antwort: `{ "ok": true, "succeeded": 2, "failed": 0, "results": [{ "kind": "slug", "name": "diffs", "ok": true, "action": "reserved" }] }`

### `POST /api/v1/users/publisher-recovery`

Nur für Administratoren. Stellt einen persönlichen Publisher für einen verifizierten Ersatz-GitHub-OAuth-Principal
wieder her, ohne Convex-Auth-Kontozeilen zu bearbeiten. Die Anfrage muss beide unveränderlichen GitHub-
Provider-Konto-IDs nennen; veränderliche Handles werden nur als Schutzmechanismus für den Bediener verwendet.

Der Endpunkt verwendet standardmäßig einen Probelauf. Die Anwendung der Wiederherstellung erfordert `dryRun: false` und
`confirmIdentityVerified: true`, nachdem Mitarbeiter die Kontinuität zwischen beiden
GitHub-Principals unabhängig verifiziert haben. Die Wiederherstellung schlägt sicher fehl, wenn der aktuelle persönliche
Publisher des Zielbenutzers Skills, Pakete oder GitHub-Skill-Quellen besitzt.
Die Wiederherstellung migriert außerdem veraltete `ownerUserId`-Felder für die Skills des wiederhergestellten Publishers,
Skill-Slug-Aliasse, Pakete, Warnungen des Paket-Inspektors und abgeleitete Such-Digest-Zeilen, sodass
Pfade mit direktem Eigentümer mit der neuen Publisher-Autorität übereinstimmen. Eine aktive Reservierung
eines geschützten Handles für den wiederhergestellten Handle wird ebenfalls dem Ersatzbenutzer zugewiesen, damit eine spätere
Profilsynchronisierung die konkurrierende Autorität des früheren Benutzers nicht wiederherstellen kann. Jede Primärtabelle ist auf
100 Zeilen pro Anwendungstransaktion begrenzt; größere Wiederherstellungen müssen zunächst eine fortsetzbare Eigentümermigration verwenden.
GitHub-Skill-Quellen sind Publisher-bezogen und werden als geprüft gemeldet, statt neu geschrieben zu werden.

- Body: `{ "handle": "gingiris", "nextUserHandle": "gingiris-1031", "previousGitHubProviderAccountId": "123", "nextGitHubProviderAccountId": "456", "reason": "Verified account continuity for issue #2555", "confirmIdentityVerified": true, "dryRun": false }`
- Antwort: `{ "ok": true, "dryRun": false, "recovered": true, "publisherId": "...", "handle": "gingiris", "previousUser": { "userId": "...", "handle": "gingiris", "nextHandle": "gingiris-recovered", "githubProviderAccountId": "123", "authAccountCount": 1 }, "nextUser": { "userId": "...", "handle": "gingiris-1031", "nextHandle": "gingiris", "githubProviderAccountId": "456", "authAccountCount": 1 }, "retiredPersonalPublisher": null, "resourceOwnerMigration": { "limitPerTable": 100, "skills": 1, "skillSlugAliases": 1, "packages": 0, "packageInspectorWarnings": 0, "githubSourcesChecked": 1, "handleReservations": 1 }, "identityVerified": true, "reason": "Verified account continuity for issue #2555" }`

### Endpunkte zur Verwaltung von Eigentümer-Slugs

- `POST /api/v1/skills/{slug}/rename`
  - Body: `{ "newSlug": "new-canonical-slug" }`
  - Antwort: `{ "ok": true, "slug": "new-canonical-slug", "previousSlug": "old-slug" }`
- `POST /api/v1/skills/{slug}/merge`
  - Body: `{ "targetSlug": "canonical-target-slug" }`
  - Antwort: `{ "ok": true, "sourceSlug": "old-slug", "targetSlug": "canonical-target-slug" }`

Hinweise:

- Beide Endpunkte erfordern eine Authentifizierung mit API-Token und funktionieren nur für den Eigentümer des Skills.
- `rename` bewahrt den vorherigen Slug als Weiterleitungsalias auf.
- `merge` blendet den Quelleintrag aus und leitet den Quell-Slug zum Zieleintrag weiter.

### Endpunkte zur Eigentumsübertragung

- `POST /api/v1/skills/{slug}/transfer`
  - Body: `{ "toUserHandle": "target_handle", "message": "optional" }`
  - Antwort: `{ "ok": true, "transferId": "skillOwnershipTransfers:...", "toUserHandle": "target_handle", "expiresAt": 1730000000000 }`
- `POST /api/v1/skills/{slug}/transfer/accept`
- `POST /api/v1/skills/{slug}/transfer/reject`
- `POST /api/v1/skills/{slug}/transfer/cancel`
  - Antwort (Annehmen/Ablehnen/Abbrechen): `{ "ok": true, "skillSlug": "demo-skill?" }`
- `GET /api/v1/transfers/incoming`
- `GET /api/v1/transfers/outgoing`
  - Antwortstruktur: `{ "transfers": [{ "_id": "...", "skill": { "slug": "demo", "displayName": "Demo" }, "fromUser"|"toUser": { "handle": "..." }, "message": "...", "requestedAt": 0, "expiresAt": 0 }] }`

### `POST /api/v1/users/ban`

Einen Benutzer sperren und dessen Skills unwiderruflich löschen (nur Moderatoren/Administratoren).

Body:

```json
{ "handle": "user_handle", "reason": "optionaler Sperrgrund" }
```

oder

```json
{ "userId": "users_...", "reason": "optionaler Sperrgrund" }
```

Antwort:

```json
{ "ok": true, "alreadyBanned": false, "deletedSkills": 3 }
```

### `POST /api/v1/users/unban`

Die Sperre eines Benutzers aufheben und berechtigte Skills wiederherstellen (nur Administratoren).

Body:

```json
{ "handle": "user_handle", "reason": "optionaler Grund für die Aufhebung der Sperre" }
```

oder

```json
{ "userId": "users_...", "reason": "optionaler Grund für die Aufhebung der Sperre" }
```

Antwort:

```json
{ "ok": true, "alreadyUnbanned": false, "restoredSkills": 3 }
```

### `POST /api/v1/users/reclassify-ban`

Den gespeicherten Grund einer bestehenden Sperre ändern, ohne die Sperre aufzuheben oder
Inhalte wiederherzustellen (nur Administratoren). Verwendet standardmäßig einen Probelauf, sofern `dryRun` nicht `false` ist.

Body:

```json
{ "handle": "user_handle", "reason": "Spam durch massenhafte Veröffentlichungen", "dryRun": true }
```

oder

```json
{ "userId": "users_...", "reason": "Spam durch massenhafte Veröffentlichungen", "dryRun": false }
```

Antwort:

```json
{
  "ok": true,
  "dryRun": false,
  "userId": "users_...",
  "handle": "user_handle",
  "previousReason": "automatische Sperre wegen Schadsoftware",
  "nextReason": "Spam durch massenhafte Veröffentlichungen",
  "changed": true
}
```

### `POST /api/v1/users/role`

Eine Benutzerrolle ändern (nur Administratoren).

Body:

```json
{ "handle": "user_handle", "role": "moderator" }
```

oder

```json
{ "userId": "users_...", "role": "admin" }
```

Antwort:

```json
{ "ok": true, "role": "moderator" }
```

### `GET /api/v1/users`

Benutzer auflisten oder suchen (nur Administratoren).

Abfrageparameter:

- `q` (optional): Suchabfrage
- `query` (optional): Alias für `q`
- `limit` (optional): maximale Anzahl von Ergebnissen (Standardwert 20, maximal 200)

Antwort:

```json
{
  "items": [
    {
      "userId": "users_...",
      "handle": "user_handle",
      "displayName": "Benutzer",
      "name": "Benutzer",
      "role": "moderator"
    }
  ],
  "total": 1
}
```

### `POST /api/v1/stars/{slug}` / `DELETE /api/v1/stars/{slug}`

Ein Lesezeichen hinzufügen/entfernen. Die veraltete Route `stars` und die Namen der Antwortfelder bleiben
aus Kompatibilitätsgründen erhalten. Beide Endpunkte sind idempotent.

Antworten:

```json
{ "ok": true, "starred": true, "alreadyStarred": false }
```

```json
{ "ok": true, "unstarred": true, "alreadyUnstarred": false }
```

## Veraltete CLI-Endpunkte

Für ältere CLI-Versionen weiterhin unterstützt:

- `GET /api/cli/whoami`
- `POST /api/cli/upload-url`
- `POST /api/cli/publish`
- `POST /api/cli/telemetry/install`
- `POST /api/cli/skill/delete`
- `POST /api/cli/skill/undelete`

Siehe `DEPRECATIONS.md` für den Entfernungsplan.

`POST /api/cli/upload-url` gibt `uploadUrl` und `uploadTicket` zurück. Paket-
veröffentlichungen, die einen ClawPack-Tarball bereitstellen, müssen die resultierende Speicher-ID als
`clawpack` und das zurückgegebene Ticket als `clawpackUploadTicket` senden.

## Registry-Erkennung (`/.well-known/clawhub.json`)

Die CLI kann Registry-/Authentifizierungseinstellungen von der Website erkennen:

- `/.well-known/clawhub.json` (JSON, bevorzugt)
- `/.well-known/clawdhub.json` (veraltet)

Schema:

```json
{ "apiBase": "https://clawhub.ai", "authBase": "https://clawhub.ai", "minCliVersion": "0.0.5" }
```

Wenn Sie selbst hosten, stellen Sie diese Datei bereit (oder legen Sie `CLAWHUB_REGISTRY` ausdrücklich fest; veraltet: `CLAWDHUB_REGISTRY`).
