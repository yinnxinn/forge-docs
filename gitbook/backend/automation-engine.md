# Automation Engine

The Automation Engine (`backend/app/services/automation_engine.py`) executes event-driven rules defined in DSL `logic.automation`.

## Overview

Automation enables cross-app data flows without custom code. When a record event occurs (create, update, workflow transition), the engine checks for matching rules and executes their actions.

## Architecture

```
Event (record_created / record_updated / workflow_transition)
        │
        ▼
AutomationEngine.fire(db, rules, context)
        │
        ├─ Filter rules by trigger type and trigger_meta
        ├─ Evaluate conditions (if any)
        │
        └─ For each matching rule:
            ├─ Resolve field_mapping values
            │   ├─ source: $current.field
            │   ├─ literal: fixed value ($now, $creator)
            │   └─ expr: FormulaEngine.evaluate(...)
            │
            └─ Execute action:
                ├─ create_record → DataService.create_row(target_app)
                ├─ update_record → DataService.update_row(target_app)
                ├─ update_current → DataService.update_row(current_app)
                └─ notify → dispatch_service_event
```

## AutomationContext

The context object passed to the engine:

```python
@dataclass
class AutomationContext:
    tenant_id: str
    app_id: str
    table_name: str
    record_id: int
    record_data: dict
    current_user_id: int
    current_user_name: str
    trigger_type: str          # "record_created", "record_updated", "workflow_transition"
    trigger_meta: dict = {}    # e.g., {"transition_id": "approve"} or {"changed_fields": [...]}
```

## Rule Matching

A rule fires when:
1. `rule.trigger` matches `context.trigger_type`
2. `rule.trigger_meta` matches `context.trigger_meta` (if specified)
3. `rule.condition` evaluates to true (if specified)

## Field Mapping Resolution

| Mode | Source | Example |
|------|--------|---------|
| `source` | `$current.field` → `context.record_data[field]` | `"source": "$current.customer_name"` |
| `literal` | Fixed value | `"value": "Pending"` |
| `literal` + `$now` | Current timestamp | `"value": "$now"` |
| `literal` + `$creator` | Current user ID | `"value": "$creator"` |
| `expr` | Formula evaluation | `"value": "IF($current.amount > 10000, 'high', 'normal')"` |

## Cascade Depth Control

To prevent infinite loops (A→B→C→A), the engine tracks recursion depth:

```python
MAX_CASCADE_DEPTH = 3

async def fire(self, db, rules, context, _depth=0):
    if _depth >= MAX_CASCADE_DEPTH:
        logger.warning("Automation cascade depth limit reached")
        return []
    # ... execute actions ...
    # When an action creates/updates a record, it can trigger
    # another fire() call with _depth + 1
```

## Side Effects Tracking

The `fire()` method returns a list of side effects for transparency:

```python
side_effects = await automation_engine.fire(db, rules, ctx)
# Returns: [
#   {"action": "create_record", "target_app": "warehouse_tasks", "record_id": 456},
#   {"action": "notify", "message": "Order approved"},
# ]
```

These side effects are returned in the API response so the frontend can display feedback.

## Usage Example

In the Data API, after creating a record:

```python
dsl = await _load_app_dsl(db, app_id)
if dsl:
    rules = dsl.get_automation_rules()
    if rules:
        ctx = AutomationContext(
            tenant_id=tid, app_id=app_id, table_name=table_name,
            record_id=row.id, record_data=dict(row.data or {}),
            current_user_id=user.id,
            current_user_name=user.display_name or user.username,
            trigger_type="record_created",
        )
        await automation_engine.fire(db, rules, ctx)
```
