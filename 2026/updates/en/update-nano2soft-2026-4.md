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

## 2026-3-15 - 2026-04-03

**Launch of `Nano.FileUpload` Plugin – A Centralized File Upload Management System for NanoSoft Applications**

### Launch of `Nano.FileUpload` Plugin – A Centralized File Upload Management System

The **`Nano.FileUpload`** plugin has been launched as an independent software plugin within the NanoSoft system, aiming to unify and manage file upload operations across API endpoints in all system plugins. The plugin provides an integrated structure for registering models that contain upload fields, verifying user permissions for different user types (backend/frontend), managing temporary files before record saving, and providing ready-to-use API endpoints with a unified response structure.

The plugin is designed to operate independently from applications, allowing any other plugin to easily register with it via dedicated methods or events, ensuring reuse of upload and permission logic without code duplication.

---

### 1. Introduction

In response to the recurring need to add file upload fields in many NanoSoft plugins (such as products, users, orders, etc.), the `Nano.FileUpload` plugin has been developed as a centralized solution that manages:

- Registration of models and upload fields with settings (maximum size, allowed types, etc.).
- Specification of allowed user types (backend / frontend) at the model or field level.
- Assignment of granular permissions for each operation (add, edit, delete, view) via an integrated permission system.
- Support for temporary upload using temporary session keys to link files with unsaved models.
- Provision of a RESTful API with OAuth 2.0 authentication.

The plugin is built on separation of concerns using specialized classes (Registry, Service, UserManager, Controller), making it easily extensible and maintainable.

---

### 2. Update Objectives

- **Create an independent software plugin**: Provide `Nano.FileUpload` as a reusable package in all NanoSoft projects.
- **Central model registration**: Allow any model to be registered with its field settings via the `registerFileUploadFields` method in plugins or via the `nano.api.fileupload.registerModels` event.
- **Support user types and permissions**: Handle `backend` and `frontend` users separately, with the ability to customize permissions for each operation.
- **Temporary upload**: Enable file upload before model saving using temporary session keys, then link them after saving.
- **Unified API**: Provide endpoints for single and multiple uploads, deletion, and retrieval, with a uniform response structure and OAuth security.
- **Comprehensive documentation**: Prepare complete documentation in Arabic covering classes, examples, and endpoints (now translated to English).

---

### 3. Developed Components

#### 3.1 `Nano.FileUpload` Plugin

- **Name:** `Nano.FileUpload`
- **Namespace:** `Nano\FileUpload`
- **Goal:** Provide a centralized system for managing file uploads in NanoSoft applications with support for permissions and different user types.
- **Basic Structure:**
  - `classes/FileUploadRegistry.php` – Registry of registered models
  - `classes/FileUploadService.php` – Service layer to execute upload, delete, and retrieval operations
  - `classes/FileUploadUserManager.php` – Management of current user and permission verification
  - `controllers/FileUploadController.php` – API controller
  - `routes.php` – Endpoints
  - `Plugin.php` – Plugin initialization, route registration, and events
  - `lang/ar/lang.php` – Arabic language file
  - `docs/FileUpload/` – Documentation in Arabic (now translated)

#### 3.2 `FileUploadRegistry` Class

Located at `Nano\FileUpload\Classes\FileUploadRegistry`, this is the central registry that manages model definitions and upload fields.

**Core Methods:**

| Method | Description |
|--------|-------------|
| `registerModel($modelClass, $config)` | Register a model with its settings. |
| `getRegisteredModels()` | Retrieve all registered models. |
| `isModelRegistered($modelClass)` | Check if a model is registered. |
| `getModelConfig($modelClass)` | Get the configuration of a model. |
| `getFieldConfig($modelClass, $field)` | Get the configuration of a specific field. |
| `isUserTypeAllowed($modelClass, $userType)` | Check if a user type is allowed. |
| `can($modelClass, $operation, $userType, $user, $field)` | Check permission for a specific operation. |
| `getFieldConstraints($modelClass, $field)` | Get field constraints (size, types). |
| `updateModelConfig($modelClass, $config)` | Update the configuration of a registered model. |

**Model Configuration Structure:**

```php
[
    'enabled' => true,
    'allowed_user_types' => ['backend', 'frontend'],
    'permissions' => [
        'add'    => null,
        'edit'   => null,
        'delete' => null,
        'view'   => null,
    ],
    'fields' => [
        'image' => [
            'type' => 'image', // file, image, multiple
            'label' => 'Main image',
            'max_filesize' => 2048,
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
            'multiple' => false,
            'permissions' => [ /* override model permissions */ ]
        ],
    ],
    'defaults' => [ /* default settings */ ]
]
```

#### 3.3 `FileUploadService` Class

Located at `Nano\FileUpload\Classes\FileUploadService`, this class is responsible for executing file upload, deletion, and retrieval operations.

**Core Methods:**

| Method | Description |
|--------|-------------|
| `generateTempSessionKey($modelClass, $field, $userId)` | Generate a temporary session key. |
| `upload($modelClass, $field, $fileData, $options)` | Upload a single file. |
| `uploadMultiple($modelClass, $field, $filesData, $options)` | Upload multiple files. |
| `deleteFile($fileId, $modelClass, $field, $user)` | Delete a file with permission check. |
| `getFiles($modelClass, $field, $modelId, $options)` | Retrieve files associated with a model. |
| `attachTempFiles($model, $field, $tempSessionKey)` | Link temporary files to a saved model. |
| `validateFile($modelClass, $field, $fileData)` | Validate a file (size, type). |

#### 3.4 `FileUploadUserManager` Class

Located at `Nano\FileUpload\Classes\FileUploadUserManager`, this is the unified user and permission manager.

**Core Methods:**

| Method | Description |
|--------|-------------|
| `setUserResolver(callable $resolver)` | Set a custom user resolver. |
| `setUser($user)` | Set the user directly. |
| `getUser()` | Get the current user (with caching). |
| `clearUser()` | Reset the cached user. |
| `setPermissionChecker(callable $checker)` | Set a custom permission checker. |
| `getUserType()` | Determine the user type (backend/frontend/guest). |
| `getId()` | Get the current user ID. |
| `checkPermission($operation, $permissions, $user)` | Check permission. |

#### 3.5 `FileUploadController` Controller

Located at `Nano\FileUpload\APIControllers\FileUploadController`, it provides RESTful endpoints protected by OAuth. All methods call `FileUploadService` and return responses in a unified structure.

**Main Endpoints:**

| Route | Method | Description |
|-------|--------|-------------|
| `/upload` | POST | Upload a single file (multipart or base64). |
| `/upload-multiple` | POST | Upload multiple files. |
| `/delete/{id}` | DELETE | Delete a file. |
| `/files` | GET | Retrieve files associated with a model or temporary key. |

#### 3.6 Routing File (routes.php)

A comprehensive routing file was created, providing all endpoints within an external group with the base path `/api/v1/fileupload`, and an internal group protected by the `oauth-users` middleware.

---

### 4. Workflow (New Flow)

1. **Model Registration (once):**
   - In another plugin, the `registerFileUploadFields()` method is executed, or the plugin listens to the `nano.api.fileupload.registerModels` event.
   - `FileUploadRegistry` collects the definitions and registers them in memory.

2. **File Upload (via API):**
   - The client sends a `POST /upload` request with `model_class`, `field`, and the file (or base64).
   - `FileUploadController` verifies the existence of the model and field.
   - It calls `FileUploadService::upload` with the options.
   - `FileUploadService` checks permissions via `FileUploadRegistry::can` using the current user from `FileUploadUserManager`.
   - The file is uploaded via `Base64::onUpload` (from `Nano.Api`).
   - If no saved model is passed, a temporary session key is generated and returned in the response.

3. **Linking Temporary Files:**
   - After saving the model (e.g., a new product), the developer calls `FileUploadService::attachTempFiles` to move the temporary files to the model.

4. **Retrieving Files:**
   - `GET /files` can be called with `model_id` to retrieve files associated with a model, or with `temp_session_key` to retrieve temporary files.

---

### 5. Key Achievements and Features

- **Integrated Software Plugin:** `Nano.FileUpload` has been launched as a centralized solution for managing file uploads in all NanoSoft plugins.
- **Flexible Model Registration:** Support for registration via plugin methods or events, with detailed settings per field.
- **User Type Support:** Handle `backend` and `frontend` users separately, with the ability to specify `allowed_user_types` per model or field.
- **Granular Permissions:** Permissions for each operation (`add`, `edit`, `delete`, `view`) at the model or field level, integrating `backend` permission system (`hasAccess`/`hasPermission`) and allowing custom checks for `frontend`.
- **Temporary Upload:** Ability to upload files before saving a model using temporary session keys (`temp_session_key`), then link them later via `attachTempFiles`.
- **Ready-to-Use API:** RESTful endpoints with OAuth authentication and a unified response structure (`code`, `status`, `message`, `data`, ...).
- **Multiple Format Support:** Upload files via `multipart/form-data` or base64 string.
- **Professional Error Handling:** Display safe messages to the user in production, while logging full details in development.
- **Comprehensive Documentation:** Detailed documentation files in Arabic covering all classes, examples, and endpoints (now translated).

---

### 6. Benefits and Added Value

- **For Developers:**
  - Ability to add file upload fields in any plugin simply by registering.
  - Unifying upload and permission logic reduces code duplication.
  - Temporary upload support simplifies multi‑step model creation.
  - Unified and securely protected API.

- **For End Users:**
  - Smooth and consistent file upload experience across the application.
  - Clear error messages that help correct inputs.

- **For the System as a Whole:**
  - Enhanced security via permission checks before every operation.
  - High flexibility in managing user types.
  - Extensible structure for adding future features (e.g., automatic compression, image optimization).

---

### 7. Future Development Plans

- **Support Additional File Types:** Add special handling for specific types (PDF, documents).
- **Image Processing Improvements:** Support automatic image compression, watermarks.
- **Graphical Administration Interface:** Develop an admin interface to monitor uploaded files and manage permissions.
- **Cloud Service Integration:** Support direct upload to cloud storage services (AWS S3, etc.).
- **Webhook Notifications:** Send notifications after successful file uploads.

---

### 8. Conclusion

