# Update 2026-4

**Updates month four April**

## 2026-3-31 - 2026-4-1

### Adding support for default country, state, and directorate when adding addresses via API (version 2)

**The `Nano.LocationApi` module has been enhanced with new options that allow using default values for Country, State, and Directorate when creating a new address through the API in its second version (`api_version=2` or `v2`).**

Previously, developers were required to pass the IDs of these entities when adding an address; otherwise the request would fail. After this update, it is possible to enable automatic assignment of default values when they are not provided, making it easier to integrate applications with the system and reducing errors related to location data.

Three new options have been added to the configuration file `config.php`, and the `addV2` method in the `Address` controller has been modified to implement this behavior according to the settings.

---

### 1. Updated components

| Behavior | Component | Description |
|----------|-----------|-------------|
| Adding address (version 2) | `Address@addV2` | Logic added to use default values when country/state/directorate are not provided. |
| Configuration | `config.php` (`address` section) | New keys added to control the activation of default values. |

#### New configuration options

| Key | Environment variable | Description |
|-----|----------------------|-------------|
| `is_allow_default_country_in_add` | `NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD` | If `true`, the default country (from `RainLab\Location\Models\Country::getDefault()`) will be set when `country_id` is not provided. |
| `is_allow_default_state_in_add` | `NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_STATE_IN_ADD` | If `true`, the default state (from `RainLab\Location\Models\State::getDefault()` or the first state belonging to the selected country) will be set when `state_id` is not provided. |
| `is_allow_default_directorate_in_add` | `NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_DIRECTORATE_IN_ADD` | If `true`, the default directorate (from `RainLab\Location\Models\Directorate::getDefault()` or the first directorate belonging to the selected state) will be set when `directorate_id` is not provided. |

All options are `false` by default to maintain backward compatibility.

---

### 2. Code details

#### 2.1. Default value assignment logic in `addV2`

The following code was added after preparing the address data `$data_add` and before creating the record:

```php
// Set default country if enabled and country_id was not provided
if(\Config::get('nano.locationapi::address.is_allow_default_country_in_add',false) && (! isset($data_add['country_id']) || empty($data_add['country_id'])))
{
    $def = \RainLab\Location\Models\Country::getDefault();
    if($def) {
        $data_add['country_id'] = $def->id;
    }
}

// Set default state if enabled, country_id is present, and state_id was not provided
if(\Config::get('nano.locationapi::address.is_allow_default_state_in_add',false) && (isset($data_add['country_id']) && !empty($data_add['country_id'])) && (! isset($data_add['state_id']) || empty($data_add['state_id'])))
{
    $def = \RainLab\Location\Models\State::getDefault();
    $def_state_id = null;
    if($def) {
        $def_state_id = $def->id;
    }
    
    $nameList = \RainLab\Location\Models\State::getNameList($data_add['country_id']);
    
    if(!empty($nameList)) {
        if($def_state_id && isset($nameList[$def_state_id]))
            $data_add['state_id'] = $def_state_id;
        else
            $data_add['state_id'] = key($nameList);
    }
}

// Set default directorate if enabled, country_id and state_id are present, and directorate_id was not provided
if(\Config::get('nano.locationapi::address.is_allow_default_directorate_in_add',false) && (isset($data_add['country_id']) && !empty($data_add['country_id'])) && (isset($data_add['state_id']) && !empty($data_add['state_id'])) && (! isset($data_add['directorate_id']) || empty($data_add['directorate_id'])))
{
    $def = \RainLab\Location\Models\Directorate::getDefault();
    $def_directorate_id = null;
    if($def) {
        $def_directorate_id = $def->id;
    }
    
    $nameList = \RainLab\Location\Models\Directorate::getNameList($data_add['state_id']);
    
    if(!empty($nameList)) {
        if($def_directorate_id && isset($nameList[$def_directorate_id]))
            $data_add['directorate_id'] = $def_directorate_id;
        else
            $data_add['directorate_id'] = key($nameList);
    }
}
```

2.2. Priority order

Default values are applied in sequence:

1. Default country – uses Country::getDefault().
2. Default state – uses State::getDefault() if available; otherwise the first state belonging to the selected country is chosen.
3. Default directorate – uses Directorate::getDefault() if available; otherwise the first directorate belonging to the selected state is chosen.

