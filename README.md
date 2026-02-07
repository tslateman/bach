# Bach

Two-tier agent orchestration for Claude Code. A Manager decomposes complex tasks and delegates to specialist Workers.

## The Pattern

```
User Request
     ↓
┌───────────────────────────────────────────────┐
│                    MANAGER                    │
│  • Analyze objectives                         │
│  • Decompose into subtasks                    │
│  • Delegate to workers                        │
│  • Synthesize results                         │
│  • Never does the work                        │
└───────────────────────────────────────────────┘
       ↓                ↓                ↓
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  RESEARCHER │  │    CODER    │  │   REVIEWER  │
│ Investigate │  │  Implement  │  │   Evaluate  │
│  Recommend  │  │    Test     │  │   Approve   │
└─────────────┘  └─────────────┘  └─────────────┘
       ↓                ↓                ↓
       └────────────────┴────────────────┘
                        ↓
                 Final Result
```

## Project Structure

```
bach/
├── .claude-plugin/
│   └── plugin.json                  # Plugin metadata
├── commands/
│   └── orchestrate.md               # /orchestrate slash command
├── skills/
│   ├── orchestrating-work/
│   │   ├── SKILL.md                 # Core two-tier orchestration pattern
│   │   └── workers/
│   │       ├── coder-prompt.md      # Implementation task template
│   │       ├── researcher-prompt.md # Investigation task template
│   │       └── reviewer-prompt.md   # Code review task template
│   └── creating-workers/
│       └── SKILL.md                 # Guide to adding custom workers
├── README.md
└── LICENSE
```

## Installation

```bash
# Clone to your plugins directory
git clone https://github.com/tslateman/bach ~/.claude/plugins/bach
```

Or add to an existing project:

```bash
git clone https://github.com/tslateman/bach .claude-plugins/bach
```

## Quick Start

Here's a hypothetical example showing the orchestration flow:

```
/orchestrate Add input validation to the user registration form
```

The Manager analyzes this and creates an execution plan:

```
## Execution Plan

### Objective
Add robust input validation to user registration

### Subtasks
1. Research validation patterns (researcher)
   - Compare client-side vs server-side approaches
   - Recommend validation library

2. Implement validation (coder)
   - Add form validation rules
   - Display error messages
   - Write unit tests

3. Review implementation (reviewer)
   - Check for edge cases
   - Verify security (XSS, injection)
   - Approve or request changes
```

Each worker reports back with structured results. The Manager synthesizes findings and reports the final outcome.

## Usage

### Command: `/orchestrate`

Break down a complex task using the two-tier pattern:

```
/orchestrate Create a REST API with JWT authentication and rate limiting
```

The Manager will:

1. Identify objectives
2. Create an execution plan with subtasks
3. Dispatch workers (researcher → coder → reviewer)
4. Synthesize and report results

### Skills

**`bach:orchestrating-work`** - Core skill teaching the orchestration pattern

**`bach:creating-workers`** - How to create custom specialist workers

### Worker Prompt Templates

- `bach:orchestrating-work/workers/coder-prompt.md`
- `bach:orchestrating-work/workers/researcher-prompt.md`
- `bach:orchestrating-work/workers/reviewer-prompt.md`

## When to Use

**Good fit:**

- Complex tasks with 3+ distinct parts
- Tasks needing different skills (research, code, review)
- Quality benefits from focused attention
- You want structured delegation

**Not a good fit:**

- Simple tasks (just do them)
- Highly interdependent work
- Exploratory investigation (don't know scope yet)
- One-line fixes

## Core Workers

| Worker       | Specialty                                                     |
| ------------ | ------------------------------------------------------------- |
| `researcher` | Investigate options, analyze trade-offs, recommend approaches |
| `coder`      | Implement features, write tests, fix bugs                     |
| `reviewer`   | Evaluate quality, find issues, approve/reject                 |

## Creating Custom Workers

See `bach:creating-workers` for the full guide. Quick example:

```markdown
# Tester Worker Prompt Template

Task tool (general-purpose):
description: "Tester: [what to test]"
prompt: |
You are a Tester specialist...

    ## Your Task
    [PASTE: What to test]

    ## Scope Boundaries
    **You handle:** Writing tests, finding edge cases
    **You do NOT handle:** Fixing bugs, writing implementation

    [... rest of template]
```

Save to `bach/skills/orchestrating-work/workers/tester-prompt.md`.

## Philosophy

**Manager mindset:**

- Plan, don't execute
- Delegate, don't implement
- Verify, don't assume
- Synthesize, don't just collect

**Worker mindset:**

- Focus on your specialty
- Report INCAPABLE honestly
- Return structured results
- Don't scope creep

## Integration

Works well with:

- `superpowers:writing-plans` - For larger initiatives
- `superpowers:test-driven-development` - Workers follow TDD
- `superpowers:dispatching-parallel-agents` - Independent subtasks

## License

MIT
