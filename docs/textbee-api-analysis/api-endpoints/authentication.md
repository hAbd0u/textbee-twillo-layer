# TextBee API Endpoints: Authentication & Identity

This document details all 16 endpoints in `api/src/auth/auth.controller.ts` providing user registration, session management, identity verification, onboarding progression, and full API key administration.

---

## 1. POST /api/v1/auth/login

- **Endpoint ID:** `AUTH-01`
- **Method:** `POST`
- **Path:** `/api/v1/auth/login`
- **Classification:** Public Session Establishment (Dashboard)
- **Authentication:** None (Public)
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:85`
  - Service: `api/src/auth/auth.service.ts:login`
  - DTO: `LoginInputDTO` (`api/src/auth/auth.dto.ts:277`)

### Purpose
Authenticates a user via email and password, validates Cloudflare Turnstile token if configured, records access footprint telemetry (IP, User-Agent, location), and returns a signed JWT session for dashboard operations.

### Request Headers
- `Content-Type: application/json` (Required)

### Request Body (`LoginInputDTO`)
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "turnstileToken": "0.xxxxxx",
  "client": {
    "device": "desktop",
    "os": "Windows",
    "browser": "Chrome",
    "userAgent": "Mozilla/5.0..."
  }
}
```
| Field | Type | Required | Description |
|---|---|---|---|
| `email` | `string` | **Yes** | Account email (automatically converted to lowercase in database). |
| `password` | `string` | **Yes** | Plaintext password (verified against bcrypt hash). |
| `turnstileToken` | `string` | No | Cloudflare Turnstile anti-bot token (verified server-side if enabled). |
| `client` | `ClientContextDTO` | No | Optional telemetry metadata describing client environment. |

### Responses
#### Success (HTTP 200 OK)
```json
{
  "data": {
    "user": {
      "_id": "64bf876543210fedcba98765",
      "name": "Jane Doe",
      "email": "user@example.com",
      "role": "regular",
      "emailVerifiedAt": "2026-01-15T10:00:00.000Z",
      "isBanned": false,
      "createdAt": "2026-01-10T08:30:00.000Z",
      "updatedAt": "2026-01-15T10:00:00.000Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### Errors
- **HTTP 400 Bad Request:** Turnstile token verification failed (`{ error: 'Turnstile check failed' }`).
- **HTTP 401 Unauthorized:** Invalid email or incorrect password (`{ error: 'Invalid credentials' }`).
- **HTTP 403 Forbidden:** User account has been banned (`isBanned: true`).

---

## 2. POST /api/v1/auth/google-login

- **Endpoint ID:** `AUTH-02`
- **Method:** `POST`
- **Path:** `/api/v1/auth/google-login`
- **Classification:** Public OAuth Session Establishment (Dashboard)
- **Authentication:** None (Validates Google ID Token)
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:110`
  - Service: `api/src/auth/auth.service.ts:loginWithGoogle`
  - DTO: `GoogleLoginInputDTO` (`api/src/auth/auth.dto.ts:304`)

### Purpose
Exchanges a client-obtained Google OAuth ID Token for a TextBee session token. If the user does not exist, an account is automatically provisioned and marked email-verified.

### Request Body (`GoogleLoginInputDTO`)
```json
{
  "idToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6...",
  "attribution": {
    "first": { "source": "google", "medium": "cpc" },
    "last": { "source": "google", "medium": "cpc" }
  },
  "client": { "browser": "Chrome", "os": "Android" }
}
```
| Field | Type | Required | Description |
|---|---|---|---|
| `idToken` | `string` | **Yes** | Google OpenID Connect ID token issued by Google Auth. |
| `attribution` | `AttributionDTO` | No | Marketing and campaign tracking metadata. |
| `client` | `ClientContextDTO` | No | Client device and browser telemetry. |

