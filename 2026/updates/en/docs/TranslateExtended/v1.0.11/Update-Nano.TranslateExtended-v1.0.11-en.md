## 2026-06-03 – 2026-06-05

**Updates to the `Nano.TranslateExtended` plugin – Version 1.0.11**  
**Comprehensive improvement of API translation calls and handling of translatable content**

### Summary of Updates

The `Nano.TranslateExtended` plugin underwent a major update in version **1.0.11** that focused on fixing the issue where translations did not appear when including `translatable_fields` in API responses, and expanding the flexibility of fetching translations by supporting dynamic field and language selection via `Input`, configuration, and public properties. It also improved the `TranslatableContentCaching` behaviour to allow programmatic control over translatable fields, corrected the fallback logic for the default locale, added support for decoding JSON with error logging, and improved compatibility with the `RainLab.Translate` data structure.

---

## Nano.TranslateExtended v1.0.11 – Improved API Translations and Core Fixes

### Release Objectives

- **Fix the issue of translations not appearing** when `translatable_fields` is included in API calls.
- **Provide full flexibility in specifying the fields and languages** for which translations should be fetched via `Input`, configuration, or public properties.
- **Add new `includes`** such as `translatable_attributes` and `translatable_dirty_locales` to support advanced use cases.
- **Improve `TranslatableContentCaching`** by adding dynamic methods to control fields, correcting the fallback logic, and supporting JSON decoding with error logging.
- **Unify the interface for passing fields** as a comma‑separated string (`title,content`) or the word `all`/`*`.
- **Enhance compatibility with `RainLab.Translate`'s translation storage structure.**

### New Features and Improvements

---

#### First: Updates to `DynamicAddIncludeTranslatableApiFields`

##### 1. Enhanced `includeTranslatableFields` method

- **Flexible parameter reading** according to the priority: public property ← configuration ← `Input` ← default value.
- **Support for specifying fields (`fields`) in multiple formats**:
  - Plain array.
  - Comma‑separated string: `"title,content"`.
  - The word `"all"` or `"*"` to fetch all translatable fields.
- **Support for specifying languages (`locales`)** with the same flexibility.
- **Call `$item->transCollectFields()`** before calling `getTranslationsInFormat()` to ensure the field list is updated dynamically.
- **Handle `exclude`** from `Input` or configuration to exclude certain keys from the result.

##### 2. Added new `includes` to the Transformer

- `translatable_attributes`: returns the list of translatable field names in the model (`getTranslatableAttributes`).
- `translatable_dirty_locales` (optionally commented out): returns information about dirty locales and originals.

##### 3. Improved `getConfigValue` method

- Added support for reading from `Input` via the key `translated_fields` as an alternative to `translatable_fields.fields`, providing a simpler way for developers.

##### 4. Example API usage

```json
GET /api/orders/orders/1?include=translatable_fields&translatable_fields.fields=title,content&translatable_fields.locales=en,ar
```

Or using the shortcut:
```json
GET /api/orders/orders/1?include=translatable_fields&translated_fields=title,content
```

---

#### Second: Updates to `TranslatableContentCaching`

##### 1. Added programmatic methods to control fields

- `getTransCachedFields()`: returns the current list of translatable fields.
- `setTransCachedFields($fields)`: dynamically sets the list, supporting comma‑separated strings or `'all'`/`'*'`.

##### 2. Improved `getTranslated` method

- Added the `$useInBackend` parameter (default `false`) to allow translation even in the backend environment when needed.
- Corrected the fallback logic for the default locale: when the requested `$locale` matches the default locale, `$fallback` is set to `true` to ensure fallback to the original value if no translation exists.
- Added a helper function `processTranslatableJsonableValue` with JSON error logging in debug mode.

##### 3. Improved `getTranslatedAttributes` method

- Made the search for the translation more compatible with `RainLab.Translate`’s structure, which stores `locale` inside an `attributes` array.
- Added a fallback to search via the direct property `$item->locale`.

##### 4. Improved `getTranslationsInFormat` method

- Supports passing `$fields` as a comma‑separated string or `'all'`/`'*'`, converting it to an array inside the method.
- Unified logic with the rest of the plugin.

##### 5. Improved dynamic model registration (`registerModel`)

- Added a static function `isPropertyExists` to check for property existence before adding it.
- Avoid re‑adding existing properties.

##### 6. Helper function `isPropertyExists`

- A static function that checks whether a property exists on an object, with support for `propertyExists` (for `October\Rain\Extension\ExtensionBase` extensions).

---

