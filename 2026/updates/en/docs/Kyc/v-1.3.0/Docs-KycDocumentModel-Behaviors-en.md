# `KycDocumentModel` Behavior and Document Helper Scopes Documentation

## Overview

`Nano3\Kyc\Behaviors\KycDocumentModel` is a behavior that can be added to any model in NanoSoft applications to link it to KYC (Know Your Customer) documents as an owner model. The behavior consists of:

- The main class `KycDocumentModel` which defines a `documents` relationship (morphic) with the `Document` model, advanced Toggle scopes for filters, helper scopes for group filters, and new counting scopes.
- The trait `KycDocumentScopesAndHelpers` which contains the advanced `whereHasDocuments` scope, helper scopes, document counting scopes, statistical functions, and direct checking functions.

Once this behavior is added to a model (e.g., `RainLab\User\Models\User`), you can:

- Use the `$user->documents` relationship to view their documents.
- Use powerful scopes such as `whereHasDocuments` to filter results based on document properties (type, category, status, verification...).
- Use Toggle scopes (e.g., `isToggelAnyKycDocuments`) specifically designed for `switch` and `checkbox` filters in OctoberCMS.
- Use group filter scopes such as `kycDocumentsCategory`, `kycDocumentsType`, and `kycDocumentsStatus`.
- Use specialized counting scopes like `addVerifiedDocumentsCount` and `addPendingDocumentsCount`.
- Use ready-made statistical functions like `documents_count` and `verified_documents_count`.
- Quickly check for specific document types via functions like `hasVerifiedDocument()`.

The behavior is designed to be fully compatible with `KycDocumentManager`, which applies it automatically to registered models, but it can also be applied manually using `extendClassWith`.

---

## The `documents` Relationship

When the behavior is applied, the following relationship is added to the model:

```php
public $morphMany = [
    'documents' => [
        Document::class,
        'name' => 'owner',
    ],
];
```

This means that any model applying the behavior becomes an owner of documents via the `owner_id` and `owner_type` fields in the `nano3_kyc_documents` table. You can use the relationship directly:

```php
$user = User::find(1);
foreach ($user->documents as $doc) {
    echo $doc->document_type . ' - ' . $doc->status;
}
```

---

## Advanced Scope `scopeWhereHasDocuments`

This is the main scope that provides comprehensive filtering for the existence of documents with flexible conditions. It can be used directly on any query.

### Signature

```php
public function scopeWhereHasDocuments(
    \October\Rain\Database\Builder $builder,
    array $options = []
): \October\Rain\Database\Builder
```

### Supported Options (`$options`)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `type` | `string\|array` | `null` | Document type (e.g., `'passport'`) or array `['passport','national_id']`. |
| `category` | `string\|array` | `null` | Document category (e.g., `'primary_id'`) or array. |
| `status` | `string\|array` | `null` | Document status (e.g., `'verified'`) or array. |
| `has_expiry` | `bool` | `null` | `true` for documents with an expiry date, `false` for those without. |
| `is_verified` | `bool` | `null` | `true` for verified documents, `false` for unverified. |
| `conditions` | `array\|callable` | `[]` | Additional conditions on the documents table (array or closure). |
| `boolean` | `string` | `'and'` | How to attach the condition to the original query: `'and'` or `'or'`. |
| `not` | `bool` | `false` | If `true`, inverts the condition (uses `whereDoesntHave`). |
| `count` | `int` | `null` | Specifies the required number of documents (uses `has` with a comparison operator). When set, `has` is used instead of `exists`. |
| `countOperator` | `string` | `'>=`' | Comparison operator for the count (`'>=', '=', '<', ...`). |

### How It Works

- The scope uses `whereHas` (or `orWhereHas` / `whereDoesntHave` / `orWhereDoesntHave`) on the `documents` relationship with a closure that adds the specified conditions to the subquery.
- When `count` is set, `whereHas('documents', ..., $countOperator, $count)` is used instead of a simple `whereHas`, allowing filtering by document count.

