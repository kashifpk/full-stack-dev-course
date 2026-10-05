# Module 3: FastAPI Fundamentals

## Learning Objectives

By the end of this module, you will:
- Build RESTful APIs with FastAPI
- Use Pydantic models for request/response validation
- Implement dependency injection
- Handle errors properly
- Organize code with routers
- Understand automatic API documentation

## 3.1 Why FastAPI?

FastAPI has become the go-to Python web framework because:

| Feature | Benefit |
|---------|---------|
| Type hints | Automatic validation and documentation |
| Async native | High performance for I/O-bound operations |
| Pydantic integration | Data validation out of the box |
| Auto documentation | Swagger UI and ReDoc generated automatically |
| Standards-based | OpenAPI, JSON Schema compliant |

Coming from PHP, think of FastAPI as Laravel's elegance meets static typing.

---

## 3.2 Request Handling Basics

### Path Parameters

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/jobs/{job_id}")
async def get_job(job_id: int):
    """
    Path parameter automatically:
    - Validated as integer
    - Converted from string to int
    - Returns 422 if not a valid int
    """
    return {"job_id": job_id}

# Multiple path parameters
@app.get("/companies/{company_id}/jobs/{job_id}")
async def get_company_job(company_id: int, job_id: int):
    return {"company_id": company_id, "job_id": job_id}
```

### Query Parameters

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/jobs")
async def list_jobs(
    # Required query parameter
    company_id: int,
    # Optional with default
    page: int = 1,
    # Optional, can be None
    search: str | None = None,
    # With validation
    per_page: int = Query(default=20, ge=1, le=100),
    # List parameter (?skills=python&skills=fastapi)
    skills: list[str] = Query(default=[]),
):
    return {
        "company_id": company_id,
        "page": page,
        "per_page": per_page,
        "search": search,
        "skills": skills
    }
```

### Request Body

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class JobCreate(BaseModel):
    title: str = Field(..., min_length=5, max_length=100)
    description: str = Field(..., min_length=50)
    salary_min: int = Field(..., ge=0)
    salary_max: int = Field(..., ge=0)

    class Config:
        json_schema_extra = {
            "example": {
                "title": "Python Developer",
                "description": "A" * 50,
                "salary_min": 80000,
                "salary_max": 120000
            }
        }

@app.post("/jobs", status_code=201)
async def create_job(job: JobCreate):
    """
    Request body automatically:
    - Parsed from JSON
    - Validated against schema
    - Returns 422 with details if invalid
    """
    return {"id": 1, **job.model_dump()}
```

### Combining Parameters

```python
@app.put("/jobs/{job_id}")
async def update_job(
    job_id: int,                    # Path parameter
    job: JobCreate,                 # Request body
    notify: bool = False,           # Query parameter
):
    return {"job_id": job_id, "notify": notify, **job.model_dump()}
```

---

## 3.3 Response Models

Control what gets returned to clients:

```python
from fastapi import FastAPI
from pydantic import BaseModel
from datetime import datetime

class JobCreate(BaseModel):
    title: str
    description: str
    salary_min: int
    salary_max: int

class JobResponse(BaseModel):
    id: int
    title: str
    description: str
    salary_min: int
    salary_max: int
    created_at: datetime

    model_config = {"from_attributes": True}

class JobListResponse(BaseModel):
    items: list[JobResponse]
    total: int
    page: int
    pages: int

app = FastAPI()

@app.post("/jobs", response_model=JobResponse, status_code=201)
async def create_job(job: JobCreate):
    # Even if we return extra fields, only JobResponse fields are sent
    return {
        "id": 1,
        **job.model_dump(),
        "created_at": datetime.now(),
        "internal_notes": "This won't be in response"  # Filtered out!
    }

@app.get("/jobs", response_model=JobListResponse)
async def list_jobs(page: int = 1):
    return {
        "items": [...],
        "total": 100,
        "page": page,
        "pages": 10
    }
```

### Response Model Options

```python
# Exclude unset fields
@app.get("/jobs/{job_id}", response_model=JobResponse, response_model_exclude_unset=True)
async def get_job(job_id: int):
    ...

# Exclude specific fields
@app.get("/jobs/{job_id}", response_model=JobResponse, response_model_exclude={"internal_id"})
async def get_job(job_id: int):
    ...
```

---

## 3.4 Dependency Injection

FastAPI's DI system is powerful and clean:

### Basic Dependencies

```python
from fastapi import FastAPI, Depends, Query

app = FastAPI()

# Simple dependency
def get_pagination(
    page: int = Query(default=1, ge=1),
    per_page: int = Query(default=20, ge=1, le=100),
):
    return {"page": page, "per_page": per_page, "offset": (page - 1) * per_page}

@app.get("/jobs")
async def list_jobs(pagination: dict = Depends(get_pagination)):
    return {
        "items": [],
        "page": pagination["page"],
        "per_page": pagination["per_page"]
    }
```

### Class-Based Dependencies

```python
from dataclasses import dataclass

@dataclass
class Pagination:
    page: int = 1
    per_page: int = 20

    @property
    def offset(self) -> int:
        return (self.page - 1) * self.per_page

# FastAPI can instantiate classes directly
@app.get("/jobs")
async def list_jobs(pagination: Pagination = Depends()):
    return {"offset": pagination.offset}
```

### Database Session Dependency

```python
from collections.abc import AsyncGenerator
from sqlalchemy.ext.asyncio import AsyncSession

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

