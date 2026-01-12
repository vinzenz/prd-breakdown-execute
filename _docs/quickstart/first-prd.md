---
layout: default
title: Creating Your First PRD
parent: Quick Start
nav_order: 1
---

# Creating Your First PRD

The `/prd` command guides you through an interactive 8-phase wizard to create a comprehensive Product Requirements Document.

## Starting the Wizard

```bash
claude /prd
```

## Phase 1: Idea Capture

The wizard asks about your project:

```
What would you like to build?

> A task management API with user authentication,
  project organization, and real-time notifications
```

Be descriptive. The more detail you provide, the better the generated tasks.

## Phase 2: Tech Stack Selection

Choose your preferred technology:

```
Which tech stack would you like to use?

1. Python Backend (FastAPI + SQLAlchemy)
2. Go Backend (Chi + sqlc)
3. TanStack (TanStack Start + Drizzle)

> 1
```

The stack determines:
- Project structure templates
- Testing frameworks
- Database patterns
- API conventions

## Phase 3: Feature Definition

Define your core features:

```
List the main features of your application:

> 1. User authentication (signup, login, logout, password reset)
  2. Project management (CRUD, team members, settings)
  3. Task management (CRUD, assignments, due dates, priorities)
  4. Real-time notifications (WebSocket, email digest)
  5. Dashboard with analytics
```

Each feature becomes a detailed specification.

## Phase 4: Dependencies

Identify external services:

```
What external services or dependencies does your project need?

> - PostgreSQL database
  - Redis for caching and sessions
  - SendGrid for email
  - AWS S3 for file storage
```

## Phase 5: Options

Configure preferences:

```
Additional options:

Authentication method: [JWT / Session / OAuth]
> JWT

API documentation: [OpenAPI / None]
> OpenAPI

Containerization: [Docker / None]
> Docker
```

## Phase 6: Review

The wizard summarizes your inputs:

```
┌─────────────────────────────────────────────────┐
│ PRD Summary                                      │
├─────────────────────────────────────────────────┤
│ Project: Task Management API                     │
│ Stack: Python (FastAPI + SQLAlchemy)            │
│ Features: 5                                      │
│ Dependencies: PostgreSQL, Redis, SendGrid, S3   │
├─────────────────────────────────────────────────┤
│ Ready to proceed? [y/n]                          │
└─────────────────────────────────────────────────┘
```

## Phase 7: Feature Details

For each feature, provide detailed specifications:

```
Feature: User Authentication

Describe the user stories:

> - As a user, I can sign up with email and password
  - As a user, I can log in and receive a JWT token
  - As a user, I can request a password reset email
  - As a user, I can log out and invalidate my token

What edge cases should be handled?

> - Email already registered
  - Invalid password format
  - Expired reset tokens
  - Rate limiting on login attempts
```

## Phase 8: Output

The wizard generates your PRD:

```
docs/prd/task-management/
├── index.md              # Main PRD document
└── features/
    ├── authentication.md
    ├── projects.md
    ├── tasks.md
    ├── notifications.md
    └── dashboard.md
```

## PRD Structure

The generated `index.md` includes:

```markdown
# Task Management API - PRD

## Overview
[Project description and goals]

## Tech Stack
- Framework: FastAPI
- Database: PostgreSQL + SQLAlchemy
- Cache: Redis
- Testing: pytest

## Features
1. [User Authentication](features/authentication.md)
2. [Project Management](features/projects.md)
...

## Non-Functional Requirements
- API response time < 200ms
- 99.9% uptime target
- JWT token expiry: 24 hours

## Dependencies
- PostgreSQL 15+
- Redis 7+
- SendGrid API
- AWS S3
```

## Tips for Better PRDs

1. **Be specific** - "Users can filter tasks by status" is better than "filtering"
2. **Include edge cases** - What happens when things go wrong?
3. **Define boundaries** - What's NOT in scope?
4. **Think about data** - What entities exist? What are their relationships?
5. **Consider auth early** - Who can do what?

## Next Step

Once your PRD is complete:

```bash
claude /breakdown docs/prd/task-management/
```

See [Understanding Breakdown]({{ '/docs/quickstart/first-breakdown/' | relative_url }}) for the next phase.
