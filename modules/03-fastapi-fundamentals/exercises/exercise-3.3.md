# Exercise 3.3: Error Handling

## Objective
Implement consistent error handling across your API.

## Tasks

### Task 1: Create Custom Exception Classes

```python
# backend/app/exceptions.py
from fastapi import HTTPException

class AppException(Exception):
    """Base exception for application errors."""

    def __init__(self, message: str, error_code: str):
        self.message = message
        self.error_code = error_code
        super().__init__(message)

class NotFoundError(AppException):
    """Resource not found."""

    def __init__(self, resource: str, resource_id: int | str):
        self.resource = resource
        self.resource_id = resource_id
        super().__init__(
            message=f"{resource} with ID {resource_id} not found",
            error_code="not_found"
        )

class ValidationError(AppException):
    """Business logic validation error."""

    def __init__(self, message: str, field: str | None = None):
        self.field = field
        super().__init__(message=message, error_code="validation_error")

class AuthorizationError(AppException):
    """User not authorized for this action."""

    def __init__(self, message: str = "Not authorized"):
        super().__init__(message=message, error_code="unauthorized")

class ConflictError(AppException):
    """Resource conflict (e.g., duplicate)."""

    def __init__(self, message: str):
        super().__init__(message=message, error_code="conflict")
```

### Task 2: Create Exception Handlers

```python
# backend/app/exception_handlers.py
from fastapi import Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from app.exceptions import AppException, NotFoundError, AuthorizationError

async def app_exception_handler(request: Request, exc: AppException) -> JSONResponse:
    """Handle custom application exceptions."""
    status_codes = {
        "not_found": 404,
        "validation_error": 422,
        "unauthorized": 401,
        "forbidden": 403,
        "conflict": 409,
    }

    # Your implementation: return JSONResponse with error details
    pass

async def validation_exception_handler(
    request: Request, exc: RequestValidationError
) -> JSONResponse:
    """
    Handle Pydantic validation errors with a cleaner format.

    Transform from:
    [{"loc": ["body", "title"], "msg": "...", "type": "..."}]

    To:
    {"errors": [{"field": "title", "message": "..."}]}
    """
    pass

async def generic_exception_handler(request: Request, exc: Exception) -> JSONResponse:
    """
    Handle unexpected errors.
    Log the full error, return generic message to client.
    """
    pass
```

### Task 3: Register Exception Handlers

```python
# backend/app/main.py
from fastapi import FastAPI
from fastapi.exceptions import RequestValidationError
from app.exceptions import AppException
from app.exception_handlers import (
    app_exception_handler,
    validation_exception_handler,
    generic_exception_handler,
)

app = FastAPI()

app.add_exception_handler(AppException, app_exception_handler)
app.add_exception_handler(RequestValidationError, validation_exception_handler)
app.add_exception_handler(Exception, generic_exception_handler)
```

### Task 4: Use Exceptions in Router

Update your companies router:

```python
from app.exceptions import NotFoundError, ConflictError, ValidationError

@router.get("/{company_id}")
async def get_company(company_id: int):
    if company_id not in companies_db:
        raise NotFoundError("Company", company_id)
    return companies_db[company_id]

@router.post("")
async def create_company(company: CompanyCreate):
    # Check for duplicate name
    for existing in companies_db.values():
        if existing["name"].lower() == company.name.lower():
            raise ConflictError(f"Company '{company.name}' already exists")
    # Create company...

@router.delete("/{company_id}")
async def delete_company(company_id: int):
    company = companies_db.get(company_id)
    if not company:
        raise NotFoundError("Company", company_id)

    # Business rule: can't delete company with active jobs
    if company.get("job_count", 0) > 0:
        raise ValidationError(
            "Cannot delete company with active job postings",
            field="company_id"
        )

    del companies_db[company_id]
```

### Task 5: Create Error Response Schema

Document your error format:

```python
# backend/app/schemas/error.py
from pydantic import BaseModel

class ErrorDetail(BaseModel):
    field: str | None = None
    message: str

class ErrorResponse(BaseModel):
    error_code: str
    message: str
    details: list[ErrorDetail] = []

    model_config = {
        "json_schema_extra": {
            "example": {
                "error_code": "not_found",
                "message": "Company with ID 123 not found",
                "details": []
            }
        }
    }
```

Use in endpoint documentation:

```python
@router.get(
    "/{company_id}",
    responses={
        404: {"model": ErrorResponse, "description": "Company not found"},
        422: {"model": ErrorResponse, "description": "Validation error"},
    }
)
async def get_company(company_id: int):
    ...
```

## Verification

Test error responses:

```bash
# Not found
curl http://localhost:8000/companies/999
# Should return: {"error_code": "not_found", "message": "Company with ID 999 not found"}

# Validation error (bad request body)
curl -X POST http://localhost:8000/companies \
  -H "Content-Type: application/json" \
  -d '{"name": "x"}'  # Too short
# Should return formatted validation errors

# Conflict
# Create same company twice
```

## Need Help?

Solutions will be provided separately. Try to complete the tasks on your own first!
