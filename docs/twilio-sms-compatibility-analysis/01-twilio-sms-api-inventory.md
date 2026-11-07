# Twilio SMS API Inventory & Specification Reference

This document provides a comprehensive technical catalog of the official **Twilio Programmable Messaging REST API** (API version `2010-04-01`), detailing all SMS-related operations, parameter constraints, authentication requirements, and response contracts.

---

## 1. Authentication & Base Conventions

- **Base URL:** `https://api.twilio.com/2010-04-01`
- **Authentication Scheme:** HTTP Basic Authentication
  - **Username:** Twilio Account SID (`ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`, 34 alphanumeric characters).
  - **Password:** Twilio Auth Token (32-character hexadecimal string) OR Twilio API Key Secret.
- **Request Encoding:** `application/x-www-form-urlencoded` for `POST` requests.
- **Response Format:** `application/json` (when requested with `.json` path suffix or `Accept: application/json`).
- **Timestamp Format:** RFC 2822 (e.g., `Wed, 01 Mar 2026 14:00:00 +0000`).

---

## 2. Twilio Programmable Messaging Operations Catalog

### TW-01: Create Message (Send Outbound SMS)

- **Method:** `POST`
- **Path:** `/2010-04-01/Accounts/{AccountSid}/Messages.json`
- **Purpose:** Initiates sending an SMS or MMS message to a destination phone number.
- **Content-Type:** `application/x-www-form-urlencoded`
- **Success Status:** `201 Created`

#### Request Parameters:

| Parameter | Type | Required? | Description & Validation Rules |
|---|---|:---:|---|
| `To` | `string` | **Yes** | Destination recipient phone number in E.164 format (e.g., `+12025550199`) or SIP endpoint. |
| `From` | `string` | **Conditional** | Twilio phone number, short code, or Alphanumeric Sender ID. Required unless `MessagingServiceSid` is supplied. |
| `MessagingServiceSid` | `string` | **Conditional** | 34-character SID of a Messaging Service (`MG...`). Required if `From` is omitted. |
| `Body` | `string` | **Conditional** | The text content of the message. Up to 1,600 characters. Required unless `MediaUrl` is provided. |
| `MediaUrl` | `string[]` | No | URL of media attachments (up to 10 for MMS). *(Out of SMS scope)* |
| `StatusCallback` | `string` | No | Full HTTPS URL to which Twilio will send asynchronous delivery status updates. |
| `ApplicationSid` | `string` | No | SID of a Twilio Application whose configured `StatusCallback` will be used. |
| `MaxPrice` | `number` | No | The maximum price in USD allowed for sending this message. |
| `ProvideFeedback` | `boolean` | No | Whether message delivery will be confirmed via Twilio Message Feedback API. Default: `false`. |
| `ValidityPeriod` | `integer` | No | How long (in seconds, 1–14400) the message is allowed to wait in the queue before timing out. |
| `ForceDelivery` | `boolean` | No | Bypasses local carrier filtering checks when using short codes. |
| `SmartEncoded` | `boolean` | No | Automatically converts unicode characters to GSM-7 equivalents. Default: `false`. |
| `ScheduleType` | `string` | No | Set to `'fixed'` for scheduled messaging. Requires `SendAt` and `MessagingServiceSid`. |
| `SendAt` | `string` | No | ISO 8601 timestamp (15 minutes to 7 days in future) when scheduled message should send. |
| `SendAsMms` | `boolean` | No | Forces transmission as MMS even when text is short. |

#### Example Request:
```http
POST /2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages.json HTTP/1.1
Host: api.twilio.com
Authorization: Basic QUNYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYzp5b3VyX2F1dGhfdG9rZW4=
Content-Type: application/x-www-form-urlencoded

To=%2B12025550199&From=%2B12025550144&Body=Your+verification+code+is+839210&StatusCallback=https%3A%2F%2Fclient.example.com%2Fstatus
```

#### Success Response (`201 Created`):
```json
{
  "account_sid": "ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "api_version": "2010-04-01",
  "body": "Your verification code is 839210",
  "date_created": "Wed, 01 Mar 2026 14:00:00 +0000",
  "date_sent": null,
  "date_updated": "Wed, 01 Mar 2026 14:00:00 +0000",
  "direction": "outbound-api",
  "error_code": null,
  "error_message": null,
  "from": "+12025550144",
  "messaging_service_sid": null,
  "num_media": "0",
  "num_segments": "1",
  "price": null,
  "price_unit": "USD",
  "sid": "SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "status": "queued",
  "subresource_uris": {
    "media": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages/SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Media.json"
  },
  "to": "+12025550199",
  "uri": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages/SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.json"
}
```

