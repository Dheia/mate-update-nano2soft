## 2026-04-22 - 2026-04-23

**`Nano3.Kyc` Plugin Updates – Version 1.2.4**

### Summary of Updates

After improving the performance of fast-check functions in version 1.2.3 and adding caching and atomic locks, version 1.2.4 brings comprehensive improvements to the `DynamicAddIncludeKyc` behavior – responsible for injecting KYC relationships into `Transformers` of `Nano.API`. This release focuses on:

- **Radical performance improvement** when requesting multiple KYC relationships together through request-level caching.
- **Adding 7 new relationships** including the verification summary (`kyc_verification_summary`) and quick checks for each of the six document categories (`is_verifier_primary_id`, `is_verifier_address`, ...).
- **Support for advanced options** such as `expiring_within_days` to detect documents nearing expiration.
- **Updated comprehensive documentation** covering all new relationships with integrated practical examples.

This release makes integrating KYC data into API responses more powerful and flexible, providing developers with precise tools to control check behavior according to application needs.

---

### Version 1.2.4 – Comprehensive Improvements to `DynamicAddIncludeKyc` Behavior and New KYC Relationships

#### Version Goals

- Improve the performance of the `DynamicAddIncludeKyc` behavior by sharing database queries through request-level caching (`$documentsCache` and `$categoryCheckCache`).
- Add the `kyc_verification_summary` relationship, providing a quick summary of all six document categories with statistics (expired, pending, rejected counts).
- Add quick boolean verifiers for each document category: `is_verifier_primary_id`, `is_verifier_secondary_id`, `is_verifier_address`, `is_verifier_corporate`, `is_verifier_ubo`, `is_verifier_additional`.
- Support advanced options for all boolean verifiers and the verification summary, including `include_pending`, `include_expired`, `check_validity`, and `expiring_within_days` (to detect documents expiring within a specified number of days).
- Improve the relationship registration mechanism in `__construct` so they are added to `secretIncludes` only if the original `Transformer` supports them; otherwise, they are added to `availableIncludes`.
- Comprehensively update the documentation files `Docs-DynamicAddIncludeKyc-Behaviors-en.md` and `Docs-API-Documentation-en.md` to cover all new features with practical examples.

#### New Features

##### 1. Request-Level Caching

Previously, requesting multiple KYC relationships in a single API call (e.g., `?include=is_verifier_primary_id,is_verifier_address,kyc_verification_summary`) resulted in separate, repeated database queries for the same owner. In this version, two request-level caching mechanisms have been added:

| Property | Description |
| :--- | :--- |
| `$documentsCache` | Stores all owner documents retrieved from the database. Fetched only once per request, regardless of how many relationships are requested. |
| `$categoryCheckCache` | Stores category check results (`true`/`false`) considering the passed options. Prevents recalculating the same category with the same options within a single request. |

**Result:** Significant improvement in response time and reduction in database load when using multiple KYC relationships together.

##### 2. `kyc_verification_summary` Relationship – Verification Summary

This relationship provides an object containing the status of all six document categories (`primary_id`, `secondary_id`, `address`, `corporate`, `ubo`, `additional`) along with quick statistics:

| Field | Description |
| :--- | :--- |
| `primary_id` | `true` if the owner has a valid primary identity document. |
| `secondary_id` | `true` if the owner has a valid secondary identity document. |
| `address` | `true` if the owner has a valid proof of address document. |
| `corporate` | `true` if the owner has a valid corporate document. |
| `ubo` | `true` if the owner has a valid ultimate beneficial owner document. |
| `additional` | `true` if the owner has a valid additional document. |
| `total_verified` | Number of verified documents. |
| `total_pending` | Number of pending documents. |
| `total_expired` | Number of expired documents. |
| `total_rejected` | Number of rejected documents. |

**Supported options:** `include_pending`, `include_expired`, `check_validity`, `expiring_within_days`.

##### 3. Boolean Verifiers for Each Category

6 new boolean relationships (`true`/`false`) have been added, allowing quick checking for a valid document within a specific category:

| Relationship | Description |
| :--- | :--- |
| `is_verifier_primary_id` | Is there a valid primary identity document? |
| `is_verifier_secondary_id` | Is there a valid secondary identity document? |
| `is_verifier_address` | Is there a valid proof of address document? |
| `is_verifier_corporate` | Is there a valid corporate document? |
| `is_verifier_ubo` | Is there a valid ultimate beneficial owner document? |
| `is_verifier_additional` | Is there a valid additional document? |

All these relationships support the same advanced options (`include_pending`, `include_expired`, `check_validity`, `expiring_within_days`).

> **Note:** Relationships from `is_verifier_secondary_id` to `is_verifier_additional` are added to `secretIncludes` (do not appear in automatic API documentation but can be used), while `is_verifier_primary_id` and `kyc_verification_summary` are public relationships added to `availableIncludes`.

