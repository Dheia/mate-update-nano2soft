# `StepPhotos` Trait Documentation

**Namespace:** `Nano\Orders\Traits\Steps`  
**Plugin:** `Nano.Orders`  
**Version:** 2.2.12  

---

## 📋 Introduction

The `StepPhotos` trait is a complete system for managing photos associated with an order, fully integrated with `Nano.FileUpload`. It is designed to provide a flexible and secure mechanism for uploading and deleting photos, as well as checking user permissions (order owner, delivery person, admin) based on the **current order status** and customisable per‑field rules.

This trait is used in the typical short‑term rental scenario, where the lessee and lessor take photos of the item **before delivery** to document its condition, and **after delivery** to prove proper return. Changing the order status to `DELIVERY` or `COMPLETE` is prevented if the required photos are missing, ensuring process integrity.

---

## 📌 Key Properties

| Property | Type | Description |
|----------|------|-------------|
| `$cachedPhotoRules` | `array\|null` | Static cache for photo rules after merging with settings (shared across all `OrderManager` instances). |

---

## 📌 Main Methods

### 1. Getting Photo Rules

#### `protected function getDefaultPhotoUploadRules(): array`
- **Purpose:** Returns the default rules for uploading photos (supports caching and config overrides).
- **Output:** An array containing rules for each of the four fields.

**Structure of rules per field:**

| Key | Type | Description |
|-----|------|-------------|
| `allowed_statuses` | `array` | List of status codes where photo upload is allowed (e.g., `['NEW', 'PROCESSING']`). |
| `allowed_roles` | `array` | Allowed roles (`user`, `delivery`, `admin`). |
| `required_on_transition` | `array` | Statuses for which photos must exist before transitioning to that status. |
| `max_files` | `int` | Maximum number of photos (for multi‑file fields). |
| `allowed_types` | `string` | Allowed file extensions (comma‑separated). |
| `max_size` | `int` | Maximum file size in KB. |
| `description` | `string` | Field description (for documentation or UI). |

**Example return value:**
```php
[
    'user_before_delivery' => [
        'allowed_statuses' => ['NEW', 'PROCESSING'],
        'allowed_roles'    => ['user', 'admin'],
        'required_on_transition' => ['DELIVERY'],
        'max_files'        => 10,
        'allowed_types'    => 'jpg,jpeg,png,gif',
        'max_size'         => 5120,
        'description'      => 'Lessee photos before delivery',
    ],
    // ... remaining fields
]
```

**Notes:**
- The default rules are merged with `Config::get('nano.orders::photo_rules', [])` if they exist.
- The result is stored in `self::$cachedPhotoRules` to avoid re‑merging within the same request.

---

### 2. Uploading Photos

#### `public function uploadOrderPhoto(string $field, mixed $fileData = null, array $options = []): array`
- **Purpose:** Upload a single photo to a specific field of the order.
- **Parameters:**
  - `$field`: field name (`user_before_delivery`, `user_after_delivery`, `delivery_before_delivery`, `delivery_after_delivery`).
  - `$fileData`: `UploadedFile` object (from `Input::file()`) or a base64 string.
  - `$options`: Additional options array (see table below).

**`$options` keys:**

| Key | Type | Description |
|-----|------|-------------|
| `order` | `Order\|null` | The order model (auto‑used if not passed). |
| `user` | `mixed` | The user performing the upload (auto‑fetched if not passed). |
| `skip_permission` | `bool` | Bypass permission checking (default `false`). |
| `title` | `string\|null` | Photo title (saved in `title`). |
| `description` | `string\|null` | Photo description. |
| `is_public` | `bool` | Whether the photo is public (default `false`). |
| `auto_resize` | `bool` | Enable automatic resizing. |
| `resize_options` | `array` | Resize settings (width, height, mode). |
| `auto_watermark` | `bool` | Enable watermark. |
| `watermark_options` | `array` | Watermark settings. |
| `custom_message` | `string\|null` | Custom success message. |
| `custom_error` | `string\|null` | Custom error message. |
| `allowed_statuses` | `array\|null` | Override allowed statuses. |
| `allowed_roles` | `array\|null` | Override allowed roles. |
| `required_on_transition` | `array\|null` | Override required transition statuses. |

