# DSL-Driven Design

The central design principle of Lingtarn Forge is: **DSL is the single contract between frontend and backend.**

## Why DSL?

Traditional web apps require coordinated changes across multiple layers:

```
Traditional: Database Schema ↔ API ↔ Frontend Components ↔ Business Logic
Lingtarn:    DSL → Everything
```

With DSL-driven design:
- **One change** (to the DSL) propagates everywhere
- **No build/deploy** for new app features — changes take effect immediately
- **AI can generate apps** by producing valid JSON, not code
- **Validation is centralized** — Pydantic validates the DSL on the backend

## DSL Structure

Every Lingtarn app is defined by a single JSON document:

```json
{
  "app_meta": { ... },     // App identity and display
  "schema": { ... },       // Field definitions and types
  "layout": { ... },       // How fields render in list and form views
  "logic": { ... }         // Validators, computations, workflow, automation, actions
}
```

### How Each Section Maps to the Platform

| DSL Section | Backend | Frontend |
|-------------|---------|----------|
| `app_meta` | `lt_sys_app` row | Sidebar menu item, app header |
| `schema.records` | `lt_sys_field_registry` rows | FormCanvas field rendering |
| `layout.list` | (not stored separately) | ListCanvas column configuration |
| `layout.form` | (not stored separately) | FormCanvas layout grid |
| `logic.validators` | `table_logic.validators` | Pre-submit validation |
| `logic.computations` | `table_logic.computations` → Formula Engine | Auto-computed fields |
| `logic.workflow` | Workflow Engine state machine | WorkflowCanvas state badges |
| `logic.automation` | Automation Engine event rules | (transparent to user) |
| `logic.actions` | Action Handler registry | Row/list action buttons |

## The DSL Lifecycle

```
1. Define DSL (manually or via AI Forge)
        │
        ▼
2. Validate (Pydantic: LingtarnDSL model)
        │
        ▼
3. Register App
   ├─ Create lt_sys_app (dsl_snapshot)
   ├─ Create lt_sys_table_registry (table_logic)
   ├─ Create lt_sys_field_registry (per field)
   └─ Mount lt_sys_menu entries
        │
        ▼
4. Render
   ├─ Frontend fetches DSL via API
   ├─ Registry maps field types → components
   ├─ ListCanvas renders table from layout.list
   └─ FormCanvas renders form from layout.form
        │
        ▼
5. Execute
   ├─ DataService resolves table from app_id + model
   ├─ Formula Engine runs computations on create/update
   ├─ Workflow Engine handles state transitions
   └─ Automation Engine fires rules on events
```

## DSL Versioning

Each app stores its DSL in `lt_sys_app.dsl_snapshot`. When the DSL is updated:

1. A new version is saved to `lt_sys_app_version`
2. The `dsl_snapshot` is updated atomically
3. Table and field registries are synced
4. Frontend components immediately reflect the new DSL

## Benefits of DSL-Driven Design

1. **Speed** — New apps in minutes, not weeks
2. **Consistency** — All apps follow the same patterns
3. **Auditability** — DSL is version-controlled and diffable
4. **AI-Friendly** — LLMs produce JSON more reliably than code
5. **Separation of Concerns** — DSL authors focus on "what", platform handles "how"
