# Module 2: Modern Python Refresher

## Learning Objectives

By the end of this module, you will:
- Understand Python type hints and why they matter
- Master async/await for concurrent programming
- Know modern Python features (3.8-3.12)
- Use dataclasses and Pydantic for data modeling
- Understand Python packaging with pyproject.toml

## 2.1 Type Hints (Python 3.5+, improved in 3.9+)

### Why Type Hints?

Coming from PHP, you might remember the days of "what type is this variable?" Type hints solve this:

```python
# Old Python (still valid, but harder to maintain)
def greet(name):
    return f"Hello, {name}"

# Modern Python with type hints
def greet(name: str) -> str:
    return f"Hello, {name}"
```

**Benefits:**
- IDE autocompletion and error detection
- Self-documenting code
- Catch bugs before runtime
- Required by FastAPI/Pydantic for validation

### Basic Types

```python
from typing import Optional, Union
from collections.abc import Sequence

# Basic types
name: str = "Alice"
age: int = 30
price: float = 19.99
is_active: bool = True

# None and Optional
middle_name: str | None = None          # Python 3.10+ syntax
middle_name: Optional[str] = None       # Older syntax (same meaning)

# Collections (Python 3.9+)
names: list[str] = ["Alice", "Bob"]
ages: dict[str, int] = {"Alice": 30, "Bob": 25}
coordinates: tuple[float, float] = (1.0, 2.0)
unique_ids: set[int] = {1, 2, 3}

# Union types
id_value: int | str = "abc123"          # Python 3.10+
id_value: Union[int, str] = "abc123"    # Older syntax
```

### Function Signatures

```python
from collections.abc import Callable, Sequence

def process_items(
    items: list[str],
    transformer: Callable[[str], str],
    limit: int | None = None
) -> list[str]:
    """Process items with a transformer function."""
    result = [transformer(item) for item in items]
    if limit:
        return result[:limit]
    return result

# Usage
processed = process_items(
    ["hello", "world"],
    transformer=str.upper,
    limit=1
)
```

### Generic Types

```python
from typing import TypeVar, Generic

T = TypeVar("T")

class Repository(Generic[T]):
    """Generic repository for any entity type."""

    def __init__(self):
        self._items: list[T] = []

    def add(self, item: T) -> None:
        self._items.append(item)

    def get_all(self) -> list[T]:
        return self._items.copy()

# Usage
class User:
    def __init__(self, name: str):
        self.name = name

user_repo: Repository[User] = Repository()
user_repo.add(User("Alice"))
```

### TypedDict for Structured Dictionaries

```python
from typing import TypedDict, NotRequired

class JobPosting(TypedDict):
    title: str
    company: str
    salary_min: int
    salary_max: int
    remote: NotRequired[bool]  # Optional key

# Type checker knows the structure
job: JobPosting = {
    "title": "Python Developer",
    "company": "TechCorp",
    "salary_min": 80000,
    "salary_max": 120000
}
```

---

## 2.2 Async/Await (Essential for FastAPI)

### Why Async?

Web servers handle many concurrent requests. Async allows handling thousands of connections without threads:

```python
# Synchronous - blocks while waiting
import time

def fetch_data():
    time.sleep(1)  # Simulates I/O (database, API call)
    return "data"

# 3 calls = 3 seconds total
result1 = fetch_data()
result2 = fetch_data()
result3 = fetch_data()
```

```python
# Asynchronous - doesn't block
import asyncio

async def fetch_data():
    await asyncio.sleep(1)  # Non-blocking wait
    return "data"

# 3 calls = ~1 second total (concurrent)
async def main():
    results = await asyncio.gather(
        fetch_data(),
        fetch_data(),
        fetch_data()
    )
    return results

asyncio.run(main())
```

### Basic Syntax

```python
import asyncio

# Define async function with 'async def'
async def greet(name: str) -> str:
    await asyncio.sleep(0.1)  # Simulate async operation
    return f"Hello, {name}"

# Call with 'await' (must be inside async function)
async def main():
    message = await greet("World")
    print(message)

# Run the async function
asyncio.run(main())
```

