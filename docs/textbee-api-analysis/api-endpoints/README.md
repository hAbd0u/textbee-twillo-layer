# TextBee Endpoint Documentation Directory

This directory provides comprehensive, field-by-field technical documentation for every HTTP endpoint implemented in the TextBee repository.

## Documentation Structure

Each document adheres to a rigorous specification standard documenting:
- Exact HTTP method, full URI path (including global `/api/v1` prefix), and controller handler.
- Authentication and authorization rules (Guards, roles, ownership checks).
- Full request schemas (Headers, Path Parameters, Query Parameters, Request Body).
- Field validation constraints, defaults, types, and error triggers.
- Success and error response schemas with realistic JSON examples.
- Internal execution flow, database mutations, side effects (FCM push, Bull queues, webhooks).
- Implementation caveats, deprecations, and architectural observations.

## Category Index

1. [Authentication & API Key Endpoints (`authentication.md`)](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/authentication.md): Covers user registration, password/Google sign-in, profile updates, password recovery, email confirmation, and full CRUD operations on API keys.
2. [Device Management Endpoints (`devices.md`)](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/devices.md): Covers pairing new Android hardware, telemetry reporting, device renaming, default device routing, device unpairing, heartbeat handling, and permission monitoring.
3. [SMS Messaging Endpoints (`sms-messaging.md`)](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md): Covers primary send endpoints (`/send-sms`, `/send-bulk-sms`), unified account-wide message history (`/messages`), individual SMS and batch lookups, device-side status callback (`/sms-status`), incoming message ingest (`/receive-sms`), pending recovery polling, and legacy device-scoped routes.
4. [Webhooks Endpoints (`webhooks.md`)](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/webhooks.md): Covers subscribing to SMS delivery/receipt events, HMAC SHA-256 signature verification, delivery attempt logs, and subscription lifecycle management.
5. [Administration & Support Endpoints (`administration.md`)](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/administration.md): Covers customer support ticket submission and authenticated account deletion workflows.
6. [Billing, Mail & Other Endpoints (`other-endpoints.md`)](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/other-endpoints.md): Covers Polar.sh checkout, plan catalogue, subscription retrieval, email unsubscribe/resubscribe actions, AWS SES webhooks, Polar inbound webhooks, and dashboard banner alerts.

