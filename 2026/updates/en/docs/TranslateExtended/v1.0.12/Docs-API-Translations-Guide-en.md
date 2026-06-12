# Enhanced Translations API Usage Guide

## 1. Introduction

The **Nano.TranslateExtended** plugin provides advanced mechanisms for including translations in your API responses, with full flexibility to specify the required fields and languages for each request. This guide explains how to use `include=translatable_fields` in API endpoints, control translation output via query string parameters, and handle different scenarios.

**Basic requirements:**
- `Nano.TranslateExtended` plugin version 1.0.11 or later installed.
- `RainLab.Translate` plugin installed and configured with the required languages.
- `Nano.API` or any `Transformer` system that supports `League\Fractal` with class extension capability.
- Models to be translated must use the `TranslatableContentCaching` behavior (see its documentation).

---

## 2. Preparing the Transformer to Receive Translations

Before using translations in the API, you must add the `DynamicAddIncludeTranslatableApiFields` behavior to the relevant `Transformer`.

### Automatic method (recommended)

In the `Plugin.php` file of the `Nano.TranslateExtended` plugin, the `extendTransformer()` function is already called to automatically register the required Transformers. You can add additional Transformers in the same pattern:

```php
// In Plugin.php
public function extendTransformer(): void
{
    if (class_exists(\Nano\ShopApi\Transformers\ProductTransformer::class)) {
        $this->extendTransformerTranslatableApiFields(\Nano\ShopApi\Transformers\ProductTransformer::class);
    }
    // Add other Transformers here
    if (class_exists(\Custom\Api\Transformers\ArticleTransformer::class)) {
        $this->extendTransformerTranslatableApiFields(\Custom\Api\Transformers\ArticleTransformer::class);
    }
}
```

### Manual method

If you prefer to manually extend a specific Transformer:

```php
use Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields;

$transformerClass::extend(function($transformer) {
    if (!$transformer->isClassExtendedWith(DynamicAddIncludeTranslatableApiFields::class)) {
        $transformer->extendClassWith(DynamicAddIncludeTranslatableApiFields::class);
    }
});
```

After that, your Transformer will have the following `availableIncludes`:
- `translatable_fields`
- `translatable_attributes`
- `translatable_dirty_locales` (disabled by default, can be enabled in the constructor)

---

## 3. Using `include=translatable_fields` in API Requests

When calling any endpoint that uses a prepared Transformer, you can add `include=translatable_fields` to the query string, and the translations will appear under the `translatable_fields` key in the response.

**Basic example:**
```http
GET /api/products/1?include=translatable_fields
```

**Typical response (using the default `group_by_field_under_all` format):**
```json
{
    "data": {
        "id": 1,
        "name": "Product 1",
        "description": "Product description",
        "translatable_fields": {
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
    }
}
```

---

## 4. Translation Control Parameters (Query String)

You can override the default translation settings (from `config.php` or the Transformer’s properties) by adding parameters to `translatable_fields.*` in the query string. All parameters are optional.

### 4.1. Specifying Fields (`fields`)

To fetch translations for only specific fields (instead of all translatable fields):

```http
GET /api/products/1?include=translatable_fields&translatable_fields.fields=name,description
```

**You can also use the shortcut `translated_fields`:**
```http
GET /api/products/1?include=translatable_fields&translated_fields=name,description
```

**Other supported formats:**
- `translatable_fields.fields=all` or `*` to fetch all translatable fields (calls `getTranslatableAttributes()`).
- Pass values as a comma‑separated string (`name,description,seo_title`).

### 4.2. Specifying Languages (`locales`)

To fetch translations for only specific languages (instead of all active languages):

```http
GET /api/products/1?include=translatable_fields&translatable_fields.locales=ar,en
```

**Supported formats:**
- Comma‑separated string (`ar,en,fr`).
- `all` or `*` (uses all active languages, which is the default behaviour when not specified).

### 4.3. Output Format (`format`)

Determines the shape of the returned translations object. Supported values (see `TranslatableContentCaching` formats):

| Value | Description |
|-------|-------------|
| `group_by_field` | Adds for each field a `{field}_all` object with translations `{x, ar, en}`. |
| `array` | `{field}_all` as an array of objects `{locale, value}`. |
| `both` | Both (`_object` and `_array`). |
| `group_by_locale` | A single `allTranslate` object grouped by language. |
| `group_by_field_under_all` | A single `allTranslate` object grouped by field (default). |
| `both_grouped` | Combines `group_by_field_under_all` with the individual `{field}_all` fields. |

**Example:**
```http
GET /api/products/1?include=translatable_fields&translatable_fields.format=group_by_locale
```

**Response:**
```json
{
    "data": {
        "id": 1,
        "name": "Product 1",
        "description": "Product description",
        "translatable_fields": {
            "allTranslate": {
                "x": { "name": "Product 1", "description": "Product description" },
                "ar": { "name": "منتج 1 بالعربية", "description": "وصف المنتج بالعربية" },
                "en": { "name": "Product 1", "description": "Product description" }
            }
        }
    }
}
```

### 4.4. Including the Original Value (`includeOriginal`)

