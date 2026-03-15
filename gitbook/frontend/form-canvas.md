# Form Canvas

The `FormCanvas` component (`components/renderer/FormCanvas.tsx`) dynamically renders form dialogs from DSL `layout.form` and `schema.records`.

## How It Works

1. Receives DSL schema records and form layout
2. For each row in `layout.form`, creates a grid row
3. For each field in the row, looks up the field definition from schema
4. Uses the Registry to get the appropriate component for the field type
5. Manages form state and validation
6. Submits data to the Data API on save

## Layout Modes

### Grid Layout

The most common mode — fields are arranged in a 2D grid:

```json
{
  "layout": {
    "form": [
      ["name", "email"],      // Row 1: 2 columns
      ["address"],             // Row 2: 1 column (full width)
      ["city", "state", "zip"] // Row 3: 3 columns
    ]
  }
}
```

Each row distributes available width equally among its fields.

### Tabbed Layout

For complex forms with many fields:

```json
{
  "layout": {
    "form": {
      "tabs": [
        {
          "label": "Basic Info",
          "fields": [["name", "email"], ["phone"]]
        },
        {
          "label": "Address",
          "fields": [["address"], ["city", "state"]]
        }
      ]
    }
  }
}
```

Renders as a tabbed interface where each tab contains its own grid layout.

### Inline Lists

For parent-child relationships:

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

The `__inline_list__` prefix renders the sub-schema as an editable table within the form.

## Form State Management

FormCanvas manages an internal `formData` state (key-value pairs). On field change:

```typescript
const handleFieldChange = (fieldId: string, value: any) => {
  setFormData(prev => ({ ...prev, [fieldId]: value }));
};
```

## Validation

On submit, FormCanvas checks:
- `required` fields have values
- Field type constraints (number fields have numeric values, etc.)
- Custom validators from `logic.validators` (if provided)

Invalid fields are highlighted with error messages.

## Create vs Edit Mode

- **Create**: FormCanvas opens with empty fields (or defaults from schema)
- **Edit**: FormCanvas pre-fills with existing record data

The form detects mode based on whether a `recordId` prop is provided.

## Read-Only Mode

When used in `DslPreviewCard` (AI Forge), FormCanvas renders in read-only mode with all fields disabled, serving as a preview of the DSL layout.
