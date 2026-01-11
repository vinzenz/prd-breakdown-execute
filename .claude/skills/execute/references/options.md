# Execute Command Options

## Usage

```
/execute <tasks-path> [options]
```

## Required Arguments

### `<tasks-path>`

Path to the tasks directory containing:
- `manifest.json` - PRD metadata and task list
- `layer_plan.json` - Dependency graph
- Layer subdirectories with task XML files

Example: `docs/tasks/voice-prd-generator`

## Target Directory Options

### `--project-path <path>`

Target project directory where code will be implemented.

- **Optional** if `manifest.json` contains `project_path`
- **Required** if manifest doesn't specify a path
- **Overrides** manifest setting when provided

```bash
# Use manifest path
/execute docs/tasks/my-project

# Override manifest path
/execute docs/tasks/my-project --project-path /home/user/different-location
```

### `--worktree-dir <path>`

Directory where git worktrees are created.

- **Default**: `{project-path}/../.worktrees`
- Worktrees are named by task ID: `{worktree-dir}/L1-001`

```bash
# Custom worktree location
/execute docs/tasks/my-project --worktree-dir /tmp/worktrees
```

## Execution Control Options

### `--max-parallel <N>`

Maximum concurrent tasks per batch.

- **Default**: 3
- **Range**: 1-10
- Higher values = faster execution, more resource usage

```bash
# Run 5 tasks in parallel
/execute docs/tasks/my-project --max-parallel 5

# Sequential execution (one at a time)
/execute docs/tasks/my-project --max-parallel 1
```

### `--layer <name>`

Execute only the specified layer.

- Skips all other layers
- Useful for targeted execution or debugging

Valid layer names:
- `0-setup`
- `1-foundation`
- `2-backend`
- `3-frontend`
- `4-integration`

```bash
# Only execute backend layer
/execute docs/tasks/my-project --layer 2-backend
```

### `--task <id>`

Execute only the specified task.

- Skips all other tasks
- Useful for retrying a specific task
- Task dependencies must already be completed

```bash
# Only execute task L2-003
/execute docs/tasks/my-project --task L2-003
```

### `--start-layer <name>`

Start execution from the specified layer.

- Skips earlier layers (assumes they're complete)
- Useful when resuming after manual intervention

```bash
# Skip setup and foundation, start at backend
/execute docs/tasks/my-project --start-layer 2-backend
```

## Resume & Recovery Options

### `--resume`

Resume from existing state file.

- Loads `execute-state.json` from tasks directory
- Re-evaluates ready queue
- Continues from where execution stopped
- Retries failed tasks (if attempts < 5)

```bash
# Resume after interruption or failure
/execute docs/tasks/my-project --resume
```

### `--reset`

Delete state file and start fresh.

- Removes `execute-state.json`
- Does NOT remove worktrees (clean them manually if needed)
- Starts execution from beginning

```bash
# Start over from scratch
/execute docs/tasks/my-project --reset
```

### `--reset-task <id>`

Reset a specific task to pending status.

- Sets task status back to `pending`
- Clears error history and retry feedback
- Keeps worktree intact (if exists)
- Combine with `--resume` to re-execute

```bash
# Retry a specific task after fixing manually
/execute docs/tasks/my-project --reset-task L2-003 --resume
```

## Output Control Options

### `--dry-run`

Show execution plan without implementing.

- Lists all tasks by layer
- Shows dependency order
- Shows which tasks can run in parallel
- Does NOT modify any files or create state

```bash
# Preview execution plan
/execute docs/tasks/my-project --dry-run
```

Output example:
```
Execution Plan: voice-prd-generator

Layer 0-setup (4 tasks):
  Batch 1: L0-001, L0-002, L0-003, L0-004 (sequential - dependencies)

Layer 1-foundation (6 tasks):
  Batch 1: L1-001, L1-002, L1-006 (parallel - no deps)
  Batch 2: L1-003 (depends on L1-002)
  Batch 3: L1-004, L1-005 (depends on L1-003)

...

Total: 48 tasks across 5 layers
Estimated batches: 15
```

### `--verbose`

Show detailed per-task output.

- Shows each verification step result
- Shows file changes
- Shows commit hashes
- Useful for debugging

```bash
/execute docs/tasks/my-project --verbose
```

### `--quiet`

Minimal output - only errors and final summary.

- Suppresses per-task status lines
- Only shows failures and completion
- Useful for automated pipelines

```bash
/execute docs/tasks/my-project --quiet
```

## Git Options

### `--no-commits`

Skip git commits during implementation.

- Tasks still run in worktrees
- No commits created
- Useful for testing/debugging
- **Warning**: Resume won't work properly without commits

```bash
# Test without committing
/execute docs/tasks/my-project --no-commits
```

### `--commit-prefix <text>`

Add prefix to all commit messages.

- Prepended before task ID
- Useful for identifying automated commits

```bash
/execute docs/tasks/my-project --commit-prefix "[AUTO]"

# Commits will look like:
# [AUTO][L1-001] Create enums and constants
```

### `--author <name>`

Git author for commits.

- **Default**: Uses git config user.name and user.email
- Override for specific attribution

```bash
/execute docs/tasks/my-project --author "Bot <bot@example.com>"
```

## Option Combinations

### Common Patterns

```bash
# Normal execution
/execute docs/tasks/my-project

# Resume after failure
/execute docs/tasks/my-project --resume

# Retry specific failed task
/execute docs/tasks/my-project --reset-task L2-003 --resume

# Execute single layer with more parallelism
/execute docs/tasks/my-project --layer 2-backend --max-parallel 5

# Debug mode - verbose, no commits
/execute docs/tasks/my-project --verbose --no-commits

# CI/CD mode - quiet, custom author
/execute docs/tasks/my-project --quiet --author "CI <ci@example.com>"

# Preview before running
/execute docs/tasks/my-project --dry-run
```

### Invalid Combinations

```bash
# Error: Can't reset and resume at same time
/execute docs/tasks/my-project --reset --resume

# Error: --quiet and --verbose are mutually exclusive
/execute docs/tasks/my-project --quiet --verbose
```

## Environment Variables

The following environment variables can provide defaults:

| Variable | Option | Description |
|----------|--------|-------------|
| `EXECUTE_MAX_PARALLEL` | `--max-parallel` | Default parallelism |
| `EXECUTE_WORKTREE_DIR` | `--worktree-dir` | Default worktree location |
| `EXECUTE_VERBOSE` | `--verbose` | Enable verbose by default |

CLI options always override environment variables.
