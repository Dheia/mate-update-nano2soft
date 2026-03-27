# Documentation for `FollowableModel` Behavior

---

## 1. Introduction

`FollowableModel` is an advanced behavior that adds a polymorphic following system to any model in NanoSoft applications. This behavior allows users to follow various objects (such as articles, products, users, etc.) with comprehensive support for acceptance options, activity status, and restoration of deleted follows. The behavior comes with a rich set of scopes and helper functions that facilitate building advanced social systems with flexibility and high performance.

---

## 2. Key Benefits and Features

- **Polymorphic Following System** – Any model can be made followable.
- **Flexibility in Follow Statuses** – Support for `is_accepted` (follow acceptance) and `is_active` (follow activity) fields to represent multiple states.
- **Support for Soft Deletes** – Ability to include soft-deleted follows in queries as needed.
- **Advanced Sorting Scopes** – Sort objects by follow count, latest follow, or earliest follow, with options to filter by acceptance and activity.
- **Filtering Scopes** – Filter objects followed by a specific user, objects that have followers, or objects that have no followers.
- **Adding `is_followed_by_user` Column** – Appears in query results indicating whether the current user follows the object, without needing to load the relationship.
- **Consistent Interface** – Functions similar to FavoriteableModel and ReviewRateable behaviors for a unified development experience.
- **Performance Optimization** – Uses efficient subqueries and avoids loading full relationships where possible.
- **Backward Compatibility** – Preserves old scope names (`addSortByCountFollows`, `withSortByLatestAtFollows`, etc.) to ensure legacy code does not break.

---

## 3. Usage Requirements

1. Existence of the `Nano\Follows\Models\Follow` model (which represents the follow record) with its `nano_follows_follows` table.
2. The model to be made followable must be an Eloquent model (or `October\Rain\Database\Model` in the case of OctoberCMS).
3. Users (such as the `User` model) must be able to use the following system (i.e., they satisfy the `Authenticatable` interface).
4. Optional: Use the `SoftDeletes` trait in the `Follow` model to benefit from the `withTrashed` option.

---

## 4. Installation and Setup

### Prerequisites
- NanoSoft System version 3 or higher.
- The `Nano\Follows\Models\Follow` model with its `nano_follows_follows` table.
- The model to be made followable must use the `ExtensionBase` system (or be a `Model` in OctoberCMS).

### Setup Steps
1. Add the behavior to the model via the `$implement` property:

```php
class Product extends Model
{
    public $implement = [
        \Nano\Follows\Behaviors\FollowableModel::class
    ];
}
```

2. Ensure that the follows table (`nano_follows_follows`) contains at least the following fields:
   - `followable_id`, `followable_type` (for the polymorphic relationship)
   - `user_id`, `user_type` (for the relationship to the user)
   - `is_accepted`, `is_active`, `accepted_at`, `re_follow_at` (for follow statuses)
   - `deleted_at` (if using Soft Deletes)

3. No need to add any additional fields to the model; the `follows` relationship is added automatically.

---

## 5. Basic Structure

After adding the behavior, the model gets the following relationship:

```php
$product->follows() // Returns a morphMany relationship linking Follow records to this model
```

A shortcut function is also available to check if the current user follows:

```php
if ($product->isUserFollow()) {
    // The user follows this product
}
```

---

## 6. Available Methods and Scopes

### 6.1 Subquery Builder Methods (Internal)

#### `protected function buildFollowCountSubQuery($onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Builds a subquery to count the follows for the current object, with filtering options.
- **Parameters**:
  - `$onlyAccepted` (bool|null): `true` for only accepted follows, `false` for only non-accepted follows, `null` for all.
  - `$onlyActive` (bool|null): `true` for only active follows, `false` for only inactive follows, `null` for all.
  - `$withTrashed` (bool): Include soft-deleted follows.
- **Output**: A subquery of type `\Illuminate\Database\Query\Builder`.

#### `protected function buildFollowLatestDateSubQuery($field = 'created_at', $onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Builds a subquery for the latest follow date (MAX).
- **Parameters**:
  - `$field`: The name of the timestamp field (`created_at`, `updated_at`, `accepted_at`, `re_follow_at`).
  - Other parameters as above.

#### `protected function buildFollowFirstDateSubQuery($field = 'created_at', $onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Builds a subquery for the earliest follow date (MIN).

### 6.2 Follow Count Scopes

