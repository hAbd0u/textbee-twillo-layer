# TextBee API Endpoints: Support & Administration

This document details the support and account administration endpoints implemented in `api/src/support/support.controller.ts`.

---

## 1. POST /api/v1/support/customer-support

- **Endpoint ID:** `SUP-01`
- **Method:** `POST`
- **Path:** `/api/v1/support/customer-support`
- **Classification:** Authenticated Customer Support API
- **Authentication:** `AuthGuard` (API Key / JWT) + Cloudflare Turnstile
- **Source References:**
  - Controller: `api/src/support/support.controller.ts:55`
  - Service: `api/src/support/support.service.ts:createSupportMessage`
  - DTO: `CreateSupportMessageDto` (`api/src/support/dto/create-support-message.dto.ts:18`)

### Purpose
Submits a customer support message or assistance inquiry. Persists the ticket in the `SupportMessage` collection, emails a confirmation to the user, and attaches client IP and User-Agent metadata.

### Request Body (`CreateSupportMessageDto`)
```json
{
  "name": "Jane Doe",
  "email": "user@example.com",
  "phone": "+12025550199",
  "category": "technical",
  "message": "FCM push token appears to drop after overnight battery save.",
  "turnstileToken": "0.xxxxxx"
}
```
| Field | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `name` | `string` | **Yes** | Non-empty | User's full name. |
| `email` | `string` | **Yes** | Valid email | Email address for support communication. |
| `phone` | `string` | No | — | Optional callback number. |
| `category` | `string` | **Yes** | Enum | Category: `general`, `technical`, `billing-and-payments`, `account-deletion`, `other`. |
| `message` | `string` | **Yes** | Non-empty | Ticket description or question. |
| `turnstileToken`| `string` | **Yes** | Non-empty | Cloudflare Turnstile anti-bot token. |

### Responses
- **HTTP 201 Created:** `{ "message": "Support request submitted successfully" }`
- **HTTP 400 Bad Request:** Turnstile token verification failed.
- **HTTP 401 Unauthorized:** Missing or invalid credentials.
- **HTTP 429 Too Many Requests:** Account rate limited for support submissions.

---

## 2. POST /api/v1/support/request-account-deletion

- **Endpoint ID:** `SUP-02`
- **Method:** `POST`
- **Path:** `/api/v1/support/request-account-deletion`
- **Classification:** Authenticated Dashboard Session Only
- **Authentication:** `JwtAuthGuard` (Requires active dashboard JWT session; API keys are rejected) + Turnstile
- **Source References:**
  - Controller: `api/src/support/support.controller.ts:89`
  - Service: `api/src/support/support.service.ts:requestAccountDeletion`
  - DTO: `RequestAccountDeletionDto` (`api/src/support/dto/request-account-deletion.dto.ts:3`)

### Purpose
Initiates an account closure and data deletion request for the currently signed-in user. Updates `User` record with `accountDeletionRequestedAt` and `accountDeletionReason`.

> [!IMPORTANT]
> This endpoint explicitly uses `JwtAuthGuard` rather than `AuthGuard`. Programmatic API keys cannot invoke this endpoint; it requires an interactive user dashboard session.

### Request Body (`RequestAccountDeletionDto`)
```json
{
  "message": "Closing business and migrating to other infrastructure.",
  "turnstileToken": "0.xxxxxx"
}
```

### Responses
- **HTTP 201 Created:** `{ "message": "Account deletion request submitted successfully" }`
- **HTTP 400 Bad Request:** Turnstile token verification failed.
- **HTTP 401 Unauthorized:** Invalid or missing dashboard session token.

