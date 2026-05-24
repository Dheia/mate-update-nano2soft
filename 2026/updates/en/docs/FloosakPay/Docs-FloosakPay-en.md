# FloosakPay Payment Gateway Documentation (Floosak Wallet) – Developer Guide

## 1. Overview

**FloosakPay** is an electronic payment gateway integrated within the `Nano.Yepayment` package in NanoSoft applications, based on the **Floosak Wallet API** provided by Floosak. The gateway operates according to the **Two‑Step payment pattern with OTP confirmation**:

1. **`process()` – Create a pending transaction (Send):**  
   The merchant sends a request to Floosak to create a pending purchase. An OTP is sent to the customer’s mobile, and transaction data including the `purchase_id` is returned. At this stage **the order is not changed to “paid”**; only transaction data is stored and the customer is asked to enter the code.

2. **`complete()` – Confirm the payment (Confirm):**  
   After the customer enters the code, the system sends a confirmation request containing the `purchase_id` and `otp`. If the code is correct, the transaction becomes `Completed`, the order status is updated to `PaidState`, the cart is cleared, and associated events are triggered.

The gateway supports the currencies offered by Floosak (Yemeni Rial, US Dollar, Saudi Riyal). It also provides an **idempotency mechanism** via `request_id`, and logs a `PaymentLog` in the first stage to track incomplete attempts.

This gateway is built according to `Nano.Yepayment` standards and follows the same pattern used in `CashPay` and `YottaPay`.

---

## 2. Requirements and Settings

### 2.1. Basic Requirements

- NanoSoft system (Nano2Soft) version 2.0+
- Required plugins:
  - `Nano.MicroCart` (>=2.0)
  - `Nano.Yepayment` (>=1.2)
  - `Nano.Helpers`
- Floosak Wallet API credentials (provided by the service provider):
  - Merchant phone number (`phone`)
  - Password (`password`)
  - Source wallet ID (`source_wallet_id`)
  - Base API URL (production or test)

### 2.2. Gateway Settings in Control Panel

When activating the **"Floosak Pay (Floosak Wallet)"** payment method, the following fields appear on the payment settings page:

| Field | Key | Description | Default Value |
|-------|---------|-------|-------------------|
| Base API URL | `floosakpay_url` | Production API endpoint | `https://staging.fintech-expert.net` |
| Test API URL | `floosakpay_test_url` | Test environment API endpoint | `https://staging.fintech-expert.net` |
| Merchant Phone Number | `floosakpay_phone` | Mobile number used for authentication (with country code) | - |
| Password | `floosakpay_password` | Merchant account password (stored encrypted) | - |
| Source Wallet ID | `floosakpay_source_wallet_id` | ID of the wallet from which funds will be deducted | - |
| Default Purpose | `floosakpay_default_purpose` | Text sent as transaction description | `دفع الطلب` (Payment for order) |

> **Security note:** The password is stored encrypted via `encryptedSettings()`.

---

## 3. Payment Process Lifecycle

FloosakPay operates according to a **Two‑Step** pattern, consisting of two separate stages.

### 3.1. First Stage: Create a Pending Transaction (`process`)

When payment is initiated for a given order, `process()` is called and performs the following steps:

1. **Data validation** – checks that `target_phone`, `amount`, and `purpose` are present.
2. **Ensure the order is not already paid** – if `payment_state` equals `PaidState`, the process stops.
3. **Check for an existing transaction** – searches for a `request_id` stored in `payment_first_trans_id` or `other_data['floosakpay']['request_id']`. If found, queries its status via `checkTransactionStatus()`:
   - If status is `Completed` → calls `$result->success()` directly to update the order to `PaidState`.
   - If status is `Pending` → informs the user to wait for OTP input.
   - If status is unknown or failed → creates a new transaction.
