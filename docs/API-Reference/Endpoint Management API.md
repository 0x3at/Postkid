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
