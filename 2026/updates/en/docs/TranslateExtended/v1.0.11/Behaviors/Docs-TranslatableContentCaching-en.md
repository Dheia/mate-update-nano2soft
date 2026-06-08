# Comprehensive Documentation of the `TranslatableContentCaching` Behavior (Enhanced Version)

## 1. Introduction

**TranslatableContentCaching** is an advanced behavior for OctoberCMS / WinterCMS, designed to enhance and extend the functionality of the **RainLab.Translate** plugin. This behavior aims to improve performance through caching of translated values, provide easy and fast access to translations via dynamic accessors, and add advanced features for displaying multiple translations in API responses.

While the base RainLab.Translate behavior provides a mechanism for storing and retrieving translations, this behavior builds an integrated layer on top, offering:

- Intelligent caching of translated attributes with automatic invalidation upon updates.
- Dynamic accessors for any translatable field, without the need to write repetitive getter methods.
- The ability to return all translations of a single field in multiple formats (suitable for building multilingual API interfaces).
- Helper functions to obtain alternative language content with their links.
- Full support for the original `translatableUseFallback` property, with fine‑grained control over fallback behaviour.
- Control over enabling/disabling caching globally or per call.
- Handling of JSON‑castable fields with a safe error‑handling mechanism.
- Ability to dynamically set translatable fields after the behavior is initialised.
- Support for passing field names as comma‑separated strings (`'title,content'`) or the word `'all'`/`'*'` in all functions that accept an array of fields.

## 2. Key Benefits and Features

- **Performance improvement**: Caching translations significantly reduces database queries, especially when displaying the same translated content repeatedly.
- **High flexibility**: Support for multiple output formats for translations in APIs, allowing developers to choose the shape that best suits their applications (group by field, group by language, or both).
- **Full integration**: Works seamlessly alongside the RainLab.Translate behavior without conflict, extending its functionality.
- **Time saving**: Dynamic accessors eliminate the need to write individual getter methods for each field.
- **Smart cache management**: Automatic cache invalidation when the model is saved or deleted, with manual invalidation when needed.
- **Alternative link support**: Helper functions to obtain a list of available languages for the current content with their links (useful for SEO and language switching).
- **Full fallback control**: Ability to disable or enable fallback at the model level or within individual calls.
- **JSONable field handling**: Support for decoding JSON‑stored fields with error handling (try‑catch) to prevent application crashes, with error logging in debug mode.
- **Dynamic control of translatable fields**: Ability to modify the list of fields (`transCachedFields`) after behavior initialisation using `getTransCachedFields` and `setTransCachedFields`.
- **Flexible field passing**: Accept fields as a comma‑separated string (`'title,content'`), array, or the value `'all'`/`'*'` in functions such as `setTransCachedFields` and `getTranslationsInFormat`.

## 3. Installation and Setup

### Prerequisites
- OctoberCMS or WinterCMS version 2 or higher.
- **RainLab.Translate** plugin installed and enabled.
- The model to be translated must use the `RainLab.Translate.Behaviors.TranslatableModel` behavior.

### Installation Steps
1. Create the behavior file inside your own plugin, for example:
   `plugins/nano/translateextended/behaviors/TranslatableContentCaching.php`
2. Copy the full behavior code (latest version) into the file.
3. In the model you want to enhance, add the behavior to the `$implement` list:

```php
class Post extends Model
{
    public $implement = [
        '@RainLab.Translate.Behaviors.TranslatableModel',
        'Nano.TranslateExtended.Behaviors.TranslatableContentCaching'
    ];

    // Translatable fields (specific to RainLab.Translate)
    public $translatable = ['title', 'content', 'slug'];

    // Additional fields we want to cache (e.g., JSON fields or relationships)
    public $translatableCaching = ['images', 'seo'];
}
```

## 4. Customisable Properties in the Model

You can customise the translation behaviour by adding the following properties to your model. All property names start with `translatable` to avoid conflicts with other behaviours or model properties.

