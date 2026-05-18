# BasPay Payment Gateway Documentation – BAS Platform

## 1. Overview

**BasPay** is an immediate/direct payment gateway integrated within the `Nano.Yepayment` package in Nanosoft applications. The gateway relies on the **BAS Platform** API and enables electronic payment execution instantly without needing user redirection or additional confirmation steps.

The gateway operates according to the following mechanism:
1. **OAuth 2.0 Authentication** – obtaining an access token (Bearer Token) using the merchant credentials.
2. **Creating an immediate payment transaction** – sending the order data (amount, currency, order number) with an AES-256-CBC encrypted signature to the BAS Platform API, and receiving a `trxToken` immediately.

The gateway supports the currencies: Yemeni Rial (`YER`), US Dollar (`USD`), and Saudi Riyal (`SAR`). The gateway also features optional support for callback URLs using `RedirectHelper` to be compatible with mobile applications.

This gateway is built according to `Nano.Yepayment` standards and follows the `PaymentProvider` pattern like `YottaPay` and `QasemiPay`.

---

## 2. Operating Requirements and Settings

### 2.1. Basic Requirements

- Nanosoft System (Nano2Soft) version 2.0+
- Required plugins:
  - `Nano.MicroCart` (>=2.0)
  - `Nano.Yepayment` (>=1.2)
  - `Nano.Helpers`
- BAS platform credentials (provided by technical support):
  - `Client ID`
  - `Client Secret`
  - `App ID`
  - `Merchant Key` (for encrypted signature)
  - `IV` – initialization vector value (16 bytes), often the default value `@@@@&&&&####$$$$`
  - Base API URL (example: `https://api.basgate.com`)

### 2.2. Gateway Settings in the Control Panel

When activating the **"BAS Payment Gateway"** payment method, the following fields appear on the payment settings page:

| Field | Key | Description | Default Value |
|-------|-----|-------------|---------------|
| Base API URL | `baspay_url` | BAS Platform API address | `https://api.basgate.com` |
| Client ID | `baspay_client_id` | Client identifier | - |
| Client Secret | `baspay_client_secret` | Client secret key (stored encrypted) | - |
| App ID | `baspay_app_id` | Application identifier | - |
| Merchant Key | `baspay_merchant_key` | Merchant key for encrypted signature (stored encrypted) | - |
| IV | `baspay_iv` | Initialization vector for encryption (16 bytes) | `@@@@&&&&####$$$$` |
| Default Currency | `baspay_default_currency` | Currency used when not specified | `YER` |

> **Security Note:** Sensitive fields (`Client Secret`, `Merchant Key`) are stored encrypted in the database via `encryptedSettings()`.

---

## 3. Payment Lifecycle (Payment Flow)

### 3.1. Immediate Payment Execution (`process`)

When initiating payment for a specific order, `process()` is called and performs the following steps:

1. **Check order status** – if the order is already paid (`PaidState`), the process is halted.
2. **Obtain access token** via OAuth 2.0 using `client_credentials` (stored in Cache for 3500 seconds).
3. **Generate encrypted signature** according to the BAS mechanism:
   - Create a random `salt` string of 4 characters.
   - Compute `SHA256` of the string to be signed with the `salt`.
   - Encrypt the result using `AES-256-CBC` with the merchant key and `IV`.
4. **Send transaction creation request** to `/api/v1/merchant/sdk-payment/initiate-transaction` with the following data:
   ```json
   {
     "head": {
       "signature": "<signature>",
       "requestTimestamp": 1696000000000
     },
     "body": {
       "amount": { "value": 100, "currency": "YER" },
       "ordertype": "PayBill",
       "orderId": "200",
       "requestTimestamp": 1696000000000,
       "appId": "<app_id>"
     }
   }
   ```
5. **Receive the response** – if `status = 1` and `code = '1111'`, the transaction is successful, and `trxToken` is extracted from `response.body.trxToken`.
6. **Update order data**:
   - Set `order.payment_first_trans_id` and `order.payment_trans_id` to the `trxToken` value.
   - Store operation information in `order.other_data['baspay']` (token, amount, currency, timestamp).
   - Change payment status to `PaidState`.
