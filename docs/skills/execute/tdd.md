# TDD Workflow

Test-Driven Development in task execution.

---

## Overview

Every task follows a strict TDD workflow:

```
1. Read requirements ──────────────────────────►
2. Write tests ◄─── From <test-requirements>
3. Run tests ──────► Should FAIL (red)
4. Implement ◄───── From <requirements>
5. Run tests ──────► Should PASS (green)
6. Commit ──────────────────────────────────────►
7. Verify ─────────► Independent verification
```

---

## Why TDD?

### 1. Specification Clarity

Tests define exactly what the code should do:

```python
def test_create_task_with_empty_title():
    response = client.post("/api/tasks", json={"title": ""})
    assert response.status_code == 422
```

No ambiguity about edge cases.

### 2. Early Failure Detection

If tests don't fail initially, something is wrong:
- Test not testing the right thing
- Implementation already exists
- Test is broken

### 3. Verification Confidence

When tests pass, we know:
- Implementation matches specification
- Edge cases handled
- Contract fulfilled

### 4. Independent Verification

The verifier only runs tests - it doesn't need to understand implementation.

---

## The Red-Green Cycle

### Red Phase (Tests Fail)

```
Task starts
    │
    ├── Read <test-requirements> from task XML
    │
    ├── Create test file
    │   File: tests/api/test_tasks.py
    │
    ├── Write test code:
    │   def test_create_task():
    │       response = client.post("/api/tasks", json={"title": "Test"})
    │       assert response.status_code == 201
    │
    └── Run: pytest tests/api/test_tasks.py
            │
            └── Expected: FAIL (no implementation yet)
```

If tests pass at this point, the task stops and reports an error - either the test is wrong or the code already exists.

### Green Phase (Tests Pass)

```
Tests are red
    │
    ├── Read <requirements> from task XML
    │
    ├── Implement code:
    │   # src/api/tasks.py
    │   @router.post("/")
    │   async def create_task(request: TaskCreateRequest):
    │       task = await repository.create(...)
    │       return TaskResponse.from_orm(task)
    │
    └── Run: pytest tests/api/test_tasks.py
            │
            └── Expected: PASS
```

---

## Test Requirements Format

From task XML:

```xml
<test-requirements>
  <test id="T1">
    File: tests/api/test_tasks.py

    Test: test_create_task_success
    Setup:
      - Create mock TaskRepository
      - Mock create() to return Task(id=uuid4(), title="Test", ...)
      - Create TestClient with app
    Action:
      - POST /api/tasks with {"title": "Test task"}
    Assert:
      - response.status_code == 201
      - response.json()["title"] == "Test task"
  </test>

  <test id="T2">
    Test: test_create_task_empty_title
    Setup:
      - Create TestClient with app
    Action:
      - POST /api/tasks with {"title": ""}
    Assert:
      - response.status_code == 422
  </test>
</test-requirements>
```

### Key Elements

| Element | Purpose |
|---------|---------|
| `File:` | Where to write tests |
| `Test:` | Test function name |
| `Setup:` | Preconditions and mocks |
| `Action:` | What to do |
| `Assert:` | Expected outcomes |

### Concrete Values

Tests use specific values:

```
Good:
  Assert: response.status_code == 201
  Assert: response.json()["title"] == "Test task"

Bad:
  Assert: response.status_code is success
  Assert: response contains the title
```

---

## Implementation Flow

```python
# Step 1: Read test requirements
test_file = "tests/api/test_tasks.py"
tests = parse_test_requirements(task_xml)

# Step 2: Write tests
write_file(test_file, generate_test_code(tests))

# Step 3: Run tests (expect failure)
result = run_pytest(test_file)
if result.passed:
    raise Error("Tests should fail before implementation!")

# Step 4: Read requirements
requirements = parse_requirements(task_xml)

# Step 5: Implement
for req in requirements:
    write_file(req.file, generate_implementation(req))

# Step 6: Run tests (expect success)
result = run_pytest(test_file)
if not result.passed:
    # On first attempt: this is expected, will retry
    # On retry: use feedback to fix
    pass
```

---

## Commit Format

After tests pass:

```
L2-003: Implement Task API endpoints

Requirements:
- R1: Create src/api/tasks.py with CRUD endpoints
- R2: Create src/api/schemas/task.py with Pydantic models

Tests:
- T1: test_create_task_success
- T2: test_create_task_empty_title
- T3: test_get_task_not_found

Files:
- src/api/tasks.py
- src/api/schemas/task.py
- tests/api/test_tasks.py
```

Fix commits (on retry):

```
L2-003: Fix title validation

Feedback: test_create_task_empty_title failed
Fix: Added min_length=1 to TaskCreateRequest.title

Attempt: 2
```

---

## Verification

Independent verification runs the same tests:

```
execute-verify receives:
    │
    ├── Task XML (with verification steps)
    │
    └── Does NOT receive:
            - Implementation code
            - Implementation decisions
            - Previous errors

Verification steps:
    │
    ├── Run: pytest tests/api/test_tasks.py -v
    │       │
    │       └── Check: All tests pass
    │
    └── Verify: Expected outcomes met
```

This ensures:
- Tests actually verify the requirements
- Implementation wasn't "faked"
- No bias from seeing implementation

---

## Retry with Feedback

When verification fails:

```
Attempt 1:
    │
    ├── Write tests
    ├── Implement
    ├── Commit
    └── Verify → FAIL
            │
            └── Feedback: "test_create_task_empty_title failed:
                          expected 422, got 201. Add validation
                          for empty title in TaskCreateRequest."

Attempt 2:
    │
    ├── Read feedback
    ├── Apply fix:
    │       title: str = Field(min_length=1, max_length=200)
    │
    ├── Commit fix
    └── Verify → PASS
```

Feedback is actionable:
- Specific test that failed
- Expected vs actual
- Suggested fix

---

## Maximum Attempts

```
Attempt 1: FAIL → Retry with feedback
Attempt 2: FAIL → Retry with feedback
Attempt 3: FAIL → Retry with feedback
Attempt 4: FAIL → Retry with feedback
Attempt 5: FAIL → ABANDON

Task abandoned:
- Worktree preserved
- Execution stops
- User can fix manually
```

---

## Best Practices

### 1. Test First, Always

Never implement before writing tests. The red phase is essential.

### 2. One Behavior Per Test

```python
# Good
def test_create_task_success(): ...
def test_create_task_empty_title(): ...
def test_create_task_title_too_long(): ...

# Bad
def test_create_task():
    # Tests success, empty, and too long in one test
```

### 3. Mock External Dependencies

```python
def test_create_task(mock_repository):
    mock_repository.create.return_value = Task(id=uuid4(), ...)
    # Test without hitting database
```

### 4. Use Concrete Values

```python
# Good
assert response.json()["title"] == "My Task"

# Bad
assert "title" in response.json()
```

---

## Next Steps

- [Verification Process](sub-skills.md#execute-verify)
- [State Management](state.md)
- [Execute Overview](README.md)
