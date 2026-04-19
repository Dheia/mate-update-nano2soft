## 2026-04-20

**`Nano3.Kyc` Plugin Updates – Versions 1.1.1 to 1.2.0**

### Summary of Updates

After adding KYC assessment functions and assessment endpoints in previous versions, the new updates focused on **enabling file upload and management** through full integration with the `Nano.FileUpload` package, as well as **developing a comprehensive testing system** to ensure add-on stability and facilitate bug detection. Updates from versions 1.1.1 to 1.2.0 included:

- **Integration of file upload into the API**: Support for uploading `document_front`, `document_back`, `signature_image`, `image`, `files` when creating and updating documents.
- **Protection of files after verification**: Prevent changing or deleting document files once verified (`is_verified = true`).
- **Registration of the `Document` model in `FileUploadRegistry`**: To enable advanced upload permissions and temporary file management.
- **Addition of a comprehensive test class `KycPlusTest`**: Covers `DocumentType`, `Manager`, file upload, KYC assessment, and API permission tests.
- **`GET /tests` endpoint**: To run the test suite in a development environment and display results uniformly.
- **General improvements to error handling and translation messages**.

---

### Version 1.1.1 – Minor Improvements to Assessment Endpoints

#### Version Goals

- Stabilize the `assessKycStatus` and `assessKycStatusByCategory` endpoints added in 1.1.0.
- Improve handling of `include_expired`, `include_pending`, `include_rejected` options with support for string values (`"true"`/`"false"`).

#### New Features

- Automatic conversion of inclusion options from strings to boolean values, allowing easy use in query strings.

---

### Version 1.1.2 – Supporting File Upload in API with Verified Document Protection

#### Version Goals

- Enable users to upload identity files (images, PDF, signature) directly via `create` and `update` endpoints.
- Prevent any modification to document files after verification (`is_verified = true`), preserving KYC data integrity.
- Automatically register the `Document` model in `FileUploadRegistry`.

#### New Features

##### 1. `FileUploadDocuments` Trait

A new trait `Nano3\Kyc\APIControllers\Documents\FileUploadDocuments` was created containing the following functions:

| Function | Description |
| :--- | :--- |
| `extractUploadedFiles()` | Extract files from request data (whether `UploadedFile` or base64). |
| `attachFilesToDocument()` | Upload files via `FileUploadService` and attach them to the document. |
| `documentHasFile()` | Check if a file is associated with a specific field. |
| `registerDocumentModel()` | Register the `Document` model in `FileUploadRegistry`. |

##### 2. Updates to `create` and `update` Functions

- **`create`**: After creating the document, attached files are uploaded and attached. If some file uploads fail, the entire operation does not fail; instead, a warning is returned with error details.
- **`update`**: Before allowing new file uploads, the document's status is checked. If `is_verified = true` and it already has an associated file, the request is rejected with an appropriate error message.

##### 3. Registering `Document` in `FileUploadRegistry`

In `Plugin::boot()`, `registerDocumentInFileUpload()` is called to register the following fields:
- `document_front`, `document_back`, `signature_image`, `image` (type `image`, max 10MB for images).
- `files` (type `multiple`, allows multiple files).

This enables `FileUploadService` to automatically apply upload permissions and constraint checks.

#### Benefits

- Unified user experience for uploading KYC files without needing a separate call to the `FileUpload` API.
- Strong protection for verified documents against tampering.
- Seamless integration with `Nano.FileUpload`'s permission and constraint system.

---

### Version 1.1.3 – Improvements to File Upload and Translation Messages

#### Version Goals

- Improve handling of files uploaded in base64 format.
- Add translation keys for partial success messages and verified file protection.
- Better error handling in file uploads.

#### New Features

- Support for fields ending with `_base64` to receive encoded files.
- New translation keys:
  - `public.helpers.create_document.msg_files_partial`
  - `public.helpers.update_document.msg_files_partial`
  - `public.helpers.update_document.msg_cannot_change_verified_files`
- Return upload error details in `process_data.file_errors`.

---

### Version 1.2.0 – Integrated Testing System (`KycPlusTest`)

#### Version Goals

- Provide a comprehensive testing mechanism covering all add-on components (DocumentType, Manager, FileUpload, API).
- Enable developers to easily run tests via an API endpoint in the development environment.
- Ensure add-on stability when making future changes.

#### New Features

##### 1. `KycPlusTest` Class

A test class located in `Nano3\Kyc\Tests\KycPlusTest` covering:

- **DocumentType**: Registration, field retrieval, validation rules.
- **Manager**: Create, update, verify, reject, delete, restore, fetch records, duplicate checking.
- **File Upload**: Create and update documents with files, prevent changing files of verified documents.
- **KYC Assessment**: `assessKycStatus` and `assessKycStatusByCategory`.
- **Permissions and Isolation**: API authentication verification and owner data isolation.

**Test result structure:**
Each test returns an array containing:
```json
{
    "code": 200,
    "status": true,
    "test_code": "DOCTYPE_REG_001",
    "name": "Document Type Registration",
    "description": "...",
    "message": "Test passed",
    "data": {...},
    "input_data": {...},
    "process_data": {...}
}
```

##### 2. `GET /api/v1/kyc/tests` Endpoint

Allows running the test suite and displaying results. **Available only** when `app.debug = true` and the user is of type `backend`.

**Query parameters:**
| Parameter | Description |
| :--- | :--- |
| `test_version` | `v1` or `v2` (default `v2`). |
| `filter` | `all`, `passed`, `failed` (default `all`). |

**Example request:**
```
GET /api/v1/kyc/tests?test_version=v2&filter=passed
```

**Response:** Contains `data` (filtered test results) and `meta.summary` with statistics (passed, failed, success rate).

##### 3. Test Environment Isolation

Tests use `Db::beginTransaction()` and `Db::rollBack()` to ensure they do not affect the real database. Uploaded files are also cleaned up temporarily.

#### Benefits

- Early bug detection before deployment.
- Living documentation of add-on behavior through tests.
- Easy verification of installation correctness in different environments.

---

### Version Summary (1.1.1 – 1.2.0)

| Version | Key Features |
| :--- | :--- |
| 1.1.1 | Improved assessment endpoints. |
| 1.1.2 | Supported file upload in API, prevented changes to verified document files, registered Document in FileUploadRegistry. |
| 1.1.3 | Improved base64 handling and translation messages. |
| 1.2.0 | Added comprehensive test system (`KycPlusTest`) and `/tests` endpoint. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace the following files with the new versions:
     - `Plugin.php`
     - `apicontrollers/Documents.php`
     - `apicontrollers/documents/FileUploadDocuments.php`
     - `tests/KycPlusTest.php` (new)

2. **Add the tests route** in `routes.php`:
   ```php
   Route::get('tests', 'Documents@tests');
   ```

3. **Update the translation file** `lang.php` with the new keys (see version 1.1.3).

4. **Run the tests** (in development environment):
   ```
   GET /api/v1/kyc/tests
   ```
   to verify the upgrade was successful.

5. **No new migrations** in these versions.

---

### Conclusion

With versions 1.1.1 to 1.2.0, the `Nano3.Kyc` add-on has become fully integrated with the `Nano.FileUpload` file management system, providing a seamless experience for managing KYC documents with advanced protection for verified data. The addition of a comprehensive testing system enhances confidence in the add-on's stability and facilitates future maintenance and development. These improvements make the add-on ready for production use while ensuring the highest quality standards.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`Document` Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)
- [`DynamicAddIncludeKyc` Behaviors Documentation](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-en.md)