4. **Obtain an Access Token** via `POST /api/v1/auth/login` – stored in Cache for 3500 seconds.
5. **Send the payment creation request** `POST /api/v1/merchant/p2mcl` with:
   - `source_wallet_id`
   - `request_id` (unique UUID)
   - `target_phone`
   - `amount`
   - `purpose`
6. **Receive `purchase_id` and `reference_id`** from the response.
7. **Save transaction data** in the order:
   - `order.payment_first_trans_id` = `request_id`
   - `order.payment_trans_id` = `purchase_id`
   - Store additional data in `order.other_data['floosakpay']` (phone number, amount, purpose, return URLs, etc.)
   - **The payment status is not changed to `PaidState` at this stage.**
8. **Log the payment attempt** in the `PaymentLog` table via `$result->logSuccessfulPayment()` with status `'initiated'` (even if payment is not yet completed).
9. **Return a successful result with a message "Please enter the verification code"** – without calling `$result->success()`.

### 3.2. Second Stage: Confirm the Payment (`complete`)

After the customer enters the received OTP, the application calls the `/floosakpay/success` endpoint (or directly `complete()`). The `complete()` method performs the following steps:

1. **Extract `purchase_id`** from `order.payment_trans_id`.
2. **Obtain an Access Token** again (or from cache).
3. **Send the confirmation request** `POST /api/v1/merchant/p2mcl/confirm` with:
   - `otp` (the code entered by the customer)
   - `purchase_id`
4. **Parse the response**:
   - If `is_success = true` and `status.en = "Completed"` → calls `$result->success()`, which updates the order to `PaidState`, triggers events, and clears the cart.
   - If `status.en = "Pending"` → returns a message indicating that the code is incorrect or the transaction is still pending.
5. **Return the final result** (success or failure).

### 3.3. Optional Callback URLs

FloosakPay supports storing `callback_success_url` and `callback_error_url` if they are passed within the payment data (e.g., from a mobile application). After payment confirmation, the user can be directed via the two routes:

- `GET /api/v1/yepayment/floosakpay/success` – retrieves the success callback URL and uses `RedirectHelper` to redirect to the application or website.
- `GET /api/v1/yepayment/floosakpay/cancel` – similar for cancellation.

This ensures full compatibility with mobile applications that require deep links after payment.

---

## 4. Data Structure Stored in `order->other_data['floosakpay']`

```php
[
    'request_id'         => '550e8400-e29b-41d4-a716-446655440000',   // Unique request ID (UUID)
    'purchase_id'        => 1699240,                                   // Purchase ID from Floosak
    'source_wallet_id'   => 10000,                                     // Source wallet ID
    'target_phone'       => '967771234567',                            // Customer mobile number
    'amount'             => 100.00,                                    // Amount
    'purpose'            => 'دفع الطلب #200',                          // Purpose
    'fee'                => 1.00,                                      // Service fee (if any)
    'gross'              => 101.00,                                    // Gross amount including fees
    'reference_id'       => '4633395571756870',                        // Reference from Floosak
    'requires_otp'       => true,
    'created_at'         => '2025-01-01T12:00:00Z',
    'callback_success_url' => 'myapp://pay/success',                   // (optional)
    'callback_error_url'   => 'myapp://pay/error'                      // (optional)
]
```

---

## 5. Integrated Scenario for Using FloosakPay in Your Application

This scenario illustrates the steps a developer or store owner should follow to handle FloosakPay via the `Checkout2` API.

### 5.1. Prerequisites
- A mobile application or website using `Nano.Yepayment` and `Nano.ShopApi`.
- FloosakPay gateway is configured in the control panel (with phone, password, source_wallet_id).
- The customer has a Floosak Wallet account and their number is registered.

### 5.2. Steps from the Developer's Perspective

#### **Step 1: The user selects FloosakPay as the payment method**
The application sends a request to the `checkout` endpoint with `step=pay` and the payment method identifier `floosakpay`.

