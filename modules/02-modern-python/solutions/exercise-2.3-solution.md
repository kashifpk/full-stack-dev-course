# Exercise 2.3 Solutions

## Task 1 Solution

```python
import re
from pydantic import BaseModel, Field, EmailStr, field_validator

class UserRegistration(BaseModel):
    username: str = Field(..., min_length=3, max_length=20)
    email: EmailStr
    password: str = Field(..., min_length=8)
    age: int | None = None

    @field_validator("username")
    @classmethod
    def username_alphanumeric(cls, v: str) -> str:
        if not re.match(r"^[a-zA-Z0-9_]+$", v):
            raise ValueError("must be alphanumeric with underscores only")
        return v

    @field_validator("password")
    @classmethod
    def password_strength(cls, v: str) -> str:
        if not re.search(r"[a-zA-Z]", v):
            raise ValueError("must contain at least one letter")
        if not re.search(r"\d", v):
            raise ValueError("must contain at least one number")
        return v

    @field_validator("age")
    @classmethod
    def age_adult(cls, v: int | None) -> int | None:
        if v is not None and v < 18:
            raise ValueError("must be 18 or older")
        return v
```

## Task 2 Solution

```python
import re
from pydantic import BaseModel, Field, EmailStr, HttpUrl, field_validator
from datetime import date

class Education(BaseModel):
    degree: str
    institution: str
    year: int = Field(..., ge=1950, le=2030)

class WorkExperience(BaseModel):
    company: str
    title: str
    start_date: date
    end_date: date | None = None

class Resume(BaseModel):
    name: str
    email: EmailStr
    phone: str
    linkedin: HttpUrl | None = None
    education: list[Education] = Field(..., min_length=1)
    experience: list[WorkExperience] = []
    skills: list[str] = Field(..., min_length=3)

    @field_validator("phone")
    @classmethod
    def validate_phone(cls, v: str) -> str:
        if not re.match(r"^\d{3}-\d{3}-\d{4}$", v):
            raise ValueError("must be in format XXX-XXX-XXXX")
        return v
```

## Task 3 Solution

```python
from pydantic import BaseModel, Field, computed_field
from datetime import datetime

class JobPostingResponse(BaseModel):
    id: int
    title: str
    company: str = Field(validation_alias="company_name")
    salary_min: int = Field(exclude=True)
    salary_max: int = Field(exclude=True)
    remote: bool = Field(validation_alias="is_remote")

    @computed_field
    @property
    def salary_range(self) -> str:
        return f"${self.salary_min:,} - ${self.salary_max:,}"

    model_config = {"from_attributes": True}
```

## Task 4 Solution

```python
from pydantic import BaseModel, Field, EmailStr, field_validator, field_serializer
from datetime import date, datetime, timedelta

class JobApplication(BaseModel):
    applicant_email: EmailStr
    cover_letter: str = Field(..., min_length=100, max_length=5000)
    salary_expectation: int = Field(..., gt=0)
    available_from: date
    applied_at: datetime = Field(default_factory=datetime.now)

    @field_validator("available_from")
    @classmethod
    def must_be_future(cls, v: date) -> date:
        if v <= date.today():
            raise ValueError("must be a future date")
        return v

    @field_serializer("salary_expectation")
    def serialize_salary(self, v: int) -> str:
        return f"${v:,}"

    @field_serializer("applied_at")
    def serialize_datetime(self, v: datetime) -> str:
        return v.isoformat()
```
