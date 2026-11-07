# TextBee Architectural Findings, Technical Debt & Architectural Gaps

This document synthesizes critical architectural findings, discovered technical debt, security nuances, and protocol gaps identified during the reverse engineering of the TextBee platform. These insights provide the essential foundation for subsequent architectural phases.

---

## 1. Discovered Technical Debt & Deprecations

### 1.1 Deprecated Gateway Endpoints
The backend codebase maintains two legacy camelCase endpoints for backwards compatibility with earlier Android client builds:
- **`POST /gateway/devices/:deviceId/sendSMS`** ([`gateway.controller.ts:316`](file:///e:/projects/textbee/api/src/modules/gateway/controllers/gateway.controller.ts#L316))
  - Replaced by kebab-case: `POST /gateway/devices/:deviceId/send-sms`.
  - Both routes delegate to the identical `gatewayService.sendSMS` handler.
- **`GET /gateway/devices/:deviceId/getReceivedSMS`** ([`gateway.controller.ts:348`](file:///e:/projects/textbee/api/src/modules/gateway/controllers/gateway.controller.ts#L348))
  - Replaced by query filter: `GET /gateway/devices/:deviceId/messages?type=incoming`.
  - Returns paginated incoming messages.

### 1.2 Validation Infrastructure Inconsistencies
- Unlike idiomatic NestJS enterprise applications, TextBee does not register a global `ValidationPipe` with `{ whitelist: true, forbidNonWhitelisted: true }` in `main.ts`.
- Validation is applied selectively via class-validator decorators on specific DTOs, but several endpoints rely on manual in-handler validation routines (e.g. `pickDeviceWritableFields` in `device.service.ts` or custom regex assertions like `assertValidRecipients`).
- Consequence: Extraneous JSON keys submitted in payloads may bypass stripping unless explicitly sanitized before persistence.

### 1.3 Legacy Bcrypt Fallback on API Key Lookup
- To accelerate key verification, TextBee introduced a deterministic SHA-256 indexed hash (`hashedApiKeySha256`).
- However, for backwards compatibility with keys created before this migration, `ApiKeyService.validateApiKey` falls back to querying active keys and running `bcrypt.compareSync(key, record.hashedApiKey)`.
- If an attacker hammers an invalid key, the fallback loop can cause CPU consumption spikes on the Node.js event loop until all legacy keys are fully migrated or pruned.

---

## 2. Hardware Gateway Architectural Constraints

### 2.1 Asynchronous Delivery & Telco Carrier Asymmetry
In cloud-native carrier messaging architectures, delivery confirmation is guaranteed via SMPP submit_sm_resp and deliver_sm PDUs. In an Android cellular gateway:
1. **Radio Confirmation vs Recipient Handset Receipt:**
   - `sentIntent` confirms only that the Android phone's cellular baseband modem transmitted the PDU to the local cell tower (`RESULT_OK`).
   - `deliveryIntent` depends on whether the destination carrier supports and returns an SMS-DELIVERY-REPORT PDU.
2. **Prepaid / MVNO Carrier Dropping:**
   - Many prepaid SIMs and mobile virtual network operators (MVNOs) silently discard delivery receipts or return `Activity.RESULT_CANCELED`.
   - TextBee implements a non-downgrade policy: messages remain in `sent` status, appending a warning entry to `issueHistory` rather than transitioning to `failed`.

### 2.2 OEM Battery Killer & Doze Mode Disruption
- Android OEMs (Xiaomi, Huawei, Samsung, Oppo) employ aggressive background execution limits that freeze network sockets and suppress FCM high-priority data pushes when devices enter deep sleep (Doze mode).
- TextBee mitigates this through:
  - Sticky foreground notification service (`StickyNotificationService.kt`).
  - Active battery optimization whitelist checks (`isIgnoringBatteryOptimizations`).
  - Scheduled recovery polling on heartbeat (`GET /gateway/devices/:id/messages/pending`), throttled to 25 messages per 5-minute cooldown.

### 2.3 Carrier-Enforced Outbound Throttling
- Sending high-volume SMS bursts through a single consumer SIM card triggers carrier anti-spam firewalls (resulting in `SmsManager.RESULT_ERROR_LIMIT_EXCEEDED` or silent line disconnection).
- TextBee addresses this via:
  - Bull queue wave pacing (`sms-send.processor.ts`).
  - Configurable hardware send delay (`smsSendDelaySeconds`, defaulting to 5 seconds).
  - Multi-SIM rotation support.

---

## 3. Security & Integration Nuances

### 3.1 Webhook Dispatch & SSRF Considerations
- When clients register webhook endpoints via `POST /webhook/subscriptions`, the server verifies the URL scheme (`https://` in production).
- However, there is no network-level DNS resolution check or private IP address blocklisting (e.g. blocking loopback `127.0.0.1`, link-local `169.254.169.254`, or private subnets `10.0.0.0/8`, `192.168.0.0/16`).
- Recommendation: Ensure future egress proxies validate target IPs post-DNS resolution to eliminate Server-Side Request Forgery (SSRF) attack vectors.

### 3.2 Webhook Signature Migration (v1 $\to$ v2)
- TextBee supports two signature headers:
  - `x-textbee-signature` (v1): Plain HMAC-SHA256 of the raw body using the subscription secret. Susceptible to replay attacks if timestamps are not validated independently.
  - `x-textbee-signature-v2`: Formatted as `t=<timestamp>,v1=<hex_hmac>`, where the HMAC is computed over `${timestamp}.${rawBody}`. This provides robust replay attack prevention within a recommended 5-minute tolerance window.

### 3.3 Keyset Cursor Pagination
- The platform avoids `skip`-based MongoDB pagination for message queries, using opaque base64-encoded cursors constructed from `{ createdAt: Date, _id: ObjectId }`.
- This ensures constant-time $O(1)$ database execution performance across millions of historical SMS records.

