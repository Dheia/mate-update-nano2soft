## 2026-04-29 – 2026-04-30

### Adding JaibPay (Jaib Wallet) Payment Gateway to NanoSoft Payment System

Within the `Nano.Yepayment` software module

---

### 1. Introduction

A new payment method named **JaibPay** (operating under the trade name **Jaib Wallet**) has been developed and added to the unified payment system in NanoSoft within the `Nano.Yepayment` module. This addition comes in response to the need to support local payment gateways in Yemen, where **Jaib Pay** is one of the electronic payment platforms that enables accepting payments via purchase codes generated from the Jaib application.

JaibPay uses a **Direct Payment** method of type **Execute Buy Online By Code**, where the customer enters the purchase code they previously created in the Jaib app, and the amount is deducted immediately without the need for redirection or additional confirmation (unlike YottaPay which requires OTP). This method is suitable for instant services where payment is made with one click.

The gateway has been developed according to the highest security standards, with tokens stored in Cache, using the unified `HttpHelper` for API requests, and providing integrated test interfaces.

---

### 2. Developed Components

#### 2.1. `JaibPay` – Main Payment Gateway Class

- **Path:** `Nano\Yepayment\PaymentTypes\JaibPay`
- **Inheritance:** Extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function:** Responsible for authentication with Jaib API, executing direct payment, checking transaction status, and managing tokens.

**Main Methods:**

| Method | Description |
|--------|-------------|
| `process(PaymentResult $result)` | Executes direct payment: obtains token, sends `ExeBuy` request, saves `requestID` and `referenceID`, updates order status to paid. |
| `complete(PaymentResult $result)` | Not used in this type (immediate payment), but exists for interface compatibility. |
| `getAuthToken(): ?array` | Requests OAuth 2.0 token from Jaib via `POST /api/v1/TokenAuth/LogAPI`, stores `accessToken` and `pinApi` in Cache. |
| `executeBuy($accessToken, $pinApi, $requestID): array` | Sends `POST /api/v1/BuyOnline/ExeBuy` request with `code` (purchase code), `mobile`, `amount`, `currencyCode`. |
| `checkTransactionStatus($requestID): array` | Queries transaction status via `POST /api/v1/BuyOnline/CheckProgress`. |
| `getApiUrl($type): string` | Builds API URLs (login, buy, refund, check). |
| `parseResponse($response): array` | Converts Guzzle response to PHP array. |
| `settings(): array` | Defines settings fields in the control panel (URL, username, password, agentCode, default currency). |
| `encryptedSettings(): array` | Specifies fields to be stored encrypted (`jaibpay_password`). |

#### 2.2. Partial Files and Settings

| File | Path | Description |
|------|------|-------------|
| `_info.htm` | `paymenttypes/jaibpay/_info.htm` | Displays setup instructions and gateway information in the control panel. |
| `_test_info.htm` | `paymenttypes/jaibpay/_test_info.htm` | Quick test tools within the settings page (Create Payment, Query, Full Test, Clear Logs). |
| `jaibpay-ui.htm` | `views/jaibpay-ui.htm` | Integrated interactive web interface for testing all JaibPay functions (Manual, Automatic, Statistics, Logs). |

#### 2.3. API Endpoints (`routes.php`)

A full set of routes has been added within the `yepayment` group:

| Path | Method | Description |
|------|--------|-------------|
| `/jaibpay/test-auth` | POST | Test authentication (verify username/password/agentCode). |
| `/jaibpay/test-create-payment` | POST | Create a new payment (execute direct payment). |
| `/jaibpay/test-check-status` | GET | Query transaction status using `request_id`. |
| `/jaibpay/test-full-payment` | POST | Full test (create payment + query). |
| `/jaibpay/stats` | GET | Gateway usage statistics (order count, success rate, recent logs). |
| `/jaibpay/test-ui` | GET | Integrated test interface (HTML). |

All routes are protected with `BackendAuth::getUser()` to prevent unauthorized access.

#### 2.4. `Plugin.php` – Payment Provider Registration

