# Documentation for `ReactionableModel` Behavior

---

## 1. Introduction

`ReactionableModel` is an advanced behavior that adds a polymorphic reactions system to any model in NanoSoft applications. This behavior allows users to react to various objects (e.g., products, articles, comments) with multiple reaction types (e.g., `like`, `heart`, `wow`, `angry`) via the `value` field. The behavior comes equipped with a rich set of scopes and helper functions that facilitate building sophisticated interaction systems with flexibility and high performance.

### Key Benefits

- **Polymorphic Reactions System** – Any model can be made reactionable.
- **Support for Multiple Types** – Ability to categorize reactions via the `value` field (e.g., `like`, `heart`, `wow`) with allowed values configurable from settings.
- **Advanced Ordering Scopes** – Order objects by reaction count, latest reaction, or earliest reaction.
- **Filtering Scopes** – Filter objects that a specific user has reacted to, that have reactions, or that have no reactions.
- **Adding the `is_reacted_by_user` Column** – Appears in query results indicating whether the current user has reacted to the object, without needing to load the relationship.
- **Consistent Interface** – Functions similar to FavoriteableModel, LikeableModel behaviors for a unified development experience.
- **Performance Improvement** – Use of efficient subqueries and avoiding full relationship loading where possible.
- **Backward Compatibility** – Retention of old scope names (`sortByCountReactionsOld`, `withSortByCountReactions`, `whereHasReaction`, etc.) to ensure existing code is not broken, with additional enhancements for controlling behavior when no user is present.

---

## 2. Usage Requirements

1. Existence of the `Nano\Markable\Models\Reaction` model (representing the reaction record) with the `nano_markable_reactions` table.
2. The model to be made reactionable must be an Eloquent model (or `October\Rain\Database\Model` in the case of OctoberCMS).
3. Users (e.g., `User` model) must be able to use the reactions system (i.e., they implement the `Authenticatable` interface).
4. Optional: If you wish to support multiple reaction types, set the allowed values via settings (`nano.markable::allowed_values.reaction`).

---

## 3. Adding the Behavior to a Model

The behavior can be added to any model using the `$implement` property (in the NanoSoft context). The behavior includes a Trait containing the scopes and helper functions, so after adding it, the model can use all features.

### Example: Making the `Product` Model Reactionable

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano\Markable\Behaviors\ReactionableModel;

