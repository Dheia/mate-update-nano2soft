# SKILL: Creating a New Payment Method within the `Nano\Yepayment` Plugin

## 📖 Overview

The `Nano.Yepayment` system allows adding new payment gateways by following a standardized pattern that ensures seamless integration with `Nano.MicroCart` and `Nano.Orders`. This skill enables developers to create any payment provider by following the specified steps and standards, with support for three main types of payment flows.

---

## 🧩 Types of Payment Methods (Payment Flows)

| Type | Description | Example | Key Methods |
|------|-------------|---------|--------------|
| **Redirect Type** | Redirects the user to an external gateway (bank, PayPal, etc.) to complete payment, then returns to `success_url` or `cancel_url`. | `ThawaniPay` | `process()` → `$result->redirect()`<br>`success` route completes payment. |
| **Two-Step Type** | Creates a transaction first (`process`), then requires later confirmation (`complete`) via OTP or code from the customer. | `YottaPay` | `process()` returns `successful = true` and message "Enter OTP".<br>`complete()` confirms the payment. |
| **Direct Type (Immediate)** | Payment is completed immediately via API without need for redirection or additional confirmation. | `QasemiPay` (if confirmation is automatic) | `process()` executes payment and returns `$result->success()` directly. |

> **Note:** A single gateway can combine more than one pattern (e.g., create transaction then redirect, or confirm later). Choose the appropriate pattern according to the gateway's API documentation.

---

## 🏗️ Payment Method Class Structure

**Location:** `plugins/nano/yepayment/paymenttypes/NewPay.php`

```php
<?php namespace Nano\Yepayment\PaymentTypes;

use Nano\MicroCart\Classes\Payments\PaymentProvider;
use Nano\MicroCart\Classes\Payments\PaymentResult;
use Nano\MicroCart\Models\PaymentGatewaySettings;
use Nano\Helpers\Classes\Helpers\HttpHelper;
use Nano\Yepayment\Classes\RedirectHelper; // when redirection is needed
use Config, Request, Validator, Throwable, ApplicationException;

class NewPay extends PaymentProvider
{
    // Required properties
    public $success_url = 'api/v1/yepayment/newpay/success';
    public $cancel_url  = 'api/v1/yepayment/newpay/cancel';
    public $is_test_mod = false;

    // Abstract methods to implement
    public function name(): string { return 'New Payment Gateway'; }
    public function identifier(): string { return 'newpay'; }
    public function validate(): bool { return true; }
    public function settings(): array { /* ... */ }
    public function process(PaymentResult $result): PaymentResult { /* ... */ }
    public function complete(PaymentResult $result): PaymentResult { /* ... */ }
    public function encryptedSettings(): array { return ['newpay_password']; }

    // Helper methods (as needed)
    public function defineValidationRules(): array { /* ... */ }
    public function getFieldNames(): array { /* ... */ }
    private function getAuthToken(): ?string { /* ... */ }
    private function createPayment($token): array { /* ... */ }
    private function confirmPayment($token, $id, $code): array { /* ... */ }
    public function checkTransactionStatus($refId): array { /* ... */ }
    private function getApiUrl(string $type): string { /* ... */ }
    private function parseResponse($response): array { /* ... */ }
}
```

---

## 📝 Detailed Creation Steps

### 1. Create the Main Class

- Create a new PHP file in `paymenttypes/`.
- Ensure the class extends `PaymentProvider`.
- Define `$success_url` and `$cancel_url` according to the route group used (`yepayment` or `ompayment`).
- Define `$is_test_mod` to switch between test and production environments.

### 2. Implement Abstract Methods

#### `name(): string`
- Return the trade name of the gateway as it will appear to the user.

#### `identifier(): string`
- Return a unique identifier (lowercase Latin letters, no spaces). Used for storing records and identifying the gateway.

#### `validate(): bool`
- Can return `true` and use `defineValidationRules()` separately, or implement validation manually.

#### `settings(): array`
- Define the settings fields that will appear in the control panel (URL, API keys, username, etc.).
- Use `type => 'partial'` to include `_info.htm` and `_test_info.htm` files.
- Use `type => 'password'` for sensitive fields, and add them to `encryptedSettings()`.

#### `process(PaymentResult $result): PaymentResult`
This is the core payment logic. Typical steps:

