## 2026-4-9 – 2026-4-25

### Adding YottaPay (Sabacash) Payment Gateway to NanoSoft Payment System

Within the `Nano.Yepayment` software module  

---

### 1. Introduction

A new payment method named **YottaPay** (operating under the trade name **Sabacash**) has been developed and added to the unified payment system in NanoSoft within the `Nano.Yepayment` module. This addition comes in response to the need to support local payment gateways in Yemen, where **YottaGate** is one of the leading electronic payment platforms that enables accepting payments via direct transfer from customer balances using an OTP code.

This addition aims to:
- Enable NanoSoft merchants to accept payments via the YottaGate (Sabacash) system in two secure steps (create transaction then confirm with OTP).
- Provide a seamless payment experience without redirecting the user to external pages (in-app payment).
- Fully integrate with the existing `Nano.Yepayment` system, including transaction management, status checking, and refund operations.
- Support both test (UAT) and production (Live) environments via separate login credentials.

---

### 2. Developed Components

To implement this integration, the following classes were created and modified within the `Nano.Yepayment` module:

#### 2.1 `YottaPay` – Main Payment Gateway Class
- **Path**: `Nano\Yepayment\PaymentTypes\YottaPay`
- **Inheritance**: Extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function**: Responsible for creating payment transactions, confirming them using OTP, checking transaction status, and changing passwords.

**Main Methods**:

| Method | Description |
|--------|-------------|
| `process(PaymentResult $result)` | Create a new payment transaction via YottaGate API (`POST /onLinePayment`). Stores `adjustment.id` in `order.payment_first_trans_id` and returns a message asking for OTP input. |
| `complete(PaymentResult $result)` | Confirm payment using OTP code (`PATCH /onLinePayment`). Updates order status to paid on success. |
| `getAuthToken(): ?string` | Request OAuth 2.0 token from YottaGate via `POST /login` using stored credentials. |
| `checkTransactionStatus($transactionId): array` | Query transaction status using Transaction ID via `GET /checkAdjustmentByTransactionId`. |
| `changePassword($oldPassword, $newPassword): array` | Change merchant account password via `PATCH /changeUserPassword`. |
| `getGateway(): Http` | Returns an `Http` object configured with the appropriate token. |
| `settings(): array` | Defines settings fields in the control panel (URL, username, password, default terminal, default currency). |
| `encryptedSettings(): array` | Specifies fields to be stored encrypted (`yottapay_password`). |

#### 2.2 `Http` – Communication Class (CURL)
- **Path**: `Nano\Yepayment\Classes\Http`
- **Function**: Provides low-level functions for sending HTTP requests (GET, POST, PATCH, JSON, Form-data) using cURL, with proxy and redirect support. Used by `YottaPay` to send requests to YottaGate.

**Main Methods Used**:

| Method | Description |
|--------|-------------|
| `postJsonData($url, $array_post, $headers, $basic)` | Send POST request with `Content-Type: application/json`. |
| `getData($url, $headers)` | Send GET request and return raw text. |
| `getResponsData($response)` | Convert response (object or string) to PHP array. |

#### 2.3 `RedirectHelper` – Redirect Helper Class
- **Path**: `Nano\Yepayment\Classes\RedirectHelper`
- **Function**: Used in test endpoints for smart redirection (supports deeplinks and web). Not directly used in `YottaPay` because payment does not require redirection, but useful for test interfaces.

#### 2.4 `yottapay.php` – Configuration File (Optional)
- **Path**: `config/yottapay.php` (can be added in the future)
- **Function**: Stores YottaGate API connection settings, including endpoint URLs. Currently `PaymentGatewaySettings` is used to store settings.

#### 2.5 `Plugin.php` – Payment Provider Registration
- **Path**: `Nano\Yepayment\Plugin`
- **Function**: Registers `YottaPay` as a payment provider in the `Nano.MicroCart` system via the `registerPaymentProviders()` function when the `allow_yemen_payment` option is enabled.

#### 2.6 `routes.php` – API Endpoints
- **Path**: `routes.php`
- **Function**: Provides endpoints for testing the YottaPay gateway, querying, and statistics.

