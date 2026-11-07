# TextBee API Endpoints: Device Management & Telemetry

This document details all 9 device-related endpoints implemented in `api/src/gateway/gateway.controller.ts` providing device registration, hardware updates, multi-SIM telemetry, heartbeat ingestion, default sending device selection, and permission checking.

---

## 1. GET /api/v1/gateway/stats

- **Endpoint ID:** `DEV-01`
- **Method:** `GET`
- **Path:** `/api/v1/gateway/stats`
- **Classification:** Public Developer API & Dashboard
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:113`
  - Service: `api/src/gateway/gateway.service.ts:getStatsForUser`
  - DTO: `GatewayStatsResponseDTO` (`api/src/gateway/gateway.dto.ts:1840`)

### Purpose
Retrieves aggregate metrics across the user's entire account: total sent SMS, total received SMS, count of active devices, and count of active API keys.

### Request Headers
- `x-api-key: <api_key>` OR `Authorization: Bearer <jwt>`

### Responses
#### Success (HTTP 200 OK)
```json
{
  "data": {
    "totalSentSMS": 1420,
    "totalReceivedSMS": 315,
    "totalDevices": 3,
    "totalApiKeys": 2
  }
}
```
- **HTTP 401 Unauthorized:** Invalid or missing authentication.

---

## 2. GET /api/v1/gateway/devices

- **Endpoint ID:** `DEV-02`
- **Method:** `GET`
- **Path:** `/api/v1/gateway/devices`
- **Classification:** Public Developer API & Dashboard
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:170`
  - Service: `api/src/gateway/gateway.service.ts:getDevicesForUser`
  - DTO: `DeviceListResponseDTO` (`api/src/gateway/gateway.dto.ts:1862`)

### Purpose
Lists all Android gateway devices paired with the caller's account.

> [!NOTE]
> Sensitive fields `fcmToken` and `serial` are stripped from the response projection (`-fcmToken -serial`) to protect mobile device credentials.

### Responses
#### Success (HTTP 200 OK)
```json
{
  "data": [
    {
      "_id": "64bf800100010fedcba98765",
      "name": "Warehouse Gateway 1",
      "enabled": true,
      "isDefault": true,
      "brand": "google",
      "manufacturer": "Google",
      "model": "Pixel 7a",
      "os": "Android",
      "osVersion": "14",
      "appVersionName": "2.8.0",
      "appVersionCode": 28,
      "sentSMSCount": 850,
      "receivedSMSCount": 120,
      "lastHeartbeat": "2026-02-05T14:20:00.000Z",
      "batteryInfo": {
        "percentage": 94,
        "isCharging": true,
        "lastUpdated": "2026-02-05T14:20:00.000Z"
      },
      "networkInfo": {
        "networkType": "wifi",
        "lastUpdated": "2026-02-05T14:20:00.000Z"
      },
      "simInfo": {
        "lastUpdated": "2026-02-05T14:20:00.000Z",
        "sims": [
          {
            "subscriptionId": 1,
            "carrierName": "T-Mobile",
            "displayName": "Business Line",
            "simSlotIndex": 0,
            "countryIso": "us",
            "subscriptionType": "PHYSICAL_SIM",
            "serviceState": "IN_SERVICE"
          }
        ]
      }
    }
  ]
}
```

---

## 3. GET /api/v1/gateway/devices/:id

- **Endpoint ID:** `DEV-03`
- **Method:** `GET`
- **Path:** `/api/v1/gateway/devices/:id`
- **Classification:** Public Developer API & Dashboard
- **Authentication:** `AuthGuard`, `CanModifyDevice`
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:191`
  - Service: `api/src/gateway/gateway.service.ts:getDeviceById`
  - DTO: `DeviceResponseDTO` (`api/src/gateway/gateway.dto.ts:1852`)

### Path Parameters
- `id` (`string`, Required): Valid MongoDB ObjectId representing the device.

### Responses
- **HTTP 200 OK:** Full device object (excluding `fcmToken` and `serial`).
- **HTTP 400 Bad Request:** Malformed device ObjectId.
- **HTTP 404 Not Found:** Device not found or belongs to another user.

---

## 4. POST /api/v1/gateway/devices

- **Endpoint ID:** `DEV-04`
- **Method:** `POST`
- **Path:** `/api/v1/gateway/devices`
- **Classification:** Android Device Onboarding & Registration
- **Authentication:** `AuthGuard` (API Key)
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:149`
  - Service: `api/src/gateway/gateway.service.ts:registerDevice`
  - DTO: `RegisterDeviceInputDTO` (`api/src/gateway/gateway.dto.ts:139`)
  - Sanitization: `pickDeviceWritableFields` (`api/src/gateway/gateway.dto.ts:285`)

