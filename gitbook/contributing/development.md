# Development Guide

How to set up a development environment and contribute to Lingtarn Forge.

## Development Setup

### Backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate  # or .venv\Scripts\Activate.ps1 on Windows
pip install -r requirements.txt
cp .env.example .env
# Edit .env with your database URL

python -m scripts.init_db
python -m scripts.ensure_all_tables
python -m scripts.seed_admin_user

uvicorn app.main:app --reload --port 8000
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

The Vite dev server runs on port 5173 and proxies `/api` to `localhost:8000`.

## Code Style

### Backend (Python)

- Python 3.12+ with type hints
- Async/await throughout (async SQLAlchemy, async FastAPI)
- Pydantic v2 for schemas and validation
- No business-specific code in core modules

### Frontend (TypeScript)

- React 18 with functional components and hooks
- TypeScript strict mode
- Tailwind CSS for styling
- shadcn/ui for base components

## Architecture Rules

1. **DSL is the contract** — All app behavior should be expressible in DSL. Avoid adding app-specific code to the core.
2. **Generic over specific** — Core services (DataService, FormulaEngine, etc.) should work for any app, not just specific use cases.
3. **Extension points** — Use the Action Handler registry and Plugin system for custom logic, not core modifications.
4. **Tenant isolation** — All data queries must be scoped by `tenant_id`. Never bypass this.

## Adding a New Field Type

### Backend

1. Add the type to `schemas/dsl.py` field type enum
2. Handle any special storage logic in `data_service.py` (usually not needed — JSONB stores anything)

### Frontend

1. Create a field component in `components/renderer/`
2. Register it in `Registry.tsx`
3. Add TypeScript type to `types/dsl.ts`

## Adding a New Formula Function

1. Add the implementation to `services/formula_engine.py`
2. Register in the function dispatch table
3. Add tests
4. Document in the DSL reference

## Adding a New Action Handler

1. Create a handler function with the standard signature
2. Register with `@register_action("my_handler")`
3. Use it in DSL: `{ "backend": "my_handler" }`

## Running Tests

### Backend

```bash
cd backend
pytest tests/ -v
```

### Frontend

```bash
cd frontend
npm test
```

## Project Structure

```
lingtarn-forge/
├── backend/          # FastAPI backend
├── frontend/         # React frontend
├── gitbook/          # Documentation (this)
├── scripts/          # Deployment scripts
├── docker-compose.yml
└── README.md
```

## Branch Strategy

- `main` — Stable release
- `opensource` — Open-source core framework
- `feat/*` — Feature branches
- `fix/*` — Bug fix branches
