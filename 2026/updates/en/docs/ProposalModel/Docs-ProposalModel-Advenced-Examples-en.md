# Advanced Practical Examples for Using the `ProposalModel` Behavior

This section provides a set of integrated examples illustrating how to use the `ProposalModel` behavior in real-world scenarios, from basic operations to complex queries and advanced statistics.

---

### 1. Basic Operations

#### 1.1 Adding Reports of Different Types

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano2\Proposals\Models\Proposal;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add a report about a product
$report = $product->addProposal([
    'type' => 'reports',
    'content' => 'Product does not match description',
    'is_active' => true,
], $user);

// Add a proposal
$proposal = $product->addProposal([
    'type' => 'proposals',
    'content' => 'I suggest adding installment payment feature',
], $user);

// Add a complaint
$complaint = $product->addProposal([
    'type' => 'complaints',
    'content' => 'Shipping delayed beyond the specified date',
    'is_active' => true,
], $user);

// Add a report with additional metadata (via `other_data` or `config_data`)
$report = $product->addProposal([
    'type' => 'reports',
    'content' => 'Unreasonable price',
    'other_data' => ['priority' => 'high', 'order_id' => 123],
], $user);
```

#### 1.2 Updating an Existing Report

```php
$user = AuthHelpers::getCurrentUser();
$report = $product->getUserProposal($user);

if ($report) {
    $product->updateProposal($report->id, [
        'content' => 'Content updated after verification',
        'is_active' => true,
    ]);
}
```

#### 1.3 Toggling Between Adding and Removing (Toggle)

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// If the user has sent a report, it deletes it; otherwise, it adds a new report
$product->toggleProposal([
    'type' => 'reports',
    'content' => 'New report',
], $user);
```

#### 1.4 Deleting a Report (Soft Delete)

```php
$user = AuthHelpers::getCurrentUser();
$report = $product->getUserProposal($user);

if ($report) {
    $product->deleteProposal($report->id);
}
```

---

### 2. Filtering by Users

#### 2.1 Retrieving Products that the Current User Has Reported (Target Side)

```php
// All products that the current user has reported (any type)
$myReportedProducts = Product::proposedByUser()->get();

// Only products that the current user has reported of type 'complaints'
$myComplaints = Product::proposedByUser(null, 'complaints')->get();
```

#### 2.2 Retrieving Products that the Current User Has Not Reported

```php
$unreportedProducts = Product::notProposedByUser()->paginate(20);
```

#### 2.3 Retrieving Users Who Have Sent Reports (User Side)

```php
// Users who have sent reports (any type)
$usersWhoReported = User::proposedToUser()->get();

// Users who have sent reports of type 'reports'
$usersWithReports = User::proposedToUser(null, 'reports')->get();
```

#### 2.4 Retrieving Products that a Specific User Has Reported, Loading Report Details

```php
$user = User::find(2);
$products = Product::proposedByUser($user)
    ->with(['all_proposals' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

---

### 3. Advanced Ordering and Ranking

#### 3.1 Ordering Products by Report Count (Most Reported)

```php
// Using the direct scope
$topProducts = Product::sortByCountProposals('DESC')->take(10)->get();

// Using the shortcut topProposed
$topProducts = Product::topProposed(10)->get();

// Ordering by report count of type 'complaints' only
$topComplaints = Product::sortByCountProposals('DESC', 'complaints')->get();
```

#### 3.2 Adding a Report Count Column to Results (Without Ordering)

```php
$products = Product::addCountProposals()->get();
foreach ($products as $product) {
    echo $product->name . ' - Report Count: ' . $product->proposals_count;
}
```

#### 3.3 Ordering by Latest Report (Last User Who Reported)

```php
// Order by the creation date of the latest report (most recent first)
$products = Product::sortByLatestProposal('DESC')->get();

// Using updated_at instead of created_at
$products = Product::sortByLatestProposal('DESC', 'latest_proposal_at', null, null, null, 'updated_at')->get();
```

#### 3.4 Adding the Latest Report Date Column with Ordering

```php
$products = Product::withLatestProposal('DESC', 'last_proposal_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - Latest Report: ' . $product->last_proposal_date;
}
```

#### 3.5 Ordering by Earliest Report (First Who Reported)

```php
$products = Product::sortByEarliestProposal('ASC')->get();
```

---

### 4. Statistics and Reports

#### 4.1 Total Number of Reports for a Specific Product

```php
$product = Product::find(1);

// All reports (all types)
$total = $product->getTotalProposals();

// Reports of type 'complaints' only
$totalComplaints = $product->getTotalProposals('complaints');

