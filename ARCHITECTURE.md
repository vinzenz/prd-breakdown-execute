# Architecture

This document describes the system architecture for developers who want to understand, extend, or contribute to the PRD Breakdown Execute workflow.

---

## System Overview

```
                                    USER
                                      │
                                      ▼
          ┌───────────────────────────────────────────────────────┐
          │                    Claude Code CLI                    │
          └───────────────────────────────────────────────────────┘
                                      │
             ┌────────────────────────┼────────────────────────┐
             │                        │                        │
             ▼                        ▼                        ▼
      ┌─────────────┐          ┌─────────────┐          ┌─────────────┐
      │    /prd     │          │  /breakdown │          │  /execute   │
      │   Command   │          │    Skill    │          │    Skill    │
      │             │          │             │          │             │
      │  context:   │          │  context:   │          │  context:   │
      │  (default)  │          │  fork       │          │  fork       │
      └──────┬──────┘          └──────┬──────┘          └──────┬──────┘
             │                        │                        │
             │                        │                        │
             ▼                        │                        │
      ┌─────────────┐                 │                        │
      │  8-Phase    │                 │                        │
      │  Workflow   │                 │                        │
      └──────┬──────┘                 │                        │
             │                        │                        │
             ▼                        │                        │
      docs/prd/{slug}/                │                        │
      ├── index.md                    │                        │
      ├── what-next.md                │                        │
      └── features/                   │                        │
                                      │                        │
                    ┌─────────────────┴─────────────────┐      │
                    │                                   │      │
                    ▼                                   │      │
          ┌───────────────────┐                         │      │
          │  breakdown-       │                         │      │
          │  analyze-prd      │                         │      │
          │  context: fork    │                         │      │
          └─────────┬─────────┘                         │      │
                    │                                   │      │
                    ▼                                   │      │
          ┌───────────────────┐                         │      │
          │  breakdown-       │                         │      │
          │  plan-layers      │                         │      │
          │  context: fork    │                         │      │
          └─────────┬─────────┘                         │      │
                    │                                   │      │
                    ▼                                   │      │
          ┌───────────────────┐                         │      │
          │  breakdown-       │◄────────────────────────┘      │
          │  generate-tasks   │                                │
          │  context: fork    │                                │
          └─────────┬─────────┘                                │
                    │                                          │
                    ▼                                          │
          ┌───────────────────┐                                │
          │  breakdown-       │                                │
          │  review-tasks     │                                │
          │  context: fork    │                                │
          └─────────┬─────────┘                                │
                    │                                          │
                    ▼                                          │
          docs/tasks/{slug}/                                   │
          ├── analysis.json                                    │
          ├── layer_plan.json                                  │
          ├── manifest.json                                    │
          ├── 0-setup/                                         │
          ├── 1-foundation/                                    │
          ├── 2-backend/                                       │
          ├── 3-frontend/                                      │
          └── 4-integration/                                   │
                                                               │
                              ┌────────────────────────────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │   execute-layer   │
                    │   context: fork   │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │   execute-batch   │
                    │   context: fork   │
                    └─────────┬─────────┘
                              │
           ┌──────────────────┼──────────────────┐
           │                  │                  │
           ▼                  ▼                  ▼
    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
    │execute-task │    │execute-task │    │execute-task │
    │context: fork│    │context: fork│    │context: fork│
    └──────┬──────┘    └──────┬──────┘    └──────┬──────┘
           │                  │                  │
           ▼                  ▼                  ▼
    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
    │exec-verify  │    │exec-verify  │    │exec-verify  │
    │context: fork│    │context: fork│    │context: fork│
    └──────┬──────┘    └──────┬──────┘    └──────┬──────┘
           │                  │                  │
           └──────────────────┼──────────────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │   execute-merge   │
                    │   (sequential)    │
                    └───────────────────┘
```

---

## Context Fork Mechanics

