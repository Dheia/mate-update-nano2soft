# Advanced Practical Examples for Using the `ReviewRateable` Behavior

This section provides a set of integrated examples illustrating how to use the `ReviewRateable` behavior in real-world scenarios, from basic operations to complex queries and advanced statistics.

---

### 1. Basic Operations

#### 1.1 Adding a New Review with Custom Data

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Reviews\Models\Review;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add a standard review (5 stars, text review)
$review = $product->addRating([
    'rating' => 5,
    'title' => 'Excellent product',
    'content' => 'High quality and reasonable price',
    'approved' => true,
    'is_positive' => true,
], $user);

// Add an unapproved review (requires moderator approval)
$review = $product->addRating([
    'rating' => 2,
    'title' => 'Not satisfied',
    'content' => 'Product does not match description',
    'approved' => false,
    'is_positive' => false,
], $user);
```

#### 1.2 Updating an Existing Review

```php
$user = AuthHelpers::getCurrentUser();
$review = $product->getRating($user);

if ($review) {
    $product->updateRating($review->id, [
        'rating' => 4,
        'content' => 'After some use, it got better',
    ]);
}
```

#### 1.3 Toggling Between Rating and Canceling (Toggle)

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// If the user has rated, updates the rating (new data can be passed)
// If not rated, adds a new rating
$product->toggleRating(['rating' => 5, 'content' => 'Excellent experience'], $user);
```

#### 1.4 Deleting a Review (Soft Delete)

```php
$user = AuthHelpers::getCurrentUser();
$review = $product->getRating($user);

if ($review) {
    $product->deleteRating($review->id);
}
```

---

### 2. Filtering by Users

#### 2.1 Retrieving All Products that the Current User Has Rated

```php
$myRatedProducts = Product::ratedByUser()->get();

// Including only inactive reviews
$myRatedProducts = Product::ratedByUser(null, false)->get();

// Including only approved reviews
$myRatedProducts = Product::ratedByUser(null, null, false, true)->get();
```

#### 2.2 Retrieving Products that the Current User Has Not Rated (for Suggestions)

```php
$unratedProducts = Product::notRatedByUser()->paginate(20);
```

#### 2.3 Retrieving Products that a Specific User Has Rated with Positive Reviews

```php
$user = User::find(2);
$products = Product::ratedByUser($user, null, false, null, true)->get();
```

---

### 3. Advanced Ordering and Ranking

#### 3.1 Ordering Products by Number of Reviews (Most Reviewed)

```php
// Using the direct scope
$topProducts = Product::sortByCountReviews('DESC', 'reviews_count', true)->take(10)->get();

// Using the shortcut topRatedByCount
$topProducts = Product::topRatedByCount(10)->get();

// Ordering by number of approved reviews only
$topProducts = Product::sortByCountReviews('DESC', 'reviews_count', true, false, true)->get();
```

#### 3.2 Ordering Products by Average Rating (Highest Rated)

```php
// Using the direct scope
$topProducts = Product::sortByAvgRating('DESC', 'avg_rating', true, false, true)->get();

// Using the shortcut topRated
$topProducts = Product::topRated(10)->get();

// Ordering by average rating of positive reviews only
$topProducts = Product::sortByAvgRating('DESC', 'avg_rating', null, false, null, true)->get();
```

#### 3.3 Adding a Number of Reviews Column to Results (Without Ordering)

```php
$products = Product::addCountReviews()->get();
foreach ($products as $product) {
    echo $product->name . ' - Number of Reviews: ' . $product->reviews_count;
}
```

#### 3.4 Adding an Average Rating Column with Ordering

```php
$products = Product::withAvgRating('DESC', 'avg_rating', true, false, true)->get();
foreach ($products as $product) {
    echo $product->name . ' - Average Rating: ' . $product->avg_rating;
}
```

#### 3.5 Ordering by Latest Review (Last User Who Reviewed)

```php
$products = Product::sortByLatestReview('DESC')->get();
```

#### 3.6 Ordering by Earliest Review (First Who Reviewed)

```php
$products = Product::sortByEarliestReview('ASC')->get();
```

---

### 4. Statistics and Reports

#### 4.1 Total Ratings for a Specific Product (Sum of Rating Values)

