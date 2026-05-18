## 2026-05-6 – 2026-05-15

### Adding BasPay Payment Gateway (BAS Platform) to the Payment System in Nanosoft

Within the `Nano.Yepayment` module

---

### 1. Introduction

A new payment method named **BasPay** has been developed and added to the unified payment system in Nanosoft within the `Nano.Yepayment` module. This addition comes in response to the need to support local payment gateways in Yemen, where **BAS Platform** is an electronic payment platform that allows accepting payments via a direct API with AES-256-CBC encrypted signature.

The BasPay gateway uses a **Direct / Immediate Payment** method, where the amount is deducted immediately upon calling the payment API, without needing to redirect the user to an external page or wait for additional confirmation (unlike ThawaniPay which requires redirection, or YottaPay/QasemiPay which require confirmation). This method is ideal for immediate payments via applications or e-commerce sites where the merchant wants to complete the transaction in the background.

The gateway was developed according to the highest security and quality standards, with token caching, use of a unified `HttpHelper` for all API requests, and a signature generation mechanism perfectly matching BAS Platform requirements (SHA256 + AES-256-CBC), in addition to integrated test interfaces and dedicated API endpoints for developers and administrators. `RedirectHelper` has also been optionally supported to ensure the gateway is compatible with mobile applications that need return URLs after payment.

---

### 2. Developed Components

#### 2.1. `BasPay` – Main Payment Gateway Class

- **Path:** `Nano\Yepayment\PaymentTypes\BasPay`
- **Inheritance:** extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function:** Responsible for authentication with BAS Platform via OAuth2, executing direct payment, checking transaction status, and generating encrypted signature.

**Main Methods:**

| Method | Description |
|--------|-------------|
| `process(PaymentResult $result)` | Execute direct payment: get token, send transaction creation request with signature, save `trxToken`, and update order to paid. |
| `complete(PaymentResult $result)` | Not used in this type (immediate payment), but exists for interface compatibility. |
| `getAuthToken(): ?string` | Request OAuth 2.0 token from BAS via `POST /api/v1/auth/token` (grant_type: client_credentials), caching it for 3500 seconds. |
| `createPayment($token): array` | Send `POST /api/v1/merchant/sdk-payment/initiate-transaction` with request body and signature. |
| `checkTransactionStatus($trxToken): array` | Query transaction status via `POST /api/v1/merchant/sdk-payment/get-transaction-status`. |
| `generateSignature($paramsString): string` | Generate BAS signature: random 4-character salt, SHA256, then AES-256-CBC encryption using Merchant Key and IV. |
| `encryptAes256Cbc($input, $key, $iv): string` | AES-256-CBC encryption function using OpenSSL. |
| `parseResponse($response): array` | Convert HTTP response to PHP array. |
| `getCommonHeaders(): array` | Return required headers for each request (x-client-id, x-app-id, x-sdk-version, ...). |
| `settings(): array` | Define settings fields in the control panel (URL, Client ID, Client Secret, App ID, Merchant Key, IV, default currency). |
| `encryptedSettings(): array` | Specify fields stored encrypted (`baspay_client_secret`, `baspay_merchant_key`). |

#### 2.2. Partial Files (Partials) and Settings

| File | Path | Description |
|------|------|-------------|
| `_info.htm` | `paymenttypes/baspay/_info.htm` | Display setup guidance and gateway information in the control panel. |
| `_test_info.htm` | `paymenttypes/baspay/_test_info.htm` | Quick test tools within the settings page (buttons for auth, create payment, check status). |
| `baspay-ui.htm` | `views/baspay-ui.htm` | Integrated interactive web interface to test all BasPay functions (manual, automatic, statistics, logs). |

#### 2.3. API Endpoints (`routes.php`)

A complete set of routes has been added within the `yepayment` group to support testing and monitoring:

| Path | Method | Description |
|------|--------|-------------|
| `/baspay/test-auth` | POST | Test authentication and obtain Bearer Token. |
| `/baspay/test-create-payment` | POST | Create an immediate payment for a specified order. |
| `/baspay/test-check-status` | POST | Check transaction status using `trx_token`. |
| `/baspay/test-full-payment` | POST | Comprehensive test (create payment + check status). |
| `/baspay/stats` | GET | Gateway usage statistics (number of requests, success rate). |
| `/baspay/test-ui` | GET | Integrated test interface (HTML). |
| `/baspay/success` | GET | Optional endpoint for redirect after payment (for applications). |
| `/baspay/cancel` | GET | Optional endpoint for redirect on cancellation. |

