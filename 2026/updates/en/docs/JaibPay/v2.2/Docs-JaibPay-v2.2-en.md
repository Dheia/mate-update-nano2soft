# JaibPay Payment Gateway Documentation (Jeib Wallet) – Developer Guide

## 1. Overview

**JaibPay** is an electronic payment gateway integrated within the `Nano.Yepayment` package in Nano2Soft applications. The gateway relies on the **Jaib Wallet API** and uses a **Direct Payment** mechanism in a single step:

1. **Execute an online purchase via code** (Execute Buy Online By Code) – The customer enters a purchase code that was previously created in the Jeib app, and the amount is deducted from their balance immediately without needing additional confirmation or an OTP code.

After recent updates, the gateway also supports:
- **Refund** – via `POST /api/v1/BuyOnline/RefoundBuy`
- **Port testing** – `testPortConnection` function to diagnose connectivity issues
- **Unified response processing** – `normalizeApiResponse` function to provide clear error messages
- **Secure retrieval of encrypted settings** – `getSettingsByKey` function to securely fetch the stored password

This class can be used directly through `Nano\Yepayment\PaymentTypes\JaibPay` or via the unified payment system `Nano\MicroCart\Classes\Payments\PaymentGateway`.

---

## 2. Runtime Requirements and Settings

### 2.1. Basic Requirements

- Nano2Soft system version 2.0+
- Required plugins:
  - `Nano.MicroCart` (>=2.0)
  - `Nano.Yepayment` (>=1.2)
  - `Nano.Helpers`
- Jaib API credentials (provided by the gateway operator):
  - Username (`userName`)
  - Password (`password`)
  - Agent Code (`agentCode` – default value `10004`)
  - Base API URL (example: `https://www.api2.e-jaib.com:5088`)

### 2.2. Gateway Settings in the Control Panel

When activating the payment method **"Jaib Pay (Jeib Wallet)"**, the following fields appear (stored in the `nano_microcart_payment_gateway_settings` table):

| Field | Key | Description |
|-------|---------|-------|
| Base API URL | `jaibpay_url` | The Jaib API address (e.g., `https://www.api2.e-jaib.com:5088`) |
| Test API URL | `jaibpay_test_url` | (Optional) Test environment address |
| Username | `jaibpay_username` | Merchant username |
| Password | `jaibpay_password` | Merchant password (stored encrypted) |
| Agent Code (agentCode) | `jaibpay_agentcode` | Agent code provided by the Jeib gateway (default `10004`) |
| Default Currency | `jaibpay_default_currency` | Default value (YER, USD, SAR) |

These settings can be accessed within the class via:
```php
$url = PaymentGatewaySettings::get('jaibpay_url', '');
$username = PaymentGatewaySettings::get('jaibpay_username', '');
$agentCode = PaymentGatewaySettings::get('jaibpay_agentcode', '10004');
```

> **Important:** For encrypted settings such as `jaibpay_password`, you must use the `getSettingsByKey()` function instead of `PaymentGatewaySettings::get()` directly to ensure the decrypted value is correctly retrieved:
> ```php
> $password = $this->getSettingsByKey('jaibpay_password');
> ```

---

## 3. JaibPay Class – Core Methods

### 3.1. Class Definition

```php
namespace Nano\Yepayment\PaymentTypes;

use Nano\MicroCart\Classes\Payments\PaymentProvider;
use Nano\MicroCart\Classes\Payments\PaymentResult;
use Nano\MicroCart\Models\PaymentGatewaySettings;

class JaibPay extends PaymentProvider
{
    // ...
}
```

### 3.2. Core Properties

| Property | Type | Description |
|----------|------|-------|
| `$order` | `Order` | The order object associated with the payment |
| `$data` | `array` | User-supplied data (purchase code, mobile, amount, etc.) |
| `$success_url` | `string` | (Not used in JaibPay because payment is direct without redirect) |
| `$cancel_url` | `string` | (Not used) |

### 3.3. Main Methods

#### `public function identifier(): string`
Returns the unique identifier for the payment method (`jaibpay`).

#### `public function name(): string`
Returns the display name (`Jaib Pay (Jeib Wallet)`).

#### `public function process(PaymentResult $result): PaymentResult`
Creates a new payment (executes direct payment) via the API.

