# Advanced Practical Examples for Using the `ReactionableModel` Behavior

This section provides a set of integrated examples illustrating how to use the `ReactionableModel` behavior in real-world scenarios, from basic operations to complex queries and advanced statistics.

---

### 1. Basic Operations

#### 1.1 Adding Reactions of Different Types

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Reaction;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add a reaction of type 'heart'
$reaction = Reaction::add($product, $user, 'heart');

// Add a reaction of type 'wow'
$reaction = Reaction::add($product, $user, 'wow', ['notes' => 'Amazing!']);

// Add a reaction of type 'like'
$reaction = Reaction::add($product, $user, 'like');

// Add a reaction with additional metadata
$reaction = Reaction::add($product, $user, 'angry', [
    'reason' => 'Low quality',
    'priority' => 'high'
]);
```

#### 1.2 Removing a Reaction

```php
// Remove the reaction (any type)
Reaction::remove($product, $user);

// Remove a reaction of type 'heart' only
Reaction::remove($product, $user, 'heart');
```

#### 1.3 Toggling Reactions (Toggle)

```php
// If the user has already reacted, it removes the reaction; otherwise, it adds a reaction
$result = Reaction::toggle($product, $user, 'heart');

// Toggle a reaction of type 'wow'
$result = Reaction::toggle($product, $user, 'wow');
```

#### 1.4 Getting the Reaction Record for the Current User

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

$reaction = $product->getReacted($user);
if ($reaction) {
    echo "Reaction type: " . $reaction->value; // heart, wow, like, angry
    echo "Additional data: " . $reaction->metadata['reason'] ?? '';
}
```

---

### 2. Filtering by Users

#### 2.1 Retrieving All Products that the Current User Has Reacted To

```php
// All reactions (any type)
$myReactedProducts = Product::reactedByUser()->get();

// Only reactions of type 'heart'
$myHearts = Product::reactedByUser(null, 'heart')->get();

// Only reactions of type 'wow'
$myWow = Product::reactedByUser(null, 'wow')->get();
```

#### 2.2 Retrieving Products that the Current User Has Not Reacted To (for Suggestions)

```php
$suggestedProducts = Product::notReactedByUser()->paginate(20);
```

#### 2.3 Retrieving Products that a Specific User Has Reacted To, Loading Reaction Details

```php
$user = User::find(2);
$products = Product::reactedByUser($user)
    ->with(['reactions' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

#### 2.4 Using the Old `whereHasReaction` Functions with Control over Behavior When No User Exists

```php
// Default behavior (returns no results when no user exists)
$products = Product::whereHasReaction(null, 'heart')->get();

// Disable forcing (returns all products when no user exists)
$products = Product::whereHasReaction(null, 'heart', false)->get();
```

---

### 3. Advanced Ordering and Ranking

#### 3.1 Ordering Products by Reaction Count (Most Reacted)

```php
// Using the direct scope
$topProducts = Product::sortByCountReactions('DESC', 'reactions_count')->take(10)->get();

// Using the shortcut topReacted
$topProducts = Product::topReacted(10)->get();

// Ordering by reaction count of type 'heart' only
$topHearts = Product::sortByCountReactions('DESC', 'hearts_count', 'heart')->get();

// Ordering by reaction count of type 'wow' only
$topWow = Product::sortByCountReactions('DESC', 'wow_count', 'wow')->get();
```

#### 3.2 Adding a Reaction Count Column to Results (Without Ordering)

```php
$products = Product::addCountReactions()->get();
foreach ($products as $product) {
    echo $product->name . ' - Reaction Count: ' . $product->reactions_count;
}
```

#### 3.3 Ordering by Latest Reaction (Last User Who Reacted)

```php
// Order by the creation date of the latest reaction (most recent first)
$products = Product::sortByLatestReaction('DESC')->get();

// Using updated_at instead of created_at
$products = Product::sortByLatestReaction('DESC', 'latest_reaction_at', null, 'updated_at')->get();
```

#### 3.4 Adding the Latest Reaction Date Column with Ordering

```php
$products = Product::withLatestReaction('DESC', 'last_reaction_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - Latest Reaction: ' . $product->last_reaction_date;
}
```

#### 3.5 Ordering by Earliest Reaction (First Who Reacted)

```php
$products = Product::sortByEarliestReaction('ASC')->get();
```

---

### 4. Statistics and Reports

#### 4.1 Total Number of Reactions for a Specific Product

```php
$product = Product::find(1);

// All reactions (all types)
$total = $product->getTotalReactions();

