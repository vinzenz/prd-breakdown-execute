# Task XML Format Specification

Complete reference for the self-contained task file format.

---

## Overview

Each task is an XML file containing everything needed for autonomous implementation. The format is designed for:

- **Self-containment:** All context inline, no external lookups
- **Small context windows:** ~50k tokens (optimized for Haiku)
- **TDD approach:** Tests defined before implementation
- **Clear verification:** Runnable commands with expected outcomes

---

## Complete Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<task>
  <meta>
    <!-- Task identification -->
  </meta>

  <context>
    <!-- Background information -->
  </context>

  <dependencies>
    <!-- Interface contracts from previous tasks -->
  </dependencies>

  <objective>
    <!-- What to achieve (1-3 sentences) -->
  </objective>

  <requirements>
    <!-- Detailed specifications -->
  </requirements>

  <test-requirements>
    <!-- TDD test cases -->
  </test-requirements>

  <files-to-create>
    <!-- File paths (max 3) -->
  </files-to-create>

  <verification>
    <!-- Runnable verification steps -->
  </verification>

  <exports>
    <!-- Interface contracts for downstream tasks -->
  </exports>
</task>
```

---

## Section Details

### Meta Section

```xml
<meta>
  <id>L2-003</id>
  <name>Task API Endpoints</name>
  <layer>2-backend</layer>
  <priority>1</priority>
  <estimated-files>2</estimated-files>
</meta>
```

| Field | Description |
|-------|-------------|
| `id` | Unique identifier: `L{layer}-{sequence}` |
| `name` | Human-readable task name |
| `layer` | Layer identifier: `{n}-{name}` |
| `priority` | Execution order within layer (1 = first) |
| `estimated-files` | Expected number of files to create |

---

### Context Section

```xml
<context>
  <prd-excerpt>
    Users can create, view, update, and delete tasks.
    Each task has title, description, status, and assignee.
    Tasks can be filtered by status and assigned user.
  </prd-excerpt>

  <tech-stack>
    Python 3.12, FastAPI 0.109, SQLAlchemy 2.0, PostgreSQL 16
  </tech-stack>

  <template-base>webapps/backends/python</template-base>

  <project-structure>
    src/
    ├── models/
    ├── repositories/
    ├── services/
    └── api/
  </project-structure>
</context>
```

| Field | Required | Description |
|-------|----------|-------------|
| `prd-excerpt` | Yes | Relevant PRD sections (focused, not entire PRD) |
| `tech-stack` | Yes | Technologies and versions |
| `template-base` | Optional | Template path for greenfield |
| `project-structure` | Optional | Directory structure |

---

### Dependencies Section

```xml
<dependencies>
  <interface name="Task" type="sqlalchemy-model">
    from sqlalchemy.orm import Mapped
    from uuid import UUID
    from datetime import datetime
    from app.models.base import Base
    from app.models.enums import TaskStatus

    class Task(Base):
        __tablename__ = "tasks"
        id: Mapped[UUID]
        title: Mapped[str]
        description: Mapped[str | None]
        status: Mapped[TaskStatus]
        assignee_id: Mapped[UUID | None]
        created_at: Mapped[datetime]
  </interface>

  <interface name="TaskRepository" type="repository">
    from uuid import UUID
    from app.models.task import Task

    class TaskRepository:
        async def create(self, task: Task) -> Task: ...
        async def get_by_id(self, id: UUID) -> Task | None: ...
        async def list_all(self) -> list[Task]: ...
        async def update(self, task: Task) -> Task: ...
        async def delete(self, id: UUID) -> bool: ...
  </interface>
</dependencies>
```

**Critical requirements:**

| Requirement | Description |
|-------------|-------------|
| `name` attribute | Interface identifier |
| `type` attribute | Category (model, repository, service, etc.) |
| Complete imports | All imports included in code block |
| Type annotations | Full type signatures, not `Any` |
| Docstrings | For complex interfaces |

**This is NOT implementation code** - just signatures that define the contract.

---

### Objective Section

```xml
<objective>
  Create REST API endpoints for task CRUD operations using FastAPI,
  implementing all five standard operations with proper error handling
  and Pydantic validation.
