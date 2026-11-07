# TextBee API Endpoints: Webhooks & Event Subscriptions

This document details all 6 endpoints implemented in `api/src/webhook/webhook.controller.ts` providing webhook subscription lifecycle management, HMAC signature specifications, and delivery attempt audit logs.

---

## 1. GET /api/v1/webhooks

- **Endpoint ID:** `WH-01`
- **Method:** `GET`
- **Path:** `/api/v1/webhooks`
- **Classification:** Public Developer API & Dashboard
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/webhook/webhook.controller.ts:79`
  - Service: `api/src/webhook/webhook.service.ts:findWebhooksForUser`
  - DTO: `WebhookListResponseDTO` (`api/src/webhook/webhook.dto.ts:188`)

### Purpose
Lists all active and paused webhook subscriptions belonging to the caller's account.

### Responses
#### Success (HTTP 200 OK)
```json
{
  "data": [
    {
      "_id": "65cb999999990fedcba98765",
      "name": "Production Delivery Webhook",
      "deliveryUrl": "https://api.mycrm.com/textbee/webhook",
      "isActive": true,
      "events": [
        "MESSAGE_RECEIVED",
        "MESSAGE_SENT",
        "MESSAGE_DELIVERED",
        "MESSAGE_FAILED"
      ],
      "signingSecret": "sk_live_very_secret_key_minimum_20_chars",
      "signatureVersion": 2,
      "successfulDeliveryCount": 1240,
      "deliveryFailureCount": 2,
      "deliveryAttemptCount": 1242,
      "lastDeliveryAttemptAt": "2026-02-05T14:10:00.000Z",
      "lastDeliverySuccessAt": "2026-02-05T14:10:01.000Z",
      "createdAt": "2026-01-01T00:00:00.000Z"
    }
  ]
}
```

---

## 2. POST /api/v1/webhooks

- **Endpoint ID:** `WH-02`
- **Method:** `POST`
- **Path:** `/api/v1/webhooks`
- **Classification:** Public Developer API & Dashboard
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/webhook/webhook.controller.ts:220`
  - Service: `api/src/webhook/webhook.service.ts:create`
  - SSRF Validation: `api/src/webhook/webhook.service.ts:validateDeliveryUrl`
  - DTO: `CreateWebhookDto` (`api/src/webhook/webhook.dto.ts:5`)

### Purpose
Registers a new outbound webhook subscription. Validates the destination URL against SSRF vulnerabilities (blocks loopback, private IPv4/IPv6 ranges), enforces signing secret length, and attaches the subscription to the caller.

### Request Body (`CreateWebhookDto`)
```json
{
  "name": "Customer Support Inbound SMS",
  "deliveryUrl": "https://support.example.com/api/v1/sms-events",
  "signingSecret": "secret_passphrase_min_20_characters_long",
  "events": ["MESSAGE_RECEIVED", "MESSAGE_DELIVERED", "MESSAGE_FAILED"]
}
```
| Field | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `name` | `string` | No | Max 64 chars | Descriptive label for dashboard display. |
| `deliveryUrl` | `string` | **Yes** | Valid URL | Publicly routable HTTP/HTTPS URL. Private IPs (127.0.0.1, 10.x, 192.168.x, etc.) are rejected. |
| `signingSecret` | `string` | **Yes** | $\ge 20$ chars | Secret key used to generate HMAC-SHA256 delivery signatures. |
| `events` | `WebhookEvent[]` | **Yes** | Non-empty array | Array of event names to deliver. |

### Supported Event Types (`WebhookEvent`)
- `MESSAGE_RECEIVED`: Received inbound SMS on Android device.
- `MESSAGE_SENT`: Outbound message radio dispatch confirmed by device.
- `MESSAGE_DELIVERED`: Mobile carrier delivery receipt confirmed.
- `MESSAGE_FAILED`: Outbound send failed (radio error, no carrier service, quota limit).
- `UNKNOWN_STATE`: Stale pending message timed out ($>20$m).
- `SMS_STATUS_UPDATED`: Generic status transition notification.

### Responses
- **HTTP 201 Created:** Returns `WebhookResponseDTO` (`{ data: WebhookSubscription }`).
- **HTTP 400 Bad Request:**
  - URL unreachable or resolves to private/loopback IP.
  - Secret shorter than 20 characters (`signingSecret must be at least 20 characters long`).
  - Empty event array or invalid event enum name.
  - User reached plan webhook subscription quota (`getWebhookLimit`).

---

## 3. GET /api/v1/webhooks/:webhookId

- **Endpoint ID:** `WH-03`
- **Method:** `GET`
- **Path:** `/api/v1/webhooks/:webhookId`
- **Classification:** Public Developer API & Dashboard
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/webhook/webhook.controller.ts:194`
  - Service: `api/src/webhook/webhook.service.ts:findOne`

### Path Parameters
- `webhookId` (`string`, Required): MongoDB ObjectId of the webhook subscription.

### Responses
- **HTTP 200 OK:** `{ "data": WebhookSubscriptionDTO }`
- **HTTP 404 Not Found:** Subscription does not exist or belongs to another user.

---

## 4. PATCH /api/v1/webhooks/:webhookId

- **Endpoint ID:** `WH-04`
- **Method:** `PATCH`
- **Path:** `/api/v1/webhooks/:webhookId`
- **Classification:** Public Developer API & Dashboard
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/webhook/webhook.controller.ts:251`
  - Service: `api/src/webhook/webhook.service.ts:update`
  - DTO: `UpdateWebhookDto` (`api/src/webhook/webhook.dto.ts:41`)

