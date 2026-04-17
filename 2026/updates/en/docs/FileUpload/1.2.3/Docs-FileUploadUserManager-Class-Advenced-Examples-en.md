# Advanced and Practical Examples for `FileUploadUserManager` Class Documentation

**Namespace:** `Nano\FileUpload\Classes`  
**Purpose:** Provide comprehensive practical examples for using `FileUploadUserManager` in various scenarios, from basic retrieval of the current user to customizing the user resolver and permission checker function, and integration with `FileUploadRegistry` and `FileUploadService`, including expected inputs/outputs and best practices.

> **Version Note:** Starting from version 1.1.0, the `edit` operation (file replacement) is supported and uses the same permission checking mechanism with `$operation = 'edit'`. Examples have been updated to include this operation.

---

## Introduction

This document provides a set of practical examples covering various use cases of the `FileUploadUserManager` class, which represents the **User and Permission Manager** in the file upload system for NanoSoft applications. This class is responsible for:

- Retrieving the current user from multiple sources (backend, frontend, or any custom user system).
- Determining the user type (`backend`, `frontend`, `guest`).
- Checking user permissions based on their type and the required permissions (including `edit` permission for file replacement).

Through these examples, you will learn how to:

- Use the default user resolver to fetch the current user.
- Customize the user resolver (`setUserResolver`) to fit your authentication system.
- Set a custom permission checker function (`setPermissionChecker`).
- Handle different user types (backend, frontend, guest).
- Check `edit` permission for file replacement.
- Integrate `FileUploadUserManager` with `FileUploadRegistry` and `FileUploadService`.
- Troubleshoot common issues and their solutions.

> **Note:** The examples here use `FileUploadUserManager` directly, but in real applications it is often used via `FileUploadRegistry` and `FileUploadService`.

---

## Prerequisites

- `Nano.FileUpload` add-on installed and configured in your application (version 1.1.0 or later to benefit from `edit` permission).
- A working authentication system (backend or frontend) from which the current user can be obtained.
- Basic understanding of the user types supported by the application (backend/frontend).

### Setting up the class in the application

```php
use Nano\FileUpload\Classes\FileUploadUserManager;

$manager = FileUploadUserManager::instance();
```

Since the class follows the Singleton pattern, it is called via `instance()`.

---

## 1. Basic Usage (Default Resolver)

**Scenario:** Fetch the current user using the default resolver, and display their type and ID.

```php
$manager = FileUploadUserManager::instance();
$user = $manager->getUser();

if ($user) {
    echo "User type: " . $manager->getUserType(); // 'backend' or 'frontend'
    echo "User ID: " . $manager->getId();
    echo "Name: " . $user->name ?? $user->full_name ?? 'Unknown';
} else {
    echo "User not logged in (guest)";
}
```

**Expected Output (example):**

```
User type: backend
User ID: 5
Name: Ahmed Ali
```

**Notes:**  
- The default resolver attempts to use `Nano\AuthApi\Classes\AuthHelpers::getCurrentUser()` first, then `BackendAuth::getUser()`, then `Auth::getUser()`, then `auth()->user()`.  
- If no user is found, returns `null` and the user type is `'guest'`.

---

## 2. Customizing the User Resolver (`setUserResolver`)

**Scenario:** You have a custom user system based on a `custom_users` table, and you want `FileUploadUserManager` to use it.

```php
$manager->setUserResolver(function() {
    // Custom logic to fetch the user from your custom system
    if (isset($_SESSION['custom_user_id'])) {
        return CustomUser::find($_SESSION['custom_user_id']);
    }
    return null;
});

$user = $manager->getUser();
if ($user) {
    echo "Custom user retrieved successfully";
}
```

**Notes:**  
- After setting a custom resolver, the default resolver is ignored.  
- You can also set the user directly using `setUser($user)` to completely override the resolver.

---

## 3. Setting the User Directly (`setUser`)

**Scenario:** You already have the user (e.g., from an API request) and want to pass it to `FileUploadUserManager` without using the resolver.

```php
$user = CustomUser::find(10);
$manager->setUser($user);

echo "User ID: " . $manager->getId(); // 10
```

**Notes:**  
- `setUser` overrides the resolver and the custom resolver, using the specified user directly.  
- Use `clearUser()` to reset the cached user and fall back to the resolver.

---

## 4. Resetting the Cached User (`clearUser`)

**Scenario:** After the user logs out, you want to force `FileUploadUserManager` to re-resolve the user.

```php
// Assume the user was logged in
$user = $manager->getUser(); // returns the old user

// After logout
$manager->clearUser();

$user = $manager->getUser(); // returns the new user (null or another user)
```

---

## 5. Checking Permissions for a Backend User (Add Operation)

**Scenario:** A backend user wants to upload an image. Check if they have the `nano.shop.upload_image` permission.

