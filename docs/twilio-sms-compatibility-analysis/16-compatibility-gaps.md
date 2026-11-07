# Architectural Gaps, Limitations & Workarounds

This document analyzes the fundamental architectural differences, cellular carrier limitations, and unsupported operations between the cloud-based **Twilio Programmable Messaging API** and the **TextBee Android SMS Gateway**.

---

## 1. Summary of Identified Gaps

| Gap Category | Twilio Cloud Behavior | TextBee Gateway Behavior | Severity | Compatibility Impact & Workaround |
|---|---|---|:---:|---|
| **Sender Identity (Alphanumeric)** | Supports custom alphanumeric strings (e.g., `MYCOMPANY`) where permitted by national regulators. | Hardwired to the physical phone number (MSISDN) of the SIM card inserted in the Android handset. | **High** | **Unsupported**: The cellular baseband modem transmits using the SIM's IMSI/MSISDN. The adapter must map `From` to an enrolled SIM number or ignore arbitrary alpha strings. |
| **MMS (Media Attachments)** | Supports images, audio, and PDF attachments via `MediaUrl` up to 5MB. | Does not construct MMS PDUs or interface with carrier MMS WAP/HTTP APN gateways. | **Medium** | **Unsupported in SMS**: If a client provides `MediaUrl`, the adapter returns Twilio error code `21620` (MMS not supported on this number). |
| **Dynamic `StatusCallback` Routing**| Each message can specify a distinct destination URL via the `StatusCallback` parameter. | Webhooks are registered globally at the user account level via `POST /webhook/subscriptions`. | **Low** | **Adapter Managed**: The adapter registers itself as the global TextBee webhook consumer, maintains a lightweight key-value store of `(smsId -> StatusCallbackUrl)`, and proxies events out. |
| **Message Cancellation & Redaction**| Allows setting `Status=canceled` or wiping text (`Body=""`) before transmission. | Bull queue jobs execute immediately or at scheduled wave intervals; no route exists to delete a job or redact text. | **Low** | **Adapter Managed / Mocked**: The adapter can maintain an in-memory queue or return a mocked success response. |
| **Message Record Deletion** | Supports `DELETE /Messages/{MessageSid}.json` to remove records from account history. | TextBee does not expose an individual message deletion route (records are preserved for billing auditing). | **Low** | **Mocked / Soft Tombstone**: The adapter can return HTTP `204 No Content` to satisfy SDK expectations while leaving TextBee records intact. |
| **Carrier Delivery Reports (DLR)** | High reliability via SS7/SMPP direct mobile network operator interconnects. | Dependent on the consumer SIM plan and local cell tower signaling. Prepaid and MVNO SIMs often suppress DLRs. | **Medium** | **Architectural Limitation**: TextBee correctly treats missing DLRs as non-downgrades (status remains `sent`). The adapter documents this best-effort behavior. |
| **Throughput & Burst Pacing** | Cloud numbers scale to 10–100+ transactions per second (TPS). | Physical SIM cards trigger carrier anti-spam bans if sent faster than 1 message per 3–5 seconds. | **Medium** | **Queued Pacing**: TextBee's Bull queue paces outbound transmissions automatically using `smsSendDelaySeconds`. Large batches experience higher queue latencies. |

---

## 2. In-Depth Gap Analysis & Engineering Workarounds

### 2.1 Alphanumeric Sender ID vs. Physical Cellular SIM
- **Root Cause:** Twilio routes messages through SMPP carrier connections where the originator field in the `submit_sm` PDU can be arbitrarily populated with an alphanumeric string. In contrast, TextBee delegates transmission to the Android Telephony framework (`SmsManager.sendTextMessage`), which communicates with a baseband cellular modem tied to a physical SIM card.
- **Engineering Workaround:**
  1. The adapter inspects the `From` parameter.
  2. If `From` is formatted as an E.164 phone number, the adapter searches active devices for a matching SIM MSISDN.
  3. If `From` is an alphanumeric string (or does not match any SIM), the adapter routes the message through the account's configured `isDefault` device, overriding the sender identity with the device's actual cellular number.

### 2.2 Dynamic StatusCallback Delivery
- **Root Cause:** Twilio API developers frequently pass dynamic callback URLs per request (e.g. embedding session tokens in query parameters: `https://app.com/callback?orderId=1029`). TextBee requires webhook targets to be pre-registered via `POST /api/v1/webhook/subscriptions`.
- **Engineering Workaround:**
  1. During adapter bootstrap, the adapter creates a webhook subscription on the user's TextBee account pointing to its own internal callback intake: `https://adapter.internal/webhooks/textbee`.
  2. When a client calls `POST /Messages.json`, the adapter stores the mapping in memory or Redis:
     ```typescript
     await redis.setex(`callback:${smsBatchId}`, 86400, req.body.StatusCallback);
     ```
  3. When TextBee fires an `sms.*` event to the adapter, the adapter resolves the customer's callback URL from Redis, converts the payload to Twilio form-encoded format, signs it with `X-Twilio-Signature`, and executes the HTTP POST.

### 2.3 Throughput & Carrier Throttling Differences
- **Root Cause:** Enterprise cloud SMS gateways utilize dedicated short codes or 10DLC (10-Digit Long Code) carrier-approved throughputs. TextBee uses standard consumer/business Android subscriptions. Sending high volumes without pauses leads to carrier SIM disconnections (`SmsManager.RESULT_ERROR_LIMIT_EXCEEDED`).
- **Engineering Workaround:**
  - The adapter communicates immediate HTTP `201 Created` with `status: "queued"`, aligning with Twilio's asynchronous paradigm.
  - TextBee's internal Redis Bull queue (`sms-send.processor.ts`) handles wave spacing (5 seconds per message by default).
  - The adapter's `StatusCallback` will accurately inform the client as messages move from `queued` $\to$ `sending` $\to$ `sent`.

