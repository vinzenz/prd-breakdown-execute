---
name: execute
description: Main entry point for hierarchical task execution. Orchestrates layer-by-layer implementation of PRD tasks with parallel worktree execution.
context: fork
allowed-tools: Read Write Bash Skill Glob Task
model: claude-sonnet-4-5
user-invocable: true
---

# Task Execution Orchestrator

You are the main orchestrator for executing PRD implementation tasks. You coordinate layer-by-layer execution through a 4-level hierarchy:

```
/execute (you)
    └─► /execute-layer (per layer)
            └─► /execute-batch (per batch)
                    ├─► /execute-task (per task, parallel)
                    │       └─► /execute-verify (verification)
                    └─► /execute-merge (sequential merges)
```

## Arguments

See `.claude/skills/execute/references/options.md` for complete documentation.

### Required

- `<tasks-path>`: Path to tasks directory (contains manifest.json, layer_plan.json)

### Common Options

```
--project-path <path>   Target project (optional if in manifest)
--worktree-dir <path>   Worktree directory (default: {project}/../.worktrees)
--max-parallel <N>      Max concurrent tasks (default: 3)
--layer <name>          Execute specific layer only
--task <id>             Execute specific task only
--resume                Resume from existing state
--reset                 Delete state and start fresh
--dry-run               Show plan without executing
```

## Execution Flow

### Step 1: Parse Arguments

Extract all arguments from the prompt:

```python
tasks_path = required
project_path = optional  # from args or manifest
worktree_dir = optional  # default derived from project_path
max_parallel = 3
layer_filter = None
task_filter = None
resume = False
reset = False
dry_run = False
```

### Step 2: Load Manifest

Read `manifest.json`:

```bash
cat {tasks_path}/manifest.json
```

Extract:
- `prd.slug`: PRD identifier
- `prd.project_path`: Default project path (if not specified in args)
- `layers`: Layer definitions
- `summary.total_tasks`: Total task count

**Project path resolution:**
1. Use `--project-path` if provided
2. Fall back to `manifest.prd.project_path` if exists
3. Error if neither available

### Step 3: Validate Inputs

Check:
- Tasks path exists
- `manifest.json` exists
- `layer_plan.json` exists
- Project path exists (or will be created for greenfield)
- Git repository initialized in project

```bash
test -d {tasks_path}
test -f {tasks_path}/manifest.json
test -f {tasks_path}/layer_plan.json
test -d {project_path}
test -d {project_path}/.git
```

### Step 4: Handle State

**If `--reset`:**
```bash
rm -f {tasks_path}/execute-state.json
```

**If `--resume`:**
```python
if state_file_exists:
    state = load_state()
    validate_state_schema()
else:
    error("No state file to resume from")
```

**If new execution:**
```python
if state_file_exists:
    ask_user("State file exists. Resume or start fresh?")
else:
    state = init_state()
```

### Step 5: Dry Run (if requested)

If `--dry-run`:

```
Execution Plan: {prd_slug}
Project: {project_path}
Worktrees: {worktree_dir}

Layer 0-setup (4 tasks):
  Batch 1: L0-001 → L0-002 → L0-003 → L0-004 (sequential)

Layer 1-foundation (6 tasks):
  Batch 1: L1-001, L1-002, L1-006 (parallel, 3 tasks)
  Batch 2: L1-003 (depends on L1-002)
  Batch 3: L1-004, L1-005 (depends on L1-003)

Layer 2-backend (9 tasks):
  Batch 1: L2-001, L2-002, L2-003 (parallel)
  ...

Total: 48 tasks
Estimated batches: 15
Max parallelism: 3
```

Exit after dry run output.

### Step 6: Execute Layers

For each layer in order:

```python
layers = ["0-setup", "1-foundation", "2-backend", "3-frontend", "4-integration"]

for layer in layers:
    # Skip if filter doesn't match
    if layer_filter and layer != layer_filter:
        continue

    # Skip if already completed
    if state["layers"].get(layer, {}).get("status") == "completed":
        print(f"[EXEC] Skipping {layer} (already completed)")
        continue

    # Execute layer
    print(f"[EXEC] Starting {layer}")
    result = invoke_layer(layer)

    # Check for stop condition
    if result["should_stop"]:
        print(f"[EXEC] STOPPED: {result['stop_reason']}")
        update_state(status="stopped")
        report_final_status()
        return

    print(f"[EXEC] {layer} complete ({result['tasks_completed']}/{result['tasks_total']})")
```

### Step 7: Invoke Layer Agent

For each layer, call `/execute-layer`:

```
/execute-layer --tasks-path {tasks_path} --layer {layer} --project-path {project_path} --worktree-dir {worktree_dir} --max-parallel {max_parallel}
```

Wait for layer completion and parse `LAYER_RESULT`.

### Step 8: Handle Stop Condition

If any layer returns `should_stop: true`:

- A task was abandoned (5 failed attempts)
- Update state to `stopped`
- Output clear message with:
  - Which task failed
  - Path to preserved worktree
  - How to resume

```
STOPPED: Task L2-006 abandoned after 5 attempts

Completed: 15/48 tasks
Abandoned: L2-006
Blocked: 32 tasks (dependencies not met)

Worktree preserved: {worktree_dir}/L2-006
Review errors and fix issues, then run:
  /execute {tasks_path} --resume
```

### Step 9: Report Final Status

On completion:

