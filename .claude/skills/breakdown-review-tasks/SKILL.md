---
name: breakdown-review-tasks
description: Review generated tasks for quality and completeness. Called by /breakdown skill after task generation.
context: fork
agent: task-reviewer
allowed-tools: Read Glob Grep Write
model: claude-haiku-4-5
---

# Task Review

You are reviewing task files for quality and completeness.

## Input

The calling skill will provide:
1. Path to directory containing generated task files
2. Layer being reviewed

## Review Criteria

### Critical (Must Pass)

Failure on ANY critical criterion means the task FAILS.

#### 1. Completeness
- All required XML sections present
- No empty sections
- No placeholder text: "TODO", "TBD", "...", "[fill in]", "etc."

#### 2. Self-Containment
- All context is inline (no "see PRD" or "check docs")
- Interface contracts include imports
- Tech stack versions specified

#### 3. Interface Contracts
- Every `<interface>` has `name` and `type` attributes
- Complete type annotations
- Import statements included
- Return types specified

#### 4. Requirements Specificity
- Each requirement has unique `id`
- Exact file paths (not "in the models folder")
- Exact class/function names
- Exact field names and types
- No ambiguous terms: "appropriate", "suitable", "proper"

#### 5. Test Requirements
- Test file path specified
- Each test has unique `id`
- Setup steps are concrete
- Assertions are specific
- Concrete test values (not "some value")

#### 6. Verification Steps
- All steps are runnable commands
- Expected outcome for each step
- Commands use correct syntax

#### 7. File Scope
- Maximum 3 files (excluding test)
- Full relative paths from project root

### Warning (Note but Don't Fail)

#### 1. Context Quality
- PRD excerpt is focused (not entire PRD)
- Project structure is accurate

#### 2. Export Completeness
- All public interfaces exported
- Usable by downstream tasks

## Review Process

For each task file in the directory:

1. **Parse XML**: Verify well-formed
2. **Check structure**: All required sections present
3. **Critical scan**: Check each critical criterion
4. **Warning scan**: Note any warnings
5. **Record result**: Pass/Fail with details

## Output Format

Write review results to `{layer}_review.json`:

```json
{
  "layer": "1-foundation",
  "reviewed_at": "2024-01-15T10:30:00Z",
  "summary": {
    "total": 5,
    "passed": 4,
    "failed": 1
  },
  "results": [
    {
      "task_id": "L1-001",
      "task_file": "L1-001-project-model.xml",
      "pass": true,
      "critical_issues": [],
      "warnings": [
        {
          "criterion": "context_quality",
          "location": "<context><prd-excerpt>",
          "issue": "PRD excerpt could be more focused"
        }
      ]
    },
    {
      "task_id": "L1-002",
      "task_file": "L1-002-conversation-model.xml",
      "pass": false,
      "critical_issues": [
        {
          "criterion": "completeness",
          "location": "<requirements><requirement id=\"3\">",
          "issue": "Contains placeholder 'TBD' for field type"
        },
        {
          "criterion": "test_requirements",
          "location": "<test-requirements><test id=\"2\">",
          "issue": "Missing concrete assertion value"
        }
      ],
      "warnings": []
    }
  ],
  "verdict": "FAILED",
  "action_required": "Fix L1-002: Remove placeholder, add concrete test values"
}
```

## Failure Patterns to Detect

### Placeholders
```
TODO
TBD
FIXME
[fill in]
[to be determined]
...
etc.
```

### Vague Requirements
```
"Create the appropriate model"
"Add necessary fields"
"Implement proper validation"
"Handle errors appropriately"
"Follow best practices"
```

### Missing Context
```
"As described in the PRD"
"Using the standard approach"
"Like other models"
"Following the existing pattern"
```

### Incomplete Interfaces
```
def some_function():  # Missing return type
    ...

class Model:  # Missing type annotations
    field = Column(...)  # Old SQLAlchemy style
```

## Review Output

After reviewing all tasks:

1. Write `{layer}_review.json` to the layer directory
2. Print summary to console:

```
Review Results for 1-foundation:
================================
Total tasks: 5
Passed: 4
Failed: 1

FAILED:
- L1-002: Contains placeholder 'TBD', missing test assertion

Action required before proceeding.
```

## Verdict Rules

- `PASSED`: All tasks pass all critical criteria
- `FAILED`: Any task fails any critical criterion

The layer can only proceed (create `.done`) if verdict is `PASSED`.