// Active reports of type 'reports'
$totalActiveReports = $product->getTotalProposals('reports', true);
```

#### 4.2 Distribution of Reports by User Type

```php
$counts = $product->getProposalsCountByType('reports');
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]

// Distribution of reports of type 'complaints'
$complaintsByType = $product->getProposalsCountByType('complaints');
```

#### 4.3 Retrieving a List of User Names Who Reported a Specific Product

```php
$users = $product->getProposersUsers('reports');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 Report Statistics within a Time Range (Using Scopes with where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// Products that received reports in the last week
$products = Product::whereHas('all_proposals', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. Combining Scopes for Complex Queries

#### 5.1 Most Reported Products that the Current User Has Also Reported

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topProposed(20)
    ->proposedByUser($user)
    ->withIsProposedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (Report Count: ' . $product->proposals_count . ') - ';
    echo $product->is_proposed_by_user ? 'You have reported' : 'You have not reported';
}
```

#### 5.2 Products with More Than 5 Reports of Type 'complaints', Ordered by Latest Report

```php
$products = Product::hasProposals(5, 'complaints')
    ->withLatestProposal('DESC', 'last_complaint_date')
    ->get();
```

#### 5.3 Active Products that the Current User Has Reported, with Report Count

```php
$products = Product::active()  // assuming an active scope exists in the model
    ->proposedByUser()
    ->withCountProposals('DESC', 'proposals_count')
    ->get();
```

#### 5.4 Products that the Current User Has Reported, Adding the Status Column (Will Always Be 1)

```php
$products = Product::proposedByUser()
    ->withIsProposedByUser()
    ->get();
// All products will have is_proposed_by_user = 1
```

#### 5.5 Products that Have Reports of Type 'reports' Only, Excluding Products with No Reports

```php
$products = Product::hasProposals(1, 'reports')
    ->sortByCountProposals('DESC', 'reports_count', 'reports')
    ->get();
```

---

### 6. Using Helper Functions in Templates (Twig/Blade Example)

#### 6.1 Displaying Product Report Count and Ability to Add a Report

```php
@php
$product = Product::find(1);
$hasReported = $product->isUserProposal(null, 'reports');
$reportCount = $product->getTotalProposals('reports');
@endphp

<div class="proposal-buttons">
    <button class="report-btn" data-type="reports" data-id="{{ $product->id }}">
        @if($hasReported)
            Cancel Report
        @else
            Report an Issue
        @endif
    </button>
    <span class="report-count">{{ $reportCount }} Reports</span>
</div>
```

#### 6.2 Displaying the List of Reporters on a Product Page

```php
$reporters = $product->getProposersUsers('reports');
?>

<div class="reporters-list">
    <h4>Top Reporters</h4>
    @foreach($reporters->take(5) as $user)
        <div class="reporter">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

#### 6.3 Displaying Buttons for Different Report Types

```php
@php
$product = Product::find(1);
$isReported = $product->isUserProposal(null, 'reports');
$isComplaint = $product->isUserProposal(null, 'complaints');
@endphp

<button class="report-btn" data-type="reports">
    {{ $isReported ? 'Cancel Report' : 'Report' }}
</button>
<button class="complaint-btn" data-type="complaints">
    {{ $isComplaint ? 'Cancel Complaint' : 'Complaint' }}
</button>
```

---

### 7. Examples for API Endpoints (JSON)

#### 7.1 Returning Product Details with Report Information

```php
public function show($id)
{
    $product = Product::withIsProposedByUser()
        ->withCountProposals('DESC', 'proposals_count')
        ->find($id);
    
    return response()->json($product);
}
```

Result:
```json
{
    "id": 1,
    "name": "Smartphone",
    "proposals_count": 245,
    "is_proposed_by_user": 1
}
```

#### 7.2 List of Most Reported Products with User Report Status Column

```php
public function topReported()
{
    $products = Product::topProposed(10, 'DESC', 'reports')
        ->withIsProposedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API to Add or Remove a Report

```php
public function toggleReport($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    $type = $request->input('type', 'reports');
    
    $result = $product->toggleProposal([
        'type' => $type,
        'content' => $request->input('content'),
    ], $user);
    
    return response()->json([
        'success' => true,
        'is_proposed' => $product->isUserProposal($user, $type),
        'proposals_count' => $product->getTotalProposals($type)
    ]);
}
```

---

### 8. Backward Compatibility: Using Old Scopes

If you rely on old code that uses previous scopes, you can continue using them, but it is recommended to gradually upgrade.

```php
// Old
$products = Product::sortByCountProposalsOld('DESC')->get();

// New
$products = Product::sortByCountProposals('DESC')->get();
```

```php
// Old
$products = Product::withSortByCountProposals('DESC', 'reports')->get();

// New
$products = Product::withCountProposals('DESC', 'proposals_count', 'reports')->get();
```

```php
// Old
$products = Product::sortByCreatedAtProposals('DESC', 'complaints')->get();

// New
$products = Product::sortByLatestProposal('DESC', 'latest_proposal_at', 'complaints')->get();
```

```php
// Old
$products = Product::whereHasProposal($user, 'reports')->get();

// New
$products = Product::proposedByUser($user, 'reports')->get();
```

---

### 9. Advanced Tips and Tricks

#### 9.1 Using the Advanced `scopeWhereHasProposalAdvanced` for Complex Conditions

```php
// Products that have reports of type 'reports' or 'complaints' (OR)
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'reports']
])->orWhereHasProposalAdvanced([
    'conditions' => ['type' => 'complaints']
])->get();

