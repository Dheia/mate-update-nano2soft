# CashPay Payment Gateway Documentation – Electronic Cash Payment (OTP)

## 1. Overview

**CashPay** is an electronic payment gateway integrated within the `Nano.Yepayment` package in NanoSoft applications, based on the **Cash-Pay API**. The gateway operates according to the **Two‑Step Payment** pattern using a One‑Time Password (OTP) sent to the customer's mobile phone:

1. **`process()` – InitPayment:** The merchant sends a transaction creation request to CashPay and receives a unique identifier (`TransactionRef`). At this stage **no amount is deducted**, and the reference is saved in the order without changing the payment status.
2. **`complete()` – ConfirmPayment:** After the customer receives the OTP (via SMS or banking app), the merchant sends the code together with the `TransactionRef` to CashPay to confirm the payment. If the data is correct, the amount is deducted and the order is updated to `PaidState`.

The gateway supports the following currencies: Yemeni Rial (`YER`), US Dollar (`USD`), and Saudi Riyal (`SAR`). It also provides additional functions to query transaction status (`OperationStatus`) and change the password (`ChangePass`) when it expires or needs to be updated.

This gateway is built according to `Nano.Yepayment` standards and follows the `PaymentProvider` pattern similar to `YottaPay` and `BasPay`.

---

## 2. Requirements and Settings

### 2.1. Basic Requirements

- NanoSoft system (Nano2Soft) version 2.0+
- Required plugins:
  - `Nano.MicroCart` (>=2.0)
  - `Nano.Yepayment` (>=1.2)
  - `Nano.Helpers`
- CashPay API credentials (provided by the company):
  - `Username`
  - `Password`
  - `Encryption Key` – used to encrypt the password before sending it in the header
  - `SpId` (Service Provider ID)
  - Base API URL (production and test)

### 2.2. Gateway Settings in Control Panel

When activating the **"CashPay (Electronic Cash Payment)"** payment method, the following fields appear on the payment settings page:

| Field | Key | Description | Default Value |
|-------|---------|-------|-------------------|
| Base API URL | `cashpay_url` | CashPay API endpoint | `https://api.cash-pay.com` |
| Test API URL | `cashpay_test_url` | UAT API endpoint | `https://test-api.cash-pay.com` |
| Username | `cashpay_username` | Merchant username | - |
| Password | `cashpay_password` | Merchant password (stored encrypted) | - |
| Encryption Key (AES‑256‑CBC) | `cashpay_encryption_key` | Key used to encrypt the password in the header | - |
| Client Code (SpId) | `cashpay_sp_id` | Service Provider ID provided by CashPay | - |
| Default Currency | `cashpay_default_currency_id` | Default currency (shown as dropdown) | `YER` |

> **Security note:** Sensitive fields (`cashpay_password`, `cashpay_encryption_key`) are stored encrypted in the database via `encryptedSettings()`.

---

## 3. Payment Process Lifecycle

CashPay operates according to the **Two‑Step (OTP)** payment pattern.

### 3.1. First Stage: Creating the Transaction (`process`)

When payment is initiated for a given order, `process()` is called and performs the following steps:

1. **Data validation** – Ensures that the mobile number (`target_msisdn`), cash payment code (`customer_cash_pay_code`), amount, and currency are present.
2. **Ensure the order is not already paid** – If `payment_state` equals `PaidState`, the process is stopped.
3. **Check for an existing transaction** – Searches for a `TransactionRef` stored in `payment_first_trans_id` or `other_data['cashpay']['transaction_ref']`. If found:
   - Calls `operationStatus()` to query the status.
   - If status is `success` → calls `$result->success()` directly (order already paid).
   - If status is `pending` → returns a message "Enter OTP" without creating a new transaction.
4. **Create a new transaction via `InitPayment`** – Sends a `POST /api/CashPay/InitPayment` request with the required data.
5. **Receive `TransactionRef`** – Saves it in `order.payment_first_trans_id` and `order.payment_trans_id`.
6. **Save transaction data** in `order.other_data['cashpay']` (amount, currency, mobile number, payment code, creation date, callback URLs).
7. **Log the payment attempt** in the `PaymentLog` table via `$result->logSuccessfulPayment()` (even though payment is not yet completed).
8. **Return a partial success** with a message asking the user to enter the OTP.

> **Developer note:** At this stage, the order status **does not change** (`processed = false`). The system awaits the customer's OTP input.

### 3.2. Second Stage: Confirming the Payment (`complete`)

