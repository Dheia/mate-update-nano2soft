## 2026-06-10 – 2026-06-13

**Nano.TranslateExtended Plugin Updates – Version 1.0.12**  
**Added a complete translation override system with multi‑path support, caching, backend settings, and environment variables**

### Summary of Updates

Version **1.0.12** introduces a complete and flexible system to override any translation in any plugin or the core system, without modifying the original translation files. The system relies on the `TranslationOverrideManager` class that reads translations from dedicated folders, caches them, and merges them into the translation system using the `set()` method of `October\Rain\Translation` to ensure effective override. The system is controlled via a new backend settings interface, environment variables, or the `config.php` file.

---

## Nano.TranslateExtended v1.0.12 – Translation Override System

### Release Goals

- **Provide a professional translation override mechanism** without modifying original plugin or core files.
- **Support multiple override folders** with priority (the last processed path takes precedence).
- **Integrate the override system with plugin settings** so it can be enabled/disabled, duration and paths configured via the UI.
- **Full support for environment variables** to control every setting without relying on the database.
- **Smart caching** to avoid reading files on every request, with the ability to disable caching during development.
- **Comprehensive exception handling** and logging to `system.log` for easier debugging.
- **Full compatibility with `October\Rain\Translation::set()`** to ensure correct replacement of translations.

### New Features and Improvements

---

#### First: `TranslationOverrideManager` – Override Manager

##### 1. Reading files from multiple paths

- Searches specified paths (e.g., `lang/override` and `plugins/nano/translateextended/override/lang`).
- Supports absolute and relative paths (relative paths are automatically converted to `base_path()`).
- Merges all PHP files found in the folder (any file name) and subfolders.
- Converts nested arrays to dot notation (e.g., `['a' => ['b' => 'c']]` becomes `['a.b' => 'c']`).

##### 2. Merging translations using `set()`

- Uses `$translator->set($key, $value, $locale)` provided by OctoberCMS to ensure override even if translations were already loaded.
- Supports regular keys (`messages.welcome`) and namespaced keys (`nano.plugin::lang.key`).
- Logs errors if adding any translation fails.

##### 3. Caching

- Stores translations read from files for a configurable duration (default 60 minutes).
- Can disable cache by setting `override_cache_minutes = 0`.
- Automatically clears cache when plugin settings are saved.
- Manual cache clearing via `clearCache()` method or the `locale.changed` event.

##### 4. General helper methods

- `loadAll($locale = null)`: loads translations for all available languages or a single language.
- `loadForLocale($locale)`: loads translations for a specific language and merges them.
- `reload($locale = null)`: clears cache and reloads the language.
- `addPath($path)`: dynamically adds an extra path.
- `getPaths()`, `isEnabled()`, `getCacheMinutes()` for diagnostics.

##### 5. Exception handling and logging

- Every method is wrapped in `try/catch` with error logging to `Log::error` or `Log::warning`.
- Handles `Throwable` to catch all error types.
- Logs information when a translation file fails to include or does not return an array.

---

#### Second: Updates to the `Settings` Model

##### 1. New fields in `fields.yaml`

- `enable_overrides` (switch): enables/disables the override system.
- `override_cache_minutes` (number): cache duration (in minutes).
- `override_paths` (textarea): list of paths (one per line).

##### 2. New helper methods in `Settings`

- `getWithEnvPriority($key, $envVarName, $default)`: retrieves value according to priority (env > database > config).
- `getOverridePaths()`: returns an array of paths after converting relative to absolute and supporting multi‑line strings.
- Improved boolean value parsing from `env` (e.g., `false`, `true`, `null`).

##### 3. Improved compatibility with PHP 7.4+

- Replaced `str_starts_with()` with `strpos() === 0` to ensure compatibility with older versions.

---

#### Third: New `config.php` settings

The following keys have been added to the plugin's `config.php` file:

```php
'enable_overrides'          => env('NANO_TRANSLATE_ENABLE_OVERRIDES', true),
'override_paths'            => [
    base_path('lang/override'),
    plugins_path('nano/translateextended/override/lang'),
],
'override_cache_minutes'    => env('NANO_TRANSLATE_CACHE_MINUTES', 60),
'forceDefaultLocale'        => env('NANO_TRANSLATE_FORCE_LOCALE', null),
'prefixDefaultLocale'       => env('NANO_TRANSLATE_PREFIX_LOCALE', true),
'disableLocalePrefixRoutes' => env('NANO_TRANSLATE_DISABLE_PREFIX_ROUTES', false),
```

---

#### Fourth: Supported Environment Variables (`.env`)

| Variable | Purpose | Example |
|----------|---------|---------|
| `NANO_TRANSLATE_ENABLE_OVERRIDES` | Enable/disable the system | `true` / `false` |
| `NANO_TRANSLATE_CACHE_MINUTES` | Cache duration in minutes | `120` |
| `NANO_TRANSLATE_OVERRIDE_PATHS` | Override paths (single line, comma or newline separated) | `lang/override,plugins/nano/translateextended/override/lang` |
| `NANO_TRANSLATE_FORCE_LOCALE` | Force a fixed locale on the site | `ar` |
| `NANO_TRANSLATE_PREFIX_LOCALE` | Prefix locale in URLs (even for default locale) | `true` / `false` |
| `NANO_TRANSLATE_DISABLE_PREFIX_ROUTES` | Completely disable locale‑based routing | `true` / `false` |

