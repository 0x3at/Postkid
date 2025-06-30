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
