# Plugin System

The Plugin System provides a structured way to extend the platform when DSL alone isn't sufficient.

## When to Use Plugins

| Use Case | Without Plugins | With Plugins |
|----------|----------------|-------------|
| CRUD with workflow | DSL only | Not needed |
| Computed fields | DSL only | Not needed |
| External API calls | Not possible | Plugin handler |
| Scheduled tasks | Not possible | Plugin with scheduler |
| File generation (PDF, Excel) | Not possible | Plugin handler |
| Complex business logic | Action handler | Plugin with full context |

## Architecture

```
backend/app/plugins/
├── base.py           # BasePlugin abstract class
├── registry.py       # Plugin discovery and loading
├── platform_api.py   # PluginContext (safe data access)
└── {plugin_id}/      # Individual plugins
    ├── plugin.json   # Plugin manifest
    └── handler.py    # Plugin implementation
```

## Creating a Plugin

### 1. Plugin Manifest (`plugin.json`)

```json
{
  "id": "pdf_generator",
  "name": "PDF Report Generator",
  "version": "1.0.0",
  "description": "Generate PDF reports from app data",
  "author": "Your Name",
  "actions": ["generate_pdf"]
}
```

### 2. Plugin Handler (`handler.py`)

```python
from app.plugins.base import BasePlugin, PluginContext, PluginResult

class PdfGeneratorPlugin(BasePlugin):
    async def execute(self, ctx: PluginContext, action: str, params: dict) -> PluginResult:
        if action == "generate_pdf":
            rows = await ctx.list_rows(ctx.app_id, "main", limit=100)
            # Generate PDF logic here...
            return PluginResult(ok=True, message="PDF generated", data={"url": "/files/report.pdf"})
        return PluginResult(ok=False, message=f"Unknown action: {action}")
```

## PluginContext

Plugins interact with the platform through a controlled `PluginContext` that provides safe access to data without direct database access:

```python
class PluginContext:
    app_id: str
    tenant_id: str
    user_id: int

    async def list_rows(self, app_id, model, **filters) -> list[dict]
    async def get_row(self, app_id, model, row_id) -> dict | None
    async def create_row(self, app_id, model, data) -> dict
    async def update_row(self, app_id, model, row_id, data) -> dict | None
```

## PluginResult

```python
@dataclass
class PluginResult:
    ok: bool
    message: str
    data: dict | None = None
```

## Registering with DSL

Plugins can be triggered as actions in DSL:

```json
{
  "logic": {
    "actions": [
      {
        "id": "generate_report",
        "label": "Generate PDF",
        "type": "row",
        "backend": "call_plugin",
        "config": {
          "plugin_id": "pdf_generator",
          "action": "generate_pdf"
        }
      }
    ]
  }
}
```

## Frontend Rendering

Plugins can also provide custom frontend rendering via `PluginCanvas`:

```json
{
  "app_meta": {
    "name": "Dashboard",
    "render_type": "plugin",
    "plugin_id": "dashboard_renderer"
  }
}
```

The `PluginCanvas` component in the frontend loads and renders plugin-specific UI.

## Security

- Plugins run in the same process but access data only through `PluginContext`
- No direct database access — all queries go through the Data Service
- Plugin actions are permission-controlled through the standard RBAC system
- Plugin manifest declares which actions it provides