</objective>
```

**Rules:**
- 1-3 sentences only
- Focus on WHAT, not HOW
- Clear value statement

---

### Requirements Section

```xml
<requirements>
  <requirement id="R1">
    Create file: src/api/tasks.py

    Define APIRouter with prefix "/api/tasks" and tag "tasks"

    Endpoints:
      GET /
        - Returns TaskListResponse with all tasks
        - Query params: status (optional), assignee_id (optional)

      POST /
        - Accepts TaskCreateRequest body
        - Returns TaskResponse with status 201
        - Raises 422 if validation fails

      GET /{id}
        - Path param: id (UUID)
        - Returns TaskResponse
        - Raises 404 if not found

      PUT /{id}
        - Path param: id (UUID)
        - Accepts TaskUpdateRequest body
        - Returns updated TaskResponse
        - Raises 404 if not found

      DELETE /{id}
        - Path param: id (UUID)
        - Returns 204 No Content
        - Raises 404 if not found

    Use dependency injection for TaskRepository.
  </requirement>

  <requirement id="R2">
    Create file: src/api/schemas/task.py

    Pydantic models:

      TaskCreateRequest:
        title: str (min_length=1, max_length=200)
        description: str | None = None

      TaskUpdateRequest:
        title: str | None = None
        description: str | None = None
        status: TaskStatus | None = None

      TaskResponse:
        id: UUID
        title: str
        description: str | None
        status: TaskStatus
        assignee_id: UUID | None
        created_at: datetime

        class Config:
            from_attributes = True

      TaskListResponse:
        tasks: list[TaskResponse]
        count: int
  </requirement>
</requirements>
```

**Critical requirements:**

| Requirement | Bad Example | Good Example |
|-------------|-------------|--------------|
| Exact paths | "in the api folder" | `src/api/tasks.py` |
| Exact names | "appropriate validation" | `min_length=1, max_length=200` |
| Exact types | "proper type" | `UUID`, `str | None` |
| No placeholders | `TODO`, `...`, `etc.` | Complete specification |

---

### Test Requirements Section

```xml
<test-requirements>
  <test id="T1">
    File: tests/api/test_tasks.py

    Test: test_create_task_success
    Setup:
      - Create mock TaskRepository
      - Mock create() to return Task(id=uuid4(), title="Test", ...)
      - Create TestClient with app
    Action:
      - POST /api/tasks with {"title": "Test task", "description": "A test"}
    Assert:
      - response.status_code == 201
      - response.json()["title"] == "Test task"
      - response.json()["id"] is valid UUID
  </test>

  <test id="T2">
    Test: test_create_task_empty_title
    Setup:
      - Create TestClient with app
    Action:
      - POST /api/tasks with {"title": ""}
    Assert:
      - response.status_code == 422
      - "title" in response.json()["detail"][0]["loc"]
  </test>

  <test id="T3">
    Test: test_get_task_not_found
    Setup:
      - Create mock TaskRepository
      - Mock get_by_id() to return None
    Action:
      - GET /api/tasks/{random_uuid}
    Assert:
      - response.status_code == 404
  </test>
</test-requirements>
```

**Critical requirements:**
- Concrete test values (not "some value")
- Clear setup, action, assertion structure
- Test file path specified
- Each test has unique ID

---

### Files to Create Section

```xml
<files-to-create>
  <file>src/api/tasks.py</file>
  <file>src/api/schemas/task.py</file>
</files-to-create>
```

**Rules:**
- Maximum 3 files (excluding test file)
- Full paths from project root
- Listed in creation order

---

### Verification Section

```xml
<verification>
  <step>Run: pytest tests/api/test_tasks.py -v</step>
  <step>Verify: All 5 tests pass</step>
  <step>Run: python -c "from src.api.tasks import router"</step>
  <step>Verify: Import succeeds without error</step>
  <step>Check: router.routes contains 5 routes</step>