**Execution steps:**

1. **Validate the field** – checks that `$field` is one of the allowed fields.
2. **Fetch file data** – if `$fileData` is not provided, tries to fetch it from `Input::file($field)`.
3. **Check that a file exists** – rejects if `null`.
4. **Determine the actor** – uses `resolveActor()` if `user` is not provided.
5. **Check permission** – calls `validatePhotoUploadPermission` (unless `skip_permission = true`).
6. **Prepare upload options** – builds `$uploadOptions` for `FileUploadService::upload`.
7. **Upload the file** – calls the file upload service.
8. **Process the result** – if successful, extracts `$newFile` from the result and fires the `nano.orders.photo.uploaded` event.
9. **Return the response** – a unified array containing `code`, `status`, `message`, `data`, `process_data`.

**Success response structure:**
```php
[
    'code' => 200,
    'status' => true,
    'message' => 'Photo uploaded successfully',
    'data' => [
        'id' => 123,
        'path' => 'https://.../image.jpg',
        'thumb' => 'https://.../thumb.jpg'
    ],
    'process_data' => [ ... ]
]
```

---

#### `public function uploadOrderPhotos(string $field, array $filesData = [], array $options = []): array`
- **Purpose:** Upload multiple photos in a batch to a multi‑file field.
- **Parameters:**
  - `$field`: field name.
  - `$filesData`: array of `UploadedFile` objects or base64 strings.
  - `$options`: same as `uploadOrderPhoto`.
- **Behaviour:**
  - If `$filesData` is not provided, tries to fetch it from `Input::file($field)`.
  - Iterates over each file and calls `uploadOrderPhoto` with `skip_permission = true` (because permission is checked only once).
  - Aggregates results and counts successes.
  - If all files succeed, `code = 200`; if only some succeed, `code = 207 (Partial Content)`.
- **Return:** array containing `data` (results for each file), `errors` (errors for each failed file), `process_data.success_count`.

---

### 3. Deleting Photos

#### `public function deleteOrderPhoto(string $field, int $fileId, array $options = []): array`
- **Purpose:** Delete an existing photo from a specific field.
- **Parameters:**
  - `$field`: field name.
  - `$fileId`: ID of the photo to delete.
  - `$options`: contains `order`, `user`, `skip_permission`.
- **Behaviour:**
  - Checks that the order and file exist.
  - If `skip_permission = false`, calls `validatePhotoUploadPermission` (to ensure the user has delete permission).
  - Deletes the file via `$file->delete()`.
  - Fires the `nano.orders.photo.deleted` event.
- **Return:** array containing `code`, `status`, `message`, `data.deleted_file_id`.

---

### 4. Permission Validation

#### `public function validatePhotoUploadPermission(string $field, mixed $user, Order $order, ?string $newStatus = null, array $options = []): void`
- **Purpose:** Check whether the user is authorised to upload a photo in the given field.
- **Exceptions:** Throws `ApplicationException` on failure with an appropriate message.

**Validation steps:**

1. **Get rules** – from `$options['rules']` or the default rules.
2. **Extract field rules** – if no rules exist for the field, upload is allowed (skip).
3. **Determine the user’s role** – via `getUserRoleForOrder` (returns `user`, `delivery`, `admin`, or `null`).
4. **Check allowed role** – if the role is not in `allowed_roles` and is not `admin`, reject.
5. **Check allowed status** – if the list is not empty and the current order status is not in it, reject.
6. **Check maximum count** – if `max_files` is set and the current number of photos equals or exceeds it, reject.
7. **Check new status requirement** – if `$newStatus` is passed and it is in `required_on_transition` and the current photo count is 0, reject.