@app.get("/jobs")
async def list_jobs(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Job))
    return result.scalars().all()
```

### Nested Dependencies

```python
from fastapi import Depends, HTTPException, Header

async def get_token(authorization: str = Header(...)):
    if not authorization.startswith("Bearer "):
        raise HTTPException(401, "Invalid token format")
    return authorization.split(" ")[1]

async def get_current_user(token: str = Depends(get_token)):
    user = await verify_token(token)
    if not user:
        raise HTTPException(401, "Invalid token")
    return user

@app.get("/me")
async def get_me(user: User = Depends(get_current_user)):
    return user
```

---

## 3.5 Error Handling

### HTTP Exceptions

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/jobs/{job_id}")
async def get_job(job_id: int):
    job = await find_job(job_id)
    if not job:
        raise HTTPException(
            status_code=404,
            detail=f"Job {job_id} not found"
        )
    return job
```

### Custom Exception Handlers

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

class JobNotFoundError(Exception):
    def __init__(self, job_id: int):
        self.job_id = job_id

app = FastAPI()

@app.exception_handler(JobNotFoundError)
async def job_not_found_handler(request: Request, exc: JobNotFoundError):
    return JSONResponse(
        status_code=404,
        content={
            "error": "not_found",
            "message": f"Job {exc.job_id} not found",
            "job_id": exc.job_id
        }
    )

@app.get("/jobs/{job_id}")
async def get_job(job_id: int):
    job = await find_job(job_id)
    if not job:
        raise JobNotFoundError(job_id)
    return job
```

### Validation Error Customization

```python
from fastapi import FastAPI, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    errors = []
    for error in exc.errors():
        errors.append({
            "field": ".".join(str(x) for x in error["loc"][1:]),  # Skip 'body'
            "message": error["msg"],
            "type": error["type"]
        })
    return JSONResponse(
        status_code=422,
        content={"errors": errors}
    )
```

---

## 3.6 Routers and Code Organization

Split your application into modules:

```
backend/app/
├── main.py
├── routers/
│   ├── __init__.py
│   ├── jobs.py
│   ├── companies.py
│   └── users.py
├── schemas/
│   └── ...
└── services/
    └── ...
```

### Creating a Router

```python
# app/routers/jobs.py
from fastapi import APIRouter, Depends, HTTPException
from app.schemas.job import JobCreate, JobResponse, JobListResponse

router = APIRouter(
    prefix="/jobs",
    tags=["jobs"],  # Groups in documentation
)

@router.get("", response_model=JobListResponse)
async def list_jobs(page: int = 1):
    ...

@router.post("", response_model=JobResponse, status_code=201)
async def create_job(job: JobCreate):
    ...

@router.get("/{job_id}", response_model=JobResponse)
async def get_job(job_id: int):
    ...

@router.put("/{job_id}", response_model=JobResponse)
async def update_job(job_id: int, job: JobCreate):
    ...

@router.delete("/{job_id}", status_code=204)
async def delete_job(job_id: int):
    ...
```

### Including Routers

```python
# app/main.py
from fastapi import FastAPI
from app.routers import jobs, companies, users

app = FastAPI(
    title="JobBoard API",
    description="A modern job board application",
    version="1.0.0",
)

app.include_router(jobs.router)
app.include_router(companies.router)
app.include_router(users.router, prefix="/api/v1")
```

---

## 3.7 API Documentation

FastAPI generates documentation automatically:

- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`
- **OpenAPI JSON**: `http://localhost:8000/openapi.json`

### Enhancing Documentation

```python
from fastapi import FastAPI, Path, Query

app = FastAPI(
    title="JobBoard API",
    description="""
    ## Features
    * Create and manage job postings
    * Search and filter jobs
    * Apply to jobs

    ## Authentication
    Use Bearer token in Authorization header.
    """,
    version="1.0.0",
    contact={
        "name": "API Support",
        "email": "support@jobboard.com"
    }
)

@app.get(
    "/jobs/{job_id}",
    summary="Get a job by ID",
    description="Retrieve detailed information about a specific job posting.",
    response_description="The job details",
    responses={
        404: {"description": "Job not found"},
        200: {
            "description": "Successful response",
            "content": {
                "application/json": {
                    "example": {"id": 1, "title": "Python Developer"}
                }
            }
        }
    }
)
async def get_job(
    job_id: int = Path(..., title="Job ID", description="The ID of the job to retrieve", ge=1)
):
    """
    Get a job posting by its ID.

    - **job_id**: The unique identifier of the job
    """
    ...
```

---

## 3.8 Middleware and CORS

### CORS (for frontend access)

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],  # Vue dev server
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Custom Middleware

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

---

## Exercises

1. [Exercise 3.1: Building CRUD Endpoints](exercises/exercise-3.1.md)
2. [Exercise 3.2: Dependency Injection](exercises/exercise-3.2.md)
3. [Exercise 3.3: Error Handling](exercises/exercise-3.3.md)

## Assignment

[Assignment 3: Job Listings API](assignments/assignment-3.md)

## Quiz

[Module 3 Quiz](quiz/quiz-3.md)

---

## Summary

You now can:
- ✅ Create REST endpoints with path, query, and body parameters
- ✅ Validate requests and format responses with Pydantic
- ✅ Use dependency injection for clean, testable code
- ✅ Handle errors gracefully
- ✅ Organize code with routers
- ✅ Configure CORS and middleware

**Next Module:** [PostgreSQL & SQLAlchemy](../04-database/README.md)
