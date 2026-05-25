## 2026-05-25 – 2026-05-26

**Update to the `Nano.AuthApi` Plugin – Version 1.0.21**

### Adding Advanced Features to `AccessManager`: Dynamic Resolver for Students and Parents, Advanced Filters, Centralised Configuration Loading, Simplified Methods, and Fallback Support.

---

### Summary of Updates

Version **1.0.21** of the `Nano.AuthApi` plugin introduces several substantial improvements to the `AccessManager` class, which is now responsible not only for permission management but also for handling advanced scenarios such as:

- **Dynamic resolver for students and parents (`resolveDynamicFrontendOptions`)** – automatically populates `student_id` and `record_id` based on user type and flexible rules from `config.php`.
- **Advanced filters control (`advanced_filters`)** – allows specifying which fields can use `is_or`, `is_not`, `is_or_null`, with rules per user type and `ref_type`.
- **Centralised permission configuration loading** – methods `getOperationByConfig`, `checkByResource` read settings directly from `config.php` without manual array building.
- **Fallback support** – methods `getOperationConfigWithFallback`, `checkWithFallback` allow operations like `show` to reuse `list` settings.
- **Simplified access methods** – `getOperationBySimpleKey`, `getAdvancedFiltersConfig` ease daily usage.

These improvements make `AccessManager` a comprehensive tool for permission management, filtering, and handling student/parent requirements in any API.

---

### Release Objectives

- **Enable school applications** to easily handle student and parent scenarios via `frontend_resolver`.
- **Provide fine-grained filter control** to ensure data security and prevent unauthorised users from passing dangerous filters.
- **Centralise permission settings** in `config.php` to reduce code duplication and simplify maintenance.
- **Simplify `AccessManager` usage** via shortcut methods (e.g., `checkByResource` instead of passing three parameters).
- **Support automatic fallback** for operations like `show` without writing complete settings.

---

### New Features and Improvements

#### 1. Dynamic Resolver for Students and Parents (`resolveDynamicFrontendOptions`)

A set of methods has been added to handle student and parent scenarios:

- **`resolveDynamicFrontendOptions($user, $options, $query, $resourceKey)`** – main method that analyses the user’s `ref_type` and applies rules defined in `frontend_resolver` from `config.php`.
- **`getFrontendResolverConfig($resourceKey)`** – loads resolver settings from the configuration file.
- **`ensureOptionsKeys($options, $keys)`** – ensures certain keys exist in the options array to avoid `undefined index` errors.

**Example `frontend_resolver` configuration in `config.php`:**

```php
'frontend_resolver' => [
    'enabled' => true,
    'apply_for_backend' => false,
    'apply_for_frontend' => true,
    'ref_type_rules' => [
        'student' => [
            'fields' => [
                'student_id' => ['source' => 'user', 'auto_fill' => true, 'force' => true],
                'record_id'  => ['source' => 'current_record', 'auto_fill_if_missing' => true, 'force' => true],
            ],
        ],
        'parent' => [
            'fields' => [
                'student_id' => ['source' => 'request', 'required' => true, 'validate_ownership' => true, 'force' => true],
                'record_id'  => ['source' => 'request_or_current', 'fallback_field' => 'student_id', 'force' => true],
            ],
        ],
    ],
],
```

When `resolveDynamicFrontendOptions` is called, `AccessManager` will automatically populate `student_id` and `record_id` according to these rules.

#### 2. Advanced Filters Control (`advanced_filters`)

Methods have been added to check whether a user is allowed to use `is_or_*`, `is_not_*`, `is_or_null_*` keys:

- **`isAdvancedFilterAllowed($user, $field, $operationConfig)`** – checks allowance based on `advanced_filters` in the operation configuration.
- **`filterAdvancedOptions($user, $options, $operationConfig)`** – removes any disallowed keys from `$options`.
- **`applyAdvancedFilterConstraints($user, $options, $operationConfig)`** – comprehensive function (alias for `filterAdvancedOptions`).

**Example `advanced_filters` configuration:**

```php
'advanced_filters' => [
    'enabled' => true,
    'default_for_backend' => true,
    'default_for_frontend' => false,
    'field_specific_rules' => [
        'student_id' => [
            'backend' => true,
            'frontend' => ['*' => false, 'parent' => true],
        ],
    ],
],
```

#### 3. Centralised Permission Configuration Loading from `config.php`

Methods have been added to read and normalise settings:

- **`getDefaultOperationConfig()`** – returns standard default settings.
- **`normalizeOperationConfig($config)`** – merges provided configuration with defaults.
- **`arrayMergeRecursiveDistinct($array1, $array2)`** – recursively merges arrays, replacing values instead of concatenating.
- **`getOperationConfig($plugin, $resource, $operation, $overrideConfig)`** – fetches settings from `config` using three parameters.
- **`checkWithConfig($operation, $plugin, $resource, $user, $overrideConfig)`** – combines fetching and checking.

#### 4. Simplified Methods Using a Single Key

To ease usage, methods that accept a single key instead of three have been added:

- **`getOperationByConfig($resourceKey, $overrideConfig)`** – key like `'nano.absenceapi::absences.list'`.
- **`checkByResource($resourceKey, $user, $overrideConfig)`** – automatically extracts the operation from the last part of the key.
- **`getAdvancedFiltersConfig($resourceKey, $overrideConfig)`** – returns only the `advanced_filters` array for the resource.
- **`getOperationBySimpleKey($simpleKey, $overrideConfig)`** – shortens writing `'nano.absenceapi::'` (e.g., `'absences.list'`).