### Purpose
Registers a new physical phone with the account. Called by the TextBee Android app during initial onboarding and whenever the device pairs with a new API key. Sanitized via `pickDeviceWritableFields`.

### Request Body (`RegisterDeviceInputDTO`)
```json
{
  "name": "Front Desk SMS Relay",
  "enabled": true,
  "fcmToken": "c8xY9z...FCM_PUSH_TOKEN...",
  "brand": "samsung",
  "manufacturer": "Samsung",
  "model": "SM-A536B",
  "serial": "R5CT10XXXXX",
  "buildId": "UP1A.231005.007",
  "os": "Android",
  "osVersion": "14",
  "osApiLevel": 34,
  "appVersionName": "2.8.0",
  "appVersionCode": 28,
  "simInfo": {
    "sims": [
      {
        "subscriptionId": 2,
        "carrierName": "Vodafone",
        "displayName": "SIM 1",
        "simSlotIndex": 0,
        "countryIso": "gb",
        "subscriptionType": "PHYSICAL_SIM",
        "serviceState": "IN_SERVICE"
      }
    ]
  }
}
```

### Responses
#### Success (HTTP 201 Created)
```json
{
  "data": {
    "_id": "64bf800100010fedcba98765",
    "name": "Front Desk SMS Relay",
    "enabled": true,
    "isDefault": true,
    "createdAt": "2026-02-05T14:30:00.000Z",
    "updatedAt": "2026-02-05T14:30:00.000Z"
  }
}
```
- **HTTP 400 Bad Request:** User exceeded maximum allowed enabled devices under their subscription plan (`assertDeviceLimitNotReached`).

---

## 5. PATCH /api/v1/gateway/devices/:id

- **Endpoint ID:** `DEV-05`
- **Method:** `PATCH`
- **Path:** `/api/v1/gateway/devices/:id`
- **Classification:** Android Device & Dashboard API
- **Authentication:** `AuthGuard`, `CanModifyDevice`
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:216`
  - Service: `api/src/gateway/gateway.service.ts:updateDevice`

### Purpose
Updates device properties (e.g. user-defined name, enabled/disabled toggle, refreshed FCM push token). Filters updates through `pickDeviceWritableFields`.

### Responses
- **HTTP 200 OK:** Returns updated `DeviceResponseDTO`.

---

## 6. POST /api/v1/gateway/devices/:id/heartbeat

- **Endpoint ID:** `DEV-06`
- **Method:** `POST`
- **Path:** `/api/v1/gateway/devices/:id/heartbeat`
- **Classification:** Android Device Protocol (Internal Telemetry & Sync)
- **Authentication:** `AuthGuard`, `CanModifyDevice`
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:244`
  - Service: `api/src/gateway/gateway.service.ts:heartbeat`
  - Android Client: `HeartbeatHelper.kt:154`
  - DTO: `HeartbeatInputDTO` (`api/src/gateway/gateway.dto.ts:1522`)

### Purpose
Periodic device check-in (default: every 30 minutes). Uploads hardware telemetry (battery, network, RAM, disk, uptime, OS power optimization status, permissions) and receives backend operational configurations and pending message counts.

### Request Body (`HeartbeatInputDTO`)
```json
{
  "fcmToken": "c8xY9z...UPDATED_TOKEN...",
  "batteryPercentage": 88,
  "isCharging": false,
  "networkType": "cellular",
  "appVersionName": "2.8.0",
  "appVersionCode": 28,
  "osVersion": "14",
  "osApiLevel": 34,
  "deviceUptimeMillis": 149204000,
  "memoryFreeBytes": 1048576000,
  "memoryTotalBytes": 4194304000,
  "storageAvailableBytes": 32212254720,
  "storageTotalBytes": 68719476736,
  "timezone": "Europe/London",
  "locale": "en_GB",
  "receiveSMSEnabled": true,
  "smsSendDelaySeconds": 5,
  "isIgnoringBatteryOptimizations": true,
  "isDeviceIdleMode": false,
  "isPowerSaveMode": false,
  "hasSendSmsPermission": true,
  "hasReceiveSmsPermission": true,
  "hasReadPhoneStatePermission": true,
  "hasPostNotificationsPermission": true,
  "stickyNotificationEnabled": true,
  "usingLegacyUi": false,
  "simInfo": {
    "sims": [
      {
        "subscriptionId": 1,
        "carrierName": "EE",
        "displayName": "Primary",
        "simSlotIndex": 0,
        "countryIso": "gb",
        "serviceState": "IN_SERVICE"
      }
    ]
  }
}
```

