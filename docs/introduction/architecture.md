# Architecture Overview

A visual guide to how the PRD Breakdown Execute system works.

---

## System Diagram

```
                                      USER
                                        │
                                        │ claude /prd
                                        │ claude /breakdown
                                        │ claude /execute
                                        │
                                        ▼
            ┌───────────────────────────────────────────────────────┐
            │                    Claude Code CLI                    │
            │                                                       │
            │   Loads skills from .claude/skills/                   │
            │   Loads commands from .claude/commands/               │
            │   Manages context forking                             │
            └───────────────────────────────────────────────────────┘
                                        │
               ┌────────────────────────┼────────────────────────┐
               │                        │                        │
               ▼                        ▼                        ▼
        ┌─────────────┐          ┌─────────────┐          ┌─────────────┐
        │    /prd     │          │  /breakdown │          │  /execute   │
        │   Command   │          │    Skill    │          │    Skill    │
        │             │          │             │          │             │
        │  context:   │          │  context:   │          │  context:   │
        │  (default)  │          │    fork     │          │    fork     │
        └──────┬──────┘          └──────┬──────┘          └──────┬──────┘
               │                        │                        │
               │                        │                        │
               ▼                        ▼                        ▼
        8 Interactive              Sub-Skills               Sub-Skills
           Phases                  (see below)              (see below)
```

---

## Phase 1: PRD Creation

```
User
 │
 │ "claude /prd"
 │
 ▼
┌────────────────────────────────────────────────────────────┐
│                      /prd Command                          │
│                                                            │
│  Phase 1: Idea Capture                                     │
│           └── Problem, users, value proposition            │
│                                                            │
│  Phase 2: Tech Stack                                       │
│           └── Greenfield vs brownfield, technologies       │
│                                                            │
│  Phase 3: Features (MoSCoW)                                │
│           └── Must/Should/Could/Won't have                 │
│                                                            │
│  Phase 4: Dependencies                                     │
│           └── External services, libraries                 │
│                                                            │
│  Phase 5-6: Optional Sections                              │
│           └── Motivation, competition, NFRs                │
│                                                            │
│  Phase 7: Interactive Review                               │
│           └── User revisions                               │
│                                                            │
│  Phase 8: Output Generation                                │
│           └── Create files                                 │
└────────────────────────────────────────────────────────────┘
 │
 ▼
docs/prd/{project-slug}/
├── index.md         ◄── Full PRD in XML format
├── what-next.md     ◄── Progress tracking
└── features/
    ├── feature-1.md ◄── Feature details
    └── feature-2.md
```

---

## Phase 2: Breakdown

```
docs/prd/{slug}/index.md
         │
         │ "claude /breakdown docs/prd/{slug}/index.md"
         │
         ▼
┌────────────────────────────────────────────────────────────┐
│                    /breakdown Skill                        │
│                    context: fork                           │
└────────────────────────────────────────────────────────────┘
         │
         │ Fork
         ▼
┌────────────────────────────────────────────────────────────┐
│                 breakdown-analyze-prd                      │
│                 context: fork                              │
│                                                            │
│  • Extract features from PRD                               │
│  • Infer data models from descriptions                     │
│  • Infer API endpoints                                     │
│  • Infer frontend components                               │
│  • Detect template from tech stack                         │
└────────────────────────────────────────────────────────────┘
         │
         │ analysis.json
         ▼
┌────────────────────────────────────────────────────────────┐
│                 breakdown-plan-layers                      │
│                 context: fork                              │
│                                                            │
│  • Organize into 5 layers                                  │
│  • Build dependency graph                                  │
│  • Assign each component to layer                          │
│  • Order tasks within layers                               │
└────────────────────────────────────────────────────────────┘
         │
         │ layer_plan.json
         ▼
┌────────────────────────────────────────────────────────────┐
│              For each layer (0-4):                         │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │            breakdown-generate-tasks                  │  │
│  │            context: fork                             │  │
│  │                                                      │  │
│  │  • Create self-contained task XML files              │  │
│  │  • Include all context inline                        │  │
│  │  • Define interface contracts                        │  │
│  │  • Specify verification steps                        │  │
│  └──────────────────────────────────────────────────────┘  │
│                          │                                 │
│                          ▼                                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │            breakdown-review-tasks                    │  │
│  │            context: fork                             │  │
│  │                                                      │  │
│  │  • Validate completeness                             │  │
│  │  • Check for placeholders                            │  │
│  │  • Verify self-containment                           │  │
│  │  • PASS or FAIL (retry on fail)                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
└────────────────────────────────────────────────────────────┘
         │
         ▼
docs/tasks/{slug}/
├── analysis.json
├── layer_plan.json
├── manifest.json
├── 0-setup/
│   └── L0-*.xml
├── 1-foundation/
│   └── L1-*.xml
├── 2-backend/
│   └── L2-*.xml
├── 3-frontend/
│   └── L3-*.xml
└── 4-integration/
    └── L4-*.xml
```

---

## Phase 3: Execution