```php
$product = Product::find(1);

// All ratings
$total = $product->getTotalRatings();

// Approved ratings only
$totalApproved = $product->getTotalRatings(null, false, true);

// Approved positive ratings only
$totalPositive = $product->getTotalRatings(null, false, true, true);
```

#### 4.2 Average Rating for a Specific Product

```php
$product = Product::find(1);

// Average of all ratings
$avg = $product->getAverageRating();

// Average of approved ratings only (rounded to two decimals)
$avgApproved = $product->getAverageRating(null, false, true, null, null, 2);
```

#### 4.3 Distribution of Ratings by User Type

```php
$counts = $product->getRatingsCountByType(true, false, true);
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]
```

#### 4.4 Retrieving a List of User Names Who Rated a Specific Product

```php
$users = $product->getRatersUsers(true, false, true);
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.5 Rating Statistics within a Time Range (Using Scopes with where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// Products that were rated in the last week
$products = Product::whereHas('ratings', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. Combining Scopes for Complex Queries

#### 5.1 Highest Rated Products (Average Rating) that the Current User Has Also Rated

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topRated(20)
    ->ratedByUser($user)
    ->withIsRatedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (Average Rating: ' . $product->avg_rating . ') - ';
    echo $product->is_rated_by_user ? 'You rated' : 'You did not rate';
}
```

#### 5.2 Products that Have More Than 5 Approved Reviews, Ordered by Average Rating (with Column)

```php
$products = Product::hasRatings(5, true, false, true)
    ->withAvgRating('DESC', 'avg_rating')
    ->get();
```

#### 5.3 Active Products that the Current User Has Rated, with Number of Reviews and Average Rating Columns

```php
$products = Product::active()  // assuming an active scope exists in the model
    ->ratedByUser()
    ->withCountReviews('DESC', 'reviews_count')
    ->withAvgRating('DESC', 'avg_rating')
    ->get();
```

#### 5.4 Products that Received a 5-Star Rating from Specific Users

```php
$users = [1, 2, 3]; // User IDs
$products = Product::whereHas('ratings', function($q) use ($users) {
    $q->whereIn('user_id', $users)
      ->where('rating', 5);
})->get();
```

#### 5.5 Products with No Approved Reviews

```php
$products = Product::hasNoRatings(true, false, true)->get();
```

---

### 6. Using Helper Functions in Templates (Twig/Blade Example)

#### 6.1 Displaying Average Rating and Number of Reviews for a Product

```php
@php
$product = Product::find(1);
$avgRating = $product->getAverageRating(null, false, true);
$count = $product->getTotalRatings(null, false, true);
$isRated = $product->isRating();
@endphp

<div class="product-rating">
    <span class="stars">@for($i=1;$i<=5;$i++)★@endfor</span>
    <span class="avg">{{ number_format($avgRating, 1) }}</span>
    <span class="count">({{ $count }} reviews)</span>
    @if($isRated)
        <span class="rated">You have rated this product</span>
    @else
        <button class="rate-btn">Rate Product</button>
    @endif
</div>
```

#### 6.2 Displaying a List of Reviews on a Product Page

```php
$ratings = $product->ratings()
    ->where('approved', true)
    ->with('user')
    ->latest()
    ->take(5)
    ->get();
?>

<div class="reviews-list">
    <h4>Latest Reviews</h4>
    @foreach($ratings as $review)
        <div class="review-item">
            <div class="user-info">
                <img src="{{ $review->user->avatar }}" width="40">
                <strong>{{ $review->user->name }}</strong>
                <span>{{ $review->created_at->diffForHumans() }}</span>
            </div>
            <div class="rating">Rating: {{ $review->rating }}/5</div>
            <div class="title">{{ $review->title }}</div>
            <div class="content">{{ $review->content }}</div>
        </div>
    @endforeach
</div>
```

---

### 7. Examples for API Endpoints (JSON)

#### 7.1 Returning Product Details with Rating Statistics

```php
public function show($id)
{
    $product = Product::withIsRatedByUser()
        ->withCountReviews('DESC', 'reviews_count')
        ->withAvgRating('DESC', 'avg_rating')
        ->find($id);
    
    return response()->json($product);
}
```

Result:
```json
{
    "id": 1,
    "name": "Smartphone",
    "reviews_count": 45,
    "avg_rating": 4.7,
    "is_rated_by_user": 1
}
```

#### 7.2 List of Highest Rated Products with User Rating Status Column

