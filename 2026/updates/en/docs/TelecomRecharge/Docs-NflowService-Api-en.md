# API Interface Documentation – Electronic Payment Services (Nflow Integration)

**Version:** v1  
**Base Path:** `https://[domain]/api/v1/telecomrecharge/nflow`  
**Or:** `http://localhost:8006/api/v1/telecomrecharge/nflow`  

**Authentication:** OAuth 2.0 (Access token passed in header `Authorization: Bearer <token>`)

---

## Overview

This document explains all available endpoints for interacting with the **nflow.tech** platform (Master Host) via the API integrated into the `Nano3.TelecomRecharge` add-on. Services covered:

- Agent account management (balance, deposits, transaction status)
- Balance inquiries for Yemeni telecommunications companies (Yemen Mobile, ADSL, Land Phone, Yemen 4G, Aden Net)
- Direct balance recharge for all supported networks
- Activation of internet packages and services
- Recharging games and gift cards (PUBG, Yoyo Chat, etc.)

All requests require a **valid OAuth token** (added in the header) and use `POST` or `GET` methods as needed. All endpoints respond with a unified JSON format.

---

## Unified Response Structure

All endpoints return a response with the same basic structure:

```json
{
    "code": 200,
    "status": true,
    "message": "Message text",
    "error": null,
    "errors": [],
    "data": {},
    "meta": {},
    "input_data": {},
    "process_data": {},
    "debug": {}
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `code` | integer | HTTP code (200 for success, 400, 401, 402, 403, 404, 500 for errors) |
| `status` | boolean | `true` for success, `false` for failure |
| `message` | string | Descriptive message suitable for the user |
| `error` | string | Primary error message (in case of failure) |
| `errors` | array | Array of detailed error messages (e.g., validation errors) |
| `data` | mixed | Main response data (array or object) |
| `meta` | mixed | Additional data (e.g., pagination info, elapsed time) |
| `input_data` | mixed | Data sent by the user (for tracking) |
| `process_data` | mixed | Internal processing data (e.g., service result) |
| `debug` | mixed | Debug information (only appears when `app.debug` is enabled) |

---

## Authentication

The access token is sent in the `Authorization` header in the format:

```
Authorization: Bearer {access_token}
```

This token is obtained through the application's OAuth system. (If you are using October CMS with Laravel OAuth, it is added automatically after login).

---

## Endpoints

### 1. General Operations (Account Operations)

#### 1.1 Query Agent Balance

**GET** `/account-balance`

**Description:** Retrieve the current balance of the agent account.

**Example Request:**

```bash
curl -X GET "http://localhost:8006/api/v1/telecomrecharge/nflow/account-balance" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Successful Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Balance queried successfully",
    "error": null,
    "errors": null,
    "data": {
        "status": true,
        "agentBalance": 1500.50,
        "message": "Agent Balance Query Success",
        "transactionID": 0
    },
    "input_data": {},
    "process_data": {
        "balance_result": { ... }
    }
}
```

---

#### 1.2 Query Status of a Previous Transaction

**GET** `/operation-status?transactionId={transactionId}`

**Description:** Query the status of a previously submitted transaction.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| transactionId | string | Yes | Your transaction ID (as sent in the original request) |

**Example Request:**

```bash
curl -X GET "http://localhost:8006/api/v1/telecomrecharge/nflow/operation-status?transactionId=TXN_123" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Successful Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Transaction status queried successfully",
    "data": {
        "status": true,
        "operationStatus": 1,
        "mobileNumber": "777777777",
        "price": 100,
        "message": "Ready",
        "transactionID": "TXN_123",
        "referenceID": 98765
    },
    "input_data": {
        "transactionId": "TXN_123"
    }
}
```

**Status Codes:**  
- `operationStatus = 1` → Success  
- `operationStatus = 0` → Failure  
- `operationStatus = -1` → Pending

---

#### 1.3 Query Agent Deposits

**GET** `/feed-clients-balance`

**Description:** View the list of deposits added to the agent account.

**Successful Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Agent deposits fetched successfully",
    "data": {
        "status": true,
        "data": [
            {
                "Date": "2024-08-03",
                "Amount": "70000.00000",
                "Currency": "Yemeni Rial",
                "Notes": "Notes"
            }
        ],
        "message": "Query Success"
    }
}
```

