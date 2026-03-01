## Integrated Documentation for TranslatableContentCaching Behavior

### 1. Introduction

**TranslatableContentCaching** is an advanced behavior for the NanoSoft System, designed to enhance and extend the functionality of the **RainLab.Translate** plugin. This behavior aims to improve performance through caching of translated values, provide easy and fast access to translations via dynamic accessors, and add advanced features for displaying multiple translations in API interfaces.

While the core RainLab.Translate behavior provides a mechanism for storing and retrieving translations, this behavior builds an integrated layer on top of it, offering:

- Intelligent caching for translated attributes with automatic invalidation upon update.
- Dynamic accessors for any translatable field, eliminating the need for writing repetitive getter functions.
- Ability to return all translations of a single field in multiple formats (suitable for building multilingual APIs).
- Helper functions to retrieve alternate language versions of content with their links.
- Full support for the `translatableUseFallback` property from the original behavior, with fine-grained control over fallback.
- Control over enabling/disabling caching globally or per call.
- Handling of Jsonable fields with a safe error-handling mechanism.

### 2. Key Benefits and Features

- **Performance Improvement**: Storing translations in the cache significantly reduces database queries, especially when repeatedly displaying the same translated content.
- **High Flexibility**: Supports multiple output formats for translations in APIs, allowing the developer to choose the most suitable shape for their applications (group by field, by locale, or both).
- **Seamless Integration**: Works alongside the RainLab.Translate behavior without conflict, extending its functionality.
- **Time Saving**: Dynamic accessors eliminate the need to write individual getter functions for each field.
- **Intelligent Cache Management**: Automatic cache clearing when the model is updated or deleted, with manual clearing available when needed.
- **Alternate Links Support**: Helper functions to get a list of available languages for the current content with their links (useful for SEO and language switching).
- **Full Fallback Control**: Ability to disable or enable fallback at the model level or within individual calls.
- **JSONable Field Handling**: Support for decoding fields stored as JSON with error handling (try-catch) to prevent application crashes.

### 3. Installation and Setup

#### Prerequisites
- NanoSoft System version 3 or higher (or version 2 with Traits support).
- **RainLab.Translate** plugin installed and activated.
- The model to be translated must use the `RainLab.Translate.Behaviors.TranslatableModel` behavior.

#### Installation Steps
1. Create the behavior file within your own plugin, for example:
   `plugins/nano/translateextended/behaviors/TranslatableContentCaching.php`
2. Copy the complete behavior code (latest version) into the file.
3. In the model you wish to enhance, add the behavior to the `$implement` array:

```php
class Post extends Model
{
    public $implement = [
        '@RainLab.Translate.Behaviors.TranslatableModel',
        'Nano.TranslateExtended.Behaviors.TranslatableContentCaching'
    ];

    // Translatable fields (specific to RainLab.Translate)
    public $translatable = ['title', 'content', 'slug'];

    // Additional fields we want to cache (e.g., JSON fields or relations)
    public $translatableCaching = ['images', 'seo'];
}
```

### 4. Customizable Properties in the Model

You can customize the translation behavior by adding the following properties to your model. All property names start with `translatable` to avoid conflicts with other behaviors or model properties.

| Property | Type | Description | Default Value |
|---------|------|-------------|---------------|
| `$translatableCaching` | array | Additional fields (not in `$translatable`) whose translations you want to cache. | `[]` |
| `$translatableApiFields` | array | Fields for which you want to display full translations in the API (via `toArrayWithApiTranslations`). | `[]` |
| `$translatableApiFormat` | string | Output format for multiple translations (see section 5). | `'group_by_field'` |
| `$translatableApiGroupKey` | string | The grouping key name for formats that use a single object. | `'allTranslate'` |
| `$translatableCacheDuration` | int | Cache duration in hours. | `1` |
| `$translatableUseFallback` | bool | Controls whether to use the original value when a translation is missing (equivalent to `translatableUseFallback` in the original behavior). | `true` |
| `$translatableCacheEnabled` | bool | Controls whether caching is enabled globally. | `true` |

### 5. Supported Output Formats for API Translations

The behavior provides 6 different formats to control the shape of the returned data. You can specify the desired format via the `$translatableApiFormat` property.

#### Supported Values (Use the strings directly)

