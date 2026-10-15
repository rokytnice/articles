---
name: "token-efficiency"
description: "Senkt den Token-Verbrauch in Agenten-Sessions: sparsame Arbeitsregeln anwenden, sie in CLAUDE.md/AGENTS.md verankern oder ein Projekt auf Token-Verschwendung prüfen."
---

# Token-Effizienz für Agenten

Ziel: weniger Tokens, günstigere Tokens, weniger Aufrufe, ohne Qualitätsverlust. Der größte Hebel liegt fast immer im Input-Kontext, weil jeder Turn den gesamten Verlauf erneut sendet. Output-Kürzung wirkt, aber kleiner.

Der Skill hat drei Modi. Wähle anhand der Anfrage:
- **Arbeiten** (Standard): Die Aufgabe des Nutzers erledigen und dabei die Arbeitsregeln befolgen.
- **Setup**: Nutzer will die Regeln dauerhaft im Projekt verankern ("richte das ein", "in CLAUDE.md").
- **Audit**: Nutzer will wissen, wo ein Projekt oder Setup Tokens verschwendet ("prüfe", "analysiere", "wo verbrauche ich Tokens").

## Modus 1: Arbeitsregeln

### Phase 0: Planen
- Bei Aufgaben, die mehr als drei Dateien berühren oder einen unklaren Scope haben: zuerst einen kurzen Plan (max. 10 Punkte) vorlegen und bestätigen lassen. Fehlschleifen sind der teuerste Verbrauch.
- Bei trivialen Aufgaben keinen Plan, direkt umsetzen.

### Phase 2: Kontext
- Dateien über ca. 300 Zeilen nicht komplett lesen. Erst per grep oder Symbolsuche die Stelle finden, dann gezielt den Zeilenbereich lesen.
- Eine bereits gelesene, unveränderte Datei nicht erneut lesen.
- Für Repo-Überblick zuerst Verzeichnisstruktur und Symbolnamen ansehen, nicht ganze Quelldateien.
- Tabellarische Daten im Kontext als CSV oder YAML halten, nicht als formatiertes JSON.
- Dokumentation erst laden, wenn sie für den aktuellen Schritt gebraucht wird.

### Phase 4: Tool-Ausführung
- Shell-Ausgaben immer begrenzen oder filtern, zum Beispiel:
  - `pytest -q --tb=short`, `mvn -q`, `./gradlew --quiet`, `npm test --silent`
  - `git --no-pager log --oneline -20`, `git status --short`, `git diff --stat` vor vollem Diff
  - `docker logs --tail 100`, `kubectl logs --tail=100`
  - `... 2>&1 | grep -E "FAIL|ERROR|Exception" | head -50`, `... | tail -n 50`
- Falls `rtk` installiert ist (`command -v rtk`), Shell-Befehle darüber ausführen.
- Breite Suche, Log-Analyse oder Recherche an einen Sub-Agenten delegieren, sofern verfügbar. Auftrag: nur das Ergebnis zurückgeben, max. ca. 20 Zeilen, mit Dateipfaden und Zeilennummern.
- Formatierung und Linting nicht von Hand korrigieren, wenn ein Formatter oder Linter im Projekt existiert: ihn ausführen.

### Phase 3: Edits und Antworten
- Bestehende Dateien mit gezielten Edits oder Diffs ändern, nie komplett neu schreiben, außer bei neuen Dateien oder wenn sich mehr als die Hälfte ändert.
- Antworten knapp halten: keine Einleitung, keine Wiederholung des Auftrags, keine Zusammenfassung von Dingen, die der Nutzer selbst sieht (Diffs, Dateiinhalte).
- Nicht kürzen an der falschen Stelle: Begründungen für Architekturentscheidungen, Risiken und offene Fragen bleiben drin.
- Bei strukturierten Ausgaben ein Feld für die Begründung vor das Ergebnisfeld stellen.

### Phase 5: Verlauf
- In langen Sessions Entscheidungen, offene Punkte und geänderte Dateien in `NOTES.md` (oder einer vom Projekt vorgegebenen Datei) festhalten, damit eine Kontext-Komprimierung oder ein Neustart verlustfrei bleibt.
- Nach einem Neustart zuerst diese Datei lesen statt Dateien neu zu erkunden.

## Modus 2: Setup

Ein Skill wird nur geladen, wenn er ausgelöst wird. Regeln, die immer gelten sollen, gehören in die Projekt-Instruktionen.

1. Prüfen, ob `CLAUDE.md` oder `AGENTS.md` im Projektwurzelverzeichnis existiert. Gibt es beide, die vom Projekt genutzte wählen oder nachfragen.
2. Prüfen, ob dort bereits ein Abschnitt `## Token-Effizienz` steht. Wenn ja, nur aktualisieren, nichts doppelt einfügen.
3. Den Regelblock unten dem Nutzer zeigen, an Projektspezifika anpassen (echte Test- und Build-Befehle aus Makefile, package.json, pom.xml usw. einsetzen) und nach Bestätigung anhängen.
4. Den Block kurz halten (unter ca. 300 Tokens): er wird bei jedem Aufruf mitgesendet.

Regelblock:

```markdown
## Token-Effizienz
- Große Dateien: erst grep/Symbolsuche, dann nur den relevanten Zeilenbereich lesen. Nichts doppelt lesen.
- Shell-Ausgaben immer filtern oder begrenzen (quiet-Flags, `| tail -n 50`, `| grep -E "FAIL|ERROR"`).
- Tests: <PROJEKT-TESTBEFEHL im quiet-Modus>
- Breite Suche und Log-Analyse an Sub-Agenten delegieren; nur Ergebnis zurück.
- Bestehende Dateien per gezieltem Edit ändern, nie komplett neu schreiben.
- Formatter/Linter ausführen statt Formatierung von Hand zu korrigieren.
- Bei Aufgaben über mehr als drei Dateien: erst kurzer Plan, dann Umsetzung.
- Antworten knapp: keine Einleitung, keine Wiederholung, keine Zusammenfassung sichtbarer Diffs.
- Lange Sessions: Entscheidungen und offene Punkte in NOTES.md festhalten.
```

## Modus 3: Audit

Das Projekt und das Setup prüfen, nichts ändern. Danach eine priorisierte Liste vorlegen.

Prüfpunkte:
1. **Instruktionsdateien**: Größe von `CLAUDE.md`, `AGENTS.md` und importierten Dateien (Faustregel: Zeichen / 4 = Tokens). Über ca. 2.000 Tokens: Selten genutztes in Skills oder separate Dateien auslagern, die bei Bedarf geladen werden.
2. **Cache-Stabilität**: Stehen dynamische Inhalte (Datum, Zeitstempel, Git-Status, wechselnde Listen) in Instruktionsdateien oder System-Prompts? Sie brechen das Prefix-Caching.
3. **MCP-Server und Tools**: Konfigurierte Server zählen (`.mcp.json`, Einstellungen). Server, die im Projekt nicht gebraucht werden, zur Deaktivierung vorschlagen.
4. **Ausschlüsse**: Sind große oder generierte Inhalte (node_modules, target, build, dist, Logs, Lockfiles, generierter Code) vom Lesen ausgeschlossen, etwa über `.gitignore` oder Read-Deny-Regeln in den Agent-Einstellungen?
5. **Hooks**: Laufen Formatter und Linter automatisch nach Edits? Wenn nicht, einen Hook vorschlagen.
6. **Befehle**: Gibt es dokumentierte quiet-Varianten für Test, Build und Lint? Wenn nicht, konkrete Befehle vorschlagen.
7. **Große Einzeldateien**: Dateien über ca. 1.000 Zeilen auflisten, die der Agent häufig anfassen dürfte.
8. **Infrastruktur** (außerhalb des Repos, nur als Empfehlung): Prompt Caching bei eigener API-Nutzung, Batch API für nicht zeitkritische Jobs, Model Routing für Routineaufgaben, Reasoning-Budget pro Aufgabentyp.

Ausgabe als Tabelle, sortiert nach Wirkung im Verhältnis zum Aufwand:

| Befund | Methode | Wirkung | Aufwand | Konkrete Änderung |
|---|---|---|---|---|

Wirkung als hoch, mittel oder gering, Aufwand als klein, mittel oder groß. Änderungen erst nach Bestätigung umsetzen.

## Grenzen

Folgende Methoden kann der Skill nicht selbst umsetzen, nur empfehlen: Prompt-Caching-Breakpoints, Batch APIs, Model Routing, Semantic Caching, Reasoning-Budget-Einstellungen, Installation von RTK. Viele Harnesses wie Claude Code erledigen Caching, Kontext-Komprimierung und das Nachladen von Tools bereits selbst.

## Referenz: die 21 Methoden nach Phase

| Phase | Nr. | Methode | Wirkt auf |
|---|---|---|---|
| 0 Planen | 20 | Spec-first / Plan-then-Execute | Iterationen |
| 1 Aufruf | 11 | Model Routing | Preis |
| 1 Aufruf | 17 | Reasoning-Budget | Output |
| 1 Aufruf | 18 | Batch APIs | Preis |
| 1 Aufruf | 16 | Semantic Caching | Aufrufe |
| 2 Kontext | 2 | Prefix Alignment & Caching | Preis |
| 2 Kontext | 13 | Dynamic Tool Loading | Input |
| 2 Kontext | 6 | JIT Retrieval | Input |
| 2 Kontext | 5 | AST Repo Mapping | Input |
| 2 Kontext | 15 | Token-effiziente Formate | Input |
| 2 Kontext | 14 | Prompt Compression | Input |
| 3 Antwort | 3 | Caveman Pattern | Output |
| 3 Antwort | 7 | Strict Schemas | Output |
| 3 Antwort | 12 | Diff-basierte Edits | Output |
| 4 Tools | 8 | Sub-Agent-Isolation | Input |
| 4 Tools | 1 | RTK | Input |
| 4 Tools | 4 | File Piping & Filtering | Input |
| 4 Tools | 21 | Gezieltes Lesen & Dedup | Input |
| 4 Tools | 19 | Deterministische Hooks | Iterationen |
| 5 Verlauf | 9 | Observation Masking | Input |
| 5 Verlauf | 10 | Context Compaction | Input |