# Example: Todo App

A complete walkthrough building a task management API.

---

## The Idea

A simple REST API for managing todos with:
- Create, read, update, delete operations
- Status tracking (pending, in_progress, done)
- Priority levels
- Due dates

---

## Phase 1: PRD Creation

```bash
claude /prd
```

### Idea Capture

```
What problem are you solving?
> Personal task management - I need a simple API to track my todos
> with status, priority, and due dates.

Who are the target users?
> Developers who want a simple backend for todo apps.

What's the core value proposition?
> A clean, well-tested REST API that can be used as a backend
> for any todo frontend.
```

### Tech Stack

```
Is this a new project or existing?
> Greenfield (new project)

What technologies do you prefer?
> Python, something simple and fast

Recommended stack:
┌─────────────────────────────────────────┐
│ Python 3.12 + FastAPI + SQLite         │
│                                         │
│ - FastAPI: Fast, modern API framework   │
│ - SQLite: Simple, file-based database   │
│ - SQLAlchemy: ORM with async support    │
│ - Pydantic: Request/response validation │
└─────────────────────────────────────────┘
```

### Features

```
Must-Have:
  - Todo CRUD operations
  - Status tracking (pending, in_progress, done)
  - Priority levels (low, medium, high)

Should-Have:
  - Due date support
  - Filtering by status and priority

Could-Have:
  - Tags/labels
  - Search

Won't-Have:
  - User authentication (single-user for now)
  - Real-time updates
```

### Generated PRD

```
docs/prd/todo-api/
├── index.md
├── what-next.md
└── features/
    ├── todo-crud.md
    ├── status-tracking.md
    └── priority-levels.md
```

---

## Phase 2: PRD Content

### index.md (excerpt)

```xml
<prd>
  <meta>
    <name>Todo API</name>
    <slug>todo-api</slug>
    <status>complete</status>
  </meta>

  <overview>
    <problem-statement>
      Need a simple REST API for personal task management with
      status tracking, priorities, and due dates.
    </problem-statement>
    <target-users>
      Developers building todo app frontends.
    </target-users>
    <value-proposition>
      Clean, well-tested REST API backend for todo applications.
    </value-proposition>
  </overview>

  <tech-stack>
    <project-type>greenfield</project-type>
    <technologies>
      <technology name="Python" version="3.12" />
      <technology name="FastAPI" version="0.109" />
      <technology name="SQLAlchemy" version="2.0" />
      <technology name="SQLite" version="3" />
    </technologies>
  </tech-stack>

  <features>
    <feature slug="todo-crud" priority="must-have">
      Todo CRUD Operations
    </feature>
    <feature slug="status-tracking" priority="must-have">
      Status Tracking
    </feature>
    <feature slug="priority-levels" priority="must-have">
      Priority Levels
    </feature>
    <feature slug="due-dates" priority="should-have">
      Due Date Support
    </feature>
    <feature slug="filtering" priority="should-have">
      Filtering
    </feature>
  </features>
</prd>
```

### features/todo-crud.md

```xml
<feature>
  <meta>
    <name>Todo CRUD Operations</name>
    <slug>todo-crud</slug>
    <priority>must-have</priority>
  </meta>

  <description>
    Create, read, update, and delete todo items.
    Each todo has: title (required), description (optional),
    status, priority, and timestamps.
  </description>

  <acceptance-criteria>
    <criterion id="AC1">
      <given>A user wants to create a todo</given>
      <when>They POST to /api/todos with title "Buy groceries"</when>
      <then>A todo is created with status "pending" and priority "medium"</then>
    </criterion>
    <criterion id="AC2">
      <given>A todo with ID 123 exists</given>
      <when>They GET /api/todos/123</when>
      <then>The todo details are returned</then>
    </criterion>
    <criterion id="AC3">
      <given>A todo with ID 123 exists</given>
      <when>They DELETE /api/todos/123</when>
      <then>The todo is removed and 204 is returned</then>
    </criterion>
  </acceptance-criteria>
</feature>
```

---

## Phase 3: Breakdown

```bash
claude /breakdown docs/prd/todo-api/index.md --output-dir ./todo-api
```

### Analysis Output

```
Analyzing PRD...

Features: 5
  - todo-crud (must-have)
  - status-tracking (must-have)
  - priority-levels (must-have)
  - due-dates (should-have)
  - filtering (should-have)

Inferred models:
  - Todo (id, title, description, status, priority, due_date, created_at)

Inferred endpoints:
  - GET /api/todos (list, with filters)
  - POST /api/todos (create)
  - GET /api/todos/{id} (get one)
  - PUT /api/todos/{id} (update)
  - DELETE /api/todos/{id} (delete)

Template: webapps/backends/python
```