```php
$user = BackendAuth::getUser(); // backend user
$hasPermission = $manager->checkPermission('add', 'nano.shop.upload_image', $user);

if ($hasPermission) {
    echo "Allowed to upload image";
} else {
    echo "Not allowed";
}
```

**Output:**  
- If the user has the permission (via `hasAccess` or `hasPermission`), returns `true`.  
- Otherwise, returns `false`.

**Notes:**  
- The function internally calls `checkBackendPermissions` for backend users.  
- You can pass an array of permissions instead of a single one: `['nano.shop.upload', 'nano.shop.manage']`.

---

## 6. Checking `edit` Permission for File Replacement (New in version 1.1.0)

**Scenario:** A backend user wants to replace an existing product image (edit operation). Check if they have the `nano.shop.edit_image` permission.

```php
$user = BackendAuth::getUser();
$hasEditPermission = $manager->checkPermission('edit', 'nano.shop.edit_image', $user);

if ($hasEditPermission) {
    echo "Allowed to replace the image";
} else {
    echo "Not allowed to replace the image";
}
```

**Notes:**  
- The `edit` permission is used in `FileUploadService` when it detects a file replacement operation in an `attachOne` relation.  
- The permission must be set in `FileUploadRegistry` at the model or field level.

---

## 7. Setting a Custom Permission Checker Function (`setPermissionChecker`)

**Scenario:** You have a custom permission system for frontend users based on a `user_roles` table.

```php
$manager->setPermissionChecker(function($user, $operation, $permissions) {
    // Custom logic: e.g., check the user's role
    if ($user->hasRole('editor')) {
        return true;
    }
    return false;
});

$user = Auth::getUser(); // frontend user
$allowed = $manager->checkPermission('add', null, $user); // no backend permissions passed
if ($allowed) {
    echo "Allowed";
}
```

**Notes:**  
- The custom checker function is called for both backend and frontend if set, and overrides the default logic.  
- `$permissions` may be `null` if no permissions are passed (as in the frontend case).

---

## 8. Using the `canUpload` Method in a User Model (Frontend)

**Scenario:** The user model contains a `canUpload` method that is used to check permissions.

```php
class User extends Model {
    public function canUpload($operation, $permissions) {
        // Custom logic: e.g., check a permissions table
        return $this->hasPermission('upload') || $operation === 'view';
    }
}

// Elsewhere
$manager = FileUploadUserManager::instance();
$user = Auth::getUser();

if ($manager->checkPermission('add', null, $user)) {
    // $user->canUpload('add', null) will be called
    echo "Allowed";
}
```

**Notes:**  
- `canUpload` is called only if the user is of type `frontend` and no custom `permissionChecker` is set.  
- If the user has a `canUpload` method, its result is returned; otherwise, it returns `true` (default allowance).

---

## 9. Integration with `FileUploadRegistry` to Check a Specific Field's Permission (including `edit`)

**Scenario:** Before replacing a file for a specific field, use `FileUploadRegistry::can`, which in turn relies on `FileUploadUserManager`.

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

**Notes:**  
- `FileUploadRegistry::can` calls `FileUploadUserManager::checkPermission` with the field or model permissions.  
- This is the common usage pattern in real applications.

---

## 10. Getting the User Type (`getUserType`)

**Scenario:** Display a different interface for backend and frontend users.

```php
$userType = $manager->getUserType();

switch ($userType) {
    case 'backend':
        echo "Welcome, Admin";
        break;
    case 'frontend':
        echo "Welcome, Dear Visitor";
        break;
    default:
        echo "Please log in";
}
```

**Output:**  
- `'backend'` for system users.  
- `'frontend'` for frontend users.  
- `'guest'` for unauthenticated users.

---

## 11. Handling Unauthenticated Users (Guest)

**Scenario:** Do not allow unauthenticated users to upload files.

```php
$user = $manager->getUser();
if (!$user) {
    die("Please log in to upload files");
}
```

---

## 12. Integration with `FileUploadService` in a Complete Upload Process (including `edit`)

**Scenario:** Upload a file with permission checking using `FileUploadService`, which internally uses `FileUploadUserManager`.

```php
$service = FileUploadService::instance();
$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);

if (!$result['status'] && $result['code'] == 403) {
    echo "Not allowed to upload file: " . $result['error'];
}
```

**Notes:**  
- `FileUploadService` calls `FileUploadRegistry::can`, which in turn uses `FileUploadUserManager`.  
- Any permission error will appear with HTTP code 403.

---

## 13. Testing `checkPermission` with Different Types of Permissions

**Scenario:** Test a single permission and an array of permissions.

