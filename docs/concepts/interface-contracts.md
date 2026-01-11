# Interface Contracts

How tasks communicate through type signatures.

---

## The Principle

**Tasks share type signatures, not implementations.**

```
Task L1-001 ───exports───► Interface Contract ───depends───► Task L2-001
                                │
                          Type signatures
                          NOT implementation
```

---

## What Is an Interface Contract?

A complete type signature with:
- All imports
- Class/function definitions
- Type annotations
- Docstrings (for complex interfaces)

### Example

Task L1-001 exports:

```xml
<exports>
  <interface name="Task" type="sqlalchemy-model">
    from sqlalchemy.orm import Mapped, mapped_column
    from sqlalchemy import String, Enum
    from uuid import UUID
    from datetime import datetime
    from app.models.base import Base
    from app.models.enums import TaskStatus

    class Task(Base):
        """Task entity for the task management system."""

        __tablename__ = "tasks"

        id: Mapped[UUID] = mapped_column(primary_key=True)
        title: Mapped[str] = mapped_column(String(200))
        description: Mapped[str | None] = mapped_column(String(2000))
        status: Mapped[TaskStatus] = mapped_column(Enum(TaskStatus))
        assignee_id: Mapped[UUID | None] = mapped_column(nullable=True)
        created_at: Mapped[datetime]
        updated_at: Mapped[datetime | None]
  </interface>
</exports>
```

Task L2-001 receives as dependency:

```xml
<dependencies>
  <interface name="Task" type="sqlalchemy-model">
    <!-- Same content as L1-001's export -->
    from sqlalchemy.orm import Mapped, mapped_column
    ...
    class Task(Base):
        ...
  </interface>
</dependencies>
```

---

## Why Not Share Implementation?

### Problem: Context Pollution

If L2-001 saw L1-001's full implementation:

```
L1-001's context:
  - Multiple attempts at model design
  - Error messages from failed migrations
  - Internal comments and TODOs
  - Implementation details not relevant to L2-001
```

This would:
- Fill context window with irrelevant info
- Confuse the implementing agent
- Create false dependencies

### Solution: Clean Boundaries

With interface contracts:

```
L2-001 sees:
  - Type signature only
  - Exactly what it needs to call/use
  - No implementation noise
```

---

## Contract Types

### Model Contracts

```xml
<interface name="Task" type="sqlalchemy-model">
  from sqlalchemy.orm import Mapped
  from uuid import UUID

  class Task(Base):
      __tablename__ = "tasks"
      id: Mapped[UUID]
      title: Mapped[str]
      status: Mapped[TaskStatus]
</interface>
```

### Repository Contracts

```xml
<interface name="TaskRepository" type="repository">
  from uuid import UUID
  from app.models.task import Task

  class TaskRepository:
      async def create(self, task: Task) -> Task:
          """Create a new task."""
          ...

      async def get_by_id(self, id: UUID) -> Task | None:
          """Get task by ID, or None if not found."""
          ...

      async def list_all(self) -> list[Task]:
          """List all tasks."""
          ...
</interface>
```

### Service Contracts

```xml
<interface name="TaskService" type="service">
  from uuid import UUID
  from app.models.task import Task
  from app.schemas.task import TaskCreate, TaskUpdate

  class TaskService:
      async def create_task(self, data: TaskCreate) -> Task:
          """Create task with business validation."""
          ...

      async def complete_task(self, id: UUID) -> Task:
          """Mark task as complete. Raises NotFoundError."""
          ...
</interface>
```

### API Contracts

```xml
<interface name="TaskRouter" type="fastapi-router">
  from fastapi import APIRouter

  router = APIRouter(prefix="/api/tasks", tags=["tasks"])

  # Endpoints:
  # GET /           -> TaskListResponse
  # POST /          -> TaskResponse (201)
  # GET /{id}       -> TaskResponse (404)
  # PUT /{id}       -> TaskResponse (404)
  # DELETE /{id}    -> 204 (404)
</interface>
```

