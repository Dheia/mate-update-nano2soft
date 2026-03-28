# Advanced Practical Examples for Using the `LikeableModel` Behavior

This section provides a set of integrated examples illustrating how to use the `LikeableModel` behavior in real-world scenarios, from basic operations to complex queries and advanced statistics.

---

### 1. Basic Operations

#### 1.1 Adding a Like and a Dislike

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Like;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add a like (default type 'like')
$like = Like::add($product, $user);

// Add a dislike (type 'dislike')
$dislike = Like::add($product, $user, 'dislike');

// Add a like with metadata
$like = Like::add($product, $user, 'like', ['reason' => 'High quality']);
```

#### 1.2 Removing a Like or Dislike

```php
// Remove the like (default type)
Like::remove($product, $user);

// Remove only the dislike
Like::remove($product, $user, 'dislike');
```

#### 1.3 Toggling Between Like and Dislike (Toggle)

```php
// If the user liked, it removes the like; otherwise it adds a like
$result = Like::toggle($product, $user, 'like');

// Toggle dislike status
$result = Like::toggle($product, $user, 'dislike');
```

#### 1.4 Getting the Like Record for the Current User

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

$like = $product->getLiked($user);
if ($like) {
    echo "Like type: " . $like->value; // like or dislike
    echo "Additional data: " . $like->metadata['reason'] ?? '';
}
```

---

### 2. Filtering by Users

#### 2.1 Retrieving All Products that the Current User Has Liked

```php
$myLikedProducts = Product::likedByUser()->get();

// Only likes of type 'like'
$myLikes = Product::likedByUser(null, 'like')->get();

// Only dislikes
$myDislikes = Product::likedByUser(null, 'dislike')->get();
```

#### 2.2 Retrieving Products that the Current User Has Not Liked (for Suggestions)

```php
$suggestedProducts = Product::notLikedByUser()->paginate(20);
```

#### 2.3 Retrieving Products that a Specific User Has Liked, Loading Like Details

```php
$user = User::find(2);
$products = Product::likedByUser($user)
    ->with(['likes' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

#### 2.4 Using the Old `whereHasLike` Functions with Control over Behavior When No User Exists

```php
// Default behavior (returns no results when no user exists)
$products = Product::whereHasLike(null, 'like')->get();

// Disable forcing (returns all products when no user exists)
$products = Product::whereHasLike(null, 'like', false)->get();
```

---

### 3. Advanced Ordering and Ranking

#### 3.1 Ordering Products by Like Count (Most Liked)

```php
// Using the direct scope
$topProducts = Product::sortByCountLikes('DESC', 'likes_count')->take(10)->get();

// Using the shortcut topLiked
$topProducts = Product::topLiked(10)->get();

// Ordering by like count of type 'like' only
$topLiked = Product::sortByCountLikes('DESC', 'likes_count', 'like')->get();

// Ordering by dislike count
$topDisliked = Product::sortByCountLikes('DESC', 'dislikes_count', 'dislike')->get();
```

#### 3.2 Adding a Like Count Column to Results (Without Ordering)

```php
$products = Product::addCountLikes()->get();
foreach ($products as $product) {
    echo $product->name . ' - Likes: ' . $product->likes_count;
}
```

#### 3.3 Ordering by Latest Like (Last User Who Liked)

```php
// Order by the creation date of the latest like (most recent first)
$products = Product::sortByLatestLike('DESC')->get();

// Using updated_at instead of created_at
$products = Product::sortByLatestLike('DESC', 'latest_like_at', null, 'updated_at')->get();
```

#### 3.4 Adding the Latest Like Date Column with Ordering

```php
$products = Product::withLatestLike('DESC', 'last_like_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - Latest Like: ' . $product->last_like_date;
}
```

#### 3.5 Ordering by Earliest Like (First Who Liked)

```php
$products = Product::sortByEarliestLike('ASC')->get();
```

---

### 4. Statistics and Reports

#### 4.1 Total Number of Likes for a Specific Product

```php
$product = Product::find(1);

// All likes (including both like and dislike)
$total = $product->getTotalLikes();

// Likes of type 'like' only
$totalLikes = $product->getTotalLikes('like');

