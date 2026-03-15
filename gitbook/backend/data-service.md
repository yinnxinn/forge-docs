# Data Service

The Data Service (`backend/app/services/data_service.py`) is the core data access layer. It provides unified CRUD operations for all DSL-defined apps.

## Key Functions

### `resolve_table_name(db, app_id, model) → str | None`

Resolves an `app_id` + `model` pair to a physical table name by querying `lt_sys_table_registry`.

- `model = "main"` → returns the app's primary table
- `model = "items"` → returns the sub-table (e.g., `myapp_items`)

### `get_table_registry(db, app_id, table_name) → LtSysTableRegistry | None`

Returns the table registry entry, including `table_logic` (JSON containing validators, computations, automation config).

### `list_rows(db, app_id, table_name, **filters) → (list[LtAppData], int)`

Lists rows with pagination and filtering:

```python
rows, total = await data_service.list_rows(
    db,
    app_id="crm",
    table_name="crm_main",
    tenant_id="acme",
    department_id=5,
    limit=50,
    offset=0,
    filter_data={"record_type": "lead"},
)
```

**Filters:**
- `tenant_id` — Required, tenant isolation
- `department_id` — Optional, department scoping
- `department_ids` — Optional, multi-department scoping (list)
- `filter_data` — Optional, dict of field→value equality filters
- `limit` / `offset` — Pagination

### `get_row(db, app_id, table_name, row_id, **filters) → LtAppData | None`

Fetches a single row by ID, with tenant/department scoping.

### `create_row(db, app_id, table_name, data, **kwargs) → LtAppData`

Creates a new row:

```python
row = await data_service.create_row(
    db,
    app_id="crm",
    table_name="crm_main",
    data={"name": "Acme Corp", "status": "Lead"},
    tenant_id="acme",
    department_id=5,
    creator_id=7,
)
```

### `update_row(db, app_id, table_name, row_id, data, **kwargs) → LtAppData | None`

Updates an existing row's data (JSONB merge):

```python
updated = await data_service.update_row(
    db,
    app_id="crm",
    table_name="crm_main",
    row_id=123,
    data={"status": "Customer", "notes": "Converted from lead"},
    tenant_id="acme",
)
```

### `delete_row(db, app_id, table_name, row_id, **kwargs) → bool`

Deletes a row. Returns `True` if found and deleted.

## Department Scoping

For multi-department users, the Data Service supports two scoping modes:

1. **Single department** — `department_id=5` filters rows by `department_id = 5`
2. **Multi-department** — `department_ids=[5, 8, 12]` filters rows by `department_id IN (5, 8, 12)`

The API layer determines which mode to use based on the user's department memberships and the app's `multi_department_scope` flag.

## Table Logic

Each table in `lt_sys_table_registry` has a `table_logic` JSON column containing:

```json
{
  "validators": [...],
  "computations": [...],
  "automation": [...],
  "actions": [...],
  "constraints": [...]
}
```

This is loaded by the Data API endpoints and passed to the appropriate engines (Formula, Workflow, Automation) during CRUD operations.
