# Introduction

## What is PRD Breakdown Execute?

A complete autonomous software development workflow using Claude Code skills.

It transforms your ideas into working code through three orchestrated phases:

```
     Your Idea
         │
         ▼
    ┌─────────┐
    │  /prd   │  Interactive PRD creation
    └────┬────┘
         │
         ▼
    ┌──────────┐
    │/breakdown│  Analyze, plan, generate tasks
    └────┬─────┘
         │
         ▼
    ┌──────────┐
    │ /execute │  Parallel implementation
    └────┬─────┘
         │
         ▼
    Working Code
    with Tests
```

---

## Why This Exists

### The Problem

Building software with LLMs traditionally suffers from:

1. **Context Pollution** - Earlier work interferes with later work
2. **No True Parallelism** - One thing at a time
3. **Lost Context** - Important details forgotten as conversation grows
4. **Manual Coordination** - You have to orchestrate everything

### The Solution

This project demonstrates Claude Code's **context fork** feature:

- Each task runs in complete isolation
- Multiple tasks execute simultaneously
- No information bleeds between tasks
- Automatic orchestration from PRD to code

---

## The Three Phases

### Phase 1: PRD Creation (`/prd`)

An interactive 8-phase workflow that guides you through defining your project:

| Phase | What Happens |
|-------|--------------|
| 1. Idea | Describe your idea, problem, target users |
| 2. Tech | Choose technology stack |
| 3. Features | Define features with MoSCoW prioritization |
| 4. Dependencies | Specify external services, libraries |
| 5-6. Options | Add motivation, competitive analysis, NFRs |
| 7. Review | Interactive revision |
| 8. Output | Generate structured PRD files |

**Output:** `docs/prd/{project}/index.md` and feature files

### Phase 2: Task Breakdown (`/breakdown`)

Converts your PRD into executable task files:

```
PRD
 │
 ├──► Analyze: Extract features, infer models/APIs
 │
 ├──► Plan: Organize into 5 layers
 │
 ├──► Generate: Create self-contained task XML
 │
 └──► Review: Validate each task for quality
```

**Output:** `docs/tasks/{project}/` with layer directories and task files

### Phase 3: Parallel Execution (`/execute`)

Implements all tasks with TDD:

```
Tasks
 │
 ├──► Orchestrate: Process layers sequentially
 │
 ├──► Parallelize: Run tasks in isolated worktrees
 │
 ├──► Verify: Independent verification per task
 │
 └──► Merge: Sequential merge to main branch
```

**Output:** Working code in your project directory

---

## The 5-Layer Architecture

Tasks are organized into architectural layers:

```
Layer 4: Integration     ◄── E2E flows, final wiring
         ▲
         │
Layer 3: Frontend        ◄── UI components, state
         ▲
         │ (depends on contracts, not implementation)
         │
Layer 2: Backend         ◄── APIs, services, business logic
         ▲
         │
Layer 1: Foundation      ◄── Models, migrations, config
         ▲
         │
Layer 0: Setup           ◄── Project initialization
                             (greenfield projects only)
```

Each layer depends only on the one below it. Within a layer, tasks can run in parallel.

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Parallel Execution** | Multiple tasks run simultaneously in git worktrees |
| **Context Isolation** | Each task gets fresh, focused context |
| **TDD by Default** | Tests written first, verified independently |
| **Self-Healing** | Failed tasks retry with feedback (up to 5 times) |
| **Resume Anywhere** | Stop and continue without losing progress |
| **Model Specialization** | Right model for each job |

---

## Who Is This For?

- **Developers** exploring AI-assisted development workflows
- **Teams** interested in autonomous code generation
- **Contributors** to Claude Code who want to understand advanced patterns
- **Anyone** curious about the context fork feature

---

## Next Steps

- [Context Fork Deep Dive](context-fork.md) - Understand the key innovation
- [Architecture Overview](architecture.md) - See the system design
- [Quick Start](../quickstart/README.md) - Get running in 5 minutes
