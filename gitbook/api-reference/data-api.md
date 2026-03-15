# Data API

The Data API provides unified CRUD operations for all DSL-defined apps. A single set of endpoints handles every app — no app-specific API code needed.

## Base URL

```
/api/v1/data/{app_id}/{model}
```

- `app_id` — The registered app identifier (e.g., `"crm"`, `"sales"`)
- `model` — The table within the app (usually `"main"`, or a sub-schema name)

## List Records

```
GET /api/v1/data/{app_id}/{model}?tenant_id=default&limit=50&offset=0
Authorization: Bearer {token}
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `tenant_id` | `string` | `"default"` | Tenant ID (non-superusers use own tenant) |
| `department_id` | `int?` | `null` | Department scope |
| `limit` | `int` | `50` | Page size (1-200) |
| `offset` | `int` | `0` | Pagination offset |
| `record_type` | `string?` | `null` | Filter by `data.record_type` |

**Response:**
```json
{
  "total": 42,
  "items": [
    {
      "id": 1,
      "data": {
        "name": "Acme Corp",
        "status": "Active",
        "amount": 5000
      },
      "created_at": "2026-03-14T10:00:00Z",
      "updated_at": "2026-03-14T10:00:00Z"
    }
  ]
}
```

## Get Single Record

```
GET /api/v1/data/{app_id}/{model}/{row_id}
Authorization: Bearer {token}
```

**Response:**
```json
{
  "id": 1,
  "data": { "name": "Acme Corp", "status": "Active" },
  "created_at": "2026-03-14T10:00:00Z",
  "updated_at": "2026-03-14T10:00:00Z"
}
```

## Create Record

```
POST /api/v1/data/{app_id}/{model}
Authorization: Bearer {token}
Content-Type: application/json

{
  "data": {
    "name": "New Customer",
    "email": "customer@example.com",
    "status": "Lead"
  }
}
```

Before saving, the backend:
1. Runs `logic.computations` (Formula Engine)
2. Fires `logic.automation` rules (`record_created`)

## Update Record

```
PATCH /api/v1/data/{app_id}/{model}/{row_id}
Authorization: Bearer {token}
Content-Type: application/json

{
  "data": {
    "status": "Customer",
    "notes": "Converted from lead"
  }
}
```

Before saving, the backend:
1. Checks `logic.constraints` (e.g., `lock_when_status`)
2. Runs `logic.computations`
3. Fires `logic.automation` rules (`record_updated`)

## Delete Record

```
DELETE /api/v1/data/{app_id}/{model}/{row_id}
Authorization: Bearer {token}
```

Returns `204 No Content` on success.

Checks `logic.constraints` before deleting (e.g., locked records cannot be deleted).

## Execute Action

```
POST /api/v1/data/{app_id}/{model}/action
Authorization: Bearer {token}
Content-Type: application/json

{
  "action_id": "approve",
  "row_id": 42
}
```

**Request Body:**

| Field | Type | Description |
|-------|------|-------------|
| `action_id` | `string` | Action ID from `logic.actions` |
| `row_id` | `int?` | Target row (for row actions) |
| `order` | `int[]?` | Row order (for reorder actions) |
| `row_ids` | `int[]?` | Multiple rows (for bulk actions) |

**Response:**
```json
{
  "ok": true,
  "message": "Action completed",
  "updated_row": { "id": 42, "data": {...}, ... },
  "updated_count": 1
}
```

## Workflow Endpoints

### Get Workflow State

```
GET /api/v1/data/{app_id}/{model}/{record_id}/workflow
Authorization: Bearer {token}
```

### Execute Transition

```
POST /api/v1/data/{app_id}/{model}/{record_id}/workflow/{transition_id}?comment=...
Authorization: Bearer {token}
```

See [Workflow Engine](../backend/workflow-engine.md) for details.

## Error Responses

| Status | Description |
|--------|-------------|
| `400` | Validation error, constraint violation, workflow error |
| `401` | Missing or invalid token |
| `403` | Insufficient permissions |
| `404` | App, table, or record not found |
| `501` | Action handler not implemented |
