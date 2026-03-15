# Docker Deployment

The recommended deployment method for Lingtarn Forge uses Docker Compose.

## Architecture

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Nginx   │───→│ Frontend │    │  Backend  │
│  (proxy) │───→│  :3000   │    │  :8000    │
└──────────┘    └──────────┘    └─────┬─────┘
                                      │
                               ┌──────▼──────┐
                               │  PostgreSQL  │
                               │    :5432     │
                               └──────────────┘
```

## Quick Start

```bash
# Clone repository
git clone https://github.com/your-org/lingtarn-forge.git
cd lingtarn-forge

# Configure environment
cp docker-compose.env.example .env
# Edit .env with your settings

# Start all services
docker compose up -d

# Initialize database (first time only)
docker compose exec backend python -m scripts.init_db
docker compose exec backend python -m scripts.ensure_all_tables
docker compose exec backend python -m scripts.seed_admin_user
```

## docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-lingtan}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-postgres}
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  backend:
    build: ./backend
    environment:
      DATABASE_URL: postgresql+asyncpg://postgres:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
      JWT_SECRET: ${JWT_SECRET}
      CORS_ORIGINS: ${CORS_ORIGINS:-http://localhost:3000}
      LLM_BASE_URL: ${LLM_BASE_URL:-}
      LLM_API_KEY: ${LLM_API_KEY:-}
      LLM_MODEL: ${LLM_MODEL:-gpt-4}
    ports:
      - "${BACKEND_PORT:-8000}:8000"
    depends_on:
      - postgres

  frontend:
    build: ./frontend
    environment:
      VITE_API_ORIGIN: ${VITE_API_ORIGIN:-}
    ports:
      - "${FRONTEND_PORT:-3000}:3000"
    depends_on:
      - backend

volumes:
  pgdata:
```

## Production Configuration

For production, use `docker-compose.prod.yml`:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

Production differences:
- Frontend served via Nginx with static build
- Backend runs with Gunicorn (multiple workers)
- Environment variables from `.env.prod`
- Health checks enabled
- Restart policies set to `always`

## Environment Variables

Create a `.env` file:

```env
# Database
POSTGRES_DB=lingtan
POSTGRES_PASSWORD=secure-password-here

# Backend
JWT_SECRET=your-jwt-secret-change-this
CORS_ORIGINS=https://your-domain.com

# AI (optional)
LLM_BASE_URL=https://api.openai.com/v1
LLM_API_KEY=sk-...
LLM_MODEL=gpt-4

# Ports
BACKEND_PORT=8000
FRONTEND_PORT=3000
```

## Database Backups

```bash
# Backup
docker compose exec postgres pg_dump -U postgres lingtan > backup.sql

# Restore
cat backup.sql | docker compose exec -T postgres psql -U postgres lingtan
```

## Updating

```bash
git pull
docker compose build
docker compose up -d
docker compose exec backend python -m scripts.ensure_all_tables
```

## Monitoring

```bash
# View logs
docker compose logs -f backend
docker compose logs -f frontend

# Check health
docker compose ps
```