| Value | Description |
|--------|-------------|
| `'group_by_field'` | For each field, adds a new field named `[field]_all` containing an object with all translations. |
| `'array'` | For each field, adds a `[field]_all` field as an array of objects `{locale, value}`. |
| `'both'` | Adds both previous formats with keys `[field]_all_object` and `[field]_all_array`. |
| `'group_by_locale'` | Adds a single object named `allTranslate` (customizable) that groups translations by locale. |
| `'group_by_field_under_all'` | Adds an `allTranslate` object that groups translations by field. |
| `'both_grouped'` | Adds an `allTranslate` object (grouped by field) along with individual `[field]_all` fields. |

#### Illustrative Examples with Outputs

Assume we have a `Product` model containing the translatable fields `name` and `description`, with the following values:
- Original value (x): `"منتج 1"` and `"وصف المنتج"`
- Arabic translation (ar): `"منتج 1 بالعربية"` and `"وصف المنتج بالعربية"`
- English translation (en): `"Product 1"` and `"Product description"`

When calling `$product->toArrayWithApiTranslations()`, we get the following outputs for each format:

---

##### a) `'group_by_field'`
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "name_all": {
        "x": "منتج 1",
        "ar": "منتج 1 بالعربية",
        "en": "Product 1"
    },
    "description_all": {
        "x": "وصف المنتج",
        "ar": "وصف المنتج بالعربية",
        "en": "Product description"
    }
}
```

##### b) `'array'`
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "name_all": [
        { "locale": "x", "value": "منتج 1" },
        { "locale": "ar", "value": "منتج 1 بالعربية" },
        { "locale": "en", "value": "Product 1" }
    ],
    "description_all": [
        { "locale": "x", "value": "وصف المنتج" },
        { "locale": "ar", "value": "وصف المنتج بالعربية" },
        { "locale": "en", "value": "Product description" }
    ]
}
```

##### c) `'both'`
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "name_all_object": {
        "x": "منتج 1",
        "ar": "منتج 1 بالعربية",
        "en": "Product 1"
    },
    "name_all_array": [
        { "locale": "x", "value": "منتج 1" },
        { "locale": "ar", "value": "منتج 1 بالعربية" },
        { "locale": "en", "value": "Product 1" }
    ],
    "description_all_object": {
        "x": "وصف المنتج",
        "ar": "وصف المنتج بالعربية",
        "en": "Product description"
    },
    "description_all_array": [
        { "locale": "x", "value": "وصف المنتج" },
        { "locale": "ar", "value": "وصف المنتج بالعربية" },
        { "locale": "en", "value": "Product description" }
    ]
}
```

##### d) `'group_by_locale'`
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "allTranslate": {
        "x": {
            "name": "منتج 1",
            "description": "وصف المنتج"
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

##### e) `'group_by_field_under_all'`
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "allTranslate": {
        "name": {
            "x": "منتج 1",
            "ar": "منتج 1 بالعربية",
            "en": "Product 1"
        },
        "description": {
            "x": "وصف المنتج",
            "ar": "وصف المنتج بالعربية",
            "en": "Product description"
        }
    }
}
```

