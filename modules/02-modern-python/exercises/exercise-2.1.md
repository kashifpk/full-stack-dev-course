# Exercise 2.1: Type Hints Practice

## Objective
Practice writing properly typed Python code.

## Tasks

### Task 1: Basic Function Typing

Add type hints to these functions:

```python
# Before
def calculate_salary_range(min_salary, max_salary):
    if max_salary < min_salary:
        return None
    return max_salary - min_salary

def format_job_title(title, company, remote):
    location = "Remote" if remote else "On-site"
    return f"{title} at {company} ({location})"

def filter_jobs_by_salary(jobs, min_salary):
    return [job for job in jobs if job["salary"] >= min_salary]
```

### Task 2: Complex Types

Type these functions using `TypedDict`, `Callable`, etc.:

```python
from typing import TypedDict

# Define a JobDict type with these fields:
# - id: int
# - title: str
# - company: str
# - salary: int
# - remote: bool (optional, defaults present)

class JobDict(TypedDict):
    # Your code here
    pass

def process_jobs(
    jobs,           # List of JobDict
    filter_fn,      # Function that takes JobDict and returns bool
    transform_fn    # Function that takes JobDict and returns str
):
    # Returns list of strings
    filtered = [job for job in jobs if filter_fn(job)]
    return [transform_fn(job) for job in filtered]
```

### Task 3: Generic Repository

Create a typed generic repository:

```python
from typing import TypeVar, Generic, Protocol

# Define a protocol for entities with an 'id' attribute
class HasId(Protocol):
    id: int

T = TypeVar("T", bound=HasId)

class InMemoryRepository(Generic[T]):
    """Generic repository for storing entities."""

    def __init__(self):
        # Initialize storage
        pass

    def add(self, entity: T) -> T:
        """Add entity and return it."""
        pass

    def get(self, id: int) -> T | None:
        """Get entity by ID or None if not found."""
        pass

    def get_all(self) -> list[T]:
        """Return all entities."""
        pass

    def delete(self, id: int) -> bool:
        """Delete entity by ID. Return True if deleted."""
        pass
```

## Verification

Run mypy to check your types:

```bash
pip install mypy
mypy your_file.py
```

All checks should pass with no errors.

## Need Help?

Solutions will be provided separately. Try to complete the tasks on your own first!
