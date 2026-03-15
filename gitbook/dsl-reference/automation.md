# automation

The `automation` section defines event-driven rules that fire when records are created, updated, or undergo workflow transitions.

## Structure

```json
{
  "logic": {
    "automation": [
      {
        "id": "rule_id",
        "trigger": "record_created | record_updated | workflow_transition",
        "trigger_meta": { ... },
        "condition": { ... },
        "actions": [ ... ]
      }
    ]
  }
}
```

## Triggers

| Trigger | Fires When | trigger_meta |
|---------|-----------|--------------|
| `record_created` | A new record is created | (none) |
| `record_updated` | An existing record is updated | `{ "changed_fields": ["field1"] }` (optional) |
| `workflow_transition` | A workflow transition completes | `{ "transition_id": "approve" }` |

## Conditions

Optional conditions to further filter when the rule fires:

```json
{
  "condition": {
    "field": "amount",
    "op": "gt",
    "value": 10000
  }
}
```

## Actions

Each rule can have one or more actions:

### `create_record`

Create a new record in another app.

```json
{
  "type": "create_record",
  "target_app_id": "warehouse_tasks",
  "target_model": "main",
  "field_mapping": [
    { "target": "source_order_id", "source": "$current.id" },
    { "target": "product_id", "source": "$current.product_id" },
    { "target": "quantity", "source": "$current.quantity" },
    { "target": "priority", "mode": "expr", "value": "IF($current.amount > 10000, 'high', 'normal')" },
    { "target": "created_at", "mode": "literal", "value": "$now" }
  ]
}
```

### `update_record`

Update existing records in another app.

```json
{
  "type": "update_record",
  "target_app_id": "inventory",
  "target_model": "main",
  "match": { "product_id": "$current.product_id" },
  "field_mapping": [
    { "target": "reserved_qty", "mode": "expr", "value": "add($target.reserved_qty, $current.quantity)" }
  ]
}
```

### `update_current`

Update fields on the current record.

```json
{
  "type": "update_current",
  "field_mapping": [
    { "target": "processed_at", "mode": "literal", "value": "$now" },
    { "target": "processed_by", "source": "$creator" }
  ]
}
```

### `notify`

Send a notification (via service runtime).

```json
{
  "type": "notify",
  "message": "Order {order_no} has been approved",
  "recipients": {
    "from_row_fields": ["assignee_user_ids"],
    "static_user_ids": [1, 2]
  }
}
```

## Field Mapping

Field mapping defines how values flow from the source record to the target.

| Mode | Description | Example |
|------|-------------|---------|
| `source` (default) | Copy from current record | `{ "target": "name", "source": "$current.customer_name" }` |
| `literal` | Fixed value | `{ "target": "status", "mode": "literal", "value": "Pending" }` |
| `expr` | Formula expression | `{ "target": "total", "mode": "expr", "value": "mul($current.price, $current.qty)" }` |
| `lookup` | (planned) Query another app | `{ "target": "email", "mode": "lookup", "app_id": "contacts", "field": "email" }` |

### Special Variables

| Variable | Description |
|----------|-------------|
| `$current.field` | Field value from the triggering record |
| `$current.id` | ID of the triggering record |
| `$creator` | User ID who triggered the event |
| `$now` | Current timestamp (ISO format) |
| `$target.field` | Field value from the target record (in `update_record`) |

## Complete Example: Sales → Warehouse Automation

```json
{
  "logic": {
    "automation": [
      {
        "id": "create_warehouse_task_on_approval",
        "trigger": "workflow_transition",
        "trigger_meta": { "transition_id": "approve" },
        "actions": [
          {
            "type": "create_record",
            "target_app_id": "warehouse_tasks",
            "target_model": "main",
            "field_mapping": [
              { "target": "source_order_id", "source": "$current.id" },
              { "target": "customer", "source": "$current.customer" },
              { "target": "product_id", "source": "$current.product_id" },
              { "target": "quantity", "source": "$current.quantity" },
              { "target": "priority", "mode": "expr",
                "value": "IF($current.amount > 10000, 'high', 'normal')" },
              { "target": "status", "mode": "literal", "value": "Pending" },
              { "target": "created_at", "mode": "literal", "value": "$now" }
            ]
          }
        ]
      },
      {
        "id": "update_inventory_on_ship",
        "trigger": "workflow_transition",
        "trigger_meta": { "transition_id": "ship" },
        "actions": [
          {
            "type": "update_record",
            "target_app_id": "inventory",
            "target_model": "main",
            "match": { "product_id": "$current.product_id" },
            "field_mapping": [
              { "target": "on_hand", "mode": "expr",
                "value": "sub($target.on_hand, $current.quantity)" }
            ]
          }
        ]
      }
    ]
  }
}
```

## Cascade Depth Limit

To prevent infinite automation loops (A creates B, B creates C, C creates A...), the automation engine enforces a configurable cascade depth limit (default: 3). Beyond this limit, automation rules are silently skipped.
