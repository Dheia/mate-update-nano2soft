# Advanced and Practical Examples for the `FileUploadUserManager` Class

**Namespace:** `Nano\FileUpload\Classes`  
**Purpose:** To provide comprehensive application examples for using `FileUploadUserManager` in various scenarios, from basic user retrieval to customizing the user resolver and permission checker, and integration with `FileUploadRegistry` and `FileUploadService`, including expected inputs/outputs and best practices.

---

## Introduction

This document presents a set of practical examples covering various use cases of the `FileUploadUserManager` class, which acts as the **user and permission manager** in the file upload system for NanoSoft applications. This class is responsible for:

- Retrieving the current user from multiple sources (backend, frontend, or any custom user system).
- Determining the user type (`backend`, `frontend`, `guest`).
- Checking user permissions based on their type and the required permissions.

Through these examples, you will learn how to:

- Use the default user resolver to fetch the current user.
- Customize the user resolver (`setUserResolver`) to fit your authentication system.
- Set a custom permission checker (`setPermissionChecker`).
- Handle different user types (backend, frontend, guest).
- Integrate `FileUploadUserManager` with `FileUploadRegistry` and `FileUploadService`.
- Troubleshoot common issues and apply best practices.

> **Note:** The examples here use `FileUploadUserManager` directly. In real applications, it is often used through `FileUploadRegistry` and `FileUploadService`.

---

## Prerequisites

- The `Nano.FileUpload` plugin installed and configured.
- A working authentication system (backend or frontend) that can provide the current user.
- Basic understanding of the user types supported by the application (backend/frontend).

### Setting Up the Class in the Application

```php
use Nano\FileUpload\Classes\FileUploadUserManager;

$manager = FileUploadUserManager::instance();
```

Because the class follows the Singleton pattern, it is accessed via `instance()`.

---

## 1. Basic Usage (Default Resolver)

**Scenario:** Retrieve the current user using the default resolver, and display their type and ID.

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
    // Custom logic to fetch the user from your system
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
- You can also set the user directly with `setUser($user)` to bypass the resolver entirely.

---

## 3. Setting the User Directly (`setUser`)

**Scenario:** You already have the user (e.g., from an API request) and want to pass it to `FileUploadUserManager` without using the resolver.

```php
$user = CustomUser::find(10);
$manager->setUser($user);

echo "User ID: " . $manager->getId(); // 10
```

**Notes:**  
- `setUser` overrides the resolver and the custom resolver; it uses the specified user directly.
- Use `clearUser()` to reset the cached user and revert to the resolver.

---

## 4. Clearing the Cached User (`clearUser`)

**Scenario:** After a user logs out, force `FileUploadUserManager` to re‑resolve the user.

```php
// Assume a user was logged in
$user = $manager->getUser(); // returns the old user

// After logout
$manager->clearUser();

$user = $manager->getUser(); // now returns the new user (null or another user)
```

---

## 5. Checking Permissions for a Backend User (`checkPermission`)

**Scenario:** A backend user wants to upload an image; check if they have the permission `nano.shop.upload_image`.

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
- Otherwise returns `false`.

**Notes:**  
- The method internally calls `checkBackendPermissions` for backend users.
- You can pass an array of permissions instead of a single one: `['nano.shop.upload', 'nano.shop.manage']`.

---

## 6. Setting a Custom Permission Checker (`setPermissionChecker`)

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
- The custom checker is called for both backend and frontend users if set, overriding the default logic.
- `$permissions` may be `null` if no permissions are passed (as in the frontend case).

---

## 7. Using the `canUpload` Method in a Frontend User Model

**Scenario:** The user model has a `canUpload` method used for permission checking.

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
    // This will call $user->canUpload('add', null)
    echo "Allowed";
}
```

**Notes:**  
- `canUpload` is called only if the user is frontend and no custom `permissionChecker` is set.
- If the user has a `canUpload` method, its result is returned; otherwise, `true` (default allowance) is returned.

---

## 8. Integration with `FileUploadRegistry` to Check Permission for a Specific Field

**Scenario:** Before uploading a file for a specific field, use `FileUploadRegistry::can`, which internally uses `FileUploadUserManager`.

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

**Notes:**  
- `FileUploadRegistry::can` calls `FileUploadUserManager::checkPermission` with the field or model permissions.
- This is the common usage in real applications.

---

## 9. Getting the User Type (`getUserType`)

**Scenario:** Display different interfaces for backend and frontend users.

```php
$userType = $manager->getUserType();

