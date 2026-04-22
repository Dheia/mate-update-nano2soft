# `HasAssessKycStatus` Trait Documentation – KYC Status Assessment Engine

## Overview

The `HasAssessKycStatus` trait is located at `Nano3\Kyc\Classes\Manager\HasAssessKycStatus` and is responsible for the entire logic of assessing Know Your Customer (KYC) status within the `Nano3.Kyc` add-on. This trait was extracted from the main `Manager` class with the following goals:

- **Separation of concerns**: Keep the `Manager` class focused on document management operations (CRUD), while this trait exclusively handles assessment and analysis tasks.
- **Improved maintainability and testability**: KYC assessment logic can be modified or tested independently without affecting the rest of the class.
- **Future extensibility**: Facilitates adding new assessment policies or alternative groups without complicating the core code.

The trait relies on the `DocumentType` class to obtain document definitions and alternative groups, and on the `Document` model to retrieve documents for the owner. It provides two main assessment functions:

- `assessKycStatus`: Comprehensive KYC status assessment based on risk level and mandatory requirements.
- `assessKycStatusByCategory`: KYC status assessment limited to a specific document category (e.g., primary identity, proof of address).

**What makes this trait stand out** is its support for an **Alternative Groups** system, which allows defining groups of document types (e.g., `primary_identity` including passport, national ID, driver's license, etc.), considering any document from the group sufficient to satisfy the mandatory requirement. This approach reflects real-world compliance practices where not a specific type but a category of acceptable documents is required.

---

## Unified Response Structure

All assessment methods in the trait return a response with the same unified structure used by `Manager`, facilitating integration with APIs and consuming applications.

| Key | Type | Description |
| :--- | :--- | :--- |
| `code` | `int` | Status code (200 for success, 400 for error). |
| `status` | `bool` | Operation status (`true` for success, `false` for failure). |
| `message` | `string` | Descriptive message (translatable). |
| `error` | `string\|null` | Error message (if any). |
| `data` | `array` | Assessment data (see `data` structure below). |
| `input_data` | `array` | Input parameters after initial processing. |
| `process_data` | `array` | Data actually used during operation execution. |
| `debug` | `array\|null` | Debug information (only appears in development environment when `app.debug = true`). |

**Structure of `data` for assessment:**

| Key | Type | Description |
| :--- | :--- | :--- |
| `is_compliant` | `bool` | Is the KYC status fully compliant with requirements? |
| `completion_percentage` | `float` | Percentage of mandatory requirements completed (0-100). |
| `overall_score` | `int` | Overall verification score (0-100) calculated from document weights. |
| `total_documents_required` | `int` | Number of mandatory requirements (including groups). |
| `verified_documents_count` | `int` | Number of verified and valid documents. |
| `pending_documents_count` | `int` | Number of pending documents. |
| `expired_documents_count` | `int` | Number of expired documents. |
| `rejected_documents_count` | `int` | Number of rejected documents. |
| `verified_documents` | `array` | List of verified documents with brief details. |
| `pending_documents` | `array` | List of pending documents. |
| `expired_documents` | `array` | List of expired documents. |
| `rejected_documents` | `array` | List of rejected documents. |
| `missing_required_types` | `array` | List of missing document types (or examples of unmet groups). |
| `recommendations` | `array` | Textual recommendations for the user (e.g., "Proof of address is required"). |

---

## Public Methods in the Trait

### 1. `assessKycStatus`

Comprehensive assessment of KYC status for a given owner (individual or company). It uses `DocumentType::getMandatoryTypesForParty` to determine mandatory requirements based on party type and risk level, and employs the alternative groups system to determine whether requirements are satisfied.

```php
public static function assessKycStatus($owner, ?string $ownerType = null, ?int $ownerId = null, array $options = []): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$owner` | `object\|string` | Owner object **or** owner type (if `$ownerType` and `$ownerId` are provided). |
| `$ownerType` | `?string` | Owner type (required if `$owner` is not an object). Example: `RainLab\User\Models\User`. |
| `$ownerId` | `?int` | Owner ID (required if `$owner` is not an object). |
| `$options` | `array` | Additional options (see table below). |

**Supported options in `$options`:**

| Option | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `risk_level` | `string` | `'medium'` | Risk level to determine mandatory documents (`low`, `medium`, `high`). |
| `category` | `?string` | `null` | Filter by a specific document category (`primary_id`, `address`, `corporate`, ...). |
| `document_type` | `string\|array\|null` | `null` | Filter by a specific document type or array of types. |
| `include_expired` | `bool` | `true` | Include expired documents in the results. |
| `include_pending` | `bool` | `true` | Include pending documents. |
| `include_rejected` | `bool` | `false` | Include rejected documents. |

**Practical example (assessing a user with medium risk):**

```php
$user = Auth::getUser();
$assessment = Manager::assessKycStatus($user, null, null, ['risk_level' => 'medium']);

if ($assessment['status']) {
    echo "Completion percentage: " . $assessment['data']['completion_percentage'] . "%\n";
    foreach ($assessment['data']['recommendations'] as $rec) {
        echo "- $rec\n";
    }
}
```

---

### 2. `assessKycStatusByCategory`

KYC status assessment for a given owner based on a specific document category (e.g., `primary_id`). Unlike the general function, this **does not** rely on `risk_level` or `getMandatoryTypesForParty`; instead, it determines requirements based on the requested category and optionally specified types, while also leveraging the alternative groups system.

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

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$owner` | `object\|string` | Owner object **or** owner type. |
| `$ownerType` | `?string` | Owner type (required if `$owner` is not an object). |
| `$ownerId` | `?int` | Owner ID. |
| `$category` | `?string` | Document category (**required**; if `null` or empty, `CATEGORY_PRIMARY_ID` is used). |
| `$documentType` | `string\|array\|null` | Specific document type or array of types (optional). If provided, narrows the requirements to only these types. |
| `$options` | `array` | Additional options (`include_expired`, `include_pending`, `include_rejected`). |

**Practical example (assessing primary ID category for a user, requiring only passport):**

```php
$user = Auth::getUser();
$assessment = Manager::assessKycStatusByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    'passport',
    ['include_expired' => true]
);

