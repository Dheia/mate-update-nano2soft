# `FileUploadUserManager` Class Documentation

**Overview of the `Nano.FileUpload` Package and User & Permission Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload` package**, an integrated software package provided by **NanoSoft**, specifically designed for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible manner.

All classes in this module reside under the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models (modules) that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, as well as unified error handling. Thanks to this design, developers can easily build scalable file upload systems that meet the needs of advanced applications without compromising application security or performance.

---

### Introduction

The `FileUploadUserManager` class is the **User and Permission Manager** in the file upload system for NanoSoft applications. This class retrieves the current user from multiple sources (backend, frontend, any custom model), and provides a unified interface for checking user permissions based on their type (`backend`/`frontend`) and the required permissions.

Simply put, `FileUploadUserManager` is **the layer responsible for knowing who the current user is and whether they have the necessary permissions to perform a given operation**, with the ability to customize the user resolver function and permission checker function according to application requirements.

**Version Note:** Starting from version 1.1.0, the `edit` operation (file replacement) is supported and uses the same permission checking mechanism with `$operation = 'edit'`.

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$userResolver` | `callable|null` | Custom function to resolve the current user (overrides default resolver). |
| `$user` | `mixed|null` | Current user after resolution (cached for performance). |
| `$permissionChecker` | `callable|null` | Custom function for permission checking (overrides default logic). |

---

### Important Methods

#### 1. Managing the User Resolver

##### `public function setUserResolver(callable $resolver): self`
- **Purpose**: Set a custom function to resolve the current user.
- **Parameters**:
  - `$resolver`: A function that is called and returns a user object or `null`.
- **Returns**: Current instance for method chaining.
- **Example**:
  ```php
  $manager->setUserResolver(function() {
      return \Auth::user(); // or any logic to get the user
  });
  ```

##### `public function setUser($user): self`
- **Purpose**: Set the user directly (overrides the resolver).
- **Parameters**:
  - `$user`: User object or `null`.
- **Returns**: Current instance for method chaining.

##### `public function getUser()`
- **Purpose**: Get the current user (with caching).
- **Returns**: User object or `null`.
- **Behavior**:
  - If `$user` is cached, returns it.
  - If a custom resolver is set, calls it and caches the result.
  - Otherwise, calls the default resolver.

##### `public function clearUser(): self`
- **Purpose**: Reset the cached user (to force re-resolution next time).
- **Returns**: Current instance for method chaining.

#### 2. Managing the Permission Checker Function

##### `public function setPermissionChecker(callable $checker): self`
- **Purpose**: Set a custom function for permission checking.
- **Parameters**:
  - `$checker`: A function that receives `($user, $operation, $permissions)` and returns `bool`.
- **Returns**: Current instance for method chaining.
- **Example**:
  ```php
  $manager->setPermissionChecker(function($user, $operation, $permissions) {
      return $user->hasCustomPermission($operation, $permissions);
  });
  ```

#### 3. Getting User Information

##### `public function getUserType(): ?string`
- **Purpose**: Determine the current user's type.
- **Returns**: `'backend'`, `'frontend'`, `'guest'`, or `null` if unknown.
- **Behavior**:
  - Checks the object type: if it is `Backend\Models\User` returns `'backend'`.
  - If it is `RainLab\User\Models\User` returns `'frontend'`.
  - If the user does not exist returns `'guest'`.
  - Any other model is considered `'frontend'` by default.

##### `public function getId()`
- **Purpose**: Get the current user's ID.
- **Returns**: User ID (`int`, `string`, `uuid`) or `null`.
- **Behavior**:
  - Uses `getKey()` if available.
  - Otherwise looks for `id` or `uuid`.

#### 4. Checking Permissions

##### `public function checkPermission($operation, $permissions = null, $user = null): bool`
- **Purpose**: Check whether the user has the required permission.
- **Parameters**:
  - `$operation`: The operation (`add`, `edit`, `delete`, `view`) – may not be used directly in all cases, but is passed to the custom function if available.
  - `$permissions`: A single permission (string) or array of permissions (array) – primarily used for backend users.
  - `$user`: (Optional) User object; if not provided, uses the current user.