// Dislikes
$totalDislikes = $product->getTotalLikes('dislike');
```

#### 4.2 Distribution of Likes by User Type

```php
$counts = $product->getLikesCountByType('like');
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]

// Distribution of dislikes
$dislikeCounts = $product->getLikesCountByType('dislike');
```

#### 4.3 Retrieving a List of User Names Who Liked a Specific Product

```php
$users = $product->getLikersUsers('like');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 Like Statistics within a Time Range (Using Scopes with where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// Products that were liked in the last week
$products = Product::whereHas('likes', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. Combining Scopes for Complex Queries

#### 5.1 Most Liked Products that the Current User Has Also Liked

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topLiked(20)
    ->likedByUser($user)
    ->withIsLikedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (Like Count: ' . $product->likes_count . ') - ';
    echo $product->is_liked_by_user ? 'You liked' : 'You did not like';
}
```

#### 5.2 Products with More Than 5 Likes of Type 'like', Ordered by Latest Like

```php
$products = Product::hasLikes(5, 'like')
    ->withLatestLike('DESC', 'last_like_date')
    ->get();
```

#### 5.3 Active Products that the Current User Has Liked, with Like Count

```php
$products = Product::active()  // assuming an active scope exists in the model
    ->likedByUser()
    ->withCountLikes('DESC', 'likes_count')
    ->get();
```

#### 5.4 Products that the Current User Has Liked, Adding the Status Column (Will Always Be 1)

```php
$products = Product::likedByUser()
    ->withIsLikedByUser()
    ->get();
// All products will have is_liked_by_user = 1
```

#### 5.5 Products that Have Likes of Type 'like' Only, Excluding Products with No Likes

```php
$products = Product::hasLikes(1, 'like')
    ->sortByCountLikes('DESC', 'likes_count', 'like')
    ->get();
```

---

### 6. Using Helper Functions in Templates (Twig/Blade Example)

#### 6.1 Displaying Like/Dislike Buttons

```php
@php
$product = Product::find(1);
$isLiked = $product->isLiked(null, 'like');
$isDisliked = $product->isLiked(null, 'dislike');
$likeCount = $product->getTotalLikes('like');
$dislikeCount = $product->getTotalLikes('dislike');
@endphp

<button class="like-btn" data-id="{{ $product->id }}" data-value="like">
    @if($isLiked)
        ❤️ Unlike
    @else
        🤍 Like
    @endif
</button>
<button class="dislike-btn" data-id="{{ $product->id }}" data-value="dislike">
    @if($isDisliked)
        👎 Undislike
    @else
        👍 Dislike
    @endif
</button>
<span class="likes-count">{{ $likeCount }} Likes</span>
<span class="dislikes-count">{{ $dislikeCount }} Dislikes</span>
```

#### 6.2 Displaying the List of Users Who Liked a Product

```php
$likers = $product->getLikersUsers('like');
?>

<div class="likers-list">
    <h4>Top Likers</h4>
    @foreach($likers->take(5) as $user)
        <div class="liker">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

---

### 7. Examples for API Endpoints (JSON)

#### 7.1 Returning Product Details with Like Information

```php
public function show($id)
{
    $product = Product::withIsLikedByUser()
        ->withCountLikes('DESC', 'likes_count')
        ->find($id);
    
    return response()->json($product);
}
```

Result:
```json
{
    "id": 1,
    "name": "Smartphone",
    "likes_count": 245,
    "is_liked_by_user": 1
}
```

#### 7.2 List of Most Liked Products with User Like Status Column

```php
public function topLiked()
{
    $products = Product::topLiked(10)
        ->withIsLikedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API to Add or Remove a Like

```php
public function toggleLike($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    $value = $request->input('value', 'like');
    
    $result = Like::toggle($product, $user, $value);
    
    return response()->json([
        'success' => true,
        'is_liked' => $product->isLiked($user, $value),
        'likes_count' => $product->getTotalLikes($value)
    ]);
}
```

---

### 8. Backward Compatibility: Using Old Scopes

If you rely on old code that uses previous scopes, you can continue using them, but it is recommended to gradually upgrade.

```php
// Old
$products = Product::sortByCountLikesOld('DESC')->get();

// New
$products = Product::sortByCountLikes('DESC')->get();
```

```php
// Old
$products = Product::withSortByCountLikes('DESC', 'like')->get();

// New
$products = Product::withCountLikes('DESC', 'likes_count', 'like')->get();
```

```php
// Old
$products = Product::sortByCreatedAtLikes('DESC', 'like')->get();

// New
$products = Product::sortByLatestLike('DESC', 'latest_like_at', 'like')->get();
```

```php
// Old
$products = Product::whereHasLike($user, 'like')->get();

// New
$products = Product::likedByUser($user, 'like')->get();
```

---

### 9. Advanced Tips and Tricks

#### 9.1 Using Multiple Like Types

```php
// Set allowed types in settings
// config/nano/markable.php
return [
    'allowed_values' => [
        'like' => ['like', 'dislike', 'love', 'angry'],
    ],
];

// Query products liked as type 'love'
$loveProducts = Product::likedByUser(null, 'love')->get();
```

#### 9.2 Getting Like Count for Each Type

```php
$product = Product::find(1);

$likesByType = [
    'like' => $product->getTotalLikes('like'),
    'dislike' => $product->getTotalLikes('dislike'),
    'love' => $product->getTotalLikes('love'),
];
```

#### 9.3 Using the `hasLikes` Scope with a Variable Minimum Count

```php
$minCount = request('min_likes', 5);
$products = Product::hasLikes($minCount, 'like')->get();
```

#### 9.4 Customizing the Column Name in Scopes

```php
$products = Product::withCountLikes('DESC', 'my_likes_column', 'like')->get();
echo $products->first()->my_likes_column;
```

#### 9.5 Combining `likedByUser` and `notLikedByUser` in One Query (Using union)

```php
$liked = Product::likedByUser()->get();
$notLiked = Product::notLikedByUser()->get();
$all = $liked->merge($notLiked);
```

---

### 10. Advanced Use Cases in a Social System

#### 10.1 Suggesting Products to a User Based on Friends' Likes

```php
$user = AuthHelpers::getCurrentUser();

// Users followed by the current user (assuming a follow system exists)
$followedUsers = $user->followedBy()->pluck('id');

// Products that these users have liked (type like)
$suggestedProducts = Product::whereHas('likes', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers)
      ->where('value', 'like');
})->whereNotIn('id', $user->likedBy()->pluck('id')) // exclude products the user has liked
  ->withCountLikes('DESC', 'likes_count')
  ->limit(10)
  ->get();
