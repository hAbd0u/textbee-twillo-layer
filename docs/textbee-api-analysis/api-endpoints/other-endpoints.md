# TextBee API Endpoints: Billing, Mail, Notifications & Ancillary Routes

This document details the ancillary operational endpoints across Billing, Email handling, In-Dashboard Notifications, and Next.js frontend helper proxies.

---

## 1. Billing & Subscription Endpoints (`BillingController`)

### 1.1. GET /api/v1/billing/plans
- **Endpoint ID:** `BILL-01`
- **Method:** `GET`
- **Path:** `/api/v1/billing/plans`
- **Classification:** Public Plan Catalogue
- **Authentication:** None (Public)
- **Source Reference:** `api/src/billing/billing.controller.ts:58`
- **Purpose:** Returns the active subscription plans catalogue, listing monthly/yearly pricing, daily/monthly SMS quotas, bulk limits, and device/webhook allowances.
- **Success Response (HTTP 200 OK):**
  ```json
  [
    {
      "name": "free",
      "monthlyPrice": 0,
      "dailyLimit": 50,
      "monthlyLimit": 1500,
      "bulkSendLimit": 50,
      "deviceLimit": 1,
      "webhookLimit": 2,
      "isActive": true
    },
    {
      "name": "pro",
      "monthlyPrice": 1900,
      "yearlyPrice": 19000,
      "dailyLimit": 5000,
      "monthlyLimit": 150000,
      "bulkSendLimit": 500,
      "deviceLimit": -1,
      "webhookLimit": 10,
      "isActive": true
    }
  ]
  ```

### 1.2. GET /api/v1/billing/current-subscription
- **Endpoint ID:** `BILL-02`
- **Method:** `GET`
- **Path:** `/api/v1/billing/current-subscription`
- **Classification:** Authenticated Dashboard & Android Client API
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source Reference:** `api/src/billing/billing.controller.ts:78`
- **Purpose:** Retrieves the caller's active subscription tier and current billing period usage metrics (messages sent today, messages sent this month).
- **Success Response (HTTP 200 OK):**
  ```json
  {
    "subscription": {
      "plan": { "name": "pro", "dailyLimit": 5000, "monthlyLimit": 150000 },
      "status": "active",
      "currentPeriodStart": "2026-02-01T00:00:00.000Z",
      "currentPeriodEnd": "2026-03-01T00:00:00.000Z"
    },
    "usage": {
      "todaySentCount": 142,
      "monthSentCount": 3890
    }
  }
  ```

### 1.3. GET /api/v1/billing/notifications
- **Endpoint ID:** `BILL-03`
- **Method:** `GET`
- **Path:** `/api/v1/billing/notifications`
- **Classification:** Authenticated Dashboard Alerts
- **Authentication:** `AuthGuard`
- **Source Reference:** `api/src/billing/billing.controller.ts:96`
- **Purpose:** Fetches up to 50 billing-related alerts (e.g. 80% quota reached, 100% limit reached, past due payment alerts).

### 1.4. POST /api/v1/billing/checkout
- **Endpoint ID:** `BILL-04`
- **Method:** `POST`
- **Path:** `/api/v1/billing/checkout`
- **Classification:** Payment Checkout Initiation
- **Authentication:** `AuthGuard`
- **Source Reference:** `api/src/billing/billing.controller.ts:120`
- **Request Body:** `{ "planName": "pro", "interval": "month" }`
- **Success Response (HTTP 201 Created):** `{ "url": "https://polar.sh/checkout/..." }`

### 1.5. POST /api/v1/billing/change-plan
- **Endpoint ID:** `BILL-05`
- **Method:** `POST`
- **Path:** `/api/v1/billing/change-plan`
- **Classification:** Subscription Upgrade/Downgrade
- **Authentication:** `AuthGuard`
- **Source Reference:** `api/src/billing/billing.controller.ts:150`
- **Request Body:** `{ "planName": "pro", "interval": "year" }`

### 1.6. Billing Email Redirect Routes (Unauthenticated Token Links)
- `GET /api/v1/billing/card?t=<token>` (`BILL-06`): Validates signed token from billing reminder emails and redirects user to Polar customer portal for credit card updates.
- `GET /api/v1/billing/yearly?t=<token>` (`BILL-07`): Validates token and redirects user to yearly discounted upgrade checkout.
- `GET /api/v1/billing/checkout/resume?t=<token>` (`BILL-08`): Validates token and resumes an incomplete checkout session.

