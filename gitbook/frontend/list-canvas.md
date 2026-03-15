# List Canvas

The `ListCanvas` component (`components/renderer/ListCanvas.tsx`) dynamically renders data tables from DSL `layout.list`, `schema.records`, and `logic.actions`.

## How It Works

1. Receives DSL definitions and data rows
2. Creates table columns from `layout.list`, enriched with field metadata from `schema.records`
3. Renders cell values using the Registry (type-aware formatting)
4. Adds action columns from `logic.actions` (row-level buttons)
5. Handles pagination, sorting, and row selection

## Column Generation

```json
{
  "layout": {
    "list": ["name", "email", "status", "created_at"]
  }
}
```

For each column ID, ListCanvas:
1. Looks up the field definition in `schema.records`
2. Gets the field type (`text`, `select`, `date`, etc.)
3. Uses the Registry to render cell values appropriately:
   - `select` → colored badge
   - `date` → formatted date string
   - `boolean` → check/cross icon
   - `tenant_users_multi` → user avatar list

## Action Buttons

Row-level actions from `logic.actions` appear as buttons in an "Actions" column:

```json
{
  "logic": {
    "actions": [
      {
        "id": "approve",
        "label": "Approve",
        "type": "row",
        "backend": "set_status",
        "icon": "CheckCircle",
        "condition": { "field": "status", "eq": "pending" }
      }
    ]
  }
}
```

ListCanvas evaluates `condition` against each row's data to determine button visibility.

## Features

### Pagination

ListCanvas implements offset-based pagination:
- `limit` and `offset` query parameters
- Page size selector
- Total count display

### Row Selection

When bulk actions are available, ListCanvas shows checkboxes for row selection.

### Drag-and-Drop Reordering

When a `reorder` action is configured, rows can be dragged to reorder:

```json
{
  "logic": {
    "actions": [
      { "id": "reorder", "type": "reorder", "backend": "reorder_by_priority" }
    ]
  }
}
```

### Inline Workflow Badge

If the app has a workflow, the status column renders as a colored badge matching the workflow state colors.

## Data Flow

```
AppDetail
    │
    ├─ GET /data/{appId}/main → rows[]
    │
    └─ ListCanvas
         ├─ Render columns from layout.list
         ├─ Render rows from data
         ├─ On row click → open FormCanvas (edit mode)
         ├─ On action click → POST /data/{appId}/main/action
         └─ On page change → re-fetch with new offset
```

## Responsive Behavior

On small screens, ListCanvas:
- Hides lower-priority columns
- Makes the table horizontally scrollable
- Collapses action buttons into a dropdown menu
