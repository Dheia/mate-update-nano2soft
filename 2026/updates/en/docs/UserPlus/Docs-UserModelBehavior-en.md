# UserModelBehavior Documentation

## Overview

`Nano\UserPlus\Behaviors\UserModelBehavior` is a behavior dynamically added to the model `RainLab\User\Models\User` via the `Nano.UserPlus` plugin. This behavior aims to enrich the user model with additional relationships to the preferences system, useful query scopes, and helper functions for dropdown list options.

This behavior is not loaded by default in `RainLab.User`; instead, `Nano.UserPlus` extends the user model with it via `$model->extendClassWith(...)` in the `boot()` function.

## Added Relationships

In the `__construct` function, the behavior adds the following relationships to the user model:

| Relationship | Type | Related Model | Description |
| :--- | :--- | :--- | :--- |
| `preferences` | `hasMany` | `Nano\UserPlus\Models\UserPreference` | All user preferences (no scope). |
| `user_preferences` | `hasMany` | `Nano\UserPlus\Models\UserPreference` | User preferences of type `frontend::frontend.preferences` (scope `isUserPreference`). |
| `user_preferences_location` | `hasMany` | `Nano\UserPlus\Models\UserPreference` | User location preferences of type `nano.sitemanager::location.manager` (scope `isLocationPreference`). |

## Query Scopes

### `scopeWhereUserPreferenceStates($query, $state_id = null)`

Finds users who have a location preference matching a specific `state_id`.

- If `$state_id` is not passed, it tries to fetch it from `\SiteLocation::getEditState()`.
- Uses the `user_preferences_location` relationship to search in the stored JSON value (`value->editState_id`).

**Example:**
```php
$usersInState = User::whereUserPreferenceStates(5)->get();
```

### `scopeWhereUserPreferenceStatesFilter($query, $state_id = null)`

A wrapper function for `scopeWhereUserPreferenceStates`, but handles arrays (if it comes from a filter group).

- If `$state_id` is an array, takes the first value.

**Example:**
```php
// Usage in filters
User::whereUserPreferenceStatesFilter([5])->get();
```

### `scopeIsSysCompanys($query)`

A complex scope to filter users according to organizational permissions:

- Excludes email `info@nano2soft.com`.
- If the current user does not have permission to access all companies (`BasicHelper::checkAccessAllCompanys`), applies scope `IsCompany`.
- If they do not have permission to access all departments (`BasicHelper::checkAccessAllDepartments`), applies scope `IsDepartment`.

### `scopeIsSuspendedV1($query, $value = true)`

Finds users whose accounts are suspended via the `throttles` relationship (must exist).

- `$value = true`: search for suspended.
- `$value = false`: search for not suspended.

### `scopeIsBannedV1($query, $value = true)`

Similar to suspension but for banned (banned/not banned).

### `getInetisIsBannedAttributeV1()`

A custom accessor to check the ban status of the current user via `AuthManager`.

### `setInetisIsBannedAttributeV1($value)`

A mutator to change the ban status. Prevents the user from banning themselves and throws `ApplicationException` in that case.

## Helper Functions for Dropdown Options

### `getEditStateListOptions()`

Returns a list of states/cities based on current or default location settings.

**Logic:**
1. Checks for existence of `RainLab\Location\Models\State`.
2. Tries to fetch `SiteLocation::getModelState()`.
3. If not found, uses default country and city.
4. Returns `State::getNameList($country_id)` with the current city added to the top of the list if not present.

**Example:**
```php
$stateOptions = $user->getEditStateListOptions();
// ['1' => 'Sanaa', '2' => 'Aden', ...]
```

### `getPhoneTypeOptions($value, $formData)`

Returns supported phone types (mobile, home, work, fax...). If `Tss.Basic` plugin is installed, delegates to it.

### `getSortShowOptions($value, $formData)`

Returns display sorting options from 1 to 10. Delegates to `Tss.Basic` if present.

### `getWebsiteTypeOptions($value, $formData)`

Returns website types (website, facebook). Delegates to `Tss.Basic` if present.

## Important Notes

- All helper functions for dropdown options check for the existence of `Tss\Basic\Classes\RepeaterFieldsData` to give priority to platform settings.
- The scopes `IsSuspendedV1` and `IsBannedV1` assume the existence of a `throttles` relationship in the user model (may need to be added manually).
- The function `getInetisIsBannedAttributeV1` uses a custom accessor (not `getIsBannedAttribute`) to avoid conflict with any other plugin.
