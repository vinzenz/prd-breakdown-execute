---
layout: default
title: /prd Command
parent: Skills Reference
nav_order: 1
---

# /prd Command

The `/prd` command creates Product Requirements Documents through an interactive 8-phase wizard.

## Usage

```bash
claude /prd
```

## The 8 Phases

### Phase 1: Idea Capture

Describe your project idea:

```
What would you like to build?

> A REST API for managing a book library with user accounts,
  book catalog, borrowing system, and late fee tracking.
```

**Tips:**
- Be descriptive but not exhaustive
- Focus on the core purpose
- Mention key differentiators

### Phase 2: Tech Stack Selection

Choose your preferred technology:

```
Which tech stack would you like to use?

1. Python Backend (FastAPI + SQLAlchemy)
   - Modern async framework
   - Type hints + Pydantic validation
   - pytest for testing

2. Go Backend (Chi + sqlc)
   - High performance
   - Type-safe SQL
   - Built-in testing

3. TanStack (TanStack Start + Drizzle)
   - Full-stack TypeScript
   - Server functions
   - Type-safe ORM

> 1
```

### Phase 3: Feature Definition

List your main features:

```
List the main features of your application:

> 1. User management (registration, login, profiles)
  2. Book catalog (CRUD, search, categories)
  3. Borrowing system (checkout, return, history)
  4. Late fee calculation and tracking
  5. Admin dashboard with reports
```

### Phase 4: Dependencies

Identify external services:

```
What external services does your project need?

> - PostgreSQL database
  - Redis for caching
  - Stripe for fee payments
  - SendGrid for notifications
```

### Phase 5: Options

Configure additional settings:

```
Authentication method:
  [1] JWT tokens
  [2] Session-based
  [3] OAuth 2.0
> 1

API documentation:
  [1] OpenAPI/Swagger
  [2] None
> 1

Containerization:
  [1] Docker + docker-compose
  [2] None
> 1
```

### Phase 6: Review

Summary for confirmation:

```
┌─────────────────────────────────────────────┐
│ PRD Summary: Library Management API          │
├─────────────────────────────────────────────┤
│ Tech Stack: Python (FastAPI + SQLAlchemy)   │
│ Features: 5                                  │
│ Dependencies: 4                              │
│ Auth: JWT                                    │
│ Docs: OpenAPI                                │
│ Container: Docker                            │
├─────────────────────────────────────────────┤
│ Proceed to feature details? [y/n]           │
└─────────────────────────────────────────────┘
```

### Phase 7: Feature Details

For each feature, provide specifics:

```
Feature 1: User Management

User stories:
> - As a user, I can register with email/password
  - As a user, I can log in and receive a token
  - As a user, I can view/update my profile
  - As an admin, I can manage user accounts

Edge cases:
> - Duplicate email registration
  - Password reset flow
  - Account deactivation
```

### Phase 8: Output Generation

The wizard generates:

```
docs/prd/library-management/
├── index.md              # Main PRD document
└── features/
    ├── user-management.md
    ├── book-catalog.md
    ├── borrowing-system.md
    ├── late-fees.md
    └── admin-dashboard.md
```

## Output Format

### Main PRD (index.md)

```markdown
# Library Management API - PRD

## Overview

A REST API for managing library operations including
user accounts, book catalog, and borrowing system.

## Tech Stack

| Component | Technology |
|-----------|------------|
| Framework | FastAPI |
| Database | PostgreSQL |
| ORM | SQLAlchemy |
| Cache | Redis |
| Testing | pytest |

## Features

1. [User Management](features/user-management.md)
2. [Book Catalog](features/book-catalog.md)
3. [Borrowing System](features/borrowing-system.md)
4. [Late Fees](features/late-fees.md)
5. [Admin Dashboard](features/admin-dashboard.md)

## Non-Functional Requirements

- Response time: < 200ms (p95)
- Uptime: 99.9%
- Security: OWASP compliance

## External Dependencies

- PostgreSQL 15+
- Redis 7+
- Stripe API
- SendGrid API
```

### Feature File

```markdown
# User Management

## Description

User registration, authentication, and profile management.

## User Stories

### US-1: Registration
As a new user, I can register with email and password.

**Acceptance Criteria:**
- Email must be unique
- Password must be 8+ characters
- Confirmation email sent

### US-2: Login
As a registered user, I can log in and receive a JWT.

**Acceptance Criteria:**
- Valid credentials return JWT
- Invalid credentials return 401
- Failed attempts logged

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | /users/register | Create account |
| POST | /users/login | Get JWT token |
| GET | /users/me | Get profile |
| PATCH | /users/me | Update profile |

## Data Model

```python
class User:
    id: UUID
    email: str
    hashed_password: str
    full_name: str
    is_active: bool
    is_admin: bool
    created_at: datetime
```

## Edge Cases

- Duplicate email: Return 409 Conflict
- Invalid password format: Return 422
- Inactive account login: Return 403
```

## Next Steps

After creating your PRD:

```bash
claude /breakdown docs/prd/library-management/
```

See [Understanding Breakdown]({{ '/docs/quickstart/first-breakdown/' | relative_url }}).