### Examples of `scopeWhereHasDocuments`

```php
// Users who have a verified primary identity document
User::whereHasDocuments([
    'category'    => 'primary_id',
    'is_verified' => true,
])->get();

// Users who have a passport or national ID with pending status
User::whereHasDocuments([
    'type'   => ['passport', 'national_id'],
    'status' => 'pending',
])->get();

// Users who have no documents at all
User::whereHasDocuments(['not' => true])->get();

// Users who have at least two verified documents
User::whereHasDocuments([
    'is_verified'   => true,
    'count'         => 2,
    'countOperator' => '>=',
])->get();

// Chaining the filter with OR on an original query
User::where('is_active', true)
    ->whereHasDocuments(['status' => 'verified'], 'or')
    ->get();

// Using a closure for custom conditions
User::whereHasDocuments([
    'conditions' => function ($q) {
        $q->where('expiry_date', '>=', Carbon::now()->addMonth());
    }
])->get();
```

---

## Helper Scopes

A set of shorthand scopes built on top of `scopeWhereHasDocuments` for common cases:

| Scope | Parameters | Equivalent |
|-------|------------|------------|
| `scopeHasDocuments($query, $count=1, $boolean='and')` | document count and boolean | `whereHasDocuments(['count' => $count, 'boolean' => $boolean])` |
| `scopeHasVerifiedDocuments($query, $count=1, $boolean='and')` | count of verified documents | `whereHasDocuments(['is_verified' => true, 'count' => $count, 'boolean' => $boolean])` |
| `scopeHasDocumentType($query, $type, $boolean='and')` | document type | `whereHasDocuments(['type' => $type, 'boolean' => $boolean])` |
| `scopeHasDocumentCategory($query, $category, $boolean='and')` | document category | `whereHasDocuments(['category' => $category, 'boolean' => $boolean])` |
| `scopeHasDocumentStatus($query, $status, $boolean='and')` | document status | `whereHasDocuments(['status' => $status, 'boolean' => $boolean])` |

**Examples:**

```php
// Users who have at least one verified document
User::hasVerifiedDocuments()->get();

// Users who have a document from the address category
User::hasDocumentCategory('address')->get();

// Users who have an expired document (or condition)
User::where('role', 'customer')
    ->hasDocumentStatus('expired', 'or')
    ->get();
```

---

## Toggle Scopes Compatible with OctoberCMS Filters

These scopes allow creating `switch` and `checkbox` filters in the control panel. They intelligently toggle the value automatically on click, and support "presence/absence" modes for documents.

### `scopeIsToggelAnyKycDocuments($query, $value)`

`switch` filter for any documents. Value `1` means "has documents", value `0` means "does not have documents".

```php
// Users who have any documents
User::isToggelAnyKycDocuments(1)->get();

// Users who have no documents
User::isToggelAnyKycDocuments(0)->get();
```

### `scopeIsToggelKycDocumentsVerified($query, $value)`

`checkbox` filter for having verified documents.

```php
// Have verified documents
User::isToggelKycDocumentsVerified(1)->get();

// Do not have verified documents
User::isToggelKycDocumentsVerified(0)->get();
```

### `scopeIsToggelKycDocumentsNotVerified($query, $value)`

The opposite of the previous filter: `1` = has unverified documents, `0` = does not have.

### `scopeIsToggelKycDocumentsPending($query, $value)`

Filter for having pending documents.

### `scopeIsToggelKycDocumentsExpired($query, $value)`

Filter for having expired documents.

### `scopeIsToggelKycDocumentsByType($query, $value)`

Filter by document type (e.g., `'passport'`). The passed value is the selected document type.

```php
// Users who have a passport
User::isToggelKycDocumentsByType('passport')->get();

// Users who do not have a passport
User::isToggelKycDocumentsByType(0, 'passport')->get(); // with toggle: send 0 after click
```

### `scopeIsToggelKycDocumentsByCategory($query, $value)`

Filter by document category (e.g., `'primary_id'`).

