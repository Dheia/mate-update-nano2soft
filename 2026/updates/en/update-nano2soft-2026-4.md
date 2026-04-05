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