The launch of the `Nano.FileUpload` plugin represents a significant advancement in how file upload operations are managed in NanoSoft applications. By providing the `FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, and `FileUploadController` classes, developers can add file upload fields to any plugin quickly and securely, with full control over permissions and user types. The plugin is designed to be extensible, allowing advanced features to be added in the future without affecting current stability.

With this phase completed, the plugin is ready for use in various NanoSoft projects, and we look forward to continued development based on user feedback and evolving requirements.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)

## 2026-04-03 - 2026-04-05

**Updates to the `Nano.FileUpload` Plugin – Versions 1.0.2 to 1.0.5**

### Summary of Updates

Since the first release of the plugin, we have continued to develop `Nano.FileUpload` based on user feedback and security requirements. Updates in versions 1.0.2 through 1.0.5 included significant improvements in the permission system, default settings for file types, temporary key security, caching, multilingual support, error logging, and blocking dangerous files.

---

## Version 1.0.2 – Security, Permission, and Default Settings Enhancements

### Version Objectives

- Provide global control over API operations (disable upload, delete, retrieve) via environment variables.
- Add default settings for file types (`image`, `audio`, `video`, `file`) to reduce repetition when registering models.
- Enhance temporary session key security by including `userType` and signing with HMAC-SHA256.
- Add validation of the temporary key before linking files.

### New Features

#### 1. Global Control of API Operations

We added global settings in `config.php` that allow disabling upload, delete, or retrieve operations at the entire API level, with control via environment variables:

```ini
NANO_FILE_UPLOAD_DISABLE_UPLOAD=false
NANO_FILE_UPLOAD_DISABLE_DELETE=false
NANO_FILE_UPLOAD_DISABLE_GET=false
```

We added helper methods in `FileUploadRegistry`:
- `isUploadEnabledGlobally()`
- `isDeleteEnabledGlobally()`
- `isGetEnabledGlobally()`

The `can()` method now checks these settings before any operation.

#### 2. Default Settings for File Types

We added a `defaults` section in `config.php` with settings for each file type:
- `image`: max size, allowed types, use caption, thumbnail options.
- `audio`, `video`, `file` with similar settings.

When registering a field, if the developer does not specify `max_filesize`, `allowed_types`, or `use_caption`, the default settings are automatically applied based on the field type.

#### 3. Enhanced Temporary Session Key Security

- Modified `generateTempSessionKey()` to include `userType` and `timestamp` in the signed data.
- Use `hash_hmac('sha256', $data, $secret)` to sign the key, preventing tampering or guessing.
- Added `validateTempSessionKey()` to verify the key's validity and expiration (default expiry of one hour).
- In `attachTempFiles()`, we check:
  - Key validity (signature, expiration).
  - Matching `modelClass` and `field`.
  - Matching `userId` and `userType` with the current user.

This prevents any user from accessing temporary files belonging to another user.

#### 4. Updated `FileUploadRegistry::can()` to Include Global Checks

The `can()` method now first checks the global settings for each operation (`add`, `delete`, `view`) before checking user type and specific permissions.

#### 5. Additional Helper Methods

- `getDefaultConfigForType($type)`: Retrieve default settings for a given type.
- `applyDefaultsToFieldConfig()`: Apply default settings to a field configuration.

### Benefits

- Ability to temporarily disable API operations without modifying code (e.g., for maintenance).
- Reduce repetitive code when registering models.
- Higher security for temporary files, preventing unauthorized access.

---

## Version 1.0.3 – Performance and Flexibility Improvements for `FileUploadRegistry`

### Version Objectives

- Refactor `FileUploadRegistry` to improve performance and scalability.
- Support caching for field configuration and constraints query results.
- Provide flexible registration via `registerDefinition()` and `registerCallback()`.
- Normalize model and field definitions using specialized methods.

### New Features

#### 1. Separation of Raw Definitions from Built Objects

- `$rawDefinitions`: Store definitions as they come from plugins.
- `$builtModels`: Store built objects after construction (Lazy Loading).
- Load definitions from plugins only once (`$loaded` flag).

#### 2. Cache Support

We added a `cache` section in `config.php`:

```php
'registry' => [
    'cache' => [
        'enabled' => env('NANO_FILE_UPLOAD_REGISTRY_CACHE_ENABLED', true),
        'ttl' => env('NANO_FILE_UPLOAD_REGISTRY_CACHE_TTL', 3600),
    ],
],
```

And methods:
- `getFieldConfig()` and `getFieldConstraints()` use `Cache::get()` and `Cache::put()`.
- `clearModelCache()` to clear cache when updating a model definition.
- `setCacheEnabled()`, `setCacheTtl()` for programmatic control.

#### 3. Flexible Registration Methods

- `registerDefinition($modelClass, array $definition)`: Register a raw definition.
- `registerDefinitions(array $definitions)`: Register multiple definitions.
- `registerCallback(callable $callback)`: Register a callback to be executed at load time (e.g., like `ReportsManager`).
- The `registerModel()` method remains for backward compatibility.

#### 4. Definition Normalization

- `normalizeModelDefinition()`: Merge default settings.
- `normalizeFieldDefinition()`: Apply default settings based on field type.
- `getDefaultConfigForType()`: Fetch settings from the configuration file.

#### 5. Improved Query Methods

- `isModelRegistered()` automatically calls `getRegisteredModels()` to load definitions.
- `getModelConfig()` uses `buildModel()` to build the model when needed.

### Benefits

- Better performance thanks to caching (avoid re‑processing definitions on every request).
- Greater flexibility in registering models (via raw definitions or callbacks).
- Cleaner, more maintainable code structure.

---

## Version 1.0.4 – Enhanced Security, Logging, and Exceptions

### Version Objectives

- Permanently block dangerous files (PHP, JS, HTML, EXE, etc.) even if they are in `allowed_types`.
- Create a separate log channel for recording failed upload attempts.
- Create a custom exception class `FileUploadException` with unique error codes.
- Improve error messages and make them translatable (foundation for localization).

### New Features

#### 1. Blacklist for Dangerous Files

We added a constant `BLACKLISTED_EXTENSIONS` in `FileUploadService` containing dangerous extensions such as:
`php, js, html, exe, bat, sh, dll, ...`

And a method `isBlacklistedExtension()` that checks the extension before any other validation. If the extension is blacklisted, the file is immediately rejected with an appropriate message.

#### 2. Logging Failed Upload Attempts

- Added a new log channel `fileupload` in `config/logging.php` (configured in `Plugin::boot()`).
- The `logFailedAttempt()` method logs the following details:
  - User (ID, type, IP, User Agent)
  - Model and field
  - Reason for failure (size, type, blacklist, permission)
  - File information (name, size, type, extension)
- This method is called whenever an error occurs in `validateFile()`.

#### 3. `FileUploadException` Class

A custom exception class inheriting from `Exception` was created, providing:
- Numeric error codes (1000-1999 general, 2000-2999 for files, 3000-3999 for permissions, 4000-4999 for temporary keys).
- Generation of a unique textual error code (e.g., `FILE_UPLOAD_FILE_SIZE_EXCEEDED`).
- Ability to add context to the error (e.g., allowed size, uploaded extension).

#### 4. Updated `FileUploadService::validateFile()` to include:

- Blacklist check first.
- Size and allowed type checks (as before).
- Throwing `FileUploadException` instead of `ApplicationException`.
- Calling `logFailedAttempt()` before throwing the exception.

#### 5. Updated `FileUploadController`:

- The `handleException()` method recognises `FileUploadException` and extracts the `errorCode`.
- The `errorResponse()` method accepts an `$errorCode` parameter and adds it to the response.

### Benefits

- Higher security by blocking dangerous files even if `allowed_types` is misconfigured.
- Ability to track attacks or issues through a separate log.
- Unified error handling with unique codes to facilitate frontend handling.

---

## Version 1.0.5 – Full Multilingual Support

### Version Objectives

- Make all API messages (success, error) translatable into Arabic and English.
- Add translation keys for every operation (`upload`, `uploadMultiple`, `delete`, `getFiles`).
- Update the controller and service to use `trans()` instead of hard‑coded strings.

### New Features

#### 1. Added Translation Keys to `lang.php`

We added a new section `public.helpers.upload` containing keys such as:
- `msg_upload_success`
- `msg_upload_multiple_success`
- `msg_delete_success`
- `msg_get_success`
- `msg_permission_denied`
- `msg_validation_failed`
- `msg_upload_disabled`, `msg_delete_disabled`, `msg_get_disabled`
- and other common error messages.

We also added keys in the `errors` section for detailed messages such as `file_size_exceeded`, `file_type_not_allowed`, `file_type_blacklisted`.

#### 2. Updated `FileUploadController`

- The `successResponse()` and `errorResponse()` methods use `trans()` with default values.
- The `upload()`, `uploadMultiple()`, `delete()`, `getFiles()` methods use `trans()` for every message (success or failure).
- The `getSafeErrorMessage()` method uses `trans()` for generic messages.
- In `uploadMultiple()`, `success_count` and `total` are passed to the translation function.

#### 3. Updated `FileUploadService`

- Replaced all hard‑coded `ApplicationException` messages with `trans()` using the appropriate keys.
- In `validateFile()`, `trans()` is used in `FileUploadException` messages, passing parameters (e.g., `max_size`, `actual_size`, `extension`, `allowed`).

#### 4. Support for Arabic and English

- Translation files are located in `lang/ar/lang.php` and `lang/en/lang.php`.
- Users can switch language via `app.locale`.

### Benefits

- Improved user experience for Arabic and English speakers.
- Easy addition of new languages in the future.
- All messages are centralised in translation files, facilitating maintenance and modification.

---

## Version Summary (1.0.2 – 1.0.5)

| Version | Key Features |
|---------|---------------|
| 1.0.2 | Global API control, default file type settings, enhanced temporary key security. |
| 1.0.3 | Refactored `FileUploadRegistry`, caching support, flexible registration via definitions and callbacks. |
| 1.0.4 | Blacklist for dangerous files, logging of failed upload attempts, custom exception class. |
| 1.0.5 | Full multilingual support (Arabic/English) for all API messages. |

---

## Conclusion

Thanks to these updates, the `Nano.FileUpload` plugin has become more secure, performant, and flexible. It now provides:
- Fine‑grained permission control at multiple levels.
- Smart default settings that reduce code duplication.
- High security for temporary files and protection against dangerous files.
- Comprehensive error logging to monitor malicious attempts.
- A unified API with translatable messages.

These improvements make the plugin ready for use in the largest projects, with the ability to scale in the future.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)


## 2026-04-05 - 2026-04-07

**Updates to `Nano.FileUpload` Add-on – Versions 1.0.6 and 1.0.7**

### Summary of Updates

After releasing version 1.0.5, which focused on multi-language support, we continued developing the add-on by adding advanced features aimed at increasing flexibility, scalability, and improving file management at the database level. Versions 1.0.6 and 1.0.7 included:

- Support for multi-storage via different disks (S3, FTP, Local).
- Automatic image transformation (automatic resizing and watermarking) with global control via settings.
- Event hooks before and after upload and delete operations.
- Optional WebSocket notifications.
- Adding new columns to the `system_files` table: `disk`, `hash`, `meta`, `expires_at`.
- Automatic population of the new fields (SHA256 of content, image dimensions, expiration date for temporary files).
- General performance and security improvements.

---

## Version 1.0.6 – Multi-Storage, Automatic Image Transformation, Event Hooks, and Database Enhancements

### Version Goals

- Enable uploading files to multiple storage services (AWS S3, FTP, Local) by specifying the storage disk at the field level.
- Add the ability to automatically transform images (resizing and watermarking) during upload.
- Provide event hooks to extend behavior without modifying the core code.
- Support WebSocket notifications for real-time updates in frontends.
- Add new columns to the `system_files` table to support advanced features (storage, deduplication, metadata, expiration).

### New Features

#### 1. Multi-Storage Support

We added the ability to specify a custom storage disk per field via the `storage_disk` setting in the field definition. This overrides the default disk set in the system configuration.

- **Added `disk` column** to the `system_files` table to store the name of the disk used.
- **Modified `System\Models\File` model** (via `Plugin::extendSystemFile()`) to support the `disk` property and override the `getDisk()` method to use the custom disk.
- **Added `getStorageDiskForField()` method** in `FileUploadService` to retrieve the disk name from field settings and check its existence.
- During upload, `$file->disk = $diskName` is set and then the file is saved on the specified disk.

#### 2. Automatic Image Transformation (Auto-resize & Auto-watermark)

We added new settings in the field definition:
- `auto_resize` (bool): enable automatic image resizing.
- `resize_options` (array): resize options (width, height, mode).
- `auto_watermark` (bool): enable automatic watermarking.
- `watermark_options` (array): watermark options (position, resize percentage).

We added the `applyAutoProcessing()` method in `FileUploadService` which:
- Checks if the file is an image.
- Uses `October\Rain\Resize\Resizer` to resize the image according to the options.
- Uses the `Nano2.Watermark` add-on (if installed and enabled) to add the watermark.

#### 3. Event Hooks

We added the following events to easily extend behavior:

| Event | Call Location | Parameters |
|-------|---------------|------------|
| `nano.fileupload.beforeUpload` | Before starting upload processing | `$modelClass, $field, $fileData, &$options` |
| `nano.fileupload.afterUpload` | After successful upload and before returning the result | `$file, $modelClass, $field, $options` |
| `nano.fileupload.beforeDelete` | Before deleting the file | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterDelete` | After deleting the file | `$fileId, $modelClass, $field` |

Developers can listen to these events to add custom logic (e.g., sending notifications, additional logging, modifying data).

#### 4. WebSocket Support for Instant Notifications

We added `websocket` settings in `config.php`:
- `enabled`: enable/disable notifications.
- `channel`: the channel name used.

We added the `notifyWebSocket()` method in `FileUploadService` which dispatches the `nano.fileupload.websocket.notify` event after a successful upload or deletion. Developers can use a WebSocket library (e.g., Laravel WebSockets or Pusher) to catch this event and send notifications.

#### 5. Adding New Columns to the `system_files` Table

We created a migration `add_columns_to_system_files.php` to add the following columns:

| Column | Type | Purpose |
|--------|------|---------|
| `disk` | `string, nullable` | Store the name of the custom storage disk (e.g., `s3`, `ftp`). |
| `hash` | `string, nullable, index` | Store SHA256 of the content to prevent duplicates. |
| `meta` | `text, nullable` | Store additional data as JSON (image dimensions, audio duration, etc.). |
| `expires_at` | `dateTime, nullable` | Expiration date for temporary files to be automatically cleaned up. |

### Benefits

- Ability to store files on multiple cloud services as needed.
- Improved user experience by automatically resizing images and adding watermarks without manual intervention.
- Easy system extension via events without modifying core code.
- Instant notifications to users when upload or deletion completes.
- Better management of duplicate files, metadata, and temporary files.

---

## Version 1.0.7 – Completing Automatic Population of `hash`, `meta`, `expires_at` Fields and Adding Global Control over Transformations

### Version Goals

- Complete the implementation of automatically populating the new fields (`hash`, `meta`, `expires_at`) during upload.
- Add global settings to enable/disable automatic transformations (`auto_resize`, `auto_watermark`) at the entire API level.
- Improve performance and security by calculating hashes, storing image dimensions, and setting expiration for temporary files.

### New Features

#### 1. Automatic Population of `hash` (SHA256)

We added the following inside the `upload()` method after obtaining the file object:
```php
if (!$file->hash) {
    $content = file_get_contents($file->getLocalPath());
    $file->hash = hash('sha256', $content);
    $is_changed = true;
}
```
This ensures a unique hash of the content is calculated, helping to prevent storing duplicate copies (the `hash` can be checked before saving).

#### 2. Automatic Population of `meta` with Image Dimensions

We added:
```php
if ($file->isImage()) {
    $dimensions = getimagesize($file->getLocalPath());
    $file->meta = array_merge($file->meta ?: [], [
        'width'  => $dimensions[0],
        'height' => $dimensions[1],
        'mime'   => $dimensions['mime'],
    ]);
    $is_changed = true;
}
```
Dimensions and MIME type are stored in the `meta` field as JSON, making it easy to retrieve them later without needing to read the file again.

#### 3. Setting `expires_at` for Temporary Files

When a `temp_session_key` is used (i.e., the file is not yet linked to a model), we automatically set an expiration date:
```php
if ($tempSessionKey) {
    if (!$file->expires_at && (!$file->attachment_type || !$file->attachment_id)) {
        $file->expires_at = Carbon::now()->addHours(24);
        $is_changed = true;
    }
}
```
A scheduled task (cron) can be run to automatically delete expired and unlinked files.

#### 4. Adding Global Control over Automatic Transformations

We added two settings in the `general` section of `config.php`:
```php
'disable_auto_resize'    => env('NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE', true),
'disable_auto_watermark' => env('NANO_FILE_UPLOAD_DISABLE_AUTO_WATERMARK', true),
```
And two methods in `FileUploadRegistry`:
- `isAutoResizeEnabledGlobally()`
- `isAutoWatermarkEnabledGlobally()`

Then we modified the `applyAutoProcessing()` method to respect these settings:
```php
if ($this->registry->isAutoResizeEnabledGlobally() && $procOptions['auto_resize']) { ... }
if ($this->registry->isAutoWatermarkEnabledGlobally() && $procOptions['auto_watermark'] && ...) { ... }
```
This allows the administrator to temporarily disable all resizing or watermarking operations via environment variables without modifying field definitions.

#### 5. Other Improvements

- Use `Carbon::now()` instead of `now()` to ensure compatibility.
- Fixed variable name `$is_changed` (instead of `$is_chage`).
- Ensure the file is saved again when any of the fields (`hash`, `meta`, `expires_at`) are changed.

### Benefits

- Better management of duplicate files via `hash` checking.
- Store useful metadata (image dimensions) for use in frontends.
- Automatic cleanup of temporary files via `expires_at`.
- Centralized control over enabling/disabling automatic transformations.

---

## Version Summary (1.0.6 and 1.0.7)

| Version | Key Features |
|---------|---------------|
| 1.0.6 | Multi-storage support (disk), automatic image transformation, event hooks, WebSocket, adding new columns to database. |
| 1.0.7 | Automatic population of `hash`, `meta`, `expires_at`, global control over automatic transformations, performance and security improvements. |