##### 4. Support for the `expiring_within_days` Option

Developers can now pass the `expiring_within_days` option (positive integer) to any of the boolean verifiers or the verification summary. When specified, the check will return `true` if there is a **currently valid** document that will expire within the specified number of days. This is useful for alerting users to renew their documents before expiration.

**Example:** Check if the user has an identity expiring within 30 days:
```
?include=is_verifier_primary_id&is_verifier_primary_id[expiring_within_days]=30
```

##### 5. Improved Relationship Registration Mechanism (`__construct`)

The constructor function of the `DynamicAddIncludeKyc` behavior has been improved to be smarter and more compatible with all types of `Transformers`:

- Checks for the existence of the `secretIncludes` property in the original class (`property_exists`).
- If the property exists, new relationships are added to `secretIncludes` (except for public relationships, which are also added to `availableIncludes`).
- If the property does not exist (i.e., the `Transformer` does not support secret relationships), all relationships are added directly to `availableIncludes`.

This ensures predictable behavior in all environments without errors.

##### 6. Documentation Updates

The two main documentation files have been updated to reflect all new features:

| File | Updates |
| :--- | :--- |
| `Docs-DynamicAddIncludeKyc-Behaviors-en.md` | Completely rewritten to include detailed explanations of all new relationships, request-level caching mechanisms, advanced options, and integrated practical examples. |
| `Docs-API-Documentation-en.md` | Added a dedicated section titled "Including KYC Data in API Responses" explaining the available relationships, how to use them, supported options, with practical examples. |

---

### Practical Examples

#### 1. Getting a Quick KYC Status Summary with Expiring Within 30 Days Check

**Request:**
```http
GET /api/v1/me?include=kyc_verification_summary&kyc_verification_summary[include_pending]=true&kyc_verification_summary[expiring_within_days]=30 HTTP/1.1
Authorization: Bearer ...
```

**Response (excerpt):**
```json
{
    "id": 24,
    "name": "Ahmed Mohammed",
    "kyc_verification_summary": {
        "primary_id": true,
        "secondary_id": false,
        "address": true,
        "corporate": false,
        "ubo": false,
        "additional": false,
        "total_verified": 2,
        "total_pending": 1,
        "total_expired": 0,
        "total_rejected": 0
    }
}
```

#### 2. Using Multiple Boolean Verifiers Together in One Request

**Request:**
```http
GET /api/v1/me?include=is_verifier_primary_id,is_verifier_address,is_verifier_corporate HTTP/1.1
Authorization: Bearer ...
```

**Response (excerpt):**
```json
{
    "id": 24,
    "name": "Ahmed Mohammed",
    "is_verifier_primary_id": true,
    "is_verifier_address": false,
    "is_verifier_corporate": false
}
```

> **Performance Note:** In this example, the user's documents are fetched from the database **only once** and stored in `$documentsCache`, then each category is checked against the same cached data.

#### 3. Checking Primary Identity Including Pending Documents

**Request:**
```http
GET /api/v1/me?include=is_verifier_primary_id&is_verifier_primary_id[include_pending]=true HTTP/1.1
Authorization: Bearer ...
```

In this case, `is_verifier_primary_id` will return `true` even if the document is still in `pending` status (under review).

---

### Version Summary (1.2.3 – 1.2.4)

| Version | Key Features |
| :--- | :--- |
| 1.2.3 | Moved fast-check functions to `HasValidKycDocuments` trait, multi-level caching, atomic locks, automatic cache invalidation. |
| 1.2.4 | Improved `DynamicAddIncludeKyc` behavior performance via request-level caching, added 7 new relationships (`kyc_verification_summary` and 6 category verifiers), support for `expiring_within_days`, improved relationship registration mechanism, comprehensive documentation updates. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace the `behaviors/DynamicAddIncludeKyc.php` file with the new version.
   - Ensure the `HasValidKycDocuments.php` trait exists (added in version 1.2.3).

2. **Run database migrations** (no changes in this version).

3. **Update API definitions** (no endpoint changes).

4. **Test the new relationships**:
   - Try `?include=kyc_verification_summary` on `/api/v1/me` to ensure the summary works correctly.
   - Try multiple boolean verifiers together and observe the improved response time.

5. **Review the new documentation**:
   - Read the "Including KYC Data in API Responses" section in `Docs-API-Documentation-en.md` to understand all available options.

---

### Conclusion

With version 1.2.4, we have completed the cycle of improving the performance and efficiency of the KYC system from all angles: from backend check functions (version 1.2.3) to the API interface and data inclusion (version 1.2.4). Developers can now:

- Heavily use KYC relationships within API responses without worrying about database performance.
- Obtain quick summaries or precise checks as needed.
- Customize check behavior precisely using advanced options such as `expiring_within_days`.

We look forward to your feedback and suggestions to continue developing this essential add-on.

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
