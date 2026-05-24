# SKILL: Creating a New Payment Method within the `Nano\Yepayment` Plugin

---

## 📖 Overview

The `Nano.Yepayment` system allows adding new payment gateways by following a unified pattern that ensures seamless integration with `Nano.MicroCart` and `Nano.Orders`. This guide aims to enable developers to create any payment provider (gateway) by following clear steps and specific standards, supporting three main types of payment flows, and providing helper tools such as `HttpHelper` for making API requests, and `RedirectHelper` for secure redirection to applications and websites.

**Basic requirements:**
- Familiarity with Laravel / October CMS (NanoSoft framework).
- Basic understanding of payment gateways and their APIs.
- Knowledge of PHP and OOP.

**What does this guide provide?**
- Detailed explanation of the payment class structure and required methods.
- Ready‑to‑use templates for each payment type (Direct, Two‑Step, Redirect).
- Guidance on using `HttpHelper` to unify HTTP requests.
- Advanced mechanisms: idempotency, token management, payment attempt logging, special error handling, currency support, refund, and deep link support for mobile applications.
- A comprehensive checklist to ensure the gateway is complete before launch.

Using this guide, you will be able to integrate any external payment gateway regardless of the complexity of its API, while ensuring the highest levels of security, performance, and compatibility with the NanoSoft system.

---

## 🧩 Types of Payment Methods (Payment Flows)

Based on the nature of the external gateway's API, payment methods are classified into three main patterns. Choosing the appropriate pattern is the first and most important step in development.

