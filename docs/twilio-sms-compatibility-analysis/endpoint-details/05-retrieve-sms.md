# Endpoint Mapping: Retrieve Single SMS

## 1. Twilio Endpoint Specification

- **HTTP Method:** `GET`
- **Route:** `/2010-04-01/Accounts/{AccountSid}/Messages/{MessageSid}.json`
- **Authentication:** HTTP Basic Authentication (`AccountSid` / `AuthToken`)
- **Headers:** `Accept: application/json`
- **Path Parameters:**
  - `AccountSid` (`string`, required): Twilio Account SID (`AC...`).
  - `MessageSid` (`string`, required): Unique message identifier (`SM...`).
- **Success Status:** `200 OK`

#### Example Request:
```http
GET /2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages/SM65cb888888880fedcba98765.json HTTP/1.1
Host: api.twilio.com
Authorization: Basic QUNYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYzp5b3VyX2F1dGhfdG9rZW4=
Accept: application/json
```

#### Success Response (`200 OK`):
```json
{
  "account_sid": "ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "api_version": "2010-04-01",
  "body": "Your verification code is 839210",
  "date_created": "Wed, 01 Mar 2026 14:00:00 +0000",
  "date_sent": "Wed, 01 Mar 2026 14:00:05 +0000",
  "date_updated": "Wed, 01 Mar 2026 14:00:08 +0000",
  "direction": "outbound-api",
  "error_code": null,
  "error_message": null,
  "from": "+12025550144",
  "messaging_service_sid": null,
  "num_media": "0",
  "num_segments": "1",
  "price": "-0.00750",
  "price_unit": "USD",
  "sid": "SM65cb888888880fedcba98765",
  "status": "delivered",
  "subresource_uris": {
    "media": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages/SM65cb888888880fedcba98765/Media.json"
  },
  "to": "+12025550199",
  "uri": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages/SM65cb888888880fedcba98765.json"
}
```

---

## 2. TextBee Equivalent Endpoint Specification

TextBee provides two alternative paths for message retrieval:

### Option A: Direct Device-Scoped Fetch (`GW-04`)
- **Route:** `GET /api/v1/gateway/devices/:deviceId/sms/:smsId` ([`GW-04`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md#4-get-apiv1gatewaydevicesidsmssmsid))
- **Source Reference:** [`api/src/gateway/gateway.controller.ts:789`](file:///e:/projects/textbee/api/src/gateway/gateway.controller.ts#L789)
- **Parameters:** Requires both `:deviceId` and `:smsId`.
- **Response Payload:** Returns single `SMS` document.

### Option B: Account-Level Unified History Query (`GW-03`)
- **Route:** `GET /api/v1/gateway/messages` ([`GW-03`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md#3-get-apiv1gatewaymessages))
- **Source Reference:** [`api/src/gateway/gateway.controller.ts:584`](file:///e:/projects/textbee/api/src/gateway/gateway.controller.ts#L584)
- **Parameters:** Account-level query with `search=:smsId` or cursor filter.

---

## 3. Compatibility Analysis

### 3.1 Addressing Inconsistency
- **Twilio Paradigm:** The client only holds the `MessageSid`. The Twilio cloud resolves the message globally under the `AccountSid`.
- **TextBee Paradigm:** `GW-04` requires the client to supply `:deviceId` in the URL path. An external Twilio client does not know which Android phone transmitted the message.
- **Adapter Resolution Strategy:**
  1. **Deterministic SID Encoding:** The adapter can encode both `deviceId` and `smsId` into the Twilio SID (e.g., base64 or prefixed hex string).
  2. **Adapter State Cache:** The adapter caches `smsId -> deviceId` when the message is created.
  3. **Unified Query Fallback:** The adapter queries `GET /api/v1/gateway/messages` with the ID directly across the user account, eliminating the need for `:deviceId`.

### 3.2 Status Mapping & Normalization
TextBee statuses map to Twilio lifecycle states as follows:

| TextBee `SMS.status` | Condition | Twilio `status` | Notes |
|---|---|---|---|
| `pending` | In MongoDB, awaiting queue worker | `queued` | Initial accepted state |
| `dispatched` | Pushed via FCM, enqueued in Android WorkManager | `sending` | In transit to device |
| `sent` | Carrier cellular radio confirmed `RESULT_OK` | `sent` | Baseband transmitted |
| `delivered` | Carrier delivery receipt confirmed | `delivered` | Terminal success |
| `failed` | Radio error or timeout | `failed` / `undelivered` | Populates `error_code` |
| `unknown` | Timeout sweeper (>20m without Android report) | `undelivered` | Maps to error `30008` |
| `received` | Inbound message captured from handset radio | `received` | Inbound direction |

### 3.3 Timestamp & Currency Serialization
- TextBee `createdAt` $\to$ Twilio `date_created` (converted to RFC 2822).
- TextBee `sentAt` $\to$ Twilio `date_sent` (converted to RFC 2822).
- TextBee `updatedAt` $\to$ Twilio `date_updated` (converted to RFC 2822).
- Twilio `price`: TextBee SIM gateway utilizes the user's mobile carrier plan, incurring no per-message credit in TextBee. Set to `"0.00000"` or `null` with `price_unit: "USD"`.

---

## 4. Implementation Feasibility

| Assessment | Details |
|---|---|
| **Feasibility Level** | **Adapter Required (High Feasibility)** |
| **Prerequisites** | Adapter must resolve the message via account-level `GET /gateway/messages` or maintain a local `(MessageSid -> deviceId)` mapping cache. |
| **Backend Changes Required?** | None. |

