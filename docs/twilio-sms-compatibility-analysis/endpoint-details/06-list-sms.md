# Endpoint Mapping: List Message Records

## 1. Twilio Endpoint Specification

- **HTTP Method:** `GET`
- **Route:** `/2010-04-01/Accounts/{AccountSid}/Messages.json`
- **Authentication:** HTTP Basic Authentication (`AccountSid` / `AuthToken`)
- **Headers:** `Accept: application/json`
- **Query Parameters:**

| Parameter | Type | Default | Description |
|---|---|:---:|---|
| `To` | `string` | — | Filter by destination phone number (E.164). |
| `From` | `string` | — | Filter by originating sender phone number. |
| `DateSent` | `string` | — | Match exact date sent (`YYYY-MM-DD`). |
| `DateSent<` | `string` | — | Filter messages sent strictly before timestamp (`YYYY-MM-DD` or ISO 8601). |
| `DateSent>` | `string` | — | Filter messages sent strictly after timestamp (`YYYY-MM-DD` or ISO 8601). |
| `PageSize` | `integer` | `50` | Number of records per page (1–1000). |
| `Page` | `integer` | `0` | Zero-based page number. |
| `PageToken`| `string` | — | Cursor token for next page navigation. |

#### Success Response (`200 OK`):
```json
{
  "first_page_uri": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages.json?PageSize=50&Page=0",
  "next_page_uri": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages.json?PageSize=50&Page=1",
  "previous_page_uri": null,
  "page": 0,
  "page_size": 50,
  "start": 0,
  "end": 49,
  "uri": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages.json?PageSize=50&Page=0",
  "messages": [
    {
      "account_sid": "ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      "api_version": "2010-04-01",
      "body": "Hello world",
      "date_created": "Wed, 01 Mar 2026 14:00:00 +0000",
      "date_sent": "Wed, 01 Mar 2026 14:00:04 +0000",
      "date_updated": "Wed, 01 Mar 2026 14:00:08 +0000",
      "direction": "outbound-api",
      "error_code": null,
      "error_message": null,
      "from": "+12025550144",
      "messaging_service_sid": null,
      "num_media": "0",
      "num_segments": "1",
      "price": "-0.00750",
      "price_unit": "USD",
      "sid": "SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      "status": "delivered",
      "subresource_uris": {
        "media": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages/SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Media.json"
      },
      "to": "+12025550199",
      "uri": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages/SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.json"
    }
  ]
}
```

---

## 2. TextBee Equivalent Endpoint Specification

- **HTTP Method:** `GET`
- **Route:** `/api/v1/gateway/messages` ([`GW-03`](file:///e:/projects/textbee/docs/textbee-api-analysis/api-endpoints/sms-messaging.md#3-get-apiv1gatewaymessages))
- **Source References:**
  - Controller: [`api/src/gateway/gateway.controller.ts:584`](file:///e:/projects/textbee/api/src/gateway/gateway.controller.ts#L584)
  - Parser: [`api/src/gateway/message-query.ts`](file:///e:/projects/textbee/api/src/gateway/message-query.ts)
  - Service: [`api/src/gateway/gateway.service.ts:getMessagesForUser`](file:///e:/projects/textbee/api/src/gateway/gateway.service.ts)
- **Authentication:** `x-api-key: <api_key>` in header.
- **Query Parameters:** `page` (1-based), `limit`, `direction` (`all`, `sent`, `received`), `status`, `search`, `from`, `to`, `order` (`desc`, `asc`).
- **Success Response (`200 OK`):**
```json
{
  "data": [
    {
      "_id": "65cb888888880fedcba98765",
      "device": "64bf800100010fedcba98765",
      "message": "Hello world",
      "type": "sent",
      "status": "delivered",
      "recipient": "+12025550199",
      "sentAt": "2026-03-01T14:00:04.000Z",
      "deliveredAt": "2026-03-01T14:00:08.000Z",
      "createdAt": "2026-03-01T14:00:00.000Z"
    }
  ],
  "meta": {
    "page": 1,
    "limit": 50,
    "total": 1,
    "pages": 1
  }
}
```

---

## 3. Compatibility Analysis

### 3.1 Pagination Model Alignment
- **Zero vs. One-Based Indexing:** Twilio `Page=0` maps to TextBee `page=1`.
  $$\text{TextBee Page} = \text{Twilio Page} + 1$$
- **Page Size Limits:** Twilio allows up to 1000 items per page; TextBee enforces `limit <= 100` in [`message-query.ts`](file:///e:/projects/textbee/api/src/gateway/message-query.ts). If a Twilio client requests `PageSize=200`, the adapter must clamp `limit=100` or execute multi-page fetches to aggregate results.
- **HATEOAS Navigation URIs:** The adapter constructs `first_page_uri`, `next_page_uri`, and `previous_page_uri` dynamically based on TextBee's `meta.pages` calculation:
  ```typescript
  const hasNextPage = textbeeMeta.page < textbeeMeta.pages;
  const nextPageUri = hasNextPage 
    ? `/2010-04-01/Accounts/${accountSid}/Messages.json?PageSize=${pageSize}&Page=${twilioPage + 1}`
    : null;
  ```

### 3.2 Filtering Capabilities
1. `To` & `From`:
   - TextBee does not have independent `to` and `from` phone filter parameters. It supports `search`, which performs a regex match over both `recipient`, `sender`, and `message` text.
   - **Adapter Strategy:** Pass phone number to `search=<phone>`. For strict filtering, the adapter filters the returned `data` array in memory before serialization.
2. `DateSent<` and `DateSent>`:
   - TextBee natively supports `from` (ISO lower bound) and `to` (ISO upper bound) on `createdAt`.
   - The adapter parses Twilio's `DateSent<` and `DateSent>` query strings and maps them to TextBee's `to` and `from` parameters.

---

## 4. Implementation Feasibility

| Assessment | Details |
|---|---|
| **Feasibility Level** | **Adapter Required (High Feasibility)** |
| **Prerequisites** | Query parameter translation, zero-to-one index conversion, and HATEOAS pagination URI synthesizer. |
| **Backend Changes Required?** | None. |