| Type | Description | Example | Main Methods | When to Use |
|------|-------------|---------|---------------|--------------|
| **Redirect** | Redirects the user to an external gateway (bank, PayPal, etc.) to complete the payment, then returns to the specified `success_url` or `cancel_url`. | `ThawaniPay`, `YkbPay` | `process()` → `$result->redirect()`<br>`success` route completes the payment. | When the gateway requires the user to complete payment on an external page (e.g., bank portal). |
| **Two‑Step** | First creates a transaction (`process`), then requires a later confirmation (`complete`) via an OTP or code from the customer, or waiting for confirmation from an external application. | `YottaPay`, `CashPay`, `BasPay` | `process()` returns `successful = true` and a message "Enter OTP" (without calling `success()`).<br>`complete()` confirms the payment and calls `success()`. | When the gateway needs an additional confirmation step (e.g., entering a code from the customer's phone). |
| **Direct (Immediate)** | Payment is made immediately via a single API call without the need for redirection or additional confirmation. | `QasemiPay` (when confirmation is automatic), `KuraimiPay` | `process()` executes the payment and returns `$result->success()` directly. | When the gateway returns an immediate success status after a single API call. |

### Comparison of the types in terms of flow and impact on order status

| Type | User Flow | Does `process()` call `success()`? | Does `complete()` call `success()`? | When does the order become paid (`PaidState`)? |
|------|-----------|------------------------------------|--------------------------------------|--------------------------------------------------|
| **Redirect** | User → your site → external gateway → return to `success` | No (calls `redirect()`) | Yes (after return and confirmation) | After returning from the external gateway and successful confirmation |
| **Two‑Step** | User → your site → create transaction → enter OTP → confirm | No (returns `successful=true` only) | Yes (after OTP verification) | After entering a correct OTP and successful confirmation |
| **Direct** | User → your site → immediate payment → completed | Yes (immediately upon API success) | Not used | Immediately after `process()` finishes |

> **Important note:** A single gateway can combine more than one pattern (e.g., create a transaction then redirect, then later confirm upon return). In that case, choose the pattern that fits the gateway's API documentation, and implement the methods accordingly.

---

## 🏗️ Payment Method Class Structure

Each payment gateway must be created as a PHP class extending `PaymentProvider` (found in `Nano\MicroCart\Classes\Payments\PaymentProvider`). The specific file location is:

```
plugins/nano/yepayment/paymenttypes/NewPay.php
```

Where `NewPay` is the gateway name (preferably using a brand name like `ThawaniPay`, `BasPay`).

### Basic Class Structure

```php
<?php

namespace Nano\Yepayment\PaymentTypes;

use Nano\MicroCart\Classes\Payments\PaymentProvider;
use Nano\MicroCart\Classes\Payments\PaymentResult;
use Nano\MicroCart\Models\PaymentGatewaySettings;
use Nano\Helpers\Classes\Helpers\HttpHelper;
use Nano\Yepayment\Classes\RedirectHelper;
use Config, Request, Validator, Throwable, ApplicationException;

class NewPay extends PaymentProvider
{
    // ========== Mandatory Properties ==========
    /**
     * Success path after payment (used for Redirect type)
     * @var string
     */
    public $success_url = 'api/v1/yepayment/newpay/success';
    
    /**
     * Cancel path (used for Redirect type)
     * @var string
     */
    public $cancel_url  = 'api/v1/yepayment/newpay/cancel';
    
    /**
     * Test mode (true = Sandbox, false = Production)
     * Set automatically from gateway settings
     * @var bool
     */
    public $is_test_mod = false;

    // ========== Abstract Methods to Implement ==========
    
    /**
     * Commercial name of the gateway (appears in UI)
     */
    public function name(): string { return 'New Payment Gateway'; }
    
    /**
     * Unique identifier for the gateway (lowercase Latin letters, no spaces)
     */
    public function identifier(): string { return 'newpay'; }
    
    /**
     * Validate customer input (can return true and validate later)
     */
    public function validate(): bool { return true; }
    
    /**
     * Define gateway settings (will appear in control panel)
     */
    public function settings(): array { /* ... */ }
    
    /**
     * Logic for initial payment transaction creation
     */
    public function process(PaymentResult $result): PaymentResult { /* ... */ }
    
    /**
     * Logic for completing the payment (for Two‑Step / Redirect)
     */
    public function complete(PaymentResult $result): PaymentResult { /* ... */ }
    
    /**
     * Fields that should be encrypted when stored (e.g., passwords)
     */
    public function encryptedSettings(): array { return ['newpay_password']; }

    // ========== Recommended Helper Methods (not abstract but essential for full integration) ==========
    
    /**
     * Define validation rules for input data (e.g., order_id, otp, callback_url)
     */
    public function defineValidationRules(): array { /* ... */ }
    
    /**
     * Human-readable field names (for translation)
     */
    public function getFieldNames(): array { /* ... */ }
    
    /**
     * Obtain authentication token (OAuth2 or any token system)
     */
    private function getAuthToken(): ?string { /* ... */ }
    
    /**
     * Create a payment via the gateway API
     */
    private function createPayment($token): array { /* ... */ }
    
    /**
     * Confirm payment via the gateway API (using OTP or any identifier)
     */
    private function confirmPayment($token, $id, $code): array { /* ... */ }
    
    /**
     * Check transaction status using its ID
     */
    public function checkTransactionStatus($refId): array { /* ... */ }
    
    /**
     * Build API URL based on operation type and test mode
     */
    private function getApiUrl(string $type): string { /* ... */ }
    
    /**
     * Convert HTTP response to PHP array
     */
    private function parseResponse($response): array { /* ... */ }
    
    /**
     * Refund an amount
     */
    public function reverse(string $refNo, string $reason = '', ?string $customerId = null): array { /* ... */ }
    
    /**
     * Static function to complete payment from the success route (unified logic)
     */
    public static function checkAndCompletePay(array $options): array { /* ... */ }
    
    /**
     * Validate essential settings
     */
    public function validateSettings(): void { /* ... */ }
    
    /**
     * Check if the gateway is available (credentials valid)
     */
    public function isAvailable(): bool { /* ... */ }
}
```

### Explanation of Core Components

| Component | Type | Description |
|-----------|------|-------------|
| `$success_url` | public property | The path to redirect to after successful payment (for Redirect type). Also used as callback in some APIs. |
| `$cancel_url` | public property | The path to redirect to when payment is cancelled. |
| `$is_test_mod` | public property | Set automatically from gateway settings (`test_mode`). Used to switch between test and production environments. |
| `name()` | abstract method | Returns the commercial name of the gateway as shown to the user. Prefer using `trans()` for translation. |
| `identifier()` | abstract method | Returns a unique identifier for the gateway (lowercase Latin letters). Used as a key for storing settings and to distinguish the gateway in `payment_first_trans_id` and `other_data`. |
| `validate()` | abstract method | Validates customer input (e.g., existence of `order_id`). Prefer returning `true` and using `defineValidationRules()` with `Validator` inside `process()`. |
| `settings()` | abstract method | Defines the gateway settings fields (url, api_key, username, password, ...). Returns an array, automatically displayed in the control panel. |
| `process()` | abstract method | Core payment logic: validate input, prevent duplication, create transaction, save data, log attempt, then return appropriate result (redirect, pending, or success). |
| `complete()` | abstract method | Completes the payment after returning from external gateway (Redirect) or after OTP entry (Two‑Step). Called from the `success` route or a dedicated API endpoint. |
| `encryptedSettings()` | abstract method | Returns an array of field names defined in `settings()` with type `password`, to be automatically encrypted when stored. |

### Highly Recommended Helper Methods

| Method | Purpose |
|--------|---------|
| `defineValidationRules()` | Define validation rules for `order_id`, `otp`, `amount`, `callback_success_url`, etc. |
| `getFieldNames()` | Return field names as they will appear to the user (e.g., 'otp' => 'Verification Code'). |
| `getAuthToken()` | Manage token with cache and retry upon expiration. |
| `createPayment()` / `confirmPayment()` | Execute API requests to the gateway using `HttpHelper`. |
| `checkTransactionStatus()` | Query transaction status (for idempotency and testing). |
| `getApiUrl()` | Build the full API URL based on `$is_test_mod` and operation type. |
| `parseResponse()` | Convert Guzzle response to a safe array. |
| `reverse()` | Refund a transaction (if the gateway supports refund). |
| `checkAndCompletePay()` | Static function used in the `success` route to complete payment and return a unified result. |
| `validateSettings()` | Check that all essential settings exist before processing payment. |
| `isAvailable()` | Check whether the gateway is ready for use (credentials valid). |
| `getSupportedCurrencies()` / `getCurrencyId()` | Support different currencies. |
| `generateIdempotencyKey()` | Generate a unique identifier to prevent duplicate transactions. |

### Best Practices When Writing the Class

1. **Use PHPDoc** for every property and method, explaining parameters, return values, and exceptions.
2. **Do not deviate from the unified pattern** to ensure compatibility with `OrderManager` and `Checkout2`.
3. **Do not call `$result->success()` inside `process()`** if the gateway is of type Two‑Step or Redirect.
4. **Use `trace_log()` to log errors** but without exposing sensitive data (tokens, passwords).
5. **Test the class thoroughly** using the built‑in tools (`_test_info.htm` and `xxxpay-ui.htm`).

With this structure, you have completed the basic framework for any new payment gateway. The following steps will explain in detail how to implement each of the parts mentioned above.

---

## 📝 Detailed Creation Steps

### 1. Create the Main Class

- Create a new PHP file in `paymenttypes/`.
- Ensure the class extends `PaymentProvider`.
- Set `$success_url` and `$cancel_url` according to the route group used (`yepayment` or `ompayment`).
- Define `$is_test_mod` to switch between test and production environments.

### 2. Implement the Abstract Methods

All abstract methods inherited from `PaymentProvider` must be implemented correctly and completely. Below is a detailed explanation of each method with its requirements and best practices.

---

### 2.1 `name(): string`

**Purpose:** Return the commercial name of the payment gateway as it will appear to the user in the options interface.

**Requirements:**
- The name should be clear and match the gateway's brand.
- Prefer using translation via `trans()` if the name will be displayed in multiple languages.

**Example:**
```php
public function name(): string
{
    return trans('nano.yepayment::lang.payment_gateways.newpay.name');
    // Or simply: return 'New Payment Gateway';
}
```

---

### 2.2 `identifier(): string`

**Purpose:** Return a unique identifier for the gateway used internally (storing settings, distinguishing records, building routes).

**Requirements:**
- Lowercase Latin letters only.
- No spaces or special characters (underscore `_` is allowed).
- Must be unique among all registered gateways.
- This identifier is used in storage keys, in `payment_first_trans_id`, in `other_data`, and in the `success_url` and `cancel_url` paths.

**Example:**
```php
public function identifier(): string
{
    return 'newpay'; // e.g., 'newpay', 'cashpay', 'baspay'
}
```

**Note:** The final route for the gateway will be: `api/v1/yepayment/{identifier}/success` based on the `$success_url` property.

---

### 2.3 `validate(): bool`

**Purpose:** Validate the data sent from the client (e.g., `order_id`, `amount`, `phone`, etc.) before starting the payment process.

**Requirements:**
- You can always return `true` and rely on `defineValidationRules()` separately.
- Or implement custom validation logic and return `false` with an appropriate error message via `$result->fail()`.

**Recommended approach:**
Use `defineValidationRules()` with `Validator` inside `process()` instead of relying on `validate()`, because `validate()` is not widely used in the core system. So it is better to implement it as:

```php
public function validate(): bool
{
    return true; // Validation will be done inside process() using defineValidationRules()
}
```

---

### 2.4 `settings(): array`

**Purpose:** Define the settings fields that will appear on the gateway settings page in the control panel. Returns an array of fields, each with a type, label, and default value.

**Requirements:**
- Use `type => 'partial'` to include partial files such as `_info.htm` and `_test_info.htm`.
- Use `type => 'password'` for sensitive fields (API keys, passwords) and add them to `encryptedSettings()`.
- Specify `placeholder`, `label`, `span`, `default` as needed.

**Typical field structure:**
```php
public function settings(): array
{
    return [
        'newpay_setup' => [
            'type' => 'partial',
            'path' => '$/nano/yepayment/paymenttypes/newpay/_info.htm',
        ],
        'newpay_url' => [
            'label' => trans('nano.yepayment::lang.payment_gateway_settings.newpay.url'),
            'type'  => 'text',
            'span'  => 'left',
            'placeholder' => 'https://api.newpay.com/v1',
            'default' => 'https://api.newpay.com/v1',
            'tab'   => 'general',
        ],
        'newpay_test_url' => [
            'label' => trans('nano.yepayment::lang.payment_gateway_settings.newpay.test_url'),
            'type'  => 'text',
            'span'  => 'right',
            'placeholder' => 'https://sandbox.newpay.com/v1',
            'default' => 'https://sandbox.newpay.com/v1',
            'tab'   => 'general',
        ],
        'newpay_api_key' => [
            'label' => trans('nano.yepayment::lang.payment_gateway_settings.newpay.api_key'),
            'type'  => 'text',
            'span'  => 'left',
            'tab'   => 'credentials',
        ],
        'newpay_secret_key' => [
            'label' => trans('nano.yepayment::lang.payment_gateway_settings.newpay.secret_key'),
            'type'  => 'password', // سيتم تشفيره تلقائياً إذا أضيف إلى encryptedSettings()
            'span'  => 'right',
            'tab'   => 'credentials',
        ],
        'newpay_username' => [
            'label' => trans('nano.yepayment::lang.payment_gateway_settings.newpay.username'),
            'type'  => 'text',
            'span'  => 'left',
            'tab'   => 'credentials',
        ],
        'newpay_password' => [
            'label' => trans('nano.yepayment::lang.payment_gateway_settings.newpay.password'),
            'type'  => 'password',
            'span'  => 'right',
            'tab'   => 'credentials',
        ],
        'newpay_default_currency' => [
            'label' => trans('nano.yepayment::lang.payment_gateway_settings.newpay.default_currency'),
            'type'  => 'dropdown',
            'options' => [
                'YER' => 'ريال يمني',
                'USD' => 'دولار أمريكي',
                'SAR' => 'ريال سعودي',
            ],
            'default' => 'YER',
            'tab'   => 'advanced',
        ],
        'newpay_test_info' => [
            'type' => 'partial',
            'path' => '$/nano/yepayment/paymenttypes/cashpay/_test_info.htm',
        ],
    ];
}
```

**Important notes:**
- Use `tab` to organise fields into tabs (`general`, `credentials`, `advanced`).
- Fields of type `password` are automatically encrypted in the database if they are included in `encryptedSettings()`.
- `partials` do not need `encryptedSettings`.

---

### 2.5 `encryptedSettings(): array`

**Purpose:** Specify which of the fields defined in `settings()` should be stored encrypted in the database.

**Requirements:**
- Returns an array of field names that were defined with `type => 'password'`.
- Other fields are not automatically encrypted.

**Example:**
```php
public function encryptedSettings(): array
{
    return ['secret_key', 'password'];
}
```

**To retrieve settings values correctly, it is preferable to create a function as follows:**

```php
public function getSettingsByKey(string $key, $default = null)
{
    try {
        $setting = PaymentGatewaySettings::instance();
        if (in_array($key, $this->encryptedSettings())) {
            return $setting->getEncryptableValue($key) ?: $default;
        }
        return $setting->get($key, $default);
    } catch (\Throwable $e) {
        return $default;
    }
}
```

---

### 2.6 `process(PaymentResult $result): PaymentResult`

**Purpose:** Execute the initial payment transaction logic. This is the heart of the gateway, where communication with the payment provider's API takes place to create a new transaction or resume an existing one.

**Detailed required steps:**

#### 2.6.1 Validate input data
Use `Validator` with the validation rules defined in `defineValidationRules()`.

```php
$validator = Validator::make($this->data, $this->defineValidationRules());
if ($validator->fails()) {
    throw new ValidationException($validator);
}
```

#### 2.6.2 Ensure the order is not already paid (quick check)
```php
if ($this->order->payment_state == PaidState::class) {
    $result->successful = false;
    $result->message = trans('nano.yepayment::lang.public.order_already_paid');
    return $result;
}
```

#### 2.6.3 Search for an existing transaction to prevent duplication (Idempotency)
```php
$existingTxId = $this->order->payment_first_trans_id 
    ?? ($this->order->other_data[$this->identifier()]['transaction_id'] ?? null);

if ($existingTxId) {
    $status = $this->checkTransactionStatus($existingTxId);
    if ($status['success'] && ($status['status'] ?? '') === 'Complete') {
        return $result->success($status, null);
    }
    if ($status['success'] && ($status['status'] ?? '') === 'Pending') {
        $result->successful = true;
        $result->message = trans('nano.yepayment::lang.public.payment_pending_continue');
        return $result;
    }
    // If status is Failed/Expired/unknown, continue to create a new transaction
}
```

#### 2.6.4 Obtain authentication token (if required)
```php
$token = $this->getAuthToken();
if (!$token) {
    throw new ApplicationException(trans('nano.yepayment::lang.public.auth_failed'));
}
```

#### 2.6.5 Create payment transaction via API
```php
$paymentResult = $this->createPayment($token);
if (!$paymentResult['success']) {
    throw new ApplicationException($paymentResult['message']);
}
```

#### 2.6.6 Save transaction data in the order
- `payment_first_trans_id`: primary transaction identifier (e.g., `trxToken`, `TransactionRef`, `PH_REF_NO`).
- `payment_trans_id`: secondary identifier if available.
- `other_data[identifier]`: object containing all transaction data, return URLs, and any data needed for refund (e.g., `scustId`).

```php
$this->order->payment_first_trans_id = $paymentResult['transaction_id'];
$this->order->payment_trans_id = $paymentResult['ref_no'] ?? null;

$other = $this->order->other_data;
$other[$this->identifier()] = [
    'transaction_id'   => $paymentResult['transaction_id'],
    'ref_no'           => $paymentResult['ref_no'] ?? null,
    'callback_success_url' => $this->data['callback_success_url'] ?? null,
    'callback_error_url'   => $this->data['callback_error_url'] ?? null,
    'currency'         => $this->data['currency'] ?? $this->order->Currency->currency_code ?? 'YER',
    'amount'           => $this->order->total,
    // any additional data required for refund or later queries
];
$this->order->other_data = $other;
$this->order->save();
```

#### 2.6.7 Log the payment attempt in `PaymentLog` (even if not completed)
This is important for tracking payment attempts and analysis.

```php
try {
    $paymentLog = $result->logSuccessfulPayment($paymentResult, null, 'initiated');
    $this->order->payment_id = $paymentLog->id ?? null;
    $this->order->save();
} catch (\Throwable $e) {
    trace_log($this->identifier() . ': Failed to log PaymentLog: ' . $e->getMessage());
}
```

#### 2.6.8 Return the appropriate result based on gateway type

| Gateway Type | Action |
|--------------|--------|
| **Redirect** | `return $result->redirect($paymentResult['redirect_url']);` |
| **Two‑Step** | Set `$result->successful = true` and a message "Enter verification code", then `return $result;` **without** calling `$result->success()`. |
| **Direct** | Update `payment_state` to `PaidState::class`, then `return $result->success($paymentResult, null);` |

```php
if (isset($paymentResult['redirect_url'])) {
    return $result->redirect($paymentResult['redirect_url']);
}

if ($paymentResult['requires_confirmation'] ?? false) {
    $result->successful = true;
    $result->message = trans('nano.yepayment::lang.public.confirmation_required');
    $result->api_data = $paymentResult;
    return $result;
}

// Direct
$this->order->payment_state = PaidState::class;
$this->order->save();
return $result->success($paymentResult, null);
```

#### 2.6.9 Exception handling
```php
catch (Throwable $e) {
    return $result->fail([], $e);
}
```

**Note:** Never call `$result->success()` inside `process()` for Two‑Step or Redirect gateways, because that would mark the order as paid prematurely.

---

### 2.7 `complete(PaymentResult $result): PaymentResult`

**Purpose:** Complete the payment process after the user returns from an external gateway (Redirect) or after entering an OTP (Two‑Step). This method is called from the `success` route or a dedicated API endpoint.

**Detailed steps:**

#### 2.7.1 Retrieve stored transaction data
```php
$transactionId = $this->order->payment_first_trans_id;
if (!$transactionId) {
    throw new ApplicationException('No transaction found to complete');
}
```

#### 2.7.2 Obtain verification code (if needed)
```php
$otp = $this->data['otp'] ?? null;
```

#### 2.7.3 Request payment confirmation from API
```php
$token = $this->getAuthToken();
$confirmResult = $this->confirmPayment($token, $transactionId, $otp);
```

#### 2.7.4 Handle special error codes
```php
if (isset($confirmResult['requires_password_change']) && $confirmResult['requires_password_change']) {
    trace_log($this->identifier() . ': Password expired, needs to be changed');
    return $result->fail($confirmResult, null);
}
```

#### 2.7.5 Update order status on success
```php
if (!$confirmResult['success']) {
    throw new ApplicationException($confirmResult['message']);
}

return $result->success($confirmResult, null);
```

#### 2.7.6 Exception handling
```php
catch (Throwable $e) {
    return $result->fail([], $e);
}
```

> **Important:** The `complete()` method is the only place (along with `process()` in Direct type) that should call `$result->success()`.

---

### 2.8 Additional Helper Methods (not abstract but highly recommended)

#### `defineValidationRules(): array`
Defines the validation rules for input data sent by the client (e.g., `order_id`, `otp`, `callback_success_url`).

```php
public function defineValidationRules(): array
{
    return [
        'order_id' => 'required|exists:orders,id',
        'otp'      => 'sometimes|required|string|size:4',
        'callback_success_url' => 'nullable|url',
    ];
}
```

#### `getFieldNames(): array`
Returns the field names that will appear in the customer interface (e.g., requesting OTP).

```php
public function getFieldNames(): array
{
    return [
        'otp' => 'Verification Code',
    ];
}
```

#### `getApiUrl(string $type): string`
Returns the appropriate API URL based on the operation type (`token`, `payment`, `confirm`, `reverse`, `status`) and test mode (`$this->is_test_mod`).

```php
private function getApiUrl(string $type): string
{
    $base = $this->is_test_mod 
        ? PaymentGatewaySettings::get($this->identifier() . '_test_url')
        : PaymentGatewaySettings::get($this->identifier() . '_url');
    
    $endpoints = [
        'token'   => '/oauth/token',
        'payment' => '/payments',
        'confirm' => '/payments/confirm',
        'reverse' => '/refunds',
        'status'  => '/payments/status',
    ];
    
    return rtrim($base, '/') . ($endpoints[$type] ?? '');
}
```

#### `parseResponse($response): array`
Converts an HTTP response to a PHP array, handling JSON and text responses.

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

#### `getAuthToken(): ?string`
If the gateway requires a token (OAuth, JWT), implement this method with caching as described in the "Token Management" section.

#### `createPayment($token): array`, `confirmPayment($token, $id, $code): array`
Gateway‑specific methods to execute the API requests for creating and confirming transactions.

#### `checkTransactionStatus($refId): array`
To query the status of a transaction using its ID (if the gateway supports it). If not supported, return an explanatory message.

---

### ✅ Summary of Abstract Method Implementation

| Method | Is abstract? | Essentials |
|--------|--------------|------------|
| `name()` | Yes | Return a clear commercial name |
| `identifier()` | Yes | Unique identifier with lowercase Latin letters |
| `validate()` | Yes | Prefer returning `true` and validate inside `process()` |
| `settings()` | Yes | Define settings fields with `partials` and `password` |
| `process()` | Yes | Transaction creation logic, idempotency, logging, return result based on type |
| `complete()` | Yes | Complete payment, call `$result->success()` on success |
| `encryptedSettings()` | Yes | Return array of field names of type `password` |

With adherence to the golden rules:
- **Do not** call `$result->success()` in `process()` for Two‑Step or Redirect gateways.
- **Always** check for an existing transaction before creating a new one.
- **Always** log the payment attempt in `PaymentLog` even if not completed.
- **Always** store return URLs in `other_data` to support mobile applications.

🚀 **By following these guidelines, your abstract methods will be complete and professional.**

### 3. Using `HttpHelper` for API Requests

#### 3.1 Overview

`HttpHelper` is a unified helper class that provides a simplified interface for making HTTP requests using Guzzle. **It must be used exclusively** for all external API requests; direct `curl` or raw `Guzzle` without wrapping is not permitted.

**Features:**
- Supports all HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Supports different formats: JSON, Form Data, Multipart
- Unified error and exception handling
- Ability to customise headers and options
- Built‑in documentation and ease of use

**Import the class:**
```php
use Nano\Helpers\Classes\Helpers\HttpHelper;
```

---

#### 3.2 Available `HttpHelper` Methods

| Method | HTTP Method | Sending Format | Typical Use |
|--------|-------------|----------------|--------------|
| `HttpHelper::sendJson()` | POST (or as per `method`) | `application/json` | Create transaction, confirm payment, OAuth2 requests with JSON body |
| `HttpHelper::sendForm()` | POST (or as per `method`) | `application/x-www-form-urlencoded` | OAuth2 password / client_credentials grant |
| `HttpHelper::get()` | GET | Query parameters | Query transaction status, fetch data |
| `HttpHelper::post()` | POST | Depends on `options` | General cases |
| `HttpHelper::put()` | PUT | Depends on `options` | Full resource update |
| `HttpHelper::patch()` | PATCH | Depends on `options` | Partial resource update |
| `HttpHelper::delete()` | DELETE | Depends on `options` | Delete a resource |

---

#### 3.3 Using `HttpHelper::sendJson()` – JSON Requests

**Typical use:** Creating a payment transaction, confirming OTP, API calls expecting `Content-Type: application/json`.

**Syntax:**
```php
$response = HttpHelper::sendJson([
    'method' => 'POST',           // POST, PUT, PATCH, DELETE
    'url'    => 'https://api.example.com/v1/payments',
    'json'   => [                  // will be automatically converted to JSON
        'amount' => 100,
        'currency' => 'YER',
        'order_id' => $orderId,
    ],
    'options' => [                // optional: additional Guzzle settings
        'headers' => [
            'Authorization' => 'Bearer ' . $token,
            'Idempotency-Key' => $idempotencyKey,
        ],
        'timeout' => 30,
        'verify' => true,
    ],
]);
```

**Practical example from a real gateway (creating a transaction in YottaPay):**
```php
$response = HttpHelper::sendJson([
    'method' => 'POST',
    'url' => $this->getApiUrl('payment'),
    'json' => [
        'amount' => $this->order->total,
        'currency' => $currencyCode,
        'order_id' => $this->order->id,
        'customer_name' => $this->order->name,
        'customer_email' => $this->order->email,
        'callback_url' => $this->success_url,
        'cancel_url' => $this->cancel_url,
    ],
    'options' => [
        'headers' => [
            'Authorization' => 'Bearer ' . $token,
            'Content-Type' => 'application/json',
            'Accept' => 'application/json',
        ],
        'timeout' => 30,
    ],
]);
```

**Processing the response:**
```php
$parsed = $this->parseResponse($response);
if (isset($parsed['transaction_id'])) {
    // success
} else {
    // failure
}
```

---

#### 3.4 Using `HttpHelper::sendForm()` – Form Data Requests

**Typical use:** Obtaining an OAuth2 token (`grant_type=password` or `client_credentials`), or any API expecting `application/x-www-form-urlencoded`.

**Syntax:**
```php
$response = HttpHelper::sendForm([
    'method' => 'POST',
    'url'    => 'https://api.example.com/oauth/token',
    'form_params' => [
        'grant_type' => 'password',
        'client_id' => $clientId,
        'client_secret' => $clientSecret,
        'username' => $username,
        'password' => $password,
    ],
    'options' => [
        'headers' => [
            'Content-Type' => 'application/x-www-form-urlencoded',
        ],
        'timeout' => 15,
    ],
]);
```

**Practical example from a real gateway (BasPay – obtain token):**
```php
$response = HttpHelper::sendForm([
    'method' => 'POST',
    'url' => $this->getApiUrl('token'),
    'form_params' => [
        'grant_type' => 'password',
        'client_id' => $clientId,
        'client_secret' => $clientSecret,
        'username' => $username,
        'password' => $password,
        'scope' => 'write',
    ],
    'options' => [
        'headers' => [
            'Content-Type' => 'application/x-www-form-urlencoded',
        ],
    ],
]);
```

**Processing the response (extract token):**
```php
$parsed = $this->parseResponse($response);
$accessToken = $parsed['access_token'] ?? null;
```

---

#### 3.5 Using `HttpHelper::get()` – GET Requests

**Typical use:** Query transaction status, fetch data, test connectivity.

**Syntax:**
```php
$response = HttpHelper::get([
    'url' => 'https://api.example.com/v1/transactions/' . $transactionId,
    'options' => [
        'query' => ['param1' => 'value1'], // query string parameters
        'headers' => [
            'Authorization' => 'Bearer ' . $token,
            'Accept' => 'application/json',
        ],
        'timeout' => 20,
    ],
]);
```

**Practical example (checking transaction status):**
```php
$response = HttpHelper::get([
    'url' => $this->getApiUrl('status') . '/' . $refNo,
    'options' => [
        'headers' => [
            'Authorization' => 'Bearer ' . $token,
        ],
    ],
]);
```

---

#### 3.6 Using `HttpHelper::patch()` – Partial Updates

**Typical use:** Confirm payment (change transaction status), update partial data.

```php
$response = HttpHelper::sendJson([
    'method' => 'PATCH',
    'url' => $this->getApiUrl('payment') . '/' . $transactionId,
    'json' => [
        'status' => 'confirm',
        'otp' => $otp,
    ],
    'options' => [
        'headers' => ['Authorization' => 'Bearer ' . $token],
    ],
]);
```

---

#### 3.7 `parseResponse()` – Convert Response to Array

**This function must be implemented in every gateway class** to convert the Guzzle `ResponseInterface` to a PHP array that can be easily handled.

```php
private function parseResponse($response): array
{
    // Get response body
    $body = $response->getBody()->getContents();
    $contentType = $response->getHeaderLine('content-type');
    
    // If response is JSON, parse it
    if (strpos($contentType, 'application/json') !== false) {
        $decoded = json_decode($body, true);
        return is_array($decoded) ? $decoded : [];
    }
    
    // For non‑JSON responses (plain text, XML, etc.)
    return [
        'raw_body' => $body,
        'status_code' => $response->getStatusCode(),
        'content_type' => $contentType,
    ];
}
```

**Usage:**
```php
$response = HttpHelper::sendJson([...]);
$parsed = $this->parseResponse($response);
if (isset($parsed['error'])) {
    // failure
}
```

---

#### 3.8 Retry Mechanism for Failed Requests

To handle temporary errors (e.g., network issues or server timeouts), you can add a retry mechanism:

```php
private function requestWithRetry(array $requestConfig, int $maxRetries = 3): array
{
    $attempt = 0;
    $lastException = null;
    
    while ($attempt < $maxRetries) {
        try {
            $response = HttpHelper::sendJson($requestConfig);
            return $this->parseResponse($response);
        } catch (\Exception $e) {
            $lastException = $e;
            $attempt++;
            if ($attempt >= $maxRetries) break;
            usleep(500000 * $attempt); // delay 0.5, 1, 1.5 seconds
        }
    }
    
    return ['success' => false, 'message' => $lastException->getMessage()];
}
```

---

#### 3.9 Common Errors and How to Avoid Them

| Error | Cause | Solution |
|-------|-------|----------|
| `cURL error 60: SSL certificate problem` | Environment does not trust server certificate | Add `'verify' => false` in `options` (only for testing) |
| Connection timeout | Server is slow or network is weak | Increase `'timeout'` to 30 or 60 |
| `401 Unauthorized` | Token expired or invalid | Clear token from Cache and retry |
| `422 Unprocessable Entity` | Sent data is invalid | Check format of `json` or `form_params` |
| Non‑JSON response | API returned HTML or text error | Use `parseResponse()` to handle both cases |

---

#### 3.10 Golden Rules for Using `HttpHelper`

1. **Use `sendJson()`** when the API documentation specifies `Content-Type: application/json`.
2. **Use `sendForm()`** when the API expects `application/x-www-form-urlencoded` (especially OAuth).
3. **Never use raw `curl`** – `HttpHelper` is the unified standard.
4. **Always parse the response** via `parseResponse()`.
5. **Log responses in case of errors** using `trace_log()` to facilitate debugging.
6. **Use an appropriate `timeout`** (30 seconds minimum for sensitive requests).
7. **Add `Accept: application/json`** in headers to ensure a JSON response.

---

### 4. Token Management

#### 4.1 Overview

Most modern payment gateways use OAuth2 or JWT for authentication, where a token with a limited lifetime (usually 3600 seconds) is obtained and then used for subsequent API requests. Proper token management includes:
- Caching to avoid repeated requests.
- Handling expiration (401) and retrying.
- Automatically refreshing the token when needed.

---

#### 4.2 Structure of `getAuthToken()`

```php
use Illuminate\Support\Facades\Cache;

public function getAuthToken(bool $useCache = true): ?string
{
    $cacheKey = $this->identifier() . '_access_token';
    
    // 1. Attempt to read from Cache
    if ($useCache && ($cached = Cache::get($cacheKey)) && !empty($cached)) {
        return $cached;
    }
    
    // 2. Request a new token
    $tokenData = $this->requestNewToken();
    
    if (!$tokenData || empty($tokenData['access_token'])) {
        trace_log($this->identifier() . ': Failed to obtain token');
        return null;
    }
    
    // 3. Store token in Cache with a lifetime slightly shorter than the actual expiry
    $expiresIn = $tokenData['expires_in'] ?? 3600;
    $cacheTtl = $expiresIn - 300; // store for 5 minutes less
    if ($useCache) {
        Cache::put($cacheKey, $tokenData['access_token'], $cacheTtl);
    }
    
    return $tokenData['access_token'];
}
```

**Parameter explanation:**
- `$useCache`: in some cases (e.g., testing) you may want to disable caching.
- `$cacheKey`: must be unique per gateway (using `identifier()`).
- `$cacheTtl`: store for less than the actual expiry to avoid using an expired token.

---

#### 4.3 Implementing `requestNewToken()`

This is a gateway‑specific method that actually contacts the authentication API.

```php
private function requestNewToken(): array
{
    // Read credentials from settings
    $clientId = PaymentGatewaySettings::get($this->identifier() . '_client_id');
    $clientSecret = PaymentGatewaySettings::get($this->identifier() . '_client_secret');
    $username = PaymentGatewaySettings::get($this->identifier() . '_username');
    $password = PaymentGatewaySettings::get($this->identifier() . '_password');
    
    if (empty($clientId) || empty($clientSecret)) {
        return ['success' => false, 'message' => 'Missing credentials'];
    }
    
    // Use HttpHelper::sendForm() because OAuth2 usually uses form data
    try {
        $response = HttpHelper::sendForm([
            'method' => 'POST',
            'url' => $this->getApiUrl('token'),
            'form_params' => [
                'grant_type' => 'password',
                'client_id' => $clientId,
                'client_secret' => $clientSecret,
                'username' => $username,
                'password' => $password,
            ],
            'options' => [
                'timeout' => 15,
            ],
        ]);
        
        $parsed = $this->parseResponse($response);
        
        if (isset($parsed['access_token'])) {
            return [
                'success' => true,
                'access_token' => $parsed['access_token'],
                'expires_in' => $parsed['expires_in'] ?? 3600,
                'token_type' => $parsed['token_type'] ?? 'Bearer',
            ];
        }
        
        return ['success' => false, 'message' => $parsed['error_description'] ?? 'Unknown error'];
    } catch (\Exception $e) {
        return ['success' => false, 'message' => $e->getMessage()];
    }
}
```

---

#### 4.4 Handling Token Expiration (401)

When executing any API request that may fail due to an expired token (HTTP 401), you should:
1. Clear the token from Cache.
2. Retry the request once with a new token.

**Helper function for requests with automatic token refresh:**

```php
private function requestWithTokenRefresh(array $requestConfig, callable $apiCall): array
{
    // First attempt using current token
    $result = $apiCall($this->getAuthToken());
    
    // If it fails due to token expiration
    if (isset($result['http_code']) && $result['http_code'] == 401) {
        // Clear the stored token
        Cache::forget($this->identifier() . '_access_token');
        
        // Obtain a new token
        $newToken = $this->getAuthToken(false); // false to bypass cache
        if ($newToken) {
            // Retry
            $result = $apiCall($newToken);
        }
    }
    
    return $result;
}
```

**Example usage:**
```php
$paymentResult = $this->requestWithTokenRefresh([], function($token) {
    return $this->createPayment($token);
});
```

---

#### 4.5 Storing Token in Database (Alternative to Cache)

In some environments where Cache is not reliable, the token can be stored in `payment_gateway_settings`:

```php
public function getAuthToken(): ?string
{
    $storedToken = PaymentGatewaySettings::get($this->identifier() . '_stored_token');
    $expiresAt = PaymentGatewaySettings::get($this->identifier() . '_token_expires_at');
    
    if ($storedToken && $expiresAt && time() < $expiresAt) {
        return $storedToken;
    }
    
    $newToken = $this->requestNewToken();
    if ($newToken['success']) {
        PaymentGatewaySettings::set($this->identifier() . '_stored_token', $newToken['access_token']);
        PaymentGatewaySettings::set($this->identifier() . '_token_expires_at', time() + $newToken['expires_in'] - 300);
        return $newToken['access_token'];
    }
    
    return null;
}
```

**However, the recommended method is to use Cache** because it is faster and puts less load on the database.

---

#### 4.6 Static Token (API Key) instead of OAuth

Some gateways use a static API Key sent in the header or as a query parameter. In this case, we do not need `getAuthToken()`; we use the key directly.

```php
private function getApiKey(): string
{
    return PaymentGatewaySettings::get($this->identifier() . '_api_key');
}

// In API requests
$response = HttpHelper::sendJson([
    'options' => [
        'headers' => [
            'X-API-Key' => $this->getApiKey(),
        ],
    ],
]);
```

---

#### 4.7 Token Security

- **Do not log tokens** – use `trace_log()` carefully and avoid printing the token.
- **Do not send tokens via URL** – always in the header `Authorization: Bearer {token}`.
- **Refresh the token before it expires** – using `Cache` with a lower TTL.
- **Use HTTPS exclusively** – all API requests must be over HTTPS.

---

#### 4.8 Token Management Checklist

- [ ] Do you use `Cache` to store the token?
- [ ] Do you define a `$cacheKey` unique using `identifier()`?
- [ ] Do you store the token with a lifetime shorter than its actual expiry?
- [ ] Do you handle `401` by clearing Cache and retrying?
- [ ] Do you validate credentials before requesting the token?
- [ ] Do you log errors when token acquisition fails?

---

### 5. Handling Redirection and `RedirectHelper`

#### 5.1 Overview

**Redirect** type payment gateways (e.g., ThawaniPay) redirect the user to an external page (bank portal, PayPal, etc.) to complete the payment, then return to your application via a `success` or `cancel` route. To make this process compatible with mobile applications that use Deep Links, `RedirectHelper` was created.

**What `RedirectHelper` does:**
- Extracts return URLs (`callback_success_url`, `callback_error_url`) from `order->other_data`.
- Supports Deep Links (e.g., `myapp://payment/success`) and regular web URLs (e.g., `https://example.com/success`).
- Stores large data in Cache to pass it through applications.
- Automatically adds query parameters to the URL.
- Checks allowed domains and schemes for security.

---

#### 5.2 Import `RedirectHelper`

```php
use Nano\Yepayment\Classes\RedirectHelper;
```

---

#### 5.3 Storing Return URLs in `process()`

Return URLs sent by the application (or browser) must be stored in `order->other_data` during `process()`.

```php
// Inside process() after successfully creating the transaction
$other = $this->order->other_data;
$other[$this->identifier()] = array_merge($other[$this->identifier()] ?? [], [
    'callback_success_url' => $this->data['callback_success_url'] ?? null,
    'callback_error_url'   => $this->data['callback_error_url'] ?? null,
]);
$this->order->other_data = $other;
$this->order->save();
```

**Source of these URLs:**
- When using an app, it sends `callback_success_url` as a Deep Link (e.g., `myapp://payment/success`).
- When using a browser, it sends a regular web URL.

---

#### 5.4 Extracting Return URLs using `RedirectHelper::getCallbackSuccessUrlByOrder`

**Signature:**
```php
RedirectHelper::getCallbackSuccessUrlByOrder($orderId, $keys, $default = null)
```

**Parameters:**
- `$orderId`: Order ID.
- `$keys`: Array of keys to search for in `other_data` (in order of preference).
- `$default`: Default value if no URL is found.

**Example:**
```php
$callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, [
    'callback_success_url', 
    'newpay.callback_success_url'
]);
```

---

#### 5.5 Redirecting using `RedirectHelper::redirectToApp`

**Signature:**
```php
RedirectHelper::redirectToApp($callbackUrl, $data, $forceJson = false, $queryParams = [], $deepLinkSchemes = ['*'], $isForceDeepList = false)
```

**Detailed parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `$callbackUrl` | string | Original URL (e.g., `myapp://success` or `https://example.com/callback`) |
| `$data` | array | Data to be passed (payment status, order number, etc.) |
| `$forceJson` | bool | If `true`, returns JSON instead of a redirect (for WebView usage) |
| `$queryParams` | array | Field names from `$data` to be added as query parameters (e.g., `['order_id', 'status']`) |
| `$deepLinkSchemes` | array | List of allowed schemes for apps (e.g., `['myapp', 'app']`). `['*']` means any scheme not http/https. |
| `$isForceDeepList` | bool | If `true`, forces the use of only the specified list. |

**How it works:**
1. If `$forceJson = true`, returns a JSON response containing the data.
2. If `$callbackUrl` starts with `http://` or `https://`, performs a normal HTTP redirect.
3. If `$callbackUrl` starts with another scheme (e.g., `myapp://`), builds a Deep Link:
   - Compresses large data and stores it in Cache with a random key.
   - Adds the key as a query parameter.
   - Adds any other query parameters specified in `$queryParams`.
   - Checks that the scheme is allowed (if `$deepLinkSchemes` does not contain `'*'` or the required scheme).
4. If no valid URL is found, returns a default textual response.

---

#### 5.6 Implementing the Full `success` Route

In `routes.php`, within the `yepayment` or `ompayment` group:

```php
Route::get('newpay/success', function () {
    // Route protection (optional, but recommended)
    // if (!BackendAuth::getUser() && !App::runningInBackend()) { ... }
    
    $options = Input::get();
    
    // 1. Complete payment via static function
    $result = NewPay::checkAndCompletePay($options);
    $order_id = $result['order_id'] ?? null;
    
    if (!$order_id) {
        return response()->json(['error' => 'Order not found'], 404);
    }
    
    // 2. Extract stored return URL
    $callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, [
        'callback_success_url',
        'newpay.callback_success_url'
    ]);
    
    // 3. Prepare data to send
    $responseData = [
        'success' => $result['success'],
        'message' => $result['message'],
        'order_id' => $order_id,
        'payment_status' => $result['success'] ? 'paid' : 'failed',
    ];
    
    // 4. Redirect using RedirectHelper
    if ($callbackUrl) {
        return RedirectHelper::redirectToApp(
            $callbackUrl,
            $responseData,
            false,                      // forceJson
            ['order_id', 'payment_status'], // queryParams
            ['*'],                      // deepLinkSchemes (all allowed)
            false                       // isForceDeepList
        );
    }
    
    // 5. If no return URL, return JSON or a simple success page
    if (request()->wantsJson()) {
        return response()->json($responseData);
    }
    
    return Response::make('Payment successful. You can return to the application.', 200);
});
```

---

#### 5.7 `cancel` Route (Payment Cancellation)

Similar to the `success` route, but uses `callback_error_url` and sends a failure status.

```php
Route::get('newpay/cancel', function () {
    $options = Input::get();
    $order_id = $options['order_id'] ?? null;
    
    if (!$order_id) {
        return response()->json(['error' => 'Order not found'], 404);
    }
    
    $callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, [
        'callback_error_url',
        'newpay.callback_error_url'
    ]);
    
    $responseData = [
        'success' => false,
        'message' => 'Payment cancelled by user',
        'order_id' => $order_id,
        'payment_status' => 'cancelled',
    ];
    
    if ($callbackUrl) {
        return RedirectHelper::redirectToApp($callbackUrl, $responseData, false, ['order_id', 'payment_status'], ['*'], false);
    }
    
    return response()->json($responseData);
});
```

---

#### 5.8 `checkAndCompletePay` (static) – Required for the `success` Route

To separate the confirmation logic from the route, this static function must be added in the gateway class:

```php
public static function checkAndCompletePay(array $options): array
{
    $orderId = $options['order_id'] ?? null;
    if (!$orderId) {
        return ['success' => false, 'message' => 'Order ID is required'];
    }
    
    $order = Order::find($orderId);
    if (!$order) {
        return ['success' => false, 'message' => 'Order not found'];
    }
    
    $instance = new self($order, $options);
    $result = new PaymentResult($instance, $order);
    $completeResult = $instance->complete($result);
    
    return [
        'success' => $completeResult->successful,
        'message' => $completeResult->message,
        'api_data' => $completeResult->api_data,
        'order_id' => $order->id,
    ];
}
```

---

#### 5.9 Supporting Deep Links with Mobile Applications

**Example Deep Link sent by the app:**
```
myapp://payment/success?order_id=123
```

**What `RedirectHelper` does:**
1. Extracts the scheme `myapp`.
2. Checks whether it is allowed (in `$deepLinkSchemes`).
3. Takes the data (`$responseData`) and stores it in Cache with a random key.
4. Builds a new URL: `myapp://payment/success?order_id=123&_cache_key=xyz123`
5. Redirects the user to this URL.
6. The app receives the URL and reads the data from Cache using `_cache_key`.

**Advantages of this approach:**
- Bypasses URL length limits (large data).
- Secure because data does not pass through the URL directly.
- Works on all operating systems (iOS, Android).

---

#### 5.10 Security of Return URLs

To avoid open redirect attacks, it is recommended to:
- Check that the return URL belongs to a trusted domain.
- Use a whitelist of allowed domains.

You can modify `RedirectHelper` to add this feature, or implement it before calling `redirectToApp`:

```php
$allowedDomains = ['example.com', 'api.example.com'];
$parsedUrl = parse_url($callbackUrl);
$host = $parsedUrl['host'] ?? '';
if (!in_array($host, $allowedDomains) && strpos($callbackUrl, 'http') === 0) {
    // External unauthorised URL
    return response()->json(['error' => 'Invalid callback URL'], 400);
}
```

---

#### 5.11 Checklist for Handling Redirect

- [ ] Did you store `callback_success_url` and `callback_error_url` in `order->other_data` during `process()`?
- [ ] Did you add the `static checkAndCompletePay` function in the gateway class?
- [ ] Did you add the `success` and `cancel` routes in `routes.php`?
- [ ] Did you use `RedirectHelper::getCallbackSuccessUrlByOrder` to extract the URLs?
- [ ] Did you use `RedirectHelper::redirectToApp` to redirect?
- [ ] Did you specify `$queryParams` correctly (fields to be added as query parameters)?
- [ ] Did you specify appropriate `$deepLinkSchemes` for your app?
- [ ] Did you test the scenarios: web, app, and WebView with `forceJson = true`?

---

#### 5.12 Practical Examples of Using `RedirectHelper`

##### Example 1: Simple redirect to a web URL
```php
$callbackUrl = 'https://example.com/thank-you';
return RedirectHelper::redirectToApp($callbackUrl, ['order_id' => 123], false, ['order_id'], ['*'], false);
// Result: HTTP redirect to https://example.com/thank-you?order_id=123
```

##### Example 2: Deep Link to a mobile app
```php
$callbackUrl = 'myapp://payment/result';
return RedirectHelper::redirectToApp($callbackUrl, ['order_id' => 123, 'status' => 'paid'], false, ['order_id', 'status'], ['myapp'], false);
// Result: redirect to myapp://payment/result?order_id=123&status=paid&_cache_key=abc123
```

##### Example 3: Return JSON instead of redirect (for WebView)
```php
return RedirectHelper::redirectToApp($callbackUrl, $data, true, [], ['*'], false);
// Result: JSON response with data
```

---

### 🏁 Summary of the Three Parts

| Part | Key Points |
|------|-------------|
| **HttpHelper** | Use `sendJson()` for JSON, `sendForm()` for Form Data, `get()` for queries, `parseResponse()` to convert responses, retry for temporary errors. |
| **Token Management** | Store in Cache with a shorter TTL, handle 401 and refresh, use gateway‑specific `requestNewToken()`. |
| **RedirectHelper** | Store return URLs in `other_data`, extract them with `getCallbackSuccessUrlByOrder`, redirect with `redirectToApp` supporting Deep Links, static `checkAndCompletePay` function. |

### 6. Understanding `PaymentResult` and Its Impact on Order Status and System Integration

#### 6.1 Concept of `PaymentResult`

`PaymentResult` is an intermediary class returned by both `process()` and `complete()` to express the outcome of the payment operation. Based on this result, `OrderManager` and `Checkout2` decide the next step (consider the order paid, wait for confirmation, redirect, or fail).

**Key properties of `PaymentResult`:**
- `successful`: bool – whether the operation technically succeeded.
- `redirect`: bool – whether redirection is required.
- `redirectUrl`: string – redirection URL if any.
- `message`: string – explanatory message for the user.
- `api_data`: array – raw data from the API (for internal use).

---

#### 6.2 `PaymentResult` Methods and Their Full Impact

| Method | Correct Usage | Impact on `Order` | Impact on System |
|--------|---------------|-------------------|------------------|
| `$result->success($data, $response)` | Only when the amount has been definitively and irreversibly deducted from the customer (except for refund). | `payment_state = PaidState::class`<br>`processed = true` | 1. Logs `PaymentLog` as success.<br>2. Fires event `nano.orders.paymentProcessed` (sends emails, updates inventory, notifications).<br>3. **Permanently clears the user's cart**. |
| `$result->fail($data, $response)` | When the operation fails definitively (insufficient balance, wrong PIN, authentication error). | `payment_state = FailedState::class` | 1. Logs `PaymentLog` as failure.<br>2. **Does not clear the cart**.<br>3. Does not fire the `paymentProcessed` event. |
| `$result->redirect($url)` | To redirect the user to an external gateway (bank, PayPal, external app). | **Does not change** `payment_state`<br>`processed` remains `false` | 1. Does not automatically log `PaymentLog` (you may log it manually).<br>2. Does not clear the cart. |
| `$result->pending($data, $response)` | When the transaction is pending (amount not yet deducted) but the cart should be cleared and retries prevented. | `payment_state = PendingState::class`<br>`processed = true` | 1. Logs `PaymentLog` as success with status `pending`.<br>2. **Clears the cart** (because the user cannot modify the cart after starting payment).<br>3. **Does not fire** the `paymentProcessed` event. |

> ⚠️ **Security warning:** Calling `$result->success()` in the wrong place (e.g., inside `process()` for a Two‑Step gateway before OTP confirmation) will mark the order as paid even though the customer has not actually paid, causing serious financial and business issues.

---

#### 6.3 When to Use Each Method by Gateway Type

| Gateway Type | In `process()` | In `complete()` |
|--------------|----------------|-----------------|
| **Direct** | `$result->success()` immediately after API success. | Not used. |
| **Two‑Step** | Either `$result->pending()` or `$result->successful = true` with `message` (without calling `success()`). | `$result->success()` only when confirmation succeeds; `$result->fail()` on failure. |
| **Redirect** | `$result->redirect($url)`. | `$result->success()` or `$result->fail()` after returning from external gateway. |

**Correct Two‑Step example (CashPay):**
```php
// In process()
$result->successful = true;
$result->message = 'Please enter the verification code sent to your phone';
return $result;  // $result->success() not called

// In complete()
if ($otpCorrect) {
    return $result->success($data, null);  // Only now the order becomes paid
} else {
    return $result->fail($data, null);
}
```

---

#### 6.4 Interaction of `PaymentResult` with `OrderManager` and `Checkout2`

**Context:** When the user reaches the payment step (`step=pay`) in the client application, `OrderManager::setStepPayments($data)` is called.

**Detailed flow:**
1. `Checkout2` calls `$this->orderManager->setStepPayments($data)`.
2. `OrderManager` validates previous steps (addresses, shipping, coupons).
3. A `PaymentGateway` and `PaymentService` object are created.
4. `$paymentService->process()` is called, which in turn calls `$gateway->process($order, $data)`.
5. After `PaymentResult` is returned:
   - If `$result->successful == true` and `$order->isPaymentProcessed() == true` (i.e., `payment_state` is `PaidState`) → payment is considered complete and the order moves to the completion step.
   - If `$result->successful == true` but `$order->isPaymentProcessed() == false` (e.g., Two‑Step after `process` only) → payment is not yet complete; an interface "Enter OTP" or "Wait for confirmation from external app" is shown.
   - If `$result->redirect == true` → redirect to `$result->redirectUrl`.
   - If `$result->successful == false` → display failure message.

**Conclusion:** `isPaymentProcessed()` is the true indicator of whether the order is paid. Do not rely on `$result->successful` alone.

---

#### 6.5 Automatic `PaymentLog` Logging via `PaymentResult`

When `$result->success()` is called, a `PaymentLog` is automatically logged with status `'success'`.  
When `$result->fail()` is called, a `PaymentLog` is automatically logged with status `'fail'`.  
When `$result->pending()` is called, a `PaymentLog` is automatically logged with status `'pending'`.

**If you need to manually log an initial attempt (before payment completion), use:**
```php
$paymentLog = $result->logSuccessfulPayment($apiData, null, 'initiated');
$this->order->payment_id = $paymentLog->id ?? null;
$this->order->save();
```

> This is useful in Two‑Step gateways to track payment attempts even if the customer never completes the process.

---

#### 6.6 Handling `processed` and `payment_state`

- `$this->order->processed` is set to `true` when `$result->success()` or `$result->pending()` is called.
- `$this->order->payment_state` changes to `PaidState::class` only when `$result->success()` is called.
- `$this->order->payment_state` changes to `PendingState::class` when `$result->pending()` is called.
- `$this->order->payment_state` changes to `FailedState::class` when `$result->fail()` is called.

**`isPaymentProcessed()` function in the `Order` model:**
```php
public function isPaymentProcessed(): bool
{
    return $this->payment_state == PaidState::class;
}
```

Therefore, do not use `$this->order->processed` as a criterion for successful payment; use `$this->order->isPaymentProcessed()`.

---

#### 6.7 Best Practices

| Scenario | What to do | What to avoid |
|----------|------------|----------------|
| Successfully creating a transaction in a Two‑Step gateway | `$result->successful = true; return $result;` | Calling `$result->success()` |
| Correct OTP confirmation | `return $result->success($data, null);` | Returning only `$result->successful = true` |
| OTP failure | `return $result->fail($data, null);` | Returning `$result->successful = false` without `fail()` |
| Redirect to external gateway | `return $result->redirect($url);` | Trying to execute `header()` directly |
| Pending transaction but cart must be cleared | `return $result->pending($data, null);` | Using only `successful = true` |

---

#### 6.8 Complete Example of Integration with `OrderManager`

**In the `pay` route (handled automatically by `Checkout2`):**  
No intervention needed; `OrderManager` handles `PaymentResult`.

**In your own `success` route (for Redirect or Two‑Step after confirmation):**  
Use `checkAndCompletePay`, which calls `complete()` then `$result->success()`, then `RedirectHelper`.

**Checking payment status after return:**
```php
$order = Order::find($orderId);
if ($order->isPaymentProcessed()) {
    // Payment complete, show thank you page
} else {
    // Not complete, perhaps still pending
}
```

---

### 7. Checking for Existing Transaction in `process()` – Idempotency

#### 7.1 Why We Need Idempotency

When a user reloads the payment page, clicks the "Pay" button multiple times, or resends the request from an app, `process()` may be called several times for the same order. Without an idempotency mechanism, multiple transactions may be created with the same payment provider for the same order, leading to:
- Deduction of the amount more than once (for some gateways).
- Confusion in transaction logs.
- Confusing error messages for the user.

**Solution:** Check whether there is an existing transaction associated with the order (`payment_first_trans_id` or data in `other_data`), then query its status and take appropriate action.

---

#### 7.2 Structure of Idempotency Check

```php
// Inside process(), after checking not already paid (PaidState)
$existingTransactionId = $this->order->payment_first_trans_id 
    ?? ($this->order->other_data[$this->identifier()]['transaction_id'] ?? null);

if ($existingTransactionId) {
    $statusResult = $this->checkTransactionStatus($existingTransactionId);
    
    // Analyse transaction status
    $transactionStatus = $statusResult['status'] ?? $statusResult['data']['status'] ?? null;
    
    switch ($transactionStatus) {
        case 'Complete':
        case 'Success':
        case 'Paid':
            // Transaction already completed → update order to paid
            return $result->success($statusResult, null);
            
        case 'Pending':
        case 'Initiated':
        case 'Processing':
            // Transaction is being processed → inform user to wait or redirect
            $result->successful = true;
            $result->message = trans('nano.yepayment::lang.public.payment_pending_continue');
            // If there is a follow‑up URL, use it
            if (isset($statusResult['redirect_url'])) {
                return $result->redirect($statusResult['redirect_url']);
            }
            return $result;
            
        case 'Failed':
        case 'Expired':
        case 'Cancelled':
        default:
            // Transaction failed or expired → can create a new one
            // do nothing, continue to create a new transaction.
            break;
    }
}
// Create a new transaction...
```

---

#### 7.3 Implementing `checkTransactionStatus()`

This function should contact the gateway's API to query the status of a transaction using its ID (`transaction_id` or `ref_no`).

```php
public function checkTransactionStatus(string $transactionId): array
{
    try {
        $token = $this->getAuthToken();
        if (!$token) {
            return ['success' => false, 'message' => 'Authentication failed'];
        }
        
        $response = HttpHelper::get([
            'url' => $this->getApiUrl('status') . '/' . $transactionId,
            'options' => [
                'headers' => ['Authorization' => 'Bearer ' . $token],
            ],
        ]);
        
        $parsed = $this->parseResponse($response);
        
        // Differs per gateway API, but generally:
        return [
            'success' => true,
            'status' => $parsed['status'] ?? $parsed['transaction_status'] ?? 'Unknown',
            'redirect_url' => $parsed['next_action']['redirect_url'] ?? null,
            'raw' => $parsed,
        ];
        
    } catch (\Exception $e) {
        return ['success' => false, 'message' => $e->getMessage()];
    }
}
```

**If the gateway does not support direct status query:**
```php
public function checkTransactionStatus(string $transactionId): array
{
    return [
        'success' => false,
        'supported' => false,
        'message' => 'This gateway does not support transaction status inquiry',
    ];
}
```

In that case, in the idempotency check, you can handle `supported == false` by ignoring the query and allowing a new transaction to be created (or returning an appropriate message).

---

#### 7.4 Special Cases

##### 7.4.1 Gateway does not return `Pending` but `Initiated`
Normalise the terms within a `normalizeStatus()` function.

##### 7.4.2 `payment_first_trans_id` exists but cannot be queried
In this case, you can assume the transaction is still valid if not much time has passed, or provide a "retry" option after a certain period.

##### 7.4.3 Transaction expired but gateway does not allow a new transaction for the same order
You may need to use a new reference (e.g., append `_retry` to `order_id`). This is rare, but can be handled in `createPayment()`.

---

#### 7.5 Using `getTransactionStatusForOrder` to Simplify the Query

Add a helper function that receives `order_id` and calls `checkTransactionStatus` using `payment_first_trans_id`.

```php
public function getTransactionStatusForOrder(string $orderId): array
{
    $order = Order::find($orderId);
    if (!$order) {
        return ['success' => false, 'message' => 'Order not found'];
    }
    $transactionId = $order->payment_first_trans_id;
    if (!$transactionId) {
        return ['success' => false, 'message' => 'No transaction associated'];
    }
    return $this->checkTransactionStatus($transactionId);
}
```

This function is useful in the test UI and admin tools.

---

#### 7.6 Idempotency Checklist

- [ ] Do you check for `payment_first_trans_id` or `other_data[identifier]['transaction_id']`?
- [ ] Do you call `checkTransactionStatus()` if the identifier exists?
- [ ] Do you handle the `Complete`, `Pending`, and `Failed` cases appropriately?
- [ ] Do you return `$result->success()` if the transaction is complete?
- [ ] Do you return `$result->pending()` or a confirmation message if pending?
- [ ] Do you log any action (e.g., reusing a transaction) in the logs?
- [ ] Do you handle gateways that do not support status query with `supported = false`?

---

### 8. Logging Payment Attempts (`PaymentLog`) in `process()` – Even When Not Completed

#### 8.1 Importance of Early Logging

In **Two‑Step** gateways (e.g., CashPay, YottaPay), the customer may create a transaction and then leave without entering the OTP. In this case, `complete()` is never called, and thus no `PaymentLog` is automatically logged (because `$result->success()` was never called). This makes it difficult to track:
- Number of failed payment attempts.
- Reasons why customers don't complete payments.
- Linking the initial payment log with the transaction if the customer returns later.

**Solution:** Manually log a `PaymentLog` in `process()` after successfully creating the transaction, with a status such as `'initiated'` or `'pending'`.

---

#### 8.2 How to Log `PaymentLog` in `process()`

```php
// After successfully saving order->payment_first_trans_id and order->other_data
try {
    $paymentLog = $result->logSuccessfulPayment($paymentResult, null, 'initiated');
    $this->order->payment_id = $paymentLog->id ?? null;
    $this->order->save();
} catch (\Throwable $e) {
    trace_log($this->identifier() . ': Failed to log PaymentLog: ' . $e->getMessage());
}
```

**Parameter explanation:**
- `$paymentResult`: raw data returned by the API (will be stored in the `data` field of `PaymentLog`).
- Second parameter (`null`): can be used to pass a `Response` object or any additional data.
- `'initiated'`: custom status indicating the transaction has been initiated but not yet completed.

---

#### 8.3 Suggested `PaymentLog` Statuses

| Status | When to Use | Method |
|--------|-------------|--------|
| `'initiated'` | After successfully creating a transaction in `process()` and payment not yet made. | `$result->logSuccessfulPayment(..., 'initiated')` |
| `'pending'` | Automatically when `$result->pending()` is called. | Automatic from `$result->pending()` |
| `'success'` | Automatically when `$result->success()` is called. | Automatic |
| `'fail'` | Automatically when `$result->fail()` is called. | Automatic |
| `'partial'` | For partial payments (rare). | Manual |

---

#### 8.4 Linking `order->payment_id` to the Payment Log

After logging the `PaymentLog`, it is advisable to store its ID in `order->payment_id`. This allows easy reference to the latest payment attempt.

```php
$this->order->payment_id = $paymentLog->id;
$this->order->save();
```

In `complete()`, you can either update the same record or add a new one. However, it is better to add a new record for each attempt (even for the same transaction) to track the full history of attempts.

---

#### 8.5 Viewing `PaymentLog` in the Control Panel

`PaymentLog` records:
- `created_at`: attempt time.
- `status`: status (`initiated`, `pending`, `success`, `fail`).
- `data`: raw data from the API (useful for debugging).
- `message`: error message if any.

This data can be used to analyse success rates, average completion time, and reasons for failure.

---

#### 8.6 Handling Multiple Attempts for the Same Order

If the user retries payment for the same order (after a previous failure or cancellation), a new `PaymentLog` will be created with `status = 'initiated'`. This is good because it allows tracking each attempt separately.

**If one of the attempts later succeeds:**
- A new `PaymentLog` with status `'success'` will be created.
- `order->payment_id` will point to the latest record if you update it each time.

---

#### 8.7 Complete Example of Logging in `process()` with Idempotency

```php
public function process(PaymentResult $result): PaymentResult
{
    try {
        // ... existing transaction check (section 7) ...
        
        // Create new transaction
        $paymentResult = $this->createPayment($token);
        if (!$paymentResult['success']) {
            throw new ApplicationException($paymentResult['message']);
        }
        
        // Save transaction data
        $this->order->payment_first_trans_id = $paymentResult['transaction_id'];
        $other = $this->order->other_data;
        $other[$this->identifier()] = [
            'transaction_id' => $paymentResult['transaction_id'],
            'callback_success_url' => $this->data['callback_success_url'] ?? null,
        ];
        $this->order->other_data = $other;
        $this->order->save();
        
        // Log payment attempt (even if not completed)
        try {
            $paymentLog = $result->logSuccessfulPayment($paymentResult, null, 'initiated');
            $this->order->payment_id = $paymentLog->id;
            $this->order->save();
        } catch (\Throwable $e) {
            trace_log($this->identifier() . ': Failed to log PaymentLog: ' . $e->getMessage());
        }
        
        // Return result based on type (Redirect/Two‑Step/Direct)
        if (isset($paymentResult['redirect_url'])) {
            return $result->redirect($paymentResult['redirect_url']);
        }
        if ($paymentResult['requires_confirmation'] ?? false) {
            $result->successful = true;
            $result->message = 'Please enter the verification code';
            return $result;
        }
        // Direct
        $this->order->payment_state = PaidState::class;
        $this->order->save();
        return $result->success($paymentResult, null);
        
    } catch (Throwable $e) {
        return $result->fail([], $e);
    }
}
```

---

#### 8.8 PaymentLog Logging Checklist

- [ ] Do you log a `PaymentLog` in `process()` after successfully creating the transaction?
- [ ] Do you use the status `'initiated'` for incomplete transactions?
- [ ] Do you save `order->payment_id` after logging?
- [ ] Do you handle exceptions from logging (e.g., database issues) without failing the entire payment process?
- [ ] Do you view `PaymentLog` in the test UI to show the attempt history?

---

### 🏁 Summary of Parts 6, 7, and 8

| Part | Key Points |
|------|-------------|
| **6. PaymentResult** | Understand the impact of `success`, `fail`, `redirect`, `pending` on `payment_state`, `processed`, cart, and events. Do not call `success()` in `process()` for Two‑Step. |
| **7. Idempotency** | Check for an existing transaction (`payment_first_trans_id`) and query its status. Handle `Complete`, `Pending`, `Failed` appropriately. |
| **8. Early PaymentLog** | Log a payment attempt in `process()` with `'initiated'` status even if not completed, to track and analyse attempts. |

### 9. Handling Gateway‑Specific Error Codes

#### 9.1 Why We Need Special Handling

Each payment gateway has a unique set of error codes that may carry special meanings not understood by the general system. Some of these codes may require human intervention or a specific action, such as changing the merchant's password, retrying with a different format, or redirecting the user to a specific page.

**Examples from real gateways:**
- **CashPay:** Code `6022` means the merchant's password has expired and must be changed.
- **BasPay:** Code `NotFound3000` means the transaction does not exist (possibly deleted or `trxToken` is invalid).
- **KuraimiPay:** Code `-1` may indicate an authentication error, while `1` indicates success.

---

#### 9.2 Structuring the Error Handling

The best practice is to create a dedicated function `handleSpecificErrors($responseData, $defaultMessage)` inside the gateway class, called after parsing the API response.

```php
private function handleSpecificErrors(array $responseData, string $defaultMessage): array
{
    // Extract error code from response (differs per gateway)
    $errorCode = $responseData['code'] ?? $responseData['error_code'] ?? $responseData['Code'] ?? null;
    $errorMessage = $responseData['message'] ?? $responseData['Message'] ?? $responseData['error_description'] ?? $defaultMessage;
    
    switch ($errorCode) {
        case 6022:
            return [
                'success' => false,
                'message' => 'Merchant password has expired. Please change it in the gateway settings.',
                'requires_password_change' => true,
                'error_code' => $errorCode,
                'raw' => $responseData,
            ];
        case 'NotFound3000':
            return [
                'success' => false,
                'message' => 'Transaction not found. It may be expired or invalid.',
                'requires_new_transaction' => true,
                'error_code' => $errorCode,
                'raw' => $responseData,
            ];
        case 401:
            return [
                'success' => false,
                'message' => 'Session expired or credentials are incorrect.',
                'requires_reauth' => true,
                'error_code' => $errorCode,
                'raw' => $responseData,
            ];
        case 429:
            return [
                'success' => false,
                'message' => 'Too many attempts. Please wait a while and try again.',
                'retry_after' => $responseData['retry_after'] ?? 60,
                'error_code' => $errorCode,
                'raw' => $responseData,
            ];
        default:
            // Ordinary errors
            return [
                'success' => false,
                'message' => $errorMessage,
                'error_code' => $errorCode,
                'raw' => $responseData,
            ];
    }
}
```

---

#### 9.3 Integrating Error Handling into API Methods

In every function that makes an API call (e.g., `createPayment`, `confirmPayment`, `getAuthToken`), after parsing the response, call `handleSpecificErrors` if the response indicates an error.

```php
private function createPayment(string $token): array
{
    try {
        $response = HttpHelper::sendJson([...]);
        $parsed = $this->parseResponse($response);
        
        // Check if response contains an error
        if (isset($parsed['error']) || isset($parsed['code']) && $parsed['code'] != 1) {
            return $this->handleSpecificErrors($parsed, 'Failed to create payment transaction');
        }
        
        return [
            'success' => true,
            'transaction_id' => $parsed['transaction_id'] ?? $parsed['ref_no'],
            'redirect_url' => $parsed['redirect_url'] ?? null,
            'requires_confirmation' => $parsed['requires_otp'] ?? false,
            'raw' => $parsed,
        ];
    } catch (\Exception $e) {
        return ['success' => false, 'message' => $e->getMessage()];
    }
}
```

---

#### 9.4 Handling Errors in `process()` and `complete()`

When you receive a result with, for example, `requires_password_change`, you can handle it specially:

```php
public function process(PaymentResult $result): PaymentResult
{
    try {
        $paymentResult = $this->createPayment($token);
        
        if (!$paymentResult['success']) {
            // Check whether the error requires special action
            if (isset($paymentResult['requires_password_change'])) {
                // Log a special error to notify the admin
                trace_log($this->identifier() . ': Password expired, please change it.');
                // Could send an email notification to the admin
                $this->notifyAdminPasswordExpired();
            }
            throw new ApplicationException($paymentResult['message']);
        }
        // ...
    } catch (Throwable $e) {
        return $result->fail([], $e);
    }
}
```

---

#### 9.5 Logging Special Errors

For future diagnosis, special errors should be logged in the system log:

```php
private function logSpecificError(array $errorData): void
{
    trace_log(json_encode([
        'gateway' => $this->identifier(),
        'error_code' => $errorData['error_code'] ?? 'unknown',
        'message' => $errorData['message'],
        'requires_action' => array_keys(array_filter($errorData, function($key) {
            return strpos($key, 'requires_') === 0;
        }, ARRAY_FILTER_USE_KEY)),
        'timestamp' => now()->toIso8601String(),
    ]));
}
```

---

### 10. Currency Support – Converting between ISO Code and ID

#### 10.1 The Problem

Payment gateways often expect a numeric currency ID (e.g., `2` for Yemeni Rial) instead of its code (`YER`). They may also support currencies in several ways:
- ISO code (e.g., `USD`, `EUR`, `YER`).
- Numeric ID (e.g., `1`, `2`, `3`).
- Localised text (e.g., `ريال يمني`).

To simplify handling, we add flexible conversion functions.

---

#### 10.2 Core Currency Support Functions

```php
/**
 * List of supported currencies with their IDs (according to the gateway's documentation)
 * 
 * @return array [currencyCode => currencyId]
 */
public function getSupportedCurrencies(): array
{
    return [
        'YER' => 2,   // Yemeni Rial
        'USD' => 1,   // US Dollar
        'SAR' => 3,   // Saudi Riyal
    ];
}

/**
 * List of supported currencies for display options (translated)
 */
public function getSupportedCurrenciesOptions(): array
{
    return [
        'YER' => '(2) ' . trans('nano.yepayment::lang.public.helpers.currencies.YER'),
        'USD' => '(1) ' . trans('nano.yepayment::lang.public.helpers.currencies.USD'),
        'SAR' => '(3) ' . trans('nano.yepayment::lang.public.helpers.currencies.SAR'),
    ];
}

/**
 * Get the numeric currency ID from its code, or validate a numeric ID
 * 
 * @param string|int|null $currency Currency code (e.g., 'YER') or currency ID (e.g., 2)
 * @return int|null Currency ID or null if invalid
 */
public function getCurrencyId($currency): ?int
{
    if ($currency === null || $currency === '') {
        return null;
    }
    
    $supported = $this->getSupportedCurrencies();
    
    // If input is numeric (int or numeric string)
    if (is_numeric($currency)) {
        $currencyId = (int) $currency;
        // Check if the number exists in the values of the array
        return in_array($currencyId, $supported, true) ? $currencyId : null;
    }
    
    // If input is a string, treat it as a currency code and convert to uppercase
    $currencyCode = strtoupper(trim((string) $currency));
    return $supported[$currencyCode] ?? null;
}

/**
 * Get the currency code from its numeric ID
 * 
 * @param int|null $currencyId Numeric currency ID
 * @return string|null Currency code or null if not found
 */
public function getCurrencyCode(?int $currencyId): ?string
{
    if ($currencyId === null) {
        return null;
    }
    
    $supported = $this->getSupportedCurrencies();
    $code = array_search($currencyId, $supported, true);
    return $code !== false ? $code : null;
}

/**
 * Check whether a currency is supported (accepts code or number)
 * 
 * @param string|int|null $currency Currency code or ID
 * @return bool
 */
public function isCurrencySupported($currency): bool
{
    return $this->getCurrencyId($currency) !== null;
}
```

---

#### 10.3 Using the Functions in `process()`

```php
public function process(PaymentResult $result): PaymentResult
{
    // Determine the currency used
    $currencyCode = $this->data['currency'] ?? 
                    $this->order->Currency->currency_code ?? 
                    PaymentGatewaySettings::get($this->identifier() . '_default_currency', 'YER');
    
    if (!$this->isCurrencySupported($currencyCode)) {
        throw new ApplicationException("Currency {$currencyCode} is not supported by the payment gateway");
    }
    
    $currencyId = $this->getCurrencyId($currencyCode);
    
    // Use $currencyId in building the API payload
    $payload = [
        'amount' => $this->order->total,
        'currency_id' => $currencyId,  // or 'currency' => $currencyCode depending on API
        // ...
    ];
    
    // Store currency data in order->other_data for tracking
    $other = $this->order->other_data;
    $other[$this->identifier()]['currency'] = [
        'code' => $currencyCode,
        'id' => $currencyId,
        'name' => $this->order->Currency->name ?? null,
    ];
    $this->order->other_data = $other;
    $this->order->save();
    
    // ...
}
```

---

#### 10.4 Ensuring Currency Matches Order Amount

In some gateways, you must ensure that the sent currency matches the order's currency recorded in the system:

```php
$orderCurrencyCode = $this->order->Currency->currency_code ?? 'YER';
if ($currencyCode !== $orderCurrencyCode) {
    // You can automatically convert currency if the gateway supports it
    // Or reject the operation
    throw new ApplicationException('Payment currency does not match order currency');
}
```

---

### 11. Generating Unique Transaction Identifiers (Idempotency Key)

#### 11.1 Why You Need a Unique Identifier

When creating a payment transaction, the gateway may receive duplicate requests (due to client retries or delayed responses). Without a unique identifier (Idempotency Key), multiple transactions may be created for the same order. The solution is to send a unique key with each transaction creation request, and the gateway will ignore subsequent requests with the same key.

---

#### 11.2 Generating a Unique Key

```php
protected function generateIdempotencyKey(): string
{
    // Use order_id + uuid to ensure uniqueness
    return $this->order->id . '_' . \Illuminate\Support\Str::uuid()->toString();
    
    // Or more simply:
    // return $this->order->id . '_' . uniqid();
}
```

**Best practices:**
- Use UUID to guarantee global uniqueness.
- Store the key in `order->other_data` if you need to reuse it (e.g., retrying the same transaction).
- Do not rely solely on `order_id` because the gateway may allow multiple transactions for the same order (e.g., failed attempts then a successful one).

---

#### 11.3 Sending the Key in the Request

```php
$idempotencyKey = $this->generateIdempotencyKey();

$response = HttpHelper::sendJson([
    'method' => 'POST',
    'url' => $this->getApiUrl('payment'),
    'json' => $payload,
    'options' => [
        'headers' => [
            'Authorization' => 'Bearer ' . $token,
            'Idempotency-Key' => $idempotencyKey,   // Header name depends on gateway requirements
            'Content-Type' => 'application/json',
        ],
    ],
]);
```

**Note:** The header name varies by gateway:
- It might be `Idempotency-Key` (GitHub, Stripe)
- Might be `X-Idempotency-Key`
- Might be `IdempotencyKey`
- Some gateways require it to be sent in the request body as a field `idempotency_key`.

---

#### 11.4 Storing the Key for Later Use

```php
$other = $this->order->other_data;
$other[$this->identifier()]['idempotency_key'] = $idempotencyKey;
$this->order->other_data = $other;
$this->order->save();
```

It can be used in a retry if the first request fails:

```php
// Inside createPayment, if the request fails due to timeout, retry with the same key
$idempotencyKey = $this->order->other_data[$this->identifier()]['idempotency_key'] ?? $this->generateIdempotencyKey();
```

---

### 12. Validating `validateSettings` and `isAvailable`

#### 12.1 `validateSettings()`

Used to ensure that all essential gateway settings (e.g., URLs, API keys, usernames) exist before starting any payment operation. It should be called at the beginning of `process()`, or can be called manually in `settings()`.

**Implementation:**

```php
public function validateSettings(): void
{
    $requiredFields = [
        $this->identifier() . '_url',
        $this->identifier() . '_api_key',
        $this->identifier() . '_secret_key',
    ];
    
    // If the gateway requires a username and password
    $requiredFields[] = $this->identifier() . '_username';
    $requiredFields[] = $this->identifier() . '_password';
    
    foreach ($requiredFields as $field) {
        $value = PaymentGatewaySettings::get($field, '');
        if (empty($value)) {
            throw new ApplicationException(
                trans('nano.yepayment::lang.public.missing_credentials', ['field' => $field])
            );
        }
    }
    
    // Additional validation such as URL format
    $url = PaymentGatewaySettings::get($this->identifier() . '_url');
    if (!filter_var($url, FILTER_VALIDATE_URL)) {
        throw new ApplicationException('Invalid API URL format.');
    }
}
```

**Usage in `process()`:**
```php
public function process(PaymentResult $result): PaymentResult
{
    try {
        $this->validateSettings(); // call at the beginning of process()
        // ...
    }
}
```

---

#### 12.2 `isAvailable()`

Used to determine whether the gateway is ready for use (e.g., credentials are correct, token can be obtained). Typically used in the list of available payment gateways shown to the customer.

```php
public function isAvailable(): bool
{
    try {
        // Attempt to obtain a token or make a simple test connection
        $token = $this->getAuthToken();
        return !empty($token);
    } catch (\Exception $e) {
        trace_log($this->identifier() . ' isAvailable check failed: ' . $e->getMessage());
        return false;
    }
}
```

**Caution:** If `getAuthToken()` is heavy (takes time), do not call it in `isAvailable()` unless necessary. Instead, you can just check for the existence of basic settings:

```php
public function isAvailable(): bool
{
    $requiredFields = ['url', 'api_key', 'username', 'password'];
    foreach ($requiredFields as $field) {
        if (empty(PaymentGatewaySettings::get($this->identifier() . '_' . $field))) {
            return false;
        }
    }
    return true;
}
```

---

### 13. `checkTransactionStatus` When Not Supported

Some payment gateways do not provide an API to directly query transaction status. In that case, the function should be implemented to indicate lack of support.

```php
public function checkTransactionStatus(string $refNo): array
{
    // If the gateway does not support status query
    if (!config($this->identifier() . '.support_status_check', false)) {
        return [
            'success' => false,
            'supported' => false,
            'message' => 'This gateway does not provide a transaction status inquiry API. Please use the reverse function to check if a refund is possible or rely on the order payment state.',
        ];
    }
    
    // If supported, perform the actual query
    try {
        $token = $this->getAuthToken();
        $response = HttpHelper::get([...]);
        $parsed = $this->parseResponse($response);
        return [
            'success' => true,
            'status' => $parsed['status'] ?? 'Unknown',
            'raw' => $parsed,
        ];
    } catch (\Exception $e) {
        return ['success' => false, 'message' => $e->getMessage()];
    }
}
```

**In `process()` when checking for an existing transaction, if `supported == false`, handle it like this:**
```php
if ($existingTransactionId) {
    $statusResult = $this->checkTransactionStatus($existingTransactionId);
    if (!$statusResult['success'] && isset($statusResult['supported']) && !$statusResult['supported']) {
        // Gateway does not support status query, give the user an option to retry or proceed with caution
        // Log a warning
        trace_log($this->identifier() . ': Status check not supported, proceeding cautiously.');
        // Continue to create a new transaction (or return a message)
    } elseif ($statusResult['success'] && $statusResult['status'] === 'Complete') {
        return $result->success($statusResult, null);
    }
}
```

---

### 14. Helper Function `getTransactionStatusForOrder`

This function simplifies querying transaction status using only the `order_id`, without needing to know the `transaction_id`.

```php
/**
 * Query the status of the transaction associated with the order
 * @param string $orderId
 * @return array
 */
public function getTransactionStatusForOrder(string $orderId): array
{
    $order = Order::find($orderId);
    if (!$order) {
        return ['success' => false, 'message' => 'Order not found'];
    }
    
    $transactionId = $order->payment_first_trans_id;
    if (!$transactionId) {
        return ['success' => false, 'message' => 'No transaction associated with this order'];
    }
    
    return $this->checkTransactionStatus($transactionId);
}
```

**Usage in the test UI or admin tools:**
```php
Route::get('newpay/test-check-status', function (Request $request) {
    if (!BackendAuth::getUser()) return Response::make('Access Denied', 404);
    $orderId = $request->input('order_id');
    $newpay = new NewPay();
    $status = $newpay->getTransactionStatusForOrder($orderId);
    return response()->json($status);
});
```

---

### 15. Encrypting and Encoding Sensitive Data

Some payment gateways require special encryption for sensitive data (e.g., `pin_pass`, `password`, `card_number`) before sending.

**Common examples:**

#### 15.1 Base64 encoding (KuraimiPay)

```php
$payload['PINPASS'] = base64_encode($this->data['pin_pass']);
```

#### 15.2 AES-256-CBC encryption (CashPay)

```php
private function encryptAes256Cbc(string $data, string $key): string
{
    $iv = substr(hash('sha256', $key), 0, 16);
    $encrypted = openssl_encrypt($data, 'AES-256-CBC', $key, OPENSSL_RAW_DATA, $iv);
    return base64_encode($encrypted);
}

// Usage
$encryptedPassword = $this->encryptAes256Cbc($password, $encryptionKey);
$headers['encPassword'] = $encryptedPassword;
```

#### 15.3 SHA256 hashing (for signatures)

```php
$signature = hash('sha256', $data . $secretKey);
```

#### 15.4 HMAC

```php
$signature = hash_hmac('sha256', $data, $secretKey);
```

**Golden rule:** Do not store data after encryption in the database; perform the conversion only at the time of sending. Keep the keys (encryption keys) in the gateway settings (already encrypted).

---

### 16. Implementing Refund (`reverse`)

If the gateway supports refunds, the `reverse` function must be implemented. This function is not abstract in the base `PaymentProvider`, but it is common in Yemeni and international gateways.

```php
/**
 * Refund a transaction amount
 * @param string $refNo Primary transaction reference (e.g., payment_first_trans_id)
 * @param string $reason Reason for refund
 * @param string|null $customerId Customer ID (if required)
 * @return array
 */
public function reverse(string $refNo, string $reason = '', ?string $customerId = null): array
{
    try {
        if (empty($refNo)) {
            return ['success' => false, 'message' => 'Transaction reference is required'];
        }
        
        // Retrieve scustId from other_data if previously stored
        $scustId = $customerId ?? ($this->order->other_data[$this->identifier()]['scust_id'] ?? null);
        
        $token = $this->getAuthToken();
        if (!$token) {
            return ['success' => false, 'message' => 'Authentication failed'];
        }
        
        $payload = [
            'REFNO' => $refNo,
            'REASON' => $reason,
        ];
        if ($scustId) {
            $payload['SCUSTID'] = $scustId;
        }
        
        $response = HttpHelper::sendJson([
            'method' => 'POST',
            'url' => $this->getApiUrl('reverse'),
            'json' => $payload,
            'options' => [
                'headers' => ['Authorization' => 'Bearer ' . $token],
            ],
        ]);
        
        $parsed = $this->parseResponse($response);
        
        // Check refund success (differs per gateway)
        if (($parsed['CODE'] ?? 0) == 1 || ($parsed['status'] ?? '') === 'success') {
            // Update order status to refunded
            $this->order->payment_state = \Nano\Orders\States\RefundedState::class;
            $other = $this->order->other_data;
            $other[$this->identifier()]['refund'] = [
                'ref_no' => $refNo,
                'reason' => $reason,
                'created_at' => now()->toISOString(),
                'refund_reference' => $parsed['refund_id'] ?? $parsed['PH_REF_NO'] ?? null,
            ];
            $this->order->other_data = $other;
            $this->order->save();
            
            return [
                'success' => true,
                'message' => 'Refund processed successfully',
                'refund_reference' => $parsed['refund_id'] ?? null,
                'raw' => $parsed,
            ];
        }
        
        return [
            'success' => false,
            'message' => $parsed['MESSAGE'] ?? $parsed['message'] ?? 'Refund failed',
            'raw' => $parsed,
        ];
        
    } catch (\Throwable $e) {
        return ['success' => false, 'message' => $e->getMessage()];
    }
}
```

**Notes:**
- `reverse` can be called from the control panel or from an external API.
- You should save `scustId` or any customer identifier in `other_data` during `process()` to be able to use it later for refund.
- After a successful refund, it is recommended to change the order status to `RefundedState` to prevent repeated refund attempts.

---

## 🧩 Creating Partial Files

### 1.1 File `_info.htm`

**Path:** `plugins/nano/yepayment/paymenttypes/newpay/_info.htm`

This file is displayed on the gateway settings page to provide informational and guidance content to the administrator.

```html
<div class="callout callout-info">
    <h4><i class="fa fa-credit-card"></i> {{ 'NewPay Gateway' | trans }}</h4>
    <p>{{ 'An integrated electronic payment gateway that allows accepting payments via credit cards and digital wallets.' | trans }}</p>
    <hr>
    <strong>{{ '📋 Setup Requirements:' | trans }}</strong>
    <ol>
        <li>{{ 'Register with NewPay gateway and obtain a merchant account.' | trans }}</li>
        <li>{{ 'Create an App from the dashboard to obtain Client ID and Client Secret.' | trans }}</li>
        <li>{{ 'Specify the base API URL (production or test).' | trans }}</li>
        <li>{{ 'Enter the merchant account username and password.' | trans }}</li>
        <li>{{ 'Set the default transaction currency (Yemeni Rial, Dollar, Saudi Riyal).' | trans }}</li>
    </ol>
    <hr>
    <strong>{{ '🔧 Supported Features:' | trans }}</strong>
    <ul>
        <li><i class="fa fa-check-circle text-green"></i> {{ 'Two‑Step payment with OTP' | trans }}</li>
        <li><i class="fa fa-check-circle text-green"></i> {{ 'Transaction status inquiry' | trans }}</li>
        <li><i class="fa fa-check-circle text-green"></i> {{ 'Refund via API' | trans }}</li>
        <li><i class="fa fa-check-circle text-green"></i> {{ 'Return URL (Deep Link) support for apps' | trans }}</li>
    </ul>
    <hr>
    <div class="alert alert-warning">
        <i class="fa fa-exclamation-triangle"></i> {{ 'Note: Ensure that your server IP address is whitelisted in the NewPay gateway.' | trans }}
    </div>
</div>
```

---

### 1.2 File `_test_info.htm`

**Path:** `plugins/nano/yepayment/paymenttypes/newpay/_test_info.htm`

Provides a quick testing tool within the settings page, with buttons to test authentication, create transactions, confirm, query, and refund.

```html
<div id="newpay-test-widget" class="payment-gateway-test-widget" style="padding: 15px; background: #f9f9f9; border-radius: 5px;">
    <div class="row">
        <div class="col-md-12">
            <div class="alert alert-info">
                <i class="fa fa-flask"></i> {{ 'Quick Test Tool - you can try payment gateways without needing a storefront.' | trans }}
            </div>
        </div>
    </div>
    
    <div class="row">
        <div class="col-md-6">
            <div class="form-group">
                <label><i class="fa fa-hashtag"></i> {{ 'Order ID:' | trans }}</label>
                <input type="number" id="newpay-test-order-id" class="form-control" placeholder="e.g., 200" value="200">
                <small class="text-muted">{{ 'The order must exist in the database.' | trans }}</small>
            </div>
            <div class="form-group">
                <label><i class="fa fa-money"></i> {{ 'Amount:' | trans }}</label>
                <input type="number" id="newpay-test-amount" class="form-control" placeholder="e.g., 100" value="100">
            </div>
            <div class="form-group">
                <label><i class="fa fa-globe"></i> {{ 'Currency:' | trans }}</label>
                <select id="newpay-test-currency" class="form-control">
                    <option value="YER">Yemeni Rial (YER)</option>
                    <option value="USD">US Dollar (USD)</option>
                    <option value="SAR">Saudi Riyal (SAR)</option>
                </select>
            </div>
        </div>
        <div class="col-md-6">
            <div class="form-group">
                <label><i class="fa fa-key"></i> {{ 'Verification Code (OTP):' | trans }}</label>
                <input type="text" id="newpay-test-code" class="form-control" placeholder="e.g., 123456" value="123456">
                <small class="text-muted">{{ 'Sent to the customer's phone when creating the transaction.' | trans }}</small>
            </div>
            <div class="form-group">
                <label><i class="fa fa-phone"></i> {{ 'Phone Number:' | trans }}</label>
                <input type="text" id="newpay-test-phone" class="form-control" placeholder="e.g., 777777777">
            </div>
            <div class="form-group">
                <label><i class="fa fa-link"></i> {{ 'Return URL (callback_success_url):' | trans }}</label>
                <input type="text" id="newpay-test-callback" class="form-control" placeholder="https://example.com/success">
                <small class="text-muted">{{ 'For apps (Deep Link) you can use: myapp://payment/success' | trans }}</small>
            </div>
        </div>
    </div>
    
    <div class="text-center" style="margin: 20px 0;">
        <button id="newpay-btn-auth" class="btn btn-info"><i class="fa fa-key"></i> {{ 'Test Authentication' | trans }}</button>
        <button id="newpay-btn-create" class="btn btn-primary"><i class="fa fa-credit-card"></i> {{ 'Create Transaction' | trans }}</button>
        <button id="newpay-btn-confirm" class="btn btn-success"><i class="fa fa-check-circle"></i> {{ 'Confirm Payment' | trans }}</button>
        <button id="newpay-btn-status" class="btn btn-warning"><i class="fa fa-search"></i> {{ 'Check Status' | trans }}</button>
        <button id="newpay-btn-refund" class="btn btn-danger"><i class="fa fa-undo"></i> {{ 'Refund' | trans }}</button>
        <button id="newpay-btn-clear" class="btn btn-default"><i class="fa fa-trash"></i> {{ 'Clear' | trans }}</button>
    </div>
    
    <div id="newpay-test-results" style="display: none; margin-top: 20px;">
        <div class="alert" id="newpay-test-alert"></div>
        <pre id="newpay-test-details" style="background: #2d2d2d; color: #f8f8f2; padding: 10px; border-radius: 5px; max-height: 350px; overflow: auto;"></pre>
    </div>
</div>

<script>
    const newpayApiBase = '/api/v1/yepayment';
    
    function showResult(data, isError = false) {
        const container = document.getElementById('newpay-test-results');
        const alertDiv = document.getElementById('newpay-test-alert');
        const detailsPre = document.getElementById('newpay-test-details');
        
        container.style.display = 'block';
        if (isError || (data && !data.success)) {
            alertDiv.className = 'alert alert-danger';
            alertDiv.innerHTML = '<i class="fa fa-exclamation-circle"></i> ' + (data.message || 'An unexpected error occurred');
        } else {
            alertDiv.className = 'alert alert-success';
            alertDiv.innerHTML = '<i class="fa fa-check-circle"></i> ' + (data.message || 'Operation successful');
        }
        detailsPre.textContent = JSON.stringify(data, null, 2);
        
        // Scroll to result
        container.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
    }
    
    function getPayload() {
        return {
            order_id: document.getElementById('newpay-test-order-id').value,
            amount: document.getElementById('newpay-test-amount').value,
            currency: document.getElementById('newpay-test-currency').value,
            otp: document.getElementById('newpay-test-code').value,
            phone: document.getElementById('newpay-test-phone').value,
            callback_success_url: document.getElementById('newpay-test-callback').value,
        };
    }
    
    // Test authentication
    document.getElementById('newpay-btn-auth')?.addEventListener('click', () => {
        fetch(`${newpayApiBase}/newpay/test-auth`, { method: 'POST' })
            .then(r => r.json())
            .then(d => showResult(d))
            .catch(e => showResult({ success: false, message: e.message }, true));
    });
    
    // Create transaction
    document.getElementById('newpay-btn-create')?.addEventListener('click', () => {
        fetch(`${newpayApiBase}/newpay/test-create-payment`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(getPayload())
        })
        .then(r => r.json())
        .then(d => showResult(d))
        .catch(e => showResult({ success: false, message: e.message }, true));
    });
    
    // Confirm payment
    document.getElementById('newpay-btn-confirm')?.addEventListener('click', () => {
        fetch(`${newpayApiBase}/newpay/test-confirm-payment`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(getPayload())
        })
        .then(r => r.json())
        .then(d => showResult(d))
        .catch(e => showResult({ success: false, message: e.message }, true));
    });
    
    // Check status
    document.getElementById('newpay-btn-status')?.addEventListener('click', () => {
        const orderId = document.getElementById('newpay-test-order-id').value;
        fetch(`${newpayApiBase}/newpay/test-check-status?order_id=${orderId}`)
            .then(r => r.json())
            .then(d => showResult(d))
            .catch(e => showResult({ success: false, message: e.message }, true));
    });
    
    // Refund
    document.getElementById('newpay-btn-refund')?.addEventListener('click', () => {
        const orderId = document.getElementById('newpay-test-order-id').value;
        fetch(`${newpayApiBase}/newpay/test-refund`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ order_id: orderId, reason: 'Test refund' })
        })
        .then(r => r.json())
        .then(d => showResult(d))
        .catch(e => showResult({ success: false, message: e.message }, true));
    });
    
    // Clear results
    document.getElementById('newpay-btn-clear')?.addEventListener('click', () => {
        document.getElementById('newpay-test-results').style.display = 'none';
        document.getElementById('newpay-test-alert').innerHTML = '';
        document.getElementById('newpay-test-details').textContent = '';
    });
</script>
```

---

## 🖥️ Creating the Integrated Testing Page (UI)

**Path:** `plugins/nano/yepayment/views/newpay-ui.htm`

This is a standalone page accessible via `/api/v1/yepayment/newpay/test-ui` (protected for administrators only). It contains tabs (Manual, Automated, Statistics, Logs) and uses advanced JavaScript.

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ 'NewPay Gateway Testing' | trans }}</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.0/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        body { background: #f0f2f5; padding: 20px; }
        .tab-content { background: white; padding: 20px; border: 1px solid #dee2e6; border-top: none; border-radius: 0 0 5px 5px; }
        .nav-tabs .nav-link { color: #495057; }
        .nav-tabs .nav-link.active { background-color: white; border-bottom-color: white; }
        pre { background: #2d2d2d; color: #f8f8f2; padding: 15px; border-radius: 8px; overflow-x: auto; }
        .badge-success { background-color: #28a745; }
        .badge-danger { background-color: #dc3545; }
        .badge-warning { background-color: #ffc107; }
    </style>
</head>
<body>
<div class="container-fluid">
    <div class="card shadow-sm">
        <div class="card-header bg-primary text-white">
            <h3 class="mb-0"><i class="fas fa-credit-card"></i> {{ 'NewPay Gateway Testing' | trans }}</h3>
        </div>
        <div class="card-body">
            <ul class="nav nav-tabs" id="newpayTabs">
                <li class="nav-item"><a class="nav-link active" data-bs-toggle="tab" href="#manual"><i class="fas fa-hand-pointer"></i> {{ 'Manual Test' | trans }}</a></li>
                <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#auto"><i class="fas fa-robot"></i> {{ 'Automated Test' | trans }}</a></li>
                <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#stats"><i class="fas fa-chart-line"></i> {{ 'Statistics' | trans }}</a></li>
                <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#logs"><i class="fas fa-history"></i> {{ 'Transaction Logs' | trans }}</a></li>
            </ul>
            <div class="tab-content">
                <!-- Manual Test Tab -->
                <div class="tab-pane fade show active" id="manual">
                    @include('nano.yepayment::paymenttypes.newpay._test_info')
                </div>
                
                <!-- Automated Test Tab -->
                <div class="tab-pane fade" id="auto">
                    <div class="alert alert-secondary">
                        <i class="fas fa-info-circle"></i> {{ 'This test creates a new order, executes payment, then automatically confirms (simulating a correct OTP).' | trans }}
                    </div>
                    <div class="row">
                        <div class="col-md-4">
                            <div class="form-group">
                                <label>{{ 'Number of test iterations:' | trans }}</label>
                                <input type="number" id="auto-iterations" class="form-control" value="1" min="1" max="10">
                            </div>
                        </div>
                        <div class="col-md-4">
                            <div class="form-group">
                                <label>{{ 'Starting order ID:' | trans }}</label>
                                <input type="number" id="auto-start-order" class="form-control" value="1000">
                            </div>
                        </div>
                        <div class="col-md-4">
                            <div class="form-group">
                                <label>&nbsp;</label>
                                <button id="runAutoTest" class="btn btn-success btn-block"><i class="fas fa-play"></i> {{ 'Run Automated Test' | trans }}</button>
                            </div>
                        </div>
                    </div>
                    <div id="autoResults" style="margin-top: 20px;"></div>
                </div>
                
                <!-- Statistics Tab -->
                <div class="tab-pane fade" id="stats">
                    <div class="row">
                        <div class="col-md-3">
                            <div class="card text-white bg-info mb-3">
                                <div class="card-body">
                                    <h5 class="card-title">{{ 'Total Attempts' | trans }}</h5>
                                    <p class="card-text display-4" id="stat-total">-</p>
                                </div>
                            </div>
                        </div>
                        <div class="col-md-3">
                            <div class="card text-white bg-success mb-3">
                                <div class="card-body">
                                    <h5 class="card-title">{{ 'Successful' | trans }}</h5>
                                    <p class="card-text display-4" id="stat-success">-</p>
                                </div>
                            </div>
                        </div>
                        <div class="col-md-3">
                            <div class="card text-white bg-danger mb-3">
                                <div class="card-body">
                                    <h5 class="card-title">{{ 'Failed' | trans }}</h5>
                                    <p class="card-text display-4" id="stat-fail">-</p>
                                </div>
                            </div>
                        </div>
                        <div class="col-md-3">
                            <div class="card text-white bg-warning mb-3">
                                <div class="card-body">
                                    <h5 class="card-title">{{ 'Pending' | trans }}</h5>
                                    <p class="card-text display-4" id="stat-pending">-</p>
                                </div>
                            </div>
                        </div>
                    </div>
                    <button class="btn btn-sm btn-secondary" id="refreshStats"><i class="fas fa-sync-alt"></i> {{ 'Refresh' | trans }}</button>
                </div>
                
                <!-- Logs Tab -->
                <div class="tab-pane fade" id="logs">
                    <div class="table-responsive">
                        <table class="table table-striped table-bordered">
                            <thead>
                                <tr><th>{{ 'ID' | trans }}</th><th>{{ 'Order' | trans }}</th><th>{{ 'Status' | trans }}</th><th>{{ 'Message' | trans }}</th><th>{{ 'Date' | trans }}</th></tr>
                            </thead>
                            <tbody id="logs-table-body">
                                <tr><td colspan="5" class="text-center">{{ 'Loading...' | trans }}</td></tr>
                            </tbody>
                        </table>
                    </div>
                    <button class="btn btn-sm btn-secondary" id="refreshLogs"><i class="fas fa-sync-alt"></i> {{ 'Refresh' | trans }}</button>
                </div>
            </div>
        </div>
    </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.0/js/bootstrap.bundle.min.js"></script>
<script>
    // Load statistics and logs automatically
    function loadStats() {
        fetch('/api/v1/yepayment/newpay/stats')
            .then(r => r.json())
            .then(data => {
                if (data.success) {
                    document.getElementById('stat-total').innerText = data.total_attempts || 0;
                    document.getElementById('stat-success').innerText = data.success_count || 0;
                    document.getElementById('stat-fail').innerText = data.failed_count || 0;
                    document.getElementById('stat-pending').innerText = data.pending_count || 0;
                }
            })
            .catch(e => console.error(e));
    }
    
    function loadLogs() {
        fetch('/api/v1/yepayment/newpay/logs')
            .then(r => r.json())
            .then(data => {
                const tbody = document.getElementById('logs-table-body');
                if (data.logs && data.logs.length) {
                    tbody.innerHTML = data.logs.map(log => `
                        <tr>
                            <td>${log.id}</td>
                            <td>${log.order_id || '-'}</td>
                            <td><span class="badge bg-${log.status === 'success' ? 'success' : (log.status === 'fail' ? 'danger' : 'warning')}">${log.status}</span></td>
                            <td>${log.message || '-'}</td>
                            <td>${new Date(log.created_at).toLocaleString()}</td>
                        </tr>
                    `).join('');
                } else {
                    tbody.innerHTML = '<tr><td colspan="5" class="text-center">No logs found</td></tr>';
                }
            })
            .catch(e => console.error(e));
    }
    
    document.getElementById('refreshStats')?.addEventListener('click', loadStats);
    document.getElementById('refreshLogs')?.addEventListener('click', loadLogs);
    
    // Automated test
    document.getElementById('runAutoTest')?.addEventListener('click', async () => {
        const iterations = parseInt(document.getElementById('auto-iterations').value);
        const startOrder = parseInt(document.getElementById('auto-start-order').value);
        const resultsDiv = document.getElementById('autoResults');
        resultsDiv.innerHTML = '<div class="alert alert-info">Running test...</div>';
        
        let results = [];
        for (let i = 0; i < iterations; i++) {
            const orderId = startOrder + i;
            // Simulate creating an order (you can call an API to create a dummy order)
            const createRes = await fetch('/api/v1/yepayment/newpay/test-create-payment', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ order_id: orderId, amount: 10, currency: 'YER' })
            }).then(r => r.json());
            
            if (createRes.success) {
                // Confirm payment
                const confirmRes = await fetch('/api/v1/yepayment/newpay/test-confirm-payment', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ order_id: orderId, otp: '123456' })
                }).then(r => r.json());
                results.push({ orderId, success: confirmRes.success, message: confirmRes.message });
            } else {
                results.push({ orderId, success: false, message: createRes.message });
            }
        }
        
        resultsDiv.innerHTML = `<pre>${JSON.stringify(results, null, 2)}</pre>`;
        loadStats();
        loadLogs();
    });
    
    loadStats();
    loadLogs();
</script>
</body>
</html>
```

---

## 🛣️ Adding Routes in `routes.php`

**Location:** `plugins/nano/yepayment/routes.php` (within the `yepayment` or `ompayment` group)

All required routes must be added with exception handling using `try-catch` and returning a unified response.

```php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use Nano\MicroCart\Models\Order;
use Nano\Yepayment\Classes\RedirectHelper;
use Nano\Yepayment\PaymentTypes\NewPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;
use Nano\MicroCart\Models\PaymentLog;
use BackendAuth;
use Response;
use Validator;
use Exception;

// Route group for NewPay
Route::group(['prefix' => 'newpay', 'namespace' => 'Nano\Yepayment\Http\Controllers'], function () {
    
    /**
     * Payment success route (return from external gateway)
     */
    Route::get('success', function () {
        try {
            $options = Input::get();
            // Validate presence of order_id
            $validator = Validator::make($options, ['order_id' => 'required|exists:orders,id']);
            if ($validator->fails()) {
                throw new Exception('Invalid data: ' . $validator->errors()->first());
            }
            
            // Complete payment via static function
            $result = NewPay::checkAndCompletePay($options);
            $orderId = $result['order_id'] ?? null;
            
            if (!$orderId) {
                throw new Exception('Order ID not found');
            }
            
            // Extract stored return URL
            $callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($orderId, [
                'callback_success_url',
                'newpay.callback_success_url'
            ]);
            
            $responseData = [
                'success' => $result['success'],
                'message' => $result['message'],
                'order_id' => $orderId,
                'payment_status' => $result['success'] ? 'paid' : 'failed',
                'timestamp' => now()->toIso8601String(),
            ];
            
            if ($callbackUrl) {
                return RedirectHelper::redirectToApp(
                    $callbackUrl,
                    $responseData,
                    false,
                    ['order_id', 'payment_status'],
                    ['*'],
                    false
                );
            }
            
            // If no return URL, return JSON
            return response()->json($responseData);
            
        } catch (Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Error processing payment: ' . $e->getMessage(),
                'code' => $e->getCode(),
            ], 500);
        }
    });
    
    /**
     * Payment cancel route
     */
    Route::get('cancel', function () {
        try {
            $orderId = Input::get('order_id');
            if (!$orderId) {
                throw new Exception('Order ID is required');
            }
            
            $callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($orderId, [
                'callback_error_url',
                'newpay.callback_error_url'
            ]);
            
            $responseData = [
                'success' => false,
                'message' => 'Payment cancelled by user',
                'order_id' => $orderId,
                'payment_status' => 'cancelled',
                'timestamp' => now()->toIso8601String(),
            ];
            
            if ($callbackUrl) {
                return RedirectHelper::redirectToApp(
                    $callbackUrl,
                    $responseData,
                    false,
                    ['order_id', 'payment_status'],
                    ['*'],
                    false
                );
            }
            
            return response()->json($responseData);
            
        } catch (Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Error: ' . $e->getMessage(),
            ], 400);
        }
    });
    
    /**
     * Test authentication (admin only)
     */
    Route::post('test-auth', function () {
        if (!BackendAuth::getUser()) {
            return response()->json(['success' => false, 'message' => 'Unauthorised'], 403);
        }
        try {
            $gateway = new NewPay();
            $token = $gateway->getAuthToken(false); // bypass cache
            return response()->json([
                'success' => !empty($token),
                'message' => $token ? 'Token obtained successfully' : 'Failed to obtain token',
                'token_preview' => $token ? substr($token, 0, 10) . '...' : null,
            ]);
        } catch (Exception $e) {
            return response()->json(['success' => false, 'message' => $e->getMessage()], 500);
        }
    });
    
    /**
     * Create a test transaction
     */
    Route::post('test-create-payment', function (Request $request) {
        if (!BackendAuth::getUser()) {
            return response()->json(['success' => false, 'message' => 'Unauthorised'], 403);
        }
        try {
            $order = Order::find($request->input('order_id'));
            if (!$order) {
                return response()->json(['success' => false, 'message' => 'Order not found']);
            }
            $gateway = new NewPay($order, $request->all());
            $result = new PaymentResult($gateway, $order);
            $processResult = $gateway->process($result);
            
            return response()->json([
                'success' => $processResult->successful,
                'message' => $processResult->message,
                'redirect_url' => $processResult->redirectUrl,
                'api_data' => $processResult->api_data,
            ]);
        } catch (Exception $e) {
            return response()->json(['success' => false, 'message' => $e->getMessage()], 500);
        }
    });
    
    /**
     * Test payment confirmation
     */
    Route::post('test-confirm-payment', function (Request $request) {
        if (!BackendAuth::getUser()) {
            return response()->json(['success' => false, 'message' => 'Unauthorised'], 403);
        }
        try {
            $order = Order::find($request->input('order_id'));
            if (!$order) {
                return response()->json(['success' => false, 'message' => 'Order not found']);
            }
            $gateway = new NewPay($order, $request->all());
            $result = new PaymentResult($gateway, $order);
            $completeResult = $gateway->complete($result);
            
            return response()->json([
                'success' => $completeResult->successful,
                'message' => $completeResult->message,
                'api_data' => $completeResult->api_data,
            ]);
        } catch (Exception $e) {
            return response()->json(['success' => false, 'message' => $e->getMessage()], 500);
        }
    });
    
    /**
     * Check transaction status
     */
    Route::get('test-check-status', function (Request $request) {
        if (!BackendAuth::getUser()) {
            return response()->json(['success' => false, 'message' => 'Unauthorised'], 403);
        }
        try {
            $orderId = $request->input('order_id');
            $gateway = new NewPay();
            $status = $gateway->getTransactionStatusForOrder($orderId);
            return response()->json($status);
        } catch (Exception $e) {
            return response()->json(['success' => false, 'message' => $e->getMessage()], 500);
        }
    });
    
    /**
     * Refund an amount
     */
    Route::post('test-refund', function (Request $request) {
        if (!BackendAuth::getUser()) {
            return response()->json(['success' => false, 'message' => 'Unauthorised'], 403);
        }
        try {
            $order = Order::find($request->input('order_id'));
            if (!$order) {
                return response()->json(['success' => false, 'message' => 'Order not found']);
            }
            $gateway = new NewPay($order);
            $result = $gateway->reverse($order->payment_first_trans_id, $request->input('reason', ''));
            return response()->json($result);
        } catch (Exception $e) {
            return response()->json(['success' => false, 'message' => $e->getMessage()], 500);
        }
    });
    
    /**
     * Gateway statistics (admin only)
     */
    Route::get('stats', function () {
        if (!BackendAuth::getUser()) {
            return response()->json(['success' => false, 'message' => 'Unauthorised'], 403);
        }
        try {
            $logs = PaymentLog::where('payment_method', 'newpay');
            return response()->json([
                'success' => true,
                'total_attempts' => (clone $logs)->count(),
                'success_count' => (clone $logs)->where('status', 'success')->count(),
                'failed_count' => (clone $logs)->where('status', 'fail')->count(),
                'pending_count' => (clone $logs)->where('status', 'pending')->count(),
            ]);
        } catch (Exception $e) {
            return response()->json(['success' => false, 'message' => $e->getMessage()], 500);
        }
    });
    
    /**
     * Transaction logs (admin only)
     */
    Route::get('logs', function () {
        if (!BackendAuth::getUser()) {
            return response()->json(['success' => false, 'message' => 'Unauthorised'], 403);
        }
        try {
            $logs = PaymentLog::where('payment_method', 'newpay')
                ->latest()
                ->limit(100)
                ->get(['id', 'order_id', 'status', 'message', 'created_at']);
            return response()->json(['success' => true, 'logs' => $logs]);
        } catch (Exception $e) {
            return response()->json(['success' => false, 'message' => $e->getMessage()], 500);
        }
    });
    
    /**
     * Integrated testing page (UI)
     */
    Route::get('test-ui', function () {
        if (!BackendAuth::getUser()) {
            return Response::make('Unauthorised', 403);
        }
        try {
            return View::make('nano.yepayment::newpay-ui');
        } catch (Exception $e) {
            return Response::make('Error loading page: ' . $e->getMessage(), 500);
        }
    });
});
```

---

## 📦 Registering the Gateway in `Plugin.php`

**Location:** `plugins/nano/yepayment/Plugin.php`

```php
<?php namespace Nano\Yepayment;

