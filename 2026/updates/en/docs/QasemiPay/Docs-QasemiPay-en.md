# QasemiPay Payment Gateway Documentation – Qasemi Islamic Bank

## 1. Overview

**QasemiPay** is an integrated payment gateway that enables accepting payments via the **Purchase Code Service** provided by Qasemi Islamic Bank.  
The gateway relies on an API that uses the **OAuth 2.0** protocol for authentication, and payment operations are performed through two basic steps:

1. **Create Transaction** – Customer data and purchase code are sent.
2. **Confirm Transaction** – Uses `concurrencyStamp` to complete the operation.

The gateway also provides the ability to **check transaction status** at any time via the reference ID (`refId`).

This gateway is built specifically for **Nano2Soft App** and works as an add-on within `Nano.Yepayment`, following the same pattern as other payment gateways such as `YottaPay` and `ThawaniPay`.

---

## 2. Installation and Operation Requirements

- Nano2Soft App version 2+.
- `Nano.MicroCart` add-on for shopping cart and order management.
- `Nano.Yepayment` add-on (which includes the QasemiPay gateway).
- API access credentials from Qasemi Bank:
  - `Client ID`
  - `Client Secret`
  - `Username`
  - `Password`
  - `Scope` (default: `purchase_code_service`)

---

## 3. Payment Gateway Settings (Backend Settings)

When activating the **"Qasemi Islamic Bank"** payment method in the control panel, the following settings will appear:

| Field | Description | Default Value |
|-------|-------------|---------------|
| **Base API URL** | Bank API endpoint URL | `https://api.qasemi-bank.com` |
| **Test API URL** | Testing environment URL (optional) | `https://test-api.qasemi-bank.com` |
| **Client ID** | Client ID from the bank | - |
| **Client Secret** | Client secret key | - |
| **Username** | Integration username | - |
| **Password** | User password | - |
| **Scope** | Required access scope | `purchase_code_service` |
| **Default Currency** | Currency used if not specified by customer | `YER` (Yemeni Rial) or `SAR` |
| **Default Zone** | Merchant geographic zone | `Z1` |
| **Default Agent** | Agent ID (numeric) | `0` |
| **Default Branch** | Branch ID (string) | empty |

> **Note:** `Client Secret` and `Password` fields are stored encrypted in the database.

---

## 4. Payment Flow

### 4.1. Create New Transaction

The system calls `process()` when payment starts. Steps:

1. **Validate inputs** – `purchase_code`, `mobile_number`, `amount`, `currency` are required.
2. **Request OAuth 2.0 token** – using registered credentials.
3. **Send create transaction request** to `/api/purchase-code-management/purchase-transactions` with the following data:
   ```json
   {
     "purchaseCode": "PC-987654321",
     "currency": "YER",
     "mobileNumber": "+967700000000",
     "amount": 100,
     "refId": "uuid-v4",
     "refNo": "uuid-v4",
     "notes": "Payment for order #200",
     "region": {
       "zone": "Z1",
       "user": "Username",
       "agent": 0,
       "branch": ""
     }
   }
   ```
4. **Save transaction data** inside `order->other_data['qasemi']`:
   - `transaction_id` (internal transaction ID)
   - `concurrency_stamp`
   - `ref_id` and `ref_no`
5. **Log payment attempt** in `nano_microcart_payment_logs` table (status: unconfirmed).
6. Return `PaymentResult` with `successful = true` and message **"Payment must be confirmed"**, without redirection.

### 4.2. Confirm Transaction

`complete()` is called after obtaining `concurrencyStamp` (can be requested from the user via OTP or called automatically from the merchant interface). Steps:

1. Retrieve `transaction_id` and `concurrencyStamp` from `order->other_data`.
2. Request a new token (or use cached one).
3. Send confirmation request to `/api/purchase-code-management/purchase-transactions/confirm`:
   ```json
   {
     "id": "d625aefc-7204-f951-c33c-3a1d09bcc8da",
     "concurrencyStamp": "stamp_string_here"
   }
   ```
4. Check `status == 2` (Succeeded).
5. Update `order->payment_state` to `PaidState`, save `ref_no` in `payment_trans_id`.
6. Log payment as **successful** via `$result->success()`.

### 4.3. Check Transaction Status

`checkTransactionStatus($refId)` can be called at any time to check transaction status.  
Status codes according to bank documentation:

| Status Code | Status           |
|-------------|------------------|
| 2           | Succeeded        |
| 3           | Failed           |
| 4           | Pending          |
| 6           | Refunded         |
| 8           | Cancelled        |

---

## 5. Data Structure Stored in `order->other_data['qasemi']`

```php
[
    'order_total' => 100.00,                     // Total order amount
    'transaction_id' => 'd625aefc-7204-...',     // id from create transaction response
    'concurrency_stamp' => 'stamp...',           // Concurrency stamp from response
    'ref_id' => 'uuid-v4',                       // Reference ID sent in request
    'ref_no' => 'uuid-v4',                       // Another reference number
    'requires_confirmation' => true,             // Indicates payment awaiting confirmation
    'created_at' => '2025-01-01T12:00:00Z',      // Creation time
    'callback_success_url' => '...',             // Success callback URL (optional)
    'callback_error_url' => '...'                // Error callback URL (optional)
]
```

---

## 6. API Test UI

The gateway provides an integrated web interface for testing all functions, accessible at:

```
/api/v1/yepayment/qasemipay/test-ui
```

### 6.1. Features