---

## Upgrade Requirements

1. **Run the migration**:
   ```bash
   php artisan plugin:refresh Nano.FileUpload
   ```
   Or execute the new migration `add_columns_to_system_files.php`.

2. **Add environment variables** (optional) in your `.env` file:
   ```ini
   # Disable automatic transformations (default values: true)
   NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE=true
   NANO_FILE_UPLOAD_DISABLE_AUTO_WATERMARK=true

   # WebSocket settings
   NANO_FILE_UPLOAD_WEBSOCKET_ENABLED=false
   NANO_FILE_UPLOAD_WEBSOCKET_CHANNEL=file-uploads

   # Multi-storage settings (at field level, not global)
   # (set directly in field definitions)
   ```

3. **Update field definitions** to add `storage_disk`, `auto_resize`, `auto_watermark` as needed.

4. **Listen to events** to extend behavior:
   ```php
   Event::listen('nano.fileupload.afterUpload', function ($file, $modelClass, $field, $options) {
       // Send WebSocket notification
   });
   ```

---

## Conclusion

Thanks to versions 1.0.6 and 1.0.7, the `Nano.FileUpload` add-on is now more powerful and flexible than ever. It can now:

- Handle multiple storage disks (S3, FTP, Local).
- Automatically transform images (resizing, watermarking).
- Dispatch custom events to extend behavior.
- Send instant notifications via WebSocket.
- Store rich metadata (image dimensions, hashes).
- Automatically clean up temporary files.

These features make the add-on ready for use in large, complex projects that demand high performance and advanced security.

---

**Reference Documentation**:
- [General Add-on Documentation](./docs/FileUpload/Docs-FileUpload.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class.md)
- [Advanced Examples for `FileUploadRegistry` Class](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class.md)
- [Advanced Examples for `FileUploadService` Class](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class.md)
- [Advanced Examples for `FileUploadUserManager` Class](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation.md)


## 2026-03-11 – 2026-04-09

### Adding Visual Query Builder System and Advanced Condition Processor  
Within the `Nano2.QueryBuilder` software module

---

### 1. Introduction

A **Visual Query Builder** system has been developed to provide an interactive, easy-to-use interface that allows end users and developers to create complex query conditions without writing SQL or manual programming logic. The system relies on the `jQuery QueryBuilder` library on the frontend, and provides an integrated backend within NanoSoft software through a custom `FormWidget` and a helper class to convert conditions into executable queries on Data Models.

This update aims to enable use cases such as:
- Building dynamic reports with multi-level conditions (AND/OR) and various input types (text, numbers, dates, dropdowns, checkboxes, etc.).
- Applying conditions directly to a `Model` or `Builder` via the `QueryBuilderParserHelper` class.
- Reusing filter and operation definitions via YAML configuration or model methods.
- Supporting plugins such as drag-and-drop sorting, filter descriptions, custom icons, and more.

---

### 2. Developed Components

#### 2.1 `QueryBuilderParserHelper` – Condition Processing and Query Conversion Class  
- **Path**: `Nano2\QueryBuilder\Classes\QueryBuilderParserHelper`
- **Function**: A helper class that parses the condition object (rules) coming from the FormWidget, applies them to a Builder query, supporting macros, nested tables, and multiple operators.

**Key Methods**:

| Method | Description |
|--------|-------------|
| `parse($builder, $rules, $rows, $is_exception)` | Main method: receives `$builder` (model, model class name, or Builder object) and `$rules` array (usually from QueryBuilder's `getRules()`), returns a `Builder` with conditions applied. |
| `setFields($rules)` | Extracts field names used in the rules to pass to the `timgws\QueryBuilderParser` library (necessary to specify allowed columns). |
| `decodeJSON($json)` | Converts a JSON string to an object to validate rules. |
| `isNested($rule)` | Checks if a rule contains a sub-group of rules (for handling nested conditions). |
| `getQueryOrModel($obj, $is_model, $is_exception)` | Accepts an object that may be a `Model`, class name, or `Builder`, and returns either a `Builder` object or the model itself as needed. |
| `toSql($builder, $expand)` | (Taken from LibreNMS) Converts conditions to an SQL string for debugging or display. |

**Workflow**:
1. The `$rules` object (usually an array or JSON) is converted to a stdClass object using `decodeJSON`.
2. Fields used are extracted via `setFields`.
3. An instance of the `timgws\QueryBuilderParser` library is created with the allowed fields.
4. `parse($json, $query)` is called to apply all `where`, `orWhere`, and logical groups.
5. The modified `Builder` object is returned.

**Usage Example**:
```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

$rules = [
    'condition' => 'AND',
    'rules' => [
        ['id' => 'price', 'operator' => 'greater', 'value' => 100],
        ['id' => 'category', 'operator' => 'equal', 'value' => 5]
    ]
];
$query = Product::query();
$finalQuery = QueryBuilderParserHelper::parse($query, $rules);
$products = $finalQuery->get();
```

---

#### 2.2 `QueryBuilder` – FormWidget for Building Queries  
- **Path**: `Nano2\QueryBuilder\FormWidgets\QueryBuilder`
- **Function**: Provides a custom field in NanoSoft forms that displays the jQuery QueryBuilder interface to the user, and stores the rules as JSON in the database.

**Main Configurable Properties (via YAML or class properties)**:

| Property | Description | Default Value |
|----------|-------------|---------------|
| `filters` | Definition of available filters (Filter objects). Can be an array or a string referring to a model method. | `[]` |
| `rules` | Default rules displayed when the interface loads. | `[]` |
| `sort_filters` | Sort filters alphabetically by translation. | `false` |
| `allow_groups` | Maximum number of nested groups (`-1` = unlimited, `0` = no groups). | `-1` |
| `allow_empty` | Allow empty rules. | `false` |
| `conditions` | List of allowed logical connectors (`AND`, `OR`). | `['AND', 'OR']` |
| `default_condition` | Default connector for the root group. | `'AND'` |
| `inputs_separator` | Separator between multiple input fields (e.g., BETWEEN). | `' , '` |
| `display_empty_filter` | Show an empty option in the filters list. | `true` |
| `select_placeholder` | Text shown when no filter is selected. | `'------'` |
| `plugins` | List of enabled plugins (e.g., `sortable`, `filter-description`). | `[]` |
| `isShowParseSqlBtn` | Show a button to translate rules to SQL. | `false` |
| `isShowResetBtn` / `isShowSetBtn` / `isShowGetBtn` | Reset, load rules, and get rules buttons. | `true` |

**Workflow in the Form**:
1. In the `init()` method, settings are loaded from YAML or properties.
2. In `prepareVars()`, the `filters` array is prepared (supporting translation, calling model methods to get options).
3. `$value` (stored JSON rules) is converted to a suitable object for JavaScript.
4. Assets (CSS/JS) for the library and plugins are added.
5. When the partial template (`_query_builder_basic.htm`) is rendered, jQuery QueryBuilder is initialized using the options and rules.
6. When the user changes the rules, a hidden input field (`input[type=hidden]`) is updated with the JSON value, and upon form save, the value is stored in the database.

**Support for Dynamic Option Sources**:
The `values` property in a filter can be a string referring to:
- A method name in the current model (e.g., `getCategoryOptions`).
- A static method in the format `ClassName::method`.
- It is called during filter preparation and passed current values and context.

**Example YAML Configuration**:
```yaml
query_builder:
    type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
    filters:
        category:
            id: category
            label: Category
            type: integer
            input: select
            values:
                1: Books
                2: Movies
                3: Music
            operators: ['equal', 'not_equal', 'in']
        price:
            id: price
            label: Price
            type: double
            validation:
                min: 0
                step: 0.01
    rules: 
        condition: AND
        rules: []
    allow_groups: 3
    sort_filters: true
    plugins:
        sortable: null
        filter-description: { mode: 'popover' }
```

---

### 3. Advanced Filter Options

Each filter follows the same structure defined in the jQuery QueryBuilder library with NanoSoft additions:

| Property | Description |
|----------|-------------|
| `id` | Unique filter identifier. |
| `label` | Display text. Supports translation as array or string. |
| `type` | Data type: `string`, `integer`, `double`, `date`, `time`, `datetime`, `boolean`. |
| `input` | Input element type: `text`, `number`, `textarea`, `radio`, `checkbox`, `select`. |
| `values` | Array of options (for `select`, `radio`, `checkbox`). Can be a function or static array. |
| `operators` | List of allowed operators for this filter (e.g., `equal`, `contains`, `between`). |
| `default_operator` | Default operator. |
| `validation` | Validation rules: `min`, `max`, `step`, `format` (regex or date format), `allow_empty_value`. |
| `placeholder` | Placeholder text inside the input field. |
| `plugin` | jQuery plugin name (e.g., `datepicker`, `selectize`). |
| `plugin_config` | Plugin-specific settings. |

**Two ways to provide filters**:
1. **Directly in YAML file** (as in the example above).
2. **Via a model method**:
   - General method: `getQueryBuilderFiltersOptions($columnName)`
   - Field-specific method: `get{FieldName}QueryBuilderFiltersOptions($fieldName, $columnName)`

**Example model method**:
```php
public function getQueryBuilderFiltersOptions($columnName)
{
    if ($columnName == 'filters') {
        return [
            ['id' => 'status', 'label' => 'Status', 'type' => 'integer', 'input' => 'select', 
             'values' => [1 => 'Active', 0 => 'Inactive']],
            // ... 
        ];
    }
    if ($columnName == 'rules') {
        return ['condition' => 'AND', 'rules' => []];
    }
    return [];
}
```

---

### 4. Plugin Support in the FormWidget

A set of plugins provided by the jQuery QueryBuilder library can be enabled with custom settings:

| Plugin Name | Description | Options |
|-------------|-------------|---------|
| `sortable` | Enable drag-and-drop to reorder rules and groups. | `icon`, `inherit_no_sortable` |
| `filter-description` | Show a descriptive tooltip for the filter (Inline, Popover, or Bootbox). | `mode`, `icon` |
| `bt-tooltip-errors` | Display validation errors as Bootstrap tooltips. | `placement` |
| `bt-checkbox` | Style `checkbox` and `radio` elements using Bootstrap. | `color` |
| `unique-filter` | Prevent using the same filter more than once (globally or within a group). | – |
| `not-group` | Add a "NOT" option to invert group logic. | `icon_checked`, `icon_unchecked` |
| `invert` | Button to invert conditions (convert `=` to `!=`, `AND` to `OR`...). | `recursive`, `invert_rules` |

**Activation in YAML**:
```yaml
plugins:
    sortable: { icon: 'bi-sort-down' }
    filter-description: { mode: 'bootbox', icon: 'bi-info-circle' }
    bt-tooltip-errors: { placement: 'top' }
```

---

### 5. Complete Example Using FormWidget and ParserHelper

#### 5.1 Field Definition in a Model (fields.yaml)
```yaml
fields:
    query_conditions:
        label: Report Conditions
        type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
        filters:
            name: { id: name, label: Name, type: string }
            created_at: { id: created_at, label: Created At, type: date, input: datepicker, plugin: datepicker }
        allow_groups: 2
        sort_filters: true
        plugins: { 'sortable': null, 'filter-description': { mode: 'inline' } }
```

#### 5.2 Saving Rules from the Controller
When the form is saved, the value will be stored as JSON in the column associated with the field.

#### 5.3 Using Rules in a Query
```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

$reportRules = $model->query_conditions; // JSON string or array
if (!empty($reportRules)) {
    $query = Product::query();
    $query = QueryBuilderParserHelper::parse($query, $reportRules);
    $results = $query->get();
}
```

---

### 6. Added Value

- **For developers**: Significant reduction in development time for reports and search screens; no need to write complex `where` conditions manually. Unified way to define filters and reuse them across projects in the NanoSoft environment.
- **For end users**: Intuitive and easy interface to build custom queries without needing SQL or backend data structure knowledge.
- **For the system**: Separation of query building logic from the user interface, with extensibility via new plugins and seamless integration with the NanoSoft system (translation, permissions, dynamic model methods).

---

### 7. Conclusion

The `Nano2.QueryBuilder` update is a qualitative addition to the NanoSoft platform, providing developers with a powerful and flexible tool to create complex query conditions visually and securely. Thanks to the integration between the powerful `jQuery QueryBuilder` library on one hand, and the `QueryBuilderParserHelper` class that processes conditions and applies them to a `Builder` on the other, it is now possible to build advanced reporting and filtering systems quickly and with high quality. The modular design of the `FormWidget` also allows the component to be reused in any form with full support for customizing filters and plugins.


---

## Reference Documentation

- [`QueryBuilder` FormWidget Documentation](./docs/querybuilder/Docs-FormWidgets-QueryBuilder-en.md)
- [`QueryBuilderParserHelper` Class Documentation](./docs/querybuilder/Docs-QueryBuilderParserHelper-en.md)

## 2026-04-07 – 2026-04-10

**`Nano.FileUpload` Plugin Updates – Versions 1.0.8 to 1.2.0**

### Update Summary

After previous releases that focused on multiple storage support, automatic image transformation, and event hooks, we continued developing the add-on to enhance security, flexibility, and add advanced file management features. Updates in versions 1.0.8 through 1.2.0 included:

- **Addition of a comprehensive testing suite** to verify all add-on classes.
- **Addition of `session_key` column** to the `system_files` table to support temporary uploads and deferred binding.
- **Support for Edit operation** to replace files in `attachOne` relations with custom permissions.
- **Addition of field-level update functions** in `FileUploadRegistry`.
- **Performance and reliability improvements** for upload and delete operations using transactions and new events.
- **Restructuring of the test class** into `FileUploadPlusTest` with unified output format and advanced exception handling.

---

### Version 1.0.8 – Comprehensive Testing System and Improved Delete Security

#### Release Goals

- Create an integrated test class (`FileUploadTest`) covering all add-on classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`).
- Improve security of the `deleteFile` function to automatically extract `modelClass` and `field` from the file record if not provided.
- Integrate tests into `Plugin.php` so they can be run via a `test_fileupload` parameter in development environment.

#### New Features

##### 1. `FileUploadTest` Class for Comprehensive Testing

A test class was created located at `Nano\FileUpload\Tests\FileUploadTest` containing functions to test:

- **FileUploadRegistry**: model registration, getting field config, model existence check, defaults, processing options, model update, global API operation controls, and caching mechanism.
- **FileUploadUserManager**: getting user, determining user type, permission checking, customizing user resolver, and customizing permission checker function.
- **FileUploadService**: generating temporary keys, validating them, direct file upload, base64 upload, multiple upload, size/type/blacklist validation, temporary upload, attaching temporary files, retrieving files, deleting files, automatic resizing, watermarking, custom storage disk, `hash` and `meta` calculation, temporary file expiry, events, WebSocket, and globally disabling operations.

The test uses an isolated environment (`Storage::fake('local')`) and a dummy model (`TestModel`) to ensure no effect on real data.

##### 2. Improved Security of `deleteFile` Function

The `deleteFile` function now handles three cases for permission checking:

- **Case 1:** `modelClass` and `field` are provided – permission is checked using them.
- **Case 2:** File is attached to a saved model (`attachment_type` and `attachment_id`) – `modelClass` and `field` are extracted from the file record and permission is checked.
- **Case 3:** File is temporary (`session_key`) – the key is validated and that the current user is the one who created the temporary file.

This ensures no user can delete a file they do not have permission for, even if they do not explicitly pass `modelClass` and `field`.

##### 3. Integration of Tests into `Plugin.php`

We added a `testFileUpload()` function in `Plugin.php` that is called when the `is_testFileUpload=true` parameter is present in the request. It creates an instance of `FileUploadTest`, runs all tests, and displays results. This allows developers to easily run tests via browser.

#### Benefits

- Ensure code quality through automated testing of all core functions.
- Early detection of bugs during development.
- Living documentation of class behavior via tests.
- Increased confidence in add-on stability when adding new features.

---

### Version 1.0.9 – Support for `session_key` in Files Table

#### Release Goals

- Add a `session_key` column to the `system_files` table to fully support deferred binding.
- Enable filtering of temporary files belonging to a specific user session.

#### New Features

##### 1. Migration `add_session_key_to_system_files`

A migration file was created that adds the `session_key` column of type `string` and `nullable` with an index (after the `field` column). This column is used to store the temporary session key when a file is uploaded without being attached to a saved model.

##### 2. Update `FileUploadService::upload`

During upload, if a temporary key (`tempSessionKey`) is used, `$file->session_key = $tempSessionKey` is set before saving the file. This ensures temporary files can later be found using this key.

##### 3. Update `FileUploadService::attachTempFiles`

The `attachTempFiles` function now relies on the `session_key` column to find temporary files to attach to the saved model. After attachment, `session_key` is cleared and `attachment_id` and `attachment_type` are set.

##### 4. Update `FileUploadService::deleteFile`

In the case of a temporary file, the `session_key` is checked to ensure the current user is the one who created the file before allowing deletion.

#### Benefits

- Full support for temporary upload and deferred binding.
- Ability to clean up expired temporary files via scheduled task using `expires_at` and `session_key`.
- Higher security: cannot access temporary files belonging to another user.

---

### Version 1.1.0 – Support for Edit Operation and File Replacement

#### Release Goals

- Enable replacing an existing file in an `attachOne` relation by uploading a new file (edit operation).
- Add a global `disable_edit` setting to enable/disable file replacement at the API level.
- Add `updateFieldConfig` function in `FileUploadRegistry` to update a specific field's settings with normalization and cache clearing.
- Improve `updateModelConfig` to normalize the entire model after merging.
- Move the `Base64` class from `Nano.API` into the add-on to reduce dependencies.
- Add new events `beforeEditDelete` and `afterEditDelete` for the edit lifecycle.
- Improve `attachTempFiles` function to return a unified response.

#### New Features

##### 1. Determine Upload Operation (add/edit)

In `FileUploadService::upload`, it checks if the model exists and has an `attachOne` relation for the field. If a file is already attached, `$operation = 'edit'` is set; otherwise `'add'`.

##### 2. Permission Check for `edit` Operation

We added in `FileUploadRegistry`:

- `isEditEnabledGlobally()` function that checks `disable_edit` in global settings.
- `can()` function now supports the `'edit'` operation and checks the global setting and field/model permissions.
- In model and field default definitions, we added the `'edit' => null` key in the `permissions` array.

##### 3. Use Transaction in `upload` Operation

The upload operation is now wrapped in `Db::transaction` to ensure integrity:

- The new file is uploaded and saved first.
- If the operation is `edit` and the upload succeeds, the old file is deleted.
- Then the new file is attached to the model relation.
- If any part fails, all changes are rolled back (including deletion of the old file).

This prevents loss of the old file if the new file upload fails.

##### 4. New Events for Edit Operation

- `nano.fileupload.beforeEditDelete`: Dispatched before deleting the old file (when `operation === 'edit'`).
- `nano.fileupload.afterEditDelete`: Dispatched after deleting the old file.

This allows developers to execute custom logic (e.g., send notification, log, or backup) before and after replacing the file.

##### 5. `updateFieldConfig` Function in `FileUploadRegistry`

This function provides an updated way to modify settings of a specific field without re-registering the entire model. It:

- Checks existence of model and field.
- Merges new settings with existing using `array_replace_recursive`.
- Normalizes the field via `normalizeFieldDefinition`.
- Updates `rawDefinitions` and resets built model (`builtModels`).
- Clears cache for this specific field via `clearFieldCache`.

##### 6. Improved `updateModelConfig`

The function now re-normalizes the entire model after merging new settings, ensuring all default fields (e.g., `is_public`, `thumb_options`) are complete.

##### 7. Move `Base64` Class into the Plugin

The `Base64` class was moved from `Nano\API\Classes\Base64` to `Nano\FileUpload\Classes\Base64` with advanced functions for handling file sizes (`normalizeUploadFileSize`, `getUploadMaxFilesize`, `checkUploadMaxFilesize`). This removes the dependency on `Nano.API` and makes the add-on independent.

##### 8. Improved `attachTempFiles`

The function now returns a unified array (like `upload`) containing `code`, `status`, `message`, `data`, `process_data`, `error`, `debug`, and accepts the `skip_permission_check` option to bypass permission checks in tests. It also adds an `afterAttach` event and WebSocket notification.

#### Benefits

- Ability to safely replace files in `attachOne` relations (using transactions).
- Fine-grained control over edit permission via global setting or custom permissions.
- More flexibility in updating field settings without re-registering the model.
- Reduced dependencies by moving `Base64` into the add-on.
- Unified output for `attachTempFiles` with other service functions.

---

### Version 1.2.0 – Testing System Overhaul

#### Release Goals

- Completely rewrite the test class into `FileUploadPlusTest` with a unified output structure.
- Unify each test's output to include: `code`, `status`, `test_code`, `name`, `description`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`.
- Add advanced exception handling so that one test failure does not stop the execution of other tests.
- Use transactions (`Db::beginTransaction()`, `Db::rollBack()`) in all tests that modify the database.
- Remove any dependency on external testing packages (e.g., `assert`).
- Add new tests for the edit operation (`testEditReplaceFile`, `testEditWithoutPermission`, `testEditWithGlobalDisable`, `testEditTriggersEvents`, `testEditKeepsOldFileOnFailure`).

#### New Features

##### 1. Unifying Test Outputs

We added a `recordResult(array $result)` function that accepts an array with the required structure and merges it with default values. This ensures all test results are consistent and easy to process automatically.

Example of a successful test output:
```json
{
    "code": 200,
    "status": true,
    "test_code": "UPLOAD_DIRECT_001",
    "name": "Direct file upload (UploadedFile) with temporary key",
    "description": "Checks uploading an image file using UploadedFile object...",
    "message": "File uploaded successfully",
    "error": null,
    "errors": [],
    "data": { ... },
    "input_data": { ... },
    "process_data": { ... },
    "debug": null
}
```

##### 2. Exception Handling

Each test is wrapped in `try-catch`. In case of an exception, `status` is set to `false`, `code` to `500`, the error message is stored in `error`, and debug details in `debug` (if `app.debug` is enabled). This ensures other tests continue to run.

##### 3. Using Transactions in Tests

All tests that make changes to the database (upload, delete, update, attach) use `Db::beginTransaction()` and `Db::rollBack()` in a `finally` block. This ensures no leftover data after tests finish.

##### 4. Adding Edit Operation Tests

We added 5 new tests to cover file replacement scenarios:

- `testEditReplaceFile`: Test successful replacement of a file in an `attachOne` relation.
- `testEditWithoutPermission`: Test rejection of replacement when `edit` permission is missing.
- `testEditWithGlobalDisable`: Test rejection of replacement when editing is globally disabled (`disable_edit = true`).
- `testEditTriggersEvents`: Test that `beforeEditDelete` and `afterEditDelete` events are dispatched.
- `testEditKeepsOldFileOnFailure`: Test that the old file is not deleted when the new file upload fails (thanks to transaction).

##### 5. Restructuring Existing Tests

All existing tests (43 tests) were rewritten to conform to the new structure. A unique `test_code` was added for each test to facilitate identification.

##### 6. Update `FileUploadController::tests()`

The controller now supports running both versions of tests (`FileUploadTest` for old version and `FileUploadPlusTest` for new version) via the `test_version=v1` or `test_version=v2` parameter (default is `v2`). This allows gradual migration.

#### Benefits

- Unified and easy-to-read test results.
- Ability to integrate tests into CI/CD systems easily.
- Complete isolation of tests from real data.
- Full coverage of edit operations added in version 1.1.0.
- Ease of adding new tests in the future by following the same pattern.

---

### Version Summary (1.0.8 – 1.2.0)

| Version | Key Features |
|---------|---------------|
| 1.0.8 | Comprehensive testing system (`FileUploadTest`), improved `deleteFile` security, test integration into `Plugin.php`. |
| 1.0.9 | Add `session_key` column to `system_files` to fully support temporary uploads. |
| 1.1.0 | Support for file replacement (edit), add `disable_edit`, `updateFieldConfig` function, improve `updateModelConfig`, move `Base64` into add-on, `beforeEditDelete/afterEditDelete` events, improve `attachTempFiles`. |
| 1.2.0 | Restructure testing system into `FileUploadPlusTest` with unified outputs, advanced exception handling, use of transactions, and addition of edit tests. |

---

### Upgrade Requirements

1. **Run new migrations** (to add `session_key` column):
   ```bash
   php artisan plugin:refresh Nano.FileUpload
   ```
   Or manually execute the `add_session_key_to_system_files.php` migration.

2. **Add environment variable** (optional) to control edit operation:
   ```ini
   NANO_FILE_UPLOAD_DISABLE_EDIT=false
   ```

3. **Update field definitions** to add `edit` permission if needed:
   ```php
   'permissions' => [
       'add'    => 'some.permission',
       'edit'   => 'some.edit.permission', // new
       'delete' => 'some.delete.permission',
       'view'   => 'some.view.permission',
   ]
   ```

4. **Listen to new events** (optional):
   ```php
   Event::listen('nano.fileupload.beforeEditDelete', function ($file, $modelClass, $field, $model) {
       // Backup or log
   });
   ```

5. **Run tests** (in development environment):
   - Via browser: `https://yourdomain.com/api/v1/fileupload/tests?test_version=v2`
   - Or via command line (after adding a custom command).

---

### Conclusion

Thanks to versions 1.0.8 through 1.2.0, the `Nano.FileUpload` add-on is more mature and reliable than ever. It can now:

- **Test itself automatically** through an integrated test system compatible with CI/CD.
- **Support file replacement** in `attachOne` relations safely using transactions and custom events.
- **Update field settings flexibly** without redefining the entire model.
- **Fully manage temporary files** via the `session_key` column.
- **Provide unified test results** to facilitate debugging and integration with external tools.

These improvements make the add-on ready for use in the largest and most complex projects, ensuring stability, security, and ease of maintenance.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService` Class](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager` Class](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)

## 2026-04-10 – 2026-04-11

** `Nano.FileUpload` Plugin Update – Version 1.2.1**

### Restructuring the Permission Validation System and Adding Fine-Grained Control at Model and Field Levels

---

### Summary of Updates

In version **1.2.1**, the permission validation system in `FileUploadRegistry` was completely restructured, with the addition of multiple validation layers (global, module, field) and the ability to independently disable specific operations. `FileUploadService` was also improved to use the new `validate` methods, the `attachTempFiles` function was completely rewritten, and `FileUploadException` was updated with new error codes and appropriate HTTP statuses.

---

### Version Goals

- **Unify and restructure validation methods** in `FileUploadRegistry` to be clearer and more organized.
- **Add `disabled_operations` mechanism** at model and field levels to independently disable specific operations (`add`, `edit`, `delete`, `view`).
- **Provide specialized `validate` methods** that throw appropriate exceptions instead of returning `false`, simplifying code in the service layer.
- **Improve `FileUploadService` code** to leverage the new `validate` methods, reducing duplicated code and unifying error messages.
- **Add a fully enhanced `attachTempFiles` function** with a unified response structure and WebSocket events.
- **Update `FileUploadException`** with new error codes and modify HTTP codes for some errors to avoid conflicts.

---

### New Features

#### 1. Specialized Validation Methods in `FileUploadRegistry`

A set of new methods was added to separate validation levels:

| Method | Description |
|--------|-------------|
| `canGlobal($operation)` | Check if the operation is globally enabled (via global settings `disable_upload`, `disable_edit`, `disable_delete`, `disable_get`). |
| `validateGlobal($operation)` | Like `canGlobal` but throws `FileUploadException` if the operation is disabled. |
| `canModel($modelClass, $operation)` | Check if the operation is enabled at model level (via `disabled_operations` in model definition). |
| `validateModel($modelClass, $operation)` | Like `canModel` but throws exception if operation is disabled. |
| `canField($modelClass, $field, $operation)` | Check if the operation is enabled at field level (via `disabled_operations` in field definition). |
| `validateField($modelClass, $field, $operation)` | Like `canField` but throws exception. |
| `validate($modelClass, $operation, $userType, $user, $field)` | Integrated method that combines checks for global, module, field, user type, and permissions, throwing the appropriate exception at the first failure. |

**Example of using `validate`:**
```php
// In FileUploadService::upload
$this->registry->validate($modelClass, $operation, $userType, $user, $field);
$this->registry->validateField($modelClass, $field, $operation);
```

