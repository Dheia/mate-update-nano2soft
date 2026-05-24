## 2026-05-18 – 2026-05-24

### Addition and Update of FloosakPay Payment Gateway (Floosak Wallet)

---

## 1. Introduction

A new payment method named **FloosakPay** has been developed and added to the unified payment system in NanoSoft within the `Nano.Yepayment` module. This addition responds to the need to support digital wallet payment gateways in Yemen, where **Floosak Wallet** is one of the leading digital financial solutions that allows users to make electronic payments easily and securely.

FloosakPay relies on the **Floosak Wallet API** (v1), which provides:
- Merchant authentication via `/api/v1/auth/login` to obtain a `Bearer token`.
- Create a pending purchase via `/api/v1/merchant/p2mcl`.
- Confirm payment using an OTP via `/api/v1/merchant/p2mcl/confirm`.
- Query transaction status via `/api/v1/merchant/check-status`.
- Refund an amount via `/api/v1/merchant/p2mcl/refund`.

The gateway is designed according to the **Two‑Step payment pattern with OTP confirmation**, where:
1. **First step (`process`):** A pending transaction (Send) is created, an OTP is sent to the customer’s mobile, and transaction data (`purchase_id`) is returned. **The order is not changed to paid.**
2. **Second step (`complete`):** After the customer enters the code, the payment is confirmed using the OTP. If the code is correct, the order is updated to `PaidState` and the cart is cleared.

This design ensures full compatibility with `OrderManager`, `Checkout2`, and `PaymentResult`, and prevents the order from being marked as paid before the correct OTP is confirmed.

The gateway has been developed according to the highest security and quality standards, with tokens stored in Cache, using the unified `HttpHelper` for all API requests, and providing integrated testing interfaces and dedicated API endpoints for developers and administrators. Support for `RedirectHelper` has also been added to ensure compatibility with mobile applications that need return URLs after payment, along with an **idempotency mechanism** via `request_id` to prevent duplicate transactions, and logging of `PaymentLog` in the first stage to track incomplete attempts.

---

## 2. Developed Components

### 2.1. `FloosakPay` – Main Payment Gateway Class

- **Path:** `Nano\Yepayment\PaymentTypes\FloosakPay`
- **Inheritance:** Extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function:** Responsible for authentication via `/api/v1/auth/login`, creating a pending transaction (Send), confirming payment (Confirm), querying status, and refunding.

**Main Methods:**

| Method | Description |
|--------|-------|
| `process(PaymentResult $result)` | **First step:** Validates input (`target_phone`, `amount`, `purpose`), checks for an existing transaction (idempotency), obtains a token, sends Send request, saves `request_id` and `purchase_id` in the order, logs `PaymentLog` as `initiated`, then returns `successful = true` with a message asking for OTP (**does not call `$result->success()`**). |
| `complete(PaymentResult $result)` | **Second step:** Extracts `purchase_id`, obtains a new token, sends Confirm request with OTP. If successful and status is `Completed`, calls `$result->success()` to update the order to `PaidState`. |
| `reverse(string $refNo, string $reason = ''): array` | Refunds a completed transaction via `/api/v1/merchant/p2mcl/refund`. On success, changes order status to `RefundedState` and stores refund data in `other_data`. |
| `checkTransactionStatus(string $requestId): array` | Queries transaction status using `request_id` via `/api/v1/merchant/check-status`. Used in idempotency and testing tools. |
| `getTransactionStatusForOrder(string $orderId): array` | Helper function to query status using `order_id` directly. |
| `getAuthToken(bool $useCache = true): ?string` | Obtains a `Bearer token` via `POST /api/v1/auth/login` (with `x-channel: merchant`). Token is cached for 3500 seconds. |
| `sendPayment(string $token): array` | Sends `POST /api/v1/merchant/p2mcl` to create a pending transaction. Generates `request_id` as UUID. Returns `purchase_id`, `reference_id`, `fee`, `gross`. |
| `confirmPayment(string $token, int $purchaseId, string $otp): array` | Sends `POST /api/v1/merchant/p2mcl/confirm` to confirm payment. Checks that `status.en == "Completed"`. |
| `refundPayment(string $token, int $transactionId, string $requestId, float $amount, string $reason): array` | Sends `POST /api/v1/merchant/p2mcl/refund`. |
| `settings(): array` | Defines settings fields in the control panel (URL, phone, password, source_wallet_id, default_purpose). |
| `encryptedSettings(): array` | Specifies fields stored encrypted (`floosakpay_password`). |
| `defineValidationRules(): array` | Validation rules for `target_phone`, `amount`, `purpose`. |
| `handleSpecificErrors(array $responseData, string $defaultMessage): array` | Extracts error messages from Floosak response (supports `errors` and `message`). |

