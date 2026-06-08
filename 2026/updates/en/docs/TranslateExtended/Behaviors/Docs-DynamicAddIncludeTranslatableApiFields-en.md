# Comprehensive Documentation of the `DynamicAddIncludeTranslatableApiFields` Behavior

## 1. Introduction

**DynamicAddIncludeTranslatableApiFields** is a behavior specific to **Nano.TranslateExtended**, designed to extend the scope of a **Transformer** in the **Nano.API** system (or any system using League Fractal) to support including (`include`) multilingual translations in API responses.

The core problem this behaviour solves: in previous versions, translations did not appear when adding `translatable_fields` to the `include` list in an API request. Additionally, there was no flexible way to specify the required fields or languages per call. This behaviour fully addresses these issues.

The behaviour provides three include methods:

- `includeTranslatableFields`: responsible for returning the actual translations for fields, with support for specifying fields, languages, format, and exclusion.
- `includeTranslatableAttributes`: returns the list of translatable field names from the model (`getTranslatableAttributes`).
- `includeTranslatableDirtyLocales`: (optional, disabled by default) returns information about dirty locales and originals for debugging.

## 2. Goal and Features

- **Fix the issue of translations not appearing** when `translatable_fields` is included in the API.
- **Full flexibility in specifying fields and languages** via:
  - Public properties that can be set in the Transformer itself.
  - Configuration file `nano.translateextended::translatable_fields`.
  - Query string parameters in the API request (highest priority).
- **Support multiple field formats**: array, comma‑separated string (`title,content`), or the words `'all'`/`'*'` to fetch all translatable fields.
- **Support excluding certain keys** from the result (e.g., `exclude=password,secret`).
- **Ability to override settings per request** without modifying code or global configuration.
- **Full compatibility with the `Nano.API` architecture** (Transformer extends `October\Rain\Extension\ExtensionBase`).

## 3. Installation and Setup

### Prerequisites
- `Nano.TranslateExtended` plugin installed and enabled.
- `Nano.API` (or any system using `League\Fractal` with an extendable `Transformer`).
- Models to be translated must use `TranslatableContentCaching` (or at least have a `getTranslationsInFormat` method).

### Adding the Behavior to the Transformer

In your plugin’s `Plugin.php`, or directly in the Transformer definition, use the helper function `extendTransformerTranslatableApiFields` provided in the base plugin, or manually extend the Transformer:

**Automatic method (recommended):**
```php
// In Plugin.php of Nano.TranslateExtended (already present)
public function extendTransformer(): void
{
    if(class_exists(\Nano\ShopApi\Transformers\ProductTransformer::class)){
        $this->extendTransformerTranslatableApiFields(\Nano\ShopApi\Transformers\ProductTransformer::class);
    }
    // Add other Transformers that need translations
}
```

**Manual method (in any plugin):**
```php
use Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields;

$transformerClass::extend(function($transformer) {
    if (!$transformer->isClassExtendedWith(DynamicAddIncludeTranslatableApiFields::class)) {
        $transformer->extendClassWith(DynamicAddIncludeTranslatableApiFields::class);
    }
});
```

After this, your Transformer will have three new `availableIncludes`:
- `translatable_fields`
- `translatable_attributes`
- `translatable_dirty_locales` (commented out; can be enabled by uncommenting in the constructor)

## 4. Public Properties to Control Parameters

You can set default parameter values directly in the Transformer itself using the following public properties (all default to `null`):

| Property | Description |
|----------|-------------|
| `$translatableFieldsConfigFields` | Required fields (array, comma‑separated string, or `'all'`/`'*'`). |
| `$translatableFieldsConfigFormat` | Output format (see `TranslatableContentCaching` formats). |
| `$translatableFieldsConfigLocales` | Required languages (array or comma‑separated string). |
| `$translatableFieldsConfigIncludeOriginal` | boolean: include the original value under the key `'x'` or not. |
| `$translatableFieldsConfigUseFallback` | boolean: use fallback (fall back to original value when translation missing). |
| `$translatableFieldsConfigUseCache` | boolean: use caching or not. |

**Example:**
```php
class ProductTransformer extends Transformer
{
    public $translatableFieldsConfigFields = 'name,description';
    public $translatableFieldsConfigFormat = 'group_by_field_under_all';
    public $translatableFieldsConfigLocales = ['ar', 'en'];
}
```

## 5. Priority of Value Resolution

When `includeTranslatableFields` is called, parameter values are determined in the following priority (highest to lowest):

