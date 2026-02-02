# Assignment 1: Project Initialization

## Objective

Set up the complete JobBoard project structure with all necessary configuration files.

## Requirements

Create the following structure and files:

```
jobboard/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   └── config.py
│   ├── tests/
│   │   └── __init__.py
│   ├── requirements.txt
│   └── .env
├── frontend/           (empty for now)
├── docker/
│   └── postgres/
│       └── init.sql
├── docker-compose.yml
├── .env.example
├── .gitignore
├── .vscode/
│   ├── settings.json
│   ├── launch.json
│   └── extensions.json
└── README.md
```

## Deliverables

### 1. Working PostgreSQL Container

- Database should be accessible at `localhost:5432`
- Should have `uuid-ossp` and `pg_trgm` extensions enabled
- Credentials: user=`jobboard`, password=`jobboard_dev_password`, database=`jobboard`

**Verification:**
```bash
# Using Podman
podman exec -it jobboard-db psql -U jobboard -d jobboard -c "SELECT 1 as test;"

# Using Docker
docker exec -it jobboard-db psql -U jobboard -d jobboard -c "SELECT 1 as test;"
```

### 2. FastAPI Application

The API should:
- Run at `http://localhost:8000`
- Have automatic documentation at `/docs`
- Include these endpoints:

| Method | Endpoint | Response |
|--------|----------|----------|
| GET | `/` | `{"message": "Welcome to JobBoard API", "version": "0.1.0"}` |
| GET | `/health` | `{"status": "healthy", "database": "connected"}` |

The `/health` endpoint should actually test the database connection.

**Hint for database health check:**
```python
from sqlalchemy import create_engine, text

def check_db_connection():
    try:
        engine = create_engine(settings.database_url.replace('+asyncpg', ''))
        with engine.connect() as conn:
            conn.execute(text("SELECT 1"))
        return True
    except Exception:
        return False
```

### 3. Configuration

- All sensitive values must come from environment variables
- `.env` file should NOT be committed (verify it's in `.gitignore`)
- `.env.example` should be committed with placeholder values

### 4. Project README

Create a `README.md` in the project root with:
- Project description
- How to start the development environment
- How to run the backend
- List of available API endpoints

## Submission Checklist

- [ ] `podman compose up -d` (or `docker compose up -d`) starts PostgreSQL successfully
- [ ] Can connect to PostgreSQL from host machine
- [ ] `uvicorn app.main:app --reload` starts without errors
- [ ] `http://localhost:8000/docs` shows Swagger UI
- [ ] `http://localhost:8000/health` returns database status
- [ ] `.env` is in `.gitignore`
- [ ] VS Code debugging works (can hit breakpoints)
- [ ] README.md has setup instructions

## Bonus Challenges

1. **Add Redis container** to docker-compose (we'll use it later for caching)
2. **Add a `/ready` endpoint** that checks both database AND returns environment info
3. **Configure VS Code tasks** (`.vscode/tasks.json`) to start/stop containers

## Need Help?

If you get stuck, solutions will be provided separately. Try to complete the assignment on your own first!
