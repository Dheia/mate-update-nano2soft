## Comprehensive Documentation for `TranslationOverrideManager` (Version 1.0.12)

### 1. Introduction

**TranslationOverrideManager** is an advanced class within the **Nano.TranslateExtended** plugin, designed to provide a centralized and flexible mechanism to override any translation in the system without modifying the original translation files of plugins or the core system. The class reads custom translations from specified folders (e.g., `lang/override` or other custom paths), caches them for performance, and merges them into OctoberCMS's translation system using the `set()` method of the `October\Rain\Translation\Translator` object.

The primary goal is to enable developers to customize user-facing texts (front-end or API) without touching the original plugin files, facilitating maintenance and updates while preserving customizations when plugins are updated.

---

### 2. Key Benefits and Features

- **Override any translation**: Change any translation key existing in any plugin or the core system (e.g., `nano.sitemanager::lang.site_switcher.btn.manage_shop`) without modifying original files.
- **Multiple path support**: Searches in several folders (configurable via settings or environment variables), with priority (the last processed path takes precedence when the same key exists in multiple files).
- **Smart caching**: Reads translations from files once and stores them in cache for a configurable duration (minutes). Can disable caching entirely (set to 0) in development environments.
- **Efficient merging using `set()`**: Uses October's translator `set()` method instead of `addLines()` to ensure override even if translations were already loaded.
- **Complex key support**: Handles regular keys (`messages.welcome`) and namespaced keys (`vendor.plugin::lang.group.key`).
- **Dynamic and safe**: Provides methods to add extra paths at runtime (`addPath`), reload translations (`reload`), and clear cache (`clearCache`).
- **Full integration with plugin settings**: Reads settings from the `Settings` model with priority: environment variables ← database ← `config.php`.
- **Exception handling and logging**: Catches all exceptions (`Exception|Throwable`) and logs them to `system.log` without crashing the application.
- **Nested array support**: Allows writing translations hierarchically in PHP files (e.g., `['products' => ['new' => 'New']]`) which will be automatically flattened to dot notation (`products.new`).

---

### 3. Requirements

- OctoberCMS or WinterCMS version 2.0 or higher.
- `RainLab.Translate` plugin installed and activated (we use the core translation system).
- `Nano.TranslateExtended` plugin installed and activated.

---

### 4. Internal Properties

These properties are used internally and do not need direct modification by developers, but understanding them helps with customization:

| Property | Type | Description |
|----------|------|-------------|
| `$paths` | `array` | List of absolute paths to search for override files (ordered by priority). |
| `$cacheMinutes` | `int` | Cache duration in minutes (0 = disable cache). |
| `$enabled` | `bool` | Whether the system is enabled (set automatically from settings). |
| `$loaded` | `array` | Request‑local memory to avoid reloading translations for the same language multiple times. |
| `$translator` | `Translator` | Reference to the core translator object (`October\Rain\Translation\Translator`). |

---

### 5. Public Methods

#### 5.1 `__construct()`

Called automatically when the object is created. It reads settings from `Settings`, `config.php`, or environment variables, and initialises paths and cache duration.

**No manual call needed**; the object is created via the Service Container:

```php
$manager = app('translateextended.translationOverride');
```

---

#### 5.2 `loadAll(?string $locale = null): array`

Loads all available translations for all languages (or a single language if `$locale` is specified).

**Parameters:**
- `$locale` (optional) – language code (e.g., `'ar'`). If `null`, loads all languages found in the paths.

**Return value:** Array of loaded translations (per language).

**Example:**
```php
$manager->loadAll();     // loads all languages
$manager->loadAll('en'); // loads only English
```

---

#### 5.3 `loadForLocale(string $locale): array`

Loads custom translations for a specific language and merges them into the translator. Called automatically by `loadAll` or can be called directly.

**Parameters:**
- `$locale` – language code (e.g., `'ar'`).

**Return value:** Array of keys and values loaded for this language.

**Example:**
```php
$overrides = $manager->loadForLocale('ar');
```

**Note:** This method uses cache first, then falls back to files if needed.

---

#### 5.4 `clearCache(?string $locale = null): bool`

Clears the translation cache for all languages or a specific language.

**Parameters:**
- `$locale` (optional) – language code. If `null`, clears cache for all languages.

**Return value:** `true` if cleared successfully, `false` otherwise.

**Example:**
```php
$manager->clearCache();     // clears all cache
$manager->clearCache('ar'); // clears Arabic cache only
```

---

#### 5.5 `addPath(string $path): self`

Dynamically adds an additional search path at runtime. The path is appended to the end of the paths list (lowest priority).

**Parameters:**
- `$path` – absolute or relative path (relative paths are converted to absolute via `base_path` in `getOverridePaths`, but it is recommended to pass an absolute path).

**Return value:** The object itself (`self`) for fluent interface.

**Example:**
```php
$manager->addPath(base_path('my/custom/translations'));
```

---

#### 5.6 `reload(?string $locale = null): array`