- **Returns**: `true` if allowed, otherwise `false`.
- **Behavior**:
  - Gets the user type from the object.
  - If the user is of type **backend** and permissions are provided, uses `checkBackendPermissions()` to check using `hasAccess` or `hasPermission`.
  - If the user is of type **frontend** or no permissions are provided:
    - If a custom `permissionChecker` is set, uses it.
    - Else if the user has a `canUpload` method, calls it.
    - Else returns `true` (default allowance for frontend users).

##### `protected function checkBackendPermissions($user, $permissions): bool`
- **Purpose**: Check permissions for a backend user using `hasAccess` or `hasPermission` methods.
- **Parameters**:
  - `$user`: User object (backend type).
  - `$permissions`: A single permission or an array.
- **Returns**: `true` if the user has any of the required permissions, otherwise `false`.

#### 5. Default Resolver

##### `protected function defaultUserResolver()`
- **Purpose**: The default resolver for the current user.
- **Behavior**:
  - First tries to use `Nano\AuthApi\Classes\AuthHelpers::getCurrentUser()` if available (from the `Nano.Api` add-on).
  - Then tries `BackendAuth::getUser()`.
  - Then `Auth::getUser()`.
  - Then `auth()->user()`.
- **Returns**: User object or `null`.

##### `public static function defaultResolver(): callable`
- **Purpose**: Create a default resolver function (useful for tests).
- **Returns**: A callable that returns the result of `defaultUserResolver()`.

---

### Events

This class does not directly dispatch events, but it is used within the context of `FileUploadRegistry` and `FileUploadService`, which may dispatch their own events (e.g., `beforeUpload`, `afterEditDelete`).

---

### Comprehensive Practical Examples

#### Example 1: Using the Default Resolver (Fetching a Backend User)

```php
$manager = FileUploadUserManager::instance();
$user = $manager->getUser();
if ($user) {
    echo "User type: " . $manager->getUserType(); // 'backend'
    echo "ID: " . $manager->getId();
    echo "Name: " . $user->full_name;
}
```

#### Example 2: Setting a Custom Resolver for Frontend Users

```php
$manager->setUserResolver(function() {
    return Auth::user(); // RainLab.User user
});

$user = $manager->getUser();
echo "User type: " . $manager->getUserType(); // 'frontend'
```

#### Example 3: Checking a Backend User's Permission (Add Operation)

```php
$manager = FileUploadUserManager::instance();
$user = $manager->getUser();

if ($manager->checkPermission('add', 'nano.shop.upload_image', $user)) {
    echo "Allowed to upload image";
} else {
    echo "Not allowed";
}
```

#### Example 4: Checking `edit` Permission for File Replacement (New in version 1.1.0)

```php
$manager = FileUploadUserManager::instance();
$user = $manager->getUser();
$hasEditPermission = $manager->checkPermission('edit', 'nano.shop.edit_image', $user);
if ($hasEditPermission) {
    echo "Allowed to replace the image";
}
```

#### Example 5: Setting a Custom Permission Checker Function (e.g., for Frontend Users)

```php
$manager->setPermissionChecker(function($user, $operation, $permissions) {
    // Custom logic: e.g., check a custom permissions table
    return $user->hasCustomPermission($operation);
});

$allowed = $manager->checkPermission('add', null, $user); // No backend permissions passed
```

#### Example 6: Using `canUpload` in a User Model (Frontend)

If the user model contains a `canUpload` method, it can be used:

```php
class User extends Model {
    public function canUpload($operation, $permissions) {
        return $this->hasRole('editor') || $this->id == 1;
    }
}

// Elsewhere
$manager = FileUploadUserManager::instance();
if ($manager->checkPermission('add', null, $user)) {
    // canUpload will be called automatically
}
```

#### Example 7: Integration with `FileUploadRegistry` to Check a Specific Model's Permission (including `edit`)

