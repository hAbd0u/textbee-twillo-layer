# Complete Request & Response Payload Mappings

This document provides complete, side-by-side, bidirectional request and response payload transformations between the **Twilio Programmable Messaging API** and the **TextBee SMS Gateway API**.

---

## 1. Outbound SMS Dispatch

### 1.1 Inbound Twilio Request $\longrightarrow$ Outbound TextBee Request

```text
[Twilio Client / SDK] 
       │
       │ HTTP POST /2010-04-01/Accounts/AC.../Messages.json
       │ Content-Type: application/x-www-form-urlencoded
       ▼
```
**Twilio Incoming Request Payload:**
```http
POST /2010-04-01/Accounts/AC3f890123456789abcdef0123456789ab/Messages.json HTTP/1.1
Host: api.twilio.com
Authorization: Basic QUMzZjg5MDEyMzQ1Njc4OWFiY2RlZjAxMjM0NTY3ODlhYjpkM2Y5YThjMTJlM2Y0YTViNmE3Yjg5MDFjMmQzZTQ=
Content-Type: application/x-www-form-urlencoded

To=%2B12025550199&From=%2B12025550144&Body=Your+security+passcode+is+710492&StatusCallback=https%3A%2F%2Fclient.example.com%2Ftwilio%2Fcallbacks
```

```text
       │
       │ [Adapter Layer Transformation]
       │ • Extract Basic Auth -> Look up TextBee API Key
       │ • Parse Form-urlencoded -> JSON
       │ • To -> recipients: [To]
       │ • Body -> message
       │ • From -> deviceId & simSubscriptionId
       │ • StatusCallback -> Cache (smsBatchId -> URL)
       ▼
```
**Transformed TextBee Request Payload:**
```http
POST /api/v1/gateway/send-sms HTTP/1.1
Host: api.textbee.dev
x-api-key: tb_live_a1b2c3d4e5f60718293a4b5c6d7e8f90
Content-Type: application/json

{
  "message": "Your security passcode is 710492",
  "recipients": ["+12025550199"],
  "deviceId": "64bf800100010fedcba98765",
  "simSubscriptionId": 1
}
```

---

### 1.2 Inbound TextBee Response $\longrightarrow$ Outbound Twilio Response

```text
[TextBee Backend]
       │
       │ HTTP 200 OK (Queued Mode)
       ▼
```
**TextBee Incoming Response Payload:**
```json
{
  "data": {
    "success": true,
    "message": "SMS added to queue for processing",
    "smsBatchId": "65cb76543210fedcba987654",
    "recipientCount": 1,
    "estimatedCompletionAt": "2026-03-01T14:00:05.000Z"
  }
}
```

```text
       │
       │ [Adapter Layer Transformation]
       │ • Status HTTP 200 -> HTTP 201 Created
       │ • smsBatchId -> MessageSid (SM0000000065cb76543210fedcba987654)
       │ • Set status: "queued", direction: "outbound-api"
       │ • Format RFC 2822 timestamps
       │ • Build Twilio HATEOAS relative URIs
       ▼
```
**Transformed Twilio Response Payload:**
```http
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8

{
  "account_sid": "AC3f890123456789abcdef0123456789ab",
  "api_version": "2010-04-01",
  "body": "Your security passcode is 710492",
  "date_created": "Sun, 01 Mar 2026 14:00:00 +0000",
  "date_sent": null,
  "date_updated": "Sun, 01 Mar 2026 14:00:00 +0000",
  "direction": "outbound-api",
  "error_code": null,
  "error_message": null,
  "from": "+12025550144",
  "messaging_service_sid": null,
  "num_media": "0",
  "num_segments": "1",
  "price": null,
  "price_unit": "USD",
  "sid": "SM0000000065cb76543210fedcba987654",
  "status": "queued",
  "subresource_uris": {
    "media": "/2010-04-01/Accounts/AC3f890123456789abcdef0123456789ab/Messages/SM0000000065cb76543210fedcba987654/Media.json"
  },
  "to": "+12025550199",
  "uri": "/2010-04-01/Accounts/AC3f890123456789abcdef0123456789ab/Messages/SM0000000065cb76543210fedcba987654.json"
}
```

---

## 2. Fetch Single Message Instance

### 2.1 Inbound Twilio Request $\longrightarrow$ TextBee Query

