---
name: token-efficiency
description: Reduces token usage in agent sessions: apply lean working rules, anchor them in CLAUDE.md/AGENTS.md, or audit a project for token waste.
---

# Token Efficiency for Agents

Goal: fewer tokens, cheaper tokens, fewer calls, without losing quality. The biggest lever is almost always the input context, because every turn resends the entire history. Shortening output helps, but less.

The skill has three modes. Choose based on the request:
- **Work** (default): Complete the user's task while following the working rules.
- **Setup**: The user wants to anchor the rules permanently in the project ("set this up", "put it in CLAUDE.md").
- **Audit**: The user wants to know where a project or setup wastes tokens ("check", "analyze", "where am I using tokens").

## Mode 1: Working rules

### Phase 0: Plan
- For tasks that touch more than three files or have an unclear scope: first present a short plan (max. 10 points) and get it confirmed. Failure loops are the most expensive consumption.
- For trivial tasks, skip the plan and implement directly.

### Phase 2: Context
- Do not read files longer than about 300 lines in full. First locate the spot via grep or symbol search, then read only that line range.
- Do not reread a file that was already read and has not changed.
- For a repo overview, look at the directory structure and symbol names first, not entire source files.
- Keep tabular data in context as CSV or YAML, not as pretty-printed JSON.
- Load documentation only when the current step needs it.

### Phase 4: Tool execution
- Always limit or filter shell output, for example:
  - `pytest -q --tb=short`, `mvn -q`, `./gradlew --quiet`, `npm test --silent`
  - `git --no-pager log --oneline -20`, `git status --short`, `git diff --stat` before the full diff
  - `docker logs --tail 100`, `kubectl logs --tail=100`
  - `... 2>&1 | grep -E "FAIL|ERROR|Exception" | head -50`, `... | tail -n 50`
- If `rtk` is installed (`command -v rtk`), run shell commands through it.
- Delegate broad searches, log analysis or research to a sub-agent when available. Instruction: return only the result, max. about 20 lines, with file paths and line numbers.
- Do not fix formatting or lint issues by hand if the project has a formatter or linter: run it.

### Phase 3: Edits and responses
- Change existing files with targeted edits or diffs; never rewrite them in full, except for new files or when more than half the file changes.
- Keep responses short: no preamble, no restating the task, no summary of things the user can see (diffs, file contents).
- Don't cut in the wrong place: reasoning behind architecture decisions, risks and open questions stay in.
- For structured output, put a reasoning field before the result field.

### Phase 5: History
- In long sessions, record decisions, open items and changed files in `NOTES.md` (or a file the project specifies), so that context compaction or a restart loses nothing.
- After a restart, read this file first instead of re-exploring files.

## Mode 2: Setup

A skill only loads when it is triggered. Rules that should always apply belong in the project instructions.

1. Check whether `CLAUDE.md` or `AGENTS.md` exists in the project root. If both exist, pick the one the project uses or ask.
2. Check whether a `## Token Efficiency` section already exists there. If so, update it only; never insert it twice.
3. Show the user the rule block below, adapt it to the project (insert the real test and build commands from Makefile, package.json, pom.xml etc.) and append it after confirmation.
4. Keep the block short (under about 300 tokens): it is sent with every call.

Rule block:

```markdown
## Token Efficiency
- Large files: grep/symbol search first, then read only the relevant line range. Never read twice.
- Always filter or limit shell output (quiet flags, `| tail -n 50`, `| grep -E "FAIL|ERROR"`).
- Tests: <PROJECT TEST COMMAND in quiet mode>
- Delegate broad search and log analysis to sub-agents; return only the result.
- Change existing files with targeted edits, never rewrite them in full.
- Run the formatter/linter instead of fixing formatting by hand.
- Tasks touching more than three files: short plan first, then implementation.
- Short responses: no preamble, no repetition, no summary of visible diffs.
- Long sessions: record decisions and open items in NOTES.md.
```

## Mode 3: Audit

Inspect the project and setup; change nothing. Then present a prioritized list.

Checkpoints:
1. **Instruction files**: Size of `CLAUDE.md`, `AGENTS.md` and imported files (rule of thumb: characters / 4 = tokens). Above about 2,000 tokens: move rarely used content into skills or separate files loaded on demand.
2. **Cache stability**: Do instruction files or system prompts contain dynamic content (dates, timestamps, git status, changing lists)? It breaks prefix caching.
3. **MCP servers and tools**: Count configured servers (`.mcp.json`, settings). Suggest disabling servers the project doesn't need.
4. **Exclusions**: Are large or generated contents (node_modules, target, build, dist, logs, lockfiles, generated code) excluded from reading, e.g. via `.gitignore` or read-deny rules in the agent settings?
5. **Hooks**: Do formatter and linter run automatically after edits? If not, suggest a hook.
6. **Commands**: Are quiet variants for test, build and lint documented? If not, suggest concrete commands.
7. **Large individual files**: List files over about 1,000 lines that the agent is likely to touch often.
8. **Infrastructure** (outside the repo, recommendation only): prompt caching for your own API use, batch API for non-time-critical jobs, model routing for routine tasks, reasoning budget per task type.

Output as a table, sorted by impact relative to effort:

| Finding | Method | Impact | Effort | Concrete change |
|---|---|---|---|---|

Impact as high, medium or low; effort as small, medium or large. Apply changes only after confirmation.

## Limits

The skill cannot implement the following methods itself, only recommend them: prompt caching breakpoints, batch APIs, model routing, semantic caching, reasoning budget settings, installing RTK. Many harnesses such as Claude Code already handle caching, context compaction and on-demand tool loading themselves.

## Reference: the 21 methods by phase

| Phase | No. | Method | Affects |
|---|---|---|---|
| 0 Plan | 20 | Spec-first / Plan-then-Execute | Iterations |
| 1 Call | 11 | Model Routing | Price |
| 1 Call | 17 | Reasoning Budget | Output |
| 1 Call | 18 | Batch APIs | Price |
| 1 Call | 16 | Semantic Caching | Calls |
| 2 Context | 2 | Prefix Alignment & Caching | Price |
| 2 Context | 13 | Dynamic Tool Loading | Input |
| 2 Context | 6 | JIT Retrieval | Input |
| 2 Context | 5 | AST Repo Mapping | Input |
| 2 Context | 15 | Token-efficient Formats | Input |
| 2 Context | 14 | Prompt Compression | Input |
| 3 Response | 3 | Caveman Pattern | Output |
| 3 Response | 7 | Strict Schemas | Output |
| 3 Response | 12 | Diff-based Edits | Output |
| 4 Tools | 8 | Sub-Agent Isolation | Input |
| 4 Tools | 1 | RTK | Input |
| 4 Tools | 4 | File Piping & Filtering | Input |
| 4 Tools | 21 | Targeted Reading & Dedup | Input |
| 4 Tools | 19 | Deterministic Hooks | Iterations |
| 5 History | 9 | Observation Masking | Input |
| 5 History | 10 | Context Compaction | Input |
