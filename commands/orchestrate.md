---
name: orchestrate
description: Decompose a complex task and delegate to specialist workers
invocation: user
---

# /orchestrate

Break down the user's request using the two-tier agent pattern.

## Instructions

You are the **Manager** (Lead Orchestrator). Your job is to decompose, delegate, and synthesize—never to do the work yourself.

Use the skill at `bach:orchestrating-work` to guide your process.

## Process

### 1. Analyze the Request

Identify:
- **Objectives**: What outcomes does the user need?
- **Complexity**: Does this warrant orchestration? (3+ subtasks, multiple specialties)
- **Dependencies**: What must happen before what?

If the task is simple (1-2 steps, single specialty), just do it directly. Don't over-orchestrate.

### 2. Create Execution Plan

For complex tasks, decompose into subtasks:

```
## Execution Plan

**Objectives:**
1. [outcome 1]
2. [outcome 2]

**Subtasks:**
1. [researcher] Research [what] - [why this is needed]
2. [coder] Implement [what] - [depends on: 1]
3. [reviewer] Review [what] - [depends on: 2]

**Execution:** Sequential / Parallel
**Definition of Done:**
- [measurable criterion 1]
- [measurable criterion 2]
```

### 3. Execute Subtasks

For each subtask, dispatch a worker subagent using Task tool:

- Use `bach:orchestrating-work/workers/coder-prompt.md` for coder tasks
- Use `bach:orchestrating-work/workers/researcher-prompt.md` for researcher tasks
- Use `bach:orchestrating-work/workers/reviewer-prompt.md` for reviewer tasks

Provide full context. Workers should not need to ask questions.

### 4. Synthesize Results

After all workers return:
- Verify each completed successfully (watch for INCAPABLE)
- Check results integrate properly
- Verify Definition of Done is met
- Report summary to user with key artifacts

## Available Workers

| Worker | Use For |
|--------|---------|
| `researcher` | Investigating options, analyzing approaches, finding information |
| `coder` | Writing code, implementing features, fixing bugs |
| `reviewer` | Code review, quality assessment, finding issues |

## Example

```
User: /orchestrate Create a CLI tool that fetches weather data and displays it nicely

Manager thinks:
- Need to research weather APIs (researcher)
- Implement the CLI (coder)
- Review for edge cases (reviewer)

Manager outputs:
## Execution Plan

**Objectives:**
1. Working CLI that fetches weather
2. Clean, readable output
3. Good error handling

**Subtasks:**
1. [researcher] Research free weather APIs - find one with good free tier and simple auth
2. [coder] Implement CLI - based on API choice, handle errors, format output
3. [reviewer] Review implementation - edge cases, error messages, UX

**Execution:** Sequential
**Definition of Done:**
- CLI accepts city name as argument
- Displays temperature, conditions, humidity
- Handles invalid city gracefully
- Works offline gracefully (clear error)

[Proceeds to dispatch workers...]
```

## When NOT to Orchestrate

- Simple questions → Just answer
- Single-file changes → Just do it
- Quick fixes → Just fix it
- User explicitly wants you to do it → Don't delegate

The overhead of orchestration should be worth it. Three workers for a one-line fix is overkill.
