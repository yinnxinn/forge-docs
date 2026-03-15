# app_meta

The `app_meta` section defines the app's identity and display properties.

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | `string` | Yes | Display name of the app |
| `icon` | `string` | No | Icon name (Lucide icon set) |
| `description` | `string` | No | Short description shown in marketplace |
| `version` | `string` | No | Semantic version (e.g., "1.0.0") |
| `category` | `string` | No | App category for marketplace grouping |
| `multi_department_scope` | `boolean` | No | If true, multi-department users see data from all their departments |
| `standalone` | `boolean` | No | If true, renders as a standalone page (not list+form) |
| `embed_url` | `string` | No | URL to embed as iframe (for embed-type apps) |

## Example

```json
{
  "app_meta": {
    "name": "Customer Relationship Management",
    "icon": "Users",
    "description": "Track customers, leads, and deals",
    "version": "1.2.0",
    "category": "Sales"
  }
}
```

## Icon Names

Icons use the [Lucide](https://lucide.dev/icons/) icon set. Common examples:

- `Users`, `UserPlus`, `Building2`
- `ShoppingCart`, `ShoppingBag`, `Package`
- `FileText`, `ClipboardList`, `Calendar`
- `Settings`, `Wrench`, `Cog`
- `BarChart3`, `TrendingUp`, `PieChart`

## App Types

The `app_meta` determines how the app renders:

| Configuration | Rendering |
|--------------|-----------|
| Default (no special flags) | List view + Form dialog |
| `standalone: true` | Full-page form (no list) |
| `embed_url: "https://..."` | Iframe embedding |
| Plugin-type | Plugin canvas rendering |