```php
public function process(PaymentResult $result): PaymentResult
{
    try {
        // 1. Validate input data (if any)
        $validator = Validator::make($this->data, $this->defineValidationRules());
        if ($validator->fails()) throw new ValidationException($validator);

        // 2. Ensure order is not already paid
        if ($this->order->payment_state == PaidState::class) {
            $result->successful = false;
            $result->message = trans('nano.yepayment::lang.public.newpay.errors.order_already_paid');
            return $result;
        }

        // 3. Obtain authentication token (if needed)
        $token = $this->getAuthToken();
        if (!$token) throw new ApplicationException('Auth failed');

        // 4. Create payment transaction via API
        $paymentResult = $this->createPayment($token);
        if (!$paymentResult['success']) throw new ApplicationException($paymentResult['message']);

        // 5. Save transaction data in order->other_data and payment_first_trans_id
        $this->order->payment_first_trans_id = $paymentResult['transaction_id'];
        $this->order->payment_trans_id = $paymentResult['ref_no'] ?? null;
        $other = $this->order->other_data;
        $other['newpay'] = [ /* additional data */ ];
        $this->order->other_data = $other;
        $this->order->save();

        // 6. Determine payment path based on gateway type
        if (isset($paymentResult['redirect_url'])) {
            return $result->redirect($paymentResult['redirect_url']);
        } elseif ($paymentResult['requires_confirmation'] ?? false) {
            $result->successful = true;
            $result->message = 'Please enter confirmation code';
            $result->api_data = $paymentResult;
            return $result;
        } else {
            // Immediate payment
            $this->order->payment_state = PaidState::class;
            $this->order->save();
            return $result->success($paymentResult, null);
        }
    } catch (Throwable $e) {
        return $result->fail([], $e);
    }
}
```

#### `complete(PaymentResult $result): PaymentResult`
- Used for **Two-Step** type or when additional confirmation is needed after user returns from an external gateway.
- Should retrieve `transaction_id` from `payment_first_trans_id` or `other_data`.
- Request payment confirmation from API, then update order status to `PaidState` via `$result->success()`.

### 3. Using `HttpHelper` for API Requests

All HTTP requests (GET, POST, PUT, PATCH, DELETE) must use the class `Nano\Helpers\Classes\Helpers\HttpHelper`. This class provides a unified interface for Guzzle and supports JSON, Form Data, and Multipart.

**Import the class:**
```php
use Nano\Helpers\Classes\Helpers\HttpHelper;
```

**Practical examples from existing gateways:**

#### a. JSON request (e.g., YottaPay – create transaction):
```php
$response = HttpHelper::sendJson([
    'method' => 'POST',
    'url' => $this->getApiUrl('payment'),
    'json' => $payload,
    'options' => [
        'headers' => [
            'Authorization' => 'Bearer ' . $token,
            'Content-Type' => 'application/json',
        ],
        'timeout' => 30,
    ],
]);
```

#### b. Form Data request (e.g., QasemiPay – get token):
```php
$response = HttpHelper::sendForm([
    'method' => 'POST',
    'url' => $this->getApiUrl('token'),
    'form_params' => [
        'client_id' => $clientId,
        'client_secret' => $clientSecret,
        'grant_type' => 'password',
        // ...
    ],
    'options' => [
        'headers' => [
            'Content-Type' => 'application/x-www-form-urlencoded',
        ],
    ],
]);
```

#### c. GET request with Query Parameters:
```php
$response = HttpHelper::get([
    'url' => $this->getApiUrl('check_status') . '/' . $refId,
    'options' => [
        'query' => ['param' => 'value'],
        'headers' => ['Accept' => 'application/json'],
    ],
]);
```

#### d. PATCH request (e.g., YottaPay – confirm payment):
```php
$response = HttpHelper::sendJson([
    'method' => 'PATCH',
    'url' => $this->getApiUrl('payment'),
    'json' => ['id' => $id, 'otp' => $otp],
    'options' => [...],
]);
```

**Response handling:**  
After executing the request, use the helper function `parseResponse()` to convert the response to an array (as in `YottaPay`, `QasemiPay`, `ThawaniPay`):

```php
private function parseResponse($response): array
{
    $body = $response->getBody()->getContents();
    $contentType = $response->getHeaderLine('content-type');
    if (strpos($contentType, 'application/json') !== false) {
        return json_decode($body, true) ?: [];
    }
    return ['raw_body' => $body, 'status_code' => $response->getStatusCode()];
}
```

### 4. Token Management

- If the gateway requires a token (OAuth, JWT), create a `getAuthToken()` function.
- It is recommended to store the token in Cache to avoid repeated authentication requests:

```php
use Illuminate\Support\Facades\Cache;

private function getAuthToken(): ?string
{
    return Cache::remember('newpay_token', 3500, function () {
        // Request token from API using HttpHelper
        $response = HttpHelper::sendForm([...]);
        return $response['access_token'] ?? null;
    });
}
```

