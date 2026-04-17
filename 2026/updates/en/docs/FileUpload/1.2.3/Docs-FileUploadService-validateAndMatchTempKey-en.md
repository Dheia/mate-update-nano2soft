# `validateAndMatchTempKey` Function Documentation

## Overview

The `validateAndMatchTempKey` function is a helper method inside the `FileUploadService` class, aimed at unifying the process of validating temporary session keys (`temp_session_key`) and matching them against provided data (model, field, user). This function is designed to be flexible and extensible, with support for events, caching, error aggregation, and full control over exception behavior.

**Main Purpose:**  
- Avoid code duplication in multiple methods such as `deleteFile`, `getFiles`, `attachTempFiles`.
- Provide detailed information about why validation failed (for developers).
- Allow customization of validation logic via events without modifying core code.

---

## Signature

```php
protected function validateAndMatchTempKey(
    string|array $tempKey,
    string $modelClass,
    string $field,
    mixed $user = null,
    array $options = []
): array
```

---

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `$tempKey` | `string\|array` | Temporary session key (as string) or key data after validation (as array). If an array is passed, it is used directly without additional validation. |
| `$modelClass` | `string` | Full class name of the expected model (e.g., `Nano\Shop\Models\Product`). |
| `$field` | `string` | Expected field name (e.g., `'image'`). |
| `$user` | `mixed` | (Optional) Current user object. If not provided, it is fetched from `FileUploadUserManager`. |
| `$options` | `array` | Additional options array (see table below). |

### `$options` Details

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `skip_user_check` | `bool` | `false` | Skip user matching check (userId and userType). |
| `throw_on_failure` | `bool` | `true` | If `true`, throws an exception on failure. If `false`, returns result array with `status = false`. |
| `event_prefix` | `string` | `'nano.fileupload.tempKey'` | Prefix for event names (e.g., `nano.fileupload.tempKey.validating`). |
| `disable_events` | `bool` | `false` | Disable all events (for performance or testing). |
| `strict_mode` | `bool` | `true` | Strict mode: requires exact model match (`===`). If `false`, allows subclasses (`is_a`). |
| `cache_results` | `bool` | `false` | Cache validation results in memory (per request) to avoid repeated validation of the same key. |
| `retry_on_expiry` | `bool` | `false` | Attempt to renew the key if expired (requires `regenerate_callback`). |
| `regenerate_callback` | `callable\|null` | `null` | Function called to generate a new key when expired. Receives the expired key and returns a new key. |
| `stop_on_first_failure` | `bool` | `true` | If `true`, stops at first failure and throws exception. If `false`, collects all errors and throws a single exception at the end. |
| `allow_expired_key` | `bool` | `false` | Allow using expired keys within a grace period. |
| `expiry_grace_period` | `int` | `300` | Grace period in seconds (when `allow_expired_key = true`). |
| `custom_validator` | `callable\|null` | `null` | Additional validation function called after basic validation. Receives `$keyData, $modelClass, $field, $user, $options` and returns `bool`. |
| `key_data_overrides` | `array` | `[]` | Overrides for key data. Example: `['userId' => fn($v) => 'new_user_id']`. |
| `before_validation` | `callable\|null` | `null` | Function called before starting validation. Receives `$tempKey, $modelClass, $field, $options`. |
| `after_validation` | `callable\|null` | `null` | Function called after validation ends. Receives `$result`. |
| `events_payload` | `array` | `[]` | Additional data passed to all events. |
| `log_failures_only` | `bool` | `true` | If `true`, logs only failures. If `false`, logs all attempts. |
| `collect_metadata` | `bool` | `false` | Collect performance data (execution time, steps performed) and add to `meta`. |

---

## Return Value

The function returns an **array** following the same unified response structure as all `FileUploadService` methods. The array contains the following keys:

| Key | Type | Description |
|-----|------|-------------|
| `code` | `int` | HTTP status code (200 for success, 400/403/500 for failure). |
| `status` | `bool` | `true` if validation succeeded, `false` if failed. |
| `message` | `string` | Explanatory message (translatable). |
| `error` | `string\|null` | Error details (in case of failure). |
| `errors` | `array` | Array of additional errors (e.g., exception context). |
| `data` | `array` | Contains `key_data` (key data) and `matched_data` (matching results for each step). |
| `meta` | `array` | Metadata (e.g., execution time if requested). |
| `input_data` | `array` | Copy of input data (for tracing). |
| `process_data` | `array` | Internal information about the validation process. |
| `debug` | `array\|null` | Debug details (appears only when `app.debug = true`). |

### Structure of `data` on success:

