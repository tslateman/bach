# Task Contract: Flow → Bach

Flow orchestrates. Bach executes. The contract is a task envelope.

## Task Envelope (Flow sends)

```json
{
  "id": "2.1",
  "worker": "coder",
  "task": "Implement POST /auth/login endpoint",
  "context": {
    "acceptanceCriteria": "Returns JWT on valid credentials, 401 on invalid",
    "dependencies": ["Research recommended bcrypt for password hashing"],
    "constraints": ["Use existing Express setup", "PostgreSQL for storage"],
    "files": ["src/routes/auth.ts", "src/models/user.ts"]
  }
}
```

## Envelope Fields

| Field                        | Type      | Purpose                           |
| ---------------------------- | --------- | --------------------------------- |
| `id`                         | string    | Task ID for Flow to track         |
| `worker`                     | string    | Bach worker type to invoke        |
| `task`                       | string    | What to do (imperative, specific) |
| `context.acceptanceCriteria` | string    | How Flow will judge completion    |
| `context.dependencies`       | string[]  | Outputs from prior tasks          |
| `context.constraints`        | string[]  | Boundaries from project scope     |
| `context.files`              | string[]? | Relevant files to read/modify     |

## Worker Vocabulary

| `worker`     | Bach template                    | Specialty                             |
| ------------ | -------------------------------- | ------------------------------------- |
| `researcher` | `./workers/researcher-prompt.md` | Investigate, analyze, recommend       |
| `coder`      | `./workers/coder-prompt.md`      | Implement, test, debug                |
| `reviewer`   | `./workers/reviewer-prompt.md`   | Evaluate, find issues, approve/reject |
| `tester`     | `./workers/tester-prompt.md`     | Write tests, find edge cases          |

## Result Envelope (Bach returns)

```json
{
  "id": "2.1",
  "status": "complete",
  "summary": "Implemented login endpoint with bcrypt password verification",
  "artifacts": ["src/routes/auth.ts", "src/routes/auth.test.ts"],
  "notes": "Added rate limiting middleware as defensive measure"
}
```

**Or on failure:**

```json
{
  "id": "2.1",
  "status": "incapable",
  "reason": "Task requires database schema changes outside my scope",
  "suggestion": "Spawn a coder task first to add password_hash column"
}
```

## Result Fields

| Field        | Type      | Purpose                            |
| ------------ | --------- | ---------------------------------- |
| `id`         | string    | Task ID (echo back)                |
| `status`     | string    | `complete`, `incapable`, `blocked` |
| `summary`    | string    | What was done (past tense)         |
| `artifacts`  | string[]? | Files created/modified             |
| `notes`      | string?   | Observations for Flow              |
| `reason`     | string?   | Why incapable/blocked              |
| `suggestion` | string?   | How to unblock                     |

## Status Vocabulary

| `status`    | Meaning                            | Flow's response                  |
| ----------- | ---------------------------------- | -------------------------------- |
| `complete`  | Task done, criteria met            | Mark task complete, continue     |
| `incapable` | Wrong worker or missing dependency | Reassign or add blocking task    |
| `blocked`   | External blocker                   | Add to blockers, try other tasks |
| `partial`   | Some progress, needs continuation  | Record progress, re-queue        |

## Invocation Pattern

Flow's `/flow:execute` does this:

```
For each task in current wave:
  1. Build envelope from task + project context
  2. Select Bach template based on task.worker
  3. Invoke via Task tool:
       description: "{worker}: {task.title}"
       prompt: [Bach template with envelope injected]
  4. Parse result envelope
  5. Update task status in state.json
  6. If incapable/blocked: record and continue others
```

## Template Injection Point

Bach templates expect a `## Task Envelope` section:

```markdown
# Coder Worker

You are a Coder specialist. You implement, test, and debug.

## Task Envelope

{envelope as YAML or JSON}

## Your Scope

**You handle:** Writing code, writing tests, fixing bugs
**You do NOT handle:** Architecture decisions, research, deployment

## Output Format

Return a JSON result envelope:
...
```

## Interface Boundaries

| Flow knows                  | Flow doesn't know             |
| --------------------------- | ----------------------------- |
| Worker types exist          | How workers do their job      |
| Envelope format             | Internal worker prompts       |
| Result format               | Worker decision-making        |
| Which worker for which task | Worker implementation details |

| Bach knows              | Bach doesn't know             |
| ----------------------- | ----------------------------- |
| Envelope it received    | Other tasks in the phase      |
| Its specialty           | Project state                 |
| How to signal incapable | Flow's DAG structure          |
| Output format           | What happens after it returns |
