---
x-i18n:
    generated_at: "2026-07-26T17:38:07Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: a8712b1aeb2e605055c22cf308049e5e74fdf33061870026be20bd55cb0c3d1d
    source_path: AGENTS.md
    workflow: 16
---

# Dokumentationsleitfaden

Dieses Verzeichnis ist für die Erstellung der Dokumentation, die Mintlify-Linkregeln und die i18n-Richtlinie der Dokumentation zuständig.

## Mintlify-Regeln

- Die Dokumentation wird auf Mintlify gehostet (`https://docs.openclaw.ai`).
- Interne Dokumentationslinks in `docs/**/*.md` müssen relativ zum Stammverzeichnis bleiben und dürfen weder das Suffix `.md` noch `.mdx` enthalten (Beispiel: `[Config](/gateway/configuration)`).
- Querverweise auf Abschnitte sollten Anker in stammrelativen Pfaden verwenden (Beispiel: `[Hooks](/gateway/configuration-reference#hooks)`).
- Dokumentationsüberschriften sollten Gedankenstriche und Apostrophe vermeiden, da die Ankererzeugung von Mintlify dabei fehleranfällig ist.
- README-Dateien und andere von GitHub gerenderte Dokumentationen sollten absolute Dokumentations-URLs beibehalten, damit Links außerhalb von Mintlify funktionieren.
- Dokumentationsinhalte müssen generisch bleiben: keine persönlichen Gerätenamen, Hostnamen oder lokalen Pfade; verwenden Sie Platzhalter wie `user@gateway-host`.

## Regeln für Dokumentationsinhalte

- Ordnen Sie Dienste/Provider in Dokumentation, UI-Texten und Auswahllisten alphabetisch, sofern der Abschnitt nicht ausdrücklich die Laufzeitreihenfolge oder die Reihenfolge der automatischen Erkennung beschreibt.
- Halten Sie die Benennung gebündelter Plugins mit den repositoryweiten Regeln zur Plugin-Terminologie in der `AGENTS.md` im Stammverzeichnis konsistent.
- Generierte Dokumentation, niemals manuell bearbeiten: `docs/plugins/reference/**`, `docs/plugins/reference.md` und `docs/plugins/plugin-inventory.md` stammen aus `pnpm plugins:inventory:gen`; `docs/docs_map.md` aus `pnpm docs:map:gen`; `docs/maturity/**` aus `pnpm maturity:render`.

## Interne Dokumentation

- Langfristig verwendete private Betriebsdokumentation gehört in `~/Projects/manager/docs/`.
- Repositorylokale interne Entwurfs-/Spiegeldokumentation kann sich unter dem ignorierten Pfad `docs/internal/` befinden.
- Fügen Sie Seiten aus `docs/internal/**` niemals zur Navigation in `docs/docs.json` hinzu und verlinken Sie sie nicht aus der öffentlichen Dokumentation.
- `scripts/docs-sync-publish.mjs` schließt `docs/internal/**` aus und entfernt es aus dem öffentlichen Veröffentlichungsrepository `openclaw/docs`, falls eine Seite später zwangsweise hinzugefügt wird.
- Interne Dokumentation darf Repositorypfade, Namen privater Apps, Namen von 1Password-Elementen und Runbooks erwähnen, aber niemals geheime Werte enthalten.

## Bearbeitung der Reifegrad-Scorecard

`taxonomy.yaml` und `qa/maturity-scores.yaml` sind die Quelleingaben; generierte Reifegraddokumentation unter `docs/maturity/` ist eine Projektion und sollte für Bewertungen, LTS, Taxonomie, QA-Profile oder Evidenztabellen nicht manuell bearbeitet werden.
`scripts/qa/render-maturity-docs.ts` ist für die Generierung zuständig; verwenden Sie `pnpm maturity:render`, um eingecheckte Dokumentation zu aktualisieren, und `pnpm maturity:check`, um sie zu überprüfen.
`.github/workflows/maturity-scorecard.yml` rendert Artefaktvorschauen und kann Pull Requests für generierte Dokumentation öffnen; `.github/workflows/openclaw-release-checks.yml` startet diesen Vorgang für die Release-QA.
Bewahren Sie deterministische `qa-evidence.json.scorecard`-Daten in GitHub-Actions-Artefakten auf, sofern ein Maintainer nicht ausdrücklich eine bereinigte, eingecheckte Projektion verlangt.
Manuelle Überschreibungen müssen den Quellzustand in einem Pull Request ändern und den Grund sowie öffentliche oder redigierte Nachweise erläutern.

## Dokumentations-i18n

- Fremdsprachige Dokumentation wird in diesem Repository nicht gepflegt. Die generierte Veröffentlichungsausgabe befindet sich im separaten Repository `openclaw/docs` (lokal häufig als `../openclaw-docs` geklont).
- Fügen Sie hier keine lokalisierten Dokumentationen unter `docs/<locale>/**` hinzu und bearbeiten Sie dort keine.
- Betrachten Sie die englische Dokumentation in diesem Repository sowie die Glossardateien als maßgebliche Quelle.
- Pipeline: Aktualisieren Sie hier die englische Dokumentation, aktualisieren Sie `docs/.i18n/glossary.<locale>.json` nach Bedarf und lassen Sie anschließend die Synchronisierung des Veröffentlichungsrepositorys sowie `scripts/docs-i18n` in `openclaw/docs` ausführen.
- Fügen Sie vor der erneuten Ausführung von `scripts/docs-i18n` Glossareinträge für alle neuen technischen Begriffe, Seitentitel oder kurzen Navigationsbeschriftungen hinzu, die auf Englisch bleiben oder eine feste Übersetzung verwenden müssen.
- `pnpm docs:check-i18n-glossary` ist die Schutzprüfung für geänderte englische Dokumentationstitel und kurze interne Dokumentationsbeschriftungen.
- Der Übersetzungsspeicher befindet sich in den generierten `docs/.i18n/*.tm.jsonl`-Dateien im Veröffentlichungsrepository.
- Siehe `docs/.i18n/README.md`.
