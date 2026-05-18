# Documentation for `ProposalModel` Behavior

---

## 1. Introduction

`ProposalModel` is an advanced behavior that adds a polymorphic system for proposals, reports, and complaints to any model in NanoSoft applications. This behavior allows users to send reports, proposals, or complaints about various objects (e.g., products, articles, users), with full support for both sides: the **target** against which the report is sent, and the **user** who sends the report. The behavior comes equipped with a rich set of scopes and helper functions that facilitate building sophisticated reporting systems with flexibility and high performance.

### Key Benefits

- **Polymorphic Reporting System** – Any model can be made reportable (as a target) or capable of sending reports (as a user).
- **Support for Multiple Types** – Reports can be distinguished via the `type` field (`proposals`, `reports`, `complaints`).
- **Flexibility in Status** – Support for `is_active` (report activity) and `deleted_at` (soft delete) fields.
- **Advanced Ordering Scopes** – Order objects by report count, latest report, or earliest report, with filtering by type and activity.
- **Filtering Scopes** – Filter objects that the user has reported (as target), that have sent a report (as user), that have reports, or that have no reports.
- **Adding a Status Column for the Current User** – `is_proposed_by_user` (did the user send a report for the object) and `is_proposed_to_user` (is the object the target of a report sent by the user).
- **Advanced Scope** (`scopeWhereHasProposalAdvanced`) – Provides full flexibility for filtering by side (target/user), relationship type (`has`, `orHas`, `doesntHave`, `orDoesntHave`), flexible conditions (array or closure), and support for `withTrashed`/`onlyTrashed`.
- **Helper Statistical Functions** – To get total reports, distribution by user type, and the list of reporting users.
- **Consistent Interface** – Functions similar to FavoriteableModel, LikeableModel, ReactionableModel, BookmarkableModel behaviors for a unified development experience.
- **Performance Improvement** – Use of efficient subqueries and avoiding full relationship loading where possible.
- **Backward Compatibility** – Retention of old scope names (`sortByCountProposalsOld`, `withSortByCountProposals`, `whereHasProposal`, etc.) to ensure existing code is not broken.

---

## 2. Usage Requirements

1. Existence of the `Nano2\Proposals\Models\Proposal` model (representing the report record) with the `tss_proposals_proposals` table.
2. The model to be made reportable must be an Eloquent model (or `October\Rain\Database\Model` in the case of OctoberCMS).
3. Users (e.g., `User` model) must be able to use the reporting system (i.e., they implement the `Authenticatable` interface).
4. Optional: Use `SoftDeletes` in the `Proposal` model to benefit from the `withTrashed` option.

---

## 3. Adding the Behavior to a Model

The behavior can be added to any model using the `$implement` property (in the NanoSoft context). The behavior includes a Trait containing the scopes and helper functions, so after adding it, the model can use all features.

### Example: Making the `Product` Model Reportable (as Target) and Capable of Sending Reports (as User)

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano2\Proposals\Behaviors\ProposalModel;

