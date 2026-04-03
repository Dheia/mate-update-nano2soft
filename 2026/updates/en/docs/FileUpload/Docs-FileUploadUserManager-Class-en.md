## `FileUploadUserManager` Class Documentation

**Overview of the `Nano.FileUpload` Package and User/Permission Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload`** package, an integrated software package provided by **NanoSoft**, designed specifically for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible way.

All classes of this module reside under the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, and handling errors uniformly. With this design, developers can easily build scalable file upload systems that meet advanced application requirements without compromising security or performance.

---

### Introduction

The `FileUploadUserManager` class is the **user and permission manager** in the file upload system for NanoSoft applications. This class retrieves the current user from multiple sources (backend, frontend, or any custom model), and provides a unified interface for checking user permissions based on their type (backend/frontend) and the required permissions.

Simply put, `FileUploadUserManager` is the **layer responsible for knowing who the current user is and whether they have the necessary permissions to perform a given operation**, with the ability to customize the user resolver and permission checker according to application needs.

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$userResolver` | `callable|null` | Custom function to resolve the current user (overrides the default resolver). |
| `$user` | `mixed|null` | The current user after resolution (cached for performance). |
| `$permissionChecker` | `callable|null` | Custom function to check permissions (overrides the default logic). |

---

### Important Methods

#### 1. Managing the User Resolver

##### `public function setUserResolver(callable $resolver): self`
- **Purpose**: Set a custom function to resolve the current user.
- **Parameters**:
  - `$resolver`: A function that returns a user object or `null`.
- **Returns**: The current instance for method chaining.
- **Example**:
  ```php
  $manager->setUserResolver(function() {
      return \Auth::user(); // or any custom logic
  });
  ```

##### `public function setUser($user): self`
- **Purpose**: Set the user directly (bypasses the resolver).
- **Parameters**:
  - `$user`: User object or `null`.
- **Returns**: The current instance for method chaining.

##### `public function getUser()`
- **Purpose**: Get the current user (cached).
- **Returns**: User object or `null`.
- **Behavior**:
  - If `$user` is cached, returns it.
  - If a custom resolver is set, calls it and caches the result.
  - Otherwise, calls the default resolver.

##### `public function clearUser(): self`
- **Purpose**: Clear the cached user (forces re‑resolution next time).
- **Returns**: The current instance for method chaining.

#### 2. Managing the Permission Checker

##### `public function setPermissionChecker(callable $checker): self`
- **Purpose**: Set a custom function to check permissions.
- **Parameters**:
  - `$checker`: A function that takes `($user, $operation, $permissions)` and returns `bool`.
- **Returns**: The current instance for method chaining.
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
  - If user is `null`, returns `'guest'`.
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
  - `$operation`: The operation (`add`, `edit`, `delete`, `view`) – may not be used directly in all cases.
  - `$permissions`: A single permission string or array of permissions – mainly used for backend users.
  - `$user`: (optional) User object; if not provided, uses the current user.
- **Returns**: `true` if allowed, otherwise `false`.
- **Behavior**:
  - Gets the user type from the object.
  - If the user is **backend** and permissions are provided, uses `checkBackendPermissions()` to check via `hasAccess` or `hasPermission`.
  - If the user is **frontend** or no permissions are provided:
    - If a custom `permissionChecker` exists, uses it.
    - Else if the user has a `canUpload` method, calls it.
    - Else returns `true` (default allowance for frontend users).

##### `protected function checkBackendPermissions($user, $permissions): bool`
- **Purpose**: Check permissions for a backend user using `hasAccess` or `hasPermission` methods.
- **Parameters**:
  - `$user`: User object (backend type).
  - `$permissions`: Single permission or array.
- **Returns**: `true` if the user has any of the required permissions, otherwise `false`.

#### 5. Default Resolver

##### `protected function defaultUserResolver()`
- **Purpose**: The default resolver for the current user.
- **Behavior**:
  - Tries to use `Nano\AuthApi\Classes\AuthHelpers::getCurrentUser()` if available (from `Nano.Api`).
  - Then tries `BackendAuth::getUser()`.
  - Then tries `Auth::getUser()`.
  - Then tries `auth()->user()`.
- **Returns**: User object or `null`.

##### `public static function defaultResolver(): callable`
- **Purpose**: Create a default resolver callable (useful for testing).
- **Returns**: A callable that returns the result of `defaultUserResolver()`.

---

### Events

This class does not fire events directly, but it is used in the context of `FileUploadRegistry` and `FileUploadService`, which may fire their own events.