#### 2. Addition of `disabled_operations` Property

It is now possible to disable specific operations at the model or field level using the `disabled_operations` key in registration definitions:

```php
// In model definition (Plugin::registerFileUploadFields)
\Nano\Shop\Models\Product::class => [
    'disabled_operations' => ['delete'], // disable deletion at product level
    'fields' => [
        'image' => [
            'type' => 'image',
            'disabled_operations' => ['edit'], // disable image replacement only
        ],
    ],
];
```

This allows precise prevention of certain operations without needing to disable full permissions or modify global settings.

#### 3. Improved `can()` Method in `FileUploadRegistry`

The `can()` method was rewritten to rely on the new validation chain:

1. Global level check (`canGlobal`)
2. Module level check (`canModel`)
3. Field level check (`canField`) if applicable
4. User type check (`isUserTypeAllowed`)
5. Specific permissions check (`permissions`)

This ensures that no operation that is disabled at any level is allowed.

#### 4. New Error Codes in `FileUploadException`

The following constants were added:

- `ERR_EDIT_DISABLED_GLOBALLY` – when editing is globally disabled (`disable_edit = true`)
- `ERR_MODEL_OPERATION_DISABLED` – when operation is disabled at model level
- `ERR_FIELD_OPERATION_DISABLED` – when operation is disabled at field level

**HTTP code changes:**
- `ERR_PERMISSION_DENIED` HTTP code changed from `403` to `422` (Unprocessable Entity) to avoid conflict with "inactive account" status which uses `403`.
- `ERR_UPLOAD_DISABLED_GLOBALLY`, `ERR_EDIT_DISABLED_GLOBALLY`, `ERR_DELETE_DISABLED_GLOBALLY`, `ERR_GET_DISABLED_GLOBALLY` now use code `503` (Service Unavailable).
- `ERR_MODEL_OPERATION_DISABLED`, `ERR_FIELD_OPERATION_DISABLED` use code `423` (Locked).

#### 5. Improved `upload` Method in `FileUploadService`

Manual permission checking code was replaced with `validate` calls:

**Before:**
```php
if (!$this->registry->can($modelClass, $operation, $userType, $user, $field)) {
    throw new FileUploadException(...);
}
```

**After:**
```php
$this->registry->validate($modelClass, $operation, $userType, $user, $field);
$this->registry->validateField($modelClass, $field, $operation);
```

The `skip_permission_check` option was also added to bypass permission checks (useful in automated tests).

#### 6. Complete Rewrite of `attachTempFiles` Method

The method now returns a unified response structure containing `code`, `status`, `message`, `data`, `error_code`, `process_data`, `debug`. It performs:

- Check that the model and field are registered in `FileUploadRegistry`.
- Permission check (using `edit` operation because attaching is considered a modification to the model).
- Validate the temporary key using `validateTempSessionKey`.
- Verify that `modelClass` and `field` match the key data.
- Verify that the current user is the creator of the temporary files (compare `userId` and `userType`).
- Find temporary files (`session_key`).
- Attach files to the model (set `attachment_id`, `attachment_type`, clear `session_key`, `expires_at`).
- Add files to the model relation (if exists).
- Fire `nano.fileupload.afterAttach` event.
- Send WebSocket notification.

**Example success response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Successfully attached 3 files",
    "data": [
        {"id": 10, "title": "image.jpg", "path": "/storage/...", "size": 1024, "content_type": "image/jpeg"}
    ],
    "process_data": {
        "field_config": {...},
        "key_data": {...},
        "attached_count": 3,
        "temp_session_key": "tmp_xxx",
        "user_type": "backend"
    }
}
```

#### 7. Improved `getFiles` Method

Improved extraction of `morphClass` using `app($modelClass)->getMorphClass()` instead of `$modelClass::getMorphClass()` to avoid errors in some contexts (e.g., when the model is not preloaded). Permission checking for `view` was also unified using `$this->registry->validate($modelClass, 'view', ...)`.

#### 8. Improved `deleteFile` Method

The permission checking logic was restructured to cover four cases:

| Case | Description | Validation Method |
|------|-------------|-------------------|
| 1 | `modelClass` and `field` explicitly provided | Use `registry->can()` with provided parameters |
| 2 | File attached to saved model (`attachment_type`, `attachment_id`, `field`) | Extract values from file and use `registry->can()` |
| 3 | Temporary file (`session_key`) | Validate key and match user |
| 4 | Insufficient information | Reject deletion directly |

**Improvement for temporary files:**
```php
elseif ($file->session_key) {
    $keyData = $this->validateTempSessionKey($file->session_key);
    if (!$keyData) throw ...;
    $currentUserId = $user->getKey() ?? 'guest';
    if ($keyData['userId'] != $currentUserId) throw ...;
    $usedModelClass = $keyData['modelClass'];
    $usedField = $keyData['field'];
    $isAuthorized = true;
}
```

#### 9. Improved `logFailedAttempt` Method

Improved extraction of `userId` to support multiple types of user objects:

```php
$userId = 'guest';
if ($user) {
    if (method_exists($user, 'getKey')) $userId = $user->getKey();
    elseif (property_exists($user, 'id')) $userId = $user->id;
    elseif (method_exists($user, 'getId')) $userId = $user->getId();
    else $userId = 'unknown';
}
```

This reduces errors when different user types (backend, frontend, custom) exist.

#### 10. Added `skip_permission_check` Option in Multiple Methods

The `skip_permission_check` option was added to `upload`, `uploadMultiple`, and `attachTempFiles` methods to completely bypass permission checks. This option is very useful in automated test environments (e.g., `FileUploadPlusTest`) where there is no real user or specific permissions.

**Usage example:**
```php
$result = $service->upload($modelClass, $field, $file, [
    'skip_permission_check' => true
]);
```

---

### Benefits and Added Value

- **Cleaner code**: `validate` methods eliminate the need for repetitive conditional blocks for permission checking.
- **Fine-grained control**: `disabled_operations` allows disabling specific operations at model or field level without affecting overall permissions or global settings.
- **Better compatibility with authentication systems**: Changing HTTP code for `ERR_PERMISSION_DENIED` from `403` to `422` avoids confusion between "unauthorized" and "inactive account".
- **Professional `attachTempFiles` method**: The method is now complete and integrated with the unified response structure, events, and WebSocket notifications.
- **Testing flexibility**: The `skip_permission_check` option simplifies writing unit tests that don't rely on a real user.
- **Enhanced security**: Permission checking in `deleteFile` covers all possible cases, preventing unauthorized deletion.

---

### Breaking Changes

1. **HTTP code change for `ERR_PERMISSION_DENIED`**:
   - If you have code that expects a `403` for this error, it will now receive `422`. It is recommended to update any error handling that relies on the code.

2. **`attachTempFiles` method now returns a different structure**:
   - Previously returned `true/false` or a simple array. Now returns a unified array containing `code`, `status`, `message`, `data`, etc. If you call this method directly, you will need to update your code to handle the new structure.

3. **Addition of `disabled_operations`**:
   - Does not affect existing settings unless you explicitly add it. Default settings allow all operations.

4. **New `validate` methods**:
   - They are used inside `FileUploadService`, but if you call `can()` directly, it will still work (with the improved chain).

---

### Upgrade Requirements

1. **Update code**:
   - Replace `FileUploadRegistry.php` with the version containing the new `validate` methods and `disabled_operations` property.
   - Replace `FileUploadService.php` with the improved version that uses `validate` and contains the new `attachTempFiles`.
   - Replace `FileUploadException.php` with the version containing the new error codes.

2. **No new migrations**:
   - This version does not require database changes or new columns.

3. **No configuration changes**:
   - `config.php` remains the same. Existing environment variables (`disable_upload`, `disable_edit`, `disable_delete`, `disable_get`) can be used the same way.

4. **Review model definitions** (optional):
   - If you wish to use `disabled_operations`, add the key to model or field definitions as shown in the examples above.

5. **Compatibility testing**:
   - It is recommended to run the `FileUploadPlusTest` (version 1.2.0) to ensure all operations work correctly with the new changes.
   - Verify that `edit` permissions work as expected when replacing files (if used).

6. **Update any code that depends on `attachTempFiles`**:
   - If you call `attachTempFiles` directly, you will need to modify your code to handle the new response structure.

---

### Conclusion

Version **1.2.1** represents a significant leap in the maturity of permission management within the `Nano.FileUpload` add-on. By adding specialized `validate` methods, the `disabled_operations` property, and improving the `attachTempFiles` and `deleteFile` methods, the system has become more organized, secure, and flexible. These changes enable the add-on to handle complex scenarios such as file replacement, temporarily disabling specific operations, and multi-level permission checks, while maintaining backward compatibility as much as possible.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)

## 2026-04-12 – 2026-04-14

**`Nano.FileUpload` Plugin Updates – Version 1.2.2**

### Summary of Updates

As part of the ongoing effort to enhance the security and flexibility of the file upload system, version 1.2.2 focuses on unifying and simplifying the validation process for temporary session keys across all service methods. A new trait called `HasFileUploadsMatchTempKey` was created, containing an advanced function `validateAndMatchTempKey` that combines key validation and matching against model, field, and user, with advanced options to control behavior (strict mode, grace period, caching, etc.). A helper function `fireEventSafeInTrait` was also added to ensure event firing compatibility with both Laravel and OctoberCMS.

The `upload`, `deleteFile`, `attachTempFiles`, and `getFiles` methods in `FileUploadService` were updated to use this new function instead of duplicated code, resulting in reduced complexity and increased security and uniformity.

---

### Version Goals

- **Unify temporary key validation logic**: Create a single central point for validating keys and matching them against model, field, and user.
- **Remove duplicated code**: Replace repetitive manual validation operations in several methods with a single function call.
- **Enhance security**: Add advanced options such as strict mode, grace period for expired keys, and retry support on expiry.
- **Improve compatibility**: Ensure events fire correctly in both Laravel and OctoberCMS environments via `fireEventSafeInTrait`.
- **Facilitate testing**: Provide options like `skip_user_check` and `cache_results` to simplify unit testing.

---

### New Features

#### 1. Trait `HasFileUploadsMatchTempKey`

The trait was created at path `Nano\FileUpload\Classes\FileUploadService\HasFileUploadsMatchTempKey` and contains:

##### a. `validateAndMatchTempKey` Function

This function is the core of the update. It validates the temporary key and matches it against expected data. The function provides the following options:

| Option | Type | Description |
|--------|------|-------------|
| `throw_on_failure` | bool | Throw exception on failure (true) or return error result array (false) |
| `skip_user_check` | bool | Skip user check (for internal use or tests) |
| `strict_mode` | bool | Strict mode: requires exact model and field match (true), or allows mismatch (false) |
| `stop_on_first_failure` | bool | Stop at first failure (true) or collect errors (false) |
| `allow_expired_key` | bool | Allow expired keys within a grace period |
| `expiry_grace_period` | int | Grace period in seconds (default 300) |
| `custom_validator` | callable | Additional validation function (receives `$keyData` and returns bool) |
| `cache_results` | bool | Cache validation results within the same request |
| `collect_metadata` | bool | Collect performance data (execution time, steps performed) |

**Basic usage example:**
```php
$keyData = $this->validateAndMatchTempKey(
    $tempSessionKey,
    $modelClass,
    $field,
    $user,
    ['throw_on_failure' => true, 'strict_mode' => true]
);
```

##### b. `manualValidateTempSessionKeyWithGrace` Function

An internal helper function used when the `allow_expired_key` option is enabled. It validates an expired key with a grace period without modifying the original `validateTempSessionKey` function.

##### c. `fireEventSafeInTrait` Function

A helper function to fire events safely and compatibly with both Laravel and OctoberCMS. It first checks whether the used object (e.g., `FileUploadService`) has a `fireEventSafe` method, uses it if found, otherwise applies equivalent logic supporting `Event::fire`, `Event::dispatch`, and the `event()` helper.

#### 2. Updated `upload` Method in `FileUploadService`

- **Before update**: Any temporary key passed from the client was accepted without validating or matching it against the model and field.
- **After update**: When `temp_session_key` is present in options, `validateAndMatchTempKey` is called to verify that the key belongs to the same model, field, and user; otherwise an appropriate exception is thrown.

**New code:**
```php
} elseif (isset($options['temp_session_key']) && !empty($options['temp_session_key'])) {
    $tempSessionKey = $options['temp_session_key'];
    $user = $this->userManager->getUser();
    $this->validateAndMatchTempKey($tempSessionKey, $modelClass, $field, $user, [
        'throw_on_failure' => true,
        'strict_mode' => true,
        'skip_user_check' => false,
    ]);
}
```

#### 3. Updated `deleteFile` Method in `FileUploadService`

- **Before update**: Manual key validation followed by separate `userId` comparison.
- **After update**: Use `validateAndMatchTempKey` with `strict_mode = false` (because a temporary file may not have an `attachment_type`) to unify the logic.

**New code:**
```php
elseif ($file->session_key) {
    $keyData = $this->validateAndMatchTempKey(
        $file->session_key,
        $file->attachment_type ?: '',
        $file->field,
        $user,
        ['throw_on_failure' => true, 'strict_mode' => false, 'skip_user_check' => false]
    );
    $usedModelClass = $keyData['modelClass'];
    $usedField = $keyData['field'];
    $isAuthorized = true;
}
```

#### 4. Updated `attachTempFiles` Method in `FileUploadService`

- **Before update**: Called `validateTempSessionKey` then separately compared `modelClass`, `field`, `userId`, and `userType` (more than 20 lines of code).
- **After update**: Replaced all this logic with a single call to `validateAndMatchTempKey`.

**New code:**
```php
$keyData = $this->validateAndMatchTempKey(
    $tempSessionKey,
    $modelClass,
    $field,
    $user,
    ['throw_on_failure' => true, 'strict_mode' => true, 'skip_user_check' => false]
);
$process_data['key_data'] = $keyData;
```

#### 5. Updated `getFiles` Method in `FileUploadService`

- **Before update**: Manual key validation followed by separate comparison of `userId`, `userType`, `modelClass`, and `field`.
- **After update**: Use `validateAndMatchTempKey` to validate the key before using it in the query.

**New code:**
```php
} elseif (isset($options['temp_session_key'])) {
    $user = $this->userManager->getUser();
    $this->validateAndMatchTempKey(
        $options['temp_session_key'],
        $modelClass,
        $field,
        $user,
        ['throw_on_failure' => true, 'strict_mode' => true, 'skip_user_check' => false]
    );
    $query->where('session_key', $options['temp_session_key']);
}
```

#### 6. Improved Event Firing in the Trait

All direct `Event::fire` calls inside `validateAndMatchTempKey` were replaced with calls to `$this->fireEventSafeInTrait`, ensuring events work in both Laravel and OctoberCMS environments, and leveraging the `fireEventSafe` method in `FileUploadService` if available.

---

### Benefits

- **Reduced duplicated code**: Removed more than 50 lines of repetitive code from various methods.
- **Increased security**: Unified validation ensures no gaps from forgetting to check model or user matching in any function.
- **High flexibility**: Multiple options in `validateAndMatchTempKey` allow customizing behavior per case (strict mode, grace period, caching, etc.).
- **Improved compatibility**: `fireEventSafeInTrait` ensures events work in all environments.
- **Ease of testing**: `skip_user_check`, `cache_results`, and `collect_metadata` options simplify unit testing and performance analysis.
- **Built-in documentation**: The new function includes comprehensive documentation for all options.

---

### Upgrade Requirements

1. **Update code**: Replace `FileUploadService.php` with the updated version that uses the new trait.
2. **Add the trait**: Create the file `HasFileUploadsMatchTempKey.php` in the path `classes/FileUploadService/` with the required content.
3. **No new migrations**: This version does not require database changes.
4. **No configuration changes**: `config.php` remains the same.
5. **Compatibility testing**: It is recommended to run the `FileUploadPlusTest` tests to ensure all operations work correctly.

---

### Conclusion

Version 1.2.2 is an important step toward unifying and simplifying temporary key validation logic in the file upload system. By introducing the `HasFileUploadsMatchTempKey` trait and the `validateAndMatchTempKey` function, it is now possible to perform secure and comprehensive validation of temporary keys in any function that needs it, with advanced options for fine-grained control. The updates made to the `upload`, `deleteFile`, `attachTempFiles`, and `getFiles` methods make the code cleaner and less error-prone, while maintaining full backward compatibility.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)


## 2026-04-15 – 2026-04-16

**`Nano.FileUpload` Plugin Update – Version 1.2.3**

### Adding New API Endpoints for Querying Modules, Permissions, and Validating Temporary Keys

---

### Summary of Updates

In version **1.2.3**, the add-on's API was enriched with several new endpoints that allow developers and external systems to:

- **Query registered modules** and their fields while respecting user permissions.
- **Fetch detailed settings** for a specific module or field.
- **Fetch field constraints** (e.g., max size, allowed types).
- **Fetch advanced processing options** (storage disk, automatic resizing, watermark).
- **Check permissions** at global, module, and field levels.
- **Integrated permission check** using the `validate()` method.
- **Validate and match a temporary key** via API (using the `validateAndMatchTempKey` function from the trait).

All new endpoints are protected by `oauth-users` and follow the unified response structure (`code`, `status`, `message`, `data`, ...).

---

### Version Goals

- **Enable developers to explore registered settings** in `FileUploadRegistry` without needing direct access to code or database.
- **Facilitate front-end integration** (e.g., React or Vue applications) by providing dynamic information about allowed fields and constraints.
- **Check permissions before attempting upload** to avoid rejection errors and improve user experience.
- **Provide an endpoint to validate temporary keys**, allowing front-ends to ensure key validity before using it.
- **Standardize the way of querying system information** via API instead of relying on internal tools.

---

### New Endpoints

#### 1. Get All Registered Modules (Filtered by Permission)

- **Path:** `GET /api/v1/fileupload/models`
- **Description:** Returns a list of registered modules that the current user has `view` permission on, with brief information about their fields (name, type, whether multiple).
- **Response:** Array of objects containing `class`, `label`, `enabled`, `allowed_user_types`, `fields`.

#### 2. Get Settings for a Specific Module

- **Path:** `GET /api/v1/fileupload/models/{modelClass}`
- **Description:** Returns the full settings of a specific module (after checking existence and `view` permission).
- **Response:** Contains `enabled`, `label`, `allowed_user_types`, `disabled_operations`, and `fields` (with brief information per field).

#### 3. Get Settings for a Specific Field

- **Path:** `GET /api/v1/fileupload/models/{modelClass}/fields/{field}`
- **Description:** Returns settings for a specific field (e.g., type, max size, allowed types).
- **Response:** Includes `type`, `label`, `multiple`, `required`, `max_filesize`, `allowed_types`, `use_caption`, `disabled_operations`.

#### 4. Get Field Constraints

- **Path:** `GET /api/v1/fileupload/models/{modelClass}/fields/{field}/constraints`
- **Description:** Returns the field constraints used in file validation (`max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`).
- **Benefit:** Front-ends can apply the same constraints before uploading.

#### 5. Get Advanced Processing Options for a Specific Field

- **Path:** `GET /api/v1/fileupload/processing-options/{modelClass}/{field}`
- **Description:** Returns the field's processing options: `storage_disk`, `auto_resize`, `resize_options`, `auto_watermark`, `watermark_options`.
- **Benefit:** Used primarily by advanced front-ends that need to know automatic transformation settings.

#### 6. Check if an Operation is Globally Enabled

- **Path:** `GET /api/v1/fileupload/permissions/global/{operation}`
- **Parameters:** `operation` can be `add`, `edit`, `delete`, `view`.
- **Response:** Returns `allowed` (true/false) and `disabled_globally` (opposite of `allowed`).

#### 7. Check Operation Permission at a Specific Module Level

- **Path:** `GET /api/v1/fileupload/permissions/model/{modelClass}/{operation}`
- **Response:** Returns `model_operation_enabled`, `user_type_allowed`, `can_proceed` (combination of both conditions).

#### 8. Check Operation Permission at a Specific Field Level

- **Path:** `GET /api/v1/fileupload/permissions/field/{modelClass}/{field}/{operation}`
- **Response:** Returns `field_operation_enabled`, `user_has_full_permission`, `can_proceed`.

#### 9. Integrated Permission Check (using `validate`)

- **Path:** `POST /api/v1/fileupload/permissions/check`
- **Data (JSON):**
  ```json
  {
    "model_class": "Nano\\Shop\\Models\\Product",
    "operation": "edit",
    "field": "image"
  }
  ```
- **Response:** Returns `allowed` (true/false) and an explanatory message.
- **Behavior:** Uses `$this->registry->validate()` which checks global, module, field, user type, and specific permissions, and returns the result without throwing an exception (the exception is caught and converted to a response).

#### 10. Validate and Match a Temporary Key

- **Path:** `POST /api/v1/fileupload/temp-key/validate`
- **Data (JSON):**
  ```json
  {
    "temp_key": "tmp_xxxx:yyyy",
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "strict_mode": true,
    "allow_expired_key": false,
    "expiry_grace_period": 300
  }
  ```
- **Description:** Calls the `validateAndMatchTempKey` function from the `HasFileUploadsMatchTempKey` trait and returns the result as JSON.
- **Response:** On success returns `valid: true`, `key_data`, and `matched_data`; on failure returns an error message with `error_code`.

---

### Usage Examples

#### Get modules available to the current user

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/models" \
  -H "Authorization: Bearer <token>"
```

