```
╔══════════════════════════════════════════════════════════════════════════════╗
║                                                                              ║
║    ██████╗ ██████╗ ██████╗     ██████╗ ██████╗ ███████╗ █████╗ ██╗  ██╗     ║
║    ██╔══██╗██╔══██╗██╔══██╗    ██╔══██╗██╔══██╗██╔════╝██╔══██╗██║ ██╔╝     ║
║    ██████╔╝██████╔╝██║  ██║    ██████╔╝██████╔╝█████╗  ███████║█████╔╝      ║
║    ██╔═══╝ ██╔══██╗██║  ██║    ██╔══██╗██╔══██╗██╔══╝  ██╔══██║██╔═██╗      ║
║    ██║     ██║  ██║██████╔╝    ██████╔╝██║  ██║███████╗██║  ██║██║  ██╗     ║
║    ╚═╝     ╚═╝  ╚═╝╚═════╝     ╚═════╝ ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝     ║
║                                                                              ║
║         Autonomous Software Development with Claude Code Skills             ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

> Transform ideas into working code through AI-orchestrated parallel development.

---

## The Workflow

```
     ┌──────────────┐         ┌───────────────┐         ┌──────────────┐
     │    /prd      │         │   /breakdown  │         │   /execute   │
     │              │  ────>  │               │  ────>  │              │
     │  8 Phases    │         │   5 Layers    │         │   Parallel   │
     │  Interactive │         │   Task Gen    │         │   Worktrees  │
     └──────────────┘         └───────────────┘         └──────────────┘
           │                        │                         │
           │                        │                         │
     Your idea in             Self-contained             Working code
     structured form          task XML files             with tests
```

**Three commands. One pipeline. Autonomous development.**

---

## The Innovation: Context Fork

> Traditional LLM development suffers from context pollution - earlier task details bleed into later ones, errors compound, and context windows fill with irrelevant information.
>
> This repository demonstrates Claude Code's **context fork** feature:
>
> ```
> Main Context (Orchestrator)
>      │
>      ├── Fork: Task L1-001 ──── Fresh, isolated context
>      │
>      ├── Fork: Task L1-002 ──── No knowledge of L1-001
>      │
>      └── Fork: Task L1-003 ──── Completely independent
> ```
>
> Each fork starts clean. No context bleed. True parallelism.
>
> [Learn more about Context Fork](docs/introduction/context-fork.md)

---

## See It In Action

<!-- Replace with actual asciinema recording -->
[![asciicast](https://asciinema.org/a/placeholder.svg)](https://asciinema.org/a/placeholder)

**Quick demos:**
- Creating a PRD <!-- demos/prd.gif -->
- Breaking down tasks <!-- demos/breakdown.gif -->
- Parallel execution <!-- demos/execute.gif -->

---

## Quick Start

### 1. Install

Copy the `.claude` directory to your project:

```bash
cp -r .claude /path/to/your/project/
```

### 2. Create a PRD

```bash
claude /prd
```

Follow the interactive 8-phase workflow to define your project requirements.

### 3. Break It Down

```bash
claude /breakdown docs/prd/your-project/index.md
```

Generates self-contained task XML files organized into 5 architectural layers.

### 4. Execute

```bash
claude /execute docs/tasks/your-project
```

Tasks execute in parallel using git worktrees, with TDD verification.

---

## What Gets Created

```
your-project/
├── docs/
│   ├── prd/
│   │   └── your-project/
│   │       ├── index.md              # Your PRD
│   │       ├── what-next.md          # Progress tracking
│   │       └── features/             # Feature details
│   │           ├── user-auth.md
│   │           └── dashboard.md
│   │
│   └── tasks/
│       └── your-project/
│           ├── analysis.json         # PRD analysis
│           ├── layer_plan.json       # Task organization
│           ├── manifest.json         # Execution inventory
│           ├── 0-setup/              # Layer 0: Project setup
│           │   └── L0-001-*.xml
│           ├── 1-foundation/         # Layer 1: Models, migrations
│           │   ├── L1-001-*.xml
│           │   └── L1-002-*.xml
│           ├── 2-backend/            # Layer 2: APIs, services
│           │   └── L2-*.xml
│           ├── 3-frontend/           # Layer 3: UI components
│           │   └── L3-*.xml
│           └── 4-integration/        # Layer 4: E2E wiring
│               └── L4-*.xml
│
└── src/                              # Your generated code
    └── ...
```

---

## Key Features

### Parallel Execution
Multiple tasks run simultaneously in isolated git worktrees. No blocking. No interference.

### TDD by Default
Every task writes tests first. Implementation follows. Independent verification confirms.

### Self-Healing
Failed tasks retry with actionable feedback. Up to 5 attempts before abandonment.

### Resume Anywhere
State management tracks every task. Stop and continue anytime. No progress lost.

### Model Specialization
- **Sonnet**: Orchestration, architecture, implementation
- **Haiku**: Fast verification, quality checks

### 5-Layer Architecture
```
Layer 4: Integration     ◄── E2E flows, wiring
         ▲