7. **Log payment record** in the `PaymentLog` table.
8. Return a `PaymentResult` with success (`successful = true`).

### 3.2. Optional Redirect Links (Optional)

Although BasPay is a direct payment gateway, it supports storing `callback_success_url` and `callback_error_url` if they are passed within payment data (e.g., from a mobile app). After executing the payment, the developer can redirect the user using the following routes:

- `GET /api/v1/yepayment/baspay/success` – retrieves the return URL upon success and uses `RedirectHelper` to redirect to the app or website.
- `GET /api/v1/yepayment/baspay/cancel` – similar for cancellation.

This ensures full compatibility with mobile applications that require deep links after payment.

---

## 4. Data Structure Stored in `order->other_data['baspay']`

```php
[
    'trx_token' => 'abc123...',             // Unique transaction identifier from BAS
    'amount'    => 100.00,                  // Paid amount
    'currency'  => 'YER',                   // Currency
    'order_id'  => '200',                   // Order number
    'timestamp' => '2025-01-01T12:00:00Z',  // Execution time
    'callback_success_url' => 'myapp://...',// (Optional) Return URL on success
    'callback_error_url'   => 'myapp://...' // (Optional) Return URL on error
]
```

---

## 5. API Test Interface (Test UI)

The gateway provides an integrated web interface for testing all functions, accessible via:

```
/api/v1/yepayment/baspay/test-ui
```

### 5.1. Features

- **Authentication Test** (Auth) – verify Client ID/Secret validity via `/baspay/test-auth`.
- **Create Immediate Payment** – enter order number, amount, currency to execute a new payment.
- **Check Status** – query a transaction using `trxToken`.
- **Comprehensive Test** – combines creating a payment and checking status in one step.
- **Automated Repeated Testing** – ability to specify the number of repetitions to measure stability.
- **Real-time Statistics** – number of requests, success rate, latest logs.
- **Local Logs** – test results stored in browser `localStorage`.

### 5.2. API Endpoints Used in Testing

| Purpose | Method | Path |
|---------|--------|------|
| Authentication | POST | `/baspay/test-auth` |
| Create Payment | POST | `/baspay/test-create-payment` |
| Check Status | POST | `/baspay/test-check-status` (body: `{trx_token}`) |
| Comprehensive Test | POST | `/baspay/test-full-payment` |
| Statistics | GET | `/baspay/stats` |

---

## 6. Request and Response Examples

### 6.1. Create a New Payment (Successful)

**Request:**
```http
POST /api/v1/yepayment/baspay/test-create-payment
Content-Type: application/json

{
    "order_id": 200,
    "amount": 100,
    "currency": "YER"
}
```

**Response:**
```json
{
    "success": true,
    "message": "Payment completed",
    "trx_token": "bas_trx_a1b2c3d4...",
    "api_data": {
        "success": true,
        "trx_token": "bas_trx_a1b2c3d4...",
        "amount": 100,
        "currency": "YER",
        "order_id": "200",
        "raw_response": { ... }
    }
}
```

### 6.2. Check Transaction Status

**Request:**
```http
POST /api/v1/yepayment/baspay/test-check-status
Content-Type: application/json

{
    "trx_token": "bas_trx_a1b2c3d4..."
}
```

**Response (example):**
```json
{
    "success": true,
    "data": {
        "head": { ... },
        "body": {
            "trxToken": "bas_trx_a1b2c3d4...",
            "trxStatus": "SUCCESS",
            "order": { "orderId": "200" }
        }
    }
}
```

### 6.3. Comprehensive Test (Create + Check)

**Request:**
```http
POST /api/v1/yepayment/baspay/test-full-payment
{
    "order_id": 200,
    "amount": 100,
    "currency": "YER"
}
```

**Response:**
```json
{
    "success": true,
    "results": {
        "step1_create": {
            "success": true,
            "trx_token": "bas_trx_..."
        },
        "step2_status": {
            "success": true,
            "data": { ... }
        }
    }
}
```

