# PROJECT.md Format Specification

PROJECT.md provides context about an existing codebase for efficient change requests. It contains human-readable documentation and machine-readable structured data.

## File Location

`{project_root}/PROJECT.md`

## Structure

```markdown
# Project: {name}

## Overview
{Brief description of what this project does}

## Architecture

### Tech Stack
- Backend: {framework} ({language} {version})
- Frontend: {framework} + {bundler}
- Database: {database} with {ORM}
- Auth: {auth mechanism}

### Component Structure
```
{directory tree with annotations}
```

### Key Patterns
- {Pattern 1}: {Description}
- {Pattern 2}: {Description}

## Context Metadata
<!-- Machine-readable section - do not edit manually -->
<project-context version="1.0">
  {structured XML content}
</project-context>
```

## Machine-Readable Section

### Root Element

```xml
<project-context version="1.0">
  <meta>...</meta>
  <features>...</features>
  <api-registry>...</api-registry>
  <schema-registry>...</schema-registry>
</project-context>
```

### Meta Section

```xml
<meta>
  <last-updated>2026-01-12T10:30:00Z</last-updated>
  <last-context-hash>abc123def456</last-context-hash>
  <prd-path>docs/prd/my-project/index.md</prd-path>  <!-- optional -->
</meta>
```

| Field | Required | Description |
|-------|----------|-------------|
| `last-updated` | Yes | ISO 8601 timestamp of last context update |
| `last-context-hash` | Yes | Git commit hash when context was captured |
| `prd-path` | No | Path to PRD if project was created with /prd |

### Features Section

```xml
<features>
  <feature id="auth" status="complete">
    <name>User Authentication</name>
    <files>src/api/auth.py, src/services/auth.py, src/models/user.py</files>
    <crd-ref>docs/crd/auth-enhancement.md</crd-ref>  <!-- optional -->
  </feature>

  <feature id="settings" status="complete">
    <name>User Settings</name>
    <files>src/api/settings.py, src/components/SettingsModal.tsx</files>
  </feature>
</features>
```

| Attribute/Element | Required | Description |
|-------------------|----------|-------------|
| `id` | Yes | Unique slug identifier |
| `status` | Yes | `complete`, `partial`, `planned` |
| `name` | Yes | Human-readable feature name |
| `files` | Yes | Comma-separated list of primary files |
| `crd-ref` | No | Reference to CRD that created/modified this feature |

### API Registry Section

```xml
<api-registry>
  <endpoint method="POST" path="/api/auth/login">
    <request>{ email: string, password: string }</request>
    <response>{ token: string, refresh_token: string, user: User }</response>
    <auth>none</auth>
  </endpoint>

  <endpoint method="GET" path="/api/users/me">
    <request>none</request>
    <response>{ id: string, email: string, settings: object }</response>
    <auth>bearer</auth>
  </endpoint>

  <endpoint method="PUT" path="/api/settings">
    <request>{ settings: object }</request>
    <response>{ success: boolean }</response>
    <auth>bearer</auth>
  </endpoint>
</api-registry>
```

| Attribute/Element | Required | Description |
|-------------------|----------|-------------|
| `method` | Yes | HTTP method (GET, POST, PUT, DELETE, PATCH) |
| `path` | Yes | API path |
| `request` | Yes | Request body shape or "none" |
| `response` | Yes | Response body shape |
| `auth` | No | Authentication requirement (none, bearer, api-key) |

### Schema Registry Section

```xml
<schema-registry>
  <model name="User" table="users">
    <field name="id" type="uuid" primary="true"/>
    <field name="email" type="string" unique="true"/>
    <field name="password_hash" type="string"/>
    <field name="settings" type="jsonb"/>
    <field name="created_at" type="timestamp"/>
  </model>

  <model name="Session" table="sessions">
    <field name="id" type="uuid" primary="true"/>
    <field name="user_id" type="uuid" foreign="users.id"/>
    <field name="token" type="string" unique="true"/>
    <field name="expires_at" type="timestamp"/>
  </model>
</schema-registry>
```

| Attribute | Required | Description |
|-----------|----------|-------------|
| `name` | Yes | Model/class name |
| `table` | Yes | Database table name |
| Field `name` | Yes | Column/field name |
| Field `type` | Yes | Data type (uuid, string, integer, boolean, timestamp, jsonb, etc.) |
| Field `primary` | No | "true" if primary key |
| Field `unique` | No | "true" if unique constraint |
| Field `foreign` | No | Foreign key reference (table.column) |

## Example Complete PROJECT.md

```markdown
# Project: My E-commerce App

## Overview
A full-stack e-commerce application with user authentication, product catalog, and order management.

## Architecture

### Tech Stack
- Backend: FastAPI (Python 3.11)
- Frontend: React + Vite + TypeScript
- Database: PostgreSQL with SQLAlchemy ORM
- Auth: JWT with refresh tokens

### Component Structure
```
src/
├── api/              # FastAPI route handlers
│   ├── auth.py       # Authentication endpoints
│   ├── products.py   # Product CRUD
│   └── orders.py     # Order management
├── models/           # SQLAlchemy models
├── services/         # Business logic layer
├── frontend/         # React application
│   ├── components/   # Reusable components
│   ├── pages/        # Page components
│   └── hooks/        # Custom hooks
└── tests/            # Test suites
```

### Key Patterns
- Repository pattern for data access
- Service layer for business logic
- React Query for server state management
- JWT stored in httpOnly cookies

## Context Metadata
<!-- Machine-readable section - do not edit manually -->
<project-context version="1.0">
  <meta>
    <last-updated>2026-01-12T10:30:00Z</last-updated>
    <last-context-hash>abc123def456789</last-context-hash>
  </meta>

  <features>
    <feature id="auth" status="complete">
      <name>User Authentication</name>
      <files>src/api/auth.py, src/models/user.py, src/frontend/pages/Login.tsx</files>
    </feature>
    <feature id="products" status="complete">
      <name>Product Catalog</name>
      <files>src/api/products.py, src/models/product.py, src/frontend/pages/Products.tsx</files>
    </feature>
  </features>

  <api-registry>
    <endpoint method="POST" path="/api/auth/login">
      <request>{ email: string, password: string }</request>
      <response>{ token: string, user: User }</response>
      <auth>none</auth>
    </endpoint>
    <endpoint method="GET" path="/api/products">
      <request>none</request>
      <response>{ products: Product[], total: number }</response>
      <auth>none</auth>
    </endpoint>
  </api-registry>

  <schema-registry>
    <model name="User" table="users">
      <field name="id" type="uuid" primary="true"/>
      <field name="email" type="string" unique="true"/>
      <field name="password_hash" type="string"/>
    </model>
    <model name="Product" table="products">
      <field name="id" type="uuid" primary="true"/>
      <field name="name" type="string"/>
      <field name="price" type="decimal"/>
      <field name="stock" type="integer"/>
    </model>
  </schema-registry>
</project-context>
```

## Parsing Guidelines

### For Context Check

1. Find `<last-context-hash>` tag
2. Extract hash value
3. Compare with `git rev-parse HEAD`

### For Feature Lookup

1. Parse `<features>` section
2. Build map of `id` → feature object
3. Use for impact analysis matching

### For API Lookup

1. Parse `<api-registry>` section
2. Index by `method + path`
3. Use for endpoint impact detection

### For Schema Lookup

1. Parse `<schema-registry>` section
2. Index by model `name` and `table`
3. Use for migration impact detection

## Version History

| Version | Changes |
|---------|---------|
| 1.0 | Initial format specification |