**Example request:**
```http
POST /api/v1/shop/checkout
Content-Type: application/json

{
  "step": "pay",
  "payment_method_id": 5,
  "target_phone": "967771234567",
  "amount": 100,
  "purpose": "دفع الطلب",
  "callback_success_url": "myapp://checkout/success",
  "callback_error_url": "myapp://checkout/error"
}
```

#### **Step 2: The system processes the payment via `Checkout2` and `OrderManager`**
- The system calls `OrderManager->setStepPayments()`, which:
  1. Validates shipping addresses and coupons.
  2. Creates a `PaymentService` and `PaymentGateway` based on the selected payment method (FloosakPay).
  3. Calls `FloosakPay->process()`.

#### **Step 3: `FloosakPay->process()` creates the pending transaction**
- The gateway communicates with the Floosak API and creates a pending purchase, receiving a `purchase_id`.
- The system saves `request_id` and `purchase_id` in the order (without changing the payment status).
- The developer receives the following in the API response:

```json
{
  "status": true,
  "step_status": true,
  "redirect": false,
  "processed": false,
  "process_data": {
    "paymentResult": {
      "successful": true,
      "message": "Please enter the verification code sent to your phone",
      "api_data": {
        "purchase_id": 1699240,
        "request_id": "550e8400-..."
      }
    }
  }
}
```

- **Important here:** `processed` is still `false`, and the application displays an OTP input interface.

#### **Step 4: The customer enters the OTP and confirms**
The application sends a request to a dedicated endpoint (e.g., `POST /api/v1/shop/confirm-payment`) with `order_id` and `otp`.

**Example request:**
```json
{
  "order_id": 200,
  "otp": "123456"
}
```

#### **Step 5: The system calls `FloosakPay->complete()`**
- The system contacts the Floosak API to confirm the payment.
- If successful, `order.payment_state` is updated to `PaidState` and the cart is cleared.
- A success response is returned to the application.

#### **Step 6: Final redirection**
- The user is redirected to the payment success page (or to the deep link `myapp://checkout/success`) with order data.

---

## 6. FloosakPay Class – Core Methods

### 6.1. Class Definition

```php
namespace Nano\Yepayment\PaymentTypes;

use Nano\MicroCart\Classes\Payments\PaymentProvider;
use Nano\MicroCart\Classes\Payments\PaymentResult;
use Nano\MicroCart\Models\PaymentGatewaySettings;

class FloosakPay extends PaymentProvider
{
    // ...
}
```

### 6.2. Basic Properties

| Property | Type | Description |
|----------|------|-------|
| `$order` | `Order` | The order object associated with the payment |
| `$data` | `array` | Data received from the user (`target_phone`, `amount`, `purpose`, `otp`, etc.) |
| `$success_url` | `string` | Success path (`api/v1/yepayment/floosakpay/success`) |
| `$cancel_url` | `string` | Cancel path (`api/v1/yepayment/floosakpay/cancel`) |
| `$is_test_mod` | `bool` | Test mode (read from gateway settings) |

### 6.3. Main Methods

| Method | Description |
|--------|-------|
| `identifier(): string` | Returns `'floosakpay'` |
| `name(): string` | Returns `'Floosak Pay (Floosak Wallet)'` |
| `process(PaymentResult $result): PaymentResult` | Creates a pending transaction (Send) and returns a message asking for OTP. |
| `complete(PaymentResult $result): PaymentResult` | Confirms the payment using OTP and updates the order status to paid. |
| `reverse(string $refNo, string $reason = ''): array` | Refunds a completed transaction. |
| `checkTransactionStatus(string $requestId): array` | Queries transaction status using `request_id`. |
| `getAuthToken(bool $useCache = true): ?string` | Obtains an authentication token (Bearer) from `/api/v1/auth/login`. |
| `validateSettings(): void` | Checks that essential settings are complete. |
| `isAvailable(): bool` | Checks credentials validity (attempts to obtain a token). |

### 6.4. Important Helper Methods

