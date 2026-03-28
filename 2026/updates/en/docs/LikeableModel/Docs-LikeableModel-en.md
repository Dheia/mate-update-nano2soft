# Documentation for `LikeableModel` Behavior

---

## 1. Introduction

`LikeableModel` is an advanced behavior that adds a polymorphic likes system to any model in NanoSoft applications. This behavior allows users to like or dislike various objects (e.g., products, articles, comments) with support for distinguishing like types via the `value` field. The behavior comes equipped with a rich set of scopes and helper functions that facilitate building sophisticated interaction systems with flexibility and high performance.

### Key Benefits

- **Polymorphic Likes System** – Any model can be made likeable.
- **Support for Multiple Types** – Ability to categorize likes via the `value` field (e.g., `like`, `dislike`) with default values configurable from settings.
- **Advanced Ordering Scopes** – Order objects by like count, latest like, or earliest like.
- **Filtering Scopes** – Filter objects that a specific user has liked, that have likes, or that have no likes.
- **Adding the `is_liked_by_user` Column** – Appears in query results indicating whether the current user has liked the object, without needing to load the relationship.
- **Consistent Interface** – Functions similar to FavoriteableModel, ReactionableModel behaviors for a unified development experience.
- **Performance Improvement** – Use of efficient subqueries and avoiding full relationship loading where possible.
- **Backward Compatibility** – Retention of old scope names (`sortByCountLikesOld`, `withSortByCountLikes`, `whereHasLike`, etc.) to ensure existing code is not broken, with additional enhancements for controlling behavior when no user is present.

---

## 2. Usage Requirements

1. Existence of the `Nano\Markable\Models\Like` model (representing the like record) with the `nano_markable_likes` table.
2. The model to be made likeable must be an Eloquent model (or `October\Rain\Database\Model` in the case of OctoberCMS).
3. Users (e.g., `User` model) must be able to use the likes system (i.e., they implement the `Authenticatable` interface).
4. Optional: If you wish to support multiple like types (`like` / `dislike`), set the allowed values via settings (`nano.markable::allowed_values.like`).

---

## 3. Adding the Behavior to a Model

The behavior can be added to any model using the `$implement` property (in the NanoSoft context). The behavior includes a Trait containing the scopes and helper functions, so after adding it, the model can use all features.

### Example: Making the `Product` Model Likeable

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano\Markable\Behaviors\LikeableModel;

class Product extends Model
{
    public $implement = [
        LikeableModel::class
    ];
}
```

**Note:** If the model uses the `ExtensionBase` system (as is the case in NanoSoft applications), adding it is done via `$implement` as shown.

---

## 4. Core Structure

After adding the behavior, the model has the following relationships:

```php
$product->likes()      // All like records (all types)
$product->my_likes()   // Likes of type 'like' only
$product->my_dislikes() // Likes of type 'dislike' only
```

Dynamic computed properties are also available (added automatically):

- `$product->likes_count` – Number of likes (can pass `value` as a parameter).
- `$product->is_liked()` – Whether the current user has liked the object (default value `like`).
- `$product->getLiked()` – Returns the like record for the current user (if exists).

---

## 5. Available Functions and Scopes

### 5.1 Subquery Building Functions (Internal)

#### `protected function buildLikeCountSubQuery($value = null)`
- **Description**: Builds a subquery to count the number of like records (COUNT).
- **Parameters**:
  - `$value` (string|null): Filter by like type (e.g., `like` or `dislike`).

#### `protected function buildLikeLatestDateSubQuery($field = 'created_at', $value = null)`
- **Description**: Builds a subquery for the latest like date (MAX).

#### `protected function buildLikeEarliestDateSubQuery(...)` – Uses `MIN(field)` to get the earliest date.

### 5.2 Like Count Scopes (COUNT)

#### `scopeAddCountLikes(Builder $query, $orderDirection = 'DESC', $columnName = 'likes_count', $value = null)`
- **Description**: Adds a column containing the number of like records to the query result (without ordering).
- **Example**:
  ```php
  $products = Product::addCountLikes()->get();
  foreach ($products as $product) {
      echo $product->likes_count;
  }
  ```

#### `scopeSortByCountLikes(...)` – Orders results by like count (without adding the column). If `$value` is not passed, the default value from `Like::getDefaultValue()` is used.

#### `scopeWithCountLikes(...)` – Combines `addCountLikes` and `sortByCountLikes` (adds column and ordering together).

**Example**:
```php
$products = Product::withCountLikes('DESC', 'likes_count', 'like')->get(); // likes of type 'like' only
```

### 5.3 Like Date Scopes

#### `scopeAddLatestLike(...)` – Adds a column with the latest like date (optionally specifying the field via `$field`).
#### `scopeSortByLatestLike(...)` – Orders by the latest like date.
#### `scopeWithLatestLike(...)` – Adds the column and ordering together.

**Example**:
```php
$products = Product::withLatestLike('DESC', 'last_like', 'like')->get();
```

#### `scopeAddEarliestLike(...)` – Adds a column with the earliest like date.
#### `scopeSortByEarliestLike(...)` – Orders by the earliest like date.
#### `scopeWithEarliestLike(...)` – Adds the column and ordering together.

### 5.4 Filtering Scopes by User

#### `scopeLikedByUser(Builder $query, $user = null, $value = null)`
- **Description**: Filters objects that a specific user has liked (if `$user` is not passed, the current user is used).
- **Example**:
  ```php
  // Products that the current user has liked (likes of type 'like' by default)
  $products = Product::likedByUser()->get();
  
  // Products that a specific user has liked as type 'like'
  $products = Product::likedByUser($user, 'like')->get();
  ```

#### `scopeNotLikedByUser(...)` – Filters objects that a specific user has **not** liked.

### 5.5 Filtering Scopes by Existence of Likes

#### `scopeHasLikes(Builder $query, $minCount = 1, $value = null)`
- **Description**: Filters objects that have at least `$minCount` likes (with filtering by `value`).
- **Example**:
  ```php
  // Products that have at least 5 likes of type 'like'
  $products = Product::hasLikes(5, 'like')->get();
  ```

#### `scopeHasNoLikes(...)` – Filters objects that have no likes.

### 5.6 Scope for Adding Like Status Column for Current User

#### `scopeWithIsLikedByUser(Builder $query, $user = null, $columnName = 'is_liked_by_user', $value = null)`
- **Description**: Adds a boolean column (`1` or `0`) to the query result indicating whether the specified user has liked the object.
- **Example**:
  ```php
  $products = Product::withIsLikedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_liked_by_user) {
          echo "You liked this product";
      }
  }
  ```

### 5.7 Special Scopes

#### `scopeTopLiked(Builder $query, $limit = 10, $orderDirection = 'DESC', $value = null)`
- **Description**: Retrieves the objects with the most likes (by like count), with a limit.
- **Example**:
  ```php
  $topProducts = Product::topLiked(10, 'DESC', 'like')->get();
  ```

### 5.8 Helper Statistical Functions

#### `getTotalLikes($value = null)`
- **Description**: Total number of likes (sum of `COUNT`) according to filtering options.
- **Output**: Integer.

#### `getLikesCountByType($value = null)`
- **Description**: Distribution of likes by user type (`user_type`).
- **Output**: `Collection` where the key is `user_type` and the value is the count.

#### `getLikersUsers($value = null)`
- **Description**: List of users who liked the object (loads the `user` relationship).
- **Output**: `Collection` of user objects.

---

## 6. Comprehensive Practical Examples

### Example 1: Adding a Like and Checking

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Like;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add a like (default type 'like')
$like = Like::add($product, $user);

// Add a dislike (type 'dislike')
$dislike = Like::add($product, $user, 'dislike');

// Check if the user has liked the product
if ($product->isLiked()) {
    echo "You liked the product";
}

// Get like count
echo $product->likes_count; // all likes
echo $product->likes_count('like'); // likes of type 'like' only
```