#### Get settings for the `image` field in the `Product` module

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/models/Nano%5CShop%5CModels%5CProduct/fields/image" \
  -H "Authorization: Bearer <token>"
```

#### Check `edit` permission on a specific field

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/permissions/field/Nano%5CShop%5CModels%5CProduct/image/edit" \
  -H "Authorization: Bearer <token>"
```

#### Validate a temporary key

```bash
curl -X POST "https://yourdomain.com/api/v1/fileupload/temp-key/validate" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "temp_key": "tmp_eyJtb2RlbENsYXNzIjoiTmFub1xTaG9wXE1vZGVsc1xQcm9kdWN0Iiw...",
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "strict_mode": true
  }'
```

---

### Benefits and Added Value

- **Empower front-ends**: Front-end applications (e.g., Single Page Applications) can now dynamically discover available fields, constraints, and permissions, allowing them to build adaptive file upload forms.
- **Reduce server dependency**: Clients can check their permissions before attempting upload, reducing rejection requests and improving user experience.
- **Live settings documentation**: Developers can explore add-on settings via API instead of searching through code.
- **Additional security**: Permission endpoints do not expose sensitive information (e.g., storage keys) and are subject to the same user permissions.
- **Easy integration with external tools**: These endpoints can be used in CI/CD systems or content management tools.

---

### Upgrade Requirements

1. **Update code**:
   - Replace `FileUploadController.php` with the version containing the new methods.
   - Update `routes.php` to add the new routes.

2. **No new migrations**:
   - This version does not require database changes.

3. **No configuration changes**:
   - `config.php` remains the same.

4. **API permissions**:
   - All new endpoints are protected by `oauth-users`, so the user must have a valid token to access them.

5. **Compatibility testing**:
   - It is recommended to run the `FileUploadPlusTest` tests to ensure the new version does not affect existing functionality.
   - The new endpoints can be tested using tools like Postman or cURL.

---

### Conclusion

Version **1.2.3** represents a significant leap in the accessibility of file upload system information via API. By adding endpoints to query modules, fields, permissions, and validate temporary keys, the add-on has become more open and integrable with external systems and modern front-ends. These features make it easy to build rich applications that rely on `Nano.FileUpload` as a back-end for file management, while maintaining the highest levels of security and control.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)

## 2026-04-16 - 2026-04-18

**`Nano.TagsApi` Add-on Updates – Version 1.0.10**

### Summary of Updates

In this version, comprehensive improvements have been made to `TagsApi` management to make it more flexible and integrated with system-wide settings, with support for new options in `config.php`, and improvements to the `Categories`, `Tags`, `Types`, and `Menus` controllers. New events have also been added to extend queries, pagination and ordering mechanisms have been improved, and advanced filters such as `is_has_products`, `is_has_shops`, and `is_has_cateables` are now supported in `Categories`.

Key updates include:

- **Addition of new configuration options** in `config.php` (e.g., `include`, `exclude`, `per_page`, `is_public`) for all sections.
- **Improvement of the `Categories` controller** to support `mainCategoriesId`, `is_has_*` with complex `leftJoin`, and `is_to_sql` for debugging.
- **Addition of `extendQueryBefore` and `extendQuery` events** in all controllers.
- **Support for reading `is_public`, `per_page`, `page` from settings** with the ability to override via `Input::get()`.
- **Addition of `is_departments` option** in `Types` to control department filtering.
- **Addition of `is_stop_order_type` and `is_stop_order_type_departments` options** to control `OrderHelper` integration.
- **Improvement of `availablefilters` and `formfields` functions** by adding `input_data` and `process_data` in responses.

---

### Version 1.0.10 – Comprehensive Improvement of Settings and Controllers

#### Version Goals

- Make all controllers rely on settings stored in `config.php` rather than hardcoded values or `Input::get()` only.
- Add greater flexibility in filtering categories, types, and tags based on relationships (e.g., products, shops, associated objects).
- Improve performance of complex queries in `Categories` by using `leftJoin` with `nano_tags_cateables`.
- Provide debugging tools (`is_to_sql`) to facilitate query development.
- Standardize the handling of `page` and `per_page` across all controllers.

#### New Features

##### 1. Expanded `config.php` Settings File

New keys have been added for all sections (`types`, `tags`, `categories`, `menus`):

| Key | Description | Example Value |
|-----|-------------|---------------|
| `include` | Default relationships to include | `'children,products'` |
| `exclude` | Columns to exclude | `'password,remember_token'` |
| `per_page` | Number of results per page | `50` |
| `is_public` | Filter public records | `true` |
| `is_has_cateables` (for categories) | Filter categories associated with any object | `true` |
| `is_has_shops` (for categories) | Filter categories associated with shops | `true` |
| `is_has_products` (for categories) | Filter categories associated with products | `true` |
| `is_departments` (for types) | Filter types that represent departments | `true` |

**Example from the new `config.php`:**
```php
'categories' => [
    'include' => env('NANO_TAGSAPI_CATEGORIES_INCLUDE', ''),
    'exclude' => env('NANO_TAGSAPI_CATEGORIES_EXCLUDE', ''),
    'is_display_empty' => env('NANO_TAGSAPI_CATEGORIES_IS_DISPLAY_EMPTY', true),
    'is_has_cateables' => env('NANO_TAGSAPI_CATEGORIES_IS_HAS_CATEABLES', false),
    'is_has_shops' => env('NANO_TAGSAPI_CATEGORIES_IS_HAS_SHOP', false),
    'is_has_products' => env('NANO_TAGSAPI_CATEGORIES_IS_HAS_PRODUCTS', false),
    'is_public' => env('NANO_TAGSAPI_CATEGORIES_IS_PUBLIC', true),
    'per_page' => env('NANO_TAGSAPI_CATEGORIES_PER_PAGE', 50),
],
```

##### 2. Improvement of the `Categories` Controller (`Categories.php`)

- **Read `isPublic` from settings** if not passed in the request.
- **Support `mainCategoriesId`** to merge with `parent_id` (useful for displaying specific main categories along with certain subcategories).
- **Add `is_has_products`, `is_has_shops`, `is_has_cateables` options**:
  - When any of these options are enabled, a complex query is built using `leftJoin` with the `nano_tags_categories as child` and `nano_tags_cateables` tables.
  - The count of associated objects (`cateables_count`, `sub_cateables_count`) is calculated and results are filtered based on `cateables_count`.
- **Add `is_to_sql` option** to print the final SQL query in `trace_log` for easier debugging.
- **Improve handling of `orderBy` and `orderDirection`**:
  ```php
  if ($orderBy && $orderDirection) {
      $posts->orderBy(\Db::raw($orderBy), \Db::raw($orderDirection));
  } else {
      $posts->orderBy(\Db::raw(\Config::get('nano.tagsapi::categories.order_by','sort_order')), 
                     \Db::raw(\Config::get('nano.tagsapi::categories.order_dir','asc')));
  }
  ```
