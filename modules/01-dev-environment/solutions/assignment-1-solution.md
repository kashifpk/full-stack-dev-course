# Assignment 1 Solution

## Example main.py

```python
from fastapi import FastAPI
from sqlalchemy import create_engine, text
from app.config import get_settings

settings = get_settings()

app = FastAPI(
    title="JobBoard API",
    description="A modern job board application",
    version="0.1.0",
)


def check_database():
    """Check if database is reachable."""
    try:
        # Use sync engine for simple health check
        sync_url = settings.database_url.replace("+asyncpg", "")
        engine = create_engine(sync_url)
        with engine.connect() as conn:
            conn.execute(text("SELECT 1"))
        return True
    except Exception:
        return False


@app.get("/")
async def root():
    return {
        "message": "Welcome to JobBoard API",
        "version": "0.1.0"
    }


@app.get("/health")
async def health():
    db_connected = check_database()
    return {
        "status": "healthy" if db_connected else "degraded",
        "database": "connected" if db_connected else "disconnected"
    }
```

## Example config.py

```python
from pydantic_settings import BaseSettings
from functools import lru_cache


class Settings(BaseSettings):
    database_url: str
    secret_key: str
    debug: bool = False

    class Config:
        env_file = ".env"


@lru_cache
def get_settings() -> Settings:
    return Settings()
```
