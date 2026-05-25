# 📘 `StudentRecordsHelper` Class Documentation

**Version:** 1.0.13  
**Path:** `Tss\Student\Helpers\StudentRecordsHelper`  
**Purpose:** A unified helper for retrieving student records (from the `tss_school_students` table) using an advanced filtering system, caching, events, and full support for `AccessManager` and `AdvancedQueryHelper`.

---

## 1. Introduction

The `StudentRecordsHelper` class is responsible for executing all student data retrieval queries in the `Tss.Student` plugin. It follows a standard, reusable pattern and can be customised for any other model (e.g., `StudentRecord` or `Mparent`) by adding model‑specific filters.

**Key features:**

- **Unified response structure:** All methods return an array containing `code`, `status`, `message`, `data`, `error`, `errors`, `input_data`, `process_data`, `debug`.
- **Advanced dynamic filtering:** Supports `is_or`, `is_not`, `is_force`, `is_or_null` for every filterable field.
- **Intelligent text search:** With advanced search options and result weighting.
- **Built‑in caching:** With unique keys, tags, and forced refresh.
- **Events:** Can be disabled or customised, enabling extensibility without modifying the core code.
- **`AccessManager` support:** Access scopes (company, department, state, created_by, children) can be automatically applied to the query.
- **Flexible output:** Can return a `Builder`, `Collection`, `Paginator`, first record, or data transformed via a Transformer.
- **Performance optimisation:** Excludes unwanted columns (`exclude`), loads only necessary relationships.

---

## 2. Benefits and Use Cases

| Benefit | Description |
|---------|-------------|
| **Unified query logic** | Prevents repeating `where` and `join` queries across multiple controllers. |
| **Ease of maintenance** | Any change to the filtering method (e.g., adding `is_or_null` support) is made in one place. |
| **Higher security** | Through `AccessManager` integration, permission conditions are applied directly to the query. |
| **High API flexibility** | The client can specify excluded fields, included relationships, ordering, logical filtering, etc. |
| **Production‑ready** | Supports caching and events, allowing performance improvements and functionality extension without touching the original code. |
| **Easy testing** | The `is_to_sql` option prints the final query for debugging. |

**Typical use cases:**

- Fetching a list of students from the API controller (`Nano\StudentsApi\APIControllers\Students::index`).
- Creating custom reports in the Backend.
- Complex queries that need to combine several logical conditions (`OR`/`AND`).
- Exporting student data with the ability to specify only the required columns.

---

## 3. Class Methods and Properties

### 3.1 Public Static Methods

| Method | Description |
|--------|-------------|
| `getRecords(array $options, bool $isException = false): array` | The main method for retrieving student records. |
| `applyFieldFilter(...)` | Applies a single‑field filter with support for `is_or`, `is_not`, `is_force`, `is_or_null` (can be used externally). |
| `applyImagesFilters(...)` | Applies image/file filters (`is_has_any_images`, `is_has_image`, …). |
| `applySearchQuery(...)` | Applies advanced text search with weighting options. |
| `applyUserFilter(...)` | Applies user filtering (`is_user`). |
| `applyInteractionFilters(...)` | Applies interaction filters (`isFavorites`, `isLikes`, …). |
| `applyBlockedFilter(...)` | Applies blocking filter (`is_support_blocked`). |
| `applyGroupBy(...)` | Applies `group by` as an array or string. |
| `applyOrderBy(...)` | Applies sorting with support for sortable scopes. |
| `applySingleOrderBy(...)` | Applies sorting on a single field, checking for the existence of a scope. |
| `executeQuery(...)` | Executes the query according to the desired output type. |
| `transformResult(...)` | Transforms a `Paginator` into an array using `StudentTransformer`. |
| `normalizeWithArray(...)` | Converts a `custom_with` value into an array. |
| `isMethodExists(...)` | Checks whether a method exists on an object. |

### 3.2 `getRecords` Method (Details)

**Options (`$options`):**

