# 快速开始

约 5 分钟内在本地运行灵碳云铸。

## 前置条件

- Docker 与 Docker Compose
- Git
- （可选）Node.js 18+、Python 3.12+（开发模式）

## 方式一：Docker Compose（推荐）

```bash
# 克隆仓库
git clone https://github.com/your-org/lingtarn-forge.git
cd lingtarn-forge

# 复制环境模板
cp docker-compose.env.example .env

# 启动所有服务
docker compose up -d
```

将启动：

- **PostgreSQL** 端口 5432
- **Redis** 端口 6379
- **后端**（FastAPI）端口 8000
- **前端**（Vite/React）端口 3000

访问 `http://localhost:3000` 进入平台。

## 方式二：手动安装（开发）

### 后端

```bash
cd backend

# 创建虚拟环境
python -m venv .venv
source .venv/bin/activate   # Linux/macOS
# .venv\Scripts\Activate.ps1  # Windows PowerShell

# 安装依赖
pip install -r requirements.txt

# 复制环境配置
cp .env.example .env
# 编辑 .env 中的数据库地址

# 初始化数据库
python -m scripts.init_db
python -m scripts.ensure_all_tables
python -m scripts.seed_admin_user

# 启动服务
uvicorn app.main:app --reload --port 8000
```

### 前端

```bash
cd frontend

# 安装依赖
npm install

# 复制环境配置
cp .env.example .env

# 启动开发服务
npm run dev
```

## 首次登录

安装完成后，使用默认管理员账号登录：

- **用户名：** `admin`
- **密码：** `admin123`（生产环境请立即修改）

## 创建第一个应用

1. 进入 **AI 铸造**（管理员/开发者角色可用）
2. 用自然语言描述应用，例如：
   > 「创建一个任务管理应用：标题（文本，必填）、负责人（文本）、优先级（下拉：高/中/低）、截止日期（日期）、状态（下拉：待办/进行中/已完成）」
3. AI 生成 Lingtarn DSL
4. 点击 **发布** 注册应用
5. 应用出现在侧边栏，即可使用

## 项目结构

```
lingtarn-forge/
├── backend/                 # FastAPI 后端
│   ├── app/
│   │   ├── main.py          # 应用入口
│   │   ├── config.py        # 环境配置
│   │   ├── core/            # 数据库、安全
│   │   ├── api/v1/          # API 路由
│   │   ├── models/          # SQLAlchemy 模型
│   │   ├── schemas/         # Pydantic 模式
│   │   ├── services/        # 业务逻辑引擎
│   │   ├── assistant/       # AI 助手
│   │   └── plugins/         # 插件
│   ├── scripts/            # 数据库初始化与种子脚本
│   └── requirements.txt
├── frontend/                # React 前端
│   ├── src/
│   │   ├── api/             # API 客户端
│   │   ├── components/     # UI 组件
│   │   │   ├── renderer/   # DSL 渲染器
│   │   │   ├── layout/     # 布局
│   │   │   └── ui/         # 基础 UI (shadcn)
│   │   ├── contexts/       # React 上下文
│   │   ├── pages/          # 路由页面
│   │   ├── types/          # TypeScript 类型
│   │   └── lib/             # 工具
│   └── package.json
├── docker-compose.yml
├── gitbook/                 # 本文档
└── scripts/                 # 部署脚本
```

## 下一步

- [配置](configuration.md) — 环境与变量说明
- [架构概览](../architecture/overview.md) — 系统设计
- [DSL 参考](../dsl-reference/overview.md) — 手写 DSL