| Property | Type | Description | Default Value |
|----------|------|-------------|---------------|
| `$translatableCaching` | array | Additional fields (not in `$translatable`) for which you want to cache translations. | `[]` |
| `$translatableApiFields` | array | Fields for which you want to display full translations in the API (via `toArrayWithApiTranslations`). | `[]` |
| `$translatableApiFormat` | string | Output format for multiple translations (see section 5). | `'group_by_field'` |
| `$translatableApiGroupKey` | string | Name of the container key in formats that use a single object. | `'allTranslate'` |
| `$translatableCacheDuration` | int | Cache duration in hours. | `1` |
| `$translatableUseFallback` | bool | Controls whether to use the original value when no translation exists (equivalent to `translatableUseFallback` in the original behaviour). | `true` |
| `$translatableCacheEnabled` | bool | Controls whether caching is enabled globally. | `true` |

## 5. Supported Output Formats for API Translations

The behaviour provides 6 different formats to control the shape of the returned data. You can specify the desired format via the `$translatableApiFormat` property.

### Supported Values (use the strings directly)

| Value | Description |
|-------|-------------|
| `'group_by_field'` | For each field, adds a new field named `[field]_all` containing an object with all translations. |
| `'array'` | For each field, adds a `[field]_all` field as an array of objects `{locale, value}`. |
| `'both'` | Adds both of the above formats under keys `[field]_all_object` and `[field]_all_array`. |
| `'group_by_locale'` | Adds a single object named `allTranslate` (customisable) grouping translations by language. |
| `'group_by_field_under_all'` | Adds an `allTranslate` object grouping translations by field. |
| `'both_grouped'` | Adds the `allTranslate` object (grouped by field) plus the individual `[field]_all` fields. |

### Illustrative Examples with Outputs

Assume we have a `Product` model with translatable fields `name` and `description`, with the following values:
- Original value (x): `"Product 1"` and `"Product description"`
- Arabic translation (ar): `"منتج 1 بالعربية"` and `"وصف المنتج بالعربية"`
- English translation (en): `"Product 1"` and `"Product description"`

When calling `$product->toArrayWithApiTranslations()`, we get the following outputs for each format:

#### a) `'group_by_field'`
```json
{
    "id": 1,
    "name": "Product 1",
    "description": "Product description",
    "name_all": {
        "x": "Product 1",
        "ar": "منتج 1 بالعربية",
        "en": "Product 1"
    },
    "description_all": {
        "x": "Product description",
        "ar": "وصف المنتج بالعربية",
        "en": "Product description"
    }
}
```

#### b) `'array'`
```json
{
    "id": 1,
    "name": "Product 1",
    "description": "Product description",
    "name_all": [
        { "locale": "x", "value": "Product 1" },
        { "locale": "ar", "value": "منتج 1 بالعربية" },
        { "locale": "en", "value": "Product 1" }
    ],
    "description_all": [
        { "locale": "x", "value": "Product description" },
        { "locale": "ar", "value": "وصف المنتج بالعربية" },
        { "locale": "en", "value": "Product description" }
    ]
}
```

#### c) `'both'`
```json
{
    "id": 1,
    "name": "Product 1",
    "description": "Product description",
    "name_all_object": {
        "x": "Product 1",
        "ar": "منتج 1 بالعربية",
        "en": "Product 1"
    },
    "name_all_array": [
        { "locale": "x", "value": "Product 1" },
        { "locale": "ar", "value": "منتج 1 بالعربية" },
        { "locale": "en", "value": "Product 1" }
    ],
    "description_all_object": {
        "x": "Product description",
        "ar": "وصف المنتج بالعربية",
        "en": "Product description"
    },
    "description_all_array": [
        { "locale": "x", "value": "Product description" },
        { "locale": "ar", "value": "وصف المنتج بالعربية" },
        { "locale": "en", "value": "Product description" }
    ]
}
```