---

#### 1.4 Fetch Game and Gift Card Categories

**GET** `/games-categories`

**Description:** Get a list of available games and cards with their prices and required fields.

**Successful Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Game and gift card categories fetched successfully",
    "data": {
        "status": true,
        "data": [
            {
                "TheNumber": 1,
                "ServiceName": "PUBG",
                "CategoryName": "PUBG (60 UC)",
                "LinkCode": "pubg_60",
                "LocalPrice": 467.61,
                "LocalCurrencyName": "Yemeni Rial",
                "RequiredFields": [
                    { "FieldCode": "PlayerID", "FieldName": "Player ID" }
                ]
            }
        ],
        "message": "Success"
    }
}
```

---

### 2. Generic Functions (Flexible)

#### 2.1 Execute Custom Service

**POST** `/execute`

**Description:** Execute any service by manually specifying the network number and service number (maximum flexibility).

**Request Body (JSON):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| networkNumber | int | Yes | Network number (e.g., 1 for Yemen Mobile) |
| serviceNumber | int | Yes | Service number (e.g., 101 for Yemen Mobile balance inquiry) |
| additionalFields | object | No | Additional fields such as `MobileNumber`, `Amount`, `OfferCode` |
| transactionId | string | No | Transaction ID (auto-generated if not passed) |

**Example Request:**

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/execute" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "networkNumber": 1,
    "serviceNumber": 102,
    "additionalFields": {
        "MobileNumber": "777777777"
    },
    "transactionId": "TXN_123"
  }'
```

**Successful Response (200):**  
Depends on the service, but typically includes `operationStatus`, `agentBalance`, and `price`.

---

#### 2.2 Query Balance for Any Network

**GET** `/balance/query?networkNumber={networkNumber}&mobileNumber={mobileNumber}&transactionId={transactionId}`

**Description:** Query the balance of a number on any network that supports the inquiry service.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| networkNumber | int | Yes | Network number (1,5,6,7,13) |
| mobileNumber | string | Yes | Mobile number or line |
| transactionId | string | No | Transaction ID (auto-generated) |

**Example Request:**

```bash
curl -X GET "http://localhost:8006/api/v1/telecomrecharge/nflow/balance/query?networkNumber=1&mobileNumber=777777777" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Successful Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Balance queried successfully",
    "data": {
        "status": true,
        "mobileBalance": 154.15,
        "mobileTypeName": "Prepaid",
        "message": "Balance Query Success"
    },
    "input_data": {
        "networkNumber": "1",
        "mobileNumber": "777777777"
    }
}
```

**Error Cases:**  
- If the network does not support balance inquiry → Code 400 with message "This network does not support balance inquiry service".

---

#### 2.3 Recharge Balance for Any Network

**POST** `/balance/charge`

**Description:** Recharge balance for a number on any network that supports the direct recharge service.

**Request Body (JSON):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| networkNumber | int | Yes | Network number |
| mobileNumber | string | Yes | Mobile number or line |
| amount | float | Yes | Amount (for networks using units, this represents the number of units) |
| transactionId | string | No | Transaction ID (auto-generated) |
| webHookUrl | string | No | WebHook URL to receive status updates |
| webHookCode | string | No | Security code for the WebHook |

**Example Request:**

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/balance/charge" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "networkNumber": 1,
    "mobileNumber": "777777777",
    "amount": 500,
    "transactionId": "TXN_456",
    "webHookUrl": "https://example.com/webhook"
  }'
```

**Successful Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Recharge process completed successfully",
    "data": {
        "status": true,
        "operationStatus": 1,
        "agentBalance": 1500.50,
        "price": 500,
        "transactionID": "TXN_456",
        "referenceID": 123456
    },
    "input_data": { ... },
    "process_data": { "charge_result": { ... } }
}
```

---

#### 2.4 Activate Package for Any Network

**POST** `/offer/activate`

**Description:** Activate an internet package or service for a number on any network that supports packages.

