# DocumentType Class Documentation

## Overview

`Nano3\Kyc\Classes\DocumentType` is a central class for managing identity verification (KYC) document types within the `Nano3.Kyc` plugin for NanoSoft App. The class provides a unified definition for all document types, their properties, validation rules, required fields, and trust weights, with full multi‑language support and customisation via configuration files.

**Key Features:**
- Defines over **28 document types** distributed across **6 categories**.
- Loads settings from a `config` file with programmatic defaults.
- Supports translation via language files.
- Helper functions for checking document validity and extracting validation rules.
- Internal caching for better performance.

---

## Constants

### Document Types (`TYPE_*`)

| Constant | Value | Description |
| :--- | :--- | :--- |
| `TYPE_PASSPORT` | `passport` | Passport |
| `TYPE_NATIONAL_ID` | `national_id` | National ID Card |
| `TYPE_DRIVERS_LICENSE` | `drivers_license` | Driver's License |
| `TYPE_RESIDENCE_PERMIT` | `residence_permit` | Residence Permit |
| `TYPE_REFUGEE_DOCUMENT` | `refugee_document` | Refugee Document |
| `TYPE_VOTER_ID` | `voter_id` | Voter ID Card |
| `TYPE_MILITARY_ID` | `military_id` | Military ID |
| `TYPE_GOVERNMENT_ID` | `government_id` | Other Government ID |
| `TYPE_UTILITY_BILL` | `utility_bill` | Utility Bill |
| `TYPE_BANK_STATEMENT` | `bank_statement` | Bank Statement |
| `TYPE_TAX_DOCUMENT` | `tax_document` | Tax Document |
| `TYPE_RENTAL_AGREEMENT` | `rental_agreement` | Rental Agreement |
| `TYPE_MORTGAGE_STATEMENT` | `mortgage_statement` | Mortgage Statement |
| `TYPE_CERTIFICATE_OF_INCORPORATION` | `certificate_of_incorporation` | Certificate of Incorporation |
| `TYPE_ARTICLES_OF_ASSOCIATION` | `articles_of_association` | Articles of Association |
| `TYPE_BUSINESS_LICENSE` | `business_license` | Business License |
| `TYPE_TAX_REGISTRATION` | `tax_registration` | Tax Registration |
| `TYPE_SHAREHOLDER_REGISTER` | `shareholder_register` | Shareholder Register |
| `TYPE_BOARD_RESOLUTION` | `board_resolution` | Board Resolution |
| `TYPE_FINANCIAL_STATEMENTS` | `financial_statements` | Financial Statements |
| `TYPE_PROOF_OF_ADDRESS_BUSINESS` | `proof_of_address_business` | Proof of Business Address |
| `TYPE_UBO_DECLARATION` | `ubo_declaration` | UBO Declaration |
| `TYPE_OWNERSHIP_STRUCTURE_CHART` | `ownership_structure_chart` | Ownership Structure Chart |
| `TYPE_TRUST_DEED` | `trust_deed` | Trust Deed |
| `TYPE_SELFIE` | `selfie` | Selfie |
| `TYPE_LIVENESS_CHECK` | `liveness_check` | Liveness Check |
| `TYPE_VIDEO_VERIFICATION` | `video_verification` | Video Verification |
| `TYPE_SIGNATURE_SPECIMEN` | `signature_specimen` | Signature Specimen |

### Document Categories (`CATEGORY_*`)

| Constant | Value | Description |
| :--- | :--- | :--- |
| `CATEGORY_PRIMARY_ID` | `primary_id` | Primary Identity |
| `CATEGORY_SECONDARY_ID` | `secondary_id` | Secondary Identity |
| `CATEGORY_ADDRESS` | `address` | Proof of Address |
| `CATEGORY_CORPORATE` | `corporate` | Corporate Documents |
| `CATEGORY_UBO` | `ubo` | Ultimate Beneficial Owner |
| `CATEGORY_ADDITIONAL` | `additional` | Additional Verification Documents |

---

## Basic Query Methods

### `getAllTypeCodes(): array`

Returns a list of all defined document type codes.

