# Twilio SMS API Compatibility Analysis & TextBee Endpoint Mapping (Phase 2)

## Executive Summary

This documentation suite establishes the comprehensive, field-by-field, and protocol-level compatibility analysis between the **Twilio Programmable Messaging REST API** (target interface) and the **TextBee Android SMS Gateway** (underlying cellular infrastructure).

The objective of Phase 2 is to specify the architectural, contract, and payload transformations necessary to enable any existing Twilio client application or SDK to seamlessly send, receive, track, and manage SMS messages via physical Android handsets and SIM cards orchestrated by TextBee.

> [!IMPORTANT]
> **Scope Restriction (Phase 2):**
> This phase is strictly an **analysis, endpoint mapping, and technical compatibility specification**. No adapter implementation code has been added, and no existing TextBee production files or schemas have been altered.

---

## 1. Analysis Metadata

| Specification Attribute | Value | Verification Reference |
|---|---|---|
| **Target Public API** | Twilio Programmable Messaging REST API | [Twilio Official Messaging Docs](https://www.twilio.com/docs/messaging/api) |
| **Twilio API Version** | `2010-04-01` (`/2010-04-01/Accounts/{AccountSid}/Messages.json`) | Twilio Standard Messaging Resource Specification |
| **Underlying SMS Gateway** | TextBee Android SMS Gateway | Upstream: `https://github.com/textbee/textbee` |
| **Analyzed Git Branch** | `twillo-layer` | Local repository branch |
| **Analyzed Git Commit** | `7bde7e0c14f465b79032de18b4187d7e41caba6f` | Base TextBee checkout |
| **Phase 1 Baseline** | [`docs/textbee-api-analysis/`](file:///e:/projects/textbee/docs/textbee-api-analysis) | 67 discovered endpoints, OpenAPI 3.0 spec |
| **Analysis Date** | November 2026 | Technical Architecture Review |

---

## 2. Scope Boundaries

### In Scope: SMS Programmable Messaging Only
- **Outbound SMS Dispatch:** Single and multi-recipient message creation (`POST /Messages.json`).
- **Message Fetch & Status Query:** Querying single message status by identifier (`GET /Messages/{MessageSid}.json`).
- **Message History & Filtering:** Listing messages with pagination, date ranges, and recipient/sender filters (`GET /Messages.json`).
- **Inbound SMS Forwarding:** Processing incoming cellular SMS messages into Twilio webhook notifications (`POST /webhook/sms`).
- **Delivery Status Callbacks:** Forwarding cellular carrier delivery reports and handset status updates into Twilio `StatusCallback` webhooks.
- **Identifier Reconciliation:** Mapping Twilio 34-character string SIDs (`SM...` / `MM...`) to TextBee 24-character hexadecimal MongoDB ObjectIds.
- **Credential Translation:** Bridging HTTP Basic Authentication (`AccountSid:AuthToken`) to TextBee deterministic SHA-256 indexed API keys (`x-api-key`).
- **Carrier & SIM Routing:** Resolving Twilio `From` numbers and `MessagingServiceSid` to specific Android gateway devices and SIM subscription slots.
- **Error Normalization:** Translating TextBee/Android telephony errors into standard Twilio error codes (e.g., `21211`, `21606`, `30003`, `30008`).

### Out of Scope: Non-SMS Twilio Features
- Voice calls, IVR, SIP trunking, and call forwarding.
- WhatsApp Business API and social messaging channels.
- Video, conferencing, and WebRTC streaming.
- Twilio Verify, Lookup, Authy, and MFA services.
- Twilio Conversations and Flex contact center stacks.
- Virtual phone number procurement, regulatory compliance bundles, and SIP registration.
- Billing, usage records, and carrier invoice management.

---

## 3. Compatibility Classification Taxonomy

Every operation, parameter, and behavior evaluated in this analysis is classified into one of five categories:

| Status | Meaning & Criteria |
|---|---|
| **`Direct`** | TextBee provides an identical functional capability with a directly compatible contract and behavior requiring no intermediate conversion. |
| **`Adapter Required`** | TextBee fully supports the underlying functional operation, but an intermediate adapter is required to convert request encoding (`application/x-www-form-urlencoded` $\leftrightarrow$ `application/json`), rename fields, or transform response payloads. |
| **`Partial`** | TextBee supports only a subset of the capability (e.g., paging without date-range compound filters, or outbound delivery confirmation without carrier DLR guarantees). |
| **`Unsupported`** | Functionality does not exist in TextBee and cannot be synthetically produced (e.g., true MMS multimedia attachment transmission without data APN, carrier-side number portability lookup, or short code leasing). |
| **`Unverified`** | Insufficient source code or runtime documentation exists to guarantee edge-case equivalence without live cellular validation. |

---

## 4. Documentation Reading Order

To systematically evaluate and implement the Twilio compatibility layer, read the documentation suite in the following recommended sequence:

| Seq | Document Title | File Link | Stage / Implementation Workflow | Scope & Purpose |
|:---:|---|---|---|---|
| **00** | **Executive Summary & Analysis Overview** | [`00-README.md`](./00-README.md) | *1. Orientation & Governance* | Executive summary, metadata, scope boundaries, taxonomy, reading order index, and vector diagrams catalog. |
| **01** | **Twilio SMS API Inventory** | [`01-twilio-sms-api-inventory.md`](./01-twilio-sms-api-inventory.md) | *2. Target API Contract* | Complete inventory of official Twilio Messages operations, HTTP methods, headers, schemas, and specs. |
| **02** | **Twilio to TextBee SMS Compatibility Matrix** | [`02-compatibility-matrix.md`](./02-compatibility-matrix.md) | *3. Compatibility Assessment* | High-level operation-by-operation and feature-by-feature comparative matrix and compatibility grading. |
| **03** | **Twilio to TextBee Endpoint Mapping & Routing** | [`03-endpoint-mapping.md`](./03-endpoint-mapping.md) | *3. Compatibility Assessment* | Master routing table, data flow sequence topology, and technical rationale for unsupported operations. |
| **04** | **Endpoint Mapping: Send Outbound SMS** | [`04-send-sms.md`](./endpoint-details/04-send-sms.md) | *4. Endpoint Deep Dives* | Deep dive: POST Messages.json -> TextBee POST /api/v1/gateway/send-sms (GW-01). |
| **05** | **Endpoint Mapping: Retrieve Message Instance** | [`05-retrieve-sms.md`](./endpoint-details/05-retrieve-sms.md) | *4. Endpoint Deep Dives* | Deep dive: GET Messages/{Sid}.json -> TextBee GET /gateway/devices/:deviceId/sms/:smsId (GW-04). |
| **06** | **Endpoint Mapping: List SMS Message History** | [`06-list-sms.md`](./endpoint-details/06-list-sms.md) | *4. Endpoint Deep Dives* | Deep dive: GET Messages.json -> TextBee GET /gateway/messages (GW-03). |
| **07** | **Inbound SMS Webhook Translation** | [`07-inbound-sms.md`](./endpoint-details/07-inbound-sms.md) | *4. Endpoint Deep Dives* | Deep dive: Inbound SMS -> Twilio Inbound Webhook (sms.received event -> form-urlencoded). |
| **08** | **Delivery StatusCallback Webhook Translation** | [`08-status-callbacks.md`](./endpoint-details/08-status-callbacks.md) | *4. Endpoint Deep Dives* | Deep dive: Telephony DLR intents -> Twilio StatusCallback (sms.sent, sms.delivered, sms.failed). |
| **09** | **Complete Request & Response Payload Mappings** | [`09-request-response-mapping.md`](./09-request-response-mapping.md) | *5. Payload & Data Contracts* | Concrete side-by-side payload transformations for sending, retrieval, listing, and webhooks. |
| **10** | **Bidirectional Field Transformation Dictionary** | [`10-field-transformations.md`](./10-field-transformations.md) | *5. Payload & Data Contracts* | Comprehensive field mapping tables, conversions, derivations, array wrapping, and default rules. |
| **11** | **Authentication & Authorization Mapping** | [`11-authentication-mapping.md`](./11-authentication-mapping.md) | *6. Security & State Mechanics* | HTTP Basic Auth -> TextBee API Key translation strategies (zero-storage vs credential vault). |
| **12** | **HTTP Status & Error Code Dictionary** | [`12-error-mapping.md`](./12-error-mapping.md) | *6. Security & State Mechanics* | Comprehensive error mapping connecting Android RIL/TextBee failures to numeric Twilio error codes. |
| **13** | **Message Identifier Reconciliation & Strategy** | [`13-message-id-mapping.md`](./13-message-id-mapping.md) | *6. Security & State Mechanics* | Stateless 34-character SID derivation (SM00000000{objectId}) and smsBatchId reconciliation. |
| **14** | **Message Lifecycle & Status Transition Alignment** | [`14-message-status-mapping.md`](./14-message-status-mapping.md) | *6. Security & State Mechanics* | State machine alignment, status transition matrix, and carrier DLR non-downgrade behavior. |
| **15** | **Webhook Architecture & Signature Verification** | [`15-webhook-mapping.md`](./15-webhook-mapping.md) | *6. Security & State Mechanics* | Protocol translation (JSON <-> form-urlencoded) and X-Twilio-Signature HMAC-SHA1 calculation. |
| **16** | **Architectural Mismatches, Carrier Limitations & Workarounds** | [`16-compatibility-gaps.md`](./16-compatibility-gaps.md) | *7. Gaps & Constraints* | Detailed analysis of hardware cellular gaps (Alphanumeric IDs, MMS, throughput limits, and mitigations). |
| **17** | **Lightweight Compatibility Adapter Requirements (Phase 3 Blueprint)** | [`17-adapter-requirements.md`](./17-adapter-requirements.md) | *8. Implementation Blueprint* | Architectural blueprint, stateful vs stateless design, tech stack recommendations, and SSRF controls. |
| **18** | **Source Code & API Reference Bibliography** | [`18-references.md`](./18-references.md) | *9. Reference & Traceability* | Exhaustive citations of official Twilio API documentation and TextBee source code traceability. |

---

## 5. Documentation Suite Structure

```text
docs/twilio-sms-compatibility-analysis/
├── 00-README.md                          # Executive summary, metadata, scope, and reading order index
├── 01-twilio-sms-api-inventory.md        # Complete catalog of Twilio SMS operations, schemas, and specs
├── 02-compatibility-matrix.md            # High-level feature-by-feature and endpoint compatibility matrix
├── 03-endpoint-mapping.md                # One-to-one endpoint mapping summary and routing table
├── endpoint-details/                     # Deep-dive technical specifications per endpoint
│   ├── 04-send-sms.md                    # POST Messages.json -> TextBee POST /gateway/send-sms
│   ├── 05-retrieve-sms.md                # GET Messages/{Sid}.json -> TextBee GET /gateway/devices/:id/sms/:smsId
│   ├── 06-list-sms.md                    # GET Messages.json -> TextBee GET /gateway/messages
│   ├── 07-inbound-sms.md                 # TextBee POST /receive-sms -> Twilio Inbound Webhook
│   └── 08-status-callbacks.md            # TextBee PATCH /sms-status -> Twilio StatusCallback Webhook
├── 09-request-response-mapping.md        # Comprehensive payload translations and format bridges
├── 10-field-transformations.md           # Bidirectional field transformation reference dictionaries
├── 11-authentication-mapping.md          # HTTP Basic Auth vs API Key SHA-256 and AccountSid translation
├── 12-error-mapping.md                   # HTTP status and error code dictionary (Twilio 20000-30000 series)
├── 13-message-id-mapping.md              # Twilio SM... SIDs vs TextBee ObjectIds (deterministic vs stateful)
├── 14-message-status-mapping.md          # State transition alignment (queued, sending, sent, delivered, failed)
├── 15-webhook-mapping.md                 # Form-urlencoded webhook vs TextBee JSON webhook bridging
├── 16-compatibility-gaps.md              # Architectural mismatches, carrier limitations, and workarounds
├── 17-adapter-requirements.md            # Architectural blueprint and minimal responsibilities for Phase 3
├── 18-references.md                      # Source code and official Twilio documentation bibliography
└── diagrams/                             # High-resolution standalone vector SVGs
    ├── adapter-architecture.svg          # Lightweight compatibility adapter tier topology
    ├── authentication-flow.svg           # Basic Auth to x-api-key credential derivation
    ├── endpoint-data-flow-topology.svg   # Complete request/response lifecycle sequence
    ├── message-id-reconciliation.svg     # SM00000000 34-char SID vs ObjectId sequence
    ├── message-status-state-machine.svg   # Twilio to TextBee state transition machine
    ├── inbound-webhook-translation.svg   # JSON to form-urlencoded inbound webhook bridge
    ├── inbound-sms-pipeline.svg          # Android handset receive to webhook delivery
    └── status-callbacks-pipeline.svg     # Telephony RIL intent to StatusCallback relay
```

---

## 6. Vector Diagrams Suite

All architectural flows and state machines are rendered as high-resolution vector SVGs in [`diagrams/`](file:///e:/projects/textbee/docs/twilio-sms-compatibility-analysis/diagrams/):
1. **[Adapter Architecture](file:///e:/projects/textbee/docs/twilio-sms-compatibility-analysis/diagrams/adapter-architecture.svg)** (`adapter-architecture.svg`): Client SDK $\to$ Adapter $\to$ TextBee API topology.
2. **[Authentication Flow](file:///e:/projects/textbee/docs/twilio-sms-compatibility-analysis/diagrams/authentication-flow.svg)** (`authentication-flow.svg`): HTTP Basic Auth extraction and zero-storage derivation.
3. **[Endpoint Data Flow Topology](file:///e:/projects/textbee/docs/twilio-sms-compatibility-analysis/diagrams/endpoint-data-flow-topology.svg)** (`endpoint-data-flow-topology.svg`): End-to-end send, queuing, radio dispatch, and status callback relay.
4. **[Message ID Reconciliation](file:///e:/projects/textbee/docs/twilio-sms-compatibility-analysis/diagrams/message-id-reconciliation.svg)** (`message-id-reconciliation.svg`): Reversible 34-char SID transformation and batch mapping.
5. **[Message Status State Machine](file:///e:/projects/textbee/docs/twilio-sms-compatibility-analysis/diagrams/message-status-state-machine.svg)** (`message-status-state-machine.svg`): Complete Twilio $\leftrightarrow$ TextBee status transition alignment.
6. **[Inbound Webhook Translation](file:///e:/projects/textbee/docs/twilio-sms-compatibility-analysis/diagrams/inbound-webhook-translation.svg)** (`inbound-webhook-translation.svg`): Inbound handset notification translation and HMAC signing.
7. **[Inbound SMS Pipeline](file:///e:/projects/textbee/docs/twilio-sms-compatibility-analysis/diagrams/inbound-sms-pipeline.svg)** (`inbound-sms-pipeline.svg`): Hardware radio reception and backend storage.
8. **[Status Callbacks Pipeline](file:///e:/projects/textbee/docs/twilio-sms-compatibility-analysis/diagrams/status-callbacks-pipeline.svg)** (`status-callbacks-pipeline.svg`): Telephony intent to customer StatusCallback dispatch.

