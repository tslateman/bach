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
├── agents/
│   └── architect.md                 # CC agent: design authority
├── commands/
│   ├── orchestrate.md               # /orchestrate — single-pass decomposition
│   └── mission.md                   # /mission — multi-phase lifecycle
├── skills/
│   ├── orchestrating-work/
│   │   ├── SKILL.md                 # Core two-tier orchestration pattern
│   │   └── workers/
│   │       ├── architect-prompt.md  # Design and phase planning template
│   │       ├── coder-prompt.md      # Implementation task template
│   │       ├── researcher-prompt.md # Investigation task template
│   │       ├── reviewer-prompt.md   # Code review task template
│   │       └── tester-prompt.md     # Test writing task template
│   ├── lifecycle/
│   │   ├── SKILL.md                 # Multi-phase orchestration with gates
│   │   └── reference/
│   │       ├── phase-gates.md       # Gate pattern reference
│   │       └── dependency-waves.md  # Wave algorithm reference
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

### Single-Pass: `/orchestrate`

For tasks that decompose into a few subtasks without phase boundaries:

```
/orchestrate Add input validation to the user registration form
```

The Manager analyzes, decomposes, dispatches workers, and synthesizes results in one pass.

### Multi-Phase: `/mission`

For projects that span multiple phases with review gates:

```
/mission Build a CLI tool with auth, config management, and plugin system
```

The Manager dispatches an architect to plan phases, creates tasks with phase prefixes and gate barriers, executes each phase via the two-tier pattern, and gates transitions with reviewer approval.

## Usage

### Commands

| Command        | Pattern     | Use When                                    |
| -------------- | ----------- | ------------------------------------------- |
| `/orchestrate` | Single-pass | 3+ subtasks, no phase boundaries            |
| `/mission`     | Multi-phase | Distinct phases, dependencies, review gates |

### Skills

**`bach:orchestrating-work`** — Core skill teaching the two-tier delegation pattern

**`bach:lifecycle`** — Multi-phase orchestration with architect planning and review gates

**`bach:creating-workers`** — How to create custom specialist workers

### Worker Prompt Templates

- `bach:orchestrating-work/workers/architect-prompt.md`
- `bach:orchestrating-work/workers/coder-prompt.md`
- `bach:orchestrating-work/workers/researcher-prompt.md`
- `bach:orchestrating-work/workers/reviewer-prompt.md`
- `bach:orchestrating-work/workers/tester-prompt.md`

## When to Use

**Good fit for `/orchestrate`:**

- Complex tasks with 3+ distinct parts
- Tasks needing different skills (research, code, review)
- Quality benefits from focused attention
- You want structured delegation

**Good fit for `/mission`:**

- Multi-phase projects (design → build → integrate)
- Phase dependencies (phase 2 needs phase 1's output)
- Quality gates between phases matter
- Architect planning adds value

**Not a good fit:**

- Simple tasks (just do them)
- One-line fixes
- Exploratory investigation (don't know scope yet)

## Core Workers

| Worker       | Specialty                                                        |
| ------------ | ---------------------------------------------------------------- |
| `architect`  | Plan phases, design systems, define interfaces, resolve disputes |
| `researcher` | Investigate options, analyze trade-offs, recommend approaches    |
| `coder`      | Implement features, write tests, fix bugs                        |
| `reviewer`   | Evaluate quality, find issues, approve/reject, gate reviews      |
| `tester`     | Write tests, find edge cases, verify behavior                    |

## Creating Custom Workers

See `bach:creating-workers` for the full guide. Save worker templates to `bach/skills/orchestrating-work/workers/[role]-prompt.md`.

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

- `superpowers:writing-plans` — For larger initiatives
- `superpowers:test-driven-development` — Workers follow TDD
- `superpowers:dispatching-parallel-agents` — Independent subtasks

## Related

- [duet](https://github.com/tslateman/duet) — Git workflows, reflection, code quality commands

## License

MIT
