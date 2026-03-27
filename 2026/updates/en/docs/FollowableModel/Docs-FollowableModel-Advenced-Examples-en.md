## Advanced Practical Examples for Using the `FollowableModel` Behavior

This section provides a collection of integrated examples demonstrating how to use the `FollowableModel` behavior in real-world scenarios, from basic operations to complex queries and advanced statistics.

---

### 1. Basic Operations

#### 1.1 Adding a New Follow with Custom Data

```php
use Nano\Follows\Models\Follow;
use Nano\AuthApi\Classes\AuthHelpers;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add a regular follow (accepted by default)
$follow = $product->addFollow([], $user);

// Add a follow with acceptance disabled (pending follow)
$follow = $product->addFollow(['is_accepted' => false], $user);

// Add a follow with additional content (e.g., reason for following)
$follow = $product->addFollow([
    'content' => 'Following to receive special offers',
    'is_accepted' => true,
], $user);
```

#### 1.2 Toggle Follow/Unfollow

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// If the user is following, it deletes the follow
// If they are not following, it adds it
$product->toggleFollow([], $user);
```

#### 1.3 Updating an Existing Follow Status

```php
$user = AuthHelpers::getCurrentUser();
$follow = $product->getUserFollow($user);

if ($follow) {
    // Accept the follow after it was pending
    $follow = $product->updateFollow($follow->id, ['is_accepted' => true, 'accepted_at' => now()]);
}
```

#### 1.4 Deleting a Follow (Soft Delete)

```php
$user = AuthHelpers::getCurrentUser();
$follow = $product->getUserFollow($user);

if ($follow) {
    $product->deleteFollow($follow->id);
}
```

---

### 2. Filtering by Users

#### 2.1 Retrieving All Products Followed by the Current User

```php
$myFollowedProducts = Product::followedByUser()->get();

// Including non-accepted follows
$myFollowedProducts = Product::followedByUser(null, false)->get();