#### d) `'group_by_locale'`
```json
{
    "id": 1,
    "name": "Product 1",
    "description": "Product description",
    "allTranslate": {
        "x": {
            "name": "Product 1",
            "description": "Product description"
        },
        "ar": {
            "name": "منتج 1 بالعربية",
            "description": "وصف المنتج بالعربية"
        },
        "en": {
            "name": "Product 1",
            "description": "Product description"
        }
    }
}
```

#### e) `'group_by_field_under_all'`
```json
{
    "id": 1,
    "name": "Product 1",
    "description": "Product description",
    "allTranslate": {
        "name": {
            "x": "Product 1",
            "ar": "منتج 1 بالعربية",
            "en": "Product 1"
        },
        "description": {
            "x": "Product description",
            "ar": "وصف المنتج بالعربية",
            "en": "Product description"
        }
    }
}
```

#### f) `'both_grouped'`
Combines the `group_by_field_under_all` format with the individual `[field]_all` fields (same as `group_by_field`).
```json
{
    "id": 1,
    "name": "Product 1",
    "description": "Product description",
    "name_all": {
        "x": "Product 1",
        "ar": "منتج 1 بالعربية",
        "en": "Product 1"
    },
    "description_all": {
        "x": "Product description",
        "ar": "وصف المنتج بالعربية",
        "en": "Product description"
    },
    "allTranslate": {
        "name": {
            "x": "Product 1",
            "ar": "منتج 1 بالعربية",
            "en": "Product 1"
        },
        "description": {
            "x": "Product description",
            "ar": "وصف المنتج بالعربية",
            "en": "Product description"
        }
    }
}
```

## 6. Basic Usage

### a) Accessing Translated Values in Templates or Code

Once the behaviour is added to the model, you can access any translatable field directly using the normal property. For example, in a Twig template:

```twig
{{ post.title }} {# will return the translated value according to the current language #}
{{ post.content }}
```

No additional method calls are needed; the behaviour creates dynamic accessors for each translatable field.

### b) Getting All Translations of a Specific Field

You can use the `getAllFieldTranslations` method to retrieve all translations of a specific field (including the default language), with control over fallback and caching:

```php
$allTitles = $post->getAllFieldTranslations(
    'title',          // field
    true,             // includeOriginal (includes the original value)
    ['ar', 'en'],     // requested languages
    true,             // useFallback (use fallback)
    false             // useCache (do not use cache)
);
// Result: ['x' => 'Original title', 'ar' => 'العنوان بالعربية', 'en' => 'Title in English']
```

### c) Using the API with Multiple Formats

In your controller, you can use the `toArrayWithApiTranslations` method to obtain a ready‑to‑return JSON array, passing control parameters:

```php
public function show($id)
{
    $post = Post::find($id);
    return response()->json(
        $post->toArrayWithApiTranslations(
            'ar',        // locale
            true,        // useFallback
            false        // useCache (no cache for this call)
        )
    );
}
```

If you want to pass only a specific language:

```php
$data = $post->toArrayWithApiTranslations('ar'); // fetches data with API translations in Arabic
```

### d) Getting Alternate Language Content

You can use `getAlternateLocales` to obtain a list of available languages for this content, along with their URLs (if you provide a URL generator function):

```php
$locales = $post->getAlternateLocales(function($locale) {
    return Url::to('post/' . $post->slug . '?lang=' . $locale);
});
```

Result:
```php
[
    ['code' => 'en', 'url' => 'https://example.com/post/my-post?lang=en', 'default' => true],
    ['code' => 'ar', 'url' => 'https://example.com/post/my-post?lang=ar'],
    ['code' => 'fr', 'url' => 'https://example.com/post/my-post?lang=fr'],
]
```

## 7. Additional Useful Methods (Updated)