**Request Body (JSON):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| networkNumber | int | Yes | Network number |
| mobileNumber | string | Yes | Mobile number |
| offerCode | string | Yes | Package code (e.g., `PREWhatsApp`) |
| transactionId | string | No | Transaction ID |
| webHookUrl | string | No | WebHook URL |
| webHookCode | string | No | Security code |

**Example Request:**

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/offer/activate" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "networkNumber": 2,
    "mobileNumber": "777777777",
    "offerCode": "PREWhatsApp",
    "transactionId": "TXN_789"
  }'
```

**Successful Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Package activated successfully",
    "data": {
        "status": true,
        "operationStatus": 1,
        "agentBalance": 1490.00,
        "price": 10.50,
        "transactionID": "TXN_789",
        "referenceID": 78901
    }
}
```

---

### 3. Balance Queries for Networks (Shortcuts)

| Method | Route | Description | Example |
|--------|-------|-------------|---------|
| GET | `/yemen-mobile/balance?mobileNumber={num}` | Yemen Mobile balance | `?mobileNumber=777777777` |
| GET | `/yemen-mobile/loan?mobileNumber={num}` | Yemen Mobile loan | |
| GET | `/yemen-mobile/offers?mobileNumber={num}` | Yemen Mobile packages | |
| GET | `/adsl/balance?mobileNumber={num}` | ADSL balance | |
| GET | `/land-phone/balance?mobileNumber={num}` | Land Phone balance | |
| GET | `/yemen-4g/balance?mobileNumber={num}` | Yemen 4G balance | |
| GET | `/aden-net/balance?mobileNumber={num}` | Aden Net balance | |

**Example Response (Yemen Mobile):**

```json
{
    "code": 200,
    "status": true,
    "message": "Query successful",
    "data": {
        "status": true,
        "mobileBalance": 154.15,
        "mobileTypeName": "Prepaid",
        "message": "Balance Query Success"
    },
    "input_data": {
        "mobileNumber": "777777777"
    }
}
```

---

### 4. Recharge Balance (Shortcuts)

#### 4.1 Recharge Yemen Mobile

**POST** `/yemen-mobile/charge`

**Request Body (JSON):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| mobileNumber | string | Yes | Mobile number |
| amount | float | Yes | Amount |
| transactionId | string | Yes | Transaction ID |
| webHookUrl | string | No | WebHook URL |
| webHookCode | string | No | WebHook code |

**Response:** Similar to `balance/charge`.

---

### 5. Activate Packages (Shortcuts)

| Method | Route | Description | Example Request Body |
|--------|-------|-------------|----------------------|
| POST | `/you/offer/activate` | Activate YOU Telecom package | `{"mobileNumber":"777777777","offerCode":"PREWhatsApp","transactionId":"..."}` |
| POST | `/sabafon/offer/activate` | Activate Sabafon package | `{"mobileNumber":"777777777","offerCode":"SAB_OFFER","transactionId":"..."}` |

---

### 6. Games and Gift Cards

#### 6.1 Recharge Game or Card

**POST** `/game-charge`

**Request Body (JSON):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| linkCode | string | Yes | Game code (from game categories) |
| fields | object | Yes | Required fields (e.g., `PlayerID`) |
| quantity | int | Yes | Quantity (for games allowing variable quantity) |
| transactionId | string | Yes | Transaction ID |
| mobileNumber | string | No | Mobile number if required |
| webHookUrl | string | No | WebHook URL |
| webHookCode | string | No | WebHook code |

**Example Request (PUBG):**

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/game-charge" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "linkCode": "pubg_60",
    "fields": {"PlayerID": "123456789"},
    "quantity": 1,
    "transactionId": "TXN_123"
  }'
```

**Successful Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Game or gift card recharged successfully",
    "data": {
        "status": true,
        "operationStatus": 1,
        "agentBalance": 1440.00,
        "price": 467.61,
        "transactionID": "TXN_123",
        "referenceID": 98765
    }
}
```

---

### 7. Data and Helper Functions

#### 7.1 Get List of Networks

**GET** `/networks`

**Description:** Returns an array of network numbers and names.