When the merchant obtains the OTP from the customer (entered in the payment interface), `complete()` is called and performs the following steps:

1. **Extract `TransactionRef`** from `order.payment_first_trans_id`.
2. **Get the OTP** from the input data (`$this->data['otp']`).
3. **Send a `ConfirmPayment` request** to `POST /api/CashPay/ConfirmPayment` with `TransactionRef` and `TRCode = md5(TransactionRef + OTP)`.
4. **Parse the response**:
   - If `ResultCode == 1` → calls `$result->success()`, which updates the order to `PaidState`, triggers events, and clears the cart.
   - If `ResultCode == 6022` → indicates a required password change (appropriate error message is sent).
   - Any other code → considered a payment failure.
5. **Return the final result** (success or failure).

### 3.3. Additional Functions

#### `operationStatus(string $requestId, string $type = 'InitOP'): array`
Queries the status of a transaction using the `RequestID` (sent in `InitPayment`). The type `InitOP` is for the initial transaction, and `PayOP` for confirmation. Used internally to check for existing transactions.

#### `changePassword(string $newPassword): array`
Changes the merchant's password with CashPay. Should be called when receiving error code `6022` (password expired). The new password is encrypted using `encryptAes256Cbc()` and then sent to `POST /api/CashPay/ChangePass`.

### 3.4. Optional Callback URLs

CashPay supports storing `callback_success_url` and `callback_error_url` if they are passed within the payment data (e.g., from a mobile application). After payment confirmation, the developer can use the following routes:

- `GET /api/v1/yepayment/cashpay/success` – retrieves the success callback URL and uses `RedirectHelper` to redirect to the application or website.
- `GET /api/v1/yepayment/cashpay/cancel` – similar for cancellation.

This ensures full compatibility with mobile applications that require deep links after payment.

---

## 4. Data Structure Stored in `order->other_data['cashpay']`

```php
[
    'transaction_ref'     => '1234567890',                  // Unique reference from CashPay
    'amount'              => 100.00,                        // Amount
    'currency_id'         => 2,                             // Currency number (according to system)
    'currency_code'       => 'YER',                         // Currency code (ISO 4217)
    'target_msisdn'       => '771234567',                   // Customer mobile number
    'customer_cash_pay_code' => 555,                        // Cash payment code
    'desc'                => 'Payment for order #200',      // Product description
    'created_at'          => '2025-01-01T12:00:00Z',       // Creation time
    'requires_otp'        => true,                          // Requires OTP
    'callback_success_url'=> 'myapp://pay/success',         // (optional) Success callback URL
    'callback_error_url'  => 'myapp://pay/error'            // (optional) Error callback URL
]
```

---

## 5. Integrated Scenario for Using CashPay in Your Application

### 5.1. Prerequisites
- A mobile application or website using `Nano.Yepayment` and `Nano.ShopApi`.
- CashPay gateway is configured in the control panel (with Username, Password, Encryption Key, SpId).
- The customer is registered in the CashPay system and has a cash payment code (`CashPay Code`) that is entered during checkout.

### 5.2. Steps from the Developer's Perspective

#### **Step 1: The user selects CashPay as the payment method**
The application sends a request to the `checkout` endpoint with `step=pay` and the payment method identifier `cashpay`.

**Example request:**
```http
POST /api/v1/shop/checkout
Content-Type: application/json

{
  "step": "pay",
  "payment_method_id": 5,
  "target_msisdn": "771234567",
  "customer_cash_pay_code": 555,
  "amount": 100,
  "currency": "YER",
  "desc": "Payment for order",
  "callback_success_url": "myapp://checkout/success",
  "callback_error_url": "myapp://checkout/error"
}
```

#### **Step 2: The system processes the payment via `Checkout2` and `OrderManager`**
- The system calls `OrderManager->setStepPayments()`, which:
  1. Validates shipping addresses and coupons.
  2. Creates a `PaymentService` and `PaymentGateway` based on the selected payment method (CashPay).
  3. Calls `CashPay->process()`.

#### **Step 3: `CashPay->process()` creates the transaction and returns `TransactionRef`**
- The gateway communicates with the CashPay API and creates a new transaction.
- The developer receives the following in the API response:
  ```json
  {
    "status": true,
    "step_status": true,
    "payment_state": null,
    "processed": false,
    "process_data": {
      "paymentResult": {
        "successful": true,
        "message": "Please enter the confirmation code (OTP) sent to your mobile",
        "api_data": {
          "transaction_ref": "1234567890",
          "requires_otp": true
        }
      }
    }
  }
  ```
