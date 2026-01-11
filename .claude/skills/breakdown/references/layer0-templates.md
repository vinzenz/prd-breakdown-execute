# Layer 0 Task Templates

Reference templates for Layer 0 (setup) tasks. These templates are for greenfield projects only.

## L0-001: Copy Template

```xml
<task>
  <meta>
    <id>L0-001</id>
    <name>Copy Template to Target Directory</name>
    <layer>0-setup</layer>
    <priority>1</priority>
    <estimated-files>0</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
    Project type: greenfield
    Template: {TEMPLATE_PATH}
    Target: {TARGET_DIR}
    </prd-excerpt>

    <tech-stack>{TECH_STACK_SUMMARY}</tech-stack>
    <template-source>{TEMPLATE_PATH}</template-source>
    <target-directory>{TARGET_DIR}</target-directory>
  </context>

  <dependencies>
    <interface name="template" type="filesystem">
    Template exists at: {TEMPLATE_PATH}
    Contains: Makefile, app/, frontend/, docker-compose.yml
    </interface>
  </dependencies>

  <objective>
  Copy the template files to the target project directory,
  creating the foundation for the new project.
  </objective>

  <requirements>
    <requirement id="1">
    Create target directory: mkdir -p {TARGET_DIR}
    </requirement>
    <requirement id="2">
    Copy template contents: cp -r {TEMPLATE_PATH}/* {TARGET_DIR}/
    </requirement>
    <requirement id="3">
    Copy hidden files: cp -r {TEMPLATE_PATH}/.[!.]* {TARGET_DIR}/ (if any)
    </requirement>
    <requirement id="4">
    Remove any existing .git: rm -rf {TARGET_DIR}/.git
    </requirement>
  </requirements>

  <test-requirements>
    <test id="1">
    Verify target exists: test -d {TARGET_DIR}
    </test>
    <test id="2">
    Verify Makefile: test -f {TARGET_DIR}/Makefile
    </test>
  </test-requirements>

  <files-to-create></files-to-create>

  <verification>
    <step>Run: ls -la {TARGET_DIR} - shows template files</step>
    <step>Run: test -f {TARGET_DIR}/Makefile && echo "OK"</step>
  </verification>

  <exports>
    <interface name="project_directory" type="path">
    {TARGET_DIR}
    Contains: Template files ready for initialization
    </interface>
  </exports>
</task>
```

## L0-002: Initialize Git Repository

```xml
<task>
  <meta>
    <id>L0-002</id>
    <name>Initialize Git Repository</name>
    <layer>0-setup</layer>
    <priority>2</priority>
    <estimated-files>0</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
    Project: {PROJECT_NAME}
    Location: {TARGET_DIR}
    </prd-excerpt>

    <project-directory>{TARGET_DIR}</project-directory>
  </context>

  <dependencies>
    <interface name="project_directory" type="path">
    {TARGET_DIR} - contains template files from L0-001
    </interface>
  </dependencies>

  <objective>
  Initialize a git repository with the template files as the initial commit.
  </objective>

  <requirements>
    <requirement id="1">
    Navigate to project: cd {TARGET_DIR}
    </requirement>
    <requirement id="2">
    Initialize git: git init
    </requirement>
    <requirement id="3">
    Add all files: git add -A
    </requirement>
    <requirement id="4">
    Initial commit: git commit -m "Initial commit from {TEMPLATE_NAME} template"
    </requirement>
  </requirements>

  <test-requirements>
    <test id="1">
    Verify .git exists: test -d {TARGET_DIR}/.git
    </test>
    <test id="2">
    Verify commit exists: git -C {TARGET_DIR} log --oneline -1
    </test>
  </test-requirements>

  <files-to-create></files-to-create>

  <verification>
    <step>Run: git -C {TARGET_DIR} status - shows clean working tree</step>
    <step>Run: git -C {TARGET_DIR} log --oneline -1 - shows initial commit</step>
  </verification>

  <exports>
    <interface name="git_repo" type="status">
    Git repository initialized at {TARGET_DIR}
    Initial commit: "Initial commit from {TEMPLATE_NAME} template"
    </interface>
  </exports>
</task>
```