**Expected inputs in `$this->data` (from the payment form):**
- `purchase_code` – Purchase code (required)
- `mobile` – Customer mobile number (required)
- `amount` – Amount (required)
- `currency` – Currency (optional, uses default if not provided)
- `notes` – Note (optional)

**Actions:**
1. Validates data via `defineValidationRules()`.
2. Obtains `accessToken` and `pinApi` via `getAuthToken()` (from Cache or API).
3. Generates a unique `requestID` (UUID).
4. Sends a POST request to `/api/v1/BuyOnline/ExeBuy` with the data.
5. If successful, saves `requestID` to `order->payment_first_trans_id` and `referenceID` to `order->payment_trans_id`.
6. Saves additional data in `order->other_data['jaibpay']`.
7. Updates order status to `PaidState` via `$result->success()`.
8. Returns a successful `PaymentResult`.
9. **On failure:** sets `$result->message` from the error message coming from Jaib (e.g., "Invalid purchase code"), then calls `$result->fail()`.

#### `public function complete(PaymentResult $result): PaymentResult`
Not used in this type (instant payment). Left empty or could be used for refunds in the future.

#### `public function getAuthToken(): ?array`
Requests `accessToken` and `pinApi` from the Jaib API.

**Endpoint:** `POST /api/v1/TokenAuth/LogAPI`  
**Data:** `{"userName": "...", "password": "...", "agentCode": "..."}`  
**Return:** An array containing `accessToken`, `pinApi`, `expire` or `null` on failure.  
**Caching:** Stored in Cache for 86000 seconds.  
**Improvements:** Uses `normalizeApiResponse` to provide clear error messages, and uses `getSettingsByKey` to fetch the encrypted password.

#### `private function executeBuy(string $accessToken, string $pinApi, string $requestID): array`
Executes the purchase via code.

**Endpoint:** `POST /api/v1/BuyOnline/ExeBuy`  
**Data:** `{"pinApi": "...", "mobile": "...", "requestID": "...", "code": "...", "amount": ..., "currencyCode": "...", "notes": "..."}`  
**Return:** An array containing `success`, `referenceID`, `requestID`, `msg`, `amount`, `currencyCode`.  
**Improvements:** Uses `normalizeApiResponse` to provide precise error messages such as "Invalid purchase code" or "The code has already been used".

#### `public function checkTransactionStatus(string $requestID): array`
Queries a transaction status using `requestID`.

**Endpoint:** `POST /api/v1/BuyOnline/CheckProgress`  
**Data:** `{"pinApi": "...", "requestID": "..."}`  
**Return:** An array containing `success`, `request_id`, `reference_id`, `raw_response`.

#### `public function refund(string $referenceID, ?string $requestID = null, ?float $amount = null, ?string $currencyCode = null, ?string $notes = null): array`
**🆕 New function** – Refunds a previous transaction amount.

**Endpoint:** `POST /api/v1/BuyOnline/RefoundBuy`  
**Parameters:**
- `$referenceID` – The primary reference number (required)
- `$requestID` – A new unique request ID (optional, auto-generated)
- `$amount` – Amount to refund (optional, fetched from order)
- `$currencyCode` – Currency code (optional)
- `$notes` – Notes (optional)

**Impact on order:**
- On success: `order->payment_state` is changed to `RefundedState`.
- Refund data is stored in `order->other_data['jaibpay_refund']` as an array containing:
  ```php
  [
      'refund_referenceID'   => 'Refund reference number from Jaib',
      'original_referenceID' => 'Original transaction number',
      'requestID'            => 'Unique request ID',
      'amount'               => 5000.0,
      'currencyCode'         => 'YER',
      'notes'                => 'Refund notes',
      'msg'                  => 'Transaction completed successfully',
      'created_at'           => '2025-01-01 12:00:00'
  ];
  ```

#### `private function getApiUrl(string $type): string`
Builds API URLs based on the type (`login`, `buy`, `refund`, `check`).

#### `private function parseResponse($response): array`
Converts a Guzzle response to a PHP array.

---

## 4. New Helper Functions

### 4.1. `normalizeApiResponse` – Unified API Response Processing

