# DSL Renderer

The DSL Renderer is the frontend engine that transforms Lingtarn DSL into interactive React components.

## Registry Pattern

The core of the rendering engine is the **Registry** (`components/renderer/Registry.tsx`), which maps DSL field types to React components.

### Type → Component Mapping

| DSL Type | React Component | Renders As |
|----------|----------------|-----------|
| `text` | Text input | `<Input type="text" />` |
| `number` | Number input | `<Input type="number" />` |
| `select` | Dropdown | `<Select>` with options |
| `date` | Date picker | Date picker component |
| `datetime` | DateTime picker | DateTime picker component |
| `boolean` | Checkbox | `<Checkbox />` or `<Switch />` |
| `LinkToApp` | Lookup select | Searchable select with app data |
| `relation` | Relation select | Related record selector |
| `tenant_users_multi` | Multi-user picker | User search with multi-select |
| `tenant_user_single` | User picker | Single user selector |
| `config_ref` | Config lookup | Configuration reference picker |

### How Registration Works

```typescript
const FIELD_RENDERERS: Record<string, React.ComponentType<FieldProps>> = {
  text: TextField,
  number: NumberField,
  select: SelectField,
  date: DateField,
  datetime: DateTimeField,
  boolean: BooleanField,
  LinkToApp: LinkToAppField,
  relation: RelationField,
  tenant_users_multi: TenantUsersMultiField,
  tenant_user_single: TenantUserSingleField,
  config_ref: ConfigRefField,
};
```

## DSL Parsing

The `lib/dsl.ts` module provides helpers for extracting DSL sections:

```typescript
import { getSchemaRecords, getLayoutList, getLayoutForm } from '@/lib/dsl';

const dsl: LingtarnDSL = await fetchAppDsl(appId);
const fields = getSchemaRecords(dsl);   // SchemaRecord[]
const listCols = getLayoutList(dsl);    // string[] or ColumnConfig[]
const formRows = getLayoutForm(dsl);    // string[][]
```

### Alias Support

The DSL parser supports both standard and aliased keys (for Forge/LLM compatibility):

- `schema` or `schema_` → `getSchemaRecords()`
- `layout` or `layout_` → `getLayoutList()`, `getLayoutForm()`

## Rendering Flow

```
LingtarnDSL (JSON)
    │
    ├─ schema.records → Field definitions
    │       │
    │       ▼
    │   Registry.getComponent(field.type)
    │       │
    │       ▼
    │   React Component (TextField, SelectField, ...)
    │
    ├─ layout.list → ListCanvas
    │       │
    │       ▼
    │   Table with column headers and cell renderers
    │   Action buttons from logic.actions
    │
    ├─ layout.form → FormCanvas
    │       │
    │       ▼
    │   Grid form with rows of field components
    │   Save/Cancel buttons
    │
    └─ logic.workflow → WorkflowCanvas
            │
            ▼
        State badge + transition buttons
```

## Extending the Registry

To add a new field type:

1. Create a new field component in `components/renderer/`:

```typescript
interface FieldProps {
  field: SchemaRecord;
  value: any;
  onChange: (value: any) => void;
  readonly?: boolean;
}

export function MyCustomField({ field, value, onChange, readonly }: FieldProps) {
  return (
    <div>
      <label>{field.label}</label>
      <input value={value} onChange={e => onChange(e.target.value)} disabled={readonly} />
    </div>
  );
}
```

2. Register it in `Registry.tsx`:

```typescript
FIELD_RENDERERS['my_custom_type'] = MyCustomField;
```

3. Use it in DSL:

```json
{ "id": "custom_field", "label": "Custom", "type": "my_custom_type" }
```
