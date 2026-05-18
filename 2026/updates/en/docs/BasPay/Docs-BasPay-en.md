# BasPay Payment Gateway Documentation – BAS Platform (Updated)

## 1. Overview

**BasPay** is a payment gateway that operates in accordance with the **Two‑Step Payment** pattern and is integrated within the `Nano.Yepayment` package in NanoSoft applications. The gateway relies on the **BAS Platform API**, allowing electronic payments to be executed by first creating a transaction and then confirming it later after the customer completes the payment via the BAS app.

Unlike instant gateways, **BasPay does not deduct the amount directly** when the API is called; instead, it follows a two‑phase flow:

1. **`process()` – Transaction Creation:** The merchant sends a transaction creation request to the BAS platform and receives a unique transaction identifier (`trxToken`). At this stage, the identifier is saved in the order, but the order **does not** change to a "paid" status.
2. **`complete()` – Transaction Confirmation:** After the customer completes the payment in the BAS app, the system queries the transaction status using the `trxToken`. If the status is successful (`SUCCESS`), the order is updated to "paid" and the associated actions are triggered.

The gateway supports the currencies: Yemeni Rial (`YER`), US Dollar (`USD`), and Saudi Riyal (`SAR`). It also provides optional support for callback URLs using `RedirectHelper` to ensure compatibility with mobile applications.

This gateway is built according to `Nano.Yepayment` standards and follows the `PaymentProvider` pattern like `YottaPay` and `QasemiPay`.

---

## 2. Operating Requirements and Settings

### 2.1. Basic Requirements

- Nano2Soft system version 2.0+
- Required plugins:
  - `Nano.MicroCart` (>=2.0)
  - `Nano.Yepayment` (>=1.2)
  - `Nano.Helpers`
- BAS platform credentials (provided by technical support):
  - `Client ID`
  - `Client Secret`
  - `App ID`
  - `Merchant Key` (for encrypted signature)
  - `IV` – Initialization vector (16 bytes), often the default `@@@@&&&&####$$$$`
  - Base API URL (example: `https://api.basgate.com`)

### 2.2. Gateway Settings in the Control Panel

When the **"BAS Payment Gateway"** payment method is enabled, the following fields appear in the payment settings page:

| Field | Key | Description | Default Value |
|-------|-----|-------------|---------------|
| Base API URL | `baspay_url` | BAS Platform API address | `https://api.basgate.com` |
| Client ID | `baspay_client_id` | Client identifier | - |
| Client Secret | `baspay_client_secret` | Client secret key (stored encrypted) | - |
| App ID | `baspay_app_id` | Application ID | - |
| Merchant Key | `baspay_merchant_key` | Merchant key for encrypted signature (stored encrypted) | - |
| IV | `baspay_iv` | Initialization vector for encryption (16 bytes) | `@@@@&&&&####$$$$` |
| Default Currency | `baspay_default_currency` | Currency used when not specified | `YER` |

> **Security Note:** Sensitive fields (`Client Secret`, `Merchant Key`) are stored encrypted in the database via `encryptedSettings()`.

---

## 3. Payment Flow Lifecycle

The BasPay gateway operates in two separate phases, aligning with the "Two‑Step Payment" pattern in the `Nano.Yepayment` system.

### 3.1. Phase One: Transaction Creation (`process`)

When initiating payment for a specific order, `process()` is called and performs the following steps:

1. **Validate order status** – If the order is already paid (`PaidState`), the process is stopped.
2. **Obtain an access token** via OAuth 2.0 using `client_credentials` (stored in cache for 3500 seconds).
3. **Generate an encrypted signature** according to the BAS mechanism:
   - Create a random `salt` string of 4 characters.
   - Calculate `SHA256` of the text to be signed with the `salt`.
   - Encrypt the result using `AES-256-CBC` with the merchant key and `IV`.
