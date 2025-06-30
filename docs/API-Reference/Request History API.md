## Request History API

### GET /api/logs/
```json
Query Parameters:
- page: integer (default: 1)
- limit: integer (default: 50, max: 100)
- method: string (e.g., GET, POST - optional filter)
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

### GET /api/logs/{log_id}
Path Parameter:
- `log_id`: UUID of the specific log entry (execution_id).

Response (200 OK):
```json
{
  "id": "log_uuid",
  "user_id": "user_uuid",
  "endpoint_id": "endpoint_uuid_or_null",
  "collection_id": "collection_uuid_or_null",
  "request_snapshot": {
    "method": "GET",
    "url": "https://api.example.com/users/12345",
    "headers": {"Authorization": "Bearer ..."},
    "query_params": {"include": "profile"},
    "body": null,
    "body_type": null
  },
  "response_status": 200,
  "response_headers": {
    "content-type": "application/json"
  },
  "response_body": "{\"id\":\"12345\",\"name\":\"John Doe\"}",
  "response_size_bytes": 123,
  "duration_ms": 245,
  "error_message": null,
  "ip_address": "192.0.2.1",
  "user_agent": "PostkidClient/1.0",
  "created_at": "2025-06-01T10:00:00Z"
}
```
Response (404 Not Found): If the log entry does not exist or the user does not have access.
Description: Retrieves a specific log entry by its ID, including the full request and response details.

---