### Layer Plan

```
Layer 0: Setup (3 tasks)
  L0-001: Project initialization
  L0-002: Database configuration
  L0-003: Initial structure

Layer 1: Foundation (3 tasks)
  L1-001: Todo model with enums
  L1-002: Database migrations
  L1-003: Base repository

Layer 2: Backend (4 tasks)
  L2-001: Todo repository
  L2-002: Todo service
  L2-003: Todo API endpoints
  L2-004: Filtering logic

Layer 3: Frontend (0 tasks)
  - API-only project

Layer 4: Integration (1 task)
  L4-001: Error handling and OpenAPI docs

Total: 11 tasks
```

---

## Phase 4: Example Task

### L2-003: Todo API Endpoints

```xml
<task>
  <meta>
    <id>L2-003</id>
    <name>Todo API Endpoints</name>
    <layer>2-backend</layer>
    <priority>1</priority>
  </meta>

  <context>
    <prd-excerpt>
      Create, read, update, delete todos.
      Each todo: title (required), description (optional),
      status (pending/in_progress/done), priority (low/medium/high),
      due_date (optional datetime).
    </prd-excerpt>
    <tech-stack>Python 3.12, FastAPI 0.109, SQLAlchemy 2.0</tech-stack>
  </context>

  <dependencies>
    <interface name="Todo" type="sqlalchemy-model">
      from sqlalchemy.orm import Mapped
      from uuid import UUID
      from datetime import datetime
      from app.models.base import Base
      from app.models.enums import TodoStatus, Priority

      class Todo(Base):
          __tablename__ = "todos"
          id: Mapped[UUID]
          title: Mapped[str]
          description: Mapped[str | None]
          status: Mapped[TodoStatus]
          priority: Mapped[Priority]
          due_date: Mapped[datetime | None]
          created_at: Mapped[datetime]
    </interface>

    <interface name="TodoRepository" type="repository">
      from uuid import UUID
      from app.models.todo import Todo

      class TodoRepository:
          async def create(self, todo: Todo) -> Todo: ...
          async def get_by_id(self, id: UUID) -> Todo | None: ...
          async def list_all(self, status: str = None) -> list[Todo]: ...
          async def update(self, todo: Todo) -> Todo: ...
          async def delete(self, id: UUID) -> bool: ...
    </interface>
  </dependencies>

  <objective>
    Create REST API endpoints for todo CRUD operations.
  </objective>

  <requirements>
    <requirement id="R1">
      Create file: src/api/todos.py

      FastAPI router with prefix "/api/todos":

      GET /
        Query params: status (optional), priority (optional)
        Returns: TodoListResponse

      POST /
        Body: TodoCreateRequest
        Returns: TodoResponse (201)

      GET /{id}
        Returns: TodoResponse (404 if not found)

      PUT /{id}
        Body: TodoUpdateRequest
        Returns: TodoResponse (404 if not found)

      DELETE /{id}
        Returns: 204 (404 if not found)
    </requirement>

    <requirement id="R2">
      Create file: src/api/schemas/todo.py

      Pydantic models:
        TodoCreateRequest:
          title: str (min 1, max 200)
          description: str | None = None
          priority: Priority = Priority.MEDIUM
          due_date: datetime | None = None

        TodoUpdateRequest:
          title: str | None = None
          description: str | None = None
          status: TodoStatus | None = None
          priority: Priority | None = None
          due_date: datetime | None = None

        TodoResponse:
          id: UUID
          title: str
          description: str | None
          status: TodoStatus
          priority: Priority
          due_date: datetime | None
          created_at: datetime
    </requirement>
  </requirements>

  <test-requirements>
    <test id="T1">
      File: tests/api/test_todos.py

      Test: test_create_todo
      Action: POST /api/todos {"title": "Test todo"}
      Assert: status 201, title matches, status is "pending"
    </test>

    <test id="T2">
      Test: test_get_todo_not_found
      Action: GET /api/todos/{random_uuid}
      Assert: status 404
    </test>

    <test id="T3">
      Test: test_list_todos_with_filter
      Setup: Create todos with different statuses
      Action: GET /api/todos?status=pending
      Assert: Only pending todos returned
    </test>
  </test-requirements>

  <files-to-create>
    <file>src/api/todos.py</file>
    <file>src/api/schemas/todo.py</file>
  </files-to-create>

  <verification>
    <step>Run: pytest tests/api/test_todos.py -v</step>
    <step>Verify: All tests pass</step>
  </verification>

  <exports>
    <interface name="TodoRouter" type="fastapi-router">
      from fastapi import APIRouter
      router = APIRouter(prefix="/api/todos", tags=["todos"])
    </interface>
  </exports>
</task>
```