// Reactions of type 'heart' only
$totalHearts = $product->getTotalReactions('heart');

// Reactions of type 'wow' only
$totalWow = $product->getTotalReactions('wow');
```

#### 4.2 Distribution of Reactions by User Type

```php
$counts = $product->getReactionsCountByType('heart');
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]

// Distribution of 'wow' reactions
$wowCounts = $product->getReactionsCountByType('wow');
```

#### 4.3 Retrieving a List of User Names Who Reacted to a Specific Product

```php
$users = $product->getReactorsUsers('heart');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 Reaction Statistics within a Time Range (Using Scopes with where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// Products that were reacted to in the last week
$products = Product::whereHas('reactions', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. Combining Scopes for Complex Queries

#### 5.1 Most Reacted Products that the Current User Has Also Reacted To

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topReacted(20)
    ->reactedByUser($user)
    ->withIsReactedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (Reaction Count: ' . $product->reactions_count . ') - ';
    echo $product->is_reacted_by_user ? 'You reacted' : 'You did not react';
}
```

#### 5.2 Products with More Than 5 Reactions of Type 'heart', Ordered by Latest Reaction

```php
$products = Product::hasReactions(5, 'heart')
    ->withLatestReaction('DESC', 'last_heart_date')
    ->get();
```

#### 5.3 Active Products that the Current User Has Reacted To, with Reaction Count

```php
$products = Product::active()  // assuming an active scope exists in the model
    ->reactedByUser()
    ->withCountReactions('DESC', 'reactions_count')
    ->get();
```

#### 5.4 Products that the Current User Has Reacted To, Adding the Status Column (Will Always Be 1)

```php
$products = Product::reactedByUser()
    ->withIsReactedByUser()
    ->get();
// All products will have is_reacted_by_user = 1
```

#### 5.5 Products that Have Reactions of Type 'wow' Only, Excluding Products with No Reactions

```php
$products = Product::hasReactions(1, 'wow')
    ->sortByCountReactions('DESC', 'wow_count', 'wow')
    ->get();
```

---

### 6. Using Helper Functions in Templates (Twig/Blade Example)

#### 6.1 Displaying Multiple Reaction Buttons

```php
@php
$product = Product::find(1);
$isHeart = $product->isReacted(null, 'heart');
$isWow = $product->isReacted(null, 'wow');
$heartCount = $product->getTotalReactions('heart');
$wowCount = $product->getTotalReactions('wow');
@endphp

<button class="reaction-btn" data-type="heart" data-id="{{ $product->id }}">
    @if($isHeart)
        ❤️ Remove
    @else
        🤍 Heart
    @endif
</button>
<button class="reaction-btn" data-type="wow" data-id="{{ $product->id }}">
    @if($isWow)
        😲 Remove
    @else
        😲 Wow
    @endif
</button>
<span class="heart-count">{{ $heartCount }}</span>
<span class="wow-count">{{ $wowCount }}</span>
```

#### 6.2 Displaying the List of Users Who Reacted on a Product Page

```php
$reactors = $product->getReactorsUsers('heart');
?>

<div class="reactors-list">
    <h4>Top Reactors (Heart)</h4>
    @foreach($reactors->take(5) as $user)
        <div class="reactor">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

---

### 7. Examples for API Endpoints (JSON)

#### 7.1 Returning Product Details with Reaction Information

```php
public function show($id)
{
    $product = Product::withIsReactedByUser()
        ->withCountReactions('DESC', 'reactions_count')
        ->find($id);
    
    return response()->json($product);
}
```

Result:
```json
{
    "id": 1,
    "name": "Smartphone",
    "reactions_count": 245,
    "is_reacted_by_user": 1
}
```

#### 7.2 List of Most Reacted Products with User Reaction Status Column

```php
public function topReacted()
{
    $products = Product::topReacted(10)
        ->withIsReactedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API to Add or Remove a Reaction

```php
public function toggleReaction($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    $value = $request->input('value', 'heart');
    
    $result = Reaction::toggle($product, $user, $value);
    
    return response()->json([
        'success' => true,
        'is_reacted' => $product->isReacted($user, $value),
        'reactions_count' => $product->getTotalReactions($value)
    ]);
}
```

---

### 8. Backward Compatibility: Using Old Scopes

If you rely on old code that uses previous scopes, you can continue using them, but it is recommended to gradually upgrade.

```php
// Old
$products = Product::sortByCountReactionsOld('DESC')->get();

// New
$products = Product::sortByCountReactions('DESC')->get();
```

```php
// Old
$products = Product::withSortByCountReactions('DESC', 'heart')->get();

// New
$products = Product::withCountReactions('DESC', 'reactions_count', 'heart')->get();
```

```php
// Old
$products = Product::sortByCreatedAtReactions('DESC', 'heart')->get();

// New
$products = Product::sortByLatestReaction('DESC', 'latest_reaction_at', 'heart')->get();
```

```php
// Old
$products = Product::whereHasReaction($user, 'heart')->get();

// New
$products = Product::reactedByUser($user, 'heart')->get();
```

---

### 9. Advanced Tips and Tricks

#### 9.1 Using Multiple Reaction Types

```php
// Set allowed types in settings
// config/nano/markable.php
return [
    'allowed_values' => [
        'reaction' => ['like', 'heart', 'wow', 'angry', 'sad'],
    ],
];

// Query products reacted to as type 'wow'
$wowProducts = Product::reactedByUser(null, 'wow')->get();
```

#### 9.2 Getting Reaction Count for Each Type

```php
$product = Product::find(1);

$reactionsByType = [
    'heart' => $product->getTotalReactions('heart'),
    'wow' => $product->getTotalReactions('wow'),
    'like' => $product->getTotalReactions('like'),
    'angry' => $product->getTotalReactions('angry'),
];
```

#### 9.3 Using the `hasReactions` Scope with a Variable Minimum Count

```php
$minCount = request('min_reactions', 5);
$products = Product::hasReactions($minCount, 'heart')->get();
```

#### 9.4 Customizing the Column Name in Scopes

```php
$products = Product::withCountReactions('DESC', 'my_heart_column', 'heart')->get();
echo $products->first()->my_heart_column;
```

#### 9.5 Combining `reactedByUser` and `notReactedByUser` in One Query (Using union)

```php
$reacted = Product::reactedByUser()->get();
$notReacted = Product::notReactedByUser()->get();
$all = $reacted->merge($notReacted);
```

---

### 10. Advanced Use Cases in a Social System

#### 10.1 Suggesting Products to a User Based on Friends' Reactions

```php
$user = AuthHelpers::getCurrentUser();

// Users followed by the current user (assuming a follow system exists)
$followedUsers = $user->followedBy()->pluck('id');

// Products that these users have reacted to (type heart)
$suggestedProducts = Product::whereHas('reactions', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers)
      ->where('value', 'heart');
})->whereNotIn('id', $user->reactedBy()->pluck('id')) // exclude products the user has reacted to
  ->withCountReactions('DESC', 'hearts_count', 'heart')
  ->limit(10)
  ->get();