## L0-003: Set Up Environment (Python Template)

```xml
<task>
  <meta>
    <id>L0-003</id>
    <name>Set Up Python Environment</name>
    <layer>0-setup</layer>
    <priority>3</priority>
    <estimated-files>1</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
    Template: Python + FastAPI
    Location: {TARGET_DIR}
    </prd-excerpt>

    <tech-stack>Python 3.11, FastAPI, SQLAlchemy 2.x</tech-stack>
    <project-directory>{TARGET_DIR}</project-directory>
  </context>

  <dependencies>
    <interface name="git_repo" type="status">
    Git repo at {TARGET_DIR} from L0-002
    </interface>
  </dependencies>

  <objective>
  Create Python virtual environment, install dependencies, and configure .env file.
  </objective>

  <requirements>
    <requirement id="1">
    Create venv: python3 -m venv {TARGET_DIR}/.venv
    </requirement>
    <requirement id="2">
    Activate and install backend deps:
    source {TARGET_DIR}/.venv/bin/activate && pip install -r {TARGET_DIR}/requirements.txt
    </requirement>
    <requirement id="3">
    Install frontend deps: cd {TARGET_DIR}/frontend && npm install
    </requirement>
    <requirement id="4">
    Copy .env.example to .env: cp {TARGET_DIR}/.env.example {TARGET_DIR}/.env
    </requirement>
    <requirement id="5">
    Update DATABASE_URL in .env to use local PostgreSQL:
    DATABASE_URL=postgresql://postgres:postgres@localhost:5432/{PROJECT_SLUG}
    </requirement>
  </requirements>

  <test-requirements>
    <test id="1">
    Verify venv: test -f {TARGET_DIR}/.venv/bin/python
    </test>
    <test id="2">
    Verify .env: test -f {TARGET_DIR}/.env
    </test>
    <test id="3">
    Verify node_modules: test -d {TARGET_DIR}/frontend/node_modules
    </test>
  </test-requirements>

  <files-to-create>
    <file>{TARGET_DIR}/.env</file>
  </files-to-create>

  <verification>
    <step>Run: {TARGET_DIR}/.venv/bin/python --version - shows Python 3.11+</step>
    <step>Run: test -f {TARGET_DIR}/.env && echo "OK"</step>
  </verification>

  <exports>
    <interface name="environment" type="config">
    Virtual environment: {TARGET_DIR}/.venv
    Config file: {TARGET_DIR}/.env
    Frontend ready: node_modules installed
    </interface>
  </exports>
</task>
```

## L0-003: Set Up Environment (Go Template)

```xml
<task>
  <meta>
    <id>L0-003</id>
    <name>Set Up Go Environment</name>
    <layer>0-setup</layer>
    <priority>3</priority>
    <estimated-files>1</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
    Template: Go + Chi
    Location: {TARGET_DIR}
    </prd-excerpt>

    <tech-stack>Go 1.21+, Chi router, sqlx</tech-stack>
    <project-directory>{TARGET_DIR}</project-directory>
  </context>

  <dependencies>
    <interface name="git_repo" type="status">
    Git repo at {TARGET_DIR} from L0-002
    </interface>
  </dependencies>

  <objective>
  Download Go dependencies, install frontend packages, and configure .env file.
  </objective>

  <requirements>
    <requirement id="1">
    Download Go modules: cd {TARGET_DIR} && go mod download
    </requirement>
    <requirement id="2">
    Install frontend deps: cd {TARGET_DIR}/frontend && npm install
    </requirement>
    <requirement id="3">
    Copy .env.example to .env: cp {TARGET_DIR}/.env.example {TARGET_DIR}/.env
    </requirement>
    <requirement id="4">
    Update DATABASE_URL in .env:
    DATABASE_URL=postgres://postgres:postgres@localhost:5432/{PROJECT_SLUG}?sslmode=disable
    </requirement>
  </requirements>

  <test-requirements>
    <test id="1">
    Verify go.sum: test -f {TARGET_DIR}/go.sum
    </test>
    <test id="2">
    Verify .env: test -f {TARGET_DIR}/.env
    </test>
    <test id="3">
    Verify node_modules: test -d {TARGET_DIR}/frontend/node_modules
    </test>
  </test-requirements>

  <files-to-create>
    <file>{TARGET_DIR}/.env</file>
  </files-to-create>

  <verification>
    <step>Run: go version - shows Go 1.21+</step>
    <step>Run: test -f {TARGET_DIR}/.env && echo "OK"</step>
  </verification>

  <exports>
    <interface name="environment" type="config">
    Go modules: downloaded
    Config file: {TARGET_DIR}/.env
    Frontend ready: node_modules installed
    </interface>
  </exports>
</task>
```

