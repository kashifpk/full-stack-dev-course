# Exercise 1.2: Environment Variables

## Objective
Understand environment variable management in Python applications.

## Tasks

### Task 1: Create a Configuration Module

Create `backend/app/config.py`:

```python
from pydantic_settings import BaseSettings
from functools import lru_cache


class Settings(BaseSettings):
    """Application settings loaded from environment variables."""

    # Database
    database_url: str

    # Application
    debug: bool = False
    secret_key: str

    # API
    api_prefix: str = "/api/v1"

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"
        case_sensitive = False


@lru_cache
def get_settings() -> Settings:
    """Cache settings to avoid reading .env on every request."""
    return Settings()
```

### Task 2: Test Environment Loading

Create a test script `backend/test_config.py`:

```python
from app.config import get_settings

settings = get_settings()

print(f"Database URL: {settings.database_url}")
print(f"Debug mode: {settings.debug}")
print(f"API Prefix: {settings.api_prefix}")
```

Run it:
```bash
cd backend
source venv/bin/activate
python test_config.py
```

### Task 3: Use Settings in FastAPI

Update `backend/app/main.py`:

```python
from fastapi import FastAPI
from app.config import get_settings

settings = get_settings()

app = FastAPI(
    title="JobBoard API",
    description="A job board application API",
    version="0.1.0",
    debug=settings.debug
)

@app.get("/")
async def root():
    return {"message": "Welcome to JobBoard API"}

@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "debug": settings.debug
    }

@app.get("/config")
async def show_config():
    """Only available in debug mode - shows non-sensitive config."""
    if not settings.debug:
        return {"error": "Not available in production"}
    return {
        "api_prefix": settings.api_prefix,
        "debug": settings.debug
    }
```

### Task 4: Environment Overrides

Test environment variable precedence:

```bash
# .env file has DEBUG=true
# Override with shell environment variable
DEBUG=false uvicorn app.main:app --reload

# Visit http://localhost:8000/health
# Should show debug: false
```

## Verification

- [ ] Settings load from `.env` file
- [ ] Shell environment variables override `.env` values
- [ ] Settings are cached (check with print statements)
- [ ] FastAPI uses settings correctly

## Key Concepts

1. **pydantic-settings** validates environment variables with type checking
2. **@lru_cache** prevents re-reading `.env` on every request
3. **Environment precedence:** Shell env > .env file > defaults
4. **Never expose secrets** in API responses