**Successful Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Network list fetched successfully",
    "data": {
        "0": "Global",
        "1": "Yemen Mobile",
        "2": "YOU Telecom",
        "3": "Sabafon",
        "4": "Y Telecom",
        "5": "ADSL",
        "6": "Land Phone",
        "7": "Yemen 4G",
        "8": "Games And Gift Cards",
        "9": "Games And Gift Cards",
        "12": "Sabafon South",
        "13": "Aden Net"
    }
}
```

---

#### 7.2 Get List of Services

**GET** `/services`

**Description:** Returns a full list of services for each network (as in the Services Data table).

**Successful Response (200):**  
Returns an array similar to `getAllServices()` in the class.

---

#### 7.3 Test Connection

**GET** `/test-connection`

**Description:** Test credential validity and ability to connect to nflow.tech.

**Successful Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Connection test completed",
    "data": {
        "success": true,
        "message": "Connection successful",
        "token": true
    }
}
```

---

## Response Codes (HTTP Code)

| Code | Meaning |
|------|---------|
| 200 | Operation successful |
| 400 | Bad request (missing or invalid data) |
| 401 | Authentication failed (invalid or expired token) |
| 402 | Insufficient balance |
| 403 | Forbidden (number blocked or insufficient permissions) |
| 404 | Service or number not found |
| 406 | Invalid format (e.g., incorrect phone number) |
| 416 | Value out of allowed range |
| 423 | Service locked or temporarily unavailable |
| 429 | Rate limit exceeded |
| 500 | Internal server error |

All responses contain a `status` field (true/false) and a descriptive `message` field.

---

## Error Handling

In case of an error, the endpoint returns a response with a unified structure:

```json
{
    "code": 400,
    "status": false,
    "message": "Invalid data",
    "error": "Validation error",
    "errors": [
        "networkNumber is required"
    ],
    "input_data": { ... },
    "debug": { ... } // Only appears when app.debug is enabled
}
```

**Examples of common errors:**

| Error | Code | Message |
|-------|------|---------|
| Missing data | 400 | "networkNumber and mobileNumber are required" |
| Authentication failed | 401 | "Authentication failed, check credentials" |
| Insufficient balance | 402 | "Insufficient balance" |
| Number blocked | 403 | "Number is blocked" |
| Service not found | 404 | "Service or number not found" |
| Invalid number format | 406 | "Phone number format is incorrect" |
| Service locked | 423 | "Service temporarily locked" |
| Rate limit | 429 | "Rate limit exceeded" |

---

## Notes

- **Transaction ID (transactionId):** It is recommended to be unique for each request to avoid duplication. Can be generated with `uniqid('TXN_', true)`.
- **Webhook:** When passing `webHookUrl`, the platform will send status updates (GET) to this URL with parameters `OperationStatus`, `TransactionID`, `ReferenceID`, `price`, `message`.
- **Unit Networks:** For networks (2) YOU, (3) Sabafon, (4) Y Telecom, (12) Sabafon South, the `amount` value represents the number of units, not the monetary amount. It is multiplied by the unit price set in the system.
- **OAuth Compatibility:** All endpoints are protected by `oauth-users`, so the user must be logged in and have appropriate permissions.
- **Debugging:** In production environment (`app.debug = false`), `debug` fields and sensitive details do not appear in error messages. In development environment, additional information appears to aid debugging.

---

## Examples Using cURL

### Query Yemen Mobile Balance

```bash
curl -X GET "http://localhost:8006/api/v1/telecomrecharge/nflow/yemen-mobile/balance?mobileNumber=777777777" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Recharge Balance

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/balance/charge" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "networkNumber": 1,
    "mobileNumber": "777777777",
    "amount": 100,
    "transactionId": "TXN_123"
  }'
```

### Recharge Game

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/game-charge" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "linkCode": "pubg_60",
    "fields": {"PlayerID": "123456789"},
    "quantity": 1,
    "transactionId": "TXN_456"
  }'
```

### Query Transaction Status

```bash
curl -X GET "http://localhost:8006/api/v1/telecomrecharge/nflow/operation-status?transactionId=TXN_123" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

**Note:** This documentation is based on version v1 of the API. Any future updates will be announced in the change log.