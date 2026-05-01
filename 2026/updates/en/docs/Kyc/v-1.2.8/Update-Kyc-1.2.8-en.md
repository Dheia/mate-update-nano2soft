## 2026-04-30

**`Nano3.Kyc` Plugin Updates – Version 1.2.8**

### Summary of Updates

Version 1.2.8 serves as a comprehensive fix for the field merging and validation rule generation mechanism in the `DocumentType` class. Despite the success of version 1.2.7 in unifying the field structure with `FormFieldsManager`, practical testing revealed issues in the way default fields were merged with overrides and with the `document_types.php` configuration file settings. These issues led to unwanted fields appearing in some document types (e.g., corporate fields appearing in a passport), duplication of some fields, and incorrect or malformed validation rules.

This release focuses on **radically fixing the merging algorithms** to ensure:

- Unwanted fields are completely removed when set to `null` in overrides.
- Validation rules are merged intelligently, replacing duplicate rules instead of accumulating them.
- Fields are completely replaced when defined in the configuration file (`document_types.php`) rather than merged with default fields.
- Validation rules are generated correctly and fully compatible with Laravel's syntax, ready for direct use in `Validator::make`.

All of this while maintaining full backward compatibility and not breaking any existing APIs.

---

### Version 1.2.8 – Fixing Field Merging and Validation Rule Generation

#### Version Goals

- **Fix the `mergeFieldsWithOverrides` function** to support field deletion (`null`) and correct merging of validation rules.
- **Add a `mergeValidation` function** to replace duplicate rules instead of accumulating them, solving the issue of malformed `validation` values.
- **Fix the `loadConfig` function** to make the configuration file the sole source of fields when defined, preventing default fields from leaking into customized types.
- **Remove duplicates** in fields (e.g., `religion` and `passport_type`).
- **Correct validation rules** across all document types to reflect proper values (e.g., `max:50` instead of `max:100` or unintended values).

#### Improvements and Fixes

##### 1. Rewriting `mergeFieldsWithOverrides`

The previous implementation, which relied on `array_merge` and `unset` (which did not correctly delete fields in some cases), was replaced with a new implementation based on **indexing by the `id` key**:

```php
protected static function mergeFieldsWithOverrides(array $baseFields, array $extraFields = [], array $overrides = []): array
{
    // Convert baseFields to associative array keyed by id
    $fieldsAssoc = [];
    foreach ($baseFields as $field) {
        $id = $field['id'] ?? null;
        if ($id) {
            $fieldsAssoc[$id] = $field;
        }
    }

    // extraFields completely replace a field if its id exists
    foreach ($extraFields as $extraField) {
        $id = $extraField['id'] ?? null;
        if ($id) {
            $fieldsAssoc[$id] = $extraField;
        } else {
            $fieldsAssoc[] = $extraField;
        }
    }

    // overrides apply modifications or deletion (null)
    foreach ($overrides as $fieldId => $override) {
        if ($override === null) {
            unset($fieldsAssoc[$fieldId]);   // permanent deletion
            continue;
        }

        if (isset($fieldsAssoc[$fieldId])) {
            $base = $fieldsAssoc[$fieldId];
            foreach ($override as $key => $value) {
                if ($key === 'validation') {
                    $base['validation'] = self::mergeValidation($base['validation'] ?? [], $value);
                } else {
                    $base[$key] = $value;
                }
            }
            $fieldsAssoc[$fieldId] = $base;
        } else {
            $fieldsAssoc[$fieldId] = $override;
        }
    }

    $fields = array_values($fieldsAssoc);
    usort($fields, fn($a, $b) => ($a['sort_order'] ?? 999) <=> ($b['sort_order'] ?? 999));
    return $fields;
}
```

**Results**:
- Deleting fields specified as `null` in `overrides` now works reliably.
- Property merging is done precisely with special handling for `validation`.

##### 2. `mergeValidation` Function for Intelligent Validation Rule Merging

A new helper function was added that indexes existing rules by the `rule` key, then replaces a rule if its name is duplicated, preventing accumulation of duplicate rules and malformed values (e.g., `rule: "required", value: 100`).

```php
protected static function mergeValidation(array $baseValidation, array $newValidation): array
{
    $indexed = [];
    foreach ($baseValidation as $rule) {
        if (isset($rule['rule'])) {
            $indexed[$rule['rule']] = $rule;
        } else {
            $indexed[] = $rule;
        }
    }

    foreach ($newValidation as $rule) {
        if (isset($rule['rule'])) {
            $indexed[$rule['rule']] = $rule;   // replacement
        } else {
            $indexed[] = $rule;
        }
    }

    return array_values($indexed);
}
```

