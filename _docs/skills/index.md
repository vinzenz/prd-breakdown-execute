---
layout: default
title: Skills Reference
nav_order: 3
has_children: true
permalink: /docs/skills/
---

# Skills Reference

PRD Breakdown Execute includes several skills organized into commands and sub-skills.

## Commands (User-Invocable)

These are the entry points you use directly:

| Command | Purpose | Workflow |
|---------|---------|----------|
| [`/prd`]({{ '/docs/skills/prd/' | relative_url }}) | Create PRD interactively | Greenfield |
| [`/crd`]({{ '/docs/skills/crd/' | relative_url }}) | Create change request | Brownfield |
| [`/crd-context`]({{ '/docs/skills/crd-context/' | relative_url }}) | Manage PROJECT.md | Brownfield |
| `/breakdown` | Generate tasks from PRD/CRD | Both |
| `/execute` | Run tasks in parallel | Both |

## Skill Hierarchy

```
                    User Commands
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
    ┌────────┐     ┌──────────┐    ┌─────────┐
    │  /prd  │     │/breakdown│    │/execute │
    │        │     │  (fork)  │    │ (fork)  │
    └────────┘     └────┬─────┘    └────┬────┘
                        │               │
         ┌──────────────┼───────┐       │
         │              │       │       │
         ▼              ▼       ▼       │
    ┌─────────┐  ┌──────────┐  ┌──┐    │
    │ analyze │  │plan-     │  │ge│    │
    │   prd   │  │ layers   │  │ne│    │
    └─────────┘  └──────────┘  │ra│    │
                               │te│    │
                               └──┘    │
                                       │
              ┌────────────────────────┼───────────────┐
              │                        │               │
              ▼                        ▼               ▼
        ┌──────────┐           ┌────────────┐   ┌───────────┐
        │  layer   │           │   batch    │   │   merge   │
        │  (fork)  │           │   (fork)   │   │  (fork)   │
        └────┬─────┘           └──────┬─────┘   └───────────┘
             │                        │
             │              ┌─────────┴─────────┐
             │              │                   │
             │              ▼                   ▼
             │        ┌──────────┐       ┌──────────┐
             │        │  task    │       │  verify  │
             │        │  (fork)  │       │  (fork)  │
             │        └──────────┘       └──────────┘
             │
             └──────────────────────────────────────────────
```

## Breakdown Sub-Skills

| Skill | Model | Context | Purpose |
|-------|-------|---------|---------|
| `breakdown` | sonnet | fork | Main orchestrator |
| `breakdown-analyze-prd` | sonnet | default | Extract features, entities |
| `breakdown-plan-layers` | sonnet | default | Organize into layers |
| `breakdown-generate-tasks` | sonnet | default | Create task XML files |
| `breakdown-review-tasks` | haiku | default | Validate task quality |

[Detailed breakdown documentation →]({{ '/docs/skills/breakdown/' | relative_url }})

## Execute Sub-Skills

| Skill | Model | Context | Purpose |
|-------|-------|---------|---------|
| `execute` | sonnet | fork | Main orchestrator |
| `execute-layer` | sonnet | fork | Layer management |
| `execute-batch` | sonnet | fork | Parallel batching |
| `execute-task` | sonnet | fork | TDD implementation |
| `execute-verify` | haiku | fork | Verification |
| `execute-merge` | sonnet | fork | Merge to main |

[Detailed execute documentation →]({{ '/docs/skills/execute/' | relative_url }})

## CRD Sub-Skills

| Skill | Model | Context | Purpose |
|-------|-------|---------|---------|
| `crd` | sonnet | fork | CRD orchestration |
| `crd-investigate` | sonnet | fork | Deep codebase analysis |
| `crd-context-update` | sonnet | fork | Incremental updates |
| `crd-impact-analysis` | sonnet | fork | Impact detection |

[Detailed CRD documentation →]({{ '/docs/skills/crd/' | relative_url }})

## Agents

Specialized executors invoked via the Task tool:

| Agent | Used By | Purpose |
|-------|---------|---------|
| `task-generator` | breakdown | Create task XML |
| `task-implementer` | execute-task | TDD implementation |
| `task-reviewer` | breakdown-review | Quality validation |
| `verification-runner` | execute-verify | Run commands |
| `crd-investigator` | crd | Generate PROJECT.md |
| `crd-context-updater` | crd | Update PROJECT.md |
| `crd-impact-analyzer` | crd | Analyze impact |
| `project-context-finalizer` | execute | Update after execute |

## Skill Definition

Skills are defined in `SKILL.md` files:

```yaml
---
name: execute-task
context: fork
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
---

# Execute Task

Implements a single task using TDD workflow.

## Instructions

1. Read the task XML file
2. Write tests first (red phase)
3. Run tests to confirm failure
4. Write implementation (green phase)
5. Run tests to confirm pass
6. Commit changes

## Output

Return task status and any errors.
```

## Reference Files

Skills can include reference files:

```
.claude/skills/breakdown/
├── SKILL.md
└── references/
    ├── layer-definitions.md
    ├── task-format-spec.md
    └── review-criteria.md
```

These are included in the skill's context automatically.
