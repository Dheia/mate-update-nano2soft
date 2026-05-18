## 2026-05-06 – 2026-05-18

### Adding and Updating the BasPay Payment Gateway (BAS Platform) in the NanoSoft Payment System

Within the software module `Nano.Yepayment`

---

### 1. Introduction

A new payment method named **BasPay** was developed and added to the unified payment system in NanoSoft within the `Nano.Yepayment` module. This addition comes in response to the need to support local payment gateways in Yemen, where the **BAS Platform** is one of the electronic payment platforms that allows accepting payments via a direct API with AES-256-CBC encrypted signature.

After careful examination of the payment flow on the BAS platform, **the gateway was updated** to operate according to the **Two‑Step Payment** pattern, which is the pattern that matches the nature of the platform where:
1. **Step One (process):** A transaction is created and a `trxToken` is obtained, without the amount being deducted yet.
2. **Step Two (complete):** After the customer completes the payment via the BAS app, the transaction status is verified and the order is updated to "paid".

This update made the gateway fully compatible with `OrderManager`, `Checkout2`, and `PaymentResult`, and prevents the order from moving to paid prematurely.

The gateway was developed to the highest security and quality standards, with token caching, use of the unified `HttpHelper` for all API requests, and a mechanism for generating a signature that exactly matches the BAS platform requirements (SHA256 + AES-256-CBC). Additionally, comprehensive test interfaces and dedicated API endpoints for developers and administrators were provided. `RedirectHelper` was also supported to ensure compatibility with mobile apps that need return URLs after payment.

---

### 2. Developed Components

#### 2.1. `BasPay` – Main Payment Gateway Class

- **Path:** `Nano\Yepayment\PaymentTypes\BasPay`
- **Inheritance:** Extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function:** Responsible for authentication with the BAS platform via OAuth2, executing step one (transaction creation), step two (status check and payment confirmation), and generating the encrypted signature.

**Main Functions:**

| Function | Description |
|----------|-------------|
| `process(PaymentResult $result)` | **Step one:** Creates a transaction via `initiateTransaction`, receives `trxToken`, and saves it in the order without changing payment status. Returns success with a confirmation message. |
| `complete(PaymentResult $result)` | **Step two:** Checks transaction status via `checkTransactionStatusByToken`, and if status is `SUCCESS` calls `$result->success()` to update the order to `PaidState`. |
| `checkAndCompletePay(array $options): array` | Public `static` function used in the `baspay/success` route to link the return from the app with the confirmation process. |
| `getAuthToken(): ?string` | Requests an OAuth 2.0 token from BAS via `POST /api/v1/auth/token` (grant_type: client_credentials), and caches it for 3500 seconds. |
| `initiateTransaction($token): array` | Sends `POST /api/v1/merchant/sdk-payment/initiate-transaction` with the request body and signature. |
| `checkTransactionStatusByToken($token, $trxToken): array` | Queries transaction status via `POST /api/v1/merchant/sdk-payment/get-transaction-status` (used internally). |
| `checkTransactionStatus($trxToken): array` | Public function to query a transaction status (for external use). |
| `generateSignature($paramsString): string` | Generates BAS signature: random salt (4 chars), SHA256, then AES-256-CBC encryption using Merchant Key and IV. |
| `encryptAes256Cbc($input, $key, $iv): string` | AES-256-CBC encryption function using OpenSSL. |
| `parseResponse($response): array` | Converts HTTP response to a PHP array. |
| `getCommonHeaders(): array` | Returns required headers for every request (x-client-id, x-app-id, x-sdk-version, ...). |
| `settings(): array` | Defines settings fields in the control panel (URL, Client ID, Client Secret, App ID, Merchant Key, IV, default currency). |
| `encryptedSettings(): array` | Specifies fields that are stored encrypted (`baspay_client_secret`, `baspay_merchant_key`). |

#### 2.2. Partials and Settings

| File | Path | Description |
|------|------|-------------|
| `_info.htm` | `paymenttypes/baspay/_info.htm` | Displays setup instructions and gateway information in the control panel. |
| `_test_info.htm` | `paymenttypes/baspay/_test_info.htm` | Quick test tools within the settings page (authentication, create payment, status check buttons). |
| `baspay-ui.htm` | `views/baspay-ui.htm` | Full interactive web interface for testing all BasPay functions (manual, automatic, statistics, logs). |

#### 2.3. API Endpoints (`routes.php`)

A complete set of routes was added within the `yepayment` group to support testing and monitoring:

| Route | Method | Description |
|-------|--------|-------------|
| `/baspay/test-auth` | POST | Test authentication and obtain Bearer Token. |
| `/baspay/test-create-payment` | POST | Create a new transaction (step one). |
| `/baspay/test-check-status` | POST | Check a transaction's status using `trx_token`. |
| `/baspay/test-full-payment` | POST | Full test (create transaction + check status). |
| `/baspay/stats` | GET | Gateway usage statistics (request count, success rate). |
| `/baspay/test-ui` | GET | Full test interface (HTML). |
| `/baspay/success` | GET | Endpoint to confirm payment after user returns from BAS app. |
| `/baspay/cancel` | GET | Optional endpoint for cancellation redirect. |

