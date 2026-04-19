## 2026-04-20

**`Nano3.Kyc` Plugin Updates – Versions 1.0.7 to 1.1.0**

### Summary of Updates

Following the official launch of the add-on in previous versions, we continued developing `Nano3.Kyc` to add advanced features that enable developers to easily and flexibly assess KYC status for users and various entities. Updates from versions 1.0.7 to 1.1.0 included:

- **Addition of comprehensive KYC assessment functions** in `Manager` (`assessKycStatus` and `assessKycStatusByCategory`).
- **Development of `DynamicAddIncludeKyc` behavior** to inject KYC data directly into API responses via `include`.
- **Expansion of user type support** in `DocumentType` to include `backend` and `frontend` users as individuals.
- **Improvement of mandatory document determination mechanism** based on `category` and `document_type`.

---

### Version 1.0.7 – Adding KYC Status Assessment Functions in `Manager`

#### Version Goals

- Provide an integrated mechanism to assess KYC status for a specific owner (individual/company).
- Calculate completion percentage, verified, missing, expired documents, and overall score.
- Support filtering by `category`, `document_type`, and `risk_level`.

#### New Features

##### 1. `assessKycStatus` Function in `Manager`

The `Manager::assessKycStatus` function was added to provide a comprehensive assessment of KYC status for a given owner.

**Function signature:**
```php
public static function assessKycStatus($owner, ?string $ownerType = null, ?int $ownerId = null, array $options = []): array
```

**Supported options:**
| Option | Description | Default |
| :--- | :--- | :--- |
| `category` | Filter by a specific document category (`primary_id`, `address`, `corporate`, ...). | `null` |
| `document_type` | Filter by a specific document type (`passport`, `utility_bill`, ...). | `null` |
| `risk_level` | Risk level to determine mandatory requirements (`low`, `medium`, `high`). | `medium` |
| `include_expired` | Include expired documents in results. | `true` |
| `include_pending` | Include pending documents. | `true` |
| `include_rejected` | Include rejected documents. | `false` |

**Response structure:**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "is_compliant": false,
        "completion_percentage": 0,
        "overall_score": 0,
        "total_documents_required": 0,
        "verified_documents_count": 0,
        "pending_documents_count": 0,
        "expired_documents_count": 0,
        "rejected_documents_count": 0,
        "verified_documents": [],
        "pending_documents": [],
        "expired_documents": [],
        "rejected_documents": [],
        "missing_required_types": [],
        "recommendations": []
    }
}
```

##### 2. Mechanism for Determining Mandatory and Recommended Documents

The function uses the following `DocumentType` methods:
- `getMandatoryTypesForParty($ownerType, $riskLevel)`: to determine mandatory document types based on owner type and risk level.
- `getRecommendedTypes($ownerType, $riskLevel)`: to determine recommended document types.

##### 3. Document Validity Checking

The function checks the validity of each verified document using:
- `DocumentType::hasExpiry()` and `DocumentType::isDocumentAcceptable()` for documents with an expiry date.
- `DocumentType::getMaxAgeDays()` and `DocumentType::isDocumentAcceptable()` for age-limited documents.

#### Benefits

- Unified and comprehensive KYC assessment that can be used in decision-making processes (e.g., allowing transactions).
- High flexibility in customizing the assessment by category, document type, or risk level.
- Automatic recommendations for missing or expiring documents.

---

### Version 1.0.8 – `DynamicAddIncludeKyc` Behavior for Injecting KYC Data into API

#### Version Goals

- Enable API responses to easily include KYC data associated with an object (user, order, product) via `include`.
- Provide a flexible mechanism to inject `include` functions into any `Transformer` without modifying the original code.
- Support assessment (`kyc_status`), document list (`kyc_documents`), and document count (`kyc_documents_count`).

#### New Features

##### 1. `DynamicAddIncludeKyc` Behavior

A class `DynamicAddIncludeKyc` (`Nano3\Kyc\Behaviors\DynamicAddIncludeKyc`) was created to inject the following functions into any `Transformer`:

| Function | Description |
| :--- | :--- |
| `includeKycStatus` | KYC status assessment for the owner (uses `Manager::assessKycStatus`). |
| `includeKycDocuments` | List of owner documents with pagination support. |
| `includeKycDocumentsCount` | Number of owner documents. |

**Usage in API request:**
```
GET /api/v1/me?include=kyc_status,kyc_documents,kyc_documents_count
```

##### 2. Support for Advanced Options via Query Parameters

Additional options can be passed for each `include`:

```
?include=kyc_status&kyc_status[risk_level]=high&kyc_status[category]=primary_id
?include=kyc_documents&kyc_documents[per_page]=20&kyc_documents[status]=verified
```

##### 3. Injecting the Behavior into Multiple Transformers

`Plugin::boot()` was updated to inject the behavior into the following Transformers:
- `UserTransformer` (website users)
- `AdminTransformer` (control panel users)
- `DeliveryTransformer` (delivery personnel)
- `ParentTransformer` (parents)
- `StudentTransformer` (students)
- `DepartmentTransformer` (branches and departments)

#### Benefits

- Enriching API responses with KYC data without requiring separate requests.
- Seamless integration with the `Nano.API` system and Fractal.
- Extensibility to easily add new Transformers.

---

### Version 1.0.9 – Adding `assessKycStatusByCategory` and Updating the Behavior

#### Version Goals

- Provide a KYC assessment function that directly relies on document `category` without needing `risk_level` or `getMandatoryTypesForParty`.
- Add a new `include` named `kyc_status_by_category` in the behavior to use this function.
- Improve flexibility in assessing specific document categories.

#### New Features

##### 1. `assessKycStatusByCategory` Function in `Manager`

A new function was added to allow KYC assessment based directly on document category.

**Function signature:**
```php
public static function assessKycStatusByCategory(
    $owner,
    ?string $ownerType = null,
    ?int $ownerId = null,
    ?string $category = null,
    $documentType = null,
    array $options = []
): array
```

**Parameters:**
| Parameter | Description |
| :--- | :--- |
| `$category` | Document category (required, default `CATEGORY_PRIMARY_ID`). |
| `$documentType` | Specific document type or array of types (optional). |

**Mechanism:**
- Builds the list of required documents directly from all document types belonging to `$category`.
- If `$documentType` is provided, narrows the scope to only the specified types.
- Does not rely on `risk_level` or `getMandatoryTypesForParty`.

##### 2. Updating `DynamicAddIncludeKyc` with `includeKycStatusByCategory`

The function `includeKycStatusByCategory` was added to the behavior, which uses `Manager::assessKycStatusByCategory`.

**Usage example:**
```
GET /api/v1/me?include=kyc_status_by_category&kyc_category=address&kyc_document_type[]=utility_bill&kyc_document_type[]=bank_statement
```

**Supported options:**
| Option | Description |
| :--- | :--- |
| `category` | Document category (required, default `primary_id`). |
| `document_type` | Specific document type or array. |
| `include_expired` | Include expired documents. |
| `include_pending` | Include pending documents. |
| `include_rejected` | Include rejected documents. |

#### Benefits

- Accurate assessment of a specific document category (e.g., "proof of address") without needing to specify a risk level.
- Ideal for verifying specific requirements (e.g., "Does the user have a valid proof of address?").
- Full integration with the API's `include` system.

---

### Version 1.1.0 – Improvements to `DocumentType` and User Type Support

#### Version Goals

- Expand `getMandatoryTypesForParty` to support `backend` and `frontend` users as individuals (`individual`).
- Ensure KYC assessment works correctly for all user types in the system.
- General code improvements and output standardization.

#### New Features

##### 1. Update to `DocumentType::getMandatoryTypesForParty`

The function was modified to convert the following user types to `individual`:
- `RainLab\User\Models\User` (website users)
- `Backend\Models\User` (control panel users)

**Code before update:**
```php
if ($partyType === 'individual') {
    $mandatory[] = self::TYPE_NATIONAL_ID;
}
```

**Code after update:**
```php
if (in_array($partyType, ['RainLab\User\Models\User', 'Backend\Models\User'])) {
    $partyType = 'individual';
}