```
Execution Complete: {prd_slug}

Layers:
  0-setup:      4/4 completed
  1-foundation: 6/6 completed
  2-backend:    9/9 completed
  3-frontend:   13/13 completed
  4-integration: 12/12 completed

Total: 44/44 tasks completed
Duration: 2h 15m
Retries: 3 (all succeeded)
```

### Step 10: Finalize Context (CRD Projects)

If PROJECT.md exists in the project root, update it with implemented features.

**Check for PROJECT.md:**
```bash
test -f {project_path}/PROJECT.md
```

**If exists, update context:**

1. Read all completed task XML files
2. Extract `<exports>` sections from each task
3. Map exports to PROJECT.md sections:
   - `<api endpoint>` → `<api-registry>`
   - `<interface type="react-component">` → `<features>`
   - `<interface type="sqlalchemy-model">` → `<schema-registry>`

4. Update PROJECT.md:
   - Add new features to `<features>` section
   - Add new endpoints to `<api-registry>` section
   - Add new models to `<schema-registry>` section
   - Update `<last-context-hash>` to current HEAD
   - Update `<last-updated>` timestamp

5. Commit the context update:
```bash
git -C {project_path} add PROJECT.md
git -C {project_path} commit -m "docs: Update PROJECT.md with features from {prd_slug}"
```

6. Update state with context finalization:
```json
{
  "context_update": {
    "status": "completed",
    "project_md_path": "{project_path}/PROJECT.md",
    "features_added": ["feature-1", "feature-2"],
    "endpoints_added": ["/api/new-endpoint"],
    "models_added": ["NewModel"]
  }
}
```

**If no PROJECT.md:**
Skip context finalization silently (not a CRD-based project).

### Step 11: Complete

Update state:
```json
{
  "status": "completed",
  "completed_at": "{now}"
}
```

Output final summary including context update if performed:

```
Execution Complete: {prd_slug}

Total: 44/44 tasks completed
Duration: 2h 15m

Context Update:
  - PROJECT.md updated at {project_path}/PROJECT.md
  - Features added: 3
  - Endpoints added: 5
  - Models added: 2
  - New context hash: {hash}
```

## State Initialization

```python
def init_state(prd_slug, project_path, worktree_dir, tasks_path, options):
    return {
        "schema_version": "2.0",
        "prd_slug": prd_slug,
        "project_path": project_path,
        "worktree_dir": worktree_dir,
        "tasks_path": tasks_path,
        "started_at": now(),
        "updated_at": now(),
        "completed_at": None,
        "status": "in_progress",
        "current_layer": None,
        "current_batch": None,
        "options": options,
        "layers": {},
        "tasks": {},
        "worktrees": {},
        "merge_queue": [],
        "completed": [],
        "failed": [],
        "abandoned": [],
        "metrics": {
            "tasks_total": total_tasks,
            "tasks_completed": 0,
            "tasks_failed": 0,
            "tasks_abandoned": 0,
            "tasks_remaining": total_tasks,
            "total_attempts": 0,
            "total_retries": 0,
            "elapsed_seconds": 0
        }
    }
```

## Resume Behavior

On resume:

1. Load existing state
2. Find current layer (from `current_layer` or first incomplete)
3. Re-evaluate ready queue within layer
4. Continue execution from ready tasks
5. Retry `failed` tasks (if attempts < 5)
6. Skip `completed` and `abandoned` tasks

## Error Handling

### Missing Manifest

```
Error: manifest.json not found at {tasks_path}/manifest.json
Run /breakdown first to generate tasks.
```

### Missing Project

If project path doesn't exist and not greenfield:

```
Error: Project path does not exist: {project_path}
For greenfield projects, Layer 0 will create it.
```

### Git Not Initialized

```
Error: Git repository not initialized at {project_path}
Run: cd {project_path} && git init
```

### Resume Without State

```
Error: No state file found at {tasks_path}/execute-state.json
Cannot resume. Start fresh without --resume flag.
```

## Output Format

### Minimal Mode (default)

```
[EXEC] Starting layer 0-setup (4 tasks)
[EXEC] Layer 0-setup complete (4/4)
[EXEC] Starting layer 1-foundation (6 tasks)
[EXEC] Layer 1-foundation complete (6/6)
...
[EXEC] All layers complete (44/44 tasks)
```

### Verbose Mode

```
[EXEC] Execution Plan:
  PRD: voice-prd-generator
  Project: /home/user/projects/voice-prd
  Worktrees: /home/user/projects/.worktrees
  Max parallel: 3

[EXEC] Starting layer 0-setup (4 tasks)
  [LAYER 0-setup] Batch 1/1: L0-001, L0-002, L0-003, L0-004
    [L0-001] Creating worktree...
    [L0-001] Implementing...
    [L0-001] Verified (3/3 steps)
    ...
  [LAYER 0-setup] Merged: L0-001, L0-002, L0-003, L0-004
[EXEC] Layer 0-setup complete (4/4)
...
```

## Context Isolation

This skill runs in `context: fork`:
- Fresh context for each execution
- No context bleed from previous runs
- Spawns child skills which also fork
- State file is the persistence mechanism

## Critical Rules

1. **Never skip layers**: Execute in order (0→1→2→3→4)
2. **Respect dependencies**: Only execute tasks with satisfied deps
3. **Stop on abandon**: If task hits 5 failures, STOP immediately
4. **Preserve worktrees**: Never delete worktrees on failure
5. **Sequential merges**: Merge one task at a time to avoid conflicts
6. **Update state**: Write state after every significant event
