# Skills Reference

An overview of all skills in the PRD Breakdown Execute workflow.

---

## Skill Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                             User Commands                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│       /prd                  /breakdown                  /execute            │
│      (command)               (skill)                    (skill)             │
└────────┬────────────────────────┬────────────────────────────┬──────────────┘
         │                        │                            │
         │                        │                            │
         ▼                        ▼                            ▼
    ┌─────────┐          ┌────────────────┐           ┌────────────────┐
    │8 Phases │          │ analyze-prd    │           │ execute-layer  │
    │         │          └────────────────┘           └────────────────┘
    │         │          ┌────────────────┐           ┌────────────────┐
    │         │          │ plan-layers    │           │ execute-batch  │
    │         │          └────────────────┘           └────────────────┘
    │         │          ┌────────────────┐           ┌────────────────┐
    │         │          │ generate-tasks │           │ execute-task   │
    │         │          └────────────────┘           └────────────────┘
    │         │          ┌────────────────┐           ┌────────────────┐
    │         │          │ review-tasks   │           │ execute-verify │
    └─────────┘          └────────────────┘           └────────────────┘
                                                      ┌────────────────┐
                                                      │ execute-merge  │
                                                      └────────────────┘
```

---

## Quick Reference

| Skill | Context | Model | Purpose |
|-------|---------|-------|---------|
| [`/prd`](prd.md) | default | sonnet | Interactive PRD creation |
| [`/breakdown`](breakdown/README.md) | fork | sonnet | PRD to task conversion |
| `breakdown-analyze-prd` | fork | sonnet | Extract features, infer models |
| `breakdown-plan-layers` | fork | sonnet | Organize into 5 layers |
| `breakdown-generate-tasks` | fork | sonnet | Create task XML files |
| `breakdown-review-tasks` | fork | haiku | Validate task quality |
| [`/execute`](execute/README.md) | fork | sonnet | Task orchestration |
| `execute-layer` | fork | sonnet | Layer management |
| `execute-batch` | fork | sonnet | Parallel task launch |
| `execute-task` | fork | sonnet | TDD implementation |
| `execute-verify` | fork | haiku | Independent verification |
| `execute-merge` | fork | sonnet | Sequential merge |

---

## Context Modes

### Default Context
The skill shares context with its caller. Used for interactive commands like `/prd` where conversation history matters.

### Fork Context
The skill runs in isolated context. Used for:
- Parallel execution (no interference)
- Independent verification (no bias)
- Clean separation of concerns

```yaml
# In SKILL.md
---
name: execute-task
context: fork      # ← Isolated context
model: sonnet
---
```

[Learn more about Context Fork](../introduction/context-fork.md)

---

## Model Selection

### Sonnet (Reasoning)
Used for tasks requiring:
- Architecture decisions
- Code implementation
- Complex orchestration
- Feature extraction

### Haiku (Speed)
Used for tasks requiring:
- Quick verification
- Quality checks
- Simple validation
- High volume operations

---

## Skill Categories

### PRD Creation
- [/prd](prd.md) - Interactive 8-phase PRD creation

### Task Breakdown
- [/breakdown](breakdown/README.md) - Main orchestrator
- [Layers](breakdown/layers.md) - 5-layer architecture
- [Task Format](breakdown/task-format.md) - XML specification
- [Sub-skills](breakdown/sub-skills.md) - Component skills

### Task Execution
- [/execute](execute/README.md) - Main orchestrator
- [Hierarchy](execute/hierarchy.md) - 4-level structure
- [Worktrees](execute/worktrees.md) - Git isolation
- [TDD](execute/tdd.md) - Test-first workflow
- [State](execute/state.md) - State management
- [Sub-skills](execute/sub-skills.md) - Component skills

---

## Skill Files Location

```
.claude/
├── commands/
│   └── prd.md                    # /prd command
│
└── skills/
    ├── breakdown/
    │   ├── SKILL.md              # Main skill
    │   └── references/           # Supporting docs
    │
    ├── breakdown-analyze-prd/
    │   └── SKILL.md
    │
    ├── breakdown-plan-layers/
    │   └── SKILL.md
    │
    ├── breakdown-generate-tasks/
    │   └── SKILL.md
    │
    ├── breakdown-review-tasks/
    │   └── SKILL.md
    │
    ├── execute/
    │   ├── SKILL.md
    │   └── references/
    │
    ├── execute-layer/
    │   └── SKILL.md
    │
    ├── execute-batch/
    │   └── SKILL.md
    │
    ├── execute-task/
    │   ├── SKILL.md
    │   └── references/
    │
    ├── execute-verify/
    │   └── SKILL.md
    │
    └── execute-merge/
        ├── SKILL.md
        └── references/
```

---

## Next Steps

- [/prd Command](prd.md) - PRD creation reference
- [/breakdown Skill](breakdown/README.md) - Task generation
- [/execute Skill](execute/README.md) - Parallel execution
