# Lingtarn Forge（灵碳云铸）

基于 DSL 的开源全栈平台，用 AI 构建业务应用。

**开源分支说明：** `open-source` 分支仅包含**核心框架**（DSL 运行时、CRUD 引擎、公式/工作流/自动化引擎、前端渲染器、认证与 RBAC）。详见 [开源范围](architecture/open-source-scope.md)。

## 什么是灵碳云铸？

灵碳云铸是一套**元数据驱动的应用平台**：用户用自然语言描述需求，AI 生成结构化 DSL，平台自动渲染界面并执行业务逻辑，无需手写应用代码。

**核心思想：** DSL 是前后端的唯一契约；AI 只改 DSL，两端自动响应。

## 核心能力

- **DSL 即唯一真相源** — 应用行为（schema、layout、logic、workflow、automation）全部由 JSON DSL 表达，无需应用级代码。
- **AI 生成应用** — 自然语言描述需求，AI Forge 生成合法 DSL，平台即时渲染。
- **动态渲染引擎** — FormCanvas、ListCanvas、WorkflowCanvas 根据 DSL 直接渲染。
- **元数据驱动 CRUD** — 单一接口 `/api/v1/data/{app_id}/{model}` 处理所有应用的数据操作。
- **工作流引擎** — DSL 定义状态机、按角色流转、审批与审计。
- **自动化引擎** — 记录创建/更新、工作流流转等事件触发跨应用数据操作。
- **公式引擎** — 计算字段（IF、SEQUENCE、concat、四则运算、日期函数等）。
- **多租户 RBAC** — 租户隔离、角色权限、部门层级、行级数据规则。
- **插件架构** — DSL 不足时通过插件扩展。

## 架构一览

```
┌─────────────────────────────────────────────────────┐
│              Lingtarn DSL (JSON)                    │
│         (app_meta, schema, layout, logic)           │
└──────────┬──────────────────────────┬──────────────┘
           │                          │
  ┌────────▼────────┐       ┌────────▼────────┐
  │  后端引擎        │       │  前端引擎        │
  │  · DataService   │       │  · FormCanvas   │
  │  · FormulaEngine │       │  · ListCanvas   │
  │  · WorkflowEngine│       │  · Registry     │
  │  · AutomationEng │       │                 │
  └────────┬────────┘       └─────────────────┘
           │
  ┌────────▼────────┐
  │  PostgreSQL      │
  │  · lt_app_data   │  (JSONB)
  │  · lt_sys_*      │  (元数据, RBAC)
  └─────────────────┘
```

## 技术栈

| 层级 | 技术 |
|------|------|
| 后端 | Python 3.12+、FastAPI、SQLAlchemy (async)、PostgreSQL |
| 前端 | React 18、TypeScript、Vite、Tailwind CSS、shadcn/ui |
| AI | 兼容 OpenAI 的 LLM API（可配置） |
| 部署 | Docker Compose、Nginx |

## 快速开始

```bash
git clone https://github.com/your-org/lingtarn-forge.git
cd lingtarn-forge
docker compose up -d
```

访问 `http://localhost:3000`，使用默认管理员账号登录。

详见 [快速开始](getting-started/quick-start.md)。

## 文档导航

- [架构概览](architecture/overview.md)
- [开源范围](architecture/open-source-scope.md)
- [DSL 参考](dsl-reference/overview.md)
- [后端框架](backend/overview.md)
- [前端框架](frontend/overview.md)
- [API 参考](api-reference/data-api.md)
- [部署指南](deployment/docker.md)
- [参与贡献](contributing/development.md)

## 构建本书

本目录符合 [GitBook](https://www.gitbook.com/) 结构；多语言由根目录 `LANGS.md` 配置。构建示例：

- **HonKit：** `npx honkit build`
- **GitBook CLI：** `npx gitbook-cli build . ./_book`

## 许可证

MIT License — 详见 [LICENSE](../../LICENSE)。