---

#### Fifth: Event Listeners

- **`backend.form.beforeSave`**: clears override cache when plugin settings are saved.
- **`locale.changed`**: reloads translations for the new language when the language changes.
- (Optional) **`rainlab.translate.beforeResolveLocale`** and **`LocaleMiddleware::$prefixDefaultLocale`** are supported in `applyLocaleSettings` to apply `forceDefaultLocale` and `prefixDefaultLocale`.

---

#### Sixth: Example Override Files

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

---

### How to Use (for Developers)

#### 1. Enable the system

- From the backend: `Settings → Translate Extended → Enable translation overrides`.
- Or via `.env`: `NANO_TRANSLATE_ENABLE_OVERRIDES=true`.

#### 2. Create override files

- Create the folder `lang/override/{locale}/` in the project root.
- Put any `.php` file that returns an array of keys and values you want to override.

#### 3. Clear cache

- Save plugin settings (automatic).
- Or run `php artisan cache:clear`.
- Or call `$manager->clearCache()` programmatically.

#### 4. Use programmatically

```php
$manager = app('translateextended.translationOverride');
$manager->addPath(base_path('my/extra/path'));
$manager->reload('ar');
```

---

### Upgrading from Previous Versions

1. Update plugin files via `php artisan plugin:refresh Nano.TranslateExtended` or by copying files manually.
2. Run `php artisan cache:clear` to clear old cache.
3. (Optional) Add new environment variables to `.env`.
4. Go to the plugin settings page and adjust the values as needed.

> **Note:** There are no database schema changes; the upgrade is safe and does not affect existing data.

---

### Common Issues and Solutions

| Problem | Solution |
|---------|----------|
| Overrides not working despite files existing | Ensure `enable_overrides` is true and paths are correct. Clear cache. |
| Error `Too few arguments to function ...` | The `locale.changed` event needs `$oldLocale = null` (fixed in this version). |
| Old translation appears immediately after modification | Clear cache or use `$manager->reload()`. |
| `str_starts_with` causes error on PHP 7.4 | Replaced with `strpos() === 0`. |

---

### Version Summary (1.0.12)

| Version | Key Features |
|---------|---------------|
| **Nano.TranslateExtended 1.0.12** | Complete translation override system, multiple paths, configurable caching, UI and environment variable settings, full compatibility with `October\Rain\Translation::set()`, exception handling and logging. |

---

### Upgrade Requirements

1. **Files to update**:
   - `Plugin.php`
   - `classes/TranslationOverrideManager.php`
   - `models/Settings.php`
   - `config/config.php`
   - `fields.yaml`
   - `version.yaml`

2. **No new database migrations** – only optional settings fields.

3. **Dependencies**:
   - `RainLab.Translate` (any recent version)
   - `OctoberCMS` ≥ 2.0 or `WinterCMS` ≥ 1.2

4. **Recommended additional `.env` settings**:
   ```
   NANO_TRANSLATE_ENABLE_OVERRIDES=true
   NANO_TRANSLATE_CACHE_MINUTES=60
   NANO_TRANSLATE_OVERRIDE_PATHS="lang/override,plugins/nano/translateextended/override/lang"
   ```

5. **Compatibility testing**:
   - Create a simple override file and verify the new translation appears.
   - Test language change (`locale.changed`) and automatic reload.
   - Test enabling/disabling the system and saving settings.

---

### Conclusion

Version **1.0.12** represents a major leap in the plugin's capabilities, giving developers the power to modify any translation in the system without touching original files. The system is designed to be flexible, high‑performance, and easy to use, with full support for OctoberCMS's usual tools (settings, events, cache). It can be used to customise user interfaces, fix incorrect translations, or even create client‑specific translations without conflicting with plugin updates.

---

**Reference Documentation**:
- [Nano.TranslateExtended General Documentation](./docs/TranslateExtended/Docs-Nano.TranslateExtended-en.md)
- [DynamicAddIncludeTranslatableApiFields Behaviour](./docs/TranslateExtended/Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-en.md)
- [TranslatableContentCaching Behaviour](./docs/TranslateExtended/Behaviors/Docs-TranslatableContentCaching-en.md)
- [Advanced API Translations Guide](./docs/TranslateExtended/Docs-API-Translations-Guide-en.md)
- [Translation Override Guide](./docs/TranslateExtended/Docs-Translation-Override-Guide-en.md)
- [TranslationOverrideManager Class Documentation](./docs/TranslateExtended/Classes/Docs-TranslationOverrideManager-Class-en.md)
- [Configuration file `config.php`](./plugins/nano/translateextended/config/config.php)
- [Settings Model `Settings`](./plugins/nano/translateextended/models/Settings.php)
