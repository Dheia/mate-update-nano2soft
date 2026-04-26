## 2026-04-23 - 2026-04-25

**`Nano3.Kyc` Plugin Updates – Version 1.2.5**

### Summary of Updates

After improving the performance of KYC relationships in the API (version 1.2.4) and adding advanced caching mechanisms (version 1.2.3), version 1.2.5 focuses on **enhancing document security and reliability** through an integrated protection system for verified documents. This release aims to:

- **Prevent modification of documents after verification** except with specific permissions.
- **Provide fine-grained permission control** through flexible settings (owner, backend user, special permissions).
- **Automatically calculate the verification score** upon verification.
- **Automatically set the verifier** based on the current user.
- **Prevent duplicate document types for the same owner** (optional).

These improvements make the KYC system more secure and ready for use in environments requiring strict regulatory compliance, while maintaining high flexibility for developers.

---

### Version 1.2.5 – Verified Document Protection System and Security Enhancements

#### Version Goals

- Develop an integrated protection system that prevents modification of verified (`is_verified = true`) and valid documents.
- Provide professional helper functions to check modification permission (`canModifyDocument`, `shouldProtectVerifiedDocument`).
- Support flexible settings to control protection behavior (`protect_verified` in `config.php`).
- Integrate the `Backend` permission system to grant exceptions for administrative users.
- Improve the `prepareChangesIsVerified` function to automatically calculate `verification_score`.
- Add the `preperSetVerifier` function to automatically set the verifier upon verification.
- Enable the option to prevent duplicate document types for the same owner (`is_check_duplicate`).
- Add the `isDocumentValid` function to check document validity (expiry / max age).

#### New Features

##### 1. Verified Document Modification Prevention System (`protect_verified`)

A new section has been added to the `config.php` settings file named `protect_verified`, providing complete control over verified document modification policies:

```php
'protect_verified' => [
    'is_check' => true,                     // Enable/disable protection entirely
    'allow_owner_modify_verified' => false, // Allow the document owner to modify it
    'backend_permissions' => 'nano3.kyc.documents.edit_verified', // Special backend permission
    'allowed_fields' => [                   // Fields allowed for modification on verified documents
        'verification_score',
        'is_verified',
        'status',
        'metadata',
        'other_data',
        'config_data',
    ],
],
```

| Setting | Description | Default Value |
| :--- | :--- | :--- |
| `is_check` | Enable protection entirely. If `false`, any document can be modified. | `true` |
| `allow_owner_modify_verified` | Allow the document owner to modify it even if verified. | `false` |
| `backend_permissions` | Permission (or permissions) required for a `Backend` user to bypass protection. Can be a single string or comma-separated. | `'nano3.kyc.documents.edit_verified'` |
| `allowed_fields` | The only fields allowed to be modified when protection applies (for unauthorized users). | List of safe fields like `metadata` and `status`. |

**How It Works:**

1. When attempting to save (update) a document, the `protectVerifiedDocument()` function is called within `beforeSave`.
2. The function checks `canModifyDocument()` to determine if the current user is allowed to modify.
3. If allowed, full modification is permitted.
4. If disallowed, modification is completely blocked or only allowed for the fields specified in `allowed_fields` (depending on user type).

##### 2. New Protection and Validation Functions

| Function | Type | Description |
| :--- | :--- | :--- |
| `canModifyDocument(?Document $doc, $user)` | `public static` | Determines whether the user (or current user) is allowed to modify the document. Considers settings and permissions. |
| `shouldProtectVerifiedDocument(?Document $doc)` | `public static` | Quick check to see if protection should be applied to the document (regardless of user). |
| `isDocumentValid(Document $doc)` | `public static` | Checks document validity (future expiry date, within maximum age). |
| `checkBackendPermissions($user, $permissions)` | `protected static` | Checks backend user permissions using the OctoberCMS permission system. |
| `protectVerifiedDocument()` | `protected` | Applies protection during `beforeSave`, preventing unauthorized modification. |
| `prepareDuplicate()` | `public` | Prevents duplicate document types for the same owner (if `is_check_duplicate` is enabled). |
| `prepareChangesIsVerified()` | `public` | Handles `is_verified` changes, automatically calculates `verification_score`, and sets `verified_at`. |
| `preperSetVerifier($user)` | `public` | Automatically sets `verifier_id` and `verifier_type` from the current user upon verification. |