2.3. Backward compatibility

· The new configuration options are disabled by default (false), so existing applications are unaffected.
· Changes are limited to the addV2 method (second version of the API) and do not affect addV1.
· Developers can enable the features as needed via the .env file or by changing the configuration values directly.

---

3. Usage examples

3.1. Adding an address with only default country enabled

```env
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD=true
```

```json
POST /api/v1/location/address/add?api_version=v2
{
    "address_1": "Nile Street",
    "address_2": "Next to the bank"
}
```

In this case, country_id will be automatically set to the default country.

3.2. Adding an address with default country and state enabled

```env
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD=true
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_STATE_IN_ADD=true
```

```json
POST /api/v1/location/address/add?api_version=v2
{
    "address_1": "Nile Street",
    "address_2": "Next to the bank"
}
```

Both country_id and state_id will be automatically set to their default values.

3.3. Adding an address with country provided and default state enabled

```json
POST /api/v1/location/address/add?api_version=v2
{
    "address_1": "Nile Street",
    "country_id": 1
}
```

If default state is enabled, the appropriate state for country 1 will be selected.

3.4. Disabling all defaults (old behavior)

```env
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD=false
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_STATE_IN_ADD=false
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_DIRECTORATE_IN_ADD=false
```

In this case, any request missing country_id, state_id, or directorate_id will fail as before.

---

4. Added value

· For developers: Reduces the need to fetch country/state/directorate IDs before adding an address, simplifying code in client applications (mobile apps, frontends).
· For end users: A smoother experience when adding an address, as they can leave optional fields empty without needing to enter them manually.
· For the system: Improves API flexibility and intelligence while preserving backward compatibility.
· Extensibility: Adding settings via config.php allows per‑project customization without modifying the core code.

---

5. Conclusion

This update is a significant enhancement to the Nano.LocationApi module, giving developers control over using default values for location entities (country, state, directorate) when creating an address via the API. Through flexible configuration and precise implementation in the addV2 method, we achieved a balance between simplicity and flexibility, while maintaining stability for existing projects that do not require this feature.

---

Note: For more details about the module’s configuration and how to use the second version of the API, please refer to the Nano.LocationApi documentation or review the examples provided above.

## 2025-9-17 - 2026-4-2

### Adding ThawaniPay Payment Gateway to the NanoSoft Payment System

Within the `Nano.Yepayment` software module

---

### 1. Introduction

A new payment method named **ThawaniPay** has been developed and added to the unified payment system in NanoSoft within the `Nano.Yepayment` module. This addition comes in response to the need to support regional payment gateways in the Sultanate of Oman, as **Thawani Pay** is the first e-payment gateway licensed by the Central Bank of Oman (CBO).

This addition aims to:
- Enable NanoSoft merchants to accept payments via credit and debit cards through the Thawani gateway.
- Provide a seamless and secure payment experience with user redirection to a hosted payment page.
- Fully integrate with the existing `Nano.Yepayment` system, including transaction management, refunds, and recurring subscriptions.
- Support both testing (UAT) and live (Production) environments via separate API keys.

---

### 2. Developed Components

To implement this integration, the following classes were created and modified within the `Nano.Yepayment` module:

#### 2.1 `ThawaniPay` – Main Payment Gateway Class
- **Path**: `Nano\Yepayment\PaymentTypes\ThawaniPay`
- **Inheritance**: Extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function**: Responsible for creating payment sessions, handling notifications, querying payment status, and processing refunds.

**Main Methods**:

| Method | Description |
|--------|-------------|
| `process(PaymentResult $result)` | Create a new payment session via Thawani API, store `session_id` in `order.payment_first_trans_id`, and return the redirection link. |
| `complete(PaymentResult $result)` | Confirm payment after the user returns from the Thawani gateway (currently used for verification and update). |
| `createPaymentThawani($order, $is_test)` | Static method to create a payment session based on an order object or invoice data. |
| `checkSessionPay($options)` | Static method to query the status of a payment session (`session_id`, `invoice`, or `reference`). |
| `checkAndCompletePay($options)` | Static method that combines session checking and order status update if paid. |
| `createRedirectUrl($session_id, $is_test)` | Generate the redirection URL to the Thawani gateway in the format `https://checkout.thawani.om/pay/{session_id}?key={publishable_key}`. |
| `createRequestDataByOrder($order, $is_test_mod)` | Build request data (products, metadata, success_url, cancel_url) from an Order object. |
| `createRequestProcessDataByOrder($order, $is_test_mod)` | Process order data (convert currency to smallest unit, calculate total_thawani). |