class Product extends Model
{
    public $implement = [
        ProposalModel::class
    ];
}
```

**Note:** If the model uses the `ExtensionBase` system (as is the case in NanoSoft applications), adding it is done via `$implement` as shown.

---

## 4. Core Structure

After adding the behavior, the model has the following relationships:

| Relationship | Description |
|--------------|-------------|
| `all_proposals` | All report records (received by the object as target) |
| `proposals` | Reports of type `proposals` only (received) |
| `reports` | Reports of type `reports` only (received) |
| `complaints` | Reports of type `complaints` only (received) |
| `all_user_proposals` | All report records (sent by the object as user) |
| `user_proposals` | Sent reports of type `proposals` |
| `user_reports` | Sent reports of type `reports` |
| `user_complaints` | Sent reports of type `complaints` |

Dynamic computed properties are also available (added automatically):

- `$product->proposals_count` – Number of reports (can pass `type` as a parameter).
- `$product->is_proposed_by_user()` – Whether the current user has reported this object (as target).
- `$product->is_proposed_to_user()` – Whether this object (as user) has sent a report.
- `$product->getUserProposal()` – Returns the report record for the current user (as target).

---

## 5. Available Functions and Scopes

### 5.1 Subquery Building Functions (Internal)

#### `protected function buildProposalCountSubQuery($type = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Builds a subquery to count the number of report records (COUNT) for the current object as target.
- **Parameters**:
  - `$type` (string|null): Filter by report type (`proposals`, `reports`, `complaints`).
  - `$onlyActive` (bool|null): `true` for active reports only, `false` for inactive only, `null` for all.
  - `$withTrashed` (bool): Include soft-deleted reports.

#### `protected function buildProposalLatestDateSubQuery(...)` – Uses `MAX(field)` to get the latest report date.
#### `protected function buildProposalEarliestDateSubQuery(...)` – Uses `MIN(field)` to get the earliest report date.

### 5.2 Report Count Scopes (COUNT)

#### `scopeAddCountProposals(Builder $query, $orderDirection = 'DESC', $columnName = 'proposals_count', $type = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Adds a column containing the number of report records to the query result (without ordering).
- **Example**:
  ```php
  $products = Product::addCountProposals()->get();
  foreach ($products as $product) {
      echo $product->proposals_count;
  }
  ```

#### `scopeSortByCountProposals(...)` – Orders results by report count (without adding the column).
#### `scopeWithCountProposals(...)` – Combines `addCountProposals` and `sortByCountProposals` (adds column and ordering together).

**Example**:
```php
$products = Product::withCountProposals('DESC', 'proposals_count', 'reports')->get(); // reports of type 'reports' only
```

### 5.3 Report Date Scopes

#### `scopeAddLatestProposal(...)` – Adds a column with the latest report date (optionally specifying the field via `$field`).
#### `scopeSortByLatestProposal(...)` – Orders by the latest report date.
#### `scopeWithLatestProposal(...)` – Adds the column and ordering together.

**Example**:
```php
$products = Product::withLatestProposal('DESC', 'last_proposal', 'complaints')->get();
```

#### `scopeAddEarliestProposal(...)` – Adds a column with the earliest report date.
#### `scopeSortByEarliestProposal(...)` – Orders by the earliest report date.
#### `scopeWithEarliestProposal(...)` – Adds the column and ordering together.

### 5.4 Filtering Scopes by User (Target Side)

#### `scopeProposedByUser(Builder $query, $user = null, $type = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Filters objects that the specified user has reported (as target). If `$user` is not passed, the current user is used.
- **Example**:
  ```php
  // Products that the current user has reported
  $products = Product::proposedByUser()->get();
  
  // Products that a specific user has reported of type 'complaints'
  $products = Product::proposedByUser($user, 'complaints')->get();
  ```

#### `scopeNotProposedByUser(...)` – Filters objects that the user has **not** reported.

### 5.5 Filtering Scopes by User (Sender Side)

#### `scopeProposedToUser(Builder $query, $user = null, $type = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Filters objects that have sent a report (as user). If `$user` is not passed, the current user is used.
- **Example**:
  ```php
  // Users who have sent reports
  $users = User::proposedToUser()->get();
  
  // Users who have sent reports of type 'reports'
  $users = User::proposedToUser(null, 'reports')->get();
  ```

#### `scopeNotProposedToUser(...)` – Filters objects that have **not** sent a report.

### 5.6 Filtering Scopes by Existence of Reports

#### `scopeHasProposals(Builder $query, $minCount = 1, $type = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Filters objects that have at least `$minCount` reports (with filtering options).
- **Example**:
  ```php
  // Products that have at least 5 reports of type 'complaints'
  $products = Product::hasProposals(5, 'complaints')->get();
  ```

#### `scopeHasNoProposals(...)` – Filters objects that have no reports.

### 5.7 Scopes for Adding Status Column for Current User

#### `scopeWithIsProposedByUser(Builder $query, $user = null, $columnName = 'is_proposed_by_user', $type = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Adds a boolean column (`1` or `0`) to the query result indicating whether the specified user has sent a report for the object.
- **Example**:
  ```php
  $products = Product::withIsProposedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_proposed_by_user) {
          echo "You have reported this product";
      }
  }
  ```

#### `scopeWithIsProposedToUser(...)` – Adds a column indicating whether the specified user is the target of a report sent by the object.

### 5.8 Advanced Scope `scopeWhereHasProposalAdvanced`

This is the most powerful and flexible scope, allowing filtering by reports with unlimited options. It takes an options array containing:

| Option | Type | Description |
|--------|------|-------------|
| `side` | string | `target` (reports received by the object) or `user` (reports sent from the object). Default `target`. |
| `relation` | string | The relationship name to use (determined automatically if not passed). |
| `type` | string | Relationship type: `has`, `orHas`, `doesntHave`, `orDoesntHave`. Default `has`. |
| `conditions` | array\|callable | Additional conditions on the reports table. Can be an array of fields and values, a closure, or an array supporting advanced queries (e.g., `['whereIn', [1,2,3]]`). |
| `user` | Model\|null | The specified user (for user side). |
| `target` | Model\|null | The specified target (for target side). |
| `getUserFromHelpers` | bool | Whether to try to fetch the current user if `user = null`? Default `true`. |
| `getTargetFromHelpers` | bool | Whether to try to fetch the target from the current model if `target = null`? Default `true`. |
| `userSource` | string | Source for the user: `'current_user'` (from AuthHelpers) or `'current_model'` (the current model). Reads from settings. |
| `targetSource` | string | Source for the target: `'current_user'` or `'current_model'`. Reads from settings. |
| `isForceUser` | bool\|null | When user is absent: `true` adds an impossible condition (no results), `false` adds no condition. |
| `isForceTarget` | bool\|null | When target is absent: `true` adds an impossible condition, `false` adds no condition. |
| `useUserConditions` | bool\|array | Apply user conditions (`user_id`, `user_type`). Can be `true` (all), `false` (none), or an array containing the required field names (e.g., `['user_type']`). |
| `useTargetConditions` | bool\|array | Apply target conditions (`target_id`, `target_type`). |

**Example**:
```php
// Products that have reports of type 'reports' (any user)
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'reports']
])->get();

// Products that have active reports of type 'complaints' from the current user
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'complaints', 'is_active' => true]
])->get();

// Products that have no reports of type 'proposals'
$products = Product::whereHasProposalAdvanced([
    'type' => 'doesntHave',
    'conditions' => ['type' => 'proposals']
])->get();

// Using a closure
$products = Product::whereHasProposalAdvanced([
    'conditions' => function($q) {
        $q->where('type', 'reports')
          ->where('created_at', '>=', Carbon::now()->subWeek());
    }
])->get();
```

### 5.9 Special Scopes

#### `scopeTopProposed(Builder $query, $limit = 10, $orderDirection = 'DESC', $type = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Retrieves the objects with the most reports (by report count), with a limit.
- **Example**:
  ```php
  $topProducts = Product::topProposed(10, 'DESC', 'complaints')->get();
  ```

### 5.10 Helper Statistical Functions

#### `getTotalProposals($type = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Total number of reports (sum of `COUNT`) according to filtering options.
- **Output**: Integer.

#### `getProposalsCountByType($type = null, $onlyActive = null, $withTrashed = false)`
- **Description**: Distribution of reports by user type (`user_type`).
- **Output**: `Collection` where the key is `user_type` and the value is the count.

#### `getProposersUsers($type = null, $onlyActive = null, $withTrashed = false)`
- **Description**: List of users who sent reports for the object (loads the `user` relationship).
- **Output**: `Collection` of user objects.

### 5.11 Core Functions in the Behavior (Not in the Trait)

#### `proposaledBy()`
- **Description**: Returns a collection of users who sent reports for the current object (as target).
- **Output**: `Collection` of users (with `user` loaded).

#### `isUserProposal($user = null)`
- **Description**: Checks if the user (or current user) has sent a report for the object.
- **Output**: `bool`.

#### `getUserProposal($user = null)`
- **Description**: Returns the report record for the user (if exists).
- **Output**: `Proposal` object or `null`.

#### `addProposal($data, $user = null, $parent = null): Proposal`
- **Description**: Adds a new report for the object. `$data` can be passed as an `array` or as a `string` (which will be interpreted as `content`).
- **Output**: The created `Proposal` object.

#### `toggleProposal($data, $user = null, $parent = null)`
- **Description**: If the user has sent a report, it deletes it (or restores it if soft-deleted); otherwise, it adds a new report.
- **Output**: `Proposal` object or `null` upon deletion.

#### `updateProposal($id, $data): Proposal`
- **Description**: Updates data of an existing report.

#### `deleteProposal($id): ?bool`
- **Description**: Deletes a specific report (Soft delete).

#### `countProposal(bool $onlyAccepted = false)`
- **Description**: Number of reports (optionally only accepted – currently not used).

---

## 6. Comprehensive Practical Examples

### Example 1: Adding a Report and Checking

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano2\Proposals\Models\Proposal;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// Add a report of type 'reports'
$report = $product->addProposal([
    'type' => 'reports',
    'content' => 'This product contains incorrect information',
    'is_active' => true,
], $user);

// Add a proposal
$proposal = $product->addProposal([
    'type' => 'proposals',
    'content' => 'I suggest adding a new feature',
], $user);

// Check if the user has reported this product
if ($product->isUserProposal()) {
    echo "You have reported this product";
}
```

### Example 2: Ordering Objects by Report Count

```php
$topProducts = Product::topProposed(10)->get();
```

### Example 3: Adding the Report Status Column for the Current User

```php
$products = Product::withIsProposedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_proposed_by_user ? 'You have reported' : 'You have not reported');
}
```

### Example 4: Filtering Objects that the User Has Reported (Target Side)

```php
$myReportedProducts = Product::proposedByUser()->get();
```

### Example 5: Filtering Objects that Have Sent a Report (User Side)

```php
$usersWhoReported = User::proposedToUser()->get();
```

### Example 6: Statistics

```php
$product = Product::find(1);
echo "Total reports of type 'reports': " . $product->getTotalProposals('reports');
print_r($product->getProposalsCountByType('reports')->toArray());
$users = $product->getProposersUsers('reports');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### Example 7: Combining Scopes

