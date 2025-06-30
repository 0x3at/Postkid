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
