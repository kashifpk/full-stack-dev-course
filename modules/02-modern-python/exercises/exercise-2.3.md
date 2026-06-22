# Exercise 2.3: Pydantic Models

## Objective
Master Pydantic for data validation and serialization.

## Tasks

### Task 1: Basic Model with Validation

Create a Pydantic model for a user registration:

```python
from pydantic import BaseModel, Field, EmailStr, field_validator

class UserRegistration(BaseModel):
    """
    Requirements:
    - username: 3-20 characters, alphanumeric and underscores only
    - email: valid email format
    - password: minimum 8 characters, must contain number and letter
    - age: optional, must be 18+ if provided
    """
    pass  # Your code here

# Test cases - these should work:
valid_user = UserRegistration(
    username="john_doe",
    email="john@example.com",
    password="secure123"
)

# These should raise ValidationError:
# UserRegistration(username="ab", ...)           # too short
# UserRegistration(username="john doe", ...)     # contains space
# UserRegistration(email="not-email", ...)       # invalid email
# UserRegistration(password="short", ...)        # too short
# UserRegistration(password="nonnumber", ...)    # no number
# UserRegistration(..., age=16)                  # under 18
```

### Task 2: Nested Models

Create models for a job application with nested data:

```python
from pydantic import BaseModel, HttpUrl
from datetime import date

class Education(BaseModel):
    """Degree, institution, graduation year"""
    pass

class WorkExperience(BaseModel):
    """Company, title, start_date, end_date (optional for current job)"""
    pass

class Resume(BaseModel):
    """
    - name: str
    - email: EmailStr
    - phone: str (validate format: XXX-XXX-XXXX)
    - linkedin: HttpUrl (optional)
    - education: list of Education (at least 1)
    - experience: list of WorkExperience (can be empty)
    - skills: list of str (at least 3)
    """
    pass

# Should work:
resume = Resume(
    name="Jane Smith",
    email="jane@example.com",
    phone="555-123-4567",
    education=[
        Education(degree="BS Computer Science", institution="MIT", year=2020)
    ],
    experience=[
        WorkExperience(
            company="Google",
            title="Software Engineer",
            start_date=date(2020, 6, 1),
            end_date=date(2023, 5, 31)
        )
    ],
    skills=["Python", "JavaScript", "SQL"]
)
```

### Task 3: Model Serialization

Create models that demonstrate different serialization options:

```python
from pydantic import BaseModel, Field, computed_field
from datetime import datetime

class JobPostingDB(BaseModel):
    """Model representing database record."""
    id: int
    title: str
    company_name: str
    salary_min: int
    salary_max: int
    is_remote: bool
    created_at: datetime
    updated_at: datetime

    model_config = {"from_attributes": True}

class JobPostingResponse(BaseModel):
    """
    API response model with:
    - Renamed fields (company_name -> company)
    - Computed field (salary_range: "$80,000 - $120,000")
    - Excluded fields (created_at, updated_at hidden)
    - Custom serialization
    """
    pass  # Your code here

# Usage:
db_job = JobPostingDB(
    id=1,
    title="Python Developer",
    company_name="TechCorp",
    salary_min=80000,
    salary_max=120000,
    is_remote=True,
    created_at=datetime.now(),
    updated_at=datetime.now()
)

response = JobPostingResponse.model_validate(db_job)
print(response.model_dump())
# Should output something like:
# {"id": 1, "title": "Python Developer", "company": "TechCorp",
#  "salary_range": "$80,000 - $120,000", "remote": true}
```

### Task 4: Custom Validators and Serializers

```python
from pydantic import BaseModel, field_validator, field_serializer
from datetime import datetime

class JobApplication(BaseModel):
    """
    - applicant_email: EmailStr
    - cover_letter: str (min 100 chars, max 5000 chars)
    - salary_expectation: int (must be positive, serialized as "$XX,XXX")
    - available_from: date (must be in the future)
    - applied_at: datetime (auto-set to now, serialized as ISO string)
    """
    pass  # Your code here

# Usage:
app = JobApplication(
    applicant_email="jane@example.com",
    cover_letter="A" * 100,
    salary_expectation=95000,
    available_from=date.today() + timedelta(days=14)
)

print(app.model_dump())
# salary_expectation should be "$95,000"
```

## Verification

All test cases should pass without ValidationError for valid data, and raise ValidationError with clear messages for invalid data.

## Need Help?

Solutions will be provided separately. Try to complete the tasks on your own first!
