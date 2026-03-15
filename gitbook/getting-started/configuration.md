# Configuration

Lingtarn Forge uses environment variables for all configuration. The backend reads from `backend/.env` via Pydantic Settings.

## Backend Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL async connection string | `postgresql+asyncpg://postgres:postgres@localhost:5432/lingtan` |
| `JWT_SECRET` | Secret key for JWT token signing | (required) |
| `JWT_EXPIRE_MINUTES` | Token expiration time in minutes | `1440` (24h) |
| `CORS_ORIGINS` | Comma-separated allowed origins | `http://localhost:3000` |
| `LLM_BASE_URL` | OpenAI-compatible API base URL | `https://api.openai.com/v1` |
| `LLM_API_KEY` | API key for LLM service | (required for AI Forge) |
| `LLM_MODEL` | Model name for AI Forge | `gpt-4` |
| `REDIS_URL` | Redis connection string (optional) | `redis://localhost:6379` |

### Example `.env`

```env
DATABASE_URL=postgresql+asyncpg://postgres:mypassword@localhost:5432/lingtan
JWT_SECRET=your-secret-key-change-in-production
JWT_EXPIRE_MINUTES=1440
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
LLM_BASE_URL=https://api.openai.com/v1
LLM_API_KEY=sk-...
LLM_MODEL=gpt-4
```

## Frontend Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `VITE_API_ORIGIN` | Backend API base URL | (empty, uses proxy) |

In development, Vite proxies `/api` requests to the backend. In production, set `VITE_API_ORIGIN` to the backend URL or use Nginx reverse proxy.

## Database Setup

The platform uses PostgreSQL with async SQLAlchemy. Tables are created automatically by the init scripts:

```bash
# Create all tables
python -m scripts.init_db

# Ensure latest schema
python -m scripts.ensure_all_tables

# Create admin user
python -m scripts.seed_admin_user
```

## Docker Configuration

For Docker Compose deployment, copy and edit the environment template:

```bash
cp docker-compose.env.example .env
```

Key Docker variables:

| Variable | Description |
|----------|-------------|
| `POSTGRES_PASSWORD` | PostgreSQL root password |
| `POSTGRES_DB` | Database name |
| `BACKEND_PORT` | Exposed backend port |
| `FRONTEND_PORT` | Exposed frontend port |

## LLM Provider Configuration

The AI Forge works with any OpenAI-compatible API. Configure the provider:

```env
# OpenAI
LLM_BASE_URL=https://api.openai.com/v1
LLM_API_KEY=sk-...
LLM_MODEL=gpt-4

# Azure OpenAI
LLM_BASE_URL=https://your-resource.openai.azure.com/openai/deployments/your-deployment
LLM_API_KEY=your-azure-key
LLM_MODEL=gpt-4

# Local (Ollama, vLLM, etc.)
LLM_BASE_URL=http://localhost:11434/v1
LLM_API_KEY=not-needed
LLM_MODEL=llama3
```

## Security Notes

- Always change `JWT_SECRET` in production
- Use strong `POSTGRES_PASSWORD`
- Restrict `CORS_ORIGINS` to your actual domain
- Never expose the backend directly; use a reverse proxy (Nginx)
- Store sensitive config in environment variables, not in files committed to git