</verification>
```

**Step types:**

| Type | Description |
|------|-------------|
| `Run:` | Execute command, check exit code |
| `Verify:` | Assert expected outcome |
| `Check:` | Validate condition |

All steps must be runnable without human intervention.

---

### Exports Section

```xml
<exports>
  <interface name="TaskRouter" type="fastapi-router">
    from fastapi import APIRouter

    router = APIRouter(prefix="/api/tasks", tags=["tasks"])

    # Routes:
    # GET / -> TaskListResponse
    # POST / -> TaskResponse (201)
    # GET /{id} -> TaskResponse
    # PUT /{id} -> TaskResponse
    # DELETE /{id} -> 204
  </interface>

  <interface name="TaskSchemas" type="pydantic-models">
    from pydantic import BaseModel
    from uuid import UUID
    from datetime import datetime

    class TaskCreateRequest(BaseModel): ...
    class TaskUpdateRequest(BaseModel): ...
    class TaskResponse(BaseModel): ...
    class TaskListResponse(BaseModel): ...
  </interface>
</exports>
```

These become `<dependencies>` for downstream tasks.

---

## Validation Rules

### Critical (Task Fails)

1. **No empty sections** - All required sections must have content
2. **No placeholders** - `TODO`, `TBD`, `...`, `[fill in]`, `etc.` not allowed
3. **Complete imports** - Every interface must include all imports
4. **Full type annotations** - No `Any`, no missing types
5. **Specific values** - No "appropriate", "suitable", "proper"
6. **Max 3 files** - Excluding test file
7. **Runnable verification** - Commands must execute

### Warnings (Noted but Pass)

1. **Context relevance** - PRD excerpt should be focused
2. **Dependency necessity** - Only include used dependencies
3. **Objective clarity** - Should be 1-3 sentences

---

## Complete Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<task>
  <meta>
    <id>L2-003</id>
    <name>Task API Endpoints</name>
    <layer>2-backend</layer>
    <priority>1</priority>
    <estimated-files>2</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
      Users can create, view, update, and delete tasks.
      Each task has title (required, max 200 chars),
      description (optional), status (pending/in_progress/done),
      and optional assignee.
    </prd-excerpt>
    <tech-stack>
      Python 3.12, FastAPI 0.109, SQLAlchemy 2.0, Pydantic 2.0
    </tech-stack>
  </context>

  <dependencies>
    <interface name="Task" type="sqlalchemy-model">
      from sqlalchemy.orm import Mapped
      from uuid import UUID
      from app.models.base import Base
      from app.models.enums import TaskStatus

      class Task(Base):
          __tablename__ = "tasks"
          id: Mapped[UUID]
          title: Mapped[str]
          description: Mapped[str | None]
          status: Mapped[TaskStatus]
    </interface>

    <interface name="TaskRepository" type="repository">
      from uuid import UUID
      from app.models.task import Task

      class TaskRepository:
          async def create(self, task: Task) -> Task: ...
          async def get_by_id(self, id: UUID) -> Task | None: ...
          async def list_all(self) -> list[Task]: ...
    </interface>
  </dependencies>

  <objective>
    Create REST API endpoints for task CRUD operations.
  </objective>

  <requirements>
    <requirement id="R1">
      Create file: src/api/tasks.py

      Implement FastAPI router with:
      - GET / returning list of tasks
      - POST / creating new task (201)
      - GET /{id} returning single task (404 if not found)
      - DELETE /{id} removing task (204 or 404)
    </requirement>
  </requirements>

  <test-requirements>
    <test id="T1">
      File: tests/api/test_tasks.py
      Test: test_create_task
      Action: POST /api/tasks with {"title": "Test"}
      Assert: status_code == 201
    </test>
  </test-requirements>

  <files-to-create>
    <file>src/api/tasks.py</file>
  </files-to-create>

  <verification>
    <step>Run: pytest tests/api/test_tasks.py -v</step>
    <step>Verify: All tests pass</step>
  </verification>

  <exports>
    <interface name="TaskRouter" type="fastapi-router">
      from fastapi import APIRouter
      router = APIRouter(prefix="/api/tasks")
    </interface>
  </exports>
</task>
```

---

## Next Steps

- [Layer Architecture](layers.md)
- [Sub-skills Reference](sub-skills.md)
- [Execute Skill](../execute/README.md)
