# Advanced Practical Examples for Using the `VisitModel` Behavior

This section provides a collection of integrated examples demonstrating how to use the `VisitModel` behavior in real-world scenarios, from basic operations to complex queries and advanced statistics.

---

### 1. Basic Operations

#### 1.1 Recording a New Visit with Custom Data

```php
use Nano2\Visitors\Classes\Manager;
use Nano\AuthApi\Classes\AuthHelpers;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Record a regular visit (type add, process in)
Manager::createVisits([
    'visitable' => $product,
    'user' => $user,
    'type' => 'add',
    'process_type' => 'in',
    'event_type' => 'product_view',
    'visits' => 1,
    'views' => 1,
]);

// Record a visit with additional data (e.g., visit reason in other_data field)
Manager::createVisits([
    'visitable' => $product,
    'user' => $user,
    'type' => 'add',
    'process_type' => 'in',
    'event_type' => 'special_offer',
    'visits' => 1,
    'views' => 2,
    'other_data' => json_encode(['source' => 'email_campaign']),
]);
```

#### 1.2 Updating Visit Statistics (Adding Views or Visits)

```php
$product = Product::find(1);
$user = AuthHelpers::getCurrentUser();

// Retrieve the current visit record
$visit = $product->getVisit($user);

if ($visit) {
    // Increase view count
    $visit->views += 1;
    $visit->save();
}
```

#### 1.3 Deleting a Visit Record (Soft Delete)

```php
$visit = Visit::find($visitId);
if ($visit) {
    $visit->delete();
}
```

---

### 2. Filtering by Users

#### 2.1 Retrieving All Products Visited by the Current User

```php
$myVisitedProducts = Product::visitedByUser()->get();

// Including only inactive visits
$myVisitedProducts = Product::visitedByUser(null, false)->get();

// Including soft-deleted visits
$myVisitedProducts = Product::visitedByUser(null, null, true)->get();
```

#### 2.2 Retrieving Products Not Visited by the Current User (to Suggest Them)

```php
$suggestedProducts = Product::notVisitedByUser()->paginate(20);
```

#### 2.3 Using the Legacy `whereHasVisit` Scope (for Backward Compatibility)

```php
$products = Product::whereHasVisit($user)->get();
```

#### 2.4 Retrieving Products Visited by a Specific User, Loading Visit Details

```php
$user = User::find(2);
$products = Product::visitedByUser($user)
    ->with(['visits' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

---

### 3. Advanced Sorting and Ranking

#### 3.1 Sorting Products by Visit Count (Number of Records)

```php
// Using the direct scope
$topProducts = Product::sortByCountVisits('DESC', 'visits_count', true)->take(10)->get();

// Using the topVisited shortcut
$topProducts = Product::topVisited(10)->get();

// Sort by active visits only of a specific type
$topActive = Product::sortByCountVisits('DESC', 'visits_count', true, false, 'add')->get();
```

#### 3.2 Sorting Products by Total Visits (Field `visits`)

```php
$products = Product::sortBySumVisits('DESC', 'total_visits', true)->get();
```

#### 3.3 Sorting Products by Total Views (Field `views`)

```php
$products = Product::sortBySumViews('DESC', 'total_views')->get();
```

#### 3.4 Sorting by Latest Visit Date (Newest First)

```php
$products = Product::sortByLatestVisit('DESC')->get();
```

#### 3.5 Adding a Latest Visit Date Column with Sorting

```php
$products = Product::withLatestVisit('DESC', 'last_visit_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - Latest visit: ' . $product->last_visit_date;
}
```

---

### 4. Statistics and Reports

#### 4.1 Total Visits and Views for a Specific Product

```php
$product = Product::find(1);

// All visits
$totalVisits = $product->getTotalVisits();

// Only active visits
$totalActiveVisits = $product->getTotalVisits(true);

// Active visits of type 'add' only
$totalAddVisits = $product->getTotalVisits(true, false, 'add');

