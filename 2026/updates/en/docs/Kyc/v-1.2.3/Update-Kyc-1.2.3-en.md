## 2026-04-21 - 2026-04-22

**`Nano3.Kyc` Plugin Updates – Version 1.2.3**

### Summary of Updates

After restructuring the KYC assessment logic in version 1.2.2 and introducing the concept of "alternative groups" for documents, version 1.2.3 brings radical improvements to the performance and speed of KYC status checking. The fast-checking functions have been moved to an independent trait `HasValidKycDocuments` with advanced caching and atomic lock support, making repeated checks (e.g., verifying the existence of an identity or proof of address) extremely lightweight on the database and suitable for use in Middleware, Policies, and every API request.

---

### Version 1.2.3 – Performance and Caching Improvements for Fast Check Functions

#### Version Goals

- Move the `hasValidKycDocumentByCategory` logic to an independent trait `HasValidKycDocuments` to improve maintainability.
- Support **alternative groups** (`group:primary_identity`, `group:address_verification`) within the fast-check functions.
- Implement **multi-level caching** with support for Cache Tags (for Redis/Memcached) and fallback key groups.
- Add **atomic locks** to prevent cache stampede during peak times.
- Automatically clear the cache upon any change to a user's documents (creation, update, deletion, verification, rejection).
- Provide a shorthand function `hasValidKycDocument()` to check for the existence of any valid document.

#### New Features

##### 1. `HasValidKycDocuments` Trait

All fast-checking functions have been moved to a new trait `Nano3\Kyc\Classes\Manager\HasValidKycDocuments`, which provides:

| Method | Description |
| :--- | :--- |
| `hasValidKycDocumentByCategory($owner, $ownerType, $ownerId, $category, $documentType, $options)` | Checks for a valid document within a specified category and types with advanced options. |
| `hasValidKycDocument($owner, $ownerType, $ownerId)` | Shorthand function to check for **any** valid KYC document for the owner. |
| `clearKycCheckCache($ownerType, $ownerId)` | Clears all cache results associated with a specific owner (called automatically when documents change). |

##### 2. Support for Alternative Groups in Fast Checks

You can now pass `group:primary_identity` or `group:address_verification` within `$documentType`, and it will be automatically expanded into the appropriate list of individual types. This ensures consistency with the new `assessKycStatus` logic.

**Example:**
```php
// Check for any primary identity (passport, national ID, driver's license)
$hasId = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    'group:primary_identity'
);
```

##### 3. Multi-Level Caching

To reduce the database load on frequent calls (e.g., `include=kyc_status` for each user), an intelligent caching mechanism has been implemented:

- **Cache Tags (for supported systems)**: Cache keys are grouped under a tag like `owner_{hash}`, allowing them to be cleared all at once when needed.
- **Fallback key group**: If the cache driver does not support tags (e.g., `file`), keys are stored in a separate array (`owner_keys`) for manual clearing.
- **Default TTL of 5 minutes** (configurable via `cache_ttl`).

##### 4. Atomic Locks

In high-concurrency scenarios (multiple requests for the same user at the same moment), several operations may attempt to rebuild the same cache result, wasting resources. An atomic lock (`Cache::lock`) has been added to ensure that only one operation executes the actual query, while other operations wait and use the stored result.

**New options in `$options`:**
| Option | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `use_cache` | `bool` | `true` | Enable/disable caching. |
| `cache_ttl` | `int` | `300` | Cache TTL in seconds. |
| `use_atomic_lock` | `bool` | `true` | Enable/disable atomic lock. |

##### 5. Automatic Cache Invalidation on Document Changes

The `Document` model (`afterSave` and `afterDelete`) has been modified to automatically call `Manager::clearKycCheckCache()` upon any change to a document (creation, update, deletion, verification, rejection). This ensures that fast-check results always reflect the latest user state without manual intervention.

##### 6. Query Performance Improvements

- Use `select()` to retrieve only necessary columns (`id`, `document_type`, `status`, `issue_date`, `expiry_date`, `verification_score`).
- Use `exists()` instead of `get()` when validity checking is not needed (`check_validity = false` and `require_all_types = false`).
- Order `$requiredTypes` to ensure cache key consistency.

---

### Practical Examples

#### 1. Checking for a valid primary identity (with caching)

```php
$hasId = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    'group:primary_identity',
    ['use_cache' => true, 'cache_ttl' => 600] // 10 minutes
);

if ($hasId) {
    // Allow access
}
```

#### 2. Checking for any proof of address (including pending documents)

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

#### 3. Using the shorthand function to check for any document

```php
if (Manager::hasValidKycDocument($user)) {
    echo "User has at least one valid KYC document.";
}
```

#### 4. Using in Middleware to protect routes

```php
class RequireKycMiddleware
{
    public function handle($request, Closure $next)
    {
        $user = Auth::getUser();
        if (!Manager::hasValidKycDocumentByCategory($user, null, null, DocumentType::CATEGORY_PRIMARY_ID, 'group:primary_identity')) {
            return response()->json(['error' => 'KYC verification required'], 403);
        }
        return $next($request);
    }
}
```

---

### Version Summary (1.2.2 – 1.2.3)

| Version | Key Features |
| :--- | :--- |
| 1.2.2 | Restructured assessment logic into `HasAssessKycStatus` trait, added alternative document groups, improved assessment accuracy. |
| 1.2.3 | Moved fast-check functions to `HasValidKycDocuments` trait, multi-level caching, atomic locks, support for alternative groups in fast checks, automatic cache invalidation, comprehensive performance improvements. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace `Manager.php`, `Document.php`, and add new files:
     - `classes/Manager/HasValidKycDocuments.php`
     - `classes/Manager/HasAssessKycStatus.php` (if not already present)

2. **Run database migrations** (no changes in this version).

3. **Update API definitions** (no endpoint changes).

4. **Performance testing**:
   - Monitor the performance of endpoints using `include=kyc_status` after activation; they should see significant improvement due to caching.

5. **Manually clear cache (optional)**:
   - If you want to apply caching immediately, you can clear the application's general cache: `php artisan cache:clear`.

---

### Conclusion

Version 1.2.3 represents a significant leap in the performance of KYC checking operations. Thanks to intelligent caching and atomic locks, functions that are called thousands of times per day (such as checking identity status) have become extremely fast and do not burden the database. Automatic cache invalidation ensures result accuracy at all times. These improvements make the `Nano3.Kyc` add-on ready for use in large-scale, high-traffic projects without any performance concerns.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./docs/Kyc/Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./docs/Kyc/Docs-HasValidKycDocuments-Trait-en.md)
- [`Document` Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [`DynamicAddIncludeKyc` Behavior Documentation](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)