##### f) `'both_grouped'`
Combines the `group_by_field_under_all` format with individual `[field]_all` fields (in the same format as `group_by_field`).
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "name_all": {
        "x": "منتج 1",
        "ar": "منتج 1 بالعربية",
        "en": "Product 1"
    },
    "description_all": {
        "x": "وصف المنتج",
        "ar": "وصف المنتج بالعربية",
        "en": "Product description"
    },
    "allTranslate": {
        "name": {
            "x": "منتج 1",
            "ar": "منتج 1 بالعربية",
            "en": "Product 1"
        },
        "description": {
            "x": "وصف المنتج",
            "ar": "وصف المنتج بالعربية",
            "en": "Product description"
        }
    }
}
```

### 6. Basic Usage

#### a) Accessing Translated Values in Templates or Code

When the behavior is added to the model, you can access any translatable field directly using the regular property. For example, in a Twig template:

```twig
{{ post.title }} {# Will return the translated value according to the current locale #}
{{ post.content }}
```

No need to call any additional functions; the behavior creates dynamic accessors for each translatable field.

#### b) Getting All Translations of a Specific Field

You can use the `getAllFieldTranslations` function to fetch all translations of a specific field (including the default language) with control over fallback and cache usage:

```php
$allTitles = $post->getAllFieldTranslations(
    'title',          // Field
    true,             // includeOriginal (include the original value)
    ['ar', 'en'],     // Requested locales
    true,             // useFallback (use fallback)
    false             // useCache (do not use cache)
);
// Result: ['x' => 'Original title', 'ar' => 'Arabic title', 'en' => 'Title in English']
```

#### c) Using the API with Multiple Formats

In your controller, you can use the `toArrayWithApiTranslations` function to get an array ready to be returned as JSON, passing control parameters:

```php
public function show($id)
{
    $post = Post::find($id);
    return response()->json(
        $post->toArrayWithApiTranslations(
            'ar',        // Locale
            true,        // useFallback
            false        // useCache (no cache for this call)
        )
    );
}
```

If you want to pass a specific locale only:

```php
$data = $post->toArrayWithApiTranslations('ar'); // Fetches data with API translations in Arabic
```

#### d) Getting Alternate Locales for Content

You can use `getAlternateLocales` to obtain a list of available languages for this content, along with their links (if you provide a URL generation function):

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

### 7. Additional Useful Functions (Updated)

| Function | Description |
|--------|-------------|
| `getTranslated($attribute, $default = null, $locale = null, $fallback = null, $useCache = null, $useInBackend = false)` | Fetches a translated value for a specific field. Allows control over fallback, cache usage, and forcing translation in the backend via `$useInBackend`. |
| `getTranslatedAttributes($locale, $useCache = null)` | Fetches all translated attributes for a given locale from the cache (or directly). |
| `getAllFieldTranslations($field, $includeDefault = true, $locales = null, $useFallback = null, $useCache = null)` | Fetches all translations of a specific field for all requested locales, with fallback and cache options. |
| `getTranslationsInFormat($fields = null, $format = null, $locales = null, $includeOriginal = true, $useFallback = null, $useCache = null)` | A flexible general function that allows retrieving translations in any format, specifying fields, locales, and controlling fallback and cache. |
| `addApiTranslations(&$array, $locale = null, $useFallback = null, $useCache = null)` | Adds API translations to an existing array (used internally). |
| `toArrayWithApiTranslations($locale = null, $useFallback = null, $useCache = null)` | Returns the model's array with API translations added according to the specified format. |
| `invalidateAllCaches()` | Clears all cache types for this model (called automatically on save/delete). |
| `enableTranslatableCache()` / `disableTranslatableCache()` | To enable or disable caching dynamically. |
| `isTranslatableCacheEnabled()` | To check the cache status. |
| `setUseFallback($useFallback)` / `getUseFallback()` | To set or read the current fallback status. |
| `noFallbackLocale()` / `withFallbackLocale()` | To temporarily disable or enable fallback (chainable). |

### 8. Advanced Cache Management

#### Invalidating Cache for a Specific Locale
The behavior does not provide a dedicated general function to invalidate cache for a specific locale, but you can do it manually by constructing the cache key and using `Cache::forget()`.

The cache key for translated attributes for a specific locale is built as:
```
{ClassName}_{ModelId}_attributes_{locale}
```
Then passed through `md5()`. Example:

```php
use Cache;

$cacheKey = md5(get_class($post) . '_' . $post->id . '_attributes_ar');
Cache::forget($cacheKey);
```

For the specific field cache (from `getAllFieldTranslations`), the pattern is more detailed to include fallback information and requested locales:
```
{ClassName}_{ModelId}_field_{fieldName}_{localesString}_fb{fallback}
```
Where `localesString` is the concatenation of locale codes with underscores, and `fb` indicates the fallback status (`default`, `1`, `0`). Example for a cache containing the locales `ar_en` with fallback enabled:
```php
$cacheKey = md5(get_class($post) . '_' . $post->id . '_field_title_ar_en_fb1');
Cache::forget($cacheKey);
```
If you want to clear all cache forms for a specific field, you can clear the base key without additions:
```php
$cacheKeyBase = md5(get_class($post) . '_' . $post->id . '_field_title');
Cache::forget($cacheKeyBase); // This may not clear all variants, so it's better to clear keys precisely.
```

To simplify, you can call `invalidateAllCaches()` which clears all cache associated with this model.

### 9. Integrated Example Model with All Settings

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

    // Additional fields for caching (e.g., JSON)
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

    // Custom key for the grouping object
    public $translatableApiGroupKey = 'translations';

    // Cache duration (2 hours)
    public $translatableCacheDuration = 2;

    // Disable fallback (optional)
    public $translatableUseFallback = false;

    // Enable cache (default true)
    public $translatableCacheEnabled = true;

    // ... Rest of model definitions (tables, relations, etc.)
}
```

### 10. Important Notes

- **Translation in the Backend**: Translation is automatically disabled in the backend interface to avoid interference during editing, unless `$useInBackend = true` is passed.
- **Compatibility with Other Plugins**: The behavior does not conflict with any other plugins and can be used with any model that uses RainLab.Translate. All internal properties and functions are prefixed with `trans` to ensure no conflicts.
- **Cache Key Uniqueness**: The cache key is generated based on the class name, model ID, and cache type, ensuring no collisions between different models.
- **Automatic Cache Update**: When the model is saved or deleted, all associated cache keys are automatically cleared to ensure data freshness.
- **JSONable Field Handling**: JSON decoding is performed automatically with error handling (try-catch), preventing application crashes in case of invalid JSON.

### 11. The `getTranslationsInFormat` Function (Updated)

The general function has been enhanced to accept additional parameters for controlling fallback and cache.

#### Signature:
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

#### Parameters:
- `$fields`: An array of field names. If `null`, all translatable fields (from `$translatableCaching` and `$translatable`) are used.
- `$format`: The desired format. Can be one of: `'group_by_field'`, `'array'`, `'both'`, `'group_by_locale'`, `'group_by_field_under_all'`, `'both_grouped'`. If `null`, the model's default format (`$translatableApiFormat`) is used.
- `$locales`: An array of locale codes (e.g., `['ar', 'en']`). If `null`, all active locales are used.
- `$includeOriginal`: Boolean, if `true` includes the original value under the key `'x'`.
- `$useFallback`: Boolean, controls fallback usage (`null` to use the global setting).
- `$useCache`: Boolean, controls cache usage (`null` to use the global setting).

#### Return Value:
An array containing the translations in the requested format.

#### Usage Examples:

```php
// Use all default settings (all fields, default format, all locales)
$translations = $product->getTranslationsInFormat();

// Specify a single field and group_by_field format with fallback disabled
$translations = $product->getTranslationsInFormat(
    ['name'],
    'group_by_field',
    null,
    true,
    false,   // useFallback
    true     // useCache
);

// Specify multiple fields and group_by_locale format with specific locales, no cache
$translations = $product->getTranslationsInFormat(
    ['name', 'description'],
    'group_by_locale',
    ['ar', 'en'],
    true,    // includeOriginal
    true,    // useFallback
    false    // useCache
);

// Get translations for all fields in array format without the original value
$translations = $product->getTranslationsInFormat(null, 'array', null, false);
```

#### Example Output (for call with `['name', 'description'], 'group_by_field_under_all', ['ar', 'en']` and `includeOriginal = true`):
```php
[
    'allTranslate' => [
        'name' => [
            'x' => 'منتج 1',
            'ar' => 'منتج 1 بالعربية',
            'en' => 'Product 1'
        ],
        'description' => [
            'x' => 'وصف المنتج',
            'ar' => 'وصف المنتج بالعربية',
            'en' => 'Product description'
        ]
    ]
]
```

### 12. Additional Examples for Fallback and Cache Control

#### a) Calling `getTranslated` with Fallback Temporarily Disabled

```php
$value = $product->getTranslated('name', null, 'ar', false); // Does not use fallback even if globally enabled
```

#### b) Disabling Cache for a Specific Call

```php
$attributes = $product->getTranslatedAttributes('en', false); // Fetches directly from the database
```

#### c) Using `getAllFieldTranslations` with Specified Fallback and Cache

```php
$allNames = $product->getAllFieldTranslations(
    'name',
    true,
    ['ar', 'en', 'fr'],
    true,   // useFallback
    false   // useCache (no cache)
);
```

#### d) Dynamically Checking Cache and Fallback Status

```php
// Temporarily disable cache for all subsequent calls on this model
$product->disableTranslatableCache();

// Enable fallback
$product->withFallbackLocale();

// Call a function that will be affected by these settings
$data = $product->toArrayWithApiTranslations();
```

---

This concludes the comprehensive documentation for the `TranslatableContentCaching` behavior with the latest updates. You can now confidently use it in your multilingual NanoSoft System projects.