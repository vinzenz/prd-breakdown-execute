---
name: execute-layer
description: Handles one layer of task execution. Groups ready tasks into batches, spawns batch agents, and processes merge queue after each batch.
context: fork
allowed-tools: Read Write Bash Skill Glob
model: claude-sonnet-4-5
---

# Layer Execution Agent

You manage the execution of a single layer. You build the ready queue, group tasks into batches, spawn batch agents, and coordinate merges.

## Input Arguments

Parse these from the prompt:

| Argument | Required | Description |
|----------|----------|-------------|
| `--tasks-path <path>` | Yes | Path to tasks directory |
| `--layer <name>` | Yes | Layer to execute (e.g., "1-foundation") |
| `--project-path <path>` | Yes | Main project directory |
| `--worktree-dir <path>` | Yes | Directory for worktrees |
| `--max-parallel <N>` | No | Max concurrent tasks (default: 3) |

## Execution Flow

### Step 1: Load Layer Plan

Read `layer_plan.json` to get dependency graph:

```bash
cat {tasks_path}/layer_plan.json
```

Extract:
- `dependency_graph`: Map of task_id → list of dependency task_ids
- `layers`: List of layer definitions with task lists

### Step 2: Load State

Read `execute-state.json`:

```bash
cat {tasks_path}/execute-state.json
```

Get:
- `completed`: List of completed task IDs
- `failed`: List of failed task IDs
- `abandoned`: List of abandoned task IDs
- `tasks`: Individual task status details
- `merge_queue`: Current merge queue

### Step 3: Build Ready Queue

Find tasks that are ready to execute:

```python
def build_ready_queue(layer, dependency_graph, state):
    ready = []

    for task in layer["tasks"]:
        task_id = task["id"]

        # Skip completed
        if task_id in state["completed"]:
            continue

        # Skip abandoned
        if task_id in state["abandoned"]:
            continue

        # Check dependencies
        deps = dependency_graph.get(task_id, [])
        all_deps_complete = all(d in state["completed"] for d in deps)

        if all_deps_complete:
            ready.append(task_id)

    return ready
```

### Step 4: Update Layer Status

Mark layer as in_progress in state:

```json
{
  "layers": {
    "{layer}": {
      "status": "in_progress",
      "started_at": "{now}",
      "tasks_total": 6,
      "tasks_completed": 0,
      "tasks_failed": 0
    }
  },
  "current_layer": "{layer}"
}
```

### Step 5: Execute Batches Loop

While there are ready tasks:

#### 5a. Build Batch

Group ready tasks up to `max_parallel`:

```python
batch = ready_queue[:max_parallel]
```

Assign batch number:
```python
batch_number = current_batch + 1
state["current_batch"] = batch_number
```

#### 5b. Spawn Batch Agent

Invoke `/execute-batch` skill:

```
/execute-batch --tasks-path {tasks_path} --task-ids {comma_separated_ids} --project-path {project_path} --worktree-dir {worktree_dir} --batch-number {batch_number} --layer {layer}
```

Wait for batch completion.

#### 5c. Handle Batch Result

Parse batch result:

```json
{
  "batch_number": 1,
  "verified": ["L1-001", "L1-002"],
  "failed": ["L1-003"],
  "abandoned": [],
  "should_stop": false
}
```

**If `should_stop: true`:**
- A task was abandoned (max 5 retries)
- Report to orchestrator immediately
- Do NOT process more batches

#### 5d. Process Merge Queue

After each batch, merge verified tasks in order:

```python
for item in merge_queue:
    if item["status"] == "ready":
        # Invoke merge
        invoke_merge(item["task_id"])
        item["status"] = "merged"
```

Call `/execute-merge` for each ready task:

```
/execute-merge --task-id {task_id} --project-path {project_path} --worktree-path {worktree_path} --task-file {task_file}
```

**IMPORTANT**: Merge tasks **sequentially** in priority order to avoid conflicts.

#### 5e. Re-evaluate Ready Queue

After batch completes and merges finish:
- Some tasks may now have all dependencies satisfied
- Rebuild ready queue with fresh state
- Continue loop if tasks remain

### Step 6: Update Layer Completion

When no more ready tasks:

```json
{
  "layers": {
    "{layer}": {
      "status": "completed",
      "completed_at": "{now}",
      "tasks_completed": 6,
      "tasks_failed": 0
    }
  }
}
```

### Step 7: Report Layer Result

Output structured result for orchestrator:

