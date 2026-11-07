# Field-by-Field Transformation Dictionary

This document specifies the exact field-by-field translation rules between the **Twilio Programmable Messaging API** and the **TextBee SMS Gateway API**.

---

## 1. Outbound Request: Twilio $\longrightarrow$ TextBee

| Twilio Parameter (`POST Messages.json`) | TextBee Field (`POST /gateway/send-sms`) | Target Type | Transformation Rule | Adapter Action & Fallback Policy |
|---|---|---|---|---|
| `To` | `recipients` | `string[]` | **Array Wrapping** | Takes scalar string and converts to single-element array: `recipients = [req.body.To]`. Rejects if non-numeric/invalid. |
| `Body` | `message` | `string` | **Direct Rename** | `message = req.body.Body`. Rejects with Twilio 21602 if empty and no MediaUrl present. |
| `From` | `deviceId`, `simSubscriptionId` | `string`, `number` | **Lookup & Resolution** | Look up enrolled devices: if `From` matches an active SIM's MSISDN in `device.simInfo.sims`, sets target `deviceId` and `simSubscriptionId`. If no match, falls back to default gateway device. |
| `MessagingServiceSid`| `deviceId` | `string` | **Group Mapping** | If provided, maps to an adapter-defined device pool or user default device. |
| `StatusCallback` | *Adapter State* | `string` (URL) | **Intercept & Retain** | Intercepted by adapter and saved into mapping table `(smsBatchId / smsId -> StatusCallback)` for subsequent callback delivery. |
| `SendAt` | `scheduledAt` | `string` (ISO 8601) | **Pass-through** | If present, passed to `scheduledAt`. Twilio requires `ScheduleType=fixed`. |
| `ValidityPeriod` | *Ignored* | `number` | **Unsupported** | Android gateway does not support per-message carrier validity period expiry. Ignored gracefully. |
| `MediaUrl` | *Rejected* | `string[]` | **Unsupported** | TextBee does not implement cellular MMS APN sending. If present, returns Twilio error `21620` (MMS not supported on this number). |
| `MaxPrice` | *Ignored* | `number` | **Ignored** | TextBee does not charge per-SMS credit; carrier cellular plan is used. |
| `ProvideFeedback` | *Ignored* | `boolean` | **Ignored** | Twilio Message Feedback API is out of scope. |
| `SmartEncoded` | *Pre-processed* | `boolean` | **Optional Transcode** | If `true`, adapter can transcode unicode smart quotes/emojis to GSM-7 before handing to TextBee. |

---

## 2. Outbound Response: TextBee $\longrightarrow$ Twilio

| TextBee Response Field | Twilio Property | Target Type | Transformation Rule |
|---|---|---|---|
| `data.smsBatchId` / `data._id` | `sid` | `string` | **Format Prefixing**: Converts 24-char hex ObjectId to 34-char Twilio SID: `SM00000000${objectId}`. |
| `createdAt` (or current timestamp) | `date_created` | `string` | **RFC 2822 Serialization**: `new Date(createdAt).toUTCString()`. |
| `sentAt` | `date_sent` | `string` | `null` on initial queued response; RFC 2822 timestamp once status reaches `sent` or `delivered`. |
| `updatedAt` | `date_updated` | `string` | RFC 2822 serialization of latest record update. |
| *Synthesized* | `account_sid` | `string` | Populated with the authenticated Twilio `AccountSid`. |
| *Synthesized* | `api_version` | `string` | Constant: `"2010-04-01"`. |
| `message` | `body` | `string` | Direct string assignment. |
| `recipient` / `recipients[0]` | `to` | `string` | E.164 phone number string. |
| `device.simInfo[x].number` / `From` | `from` | `string` | Physical phone number of transmitting SIM card or caller's requested `From`. |
| `status` | `status` | `string` | **State Normalization**: Maps `pending` $\to$ `queued`, `dispatched` $\to$ `sending`, `sent` $\to$ `sent`, `delivered` $\to$ `delivered`, `failed`/`unknown` $\to$ `failed`/`undelivered`. |
| *Synthesized* | `direction` | `string` | `"outbound-api"` for sent messages; `"inbound"` for received messages. |
| *Calculated* | `num_segments` | `string` | Calculated string: `"1"` if body $\le 160$ chars GSM-7 (or 70 UCS-2), otherwise `Math.ceil(length / 153).toString()`. |
| *Synthesized* | `num_media` | `string` | Constant `"0"`. |
| *Synthesized* | `price` | `string` | `null` or `"-0.00750"`. |
| *Synthesized* | `price_unit` | `string` | Constant `"USD"`. |
| `errorCode` | `error_code` | `integer` | Translated Twilio integer error code (e.g. `30003`, `30008`) or `null`. |
| `errorMessage` | `error_message` | `string` | Human-readable explanation string or `null`. |
| *Synthesized* | `uri` | `string` | Relative path: `"/2010-04-01/Accounts/{AccountSid}/Messages/{sid}.json"`. |
| *Synthesized* | `subresource_uris` | `object` | `{ "media": "/2010-04-01/Accounts/{AccountSid}/Messages/{sid}/Media.json" }`. |

---

## 3. Inbound Webhook: TextBee $\longrightarrow$ Twilio

| TextBee Webhook Data Field | Twilio Webhook Form Parameter | Format / Rule |
|---|---|---|
| `data._id` | `MessageSid`, `SmsSid`, `SmsMessageSid` | Prefix ObjectId: `SM00000000${data._id}`. |
| *Lookup via User Account* | `AccountSid` | Associated customer Account SID. |
| `data.sender` | `From` | Originator phone number in E.164 format. |
| `simUsed.number` / Device Phone | `To` | Destination gateway phone number. |
| `data.message` | `Body` | Inbound text content. |
| *Synthesized* | `NumMedia` | `'0'`. |
| *Calculated* | `NumSegments` | `'1'` (or calculated segment count). |
| *Geo-ip / SIM countryIso* | `FromCountry`, `ToCountry` | Uppercase 2-letter ISO code from SIM info (e.g., `'US'`). |
| *Synthesized* | `SmsStatus` | `'received'`. |
| *Synthesized* | `ApiVersion` | `'2010-04-01'`. |