```

#### 10.2 New Like Notifications (When a User Likes a Product)

```php
// After adding the like
$like = Like::add($product, $user, 'like');

// Send notification to the product owner
if ($product->owner_id != $user->id) {
    Notification::send($product->owner, new NewLikeNotification($product, $user));
}
```

#### 10.3 Displaying Like Count in Product List with Lazy Loading

```php
$products = Product::withCountLikes('DESC', 'likes_count')
    ->paginate(20);
```

#### 10.4 Analyzing User Behavior (Most Active via Likes)

```php
// Users who have liked more than 10 products
$activeUsers = User::has('likes', '>=', 10)->get();

// Products most liked by active users
$topProducts = Product::whereHas('likes', function($q) use ($activeUsers) {
    $q->whereIn('user_id', $activeUsers->pluck('id'))
      ->where('value', 'like');
})->withCountLikes('DESC', 'likes_count')
  ->limit(10)
  ->get();
```

#### 10.5 Creating a Composite Rating System (Like/Dislike)

```php
// Calculate like ratio for a product
$product = Product::find(1);
$likes = $product->getTotalLikes('like');
$dislikes = $product->getTotalLikes('dislike');
$total = $likes + $dislikes;
$ratio = $total > 0 ? round(($likes / $total) * 100, 1) : 0;

echo "Like ratio: {$ratio}%";
```

---

### Conclusion

The `LikeableModel` behavior provides a rich set of tools that enable you to easily build an integrated likes system. Through the examples provided, you can apply these features in your projects to create interactive user experiences, analyze user behavior, and improve application performance. Use the advanced scopes and statistical functions to reduce complexity and increase productivity.
