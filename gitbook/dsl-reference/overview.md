# DSL Overview

The Lingtarn DSL is a JSON-based domain-specific language that defines an entire application — its data model, user interface, and business logic.

## Top-Level Structure

```json
{
  "app_meta": { ... },
  "schema": { ... },
  "layout": { ... },
  "logic": { ... }
}
```

| Section | Purpose |
|---------|---------|
| [`app_meta`](app-meta.md) | App identity: name, icon, version, description |
| [`schema`](schema.md) | Field definitions: types, validation, relations |
| [`layout`](layout.md) | UI configuration: list columns, form grid, tabs |
| [`logic`](logic.md) | Business rules: validators, computations, actions, constraints |
| [`workflow`](workflow.md) | State machine: states, transitions, roles |
| [`automation`](automation.md) | Event-driven rules: triggers, actions, field mapping |

## Minimal Example

The smallest valid DSL:

```json
{
  "app_meta": {
    "name": "Notes"
  },
  "schema": {
    "records": [
      { "id": "title", "label": "Title", "type": "text", "required": true },
      { "id": "content", "label": "Content", "type": "text" }
    ]
  },
  "layout": {
    "list": ["title"],
    "form": [["title"], ["content"]]
  }
}
```

This creates a fully functional note-taking app with a list view (showing title) and a form view (title on first row, content on second row).

## Complete Example

A more realistic example with workflow and automation:

```json
{
  "app_meta": {
    "name": "Purchase Orders",
    "icon": "ShoppingBag",
    "description": "Manage purchase orders with approval workflow"
  },
  "schema": {
    "records": [
      { "id": "po_number", "label": "PO Number", "type": "text", "required": true },
      { "id": "supplier", "label": "Supplier", "type": "text", "required": true },
      { "id": "amount", "label": "Amount", "type": "number" },
      { "id": "status", "label": "Status", "type": "select",
        "options": ["Draft", "Pending", "Approved", "Rejected"] },
      { "id": "notes", "label": "Notes", "type": "text" },
      { "id": "created_by", "label": "Created By", "type": "text", "readonly": true }
    ]
  },
  "layout": {
    "list": ["po_number", "supplier", "amount", "status"],
    "form": [
      ["po_number", "supplier"],
      ["amount", "status"],
      ["notes"],
      ["created_by"]
    ]
  },
  "logic": {
    "computations": [
      {
        "target": "created_by",
        "formula": "$current.creator_display_name",
        "trigger": "create"
      }
    ],
    "workflow": {
      "field": "status",
      "states": [
        { "id": "Draft", "label": "Draft", "color": "gray" },
        { "id": "Pending", "label": "Pending Approval", "color": "yellow" },
        { "id": "Approved", "label": "Approved", "color": "green" },
        { "id": "Rejected", "label": "Rejected", "color": "red" }
      ],
      "transitions": [
        { "id": "submit", "from": "Draft", "to": "Pending", "label": "Submit for Approval" },
        { "id": "approve", "from": "Pending", "to": "Approved", "label": "Approve",
          "allowed_roles": ["manager", "admin"] },
        { "id": "reject", "from": "Pending", "to": "Rejected", "label": "Reject",
          "allowed_roles": ["manager", "admin"], "requires_comment": true }
      ]
    },
    "constraints": [
      { "rule": "lock_when_status", "value": ["Approved", "Rejected"] }
    ],
    "automation": [
      {
        "id": "notify_on_approve",
        "trigger": "workflow_transition",
        "trigger_meta": { "transition_id": "approve" },
        "actions": [
          {
            "type": "notify",
            "message": "PO {po_number} has been approved"
          }
        ]
      }
    ]
  }
}
```

## DSL Validation

When a DSL is submitted (via AI Forge or API), it is validated by the `LingtarnDSL` Pydantic model on the backend. Validation checks:

- Required fields are present (`app_meta.name`, at least one schema record)
- Field types are valid (text, number, select, date, boolean, etc.)
- Workflow states referenced in transitions exist
- Automation trigger types are valid
- Field IDs referenced in layout exist in schema

## Sub-Schemas (Multi-Table Apps)

For apps with multiple related tables, use `sub_schemas`:

```json
{
  "schema": {
    "records": [ ... ],
    "sub_schemas": {
      "line_items": {
        "records": [
          { "id": "product", "label": "Product", "type": "text" },
          { "id": "quantity", "label": "Qty", "type": "number" },
          { "id": "price", "label": "Price", "type": "number" }
        ]
      }
    }
  }
}
```

Each sub-schema creates a separate table (e.g., `myapp_line_items`) that can be rendered as inline lists within the main form.
