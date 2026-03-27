# Advanced Practical Examples for Using the `BookmarkableModel` Behavior

This section provides a set of integrated examples illustrating how to use the `BookmarkableModel` behavior in real-world scenarios, from basic operations to complex queries and advanced statistics.

---

### 1. Basic Operations

#### 1.1 Adding Bookmarks of Different Types

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Bookmark;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add a bookmark of type 'favorite'
$bookmark = Bookmark::add($product, $user, 'favorite');

// Add a bookmark of type 'read_later'
$bookmark = Bookmark::add($product, $user, 'read_later', ['notes' => 'Important article']);

// Add a bookmark of type 'wishlist'
$bookmark = Bookmark::add($product, $user, 'wishlist');

// Add a bookmark with additional metadata
$bookmark = Bookmark::add($product, $user, 'favorite', [
    'priority' => 'high',
    'tags' => ['electronics', 'sale']
]);
```

#### 1.2 Removing a Bookmark

```php
// Remove the bookmark (any type)
Bookmark::remove($product, $user);

// Remove a bookmark of type 'read_later' only
Bookmark::remove($product, $user, 'read_later');
```

#### 1.3 Toggling Between Adding and Removing (Toggle)

```php
// If the product is bookmarked, it removes it; otherwise, it adds it
$result = Bookmark::toggle($product, $user, 'favorite');

// Toggle a bookmark of type 'wishlist'
$result = Bookmark::toggle($product, $user, 'wishlist');
```

#### 1.4 Getting the Bookmark Record for the Current User

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

$bookmark = $product->getBookmarked($user);
if ($bookmark) {
    echo "Bookmark type: " . $bookmark->value; // favorite, read_later, wishlist
    echo "Notes: " . $bookmark->metadata['notes'] ?? '';
}
```

---

### 2. Filtering by Users

#### 2.1 Retrieving All Products that the Current User Has Bookmarked

```php
// All bookmarks (any type)
$myBookmarks = Product::bookmarkedByUser()->get();

// Only bookmarks of type 'favorite'
$myFavorites = Product::bookmarkedByUser(null, 'favorite')->get();

// Only bookmarks of type 'read_later'
$myReadLater = Product::bookmarkedByUser(null, 'read_later')->get();
```

#### 2.2 Retrieving Products that the Current User Has Not Bookmarked (for Suggestions)

```php
$suggestedProducts = Product::notBookmarkedByUser()->paginate(20);
```

#### 2.3 Retrieving Products that a Specific User Has Bookmarked, Loading Bookmark Details

```php
$user = User::find(2);
$products = Product::bookmarkedByUser($user)
    ->with(['bookmarks' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

#### 2.4 Using the Old `whereHasBookmark` Functions with Control over Behavior When No User Exists

```php
// Default behavior (returns no results when no user exists)
$products = Product::whereHasBookmark(null, 'favorite')->get();

// Disable forcing (returns all products when no user exists)
$products = Product::whereHasBookmark(null, 'favorite', false)->get();
```

---

### 3. Advanced Ordering and Ranking

#### 3.1 Ordering Products by Bookmark Count (Most Bookmarked)

```php
// Using the direct scope
$topProducts = Product::sortByCountBookmarks('DESC', 'bookmarks_count')->take(10)->get();

// Using the shortcut topBookmarked
$topProducts = Product::topBookmarked(10)->get();

// Ordering by bookmark count of type 'favorite' only
$topFavorites = Product::sortByCountBookmarks('DESC', 'favorites_count', 'favorite')->get();

// Ordering by bookmark count of type 'read_later' only
$topReadLater = Product::sortByCountBookmarks('DESC', 'read_later_count', 'read_later')->get();
```

#### 3.2 Adding a Bookmark Count Column to Results (Without Ordering)

```php
$products = Product::addCountBookmarks()->get();
foreach ($products as $product) {
    echo $product->name . ' - Bookmark Count: ' . $product->bookmarks_count;
}
```

#### 3.3 Ordering by Latest Bookmark (Last User Who Bookmarked)

```php
// Order by the creation date of the latest bookmark (most recent first)
$products = Product::sortByLatestBookmark('DESC')->get();

// Using updated_at instead of created_at
$products = Product::sortByLatestBookmark('DESC', 'latest_bookmark_at', null, 'updated_at')->get();
```

#### 3.4 Adding the Latest Bookmark Date Column with Ordering

```php
$products = Product::withLatestBookmark('DESC', 'last_bookmark_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - Latest Bookmark: ' . $product->last_bookmark_date;
}
```

#### 3.5 Ordering by Earliest Bookmark (First Who Bookmarked)

```php
$products = Product::sortByEarliestBookmark('ASC')->get();
```

---

### 4. Statistics and Reports

#### 4.1 Total Number of Bookmarks for a Specific Product

```php
$product = Product::find(1);

// All bookmarks (all types)
$total = $product->getTotalBookmarks();