**Examples of error messages:**
- `You do not have permission to upload photos for this order.`
- `User of type user is not allowed to upload photos in field delivery_before_delivery.`
- `Cannot upload photos in field user_before_delivery in current status (COMPLETE).`
- `Maximum number of photos (10) reached in field user_before_delivery.`
- `You must upload photos in field user_before_delivery before changing the status to DELIVERY.`

---

### 5. Quick Check Functions

#### `public function canUploadPhoto(string $field, mixed $user, Order $order): bool`
- **Purpose:** Check whether uploading a photo is possible (without throwing an exception).
- **Return:** `true` if allowed, `false` otherwise.

#### `public function canUploadPhotoToField(string $field, mixed $user, Order $order, ?string $newStatus = null): bool`
- **Purpose:** Same as above, with support for passing the new status (to check transition requirements).
- **Return:** `bool`.

---

### 6. Status‑Change Requirement Functions

#### `public function enforcePhotoRequirementsOnStatusChange(string $oldStatus, string $newStatus, Order $order, mixed $actor = null, array $options = []): void`
- **Purpose:** Called before changing the order status to ensure that the required photos are present.
- **Exceptions:** Throws `ApplicationException` if any field required for the transition to `$newStatus` has no photos.
- **Behaviour:**
  - Uses rules from `$options['rules']` or the default rules.
  - For each field, if `$newStatus` is in `required_on_transition` and the current photo count is 0, throws an exception.
  - If `$actor` is passed, also checks that the actor’s role is allowed to upload this field (optional, for extra security).

#### `public function hasRequiredPhotosForStatus(string $newStatus, Order $order, ?string $role = null): bool`
- **Purpose:** Quick check (without exception) whether all required photos for the new status are present.
- **Return:** `true` if present, `false` otherwise.
- **Use case:** UI (show/hide status change button).

---

### 7. Fetching Photo URLs

#### `public function getOrderPhotoUrls(string $field, array $thumbSizes = [], bool $includeModel = false): ?array`
- **Purpose:** Returns the URLs of photos for a given field, with custom thumbnail sizes.
- **Parameters:**
  - `$field`: field name.
  - `$thumbSizes`: array of size names and dimensions (e.g., `['thumb' => [150,150,'crop']]`).
  - `$includeModel`: whether to include the full file object in the result.
- **Return:**
  - If the field is multi‑file (`attachMany`): array of photos.
  - If single‑file (`attachOne`): a single photo.
  - If no photos: `null`.

**Example result:**
```php
[
    'id' => 456,
    'title' => 'Photo before delivery',
    'path' => 'https://.../original.jpg',
    'thumb' => 'https://.../thumb.jpg',
    'medium' => 'https://.../medium.jpg',
    'size' => 12345,
    'content_type' => 'image/jpeg',
    'created_at' => '2026-06-05 10:00:00',
]
```

#### `public function formatPhotoFile($file, array $thumbSizes = [], bool $includeModel = false): array`
- **Purpose:** Format a single file into an array containing basic data and thumbnails.
- **Internal use:** Called by `getOrderPhotoUrls`.

---

### 8. Internal Helper Functions

#### `protected function getUserRoleForOrder(mixed $user, Order $order): ?string`
- **Purpose:** Determine the user’s role relative to the order (admin, owner, delivery, or `null`).
- **Logic:**
  - If `$user` is of type `Backend\User` → `'admin'`.
  - If of type `RainLab\User\Models\User`:
    - If `user->id == $order->user_id` → `'user'`.
    - If `user->id == $order->delivery_user_id` → `'delivery'`.
  - Otherwise → `null`.

#### `protected function resolveActor(): mixed`
- **Purpose:** Fetch the current user (same logic as in `updateOrderStatusAdvanced`).
- **Order:**
  1. `\Nano\AuthApi\Classes\AuthHelpers::getCurrentUser()`
  2. `\BackendAuth::getUser()`
  3. `\Auth::getUser()`