```php
public function topRated()
{
    $products = Product::topRated(10)
        ->withIsRatedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API to Add or Update a Rating

```php
public function rate($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    
    $review = $product->toggleRating([
        'rating' => $request->rating,
        'title' => $request->title,
        'content' => $request->content,
        'approved' => false, // requires moderator approval
        'is_positive' => $request->rating >= 4,
    ], $user);
    
    return response()->json([
        'success' => true,
        'review' => $review,
        'avg_rating' => $product->getAverageRating(null, false, true),
        'reviews_count' => $product->getTotalRatings(null, false, true)
    ]);
}
```

---

### 8. Backward Compatibility: Using Old Scopes

If you rely on old code that uses previous scopes, you can continue using them, but it is recommended to gradually upgrade.

```php
// Old
$products = Product::sortByRating('DESC')->get();

// New
$products = Product::sortByAvgRating('DESC')->get();
```

```php
// Old
$products = Product::withRating('DESC')->get();

// New
$products = Product::withAvgRating('DESC')->get();
```

```php
// Old
$products = Product::sortByCreatedAtReviews('DESC')->get();

// New
$products = Product::sortByLatestReview('DESC')->get();
```

```php
// Old
$products = Product::withSortByCountReviews('DESC')->get();

// New
$products = Product::withCountReviews('DESC')->get();
```

---

### 9. Advanced Tips and Tricks

#### 9.1 Using `withTrashed` in Statistics to Include Soft-Deleted Reviews

```php
$totalWithDeleted = $product->getTotalRatings(null, true);
```

#### 9.2 Getting the Count of Positive Reviews Only

```php
$positiveCount = $product->getRatingsCountByType(null, false, null, true)->sum();
```

#### 9.3 Using the `hasRatings` Scope with a Variable Minimum Count

```php
$minCount = request('min_ratings', 5);
$products = Product::hasRatings($minCount, true, false, true)->get();
```

#### 9.4 Customizing the Column Name in Scopes

```php
$products = Product::withCountReviews('DESC', 'my_ratings_column', true)->get();
echo $products->first()->my_ratings_column;
```

#### 9.5 Obtaining Average Rating per User (Detailed Analysis)

```php
$userRatings = $product->ratings()
    ->select('user_id', 'user_type', DB::raw('AVG(rating) as avg'))
    ->groupBy('user_id', 'user_type')
    ->get();
```

---

### 10. Advanced Use Cases in a Social System

#### 10.1 Suggesting Products to a User Based on Friends' Ratings

```php
$user = AuthHelpers::getCurrentUser();

// Users followed by the current user (assuming a follow system exists)
$followedUsers = $user->followedBy()->pluck('id');

// Products that these users have rated with high ratings (more than 4)
$suggestedProducts = Product::whereHas('ratings', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers)
      ->where('rating', '>=', 4);
})->whereNotIn('id', $user->ratedBy()->pluck('id')) // exclude products the user has rated
  ->withAvgRating('DESC', 'avg_rating')
  ->limit(10)
  ->get();
```

#### 10.2 New Review Notifications (When a Review Is Approved)

```php
// After a review is approved by a moderator
$review->approved = true;
$review->save();

if ($review->wasChanged('approved')) {
    // Send notification to the user who wrote the review
    Notification::send($review->user, new ReviewApprovedNotification($product, $review));
    
    // Notification to the product owner
    Notification::send($product->owner, new NewReviewNotification($product, $review));
}
```

#### 10.3 Displaying Average Rating in Product List with Lazy Loading

```php
$products = Product::withAvgRating('DESC', 'avg_rating')
    ->withCountReviews('DESC', 'reviews_count')
    ->paginate(20);
```

#### 10.4 Analyzing Rating Trends (Average Rating Change Over Time)

```php
$product = Product::find(1);

$ratingsByMonth = $product->ratings()
    ->select(DB::raw('DATE_FORMAT(created_at, "%Y-%m") as month'), DB::raw('AVG(rating) as avg_rating'))
    ->where('approved', true)
    ->groupBy('month')
    ->orderBy('month')
    ->get();
```

---

### Conclusion

The `ReviewRateable` behavior provides a rich set of tools that enable you to easily build an integrated rating system. Through the examples provided, you can apply these features in your projects to create interactive user experiences, analyze customer feedback, and improve products based on reviews. Use the advanced scopes and statistical functions to reduce complexity and increase productivity.
