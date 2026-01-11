# Layer Definitions

Tasks are organized into layers based on architectural dependencies. Greenfield projects include Layer 0 for setup; brownfield projects skip directly to Layer 1.

## Layer 0: Setup (Greenfield Only)

**Identifier:** `0-setup`

**Purpose:** Initialize project from template and prepare development environment.

**Contents:**
- Create project directory at target location
- Copy template files from `webapps/backends/{template}`
- Initialize git repository with first commit
- Set up virtual environment / install dependencies
- Configure environment variables
- Run initial database migrations
- Verify template works (make dev starts successfully)

**Dependencies:** None (first layer, greenfield only)

**Typical Tasks:**
- L0-001: Copy template to target directory
- L0-002: Initialize git repository
- L0-003: Create virtual environment and install dependencies
- L0-004: Configure environment and run migrations
- L0-005: Verify setup works

**Exit Criteria:**
- Project directory exists with all template files
- Git repo initialized with "Initial commit from template"
- Dependencies installed (venv active, packages installed)
- `.env` configured with database connection
- `make dev` or equivalent starts without errors
- Database migrations complete
- Can access app at http://localhost:3000 (or equivalent)

**Template-Specific Notes:**

| Template | Setup Command | Dev Command | Default Port |
|----------|--------------|-------------|--------------|
| `python` | `make setup` | `make dev` | 8000 (API), 3000 (UI) |
| `go` | `make setup` | `make dev` | 8080 (API), 3000 (UI) |
| `tanstack` | `npm install` | `npm run dev` | 3000 |

---

## Layer 1: Foundation

**Identifier:** `1-foundation`

**Purpose:** Establish the data layer and core infrastructure.

**Contents:**
- Database models (SQLAlchemy, Prisma, etc.)
- Database migrations
- Core configuration (environment, settings)
- Base classes and utilities
- Enum definitions

**Dependencies:** Template only (no other layers)

**Typical Tasks:**
- Create User model
- Create Project model
- Set up database migrations
- Configure environment variables
- Create base repository class

**Exit Criteria:**
- All models defined
- Migrations run successfully
- `alembic upgrade head` (or equivalent) succeeds
- Model tests pass

## Layer 2: Backend Logic

**Identifier:** `2-backend`

**Purpose:** Implement business logic, APIs, and services.

**Contents:**
- API endpoints (REST, GraphQL)
- Service layer (business logic)
- Authentication/authorization
- External integrations (LLM, email, etc.)
- Background tasks

**Dependencies:** Layer 1 (Foundation)

**Typical Tasks:**
- Create CRUD endpoints for Project
- Implement chat endpoint with SSE streaming
- Set up JWT authentication
- Create LLM service wrapper
- Implement file upload handling

**Exit Criteria:**
- All endpoints implemented
- API tests pass
- Services have unit tests
- Integration with external services works

## Layer 3: Frontend Components

**Identifier:** `3-frontend`

**Purpose:** Build the user interface components.

**Contents:**
- React/Vue/Svelte components
- State management (Zustand, Redux, etc.)
- Routing configuration
- Forms and validation
- UI utilities and hooks

**Dependencies:** Layer 2 contracts (API shapes, not implementation)

**Typical Tasks:**
- Create Dashboard component
- Build Chat interface
- Implement persona tab navigation
- Create form components with validation
- Set up TanStack Query hooks

**Exit Criteria:**
- All components render correctly
- Component tests pass
- State management works
- Forms validate correctly

## Layer 4: Integration

**Identifier:** `4-integration`

**Purpose:** Wire everything together and polish.

**Contents:**
- Page-level components
- API integration wiring
- End-to-end flows
- Error boundaries
- Loading states
- Mobile responsiveness

**Dependencies:** Layers 1, 2, 3

**Typical Tasks:**
- Wire chat component to API
- Implement error handling
- Add loading spinners
- Test E2E flows
- Polish mobile layout

**Exit Criteria:**
- Full flows work end-to-end
- E2E tests pass
- Error states handled gracefully
- Responsive on mobile

## Layer Dependency Rules

### Greenfield Projects (includes Layer 0)
```
4-integration
     │
     ├── 3-frontend
     │        │
     │        └── 2-backend (contracts only)
     │                 │
     │                 └── 1-foundation
     │                          │
     │                          └── 0-setup
     │
     ├── 2-backend
     │        │
     │        └── 1-foundation
     │                 │
     │                 └── 0-setup
     │
     ├── 1-foundation
     │        │
     │        └── 0-setup
     │
     └── 0-setup (template + environment)
```

### Brownfield Projects (skips Layer 0)
```
4-integration
     │
     ├── 3-frontend
     │        └── 2-backend (contracts only)
     │                 └── 1-foundation
     │
     ├── 2-backend
     │        └── 1-foundation
     │
     └── 1-foundation (existing project)
```

**Key Rules:**
1. Each layer can only depend on lower-numbered layers
2. Layer 3 depends on Layer 2 *contracts* (API shapes), not implementation
3. No circular dependencies allowed
4. Layer must be complete before dependent layer starts
5. **Greenfield**: Layer 0 must complete before any other layer
6. **Brownfield**: Skip Layer 0, start from Layer 1 with existing project

## Task Assignment Guidelines

When assigning tasks to layers, consider:

1. **Data first**: If it involves database schema, it's Layer 1
2. **API second**: If it involves HTTP endpoints, it's Layer 2
3. **UI third**: If it involves React components, it's Layer 3
4. **Wiring last**: If it connects UI to API, it's Layer 4

**Edge Cases:**
- Shared types/interfaces: Layer 1 (foundation)
- API client generation: Layer 3 (frontend needs it)
- Form validation rules: Layer 3 (UI concern)
- API validation rules: Layer 2 (backend concern)
- E2E tests: Layer 4 (integration)
- Unit tests: Same layer as the code being tested

## Template Handling

**If template exists:**
- Layer 1 assumes template's base models exist
- Reference template's patterns in task context
- Don't recreate template boilerplate

**If no template:**
- Layer 1 includes project setup tasks
- First task creates project structure
- Include dependency installation commands