| Group | Options |
|-------|---------|
| Basic filtering | `id`, `students_id`, `parents_id`, `gender`, `marital_status`, `passport_type`, `groups_class_id`, `type`, `country_id`, `state_id`, `directorate_id`, `year_id`, `class_id`, `group_id` |
| Statuses | `is_active`, `is_effective`, `is_published`, `has_parents` |
| Location and distance | `latitude`, `longitude`, `radius` |
| Search | `q`, `is_advanced_search`, `advanced_search_mode`, `is_search_weights`, `search_weights_direction` |
| User and interactions | `is_user`, `user`, `user_type`, `user_id`, `is_support_blocked`, `isFavorites`, `isLikes`, `isBookmarks`, `isReactions` |
| Advanced query | `exclude`, `custom_with`, `with_count`, `orderBy`, `orderDirection`, `group_by`, `group_by2`, `group_by3`, `having` |
| Output | `is_query`, `is_first`, `is_model`, `is_collection`, `is_paginator`, `is_to_sql`, `withTrashed`, `onlyTrashed` |
| Caching | `cache_enabled`, `cache_key`, `cache_ttl`, `cache_tags`, `cache_force_refresh` |
| Events | `fire_before_event`, `fire_after_event`, `event_before_name`, `event_after_name`, `event_list_name` |
| Transformer | `is_mate_data_images`, `is_show_image`, `is_show_files`, `is_mask_location`, `is_stop_user_include`, `is_stop_user_secret`, `is_stop_parent_include`, `is_stop_parent_secret` |
| `AdvancedQueryManager` control | `disable_advanced_query`, `force_advanced_query` |

**Return value (array):**

```php
[
    'code'    => 200,
    'status'  => true,
    'message' => 'Data retrieved successfully',
    'data'    => [...]   // Builder, Collection, Paginator, or transformed array
    'error'   => null,
    'errors'  => null,
    'input_data'   => [...],  // original options
    'process_data' => [...],  // options after merging
    'debug'   => [...]        // only when APP_DEBUG = true
]
```

---

## 4. Complete and Detailed Examples

### 4.1 Basic Usage – Fetch Active Students Ordered by Name

```php
$result = StudentRecordsHelper::getRecords([
    'is_active' => true,
    'orderBy' => 'full_name',
    'orderDirection' => 'asc',
    'per_page' => 20,
]);

if ($result['status']) {
    $paginator = $result['data']; // Paginator object
    foreach ($paginator as $student) {
        echo $student->full_name;
    }
} else {
    Log::error($result['error']);
}
```

### 4.2 Using Advanced Filters (is_or, is_not, is_force, is_or_null)

```php
$result = StudentRecordsHelper::getRecords([
    'gender' => 'male,female',
    'is_or_gender' => true,
    'marital_status' => 'single,married',
    'is_or_marital_status' => true,
    'is_not_marital_status' => false,
    'country_id' => 'YE',
    'is_or_null_country_id' => true,   // includes records where country_id = null
]);
```

### 4.3 Advanced Text Search with Weighted Ordering

```php
$result = StudentRecordsHelper::getRecords([
    'q' => 'Mohamed',
    'is_advanced_search' => true,
    'advanced_search_mode' => 'OR',
    'is_search_weights' => true,
    'search_weights_direction' => 'desc',
]);
```

### 4.4 Applying Access Scopes (AccessManager) via `access_result`

```php
$access = AccessManager::instance()->check('list', $config, $user);
$options = [
    'access_result' => $access,
    'is_active' => true,
];
$result = StudentRecordsHelper::getRecords($options);
// Inside the method, AccessManager::applyAccessScope is called automatically.
```

### 4.5 Excluding Columns and Loading Specific Relationships

```php
$result = StudentRecordsHelper::getRecords([
    'exclude' => 'password,remember_token',   // columns not to fetch
    'custom_with' => ['parents', 'studentRecord'], // load relationships
    'with_count' => ['parents'], // load the count of parents
]);
```

### 4.6 Using Caching with Tags

