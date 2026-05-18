# SKILL: Creating a New Payment Method within the `Nano\Yepayment` Plugin

## 📖 Overview

The `Nano.Yepayment` system allows adding new payment gateways following a unified pattern that ensures seamless integration with `Nano.MicroCart` and `Nano.Orders`. This skill enables a developer to create any payment provider by following the specified steps and standards, with support for three main types of payment flows.

---

## 🧩 Payment Flow Types

| Type | Description | Example | Main Functions |
|------|-------------|---------|----------------|
| **Redirect Type** | Redirects the user to an external gateway (bank, PayPal, etc.) to complete payment, then returns to `success_url` or `cancel_url`. | `ThawaniPay` | `process()` → `$result->redirect()`<br>`success` route completes payment. |
| **Two-Step Type** | Creates a transaction first (process), then requires a later confirmation (complete) via OTP or a code from the customer. | `YottaPay` | `process()` returns `successful = true` and message "Enter OTP".<br>`complete()` confirms payment. |
| **Direct Type (Instant)** | Payment is done instantly via API without the need for redirect or additional confirmation. | `QasemiPay` (if confirmation is automatic) | `process()` executes payment and returns `$result->success()` directly. |

> **Note:** A single gateway can combine more than one pattern (e.g., create a transaction then redirect, or later confirmation). Choose the appropriate pattern based on the gateway's API documentation.

---

## 🏗️ Payment Method Class Structure

**Location:** `plugins/nano/yepayment/paymenttypes/NewPay.php`