### 2.2. Idempotency Mechanism (Preventing Duplicate Transactions)

- In `process()`, a search is performed for an existing transaction via `payment_first_trans_id` or `other_data['floosakpay']['request_id']`.
- If found, `checkTransactionStatus()` is called:
  - If status is `Completed` → calls `$result->success()` directly (order already paid).
  - If status is `Pending` → informs the user to wait for OTP without creating a new transaction.
  - If status is unknown or failed → creates a new transaction.
- This ensures no duplicate transactions are created when the user retries the payment process.

### 2.3. Logging `PaymentLog` in the First Stage

- After successfully creating the transaction, a `PaymentLog` is manually logged:
  ```php
  $paymentLog = $result->logSuccessfulPayment($sendResult, null, 'initiated');
  $this->order->payment_id = $paymentLog->id;
  $this->order->save();
  ```
- This allows tracking of payment attempts even if the customer never completes the OTP, helping to analyse completion rates.

### 2.4. Partials and Settings Files

| File | Path | Description |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/floosakpay/_info.htm` | Displays setup instructions and gateway information in the control panel. |
| `_test_info.htm` | `paymenttypes/floosakpay/_test_info.htm` | Quick testing tools within the settings page (authentication, create, confirm, query, refund buttons). |
| `floosakpay-ui.htm` | `views/floosakpay-ui.htm` | Complete interactive web interface for testing all FloosakPay functions (manual, automated, statistics, logs). |

### 2.5. API Endpoints (`routes.php`)

A complete set of routes has been added within the `yepayment` group to support testing and monitoring:

| Path | Method | Description |
|--------|---------|-------|
| `/floosakpay/test-auth` | POST | Test authentication and obtain Bearer Token. |
| `/floosakpay/test-create-payment` | POST | Create a pending transaction (simulates `process`). |
| `/floosakpay/test-confirm-payment` | POST | Confirm payment using OTP (simulates `complete`). |
| `/floosakpay/test-check-status` | GET | Query transaction status using `request_id` or `order_id`. |
| `/floosakpay/test-refund` | POST | Refund a completed transaction. |
| `/floosakpay/test-full-payment` | POST | Full test (create + confirm + query) in one step. |
| `/floosakpay/stats` | GET | Gateway usage statistics (number of orders, success rate, recent logs). |
| `/floosakpay/logs` | GET | Transaction logs from the database (for admin). |
| `/floosakpay/test-ui` | GET | Integrated testing interface (HTML). |
| `/floosakpay/success` | GET | Endpoint to confirm payment after OTP entry (uses `checkAndCompletePay`). |
| `/floosakpay/cancel` | GET | Optional endpoint for cancellation redirection. |

All routes (except success/cancel) are protected by `BackendAuth::getUser()` to ensure only administrators can access them.

### 2.6. `Plugin.php` – Payment Provider Registration

The following line was added to the `registerPaymentProviders()` function within the `allow_yemen_payment` section:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\FloosakPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\FloosakPay();
}
```

### 2.7. Translation Files

