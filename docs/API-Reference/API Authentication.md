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
