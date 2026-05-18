## 2026-05-13 - 2026-05-14

**Updates for `Nano.UserPlus` Plugin – version 1.1.7**

### Summary of Updates

Following support for multiple account types in previous versions, version 1.1.7 arrives to complete building a flexible and integrated user management experience. This release focuses on three main axes:

- **Improving the account type definition mechanism** and making it fully controllable from the config file.
- **Full compatibility** with modern versions of RainLab.User (which use `first_name` and `last_name` instead of `name` and `surname`).
- **Advanced management of user groups** by adding new fields (`is_new_user_default`, `is_active`, `sort_order`) to the `user_groups` table, and extending the backend interface to manage them seamlessly.

These improvements give developers and system administrators finer control over the user experience, unify default group behavior, and ensure the plugin's compatibility with the latest platform versions.

---

### Version 1.1.7 – Flexible Account Types, Improved Compatibility, and Advanced User Group Management

#### Release Goals

- **Make account types (`ref_type`) customizable** from the config file without touching the code.
- **Ensure compatibility** with the name field changes in RainLab.User 2.x via a custom event handler.
- **Extend the `user_groups` table** with columns for activation control, ordering, and default assignment.
- **Integrate the new fields** into the user group management forms and lists in the backend panel.
- **Support multilingual** for all new extensions with safe error handling.

#### New Features

##### 1. Customizable Account Types via Settings

The `getRefTypeList` function in the `Nano\UserPlus\Models\Preference` model has been restructured to be fully dynamic, relying on configuration keys like:
```php
Config::get('nano.userplus::ref_type.department', true)
```
Thus, any account type (such as `delivery`, `partner`, `student`, `employee`...) can be enabled or disabled through the `config.php` file without modifying the source code. The helper functions `getRefTypeOptions` in the user model were also updated to feed dropdowns with the same dynamic options.

##### 2. Compatibility with RainLab.User 2.x

Due to the change of `name` and `surname` fields to `first_name` and `last_name` in recent versions of RainLab.User, an event class `Nano\UserPlus\EventHandlers\RainlabUser2CompatibilityHandler` was added that creates a virtual link between the old and new fields via dynamic methods (`addDynamicMethod`). This ensures any code depending on `name` and `surname` continues to work without errors, while preserving the original data in `first_name` and `last_name`.

This handler is automatically activated when the `first_name` column exists in the `users` table.

##### 3. New Columns in the `user_groups` Table

The migration `builder_table_add_is_active_columns_to_frontend_user_groups_table.php` was added which introduces the following columns to the `user_groups` table:

| Column | Type | Description |
| :--- | :--- | :--- |
| `is_new_user_default` | boolean (default: false) | Determine whether this group is the default for new users. |
| `is_active` | boolean (default: true) | Enable or disable the group (to hide it from lists). |
| `sort_order` | integer (nullable) | Group display order in dropdown lists and interfaces. |

##### 4. Extension of `RainLab\User\Models\UserGroup` Model

Through the `extendUserGroupModel` function in `Plugin.php`:

- Fields were added to `$fillable`.
- Appropriate validation rules (`boolean`, `integer`) were set.
- Smart default values were set when creating a new group.
- Helper functions like `getIsActiveOptions` and `getIsNewUserDefaultOptions` were added to return translated options.

All of this with `try-catch` exception handling and error logging without disrupting the system.

##### 5. Improved User Groups Management Interface

The `RainLab\User\Controllers\UserGroups` controller was extended in two complementary ways:

- **List Columns**: the columns `is_new_user_default` and `is_active` (as a Switch) and `sort_order` were added to the groups list, with sorting and show/hide abilities.
- **Form Fields**: the three fields were injected into the group creation/edit form, with helper comments, distributed over `left`/`right` sections for space organization.

All extensions were done considering context checks (e.g., the form is not nested `isNested`) to ensure no interference with other forms.

##### 6. Full Translation Support

New translation keys were added in the `lang.php` file under the `user_groups` section, including:

- Field names.
- Helper comments.
- Value options (`is_default_yes`, `is_active_yes`...).

The `trans()` function was used in all locations to ensure text appears in the appropriate language.

---

### Practical Examples

#### 1. Customizing Account Types via Settings

In the `config.php` file, you can disable the "Delivery" type, for example:

```php
'ref_type' => [
    'delivery' => false,
    'department' => true,
    // ... remaining types
]
```

Once the file is saved, the "Delivery Worker" options will disappear from `ref_type` dropdowns in user forms.

#### 2. Setting a Default Group for New Users

After the update, when entering the group management, you can select a group and make it the "default group for new users". This is coupled with an event that listens for a new user registration to automatically link them to this group (if the feature is enabled).

#### 3. Sorting Groups in the Frontend

Using the `sort_order` field, groups can be ordered in dropdown lists, improving the user experience during registration or in the backend panel.

---

### Version Summary (1.1.6 – 1.1.7)

| Version | Key Features |
| :--- | :--- |
| 1.1.6 | Support additional user fields (`phone`, `mobile`, `company`...) and extended filters. |
| 1.1.7 | Dynamic account types, `RainlabUser2CompatibilityHandler` compatibility handler, new `user_groups` columns, and extended group management interface. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace the `Plugin.php` file with the new version containing `extendUserGroupModel` and `extendUserGroupsController` functions.
   - Add the file `eventhandlers/RainlabUser2CompatibilityHandler.php`.
   - Update the `models/Preference.php` file if the type-related changes are included.
   - Update the language file `lang.php` to add `user_groups` keys.

2. **Execute migration**:
   - Make sure to run the migration `builder_table_add_is_active_columns_to_frontend_user_groups_table.php` that adds the new columns to the `user_groups` table. This can be executed via `php artisan october:up` or manually.

3. **Update settings** (optional):
   - Review the `config.php` file to customize the desired account types (e.g., `ref_type.delivery` = true/false).

4. **Clear cache**:
   - After updating, run `php artisan cache:clear` to ensure new translation and configuration files are loaded.

5. **Test functionalities**:
   - Verify that the new columns appear in the user groups list.
   - Try creating a new group and enabling the "default group" option, then register a new user to check automatic linkage.
   - Verify that account types (`ref_type`) appear according to `config.php` settings.

---

### Conclusion

With version 1.1.7, we have taken a significant step towards making the `Nano.UserPlus` plugin more flexible and integrated with the platform. Account type management has become fully dynamic, the compatibility challenge with modern versions has been resolved, and user group management has been enhanced with advanced control tools.

These improvements open the door to more customized uses, such as:

- Automatically directing new users to specific groups.
- Hiding inactive groups from interfaces.
- Ordering groups by priority in registration forms.

We look forward to your feedback and suggestions to continue developing this vital plugin.

---

**Reference Documents**:
- [UserPlus Plugin Documentation](./docs/UserPlus/Docs-Nano-UserPlus-en.md)
- [Preference Model Documentation](./docs/UserPlus/Docs-Preference-Model-en.md)
- [UserModelBehavior Documentation](./docs/UserPlus/Docs-UserModelBehavior-en.md)