Translation keys were added in `lang/ar/lang.php` and `lang/en/lang.php` for settings, general messages, and errors (e.g., `auth_success`, `confirmation_required`, `payment_success`, `order_already_paid`, `auth_failed`, `missing_credentials`, `missing_wallet_id`).

---

## 3. Payment Workflow – Two‑Step (OTP) Pattern

FloosakPay operates according to a **Two‑Step** flow with OTP confirmation:

### 3.1. Authentication

`getAuthToken()` is called:
- Checks for an `access_token` in Cache (validity 3500 seconds).
- If not present, sends a `POST /api/v1/auth/login` request with `phone`, `password`, and `x-channel: merchant`.
- The token is stored in Cache for slightly less than its actual lifetime.

### 3.2. First Step – Create Pending Transaction (`process`)

- Validate data (`target_phone`, `amount`, `purpose`) using `Validator`.
- Ensure the order is not already paid (`payment_state != PaidState`).
- **Check for an existing transaction (Idempotency):**
  - Search for `request_id` in `payment_first_trans_id` or `other_data['floosakpay']['request_id']`.
  - If found, call `checkTransactionStatus()`:
    - If `Completed` → call `$result->success()` (order already paid).
    - If `Pending` → return a message asking for OTP without creating a new transaction.
    - Otherwise → continue with creation.
- **No existing transaction or invalid status:**
  - Obtain `access_token`.
  - Send `POST /api/v1/merchant/p2mcl` with:
    - `source_wallet_id` (from settings)
    - `request_id` (unique UUID)
    - `target_phone`, `amount`, `purpose`
  - Receive `purchase_id`, `reference_id`, `fee`, `gross`.
  - Save `request_id` in `order.payment_first_trans_id`.
  - Save `purchase_id` in `order.payment_trans_id`.
  - Store additional data in `order.other_data['floosakpay']` (phone, amount, purpose, return URLs, etc.).
  - **Do not call `$result->success()`**, and do not change order to `PaidState`.
  - Log a `PaymentLog` with status `'initiated'`.
  - Return `$result->successful = true` with a message "Please enter the verification code".

### 3.3. Second Step – Confirm Payment (`complete`)

- Called from the `floosakpay/success` route after the customer enters the OTP.
- Extract `purchase_id` from `order.payment_trans_id`.
- Obtain a fresh `access_token` (may have expired).
- Send `POST /api/v1/merchant/p2mcl/confirm` with `otp` and `purchase_id`.
- Parse the response:
  - If `is_success == true` and `status.en == "Completed"`:
    - Call `$result->success()` which:
      - Changes `payment_state` to `PaidState`.
      - Sets `processed = true`.
      - Notifies the system of associated events (sends email, updates inventory, clears cart).
  - If `status.en == "Pending"` → return a message that the code is incorrect or the transaction is still pending.
- Any other status is considered a failure.

### 3.4. Optional Callback URLs

When `callback_success_url` or `callback_error_url` are passed in the payment data, they are stored in `other_data`. After confirmation, the developer can use `RedirectHelper` in the `success` route to intelligently redirect the user (deep link or web) while passing the payment result.

---

## 4. Configuration

To activate FloosakPay, the following settings must be entered in the payment gateway settings interface in the NanoSoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| Setting | Key | Description | Default Value |
|---------|---------|-------|-------------------|
| Base API URL | `floosakpay_url` | Production API endpoint | `https://staging.fintech-expert.net` |
| Test API URL | `floosakpay_test_url` | Test environment API endpoint (optional) | `https://staging.fintech-expert.net` |
| Merchant Phone Number | `floosakpay_phone` | Mobile number used for authentication (with country code) | - |
| Password | `floosakpay_password` | Merchant account password (stored encrypted) | - |
| Source Wallet ID | `floosakpay_source_wallet_id` | ID of the wallet from which funds will be deducted | - |
| Default Purpose | `floosakpay_default_purpose` | Text sent as transaction description | `دفع الطلب` (Payment for order) |

