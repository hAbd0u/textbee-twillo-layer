# Endpoint Mapping: Send Outbound SMS

## 1. Twilio Endpoint Specification

- **HTTP Method:** `POST`
- **Route:** `/2010-04-01/Accounts/{AccountSid}/Messages.json`
- **Authentication:** HTTP Basic Authentication (`AccountSid` / `AuthToken`)
- **Headers:**
  - `Content-Type: application/x-www-form-urlencoded`
  - `Accept: application/json`
- **Request Body (Form URL Encoded):**

| Parameter | Type | Required | Description |
|---|---|:---:|---|
| `To` | `string` | **Yes** | Destination recipient phone number in E.164 format (e.g. `+12025550199`). |
| `From` | `string` | **Conditional** | Twilio phone number, short code, or alphanumeric sender ID. |
| `MessagingServiceSid` | `string` | **Conditional** | Twilio messaging service identifier (`MG...`). |
| `Body` | `string` | **Conditional** | Plaintext message text (up to 1,600 characters). |
| `StatusCallback` | `string` | No | Customer HTTPS webhook URL for asynchronous status notifications. |
| `ValidityPeriod` | `integer` | No | Maximum time in seconds (1–14400) message may remain queued. |
| `SendAt` | `string` | No | Future ISO 8601 execution timestamp (scheduled sending). |

#### Example Request:
```http
POST /2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages.json HTTP/1.1
Host: api.twilio.com
Authorization: Basic QUNYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYzp5b3VyX2F1dGhfdG9rZW4=
Content-Type: application/x-www-form-urlencoded

To=%2B12025550199&From=%2B12025550144&Body=Your+verification+code+is+839210&StatusCallback=https%3A%2F%2Fmyapp.example.com%2Ftwilio%2Fcallback
```

#### Success Response (`201 Created`):
```json
{
  "account_sid": "ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "api_version": "2010-04-01",
  "body": "Your verification code is 839210",
  "date_created": "Wed, 01 Mar 2026 14:00:00 +0000",
  "date_sent": null,
  "date_updated": "Wed, 01 Mar 2026 14:00:00 +0000",
  "direction": "outbound-api",
  "error_code": null,
  "error_message": null,
  "from": "+12025550144",
  "messaging_service_sid": null,
  "num_media": "0",
  "num_segments": "1",
  "price": null,
  "price_unit": "USD",
  "sid": "SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "status": "queued",
  "subresource_uris": {
    "media": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages/SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Media.json"
  },
  "to": "+12025550199",
  "uri": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages/SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.json"
}
```

---

## 2. TextBee Equivalent Endpoint Specification

- **HTTP Method:** `POST`
- **Route:** `/api/v1/gateway/send-sms` ([`GW-01`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md#1-post-apiv1gatewaysend-sms))
- **Source References:**
  - Controller: [`api/src/gateway/gateway.controller.ts:321`](file:///e:/projects/textbee/api/src/gateway/gateway.controller.ts#L321)
  - Service: [`api/src/gateway/gateway.service.ts`](file:///e:/projects/textbee/api/src/gateway/gateway.service.ts)
  - Validation: [`api/src/gateway/recipients.ts`](file:///e:/projects/textbee/api/src/gateway/recipients.ts)
- **Authentication:** `x-api-key: <api_key>` in header or `?apiKey=` query param.
- **Headers:** `Content-Type: application/json`
- **Request Body (JSON):**
```json
{
  "message": "Your verification code is 839210",
  "recipients": ["+12025550199"],
  "deviceId": "64bf800100010fedcba98765",
  "simSubscriptionId": 1,
  "scheduledAt": "2026-03-01T14:00:00.000Z"
}
```
- **Success Response (`200 OK` - Queued Mode):**
```json
{
  "data": {
    "success": true,
    "message": "SMS added to queue for processing",
    "smsBatchId": "65cb76543210fedcba987654",
    "recipientCount": 1,
    "estimatedCompletionAt": "2026-03-01T14:00:05.000Z"
  }
}
```

---

## 3. Compatibility Analysis

### 3.1 Content-Type & Payload Mismatch
- **Twilio:** Standard HTML form encoded (`application/x-www-form-urlencoded`).
- **TextBee:** JSON encoded (`application/json`).
- **Adapter Transformation:** Adapter must parse incoming `req.body` as form fields and serialize them into a JSON payload for TextBee.

### 3.2 Field & Hierarchy Translation
1. `To` $\to$ `recipients: [To]`: Twilio accepts a single scalar recipient per call. TextBee requires an array of recipient strings.
2. `Body` $\to$ `message`: Direct string assignment.
3. `From` $\to$ `deviceId` + `simSubscriptionId`:
   - Twilio specifies the sender phone number in `From`.
   - TextBee routes messages via physical Android phones (`deviceId`) and optional carrier SIM slots (`simSubscriptionId`).
   - If `From` matches the phone number of a SIM card reported in an enrolled device's `simInfo` ([`device.schema.ts`](file:///e:/projects/textbee/api/src/gateway/schemas/device.schema.ts#L210)), the adapter maps `From` to that specific `deviceId` and `simSubscriptionId`.
   - If `From` is omitted or does not match, the adapter delegates device resolution to TextBee's native fallback logic (default enabled device).
4. `StatusCallback`:
   - TextBee does not accept a per-request `StatusCallback` URL on `POST /gateway/send-sms`.
   - Instead, TextBee dispatches webhook events to account-level subscriptions configured via `POST /webhook/subscriptions`.
   - **Adapter Role:** The adapter records the mapping `(smsBatchId / smsId) -> StatusCallbackURL` in its internal store. When TextBee fires its account-level webhook, the adapter relays the event to the exact `StatusCallback` URL requested by the client.
5. `SendAt` $\to$ `scheduledAt`: ISO 8601 string passed directly.

### 3.3 Response Status & Structure Mismatch
- **Twilio:** Responds with HTTP `201 Created` returning the full Message Resource object with `status: "queued"`, a generated `sid` (`SM...`), and `date_created`.
- **TextBee:** Responds with HTTP `200 OK` returning `{ data: { success: true, smsBatchId: "..." } }`.
- **Adapter Transformation:** The adapter synthesizes a compliant Twilio Message Resource:
  - Generates or maps `sid` from `smsBatchId` (or the underlying single recipient's `smsId`).
  - Sets `status = "queued"`.
  - Sets `date_created = new Date().toRFC2822String()`.
  - Formats `uri = "/2010-04-01/Accounts/{AccountSid}/Messages/{sid}.json"`.
  - Returns HTTP `201 Created`.

---

## 4. Implementation Feasibility

| Assessment | Details |
|---|---|
| **Feasibility Level** | **Adapter Required (High Feasibility)** |
| **Prerequisites** | 1. Form-url-encoded body parser.<br/>2. Device/SIM phone number lookup table.<br/>3. Per-message `StatusCallback` routing registry. |
| **Backend Changes Required?** | None. Completely implementable as an external facade. |

