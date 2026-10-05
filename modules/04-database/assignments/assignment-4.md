# Assignment 4: Complete Database Layer

## Objective

Replace the in-memory storage with PostgreSQL, implementing all models, repositories, and migrations.

## Requirements

### 1. Database Models

Create all models in `backend/app/models/`:

```
models/
├── __init__.py      # Export all models
├── base.py          # TimestampMixin
├── user.py          # User model
├── company.py       # Company model
├── job.py           # Job model
└── application.py   # Application model
```

**Model Relationships:**
- User has many Companies (employer)
- User has many Applications (job seeker)
- Company belongs to User (owner)
- Company has many Jobs
- Job belongs to Company
- Job has many Applications
- Application belongs to User and Job

### 2. Migrations

Create and run migrations:

```bash
alembic revision --autogenerate -m "Create users table"
alembic revision --autogenerate -m "Create companies table"
alembic revision --autogenerate -m "Create jobs table"
alembic revision --autogenerate -m "Create applications table"
alembic upgrade head
```

### 3. Repositories

Create repositories for each model:

```python
# backend/app/repositories/user.py
class UserRepository:
    async def create(self, user_data: UserCreate) -> User
    async def get_by_id(self, user_id: int) -> User | None
    async def get_by_email(self, email: str) -> User | None
    async def update(self, user: User, user_data: UserUpdate) -> User
    async def delete(self, user: User) -> None

# backend/app/repositories/company.py
class CompanyRepository:
    async def create(self, company_data: CompanyCreate, owner_id: int) -> Company
    async def get_by_id(self, company_id: int) -> Company | None
    async def get_list(self, *, offset, limit, owner_id=None, search=None) -> tuple[list, int]
    async def update(self, company: Company, data: CompanyUpdate) -> Company
    async def delete(self, company: Company) -> None

# backend/app/repositories/job.py
class JobRepository:
    async def create(self, job_data: JobCreate) -> Job
    async def get_by_id(self, job_id: int) -> Job | None
    async def get_list(self, *, offset, limit, filters...) -> tuple[list, int]
    async def update(self, job: Job, data: JobUpdate) -> Job
    async def delete(self, job: Job) -> None
    async def get_by_company(self, company_id: int, offset, limit) -> tuple[list, int]

# backend/app/repositories/application.py
class ApplicationRepository:
    async def create(self, application_data: ApplicationCreate, user_id: int) -> Application
    async def get_by_id(self, application_id: int) -> Application | None
    async def get_by_user(self, user_id: int, offset, limit) -> tuple[list, int]
    async def get_by_job(self, job_id: int, offset, limit) -> tuple[list, int]
    async def update_status(self, application: Application, status: str) -> Application
```

### 4. Update Routers

Update all routers to use repositories with database sessions:

```python
@router.get("/{job_id}")
async def get_job(
    job_id: int,
    db: AsyncSession = Depends(get_db),
):
    repo = JobRepository(db)
    job = await repo.get_by_id(job_id)
    if not job:
        raise NotFoundError("Job", job_id)
    return job
```

### 5. Seed Data Script

Create a script to populate initial data:

```python
# backend/scripts/seed.py
import asyncio
from app.database.connection import async_session_maker
from app.models import User, Company, Job

async def seed():
    async with async_session_maker() as session:
        # Create admin user
        admin = User(
            email="admin@jobboard.com",
            hashed_password="...",
            full_name="Admin User",
            role="admin"
        )
        session.add(admin)

        # Create sample companies
        # Create sample jobs
        # ...

        await session.commit()

if __name__ == "__main__":
    asyncio.run(seed())
```

## Deliverables

- [ ] All models created with proper relationships
- [ ] Migrations run successfully
- [ ] All repositories implemented
- [ ] Routers updated to use database
- [ ] Seed script creates test data
- [ ] All API endpoints work with real database

## Testing

```bash
# Verify database connection
python -c "from app.database.connection import engine; print('OK')"

# Run migrations
alembic upgrade head

# Seed data
python scripts/seed.py

# Test endpoints
curl http://localhost:8000/jobs
curl http://localhost:8000/companies
```

## Bonus

1. Add database indexes for frequently queried fields
2. Implement soft delete (is_deleted flag) instead of hard delete
3. Add full-text search using PostgreSQL's `tsvector`
