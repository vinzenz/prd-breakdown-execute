# Layer 2: Backend - Implementation Tasks

## Overview

Generated 14 complete implementation task files for backend layer of Voice PRD Generator application.

**Technology Stack:**
- Python 3.11+
- FastAPI 0.100+
- SQLAlchemy 2.x (async)
- PostgreSQL 15
- Anthropic SDK (Z.ai endpoint)
- sse-starlette (streaming)
- Pillow (image generation)

**Template Base:** webapps/backends/python

## Task Files Generated

### Data Access Layer (Repositories)
- **L2-001-project-repository.xml** - CRUD operations for Project model
- **L2-002-conversation-message-repositories.xml** - Query layers for Conversation and Message models

### Business Logic Layer (Services)
- **L2-003-project-service.xml** - Project management with slug generation and name validation
- **L2-004-llm-service.xml** - Anthropic SDK integration with streaming support
- **L2-005-persona-workflow-service.xml** - Workflow tracking and question flow management
- **L2-006-prd-generation-service.xml** - PRD document assembly from conversation history
- **L2-007-wireframe-conversion-service.xml** - ASCII to PNG image conversion with Pillow

### API Endpoints Layer
- **L2-008-projects-api.xml** - REST CRUD endpoints for projects
- **L2-009-conversations-api.xml** - REST endpoints for conversation management
- **L2-010-chat-sse-streaming.xml** - SSE streaming chat with tool execution
- **L2-011-prd-retrieval-endpoint.xml** - PRD viewer and export endpoints
- **L2-012-whats-next-endpoint.xml** - What's Next panel management endpoints
- **L2-013-wireframe-endpoint.xml** - Wireframe rendering endpoint
- **L2-014-conversation-clear-endpoint.xml** - Conversation reset functionality

## Task Characteristics

Each task file includes:
- Complete specifications (no placeholders)
- Explicit type annotations and imports
- Concrete test cases with specific values
- Runnable verification commands
- Interface contracts for dependencies
- Export definitions for downstream tasks

## Dependency Chain

```
L2-001 (ProjectRepository)
  ↓
L2-003 (ProjectService)
  ↓
L2-008 (Projects API)

L2-002 (Conversation/Message Repos)
  ↓
L2-009 (Conversations API)
  ↓
L2-014 (Clear Conversation)

L2-004 (LLM Service)
  ↓
L2-010 (Chat SSE Streaming)

L2-006 (PRD Service)
  ↓
L2-011 (PRD Retrieval)

L2-007 (Wireframe Service)
  ↓
L2-013 (Wireframe Endpoint)

L2-005 (Workflow Service)
  → (Used by chat streaming for workflow tracking)

L2-012 (What's Next)
  → (Standalone - stores in project.prd_content)
```

## Implementation Order

Recommended implementation sequence:

1. **Repositories First** (L2-001, L2-002)
   - Foundation for data access
   - Can be tested independently

2. **Core Services** (L2-003, L2-004, L2-006, L2-007)
   - Business logic layer
   - Depends on repositories
   - Independent of each other

3. **Supporting Services** (L2-005)
   - Workflow tracking
   - Optional for MVP

4. **Basic APIs** (L2-008, L2-009)
   - CRUD operations
   - Foundation for frontend

5. **Advanced APIs** (L2-010, L2-011, L2-012, L2-013)
   - Chat streaming with tool execution
   - PRD viewer
   - What's Next panel
   - Wireframe rendering

6. **Optional Features** (L2-014)
   - Conversation clear/reset
   - Could-have priority

## Key Features

### LLM Integration
- Anthropic SDK with Z.ai endpoint
- Streaming responses via SSE
- Tool execution (set_project_name, update_prd, render_wireframe, etc.)
- Per-persona system prompts
- Project context injection

### Data Storage
- PostgreSQL with async SQLAlchemy
- JSONB for flexible storage (What's Next items, PRD content)
- Separate conversation contexts per persona
- Message history for LLM context

### Tool Execution
- set_project_name: Updates project name
- update_prd: Incremental PRD updates
- render_wireframe: ASCII to PNG conversion
- update_whats_next: Task tracking
- ask_user_question: Structured UI questions

### API Patterns
- RESTful design
- JWT authentication (from template)
- Pydantic request/response validation
- FastAPI dependency injection
- Async/await throughout

## File Count

- Total XML task files: 14
- Total estimated implementation files: ~35-40
  - Repositories: 3 files
  - Services: 7 files
  - API routers: 7 files
  - Test files: ~20 files
  - Schema updates: 1 file

## Testing Strategy

Each task includes:
- Unit tests for services/repositories
- Integration tests for API endpoints
- Mock tests for LLM integration
- Fixture cleanup for generated files

Total estimated test cases: ~120-140 tests

## Verification Commands

All tasks include runnable verification:
```bash
cd backend && pytest tests/repositories/test_*.py -v
cd backend && pytest tests/services/test_*.py -v
cd backend && pytest tests/api/test_*.py -v
```

Manual API testing with curl examples provided.

## Next Steps

After implementing Layer 2:
1. Run all backend tests
2. Start FastAPI server
3. Verify API endpoints with Swagger UI (http://localhost:8080/docs)
4. Test SSE streaming with curl -N
5. Proceed to Layer 3 (Frontend) implementation

## Notes

- Authentication uses template's JWT system
- Database migrations assumed from Layer 1
- Static file serving required for wireframes
- Environment variables needed for Anthropic API
- All paths are absolute from project root