**Example:**
```php
$types = DocumentType::getAllTypeCodes();
// ['passport', 'national_id', 'drivers_license', ...]
```

---

### `getCategories(): array`

Returns a list of all categories with their translated names.

**Example:**
```php
$categories = DocumentType::getCategories();
/*
[
    'primary_id'   => 'Primary Identity',
    'secondary_id' => 'Secondary Identity',
    'address'      => 'Address Verification',
    'corporate'    => 'Corporate Documents',
    'ubo'          => 'Beneficial Ownership',
    'additional'   => 'Additional Verification',
]
*/
```

---

### `getAllTypesFlat(): array`

Returns a flat array of `[code => localized_name]` for all document types.

**Example:**
```php
$flat = DocumentType::getAllTypesFlat();
// ['passport' => 'Passport', 'national_id' => 'National ID Card', ...]
```

---

### `getAllTypesGrouped(bool $includeCategoryNames = true): array`

Returns document types grouped by category.

**Parameters:**
| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `$includeCategoryNames` | `bool` | `true` | `true`: group key is translated category name, `false`: category code. |

**Example:**
```php
$grouped = DocumentType::getAllTypesGrouped();
/*
[
    'Primary Identity' => [
        'passport' => 'Passport',
        'national_id' => 'National ID Card',
        ...
    ],
    'Address Verification' => [
        'utility_bill' => 'Utility Bill',
        ...
    ],
    ...
]
*/
```

---

### `getTypesByCategory(string $category): array`

Returns document types belonging to a specific category.

**Parameters:**
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$category` | `string` | Category code (one of the `CATEGORY_*` constants). |

**Example:**
```php
$addressTypes = DocumentType::getTypesByCategory(DocumentType::CATEGORY_ADDRESS);
/*
[
    'utility_bill' => 'Utility Bill',
    'bank_statement' => 'Bank Statement',
    'tax_document' => 'Tax Document',
    'rental_agreement' => 'Rental Agreement',
    'mortgage_statement' => 'Mortgage Statement',
]
*/
```

---

### `getOptionsForSelect(bool $grouped = false, ?string $categoryFilter = null): array`

Prepares options for a dropdown select. Useful in backend forms and frontend interfaces.

**Parameters:**
| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `$grouped` | `bool` | `false` | Whether to return options grouped by category. |
| `$categoryFilter` | `?string` | `null` | Filter by a specific category. |

**Example 1 – Flat list for all types:**
```php
$options = DocumentType::getOptionsForSelect();
// ['passport' => 'Passport', 'national_id' => 'National ID Card', ...]
```

**Example 2 – Grouped by category:**
```php
$options = DocumentType::getOptionsForSelect(true);
/*
[
    'Primary Identity' => ['passport' => 'Passport', ...],
    'Address Verification' => ['utility_bill' => 'Utility Bill', ...],
]
*/
```

**Example 3 – Only address types:**
```php
$options = DocumentType::getOptionsForSelect(false, DocumentType::CATEGORY_ADDRESS);
// ['utility_bill' => 'Utility Bill', 'bank_statement' => 'Bank Statement', ...]
```

---

## Detailed Document Information Functions

### `getTypeDetails(string $type): ?array`

Returns comprehensive information about a specific document type, including name, description, category, required fields, validation rules, and more.

**Parameters:**
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$type` | `string` | Document type code. |

**Returns:** Information array or `null` if the type is invalid.

**Example:**
```php
$details = DocumentType::getTypeDetails(DocumentType::TYPE_PASSPORT);

/*
[
    'code' => 'passport',
    'name' => 'Passport',
    'description' => 'International passport issued by a national government.',
    'category' => 'primary_id',
    'category_name' => 'Primary Identity',
    'has_expiry' => true,
    'max_age_days' => null,
    'verification_weight' => 100,
    'required_fields' => ['document_front', 'document_number', 'issue_date', 'expiry_date', 'country_of_issue'],
    'allowed_mime_types' => ['image/jpeg', 'image/png', 'application/pdf'],
    'max_file_size_kb' => 10240,
    'allowed_for_entity' => ['individual'],
    'requires_back_side' => false,
    'requires_selfie_match' => true,
    'extraction_rules' => [
        'document_number' => ['type' => 'alphanumeric', 'pattern' => '/^[A-Z0-9<]+$/'],
        'expiry_date' => ['type' => 'date', 'min' => 'today'],
        'country_of_issue' => ['type' => 'country_code'],
    ],
    'is_primary_id' => true,
    'is_address_document' => false,
    'is_corporate_document' => false,
    'is_ubo_document' => false,
    'is_additional' => false,
]
*/
```

