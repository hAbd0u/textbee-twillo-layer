# TextBee Repository Overview

## 1. Repository Purpose and Scope

TextBee is an open-source Android SMS gateway platform. It bridges standard RESTful HTTP clients with physical Android smartphones to allow sending and receiving SMS messages over real cellular network interfaces. The repository contains all software components required to host, manage, and operate the platform.

```
                          ┌────────────────────────┐
                          │   External Developer   │
                          │   Application / API    │
                          └───────────┬────────────┘
                                      │ HTTP REST API (x-api-key)
                                      ▼
┌──────────────────┐      ┌────────────────────────┐      ┌────────────────────────┐
│  Next.js 16 Web  │◄────►│  NestJS Backend API    │◄────►│     MongoDB Database   │
│  User Dashboard  │ HTTP │  (Core Orchestration)  │ Mongoose (Persistence Layer)  │
└──────────────────┘      └───────────┬────────────┘      └────────────────────────┘
                                      │ Firebase Cloud Messaging (FCM Push)
                                      │ + HTTP REST Callbacks (Heartbeat / Status)
                                      ▼
                          ┌────────────────────────┐
                          │     Android Device     │
                          │   (TextBee Gateway)    │
                          └───────────┬────────────┘
                                      │ Cellular Radio (SmsManager)
                                      ▼
                          ┌────────────────────────┐
                          │   Mobile Network /     │
                          │   Cellular Carrier     │
                          └────────────────────────┘
```

---

## 2. Directory Layout & Module Structure

The project root consists of three major codebases and orchestration configurations:

```text
textbee/
├── api/                           # Backend REST API Service
│   ├── src/
│   │   ├── analytics/             # Conversion reporting (Meta CAPI)
│   │   ├── auth/                  # Authentication, JWT, API Keys, Footprint tracking
│   │   ├── billing/               # Subscriptions, Polar.sh checkout, limits
│   │   ├── common/                # Turnstile, IP resolution, shared utilities
│   │   ├── gateway/               # SMS dispatch, Android devices, queue, statuses
│   │   ├── mail/                  # SES/SNS handler, email suppression & templates
│   │   ├── notifications/         # Dashboard user notifications engine
│   │   ├── openapi/               # Swagger configuration and public OpenAPI export
│   │   ├── support/               # Support tickets, account deletion requests
│   │   ├── users/                 # User schema, user rollups, background tasks
│   │   ├── webhook/               # Outbound webhook subscriptions & delivery queue
│   │   ├── app.module.ts          # Root NestJS application module
│   │   └── main.ts                # Application bootstrap entry point
│   ├── package.json               # Backend dependencies and scripts
│   ├── tsconfig.json              # TypeScript compilation configuration
│   └── openapi.json               # Generated OpenAPI 3.0 document
│
├── android/                       # Android Gateway Client
│   ├── app/src/main/
│   │   ├── AndroidManifest.xml    # Permissions, Receivers, and Services
│   │   └── java/com/vernu/sms/
│   │       ├── activities/        # Native Android activities
│   │       ├── database/          # Room DB (SmsDao, SmsDedupeStore)
│   │       ├── dtos/              # Data transfer objects for backend interaction
│   │       ├── helpers/           # Heartbeat, SIM detection, SMS filters
│   │       ├── receivers/         # BootCompletedReceiver, SMSBroadcastReceiver
│   │       ├── services/          # FCMService, Retrofit GatewayApiService
│   │       ├── ui/                # Jetpack Compose UI (Dashboard, Onboarding)
│   │       └── workers/           # WorkManager workers (SmsSend, StatusUpdate)
│   ├── build.gradle               # Root Gradle build script
│   └── app/build.gradle           # Application Gradle build script
│
├── web/                           # Customer Web Dashboard (Next.js)
│   ├── app/                       # Next.js 16 App Router (pages and API routes)
│   ├── components/                # React components (Radix UI, Tailwind CSS)
│   ├── config/                    # API client endpoints and route paths
│   ├── lib/                       # NextAuth options, API client hooks, utilities
│   └── proxy.ts                   # Next.js proxy middleware (Auth gating)
│
├── docker-compose.yaml            # Local development orchestration (MongoDB, Redis)
└── LICENSES/                      # License definitions (AGPL-3.0 / MIT)
```

---

## 3. Technology Stacks and Tooling

