# The 5-Layer Architecture

How breakdown organizes tasks into architectural layers.

---

## Overview

Tasks are organized into 5 layers, each with a specific purpose and dependency rules:

```
Layer 4: Integration     ◄── Wires everything together
         ▲
         │
Layer 3: Frontend        ◄── UI components and state
         ▲
         │ depends on contracts (not implementation)
         │
Layer 2: Backend         ◄── APIs, services, business logic
         ▲
         │
Layer 1: Foundation      ◄── Models, migrations, config
         ▲
         │
Layer 0: Setup           ◄── Project initialization
         ▲                   (greenfield only)
         │
      Template
```

---

## Layer 0: Setup

**Purpose:** Initialize a new project from template.

**When used:** Greenfield projects only.

**Contains:**
- Copy template files
- Initialize git repository
- Set up environment configuration
- Configure database connection
- Run initial migrations
- Verify setup works

**Dependencies:** None (first layer)

**Example tasks:**
```
L0-001: Project initialization
L0-002: Environment configuration
L0-003: Database setup
L0-004: Initial migration
```

**Skipped when:** Brownfield projects (existing codebase)

---

## Layer 1: Foundation

**Purpose:** Build the data layer and core infrastructure.

**Contains:**
- Database models
- Schema migrations
- Configuration classes
- Base classes and utilities
- Enums and constants
- Core types

**Dependencies:** Layer 0 (or existing project)

**Characteristics:**
- No business logic
- No API endpoints
- Pure data definitions
- Foundational code

**Example tasks:**
```
L1-001: User model and enum types
L1-002: Project model
L1-003: Task model with status enum
L1-004: Database migrations
```

**Exports:** Model interfaces for Layer 2

---

## Layer 2: Backend

**Purpose:** Implement business logic and API layer.

**Contains:**
- REST/GraphQL endpoints
- Service layer
- Repository layer
- Authentication logic
- External integrations
- Background tasks
- Webhooks

**Dependencies:** Layer 1 models

**Characteristics:**
- All business logic
- API contracts defined here
- External service integration
- No UI code

**Example tasks:**
```
L2-001: Task repository
L2-002: Task service
L2-003: Task API endpoints
L2-004: Slack webhook handler
L2-005: User authentication
```

**Exports:** API contracts (schemas, router interfaces) for Layer 3

---

## Layer 3: Frontend

**Purpose:** Build UI components and state management.

**Contains:**
- React/Vue/Svelte components
- State management (Redux, Zustand, etc.)
- Routing configuration
- Form handling
- Custom hooks
- UI utilities

**Dependencies:** Layer 2 contracts (not implementation)

**Key principle:** Frontend depends on API response shapes, not on how the backend implements them.

```
Layer 3 sees:                    Layer 3 does NOT see:
─────────────────────           ─────────────────────────
TaskResponse {                   class TaskRepository:
  id: string                       def get_by_id(...):
  title: string                      session.query(...)
  status: "pending"...               ...
}
```

**Example tasks:**
```
L3-001: Task list component
L3-002: Task form component
L3-003: Dashboard page
L3-004: State management setup
```

**Note:** API-only projects may have empty Layer 3.

---

## Layer 4: Integration

**Purpose:** Wire everything together and polish.

**Contains:**
- Page-level wiring
- End-to-end flows
- Error handling integration
- Loading states
- Mobile responsiveness
- Final integrations
- E2E test setup

**Dependencies:** All previous layers

**Characteristics:**
- Connects Layer 2 and Layer 3
- No new core functionality
- Polish and refinement
- Integration testing

**Example tasks:**
```
L4-001: Wire dashboard to API
L4-002: Slack notification flow
L4-003: Error boundary setup
L4-004: E2E test suite
```

---

## Dependency Rules

### Between Layers

```
Layer N can depend on Layer N-1.
Layer N CANNOT depend on Layer N+1.
Layer N can depend on ANY layer < N.
```

### Within Layers

Tasks within the same layer can have dependencies:

```json
{
  "L2-003": ["L2-001", "L2-002"],
  "L2-004": ["L2-001"]
}
```

This means:
- L2-003 depends on L2-001 and L2-002
- L2-004 depends on L2-001
- L2-001, L2-002 can run in parallel

### Dependency Graph Visualization

```
Layer 2:

L2-001 (repository) ────┐
                        ├───► L2-003 (api)
L2-002 (service) ───────┘

L2-004 (slack) ─────────────► L2-006 (integration)

L2-005 (auth) ─────────────► (layer 4)
```

---

## Interface Contracts

Layers communicate through interface contracts, not implementation:

### Layer 1 exports to Layer 2:

```python
# Interface contract (what Layer 2 sees)
from sqlalchemy.orm import Mapped
from uuid import UUID

class Task(Base):
    __tablename__ = "tasks"
    id: Mapped[UUID]
    title: Mapped[str]
    status: Mapped[TaskStatus]
```

### Layer 2 exports to Layer 3:

```python
# Interface contract (what Layer 3 sees)
from pydantic import BaseModel

class TaskResponse(BaseModel):
    id: str
    title: str
    status: str
```

This decoupling means:
- Layer 2 can change implementation without affecting Layer 3
- Layer 3 tests can mock Layer 2 responses
- Parallel development possible with agreed contracts

---

## Greenfield vs Brownfield

### Greenfield Project

```
Layers: [L0, L1, L2, L3, L4]

L0 creates the project from template
L1+ builds on top of template structure
```

### Brownfield Project

```
Layers: [L1, L2, L3, L4]

L0 is skipped
L1 integrates with existing models/structure
L2+ builds new features on existing foundation
```

---

## Layer Planning

The `breakdown-plan-layers` skill:

1. **Analyzes** inferred models, APIs, components
2. **Assigns** each to appropriate layer
3. **Determines** dependencies
4. **Orders** tasks within layers
5. **Splits** large features across tasks (max 3 files each)

### layer_plan.json Structure

```json
{
  "project_type": "greenfield",
  "layers": {
    "0-setup": {
      "tasks": ["L0-001", "L0-002", "L0-003"],
      "dependencies": {}
    },
    "1-foundation": {
      "tasks": ["L1-001", "L1-002", "L1-003"],
      "dependencies": {
        "L1-003": ["L1-001", "L1-002"]
      }
    },
    "2-backend": {
      "tasks": ["L2-001", "L2-002", "L2-003"],
      "dependencies": {
        "L2-003": ["L2-001", "L2-002"]
      }
    }
  }
}
```

---

## Best Practices

### Keep Layers Focused

- Layer 1: Only data definitions
- Layer 2: Only backend logic
- Layer 3: Only frontend code
- Layer 4: Only integration

### Design for Parallelism

- Minimize intra-layer dependencies
- Independent tasks can run in parallel
- Plan interface contracts early

### Use Interface Contracts

- Define clear boundaries between layers
- Export type signatures, not implementations
- Enable independent testing

---

## Next Steps

- [Task Format Specification](task-format.md)
- [Execute Skill Reference](../execute/README.md)
- [Back to Breakdown Overview](README.md)
