# Error Code & HTTP Status Translation Dictionary

This document establishes the bidirectional error translation matrix between the **TextBee Platform / Android Gateway** and the **Twilio Programmable Messaging API**.

---

## 1. Synchronous REST API Request Errors

These errors occur immediately during the HTTP request lifecycle before or during message validation.

| Failure Scenario | TextBee HTTP Status | TextBee Internal Error | Twilio HTTP Status | Twilio Error Code | Twilio Error Message (`message`) | Twilio `more_info` URL | Adapter Translation Rule |
|---|:---:|---|:---:|:---:|---|---|---|
| **Missing / Invalid Credentials** | `401` | `Unauthorized` | `401` | **`20003`** | `Authentication Error - Invalid credentials` | `https://www.twilio.com/docs/errors/20003` | Intercept 401; synthesize Twilio 20003 JSON envelope. |
| **Missing `To` Recipient** | `400` | `recipients must contain at least 1 elements` | `400` | **`21604`** | `A 'To' phone number is required.` | `https://www.twilio.com/docs/errors/21604` | Validate in adapter before forwarding, or map 400 recipient error. |
| **Malformed `To` Number** | `400` | `Recipient is not a phone number` | `400` | **`21211`** | `The 'To' number is not a valid phone number.` | `https://www.twilio.com/docs/errors/21211` | Intercept TextBee `recipients.ts` regex failure and return 21211. |
| **Missing `Body` Text** | `400` | `Message cannot be blank` | `400` | **`21602`** | `Message body is required.` | `https://www.twilio.com/docs/errors/21602` | Validate in adapter or map TextBee blank message error. |
| **Unmapped `From` Number** | `400` | `Device does not exist or is not enabled` | `400` | **`21606`** | `The 'From' phone number provided is not a valid, message-capable Twilio phone number.` | `https://www.twilio.com/docs/errors/21606` | When `From` matches no enrolled SIM card or device. |
| **Quota / Plan Limit Exceeded** | `429` | `Daily send limit reached for your plan` | `429` | **`20429`** / **`30001`**| `Too Many Requests - Plan send limit exceeded` | `https://www.twilio.com/docs/errors/20429` | Map TextBee `BillingService.canPerformAction` 429 response. |
| **Message Not Found** | `404` | `SMS not found` | `404` | **`20404`** | `The requested resource was not found.` | `https://www.twilio.com/docs/errors/20404` | Map TextBee 404 to standard Twilio 20404 not found error. |
| **Account Banned** | `403` | `Account is suspended` | `403` | **`20008`** / **`30002`**| `Account has been suspended` | `https://www.twilio.com/docs/errors/20008` | Intercept `isBanned` flag on user document. |
| **Server / Redis Failure** | `500` | `Internal Server Error` | `500` | **`20500`** | `Internal Server Error` | `https://www.twilio.com/docs/errors/20500` | Synthesize standard 20500 envelope. |

#### Twilio Synchronous Error Envelope:
```json
{
  "code": 21211,
  "message": "The 'To' number +1202555INVALID is not a valid phone number.",
  "more_info": "https://www.twilio.com/docs/errors/21211",
  "status": 400
}
```

---

## 2. Asynchronous Cellular & Telephony Delivery Errors

These errors occur asynchronously after message acceptance, when the Android handset or cellular radio network reports failure via `SMSStatusReceiver.kt` or when the timeout sweeper triggers.

| Android / TextBee Telephony Failure | Android Result Code | TextBee State & Error | Twilio Callback Status | Twilio Error Code | Official Twilio Error Description |
|---|:---:|---|:---:|:---:|---|
| **Airplane Mode / Radio Off** | `2` | `RADIO_OFF` | `failed` | **`30003`** | `Unreachable destination handset - Radio is off` |
| **No Cellular Tower Signal** | `4` | `NO_SERVICE` | `failed` | **`30003`** | `Unreachable destination handset - No cellular service` |
| **Device Send Limit Exceeded** | `5` | `LIMIT_EXCEEDED` | `failed` | **`30001`** | `Queue overflow - Device/carrier throughput exceeded` |
| **Premium SMS / Short Code Blocked** | `7`, `8` | `SHORT_CODE_NOT_ALLOWED` | `failed` | **`30004`** | `Message blocked - Short code permission denied` |
| **Generic Modem Failure / Out of Credit**| `1` | `GENERIC_FAILURE` | `failed` | **`30008`** | `Unknown error - Carrier/modem rejected PDU transmission` |
| **Network Error / Tower Reject** | `17`, `10`| `NETWORK_ERROR` | `failed` | **`30008`** | `Unknown error - Mobile carrier network error` |
| **Android SMS Permission Denied** | Pre-radio | `PERMISSION_DENIED` | `failed` | **`30008`** | `Unknown error - Gateway application lacked SEND_SMS permission` |
| **Stale Timeout Sweep (>20 min)** | Cron | `unknown` | `undelivered` | **`30008`** | `Unknown error - Message unacknowledged by carrier within window` |
| **Delivery Report Canceled by Carrier** | `0` (`RESULT_CANCELED`) | `delivery_failed` | `sent` *(no error)*| *None* | Many prepaid SIM carriers do not return DLR; status preserved as `sent`. |

---

## 3. Translation Mechanics in the Adapter

```typescript
function translateTextBeeErrorToTwilio(textbeeError: any, httpStatus: number): TwilioErrorResponse {
  if (httpStatus === 401) {
    return {
      code: 20003,
      message: "Authentication Error - Invalid credentials",
      more_info: "https://www.twilio.com/docs/errors/20003",
      status: 401
    };
  }

  const message = textbeeError?.message || textbeeError?.error || "";

  if (message.includes("is not a phone number")) {
    return {
      code: 21211,
      message: "The 'To' number is not a valid phone number.",
      more_info: "https://www.twilio.com/docs/errors/21211",
      status: 400
    };
  }

  if (message.includes("cannot be blank")) {
    return {
      code: 21602,
      message: "Message body is required.",
      more_info: "https://www.twilio.com/docs/errors/21602",
      status: 400
    };
  }

  if (message.includes("Device does not exist") || message.includes("not enabled")) {
    return {
      code: 21606,
      message: "The 'From' phone number provided is not a valid, message-capable Twilio phone number.",
      more_info: "https://www.twilio.com/docs/errors/21606",
      status: 400
    };
  }

  if (httpStatus === 429) {
    return {
      code: 20429,
      message: "Too Many Requests - Plan send limit exceeded.",
      more_info: "https://www.twilio.com/docs/errors/20429",
      status: 429
    };
  }

  return {
    code: 20500,
    message: message || "Internal Server Error",
    more_info: "https://www.twilio.com/docs/errors/20500",
    status: httpStatus
  };
}
```