All routes (except success/cancel) are protected by `BackendAuth::getUser()` to ensure only admin access.

#### 2.4. `Plugin.php` – Registering the Payment Provider

The following line has been added in the `registerPaymentProviders()` function within the `allow_yemen_payment` section:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\BasPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\BasPay();
}
```

#### 2.5. Language Files (Translation)

Translation keys have been added in `lang/ar/lang.php` and `lang/en/lang.php` for settings, general messages, and errors (like `payment_success`, `auth_failed`, `order_already_paid`).

---

### 3. Payment Workflow

BasPay relies on a **Direct (Immediate)** flow without redirection or extra confirmation:

1. **Authentication**  
   `getAuthToken()` is called:
   - Check for `access_token` in Cache.
   - If not present, send `POST /api/v1/auth/token` with `client_id`, `client_secret`, and `grant_type=client_credentials`.
   - Token is cached for 3500 seconds.

2. **Payment Execution (Initiate Transaction)**  
   When `process()` is called:
   - Verify the order is not already paid.
   - Build request body (`amount`, `currency`, `orderId`, `appId`, `requestTimestamp`).
   - Generate signature via `generateSignature()`.
   - Send `POST /api/v1/merchant/sdk-payment/initiate-transaction` with custom headers (`x-client-id`, `x-app-id`, `Authorization`).
   - Receive response: if `status=1` and `code='1111'`, extract `trxToken` from `body.trxToken`.
   - Save `trxToken` in `order.payment_first_trans_id` and `order.payment_trans_id`.
   - Store additional data in `order.other_data['baspay']` (including optional return URLs).
   - Update order status to `PaidState` via `$result->success()`.

3. **Status Check (Check Status)**  
   `checkTransactionStatus($trxToken)` can be called to query the transaction via `POST /api/v1/merchant/sdk-payment/get-transaction-status`.

4. **Optional Return URLs**  
   If `callback_success_url` or `callback_error_url` are passed (e.g., from a mobile app), they are stored in `other_data`. The developer can use the `/baspay/success` and `/baspay/cancel` paths after payment to redirect the user using `RedirectHelper`, making the gateway fully compatible with mobile applications.

---

### 4. Configuration

To activate BasPay, the following settings must be entered in the payment gateway settings interface in the Nanosoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| Setting | Key | Description | Default Value |
|---------|-----|-------------|---------------|
| Base API URL | `baspay_url` | BAS Platform API address | `https://api.basgate.com` |
| Client ID | `baspay_client_id` | Client identifier from BAS Platform | - |
| Client Secret | `baspay_client_secret` | Secret key (stored encrypted) | - |
| App ID | `baspay_app_id` | Application identifier | - |
| Merchant Key | `baspay_merchant_key` | Merchant key for signature (stored encrypted) | - |
| IV | `baspay_iv` | Initialization vector (16 bytes) | `@@@@&&&&####$$$$` |
| Default Currency | `baspay_default_currency` | Currency used if order does not specify one | `YER` |

**Security Notes:**
- `baspay_client_secret` and `baspay_merchant_key` are stored encrypted via `encryptedSettings()`.
- These sensitive values are not logged.

---

### 5. Usage Examples

#### 5.1. Create a Payment for an Existing Order (within a Nanosoft Application)

```php
use Nano\Yepayment\PaymentTypes\BasPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$bas = new BasPay($order, [
    'amount'   => 100,
    'currency' => 'YER',
    'callback_success_url' => 'myapp://pay/success',   // optional
]);
$result = new PaymentResult($bas, $order);
$processResult = $bas->process($result);

if ($processResult->successful) {
    $trxToken = $order->payment_first_trans_id;
    // Payment succeeded
}
```

#### 5.2. Query Transaction Status

```php
$bas = new BasPay();
$status = $bas->checkTransactionStatus('bas_trx_a1b2c3d4...');
if ($status['success']) {
    // $status['data'] contains transaction details
}
```

#### 5.3. Using the Integrated Test Interface

After logging into the control panel as an admin, open the link:
```
https://yourdomain.com/api/v1/yepayment/baspay/test-ui
```

The interface allows:
- Creating an immediate payment.
- Checking status.
- Automated testing with a specified number of times (up to 10).
- Displaying usage statistics (number of requests, success rate).
- Local test logs.

