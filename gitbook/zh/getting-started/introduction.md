# 简介

## 什么是灵碳云铸？

灵碳云铸（Lingtarn Forge）是一套开源的 **DSL 驱动全栈平台**，用于构建业务应用。你无需为每个功能或模块手写应用代码，只需用结构化的 JSON DSL 描述应用，平台即可自动完成界面渲染、数据存储、权限、工作流与自动化。

## 要解决的问题

企业级业务应用通常需要：

1. 为每个模块设计数据库表结构
2. 为每个实体编写 CRUD 接口
3. 开发前端表单与列表页
4. 实现工作流与审批逻辑
5. 串联跨模块数据流
6. 处理多租户隔离与权限

这些工作重复、易错、成本高，每个新模块都要数周开发。

## 解决方案

灵碳云铸引入**单一抽象** —— Lingtarn DSL，用一份 JSON 描述上述全部能力：

```json
{
  "app_meta": { "name": "销售订单", "icon": "ShoppingCart" },
  "schema": {
    "records": [
      { "id": "customer", "label": "客户", "type": "text", "required": true },
      { "id": "amount", "label": "金额", "type": "number" },
      { "id": "status", "label": "状态", "type": "select", "options": ["草稿", "已确认", "已发货"] }
    ]
  },
  "layout": {
    "list": ["customer", "amount", "status"],
    "form": [["customer", "amount"], ["status"]]
  },
  "logic": {
    "workflow": {
      "field": "status",
      "states": [
        { "id": "Draft", "label": "草稿", "color": "gray" },
        { "id": "Confirmed", "label": "已确认", "color": "blue" },
        { "id": "Shipped", "label": "已发货", "color": "green" }
      ],
      "transitions": [
        { "id": "confirm", "from": "Draft", "to": "Confirmed", "label": "确认" },
        { "id": "ship", "from": "Confirmed", "to": "Shipped", "label": "发货" }
      ]
    }
  }
}
```

凭借这一份 DSL，灵碳云铸会自动：

- 创建数据存储（PostgreSQL JSONB）
- 注册表与字段元数据
- 渲染列表页与表单页
- 支持按角色的工作流状态流转
- 在状态变化时触发自动化规则
- 通过公式引擎应用计算字段

## 适用对象

- **开发者** — 希望快速搭建业务应用、减少重复 CRUD 开发
- **产品经理** — 希望快速原型与迭代应用设计
- **企业** — 需要多租户、权限可控的业务模块
- **AI 爱好者** — 希望了解如何用 LLM 从自然语言生成可运行应用

## 核心概念

| 概念 | 说明 |
|------|------|
| **DSL** | 定义整应用的 JSON 文档（schema、layout、logic） |
| **应用 (App)** | 已注册的 DSL 实例，拥有独立数据、菜单与权限 |
| **租户 (Tenant)** | 平台内的隔离组织 |
| **铸造 (Forge)** | 用户用自然语言描述应用的 AI 界面 |
| **渲染器 (Renderer)** | 根据 DSL 动态渲染界面的前端组件 |
| **引擎 (Engine)** | 执行 DSL 逻辑的后端服务（公式、工作流、自动化） |

## 下一步

- [快速开始](quick-start.md) — 5 分钟内本地跑通
- [架构概览](../architecture/overview.md) — 理解系统设计
- [DSL 参考](../dsl-reference/overview.md) — 学习 DSL 规范