**YottaPay-specific Endpoints**:

| Path | Method | Description |
|------|--------|-------------|
| `/api/v1/yepayment/yottapay/test-auth` | POST | Test authentication with YottaGate (get token) |
| `/api/v1/yepayment/yottapay/test-create-payment` | POST | Create a test transaction (mimics `process`) |
| `/api/v1/yepayment/yottapay/test-confirm-payment` | POST | Confirm a transaction using OTP |
| `/api/v1/yepayment/yottapay/test-check-status` | GET | Query transaction status via `transaction_id` |
| `/api/v1/yepayment/yottapay/test-full-payment` | POST | Full test (create + confirm + query) |
| `/api/v1/yepayment/yottapay/test-change-password` | POST | Test password change |
| `/api/v1/yepayment/yottapay/stats` | GET | Gateway usage statistics |
| `/api/v1/yepayment/yottapay/test-ui` | GET | Interactive web interface for testing all functions |

#### 2.7 `_info.htm` – Settings Information Template
- **Path**: `paymenttypes/yottapay/_info.htm`
- **Function**: Displays setup instructions in the control panel, with test links and documentation.

#### 2.8 `_test_info.htm` – Quick Test Tools Template
- **Path**: `paymenttypes/yottapay/_test_info.htm`
- **Function**: Embeds quick test tools (test button, statistics display) within the gateway settings page in the control panel.

#### 2.9 `yottapay-ui.htm` – Integrated Test Interface
- **Path**: `views/yottapay-ui.htm`
- **Function**: Full HTML/JS interface for testing all YottaPay functions (manual test, automatic test, statistics, logs).

---

### 3. Payment Workflow

The payment process via YottaPay follows these steps:

1. **Create Payment Transaction**  
   When `process()` is called, the class:
   - Validates inputs (phone number, amount, terminal) via `checkValidate()`.
   - Requests an OAuth 2.0 token via `getAuthToken()` using stored credentials.
   - Sends a `POST` request to `{base_url}/api/accounts/v1/adjustment/onLinePayment` with the data:
     ```json
     {
       "source": { "code": "771234567", "currencyId": "1" },
       "beneficiary": { "terminal": "1", "currencyId": "1" },
       "amount": "1000",
       "amountCurrencyId": "1",
       "note": "Payment for order #200"
     }
     ```
   - Receives `adjustment.id` from the response.
   - Stores `adjustment.id` in `order.payment_first_trans_id` and in `order.other_data['yottapay']`.
   - Returns `PaymentResult` with `successful = true` and a message asking the user to enter OTP (**without redirection**).

2. **User Enters OTP Code**  
   The customer receives an SMS containing an OTP code from YottaGate (sent automatically after transaction creation). The user enters the code in the payment form.

3. **Confirm Payment**  
   `complete()` is called with the OTP code:
   - Sends a `PATCH` request to the same endpoint `/onLinePayment` with the data:
     ```json
     { "id": "816613", "otp": "4320", "note": "Confirm payment" }
     ```
   - Checks for `completed == true` in the response.
   - Updates `order.payment_state` to `PaidState` and saves `transactionId` in `order.payment_trans_id`.
   - Logs the payment as **successful** via `PaymentResult::success()`.

4. **Check Transaction Status (Optional)**  
   `checkTransactionStatus($transactionId)` can be called at any time to send a `GET` request to `/checkAdjustmentByTransactionId` and check the `statusCode` (completed, not-completed, not-exist).

5. **Refund**  
   (Can be added in the future) – YottaGate supports refund operations via `POST /onlineMoneyReturn` and `PATCH /onlineMoneyReturn`, which can be implemented via additional methods in the class.

---

### 4. Configuration

