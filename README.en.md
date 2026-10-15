# Saving Tokens with AI Agents

🇩🇪 [Deutsche Version](README.md)

Article and graphics on how to save tokens and cost when working with AI agents (Claude Code, Codex, Gemini CLI and others) – using vendor-agnostic methods.

![21 agnostic methods to save tokens](bilder/infographic-token-efficiency.png)

## Contents

- [methods.md](methods.md) – Article: 21 agnostic methods, sorted by layer, with savings, notes and prioritization
- [bilder/](bilder/) – Infographic: all methods across the agent cycle
  - [PNG](bilder/infographic-token-efficiency.png) · [PNG @2x](bilder/infographic-token-efficiency@2x.png) · [PDF](bilder/infographic-token-efficiency.pdf)
- [skills/](skills/) – `token-efficiency` skill for Claude (English and German)

## Installing the skill

The skill implements the methods an agent can control itself and has three modes: **Work**, **Setup** (anchor a rule block in `CLAUDE.md`/`AGENTS.md`) and **Audit** (check a project for token waste).

| Version | Folder | ZIP |
|---|---|---|
| English | [skills/en/token-efficiency](skills/en/token-efficiency/SKILL.md) | [token-efficiency-en.zip](skills/token-efficiency-en.zip) |
| Deutsch | [skills/de/token-efficiency](skills/de/token-efficiency/SKILL.md) | [token-efficiency-de.zip](skills/token-efficiency-de.zip) |

- **claude.ai:** Upload the ZIP under Skills in the settings.
- **Claude Code:** Copy the folder to `~/.claude/skills/token-efficiency/` (or per project to `.claude/skills/`), then run `/token-efficiency Audit` or `/token-efficiency Setup`.

The skill is most effective in agentic environments with a shell, files and long sessions. Infrastructure methods such as caching, batch APIs or model routing can only be recommended, not implemented by the skill itself.
