## 2026-04-27 - 2026-04-28

**`Nano3.Kyc` Plugin Updates – Version 1.2.7**

### Summary of Updates

After launching the new endpoints for categories and types in version 1.2.6, version 1.2.7 introduces a fundamental improvement to the **structure of form fields** for document types. This release aims to unify the mechanism for defining, displaying, and validating fields by integrating `FormFieldsManager` – the unified form field management system in the NanoSoft platform.

This release focuses on:

- **Restructuring field definitions** in `DocumentType` to comply with `FormFieldsManager` standards (sequential array instead of associative).
- **Smart conversion functions** to ensure full backward compatibility (Legacy Support) without breaking any existing functionality.
- **Improved validation rule performance and accuracy** by delegating the task to `FormFieldsManager`.
- **Enriching API responses** with resolved options for dropdown lists.

These improvements make the KYC system more integrated with other platform components and open the door to using unified tools for building dynamic forms in the control panel and APIs.

---

### Version 1.2.7 – Full Compatibility with `FormFieldsManager` and Document Fields Restructuring

#### Version Goals

- Convert all field definitions in `DocumentType` and the `document_types.php` configuration file to the `FormFieldsManager` format (sequential array).
- Provide automatic conversion functions (`convertLegacyFieldsToNewFormat`, `convertLegacyValidation`, etc.) to ensure existing systems do not break.
- Delegate field validation operations to `FormFieldsManager` to unify the logic.
- Improve the `GET /document-fields/{type}` endpoint to return fields with resolved options (e.g., country list from API).
- Fix the configuration file loading issue due to an extra dot in `CONFIG_KEY`.

#### New Features

##### 1. Converting Field Definitions to `FormFieldsManager` Format

The field structure in `config/nano3/kyc/document_types.php` has been changed from:

```php
// Old (associative array)
'fields' => [
    'document_number' => [
        'label' => 'Document Number',
        'type' => 'text',
        'required' => true,
        'validation' => ['max_length' => 50],
    ],
]
```

To the new sequential format supported by `FormFieldsManager`:

```php
'fields' => [
    [
        'id' => 'document_number',
        'label' => ['ar' => 'Document Number', 'en' => 'Document Number'],
        'type' => 'text',
        'required' => true,
        'validation' => [
            ['rule' => 'required'],
            ['rule' => 'max', 'value' => 50],
        ],
    ],
]
```

This change makes field definitions compatible with unified form management tools in the platform.

##### 2. Smart Legacy Conversion Functions

To ensure systems still relying on the old format are not affected, automatic conversion functions have been added to `DocumentType`:

| Function | Description |
| :--- | :--- |
| `convertLegacyFieldsToNewFormat($legacyFields)` | Converts the old associative field array to the new sequential format. |
| `normalizeMultilingualValue($value)` | Converts any text to a multilingual array (`['ar' => ..., 'en' => ...]`). |
| `convertOptionsToDataSource($optionType)` | Converts old text options (e.g., `'countries'`) to a `data_source` definition compatible with `FormFieldsManager`. |
| `convertLegacyValidation($legacyValidation)` | Converts validation rules from the associative format to the `FormFieldsManager` format (array of `['rule' => ..., 'value' => ...]`). |

Thanks to these functions, `DocumentType` can read both old and new formats and always return fields in the unified format.

##### 3. Enhanced and New Functions in `DocumentType`

| Function | Description |
| :--- | :--- |
| `getFieldsSchema($type)` | **Enhanced**: Automatically detects field format (old/new) and returns them always in the sequential format after normalization via `FormFieldsManager`. |
| `getFieldDefinition($type, $field)` | **Enhanced**: Searches the sequential array for the field using the `id` key. |
| `getValidationRules($type)` | **Enhanced**: Uses `FormFieldsManager` to generate validation rules instead of custom logic. |
| `getDefaultFields()` | **Rewritten**: Returns default fields in the new sequential format. |
| `buildFieldsForType($customFields, $overrideDefaults)` | **Rewritten**: Handles sequential arrays and applies overrides based on `id`. |
| `mapInputToDocumentData($type, $input)` | **Enhanced**: Uses the field `id` to extract data. |
| `mapDocumentToOutput($type, $document)` | **Enhanced**: Rebuilds output using field `id`s. |
| `getFieldLabel($type, $field, $locale)` | **Enhanced**: Extracts the required text from the multilingual `label` array. |
| `getFormFieldsManager()` | **New**: Creates an instance of `FormFieldsManager` with supported languages (`ar`, `en`). |