---

### `isValidType(?string $type): bool`

Checks whether a document type code is valid.

**Example:**
```php
DocumentType::isValidType('passport');        // true
DocumentType::isValidType('invalid_type');    // false
```

---

### `getCategory(string $type): ?string`

Returns the category code to which the type belongs.

```php
$cat = DocumentType::getCategory(DocumentType::TYPE_UTILITY_BILL);
// 'address'
```

---

### `getTypeName(string $type, ?string $locale = null): string`

Returns the translated name of the document type.

```php
$name = DocumentType::getTypeName('passport', 'ar');
// 'جواز سفر'
```

---

### `getTypeDescription(string $type, ?string $locale = null): string`

Returns the translated description of the document type.

```php
$desc = DocumentType::getTypeDescription('utility_bill');
// 'A recent utility bill showing your name and address...'
```

---

### `getConstantName(string $type): ?string`

Returns the corresponding programming constant name for the type code.

```php
$constant = DocumentType::getConstantName('national_id');
// 'TYPE_NATIONAL_ID'
```

---

## Advanced Property Functions

### `getRequiredFields(string $type): array`

Returns the fields that must be filled/extracted for this document type.

```php
$fields = DocumentType::getRequiredFields(DocumentType::TYPE_PASSPORT);
// ['document_front', 'document_number', 'issue_date', 'expiry_date', 'country_of_issue']
```

---

### `getAllowedMimeTypes(string $type): array`

Allowed file MIME types for upload.

```php
$mimes = DocumentType::getAllowedMimeTypes('passport');
// ['image/jpeg', 'image/png', 'application/pdf']
```

---

### `getMaxFileSizeKb(string $type): int`

Maximum file size in kilobytes.

```php
$maxKb = DocumentType::getMaxFileSizeKb('passport');
// 10240
```

---

### `requiresBackSide(string $type): bool`

Does the document require uploading a back‑side image?

```php
DocumentType::requiresBackSide('drivers_license');   // true
DocumentType::requiresBackSide('passport');          // false
```

---

### `requiresSelfieMatch(string $type): bool`

Does the document require matching a selfie with the document photo?

```php
DocumentType::requiresSelfieMatch('passport');   // true
DocumentType::requiresSelfieMatch('utility_bill'); // false
```

---

### `getExtractionRules(string $type): array`

Data extraction rules (OCR) per field. Used in automated verification processes.

```php
$rules = DocumentType::getExtractionRules('passport');
/*
[
    'document_number' => ['type' => 'alphanumeric', 'pattern' => '/^[A-Z0-9<]+$/'],
    'expiry_date' => ['type' => 'date', 'min' => 'today'],
    'country_of_issue' => ['type' => 'country_code'],
]
*/
```

---

### `getVerificationWeight(string $type): int`

The weight of the document in calculating the verification score (0‑100).

```php
$weight = DocumentType::getVerificationWeight('passport');
// 100
```

---

### `hasExpiry(string $type): bool`

Does the document have an expiry date?

```php
DocumentType::hasExpiry('passport');      // true
DocumentType::hasExpiry('utility_bill');  // false
```

---

### `getMaxAgeDays(string $type): ?int`

Maximum age in days for documents that do not have an expiry date (e.g. utility bill).

```php
$days = DocumentType::getMaxAgeDays('utility_bill');
// 90
```

---

## Document Validity Functions

### `isDocumentAcceptable(string $type, $date): bool`

Checks whether a document is still acceptable (not expired and not beyond maximum age).