### Concurrent Execution

```python
import asyncio

async def fetch_user(user_id: int) -> dict:
    await asyncio.sleep(0.5)  # Simulate DB call
    return {"id": user_id, "name": f"User {user_id}"}

async def fetch_posts(user_id: int) -> list:
    await asyncio.sleep(0.5)  # Simulate DB call
    return [{"id": 1, "title": "Post 1"}]

async def get_user_with_posts(user_id: int):
    # Sequential - takes 1 second
    # user = await fetch_user(user_id)
    # posts = await fetch_posts(user_id)

    # Concurrent - takes 0.5 seconds
    user, posts = await asyncio.gather(
        fetch_user(user_id),
        fetch_posts(user_id)
    )
    return {"user": user, "posts": posts}
```

### Async Context Managers

```python
import asyncio
from contextlib import asynccontextmanager

@asynccontextmanager
async def get_db_connection():
    print("Opening connection")
    conn = "fake_connection"  # Would be real DB connection
    try:
        yield conn
    finally:
        print("Closing connection")

async def main():
    async with get_db_connection() as conn:
        print(f"Using {conn}")

asyncio.run(main())
```

### FastAPI Uses Async Naturally

```python
from fastapi import FastAPI

app = FastAPI()

# Async endpoint - handles concurrent requests efficiently
@app.get("/users/{user_id}")
async def get_user(user_id: int):
    # Async database call
    user = await database.fetch_user(user_id)
    return user

# Sync endpoint - also works, but blocks the worker
@app.get("/sync-endpoint")
def sync_endpoint():
    return {"message": "This works too"}
```

---

## 2.3 Modern Python Features

### Walrus Operator := (Python 3.8)

Assign and use a value in one expression:

```python
# Before
line = input()
while line != "quit":
    print(line)
    line = input()

# After
while (line := input()) != "quit":
    print(line)

# Useful in comprehensions
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
filtered = [y for x in data if (y := x * 2) > 10]
# [12, 14, 16, 18, 20]
```

### Pattern Matching (Python 3.10)

Like switch/case but much more powerful:

```python
def handle_response(response: dict):
    match response:
        case {"status": "success", "data": data}:
            return process_data(data)
        case {"status": "error", "message": msg}:
            raise ValueError(msg)
        case {"status": status}:
            print(f"Unknown status: {status}")
        case _:
            print("Invalid response format")

# With type patterns
def describe(value):
    match value:
        case int():
            return "It's an integer"
        case str():
            return "It's a string"
        case [first, *rest]:
            return f"List starting with {first}"
        case _:
            return "Something else"
```

### F-strings Improvements

```python
name = "Alice"
age = 30

# Basic (Python 3.6+)
print(f"Name: {name}, Age: {age}")

# Expressions
print(f"In 5 years: {age + 5}")

# Debug mode (Python 3.8+) - prints variable name and value
print(f"{name=}, {age=}")  # name='Alice', age=30

# Format specifiers
price = 19.99
print(f"Price: {price:.2f}")        # Price: 19.99
print(f"Price: {price:>10.2f}")     # Price:      19.99

# Date formatting
from datetime import datetime
now = datetime.now()
print(f"Today: {now:%Y-%m-%d}")     # Today: 2024-01-15
```

### Dataclasses (Python 3.7+)

Reduce boilerplate for data containers:

```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class Job:
    title: str
    company: str
    salary_min: int
    salary_max: int
    remote: bool = False
    created_at: datetime = field(default_factory=datetime.now)

    @property
    def salary_range(self) -> str:
        return f"${self.salary_min:,} - ${self.salary_max:,}"

# Auto-generates __init__, __repr__, __eq__
job = Job(
    title="Python Developer",
    company="TechCorp",
    salary_min=80000,
    salary_max=120000
)

print(job)
# Job(title='Python Developer', company='TechCorp', ...)

print(job.salary_range)
# $80,000 - $120,000
```

