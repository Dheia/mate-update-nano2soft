# Advanced Practical Examples for Using the `FavoriteableModel` Behavior

This section provides a set of integrated examples illustrating how to use the `FavoriteableModel` behavior in real-world scenarios, from basic operations to complex queries and advanced statistics.

---

### 1. Basic Operations

#### 1.1 Adding an Item to Favorites with Different Types

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Favorite;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add product to favorites (default type 'favorite')
$favorite = Favorite::add($product, $user);

// Add product as a 'like' favorite
$like = Favorite::add($product, $user, 'like');

// Add product as a 'watch_later' favorite
$watchLater = Favorite::add($product, $user, 'watch_later', ['notes' => 'Watch later']);

// Add product with additional metadata
$favorite = Favorite::add($product, $user, 'favorite', [
    'notes' => 'Great product',
    'priority' => 'high'
]);
```

#### 1.2 Removing an Item from Favorites

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Remove product from favorites (default type)
Favorite::remove($product, $user);

// Remove product from favorites of type 'like' only
Favorite::remove($product, $user, 'like');
```

#### 1.3 Toggling Between Adding and Removing (Toggle)

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// If the product is favorited, it removes it; otherwise, it adds it
$result = Favorite::toggle($product, $user, 'favorite', ['notes' => 'Toggled']);
```

#### 1.4 Getting the Favorite Record for the Current User

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

$favorite = $product->getFavorited($user);
if ($favorite) {
    echo "Favorite type: " . $favorite->value;
    echo "Notes: " . $favorite->metadata['notes'] ?? '';
}
```

---

### 2. Filtering by Users

#### 2.1 Retrieving All Products that the Current User Has Favorited

```php
$myFavorites = Product::favoritedByUser()->get();

// Including only a specific type
$myLikes = Product::favoritedByUser(null, 'like')->get();

// Including two different types (using orWhereHas)
$myFavorites = Product::whereHas('favorites', function($q) use ($user) {
    $q->where('user_id', $user->id)
      ->whereIn('value', ['favorite', 'like']);
})->get();
```

#### 2.2 Retrieving Products that the Current User Has Not Favorited (for Suggestions)

```php
$suggestedProducts = Product::notFavoritedByUser()->paginate(20);
```

#### 2.3 Retrieving Products that a Specific User Has Favorited, Loading Favorite Details

```php
$user = User::find(2);
$products = Product::favoritedByUser($user)
    ->with(['favorites' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

#### 2.4 Using the Old `whereHasFavorite` Functions with Control over Behavior When No User Exists

```php
// Default behavior (returns no results when no user exists)
$products = Product::whereHasFavorite(null, 'like')->get();

// Disable forcing (returns all products when no user exists)
$products = Product::whereHasFavorite(null, 'like', false)->get();
```

---

### 3. Advanced Ordering and Ranking

#### 3.1 Ordering Products by Favorite Count (Most Favorited)

```php
// Using the direct scope
$topProducts = Product::sortByCountFavorites('DESC', 'favorites_count')->take(10)->get();

// Using the shortcut topFavorited
$topProducts = Product::topFavorited(10)->get();

// Ordering by favorite count of type 'like' only
$topLiked = Product::sortByCountFavorites('DESC', 'likes_count', 'like')->get();
```

#### 3.2 Adding a Favorite Count Column to Results (Without Ordering)

```php
$products = Product::addCountFavorites()->get();
foreach ($products as $product) {
    echo $product->name . ' - Favorite Count: ' . $product->favorites_count;
}
```

#### 3.3 Ordering by Latest Favorite (Last User Who Favorited)

```php
// Order by the creation date of the latest favorite (most recent first)
$products = Product::sortByLatestFavorite('DESC')->get();

// Using updated_at instead of created_at
$products = Product::sortByLatestFavorite('DESC', 'latest_favorite_at', null, null, null, 'updated_at')->get();
```

#### 3.4 Adding the Latest Favorite Date Column with Ordering

```php
$products = Product::withLatestFavorite('DESC', 'last_favorite_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - Latest Favorite: ' . $product->last_favorite_date;
}
```

#### 3.5 Ordering by Earliest Favorite (First Who Favorited)

```php
$products = Product::sortByEarliestFavorite('ASC')->get();
```

---

### 4. Statistics and Reports

#### 4.1 Total Number of Favorites for a Specific Product

```php
$product = Product::find(1);

// All favorites
$total = $product->getTotalFavorites();

