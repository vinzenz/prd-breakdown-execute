---
layout: default
title: Todo App Example
parent: Examples
nav_order: 1
---

# Todo App Example

A complete walkthrough building a task management API from scratch.

## Overview

We'll build a REST API with:
- User authentication (JWT)
- Project management (CRUD)
- Task management (with assignments)
- Due date tracking

Tech stack: Python (FastAPI + SQLAlchemy)

## Step 1: Create PRD

```bash
claude /prd
```

### Phase 1: Idea

```
What would you like to build?

> A task management API where users can create projects,
  add tasks with due dates and priorities, and assign
  tasks to team members.
```

### Phase 2: Tech Stack

```
Which tech stack?
> 1. Python Backend (FastAPI + SQLAlchemy)
```

### Phase 3: Features

```
List the main features:

> 1. User authentication (signup, login, JWT tokens)
  2. Project management (CRUD, team members)
  3. Task management (CRUD, assignments, priorities)
  4. Due date tracking and notifications
```

### Output Structure

```
docs/prd/task-manager/
├── index.md
└── features/
    ├── authentication.md
    ├── projects.md
    ├── tasks.md
    └── notifications.md
```

## Step 2: Breakdown

```bash
claude /breakdown docs/prd/task-manager/
```

### Analysis Output

```json
// analysis.json
{
  "entities": [
    {"name": "User", "fields": ["id", "email", "password"]},
    {"name": "Project", "fields": ["id", "name", "owner_id"]},
    {"name": "Task", "fields": ["id", "title", "project_id", "assignee_id"]}
  ],
  "relationships": [
    {"from": "Project", "to": "User", "type": "belongs_to"},
    {"from": "Task", "to": "Project", "type": "belongs_to"},
    {"from": "Task", "to": "User", "type": "belongs_to"}
  ]
}
```

### Layer Plan

```json
// layer_plan.json
{
  "layers": {
    "0": {
      "name": "setup",
      "tasks": ["L0-001-project-init"]
    },
    "1": {
      "name": "foundation",
      "tasks": [
        "L1-001-user-model",
        "L1-002-project-model",
        "L1-003-task-model",
        "L1-004-database-config"
      ]
    },
    "2": {
      "name": "backend",
      "tasks": [
        "L2-001-auth-api",
        "L2-002-project-api",
        "L2-003-task-api"
      ]
    },
    "4": {
      "name": "integration",
      "tasks": ["L4-001-e2e-tests"]
    }
  }
}
```

### Generated Tasks

```
tasks/
├── 0-setup/
│   └── L0-001-project-init.xml
├── 1-foundation/
│   ├── L1-001-user-model.xml
│   ├── L1-002-project-model.xml
│   ├── L1-003-task-model.xml
│   └── L1-004-database-config.xml
├── 2-backend/
│   ├── L2-001-auth-api.xml
│   ├── L2-002-project-api.xml
│   └── L2-003-task-api.xml
├── 4-integration/
│   └── L4-001-e2e-tests.xml
└── manifest.json
```

### Example Task: L1-001-user-model.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<task>
  <meta>
    <id>L1-001-user-model</id>
    <name>User Model</name>
    <layer>1</layer>
    <priority>1</priority>
  </meta>

  <context>
    <prd-excerpt>
      Users can sign up with email and password.
      Users have unique emails.
      Users can be project owners or team members.
    </prd-excerpt>
    <tech-stack>
      ORM: SQLAlchemy 2.0 with async
      Database: PostgreSQL
      Migrations: Alembic
    </tech-stack>
  </context>

  <dependencies/>

  <objective>
    Create User SQLAlchemy model with password hashing.
  </objective>

  <requirements>
    <requirement id="R1">User model with id, email, hashed_password, created_at</requirement>
    <requirement id="R2">Email must be unique</requirement>
    <requirement id="R3">Password hashing with bcrypt via passlib</requirement>
    <requirement id="R4">Alembic migration for users table</requirement>
  </requirements>

  <test-requirements>
    <test id="T1">Test User creation with valid data</test>
    <test id="T2">Test password hashing on set</test>
    <test id="T3">Test password verification</test>
    <test id="T4">Test unique email constraint</test>
  </test-requirements>

  <files-to-create>
    <file>src/models/user.py</file>
    <file>tests/models/test_user.py</file>
    <file>alembic/versions/001_create_users.py</file>
  </files-to-create>

  <verification>
    <command>pytest tests/models/test_user.py -v</command>
    <command>alembic upgrade head</command>
    <command>python -m mypy src/models/user.py</command>
  </verification>

  <exports>
    <export name="User">
      from sqlalchemy import Column, String, DateTime
      from sqlalchemy.dialects.postgresql import UUID
      from src.database import Base
      import uuid

      class User(Base):
          __tablename__ = "users"

          id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
          email = Column(String, unique=True, nullable=False, index=True)
          hashed_password = Column(String, nullable=False)
          created_at = Column(DateTime, server_default=func.now())

          def verify_password(self, password: str) -> bool:
              """Verify password against hash."""

          @staticmethod
          def hash_password(password: str) -> str:
              """Hash a password for storage."""
    </export>
  </exports>
