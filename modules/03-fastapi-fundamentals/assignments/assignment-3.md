# Assignment 3: Job Listings API

## Objective

Build the Jobs API with full CRUD operations, building on top of your Companies API from the exercises. Jobs should belong to companies.

## Requirements

### API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/jobs` | Create a job posting |
| GET | `/jobs` | List jobs with filters and pagination |
| GET | `/jobs/{job_id}` | Get a single job |
| PUT | `/jobs/{job_id}` | Update a job |
| DELETE | `/jobs/{job_id}` | Delete a job |
| GET | `/companies/{company_id}/jobs` | List jobs for a company |

### Job Schema

Use the schemas you created in Assignment 2:

```python
class JobCreate(BaseModel):
    title: str = Field(..., min_length=5, max_length=100)
    description: str = Field(..., min_length=50, max_length=10000)
    company_id: int
    job_type: JobType  # full_time, part_time, contract, internship
    experience_level: ExperienceLevel  # entry, mid, senior, lead
    salary_min: int | None = Field(None, ge=0)
    salary_max: int | None = Field(None, ge=0)
    is_remote: bool = False
    location: str | None = None
    skills: list[str] = Field(..., min_length=1, max_length=20)

class JobResponse(BaseModel):
    id: int
    # ... all fields
    company: CompanyResponse  # Nested company info
    salary_range: str  # Computed field
    created_at: datetime
```

### Filtering & Search

The `GET /jobs` endpoint should support:

```
GET /jobs?page=1&per_page=20
GET /jobs?search=python
GET /jobs?job_type=full_time
GET /jobs?experience_level=senior
GET /jobs?is_remote=true
GET /jobs?salary_min=80000
GET /jobs?company_id=5
GET /jobs?skills=python&skills=fastapi
```

Multiple filters should combine with AND logic.

### Business Rules

1. **Company validation:** Cannot create job for non-existent company
2. **Salary validation:** `salary_max >= salary_min` when both provided
3. **Location validation:** `location` required when `is_remote=false`
4. **Skills validation:** Skills list must have unique items

### Response Format

List endpoint should return paginated response:

```json
{
  "items": [...],
  "total": 150,
  "page": 1,
  "per_page": 20,
  "pages": 8
}
```

## Project Structure

```
backend/app/
├── routers/
│   ├── jobs.py         # Job endpoints
│   └── companies.py    # Company endpoints
├── schemas/
│   ├── job.py          # Job schemas
│   ├── company.py      # Company schemas
│   └── common.py       # Pagination, etc.
├── dependencies/
│   └── pagination.py   # Pagination dependency
├── exceptions.py       # Custom exceptions
├── exception_handlers.py
└── main.py
```

## Deliverables

### 1. Jobs Router (`/jobs`)

```python
# POST /jobs - Create job
# Validate company exists
# Validate salary range
# Validate location if not remote
# Return 201 with JobResponse

# GET /jobs - List with filters
# Support all filter parameters
# Return paginated JobListResponse

# GET /jobs/{job_id}
# Return JobResponse with nested company
# Return 404 if not found

# PUT /jobs/{job_id}
# Partial update (only provided fields)
# Re-validate business rules
# Return updated JobResponse

# DELETE /jobs/{job_id}
# Return 204 No Content
```

### 2. Company Jobs Route

```python
# GET /companies/{company_id}/jobs
# Return jobs for specific company
# Support pagination
# Return 404 if company not found
```

### 3. Error Handling

All errors should follow this format:

```json
{
  "error_code": "not_found",
  "message": "Job with ID 123 not found",
  "details": []
}
```

### 4. Documentation

- All endpoints documented in Swagger UI
- Request/response examples provided
- Error responses documented

## Testing Checklist

```bash
# Create a company first
curl -X POST http://localhost:8000/companies \
  -H "Content-Type: application/json" \
  -d '{"name": "TechCorp", "size": "medium"}'
# Note the company ID (e.g., 1)

# Create a job
curl -X POST http://localhost:8000/jobs \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Senior Python Developer",
    "description": "We are looking for an experienced Python developer...(50+ chars)",
    "company_id": 1,
    "job_type": "full_time",
    "experience_level": "senior",
    "salary_min": 100000,
    "salary_max": 150000,
    "is_remote": true,
    "skills": ["Python", "FastAPI", "PostgreSQL"]
  }'

# List jobs with filters
curl "http://localhost:8000/jobs?is_remote=true&experience_level=senior"

# Get single job
curl http://localhost:8000/jobs/1

# Update job
curl -X PUT http://localhost:8000/jobs/1 \
  -H "Content-Type: application/json" \
  -d '{"salary_max": 160000}'

# Get company jobs
curl http://localhost:8000/companies/1/jobs

# Delete job
curl -X DELETE http://localhost:8000/jobs/1
```

## Grading Criteria

| Criteria | Points |
|----------|--------|
| All CRUD endpoints working | 30 |
| Filtering and search working | 20 |
| Pagination implemented correctly | 15 |
| Business rule validation | 15 |
| Error handling consistent | 10 |
| Code organization | 10 |
| **Total** | **100** |

## Bonus Challenges

1. **Sorting:** Add `sort_by` and `order` query parameters
2. **Bulk operations:** Add `POST /jobs/bulk` for creating multiple jobs
3. **Job statistics:** Add `GET /jobs/stats` returning counts by type, level, etc.
4. **Search highlighting:** Return which part of title/description matched search

## Hints

<details>
<summary>Filtering pattern</summary>

```python
@router.get("")
async def list_jobs(
    search: str | None = None,
    job_type: JobType | None = None,
    experience_level: ExperienceLevel | None = None,
    is_remote: bool | None = None,
    salary_min: int | None = None,
    company_id: int | None = None,
    skills: list[str] = Query(default=[]),
    pagination: PaginationParams = Depends(get_pagination),
):
    results = list(jobs_db.values())

    if search:
        search_lower = search.lower()
        results = [
            j for j in results
            if search_lower in j["title"].lower()
            or search_lower in j["description"].lower()
        ]

    if job_type:
        results = [j for j in results if j["job_type"] == job_type]

    if is_remote is not None:
        results = [j for j in results if j["is_remote"] == is_remote]

    if salary_min:
        results = [
            j for j in results
            if j.get("salary_min") and j["salary_min"] >= salary_min
        ]

    if skills:
        skills_set = set(s.lower() for s in skills)
        results = [
            j for j in results
            if skills_set.issubset(set(s.lower() for s in j["skills"]))
        ]

    total = len(results)
    results = results[pagination.offset : pagination.offset + pagination.per_page]

    return {
        "items": results,
        "total": total,
        "page": pagination.page,
        "per_page": pagination.per_page,
        "pages": (total + pagination.per_page - 1) // pagination.per_page
    }
```

</details>
