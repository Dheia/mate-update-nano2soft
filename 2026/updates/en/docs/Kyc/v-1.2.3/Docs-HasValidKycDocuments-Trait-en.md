# `HasValidKycDocuments` Trait Documentation – Fast KYC Checker with Advanced Caching

**Namespace:** `Nano3\Kyc\Classes\Manager`  
**Purpose:** Provide fast, high-performance functions to check for the existence of valid KYC documents for a given owner, with support for multi-level caching and atomic locks to prevent request contention.

---

## Overview

The `HasValidKycDocuments` trait is part of the `Manager` class in the `Nano3.Kyc` add-on. This trait provides a set of functions designed specifically for **fast** KYC status checking (yes/no) without the need for comprehensive analysis or complex calculations. These functions are ideal for use in:

- **Middleware** for checking access permissions.
- **Policies** to determine whether a user can perform a certain action.
- **API responses** that need a quick KYC status indicator (e.g., `has_verified_identity: true`).
- **Front-end interfaces** to decide whether to show a prompt to upload documents.

**Key Features:**

- **Superior performance**: Uses optimized queries with `exists()` and limited `select()`, stopping at the first valid document.
- **Smart caching**: Supports Cache Tags (for Redis/Memcached) with a fallback mechanism for stores that do not support them. Default TTL is 5 minutes.
- **Atomic locks**: Prevents "cache stampede" when multiple concurrent operations attempt to rebuild the same cache result simultaneously.
- **Automatic cache invalidation**: The cache associated with an owner is automatically cleared when any document belonging to them is created, updated, deleted, verified, or rejected (via model events).
- **Alternative groups support**: You can pass `group:primary_identity` or `group:address_verification` to check for the existence of any document within a specific group.
- **Flexible options**: Control inclusion of expired, pending, rejected documents, require all specified types, and set a minimum verification score.

---

## Public Methods

### `hasValidKycDocumentByCategory`

Checks for the existence of at least one valid KYC document (or all specified types) within a given category and specific types.

```php
public static function hasValidKycDocumentByCategory(
    $owner,
    ?string $ownerType = null,
    ?int $ownerId = null,
    ?string $category = null,
    $documentType = null,
    array $options = []
): bool
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$owner` | `object\|string` | Owner object **or** owner type. |
| `$ownerType` | `?string` | Owner type (required if `$owner` is not an object). |
| `$ownerId` | `?int` | Owner ID (required if `$owner` is not an object). |
| `$category` | `?string` | Document category (default `CATEGORY_PRIMARY_ID`). |
| `$documentType` | `string\|array\|null` | Specific document type, array of types, or a string like `group:primary_identity` (optional). |
| `$options` | `array` | Additional options (see options table below). |

#### `$options` Table

| Option | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `include_expired` | `bool` | `false` | Should expired documents be considered valid? |
| `include_pending` | `bool` | `false` | Should pending documents be considered valid? |
| `include_rejected` | `bool` | `false` | Should rejected documents be considered valid? |
| `require_all_types` | `bool` | `false` | If `true`, **all** specified types must exist and be valid. |
| `min_verification_score` | `int\|null` | `null` | Minimum required verification score (0-100). |
| `check_validity` | `bool` | `true` | Should document validity (expiry date / max age) be checked? |
| `use_cache` | `bool` | `true` | Should caching be used? |
| `cache_ttl` | `int` | `300` | Cache TTL in seconds (5 minutes). |
| `use_atomic_lock` | `bool` | `true` | Use atomic lock to prevent cache stampede in concurrent requests. |

#### Return Value

`bool` – `true` if the conditions are satisfied, `false` in all other cases (including errors).

---

### `hasValidKycDocument`

A shorthand function to check for the existence of **any** valid KYC document for the owner (regardless of category or type).

```php
public static function hasValidKycDocument(
    $owner,
    ?string $ownerType = null,
    ?int $ownerId = null
): bool
```

Equivalent to calling `hasValidKycDocumentByCategory` with `$category = null` and `$documentType = null` and default options (`check_validity = true`).

---

## Comprehensive Practical Examples

### 1. Checking for a valid primary identity (passport, national ID, or driver's license)

```php
use Nano3\Kyc\Classes\Manager;
use Nano3\Kyc\Classes\DocumentType;

$user = Auth::getUser();

$hasValidId = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    'group:primary_identity'
);

if ($hasValidId) {
    echo "User has a valid primary identity document.";
} else {
    echo "Please upload a valid passport, national ID, or driver's license.";
}
```

