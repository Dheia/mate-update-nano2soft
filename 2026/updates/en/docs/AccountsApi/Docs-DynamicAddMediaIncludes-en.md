# Documentation of `DynamicAddMediaIncludes` Behavior

**Version:** 1.1.1 (`Nano.AccountsApi` plugin)  
**Path:** `plugins/nano/accountsapi/behaviors/DynamicAddMediaIncludes.php`  
**Author:** NanoSoft Development Team  
**License:** Proprietary to NanoSoft  
**Last updated:** 2026-05-30

---

## Table of Contents

1. [Introduction](#introduction)
2. [Purpose of the Behavior](#purpose-of-the-behavior)
3. [Requirements and Dependencies](#requirements-and-dependencies)
4. [Structure and Properties](#structure-and-properties)
5. [Provided Include Methods](#provided-include-methods)
6. [Mechanism for Controlling Metadata](#mechanism-for-controlling-metadata)
7. [Mechanism for Extracting Transformer-Specific Settings](#mechanism-for-extracting-transformer-specific-settings)
8. [Integration with Transformers in `Nano.AccountsApi`](#integration-with-transformers-in-nanoaccountsapi)
9. [Configuration](#configuration)
10. [Practical Examples](#practical-examples)
11. [Troubleshooting](#troubleshooting)
12. [References](#references)

---

## Introduction

`DynamicAddMediaIncludes` is a custom Behavior designed to dynamically add multimedia `includes` (images, files, videos, audio recordings, introductory files) to any Transformer in the `Nano.AccountsApi` plugin. This Behavior aims to unify the way file data associated with accounting models (e.g., receipts, journal entries, accounts, partners, etc.) is included in API responses, while providing full flexibility to control metadata return for each file type.

The Behavior relies on helper functions from `Nano\Api\Helpers\ApiHelper` (such as `image()`, `images()`, `files()`) to format data consistently with other NanoSoft plugins (e.g., `Nano.LocationApi`). It also supports reading settings from `config.php` and environment variables, as well as overriding them via request parameters.

---

## Purpose of the Behavior

- **Add media includes to accounting Transformers** without manually modifying each Transformer.
- **Unify media response format** across all Transformers (same format as `ProductTransformer` in `Nano.ShopApi`).
- **Fine-grained control over metadata** for each include individually (global, per‑transformer, or per request).
- **Improve API performance** by not returning metadata unless explicitly requested.
- **Easy extensibility** by adding new media types (e.g., `videos`, `audios`) once relationships are enabled in the models and the methods are uncommented in the Behavior.

---

## Requirements and Dependencies

- **`Nano.API` plugin** – provides `ApiHelper` functions and the base Transformer structure.
- **`Tss.Accounts` plugin (version 1.0.39 or later)** – provides `attachOne` and `attachMany` relationships on accounting models.
- **`System\Models\File`** – built into NanoSoft APP.
- **PHP 8.0 or later**.

---

## Structure and Properties

The Behavior is located at `Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes` and extends `October\Rain\Extension\ExtensionBase`.

### Main Properties

| Property | Type | Description |
| :--- | :--- | :--- |
| `$transformer` | `Transformer` | Reference to the associated Transformer. |
| `$isMateData` | `bool` | (Internal) temporary storage for global metadata settings. |

### Constructor (`__construct`)

- Checks for the existence of `availableIncludes` and adds `image`, `images`, `files` to it (others like `videos`, `audios`, `book_intro` can be activated by modifying the array).
- Adds some of them to `defaultIncludes` according to the `default_media_includes` setting (from config or environment variable).
- Calls `getDefaultIncludeForConfig` to obtain transformer‑specific settings if they exist.

### Internal Helper Methods

| Method | Description |
| :--- | :--- |
| `shouldReturnMateData()` | Determines whether metadata is required globally (from request or settings). |
| `getMateDataForInclude(string $includeName)` | Determines whether metadata is required for a specific include (overrides the global setting). |
| `getDefaultIncludeForConfig($class, $defaultValue, $prefixConfig, $suffixConfig)` | Extracts transformer‑specific settings based on the class name. |
| `pascalToSnake(string $input)` | Converts a transformer name from `PascalCase` to `snake_case`. |

---

## Provided Include Methods

Each method represents an include that can be requested via the `include` parameter in the API. All of them return either a `Primitive` (for single items) or a `Collection` (for multiple items) from Fractal.

| Method | Include | Resource Type | Expected relationship on model | Default value on error |
| :--- | :--- | :--- | :--- | :--- |
| `includeImage($model)` | `image` | `Primitive` | `attachOne 'image'` | `null` |
| `includeImages($model)` | `images` | `Primitive` (returns array) | `attachMany 'images'` | `[]` |
| `includeFiles($model)` | `files` | `Primitive` (returns array) | `attachMany 'files'` | `[]` |
| `includeVideos($model)` | `videos` | `Primitive` | `attachMany 'videos'` (disabled by default) | `[]` |
| `includeAudios($model)` | `audios` | `Primitive` | `attachMany 'audios'` (disabled) | `[]` |
| `includeBookIntro($model)` | `book_intro` | `Primitive` | `attachOne 'book_intro'` (disabled) | `null` |

**Note:** The methods `includeImages`, `includeFiles`, `includeVideos`, `includeAudios` return a `Primitive` containing an array, rather than a `Collection`. This ensures a unified format with the `ApiHelper` methods that already return ready‑to‑use arrays.

### How the Methods Work

1. They check the `$model` for validity and that the relationship method exists (`method_exists($model, 'image')` or `isset($model->images)`).
2. They call the appropriate helper method from `ApiHelper` (`image()`, `images()`, `files()`) passing the `$isMate` parameter obtained from `getMateDataForInclude()`.
3. In case of an exception, they log it (only in development environment) and return the safe default value (`null` or `[]`).
4. The result is returned inside a `Primitive`.

**Why use `ApiHelper` instead of the original Transformer methods?**  
To ensure a unified data format (e.g., `original`, `small`, `medium`, `thumb` for images) and to benefit from resize support and other features already present in `ApiHelper`.

---

## Mechanism for Controlling Metadata

Metadata includes file information such as `id`, `title`, `description`, `file_name`, `file_size`, `extension`, `content_type`, `path`, `created_at`, `updated_at`. Its return can be controlled through three layers (the first has priority):

### 1. Include‑specific request parameter

By sending a parameter with the same name as the include that contains `mate_data`:

```http
GET /api/v1/accounts/catchreceipts/1?include=image,files&image[mate_data]=1
```

### 2. Global request parameter `is_mate_data_media`

```http
GET /api/v1/accounts/catchreceipts/1?include=image&is_mate_data_media=1
```

### 3. Transformer‑specific settings (via `config.php`)

Example:

```php
'catch_receipts_v2' => [
    'is_mate_data_image' => true,
    'is_mate_data_files' => false,
],
```

### 4. Global setting `is_mate_data_media` in `config.php` or environment variable

```php
'is_mate_data_media' => env('NANO_ACCOUNTSAPI_IS_MATE_DATA_MEDIA', false),
```

**Priority order:**  
include‑specific > global request > transformer‑specific > global settings.

---

## Mechanism for Extracting Transformer‑Specific Settings

The `getDefaultIncludeForConfig` function generates a setting key based on the transformer’s class name. The steps are:

1. Extract the base class name (e.g., `CatchReceiptTransformer`).
2. Remove the `Transformer` suffix (becomes `CatchReceipt`).
3. Convert the name to `snake_case` (becomes `catch_receipt`).
4. Attempt to retrieve the setting from `config('nano.accountsapi::config.' . $name . '.include')` or `... . 'is_mate_data_image'`.
5. Try adding or removing the final `s` (to distinguish singular/plural).
6. Try removing underscores (`_`).
7. If no setting is found, return the default value.

**Example:**  
Transformer named `CatchReceiptTransformer` → looks for:
- `nano.accountsapi::config.catch_receipt.include`
- `nano.accountsapi::config.catch_receipt.is_mate_data_image`
- then `nano.accountsapi::config.catch_receipts.include` (adding `s`)
- then `nano.accountsapi::config.catchreceipt.include` (without underscores)
- etc.

This allows precise customisation for each document type.

---

## Integration with Transformers in `Nano.AccountsApi`

In the `Plugin.php` file of `Nano.AccountsApi`, there is a function `extendAccountsApiTransformersWithMedia()` that does the following:

```php
protected function extendAccountsApiTransformersWithMedia()
{
    $transformers = [
        \Nano\AccountsApi\Transformers\AccountTransformer::class,
        \Nano\AccountsApi\Transformers\ReferenceTransformer::class,
        // ... all other Transformers
    ];

    foreach ($transformers as $transformerClass) {
        $transformerClass::extend(function ($transformer) {
            if (!$transformer->isClassExtendedWith('Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes')) {
                $transformer->extendClassWith('Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes');
            }
        });
    }
}
```

This ensures that all the following Transformers have the Behavior and support `image`, `images`, `files` in `availableIncludes`:

- `AccountTransformer`
- `ReferenceTransformer`
- `GroupTransformer`
- `CostCenterTransformer`
- `PeriodTransformer`
- `BoxeTransformer`
- `BankTransformer`
- `PartnerTransformer`
- `TransactionHeaderTransformer`
- `TransactionsDetailTransformer`
- `BondsDayTransformer`
- `CatchReceiptTransformer`
- `PayReceiptTransformer`
- `CreditNoteTransformer`
- `DebitNoteTransformer`
- `TransferTransformer`
- `ExpenseTransformer`
- `IncomeTransformer`
- `OpeningBalanceTransformer`
- `BindingBondTransformer`
- `RelayTransformer`
- `Bonds2DayTransformer`
- `CreditDeliveryTransformer`
- as well as the base transformer (`\Nano\AccountsApi\Transformers\Transformer`).

---

## Configuration

The Behaviour can be customised via the plugin’s configuration file `config/nano/accountsapi/config.php` (published using `php artisan config:publish Nano.AccountsApi`).

### Global Settings

| Key | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `default_media_includes` | `string\|array\|null` | `null` | List of `includes` added to `defaultIncludes`. Examples: `'image,files'` or `['image', 'files']`. |
| `is_mate_data_media` | `bool` | `false` | Return metadata for media globally. |

### Transformer‑specific Settings (example for `catch_receipts_v2`)

You can add any of the following keys under the transformer’s name converted to `snake_case`:

```php
'catch_receipts_v2' => [
    // other settings (order_by, transactions_type, is_allow_add...)
    'include' => ['image', 'files'],   // to override default_media_includes for this type
    'is_mate_data_image' => true,      // enable metadata for image
    'is_mate_data_files' => false,     // disable metadata for files
],
```

### Environment Variables (`.env`)

Environment variables can be used to override any of the settings:

```
NANO_ACCOUNTSAPI_DEFAULT_MEDIA_INCLUDES=image,files
NANO_ACCOUNTSAPI_IS_MATE_DATA_MEDIA=true
NANO_ACCOUNTSAPI_CATCH_RECEIPTS_IS_MATE_DATA_IMAGE=false
```

---

## Practical Examples

### 1. Request an Account with Only the Main Image (no metadata)

```http
GET /api/v1/accounts/accounts/2-2-1231010001?include=image
```

**Response:**

```json
{
  "data": {
    "id": 2,
    "code": "2-2-1231010001",
    "name": "Customers",
    "image": {
      "original": "https://domain.com/storage/accounts/customer.jpg",
      "small": "https://domain.com/storage/accounts/customer_small.jpg"
    }
  }
}
```

### 2. Request a Catch Receipt with Image and Files, and Metadata for the Image Only

```http
GET /api/v1/accounts/catchreceipts/1?include=image,files&image[mate_data]=1
```

**Response:**

```json
{
  "data": {
    "id": 1,
    "code": "CR-001",
    "image": {
      "original": "https://domain.com/receipts/1.jpg",
      "mate_data": {
        "id": 101,
        "file_name": "receipt.jpg",
        "file_size": 204800
      }
    },
    "files": [
      {
        "file_name": "invoice.pdf",
        "path": "https://domain.com/receipts/invoice.pdf"
      }
    ]
  }
}
```

### 3. Set `default_media_includes` to Automatically Fetch Image and Files for All Requests

In `.env`:

```
NANO_ACCOUNTSAPI_DEFAULT_MEDIA_INCLUDES=image,files
```

Then any request to `/accounts/catchreceipts` will return the image and files without needing to write `include`.

### 4. Disable Metadata for the Image in a Specific Transformer Only

In `config/nano/accountsapi/config.php`:

```php
'catch_receipts_v2' => [
    'is_mate_data_image' => false,
],
```

Even if the client sends `is_mate_data_media=1`, metadata for the image will not be returned for catch receipts.

### 5. Add a Custom Include (e.g., `videos`) for a Specific Transformer

1. Enable the `videos` relationship on the model (ensure `attachMany 'videos'` exists).
2. In `DynamicAddMediaIncludes.php`, add `'videos'` to the `$mediaIncludes` array in the constructor.
3. Add the `includeVideos` method (already exists but commented out).
4. Clear the cache.

---

## Troubleshooting

| Problem | Likely Cause | Solution |
| :--- | :--- | :--- |
| `include=image` does not return image data | The `image` relationship does not exist on the model | Ensure the model has `$attachOne['image']` or that it was added dynamically. |
| Error `Call to undefined method ApiHelper::image()` | `Nano.API` plugin is not installed or outdated | Install `Nano.API` and ensure the helper methods exist. |
| Metadata does not appear despite `is_mate_data_media=1` | Possibly disabled by the transformer‑specific setting | Check `is_mate_data_image` or `is_mate_data_files` in the config. |
| `include=images` returns an empty array | No images have been uploaded | Upload images via the backend or API. |
| Error `Class 'Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes' not found` | The Behavior is not in the correct path | Ensure the file exists at `plugins/nano/accountsapi/behaviors/DynamicAddMediaIncludes.php`. |
| `getDefaultIncludeForConfig` does not return expected settings | The transformer name does not match the key in the config | Check the conversion to `snake_case` and try adding a key with the full name. |

---

## Conclusion

The `DynamicAddMediaIncludes` Behavior is a powerful and flexible tool for unifying multimedia management in the accounting API. Thanks to its modular design and reliance on `ApiHelper` methods, it ensures a consistent format with other NanoSoft plugins (`Nano.LocationApi`, `Nano.ShopApi`). Its multi‑level control (global, transformer‑specific, request‑level) gives developers precise control over API responses, improving performance and reducing unnecessary data size.

To get the most out of this Behavior, it is recommended to read the documentation of the `Tss.Accounts` plugin (especially the multimedia section) and the `Nano.API` documentation for a deeper understanding of the `ApiHelper` functions.
