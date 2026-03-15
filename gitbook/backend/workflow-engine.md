# Workflow Engine

The Workflow Engine (`backend/app/services/workflow_engine.py`) implements a state machine for record lifecycle management.

## Overview

Workflows enable approval flows, status progressions, and lifecycle management — all configured in DSL without code.

## How It Works

### State Machine Model

A workflow is a directed graph of states connected by transitions:

```
[Draft] ──submit──→ [Pending] ──approve──→ [Approved]
                              ──reject───→ [Rejected] ──revise──→ [Draft]
```

### Key Methods

#### `get_state_info(workflow_cfg, current_state) → dict`

Returns information about the current state (label, color).

#### `get_allowed_transitions(workflow_cfg, current_state, user_roles) → list`

Returns which transitions the current user can execute, based on:
- Current state matches `transition.from`
- User has one of `transition.allowed_roles` (if specified)

#### `execute_transition(db, workflow_cfg, app_id, table_name, tenant_id, record_id, transition_id, current_user_roles, current_user_id, current_user_name, comment=None) → dict`

Executes a workflow transition:

1. Load the record from `lt_app_data`
2. Validate the current state matches `transition.from`
3. Check `allowed_roles` (if specified)
4. Check `requires_comment` (if true, comment must be provided)
5. Update `data[workflow.field]` to `transition.to`
6. Append entry to `data._workflow_history`
7. Return `{ from_state, to_state, label, record }`

### Workflow History

Each transition appends to `_workflow_history` in the record's data:

```json
{
  "_workflow_history": [
    {
      "transition_id": "submit",
      "from": "Draft",
      "to": "Pending",
      "user_id": 7,
      "user_name": "Alice",
      "at": "2026-03-14T10:00:00Z"
    },
    {
      "transition_id": "approve",
      "from": "Pending",
      "to": "Approved",
      "user_id": 3,
      "user_name": "Bob Manager",
      "comment": "Approved - within budget",
      "at": "2026-03-14T14:30:00Z"
    }
  ]
}
```

## API Endpoints

### Get Workflow State

```
GET /api/v1/data/{app_id}/{model}/{record_id}/workflow
```

Returns:
```json
{
  "workflow_field": "status",
  "current_state": "Pending",
  "state_info": { "id": "Pending", "label": "Pending Approval", "color": "yellow" },
  "allowed_transitions": [
    { "id": "approve", "label": "Approve", "to": "Approved" },
    { "id": "reject", "label": "Reject", "to": "Rejected" }
  ],
  "history": [ ... ],
  "all_states": [ ... ]
}
```

### Execute Transition

```
POST /api/v1/data/{app_id}/{model}/{record_id}/workflow/{transition_id}?comment=...
```

Returns:
```json
{
  "ok": true,
  "transition_id": "approve",
  "from_state": "Pending",
  "to_state": "Approved",
  "label": "Approve",
  "side_effects": [ ... ],
  "message": "「Approve」executed successfully"
}
```

## Integration with Automation

After a successful transition, the API layer fires the Automation Engine with `trigger_type = "workflow_transition"`. Automation rules matching the transition ID are executed, enabling cross-app data flows triggered by workflow events.

## Error Handling

The Workflow Engine raises `WorkflowError` for:
- Record not found
- Invalid transition (current state doesn't match `from`)
- Unauthorized (user doesn't have required role)
- Missing comment (when `requires_comment = true`)

These are caught by the API layer and returned as HTTP 400 errors.