Reloads translations for the current language (or a specific language) after clearing the cache. Use when you need to apply immediate changes to override files without waiting for cache expiration.

**Parameters:**
- `$locale` (optional) – language code. If `null`, uses the current application locale.

**Return value:** Array of reloaded translations.

**Example:**
```php
$manager->reload();    // reloads current language
$manager->reload('fr'); // reloads French
```

---

#### 5.7 `isEnabled(): bool`

Checks whether the override system is enabled (based on settings).

**Return value:** `true` if enabled, `false` otherwise.

**Example:**
```php
if ($manager->isEnabled()) {
    // ... load translations
}
```

---

#### 5.8 `getPaths(): array`

Returns the list of currently used paths (after processing).

**Return value:** Array of absolute paths.

**Example:**
```php
$paths = $manager->getPaths();
```

---

#### 5.9 `getCacheMinutes(): int`

Returns the current cache duration (in minutes).

**Return value:** Number of minutes (may be 0 if cache is disabled).

**Example:**
```php
$minutes = $manager->getCacheMinutes();
```

---

### 6. Protected Methods – Internal Use

| Method | Description |
|--------|-------------|
| `getOverridesFromCacheOrFiles(string $locale): array` | Fetches translations from cache if available and enabled, otherwise reads from files. |
| `loadFromFiles(string $locale): array` | Scans all paths, loads `.php` files, and flattens nested arrays to dot notation. |
| `loadTranslationFile(string $filePath): array` | Includes a single PHP file and returns its content (with error handling). |
| `flattenArray(array $array, string $prefix = ''): array` | Converts a nested array to a flat array with dot notation keys. |
| `mergeToTranslator(array $overrides, string $locale): void` | **Most important method**: uses `$this->translator->set()` to merge each key/value into the translator. |
| `getAvailableLocales(): array` | Returns a list of language codes available in the override folders (based on subdirectory names within the paths). |

---

### 7. Detailed Workflow

1. **Object creation** (`__construct`):
   - Reads settings from `Settings::getWithEnvPriority` (priority: env → database → config).
   - If `enable_overrides = false`, the class stops immediately.
   - Fetches paths (from `getOverridePaths` or `config.php`).
   - Cleans paths and ensures a fallback default path exists.

2. **Request to load translations** (`loadForLocale`):
   - Checks `$this->loaded[$locale]` to avoid reloading the same language in the same request.
   - Calls `getOverridesFromCacheOrFiles`:
     - If `cacheMinutes > 0` tries to retrieve from cache.
     - Otherwise or on cache error, reads from files (`loadFromFiles`).
   - Then calls `mergeToTranslator` to add the keys to the translator using `set()`.

3. **Reading files** (`loadFromFiles`):
   - Iterates over each path in `$this->paths`.
   - Builds the full path: `$path . '/' . $locale`.
   - Uses `File::allFiles` to get all PHP files (any name, supports subdirectories).
   - For each file, calls `loadTranslationFile` (which `include`s the file and catches errors).
   - If the file returns an array, it is flattened (`flattenArray`) and merged into `$allOverrides`.

4. **Flattening nested arrays**:
   - Example: a file containing `['a' => ['b' => 'c']]` becomes `['a.b' => 'c']`.
   - This allows writing translations in a hierarchical structure for easier organisation.

5. **Merging using `set()`**:
   - For each key (`$key`) like `'nano.sitemanager::lang.site_switcher.btn.manage_shop'` and value (`$value`).
   - Calls `$this->translator->set($key, $value, $locale)`.
   - This method (from October) stores the translation in `$this->loaded['*']['*'][$locale][$key]`, making it directly available when `Lang::get($key)` is called.

6. **Clearing cache**:
   - When plugin settings are saved (event `backend.form.beforeSave`), `clearCache()` is called.
   - Can be called manually to apply immediate changes to override files.

---

### 8. Integration with Plugin Settings (Settings model)

The class reads the following values from the `Nano\TranslateExtended\Models\Settings` model using the `getWithEnvPriority` method:

| Setting key | Environment variable | Default value | Description |
|-------------|----------------------|---------------|-------------|
| `enable_overrides` | `NANO_TRANSLATE_ENABLE_OVERRIDES` | `true` | Enable/disable the system |
| `override_cache_minutes` | `NANO_TRANSLATE_CACHE_MINUTES` | `60` | Cache duration in minutes |
| `override_paths` | `NANO_TRANSLATE_OVERRIDE_PATHS` | `lang/override`<br>`plugins/nano/translateextended/override/lang` | List of paths (one per line, relative paths allowed) |

**Example of writing paths in the database (textarea):**
```
lang/override
plugins/nano/translateextended/override/lang
/my/absolute/path
```

**Note:** If you want to set paths via `.env`, use commas or newlines inside the value (not recommended). It is better to use the database or `config.php`.

---

### 9. Complete Usage Example

#### a) Creating override files

