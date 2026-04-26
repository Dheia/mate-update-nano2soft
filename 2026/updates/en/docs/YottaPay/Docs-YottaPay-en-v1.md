# YottaPay (Sabacash) Payment Gateway Documentation – NanoSoft Version

## 1. Overview

**YottaPay** (operating under the trade name **Sabacash**) is an integrated electronic payment gateway that enables accepting online payments using the **YottaGate** API. This gateway is built specifically for **Nano2Soft** applications and works as an add-on within the `Nano.Yepayment` package. The gateway relies on a two-step mechanism to ensure transaction security:

1. **Create Payment Transaction** (Create Adjustment) – Customer data, amount, and terminal are sent.
2. **Confirm Transaction** (Confirm Adjustment) – Using an OTP code sent to the customer.

The gateway also supports **money return** (refund) operations and transaction status checking via the unique Transaction ID. Authentication uses **OAuth 2.0** (bearer token obtained by logging into the YottaGate system).

---

## 2. Installation and Operation Requirements

- **Nano2Soft App** version 2 or higher.
- Required add-ons:
  - `Nano.MicroCart` (>= 2.0) for cart and order management.
  - `Nano.Yepayment` (>= 1.2) – includes the YottaPay gateway.
  - `Nano.Helpers` for HTTP service.
- **YottaGate** login credentials (provided by the operator):
  - `username`
  - `password`
  - `terminal` (default terminal number, e.g., "1")
  - Base API URL (usually `https://api.sabacash.com:49901`)

---

## 3. Payment Gateway Settings in Control Panel

When activating the **YottaPay (Sabacash)** payment method in the store control panel, the following settings appear:

| Field | Description | Example |
|-------|-------------|---------|
| **Base API URL** | YottaGate API address | `https://api.sabacash.com:49901` |
| **Username** | Merchant username in the system | `712988875` |
| **Password** | Merchant password | (stored encrypted) |
| **Default Terminal Number** | Terminal number to be used in transactions | `1` |
| **Default Currency ID** | Currency ID (1 = Yemeni Rial, 2 = Saudi Riyal, etc.) | `1` |

> **Note:** The password is stored encrypted in the database. Developers can modify `encryptedSettings()` in the class to add any other sensitive fields.

---

## 4. Payment Flow

### 4.0 Payment Workflow

<img src="./images/sequenceDiagram-YottaGate-en.jpg" alt="YottaGate Flow Diagram" width="600">

### 4.1. Create New Payment Transaction

When `process()` is called in the `YottaPay` class at the start of payment, the steps are:

1. **Validate inputs** (phone number, amount, terminal) – via `checkValidate()`.
2. **Request OAuth 2.0 token** via `getAuthToken()` (uses stored login credentials).
3. **Send `POST /api/accounts/v1/adjustment/onLinePayment` request** to YottaGate with the following data:
   ```json
   {
     "source": {
       "code": "771234567",
       "currencyId": "1"
     },
     "beneficiary": {
       "terminal": "1",
       "currencyId": "1"
     },
     "amount": "1000",
     "amountCurrencyId": "1",
     "note": "Payment for order #200"
   }
   ```
4. **Store data**:
   - Save `adjustment.id` in `order->payment_first_trans_id`.
   - Save `transactionId` (e.g., `DF-01-...`) in `order->payment_trans_id` (if provided).
   - Store additional data in `order->other_data['yottapay']`.
5. **Log payment attempt** in the `nano_microcart_payment_logs` table (status: unconfirmed).
6. Return `PaymentResult` with `successful = true` and a message stating that OTP input is required, without redirection (because payment is done directly via API).

### 4.2. Confirm Payment Using OTP

`complete()` is called after the customer receives the OTP code (sent via SMS by YottaGate). Steps:

1. Retrieve `adjustmentId` from `order->payment_first_trans_id`.
2. Retrieve `otp` from the request data (e.g., `Request::input('otp')`).
3. Send `PATCH /api/accounts/v1/adjustment/onLinePayment` request to YottaGate:
   ```json
   {
     "id": "816613",
     "otp": "4320",
     "note": "Confirm payment"
   }
   ```
4. Check for success (`completed: true`).
5. Update `order->payment_state` to `PaidState`.
6. Save `transactionId` (from response) in `order->payment_trans_id` if not already present.
7. Log the payment as **successful** via `$result->success()`.

### 4.3. Money Return (Refund)

The gateway supports refund operations via a separate API. It is primarily used by the system or merchant:

**Create refund request:**
```http
POST /api/accounts/v1/adjustment/onlineMoneyReturn
{
  "id": "816613",   // original adjustment ID
  "source": { "currencyId": "1", "isTerminal": true },
  "beneficiary": { "code": "711592626", "currencyId": "1" },
  "amount": "100",
  "amountCurrencyId": "1",
  "transactionId": "5697302956",
  "note": "Money return"
}
```