---

## 7. Integrating the Gateway into Your Project (Developer Guide)

### 7.1. Registering the Payment Method in `Plugin.php`

```php
if (config('nano.microcart::paymenttypes.allow_yemen_payment', true)) {
    // ... other providers ...
    if (class_exists(\Nano\Yepayment\PaymentTypes\BasPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\BasPay();
    }
}
```

### 7.2. Using the Gateway via `PaymentGateway`

```php
use Nano\MicroCart\Classes\Payments\DefaultPaymentGateway;
use Nano\Orders\Models\Order;

$order = Order::find($orderId);
$gateway = new DefaultPaymentGateway();
$gateway->init($paymentMethod, ['callback_success_url' => 'myapp://pay/success']);
$result = $gateway->process($order);
if ($result->successful) {
    // Payment successful – can redirect to success_url if present
}
```

### 7.3. Executing Payment Directly (without PaymentGateway)

```php
use Nano\Yepayment\PaymentTypes\BasPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$bas = new BasPay($order, [
    'callback_success_url' => 'myapp://pay/success', // optional
]);
$paymentResult = new PaymentResult($bas, $order);
$processResult = $bas->process($paymentResult);

if ($processResult->successful) {
    $trxToken = $order->payment_first_trans_id;
    // Redirect if needed
}
```

---

## 8. Troubleshooting

| Problem | Possible Cause | Solution |
|---------|----------------|----------|
| Authentication failure (`auth_failed`) | Incorrect Client ID/Secret | Verify the entered settings in the control panel |
| `payment_creation_failed` with an error response from BAS | `trxToken` missing or `code` not equal to `1111` | Examine logs for the full response; amount may be below the minimum or currency unsupported |
| Invalid signature | Merchant Key or IV mismatch | Ensure values match those provided by BAS Platform |
| Order already paid | `order.payment_state == PaidState` | Cannot pay again for the same order |
| `cURL error` | Connection issue with the API | Verify URL correctness and internet connectivity |
| Unknown transaction status when checking | `trxToken` is incorrect | Ensure the token used in the status check is the same as the one from `createPayment` |

---

## 9. Additional Notes

- The **BasPay** gateway does not require `complete()` because payment is immediate; the function exists for interface compatibility only.
- The encrypted signature relies on the same algorithm used in **BAS SDK** (`AES-256-CBC + SHA256 + salt`), ensuring full compatibility with BAS Platform requirements.
- The `Access Token` is cached for 3500 seconds to reduce the number of authentication requests.
- The class can be extended to add **Webhook** or refund features in the future without breaking the current structure.

---

## 10. Gateway Code Files

| File | Description |
|------|-------------|
| `BasPay.php` | Main gateway class (extends `PaymentProvider`) |
| `_info.htm` | Settings information template in the control panel |
| `_test_info.htm` | Quick test tools with interactive buttons |
| `baspay-ui.htm` | Full test interface (Test UI) |
| `routes.php` (BasPay section) | Definition of test endpoints, statistics, and user interface |
| `Plugin.php` (addition) | Registration of the gateway as a payment provider |

---

## 11. Related Links

- [SKILL-en.md Document for Creating Payment Gateways](./SKILL-en.md)
- [BAS Platform API Guide](./external/BAS/README-en.md)
- [BasGate Documentation Repository on GitHub](https://github.com/basgate/basgate.github.io)
- [Laravel Payment SDK Package on GitHub](https://github.com/basgate/laravel-payment-sdk)
- [BasPaymentFlutter Repository on GitHub](https://github.com/BasPlatform/BasPaymentFlutter.git)
- [bas_php_sdk Package on GitHub](https://github.com/basgate/bas_php_sdk)
- [bas-laravel-sdk Package on GitHub](https://github.com/basgate/bas-laravel-sdk)

**This documentation has been prepared to support developers in easily and efficiently integrating and using the BasPay gateway.**  
For inquiries or technical support, please contact us via the official website [nano2soft.com](https://nano2soft.com).