---

## Phase 5: Execution

```bash
claude /execute docs/tasks/todo-api
```

### Execution Log

```
Starting execution...

╔════════════════════════════════════════════════════════════════╗
║ Layer 0: Setup                                                 ║
╚════════════════════════════════════════════════════════════════╝

L0-001: Project initialization
  Creating worktree: .worktrees/L0-001
  Writing pyproject.toml...
  Creating src/ structure...
  ✓ Verified (attempt 1)

L0-002: Database configuration
  ...
  ✓ Verified (attempt 1)

L0-003: Initial structure
  ...
  ✓ Verified (attempt 1)

Merging to main...
  Merge L0-001 ✓
  Merge L0-002 ✓
  Merge L0-003 ✓

Layer 0 complete: 3/3 tasks

╔════════════════════════════════════════════════════════════════╗
║ Layer 1: Foundation                                            ║
╚════════════════════════════════════════════════════════════════╝

L1-001: Todo model with enums
  ...
  ✓ Verified (attempt 1)

L1-002: Database migrations
  ...
  ✓ Verified (attempt 2)  # Retry after migration fix

L1-003: Base repository
  ...
  ✓ Verified (attempt 1)

Layer 1 complete: 3/3 tasks

╔════════════════════════════════════════════════════════════════╗
║ Layer 2: Backend                                               ║
╚════════════════════════════════════════════════════════════════╝

Batch 1: L2-001, L2-002 (parallel)
  L2-001: [████████████████████] ✓ Verified
  L2-002: [████████████████████] ✓ Verified

Batch 2: L2-003, L2-004 (parallel)
  L2-003: [████████████████████] ✓ Verified
  L2-004: [████████████████████] ✓ Verified

Layer 2 complete: 4/4 tasks

╔════════════════════════════════════════════════════════════════╗
║ Layer 4: Integration                                           ║
╚════════════════════════════════════════════════════════════════╝

L4-001: Error handling and docs
  ...
  ✓ Verified (attempt 1)

Layer 4 complete: 1/1 tasks

╔════════════════════════════════════════════════════════════════╗
║                      Execution Complete                        ║
╠════════════════════════════════════════════════════════════════╣
║  Tasks: 11/11 completed                                        ║
║  Retries: 1 (L1-002)                                           ║
║  Time: 4m 32s                                                  ║
╚════════════════════════════════════════════════════════════════╝
```

---

## Phase 6: Final Result

### Project Structure

```
todo-api/
├── pyproject.toml
├── alembic.ini
├── src/
│   ├── __init__.py
│   ├── main.py                 # FastAPI app
│   ├── config.py               # Configuration
│   ├── database.py             # Database setup
│   ├── models/
│   │   ├── __init__.py
│   │   ├── base.py             # SQLAlchemy base
│   │   ├── todo.py             # Todo model
│   │   └── enums.py            # TodoStatus, Priority
│   ├── repositories/
│   │   ├── __init__.py
│   │   ├── base.py             # Base repository
│   │   └── todo.py             # Todo repository
│   ├── services/
│   │   ├── __init__.py
│   │   └── todo.py             # Todo service
│   └── api/
│       ├── __init__.py
│       ├── todos.py            # Todo endpoints
│       └── schemas/
│           ├── __init__.py
│           └── todo.py         # Pydantic schemas
├── tests/
│   ├── __init__.py
│   ├── conftest.py             # Test fixtures
│   ├── models/
│   │   └── test_todo.py
│   ├── repositories/
│   │   └── test_todo.py
│   └── api/
│       └── test_todos.py
└── alembic/
    └── versions/
        └── 001_initial.py
```

### Running the API

```bash
cd todo-api

# Install dependencies
pip install -e .

# Run migrations
alembic upgrade head

# Start server
uvicorn src.main:app --reload

# Test endpoints
curl http://localhost:8000/api/todos
curl -X POST http://localhost:8000/api/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "My first todo"}'
```

### OpenAPI Docs

Available at: `http://localhost:8000/docs`

---

## Key Takeaways

1. **Clear PRD = Clean tasks** - Specific requirements lead to focused implementation
2. **Layers enforce order** - Foundation before backend, backend before integration
3. **Parallel execution saves time** - Layer 2 tasks ran in parallel
4. **Retries handle failures** - L1-002 failed once but succeeded on retry
5. **Working code from idea** - Complete API in under 5 minutes

---

## Next Steps

- [Quick Start](../quickstart/README.md) - Try it yourself
- [PRD Reference](../skills/prd.md) - All PRD options
- [Task Format](../skills/breakdown/task-format.md) - Understand task structure