### `scopeIsToggelKycDocumentsStatus($query, $value)`

Filter by a custom status (e.g., `'rejected'`).

### Combined AND Toggle Scopes

These scopes allow achieving two conditions together (e.g., verified document **AND** belonging to a specific category).

#### `scopeIsToggelKycDocumentsVerifiedAndCategory($query, $value, $category = 'primary_id')`

```php
// Users who have a verified document from the primary_id category
User::isToggelKycDocumentsVerifiedAndCategory(1, 'primary_id')->get();
```

#### `scopeIsToggelKycDocumentsVerifiedAndType($query, $value, $type = 'passport')`

```php
// Users who have a verified passport
User::isToggelKycDocumentsVerifiedAndType(1, 'passport')->get();
```

#### `scopeIsToggelKycDocumentsVerifiedAndCategoryPrimaryId($query, $value)`

Shorthand for the combined condition with `primary_id`:

```php
User::isToggelKycDocumentsVerifiedAndCategoryPrimaryId(1)->get();
```

---

## Group Filter Scopes

These scopes provide direct support for `group` filters in OctoberCMS, where they can be used as `scope` in filter definitions.

### `scopeKycDocumentsCategory($query, $value)`

Filters records that have documents belonging to a specific category.

```yaml
kyc_documents_category:
    label: 'Document Category'
    type: group
    modelClass: Nano3\Kyc\Models\Document
    options: getDocumentCategoryFilterOptions
    scope: kycDocumentsCategory
```

### `scopeKycDocumentsType($query, $value)`

Filter by document type.

### `scopeKycDocumentsStatus($query, $value)`

Filter by document status (verified, pending, rejected, expired).

```yaml
kyc_documents_status:
    label: 'Document Status'
    type: group
    options:
        verified: 'Verified'
        pending: 'Pending'
    scope: kycDocumentsStatus
```

---

## Document Counting Scopes

These scopes allow adding the document count to query results or ordering results based on that count. The behavior provides both general and specialized counting scopes.

### General Scope `scopeAddDocumentsCount`

```php
public function scopeAddDocumentsCount(Builder $query, $columnName = 'documents_count', $options = [])
```

**Parameters:**
- `$columnName`: Name of the column added to SELECT.
- `$options`: Optional array to filter the counted documents (same options as `whereHasDocuments` without `boolean`/`not`), can use `type`, `category`, `status`, `is_verified`, `is_active`.

### `scopeSortByDocumentsCount(Builder $query, $orderDirection = 'DESC', $options = [])`

Orders results ascending or descending based on the document count (with optional filtering).

### Specialized Counting Scopes (New in 1.3.0)

| Scope | Description |
|-------|-------------|
| `scopeAddVerifiedDocumentsCount($query, $columnName)` | Number of verified documents. |
| `scopeAddPendingDocumentsCount($query, $columnName)` | Number of pending documents. |
| `scopeAddRejectedDocumentsCount($query, $columnName)` | Number of rejected documents. |
| `scopeAddExpiredDocumentsCount($query, $columnName)` | Number of expired documents. |

**Examples:**

```php
// Fetch users with their total document count
$users = User::addDocumentsCount('total_docs')->get();

// Fetch users with only verified document count
User::addVerifiedDocumentsCount('verified_count')->get();

// Fetch users with pending document count
User::addPendingDocumentsCount('pending_count')->get();

// Sort users descending by verified document count
User::sortByDocumentsCount('DESC', ['is_verified' => true])->get();

// Add column and sort together
User::addDocumentsCount('doc_count')->sortByDocumentsCount()->get();
```

**Note:** The internal function `buildDocumentCountSubQuery` builds the subquery used in these scopes.

---

## Statistical Accessors

These functions are added to the extended model to provide quick statistics:

| Function | Description |
|----------|-------------|
| `getDocumentsCountAttribute()` | Total number of documents associated with the entity. |
| `getVerifiedDocumentsCountAttribute()` | Number of verified documents. |
| `getPendingDocumentsCountAttribute()` | Number of pending documents. |

