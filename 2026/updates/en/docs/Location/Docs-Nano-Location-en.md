# `Nano.Location` Plugin Documentation

**Version:** 1.0.16  
**Author:** Dheia Ali  
**License:** Proprietary to Nano2Soft  
**Last updated:** 2025-04-30

---

## Table of Contents

1. [Introduction](#introduction)
2. [Requirements](#requirements)
3. [Installation and Upgrade](#installation-and-upgrade)
4. [Main Features](#main-features)
5. [Database Structure](#database-structure)
6. [Models](#models)
7. [Behaviors](#behaviors)
8. [Traits](#traits)
9. [Backend Interface](#backend-interface)
10. [Plugin Settings](#plugin-settings)
11. [Permissions](#permissions)
12. [Components](#components)
13. [Helper Functions](#helper-functions)
14. [Events](#events)
15. [Internationalisation](#internationalisation)
16. [Practical Examples](#practical-examples)
17. [Troubleshooting](#troubleshooting)
18. [References](#references)

---

## Introduction

The `Nano.Location` plugin is an advanced plugin for October CMS systems that aims to provide a comprehensive geographic structure including **Countries**, **States**, **Directorates**, as well as **Addresses**, **Nationalities**, **Languages**, **Religions**, and **Traceroutes**. The plugin is designed to be flexible and extensible, with full support for the backend and an accompanying `Nano.LocationApi` plugin for a complete API.

The plugin builds on `RainLab.Location` as a base, but extends it by adding:
- **Directorates**: A third level of geographic division (e.g., districts, sub‑districts).
- **Addresses**: A flexible polymorphic model for managing addresses for many entity types.
- **Nationalities**: A list of nationalities with codes and flags.
- **Traceroutes**: Logging of users’ geographic movements.
- **Multimedia support**: Adding images, videos, audio files, and documents to geographic entities (added in version 1.0.16).
- **Multi‑language support**: Via `RainLab.Translate`.

---

## Requirements

- **October CMS** version 3.x or 2.x (with some limitations).
- **PHP** >= 8.0.
- **Required plugins** (declared in `$require`):
  - `RainLab.Location`
  - `RainLab.User`
- **Recommended plugins**:
  - `RainLab.Translate` (for translating country, state, and directorate names).
  - `Nano.LocationApi` (to provide a complete API interface).
  - `Nano.ActivityLog` (to log activities on geographic entities).

---

## Installation and Upgrade

### Fresh installation

```bash
php artisan plugin:install Nano.Location
php artisan nano:up
```

### Upgrade from a previous version

```bash
composer update nano/location
php artisan nano:up
php artisan cache:clear
```

### Publish configuration file (optional)

```bash
php artisan config:publish Nano.Location
```

This will create the configuration file at `config/nano/location/config.php`.

---

## Main Features

| Feature | Description |
| :--- | :--- |
| **Countries** | Extends `RainLab\Location\Models\Country` by adding fields: `wikiDataId`, `latitude`, `longitude`, `sort_order`, and multiple media relationships. |
| **States** | Extends `RainLab\Location\Models\State` by adding `wikiDataId`, `latitude`, `longitude`, `sort_order`, and a `hasMany` relationship to `Directorate`. |
| **Directorates** | A new model representing the third administrative level, with translations, enabling, and sorting. |
| **Addresses** | The `Addres` model (note the intentional spelling for compatibility) supports polymorphic relationships with any other model, with integrated geographic fields and the ability to set a default address. |
| **Nationalities** | `Nationality` model contains: `name`, `code`, `nationality_code`, `country_code`, `iso_code`, `phone_code`, `region`, `subregion`, `flag_emoji`, with support for a main image. |
| **Traceroutes** | The `Traceroute` model records user visits with geographic data (IP, country, city, coordinates). |
| **Languages & Religions** | Helper functions `LanguagesHelper` and `ReligionsHelper` provide ready‑to‑use lists of known languages and religions. |
| **Multimedia** | `attachOne` and `attachMany` support for images, videos, audio, and files on countries, states, and directorates (from version 1.0.16). |
| **Backend** | Fully integrated lists and forms for managing all entities, with custom columns, filters, and permissions. |
| **Advanced permissions** | Separate permissions for addresses (access, add, edit, delete, access_all). |
| **Multi‑language support** | Translation of names for countries, states, directorates, and nationalities. |
| **Activity logging** | Integration with `Nano.ActivityLog` to track changes on geographic entities. |

---

## Database Structure

### Tables created by the plugin

| Table | Description |
| :--- | :--- |
| `nano_location_directorates` | Directorates (linked to `country_id` and `state_id`) |
| `nano_location_address` | Addresses (with `addressable_type`, `addressable_id` columns) |
| `nano_location_traceroutes` | Traceroutes |
| `nano_location_nationalities` | Nationalities |

### Tables extended from other plugins

| Original table | Added columns |
| :--- | :--- |
| `rainlab_location_countries` | `wikiDataId`, `latitude`, `longitude`, `sort_order` |
| `rainlab_location_states` | `wikiDataId`, `latitude`, `longitude`, `sort_order` |

### Simplified relationship diagram

```
countries (RainLab)
    ├── states (hasMany)
    │       └── directorates (hasMany)
    └── nationalities (hasMany?)

address (Nano)
    ├── addressable (morphTo)
    ├── country (belongsTo)
    ├── state (belongsTo)
    └── directorate (belongsTo)

traceroute
    ├── user (morphTo)
    └── geolocation data
```

---

## Models

### 1. `Country` (extended from `RainLab\Location\Models\Country`)

The following properties and relationships have been added:

```php
// Fillable fields
$fillable = ['wikiDataId', 'latitude', 'longitude', 'sort_order'];

// Media relationships (from version 1.0.16)
public $attachOne = ['image', 'book_intro'];
public $attachMany = ['images', 'videos', 'audios', 'files'];

// Before‑save event to automatically set sort_order
public function beforeSave()
{
    if ($this->id && !$this->sort_order) {
        $this->sort_order = $this->id;
    }
}
```

### 2. `State` (extended from `RainLab\Location\Models\State`)

```php
// Additional fields
$fillable = ['wikiDataId', 'latitude', 'longitude', 'sort_order'];

// Relationship to directorates
public $hasMany = ['directorates' => Directorate::class];

// Media relationships
public $attachOne = ['image', 'book_intro'];
public $attachMany = ['images', 'videos', 'audios', 'files'];

// beforeSave event for sort_order
```

### 3. `Directorate`

Full directorate model:

```php
class Directorate extends Model
{
    use \October\Rain\Database\Traits\Validation;
    use \Nano\Location\Traits\HasTimestamps;
    use \October\Rain\Database\Traits\Sortable;
    use \RainLab\Translate\Behaviors\TranslatableModel;
    
    public $table = 'nano_location_directorates';
    public $translatable = ['name'];
    public $fillable = ['name', 'code', 'is_enabled', 'sort_order'];
    
    public $belongsTo = [
        'country' => [Country::class],
        'state'   => [State::class],
    ];
    
    // Helper methods
    public static function getNameList($countryId);
    public static function formSelect($name, $countryId = null, $selectedValue = null);
    public static function getDefault();
}
```

### 4. `Addres` (note: model name is "Addres", not "Address", for legacy compatibility)

```php
class Addres extends Model
{
    use \October\Rain\Database\Traits\Validation;
    use \Nano\Location\Traits\HasTimestamps;
    use \October\Rain\Database\Traits\Sortable;
    use \RainLab\Translate\Behaviors\TranslatableModel;
    
    public $table = 'nano_location_address';
    public $translatable = ['address_1', 'address_2', 'street_name', 'city'];
    
    // Polymorphic relationships
    public $morphTo = ['addressable'];
    
    public $belongsTo = [
        'country'    => [Country::class],
        'state'      => [State::class],
        'directorate'=> [Directorate::class],
    ];
    
    // JSON fields
    protected $jsonable = ['config_data', 'other_data'];
    
    // Helper methods
    public static function getDefaultAddress($addressable);
    public function scopeIsDefault($query);
}
```

### 5. `Nationality`

```php
class Nationality extends Model
{
    use \October\Rain\Database\Traits\Validation;
    use \Nano\Location\Traits\HasTimestamps;
    use \October\Rain\Database\Traits\Sortable;
    
    public $table = 'nano_location_nationalities';
    public $fillable = [
        'name', 'code', 'nationality_code', 'country_code', 'iso_code',
        'phone_code', 'region', 'subregion', 'flag_emoji', 'description',
        'is_active', 'is_default', 'sort_order', 'metadata'
    ];
    
    public $attachOne = ['image' => \System\Models\File::class];
}
```

### 6. `Traceroute`

```php
class Traceroute extends Model
{
    use \October\Rain\Database\Traits\Validation;
    
    public $table = 'nano_location_traceroutes';
    public $fillable = [
        'ip', 'country_code', 'country_name', 'region_code', 'region_name',
        'city', 'zip_code', 'latitude', 'longitude', 'timezone', 'user_agent',
        'user_id', 'user_type', 'geo_point'
    ];
    
    // Polymorphic relationship to user
    public $morphTo = ['user'];
}
```

---

## Behaviors

### `Nano\Location\Behaviors\LocationModel`

This behavior adds location fields (country_id, state_id, directorate_id) to any model that implements it. It is used in `ProductModel`, `UserModel`, `Delivery`, and others.

```php
// Add the behavior to a model
class MyModel extends Model
{
    public $implement = ['Nano\Location\Behaviors\LocationModel'];
}
```

### `Nano\Location\Behaviors\Address`

Adds address relationships (address_1, address_2, latitude, longitude, etc.) and provides functions for handling addresses.

```php
// Usage
$model->extendClassWith('Nano\Location\Behaviors\Address');
```

---

## Traits

### `Nano\Location\Traits\HasTimestamps`

Adds `created_at`, `updated_at`, `created_by`, `updated_by`, `deleted_by` fields with automatic handling when saving the model.

```php
use Nano\Location\Traits\HasTimestamps;

class MyModel extends Model
{
    use HasTimestamps;
}
```

---

## Backend Interface

### Directorates List

- Path: `/backend/nano/location/directorates`
- Columns: `id`, `name`, `code`, `country`, `state`, `is_enabled`, `sort_order`
- Filters: `country`, `state`, `is_enabled`
- Bulk actions: enable/disable, delete

### Addresses List

- Path: `/backend/nano/location/address`
- Visible to users with permission `nano.location.address.access` (their own addresses) or `nano.location.address.access_all` (all addresses).
- Columns: `address_1`, `country`, `state`, `directorate`, `user`, `addressable`, `is_default`, `created_at`

### Nationalities List

- Path: `/backend/nano/location/nationalitys`
- Supports automatic import of default nationalities via a button in the interface.

### Plugin Settings

- Path: `Settings > Location & Geography > Location settings` (if enabled in `registerSettings` – note: most settings are currently commented out).

---

## Plugin Settings

The plugin can be customised via the configuration file `config/nano/location/config.php` after publishing:

```php
return [
    'address' => [
        'is_allow_default_country_in_add' => true,
        'is_allow_default_state_in_add' => true,
        'is_allow_default_directorate_in_add' => true,
    ],
    'models' => [
        'use_soft_delete' => true,
    ],
];
```

**Available settings in the code (some are not yet active):**

| Key | Default | Description |
| :--- | :--- | :--- |
| `address.is_allow_default_country_in_add` | `true` | Use the default country when creating a new address if the user has not selected one. |
| `address.is_allow_default_state_in_add` | `true` | Use the default state when creating a new address. |
| `address.is_allow_default_directorate_in_add` | `true` | Use the default directorate when creating a new address. |

---

## Permissions

| Permission | Tab | Description |
| :--- | :--- | :--- |
| `nano.location.address.access` | Addresses | Access only the user’s own addresses. |
| `nano.location.address.access_all` | Addresses | Access all addresses (full management). |
| `nano.location.address.add` | Addresses | Add a new address. |
| `nano.location.address.edit` | Addresses | Edit an existing address. |
| `nano.location.address.delete` | Addresses | Delete an address. |

**Note:** Permissions for nationalities and directorates are currently managed via `rainlab.location.access_settings`.

---

## Components

### `countrySelect`

A component for displaying a country dropdown in the frontend.

**Usage in a CMS page:**

```twig
[countrySelect]
inputName = "country"
inputId = "country_id"
inputClass = "form-control"
defaultValueSessionKey = "user_country"
selectedValue = "YE"
==
{% component 'countrySelect' %}
```

**Properties:**

| Property | Description |
| :--- | :--- |
| `inputName` | The field name in the POST data |
| `inputId` | The HTML element ID |
| `inputClass` | CSS class |
| `inputRequired` | Whether the field is required |
| `defaultValueSessionKey` | Session key to retrieve the default value |
| `selectedValue` | Pre‑selected value (country code) |
| `defaultLanguage` | Default display language |

---

## Helper Functions

The plugin provides several helper functions in the `helpers/` folder:

### `country_helper.php`

```php
// Get country ID from its code (YE => 1)
Nano\Location\Classes\Country::getIdByCountryCode($code);

// Get country code from its ID
Nano\Location\Classes\Country::getCountryCodeById($id);
```

### `nationality_helper.php`

```php
// Get list of nationalities
NationalityHelper::getList();

// Get a nationality automatically based on the country
NationalityHelper::getDefaultForCountry($countryCode);
```

### `religions_helper.php`

```php
// List of known religions
ReligionsHelper::getList(); // ['muslim', 'christian', 'jewish', ...]
```

### `languages_helper.php`

```php
// List of known languages with their codes
LanguagesHelper::getList(); // ['ar' => 'Arabic', 'en' => 'English', ...]
```

### `locale_helper.php`

```php
// Helper functions for locale handling
LocaleHelper::getCurrentLocale();
LocaleHelper::getDirection(); // 'rtl' or 'ltr'
```

---

## Events

The plugin fires the following events, which you can listen to:

| Event name | Parameters | Description |
| :--- | :--- | :--- |
| `nano.location.beforeSaveAddress` | `$address` | Before saving an address |
| `nano.location.afterSaveAddress` | `$address` | After saving an address |
| `nano.location.beforeDeleteAddress` | `$address` | Before deleting an address |
| `nano.location.afterGetDefaultAddress` | `$address, $addressable` | After retrieving the default address |

**Example of listening:**

```php
Event::listen('nano.location.beforeSaveAddress', function($address) {
    if (empty($address->address_1)) {
        throw new \Exception('Address line 1 is required');
    }
});
```

---

## Internationalisation

The plugin fully supports translation of the backend and the models `Country`, `State`, `Directorate`, `Addres`, `Nationality` via `RainLab.Translate`.

**Translation files:**

| File | Content |
| :--- | :--- |
| `lang/ar/lang.php` | Arabic |
| `lang/en/lang.php` | English |

**Main translation keys:**

```php
// In lang.php
'plugin' => [
    'name' => 'Location Directorates',
    'description' => 'Location based features...',
],
'menus' => [...],
'permissions' => [...],
'public' => [
    'country_label' => 'Country',
    'state_label' => 'City/State',
    'directorate_label' => 'District/Directorate',
    // ...
],
'models' => [ // added in version 1.0.16
    'media_tab' => 'Media',
    'image' => 'Main Image',
    // ...
],
```

---

## Practical Examples

### 1. Add an address relationship to a custom model

```php
use Nano\Location\Behaviors\Address;

class MyPost extends Model
{
    public $implement = [Address::class];
    
    // This will add fields: country_id, state_id, directorate_id, address_1, address_2, ...
}
```

### 2. Retrieve the default address of a user

```php
$user = User::find(1);
$defaultAddress = Addres::getDefaultAddress($user);
if ($defaultAddress) {
    echo $defaultAddress->address_1 . ', ' . $defaultAddress->directorate->name;
}
```

### 3. Create a new address using the API (via `Nano.LocationApi`)

```http
POST /api/v2/addresses
{
    "addressable_type": "RainLab\\User\\Models\\User",
    "addressable_id": 1,
    "country_id": 1,
    "state_id": 5,
    "directorate_id": 12,
    "address_1": "Taiz Street",
    "is_default": true
}
```

### 4. Add a main image to a country in code

```php
$country = Country::find(1);
$country->image = Input::file('flag');
$country->save();
```

### 5. Get a list of directorates for a specific country (for use in the frontend)

```php
$directorates = Directorate::whereHas('state.country', function($q) {
    $q->where('code', 'YE');
})->get();
```

---

## Troubleshooting

### Problem: Address fields do not appear in the user model

**Cause:** The `Address` or `LocationModel` behavior has not been applied to the model.

**Solution:** Ensure that `extendBackendUserModel()` and `extendFrontendUserModel()` are correctly called in `Plugin.php`.

### Problem: Error `Class 'Nano\Location\Models\Directorate' not found`

**Cause:** The plugin migrations have not been run.

**Solution:**
```bash
php artisan nano:up
```

### Problem: Image does not appear in the countries list

**Cause:** `image` has not been set on the model, or the `simpleimage` column type is not supported.

**Solution:** Ensure an image is uploaded and saved, and check that the `system/models/file/columns.yaml` file contains a definition for `simpleimage`.

### Problem: Conflict with the original `RainLab.Location` plugin

**Cause:** Some extensions may not work if `RainLab.Location` is loaded after `Nano.Location`.

**Solution:** Declare the dependency in `$require = ['RainLab.Location', ...]` and check the loading order.

### Problem: Error in the `getNameList` function of directorates

**Cause:** Passing `$countryId` instead of `$stateId` sometimes.

**Solution:** Use `Directorate::getNameList($stateId)`, not `$countryId`.

---

## References

- [GitHub - Nano.Location](https://github.com/nano/location-plugin)
- [RainLab.Location Documentation](https://github.com/rainlab/location-plugin)
- [October CMS Models Documentation](https://docs.octobercms.com/3.x/database/model.html)
- [DynamicAddMediaIncludes Behavior Documentation](./Docs-DynamicAddMediaIncludes-en.md)
- [Nano.LocationApi Plugin Documentation](./Docs-Nano-LocationApi-en.md)

---

## Summary

The `Nano.Location` plugin is the cornerstone for managing geographic data in NanoSoft projects. Thanks to its flexible structure, its extension of `RainLab.Location`, and its support for polymorphic addresses, nationalities, directorates, traceroutes, and multimedia, developers can build location‑rich applications quickly and efficiently. The plugin is integrated with `Nano.LocationApi` to provide a RESTful API, and with `Nano.ActivityLog` for activity tracking.

**Latest updates (1.0.16):**
- Multimedia support on countries, states, and directorates.
- Added backend columns and fields for media.
- Translation support for the new fields.

We recommend upgrading to the latest version to take advantage of these features.

---

**Documentation date:** 2025-04-30  
**Author:** NanoSoft Development Team  
**License:** Copyright reserved to NanoSoft Company

---
