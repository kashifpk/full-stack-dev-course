# Module 14: DevOps & Deployment

## Learning Objectives

- Containerize the application with Docker/Podman
- Set up development and production environments
- Deploy to a cloud server
- Configure basic CI/CD

## 14.1 Docker/Podman Setup

### Backend Dockerfile

```dockerfile
# docker/backend.Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY backend/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY backend/app ./app
COPY backend/alembic ./alembic
COPY backend/alembic.ini .

# Create non-root user
RUN useradd --create-home appuser
USER appuser

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Frontend Dockerfile

```dockerfile
# docker/frontend.Dockerfile

# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

COPY frontend/package*.json ./
RUN bun install --frozen-lockfile

COPY frontend/ .
RUN bun run build

# Production stage
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY docker/nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Nginx Configuration

```nginx
# docker/nginx.conf
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    # Vue Router history mode support
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API proxy (if running on same server)
    location /api {
        proxy_pass http://backend:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
}
```

---

## 14.2 Docker Compose

### Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:16-alpine
    container_name: jobboard-db
    environment:
      POSTGRES_USER: ${DB_USER:-jobboard}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-jobboard_dev}
      POSTGRES_DB: ${DB_NAME:-jobboard}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U jobboard"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: jobboard-redis
    ports:
      - "6379:6379"

  backend:
    build:
      context: .
      dockerfile: docker/backend.Dockerfile
    container_name: jobboard-backend
    environment:
      DATABASE_URL: postgresql+asyncpg://${DB_USER:-jobboard}:${DB_PASSWORD:-jobboard_dev}@db:5432/${DB_NAME:-jobboard}
      SECRET_KEY: ${SECRET_KEY:-dev-secret-key}
      DEBUG: "true"
    ports:
      - "8000:8000"
    volumes:
      - ./backend/app:/app/app:ro  # Hot reload in dev
      - ./uploads:/app/uploads
    depends_on:
      db:
        condition: service_healthy
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  frontend:
    build:
      context: .
      dockerfile: docker/frontend.dev.Dockerfile
    container_name: jobboard-frontend
    environment:
      VITE_API_URL: http://localhost:8000
    ports:
      - "5173:5173"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    command: bun run dev -- --host 0.0.0.0

volumes:
  postgres_data:
```

### Production

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  db:
    image: postgres:16-alpine
    restart: always
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: .
      dockerfile: docker/backend.Dockerfile
    restart: always
    environment:
      DATABASE_URL: postgresql+asyncpg://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      SECRET_KEY: ${SECRET_KEY}
      DEBUG: "false"
    depends_on:
      db:
        condition: service_healthy
    expose:
      - "8000"

  frontend:
    build:
      context: .
      dockerfile: docker/frontend.Dockerfile
      args:
        VITE_API_URL: ${API_URL}
    restart: always
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - backend
    volumes:
      - ./ssl:/etc/nginx/ssl:ro  # SSL certificates

volumes:
  postgres_data:
```

---

## 14.3 Environment Configuration

### .env Files

```bash
# .env.example (commit this)
# Database
DB_USER=jobboard
DB_PASSWORD=change_me
DB_NAME=jobboard

# Application
SECRET_KEY=change_me_in_production
DEBUG=false

# URLs
API_URL=https://api.yourdomain.com
FRONTEND_URL=https://yourdomain.com

# Email
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=
SMTP_PASSWORD=
```

---

## 14.4 Database Migrations in Production

```bash
# Run migrations before starting the app
docker compose -f docker-compose.prod.yml exec backend alembic upgrade head
```

Or add to entrypoint:

```dockerfile
# docker/backend.Dockerfile
COPY docker/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
#!/bin/bash
# docker/entrypoint.sh

# Run migrations
alembic upgrade head

# Execute CMD
exec "$@"
```

---

## 14.5 Simple Deployment

### Deploy to a VPS

```bash
# On your local machine
# 1. Build and push images (or use docker-compose build on server)
docker compose -f docker-compose.prod.yml build

# 2. Copy files to server
scp -r docker-compose.prod.yml docker/ .env.prod user@server:/app/jobboard/

# On the server
# 3. Start services
cd /app/jobboard
docker compose -f docker-compose.prod.yml up -d

# 4. Run migrations
docker compose -f docker-compose.prod.yml exec backend alembic upgrade head

# 5. Check status
docker compose -f docker-compose.prod.yml ps
docker compose -f docker-compose.prod.yml logs -f
```

### Basic GitHub Actions CI

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  backend-test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        working-directory: backend
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-asyncio httpx

      - name: Run tests
        working-directory: backend
        env:
          DATABASE_URL: postgresql+asyncpg://test:test@localhost:5432/test
          SECRET_KEY: test-secret
        run: pytest

  frontend-test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        working-directory: frontend
        run: bun install --frozen-lockfile

      - name: Run tests
        working-directory: frontend
        run: bun test

      - name: Build
        working-directory: frontend
        run: bun run build
```

---

## 14.6 Useful Commands

```bash
# Development
docker compose up -d              # Start all services
docker compose down               # Stop all services
docker compose logs -f backend    # Follow backend logs
docker compose exec backend bash  # Shell into backend container

# Production
docker compose -f docker-compose.prod.yml up -d
docker compose -f docker-compose.prod.yml down
docker compose -f docker-compose.prod.yml logs -f

# Database
docker compose exec db psql -U jobboard -d jobboard
docker compose exec backend alembic upgrade head
docker compose exec backend alembic downgrade -1

# Cleanup
docker system prune -a            # Remove unused images
docker volume prune               # Remove unused volumes
```

---

## Exercises

1. [Exercise 14.1: Dockerize Application](exercises/exercise-14.1.md)
2. [Exercise 14.2: Production Config](exercises/exercise-14.2.md)
3. [Exercise 14.3: CI Pipeline](exercises/exercise-14.3.md)

## Assignment

[Assignment 14: Deploy Job Board](assignments/assignment-14.md)

## Quiz

[Module 14 Quiz](quiz/quiz-14.md)

---

## Summary

- ✅ Docker/Podman containerization
- ✅ Development and production compose files
- ✅ Environment configuration
- ✅ Basic deployment workflow
- ✅ GitHub Actions CI

---

## Course Complete!

Congratulations! You've completed the Full-Stack Development Course.

### What You've Built

A complete Job Board application with:
- FastAPI backend with PostgreSQL
- Vue 3 frontend with PrimeVue
- JWT authentication
- File uploads
- Search and filtering
- Containerized deployment

### Next Steps

1. **Polish your project** - Add more features, improve UI
2. **Deploy it** - Put it online for your portfolio
3. **Contribute to open source** - Apply your skills
4. **Keep learning** - Explore GraphQL, WebSockets, Kubernetes

Good luck with your job search!
