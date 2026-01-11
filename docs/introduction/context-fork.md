# Context Fork: The Key Innovation

This is the most important concept in this repository. Understanding context fork unlocks the full power of the PRD Breakdown Execute workflow.

---

## The Problem

Traditional LLM-based development suffers from **context pollution**.

### Without Isolation

```
┌───────────────────────────────────────────────────────────────┐
│                      Single Context                           │
│                                                               │
│  Task 1: Creating User model...                               │
│  Task 1: Added fields: id, email, name, password_hash...      │
│  Task 1: Error: forgot to import datetime...                  │
│  Task 1: Fixed import, model complete.                        │
│                                                               │
│  Task 2: Now creating Project model...                        │
│  Task 2: Wait, was User using UUID or int for id?             │
│  Task 2: Checking... scrolling up... context getting long...  │
│  Task 2: Accidentally copied User patterns that don't apply...│
│                                                               │
│  Task 3: Creating API endpoints...                            │
│  Task 3: Context window filling up...                         │
│  Task 3: Can't remember early decisions...                    │
│  Task 3: Making inconsistent choices...                       │
│                                                               │
│  Result: Degraded quality, compounding errors                 │
└───────────────────────────────────────────────────────────────┘
```

**Problems:**
- Earlier task details pollute later tasks
- Errors from one task confuse subsequent work
- Context window fills with irrelevant information
- No true parallelism - must wait for each task
- Verification biased by seeing implementation

---

## The Solution: Context Fork

Claude Code skills can declare `context: fork` to run in isolated context.

### In SKILL.md

```yaml
---
name: execute-task
context: fork        # ← This is the key
model: sonnet
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
---

Instructions for this skill...
```

### What Happens

When a skill with `context: fork` is invoked:

1. **Fresh context created** - Starts with empty history
2. **Only passed data visible** - Receives only what's explicitly given
3. **Complete isolation** - Cannot see parent or sibling contexts
4. **Clean return** - Only the result goes back to parent

### With Isolation

```
Main Context (Orchestrator)
     │
     │ "Execute these 3 tasks"
     │
     ├──► Fork: Task L1-001 ────────────────────► "Complete"
     │         │
     │         └── Fresh context
     │         └── Only sees: Task L1-001 spec
     │         └── Full focus on one task
     │         └── Returns only: result status
     │
     ├──► Fork: Task L1-002 ────────────────────► "Complete"
     │         │
     │         └── Fresh context
     │         └── No knowledge of L1-001
     │         └── Cannot be confused by L1-001's errors
     │         └── Completely independent
     │
     └──► Fork: Task L1-003 ────────────────────► "Complete"
               │
               └── Fresh context
               └── No knowledge of L1-001 or L1-002
               └── Clean implementation
```

Each task:
- Sees only its own specification
- Cannot be confused by other tasks
- Uses full context window for its work
- Returns only its result

---

## Hierarchical Orchestration

The execute skill demonstrates multi-level forking:

```
/execute (Sonnet)                           Level 0: Orchestrator
    │
    │ Fork with: layer info, state reference
    │
    └─► /execute-layer (Sonnet)             Level 1: Layer Manager
            │
            │ Fork with: batch info, task list
            │
            └─► /execute-batch (Sonnet)     Level 2: Batch Coordinator
                    │
                    │ Fork with: task spec only
                    │
                    ├─► /execute-task (Sonnet)    Level 3: Implementer
                    │       │
                    │       │ Fork with: task spec only
                    │       │
                    │       └─► /execute-verify (Haiku)  Verifier
                    │               └── Cannot see implementation
                    │               └── Only sees: verification steps
                    │               └── Unbiased assessment
                    │
                    ├─► /execute-task (Sonnet)    (parallel)
                    │       └─► /execute-verify (Haiku)
                    │
                    └─► /execute-task (Sonnet)    (parallel)
                            └─► /execute-verify (Haiku)
```

Each level:
- Has its own isolated context
- Passes only necessary information down
- Receives only results back up

---

## Parallel Execution

Context fork enables true parallelism:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         execute-batch context                           │
│                                                                         │
│   t=0s:  Launch all tasks simultaneously                               │
│                                                                         │
│   ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐     │
│   │   Fork: L1-001   │  │   Fork: L1-002   │  │   Fork: L1-003   │     │
│   │                  │  │                  │  │                  │     │
│   │ Read task XML    │  │ Read task XML    │  │ Read task XML    │     │
│   │ Write tests      │  │ Write tests      │  │ Write tests      │     │
│   │ Implement code   │  │ Implement code   │  │ Implement code   │     │
│   │ Run verification │  │ Run verification │  │ Run verification │     │
│   │                  │  │                  │  │                  │     │
│   │ t=8s: Complete   │  │ t=15s: Complete  │  │ t=10s: Complete  │     │
│   └──────────────────┘  └──────────────────┘  └──────────────────┘     │
│                                                                         │
│   No task sees any other task's:                                       │
│   - Code                                                                │
│   - Errors                                                              │
│   - Context                                                             │
│   - Decisions                                                           │
│                                                                         │
│   Results collected: [L1-001: pass, L1-002: pass, L1-003: pass]        │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Independent Verification

The verifier runs in its own fork:

```
execute-task context                    execute-verify context
         │                                       │
         │ Implements code...                    │
         │ Makes decisions...                    │
         │ Might have bugs...                    │
         │                                       │
         ▼                                       │
    "Please verify"  ─────────────────────────►  │
                                                 │
         (Does NOT pass implementation)          │
         (Only passes verification spec)         │
                                                 ▼
                                          Fresh context
                                          Only sees:
                                          - Verification commands
                                          - Expected outcomes

                                          Does NOT see:
                                          - Implementation code
                                          - Design decisions
                                          - Error history

                                          Unbiased: "PASS" or "FAIL"
```

This prevents the verifier from:
- Assuming the implementation is correct
- Missing bugs because it saw the intention
- Being influenced by error-fix cycles

---

## Benefits Summary

| Aspect | Without Fork | With Fork |
|--------|--------------|-----------|
| **Context isolation** | None - everything shares | Complete - each task separate |
| **Parallelism** | Sequential only | True parallel execution |
| **Error propagation** | Spreads to all tasks | Contained to single task |
| **Context usage** | Cumulative growth | Fixed per-task |
| **Verification bias** | High - saw implementation | None - independent |
| **Scalability** | Degrades with task count | Linear performance |
| **Quality** | Degrades over time | Consistent per task |

---

## How to Use Context Fork

### In Your Own Skills

Create a `SKILL.md` file:

```yaml
---
name: my-isolated-skill
context: fork
model: sonnet
allowed-tools:
  - Read
  - Write
  - Bash
---

# My Isolated Skill

This skill runs in its own context.

## Instructions

1. Receive task specification
2. Complete the work
3. Return result

The skill cannot see parent context or sibling forks.
```

### Key Considerations

1. **Pass only needed data** - Fork receives only what you explicitly provide
2. **Return only results** - Intermediate work stays in fork
3. **Use for parallelizable work** - Independent tasks benefit most
4. **Consider model selection** - Cheaper models for focused tasks

---

## Real-World Impact

In this workflow:

- **10 tasks** can run in 3 parallel batches instead of 10 sequential steps
- **Each task** gets full context window for its work
- **Verification** is truly independent
- **Failures** are isolated and retryable
- **State** is cleanly managed outside forked contexts

---

## Next Steps

- [Architecture Overview](architecture.md) - See the full system design
- [Execute Hierarchy](../skills/execute/hierarchy.md) - How execution uses forks
- [Quick Start](../quickstart/README.md) - See it in action
