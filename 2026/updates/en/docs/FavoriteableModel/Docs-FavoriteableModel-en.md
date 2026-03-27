# Documentation for `FavoriteableModel` Behavior

---

## 1. Introduction

`FavoriteableModel` is an advanced behavior that adds a polymorphic favorites system to any model in NanoSoft applications. This behavior allows users to add objects (e.g., products, articles, users) to their favorites list, with support for distinguishing different types of favorites (e.g., `favorite`, `like`, `bookmark`) via the `value` field. The behavior comes equipped with a rich set of scopes and helper functions that facilitate building sophisticated favorites systems with flexibility and high performance.

### Key Benefits

- **Polymorphic Favorites System** – Any model can be made favoritable.
- **Support for Multiple Types** – Ability to categorize favorites via the `value` field (e.g., `favorite`, `like`, `bookmark`).
- **Advanced Ordering Scopes** – Order objects by favorite count, latest favorite, or earliest favorite.
- **Filtering Scopes** – Filter objects that a specific user has favorited, that have favorites, or that have no favorites.
- **Adding the `is_favorited_by_user` Column** – Appears in query results indicating whether the current user has favorited the object, without needing to load the relationship.
- **Consistent Interface** – Functions similar to FollowableModel, ReviewRateable, VisitModel behaviors for a unified development experience.
- **Performance Improvement** – Use of efficient subqueries and avoiding full relationship loading where possible.
- **Backward Compatibility** – Retention of old scope names (`sortByCountFavoritesOld`, `withSortByCountFavorites`, `whereHasFavorite`, etc.) to ensure existing code is not broken, with additional enhancements for controlling behavior when no user is present.

---

## 2. Usage Requirements

1. Existence of the `Nano\Markable\Models\Favorite` model (representing the favorite record) with the `nano_markable_favorites` table.
2. The model to be made favoritable must be an Eloquent model (or `October\Rain\Database\Model` in the case of OctoberCMS).
3. Users (e.g., `User` model) must be able to use the favorites system (i.e., they implement the `Authenticatable` interface).
4. Optional: If you wish to support multiple favorite types, set the allowed values via settings (`nano.markable::allowed_values.favorite`).

---

## 3. Adding the Behavior to a Model

The behavior can be added to any model using the `$implement` property (in the NanoSoft context). The behavior includes a Trait containing the scopes and helper functions, so after adding it, the model can use all features.

### Example: Making the `Product` Model Favoritable

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano\Markable\Behaviors\FavoriteableModel;

