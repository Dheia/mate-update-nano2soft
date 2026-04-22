# DocumentType Class Documentation

## Overview

`Nano3\Kyc\Classes\DocumentType` is the central component for managing identity verification (KYC) document types within the `Nano3.Kyc` plugin for NanoSoft App. The class provides a unified and comprehensive definition for over **28 document types** distributed across **6 categories**, with advanced features including:

- Required fields and validation rules per type.
- Allowed file types and maximum file sizes.
- Trust weights (for automatic verification).
- Expiry support (expiry date or maximum age).
- Flexible field schemas for building dynamic forms.
- Integrated functions to map data between input and database.
- Full multi‑language support and customisation via `config` files.

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
| `TYPE_GOVERNMENT_ID` | `government_id` | Government ID |
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
| `CATEGORY_PRIMARY_ID` | `primary_id` | Primary ID |
| `CATEGORY_SECONDARY_ID` | `secondary_id` | Secondary ID |
| `CATEGORY_ADDRESS` | `address` | Proof of Address |
| `CATEGORY_CORPORATE` | `corporate` | Corporate Documents |
| `CATEGORY_UBO` | `ubo` | Ultimate Beneficial Owner |
| `CATEGORY_ADDITIONAL` | `additional` | Additional Verification |

---

## Functions for Fetching Types and Categories

### `getAllTypeCodes(): array`
Returns a list of all document type codes.

```php
$codes = DocumentType::getAllTypeCodes();
// ['passport', 'national_id', 'drivers_license', ...]
```

### `getCategories(): array`
Returns the categories with their localized names.

```php
$categories = DocumentType::getCategories();
// ['primary_id' => 'Primary ID', 'address' => 'Proof of Address', ...]
```

### `getAllTypesFlat(): array`
Returns a flat array `[code => localized_name]` for all types.

```php
$flat = DocumentType::getAllTypesFlat();
// ['passport' => 'Passport', 'national_id' => 'National ID Card', ...]
```

### `getAllTypesGrouped(bool $includeCategoryNames = true): array`
Returns types grouped by category.

```php
$grouped = DocumentType::getAllTypesGrouped();
/*
[
    'Primary ID' => ['passport' => 'Passport', ...],
    'Proof of Address' => ['utility_bill' => 'Utility Bill', ...],
]
*/
```

### `getTypesByCategory(string $category): array`
Returns types for a specific category.

```php
$addressTypes = DocumentType::getTypesByCategory(DocumentType::CATEGORY_ADDRESS);
// ['utility_bill' => 'Utility Bill', 'bank_statement' => 'Bank Statement', ...]
```

### `getOptionsForSelect(bool $grouped = false, ?string $categoryFilter = null): array`
Prepares options for dropdown selects.

```php
// Flat list
$options = DocumentType::getOptionsForSelect();

// Grouped by category
$options = DocumentType::getOptionsForSelect(true);

// Only address types
$options = DocumentType::getOptionsForSelect(false, DocumentType::CATEGORY_ADDRESS);
```

---

## Detailed Document Information Functions

### `getTypeDetails(string $type): ?array`
Returns comprehensive information about a document type.

```php
$details = DocumentType::getTypeDetails('passport');
/*
[
    'code' => 'passport',
    'name' => 'Passport',
    'category' => 'primary_id',
    'has_expiry' => true,
    'verification_weight' => 100,
    'required_fields' => ['document_front', 'document_number', ...],
    'allowed_mime_types' => ['image/jpeg', 'image/png', 'application/pdf'],
    'fields' => [ ... ], // full field schema
    ...
]
*/
```

### `isValidType(?string $type): bool`
Checks if a type code is valid.

```php
DocumentType::isValidType('passport');     // true
DocumentType::isValidType('invalid');      // false
```

### `getCategory(string $type): ?string`
Returns the category code.

```php
$cat = DocumentType::getCategory('utility_bill'); // 'address'
```

### `getTypeName(string $type, ?string $locale = null): string`
Localised type name.

```php
$name = DocumentType::getTypeName('passport', 'en'); // 'Passport'
```

### `getTypeDescription(string $type, ?string $locale = null): string`
Localised type description.

### `getConstantName(string $type): ?string`
Corresponding programming constant name (e.g. `TYPE_PASSPORT`).

---

## Advanced Property Functions

| Function | Description | Example |
| :--- | :--- | :--- |
| `getRequiredFields(string $type): array` | Mandatory fields | `['document_front', ...]` |
| `getAllowedMimeTypes(string $type): array` | Allowed file types | `['image/jpeg', 'application/pdf']` |
| `getMaxFileSizeKb(string $type): int` | Maximum file size in KB | `10240` |
| `requiresBackSide(string $type): bool` | Whether the document back is required | `true` / `false` |
| `requiresSelfieMatch(string $type): bool` | Whether selfie matching is required | `true` / `false` |
| `getExtractionRules(string $type): array` | OCR extraction rules | `['document_number' => [...]]` |
| `getVerificationWeight(string $type): int` | Document weight in scoring (0‑100) | `100` |
| `hasExpiry(string $type): bool` | Whether it has an expiry | `true` / `false` |
| `getMaxAgeDays(string $type): ?int` | Maximum age in days | `90` |