### Third: Updated `version.yaml`

Version `1.0.11` was added with a brief description of the updates:

```yaml
1.0.11:
    - Improved translation API includes with flexible field/locale parameters
    - Added support for passing fields as comma-separated string in translatable_fields include
    - Added new includes: translatable_attributes and translatable_dirty_locales
    - Enhanced TranslatableContentCaching with dynamic field setter/getter
    - Improved fallback handling for default locale in getTranslated method
    - Better compatibility with RainLab.Translate attribute data structure
    - Added error logging for JSON decoding issues in debug mode
    - Optimized cache key generation and invalidation
```

---

### Application Examples

#### 1. Using the new includes in the API (via inclusion)

**Request to fetch translations for only two fields with one language:**
```http
GET /api/products/products/1?include=translatable_fields&translatable_fields.fields=title,description&translatable_fields.locales=ar
```

**Using the shortcut `translated_fields`:**
```http
GET /api/products/products/1?include=translatable_fields&translated_fields=title
```

**Fetch all translatable fields and exclude a specific field:**
```http
GET /api/products/products/1?include=translatable_fields&translatable_fields.fields=all&translatable_fields.exclude=seo_keywords
```

#### 2. Using `includeTranslatableAttributes` to get only the field names

```http
GET /api/products/products/1?include=translatable_attributes
```

**Response:**
```json
{
    "data": {
        "id": 1,
        "translatable_attributes": ["title", "description", "seo_title"]
    }
}
```

#### 3. Programmatic control in `TranslatableContentCaching`

```php
// Dynamically set the translatable fields
$order = Order::find(1);
$order->setTransCachedFields('title,content,notes');

// Get translations in a custom format
$translations = $order->getTranslationsInFormat(
    fields: 'title',
    format: TranslatableContentCaching::FORMAT_GROUP_BY_FIELD_UNDER_ALL,
    locales: ['en', 'ar'],
    includeOriginal: true,
    useFallback: false
);
```

#### 4. Enabling translation in the backend (special cases)

```php
// Force translation even in the backend environment
$value = $model->getTranslated('title', null, 'ar', true, true, true);
```

---

## Version Summary (1.0.11)

| Version | Key Features |
|---------|---------------|
| **Nano.TranslateExtended 1.0.11** | Fixed the `translatable_fields` issue in the API, added flexible field/language specification, added `translatable_attributes` and `translatable_dirty_locales`, improved `TranslatableContentCaching` with dynamic methods and fallback logic for the default locale, better compatibility with `RainLab.Translate`, JSON error logging. |

---

### Upgrade Requirements

1. **Update the files**:
   - `DynamicAddIncludeTranslatableApiFields.php`
   - `TranslatableContentCaching.php`
   - `version.yaml`

2. **No new migrations**: This version does not require database changes.

3. **Dependencies**:
   - `RainLab.Translate` (latest version)
   - `OctoberCMS` ≥ 2.0 or `WinterCMS`

4. **Optional settings**:
   - You can add custom settings in the plugin’s `config.php` under the key `nano.translateextended::translatable_fields` to set default values for `fields`, `format`, `locales`, `includeOriginal`, `useFallback`, `useCache`, `exclude`.

5. **Compatibility testing**:
   - Test API calls with `include=translatable_fields` and various parameters.
   - Test models using `TranslatableContentCaching` and verify that `getTranslated` works in both frontend and backend.
   - Test JSON decoding for translatable JSON fields.

---

### Conclusion

Version **Nano.TranslateExtended 1.0.11** represents a significant leap in how translations are handled both via the API and in code. Developers no longer need to write complex queries or manually extend the Transformer; they can now use `translatable_fields` with the ability to specify fields and languages directly via `Input`, and obtain translations in a clean, flexible format. Many behavioural issues in `TranslatableContentCaching` have been fixed to ensure translations work as expected in all scenarios, including the default locale and fallback to original values. The code is fully documented and includes performance and compatibility improvements.

---

**Reference documentation**:
- [`Nano.TranslateExtended` plugin documentation](./docs/TranslateExtended/Behaviors/Docs-Nano.TranslateExtended-en.md)
- [`DynamicAddIncludeTranslatableApiFields` behaviour documentation](./docs/TranslateExtended/Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-en.md)
- [`TranslatableContentCaching` behaviour documentation](./docs/TranslateExtended/Behaviors/Docs-TranslatableContentCaching-en.md)
- [Enhanced Translations API Usage Guide](./docs/TranslateExtended/Docs-API-Translations-Guide-en.md)
