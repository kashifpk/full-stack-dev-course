# Module 4: PostgreSQL & SQLAlchemy

## Learning Objectives

By the end of this module, you will:
- Use SQLAlchemy 2.0 with async support
- Design database models with relationships
- Manage migrations with Alembic
- Write efficient queries
- Implement the repository pattern

## 4.1 SQLAlchemy 2.0 Overview

SQLAlchemy 2.0 brought major improvements:

| Feature | SQLAlchemy 1.x | SQLAlchemy 2.0 |
|---------|----------------|----------------|
| Query style | `session.query(Model)` | `select(Model)` |
| Async support | Limited/third-party | Native |
| Type hints | Minimal | Full support |
| Session | Implicit | Explicit |

---

## 4.2 Database Setup

### Configuration

```python
# backend/app/database/connection.py
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession
from sqlalchemy.orm import DeclarativeBase
from app.config import get_settings

settings = get_settings()

engine = create_async_engine(
    settings.database_url,
    echo=settings.debug,  # Log SQL in debug mode
    pool_size=5,
    max_overflow=10,
)

async_session_maker = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
)

class Base(DeclarativeBase):
    """Base class for all models."""
    pass
```

### Session Dependency

```python
# backend/app/database/session.py
from collections.abc import AsyncGenerator
from sqlalchemy.ext.asyncio import AsyncSession
from app.database.connection import async_session_maker

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
```

---

## 4.3 Defining Models

### Base Model with Common Fields

```python
# backend/app/models/base.py
from datetime import datetime
from sqlalchemy import DateTime, func
from sqlalchemy.orm import Mapped, mapped_column
from app.database.connection import Base

class TimestampMixin:
    """Mixin for created_at and updated_at fields."""

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False,
    )
```

### User Model

```python
# backend/app/models/user.py
from enum import Enum as PyEnum
from sqlalchemy import String, Boolean, Enum
from sqlalchemy.orm import Mapped, mapped_column, relationship
from app.database.connection import Base
from app.models.base import TimestampMixin

class UserRole(str, PyEnum):
    job_seeker = "job_seeker"
    employer = "employer"
    admin = "admin"

class User(Base, TimestampMixin):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, index=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    full_name: Mapped[str] = mapped_column(String(100))
    role: Mapped[UserRole] = mapped_column(Enum(UserRole), default=UserRole.job_seeker)
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)

    # Relationships
    companies: Mapped[list["Company"]] = relationship(back_populates="owner")
    applications: Mapped[list["Application"]] = relationship(back_populates="user")
```

### Company Model

```python
# backend/app/models/company.py
from sqlalchemy import String, Text, ForeignKey, Enum
from sqlalchemy.orm import Mapped, mapped_column, relationship
from enum import Enum as PyEnum
from app.database.connection import Base
from app.models.base import TimestampMixin

class CompanySize(str, PyEnum):
    startup = "startup"
    small = "small"
    medium = "medium"
    large = "large"
    enterprise = "enterprise"

class Company(Base, TimestampMixin):
    __tablename__ = "companies"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100), index=True)
    description: Mapped[str | None] = mapped_column(Text)
    website: Mapped[str | None] = mapped_column(String(255))
    location: Mapped[str | None] = mapped_column(String(100))
    size: Mapped[CompanySize] = mapped_column(Enum(CompanySize))

    owner_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    owner: Mapped["User"] = relationship(back_populates="companies")

    jobs: Mapped[list["Job"]] = relationship(back_populates="company", cascade="all, delete-orphan")
```

### Job Model

```python
# backend/app/models/job.py
from sqlalchemy import String, Text, Integer, Boolean, ForeignKey, Enum, ARRAY
from sqlalchemy.orm import Mapped, mapped_column, relationship
from enum import Enum as PyEnum
from datetime import datetime
from app.database.connection import Base
from app.models.base import TimestampMixin

class JobType(str, PyEnum):
    full_time = "full_time"
    part_time = "part_time"
    contract = "contract"
    internship = "internship"

class ExperienceLevel(str, PyEnum):
    entry = "entry"
    mid = "mid"
    senior = "senior"
    lead = "lead"
    executive = "executive"

class Job(Base, TimestampMixin):
    __tablename__ = "jobs"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(100), index=True)
    description: Mapped[str] = mapped_column(Text)
    job_type: Mapped[JobType] = mapped_column(Enum(JobType))
    experience_level: Mapped[ExperienceLevel] = mapped_column(Enum(ExperienceLevel))
    salary_min: Mapped[int | None] = mapped_column(Integer)
    salary_max: Mapped[int | None] = mapped_column(Integer)
    is_remote: Mapped[bool] = mapped_column(Boolean, default=False)
    location: Mapped[str | None] = mapped_column(String(100))
    skills: Mapped[list[str]] = mapped_column(ARRAY(String), default=[])
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)
    expires_at: Mapped[datetime | None] = mapped_column(DateTime(timezone=True))

    company_id: Mapped[int] = mapped_column(ForeignKey("companies.id"))
    company: Mapped["Company"] = relationship(back_populates="jobs")

    applications: Mapped[list["Application"]] = relationship(back_populates="job")
```

---

## 4.4 Alembic Migrations

### Initialize Alembic

```bash
cd backend
alembic init alembic
```

### Configure Alembic

