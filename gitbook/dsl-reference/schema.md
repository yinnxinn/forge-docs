# schema

The `schema` section defines the data model — what fields the app has, their types, and validation rules.

## Structure

```json
{
  "schema": {
    "records": [
      { "id": "field_id", "label": "Display Label", "type": "text", ... }
    ],
    "sub_schemas": {
      "child_table_name": {
        "records": [ ... ]
      }
    }
  }
}
```

## Field Definition

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | `string` | Yes | Unique field identifier (used in data storage) |
| `label` | `string` | Yes | Display label in UI |
| `type` | `string` | Yes | Field type (see below) |
| `required` | `boolean` | No | Whether field is required on create |
| `readonly` | `boolean` | No | Whether field is read-only in forms |
| `options` | `string[]` | Conditional | Options for `select` type |
| `default` | `any` | No | Default value on create |
| `placeholder` | `string` | No | Placeholder text in input |
| `semantic_tag` | `string` | No | Semantic hint (e.g., "order_no", "customer_name") |
| `source_config` | `object` | No | Configuration for relation/lookup fields |
| `ui_config` | `object` | No | UI rendering hints (width, color, format) |

## Field Types

| Type | Description | Renders As |
|------|-------------|-----------|
| `text` | Free-form text | Text input |
| `number` | Numeric value | Number input |
| `select` | Single choice from options | Dropdown select |
| `date` | Date only | Date picker |
| `datetime` | Date and time | DateTime picker |
| `boolean` | True/false | Checkbox / toggle |
| `LinkToApp` | Reference to another app's record | Lookup select |
| `relation` | Generic relation to another table | Relation select |
| `tenant_users_multi` | Multiple user selection within tenant | Multi-user picker |
| `tenant_user_single` | Single user selection within tenant | User picker |
| `config_ref` | Reference to a configuration table | Config lookup |

## Examples

### Basic Fields

```json
{
  "schema": {
    "records": [
      { "id": "name", "label": "Name", "type": "text", "required": true },
      { "id": "email", "label": "Email", "type": "text", "placeholder": "user@example.com" },
      { "id": "age", "label": "Age", "type": "number" },
      { "id": "active", "label": "Active", "type": "boolean", "default": true },
      { "id": "joined_at", "label": "Joined", "type": "date" }
    ]
  }
}
```

### Select Field

```json
{
  "id": "priority",
  "label": "Priority",
  "type": "select",
  "options": ["Low", "Medium", "High", "Critical"],
  "default": "Medium"
}
```

### Relation Field (LinkToApp)

```json
{
  "id": "customer_id",
  "label": "Customer",
  "type": "LinkToApp",
  "source_config": {
    "app_id": "customers",
    "model": "main",
    "display_field": "name"
  }
}
```

### User Selection

```json
{
  "id": "assignees",
  "label": "Assignees",
  "type": "tenant_users_multi"
}
```

## Sub-Schemas

For apps with parent-child relationships (e.g., order → line items):

```json
{
  "schema": {
    "records": [
      { "id": "order_no", "label": "Order No", "type": "text" },
      { "id": "customer", "label": "Customer", "type": "text" }
    ],
    "sub_schemas": {
      "items": {
        "records": [
          { "id": "product", "label": "Product", "type": "text" },
          { "id": "qty", "label": "Quantity", "type": "number" },
          { "id": "price", "label": "Unit Price", "type": "number" }
        ]
      }
    }
  }
}
```

Sub-schemas are stored as separate tables (e.g., `myapp_items`) and rendered as inline editable lists within the parent form.