4. **Send a transaction creation request** to `/api/v1/merchant/sdk-payment/initiate-transaction` with the order data and signature.
5. **Receive the response** – If `status = 1` and `code = '1111'`, the transaction is successful, and the `trxToken` is extracted from `response.body.trxToken`.
6. **Save transaction data** in the order without changing payment status:
   - Set `order.payment_first_trans_id` and `order.payment_trans_id` to the `trxToken` value.
   - Store operation information in `order.other_data['baspay']` (token, amount, currency, timestamp, return URLs).
   - **Do not** change payment status to `PaidState` at this stage.
7. **Record payment log** in the `PaymentLog` table.
8. Return a `PaymentResult` with partial success (`successful = true`) and a message indicating that payment confirmation is required (`confirmation_required`).

> **Developer Note:** At this stage, the system considers the order **pending confirmation** and waits for the customer to complete payment via the BAS app.

### 3.2. Phase Two: Transaction Confirmation (`complete`)

After the customer completes the payment process in the BAS app, the system must verify the transaction status and finalize the order. This is done via the `complete()` function (or the helper function `checkAndCompletePay`) which performs the following steps:

1. **Extract `trxToken`** from the order data.
2. **Obtain a new access token** (or from cache).
3. **Send a status check request** to `/api/v1/merchant/sdk-payment/get-transaction-status` with the `trxToken`.
4. **Analyze the response**:
   - If `trxStatus` equals `SUCCESS` or `COMPLETED`: call `$result->success()` which updates the order to `PaidState`, fires events (such as `nano.orders.paymentProcessed`), and empties the cart.
   - If `PENDING` or `PROCESSING`: call `$result->pending()` and the order remains pending.
   - Any other status: return a failure.
5. **Return the final result** so that other system modules (such as `RedirectHelper`) can perform the appropriate redirection.

### 3.3. Optional Callback Redirect URLs

BasPay supports storing `callback_success_url` and `callback_error_url` if passed within payment data (e.g., from a mobile app). After payment confirmation, the developer can direct the user using the routes:

- `GET /api/v1/yepayment/baspay/success` – retrieves the return URL on success and uses `RedirectHelper` to redirect to the app or website.
- `GET /api/v1/yepayment/baspay/cancel` – similar for cancellation.

This ensures full compatibility with mobile apps that require Deep Links after payment.

---

## 4. Data Structure Stored in `order->other_data['baspay']`

```php
[
    'trx_token'             => 'bas_trx_a1b2c3d4...',  // Unique transaction identifier from BAS
    'amount'                => 100.00,                  // Amount
    'currency'              => 'YER',                   // Currency
    'order_id'              => '200',                   // Order number
    'requires_confirmation' => true,                    // Does it require confirmation? (always true)
    'created_at'            => '2025-01-01T12:00:00Z', // Transaction creation time
    'callback_success_url'  => 'myapp://pay/success',  // (Optional) Return URL on success
    'callback_error_url'    => 'myapp://pay/error'     // (Optional) Return URL on error
]
```

---

## 5. Integrated Scenario for Using BasPay in Your Application

This scenario illustrates the steps taken by an app or store developer to interact with BasPay through the `Checkout2` API.

### 5.1. Prerequisites
- A mobile app or website using `Nano.Yepayment` and `Nano.ShopApi`.
- BasPay gateway configured in the control panel.
- The customer has an account on the BAS platform (to confirm payment later).

### 5.2. Steps from the Developer's Perspective

#### **Step 1: User selects BasPay payment method**
The app sends a request to the `checkout` endpoint with `step=pay` and the payment method id `baspay`.

**Example Request:**
```http
POST /api/v1/shop/checkout
Content-Type: application/json

{
  "step": "pay",
  "payment_method_id": 5,
  "callback_success_url": "myapp://checkout/success",
  "callback_error_url": "myapp://checkout/error"
}
```

#### **Step 2: System processes payment via `Checkout2` and `OrderManager`**
- The system calls `OrderManager->setStepPayments()` which in turn:
  1. Validates shipping addresses and coupons.
  2. Creates `PaymentService` and `PaymentGateway` based on the chosen payment method (BasPay).
  3. Calls `BasPay->process()`.

