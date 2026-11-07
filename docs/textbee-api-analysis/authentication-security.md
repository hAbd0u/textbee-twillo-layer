# TextBee Authentication and Security Architecture

## 1. Authentication Mechanisms Overview

TextBee implements a hybrid authentication architecture serving three distinct caller profiles:
1. **Programmatic Developers:** Using API Keys (`x-api-key` header or `?apiKey=` query parameter).
2. **Dashboard Web Users:** Using signed JSON Web Tokens (`Authorization: Bearer <jwt>`) managed through NextAuth.js.
3. **Android Gateway Hardware:** Using the account's API key passed via Retrofit HTTP headers.

```
                    ┌──────────────────────────────────────────────┐
                    │            Incoming HTTP Request             │
                    └──────────────────────┬───────────────────────┘
                                           │
                                           ▼
                             [ ThrottlerByIpGuard ]
                       (Canonical IP Rate Limit: 500/60s)
                                           │
                                           ▼
                                    [ AuthGuard ]
                                           │
                   ┌───────────────────────┴───────────────────────┐
                   │                                               │
        Authorization: Bearer <JWT>                     x-api-key: tb_... / ?apiKey=
                   │                                               │
                   ▼                                               ▼
         [ JwtService.verify() ]                      [ SHA-256 Hashing & Lookup ]
          Extract payload.sub                         Query: hashedApiKeySha256
                   │                                               │
                   ▼                                               ▼
         [ User Lookup by _id ]                       [ Check revokedAt == null ]
                   │                                               │
                   └───────────────────────┬───────────────────────┘
                                           │
                                     User Valid?
                                    /           \
                                  Yes            No
                                  /                \
                    Attach req.user, req.apiKey    Throw HTTP 401 Unauthorized
                    Record AccessFootprint         { code: "AUTH_INVALID" }
                    Return true
```

---

## 2. API Key Architecture and Hashing

### 2.1. Key Generation and Storage Flow
- **Generation:** Defined in `api/src/auth/auth.service.ts:generateApiKey`.
- **Raw Key Exposure:** The unhashed plaintext key (e.g. `tb_live_...`) is returned to the client **exactly once** during the `POST /api/v1/auth/api-keys` response.
- **Dual-Hash Persistence Model:**
  1. `apiKey`: Masked representation storing only prefix/suffix for dashboard listing (e.g., `tb_live_9f8e********`).
  2. `hashedApiKey`: Salted bcrypt hash (`bcrypt.hash(rawKey, 10)`), preserved for legacy verification compatibility.
  3. `hashedApiKeySha256`: Deterministic SHA-256 hex digest (`crypto.createHash('sha256').update(rawKey).digest('hex')`). This enables fast $O(1)$ index lookups in MongoDB without comparing bcrypt hashes across all keys.
- **Revocation:**
  - Revoking a key (`POST /api/v1/auth/api-keys/:id/revoke`) sets `revokedAt = new Date()`.
  - Keys with `revokedAt != null` are immediately rejected by `verifyApiKey`.
  - Usage metrics (`usageCount`, `lastUsedAt`) remain preserved for billing and audit history.
- **Tombstones:** Hard deletion (`DELETE /api/v1/auth/api-keys/:id`) deletes the key and creates an audit record in `ApiKeyTombstone`.

---

## 3. JSON Web Token (JWT) Session Flow

- **Strategy:** Managed via `@nestjs/jwt` and Passport JWT strategy (`api/src/auth/jwt.strategy.ts`).
- **Issuance:** Issued during `POST /auth/login`, `POST /auth/google-login`, and `POST /auth/register`.
- **Secret:** Read from environment variable `JWT_SECRET`.
- **Payload Structure:**
  ```json
  {
    "sub": "64bf876543210fedcba98765",
    "email": "user@example.com",
    "role": "regular",
    "iat": 1789295400,
    "exp": 1789900200
  }
  ```
- **Validation:** `AuthGuard` extracts the token from `Authorization: Bearer <token>`, verifies signature and expiration, and extracts `sub` as the user ObjectId.

---

## 4. Authorization Guards & Scoping