```php
$result = StudentRecordsHelper::getRecords([
    'cache_enabled' => true,
    'cache_ttl' => 30, // 30 minutes
    'cache_tags' => ['students', 'list'],
    'cache_force_refresh' => false,
]);
```

### 4.7 Dumping SQL for Debugging

```php
$result = StudentRecordsHelper::getRecords([
    'is_to_sql' => true,
    'gender' => 'male',
]);
// The full query will be printed in trace_log
```

### 4.8 Fetching a Collection Instead of a Paginator

```php
$result = StudentRecordsHelper::getRecords([
    'is_collection' => true,
    'is_active' => true,
    'take_limet' => 50,
]);
$students = $result['data']; // Collection
```

### 4.9 Using Legacy Filters (disable_advanced_query)

```php
$result = StudentRecordsHelper::getRecords([
    'disable_advanced_query' => true,
    'gender' => 'male', // will be transformed to where('gender', 'male') instead of scopeWhereField
]);
```

---

## 5. Helper Methods (Reusable Externally)

### 5.1 `applyFieldFilter`

Can be used to apply an advanced filter on any query:

```php
$query = Student::query();
StudentRecordsHelper::applyFieldFilter(
    $query, 'tss_school_students', 'gender', 'male,female',
    ['is_or_gender' => true], $useAdvanced = true, $useLegacy = false, 'gender'
);
// Result: WHERE gender = 'male' OR gender = 'female'
```

### 5.2 `applyImagesFilters`

Easily applies image/file filters:

```php
$query = Student::query();
StudentRecordsHelper::applyImagesFilters($query, [
    'is_has_any_images' => true,
    'is_has_files' => false,
]);
```

### 5.3 `normalizeWithArray`

Converts `custom_with` into a prepared array:

```php
$with = StudentRecordsHelper::normalizeWithArray('parents,studentRecord');
// ['parents', 'studentRecord']
```

### 5.4 `isMethodExists`

Checks whether a method exists before calling it:

```php
if (StudentRecordsHelper::isMethodExists($query, 'withRating')) {
    $query->withRating('desc');
}
```

---

## 6. Backward Compatibility

- **No old functions have been changed** in `StudentHelper` or `Tss\School\Models\StudentRecord`.
- The `StudentHelper::getRecords` function has become a simple wrapper that calls `StudentRecordsHelper::getRecords`, ensuring that existing code continues to work.
- `StudentRecordsHelper` can be used directly in new projects without needing `StudentHelper`.

---

## 7. Summary

| Feature | Details |
|---------|---------|
| **Centralised student queries** | All student retrieval queries go through a single class. |
| **Full `AdvancedQueryHelper` support** | `scopeWhereField`, `getQueryDate`, etc. |
| **`AccessManager` support** | Automatically applies permission scopes. |
| **Integrated caching** | Unique keys, tags, forced refresh. |
| **Flexible events** | Can be disabled or customised. |
| **Flexible output** | Builder, Collection, Paginator, first record, or Transformer. |
| **Easy extensibility** | Adding new filters requires only a few lines. |

**Recommended usage:**

In any API controller that needs to fetch a list of students, call `StudentRecordsHelper::getRecords` directly, passing the `access_result` option if you are using `AccessManager`. This ensures high performance and tight security.

---

## 8. Conclusion

The `StudentRecordsHelper` class is the foundation for all student queries in the Nano ecosystem. Thanks to its modular design and use of `AdvancedQueryHelper` and `AccessManager`, it offers unprecedented flexibility in building complex queries while maintaining excellent performance and high security. We strongly recommend using this class as a reference when creating `getRecords` methods for any other model (such as `Mparent` or `StudentRecord`).

**References:**

- [Full documentation of the `Tss.Student` plugin](./Docs-Student-en.md)
- [`AccessManager` class (Nano.AuthApi)](../AuthApi/Docs-AccessManager-en.md)
- [`AdvancedQueryHelper` class (Nano2.QueryBuilder)](../querybuilder/Docs-AdvancedQueryHelper.md)
- [API Plugin Development Guide (Nano-Api-SKILL)](../mcp/Nano-Api-SKILL.md)