# workflow

The `workflow` section defines a state machine for record lifecycle management (e.g., approval flows).

## Structure

```json
{
  "logic": {
    "workflow": {
      "field": "status",
      "states": [ ... ],
      "transitions": [ ... ]
    }
  }
}
```

## States

Each state represents a step in the lifecycle.

```json
{
  "states": [
    { "id": "draft", "label": "Draft", "color": "gray" },
    { "id": "pending", "label": "Pending Approval", "color": "yellow" },
    { "id": "approved", "label": "Approved", "color": "green" },
    { "id": "rejected", "label": "Rejected", "color": "red" }
  ]
}
```

| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | Unique state identifier (stored in data) |
| `label` | `string` | Display label |
| `color` | `string` | Badge color: `gray`, `yellow`, `green`, `red`, `blue`, `purple` |

## Transitions

Transitions define how records move between states.

```json
{
  "transitions": [
    {
      "id": "submit",
      "from": "draft",
      "to": "pending",
      "label": "Submit for Approval"
    },
    {
      "id": "approve",
      "from": "pending",
      "to": "approved",
      "label": "Approve",
      "allowed_roles": ["manager", "admin"],
      "requires_comment": false
    },
    {
      "id": "reject",
      "from": "pending",
      "to": "rejected",
      "label": "Reject",
      "allowed_roles": ["manager", "admin"],
      "requires_comment": true
    },
    {
      "id": "revise",
      "from": "rejected",
      "to": "draft",
      "label": "Revise & Resubmit"
    }
  ]
}
```

| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | Unique transition identifier |
| `from` | `string` | Source state ID |
| `to` | `string` | Target state ID |
| `label` | `string` | Button label |
| `allowed_roles` | `string[]` | Roles that can execute this transition (empty = anyone) |
| `requires_comment` | `boolean` | Whether a comment is required |

## How It Works

### Frontend (WorkflowCanvas)

1. Fetches current workflow state via `GET /data/{app_id}/{model}/{id}/workflow`
2. Displays current state as a colored badge
3. Shows allowed transitions as buttons (filtered by user's roles)
4. Clicking a transition button calls `POST /data/{app_id}/{model}/{id}/workflow/{transition_id}`

### Backend (WorkflowEngine)

1. Loads the workflow config from DSL
2. Validates the transition:
   - Current state matches `transition.from`
   - User has one of `allowed_roles` (if specified)
   - Comment provided if `requires_comment`
3. Updates `data[workflow.field]` to `transition.to`
4. Appends to `data._workflow_history`:
   ```json
   {
     "transition_id": "approve",
     "from": "pending",
     "to": "approved",
     "user_id": 7,
     "user_name": "John Manager",
     "comment": "Looks good",
     "at": "2026-03-14T10:30:00Z"
   }
   ```
5. Triggers automation rules matching `workflow_transition`

## Workflow + Automation

Workflow transitions are a powerful trigger for automation. When a transition fires, the automation engine checks for matching rules:

```json
{
  "automation": [
    {
      "trigger": "workflow_transition",
      "trigger_meta": { "transition_id": "approve" },
      "actions": [
        {
          "type": "create_record",
          "target_app_id": "warehouse_tasks",
          "target_model": "main",
          "field_mapping": [
            { "target": "order_id", "source": "$current.id" },
            { "target": "status", "mode": "literal", "value": "Pending" }
          ]
        }
      ]
    }
  ]
}
```

## Common Patterns

### Simple Approval

```
Draft → Pending → Approved
                → Rejected → Draft (revise)
```

### Multi-Stage Approval

```
Draft → Review → Manager Approval → Director Approval → Active
```

### Lifecycle with Cancellation

```
Draft → Confirmed → In Progress → Completed
   ↓                    ↓
Cancelled            Cancelled
```