### Frozen Dataclasses (Immutable)

```python
@dataclass(frozen=True)
class Point:
    x: float
    y: float

p = Point(1.0, 2.0)
# p.x = 3.0  # Raises FrozenInstanceError
```

---

## 2.4 Pydantic for Data Validation

Pydantic is used extensively in FastAPI for request/response validation:

```python
from pydantic import BaseModel, Field, EmailStr, field_validator
from datetime import datetime

class JobCreate(BaseModel):
    """Schema for creating a new job posting."""

    title: str = Field(..., min_length=3, max_length=100)
    company: str = Field(..., min_length=2)
    description: str = Field(..., min_length=50)
    salary_min: int = Field(..., ge=0)  # >= 0
    salary_max: int = Field(..., ge=0)
    remote: bool = False
    contact_email: EmailStr

    @field_validator("salary_max")
    @classmethod
    def salary_max_must_be_greater(cls, v, info):
        if "salary_min" in info.data and v < info.data["salary_min"]:
            raise ValueError("salary_max must be >= salary_min")
        return v

class JobResponse(BaseModel):
    """Schema for job response with computed fields."""

    id: int
    title: str
    company: str
    description: str
    salary_min: int
    salary_max: int
    remote: bool
    contact_email: EmailStr
    created_at: datetime

    model_config = {
        "from_attributes": True  # Allow creation from ORM objects
    }
```

### Usage Example

```python
# Valid data
job = JobCreate(
    title="Python Developer",
    company="TechCorp",
    description="A" * 50,  # Min 50 chars
    salary_min=80000,
    salary_max=120000,
    contact_email="hr@techcorp.com"
)
print(job.model_dump())  # Convert to dict

# Invalid data - raises ValidationError
try:
    bad_job = JobCreate(
        title="Py",  # Too short
        company="TC",
        description="Short",  # Too short
        salary_min=100000,
        salary_max=80000,  # Less than min!
        contact_email="not-an-email"
    )
except ValidationError as e:
    print(e.errors())
```

---

## 2.5 Modern Package Management

### pyproject.toml (PEP 517/518)

Modern Python projects use `pyproject.toml` instead of `setup.py`:

```toml
# backend/pyproject.toml
[project]
name = "jobboard"
version = "0.1.0"
description = "A modern job board API"
readme = "README.md"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.109.0",
    "uvicorn[standard]>=0.27.0",
    "sqlalchemy[asyncio]>=2.0.25",
    "asyncpg>=0.29.0",
    "alembic>=1.13.1",
    "pydantic>=2.5.3",
    "pydantic-settings>=2.1.0",
    "python-dotenv>=1.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-asyncio>=0.23.0",
    "httpx>=0.26.0",
    "ruff>=0.1.0",
]

[tool.ruff]
line-length = 88
select = ["E", "F", "I"]

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
```

**Install with pip:**
```bash
pip install -e ".[dev]"  # Install with dev dependencies
```

---

## Exercises

Complete these exercises in [exercises/](exercises/):

1. [Exercise 2.1: Type Hints Practice](exercises/exercise-2.1.md)
2. [Exercise 2.2: Async Programming](exercises/exercise-2.2.md)
3. [Exercise 2.3: Pydantic Models](exercises/exercise-2.3.md)

## Assignment

[Assignment 2: Job Board Data Models](assignments/assignment-2.md)

## Quiz

Test your knowledge: [Module 2 Quiz](quiz/quiz-2.md)

---

## Summary

You now understand:
- ✅ Type hints for better code quality
- ✅ Async/await for concurrent programming
- ✅ Modern Python features (walrus, pattern matching, f-strings)
- ✅ Dataclasses and Pydantic for data modeling
- ✅ Modern package management with pyproject.toml

**Next Module:** [FastAPI Fundamentals](../03-fastapi-fundamentals/README.md)