```php
private function normalizeApiResponse($response): array
```

**Functionality:** Receives an HTTP response (or exception) and uniformly extracts all important information:

| Key | Description |
|---------|-------|
| `success` | Operation success (boolean) |
| `status_code` | HTTP code (e.g., 200, 500) |
| `error_code` | Error code from Jaib (e.g., `51`, `-100`, `-1026`) |
| `error_message` | Textual error message from Jaib |
| `data` | Result data (full content of `result`) |
| `raw` | Full raw response |

**Usage:** Used in all API functions (`getAuthToken`, `executeBuy`, `checkTransactionStatus`, `refund`) to standardize response handling and provide accurate error messages to the user.

### 4.2. `looksLikeJson` – Check JSON Shape

```php
private function looksLikeJson($string): bool
```

Helper function that checks if a given string looks like JSON (starts with `{` or `[`). Used internally in `normalizeApiResponse`.

### 4.3. `testPortConnection` – Port Testing

```php
public static function testPortConnection(string $url, ?int $port = null, int $timeout = 5): array
```

**Functionality:** Tests whether a specific port is open on a server. Supports:
- Automatic port extraction from URL (e.g., `https://www.api2.e-jaib.com:5088`)
- Prioritizing the passed parameter port over the port in the URL
- Using the default port according to the scheme (443 for HTTPS, 80 for HTTP)
- DNS check and determining whether the problem is with DNS or the port
- Measuring response time in milliseconds

**Example call:**
```php
$result = JaibPay::testPortConnection('https://www.api2.e-jaib.com:5088');
if ($result['success']) {
    echo "Port open, response time: " . $result['details']['elapsed_ms'] . "ms";
} else {
    echo "Connection failed: " . $result['message'];
}
```

### 4.4. `getSettingsByKey` – Secure Settings Retrieval

```php
public function getSettingsByKey($key, $defaultValue = null)
```

**Functionality:** Retrieves a setting value from `PaymentGatewaySettings` correctly, with support for encrypted fields:
- If the key is within `encryptedSettings()`, it uses `getEncryptableValue()` to decrypt.
- Otherwise, uses regular `get()`.

**Primary use:** Fetching the encrypted password (`jaibpay_password`).

---

## 5. Payment Mechanism Step-by-Step (for Developers)

### 5.1. Full Process Flow

1. **Authentication:** `getAuthToken()` → `POST /api/v1/TokenAuth/LogAPI` ← `{ accessToken, pinApi, expire }`
2. **Execute Payment:** `executeBuy()` → `POST /api/v1/BuyOnline/ExeBuy` ← `{ referenceID, requestID, msg }`
3. **Save Data:** Store `requestID` in `order->payment_first_trans_id` and `referenceID` in `order->payment_trans_id`, update order to `PaidState`
4. **Query (Optional):** `checkTransactionStatus()` → `POST /api/v1/BuyOnline/CheckProgress`
5. **Refund (New):** `refund()` → `POST /api/v1/BuyOnline/RefoundBuy` ← Change order status to `RefundedState`


### 5.2. Complete Process Flow Diagram

<img src="./images/sequenceDiagram-JaibPay-en.jpg" alt="JaibPay Flow Diagram" width="600">


### 5.3. Integrating the Gateway in a Custom API

#### A. Execute a New Payment (Initiate Payment)

**Custom endpoint in `routes/api.php`:**

```php
Route::post('/payment/jaibpay/create', function (Request $request) {
    $order = Order::find($request->order_id);
    $jaib = new JaibPay($order, [
        'purchase_code' => $request->purchase_code,
        'mobile'        => $request->mobile,
        'amount'        => $request->amount,
        'currency'      => $request->currency ?? 'YER',
        'notes'         => $request->notes,
    ]);
    $result = new PaymentResult($jaib, $order);
    $processResult = $jaib->process($result);
    return response()->json([
        'success'      => $processResult->successful,
        'request_id'   => $order->payment_first_trans_id,
        'reference_id' => $order->payment_trans_id,
        'message'      => $processResult->message,
    ]);
});
```

**Example request:**
```json
POST /api/payment/jaibpay/create
{
    "order_id": 200,
    "purchase_code": "3719",
    "mobile": "774760761",
    "amount": 5000,
    "currency": "YER",
    "notes": "Payment for order #200"
}
```