### What is Context Fork?

Skills can declare `context: fork` in their SKILL.md frontmatter:

```yaml
---
name: execute-task
context: fork
model: sonnet
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
---
```

When a skill with `context: fork` is invoked:
1. A new, isolated context is created
2. The skill receives only the information passed to it
3. It cannot see parent context or sibling forks
4. Only the final result is returned to the parent

### Why Context Fork Matters

**Without fork:**
```
┌─────────────────────────────────────────────────────────────┐
│                      Single Context                         │
│                                                             │
│  Task 1: Wrote User model...                               │
│  Task 1: Error in line 42...                               │
│  Task 1: Fixed error...                                    │
│  Task 2: Now confused by Task 1 details...                 │
│  Task 2: Incorrectly references Task 1 patterns...         │
│  Task 3: Context window nearly full...                     │
│  Task 3: Losing early context...                           │
│                                                             │
│  Result: Degraded quality, context exhaustion              │
└─────────────────────────────────────────────────────────────┘
```

**With fork:**
```
Main Context                       │
(Orchestrator)                     │ Returns: "L1-001 completed"
    │                              │
    ├── Fork: Task L1-001 ─────────┘
    │   └── Clean context
    │   └── Only task spec
    │   └── Full focus
    │
    ├── Fork: Task L1-002 ────────────── Returns: "L1-002 completed"
    │   └── No knowledge of L1-001
    │   └── Fresh context
    │
    └── Fork: Task L1-003 ────────────── Returns: "L1-003 completed"
        └── Completely independent
```

### Benefits

| Aspect | Without Fork | With Fork |
|--------|--------------|-----------|
| Context isolation | None | Complete |
| Parallelism | Sequential only | True parallel |
| Error propagation | Spreads to all | Contained |
| Context usage | Cumulative | Per-task |
| Verification bias | High | None |
| Scalability | Degrades | Linear |

---

## Model Selection Strategy

Different components use different models based on their requirements:

| Component | Model | Reasoning |
|-----------|-------|-----------|
| `/prd` | sonnet | Complex reasoning for requirements |
| `/breakdown` | sonnet | Architecture decisions |
| `breakdown-analyze-prd` | sonnet | Feature extraction, inference |
| `breakdown-plan-layers` | sonnet | Dependency analysis |
| `breakdown-generate-tasks` | sonnet | Detailed specification |
| `breakdown-review-tasks` | haiku | Fast quality validation |
| `/execute` | sonnet | Orchestration logic |
| `execute-layer` | sonnet | Layer coordination |
| `execute-batch` | sonnet | Parallel task launch |
| `execute-task` | sonnet | Code implementation |
| `execute-verify` | haiku | Fast, focused verification |
| `execute-merge` | sonnet | Git operations |

### Selection Principles

1. **Sonnet for reasoning**: Architecture, implementation, orchestration
2. **Haiku for speed**: Verification, review, quality checks
3. **Independent verification**: Verifier uses different model than implementer

---

## File Structure