#### **Step 3: `BasPay->process()` creates the transaction and returns `trxToken`**
- The gateway communicates with BAS API and creates a new transaction.
- The developer receives in the API Response:
  ```json
  {
    "status": true,
    "step_status": true,
    "payment_state": null,
    "processed": false,
    "process_data": {
      "paymentResult": {
        "successful": true,
        "message": "Please confirm payment via the BAS app",
        "api_data": {
          "trx_token": "bas_trx_a1b2c3d4..."
        }
      }
    }
  }
  ```
- **Important:** `processed` is still `false`, meaning payment is not complete.

#### **Step 4: Developer asks the customer to complete payment in the BAS app**
- The app displays a message: "Please open the BAS app and complete the payment."
- This may include a "Open BAS" button or a Deep Link to the app.
- Meanwhile, the developer has the `trxToken` (available in `payment_first_trans_id` and `other_data`).

#### **Step 5: Developer verifies payment completion**
After the customer completes payment in the BAS app, the developer has two options:

**Option A: Use the `baspay/success` route with return URLs (recommended for mobile apps):**
- When the user returns from the BAS app to your app via Deep Link `myapp://checkout/success?order_id=200`, the app calls:
  ```http
  GET /api/v1/yepayment/baspay/success?order_id=200
  ```
- The route responds using `RedirectHelper` and redirects the user to the correct page with the payment result.

**Option B: Direct status query (for apps without Deep Links):**
- The developer can call `POST /api/v1/yepayment/baspay/test-check-status` (for admins) or create a custom endpoint that uses `BasPay::checkAndCompletePay()`.
- Example manual call:
  ```php
  $result = \Nano\Yepayment\PaymentTypes\BasPay::checkAndCompletePay(['order_id' => 200]);
  if ($result['success']) {
      // Payment done, update UI
  }
  ```

#### **Step 6: Order becomes paid**
- Once `complete()` succeeds, the system calls `$result->success()` which:
  - Changes `payment_state` to `PaidState`.
  - Sets `processed = true`.
  - Notifies the system with associated events (sending email, updating stock, etc.).
  - Empties the user's cart.

#### **Step 7: Final redirection**
- The user is redirected to the payment success page (or to the Deep Link `myapp://checkout/success`) with order data.

---

## 6. API Test Interface (Test UI)

The gateway provides a comprehensive web interface to test all functions, accessible at:

```
/api/v1/yepayment/baspay/test-ui
```

### 6.1. Features

- **Auth Test** – Verify Client ID/Secret via `/baspay/test-auth`.
- **Initiate Transaction** – Enter order number, amount, currency to create a new transaction and receive a `trxToken`.
- **Status Check** – Query a transaction using `trxToken` and check its status.
- **Full Flow Test** – Combines creation and status check in one step (simulates both phases).
- **Automated Repeat Test** – Option to specify the number of repetitions to measure stability.
- **Real‑time Statistics** – Request count, success rate, recent logs.
- **Local Logs** – Store test results in the browser's `localStorage`.

### 6.2. API Endpoints Used in Testing

| Purpose | Method | Path |
|---------|--------|------|
| Authentication | POST | `/baspay/test-auth` |
| Create Transaction | POST | `/baspay/test-create-payment` |
| Check Status | POST | `/baspay/test-check-status` (body: `{trx_token}`) |
| Full Flow (both phases) | POST | `/baspay/test-full-payment` |
| Statistics | GET | `/baspay/stats` |

---

## 7. Example Requests and Responses

### 7.1. Create New Transaction (Phase One)

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
    "message": "Please confirm payment via the BAS app",
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

### 7.2. Check Transaction Status (Phase Two)

**Request:**
```http
POST /api/v1/yepayment/baspay/test-check-status
Content-Type: application/json

{
    "trx_token": "bas_trx_a1b2c3d4..."
}
```

**Response (Example – Success):**
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