---

## Document Validity Functions

### `isDocumentAcceptable(string $type, $date): bool`
Checks whether a document is still acceptable (not expired and not beyond maximum age).

```php
// Passport expiring in 2028
$acceptable = DocumentType::isDocumentAcceptable('passport', '2028-12-31'); // true

// Bill issued 120 days ago (older than 90 days)
$acceptable = DocumentType::isDocumentAcceptable('utility_bill', Carbon::now()->subDays(120)); // false
```

### `isDocumentExpiring(string $type, $date, int $warningDays = 30): bool`
Checks whether a document is expired or about to expire.

```php
$expiring = DocumentType::isDocumentExpiring('passport', Carbon::now()->addDays(20), 30); // true
```

---

## Field Schema Functions

### `getFieldsSchema(string $type): array`
Returns the complete field schema for the type, including labels, types, required flags, and validation rules.

```php
$schema = DocumentType::getFieldsSchema('passport');
/*
[
    'document_front' => [
        'label' => 'Passport Image',
        'type' => 'file',
        'required' => true,
        'validation' => ['mimes' => ['jpeg', 'jpg', 'png', 'pdf'], 'max_size_kb' => 10240],
        'order' => 10,
    ],
    'document_number' => [
        'label' => 'Passport Number',
        'type' => 'text',
        'required' => true,
        'validation' => ['max_length' => 50],
        'extractable' => true,
        'order' => 30,
    ],
    // ... all fields
]
*/
```

### `getFieldDefinition(string $type, string $field): ?array`
Returns the definition of a specific field.

```php
$def = DocumentType::getFieldDefinition('passport', 'expiry_date');
```

### `getFieldLabel(string $type, string $field, ?string $locale = null): string`
Localised label for a specific field.

```php
$label = DocumentType::getFieldLabel('passport', 'expiry_date', 'en'); // 'Expiry Date'
```

### `getAllFieldCodes(): array`
All field codes used across all types.

---

## Validation Functions

### `getValidationRules(string $type): array`
Returns validation rules for all fields of a document type, ready for use with `Validator`.

```php
$rules = DocumentType::getValidationRules('passport');
/*
[
    'document_front' => ['required', 'file', 'mimes:jpeg,jpg,png,pdf', 'max:10240'],
    'document_number' => ['required', 'string', 'max:50'],
    'expiry_date' => ['required', 'date', 'after_or_equal:today'],
    ...
]
*/
```

### `getValidationRulesForField(string $type, string $field): array`
Validation rules for a single field only.

```php
$rules = DocumentType::getValidationRulesForField('passport', 'expiry_date');
// ['required', 'date', 'after_or_equal:today']
```

---

## Data Mapping Functions Between Input and Database

These functions facilitate converting data from input forms to the database structure (separate columns + JSON) and vice versa.

### `getFieldColumnMapping(): array`
Returns a map of field names that have actual columns in the `nano3_kyc_documents` table.

```php
$map = DocumentType::getFieldColumnMapping();
// ['document_number' => 'document_number', 'full_name' => 'full_name', ...]
```

### `mapInputToDocumentData(string $type, array $input): array`
Transforms raw input data (e.g. `$request->all()`) into a structure suitable for saving.

**Output:** Array containing:
- `attributes`: values for actual columns.
- `fields_data`: values that will be stored in the JSON field.

**Example:**
```php
$input = [
    'document_number' => 'A12345678',
    'full_name' => 'John Doe',
    'expiry_date' => '2030-01-01',
    'passport_type' => 'P', // custom field not present as a column
];

$mapped = DocumentType::mapInputToDocumentData('passport', $input);

/*
[
    'attributes' => [
        'document_type' => 'passport',
        'document_category' => 'primary_id',
        'document_number' => 'A12345678',
        'full_name' => 'John Doe',
        'expiry_date' => '2030-01-01',
    ],
    'fields_data' => [
        'passport_type' => 'P',
    ],
]
*/

// Usage with the model
$document = new Document();
$document->fill($mapped['attributes']);
$document->fields_data = $mapped['fields_data'];
$document->save();
```

### `mapDocumentToOutput(string $type, $document): array`
The reverse operation: takes a model object and returns a unified array containing all fields (from columns + from `fields_data`). Useful for populating edit forms.

```php
$document = Document::find(1);
$formData = DocumentType::mapDocumentToOutput('passport', $document);

/*
[
    'document_number' => 'A12345678',
    'full_name' => 'John Doe',
    'expiry_date' => '2030-01-01',
    'passport_type' => 'P',
    // ... all other fields with their values or null
]
*/
```

---

## Recommendation and Mandatory Types Functions

### `isValidForPartyType(string $docType, string $partyType): bool`
Whether the document type is valid for a given party type (`individual`, `corporate`, `trust`).

```php
DocumentType::isValidForPartyType('certificate_of_incorporation', 'individual'); // false
DocumentType::isValidForPartyType('certificate_of_incorporation', 'corporate');  // true
```

### `getRecommendedTypes(string $partyType, string $riskLevel): array`
Returns recommended document types based on party type and risk level (`low`, `medium`, `high`).

