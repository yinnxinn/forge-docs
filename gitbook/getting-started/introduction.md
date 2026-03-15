# Introduction

## What is Lingtarn Forge?

Lingtarn Forge (灵碳云铸) is an open-source, **DSL-driven full-stack platform** for building business applications. Instead of writing application code for each new feature or module, you define your application in a structured JSON-based DSL — and the platform handles rendering, data storage, permissions, workflows, and automation automatically.

## The Problem

Building enterprise business applications typically requires:

1. Designing database schemas for each module
2. Writing CRUD APIs for each entity
3. Building frontend forms and list views
4. Implementing workflow/approval logic
5. Wiring up cross-module data flows
6. Handling multi-tenant isolation and permissions

This is repetitive, error-prone, and expensive. Each new module takes weeks of development.

## The Solution

Lingtarn Forge introduces a **single abstraction** — the Lingtarn DSL — that captures all of the above in one JSON document:

```json
{
  "app_meta": { "name": "Sales Orders", "icon": "ShoppingCart" },
  "schema": {
    "records": [
      { "id": "customer", "label": "Customer", "type": "text", "required": true },
      { "id": "amount", "label": "Amount", "type": "number" },
      { "id": "status", "label": "Status", "type": "select", "options": ["Draft", "Confirmed", "Shipped"] }
    ]
  },
  "layout": {
    "list": ["customer", "amount", "status"],
    "form": [["customer", "amount"], ["status"]]
  },
  "logic": {
    "workflow": {
      "field": "status",
      "states": [
        { "id": "Draft", "label": "Draft", "color": "gray" },
        { "id": "Confirmed", "label": "Confirmed", "color": "blue" },
        { "id": "Shipped", "label": "Shipped", "color": "green" }
      ],
      "transitions": [
        { "id": "confirm", "from": "Draft", "to": "Confirmed", "label": "Confirm" },
        { "id": "ship", "from": "Confirmed", "to": "Shipped", "label": "Ship" }
      ]
    }
  }
}
```

From this single DSL document, Lingtarn Forge automatically:

- Creates the data storage (JSONB in PostgreSQL)
- Registers table and field metadata
- Renders a list view and form view in the frontend
- Enables workflow state transitions with role-based access control
- Fires automation rules on state changes
- Applies computed fields via the formula engine

## Who Is It For?

- **Developers** who want to rapidly build business applications without repetitive CRUD code
- **Product Managers** who want to prototype and iterate on app designs quickly
- **Enterprises** that need multi-tenant, permission-controlled business modules
- **AI Enthusiasts** who want to see how LLMs can generate functional apps from natural language

## Core Concepts

| Concept | Description |
|---------|-------------|
| **DSL** | JSON document defining an entire app (schema, layout, logic) |
| **App** | A registered DSL instance with its own data, menus, and permissions |
| **Tenant** | An isolated organization within the platform |
| **Forge** | The AI-powered interface where users describe apps in natural language |
| **Renderer** | Frontend components that dynamically render UI from DSL |
| **Engine** | Backend services (Formula, Workflow, Automation) that execute DSL logic |

## Next Steps

- [Quick Start](quick-start.md) — Get the platform running in 5 minutes
- [Architecture Overview](../architecture/overview.md) — Understand the system design
- [DSL Reference](../dsl-reference/overview.md) — Learn the DSL specification
