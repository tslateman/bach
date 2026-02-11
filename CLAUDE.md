# Bach

Two-tier agent orchestration plugin for Claude Code. Markdown only — no build system.

## Lint

```bash
markdownlint commands/**/*.md skills/**/*.md
```

## Architecture

A Manager decomposes tasks and delegates to specialist Workers (researcher, coder, reviewer). Workers are stateless prompt templates.

- `commands/orchestrate.md` — `/orchestrate` slash command
- `skills/orchestrating-work/` — Core two-tier pattern + worker templates
- `skills/creating-workers/` — Guide for adding custom workers

## Contracts

- [TASK_CONTRACT.md](TASK_CONTRACT.md) — Defines the Flow → Bach interface (task/result envelopes, worker vocabulary)
- Flow sends task envelopes; Bach returns result envelopes with `complete`, `incapable`, `blocked`, or `partial` status
