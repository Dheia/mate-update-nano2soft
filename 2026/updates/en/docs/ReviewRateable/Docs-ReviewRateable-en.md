# Documentation for `ReviewRateable` Behavior

---

## 1. Introduction

`ReviewRateable` is an advanced behavior that adds a polymorphic rating system to any model in NanoSoft applications. This behavior allows users to add ratings (stars) with textual reviews to various objects (e.g., products, articles, services) with comprehensive support for approval (`approved`), activity (`is_active`), positivity/negativity (`is_positive`), and soft delete. The behavior comes equipped with a rich set of scopes and helper functions that facilitate building sophisticated rating systems with flexibility and high performance.

### Key Benefits

- **Polymorphic Rating System** – Any model can be made rateable.
- **Flexibility in Rating States** – Support for `rating` (numeric value), `approved` (approval status), `is_active` (activity), `is_positive` (positive/negative) to represent multiple states.
- **Support for Soft-Deleted Reviews** – Ability to include soft-deleted reviews in queries when needed.
- **Advanced Ordering Scopes** – Order objects by number of reviews, average rating, latest review, or earliest review, with options for filtering by approval, activity, and positivity.
- **Filtering Scopes** – Filter objects that a specific user has rated, that have reviews, or that have no reviews.
- **Adding the `is_rated_by_user` Column** – Appears in query results indicating whether the current user has rated the object, without needing to load the relationship.
- **Consistent Interface** – Functions similar to FollowableModel and VisitModel behaviors for a unified development experience.
- **Performance Improvement** – Use of efficient subqueries and avoiding full relationship loading where possible.
- **Backward Compatibility** – Retention of old scope names (`sortByRating`, `withRating`, `sortByCreatedAtReviews`, etc.) to ensure existing code is not broken.

---

## 2. Usage Requirements

1. Existence of the `Nano\Reviews\Models\Review` model (representing the review record) with the `nano_reviews_reviews` table.
2. The model to be made rateable must be an Eloquent model (or `October\Rain\Database\Model` in the case of OctoberCMS).
3. Users (e.g., `User` model) must be able to use the rating system (i.e., they implement the `Authenticatable` interface).
4. Optional: Use `SoftDeletes` in the `Review` model to benefit from the `withTrashed` option.

---

## 3. Adding the Behavior to a Model

The behavior can be added to any model using the `$implement` property (in the NanoSoft context). The behavior includes a Trait containing the scopes and helper functions, so after adding it, the model can use all features.

### Example: Making the `Product` Model Rateable

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano\Reviews\Behaviors\ReviewRateable;

