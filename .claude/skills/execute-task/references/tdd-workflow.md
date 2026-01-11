# TDD Workflow for Task Implementation

## Overview

All tasks follow Test-Driven Development (TDD). Write tests first, then implement code to pass them.

## Workflow Steps

### 1. Read Test Requirements

From the task XML `<test-requirements>` section:

```xml
<test-requirements>
  <test id="1">
  Test file: `tests/models/test_enums.py`
  </test>
  <test id="2">
  Test: test_project_status_values
  - Assert ProjectStatus.draft.value == "draft"
  - Assert ProjectStatus.in_progress.value == "in_progress"
  - Assert ProjectStatus.complete.value == "complete"
  </test>
  <test id="3">
  Test: test_persona_type_values
  - Assert PersonaType.developer.value == "developer"
  - Assert PersonaType.designer.value == "designer"
  </test>
</test-requirements>
```

### 2. Create Test File

Create the test file with all test cases:

```python
# tests/models/test_enums.py
import pytest
from app.models.enums import ProjectStatus, PersonaType


class TestProjectStatus:
    def test_project_status_values(self):
        assert ProjectStatus.draft.value == "draft"
        assert ProjectStatus.in_progress.value == "in_progress"
        assert ProjectStatus.complete.value == "complete"


class TestPersonaType:
    def test_persona_type_values(self):
        assert PersonaType.developer.value == "developer"
        assert PersonaType.designer.value == "designer"
```

### 3. Verify Tests Fail (Red Phase)

Run tests to confirm they fail (implementation doesn't exist yet):

```bash
pytest tests/models/test_enums.py -v
# Expected: ImportError or test failures
```

This confirms tests are correctly written and will detect implementation.

### 4. Implement Code (Green Phase)

Create the implementation to make tests pass:

```python
# app/models/enums.py
from enum import Enum


class ProjectStatus(Enum):
    draft = "draft"
    in_progress = "in_progress"
    complete = "complete"


class PersonaType(Enum):
    developer = "developer"
    designer = "designer"
```

### 5. Run Tests Again

```bash
pytest tests/models/test_enums.py -v
# Expected: All tests pass
```

### 6. Refactor if Needed

If tests pass, optionally refactor while keeping tests green.

## Test Quality Guidelines

### Use Concrete Values

**Good:**
```python
def test_project_default_status(self):
    project = Project(name="Test Project", slug="test-project")
    assert project.status == ProjectStatus.draft
```

**Bad:**
```python
def test_project_default_status(self):
    project = Project(name=some_name, slug=some_slug)
    assert project.status is not None  # Too vague
```

### Test One Thing Per Test

**Good:**
```python
def test_project_name_required(self):
    with pytest.raises(ValueError):
        Project(name="", slug="test")

def test_project_slug_required(self):
    with pytest.raises(ValueError):
        Project(name="Test", slug="")
```

**Bad:**
```python
def test_project_validation(self):
    # Tests too many things at once
    ...
```

### Use Descriptive Test Names

Names should describe what's being tested:

- `test_project_status_default_is_draft`
- `test_create_project_with_valid_data`
- `test_project_slug_must_be_unique`

### Follow AAA Pattern

**Arrange** - Set up test data
**Act** - Execute the code under test
**Assert** - Verify the results

```python
def test_project_creation(self):
    # Arrange
    name = "Test Project"
    slug = "test-project"

    # Act
    project = Project(name=name, slug=slug)

    # Assert
    assert project.name == name
    assert project.slug == slug
    assert project.status == ProjectStatus.draft
```

## Common Test Patterns

### Testing Enums

```python
def test_enum_values(self):
    assert MyEnum.value_a.value == "value_a"
    assert len(MyEnum) == 3  # Total enum members

def test_enum_from_string(self):
    assert MyEnum("value_a") == MyEnum.value_a
```

### Testing Models

```python
def test_model_creation(self):
    obj = MyModel(field1="value", field2=123)
    assert obj.field1 == "value"
    assert obj.field2 == 123

def test_model_defaults(self):
    obj = MyModel(field1="value")
    assert obj.optional_field is None
    assert obj.created_at is not None
```

### Testing API Endpoints

```python
def test_create_endpoint(self, client):
    response = client.post("/api/items", json={"name": "Test"})
    assert response.status_code == 201
    assert response.json()["name"] == "Test"

def test_validation_error(self, client):
    response = client.post("/api/items", json={})
    assert response.status_code == 422
```

### Testing Services

```python
def test_service_success(self, db_session):
    service = MyService(db_session)
    result = service.do_something(input="test")
    assert result.success is True

def test_service_failure(self, db_session):
    service = MyService(db_session)
    with pytest.raises(ValidationError):
        service.do_something(input="invalid")
```
