## 2026-05-20 – 2026-05-22

**CashPay Payment Gateway Update (Electronic Cash Payment – OTP) in the NanoSoft Payment System**


### Addition of CashPay Payment Gateway (Electronic Cash Payment – OTP)

Within the `Nano.Yepayment` software module

---

## 1. Introduction

A new payment method named **CashPay** has been developed and added to the unified payment system in NanoSoft within the `Nano.Yepayment` module. This addition responds to the need to support electronic payment gateways that rely on payment confirmation via OTP (One‑Time Password), as **CashPay** is a payment platform that enables accepting payments through a direct API with AES‑256‑CBC password encryption and a secure exchange mechanism.

CashPay operates according to the **Two‑Step Payment using OTP** pattern, which matches the platform's nature where:
1. **First step (`process`):** A transaction (InitPayment) is created and a `TransactionRef` is obtained, without deducting the amount yet. At this stage, the customer is asked to enter the OTP that will be sent later.
2. **Second step (`complete`):** After the customer enters the OTP, a payment confirmation request (ConfirmPayment) is sent to CashPay. If the code is correct, the amount is deducted and the order is updated to "paid".

This design makes the gateway fully compatible with `OrderManager`, `Checkout2`, and `PaymentResult`, and prevents the order from being marked as paid before the correct OTP is entered.

The gateway has been developed according to the highest security and quality standards, using the unified `HttpHelper` for all API requests, encrypting the password in the header using AES‑256‑CBC, and providing integrated testing interfaces and dedicated API endpoints for developers and administrators. Support for `RedirectHelper` has also been added to ensure compatibility with mobile applications that need return URLs after payment.

---

## 2. Developed Components

### 2.1. `CashPay` – Main Payment Gateway Class

- **Path:** `Nano\Yepayment\PaymentTypes\CashPay`
- **Inheritance:** Extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function:** Responsible for authentication via encrypted header (`encPassword`), executing the first step (InitPayment), the second step (ConfirmPayment), querying transaction status (OperationStatus), and changing the password (ChangePass).

**Main Methods:**

| Method | Description |
|--------|-------|
| `process(PaymentResult $result)` | **First step:** Checks for an existing transaction, creates a new transaction via `InitPayment`, receives `TransactionRef`, saves it in the order without changing payment status. Returns success with a message "Enter OTP". |
| `complete(PaymentResult $result)` | **Second step:** Verifies `TransactionRef` and `OTP`, sends a `ConfirmPayment` request, and on success calls `$result->success()` to update the order to `PaidState`. |
| `operationStatus(string $requestId, string $type): array` | Queries transaction status using `RequestID` (type `InitOP` for initial transaction, `PayOP` for confirmation). |
| `changePassword(string $newPassword): array` | Changes the merchant's password with CashPay (when expired or needs updating). |
| `getAuthToken(): ?string` | (Not used in CashPay because authentication relies on `encPassword` in the header. Present for compatibility.) |
| `initPayment(): array` | Builds the `InitPayment` request and sends it to `/api/CashPay/InitPayment`, returning `TransactionRef`. |
| `confirmPayment(string $transactionRef, string $otp): array` | Builds the `ConfirmPayment` request with `TRCode = md5(TransactionRef + OTP)` and sends it to `/api/CashPay/ConfirmPayment`. |
| `encryptAes256Cbc(string $data, string $key): string` | Encrypts data (password) using AES‑256‑CBC to obtain `encPassword`. |
| `getEncryptedPassword(): string` | Encrypts the stored password using the encryption key from settings. |
| `buildHeaders(): array` | Builds request headers (`encPassword`, `unixtimestamp`, `Content-Type`). |
| `settings(): array` | Defines the settings fields in the control panel (URL, Username, Password, Encryption Key, SpId, default currency). |
| `encryptedSettings(): array` | Specifies fields that are stored encrypted (`cashpay_password`, `cashpay_encryption_key`). |
| `getSupportedCurrencies(): array` | List of supported currencies with their numeric IDs (YER=2, USD=1, SAR=3). |
| `getCurrencyId($currency): ?int` | Converts a currency code or number to the correct supported numeric ID. |
| `isCurrencySupported($currency): bool` | Checks whether a given currency is supported. |