// Bookmarks of type 'favorite' only
$totalFavorites = $product->getTotalBookmarks('favorite');

// Bookmarks of type 'read_later' only
$totalReadLater = $product->getTotalBookmarks('read_later');
```

#### 4.2 Distribution of Bookmarks by User Type

```php
$counts = $product->getBookmarksCountByType('favorite');
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]

// Distribution of 'read_later' bookmarks
$readLaterCounts = $product->getBookmarksCountByType('read_later');
```

#### 4.3 Retrieving a List of User Names Who Bookmarked a Specific Product

```php
$users = $product->getBookmarkersUsers('favorite');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 Bookmark Statistics within a Time Range (Using Scopes with where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// Products that were bookmarked in the last week
$products = Product::whereHas('bookmarks', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. Combining Scopes for Complex Queries

#### 5.1 Most Bookmarked Products that the Current User Has Also Bookmarked

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topBookmarked(20)
    ->bookmarkedByUser($user)
    ->withIsBookmarkedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (Bookmark Count: ' . $product->bookmarks_count . ') - ';
    echo $product->is_bookmarked_by_user ? 'You bookmarked' : 'You did not bookmark';
}
```

#### 5.2 Products with More Than 5 Bookmarks of Type 'favorite', Ordered by Latest Bookmark

```php
$products = Product::hasBookmarks(5, 'favorite')
    ->withLatestBookmark('DESC', 'last_favorite_date')
    ->get();
```

#### 5.3 Active Products that the Current User Has Bookmarked, with Bookmark Count

```php
$products = Product::active()  // assuming an active scope exists in the model
    ->bookmarkedByUser()
    ->withCountBookmarks('DESC', 'bookmarks_count')
    ->get();
```

#### 5.4 Products that the Current User Has Bookmarked, Adding the Status Column (Will Always Be 1)

```php
$products = Product::bookmarkedByUser()
    ->withIsBookmarkedByUser()
    ->get();
// All products will have is_bookmarked_by_user = 1
```

#### 5.5 Products that Have Bookmarks of Type 'read_later' Only, Excluding Products with No Bookmarks

```php
$products = Product::hasBookmarks(1, 'read_later')
    ->sortByCountBookmarks('DESC', 'read_later_count', 'read_later')
    ->get();
```

---

### 6. Using Helper Functions in Templates (Twig/Blade Example)

#### 6.1 Displaying Multiple Bookmark Buttons

```php
@php
$product = Product::find(1);
$isFavorite = $product->isBookmarked(null, 'favorite');
$isReadLater = $product->isBookmarked(null, 'read_later');
$favoriteCount = $product->getTotalBookmarks('favorite');
$readLaterCount = $product->getTotalBookmarks('read_later');
@endphp

<button class="bookmark-btn" data-type="favorite" data-id="{{ $product->id }}">
    @if($isFavorite)
        ⭐ Remove from Favorites
    @else
        ☆ Add to Favorites
    @endif
</button>
<button class="bookmark-btn" data-type="read_later" data-id="{{ $product->id }}">
    @if($isReadLater)
        📖 Remove from Read Later
    @else
        📚 Read Later
    @endif
</button>
<span class="favorite-count">{{ $favoriteCount }}</span>
<span class="read-later-count">{{ $readLaterCount }}</span>
```

#### 6.2 Displaying the List of Users Who Favorited a Product

```php
$bookmarkers = $product->getBookmarkersUsers('favorite');
?>

<div class="bookmarkers-list">
    <h4>Top Favoriters</h4>
    @foreach($bookmarkers->take(5) as $user)
        <div class="bookmarker">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

---

### 7. Examples for API Endpoints (JSON)

#### 7.1 Returning Product Details with Bookmark Information

```php
public function show($id)
{
    $product = Product::withIsBookmarkedByUser()
        ->withCountBookmarks('DESC', 'bookmarks_count')
        ->find($id);
    
    return response()->json($product);
}
```

Result:
```json
{
    "id": 1,
    "name": "Smartphone",
    "bookmarks_count": 245,
    "is_bookmarked_by_user": 1
}
```

#### 7.2 List of Most Bookmarked Products with User Bookmark Status Column

```php
public function topBookmarked()
{
    $products = Product::topBookmarked(10)
        ->withIsBookmarkedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API to Add or Remove a Bookmark

```php
public function toggleBookmark($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    $value = $request->input('value', 'favorite');
    
    $result = Bookmark::toggle($product, $user, $value);
    
    return response()->json([
        'success' => true,
        'is_bookmarked' => $product->isBookmarked($user, $value),
        'bookmarks_count' => $product->getTotalBookmarks($value)
    ]);
}
```

---

### 8. Backward Compatibility: Using Old Scopes

If you rely on old code that uses previous scopes, you can continue using them, but it is recommended to gradually upgrade.

```php
// Old
$products = Product::sortByCountBookmarksOld('DESC')->get();

// New
$products = Product::sortByCountBookmarks('DESC')->get();
```

```php
// Old
$products = Product::withSortByCountBookmarks('DESC', 'favorite')->get();

// New
$products = Product::withCountBookmarks('DESC', 'bookmarks_count', 'favorite')->get();
```

```php
// Old
$products = Product::sortByCreatedAtBookmarks('DESC', 'favorite')->get();

// New
$products = Product::sortByLatestBookmark('DESC', 'latest_bookmark_at', 'favorite')->get();
```

```php
// Old
$products = Product::whereHasBookmark($user, 'favorite')->get();

// New
$products = Product::bookmarkedByUser($user, 'favorite')->get();
```

---

### 9. Advanced Tips and Tricks

#### 9.1 Using Multiple Bookmark Types

```php
// Set allowed types in settings
// config/nano/markable.php
return [
    'allowed_values' => [
        'bookmark' => ['favorite', 'read_later', 'wishlist', 'archive'],
    ],
];

// Query products bookmarked as type 'wishlist'
$wishlist = Product::bookmarkedByUser(null, 'wishlist')->get();
```

#### 9.2 Getting Bookmark Count for Each Type

```php
$product = Product::find(1);

$bookmarksByType = [
    'favorite' => $product->getTotalBookmarks('favorite'),
    'read_later' => $product->getTotalBookmarks('read_later'),
    'wishlist' => $product->getTotalBookmarks('wishlist'),
];
```

#### 9.3 Using the `hasBookmarks` Scope with a Variable Minimum Count

```php
$minCount = request('min_bookmarks', 5);
$products = Product::hasBookmarks($minCount, 'favorite')->get();
```

#### 9.4 Customizing the Column Name in Scopes

```php
$products = Product::withCountBookmarks('DESC', 'my_favorites_column', 'favorite')->get();
echo $products->first()->my_favorites_column;
```

#### 9.5 Combining `bookmarkedByUser` and `notBookmarkedByUser` in One Query (Using union)

```php
$bookmarked = Product::bookmarkedByUser()->get();
$notBookmarked = Product::notBookmarkedByUser()->get();
$all = $bookmarked->merge($notBookmarked);
```

---

### 10. Advanced Use Cases in a Social System

#### 10.1 Suggesting Products to a User Based on Friends' Bookmarks

```php
$user = AuthHelpers::getCurrentUser();

// Users followed by the current user (assuming a follow system exists)
$followedUsers = $user->followedBy()->pluck('id');

// Products that these users have bookmarked (type favorite)
$suggestedProducts = Product::whereHas('bookmarks', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers)
      ->where('value', 'favorite');
})->whereNotIn('id', $user->bookmarkedBy()->pluck('id')) // exclude products the user has bookmarked
  ->withCountBookmarks('DESC', 'favorites_count', 'favorite')
  ->limit(10)
  ->get();
