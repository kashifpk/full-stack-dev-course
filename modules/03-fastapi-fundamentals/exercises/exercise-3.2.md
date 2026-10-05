# Exercise 3.2: Dependency Injection

## Objective
Learn to use FastAPI's dependency injection for clean, reusable code.

## Tasks

### Task 1: Pagination Dependency

Create a reusable pagination dependency:

```python
# backend/app/dependencies/pagination.py
from fastapi import Query
from pydantic import BaseModel

class PaginationParams(BaseModel):
    page: int
    per_page: int
    offset: int

def get_pagination(
    page: int = Query(default=1, ge=1, description="Page number"),
    per_page: int = Query(default=20, ge=1, le=100, description="Items per page"),
) -> PaginationParams:
    """
    Return pagination parameters with computed offset.
    """
    # Your implementation
    pass
```

Use it in your companies router:

```python
from app.dependencies.pagination import get_pagination, PaginationParams

@router.get("")
async def list_companies(
    pagination: PaginationParams = Depends(get_pagination)
):
    # Use pagination.offset and pagination.per_page
    pass
```

### Task 2: Common Query Filters

Create a dependency for common filters:

```python
# backend/app/dependencies/filters.py
from fastapi import Query
from datetime import date

class DateRangeFilter:
    def __init__(
        self,
        created_after: date | None = Query(None, description="Filter by creation date"),
        created_before: date | None = Query(None, description="Filter by creation date"),
    ):
        self.created_after = created_after
        self.created_before = created_before

    def apply(self, items: list[dict]) -> list[dict]:
        """Filter items by date range."""
        # Your implementation
        pass
```

### Task 3: Nested Dependencies

Create a dependency that depends on another:

```python
# Simulated auth dependency
async def get_api_key(x_api_key: str = Header(..., description="API Key")):
    """Extract API key from header."""
    return x_api_key

async def get_current_user(api_key: str = Depends(get_api_key)):
    """Validate API key and return user."""
    # Simulate user lookup
    valid_keys = {
        "admin-key": {"id": 1, "name": "Admin", "role": "admin"},
        "user-key": {"id": 2, "name": "User", "role": "user"},
    }
    if api_key not in valid_keys:
        raise HTTPException(status_code=401, detail="Invalid API key")
    return valid_keys[api_key]

# Use in endpoint
@router.post("")
async def create_company(
    company: CompanyCreate,
    current_user: dict = Depends(get_current_user)
):
    if current_user["role"] != "admin":
        raise HTTPException(status_code=403, detail="Admin required")
    # Create company...
```

### Task 4: Dependency with Yield (Cleanup)

Create a dependency that cleans up after the request:

```python
from collections.abc import Generator
import logging

logger = logging.getLogger(__name__)

def get_request_logger() -> Generator[logging.Logger, None, None]:
    """
    Provide a logger and log completion after request.
    """
    request_logger = logging.getLogger("request")
    request_logger.info("Request started")
    try:
        yield request_logger
    finally:
        request_logger.info("Request completed")

@router.get("/{company_id}")
async def get_company(
    company_id: int,
    logger: logging.Logger = Depends(get_request_logger)
):
    logger.info(f"Fetching company {company_id}")
    # ...
```

### Task 5: Class-Based Dependency with Configuration

```python
class RateLimiter:
    """
    Rate limiting dependency.
    In real app, would use Redis for tracking.
    """

    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests: dict[str, list[float]] = {}

    async def __call__(self, request: Request):
        client_ip = request.client.host
        now = time.time()

        # Get requests in window
        if client_ip not in self.requests:
            self.requests[client_ip] = []

        # Remove old requests
        self.requests[client_ip] = [
            t for t in self.requests[client_ip]
            if now - t < self.window_seconds
        ]

        if len(self.requests[client_ip]) >= self.max_requests:
            raise HTTPException(
                status_code=429,
                detail="Too many requests"
            )

        self.requests[client_ip].append(now)

# Create instance with configuration
rate_limiter = RateLimiter(max_requests=10, window_seconds=60)

@router.post("")
async def create_company(
    company: CompanyCreate,
    _: None = Depends(rate_limiter)  # Just need the check
):
    # ...
```

## Verification

- [ ] Pagination works with query parameters
- [ ] Date range filter correctly filters results
- [ ] Auth dependency protects endpoints
- [ ] Logger dependency logs start and end
- [ ] Rate limiter blocks after threshold

## Need Help?

Solutions will be provided separately. Try to complete the tasks on your own first!
