# TextBee API Endpoints: SMS Messaging & History

This document details all 15 SMS-related endpoints implemented in `api/src/gateway/gateway.controller.ts`, spanning modern account-level endpoints, device status reporting, missed message recovery, and deprecated device-scoped routes.

---

## 1. POST /api/v1/gateway/send-sms

- **Endpoint ID:** `GW-01`
- **Method:** `POST`
- **Path:** `/api/v1/gateway/send-sms`
- **Classification:** Public Developer API (Primary Sending Endpoint)
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:321`
  - Service: `api/src/gateway/gateway.service.ts:sendSMSDeviceless` / `sendSMS`
  - Validation: `api/src/gateway/recipients.ts:assertValidRecipients`
  - DTO: `SendSMSRequestDTO` (`api/src/gateway/gateway.dto.ts:377`)

### Purpose
Dispatches an SMS message to one or more recipient phone numbers. If `deviceId` is omitted, the backend dynamically resolves the caller's default device or most recently active device.

### Request Headers
- `Content-Type: application/json`
- `x-api-key: <api_key>` (or Bearer JWT)

### Request Body (`SendSMSRequestDTO`)
```json
{
  "message": "Your one-time verification code is 492019",
  "recipients": ["+12025550199", "+12025550188"],
  "deviceId": "64bf800100010fedcba98765",
  "simSubscriptionId": 1,
  "scheduledAt": "2026-03-01T15:30:00.000Z"
}
```
| Field | Type | Required | Default | Validation & Description |
|---|---|---|---|---|
| `message` | `string` | **Yes** | — | Plain text message body. If blank, returns 400 (`'Message cannot be blank'`). Legacy alias: `smsBody`. |
| `recipients` | `string[]` | **Yes** | — | Array of recipient phone numbers. Rejected if empty or if any element contains alphabetical characters (`assertValidRecipients`). Whitespace is automatically stripped. Legacy alias: `receivers`. |
| `deviceId` | `string` | No | Default device | Target Android device ObjectId. If omitted, falls back to `isDefault: true`, then latest active `lastHeartbeat`. |
| `simSubscriptionId`| `number` | No | System default | Android SIM subscription ID from `simInfo`. Directs Android to use a specific SIM card. |
| `scheduledAt` | `string` | No | Immediate | ISO 8601 future timestamp. Requires Redis/Bull queue (`smsQueueService.isQueueEnabled()`). |

### Execution Flow & Side Effects
1. Validates device existence, ownership, and that `device.enabled === true`.
2. Rejects invalid recipients (non-numeric, alphabet letters, template tags).
3. Calls `BillingService.canPerformAction(..., 'send_sms', recipients.length)`: checks daily and monthly limits.
4. Checks device permission history (`reportsSmsPermissionOff`). If the phone reported permission off within 7 days, attaches a non-blocking `warning`.
5. Creates `SMSBatch` record with status `pending`.
6. Creates individual `SMS` records for each recipient in `SMS` collection.
7. **Queued Mode:** Adds job to Bull queue `sms-send` with wave pacing calculated by `smsSendDelaySeconds`. Batch status becomes `processing`.
8. **Direct Mode:** Invokes `firebaseAdmin.messaging().sendEach()` pushing FCM message to device push token. Increments `device.sentSMSCount`.

### Responses
#### Success (HTTP 200 OK - Queued Mode)
```json
{
  "data": {
    "success": true,
    "message": "SMS added to queue for processing",
    "smsBatchId": "65cb76543210fedcba987654",
    "recipientCount": 2,
    "estimatedCompletionAt": "2026-03-01T15:30:10.000Z",
    "warning": {
      "code": "SMS_PERMISSION_MISSING",
      "message": "The textbee app on this phone last reported the SMS permission as off. The message fails if it is still off."
    }
  }
}
```

#### Success (HTTP 200 OK - Direct Direct FCM Mode)
```json
{
  "data": {
    "responses": [
      { "success": true, "messageId": "projects/textbee/messages/0:178929..." }
    ],
    "successCount": 2,
    "failureCount": 0
  }
}
```

#### Errors
- **HTTP 400 Bad Request:**
  - Device disabled: `{ "success": false, "error": "Device does not exist or is not enabled" }`
  - Blank message: `{ "success": false, "error": "Message cannot be blank" }`
  - Invalid recipient: `{ "success": false, "error": "Recipient is not a phone number: \"INVALID\"" }`
  - Queue required for scheduling: `{ "success": false, "error": "SMS scheduling requires queue to be enabled" }`
- **HTTP 401 Unauthorized:** Missing or invalid API key.
- **HTTP 403 Forbidden:** Account blocked (`isBanned: true`).
- **HTTP 404 Not Found:** Specified `deviceId` not found on account.
- **HTTP 429 Too Many Requests:** Daily or monthly plan quota exhausted.

---

## 2. POST /api/v1/gateway/send-bulk-sms

- **Endpoint ID:** `GW-02`
- **Method:** `POST`
- **Path:** `/api/v1/gateway/send-bulk-sms`
- **Classification:** Public Developer API (Bulk Sending)
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:355`
  - Service: `api/src/gateway/gateway.service.ts:sendBulkSMSDeviceless` / `sendBulkSMS`
  - DTO: `SendBulkSMSRequestDTO` (`api/src/gateway/gateway.dto.ts:388`)