// Including soft-deleted follows
$myFollowedProducts = Product::followedByUser(null, true, null, true)->get();
```

#### 2.2 Retrieving Products Not Followed by the Current User (to Suggest Them)

```php
$suggestedProducts = Product::notFollowedByUser()->paginate(20);
```

#### 2.3 Using the Legacy `whereHasFollow` Scope (for Backward Compatibility)

```php
$products = Product::whereHasFollow($user, true)->get();
```

#### 2.4 Retrieving Products Followed by a Specific User, Loading Follow Details

```php
$user = User::find(2);
$products = Product::followedByUser($user)
    ->with(['follows' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

---

### 3. Advanced Sorting and Ranking

#### 3.1 Sorting Products by Follower Count (Most Followed)

```php
// Using the direct scope
$topProducts = Product::sortByCountFollows('DESC', 'follows_count', true)->take(10)->get();

// Using the topFollowed shortcut
$topProducts = Product::topFollowed(10)->get();

// Sort by count of active follows only (including non-accepted)
$topActive = Product::sortByCountFollows('DESC', 'active_follows_count', null, true)->get();
```

#### 3.2 Adding a Follow Count Column to Results (Without Sorting)

```php
$products = Product::addCountFollows()->get();
foreach ($products as $product) {
    echo $product->name . ' - Followers: ' . $product->follows_count;
}
```

#### 3.3 Sorting by Latest Follow (Most Recent Follower)

```php
// Sort by the latest follow creation date (newest first)
$products = Product::sortByLatestFollow('DESC')->get();

// Use the accepted_at field instead of created_at
$products = Product::sortByLatestFollow('DESC', 'latest_follow_at', true, null, false, 'accepted_at')->get();
```

#### 3.4 Adding a Latest Follow Date Column with Sorting

```php
$products = Product::withLatestFollow('DESC', 'last_follow_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - Latest follow: ' . $product->last_follow_date;
}
```

#### 3.5 Sorting by Earliest Follow (First Follower)

```php
$products = Product::sortByEarliestFollow('ASC')->get();
```

---

### 4. Statistics and Reports

#### 4.1 Total Number of Followers for a Given Product

```php
$product = Product::find(1);

// All followers
$total = $product->getTotalFollowers();

// Only accepted followers
$totalAccepted = $product->getTotalFollowers(true);

// Only accepted and active followers
$totalActiveAccepted = $product->getTotalFollowers(true, true);
```

#### 4.2 Distribution of Followers by User Type

```php
$counts = $product->getFollowersCountByType(true, true);
// ['RainLab\User\Models\User' => 42, 'App\Models\Admin' => 5]
```

#### 4.3 Fetching a List of User Names Who Follow a Specific Product

```php
$users = $product->getFollowersUsers(true, true);
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 Follow Statistics Within a Time Range (Using Scopes with where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// Products that were followed during the last week
$products = Product::whereHas('follows', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. Combining Scopes for Complex Queries

#### 5.1 Most Followed (Accepted) Products That the Current User Also Follows

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topFollowed(20)
    ->followedByUser($user)
    ->withIsFollowedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (Followers: ' . $product->follows_count . ') - ';
    echo $product->is_followed_by_user ? 'You follow it' : 'You do not follow it';
}
```

#### 5.2 Products with More than 3 Accepted Followers, Sorted by Latest Follow (with Column Added)

```php
$products = Product::hasFollowers(3, true)
    ->withLatestFollow('DESC', 'last_follow_date')
    ->get();
```

#### 5.3 Active Products Followed by the Current User, with Follower Count

```php
$products = Product::active()  // assuming an active scope exists on the model
    ->followedByUser()
    ->withCountFollows('DESC', 'follows_count', true)
    ->get();
```

#### 5.4 Getting Products Followed by the Current User, Adding the Follow Status Column for the User (Will Always Be 1)

```php
$products = Product::followedByUser()
    ->withIsFollowedByUser()
    ->get();
// All products will have is_followed_by_user = 1
```

#### 5.5 Products with Accepted Followers, Excluding Products with No Followers, and Sorting by Follower Count

```php
$products = Product::hasFollowers(1, true)
    ->sortByCountFollows('DESC', 'follows_count', true)
    ->get();
```

---

### 6. Using Helper Functions in Templates (Example Twig/Blade)

#### 6.1 Displaying a Follow/Unfollow Button

```php
@php
$product = Product::find(1);
$isFollowed = $product->isUserFollow();
$followCount = $product->getTotalFollowers(true);
@endphp

<button class="follow-btn" data-id="{{ $product->id }}">
    @if($isFollowed)
        Unfollow
    @else
        Follow
    @endif
</button>
<span class="followers-count">{{ $followCount }} followers</span>
```

#### 6.2 Displaying a List of Followers on the Product Page

```php
$followers = $product->getFollowersUsers(true, true);
?>

<div class="followers-list">
    <h4>Top Followers</h4>
    @foreach($followers->take(5) as $user)
        <div class="follower">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

---

### 7. API Examples (JSON)

#### 7.1 Returning Product Details with Follow Information

```php
public function show($id)
{
    $product = Product::withIsFollowedByUser()
        ->withCountFollows('DESC', 'follows_count', true)
        ->find($id);
    
    return response()->json($product);
}
```

Output:
```json
{
    "id": 1,
    "name": "Smartphone",
    "follows_count": 245,
    "is_followed_by_user": 1
}
```

#### 7.2 List of Most Followed Products with Follow Status for the User

```php
public function topFollowed()
{
    $products = Product::topFollowed(10)
        ->withIsFollowedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API to Toggle Follow

```php
public function toggleFollow($productId)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    
    $result = $product->toggleFollow([], $user);
    
    return response()->json([
        'success' => true,
        'is_followed' => $product->isUserFollow($user),
        'followers_count' => $product->getTotalFollowers(true)
    ]);
}
```

---

### 8. Backward Compatibility: Using Legacy Scopes

If you rely on legacy code that uses old scope names, you can continue using them, but it is recommended to gradually upgrade.

```php
// Legacy
$products = Product::addSortByCountFollows('DESC', 'follows_count', true)->get();

// New
$products = Product::addCountFollows('DESC', 'follows_count', true)->get();
```

```php
// Legacy
$products = Product::sortByLatestAtFollows('DESC', 'latest_follow_at', true, null, false, 'created_at')->get();

// New
$products = Product::sortByLatestFollow('DESC', 'latest_follow_at', true, null, false, 'created_at')->get();
```

---

### 9. Advanced Tips and Tricks

#### 9.1 Using `withTrashed` in Statistics to Include Soft-Deleted Follows

```php
$totalWithDeleted = $product->getTotalFollowers(null, null, true);
```

#### 9.2 Getting the Count of Non-Accepted Followers Only

```php
$pending = $product->getTotalFollowers(false);
```

#### 9.3 Using the `hasFollowers` Scope with a Variable Minimum Threshold

```php
$minCount = request('min_followers', 5);
$products = Product::hasFollowers($minCount, true)->get();
```

#### 9.4 Customizing the Column Name in Scopes

```php
$products = Product::withCountFollows('DESC', 'my_custom_column', true)->get();
echo $products->first()->my_custom_column;
```

#### 9.5 Combining `followedByUser` and `notFollowedByUser` in a Single Query (Using Union)

```php
$followed = Product::followedByUser()->get();
$notFollowed = Product::notFollowedByUser()->get();
$all = $followed->merge($notFollowed);
```

---

### 10. Advanced Use Cases in a Social System

#### 10.1 Suggesting Products to a User Based on Friends' Follows

```php
$user = AuthHelpers::getCurrentUser();

// Users followed by the current user
$followedUsers = $user->followedBy()->pluck('id');

// Products followed by those users
$suggestedProducts = Product::whereHas('follows', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers);
})->whereNotIn('id', $user->followedBy()->pluck('id')) // exclude what the user already follows
  ->withCountFollows('DESC', 'follows_count', true)
  ->limit(10)
  ->get();
```

#### 10.2 New Follow Notifications (When a Follow Is Accepted)

```php
// After accepting the follow
$follow = $product->updateFollow($followId, ['is_accepted' => true, 'accepted_at' => now()]);

if ($follow->wasChanged('is_accepted')) {
    // Send notification to the user
    Notification::send($follow->user, new FollowAcceptedNotification($product));
}
```

#### 10.3 Displaying Follower Count in a Product List with Lazy Loading

```php
$products = Product::withCountFollows('DESC', 'follows_count', true)
    ->paginate(20);
```

---

### Conclusion

The `FollowableModel` behavior provides a rich set of tools that enable you to build a complete follow system with ease. Through the examples presented, you can apply these features in your projects to create interactive user experiences, analyze user behavior, and improve application performance. Use the advanced scopes and statistical functions to reduce complexity and increase productivity.