**Security Notes:**
- `floosakpay_password` is stored encrypted via `encryptedSettings()`.
- These sensitive values are not logged.

---

## 5. Usage Examples

### 5.1. Starting Payment – First Step (Create Pending Transaction)

```php
use Nano\Yepayment\PaymentTypes\FloosakPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$floosak = new FloosakPay($order, [
    'target_phone' => '967771234567',
    'amount'       => 100,
    'purpose'      => 'دفع الطلب #200',
    'callback_success_url' => 'myapp://pay/success',
]);
$result = new PaymentResult($floosak, $order);
$processResult = $floosak->process($result);

if ($processResult->successful && !$processResult->redirect) {
    // Show OTP input interface to the user
    $purchaseId = $order->payment_trans_id;
    $requestId  = $order->payment_first_trans_id;
}
```

### 5.2. Confirming Payment – Second Step (Enter OTP)

After the customer enters the code, the process can be confirmed as follows:

```php
$floosak = new FloosakPay($order, ['otp' => '123456']);
$result = new PaymentResult($floosak, $order);
$completeResult = $floosak->complete($result);

if ($completeResult->successful) {
    // Payment confirmed and order updated to PaidState
}
```

Or using the static function `checkAndCompletePay`:

```php
$result = \Nano\Yepayment\PaymentTypes\FloosakPay::checkAndCompletePay(['order_id' => 200, 'otp' => '123456']);
if ($result['success']) {
    // Success
}
```

### 5.3. Direct Transaction Status Query

```php
$floosak = new FloosakPay();
$status = $floosak->checkTransactionStatus('550e8400-...');
if ($status['success'] && $status['completed']) {
    echo "Transaction completed";
}
```

### 5.4. Refunding a Transaction

```php
$order = Order::find(200);
$floosak = new FloosakPay($order);
$refundResult = $floosak->reverse($order->payment_first_trans_id, 'Refund requested by customer');
if ($refundResult['success']) {
    // Refund successful, order status becomes RefundedState
}
```

### 5.5. Using the Integrated Testing Interface

After logging into the control panel as an administrator, open the URL:
```
https://yourdomain.com/api/v1/yepayment/floosakpay/test-ui
```

The interface allows:
- Test authentication (verify phone, password, source_wallet_id).
- Create a pending transaction (simulate `process`) and display `purchase_id` and `request_id`.
- Confirm payment using OTP (manually or default `123456` in the test environment).
- Query transaction status.
- Refund an amount.
- Full flow (create + confirm + query) in one step.
- Automated testing with a specified number of attempts (up to 5 times).
- Usage statistics (number of orders, success rate, recent logs).
- Local test logs (stored in `localStorage`).

---

## 6. Handling the Test Environment (Sandbox)

- In the test environment, `x-channel: merchant` is set as in production.
- The OTP value is fixed: **`123456`** for testing purposes (no real SMS is sent).
- You can use the test URL `https://staging.fintech-expert.net` (set in the `floosakpay_test_url` field).
- Ensure that the `source_wallet_id` is valid in the test environment.

---

## 7. Added Value

- **For developers:** A complete example of a **Two‑Step** payment gateway using OTP, with idempotency, token caching, and PaymentLog logging in the first stage. Demonstrates how to handle APIs that require `x-channel` and standard JSON body.
- **For merchants:** Support for Floosak Wallet as a digital payment method, enabling customers to pay directly from their digital wallets without needing credit cards.
- **For end users:** A seamless payment experience via OTP, where the customer receives a code on their phone and confirms it inside the app, enhancing security.

---

## 8. Integration Testing

Several testing layers are provided:

- **Test environment:** Use the URL `staging.fintech-expert.net` (set in `floosakpay_test_url`). OTP is fixed `123456`.
- **Quick test interface:** From the gateway settings page (partial file `_test_info.htm`), you can test authentication, create a transaction, confirm, and query directly.
- **Integrated testing interface:** `/api/v1/yepayment/floosakpay/test-ui` provides all necessary tools to fully test the gateway (manual, automated, statistics, logs).
- **Independent API endpoints:** Developers can use tools like Postman or cURL to directly connect to endpoints such as `/floosakpay/test-create-payment` and `/floosakpay/test-confirm-payment`.