##### 3. Fixing `loadConfig` – Replace Fields When Defined in Configuration File

Previously, the `document_types.php` file merged its fields with the default fields, causing unwanted fields to appear. Now, if the `fields` key is defined in the configuration file, it **completely replaces** the default fields:

```php
protected static function loadConfig(): array
{
    // ...
    foreach ($fileConfig as $type => $settings) {
        if (isset($merged[$type])) {
            // If the file contains 'fields', we completely replace the fields
            if (isset($settings['fields'])) {
                $merged[$type]['fields'] = $settings['fields'];
                unset($settings['fields']);
            }
            $merged[$type] = array_replace_recursive($merged[$type], $settings);
        } else {
            $merged[$type] = $settings;
        }
    }
    // ...
}
```

##### 4. Removing Field Duplicates

The `religion` field was removed from `getDefaultFields` because it is added manually when needed via `extraFields` or `overrides`. Overrides were also adjusted to prevent duplicate fields such as `passport_type`.

##### 5. Correcting Validation Rules Across All Types

Thanks to the new `mergeValidation`, validation rules now reflect the correct values specified in `overrides` or the configuration file. For example:

- `document_number` → `['required', 'max:50']` (instead of previously malformed values).
- `issue_date` → `['required', 'date', 'before_or_equal:today']` (without extra `value`).
- `country_of_issue` → `['required']` (without incorrect `date` rules).

---

### Practical Examples

#### 1. `signature_specimen` Field – Before and After

**Before the fix (version 1.2.7)**:
The response for `GET /document-types-list/signature_specimen` returned **all default fields** (e.g., `issue_date`, `address_line1`, `company_name`, etc.) in addition to the `signature_image` field.

**After the fix (version 1.2.8)**:
The response is limited to the fields defined in `document_types.php` only:

```json
{
    "fields": [
        {
            "id": "signature_image",
            "type": "fileupload",
            "label": { "ar": "نموذج التوقيع", "en": "Signature Specimen" },
            "required": true,
            "validation": [
                { "rule": "required" },
                { "rule": "mimes", "value": "jpg,jpeg,png" },
                { "rule": "max", "value": 2048 }
            ]
        }
    ],
    "rules": {
        "signature_image": ["required", "file", "mimes:jpg,jpeg,png", "max:2048"]
    }
}
```

#### 2. `passport` Field – Clean Validation Rules

After the fixes, the validation rules for the `passport` field are as follows (excerpt):

```json
"rules": {
    "document_front": ["required", "file", "mimes:jpg,jpeg,png,pdf", "max:10240"],
    "document_number": ["required", "max:50"],
    "full_name": ["required", "max:100"],
    "issue_date": ["required", "date", "before_or_equal:today"],
    "expiry_date": ["required", "date", "after_or_equal:today"],
    "country_of_issue": ["required"],
    "nationality": ["required"]
}
```

Note the absence of erroneous rules such as duplicate `date:today` or `before_or_equal:today - 16 years` applied to `country_of_issue`.

---

### Version Summary (1.2.7 – 1.2.8)

| Version | Key Features and Fixes |
| :--- | :--- |
| 1.2.7 | Full compatibility with `FormFieldsManager`, automatic legacy field conversion, API enriched with resolved options. |
| 1.2.8 | **Radical fix for field merging and validation rules**, deletion of unwanted fields (`null`), intelligent merging of validation rules, prevention of default field leakage, duplicate removal, correction of all validation rules. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace `classes/DocumentType.php` with the new version (includes the improved `mergeFieldsWithOverrides`, `mergeValidation`, and `loadConfig` functions).
   - Ensure `getDefaultFields` is updated by removing the `religion` field if present.

2. **Test endpoints**:
   - Try `GET /api/v1/kyc/document-types-list/{type}?include_fields=1&include_rules=1` and verify that the displayed fields match expectations.
   - Test `POST /documents` and `PUT /documents/{id}` to ensure validation rules work correctly.

3. **Clear cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### Conclusion

Version 1.2.8 is an important stability release that addresses fundamental issues in the field merging mechanism that appeared after the transition to `FormFieldsManager`. Thanks to these fixes, developers can rely on field definitions in the configuration file with complete confidence, and validation rules in `Manager` will work as expected without errors.

We appreciate your valuable feedback that helped identify and resolve these issues. We are committed to continuing to improve the KYC add‑on to be powerful, flexible, and easy to use.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./docs/Kyc/Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./docs/Kyc/Docs-HasValidKycDocuments-Trait-en.md)
- [`Document` Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)