**Confirm refund (Patch):**
```http
PATCH /api/accounts/v1/adjustment/onlineMoneyReturn
{
  "id": 816614,
  "note": "Confirm return"
}
```

> **Note:** If the terminal balance is insufficient, set `isTerminal: false` to refund from the merchant's main balance.

### 4.4. Check Transaction Status

`checkTransactionStatus($transactionId)` can be called at any time to send a GET request to:
```
GET /api/accounts/v1/adjustment/checkAdjustmentByTransactionId?transactionId=DF-01-...
```
The response returns a `statusCode` which can be:
- `"completed"` – successful and completed transaction.
- `"not-completed"` – pending (not yet confirmed).
- `"not-exist"` – does not exist.

---

## 5. Data Structure Stored in `order->other_data['yottapay']`

```php
[
    'adjustment_id'      => '816613',            // id from create transaction response
    'transaction_id'     => 'DF-01-...',         // final transaction ID
    'source_phone'       => '771234567',
    'terminal'           => '1',
    'amount'             => 1000,
    'currency_id'        => '1',
    'note'               => 'Payment for order #200',
    'requires_otp'       => true,
    'created_at'         => '2025-01-01T12:00:00Z',
    'callback_success_url' => '...',            // optional
    'callback_error_url'   => '...',
]
```

---

## 6. API Test UI

The gateway provides an integrated web interface for testing all functions, accessible at:

```
/api/v1/yepayment/yottapay/test-ui
```

### 6.1. Features

- **Authentication testing** – Obtain API token.
- **Create test transaction** – With phone number, amount, terminal input.
- **Confirm transaction** – Using a test OTP (e.g., `1234`).
- **Check status** – Via transaction ID.
- **Full test** – Automatically executes all three steps and displays results.
- **Iterative test** – To measure gateway stability (up to 10 times).
- **Instant statistics** – Request count, success rate, latest logs.
- **Test logs** – LocalStorage storage with export and clear capabilities.

### 6.2. Test API Endpoints

| Path | Method | Description |
|------|--------|-------------|
| `/yottapay/test-auth` | POST | Verify login credentials and get token (test). |
| `/yottapay/test-create-payment` | POST | Create a test payment transaction (mimics `process`). |
| `/yottapay/test-confirm-payment` | POST | Confirm payment using OTP. |
| `/yottapay/test-check-status` | GET | Query transaction status. |
| `/yottapay/test-full-payment` | POST | Full test of all steps. |
| `/yottapay/test-change-password` | POST | Test password change (for the account). |
| `/yottapay/stats` | GET | Gateway usage statistics. |

> **Note:** All these endpoints are protected by `BackendAuth` (require admin login) and are not used in production.

---

## 7. Practical Request and Response Examples

### 7.1. Successful Create Payment Transaction

**Request (from inside `YottaPay::process`):**
```http
POST https://api.sabacash.com:49901/api/accounts/v1/adjustment/onLinePayment
Authorization: Bearer <token>
Content-Type: application/json

{
  "source": { "code": "771234567", "currencyId": "1" },
  "beneficiary": { "terminal": "1", "currencyId": "1" },
  "amount": "1000",
  "amountCurrencyId": "1",
  "note": "Order #200"
}
```

**Response:**
```json
{
  "destination": {
    "code": "771234567",
    "exchangePrice": 1,
    "isRegistered": true,
    "viral": false,
    "currencyId": "1"
  },
  "adjustment": {
    "amount": 1000,
    "paymentTypeId": 0,
    "sourceFeeAmount": 0,
    "thirdPartyFeeAmount": 0,
    "id": 816613,
    "makerChecker": false,
    "destinationFeeAmount": 0,
    "adjustmentTypeId": 620,
    "transactionId": "DF-01-0123456-0123456789"
  },
  "source": {
    "code": "771234567",
    "exchangePrice": 1,
    "isRegistered": true,
    "viral": false,
    "currencyId": "1"
  }
}
```

### 7.2. Confirm Payment Using OTP

**Request:**
```http
PATCH https://api.sabacash.com:49901/api/accounts/v1/adjustment/onLinePayment
Authorization: Bearer <token>
Content-Type: application/json

{
  "id": "816613",
  "otp": "4320",
  "note": "Confirm payment for order #200"
}
```

**Success Response:**
```json
{
  "id": 816613,
  "completed": true,
  "transactionId": "DF-01-0123456-0123456789"
}
```

### 7.3. Check Transaction Status

**Request:**
```http
GET https://api.sabacash.com:49901/api/accounts/v1/adjustment/checkAdjustmentByTransactionId?transactionId=DF-01-0123456-0123456789
Authorization: Bearer <token>
```