#### `scopeAddCountFollows(Builder $query, $orderDirection = 'DESC', $columnName = 'follows_count', $onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Adds a column containing the follow count to the query result (without sorting).
- **Example**:
  ```php
  $products = Product::addCountFollows()->get();
  foreach ($products as $product) {
      echo $product->follows_count; // Number of product followers
  }
  ```

#### `scopeSortByCountFollows(Builder $query, $orderDirection = 'DESC', $columnName = 'follows_count', $onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Sorts the results by follow count (without adding the column).
- **Example**:
  ```php
  // Sort products descending by the number of accepted follows
  $products = Product::sortByCountFollows('DESC', 'follows_count', true)->get();
  ```

#### `scopeWithCountFollows(Builder $query, $orderDirection = 'DESC', $columnName = 'follows_count', $onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Combines `addCountFollows` and `sortByCountFollows` (adds the column and sorts together).
- **Example**:
  ```php
  $products = Product::withCountFollows('DESC', 'follows_count', true)->get();
  ```

### 6.3 Latest Follow Date Scopes

#### `scopeAddLatestFollow(Builder $query, $orderDirection = 'DESC', $columnName = 'latest_follow_at', $onlyAccepted = null, $onlyActive = null, $withTrashed = false, $field = 'created_at')`
- **Description**: Adds a column with the latest follow date.

#### `scopeSortByLatestFollow(...)` – Sorts by the latest follow date (without adding column).

#### `scopeWithLatestFollow(...)` – Adds the column and sorts together.

**Example**:
```php
// Sort products by the latest follow (created_at) including non-accepted follows
$products = Product::sortByLatestFollow('DESC', 'latest_follow_at', null, null, false, 'created_at')->get();
```

### 6.4 Earliest Follow Date Scopes

#### `scopeAddEarliestFollow(...)` – Adds a column with the earliest follow date.
#### `scopeSortByEarliestFollow(...)` – Sorts by the earliest follow date.
#### `scopeWithEarliestFollow(...)` – Adds the column and sorts.

**Example**:
```php
$products = Product::withEarliestFollow('ASC', 'first_follow_at', true, true)->get(); // Earliest accepted and active follow
```

### 6.5 Filtering Scopes by User

#### `scopeFollowedByUser(Builder $query, $user = null, $isAccepted = true, $isActive = null, $withTrashed = false)`
- **Description**: Filters objects followed by a specific user (if `$user` is not passed, uses the current user).
- **Example**:
  ```php
  // Products followed by the current user
  $products = Product::followedByUser()->get();
  
  // Products followed by a specific user (even if the follow is not accepted)
  $products = Product::followedByUser($user, false)->get();
  ```

#### `scopeNotFollowedByUser(Builder $query, $user = null, $isAccepted = true, $isActive = null, $withTrashed = false)`
- **Description**: Filters objects **not** followed by a specific user.

#### `scopeWhereHasFollow($builder, $user = null, $is_accepted = true)` – (Legacy, for backward compatibility)
- **Description**: Returns objects followed by the specified user with the ability to specify acceptance status. Does not support `is_active` or `withTrashed` options. It is recommended to use `FollowedByUser` for greater flexibility.
- **Example**:
  ```php
  $products = Product::whereHasFollow($user, true)->get();
  ```

#### `scopeWhereHasFollows($builder, $user = null, $is_accepted = true)` – (Legacy, for backward compatibility)
- **Description**: Alias for `whereHasFollow` (exists for historical reasons). Works the same way.

### 6.6 Filtering Scopes by Follower Presence

#### `scopeHasFollowers(Builder $query, $minCount = 1, $isAccepted = null, $isActive = null, $withTrashed = false)`
- **Description**: Filters objects that have at least `$minCount` followers (with filtering options).
- **Example**:
  ```php
  // Products with at least 5 accepted followers
  $products = Product::hasFollowers(5, true)->get();
  ```

#### `scopeHasNoFollowers(Builder $query, $isAccepted = null, $isActive = null, $withTrashed = false)`
- **Description**: Filters objects that have no followers.

### 6.7 Scope for Adding Current User's Follow Status Column

#### `scopeWithIsFollowedByUser(Builder $query, $user = null, $columnName = 'is_followed_by_user', $isAccepted = true, $isActive = null, $withTrashed = false)`
- **Description**: Adds a boolean column (`1` or `0`) to the query result indicating whether the specified user follows the object.
- **Example**:
  ```php
  $products = Product::withIsFollowedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_followed_by_user) {
          echo "You follow this product";
      }
  }
  ```

### 6.8 Special Scopes

