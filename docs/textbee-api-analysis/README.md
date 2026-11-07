# TextBee API Reverse Engineering and Repository Analysis

## Executive Summary

This documentation repository contains the comprehensive, source-code-driven analysis of the **TextBee** Android SMS Gateway codebase. TextBee allows Android smartphones with active cellular SIM cards to operate as programmatic SMS gateways orchestrated through a centralized backend API.

This technical dossier was produced under **Phase 1: Repository Analysis and Existing API Documentation**. It serves as an authoritative source of truth, technical reference, and contract extraction for backend engineers and API reverse engineering specialists.

---

## Analysis Metadata

| Metadata Field | Value | Source Verification |
|---|---|---|
| **Repository** | `textbee` (upstream: `https://github.com/textbee/textbee`) | Confirmed local Git repository |
| **Analyzed Branch** | `twillo-layer` | `git branch --show-current` |
| **Analyzed Git Commit** | `7bde7e0c14f465b79032de18b4187d7e41caba6f` | `git rev-parse HEAD` |
| **Analysis Methodology** | Static Code Analysis, AST & Decorator Inspection, DTO Schema Extraction, Retrofit Service Analysis | Verified across `api/`, `android/`, and `web/` |
| **Document Date** | November 2026 | Antigravity Engine Analysis |
| **Verification Scope** | Backend API (NestJS), Android Gateway (Kotlin/Java), Web Dashboard (Next.js 16) | Read-only static analysis |

---

## Verification Classification System

All findings in this documentation suite adhere to the following strict classification labels:

- **Verified:** Directly confirmed by source code definitions, schema definitions, decorators, unit/integration tests, or runtime implementations in the repository.
- **Inferred:** Strongly suggested by implementation patterns, conventions, or interdependent module configurations, but lacking an explicit assertion or strict runtime constraint.
- **Unverified:** Insufficient source code evidence exists in the repository to guarantee exact runtime behavior under all edge cases.
- **Unsupported:** Confirmed to be entirely absent, non-implemented, or expressly rejected by the analyzed codebase.

---

## Repository Architecture At A Glance

The repository is structured as a monorepo containing three primary operational tiers:

```
textbee/
├── api/          # NestJS 10.x backend application (REST API, MongoDB/Mongoose, Redis/Bull, FCM)
├── android/      # Native Android Gateway application (Kotlin/Java, WorkManager, Retrofit, SmsManager)
└── web/          # Next.js 16 / React web dashboard and customer management frontend
```

### Core Technologies
- **Backend Service Framework:** NestJS 10.x running on Express (`NestExpressApplication`), TypeScript.
- **Database Layer:** MongoDB with Mongoose ODM 8.x (`@nestjs/mongoose`).
- **Push Notification Pipeline:** Firebase Cloud Messaging (`firebase-admin` 12.x) for dispatching commands to Android devices.
- **Job Queue & Asynchronous Workers:** Bull Queue 1.x (`@nestjs/bull`) backed by Redis.
- **Scheduled Automations:** `@nestjs/schedule` (cron jobs for timeout sweeps, stale device heartbeats, and webhook retries).
- **Authentication:** Dual-mode authentication (Bearer JWT sessions for the dashboard and deterministic SHA-256 hashed API Keys via `x-api-key` header or `apiKey` query param).
- **Billing Provider:** Polar.sh (`@polar-sh/sdk`) with webhook processing and subscription synchronizations.
- **Email Delivery:** AWS SES / SNS event handlers, custom HTML template rendering.
- **Mobile Client:** Android application built with Kotlin and Java, utilizing Android `SmsManager`, `WorkManager` for guaranteed background execution, `Room` database for deduplication, and `Retrofit` 2.x for REST communication.

---

## Documentation Structure

The documentation is organized into modular technical specifications:

1. [Repository Overview](file:///e:/projects/textbee/docs/textbee-api-analysis/repository-overview.md): High-level code organization, dependencies, build configurations, and service boundaries.
2. [System Architecture](file:///e:/projects/textbee/docs/textbee-api-analysis/architecture.md): Architectural design, module topology, data flows, and subsystem interactions.
3. [API Inventory](file:///e:/projects/textbee/docs/textbee-api-analysis/api-inventory.md): Master inventory of all 63 discovered HTTP routes, categorized by operational domain.
4. **Endpoint Specifications (`api-endpoints/`)**:
   - [API Endpoints Overview](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/README.md)
   - [Authentication & Identity](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/authentication.md): 16 endpoints covering credentials, sessions, API keys, and account recovery.
   - [Device Management](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/devices.md): 8 endpoints covering device pairing, updates, telemetry, and heartbeat.
   - [SMS Messaging](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md): 15 endpoints covering single/bulk SMS sending, message queries, batch inspection, and recovery.
   - [Webhooks](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/webhooks.md): 6 endpoints managing event subscriptions and delivery logs.
   - [Administration & Support](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/administration.md): Customer support and account deletion.
   - [Billing, Mail & Ancillary](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/other-endpoints.md): 18 endpoints covering Polar billing, email preferences, dashboard alerts, and Next.js routes.
5. **Machine-Readable API Contracts (`api-contracts/`)**:
   - [OpenAPI 3.1.0 Specification (`openapi.yaml`)](file:///e:/projects/textbee/docs/textbee-api-analysis/api-contracts/openapi.yaml)
   - [Structured Endpoint Metadata (`endpoint-metadata.json`)](file:///e:/projects/textbee/docs/textbee-api-analysis/api-contracts/endpoint-metadata.json)
6. [Authentication & Security](file:///e:/projects/textbee/docs/textbee-api-analysis/authentication-security.md): Cryptographic mechanisms, guards, scopes, and verification routines.
7. [SMS Messaging Lifecycle](file:///e:/projects/textbee/docs/textbee-api-analysis/sms-lifecycle.md): End-to-end trace from submission, queuing, and FCM delivery, through Android radio transmission and carrier status reports.
8. [Device Communication Protocol](file:///e:/projects/textbee/docs/textbee-api-analysis/device-communication.md): Protocols, payloads, polling strategies, and synchronization between Android and backend.
9. [Data Models & Schema Mapping](file:///e:/projects/textbee/docs/textbee-api-analysis/data-models.md): Complete Mongoose schema documentation, field attributes, constraints, and relationships.
10. [Errors & State Transitions](file:///e:/projects/textbee/docs/textbee-api-analysis/errors-and-statuses.md): Comprehensive error code catalog (Android RIL, SmsManager, FCM, server errors) and message state machine transitions.
11. [Feature Capabilities & Limits](file:///e:/projects/textbee/docs/textbee-api-analysis/feature-capabilities.md): Verified capabilities, constraints, SIM handling, quotas, and limits.
12. [Findings & Gaps](file:///e:/projects/textbee/docs/textbee-api-analysis/findings-and-gaps.md): Idiosyncrasies, deprecated paths, security notes, and architectural edge cases.
13. [Source References](file:///e:/projects/textbee/docs/textbee-api-analysis/source-references.md): Direct source file and line traceability matrix.

