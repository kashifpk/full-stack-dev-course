# Module 5: Backend Testing

## Learning Objectives

- Write unit and integration tests with pytest
- Test FastAPI endpoints with TestClient
- Use fixtures for test data and database setup
- Mock external dependencies
- Achieve good test coverage

## 5.1 Testing Setup

### Install Dependencies

```bash
pip install pytest pytest-asyncio httpx pytest-cov
```

### Configure pytest

```toml
# backend/pyproject.toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
addopts = "-v --tb=short"
filterwarnings = ["ignore::DeprecationWarning"]
```

### Project Structure

```
backend/tests/
├── __init__.py
├── conftest.py          # Shared fixtures
├── unit/
│   ├── __init__.py
│   ├── test_schemas.py
│   └── test_services.py
├── integration/
│   ├── __init__.py
│   ├── test_jobs_api.py
│   ├── test_companies_api.py
│   └── test_auth_api.py
└── factories.py         # Test data factories
```

---

## 5.2 Fixtures

### Database Fixtures

```python
# backend/tests/conftest.py
import pytest
from httpx import AsyncClient, ASGITransport
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker
from app.main import app
from app.database.connection import Base, get_db

# Test database URL
TEST_DATABASE_URL = "postgresql+asyncpg://jobboard:password@localhost:5432/jobboard_test"

@pytest.fixture(scope="session")
def event_loop():
    """Create event loop for async tests."""
    import asyncio
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()

@pytest.fixture(scope="session")
async def test_engine():
    """Create test database engine."""
    engine = create_async_engine(TEST_DATABASE_URL, echo=False)
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield engine
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
    await engine.dispose()

@pytest.fixture
async def db_session(test_engine):
    """Create a fresh database session for each test."""
    async_session = async_sessionmaker(test_engine, expire_on_commit=False)
    async with async_session() as session:
        yield session
        await session.rollback()

@pytest.fixture
async def client(db_session):
    """Create test client with overridden database."""
    async def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db

    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as ac:
        yield ac

    app.dependency_overrides.clear()
```

### Test Data Factories

```python
# backend/tests/factories.py
from datetime import datetime
from app.models import User, Company, Job

class UserFactory:
    @staticmethod
    def create(
        email: str = "test@example.com",
        full_name: str = "Test User",
        role: str = "job_seeker",
        **kwargs
    ) -> User:
        return User(
            email=email,
            full_name=full_name,
            hashed_password="hashed_password",
            role=role,
            **kwargs
        )

class CompanyFactory:
    @staticmethod
    def create(
        name: str = "Test Company",
        owner_id: int = 1,
        **kwargs
    ) -> Company:
        return Company(
            name=name,
            size="medium",
            owner_id=owner_id,
            **kwargs
        )

class JobFactory:
    @staticmethod
    def create(
        title: str = "Test Job",
        company_id: int = 1,
        **kwargs
    ) -> Job:
        return Job(
            title=title,
            description="A" * 50,
            job_type="full_time",
            experience_level="mid",
            company_id=company_id,
            skills=["Python"],
            **kwargs
        )
```

---

## 5.3 Unit Tests

### Testing Schemas

```python
# backend/tests/unit/test_schemas.py
import pytest
from pydantic import ValidationError
from app.schemas.job import JobCreate

class TestJobCreate:
    def test_valid_job(self):
        job = JobCreate(
            title="Python Developer",
            description="A" * 50,
            company_id=1,
            job_type="full_time",
            experience_level="senior",
            skills=["Python", "FastAPI"],
            is_remote=True,
        )
        assert job.title == "Python Developer"
        assert job.is_remote is True

    def test_title_too_short(self):
        with pytest.raises(ValidationError) as exc_info:
            JobCreate(
                title="Dev",  # Too short
                description="A" * 50,
                company_id=1,
                job_type="full_time",
                experience_level="mid",
                skills=["Python"],
            )
        assert "title" in str(exc_info.value)

    def test_salary_validation(self):
        with pytest.raises(ValidationError):
            JobCreate(
                title="Python Developer",
                description="A" * 50,
                company_id=1,
                job_type="full_time",
                experience_level="mid",
                skills=["Python"],
                salary_min=100000,
                salary_max=80000,  # Less than min
            )

    def test_location_required_when_not_remote(self):
        with pytest.raises(ValidationError):
            JobCreate(
                title="Python Developer",
                description="A" * 50,
                company_id=1,
                job_type="full_time",
                experience_level="mid",
                skills=["Python"],
                is_remote=False,
                location=None,  # Required when not remote
            )
```

---

## 5.4 Integration Tests