- **Add `$CategorieTransformer->defaultIncludes`** based on the `$include` value.

##### 3. Addition of `api.list.extendQueryBefore` and `api.list.extendQuery` Events

These two events have been added to all controllers (`Categories`, `Tags`, `Types`, `Menus`) to allow developers to modify the query before and after applying basic filters.

**Usage example:**
```php
Event::listen('api.list.extendQuery', function ($query, $controller, $model, $options) {
    $query->where('is_featured', true);
});
```

##### 4. Improvement of the `Types` Controller

- **Add `is_departments` support**:
  ```php
  $is_departments = Input::get('isDepartments', \Config::get('nano.tagsapi::types.is_departments','*'));
  if($is_departments !== null && $is_departments !== '*' && $is_departments !== 'all' && is_bool($is_departments)){
      if($is_departments) $posts = $posts->where('is_departments',true);
      else $posts = $posts->where('is_departments',false);
  }
  ```
- **Add two options to control `OrderHelper` integration**:
  - `is_stop_order_type`: to stop applying `OrderHelper` at the `types` level.
  - `is_stop_order_type_departments`: to stop applying it at the level of associated departments.
- **Improve `availablefilters` and `formfields` functions**:
  - Add `input_data` and `process_data` in responses to facilitate tracking of input and processed data.
  - Improve exception handling and add `debug` details in the development environment.

##### 5. Improvement of the `Tags` and `Menus` Controllers

- **Read `per_page` and `page` from settings** with the ability to override via `Input::get()`.
- **Add `$src_options` and event calls** in the same manner as `Categories`.
- **Support `isPublic` from settings**:
  ```php
  if(Input::get('isPublic', \Config::get('nano.tagsapi::tags.is_public', true))){
      $posts = $posts->where('is_public', true);
  }
  ```

##### 6. Standardized Pagination Handling

Across all controllers, the following code is now the standard:
```php
$per_page = Input::get('per_page', null);
if(is_null($per_page))
    $per_page = \Config::get('nano.tagsapi::categories.per_page', 15);
$per_page = intval($per_page);
if(!$per_page) $per_page = 15;

$page = intval(Input::get('page', 1));
if(!$page) $page = 1;

$paginator = $posts->paginate($per_page, $page);
```

---

### Benefits and Added Value

- **Greater flexibility**: The API's behavior can now be fully controlled via environment variables or the `config.php` file without modifying code.
- **Improved performance**: Complex queries in `Categories` are now more efficient thanks to optimized `leftJoin` and appropriate `groupBy`.
- **Ease of debugging**: The addition of `is_to_sql` allows developers to review final queries directly.
- **Compatibility with other add-ons**: The new `extendQueryBefore/After` events allow other add-ons to easily modify queries.
- **Better relationship support**: The ability to filter categories based on the existence of products, shops, or any associated object via `cateables`.

---

### Upgrade Requirements to Version 1.0.10

1. **Update the `config.php` file**:
   - Add the new variables to your settings file or in `.env` as needed.
   - Example new environment variables:
     ```ini
     NANO_TAGSAPI_CATEGORIES_IS_HAS_PRODUCTS=true
     NANO_TAGSAPI_CATEGORIES_IS_HAS_SHOP=true
     NANO_TAGSAPI_TYPES_IS_DEPARTMENTS=true
     NANO_TAGSAPI_TYPES_PER_PAGE=50
     NANO_TAGSAPI_CATEGORIES_PER_PAGE=30
     ```

2. **Update any custom code that relies on the API**:
   - If you directly use `Input::get('per_page')` in your application, it will still work, but the default value now comes from settings.
   - If you rely on the default value of `15` for pagination, ensure that the `per_page` settings in `config.php` are appropriate.

3. **Take advantage of the new events**:
   - You can now add listeners for the `api.list.extendQueryBefore` and `api.list.extendQuery` events in the `boot()` method of any add-on.

4. **Test complex queries**:
   - If you enable `is_has_products` or `is_has_shops`, ensure that the `nano_tags_cateables` tables contain correct data.
   - Use `is_to_sql=true` in the development environment to review the final query.

---

### Environment Variables Supported in `Nano.TagsApi`

You can fully customize the add-on's behavior via environment variables in the `.env` file, without needing to modify the `config.php` file directly. Below are all available variables with their descriptions and default values:

#### 1. `types` Settings

| Variable | Description | Default Value |
|----------|-------------|---------------|
| `NANO_TAGSAPI_TYPES_INCLUDE` | Default relationships to include (comma-separated) | `''` |
| `NANO_TAGSAPI_TYPES_EXCLUDE` | Columns to exclude from the query | `''` |
| `NANO_TAGSAPI_TYPES_IS_DEPARTMENTS` | Filter types that represent departments (`true`/`false`) | `true` |
| `NANO_TAGSAPI_TYPES_IS_PUBLIC` | Show only public types | `true` |
| `NANO_TAGSAPI_TYPES_PER_PAGE` | Number of results per page | `15` |
| `NANO_TAGSAPI_TYPES_ORDER_BY` | Default ordering column | `sort_order` |
| `NANO_TAGSAPI_TYPES_ORDER_DIR` | Default ordering direction (`asc`/`desc`) | `asc` |
| `NANO_TAGSAPI_TYPES_IS_STOP_ORDER_TYPE` | Stop integrating `OrderHelper` at the `types` level | `false` |
| `NANO_TAGSAPI_TYPES_IS_STOP_ORDER_TYPE_DEPARTMENTS` | Stop integrating `OrderHelper` at the level of associated departments | `false` |

#### 2. `tags` Settings

| Variable | Description | Default Value |
|----------|-------------|---------------|
| `NANO_TAGSAPI_TAGS_INCLUDE` | Default relationships to include | `''` |
| `NANO_TAGSAPI_TAGS_EXCLUDE` | Columns to exclude | `''` |
| `NANO_TAGSAPI_TAGS_IS_PUBLIC` | Show only public tags | `true` |
| `NANO_TAGSAPI_TAGS_PER_PAGE` | Number of results per page | `15` |
| `NANO_TAGSAPI_TAGS_ORDER_BY` | Default ordering column | `sort_order` |
| `NANO_TAGSAPI_TAGS_ORDER_DIR` | Default ordering direction | `asc` |

#### 3. `categories` Settings

| Variable | Description | Default Value |
|----------|-------------|---------------|
| `NANO_TAGSAPI_CATEGORIES_INCLUDE` | Default relationships to include | `''` |
| `NANO_TAGSAPI_CATEGORIES_EXCLUDE` | Columns to exclude | `''` |
| `NANO_TAGSAPI_CATEGORIES_IS_DISPLAY_EMPTY` | Show empty categories (without associated content) | `true` |
| `NANO_TAGSAPI_CATEGORIES_IS_HAS_CATEABLES` | Filter categories associated with any object via `cateables` | `false` |
| `NANO_TAGSAPI_CATEGORIES_IS_HAS_SHOP` | Filter categories associated with shops | `false` |
| `NANO_TAGSAPI_CATEGORIES_IS_HAS_PRODUCTS` | Filter categories associated with products | `false` |
| `NANO_TAGSAPI_CATEGORIES_IS_PUBLIC` | Show only public categories | `true` |
| `NANO_TAGSAPI_CATEGORIES_PER_PAGE` | Number of results per page | `15` |
| `NANO_TAGSAPI_CATEGORIES_ORDER_BY` | Default ordering column | `sort_order` |
| `NANO_TAGSAPI_CATEGORIES_ORDER_DIR` | Default ordering direction | `asc` |

#### 4. `menus` Settings

| Variable | Description | Default Value |
|----------|-------------|---------------|
| `NANO_TAGSAPI_MENUS_INCLUDE` | Default relationships to include | `''` |
| `NANO_TAGSAPI_MENUS_EXCLUDE` | Columns to exclude | `''` |
| `NANO_TAGSAPI_MENUS_IS_PUBLIC` | Show only public menus | `true` |
| `NANO_TAGSAPI_MENUS_PER_PAGE` | Number of results per page | `15` |
| `NANO_TAGSAPI_MENUS_ORDER_BY` | Default ordering column | `sort_order` |
| `NANO_TAGSAPI_MENUS_ORDER_DIR` | Default ordering direction | `asc` |

---

#### Example `.env` file:

```ini
# Nano.TagsApi Settings
NANO_TAGSAPI_CATEGORIES_IS_HAS_PRODUCTS=true
NANO_TAGSAPI_CATEGORIES_IS_HAS_SHOP=true
NANO_TAGSAPI_CATEGORIES_PER_PAGE=30
NANO_TAGSAPI_TYPES_IS_DEPARTMENTS=true
NANO_TAGSAPI_TYPES_PER_PAGE=50
NANO_TAGSAPI_TAGS_PER_PAGE=20
NANO_TAGSAPI_CATEGORIES_INCLUDE=children,parent
NANO_TAGSAPI_TYPES_EXCLUDE=password,remember_token
```

---

#### Important Notes

- All variables are optional. If not set, the default values built into `config.php` will be used.
- Any of these settings can be overridden by passing the same parameter in an API request (e.g., `?per_page=10`), as `Input::get()` takes precedence over the settings.
- It is recommended to use environment variables in the production environment.

---

### Conclusion

Version `1.0.10` of the `Nano.TagsApi` add-on represents a qualitative leap in system flexibility and extensibility. By relying on centralized settings, adding extension events, and improving `Categories` queries, it is now easier to integrate `TagsApi` with the requirements of any complex project. The improved handling of pagination and ordering ensures consistent behavior across all endpoints.

We recommend that all projects using `Nano.TagsApi` upgrade to this version to benefit from the improvements and better performance.

---

## 2026-04-18 

**Adding Support for New SMS Providers to the Nano.SmsNotify Plugin**

### Introduction
The **Nano.SmsNotify** plugin has been developed to manage and send short text messages (SMS) through multiple providers. The plugin currently supports many service providers such as Twilio, Nexmo, Alawaeltec, SmsSaudi, and others. In this latest update (version 1.0.13), we have added five new SMS providers that cover additional regional needs and provide more options for users.

### New Providers
1. **BdBulkSms** – Specializes in Bangladesh service (GreenWeb).  
2. **ReveSms** – A provider that supports sending via API with authentication using API Key and Secret Key.  
3. **SmsToday** – Provides easy SMS sending through a simple endpoint.  
4. **TonkraSms** – Supports sending using Bearer token and covers multiple regions.  
5. **ReleansSms** – A global provider that requires Bearer token and Sender ID.  

### Objective
To expand the customer base that needs to use specific local or global providers, thereby increasing system flexibility and enabling customers to choose the most suitable option in terms of cost and geographic coverage.

### Update Details

#### 1. Adding Provider Classes
Five new classes have been created within the `Nano\SmsNotify\SmsChannels` namespace:

- `BdBulkSms.php`  
- `ReveSms.php`  
- `SmsToday.php`  
- `TonkraSms.php`  
- `ReleansSms.php`

Each class inherits from `BaseChannel` and implements the `send()` method, which uses cURL to communicate with the provider's API, in addition to defining the required configuration fields via `defineFormConfig()`.

#### 2. File Structure
The classes have been placed in the path:
```
plugins/nano/smsnotify/smschannels/
```
Additionally, partial instruction files (`_info.htm`) have been added for each provider, explaining how to obtain the required data (API keys, sender numbers, etc.).

#### 3. Registering Providers in the Plugin
The `registerSmsChannels()` method in `Plugin.php` has been updated by adding the following lines:
```php
'bdbulksms'    => \Nano\SmsNotify\SmsChannels\BdBulkSms::class,
'revesms'      => \Nano\SmsNotify\SmsChannels\ReveSms::class,
'smstoday'     => \Nano\SmsNotify\SmsChannels\SmsToday::class,
'tonkrasms'    => \Nano\SmsNotify\SmsChannels\TonkraSms::class,
'releanssms'   => \Nano\SmsNotify\SmsChannels\ReleansSms::class,
```
Thus, these providers appear in the channels management interface within the plugin.

#### 4. Updating the Version File (version.yaml)
A new version `1.0.13` has been added with the update description:
```yaml
1.0.13:
    - Add new SMS channels: BdBulkSms, ReveSms, SmsToday, TonkraSms, ReleansSms
    - Extend SMS gateway support for additional regions
```

#### 5. Testing the Providers
Initial tests have been conducted for each provider to ensure connectivity and the ability to send test messages. Error handling and result return have been verified to be compatible with the `BaseChannel` structure (success/failure status, message ID, error message, etc.).

### Expected Benefits
- **Expanding the customer base**: Supporting providers that cover new markets (Bangladesh, multiple regions).  
- **Increased flexibility**: Offering multiple options to customers based on sending cost and coverage.  
- **Improved user experience**: Providing clear instructions within the plugin to easily configure each provider.

### Future Steps
- Monitor the performance of the new providers after launch.  
- Update documentation to include the new providers.  
- Add more providers based on customer requests.


## 2026-04-16 – 2026-04-18

**`Nano.FileUpload` Add-on Updates – Versions 1.2.4 and 1.2.5**

### Summary of Updates

The add-on has seen two important developments in versions **1.2.4** and **1.2.5**:

- **Version 1.2.4** added a new security feature at the field level: `allow_upload_only_when_model_exists`, which prevents uploading files to a specific field until the original model has been saved (a valid `model_id` exists). This feature is particularly useful for multiple fields (e.g., image galleries) where images should not be uploaded before the main record is created.