```php
// Products that have more than 5 reports of type 'complaints', ordered by latest report
$products = Product::hasProposals(5, 'complaints')
    ->sortByLatestProposal('DESC', 'latest_complaint_at')
    ->get();
```

### Example 8: Using the Advanced `whereHasProposalAdvanced` Scope

```php
// Products that have reports of type 'reports' (any user)
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'reports']
])->get();

// Products that have active reports of type 'complaints' from the current user
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'complaints', 'is_active' => true]
])->get();

// Products that have no reports of type 'proposals'
$products = Product::whereHasProposalAdvanced([
    'type' => 'doesntHave',
    'conditions' => ['type' => 'proposals']
])->get();

// Using a closure with date conditions
$products = Product::whereHasProposalAdvanced([
    'conditions' => function($q) {
        $q->where('type', 'reports')
          ->where('created_at', '>=', Carbon::now()->subWeek());
    }
])->get();

// Using whereIn on the report type
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => ['whereIn', ['reports', 'complaints']],
        'is_active' => true,
    ]
])->get();

// Including soft-deleted reports
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => 'complaints',
        'withTrashed' => true,
    ]
])->get();

// Controlling the user source (user side)
$users = User::whereHasProposalAdvanced([
    'side' => 'user',
    'userSource' => 'current_model', // use the current model as the user
])->get();

// Disabling user conditions (search for any reports regardless of user)
$products = Product::whereHasProposalAdvanced([
    'useUserConditions' => false,
    'conditions' => ['type' => 'reports']
])->get();
```