- **Important here:** `processed` is still `false`, and the order is not yet paid.

#### **Step 4: The application asks the customer to enter the OTP**
- The application displays an OTP input field and asks the customer to enter the number received on their mobile.
- After the OTP is entered, the application sends a request to a dedicated endpoint (or uses `complete` directly).

#### **Step 5: Confirm payment using the OTP**
The application sends a request to `/api/v1/yepayment/cashpay/test-confirm-payment` or a dedicated endpoint that calls `complete()`:

**Example request:**
```http
POST /api/v1/yepayment/cashpay/confirm
Content-Type: application/json
{
  "order_id": 200,
  "otp": "1234"
}
```

#### **Step 6: The order becomes paid**
- If the `ConfirmPayment` operation succeeds, `$result->success()` is called, which:
  - Changes `payment_state` to `PaidState`.
  - Sets `processed = true`.
  - Notifies the system of associated events (sends email, updates inventory, etc.).
  - Clears the user's cart.

#### **Step 7: Final redirection**
- The user is redirected to the payment success page (or to the deep link `myapp://checkout/success`) with the order data.

---

## 6. API Testing Interface (Test UI)

The gateway provides a complete web interface for testing all functions, accessible at:

```
/api/v1/yepayment/cashpay/test-ui
```

### 6.1. Features

- **Authentication test** – verifies the correctness of settings (Username, Password, Encryption Key, SpId).
- **Create transaction (InitPayment)** – enter order number, mobile number, payment code, amount, currency to create a transaction and receive `TransactionRef`.
- **Confirm payment (ConfirmPayment)** – enter OTP to confirm payment and update order to paid.
- **Query status (OperationStatus)** – check transaction status using `RequestID`.
- **Change password (ChangePass)** – change the password when needed.
- **Full flow** – combines transaction creation and confirmation in one step.
- **Real-time statistics** – number of orders, success rate, recent logs.
- **Local logs** – store test results in the browser's `localStorage`.

### 6.2. API Endpoints Used for Testing

| Purpose | Method | Path |
|---------|--------|------|
| Authentication | POST | `/cashpay/test-auth` |
| Create transaction | POST | `/cashpay/test-create-payment` |
| Confirm payment | POST | `/cashpay/test-confirm-payment` |
| Query status | POST | `/cashpay/test-check-status` |
| Change password | POST | `/cashpay/test-change-password` |
| Full flow | POST | `/cashpay/test-full-payment` |
| Statistics | GET | `/cashpay/stats` |

---

## 7. Example Requests and Responses

### 7.1. Creating a New Transaction (First Stage – InitPayment)

**Request:**
```http
POST /api/v1/yepayment/cashpay/test-create-payment
Content-Type: application/json

{
    "order_id": 200,
    "target_msisdn": "771234567",
    "customer_cash_pay_code": 555,
    "amount": 100,
    "currency": "YER",
    "desc": "Payment for order"
}
```

**Response (success):**
```json
{
    "success": true,
    "message": "Please enter the confirmation code (OTP) sent to your mobile",
    "data": {
        "transaction_ref": "1234567890",
        "requires_otp": true,
        "api_data": { ... }
    }
}
```

### 7.2. Confirming Payment (Second Stage – ConfirmPayment)

**Request:**
```http
POST /api/v1/yepayment/cashpay/test-confirm-payment
Content-Type: application/json

{
    "order_id": 200,
    "otp": "1234"
}
```

**Response (success):**
```json
{
    "success": true,
    "message": "Payment confirmed successfully",
    "data": { ... }
}
```

### 7.3. Querying Transaction Status (OperationStatus)

**Request:**
```http
POST /api/v1/yepayment/cashpay/test-check-status
Content-Type: application/json

{
    "request_id": "1234567890",
    "type": "InitOP"
}
```

**Response (success – completed transaction):**
```json
{
    "success": true,
    "status": "success",
    "result_code": 1,
    "message": "Success",
    "data": { ... }
}
```

### 7.4. Changing Password (ChangePass)

**Request:**
```http
POST /api/v1/yepayment/cashpay/test-change-password
Content-Type: application/json

{
    "new_password": "NewStrongPass123"
}
```

**Response (success):**
```json
{
    "success": true,
    "message": "Password changed successfully",
    "data": { ... }
}
```

---

## 8. Integrating the Gateway into Your Project

### 8.1. Registering the Payment Method in `Plugin.php`