The following line was added in the `registerPaymentProviders()` function within the `allow_yemen_payment` section:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\JaibPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\JaibPay();
}
```

#### 2.5. Translation Files

Translation keys were added in `lang/ar/lang.php` and `lang/en/lang.php` for settings, public messages, and errors.

---

### 3. Payment Workflow

JaibPay follows a **Direct** flow (immediate) without redirection or additional confirmation:

1. **Authentication**  
   `getAuthToken()` is called:
   - Checks if `accessToken` and `pinApi` exist in Cache.
   - If not, sends a `POST /api/v1/TokenAuth/LogAPI` request with `userName`, `password`, and `agentCode`.
   - Stores `accessToken` and `pinApi` in Cache for 86000 seconds (slightly less than the actual token validity of 86400).

2. **Execute Buy**  
   When `process()` is called:
   - Validates input (`purchase_code`, `mobile`, `amount`, `currency`).
   - Generates a unique `requestID` (UUID v4).
   - Sends a `POST /api/v1/BuyOnline/ExeBuy` request with:
     ```json
     {
       "pinApi": "tIcI4zm",
       "mobile": "774760761",
       "requestID": "550e8400-e29b-41d4-a716-446655440000",
       "code": "3719",
       "amount": 5000,
       "currencyCode": "YER",
       "notes": "Payment for order #200"
     }
     ```
   - Receives `referenceID` and `msg` ("Operation successful") from the response.
   - Saves `requestID` in `order.payment_first_trans_id` and `referenceID` in `order.payment_trans_id`.
   - Updates order status to `PaidState` via `$result->success()`.

3. **Check Progress (Status Query)**  
   `checkTransactionStatus($requestID)` can be called at any time:
   - Sends a `POST /api/v1/BuyOnline/CheckProgress` request with `pinApi` and `requestID`.
   - Receives `referenceID` if the transaction exists.

4. **Refund** – Not implemented in this version, but supported in the API via `/RefoundBuy` and can be added in the future.

---

### 4. Configuration

To activate JaibPay, the following settings must be added in the payment gateway settings interface in the NanoSoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| Setting | Key | Description |
|---------|-----|-------------|
| Base API URL | `jaibpay_url` | Jaib API address (e.g., `https://www.api2.e-jaib.com:5088`) |
| Test API URL | `jaibpay_test_url` | (Optional) Test environment URL |
| Username | `jaibpay_username` | Merchant username |
| Password | `jaibpay_password` | Merchant password (stored encrypted) |
| Agent Code | `jaibpay_agentcode` | Default value `10004` |
| Default Currency | `jaibpay_default_currency` | `YER` / `USD` / `SAR` (default `YER`) |

**Notes**:
- The `jaibpay_password` field is automatically stored encrypted via `encryptedSettings()`.
- These values can also be set via environment variables in `.env` if supported.

---

### 5. Usage Examples

#### 5.1. Create Payment for an Existing Order (within NanoSoft app)

```php
use Nano\Yepayment\PaymentTypes\JaibPay;
use Nano\Orders\Models\Order;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$provider = new JaibPay($order, [
    'purchase_code' => '3719',
    'mobile'        => '774760761',
    'amount'        => 5000,
    'currency'      => 'YER',
    'notes'         => 'Payment for order #200',
]);

$result = new PaymentResult($provider, $order);
$processed = $provider->process($result);

if ($processed->successful) {
    // Payment succeeded
    $requestID = $order->payment_first_trans_id;
    $referenceID = $order->payment_trans_id;
    return redirect()->route('order.success', $order->id);
} else {
    return back()->withError($processed->message);
}
```

#### 5.2. Query Transaction Status

```php
$jaib = new JaibPay();
$status = $jaib->checkTransactionStatus('550e8400-e29b-41d4-a716-446655440000');
if ($status['success']) {
    echo "Reference ID: " . $status['reference_id'];
}
```

#### 5.3. Test Payment via API (for developers)

```bash
# Test authentication
curl -X POST "https://yourdomain.com/api/v1/yepayment/jaibpay/test-auth" \
  -H "Authorization: Bearer <admin_token>"

# Create a new payment
curl -X POST "https://yourdomain.com/api/v1/yepayment/jaibpay/test-create-payment" \
  -H "Authorization: Bearer <admin_token>" \
  -H "Content-Type: application/json" \
  -d '{"order_id":200,"purchase_code":"3719","mobile":"774760761","amount":5000,"currency":"YER"}'

# Query status
curl -X GET "https://yourdomain.com/api/v1/yepayment/jaibpay/test-check-status?request_id=550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer <admin_token>"
```

#### 5.4. Use the Integrated Test Interface

