# Documentation for `BookmarkableModel` Behavior

---

## 1. Introduction

`BookmarkableModel` is an advanced behavior that adds a polymorphic bookmark system to any model in NanoSoft applications. This behavior allows users to save various objects (e.g., products, articles, pages) in custom lists via the `value` field (e.g., `favorite`, `read_later`, `wishlist`). The behavior comes equipped with a rich set of scopes and helper functions that facilitate building sophisticated bookmark systems with flexibility and high performance.

### Key Benefits

- **Polymorphic Bookmark System** – Any model can be made bookmarkable.
- **Support for Multiple Types** – Ability to categorize bookmarks via the `value` field (e.g., `favorite`, `read_later`) with allowed values configurable from settings.
- **Advanced Ordering Scopes** – Order objects by bookmark count, latest bookmark, or earliest bookmark.
- **Filtering Scopes** – Filter objects that a specific user has bookmarked, that have bookmarks, or that have no bookmarks.
- **Adding the `is_bookmarked_by_user` Column** – Appears in query results indicating whether the current user has bookmarked the object, without needing to load the relationship.
- **Consistent Interface** – Functions similar to FavoriteableModel, LikeableModel, ReactionableModel behaviors for a unified development experience.
- **Performance Improvement** – Use of efficient subqueries and avoiding full relationship loading where possible.
- **Backward Compatibility** – Retention of old scope names (`sortByCountBookmarksOld`, `withSortByCountBookmarks`, `whereHasBookmark`, etc.) to ensure existing code is not broken, with additional enhancements for controlling behavior when no user is present.

---

## 2. Usage Requirements

1. Existence of the `Nano\Markable\Models\Bookmark` model (representing the bookmark record) with the `nano_markable_bookmarks` table.
2. The model to be made bookmarkable must be an Eloquent model (or `October\Rain\Database\Model` in the case of OctoberCMS).
3. Users (e.g., `User` model) must be able to use the bookmark system (i.e., they implement the `Authenticatable` interface).
4. Optional: If you wish to support multiple bookmark types, set the allowed values via settings (`nano.markable::allowed_values.bookmark`).

---

## 3. Adding the Behavior to a Model

The behavior can be added to any model using the `$implement` property (in the NanoSoft context). The behavior includes a Trait containing the scopes and helper functions, so after adding it, the model can use all features.

### Example: Making the `Product` Model Bookmarkable

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano\Markable\Behaviors\BookmarkableModel;

