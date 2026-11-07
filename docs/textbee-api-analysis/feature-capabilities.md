# TextBee Feature Capabilities & Limitations Matrix

This document provides a comprehensive capability matrix of the TextBee Android SMS Gateway platform, categorizing every messaging, routing, device management, and integration feature.

Each capability is labeled according to the project classification taxonomy:
- **`[Verified]`**: Backed by direct source code implementations in `api/` or `android/`.
- **`[Inferred]`**: Derived logically from related architectural patterns or configurations.
- **`[Unverified]`**: Referenced in comments, DTOs, or roadmaps but lacking full end-to-end execution.
- **`[Unsupported]`**: Explicitly absent, architecturally excluded, or impossible under Android hardware gateway constraints.

---

## 1. Feature Capability Matrix

### 1.1 Outbound SMS Messaging

| Capability | Status | Implementation Details & Source Evidence |
|---|---|---|
| **Single Outbound SMS** | `[Verified]` | Handled via `POST /gateway/devices/:id/send-sms` and `POST /gateway/send-sms`. Persisted to `SMS` collection, pushed to Android via FCM high-priority data messages. ([`gateway.service.ts`](file:///e:/projects/textbee/api/src/modules/gateway/services/gateway.service.ts)) |
| **Bulk Outbound SMS** | `[Verified]` | Accepts array of `recipients`. Batch tracked via `SMSBatch` record. Paced in waves through Redis Bull queue `sms-send` with 5-second inter-message delays. ([`sms-send.processor.ts`](file:///e:/projects/textbee/api/src/modules/gateway/processors/sms-send.processor.ts)) |
| **Deviceless Dispatch Routing** | `[Verified]` | When `POST /gateway/send-sms` is invoked without specifying a device, backend automatically selects an active device owned by the user (preferring `isDefault: true`, followed by least loaded). |
| **Scheduled Sending** | `[Verified]` | Supports future dispatch timestamps via `dispatchDueAt` parameter. Bull queue delays job processing until target timestamp. |
| **Multi-Part / Concatenated SMS** | `[Verified]` | Long messages exceeding 160 characters (GSM 7-bit) or 70 characters (UCS-2) are automatically divided using `SmsManager.divideMessage(...)` and sent via `sendMultipartTextMessage(...)`. ([`SMSHelper.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/helpers/SMSHelper.kt#L45-L53)) |
| **Multi-SIM Slot Selection** | `[Verified]` | Request parameter `simSubscriptionId` routes transmission through specific SIM hardware using Android `SmsManager.getSmsManagerForSubscriptionId(simSubscriptionId)`. |
| **Carrier Transmission Confirmation** | `[Verified]` | Captures Android OS radio confirmation (`Activity.RESULT_OK`) via `SMSStatusReceiver` and updates message status to `sent`. |
| **Handset Delivery Receipts** | `[Verified]` | Captures carrier delivery PDU reports via `SMSStatusReceiver` and updates message status to `delivered`. |
| **Custom Alphanumeric Sender ID** | `[Unsupported]` | Sender identity is physically tied to the carrier SIM card installed in the Android phone; arbitrary sender IDs cannot be injected via the cellular network. |
| **MMS (Multimedia Messaging)** | `[Unsupported]` | DTO comments note: `"Message channel. Currently always sms; mms will appear here when supported."` No MMS PDU construction or APN data transmission is implemented. |
| **RCS (Rich Communication Services)** | `[Unsupported]` | Android does not expose public, third-party accessible APIs to dispatch RCS messages through the Google Messages RCS stack. |

---

### 1.2 Inbound SMS & Two-Way Messaging

| Capability | Status | Implementation Details & Source Evidence |
|---|---|---|
| **Inbound SMS Reception** | `[Verified]` | Captured by Android `SMSBroadcastReceiver.kt` on `android.provider.Telephony.SMS_RECEIVED` action and enqueued via `SMSReceivedWorker`. ([`SMSBroadcastReceiver.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/receivers/SMSBroadcastReceiver.kt)) |
| **Inbound Ingestion API** | `[Verified]` | Uploaded to `POST /gateway/devices/:id/receive-sms`. Persisted with `type = 'incoming'`. |
| **Duplicate Inbound Filtering** | `[Verified]` | Dual-layer deduplication: Client-side SHA-256 fingerprinting in Room database; Server-side check for identical sender + message within a 5-second sliding window. |
| **Inbound Webhook Forwarding** | `[Verified]` | Automatically triggers `sms.received` webhook notifications to user-configured delivery URLs. |
| **Auto-Reply Rules** | `[Unsupported]` | No automated keyword auto-responder or conversational logic exists within the core backend; responses must be orchestrated by external webhook consumers. |

---

### 1.3 Device & Fleet Management

| Capability | Status | Implementation Details & Source Evidence |
|---|---|---|
| **Multi-Device Gateway Pooling** | `[Verified]` | Users can enroll unlimited or plan-restricted Android devices on a single account. |
| **QR Code Quick Pairing** | `[Verified]` | Web dashboard generates QR code containing user API credentials; Android camera scans QR code during onboarding. |
| **Device Hardware Telemetry** | `[Verified]` | Heartbeat transmits battery percentage, charging state, WiFi/cellular network type, RAM utilization, internal flash storage, and Android OS release/API level. |
| **OS Power & Doze Mode Monitoring** | `[Verified]` | Telemetry captures `isIgnoringBatteryOptimizations`, `isDeviceIdleMode`, and `isPowerSaveMode` to detect aggressive OEM battery killer behavior. |
| **Runtime Permission Auditing** | `[Verified]` | Telemetry reports real-time state of `SEND_SMS`, `RECEIVE_SMS`, `READ_PHONE_STATE`, and `POST_NOTIFICATIONS` permissions. |
| **Missed Push Recovery Polling** | `[Verified]` | Handsets automatically poll `GET /devices/:id/messages/pending` (up to 25 messages, 5 min cooldown) if FCM pushes are delayed by Android Doze mode. |
| **Remote Device Restart / Wiping** | `[Unsupported]` | Gateway does not implement device admin / MDM profiles; cannot remotely reboot or wipe the phone. |

---

### 1.4 Webhooks & Integration

| Capability | Status | Implementation Details & Source Evidence |
|---|---|---|
| **Event Subscriptions** | `[Verified]` | Supports subscriptions to `sms.sent`, `sms.delivered`, `sms.failed`, `sms.received`, `device.online`, and `device.offline`. |
| **HMAC-SHA256 Signing (v1 & v2)** | `[Verified]` | Dispatches `x-textbee-signature` and `x-textbee-signature-v2` with timestamped nonces to defeat replay attacks. |
| **Automated Delivery Retries** | `[Verified]` | Bull queue `webhook-delivery` retries transient 5xx errors or network timeouts up to 5 attempts with exponential backoff. |
| **Delivery Audit Log** | `[Verified]` | Persists attempt timestamps, HTTP status codes, and response bodies in `WebhookNotification` collection. |
| **Auto-Disable on Persistent Failure** | `[Verified]` | Background cron automatically disables webhook subscriptions with consecutive failure counts exceeding thresholds. |

---

### 1.5 Security & Rate Limiting

| Capability | Status | Implementation Details & Source Evidence |
|---|---|---|
| **Dual-Mode Authentication** | `[Verified]` | `AuthGuard` transparently accepts JWT session cookies, `Authorization: Bearer <jwt>`, or `x-api-key: <key>` headers. |
| **$O(1)$ Deterministic Key Indexing**| `[Verified]` | Uses SHA-256 indexed hash (`hashedApiKeySha256`) for instant database lookups, preventing full collection scans. |
| **IP-Based Global Rate Limiting** | `[Verified]` | `ThrottlerByIpGuard` applies Redis-backed sliding window rate limits, with canonical IPv6 subnet normalization (`/64`). |
| **Cloudflare Turnstile Anti-Bot** | `[Verified]` | Onboarding and registration routes are protected by server-validated Turnstile CAPTCHA tokens. |
| **Device Ownership Access Control** | `[Verified]` | `CanModifyDevice` NestJS guard strictly prohibits non-owners from accessing or modifying device records. |
| **End-to-End Payload Encryption** | `[Inferred]` | Data model contains `encrypted` and `encryptedMessage` fields; client-side cryptographic handling is partially stubbed. |

