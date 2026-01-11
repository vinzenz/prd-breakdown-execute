# /breakdown Skill Reference

Convert PRD to self-contained implementation tasks.

---

## Overview

The `/breakdown` skill analyzes your PRD and generates implementation-ready task files organized into 5 architectural layers.

```bash
claude /breakdown docs/prd/my-project/index.md
```

---

## Arguments

| Argument | Description |
|----------|-------------|
| `<prd-path>` | Path to PRD index.md file (required) |
| `--output-dir <path>` | Output directory for greenfield projects |
| `--project-path <path>` | Existing project path for brownfield |
| `--layer <n>` | Only generate specific layer |
| `--skip-analysis` | Skip analysis phase, use existing analysis.json |
| `--review-only` | Only run review on existing tasks |

---

## The Workflow

```
PRD index.md
     │
     ▼
┌─────────────────────┐
│  breakdown-         │
│  analyze-prd        │───────► analysis.json
│                     │
│  Extracts:          │
│  - Features         │
│  - Inferred models  │
│  - Inferred APIs    │
│  - Template         │
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│  breakdown-         │
│  plan-layers        │───────► layer_plan.json
│                     │
│  Creates:           │
│  - Layer assignments│
│  - Dependency graph │
│  - Task ordering    │
└─────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│  For each layer (0-4):                                  │
│                                                         │
│  ┌─────────────────────┐                                │
│  │ breakdown-          │                                │
│  │ generate-tasks      │────► L{n}-*.xml files          │
│  └─────────────────────┘                                │
│           │                                             │
│           ▼                                             │
│  ┌─────────────────────┐                                │
│  │ breakdown-          │                                │
│  │ review-tasks        │────► PASS or FAIL (retry)      │
│  └─────────────────────┘                                │
│                                                         │
└─────────────────────────────────────────────────────────┘
     │
     ▼
manifest.json + task files
```

---

## Output Structure

```
docs/tasks/{project-slug}/
├── analysis.json           # PRD analysis results
├── layer_plan.json         # Layer organization
├── manifest.json           # Final task inventory
│
├── 0-setup/                # Layer 0 (greenfield only)
│   ├── L0-001-*.xml
│   ├── L0-002-*.xml
│   └── .done               # Layer completion marker
│
├── 1-foundation/           # Layer 1
│   ├── L1-001-*.xml
│   ├── L1-002-*.xml
│   └── .done
│
├── 2-backend/              # Layer 2
│   └── L2-*.xml
│
├── 3-frontend/             # Layer 3
│   └── L3-*.xml
│
└── 4-integration/          # Layer 4
    └── L4-*.xml
```

---

## Sub-Skills

The breakdown skill uses 4 sub-skills, all running in forked context:

| Skill | Model | Purpose |
|-------|-------|---------|
| `breakdown-analyze-prd` | sonnet | Extract features, infer models/APIs |
| `breakdown-plan-layers` | sonnet | Organize into layers, build dependencies |
| `breakdown-generate-tasks` | sonnet | Create self-contained task XML |
| `breakdown-review-tasks` | haiku | Validate completeness and quality |

[Sub-skill details](sub-skills.md)

---

## The 5 Layers

Tasks are organized into 5 architectural layers:

```
Layer 4: Integration     ◄── E2E flows, final wiring
         ▲
         │
Layer 3: Frontend        ◄── UI components, state management
         ▲
         │ (depends on contracts only)
         │
Layer 2: Backend         ◄── APIs, services, business logic
         ▲
         │
Layer 1: Foundation      ◄── Models, migrations, configuration
         ▲
         │
Layer 0: Setup           ◄── Project initialization (greenfield)
```

[Layer details](layers.md)

---

## Task Generation

Each task is a self-contained XML file with:

1. **Meta** - ID, name, layer, priority
2. **Context** - PRD excerpt, tech stack
3. **Dependencies** - Interface contracts from previous tasks
4. **Objective** - What to achieve
5. **Requirements** - Detailed specifications
6. **Test Requirements** - TDD test cases
7. **Files to Create** - Max 3 files per task
8. **Verification** - Runnable commands
9. **Exports** - Interface contracts for downstream

[Task format specification](task-format.md)

---

## Review Process

Every batch of generated tasks is reviewed:

### Critical Criteria (Must Pass)

- Completeness: No empty sections
- Self-containment: All context inline
- Interface contracts: Full type annotations
- Specificity: No vague language
- Test requirements: Concrete values
- Verification: Runnable commands
- File scope: Max 3 files

### Retry Logic

- On failure, specific feedback provided
- Generator retries with corrections
- Maximum 3 attempts per batch
- Failure after 3 attempts stops layer generation

---

## Greenfield vs Brownfield

### Greenfield (New Project)

```bash
claude /breakdown docs/prd/new-app/index.md \
  --output-dir /path/to/new-project
```

- Includes Layer 0 (setup tasks)
- Creates project from template
- Template detected from tech stack

### Brownfield (Existing Project)

```bash
claude /breakdown docs/prd/new-feature/index.md \
  --project-path /path/to/existing-project
```

- Skips Layer 0
- Integrates with existing structure
- Uses existing conventions

---

## Template Detection

Templates are detected from tech stack keywords:

| Stack | Template |
|-------|----------|
| Python + FastAPI | `webapps/backends/python` |
| Go + Chi | `webapps/backends/go` |
| TanStack Start | `webapps/backends/tanstack` |

---

## Common Patterns

### API-Only Project (No Frontend)

Layer 3 will be empty, with frontend tasks moved to Layer 4 integration if needed.

### Library Project

May only need Layers 1 and 2, with minimal integration in Layer 4.

### Full-Stack Project

All 5 layers populated with appropriate tasks.

---

## Troubleshooting

### Review keeps failing

1. Check PRD for vague descriptions
2. Add concrete acceptance criteria
3. Specify exact field names and types

### Wrong template detected

Edit `analysis.json` and re-run with `--skip-analysis`.

### Too many tasks in one layer

Tasks are batched (max 5 per batch) with separate review cycles.

---

## Next Steps

- [Layer Architecture](layers.md)
- [Task Format](task-format.md)
- [Sub-skills Reference](sub-skills.md)
- [Execute Tasks](../execute/README.md)