if ($partyType === 'individual') {
    $mandatory[] = self::TYPE_NATIONAL_ID;
    if (in_array($riskLevel, ['high', 'very_high'])) {
        $mandatory[] = self::TYPE_PASSPORT;
    }
}
```

##### 2. Impact of the Update on KYC Assessment

- **Before update**: `backend` users had no mandatory documents (`total_documents_required = 0`).
- **After update**: They are treated as individuals, thus required to provide a `national_id` as a minimum (and `passport` in high-risk cases).

##### 3. General Improvements

- Standardized response structure across all `Manager` functions.
- Added debug details in development environment.
- Improved error handling in `DynamicAddIncludeKyc`.

#### Benefits

- Comprehensive coverage for all user types in the system.
- Predictable and logical KYC assessment behavior.
- Plugin readiness for various real-world scenarios.

---

### Version Summary (1.0.7 – 1.1.0)

| Version | Key Features |
| :--- | :--- |
| 1.0.7 | Added `assessKycStatus` in `Manager` for comprehensive KYC assessment. |
| 1.0.8 | Developed `DynamicAddIncludeKyc` behavior to inject KYC data into API Transformers. |
| 1.0.9 | Added `assessKycStatusByCategory` and support for `kyc_status_by_category` in API. |
| 1.1.0 | Updated `DocumentType` to support `backend` and `frontend` users as individuals in assessment. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace `Manager.php`, `DocumentType.php`, `DynamicAddIncludeKyc.php`, and `Plugin.php` with the new versions.

2. **No new migrations**:
   - This version does not require database changes.

3. **Update API definitions** (optional):
   - To benefit from the new `include` features, ensure that clients use the updated endpoints.

4. **Compatibility testing**:
   - It is recommended to test the assessment functions with different user types to ensure expected results.

---

### Conclusion

With versions 1.0.7 to 1.1.0, the `Nano3.Kyc` add-on has become more intelligent and integrated. It can now:

- **Assess KYC status** comprehensively with automatic recommendations.
- **Integrate with the API** seamlessly via `include` without separate requests.
- **Support all user types** in the system uniformly.
- **Provide high flexibility** in assessing specific document categories.

These improvements make the add-on a powerful tool for enforcing KYC policies and making identity verification-based decisions.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`Document` Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)