1. **Public property** in the Transformer (or in the behaviour itself, but the behaviour primarily reads from the Transformer).
2. **Value from the configuration file** (`config/nano.translateextended::translatable_fields.{$param}`). You can set it in the plugin’s `config.php`.
3. **Value from `Input`** (query string) in two patterns:
   - `translatable_fields.{param}` (e.g., `translatable_fields.fields=name,content`)
   - Direct: `translatableFieldsConfig{Param}` (e.g., `translatableFieldsConfigFields=name`).
   - For the `fields` parameter only: `translated_fields` or `translatable_fields` (shortcut).
4. **Default value** passed to the function (in code: `'group_by_field_under_all'` for format, `null` for most, `true` for `includeOriginal`, `false` for `useFallback` and `useCache`).

This gives you full control: you can set global defaults in config, override them in a specific Transformer, and then override them again per API request.

## 6. Include Methods

### a) `includeTranslatableFields($item)`

The primary method called when `?include=translatable_fields` is requested. It does the following:

1. Retrieves parameter values according to the priority described above.
2. Processes `$fields` and `$locales` (converts comma‑separated strings to arrays, and converts `'all'`/`'*'` to `null` to fetch all fields).
3. Calls `$item->transCollectFields()` (if it exists) to ensure the list of translatable fields is updated from the model.
4. Calls `$item->getTranslationsInFormat()` with the prepared parameters.
5. If the result contains an `'allTranslate'` key (from the `group_by_field_under_all` format for example), extracts its value.
6. Applies key exclusion (`exclude`) from `Input` or configuration.
7. Returns an `Item` resource from Fractal containing the result (or an empty array on error).

**Example response (with `group_by_field_under_all` format):**
```json
{
    "data": {
        "id": 1,
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

### b) `includeTranslatableAttributes($item)`

Returns an array of the translatable field names from the model by calling `$item->getTranslatableAttributes()`.

**Example:**
```http
GET /api/products/1?include=translatable_attributes
```
```json
{
    "data": {
        "id": 1,
        "translatable_attributes": ["name", "description", "meta_title"]
    }
}
```

### c) `includeTranslatableDirtyLocales($item)`

(Disabled by default; requires uncommenting in the constructor). Returns debugging information: `getDirtyLocales`, `getTranslatableOriginals('en')`. Useful for developers when monitoring translation changes.

## 7. Supported Output Formats

The formats depend on what `getTranslationsInFormat` provides (which comes from `TranslatableContentCaching`). You can use any of the following constants:

- `'group_by_field'` – for each field, an object `{field}_all`.
- `'array'` – for each field, an array of `{locale, value}`.
- `'both'` – both, with different keys.
- `'group_by_locale'` – a single `allTranslate` object grouped by language.
- `'group_by_field_under_all'` – a single `allTranslate` object grouped by field.
- `'both_grouped'` – combines `group_by_field_under_all` with the individual `{field}_all` fields.

The default format in this behaviour is `'group_by_field_under_all'` unless overridden.

## 8. Controlling via Query String (Input)

When calling the API, you can pass the following parameters (within `?`):

| Parameter | Description | Example |
|-----------|-------------|---------|
| `translatable_fields.fields` | The field or fields required. | `?translatable_fields.fields=name,description` |
| `translated_fields` | Shortcut for `fields` only. | `?translated_fields=name` |
| `translatable_fields.format` | Output format. | `?translatable_fields.format=group_by_locale` |
| `translatable_fields.locales` | Required languages. | `?translatable_fields.locales=ar,en` |
| `translatable_fields.includeOriginal` | `true`/`false`/`1`/`0`. | `?translatable_fields.includeOriginal=false` |
| `translatable_fields.useFallback` | `true`/`false`. | `?translatable_fields.useFallback=true` |
| `translatable_fields.useCache` | `true`/`false`. | `?translatable_fields.useCache=false` |
| `translatable_fields.exclude` | Keys to exclude from the final result. | `?translatable_fields.exclude=password,secret` |

**Note:** You can also use the direct syntax `translatableFieldsConfigFields`, but the recommended syntax is `translatable_fields.*` for compatibility with API documentation.

## 9. Practical API Call Examples

### Example 1: Fetch translations for only two fields with Arabic language
```http
GET /api/products/1?include=translatable_fields&translatable_fields.fields=name,description&translatable_fields.locales=ar
```

### Example 2: Fetch all translations, exclude a specific field, without cache
```http
GET /api/products/1?include=translatable_fields&translatable_fields.fields=all&translatable_fields.useCache=false&translatable_fields.exclude=internal_notes
```

### Example 3: Use the shortcut `translated_fields` with `group_by_locale` format
```http
GET /api/products/1?include=translatable_fields&translated_fields=title&translatable_fields.format=group_by_locale
```

### Example 4: Fetch only the names of translatable fields
```http
GET /api/products/1?include=translatable_attributes
```

## 10. Customising the Behaviour in the Transformer

If you need specific default values for all calls to a particular Transformer, you can set the public properties:

```php
use Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields;

