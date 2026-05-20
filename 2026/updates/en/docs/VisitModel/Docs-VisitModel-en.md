# Documentation for `VisitModel` Behavior

---

## 1. Introduction

`VisitModel` is an advanced behavior that adds a polymorphic visit and view tracking system to any model in NanoSoft applications. This behavior allows developers to record and analyze user visits to various objects (such as articles, products, pages) with comprehensive support for advanced filtering options (type, process, event) and states (record activity, soft deletes). The behavior comes with a rich set of scopes and helper functions that facilitate building sophisticated analytics systems with flexibility and high performance.

### Key Benefits

- **Polymorphic Tracking System** – Visits can be tracked for any model.
- **Support for Multiple Types** – Fields `type` (visit type: add, out, remove, reset, level_up), `process_type` (in/out), and `event_type` (event type) to represent diverse states and contexts.
- **Support for Soft Deletes** – Ability to include soft‑deleted visits in queries as needed.
- **Advanced Sorting Scopes** – Sort objects by visit count, total visits, total views, or dates (latest/earliest visit), with flexible filtering options.
- **Filtering Scopes** – Filter objects visited by a specific user, objects that have visits, or objects that have no visits.
- **Adding `is_visited_by_user` Column** – Appears in query results indicating whether the current user has visited the object, without needing to load the relationship.
- **Consistent Interface** – Functions similar to the `FollowableModel` behavior for a unified development experience.
- **Performance Optimization** – Uses efficient subqueries and avoids loading full relationships where possible.
- **Backward Compatibility** – Preserves old scope names (`sortByCountVisitsOld`, `withSortBySumVisits`, etc.) to ensure legacy code does not break.

---

## 2. Usage Requirements

1. Existence of the `Nano2\Visitors\Models\Visit` model (which represents the visit record) with its `nano2_visitors_visits` table.
2. The model to have its visits tracked must be an Eloquent model (or `October\Rain\Database\Model` in the case of OctoberCMS).
3. Users (such as the `User` model) must be able to use the visit system (i.e., they satisfy the `Authenticatable` interface).
4. Optional: Use the `SoftDeletes` trait in the `Visit` model to benefit from the `withTrashed` option.

---

## 3. Adding the Behavior to a Model

The behavior can be added to any model using the `$implement` property (in the NanoSoft context). The behavior includes a trait that holds the scopes and helper functions, so after adding it the model becomes capable of using all features.

### Example: Making the `Product` Model Visitable

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano2\Visitors\Behaviors\VisitModel;

