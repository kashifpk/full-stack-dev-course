# Module 2 Quiz: Modern Python

Test your understanding of modern Python features.

---

## Questions

### Q1: Type Hints

What is the correct type hint for a function that accepts a list of strings and returns an optional integer?

A) `def func(items: [str]) -> int | None:`
B) `def func(items: list[str]) -> int | None:`
C) `def func(items: List<str>) -> Optional<int>:`
D) `def func(items: list(str)) -> int?:`

---

### Q2: Union Types

In Python 3.10+, how do you write a union of `int` and `str`?

A) `int || str`
B) `int | str`
C) `Union[int, str]` only
D) `int or str`

---

### Q3: Async/Await

What happens if you call an async function without `await`?

```python
async def fetch():
    return "data"

result = fetch()  # What is result?
```

A) Returns "data"
B) Returns a coroutine object (not executed)
C) Raises an error
D) Returns None

---

### Q4: asyncio.gather

What is the benefit of using `asyncio.gather()`?

A) It runs functions sequentially
B) It runs functions concurrently and waits for all to complete
C) It cancels slow functions
D) It prevents memory leaks

---

### Q5: Walrus Operator

What does this code print?

```python
if (n := len("hello")) > 3:
    print(n)
```

A) `True`
B) `hello`
C) `5`
D) Nothing (condition is false)

---

### Q6: Pattern Matching

What does `case _:` represent in a match statement?

A) Match underscore character
B) Match private variables
C) Wildcard - matches anything (default case)
D) Match empty value

---

### Q7: Dataclasses

What does `@dataclass(frozen=True)` do?

A) Compresses the data
B) Makes instances immutable (can't change attributes)
C) Stores data in cold storage
D) Prevents garbage collection

---

### Q8: Pydantic Validation

When does Pydantic validate data?

A) Only when explicitly calling `.validate()`
B) Only when saving to database
C) Automatically when creating a model instance
D) Only in production mode

---

### Q9: Pydantic field_validator

What is the correct decorator order for a Pydantic validator?

A) `@classmethod` then `@field_validator("field")`
B) `@field_validator("field")` then `@classmethod`
C) `@staticmethod` then `@field_validator("field")`
D) Just `@field_validator("field")`

---

### Q10: Computed Fields

How do you create a computed field in Pydantic v2?

A) Using `@property` only
B) Using `@computed_field` with `@property`
C) Using `default_factory`
D) Using `@validator`

---

### Q11: Async Context Managers

Which is correct for an async context manager?

A) `with async get_db():`
B) `async with get_db():`
C) `await with get_db():`
D) `with await get_db():`

---

### Q12: Type Narrowing

What does this code pattern accomplish?

```python
def process(value: str | None) -> str:
    if value is None:
        return "default"
    return value.upper()  # value is now str
```

A) Runtime type conversion
B) Type narrowing - type checker knows value is str after the check
C) Nothing special
D) Raises error if value is None

---

### Q13: Generic Types

What does `TypeVar("T", bound=BaseModel)` mean?

A) T can be any type
B) T must be BaseModel or a subclass of it
C) T is limited to built-in types
D) T is a constant

---

### Q14: pyproject.toml

What replaced `setup.py` in modern Python packaging?

A) `requirements.txt`
B) `pyproject.toml`
C) `package.json`
D) `Pipfile`

---

### Q15: Async Semaphore

What is `asyncio.Semaphore` used for?

A) Locking resources permanently
B) Limiting concurrent access to a resource
C) Synchronizing threads
D) Measuring async performance

---

*Answers will be provided separately after you complete the quiz.*
