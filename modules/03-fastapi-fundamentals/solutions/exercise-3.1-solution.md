# Exercise 3.1 Solutions

## Click to reveal solution

```python
# backend/app/routers/companies.py
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel, Field, HttpUrl
from datetime import datetime
from enum import Enum

router = APIRouter(prefix="/companies", tags=["companies"])

class CompanySize(str, Enum):
    startup = "startup"
    small = "small"
    medium = "medium"
    large = "large"
    enterprise = "enterprise"

class CompanyCreate(BaseModel):
    name: str = Field(..., min_length=2, max_length=100)
    description: str | None = Field(None, max_length=2000)
    website: HttpUrl | None = None
    size: CompanySize

class CompanyUpdate(BaseModel):
    name: str | None = Field(None, min_length=2, max_length=100)
    description: str | None = Field(None, max_length=2000)
    website: HttpUrl | None = None
    size: CompanySize | None = None

class CompanyResponse(BaseModel):
    id: int
    name: str
    description: str | None
    website: HttpUrl | None
    size: CompanySize
    created_at: datetime

# In-memory storage
companies_db: dict[int, dict] = {}
next_id = 1

@router.post("", response_model=CompanyResponse, status_code=201)
async def create_company(company: CompanyCreate):
    global next_id
    company_data = {
        "id": next_id,
        **company.model_dump(),
        "created_at": datetime.now()
    }
    companies_db[next_id] = company_data
    next_id += 1
    return company_data

@router.get("", response_model=list[CompanyResponse])
async def list_companies(
    size: CompanySize | None = None,
    search: str | None = None,
):
    results = list(companies_db.values())

    if size:
        results = [c for c in results if c["size"] == size]

    if search:
        search_lower = search.lower()
        results = [c for c in results if search_lower in c["name"].lower()]

    return results

@router.get("/{company_id}", response_model=CompanyResponse)
async def get_company(company_id: int):
    if company_id not in companies_db:
        raise HTTPException(status_code=404, detail="Company not found")
    return companies_db[company_id]

@router.put("/{company_id}", response_model=CompanyResponse)
async def update_company(company_id: int, company: CompanyUpdate):
    if company_id not in companies_db:
        raise HTTPException(status_code=404, detail="Company not found")

    existing = companies_db[company_id]
    update_data = company.model_dump(exclude_unset=True)
    existing.update(update_data)

    return existing

@router.delete("/{company_id}", status_code=204)
async def delete_company(company_id: int):
    if company_id not in companies_db:
        raise HTTPException(status_code=404, detail="Company not found")
    del companies_db[company_id]
```