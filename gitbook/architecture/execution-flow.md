# Execution Flow

This document details how data flows through the platform for the three main trigger paths: Create, Update, and Workflow Transition.

## Trigger Path 1: Create Record

```
POST /api/v1/data/{app_id}/{model}
    │
    ├─ Authenticate (JWT)
    ├─ Resolve table_name from app_id + model
    ├─ Check permission: C (Create)
    ├─ Load DSL from lt_sys_app.dsl_snapshot
    │
    ├─ Run Computations (Formula Engine)
    │   └─ For each computation in logic.computations:
    │       Evaluate formula, set computed field value
    │
    ├─ Create row in lt_app_data
    │   └─ Set app_id, table_name, tenant_id, department_id, creator_id, data
    │
    ├─ Fire Automation (record_created)
    │   └─ For each rule matching trigger=record_created:
    │       Execute actions (create_record, update_record, notify, etc.)
    │
    └─ Return DataRowOut
```

### Example: Creating a Sales Order

```
POST /data/sales/main
Body: { "data": { "customer": "Acme Corp", "amount": 5000 } }

1. Auth: user.id=7, tenant_id="tenant_a"
2. Resolve: "sales" + "main" → "sales_main"
3. Permission: check(user, "sales_main", "C") → OK
4. Load DSL: sales app DSL with computations
5. Computations:
   - order_no = SEQUENCE("SO", 6) → "SO-000042"
   - created_by = $current.creator_display_name
6. Create: INSERT lt_app_data (data = {"customer":"Acme","amount":5000,"order_no":"SO-000042",...})
7. Automation: rule "notify_on_new_order" → dispatch notification
8. Return: { id: 123, data: {...}, created_at: "..." }
```

## Trigger Path 2: Update Record

```
PATCH /api/v1/data/{app_id}/{model}/{row_id}
    │
    ├─ Authenticate (JWT)
    ├─ Resolve table_name
    ├─ Check permission: U (Update)
    ├─ Load DSL
    │
    ├─ Check Constraints
    │   └─ logic.constraints[].lock_when_status
    │       If current status is locked → 400 error
    │
    ├─ Run Computations (Formula Engine)
    │   └─ Recompute fields based on new data
    │
    ├─ Update row in lt_app_data
    │
    ├─ Fire Automation (record_updated)
    │   └─ Match rules where trigger=record_updated
    │       Optionally filter by changed_fields
    │
    └─ Return updated DataRowOut
```

## Trigger Path 3: Workflow Transition

```
POST /api/v1/data/{app_id}/{model}/{record_id}/workflow/{transition_id}
    │
    ├─ Authenticate (JWT)
    ├─ Resolve table_name
    ├─ Load DSL, extract workflow config
    │
    ├─ Workflow Engine
    │   ├─ Validate: current_state matches transition.from
    │   ├─ Validate: user has allowed_role for this transition
    │   ├─ Validate: comment provided if requires_comment=true
    │   ├─ Update: data[workflow.field] = transition.to
    │   └─ Append to: data._workflow_history
    │
    ├─ Fire Automation (workflow_transition)
    │   └─ Match rules where trigger=workflow_transition
    │       AND trigger_meta.transition_id matches
    │       Execute: create_record, update_record, update_current, notify
    │
    └─ Return { ok, from_state, to_state, side_effects }
```

### Example: Approving a Purchase Order

```
POST /data/procurement/main/45/workflow/approve

1. Load DSL → workflow config
2. Current state: "pending" (from record data)
3. Transition "approve": from="pending", to="approved", allowed_roles=["manager"]
4. User roles: ["manager"] → allowed
5. Update: data.status = "approved", append to _workflow_history
6. Automation fires: rule "on_approval_create_warehouse_task"
   → create_record in warehouse_tasks app
   → field_mapping: { source_order_id: $current.id, quantity: $current.quantity }
7. Return: { ok: true, from_state: "pending", to_state: "approved", side_effects: [...] }
```

## Cross-App Automation Flow

One of the most powerful features is cross-app automation. When an event in App A triggers record creation/update in App B:

```
App A: Sales Order                    App B: Warehouse Tasks
┌─────────────────┐                  ┌─────────────────┐
│ Order approved   │──automation───→ │ Task created     │
│ (workflow trans) │   rule fires    │ (record_created) │
└─────────────────┘                  └─────────────────┘
                                              │
                                              ▼
                                     (future: cascade trigger)
```

### Field Mapping in Automation

```json
{
  "trigger": "workflow_transition",
  "trigger_meta": { "transition_id": "approve" },
  "actions": [{
    "type": "create_record",
    "target_app_id": "warehouse_tasks",
    "target_model": "main",
    "field_mapping": [
      { "target": "source_order_id", "source": "$current.id" },
      { "target": "product_id", "source": "$current.product_id" },
      { "target": "quantity", "source": "$current.quantity" },
      { "target": "priority", "mode": "expr", "value": "IF($current.amount > 10000, 'high', 'normal')" },
      { "target": "created_at", "mode": "literal", "value": "$now" }
    ]
  }]
}
```

Supported `field_mapping` modes:
- `source` — Copy from current record (`$current.field_name`)
- `literal` — Fixed value (supports `$now`, `$creator`)
- `expr` — Formula expression evaluated by FormulaEngine
- `lookup` — (planned) Query another app for the value

## Known Limitations

1. **No cascade triggers** — When automation creates a record in App B, `record_created` rules in App B do not fire automatically (to prevent infinite loops)
2. **No lookup** in field_mapping — The `lookup` mode is not yet implemented
3. **Depth limit** — Automation has a configurable cascade depth limit to prevent runaway chains