**Parameters:**
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$type` | `string` | Document type code. |
| `$date` | `string\|DateTimeInterface` | Issue date (for age‑limited documents) or expiry date (for expirable documents). |

**Example 1 – Valid passport:**
```php
$expiryDate = '2028-12-31';
$acceptable = DocumentType::isDocumentAcceptable(DocumentType::TYPE_PASSPORT, $expiryDate);
// true
```

**Example 2 – Old utility bill:**
```php
$issueDate = Carbon::now()->subDays(120); // older than 90 days
$acceptable = DocumentType::isDocumentAcceptable(DocumentType::TYPE_UTILITY_BILL, $issueDate);
// false
```

---

### `isDocumentExpiring(string $type, $date, int $warningDays = 30): bool`

Checks whether a document is expired or about to expire within the warning period.

**Parameters:**
| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `$type` | `string` | - | Document type code. |
| `$date` | `string\|DateTimeInterface` | - | Issue/expiry date. |
| `$warningDays` | `int` | `30` | Number of days before expiry to consider it "expiring soon". |

**Example:**
```php
$expiryDate = Carbon::now()->addDays(20); // expires in 20 days
$expiring = DocumentType::isDocumentExpiring(DocumentType::TYPE_PASSPORT, $expiryDate, 30);
// true (because less than 30 days)
```

---

## Validation and Rules Functions

### `getValidationRules(string $type): array`

Returns validation rules ready for use with Laravel `Validator`.

**Parameters:**
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$type` | `string` | Document type code. |

**Example:**
```php
$rules = DocumentType::getValidationRules('passport');
/*
[
    'file' => ['required', 'file', 'mimes:jpeg,jpg,png,pdf', 'max:10240'],
    'document_number' => ['required', 'regex:/^[A-Z0-9<]+$/'],
    'expiry_date' => ['required', 'date', 'after_or_equal:today'],
]
*/
```

**Practical usage in a `Controller`:**
```php
$rules = DocumentType::getValidationRules($request->input('document_type'));
$validator = Validator::make($request->all(), $rules);
if ($validator->fails()) {
    // handle errors
}
```

---

### `isValidForPartyType(string $docType, string $partyType): bool`

Is the document type valid for the given party type (individual/corporate/trust)?

**Parameters:**
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$docType` | `string` | Document type code. |
| `$partyType` | `string` | Party type (`individual`, `corporate`, `trust`). |

**Example:**
```php
DocumentType::isValidForPartyType('certificate_of_incorporation', 'individual'); // false
DocumentType::isValidForPartyType('certificate_of_incorporation', 'corporate');  // true
```

---

## Recommendation and Mandatory Types Functions

### `getRecommendedTypes(string $partyType, string $riskLevel): array`

Returns a list of recommended document types based on party type and risk level.

**Parameters:**
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$partyType` | `string` | Party type (`individual`, `corporate`, `trust`). |
| `$riskLevel` | `string` | Risk level (`low`, `medium`, `high`). |

**Example:**
```php
$recommended = DocumentType::getRecommendedTypes('individual', 'high');
// ['passport', 'national_id', 'drivers_license', 'utility_bill', 'bank_statement']
```

---

### `getMandatoryTypesForParty(string $partyType, string $riskLevel): array`

Returns the **mandatory** document types that must be provided regardless of other factors.

**Example:**
```php
$mandatory = DocumentType::getMandatoryTypesForParty('corporate', 'high');
// ['certificate_of_incorporation', 'tax_registration', 'ubo_declaration', 'ownership_structure_chart']
```

---

## Field and Translation Functions

### `getFieldLabel(string $fieldCode): string`

Returns the translated label for a specific field (e.g. `document_number` → "Document Number").

```php
$label = DocumentType::getFieldLabel('expiry_date');
// 'Expiry Date' (or 'تاريخ الانتهاء' depending on language)
```

---

### `getAllFieldCodes(): array`

Returns all field codes used across all document types.

```php
$allFields = DocumentType::getAllFieldCodes();
// ['document_front', 'document_number', 'issue_date', 'expiry_date', 'country_of_issue', ...]
```