class Product extends Model
{
    public $implement = [
        ReviewRateable::class
    ];
}
```

**Note:** If the model uses the `ExtensionBase` system (as is the case in NanoSoft applications), adding it is done via `$implement` as shown.

---

## 4. Core Structure

After adding the behavior, the model has the following relationship:

```php
$product->ratings() // Returns a morphMany relationship linking Review records to this model
```

Computed properties are also available for easy access to rating statistics:

```php
$product->ratings_count   // Number of review records
$product->avg_rating      // Average rating (when column is added)
$product->is_rated_by_user // Whether the current user has rated
```

---

## 5. Available Functions and Scopes

### 5.1 Subquery Building Functions (Internal)

#### `protected function buildReviewCountSubQuery($onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **Description**: Builds a subquery to count the number of review records (COUNT).
- **Parameters**:
  - `$onlyActive` (bool|null): `true` for active reviews only, `false` for inactive only, `null` for all.
  - `$withTrashed` (bool): Include soft-deleted reviews.
  - `$onlyApproved` (bool|null): `true` for approved reviews only, `false` for unapproved, `null` for all.
  - `$isPositive` (bool|null): `true` for positive reviews only, `false` for negative, `null` for all.
  - `$ratingValue` (int|null): Filter by rating value (e.g., 5).

#### `protected function buildReviewAvgSubQuery(...)` – Same parameters, uses `AVG(rating)` with optional rounding.

#### `protected function buildReviewLatestDateSubQuery($field = 'created_at', ...)` – Uses `MAX(field)` to get the latest date.

#### `protected function buildReviewEarliestDateSubQuery(...)` – Uses `MIN(field)` to get the earliest date.

### 5.2 Number of Reviews Scopes (COUNT)

#### `scopeAddCountReviews(Builder $query, $orderDirection = 'DESC', $columnName = 'reviews_count', $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **Description**: Adds a column containing the number of review records to the query result (without ordering).
- **Example**:
  ```php
  $products = Product::addCountReviews()->get();
  foreach ($products as $product) {
      echo $product->reviews_count;
  }
  ```

#### `scopeSortByCountReviews(...)` – Orders results by number of reviews (without adding the column).

#### `scopeWithCountReviews(...)` – Combines `addCountReviews` and `sortByCountReviews` (adds column and ordering together).

**Example**:
```php
$products = Product::withCountReviews('DESC', 'reviews_count', true)->get(); // active reviews only
```

### 5.3 Average Rating Scopes (AVG)

#### `scopeAddAvgRating(Builder $query, $orderDirection = 'DESC', $columnName = 'avg_rating', $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null, $round = 2)`
- **Description**: Adds a column with the average rating (rounded to a number of decimal places).
- **Example**:
  ```php
  $products = Product::addAvgRating()->get();
  echo $products->first()->avg_rating;
  ```

#### `scopeSortByAvgRating(...)` – Orders by average rating.
#### `scopeWithAvgRating(...)` – Adds the column and ordering together.

### 5.4 Latest Review Date Scopes

#### `scopeAddLatestReview(...)` – Adds a column with the latest review date (optionally specifying the field via `$field`).
#### `scopeSortByLatestReview(...)` – Orders by the latest review date.
#### `scopeWithLatestReview(...)` – Adds the column and ordering together.

**Example**:
```php
$products = Product::withLatestReview('DESC', 'last_review', true, false, true, null, null, 'created_at')->get();
```

### 5.5 Earliest Review Date Scopes

#### `scopeAddEarliestReview(...)` – Adds a column with the earliest review date.
#### `scopeSortByEarliestReview(...)` – Orders by the earliest review date.
#### `scopeWithEarliestReview(...)` – Adds the column and ordering together.

### 5.6 Filtering Scopes by User

#### `scopeRatedByUser(Builder $query, $user = null, $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **Description**: Filters objects that a specific user has rated (if `$user` is not passed, the current user is used).
- **Example**:
  ```php
  // Products that the current user has rated
  $products = Product::ratedByUser()->get();
  
  // Products that a specific user has rated with positive reviews only
  $products = Product::ratedByUser($user, null, false, null, true)->get();
  ```

#### `scopeNotRatedByUser(...)` – Filters objects that a specific user has **not** rated.

### 5.7 Filtering Scopes by Existence of Reviews

#### `scopeHasRatings(Builder $query, $minCount = 1, $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **Description**: Filters objects that have at least `$minCount` reviews (with filtering options).
- **Example**:
  ```php
  // Products that have at least 5 approved reviews
  $products = Product::hasRatings(5, true, false, true)->get();
  ```

#### `scopeHasNoRatings(...)` – Filters objects that have no reviews.

### 5.8 Scope for Adding Rating Status Column for Current User

#### `scopeWithIsRatedByUser(Builder $query, $user = null, $columnName = 'is_rated_by_user', $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **Description**: Adds a boolean column (`1` or `0`) to the query result indicating whether the specified user has rated the object.
- **Example**:
  ```php
  $products = Product::withIsRatedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_rated_by_user) {
          echo "You have rated this product";
      }
  }
  ```

### 5.9 Special Scopes

#### `scopeTopRated(Builder $query, $limit = 10, $orderDirection = 'DESC', $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null, $round = 2)`
- **Description**: Retrieves the highest-rated objects (by average rating) with a limit.
- **Example**:
  ```php
  $topProducts = Product::topRated(10, 'DESC', true, false, true)->get();
  ```

#### `scopeTopRatedByCount(Builder $query, $limit = 10, $orderDirection = 'DESC', $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **Description**: Retrieves the most-reviewed objects (by number of reviews) with a limit.

### 5.10 Helper Statistical Functions

#### `getTotalRatings($onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **Description**: Total number of ratings (sum of `rating`) according to filtering options.
- **Output**: Integer.

#### `getAverageRating($onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null, $round = null)`
- **Description**: Average rating with filtering options.

#### `getRatingsCountByType($onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **Description**: Distribution of ratings by user type (`user_type`).
- **Output**: `Collection` where the key is `user_type` and the value is the count.

#### `getRatersUsers(...)`
- **Description**: List of users who rated the object (loads the `user` relationship).
- **Output**: `Collection` of user objects.

### 5.11 Core Functions in the Behavior (Not in the Trait)

#### `ratingdBy()`
- **Description**: Returns a collection of users who rated the current object.
- **Output**: `Collection` of users (with `user` loaded).

#### `isRating($user = null)`
- **Description**: Checks if the user (or current user) has rated the object.
- **Output**: `bool`.

#### `getRating($user = null)`
- **Description**: Returns the review record for the user (if exists).
- **Output**: `Review` object or `null`.

#### `addRating($data, $user, $parent = null): Review`
- **Description**: Adds a new review for the object. `$data` can be passed as an `array` containing `rating`, `title`, `content`, `approved`, `is_positive`, etc.
- **Output**: The created `Review` object.

#### `toggleRating($data, $user = null)`
- **Description**: If the user has rated, updates the review; otherwise, adds a new review.
- **Output**: `Review` object.

#### `updateRating($id, $data): Review`
- **Description**: Updates data of an existing review.

#### `deleteRating($id): ?bool`
- **Description**: Deletes a specific review (Soft delete).

#### `countRating()`
- **Description**: Number of reviews (optionally only approved).

#### `averageRating($round = null, $onlyApproved = false): Collection`
- **Description**: Average rating (legacy helper function; `getAverageRating` is preferred).

---

## 6. Comprehensive Practical Examples

### Example 1: Adding a Rating and Updating

```php
$user = Auth::getUser();
$product = Product::find(1);

// Add a new rating (5 stars, text review)
$review = $product->addRating([
    'rating' => 5,
    'title' => 'Excellent product',
    'content' => 'High quality and reasonable price',
    'approved' => true,
    'is_positive' => true,
], $user);

// Update the rating
$product->updateRating($review->id, ['rating' => 4, 'content' => 'Update after use']);

// Toggle between rating and cancel (if rated, updates; else adds)
$product->toggleRating(['rating' => 5], $user);
```

### Example 2: Ordering Products by Average Rating

```php
$topProducts = Product::topRated(10, 'DESC', true, false, true)->get(); // approved reviews only

// Add average rating column with ordering
$products = Product::withAvgRating('DESC', 'avg_rating', null, false, true)->get();
```

### Example 3: Adding the `is_rated_by_user` Column in Product List

```php
$products = Product::withIsRatedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_rated_by_user ? 'You rated' : 'You did not rate');
}
```

### Example 4: Filtering Products that the Current User Has Rated

```php
$myRatedProducts = Product::ratedByUser()->get();
```

### Example 5: Filtering by Positive Reviews Only

```php
$products = Product::hasRatings(1, true, false, null, true)->get(); // at least one positive review
```

### Example 6: Rating Statistics for a Specific Product

```php
$product = Product::find(1);

echo "Total ratings: " . $product->getTotalRatings(true, false, true);
echo "Average rating: " . $product->getAverageRating(true, false, true);
echo "Distribution of ratings by user type: ";
print_r($product->getRatingsCountByType(true, false, true)->toArray());

$users = $product->getRatersUsers(true, false, true);
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### Example 7: Combining Advanced Scopes

```php
// Products that have at least 5 approved reviews, ordered by average rating
$products = Product::hasRatings(5, true, false, true)
    ->sortByAvgRating('DESC', 'avg_rating')
    ->get();
```

---

## 7. Backward Compatibility Functions

To maintain existing code that uses previous scope names, wrapper functions are provided in the main class. These functions are marked `@deprecated` and it is recommended to use the new alternatives.

**List of Supported Old Functions:**

| Old Function | New Function |
|--------------|--------------|
| `scopeSortByRating` | `scopeSortByAvgRating` |
| `scopeWithRating` | `scopeWithAvgRating` |
| `scopeAddSortByRating` | `scopeAddAvgRating` |
| `scopeWithSortByCountReviews` | `scopeWithCountReviews` |
| `scopeSortByCountReviewsOld` | `scopeSortByCountReviews` |
| `scopeSortByCreatedAtReviews` | `scopeSortByLatestReview` |
| `scopeAddSortByCreatedAtReviews` | `scopeAddLatestReview` |
| `scopeWithSortByCreatedAtReviews` | `scopeWithLatestReview` |

**Note:** The `scopeSortByCountReviews` scope was retained with the same name but with a new signature supporting additional optional parameters, so old calls are not broken.

---

## 8. Performance Optimization and Caching

The behavior relies on optimized subqueries to avoid loading relationships entirely. There is no built-in caching system by default, but you can rely on the caching mechanism of the models themselves or use application-level caching if you need to repeat heavy queries.

- When using `withCountReviews` or `sortByCountReviews`, only one subquery is executed.
- Using `withIsRatedByUser` adds a subquery per row, so it is recommended to use it cautiously on large lists, but it is usually acceptable due to the efficiency of SQL queries.

**Note regarding `withTrashed`:** When using `withTrashed = true` in any of the scopes or statistical functions, ensure that the `Nano\Reviews\Models\Review` model uses the `SoftDeletes` trait; otherwise, query errors will occur.

---

## 9. Conclusion

`ReviewRateable` is a comprehensive and powerful solution for managing the rating system in NanoSoft applications. Thanks to the variety of scopes and helper functions, developers can easily build advanced rating systems while maintaining high flexibility and scalability. Backward compatibility ensures a smooth transition from previous versions. We hope this documentation serves as a comprehensive reference for all the behavior's capabilities.
