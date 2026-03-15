# Architecture Overview

Lingtarn Forge is a **metadata-driven, DSL-first** platform. This document explains the high-level architecture and how the pieces fit together.

## Design Principles

| Principle | Description |
|-----------|-------------|
| **DSL as the only contract** | All app behavior is expressed in Lingtarn DSL (JSON). Backend interprets it; frontend renders it. No app-level code is needed. |
| **AI work boundaries** | Level 1: edit DSL only (preferred). Level 2: write plugin code within templates. Level 3: modify core platform (last resort). |
| **Type safety end-to-end** | OpenAPI spec → generated TypeScript types → type-safe API client. |
| **Metadata-driven** | Logic and storage are separated. Metadata drives rendering and behavior at runtime. |
| **Multi-tenant by default** | Every data row, permission, and menu is scoped to a tenant. |

## System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                           User / AI Forge                            │
│                    "Create a CRM with leads and deals"               │
└────────────────────────────────┬─────────────────────────────────────┘
                                 │ Natural Language
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│                        AI Forge (LLM)                                │
│               Generates validated Lingtarn DSL (JSON)                │
└────────────────────────────────┬─────────────────────────────────────┘
                                 │ DSL
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│                      Lingtarn DSL (JSON)                             │
│                                                                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐               │
│  │ app_meta │ │  schema  │ │  layout  │ │  logic   │               │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘               │
└──────────┬────────────────────────────────────┬─────────────────────┘
           │                                    │
  ┌────────▼─────────────────┐       ┌──────────▼──────────────┐
  │     Backend Engine       │       │    Frontend Engine       │
  │                          │       │                          │
  │  · App Registration      │       │  · Registry (type→comp)  │
  │  · DataService (CRUD)    │       │  · FormCanvas            │
  │  · FormulaEngine         │       │  · ListCanvas            │
  │  · WorkflowEngine        │       │  · WorkflowCanvas        │
  │  · AutomationEngine      │       │  · PluginCanvas          │
  │  · ActionHandlers        │       │  · EmbedPageCanvas       │
  │  · PermissionService     │       │                          │
  │  · PluginSystem          │       │                          │
  └────────┬─────────────────┘       └──────────────────────────┘
           │
  ┌────────▼─────────────────┐
  │      PostgreSQL           │
  │                           │
  │  Metadata:                │
  │  · lt_sys_app             │
  │  · lt_sys_table_registry  │
  │  · lt_sys_field_registry  │
  │                           │
  │  RBAC:                    │
  │  · lt_sys_tenant          │
  │  · lt_sys_role            │
  │  · lt_sys_permission      │
  │  · lt_sys_data_rule       │
  │                           │
  │  Business Data:           │
  │  · lt_app_data (JSONB)    │
  └───────────────────────────┘
```

## Request Lifecycle

Every data operation follows this flow:

```
HTTP Request
    │
    ▼
┌─────────────┐
│  Auth Layer  │  JWT verification, user loading
└──────┬──────┘
       ▼
┌─────────────┐
│  Permission  │  Check RBAC (table + action)
│   Service    │  Check data rules (row-level)
└──────┬──────┘
       ▼
┌─────────────┐
│  Resolve     │  app_id + model → table_name
│  Metadata    │  Load table_logic, field_registry
└──────┬──────┘
       ▼
┌─────────────┐
│  Run Logic   │  Validators → Computations (Formula Engine)
└──────┬──────┘
       ▼
┌─────────────┐
│  Execute     │  INSERT / UPDATE / DELETE on lt_app_data
│  SQL         │  Tenant + department isolation
└──────┬──────┘
       ▼
┌─────────────┐
│  Automation  │  Fire rules: record_created, record_updated
│  Engine      │  Cross-app data operations
└──────┬──────┘
       ▼
┌─────────────┐
│  Response    │  Return DataRowOut (id, data, timestamps)
└─────────────┘
```

## Component Map

| Component | Location | Responsibility |
|-----------|----------|---------------|
| **FastAPI App** | `backend/app/main.py` | HTTP server, CORS, lifecycle |
| **Config** | `backend/app/config.py` | Environment settings |
| **Database** | `backend/app/core/database.py` | Async SQLAlchemy engine |
| **Security** | `backend/app/core/security.py` | JWT, password hashing |
| **Auth Deps** | `backend/app/api/deps.py` | `get_current_user`, permission helpers |
| **Data API** | `backend/app/api/v1/data.py` | Generic CRUD endpoints |
| **Data Service** | `backend/app/services/data_service.py` | Table resolution, row operations |
| **Formula Engine** | `backend/app/services/formula_engine.py` | Computed field evaluation |
| **Workflow Engine** | `backend/app/services/workflow_engine.py` | State machine transitions |
| **Automation Engine** | `backend/app/services/automation_engine.py` | Event-driven rules |
| **Action Handlers** | `backend/app/services/action_handlers.py` | Custom action registry |
| **Permission Service** | `backend/app/services/permission_service.py` | RBAC checks |
| **DSL Types** | `frontend/src/types/dsl.ts` | TypeScript DSL interfaces |
| **DSL Helpers** | `frontend/src/lib/dsl.ts` | DSL parsing utilities |
| **Registry** | `frontend/src/components/renderer/Registry.tsx` | Field type → component map |
| **FormCanvas** | `frontend/src/components/renderer/FormCanvas.tsx` | Dynamic form renderer |
| **ListCanvas** | `frontend/src/components/renderer/ListCanvas.tsx` | Dynamic table renderer |

## AI Capability Levels

| Level | Scope | What AI Can Do |
|-------|-------|----------------|
| **Level 1** | Pure DSL | CRUD apps, computed fields, workflows, automation rules |
| **Level 2** | DSL Edge | Action conditions, field mapping with lookup, cascade automation |
| **Level 3** | Plugins | External APIs, scheduled tasks, file generation |
| **Level 4** | Platform | WebSockets, ML integration, OCR |

Most business apps can be built at Level 1–2 without any code.