#### 2.2 `Http` – Communication Class (CURL)
- **Path**: `Nano\Yepayment\Classes\Http`
- **Function**: Provides low-level methods for sending HTTP requests (GET, POST, JSON, Form-data) using cURL, with support for proxy and redirection.

**Main Methods**:

| Method | Description |
|--------|-------------|
| `getData($url, $headers)` | Send a GET request and return raw text or error message. |
| `postJsonData($url, $array_post, $headers, $basic)` | Send a POST request with `Content-Type: application/json`. |
| `postData($url, $array_post, $headers, $basic)` | Send a POST request with `application/x-www-form-urlencoded`. |
| `getResponsData($response)` | Convert the response (object or string) to a PHP array, supporting multiple formats (JSON, Guzzle Response, etc.). |

#### 2.3 `RedirectHelper` – Helper Class for Redirection
- **Path**: `Nano\Yepayment\Classes\RedirectHelper`
- **Function**: Provides advanced mechanisms for redirection to applications (deeplinks) or websites, with support for token encoding, sensitive data filtering, and URL validation.

**Main Methods**:

| Method | Description |
|--------|-------------|
| `redirectToApp($callbackUrl, $data, $forceJson, $queryParams, $deepLinkSchemes, $is_force_deep_list)` | Smart redirection supporting deeplinks, JSON, and web. |
| `isValidDeepLink($url, $deepLinkSchemes, $is_force_deep_list)` | Check whether the URL is a deeplink (scheme other than http/https). |
| `filterSensitiveData($data, $sensitiveKeys)` | Remove sensitive fields (passwords, tokens, secrets) from data before sending. |
| `redirectUrlWithToken($callbackUrl, $data, $token, $queryParams)` | Store large data in Cache and return a URL with a token. |
| `getNestedValue($array, $key, $default, $delimiter)` | Extract a value from a nested array using dot notation (e.g., 'thawani.callback_success_url'). |

#### 2.4 `thawanipay.php` – Configuration File
- **Path**: `config/thawanipay.php` (within the module)
- **Function**: Stores Thawani API connection settings, including environment keys (test/live) and various endpoint URLs.

**Main Content**:
```php
return [
    'api_key' => env('THAWANIPAY_API_KEY', '...'),
    'secret_key' => env('THAWANIPAY_SECRET_KEY', '...'),
    'publishable_key' => env('THAWANIPAY_PUBLISHABLE_KEY', '...'),
    'url' => [
        'base' => 'https://checkout.thawani.om/api/v1',
        'test' => 'https://uatcheckout.thawani.om/api/v1/',
        'payment' => 'checkout/session',
        'redirect_pay' => 'https://checkout.thawani.om/pay/',
        'redirect_pay_test' => 'https://uatcheckout.thawani.om/pay/',
    ],
    'test' => [...],
];
```

#### 2.5 `Plugin.php` – Registering the Payment Provider
- **Path**: `Nano\Yepayment\Plugin`
- **Function**: Registers `ThawaniPay` as a payment provider in the `Nano.MicroCart` system via the `registerPaymentProviders()` method when the `allow_oman_payment` option is enabled.

#### 2.6 `routes.php` – API Endpoints
- **Path**: `routes.php`
- **Function**: Provides endpoints to handle Thawani gateway responses (success, cancel), query (retrieve), and testing (test).

**Endpoints**:
| Route | Description |
|-------|-------------|
| `GET /api/v1/thawani/success` | Handle successful payment: call `checkAndCompletePay`, then redirect to `callback_success_url` (if exists) or show a message. |
| `GET /api/v1/thawani/cancel` | Handle payment cancellation: redirect to `callback_error_url` or show a message. |
| `GET /api/v1/thawani/retrieve` | Query payment session status (parameters: `session_id`, `type`, `is_test`). |
| `GET /api/v1/thawani/test` | Integrated test interface to create a test payment session with options to redirect or return JSON. |

