---
layout: default
title: Interface Contracts
parent: Concepts
nav_order: 2
---

# Interface Contracts

Interface contracts are type signatures that flow between tasks, enabling type-safe boundaries without shared context.

## The Problem

In parallel execution, tasks can't see each other's work:

```
┌─────────────┐     ┌─────────────┐
│  Task L1-A  │     │  Task L2-B  │
│  Creates    │  ?  │  Needs      │
│  User model │────►│  User model │
└─────────────┘     └─────────────┘

How does L2-B know what L1-A created?
```

Tasks run in isolated contexts. L2-B can't read L1-A's files during execution.

## The Solution: Contracts

Tasks **export** their interfaces. Dependent tasks receive them as **contracts**:

```
┌─────────────────────────────────────────────────────────────────┐
│  Breakdown Phase                                                 │
│                                                                  │
│  L1-A.xml                        L2-B.xml                       │
│  ┌────────────────────┐          ┌────────────────────┐         │
│  │ <exports>          │          │ <dependencies>     │         │
│  │   <export>         │  ───►    │   <dependency      │         │
│  │     class User:    │  copy    │     task="L1-A">   │         │
│  │       id: UUID     │          │     class User:    │         │
│  │       email: str   │          │       id: UUID     │         │
│  │   </export>        │          │       email: str   │         │
│  │ </exports>         │          │   </dependency>    │         │
│  └────────────────────┘          └────────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

The contract is **copied** into the dependent task at breakdown time.

## Contract Flow

```
Layer 1                    Layer 2                    Layer 3
┌────────────┐            ┌────────────┐            ┌────────────┐
│ L1-001     │            │ L2-001     │            │ L3-001     │
│            │            │            │            │            │
│ exports:   │ ────────►  │ depends:   │            │            │
│  User      │            │  User      │            │            │
│            │            │            │            │            │
│            │            │ exports:   │ ────────►  │ depends:   │
│            │            │  auth_     │            │  auth_     │
│            │            │  router    │            │  router    │
└────────────┘            └────────────┘            └────────────┘
```

Contracts form a chain of type-safe dependencies.

## Export Format

Tasks declare exports explicitly:

```xml
<exports>
  <export name="User">
    from sqlalchemy.orm import DeclarativeBase
    from uuid import UUID

    class Base(DeclarativeBase):
        pass

    class User(Base):
        __tablename__ = "users"
        id: UUID
        email: str
        hashed_password: str
        is_active: bool = True
        created_at: datetime
  </export>

  <export name="get_user_by_email">
    async def get_user_by_email(
        db: AsyncSession,
        email: str
    ) -> User | None:
        """Retrieve user by email address."""
  </export>
</exports>
```

### Export Guidelines

1. **Include imports** - The signature should be copy-pasteable
2. **Include types** - Full type annotations
3. **Include docstrings** - Brief description of purpose
4. **Minimal scope** - Only what others need

## Dependency Format

Dependent tasks receive contracts:

```xml
<dependencies>
  <dependency task="L1-001-user-model">
    <interface-contract>
      from sqlalchemy.orm import DeclarativeBase
      from uuid import UUID

      class User(Base):
          __tablename__ = "users"
          id: UUID
          email: str
          hashed_password: str
          is_active: bool = True
    </interface-contract>
  </dependency>

  <dependency task="L1-002-database">
    <interface-contract>
      from sqlalchemy.ext.asyncio import AsyncSession

      async def get_db() -> AsyncGenerator[AsyncSession, None]:
          """FastAPI dependency for database sessions."""
    </interface-contract>
  </dependency>
</dependencies>
```

### Using Contracts

The implementing agent uses contracts directly:

```python
# In L2-001-auth-api implementation

# Import from the contract (L1-001)
from src.models.user import User

# Import from the contract (L1-002)
from src.database import get_db

# Use the exact signatures
async def create_user(
    user_data: UserCreate,
    db: AsyncSession = Depends(get_db)
) -> User:
    user = User(
        email=user_data.email,
        hashed_password=hash_password(user_data.password)
    )
    db.add(user)
    await db.commit()
    return user
```

## Contract Verification

At merge time, contracts are verified:

```
┌─────────────────────────────────────────────────────────────────┐
│  execute-merge                                                   │
│                                                                  │
│  1. Merge L1-001 (User model)                                   │
│  2. Verify exports match declared contracts                      │
│     ├── User class exists? ✓                                    │
│     ├── id: UUID? ✓                                             │
│     ├── email: str? ✓                                           │
│     └── hashed_password: str? ✓                                 │
│  3. Proceed to next task                                        │
└─────────────────────────────────────────────────────────────────┘
```

If a task's implementation doesn't match its declared exports, merge fails.

## Multiple Dependencies

Tasks often depend on multiple prior tasks:

```xml
<dependencies>
  <dependency task="L1-001-user-model">
    <interface-contract>
      class User(Base):
          id: UUID
          email: str
    </interface-contract>
  </dependency>

  <dependency task="L1-002-project-model">
    <interface-contract>
      class Project(Base):
          id: UUID
          name: str
          owner_id: UUID  # FK to User
    </interface-contract>
  </dependency>

  <dependency task="L1-003-database">
    <interface-contract>
      async def get_db() -> AsyncGenerator[AsyncSession, None]:
          pass
    </interface-contract>
  </dependency>
</dependencies>
```

## Contract Evolution

When requirements change, contracts update through breakdown:

```
Version 1:                    Version 2:
┌────────────────────┐        ┌────────────────────┐
│ class User:        │   ►    │ class User:        │
│     id: UUID       │        │     id: UUID       │
│     email: str     │        │     email: str     │
│                    │        │     role: UserRole │ ← Added
└────────────────────┘        └────────────────────┘
```

All dependent tasks are regenerated with the new contract.

## Best Practices

### Do

- Include complete type signatures
- Add necessary imports
- Document public interfaces
- Keep exports minimal

### Don't

- Export implementation details
- Include private methods
- Reference internal state
- Assume runtime behavior