if ($assessment['status']) {
    echo "Required documents: " . $assessment['data']['total_documents_required'] . "\n";
    echo "Verified documents: " . $assessment['data']['verified_documents_count'] . "\n";
}
```

---

## Internal Helper Methods

To understand how the trait works, it is helpful to look at the following helper methods (all `protected static`):

| Method | Description |
| :--- | :--- |
| `getEmptyAssessmentData()` | Returns the empty assessment data structure. |
| `processMandatoryRequirements($mandatoryRequirements, $category, $documentType)` | Processes mandatory requirements (which may contain strings like `group:primary_identity`) and separates them into `individual_types` (individual types) and `group_requirements` (groups and their member types). |
| `analyzeDocuments($documents)` | Analyzes the retrieved document set and extracts statistics (verified, pending, expired counts) and a list of compliant types (`compliant_types`). |
| `determineSatisfiedGroups($groupRequirements, $compliantTypes)` | Determines which alternative groups have been satisfied (i.e., have at least one verified document). |
| `determineMissingRequirements($mandatoryRequirements, $groupRequirements, $satisfiedGroups, $compliantTypes)` | Determines missing requirements, whether individual types or unmet alternative groups. |
| `buildRecommendations(...)` | Builds a list of textual recommendations based on analysis results (e.g., "Proof of address is required"). |

---

## Alternative Groups System

The core flexibility offered by the trait is its support for **alternative groups** defined in `DocumentType` via the `getAlternativeGroups()` method. Currently, the default supported groups are:

| Group Key | Description | Member Document Types |
| :--- | :--- | :--- |
| `primary_identity` | Primary identity documents | `passport`, `national_id`, `drivers_license`, `residence_permit`, `refugee_document` |
| `address_verification` | Proof of address documents | `utility_bill`, `bank_statement`, `tax_document`, `rental_agreement`, `mortgage_statement` |

When a mandatory requirement is defined as `'group:primary_identity'` (as in `DocumentType::getMandatoryTypesForParty`), the trait treats it as "at least one verified document from this group is required." If any verified document from the group exists, the requirement is considered satisfied.

These groups can be extended or modified via the configuration file `config/nano3/kyc/document_types.php` in future versions.

---

## Integrated Examples

### Example 1: Checking KYC completeness before allowing a specific action

```php
use Nano3\Kyc\Classes\Manager;

$user = Auth::getUser();
$assessment = Manager::assessKycStatus($user);

if ($assessment['status'] && $assessment['data']['is_compliant']) {
    // Allow the action
} else {
    $missing = implode(', ', $assessment['data']['missing_required_types']);
    throw new ApplicationException("Please complete the required documents: " . $missing);
}
```

### Example 2: Getting primary identity completion percentage only

```php
$user = Auth::getUser();
$assessment = Manager::assessKycStatusByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID
);

$progress = $assessment['data']['completion_percentage'];
echo "Primary identity progress: $progress%";
```

---

## Dependencies

The `HasAssessKycStatus` trait depends on the following components:

- **`Nano3\Kyc\Classes\DocumentType`**: To obtain document definitions, alternative groups, and mandatory requirements.
- **`Nano3\Kyc\Models\Document`**: To retrieve owner documents and apply scopes.
- **`Carbon\Carbon`**: For date handling (validity checking).

---

## Important Notes

1. **Assessment relies only on verified documents (`verified`)** to determine requirement satisfaction. Pending or rejected documents are not counted toward completion.
2. **Overall score (`overall_score`)** is calculated based on the weights of individual documents that constitute the mandatory requirements (after expanding groups), not on all existing documents.
3. **Recommendations** are generated automatically based on missing requirements and the presence of expired or pending documents.
4. **Extensibility**: New alternative groups can be added easily by modifying `DocumentType::getAlternativeGroups()`, without needing to change the trait's code.
5. **Compatibility**: The trait is designed to work statically, so its methods can be called directly from `Manager` without instantiating an object.

---

## Summary

The `HasAssessKycStatus` trait provides a flexible and powerful engine for assessing KYC status, with a clear separation of concerns and full support for alternative groups. Thanks to this trait, developers can obtain accurate and realistic compliance assessments, and easily customize requirements according to different business policies.

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)