```python
# backend/alembic/env.py
import asyncio
from logging.config import fileConfig
from sqlalchemy import pool
from sqlalchemy.ext.asyncio import create_async_engine
from alembic import context
from app.config import get_settings
from app.database.connection import Base
from app.models import user, company, job, application  # Import all models

config = context.config
settings = get_settings()

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata

def run_migrations_offline() -> None:
    context.configure(
        url=settings.database_url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )
    with context.begin_transaction():
        context.run_migrations()

def do_run_migrations(connection):
    context.configure(connection=connection, target_metadata=target_metadata)
    with context.begin_transaction():
        context.run_migrations()

async def run_migrations_online() -> None:
    connectable = create_async_engine(
        settings.database_url,
        poolclass=pool.NullPool,
    )
    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)
    await connectable.dispose()

if context.is_offline_mode():
    run_migrations_offline()
else:
    asyncio.run(run_migrations_online())
```

### Create and Run Migrations

```bash
# Create migration
alembic revision --autogenerate -m "Create initial tables"

# Run migrations
alembic upgrade head

# Rollback one version
alembic downgrade -1

# See migration history
alembic history
```

---

## 4.5 CRUD Operations

### Repository Pattern

```python
# backend/app/repositories/job.py
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.orm import selectinload
from app.models.job import Job
from app.schemas.job import JobCreate, JobUpdate

class JobRepository:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def create(self, job_data: JobCreate) -> Job:
        job = Job(**job_data.model_dump())
        self.db.add(job)
        await self.db.flush()
        await self.db.refresh(job)
        return job

    async def get_by_id(self, job_id: int) -> Job | None:
        result = await self.db.execute(
            select(Job)
            .options(selectinload(Job.company))
            .where(Job.id == job_id)
        )
        return result.scalar_one_or_none()

    async def get_list(
        self,
        *,
        offset: int = 0,
        limit: int = 20,
        search: str | None = None,
        job_type: str | None = None,
        is_remote: bool | None = None,
        company_id: int | None = None,
    ) -> tuple[list[Job], int]:
        query = select(Job).options(selectinload(Job.company))

        # Apply filters
        if search:
            query = query.where(
                Job.title.ilike(f"%{search}%") |
                Job.description.ilike(f"%{search}%")
            )
        if job_type:
            query = query.where(Job.job_type == job_type)
        if is_remote is not None:
            query = query.where(Job.is_remote == is_remote)
        if company_id:
            query = query.where(Job.company_id == company_id)

        # Count total
        count_query = select(func.count()).select_from(query.subquery())
        total = await self.db.scalar(count_query) or 0

        # Apply pagination
        query = query.offset(offset).limit(limit).order_by(Job.created_at.desc())

        result = await self.db.execute(query)
        jobs = list(result.scalars().all())

        return jobs, total

    async def update(self, job: Job, job_data: JobUpdate) -> Job:
        update_data = job_data.model_dump(exclude_unset=True)
        for field, value in update_data.items():
            setattr(job, field, value)
        await self.db.flush()
        await self.db.refresh(job)
        return job

    async def delete(self, job: Job) -> None:
        await self.db.delete(job)
```

### Using Repository in Router

```python
# backend/app/routers/jobs.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from app.database.session import get_db
from app.repositories.job import JobRepository
from app.schemas.job import JobCreate, JobResponse, JobListResponse

router = APIRouter(prefix="/jobs", tags=["jobs"])

@router.post("", response_model=JobResponse, status_code=201)
async def create_job(
    job_data: JobCreate,
    db: AsyncSession = Depends(get_db),
):
    repo = JobRepository(db)
    job = await repo.create(job_data)
    return job

@router.get("", response_model=JobListResponse)
async def list_jobs(
    page: int = 1,
    per_page: int = 20,
    search: str | None = None,
    job_type: str | None = None,
    is_remote: bool | None = None,
    db: AsyncSession = Depends(get_db),
):
    repo = JobRepository(db)
    offset = (page - 1) * per_page

    jobs, total = await repo.get_list(
        offset=offset,
        limit=per_page,
        search=search,
        job_type=job_type,
        is_remote=is_remote,
    )

    return {
        "items": jobs,
        "total": total,
        "page": page,
        "per_page": per_page,
        "pages": (total + per_page - 1) // per_page,
    }
```

---

## 4.6 Advanced Queries

### Joins and Eager Loading

```python
# Eager load to avoid N+1 queries
from sqlalchemy.orm import selectinload, joinedload

# selectinload: Separate query for related items (good for collections)
query = select(Job).options(selectinload(Job.applications))

# joinedload: JOIN in same query (good for single related items)
query = select(Job).options(joinedload(Job.company))

# Combine both
query = select(Job).options(
    joinedload(Job.company),
    selectinload(Job.applications)
)
```

### Aggregations

```python
from sqlalchemy import func, case

# Count jobs by type
query = select(
    Job.job_type,
    func.count(Job.id).label("count")
).group_by(Job.job_type)

# Average salary by experience level
query = select(
    Job.experience_level,
    func.avg(Job.salary_min).label("avg_min"),
    func.avg(Job.salary_max).label("avg_max"),
).group_by(Job.experience_level)

# Conditional counts
query = select(
    func.count(case((Job.is_remote == True, 1))).label("remote_count"),
    func.count(case((Job.is_remote == False, 1))).label("onsite_count"),
)
```

---

## Exercises

1. [Exercise 4.1: Model Relationships](exercises/exercise-4.1.md)
2. [Exercise 4.2: Migrations](exercises/exercise-4.2.md)
3. [Exercise 4.3: Complex Queries](exercises/exercise-4.3.md)

## Assignment

[Assignment 4: Complete Database Layer](assignments/assignment-4.md)

## Quiz

[Module 4 Quiz](quiz/quiz-4.md)

---

## Summary

- ✅ SQLAlchemy 2.0 async setup
- ✅ Model definitions with relationships
- ✅ Alembic migrations
- ✅ Repository pattern for clean data access
- ✅ Complex queries with joins and aggregations

**Next Module:** [Backend Testing](../05-backend-testing/README.md)