#### 2.7 `_info.htm` – Settings Info Template
- **Path**: `paymenttypes/thawanipay/_info.htm`
- **Function**: Display setup instructions in the control panel, with links to testing, documentation, and test cards.

---

### 3. Payment Workflow

The payment process via ThawaniPay follows these steps:

1. **Create a Payment Session**  
   When `process()` is called, the class:
   - Validates the request (no existing payment).
   - Calls `createRequestDataByOrder()` to build invoice data (products, amount, currency, success and cancel URLs).
   - Calls `createSessionPay()` to send a `POST` request to `https://.../checkout/session` with the `thawani-api-key`.
   - Receives a `session_id` from the response.
   - Stores the `session_id` in `order.payment_first_trans_id` and in `order.other_data['thawani']`.
   - Returns the redirection URL via `createRedirectUrl()`.

2. **Redirect to Thawani Gateway**  
   The user is redirected to `https://checkout.thawani.om/pay/{session_id}?key={publishable_key}`.

3. **Complete Payment**  
   The user enters their card details on Thawani's secure page.

4. **Handle Notification (Webhook via Endpoint)**  
   After payment completion, Thawani redirects the user to the pre‑defined `success_url`.  
   The `/thawani/success` endpoint:
   - Receives the `session_id` parameter (or extracts it from the URL).
   - Calls `ThawaniPay::checkAndCompletePay()` which:
     - Calls `retrieveSessionPayTest()` to query the session via the API.
     - Checks if `payment_status == 'paid'`.
     - Updates the order status to "paid" via `PaymentResult::success()`.
     - Logs the payment record in `PaymentLog`.
   - If `callback_success_url` exists in `order.other_data`, redirects to it (via deeplink or query string).  
   - Otherwise, displays a success message directly.

5. **Redirect to Failure Page**  
   If the user cancels or the payment fails, they are redirected to `/thawaniy/cancel`, and are then redirected to `callback_error_url` if it exists.

---

### 4. Configuration

To enable the ThawaniPay payment method, add the following settings to your environment variables (`.env`) or through the payment gateway settings interface in the NanoSoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`):

```ini
# Enable ThawaniPay
THAWANI_ENABLED=true
THAWANI_MODE=test   # test / live

# Test (UAT) Keys
THAWANI_TEST_API_KEY=rRQ26GcsZzoEHZvLYDbn9C9et
THAWANI_TEST_SECRET_KEY=rRQ26GcsZzP2HZvLYDbn9C9et
THAWANI_TEST_PUBLISHABLE_KEY=HGvJghr9tlN9gr4DVYt0qyBy
THAWANI_TEST_SUCCESS_URL=https://yourdomain.com/api/v1/thawani/success
THAWANI_TEST_CANCEL_URL=https://yourdomain.com/api/v1/thawani/cancel

# Live (Production) Keys
THAWANI_LIVE_API_KEY=...
THAWANI_LIVE_SECRET_KEY=...
THAWANI_LIVE_PUBLISHABLE_KEY=...
THAWANI_LIVE_SUCCESS_URL=...
THAWANI_LIVE_CANCEL_URL=...
```

**Settings in the Control Panel** (`settings()` method in `ThawaniPay`):
| Setting | Description |
|---------|-------------|
| `thawanipay_url` | Base API URL (default: `https://checkout.thawani.om/api/v1`). |
| `thawanipay_api_key` | API key (required). |
| `thawanipay_secret_key` | Secret key. |
| `thawanipay_publishable_key` | Publishable key (used in the redirection URL). |

**Note**: Sensitive values are stored encrypted via `encryptedSettings()`.

---

### 5. Usage Examples

#### 5.1. Create a Payment Session for an Existing Order

```php
use Nano\Yepayment\PaymentTypes\ThawaniPay;
use Nano\Orders\Models\Order;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(123);
$provider = new ThawaniPay($order, [
    'is_test' => true,  // use test environment
]);

$result = new PaymentResult($provider, $order);
$processed = $provider->process($result);

if ($processed->successful && $processed->redirect) {
    // Redirect the user to the Thawani gateway
    return redirect()->away($processed->redirectUrl);
} else {
    // Display the error message
    return back()->withError($processed->message);
}
```

