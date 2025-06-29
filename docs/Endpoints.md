## 🔌 API Specification

### Authentication Endpoints

#### POST /api/auth/register
```json
Request:
{
  "email": "user@example.com",
  "password": "securepassword123",
  "first_name": "John",
  "last_name": "Doe"
}

Response (201):
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "first_name": "John",
    "last_name": "Doe"
  },
  "tokens": {
    "access": "jwt_token",
    "refresh": "refresh_token"
  }
}
```

#### POST /api/auth/login
```json
Request:
{
  "email": "user@example.com",
  "password": "securepassword123"
}

Response (200):
{
  "user": { "id": "uuid", "email": "user@example.com" },
  "tokens": {
    "access": "jwt_token",
    "refresh": "refresh_token"
  }
}
```

### Collection Management

#### GET /api/collections/
```json
Response (200):
{
  "collections": [
    {
      "id": "uuid",
      "name": "My API Tests",
      "description": "Collection for testing external APIs",
      "is_public": false,
      "endpoint_count": 5,
      "created_at": "2025-06-01T10:00:00Z"
    }
  ],
  "total": 1
}
```

#### POST /api/collections/
```json
Request:
{
  "name": "New Collection",
  "description": "Testing payment APIs",
  "tags": ["payment", "stripe", "testing"]
}

Response (201):
{
  "id": "uuid",
  "name": "New Collection",
  "description": "Testing payment APIs",
  "tags": ["payment", "stripe", "testing"],
  "created_at": "2025-06-01T10:00:00Z"
}
```

### Endpoint Management

#### POST /api/collections/{collection_id}/endpoints/
```json
Request:
{
  "name": "Get User Profile",
  "method": "GET",
  "url": "https://api.example.com/users/{{user_id}}",
  "headers": {
    "Authorization": "Bearer {{api_token}}",
    "Content-Type": "application/json"
  },
  "query_params": {
    "include": "profile,settings"
  },
  "timeout_seconds": 15,
  "auth_config": {
    "type": "bearer",
    "token": "{{api_token}}"
  }
}

Response (201):
{
  "id": "uuid",
  "name": "Get User Profile",
  "method": "GET",
  "url": "https://api.example.com/users/{{user_id}}",
  "created_at": "2025-06-01T10:00:00Z"
}
```

### Request Execution

#### POST /api/test/{endpoint_id}/
```json
Request:
{
  "variables": {
    "user_id": "12345",
    "api_token": "sk_test_..."
  },
  "override_timeout": 20
}

Response (200):
{
  "execution_id": "uuid",
  "status": 200,
  "headers": {
    "content-type": "application/json",
    "x-ratelimit-remaining": "99"
  },
  "body": {
    "id": "12345",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "duration_ms": 245,
  "response_size_bytes": 156,
  "timestamp": "2025-06-01T10:00:00Z"
}
```

### Request History

#### GET /api/logs/
```json
Query Parameters:
- page: integer (default: 1)
- limit: integer (default: 50, max: 100)
- collection_id: UUID (optional filter)
- status_code: integer (optional filter)
- start_date: ISO date (optional filter)
- end_date: ISO date (optional filter)

Response (200):
{
  "logs": [
    {
      "id": "uuid",
      "endpoint_name": "Get User Profile",
      "method": "GET",
      "url": "https://api.example.com/users/12345",
      "status": 200,
      "duration_ms": 245,
      "created_at": "2025-06-01T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 1,
    "pages": 1
  }
}
```

---
