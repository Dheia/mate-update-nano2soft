## 📘 Translation Override Guide

**Version:** 1.0.12  
**Plugin:** Nano.TranslateExtended  
**Purpose:** Customise any translation in the system without modifying original plugin or core files.

---

### 1. What is Translation Override?

Translation override is a mechanism that allows you to replace any translated text (e.g., buttons, titles, messages) coming from a specific plugin or the core system by placing custom translation files in designated folders. These files are not affected by plugin updates, thus preserving your customisations.

**Example:**  
If the `nano.sitemanager` plugin displays a button labelled "Manage Store" and you want to change it to "Manage Branch", you can do that via an override file without touching the original plugin files.

---

### 2. Requirements

- OctoberCMS or WinterCMS version 2.0+.
- `RainLab.Translate` plugin installed and activated.
- `Nano.TranslateExtended` plugin installed and activated (version 1.0.12 or later).
- Access to the backend to modify plugin settings (optional, you can use `.env` instead).

---

### 3. Enabling the System

#### Method 1: Via Backend (recommended for production)

1. Go to **Settings** → **Translate Extended** (under the RainLab.Translate category).
2. In the general tab, find the **Translation Overrides** section.
3. Enable **Enable translation overrides**.
4. Set the **Cache duration** (in minutes). 60 is suitable for normal sites. In development, you can set 0 to disable caching.
5. In the **Override paths** field, leave the default paths (`lang/override` and `plugins/nano/translateextended/override/lang`) or add your own custom path (one per line).
6. Save the settings.

#### Method 2: Via Environment Variables (`.env`)

Add the following lines to your `.env` file in the project root:

```ini
NANO_TRANSLATE_ENABLE_OVERRIDES=true
NANO_TRANSLATE_CACHE_MINUTES=60
NANO_TRANSLATE_OVERRIDE_PATHS="lang/override,plugins/nano/translateextended/override/lang"
```

> **Note:** Environment variables have the highest priority over database settings or `config.php`.

#### Method 3: Via `config.php`

If you do not want to use the database or `.env`, you can modify the plugin's configuration file:

`plugins/nano/translateextended/config/config.php`

```php
'enable_overrides' => true,
'override_cache_minutes' => 60,
'override_paths' => [
    base_path('lang/override'),
    plugins_path('nano/translateextended/override/lang'),
],
```

> **Note:** Database settings take precedence over `config.php` if they exist.

---

### 4. Creating Override Files

#### 4.1. Folder Structure

By default, the system searches in two paths:

- **Project‑wide path:** `lang/override/{locale}/`
- **Inside the plugin:** `plugins/nano/translateextended/override/lang/{locale}/`

You can use either. It is recommended to use the project‑wide path to keep all customisations in one place.

#### 4.2. Creating the files

Assume you want to override translations for Arabic (`ar`) and English (`en`).

**Steps:**

1. Create the folders:
   ```bash
   mkdir -p lang/override/ar
   mkdir -p lang/override/en
   ```

2. Inside each folder, create a `.php` file (you can name it whatever you like, e.g., `custom.php` or `overrides.php`).

3. Write the content of the file as a PHP array.

**Example `lang/override/ar/custom.php`:**
```php
<?php

return [
    'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'Manage Another Store',
    'tss.inventory::lang.products.form.use_case_options.excellent' => 'Excellent',
    'tss.inventory::lang.products.form.use_case_options.good' => 'Good',
];
```

**Example `lang/override/en/custom.php`:**
```php
<?php

return [
    'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'Manage Another Store',
    'tss.inventory::lang.products.form.use_case_options.excellent' => 'Excellent',
    'tss.inventory::lang.products.form.use_case_options.good' => 'Good',
];
```

#### 4.3. Key Format

- **Regular key:** `'messages.welcome'` (without `::`) – belongs to the general translation in the `lang/xx/` folder.
- **Namespaced key:** `'nano.sitemanager::lang.site_switcher.btn.manage_shop'` – belongs to a specific plugin (`nano.sitemanager`), file `lang`, group `site_switcher`, and key `btn.manage_shop`.

> **Important:** Use exactly the same key as you would use in `Lang::get()`.

#### 4.4. Using Nested Arrays (optional)

You can organise translations hierarchically:

```php
<?php

return [
    'nano' => [
        'sitemanager' => [
            'lang' => [
                'site_switcher' => [
                    'btn' => [
                        'manage_shop' => 'Manage Another Store'
                    ]
                ]
            ]
        ]
    ]
];
```

The system automatically flattens the array to dot notation.

---

### 5. How Override Works (Internally)

Once the page loads, `TranslationOverrideManager` does the following:

1. Reads settings (env → db → config).
2. Searches all specified paths for `.php` files inside the current language folder.
3. Flattens nested arrays into dot notation keys.
4. Merges all overrides into October's translator via `$translator->set($key, $value, $locale)`.
5. Stores the result in cache for the specified duration.
6. On any `Lang::get($key)` call, the translator returns the overridden value instead of the original.