| Method | Description |
|--------|-------------|
| `getTranslated($attribute, $default = null, $locale = null, $fallback = null, $useCache = null, $useInBackend = false)` | Fetches a translated value for a specific field. Allows control over fallback, cache usage, and forcing translation in the backend via `$useInBackend`. Improved to automatically set `$fallback = true` when the requested locale is the default locale, ensuring fallback to the original value. |
| `getTranslatedAttributes($locale, $useCache = null)` | Fetches all translated attributes for a specific language from cache (or directly). Improved compatibility with `RainLab.Translate` structure by looking first in `$item->attributes['locale']` then in `$item->locale`. |
| `getAllFieldTranslations($field, $includeDefault = true, $locales = null, $useFallback = null, $useCache = null)` | Fetches all translations of a specific field for all requested languages, with fallback and cache options. |
| `getTranslationsInFormat($fields = null, $format = null, $locales = null, $includeOriginal = true, $useFallback = null, $useCache = null)` | A flexible general method that allows requesting translations in any format, specifying fields and languages, and controlling fallback and cache. **Improved to support passing fields as a comma‑separated string (`'title,content'`) or the word `'all'`/`'*'`.** |
| `addApiTranslations(&$array, $locale = null, $useFallback = null, $useCache = null)` | Adds API translations to an existing array (used internally). |
| `toArrayWithApiTranslations($locale = null, $useFallback = null, $useCache = null)` | Returns the model array with API translations added according to the specified format. |
| `invalidateAllCaches()` | Clears all types of cache for this model (called automatically on save/delete). |
| `enableTranslatableCache()` / `disableTranslatableCache()` | Dynamically enable or disable caching. |
| `isTranslatableCacheEnabled()` | Checks the current cache status. |
| `setUseFallback($useFallback)` / `getUseFallback()` | Sets or reads the current fallback status. |
| `noFallbackLocale()` / `withFallbackLocale()` | Temporarily disables or enables fallback (chainable). |
| `getTransCachedFields()` | Returns the current list of translatable fields (merged from `$translatable` and `$translatableCaching`). |
| `setTransCachedFields($fields = [])` | Dynamically sets the list of translatable fields. Accepts an array, a comma‑separated string, or `'all'`/`'*'`. |
| `transCollectFields()` | Collects translatable fields from the model (called automatically in the constructor, but can be called manually after dynamically changing properties). |
| `processTranslatableJsonableValue($attribute, $value)` | Helper function to decode JSON for fields that are castable to JSON, with error logging in debug mode. |

## 8. Advanced Cache Management

### Invalidating Cache for a Specific Language
The behaviour does not provide a dedicated function to invalidate cache for a specific language, but you can do it manually by constructing the cache key and using `Cache::forget()`.

The cache key for translated attributes of a specific language is built as:
```
{ClassName}_{ModelId}_attributes_{locale}
```
then passed through `md5()`. Example:

```php
use Cache;

$cacheKey = md5(get_class($post) . '_' . $post->id . '_attributes_ar');
Cache::forget($cacheKey);
```

For the field‑specific cache (from `getAllFieldTranslations`), the pattern is more detailed to include fallback and locale information:
```
{ClassName}_{ModelId}_field_{fieldName}_{localesString}_fb{fallback}
```
where `localesString` is an underscore‑separated list of locale codes, and `fb` indicates the fallback state (`default`, `1`, `0`). Example for a cache containing languages `ar_en` with fallback enabled:
```php
$cacheKey = md5(get_class($post) . '_' . $post->id . '_field_title_ar_en_fb1');
Cache::forget($cacheKey);
```
If you want to clear all variants of a specific field’s cache, you could clear the base key without suffixes, but that may not clear all variations. For simplicity, calling `invalidateAllCaches()` clears all cache associated with this model.

## 9. Complete Model Example with All Settings

