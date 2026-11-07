# Complete TextBee API Route Inventory

## 1. Inventory Summary

A static source-code analysis of the TextBee codebase discovered a total of **63 distinct backend HTTP route endpoints** registered across 7 active NestJS controllers, plus **4 Next.js route handlers** in the frontend dashboard.

### Functional Breakdown

| Category | Total Endpoints | Public / Allowlisted | Authenticated App / Dashboard | Android Device Internal | Provider Webhooks / Signed Links | Deprecated Routes |
|---|---|---|---|---|---|---|
| **SMS Messaging & Gateway** | 15 | 5 | 1 | 3 | 0 | 6 |
| **Device Management** | 8 | 3 | 2 | 3 | 0 | 0 |
| **Authentication & API Keys** | 16 | 0 | 16 | 0 | 0 | 0 |
| **Webhooks Management** | 6 | 6 | 0 | 0 | 0 | 0 |
| **Billing & Plans** | 9 | 1 | 4 | 0 | 4 (1 webhook, 3 signed links) | 0 |
| **Support & Account** | 2 | 0 | 2 | 0 | 0 | 0 |
| **Dashboard Notifications** | 3 | 0 | 3 | 0 | 0 | 0 |
| **Email & SES Events** | 4 | 0 | 0 | 0 | 4 (1 webhook, 3 signed links) | 0 |
| **Next.js Web Routes** | 4 | 0 | 1 | 0 | 3 (email redirects) | 0 |
| **Total** | **67** | **15** | **29** | **6** | **11** | **6** |

---

## 2. Master API Endpoint Matrix

### 2.1. SMS Messaging & History (`api/src/gateway/gateway.controller.ts`)

| ID | Method | Full Route Path | Handler Function | Source Location | Auth Mechanism | Classification | Status |
|---|---|---|---|---|---|---|---|
| `GW-01` | `POST` | `/api/v1/gateway/send-sms` | `sendSMSDeviceless` | `gateway.controller.ts:321` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |
| `GW-02` | `POST` | `/api/v1/gateway/send-bulk-sms` | `sendBulkSMSDeviceless` | `gateway.controller.ts:355` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |
| `GW-03` | `GET` | `/api/v1/gateway/messages` | `getAccountMessages` | `gateway.controller.ts:584` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |
| `GW-04` | `GET` | `/api/v1/gateway/devices/:id/sms/:smsId` | `getSMSById` | `gateway.controller.ts:789` | `AuthGuard`, `CanModifyDevice` | Public Developer API | **Active** |
| `GW-05` | `GET` | `/api/v1/gateway/devices/:id/sms-batch/:smsBatchId` | `getSmsBatchById` | `gateway.controller.ts:820` | `AuthGuard`, `CanModifyDevice` | Public Developer API | **Active** |
| `GW-06` | `GET` | `/api/v1/gateway/devices/:id/messages/pending` | `claimPendingMessages` | `gateway.controller.ts:735` | `AuthGuard`, `CanModifyDevice` | Android Device Internal | **Active** |
| `GW-07` | `PATCH`| `/api/v1/gateway/devices/:id/sms-status` | `updateSMSStatus` | `gateway.controller.ts:753` | `AuthGuard`, `CanModifyDevice` | Android Device Internal | **Active** |
| `GW-08` | `POST` | `/api/v1/gateway/devices/:id/receive-sms` | `receiveSMS` | `gateway.controller.ts:438` | `AuthGuard`, `CanModifyDevice` | Android Device Internal | **Active** |
| `GW-09` | `POST` | `/api/v1/gateway/devices/:id/send-sms` | `sendSMS` | `gateway.controller.ts:385` | `AuthGuard`, `CanModifyDevice` | Deprecated Device API | **Deprecated** |
| `GW-10` | `POST` | `/api/v1/gateway/devices/:id/sendSMS` | `sendSMS` | `gateway.controller.ts:385` | `AuthGuard`, `CanModifyDevice` | Deprecated Legacy Alias | **Deprecated** |
| `GW-11` | `POST` | `/api/v1/gateway/devices/:id/send-bulk-sms` | `sendBulkSMS` | `gateway.controller.ts:407` | `AuthGuard`, `CanModifyDevice` | Deprecated Device API | **Deprecated** |
| `GW-12` | `POST` | `/api/v1/gateway/devices/:id/receiveSMS` | `receiveSMSLegacy` | `gateway.controller.ts:463` | `AuthGuard`, `CanModifyDevice` | Deprecated Legacy Alias | **Deprecated** |
| `GW-13` | `GET` | `/api/v1/gateway/devices/:id/get-received-sms` | `getReceivedSMS` | `gateway.controller.ts:619` | `AuthGuard`, `CanModifyDevice` | Deprecated Device API | **Deprecated** |
| `GW-14` | `GET` | `/api/v1/gateway/devices/:id/getReceivedSMS` | `getReceivedSMSLegacy` | `gateway.controller.ts:658` | `AuthGuard`, `CanModifyDevice` | Deprecated Legacy Alias | **Deprecated** |
| `GW-15` | `GET` | `/api/v1/gateway/devices/:id/messages` | `getMessages` | `gateway.controller.ts:709` | `AuthGuard`, `CanModifyDevice` | Deprecated Device API | **Deprecated** |

