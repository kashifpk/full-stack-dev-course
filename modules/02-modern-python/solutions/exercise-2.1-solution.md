# Exercise 2.1 Solutions

## Task 1 Solution

```python
def calculate_salary_range(min_salary: int, max_salary: int) -> int | None:
    if max_salary < min_salary:
        return None
    return max_salary - min_salary

def format_job_title(title: str, company: str, remote: bool) -> str:
    location = "Remote" if remote else "On-site"
    return f"{title} at {company} ({location})"

def filter_jobs_by_salary(
    jobs: list[dict[str, any]],
    min_salary: int
) -> list[dict[str, any]]:
    return [job for job in jobs if job["salary"] >= min_salary]
```

## Task 2 Solution

```python
from typing import TypedDict, NotRequired
from collections.abc import Callable

class JobDict(TypedDict):
    id: int
    title: str
    company: str
    salary: int
    remote: NotRequired[bool]

def process_jobs(
    jobs: list[JobDict],
    filter_fn: Callable[[JobDict], bool],
    transform_fn: Callable[[JobDict], str]
) -> list[str]:
    filtered = [job for job in jobs if filter_fn(job)]
    return [transform_fn(job) for job in filtered]
```

## Task 3 Solution

```python
from typing import TypeVar, Generic, Protocol

class HasId(Protocol):
    id: int

T = TypeVar("T", bound=HasId)

class InMemoryRepository(Generic[T]):
    def __init__(self) -> None:
        self._storage: dict[int, T] = {}

    def add(self, entity: T) -> T:
        self._storage[entity.id] = entity
        return entity

    def get(self, id: int) -> T | None:
        return self._storage.get(id)

    def get_all(self) -> list[T]:
        return list(self._storage.values())

    def delete(self, id: int) -> bool:
        if id in self._storage:
            del self._storage[id]
            return True
        return False
```