```php
<?php namespace Acme\Blog\Models;

use Model;

class Product extends Model
{
    public $implement = [
        '@RainLab.Translate.Behaviors.TranslatableModel',
        'Nano.TranslateExtended.Behaviors.TranslatableContentCaching'
    ];

    public $translatable = [
        'name',
        'description',
        'meta_title',
        'meta_description'
    ];

    // Additional fields to cache (e.g., JSON)
    public $translatableCaching = [
        'images',
        'specs'
    ];

    // API fields for which we want full translations
    public $translatableApiFields = [
        'name',
        'description',
        'meta_title',
        'meta_description'
    ];

    // API format
    public $translatableApiFormat = 'group_by_field_under_all';

    // Custom container key for the grouped object
    public $translatableApiGroupKey = 'translations';

    // Cache duration (2 hours)
    public $translatableCacheDuration = 2;

    // Disable fallback (optional)
    public $translatableUseFallback = false;

    // Enable caching (default true)
    public $translatableCacheEnabled = true;

    // ... rest of model definition (table, relationships, etc.)
}
```

## 10. Dynamic Control of Translatable Fields

In some scenarios, you may need to modify the list of translatable fields after the behaviour has been initialised (e.g., based on a specific context). The behaviour provides two methods for this:

### `getTransCachedFields()`
Returns the current array of translatable fields (merged from `$translatable` and `$translatableCaching`).

### `setTransCachedFields($fields)`
Sets a new list of translatable fields. The method accepts:
- An array of field names.
- A comma‑separated string (e.g., `'title,content'`).
- The value `'all'` or `'*'` to automatically fetch all translatable fields (by calling `transCollectFields`).
- An empty value (will also call `transCollectFields`).

**Example:**
```php
$product = Product::find(1);
// Set specific fields dynamically
$product->setTransCachedFields('name,description,specs');
// Now only these fields will be handled in translation operations
$translations = $product->getTranslationsInFormat();
```

## 11. Improvements to JSON and Jsonable Field Handling

A helper function `processTranslatableJsonableValue` has been added that decodes values stored as JSON if the field is defined as `jsonable` in the model (via the `$jsonable` property or `isJsonable` function). If a JSON error occurs (e.g., invalid format), the error is logged in the trace log (`trace_log`) only when debug mode is enabled (`app.debug = true`), without crashing the application.

This ensures safe handling of data stored as JSON, such as `images`, `specs`, or any flexible fields.

## 12. The `getTranslationsInFormat` Method (Updated)

The general method has been improved to accept additional parameters for controlling fallback and cache, and now supports passing fields as a comma‑separated string.

### Signature:
```php
public function getTranslationsInFormat(
    $fields = null,
    $format = null,
    $locales = null,
    $includeOriginal = true,
    $useFallback = null,
    $useCache = null
)
```

### Parameters:
- `$fields`: array of field names, comma‑separated string (`'title,content'`), `'all'`/`'*'`, or `null` (uses all translatable fields from `transCachedFields`).
- `$format`: desired format. If `null`, the model’s default format (`$translatableApiFormat`) is used.
- `$locales`: array of locale codes (e.g., `['ar', 'en']`). If `null`, all active languages are used.
- `$includeOriginal`: boolean, if `true` includes the original value under the key `'x'`.
- `$useFallback`: boolean, controls fallback usage (`null` uses the global setting).
- `$useCache`: boolean, controls cache usage (`null` uses the global setting).

### Return value:
Array containing translations in the requested format.

### Usage examples:

```php
// Use all default settings (all fields, default format, all languages)
$translations = $product->getTranslationsInFormat();

// Single field, group_by_field format, disable fallback
$translations = $product->getTranslationsInFormat(
    'name',
    'group_by_field',
    null,
    true,
    false,   // useFallback
    true     // useCache
);

// Multiple fields as a string, group_by_locale format, specific languages, no cache
$translations = $product->getTranslationsInFormat(
    'name,description',
    'group_by_locale',
    ['ar', 'en'],
    true,    // includeOriginal
    true,    // useFallback
    false    // useCache
);

// Get all fields translations in array format, without original value
$translations = $product->getTranslationsInFormat(null, 'array', null, false);
```