#### `scopeTopFollowed(Builder $query, $limit = 10, $orderDirection = 'DESC', $onlyAccepted = true, $onlyActive = null)`
- **Description**: Retrieves the most followed objects (by number of accepted followers) with a limit.
- **Example**:
  ```php
  $topProducts = Product::topFollowed(5)->get();
  ```

#### `scopeSortByAcceptedFollowsCount(Builder $query, $orderDirection = 'DESC', $columnName = 'accepted_follows_count')`
- Shortcut for `sortByCountFollows` with `$onlyAccepted = true`.

#### `scopeSortByActiveFollowsCount(Builder $query, $orderDirection = 'DESC', $columnName = 'active_follows_count')`
- Shortcut for `sortByCountFollows` with `$onlyActive = true`.

### 6.9 Statistical Helper Functions

#### `getFollowersCountByType($onlyAccepted = null, $onlyActive = null)`
- **Description**: Returns the number of followers grouped by `user_type`.
- **Output**: `Collection` where the key is `user_type` and the value is the count.
- **Example**:
  ```php
  $counts = $product->getFollowersCountByType(true, null);
  // ['RainLab\User\Models\User' => 42, 'App\Models\Admin' => 5]
  ```

#### `getTotalFollowers($onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Total number of followers (or based on filters).
- **Output**: Integer.

#### `getFollowersUsers($onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Returns a collection of users who follow the object (loads the `user` relationship).
- **Output**: `Collection` containing user objects.

### 6.10 Core Functions in the Behavior (Not in the Trait)

#### `followedBy()`
- **Description**: Returns the collection of users who follow the current object.
- **Output**: `Collection` of users (with the `user` relationship loaded).
- **Example**:
  ```php
  $followers = $product->followedBy();
  foreach ($followers as $user) {
      echo $user->name;
  }
  ```

#### `isUserFollow($user = null)`
- **Description**: Checks if the user (or current user) follows the object.
- **Output**: `bool`.

#### `getUserFollow($user = null)`
- **Description**: Returns the follow record for the user (if exists).
- **Output**: `Follow` object or `null`.

#### `addFollow($data, $user = null, $parent = null): Follow`
- **Description**: Adds a new follow for the object. `$data` can be passed as an `array` or as a `string` (will be interpreted as `content`).
- **Output**: The created `Follow` object.

#### `toggleFollow($data, $user = null, $parent = null)`
- **Description**: If the user is following, it deletes the follow (or restores it if soft-deleted); otherwise, it adds a new follow.
- **Output**: `Follow` object or `null` on deletion.

#### `deleteFollow($id, $data = null): ?bool`
- **Description**: Deletes a specific follow (Soft delete).
- **Output**: `true` if deletion succeeded, `false` if failed, `null` in some cases.

#### `updateFollow($id, $data): Follow`
- **Description**: Updates data of an existing follow.

#### `countFollow(bool $onlyAccepted = false)`
- **Description**: Number of follows (optionally only accepted ones).
- **Output**: Integer.

#### `getSuggestedFriends()`
- **Description**: (Specific to user objects) Returns suggested friends based on the object's follows.
- **Output**: Collection of users.

---

## 7. Summary Table of Functions and Scopes by Category

| Category | Functions/Scopes |
|----------|------------------|
| **Adding Follows** | `addFollow()`, `toggleFollow()`, `deleteFollow()`, `updateFollow()` |
| **Querying Follows** | `followedBy()`, `isUserFollow()`, `getUserFollow()` |
| **Filtering Scopes by User** | `followedByUser()`, `notFollowedByUser()`, `whereHasFollow()` (legacy) |
| **Filtering Scopes by Follower Presence** | `hasFollowers()`, `hasNoFollowers()` |
| **Sorting Scopes by Count** | `sortByCountFollows()`, `addCountFollows()`, `withCountFollows()` |
| **Sorting Scopes by Date** | `sortByLatestFollow()`, `addLatestFollow()`, `withLatestFollow()`, `sortByEarliestFollow()`, `addEarliestFollow()`, `withEarliestFollow()` |
| **Special Scopes** | `topFollowed()`, `sortByAcceptedFollowsCount()`, `sortByActiveFollowsCount()` |
| **Statistical Functions** | `getFollowersCountByType()`, `getTotalFollowers()`, `getFollowersUsers()` |
| **Additional Functions** | `countFollow()`, `getSuggestedFriends()` |
| **Adding Follow Status Column** | `withIsFollowedByUser()` |

---

## 8. Comprehensive Practical Examples