```
.claude/
├── commands/
│   └── prd.md                    # /prd command (user-invocable)
│
├── skills/
│   ├── breakdown/
│   │   ├── SKILL.md              # Main orchestrator
│   │   └── references/
│   │       ├── layer-definitions.md    # 5-layer architecture
│   │       ├── layer0-templates.md     # Setup task templates
│   │       ├── review-criteria.md      # Task validation rules
│   │       └── task-format-spec.md     # XML schema
│   │
│   ├── breakdown-analyze-prd/
│   │   └── SKILL.md              # PRD analysis
│   │
│   ├── breakdown-plan-layers/
│   │   └── SKILL.md              # Layer planning
│   │
│   ├── breakdown-generate-tasks/
│   │   └── SKILL.md              # Task generation
│   │
│   ├── breakdown-review-tasks/
│   │   └── SKILL.md              # Task review
│   │
│   ├── execute/
│   │   ├── SKILL.md              # Main orchestrator
│   │   └── references/
│   │       ├── options.md        # CLI arguments
│   │       └── state-schema.md   # State file format
│   │
│   ├── execute-layer/
│   │   └── SKILL.md              # Layer execution
│   │
│   ├── execute-batch/
│   │   └── SKILL.md              # Batch coordination
│   │
│   ├── execute-task/
│   │   ├── SKILL.md              # Task implementation
│   │   └── references/
│   │       ├── commit-format.md  # Git commit conventions
│   │       └── tdd-workflow.md   # TDD process
│   │
│   ├── execute-verify/
│   │   └── SKILL.md              # Independent verification
│   │
│   └── execute-merge/
│       ├── SKILL.md              # Sequential merge
│       └── references/
│           └── merge-strategy.md # Merge process
│
└── agents/
    ├── task-generator.md         # Task XML creation
    ├── task-implementer.md       # Small-context execution
    ├── task-reviewer.md          # Quality validation
    └── verification-runner.md    # Command execution
```

### Skills vs Commands vs Agents

| Type | Location | Purpose | Invocation |
|------|----------|---------|------------|
| Command | `.claude/commands/` | User-invocable entry points | `/command-name` |
| Skill | `.claude/skills/` | Reusable workflow components | Called by other skills |
| Agent | `.claude/agents/` | Specialized task executors | Used with Task tool |

---

## Data Flow

### PRD Phase

```
User Input (idea description)
         │
         ▼
    ┌─────────────────────┐
    │    /prd Command     │
    │                     │
    │  Phase 1: Idea      │
    │  Phase 2: Tech      │
    │  Phase 3: Features  │
    │  Phase 4: Deps      │
    │  Phase 5-6: Options │
    │  Phase 7: Review    │
    │  Phase 8: Output    │
    └─────────┬───────────┘
              │
              ▼
    docs/prd/{slug}/
    ├── index.md         ◄── Full PRD in XML format
    ├── what-next.md     ◄── Progress tracking
    └── features/
        ├── feature-1.md ◄── Feature details
        └── feature-2.md
```

### Breakdown Phase

```
docs/prd/{slug}/index.md
         │
         ▼
    ┌─────────────────────┐
    │  breakdown-         │
    │  analyze-prd        │──────► analysis.json
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │  breakdown-         │
    │  plan-layers        │──────► layer_plan.json
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────────────────────────┐
    │  For each layer (0-4):                  │
    │                                         │
    │    ┌─────────────────────┐              │
    │    │ breakdown-          │              │
    │    │ generate-tasks      │──► L{n}-*.xml│
    │    └─────────┬───────────┘              │
    │              │                          │
    │              ▼                          │
    │    ┌─────────────────────┐              │
    │    │ breakdown-          │              │
    │    │ review-tasks        │──► PASS/FAIL │
    │    └─────────────────────┘              │
    │              │                          │
    │         (retry if fail)                 │
    └─────────────────────────────────────────┘
              │
              ▼
    docs/tasks/{slug}/
    ├── manifest.json    ◄── Final inventory
    ├── 0-setup/
    ├── 1-foundation/
    ├── 2-backend/
    ├── 3-frontend/
    └── 4-integration/
```

### Execute Phase