```json
{
  "layer": "1-foundation",
  "status": "completed",
  "tasks_total": 6,
  "tasks_completed": 6,
  "tasks_failed": 0,
  "tasks_abandoned": 0,
  "batches_executed": 3,
  "should_stop": false
}
```

**If a task was abandoned:**

```json
{
  "layer": "2-backend",
  "status": "stopped",
  "tasks_total": 9,
  "tasks_completed": 5,
  "tasks_failed": 0,
  "tasks_abandoned": 1,
  "abandoned_task": "L2-006",
  "batches_executed": 2,
  "should_stop": true,
  "stop_reason": "Task L2-006 abandoned after 5 attempts"
}
```

## Batch Construction Algorithm

### Basic Batching

```python
def build_batches(ready_tasks, max_parallel):
    batches = []
    remaining = list(ready_tasks)

    while remaining:
        batch = remaining[:max_parallel]
        remaining = remaining[max_parallel:]
        batches.append(batch)

    return batches
```

### Example

Ready queue: `[L1-001, L1-002, L1-006, L1-003, L1-004, L1-005]`
Max parallel: 3

Batch 1: `[L1-001, L1-002, L1-006]`
Batch 2: `[L1-003, L1-004, L1-005]`

### Dynamic Re-evaluation

After Batch 1 completes:
- L1-001, L1-002, L1-006 now complete
- L1-003 depends on L1-002 → now ready
- L1-004 depends on L1-003 → still blocked
- L1-005 depends on L1-003 → still blocked

New ready queue: `[L1-003]` (only 1 task ready now)

Batch 2 executes with just L1-003.

After Batch 2:
- L1-003 complete
- L1-004 now ready
- L1-005 now ready

Batch 3: `[L1-004, L1-005]`

## Merge Queue Processing

### Sequential Merge Order

Tasks complete in parallel but merge sequentially:

```
Execution Order (parallel):
  L1-001 completes at t=10s
  L1-006 completes at t=15s
  L1-002 completes at t=20s

Merge Order (sequential by priority):
  1. L1-001 (priority 1) → merge at t=21s
  2. L1-002 (priority 2) → merge at t=22s
  3. L1-006 (priority 3) → merge at t=23s
```

### Merge Queue State

```json
{
  "merge_queue": [
    {"task_id": "L1-001", "priority": 1, "status": "merged"},
    {"task_id": "L1-002", "priority": 2, "status": "ready"},
    {"task_id": "L1-006", "priority": 3, "status": "ready"}
  ]
}
```

### Priority Assignment

Assign priority when task is added to merge queue:

```python
def add_to_merge_queue(state, task_id):
    # Priority is order of addition
    next_priority = len(state["merge_queue"]) + 1
    state["merge_queue"].append({
        "task_id": task_id,
        "priority": next_priority,
        "status": "ready"
    })
```

## Error Handling

### Batch Agent Failure

If batch agent crashes or times out:
- Mark all tasks in batch as failed
- Add to retry queue for next batch
- Continue with remaining batches

### Merge Failure

If merge fails (shouldn't happen with sequential merges):
- Keep worktree for debugging
- Mark task as failed
- Report to orchestrator

### State Corruption

If state file is corrupted:
- Attempt to reconstruct from worktrees and git log
- If not possible, report error and stop

## Output Format

### Status Line (Minimal Mode)

```
[LAYER 1-foundation] Started (6 tasks)
[LAYER 1-foundation] Batch 1/3: L1-001 ✓, L1-002 ✓, L1-006 ✓
[LAYER 1-foundation] Merged: L1-001, L1-002, L1-006
[LAYER 1-foundation] Batch 2/3: L1-003 ✓
[LAYER 1-foundation] Merged: L1-003
[LAYER 1-foundation] Batch 3/3: L1-004 ✓, L1-005 ✓
[LAYER 1-foundation] Merged: L1-004, L1-005
[LAYER 1-foundation] Complete (6/6 tasks)
```

### Final Result

End with:

```
LAYER_RESULT:
{json object}
```

The orchestrator parses this to decide next layer or stop.

## Dependency Graph Format

From `layer_plan.json`:

```json
{
  "dependency_graph": {
    "L0-001": [],
    "L0-002": ["L0-001"],
    "L0-003": ["L0-002"],
    "L0-004": ["L0-003"],
    "L1-001": ["L0-004"],
    "L1-002": ["L1-001"],
    "L1-003": ["L1-002"],
    "L1-004": ["L1-003"],
    "L1-005": ["L1-003"],
    "L1-006": ["L0-004"]
  }
}
```

Reading: `L1-003` depends on `L1-002`. L1-003 cannot start until L1-002 is complete.