use System\Classes\PluginBase;
use Nano\Yepayment\PaymentTypes\NewPay;

class Plugin extends PluginBase
{
    // ... other methods ...
    
    /**
     * Register available payment providers
     */
    public function registerPaymentProviders()
    {
        $providers = [];
        
        // Register NewPay gateway if the class exists
        if (class_exists(NewPay::class)) {
            $providers[] = new NewPay();
        }
        
        // You can also register other gateways like:
        // $providers[] = new YottaPay();
        // $providers[] = new BasPay();
        
        return $providers;
    }
    
    /**
     * If you use geographical zones (Yemen, Oman), you can distribute gateways as follows:
     */
    public function registerYemenPaymentProviders()
    {
        $providers = [];
        if (class_exists(NewPay::class)) {
            $providers[] = new NewPay();
        }
        // Other Yemen‑specific gateways
        return $providers;
    }
    
    public function registerOmanPaymentProviders()
    {
        // Oman gateways
        return [];
    }
}
```

**Note:** The exact method name may vary depending on the system version (`registerPaymentProviders`, `registerYemenPaymentProviders`, `registerOmanPaymentProviders`). Check the original `Nano.Yepayment` code and use the one that fits.

---

## 🌐 Adding Translation Keys

### File `lang/ar/lang.php`

```php
<?php