// Favorites of type 'like' only
$totalLikes = $product->getTotalFavorites('like');
```

#### 4.2 Distribution of Favorites by User Type

```php
$counts = $product->getFavoritesCountByType();
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]

// Distribution of favorites of type 'like' only
$likesByType = $product->getFavoritesCountByType('like');
```

#### 4.3 Retrieving a List of User Names Who Favorited a Specific Product

```php
$users = $product->getFavoritersUsers();
foreach ($users as $user) {
    echo $user->name . "\n";
}

// Only users who favorited as type 'like'
$likers = $product->getFavoritersUsers('like');
```

#### 4.4 Favorite Statistics within a Time Range (Using Scopes with where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// Products that were favorited in the last week
$products = Product::whereHas('favorites', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. Combining Scopes for Complex Queries

#### 5.1 Most Favorited Products that the Current User Has Also Favorited

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topFavorited(20)
    ->favoritedByUser($user)
    ->withIsFavoritedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (Favorite Count: ' . $product->favorites_count . ') - ';
    echo $product->is_favorited_by_user ? 'You favorited' : 'You did not favorite';
}
```

#### 5.2 Products with More Than 5 Favorites of Type 'like', Ordered by Latest Favorite

```php
$products = Product::hasFavorites(5, 'like')
    ->withLatestFavorite('DESC', 'last_like_date')
    ->get();
```

#### 5.3 Active Products that the Current User Has Favorited, with Favorite Count

```php
$products = Product::active()  // assuming an active scope exists in the model
    ->favoritedByUser()
    ->withCountFavorites('DESC', 'favorites_count')
    ->get();
```

#### 5.4 Products that the Current User Has Favorited, Adding the Status Column (Will Always Be 1)

```php
$products = Product::favoritedByUser()
    ->withIsFavoritedByUser()
    ->get();
// All products will have is_favorited_by_user = 1
```

#### 5.5 Products that Have Favorites of Type 'favorite' Only, Excluding Products with No Favorites

```php
$products = Product::hasFavorites(1, 'favorite')
    ->sortByCountFavorites('DESC', 'favorites_count', 'favorite')
    ->get();
```

---

### 6. Using Helper Functions in Templates (Twig/Blade Example)

#### 6.1 Displaying a Favorite Toggle Button

```php
@php
$product = Product::find(1);
$isFavorited = $product->isFavorited();
$favoriteCount = $product->getTotalFavorites();
@endphp

<button class="favorite-btn" data-id="{{ $product->id }}" data-value="favorite">
    @if($isFavorited)
        Remove from Favorites
    @else
        Add to Favorites
    @endif
</button>
<span class="favorites-count">{{ $favoriteCount }} Favorites</span>
```

#### 6.2 Displaying the List of Users Who Favorited a Product

```php
$favoriters = $product->getFavoritersUsers();
?>

<div class="favoriters-list">
    <h4>Top Favoriters</h4>
    @foreach($favoriters->take(5) as $user)
        <div class="favoriter">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

#### 6.3 Displaying Buttons for Multiple Favorite Types

```php
@php
$product = Product::find(1);
$isFavorite = $product->isFavorited(null, 'favorite');
$isLike = $product->isFavorited(null, 'like');
@endphp

<button class="favorite-btn" data-type="favorite">
    {{ $isFavorite ? '❤️' : '🤍' }} Favorite
</button>
<button class="like-btn" data-type="like">
    {{ $isLike ? '👍' : '👎' }} Like