```php
$registry = FileUploadRegistry::instance();
$userManager = FileUploadUserManager::instance();

$modelClass = Product::class;
$field = 'image';
$user = $userManager->getUser();
$userType = $userManager->getUserType();

if ($registry->can($modelClass, 'edit', $userType, $user, $field)) {
    // Allowed to replace the image
}
```

#### Example 8: Resetting the User After Session Change

```php
$manager->clearUser();
$newUser = $manager->getUser(); // Re-resolves the user
```

---

### Interaction with Other Classes

- **With `FileUploadRegistry`**: `FileUploadRegistry` uses the `FileUploadUserManager` object to check permissions in its `can()` function (for `add`, `edit`, `delete`, `view` operations).
- **With `FileUploadService`**: `FileUploadService` uses the `FileUploadUserManager` object to get the current user and their type (`getUser()`, `getUserType()`), and calls `checkPermission()` when needed.
- **With `Base64` (moved inside the add-on)**: No direct interaction, but `FileUploadService` uses `Base64` for uploading.

---

### Best Practices

1. **Use the default resolver unless you have specific requirements**: The default resolver provides comprehensive support for backend and frontend users.
2. **Customize the resolver only if you need different logic**: For example, if you have a custom user system not supported by default.
3. **Use `setPermissionChecker` to unify permission checking logic for all users**: This is useful if your permission system is completely custom.
4. **Do not rely on the default allowance for frontend users**: If you need restrictions, set a custom checker or use `canUpload` in the user model.
5. **Test user permissions thoroughly**: Ensure that unauthorized users cannot perform upload, delete, or replace operations.
6. **Use `clearUser()` after user data changes or logout**: To avoid holding stale data.
7. **For `edit` operations (file replacement)**, ensure the user has the appropriate permission (`edit`) and that `disable_edit` is not enabled.

---

### Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `getUser()` returns `null` even though the user is logged in | The default resolver did not find the user (possibly due to a different authentication system). | Set a custom resolver using `setUserResolver`. |
| `checkPermission` always returns `true` for frontend users | No custom checker is set, and default allowance is `true`. | Use `setPermissionChecker` to implement your own validation logic. |
| Backend permissions do not work | User is of type backend but the `hasAccess` or `hasPermission` method does not exist. | Ensure the user object contains one of these methods (they usually exist in NanoSoft systems). |
| `getUserType` returns `'frontend'` for a backend model | The model does not match any known types. | Add appropriate checking in `getUserTypeFromModel` if using a custom model. |
| User changes in session but `getUser()` does not return the new value | Caching holds the old value. | Use `clearUser()` after the user changes. |
| `checkPermission` returns `false` even though the user has `edit` permission | The `'edit'` operation is not defined in the permission system or `disable_edit` is globally enabled. | Ensure `edit` permission is set in `FileUploadRegistry` and `disable_edit = false`. |

---

### Integration with NanoSoft Permission System

In NanoSoft applications, `FileUploadUserManager` is typically used indirectly through `FileUploadService`. However, it can be customized to fit any permission system:

- For **backend** users: Relies on `hasAccess` or `hasPermission` existing in the `Backend\Models\User` model.
- For **frontend** users: Can customize the `canUpload` method in the user model, or set a custom `permissionChecker`.

**Note:** Starting from version 1.1.0, the `edit` operation (file replacement) is supported and uses the same permission checking mechanism with `$operation = 'edit'`.

---

### Summary

The `FileUploadUserManager` class is the backbone for managing user identity and permissions in the file upload system for NanoSoft applications. It provides a flexible interface to get the current user and check their permissions, with support for different user types (`backend`/`frontend`) and the ability to customize the user resolver and permission checker function. Thanks to this design, developers can easily integrate the system with any existing authentication and permission mechanism in their applications. It also fully supports the `edit` operation (versions 1.1.0+), enabling control over file replacement permissions.

To see how this class is used in a broader context, refer to [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md) and [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md).

---

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadUserManager` Class](./Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)