### Purpose
Partially updates webhook subscription properties. Passing `isActive: true` re-enables a subscription that was automatically paused by the failure rate cron job.

### Request Body (`UpdateWebhookDto`)
```json
{
  "name": "Updated Webhook Label",
  "isActive": true,
  "events": ["MESSAGE_RECEIVED", "MESSAGE_FAILED"]
}
```

### Responses
- **HTTP 200 OK:** Returns updated `WebhookResponseDTO`.
- **HTTP 400 Bad Request:** Validation errors on modified fields.
- **HTTP 404 Not Found:** Webhook not found.

---

## 5. DELETE /api/v1/webhooks/:webhookId

- **Endpoint ID:** `WH-05`
- **Method:** `DELETE`
- **Path:** `/api/v1/webhooks/:webhookId`
- **Classification:** Public Developer API & Dashboard
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/webhook/webhook.controller.ts:279`
  - Service: `api/src/webhook/webhook.service.ts:remove`

### Purpose
Soft-deletes the webhook subscription (`deletedAt = new Date()`, `isActive = false`). Historical delivery records remain readable via `GET /webhooks/notifications`.

### Responses
- **HTTP 200 OK:** `{ "data": { "success": true, "message": "Webhook deleted successfully" } }`
- **HTTP 404 Not Found:** Webhook not found.

---

## 6. GET /api/v1/webhooks/notifications

- **Endpoint ID:** `WH-06`
- **Method:** `GET`
- **Path:** `/api/v1/webhooks/notifications`
- **Classification:** Public Developer API & Dashboard
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/webhook/webhook.controller.ts:155`
  - Service: `api/src/webhook/webhook.service.ts:findWebhookNotificationsForUser`
  - DTO: `WebhookNotificationListResponseDTO` (`api/src/webhook/webhook.dto.ts:326`)

### Purpose
Retrieves paginated audit logs of webhook delivery attempts made by TextBee to consumer endpoints, including HTTP response codes, latency timestamps, retries, and errors.

### Query Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `page` | `number` | No (Default: 1) | Page index. |
| `limit` | `number` | No (Default: 10) | Records per page. |
| `status` | `string` | No | Filter by attempt state (`pending`, `retrying`, `delivered`, `failed`). |
| `eventType` | `string` | No | Filter by `WebhookEvent` name. |
| `deviceId` | `string` | No | Filter deliveries for messages originating from this device. |
| `webhookSubscriptionId`| `string` | No | Filter deliveries for a specific subscription. |
| `start` | `string` | No | Range start ISO 8601 (applied together with `end`). |
| `end` | `string` | No | Range end ISO 8601. |

### Responses
#### Success (HTTP 200 OK)
```json
{
  "data": {
    "notifications": [
      {
        "_id": "65cb900000000fedcba98765",
        "webhookSubscription": "65cb999999990fedcba98765",
        "event": "MESSAGE_DELIVERED",
        "deliveryUrl": "https://api.mycrm.com/textbee/webhook",
        "httpStatusCode": 200,
        "deliveryAttemptCount": 1,
        "deliveredAt": "2026-02-05T14:10:01.000Z",
        "lastDeliveryAttemptAt": "2026-02-05T14:10:00.000Z",
        "idempotencyKey": "a9876543-210f-edcb-a987-6543210fedcb",
        "payload": {
          "smsId": "65cb888888880fedcba98765",
          "message": "Your verification code is 492019",
          "deviceId": "64bf800100010fedcba98765",
          "webhookSubscriptionId": "65cb999999990fedcba98765",
          "webhookEvent": "MESSAGE_DELIVERED",
          "recipient": "+12025550199",
          "sentAt": "2026-02-05T14:00:05.000Z",
          "deliveredAt": "2026-02-05T14:00:08.000Z",
          "idempotencyKey": "a9876543-210f-edcb-a987-6543210fedcb"
        }
      }
    ],
    "meta": {
      "page": 1,
      "limit": 10,
      "total": 1,
      "pages": 1
    }
  }
}
```

---

## 7. Webhook Delivery Payload & Verification Specification

TextBee sends webhook payloads as HTTP POST requests with raw JSON body and the following cryptographic headers:

### Request Headers Dispatched by TextBee
- `Content-Type: application/json`
- `X-Signature: <hex_hmac>`: Legacy HMAC-SHA256 signature calculated over `<raw_body>`.
- `X-Textbee-Timestamp: <timestamp>`: Unix epoch timestamp in seconds.
- `X-Textbee-Signature: t=<timestamp>,v1=<hex_hmac>`: Modern Signature v2. Signature calculated as:
  $$\text{HMAC-SHA256}(\text{key} = \text{signingSecret}, \text{data} = \text{timestamp} + \text{"."} + \text{raw\_body})$$

### Signature Verification Algorithm
```python
import hmac
import hashlib
import time

def verify_textbee_webhook(raw_body_bytes, signature_header, secret_key, max_drift_seconds=300):
    # signature_header format: "t=1789295400,v1=9f8e7d..."
    parts = dict(item.split("=") for item in signature_header.split(","))
    t = parts.get("t")
    v1 = parts.get("v1")
    
    if not t or not v1:
        return False
        
    # Replay attack prevention
    current_time = int(time.time())
    if abs(current_time - int(t)) > max_drift_seconds:
        return False
        
    # Compute expected signature
    signed_payload = f"{t}.".encode('utf-8') + raw_body_bytes
    expected_v1 = hmac.new(secret_key.encode('utf-8'), signed_payload, hashlib.sha256).hexdigest()
    
    # Constant-time comparison
    return hmac.compare_digest(expected_v1, v1)
```