// Total views
$totalViews = $product->getTotalViews(true);
```

#### 4.2 Distribution of Visits by User Type

```php
$counts = $product->getVisitsCountByType(true);
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]
```

#### 4.3 Fetching a List of User Names Who Visited a Specific Product

```php
$users = $product->getVisitorsUsers(true);
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 Visit Statistics Within a Time Range (Using Scopes with where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// Products that were visited during the last week
$products = Product::whereHas('visits', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. Combining Scopes for Complex Queries

#### 5.1 Most Visited Products (by Record Count) That the Current User Also Visited

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topVisited(20)
    ->visitedByUser($user)
    ->withIsVisitedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (Visit count: ' . $product->visits_count . ') - ';
    echo $product->is_visited_by_user ? 'You visited it' : 'You did not visit it';
}
```

#### 5.2 Products with More than 5 Active Visits, Sorted by Latest Visit (with Column Added)

```php
$products = Product::hasVisits(5, true)
    ->withLatestVisit('DESC', 'last_visit_date')
    ->get();
```

#### 5.3 Active Products Visited by the Current User, with Total Visits Column Added

```php
$products = Product::active()  // assuming an active scope exists on the model
    ->visitedByUser()
    ->withSumVisits('DESC', 'total_visits')
    ->get();
```

#### 5.4 Products Visited by the Current User of a Specific Type (add) for a Specific Event

```php
$products = Product::visitedByUser(null, null, false, 'add', null, 'contest')->get();
```

#### 5.5 Products with No Active Visits, Sorted by Name

```php
$products = Product::hasNoVisits(true)->orderBy('name')->get();
```

---

### 6. Using Helper Functions in Templates (Example Twig/Blade)

#### 6.1 Displaying Visit and View Counts for a Product

```php
@php
$product = Product::find(1);
$totalVisits = $product->getTotalVisits(true);
$totalViews = $product->getTotalViews(true);
$isVisited = $product->is_visits;
@endphp

<div class="product-stats">
    <span>Visits: {{ $totalVisits }}</span>
    <span>Views: {{ $totalViews }}</span>
    <span>{{ $isVisited ? 'Visited' : 'Not visited' }}</span>
</div>
```

#### 6.2 Displaying a List of Most Visited Products with an Indicator for the Current User

```php
$products = Product::topVisited(10)
    ->withIsVisitedByUser()
    ->get();
?>

<div class="top-products">
    @foreach($products as $product)
        <div class="product-item">
            <h3>{{ $product->name }}</h3>
            <p>Visits: {{ $product->visits_count }}</p>
            <span class="visit-status">
                {{ $product->is_visited_by_user ? 'Visited' : 'Not visited' }}
            </span>
        </div>
    @endforeach
</div>
```

---

### 7. API Examples (JSON)

#### 7.1 Returning Product Details with Visit Statistics

```php
public function show($id)
{
    $product = Product::withIsVisitedByUser()
        ->withSumVisits('DESC', 'total_visits')
        ->withSumViews('DESC', 'total_views')
        ->find($id);
    
    return response()->json($product);
}
```

Output:
```json
{
    "id": 1,
    "name": "Smartphone",
    "total_visits": 245,
    "total_views": 1230,
    "is_visited_by_user": 1
}
```

#### 7.2 List of Most Visited Products with Visit Status for the User

```php
public function topVisited()
{
    $products = Product::topVisited(10)
        ->withIsVisitedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API to Record a Visit (via Manager)

```php
public function recordVisit($productId)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    
    $result = Manager::createVisits([
        'visitable' => $product,
        'user' => $user,
        'type' => 'add',
        'process_type' => 'in',
        'event_type' => 'api_view',
        'visits' => 1,
        'views' => 1,
    ]);
    
    return response()->json([
        'success' => $result['status'],
        'message' => $result['message'],
    ]);
}
```

---

### 8. Backward Compatibility: Using Legacy Scopes

If you rely on legacy code that uses old scope names, you can continue using them, but it is recommended to gradually upgrade.

```php
// Legacy
$products = Product::sortByCountVisitsOld('DESC')->get();

// New
$products = Product::sortByCountVisits('DESC')->get();
```

```php
// Legacy
$products = Product::withSortBySumVisits('DESC')->get();

// New
$products = Product::withSumVisits('DESC')->get();
```

```php
// Legacy (whereHasVisit)
$products = Product::whereHasVisit($user)->get();

// New (visitedByUser)
$products = Product::visitedByUser($user)->get();
```

---

### 9. Advanced Tips and Tricks

#### 9.1 Using `withTrashed` in Statistics to Include Soft-Deleted Visits

```php
$totalWithDeleted = $product->getTotalVisits(null, true);
```

#### 9.2 Getting the Number of Visits by Process Type

```php
$inVisits = $product->getTotalVisits(true, false, null, 'in');
$outVisits = $product->getTotalVisits(true, false, null, 'out');
```

#### 9.3 Using the `hasVisits` Scope with a Variable Minimum Threshold

```php
$minCount = request('min_visits', 5);
$products = Product::hasVisits($minCount, true)->get();
```

#### 9.4 Customizing the Column Name in Scopes

```php
$products = Product::withCountVisits('DESC', 'my_visits_column', true)->get();
echo $products->first()->my_visits_column;
```

#### 9.5 Combining `visitedByUser` and `notVisitedByUser` in a Single Query (Using Union)

```php
$visited = Product::visitedByUser()->get();
$notVisited = Product::notVisitedByUser()->get();
$all = $visited->merge($notVisited);
```

---

### 10. Advanced Use Cases in a Social System

#### 10.1 Suggesting Products to a User Based on Friends' Visits

```php
$user = AuthHelpers::getCurrentUser();

// Users followed by the current user (assuming a follow system exists)
$followedUsers = $user->followedBy()->pluck('id');

// Products visited by those users
$suggestedProducts = Product::whereHas('visits', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers);
})->whereNotIn('id', $user->visitedBy()->pluck('id')) // exclude what the user already visited
  ->withCountVisits('DESC', 'visits_count', true)
  ->limit(10)
  ->get();
```

#### 10.2 New Visit Notifications for a Product the User Follows

```php
// After recording a visit to a product
$product = Product::find($productId);
$visitor = AuthHelpers::getCurrentUser();

// Send notification to users who follow the product (assuming a follow system exists)
$followers = $product->followedBy()->get();
foreach ($followers as $follower) {
    if ($follower->id != $visitor->id) {
        Notification::send($follower, new ProductVisitedNotification($product, $visitor));
    }
}
```

#### 10.3 Displaying Visit and View Counts in a Product List with Lazy Loading

```php
$products = Product::withCountVisits('DESC', 'visits_count', true)
    ->withSumVisits('DESC', 'total_visits')
    ->withSumViews('DESC', 'total_views')
    ->paginate(20);
```

#### 10.4 Analyzing User Behavior (Most Engaged Users)

```php
// Users who have visited more than 10 products
$activeUsers = User::has('visits', '>=', 10)->get();

// Most viewed products by active users
$topProducts = Product::whereHas('visits', function($q) use ($activeUsers) {
    $q->whereIn('user_id', $activeUsers->pluck('id'));
})->withSumViews('DESC', 'total_views')
  ->limit(10)
  ->get();
```

---

### Conclusion

The `VisitModel` behavior provides a rich set of tools that enable you to build a complete visit and view tracking system with ease. Through the examples presented, you can apply these features in your projects to create interactive user experiences, analyze user behavior, and improve application performance. Use the advanced scopes and statistical functions to reduce complexity and increase productivity.