```

#### 10.2 New Reaction Notifications (When a User Reacts to a Product)

```php
// After adding the reaction
$reaction = Reaction::add($product, $user, 'heart');

// Send notification to the product owner
if ($product->owner_id != $user->id) {
    Notification::send($product->owner, new NewReactionNotification($product, $user, 'heart'));
}
```

#### 10.3 Displaying Reaction Count in Product List with Lazy Loading

```php
$products = Product::withCountReactions('DESC', 'reactions_count')
    ->paginate(20);
```

#### 10.4 Analyzing User Behavior (Most Active Reactors)

```php
// Users who have reacted to more than 10 products
$activeUsers = User::has('reactions', '>=', 10)->get();

// Products most reacted to by active users (type heart)
$topProducts = Product::whereHas('reactions', function($q) use ($activeUsers) {
    $q->whereIn('user_id', $activeUsers->pluck('id'))
      ->where('value', 'heart');
})->withCountReactions('DESC', 'hearts_count', 'heart')
  ->limit(10)
  ->get();
```

#### 10.5 Creating a Sentiment Analysis System (Positive/Negative Reactions)

```php
$product = Product::find(1);

$positiveReactions = ['heart', 'wow', 'like'];
$negativeReactions = ['angry', 'sad'];

$positiveCount = 0;
$negativeCount = 0;

foreach ($positiveReactions as $type) {
    $positiveCount += $product->getTotalReactions($type);
}
foreach ($negativeReactions as $type) {
    $negativeCount += $product->getTotalReactions($type);
}

$total = $positiveCount + $negativeCount;
$positiveRatio = $total > 0 ? round(($positiveCount / $total) * 100, 1) : 0;

echo "Positive reaction ratio: {$positiveRatio}%";
```

---

### Conclusion

The `ReactionableModel` behavior provides a rich set of tools that enable you to easily build an integrated reactions system. Through the examples provided, you can apply these features in your projects to create interactive user experiences, analyze user sentiment, and improve application performance. Use the advanced scopes and statistical functions to reduce complexity and increase productivity.
