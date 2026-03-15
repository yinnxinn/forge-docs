# 配置

灵碳云铸通过环境变量进行配置，后端通过 Pydantic Settings 读取 `backend/.env`。

## 后端环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `DATABASE_URL` | PostgreSQL 异步连接串 | `postgresql+asyncpg://postgres:postgres@localhost:5432/lingtan` |
| `JWT_SECRET` | JWT 签名密钥 | （必填） |
| `JWT_EXPIRE_MINUTES` | Token 过期时间（分钟） | `1440`（24 小时） |
| `CORS_ORIGINS` | 允许的源（逗号分隔） | `http://localhost:3000` |
| `LLM_BASE_URL` | 兼容 OpenAI 的 API 地址 | `https://api.openai.com/v1` |
| `LLM_API_KEY` | LLM 服务 API Key | （AI 铸造必填） |
| `LLM_MODEL` | AI 铸造使用的模型名 | `gpt-4` |
| `REDIS_URL` | Redis 连接串（可选） | `redis://localhost:6379` |

### 示例 `.env`

```env
DATABASE_URL=postgresql+asyncpg://postgres:mypassword@localhost:5432/lingtan
JWT_SECRET=your-secret-key-change-in-production
JWT_EXPIRE_MINUTES=1440
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
LLM_BASE_URL=https://api.openai.com/v1
LLM_API_KEY=sk-...
LLM_MODEL=gpt-4
```

## 前端环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `VITE_API_ORIGIN` | 后端 API 根地址 | （空则使用代理） |

开发时 Vite 会将 `/api` 代理到后端。生产环境需设置 `VITE_API_ORIGIN` 或通过 Nginx 反向代理。

## 数据库初始化

使用 PostgreSQL + 异步 SQLAlchemy，表由初始化脚本创建：

```bash
# 创建所有表
python -m scripts.init_db

# 确保最新表结构
python -m scripts.ensure_all_tables

# 创建管理员用户
python -m scripts.seed_admin_user
```

## Docker 配置

Docker Compose 部署时复制并编辑环境模板：

```bash
cp docker-compose.env.example .env
```

常用 Docker 变量：

| 变量 | 说明 |
|------|------|
| `POSTGRES_PASSWORD` | PostgreSQL 密码 |
| `POSTGRES_DB` | 数据库名 |
| `BACKEND_PORT` | 后端暴露端口 |
| `FRONTEND_PORT` | 前端暴露端口 |

## LLM 提供商配置

AI 铸造支持任意兼容 OpenAI 的 API，示例：

```env
# OpenAI
LLM_BASE_URL=https://api.openai.com/v1
LLM_API_KEY=sk-...
LLM_MODEL=gpt-4

# Azure OpenAI
LLM_BASE_URL=https://your-resource.openai.azure.com/openai/deployments/your-deployment
LLM_API_KEY=your-azure-key
LLM_MODEL=gpt-4

# 本地（Ollama、vLLM 等）
LLM_BASE_URL=http://localhost:11434/v1
LLM_API_KEY=not-needed
LLM_MODEL=llama3
```

## 安全提示

- 生产环境务必更换 `JWT_SECRET`
- 使用强密码作为 `POSTGRES_PASSWORD`
- 将 `CORS_ORIGINS` 限制为实际域名
- 不要直接暴露后端，使用 Nginx 等反向代理
- 敏感配置放在环境变量中，不要提交到 Git
