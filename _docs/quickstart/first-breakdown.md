---
layout: default
title: Understanding Breakdown
parent: Quick Start
nav_order: 2
---

# Understanding Breakdown

The `/breakdown` skill transforms your PRD into executable, self-contained tasks organized by dependency layers.

## Running Breakdown

```bash
claude /breakdown docs/prd/task-management/
```

## What Happens

Breakdown runs four sub-skills in sequence:

### 1. Analyze PRD

Reads your PRD and extracts:

- **Features** - What to build
- **Tech stack** - How to build it
- **Entities** - Data models needed
- **Relationships** - How entities connect
- **API endpoints** - Expected interfaces

Output: `analysis.json`

### 2. Plan Layers

Organizes features into dependency layers:

```
Layer 0: Setup
├── Project scaffolding
├── Docker configuration
└── CI/CD setup

Layer 1: Foundation
├── Database models
├── Migrations
└── Configuration

Layer 2: Backend
├── Authentication API
├── Project API
├── Task API
└── Notification service

Layer 3: Frontend (if applicable)
├── Auth components
├── Dashboard
└── Task views

Layer 4: Integration
├── E2E tests
├── WebSocket integration
└── Email notifications
```

Output: `layer_plan.json`

### 3. Generate Tasks

For each layer, creates self-contained XML task files:

```
tasks/
├── 0-setup/
│   └── L0-001-project-init.xml
├── 1-foundation/
│   ├── L1-001-user-model.xml
│   ├── L1-002-project-model.xml
│   └── L1-003-task-model.xml
├── 2-backend/
│   ├── L2-001-auth-api.xml
│   ├── L2-002-project-api.xml
│   └── L2-003-task-api.xml
└── manifest.json
```

### 4. Review Tasks

Each task is validated for:

- **Self-containment** - All needed context present
- **Testability** - TDD requirements defined
- **Dependencies** - Interface contracts complete
- **Scope** - Maximum 3 files per task

Failed tasks are regenerated with feedback.

## Task XML Structure

Each task file contains everything needed for implementation:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<task>
  <meta>
    <id>L2-001-auth-api</id>
    <name>Authentication API Endpoints</name>
    <layer>2</layer>
    <priority>1</priority>
  </meta>

  <context>
    <prd-excerpt>
      User authentication with JWT tokens.
      Endpoints: signup, login, logout, password-reset.
      Rate limiting on login attempts.
    </prd-excerpt>
    <tech-stack>
      Framework: FastAPI
      Auth: python-jose for JWT
      Validation: Pydantic
    </tech-stack>
  </context>

  <dependencies>
    <dependency task="L1-001-user-model">
      <interface-contract>
        class User(Base):
            id: UUID
            email: str
            hashed_password: str
            created_at: datetime
      </interface-contract>
    </dependency>
  </dependencies>

  <objective>
    Implement authentication REST endpoints with JWT.
  </objective>

  <requirements>
    <requirement id="R1">POST /auth/signup - Create new user</requirement>
    <requirement id="R2">POST /auth/login - Return JWT token</requirement>
    <requirement id="R3">POST /auth/logout - Invalidate token</requirement>
    <requirement id="R4">POST /auth/password-reset - Send reset email</requirement>
  </requirements>

  <test-requirements>
    <test id="T1">Test successful signup with valid data</test>
    <test id="T2">Test signup fails with existing email</test>
    <test id="T3">Test login returns valid JWT</test>
    <test id="T4">Test login fails with wrong password</test>
  </test-requirements>

  <files-to-create>
    <file>src/api/auth.py</file>
    <file>tests/api/test_auth.py</file>
  </files-to-create>

  <verification>
    <command>pytest tests/api/test_auth.py -v</command>
    <command>python -m mypy src/api/auth.py</command>
  </verification>

  <exports>
    <export name="auth_router">
      from fastapi import APIRouter
      auth_router = APIRouter(prefix="/auth")
    </export>
  </exports>
</task>
```

## Key Concepts

### Self-Containment

Each task includes:
- **PRD excerpt** - Relevant requirements only
- **Interface contracts** - Exact signatures from dependencies
- **Test requirements** - What must pass
- **Verification commands** - How to check success

The implementing agent needs nothing else.

### Layer Dependencies

Tasks only depend on completed layers:

```
L2-001-auth-api depends on:
  └── L1-001-user-model (must be complete)

L2-001 does NOT depend on:
  └── L2-002-project-api (same layer, parallel)
```

### Interface Contracts

Dependencies provide exact interfaces:

```xml
<dependency task="L1-001-user-model">
  <interface-contract>
    class User(Base):
        id: UUID
        email: str
        hashed_password: str
  </interface-contract>
</dependency>
```

The implementing agent uses these signatures directly.

### Exports

Tasks declare what they provide to downstream tasks:

```xml
<exports>
  <export name="auth_router">
    from fastapi import APIRouter
    auth_router = APIRouter(prefix="/auth")
  </export>
</exports>
```

These become interface contracts for dependent tasks.

## Manifest File

The `manifest.json` tracks all tasks:

```json
{
  "prd": "docs/prd/task-management/index.md",
  "generated_at": "2025-01-15T10:30:00Z",
  "layers": {
    "0": ["L0-001-project-init"],
    "1": ["L1-001-user-model", "L1-002-project-model"],
    "2": ["L2-001-auth-api", "L2-002-project-api"]
  },
  "total_tasks": 12
}
```

## Next Step

Once breakdown completes:

```bash
claude /execute docs/prd/task-management/tasks/
```

See [Running Execute]({{ '/docs/quickstart/first-execute/' | relative_url }}) for the execution phase.
