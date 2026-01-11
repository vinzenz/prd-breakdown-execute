# Self-Contained Tasks

Why every task must include all context inline.

---

## The Principle

**Every task file contains everything needed for autonomous execution.**

The implementing agent:
- Cannot ask questions
- Cannot look up external files
- Cannot access other task context
- Must work from task XML alone

---

## Why Self-Containment?

### 1. Context Window Limits

Small, efficient models (like Haiku) have limited context:

```
Task XML + Instructions ≈ 50k tokens max

Includes:
- Task specification
- Interface contracts
- Requirements
- Test cases
- Verification steps

No room for:
- Entire codebase
- All PRD sections
- Other task contexts
```

Self-containment ensures the task fits.

### 2. Parallel Execution

In forked context, tasks cannot share information:

```
Fork A: Task L2-001
        └── Only sees L2-001.xml

Fork B: Task L2-002
        └── Only sees L2-002.xml

Cannot communicate between forks.
```

Everything needed must be in the XML.

### 3. Autonomous Execution

The implementing agent runs without supervision:

```
Agent starts
    │
    ├── Read task XML ✓
    │
    ├── Ask clarifying question? ✗ Not possible
    │
    ├── Look up external file? ✗ Not possible
    │
    └── Must complete from XML alone
```

No opportunity for clarification.

### 4. Reproducibility

Same task XML → Same implementation:

```
Run 1: Task L2-001.xml → Implementation A
Run 2: Task L2-001.xml → Implementation A

No external dependencies that could change.
```

---

## What Self-Containment Means

### Include Everything

```xml
<task>
  <context>
    <!-- PRD excerpt - just the relevant parts -->
    <prd-excerpt>
      Users can create tasks with title (required, max 200 chars)
      and description (optional). Tasks have status: pending,
      in_progress, or done.
    </prd-excerpt>

    <!-- Tech stack -->
    <tech-stack>
      Python 3.12, FastAPI 0.109, SQLAlchemy 2.0
    </tech-stack>
  </context>

  <dependencies>
    <!-- Full interface contract - not "see Task model" -->
    <interface name="Task" type="model">
      from sqlalchemy.orm import Mapped
      from uuid import UUID

      class Task(Base):
          id: Mapped[UUID]
          title: Mapped[str]
          status: Mapped[TaskStatus]
    </interface>
  </dependencies>
</task>
```

### No References

```xml
<!-- BAD: References external file -->
<dependencies>
  See src/models/task.py for Task model definition.
</dependencies>

<!-- GOOD: Complete definition inline -->
<dependencies>
  <interface name="Task" type="model">
    class Task(Base):
        id: Mapped[UUID]
        title: Mapped[str]
        ...complete definition...
  </interface>
</dependencies>
```

### No Assumptions

```xml
<!-- BAD: Assumes knowledge -->
<requirements>
  Implement the Task API using standard REST patterns.
</requirements>

<!-- GOOD: Explicit specification -->
<requirements>
  <requirement id="R1">
    Create file: src/api/tasks.py

    Endpoints:
      GET /api/tasks - List all tasks
      POST /api/tasks - Create task (201)
      GET /api/tasks/{id} - Get task (404 if not found)
      PUT /api/tasks/{id} - Update task
      DELETE /api/tasks/{id} - Delete task (204)
  </requirement>
</requirements>
```

---

## The Complete Task

A self-contained task includes:

| Section | Contains |
|---------|----------|
| `<meta>` | Task identification |
| `<context>` | PRD excerpt, tech stack |
| `<dependencies>` | Interface contracts from previous tasks |
| `<objective>` | Clear goal statement |
| `<requirements>` | Detailed specifications |
| `<test-requirements>` | Complete test cases |
| `<files-to-create>` | Exact file paths |
| `<verification>` | Runnable commands |
| `<exports>` | Interface contracts for downstream |

Nothing references external files or assumes implicit knowledge.

---

## Interface Contracts

The key to self-containment between tasks:

### Dependencies: What I Need

```xml
<dependencies>
  <interface name="TaskRepository" type="repository">
    from uuid import UUID
    from app.models.task import Task

    class TaskRepository:
        async def create(self, task: Task) -> Task: ...
        async def get_by_id(self, id: UUID) -> Task | None: ...
        async def list_all(self) -> list[Task]: ...
  </interface>
</dependencies>
```

Complete type signatures with imports. Not "the repository class."

### Exports: What I Provide

```xml
<exports>
  <interface name="TaskRouter" type="router">
    from fastapi import APIRouter

    router = APIRouter(prefix="/api/tasks")
    # GET /, POST /, GET /{id}, PUT /{id}, DELETE /{id}
  </interface>
</exports>
```

This becomes dependencies for downstream tasks.

---

## Validation Rules

The review process enforces self-containment:

### No Placeholders

```
Rejected: TODO, TBD, ..., [fill in], etc.
Rejected: "add appropriate validation"
Rejected: "use suitable error handling"
```

### No External References

```
Rejected: "See the User model for field definitions"
Rejected: "Follow the pattern in other API files"
Rejected: "Refer to the PRD for details"
```

### Complete Type Annotations

```
Rejected: async def get(...) -> Any
Required: async def get(id: UUID) -> Task | None
```

### All Imports Included

```
Rejected:
  class Task(Base):
      id: Mapped[UUID]

Required:
  from sqlalchemy.orm import Mapped
  from uuid import UUID
  from app.models.base import Base

  class Task(Base):
      id: Mapped[UUID]
```

---

## Benefits

### 1. Reliable Parallel Execution

Tasks don't depend on shared state.

### 2. Easy Debugging

Task XML is complete specification - if it fails, the spec is wrong.

### 3. Reproducible Results

Same input → Same output.

### 4. Model Flexibility

Works with any model that fits the context window.

### 5. Clear Boundaries

Each task is a complete unit of work.

---

## Next Steps

- [Interface Contracts](interface-contracts.md) - How tasks communicate
- [Task Format](../skills/breakdown/task-format.md) - Complete specification
- [Context Fork](../introduction/context-fork.md) - Why isolation matters
