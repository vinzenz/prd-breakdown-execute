# Execution Hierarchy

The 4-level orchestration structure for task execution.

---

## Overview

Execution uses a 4-level hierarchy, each level running in forked context:

```
Level 0: /execute (Orchestrator)
           │
           │ Controls: All layers, global state
           │ Forks: execute-layer for each layer
           │
Level 1:   └─► execute-layer
                  │
                  │ Controls: One layer, batching
                  │ Forks: execute-batch for each batch
                  │
Level 2:          └─► execute-batch
                         │
                         │ Controls: Parallel task launch
                         │ Forks: execute-task for each task (parallel)
                         │ Uses: Task tool with run_in_background
                         │
Level 3:                 ├─► execute-task ──► execute-verify
                         │       └── Worktree
                         ├─► execute-task ──► execute-verify
                         │       └── Worktree
                         └─► execute-task ──► execute-verify
                                 └── Worktree
```

---

## Level 0: /execute (Orchestrator)

**Purpose:** Top-level coordination across all layers.

**Model:** Sonnet
**Context:** Fork

**Responsibilities:**
- Parse arguments and configuration
- Load manifest and layer plans
- Initialize or resume execution state
- Process layers sequentially (0 → 4)
- Handle stop conditions
- Report final status

**Flow:**
```
Load state
    │
    ├── If resuming: Identify incomplete layers
    │
    └── For each layer:
            │
            └── Fork: execute-layer
                    │
                    └── Wait for completion
                            │
                            └── Update state
```

**Data passed down:**
- Layer configuration
- State file path
- Tasks path
- Max parallel setting

---

## Level 1: execute-layer

**Purpose:** Manage execution of a single layer.

**Model:** Sonnet
**Context:** Fork

**Responsibilities:**
- Load layer plan and dependencies
- Build ready queue (tasks with satisfied deps)
- Group ready tasks into batches
- Process batches sequentially
- Handle merge queue after each batch
- Mark layer complete

**Flow:**
```
Load layer plan
    │
    └── While ready tasks exist:
            │
            ├── Build ready queue
            │
            ├── Group into batch (max_parallel)
            │
            ├── Fork: execute-batch
            │       │
            │       └── Wait for results
            │
            └── Process merge queue
                    │
                    └── Fork: execute-merge (for each verified task)
```

**Dependency handling:**
```
Ready queue algorithm:

1. Get all pending tasks in layer
2. For each task:
   - Check if all dependencies completed
   - If yes: add to ready queue
3. Take min(ready_queue.length, max_parallel) tasks
4. Launch as batch
```

**Example:**
```
Layer 2 tasks:
  L2-001 (no deps)     ──► Ready
  L2-002 (no deps)     ──► Ready
  L2-003 (deps: 001, 002) ──► Not ready
  L2-004 (deps: 001)   ──► Not ready

Batch 1: [L2-001, L2-002]

After batch 1 completes:
  L2-003 (deps satisfied) ──► Ready
  L2-004 (deps satisfied) ──► Ready

Batch 2: [L2-003, L2-004]
```

---

## Level 2: execute-batch

**Purpose:** Launch tasks in parallel using Task tool.

**Model:** Sonnet
**Context:** Fork

**Responsibilities:**
- Receive task IDs to execute
- Load task XML for each
- Launch ALL tasks in single message (parallel)
- Wait for each to complete
- Collect results
- Update merge queue
- Return batch result

**Critical:** Uses Task tool with `run_in_background: true` to launch all tasks in one message.

**Flow:**
```
Receive task IDs: [L2-001, L2-002, L2-003]
    │
    └── SINGLE MESSAGE with multiple Task calls:
            │
            ├── Task(execute-task, L2-001, run_in_background: true)
            ├── Task(execute-task, L2-002, run_in_background: true)
            └── Task(execute-task, L2-003, run_in_background: true)
    │
    └── For each launched task:
            │
            └── TaskOutput(task_id, block: true, timeout: 600000)
                    │
                    └── Collect result
    │
    └── Return batch result
```

**Why single message?**

Launching all tasks in one message ensures true parallelism:
- All worktrees created immediately
- No sequential delay between launches
- Maximum throughput