class Product extends Model
{
    public $implement = [
        BookmarkableModel::class
    ];
}
```

**Note:** If the model uses the `ExtensionBase` system (as is the case in NanoSoft applications), adding it is done via `$implement` as shown.

---

## 4. Core Structure

After adding the behavior, the model has the following relationship:

```php
$product->bookmarks() // All bookmark records (all types)
```

Dynamic computed properties are also available (added automatically):

- `$product->bookmarks_count` – Number of bookmarks (can pass `value` as a parameter).
- `$product->is_bookmarked()` – Whether the current user has bookmarked the object (can pass `value`).
- `$product->getBookmarked()` – Returns the bookmark record for the current user (if exists).

---

## 5. Available Functions and Scopes

### 5.1 Subquery Building Functions (Internal)

#### `protected function buildBookmarkCountSubQuery($value = null)`
- **Description**: Builds a subquery to count the number of bookmark records (COUNT).
- **Parameters**:
  - `$value` (string|null): Filter by bookmark type (e.g., `favorite`, `read_later`).

#### `protected function buildBookmarkLatestDateSubQuery($field = 'created_at', $value = null)`
- **Description**: Builds a subquery for the latest bookmark date (MAX).

#### `protected function buildBookmarkEarliestDateSubQuery(...)` – Uses `MIN(field)` to get the earliest date.

### 5.2 Bookmark Count Scopes (COUNT)

#### `scopeAddCountBookmarks(Builder $query, $orderDirection = 'DESC', $columnName = 'bookmarks_count', $value = null)`
- **Description**: Adds a column containing the number of bookmark records to the query result (without ordering).
- **Example**:
  ```php
  $products = Product::addCountBookmarks()->get();
  foreach ($products as $product) {
      echo $product->bookmarks_count;
  }
  ```

#### `scopeSortByCountBookmarks(...)` – Orders results by bookmark count (without adding the column).

#### `scopeWithCountBookmarks(...)` – Combines `addCountBookmarks` and `sortByCountBookmarks` (adds column and ordering together).

**Example**:
```php
$products = Product::withCountBookmarks('DESC', 'bookmarks_count', 'favorite')->get(); // bookmarks of type 'favorite' only
```

### 5.3 Bookmark Date Scopes

#### `scopeAddLatestBookmark(...)` – Adds a column with the latest bookmark date (optionally specifying the field via `$field`).
#### `scopeSortByLatestBookmark(...)` – Orders by the latest bookmark date.
#### `scopeWithLatestBookmark(...)` – Adds the column and ordering together.

**Example**:
```php
$products = Product::withLatestBookmark('DESC', 'last_bookmark', 'favorite')->get();
```

#### `scopeAddEarliestBookmark(...)` – Adds a column with the earliest bookmark date.
#### `scopeSortByEarliestBookmark(...)` – Orders by the earliest bookmark date.
#### `scopeWithEarliestBookmark(...)` – Adds the column and ordering together.

### 5.4 Filtering Scopes by User

#### `scopeBookmarkedByUser(Builder $query, $user = null, $value = null)`
- **Description**: Filters objects that a specific user has bookmarked (if `$user` is not passed, the current user is used).
- **Example**:
  ```php
  // Products that the current user has bookmarked (any type)
  $products = Product::bookmarkedByUser()->get();
  
  // Products that a specific user has bookmarked as type 'read_later'
  $products = Product::bookmarkedByUser($user, 'read_later')->get();
  ```

#### `scopeNotBookmarkedByUser(...)` – Filters objects that a specific user has **not** bookmarked.

### 5.5 Filtering Scopes by Existence of Bookmarks

#### `scopeHasBookmarks(Builder $query, $minCount = 1, $value = null)`
- **Description**: Filters objects that have at least `$minCount` bookmarks (with filtering by `value`).
- **Example**:
  ```php
  // Products that have at least 5 bookmarks of type 'favorite'
  $products = Product::hasBookmarks(5, 'favorite')->get();
  ```

#### `scopeHasNoBookmarks(...)` – Filters objects that have no bookmarks.

### 5.6 Scope for Adding Bookmark Status Column for Current User

#### `scopeWithIsBookmarkedByUser(Builder $query, $user = null, $columnName = 'is_bookmarked_by_user', $value = null)`
- **Description**: Adds a boolean column (`1` or `0`) to the query result indicating whether the specified user has bookmarked the object.
- **Example**:
  ```php
  $products = Product::withIsBookmarkedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_bookmarked_by_user) {
          echo "This product is in your bookmarks";
      }
  }
  ```

### 5.7 Special Scopes

#### `scopeTopBookmarked(Builder $query, $limit = 10, $orderDirection = 'DESC', $value = null)`
- **Description**: Retrieves the objects with the most bookmarks (by bookmark count), with a limit.
- **Example**:
  ```php
  $topProducts = Product::topBookmarked(10, 'DESC', 'favorite')->get();
  ```

### 5.8 Helper Statistical Functions

#### `getTotalBookmarks($value = null)`
- **Description**: Total number of bookmarks (sum of `COUNT`) according to filtering options.
- **Output**: Integer.

#### `getBookmarksCountByType($value = null)`
- **Description**: Distribution of bookmarks by user type (`user_type`).
- **Output**: `Collection` where the key is `user_type` and the value is the count.

#### `getBookmarkersUsers($value = null)`
- **Description**: List of users who have bookmarked the object (loads the `user` relationship).
- **Output**: `Collection` of user objects.

---

## 6. Comprehensive Practical Examples

### Example 1: Adding a Bookmark and Checking

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Bookmark;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add a bookmark of type 'favorite'
$bookmark = Bookmark::add($product, $user, 'favorite');

// Add a bookmark of type 'read_later' with metadata
$bookmark = Bookmark::add($product, $user, 'read_later', ['notes' => 'Read this later']);

// Check if the user has bookmarked the product (any type)
if ($product->isBookmarked()) {
    echo "The product is in your bookmarks";
}

// Check for a specific type
if ($product->isBookmarked(null, 'read_later')) {
    echo "The product is in your read-later list";
}
```