### 5. Handling Redirection and `RedirectHelper`

For **Redirect** type gateways (like `ThawaniPay`), you must use the class `Nano\Yepayment\Classes\RedirectHelper` to redirect to applications or websites after payment. This class:

- Extracts `callback_success_url` and `callback_error_url` from `order->other_data`.
- Supports Deep Links (e.g., `myapp://`) and regular web URLs.
- Handles large data by storing it in Cache using a token.
- Ensures security by checking allowed domains and filtering sensitive data.

**Import the class:**
```php
use Nano\Yepayment\Classes\RedirectHelper;
```

**Practical examples from `routes.php` (ThawaniPay):**

#### a. Extract redirect URL from order data:
```php
$callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, ['callback_success_url', 'thawani.callback_success_url']);
```

#### b. Redirect to app or web:
```php
return RedirectHelper::redirectToApp($callbackUrl, $data, $forceJson, $queryParams, $deepLinkSchemes, $is_force_deep_list);
```
- `$callbackUrl`: The extracted URL (may be `https://...` or `myapp://...`).
- `$data`: Data to send (payment status, order ID, etc.).
- `$forceJson`: If `true`, returns JSON instead of redirect.
- `$queryParams`: Array of field names to add as query parameters (e.g., `['order_id', 'payment_method_id']`).
- `$deepLinkSchemes`: List of allowed schemes for apps (e.g., `['myapp', 'app', '*']`). `'*'` means any scheme not http/https.
- `$is_force_deep_list`: If `true`, forces using only the specified list.

#### c. Alternative cases when no URL is available:
```php
if (!$callbackUrl) {
    return Response::make('Payment successful. You can return to the app.', 200);
}
```

#### d. General function to verify and complete payment (like `checkAndCompletePay` in ThawaniPay):
You can create a `static` function in the gateway class that internally uses `RedirectHelper` to unify the logic.

**Note:** You must add `success` and `cancel` routes in `routes.php` to receive the return from the external gateway, then use `RedirectHelper` inside them as in `thawanipay/success`.

### 6. Handling `PaymentResult`

| Method | Effect |
|--------|--------|
| `$result->success($data, $response)` | Sets `successful = true`, logs successful payment, updates order status to `PaidState`. |
| `$result->fail($data, $response)` | Sets `successful = false`, logs failed payment, updates status to `FailedState`. |
| `$result->redirect($url)` | Requests `PaymentRedirector` to redirect the user to `$url`. |
| `$result->pending($data, $response)` | Sets `successful = true` but order status remains `PendingState` (rarely used). |

---

## 🧩 Creating Partial Files

### 1. `_info.htm`
**Path:** `paymenttypes/newpay/_info.htm`  
Displays introductory information about the gateway in the settings page.

```html
<div class="callout callout-info">
    <h4>New Payment Gateway</h4>
    <p>Integrated payment gateway that accepts online payments.</p>
    <hr>
    <strong>Setup Requirements:</strong>
    <ol>
        <li>Obtain API Key from the payment gateway.</li>
        <li>Specify the Base API URL (Live/Sandbox).</li>
    </ol>
</div>
```

### 2. `_test_info.htm`
**Path:** `paymenttypes/newpay/_test_info.htm`  
Provides a quick testing tool within the settings page.

**Important Notes:**
- All IDs and function names must be unique using the gateway prefix (e.g., `newpay-`) to avoid conflicts with other gateways.
- Use `const newpayApiBaseUrl = '/api/v1/yepayment';` (or `/ompayment` depending on the group).
- Buttons listen to events via `addEventListener`.
- Display results in a dedicated `div`.