To activate the YottaPay payment method, the following settings must be added in the payment gateway settings interface in the NanoSoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`), or via environment variables (`.env`) if supported:

```ini
# Enable YottaPay
YOTTAPAY_ENABLED=true
YOTTAPAY_URL=https://api.sabacash.com:49901
YOTTAPAY_USERNAME=712988875
YOTTAPAY_PASSWORD=your_password
YOTTAPAY_DEFAULT_TERMINAL=1
YOTTAPAY_DEFAULT_CURRENCY=1
```

**Settings in Control Panel** (`settings()` method in `YottaPay`):

| Setting | Key | Description |
|---------|-----|-------------|
| Base API URL | `yottapay_url` | YottaGate address (e.g., `https://api.sabacash.com:49901`) |
| Username | `yottapay_username` | Merchant username |
| Password | `yottapay_password` | Merchant password (stored encrypted) |
| Default Terminal Number | `yottapay_default_terminal` | Default value (e.g., `"1"`) |
| Default Currency ID | `yottapay_default_currency` | Default value (e.g., `"1"` for Yemeni Rial, `"2"` for Saudi Riyal) |

**Note**: `yottapay_password` is stored encrypted via `encryptedSettings()`.

---

### 5. Usage Examples

#### 5.1. Create Payment Transaction for an Existing Order (within NanoSoft app)

```php
use Nano\Yepayment\PaymentTypes\YottaPay;
use Nano\Orders\Models\Order;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$provider = new YottaPay($order, [
    'source_phone' => '771234567',
    'amount'       => 1000,
    'terminal'     => '1',
    'currency_id'  => '1',
    'note'         => 'Payment for order #200',
]);

$result = new PaymentResult($provider, $order);
$processed = $provider->process($result);

if ($processed->successful) {
    // Store adjustment_id in session or send to frontend
    $adjustmentId = $order->payment_first_trans_id;
    // Show OTP input form to user
    return view('payment.otp_form', ['adjustment_id' => $adjustmentId]);
} else {
    return back()->withError($processed->message);
}
```

#### 5.2. Confirm Payment Using OTP

```php
$order = Order::find(200);
$provider = new YottaPay($order, ['otp' => '4320']);
$result = new PaymentResult($provider, $order);
$confirmed = $provider->complete($result);

if ($confirmed->successful) {
    // Payment succeeded
    return redirect()->route('order.success', $order->id);
} else {
    return back()->withError('OTP code is incorrect or expired');
}
```

#### 5.3. Query Transaction Status

```php
$yotta = new YottaPay();
$status = $yotta->checkTransactionStatus('DF-01-0123456-0123456789');
if ($status['success'] && $status['statusCode'] === 'completed') {
    echo "Transaction completed";
}
```

#### 5.4. Test Payment via API (for developers)

You can use the dedicated test endpoints (require admin privileges):

```bash
# Test authentication
curl -X POST "https://yourdomain.com/api/v1/yepayment/yottapay/test-auth" \
  -H "Authorization: Bearer <admin_token>"

# Create test transaction
curl -X POST "https://yourdomain.com/api/v1/yepayment/yottapay/test-create-payment" \
  -H "Authorization: Bearer <admin_token>" \
  -H "Content-Type: application/json" \
  -d '{"order_id":200,"source_phone":"771234567","amount":1000}'

# Confirm transaction
curl -X POST "https://yourdomain.com/api/v1/yepayment/yottapay/test-confirm-payment" \
  -H "Authorization: Bearer <admin_token>" \
  -H "Content-Type: application/json" \
  -d '{"order_id":200,"adjustment_id":"816613","otp":"4320"}'

# Check status
curl -X GET "https://yourdomain.com/api/v1/yepayment/yottapay/test-check-status?transaction_id=DF-01-..." \
  -H "Authorization: Bearer <admin_token>"
```

#### 5.5. Use the Integrated Test Interface

Open the test interface in your browser (after logging into the control panel as admin):
```
https://yourdomain.com/api/v1/yepayment/yottapay/test-ui
```

The interface contains:
- Step-by-step manual testing (authentication, create transaction, confirm with OTP, check status).
- Comprehensive automated test with configurable iteration count.
- Instant statistics (request count, success rate, latest logs).
- Test logs stored locally (LocalStorage) with export and clear capabilities.

---

### 6. Handling Redirections (Deeplinks & Callbacks)

Since YottaPay does not rely on redirecting the user to external pages, `success_url` and `cancel_url` are not used in the core payment flow. However, in the test endpoints (`/yottapay/test-create-payment` and others), `RedirectHelper` is used to redirect to the test interface or display results in JSON format.