- **Auth Testing** – Verify Client ID/Secret and username/password.
- **Create Transaction** – Enter purchase code, mobile number, amount, etc.
- **Confirm Transaction** – Using `transaction_id` and `concurrencyStamp`.
- **Check Status** – Via `refId`.
- **Full Test** – Automatically executes all three steps and displays results.
- **Automated Multi-Run Test** – To measure gateway stability.
- **Instant Statistics** – Request count, success rate, latest logs.
- **Test Logs** – LocalStorage storage of test results.

### 6.2. API Endpoints Used in Testing

| Purpose | Method | Path |
|---------|--------|------|
| Authentication | POST | `/qasemipay/test-auth` |
| Create Transaction | POST | `/qasemipay/test-create-payment` |
| Confirm Transaction | POST | `/qasemipay/test-confirm-payment` |
| Check Status | GET | `/qasemipay/test-check-status?ref_id=...` |
| Full Test | POST | `/qasemipay/test-full-payment` |
| Statistics | GET | `/qasemipay/stats` |

---

## 7. Request and Response Examples

### 7.1. Successful Create Transaction

**Request:**
```http
POST /api/v1/yepayment/qasemipay/test-create-payment
Content-Type: application/json

{
    "order_id": 200,
    "purchase_code": "PC-987654321",
    "mobile_number": "+967700000000",
    "amount": 100,
    "currency": "YER",
    "zone": "Z1"
}
```

**Response:**
```json
{
    "success": true,
    "message": "Transaction created successfully",
    "data": {
        "transaction_id": "d625aefc-7204-f951-c33c-3a1d09bcc8da",
        "ref_no": "87c5dfeb-d57f-494e-86ad-96ed5a6d814b",
        "requires_confirmation": true,
        "concurrency_stamp": "stamp_string_here",
        "ref_id": "123e4567-e89b-12d3-a456-426614174000"
    }
}
```

### 7.2. Successful Confirm Transaction

**Request:**
```http
POST /api/v1/yepayment/qasemipay/test-confirm-payment
{
    "order_id": 200,
    "transaction_id": "d625aefc-7204-f951-c33c-3a1d09bcc8da",
    "concurrency_stamp": "stamp_string_here"
}
```

**Response:**
```json
{
    "success": true,
    "message": "Payment confirmed successfully",
    "data": {
        "ref_no": "028030a3-d748-4ead-babd-89a43caef612",
        "completed": true
    }
}
```

### 7.3. Check Status

**Request:**
```http
GET /api/v1/yepayment/qasemipay/test-check-status?ref_id=123e4567-e89b-12d3-a456-426614174000
```

**Response:**
```json
{
    "success": true,
    "message": "Transaction status retrieved",
    "data": {
        "status_code": 2,
        "status": "Succeeded",
        "ref_id": "123e4567...",
        "ref_no": "028030a3...",
        "operation_number": 483287097411584
    }
}
```

---

## 8. Integrating the Gateway into Your Project (Developer Guide)

### 8.1. Register the Payment Method in `Plugin.php`

```php
public function registerPaymentProviders()
{
    $providers = [];
    // ... other gateways ...
    if (class_exists(\Nano\Yepayment\PaymentTypes\QasemiPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\QasemiPay();
    }
    return $providers;
}
```

### 8.2. Using the Gateway via `PaymentGateway`

```php
use Nano\MicroCart\Classes\Payments\DefaultPaymentGateway;
use Nano\Orders\Models\Order;

$order = Order::find($orderId);
$gateway = new DefaultPaymentGateway();
$gateway->init($paymentMethod, $userInputData);
$result = $gateway->process($order);
if ($result->successful && $result->redirect) {
    return redirect()->to($result->redirectUrl);
}
```

### 8.3. Confirm Payment After User Return

```php
$paymentProvider = new QasemiPay();
$paymentProvider->setOrder($order);
$result = new PaymentResult($paymentProvider, $order);
$completeResult = $paymentProvider->complete($result);
if ($completeResult->successful) {
    // Payment completed successfully
}
```

---

## 9. Gateway Code Files

| File | Description |
|------|-------------|
| `QasemiPay.php` | Main gateway class |
| `qasemi-ui.htm` | HTML/JS test interface |
| `_info.htm` | Display information in settings |
| `_test_info.htm` | Quick test tools in settings |

**File location within the add-on:**  
`plugins/nano/yepayment/paymenttypes/qasemipay/`

---

## 10. Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Authentication failed | Client ID/Secret or username/password incorrect | Verify credentials in settings |
| `concurrencyStamp` missing | Not saved after transaction creation | Ensure `process()` succeeded before calling `complete()` |
| `Failed` status on confirmation | `concurrencyStamp` outdated or transaction not confirmable | Create a new transaction |
| `refId` not found | Not generated during creation | Check API response; field name may differ |
| `403 Forbidden` error | Token expired or insufficient permissions | Request a new token (class does this automatically) |

---

## 11. Additional Notes

- **The provided API documentation does not include an OTP mechanism** – therefore the confirmation step relies solely on `concurrencyStamp`. If the bank requires an external OTP, `complete()` can be modified to read `otp` from `Request::input('otp')` and send it in the confirmation body if supported.
- **Temporary token caching** – To improve performance, a caching layer can be added to store the token for 3600 seconds instead of requesting it each time.
- **Developer compatibility** – The `QasemiPay` class is independent and can be used outside `Nano.MicroCart` if a suitable `Order` object is provided.

---

**This documentation is prepared to help developers integrate and use the QasemiPay gateway easily and efficiently.**  
For inquiries or technical support, please contact via the official website nano2soft.com.

---
