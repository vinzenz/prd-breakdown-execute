---
name: breakdown
description: Break down a PRD or CRD into self-contained implementation tasks for LLM execution. Use when you have a PRD/CRD file and want to generate executable task files for autonomous implementation.
context: fork
allowed-tools: Read Glob Grep Write Skill Bash
model: claude-sonnet-4-5
---

# /breakdown - PRD/CRD Task Breakdown

You are orchestrating the breakdown of a PRD (Product Requirements Document) or CRD (Change Request Document) into implementation tasks optimized for autonomous LLM execution.

## Arguments

- `<input-path>`: Path to PRD index.md or CRD document (required)
- `--layer <N>`: Only generate tasks for specific layer (optional, 0-4)
- `--review-only`: Only run review pass on existing tasks (optional)
- `--output-dir <path>`: Target directory for greenfield projects (overrides default)
- `--project-path <path>`: Existing project path for brownfield/CRD (overrides PRD value)
- `--auto-setup`: Automatically execute Layer 0 tasks after generation (greenfield only)

## Input Format Detection

The skill automatically detects whether the input is a PRD or CRD:

| Root Element | Format | Description |
|--------------|--------|-------------|
| `<prd>` | PRD | Full product requirements (greenfield or brownfield) |
| `<crd>` | CRD | Change request for existing project |

**CRD handling:**
- CRDs are always brownfield (no Layer 0)
- CRDs require `--project-path` or a project with PROJECT.md
- CRDs use `<impact-analysis>` to scope task generation
- CRDs typically produce fewer tasks (focused changes)

## Output Location

Tasks are written to: `docs/tasks/<prd-slug>/`

For greenfield with `--output-dir`:
- Layer 0 tasks reference the output-dir as target
- Other layer tasks are still saved to docs/tasks/

## Workflow

Execute these phases in order:

### Phase 1: Validate Input

1. Read the input file at the provided path
2. Detect format by checking root element (`<prd>` or `<crd>`)
3. Verify valid XML structure
4. Extract the slug from `<meta><slug>` tag
5. Create output directory: `docs/tasks/{slug}/`
6. If directory exists, check for existing `.done` markers to resume

**For CRD input:**
- Require `--project-path` argument (CRDs always target existing projects)
- Verify PROJECT.md exists at `{project-path}/PROJECT.md`
- Load PROJECT.md context for use in task generation

### Phase 2: Analyze Input

**For PRD:**
Invoke the `breakdown-analyze-prd` skill with the full PRD content.

Pass the PRD XML content and request structured extraction of:
- All features with priorities
- Tech stack with versions
- Implied data models
- Implied API endpoints
- Implied frontend components
- External dependencies
- Template path if specified

**For CRD:**
Extract directly from CRD structure:
- Requirements from `<requirements>` section
- Acceptance criteria from `<acceptance-criteria>` section
- Affected files from `<impact-analysis><affected-files>`
- Affected features from `<impact-analysis><affected-features>`
- Tech stack from PROJECT.md context
- Related existing features from `<context><related-features>`

The CRD already contains impact analysis, so less inference is needed.

Save the analysis to `docs/tasks/{slug}/analysis.json`

### Phase 3: Plan Layers

**For PRD:**
Invoke the `breakdown-plan-layers` skill with the analysis JSON.

Request organization into 4-5 layers:
1. **0-setup**: Template copy, git init, environment (greenfield only)
2. **1-foundation**: Database models, migrations, base config
3. **2-backend**: API endpoints, services, business logic
4. **3-frontend**: React components, state management, routing
5. **4-integration**: Wiring, E2E flows, polish

**For CRD:**
Layer planning is simpler based on `<impact-analysis>`:

- If `<affected-schemas>` has changes: Include Layer 1 (foundation)
- If `<affected-apis>` has changes: Include Layer 2 (backend)
- If frontend files in `<affected-files>`: Include Layer 3 (frontend)
- Always include Layer 4 (integration) for wiring changes together

CRD typically produces 2-3 layers, not 5.

Save the layer plan to `docs/tasks/{slug}/layer_plan.json`

### Phase 4: Generate Tasks (Per Layer)

Determine layers to process based on input type:
- **PRD Greenfield**: `[0-setup, 1-foundation, 2-backend, 3-frontend, 4-integration]`
- **PRD Brownfield**: `[1-foundation, 2-backend, 3-frontend, 4-integration]` (skip Layer 0)
- **CRD**: Only layers identified in Phase 3 based on impact analysis

For each layer in order:

1. **Check completion**: If `{layer}/.done` exists, skip this layer

2. **Create directory**: `docs/tasks/{slug}/{layer}/`

3. **Batch tasks**: Split layer tasks into batches of max 5 tasks each
   - If layer has ≤5 tasks: single batch
   - If layer has 6-10 tasks: 2 batches
   - If layer has 11-15 tasks: 3 batches
   - Example: 14 tasks → batches of [5, 5, 4]

4. **For each batch, with retry loop** (max 3 attempts per batch):
   ```
   for batch in batches:
       for attempt in [1, 2, 3]:
           # Generate
           invoke breakdown-generate-tasks with:
             - Layer name
             - Task batch (subset of layer tasks)
             - PRD analysis
             - Template path (if any)
             - Output directory: docs/tasks/{slug}/{layer}/
             - Previous review feedback (if retry)

           # Review
           invoke breakdown-review-tasks with:
             - Path to generated task files
             - Layer name

           # Handle result
           if review.verdict == "PASSED":
               break  # Move to next batch
           elif attempt < 3:
               # Extract failures for next attempt
               feedback = {
                   "attempt": attempt + 1,
                   "previous_failures": review.critical_issues
               }
           else:
               # Max retries exceeded
               Report: "Batch failed after 3 attempts. Manual intervention required."
               Report: review.critical_issues
               Exit without creating .done
   ```

