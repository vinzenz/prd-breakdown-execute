# Task Review Criteria

Use this checklist to validate generated task files.

## Critical Criteria (Must Pass)

Failure on ANY critical criterion means the task must be regenerated.

### 1. Completeness

- [ ] All required XML sections present (`meta`, `context`, `dependencies`, `objective`, `requirements`, `test-requirements`, `files-to-create`, `verification`, `exports`)
- [ ] No empty sections
- [ ] No placeholder text: "TODO", "TBD", "...", "[fill in]", "etc."
- [ ] No "see above", "as mentioned", or references to other parts of the document

### 2. Self-Containment

- [ ] All necessary context is inline (not "see PRD" or "check docs")
- [ ] Interface contracts include all imports
- [ ] Tech stack versions specified
- [ ] No assumptions about "obvious" knowledge

### 3. Interface Contracts

- [ ] Every `<interface>` has `name` and `type` attributes
- [ ] Complete type annotations on all signatures
- [ ] Import statements included
- [ ] Return types specified
- [ ] Docstrings for functions (at minimum)

### 4. Requirements Specificity

- [ ] Each requirement has unique `id`
- [ ] Exact file paths specified (not "in the models folder")
- [ ] Exact class/function names specified
- [ ] Exact field names and types specified
- [ ] No ambiguous terms: "appropriate", "suitable", "proper", "good"

### 5. Test Requirements

- [ ] Test file path specified
- [ ] Each test has unique `id`
- [ ] Setup steps are concrete
- [ ] Assertions are specific (not "should work correctly")
- [ ] Uses concrete test values (not "some value")

### 6. Verification Steps

- [ ] All steps are runnable commands
- [ ] Expected outcome specified for each step
- [ ] Commands use correct syntax for the tech stack
- [ ] Steps are in executable order

### 7. File Scope

- [ ] Maximum 3 files to create (excluding test file)
- [ ] Files listed in creation order
- [ ] Full relative paths from project root

## Warning Criteria (Should Pass)

Warnings don't block, but should be noted for improvement.

### 1. Context Quality

- [ ] PRD excerpt is relevant (not entire PRD copied)
- [ ] Tech stack matches PRD specification
- [ ] Project structure reflects actual layout

### 2. Dependency Clarity

- [ ] Only necessary dependencies listed
- [ ] Dependency purpose is clear
- [ ] No unused imports in interface contracts

### 3. Objective Clarity

- [ ] 1-3 sentences (not a paragraph)
- [ ] Focuses on WHAT, not HOW
- [ ] Clear value statement

### 4. Export Completeness

- [ ] All public interfaces exported
- [ ] Interfaces usable by downstream tasks
- [ ] No internal implementation details exposed

## Review Output Format

```json
{
  "task_id": "L1-001",
  "task_name": "Create Project Model",
  "pass": false,
  "critical_issues": [
    {
      "criterion": "completeness",
      "location": "<requirements>",
      "issue": "Requirement 3 contains placeholder 'TBD'"
    }
  ],
  "warnings": [
    {
      "criterion": "context_quality",
      "location": "<context><prd-excerpt>",
      "issue": "PRD excerpt is 500+ lines, consider trimming"
    }
  ],
  "suggestions": [
    "Consider splitting into two tasks - model + migration separate"
  ]
}
```

## Review Process

1. **Parse XML**: Verify well-formed XML
2. **Check structure**: All required sections present
3. **Critical scan**: Check each critical criterion
4. **Warning scan**: Check each warning criterion
5. **Generate report**: Output JSON with findings
6. **Verdict**: `pass: true` only if 0 critical issues

## Common Issues

### Placeholder Patterns to Flag

```
TODO
TBD
FIXME
[fill in]
[to be determined]
...
etc.
and so on
as needed
appropriate
suitable
proper
relevant
necessary (without specifics)
```

### Vague Requirement Patterns

```
"Create the appropriate model"          → Specify exact class name
"Add necessary fields"                  → List exact fields with types
"Implement proper validation"           → Specify validation rules
"Store the data"                        → Specify table/collection name
"Handle errors appropriately"           → Specify which errors, how
"Follow best practices"                 → Specify exact pattern/approach
```

### Missing Context Patterns

```
"As described in the PRD"               → Copy relevant PRD text inline
"Using the standard approach"           → Specify the approach
"Like other models in the project"      → Copy the pattern inline
"Following the existing pattern"        → Document the pattern
```
