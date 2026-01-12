---
layout: default
title: Self-Contained Tasks
parent: Concepts
nav_order: 1
---

# Self-Contained Tasks

Self-contained tasks are the fundamental unit of work in PRD Breakdown Execute. Each task includes everything needed for an AI agent to implement it without external context.

## Why Self-Containment Matters

Traditional AI coding approaches suffer from:

| Problem | Consequence |
|---------|-------------|
| Context accumulation | Model forgets early instructions |
| External dependencies | Agent can't find needed information |
| Implicit knowledge | Assumptions lead to inconsistencies |
| Unbounded scope | Tasks never complete |

Self-contained tasks solve these by including all context explicitly.

## Task XML Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<task>
  <!-- Identity and metadata -->
  <meta>
    <id>L2-001-auth-api</id>
    <name>Authentication API Endpoints</name>
    <layer>2</layer>
    <priority>1</priority>
  </meta>

  <!-- Relevant PRD context only -->
  <context>
    <prd-excerpt>
      User authentication with JWT tokens.
      Signup, login, logout, password reset.
      Rate limiting: 5 failed logins per 15 min.
    </prd-excerpt>
    <tech-stack>
      Framework: FastAPI 0.100+
      Auth: python-jose for JWT
      Hashing: passlib with bcrypt
    </tech-stack>
  </context>

  <!-- Exact signatures from dependencies -->
  <dependencies>
    <dependency task="L1-001-user-model">
      <interface-contract>
        from sqlalchemy.orm import DeclarativeBase
        from sqlalchemy import Column, String
        from uuid import UUID

        class User(Base):
            __tablename__ = "users"
            id: UUID
            email: str
            hashed_password: str
            is_active: bool = True
      </interface-contract>
    </dependency>
  </dependencies>

  <!-- Clear goal statement -->
  <objective>
    Implement authentication REST endpoints returning JWT tokens.
    Include rate limiting on login attempts.
  </objective>

  <!-- Specific requirements -->
  <requirements>
    <requirement id="R1">
      POST /auth/signup - Create user, return 201 + user data
    </requirement>
    <requirement id="R2">
      POST /auth/login - Validate credentials, return JWT
    </requirement>
    <requirement id="R3">
      POST /auth/logout - Invalidate token (blacklist)
    </requirement>
    <requirement id="R4">
      Rate limit: 5 failed logins per IP per 15 minutes
    </requirement>
  </requirements>

  <!-- TDD requirements -->
  <test-requirements>
    <test id="T1">test_signup_success - Valid data returns 201</test>
    <test id="T2">test_signup_duplicate - Existing email returns 409</test>
    <test id="T3">test_login_success - Valid creds return JWT</test>
    <test id="T4">test_login_invalid - Wrong password returns 401</test>
    <test id="T5">test_rate_limit - 6th attempt returns 429</test>
  </test-requirements>

  <!-- Bounded scope -->
  <files-to-create>
    <file>src/api/auth.py</file>
    <file>tests/api/test_auth.py</file>
  </files-to-create>

  <!-- Verification commands -->
  <verification>
    <command>pytest tests/api/test_auth.py -v</command>
    <command>python -m mypy src/api/auth.py --strict</command>
  </verification>

  <!-- What this provides to downstream tasks -->
  <exports>
    <export name="auth_router">
      from fastapi import APIRouter
      auth_router = APIRouter(prefix="/auth", tags=["auth"])
    </export>
    <export name="get_current_user">
      async def get_current_user(token: str) -> User:
          """Dependency for protected routes."""
    </export>
  </exports>
</task>
```

## Key Principles

### 1. PRD Excerpt, Not Full PRD

Tasks include only relevant PRD sections:

```xml
<!-- Good: Focused excerpt -->
<prd-excerpt>
  User authentication with JWT tokens.
  Rate limiting: 5 failed logins per 15 min.
</prd-excerpt>

<!-- Bad: Full PRD copy -->
<prd-excerpt>
  [500 lines of unrelated features...]
</prd-excerpt>
```

The agent doesn't need to know about the payment system when implementing auth.

### 2. Interface Contracts, Not References

Tasks include exact signatures, not file paths:

```xml
<!-- Good: Exact signature -->
<interface-contract>
  class User(Base):
      id: UUID
      email: str
</interface-contract>

<!-- Bad: Just a reference -->
<interface-contract>
  See src/models/user.py for User class
</interface-contract>
```

The agent can use the signature directly without reading other files.

### 3. Bounded File Scope

Maximum 3 files per task:

```xml
<!-- Good: Focused scope -->
<files-to-create>
  <file>src/api/auth.py</file>
  <file>tests/api/test_auth.py</file>
</files-to-create>

<!-- Bad: Too many files -->
<files-to-create>
  <file>src/api/auth.py</file>
  <file>src/api/users.py</file>
  <file>src/services/auth_service.py</file>
  <file>src/utils/jwt.py</file>
  <file>tests/api/test_auth.py</file>
</files-to-create>
```

Large scopes should be split into multiple tasks.

### 4. Runnable Verification

Verification commands must be specific and runnable:

```xml
<!-- Good: Specific, runnable -->
<verification>
  <command>pytest tests/api/test_auth.py -v</command>
  <command>python -m mypy src/api/auth.py --strict</command>
</verification>

<!-- Bad: Vague or meta -->
<verification>
  <command>make sure it works</command>
  <command>tests should pass</command>
</verification>
```

### 5. Explicit Exports

Tasks declare what they provide:

```xml
<exports>
  <export name="auth_router">
    from fastapi import APIRouter
    auth_router = APIRouter(prefix="/auth")
  </export>
</exports>
```

These become interface contracts for dependent tasks.

## Benefits

| Benefit | How Self-Containment Achieves It |
|---------|----------------------------------|
| Parallel execution | No shared state to coordinate |
| Fresh context | Agent starts clean |
| Reproducible | Same input → same output |
| Verifiable | Clear success criteria |
| Debuggable | Failed task has all context |

## Anti-Patterns

### Implicit Dependencies

```xml
<!-- Bad: Assumes agent knows about config -->
<requirements>
  <requirement>Use the database config from the config module</requirement>
</requirements>

<!-- Good: Explicit -->
<dependencies>
  <dependency task="L1-002-config">
    <interface-contract>
      DATABASE_URL = os.environ["DATABASE_URL"]
    </interface-contract>
  </dependency>
</dependencies>
```

### External References

```xml
<!-- Bad: Points elsewhere -->
<context>
  Refer to docs/architecture.md for patterns
</context>

<!-- Good: Includes relevant pattern -->
<context>
  <tech-stack>
    Pattern: Repository pattern for data access
    - Services call repositories
    - Repositories handle ORM operations
  </tech-stack>
</context>
```

### Unbounded Scope

```xml
<!-- Bad: Open-ended -->
<objective>
  Implement the user management system
</objective>

<!-- Good: Specific -->
<objective>
  Implement user CRUD endpoints: create, read, update, delete.
  Admin-only access for create/delete.
</objective>
```
