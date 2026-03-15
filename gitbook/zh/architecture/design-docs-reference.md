# 设计文档索引

本页索引仓库内与核心框架相关的深度设计文档，便于与 GitBook 配套阅读。文档位于 `docs/architecture/`，与当前开源实现一致。

| 文档 | 说明 |
|------|------|
| [technical.md](../../docs/architecture/technical.md) | 技术栈、Lingtarn-DSL 与元数据知识库对应、前端渲染与后端动态逻辑 |
| [db.md](../../docs/architecture/db.md) | 元数据知识库、物理表与统一访问层、菜单与权限表结构 |
| [execution-flow-and-generality.md](../../docs/architecture/execution-flow-and-generality.md) | 执行流程与通用性分析：三条触发路径、场景级追踪、通用性结论 |
| [ai-forge-automation-boundary.md](../../docs/architecture/ai-forge-automation-boundary.md) | AI 铸造与自动化能力边界：Level 1–4 能力分层、DSL 边缘与插件边界 |
| [fullstack-vibecoding-platform.md](../../docs/architecture/fullstack-vibecoding-platform.md) | 全栈 Vibe-Coding 平台架构：DSL 契约、OpenAPI Codegen、Plugin 架构、实施路线图 |

建议先阅读 GitBook 的 [架构概览](overview.md) 与 [开源范围](open-source-scope.md)，再按需查阅上表文档。
