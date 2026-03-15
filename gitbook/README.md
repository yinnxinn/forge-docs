# Lingtarn Forge (灵碳云铸)

An open-source, DSL-driven full-stack platform for building business applications with AI.

**Open-source branch:** The `open-source` branch contains only the **core framework** (DSL runtime, CRUD engine, formula/workflow/automation engines, frontend renderer, auth & RBAC). See [Open Source Scope](architecture/open-source-scope.md) for what is included and what is not.

## What is Lingtarn Forge?

Lingtarn Forge is a **metadata-driven application platform** where users describe what they want in natural language, AI generates structured DSL (Domain-Specific Language), and the platform automatically renders UI and executes business logic — without hand-written application code.

**The core idea:** DSL is the single contract between frontend and backend. AI only edits DSL; both sides react automatically.

## Key Features

- **DSL as Single Source of Truth** — All app behavior (schema, layout, logic, workflow, automation) is expressed in a JSON-based DSL. No app-level code needed.
- **AI-Powered App Generation** — Users describe requirements in natural language; the AI Forge generates valid DSL that the platform immediately renders.
- **Dynamic Rendering Engine** — Frontend components (`FormCanvas`, `ListCanvas`, `WorkflowCanvas`) render directly from DSL definitions.
- **Metadata-Driven CRUD** — A single API endpoint (`/api/v1/data/{app_id}/{model}`) handles all data operations for any app, driven by table and field registries.
- **Workflow Engine** — DSL-defined state machines with role-based transitions, approval flows, and audit history.
- **Automation Engine** — Event-driven rules (record created/updated, workflow transitions) that trigger cross-app data operations.
- **Formula Engine** — Computed fields with expressions (`IF`, `SEQUENCE`, `concat`, arithmetic, date functions).
- **Multi-Tenant RBAC** — Tenant isolation, role-based access control, department hierarchy, and row-level data rules.
- **Plugin Architecture** — Extend the platform with custom plugins when DSL alone isn't enough.

## Architecture at a Glance

```
┌─────────────────────────────────────────────────────┐
│              Lingtarn DSL (JSON)                     │
│         (app_meta, schema, layout, logic)            │
└──────────┬──────────────────────────┬───────────────┘
           │                          │
  ┌────────▼────────┐       ┌────────▼────────┐
  │  Backend Engine  │       │ Frontend Engine  │
  │  · DataService   │       │ · FormCanvas     │
  │  · FormulaEngine │       │ · ListCanvas     │
  │  · WorkflowEngine│       │ · WorkflowCanvas │
  │  · AutomationEng │       │ · Registry       │
  └────────┬────────┘       └─────────────────┘
           │
  ┌────────▼────────┐
  │  PostgreSQL      │
  │  · lt_app_data   │  (JSONB)
  │  · lt_sys_*      │  (metadata, RBAC)
  └─────────────────┘
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.12+, FastAPI, SQLAlchemy (async), PostgreSQL |
| Frontend | React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui |
| AI | OpenAI-compatible LLM API (configurable) |
| Deployment | Docker Compose, Nginx |

## Quick Start

```bash
git clone https://github.com/your-org/lingtarn-forge.git
cd lingtarn-forge
docker compose up -d
```

Visit `http://localhost:3000` and log in with the default admin account.

See [Getting Started](getting-started/quick-start.md) for detailed instructions.

## Documentation

- [Architecture Overview](architecture/overview.md)
- [Open Source Scope](architecture/open-source-scope.md)
- [DSL Reference](dsl-reference/overview.md)
- [Backend Framework](backend/overview.md)
- [Frontend Framework](frontend/overview.md)
- [API Reference](api-reference/data-api.md)
- [Deployment Guide](deployment/docker.md)
- [Contributing](contributing/development.md)

## Languages

This documentation is available in:

- **English** (default)
- **中文** (Chinese)

Use the language switcher in the built book (HonKit/GitBook) or open the root for English and the `zh/` folder for Chinese. Configuration: [LANGS.md](LANGS.md).

## Building this book

This folder follows [GitBook](https://www.gitbook.com/) structure: `README.md` is the cover, `SUMMARY.md` is the table of contents. Multi-language is configured via `LANGS.md`. You can build with:

- **GitBook CLI:** `npx gitbook-cli build . ./_book`
- **HonKit (GitBook fork):** `npx honkit build`
- Or use any Markdown-based static site generator (MkDocs, Docusaurus, etc.) by pointing it at this directory.

## License

MIT License — see [LICENSE](../LICENSE) for details.