### Purpose
Submits a batch of heterogeneous messages, where each message entry defines its own distinct text, recipient array, and target SIM.

### Request Body (`SendBulkSMSRequestDTO`)
```json
{
  "deviceId": "64bf800100010fedcba98765",
  "messageTemplate": "Bulk Notification Batch #104",
  "messages": [
    {
      "message": "Hi Alice, your order #101 has shipped!",
      "recipients": ["+12025550111"],
      "simSubscriptionId": 1
    },
    {
      "message": "Hi Bob, your order #102 has shipped!",
      "recipients": ["+12025550222"],
      "simSubscriptionId": 1
    }
  ]
}
```

### Responses
- **HTTP 200 OK:** Returns `SendSMSResponseDTO` containing `smsBatchId` and recipient metrics.
- **HTTP 400 Bad Request:** Malformed message list, unverified email, or disabled device.
- **HTTP 429 Too Many Requests:** Batch exceeds plan bulk send limits or account quota.

---

## 3. GET /api/v1/gateway/messages

- **Endpoint ID:** `GW-03`
- **Method:** `GET`
- **Path:** `/api/v1/gateway/messages`
- **Classification:** Public Developer API (Unified History)
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:584`
  - Query Parser: `api/src/gateway/message-query.ts:parseMessageQuery`
  - Service: `api/src/gateway/gateway.service.ts:getMessagesForUser`
  - DTO: `MessageListResponseDTO` (`api/src/gateway/gateway.dto.ts:1396`)

### Purpose
Retrieves unified sent and received SMS history across all devices on the account. Supports offset-based pagination or cursor-based polling, direction filters, status filters, and ISO timestamp bounds.

### Query Parameters
| Parameter | Type | Required | Default | Allowed Values / Format |
|---|---|---|---|---|
| `deviceIds` | `string` | No | All devices | Comma-separated MongoDB ObjectIds (max 100). |
| `direction` | `string` | No | `'all'` | `all`, `sent`, `received` (Matches message direction). |
| `type` | `string` | No | — | Deprecated alias for `direction`. |
| `smsBatchId`| `string` | No | — | Filter by batch ObjectId. |
| `status` | `string` | No | — | `pending`, `dispatched`, `sent`, `delivered`, `failed`, `unknown`, `received` |
| `search` | `string` | No | — | Substring search against body text and recipient/sender phone. |
| `from` | `string` | No | — | Inclusive lower bound on `createdAt` (ISO-8601 with timezone or `YYYY-MM-DD`). |
| `to` | `string` | No | — | Exclusive upper bound on `createdAt` (ISO-8601 with timezone or `YYYY-MM-DD`). |
| `order` | `string` | No | `'desc'` | `'desc'` (newest first) or `'asc'` (for polling). |
| `page` | `integer` | No | `1` | Page number ($\ge 1$). Mutually exclusive with `cursor`. |
| `limit` | `integer` | No | `50` | Number of items per page ($\ge 1, \le 100$). |
| `cursor` | `string` | No | — | Opaque base64 cursor from previous page `nextCursor`. |

### Responses
#### Success (HTTP 200 OK - Page Mode)
```json
{
  "data": [
    {
      "_id": "65cb888888880fedcba98765",
      "device": "64bf800100010fedcba98765",
      "smsBatch": "65cb76543210fedcba987654",
      "message": "Your verification code is 492019",
      "type": "sent",
      "direction": "sent",
      "status": "delivered",
      "recipient": "+12025550199",
      "requestedAt": "2026-02-05T14:00:00.000Z",
      "dispatchedAt": "2026-02-05T14:00:02.000Z",
      "sentAt": "2026-02-05T14:00:05.000Z",
      "deliveredAt": "2026-02-05T14:00:08.000Z",
      "createdAt": "2026-02-05T14:00:00.000Z",
      "updatedAt": "2026-02-05T14:00:08.000Z"
    }
  ],
  "meta": {
    "page": 1,
    "limit": 50,
    "total": 1,
    "pages": 1
  }
}
```

---

## 4. GET /api/v1/gateway/devices/:id/sms/:smsId

- **Endpoint ID:** `GW-04`
- **Method:** `GET`
- **Path:** `/api/v1/gateway/devices/:id/sms/:smsId`
- **Classification:** Public Developer API
- **Authentication:** `AuthGuard`, `CanModifyDevice`
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:789`
  - Service: `api/src/gateway/gateway.service.ts:getSMSById`
  - DTO: `SMSResponseDTO` (`api/src/gateway/gateway.dto.ts:1955`)