#### `protected function isPhotoDuplicate(Order $order, string $field, $uploadedFile): bool`
- **Purpose:** Check whether a file with the same content (hash) already exists in the field.
- **Note:** Not used by default, but can be enabled via `$options['check_duplicate'] = true`.

---

### 9. Events

| Event Name | When Fired | Parameters |
|------------|------------|------------|
| `nano.orders.photo.uploaded` | After a photo is successfully uploaded | `['order' => $order, 'field' => $field, 'file' => $newFile, 'actor' => $actor]` |
| `nano.orders.photo.deleted` | After a photo is successfully deleted | `['order' => $order, 'field' => $field, 'file_id' => $fileId, 'actor' => $actor]` |

---

## 🧪 Practical Examples

### Example 1: Upload a single photo (order owner)

```php
$orderManager = OrderManager::instance();
$order = Order::find(123);
$orderManager->setOrderModel($order);

$uploadedFile = Input::file('user_photo');
$result = $orderManager->uploadOrderPhoto('user_before_delivery', $uploadedFile, [
    'title' => 'Lessee photo before delivery',
    'description' => 'Uploaded by the owner',
]);

if ($result['status']) {
    echo "Uploaded, photo ID: " . $result['data']['id'];
} else {
    echo "Upload failed: " . $result['error'];
}
```

### Example 2: Upload multiple photos (delivery person)

```php
$files = Input::file('delivery_photos'); // array
$result = $orderManager->uploadOrderPhotos('delivery_before_delivery', $files, [
    'user' => $deliveryUser,
]);

if ($result['status']) {
    echo "Uploaded " . $result['process_data']['success_count'] . " out of " . $result['process_data']['total'] . " photos.";
}
```

### Example 3: Delete a photo (admin)

```php
$result = $orderManager->deleteOrderPhoto('user_after_delivery', 456, [
    'skip_permission' => true, // bypass permission check
]);

if ($result['status']) {
    echo "Deleted photo " . $result['data']['deleted_file_id'];
}
```

### Example 4: Check permission before displaying the upload interface

```php
$currentUser = Auth::getUser();
$canUpload = $orderManager->canUploadPhoto('user_before_delivery', $currentUser, $order);

if ($canUpload) {
    // show upload form
} else {
    // hide or disable the form
}
```

### Example 5: Fetch photo URLs with custom thumbnails for API

```php
$photos = $orderManager->getOrderPhotoUrls('delivery_before_delivery', [
    'thumb' => [100, 100, 'crop'],
    'medium' => [300, 300, 'crop'],
]);

foreach ($photos as $photo) {
    echo "<img src='{$photo['thumb']}' alt='{$photo['title']}'>";
}
```

### Example 6: Prevent status change without required photos (integrated in `updateOrderStatusAdvanced`)

```php
// Attempt to change status to DELIVERY without uploading user_before_delivery
try {
    $result = $orderManager->updateOrderStatusAdvanced([
        'order' => $order,
        'order_states_ref_type' => 'DELIVERY',
    ]);
} catch (ApplicationException $e) {
    // Message: "You must upload photos in field user_before_delivery before changing the status to DELIVERY"
}
```

### Example 7: Bypass photo check (for administrators)

```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order' => $order,
    'order_states_ref_type' => 'COMPLETE',
    'skip_photo_check' => true,
    'admin_override' => true,
]);
```

---

## 🛠️ Customisation via Configuration (Config)

You can customise the rules for each field by adding a `photo_rules` array in `config/nano/orders.php`:

```php
'photo_rules' => [
    'user_before_delivery' => [
        'allowed_statuses' => ['NEW', 'PROCESSING'],
        'allowed_roles'    => ['user', 'admin'],
        'required_on_transition' => ['DELIVERY'],
        'max_files'        => 15,
        'allowed_types'    => 'jpg,jpeg,png,webp',
        'max_size'         => 10240, // 10 MB
        'description'      => 'Lessee photos before delivery (custom)',
    ],
    // ... remaining fields
],
```

