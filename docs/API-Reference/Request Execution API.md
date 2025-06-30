## Request Execution API

### POST /api/test/{endpoint_id}/
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

---
### Ad-hoc Request Execution (Conceptual)

While not explicitly defined as a separate endpoint, ad-hoc request execution (where the user defines the request details directly without saving to an endpoint first) can be supported by:
1.  The UI implicitly creating a temporary or unsaved endpoint definition.
2.  The UI then calling `POST /api/test/{endpoint_id}/` with the ID of this temporary/unsaved endpoint.

Alternatively, a future enhancement could be a dedicated endpoint like `POST /api/test/adhoc` that accepts the full request definition (method, URL, headers, body, variables, timeout) in its body. For the initial MVP, executing saved or temporarily created endpoints is considered sufficient.