### Responses
- **HTTP 200 OK:** Returns `AuthSessionResponseDTO` with user profile and JWT.
- **HTTP 401 Unauthorized:** Invalid ID token, mismatched audience, or unverified Google email.

---

## 3. POST /api/v1/auth/register

- **Endpoint ID:** `AUTH-03`
- **Method:** `POST`
- **Path:** `/api/v1/auth/register`
- **Classification:** Public Account Registration
- **Authentication:** None (Public)
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:133`
  - Service: `api/src/auth/auth.service.ts:register`
  - DTO: `RegisterInputDTO` (`api/src/auth/auth.dto.ts:215`)

### Purpose
Registers a new user account, stores bcrypt-hashed password, provisions an initial free tier subscription, sends an email verification code, and establishes an active session.

### Request Body (`RegisterInputDTO`)
```json
{
  "name": "Jane Doe",
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "turnstileToken": "0.xxxxxx",
  "attribution": {
    "source": "meta",
    "campaign": "launch_promo"
  },
  "client": {
    "device": "desktop",
    "os": "macOS"
  }
}
```

### Responses
- **HTTP 201 Created:** Returns `AuthSessionResponseDTO` (`{ data: { user, token } }`).
- **HTTP 400 Bad Request:** Email already in use or Turnstile token verification failed.

---

## 4. GET /api/v1/auth/who-am-i

- **Endpoint ID:** `AUTH-04`
- **Method:** `GET`
- **Path:** `/api/v1/auth/who-am-i`
- **Classification:** Identity Introspection (Dashboard & Android Client)
- **Authentication:** `AuthGuard` (Accepts `Bearer <jwt>` OR `x-api-key` header OR `?apiKey=` query param)
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:156`
  - Android Client: `GatewayApiServiceKt.kt:15`

### Purpose
Returns the complete user document for the authenticated identity. Used by the web dashboard to populate user context and by the Android application during initial setup to confirm valid API key credentials.

### Responses
#### Success (HTTP 200 OK)
```json
{
  "data": {
    "_id": "64bf876543210fedcba98765",
    "name": "Jane Doe",
    "email": "user@example.com",
    "role": "regular",
    "emailVerifiedAt": "2026-01-15T10:00:00.000Z",
    "isBanned": false,
    "onboarding": {
      "currentStepId": "pair_device",
      "skippedStepIds": []
    },
    "createdAt": "2026-01-10T08:30:00.000Z",
    "updatedAt": "2026-01-15T10:00:00.000Z"
  }
}
```
- **HTTP 401 Unauthorized:** Missing, expired, or invalid JWT or API key.

---

## 5. PATCH /api/v1/auth/update-profile

- **Endpoint ID:** `AUTH-05`
- **Method:** `PATCH`
- **Path:** `/api/v1/auth/update-profile`
- **Classification:** Authenticated Dashboard API
- **Authentication:** `AuthGuard`
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:175`
  - DTO: `UpdateProfileInputDTO` (`api/src/auth/auth.dto.ts:364`)

### Request Body (`UpdateProfileInputDTO`)
```json
{
  "name": "Jane Smith",
  "phone": "+12025550199"
}
```

### Responses
- **HTTP 200 OK:** Returns updated `UserResponseDTO` (`{ data: User }`).
- **HTTP 401 Unauthorized:** Invalid session credentials.

---

## 6. POST /api/v1/auth/change-password

- **Endpoint ID:** `AUTH-06`
- **Method:** `POST`
- **Path:** `/api/v1/auth/change-password`
- **Classification:** Authenticated Dashboard API
- **Authentication:** `AuthGuard`
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:198`
  - DTO: `ChangePasswordInputDTO` (`api/src/auth/auth.dto.ts:376`)

### Request Body (`ChangePasswordInputDTO`)
```json
{
  "currentPassword": "OldPassword123!",
  "newPassword": "NewSecurePassword456!"
}
```

