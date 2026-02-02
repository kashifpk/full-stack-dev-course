# Module 1: Development Environment Setup

## Learning Objectives

By the end of this module, you will:
- Have a fully functional containerized development environment
- Understand modern Python tooling (virtual environments, package management)
- Be comfortable with VS Code for Python and JavaScript development
- Have PostgreSQL running in a container

## 1.1 Overview of Modern Development Workflow

### What's Changed in 5 Years?

If you've been away from development, here's what's evolved:

| Then | Now |
|------|-----|
| XAMPP/WAMP for local dev | Containers (Docker/Podman) |
| Manual dependency management | Lock files, reproducible builds |
| FTP deployments | CI/CD pipelines, container orchestration |
| jQuery everywhere | Component-based frameworks (Vue, React) |
| REST APIs | REST still dominant, GraphQL for some use cases |
| Callbacks | async/await everywhere |

### Why Containers?

Containers solve the "works on my machine" problem:
- Consistent environment across development, testing, production
- Easy to spin up databases, caches, services
- No need to install PostgreSQL, Redis, etc. on your machine
- Easy cleanup - just delete the container

---

## 1.2 Installing Prerequisites

### Docker or Podman

We'll use Docker Desktop or Podman. Both work identically for this course.

**Windows:**
1. Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. During installation, enable WSL 2 backend (recommended)
3. Restart your computer when prompted

**Linux (Ubuntu/Debian):**
```bash
# Docker (recommended for Linux)
sudo apt update
sudo apt install docker.io docker-compose-v2
sudo usermod -aG docker $USER
# Log out and back in

# Or Podman (rootless alternative)
sudo apt install podman podman-compose
```

**macOS:**
1. Download and install [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/)

**Verify installation:**
```bash
docker --version
docker compose version
```

### Python 3.11+