#### Example output (for `['name', 'description'], 'group_by_field_under_all', ['ar', 'en']` with `includeOriginal = true`):
```php
[
    'allTranslate' => [
        'name' => [
            'x' => 'Product 1',
            'ar' => 'منتج 1 بالعربية',
            'en' => 'Product 1'
        ],
        'description' => [
            'x' => 'Product description',
            'ar' => 'وصف المنتج بالعربية',
            'en' => 'Product description'
        ]
    ]
]
```

## 13. Additional Examples for Fallback and Cache Control

### a) Calling `getTranslated` with temporary fallback disabled

```php
$value = $product->getTranslated('name', null, 'ar', false); // no fallback even if enabled globally
```

### b) Disabling cache for a specific call

```php
$attributes = $product->getTranslatedAttributes('en', false); // fetches directly from the database
```

### c) Using `getAllFieldTranslations` with fallback and cache options

```php
$allNames = $product->getAllFieldTranslations(
    'name',
    true,
    ['ar', 'en', 'fr'],
    true,   // useFallback
    false   // useCache (no cache)
);
```

### d) Dynamically checking cache and fallback status

```php
// Temporarily disable cache for all subsequent calls on this model
$product->disableTranslatableCache();

// Enable fallback
$product->withFallbackLocale();

// Call a function that will be affected by these settings
$data = $product->toArrayWithApiTranslations();
```

### e) Forcing translation in the backend environment

By default, translation is automatically disabled in the backend to avoid interference during editing. However, you can override this behaviour using the `$useInBackend = true` parameter:

```php
// Even if the code is executed in the backend, the translated value will be returned
$value = $product->getTranslated('title', null, 'ar', true, true, true);
```

## 14. Important Notes

- **Translation in the backend**: Translation is automatically disabled in the backend interface to avoid interference during editing, unless `$useInBackend = true` is passed.
- **Compatibility with other plugins**: The behaviour does not conflict with any other plugins and can be used with any model that uses RainLab.Translate. All internal properties and methods are prefixed with `trans` to ensure no conflicts.
- **Cache key uniqueness**: The cache key is built based on the class name, model ID, and cache type, ensuring no conflicts between different models.
- **Automatic cache invalidation**: When the model is saved or deleted, all associated cache keys are automatically cleared to ensure data freshness.
- **JSONable field handling**: JSON decoding is performed automatically with error handling (try‑catch), preventing application crashes in case of invalid JSON.
- **Improved fallback for default locale**: The `getTranslated` method has been modified to automatically set `$fallback = true` when the requested locale is the default locale, ensuring fallback to the original value stored in `attributes` if no translation exists. This fixes unexpected behaviour in previous versions.
- **Compatibility with RainLab.Translate data structure**: `getTranslatedAttributes` has been improved to search for the translation using `$item->attributes['locale']` first (the way RainLab.Translate stores the language), then `$item->locale` as a fallback. This ensures compatibility with different versions of the plugin.

## 15. Dynamically Registering Models (`registerModel` method)

The behaviour provides a static method `registerModel` that allows registering any model to use this behaviour without modifying the model’s original code. It can be called from the `boot` method in your plugin’s `Plugin.php`.

### Signature:
```php
public static function registerModel($modelClass, array $dynamicProperties = [])
```

### Parameters:
- `$modelClass`: fully qualified class name of the model.
- `$dynamicProperties`: array of dynamic properties to add to the model (e.g., `translatableApiFields`, `translatableApiFormat`, etc.).

### Example:
```php
// In Plugin.php
public function boot()
{
    \Nano\TranslateExtended\Behaviors\TranslatableContentCaching::registerModel(
        \Tss\Inventory\Models\Product::class,
        [
            'translatableApiFormat' => 'group_by_field_under_all',
            'translatableApiGroupKey' => 'translations',
            'translatableCacheDuration' => 2
        ]
    );
}
```

**Improvements in the current version:**
- The function checks whether the behaviour has already been added to the model (`$model->isClassExtendedWith($behaviorClass)`) before attempting to add it.
- It checks for the existence of dynamic properties before adding them using the helper function `isPropertyExists`, preventing redefinition of existing properties.