### Responses
- **HTTP 200 OK:** `{ "message": "Password changed successfully" }`
- **HTTP 400 Bad Request:** Incorrect `currentPassword`.
- **HTTP 401 Unauthorized:** Invalid authentication.

---

## 7. POST /api/v1/auth/api-keys

- **Endpoint ID:** `AUTH-07`
- **Method:** `POST`
- **Path:** `/api/v1/auth/api-keys`
- **Classification:** API Key Management
- **Authentication:** `AuthGuard`
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:220`
  - Service: `api/src/auth/auth.service.ts:generateApiKey`

### Purpose
Generates a new programmatic API key. The unhashed raw key is returned **once** in the response. The backend computes:
1. Masked display representation (e.g., `tb_live_abc1...****`).
2. SHA-256 deterministic hash stored in `hashedApiKeySha256` for $O(1)$ indexed authentication.
3. Legacy bcrypt hash stored in `hashedApiKey`.

### Responses
#### Success (HTTP 201 Created)
```json
{
  "data": {
    "_id": "64bf899999990fedcba98765",
    "apiKey": "tb_live_9f8e7d6c5b4a392817263544",
    "name": "API Key",
    "usageCount": 0,
    "createdAt": "2026-02-01T12:00:00.000Z"
  },
  "message": "API key generated successfully"
}
```

---

## 8. GET /api/v1/auth/api-keys

- **Endpoint ID:** `AUTH-08`
- **Method:** `GET`
- **Path:** `/api/v1/auth/api-keys`
- **Classification:** API Key Management
- **Authentication:** `AuthGuard`
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:247`

### Query Parameters
| Parameter | Type | Required | Default | Allowed Values |
|---|---|---|---|---|
| `status` | `string` | No | `'active'` | `active`, `revoked`, `all` |

### Responses
#### Success (HTTP 200 OK)
```json
{
  "data": [
    {
      "_id": "64bf899999990fedcba98765",
      "apiKey": "tb_live_9f8e********",
      "name": "Production Server",
      "usageCount": 1542,
      "lastUsedAt": "2026-02-05T09:14:22.000Z",
      "revokedAt": null,
      "createdAt": "2026-02-01T12:00:00.000Z"
    }
  ]
}
```

---

## 9. DELETE /api/v1/auth/api-keys/:id

- **Endpoint ID:** `AUTH-09`
- **Method:** `DELETE`
- **Path:** `/api/v1/auth/api-keys/:id`
- **Classification:** API Key Management
- **Authentication:** `AuthGuard`, `CanModifyApiKey`
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:273`

### Path Parameters
- `id` (`string`, Required): MongoDB ObjectId of the API key.

### Behavior
Completely deletes the API key document from the database and inserts an audit record into the `ApiKeyTombstone` collection.

### Responses
- **HTTP 200 OK:** `{ "message": "API Key Deleted" }`
- **HTTP 400 Bad Request:** Malformed ObjectId.
- **HTTP 404 Not Found:** Key does not exist or belongs to another user.

---

## 10. POST /api/v1/auth/api-keys/:id/revoke

- **Endpoint ID:** `AUTH-10`
- **Method:** `POST`
- **Path:** `/api/v1/auth/api-keys/:id/revoke`
- **Classification:** API Key Management
- **Authentication:** `AuthGuard`, `CanModifyApiKey`
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:296`

### Purpose
Immediately revokes an API key without purging historical usage counters. Sets `revokedAt = new Date()`. Any subsequent API call using this key is rejected with HTTP 401 Unauthorized.

### Responses
- **HTTP 200 OK:** `{ "message": "API Key Revoked" }`
- **HTTP 404 Not Found:** Key not found.

---

## 11. PATCH /api/v1/auth/api-keys/:id/rename