### Schema Contracts

```xml
<interface name="TaskSchemas" type="pydantic-models">
  from pydantic import BaseModel, Field
  from uuid import UUID
  from datetime import datetime
  from app.models.enums import TaskStatus

  class TaskCreate(BaseModel):
      title: str = Field(min_length=1, max_length=200)
      description: str | None = None

  class TaskUpdate(BaseModel):
      title: str | None = None
      status: TaskStatus | None = None

  class TaskResponse(BaseModel):
      id: UUID
      title: str
      status: TaskStatus
      created_at: datetime

      class Config:
          from_attributes = True
</interface>
```

---

## Flow Between Layers

```
Layer 1 (Foundation)
    │
    │ Exports: Model contracts
    │   - Task, User, Team models
    │   - Enum types
    │
    ▼
Layer 2 (Backend)
    │
    │ Depends: Model contracts
    │ Exports: Repository, Service, API contracts
    │   - TaskRepository interface
    │   - TaskService interface
    │   - TaskRouter interface
    │   - Pydantic schemas
    │
    ▼
Layer 3 (Frontend)
    │
    │ Depends: API contracts (schema shapes only)
    │   - TaskResponse shape
    │   - TaskCreate shape
    │
    │ Does NOT depend on:
    │   - SQLAlchemy models
    │   - Repository implementation
    │
    ▼
Layer 4 (Integration)
    │
    │ Depends: All previous contracts
    │ Wires everything together
```

---

## Contract Requirements

### 1. Complete Imports

```xml
<!-- BAD: Missing imports -->
<interface name="Task" type="model">
  class Task(Base):
      id: Mapped[UUID]
</interface>

<!-- GOOD: All imports included -->
<interface name="Task" type="model">
  from sqlalchemy.orm import Mapped
  from uuid import UUID
  from app.models.base import Base

  class Task(Base):
      id: Mapped[UUID]
</interface>
```

### 2. Full Type Annotations

```xml
<!-- BAD: Missing types -->
<interface name="TaskRepository" type="repository">
  class TaskRepository:
      async def get_by_id(self, id): ...
</interface>

<!-- GOOD: Complete types -->
<interface name="TaskRepository" type="repository">
  from uuid import UUID
  from app.models.task import Task

  class TaskRepository:
      async def get_by_id(self, id: UUID) -> Task | None: ...
</interface>
```

### 3. Method Signatures Only

```xml
<!-- BAD: Implementation included -->
<interface name="TaskRepository" type="repository">
  class TaskRepository:
      async def get_by_id(self, id: UUID) -> Task | None:
          query = select(Task).where(Task.id == id)
          result = await self.session.execute(query)
          return result.scalar_one_or_none()
</interface>

<!-- GOOD: Signature only -->
<interface name="TaskRepository" type="repository">
  class TaskRepository:
      async def get_by_id(self, id: UUID) -> Task | None:
          """Get task by ID, or None if not found."""
          ...
</interface>
```

---

## Contract Attributes

Each interface has:

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `name` | Identifier | `"Task"` |
| `type` | Category | `"sqlalchemy-model"` |

Common types:
- `sqlalchemy-model`
- `pydantic-model`
- `repository`
- `service`
- `fastapi-router`
- `enum`
- `function`

---

## Benefits

### 1. Decoupling

Frontend doesn't know about SQLAlchemy. Backend doesn't know about React.

### 2. Parallel Development

Teams can work on different layers with agreed contracts.

### 3. Independent Testing

Mock dependencies using contract signatures.

### 4. Clean Context

Each task sees only what it needs.

### 5. Explicit Dependencies

No hidden assumptions - everything documented.

---

## Next Steps

- [Self-Contained Tasks](self-contained-tasks.md) - Why containment matters
- [Layer Architecture](../skills/breakdown/layers.md) - Layer boundaries
- [Task Format](../skills/breakdown/task-format.md) - Full specification
