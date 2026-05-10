## 2026-05-04 – 2026-05-10

**JaibPay Payment Gateway Updates (Jeib Wallet) – May 2026**

### Comprehensive Improvements and Fixes to JaibPay Payment Gateway

Within the `Nano.Yepayment` module

---

### 1. Introduction

After completing the initial release of the **JaibPay** (Jeib Wallet) gateway, a series of essential improvements and fixes were implemented to ensure the gateway works professionally and is fully integrated with the Nano2Soft system. The updates included fixing the authentication function, improving API response handling mechanisms, adding port testing and response processing functions, creating a refund function, and resolving the issue of saving encrypted settings in `PaymentGatewaySettings`.

The gateway was actually tested with the Jaib Pay platform across multiple different payment and refund scenarios, and all tests proved successful integration.

---

### 2. Developed and Updated Components

#### 2.1. `JaibPay` – Main Payment Gateway Class

**New and improved functions:**

| Function | Description | Type |
|--------|-------|-------|
| `normalizeApiResponse($response): array` | Process Jaib API response and convert it into a unified structure containing `success`, `status_code`, `error_code`, `error_message`, `data`, `raw`. Handles HTTP 200 success, HTTP 500 errors, and JSON errors. | **New** |
| `looksLikeJson($string): bool` | Helper function to check if text resembles JSON. | **New** |
| `getSettingsByKey($key, $defaultValue = null)` | Correctly retrieve a setting value from `PaymentGatewaySettings`, supporting encrypted fields via `getEncryptableValue()`. | **New** |
| `testPortConnection(string $url, ?int $port, int $timeout): array` | Test whether a specific port is open on a server. Automatically extracts port from URL, supports DNS analysis, and measures response time. | **New** |
| `refund(string $referenceID, ...): array` | Refund a transaction via `POST /api/v1/BuyOnline/RefoundBuy`. Changes order status to `RefundedState` and stores refund data in `other_data['jaibpay_refund']`. | **New** |
| `getAuthToken()` | **Updated** – Uses `normalizeApiResponse` to display clear error messages, and uses `getSettingsByKey` to correctly fetch the encrypted password. | **Updated** |
| `executeBuy(...)` | **Updated** – Uses `normalizeApiResponse` to display precise error messages like "Invalid purchase code" or "Code already used". | **Updated** |
| `checkTransactionStatus(...)` | **Updated** – Uses `normalizeApiResponse` to improve display of query results. | **Updated** |
| `process(PaymentResult $result)` | **Updated** – Sets `$result->message` from the exception before calling `fail()` to ensure the error message appears in the user interface. | **Updated** |

#### 2.2. `_test_info.htm` – Quick Testing Tools

The quick test interface was updated to include:

| Button | Description |
|------|-------|
| **Test Authentication** | Verify login credentials (username/password/agentCode) |
| **Test Port** | Check connectivity to port 5088 on the remote server (uses `testPortConnection`) |
| **Execute Payment** | Create a new payment transaction |
| **Check Status** | Query transaction status |
| **Comprehensive Test** | Execute payment + query |
| **Refund** | Send a refund request using `referenceID` |

#### 2.3. `routes.php` – API Endpoints

New routes added:

| Path | Method | Description |
|--------|---------|-------|
| `/jaibpay/test-port` | GET | Port test (uses `JaibPay::testPortConnection`) |
| `/jaibpay/test-refund` | POST | Refund a transaction amount |

#### 2.4. `PaymentGatewaySettings` – Fix for Saving Encrypted Settings

The `Nano\MicroCart\Models\PaymentGatewaySettings` class was modified to fix the problem of encrypted values being replaced with empty values when saving settings:

- **Issue:** When opening the gateway settings page and pressing "Save" without re-entering passwords (password-type fields), the encrypted values were replaced with empty values.
- **Solution:** The magic `__set` function was overridden to prevent assigning empty values to encrypted fields if the current value is not empty, ensuring sensitive settings remain unchanged.

---

### 3. Payment Working Cycle After Update

The payment pattern remained **Direct (instant)** as before, but with fundamental improvements in error handling and user experience:

1. **Authentication** – Uses the improved `getAuthToken()` which:
   - Correctly retrieves the encrypted password via `getSettingsByKey()`.
   - Uses `normalizeApiResponse` to provide clear error messages (e.g., "Incorrect username or password").

2. **Payment Execution** – Uses the improved `executeBuy()` which:
   - Uses `normalizeApiResponse` to extract error messages directly from Jaib's response.
   - Displays precise errors such as:
     - `"Invalid purchase code"` (code: 51)
     - `"The code has already been used"` (code: -1026)
     - `"Invalid code number"` (code: 51)

3. **Error Handling** – In `process()`, `$result->message` is set before calling `fail()` to ensure the error message appears in the user interface.

4. **Refund** – The new `refund()` function which:
   - Sends a refund request to `/api/v1/BuyOnline/RefoundBuy`.
   - Changes the order status to `RefundedState`.
   - Stores refund data in `order->other_data['jaibpay_refund']`.

---

### 4. Detailed New Functions

#### 4.1. `normalizeApiResponse` – Unified API Response Processing

```php
private function normalizeApiResponse($response): array
```

**Functionality:** Receives an HTTP response (or exception) and uniformly extracts:
- `success` – operation success
- `status_code` – HTTP code
- `error_code` – error code from Jaib (e.g., `51`, `-100`, `-1026`)
- `error_message` – error message from Jaib (e.g., "Invalid purchase code")
- `data` – result data (`result` from Jaib response)
- `raw` – full raw response

**Usage:** Used in all API functions (`getAuthToken`, `executeBuy`, `checkTransactionStatus`, `refund`) to unify response handling.