### Purpose
Fetches a single SMS record by its ID and device ID, including delivery status timestamps, SIM used, and error history.

### Responses
- **HTTP 200 OK:** `{ "data": RetrieveSMSDTO }`
- **HTTP 404 Not Found:** Message not found on this device.

---

## 5. GET /api/v1/gateway/devices/:id/sms-batch/:smsBatchId

- **Endpoint ID:** `GW-05`
- **Method:** `GET`
- **Path:** `/api/v1/gateway/devices/:id/sms-batch/:smsBatchId`
- **Classification:** Public Developer API
- **Authentication:** `AuthGuard`, `CanModifyDevice`
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:820`
  - Service: `api/src/gateway/gateway.service.ts:getSmsBatchById`
  - DTO: `SMSBatchResponseDTO` (`api/src/gateway/gateway.dto.ts:2030`)

### Purpose
Retrieves an SMS batch summary along with all recipient messages contained in the batch. Counters (`successCount`, `failureCount`, `pendingCount`, `dispatchedCount`, `deliveredCount`) are recalculated directly from underlying messages.

### Responses
#### Success (HTTP 200 OK)
```json
{
  "data": {
    "smsBatch": {
      "_id": "65cb76543210fedcba987654",
      "device": "64bf800100010fedcba98765",
      "recipientCount": 2,
      "successCount": 2,
      "failureCount": 0,
      "pendingCount": 0,
      "dispatchedCount": 0,
      "sentCount": 0,
      "deliveredCount": 2,
      "status": "completed",
      "createdAt": "2026-02-05T14:00:00.000Z"
    },
    "smsList": [
      {
        "_id": "65cb888888880fedcba98765",
        "recipient": "+12025550199",
        "status": "delivered"
      }
    ]
  }
}
```

---

## 6. GET /api/v1/gateway/devices/:id/messages/pending

- **Endpoint ID:** `GW-06`
- **Method:** `GET`
- **Path:** `/api/v1/gateway/devices/:id/messages/pending`
- **Classification:** Android Device Internal (Missed Push Recovery)
- **Authentication:** `AuthGuard`, `CanModifyDevice`
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:735`
  - Service: `api/src/gateway/gateway.service.ts:claimPendingMessages`
  - Android Client: `RecoveryPoll.kt:45`

### Purpose
Polled by Android devices when `heartbeat` reports `pendingCount > 0`. Atomically claims up to 25 outbound messages that have been waiting > 30 minutes (where FCM pushes may have been dropped or delayed). Rate-limited to once every 5 minutes per device (`POLL_COOLDOWN_MS`).

### Responses
- **HTTP 200 OK:** `{ "data": [ { "smsId", "smsBatchId", "message", "recipients", "simSubscriptionId" } ] }`
- **HTTP 403 Forbidden:** App version code $< 20$ or recovery poll disabled.
- **HTTP 429 Too Many Requests:** Polled within the 5-minute cooldown period.

---

## 7. PATCH /api/v1/gateway/devices/:id/sms-status