### 1.7. POST /api/v1/billing/webhook/polar
- **Endpoint ID:** `BILL-09`
- **Method:** `POST`
- **Path:** `/api/v1/billing/webhook/polar`
- **Classification:** External Provider Inbound Webhook
- **Authentication:** Polar Webhook HMAC Signature verification (Raw body configured via `api/src/main.ts:80`)
- **Source Reference:** `api/src/billing/billing.controller.ts:203`
- **Handled Events:**
  - `subscription.created`, `subscription.active`, `subscription.updated`, `subscription.uncanceled`: Updates `Subscription` record to active, stores period dates.
  - `subscription.past_due`: Records `pastDueAt` timestamp, triggers overdue billing notices.
  - `subscription.cancelled` / `subscription.canceled`: Records churn reason, schedules plan downgrade at period end.
  - `subscription.revoked`: Immediately terminates subscription privileges.
  - `checkout.updated`: Marks `checkoutSession.isCompleted = true`.

---

## 2. In-Dashboard Notifications Engine (`NotificationsController`)

### 2.1. GET /api/v1/notifications/feed
- **Endpoint ID:** `NOTIF-01`
- **Method:** `GET`
- **Path:** `/api/v1/notifications/feed`
- **Classification:** Authenticated Dashboard Notification Banner
- **Authentication:** `AuthGuard` (JWT)
- **Source Reference:** `api/src/notifications/notifications.controller.ts:61`
- **Purpose:** Evaluates dynamic targeting rules and returns personalized notification cards/banners (e.g., "Connect your first Android phone", "Upgrade to Pro", "Verify your email").

### 2.2. POST /api/v1/notifications/events
- **Endpoint ID:** `NOTIF-02`
- **Method:** `POST`
- **Path:** `/api/v1/notifications/events`
- **Classification:** Telemetry Tracking
- **Authentication:** `AuthGuard` (JWT)
- **Source Reference:** `api/src/notifications/notifications.controller.ts:80`
- **Request Body:**
  ```json
  {
    "events": [
      { "notificationId": "connect_device_banner", "type": "impression" },
      { "notificationId": "verify_email_banner", "type": "click" }
    ]
  }
  ```

### 2.3. POST /api/v1/notifications/:id/dismiss
- **Endpoint ID:** `NOTIF-03`
- **Method:** `POST`
- **Path:** `/api/v1/notifications/:id/dismiss`
- **Classification:** Notification Dismissal
- **Authentication:** `AuthGuard` (JWT)
- **Source Reference:** `api/src/notifications/notifications.controller.ts:141`
- **Request Body:** `{ "snoozeHours": 24 }`

---

## 3. Email Handling & SES Inbound Events (`EmailController`)

### 3.1. GET / POST /api/v1/email/unsubscribe & resubscribe
- `GET /api/v1/email/unsubscribe?t=<token>` (`MAIL-01`): Renders standalone HTML confirmation page for unsubscribing from marketing/product emails.
- `POST /api/v1/email/unsubscribe?t=<token>` (`MAIL-02`): Executes unsubscribe (updates `user.emailPreferences.productEmails = false`).
- `POST /api/v1/email/resubscribe?t=<token>` (`MAIL-03`): Re-enables product emails (`productEmails = true`).

### 3.2. POST /api/v1/email/ses-events
- **Endpoint ID:** `MAIL-04`
- **Method:** `POST`
- **Path:** `/api/v1/email/ses-events`
- **Classification:** Inbound AWS SES / SNS Webhook
- **Authentication:** AWS SNS Message Signature Verification
- **Source Reference:** `api/src/mail/email.controller.ts:83`
- **Purpose:** Ingests bounce, complaint, and delivery event notifications from AWS Simple Email Service (SES) via SNS notifications. Automatically flags suppressed email addresses in the `EmailSuppression` collection.

---

## 4. Frontend Next.js Web Route Handlers (`web/app`)

The web dashboard implements 4 server-side route handlers:

| Route Path | File Location | Purpose | Behavior |
|---|---|---|---|
| `GET /app/checkout/resume` | `web/app/(app)/checkout/resume/route.ts` | Resolves email token links | Redirects browser to backend `/api/v1/billing/checkout/resume?t=...` |
| `GET /billing/card` | `web/app/billing/card/route.ts` | Resolves card update email link | Redirects browser to backend `/api/v1/billing/card?t=...` |
| `GET /billing/yearly` | `web/app/billing/yearly/route.ts` | Resolves yearly upgrade email link | Redirects browser to backend `/api/v1/billing/yearly?t=...` |
| `GET/POST /api/auth/[...nextauth]` | `web/app/api/auth/[...nextauth]/route.ts` | NextAuth Authentication Endpoint | Coordinates Google OAuth and credentials sessions with TextBee backend. |

