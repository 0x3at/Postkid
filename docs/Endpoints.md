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
  "user": { "id": "uuid", "email": "user@example.com", "first_name": "John", "last_name": "Doe" },
  "tokens": {
    "access": "jwt_token",
    "refresh": "refresh_token"
  }
}
```

#### POST /api/auth/refresh
Path Parameter: None
Request Body:
```json
{
  "refresh": "your_refresh_token"
}
```
Response (200 OK):
```json
{
  "access": "new_access_token"
}
```
Description: Obtains a new access token using a valid refresh token.

#### POST /api/auth/logout
Path Parameter: None
Request Body: (Optional, may depend on strategy, e.g., if refresh token is sent to be invalidated)
```json
{
  "refresh": "your_refresh_token"
}
```
Response (204 No Content or 200 OK):
Description: Logs the user out. Invalidates the refresh token on the server-side if applicable. Requires Authorization header with a valid access token.


### Collection Management

All endpoints under `/api/collections/` require authentication.

#### GET /api/collections/
Query Parameters:
- `page`: integer (default: 1)
- `limit`: integer (default: 20, max: 100)
- `sort_by`: string (e.g., `name`, `created_at`, default: `created_at`)
- `order`: string (`asc` or `desc`, default: `desc`)
Response (200 OK):
```json
{
  "collections": [
    {
      "id": "uuid",
      "name": "My API Tests",
      "description": "Collection for testing external APIs",
      "is_public": false,
      "owner_id": "user_uuid",
      "endpoint_count": 5,
      "created_at": "2025-06-01T10:00:00Z",
      "updated_at": "2025-06-01T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total_items": 1,
    "total_pages": 1
  }
}
```
Description: Retrieves a paginated list of collections accessible to the authenticated user.

#### POST /api/collections/
Request Body:
```json
{
  "name": "New Collection",
  "description": "Optional: Testing payment APIs",
  "is_public": false,
  "tags": ["optional", "payment", "stripe"]
}
```
Response (201 Created):
```json
{
  "id": "uuid",
  "name": "New Collection",
  "description": "Optional: Testing payment APIs",
  "is_public": false,
  "owner_id": "user_uuid",
  "tags": ["optional", "payment", "stripe"],
  "endpoint_count": 0,
  "created_at": "2025-06-01T10:00:00Z",
  "updated_at": "2025-06-01T10:00:00Z"
}
```
Description: Creates a new collection for the authenticated user.

#### GET /api/collections/{collection_id}
Path Parameter:
- `collection_id`: UUID of the collection.
Response (200 OK):
```json
{
  "id": "uuid",
  "name": "My API Tests",
  "description": "Collection for testing external APIs",
  "is_public": false,
  "owner_id": "user_uuid",
  "tags": ["payment", "stripe"],
  "endpoint_count": 5,
  "created_at": "2025-06-01T10:00:00Z",
  "updated_at": "2025-06-01T10:00:00Z"
}
```
Response (404 Not Found): If the collection does not exist or the user does not have access.
Description: Retrieves a specific collection by its ID.

#### PUT /api/collections/{collection_id}
Path Parameter:
- `collection_id`: UUID of the collection.
Request Body:
```json
{
  "name": "Updated Collection Name",
  "description": "Updated description",
  "is_public": true,
  "tags": ["updated", "tags"]
}
```
Response (200 OK):
```json
{
  "id": "uuid",
  "name": "Updated Collection Name",
  "description": "Updated description",
  "is_public": true,
  "owner_id": "user_uuid",
  "tags": ["updated", "tags"],
  "endpoint_count": 5,
  "created_at": "2025-06-01T10:00:00Z",
  "updated_at": "2025-06-02T11:00:00Z"
}
```
Response (404 Not Found): If the collection does not exist or the user does not have access.
Description: Updates an existing collection. Only fields provided in the request body will be updated.

#### DELETE /api/collections/{collection_id}
Path Parameter:
- `collection_id`: UUID of the collection.
Response (204 No Content):
Response (404 Not Found): If the collection does not exist or the user does not have access.
Description: Deletes a collection. Per `YellowPaper.md`, this should be a soft delete.

### Endpoint Management (within Collections)

All endpoints under `/api/collections/{collection_id}/endpoints/` require authentication and that the user owns the collection.

#### GET /api/collections/{collection_id}/endpoints/
Path Parameter:
- `collection_id`: UUID of the parent collection.
Query Parameters:
- `page`: integer (default: 1)
- `limit`: integer (default: 20, max: 100)
Response (200 OK):
```json
{
  "endpoints": [
    {
      "id": "endpoint_uuid",
      "collection_id": "collection_uuid",
      "name": "Get User Profile",
      "method": "GET",
      "url": "https://api.example.com/users/{{user_id}}",
      "created_at": "2025-06-01T10:00:00Z",
      "updated_at": "2025-06-01T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total_items": 1,
    "total_pages": 1
  }
}
```
Response (404 Not Found): If the collection does not exist.
Description: Retrieves a paginated list of endpoints within a specific collection.

#### POST /api/collections/{collection_id}/endpoints/
Path Parameter:
- `collection_id`: UUID of the parent collection.
Request Body:
```json
{
  "name": "Create New User",
  "method": "POST",
  "url": "https://api.example.com/users",
  "headers": {
    "Content-Type": "application/json",
    "X-Custom-Header": "custom_value"
  },
  "body_type": "json", // e.g., "json", "form-data", "xml", "text"
  "body": "{\n  \"name\": \"{{userName}}\",\n  \"email\": \"{{userEmail}}\"\n}",
  "query_params": {
    "source": "playground"
  },
  "timeout_seconds": 20,
  "auth_config": { // Optional: for specific auth handling by the playground for this request
    "type": "bearer", // "basic", "apiKey", "none"
    "token_env_variable": "API_TOKEN_XYZ" // Example: reference an environment variable
  },
  "pre_request_script": "console.log('Setting up request...');", // Optional JavaScript
  "post_request_script": "tests['Status code is 200'] = responseCode.code === 200;" // Optional JavaScript for tests
}
```
Response (201 Created):
```json
{
  "id": "endpoint_uuid",
  "collection_id": "collection_uuid",
  "name": "Create New User",
  "method": "POST",
  "url": "https://api.example.com/users",
  // ... other relevant fields from request ...
  "created_at": "2025-06-01T10:00:00Z",
  "updated_at": "2025-06-01T10:00:00Z"
}
```
Response (404 Not Found): If the collection does not exist.
Description: Creates a new endpoint within a collection.

#### GET /api/collections/{collection_id}/endpoints/{endpoint_id}
Path Parameters:
- `collection_id`: UUID of the parent collection.
- `endpoint_id`: UUID of the endpoint.
Response (200 OK):
```json
{
  "id": "endpoint_uuid",
  "collection_id": "collection_uuid",
  "name": "Get User Profile",
  "method": "GET",
  "url": "https://api.example.com/users/{{user_id}}",
  "headers": {
    "Authorization": "Bearer {{api_token}}",
    "Content-Type": "application/json"
  },
  "body_type": "json",
  "body": null,
  "query_params": {
    "include": "profile,settings"
  },
  "timeout_seconds": 15,
  "auth_config": {
    "type": "bearer",
    "token_env_variable": "API_TOKEN_XYZ"
  },
  "pre_request_script": null,
  "post_request_script": null,
  "created_at": "2025-06-01T10:00:00Z",
  "updated_at": "2025-06-01T10:00:00Z"
}
```
Response (404 Not Found): If the collection or endpoint does not exist.
Description: Retrieves a specific endpoint by its ID within a collection.

#### PUT /api/collections/{collection_id}/endpoints/{endpoint_id}
Path Parameters:
- `collection_id`: UUID of the parent collection.
- `endpoint_id`: UUID of the endpoint.
Request Body: (Similar to POST request body, only include fields to be updated)
```json
{
  "name": "Get User Profile - Updated",
  "timeout_seconds": 25
}
```
Response (200 OK):
```json
{
  "id": "endpoint_uuid",
  "collection_id": "collection_uuid",
  "name": "Get User Profile - Updated",
  "method": "GET",
  "url": "https://api.example.com/users/{{user_id}}",
  // ... other fields ...
  "timeout_seconds": 25,
  "created_at": "2025-06-01T10:00:00Z",
  "updated_at": "2025-06-02T12:00:00Z"
}
```
Response (404 Not Found): If the collection or endpoint does not exist.
Description: Updates an existing endpoint within a collection.

#### DELETE /api/collections/{collection_id}/endpoints/{endpoint_id}
Path Parameters:
- `collection_id`: UUID of the parent collection.
- `endpoint_id`: UUID of the endpoint.
Response (204 No Content):
Response (404 Not Found): If the collection or endpoint does not exist.
Description: Deletes an endpoint from a collection.

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
