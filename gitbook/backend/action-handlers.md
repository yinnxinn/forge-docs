# Action Handlers

The Action Handler system (`backend/app/services/action_handlers.py`) provides a registry pattern for custom actions that can be triggered from the frontend via DSL configuration.

## Overview

While most CRUD operations are handled generically by the Data Service, some operations require custom logic: reordering rows, bulk operations, status transitions, or cross-app data flows.

Action Handlers bridge this gap with a simple registry pattern.

## Registering a Handler

```python
from app.services.action_handlers import register_action

@register_action("my_custom_action")
async def my_custom_action(
    session: AsyncSession,
    app_id: str,
    table_name: str,
    tenant_id: str,
    *,
    row_id: int | None = None,
    order: list[int] | None = None,
    row_ids: list[int] | None = None,
    **kwargs,
) -> dict:
    """
    Custom action implementation.
    Returns: {"ok": bool, "message": str, "updated_row": DataRowOut | None}
    """
    # Your logic here
    return {"ok": True, "message": "Action completed"}
```

## Handler Signature

All handlers follow the same signature:

```python
async def handler(
    session: AsyncSession,      # Database session
    app_id: str,                # Current app ID
    table_name: str,            # Resolved table name
    tenant_id: str,             # Current tenant ID
    *,
    row_id: int | None,         # Target row (for row actions)
    order: list[int] | None,    # Row order (for reorder actions)
    row_ids: list[int] | None,  # Multiple rows (for bulk actions)
    **kwargs,
) -> dict:
```

## Return Format

```python
{
    "ok": True,                          # Success/failure
    "message": "Human-readable result",  # Display message
    "updated_row": DataRowOut | None,    # Updated row data (optional)
    "updated_count": int | None,         # Number of affected rows (optional)
}
```

## Built-in Handlers

### `reorder_by_priority`

Reorders rows by setting `priority_index` based on the provided `order` array.

```json
{ "id": "reorder", "type": "reorder", "backend": "reorder_by_priority" }
```

### `pin_to_top_priority`

Sets a row's `priority_index` to the maximum + 1, bringing it to the top.

```json
{ "id": "pin_top", "type": "row", "backend": "pin_to_top_priority", "label": "Pin to Top" }
```

### `set_status`

Generic status transition handler.

```json
{ "id": "confirm", "type": "row", "backend": "set_status", "label": "Confirm",
  "condition": { "field": "status", "eq": "Draft" } }
```

## DSL Configuration

Actions are configured in `logic.actions`:

```json
{
  "logic": {
    "actions": [
      {
        "id": "approve",
        "label": "Approve",
        "type": "row",
        "backend": "my_custom_action",
        "icon": "CheckCircle",
        "condition": { "field": "status", "eq": "pending" },
        "post_actions": [
          {
            "type": "dispatch_service_event",
            "event_type": "workflow_notify",
            "payload": { "message": "Record approved" }
          }
        ]
      }
    ]
  }
}
```

## Action Conditions

The `condition` field controls when an action button is visible:

```python
def check_action_condition(row_data: dict, condition: dict | None) -> bool:
    if not condition:
        return True
    field = condition.get("field")
    val = row_data.get(field)
    if "in" in condition:
        return val in condition["in"]
    return val == condition.get("eq")
```

## Permission Check

Actions are permission-controlled:
- `reorder` and `refresh_computed` types: checked against `U` (Update) permission
- All other types: checked against the action's `id` as a custom permission

This means you can grant specific action permissions to specific roles via the RBAC system.

## Post-Actions

After the main handler executes, `post_actions` can dispatch service events (e.g., notifications):

```json
{
  "post_actions": [
    {
      "type": "dispatch_service_event",
      "event_type": "workflow_notify",
      "payload": {
        "operation": "order_approved",
        "message": "Order {order_no} has been approved"
      },
      "recipients": {
        "from_row_fields": ["notify_user_ids"],
        "from_config_table": {
          "app_id": "notification_config",
          "model": "main",
          "match": { "event_type": "order_approval" },
          "user_fields": ["recipient_ids"]
        }
      }
    }
  ]
}
```

Template variables (`{field_name}`) in the payload are replaced with actual values from the current row.