```php
$recommended = DocumentType::getRecommendedTypes('individual', 'high');
// ['passport', 'national_id', 'utility_bill', ...]
```

### `getMandatoryTypesForParty(string $partyType, string $riskLevel): array`
Mandatory types that must be provided.

```php
$mandatory = DocumentType::getMandatoryTypesForParty('corporate', 'high');
// ['certificate_of_incorporation', 'tax_registration', 'ubo_declaration', ...]
```

---

## Caching Functions

### `clearCache(): void`
Clears the internal cache for settings and definitions. Used after updating configuration files at runtime.

```php
DocumentType::clearCache();
```

---

## Configuration via Settings File

Any property of document types can be customised via the configuration file `config/nano3/kyc/document_types.php`. Values in the file will override the default programmatic values.

**Example: `config/nano3/kyc/document_types.php`**

```php
<?php
return [
    'passport' => [
        'max_file_size_kb' => 5120, // 5MB instead of 10MB
        'requires_selfie_match' => false,
        'fields' => [
            'expiry_date' => ['required' => true],
            'document_back' => null, // hide the passport back field
            'religion' => ['required' => true], // make religion required
        ],
    ],
    'utility_bill' => [
        'max_age_days' => 60, // reduce validity to 60 days
    ],
];
```

After saving the file, calling `DocumentType::getTypeDetails('passport')` will automatically reflect the new values (after clearing the cache if necessary).

---

## Complete Practical Examples

### 1. Save a new document from an HTTP request

```php
use Nano3\Kyc\Classes\DocumentType;
use Nano3\Kyc\Models\Document;
use Validator;

public function store(Request $request)
{
    $type = $request->input('document_type');
    
    // 1. Validate the type
    if (!DocumentType::isValidType($type)) {
        throw new \Exception('Invalid document type');
    }
    
    // 2. Get validation rules and validate input
    $rules = DocumentType::getValidationRules($type);
    $validator = Validator::make($request->all(), $rules);
    $validated = $validator->validate();
    
    // 3. Map data to database structure
    $mapped = DocumentType::mapInputToDocumentData($type, $validated);
    
    // 4. Create and save the document
    $document = new Document();
    $document->fill($mapped['attributes']);
    $document->fields_data = $mapped['fields_data'];
    $document->owner_id = auth()->id();
    $document->owner_type = get_class(auth()->user());
    $document->status = 'pending';
    $document->save();
    
    // 5. Handle uploaded files (if any)
    if ($request->hasFile('document_front')) {
        $document->document_front()->create(['data' => $request->file('document_front')]);
    }
    
    return redirect()->back()->with('success', 'Document saved successfully');
}
```

### 2. Display an edit form for a document

```php
public function edit($id)
{
    $document = Document::findOrFail($id);
    $type = $document->document_type;
    
    // Get unified data (columns + JSON)
    $formData = DocumentType::mapDocumentToOutput($type, $document);
    
    // Get field schema to build the form
    $fieldsSchema = DocumentType::getFieldsSchema($type);
    
    return view('documents.edit', compact('document', 'formData', 'fieldsSchema'));
}
```

**In Blade template:**
```blade
<form method="POST">
    @foreach($fieldsSchema as $fieldCode => $fieldDef)
        <label>{{ DocumentType::getFieldLabel($type, $fieldCode) }}</label>
        
        @if($fieldDef['type'] === 'text')
            <input type="text" name="{{ $fieldCode }}" value="{{ $formData[$fieldCode] ?? '' }}">
        @elseif($fieldDef['type'] === 'date')
            <input type="date" name="{{ $fieldCode }}" value="{{ $formData[$fieldCode] ?? '' }}">
        @elseif($fieldDef['type'] === 'file')
            <input type="file" name="{{ $fieldCode }}">
        @endif
        <!-- etc... -->
    @endforeach
    <button type="submit">Update</button>
</form>
```

### 3. Check document validity before accepting it

```php
$document = Document::find($id);
$type = $document->document_type;

if (DocumentType::hasExpiry($type)) {
    $expiry = $document->expiry_date;
    if (!DocumentType::isDocumentAcceptable($type, $expiry)) {
        throw new \Exception('Document has expired');
    }
} elseif ($maxAge = DocumentType::getMaxAgeDays($type)) {
    $issue = $document->issue_date;
    if (!DocumentType::isDocumentAcceptable($type, $issue)) {
        throw new \Exception('Document is older than the allowed limit');
    }
}
```

### 4. Build a dynamic dropdown to select document type

```blade
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

---

## Important Notes

- **Caching**: The class uses internal caching. Any change to the configuration file during request execution will not be reflected until the cache is cleared with `DocumentType::clearCache()` or the page is reloaded in a development environment.
- **Language files**: Translation keys must be provided in `plugins/nano3/kyc/lang/en/lang.php` under the key `nano3.kyc::lang.document_types`.
- **Default values**: If no translation or configuration is found, the class falls back to well‑formatted programmatic values.

With this comprehensive documentation, developers can fully utilise the capabilities of the `DocumentType` class to build a flexible and extensible KYC system.
