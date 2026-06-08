# Nano.TranslateExtended Plugin Documentation

## 1. Introduction

**Nano.TranslateExtended** is an advanced plugin for **OctoberCMS / WinterCMS** that aims to extend and enhance the functionality of the popular **RainLab.Translate** plugin. It offers a set of features that improve the user and developer experience in multilingual websites, focusing on:

- Browser language detection and automatic locale setting.
- Flexible control over language prefixes in URLs.
- Intelligent caching of translated values for better performance.
- Advanced support for outputting translations in API responses (including `translatable_fields` inclusion with the ability to specify fields and languages).
- Helper tools to obtain alternative language content and handle JSON‑castable fields.

The plugin is built to be fully compatible with `RainLab.Translate` and acts as a complement, without conflict or need to modify the original plugin code.

---

## 2. Basic Requirements

- OctoberCMS or WinterCMS version 2.x or higher.
- **RainLab.Translate** plugin installed and enabled.
- PHP 7.2+ (preferably 7.4 or 8.x).

---

## 3. Installation

### Via the Marketplace
1. Open Backend → Settings → System Updates.
2. Search for `Nano.TranslateExtended`.
3. Click Install.

### Via Command Line
```bash
php artisan plugin:install Nano.TranslateExtended
```

### Manually
1. Download the source code from [GitHub](https://github.com/nano/wn-translate-extended).
2. Extract it into the folder `plugins/nano/translateextended`.
3. Run `php artisan plugin:refresh Nano.TranslateExtended`.

After installation, ensure that the `RainLab.Translate` plugin is enabled and you have defined languages in `rainlab.translate::locales`.

---

## 4. Configuration

### 4.1. Backend Settings

After installation, you will find a new settings page under **RainLab.Translate** → **Translate Extended** (or from the sidebar: Settings → Translate Extended).

Available fields:

| Field | Description | Default |
|-------|-------------|---------|
| **Browser Language Detection** | Enable browser language detection and automatic locale setting. | `true` |
| **Prefer User Session** | If enabled, the language stored in the user session takes precedence over the browser‑detected language. | `true` |
| **Query Parameter** | Name of the query parameter (e.g., `lang`) that can be used to set the language. | (empty) |
| **Header** | Name of the HTTP header that carries the language (e.g., `Accept-Language`). | (empty) |
| **Route Prefixing** | Enable adding a language prefix to URLs (e.g., `/en/about`). | `true` |
| **Homepage Redirect** | Redirect the homepage (`/`) to `/locale` (e.g., `/ar`). | `true` |
| **Force Prefix** | Force adding a language prefix to all GET requests that do not have one. | `true` |

> **Note:** These settings can be overridden via environment variables (`.env`) or directly in the configuration file (see section 4.2).

### 4.2. Configuration File (config.php)

The configuration file is located at `plugins/nano/translateextended/config.php`. You can publish it to the `config` folder using:
```bash
php artisan vendor:publish --provider="Nano\TranslateExtended\Plugin" --tag="config"
```

Key settings:

```php
return [
    // Force the default locale (overrides automatic detection)
    'forceDefaultLocale' => env('NANO_TRANSLATE_FORCE_LOCALE', null),

    // Whether to add a prefix to the default locale
    'prefixDefaultLocale' => env('NANO_TRANSLATE_PREFIX_LOCALE', true),

    // Cache timeout for translations (in minutes)
    'cacheTimeout' => env('NANO_TRANSLATE_CACHE_TIMEOUT', 1440),

    // Disable language prefixes for routes automatically (may be used for API)
    'disableLocalePrefixRoutes' => env('NANO_TRANSLATE_DISABLE_PREFIX_ROUTES', false),

    // API‑specific settings (for the DynamicAddIncludeTranslatableApiFields behaviour)
    'translatable_fields' => [
        'fields'          => env('NANO_TRANSLATE_API_FIELDS', null),
        'format'          => env('NANO_TRANSLATE_API_FORMAT', 'group_by_field_under_all'),
        'locales'         => env('NANO_TRANSLATE_API_LOCALES', null),
        'includeOriginal' => env('NANO_TRANSLATE_API_INCLUDE_ORIGINAL', null),
        'useFallback'     => env('NANO_TRANSLATE_API_USE_FALLBACK', null),
        'useCache'        => env('NANO_TRANSLATE_API_USE_CACHE', null),
        'exclude'         => env('NANO_TRANSLATE_API_EXCLUDE', ''),
    ],
];
```

---

## 5. Key Features

### 5.1. Advanced Language Detection (Browser Language Detection)

- When a user visits for the first time, the plugin reads the preferred language from the browser (via the `Accept-Language` header, a custom query parameter, or a custom header).
- The language is automatically set for the user, with the option to save it in the session (if `Prefer User Session` is enabled).
- Priorities can be controlled through the settings.

### 5.2. Route Prefix Management

- A language prefix is added to all site URLs (e.g., `/ar/contact`).
- The homepage is automatically redirected to the active language (if `Homepage Redirect` is enabled).
- Force the prefix on all GET requests that do not have one (`Force Prefix`) to ensure consistent URLs.

> **Example:**  
> If a user visits `https://example.com/about` and the default language is `en`, they will be redirected to `https://example.com/en/about`.

### 5.3. Translation Caching (`TranslatableContentCaching`)

**`TranslatableContentCaching`** behaviour provides:

- **Caching of translated attributes** per model and per language.
- **Automatic cache invalidation** when the model is saved or deleted.
- **Dynamic methods** to directly access translated values via properties (e.g., `$model->title`).
- **Ability to disable caching** globally or per call.
- **Support for JSON‑castable fields** with safe error handling.

#### Adding the behaviour to a model

```php
class Product extends Model
{
    public $implement = [
        '@RainLab.Translate.Behaviors.TranslatableModel',
        'Nano.TranslateExtended.Behaviors.TranslatableContentCaching'
    ];

    // Translatable fields (from RainLab.Translate)
    public $translatable = ['name', 'description'];

    // Additional fields for caching (e.g., JSON)
    public $translatableCaching = ['images', 'specs'];

    // Fields whose full translations will be shown in the API (see section 5.4)
    public $translatableApiFields = ['name', 'description'];
}
```

#### Additional model methods

- `getTranslated($attribute, $default, $locale, $fallback, $useCache, $useInBackend)`
- `getAllFieldTranslations($field, $includeDefault, $locales, $useFallback, $useCache)`
- `getTranslationsInFormat($fields, $format, $locales, $includeOriginal, $useFallback, $useCache)`
- `toArrayWithApiTranslations($locale, $useFallback, $useCache)`
- `invalidateAllCaches()`

#### Translation output formats

| Format | Description |
|--------|-------------|
| `group_by_field` | Adds for each field a `field_all` object with translations `{x, ar, en}`. |
| `array` | `field_all` as an array of `{locale, value}`. |
| `both` | Both (`_object` and `_array`). |
| `group_by_locale` | A single `allTranslate` object grouped by language. |
| `group_by_field_under_all` | A single `allTranslate` object grouped by field. |
| `both_grouped` | Combines `group_by_field_under_all` with the individual `{field}_all` fields. |

### 5.4. Including Translations in API Responses (`DynamicAddIncludeTranslatableApiFields`)

The **`DynamicAddIncludeTranslatableApiFields`** behaviour adds the ability to include translations in API responses via the parameter `?include=translatable_fields`. It works with any Transformer that uses `Nano\API\Classes\Transformer` (or any Transformer that can be extended with `October\Rain\Extension\ExtensionBase`).

#### Features

- Automatically adds new `availableIncludes`: `translatable_fields` and `translatable_attributes`.
- **Full flexibility in specifying fields and languages** via:
  - Public properties in the Transformer.
  - Configuration file `nano.translateextended::translatable_fields`.
  - Query string parameters (highest priority).
- **Supports exclusion** `?translatable_fields.exclude=key1,key2`.
- **Compatibility** with all `TranslatableContentCaching` methods (`getTranslationsInFormat`).

#### Usage

**In `Plugin.php` (registering the Transformer):**
```php
public function boot()
{
    \Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields::extendTransformer(
        \Nano\ShopApi\Transformers\ProductTransformer::class
    );
}
```

**In the API request:**
```http
GET /api/products/1?include=translatable_fields&translatable_fields.fields=name,description&translatable_fields.locales=ar,en
```

**Response** (with `group_by_field_under_all` format):
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

### 5.5. Other Helper Tools

- **ExtendedLocalePicker** – a component that can be added to templates to display a list of available languages with their links.
- **ControllerTrait** – a trait to easily obtain the `Cms\Classes\Controller` object inside any class.
- **TranslatableContentObjectTrait** – an older alternative trait providing similar methods to `TranslatableContentCaching`; it is recommended to use the new behaviour instead.

---

## 6. Practical Examples

### 6.1. Displaying Translations in a Template (Twig)

After adding `TranslatableContentCaching` to the model, you can access translated values directly:

```twig
<h1>{{ product.name }}</h1> {# according to the current language #}
<p>{{ product.description }}</p>
```

### 6.2. Getting All Translations for a Specific Field

```php
$allNames = $product->getAllFieldTranslations('name', true, ['ar', 'en']);
// Result: ['x' => 'Product 1', 'ar' => 'منتج 1 بالعربية', 'en' => 'Product 1']
```

### 6.3. API Call with Custom Translations

```http
GET https://example.com/api/products/5?include=translatable_fields&translatable_fields.format=both_grouped&translatable_fields.useCache=false
```

### 6.4. Temporarily Disable Caching in Code

```php
$product->disableTranslatableCache();
$data = $product->toArrayWithApiTranslations();
$product->enableTranslatableCache();
```

---

## 7. Customisation and Extensibility

### 7.1. Adding a New Trait or Behaviour

You can extend `TranslatableContentCaching` by adding new methods or modifying the behaviour using `extendClassWith`.

### 7.2. Replacing the Routing Logic (routes.php)

A `routes.php` file is included that handles all redirection and prefix logic. You can completely disable it by removing the file or modifying it as needed.

### 7.3. Environment Variables (.env)

```ini
NANO_TRANSLATE_FORCE_LOCALE=ar
NANO_TRANSLATE_PREFIX_LOCALE=true
NANO_TRANSLATE_CACHE_TIMEOUT=2880
NANO_TRANSLATE_DISABLE_PREFIX_ROUTES=false
NANO_TRANSLATE_API_FORMAT=array
NANO_TRANSLATE_API_FIELDS=name,description
NANO_TRANSLATE_API_EXCLUDE=internal_notes
```

---

## 8. Troubleshooting

| Problem | Possible Solution |
|---------|------------------|
| Translations do not appear in the API despite `include=translatable_fields` | Ensure the model uses `TranslatableContentCaching` and the Transformer is extended with the `DynamicAddIncludeTranslatableApiFields` behaviour. Check the error logs. |
| Homepage is not redirected | Ensure `Homepage Redirect` is enabled in the settings, and that there is no conflict with other plugins that modify routes. |
| Cache does not update after editing a translation | If the translation was edited directly in the database, you may need to call `$model->invalidateAllCaches()` manually. |
| Error `Method getTranslationsInFormat does not exist` | The model does not apply `TranslatableContentCaching`. Add the behaviour or use the alternative trait. |
| Browser language is not detected | Ensure `Browser Language Detection` is enabled and not disabled via environment variables. |

---

## 9. Versions and Updates

Current version: **1.0.11**

Latest updates (from `version.yaml`):
- Improved API translation calls with flexible field/locale parameters.
- Added new includes: `translatable_attributes` and `translatable_dirty_locales`.
- Enhanced `TranslatableContentCaching` with dynamic getter/setter methods.
- Better fallback handling for the default locale.
- Better compatibility with the `RainLab.Translate` attribute data structure.
- JSON error logging in debug mode.

For a detailed changelog, see `version.yaml` inside the plugin.

---

## 10. Conclusion

**Nano.TranslateExtended** offers a complete solution for managing multilingual content in OctoberCMS/WinterCMS projects. Through its seamless integration with `RainLab.Translate`, the plugin provides:

- Intelligent user language detection.
- Full control over routes and prefixes.
- High‑performance caching for translations.
- Flexible API interfaces to display translations with the ability to specify fields and languages.
- Helper tools for developers to speed up development.

Whether you are running a small blog or a large multilingual e‑commerce store, this plugin will help you achieve a smooth and scalable user experience.

---

**Reference documentation**:
- [`DynamicAddIncludeTranslatableApiFields` behaviour documentation](./Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-en.md)
- [`TranslatableContentCaching` behaviour documentation](./Behaviors/Docs-TranslatableContentCaching-en.md)
- [Enhanced Translations API Usage Guide](./Docs-API-Translations-Guide-en.md)

**Useful links:**
- [Winter.Translate Documentation](https://github.com/wintercms/wn-translate-plugin)
- [WinterCMS Documentation](https://wintercms.com/docs/)
- [RainLab.Translate Documentation](https://github.com/rainlab/translate-plugin)
- [OctoberCMS Documentation](https://docs.octobercms.com)