**Simplified template:**
```html
<div id="newpay-test-widget">
    <div class="row">
        <div class="col-md-6">
            <div class="form-group">
                <label>Order ID:</label>
                <input type="text" id="newpay-test-order-id" class="form-control" value="200">
            </div>
            <div class="form-group">
                <label>Amount:</label>
                <input type="number" id="newpay-test-amount" class="form-control" value="100">
            </div>
        </div>
        <div class="col-md-6">
            <div class="form-group">
                <label>Confirmation Code:</label>
                <input type="text" id="newpay-test-code" class="form-control" value="1234">
            </div>
        </div>
    </div>
    <div class="text-center">
        <button id="newpay-btn-auth" class="btn btn-info">Test Auth</button>
        <button id="newpay-btn-create" class="btn btn-primary">Create Transaction</button>
        <button id="newpay-btn-confirm" class="btn btn-success">Confirm Payment</button>
        <button id="newpay-btn-status" class="btn btn-warning">Check Status</button>
    </div>
    <div id="newpay-test-results" style="display:none; margin-top:15px;">
        <div class="alert" id="newpay-test-alert"></div>
        <pre id="newpay-test-details"></pre>
    </div>
</div>
<script>
    const newpayApi = '/api/v1/yepayment';
    document.getElementById('newpay-btn-auth')?.addEventListener('click', () => {
        fetch(`${newpayApi}/newpay/test-auth`, { method: 'POST' })
            .then(r => r.json())
            .then(d => showResult(d));
    });
    function showResult(data) {
        const div = document.getElementById('newpay-test-results');
        div.style.display = 'block';
        document.getElementById('newpay-test-details').textContent = JSON.stringify(data, null, 2);
    }
    // Remaining functions follow the same pattern with newpay- prefix
</script>
```

---

## 🖥️ Creating the Integrated Test Page (UI)

**Path:** `views/newpay-ui.htm`

- Use the same structure as `yottapay-ui.htm` or `thawanipay-ui.htm`.
- Change all `yottapay` to `newpay` in IDs, function names, and endpoints.
- Ensure there are tabs (manual, automatic, statistics, logs).
- Add local logs using `localStorage` with a unique key (`newpay_test_logs`).

**Basic snippets:**

- Display current settings: `{{ apiBaseUrl }}`, `{{ settings.url }}`, etc.
- Input form (order ID, amount, purchase code, etc.).
- Buttons: Create transaction, Confirm, Query, Full test, Automated test.
- Display results in JSON format.
- Statistics (order count, success rate, recent logs) via `/stats` endpoint.

---

## 🛣️ Adding Routes in `routes.php`