```php
<?php namespace Nano\Yepayment\PaymentTypes;

use Nano\MicroCart\Classes\Payments\PaymentProvider;
use Nano\MicroCart\Classes\Payments\PaymentResult;
use Nano\MicroCart\Models\PaymentGatewaySettings;
use Nano\Helpers\Classes\Helpers\HttpHelper;
use Nano\Yepayment\Classes\RedirectHelper; // When redirection is needed
use Config, Request, Validator, Throwable, ApplicationException;

class NewPay extends PaymentProvider
{
    // Mandatory properties
    public $success_url = 'api/v1/yepayment/newpay/success';
    public $cancel_url  = 'api/v1/yepayment/newpay/cancel';
    public $is_test_mod = false;

    // Abstract methods that must be implemented
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
- Define `$success_url` and `$cancel_url` based on the route group used (`yepayment` or `ompayment`).
- Define `$is_test_mod` to switch between test and production environments.

### 2. Implement the Abstract Methods

#### `name(): string`
- Return the commercial name of the gateway as it will appear to the user.

#### `identifier(): string`
- Return a unique identifier (lowercase Latin letters, no spaces). Used for storing records and distinguishing the gateway.

#### `validate(): bool`
- Can return `true` and use `defineValidationRules()` separately, or implement validation manually.

#### `settings(): array`
- Define the settings fields that will appear in the control panel (URL, API keys, username, etc.).
- Use `type => 'partial'` to include `_info.htm` and `_test_info.htm` files.
- Use `type => 'password'` for sensitive fields, and add them to `encryptedSettings()`.

#### `process(PaymentResult $result): PaymentResult`
This is the heart of the payment logic. Typical steps:

```php
public function process(PaymentResult $result): PaymentResult
{
    try {
        // 1. Validate input data (if any)
        $validator = Validator::make($this->data, $this->defineValidationRules());
        if ($validator->fails()) throw new ValidationException($validator);

        // 2. Ensure the order is not already paid
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
        $other['newpay'] = [ /* Additional data */ ];
        $this->order->other_data = $other;
        $this->order->save();

        // 6. Determine payment path based on gateway type
        if (isset($paymentResult['redirect_url'])) {
            return $result->redirect($paymentResult['redirect_url']);
        } elseif ($paymentResult['requires_confirmation'] ?? false) {
            $result->successful = true;
            $result->message = 'Please enter the confirmation code';
            $result->api_data = $paymentResult;
            return $result;
        } else {
            // Instant payment
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
- Used for **Two‑Step** type or when additional confirmation is needed after the user returns from an external gateway.
- Must retrieve `transaction_id` from `payment_first_trans_id` or from `other_data`.
- Request payment confirmation from the API, then update the order status to `PaidState` via `$result->success()`.

### 3. Using `HttpHelper` for API Requests

The `Nano\Helpers\Classes\Helpers\HttpHelper` class must be used for all HTTP requests (GET, POST, PUT, PATCH, DELETE). This class provides a unified interface for Guzzle and supports JSON, Form Data, Multipart.

**Import the class:**
```php
use Nano\Helpers\Classes\Helpers\HttpHelper;
```

**Practical examples from existing gateways:**

#### A. JSON Request (like YottaPay – create transaction):
```php
$response = HttpHelper::sendJson([
    'method' => 'POST',
    'url' => $this->getApiUrl('payment'),
    'json' => $payload,
    'options' => [
        'headers' => [
            'Authorization' => 'Bearer ' . $token,
            'Content-Type'  => 'application/json',
        ],
        'timeout' => 30,
    ],
]);
```

#### B. Form Data Request (like QasemiPay – get token):
```php
$response = HttpHelper::sendForm([
    'method' => 'POST',
    'url' => $this->getApiUrl('token'),
    'form_params' => [
        'client_id'     => $clientId,
        'client_secret' => $clientSecret,
        'grant_type'    => 'password',
        // ...
    ],
    'options' => [
        'headers' => [
            'Content-Type' => 'application/x-www-form-urlencoded',
        ],
    ],
]);
```

#### C. GET Request with Query Parameters:
```php
$response = HttpHelper::get([
    'url' => $this->getApiUrl('check_status') . '/' . $refId,
    'options' => [
        'query' => ['param' => 'value'],
        'headers' => ['Accept' => 'application/json'],
    ],
]);
```

#### D. PATCH Request (like YottaPay – confirm payment):
```php
$response = HttpHelper::sendJson([
    'method' => 'PATCH',
    'url' => $this->getApiUrl('payment'),
    'json' => ['id' => $id, 'otp' => $otp],
    'options' => [...],
]);
```

**Handling the response:**  
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

### 5. Handling Redirects and `RedirectHelper`

For **Redirect** type gateways (like `ThawaniPay`), you must use the `Nano\Yepayment\Classes\RedirectHelper` class to redirect to apps or websites after payment. This class:

- Extracts `callback_success_url` and `callback_error_url` from `order->other_data`.
- Supports Deep Links (like `myapp://`) and regular URLs.
- Handles large data by storing it in Cache using a Token.
- Ensures security by verifying allowed domains and filtering sensitive data.

**Import the class:**
```php
use Nano\Yepayment\Classes\RedirectHelper;
```

**Practical examples from `routes.php` (ThawaniPay):**

#### A. Extracting the redirect URL from order data:
```php
$callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, ['callback_success_url', 'thawani.callback_success_url']);
```

#### B. Redirecting to app or web:
```php
return RedirectHelper::redirectToApp($callbackUrl, $data, $forceJson, $queryParams, $deepLinkSchemes, $is_force_deep_list);
```
- `$callbackUrl`: The extracted URL (may be `https://...` or `myapp://...`).
- `$data`: Data you want to send (payment status, order number, etc.).
- `$forceJson`: If `true`, returns JSON instead of redirecting.
- `$queryParams`: Array of field names to add as query parameters (e.g., `['order_id', 'payment_method_id']`).
- `$deepLinkSchemes`: List of allowed schemes for apps (example `['myapp', 'app', '*']`). `'*'` means any scheme that is not http/https.
- `$is_force_deep_list`: If `true`, forces use of the specified list only.

#### C. Alternative cases when no URL exists:
```php
if (!$callbackUrl) {
    return Response::make('Payment successful. You can return to the app.', 200);
}
```

#### D. Public function to verify and complete payment (like `checkAndCompletePay` in ThawaniPay):
You can create a `static` function in the gateway class that uses `RedirectHelper` internally to unify the logic.

**Note:** You must add `success` and `cancel` routes in `routes.php` to receive the return from the external gateway, then use `RedirectHelper` inside them as in `thawanipay/success`.

### 6. Dealing with `PaymentResult` and Its Impact on Order Status and System Integration

**This section explains the precise relationship between the gateway and order status, and how it aligns with `OrderManager` and `Checkout2`.**

#### 6.1 `PaymentResult` Functions and Their Full Impact

| Function | When to Use | Full Impact |
|----------|--------------|-------------|
| `$result->success($data, $response)` | **Only** when the gateway is sure that the payment process has been final and irreversible. | 1. Sets `successful = true`.<br>2. **Changes order status to `PaidState`**.<br>3. Sets `processed = true`.<br>4. Logs successful `PaymentLog`.<br>5. **Triggers `nano.orders.paymentProcessed` event** (sending emails, updating stock, etc.).<br>6. **Automatically empties the user's cart**. |
| `$result->fail($data, $response)` | When the process fails. | 1. Sets `successful = false`.<br>2. **Changes order status to `FailedState`**.<br>3. Logs failed `PaymentLog`.<br>4. Does not empty the cart. |
| `$result->redirect($url)` | To redirect to an external gateway. | 1. Sets `redirect = true` and `redirectUrl`.<br>2. **Does not change order status**. |
| `$result->pending($data, $response)` | When the transaction is pending and not yet complete (e.g., waiting for external confirmation). | 1. Sets `successful = true`.<br>2. **Changes order status to `PendingState`** (does not become paid).<br>3. Sets `processed = true` but status is not paid.<br>4. Logs successful `PaymentLog`.<br>5. **Does not trigger `paymentProcessed` event**.<br>6. **Empties the cart** even though the order is not paid. |

> ⚠️ **Golden Rule:** Do not call `$result->success()` inside `process()` if the gateway is of the **Two‑Step** type. Use it only inside `complete()` after verifying payment success. For the **Direct** type, you can use `$result->success()` directly in `process()` because payment is instant.

#### 6.2 Gateway Interaction with `OrderManager` and `Checkout2` (Class Harmony with the Rest of the System)

When the user reaches the payment step (`step=pay`) in the client application, the call flow is as follows:
- `Checkout2` calls `$this->orderManager->setStepPayments($data)`.
- `OrderManager` in turn:
  1. Validates previous steps (addresses, shipping, coupons).
  2. Creates `PaymentGateway` and `PaymentService`.
  3. Calls `$paymentService->process()` which in turn calls `$gateway->process($order)` that passes the call to `BasPay->process()` (or any other gateway).
  4. **After `PaymentResult` returns from the gateway**, `OrderManager` acts based on its state:
     - If `$order->isPaymentProcessed()` returns `true` (meaning the order has moved to `PaidState`) → payment is considered complete.
     - If it returns `false` (as in the case of BasPay after `process` only) → displays a message to the user to complete payment (like "Please confirm payment via BAS app").

**Result:** In Two‑Step gateways, immediately after `process()`, `processed = false` and the order is **not** paid, allowing the user to complete the next step. Not calling `$result->success()` inside `process()` is what prevents the order from moving to `PaidState` prematurely.

#### 6.3 Completing Payment in Two‑Step Gateways: `complete()` and `checkAndCompletePay`

To finalize payment, `complete()` must be called when the user returns from confirmation. The optimal way is to provide a **public `static` function** named `checkAndCompletePay` that is used in the `success` route. Example:

```php
public static function checkAndCompletePay(array $options): array
{
    $orderId = $options['order_id'] ?? null;
    $order = Order::find($orderId);
    if (!$order) return ['success' => false, 'message' => 'Order not found'];
    
    $bas = new self($order);
    $result = new PaymentResult($bas, $order);
    $completeResult = $bas->complete($result);

    return [
        'success'  => $completeResult->successful,
        'message'  => $completeResult->message,
        'data'     => $completeResult->api_data,
        'order_id' => $orderId,
    ];
}
```

Then in `routes.php`:

```php
Route::get('baspay/success', function () {
    $options = Input::get();
    $result = BasPay::checkAndCompletePay($options);
    $order_id = $result['order_id'] ?? null;
    if ($order_id) {
        $callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, ['callback_success_url', 'baspay.callback_success_url']);
        if ($callbackUrl) {
            return RedirectHelper::redirectToApp($callbackUrl, $result, false, ['order_id', 'payment_method_id'], ['*'], false);
        }
    }
    return response()->json($result);
});
```

> **Note:** `complete()` should call `$result->success()` only when the transaction is successful; otherwise, use `$result->pending()` or `$result->fail()` depending on the status.

---

## 🧩 Creating Partial Files (Partials)

### 1. `_info.htm`
**Path:** `paymenttypes/newpay/_info.htm`  
Displays introductory information about the gateway on the settings page.

```html
<div class="callout callout-info">
    <h4>New Payment Gateway</h4>
    <p>An integrated payment gateway that enables accepting online payments.</p>
    <hr>
    <strong>Setup Requirements:</strong>
    <ol>
        <li>Obtain API Key from the payment gateway.</li>
        <li>Specify the base API URL (Live/Sandbox).</li>
    </ol>
</div>
```

### 2. `_test_info.htm`
**Path:** `paymenttypes/newpay/_test_info.htm`  
Provides a quick test tool on the settings page.

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
        <button id="newpay-btn-auth" class="btn btn-info">Test Authentication</button>
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
    // Remaining functions with the same pattern using the newpay- prefix
</script>
```

---

## 🖥️ Creating the Integrated Test Page (UI)

**Path:** `views/newpay-ui.htm`

- Use the same structure as `yottapay-ui.htm` or `thawanipay-ui.htm`.
- Change every `yottapay` to `newpay` in IDs, function names, and endpoints.
- Ensure there are tabs (Manual, Automatic, Statistics, Logs).
- Add local logs using `localStorage` with a unique key (`newpay_test_logs`).

**Key snippets:**

- Display current settings: `{{ apiBaseUrl }}`, `{{ settings.url }}`, etc.
- Data entry form (order ID, amount, purchase code, etc.).
- Buttons: Create Transaction, Confirm, Query, Full Test, Automated Test.
- Show results in JSON format.
- Statistics (number of requests, success rate, latest logs) via the `/stats` endpoint.

---

## 🛣️ Adding Routes in `routes.php`

Routes must be added inside the appropriate main group (`yepayment` or `ompayment`). Required routes:

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/newpay/test-auth` | Test authentication |
| `POST` | `/newpay/test-create-payment` | Create a test transaction |
| `POST` | `/newpay/test-confirm-payment` | Confirm the transaction (if needed) |
| `GET` | `/newpay/test-check-status` | Query a transaction's status |
| `POST` | `/newpay/test-full-payment` | Full test |
| `GET` | `/newpay/stats` | Gateway statistics |
| `GET` | `/newpay/test-ui` | Display test page |
| `GET` | `/newpay/success` | (For redirect type) Success route |
| `GET` | `/newpay/cancel` | (For redirect type) Cancel route |

**Important Notes:**

- All test routes are protected with `if(!BackendAuth::getUser())` to prevent unauthorized access.
- Use `HttpHelper` inside the routes (if needed) or call class functions directly.
- In `success`/`cancel` routes, use `RedirectHelper` to redirect to the app (as in `thawanipay/success`).

**Example of `success` route:**
```php
Route::get('newpay/success', function () {
    $options = Input::get();
    $result = NewPay::checkAndCompletePay($options); // Public verification function
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

**Example of `test-create-payment` route:**
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

Place it in the appropriate geographical section (`allow_yemen_payment` or `allow_oman_payment`).

---

## 🌐 Adding Translation Keys

**File:** `lang/ar/lang.php` and `lang/en/lang.php`

```php
return [
    'payment_gateway_settings' => [
        'newpay' => [
            'url'               => 'Base API URL',
            'test_url'          => 'Test API URL',
            'api_key'           => 'API Key',
            'secret_key'        => 'Secret Key',
            'username'          => 'Username',
            'password'          => 'Password',
            'default_currency'  => 'Default Currency',
        ],
    ],
    'public' => [
        'newpay' => [
            'payment_success'          => 'Payment Successful',
            'confirmation_required'    => 'Please enter the confirmation code',
            'transaction_created'      => 'Transaction Created',
            'default_note'             => 'Payment Confirmation',
            'yes'                      => 'Yes',
            'no'                       => 'No',
            'errors' => [
                'auth_failed'                => 'Authentication Failed',
                'order_already_paid'         => 'Order already paid',
                'missing_credentials'        => 'Incomplete credentials',
                'payment_creation_failed'    => 'Payment transaction creation failed',
                'payment_confirmation_failed'=> 'Payment confirmation failed',
                'incomplete_payment_data'    => 'Incomplete payment data',
            ],
        ],
    ],
];
```

---

## 🧪 Additional Tips for Professional Gateway Development

### 1. Mandatory Use of `HttpHelper` and `RedirectHelper`
- **All API requests** must go through `HttpHelper`, not direct `curl` or unwrapped `Guzzle`.
- **All post-payment redirections** (especially for Redirect type gateways) must use `RedirectHelper` to ensure compatibility with apps and web.

### 2. Webhook Support
- Add a route to receive notifications from the gateway (POST).
- Verify the signature if present.
- Automatically update the order status.

### 3. Retry Failed Requests (Retry)
- Add a `for` loop with `try-catch` to execute the request several times before failing.

### 4. Idempotency Support
- Send a unique `Idempotency-Key` (e.g., `order_id_uuid`) in transaction creation requests.

### 5. Public `checkTransactionStatus` Function
- It can be called from `stats` or the UI to display the transaction status.

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
Add a static function in the gateway class to unify the verification and payment completion logic after returning from the external gateway. `RedirectHelper` can be used inside it.

### 9. Dealing with Payment Session and Log Lifecycle (Payment Lifecycle)

- **Log payment attempt in `process`:** Even if payment is not yet complete (in Two‑Step type), you should call `$result->logSuccessfulPayment()` to save the initial log. This is what `YottaPay` and `BasPay` do.
- **Using `complete` in `PaymentRedirector`:** When returning from an external gateway (or mobile app), `PaymentRedirector::handleOffSiteReturn` calls `complete` automatically if the session contains a callback. Ensure `complete` is ready for this scenario.
- **Storing `callback_success_url`:** In gateways that need mobile support, store the URLs in `order->other_data` as in `BasPay` and `ThawaniPay`, and use `RedirectHelper` to extract and redirect to them after payment completion.
- **Handling `processed`:** The `processed` variable on the order turns `true` only when `$result->success()` is called (which sets the status to `PaidState`). When using `$result->pending()`, `processed = true` but the status is `PendingState` not `PaidState`. Rely on `isPaymentProcessed()` to check actual payment completion.

---

## ✅ Checklist for Creating a New Gateway

- [ ] Create `XxxPay.php` class extending `PaymentProvider`.
- [ ] Implement methods: `name`, `identifier`, `validate`, `settings`, `process`, `complete`.
- [ ] Add helper functions: `getAuthToken`, `createPayment`, `confirmPayment`, `checkTransactionStatus`, `getApiUrl`, `parseResponse`.
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
- [ ] **Ensure that `process()` in Two‑Step type does not call `$result->success()`**, but returns success with a confirmation message.
- [ ] **Ensure that `complete()` calls `$result->success()` only after verifying process success**.
- [ ] **Provide a `static checkAndCompletePay` function** for use in the `success` route (especially for Two‑Step and Redirect gateways).
- [ ] **Ensure return URLs** (`callback_success_url` and `callback_error_url`) are stored in `other_data` if passed.
- [ ] **Test that `OrderManager` keeps `processed = false`** immediately after `process()` in Two‑Step gateways.
- [ ] **Verify that `PaymentResult::success` leads to `PaidState`** and triggers events and empties the cart.

---

## 📚 Reference Examples (from Attached Code)

- **YottaPay**: Two‑Step (OTP). `process()` returns `successful=true` with `requires_confirmation`, and `complete()` uses the sent OTP. Shows use of `HttpHelper` in JSON and PATCH requests, and token storage.
- **BasPay**: Two‑Step (confirmation via separate app). `process()` creates a transaction and returns `trxToken`, and `complete()` checks transaction status and updates the order to `PaidState` only upon success. Shows use of `HttpHelper::sendForm` for OAuth token, and static `checkAndCompletePay` for confirmation via `success` route.
- **QasemiPay**: Direct with optional confirmation (concurrencyStamp). `process()` and `complete()` follow a similar pattern, and shows use of `HttpHelper::sendForm` for OAuth token.
- **ThawaniPay**: Redirect with redirect to external gateway. `process()` redirects, and `complete()` is called after return. Shows use of `RedirectHelper` in `success`/`cancel` routes, and `checkAndCompletePay` for confirmation.

---

## 🏁 Conclusion

By following this guide, you can create any new payment method in the `Nano.Yepayment` system regardless of its API complexity. Use the appropriate pattern (Redirect, Two‑Step, Direct) according to the payment flow, and do not forget:

- Use `HttpHelper` for all API requests.
- Use `RedirectHelper` for post-payment redirection (even in indirect gateways that support mobile).
- Understand the impact of `PaymentResult` functions (`success`, `pending`, `fail`) on order status, cart emptying, and events.
- Provide a `static checkAndCompletePay` function to simplify confirmation from return routes.
- Store return URLs (`callback_success_url`) in `other_data`.
- Remember that `OrderManager` and `Checkout2` rely on `isPaymentProcessed()` to determine the next step, so do not rush calling `$result->success()`.
- Provide necessary test tools (`_test_info.htm` and `xxxpay-ui.htm`) to ease development and maintenance.

🚀 **Start adding your new gateway now!**