```http
GET /2010-04-01/Accounts/AC3f890123456789abcdef0123456789ab/Messages/SM0000000065cb888888880fedcba98765.json HTTP/1.1
Host: api.twilio.com
Authorization: Basic QUN...
```
```text
       │ [Adapter Layer Transformation]
       │ • Extract ObjectId: 65cb888888880fedcba98765
       │ • Look up cached deviceId OR query unified history:
       ▼
GET /api/v1/gateway/messages?search=65cb888888880fedcba98765 HTTP/1.1
Host: api.textbee.dev
x-api-key: tb_live_...
```

### 2.2 TextBee Response $\longrightarrow$ Twilio Response

**TextBee Returned Document:**
```json
{
  "_id": "65cb888888880fedcba98765",
  "device": "64bf800100010fedcba98765",
  "message": "Your security passcode is 710492",
  "type": "sent",
  "status": "delivered",
  "recipient": "+12025550199",
  "requestedAt": "2026-03-01T14:00:00.000Z",
  "sentAt": "2026-03-01T14:00:04.000Z",
  "deliveredAt": "2026-03-01T14:00:08.000Z",
  "createdAt": "2026-03-01T14:00:00.000Z",
  "updatedAt": "2026-03-01T14:00:08.000Z"
}
```

**Transformed Twilio Response Payload:**
```json
{
  "account_sid": "AC3f890123456789abcdef0123456789ab",
  "api_version": "2010-04-01",
  "body": "Your security passcode is 710492",
  "date_created": "Sun, 01 Mar 2026 14:00:00 +0000",
  "date_sent": "Sun, 01 Mar 2026 14:00:04 +0000",
  "date_updated": "Sun, 01 Mar 2026 14:00:08 +0000",
  "direction": "outbound-api",
  "error_code": null,
  "error_message": null,
  "from": "+12025550144",
  "messaging_service_sid": null,
  "num_media": "0",
  "num_segments": "1",
  "price": "-0.00750",
  "price_unit": "USD",
  "sid": "SM0000000065cb888888880fedcba98765",
  "status": "delivered",
  "subresource_uris": {
    "media": "/2010-04-01/Accounts/AC3f890123456789abcdef0123456789ab/Messages/SM0000000065cb888888880fedcba98765/Media.json"
  },
  "to": "+12025550199",
  "uri": "/2010-04-01/Accounts/AC3f890123456789abcdef0123456789ab/Messages/SM0000000065cb888888880fedcba98765.json"
}
```

---

## 3. Inbound SMS Webhook Payload

```text
[TextBee Handset Ingest -> WebhookService Notification]
       │
       │ JSON POST (Event: sms.received)
       ▼
```
**TextBee Incoming Webhook:**
```json
{
  "event": "sms.received",
  "data": {
    "_id": "65cb999999990fedcba98765",
    "device": "64bf800100010fedcba98765",
    "sender": "+12025550177",
    "message": "START",
    "receivedAt": "2026-03-01T14:15:00.000Z",
    "type": "received",
    "status": "received",
    "simUsed": {
      "subscriptionId": 1,
      "carrierName": "T-Mobile"
    }
  }
}
```

```text
       │
       │ [Adapter Layer Transformation]
       │ • Look up gateway SIM phone number -> To: +12025550144
       │ • Generate MessageSid: SM0000000065cb999999990fedcba98765
       │ • Serialize application/x-www-form-urlencoded
       │ • Compute X-Twilio-Signature with AuthToken
       ▼
```
**Transformed Twilio Customer Inbound Webhook:**
```http
POST /api/sms/inbound HTTP/1.1
Host: client.example.com
X-Twilio-Signature: 2jmj7l5rSw0yVb/vlWAYkK/YBwk=
Content-Type: application/x-www-form-urlencoded

AccountSid=AC3f890123456789abcdef0123456789ab&ApiVersion=2010-04-01&Body=START&From=%2B12025550177&FromCity=Washington&FromCountry=US&FromState=DC&FromZip=20001&MessageSid=SM0000000065cb999999990fedcba98765&NumMedia=0&NumSegments=1&SmsMessageSid=SM0000000065cb999999990fedcba98765&SmsSid=SM0000000065cb999999990fedcba98765&SmsStatus=received&To=%2B12025550144&ToCountry=US
```

