---
layout: default
title: Commands
parent: Reference
nav_order: 1
---

# Command Reference

Complete syntax and options for all commands.

## /prd

Create a Product Requirements Document interactively.

### Syntax

```bash
claude /prd
```

### Phases

| Phase | Description |
|-------|-------------|
| 1. Idea | Describe your project |
| 2. Tech Stack | Choose framework |
| 3. Features | List main features |
| 4. Dependencies | External services |
| 5. Options | Configuration |
| 6. Review | Confirm details |
| 7. Feature Details | Deep dive each feature |
| 8. Output | Generate files |

### Output

```
docs/prd/{slug}/
├── index.md          # Main PRD
└── features/         # Feature specs
    ├── feature-1.md
    └── feature-2.md
```

---

## /crd

Create a Change Request Document for existing codebases.

### Syntax

```bash
claude /crd
```

### Workflow

1. Check/generate PROJECT.md
2. Capture change intent
3. Run impact analysis
4. Generate CRD document

### Output

```
docs/crd/{slug}/
└── index.xml         # CRD document
```

---

## /crd-context

Manage PROJECT.md independently.

### Syntax

```bash
claude /crd-context [action]
```

### Actions

| Action | Description |
|--------|-------------|
| `generate` | Create new PROJECT.md |
| `update` | Incremental update |
| `check` | Verify freshness |

### Output

```
PROJECT.md            # Codebase context
```

---

## /breakdown

Generate tasks from PRD or CRD.

### Syntax

```bash
claude /breakdown <path>
```

### Arguments

| Argument | Description |
|----------|-------------|
| `path` | Path to PRD or CRD directory |

### Examples

```bash
# Break down a PRD
claude /breakdown docs/prd/my-project/

# Break down a CRD
claude /breakdown docs/crd/my-change/
```

### Output

```
{path}/tasks/
├── 0-setup/
│   └── L0-001-*.xml
├── 1-foundation/
│   └── L1-*.xml
├── 2-backend/
│   └── L2-*.xml
├── analysis.json
├── layer_plan.json
└── manifest.json
```

---

## /execute

Run tasks in parallel with TDD.

### Syntax

```bash
claude /execute <path> [options]
```

### Arguments

| Argument | Description |
|----------|-------------|
| `path` | Path to tasks directory |

### Options

| Option | Description |
|--------|-------------|
| `--layers=N,N` | Execute specific layers only |
| `--from=TASK-ID` | Resume from specific task |
| `--dry-run` | Validate without executing |

### Examples

```bash
# Execute all tasks
claude /execute docs/prd/my-project/tasks/

# Execute only layers 1 and 2
claude /execute docs/prd/my-project/tasks/ --layers=1,2

# Resume from a specific task
claude /execute docs/prd/my-project/tasks/ --from=L2-001-auth-api

# Dry run
claude /execute docs/prd/my-project/tasks/ --dry-run
```

### Output

- Working code in project directory
- Test files alongside implementation
- Updated `execute-state.json`
- For CRD: Updated `PROJECT.md`

---

## Common Patterns

### Full Greenfield Workflow

```bash
# 1. Create PRD
claude /prd

# 2. Break down
claude /breakdown docs/prd/my-project/

# 3. Execute
claude /execute docs/prd/my-project/tasks/
```

### Full Brownfield Workflow

```bash
# 1. Create CRD
claude /crd

# 2. Break down
claude /breakdown docs/crd/my-change/

# 3. Execute
claude /execute docs/crd/my-change/tasks/
```

### Resume After Failure

```bash
# Check state
cat docs/prd/my-project/tasks/execute-state.json

# Resume
claude /execute docs/prd/my-project/tasks/
```

### Re-run Specific Layers

```bash
# Just layer 2 (APIs)
claude /execute docs/prd/my-project/tasks/ --layers=2
```