### 2. Checking for a **specific** passport with a verification score of 90+

```php
$hasHighScorePassport = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    DocumentType::TYPE_PASSPORT,
    ['min_verification_score' => 90]
);
```

### 3. Checking for **both** a passport and a driver's license (both required)

```php
$hasBoth = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    [DocumentType::TYPE_PASSPORT, DocumentType::TYPE_DRIVERS_LICENSE],
    ['require_all_types' => true]
);
```

### 4. Checking for any proof of address document (including pending bills)

```php
$hasAddress = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_ADDRESS,
    null,
    ['include_pending' => true]
);
```

### 5. Using the shorthand function to check for any KYC document (simple cases)

```php
if (Manager::hasValidKycDocument($user)) {
    echo "User has at least one KYC document.";
}
```

### 6. Using in Middleware to protect a route requiring verified identity

```php
namespace App\Http\Middleware;

use Closure;
use Auth;
use Nano3\Kyc\Classes\Manager;
use Nano3\Kyc\Classes\DocumentType;

class RequireVerifiedIdentity
{
    public function handle($request, Closure $next)
    {
        $user = Auth::getUser();
        
        if (!$user) {
            return response()->json(['error' => 'Unauthenticated'], 401);
        }

        // Check for a valid primary identity (automatically uses caching)
        if (!Manager::hasValidKycDocumentByCategory(
            $user,
            null,
            null,
            DocumentType::CATEGORY_PRIMARY_ID,
            'group:primary_identity'
        )) {
            return response()->json([
                'error' => 'Identity verification required',
                'message' => 'Please upload a valid passport, national ID, or driver\'s license.'
            ], 403);
        }

        return $next($request);
    }
}
```

### 7. Using with custom cache options (longer TTL)

```php
$hasId = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    null,
    ['use_cache' => true, 'cache_ttl' => 3600] // one hour
);
```

### 8. Disabling validity checking (for statistical or historical purposes)

```php
// Even if the document has expired, it will be considered present
$hasAnyPassport = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    DocumentType::TYPE_PASSPORT,
    ['check_validity' => false, 'include_expired' => true]
);
```

---

## Caching Mechanism and Atomic Locks

### How the Cache Works?

1. When the function is called for the first time, a unique cache key is generated based on all parameters (owner, category, types, options).
2. If `use_cache = true`, the key is looked up in the cache. If a result is found, it is returned immediately without a database query.
3. If no result is found, the actual query is executed, and the result is stored in the cache.
4. The key is also recorded in an owner-specific group (or using Cache Tags) to facilitate clearing all its keys later.

### Atomic Locks

In high-concurrency scenarios (e.g., a homepage displaying data for thousands of users with KYC status), multiple concurrent operations may attempt to compute the same uncached result simultaneously. This leads to a "cache stampede" and wastes database resources.

The trait uses `Cache::lock()` to ensure that **only one** operation executes the actual query, while other operations wait (up to 5 seconds) and then use the newly stored result. This protects the database from unnecessary pressure.

### Automatic Cache Invalidation

The `Document` model is tied to the trait: in `afterSave` and `afterDelete`, `Manager::clearKycCheckCache($ownerType, $ownerId)` is called automatically. This ensures that any change to a user's documents (new upload, update, deletion, verification, rejection) clears all associated cache results, maintaining data accuracy.

---

## Best Practices

1. **Use `hasValidKycDocumentByCategory` in Middleware and Policies**: It is ideal for checking conditions before allowing access to resources.
2. **Do not use it for detailed reporting**: These functions are designed for fast yes/no answers. For comprehensive analysis, use `assessKycStatus`.
3. **Take advantage of alternative groups**: Use `'group:primary_identity'` instead of manually listing all types, especially since group lists may change in the future.
4. **Leave default cache settings**: 5 minutes provides an excellent balance between data freshness and performance. It can be increased for applications where documents do not change often.
5. **Do not disable `use_atomic_lock` unless you are certain**: In high-traffic production environments, the atomic lock significantly protects the database.
6. **Ensure `Document` clears the cache**: This is included by default in the model. Do not remove it unless you are managing the cache manually in another way.

---

## Additional Documentation

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./docs/Kyc/Docs-HasAssessKycStatus-Trait-en.md)
- [`Document` Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [`DynamicAddIncludeKyc` Behavior Documentation](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)
