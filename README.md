# Token sparen mit KI-Agenten

Artikel und Grafiken dazu, wie man beim Arbeiten mit KI-Agenten (Claude Code, Codex, Gemini CLI u. a.) Tokens und Kosten spart – mit herstellerunabhängigen Methoden.

![21 agnostische Methoden, um Tokens zu sparen](bilder/infografik-token-effizienz.png)

## Inhalt

- [methoden.md](methoden.md) – Artikel: 21 agnostische Methoden, nach Layern sortiert, mit Ersparnis, Anmerkungen und Priorisierung
- [bilder/](bilder/) – Infografik: alle Methoden entlang des Agenten-Zyklus
  - [PNG](bilder/infografik-token-effizienz.png) · [PNG @2x](bilder/infografik-token-effizienz@2x.png) · [PDF](bilder/infografik-token-effizienz.pdf)
- [skills/](skills/) – Skill `token-efficiency` für Claude (Deutsch und Englisch)

## Skill installieren

Der Skill setzt die Methoden um, die ein Agent selbst steuern kann, und hat drei Modi: **Arbeiten**, **Setup** (Regelblock in `CLAUDE.md`/`AGENTS.md` verankern) und **Audit** (Projekt auf Token-Verschwendung prüfen).

| Version | Ordner | ZIP |
|---|---|---|
| Deutsch | [skills/de/token-efficiency](skills/de/token-efficiency/SKILL.md) | [token-efficiency-de.zip](skills/token-efficiency-de.zip) |
| English | [skills/en/token-efficiency](skills/en/token-efficiency/SKILL.md) | [token-efficiency-en.zip](skills/token-efficiency-en.zip) |

- **claude.ai:** ZIP in den Einstellungen bei den Skills hochladen.
- **Claude Code:** Ordner nach `~/.claude/skills/token-efficiency/` kopieren (oder projektbezogen nach `.claude/skills/`), dann mit `/token-efficiency Audit` oder `/token-efficiency Setup` aufrufen.

Der Skill wirkt vor allem in agentischen Umgebungen mit Shell, Dateien und langen Sessions. Infrastruktur-Methoden wie Caching, Batch APIs oder Model Routing kann er nur empfehlen, nicht selbst umsetzen.
