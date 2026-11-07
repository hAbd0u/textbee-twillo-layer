# TextBee Source Code Traceability Index

This document provides a comprehensive traceability index linking every API controller, service, middleware guard, queue processor, cron task, database schema, and Android gateway component directly to its source code file and line references in the repository.

---

## 1. Controllers & Route Handlers (`api/src/`)

| Endpoint Category | Controller Class | Source File | Line Range | Handlers Defined |
|---|---|---|---|---|
| **Authentication & User Profile** | `AuthController` | [`api/src/auth/auth.controller.ts`](file:///e:/projects/textbee/api/src/auth/auth.controller.ts) | L35–L215 | `register`, `login`, `whoAmI`, `updateProfile`, `changePassword`, `logout`, `verifyEmail`, `resendVerification`, `sendPasswordResetEmail`, `resetPassword`, `googleAuth`, `googleAuthCallback`, `impersonate`, `stopImpersonate` |
| **API Key Management** | `ApiKeyController` | [`api/src/auth/api-key.controller.ts`](file:///e:/projects/textbee/api/src/auth/api-key.controller.ts) | L20–L85 | `createApiKey`, `getAllApiKeys`, `getApiKey`, `revokeApiKey` |
| **Device Fleet & Heartbeat** | `DeviceController` | [`api/src/gateway/controllers/device.controller.ts`](file:///e:/projects/textbee/api/src/gateway/controllers/device.controller.ts) | L25–L165 | `registerDevice`, `getDevices`, `getDevice`, `updateDevice`, `deleteDevice`, `heartbeat`, `permissionCheck` |
| **SMS Messaging & Gateway** | `GatewayController` | [`api/src/gateway/controllers/gateway.controller.ts`](file:///e:/projects/textbee/api/src/gateway/controllers/gateway.controller.ts) | L45–L385 | `sendSMS`, `sendSMSWithoutDevice`, `getMessages`, `getMessage`, `getStats`, `claimPendingMessages`, `receiveSMS`, `updateSMSStatus`, `sendSMSLegacy`, `getReceivedSMSLegacy` |
| **Webhook Subscriptions** | `WebhookController` | [`api/src/webhook/controllers/webhook.controller.ts`](file:///e:/projects/textbee/api/src/webhook/controllers/webhook.controller.ts) | L20–L110 | `subscribe`, `getSubscriptions`, `getSubscription`, `updateSubscription`, `deleteSubscription`, `getDeliveryHistory` |
| **Polar Billing & Subscriptions**| `BillingController` | [`api/src/billing/controllers/billing.controller.ts`](file:///e:/projects/textbee/api/src/billing/controllers/billing.controller.ts) | L30–L170 | `getPlans`, `getCurrentSubscription`, `createCheckoutSession`, `polarWebhook`, `cancelSubscription` |
| **Customer Support & Deletion** | `SupportController` | [`api/src/support/controllers/support.controller.ts`](file:///e:/projects/textbee/api/src/support/controllers/support.controller.ts) | L20–L65 | `submitSupportMessage`, `requestAccountDeletion` |
| **Dashboard Notifications** | `NotificationsController` | [`api/src/notifications/controllers/notifications.controller.ts`](file:///e:/projects/textbee/api/src/notifications/controllers/notifications.controller.ts) | L20–L75 | `getNotifications`, `markAsRead`, `markAllAsRead` |

---

## 2. Business Services & Core Logic (`api/src/`)

| Service Name | Source File | Key Responsibilities |
|---|---|---|
| `GatewayService` | [`api/src/gateway/services/gateway.service.ts`](file:///e:/projects/textbee/api/src/gateway/services/gateway.service.ts) | Outbound SMS validation, quota verification, device selection, database persistence, and Bull queue enqueueing |
| `DeviceService` | [`api/src/gateway/services/device.service.ts`](file:///e:/projects/textbee/api/src/gateway/services/device.service.ts) | Device registration, hardware telemetry processing, heartbeat reconciliation, and tombstone lifecycle management |
| `FCMService` | [`api/src/gateway/services/fcm.service.ts`](file:///e:/projects/textbee/api/src/gateway/services/fcm.service.ts) | Firebase Admin SDK integration, high-priority data payload assembly, and batched `sendEach` dispatch |
| `WebhookService` | [`api/src/webhook/services/webhook.service.ts`](file:///e:/projects/textbee/api/src/webhook/services/webhook.service.ts) | Webhook subscription management, HMAC-SHA256 signature generation (v1 & v2), and delivery history audit |
| `ApiKeyService` | [`api/src/auth/services/api-key.service.ts`](file:///e:/projects/textbee/api/src/auth/services/api-key.service.ts) | Cryptographic key generation, SHA-256 deterministic indexing, legacy bcrypt validation, and key revocation |
| `BillingService` | [`api/src/billing/services/billing.service.ts`](file:///e:/projects/textbee/api/src/billing/services/billing.service.ts) | Quota boundary enforcement (`canPerformAction`), Polar checkout session lifecycle, and subscription synchronization |
| `AuthService` | [`api/src/auth/services/auth.service.ts`](file:///e:/projects/textbee/api/src/auth/services/auth.service.ts) | User registration, password hashing, JWT session signing, email verification workflows, and Google OAuth2 |

---

## 3. Security, Authentication & Guards (`api/src/`)

| Guard / Middleware | Source File | Enforcement Scope |
|---|---|---|
| `AuthGuard` | [`api/src/common/guards/auth.guard.ts`](file:///e:/projects/textbee/api/src/common/guards/auth.guard.ts) | Dual-mode authentication: extracts and verifies JWT bearer tokens, session cookies, or `x-api-key` headers |
| `ThrottlerByIpGuard` | [`api/src/common/guards/throttler-by-ip.guard.ts`](file:///e:/projects/textbee/api/src/common/guards/throttler-by-ip.guard.ts) | Sliding window rate limiting per IP address, with canonical `/64` IPv6 subnet normalization |
| `CanModifyDeviceGuard` | [`api/src/common/guards/can-modify-device.guard.ts`](file:///e:/projects/textbee/api/src/common/guards/can-modify-device.guard.ts) | Verifies that the authenticated user is the legitimate owner of the requested `deviceId` |
| `CanModifyApiKeyGuard` | [`api/src/common/guards/can-modify-api-key.guard.ts`](file:///e:/projects/textbee/api/src/common/guards/can-modify-api-key.guard.ts) | Verifies that the authenticated user is the legitimate owner of the requested `apiKeyId` |

---

## 4. Background Queues & Scheduled Tasks (`api/src/`)

| Processor / Task | Source File | Technology | Function |
|---|---|---|---|
| `SmsSendProcessor` | [`api/src/gateway/processors/sms-send.processor.ts`](file:///e:/projects/textbee/api/src/gateway/processors/sms-send.processor.ts) | Redis / Bull (`sms-send`) | Consumes queued SMS jobs, enforces wave spacing (5s), and triggers `FCMService.sendSMSPush` |
| `WebhookDeliveryProcessor` | [`api/src/webhook/processors/webhook-delivery.processor.ts`](file:///e:/projects/textbee/api/src/webhook/processors/webhook-delivery.processor.ts) | Redis / Bull (`webhook-delivery`) | Dispatches HTTP POST webhooks with exponential backoff retries (up to 5 attempts) |
| `SmsStatusUpdateTask` | [`api/src/gateway/tasks/sms-status-update.task.ts`](file:///e:/projects/textbee/api/src/gateway/tasks/sms-status-update.task.ts) | NestJS `@Cron` (every 10m) | Sweeps messages stuck in `pending` or `dispatched` for $>20$ min, marking them `unknown` |
| `DeviceStatusTask` | [`api/src/gateway/tasks/device-status.task.ts`](file:///e:/projects/textbee/api/src/gateway/tasks/device-status.task.ts) | NestJS `@Cron` (every 5m) | Evaluates `lastSeenAt` vs `heartbeatIntervalMinutes * 2`, flagging unresponsive devices `offline` |

---

## 5. Database Entities & Schemas (`api/src/`)

| Entity Model | Collection Name | Source File | Description |
|---|---|---|---|
| `User` | `users` | [`api/src/users/schemas/user.schema.ts`](file:///e:/projects/textbee/api/src/users/schemas/user.schema.ts) | Account profile, role, verification status, geofootprint, and denormalized usage rollups |
| `Device` | `devices` | [`api/src/gateway/schemas/device.schema.ts`](file:///e:/projects/textbee/api/src/gateway/schemas/device.schema.ts) | Android gateway phone records, FCM tokens, hardware telemetry, power state, and SIM info |
| `SMS` | `sms` | [`api/src/gateway/schemas/sms.schema.ts`](file:///e:/projects/textbee/api/src/gateway/schemas/sms.schema.ts) | Individual outbound and inbound SMS records with status timestamps and error history |
| `SMSBatch` | `smsbatches` | [`api/src/gateway/schemas/sms-batch.schema.ts`](file:///e:/projects/textbee/api/src/gateway/schemas/sms-batch.schema.ts) | Bulk campaign metadata, aggregate recipient counts, and progression statistics |
| `ApiKey` | `apikeys` | [`api/src/auth/schemas/api-key.schema.ts`](file:///e:/projects/textbee/api/src/auth/schemas/api-key.schema.ts) | Hashed API key credentials with deterministic SHA-256 search indexes |
| `WebhookSubscription` | `webhooksubscriptions` | [`api/src/webhook/schemas/webhook-subscription.schema.ts`](file:///e:/projects/textbee/api/src/webhook/schemas/webhook-subscription.schema.ts) | Webhook target URLs, subscribed event types, HMAC secrets, and failure metrics |
| `WebhookNotification` | `webhooknotifications` | [`api/src/webhook/schemas/webhook-notification.schema.ts`](file:///e:/projects/textbee/api/src/webhook/schemas/webhook-notification.schema.ts) | Individual webhook delivery dispatch audit entries, HTTP status codes, and response bodies |
| `Plan` | `plans` | [`api/src/billing/schemas/plan.schema.ts`](file:///e:/projects/textbee/api/src/billing/schemas/plan.schema.ts) | Subscription tier definitions, daily/monthly quotas, and Polar product mapping |
| `Subscription` | `subscriptions` | [`api/src/billing/schemas/subscription.schema.ts`](file:///e:/projects/textbee/api/src/billing/schemas/subscription.schema.ts) | Active user billing subscriptions, cycle periods, and cancellation flags |

---

## 6. Android Gateway Subsystem (`android/app/src/main/java/com/vernu/sms/`)

| Android Component | Component Type | Source File | Responsibilities |
|---|---|---|---|
| `FCMService` | `FirebaseMessagingService` | [`FCMService.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/services/FCMService.kt) | Receives high-priority data pushes (`smsData`, `heartbeat_check`) and triggers local workers |
| `SmsSendWorker` | WorkManager `Worker` | [`SmsSendWorker.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/workers/SmsSendWorker.kt) | Enforces send delays, resolves multi-SIM slot, checks idempotency store, and calls `SMSHelper` |
| `SMSStatusUpdateWorker` | WorkManager `Worker` | [`SMSStatusUpdateWorker.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/workers/SMSStatusUpdateWorker.kt) | Uploads status changes (`sent`, `delivered`, `failed`) via `PATCH /sms-status` with exponential backoff |
| `SMSReceivedWorker` | WorkManager `Worker` | [`SMSReceivedWorker.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/workers/SMSReceivedWorker.kt) | Uploads captured inbound SMS via `POST /receive-sms` with deduplication fingerprints |
| `HeartbeatWorker` | WorkManager `Worker` | [`HeartbeatWorker.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/workers/HeartbeatWorker.kt) | Periodic background telemetry collection and upload via `POST /heartbeat` |
| `SMSStatusReceiver` | `BroadcastReceiver` | [`SMSStatusReceiver.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/receivers/SMSStatusReceiver.kt) | Captures carrier radio callbacks (`sentIntent`, `deliveryIntent`), mapping `SmsManager` result codes |
| `SMSBroadcastReceiver` | `BroadcastReceiver` | [`SMSBroadcastReceiver.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/receivers/SMSBroadcastReceiver.kt) | Listens for incoming cellular SMS broadcasts (`android.provider.Telephony.SMS_RECEIVED`) |
| `SMSHelper` | Helper Utility | [`SMSHelper.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/helpers/SMSHelper.kt) | Handles `SmsManager` invocation, multi-part message splitting, and permission assertions |
| `HeartbeatHelper` | Helper Utility | [`HeartbeatHelper.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/helpers/HeartbeatHelper.kt) | Collects battery, storage, RAM, uptime, network type, and carrier SIM telemetry |
| `RecoveryPoll` | Helper Utility | [`RecoveryPoll.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/helpers/RecoveryPoll.kt) | Fetches pending messages via `GET /messages/pending` when FCM pushes are dropped by Doze mode |
| `SmsDedupeStore` | Room DAO / Store | [`SmsDedupeStore.kt`](file:///e:/projects/textbee/android/app/src/main/java/com/vernu/sms/database/SmsDedupeStore.kt) | Local SQLite idempotency database preventing duplicate cellular SMS transmission |

---

## 7. Next.js Web Dashboard Route Handlers (`web/src/app/api/`)

| Route Handler | Source File | Purpose |
|---|---|---|
| `GET /api/public-stats` | [`web/src/app/api/public-stats/route.ts`](file:///e:/projects/textbee/web/src/app/api/public-stats/route.ts) | Public landing page metrics (total devices, total messages sent) |
| `GET /api/changelog` | [`web/src/app/api/changelog/route.ts`](file:///e:/projects/textbee/web/src/app/api/changelog/route.ts) | Release history and version updates feed |
| `POST /api/polar/webhook` | [`web/src/app/api/polar/webhook/route.ts`](file:///e:/projects/textbee/web/src/app/api/polar/webhook/route.ts) | Edge ingress forwarder for Polar billing webhooks |
| `POST /api/telemetry/capture` | [`web/src/app/api/telemetry/capture/route.ts`](file:///e:/projects/textbee/web/src/app/api/telemetry/capture/route.ts) | Edge ingress handler for client-side analytical telemetry |

