# OAuth2 Authentication

## Overview

This section explains how OAuth2 authentication is implemented and used in the backend-repo.

OAuth2 allows third-party applications and clients to securely access protected backend APIs without directly exposing user credentials.

---

# Supported OAuth2 Flows

The backend currently supports the following OAuth2 flows:

| Flow                    | Description                                                    | Recommended Usage                   |
| ----------------------- | -------------------------------------------------------------- | ----------------------------------- |
| Authorization Code Flow | Standard OAuth2 login flow using redirect-based authentication | Web applications                    |
| Client Credentials Flow | Machine-to-machine authentication                              | Internal services and microservices |
| Refresh Token Flow      | Used to renew expired access tokens                            | Long-lived sessions                 |

---

# OAuth2 Configuration

## Environment Variables

Add the following variables to your `.env` file:

```env
OAUTH2_CLIENT_ID=your_client_id
OAUTH2_CLIENT_SECRET=your_client_secret
OAUTH2_REDIRECT_URI=http://localhost:3000/callback
OAUTH2_AUTH_URL=https://provider.com/oauth/authorize
OAUTH2_TOKEN_URL=https://provider.com/oauth/token
OAUTH2_SCOPE=openid profile email
```

---

# Authentication Flow

## Step 1 — Redirect User to Provider

The frontend redirects the user to the OAuth provider authorization endpoint.

Example:

```http
GET /oauth/authorize
```

Example redirect URL:

```txt
https://provider.com/oauth/authorize?
client_id=CLIENT_ID
&redirect_uri=REDIRECT_URI
&response_type=code
&scope=openid%20profile%20email
```

---

## Step 2 — Authorization Callback

After successful authentication, the provider redirects the user back with an authorization code.

Example:

```http
GET /auth/callback?code=AUTHORIZATION_CODE
```

---

## Step 3 — Exchange Code for Token

The backend exchanges the authorization code for an access token.

Example request:

```http
POST /oauth/token
Content-Type: application/json
```

Example payload:

```json
{
  "code": "AUTHORIZATION_CODE",
  "redirect_uri": "http://localhost:3000/callback"
}
```

Example response:

```json
{
  "access_token": "jwt_access_token",
  "refresh_token": "refresh_token",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

---

# Protected Routes

Protected APIs require a valid Bearer token.

Example:

```http
Authorization: Bearer <access_token>
```

Example protected endpoint:

```http
GET /api/user/profile
```

---

# Refresh Token

Use the refresh token endpoint to obtain a new access token.

Example:

```http
POST /auth/refresh
```

Example payload:

```json
{
  "refresh_token": "refresh_token_here"
}
```

---

# Logout

Invalidate the current session and revoke active tokens.

Example:

```http
POST /auth/logout
```

---

# Error Handling

| Error               | Description                             |
| ------------------- | --------------------------------------- |
| invalid_client      | Invalid client credentials              |
| invalid_token       | Token expired or malformed              |
| unauthorized_client | Client is not allowed to perform action |
| access_denied       | User denied authorization               |

---

# Security Recommendations

* Always use HTTPS in production.
* Store client secrets securely.
* Use short-lived access tokens.
* Rotate refresh tokens periodically.
* Validate token signatures server-side.
* Implement rate limiting on auth endpoints.

---

# Example Folder Structure

```txt
backend-repo/
├── auth/
│   ├── oauth2.service.ts
│   ├── oauth2.controller.ts
│   ├── oauth2.middleware.ts
│   └── token.validator.ts
├── routes/
├── middleware/
└── config/
```

---

# Future Improvements

* Add PKCE support.
* Add social login providers.
* Add OpenID Connect discovery.
* Add token revocation endpoint.
* Add role-based authorization mapping.