### Example 2: Removing a Bookmark

```php
// Remove the bookmark (any type)
Bookmark::remove($product, $user);

// Remove a bookmark of type 'favorite' only
Bookmark::remove($product, $user, 'favorite');
```

### Example 3: Ordering Products by Bookmark Count

```php
$topProducts = Product::topBookmarked(10)->get();
```

### Example 4: Adding the `is_bookmarked_by_user` Column in Product List

```php
$products = Product::withIsBookmarkedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_bookmarked_by_user ? 'Favorite' : 'Not favorite');
}
```

### Example 5: Filtering Products that the Current User Has Bookmarked

```php
$myBookmarks = Product::bookmarkedByUser()->get();
```

### Example 6: Statistics

```php
$product = Product::find(1);
echo "Number of bookmarks of type 'favorite': " . $product->getTotalBookmarks('favorite');
print_r($product->getBookmarksCountByType('favorite')->toArray());
$users = $product->getBookmarkersUsers('favorite');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### Example 7: Combining Advanced Scopes

```php
// Products that have more than 5 bookmarks of type 'read_later', ordered by latest bookmark
$products = Product::hasBookmarks(5, 'read_later')
    ->sortByLatestBookmark('DESC', 'latest_read_later_at')
    ->get();
```

---

## 7. Backward Compatibility Functions

To maintain existing code that uses previous scope names, wrapper functions are provided in the main class. These functions are marked `@deprecated` and it is recommended to use the new alternatives.

**List of Supported Old Functions:**

| Old Function | New Function |
|--------------|--------------|
| `scopeSortByCountBookmarksOld` | `scopeSortByCountBookmarks` |
| `scopeWithSortByCountBookmarks` | `scopeWithCountBookmarks` |
| `scopeAddSortByCountBookmarks` | `scopeAddCountBookmarks` |
| `scopeSortByCreatedAtBookmarks` | `scopeSortByLatestBookmark` |
| `scopeAddSortByCreatedAtBookmarks` | `scopeAddLatestBookmark` |
| `scopeWithSortByCreatedAtBookmarks` | `scopeWithLatestBookmark` |
| `scopeWhereHasBookmark` | `scopeBookmarkedByUser` (with improvements) |
| `scopeWhereHasBookmarks` | `scopeBookmarkedByUser` |
| `scopeWhereHasUserBookmark` | `scopeBookmarkedByUser` |
| `scopeWhereHasUserBookmarks` | `scopeBookmarkedByUser` |

**Note regarding the old `whereHasBookmark` functions and their counterparts:** They have been improved to support the `$isForceUser` parameter for controlling query behavior when no user is present, with the default value read from settings (`nano.markable::bookmarkable.scope.where_has_bookmark.is_force_user`). The default behavior (`true`) adds an impossible condition to return no results (matching the original behavior).

---

## 8. Performance Optimization and Caching

The behavior relies on optimized subqueries to avoid loading relationships entirely. There is no built-in caching system by default, but you can rely on the caching mechanism of the models themselves or use application-level caching if you need to repeat heavy queries.

- When using `withCountBookmarks` or `sortByCountBookmarks`, only one subquery is executed.
- Using `withIsBookmarkedByUser` adds a subquery per row, so it is recommended to use it cautiously on large lists, but it is usually acceptable due to the efficiency of SQL queries.

**Note:** If you add `SoftDeletes` to the `Bookmark` model in the future, the scopes can be easily modified to support `withTrashed`.

---

## 9. Conclusion

`BookmarkableModel` is a comprehensive and powerful solution for managing the bookmark system in NanoSoft applications. Thanks to the variety of scopes and helper functions, developers can easily build sophisticated bookmark systems while maintaining high flexibility and scalability. Backward compatibility ensures a smooth transition from previous versions. We hope this documentation serves as a comprehensive reference for all the behavior's capabilities.
