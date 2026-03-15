# 架构概览

灵碳云铸是**元数据驱动、DSL 优先**的平台。本文说明高层架构与各模块关系。

## 设计原则

| 原则 | 说明 |
|------|------|
| **DSL 为唯一契约** | 所有应用行为由 Lingtarn DSL（JSON）表达；后端解释、前端渲染，无需应用级代码。 |
| **AI 工作边界** | 层级 1：仅改 DSL（首选）。层级 2：在插件模板内写代码。层级 3：改核心平台（最后手段）。 |
| **端到端类型安全** | OpenAPI 规范 → 生成 TypeScript 类型 → 类型安全的 API 客户端。 |
| **元数据驱动** | 逻辑与存储分离，运行时由元数据驱动渲染与行为。 |
| **默认多租户** | 每条数据、权限、菜单均按租户隔离。 |

## 系统架构

```
┌──────────────────────────────────────────────────────────────────────┐
│                           用户 / AI 铸造                              │
│                    「创建一个带线索与商机的 CRM」                       │
└────────────────────────────────┬─────────────────────────────────────┘
                                 │ 自然语言
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│                        AI Forge (LLM)                                 │
│               生成校验后的 Lingtarn DSL (JSON)                        │
└────────────────────────────────┬─────────────────────────────────────┘
                                 │ DSL
                                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│                      Lingtarn DSL (JSON)                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐               │
│  │ app_meta │ │  schema  │ │  layout  │ │  logic   │               │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘               │
└──────────┬────────────────────────────────────┬─────────────────────┘
           │                                    │
  ┌────────▼─────────────────┐       ┌──────────▼──────────────┐
  │     后端引擎              │       │    前端引擎              │
  │  · 应用注册               │       │  · Registry (类型→组件) │
  │  · DataService (CRUD)    │       │  · FormCanvas           │
  │  · FormulaEngine         │       │  · ListCanvas           │
  │  · WorkflowEngine        │       │  · WorkflowCanvas       │
  │  · AutomationEngine      │       │  · PluginCanvas         │
  │  · ActionHandlers        │       │  · EmbedPageCanvas       │
  │  · PermissionService     │       │                          │
  │  · PluginSystem          │       │                          │
  └────────┬─────────────────┘       └──────────────────────────┘
           │
  ┌────────▼─────────────────┐
  │      PostgreSQL           │
  │  元数据: lt_sys_app, lt_sys_table_registry, lt_sys_field_registry  │
  │  RBAC: lt_sys_tenant, lt_sys_role, lt_sys_permission, lt_sys_data_rule │
  │  业务数据: lt_app_data (JSONB)    │
  └───────────────────────────┘
```

## 请求生命周期

每次数据操作大致流程：

```
HTTP 请求
    │
    ▼
认证层 — JWT 校验、加载用户
    ▼
权限服务 — RBAC（表+操作）、数据规则（行级）
    ▼
解析元数据 — app_id + model → 表名，加载 table_logic、field_registry
    ▼
执行逻辑 — 校验器 → 计算（公式引擎）
    ▼
执行 SQL — 对 lt_app_data 的 INSERT/UPDATE/DELETE，租户与部门隔离
    ▼
自动化引擎 — 触发 record_created、record_updated 等规则，跨应用数据操作
    ▼
响应 — 返回 DataRowOut（id、data、时间戳）
```

## 组件一览

| 组件 | 位置 | 职责 |
|------|------|------|
| **FastAPI 应用** | `backend/app/main.py` | HTTP 服务、CORS、生命周期 |
| **Config** | `backend/app/config.py` | 环境配置 |
| **Database** | `backend/app/core/database.py` | 异步 SQLAlchemy 引擎 |
| **Security** | `backend/app/core/security.py` | JWT、密码哈希 |
| **Auth Deps** | `backend/app/api/deps.py` | get_current_user、权限辅助 |
| **Data API** | `backend/app/api/v1/data.py` | 通用 CRUD 接口 |
| **Data Service** | `backend/app/services/data_service.py` | 表解析、行操作 |
| **Formula Engine** | `backend/app/services/formula_engine.py` | 计算字段求值 |
| **Workflow Engine** | `backend/app/services/workflow_engine.py` | 状态机流转 |
| **Automation Engine** | `backend/app/services/automation_engine.py` | 事件驱动规则 |
| **Action Handlers** | `backend/app/services/action_handlers.py` | 自定义动作注册 |
| **Permission Service** | `backend/app/services/permission_service.py` | RBAC 校验 |
| **DSL Types** | `frontend/src/types/dsl.ts` | TypeScript DSL 类型 |
| **Registry** | `frontend/src/components/renderer/Registry.tsx` | 字段类型 → 组件映射 |
| **FormCanvas** | `frontend/src/components/renderer/FormCanvas.tsx` | 动态表单渲染 |
| **ListCanvas** | `frontend/src/components/renderer/ListCanvas.tsx` | 动态列表渲染 |

## AI 能力层级

| 层级 | 范围 | AI 可做 |
|------|------|----------|
| **Level 1** | 纯 DSL | CRUD 应用、计算字段、工作流、自动化规则 |
| **Level 2** | DSL 边缘 | 动作条件、带查找的字段映射、级联自动化 |
| **Level 3** | 插件 | 外部 API、定时任务、文件生成 |
| **Level 4** | 平台 | WebSocket、ML、OCR 等 |

多数业务应用可在 Level 1–2 无代码完成。