**Usage:**

```php
$user = User::find(1);
echo $user->documents_count;              // 5
echo $user->verified_documents_count;     // 3
echo $user->pending_documents_count;      // 1
```

---

## Checker Functions

Simple boolean functions to check for the existence of certain documents:

| Function | Description |
|----------|-------------|
| `hasAnyDocument(): bool` | Does the entity have any document? |
| `hasVerifiedDocument(): bool` | Does it have at least one verified document? |
| `hasDocumentOfType(string $type): bool` | Does it have a specific document type (e.g., `'passport'`)? |
| `getDocuments($options = [])` | Fetches the entity's documents (wrapper for `$this->model->documents()->get()`). |

**Examples:**

```php
$user = User::find(1);

if ($user->hasAnyDocument()) {
    // show documents
}

if ($user->hasVerifiedDocument()) {
    // allow purchase
}

if ($user->hasDocumentOfType('passport')) {
    // passport exists
}

// Get all documents
$docs = $user->getDocuments();
```

---

## Advanced Practical Examples

### 1. Filtering Users by Document Status and Expiry Date

```php
$expiringUsers = User::whereHasDocuments([
    'status'         => 'verified',
    'has_expiry'     => true,
    'conditions'     => function ($q) {
        $q->where('expiry_date', '>=', now())
          ->where('expiry_date', '<=', now()->addDays(30));
    }
])->get();
```

### 2. Top 10 Users by Verified Document Count

```php
$topUsers = User::addVerifiedDocumentsCount('verified_docs')
    ->sortByDocumentsCount('DESC', ['is_verified' => true])
    ->limit(10)
    ->get();
```

### 3. Checking KYC Requirements Before an Action

```php
$user = Auth::getUser();
if ($user->hasVerifiedDocument()) {
    // allow action
} else {
    throw new \Exception('You must submit a verified identity document.');
}
```

### 4. Using Toggle Scopes in Query Building

```php
// Users who have verified documents AND from the primary identity category
User::isToggelKycDocumentsVerifiedAndCategory(1, 'primary_id')->get();

// Users who have no documents
User::isToggelAnyKycDocuments(0)->get();
```

### 5. Using the Scope with Other Relationships

```php
// Cities that have users with unverified documents
City::whereHas('users', function ($q) {
    $q->whereHasDocuments(['is_verified' => false]);
})->get();
```

---

## Important Notes

- **Dependency on `Document`**: The scopes and functions assume the existence of the `Document` model and the `nano3_kyc_documents` table. Make sure the `Nano3.Kyc` add-on is installed and migrations are run.
- **Performance**: Using `whereHas` with a large number of documents can be expensive. It is recommended to use appropriate indexes (e.g., `owner_id`, `owner_type`, `document_type`, `status`), which are created by the migration.
- **Extensibility**: You can add additional scopes to the extended model without modifying the original class.
- **Compatibility**: The behavior is designed to work with the OctoberCMS Builder. All scopes accept a `Builder` and return a `Builder`, allowing query chaining.
- **Toggle Scopes**: Support the intelligent toggle mechanism when used with `switch`/`checkbox` filters in OctoberCMS, where calling `post('scopeName')` automatically inverts the value.

---

## Summary

The `KycDocumentModel` behavior and the `KycDocumentScopesAndHelpers` trait provide a powerful and easy-to-use layer for managing KYC documents for any model. Through the advanced `whereHasDocuments` scope, helper scopes, OctoberCMS-compatible Toggle scopes, and specialized counting scopes, you can build complex queries, sophisticated filtering, and rich control panel user experiences with ease.

---

## Additional Documentation

**Reference Documentation**:
- [General Plugin Documentation](./Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./Docs-HasValidKycDocuments-Trait-en.md)
- [`Document` Model Documentation](./Docs-Document-Model-en.md)
- [`DynamicAddIncludeKyc` Behavior Documentation](./Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [`KycDocumentManager` Class Documentation](./Docs-KycDocumentManager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