##### 3. Improved Verification Process

- When `is_verified = true` is set, `prepareChangesIsVerified` performs the following:
  - Sets `verified_at` to the current time.
  - Calls `preperSetVerifier()` to automatically set the verifier (if not already set).
  - Calculates `verification_score` from `DocumentType::getVerificationWeight()` if not manually specified.
  - Changes `status` to `verified` if the status is `pending`, `rejected`, or `draft`.

##### 4. Document Duplication Prevention (`is_check_duplicate`)

The `prepareDuplicate()` function has been improved to work according to the `is_check_duplicate` setting. When enabled, creating a new document of the same `document_type` for the same `owner` is prevented if it already exists (except for the same record when updating). This ensures no duplication of essential documents.

##### 5. Integration with the Permission System

- `Backend` users who have the permission `nano3.kyc.documents.edit_verified` (or the one specified in the settings) can modify verified documents without restrictions.
- If no permission is specified, all `Backend` users are allowed to modify verified documents (for internal use).
- The document owner (a `frontend` user) can also be allowed to modify their verified document via `allow_owner_modify_verified`.

---

### Practical Examples

#### 1. Regular user (frontend) attempting to modify a verified document

User `Ahmed` has a verified passport (`is_verified = true`). He tries to update the `document_number` field.

**Result:**  
The request is rejected with an error message:  
`"Cannot modify a verified and valid document."`

> If `allow_owner_modify_verified = true`, Ahmed would be allowed to modify.

#### 2. Backend user with the `nano3.kyc.documents.edit_verified` permission

The same `Backend` user (who has the permission) attempts to modify the document.

**Result:**  
The modification is allowed without restrictions.

#### 3. Attempting to create a duplicate document (with `is_check_duplicate = true`)

User `Ahmed` already has a `passport` document. He tries to create another `passport` document.

**Result:**  
The request is rejected with an error message:  
`"Cannot duplicate the same document type for the same owner."`

#### 4. Verifying a new document – automatic `verification_score` calculation

When verifying a `passport` document without specifying `verification_score`:

```php
$document->is_verified = true;
$document->save();
```

**Result:**  
`verification_score` is set to `100` (passport weight), the verifier is set to the current user, and the status becomes `verified`.

---

### Version Summary (1.2.4 – 1.2.5)

| Version | Key Features |
| :--- | :--- |
| 1.2.4 | Improved `DynamicAddIncludeKyc` behavior performance, added 7 new relationships (`kyc_verification_summary` and 6 category verifiers), support for `expiring_within_days`, comprehensive documentation updates. |
| 1.2.5 | Integrated verified document protection system, duplicate prevention, improved verification process (automatic score calculation, auto-set verifier), integration with backend permissions, flexible settings in `config.php`. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace the `models/Document.php` file with the new version.
   - Update the `config/config.php` file to add the `protect_verified` section.

2. **Run database migrations** (no changes in this version).

3. **Update the language file**:
   - Add the following keys to `lang.php`:
     ```php
     'msg.cannot_modify_verified_document' => 'Cannot modify a verified and valid document.',
     'msg.cannot_modify_verified_document_field' => 'Cannot modify the ":field" field for a verified and valid document.',
     ```

4. **Review permissions**:
   - Ensure that the permission `nano3.kyc.documents.edit_verified` is granted to administrative users who need to modify verified documents.

5. **Test the protection**:
   - Try modifying a verified document as a `frontend` user to ensure the operation is rejected.
   - Try the same operation as a `Backend` user with the appropriate permission.

---

### Conclusion

With version 1.2.5, we have added an integrated security layer that prevents tampering with documents after verification, while maintaining full flexibility for developers through precise settings. The verification experience has also been improved so that routine operations (score calculation, setting the verifier) happen automatically. These improvements make the `Nano3.Kyc` add-on suitable for the most complex and demanding security and compliance scenarios.

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
