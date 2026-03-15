# Manual Deployment

For environments without Docker, or for development purposes.

## Prerequisites

- Python 3.12+
- Node.js 18+
- PostgreSQL 14+
- (Optional) Redis 7+

## Backend Setup

### 1. Create Virtual Environment

```bash
cd backend
python -m venv .venv

# Activate
source .venv/bin/activate        # Linux/macOS
.venv\Scripts\Activate.ps1       # Windows PowerShell
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure Environment

```bash
cp .env.example .env
```

Edit `.env`:
```env
DATABASE_URL=postgresql+asyncpg://postgres:password@localhost:5432/lingtan
JWT_SECRET=your-secret-key
CORS_ORIGINS=http://localhost:5173,http://localhost:3000
```

### 4. Initialize Database

```bash
# Create tables
python -m scripts.init_db
python -m scripts.ensure_all_tables

# Create admin user
python -m scripts.seed_admin_user

# (Optional) Seed demo data
python -m scripts.seed_demo_app
```

### 5. Start the Server

```bash
# Development
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Production
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

## Frontend Setup

### 1. Install Dependencies

```bash
cd frontend
npm install
```

### 2. Configure Environment

```bash
cp .env.example .env
```

For development, Vite proxies API requests:
```env
# Leave empty for dev proxy
VITE_API_ORIGIN=
```

For production:
```env
VITE_API_ORIGIN=https://api.your-domain.com
```

### 3. Development Server

```bash
npm run dev
```

### 4. Production Build

```bash
npm run build
# Output in dist/
```

Serve the `dist/` directory with Nginx, Apache, or any static file server.

## Nginx Configuration (Production)

```nginx
server {
    listen 80;
    server_name your-domain.com;

    # Frontend
    location / {
        root /path/to/frontend/dist;
        try_files $uri $uri/ /index.html;
    }

    # API proxy
    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

## Deployment Scripts

The repository includes deployment scripts for automated setup:

### Linux/macOS

```bash
bash scripts/deploy.sh
```

### Windows

```powershell
.\scripts\deploy.ps1
```

These scripts:
1. Create/activate virtual environment
2. Install dependencies
3. Run database migrations
4. Seed initial data
5. Start the backend server

## Systemd Service (Linux)

```ini
[Unit]
Description=Lingtarn Forge Backend
After=postgresql.service

[Service]
User=www-data
WorkingDirectory=/opt/lingtarn-forge/backend
Environment=PATH=/opt/lingtarn-forge/backend/.venv/bin
ExecStart=/opt/lingtarn-forge/backend/.venv/bin/gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
Restart=always

[Install]
WantedBy=multi-user.target
```
