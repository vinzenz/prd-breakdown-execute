# Quick Start

Get from idea to running code in 5 minutes.

---

## Prerequisites

- **Claude Code CLI** installed and configured
- **Git** installed and configured
- A project idea in mind

---

## The Three Commands

```bash
# Step 1: Create a PRD
claude /prd

# Step 2: Break it down into tasks
claude /breakdown docs/prd/your-project/index.md

# Step 3: Execute the tasks
claude /execute docs/tasks/your-project
```

That's it. Three commands to go from idea to working code.

---

## Step 1: Create a PRD

```bash
claude /prd
```

You'll be guided through 8 interactive phases:

```
Phase 1: Idea Capture
─────────────────────
What problem are you solving?

> A task management app for small teams that integrates with Slack

Who are your target users?

> Small development teams (5-15 people) who use Slack daily

What's the core value proposition?

> Simple task tracking without leaving Slack
```

The conversation continues through:
- **Phase 2**: Technology stack selection
- **Phase 3**: Feature definition (MoSCoW prioritization)
- **Phase 4**: Dependencies and integrations
- **Phase 5-6**: Optional sections
- **Phase 7**: Review and revisions
- **Phase 8**: Output generation

**Output created:**
```
docs/prd/task-manager/
├── index.md          # Complete PRD
├── what-next.md      # Progress tracking
└── features/
    ├── task-crud.md
    ├── slack-integration.md
    └── team-dashboard.md
```

[Detailed PRD walkthrough](first-prd.md)

---

## Step 2: Break It Down

```bash
claude /breakdown docs/prd/task-manager/index.md
```

The breakdown skill:

1. **Analyzes** your PRD
   - Extracts features
   - Infers data models
   - Identifies API endpoints
   - Maps frontend components

2. **Plans** the implementation
   - Organizes into 5 layers
   - Builds dependency graph
   - Orders tasks

3. **Generates** task files
   - Creates self-contained XML specs
   - Each task has all context needed
   - Includes test requirements

4. **Reviews** for quality
   - Validates completeness
   - Checks for placeholders
   - Ensures self-containment

**Output created:**
```
docs/tasks/task-manager/
├── analysis.json
├── layer_plan.json
├── manifest.json
├── 0-setup/
│   └── L0-001-project-setup.xml
├── 1-foundation/
│   ├── L1-001-task-model.xml
│   └── L1-002-user-model.xml
├── 2-backend/
│   ├── L2-001-task-api.xml
│   └── L2-002-slack-webhook.xml
├── 3-frontend/
│   └── L3-001-dashboard.xml
└── 4-integration/
    └── L4-001-slack-flow.xml
```

[Detailed breakdown walkthrough](first-breakdown.md)

---

## Step 3: Execute

```bash
claude /execute docs/tasks/task-manager
```

The execute skill:

1. **Orchestrates** layer-by-layer
   - Processes layers 0→1→2→3→4
   - Respects dependencies

2. **Parallelizes** within layers
   - Multiple tasks run simultaneously
   - Each in isolated git worktree

3. **Implements** with TDD
   - Writes tests first
   - Then implementation
   - Verifies independently

4. **Merges** to main
   - Sequential merge (no conflicts)
   - Clean git history

**What you'll see:**
```
Executing layer 0-setup...
  L0-001: Setting up project... done

Executing layer 1-foundation...
  L1-001: Creating Task model... done
  L1-002: Creating User model... done

Executing layer 2-backend...
  L2-001: [=========>          ] Implementing Task API
  L2-002: [=====>              ] Implementing Slack webhook

...

Execution complete.
- Tasks completed: 12
- Tasks failed: 0
- Total time: 8m 23s
```

[Detailed execution walkthrough](first-execute.md)

---

## What You Get

After execution completes:

```
your-project/
├── src/
│   ├── models/
│   │   ├── task.py
│   │   └── user.py
│   ├── api/
│   │   ├── tasks.py
│   │   └── slack.py
│   └── ...
├── tests/
│   ├── test_task_model.py
│   ├── test_user_model.py
│   ├── test_tasks_api.py
│   └── ...
└── ...
```

- **Working code** implementing your PRD
- **Tests** for all components
- **Clean git history** with meaningful commits

---

## Common Options

### Breakdown Options

```bash
# Greenfield project (new project from template)
claude /breakdown docs/prd/my-project/index.md \
  --output-dir /path/to/new-project

# Brownfield project (existing codebase)
claude /breakdown docs/prd/my-feature/index.md \
  --project-path /path/to/existing-project
```

### Execute Options

```bash
# Resume after interruption
claude /execute docs/tasks/my-project --resume

# Run more tasks in parallel
claude /execute docs/tasks/my-project --max-parallel 5

# Dry run (see what would happen)
claude /execute docs/tasks/my-project --dry-run
```

---

## Troubleshooting

### PRD won't complete

Check `what-next.md` for incomplete items. Resume with:
```bash
claude /prd --resume
```

### Breakdown fails review

The review will retry up to 3 times per batch. If it keeps failing:
- Check the task XML for placeholders
- Ensure PRD has concrete details

### Task fails verification

Tasks retry up to 5 times with feedback. If abandoned:
- Check `.worktrees/{task-id}/` for the failed state
- Fix manually and resume: `claude /execute ... --resume`

[Full troubleshooting guide](../troubleshooting.md)

---

## Next Steps

- [Detailed PRD walkthrough](first-prd.md)
- [Understanding the breakdown](first-breakdown.md)
- [Watching execution](first-execute.md)
- [Context Fork innovation](../introduction/context-fork.md)