// Products that have reports of type 'reports' created in the last week
$products = Product::whereHasProposalAdvanced([
    'conditions' => function($q) {
        $q->where('type', 'reports')
          ->where('created_at', '>=', Carbon::now()->subWeek());
    }
])->get();

// Products that have reports of type 'reports' with a specific category (categories_id)
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => 'reports',
        'categories_id' => 5,
    ]
])->get();

// Using whereIn
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => ['whereIn', ['reports', 'complaints']],
        'is_active' => true,
    ]
])->get();

// Including soft-deleted reports
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => 'reports',
        'withTrashed' => true,
    ]
])->get();
```

#### 9.2 Searching for Reports that Have Been Replied To (reply_at field)

```php
$products = Product::whereHasProposalAdvanced([
    'conditions' => function($q) {
        $q->whereNotNull('reply_at');
    }
])->get();
```

#### 9.3 Controlling User and Target Sources via Settings

```php
// By default, `userSource` from settings is 'current_user' and `targetSource` is 'current_model'
// This can be changed in the call:
$users = User::whereHasProposalAdvanced([
    'side' => 'user',
    'userSource' => 'current_model', // use the current model as the user
])->get();
```

#### 9.4 Customizing the Column Name in Scopes

```php
$products = Product::withCountProposals('DESC', 'my_reports_column', 'reports')->get();
echo $products->first()->my_reports_column;
```

---

### 10. Advanced Use Cases in a Social System

#### 10.1 Suggesting Products to a User Based on Friends' Reports

```php
$user = AuthHelpers::getCurrentUser();

// Users followed by the current user (assuming a follow system exists)
$followedUsers = $user->followedBy()->pluck('id');

// Products that these users have reported of type 'complaints'
$suggestedProducts = Product::whereHas('all_proposals', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers)
      ->where('type', 'complaints');
})->whereNotIn('id', $user->proposedBy()->pluck('id')) // exclude products the user has reported
  ->withCountProposals('DESC', 'complaints_count', 'complaints')
  ->limit(10)
  ->get();
```

#### 10.2 New Report Notifications (When a User Reports a Product They Follow)

```php
// After adding the report
$report = $product->addProposal(['type' => 'reports', 'content' => '...'], $user);

// Send notification to the product owner
if ($product->owner_id != $user->id) {
    Notification::send($product->owner, new NewReportNotification($product, $user));
}
```

#### 10.3 Displaying Report Count in Product List with Lazy Loading

```php
$products = Product::withCountProposals('DESC', 'proposals_count')
    ->paginate(20);
```

#### 10.4 Analyzing User Behavior (Most Active Reporters)

```php
// Users who have sent more than 10 reports
$activeReporters = User::has('all_user_proposals', '>=', 10)->get();

// Products most reported by active users (type reports)
$topProducts = Product::whereHas('all_proposals', function($q) use ($activeReporters) {
    $q->whereIn('user_id', $activeReporters->pluck('id'))
      ->where('type', 'reports');
})->withCountProposals('DESC', 'reports_count', 'reports')
  ->limit(10)
  ->get();
```

#### 10.5 Creating a Report Priority System

```php
// Adding a report with high priority via `other_data`
$product->addProposal([
    'type' => 'reports',
    'content' => 'Critical',
    'other_data' => ['priority' => 'high'],
], $user);

// Retrieving reports with high priority
$highPriorityReports = Proposal::where('type', 'reports')
    ->whereJsonContains('other_data', ['priority' => 'high'])
    ->get();
```

---

### Conclusion

The `ProposalModel` behavior provides a rich set of tools that enable you to easily build an integrated system for reports, proposals, and complaints. Through the examples provided, you can apply these features in your projects to create interactive user experiences, analyze user behavior, and improve application performance. Use the advanced scopes and statistical functions to reduce complexity and increase productivity.