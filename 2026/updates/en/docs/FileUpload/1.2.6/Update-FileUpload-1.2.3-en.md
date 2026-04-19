## 2026-04-15 – 2026-04-16

**`Nano.FileUpload` Plugin Update – Version 1.2.3**

### Adding New API Endpoints for Querying Modules, Permissions, and Validating Temporary Keys

---

### Summary of Updates

In version **1.2.3**, the add-on's API was enriched with several new endpoints that allow developers and external systems to:

- **Query registered modules** and their fields while respecting user permissions.
- **Fetch detailed settings** for a specific module or field.
- **Fetch field constraints** (e.g., max size, allowed types).
- **Fetch advanced processing options** (storage disk, automatic resizing, watermark).
- **Check permissions** at global, module, and field levels.
- **Integrated permission check** using the `validate()` method.
- **Validate and match a temporary key** via API (using the `validateAndMatchTempKey` function from the trait).

All new endpoints are protected by `oauth-users` and follow the unified response structure (`code`, `status`, `message`, `data`, ...).

---

### Version Goals

- **Enable developers to explore registered settings** in `FileUploadRegistry` without needing direct access to code or database.
- **Facilitate front-end integration** (e.g., React or Vue applications) by providing dynamic information about allowed fields and constraints.
- **Check permissions before attempting upload** to avoid rejection errors and improve user experience.
- **Provide an endpoint to validate temporary keys**, allowing front-ends to ensure key validity before using it.
- **Standardize the way of querying system information** via API instead of relying on internal tools.

---

### New Endpoints

#### 1. Get All Registered Modules (Filtered by Permission)

- **Path:** `GET /api/v1/fileupload/models`
- **Description:** Returns a list of registered modules that the current user has `view` permission on, with brief information about their fields (name, type, whether multiple).
- **Response:** Array of objects containing `class`, `label`, `enabled`, `allowed_user_types`, `fields`.

#### 2. Get Settings for a Specific Module

- **Path:** `GET /api/v1/fileupload/models/{modelClass}`
- **Description:** Returns the full settings of a specific module (after checking existence and `view` permission).
- **Response:** Contains `enabled`, `label`, `allowed_user_types`, `disabled_operations`, and `fields` (with brief information per field).

#### 3. Get Settings for a Specific Field

- **Path:** `GET /api/v1/fileupload/models/{modelClass}/fields/{field}`
- **Description:** Returns settings for a specific field (e.g., type, max size, allowed types).
- **Response:** Includes `type`, `label`, `multiple`, `required`, `max_filesize`, `allowed_types`, `use_caption`, `disabled_operations`.

#### 4. Get Field Constraints

- **Path:** `GET /api/v1/fileupload/models/{modelClass}/fields/{field}/constraints`
- **Description:** Returns the field constraints used in file validation (`max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`).
- **Benefit:** Front-ends can apply the same constraints before uploading.

#### 5. Get Advanced Processing Options for a Specific Field

- **Path:** `GET /api/v1/fileupload/processing-options/{modelClass}/{field}`
- **Description:** Returns the field's processing options: `storage_disk`, `auto_resize`, `resize_options`, `auto_watermark`, `watermark_options`.
- **Benefit:** Used primarily by advanced front-ends that need to know automatic transformation settings.

#### 6. Check if an Operation is Globally Enabled

- **Path:** `GET /api/v1/fileupload/permissions/global/{operation}`
- **Parameters:** `operation` can be `add`, `edit`, `delete`, `view`.
- **Response:** Returns `allowed` (true/false) and `disabled_globally` (opposite of `allowed`).

#### 7. Check Operation Permission at a Specific Module Level

- **Path:** `GET /api/v1/fileupload/permissions/model/{modelClass}/{operation}`
- **Response:** Returns `model_operation_enabled`, `user_type_allowed`, `can_proceed` (combination of both conditions).

#### 8. Check Operation Permission at a Specific Field Level

- **Path:** `GET /api/v1/fileupload/permissions/field/{modelClass}/{field}/{operation}`
- **Response:** Returns `field_operation_enabled`, `user_has_full_permission`, `can_proceed`.

#### 9. Integrated Permission Check (using `validate`)

- **Path:** `POST /api/v1/fileupload/permissions/check`
- **Data (JSON):**
  ```json
  {
    "model_class": "Nano\\Shop\\Models\\Product",
    "operation": "edit",
    "field": "image"
  }
  ```
- **Response:** Returns `allowed` (true/false) and an explanatory message.
- **Behavior:** Uses `$this->registry->validate()` which checks global, module, field, user type, and specific permissions, and returns the result without throwing an exception (the exception is caught and converted to a response).

#### 10. Validate and Match a Temporary Key

- **Path:** `POST /api/v1/fileupload/temp-key/validate`
- **Data (JSON):**
  ```json
  {
    "temp_key": "tmp_xxxx:yyyy",
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "strict_mode": true,
    "allow_expired_key": false,
    "expiry_grace_period": 300
  }
  ```
- **Description:** Calls the `validateAndMatchTempKey` function from the `HasFileUploadsMatchTempKey` trait and returns the result as JSON.
- **Response:** On success returns `valid: true`, `key_data`, and `matched_data`; on failure returns an error message with `error_code`.

---

### Usage Examples

#### Get modules available to the current user

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/models" \
  -H "Authorization: Bearer <token>"
```

#### Get settings for the `image` field in the `Product` module

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/models/Nano%5CShop%5CModels%5CProduct/fields/image" \
  -H "Authorization: Bearer <token>"
```

#### Check `edit` permission on a specific field

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/permissions/field/Nano%5CShop%5CModels%5CProduct/image/edit" \
  -H "Authorization: Bearer <token>"
```

#### Validate a temporary key

```bash
curl -X POST "https://yourdomain.com/api/v1/fileupload/temp-key/validate" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "temp_key": "tmp_eyJtb2RlbENsYXNzIjoiTmFub1xTaG9wXE1vZGVsc1xQcm9kdWN0Iiw...",
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "strict_mode": true
  }'
```

---

### Benefits and Added Value

- **Empower front-ends**: Front-end applications (e.g., Single Page Applications) can now dynamically discover available fields, constraints, and permissions, allowing them to build adaptive file upload forms.
- **Reduce server dependency**: Clients can check their permissions before attempting upload, reducing rejection requests and improving user experience.
- **Live settings documentation**: Developers can explore add-on settings via API instead of searching through code.
- **Additional security**: Permission endpoints do not expose sensitive information (e.g., storage keys) and are subject to the same user permissions.
- **Easy integration with external tools**: These endpoints can be used in CI/CD systems or content management tools.

---

### Upgrade Requirements

1. **Update code**:
   - Replace `FileUploadController.php` with the version containing the new methods.
   - Update `routes.php` to add the new routes.

2. **No new migrations**:
   - This version does not require database changes.

3. **No configuration changes**:
   - `config.php` remains the same.

4. **API permissions**:
   - All new endpoints are protected by `oauth-users`, so the user must have a valid token to access them.

5. **Compatibility testing**:
   - It is recommended to run the `FileUploadPlusTest` tests to ensure the new version does not affect existing functionality.
   - The new endpoints can be tested using tools like Postman or cURL.

---

### Conclusion

Version **1.2.3** represents a significant leap in the accessibility of file upload system information via API. By adding endpoints to query modules, fields, permissions, and validate temporary keys, the add-on has become more open and integrable with external systems and modern front-ends. These features make it easy to build rich applications that rely on `Nano.FileUpload` as a back-end for file management, while maintaining the highest levels of security and control.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)