---

### Comprehensive Examples

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

#### Example 3: Checking a Backend User's Permission

```php
$manager = FileUploadUserManager::instance();
$user = $manager->getUser();

if ($manager->checkPermission('add', 'nano.shop.upload_image', $user)) {
    echo "Allowed to upload image";
} else {
    echo "Not allowed";
}
```

#### Example 4: Setting a Custom Permission Checker (e.g., for Frontend Users)

```php
$manager->setPermissionChecker(function($user, $operation, $permissions) {
    // Custom logic: e.g., check a custom permissions table
    return $user->hasCustomPermission($operation);
});

$allowed = $manager->checkPermission('add', null, $user); // no backend permissions passed
```

#### Example 5: Using `canUpload` in a Frontend User Model

If the user model has a `canUpload` method, it can be used:

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

#### Example 6: Integration with `FileUploadRegistry` to Check Permission for a Specific Model

```php
$registry = FileUploadRegistry::instance();
$userManager = FileUploadUserManager::instance();

$modelClass = Product::class;
$field = 'image';
$user = $userManager->getUser();
$userType = $userManager->getUserType();

if ($registry->can($modelClass, 'add', $userType, $user, $field)) {
    // Allowed to upload
}
```

#### Example 7: Clearing the User After Session Change

```php
$manager->clearUser();
$newUser = $manager->getUser(); // re‑resolves the user
```

---

### Interaction with Other Classes

- **With `FileUploadRegistry`**: `FileUploadRegistry` uses `FileUploadUserManager` to check permissions in its `can()` method.
- **With `FileUploadService`**: `FileUploadService` uses `FileUploadUserManager` to get the current user and type (`getUser()`, `getUserType()`), and calls `checkPermission()` when needed.
- **With `Base64` (from `Nano.Api`)**: No direct interaction, but `FileUploadService` uses `Base64` for uploads.

---

### Best Practices

1. **Use the default resolver unless you have special requirements**: It provides comprehensive support for common authentication systems in NanoSoft.
2. **Customize the resolver only if you need different logic**: e.g., if you have a custom user system not supported by default.
3. **Use `setPermissionChecker` to unify permission logic across the application**: useful if you have a central permissions system (like Roles & Permissions).
4. **Don't rely on the default allowance for frontend users**: If you need restrictions, set a custom permission checker or use `canUpload` in the user model.
5. **Clear the cached user (`clearUser()`) after logout or user changes**: to avoid stale data.
6. **Test permissions thoroughly**: ensure unauthorized users cannot perform upload or delete operations.
7. **Use `getUserType()` to differentiate the user experience**: e.g., show different interfaces for backend and frontend users.
8. **Do not pass backend permissions to frontend users**: in `checkPermission`, if the user is frontend, leave `$permissions` empty and use the custom checker.

---

### Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `getUser()` returns `null` even though user is logged in | Default resolver could not find the user. | Set a custom resolver using `setUserResolver`. |
| `checkPermission` always returns `true` for frontend users | No custom permission checker set, and default allowance is `true`. | Set a `permissionChecker` to implement your own logic. |
| Backend permissions don't work | User is backend but lacks `hasAccess` or `hasPermission` methods. | Ensure the user object has one of these methods (in NanoSoft they usually do). |
| `getUserType` returns `'frontend'` for a backend model | The model does not match any known type. | Add appropriate checks in `getUserTypeFromModel` if you use a custom model. |
| User changes in session but `getUser()` still returns old value | Cached value persists. | Use `clearUser()` after user changes. |

---

### Integration with NanoSoft Permission System

In NanoSoft applications, `FileUploadUserManager` is typically used indirectly through `FileUploadService`. However, it can be customized easily:

- **For backend users**: relies on `hasAccess` or `hasPermission` present in `Backend\Models\User`.
- **For frontend users**: you can customize the `canUpload` method in the user model, or set a custom `permissionChecker` that uses `Auth` or any other permissions system.

---

### Conclusion

The `FileUploadUserManager` class is a flexible and powerful tool for managing user identity and permissions in the file upload system of NanoSoft applications. Through the examples above, developers can:

- Easily retrieve the current user from any source.
- Check user permissions according to their type (backend/frontend).
- Customize user resolution and permission checking logic to fit any existing authentication or permission system.

We recommend leveraging these examples to build secure and extensible file upload systems.

For more details on how to integrate this class with `FileUploadRegistry` and `FileUploadService`, refer to [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md) and [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md).

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)

