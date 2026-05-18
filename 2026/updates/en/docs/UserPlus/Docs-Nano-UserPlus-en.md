# Nano.UserPlus Plugin Documentation

## Overview

The `Nano.UserPlus` plugin is a comprehensive plugin for the OctoberCMS platform that radically extends the user management functions provided by the `RainLab.User` plugin. The plugin aims to transform the core user system into an integrated system supporting:

- **Rich profiles** containing dozens of additional fields (location, contact, work, social links).
- **Multiple and flexible account types** (regular user, delivery worker, employee, student, supplier...) that can be linked to other objects via polymorphic relationships.
- **Advanced integration with the notification system** `RainLab.Notify` through custom conditions and events.
- **Advanced user import and export** tools with separate permissions.
- **Advanced filters** in the backend panel to facilitate user search.
- **User preferences management** (frontend) using a system similar to backend preferences.
- **Full compatibility** with modern versions of RainLab.User (supporting `first_name`/`last_name`).
- **Advanced control of user groups** (enable/disable, ordering, default group).

## Requirements

- `RainLab.User`
- `Nano.Location`
- `RainLab.Notify`

## Main Components

| Component | Description |
| :--- | :--- |
| `Plugin.php` | The main file, contains `boot` and `register` functions that perform injection and extension operations. |
| `UserModelBehavior` | A behavior added to the `UserModel` to provide additional relationships, query scopes, and helper functions. |
| `Preference` | A model that mimics `Backend\Models\Preference` for managing frontend user preferences. |
| `UserPreference` | The actual model that stores preferences in the `frontend_user_preferences` table. |
| `UserPreferencesModel` | A behavior (Behavior) added to `Preference` to link it to the current user record. |
| `RainlabUser2CompatibilityHandler` | An event handler that ensures compatibility between old fields (`name`, `surname`) and new ones (`first_name`, `last_name`). |
| `UserLocationAttributeCondition` | A custom notification condition based on the user's location. |
| `Components\Notifications` | A component for displaying user notifications in the frontend. |

## Extended Fields in the `users` Table

The plugin adds the following fields to the user model via `addFillable`:

| Field | Type | Description |
| :--- | :--- | :--- |
| `phone` | string | Phone number |
| `mobile` | string | Mobile number (required) |
| `company` | string | Company name |
| `street_addr`, `city`, `zip` | string | Street address, city, zip code |
| `country`, `country_id` | relation | Country (relationship with `RainLab\Location\Models\Country`) |
| `state`, `state_id` | relation | State/City |
| `directorate`, `directorate_id` | string/int | Directorate |
| `address_1`, `address_2`, `street_name`, `street_number` | string | Address details |
| `latitude`, `longitude` | float | Geographic coordinates |
| `radius`, `geo_components`, `postcode`, `country_long`, `formataddress`, `vicinity` | mixed | Additional location data |
| `companys_id`, `departments_id`, `employees_id` | int | Company/Department/Employee IDs |
| `ref_type`, `ref_id` | string/int | Account type and its identifier (for polymorphic relationships) |
| `referral_id` | int | Referral ID |
| `barcode`, `manual_code`, `accounts_id` | string/int | Identification codes |
| `gender` | string | Gender (male/female) |
| `date_of_birth` | datetime | Date of birth |
| `created_by`, `updated_by` | int | Creator/Modifier ID |
| `is_blocked_orders` | boolean | Block orders |
| `business_number`, `vat_number` | string | Commercial and tax number |
| `language` | string | User language |
| `nationality` | int | Nationality ID |
| `bio` | text | Bio |
| `links` | json | Social links (JSON) |
| `website` | json | Websites (JSON) |

The following relationships are also added:
- `belongsTo['Nationalitys']`: with model `Nano\Location\Models\Nationality`.
- `morphMany['notifications']`: with `RainLab\Notify\Models\Notification`.
- `morphTo['ref']`: a polymorphic relationship based on `ref_type`.
- `hasMany['preferences']`, `user_preferences`, `user_preferences_location`: with `UserPreference` (added via `UserModelBehavior`).

## Account Types (`ref_type`)

The plugin provides a flexible account type system that allows linking a user to different objects on the platform. The available types can be controlled via the `config.php` file:

```php
'ref_type' => [
    'delivery' => true,
    'department' => true,
    'employee' => false,
    'student' => true,
    // ...
]
```

Supported types (depending on installed plugins):
- `user` (default)
- `delivery` (Nano.Deliverys)
- `department` (Tss.Basic)
- `employee` (Tss.Basic)
- `customer` (Tss.SalesAndMarketing)
- `suppler` (Tss.Purchasing)
- `syndical` (Tss.Purchasing)
- `partner` (Tss.Accounts)
- `student` (Tss.Student)
- `mparent` (Tss.Student)
- `author` (Nano.Authors)

Each type uses `getMorphClass()` to obtain the correct polymorphic relationship name.

## User Groups (`user_groups`)

Starting from version 1.1.7, the `user_groups` table has been extended with the following fields:

| Field | Description |
| :--- | :--- |
| `is_new_user_default` | Set the group as the default for new users |
| `is_active` | Enable/Disable the group |
| `sort_order` | Group order in lists |

These fields are automatically injected into the model and list of `RainLab\User\Controllers\UserGroups`.

## RainLab.User 2.x Compatibility

Through the class `RainlabUser2CompatibilityHandler`, the old fields (`name`, `surname`) are linked with the new ones (`first_name`, `last_name`), ensuring that any legacy code does not break.

## Permissions

| Permission | Description |
| :--- | :--- |
| `nano.userplus.users.import` | Import users |
| `nano.userplus.users.export` | Export users |

## Events

- `backend.list.extendColumns`: to extend the columns of user and group lists.
- `rainlab.user.register`: can be used to link the new user to the default group.

## Translation

All texts are translatable via the `lang.php` file in the `lang` folder. The default language is supported in Arabic.

## Important Notes

- The `unique` rule on the `mobile` field is dynamically set to exclude the current record during update.
- Fields `password`, `email`, `mobile`, `ref_type`, `ref_id` can be made read-only when editing an existing user according to `config.php` settings.
- Hidden fields (like `last_login`, `ref_type`) in user lists can be shown via list settings.

