# App Management API

The App Management API handles app registration, listing, installation, and version management.

## List Apps

```
GET /api/v1/apps
Authorization: Bearer {token}
```

Returns apps installed for the current user's tenant.

**Response:**
```json
[
  {
    "app_id": "crm",
    "name": "Customer CRM",
    "icon": "Users",
    "description": "Manage customers and deals",
    "version": "1.0.0",
    "category": "Sales"
  }
]
```

## Get App Detail

```
GET /api/v1/apps/{app_id}
Authorization: Bearer {token}
```

Returns app details including the full DSL snapshot.

## Register App (from DSL)

```
POST /api/v1/apps/register
Authorization: Bearer {token}
Content-Type: application/json

{
  "app_id": "my_new_app",
  "dsl": {
    "app_meta": { "name": "My App", "icon": "Star" },
    "schema": { "records": [...] },
    "layout": { "list": [...], "form": [...] },
    "logic": { ... }
  }
}
```

This endpoint:
1. Validates the DSL via Pydantic (`LingtarnDSL`)
2. Creates `lt_sys_app` row with `dsl_snapshot`
3. Creates `lt_sys_table_registry` entries
4. Creates `lt_sys_field_registry` entries (one per schema field)
5. Creates `lt_sys_menu` entry (sidebar)
6. Returns the registered app

## Install App (for Tenant)

```
POST /api/v1/apps/{app_id}/install
Authorization: Bearer {token}
```

Installs a marketplace app for the current tenant.

## Uninstall App

```
DELETE /api/v1/apps/{app_id}/uninstall
Authorization: Bearer {token}
```

Removes the app from the current tenant (data is preserved).

## App Version History

```
GET /api/v1/apps/{app_id}/versions
Authorization: Bearer {token}
```

Returns version history with DSL diffs.

## Marketplace

```
GET /api/v1/marketplace
Authorization: Bearer {token}
```

Lists all available apps in the marketplace (published apps from all tenants or global apps).
