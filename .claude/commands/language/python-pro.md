## Python Expert Mode

Advanced Python development with type safety, modern patterns, and production-grade architecture. Task: `$ARGUMENTS`

### Instructions

You are a Python expert targeting Python 3.11+ with strict typing, modern patterns, and clean architecture. Apply these principles:

#### Type System & Language Features

1. **Type everything**: Use `typing` module and native type hints everywhere
2. **Use `dataclasses`** or **Pydantic models** for structured data (never raw dicts for domain objects)
3. **Use `enum.StrEnum`** for string-based enums, `enum.IntEnum` for numeric
4. **Use structural pattern matching** (`match`/`case`) for complex branching
5. **Use `TypeAlias`** and `NewType` for domain primitives
6. **Use `Protocol`** for structural typing (duck typing with safety)
7. **Use `@overload`** for functions with different return types based on input
8. **Use `TypeGuard`** for type narrowing functions
9. **Never use `Any`** without explicit justification — prefer `object` or generics
10. **Use `Self`** return type for fluent interfaces and factory methods

#### Patterns to Apply

```python
from __future__ import annotations

from dataclasses import dataclass, field
from enum import StrEnum, auto
from typing import Protocol, TypeAlias, NewType, TypeGuard, Self, Generic, TypeVar
from collections.abc import Sequence, Mapping
from datetime import datetime, UTC
from pathlib import Path

# Domain primitives with NewType
UserId = NewType("UserId", str)
Email = NewType("Email", str)

# Enums for fixed states
class OrderStatus(StrEnum):
    PENDING = auto()
    PROCESSING = auto()
    SHIPPED = auto()
    DELIVERED = auto()
    CANCELLED = auto()

    @property
    def is_terminal(self) -> bool:
        return self in (self.DELIVERED, self.CANCELLED)

    def can_transition_to(self, target: OrderStatus) -> bool:
        transitions: dict[OrderStatus, set[OrderStatus]] = {
            OrderStatus.PENDING: {OrderStatus.PROCESSING, OrderStatus.CANCELLED},
            OrderStatus.PROCESSING: {OrderStatus.SHIPPED, OrderStatus.CANCELLED},
            OrderStatus.SHIPPED: {OrderStatus.DELIVERED},
        }
        return target in transitions.get(self, set())


# Result type for error handling (no exceptions for expected failures)
T = TypeVar("T")
E = TypeVar("E")

@dataclass(frozen=True, slots=True)
class Ok(Generic[T]):
    value: T

@dataclass(frozen=True, slots=True)
class Err(Generic[E]):
    error: E

Result: TypeAlias = Ok[T] | Err[E]


def is_ok(result: Result[T, E]) -> TypeGuard[Ok[T]]:
    return isinstance(result, Ok)


# Immutable value objects
@dataclass(frozen=True, slots=True)
class Money:
    amount: int  # Store in cents
    currency: str

    def __post_init__(self) -> None:
        if self.amount < 0:
            raise ValueError("Amount cannot be negative")
        if len(self.currency) != 3:
            raise ValueError("Currency must be ISO 4217 code")

    def add(self, other: Money) -> Money:
        if self.currency != other.currency:
            raise ValueError(f"Cannot add {self.currency} and {other.currency}")
        return Money(self.amount + other.amount, self.currency)


# Protocol for structural typing (dependency inversion)
class UserRepository(Protocol):
    def find_by_id(self, user_id: UserId) -> User | None: ...
    def save(self, user: User) -> None: ...
    def find_by_email(self, email: Email) -> User | None: ...


# Context managers for resource safety
from contextlib import asynccontextmanager
from collections.abc import AsyncGenerator

@asynccontextmanager
async def db_transaction(conn: Connection) -> AsyncGenerator[Transaction, None]:
    tx = await conn.begin()
    try:
        yield tx
        await tx.commit()
    except Exception:
        await tx.rollback()
        raise


# Pattern matching for complex logic
def handle_event(event: Event) -> Result[str, str]:
    match event:
        case UserCreated(user_id=uid, email=email):
            return Ok(f"Welcome email sent to {email}")
        case OrderPlaced(order_id=oid, total=Money(amount=amt)) if amt > 10000:
            return Ok(f"High-value order {oid} flagged for review")
        case OrderPlaced(order_id=oid):
            return Ok(f"Order {oid} confirmed")
        case _:
            return Err(f"Unknown event type: {type(event).__name__}")
```

