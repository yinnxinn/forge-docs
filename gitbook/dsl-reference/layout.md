# layout

The `layout` section controls how fields are displayed in the list view and form view.

## Structure

```json
{
  "layout": {
    "list": [ ... ],
    "form": [ ... ]
  }
}
```

## List Layout

The `list` array defines which columns appear in the table view.

```json
{
  "layout": {
    "list": ["name", "email", "status", "created_at"]
  }
}
```

Each entry can be a field `id` (string) or a column config object:

```json
{
  "layout": {
    "list": [
      "name",
      { "id": "amount", "label": "Total Amount", "width": 120 },
      "status"
    ]
  }
}
```

## Form Layout

The `form` array defines the form grid layout. Each sub-array is a row, and each element is a field:

```json
{
  "layout": {
    "form": [
      ["name", "email"],        // Row 1: two fields side-by-side
      ["address"],              // Row 2: one field full-width
      ["city", "state", "zip"]  // Row 3: three fields
    ]
  }
}
```

### Tabbed Forms

For complex forms, use tabs:

```json
{
  "layout": {
    "form": {
      "tabs": [
        {
          "label": "Basic Info",
          "fields": [
            ["name", "email"],
            ["phone"]
          ]
        },
        {
          "label": "Address",
          "fields": [
            ["address"],
            ["city", "state", "zip"]
          ]
        }
      ]
    }
  }
}
```

### Inline Lists (Sub-Schema)

To embed a sub-schema as an inline editable list:

```json
{
  "layout": {
    "form": [
      ["order_no", "customer"],
      ["__inline_list__items"]
    ]
  }
}
```

The `__inline_list__` prefix followed by the sub-schema name renders an editable table within the form.

## Responsive Behavior

The form grid automatically adjusts:
- On desktop: renders the specified column layout
- On mobile: collapses to single-column
- Field widths are distributed equally within each row

## Tips

- Put the most important fields first in both `list` and `form`
- Use 2-3 fields per row for a clean form layout
- Keep `list` to 4-6 columns for readability
- Use tabs when a form has more than 8-10 fields