```

#### 10.2 New Bookmark Notifications (When a User Bookmarks a Product)

```php
// After adding the bookmark
$bookmark = Bookmark::add($product, $user, 'favorite');

// Send notification to the product owner
if ($product->owner_id != $user->id) {
    Notification::send($product->owner, new NewBookmarkNotification($product, $user));
}
```

#### 10.3 Displaying Bookmark Count in Product List with Lazy Loading

```php
$products = Product::withCountBookmarks('DESC', 'bookmarks_count')
    ->paginate(20);
```

#### 10.4 Analyzing User Behavior (Most Active via Bookmarks)

```php
// Users who have bookmarked more than 10 products
$activeUsers = User::has('bookmarks', '>=', 10)->get();

// Products most bookmarked by active users (type favorite)
$topProducts = Product::whereHas('bookmarks', function($q) use ($activeUsers) {
    $q->whereIn('user_id', $activeUsers->pluck('id'))
      ->where('value', 'favorite');
})->withCountBookmarks('DESC', 'favorites_count', 'favorite')
  ->limit(10)
  ->get();
```

#### 10.5 Creating Custom Lists (e.g., "Wishlist")

```php
// Add product to wishlist
$product = Product::find(1);
Bookmark::add($product, $user, 'wishlist', [
    'priority' => 'high',
    'expected_price' => 500
]);

// Retrieve wishlist
$wishlist = Product::bookmarkedByUser(null, 'wishlist')
    ->with(['bookmarks' => function($q) use ($user) {
        $q->where('value', 'wishlist')
          ->where('user_id', $user->id);
    }])
    ->get();

// Display wishlist with priorities
foreach ($wishlist as $item) {
    $bookmark = $item->bookmarks->first();
    echo $item->name . ' - Priority: ' . ($bookmark->metadata['priority'] ?? 'Normal');
}
```

---

### Conclusion

The `BookmarkableModel` behavior provides a rich set of tools that enable you to easily build an integrated bookmark system. Through the examples provided, you can apply these features in your projects to create interactive user experiences, analyze user behavior, and improve application performance. Use the advanced scopes and statistical functions to reduce complexity and increase productivity.