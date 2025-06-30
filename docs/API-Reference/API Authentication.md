## Authentication API

### POST /api/auth/register
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

### POST /api/auth/login
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

### POST /api/auth/refresh
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

### POST /api/auth/logout
Path Parameter: None
Request Body: (Optional, may depend on strategy, e.g., if refresh token is sent to be invalidated)
```json
{
  "refresh": "your_refresh_token"
}
```
Response (204 No Content or 200 OK):
Description: Logs the user out. Invalidates the refresh token on the server-side if applicable. Requires Authorization header with a valid access token.

### POST /api/auth/password-reset/request
Path Parameter: None
Request Body:
```json
{
  "email": "user@example.com"
}
```
Response (200 OK or 202 Accepted):
```json
{
  "message": "If an account with that email exists, a password reset link has been sent."
}
```
Description: Initiates the password reset process for a user. Sends an email with a reset token/link.

### POST /api/auth/password-reset/confirm
Path Parameter: None
Request Body:
```json
{
  "token": "reset_token_from_email",
  "new_password": "newSecurePassword123"
}
```
Response (200 OK):
```json
{
  "message": "Password has been reset successfully."
}
```
Response (400 Bad Request): If token is invalid, expired, or password doesn't meet criteria.
Description: Confirms the password reset using the token and sets a new password.