**Result handling:**
```
Results:
  L2-001: verified → merge queue
  L2-002: verified → merge queue
  L2-003: failed   → retry next batch
```

---

## Level 3: execute-task

**Purpose:** Implement a single task with TDD.

**Model:** Sonnet
**Context:** Fork

**Responsibilities:**
- Parse task XML
- Create or reuse git worktree
- Write tests first
- Implement code
- Create git commit
- Request verification
- Handle retry with feedback

**Flow:**
```
Read task XML
    │
    ├── Extract: requirements, tests, verification
    │
    ├── First attempt: Create worktree
    │   Retry: Reuse existing worktree
    │
    ├── Write tests from <test-requirements>
    │
    ├── Run tests (expect fail)
    │
    ├── Implement from <requirements>
    │
    ├── Run tests (expect pass)
    │
    ├── Create git commit
    │
    └── Fork: execute-verify
            │
            └── Handle result:
                    │
                    ├── PASS: Return verified
                    │
                    └── FAIL:
                            │
                            ├── Attempt < 5: Apply feedback, commit fix, retry verify
                            │
                            └── Attempt = 5: Return abandoned
```

---

## Level 3: execute-verify

**Purpose:** Independent verification in separate context.

**Model:** Haiku (fast, focused)
**Context:** Fork

**Responsibilities:**
- Parse verification steps from task XML
- Execute each command
- Check expected outcomes
- Provide actionable feedback on failure

**Critical:** Cannot see implementation code or history.

**Flow:**
```
Receive: task XML only (not implementation)
    │
    └── For each verification step:
            │
            ├── Run: command
            │       │
            │       └── Check exit code
            │
            ├── Verify: expected outcome
            │       │
            │       └── Check output patterns
            │
            └── Record result
    │
    └── All passed? PASS : FAIL + feedback
```

**Feedback example:**
```
{
  "verdict": "FAIL",
  "steps": [
    {"step": "pytest tests/api/test_tasks.py", "status": "FAIL"},
    {"step": "All tests pass", "status": "FAIL"}
  ],
  "feedback": "test_create_task failed: expected 201 got 422. Add title validation: min_length=1 in TaskCreateRequest."
}
```

---

## Context Isolation Benefits

Each level is completely isolated:

```
/execute
    │ Cannot see details of:
    │   - How layers execute internally
    │   - Individual task implementations
    │   - Verification outcomes (only results)
    │
    └─► execute-layer
            │ Cannot see details of:
            │   - Other layer executions
            │   - Individual task code
            │
            └─► execute-batch
                    │ Cannot see details of:
                    │   - Other batch executions
                    │   - Task implementation details
                    │
                    └─► execute-task
                            │ Cannot see:
                            │   - Other task implementations
                            │   - Other worktree contents
                            │
                            └─► execute-verify
                                    │ Cannot see:
                                    │   - Implementation code
                                    │   - Design decisions
                                    │   - Error history
```

This prevents:
- Context pollution between tasks
- Verification bias
- Error propagation
- Context window exhaustion

---

## Model Distribution

```
Orchestration (Sonnet):
  /execute ──► execute-layer ──► execute-batch ──► execute-task

Verification (Haiku):
  execute-verify

Merging (Sonnet):
  execute-merge
```

**Sonnet:** Complex decisions, code generation
**Haiku:** Fast validation, simple checks

---

## Concurrency Model

```
Timeline for 6 tasks with max_parallel=3:

t=0s:   Batch 1: Launch L1-001, L1-002, L1-003
        │
t=8s:   L1-003 completes (verified)
t=10s:  L1-001 completes (verified)
t=15s:  L1-002 completes (verified)
        │
t=16s:  Merge L1-003 (first to complete)
t=17s:  Merge L1-001
t=18s:  Merge L1-002
        │
t=19s:  Batch 2: Launch L1-004, L1-005, L1-006
        ...
```

**Parallel:** Task execution
**Sequential:** Merging to main

---

## Next Steps

- [Git Worktrees](worktrees.md)
- [TDD Workflow](tdd.md)
- [State Management](state.md)
- [Sub-skills Reference](sub-skills.md)