- **Endpoint ID:** `AUTH-11`
- **Method:** `PATCH`
- **Path:** `/api/v1/auth/api-keys/:id/rename`
- **Classification:** API Key Management
- **Authentication:** `AuthGuard`, `CanModifyApiKey`
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:318`
  - DTO: `RenameApiKeyInputDTO` (`api/src/auth/auth.dto.ts:384`)

### Request Body
```json
{
  "name": "Staging Backend Relay"
}
```

### Responses
- **HTTP 200 OK:** `{ "message": "API Key Renamed" }`
- **HTTP 404 Not Found:** Key not found.

---

## 12. PATCH /api/v1/auth/onboarding

- **Endpoint ID:** `AUTH-12`
- **Method:** `PATCH`
- **Path:** `/api/v1/auth/onboarding`
- **Classification:** Dashboard UI State
- **Authentication:** `AuthGuard`
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:337`
  - DTO: `UpdateOnboardingDTO` (`api/src/auth/auth.dto.ts:410`)

### Request Body (`UpdateOnboardingDTO`)
```json
{
  "currentStepId": "send_test_sms",
  "skippedStepIds": ["install_app"]
}
```

### Responses
- **HTTP 200 OK:** Returns updated `UserResponseDTO`.

---

## 13. POST /api/v1/auth/request-password-reset

- **Endpoint ID:** `AUTH-13`
- **Method:** `POST`
- **Path:** `/api/v1/auth/request-password-reset`
- **Classification:** Public Account Recovery
- **Authentication:** None (Public, Turnstile protected)
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:358`
  - DTO: `RequestResetPasswordInputDTO` (`api/src/auth/auth.dto.ts:329`)

### Behavior
Generates a 6-digit numeric OTP with an expiration timestamp (stored in `PasswordReset` collection) and emails the code to the user. Always returns HTTP 200 even if the email does not exist (prevents user enumeration).

### Request Body
```json
{
  "email": "user@example.com",
  "turnstileToken": "0.xxxxxx"
}
```

### Responses
- **HTTP 200 OK:** `{ "message": "Password reset code sent to email" }`
- **HTTP 400 Bad Request:** Turnstile validation failure.

---

## 14. POST /api/v1/auth/reset-password

- **Endpoint ID:** `AUTH-14`
- **Method:** `POST`
- **Path:** `/api/v1/auth/reset-password`
- **Classification:** Public Account Recovery
- **Authentication:** None (Public)
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:377`
  - DTO: `ResetPasswordInputDTO` (`api/src/auth/auth.dto.ts:345`)

### Request Body
```json
{
  "email": "user@example.com",
  "code": "849201",
  "password": "NewSecurePassword789!"
}
```

### Responses
- **HTTP 200 OK:** `{ "message": "Password reset successfully" }`
- **HTTP 400 Bad Request:** Code is expired, invalid, or already consumed.

---

## 15. POST /api/v1/auth/send-email-verification-email

- **Endpoint ID:** `AUTH-15`
- **Method:** `POST`
- **Path:** `/api/v1/auth/send-email-verification-email`
- **Classification:** Identity Verification
- **Authentication:** `AuthGuard`
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:398`

### Purpose
Generates a 6-digit verification code stored in `EmailVerification` collection and dispatches it via AWS SES. Unverified accounts cannot send SMS messages.

### Responses
- **HTTP 200 OK:** `{ "message": "Email verification code sent" }`

---

## 16. POST /api/v1/auth/verify-email

- **Endpoint ID:** `AUTH-16`
- **Method:** `POST`
- **Path:** `/api/v1/auth/verify-email`
- **Classification:** Identity Verification
- **Authentication:** None (Public, accepts email and code)
- **Source References:**
  - Controller: `api/src/auth/auth.controller.ts:419`
  - DTO: `VerifyEmailInputDTO` (`api/src/auth/auth.dto.ts:394`)

### Request Body
```json
{
  "email": "user@example.com",
  "code": "391048"
}
```

### Responses
- **HTTP 200 OK:** `{ "message": "Email verified successfully" }`
- **HTTP 400 Bad Request:** Incorrect or expired verification code.
- **HTTP 404 Not Found:** Account not found.

