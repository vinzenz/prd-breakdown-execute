# Breakdown Sub-Skills

Reference for the component skills used by `/breakdown`.

---

## Overview

The breakdown skill orchestrates 4 sub-skills:

```
/breakdown (orchestrator)
     │
     ├──► breakdown-analyze-prd
     │
     ├──► breakdown-plan-layers
     │
     ├──► breakdown-generate-tasks (per layer)
     │
     └──► breakdown-review-tasks (per batch)
```

All sub-skills run with `context: fork` for isolation.

---

## breakdown-analyze-prd

**Purpose:** Extract structured information from the PRD.

**Context:** fork
**Model:** sonnet

### Input

- PRD XML content from `index.md`

### Process

1. Parse PRD structure
2. Extract features with priorities
3. Infer data models from descriptions
4. Infer API endpoints from features
5. Infer frontend components
6. Detect template from tech stack
7. Identify external dependencies

### Output

`analysis.json`:

```json
{
  "prd_slug": "slack-task-manager",
  "project_type": "greenfield",
  "template": "webapps/backends/python",

  "features": [
    {
      "slug": "task-crud",
      "name": "Task CRUD Operations",
      "priority": "must-have",
      "acceptance_criteria": [
        {
          "given": "A user is logged in",
          "when": "They create a task",
          "then": "The task appears in their list"
        }
      ]
    }
  ],

  "inferred_models": [
    {
      "name": "Task",
      "fields": [
        {"name": "id", "type": "UUID", "primary_key": true},
        {"name": "title", "type": "str", "max_length": 200},
        {"name": "status", "type": "TaskStatus", "enum": true}
      ],
      "relationships": [
        {"name": "assignee", "target": "User", "type": "many-to-one"}
      ]
    }
  ],

  "inferred_endpoints": [
    {"method": "GET", "path": "/api/tasks", "purpose": "List tasks"},
    {"method": "POST", "path": "/api/tasks", "purpose": "Create task"}
  ],

  "inferred_components": [
    {"name": "TaskList", "type": "component", "layer": 3},
    {"name": "Dashboard", "type": "page", "layer": 3}
  ],

  "dependencies": [
    {"name": "slack-sdk", "version": "^3.0", "purpose": "Slack API"}
  ]
}
```

### Inference Logic

**Models:** Extracted from feature descriptions mentioning data entities.

**Endpoints:** Inferred from CRUD operations mentioned in features.

**Components:** Identified from UI-related acceptance criteria.

---

## breakdown-plan-layers

**Purpose:** Organize extracted elements into 5 architectural layers.

**Context:** fork
**Model:** sonnet

### Input

- `analysis.json` from previous phase

### Process

1. Assign each model to Layer 1
2. Assign each endpoint to Layer 2
3. Assign each component to Layer 3
4. Identify integration needs for Layer 4
5. Build dependency graph within layers
6. Order tasks by dependencies
7. Split large features (max 3 files per task)

### Output

`layer_plan.json`:

```json
{
  "project_type": "greenfield",
  "include_layer_0": true,

  "layers": {
    "0-setup": {
      "tasks": [
        {
          "id": "L0-001",
          "name": "Project initialization",
          "files": ["pyproject.toml", "src/__init__.py"]
        }
      ],
      "dependencies": {}
    },

    "1-foundation": {
      "tasks": [
        {"id": "L1-001", "name": "Task model", "files": ["src/models/task.py"]},
        {"id": "L1-002", "name": "User model", "files": ["src/models/user.py"]},
        {"id": "L1-003", "name": "Migrations", "files": ["alembic/versions/001.py"]}
      ],
      "dependencies": {
        "L1-003": ["L1-001", "L1-002"]
      }
    },

    "2-backend": {
      "tasks": [
        {"id": "L2-001", "name": "Task repository"},
        {"id": "L2-002", "name": "Task service"},
        {"id": "L2-003", "name": "Task API"}
      ],
      "dependencies": {
        "L2-002": ["L2-001"],
        "L2-003": ["L2-001", "L2-002"]
      }
    }
  },

  "dependency_graph": {
    "L1-003": ["L1-001", "L1-002"],
    "L2-002": ["L2-001"],
    "L2-003": ["L2-001", "L2-002"]
  }
}
```