| Method | Description |
|--------|-------|
| `sendPayment(string $token): array` | Executes `POST /api/v1/merchant/p2mcl`. |
| `confirmPayment(string $token, int $purchaseId, string $otp): array` | Executes `POST /api/v1/merchant/p2mcl/confirm`. |
| `refundPayment(string $token, int $transactionId, string $requestId, float $amount, string $reason): array` | Executes `POST /api/v1/merchant/p2mcl/refund`. |
| `getApiUrl(string $type): string` | Builds the full API URL (login, send, confirm, refund, status). |
| `getHeaders(string $token): array` | Returns the headers array (`Authorization`, `x-channel: merchant`, etc.). |
| `handleSpecificErrors(array $responseData, string $defaultMessage): array` | Extracts error messages from Floosak response. |
| `getSettingsByKey(string $key, $default = null)` | Retrieves settings with support for encrypted fields. |

---

## 7. Built-in Test Endpoints in `routes.php`

Within the `routes.php` file of `Nano.Yepayment`, a set of helper endpoints is provided under the `/api/v1/yepayment` group, intended for developers and administrators to test the gateway.

### 7.1. List of Endpoints

| Path | Method | Description |
|--------|---------|-------|
| `/floosakpay/test-auth` | POST | Test authentication (obtain token) |
| `/floosakpay/test-create-payment` | POST | Create a pending transaction (first step) |
| `/floosakpay/test-confirm-payment` | POST | Confirm payment using OTP (second step) |
| `/floosakpay/test-check-status` | GET | Query transaction status (by `request_id` or `order_id`) |
| `/floosakpay/test-refund` | POST | Refund a completed transaction |
| `/floosakpay/test-full-payment` | POST | Full test (create + confirm + query) |
| `/floosakpay/stats` | GET | Gateway usage statistics |
| `/floosakpay/logs` | GET | Transaction logs from the database |
| `/floosakpay/test-ui` | GET | Interactive web UI to test all functions |
| `/floosakpay/success` | GET | Success path (for returning from the app) |
| `/floosakpay/cancel` | GET | Cancel path |

### 7.2. Explanation of Key Endpoints

#### `POST /floosakpay/test-auth`
Does not require input data (uses stored settings).  
**Response:**
```json
{
  "success": true,
  "message": "Token obtained successfully",
  "token_length": 123
}
```

#### `POST /floosakpay/test-create-payment`
**Request data (JSON):**
```json
{
  "order_id": 200,
  "target_phone": "967771234567",
  "amount": 100,
  "purpose": "دفع الطلب"
}
```
**Response:**
```json
{
  "success": true,
  "message": "Please enter the verification code sent to your phone",
  "requires_otp": true,
  "purchase_id": 1699240,
  "api_data": { ... }
}
```

#### `POST /floosakpay/test-confirm-payment`
**Request data:**
```json
{
  "order_id": 200,
  "otp": "123456"
}
```
**Response:**
```json
{
  "success": true,
  "message": "Payment confirmed successfully",
  "api_data": { ... }
}
```

#### `GET /floosakpay/test-check-status?request_id=...` or `?order_id=...`
**Response:**
```json
{
  "success": true,
  "status": "Completed",
  "completed": true,
  "raw_response": { ... }
}
```

#### `POST /floosakpay/test-refund`
**Request data:**
```json
{
  "order_id": 200,
  "reason": "Refund requested by customer"
}
```
**Response:**
```json
{
  "success": true,
  "message": "Refund processed successfully",
  "refund_id": 620493
}
```

#### `POST /floosakpay/test-full-payment`
Performs both steps automatically (create + confirm) with the ability to pass an OTP (if not provided, uses `123456` by default).  
**Response:** Contains `results` and `summary`.

