# Breaking Down Your First PRD

A detailed walkthrough of the `/breakdown` command.

---

## Starting the Breakdown

```bash
claude /breakdown docs/prd/slack-task-manager/index.md
```

You'll see:
```
Starting PRD breakdown...

Reading PRD: docs/prd/slack-task-manager/index.md
Project: Slack Task Manager
Type: Greenfield

Beginning analysis...
```

---

## Phase 1: Analysis

The `breakdown-analyze-prd` skill extracts everything from your PRD:

```
Analyzing PRD...

Features extracted: 5
  - task-crud (must-have)
  - slack-bot (must-have)
  - user-auth (must-have)
  - team-dashboard (should-have)
  - email-notifications (should-have)

Inferring data models...
  - Task (id, title, description, status, assignee_id, created_at)
  - User (id, slack_id, email, name, team_id)
  - Team (id, name, slack_workspace_id)

Inferring API endpoints...
  - GET/POST /api/tasks
  - GET/PUT/DELETE /api/tasks/{id}
  - POST /api/slack/events
  - GET /api/users/me

Detecting template...
  - Stack: Python + FastAPI
  - Template: webapps/backends/python

Analysis complete. Saved to analysis.json
```

### analysis.json (excerpt)

```json
{
  "prd_slug": "slack-task-manager",
  "project_type": "greenfield",
  "template": "webapps/backends/python",
  "features": [
    {
      "slug": "task-crud",
      "name": "Task CRUD Operations",
      "priority": "must-have",
      "acceptance_criteria": [...]
    }
  ],
  "inferred_models": [
    {
      "name": "Task",
      "fields": [
        { "name": "id", "type": "UUID", "primary_key": true },
        { "name": "title", "type": "str", "max_length": 200 },
        { "name": "description", "type": "str", "nullable": true },
        { "name": "status", "type": "TaskStatus", "enum": true },
        { "name": "assignee_id", "type": "UUID", "foreign_key": "User.id" }
      ]
    }
  ],
  "inferred_endpoints": [
    {
      "method": "POST",
      "path": "/api/tasks",
      "purpose": "Create a new task"
    }
  ]
}
```

---

## Phase 2: Layer Planning

The `breakdown-plan-layers` skill organizes everything into 5 layers:

```
Planning layers...

Layer 0: Setup (4 tasks)
  - Project initialization
  - Database setup
  - Environment configuration
  - Initial migration

Layer 1: Foundation (3 tasks)
  - Task model
  - User model
  - Team model

Layer 2: Backend (6 tasks)
  - Task repository
  - Task service
  - Task API endpoints
  - Slack webhook handler
  - User authentication
  - Team management

Layer 3: Frontend (0 tasks)
  - (API-only project, no frontend)

Layer 4: Integration (2 tasks)
  - Slack bot flow
  - Email notification flow

Total: 15 tasks across 5 layers
Layer plan saved to layer_plan.json
```

### Understanding the Layers

```
Layer 4: Integration
         ▲
         │ Wires Slack bot + email notifications
         │
Layer 3: Frontend (empty for API-only)
         ▲
         │
Layer 2: Backend
         ▲
         │ Uses models from Layer 1
         │
Layer 1: Foundation
         ▲
         │ Uses setup from Layer 0
         │
Layer 0: Setup
         │
         ▼
      Template
```

---

## Phase 3: Task Generation

For each layer, `breakdown-generate-tasks` creates task XML files:

```
Generating tasks for layer 0-setup...
  L0-001-project-init.xml        ✓
  L0-002-database-setup.xml      ✓
  L0-003-env-config.xml          ✓
  L0-004-initial-migration.xml   ✓

Reviewing layer 0-setup... PASSED

Generating tasks for layer 1-foundation...
  L1-001-task-model.xml          ✓
  L1-002-user-model.xml          ✓
  L1-003-team-model.xml          ✓

Reviewing layer 1-foundation... PASSED

...
```

### Example Task XML (L2-003-task-api.xml)

