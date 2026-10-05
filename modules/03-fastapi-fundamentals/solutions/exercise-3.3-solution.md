# Exercise 3.3 Solutions

## Click to reveal exception handler solution

```python
# backend/app/exception_handlers.py
import logging
from fastapi import Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from app.exceptions import AppException

logger = logging.getLogger(__name__)

async def app_exception_handler(request: Request, exc: AppException) -> JSONResponse:
    status_codes = {
        "not_found": 404,
        "validation_error": 422,
        "unauthorized": 401,
        "forbidden": 403,
        "conflict": 409,
    }

    return JSONResponse(
        status_code=status_codes.get(exc.error_code, 400),
        content={
            "error_code": exc.error_code,
            "message": exc.message,
            "details": []
        }
    )

async def validation_exception_handler(
    request: Request, exc: RequestValidationError
) -> JSONResponse:
    details = []
    for error in exc.errors():
        # Skip "body" in location path
        field_path = ".".join(str(x) for x in error["loc"] if x != "body")
        details.append({
            "field": field_path or None,
            "message": error["msg"]
        })

    return JSONResponse(
        status_code=422,
        content={
            "error_code": "validation_error",
            "message": "Request validation failed",
            "details": details
        }
    )

async def generic_exception_handler(request: Request, exc: Exception) -> JSONResponse:
    logger.exception(f"Unhandled exception: {exc}")

    return JSONResponse(
        status_code=500,
        content={
            "error_code": "internal_error",
            "message": "An unexpected error occurred",
            "details": []
        }
    )
```