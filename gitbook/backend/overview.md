# Backend Overview

The Lingtarn Forge backend is a Python FastAPI application that serves as the runtime engine for DSL-defined apps.

## Technology Stack

| Component | Technology |
|-----------|-----------|
| Framework | FastAPI (async) |
| ORM | SQLAlchemy 2.0 (async) |
| Database | PostgreSQL (asyncpg driver) |
| Validation | Pydantic v2 |
| Auth | JWT (python-jose) + bcrypt |
| LLM Client | OpenAI-compatible HTTP (httpx) |

## Directory Structure

```
backend/app/
├── main.py              # FastAPI app, CORS, lifespan
├── config.py            # Pydantic Settings from .env
├── core/
│   ├── database.py      # Async engine, session factory, Base
│   └── security.py      # JWT creation/verification, password hashing
├── api/
│   ├── deps.py          # Auth dependencies (get_current_user, etc.)
│   └── v1/
│       ├── router.py    # Route aggregation
│       ├── auth.py      # Login, register, me
│       ├── data.py      # Generic CRUD + actions + workflow
│       ├── apps.py      # App management
│       ├── forge.py     # AI Forge sessions
│       ├── menus.py     # Sidebar menus
│       ├── admin.py     # User/tenant/role admin
│       └── ...
├── models/              # SQLAlchemy ORM models
├── schemas/             # Pydantic request/response schemas
├── services/            # Business logic engines
│   ├── data_service.py
│   ├── formula_engine.py
│   ├── workflow_engine.py
│   ├── automation_engine.py
│   ├── action_handlers.py
│   ├── permission_service.py
│   ├── app_service.py
│   └── ...
├── assistant/           # AI assistant (context, prompts, LLM)
└── plugins/             # Plugin base, registry, platform API
```

## Key Patterns

### 1. Metadata-Driven CRUD

All data operations route through a single API:

```
POST   /api/v1/data/{app_id}/{model}          → Create
GET    /api/v1/data/{app_id}/{model}          → List
GET    /api/v1/data/{app_id}/{model}/{id}     → Get
PATCH  /api/v1/data/{app_id}/{model}/{id}     → Update
DELETE /api/v1/data/{app_id}/{model}/{id}     → Delete
```

The `data_service` resolves `app_id` + `model` to a physical table name via `lt_sys_table_registry`.

### 2. Action Registry

Custom actions are registered via decorators:

```python
from app.services.action_handlers import register_action

@register_action("my_custom_action")
async def my_custom_action(session, app_id, table_name, tenant_id, *, row_id=None, **kwargs):
    # Custom logic here
    return {"ok": True, "message": "Done"}
```

Actions are triggered via `POST /api/v1/data/{app_id}/{model}/action`.

### 3. Tenant Isolation

Every query is scoped by `tenant_id`. Non-superusers are forced to use their own tenant:

```python
def _tenant_id(user, query_tenant):
    if user and not user.is_superuser:
        return user.tenant_id or "default"
    return query_tenant or "default"
```

### 4. DSL-Driven Logic

On create/update, the backend:
1. Loads `dsl_snapshot` from `lt_sys_app`
2. Runs `logic.computations` through the Formula Engine
3. Checks `logic.constraints` (e.g., lock_when_status)
4. Fires `logic.automation` rules via Automation Engine

## Service Architecture

```
┌─────────────────────────────────────────────┐
│                 API Layer                     │
│  data.py  apps.py  forge.py  admin.py  ...  │
└──────────────────────┬──────────────────────┘
                       │
┌──────────────────────▼──────────────────────┐
│              Service Layer                    │
│                                              │
│  ┌─────────────┐  ┌──────────────────┐      │
│  │ DataService  │  │ PermissionService│      │
│  └──────┬──────┘  └──────────────────┘      │
│         │                                    │
│  ┌──────▼──────┐  ┌──────────────────┐      │
│  │FormulaEngine│  │ WorkflowEngine   │      │
│  └─────────────┘  └──────────────────┘      │
│                                              │
│  ┌─────────────┐  ┌──────────────────┐      │
│  │AutomationEng│  │ ActionHandlers   │      │
│  └─────────────┘  └──────────────────┘      │
└──────────────────────┬──────────────────────┘
                       │
┌──────────────────────▼──────────────────────┐
│              Data Layer                       │
│  SQLAlchemy models → PostgreSQL (lt_app_data) │
└─────────────────────────────────────────────┘
```

## Detailed Service Documentation

- [Data Service](data-service.md) — Table resolution, CRUD operations
- [Formula Engine](formula-engine.md) — Computed field evaluation
- [Workflow Engine](workflow-engine.md) — State machine transitions
- [Automation Engine](automation-engine.md) — Event-driven rules
- [Action Handlers](action-handlers.md) — Custom action registry
- [Plugin System](plugin-system.md) — Extending with plugins
