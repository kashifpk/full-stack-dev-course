# Exercise 3.1: Building CRUD Endpoints

## Objective
Build a complete CRUD API for companies (in-memory storage for now).

## Setup

Create `backend/app/routers/companies.py`:

```python
from fastapi import APIRouter

router = APIRouter(prefix="/companies", tags=["companies"])

# In-memory storage
companies_db: dict[int, dict] = {}
next_id = 1
```

## Tasks

### Task 1: Create Schema

Create `backend/app/schemas/company.py`:

```python
from pydantic import BaseModel, Field, HttpUrl
from datetime import datetime
from enum import Enum

class CompanySize(str, Enum):
    startup = "startup"
    small = "small"
    medium = "medium"
    large = "large"
    enterprise = "enterprise"

class CompanyCreate(BaseModel):
    # Add fields: name, description (optional), website (optional), size
    pass

class CompanyUpdate(BaseModel):
    # All fields optional
    pass

class CompanyResponse(BaseModel):
    # All fields plus id and created_at
    pass
```

### Task 2: Implement POST /companies

Create a new company:

```python
@router.post("", response_model=CompanyResponse, status_code=201)
async def create_company(company: CompanyCreate):
    """
    - Generate a new ID
    - Store in companies_db
    - Return the created company
    """
    pass
```

**Test:**
```bash
curl -X POST http://localhost:8000/companies \
  -H "Content-Type: application/json" \
  -d '{"name": "TechCorp", "size": "medium"}'
```

### Task 3: Implement GET /companies

List all companies with optional filtering:

```python
@router.get("", response_model=list[CompanyResponse])
async def list_companies(
    size: CompanySize | None = None,
    search: str | None = None,
):
    """
    - Return all companies
    - Filter by size if provided
    - Filter by name (case-insensitive contains) if search provided
    """
    pass
```

### Task 4: Implement GET /companies/{company_id}

Get a single company:

```python
@router.get("/{company_id}", response_model=CompanyResponse)
async def get_company(company_id: int):
    """
    - Return company if found
    - Raise HTTPException(404) if not found
    """
    pass
```

### Task 5: Implement PUT /companies/{company_id}

Update a company:

```python
@router.put("/{company_id}", response_model=CompanyResponse)
async def update_company(company_id: int, company: CompanyUpdate):
    """
    - Update only provided fields
    - Return updated company
    - Raise HTTPException(404) if not found
    """
    pass
```

### Task 6: Implement DELETE /companies/{company_id}

Delete a company:

```python
@router.delete("/{company_id}", status_code=204)
async def delete_company(company_id: int):
    """
    - Delete company
    - Return 204 No Content
    - Raise HTTPException(404) if not found
    """
    pass
```

### Task 7: Register Router

Update `backend/app/main.py`:

```python
from app.routers import companies

app.include_router(companies.router)
```

## Verification

Test all endpoints:

```bash
# Create
curl -X POST http://localhost:8000/companies \
  -H "Content-Type: application/json" \
  -d '{"name": "TechCorp", "size": "medium"}'

# List
curl http://localhost:8000/companies

# Get one
curl http://localhost:8000/companies/1

# Update
curl -X PUT http://localhost:8000/companies/1 \
  -H "Content-Type: application/json" \
  -d '{"description": "A great company"}'

# Delete
curl -X DELETE http://localhost:8000/companies/1

# Verify deleted
curl http://localhost:8000/companies/1  # Should return 404
```

Also verify at http://localhost:8000/docs

## Need Help?

Solutions will be provided separately. Try to complete the tasks on your own first!