### Example 1: Adding and Removing a Follow

```php
$user = Auth::getUser();
$product = Product::find(1);

// Add a follow
$follow = $product->addFollow(['is_accepted' => false], $user);
// Now $user follows the product but awaits acceptance

// Check the follow
if ($product->isUserFollow($user)) {
    echo "Follows this product";
}

// Unfollow (delete)
$product->toggleFollow([], $user);
// After toggling, it was deleted
```

### Example 2: Sorting Products by Follower Count

```php
// Most followed products (accepted only)
$products = Product::topFollowed(10, 'DESC', true)->get();

// Products sorted by number of active follows (including soft-deleted ones)
$products = Product::sortByCountFollows('DESC', 'follows_count', null, true, true)->get();
```

### Example 3: Filtering Products Followed by the Current User

```php
$myFollowedProducts = Product::followedByUser()->get();

// Including non-accepted follows
$allFollows = Product::followedByUser(null, false)->get();
```

### Example 4: Adding `is_followed_by_user` Column to a Product List

```php
$products = Product::withIsFollowedByUser()->paginate(20);

foreach ($products as $product) {
    echo $product->name . ' - ';
    echo $product->is_followed_by_user ? 'Followed' : 'Not followed';
}
```

### Example 5: Using Advanced Scopes

```php
// Products with more than 5 accepted followers, sorted by the latest follow (using created_at field)
$products = Product::hasFollowers(5, true)
    ->sortByLatestFollow('DESC', 'latest_follow_at', true, null, false, 'created_at')
    ->get();
```

### Example 6: Follow Statistics

```php
$product = Product::find(1);

echo "Total followers: " . $product->getTotalFollowers(true, true); // Accepted and active
echo "Followers by type: ";
print_r($product->getFollowersCountByType(true, true)->toArray());

$users = $product->getFollowersUsers(true, true);
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### Example 7: Combining `followedByUser` and `withIsFollowedByUser`

```php
// Get products followed by the current user, adding the is_followed_by_user column
// (The column will be 1 for all products in the result because they are all followed)
$products = Product::followedByUser()
    ->withIsFollowedByUser()
    ->get();
```

---

## 9. Backward Compatibility Functions

To preserve legacy code that uses previous scope names (`addSortByCountFollows`, `withSortByLatestAtFollows`, etc.), wrapper functions are provided that redirect to the new functions. These functions are marked `@deprecated` and it is recommended to use the new alternatives.

**List of Supported Legacy Functions:**

- `scopeAddSortByCountFollows` → `scopeAddCountFollows`
- `scopeWithSortByCountFollows` → `scopeWithCountFollows`
- `scopeAddSortByLatestAtFollows` → `scopeAddLatestFollow`
- `scopeSortByLatestAtFollows` → `scopeSortByLatestFollow`
- `scopeWithSortByLatestAtFollows` → `scopeWithLatestFollow`
- `scopeAddSortByFirstAtFollows` → `scopeAddEarliestFollow`
- `scopeSortByFirstAtFollows` → `scopeSortByEarliestFollow`
- `scopeWithSortByFirstAtFollows` → `scopeWithEarliestFollow`

**Note:** `scopeSortByCountFollows` did not change its name, so no wrapper function is needed.

**Additional Note:** The legacy scopes `whereHasFollow` and `whereHasFollows` still exist in the code, but it is recommended to use `FollowedByUser` for greater flexibility.

---

## 10. Performance Optimization and Caching

The behavior relies on optimized subqueries to avoid loading full relationships. There is no built-in caching system by default, but you can rely on the caching mechanism of the models themselves or use application caching if you need to repeat heavy queries.

- When using `withCountFollows` or `sortByCountFollows`, only one subquery is executed.
- Using `withIsFollowedByUser` adds a subquery per row, so it is recommended to use it cautiously on large lists, but it is generally acceptable due to the efficiency of SQL queries.

**Note regarding `withTrashed`:** When using `withTrashed = true` in any of the scopes or statistical functions, ensure that the `Nano\Follows\Models\Follow` model uses the `SoftDeletes` trait; otherwise, query errors will occur.

---

## 11. Conclusion

`FollowableModel` is a comprehensive and powerful solution for managing the follow system in NanoSoft applications. Thanks to the variety of scopes and helper functions, developers can easily build advanced social systems while maintaining high flexibility and scalability. Backward compatibility ensures a smooth transition from previous versions. We hope this documentation serves as a complete reference for all the capabilities of this behavior.