switch ($userType) {
    case 'backend':
        echo "Welcome, Admin";
        break;
    case 'frontend':
        echo "Welcome, Visitor";
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

## 10. Handling Unauthenticated Users (Guest)

**Scenario:** Do not allow unauthenticated users to upload files.

```php
$user = $manager->getUser();
if (!$user) {
    die("Please log in to upload files");
}
```

---

## 11. Integration with `FileUploadService` in a Complete Upload Process

**Scenario:** Upload a file with permission checks performed internally by `FileUploadService`.

```php
$service = FileUploadService::instance();
$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);

if (!$result['status'] && $result['code'] == 403) {
    echo "Not allowed to upload file: " . $result['error'];
}
```

**Notes:**  
- `FileUploadService` calls `FileUploadRegistry::can`, which uses `FileUploadUserManager`.
- Any permission error will return with code 403.

---

## 12. Testing `checkPermission` with Different Types of Permissions

**Scenario:** Test a single permission and an array of permissions.

```php
$user = BackendAuth::getUser();

// Single permission
$allowed = $manager->checkPermission('add', 'nano.shop.upload', $user);

// Array of permissions (checks any)
$allowed = $manager->checkPermission('add', ['nano.shop.upload', 'nano.shop.manage'], $user);
```

**Behavior:**  
- For an array, returns `true` if the user has any of the permissions.

---

## 13. Customizing the User Resolver Based on a Token (JWT)

**Scenario:** An API uses JWT; you need to extract the user from the token.

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

## 14. Customizing the Permission Checker to Use Laravel Gates

**Scenario:** Use Laravel Gates for permission checks.

```php
$manager->setPermissionChecker(function($user, $operation, $permissions) {
    if (Gate::allows($operation, $user)) {
        return true;
    }
    return false;
});
```

---

## 15. Handling Permission Errors in an API Controller

**Scenario:** In an API controller, return appropriate responses if the user is not authorized.

```php
public function upload(Request $request)
{
    $user = FileUploadUserManager::instance()->getUser();
    if (!$user) {
        return response()->json(['error' => 'Unauthorized'], 401);
    }

    $userType = FileUploadUserManager::instance()->getUserType();
    $allowed = FileUploadRegistry::instance()->can(
        Product::class, 'add', $userType, $user, 'image'
    );

    if (!$allowed) {
        return response()->json(['error' => 'Permission denied'], 403);
    }

    // ... upload file
}
```

---

## 16. Best Practices

1. **Use the default resolver unless you have special requirements**: It provides comprehensive support for common authentication systems in NanoSoft.
2. **Don't rely on the default allowance for frontend users**: If you need restrictions, set a custom permission checker or use `canUpload` in the user model.
3. **Use `setPermissionChecker` to unify permission logic across the application**: Useful if you have a central permissions system (e.g., Roles & Permissions).
4. **Clear the cached user (`clearUser()`) after logout or user changes**: to avoid stale data.
5. **Test permissions thoroughly**: ensure unauthorized users cannot perform upload or delete operations.
6. **Use `getUserType()` to differentiate user experience**: e.g., show different interfaces for backend and frontend users.
7. **Do not pass backend permissions to frontend users**: in `checkPermission`, if the user is frontend, leave `$permissions` empty and use the custom checker.

---

## 17. Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `getUser()` returns `null` even though user is logged in | Default resolver could not find the user. | Set a custom resolver using `setUserResolver`. |
| `checkPermission` always returns `true` for frontend users | No custom permission checker set, and default allowance is `true`. | Set a `permissionChecker` to implement your own logic. |
| Backend permissions don't work | User is backend but lacks `hasAccess` or `hasPermission` methods. | Ensure the user object has one of these methods (in NanoSoft they usually do). |
| `getUserType` returns `'frontend'` for a backend model | The model does not match any known type. | Add appropriate checks in `getUserTypeFromModel` if you use a custom model. |
| User changes in session but `getUser()` still returns old value | Cached value persists. | Use `clearUser()` after user changes. |

---

## 18. Integration with NanoSoft Permission System

In NanoSoft applications, `FileUploadUserManager` is typically used indirectly through `FileUploadService`. However, it can be customized easily:

- **For backend users**: relies on `hasAccess` or `hasPermission` present in `Backend\Models\User`.
- **For frontend users**: you can customize the `canUpload` method in the user model, or set a custom `permissionChecker` that uses `Auth` or any other permissions system.

---

## Conclusion

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
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)