```xml
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
      Each task has title, description, status, and assignee.
    </prd-excerpt>
    <tech-stack>
      Python 3.12, FastAPI 0.109, SQLAlchemy 2.0
    </tech-stack>
  </context>

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

  <objective>
    Create REST API endpoints for task CRUD operations using FastAPI.
  </objective>

  <requirements>
    <requirement id="R1">
      Create file: src/api/tasks.py

      Define APIRouter with prefix "/api/tasks"

      Endpoints:
        - GET / -> list all tasks (TaskListResponse)
        - POST / -> create task (TaskCreateRequest -> TaskResponse)
        - GET /{id} -> get task by id (TaskResponse or 404)
        - PUT /{id} -> update task (TaskUpdateRequest -> TaskResponse)
        - DELETE /{id} -> delete task (204 or 404)
    </requirement>
    <requirement id="R2">
      Create file: src/api/schemas/task.py

      Pydantic models:
        - TaskCreateRequest(title: str, description: str | None)
        - TaskUpdateRequest(title: str | None, status: TaskStatus | None)
        - TaskResponse(id: UUID, title: str, status: TaskStatus, ...)
        - TaskListResponse(tasks: list[TaskResponse])
    </requirement>
  </requirements>

  <test-requirements>
    <test id="T1">
      File: tests/api/test_tasks.py

      Test: test_create_task
      Setup: Mock TaskRepository.create to return Task(id=uuid4(), ...)
      Action: POST /api/tasks with {"title": "Test task"}
      Assert: response.status_code == 201
      Assert: response.json()["title"] == "Test task"
    </test>
    <test id="T2">
      Test: test_get_task_not_found
      Setup: Mock TaskRepository.get_by_id to return None
      Action: GET /api/tasks/{random_uuid}
      Assert: response.status_code == 404
    </test>
  </test-requirements>

  <files-to-create>
    <file>src/api/tasks.py</file>
    <file>src/api/schemas/task.py</file>
  </files-to-create>

  <verification>
    <step>Run: pytest tests/api/test_tasks.py -v</step>
    <step>Verify: All 5 endpoint tests pass</step>
    <step>Run: python -c "from src.api.tasks import router"</step>
    <step>Verify: Router imports without error</step>
  </verification>

  <exports>
    <interface name="TaskRouter" type="fastapi-router">
      from fastapi import APIRouter

      router = APIRouter(prefix="/api/tasks", tags=["tasks"])
    </interface>
  </exports>
</task>
```

---

## Phase 4: Review

The `breakdown-review-tasks` skill validates each task:

```
Reviewing layer 2-backend (batch 1 of 2)...

L2-001: Task Repository
  [✓] Meta section complete
  [✓] Dependencies have type annotations
  [✓] Requirements are specific
  [✓] Test requirements have concrete values
  [✓] Verification steps are runnable
  PASSED

L2-002: Task Service
  [✓] Meta section complete
  [✓] Dependencies have type annotations
  [✓] Requirements are specific
  [✓] Test requirements have concrete values
  [✓] Verification steps are runnable
  PASSED

L2-003: Task API
  [✓] All checks passed
  PASSED

Batch 1: PASSED (3/3)

Reviewing batch 2...
```

### What Review Checks

| Check | Critical? | What It Validates |
|-------|-----------|-------------------|
| Completeness | Yes | No empty sections |
| Self-containment | Yes | All context inline |
| Interface contracts | Yes | Full type annotations |
| Specificity | Yes | No "appropriate", "suitable" |
| Test values | Yes | Concrete, not "some value" |
| Verification | Yes | Runnable commands |
| File scope | Yes | Max 3 files per task |

---

## Final Output

```
Breakdown complete!

Created:
  docs/tasks/slack-task-manager/
  ├── analysis.json
  ├── layer_plan.json
  ├── manifest.json
  ├── 0-setup/
  │   ├── L0-001-project-init.xml
  │   ├── L0-002-database-setup.xml
  │   ├── L0-003-env-config.xml
  │   ├── L0-004-initial-migration.xml
  │   └── .done
  ├── 1-foundation/
  │   ├── L1-001-task-model.xml
  │   ├── L1-002-user-model.xml
  │   ├── L1-003-team-model.xml
  │   └── .done
  ├── 2-backend/
  │   └── ... (6 files)
  ├── 3-frontend/
  │   └── .done (empty layer)
  └── 4-integration/
      └── ... (2 files)

Summary:
  - Total tasks: 15
  - Layers: 5
  - Estimated files: ~35

Next steps:
  claude /execute docs/tasks/slack-task-manager
```

---

## Greenfield vs Brownfield

### Greenfield (New Project)

```bash
claude /breakdown docs/prd/new-app/index.md \
  --output-dir /path/to/new-project
```

- Includes Layer 0 (setup tasks)
- Creates project from template
- Full 5-layer structure

### Brownfield (Existing Project)

```bash
claude /breakdown docs/prd/new-feature/index.md \
  --project-path /path/to/existing-project
```

- Skips Layer 0
- Integrates with existing structure
- Layer 1+ tasks only

---

## Troubleshooting

### Review keeps failing

```
Reviewing layer 2-backend... FAILED

Errors:
  L2-003: Requirement R2 uses placeholder "appropriate validation"

Retrying (attempt 2 of 3)...
```

The generator will retry with specific feedback. If it fails 3 times:
- Check your PRD for vague descriptions
- Add more concrete acceptance criteria
- Specify exact field names and types

### Dependency inference wrong

Edit `analysis.json` manually, then:
```bash
claude /breakdown docs/prd/my-project/index.md --skip-analysis
```

---

## Next Steps

- [Executing your tasks](first-execute.md)
- [Layer Architecture](../skills/breakdown/layers.md)
- [Task Format Reference](../skills/breakdown/task-format.md)
- [Back to Quick Start](README.md)
