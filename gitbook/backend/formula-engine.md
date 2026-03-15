# Formula Engine

The Formula Engine (`backend/app/services/formula_engine.py`) evaluates DSL computation expressions to produce computed field values.

## Overview

When a record is created or updated, the backend runs `logic.computations` through the Formula Engine. Each computation specifies:

- **target**: the field to set
- **formula**: the expression to evaluate
- **trigger**: when to run (`create`, `update`, or `always`)

## Architecture

```
FormulaContext (record data + user info)
        │
        ▼
FormulaEngine.evaluate(formula, context)
        │
        ├─ Parse expression
        ├─ Resolve $current.field references
        ├─ Execute functions (IF, add, concat, ...)
        └─ Return computed value
```

## Supported Functions

### Conditional

| Function | Signature | Example |
|----------|-----------|---------|
| `IF` | `IF(condition, then_value, else_value)` | `IF($current.amount > 1000, 'VIP', 'Standard')` |

### Arithmetic

| Function | Signature | Example |
|----------|-----------|---------|
| `add` | `add(a, b)` | `add($current.price, $current.tax)` |
| `sub` | `sub(a, b)` | `sub($current.total, $current.discount)` |
| `mul` | `mul(a, b)` | `mul($current.qty, $current.unit_price)` |
| `div` | `div(a, b)` | `div($current.total, $current.count)` |
| `round` | `round(n, decimals)` | `round($current.avg, 2)` |
| `abs` | `abs(n)` | `abs($current.difference)` |

### String

| Function | Signature | Example |
|----------|-----------|---------|
| `concat` | `concat(a, b, ...)` | `concat($current.first, ' ', $current.last)` |
| `coalesce` | `coalesce(a, b, ...)` | `coalesce($current.nickname, $current.name)` |
| `len` | `len(s)` | `len($current.description)` |

### Date/Time

| Function | Signature | Example |
|----------|-----------|---------|
| `now()` | `now()` | Current ISO timestamp |
| `date_of` | `date_of(dt)` | Extract date part |
| `time_of_day` | `time_of_day(dt)` | Extract time part |
| `year` | `year(dt)` | Extract year |
| `month` | `month(dt)` | Extract month |
| `day` | `day(dt)` | Extract day |

### Sequence

| Function | Signature | Example |
|----------|-----------|---------|
| `SEQUENCE` | `SEQUENCE(prefix, width)` | `SEQUENCE('SO', 6)` → `SO-000042` |

The `SEQUENCE` function is async and generates auto-incrementing numbers scoped to the app and tenant.

## Context Variables

| Variable | Description |
|----------|-------------|
| `$current.field_name` | Value of a field in the current record |
| `$current.creator_display_name` | Display name of the record creator |
| `$current.creator_id` | ID of the record creator |

## Usage in DSL

```json
{
  "logic": {
    "computations": [
      {
        "target": "order_total",
        "formula": "mul($current.quantity, $current.unit_price)",
        "trigger": "always"
      },
      {
        "target": "order_no",
        "formula": "SEQUENCE('ORD', 6)",
        "trigger": "create"
      },
      {
        "target": "greeting",
        "formula": "concat('Hello, ', coalesce($current.nickname, $current.name), '!')",
        "trigger": "create"
      }
    ]
  }
}
```

## Extending the Formula Engine

To add a new function:

1. Add the function implementation to `FormulaEngine` in `formula_engine.py`
2. Register it in the function dispatch table
3. Document it in the DSL reference

The engine is designed to be easily extensible. Functions receive the formula context and return a computed value.