- **Version 1.2.5** completely restructured the API endpoints for querying modules and permissions by moving the `model_class` and `field` parameters from route parameters to query parameters. This change solves the problem of encoding model names containing backslashes (`\`) and makes the routes cleaner and more secure.

---

## Version 1.2.4 – Preventing File Upload Before Model Save

### Version Goals

- **Add an additional security layer** to prevent uploading files to fields that require the model to exist first (e.g., image galleries, documents attached to a saved record).
- **Improve developer experience** by providing the `allow_upload_only_when_model_exists` option in field definitions, allowing the prevention of temporary uploads (using `temp_session_key`) for that field.
- **Avoid accumulation of orphan files** (unattached) in the system.

### New Features

#### 1. Adding the `allow_upload_only_when_model_exists` Option in Field Definition

It is now possible to set this option on any field definition (especially for fields of type `multiple` or `image` that require the model to exist):

```php
'fields' => [
    'gallery' => [
        'type' => 'multiple',
        'max_filesize' => 2048,
        'allowed_types' => 'jpg,jpeg,png',
        'max_files' => 10,
        'allow_upload_only_when_model_exists' => true, // ⬅️ new
    ],
],
```

- **Default value:** `false` (allows temporary upload even for unsaved models).
- **When enabled (`true`):** Prevents uploading files to this field if a valid `model_id` is not provided (i.e., the model does not exist in the database). The request will be rejected with an error message `FILE_UPLOAD_PERMISSION_DENIED` and the message `model_must_exist_before_upload`.

#### 2. Validation in `FileUploadService::upload()`

Logic was added to the `upload()` method to check this option:

- It checks whether the model exists (`model_exists`) either through `options['model']` (if present and saved) or by searching for `model_id` and fetching the model.
- If the option is enabled and the model does not exist, a `FileUploadException` is thrown with the appropriate error code.

#### 3. New Translation Message

A new translation key `errors.model_must_exist_before_upload` was added:

```php
'model_must_exist_before_upload' => 'Cannot upload files to the ":field" field until ":model" is saved. Please save the :model first and then try again.',
```

### Benefits

- **Prevent orphan files:** Ensures there are no files unattached to any database record.
- **Fine-grained control:** The feature can be enabled only on specific fields (e.g., galleries) while leaving other fields (e.g., avatar) allowing temporary upload.
- **Enhanced security:** Prevents users from uploading files to fields that require the original record to exist before it is created.

---

## Version 1.2.5 – Restructuring API Routes (Query Parameters instead of Route Parameters)

### Version Goals

- **Solve the problem of encoding model names** containing backslashes (`\`) in routes (e.g., `Nano\Shop\Models\Product`). Passing them in the route required encoding (`Nano%5CShop%5CModels%5CProduct`), making routes impractical and unreadable.
- **Simplify endpoints** by using query parameters (`?model_class=...`) instead of route parameters.
- **Maintain backward compatibility** by keeping the old routes (deprecated) while making the new routes available.

### New Features

#### 1. New Endpoints (using query parameters)

| Function | Old Path (deprecated) | New Path |
|----------|----------------------|----------|
| Get module config | `GET /models/{modelClass}` | `GET /model/config?model_class=...` |
| Get field config | `GET /models/{modelClass}/fields/{field}` | `GET /field/config?model_class=...&field=...` |
| Get field constraints | `GET /models/{modelClass}/fields/{field}/constraints` | `GET /field/constraints?model_class=...&field=...` |
| Get processing options | `GET /processing-options/{modelClass}/{field}` | `GET /processing-options?model_class=...&field=...` |
| Check model permission | `GET /permissions/model/{modelClass}/{operation}` | `GET /permissions/model?model_class=...&operation=...` |
| Check field permission | `GET /permissions/field/{modelClass}/{field}/{operation}` | `GET /permissions/field?model_class=...&field=...&operation=...` |

**Note:** The endpoints `GET /models`, `GET /permissions/global/{operation}`, `POST /permissions/check`, and `POST /temp-key/validate` remain unchanged.

#### 2. Updated Controller Methods in `FileUploadController`

The following methods were modified to read parameters from `Input::get()` (or query parameters) instead of route parameters, with support for passing parameters via the old method as a fallback (for backward compatibility):

- `getModelConfig($modelClass = null)`
- `getFieldConfig($modelClass = null, $field = null)`
- `getFieldConstraints($modelClass = null, $field = null)`
- `getProcessingOptions($modelClass = null, $field = null)`
- `checkModelPermission($modelClass = null, $operation = null)`
- `checkFieldPermission($modelClass = null, $field = null, $operation = null)`

**Example of the new method for `getModelConfig`:**

```php
public function getModelConfig($modelClass = null)
{
    $modelClass = $modelClass ?? Input::get('model_class');
    if (!$modelClass) {
        return $this->errorResponse('Missing model_class parameter', null, 400);
    }
    // ... remaining logic
}
```

#### 3. Updated `routes.php` File

Old routes were commented out (using `/* ... */`) and new routes were added:

```php
// New routes
Route::get('model/config', [ 'as' => 'model.config', 'uses' => 'FileUploadController@getModelConfig' ]);
Route::get('field/config', [ 'as' => 'field.config', 'uses' => 'FileUploadController@getFieldConfig' ]);
Route::get('field/constraints', [ 'as' => 'field.constraints', 'uses' => 'FileUploadController@getFieldConstraints' ]);
Route::get('processing-options', [ 'as' => 'processing.options', 'uses' => 'FileUploadController@getProcessingOptions' ]);
Route::get('permissions/model', [ 'as' => 'permissions.model', 'uses' => 'FileUploadController@checkModelPermission' ]);
Route::get('permissions/field', [ 'as' => 'permissions.field', 'uses' => 'FileUploadController@checkFieldPermission' ]);
```

#### 4. Examples of New Requests

**Get `Product` module config:**

```http
GET /api/v1/fileupload/model/config?model_class=Nano\Shop\Models\Product HTTP/1.1
Authorization: Bearer ...
```

**Get constraints for `image` field:**

```http
GET /api/v1/fileupload/field/constraints?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

**Check `edit` permission on `image` field:**

```http
GET /api/v1/fileupload/permissions/field?model_class=Nano\Shop\Models\Product&field=image&operation=edit HTTP/1.1
Authorization: Bearer ...
```

### Benefits

- **Clean and readable routes** – no need to encode model names.
- **Ease of use from any HTTP client** – pass `model_class` as a normal value in the query string.
- **Backward compatibility** – old routes still work (but are deprecated) and will be removed in a future version.
- **Additional security** – avoid handling user input as part of the route (reduces risk of malicious routing).

---

## Upgrade Requirements (from 1.2.3 to 1.2.5)

1. **Update code**:
   - Replace `FileUploadController.php` with the version containing the updated methods (supports query parameters).
   - Replace `routes.php` with the version containing the new routes.
   - Replace `FileUploadRegistry.php` with the version containing the `allow_upload_only_when_model_exists` option (included in version 1.2.4).
   - Replace `FileUploadService.php` with the version implementing the new option validation.

2. **Update model definitions** (optional):
   - If you wish to use `allow_upload_only_when_model_exists`, add it to the desired field definitions.

3. **No new migrations** – no database changes.

4. **No configuration changes** – `config.php` remains the same.

5. **Update API clients**:
   - If you are using the old endpoints (with curly braces), it is recommended to switch to the new routes immediately. The old routes still work currently but are deprecated and will be removed in version 1.3.0.

6. **Compatibility testing**:
   - Ensure that all new endpoints work correctly with `model_class` as a query parameter.
   - Test a field with `allow_upload_only_when_model_exists = true` to ensure upload is rejected before the model is saved.

---

## Conclusion

Versions **1.2.4** and **1.2.5** represent an important step toward improving system security and ease of use of the API. While version 1.2.4 added fine-grained control over when file uploads are allowed (preventing temporary uploads for fields that require the model to exist), version 1.2.5 introduced a comprehensive restructuring of endpoints to be cleaner and more secure. These updates make the add-on more professional and facilitate its integration with external applications.

---

**Reference Documentation**:
- [General Add-on Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)


## 2026-04-10 – 2026-04-18

**Official Launch of the `Nano3.Kyc` Plugin – A Smart and Integrated Identity Verification (KYC) System for NanoSoft App**

---

### 🚀 Introducing the Package: Why `Nano3.Kyc`?

In the fast‑paced world of digital business, verifying customer and supplier identity (KYC) has become a fundamental requirement for regulatory compliance and building trust. Whether you run an e‑commerce platform, a digital financial institution, or an online marketplace, the need for a flexible and secure system to collect and manage identity documents is indispensable.

Enter the **`Nano3.Kyc`** plugin – the integrated solution from **Nano2Soft**, designed specifically for the **NanoSoft App** environment. It is not just a tool for storing document images; it is a **smart platform for managing the entire identity verification lifecycle**. From defining the required document types (with over 28 ready‑to‑use types) to managing dynamic fields, validity checking, physical copy tracking, and providing complete APIs and backend admin interfaces.

`Nano3.Kyc` is built on the principles of **flexibility and security**, allowing it to adapt to any business model or changing regulatory requirements without complex code modifications.

---

### 🎯 Why `Nano3.Kyc` Was Developed (The Need and the Problem)

Before `Nano3.Kyc`, developers at NanoSoft faced recurring challenges when implementing KYC systems:

| Challenge | `Nano3.Kyc` Solution |
| :--- | :--- |
| **Lack of a unified structure** | The plugin provides a flexible data model (`Document`) designed to accommodate **any document type**, with indexed columns for common fields and a JSON field for specialised fields. |
| **Repeatedly writing validation code** | The `DocumentType` class provides a centralised definition for each document type with ready‑to‑use validation rules (file size, type, expiry date, etc.). |
| **Difficulty managing physical documents** | The plugin includes a complete system for tracking **physical copies received and returned** (`is_physical_submitted`, `physical_received_at`, ...). |
| **Checking document validity over time** | Built‑in functions to check document validity (`isDocumentAcceptable`, `isDocumentExpiring`) that automatically alert when a document expires. |
| **Need for custom admin interfaces** | The plugin provides a **ready‑made backend interface** (lists, forms, filters) and a **complete RESTful API** for integration with mobile apps. |
| **Multiple user types and permissions** | The plugin supports polymorphic relations (`owner`, `verifier`, `subject`), allowing documents to be linked to their owner (individual/corporate) and the verifier, with fine‑grained permissions. |

In short, `Nano3.Kyc` was born to end fragmentation and provide **one centralised solution** for all KYC requirements in NanoSoft projects.

---

### 💎 Core Components and Capabilities of the Package

The `Nano3.Kyc` package consists of several integrated components that work in harmony:

| Component | Function |
| :--- | :--- |
| **`DocumentType`** | **The brain of the plugin**. A central class that defines over **28 ready‑to‑use document types** (passport, ID, bill, business license, etc.) distributed across 6 categories. Contains field schemas and validation rules for each type, and supports loading settings from a `config` file. |
| **`Manager`** | **The operations coordinator**. Provides a unified API for creating, updating, deleting, approving, and rejecting documents. Supports transactions, events, and a testing mode (`is_test_create`). |
| **`Document` (Model)** | **The data model**. The `nano3_kyc_documents` table is designed as the backbone for storing all documents. Uses a rich set of traits to provide search scopes, dropdown options, caching, and advanced filters (`getRecords`). |
| **`Documents` (API Controller)** | **The integration gateway**. Provides OAuth‑protected RESTful endpoints for external applications to interact with the system. All responses follow the unified `Nano.API` structure. |
| **`Documents` (Backend Controller)** | **The administrative user interface**. Provides complete management lists and forms inside the NanoSoft dashboard, with support for advanced filters, search, and sorting. |
| **`DocumentTransformer`** | **The data formatter**. Responsible for transforming model data into JSON compatible with the API, with support for including relations and formatting fields. |
| **Translation files (`lang.php`)** | **Localised user experience**. All texts and notifications are translatable (Arabic is fully provided), ensuring a smooth experience for users. |

---

### ✨ Key Features of the First Release (1.0.0)

This launch represents the culmination of intensive development work and includes a rich set of features:

#### 1. Centralised Definition of Over 28 Document Types

- **6 main categories**: Primary ID, Secondary ID, Proof of Address, Corporate Documents, Ultimate Beneficial Owner (UBO), Additional Verification.
- **Advanced properties per type**: whether it expires (`has_expiry`), maximum age in days (`max_age_days`), trust weight (`verification_weight`), allowed MIME types (`allowed_mime_types`), allowed entities (`allowed_for_entity`), and more.
- **Fully customisable**: any property can be modified, or new types can be added entirely, via the `config/nano3/kyc/document_types.php` file.

#### 2. Dynamic and Intelligent Fields

- **Automatic form generation**: the `getFieldsSchema()` function returns the complete field schema for any document type, making it easy to build dynamic input interfaces.
- **Hybrid storage**: common indexed fields (e.g. `document_number`, `full_name`, `expiry_date`) are stored in separate columns for performance, while specialised fields (e.g. `passport_type`, `religion`) are stored in a single JSON field (`fields_data`).
- **Automatic data mapping**: `mapInputToDocumentData` and `mapDocumentToOutput` simplify working with this hybrid structure.

#### 3. Complete Document Lifecycle

- **Multiple statuses**: `pending`, `verified`, `rejected`, `expired`.
- **Verification operations**: `verifyDocument` updates the status and records the verifier, verification date, and trust score.
- **Rejection with reason**: ability to reject a document and record the reason in `metadata`.

#### 4. Advanced Hard Copy Tracking

- **Precise tracking**: dedicated fields to record when a physical copy is received (`physical_received_at`), returned (`physical_returned_at`), and its type (original/copy).
- **Notes**: `physical_notes` field to document any additional information.

#### 5. Rich RESTful API

- **Comprehensive endpoints**: create, read, update, delete, verify, reject, restore, and statistics.
- **Advanced filtering**: parameters like `document_type`, `status`, `owner_id`, `issue_date` can be passed to filter results.
- **Unified formatting**: all responses follow a uniform structure that is easy to process.

#### 6. Easy‑to‑Use Backend Interface

- **Customisable document list**: sortable and searchable columns, quick filters, and bulk actions.
- **Organised input form**: divided into logical tabs (Basic, Owner, Verification, Physical Copy, Files, Publishing).
- **Smart dropdowns**: dependent dropdowns (e.g. `owner_id` depends on `owner_type`).

---

### 🔧 Looking to the Future

This launch is just the beginning. Nano2Soft is committed to continuing the development of `Nano3.Kyc` to include:

- **Optical Character Recognition (OCR)**: automatic data extraction from document images.
- **Automated document verification**: integration with external services to verify document authenticity.
- **Analytics dashboards**: advanced statistics on document statuses and verification processes.
- **Customisable document templates**: ability to add custom fields via the user interface without programming.

---

### 📚 Reference Documentation

- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [DocumentType Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [Manager Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [Document Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)

---

**Nano3.Kyc** – Developed with love by the **Nano2Soft** team 💙