Via environment variables in `.env`:

```ini
NANO_ORDERS_PHOTO_USER_BEFORE_MAX_FILES=15
NANO_ORDERS_PHOTO_USER_BEFORE_ALLOWED_TYPES="jpg,jpeg,png,webp"
NANO_ORDERS_PHOTO_USER_BEFORE_MAX_SIZE=10240
```

---

## 🔁 Integration with `updateOrderStatusAdvanced`

The `updateOrderStatusAdvanced` function (in the `StepStatus` trait) now supports two new options:

| Option | Type | Description |
|--------|------|-------------|
| `skip_photo_check` | `bool` | Skip the required photo check (default `false`). |
| `photo_validation_rules` | `array` | Custom rules to override the default rules. |

**How it works:**
1. The function determines the actual new status (priority: `order_states_ref_type`, then `user_status`/`delivery_status`).
2. Calls `enforcePhotoRequirementsOnStatusChange` with the new status and the acting user.
3. If required photos are missing, throws an exception and prevents the change.

---

## ❗ Error Handling (Common Errors)

| Error | Cause | Solution |
|-------|-------|----------|
| `Invalid field name.` | Passed a field not among the four allowed fields. | Use one of: `user_before_delivery`, `user_after_delivery`, `delivery_before_delivery`, `delivery_after_delivery`. |
| `File is required.` | No file data was passed or the file is empty. | Ensure `file` is present in the request (as `UploadedFile`) or `file_base64`. |
| `You do not have permission to upload photos for this order.` | The user is not associated with the order and is not an admin. | Check that the user is the owner, the delivery person, or an admin. |
| `User of type user is not allowed to upload photos in field ...` | The user’s role is not in `allowed_roles` for this field. | Review the `photo_rules` settings for the field. |
| `Cannot upload photos in field ... in current status (COMPLETE).` | The order status does not allow upload for this field. | First change the order status, or adjust `allowed_statuses` in the settings. |
| `Maximum number of photos (10) reached in field ...` | The maximum number of photos has already been uploaded. | Delete some photos first, or increase `max_files`. |
| `You must upload photos in field ... before changing the status to DELIVERY.` | Attempting to change the status to `DELIVERY` or `COMPLETE` without the required photos. | Upload the required photos first, or use `skip_photo_check = true`. |

---

## ✅ Best Practices

1. **Use `getOrderPhotoUrls` in the API** to display photos with thumbnails, rather than returning only the original path.
2. **Do not trust user input** – the function automatically checks field validity (`allowedFields`) and permission rules.
3. **Only administrators should use `skip_photo_check = true`** – to avoid accidentally bypassing rules.
4. **Adjust `max_files` and `max_size` in the configuration** to match your storage plan and server.
5. **Use the events (`nano.orders.photo.uploaded`, `nano.orders.photo.deleted`)** to send WebSocket notifications or log activities.
6. **Test all roles** (owner, delivery, admin) to ensure that upload permissions are correctly configured.

---

## 🔚 Conclusion

The `StepPhotos` trait is a powerful and flexible addition to the `Nano.Orders` system, enabling management of order‑associated photos with full control over permissions based on order status and user role. Thanks to its integration with `Nano.FileUpload`, uploading is secure and unified, supporting both regular files and base64. The quick‑check functions (`canUploadPhoto`, `hasRequiredPhotosForStatus`) allow developers to build smart user interfaces, while the `skip_photo_check` option in `updateOrderStatusAdvanced` provides additional flexibility for administrators when needed. The detailed documentation and practical examples make it easy to integrate this trait into any application.

---

**Reference documentation:**
- [`OrderManager` and its attributes documentation](./docs/Orders/Classes/Docs-OrderManager-Class-en.md)
- [`StepStatus` documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-en.md)
- [Advanced `StepStatus` documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advanced-en.md)
- [`Nano.FileUpload` documentation](./docs/FileUpload/Docs-FileUpload-en.md)
