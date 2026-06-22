# Module 2 Quiz Answers

1. **B** - `list[str]` is the generic syntax (Python 3.9+), `int | None` for optional
2. **B** - Python 3.10+ allows `|` for unions; `Union[int, str]` also works
3. **B** - Calling async function without await returns a coroutine object
4. **B** - `gather()` runs coroutines concurrently and waits for all
5. **C** - Walrus assigns 5 to n, condition is true, prints 5
6. **C** - `case _:` is the wildcard/default pattern
7. **B** - `frozen=True` makes instances immutable
8. **C** - Pydantic validates automatically on instantiation
9. **B** - `@field_validator` must come before `@classmethod`
10. **B** - Pydantic v2 uses `@computed_field` decorator with `@property`
11. **B** - `async with` is the correct syntax
12. **B** - Type narrowing lets the type checker infer more specific types
13. **B** - `bound` constrains T to that type or subclasses
14. **B** - `pyproject.toml` is the modern standard (PEP 517/518)
15. **B** - Semaphore limits concurrent access (e.g., max 3 connections)

## Scoring

- 13-15: Excellent! Modern Python master
- 10-12: Good understanding, minor gaps
- 7-9: Review async and Pydantic sections
- Below 7: Re-study the module before continuing