```php
'data' => [
    'key_data' => [
        'modelClass' => '...',
        'field' => '...',
        'userId' => '...',
        'userType' => '...',
        'timestamp' => 1234567890,
    ],
    'matched_data' => [
        'model_matched' => true,
        'field_matched' => true,
        'user_matched' => true,
        'user_id_matched' => true,
        'user_type_matched' => true,
    ]
]
```

### Structure of `data` on failure (with `throw_on_failure = false`):

```php
'data' => [
    'key_data' => [ /* may be null or partial data */ ],
    'matched_data' => [
        'model_matched' => false,
        'field_matched' => true,  // values may vary depending on which step failed
        'user_matched' => false,
        // ...
    ]
]
```

---

## Events

The function fires the following events (can be listened to for extending behavior):

| Event Name | Fire Time | Parameters |
|------------|-----------|------------|
| `{prefix}.validating` | Before any validation starts | `$tempKey, $modelClass, $field, $options, &$result, &$process_data, $eventsPayload` |
| `{prefix}.beforeUserCheck` | Before user check (can modify `$user` or skip check) | `$keyData, $modelClass, $field, &$user, &$skipCheck, &$result, &$process_data, $eventsPayload` |
| `{prefix}.afterUserCheck` | After user check | `$keyData, $modelClass, $field, $userMatched, $userIdMatched, $userTypeMatched, &$result, &$process_data, $eventsPayload` |
| `{prefix}.validated` | When validation succeeds | `$keyData, $modelClass, $field, $result, $eventsPayload` |
| `{prefix}.failed` | When validation fails (custom exception) | `$exception, $modelClass, $field, $result, $eventsPayload` |
| `{prefix}.error` | When an unexpected error occurs | `$exception, $modelClass, $field, $result, $eventsPayload` |

**Note:** The prefix (`{prefix}`) can be changed via the `event_prefix` option.

---

## Practical Examples

### 1. Basic usage in `deleteFile` (with exception on failure)

```php
public function deleteFile($fileId, $modelClass = null, $field = null, $user = null): array
{
    // ... (prepare result)
    try {
        // ... (find file)

        // Validate temp key (if file is temporary)
        if ($file->session_key) {
            $validation = $this->validateAndMatchTempKey(
                $file->session_key,
                $modelClass ?? $file->attachment_type,
                $field ?? $file->field,
                $user,
                ['throw_on_failure' => true] // default
            );
            // On success, $validation['data']['key_data'] contains key data
        }
        // ...
    } catch (FileUploadException $e) {
        // Handle exception
    }
}
```

### 2. Using with `throw_on_failure = false` (check errors without stopping)

```php
public function attachTempFiles(Model $model, string $field, string $tempSessionKey, array $options = []): array
{
    // ...
    try {
        $validation = $this->validateAndMatchTempKey(
            $tempSessionKey,
            get_class($model),
            $field,
            null,
            ['throw_on_failure' => false, 'stop_on_first_failure' => false]
        );

        if (!$validation['status']) {
            // Validation failed, return result directly
            return $validation;
        }

        $keyData = $validation['data']['key_data'];
        // ... continue attachment process
    } catch (Exception $e) {
        // Handle unexpected exceptions
    }
}
```

### 3. Using caching to avoid repeated validation

```php
// In getFiles method, the same key might be called multiple times in one request
$validation = $this->validateAndMatchTempKey(
    $tempKey,
    $modelClass,
    $field,
    null,
    ['cache_results' => true]
);
```

### 4. Allow expired keys within grace period

```php
$validation = $this->validateAndMatchTempKey(
    $tempKey,
    $modelClass,
    $field,
    null,
    [
        'allow_expired_key' => true,
        'expiry_grace_period' => 600, // 10 minutes
    ]
);
```

### 5. Using `custom_validator` to check additional validity (e.g., IP address)

```php
$validation = $this->validateAndMatchTempKey($tempKey, $modelClass, $field, null, [
    'custom_validator' => function($keyData, $modelClass, $field, $user) {
        // Check that the key was issued from the same current IP address
        $allowedIp = $keyData['ip'] ?? null;
        return $allowedIp === request()->ip();
    }
]);
```

### 6. Overriding key data (`key_data_overrides`)

```php
// In case of user migration, we need to map old userId to new one for old keys
$validation = $this->validateAndMatchTempKey($tempKey, $modelClass, $field, null, [
    'key_data_overrides' => [
        'userId' => function($oldUserId) {
            return UserMigration::getNewId($oldUserId);
        }
    ]
]);
```

### 7. Listening to `beforeUserCheck` event to customize user logic