**Response:**
```json
{
    "success": true,
    "request_id": "550e8400-e29b-41d4-a716-446655440000",
    "reference_id": "16986110064345",
    "message": "Payment completed successfully"
}
```

#### B. Query Transaction Status

```php
Route::get('/payment/jaibpay/status', function (Request $request) {
    $jaib = new JaibPay();
    $status = $jaib->checkTransactionStatus($request->request_id);
    return response()->json($status);
});
```

**Request:**
```
GET /api/payment/jaibpay/status?request_id=550e8400-e29b-41d4-a716-446655440000
```

**Response:**
```json
{
    "success": true,
    "request_id": "550e8400-e29b-41d4-a716-446655440000",
    "reference_id": "16986110064345",
    "raw_response": { ... }
}
```

#### C. Refund (New)

```php
Route::post('/payment/jaibpay/refund', function (Request $request) {
    $order = Order::find($request->order_id);
    $jaib = new JaibPay($order, [
        'amount'   => $request->amount,
        'currency' => $request->currency ?? 'YER',
        'notes'    => $request->notes ?? 'Refund',
    ]);
    $refundResult = $jaib->refund($request->reference_id);
    return response()->json($refundResult);
});
```

---

## 6. Built-in Test Endpoints in `routes.php`

Within the `routes.php` file of `Nano.Yepayment`, a set of helper endpoints has been provided under the group `/api/v1/yepayment`, dedicated to developers and administrators for testing the gateway.

### 6.1. List of Endpoints

| Path | Method | Description |
|--------|---------|-------|
| `/jaibpay/test-auth` | POST | Test authentication with Jaib API (obtain accessToken and pinApi) |
| `/jaibpay/test-create-payment` | POST | Create a new payment (execute direct payment) |
| `/jaibpay/test-check-status` | GET | Query transaction status using `request_id` |
| `/jaibpay/test-full-payment` | POST | Comprehensive test (create payment + query) |
| `/jaibpay/test-refund` | POST | **🆕** Refund a transaction amount |
| `/jaibpay/test-port` | GET | **🆕** Test port (uses `JaibPay::testPortConnection`) |
| `/jaibpay/stats` | GET | Gateway usage statistics (order count, success rate) |
| `/jaibpay/test-ui` | GET | Interactive web interface to test all functions |

### 6.2. Description of New Endpoints

#### `POST /jaibpay/test-refund`
**Request data (JSON):**
```json
{
    "order_id": 200,
    "reference_id": "16986110064345",
    "amount": 5000,
    "currency": "YER",
    "notes": "Refund"
}
```
> If `reference_id` is not passed and `order_id` is passed, the system will try to fetch it automatically from the order data. It also verifies that the order is paid and that the payment was made via JaibPay.  
**Response:** `{ success, data: { reference_id, request_id, api_data } }`

#### `GET /jaibpay/test-port?url=...&port=...`
**Response:** `{ success, message, details: { host, port, elapsed_ms, dns_resolved, ... } }`

### 6.3. Description of Each Endpoint

#### `POST /jaibpay/test-auth`
No input data required (uses stored settings).  
**Response:** `{ success, data: { access_token_length, pin_api, expire } }`

#### `POST /jaibpay/test-create-payment`
**Request data (JSON):**
```json
{
    "order_id": 200,
    "purchase_code": "3719",
    "mobile": "774760761",
    "amount": 5000,
    "currency": "YER",
    "notes": "Test"
}
```
**Response:** `{ success, data: { request_id, reference_id, api_data }, order_data }`

#### `GET /jaibpay/test-check-status?request_id=...`
**Response:** `{ success, request_id, reference_id, raw_response }`

#### `POST /jaibpay/test-full-payment`
Performs two steps automatically (create payment + query).  
**Response:** Contains `results` (results of each step) and `summary`.

#### `GET /jaibpay/test-ui`
Displays a fully integrated HTML interface containing:
- Step-by-step manual testing (authentication, create payment, query, refund)
- Automatic testing with a selectable number of attempts (1-10 times)
- Instant statistics (order count, success rate, last records)
- Test logs stored in LocalStorage
- Additional tools (port test, export logs, reset)

