# Extending the Platform

Lingtarn Forge is designed to be extended at multiple levels. This guide covers how to add custom functionality.

## Extension Points (by Preference)

| Level | Approach | Complexity | When to Use |
|-------|----------|-----------|-------------|
| 1 | **DSL only** | Low | Standard CRUD, workflows, automation |
| 2 | **Action Handler** | Medium | Custom button logic, cross-app flows |
| 3 | **Plugin** | Medium | External API integration, scheduled tasks |
| 4 | **Core modification** | High | New rendering types, new engine features |

Always prefer the lowest level that meets your needs.

## Level 1: DSL Configuration

Most business requirements can be met with DSL alone:

- Define fields in `schema.records`
- Configure list/form in `layout`
- Add computed fields in `logic.computations`
- Set up approval flows in `logic.workflow`
- Create cross-app automation in `logic.automation`
- Add action buttons in `logic.actions`
- Enforce constraints in `logic.constraints`

See the [DSL Reference](../dsl-reference/overview.md) for full documentation.

## Level 2: Custom Action Handlers

For logic that can't be expressed in DSL computations or automation:

```python
# backend/app/services/my_handlers.py

from app.services.action_handlers import register_action
from app.services import data_service

@register_action("calculate_totals")
async def calculate_totals(session, app_id, table_name, tenant_id, *, row_ids=None, **_):
    """Recalculate totals for selected rows."""
    if not row_ids:
        return {"ok": False, "message": "No rows selected"}

    updated = 0
    for rid in row_ids:
        row = await data_service.get_row(session, app_id, table_name, rid, tenant_id=tenant_id)
        if not row:
            continue
        data = row.data or {}
        total = sum(float(item.get("amount", 0)) for item in data.get("items", []))
        await data_service.update_row(
            session, app_id, table_name, rid,
            {**data, "total": total},
            tenant_id=tenant_id,
        )
        updated += 1

    await session.commit()
    return {"ok": True, "message": f"Updated {updated} records", "updated_count": updated}
```

Register it by importing the module in `action_handlers.py` or via an auto-discovery mechanism.

## Level 3: Plugins

For external integrations and complex features:

```python
# backend/app/plugins/slack_notifier/handler.py

from app.plugins.base import BasePlugin, PluginContext, PluginResult
import httpx

class SlackNotifierPlugin(BasePlugin):
    async def execute(self, ctx: PluginContext, action: str, params: dict) -> PluginResult:
        if action == "send_notification":
            webhook_url = params.get("webhook_url")
            message = params.get("message", "")

            # Template variables
            if params.get("row_id"):
                row = await ctx.get_row(ctx.app_id, "main", params["row_id"])
                if row:
                    for key, val in row.items():
                        message = message.replace(f"{{{key}}}", str(val or ""))

            async with httpx.AsyncClient() as client:
                await client.post(webhook_url, json={"text": message})

            return PluginResult(ok=True, message="Notification sent")

        return PluginResult(ok=False, message=f"Unknown action: {action}")
```

## Level 4: Core Extensions

For fundamental platform capabilities:

### Adding a New Rendering Type

1. Create a new Canvas component in `frontend/src/components/renderer/`
2. Register it in the rendering pipeline (e.g., `AppDetail.tsx`)
3. Add the trigger condition (e.g., `app_meta.render_type === "my_type"`)

### Adding a New Engine

1. Create the engine in `backend/app/services/`
2. Integrate it into the Data API flow (`data.py`)
3. Add DSL schema support in `schemas/dsl.py`
4. Document the new DSL section

### Adding a New API Module

1. Create the route module in `backend/app/api/v1/`
2. Register it in `router.py`
3. Add corresponding frontend API functions in `api/client.ts`

## Best Practices

1. **Keep it generic** — Avoid hardcoding app IDs or tenant names in core code
2. **Use DSL flags** — If you need conditional behavior, add a flag to `app_meta` or `logic` rather than checking app IDs
3. **Test with multiple apps** — Ensure your extension works for any DSL, not just your specific use case
4. **Document the extension** — Update the DSL reference if you add new DSL capabilities
5. **Follow the pattern** — Use the existing patterns (registry, context, etc.) rather than inventing new ones