**Success Response:**
```json
{
  "amount": "1000.000000",
  "transactionDate": "1655714348044",
  "transactionId": "DF-01-0123456-0123456789",
  "statusCode": "completed"
}
```

---

## 8. Developer Integration Guide

### 8.1. Register the Payment Method in `Plugin.php`

```php
public function registerPaymentProviders()
{
    $providers = [];
    if (class_exists(\Nano\Yepayment\PaymentTypes\YottaPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\YottaPay();
    }
    return $providers;
}
```

### 8.2. Use the Gateway via `PaymentGateway`

```php
use Nano\MicroCart\Classes\Payments\DefaultPaymentGateway;
use Nano\Orders\Models\Order;

$order = Order::find($orderId);
$gateway = new DefaultPaymentGateway();
$gateway->init($paymentMethod, [
    'source_phone' => '771234567',
    'amount'       => 1000,
    // ... other data
]);
$result = $gateway->process($order);

if ($result->successful && $result->redirect) {
    // In YottaPay there is no redirect, but we display a message asking for OTP
    return view('payment.otp_form', ['adjustment_id' => $order->payment_first_trans_id]);
}
```

### 8.3. Confirm Payment After Customer Enters OTP

```php
$paymentProvider = new YottaPay();
$paymentProvider->setOrder($order);
$result = new PaymentResult($paymentProvider, $order);

// Set OTP from request
$paymentProvider->data['otp'] = Request::input('otp');
$completeResult = $paymentProvider->complete($result);

if ($completeResult->successful) {
    // Payment succeeded
}
```

### 8.4. Call Refund Programmatically

```php
$yottaPay = new YottaPay();
$refundResult = $yottaPay->refund($adjustmentId, $amount, $note, $isTerminal = true);
// $refundResult['success'] => true/false
```

> **Note:** The `refund` method is not present in the current class; it can be added following the API documentation for refunds.

---

## 9. Common Error Codes

| HTTP Code | Example Error Message | Meaning and Solution |
|-----------|-----------------------|----------------------|
| 401 | Unauthorized | Authentication failed – check username/password in settings. |
| 400 | entity of mobile [7xxxx] is not exists | Source phone number not registered in YottaGate. |
| 400 | Sender do not has sufficient balance | Source balance insufficient for the transaction. |
| 400 | can not do transaction, you Already did transaction for this Adjustment | Transaction already confirmed – cannot reconfirm. |
| 400 | Operation expired, this Operation Timed out and Closed | Transaction expired (usually after a time limit) – create a new transaction. |
| 400 | anonymous users can not perform any transaction | Request made without a valid token. |
| 404 | Entered id [xxx] does not exist in [adjustment] | Adjustment ID not found – check `payment_first_trans_id`. |

---

## 10. Troubleshooting

| Problem | Possible Cause | Solution |
|---------|----------------|----------|
| `process()` fails with "Invalid credentials" | Username or password incorrect | Verify login credentials in gateway settings. |
| Transaction created but confirmation fails with "Operation expired" | Long time between creation and confirmation | Create a new transaction and confirm within the validity period (usually 5-10 minutes). |
| OTP code incorrect | Customer entered wrong code | Request OTP again (if supported) or create a new transaction. |
| Refund fails with "Terminal balance is not enough" | Terminal balance insufficient | Retry refund with `isTerminal: false` to deduct from merchant main balance. |
| `test-ui` interface does not appear | Current user is not an administrator (backend) | Ensure you are logged into the control panel as an admin. |

---

## 11. Future Improvements (Suggestions for Developers)

1. **Temporary token caching** – Add a cache layer to store `access_token` for 3600 seconds to reduce login requests.
2. **Webhook support** – Receive real-time notifications when payment is confirmed or transaction expires.
3. **Automatic retry** – If confirmation fails due to expiration, automatically recreate the transaction.
4. **Extend test interface** – Add full testing for refunds and password changes.

---

## 12. References and Links

- [YottaPay Login & Change Password Documentation](./yottaPay%20Login%20-%20Change%20My%20Password%20API%20Documentation%20V.1.1.pdf)
- [Online Payments API v1.3](./API%20Online%20Payments%20v%201.3.pdf)
- [Online Payment Return API v1.4](./API%20Online%20Payment%20Return%20v%201.4.pdf)
- [Online Status Check API v1.3](./API%20Online%20Status%20Check%20v1.3.pdf)
- [Postman Collection for API Testing](./Sabacash%20online%20payment.postman_collection.json)
- [YottaPay Class in NanoSoft](./YottaPay.php)
- [Routes file with YottaPay endpoints](./routes.php)

---

**This documentation is prepared to help developers integrate and use the YottaPay gateway easily and efficiently.**  
For inquiries or technical support, please contact via the official website [nano2soft.com](https://nano2soft.com) or dedicated support channels.