```
docs/tasks/{slug}/
         │
         │ "claude /execute docs/tasks/{slug}"
         │
         ▼
┌────────────────────────────────────────────────────────────┐
│                      /execute Skill                        │
│                      context: fork                         │
│                                                            │
│  • Load manifest and state                                 │
│  • Process layers sequentially                             │
│  • Coordinate sub-skills                                   │
│  • Update state file                                       │
└────────────────────────────────────────────────────────────┘
         │
         │ For each layer
         ▼
┌────────────────────────────────────────────────────────────┐
│                    execute-layer                           │
│                    context: fork                           │
│                                                            │
│  • Identify ready tasks (dependencies satisfied)           │
│  • Group into batches (max_parallel)                       │
│  • Process batches sequentially                            │
│  • Handle merge queue                                      │
└────────────────────────────────────────────────────────────┘
         │
         │ For each batch
         ▼
┌────────────────────────────────────────────────────────────┐
│                    execute-batch                           │
│                    context: fork                           │
│                                                            │
│  • Launch tasks in parallel                                │
│  • Each task in separate git worktree                      │
│  • Collect results                                         │
│  • Update merge queue                                      │
└────────────────────────────────────────────────────────────┘
         │
         │ Parallel (3 tasks shown)
         │
    ┌────┼────┬────────────────┐
    │    │    │                │
    ▼    │    ▼                ▼
┌───────┐│┌───────┐      ┌───────┐
│Task 1 │││Task 2 │      │Task 3 │
│       │││       │      │       │
│Worktree││Worktree│     │Worktree│
└───┬───┘│└───┬───┘      └───┬───┘
    │    │    │              │
    ▼    │    ▼              ▼
┌───────┐│┌───────┐      ┌───────┐
│Verify │││Verify │      │Verify │
│(Haiku)│││(Haiku)│      │(Haiku)│
└───┬───┘│└───┬───┘      └───┬───┘
    │    │    │              │
    └────┼────┴──────────────┘
         │
         │ Sequential merge (completion order)
         ▼
┌────────────────────────────────────────────────────────────┐
│                    execute-merge                           │
│                    context: fork                           │
│                                                            │
│  • Merge worktree-L1-001 to main                          │
│  • Merge worktree-L1-002 to main                          │
│  • Merge worktree-L1-003 to main                          │
│  • Clean up worktrees                                      │
└────────────────────────────────────────────────────────────┘
         │
         ▼
Working code in project directory
```

---

## Model Distribution

```
                    ┌─────────────┐
                    │   Sonnet    │  Reasoning, architecture
                    │             │  implementation
                    └──────┬──────┘
                           │
     ┌─────────────────────┼─────────────────────┐
     │                     │                     │
     ▼                     ▼                     ▼
┌─────────┐          ┌─────────┐          ┌─────────┐
│  /prd   │          │/breakdown│         │/execute │
│ Sonnet  │          │ Sonnet  │          │ Sonnet  │
└─────────┘          └────┬────┘          └────┬────┘
                          │                    │
              ┌───────────┼───────────┐       │
              │           │           │       │
              ▼           ▼           ▼       │
         ┌────────┐ ┌────────┐ ┌────────┐    │
         │analyze │ │generate│ │ review │    │
         │Sonnet  │ │Sonnet  │ │ Haiku  │    │
         └────────┘ └────────┘ └────────┘    │
                                              │
                    ┌─────────────────────────┘
                    │
         ┌──────────┼──────────┐
         │          │          │
         ▼          ▼          ▼
    ┌────────┐ ┌────────┐ ┌────────┐
    │  task  │ │ verify │ │ merge  │
    │ Sonnet │ │ Haiku  │ │ Sonnet │
    └────────┘ └────────┘ └────────┘
```

**Sonnet** (expensive, powerful): Orchestration, implementation, decisions
**Haiku** (fast, focused): Verification, review, quality checks

---

## Data Flow Summary

```
User Idea
    │
    ▼
┌───────────────────────────────────────────────────────────────────┐
│                            PRD Phase                              │
│   Interactive conversation → Structured XML documents             │
└───────────────────────────────────────────────────────────────────┘
    │
    │ docs/prd/{slug}/index.md
    ▼
┌───────────────────────────────────────────────────────────────────┐
│                         Breakdown Phase                           │
│   PRD XML → Analysis → Layer Plan → Task XML files               │
└───────────────────────────────────────────────────────────────────┘
    │
    │ docs/tasks/{slug}/*.xml
    ▼
┌───────────────────────────────────────────────────────────────────┐
│                          Execute Phase                            │
│   Task XML → Parallel Worktrees → TDD → Verified Code            │
└───────────────────────────────────────────────────────────────────┘
    │
    │ src/**/*.{py,ts,go}
    ▼
Working Software
```

---

## State Files

| File | Phase | Purpose |
|------|-------|---------|
| `index.md` | PRD | Complete PRD in XML |
| `what-next.md` | PRD | Progress tracking |
| `analysis.json` | Breakdown | PRD analysis results |
| `layer_plan.json` | Breakdown | Layer organization |
| `manifest.json` | Breakdown | Task inventory |
| `execute-state.json` | Execute | Execution state |

---

## Git Worktree Architecture

```
project/
├── .git/                    ◄── Main repository
├── src/                     ◄── Main branch code
└── .worktrees/              ◄── Parallel workspaces
    ├── L1-001/              ◄── Task L1-001 worktree
    │   └── (full project)
    ├── L1-002/              ◄── Task L1-002 worktree
    │   └── (full project)
    └── L1-003/              ◄── Task L1-003 worktree
        └── (full project)

Each worktree:
- Has its own branch (worktree-L1-001)
- Contains full project copy
- Operates independently
- Merges back to main when complete
```

---

## Next Steps

- [Context Fork Deep Dive](context-fork.md) - The key innovation
- [Skill Reference](../skills/README.md) - All skills documented
- [Quick Start](../quickstart/README.md) - Try it yourself