---

### 2.2. Device Management & Telemetry (`api/src/gateway/gateway.controller.ts`)

| ID | Method | Full Route Path | Handler Function | Source Location | Auth Mechanism | Classification | Status |
|---|---|---|---|---|---|---|---|
| `DEV-01` | `GET` | `/api/v1/gateway/stats` | `getStats` | `gateway.controller.ts:113` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |
| `DEV-02` | `GET` | `/api/v1/gateway/devices` | `getDevices` | `gateway.controller.ts:170` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |
| `DEV-03` | `GET` | `/api/v1/gateway/devices/:id` | `getDevice` | `gateway.controller.ts:191` | `AuthGuard`, `CanModifyDevice` | Public Developer API | **Active** |
| `DEV-04` | `POST`| `/api/v1/gateway/devices` | `registerDevice` | `gateway.controller.ts:149` | `AuthGuard` (API Key / JWT) | Android Device & App | **Active** |
| `DEV-05` | `PATCH`| `/api/v1/gateway/devices/:id` | `updateDevice` | `gateway.controller.ts:216` | `AuthGuard`, `CanModifyDevice` | Android Device & Dashboard | **Active** |
| `DEV-06` | `POST`| `/api/v1/gateway/devices/:id/heartbeat` | `heartbeat` | `gateway.controller.ts:244` | `AuthGuard`, `CanModifyDevice` | Android Device Internal | **Active** |
| `DEV-07` | `POST`| `/api/v1/gateway/devices/:id/set-default` | `setDefaultDevice` | `gateway.controller.ts:290` | `AuthGuard`, `CanModifyDevice` | Dashboard Application API | **Active** |
| `DEV-08` | `DELETE`| `/api/v1/gateway/devices/:id` | `deleteDevice` | `gateway.controller.ts:268` | `AuthGuard`, `CanModifyDevice` | Dashboard Application API | **Active** |
| `DEV-09` | `GET` | `/api/v1/gateway/sms-permission-status` | `getSmsPermissionStatus` | `gateway.controller.ts:131` | `AuthGuard` (API Key / JWT) | Dashboard Application API | **Active** |

---

### 2.3. Authentication & API Key Management (`api/src/auth/auth.controller.ts`)