#### 4.2. `testPortConnection` – Port Testing

```php
public static function testPortConnection(string $url, ?int $port = null, int $timeout = 5): array
```

**Functionality:** Tests whether a specific port is open on a server. Supports:
- Automatic port extraction from URL (e.g., `https://www.api2.e-jaib.com:5088`).
- Prioritizing the passed parameter port over the URL port.
- Using the default port according to the scheme (443 for HTTPS, 80 for HTTP).
- DNS check and determining if the issue is DNS or the port.

**Usage:** Used in the `jaibpay/test-port` route and in `_test_info.htm` to diagnose connection issues.

#### 4.3. `refund` – Refund

```php
public function refund(string $referenceID, ?string $requestID = null, ?float $amount = null, ?string $currencyCode = null, ?string $notes = null): array
```

**Functionality:** Performs a full refund operation:
- Uses `normalizeApiResponse` to handle the response.
- On success: changes `order->payment_state` to `RefundedState` and stores refund data in `order->other_data['jaibpay_refund']`.
- On failure: displays the error message from Jaib.

---

### 5. Bug Fixes

| Issue | Description | Solution |
|---------|-------|------|
| **Saving encrypted settings with empty values** | When saving the settings page without re-entering the password, encrypted values were replaced with empty values. | Override `__set` in `PaymentGatewaySettings` to prevent assigning empty values to encrypted fields. |
| **Unclear error messages** | When payment failed, it showed `Server error: POST ... 500 Internal Server Error` instead of the actual error message. | Use `normalizeApiResponse` to extract `error.message` from Jaib's response. |
| **`$result->message` empty on failure** | In `process()`, the error message was not passed to `PaymentResult`. | Add `$result->message = $e->getMessage();` before `$result->fail()`. |
| **Fetching encrypted password** | `PaymentGatewaySettings::get('jaibpay_password')` returned empty value for encrypted fields. | Use `getSettingsByKey()` with `getEncryptableValue()`. |
| **`null` error message not appearing** | On `process()` failure, `$result->message` remained `null`. | Improve `catch` so message is set before `fail`. |

---

### 6. Actual Integration Tests

Actual integration tests were conducted with the Jaib Pay platform across all possible scenarios:

#### 6.1. Authentication Test
| Scenario | Result |
|-----------|---------|
| Correct credentials | ✅ Success – `accessToken` and `pinApi` returned |
| Wrong credentials | ✅ Failure – "Invalid username or password" message |

#### 6.2. Port Test
| Scenario | Result |
|-----------|---------|
| Check port 5088 on `www.api2.e-jaib.com` | ✅ Success – Connected successfully |

#### 6.3. Payment Execution Test
| Scenario | Inputs | Result |
|-----------|----------|---------|
| **Correct code + amount + SAR currency** | `code`: valid, `amount`: 5000, `currency`: SAR | ✅ Success – Payment executed |
| **Correct code + wrong amount** | `code`: valid, `amount`: 6000 (exceeds balance) | ❌ Failure – Error message from Jaib |
| **Wrong code + correct amount** | `code`: invalid, `amount`: 5000 | ❌ Failure – "Invalid purchase code" |
| **Correct code + correct amount + SAR currency** | `code`: valid, `amount`: 5000, `currency`: SAR | ✅ Success – Payment executed |
| **Code already used** | `code`: used | ❌ Failure – "The code has already been used" |
| **Correct code + correct amount + YER currency** | `code`: valid, `amount`: 5000, `currency`: YER | ✅ Success – Payment executed |
| **Correct code + correct amount + expired** | `code`: expired | ❌ Failure – Error message from Jaib |

#### 6.4. Refund Test
| Scenario | Result |
|-----------|---------|
| Refund a successful transaction amount | ✅ Success – Refund executed |
| Refund a non-existent transaction | ❌ Failure – "Invalid transaction reference fee" |
| Refund without `reference_id` | ✅ Automatically fetched from order data |
| Attempt to refund an unpaid order | ❌ Failure – "Cannot refund an unpaid order" |

---

### 7. Notes for Developers

- **Using `normalizeApiResponse` has become mandatory** in all API functions to ensure accurate error messages are displayed.
- **The `getSettingsByKey` function** must be used to fetch any encrypted setting instead of direct `PaymentGatewaySettings::get()`.
- **Port testing** is available via `JaibPay::testPortConnection()` and can be used to diagnose any connection issue.
- **Refund** changes the order status to `RefundedState` automatically and stores refund data under `other_data['jaibpay_refund']`.
- **Improvement to `PaymentGatewaySettings`** prevents loss of encrypted settings, but ensure to occasionally clear cache (`Cache::forget('system::settings...')`) after modifications.
- **The `refund` function** requires the `referenceID` of the original transaction (obtainable from `order->payment_trans_id` or `other_data['jaibpay']['reference_id']`).

---

### 8. Related Links

- [JaibPay Documentation (Developer Guide)](./docs/JaibPay/Docs-JaibPay-en.md)
- [Jaib Pay API Documentation – Login (Login.pdf)](./Login.pdf)
- [Jaib Pay API Documentation – Execute and Query Payment (Jaib Wallet Pay API.pdf)](./Jaib%20Wallet%20Pay%20API.pdf)
- [Postman Collection for Jaib Pay API Testing](./Jaib%20Pay%20API.postman_collection.json)
- [Nano2Soft Payment Gateway Development Guide](./SKILL.md)
- [Technical Support Channel](https://nano2soft.com)

---

**This update has been prepared by:**  
Nano2Soft Development Team – Electronic Payments Division  
**Reviewer:** Dheia Ali, Nano2Soft