Assume we want to change some translations in the `nano.sitemanager` and `tss.inventory` plugins.

**File `lang/override/ar/custom.php`**
```php
<?php return [
    'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'Manage Another Store',
    'tss.inventory::lang.products.form.use_case_options.new' => 'New',
    'tss.inventory::lang.products.form.use_case_options.excellent' => 'Excellent',
];
```

**File `lang/override/en/custom.php`**
```php
<?php return [
    'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'Manage Another Store',
    'tss.inventory::lang.products.form.use_case_options.new' => 'New',
    'tss.inventory::lang.products.form.use_case_options.excellent' => 'Excellent',
];
```

#### b) In application code (e.g., Component or Plugin)

```php
use Lang;
use Nano\TranslateExtended\Classes\TranslationOverrideManager;

// Usually we don't need manual calls because the plugin loads automatically in boot()
// But we can use the manager if we want to add an extra path or reload:

$manager = app(TranslationOverrideManager::class);
$manager->addPath(base_path('my/extra/overrides'));
$manager->reload('ar');

// After that, any Lang::get call will return the overridden values:
echo Lang::get('tss.inventory::lang.products.form.use_case_options.excellent'); // "Excellent" (if locale is en)
```

#### c) In Twig templates

```twig
{{ 'tss.inventory::lang.products.form.use_case_options.excellent'|trans }}
```

Will display "Excellent" for English users.

---

### 10. Customising the Override System via Events

You can listen to the following relevant events:

- **`locale.changed`** – `loadForLocale` is automatically called when the language changes (registered inside `bootCustomTranslationOverrides` in `Plugin.php`).
- **`backend.form.beforeSave`** – Cache is automatically cleared when plugin settings are saved.

If you need to reload translations based on another event (e.g., dynamic change of override files), you can call `$manager->reload()`.

---

### 11. Examples of Handling Nested Arrays

**File `lang/override/ar/nested.php`**
```php
<?php return [
    'shop' => [
        'cart' => [
            'title' => 'Shopping Cart',
            'empty' => 'Your cart is empty'
        ]
    ]
];
```

Result after flattening:
```php
[
    'shop.cart.title' => 'Shopping Cart',
    'shop.cart.empty' => 'Your cart is empty'
]
```

Then you can access them as usual:
```php
Lang::get('shop.cart.title'); // "Shopping Cart"
```

**Note:** This key is not namespaced (`::`), so it will be global. If you want to use a namespace, write the full key with `namespace::` at the top level of the array:

```php
<?php return [
    'nano.shop::lang.cart.title' => 'Shopping Cart',
];
```

---

### 12. Important Notes and Common Issues

| Issue | Cause and Solution |
|-------|---------------------|
| **Override not working despite files existing** | Ensure `enable_overrides = true` in settings or `.env`, and clear cache (`php artisan cache:clear` or `$manager->clearCache()`). |
| **Changes in file do not appear immediately** | Due to cache. Either wait for the cache duration (`override_cache_minutes`) or use `$manager->reload()` after modifying the file. |
| **Error `Too few arguments` in `locale.changed` event** | Old event signature in previous versions. Make sure the closure in `bootCustomTranslationOverrides` accepts `$oldLocale = null`. Fixed in version 1.0.12. |
| **Relative paths not working** | Paths must be absolute or start with `/` (e.g., `/var/www/html/lang/override`) or be a relative path starting with a letter. The `getOverridePaths` function converts relative paths to `base_path($line)` if they do not start with `/` and do not match a Windows drive pattern. |
| **PHP files with syntax errors** | The error is logged in `system.log` and the file is skipped without disabling other files. Check the log to fix the error. |
| **Translation not overriding despite using `set()`** | Ensure that the language you pass (`$locale`) matches the language used when calling `Lang::get`. Also ensure that `RainLab.Translate` has correctly set the language. |
| **Using `addPath()` does not affect existing paths** | The path is appended to the end of the list, meaning it has lowest priority. If you want it to have higher priority, use `array_unshift` or add the path before the object is created (via settings). |

---

### 13. Relationship with `mergeToTranslatorV1` and `mergeToTranslatorV2`

The current code contains additional methods (`mergeToTranslatorV1`, `mergeToTranslatorV2`) but they are **not used** in the current version. The active method is `mergeToTranslator` which uses `$translator->set()`. The other methods exist for reference in case alternative approaches are needed, but they are not recommended because `set()` is the most compatible with October's system.

---

### 14. Conclusion

`TranslationOverrideManager` is a powerful and flexible tool for overriding translations in OctoberCMS projects. Thanks to its support for multiple paths, caching, integration with settings, and robust error handling, developers can customise any text in the system without worrying about plugin updates. The class also provides clear methods for programmatic control, making it suitable for both small applications and large multilingual systems.

---

**References:**
- [Nano.TranslateExtended General Documentation](../Docs-Nano.TranslateExtended-en.md)
- [Translation Override Guide](../Docs-Translation-Override-Guide-en.md)