```php
// In `boot()` of Plugin.php
Event::listen('nano.fileupload.tempKey.beforeUserCheck', function ($keyData, $modelClass, $field, &$user, &$skipCheck, &$result, &$processData) {
    if ($modelClass === 'Nano\Shop\Models\Product' && $field === 'image') {
        $currentUser = $user ?? BackendAuth::getUser();
        if ($currentUser && $currentUser->hasAccess('nano.shop.manage_all_products')) {
            // Allow admin to access any product's files
            $skipCheck = true;
            $processData['user_check_overridden'] = true;
            return true;
        }
    }
    return false;
});
```

### 8. Aggregating errors to know all failure reasons (for front-ends)

```php
$validation = $this->validateAndMatchTempKey($tempKey, $modelClass, $field, null, [
    'throw_on_failure' => false,
    'stop_on_first_failure' => false,
]);

if (!$validation['status']) {
    $errors = $validation['errors']; // array containing all errors
    foreach ($errors as $error) {
        if ($error['step'] === 'model') {
            echo "Model mismatch: expected {$error['expected']}, got {$error['actual']}";
        } elseif ($error['step'] === 'user_id') {
            echo "User ID mismatch";
        }
    }
}
```

---

## Complete Example in `FileUploadService::attachTempFiles`

```php
public function attachTempFiles(Model $model, string $field, string $tempSessionKey, array $options = []): array
{
    $input_data = $options;
    $process_data = [];
    $result = [
        'code' => 200,
        'status' => true,
        'message' => trans('nano.fileupload::lang.public.helpers.upload.msg_attach_success'),
        'error' => null,
        'errors' => [],
        'data' => null,
        'meta' => null,
        'input_data' => $input_data,
        'process_data' => [],
        'debug' => null,
    ];

    try {
        // Validate key and match against model, field, and user
        $validation = $this->validateAndMatchTempKey(
            $tempSessionKey,
            get_class($model),
            $field,
            null,
            [
                'throw_on_failure' => true,
                'cache_results' => true,
                'event_prefix' => 'nano.fileupload.attachTempFiles',
            ]
        );

        $keyData = $validation['data']['key_data'];
        $process_data = array_merge($process_data, $validation['process_data']);

        // Find temporary files
        $files = File::where('field', $field)
                     ->where('session_key', $tempSessionKey)
                     ->get();

        if ($files->isEmpty()) {
            throw new FileUploadException(
                trans('nano.fileupload::lang.public.helpers.upload.no_temp_files_found'),
                FileUploadException::ERR_FILE_NOT_FOUND,
                ['temp_session_key' => $tempSessionKey, 'field' => $field]
            );
        }

        // Attach files
        $attachedFiles = [];
        foreach ($files as $file) {
            $file->attachment_id = $model->getKey();
            $file->attachment_type = $model->getMorphClass();
            $file->session_key = null;
            $file->expires_at = null;
            $file->save();

            if ($model->hasRelation($field)) {
                $model->{$field}()->add($file);
            }

            $attachedFiles[] = $file->toArray();
        }

        $result['data'] = $attachedFiles;
        $result['process_data'] = array_merge($process_data, [
            'attached_count' => count($attachedFiles),
            'temp_session_key' => $tempSessionKey,
        ]);
        $result['message'] = trans('nano.fileupload::lang.public.helpers.upload.msg_attach_success', ['count' => count($attachedFiles)]);

        Event::fire('nano.fileupload.afterAttach', [$model, $field, $attachedFiles, $tempSessionKey]);

    } catch (FileUploadException $e) {
        // Unified handling
        $result['code'] = $e->getCode() ?: 400;
        $result['status'] = false;
        $result['message'] = $e->getMessage();
        $result['error'] = $e->getMessage();
        $result['error_code'] = $e->getErrorCode();
        $result['errors'] = $e->getContext() ?: [];
        // ... add debug if needed
    } catch (Throwable $e) {
        // Handle general errors
        $result['code'] = 500;
        $result['status'] = false;
        $result['message'] = trans('nano.fileupload::lang.errors.general_error');
        $result['error'] = $e->getMessage();
        // ... add debug
    }

    return $result;
}
```

---

## Summary

The `validateAndMatchTempKey` function provides a unified and flexible way to validate temporary session keys. Thanks to its multiple options and extensible events, developers can:

- Avoid code duplication.
- Customize validation logic according to application needs (e.g., allow admins to access any user's files).
- Collect detailed information about failure reasons.
- Improve performance using caching.

This function is used inside `FileUploadService` in the `deleteFile`, `getFiles`, and `attachTempFiles` methods to ensure security and integrity of operations.

## Additional Documentation

- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advanced-Examples-en.md)