class Product extends Model
{
    public $implement = [
        FavoriteableModel::class
    ];
}
```

**Note:** If the model uses the `ExtensionBase` system (as is the case in NanoSoft applications), adding it is done via `$implement` as shown.

---

## 4. Core Structure

After adding the behavior, the model has the following relationship:

```php
$product->favorites() // Returns a morphMany relationship linking Favorite records to this model
```

Dynamic computed properties are also available (added automatically):

- `$product->favorites_count` – Number of favorites for this object.
- `$product->is_favorited()` – Whether the current user has favorited this object.
- `$product->getFavorited()` – Returns the favorite record for the current user (if exists).

---

## 5. Available Functions and Scopes

### 5.1 Subquery Building Functions (Internal)

#### `protected function buildFavoriteCountSubQuery($onlyActive = null, $withTrashed = false, $value = null)`
- **Description**: Builds a subquery to count the number of favorite records (COUNT).
- **Parameters**:
  - `$value` (string|null): Filter by favorite type (e.g., `favorite` or `like`).

#### `protected function buildFavoriteLatestDateSubQuery($field = 'created_at', $onlyActive = null, $withTrashed = false, $value = null)`
- **Description**: Builds a subquery for the latest favorite date (MAX).

#### `protected function buildFavoriteEarliestDateSubQuery(...)` – Uses `MIN(field)` to get the earliest date.

### 5.2 Favorite Count Scopes (COUNT)

#### `scopeAddCountFavorites(Builder $query, $orderDirection = 'DESC', $columnName = 'favorites_count', $value = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Adds a column containing the number of favorite records to the query result (without ordering).
- **Example**:
  ```php
  $products = Product::addCountFavorites()->get();
  foreach ($products as $product) {
      echo $product->favorites_count;
  }
  ```

#### `scopeSortByCountFavorites(...)` – Orders results by favorite count (without adding the column).

#### `scopeWithCountFavorites(...)` – Combines `addCountFavorites` and `sortByCountFavorites` (adds column and ordering together).

**Example**:
```php
$products = Product::withCountFavorites('DESC', 'favorites_count', 'like')->get(); // only favorites of type 'like'
```

### 5.3 Favorite Date Scopes

#### `scopeAddLatestFavorite(...)` – Adds a column with the latest favorite date (optionally specifying the field via `$field`).
#### `scopeSortByLatestFavorite(...)` – Orders by the latest favorite date.
#### `scopeWithLatestFavorite(...)` – Adds the column and ordering together.

**Example**:
```php
$products = Product::withLatestFavorite('DESC', 'last_favorite', 'favorite')->get();
```

#### `scopeAddEarliestFavorite(...)` – Adds a column with the earliest favorite date.
#### `scopeSortByEarliestFavorite(...)` – Orders by the earliest favorite date.
#### `scopeWithEarliestFavorite(...)` – Adds the column and ordering together.

### 5.4 Filtering Scopes by User

#### `scopeFavoritedByUser(Builder $query, $user = null, $value = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Filters objects that a specific user has favorited (if `$user` is not passed, the current user is used).
- **Example**:
  ```php
  // Products that the current user has favorited
  $products = Product::favoritedByUser()->get();
  
  // Products that a specific user has favorited as type 'like'
  $products = Product::favoritedByUser($user, 'like')->get();
  ```

#### `scopeNotFavoritedByUser(...)` – Filters objects that a specific user has **not** favorited.

### 5.5 Filtering Scopes by Existence of Favorites

#### `scopeHasFavorites(Builder $query, $minCount = 1, $value = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Filters objects that have at least `$minCount` favorites (with filtering by `value`).
- **Example**:
  ```php
  // Products that have at least 5 favorites of type 'favorite'
  $products = Product::hasFavorites(5, 'favorite')->get();
  ```

#### `scopeHasNoFavorites(...)` – Filters objects that have no favorites.

### 5.6 Scope for Adding Favorite Status Column for Current User

#### `scopeWithIsFavoritedByUser(Builder $query, $user = null, $columnName = 'is_favorited_by_user', $value = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Adds a boolean column (`1` or `0`) to the query result indicating whether the specified user has favorited the object.
- **Example**:
  ```php
  $products = Product::withIsFavoritedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_favorited_by_user) {
          echo "This product is in your favorites";
      }
  }
  ```

### 5.7 Special Scopes

#### `scopeTopFavorited(Builder $query, $limit = 10, $orderDirection = 'DESC', $value = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Retrieves the objects with the most favorites (by favorite count), with a limit.
- **Example**:
  ```php
  $topProducts = Product::topFavorited(10, 'DESC', 'like')->get();
  ```

### 5.8 Helper Statistical Functions

#### `getTotalFavorites($value = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Total number of favorites (sum of `COUNT`) according to filtering options.
- **Output**: Integer.

#### `getFavoritesCountByType($value = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Distribution of favorites by user type (`user_type`).
- **Output**: `Collection` where the key is `user_type` and the value is the count.

#### `getFavoritersUsers($value = null, $onlyActive = null, $withTrashed = false)`
- **Description**: List of users who have favorited the object (loads the `user` relationship).
- **Output**: `Collection` of user objects.

---

## 6. Comprehensive Practical Examples

### Example 1: Adding an Item to Favorites and Checking

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Favorite;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add product to favorites (default type 'favorite')
$favorite = Favorite::add($product, $user);

// Add product as a 'like' favorite
$like = Favorite::add($product, $user, 'like');

// Check if the user has favorited the product
if ($product->isFavorited()) {
    echo "The product is in your favorites";
}
```

### Example 2: Removing an Item from Favorites

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Remove product from favorites (default type)
Favorite::remove($product, $user);

// Remove product from favorites of type 'like'
Favorite::remove($product, $user, 'like');
```

### Example 3: Ordering Products by Favorite Count

```php
$topProducts = Product::topFavorited(10)->get();
```

### Example 4: Adding the `is_favorited_by_user` Column in Product List

```php
$products = Product::withIsFavoritedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_favorited_by_user ? 'Favorite' : 'Not favorite');
}
```

### Example 5: Filtering Products that the Current User Has Favorited

```php
$myFavorites = Product::favoritedByUser()->get();
```

### Example 6: Statistics

```php
$product = Product::find(1);
echo "Number of favorites: " . $product->getTotalFavorites();
print_r($product->getFavoritesCountByType()->toArray());
$users = $product->getFavoritersUsers();
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### Example 7: Combining Advanced Scopes

```php
// Products that have more than 5 favorites of type 'like', ordered by latest favorite
$products = Product::hasFavorites(5, 'like')
    ->sortByLatestFavorite('DESC', 'latest_favorite_at')
    ->get();
```

---

## 7. Backward Compatibility Functions

To maintain existing code that uses previous scope names, wrapper functions are provided in the main class. These functions are marked `@deprecated` and it is recommended to use the new alternatives.

**List of Supported Old Functions:**

| Old Function | New Function |
|--------------|--------------|
| `scopeSortByCountFavoritesOld` | `scopeSortByCountFavorites` |
| `scopeWithSortByCountFavorites` | `scopeWithCountFavorites` |
| `scopeAddSortByCountFavorites` | `scopeAddCountFavorites` |
| `scopeSortByCreatedAtFavorites` | `scopeSortByLatestFavorite` |
| `scopeAddSortByCreatedAtFavorites` | `scopeAddLatestFavorite` |
| `scopeWithSortByCreatedAtFavorites` | `scopeWithLatestFavorite` |
| `scopeWhereHasFavorite` | `scopeFavoritedByUser` (with improvements) |
| `scopeWhereHasFavorites` | `scopeFavoritedByUser` |
| `scopeWhereHasUserFavorite` | `scopeFavoritedByUser` |
| `scopeWhereHasUserFavorites` | `scopeFavoritedByUser` |

**Note regarding the old `whereHasFavorite` functions and their counterparts:** They have been improved to support the `$isForceUser` parameter for controlling query behavior when no user is present, with the default value read from settings (`nano.markable::favoriteable.scope.where_has_favorite.is_force_user`). The default behavior (`true`) adds an impossible condition to return no results (matching the original behavior).

---

## 8. Performance Optimization and Caching

The behavior relies on optimized subqueries to avoid loading relationships entirely. There is no built-in caching system by default, but you can rely on the caching mechanism of the models themselves or use application-level caching if you need to repeat heavy queries.

- When using `withCountFavorites` or `sortByCountFavorites`, only one subquery is executed.
- Using `withIsFavoritedByUser` adds a subquery per row, so it is recommended to use it cautiously on large lists, but it is usually acceptable due to the efficiency of SQL queries.

**Note regarding `withTrashed`:** If you add `SoftDeletes` to the `Favorite` model in the future, the `$withTrashed` option in scopes can be enabled to include soft-deleted records.

---

## 9. Conclusion

`FavoriteableModel` is a comprehensive and powerful solution for managing the favorites system in NanoSoft applications. Thanks to the variety of scopes and helper functions, developers can easily build sophisticated favorites systems while maintaining high flexibility and scalability. Backward compatibility ensures a smooth transition from previous versions. We hope this documentation serves as a comprehensive reference for all the behavior's capabilities.