| ID | Method | Full Route Path | Handler Function | Source Location | Auth Mechanism | Classification | Status |
|---|---|---|---|---|---|---|---|
| `AUTH-01` | `POST` | `/api/v1/auth/login` | `login` | `auth.controller.ts:85` | Public (Cloudflare Turnstile) | Session Establishment | **Active** |
| `AUTH-02` | `POST` | `/api/v1/auth/google-login` | `googleLogin` | `auth.controller.ts:110` | Public (Google ID Token) | OAuth Session Establishment | **Active** |
| `AUTH-03` | `POST` | `/api/v1/auth/register` | `register` | `auth.controller.ts:133` | Public (Cloudflare Turnstile) | Account Creation | **Active** |
| `AUTH-04` | `GET` | `/api/v1/auth/who-am-i` | `whoAmI` | `auth.controller.ts:156` | `AuthGuard` (API Key / JWT) | Session & Device Identity | **Active** |
| `AUTH-05` | `PATCH`| `/api/v1/auth/update-profile` | `updateProfile` | `auth.controller.ts:175` | `AuthGuard` (API Key / JWT) | Account Profile API | **Active** |
| `AUTH-06` | `POST` | `/api/v1/auth/change-password` | `changePassword` | `auth.controller.ts:198` | `AuthGuard` (API Key / JWT) | Account Security API | **Active** |
| `AUTH-07` | `POST` | `/api/v1/auth/api-keys` | `generateApiKey` | `auth.controller.ts:220` | `AuthGuard` (API Key / JWT) | API Key Administration | **Active** |
| `AUTH-08` | `GET` | `/api/v1/auth/api-keys` | `getApiKey` | `auth.controller.ts:247` | `AuthGuard` (API Key / JWT) | API Key Administration | **Active** |
| `AUTH-09` | `DELETE`| `/api/v1/auth/api-keys/:id` | `deleteApiKey` | `auth.controller.ts:273` | `AuthGuard`, `CanModifyApiKey` | API Key Administration | **Active** |
| `AUTH-10` | `POST` | `/api/v1/auth/api-keys/:id/revoke` | `revokeApiKey` | `auth.controller.ts:296` | `AuthGuard`, `CanModifyApiKey` | API Key Administration | **Active** |
| `AUTH-11` | `PATCH`| `/api/v1/auth/api-keys/:id/rename` | `renameApiKey` | `auth.controller.ts:318` | `AuthGuard`, `CanModifyApiKey` | API Key Administration | **Active** |
| `AUTH-12` | `PATCH`| `/api/v1/auth/onboarding` | `updateOnboarding` | `auth.controller.ts:337` | `AuthGuard` (API Key / JWT) | Dashboard UI State | **Active** |
| `AUTH-13` | `POST` | `/api/v1/auth/request-password-reset` | `requestPasswordReset` | `auth.controller.ts:358` | Public (Cloudflare Turnstile) | Account Recovery | **Active** |
| `AUTH-14` | `POST` | `/api/v1/auth/reset-password` | `resetPassword` | `auth.controller.ts:377` | Public (OTP Token) | Account Recovery | **Active** |
| `AUTH-15` | `POST` | `/api/v1/auth/send-email-verification-email`| `sendEmailVerificationEmail` | `auth.controller.ts:398` | `AuthGuard` (API Key / JWT) | Identity Verification | **Active** |
| `AUTH-16` | `POST` | `/api/v1/auth/verify-email` | `verifyEmail` | `auth.controller.ts:419` | Public (OTP Token) | Identity Verification | **Active** |

---

### 2.4. Webhooks Management (`api/src/webhook/webhook.controller.ts`)

| ID | Method | Full Route Path | Handler Function | Source Location | Auth Mechanism | Classification | Status |
|---|---|---|---|---|---|---|---|
| `WH-01` | `GET` | `/api/v1/webhooks` | `getWebhooks` | `webhook.controller.ts:79` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |
| `WH-02` | `POST`| `/api/v1/webhooks` | `createWebhook` | `webhook.controller.ts:220` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |
| `WH-03` | `GET` | `/api/v1/webhooks/:webhookId` | `getWebhook` | `webhook.controller.ts:194` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |
| `WH-04` | `PATCH`| `/api/v1/webhooks/:webhookId` | `updateWebhook` | `webhook.controller.ts:251` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |
| `WH-05` | `DELETE`| `/api/v1/webhooks/:webhookId` | `deleteWebhook` | `webhook.controller.ts:279` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |
| `WH-06` | `GET` | `/api/v1/webhooks/notifications` | `getWebhookNotifications` | `webhook.controller.ts:155` | `AuthGuard` (API Key / JWT) | Public Developer API | **Active** |

---

### 2.5. Billing, Subscriptions & Inbound Webhooks (`api/src/billing/billing.controller.ts`)

| ID | Method | Full Route Path | Handler Function | Source Location | Auth Mechanism | Classification | Status |
|---|---|---|---|---|---|---|---|
| `BILL-01` | `GET` | `/api/v1/billing/plans` | `getPlans` | `billing.controller.ts:58` | Public (No credentials) | Public Plan Catalog | **Active** |
| `BILL-02` | `GET` | `/api/v1/billing/current-subscription` | `getCurrentSubscription` | `billing.controller.ts:78` | `AuthGuard` (API Key / JWT) | Dashboard & Device API | **Active** |
| `BILL-03` | `GET` | `/api/v1/billing/notifications` | `listNotifications` | `billing.controller.ts:96` | `AuthGuard` (API Key / JWT) | Dashboard Alerts API | **Active** |
| `BILL-04` | `POST`| `/api/v1/billing/checkout` | `getCheckoutUrl` | `billing.controller.ts:120` | `AuthGuard` (API Key / JWT) | Polar Checkout Initiation | **Active** |
| `BILL-05` | `POST`| `/api/v1/billing/change-plan` | `changePlan` | `billing.controller.ts:150` | `AuthGuard` (API Key / JWT) | Plan Upgrade/Downgrade | **Active** |
| `BILL-06` | `GET` | `/api/v1/billing/card` | `cardUpdate` | `billing.controller.ts:162` | Signed Link Token (`?t=`) | Redirect Handler | **Active** |
| `BILL-07` | `GET` | `/api/v1/billing/yearly` | `yearlySwitch` | `billing.controller.ts:185` | Signed Link Token (`?t=`) | Redirect Handler | **Active** |
| `BILL-08` | `GET` | `/api/v1/billing/checkout/resume` | `checkoutResume` | `billing.controller.ts:193` | Signed Link Token (`?t=`) | Redirect Handler | **Active** |
| `BILL-09` | `POST`| `/api/v1/billing/webhook/polar` | `handlePolarWebhook` | `billing.controller.ts:203` | Polar Webhook Signature | Provider Webhook Callback | **Active** |