class Product extends Model
{
    public $implement = [
        ReactionableModel::class
    ];
}
```

**Note:** If the model uses the `ExtensionBase` system (as is the case in NanoSoft applications), adding it is done via `$implement` as shown.

---

## 4. Core Structure

After adding the behavior, the model has the following relationship:

```php
$product->reactions() // All reaction records (all types)
```

Dynamic computed properties are also available (added automatically):

- `$product->reactions_count` – Number of reactions (can pass `value` as a parameter).
- `$product->is_reacted()` – Whether the current user has reacted to the object (can pass `value`).
- `$product->getReacted()` – Returns the reaction record for the current user (if exists).

---

## 5. Available Functions and Scopes

### 5.1 Subquery Building Functions (Internal)

#### `protected function buildReactionCountSubQuery($value = null)`
- **Description**: Builds a subquery to count the number of reaction records (COUNT).
- **Parameters**:
  - `$value` (string|null): Filter by reaction type (e.g., `like`, `heart`).

#### `protected function buildReactionLatestDateSubQuery($field = 'created_at', $value = null)`
- **Description**: Builds a subquery for the latest reaction date (MAX).

#### `protected function buildReactionEarliestDateSubQuery(...)` – Uses `MIN(field)` to get the earliest date.

### 5.2 Reaction Count Scopes (COUNT)

#### `scopeAddCountReactions(Builder $query, $orderDirection = 'DESC', $columnName = 'reactions_count', $value = null)`
- **Description**: Adds a column containing the number of reaction records to the query result (without ordering).
- **Example**:
  ```php
  $products = Product::addCountReactions()->get();
  foreach ($products as $product) {
      echo $product->reactions_count;
  }
  ```

#### `scopeSortByCountReactions(...)` – Orders results by reaction count (without adding the column).

#### `scopeWithCountReactions(...)` – Combines `addCountReactions` and `sortByCountReactions` (adds column and ordering together).

**Example**:
```php
$products = Product::withCountReactions('DESC', 'reactions_count', 'heart')->get(); // reactions of type 'heart' only
```

### 5.3 Reaction Date Scopes

#### `scopeAddLatestReaction(...)` – Adds a column with the latest reaction date (optionally specifying the field via `$field`).
#### `scopeSortByLatestReaction(...)` – Orders by the latest reaction date.
#### `scopeWithLatestReaction(...)` – Adds the column and ordering together.

**Example**:
```php
$products = Product::withLatestReaction('DESC', 'last_reaction', 'heart')->get();
```

#### `scopeAddEarliestReaction(...)` – Adds a column with the earliest reaction date.
#### `scopeSortByEarliestReaction(...)` – Orders by the earliest reaction date.
#### `scopeWithEarliestReaction(...)` – Adds the column and ordering together.

### 5.4 Filtering Scopes by User

#### `scopeReactedByUser(Builder $query, $user = null, $value = null)`
- **Description**: Filters objects that a specific user has reacted to (if `$user` is not passed, the current user is used).
- **Example**:
  ```php
  // Products that the current user has reacted to (any type)
  $products = Product::reactedByUser()->get();
  
  // Products that a specific user has reacted to as type 'heart'
  $products = Product::reactedByUser($user, 'heart')->get();
  ```

#### `scopeNotReactedByUser(...)` – Filters objects that a specific user has **not** reacted to.

### 5.5 Filtering Scopes by Existence of Reactions

#### `scopeHasReactions(Builder $query, $minCount = 1, $value = null)`
- **Description**: Filters objects that have at least `$minCount` reactions (with filtering by `value`).
- **Example**:
  ```php
  // Products that have at least 5 reactions of type 'heart'
  $products = Product::hasReactions(5, 'heart')->get();
  ```

#### `scopeHasNoReactions(...)` – Filters objects that have no reactions.

### 5.6 Scope for Adding Reaction Status Column for Current User

#### `scopeWithIsReactedByUser(Builder $query, $user = null, $columnName = 'is_reacted_by_user', $value = null)`
- **Description**: Adds a boolean column (`1` or `0`) to the query result indicating whether the specified user has reacted to the object.
- **Example**:
  ```php
  $products = Product::withIsReactedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_reacted_by_user) {
          echo "You have reacted to this product";
      }
  }
  ```

### 5.7 Special Scopes

#### `scopeTopReacted(Builder $query, $limit = 10, $orderDirection = 'DESC', $value = null)`
- **Description**: Retrieves the objects with the most reactions (by reaction count), with a limit.
- **Example**:
  ```php
  $topProducts = Product::topReacted(10, 'DESC', 'heart')->get();
  ```

### 5.8 Helper Statistical Functions

#### `getTotalReactions($value = null)`
- **Description**: Total number of reactions (sum of `COUNT`) according to filtering options.
- **Output**: Integer.

#### `getReactionsCountByType($value = null)`
- **Description**: Distribution of reactions by user type (`user_type`).
- **Output**: `Collection` where the key is `user_type` and the value is the count.

#### `getReactorsUsers($value = null)`
- **Description**: List of users who reacted to the object (loads the `user` relationship).
- **Output**: `Collection` of user objects.

---

## 6. Comprehensive Practical Examples

### Example 1: Adding a Reaction and Checking

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Reaction;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add a reaction of type 'heart'
$reaction = Reaction::add($product, $user, 'heart');

// Add a reaction of type 'wow'
$reaction = Reaction::add($product, $user, 'wow', ['notes' => 'Amazing!']);

// Check if the user has reacted to the product (any type)
if ($product->isReacted()) {
    echo "You have reacted to this product";
}

// Check if the reaction is of type 'heart'
if ($product->isReacted(null, 'heart')) {
    echo "You have hearted this product";
}
```