5. **Mark layer complete**: After ALL batches pass, create `{layer}/.done` marker

### Phase 5: Finalize

1. Generate `docs/tasks/{slug}/manifest.json` containing:
   - PRD slug and name
   - Generation timestamp
   - Layer completion status
   - Task inventory with IDs and file paths
2. Report completion summary:
   - Total tasks generated
   - Tasks per layer
   - Any review failures requiring attention

## Task File Format

Each task file follows this XML structure (see `references/task-format-spec.md` for full spec):

```xml
<task>
  <meta>
    <id>L1-001</id>
    <name>Task Name</name>
    <layer>1-foundation</layer>
    <priority>1</priority>
  </meta>
  <context>...</context>
  <dependencies>...</dependencies>
  <objective>...</objective>
  <requirements>...</requirements>
  <test-requirements>...</test-requirements>
  <files-to-create>...</files-to-create>
  <verification>...</verification>
  <exports>...</exports>
</task>
```

## Critical Constraints

- **Self-contained tasks**: Each task file MUST contain ALL information needed for implementation. No external lookups.
- **Small context**: Tasks are designed for ~50k token context models (Haiku, GLM 4.5-4.7)
- **Interface contracts**: Dependencies use type signatures, not full code
- **Max 3 files per task**: Keep scope manageable
- **TDD approach**: Test requirements come before implementation
- **Explicit verification**: Every task has runnable verification commands

## Error Handling

- If PRD file not found: Report error, exit
- If PRD invalid XML: Report parsing error with details, exit
- If skill invocation fails: Report which phase failed, suggest retry
- If review fails: Do NOT mark layer complete, report specific issues

## Example Usage

### Greenfield Project
```
/breakdown docs/prd/voice-prd-generator/index.md --output-dir /path/to/new-project
```

Output:
```
Analyzing PRD: voice-prd-generator
Detected: greenfield project (Python + FastAPI template)
Target directory: /path/to/new-project
Saved analysis to: docs/tasks/voice-prd-generator/analysis.json

Planning layers...
Saved layer plan to: docs/tasks/voice-prd-generator/layer_plan.json

Generating Layer 0 (Setup)...
Batch 1/1: [L0-001, L0-002, L0-003, L0-004]
- L0-001-copy-template.xml
- L0-002-init-git-env.xml
- L0-003-configure-database.xml
- L0-004-verify-setup.xml
Reviewing batch... PASSED
Created: docs/tasks/voice-prd-generator/0-setup/.done

Generating Layer 1 (Foundation)...
Batch 1/1: [L1-001, L1-002, L1-003, L1-004, L1-005]
- L1-001-project-model.xml
- L1-002-conversation-model.xml
- L1-003-message-model.xml
- L1-004-prddocument-model.xml
- L1-005-personaworkflow-model.xml
Reviewing batch... PASSED
Created: docs/tasks/voice-prd-generator/1-foundation/.done

Generating Layer 2 (Backend)...
Batch 1/3: [L2-001, L2-002, L2-003, L2-004, L2-005]
Reviewing batch... PASSED
Batch 2/3: [L2-006, L2-007, L2-008, L2-009, L2-010]
Reviewing batch... FAILED (attempt 1)
  - L2-008: Contains placeholder 'TBD' for schema
Regenerating with feedback...
Reviewing batch... PASSED (attempt 2)
Batch 3/3: [L2-011, L2-012, L2-013, L2-014]
Reviewing batch... PASSED
Created: docs/tasks/voice-prd-generator/2-backend/.done

[continues for layers 3-4...]

Breakdown complete!
- Total tasks: 28
- 0-setup: 4 tasks
- 1-foundation: 5 tasks
- 2-backend: 14 tasks
- 3-frontend: 7 tasks
- 4-integration: 4 tasks
```

### Brownfield Project (PRD)
```
/breakdown docs/prd/new-feature/index.md --project-path /existing/project
```

Output:
```
Analyzing PRD: new-feature
Detected: brownfield project
Existing project: /existing/project
Skipping Layer 0 (setup)

[continues with layers 1-4...]
```

### CRD (Change Request)
```
/breakdown docs/crd/dark-mode-toggle.md --project-path /existing/project
```

Output:
```
Analyzing CRD: dark-mode-toggle
Detected: Change Request Document
Existing project: /existing/project
Loading PROJECT.md context...

Impact Analysis:
  - Affected files: 4 (2 modify, 2 create)
  - Affected features: settings
  - Breaking changes: none

Planning layers from impact...
  - Layer 2 (backend): 1 task (API endpoint)
  - Layer 3 (frontend): 2 tasks (component, hook)
  - Layer 4 (integration): 1 task (wiring)

Generating Layer 2 (Backend)...
Batch 1/1: [L2-001]
- L2-001-theme-settings-api.xml
Reviewing batch... PASSED
Created: docs/tasks/dark-mode-toggle/2-backend/.done

Generating Layer 3 (Frontend)...
Batch 1/1: [L3-001, L3-002]
- L3-001-theme-toggle-component.xml
- L3-002-use-theme-hook.xml
Reviewing batch... PASSED
Created: docs/tasks/dark-mode-toggle/3-frontend/.done

Generating Layer 4 (Integration)...
Batch 1/1: [L4-001]
- L4-001-wire-theme-toggle.xml
Reviewing batch... PASSED
Created: docs/tasks/dark-mode-toggle/4-integration/.done

Breakdown complete!
- Total tasks: 4
- 2-backend: 1 task
- 3-frontend: 2 tasks
- 4-integration: 1 task

To execute:
  /execute docs/tasks/dark-mode-toggle/ --project-path /existing/project
```