class Product extends Model
{
    public $implement = [
        VisitModel::class
    ];
}
```

**Note:** If the model uses the `ExtensionBase` system (as is the case in NanoSoft applications), the addition is done via `$implement` as shown.

---

## 4. Basic Structure

After adding the behavior, the model gets the following relationship:

```php
$product->visits() // Returns a morphMany relationship linking Visit records to this model
```

Computed properties are also available for easy access to visit statistics:

```php
$product->visits_count   // Number of visit records
$product->sum_visits     // Total visits (visits field)
$product->sum_views      // Total views (views field)
$product->is_visits      // Whether there are any visits
```

---

## 5. Available Methods and Scopes

### 5.1 Subquery Builder Methods (Internal)

#### `protected function buildVisitCountSubQuery($onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **Description**: Builds a subquery to count the number of visit records (COUNT).
- **Parameters**:
  - `$onlyActive` (bool|null): `true` for only active visits, `false` for only inactive visits, `null` for all.
  - `$withTrashed` (bool): Include soft‑deleted visits.
  - `$type` (string|null): Filter by visit type (`add`, `out`, `remove`, ...).
  - `$processType` (string|null): Filter by process type (`in`, `out`).
  - `$eventType` (string|null): Filter by event type.

#### `protected function buildVisitSumSubQuery(...)` – Same parameters, but uses `SUM(visits)`.

#### `protected function buildViewSumSubQuery(...)` – Uses `SUM(views)`.

#### `protected function buildVisitLatestDateSubQuery($field = 'created_at', ...)` – Uses `MAX(field)` to get the latest date.

#### `protected function buildVisitEarliestDateSubQuery($field = 'created_at', ...)` – Uses `MIN(field)` to get the earliest date.

### 5.2 Visit Count Scopes (COUNT)

#### `scopeAddCountVisits(Builder $query, $orderDirection = 'DESC', $columnName = 'visits_count', $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **Description**: Adds a column containing the number of visit records to the query result (without sorting).
- **Example**:
  ```php
  $products = Product::addCountVisits()->get();
  foreach ($products as $product) {
      echo $product->visits_count;
  }
  ```

#### `scopeSortByCountVisits(...)` – Sorts the results by visit count (without adding the column).

#### `scopeWithCountVisits(...)` – Combines `addCountVisits` and `sortByCountVisits` (adds the column and sorts together).

**Example**:
```php
$products = Product::withCountVisits('DESC', 'visits_count', true)->get(); // Only active visits
```

### 5.3 Total Visits Scopes (SUM visits)

#### `scopeAddSumVisits(...)` – Adds a column with the total visits (field `visits`).
#### `scopeSortBySumVisits(...)` – Sorts by total visits.
#### `scopeWithSumVisits(...)` – Adds the column and sorts together.

**Example**:
```php
$products = Product::sortBySumVisits('DESC', 'total_visits', true)->get();
```

### 5.4 Total Views Scopes (SUM views)

#### `scopeAddSumViews(...)` – Adds a column with the total views (field `views`).
#### `scopeSortBySumViews(...)` – Sorts by total views.
#### `scopeWithSumViews(...)` – Adds the column and sorts together.

### 5.5 Latest Visit Date Scopes

#### `scopeAddLatestVisit(...)` – Adds a column with the latest visit date (optionally specify the timestamp field via `$field`).
#### `scopeSortByLatestVisit(...)` – Sorts by the latest visit date.
#### `scopeWithLatestVisit(...)` – Adds the column and sorts together.

**Example**:
```php
$products = Product::withLatestVisit('DESC', 'last_visit', true, false, 'add', 'in', 'contest', 'last_visit_at')->get();
```

### 5.6 Earliest Visit Date Scopes

#### `scopeAddEarliestVisit(...)` – Adds a column with the earliest visit date.
#### `scopeSortByEarliestVisit(...)` – Sorts by the earliest visit date.
#### `scopeWithEarliestVisit(...)` – Adds the column and sorts together.

### 5.7 Filtering Scopes by User

#### `scopeVisitedByUser(Builder $query, $user = null, $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **Description**: Filters objects visited by a specific user (if `$user` is not passed, uses the current user).
- **Example**:
  ```php
  // Products visited by the current user
  $products = Product::visitedByUser()->get();
  
  // Products visited by a specific user for a specific event
  $products = Product::visitedByUser($user, null, false, null, null, 'contest')->get();
  ```

#### `scopeNotVisitedByUser(...)` – Filters objects **not** visited by a specific user.

### 5.8 Filtering Scopes by Visit Presence

#### `scopeHasVisits(Builder $query, $minCount = 1, $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **Description**: Filters objects that have at least `$minCount` visits (with filtering options).
- **Example**:
  ```php
  // Products with at least 5 active visits
  $products = Product::hasVisits(5, true)->get();
  ```

#### `scopeHasNoVisits(...)` – Filters objects that have no visits.

### 5.9 Scope for Adding Current User's Visit Status Column

#### `scopeWithIsVisitedByUser(Builder $query, $user = null, $columnName = 'is_visited_by_user', $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **Description**: Adds a boolean column (`1` or `0`) to the query result indicating whether the specified user has visited the object.
- **Example**:
  ```php
  $products = Product::withIsVisitedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_visited_by_user) {
          echo "You have visited this product";
      }
  }
  ```

### 5.10 Special Scopes

#### `scopeTopVisited(Builder $query, $limit = 10, $orderDirection = 'DESC', $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **Description**: Retrieves the most visited objects (by record count) with a limit.
- **Example**:
  ```php
  $topProducts = Product::topVisited(5, 'DESC', true)->get();
  ```

#### `scopeTopSumVisits(Builder $query, $limit = 10, $orderDirection = 'DESC', $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **Description**: Retrieves the most visited objects (by sum of `visits`) with a limit.

### 5.11 Statistical Helper Functions

#### `getTotalVisits($onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **Description**: Total number of visits (sum of `visits`) based on filtering options.
- **Output**: Integer.

#### `getTotalViews(...)` – Total number of views (sum of `views`).

#### `getVisitsCountByType($onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **Description**: Distribution of visits by user type (`user_type`).
- **Output**: `Collection` where the key is `user_type` and the value is the count.
- **Example**:
  ```php
  $counts = $product->getVisitsCountByType(true);
  // ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]
  ```

#### `getVisitorsUsers(...)`
- **Description**: List of users who have visited the object (loads the `user` relationship).
- **Output**: `Collection` of user objects.

---

## 6. Comprehensive Practical Examples

### Example 1: Recording a New Visit

Using the `Manager` class (provided), a visit can be recorded for a product:

```php
use Nano2\Visitors\Classes\Manager;

$product = Product::find(1);
$user = Auth::getUser();

Manager::createVisits([
    'visitable' => $product,
    'user' => $user,
    'type' => 'add',
    'process_type' => 'in',
    'event_type' => 'product_view',
    'visits' => 1,
    'views' => 1,
]);
```

### Example 2: Sorting Products by Visit Count

```php
$topProducts = Product::topVisited(10, 'DESC', true)->get();
```

### Example 3: Adding the `is_visited_by_user` Column to a Product List

```php
$products = Product::withIsVisitedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_visited_by_user ? 'Visited' : 'Not visited');
}
```

### Example 4: Filtering Products Visited by the Current User

```php
$myVisitedProducts = Product::visitedByUser()->get();
```

### Example 5: Filtering by Type, Process, and Event

```php
// Products visited by the current user with type 'add' (entry) and event 'contest'
$products = Product::visitedByUser(null, null, false, 'add', 'in', 'contest')->get();
```

### Example 6: Visit Statistics for a Specific Product

```php
$product = Product::find(1);

echo "Total visits: " . $product->getTotalVisits(true);
echo "Total views: " . $product->getTotalViews(true);
echo "Visit distribution by user type: ";
print_r($product->getVisitsCountByType(true)->toArray());

$visitors = $product->getVisitorsUsers(true);
foreach ($visitors as $user) {
    echo $user->name . "\n";
}
```

### Example 7: Combining Advanced Scopes

```php
// Products with more than 5 active visits of type 'add', sorted by latest visit
$products = Product::hasVisits(5, true, false, 'add')
    ->sortByLatestVisit('DESC', 'last_visit')
    ->get();
```

---

## 7. Backward Compatibility Functions

To preserve legacy code that uses previous scope names, wrapper functions are provided in the main class. These functions are marked `@deprecated` and it is recommended to use the new alternatives.

**List of Supported Legacy Functions:**

| Legacy Function | New Function |
|-----------------|--------------|
| `scopeSortByCountVisitsOld` | `scopeSortByCountVisits` |
| `scopeWithSortByCountVisits` | `scopeWithCountVisits` |
| `scopeSortBySumVisitsOld` | `scopeSortBySumVisits` |
| `scopeWithSortBySumVisits` | `scopeWithSumVisits` |
| `scopeSortBySumViewsOld` | `scopeSortBySumViews` |
| `scopeWithSortBySumViews` | `scopeWithSumViews` |
| `scopeWhereHasVisit` | `scopeVisitedByUser` |
| `scopeWhereHasVisits` | `scopeVisitedByUser` |

**Note:** The legacy functions `scopeWhereHasVisit` and `scopeWhereHasVisits` have been enhanced to support the `$isForceUser` parameter for controlling query behavior when no user is present, while preserving the original behavior (returning no results) as the default.

---

## 8. Performance Optimization and Caching

The behavior relies on optimized subqueries to avoid loading full relationships. There is no built-in caching system by default, but you can rely on the caching mechanism of the models themselves or use application caching if you need to repeat heavy queries.

- When using `withCountVisits` or `sortByCountVisits`, only one subquery is executed.
- Using `withIsVisitedByUser` adds a subquery per row, so it is recommended to use it cautiously on large lists, but it is generally acceptable due to the efficiency of SQL queries.

**Note regarding `withTrashed`:** When using `withTrashed = true` in any of the scopes or statistical functions, ensure that the `Nano2\Visitors\Models\Visit` model uses the `SoftDeletes` trait; otherwise, query errors will occur.

---

## 9. Conclusion

`VisitModel` is a comprehensive and powerful solution for managing visit and view tracking in NanoSoft applications. Thanks to the variety of scopes and helper functions, developers can easily build advanced analytics systems while maintaining high flexibility and scalability. Backward compatibility ensures a smooth transition from previous versions. We hope this documentation serves as a complete reference for all the capabilities of this behavior.