For integration scenarios with mobile apps, merchants can provide `callback_success_url` and `callback_error_url` inside `order.other_data` (e.g., via `Nano\Yepayment\Classes\RedirectHelper::getCallbackSuccessUrlByOrder()`) to redirect the user after payment completion. However, this is not required since payment is completed entirely within the application without leaving.

---

### 7. Added Value

- **For developers**: Adding new payment gateways via inheriting `PaymentProvider` with unified methods (`process`, `complete`). Using the unified `Http` class reduces repetitive communication code. Test endpoints and performance statistics facilitate integration and monitoring.
- **For merchants**: Support for a local payment gateway in Yemen (Sabacash) with a secure two-step (OTP) payment mechanism, without needing to redirect the user to an external site, improving user experience and reducing cart abandonment rates.
- **For end users**: Smooth and fast payment experience using an OTP code sent to their mobile, with support for local currencies (Yemeni Rial and Saudi Riyal).

---

### 8. Integration Testing

The system provides dedicated endpoints for integration testing without needing a real request:

- **Test (UAT) environment**: Test credentials can be obtained from YottaGate (not publicly available at the moment; coordination with the operator is required).
- **Test cards**: No credit cards are needed because payment relies on customer balance and OTP.
- **Integrated test interface**: `/api/v1/yepayment/yottapay/test-ui` provides all necessary tools to fully test the gateway.

**Quick test steps**:
1. Ensure the login credentials (username/password) are correct in the gateway settings.
2. Open the test interface: `/api/v1/yepayment/yottapay/test-ui`.
3. Click "Test Authentication" to verify credentials.
4. Enter a test phone number (e.g., `771234567`) and the desired amount.
5. Click "Create Transaction" – a mock transaction will be created.
6. Use a test OTP code (e.g., `1234`) and click "Confirm Payment".
7. Check the transaction status via "Check Status".

---

### 9. Compatibility

- **NanoSoft App:** v2.x
- **PHP:** 8.0 / 8.1 / 8.2
- **NanoSoft Add-ons:**  
  - `Nano.MicroCart` (>=2.0)  
  - `Nano.Helpers` (>=1.2)  
  - `Nano.Orders` (>=1.5)
- **Databases:** Supports MySQL, PostgreSQL, SQLite (via Eloquent)

---

### 10. Developer Notes

- **YottaPay does not rely on redirection**: Therefore `success_url` and `cancel_url` are not used in the core payment flow. They can be added later if needed.
- **Token storage**: Currently a new token is requested for each payment operation. To improve performance, a caching layer can be added inside `getAuthToken()` to store the token for 3600 seconds.
- **Dependency on `HttpHelper`**: All API requests use the unified `Http` class, making it easy to trace errors and extend the gateway.
- **Automated testing**: The `/yottapay/test-full-payment` endpoint with the `iterations` option can be used to test gateway stability under load.
- **Extending methods**: `refund()` and `getTransactionDetails()` methods can be added in the future according to YottaGate API documentation.

---

### 11. Bug Fixes

None – this version is dedicated to adding a new feature only.

---

### 12. Related Links

- [YottaGate API Documentation – Login & Change Password](./yottaPay%20Login%20-%20Change%20My%20Password%20API%20Documentation%20V.1.1.pdf)
- [YottaGate API Documentation – Create & Confirm Payment (Online Payments v1.3)](./API%20Online%20Payments%20v%201.3.pdf)
- [YottaGate API Documentation – Money Return (Online Payment Return v1.4)](./API%20Online%20Payment%20Return%20v%201.4.pdf)
- [YottaGate API Documentation – Status Check (Online Status Check v1.3)](./API%20Online%20Status%20Check%20v1.3.pdf)
- [Postman Collection for Sabacash API](./Sabacash%20online%20payment.postman_collection.json)
- [NanoSoft Payment Gateway Development Guide](https://docs.nano2soft.com/payment-gateways)
- [Technical Support Channel](https://nano2soft.com)

---

**Prepared by:**  
NanoSoft Development Team – Electronic Payments Department  
**References:** Dheia Ali, Nano2Soft

---


---


