---
name: breakdown-plan-layers
description: Plan layer structure and task groupings from PRD analysis. Called by /breakdown skill during Phase 3.
context: fork
allowed-tools: Read Glob
model: claude-haiku-4-5
---

# Layer Planning

You are planning how to organize implementation tasks into layers.

## Input

The calling skill will provide the PRD analysis JSON from the analyze-prd phase.

## Layer Definitions

Organize tasks into layers based on project type:

### Layer 0: Setup (`0-setup`) - Greenfield Only

**Include this layer ONLY if `tech_stack.type === "greenfield"`**

- Copy template to target directory
- Initialize git repository
- Set up environment (venv, dependencies)
- Configure database connection
- Run initial migrations
- Verify setup works

Organize remaining tasks into these 4 layers:

### Layer 1: Foundation (`1-foundation`)
- Database models and migrations
- Core configuration
- Base classes and utilities
- Enum definitions

### Layer 2: Backend (`2-backend`)
- API endpoints
- Service layer / business logic
- Authentication/authorization
- External integrations
- Background tasks

### Layer 3: Frontend (`3-frontend`)
- React components
- State management
- Routing
- Forms and validation
- UI utilities

### Layer 4: Integration (`4-integration`)
- Page-level wiring
- API integration
- E2E flows
- Error handling
- Loading states
- Polish

## Your Task

Create a layer plan that:
1. Assigns each identified component to the appropriate layer
2. Orders tasks within each layer by dependency
3. Ensures no circular dependencies
4. Keeps tasks small (max 3 files each)

## Output Format

Return a JSON object with this structure:

```json
{
  "project_type": "greenfield",
  "layers": [
    {
      "id": "0-setup",
      "name": "Setup",
      "description": "Initialize project from template",
      "skip_if_brownfield": true,
      "tasks": [
        {
          "id": "L0-001",
          "name": "Copy template to target",
          "description": "Copy webapps/backends/python template to target directory",
          "files": [],
          "depends_on": [],
          "provides": ["Project directory with template files"]
        },
        {
          "id": "L0-002",
          "name": "Initialize git and environment",
          "description": "Git init, create venv, install dependencies",
          "files": [],
          "depends_on": ["L0-001"],
          "provides": ["Git repo", "Virtual environment", "Installed dependencies"]
        },
        {
          "id": "L0-003",
          "name": "Configure and migrate database",
          "description": "Set up .env and run initial migrations",
          "files": [".env"],
          "depends_on": ["L0-002"],
          "provides": ["Database connection", "Initial schema"]
        },
        {
          "id": "L0-004",
          "name": "Verify setup",
          "description": "Start dev server and verify template works",
          "files": [],
          "depends_on": ["L0-003"],
          "provides": ["Working development environment"]
        }
      ]
    },
    {
      "id": "1-foundation",
      "name": "Foundation",
      "description": "Database models and core setup",
      "tasks": [
        {
          "id": "L1-001",
          "name": "Create Project model",
          "description": "SQLAlchemy model for PRD projects",
          "files": [
            "app/models/project.py",
            "app/migrations/versions/001_create_projects.py"
          ],
          "depends_on": ["L0-004"],
          "provides": ["Project model", "ProjectStatus enum"]
        },
        {
          "id": "L1-002",
          "name": "Create Conversation model",
          "description": "SQLAlchemy model for chat conversations per persona",
          "files": [
            "app/models/conversation.py",
            "app/migrations/versions/002_create_conversations.py"
          ],
          "depends_on": ["L1-001"],
          "provides": ["Conversation model", "Persona enum"]
        }
      ]
    },
    {
      "id": "2-backend",
      "name": "Backend",
      "description": "API endpoints and business logic",
      "tasks": [
        {
          "id": "L2-001",
          "name": "Project CRUD API",
          "description": "REST endpoints for project management",
          "files": [
            "app/api/projects.py",
            "app/schemas/project.py"
          ],
          "depends_on": ["L1-001"],
          "provides": ["GET/POST/PUT /api/projects endpoints"]
        }
      ]
    },
    {
      "id": "3-frontend",
      "name": "Frontend",
      "description": "React components and state",
      "tasks": [
        {
          "id": "L3-001",
          "name": "Dashboard page",
          "description": "Main dashboard showing project list",
          "files": [
            "src/pages/Dashboard.tsx",
            "src/components/ProjectCard.tsx"
          ],
          "depends_on": [],
          "provides": ["Dashboard component", "ProjectCard component"]
        }
      ]
    },
    {
      "id": "4-integration",
      "name": "Integration",
      "description": "Wiring and polish",
      "tasks": [
        {
          "id": "L4-001",
          "name": "Wire Dashboard to API",
          "description": "Connect Dashboard to project API with loading states",
          "files": [
            "src/pages/Dashboard.tsx",
            "src/hooks/useProjects.ts"
          ],
          "depends_on": ["L2-001", "L3-001"],
          "provides": ["Working dashboard with real data"]
        }
      ]
    }
  ],
  "dependency_graph": {
    "L1-001": [],
    "L1-002": ["L1-001"],
    "L2-001": ["L1-001"],
    "L3-001": [],
    "L4-001": ["L2-001", "L3-001"]
  },
  "summary": {
    "total_tasks": 5,
    "by_layer": {
      "1-foundation": 2,
      "2-backend": 1,
      "3-frontend": 1,
      "4-integration": 1
    }
  }
}
```

## Planning Guidelines

### Task Sizing
- Each task should create/modify max 3 files
- If a feature needs more files, split into multiple tasks
- Keep related files together (model + migration, component + styles)

### Dependency Ordering
- Within Layer 1: Order by model dependencies (base models first)
- Within Layer 2: Order by API dependencies (auth before protected endpoints)
- Within Layer 3: Order by component hierarchy (atoms before molecules)
- Within Layer 4: Order by flow (setup before flows)

### Task Naming Convention
- Use verb + noun: "Create X", "Implement X", "Wire X"
- Be specific: "Create Project model" not "Create models"
- Match PRD terminology

### Splitting Large Features

If a feature maps to many tasks:

Feature: "Tabbed Persona Chat" might become:
- L1-003: Create Message model
- L2-003: Chat message endpoint
- L2-004: SSE streaming endpoint
- L3-004: ChatMessage component
- L3-005: ChatInput component
- L3-006: PersonaTabs component
- L4-002: Wire chat to SSE stream

### Template Awareness

If template exists:
- Don't include tasks for template-provided features
- Reference template's existing models/components
- Start from template's patterns

### Greenfield vs Brownfield

**Greenfield** (`tech_stack.type === "greenfield"`):
- Include Layer 0 with setup tasks
- Layer 1 tasks depend on L0-004 (verify setup)
- Template path determines which template to copy

**Brownfield** (`tech_stack.type === "brownfield"`):
- Skip Layer 0 entirely (set `skip_if_brownfield: true` layer excluded)
- Layer 1 tasks have no Layer 0 dependencies
- Use `tech_stack.project_path` as base for all file paths
- Read existing project structure to inform task context

## Do NOT

- Create tasks that are too large (>3 files)
- Create vague tasks ("Set up backend")
- Skip layers (every project needs all 4 layers)
- Create circular dependencies

Return the complete layer plan JSON.
