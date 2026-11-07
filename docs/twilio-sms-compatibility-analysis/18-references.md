# Bibliography & Technical References

This document catalogs all official documentation references, codebase source citations, and technical standards used during the Phase 2 compatibility analysis.

---

## 1. Official Twilio API Documentation References

1. **Twilio Programmable Messaging Overview:**
   - URL: [https://www.twilio.com/docs/messaging/api](https://www.twilio.com/docs/messaging/api)
   - Scope: Programmable Messaging architecture, REST base URLs, and client SDK conventions.
2. **Twilio Message Resource Reference:**
   - URL: [https://www.twilio.com/docs/messaging/api/message-resource](https://www.twilio.com/docs/messaging/api/message-resource)
   - Scope: `POST /Messages.json`, `GET /Messages/{Sid}.json`, `GET /Messages.json`, parameters, and JSON response schema.
3. **Outbound Message Status in StatusCallbacks:**
   - URL: [https://www.twilio.com/docs/messaging/guides/outbound-message-status-in-status-callbacks](https://www.twilio.com/docs/messaging/guides/outbound-message-status-in-status-callbacks)
   - Scope: Delivery callback parameters, status transitions, and error codes.
4. **Twilio Inbound Message Webhook Guide:**
   - URL: [https://www.twilio.com/docs/messaging/guides/webhook-request](https://www.twilio.com/docs/messaging/guides/webhook-request)
   - Scope: `application/x-www-form-urlencoded` parameters, TwiML response contracts.
5. **Twilio Error and Warning Dictionary:**
   - URL: [https://www.twilio.com/docs/api/errors](https://www.twilio.com/docs/api/errors)
   - Specific Error Codes: `20003` (Authentication), `21211` (Invalid Phone), `21602` (Missing Body), `21604` (Missing To), `21606` (Invalid From), `20429` (Throttled), `30001`–`30008` (Carrier / Delivery Failures).
6. **Twilio Webhook Security & Request Validation:**
   - URL: [https://www.twilio.com/docs/usage/security#validating-requests](https://www.twilio.com/docs/usage/security#validating-requests)
   - Scope: `X-Twilio-Signature` HMAC-SHA1 algorithm and parameter sorting rules.

---

## 2. TextBee Codebase Source References

- **Repository:** `textbee` (Upstream: `https://github.com/textbee/textbee`)
- **Branch:** `twillo-layer`
- **Commit SHA:** `7bde7e0c14f465b79032de18b4187d7e41caba6f`
- **Analysis Date:** November 2026

### Core Source Files:
1. **SMS Gateway Controller:**
   - [`api/src/gateway/gateway.controller.ts`](file:///e:/projects/textbee/api/src/gateway/gateway.controller.ts)
   - Lines: `L321` (send-sms), `L355` (send-bulk-sms), `L438` (receive-sms), `L584` (messages query), `L753` (sms-status), `L789` (getSMS).
2. **SMS Gateway Service:**
   - [`api/src/gateway/gateway.service.ts`](file:///e:/projects/textbee/api/src/gateway/gateway.service.ts)
   - Lines: `sendSMSDeviceless`, `getMessagesForUser`, `updateSMSStatus`, `claimPendingMessages`.
3. **Queue Processors & Wave Pacing:**
   - [`api/src/gateway/queue/sms-queue.processor.ts`](file:///e:/projects/textbee/api/src/gateway/queue/sms-queue.processor.ts)
   - Logic: 5-second inter-message wave delay, Bull delayed job processing.
4. **Webhook Fan-Out Service:**
   - [`api/src/webhook/webhook.service.ts`](file:///e:/projects/textbee/api/src/webhook/webhook.service.ts)
   - Schemas: [`webhook-subscription.schema.ts`](file:///e:/projects/textbee/api/src/webhook/schemas/webhook-subscription.schema.ts), [`webhook-notification.schema.ts`](file:///e:/projects/textbee/api/src/webhook/schemas/webhook-notification.schema.ts).
5. **Android Gateway Telephony & Workers:**
   - [`android/app/src/main/java/com/vernu/sms/receivers/SMSStatusReceiver.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/receivers/SMSStatusReceiver.kt)
   - [`android/app/src/main/java/com/vernu/sms/workers/SmsSendWorker.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/workers/SmsSendWorker.kt)
   - [`android/app/src/main/java/com/vernu/sms/helpers/SMSHelper.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/helpers/SMSHelper.kt)

---

## 3. Phase 1 Technical Analysis Baseline

All findings in this compatibility report build directly upon the Phase 1 technical documentation suite stored in [`docs/textbee-api-analysis/`](file:///e:/projects/textbee/docs/textbee-api-analysis):
- [`api-endpoints/sms-messaging.md`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md): 15 SMS endpoint contracts.
- [`api-endpoints/webhooks.md`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/webhooks.md): Webhook contracts and signature mechanisms.
- [`sms-lifecycle.md`](file:///e:/projects/textbee/docs/textbee-api-analysis/sms-lifecycle.md): End-to-end trace of outbound/inbound SMS.
- [`device-communication.md`](file:///e:/projects/textbee/docs/textbee-api-analysis/device-communication.md): FCM push and Retrofit uplink telemetry.
- [`errors-and-statuses.md`](file:///e:/projects/textbee/docs/textbee-api-analysis/errors-and-statuses.md): Error code dictionary and state transitions.
- [`api-contracts/openapi.yaml`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-contracts/openapi.yaml): Full OpenAPI 3.0.0 specification.