### 2.2. Partials and Settings Files

| File | Path | Description |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/cashpay/_info.htm` | Displays setup instructions and gateway information in the control panel. |
| `_test_info.htm` | `paymenttypes/cashpay/_test_info.htm` | Quick testing tools within the settings page (buttons for authentication, create transaction, confirm, query, change password, full flow). |
| `cashpay-ui.htm` | `views/cashpay-ui.htm` | Complete interactive web interface for testing all CashPay functions (manual, automated, statistics, logs). |

### 2.3. API Endpoints (`routes.php`)

A complete set of routes has been added within the `yepayment` group to support testing and monitoring:

| Path | Method | Description |
|--------|---------|-------|
| `/cashpay/test-auth` | POST | Test settings validity (Username, Password, Encryption Key, SpId). |
| `/cashpay/test-create-payment` | POST | Create a new transaction (InitPayment). |
| `/cashpay/test-confirm-payment` | POST | Confirm payment (ConfirmPayment) using OTP. |
| `/cashpay/test-check-status` | POST | Query transaction status (OperationStatus). |
| `/cashpay/test-change-password` | POST | Change password (ChangePass). |
| `/cashpay/test-full-payment` | POST | Full flow test (InitPayment + ConfirmPayment). |
| `/cashpay/stats` | GET | Gateway usage statistics (number of orders, success rate). |
| `/cashpay/test-ui` | GET | Integrated testing interface (HTML). |
| `/cashpay/success` | GET | Endpoint for redirection on payment success (uses `RedirectHelper`). |
| `/cashpay/cancel` | GET | Endpoint for redirection on cancellation. |

All routes (except success/cancel) are protected by `BackendAuth::getUser()` to ensure only administrators can access them.

### 2.4. `Plugin.php` – Payment Provider Registration

The following line was added to the `registerPaymentProviders()` function within the `allow_yemen_payment` section:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\CashPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\CashPay();
}
```

### 2.5. Translation Files

Translation keys were added in `lang/ar/lang.php` and `lang/en/lang.php` for settings, general messages, and errors (e.g., `payment_success`, `auth_success`, `confirmation_required`, `order_already_paid`).

---

## 3. Payment Workflow – Two‑Step (OTP) Pattern

CashPay operates according to a **Two‑Step (OTP)** flow:

1. **Authentication**  
   There is no separate token session. Every request is sent with an `encPassword` header, which is the merchant's password encrypted with AES‑256‑CBC using the provided encryption key.  
   A `unixtimestamp` (in milliseconds) is also used to prevent replay attacks.

2. **First step – Create Transaction (`process`)**  
   - Ensure the order is not already paid.
   - Check for an existing transaction associated with the order via `payment_first_trans_id` or `other_data['cashpay']['transaction_ref']`:
     - If found, call `operationStatus()` to query its status.
     - If status is `success` → calls `$result->success()` (order already paid).
     - If status is `pending` → returns a message "Enter OTP" without creating a new transaction.
   - **No existing transaction or invalid status:**  
     - Build the request body (`RequestID`, `UserName`, `SpId`, `MDToken`, `TargetMSISDN`, `CustomerCashPayCode`, `Amount`, `CurrencyId`, `Desc`).
     - Send a `POST /api/CashPay/InitPayment` request.
     - Receive `TransactionRef` and store it in `order.payment_first_trans_id` and `order.payment_trans_id`.
     - Store additional data in `order.other_data['cashpay']` (amount, currency, mobile number, payment code, creation date, callback URLs).
     - **Do not call `$result->success()`**, and do not change the order to `PaidState`. Instead, return `successful = true` with a `confirmation_required` message.
     - Log the payment attempt via `$result->logSuccessfulPayment()`.

3. **Second step – Confirm Payment (`complete`)**  
   - Called after the customer enters the OTP (passed in `$this->data['otp']`).
   - Extract `TransactionRef` from the order.
   - Compute `TRCode = md5(TransactionRef . OTP)`.
   - Send a `POST /api/CashPay/ConfirmPayment` request with `TransactionRef` and `TRCode`.
   - If `ResultCode == 1`:
     - Call `$result->success()`, which **changes the order status to `PaidState`**, triggers events (e.g., `nano.orders.paymentProcessed`), and clears the cart.
   - If `ResultCode == 6022`: return an error "Password must be changed".
   - Any other code is considered a failure.

