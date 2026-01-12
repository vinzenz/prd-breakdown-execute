# Commands Reference

Complete reference for all commands and arguments.

---

## /prd

Interactive PRD creation command.

### Usage

```bash
claude /prd [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--resume` | Resume an incomplete PRD |
| `--output-dir <path>` | Custom output directory (default: `docs/prd/`) |

### Examples

```bash
# Start new PRD
claude /prd

# Resume incomplete PRD
claude /prd --resume

# Custom output location
claude /prd --output-dir ./requirements/
```

### Output

```
docs/prd/{project-slug}/
├── index.md          # Main PRD
├── what-next.md      # Progress tracking
└── features/
    └── *.md          # Feature details
```

---

## /crd

Create Change Request Documents for existing codebases.

### Usage

```bash
claude /crd --project <path> [description] [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--project <path>` | Yes | Path to existing project |
| Description text | No | Initial change description |

### Options

| Option | Description |
|--------|-------------|
| `--list` | List existing CRDs for the project |
| `--status <slug>` | Check status of a specific CRD |

### Examples

```bash
# Create CRD with description
claude /crd --project /path/to/my-app "Add dark mode toggle"
claude /crd -p /path/to/my-app "Remove deprecated API"

# List existing CRDs
claude /crd -p /path/to/my-app --list

# Check CRD status
claude /crd -p /path/to/my-app --status dark-mode-toggle
```

### Output

```
{project}/
├── PROJECT.md           # Codebase context (generated)
└── docs/crd/
    └── {slug}.md        # CRD document
```

---

## /crd-context

Manage PROJECT.md context without creating a CRD.

### Usage

```bash
claude /crd-context --project <path> [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--full` | Force full re-investigation |
| `--check` | Check if context is stale |
| `--diff` | Show changes since last update |

### Examples

```bash
# Generate or update PROJECT.md
claude /crd-context --project /path/to/my-app

# Force full re-scan
claude /crd-context -p /path/to/my-app --full

# Check staleness
claude /crd-context -p /path/to/my-app --check

# Show changes
claude /crd-context -p /path/to/my-app --diff
```

---

## /breakdown

Convert PRD or CRD to implementation tasks.

### Usage

```bash
claude /breakdown <input-path> [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `<input-path>` | Yes | Path to PRD index.md or CRD document |

### Options

| Option | Description |
|--------|-------------|
| `--output-dir <path>` | Output for greenfield projects |
| `--project-path <path>` | Existing project for brownfield/CRD |
| `--layer <n>` | Generate specific layer only |
| `--skip-analysis` | Use existing analysis.json |
| `--review-only` | Only review, don't regenerate |
| `--auto-setup` | Auto-execute Layer 0 (greenfield only) |

### Examples

```bash
# PRD breakdown (greenfield)
claude /breakdown docs/prd/my-project/index.md

# PRD with output directory
claude /breakdown docs/prd/my-project/index.md \
  --output-dir /path/to/new-project

# PRD brownfield integration
claude /breakdown docs/prd/new-feature/index.md \
  --project-path /path/to/existing-project

# CRD breakdown (requires --project-path)
claude /breakdown docs/crd/dark-mode-toggle.md \
  --project-path /path/to/existing-project

# Regenerate specific layer
claude /breakdown docs/prd/my-project/index.md --layer 2

# Skip analysis phase
claude /breakdown docs/prd/my-project/index.md --skip-analysis
```

### Output (PRD)

```
docs/tasks/{project-slug}/
├── analysis.json
├── layer_plan.json
├── manifest.json
├── 0-setup/              # Layer 0 (greenfield only)
│   └── L0-*.xml
├── 1-foundation/
│   └── L1-*.xml
├── 2-backend/
│   └── L2-*.xml
├── 3-frontend/
│   └── L3-*.xml
└── 4-integration/
    └── L4-*.xml
```

### Output (CRD)

```
docs/tasks/{crd-slug}/
├── analysis.json
├── layer_plan.json
├── 2-backend/            # Only affected layers
│   └── L2-*.xml
├── 3-frontend/
│   └── L3-*.xml
└── 4-integration/
    └── L4-*.xml
```

---

## /execute

Execute implementation tasks.

### Usage

```bash
claude /execute <tasks-path> [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `<tasks-path>` | Yes | Path to tasks directory |

### Options

| Option | Description |
|--------|-------------|
| `--resume` | Resume from previous state |
| `--max-parallel <n>` | Max concurrent tasks (default: 3) |
| `--layer <name>` | Execute specific layer only |
| `--dry-run` | Preview without executing |
| `--reset-task <id>` | Reset task for retry |
| `--verbose` | Detailed output |
| `--quiet` | Minimal output (CI mode) |

### Examples

```bash
# Basic execution
claude /execute docs/tasks/my-project

# Resume after interruption
claude /execute docs/tasks/my-project --resume

# Increase parallelism
claude /execute docs/tasks/my-project --max-parallel 5

# Execute single layer
claude /execute docs/tasks/my-project --layer 2-backend

# Preview execution plan
claude /execute docs/tasks/my-project --dry-run

# Reset and retry specific task
claude /execute docs/tasks/my-project --reset-task L2-003 --resume

# Verbose output
claude /execute docs/tasks/my-project --verbose

# CI/CD mode
claude /execute docs/tasks/my-project --quiet
```

### Output

Creates/updates:
- `execute-state.json` in tasks directory
- `.worktrees/` directory in project
- Git commits and branches
- Implementation code
- `PROJECT.md` updated (CRD workflow only)

---

## Common Patterns

### Greenfield Workflow (PRD)

```bash
# 1. Create PRD
claude /prd

# 2. Break down
claude /breakdown docs/prd/my-project/index.md

# 3. Execute
claude /execute docs/tasks/my-project
```

### Brownfield Workflow (CRD)

```bash
# 1. Create CRD (auto-generates PROJECT.md)
claude /crd -p /path/to/existing-app "Add dark mode"

# 2. Break down CRD
claude /breakdown docs/crd/dark-mode.md -p /path/to/existing-app

# 3. Execute (auto-updates PROJECT.md after)
claude /execute docs/tasks/dark-mode -p /path/to/existing-app
```

### Resume After Failure

```bash
# Check what failed
cat docs/tasks/my-project/execute-state.json | jq '.tasks | to_entries[] | select(.value.status == "failed")'

# Fix issue manually in worktree
cd .worktrees/L2-003
# ... fix code ...
git add . && git commit -m "Manual fix"

# Resume
claude /execute docs/tasks/my-project --resume
```

### Reset Everything

```bash
# Remove state and worktrees
rm docs/tasks/my-project/execute-state.json
rm -rf .worktrees/
git worktree prune

# Start fresh
claude /execute docs/tasks/my-project
```

### Dry Run Then Execute

```bash
# Preview
claude /execute docs/tasks/my-project --dry-run

# If plan looks good
claude /execute docs/tasks/my-project
```

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_MODEL` | Override default model |

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error (check output) |

---

## Next Steps

- [File Formats](file-formats.md)
- [Troubleshooting](../troubleshooting.md)
- [Quick Start](../quickstart/README.md)