- **Endpoint ID:** `GW-07`
- **Method:** `PATCH`
- **Path:** `/api/v1/gateway/devices/:id/sms-status`
- **Classification:** Android Device Internal (Status Callback)
- **Authentication:** `AuthGuard`, `CanModifyDevice`
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:753`
  - Service: `api/src/gateway/gateway.service.ts:updateSMSStatus`
  - State Decision: `api/src/gateway/status-report-decision.ts:decideStatusReport`
  - Android Client: `SMSStatusUpdateWorker.kt:63`
  - DTO: `UpdateSMSStatusDTO` (`api/src/gateway/gateway.dto.ts:1403`)

### Purpose
Called by the Android gateway when mobile carrier transmission outcomes are reported. Moves messages to `sent`, `delivered`, or `failed`, updates telemetry, recalculates batch stats, and emits outgoing webhooks.

### Request Body (`UpdateSMSStatusDTO`)
```json
{
  "smsId": "65cb888888880fedcba98765",
  "smsBatchId": "65cb76543210fedcba987654",
  "status": "delivered",
  "sentAtInMillis": 1789295405000,
  "deliveredAtInMillis": 1789295408000,
  "pushReceivedAtInMillis": 1789295401000,
  "sendAttemptedAtInMillis": 1789295403000,
  "reportAttempt": 1,
  "simUsed": {
    "subscriptionId": 1,
    "slotIndex": 0,
    "carrierName": "T-Mobile",
    "countryIso": "us"
  }
}
```
If failed:
```json
{
  "smsId": "65cb888888880fedcba98765",
  "smsBatchId": "65cb76543210fedcba987654",
  "status": "failed",
  "failedAtInMillis": 1789295405000,
  "errorCode": "RADIO_OFF",
  "errorMessage": "Airplane mode active"
}
```

### Responses
- **HTTP 200 OK:** `{ "data": { "success": true, "message": "SMS status updated successfully" } }`
- **HTTP 400 Bad Request:** Unknown status value (must be `sent`, `delivered`, `failed`, or `delivery_failed`).
- **HTTP 403 Forbidden:** SMS does not belong to the calling device.
- **HTTP 404 Not Found:** SMS or device does not exist.

---

## 8. POST /api/v1/gateway/devices/:id/receive-sms

- **Endpoint ID:** `GW-08`
- **Method:** `POST`
- **Path:** `/api/v1/gateway/devices/:id/receive-sms`
- **Classification:** Android Device Internal (Incoming SMS Ingest)
- **Authentication:** `AuthGuard`, `CanModifyDevice`
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:438`
  - Service: `api/src/gateway/gateway.service.ts:receiveSMS`
  - Android Client: `SMSReceivedWorker.kt:68`
  - DTO: `ReceivedSMSDTO` (`api/src/gateway/gateway.dto.ts:458`)

### Purpose
Called by Android gateway app when an incoming SMS is received via cellular radio. Stores message, updates received counters, and triggers `MESSAGE_RECEIVED` webhook events.

### Request Body (`ReceivedSMSDTO`)
```json
{
  "sender": "+12025550177",
  "message": "YES, I confirm my appointment",
  "receivedAtInMillis": 1789295900000,
  "simUsed": {
    "subscriptionId": 1,
    "slotIndex": 0,
    "carrierName": "T-Mobile"
  }
}
```

### Key Business Logic
1. **Empty Message / Sender:** Returns HTTP 200 `{ ignored: true, reason: 'empty_message' }` to acknowledge receipt without storing junk.
2. **Deduplication:** Checks existing SMS records within $\pm 5$ seconds tolerance (`toleranceMs = 5000`) on `(device, type: 'received', sender, message)`. If duplicate exists, returns existing record without duplicate webhook emission.
3. **Billing Over-Quota:** If user exceeds plan inbound limits, message is stored with `overLimit: true`.
4. **Late Upload Handling:** If uploaded $>24$ hours late, `createdAt` is backdated to `receivedAt` and real upload time is kept in `originalCreatedAt`. If uploaded $>48$ hours late, webhooks are suppressed.

### Responses
- **HTTP 200 OK:** Returns created `SMS` document or `{ ignored: true }`.

---

## 9. Deprecated & Legacy Gateway Endpoints

The following routes are implemented in `GatewayController` for backward compatibility with older Android client releases and early integrations. They forward internally to their modern replacements:

| Method | Deprecated Path | Replacement Path | Reason for Deprecation |
|---|---|---|---|
| `POST` | `/api/v1/gateway/devices/:id/send-sms` | `POST /api/v1/gateway/send-sms` | Replaced by deviceless route where `deviceId` travels optionally in body. |
| `POST` | `/api/v1/gateway/devices/:id/sendSMS` | `POST /api/v1/gateway/send-sms` | Legacy camelCase route naming alias. |
| `POST` | `/api/v1/gateway/devices/:id/send-bulk-sms` | `POST /api/v1/gateway/send-bulk-sms` | Replaced by deviceless bulk route. |
| `POST` | `/api/v1/gateway/devices/:id/receiveSMS` | `POST /api/v1/gateway/devices/:id/receive-sms`| Legacy camelCase route naming alias. |
| `GET` | `/api/v1/gateway/devices/:id/get-received-sms` | `GET /api/v1/gateway/messages?direction=received` | Replaced by account-level unified query endpoint. |
| `GET` | `/api/v1/gateway/devices/:id/getReceivedSMS` | `GET /api/v1/gateway/messages?direction=received` | Legacy camelCase route naming alias. |
| `GET` | `/api/v1/gateway/devices/:id/messages` | `GET /api/v1/gateway/messages` | Replaced by account-level unified query endpoint. |

