# Bach

Two-tier agent orchestration plugin for Claude Code. Markdown only — no build system.

## Lint

```bash
markdownlint commands/**/*.md skills/**/*.md
```

## Architecture

A Manager decomposes tasks and delegates to specialist Workers (researcher, coder, reviewer, architect). Workers are stateless prompt templates.

- `commands/orchestrate.md` — `/orchestrate` slash command (single-pass)
- `commands/mission.md` — `/mission` slash command (multi-phase lifecycle)
- `skills/orchestrating-work/` — Core two-tier pattern + worker templates
- `skills/lifecycle/` — Multi-phase orchestration with review gates
- `skills/creating-workers/` — Guide for adding custom workers
- `agents/architect.md` — CC agent: design authority for planning phases

## Contracts

- [TASK_CONTRACT.md](TASK_CONTRACT.md) — Defines the Flow → Bach interface (task/result envelopes, worker vocabulary)
- Flow sends task envelopes; Bach returns result envelopes with `complete`, `incapable`, `blocked`, or `partial` status