##### 4. Improvement of the `GET /document-fields/{type}` Endpoint

The `Documents::getDocumentFields` function has been updated to:

- Fetch normalized fields from `DocumentType::getFieldsSchema`.
- Enrich fields with resolved options using `FormFieldsManager::enrichWithOptions`.
- Return data in a unified format supported by `FormFieldsManager`.

This allows front-end applications to obtain ready-to-use dropdown lists (e.g., country and nationality lists) directly within the field definition.

##### 5. Fix Configuration File Loading

The `loadConfig` function has been corrected to remove the extra dot from `CONFIG_KEY` (`nano3.kyc::document_types.` ← `nano3.kyc::document_types`), ensuring the `document_types.php` file loads correctly.

---

### Practical Examples

#### 1. Getting Passport Fields with Resolved Options

**Request:**
```http
GET /api/v1/kyc/document-fields/passport HTTP/1.1
Authorization: Bearer ...
```

**Response (excerpt):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "type": "passport",
        "type_name": "Passport",
        "fields": [
            {
                "id": "document_front",
                "type": "fileupload",
                "label": { "ar": "Document Front", "en": "Document Front" },
                "required": true,
                "validation": [
                    { "rule": "required" },
                    { "rule": "mimes", "value": "jpeg,jpg,png,pdf" }
                ]
            },
            {
                "id": "country_of_issue",
                "type": "select",
                "label": { "ar": "Country of Issue", "en": "Country of Issue" },
                "data_source": {
                    "type": "api",
                    "endpoint": "api/v1/location/countries"
                },
                "resolved_options": [
                    { "id": "YE", "name": "Yemen" },
                    { "id": "SA", "name": "Saudi Arabia" },
                    ...
                ]
            }
        ]
    }
}
```

#### 2. Using the Old Format (Automatically Converted)

If you have an old field definition (associative array) from a previous version, `DocumentType` will automatically convert it to the new format without any developer intervention.

```php
// Old definition
$oldFields = [
    'document_number' => ['type' => 'text', 'required' => true]
];

// Internally uses convertLegacyFieldsToNewFormat()
$newFields = DocumentType::getFieldsSchema('some_type');
```

---

### Version Summary (1.2.6 – 1.2.7)

| Version | Key Features |
| :--- | :--- |
| 1.2.6 | New endpoints for categories and types, advanced `DocumentType` functions, improved error handling, standardized API outputs. |
| 1.2.7 | Restructured document fields to comply with `FormFieldsManager`, automatic legacy conversion functions, improved validation rules, enriched API fields with resolved options, fixed configuration loading. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace `classes/DocumentType.php` with the new version.
   - Replace `apicontrollers/Documents.php` with the new version.
   - Update the `config/nano3/kyc/document_types.php` file to the new format (optional but recommended to fully benefit from the features).

2. **Run database migrations** (no changes in this version).

3. **Test compatibility**:
   - Ensure that existing endpoints (`GET /document-fields/{type}`) return data in the new format.
   - Verify that document creation and update operations still work correctly (as they use `mapInputToDocumentData`).

4. **Clear cache** (optional):
   - After updating the configuration file, you may need to clear the application cache: `php artisan cache:clear`.

---

### Conclusion

With version 1.2.7, we have unified the KYC field management mechanism with the `FormFieldsManager` system used in other parts of the platform. This opens new horizons such as building dynamic forms in the control panel, generating unified validation rules, and easily using external data sources. The automatic conversion functions ensure a smooth upgrade without any negative impact on existing systems.

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