**Windows:**
1. Download Python 3.11+ from [python.org](https://www.python.org/downloads/)
2. During installation, **check "Add Python to PATH"**
3. Verify: Open Command Prompt or PowerShell and run:
   ```powershell
   python --version
   ```

**Linux (Ubuntu/Debian):**
```bash
# Check current version
python3 --version

# Install if needed
sudo apt install python3.11 python3.11-venv python3-pip

# Or use pyenv for version management
curl https://pyenv.run | bash
# Follow the instructions to add to your shell
pyenv install 3.11.7
pyenv global 3.11.7
```

**macOS:**
```bash
# Using Homebrew
brew install python@3.11

# Or download from python.org
```

### Node.js 20+ (LTS)

**Windows:**
1. Download Node.js LTS from [nodejs.org](https://nodejs.org/)
2. Run the installer (includes npm)
3. Verify in PowerShell:
   ```powershell
   node --version
   npm --version
   ```

**Linux/macOS (using nvm - recommended):**
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc  # or ~/.zshrc on macOS

# Install and use Node 20
nvm install 20
nvm use 20
nvm alias default 20

# Verify
node --version  # Should show v20.x.x
npm --version
```

### VS Code

Download from https://code.visualstudio.com/ (all platforms)

**Essential Extensions:**
- Python (Microsoft)
- Pylance (Microsoft)
- Vue - Official (Vue)
- TypeScript Vue Plugin (Volar)
- ESLint
- Prettier
- Docker (Microsoft)
- GitLens

---

## 1.3 Project Structure

Create the project structure for our Job Board:

**Linux/macOS:**
```bash
mkdir -p jobboard/{backend,frontend,docker}
cd jobboard
git init
```

**Windows (PowerShell):**
```powershell
mkdir jobboard
cd jobboard
mkdir backend, frontend, docker
git init
```

**Windows (Command Prompt):**
```cmd
mkdir jobboard
cd jobboard
mkdir backend frontend docker
git init
```

**Recommended structure:**
```
jobboard/
├── backend/                 # FastAPI application
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── routers/
│   │   ├── services/
│   │   └── database/
│   ├── tests/
│   ├── alembic/
│   ├── pyproject.toml
│   └── requirements.txt
├── frontend/                # Vue application
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── vite.config.ts
├── docker/                  # Container configurations
│   ├── backend.Dockerfile
│   ├── frontend.Dockerfile
│   └── postgres/
├── docker-compose.yml       # Development environment
├── docker-compose.prod.yml  # Production configuration
├── .env.example
├── .gitignore
└── README.md
```

---

## 1.4 Setting Up PostgreSQL with Containers

Create a `docker-compose.yml` in the project root:

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:16-alpine
    container_name: jobboard-db
    environment:
      POSTGRES_USER: jobboard
      POSTGRES_PASSWORD: jobboard_dev_password
      POSTGRES_DB: jobboard
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./docker/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U jobboard"]
      interval: 5s
      timeout: 5s
      retries: 5

  # We'll add more services as the course progresses

volumes:
  postgres_data:
```

Create the initialization script:

**Linux/macOS:**
```bash
mkdir -p docker/postgres
```

**Windows (PowerShell):**
```powershell
mkdir docker\postgres -Force
```

```sql
-- docker/postgres/init.sql
-- This runs when the container is first created

-- Create additional schemas if needed
CREATE SCHEMA IF NOT EXISTS jobboard;

-- Enable useful extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";  -- For text search

-- Log that initialization is complete
DO $$
BEGIN
    RAISE NOTICE 'Database initialization complete!';
END $$;
```

**Start the database:**
```bash
# Using podman-compose
podman-compose up -d db

# Or docker compose
docker compose up -d db

# Verify it's running
podman-compose ps
# or
docker compose ps

# Check logs
podman-compose logs db
```

**Connect to verify:**
```bash
# Using psql in the container
podman exec -it jobboard-db psql -U jobboard -d jobboard

# Or with a local client
psql -h localhost -U jobboard -d jobboard
```

---

## 1.5 Python Virtual Environment Setup

**Linux/macOS:**
```bash
cd backend

# Create virtual environment
python3.11 -m venv venv

# Activate it
source venv/bin/activate

# Verify
which python  # Should point to venv/bin/python
python --version
```

**Windows (PowerShell):**
```powershell
cd backend

# Create virtual environment
python -m venv venv

# Activate it
.\venv\Scripts\Activate.ps1

# If you get an execution policy error, run:
# Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Verify
python --version
```

**Windows (Command Prompt):**
```cmd
cd backend
python -m venv venv
venv\Scripts\activate.bat
python --version
```

**Create initial requirements:**

```bash
# backend/requirements.txt
fastapi==0.109.0
uvicorn[standard]==0.27.0
sqlalchemy[asyncio]==2.0.25
asyncpg==0.29.0
alembic==1.13.1
pydantic==2.5.3
pydantic-settings==2.1.0
python-dotenv==1.0.0
```

```bash
pip install -r requirements.txt
```

---

## 1.6 Environment Configuration

Create environment files:

```bash
# .env.example (commit this)
DATABASE_URL=postgresql+asyncpg://jobboard:jobboard_dev_password@localhost:5432/jobboard
SECRET_KEY=your-secret-key-change-in-production
DEBUG=true
```

```bash
# .env (DO NOT commit - add to .gitignore)
DATABASE_URL=postgresql+asyncpg://jobboard:jobboard_dev_password@localhost:5432/jobboard
SECRET_KEY=dev-secret-key-12345
DEBUG=true
```

```bash
# .gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
venv/
.venv/
*.egg-info/
dist/
build/

# Environment
.env
.env.local
*.local

# IDE
.vscode/
.idea/
*.swp
*.swo

# Node
node_modules/
dist/

# Database
*.db
*.sqlite

# OS
.DS_Store
Thumbs.db

# Testing
.coverage
htmlcov/
.pytest_cache/

# Docker
docker/postgres/data/
```

---

## 1.7 VS Code Configuration

Create workspace settings:

**Linux/macOS:**
```bash
mkdir -p .vscode
```

**Windows:**
```powershell
mkdir .vscode -Force
```

```json
// .vscode/settings.json
{
    // Python path (VS Code will auto-detect, but you can set explicitly)
    // Linux/macOS: "${workspaceFolder}/backend/venv/bin/python"
    // Windows: "${workspaceFolder}/backend/venv/Scripts/python.exe"
    "python.defaultInterpreterPath": "${workspaceFolder}/backend/venv/bin/python",
    "python.analysis.typeCheckingMode": "basic",
    "python.analysis.autoImportCompletions": true,

    "editor.formatOnSave": true,
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "[python]": {
        "editor.defaultFormatter": "ms-python.python",
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.organizeImports": "explicit"
        }
    },

    "typescript.preferences.importModuleSpecifier": "relative",
    "vue.codeActions.enabled": true,

    "files.exclude": {
        "**/__pycache__": true,
        "**/*.pyc": true,
        "**/node_modules": true
    }
}
```

```json
// .vscode/extensions.json
{
    "recommendations": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "Vue.volar",
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "ms-azuretools.vscode-docker",
        "eamodio.gitlens"
    ]
}
```

---

## 1.8 Verification: Hello World API

Let's verify everything works with a minimal FastAPI app:

```python
# backend/app/main.py
from fastapi import FastAPI

app = FastAPI(
    title="JobBoard API",
    description="A job board application API",
    version="0.1.0"
)

@app.get("/")
async def root():
    return {"message": "Welcome to JobBoard API"}

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

**Run the server:**

**Linux/macOS:**
```bash
cd backend
source venv/bin/activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

**Windows (PowerShell):**
```powershell
cd backend
.\venv\Scripts\Activate.ps1
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

**Test it:**
- Open http://localhost:8000 - should see the welcome message
- Open http://localhost:8000/docs - interactive API documentation (Swagger UI)
- Open http://localhost:8000/redoc - alternative documentation

---

## Exercises

Complete these exercises in [exercises/](exercises/):

1. [Exercise 1.1: Container Basics](exercises/exercise-1.1.md)
2. [Exercise 1.2: Environment Variables](exercises/exercise-1.2.md)
3. [Exercise 1.3: VS Code Debugging](exercises/exercise-1.3.md)

## Assignment

Complete the assignment in [assignments/](assignments/):

[Assignment 1: Project Initialization](assignments/assignment-1.md)

## Quiz

Test your knowledge: [Module 1 Quiz](quiz/quiz-1.md)

---

## Summary

You now have:
- ✅ Docker/Podman installed and working
- ✅ PostgreSQL running in a container
- ✅ Python 3.11+ with virtual environment
- ✅ Node.js 20+ installed
- ✅ VS Code configured for the project
- ✅ Basic FastAPI app running

**Next Module:** [Modern Python Refresher](../02-modern-python/README.md)