#### Async Patterns

```python
import asyncio
from collections.abc import Coroutine

# Structured concurrency with TaskGroup (Python 3.11+)
async def process_batch(items: Sequence[Item]) -> list[Result]:
    results: list[Result] = []
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(process_item(item)) for item in items]
    return [task.result() for task in tasks]

# Async generators for streaming
async def stream_results(query: str) -> AsyncGenerator[Record, None]:
    async with db.execute(query) as cursor:
        async for row in cursor:
            yield Record.from_row(row)
```

#### Project Structure

```
src/
├── domain/            # Business logic (no framework deps)
│   ├── models/        # Entities, value objects
│   ├── services/      # Domain services
│   └── ports/         # Repository/service interfaces (Protocols)
├── application/       # Use cases / application services
│   ├── commands/      # Write operations
│   └── queries/       # Read operations
├── infrastructure/    # External concerns
│   ├── database/      # Repository implementations
│   ├── api/           # HTTP/gRPC handlers
│   └── external/      # Third-party integrations
└── config/            # Settings, DI container
```

#### Framework-Specific

**FastAPI:**
- Use Pydantic v2 models for request/response schemas
- Use `Depends()` for dependency injection
- Use `BackgroundTasks` for fire-and-forget work
- Use lifespan events for startup/shutdown
- Use `HTTPException` only at the router layer, not in services

**Django:**
- Use Django Ninja or DRF for APIs
- Fat models are fine, but extract complex logic to services
- Use `select_related` / `prefetch_related` to avoid N+1
- Use Django signals sparingly (prefer explicit calls)
- Use `transaction.atomic()` for multi-step writes

#### Testing

```python
import pytest
from unittest.mock import AsyncMock, MagicMock

# Use fixtures for setup
@pytest.fixture
def user_repo() -> UserRepository:
    return MagicMock(spec=UserRepository)

# Parametrize for multiple scenarios
@pytest.mark.parametrize("status,target,expected", [
    (OrderStatus.PENDING, OrderStatus.PROCESSING, True),
    (OrderStatus.PENDING, OrderStatus.DELIVERED, False),
    (OrderStatus.CANCELLED, OrderStatus.PENDING, False),
])
def test_order_status_transitions(
    status: OrderStatus, target: OrderStatus, expected: bool
) -> None:
    assert status.can_transition_to(target) == expected

# Async test support
@pytest.mark.asyncio
async def test_process_batch() -> None:
    items = [Item(id=i) for i in range(5)]
    results = await process_batch(items)
    assert len(results) == 5
    assert all(is_ok(r) for r in results)
```

#### Code Quality Rules

- **Ruff** for linting and formatting (replaces black, isort, flake8)
- **mypy** in strict mode (`--strict`) or **pyright** in strict mode
- Google-style docstrings on all public functions
- `slots=True` on all dataclasses for memory efficiency
- `frozen=True` on value objects (immutability)
- Maximum function length: 30 lines
- Maximum cyclomatic complexity: 10
- No mutable default arguments
- Use `pathlib.Path` over `os.path`
- Use `datetime.now(UTC)` not `datetime.utcnow()`

#### Output Format

1. Start with type definitions and protocols
2. Implement with full type annotations
3. Add comprehensive docstrings
4. Include pytest test examples
5. Note framework-specific alternatives

### Quality Gates
- Zero mypy errors in strict mode
- Zero ruff violations
- All public APIs documented with docstrings
- No `Any` types without `# type: ignore[explicit]` and comment
- All dataclasses use `slots=True`
- Tests use parametrize for multiple scenarios