4. **Optional Callback URLs**  
   When `callback_success_url` or `callback_error_url` are passed in the payment data, they are stored in `other_data`. After confirmation, the developer can use `RedirectHelper` to intelligently redirect (deep link or web).

---

## 4. Configuration

To activate CashPay, the following settings must be entered in the payment gateway settings interface in the NanoSoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| Setting | Key | Description | Default Value |
|---------|---------|-------|-------------------|
| Base API URL | `cashpay_url` | Production API endpoint | `https://api.cash-pay.com` |
| Test API URL | `cashpay_test_url` | UAT API endpoint (optional) | `https://test-api.cash-pay.com` |
| Username | `cashpay_username` | Merchant username | - |
| Password | `cashpay_password` | Merchant password (stored encrypted) | - |
| Encryption Key (AES‑256‑CBC) | `cashpay_encryption_key` | Key used to encrypt the password in the header | - |
| Client Code (SpId) | `cashpay_sp_id` | Service Provider ID provided by CashPay | - |
| Default Currency | `cashpay_default_currency_id` | Default currency (shown as dropdown) | `YER` |

**Security Notes:**
- `cashpay_password` and `cashpay_encryption_key` are stored encrypted via `encryptedSettings()`.
- These sensitive values are not logged.

---

## 5. Usage Examples

### 5.1. Starting Payment – First Step (InitPayment)

```php
use Nano\Yepayment\PaymentTypes\CashPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$cash = new CashPay($order, [
    'target_msisdn' => '771234567',
    'customer_cash_pay_code' => 555,
    'amount' => 100,
    'currency' => 'YER',
    'desc' => 'Payment for order #200',
    'callback_success_url' => 'myapp://pay/success',
]);
$result = new PaymentResult($cash, $order);
$processResult = $cash->process($result);

if ($processResult->successful) {
    // Transaction created and TransactionRef saved, system awaits OTP input
    $transactionRef = $order->payment_first_trans_id;
    // Show OTP input field to the user
}
```

### 5.2. Confirming Payment – Second Step (ConfirmPayment)

After the user enters the OTP, you can call `complete()` directly:

```php
$cash = new CashPay($order, ['otp' => '1234']);
$result = new PaymentResult($cash, $order);
$completeResult = $cash->complete($result);

if ($completeResult->successful) {
    // Payment confirmed and order updated to PaidState
}
```

Or using a static helper function (via the `success` route):

```php
$result = \Nano\Yepayment\PaymentTypes\CashPay::checkAndCompletePay(['order_id' => 200, 'otp' => '1234']);
if ($result['success']) {
    // Payment successful
}
```

### 5.3. Direct Transaction Status Query

```php
$cash = new CashPay();
$status = $cash->operationStatus('1234567890', 'InitOP');
if ($status['success']) {
    echo "Status: " . $status['status']; // success, pending, failed
}
```

### 5.4. Changing Password (ChangePass)

When receiving error code 6022, you must change the password:

```php
$cash = new CashPay();
$result = $cash->changePassword('NewStrongPass123');
if ($result['success']) {
    // Password changed successfully, also update the settings in the control panel
}
```

### 5.5. Using the Integrated Testing Interface

After logging into the control panel as an administrator, open the URL:
```
https://yourdomain.com/api/v1/yepayment/cashpay/test-ui
```

The interface allows:
- Test authentication (verify settings validity).
- Create transaction (InitPayment) and display `TransactionRef`.
- Confirm payment (ConfirmPayment) using OTP.
- Query status (OperationStatus) using `RequestID`.
- Change password (ChangePass).
- Full flow (create + confirm) in one step.
- Automated testing with a configurable number of attempts (up to 5 times).
- Usage statistics (number of orders, success rate).
- Local test logs.

---

## 6. Handling Error Codes and Response Codes

| Code | Meaning | Appropriate Action |
|------|--------|------------------|
| 1 | Success | Continue normal flow. |
| 35 | Invalid input data | Check the fields sent. |
| 6022 | Password expired or invalid | Call `changePassword()` to change the password. |
| 6025 | Customer balance insufficient | Inform customer to top up balance. |
| 9999 | General internal error | Retry or contact support. |