### Example 2: Removing a Reaction

```php
// Remove the reaction (any type)
Reaction::remove($product, $user);

// Remove a reaction of type 'heart' only
Reaction::remove($product, $user, 'heart');
```

### Example 3: Ordering Products by Reaction Count

```php
$topProducts = Product::topReacted(10)->get();
```

### Example 4: Adding the `is_reacted_by_user` Column in Product List

```php
$products = Product::withIsReactedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_reacted_by_user ? 'Reacted' : 'Not reacted');
}
```

### Example 5: Filtering Products that the Current User Has Reacted To

```php
$myReactedProducts = Product::reactedByUser()->get();
```

### Example 6: Statistics

```php
$product = Product::find(1);
echo "Total reactions of type 'heart': " . $product->getTotalReactions('heart');
print_r($product->getReactionsCountByType('heart')->toArray());
$users = $product->getReactorsUsers('heart');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### Example 7: Combining Advanced Scopes

```php
// Products that have more than 5 reactions of type 'heart', ordered by latest reaction
$products = Product::hasReactions(5, 'heart')
    ->sortByLatestReaction('DESC', 'latest_heart_at')
    ->get();
```

---

## 7. Backward Compatibility Functions

To maintain existing code that uses previous scope names, wrapper functions are provided in the main class. These functions are marked `@deprecated` and it is recommended to use the new alternatives.

**List of Supported Old Functions:**

| Old Function | New Function |
|--------------|--------------|
| `scopeSortByCountReactionsOld` | `scopeSortByCountReactions` |
| `scopeWithSortByCountReactions` | `scopeWithCountReactions` |
| `scopeSortByCreatedAtReactions` | `scopeSortByLatestReaction` |
| `scopeAddSortByCreatedAtReactions` | `scopeAddLatestReaction` |
| `scopeWithSortByCreatedAtReactions` | `scopeWithLatestReaction` |
| `scopeWhereHasReaction` | `scopeReactedByUser` (with improvements) |
| `scopeWhereHasReactions` | `scopeReactedByUser` |
| `scopeWhereHasUserReaction` | `scopeReactedByUser` |
| `scopeWhereHasUserReactions` | `scopeReactedByUser` |

**Note regarding the old `whereHasReaction` functions and their counterparts:** They have been improved to support the `$isForceUser` parameter for controlling query behavior when no user is present, with the default value read from settings (`nano.markable::reactionable.scope.where_has_reaction.is_force_user`). The default behavior (`true`) adds an impossible condition to return no results (matching the original behavior).

---

## 8. Performance Optimization and Caching

The behavior relies on optimized subqueries to avoid loading relationships entirely. There is no built-in caching system by default, but you can rely on the caching mechanism of the models themselves or use application-level caching if you need to repeat heavy queries.

- When using `withCountReactions` or `sortByCountReactions`, only one subquery is executed.
- Using `withIsReactedByUser` adds a subquery per row, so it is recommended to use it cautiously on large lists, but it is usually acceptable due to the efficiency of SQL queries.

**Note:** If you add `SoftDeletes` to the `Reaction` model in the future, the scopes can be easily modified to support `withTrashed`.

---

## 9. Conclusion

`ReactionableModel` is a comprehensive and powerful solution for managing the reactions system in NanoSoft applications. Thanks to the variety of scopes and helper functions, developers can easily build sophisticated interaction systems while maintaining high flexibility and scalability. Backward compatibility ensures a smooth transition from previous versions. We hope this documentation serves as a comprehensive reference for all the behavior's capabilities.