### Responses
#### Success (HTTP 200 OK)
```json
{
  "success": true,
  "fcmTokenUpdated": false,
  "lastHeartbeat": "2026-02-05T14:45:00.000Z",
  "name": "Front Desk SMS Relay",
  "pendingCount": 0,
  "config": {
    "sendSchedulerV2Enabled": true,
    "recoveryPollEnabled": false,
    "updateNotificationsEnabled": false,
    "latestVersionCode": 28,
    "latestVersionName": "2.8.0"
  }
}
```
| Return Field | Type | Description |
|---|---|---|
| `success` | `boolean` | Indicates heartbeat ingestion succeeded. |
| `fcmTokenUpdated` | `boolean` | `true` if the submitted FCM token changed and was updated. |
| `lastHeartbeat` | `string` | Timestamp recorded on the device record. |
| `name` | `string` | Device name stored in the database (allows remote renaming). |
| `pendingCount` | `number` | Count of messages awaiting delivery recovery. If > 0, Android triggers `RecoveryPoll`. |
| `config` | `DeviceConfig` | Operational flags controlling Android client behavior (e.g. scheduler V2). |

---

## 7. POST /api/v1/gateway/devices/:id/set-default

- **Endpoint ID:** `DEV-07`
- **Method:** `POST`
- **Path:** `/api/v1/gateway/devices/:id/set-default`
- **Classification:** Authenticated Dashboard API
- **Authentication:** `AuthGuard`, `CanModifyDevice`
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:290`
  - Service: `api/src/gateway/gateway.service.ts:setDefaultDevice`

### Purpose
Designates the target device as the account's primary default sender. Unsets `isDefault` on all other user devices.

### Responses
- **HTTP 200 OK:** Returns updated `DeviceResponseDTO` with `isDefault: true`.

---

## 8. DELETE /api/v1/gateway/devices/:id

- **Endpoint ID:** `DEV-08`
- **Method:** `DELETE`
- **Path:** `/api/v1/gateway/devices/:id`
- **Classification:** Authenticated Dashboard API
- **Authentication:** `AuthGuard`, `CanModifyDevice`
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:268`
  - Service: `api/src/gateway/gateway.service.ts:deleteDevice`

### Purpose
Unpairs and permanently removes the device. Creates a record in `DeviceTombstone` collection. Associated message history is preserved in `SMS` collection.

### Responses
- **HTTP 200 OK:** `{ "data": { "success": true, "message": "Device deleted successfully" } }`

---

## 9. GET /api/v1/gateway/sms-permission-status

- **Endpoint ID:** `DEV-09`
- **Method:** `GET`
- **Path:** `/api/v1/gateway/sms-permission-status`
- **Classification:** Dashboard Diagnostic API
- **Authentication:** `AuthGuard` (API Key / JWT)
- **Source References:**
  - Controller: `api/src/gateway/gateway.controller.ts:131`
  - Service: `api/src/gateway/gateway.service.ts:getSmsPermissionStatus`
  - DTO: `SmsPermissionStatusResponseDTO` (`api/src/gateway/gateway.dto.ts:1785`)

### Purpose
Inspects recent messages and device heartbeats. If a recent send failed with `PERMISSION_DENIED` or an enabled device reported `hasSendSmsPermission = false` in a heartbeat within the last 7 days, alerts the dashboard that Android permissions must be granted on the phone.

### Responses
#### Success (HTTP 200 OK)
```json
{
  "data": {
    "permissionNeeded": false,
    "source": null,
    "deviceId": null,
    "deviceName": null
  }
}
```
Or when permission is missing:
```json
{
  "data": {
    "permissionNeeded": true,
    "source": "failure",
    "deviceId": "64bf800100010fedcba98765",
    "deviceName": "Front Desk SMS Relay"
  }
}
```