---

### 2.6. Support & Account Administration (`api/src/support/support.controller.ts`)

| ID | Method | Full Route Path | Handler Function | Source Location | Auth Mechanism | Classification | Status |
|---|---|---|---|---|---|---|---|
| `SUP-01` | `POST` | `/api/v1/support/customer-support` | `createSupportMessage` | `support.controller.ts:55` | `AuthGuard` + Turnstile | User Support Request | **Active** |
| `SUP-02` | `POST` | `/api/v1/support/request-account-deletion` | `requestAccountDeletion` | `support.controller.ts:89` | `JwtAuthGuard` + Turnstile | Account Deletion | **Active** |

---

### 2.7. Dashboard Notifications Engine (`api/src/notifications/notifications.controller.ts`)

| ID | Method | Full Route Path | Handler Function | Source Location | Auth Mechanism | Classification | Status |
|---|---|---|---|---|---|---|---|
| `NOTIF-01` | `GET` | `/api/v1/notifications/feed` | `getFeed` | `notifications.controller.ts:61` | `AuthGuard` (Dashboard JWT) | Dashboard UI Feed | **Active** |
| `NOTIF-02` | `POST`| `/api/v1/notifications/events` | `recordEvents` | `notifications.controller.ts:80` | `AuthGuard` (Dashboard JWT) | Notification Telemetry | **Active** |
| `NOTIF-03` | `POST`| `/api/v1/notifications/:id/dismiss` | `dismiss` | `notifications.controller.ts:141` | `AuthGuard` (Dashboard JWT) | Notification Dismissal | **Active** |

---

### 2.8. Email Communications & Inbound Mail Events (`api/src/mail/email.controller.ts`)

| ID | Method | Full Route Path | Handler Function | Source Location | Auth Mechanism | Classification | Status |
|---|---|---|---|---|---|---|---|
| `MAIL-01` | `GET` | `/api/v1/email/unsubscribe` | `unsubscribePage` | `email.controller.ts:59` | Signed Link Token (`?t=`) | HTML Unsubscribe Page | **Active** |
| `MAIL-02` | `POST`| `/api/v1/email/unsubscribe` | `unsubscribe` | `email.controller.ts:66` | Signed Link Token (`?t=`) | 1-Click Mail Action | **Active** |
| `MAIL-03` | `POST`| `/api/v1/email/resubscribe` | `resubscribe` | `email.controller.ts:75` | Signed Link Token (`?t=`) | Resubscribe Action | **Active** |
| `MAIL-04` | `POST`| `/api/v1/email/ses-events` | `sesEvents` | `email.controller.ts:83` | AWS SNS/SES Signature | Inbound Provider Callback | **Active** |

---

### 2.9. Frontend Next.js Web Route Handlers (`web/app`)

| ID | Method | Full Route Path | Handler Location | Forwarding Destination | Classification | Status |
|---|---|---|---|---|---|---|
| `WEB-01` | `GET` | `/app/checkout/resume` | `web/app/(app)/checkout/resume/route.ts` | Backend `/billing/checkout/resume` | Email Link Proxy | **Active** |
| `WEB-02` | `GET` | `/billing/card` | `web/app/billing/card/route.ts` | Backend `/billing/card` | Email Link Proxy | **Active** |
| `WEB-03` | `GET` | `/billing/yearly` | `web/app/billing/yearly/route.ts` | Backend `/billing/yearly` | Email Link Proxy | **Active** |
| `WEB-04` | `GET/POST`| `/api/auth/[...nextauth]` | `web/app/api/auth/[...nextauth]/route.ts` | NextAuth Engine | Auth Session Gateway | **Active** |

