# Exercise 3.2 Solutions

## Click to reveal pagination solution

```python
from fastapi import Query
from pydantic import BaseModel

class PaginationParams(BaseModel):
    page: int
    per_page: int
    offset: int

    model_config = {"frozen": True}  # Immutable

def get_pagination(
    page: int = Query(default=1, ge=1, description="Page number"),
    per_page: int = Query(default=20, ge=1, le=100, description="Items per page"),
) -> PaginationParams:
    return PaginationParams(
        page=page,
        per_page=per_page,
        offset=(page - 1) * per_page
    )
```