```php
if (config('nano.microcart::paymenttypes.allow_yemen_payment', true)) {
    // ... other providers ...
    if (class_exists(\Nano\Yepayment\PaymentTypes\CashPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\CashPay();
    }
}
```

### 8.2. Using the Gateway via `PaymentGateway` (First Stage Only)

```php
use Nano\MicroCart\Classes\Payments\DefaultPaymentGateway;
use Nano\Orders\Models\Order;

$order = Order::find(200);
$gateway = new DefaultPaymentGateway();
$gateway->init($paymentMethod, [
    'target_msisdn' => '771234567',
    'customer_cash_pay_code' => 555,
    'amount' => 100,
    'currency' => 'YER',
    'desc' => 'Payment for order #200',
    'callback_success_url' => 'myapp://pay/success',
]);
$result = $gateway->process($order);
if ($result->successful) {
    // Transaction created – ask the user to enter OTP
    $transactionRef = $order->payment_first_trans_id;
}
```

### 8.3. Manually Confirming the Payment (Second Stage)

After the user enters the OTP, you can call `complete()` directly:

```php
$cash = new \Nano\Yepayment\PaymentTypes\CashPay($order, ['otp' => '1234']);
$result = new PaymentResult($cash, $order);
$completeResult = $cash->complete($result);
if ($completeResult->successful) {
    // Payment successful
}
```

### 8.4. Querying a Transaction Status

```php
$cash = new CashPay();
$status = $cash->operationStatus('1234567890', 'InitOP');
if ($status['success'] && $status['status'] === 'success') {
    // Transaction completed
}
```

---

## 9. Troubleshooting

| Problem | Likely Cause | Solution |
|---------|---------------|------|
| Authentication failed (`auth_failed`) | Incorrect Username/Password/EncryptionKey/SpId | Verify the settings entered in the control panel |
| `payment_creation_failed` with error code 6022 | Password expired | Use `changePassword()` function to change the password |
| Error code 6022 in any other request | Invalid password | Change the password via `changePassword()` |
| Payment confirmation fails (code 35 or other) | Incorrect `OTP` or `TransactionRef` | Verify the OTP sent to the customer and the stored `TransactionRef` |
| Status query returns `failed` | Transaction not created or expired | Check the `RequestID` and operation type (`InitOP` / `PayOP`) |
| `cURL error` or `HttpHelper` exception | Network connectivity issue with CashPay API | Verify the URL is correct and there is internet connectivity |
| `missing_encryption_key` | Encryption key not found in settings | Ensure `cashpay_encryption_key` is entered in the control panel |

---

## 10. Additional Notes

- The gateway follows the **Two‑Step (OTP)** pattern, so the amount is not deducted immediately. You must call `complete()` with the correct OTP to complete the payment.
- `TransactionRef` is stored in `order.payment_first_trans_id` and `order.payment_trans_id` for use in the confirmation and query stages.
- Callback URLs (`callback_success_url`, `callback_error_url`) are stored in `other_data['cashpay']` and used by `RedirectHelper` in the `success`/`cancel` routes.
- If you receive error code **6022** in any response, you should call `changePassword()` immediately (you can show a message to the administrator or direct them to change the password).
- The password in the request header is encrypted using `encryptAes256Cbc()` with the dedicated encryption key. Make sure the key matches the one provided by CashPay.
- Functions `getSupportedCurrencies()`, `getCurrencyId()`, and `isCurrencySupported()` are used to check supported currencies and convert currency codes (e.g., `YER`) to the correct numeric ID (e.g., `2`).

---

## 11. Gateway Code Files

| File | Description |
|-------|-------|
| `CashPay.php` | Main gateway class (extends `PaymentProvider`) |
| `_info.htm` | Settings information template in the control panel |
| `_test_info.htm` | Quick testing tools with interactive buttons (embedded in settings page) |
| `cashpay-ui.htm` | Complete test interface (Test UI) |
| `routes.php` (CashPay section) | Definition of test, statistics, and UI endpoints |
| `Plugin.php` (addition) | Registers the gateway as a payment provider |

---

## 12. Relevant Links

- [SKILL.md for Creating Payment Gateways](./SKILL-en.md)
- [Cash-Pay API Documentation (attached PDF)](./Cash-Pay%20API%20Doc2.pdf)
- [CashPay.php Class](./CashPay.php)
- [routes.php file of Nano.Yepayment](./routes.php)

---

**This documentation is prepared to help developers easily integrate and use the CashPay gateway.**  
For inquiries or technical support, please contact via the official website [nano2soft.com](https://nano2soft.com).