</task>
```

## Step 3: Execute

```bash
claude /execute docs/prd/task-manager/tasks/
```

### Execution Progress

```
┌─────────────────────────────────────────────────────────────────┐
│ Executing: task-manager                                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ Layer 0: Setup                                                   │
│   L0-001-project-init     [████████████████████] Complete ✓     │
│                                                                  │
│ Layer 1: Foundation                                              │
│   L1-001-user-model       [████████████████████] Complete ✓     │
│   L1-002-project-model    [████████████████████] Complete ✓     │
│   L1-003-task-model       [████████████████████] Complete ✓     │
│   L1-004-database-config  [████████████████████] Complete ✓     │
│                                                                  │
│ Layer 2: Backend                                                 │
│   L2-001-auth-api         [██████████████░░░░░░] In Progress    │
│   L2-002-project-api      [████████████░░░░░░░░] In Progress    │
│   L2-003-task-api         [████████░░░░░░░░░░░░] In Progress    │
│                                                                  │
│ Layer 4: Integration                                             │
│   L4-001-e2e-tests        [░░░░░░░░░░░░░░░░░░░░] Pending        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Final Project Structure

```
task-manager/
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── project.py
│   │   └── task.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── projects.py
│   │   └── tasks.py
│   └── schemas/
│       ├── __init__.py
│       ├── user.py
│       ├── project.py
│       └── task.py
├── tests/
│   ├── conftest.py
│   ├── models/
│   │   ├── test_user.py
│   │   ├── test_project.py
│   │   └── test_task.py
│   ├── api/
│   │   ├── test_auth.py
│   │   ├── test_projects.py
│   │   └── test_tasks.py
│   └── e2e/
│       └── test_workflows.py
├── alembic/
│   ├── env.py
│   └── versions/
│       ├── 001_create_users.py
│       ├── 002_create_projects.py
│       └── 003_create_tasks.py
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
└── README.md
```

## API Endpoints

The generated API:

| Method | Path | Description |
|--------|------|-------------|
| POST | /auth/signup | Create account |
| POST | /auth/login | Get JWT token |
| GET | /projects | List user's projects |
| POST | /projects | Create project |
| GET | /projects/{id} | Get project |
| PATCH | /projects/{id} | Update project |
| DELETE | /projects/{id} | Delete project |
| GET | /projects/{id}/tasks | List project tasks |
| POST | /projects/{id}/tasks | Create task |
| GET | /tasks/{id} | Get task |
| PATCH | /tasks/{id} | Update task |
| DELETE | /tasks/{id} | Delete task |

## Running the Result

```bash
# Start services
docker-compose up -d

# Run migrations
alembic upgrade head

# Start server
uvicorn src.main:app --reload

# Run tests
pytest -v
```

## Key Takeaways

1. **8 tasks total** - Organized into 4 layers
2. **Parallel execution** - Layer 1 and Layer 2 tasks ran in parallel within their layers
3. **TDD throughout** - Every task wrote tests first
4. **Clean separation** - Models, APIs, and tests in separate modules
5. **Full verification** - pytest + mypy + alembic migrations