</button>
```

---

### 7. Examples for API Endpoints (JSON)

#### 7.1 Returning Product Details with Favorite Information

```php
public function show($id)
{
    $product = Product::withIsFavoritedByUser()
        ->withCountFavorites('DESC', 'favorites_count')
        ->find($id);
    
    return response()->json($product);
}
```

Result:
```json
{
    "id": 1,
    "name": "Smartphone",
    "favorites_count": 245,
    "is_favorited_by_user": 1
}
```

#### 7.2 List of Most Favorited Products with User Favorite Status Column

```php
public function topFavorited()
{
    $products = Product::topFavorited(10)
        ->withIsFavoritedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API to Add or Remove an Item from Favorites

```php
public function toggleFavorite($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    $value = $request->input('value', 'favorite');
    
    $result = Favorite::toggle($product, $user, $value);
    
    return response()->json([
        'success' => true,
        'is_favorited' => $product->isFavorited($user, $value),
        'favorites_count' => $product->getTotalFavorites($value)
    ]);
}
```

---

### 8. Backward Compatibility: Using Old Scopes

If you rely on old code that uses previous scopes, you can continue using them, but it is recommended to gradually upgrade.

```php
// Old
$products = Product::sortByCountFavoritesOld('DESC')->get();

// New
$products = Product::sortByCountFavorites('DESC')->get();
```

```php
// Old
$products = Product::withSortByCountFavorites('DESC')->get();

// New
$products = Product::withCountFavorites('DESC')->get();
```

```php
// Old
$products = Product::sortByCreatedAtFavorites('DESC')->get();

// New
$products = Product::sortByLatestFavorite('DESC')->get();
```

```php
// Old
$products = Product::whereHasFavorite($user, 'like')->get();

// New
$products = Product::favoritedByUser($user, 'like')->get();
```

---

### 9. Advanced Tips and Tricks

#### 9.1 Using Multiple Favorite Types in the Same Model

```php
// Set allowed types in settings
// config/nano/markable.php
return [
    'allowed_values' => [
        'favorite' => ['favorite', 'like', 'watch_later', 'saved'],
    ],
];

// Query products favorited as type 'watch_later'
$watchLater = Product::favoritedByUser(null, 'watch_later')->get();
```

#### 9.2 Getting Favorite Count for Each Type

```php
$product = Product::find(1);

$favoritesByType = [
    'favorite' => $product->getTotalFavorites('favorite'),
    'like' => $product->getTotalFavorites('like'),
    'watch_later' => $product->getTotalFavorites('watch_later'),
];
```

#### 9.3 Using the `hasFavorites` Scope with a Variable Minimum Count

```php
$minCount = request('min_favorites', 5);
$products = Product::hasFavorites($minCount, 'like')->get();
```

#### 9.4 Customizing the Column Name in Scopes

```php
$products = Product::withCountFavorites('DESC', 'my_favorites_column', 'like')->get();
echo $products->first()->my_favorites_column;
```

#### 9.5 Combining `favoritedByUser` and `notFavoritedByUser` in One Query (Using union)

```php
$favorited = Product::favoritedByUser()->get();
$notFavorited = Product::notFavoritedByUser()->get();
$all = $favorited->merge($notFavorited);
```

---

### 10. Advanced Use Cases in a Social System

#### 10.1 Suggesting Products to a User Based on Friends' Favorites

```php
$user = AuthHelpers::getCurrentUser();

// Users followed by the current user (assuming a follow system exists)
$followedUsers = $user->followedBy()->pluck('id');

// Products that these users have favorited
$suggestedProducts = Product::whereHas('favorites', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers);
})->whereNotIn('id', $user->favoritedBy()->pluck('id')) // exclude products the user has favorited
  ->withCountFavorites('DESC', 'favorites_count')
  ->limit(10)
  ->get();
```

#### 10.2 New Favorite Notifications (When a User Favorites a Product)

```php
// After adding the product to favorites
$favorite = Favorite::add($product, $user, 'favorite');

// Send notification to the product owner
if ($product->owner_id != $user->id) {
    Notification::send($product->owner, new NewFavoriteNotification($product, $user));
}
```

#### 10.3 Displaying Favorite Count in Product List with Lazy Loading

```php
$products = Product::withCountFavorites('DESC', 'favorites_count')
    ->paginate(20);
```

#### 10.4 Analyzing User Behavior (Most Active via Favorites)

```php
// Users who have favorited more than 10 products
$activeUsers = User::has('favorites', '>=', 10)->get();

// Products most favorited by active users
$topProducts = Product::whereHas('favorites', function($q) use ($activeUsers) {
    $q->whereIn('user_id', $activeUsers->pluck('id'));
})->withCountFavorites('DESC', 'favorites_count')
  ->limit(10)
  ->get();
```

#### 10.5 Creating Custom Favorite Lists (e.g., "Read Later")

```php
// Add an article to "Read Later" list
$article = Article::find(1);
Favorite::add($article, $user, 'read_later', ['notes' => 'Important article']);

// Retrieve "Read Later" list
$readLater = Article::favoritedByUser(null, 'read_later')->get();
```

---

### Conclusion

The `FavoriteableModel` behavior provides a rich set of tools that enable you to easily build an integrated favorites system. Through the examples provided, you can apply these features in your projects to create interactive user experiences, analyze user behavior, and improve application performance. Use the advanced scopes and statistical functions to reduce complexity and increase productivity.
