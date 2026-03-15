# Security & Sensitive Information

## Documentation content

This repository contains **official documentation** for Lingtarn Forge (灵碳云铸). It does **not** contain:

- Real API keys, passwords, or tokens
- Production configuration or `.env` files

Examples in the docs (e.g. `JWT_SECRET=your-secret-key-change-in-production`, `admin123`, `sk-...`) are **placeholders only**. If you deploy using the main project:

- Change all default passwords and `JWT_SECRET` in production
- Do not commit `.env` or any file containing real secrets
- Restrict `CORS_ORIGINS` and use a reverse proxy in front of the backend

## Reporting security issues

If you discover a security vulnerability in the documented product (Lingtarn Forge), please report it responsibly. Prefer private disclosure (e.g. GitHub Security Advisories or a contact email) rather than opening a public issue for sensitive findings.

## Repository security

- Do not add `.env`, API keys, or credentials to this repo
- The `.gitignore` excludes `.env` and common secret paths