---

### 6. Cache Management

#### 6.1. When is cache automatically cleared?

- When plugin settings are saved (via the settings page).
- When the language is changed (event `locale.changed`).

#### 6.2. Manually clearing cache

**Artisan command:**
```bash
php artisan cache:clear
```
> This clears the entire application cache, not only translation overrides.

**From code:**
```php
$manager = app('translateextended.translationOverride');
$manager->clearCache();        // clears all languages
$manager->clearCache('ar');    // clears only Arabic
```

#### 6.3. Temporarily disabling cache (for development)

In `.env`:
```ini
NANO_TRANSLATE_CACHE_MINUTES=0
```
Or in settings: set `override_cache_minutes = 0`.

---

### 7. Practical Examples

#### Example 1: Change button text "Manage Store" in SiteManager plugin

**Original translation:** `nano.sitemanager::lang.site_switcher.btn.manage_shop` → default "Manage Store".

**We want to change it to:** "Manage Branch".

**Steps:**

1. Create `lang/override/ar/custom.php`.
2. Add:
   ```php
   <?php return [
       'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'Manage Branch',
   ];
   ```
3. Clear cache (if enabled).
4. Open the page where the button appears – you will see the new text.

#### Example 2: Add a new option in a dropdown (use_case_options) in Inventory plugin

**Key:** `tss.inventory::lang.products.form.use_case_options.excellent` (does not exist originally, but we add it).

**File:**
```php
'tss.inventory::lang.products.form.use_case_options.excellent' => 'Excellent',
```

Then in code (or template) you can use:
```php
Lang::get('tss.inventory::lang.products.form.use_case_options.excellent')
```
It will display "Excellent".

#### Example 3: Override a general translation (without namespace)

Assume the system uses `'backend::lang.form.save'` and you want to change it to "Save Changes".

```php
'backend::lang.form.save' => 'Save Changes',
```
(With namespace `backend`, full key.)

---

### 8. Troubleshooting

| Issue | Solution |
|-------|----------|
| **Override not working despite files existing** | 1. Ensure `enable_overrides` is true. 2. Check the path is correct (does `lang/override/ar` exist?). 3. Clear cache (`php artisan cache:clear`). 4. Check `storage/logs/system.log` for errors. |
| **Old translation appears after modifying file** | Due to cache. Either wait for the cache to expire or clear it manually. |
| **Error `Too few arguments to function ...`** | The `locale.changed` event in previous versions expected two arguments. Version 1.0.12 fixes this. Make sure you have updated `Plugin.php`. |
| **File is not read even though it exists** | Ensure the file extension is `.php` and it contains no syntax errors. Try `php -l custom.php`. |
| **Override works in frontend but not in backend** | In the backend, translations may be temporarily disabled to ease editing. That is normal. If you need to force translation in the backend, you can directly call `$manager->loadForLocale($locale)`. |
| **Value `null` or empty string appears** | Make sure the key is exactly correct (case‑sensitive). Try printing `Lang::get('your.key')` to see if it returns the key itself (meaning not found) or an empty value. |

---

### 9. Advanced Tips

#### 9.1. Adding an extra path dynamically

In `Plugin.php` or anywhere after initialisation:

```php
use Nano\TranslateExtended\Classes\TranslationOverrideManager;

$manager = app(TranslationOverrideManager::class);
$manager->addPath(base_path('my/custom/overrides'));
```

#### 9.2. Reloading translations after changing files without waiting for cache

```php
$manager->reload('ar');
```

#### 9.3. Using overrides with JSON translations (e.g., translatable fields in API)

You can override values returned via API in the same way, because `Lang::get` is also used in Transformers.

#### 9.4. Combining overrides with `TranslatableContentCaching`

If you use the `TranslatableContentCaching` behaviour to obtain multiple translations in API, overrides also affect those values, because the internal methods use `Lang::get`.

---

### 10. Conclusion

The translation override system in `Nano.TranslateExtended` is an elegant and powerful solution for customising user interface texts without touching plugin files. Thanks to its flexibility (multiple paths, caching, UI and env configuration), you can easily apply it in any project, small or large, while ensuring your customisations survive plugin updates.

**Quick start steps:**

1. Enable the system in settings or `.env`.
2. Create a folder `lang/override/{locale}/`.
3. Place a `.php` file that returns an array of keys and values you want to override.
4. Clear cache.
5. Enjoy your new translations.

---

**References:**

- [Nano.TranslateExtended General Documentation](./Docs-Nano.TranslateExtended-en.md)
- [DynamicAddIncludeTranslatableApiFields Behaviour](./Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-en.md)
- [TranslatableContentCaching Behaviour](./Behaviors/Docs-TranslatableContentCaching-en.md)
- [Advanced API Translations Guide](./Docs-API-Translations-Guide-en.md)
- [TranslationOverrideManager Class Documentation](./Classes/Docs-TranslationOverrideManager-Class-en.md)
