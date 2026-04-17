## 2026-3-15 - 2026-04-03

**Launch of `Nano.FileUpload` Plugin – A Centralized File Upload Management System for NanoSoft Applications**

### Launch of `Nano.FileUpload` Plugin – A Centralized File Upload Management System

The **`Nano.FileUpload`** plugin has been launched as an independent software plugin within the NanoSoft system, aiming to unify and manage file upload operations across API endpoints in all system plugins. The plugin provides an integrated structure for registering models that contain upload fields, verifying user permissions for different user types (backend/frontend), managing temporary files before record saving, and providing ready-to-use API endpoints with a unified response structure.

The plugin is designed to operate independently from applications, allowing any other plugin to easily register with it via dedicated methods or events, ensuring reuse of upload and permission logic without code duplication.

---

### 1. Introduction

In response to the recurring need to add file upload fields in many NanoSoft plugins (such as products, users, orders, etc.), the `Nano.FileUpload` plugin has been developed as a centralized solution that manages:

- Registration of models and upload fields with settings (maximum size, allowed types, etc.).
- Specification of allowed user types (backend / frontend) at the model or field level.
- Assignment of granular permissions for each operation (add, edit, delete, view) via an integrated permission system.
- Support for temporary upload using temporary session keys to link files with unsaved models.
- Provision of a RESTful API with OAuth 2.0 authentication.

The plugin is built on separation of concerns using specialized classes (Registry, Service, UserManager, Controller), making it easily extensible and maintainable.

---

### 2. Update Objectives

- **Create an independent software plugin**: Provide `Nano.FileUpload` as a reusable package in all NanoSoft projects.
- **Central model registration**: Allow any model to be registered with its field settings via the `registerFileUploadFields` method in plugins or via the `nano.api.fileupload.registerModels` event.
- **Support user types and permissions**: Handle `backend` and `frontend` users separately, with the ability to customize permissions for each operation.
- **Temporary upload**: Enable file upload before model saving using temporary session keys, then link them after saving.
- **Unified API**: Provide endpoints for single and multiple uploads, deletion, and retrieval, with a uniform response structure and OAuth security.
- **Comprehensive documentation**: Prepare complete documentation in Arabic covering classes, examples, and endpoints (now translated to English).

---

### 3. Developed Components

#### 3.1 `Nano.FileUpload` Plugin

- **Name:** `Nano.FileUpload`
- **Namespace:** `Nano\FileUpload`
- **Goal:** Provide a centralized system for managing file uploads in NanoSoft applications with support for permissions and different user types.
- **Basic Structure:**
  - `classes/FileUploadRegistry.php` – Registry of registered models
  - `classes/FileUploadService.php` – Service layer to execute upload, delete, and retrieval operations
  - `classes/FileUploadUserManager.php` – Management of current user and permission verification
  - `controllers/FileUploadController.php` – API controller
  - `routes.php` – Endpoints
  - `Plugin.php` – Plugin initialization, route registration, and events
  - `lang/ar/lang.php` – Arabic language file
  - `docs/FileUpload/` – Documentation in Arabic (now translated)

#### 3.2 `FileUploadRegistry` Class

Located at `Nano\FileUpload\Classes\FileUploadRegistry`, this is the central registry that manages model definitions and upload fields.

**Core Methods:**

| Method | Description |
|--------|-------------|
| `registerModel($modelClass, $config)` | Register a model with its settings. |
| `getRegisteredModels()` | Retrieve all registered models. |
| `isModelRegistered($modelClass)` | Check if a model is registered. |
| `getModelConfig($modelClass)` | Get the configuration of a model. |
| `getFieldConfig($modelClass, $field)` | Get the configuration of a specific field. |
| `isUserTypeAllowed($modelClass, $userType)` | Check if a user type is allowed. |
| `can($modelClass, $operation, $userType, $user, $field)` | Check permission for a specific operation. |
| `getFieldConstraints($modelClass, $field)` | Get field constraints (size, types). |
| `updateModelConfig($modelClass, $config)` | Update the configuration of a registered model. |

**Model Configuration Structure:**

```php
[
    'enabled' => true,
    'allowed_user_types' => ['backend', 'frontend'],
    'permissions' => [
        'add'    => null,
        'edit'   => null,
        'delete' => null,
        'view'   => null,
    ],
    'fields' => [
        'image' => [
            'type' => 'image', // file, image, multiple
            'label' => 'Main image',
            'max_filesize' => 2048,
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
            'multiple' => false,
            'permissions' => [ /* override model permissions */ ]
        ],
    ],
    'defaults' => [ /* default settings */ ]
]
```

#### 3.3 `FileUploadService` Class

Located at `Nano\FileUpload\Classes\FileUploadService`, this class is responsible for executing file upload, deletion, and retrieval operations.

**Core Methods:**

| Method | Description |
|--------|-------------|
| `generateTempSessionKey($modelClass, $field, $userId)` | Generate a temporary session key. |
| `upload($modelClass, $field, $fileData, $options)` | Upload a single file. |
| `uploadMultiple($modelClass, $field, $filesData, $options)` | Upload multiple files. |
| `deleteFile($fileId, $modelClass, $field, $user)` | Delete a file with permission check. |
| `getFiles($modelClass, $field, $modelId, $options)` | Retrieve files associated with a model. |
| `attachTempFiles($model, $field, $tempSessionKey)` | Link temporary files to a saved model. |
| `validateFile($modelClass, $field, $fileData)` | Validate a file (size, type). |