Layer 3: Frontend        ◄── UI components, state
         ▲
Layer 2: Backend         ◄── APIs, services, logic
         ▲
Layer 1: Foundation      ◄── Models, migrations, config
         ▲
Layer 0: Setup           ◄── Project initialization (greenfield only)
```

---

## Skill Hierarchy

```
/execute (Orchestrator)
    │
    └─► /execute-layer
            │
            └─► /execute-batch
                    │
                    ├─► /execute-task ──► /execute-verify
                    │       └── git worktree
                    ├─► /execute-task ──► /execute-verify
                    │       └── git worktree
                    └─► /execute-task ──► /execute-verify
                            └── git worktree
```

Tasks execute in parallel. Merge sequentially. No conflicts.

---

## Documentation

### Getting Started
- [Quick Start Guide](docs/quickstart/README.md) - Running in 5 minutes
- [Introduction](docs/introduction/README.md) - What and why
- [Context Fork Deep Dive](docs/introduction/context-fork.md) - The key innovation

### Skills Reference
- [Skills Overview](docs/skills/README.md) - All skills at a glance
- [/prd Command](docs/skills/prd.md) - Interactive PRD creation
- [/breakdown Skill](docs/skills/breakdown/README.md) - Task generation
- [/execute Skill](docs/skills/execute/README.md) - Parallel execution

### Deep Dives
- [Concepts](docs/concepts/README.md) - Core ideas explained
- [Full Example: Todo App](docs/examples/todo-app.md) - End-to-end walkthrough

### Reference
- [Commands](docs/reference/commands.md) - All commands and arguments
- [File Formats](docs/reference/file-formats.md) - XML and JSON schemas
- [Troubleshooting](docs/troubleshooting.md) - Common issues

---

## Requirements

- **Claude Code CLI** - The Anthropic CLI for Claude
- **Git** - For worktree-based parallel execution
- **Project Dependencies** - Varies by template (Python, Go, Node.js)

---

## Supported Templates

| Template | Stack | Setup Command |
|----------|-------|---------------|
| Python Backend | FastAPI + SQLAlchemy | `make setup` |
| Go Backend | Chi + sqlc | `make setup` |
| TanStack | TanStack Start + Drizzle | `npm install` |

---

## How It Works

### Phase 1: PRD Creation (`/prd`)
Interactive 8-phase workflow guides you through defining:
- Problem statement and target users
- Tech stack selection
- Features with MoSCoW prioritization
- Dependencies and integrations

### Phase 2: Task Breakdown (`/breakdown`)
1. **Analyze** - Extract features, infer data models and APIs
2. **Plan** - Organize into 5 architectural layers
3. **Generate** - Create self-contained task XML files
4. **Review** - Validate each task for completeness

### Phase 3: Parallel Execution (`/execute`)
1. **Orchestrate** - Process layers sequentially, tasks in parallel
2. **Implement** - Each task runs in isolated git worktree
3. **Verify** - Independent verification with Haiku
4. **Merge** - Sequential merge to main branch

---

## Example Task XML

```xml
<task>
  <meta>
    <id>L2-003</id>
    <name>Project CRUD API</name>
    <layer>2-backend</layer>
  </meta>

  <objective>
    Create REST API endpoints for project CRUD operations.
  </objective>

  <requirements>
    <requirement id="R1">
      Create file: src/api/projects.py
      Implement: GET /api/projects, POST /api/projects,
                 GET /api/projects/{id}, PUT /api/projects/{id},
                 DELETE /api/projects/{id}
    </requirement>
  </requirements>

  <test-requirements>
    <test id="T1">
      File: tests/api/test_projects.py
      Test: POST /api/projects returns 201 with valid payload
      Assert: response.status_code == 201
    </test>
  </test-requirements>

  <verification>
    <step>Run: pytest tests/api/test_projects.py -v</step>
    <step>Verify: All 5 endpoints respond correctly</step>
  </verification>

  <exports>
    <interface name="ProjectAPI" type="fastapi-router">
      router = APIRouter(prefix="/api/projects")
    </interface>
  </exports>
</task>
```

---

## State Management

Execution state is tracked in `execute-state.json`:

```json
{
  "status": "in_progress",
  "current_layer": "2-backend",
  "tasks": {
    "L2-001": { "status": "completed", "attempts": 1 },
    "L2-002": { "status": "completed", "attempts": 2 },
    "L2-003": { "status": "in_progress", "attempts": 1 }
  }
}
```

Resume after interruption:
```bash
claude /execute docs/tasks/your-project --resume
```

---

## Contributing

Contributions welcome! See [ARCHITECTURE.md](ARCHITECTURE.md) for system design.

---

## License

MIT

---

<p align="center">
  <sub>Built with Claude Code</sub>
</p>
