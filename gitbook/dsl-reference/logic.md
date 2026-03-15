# logic

The `logic` section defines business rules: validators, computations, actions, constraints, and filters.

## Structure

```json
{
  "logic": {
    "validators": [ ... ],
    "computations": [ ... ],
    "actions": [ ... ],
    "constraints": [ ... ],
    "filters": [ ... ],
    "workflow": { ... },
    "automation": [ ... ]
  }
}
```

## Validators

Validators run before a record is saved. If any validator fails, the save is rejected.

```json
{
  "logic": {
    "validators": [
      {
        "field": "amount",
        "rule": "min",
        "value": 0,
        "message": "Amount must be non-negative"
      },
      {
        "field": "email",
        "rule": "regex",
        "value": "^[\\w.+-]+@[\\w-]+\\.[\\w.-]+$",
        "message": "Invalid email format"
      }
    ]
  }
}
```

| Property | Description |
|----------|-------------|
| `field` | Target field ID |
| `rule` | Validation rule: `min`, `max`, `regex`, `required`, `unique` |
| `value` | Rule parameter |
| `message` | Error message on failure |

## Computations

Computations automatically calculate field values on create or update.

```json
{
  "logic": {
    "computations": [
      {
        "target": "total",
        "formula": "add($current.quantity, mul($current.price, $current.quantity))",
        "trigger": "always"
      },
      {
        "target": "order_no",
        "formula": "SEQUENCE('SO', 6)",
        "trigger": "create"
      },
      {
        "target": "updated_by",
        "formula": "$current.creator_display_name",
        "trigger": "always"
      }
    ]
  }
}
```

| Property | Description |
|----------|-------------|
| `target` | Field ID to set the computed value |
| `formula` | Expression evaluated by the Formula Engine |
| `trigger` | When to evaluate: `create`, `update`, or `always` |

### Available Formula Functions

| Function | Description | Example |
|----------|-------------|---------|
| `IF(cond, then, else)` | Conditional | `IF($current.amount > 1000, 'VIP', 'Standard')` |
| `add(a, b)` | Addition | `add($current.qty, $current.bonus)` |
| `sub(a, b)` | Subtraction | `sub($current.total, $current.discount)` |
| `mul(a, b)` | Multiplication | `mul($current.price, $current.qty)` |
| `div(a, b)` | Division | `div($current.total, $current.count)` |
| `round(n, decimals)` | Round | `round($current.avg, 2)` |
| `abs(n)` | Absolute value | `abs($current.diff)` |
| `concat(a, b, ...)` | String join | `concat($current.first, ' ', $current.last)` |
| `coalesce(a, b, ...)` | First non-null | `coalesce($current.nickname, $current.name)` |
| `now()` | Current timestamp | `now()` |
| `date_of(dt)` | Extract date | `date_of(now())` |
| `time_of_day(dt)` | Extract time | `time_of_day(now())` |
| `year(dt)` | Extract year | `year(now())` |
| `month(dt)` | Extract month | `month(now())` |
| `day(dt)` | Extract day | `day(now())` |
| `len(s)` | String length | `len($current.name)` |
| `SEQUENCE(prefix, width)` | Auto-increment sequence | `SEQUENCE('INV', 6)` → `INV-000001` |

## Actions

Actions define buttons that users can click on list rows or the list header.

```json
{
  "logic": {
    "actions": [
      {
        "id": "submit_for_review",
        "label": "Submit for Review",
        "type": "row",
        "backend": "set_status",
        "icon": "Send",
        "condition": { "field": "status", "eq": "Draft" },
        "post_actions": [
          {
            "type": "dispatch_service_event",
            "event_type": "workflow_notify",
            "payload": { "message": "Record {name} submitted for review" }
          }
        ]
      },
      {
        "id": "reorder",
        "label": "Reorder",
        "type": "reorder",
        "backend": "reorder_by_priority"
      }
    ]
  }
}
```

| Property | Description |
|----------|-------------|
| `id` | Unique action identifier |
| `label` | Button label |
| `type` | `row` (per-record), `list` (header button), `reorder`, `refresh_computed` |
| `backend` | Handler name registered in ActionHandler registry |
| `icon` | Lucide icon name |
| `condition` | Show button only when condition matches current row |
| `post_actions` | Side effects to run after the action completes |

## Constraints

Constraints restrict data operations based on record state.

```json
{
  "logic": {
    "constraints": [
      {
        "rule": "lock_when_status",
        "value": ["Approved", "Closed"]
      }
    ]
  }
}
```

When `lock_when_status` is set, records in the listed statuses cannot be edited or deleted.

## Filters

Filters define predefined data views.

```json
{
  "logic": {
    "filters": [
      { "id": "active", "label": "Active Only", "field": "status", "eq": "Active" },
      { "id": "mine", "label": "My Records", "field": "creator_id", "eq": "$current_user_id" }
    ]
  }
}
```
