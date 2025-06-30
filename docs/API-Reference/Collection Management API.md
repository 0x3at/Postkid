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
Description: Deletes a collection. Per `../../Specs/Yellow Paper.md`, this should be a soft delete.