---

### TW-02: Fetch Message Instance

- **Method:** `GET`
- **Path:** `/2010-04-01/Accounts/{AccountSid}/Messages/{MessageSid}.json`
- **Purpose:** Retrieves the current state, timestamps, carrier error codes, and price of a specific message record.
- **Success Status:** `200 OK`

#### Path Parameters:
- `AccountSid` (`string`, required): Twilio account SID.
- `MessageSid` (`string`, required): 34-character string starting with `SM` (SMS) or `MM` (MMS).

#### Success Response (`200 OK`):
```json
{
  "account_sid": "ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "api_version": "2010-04-01",
  "body": "Your verification code is 839210",
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
```

---

### TW-03: List Messages (Query Message History)

- **Method:** `GET`
- **Path:** `/2010-04-01/Accounts/{AccountSid}/Messages.json`
- **Purpose:** Reads a paginated list of message resources with optional filtering by sender, recipient, and date sent.
- **Success Status:** `200 OK`

#### Supported Query Parameters:

| Parameter | Type | Default | Description |
|---|---|:---:|---|
| `To` | `string` | — | Filter messages by recipient phone number in E.164 format. |
| `From` | `string` | — | Filter messages by sender phone number or alphanumeric sender ID. |
| `DateSent` | `string` | — | Filter messages sent on a date (`YYYY-MM-DD`). |
| `DateSent<` | `string` | — | Filter messages sent before a date/time (`YYYY-MM-DD` or ISO 8601). |
| `DateSent>` | `string` | — | Filter messages sent after a date/time (`YYYY-MM-DD` or ISO 8601). |
| `PageSize` | `integer` | `50` | Number of messages to return per page (min 1, max 1000). |
| `Page` | `integer` | `0` | Zero-based page number to fetch. |
| `PageToken`| `string` | — | Opaque cursor token for next page navigation. |

#### Success Response (`200 OK`):
```json
{
  "first_page_uri": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages.json?PageSize=50&Page=0",
  "next_page_uri": "/2010-04-01/Accounts/ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/Messages.json?PageSize=50&Page=1&PageToken=PAb1c2...",
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
      "body": "Your verification code is 839210",
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

### TW-04: Update Message (Cancel Queued / Redact Body)

- **Method:** `POST`
- **Path:** `/2010-04-01/Accounts/{AccountSid}/Messages/{MessageSid}.json`
- **Purpose:** 
  1. Cancels a scheduled or queued message prior to transmission by setting `Status=canceled`.
  2. Redacts the message text by setting `Body=""`.
- **Request Body:** `application/x-www-form-urlencoded`
- **Success Status:** `200 OK`

---

### TW-05: Delete Message Record

- **Method:** `DELETE`
- **Path:** `/2010-04-01/Accounts/{AccountSid}/Messages/{MessageSid}.json`
- **Purpose:** Deletes the record and media attachments from Twilio's database. Does not unsend an already transmitted message.
- **Success Status:** `204 No Content`

---

### TW-06: Inbound SMS Webhook

- **Transport:** HTTP POST (or GET) dispatched to the customer's webhook URL configured on their Twilio Phone Number or Messaging Service.
- **Content-Type:** `application/x-www-form-urlencoded`
- **Payload Schema:**
  - `MessageSid` / `SmsSid`: Unique identifier (`SM...`).
  - `AccountSid`: Receiving account SID.
  - `From`: Originating caller/mobile phone number.
  - `To`: Destination Twilio phone number.
  - `Body`: Inbound message text.
  - `NumMedia`: `'0'` for standard SMS.
  - `NumSegments`: Message carrier segment count.
  - `FromCity`, `FromState`, `FromZip`, `FromCountry`: Telco geo-metadata.

---

### TW-07: Delivery StatusCallback Webhook

- **Transport:** Asynchronous HTTP POST sent to the URL provided in `POST Messages.json (StatusCallback=...)`.
- **Content-Type:** `application/x-www-form-urlencoded`
- **Fired Events:** `queued`, `sent`, `delivered`, `undelivered`, `failed`.
- **Payload Schema:**
  - `MessageSid` / `SmsSid`: Message SID.
  - `MessageStatus` / `SmsStatus`: Current lifecycle state (`queued`, `sent`, `delivered`, `undelivered`, `failed`).
  - `To`: Destination phone number.
  - `From`: Sender identity.
  - `AccountSid`: Account SID.
  - `ApiVersion`: `2010-04-01`.
  - `ErrorCode`: (Optional) Integer error code if status is `failed` or `undelivered` (e.g., `30003`, `30008`).
  - `ErrorMessage`: (Optional) Error string description.