## L0-003: Set Up Environment (TanStack Template)

```xml
<task>
  <meta>
    <id>L0-003</id>
    <name>Set Up TanStack Environment</name>
    <layer>0-setup</layer>
    <priority>3</priority>
    <estimated-files>1</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
    Template: TanStack Start
    Location: {TARGET_DIR}
    </prd-excerpt>

    <tech-stack>TypeScript, TanStack Start, Drizzle ORM</tech-stack>
    <project-directory>{TARGET_DIR}</project-directory>
  </context>

  <dependencies>
    <interface name="git_repo" type="status">
    Git repo at {TARGET_DIR} from L0-002
    </interface>
  </dependencies>

  <objective>
  Install npm dependencies and configure environment file.
  </objective>

  <requirements>
    <requirement id="1">
    Install dependencies: cd {TARGET_DIR} && npm install
    </requirement>
    <requirement id="2">
    Copy .env.example to .env: cp {TARGET_DIR}/.env.example {TARGET_DIR}/.env
    </requirement>
    <requirement id="3">
    Update DATABASE_URL in .env:
    DATABASE_URL=postgres://postgres:postgres@localhost:5432/{PROJECT_SLUG}
    </requirement>
  </requirements>

  <test-requirements>
    <test id="1">
    Verify node_modules: test -d {TARGET_DIR}/node_modules
    </test>
    <test id="2">
    Verify .env: test -f {TARGET_DIR}/.env
    </test>
  </test-requirements>

  <files-to-create>
    <file>{TARGET_DIR}/.env</file>
  </files-to-create>

  <verification>
    <step>Run: node --version - shows Node 18+</step>
    <step>Run: test -f {TARGET_DIR}/.env && echo "OK"</step>
  </verification>

  <exports>
    <interface name="environment" type="config">
    Node modules: installed
    Config file: {TARGET_DIR}/.env
    </interface>
  </exports>
</task>
```

## L0-004: Run Database Migrations