> **Note:** Any other error code is displayed as received from CashPay via `ResultMessage`.

---

## 7. Integration Testing

Several testing layers are provided:

- **Test environment:** Use the UAT URL provided by CashPay (set in the `cashpay_test_url` field).
- **Quick test interface:** From the gateway settings page (partial file `_test_info.htm`), you can test authentication, create transaction, confirm, query, and change password.
- **Integrated testing interface:** `/api/v1/yepayment/cashpay/test-ui` provides all necessary tools to fully test the gateway (manual, automated, statistics).
- **Independent API endpoints:** Developers can use tools like Postman or cURL to directly connect to endpoints such as `/cashpay/test-create-payment`.

**Quick test steps:**
1. Ensure Username, Password, Encryption Key, and SpId are set in the gateway settings page.
2. Open `/api/v1/yepayment/cashpay/test-ui`.
3. Click "Test authentication" to verify credentials.
4. Enter an order number, mobile number, payment code, amount, then click "Create transaction".
5. `TransactionRef` will appear – enter the correct OTP (sent to the mobile) in the OTP field and click "Confirm payment".
6. The order status will change to paid.

---

## 8. Developer Notes

- **Two‑Step pattern:** `process()` **does not** complete the payment; it only creates the transaction. You must call `complete()` with the correct OTP to confirm the payment and update the order to `PaidState`.
- **Password encryption:** The password in the request header is encrypted using `encryptAes256Cbc()` with the dedicated encryption key. Make sure the key matches the one provided by CashPay.
- **Password change:** When error code `6022` appears, you should call `changePassword()` immediately (you can show a message to the administrator or direct them to change the password in the control panel).
- **Existing transaction check:** In `process()`, the system first checks for an existing transaction associated with the order. If its status is `success`, the order is completed immediately, preventing duplicate transactions.
- **Using `HttpHelper`:** All API requests use the unified `HttpHelper`, making it easier to track errors and extend the gateway.
- **Currency support:** Functions `getSupportedCurrencies()`, `getCurrencyId()`, and `isCurrencySupported()` are included to check supported currencies and convert codes to the correct numeric IDs.
- **Support for `RedirectHelper`:** Full redirection capability has been included, making the gateway suitable for mobile applications that use Deep Links.
- **Extensibility:** Support for webhooks or refunds can be added in the future via additional methods using different CashPay endpoints.

---

## 9. Bug Fixes

None – this release is dedicated to adding a new feature (CashPay).

---

## 10. Development and Testing Period

The CashPay gateway was developed during the period from **May 15, 2026** to **May 20, 2026**. This period included:
- Writing the core code for the `CashPay` class (extending `PaymentProvider`).
- Implementing `process()`, `complete()`, `initPayment()`, `confirmPayment()`, `operationStatus()`, and `changePassword()` methods.
- Adding the existing transaction check in `process()` to prevent duplication.
- Implementing the password encryption mechanism using AES‑256‑CBC.
- Creating partial view files (`_info.htm`, `_test_info.htm`) and the integrated testing interface (`cashpay-ui.htm`).
- Writing the necessary API endpoints for testing and monitoring in `routes.php` (10 endpoints).
- Adding translation keys in language files.
- Registering the gateway in `Plugin.php`.
- Conducting comprehensive integration tests to verify functionality with the CashPay environment (simulated using Postman and UI).

---

## 11. Relevant Links

- [CashPay Documentation (Developer Guide)](./docs/CashPay/Docs-CashPay-en.md)
- [SKILL.md for Creating Payment Gateways](./docs/CashPay/SKILL-en.md)
- [Cash-Pay API Documentation](./docs/CashPay/external/Cash-Pay%20API%20Doc2.pdf)
- [CashPay.php Class](./Nano/Yepayment/PaymentTypes/CashPay.php)
- [routes.php file of Nano.Yepayment](./routes.php)
- [Technical Support Channel](https://nano2soft.com)

---

**This update prepared by:**  
NanoSoft Development Team – Electronic Payments Department  
**References:** Dheia Ali, Nano2Soft