---

## Cache Management Functions

### `clearCache(): void`

Clears the internal cache for settings and definitions. Use when you need to reload settings from the `config` file (e.g. after updating settings at runtime).

```php
DocumentType::clearCache();
```

---

## Configuration and Customisation via Settings File

You can customise any document type’s properties via the configuration file `config/nano3/kyc/document_types.php`. Values in the file will override the programmatic defaults.

**Example customising passport:**
```php
// config/nano3/kyc/document_types.php
return [
    'passport' => [
        'max_file_size_kb' => 5120, // change max size to 5MB
        'requires_selfie_match' => false, // disable selfie matching
        'extraction_rules' => [
            'document_number' => ['type' => 'alphanumeric', 'pattern' => '/^[A-Z]{2}[0-9]{7}$/'],
        ],
    ],
];
```

After editing, calling `DocumentType::getTypeDetails('passport')` will reflect the new values.

---

## Complete Example: Document Upload Form

```php
use Nano3\Kyc\Classes\DocumentType;
use Carbon\Carbon;
use Validator;

class DocumentUploadController
{
    public function upload(Request $request)
    {
        $docType = $request->input('document_type');
        
        // 1. Validate the type
        if (!DocumentType::isValidType($docType)) {
            return response()->json(['error' => 'Invalid document type'], 400);
        }
        
        // 2. Get validation rules
        $rules = DocumentType::getValidationRules($docType);
        
        $validator = Validator::make($request->all(), $rules);
        if ($validator->fails()) {
            return response()->json(['errors' => $validator->errors()], 422);
        }
        
        // 3. Process uploaded file
        $file = $request->file('file');
        $mimeType = $file->getMimeType();
        $sizeKb = $file->getSize() / 1024;
        
        $allowedMimes = DocumentType::getAllowedMimeTypes($docType);
        $maxSize = DocumentType::getMaxFileSizeKb($docType);
        
        if (!in_array($mimeType, $allowedMimes)) {
            return response()->json(['error' => 'File type not allowed'], 400);
        }
        
        if ($sizeKb > $maxSize) {
            return response()->json(['error' => 'File too large'], 400);
        }
        
        // 4. Check validity of extracted data (simulated OCR)
        $extractedData = $this->ocrService->extract($file, $docType);
        $issueDate = $extractedData['issue_date'] ?? $extractedData['statement_date'] ?? null;
        
        if ($issueDate && !DocumentType::isDocumentAcceptable($docType, $issueDate)) {
            return response()->json(['error' => 'Document is too old or expired'], 400);
        }
        
        // 5. Save the document
        $document = $this->saveDocument($file, $docType, $extractedData);
        
        return response()->json(['success' => true, 'document_id' => $document->id]);
    }
}
```

---

## Example: Document Type Selection Interface

```php
// In a Blade template
<select name="document_type" class="form-control">
    @foreach(DocumentType::getOptionsForSelect(true) as $category => $types)
        <optgroup label="{{ $category }}">
            @foreach($types as $code => $name)
                <option value="{{ $code }}">{{ $name }}</option>
            @endforeach
        </optgroup>
    @endforeach
</select>
```

**Output (simplified HTML):**
```html
<select name="document_type">
    <optgroup label="Primary Identity">
        <option value="passport">Passport</option>
        <option value="national_id">National ID Card</option>
        ...
    </optgroup>
    <optgroup label="Address Verification">
        <option value="utility_bill">Utility Bill</option>
        ...
    </optgroup>
</select>
```

---

## Important Notes

- **Caching**: The class uses internal static caching. Any change to the configuration file during request execution will not be reflected until the cache is cleared with `DocumentType::clearCache()` or the page is reloaded in a development environment.
- **Language files**: Ensure translation keys exist in `plugins/nano3/kyc/lang/ar/lang.php` and `en/lang.php` under the key `nano3.kyc::lang.document_types`.
- **Default values**: If no translation or configuration is found, the class falls back to well‑formatted programmatic values.

---

This concludes the comprehensive documentation of the `DocumentType` class with practical examples covering most use cases.