### Task Splitting

Features requiring many files are split:

```
Feature: User authentication
  │
  ├──► L2-001: User repository (2 files)
  ├──► L2-002: Auth service (2 files)
  ├──► L2-003: Auth endpoints (2 files)
  └──► L2-004: Auth middleware (1 file)
```

Each task: max 3 files (excluding tests).

---

## breakdown-generate-tasks

**Purpose:** Create self-contained task XML files.

**Context:** fork
**Model:** sonnet

### Input

- `analysis.json`
- `layer_plan.json`
- Layer to generate (e.g., "2-backend")

### Process

1. Load layer plan for target layer
2. For each task in layer:
   - Gather context from PRD
   - Collect dependency interfaces
   - Write detailed requirements
   - Define test cases
   - Specify verification steps
   - Define export interfaces
3. Write XML files to layer directory

### Output

Task XML files: `docs/tasks/{slug}/{layer}/L{n}-{seq}-{name}.xml`

### Batching

Large layers are processed in batches:

- Batch size: max 5 tasks
- Each batch reviewed separately
- Allows targeted retry on failure

```
Layer 2 with 12 tasks:
  Batch 1: L2-001, L2-002, L2-003, L2-004, L2-005
  Batch 2: L2-006, L2-007, L2-008, L2-009, L2-010
  Batch 3: L2-011, L2-012
```

---

## breakdown-review-tasks

**Purpose:** Validate task files for completeness and quality.

**Context:** fork
**Model:** haiku (fast validation)

### Input

- List of task XML files to review

### Process

For each task, check:

1. **Completeness**
   - All required sections present
   - No empty sections
   - No placeholder content

2. **Self-containment**
   - All context inline
   - All imports in interface blocks
   - No references to external files

3. **Interface contracts**
   - Every interface has name and type
   - Complete type annotations
   - All imports included

4. **Requirements specificity**
   - Exact file paths
   - Exact class/function names
   - No vague language

5. **Test requirements**
   - Test file path specified
   - Concrete test values
   - Clear assertions

6. **Verification**
   - Runnable commands
   - Expected outcomes stated

7. **File scope**
   - Max 3 files per task
   - Full paths from root

### Output

```json
{
  "verdict": "PASSED" | "FAILED",
  "tasks": [
    {
      "id": "L2-001",
      "status": "PASSED",
      "issues": []
    },
    {
      "id": "L2-002",
      "status": "FAILED",
      "issues": [
        {
          "severity": "critical",
          "section": "requirements",
          "message": "R1 uses placeholder 'appropriate validation'"
        }
      ]
    }
  ],
  "summary": "1/2 tasks passed",
  "feedback": "L2-002: Replace 'appropriate validation' with specific rules"
}
```

### Retry Flow

```
Generate batch → Review → FAILED
                   │
                   ▼
              Retry with feedback (attempt 2)
                   │
                   ▼
              Review → PASSED
```

Maximum 3 attempts per batch.

---

## Skill Coordination

### Data Flow

```
PRD index.md
     │
     ▼
breakdown-analyze-prd ──────► analysis.json
     │
     │ (passed to next skill)
     ▼
breakdown-plan-layers ──────► layer_plan.json
     │
     │ (per layer)
     ▼
breakdown-generate-tasks ───► L{n}-*.xml files
     │
     │ (per batch)
     ▼
breakdown-review-tasks ─────► PASS/FAIL
     │
     │ (on failure)
     ▼
breakdown-generate-tasks ───► (retry with feedback)
```

### State Markers

Each layer gets a `.done` marker when complete:

```
docs/tasks/my-project/
├── 0-setup/
│   ├── L0-001-*.xml
│   └── .done          ◄── Layer 0 complete
├── 1-foundation/
│   └── L1-001-*.xml   ◄── No .done yet
```

This enables resuming interrupted breakdown.

---

## Next Steps

- [Task Format Specification](task-format.md)
- [Execute Skill Reference](../execute/README.md)
- [Back to Breakdown Overview](README.md)