#### `GET /floosakpay/stats`
**Response:**
```json
{
  "success": true,
  "stats": {
    "total_orders": 1500,
    "floosakpay_orders": 120,
    "successful_payments": 115,
    "pending_payments": 3,
    "failed_payments": 2,
    "success_rate": 95.83,
    "recent_logs": [ ... ]
  },
  "settings": {
    "url": "https://staging.fintech-expert.net",
    "phone": "9677XXXXXXX",
    "source_wallet_id": "10000"
  }
}
```

#### `GET /floosakpay/test-ui`
Displays a complete HTML interface containing:
- Manual testing (create, confirm, query, refund).
- Automated testing with configurable number of attempts.
- Real‑time statistics.
- Test logs stored in LocalStorage.
- Additional tools (export logs, clear logs).

---

## 8. Practical Usage Examples in Custom Code

### 8.1. Create a New Payment (First Step)

```php
use Nano\Yepayment\PaymentTypes\FloosakPay;
use Nano\Orders\Models\Order;

$order = Order::find(200);
$floosak = new FloosakPay($order, [
    'target_phone' => '967771234567',
    'amount'       => 100,
    'purpose'      => 'دفع الطلب #200',
]);

$paymentResult = new \Nano\MicroCart\Classes\Payments\PaymentResult($floosak, $order);
$processResult = $floosak->process($paymentResult);

if ($processResult->successful) {
    $purchaseId = $order->payment_trans_id;
    // Show OTP input interface to the user
} else {
    // Display error message
}
```

### 8.2. Confirm Payment (Second Step)

```php
$floosak = new FloosakPay($order, ['otp' => '123456']);
$result = new PaymentResult($floosak, $order);
$completeResult = $floosak->complete($result);

if ($completeResult->successful) {
    // Payment completed – update UI
} else {
    // Confirmation failed – show error message
}
```

### 8.3. Query Transaction Status

```php
$floosak = new FloosakPay();
$status = $floosak->checkTransactionStatus('550e8400-...');
if ($status['success'] && $status['completed']) {
    echo "Transaction completed";
}
```

### 8.4. Refund an Amount

```php
$order = Order::find(200);
$floosak = new FloosakPay($order);
$refundResult = $floosak->reverse($order->payment_first_trans_id, 'Refund');

if ($refundResult['success']) {
    // Refund successful, order status became RefundedState
}
```

---

## 9. Common Error Codes and Solutions

| HTTP Code | Error (from Floosak API) | Cause and Solution |
|-----------|--------------------------|---------------------|
| 401 | Unauthorized | Authentication failed – check `phone` and `password` in settings. |
| 400 | `"target_phone must be a valid phone number"` | Invalid phone number – ensure it includes the country code (e.g., `9677...`). |
| 400 | `"source_wallet_id not found"` | Incorrect source wallet ID – verify the settings. |
| 400 | `"insufficient balance"` | Merchant’s wallet balance is insufficient to cover amount + fees. |
| 400 | `"OTP is incorrect"` | Wrong verification code – ask the customer to retry. |
| 400 | `"transaction already completed"` | Transaction already confirmed – no need to reconfirm. |
| 404 | `"purchase not found"` | `purchase_id` does not exist – verify that `payment_trans_id` is correct. |
| 500 | Internal Server Error | Floosak server issue – try again later or contact support. |

---

## 10. Troubleshooting

| Problem | Likely Cause | Solution |
|---------|---------------|------|
| Authentication failed (`auth_failed`) | Incorrect `phone` or `password` | Verify the settings entered in the control panel. |
| `target_phone` not accepted | Number does not start with `967` or missing country code | Ensure the number is in correct international format (e.g., `967771234567`). |
| `purchase_id` not found in the order | `payment_trans_id` was not saved during `process()` | Check that `sendPayment` succeeded and returned `data.id`. |
| Order does not change to paid after confirmation | `complete()` was not called or OTP is incorrect | Ensure the `/floosakpay/success` endpoint is called with `order_id` and `otp`. |
| Duplicate `request_id` | Same `request_id` sent for a pending transaction | Use a new UUID for each creation. |
| `cURL error 60` (SSL) | SSL certificate not trusted in test environment | Add `'verify' => false` in `HttpHelper` (only for testing). |
| `PaymentLog` not logged | Exception during log saving | Check the logs: `trace_log('FloosakPay: failed to log...')`. |