return [
    // ... existing keys ...
    
    'payment_gateway_settings' => [
        'newpay' => [
            'url' => 'API Base URL (Production)',
            'test_url' => 'API Base URL (Sandbox)',
            'api_key' => 'API Key',
            'secret_key' => 'Secret Key',
            'username' => 'Username',
            'password' => 'Password',
            'default_currency' => 'Default Currency',
            'client_id' => 'Client ID',
            'client_secret' => 'Client Secret',
        ],
    ],
    
    'public' => [
        'newpay' => [
            'payment_success' => 'Payment successful',
            'confirmation_required' => 'Please enter the verification code sent to your phone',
            'payment_pending_continue' => 'Payment is being processed, please continue',
            'default_note' => 'Confirm payment via NewPay',
            'yes' => 'Yes',
            'no' => 'No',
            'errors' => [
                'auth_failed' => 'Authentication with payment gateway failed',
                'order_already_paid' => 'This order has already been paid',
                'missing_credentials' => 'Missing credentials: :field',
                'payment_creation_failed' => 'Failed to create payment transaction',
                'payment_confirmation_failed' => 'Payment confirmation failed, please check the code',
                'incomplete_payment_data' => 'Incomplete payment data',
                'password_expired' => 'Merchant password expired. Please change it in gateway settings',
                'transaction_not_found' => 'Transaction not found',
                'refund_failed' => 'Refund failed',
            ],
        ],
    ],
];
```

### File `lang/en/lang.php`

```php
<?php