```xml
<task>
  <meta>
    <id>L0-004</id>
    <name>Start Database and Run Migrations</name>
    <layer>0-setup</layer>
    <priority>4</priority>
    <estimated-files>0</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
    Database: PostgreSQL via Docker Compose
    Location: {TARGET_DIR}
    </prd-excerpt>

    <project-directory>{TARGET_DIR}</project-directory>
  </context>

  <dependencies>
    <interface name="environment" type="config">
    .env configured from L0-003
    DATABASE_URL set to local PostgreSQL
    </interface>
  </dependencies>

  <objective>
  Start PostgreSQL via Docker Compose and run initial database migrations.
  </objective>

  <requirements>
    <requirement id="1">
    Start database: cd {TARGET_DIR} && docker compose up -d db
    </requirement>
    <requirement id="2">
    Wait for database: sleep 5 (or use docker compose wait)
    </requirement>
    <requirement id="3">
    Run migrations (Python): cd {TARGET_DIR} && source .venv/bin/activate && alembic upgrade head
    OR (Go): cd {TARGET_DIR} && make migrate-up
    OR (TanStack): cd {TARGET_DIR} && npm run db:migrate
    </requirement>
  </requirements>

  <test-requirements>
    <test id="1">
    Verify container running: docker compose -f {TARGET_DIR}/docker-compose.yml ps db
    </test>
    <test id="2">
    Verify can connect: psql postgresql://postgres:postgres@localhost:5432/{PROJECT_SLUG} -c "SELECT 1"
    </test>
  </test-requirements>

  <files-to-create></files-to-create>

  <verification>
    <step>Run: docker compose -f {TARGET_DIR}/docker-compose.yml ps - shows db running</step>
    <step>Run migration command - completes without error</step>
  </verification>

  <exports>
    <interface name="database" type="status">
    PostgreSQL running on localhost:5432
    Database: {PROJECT_SLUG}
    Migrations: applied
    </interface>
  </exports>
</task>
```

## L0-005: Verify Setup

```xml
<task>
  <meta>
    <id>L0-005</id>
    <name>Verify Development Environment</name>
    <layer>0-setup</layer>
    <priority>5</priority>
    <estimated-files>0</estimated-files>
  </meta>

  <context>
    <prd-excerpt>
    Verify the complete development environment is working.
    </prd-excerpt>

    <project-directory>{TARGET_DIR}</project-directory>
  </context>

  <dependencies>
    <interface name="database" type="status">
    Database running and migrated from L0-004
    </interface>
  </dependencies>

  <objective>
  Start the development server and verify the application runs correctly.
  </objective>

  <requirements>
    <requirement id="1">
    Start dev server: cd {TARGET_DIR} && make dev
    (or npm run dev for TanStack)
    </requirement>
    <requirement id="2">
    Verify backend: curl http://localhost:{BACKEND_PORT}/health returns 200
    </requirement>
    <requirement id="3">
    Verify frontend: curl http://localhost:3000 returns HTML
    </requirement>
    <requirement id="4">
    Stop dev server after verification
    </requirement>
  </requirements>

  <test-requirements>
    <test id="1">
    Backend health check: curl -s http://localhost:{BACKEND_PORT}/health | grep -q "ok"
    </test>
    <test id="2">
    Frontend loads: curl -s http://localhost:3000 | grep -q "html"
    </test>
  </test-requirements>

  <files-to-create></files-to-create>

  <verification>
    <step>Run: make dev (in background)</step>
    <step>Run: curl http://localhost:{BACKEND_PORT}/health - returns OK</step>
    <step>Run: curl http://localhost:3000 - returns HTML page</step>
  </verification>

  <exports>
    <interface name="dev_environment" type="status">
    Development environment fully operational at {TARGET_DIR}
    Backend: http://localhost:{BACKEND_PORT}
    Frontend: http://localhost:3000
    Ready for feature development
    </interface>
  </exports>
</task>
```

## Template Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `{TEMPLATE_PATH}` | Source template directory | `webapps/backends/python` |
| `{TEMPLATE_NAME}` | Template identifier | `python`, `go`, `tanstack` |
| `{TARGET_DIR}` | Destination project path | `/home/user/projects/my-app` |
| `{PROJECT_NAME}` | Human-readable project name | `My Voice PRD App` |
| `{PROJECT_SLUG}` | URL-safe project identifier | `my-voice-prd-app` |
| `{TECH_STACK_SUMMARY}` | Brief tech description | `Python 3.11, FastAPI, PostgreSQL` |
| `{BACKEND_PORT}` | Backend server port | `8000` (Python), `8080` (Go) |

## Template Detection

| Template | Keywords in PRD | Backend Port |
|----------|----------------|--------------|
| `python` | Python, FastAPI | 8000 |
| `go` | Go, Chi | 8080 |
| `tanstack` | TanStack, TanStack Start | 3000 (unified) |