**Quick test steps:**
1. Ensure `phone`, `password`, and `source_wallet_id` are set in the gateway settings page.
2. Open `/api/v1/yepayment/floosakpay/test-ui`.
3. Click "Test authentication" to verify credentials.
4. Enter an order number, customer phone number, and amount, then click "Create transaction".
5. `purchase_id` will appear with a message "Please enter OTP". Use OTP `123456`.
6. Click "Confirm payment" – it should succeed and the order status will change to paid.
7. Try "Query status" and "Refund".

---

## 9. Developer Notes

- **Two‑Step pattern:** `process()` **does not** complete the payment; it only creates a pending transaction and returns a message asking for OTP. You must call `complete()` (or `checkAndCompletePay`) after the customer enters the code to confirm the payment and update the order to `PaidState`.
- **Idempotency:** An existing transaction is checked at the beginning of `process()` to prevent duplicate creation. This is very important when the payment page is reloaded or the "Pay" button is clicked multiple times.
- **Token caching:** The Bearer Token is stored in Cache for 3500 seconds (slightly less than the actual 3600‑second expiry) to avoid using an expired token.
- **Using `HttpHelper`:** All API requests use `HttpHelper::sendJson()` with appropriate headers (`x-channel: merchant`).
- **PaymentLog logging:** The payment attempt is logged in the first stage (status `initiated`) even if the customer never completes the OTP, helping to track and analyse completion rates.
- **Support for `RedirectHelper`:** Full redirection capability has been included, making the gateway suitable for mobile applications that use Deep Links.
- **Testing OTP:** In the test environment, the OTP is fixed as `123456`. No real SMS is sent.
- **Extensibility:** Support for webhooks or additional interfaces can be added in the future using the available endpoints.

---

## 10. Bug Fixes

None – this release is dedicated to adding a new feature (FloosakPay).

---

## 11. Development and Testing Period

The FloosakPay gateway was developed during the period from **May 18, 2026** to **May 24, 2026**. This period included:
- Writing the core code for the `FloosakPay` class (extending `PaymentProvider`).
- Implementing `process()`, `complete()`, `reverse()`, `getAuthToken()`, `sendPayment()`, `confirmPayment()`, and `checkTransactionStatus()` methods.
- Adding the idempotency mechanism (check for existing transaction).
- Adding `PaymentLog` logging in `process()` with status `initiated`.
- Creating partial view files (`_info.htm`, `_test_info.htm`) and the integrated testing interface (`floosakpay-ui.htm`).
- Writing the necessary API endpoints for testing and monitoring in `routes.php` (11 endpoints).
- Adding translation keys in language files.
- Registering the gateway in `Plugin.php`.
- Conducting comprehensive integration tests to verify functionality with the Floosak environment (simulated using Postman and UI).

---

## 12. Relevant Links

- [FloosakPay Documentation (Developer Guide)](./docs/FloosakPay/Docs-FloosakPay-en.md)
- [SKILL-v2.5 for Creating Payment Gateways](./docs/FloosakPay/SKILL-v2.5.md)
- [Official Floosak API Documentation](./docs/FloosakPay/external/Merchant_Wallet_Integration_API_Documentation_RefStyle.pdf)
- [Postman Collection for FloosakPay Testing](./docs/FloosakPay/external/Marchant%20Last%20Version.json)
- [FloosakPay.php Class](./Nano/Yepayment/PaymentTypes/FloosakPay.php)
- [routes.php file of Nano.Yepayment (FloosakPay section)](./routes.php)
- [Technical Support Channel](https://nano2soft.com)

---

**This update prepared by:**  
NanoSoft Development Team – Electronic Payments Department  
**References:** Dheia Ali, Nano2Soft