```
docs/tasks/{slug}/
         │
         ▼
    ┌─────────────────────┐
    │     /execute        │──────► execute-state.json
    │   (orchestrator)    │        (created/updated)
    └─────────┬───────────┘
              │
    ┌─────────┴─────────────────────────────────────────┐
    │  For each layer (0-4):                            │
    │                                                   │
    │    ┌─────────────────────┐                        │
    │    │   execute-layer     │                        │
    │    └─────────┬───────────┘                        │
    │              │                                    │
    │    ┌─────────┴─────────────────────────────┐      │
    │    │  For each batch of ready tasks:       │      │
    │    │                                       │      │
    │    │    ┌─────────────────────┐            │      │
    │    │    │   execute-batch     │            │      │
    │    │    └─────────┬───────────┘            │      │
    │    │              │                        │      │
    │    │         (parallel)                    │      │
    │    │    ┌─────────┼─────────┐              │      │
    │    │    │         │         │              │      │
    │    │    ▼         ▼         ▼              │      │
    │    │  ┌───┐     ┌───┐     ┌───┐            │      │
    │    │  │T1 │     │T2 │     │T3 │            │      │
    │    │  └─┬─┘     └─┬─┘     └─┬─┘            │      │
    │    │    │         │         │              │      │
    │    │    ▼         ▼         ▼              │      │
    │    │  ┌───┐     ┌───┐     ┌───┐            │      │
    │    │  │V1 │     │V2 │     │V3 │            │      │
    │    │  └───┘     └───┘     └───┘            │      │
    │    │    │         │         │              │      │
    │    │    └─────────┼─────────┘              │      │
    │    │              │                        │      │
    │    │         (sequential)                  │      │
    │    │    ┌─────────┴─────────┐              │      │
    │    │    │   execute-merge   │              │      │
    │    │    │   (one at a time) │              │      │
    │    │    └───────────────────┘              │      │
    │    │                                       │      │
    │    └───────────────────────────────────────┘      │
    │                                                   │
    └───────────────────────────────────────────────────┘
              │
              ▼
    Working code in project directory
```

---

## Extension Points

### Adding New Templates

1. Create template in your templates directory
2. Update `layer0-templates.md` with setup instructions
3. Add template detection keywords

### Adding New Layers

1. Update `layer-definitions.md` with new layer
2. Adjust layer numbering
3. Update dependency rules
4. Add to breakdown planning logic

### Customizing Review Criteria

1. Edit `review-criteria.md`
2. Add/remove critical vs warning criteria
3. Adjust pass/fail thresholds

### Adding New Skills

1. Create `SKILL.md` in `.claude/skills/{skill-name}/`
2. Define context mode (`fork` or default)
3. Specify model and allowed tools
4. Add reference files if needed

---

## Concurrency Model

### Parallel Execution

```
Batch with 3 tasks, max_parallel=3

t=0s:   Launch T1, T2, T3 in parallel
        ├── T1: worktree-L1-001
        ├── T2: worktree-L1-002
        └── T3: worktree-L1-003

t=8s:   T3 completes → merge queue
t=10s:  T1 completes → merge queue
t=15s:  T2 completes → merge queue

Merge (sequential):
t=16s:  Merge T3 to main
t=17s:  Merge T1 to main
t=18s:  Merge T2 to main
```

### Why Sequential Merge?

Multiple tasks may modify the same files (e.g., `__init__.py`). Sequential merging prevents conflicts and ensures clean git history.

### State Synchronization

- State file updated atomically (temp file + rename)
- Each task tracks its own status
- Merge queue processed in completion order

---

## Error Handling

### Task Failure Recovery

```
Task fails verification
         │
         ▼
    ┌─────────────────────┐
    │  Attempt < 5?       │
    │                     │
    │  YES → Retry with   │
    │        feedback     │
    │                     │
    │  NO → Abandon task  │
    │       Stop layer    │
    └─────────────────────┘
```

### State Recovery

If execution is interrupted:
1. Load `execute-state.json`
2. Identify incomplete tasks
3. Resume from last known state
4. Retry failed tasks (if attempts < 5)

### Worktree Preservation

Failed task worktrees are preserved for debugging:
```bash
ls .worktrees/
# L1-001/  ← Failed task, preserved
# L1-002/  ← Completed, merged and removed
```

---

## Security Considerations

- Tasks run in isolated git worktrees
- No cross-task file access
- Verification runs in separate context (cannot be biased)
- State file is local (no network exposure)
- Git history preserved for audit
