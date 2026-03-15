# Quick Start

Get Lingtarn Forge running locally in under 5 minutes.

## Prerequisites

- Docker and Docker Compose
- Git
- (Optional) Node.js 18+ and Python 3.12+ for development mode

## Option 1: Docker Compose (Recommended)

```bash
# Clone the repository
git clone https://github.com/your-org/lingtarn-forge.git
cd lingtarn-forge

# Copy environment template
cp docker-compose.env.example .env

# Start all services
docker compose up -d
```

This starts:
- **PostgreSQL** on port 5432
- **Redis** on port 6379
- **Backend** (FastAPI) on port 8000
- **Frontend** (Vite/React) on port 3000

Visit `http://localhost:3000` to access the platform.

## Option 2: Manual Setup (Development)

### Backend

```bash
cd backend

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\Activate.ps1  # Windows PowerShell

# Install dependencies
pip install -r requirements.txt

# Copy environment config
cp .env.example .env
# Edit .env with your database URL

# Initialize database
python -m scripts.init_db
python -m scripts.ensure_all_tables
python -m scripts.seed_admin_user

# Start the server
uvicorn app.main:app --reload --port 8000
```

### Frontend

```bash
cd frontend

# Install dependencies
npm install

# Copy environment config
cp .env.example .env

# Start dev server
npm run dev
```

## First Login

After setup, log in with the default admin account:

- **Username:** `admin`
- **Password:** `admin123` (change this immediately in production)

## Creating Your First App

1. Navigate to **AI Forge** (available to admin/developer roles)
2. Describe your app in natural language:
   > "Create a task management app with fields: title (text, required), assignee (text), priority (select: High/Medium/Low), due date (date), status (select: Open/InProgress/Done)"
3. The AI generates a Lingtarn DSL
4. Click **Publish** to register the app
5. The app appears in the sidebar — ready to use

## Project Structure

```
lingtarn-forge/
├── backend/                 # FastAPI backend
│   ├── app/
│   │   ├── main.py          # App entry point
│   │   ├── config.py        # Environment configuration
│   │   ├── core/            # Database, security
│   │   ├── api/v1/          # API routes
│   │   ├── models/          # SQLAlchemy models
│   │   ├── schemas/         # Pydantic schemas
│   │   ├── services/        # Business logic engines
│   │   ├── assistant/       # AI assistant
│   │   └── plugins/         # Plugin system
│   ├── scripts/             # DB init & seed scripts
│   └── requirements.txt
├── frontend/                # React frontend
│   ├── src/
│   │   ├── api/             # API client
│   │   ├── components/      # UI components
│   │   │   ├── renderer/    # DSL renderers
│   │   │   ├── layout/      # Shell components
│   │   │   └── ui/          # Base UI (shadcn)
│   │   ├── contexts/        # React contexts
│   │   ├── pages/           # Route pages
│   │   ├── types/           # TypeScript types
│   │   └── lib/             # Utilities
│   └── package.json
├── docker-compose.yml
├── gitbook/                 # This documentation
└── scripts/                 # Deployment scripts
```

## Next Steps

- [Configuration](configuration.md) — Customize environment settings
- [Architecture Overview](../architecture/overview.md) — Understand the system design
- [DSL Reference](../dsl-reference/overview.md) — Learn to write DSL by hand
