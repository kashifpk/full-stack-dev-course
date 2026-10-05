# Assignment 14: Deploy Job Board

## Objective

Containerize and deploy the complete Job Board application.

## Requirements

### 1. Dockerize Backend

Create `docker/backend.Dockerfile`:
- Use Python 3.11 slim image
- Install dependencies
- Copy application code
- Run as non-root user
- Expose port 8000

### 2. Dockerize Frontend

Create `docker/frontend.Dockerfile`:
- Multi-stage build
- Build Vue app in Node image
- Serve with Nginx
- Configure for Vue Router history mode

### 3. Docker Compose Development

Create `docker-compose.yml`:
- PostgreSQL service with health check
- Backend service with hot reload
- Frontend service with hot reload
- Redis service (for future caching)
- Shared network
- Volume for uploads

### 4. Docker Compose Production

Create `docker-compose.prod.yml`:
- No hot reload
- Environment variables from .env
- Restart policies
- Production configurations

### 5. Environment Setup

Create environment files:
- `.env.example` with all variables documented
- `.env` for local development
- Secure production values

### 6. Deployment Script

Create `deploy.sh`:
- Pull latest code
- Build images
- Run migrations
- Restart services
- Basic health check

## Deliverables

- [ ] Backend Dockerfile working
- [ ] Frontend Dockerfile working
- [ ] docker-compose.yml starts full dev environment
- [ ] docker-compose.prod.yml starts production environment
- [ ] All environment variables documented
- [ ] README with deployment instructions

## Testing

```bash
# Development
docker compose up -d
# Visit http://localhost:5173 (frontend)
# Visit http://localhost:8000/docs (backend API)

# Production simulation
docker compose -f docker-compose.prod.yml up -d
# Visit http://localhost (frontend with nginx)

# Run migrations
docker compose exec backend alembic upgrade head

# Check logs
docker compose logs -f

# Cleanup
docker compose down -v
```

## Bonus: CI/CD

Set up GitHub Actions:
- Run tests on push
- Build Docker images
- Push to registry
- Deploy to server (optional)