All routes (except success/cancel) are protected with `BackendAuth::getUser()` to ensure only administrators can access them.

#### 2.4. `Plugin.php` – Payment Provider Registration

The following line was added in the `registerPaymentProviders()` function within the `allow_yemen_payment` section:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\BasPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\BasPay();
}
```

#### 2.5. Language Files (Translation)

Translation keys were added in `lang/ar/lang.php` and `lang/en/lang.php` for settings, public messages, and errors (such as `payment_success`, `auth_failed`, `order_already_paid`).

---

### 3. Payment Workflow – Two‑Step Pattern

BasPay operates according to a **Two‑Step** flow:

1. **Authentication**  
   `getAuthToken()` is called:
   - Checks for `access_token` in Cache.
   - If not present, sends `POST /api/v1/auth/token` with `client_id`, `client_secret`, and `grant_type=client_credentials`.
   - The token is stored in cache for 3500 seconds.

2. **Step One – Transaction Creation (`process`)**  
   - Checks that the order is not already paid.
   - Builds the request body (`amount`, `currency`, `orderId`, `appId`, `requestTimestamp`).
   - Generates the signature via `generateSignature()`.
   - Sends `POST /api/v1/merchant/sdk-payment/initiate-transaction`.
   - Receives `trxToken` and stores it in `order.payment_first_trans_id` and `order.other_data['baspay']`.
   - **Does not call `$result->success()`**, and the order does not change to `PaidState`. Instead, the gateway returns `successful = true` with a `confirmation_required` message.

3. **Step Two – Payment Confirmation (`complete`)**  
   - Called from the `baspay/success` route after the user returns from the BAS app.
   - Extracts `trxToken` from the order, and communicates with BAS via `POST /api/v1/merchant/sdk-payment/get-transaction-status`.
   - If `trxStatus` equals `SUCCESS` or `COMPLETED`:
     - `$result->success()` is called, which **changes the order status to `PaidState`** and triggers events (like `nano.orders.paymentProcessed`).
   - If `PENDING`: `$result->pending()` is called and the order remains pending.

4. **Optional Return URLs**  
   When `callback_success_url` or `callback_error_url` are provided, they are stored in `other_data`. After confirmation, the developer can use `RedirectHelper` to intelligently redirect the user (Deep Link or web).

---

### 4. Configuration

To enable BasPay, the following settings must be entered in the payment gateway settings interface in the NanoSoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| Setting | Key | Description | Default Value |
|---------|-----|-------------|---------------|
| Base API URL | `baspay_url` | BAS Platform API address | `https://api.basgate.com` |
| Client ID | `baspay_client_id` | Client identifier from BAS platform | - |
| Client Secret | `baspay_client_secret` | Secret key (stored encrypted) | - |
| App ID | `baspay_app_id` | Application ID | - |
| Merchant Key | `baspay_merchant_key` | Merchant key for signing (stored encrypted) | - |
| IV | `baspay_iv` | Initialization vector (16 bytes) | `@@@@&&&&####$$$$` |
| Default Currency | `baspay_default_currency` | Currency used if the order does not specify one | `YER` |

**Security Notes:**
- `baspay_client_secret` and `baspay_merchant_key` are stored encrypted via `encryptedSettings()`.
- These sensitive values are not logged.

---

### 5. Usage Examples

#### 5.1. Initiate Payment – Step One (Transaction Creation)

```php
use Nano\Yepayment\PaymentTypes\BasPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$bas = new BasPay($order, [
    'callback_success_url' => 'myapp://pay/success',   // Optional
]);
$result = new PaymentResult($bas, $order);
$processResult = $bas->process($result);

if ($processResult->successful) {
    // Transaction created and trxToken saved, but payment not yet complete
    $trxToken = $order->payment_first_trans_id;
    // User must be directed to the BAS app to complete payment
}
```

#### 5.2. Confirm Payment – Step Two (Status Verification)

After the customer completes payment in the BAS app, the process can be confirmed as follows:

```php
$result = \Nano\Yepayment\PaymentTypes\BasPay::checkAndCompletePay(['order_id' => 200]);
if ($result['success']) {
    // Payment confirmed and order updated to PaidState
}
```

#### 5.3. Direct Status Query for a Transaction

```php
$bas = new BasPay();
$status = $bas->checkTransactionStatus('bas_trx_a1b2c3d4...');
if ($status['success']) {
    // $status['data'] contains transaction details
}
```

#### 5.4. Using the Full Test Interface

After logging into the control panel as an administrator, open the URL:
```
https://yourdomain.com/api/v1/yepayment/baspay/test-ui
```

The interface allows:
- Creating a new transaction (Initiate).
- Checking status.
- Automatic testing with a specified number of repetitions (up to 10).
- Viewing usage statistics (request count, success rate).
- Local logs for tests.