#### `GET /jaibpay/stats`
**Response:** Statistics such as `total_orders`, `jaibpay_orders`, `successful_payments`, `success_rate`, gateway settings.

---

## 7. Handling the Gateway via External API (for Other Applications)

If you are developing an external application (e.g., a mobile app or standalone e-commerce store) and wish to integrate JaibPay without directly using the `JaibPay` class, you can connect to the **public endpoints** provided by the system (mentioned above) after authentication via `oauth-users`.

### 7.1. Prerequisite Authentication

You must have a valid OAuth 2.0 token (obtainable from the Nano2Soft system via the standard login endpoint). Then send the token in the header:
```
Authorization: Bearer <token>
```

### 7.2. Complete Example Using cURL

#### A. Create a New Payment
```bash
curl -X POST "https://yourdomain.com/api/v1/yepayment/jaibpay/test-create-payment" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 200,
    "purchase_code": "3719",
    "mobile": "774760761",
    "amount": 5000,
    "currency": "YER",
    "notes": "Product purchase"
  }'
```

#### B. Query Payment Status
```bash
curl -X GET "https://yourdomain.com/api/v1/yepayment/jaibpay/test-check-status?request_id=550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer <token>"
```

#### C. Refund
```bash
curl -X POST "https://yourdomain.com/api/v1/yepayment/jaibpay/test-refund" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 200,
    "reference_id": "16986110064345",
    "amount": 5000,
    "currency": "YER"
  }'
```

#### D. Port Test
```bash
curl -X GET "https://yourdomain.com/api/v1/yepayment/jaibpay/test-port?url=https://www.api2.e-jaib.com:5088" \
  -H "Authorization: Bearer <token>"
```

> **Note:** These endpoints are also protected by `BackendAuth` (require the user to be an administrator). If you want to make them available to regular customers, you must modify `routes.php` to remove the `BackendAuth` check or add custom middleware.

---

## 8. Common Error Codes and Solutions

| HTTP Code | Error (from Jaib API) | Cause and Solution |
|----------|----------------------|--------------|
| 401 | Unauthorized | Authentication failed – verify `userName`/`password`/`agentCode` in settings. |
| 400 | "Invalid code number" (code 51) | Purchase code is incorrect or expired – verify the entered code. |
| 400 | "The code has already been used" (code -1026) | The code was consumed previously – use a new code. |
| 400 | "Insufficient balance" | Customer's Jeib wallet balance does not cover the required amount. |
| 400 | "Transaction not found" | `requestID` is incorrect – verify the stored ID. |
| 400 | "Invalid transaction reference fee" | `referenceID` is incorrect during refund. |
| 500 | Internal Server Error | Problem connecting to Jaib API – verify the URL and port, or try again later. |
| cURL 28 | Connection timed out | Port 5088 is not open from the hosting side – use `testPortConnection` for diagnosis. |

---

## 9. Data Structure Stored in the Order

### 9.1. Payment Data – `order->other_data['jaibpay']`

```php
[
    'request_id'      => '550e8400-e29b-41d4-a716-446655440000',  // Unique request ID
    'reference_id'    => '16986110064345',      // Reference transaction number from Jaib
    'pin_api'         => 'tIcI4zm',             // PIN API code used
    'amount'          => 5000.0,                // Amount
    'currency_code'   => 'YER',                 // Currency code
    'mobile'          => '774760761',           // Customer mobile number
    'purchase_code'   => '3719',                // Purchase code used
    'created_at'      => '2025-01-01T12:00:00Z' // Transaction creation time
]
```

### 9.2. Refund Data – `order->other_data['jaibpay_refund']`

```php
[
    'refund_referenceID'   => '16986138584353',  // Refund reference number from Jaib
    'original_referenceID' => '16986110064345',  // Original transaction number
    'requestID'            => 'uuid-...',        // Refund request ID
    'amount'               => 5000.0,            // Refunded amount
    'currencyCode'         => 'YER',             // Currency code
    'notes'                => 'Refund amount',   // Notes
    'msg'                  => 'Transaction completed successfully', // Message from Jaib
    'created_at'           => '2025-01-01T12:30:00Z' // Refund time
]
```

---

## 10. Practical Examples of Using the Class in Custom Code