### Example 2: Removing a Like

```php
// Remove the like (default type)
Like::remove($product, $user);

// Remove only the dislike
Like::remove($product, $user, 'dislike');
```

### Example 3: Ordering Products by Like Count

```php
$topProducts = Product::topLiked(10)->get();
```

### Example 4: Adding the `is_liked_by_user` Column in Product List

```php
$products = Product::withIsLikedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_liked_by_user ? 'Liked' : 'Not liked');
}
```

### Example 5: Filtering Products that the Current User Has Liked

```php
$myLikedProducts = Product::likedByUser()->get();
```

### Example 6: Statistics

```php
$product = Product::find(1);
echo "Number of likes: " . $product->getTotalLikes('like');
echo "Number of dislikes: " . $product->getTotalLikes('dislike');
print_r($product->getLikesCountByType('like')->toArray());
$users = $product->getLikersUsers('like');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### Example 7: Combining Advanced Scopes

```php
// Products that have more than 5 likes of type 'like', ordered by latest like
$products = Product::hasLikes(5, 'like')
    ->sortByLatestLike('DESC', 'latest_like_at')
    ->get();
```

---

## 7. Backward Compatibility Functions

To maintain existing code that uses previous scope names, wrapper functions are provided in the main class. These functions are marked `@deprecated` and it is recommended to use the new alternatives.

**List of Supported Old Functions:**

| Old Function | New Function |
|--------------|--------------|
| `scopeSortByCountLikesOld` | `scopeSortByCountLikes` |
| `scopeWithSortByCountLikes` | `scopeWithCountLikes` |
| `scopeSortByCreatedAtLikes` | `scopeSortByLatestLike` |
| `scopeAddSortByCreatedAtLikes` | `scopeAddLatestLike` |
| `scopeWithSortByCreatedAtLikes` | `scopeWithLatestLike` |
| `scopeWhereHasLike` | `scopeLikedByUser` (with improvements) |
| `scopeWhereHasLikes` | `scopeLikedByUser` |
| `scopeWhereHasUserLike` | `scopeLikedByUser` |
| `scopeWhereHasUserLikes` | `scopeLikedByUser` |

**Note regarding the old `whereHasLike` functions and their counterparts:** They have been improved to support the `$isForceUser` parameter for controlling query behavior when no user is present, with the default value read from settings (`nano.markable::likeable.scope.where_has_like.is_force_user`). The default behavior (`true`) adds an impossible condition to return no results (matching the original behavior).

---

## 8. Performance Optimization and Caching

The behavior relies on optimized subqueries to avoid loading relationships entirely. There is no built-in caching system by default, but you can rely on the caching mechanism of the models themselves or use application-level caching if you need to repeat heavy queries.

- When using `withCountLikes` or `sortByCountLikes`, only one subquery is executed.
- Using `withIsLikedByUser` adds a subquery per row, so it is recommended to use it cautiously on large lists, but it is usually acceptable due to the efficiency of SQL queries.

**Note:** If you add `SoftDeletes` to the `Like` model in the future, the scopes can be easily modified to support `withTrashed`.

---

## 9. Conclusion

`LikeableModel` is a comprehensive and powerful solution for managing the likes system in NanoSoft applications. Thanks to the variety of scopes and helper functions, developers can easily build sophisticated interaction systems while maintaining high flexibility and scalability. Backward compatibility ensures a smooth transition from previous versions. We hope this documentation serves as a comprehensive reference for all the behavior's capabilities.
