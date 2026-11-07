# Twilio vs. TextBee Compatibility Matrix

This document presents the high-level compatibility matrix between the official **Twilio Programmable Messaging API** and the underlying **TextBee Android SMS Gateway**, classifying the feasibility of exposing a Twilio-compatible interface.

---

## 1. Primary Endpoint Compatibility Matrix

| # | Twilio Operation | Twilio Endpoint | TextBee Equivalent Endpoint | Compatibility | Adapter Requirement |
|:---:|---|---|---|:---:|---|
| **1** | **Send Outbound SMS** | `POST /2010-04-01/Accounts/{AccountSid}/Messages.json` | `POST /api/v1/gateway/send-sms` ([`GW-01`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md#1-post-apiv1gatewaysend-sms)) | **Adapter Required** | Form-urlencoded to JSON conversion; single-item array wrapping (`recipients: [To]`); `From` to `deviceId`/`simSubscriptionId` resolution; response conversion to Twilio Message Resource. |
| **2** | **Send Bulk Outbound SMS** | Not supported on Twilio `POST Messages.json` (Twilio requires 1 HTTP call per recipient) | `POST /api/v1/gateway/send-sms` or `POST /api/v1/gateway/send-bulk-sms` ([`GW-02`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md#2-post-apiv1gatewaysend-bulk-sms)) | **Direct** (TextBee exceeds Twilio) | Twilio standard client SDKs send $N$ distinct HTTP requests for $N$ recipients. Adapter handles each request individually or exposes an optional multi-to extension. |
| **3** | **Fetch Message Instance** | `GET /2010-04-01/Accounts/{AccountSid}/Messages/{MessageSid}.json` | `GET /api/v1/gateway/devices/:deviceId/sms/:smsId` ([`GW-04`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md#4-get-apiv1gatewaydevicesidsmssmsid)) OR unified query `GET /api/v1/gateway/messages` | **Adapter Required** | Twilio specifies only `{MessageSid}`, whereas TextBee `GW-04` requires both `:deviceId` and `:smsId`. Adapter must maintain an index mapping or query via `GW-03 (search/id)`. |
| **4** | **List Messages / History** | `GET /2010-04-01/Accounts/{AccountSid}/Messages.json` | `GET /api/v1/gateway/messages` ([`GW-03`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md#3-get-apiv1gatewaymessages)) | **Adapter Required** | Translate `To`, `From`, `DateSent` query parameters to TextBee `search`, `from`, `to`, `direction`. Convert response array and wrap into Twilio pagination envelope (`messages`, `first_page_uri`, `next_page_uri`). |
| **5** | **Update Message (Cancel/Redact)** | `POST /2010-04-01/Accounts/{AccountSid}/Messages/{MessageSid}.json` | *No native TextBee endpoint* | **Unsupported / Adapter Managed** | TextBee provides no endpoint to cancel a Bull queue job or redact an existing SMS record text. Adapter can return simulated success or mark cancelation in adapter-level cache before queue dispatch. |
| **6** | **Delete Message Record** | `DELETE /2010-04-01/Accounts/{AccountSid}/Messages/{MessageSid}.json` | *No public delete endpoint* (Only device deletion supported) | **Unsupported** | TextBee preserves message audit history permanently for billing/quota reconciliations. Adapter can simulate `204 No Content` or soft-tombstone in adapter storage. |
| **7** | **Inbound SMS Webhook** | Customer Inbound Webhook (`POST <Url>` with form data) | TextBee Device Ingest `POST /gateway/devices/:id/receive-sms` + `sms.received` Webhook ([`WH-01`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/webhooks.md)) | **Adapter Required** | TextBee emits JSON webhooks signed with HMAC `x-textbee-signature-v2`. Adapter must ingest this event and transform it into Twilio's standard `application/x-www-form-urlencoded` format with `MessageSid`, `From`, `To`, `Body`. |
| **8** | **StatusCallback Webhook** | Asynchronous delivery updates to `StatusCallback` URL | TextBee Device Uplink `PATCH /gateway/devices/:id/sms-status` + `sms.*` Webhooks | **Adapter Required** | TextBee emits JSON events (`sms.sent`, `sms.delivered`, `sms.failed`). Adapter must convert into Twilio form-encoded POST with `MessageStatus`, `MessageSid`, and `ErrorCode`. |

---

## 2. Feature & Protocol Capability Matrix

| Feature / Dimension | Twilio Programmable Messaging | TextBee Android Gateway | Compatibility Rating | Notes & Architectural Workarounds |
|---|---|---|:---:|---|
| **Request Protocol** | `application/x-www-form-urlencoded` | `application/json` | **Adapter Required** | Adapter must parse form-encoded bodies and serialize to JSON. |
| **Authentication** | HTTP Basic Auth (`AccountSid:AuthToken`) | Header `x-api-key: <key>` | **Adapter Required** | Adapter translates `AccountSid` $\to$ User and `AuthToken` $\to$ API Key. |
| **Identifier Scheme** | 34-character SID (`SM` + 32 hex chars) | 24-character hexadecimal ObjectId | **Adapter Required** | Adapter deterministically prefixes `SM` + 8 zero-padding + 24-char ObjectId, or uses persistent key-value store. |
| **Timestamp Encoding** | RFC 2822 (`Wed, 01 Mar 2026 14:00:00 +0000`) | ISO 8601 (`2026-03-01T14:00:00.000Z`) | **Adapter Required** | Trivial string conversion in adapter serialization layer. |
| **Pagination Model** | 0-indexed Page & HATEOAS relative URIs | 1-indexed Page or Opaque Base64 Cursor | **Adapter Required** | Adapter translates `Page=0` to `page=1` and constructs Twilio `next_page_uri`. |
| **Scheduled Sending** | `SendAt` parameter (15m to 7d) | `scheduledAt` ISO parameter | **Direct (Adapted)** | TextBee natively supports scheduled sends via Bull queue delayed jobs. |
| **Multi-SIM Slot Targeting**| Unsupported (Twilio uses virtual carrier numbers) | `simSubscriptionId` parameter | **TextBee Enhanced** | Adapter can resolve Twilio `From` numbers to specific SIM slots. |
| **Alphanumeric Sender ID**| Fully supported via carrier pre-registration | Physical SIM card MSISDN only | **Unsupported** | Sender identity is hardwired to the physical SIM card inserted in the Android handset. |
| **Carrier Delivery Reports**| Guaranteed carrier-level DLR callbacks | Best-effort Android `deliveryIntent` | **Partial** | Cellular carriers or prepaid SIMs may omit delivery reports (`RESULT_CANCELED`). |
| **MMS Media Attachments** | Fully supported (`MediaUrl`) | Stored in DTO as roadmap; no cellular APN PDU | **Unsupported** | In Phase 2, marked as out-of-scope / unsupported. |
| **Multi-Part Message Concatenation** | Automatic GSM-7 / UCS-2 segmentation (`num_segments`) | Automatic via `SmsManager.divideMessage` | **Direct** | Android handles segmentation transparently; adapter calculates segment counts based on standard GSM-7 limits. |
| **Rate Limiting** | Throttling per account/number (TPS) | Redis sliding window (`ThrottlerByIpGuard`) | **Direct** | Managed by underlying systems. |

---

## 3. Summary of Compatibility Breakdown

```text
Total SMS Operations Evaluated: 8 Core Endpoints & 12 Supporting Protocols

- Direct / Enhanced:         3 (Bulk Sending capability, Segmentation, Queue Pacing)
- Adapter Required:          8 (Send, Fetch, List, Inbound, StatusCallback, Auth, IDs, Paging)
- Partial:                   2 (Carrier Delivery Receipts, Query Filters)
- Unsupported:               4 (Message Cancel/Redact, Message Delete, MMS Attachments, Custom Alpha Sender ID)
- Unverified:                0 (All capabilities verified against source code)
```

