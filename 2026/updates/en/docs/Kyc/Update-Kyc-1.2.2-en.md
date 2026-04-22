## 2026-04-21

**`Nano3.Kyc` Plugin Updates – Version 1.2.2**

### Radical Improvement to KYC Assessment Logic via Alternative Groups System and Code Restructuring

---

### Summary of Updates

In version **1.2.2**, a fundamental improvement was made to the KYC status assessment mechanism by addressing the limitations of the previous logic that required specific documents (e.g., requiring a `national_id` even when a `passport` existed). The update introduces an **Alternative Groups** system that allows any document from a given group (e.g., primary identity documents) to satisfy the mandatory requirement. Additionally, the assessment code has been completely restructured and moved to an independent trait (`HasAssessKycStatus`) to enhance maintainability and extensibility.

Key improvements include:

- **Alternative Groups system**: Ability to define groups such as `primary_identity` (passport, national ID, driver's license, etc.) and `address_verification` (utility bill, bank statement, etc.), where providing a single document from the group is sufficient.
- **More accurate assessment logic**: Modified `assessKycStatus` and `assessKycStatusByCategory` functions to handle alternative groups, resulting in more realistic assessments aligned with actual KYC compliance standards.
- **Code restructuring**: Extracted assessment methods and their helpers into the `HasAssessKycStatus` trait, reducing the size of the `Manager` class and making KYC logic easier to test and modify independently.
- **Improved `getMandatoryTypesForParty`**: Now returns requirements as special strings like `group:primary_identity` instead of specific types, allowing great flexibility in defining requirements based on risk level and party type.
- **Fixed category assessment (`assessKycStatusByCategory`)**: Updated document retrieval to include all documents of the category, ensuring existing documents appear in the assessment results, rather than being limited to the required types only.

---

### Version Goals

- **Make KYC assessment more flexible and realistic** by allowing alternatives for mandatory documents, aligning with best practices in compliance systems.
- **Improve accuracy of completion percentage (`completion_percentage`) and overall score (`overall_score`)** to accurately reflect the user's status based on satisfied groups.
- **Facilitate maintenance and development of KYC logic** by separating it into an independent trait (`HasAssessKycStatus`).
- **Fix the issue of documents not appearing in category assessment** (e.g., passport previously did not appear in `kyc_status_by_category` when assessing `primary_id`).
- **Provide clearer recommendations** to users about missing requirements, with examples of required documents.

---

### New Features

#### 1. `HasAssessKycStatus` Trait

A new trait `Nano3\Kyc\Classes\Manager\HasAssessKycStatus` was created, containing the entire KYC assessment logic:

| Method | Description |
| :--- | :--- |
| `assessKycStatus()` | General KYC status assessment for a given owner (individual/company). |
| `assessKycStatusByCategory()` | KYC status assessment based on a specific document category (`primary_id`, `address`, ...). |
| `getEmptyAssessmentData()` | Empty assessment data structure. |
| `processMandatoryRequirements()` | Processes mandatory requirements and separates groups from individual types. |
| `analyzeDocuments()` | Analyzes the document set and extracts statistics and data. |
| `determineSatisfiedGroups()` | Determines which alternative groups have been satisfied. |
| `determineMissingRequirements()` | Determines missing requirements (whether individual types or groups). |
| `buildRecommendations()` | Builds a list of recommendations based on analysis results. |

The trait is included in the `Manager` class using `use \Nano3\Kyc\Classes\Manager\HasAssessKycStatus;`, preserving the public interface of `Manager` without change.

#### 2. Alternative Groups System in `DocumentType`

New methods were added to `DocumentType` to support alternative groups:

| Method | Description |
| :--- | :--- |
| `getAlternativeGroups()` | Returns the array of alternative groups, such as `primary_identity` and `address_verification`. |
| `isInAlternativeGroup($type, $groupKey)` | Checks whether a given document type belongs to a specific group. |
| `getAlternativeGroupKey($type)` | Returns the alternative group key to which the document type belongs. |

**Default group definitions:**

```php
'primary_identity' => [
    TYPE_PASSPORT, TYPE_NATIONAL_ID, TYPE_DRIVERS_LICENSE,
    TYPE_RESIDENCE_PERMIT, TYPE_REFUGEE_DOCUMENT
],
'address_verification' => [
    TYPE_UTILITY_BILL, TYPE_BANK_STATEMENT, TYPE_TAX_DOCUMENT,
    TYPE_RENTAL_AGREEMENT, TYPE_MORTGAGE_STATEMENT
]
```

These groups can be extended or overridden via the configuration file `config/nano3/kyc/document_types.php` in the future.

#### 3. Improved `DocumentType::getMandatoryTypesForParty`

The function was updated to return requirements based on alternative groups according to risk level:

```php
if ($partyType === 'individual') {
    $mandatory[] = 'group:primary_identity';
    if (in_array($riskLevel, ['medium', 'high', 'very_high'])) {
        $mandatory[] = 'group:address_verification';
    }
    if (in_array($riskLevel, ['high', 'very_high'])) {
        $mandatory[] = self::TYPE_PASSPORT;
    }
}
```

This means:
- `low` risk level: only one primary identity document required.
- `medium` risk level: primary identity document + proof of address document.
- `high` / `very_high` risk level: primary identity document + specifically a passport + proof of address document.

#### 4. Improvements in Assessment Functions

- **`assessKycStatus`**: Now calculates `completion_percentage` based on the number of satisfied requirements (groups and types) out of total mandatory requirements. Calculates `overall_score` based on the weights of individual documents that constitute the requirements.
- **`assessKycStatusByCategory`**: Fixed document retrieval to include all documents of the specified category (`whereDocumentCategory` only, without `whereDocumentType`), ensuring all relevant documents appear in the results. It also uses the same alternative group logic after filtering by category.

#### 5. Improved Recommendations

When a requirement like `group:address_verification` is unmet, the recommendation will show an example of an acceptable document (e.g., `utility_bill`) instead of displaying `group:address_verification` as raw text, making the message clearer to the user.

---

### Comparison of Results Before and After the Update

For the same user (who has only a verified passport):

| Indicator | Before version 1.2.2 | After version 1.2.2 |
| :--- | :--- | :--- |
| **`kyc_status.is_compliant`** | `false` (missing `national_id`) | `false` (missing proof of address, but primary identity is satisfied) |
| **`kyc_status.completion_percentage`** | `0%` | `50%` |
| **`kyc_status.total_documents_required`** | `1` | `2` |
| **`kyc_status.missing_required_types`** | `["national_id"]` | `["utility_bill"]` |
| **`kyc_status_by_category.is_compliant`** (category `primary_id`) | `false` | `true` |
| **`kyc_status_by_category.completion_percentage`** | `20%` | `100%` |
| **`kyc_status_by_category.verified_documents`** | `[]` (empty) | Contains passport |

These results reflect greater realism: the user has satisfied identity requirements but needs proof of address, and the category assessment shows completeness for primary identity.

---

### Upgrade Requirements

1. **Update code**:
   - Replace the following files with the new versions:
     - `classes/Manager.php` (to add `use HasAssessKycStatus`)
     - `classes/Manager/HasAssessKycStatus.php` (new)
     - `classes/DocumentType.php` (to add alternative group methods and modify `getMandatoryTypesForParty`)

2. **No new migrations** – no database changes.

3. **No configuration changes** – `config.php` remains the same (groups can be customized via configuration in the future).

4. **Update `version.yaml`**:
   Added version `1.2.2` with a description of the changes.

5. **Compatibility testing**:
   - Ensure that the `/assess` and `/assess/{category}` endpoints work correctly with the new results.
   - Verify the behavior of the `kyc_status` and `kyc_status_by_category` includes in `UserTransformer`.

6. **Run the tests** (in development environment):
   ```
   GET /api/v1/kyc/tests
   ```
   to ensure that the `KycPlusTest` suite still passes successfully.

---

### Benefits and Added Value

- **High flexibility**: KYC requirements can be easily customized by modifying alternative groups without changing the core logic.
- **Better accuracy**: The assessment reflects the user's actual status, reducing false rejections or requests for unnecessary documents.
- **Cleaner, more maintainable code**: By moving KYC logic to a separate trait, `Manager` is more organized and focused on its core responsibilities.
- **Improved user experience**: Recommendations and missing requirements are clearer.
- **Ready for expansion**: New alternative groups (e.g., `secondary_identity`) can be added easily.

---

### Conclusion

Version **1.2.2** represents a significant leap in the accuracy and flexibility of the KYC assessment system within the `Nano3.Kyc` add-on. By introducing the concept of alternative groups and restructuring the code, the add-on can now better simulate real-world compliance requirements while maintaining high performance and ease of customization. These improvements make `Nano3.Kyc` an even more powerful and production-ready solution for advanced identity verification management.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./docs/Kyc/Docs-HasAssessKycStatus-Trait-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)