---

### 6. Handling Redirects (Deeplinks & Callbacks)

Although BasPay is of the direct payment type, **optional** support for redirect URLs has been integrated using `RedirectHelper` (as in ThawaniPay), to ensure compatibility with mobile apps that need to open a specific screen after payment.

- When `callback_success_url` or `callback_error_url` is passed in payment data, they are stored in `order.other_data['baspay']`.
- After executing `process()`, the developer can direct the user to `/baspay/success` (with `order_id`) so the gateway extracts the return URL and performs a smart redirect (deeplink or web) passing status data.
- The same mechanism applies to `/baspay/cancel`.

This makes BasPay fully compatible with various usage patterns without sacrificing the simplicity of direct payment.

---

### 7. Added Value

- **For Developers:** An additional model of a **Direct** payment gateway using OAuth2 with AES-256-CBC encrypted signature, enriching the library of payment patterns in `SKILL-en.md`. It also illustrates how to integrate an external signing mechanism (without an SDK package) within `PaymentProvider`.
- **For Merchants:** Support for the local BAS platform in Yemen, opening the door to accepting electronic payments easily through the merchant's BAS account.
- **For End Users:** A fast and invisible payment experience (executed in the background) without redirection, reducing the payment abandonment rate.

---

### 8. Integration Testing

Several testing layers have been provided:

- **Test Environment:** The same production URL (`https://api.basgate.com`) can be used with trial credentials from BAS Platform. If a future test environment becomes available, its URL can be entered in the `baspay_url` field.
- **Quick Test Interface:** Through the gateway settings page (partial file `_test_info.htm`), authentication, payment creation, and status checking can be tested directly.
- **Integrated Test Interface:** `/api/v1/yepayment/baspay/test-ui` provides all necessary tools to test the gateway comprehensively, with the ability for automated iterative testing.
- **Standalone API Endpoints:** Developers can use tools like Postman or cURL to directly connect to endpoints such as `/baspay/test-full-payment`.

**Quick Test Steps:**
1. Ensure Client ID, Client Secret, App ID, Merchant Key, IV are set in the gateway settings page.
2. Open `/api/v1/yepayment/baspay/test-ui`.
3. Click "Test Authentication" to verify data correctness.
4. Enter an order number and amount, then click "Create New Payment".
5. Copy the `trxToken` and use it in the status check field.

---

### 9. Developer Notes

- **BasPay does not rely on redirection:** The `complete()` function is not actually used, but exists to fulfill contract requirements.
- **Token Caching:** Bearer Token is cached for 3500 seconds (less than the actual token validity estimated at 3600 seconds).
- **Encrypted Signature:** The signature mechanism was implemented manually using OpenSSL without relying on an external package, to ensure compatibility with the Nanosoft environment and avoid conflicts.
- **Use of `HttpHelper`:** All API requests (JSON, Form) use the unified `HttpHelper`, making error tracking and gateway expansion easier.
- **Optional Support for `RedirectHelper`:** Redirection capability is included without being mandatory, making the gateway suitable for both background (Backend) use and scenarios requiring user interaction.
- **Extensibility:** Support for Webhook or refunds can be added in the future via additional methods using different BAS endpoints.

---

### 10. Bug Fixes

None – this release is dedicated to adding a new feature only (BasPay).

---

### 11. Development and Testing Period

The BasPay gateway was fully developed and integrated into the `Nano.Yepayment` system during the period from **May 6, 2026** to **May 15, 2026**. This period included writing the core code for the BasPay class and implementing the AES-256-CBC encrypted signature mechanism, creating the partial interface files (_info.htm, _test_info.htm) and the integrated test interface (baspay-ui.htm), as well as writing the necessary API endpoints for testing and monitoring. Comprehensive integration tests were also conducted to ensure the gateway works correctly with the BAS Platform environment, and signature compatibility with received responses was verified.

---

### 12. Related Links

- [BasPay Documentation (Developer Guide)](./docs/BasPay/Docs-BasPay-en.md)
- [SKILL-en.md Document for Creating Payment Gateways](./docs/BasPay/SKILL-en.md)
- [BAS Platform API Guide](./docs/BasPay/external/BAS/README-en.md)
- [Technical Support Channel](https://nano2soft.com)

---

**This update was prepared by:**  
Nanosoft Development Team – Electronic Payments Department  
**Reviewer:** Dheia Ali, Nano2Soft