#### 5.2. Query the Status of a Payment Session

```php
$options = [
    'session_id' => 'checkout_xxxxx',
    'type' => 'session',   // session, invoice, reference
    'is_test' => true,
];
$result = ThawaniPay::checkSessionPay($options);

if ($result['status'] && ThawaniPay::checkPaidSessionId($result)) {
    // Payment was successful
}
```

#### 5.3. Complete Payment After Returning from Thawani Gateway (in the endpoint)

```php
// success
$options = Input::get();
$options['type'] = 'session';
$result = ThawaniPay::checkAndCompletePay($options);

if ($result['status'] && $result['data']['is_complete_pay_order']) {
    // Order status updated automatically
    $callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order->id);
    return RedirectHelper::redirectToApp($callbackUrl, $result, false, ['order_id'], ['*']);
}
```

#### 5.4. Test Payment via API

Use the dedicated test endpoints:

```bash
# Create a test payment session and redirect
GET /api/v1/thawani/test?is_test=1&is_redirect=1&order_id=200

# Query a session status
GET /api/v1/retrieve?is_test=1&type=session&session_id=checkout_xxxxx
```

---

### 6. Handling Redirections (Deeplinks & Callbacks)

`RedirectHelper` is used to redirect to external applications (e.g., mobile apps) via deeplinks or to web pages. The `callback_success_url` and `callback_error_url` are extracted from `order.other_data` using methods such as:

```php
$callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, [
    'callback_success_url',
    'thawani.callback_success_url'
]);
```

If the URL is a deeplink (scheme other than `http`/`https`), a URL with status parameters (`status`, `code`, `message`, `token`) is built and redirection is performed. If it is a regular web URL, redirection is performed with data passed as query parameters or via a token if the data is large.

**`RedirectHelper` Features**:
- Automatically filters sensitive data (`password`, `token`, `secret`).
- Supports redirection via `token` for large data (cached for 5 minutes).
- Validates URLs against allowed domains or custom deeplink schemes.

---

### 7. Added Value

- **For developers**: Adding new payment gateways by extending `PaymentProvider` with unified methods (`process`, `complete`). Using the `Http` and `RedirectHelper` classes reduces repetitive code for communication and redirection.
- **For merchants**: Support for a locally licensed payment gateway in the Sultanate of Oman, with easy switching between test and live environments via environment variables or settings.
- **For end users**: A seamless and secure payment experience without having to leave the site for long, with support for local and international credit and debit cards.

---

### 8. Integration Testing

The system provides dedicated endpoints for integration testing without needing a real request:

- **Thawani Test Cards**: You can use cards from [Thawani documentation](https://thawani-technologies.stoplight.io/docs/thawani-ecommerce-api/7c0f75e1668d7-thawani-test-card) (e.g., `4242 4242 4242 4242` with any future expiry date and any random CVV).

---

### 9. Conclusion

Adding the `ThawaniPay` payment method to the `Nano.Yepayment` system is an important step in expanding the range of payment solutions provided by NanoSoft to include regional licensed gateways in the Sultanate of Oman. Thanks to the modular design based on `PaymentProvider`, powerful helper classes (`Http`, `RedirectHelper`), and ready‑to‑use endpoints, merchants and developers can seamlessly integrate ThawaniPay into their projects while ensuring a reliable and secure payment experience. As development continues, more regional and international payment gateways will be supported to meet the needs of different markets.

### Development and Testing Period

The development of the ThawaniPay payment method and its full integration into the `Nano.Yepayment` system took a period spanning from **September 17, 2025** to **April 2, 2026**. This period was not limited to writing code only, but rather included multiple stages of integration testing in the UAT environment and verifying the entire payment flow, including session creation, webhook handling, deeplink redirection, and handling failure and refund cases. The development process also included continuous improvements to the `Http` class to support secure communication, the development of `RedirectHelper` for smart redirection, and the building of integrated API endpoints for testing and monitoring. The stability of the gateway in the production environment was confirmed before its official adoption, and all use cases were documented to ensure a seamless and secure payment experience.

