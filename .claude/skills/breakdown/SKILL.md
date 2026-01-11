---
name: breakdown
description: Break down a PRD into self-contained implementation tasks for LLM execution. Use when you have a PRD file and want to generate executable task files for autonomous implementation.
allowed-tools: Read Glob Grep Write Skill Bash
model: claude-sonnet-4-5
---

# /breakdown - PRD Task Breakdown

You are orchestrating the breakdown of a PRD into implementation tasks optimized for autonomous LLM execution.

## Arguments

- `<prd-path>`: Path to PRD index.md file (required)
- `--layer <N>`: Only generate tasks for specific layer (optional, 0-4)
- `--review-only`: Only run review pass on existing tasks (optional)
- `--output-dir <path>`: Target directory for greenfield projects (overrides default)
- `--project-path <path>`: Existing project path for brownfield (overrides PRD value)
- `--auto-setup`: Automatically execute Layer 0 tasks after generation (greenfield only)

## Output Location

Tasks are written to: `docs/tasks/<prd-slug>/`

For greenfield with `--output-dir`:
- Layer 0 tasks reference the output-dir as target
- Other layer tasks are still saved to docs/tasks/

## Workflow

Execute these phases in order:

### Phase 1: Validate Input

1. Read the PRD file at the provided path
2. Verify it has valid XML structure with `<prd>` root element
3. Extract the slug from `<meta><slug>` tag
4. Create output directory: `docs/tasks/{slug}/`
5. If directory exists, check for existing `.done` markers to resume

### Phase 2: Analyze PRD

Invoke the `breakdown-analyze-prd` skill with the full PRD content.

Pass the PRD XML content and request structured extraction of:
- All features with priorities
- Tech stack with versions
- Implied data models
- Implied API endpoints
- Implied frontend components
- External dependencies
- Template path if specified

Save the analysis to `docs/tasks/{slug}/analysis.json`

### Phase 3: Plan Layers

Invoke the `breakdown-plan-layers` skill with the analysis JSON.

Request organization into 4 layers:
1. **1-foundation**: Database models, migrations, base config
2. **2-backend**: API endpoints, services, business logic
3. **3-frontend**: React components, state management, routing
4. **4-integration**: Wiring, E2E flows, polish

Save the layer plan to `docs/tasks/{slug}/layer_plan.json`

### Phase 4: Generate Tasks (Per Layer)

Determine layers to process based on project type:
- **Greenfield**: `[0-setup, 1-foundation, 2-backend, 3-frontend, 4-integration]`
- **Brownfield**: `[1-foundation, 2-backend, 3-frontend, 4-integration]` (skip Layer 0)

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

### Brownfield Project
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