class MyCustomTransformer extends \Nano\API\Classes\Transformer
{
    public $translatableFieldsConfigFields = ['name', 'description'];
    public $translatableFieldsConfigFormat = 'array';
    public $translatableFieldsConfigUseFallback = false;

    public function __construct()
    {
        // Add the behaviour manually or via Plugin.php
        $this->extendClassWith(DynamicAddIncludeTranslatableApiFields::class);
    }
}
```

## 11. Helper Function `getConfigValue`

Used internally to apply the priority order. You can call it if you want to read a certain parameter with the same priority:

```php
protected function getConfigValue($param, $default = null)
```

**Parameters:**
- `$param`: the parameter name (`'fields'`, `'format'`, `'locales'`, `'includeOriginal'`, `'useFallback'`, `'useCache'`).
- `$default`: default value if none is found in any source.

**Return value:** The value determined by priority.

**Example:**
```php
$fields = $this->getConfigValue('fields', null);
```

## 12. Helper Function `isMethodExists`

Used to safely check if a method exists on an object, with support for `methodExists` (which may be present in some extensions, such as `October\Rain\Extension\ExtensionBase`).

## 13. Important Notes

- **Dependency on `TranslatableContentCaching`**: For `includeTranslatableFields` to work correctly, the model being translated must have added the `TranslatableContentCaching` behaviour (or at least have a `getTranslationsInFormat` method). If not present, the behaviour will return an empty array.
- **Caching performance**: You can enable caching to improve performance, but be aware that translation updates may not appear immediately if cache is enabled. Temporarily use `useCache=false` or clear the cache after updates.
- **Exclusion (`exclude`)**: Applied to the final result after all processing, so you can exclude keys like `'allTranslate'` if you only want the individual fields.
- **Translation in the backend**: The behaviour does not automatically translate in the backend environment, but if the model uses `TranslatableContentCaching` with `useInBackend` enabled, translation will work.
- **Compatibility**: This behaviour works with any `Transformer` that extends `October\Rain\Extension\ExtensionBase` or `Nano\API\Classes\Transformer`. If you use Fractal directly without extending `ExtensionBase`, you may need a slight modification.

## 14. Troubleshooting

### Problem: Translations do not appear despite adding `include=translatable_fields`
- Ensure the model implements `TranslatableContentCaching` (or has a `getTranslationsInFormat` method).
- Ensure the Transformer has been extended with this behaviour (see the `extendTransformer` method in `Plugin.php`).
- Check the error logs (`storage/logs/laravel.log`) because the behaviour logs errors in debug mode.

### Problem: Error `Method getTranslationsInFormat does not exist`
- This means the model does not have the required method. Add `TranslatableContentCaching` to `$implement` in the model or register it using `TranslatableContentCaching::registerModel()`.

### Problem: Some parameters from Input are not applied
- Ensure the parameter names are written correctly (e.g., `translatable_fields.fields` not `translatable_fields.field`).
- Try using the shortcut `translated_fields` to specify fields.
- Ensure the values sent do not contain extra spaces.

## 15. Full Usage Example in a Project

**`Plugin.php` (registering the Transformer):**
```php
public function boot()
{
    // Register the behaviour for ProductTransformer
    if (class_exists(\Nano\ShopApi\Transformers\ProductTransformer::class)) {
        \Nano\ShopApi\Transformers\ProductTransformer::extend(function($transformer) {
            if (!$transformer->isClassExtendedWith(\Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields::class)) {
                $transformer->extendClassWith(\Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields::class);
            }
        });
    }
}
```

**API call (display a product with specific translations):**
```http
GET https://example.com/api/products/15?include=translatable_fields&translatable_fields.fields=name,description&translatable_fields.locales=ar,en&translatable_fields.format=both_grouped
```

**Expected response:**
```json
{
    "data": {
        "id": 15,
        "name": "Normal product",
        "description": "Normal description",
        "name_all": {
            "x": "Normal product",
            "ar": "منتج عادي بالعربية",
            "en": "Normal Product"
        },
        "description_all": {
            "x": "Normal description",
            "ar": "وصف عادي بالعربية",
            "en": "Normal description"
        },
        "allTranslate": {
            "name": {
                "x": "Normal product",
                "ar": "منتج عادي بالعربية",
                "en": "Normal Product"
            },
            "description": {
                "x": "Normal description",
                "ar": "وصف عادي بالعربية",
                "en": "Normal description"
            }
        }
    }
}
```