---

## 7. Backward Compatibility Functions

To maintain existing code that uses previous scope names, wrapper functions are provided in the main class. These functions are marked `@deprecated` and it is recommended to use the new alternatives.

**List of Supported Old Functions:**

| Old Function | New Function |
|--------------|--------------|
| `scopeSortByCountProposalsOld` | `scopeSortByCountProposals` |
| `scopeWithSortByCountProposals` | `scopeWithCountProposals` |
| `scopeAddSortByCountProposals` | `scopeAddCountProposals` |
| `scopeSortByCreatedAtProposals` | `scopeSortByLatestProposal` |
| `scopeAddSortByCreatedAtProposals` | `scopeAddLatestProposal` |
| `scopeWithSortByCreatedAtProposals` | `scopeWithLatestProposal` |
| `scopeWhereHasProposal` | `scopeWhereHasProposalAdvanced` (target side) |
| `scopeWhereHasProposals` | `scopeWhereHasProposalAdvanced` |
| `scopeWhereHasUserProposal` | `scopeWhereHasProposalAdvanced` (user side) |
| `scopeWhereHasUserProposals` | `scopeWhereHasProposalAdvanced` |

**Note regarding the old functions `whereHasProposal` and their counterparts:** They have been improved to redirect calls to the advanced scope `scopeWhereHasProposalAdvanced` while maintaining the original behavior.

---

## 8. Performance Optimization and Caching

The behavior relies on optimized subqueries to avoid loading relationships entirely. There is no built-in caching system by default, but you can rely on the caching mechanism of the models themselves or use application-level caching if you need to repeat heavy queries.

- When using `withCountProposals` or `sortByCountProposals`, only one subquery is executed.
- Using `withIsProposedByUser` or `withIsProposedToUser` adds a subquery per row, so it is recommended to use it cautiously on large lists, but it is usually acceptable due to the efficiency of SQL queries.

**Note regarding `withTrashed`:** When using `withTrashed = true` in any of the scopes or statistical functions, ensure that the `Nano2\Proposals\Models\Proposal` model uses the `SoftDeletes` trait; otherwise, query errors will occur.

---

## 9. Conclusion

`ProposalModel` is a comprehensive and powerful solution for managing the reporting, proposal, and complaint system in NanoSoft applications. Thanks to the variety of scopes and helper functions, developers can easily build sophisticated reporting systems while maintaining high flexibility and scalability. The advanced `scopeWhereHasProposalAdvanced` provides the utmost control over complex queries. Backward compatibility ensures a smooth transition from previous versions. We hope this documentation serves as a comprehensive reference for all the behavior's capabilities.
