# Three-Layer Data Model

Lingtarn Forge organizes data into three distinct layers, each serving a specific purpose.

## Layer 1: Metadata (The Brain)

The metadata layer stores definitions — what tables exist, what fields they have, and what logic applies.

### Key Tables

| Table | Purpose |
|-------|---------|
| `lt_sys_app` | App registry: `app_id`, `name`, `dsl_snapshot`, `version` |
| `lt_sys_table_registry` | Table definitions: `app_id`, `table_name`, `table_logic` (JSON) |
| `lt_sys_field_registry` | Field definitions: `field_id`, `field_type`, `semantic_tag`, `source_config`, `ui_config` |
| `lt_sys_menu` | Menu structure: `app_id`, `label`, `icon`, `path` |

### How It Works

When an app is registered (via AI Forge or API), the DSL is decomposed:

```
DSL.app_meta    → lt_sys_app
DSL.schema      → lt_sys_field_registry (one row per field)
DSL.logic       → lt_sys_table_registry.table_logic (JSON column)
DSL.app_meta    → lt_sys_menu (sidebar entry)
```

The metadata layer is the **source of truth** for the platform's runtime behavior.

## Layer 2: Permissions (The Guard)

The permission layer enforces who can do what, on which data.

### Key Tables

| Table | Purpose |
|-------|---------|
| `lt_sys_tenant` | Tenant (organization) registry |
| `lt_sys_user` | User accounts |
| `lt_sys_role` | Role definitions |
| `lt_sys_user_role` | User-role assignments |
| `lt_sys_permission` | Table-level permissions (C/R/U/D + custom actions) |
| `lt_sys_data_rule` | Row-level filters (e.g., "only see own records") |
| `lt_sys_department` | Department hierarchy |
| `lt_sys_user_department` | User-department assignments |
| `lt_sys_role_department` | Role-department scoping |

### Permission Model

```
Tenant
  └─ Roles
       └─ Permissions (per table: C/R/U/D + action IDs)
       └─ Data Rules (row-level filters)
  └─ Departments
       └─ Users (can belong to multiple departments)
```

**Enforcement points:**
- **API layer**: `check_permission(db, user, table_name, action)` before every operation
- **Data layer**: `tenant_id` filter on all queries; optional `department_id` scoping
- **Row-level**: Data rules like `creator_id = current_user_id` applied at query time

## Layer 3: Business Data (The Storage)

All app data is stored in a single table with JSONB:

### `lt_app_data`

| Column | Type | Description |
|--------|------|-------------|
| `id` | `SERIAL` | Primary key |
| `app_id` | `VARCHAR` | Which app this row belongs to |
| `table_name` | `VARCHAR` | Which table within the app |
| `tenant_id` | `VARCHAR` | Tenant isolation |
| `department_id` | `INTEGER` | Department scoping (optional) |
| `creator_id` | `INTEGER` | Who created the row |
| `data` | `JSONB` | All field values |
| `created_at` | `TIMESTAMP` | Creation time |
| `updated_at` | `TIMESTAMP` | Last update time |

### Why JSONB?

1. **No migrations** — Adding fields to an app doesn't require ALTER TABLE
2. **Flexible schema** — Each app can have different fields
3. **Fast enough** — PostgreSQL JSONB supports indexing and efficient queries
4. **Simple** — One table for all business data, filtered by `app_id` + `table_name` + `tenant_id`

### Data Access Flow

```
API Request: GET /data/crm/main?tenant_id=acme
        │
        ▼
DataService.resolve_table_name(db, "crm", "main")
        │  → looks up lt_sys_table_registry
        │  → returns "crm_main"
        ▼
DataService.list_rows(db, "crm", "crm_main", tenant_id="acme")
        │  → SELECT * FROM lt_app_data
        │    WHERE app_id='crm' AND table_name='crm_main'
        │    AND tenant_id='acme'
        ▼
Return rows as DataRowOut[]
```

## How the Three Layers Interact

```
┌─────────────────────────────┐
│  1. Metadata                │
│  "What tables and fields    │  ← DSL registration
│   exist? What logic applies?"│
└──────────┬──────────────────┘
           │ defines
           ▼
┌─────────────────────────────┐
│  2. Permissions             │
│  "Who can access what?"     │  ← Admin configuration
└──────────┬──────────────────┘
           │ guards
           ▼
┌─────────────────────────────┐
│  3. Business Data           │
│  "The actual records"       │  ← User operations
└─────────────────────────────┘
```

This separation means:
- **Metadata** can be changed without touching data
- **Permissions** can be reconfigured without changing the app definition
- **Data** is always accessed through the metadata and permission layers