Controls whether the original value (from the main table field) appears under the key `'x'`. Values: `true` / `false` / `1` / `0`.

```http
GET /api/products/1?include=translatable_fields&translatable_fields.includeOriginal=false
```

### 4.5. Using Fallback (`useFallback`)

If enabled (`true`), falls back to the original value when no translation exists for the requested language.

```http
GET /api/products/1?include=translatable_fields&translatable_fields.useFallback=true
```

### 4.6. Using Cache (`useCache`)

Disables or enables the use of caching for this request only. Values: `true` / `false` / `1` / `0`.

```http
GET /api/products/1?include=translatable_fields&translatable_fields.useCache=false
```

### 4.7. Excluding Certain Keys (`exclude`)

A comma‑separated list of keys (final object keys) to exclude from the translation result.

```http
GET /api/products/1?include=translatable_fields&translatable_fields.exclude=allTranslate,internal_notes
```

> **Note:** `exclude` is applied after the final result is processed, so you can use it to remove the container object `allTranslate` if you only want the individual fields (for example, in `group_by_field` format).

---

## 5. Complete Practical Examples

### Example 1: Fetch translations for only two fields (name and description) in Arabic and English, using the `group_by_locale` format

```http
GET /api/products/15?include=translatable_fields&translatable_fields.fields=name,description&translatable_fields.locales=ar,en&translatable_fields.format=group_by_locale
```

### Example 2: Fetch all translations, disable cache, and exclude the container object

```http
GET /api/products/15?include=translatable_fields&translatable_fields.fields=all&translatable_fields.useCache=false&translatable_fields.exclude=allTranslate
```

(This will return only the individual fields `name_all`, `description_all` in the default `group_by_field` format.)

### Example 3: Use the shortcut `translated_fields` without other parameters

```http
GET /api/products/15?include=translatable_fields&translated_fields=name,description
```

### Example 4: Combine `include=translatable_attributes` to get the list of translatable field names

```http
GET /api/products/15?include=translatable_fields,translatable_attributes&translatable_fields.locales=ar
```

**Response:**
```json
{
    "data": {
        "id": 15,
        "translatable_attributes": ["name", "description", "meta_title"],
        "translatable_fields": {
            "allTranslate": {
                "name": { "x": "Product 15", "ar": "منتج 15 بالعربية" },
                "description": { "x": "Product description", "ar": "وصف المنتج بالعربية" }
            }
        }
    }
}
```

### Example 5: Control translations through Transformer properties in advance (without passing parameters)

If you want to set default values for all calls to a specific Transformer, you can set the public properties:

```php
class ProductTransformer extends \Nano\API\Classes\Transformer
{
    public $translatableFieldsConfigFields = ['name', 'description'];
    public $translatableFieldsConfigFormat = 'array';
    public $translatableFieldsConfigUseFallback = false;
}
```

Then just `?include=translatable_fields` is enough to use these settings.

---

## 6. Getting Only the Names of Translatable Fields

You can use `include=translatable_attributes` without fetching the translations themselves, which is useful for generating dynamic interfaces.

```http
GET /api/products/1?include=translatable_attributes
```

**Response:**
```json
{
    "data": {
        "id": 1,
        "translatable_attributes": ["name", "description", "meta_title"]
    }
}
```

---

## 7. Debugging Information

If debug mode is enabled (`config('app.debug') = true`) and an error occurs while fetching translations, the `translatable_fields` response will contain `error` and `debug` keys with error details. This helps identify issues without completely breaking the API.

**Example error response:**
```json
{
    "data": {
        "id": 1,
        "translatable_fields": {
            "error": "Call to undefined method getTranslationsInFormat",
            "debug": {
                "line": 105,
                "file": "...",
                "trace": [...]
            }
        }
    }
}
```

---

## 8. Compatibility with RainLab.Translate Versions

The behaviour flexibly handles the `RainLab.Translate` data structure: it first looks for `locale` in `attributes` (the modern way), then falls back to the direct property `locale`. This ensures compatibility with different versions of the plugin.

---

## 9. Performance Tips

- **Use caching**: In production, it is recommended to enable `useCache` (by default `false` in the behaviour, but can be enabled in the global settings). This will significantly reduce database queries.
- **Specify only the required fields and languages**: The fewer fields and languages you request, the faster the response.
- **Exclude unnecessary keys** using `exclude` if you don’t need the container object or some computed fields.

---

## 10. Summary

Managing translations in the API with `Nano.TranslateExtended` has become flexible and powerful. You can now:

- Include translations in any API response using `include=translatable_fields`.
- Precisely control fields, languages, and format via the query string.
- Obtain the list of translatable field names separately.
- Override global settings per request.
- Leverage caching for better performance.

Use this guide as a quick reference to build professional multilingual API interfaces.

---

**For more technical details, see:**

- [`Nano.TranslateExtended` plugin documentation](./Docs-Nano.TranslateExtended-en.md)
- [`TranslatableContentCaching` behavior documentation](./Behaviors/Docs-TranslatableContentCaching-en.md)
- [`DynamicAddIncludeTranslatableApiFields` behavior documentation](./Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-en.md)
