# Authentication API

All API endpoints (except login and register) require a valid JWT token.

## Endpoints

### Login

```
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "admin123"
}
```

**Response:**
```json
{
  "access_token": "eyJ...",
  "token_type": "bearer",
  "user": {
    "id": 1,
    "username": "admin",
    "display_name": "Admin",
    "email": "admin@example.com",
    "tenant_id": "default",
    "is_superuser": true,
    "role_type": "admin",
    "role_types": ["admin"]
  }
}
```

### Register

```
POST /api/v1/auth/register
Content-Type: application/json

{
  "username": "newuser",
  "password": "securepassword",
  "display_name": "New User",
  "email": "user@example.com",
  "tenant_id": "my_company"
}
```

### Get Current User

```
GET /api/v1/auth/me
Authorization: Bearer {token}
```

**Response:**
```json
{
  "id": 1,
  "username": "admin",
  "display_name": "Admin",
  "email": "admin@example.com",
  "tenant_id": "default",
  "is_superuser": true,
  "role_type": "admin",
  "role_types": ["admin"],
  "departments": [
    { "id": 1, "name": "Engineering" }
  ]
}
```

## Authentication Flow

1. Client sends credentials to `/auth/login`
2. Server verifies password (bcrypt)
3. Server returns JWT token (valid for `JWT_EXPIRE_MINUTES`)
4. Client stores token in `localStorage`
5. All subsequent requests include `Authorization: Bearer {token}` header
6. Server validates token on each request via `get_current_user` dependency

## Token Format

The JWT payload contains:

```json
{
  "sub": "1",           // User ID
  "exp": 1710432000,    // Expiration timestamp
  "iat": 1710345600     // Issued at
}
```

## Role Types

| Role | Access Level |
|------|-------------|
| `admin` | Full access, user/tenant management |
| `developer_dev` | App development, Forge, dev center |
| `developer_pm` | Project management, Forge |
| `ceo` | Forge access, executive views |
| `manager` | Department management, approvals |
| `user` | Basic app access |

## Security Headers

All authenticated requests must include:

```
Authorization: Bearer eyJ...
```

The `get_current_user` dependency:
1. Extracts the token from the `Authorization` header
2. Decodes and validates the JWT
3. Loads the user from the database
4. Loads user roles and departments
5. Returns the user object (or raises 401)