Routes must be added within the appropriate main group (`yepayment` or `ompayment`). Required routes:

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/newpay/test-auth` | Test authentication |
| `POST` | `/newpay/test-create-payment` | Create a test transaction |
| `POST` | `/newpay/test-confirm-payment` | Confirm transaction (if needed) |
| `GET` | `/newpay/test-check-status` | Query transaction status |
| `POST` | `/newpay/test-full-payment` | Full test |
| `GET` | `/newpay/stats` | Gateway statistics |
| `GET` | `/newpay/test-ui` | Display test page |
| `GET` | `/newpay/success` | (for redirect type) Success route |
| `GET` | `/newpay/cancel` | (for redirect type) Cancel route |

**Important Notes:**

- All test routes are protected with `if(!BackendAuth::getUser())` to prevent unauthorized access.
- Use `HttpHelper` inside routes (if needed) or call gateway methods directly.
- In `success`/`cancel` routes, use `RedirectHelper` to redirect to the app (as in `thawanipay/success`).

**Example `success` route:**
```php
Route::get('newpay/success', function () {
    $options = Input::get();
    $result = NewPay::checkAndCompletePay($options); // general verification function
    $order_id = $result['data']['order_id'] ?? null;
    if ($order_id) {
        $callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, ['callback_success_url', 'newpay.callback_success_url']);
        if ($callbackUrl) {
            return RedirectHelper::redirectToApp($callbackUrl, $result, false, ['order_id', 'payment_method_id'], ['*'], false);
        }
    }
    return response()->json($result);
});
```

**Example `test-create-payment` route:**
```php
Route::post('newpay/test-create-payment', function (Request $request) {
    if (!BackendAuth::getUser()) return Response::make('Access Denied', 404);
    $order = Order::find($request->input('order_id'));
    if (!$order) return response()->json(['success' => false, 'message' => 'Order not found']);
    $newpay = new \Nano\Yepayment\PaymentTypes\NewPay($order, $request->all());
    $result = new PaymentResult($newpay, $order);
    $processResult = $newpay->process($result);
    return response()->json(['success' => $processResult->successful, 'data' => $processResult->api_data]);
});
```

---

## 📦 Registering the Gateway in `Plugin.php`

```php
public function registerPaymentProviders()
{
    $providers = [];
    // ... other gateways
    if (class_exists(\Nano\Yepayment\PaymentTypes\NewPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\NewPay();
    }
    return $providers;
}
```

Place it in the appropriate geographic section (`allow_yemen_payment` or `allow_oman_payment`).

---

## 🌐 Adding Translation Keys

**Files:** `lang/ar/lang.php` and `lang/en/lang.php`

```php
return [
    'payment_gateway_settings' => [
        'newpay' => [
            'url' => 'Base API URL',
            'test_url' => 'Test API URL',
            'api_key' => 'API Key',
            'secret_key' => 'Secret Key',
            'username' => 'Username',
            'password' => 'Password',
            'default_currency' => 'Default Currency',
        ],
    ],
    'public' => [
        'newpay' => [
            'payment_success' => 'Payment successful',
            'confirmation_required' => 'Please enter confirmation code',
            'transaction_created' => 'Transaction created',
            'default_note' => 'Confirm payment',
            'yes' => 'Yes',
            'no' => 'No',
            'errors' => [
                'auth_failed' => 'Authentication failed',
                'order_already_paid' => 'Order already paid',
                'missing_credentials' => 'Incomplete credentials',
                'payment_creation_failed' => 'Payment transaction creation failed',
                'payment_confirmation_failed' => 'Payment confirmation failed',
                'incomplete_payment_data' => 'Incomplete payment data',
            ],
        ],
    ],
];
```

---

## 🧪 Additional Tips for Professional Gateway Development

### 1. Mandatory Use of `HttpHelper` and `RedirectHelper`
- **All API requests** must be made via `HttpHelper`, not direct `curl` or un-wrapped Guzzle.
- **All redirections after payment** (especially for Redirect type gateways) must use `RedirectHelper` to ensure compatibility with apps and web.

### 2. Webhook Support
- Add a route to receive notifications from the gateway (POST).
- Verify signature if present.
- Update order status automatically.

### 3. Retry Failed Requests
- Add a `for` loop with `try-catch` to execute the request multiple times before failing.

### 4. Idempotency Support
- Send a unique `Idempotency-Key` (e.g., `order_id_uuid`) in create transaction requests.

### 5. General `checkTransactionStatus` Function
- Can be called from `stats` or from the UI to display transaction status.

### 6. `isAvailable()` Function to Validate Settings
```php
public function isAvailable(): bool
{
    try {
        $token = $this->getAuthToken();
        return !empty($token);
    } catch (\Exception $e) {
        return false;
    }
}
```

### 7. Detailed Error Logging
- Use `trace_log()` or `logger()` to log requests and responses when errors occur.

### 8. `checkAndCompletePay` Function (for Redirect Type)
Add a static function in the gateway class to unify the logic for verification and completing payment after returning from the external gateway. You can leverage `RedirectHelper` inside it.

---

## ✅ Checklist for Creating a New Gateway

- [ ] Create `XxxPay.php` class extending `PaymentProvider`.
- [ ] Implement methods: `name`, `identifier`, `validate`, `settings`, `process`, `complete`.
- [ ] Add helper methods: `getAuthToken`, `createPayment`, `confirmPayment`, `checkTransactionStatus`, `getApiUrl`, `parseResponse`.
- [ ] Define `defineValidationRules` and `getFieldNames`.
- [ ] Add properties `$success_url`, `$cancel_url`, `$is_test_mod`.
- [ ] Create `_info.htm` and `_test_info.htm` (with unique IDs).
- [ ] Create `views/xxxpay-ui.htm` (full test page).
- [ ] Add test routes in `routes.php` (with `BackendAuth` protection).
- [ ] Add `success`/`cancel` routes (for redirect type) using `RedirectHelper`.
- [ ] Register the class in `Plugin.php` under the appropriate section.
- [ ] Add translation keys in `lang/ar/lang.php` and `lang/en/lang.php`.
- [ ] Test the gateway via `/api/v1/yepayment/xxxpay/test-ui`.
- [ ] Document any special behavior (Webhooks, etc.) in `_info.htm` or a separate file.

---

## 📚 Reference Examples (from provided code)

- **YottaPay**: Example of Two-Step type (OTP).  
  Demonstrates using `HttpHelper` for JSON and PATCH requests, and token storage.
- **QasemiPay**: Example of direct type with later confirmation (concurrencyStamp).  
  Demonstrates using `HttpHelper::sendForm` to obtain OAuth token.
- **ThawaniPay**: Example of Redirect type with redirection to external gateway.  
  Demonstrates using `RedirectHelper` in `success`/`cancel` routes, and a `checkAndCompletePay` function.

---

## 🏁 Conclusion

By following this guide, you can create any new payment method in the `Nano.Yepayment` system regardless of the complexity of its API. Use the appropriate pattern (Redirect, Two-Step, Direct) according to the payment flow, and do not forget to:

- Use `HttpHelper` for all API requests.
- Use `RedirectHelper` for all redirections after payment.
- Provide the necessary testing tools (`_test_info.htm` and `xxxpay-ui.htm`) to facilitate development and maintenance.

🚀 **Start adding your new gateway now!**

---
