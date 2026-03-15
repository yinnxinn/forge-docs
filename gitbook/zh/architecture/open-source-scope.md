# 开源范围 / Open Source Scope

本仓库 **open-source** 分支面向社区开源的是 **灵碳云铸 (Lingtarn Forge) 核心框架**，即与具体客户、具体业务解耦的通用平台能力。配套文档即本 GitBook 目录下的全部内容。

## 开源内容（What Is Open Sourced）

| 模块 | 说明 | 代码位置（概要） |
|------|------|------------------|
| **DSL 规范与运行时** | 应用元数据、schema、layout、logic、workflow、automation 的 JSON 定义及前后端解释 | `backend/app/schemas/dsl.py`、`frontend/src/types/dsl.ts`、各 Engine |
| **通用 CRUD 引擎** | 基于 `app_id + model` 的通用数据 API，表/字段注册、权限、租户隔离 | `backend/app/api/v1/data.py`、`backend/app/services/data_service.py` |
| **公式引擎 (Formula Engine)** | 计算字段、表达式求值（IF、now、concat、四则运算等） | `backend/app/services/formula_engine.py` |
| **工作流引擎 (Workflow Engine)** | 状态机、角色驱动流转、审批 | `backend/app/services/workflow_engine.py` |
| **自动化引擎 (Automation Engine)** | record_created / record_updated / workflow_transition 触发与跨应用数据联动 | `backend/app/services/automation_engine.py` |
| **动作处理器 (Action Handlers)** | 行级/列表级动作注册与执行框架 | `backend/app/services/action_handlers.py` |
| **前端 DSL 渲染器** | FormCanvas、ListCanvas、Registry、基于 DSL 的动态表单与列表 | `frontend/src/components/renderer/` |
| **认证与 RBAC** | JWT 登录、租户/角色/权限/数据规则 | `backend/app/api/v1/auth.py`、`backend/app/services/permission_service.py`、`backend/app/models/rbac.py` |
| **应用与菜单** | 应用注册、安装/卸载、菜单树 | `backend/app/api/v1/apps.py`、`backend/app/api/v1/menus.py` |
| **AI Forge 占位** | 自然语言 → DSL 的接口与流程骨架（具体 LLM 配置由部署方决定） | `backend/app/api/v1/forge.py` |
| **插件架构** | 扩展点与插件加载机制 | 见 [插件系统](backend/plugin-system.md) |

上述能力均通过 **DSL 驱动**，不依赖某一具体行业或客户的业务代码。

## 不开源 / 分支外（What Is Not Open Sourced）

- **客户定制业务**：如某客户的专属 API、专属模型、专属 DSL 扩展（如 `hehong_dsl` 等）仅在内部或商业分支维护。
- **内部运营与部署**：内部环境变量、部署脚本中的敏感配置、内部文档与业务需求文档等。
- **商业版增强**：若存在商业版功能（如高级报表、专属集成），不在本开源范围内。

## 设计原则（Design Principles）

- **DSL 为唯一契约**：前后端仅通过 Lingtarn DSL 约定行为；AI 优先只改 DSL。
- **多租户与权限内建**：数据、菜单、权限均按租户隔离，RBAC 与数据规则为平台能力。
- **可扩展**：通过 Plugin 与 Action Handlers 在边界内扩展，避免修改核心引擎。

## 配套文档（Documentation）

本 GitBook 与开源范围一一对应：

- [架构概览](overview.md) — 整体架构与请求生命周期  
- [DSL 驱动设计](dsl-driven-design.md) — DSL 驱动设计  
- [三层数据模型](three-layer-data-model.md) — 三层数据模型  
- [执行流程](execution-flow.md) — 执行流  
- [设计文档索引](design-docs-reference.md) — 仓库内深度设计文档  

更深层的设计分析（执行路径、AI 能力边界、全栈 Vibe-Coding 方案）见仓库内 `docs/architecture/` 下的文档，与本节描述的核心框架一致。

## 许可证（License）

MIT License — 详见仓库根目录 [LICENSE](../../LICENSE)。
