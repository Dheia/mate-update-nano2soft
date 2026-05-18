# User Preferences System Documentation (Preference & UserPreference)

## Overview

The preferences system in `Nano.UserPlus` provides a mechanism exactly similar to the backend user preferences (`Backend\Models\Preference`), but oriented towards **frontend** users. The system consists of three main components:

| Component | Role |
| :--- | :--- |
| `Preference` (model) | Developer interface: defines preference settings (fields, default values, validation). |
| `UserPreference` (model) | Actual database storage (`frontend_user_preferences` table). |
| `UserPreferencesModel` (behavior) | The link between `Preference` and the current user record. |

## Database Structure

Table `frontend_user_preferences`:

| Column | Description |
| :--- | :--- |
| `id` | Unique identifier |
| `user_id` | User ID (linked to `users`) |
| `namespace` | Settings namespace (e.g., `frontend`) |
| `group` | Settings group (e.g., `frontend`) |
| `item` | Settings item (e.g., `preferences`) |
| `value` | JSON value storing all settings |

**Unique key:** `namespace + group + item + user_id`

## `Preference` Model

### Description

This model is intended for developers to create custom settings pages for users. It inherits from `Model` and uses the `UserPreferencesModel` behavior.

### Important Properties

- `$settingsCode`: `'frontend::frontend.preferences'` – defines the unique key for settings.
- `$settingsFields`: `'fields.yaml'` – YAML file for defining form fields.
- `$implement`: `[UserPreferencesModel::class]` – links the model to the user record.

### Main Functions

#### `initSettingsData()`

Called when settings are accessed for the first time (if they do not exist). Sets default values:

- `locale`: from `app.locale`.
- `timezone`: from `backend.timezone` or `app.timezone`.
- Editor settings (font_size, word_wrap, theme...).

#### `useDefaults()` (private)

If settings exist but some keys are missing (due to an update), merges default values from `BasicHelper::getPreferenceInitSettingsData()` on `afterFetch()`, provided that `nano.userplus::preference.use_defaults` is enabled.

#### `afterFetch()`

Event after fetching the record from the database. Calls `useDefaults()` if necessary.

#### `formSelectRefType($name, $selectedValue, $options)`

Helper function to generate a `select` field for the account types list (`ref_type`). Internally uses `getRefTypeList()`.

#### `formSelectGender($name, $selectedValue, $options)`

Similar, but for the gender list (`gender`).

#### `getRefTypeList()` (static)

Returns an array of all supported account types. **Types depend on `config.php` settings and the presence of related plugins.**

**Example:**
```php
$types = Preference::getRefTypeList();
// ['user' => 'Regular User', 'delivery_morph_key' => 'Delivery Worker', ...]
```

#### `getGenderOptions()` (static)

Returns gender options:
```php
[
    'male' => 'Male',
    'female' => 'Female',
]
```

#### `setAppLocale()` / `setAppFallbackLocale()`

Sets application language and fallback language based on user preference (uses `Session` for temporary storage).

#### `applyConfigValues()`

Applies user settings to `Config` (e.g., `app.locale`).

#### `resetDefault()`

Resets settings to default values and clears the session.

#### `getLocaleOptions()`

Returns a list of supported languages with flags (large array from `ar`, `en`, `fr`...).

#### `getTimezoneOptions()`

Returns list of time zones in the format `(UTC +03:00) Asia/Riyadh`.

---

## `UserPreference` Model

### Description

The actual model that deals with the `frontend_user_preferences` table. Inherits from `October\Rain\Auth\Models\PreferencesBase` and customizes it for frontend users.

### Properties

- `$table`: `'frontend_user_preferences'`
- `$timestamps`: `false`

### Main Functions

#### `resolveUser($user)`

Resolves the user: if not passed, fetches the current user from `FrontendAuth`. Throws `SystemException` if no user is logged in.

#### `scopeOnNamespace($query, $name)`

Scope to search by `namespace`.

#### `scopeOnGroup($query, $name)`

Scope to search by `group`.

#### `scopeOnItem($query, $name)`

Scope to search by `item`.

#### `scopeIsUserPreference($query)`

Predefined scope: `namespace=frontend, group=frontend, item=preferences`.

#### `scopeIsLocationPreference($query)`

Predefined scope: `namespace=nano.sitemanager, group=location, item=manager`.

#### `getAllConfigUser($user = null)` (static)

Fetches all settings of a given user (or current) and organizes them into a multidimensional array:
```
[
    'namespace_group_item' => { value object },
    ...
]
```

#### `getConfigByUserId($userId)` (static)

Like the previous, but with a specific user ID.

---

## `UserPreferencesModel` Behavior

### Description

A behavior added to any model (such as `Preference`) to make it handle the current user's preferences. Inherits from `System\Behaviors\SettingsModel` but redirects storage to `UserPreference`.

### Overridden Functions

| Function | Description |
| :--- | :--- |
| `instance()` | Fetches or creates the preference record for the current user. Temporarily stored in `$instances`. |
| `isConfigured()` | Checks if preferences have been previously set up. |
| `getSettingsRecord()` | Fetches the `UserPreference` record matching the key (`settingsCode`) and the current user. Uses `remember(1440)` for caching. |
| `beforeModelSave()` | Before saving, sets `item`, `group`, `namespace`, `user_id` and converts `fieldValues` to `value` (JSON). |
| `isKeyAllowed()` | Allows passing `namespace` and `group` in addition to normal field keys. |
| `getCacheKey()` | Generates a unique cache key combining `recordCode` and `user_id`. |

---

## Practical Examples

### 1. Creating a Custom User Settings Page

```php
// Model MyUserSettings.php
class MyUserSettings extends Model
{
    public $implement = [\Nano\UserPlus\Behaviors\UserPreferencesModel::class];
    public $settingsCode = 'myplugin::settings';
    public $settingsFields = 'fields.yaml';

    public function initSettingsData()
    {
        $this->theme = 'dark';
    }
}
```

### 2. Accessing Current User Settings

```php
$settings = MyUserSettings::instance();
echo $settings->theme; // 'dark'
$settings->theme = 'light';
$settings->save();
```

### 3. Fetching User Location Preferences

```php
$locationPref = UserPreference::forUser()
    ->scopeIsLocationPreference($query)
    ->first();
```

---

## Important Notes

- Caching: `getSettingsRecord` uses `remember(1440)` to reduce queries.
- `useDefaults()` in `Preference` ensures that any new field added later will receive a default value without needing to reset settings.
- The `getAllConfigUser` function is useful for displaying all user settings in the backend panel or for export.