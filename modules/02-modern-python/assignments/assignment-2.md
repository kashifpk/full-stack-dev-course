# Assignment 2: Job Board Data Models

## Objective

Create the Pydantic schemas (data models) for the Job Board application. These will be used by FastAPI for request validation and response serialization.

## Requirements

Create a file `backend/app/schemas/` with the following models:

### 1. User Schemas (`user.py`)

```
UserRole (Enum):
  - job_seeker
  - employer
  - admin

UserBase:
  - email: EmailStr
  - full_name: str (2-100 chars)
  - role: UserRole (default: job_seeker)

UserCreate(UserBase):
  - password: str (min 8 chars, must have letter + number)

UserUpdate:
  - email: EmailStr (optional)
  - full_name: str (optional)
  - password: str (optional)

UserResponse(UserBase):
  - id: int
  - is_active: bool
  - created_at: datetime
  (exclude password)

UserLogin:
  - email: EmailStr
  - password: str
```

### 2. Company Schemas (`company.py`)

```
CompanyBase:
  - name: str (2-100 chars)
  - description: str (optional, max 2000 chars)
  - website: HttpUrl (optional)
  - location: str (optional)
  - size: CompanySize enum (startup, small, medium, large, enterprise)

CompanyCreate(CompanyBase):
  - (inherits all from base)

CompanyUpdate:
  - All fields optional

CompanyResponse(CompanyBase):
  - id: int
  - owner_id: int
  - created_at: datetime
  - job_count: int (computed/optional)
```

### 3. Job Schemas (`job.py`)

```
JobType (Enum):
  - full_time
  - part_time
  - contract
  - internship

ExperienceLevel (Enum):
  - entry
  - mid
  - senior
  - lead
  - executive

JobBase:
  - title: str (5-100 chars)
  - description: str (50-10000 chars)
  - job_type: JobType
  - experience_level: ExperienceLevel
  - salary_min: int (optional, >= 0)
  - salary_max: int (optional, >= salary_min if both provided)
  - is_remote: bool (default: False)
  - location: str (required if not remote)
  - skills: list[str] (1-20 items)

JobCreate(JobBase):
  - company_id: int

JobUpdate:
  - All fields optional

JobResponse(JobBase):
  - id: int
  - company_id: int
  - company: CompanyResponse (nested)
  - is_active: bool
  - created_at: datetime
  - expires_at: datetime (optional)
  - salary_range: str (computed, e.g., "$80,000 - $120,000" or "Not specified")

JobListResponse:
  - items: list[JobResponse]
  - total: int
  - page: int
  - per_page: int
  - pages: int (computed)
```

### 4. Application Schemas (`application.py`)

```
ApplicationStatus (Enum):
  - pending
  - reviewed
  - shortlisted
  - rejected
  - hired

ApplicationCreate:
  - job_id: int
  - cover_letter: str (100-5000 chars)
  - resume_url: HttpUrl (optional)

ApplicationResponse:
  - id: int
  - job_id: int
  - user_id: int
  - status: ApplicationStatus
  - cover_letter: str
  - resume_url: HttpUrl (optional)
  - created_at: datetime
  - updated_at: datetime

ApplicationStatusUpdate:
  - status: ApplicationStatus
```

## Project Structure

```
backend/app/schemas/
├── __init__.py      # Export all schemas
├── user.py
├── company.py
├── job.py
├── application.py
└── common.py        # Shared types, pagination, etc.
```

## Validation Requirements

1. **Cross-field validation:**
   - `salary_max >= salary_min` when both provided
   - `location` required when `is_remote=False`

2. **Custom validators:**
   - Password must contain letter and number
   - Skills list must have unique items
   - Email should be lowercase

3. **Computed fields:**
   - `salary_range` in JobResponse
   - `pages` in JobListResponse

## Example Usage

```python
from app.schemas.job import JobCreate, JobResponse

# Creating a job
job_data = JobCreate(
    title="Senior Python Developer",
    description="A" * 50,  # At least 50 chars
    job_type="full_time",
    experience_level="senior",
    salary_min=100000,
    salary_max=150000,
    is_remote=True,
    skills=["Python", "FastAPI", "PostgreSQL"],
    company_id=1
)

# Response model
job_response = JobResponse(
    id=1,
    **job_data.model_dump(),
    company={"id": 1, "name": "TechCorp", ...},
    is_active=True,
    created_at=datetime.now()
)

print(job_response.salary_range)  # "$100,000 - $150,000"
```

## Deliverables

- [ ] All schema files created with proper type hints
- [ ] Cross-field validation working (salary, location)
- [ ] Custom validators implemented
- [ ] Computed fields working
- [ ] All schemas exported from `__init__.py`
- [ ] Test file demonstrating valid and invalid inputs

## Bonus Challenges

1. Add a `SearchParams` schema for job search with filters
2. Create a `BulkJobCreate` schema for creating multiple jobs at once
3. Add `model_config` with JSON schema examples for documentation

## Hints

<details>
<summary>Cross-field validation pattern</summary>

```python
from pydantic import model_validator

class JobBase(BaseModel):
    salary_min: int | None = None
    salary_max: int | None = None
    is_remote: bool = False
    location: str | None = None

    @model_validator(mode="after")
    def validate_salary_and_location(self):
        if self.salary_min and self.salary_max:
            if self.salary_max < self.salary_min:
                raise ValueError("salary_max must be >= salary_min")

        if not self.is_remote and not self.location:
            raise ValueError("location required for non-remote jobs")

        return self
```

</details>

<details>
<summary>Computed field pattern</summary>

```python
from pydantic import computed_field

class JobResponse(JobBase):
    @computed_field
    @property
    def salary_range(self) -> str:
        if self.salary_min and self.salary_max:
            return f"${self.salary_min:,} - ${self.salary_max:,}"
        elif self.salary_min:
            return f"From ${self.salary_min:,}"
        elif self.salary_max:
            return f"Up to ${self.salary_max:,}"
        return "Not specified"
```

</details>