TextBee enforces authorization through declarative NestJS guards:

| Guard Name | File Location | Mechanism & Verification Scope |
|---|---|---|
| `AuthGuard` | `api/src/auth/guards/auth.guard.ts` | Dual-mode credential guard. Resolves user identity from either Bearer JWT or API key. Rejects with `{ error: 'Unauthorized', code: 'AUTH_INVALID' }` (HTTP 401). |
| `JwtAuthGuard` | `api/src/auth/jwt-auth.guard.ts` | Strict JWT session guard. Rejects API keys outright. Reserved for sensitive interactive actions (e.g., `POST /support/request-account-deletion`). |
| `CanModifyDevice` | `api/src/gateway/guards/can-modify-device.guard.ts` | Device ownership validator. Verifies that `params.id` is a valid MongoDB ObjectId and that `Device.user.toString() === req.user.id.toString()`. Throws HTTP 400 for malformed ID, HTTP 404 for foreign device. |
| `CanModifyApiKey` | `api/src/auth/guards/can-modify-api-key.guard.ts` | API Key ownership validator. Validates that `params.id` belongs to `req.user.id`. Prevents cross-tenant key revocation or renaming. |
| `ThrottlerByIpGuard` | `api/src/auth/guards/throttle-by-ip.guard.ts` | Global rate limiter (`APP_GUARD` in `app.module.ts`). Tracks request counts against canonicalized client IPs (500 requests per 60-second window). |

---

## 5. Network Security, Rate Limiting & Client Resolution

### 5.1. Reverse Proxy Trust & Canonical IP Resolution
- In `api/src/main.ts`, `app.set('trust proxy', 1)` is enabled.
- `resolveClientAddress` (`api/src/common/client-address.ts`) extracts client IP from standard proxy headers (`cf-connecting-ip`, `x-real-ip`, `x-forwarded-for`).
- `canonicalAddress`: Normalizes IPv6 addresses to `/64` subnet prefixes to prevent clients from rotating IPv6 host addresses to bypass rate limiting.

### 5.2. Cloudflare Turnstile Anti-Bot Verification
- Sensitive unauthenticated public endpoints (`/auth/login`, `/auth/register`, `/auth/request-password-reset`, `/support/customer-support`, `/support/request-account-deletion`) require a `turnstileToken`.
- `TurnstileService.verify` validates the token with Cloudflare's siteverify API.

---

## 6. Webhook Cryptographic Signatures

When TextBee delivers events to external webhook subscribers, it includes two cryptographic signature headers to prevent tampering and replay attacks:

### Signature Headers
1. `X-Signature`: Legacy HMAC-SHA256 hex digest of the raw request payload:
   $$\text{Hex}(\text{HMAC-SHA256}_{\text{signingSecret}}(\text{rawBody}))$$
2. `X-Textbee-Timestamp`: Current Unix epoch seconds at attempt time ($t$).
3. `X-Textbee-Signature`: Modern Version 2 signature with replay protection:
   $$\text{X-Textbee-Signature: } t=t, v1=\text{Hex}(\text{HMAC-SHA256}_{\text{signingSecret}}(t + \text{"."} + \text{rawBody}))$$

### Freshness & Replay Defense
- TextBee documentation specifies that consumers must reject requests where $|t_{\text{current}} - t| > 300\text{ seconds}$ (5-minute drift tolerance).
- Constant-time comparison (e.g. `crypto.timingSafeEqual` in Node.js or `hmac.compare_digest` in Python) must be used to prevent timing attacks.

---

## 7. Signed Action Links (Email Security)

Sensitive email actions (unsubscribe, card update, yearly plan switch, checkout resume) operate without user logins via signed tokens:
- Generated in `api/src/mail/email-links.ts` and `api/src/billing/email-links.ts`.
- Uses HMAC-SHA256 over `userId + action + expirationTimestamp` keyed by `EMAIL_LINK_SECRET` or `JWT_SECRET`.
- Time-limited (default: 7 to 30 days depending on action).
- GET endpoints for unsubscribe present a confirmation button first; links never mutate database state on GET to prevent email scanner crawlers from inadvertently altering user preferences.