### 3.1. Backend API (`api/`)
- **Runtime:** Node.js (v18+ / v20+ recommended).
- **Framework:** NestJS 10.3.x built on top of Express 4.19.x.
- **Persistence:** MongoDB with Mongoose 8.2.x (`@nestjs/mongoose`).
- **Distributed Queuing:** Bull Queue 1.0.x (`@nestjs/bull`) powered by Redis (via `ioredis` / `redis`).
- **Scheduling:** NestJS Schedule 4.0.x (wrapping `cron`).
- **Push Notifications:** Firebase Admin SDK 12.0.x (`firebase-admin`).
- **Authentication:** Passport.js (`@nestjs/passport`), Passport JWT (`passport-jwt`), bcrypt, crypto (SHA-256).
- **HTTP Client:** Axios 1.6.x for outgoing webhook delivery.
- **API Documentation:** Swagger / OpenAPI CLI (`@nestjs/swagger` 7.3.x).
- **Anti-Bot Security:** Cloudflare Turnstile token validation service.
- **External Billing:** Polar.sh SDK (`@polar-sh/sdk`).

### 3.2. Android Gateway Application (`android/`)
- **Language:** Kotlin (primary) & Java.
- **Minimum SDK:** Android 24 (Android 7.0 Nougat).
- **Target SDK:** Android 34 (Android 14).
- **UI Framework:** Android Jetpack Compose & Material 3.
- **Background Execution:** AndroidX WorkManager 2.9.x (`androidx.work:work-runtime-ktx`).
- **Local Persistence:** Room 2.6.x SQLite database (`androidx.room`).
- **Networking:** Square Retrofit 2.9.x with OkHttp 4.12.x and Gson converter.
- **Push Client:** Firebase Messaging 23.4.x (`com.google.firebase:firebase-messaging`).
- **Telephony Access:** Android `android.telephony.SmsManager`, `SubscriptionManager`, and `TelephonyManager`.

### 3.3. Web Dashboard (`web/`)
- **Framework:** Next.js 16.0.x App Router with React 19.x.
- **Styling:** Tailwind CSS 3.4.x, Lucide React icons, Radix UI primitives (`shadcn/ui`).
- **Authentication:** NextAuth.js 4.24.x (`next-auth`) with Credentials and Google providers.
- **Data Fetching:** SWR 2.2.x and custom React hooks.
- **Testing:** Vitest and Playwright.

---

## 4. Subsystem Responsibilities and Cross-Component Interaction

### 4.1. Core Responsibilities

| Subsystem | Primary Responsibilities |
|---|---|
| **Backend API (`api/`)** | - Authenticates programmatic consumers (`x-api-key`) and web users (JWT).<br>- Validates phone numbers, sanitizes messages, and enforces plan quotas.<br>- Manages device pairing, heartbeat telemetry, and device status.<br>- Enqueues outgoing SMS messages and dispatches pushes via Firebase Cloud Messaging.<br>- Ingests delivered and received SMS reports from Android devices.<br>- Dispatches signed webhook events to external consumer systems.<br>- Synchronizes subscriptions and billing state with Polar.sh. |
| **Android Gateway (`android/`)** | - Registers hardware devices, push tokens, and multi-SIM configurations with the backend.<br>- Listens for FCM push notifications carrying outbound SMS payloads.<br>- Coordinates transmission through physical SIM radios using `SmsManager`.<br>- Uses `WorkManager` with exponential backoff to guarantee delivery of status updates to backend.<br>- Captures incoming cellular SMS messages via broadcast receivers and forwards them to backend.<br>- Periodically executes heartbeats reporting battery, network, SIM cards, and permission status.<br>- Periodically polls backend for unpushed pending messages (`claimPendingMessages`). |
| **Web Dashboard (`web/`)** | - User interface for account registration, login, and profile management.<br>- Visual management of paired Android devices and SIM cards.<br>- API key generation, revocation, and labeling.<br>- SMS composition, bulk send wizard, and message log viewing.<br>- Webhook subscription management and delivery log inspection.<br>- Plan selection, checkout redirect, and billing management. |

---

## 5. Build, Execution, and Deployment Architecture

1. **Dockerized Environment:** The repository includes a `docker-compose.yaml` file defining:
   - `mongo`: MongoDB instance with persistent volume storage.
   - `redis`: Redis server for Bull queues.
   - Development containers for `api` and `web`.
2. **Global Prefix and Versioning:**
   - As declared in `api/src/openapi/swagger-config.ts` (`applyApiConventions`), the NestJS application enforces a global prefix of `api` and URI versioning defaulting to `1`.
   - Therefore, all standard application routes are hosted under the `/api/v1/` prefix.
3. **Reverse Proxy Trust:**
   - In `api/src/main.ts`, `app.set('trust proxy', 1)` is enabled to ensure accurate IP resolution behind CDNs/reverse proxies (e.g., Cloudflare, NGINX), allowing proper functioning of rate limiters (`ThrottlerByIpGuard`).