```php
$user = BackendAuth::getUser();

// Single permission
$allowed = $manager->checkPermission('add', 'nano.shop.upload', $user);

// Array of permissions (checks any)
$allowed = $manager->checkPermission('add', ['nano.shop.upload', 'nano.shop.manage'], $user);
```

**Behavior:**  
- In the case of an array, returns `true` if the user has any of the permissions.

---

## 14. Customizing the User Resolver Based on a Token (JWT)

**Scenario:** An API uses JWT, and you need to extract the user from the token.

```php
$manager->setUserResolver(function() {
    $token = request()->bearerToken();
    if ($token) {
        return User::where('api_token', $token)->first();
    }
    return null;
});

$user = $manager->getUser();
```

---

## 15. Customizing the Permission Checker to Be Strict (e.g., Laravel Gates)

**Scenario:** Use Laravel Gates to check permissions.

```php
$manager->setPermissionChecker(function($user, $operation, $permissions) {
    if (Gate::allows($operation, $user)) {
        return true;
    }
    return false;
});
```

---

## 16. Handling Permission Errors in an API Controller

**Scenario:** In an API controller, return an appropriate response if the user is not authorized (including for `edit` operation).

```php
public function replaceImage(Request $request, $id)
{
    $user = FileUploadUserManager::instance()->getUser();
    if (!$user) {
        return response()->json(['error' => 'Unauthorized'], 401);
    }

    $userType = FileUploadUserManager::instance()->getUserType();
    $allowed = FileUploadRegistry::instance()->can(
        Product::class, 'edit', $userType, $user, 'image'
    );

    if (!$allowed) {
        return response()->json(['error' => 'You do not have permission to replace the image'], 403);
    }

    // ... upload the new file and replace
}
```

---

## 17. Best Practices

1. **Use the default resolver unless you have specific requirements**: It provides comprehensive support for common authentication systems in NanoSoft.
2. **Do not rely on the default allowance for frontend users**: If you need restrictions, set a custom checker or use `canUpload` in the user model.
3. **Use `setPermissionChecker` to unify permission checking logic across the application**: Useful if you have a centralized permission system (e.g., Roles & Permissions).
4. **Reset the cached user (`clearUser()`) after logout or user changes**: To avoid holding stale data.
5. **Test user permissions thoroughly**: Ensure unauthorized users cannot perform upload, delete, or replace operations.
6. **Use `getUserType()` to differentiate user experience**: Such as showing different interfaces for backend and frontend users.
7. **Do not pass backend permissions to frontend users**: In `checkPermission`, if the user is frontend, leave `$permissions` empty and use the custom checker function.
8. **For `edit` operations (file replacement)**, ensure the user has the appropriate permission (`edit`) and that `disable_edit` is not enabled in global settings.

---

## 18. Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `getUser()` returns `null` even though user is logged in | Default resolver did not find the user. | Use `setUserResolver` to specify your own user retrieval method. |
| `checkPermission` always returns `true` for frontend users | No custom `permissionChecker` set, and default allowance is `true`. | Set `setPermissionChecker` to implement your own validation logic. |
| Backend permissions do not work | User is backend but `hasAccess` or `hasPermission` method is missing. | Ensure the user object contains one of these methods (they usually exist in NanoSoft systems). |
| `getUserType` returns `'frontend'` for a backend model | The model does not match any known types. | If using a custom model, extend `getUserTypeFromModel` or use `setUserResolver` to specify the type. |
| User changes in session but `getUser()` does not return the new value | Caching holds the old value. | Use `clearUser()` after the user changes. |
| `checkPermission` returns `false` even though the user has `edit` permission | The `'edit'` operation is not defined in the permission system or `disable_edit` is globally enabled. | Ensure `edit` permission is set in `FileUploadRegistry` and `disable_edit = false`. |

---

## 19. Integration with NanoSoft Permission System

In NanoSoft applications, `FileUploadUserManager` is typically used indirectly through `FileUploadService`. However, it can be easily customized:

- For **backend** users: Relies on `hasAccess` or `hasPermission` existing in the `Backend\Models\User` model.
- For **frontend** users: Can customize the `canUpload` method in the user model, or set a custom `permissionChecker` that uses `Auth` or any other permission system.

**Note:** Starting from version 1.1.0, the `edit` operation (file replacement) is supported and uses the same permission checking mechanism with `$operation = 'edit'`.

---

## Summary

The `FileUploadUserManager` class is a flexible and powerful tool for managing users and permissions in the file upload system for NanoSoft applications. Through the examples above, developers can:

- Easily retrieve the current user from any source.
- Check user permissions based on their type (backend/frontend), including `edit` permission.
- Customize user retrieval and permission checking logic to fit any existing authentication or permission system.

We recommend using these examples to build secure and scalable file upload systems.

For more details on integrating this class with `FileUploadRegistry` and `FileUploadService`, refer to [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md) and [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md).

---

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)