---

### 6. Handling Redirects (Deeplinks & Callbacks)

Optional support for redirect URLs has been integrated using `RedirectHelper`, to ensure compatibility with mobile apps that need to open a specific screen after payment.

- When `callback_success_url` or `callback_error_url` are passed in payment data, they are stored in `order.other_data['baspay']`.
- After confirming payment via `complete()`, the developer can direct the user to `/baspay/success` (with `order_id`) so that the gateway extracts the return URL and performs intelligent redirection (deeplink or web) with status data.
- The same mechanism applies to `/baspay/cancel`.

---

### 7. Added Value

- **For developers:** A comprehensive model of a **Two‑Step** payment gateway that uses OAuth2 with AES-256-CBC encrypted signature, illustrating how to build gateways that require external confirmation without an SDK. It also demonstrates full integration with `OrderManager`, `PaymentResult`, and `RedirectHelper`.
- **For merchants:** Support for the local BAS platform in Yemen, opening the door to easily accept electronic payments through the merchant's BAS account.
- **For end users:** A reliable two‑step payment experience: confirm the order in the store, then complete payment in the BAS app.

---

### 8. Integration Testing

Several testing layers were provided:

- **Test environment:** The same production URL (`https://api.basgate.com`) can be used with trial credentials from the BAS platform. If a future test environment becomes available, its URL can be entered in the `baspay_url` field.
- **Quick test interface:** From the gateway settings page (partial `_test_info.htm`), you can test authentication, create a transaction, and check status directly.
- **Full test interface:** `/api/v1/yepayment/baspay/test-ui` provides all necessary tools to test the gateway thoroughly (including simulating both steps), with the ability to perform automated repeat testing.
- **Independent API endpoints:** Developers can use tools like Postman or cURL to directly connect to endpoints like `/baspay/test-full-payment`.

**Quick test steps:**
1. Ensure Client ID, Client Secret, App ID, Merchant Key, IV are set in the gateway settings page.
2. Open `/api/v1/yepayment/baspay/test-ui`.
3. Click "Test Authentication" to verify credentials.
4. Enter an order ID and amount, then click "Create New Transaction".
5. A `trxToken` will appear – you can use it in "Check Status" to simulate the second step.

---

### 9. Developer Notes

- **Two‑Step pattern:** `process()` **does not** complete the payment; it only creates the transaction. You must call `complete()` or `checkAndCompletePay` to confirm payment and update the order to `PaidState`.
- **Token caching:** The Bearer Token is cached for 3500 seconds (slightly less than the actual token expiry, estimated at 3600 seconds).
- **Encrypted signature:** The signature mechanism was implemented manually using OpenSSL without relying on an external package, to ensure compatibility with the NanoSoft environment and prevent conflicts.
- **`HttpHelper` usage:** All API requests (JSON, Form) use the unified `HttpHelper`, simplifying error tracking and gateway expansion.
- **`RedirectHelper` support:** Full redirection capability is included, making the gateway suitable for mobile apps that use Deep Links.
- **Extensibility:** Webhook or refund support can be added in the future via additional functions using different BAS endpoints.

---

### 10. Bug Fixes

None – this release is dedicated to adding and updating a new feature only (BasPay).

---

### 11. Development and Testing Period

The BasPay gateway was initially developed as a direct payment, then **updated** to the two‑step pattern during the period from **May 6, 2026** to **May 18, 2026**. This period included writing the core code for the BasPay class and implementing the AES-256-CBC encrypted signature mechanism, creating partial interface files (_info.htm, _test_info.htm) and the full test interface (baspay-ui.htm), as well as writing the necessary API endpoints for testing and monitoring, and modifying `process()` and `complete()` to align with the two‑step flow. Comprehensive integration tests were also conducted to verify the gateway's correct operation with the BAS platform environment, and signature compatibility with the responses received was confirmed.

---

### 12. Related Links

- [BasPay Documentation (Developer Guide)](./docs/BasPay/Docs-BasPay-en.md)
- [SKILL-en.md for creating payment gateways](./docs/BasPay/SKILL-en.md)
- [BAS Platform API Guide](https://basgate.apidog.io)
- [BAS Platform API Guide](./docs/BasPay/external/BAS/README-en.md)
- [BasGate documentation repository on GitHub](https://github.com/basgate/basgate.github.io)
- [Laravel Payment SDK on GitHub](https://github.com/basgate/laravel-payment-sdk)
- [BasPaymentFlutter repository on GitHub](https://github.com/BasPlatform/BasPaymentFlutter.git)
- [bas_php_sdk on GitHub](https://github.com/basgate/bas_php_sdk)
- [bas-laravel-sdk on GitHub](https://github.com/basgate/bas-laravel-sdk)
- [Technical Support Channel](https://nano2soft.com)

---

**This update was prepared by:**  
NanoSoft Development Team – Electronic Payments Department  
**Reviewer:** Dheia Ali, Nano2Soft