### 10.1. Create a New Payment Without Using `PaymentGateway`

```php
use Nano\Yepayment\PaymentTypes\JaibPay;
use Nano\Orders\Models\Order;

$order = Order::find(200);
$jaib = new JaibPay($order, [
    'purchase_code' => '3719',
    'mobile'        => '774760761',
    'amount'        => 5000,
    'currency'      => 'YER',
    'notes'         => 'Payment for order #200',
]);

$paymentResult = new \Nano\MicroCart\Classes\Payments\PaymentResult($jaib, $order);
$processResult = $jaib->process($paymentResult);

if ($processResult->successful) {
    $requestID = $order->payment_first_trans_id;
    $referenceID = $order->payment_trans_id;
    // Redirect user to success page
}
```

### 10.2. Query Payment Status

```php
$jaib = new JaibPay();
$status = $jaib->checkTransactionStatus('550e8400-e29b-41d4-a716-446655440000');
if ($status['success']) {
    echo "Reference ID: " . $status['reference_id'];
}
```

### 10.3. Refund a Transaction Amount

```php
$order = Order::find(200);
$jaib = new JaibPay($order, [
    'amount'   => 5000,
    'currency' => 'YER',
    'notes'    => 'Full refund',
]);
$refundResult = $jaib->refund('16986110064345');

if ($refundResult['success']) {
    // Refund successful
    // Order status automatically changed to RefundedState
    // Refund data stored in $order->other_data['jaibpay_refund']
}
```

### 10.4. Using the Authentication Function to Get Token Only

```php
$jaib = new JaibPay();
$auth = $jaib->getAuthToken(); // Returns array containing accessToken, pinApi
if ($auth) {
    echo "Token: " . $auth['accessToken'];
}
```

### 10.5. Port Testing

```php
$result = JaibPay::testPortConnection('https://www.api2.e-jaib.com:5088');
if ($result['success']) {
    echo "Port open, response time: " . $result['details']['elapsed_ms'] . "ms";
} else {
    echo "Connection failed: " . $result['message'];
    if ($result['details']['dns_resolved']) {
        echo " (DNS resolved successfully – problem is with the port itself)";
    }
}
```

---

## 11. Summary of Endpoints in `routes.php` (Quick Reference)

| Full Path | Method | Usage |
|---------------|---------|-----------|
| `/api/v1/yepayment/jaibpay/test-auth` | POST | Test login credentials |
| `/api/v1/yepayment/jaibpay/test-create-payment` | POST | Create a new payment |
| `/api/v1/yepayment/jaibpay/test-check-status` | GET | Query transaction status |
| `/api/v1/yepayment/jaibpay/test-full-payment` | POST | Comprehensive test (create + query) |
| `/api/v1/yepayment/jaibpay/test-refund` | POST | 🆕 Refund a transaction amount |
| `/api/v1/yepayment/jaibpay/test-port` | GET | 🆕 Port test |
| `/api/v1/yepayment/jaibpay/stats` | GET | Gateway statistics |
| `/api/v1/yepayment/jaibpay/test-ui` | GET | Web test interface |

> **Note:** All these endpoints require the current user to be an administrator (`BackendAuth`). To make them available to regular API customers, edit `routes.php` or add custom middleware.

---

## 12. References

- [JaibPay.php Class](./JaibPay.php) – Full gateway code.
- [routes.php File](./routes.php) – Definition of JaibPay endpoints.
- [JaibPay Updates Document (May 2026)](./Update-JaibPay-en.md) – Update log and improvements.
- [Nano2Soft Payment Gateway Development Guide](./SKILL.md)
- [Jaib Pay API Documentation – Login (Login.pdf)](./Login.pdf)
- [Jaib Pay API Documentation – Execute and Query Payment (Jaib Wallet Pay API.pdf)](./Jaib%20Wallet%20Pay%20API.pdf)
- [Postman Collection for Jaib Pay API Testing](./Jaib%20Pay%20API.postman_collection.json)

---

**This documentation has been prepared to help developers easily and effectively integrate and use the JaibPay (Jeib Wallet) payment gateway.**  
For inquiries or technical support, please contact us via the official website [nano2soft.com](https://nano2soft.com).