### 7.3. Full Flow (Both Phases Together)

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
            "trxStatus": "SUCCESS"
        }
    }
}
```

---

## 8. Integrating the Gateway into Your Project (Developer Guide)

### 8.1. Registering the Payment Method in `Plugin.php`

```php
if (config('nano.microcart::paymenttypes.allow_yemen_payment', true)) {
    // ... other providers ...
    if (class_exists(\Nano\Yepayment\PaymentTypes\BasPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\BasPay();
    }
}
```

### 8.2. Using the Gateway via `PaymentGateway` (Phase One Only)

```php
use Nano\MicroCart\Classes\Payments\DefaultPaymentGateway;
use Nano\Orders\Models\Order;

$order = Order::find($orderId);
$gateway = new DefaultPaymentGateway();
$gateway->init($paymentMethod, ['callback_success_url' => 'myapp://pay/success']);
$result = $gateway->process($order);
if ($result->successful) {
    // Transaction created – customer must be directed to the BAS app
    $trxToken = $order->payment_first_trans_id;
}
```

### 8.3. Confirming Payment Manually (Phase Two)

After the customer confirms payment in the BAS app, you can use the helper function:

```php
use Nano\Yepayment\PaymentTypes\BasPay;

$result = BasPay::checkAndCompletePay(['order_id' => 200]);
if ($result['success']) {
    // Payment successful
}
```

---

## 9. Troubleshooting

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| Auth failed (`auth_failed`) | Incorrect Client ID/Secret | Verify the settings entered in the control panel |
| `payment_creation_failed` with error response from BAS | Missing `trxToken` or `code` not equal to `1111` | Check logs for full response; amount may be below minimum or currency not supported |
| Invalid signature | Merchant Key or IV mismatch | Ensure values match those provided by BAS platform |
| Order already paid | `order.payment_state == PaidState` | Cannot pay again for the same order |
| `cURL error` | Connection issue with API | Verify the URL and internet connectivity |
| Unknown transaction status during check | Incorrect `trxToken` | Ensure the token used in the status check is the same as that from `initiateTransaction` |
| Order not updated to paid after `complete` | `trxStatus` is not `SUCCESS` | Check the transaction status in the BAS dashboard; the customer may not have confirmed yet |

---

## 10. Additional Notes

- **The gateway follows the Two‑Step pattern**, so payment is not immediate. You must use `complete()` or `checkAndCompletePay()` to confirm payment.
- The encrypted signature relies on the same algorithm used in **BAS SDK** (`AES-256-CBC + SHA256 + salt`), ensuring full compatibility with BAS platform requirements.
- The `Access Token` is stored in cache for 3500 seconds to reduce the number of authentication requests.
- The class can be extended to add **Webhook** or refund support in the future without breaking the existing structure.
- **The `checkAndCompletePay` function** is a public `static` function used to link the confirmation phase with the `baspay/success` route.

---

## 11. Gateway Code Files

| File | Description |
|------|-------------|
| `BasPay.php` | Main gateway class (extends `PaymentProvider`) |
| `_info.htm` | Settings information template in the control panel |
| `_test_info.htm` | Quick test tools with interactive buttons |
| `baspay-ui.htm` | Full test interface (Test UI) |
| `routes.php` (BasPay section) | Defines test endpoints, statistics, and UI |
| `Plugin.php` (addition) | Registers the gateway as a payment provider |

---

## 12. Related Links

- [SKILL-ar.md document for creating payment gateways](./SKILL-ar.md)
- [BAS Platform API Guide](https://basgate.apidog.io)
- [BAS Platform API Guide](./external/BAS/README-ar.md)
- [BasGate documentation repository on GitHub](https://github.com/basgate/basgate.github.io)
- [Laravel Payment SDK on GitHub](https://github.com/basgate/laravel-payment-sdk)
- [BasPaymentFlutter repository on GitHub](https://github.com/BasPlatform/BasPaymentFlutter.git)
- [bas_php_sdk on GitHub](https://github.com/basgate/bas_php_sdk)
- [bas-laravel-sdk on GitHub](https://github.com/basgate/bas-laravel-sdk)


**This documentation was prepared to support developers in integrating and using the BasPay gateway easily and efficiently.**  
For inquiries or technical support, please contact us via the official website [nano2soft.com](https://nano2soft.com).