---

## 11. Integrating the Gateway into an External API (for Other Applications)

If you are developing an external application (e.g., a mobile app) and want to integrate FloosakPay without using the `FloosakPay` class directly, you can call the **public endpoints** mentioned above after authenticating via `oauth-users`.

### 11.1. Pre‑authentication

You must have a valid OAuth 2.0 token (obtained from the NanoSoft system via the usual login endpoint). Then send the token in the header:
```
Authorization: Bearer <token>
```

### 11.2. Complete Example Using cURL

#### a. Create a pending transaction
```bash
curl -X POST "https://yourdomain.com/api/v1/yepayment/floosakpay/test-create-payment" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 200,
    "target_phone": "967771234567",
    "amount": 100,
    "purpose": "دفع الطلب"
  }'
```

#### b. Confirm payment
```bash
curl -X POST "https://yourdomain.com/api/v1/yepayment/floosakpay/test-confirm-payment" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 200,
    "otp": "123456"
  }'
```

#### c. Query status
```bash
curl -X GET "https://yourdomain.com/api/v1/yepayment/floosakpay/test-check-status?order_id=200" \
  -H "Authorization: Bearer <token>"
```

#### d. Refund amount
```bash
curl -X POST "https://yourdomain.com/api/v1/yepayment/floosakpay/test-refund" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"order_id": 200, "reason": "Refund"}'
```

> **Note:** These endpoints are protected by `BackendAuth` (require the user to be an administrator). If you wish to make them available to regular customers, you must modify `routes.php` to remove the `BackendAuth` check or add a custom middleware.

---

## 12. Endpoint Summary (Quick Reference)

| Full Path | Method | Usage |
|-----------|--------|-------|
| `/api/v1/yepayment/floosakpay/test-auth` | POST | Test authentication |
| `/api/v1/yepayment/floosakpay/test-create-payment` | POST | Create a pending transaction (Send) |
| `/api/v1/yepayment/floosakpay/test-confirm-payment` | POST | Confirm payment (Confirm) |
| `/api/v1/yepayment/floosakpay/test-check-status` | GET | Query transaction status |
| `/api/v1/yepayment/floosakpay/test-refund` | POST | Refund amount |
| `/api/v1/yepayment/floosakpay/test-full-payment` | POST | Full test (create + confirm) |
| `/api/v1/yepayment/floosakpay/stats` | GET | Gateway statistics |
| `/api/v1/yepayment/floosakpay/logs` | GET | Transaction logs |
| `/api/v1/yepayment/floosakpay/test-ui` | GET | Web testing UI |
| `/api/v1/yepayment/floosakpay/success` | GET | Success return path |
| `/api/v1/yepayment/floosakpay/cancel` | GET | Cancel path |

> **Note:** All these endpoints require the current user to be an administrator (`BackendAuth`). To make them available to regular API clients, modify `routes.php` or add a custom middleware.

---

## 13. References

- [FloosakPay.php Class](./FloosakPay.php) – Full gateway code.
- [routes.php (FloosakPay section)](./routes.php) – Definition of FloosakPay endpoints.
- [Official API Documentation (PDF)](./Merchant_Wallet_Integration_API_Documentation_RefStyle.pdf)
- [Postman Collection for FloosakPay Testing](./Marchant%20Last%20Version.json)
- [NanoSoft Payment Gateway Development Guide (SKILL-v2.5)](./SKILL-v2.5.md)
- [Knowledge Summary (SKILL-Summary)](./SKILL-Summary.md)

---

**This documentation is prepared to help developers easily integrate and use the FloosakPay (Floosak Wallet) gateway.**  
For inquiries or technical support, please contact via the official website [nano2soft.com](https://nano2soft.com).