#### 5. Fallback Support Between Operations

These methods help in scenarios such as `show` being able to use `list` settings if `show` settings are incomplete:

- **`getOperationConfigWithFallback($resourceKey, $fallbackResourceKey, $overrideConfig)`**
- **`checkWithFallback($resourceKey, $fallbackResourceKey, $user, $overrideConfig)`**

If `is_allow` exists in the primary settings, it is used directly. Otherwise, the fallback settings are merged with the primary ones.

#### 6. Additional Helper Methods for Student/Parent Scenarios

- **`applyFrontendStudentParentFilters($query, $user, $options, $studentIdColumn, $recordIdColumn, $tablePrefix, $resolveIfNeeded)`** – applies the resolved filters directly to the query.
- **`getStudentIdForFrontendUser($user, $options)`**, **`getRecordIdForFrontendUser($user, $options)`** – shortcut methods to fetch resolved values.
- **`validateParentStudentRelation($user, $studentId, $recordId)`** – checks that the student belongs to the parent.

---

### Practical Examples of the New Usage

#### 1. Checking `list` Permission Using `checkByResource`

```php
$access = AccessManager::checkByResource('nano.absenceapi::absences.list', $user);
if (!$access['allowed']) {
    return $this->errorUnauthorized($access['message']);
}
```

#### 2. Using `checkWithFallback` for the `show` Operation

```php
$access = AccessManager::checkWithFallback(
    'nano.absenceapi::absences.show',
    'nano.absenceapi::absences.list',
    $user
);
```

#### 3. Applying Advanced Filters and Removing Disallowed Ones

```php
$advConfig = AccessManager::getAdvancedFiltersConfig('nano.absenceapi::absences.list');
$options = AccessManager::instance()->filterAdvancedOptions($user, $options, ['advanced_filters' => $advConfig]);
```

#### 4. Applying the Dynamic Resolver to Resolve `student_id` and `record_id`

```php
$options = AccessManager::instance()->resolveDynamicFrontendOptions(
    $user, $options, null, 'nano.absenceapi::absences.list'
);
// Then in getRecords(), apply the resolved filters
if (!empty($options['student_id']) && $options['is_force_student_id']) {
    $query->where('student_id', $options['student_id']);
}
```

#### 5. Applying `applyAccessScope` with the Ability to Disable Scopes

```php
$query = AccessManager::instance()->applyAccessScope($query, $access, [
    'company_field'    => 'companys_id',
    'department_field' => 'departments_id',
    // 'state_field' omitted -> no state filter
    'created_by_field' => 'created_by',
]);
```

---

### Backward Compatibility

- **All old methods** (e.g., `check`, `applyAccessScope`, `canAccessAllCompanies`) still work as before.
- **No functionality has been removed** – only additions have been made.
- **No database migrations are required.**
- **You can upgrade directly** without modifying existing code.

---

### Upgrade Requirements (from 1.0.20 to 1.0.21)

1. **Update the code**:
   - Replace the `AccessManager.php` file with the new version (attached).
   - No other changes in the plugin are required.

2. **Update the `version.yaml` file**:
   - Add version `1.0.21` as shown above.

3. **Update `config.php` files of API plugins (optional)**:
   - You can add `advanced_filters` and `frontend_resolver` sections to the resource settings to benefit from the new features.

4. **Run `php artisan plugin:refresh Nano.AuthApi` (optional).**

5. **Test the new features**:
   - Try `checkByResource` instead of the traditional `check`.
   - Try `checkWithFallback` for a `show` endpoint.
   - Try `filterAdvancedOptions` to clean request options.
   - Try `resolveDynamicFrontendOptions` in a controller that uses students and parents.

---

### Benefits and Added Value

- **Reduces duplicate code** in API controllers by up to 70%.
- **Centralises control** over permissions and filters, making behaviour changes easy via configuration files only.
- **Higher security** through fine‑grained control over advanced filters, preventing unauthorised users from using them.
- **Native support for school scenarios** (students and parents) in a natural and straightforward way.
- **Ease of maintenance** because any change to permission logic is made in one place.

---

### Conclusion

Version **1.0.21** of `Nano.AuthApi` establishes `AccessManager` as a comprehensive and advanced tool for managing permissions, filters, and relationships between users and entities (such as students and parents). Thanks to the new methods, building a secure and flexible API has become much easier and aligns perfectly with the `Nano-Api-SKILL.md` guide.

---

**Reference documentation**:
- [`AccessManager` class documentation](./docs/AuthApi/Docs-AccessManager-en.md)
- [`AccessManager-Permission` documentation](./docs/AuthApi/Docs-AccessManager-Permission-en.md)
- [General examples `AccessManager-Example`](./docs/AuthApi/Docs-AccessManager-Example-en.md)
- [Comprehensive practical example `AccessManager-Example-Api`](./docs/AuthApi/Docs-AccessManager-Example-Api-en.md)
- [`AuthHelpers` class documentation](./docs/AuthApi/Docs-AuthHelpers-en.md)
- [API Plugin Development Guide (Nano-Api-SKILL)](./docs/mcp/Nano-Api-SKILL.md)
- [Update to the `Nano.AbsenceApi` plugin (practical example)](./docs/AbsenceApi/Update-AbsenceApi-v1.1.0-en.md)