### Testing API Endpoints

```python
# backend/tests/integration/test_jobs_api.py
import pytest
from httpx import AsyncClient
from tests.factories import CompanyFactory, JobFactory

class TestJobsAPI:
    @pytest.fixture
    async def company(self, db_session):
        """Create a test company."""
        company = CompanyFactory.create()
        db_session.add(company)
        await db_session.commit()
        await db_session.refresh(company)
        return company

    async def test_create_job(self, client: AsyncClient, company):
        response = await client.post(
            "/jobs",
            json={
                "title": "Senior Python Developer",
                "description": "A" * 50,
                "company_id": company.id,
                "job_type": "full_time",
                "experience_level": "senior",
                "skills": ["Python", "FastAPI"],
                "is_remote": True,
            },
        )
        assert response.status_code == 201
        data = response.json()
        assert data["title"] == "Senior Python Developer"
        assert data["id"] is not None

    async def test_create_job_invalid_company(self, client: AsyncClient):
        response = await client.post(
            "/jobs",
            json={
                "title": "Senior Python Developer",
                "description": "A" * 50,
                "company_id": 99999,  # Non-existent
                "job_type": "full_time",
                "experience_level": "senior",
                "skills": ["Python"],
                "is_remote": True,
            },
        )
        assert response.status_code == 404

    async def test_list_jobs(self, client: AsyncClient, db_session, company):
        # Create test jobs
        for i in range(5):
            job = JobFactory.create(title=f"Job {i}", company_id=company.id)
            db_session.add(job)
        await db_session.commit()

        response = await client.get("/jobs")
        assert response.status_code == 200
        data = response.json()
        assert data["total"] == 5
        assert len(data["items"]) == 5

    async def test_list_jobs_with_filters(self, client: AsyncClient, db_session, company):
        # Create remote and onsite jobs
        remote_job = JobFactory.create(title="Remote Job", company_id=company.id, is_remote=True)
        onsite_job = JobFactory.create(title="Onsite Job", company_id=company.id, is_remote=False, location="NYC")
        db_session.add_all([remote_job, onsite_job])
        await db_session.commit()

        # Filter by remote
        response = await client.get("/jobs?is_remote=true")
        assert response.status_code == 200
        data = response.json()
        assert all(job["is_remote"] for job in data["items"])

    async def test_get_job(self, client: AsyncClient, db_session, company):
        job = JobFactory.create(company_id=company.id)
        db_session.add(job)
        await db_session.commit()
        await db_session.refresh(job)

        response = await client.get(f"/jobs/{job.id}")
        assert response.status_code == 200
        assert response.json()["id"] == job.id

    async def test_get_job_not_found(self, client: AsyncClient):
        response = await client.get("/jobs/99999")
        assert response.status_code == 404

    async def test_update_job(self, client: AsyncClient, db_session, company):
        job = JobFactory.create(company_id=company.id, salary_min=80000)
        db_session.add(job)
        await db_session.commit()
        await db_session.refresh(job)

        response = await client.put(
            f"/jobs/{job.id}",
            json={"salary_max": 120000},
        )
        assert response.status_code == 200
        assert response.json()["salary_max"] == 120000

    async def test_delete_job(self, client: AsyncClient, db_session, company):
        job = JobFactory.create(company_id=company.id)
        db_session.add(job)
        await db_session.commit()
        await db_session.refresh(job)

        response = await client.delete(f"/jobs/{job.id}")
        assert response.status_code == 204

        # Verify deleted
        response = await client.get(f"/jobs/{job.id}")
        assert response.status_code == 404
```

---

## 5.5 Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=app --cov-report=html

# Run specific test file
pytest tests/integration/test_jobs_api.py

# Run specific test
pytest tests/integration/test_jobs_api.py::TestJobsAPI::test_create_job

# Run with verbose output
pytest -v -s
```

---

## Exercises

1. [Exercise 5.1: Schema Tests](exercises/exercise-5.1.md)
2. [Exercise 5.2: API Tests](exercises/exercise-5.2.md)
3. [Exercise 5.3: Mocking](exercises/exercise-5.3.md)

## Assignment

[Assignment 5: Comprehensive Test Suite](assignments/assignment-5.md)

## Quiz

[Module 5 Quiz](quiz/quiz-5.md)

---

## Summary

- ✅ pytest setup with async support
- ✅ Database fixtures with transaction rollback
- ✅ Test factories for data creation
- ✅ Unit tests for schemas
- ✅ Integration tests for API endpoints

**Next Module:** [JavaScript & TypeScript](../06-js-typescript/README.md)