After logging into the control panel as an admin, open the link:
```
https://yourdomain.com/api/v1/yepayment/jaibpay/test-ui
```

The interface contains:
- "Manual" tab: Create payment, Query, Full test.
- "Automatic" tab: Iterative test (up to 10 times) with success rate display.
- "Statistics" tab: Display total orders, JaibPay orders, successful payments, success rate, recent logs.
- "Logs" tab: Local storage of test results with export and clear capabilities.

---

### 6. Handling Redirections (Deeplinks & Callbacks)

Since JaibPay is a **Direct** type (immediate payment without redirection), `success_url` and `cancel_url` are not used in the core payment flow. However, `success` and `cancel` routes are defined in the class (can be added in the future if the API requires them), but they are currently unused.

In the test endpoints, `RedirectHelper` is used only minimally to redirect to the test page itself. For integration with mobile apps, merchants can provide `callback_success_url` in `order.other_data` (via `RedirectHelper::getCallbackSuccessUrlByOrder()`).

---

### 7. Added Value

- **For developers:** An additional example of a **Direct** type payment gateway (without redirection or OTP), enriching the examples library in `SKILL.md`. Uses `Cache` for token storage, unified `HttpHelper`, and complete test endpoints.
- **For merchants:** Support for a local payment gateway in Yemen (Jaib Pay) based on purchase codes, allowing customers to pay quickly without needing to enter credit card details or wait for an OTP message (instant payment).
- **For end users:** A smooth and fast payment experience using a purchase code from the Jaib app, with support for multiple currencies (YER, USD, SAR).

---

### 8. Integration Testing

The system provides dedicated endpoints for integration testing:

- **Test environment:** The same production URL (`https://www.api2.e-jaib.com:5088`) can be used with test credentials from Jaib (not publicly available; coordination with the operator is required). If Jaib provides a UAT environment, its URL can be entered in the `jaibpay_test_url` setting.
- **Test purchase codes:** According to Jaib documentation, the code `3719` and mobile number `774760761` are used for testing.
- **Integrated test interface:** `/api/v1/yepayment/jaibpay/test-ui` provides all necessary tools to fully test the gateway.

**Quick test steps**:
1. Ensure the login credentials (username/password/agentCode) are correct in the gateway settings.
2. Open the test interface: `/api/v1/yepayment/jaibpay/test-ui`.
3. Click "Test Authentication" to verify credentials.
4. Enter a test purchase code (`3719`), test mobile number (`774760761`), and desired amount (`5000`).
5. Click "Execute Payment" – a new payment will be created.
6. Copy the `Request ID` and use it in the query field to check the status.
7. You can also use "Full Test" to execute both steps together.

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

- **JaibPay does not rely on redirection**: Therefore `success_url` and `cancel_url` are not used in the core payment flow. They can be added later if needed.
- **Token storage in Cache**: `accessToken` and `pinApi` are stored for 86000 seconds (slightly less than the actual token validity) to avoid repeated authentication requests.
- **Use of `HttpHelper`**: All API requests use `HttpHelper::sendJson`, making it easy to trace errors and extend the gateway.
- **Generating `requestID`**: `Str::uuid()` is used to ensure a unique identifier for each transaction.
- **Automated testing**: The `/jaibpay/test-full-payment` endpoint with the `iterations` option in the test interface can be used to test gateway stability under load.
- **Extending methods**: A `refund()` function can be added in the future using the `/RefoundBuy` path already defined in `getApiUrl()`.

---

### 11. Bug Fixes

None – this version is dedicated to adding a new feature only.

---

### 12. Related Links

- [JaibPay API Documentation ](./Docs-JaibPay-en.md)
- [Jaib Pay API Documentation – Login (Login.pdf)](./Login.pdf)
- [Jaib Pay API Documentation – Execute and Query Payment (Jaib Wallet Pay API.pdf)](./Jaib%20Wallet%20Pay%20API.pdf)
- [Postman Collection for Jaib Pay API](./Jaib%20Pay%20API.postman_collection.json)
- [NanoSoft Payment Gateway Development Guide](https://docs.nano2soft.com/payment-gateways)
- [Technical Support Channel](https://nano2soft.com)

---

**Prepared by:**  
NanoSoft Development Team – Electronic Payments Department  
**References:** Dheia Ali, Nano2Soft

---