return [
    // ... existing keys ...
    
    'payment_gateway_settings' => [
        'newpay' => [
            'url' => 'API Base URL (Production)',
            'test_url' => 'API Base URL (Sandbox)',
            'api_key' => 'API Key',
            'secret_key' => 'Secret Key',
            'username' => 'Username',
            'password' => 'Password',
            'default_currency' => 'Default Currency',
            'client_id' => 'Client ID',
            'client_secret' => 'Client Secret',
        ],
    ],
    
    'public' => [
        'newpay' => [
            'payment_success' => 'Payment successful',
            'confirmation_required' => 'Please enter the verification code sent to your phone',
            'payment_pending_continue' => 'Payment is being processed, please continue',
            'default_note' => 'Confirm payment via NewPay',
            'yes' => 'Yes',
            'no' => 'No',
            'errors' => [
                'auth_failed' => 'Authentication with payment gateway failed',
                'order_already_paid' => 'This order has already been paid',
                'missing_credentials' => 'Missing credentials: :field',
                'payment_creation_failed' => 'Failed to create payment transaction',
                'payment_confirmation_failed' => 'Payment confirmation failed, please check the code',
                'incomplete_payment_data' => 'Incomplete payment data',
                'password_expired' => 'Merchant password expired. Please change it in gateway settings',
                'transaction_not_found' => 'Transaction not found',
                'refund_failed' => 'Refund failed',
            ],
        ],
    ],
];
```

---

## 🧪 Additional Tips for Developing Professional Gateways

### 1. Security

#### 1.1 Input Validation
- Use `Validator` with `defineValidationRules()` to validate all incoming data (`order_id`, `otp`, `callback_success_url`).
- Never trust any client data; verify its existence, correctness, and allowed ranges.

#### 1.2 Protect Test Routes
- All test routes (`/test-*`, `/stats`, `/logs`, `/test-ui`) must be protected with `BackendAuth::getUser()`.
- In production, you may add additional checks for IP address or user role.

#### 1.3 Prevent Open Redirect Attacks
- Use `RedirectHelper::redirectToApp`, which checks allowed schemes.
- Or add a whitelist of allowed domains (`$allowedDomains`) and validate before redirecting.

#### 1.4 Secure Storage of Sensitive Data
- Use `encryptedSettings()` for fields of type `password` and `secret_key`.
- Never store passwords or API keys in `other_data` in plain text.
- If you need to store sensitive data in `other_data` (e.g., `scustId`, `pin_pass`), ensure it does not appear in logs.

#### 1.5 HTTPS Mandatory
- Ensure all API requests use HTTPS.
- In the test environment, you may disable certificate verification (`'verify' => false`) only when absolutely necessary.

#### 1.6 Never Log Tokens
- Never log full tokens.
- If you need to log them for debugging, truncate them (e.g., `substr($token, 0, 10) . '...'`).

---

### 2. Performance

#### 2.1 Use Cache for Tokens
- Store tokens in Cache (Redis / File) with a TTL at least 5 minutes shorter than the actual expiry.
- Use a unique cache key per gateway (`identifier() . '_token'`).

#### 2.2 Avoid Repeated Requests
- In `process()`, check for an existing transaction (`payment_first_trans_id`) before creating a new one to avoid unnecessary API calls.

#### 2.3 Set Appropriate Timeouts
- Use `timeout` of 30 seconds for sensitive payment requests.
- For secondary requests (e.g., status query), you can use 15 seconds.

#### 2.4 Smart Retry
- For requests that may fail temporarily (network issues, server timeouts), use a loop with `usleep()`.
- Do not retry more than 3 times.
- Use `Cache::forget` upon receiving 401 (token expired) and retry once.

---

### 3. Documentation

#### 3.1 Document Functions Inside the Class
- Use PHPDoc for every public and private function.
- Explain the meaning of each parameter, return type, and possible exceptions.

#### 3.2 Document Gateway Settings
- In `_info.htm`, explain in detail how to obtain each setting (Client ID, Secret, etc.).
- Add links to registration and documentation pages of the gateway if available.

#### 3.3 Document Special Error Codes
- Add a table of error codes and their meanings in `_info.htm` or in a separate `_errors.htm` file.

#### 3.4 Create a `README.md` File
- Inside the gateway folder (`paymenttypes/newpay/`), create a `README.md` containing:
  - Gateway description
  - Installation and setup steps
  - Example API requests
  - List of error codes
  - Technical support contact information

---

### 4. Testing

#### 4.1 Test All Possible Scenarios
- Successful payment (Direct, Two‑Step, Redirect)
- Failed payment (wrong OTP, insufficient balance)
- Retrying payment for the same order (idempotency)
- Token expiration and handling
- Refund
- Returning from external gateway (`success` and `cancel` routes)
- No return URL (fallback)

#### 4.2 Automated Testing in `test-ui`
- Use the "Automated Test" tab to run several sequential requests.
- Add simulation for correct and incorrect OTP.

#### 4.3 Test Compatibility with Mobile Apps
- Test Deep Links using an emulator or tools like `adb` (for Android) or `xcrun` (for iOS).
- Ensure `RedirectHelper::redirectToApp` redirects correctly.

#### 4.4 Security Testing
- Attempt to call test routes without logging in (should be denied).
- Attempt to send `callback_success_url` to an unauthorised external domain (should be rejected or warned).

---

### 5. Logging and Monitoring

#### 5.1 Use `trace_log()` Wisely
- Log only what is necessary for debugging.
- Log the start and end of each major function (`process`, `complete`, `getAuthToken`).
- In case of errors, log the full API response.

#### 5.2 Create a Custom Log for the Gateway
- Use `PaymentLog` to record payment attempts.
- Add a `meta` field to store additional data (e.g., `transaction_id`, `error_code`).

#### 5.3 Alerts
- If a specific error recurs (e.g., 6022 – password expired), send an automatic email to the administrator.
- Use `Mail::send()` or a system notification.

---

### 6. Handling Partial Payments and Currencies

#### 6.1 Support Partial Payments
- If the gateway supports paying part of the amount, add a `partialPayment()` function.
- Store the remaining amount in `other_data`.

#### 6.2 Dynamic Currency Conversion
- If the customer chooses a currency different from the order’s base currency, use an API to convert the price.
- Add a `convertCurrency($amount, $from, $to)` function.

---

### 7. Handling Webhooks

#### 7.1 Why Webhooks Are Needed
Some gateways send notifications (webhooks) when the transaction status changes (e.g., payment confirmation some time after creation). This is especially useful for asynchronous gateways.

#### 7.2 Implementing a Webhook Route
```php
Route::post('newpay/webhook', function (Request $request) {
    try {
        $payload = $request->all();
        // Verify signature if present
        $signature = $request->header('X-Signature');
        if (!$this->verifyWebhookSignature($payload, $signature)) {
            return response()->json(['error' => 'Invalid signature'], 401);
        }
        
        $transactionId = $payload['transaction_id'];
        $order = Order::where('payment_first_trans_id', $transactionId)->first();
        if (!$order) {
            return response()->json(['error' => 'Order not found'], 404);
        }
        
        // Update order status if not already paid
        if (!$order->isPaymentProcessed() && $payload['status'] === 'completed') {
            $gateway = new NewPay($order);
            $result = new PaymentResult($gateway, $order);
            $gateway->complete($result);
        }
        
        return response()->json(['success' => true]);
    } catch (Exception $e) {
        trace_log('Webhook error: ' . $e->getMessage());
        return response()->json(['error' => $e->getMessage()], 500);
    }
});
```

#### 7.3 Verifying Webhook Signature
```php
private function verifyWebhookSignature(array $payload, ?string $signature): bool
{
    $secret = PaymentGatewaySettings::get($this->identifier() . '_webhook_secret');
    $computed = hash_hmac('sha256', json_encode($payload), $secret);
    return hash_equals($computed, $signature);
}
```

---

### 8. Versioning Support

- If the gateway’s API changes, add a `getApiVersion()` function and send it in headers.
- Keep support for the old version for a transitional period.

---

### 9. Additional Helper Tools

#### 9.1 `getOrderCurrency()` – Safely Extract Order Currency
```php
protected function getOrderCurrency(): array
{
    $currency = $this->order->Currency ?? null;
    $code = $currency->currency_code ?? 'YER';
    $id = $this->getCurrencyId($code);
    return ['code' => $code, 'id' => $id, 'object' => $currency];
}
```

#### 9.2 `logApiCall()` – Structured Logging of API Calls
```php
protected function logApiCall(string $endpoint, array $request, $response, float $duration): void
{
    trace_log(json_encode([
        'gateway' => $this->identifier(),
        'endpoint' => $endpoint,
        'request' => $request,
        'response' => $response,
        'duration_ms' => round($duration * 1000, 2),
        'timestamp' => now()->toIso8601String(),
    ]));
}
```

---

## ✅ Comprehensive Checklist for Creating a New Gateway

### 🔹 Phase 1: Basic Setup

- [ ] Create the class file `XxxPay.php` in `paymenttypes/`.
- [ ] Ensure the class extends `PaymentProvider`.
- [ ] Set mandatory properties:
  - `$success_url = 'api/v1/yepayment/xxxpay/success';`
  - `$cancel_url = 'api/v1/yepayment/xxxpay/cancel';`
  - `$is_test_mod = false;`
- [ ] Implement `name(): string` (return commercial name).
- [ ] Implement `identifier(): string` (unique identifier with lowercase Latin letters).
- [ ] Implement `validate(): bool` (prefer `return true`).
- [ ] Implement `settings(): array` (with fields for `partials`, `text`, `password`, `dropdown`).
- [ ] Implement `encryptedSettings(): array` (for `password` fields).
- [ ] Implement `defineValidationRules(): array` (for client data validation).
- [ ] Implement `getFieldNames(): array` (for translating field names).

### 🔹 Phase 2: Core Methods

- [ ] Implement `getApiUrl(string $type): string` (returns URL based on `$is_test_mod` and operation type).
- [ ] Implement `parseResponse($response): array` (converts response to array).
- [ ] Implement `getAuthToken(bool $useCache = true): ?string` with Cache storage.
- [ ] Implement `requestNewToken(): array` (request token from OAuth2 or any authentication system).
- [ ] Implement `createPayment($token): array` (create transaction via API).
- [ ] Implement `confirmPayment($token, $transactionId, $otp = null): array` (confirm payment).
- [ ] Implement `checkTransactionStatus($refId): array` (query transaction status).
- [ ] Implement `reverse(string $refNo, string $reason = '', ?string $customerId = null): array` (refund – if the gateway supports it).
- [ ] Implement `getTransactionStatusForOrder(string $orderId): array` (helper to query using `order_id`).

### 🔹 Phase 3: Payment Logic

- [ ] **In `process()`**:
  - [ ] Validate data with `Validator`.
  - [ ] Check that `payment_state != PaidState`.
  - [ ] Look for an existing transaction (`payment_first_trans_id`) and call `checkTransactionStatus`.
  - [ ] Handle cases: `Complete` → `$result->success()`, `Pending` → return waiting message, `Failed` → continue creation.
  - [ ] Obtain token via `getAuthToken()`.
  - [ ] Create transaction via `createPayment()`.
  - [ ] Save `payment_first_trans_id`, `payment_trans_id` and additional data in `other_data`.
  - [ ] Store `callback_success_url` and `callback_error_url` in `other_data`.
  - [ ] Log `PaymentLog` with status `'initiated'` via `$result->logSuccessfulPayment()`.
  - [ ] Return result by type: `Redirect` → `$result->redirect()`, `Two‑Step` → `$result->successful = true` (without `success()`), `Direct` → `$result->success()`.
- [ ] **In `complete()`**:
  - [ ] Retrieve `transaction_id` from `payment_first_trans_id`.
  - [ ] Obtain `otp` from `$this->data` (if needed).
  - [ ] Call `confirmPayment()` or query status.
  - [ ] Handle special error codes (e.g., 6022).
  - [ ] On success: `return $result->success()`.
  - [ ] On failure: `return $result->fail()`.

### 🔹 Phase 4: Helper Methods

- [ ] Implement `getSupportedCurrencies(): array`, `getCurrencyId()`, `isCurrencySupported()`.
- [ ] Implement `generateIdempotencyKey(): string` (to create a unique identifier for the transaction).
- [ ] Implement `validateSettings(): void` (to check that essential settings exist).
- [ ] Implement `isAvailable(): bool` (to check credentials validity).
- [ ] Implement `static checkAndCompletePay(array $options): array` (unified confirmation logic for the `success` route).
- [ ] Implement `handleSpecificErrors(array $responseData, string $defaultMessage): array` (to translate error codes).

### 🔹 Phase 5: Create Partial Files

- [ ] Create `_info.htm` (informational and guidance) in `paymenttypes/xxxpay/`.
- [ ] Create `_test_info.htm` (quick testing tool) with unique IDs (`xxxpay-...`), buttons, and JavaScript.
- [ ] Ensure `_test_info.htm` calls the correct routes (`/api/v1/yepayment/xxxpay/test-*`).
- [ ] Create an integrated testing page `views/xxxpay-ui.htm` with tabs (Manual, Automated, Statistics, Logs).

### 🔹 Phase 6: Add Routes

- [ ] Add `GET /xxxpay/success` route calling `XxxPay::checkAndCompletePay` and using `RedirectHelper`.
- [ ] Add `GET /xxxpay/cancel` route using `RedirectHelper`.
- [ ] Add `POST /xxxpay/test-auth` (protected by `BackendAuth`).
- [ ] Add `POST /xxxpay/test-create-payment` (protected).
- [ ] Add `POST /xxxpay/test-confirm-payment` (protected).
- [ ] Add `GET /xxxpay/test-check-status` (protected).
- [ ] Add `POST /xxxpay/test-refund` (protected – optional).
- [ ] Add `GET /xxxpay/stats` (protected).
- [ ] Add `GET /xxxpay/logs` (protected).
- [ ] Add `GET /xxxpay/test-ui` (protected) to display the testing page.
- [ ] Use `try-catch` in all routes and return a unified JSON response with `success` and `message`.

### 🔹 Phase 7: Register the Gateway and Configuration

- [ ] Register the class in `Plugin.php` inside `registerPaymentProviders()` (or `registerYemenPaymentProviders` / `registerOmanPaymentProviders`).
- [ ] Add translation keys in `lang/ar/lang.php` and `lang/en/lang.php` for all texts used in settings and public messages.
- [ ] Gateway settings will be automatically stored in the database when saved from the control panel.

### 🔹 Phase 8: Testing and Validation

- [ ] Test authentication (`/test-auth`) and ensure a valid token is obtained.
- [ ] Test creating a transaction (`/test-create-payment`) for a new order.
- [ ] Test idempotency (resend the same request) and ensure no duplicate transaction is created.
- [ ] Test payment confirmation (`/test-confirm-payment`) with correct and incorrect OTP.
- [ ] Test the `success` and `cancel` routes using `callback_success_url`.
- [ ] Test refund (`/test-refund`) if the gateway supports it.
- [ ] Test token expiration (simulate `401` and clear Cache).
- [ ] Test the `test-ui` page and all tabs.
- [ ] Test compatibility with a real mobile app using a genuine Deep Link.
- [ ] Test security (attempt to access test routes without login).
- [ ] Test edge cases (order not found, unsupported currency, zero amount).

### 🔹 Phase 9: Final Documentation

- [ ] Write a `README.md` in the gateway folder containing everything a developer needs.
- [ ] Add PHPDoc comments for every function in the class.
- [ ] Update `_info.htm` by adding links to external gateway documentation.
- [ ] Ensure all error messages are translated and understandable.

### 🔹 Phase 10: Post‑Launch

- [ ] Monitor error logs (`trace_log`) during the first days.
- [ ] Set up alerts for critical error codes (e.g., 6022).
- [ ] Add webhook support if the gateway sends notifications.
- [ ] Keep a backup of gateway settings.

---

## 🏁 Final Summary

### What Makes This Guide Stand Out?

1. **Complete coverage**: Covers all aspects of creating a new payment gateway, from class structure to testing and documentation.
2. **Practical application**: Based on real examples from actual gateways (`YottaPay`, `BasPay`, `CashPay`, `KuraimiPay`, `YkbPay`, `ThawaniPay`).
3. **Handles real‑world scenarios**: Idempotency, logging incomplete attempts, special error handling, Deep Link support.
4. **Security and performance**: Data encryption, token caching, input validation.
5. **Production‑ready**: A comprehensive checklist ensures no step is missed.

### Core Principles to Remember When Creating Any New Payment Gateway

| Principle | Description |
|-----------|-------------|
| **Use `HttpHelper`** | For all API requests; never use raw `curl`. |
| **Use `RedirectHelper`** | For post‑payment redirection, especially to support mobile apps. |
| **Do not call `$result->success()` in `process()` for Two‑Step/Redirect** | This prevents marking the order as paid prematurely. |
| **Check for an existing transaction** | Using `payment_first_trans_id` and calling `checkTransactionStatus`. |
| **Log `PaymentLog` in `process()`** | Even for incomplete transactions, to track payment attempts. |
| **Store return URLs in `other_data`** | To be used with `RedirectHelper` when returning from the gateway. |
| **Add a static `checkAndCompletePay` function** | To unify confirmation logic in the `success` route. |
| **Use `try-catch` in all routes** | And return a unified JSON response. |
| **Document everything** | PHPDoc, `_info.htm`, `README.md`, and translated error messages. |
| **Test all scenarios** | Manually and automatically, simulating errors. |

### Final Short Summary:

🚀 **You now have a complete guide that any developer – even a beginner – can follow to create a new payment gateway in the `Nano.Yepayment` system with full professionalism and confidence.**

Always remember:
- Start by understanding the API documentation of the gateway you are integrating.
- Choose the appropriate flow type (Direct, Two‑Step, Redirect).
- Implement each part according to the guidelines above.
- Test thoroughly before launch.
- Document your work to help yourself and others in the future.

With this, we have completed all parts of the guide. You can now use this guide as a complete reference for developing any payment gateway, no matter how complex.

🎉 **Great work! Good luck with your upcoming projects.** 🎉