#### 3.4 `FileUploadUserManager` Class

Located at `Nano\FileUpload\Classes\FileUploadUserManager`, this is the unified user and permission manager.

**Core Methods:**

| Method | Description |
|--------|-------------|
| `setUserResolver(callable $resolver)` | Set a custom user resolver. |
| `setUser($user)` | Set the user directly. |
| `getUser()` | Get the current user (with caching). |
| `clearUser()` | Reset the cached user. |
| `setPermissionChecker(callable $checker)` | Set a custom permission checker. |
| `getUserType()` | Determine the user type (backend/frontend/guest). |
| `getId()` | Get the current user ID. |
| `checkPermission($operation, $permissions, $user)` | Check permission. |

#### 3.5 `FileUploadController` Controller

Located at `Nano\FileUpload\APIControllers\FileUploadController`, it provides RESTful endpoints protected by OAuth. All methods call `FileUploadService` and return responses in a unified structure.

**Main Endpoints:**

| Route | Method | Description |
|-------|--------|-------------|
| `/upload` | POST | Upload a single file (multipart or base64). |
| `/upload-multiple` | POST | Upload multiple files. |
| `/delete/{id}` | DELETE | Delete a file. |
| `/files` | GET | Retrieve files associated with a model or temporary key. |

#### 3.6 Routing File (routes.php)

A comprehensive routing file was created, providing all endpoints within an external group with the base path `/api/v1/fileupload`, and an internal group protected by the `oauth-users` middleware.

---

### 4. Workflow (New Flow)

1. **Model Registration (once):**
   - In another plugin, the `registerFileUploadFields()` method is executed, or the plugin listens to the `nano.api.fileupload.registerModels` event.
   - `FileUploadRegistry` collects the definitions and registers them in memory.

2. **File Upload (via API):**
   - The client sends a `POST /upload` request with `model_class`, `field`, and the file (or base64).
   - `FileUploadController` verifies the existence of the model and field.
   - It calls `FileUploadService::upload` with the options.
   - `FileUploadService` checks permissions via `FileUploadRegistry::can` using the current user from `FileUploadUserManager`.
   - The file is uploaded via `Base64::onUpload` (from `Nano.Api`).
   - If no saved model is passed, a temporary session key is generated and returned in the response.

3. **Linking Temporary Files:**
   - After saving the model (e.g., a new product), the developer calls `FileUploadService::attachTempFiles` to move the temporary files to the model.

4. **Retrieving Files:**
   - `GET /files` can be called with `model_id` to retrieve files associated with a model, or with `temp_session_key` to retrieve temporary files.

---

### 5. Key Achievements and Features

- **Integrated Software Plugin:** `Nano.FileUpload` has been launched as a centralized solution for managing file uploads in all NanoSoft plugins.
- **Flexible Model Registration:** Support for registration via plugin methods or events, with detailed settings per field.
- **User Type Support:** Handle `backend` and `frontend` users separately, with the ability to specify `allowed_user_types` per model or field.
- **Granular Permissions:** Permissions for each operation (`add`, `edit`, `delete`, `view`) at the model or field level, integrating `backend` permission system (`hasAccess`/`hasPermission`) and allowing custom checks for `frontend`.
- **Temporary Upload:** Ability to upload files before saving a model using temporary session keys (`temp_session_key`), then link them later via `attachTempFiles`.
- **Ready-to-Use API:** RESTful endpoints with OAuth authentication and a unified response structure (`code`, `status`, `message`, `data`, ...).
- **Multiple Format Support:** Upload files via `multipart/form-data` or base64 string.
- **Professional Error Handling:** Display safe messages to the user in production, while logging full details in development.
- **Comprehensive Documentation:** Detailed documentation files in Arabic covering all classes, examples, and endpoints (now translated).

---

### 6. Benefits and Added Value

- **For Developers:**
  - Ability to add file upload fields in any plugin simply by registering.
  - Unifying upload and permission logic reduces code duplication.
  - Temporary upload support simplifies multi‑step model creation.
  - Unified and securely protected API.

- **For End Users:**
  - Smooth and consistent file upload experience across the application.
  - Clear error messages that help correct inputs.

- **For the System as a Whole:**
  - Enhanced security via permission checks before every operation.
  - High flexibility in managing user types.
  - Extensible structure for adding future features (e.g., automatic compression, image optimization).

---

### 7. Future Development Plans

- **Support Additional File Types:** Add special handling for specific types (PDF, documents).
- **Image Processing Improvements:** Support automatic image compression, watermarks.
- **Graphical Administration Interface:** Develop an admin interface to monitor uploaded files and manage permissions.
- **Cloud Service Integration:** Support direct upload to cloud storage services (AWS S3, etc.).
- **Webhook Notifications:** Send notifications after successful file uploads.

---

### 8. Conclusion

The launch of the `Nano.FileUpload` plugin represents a significant advancement in how file upload operations are managed in NanoSoft applications. By providing the `FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, and `FileUploadController` classes, developers can add file upload fields to any plugin quickly and securely, with full control over permissions and user types. The plugin is designed to be extensible, allowing advanced features to be added in the future without affecting current stability.

With this phase completed, the plugin is ready for use in various NanoSoft projects, and we look forward to continued development based on user feedback and evolving requirements.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)
