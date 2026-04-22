## 2026-04-20 - 2026-04-21

**Nano3.Kyc Plugin Update – Version 1.2.1**

### Refactoring API Routes for Verify, Reject, and Restore to a More REST-Friendly Pattern

---

### Summary of Updates

In version **1.2.1**, the API routes for non-standard operations (verification `verify`, rejection `reject`, and restoration `restore`) have been improved to be clearer and better organized. Instead of the previous pattern `POST documents/{id}/verify`, the new pattern `POST documents/verify/{id?}` has been adopted, with the document identifier made optional in the path (also supported via the request body). This change achieves several objectives:

- **Better alignment with REST principles**: The custom action (`verify`) appears under the resource scope (`documents`) before the identifier, clarifying that the action belongs to the `documents` collection.
- **Flexibility in passing the identifier**: The `id` can be sent as part of the path or as a parameter in the request body (`id` or `document_id`), preserving backward compatibility with any existing client using the previous method.
- **Ease of extensibility**: Facilitates adding new actions such as `documents/bulk-verify` in the future within the same scope.

The controller methods `verify`, `reject`, and `restore` have been updated to support this new pattern while retaining all internal logic unchanged. Documentation files have also been updated to reflect the new routes.

---

### Objectives of This Release

- **Improve route organization** and make them clearer for developers using the API.
- **Increase flexibility** in passing the document identifier without being restricted to its position in the path.
- **Maintain backward compatibility** so that existing applications using the old routes (which remain supported) are not broken.
- **Facilitate future maintenance** of the controller and routes.

---

### New Features

#### 1. New API Routes

The old routes have been replaced:

| Old Route (still supported) | New Route (recommended) |
| :--- | :--- |
| `POST documents/{id}/verify` | `POST documents/verify/{id?}` |
| `POST documents/{id}/reject` | `POST documents/reject/{id?}` |
| `POST documents/{id}/restore` | `POST documents/restore/{id?}` |

The new routes place the action (`verify`, `reject`, `restore`) as part of the resource path before the optional identifier.

#### 2. Controller Method Updates

The method signatures in the `Documents` controller have been modified to accept `$id` as an optional parameter from the route, while retaining the logic to read from `Input::get()` as a fallback:

```php
public function verify($id = null)
{
    if (!$id) {
        $id = Input::get('id') ?? Input::get('document_id');
    }
    // ... rest of the logic
}
```

This change allows the method to be invoked via the new route `POST documents/verify/123`, the old route `POST documents/123/verify`, or even by sending `id` in the request body with `POST documents/verify`.

#### 3. `routes.php` File Update

The routing file has been modified to include the new routes while keeping the old routes commented out (for backward compatibility):

```php
// New routes (recommended)
Route::post('documents/verify/{id?}', 'Documents@verify');
Route::post('documents/reject/{id?}', 'Documents@reject');
Route::post('documents/restore/{id?}', 'Documents@restore');

// Old routes (for backward compatibility - deprecated)
// Route::post('documents/{id}/verify', 'Documents@verify');
// Route::post('documents/{id}/reject', 'Documents@reject');
// Route::post('documents/{id}/restore', 'Documents@restore');
```

The old routes can be removed in a future version after notifying users.

#### 4. `version.yaml` File Update

A new version `1.2.1` has been added to document these changes.

#### 5. Documentation Update

The API documentation has been updated to reflect the new routes, with a note that the old routes still function for compatibility purposes.

---

### Comparison Between Patterns

| Criteria | Old Pattern `documents/{id}/verify` | New Pattern `documents/verify/{id?}` |
| :--- | :--- | :--- |
| **Clarity** | Good (action on a resource) | Better (action under resource scope) |
| **REST Compliance** | Acceptable (RPC-style) | Better grouped |
| **Extensibility** | Difficult to add bulk actions | Easy to add `documents/bulk-verify` |
| **Backward Compatibility** | Supported (by keeping old route) | Supported (optional parameter + reading from Input) |
| **Code Impact** | No change | Slight change in method signatures |

---

### Usage Examples

#### 1. Using the New Route with `id` in the Path

```bash
curl -X POST "https://api.example.com/api/v1/kyc/documents/verify/123" \
  -H "Authorization: Bearer <token>"
```

#### 2. Using the New Route with `id` in the Body (Optional)

```bash
curl -X POST "https://api.example.com/api/v1/kyc/documents/verify" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"id": 123}'
```

#### 3. Using the Old Route (Still Works)

```bash
curl -X POST "https://api.example.com/api/v1/kyc/documents/123/verify" \
  -H "Authorization: Bearer <token>"
```

---

### Benefits and Added Value

- **Better route organization**: Makes the API easier to understand and navigate.
- **Greater flexibility for developers**: They can choose the most convenient way to pass the identifier.
- **Readiness for extension**: Paves the way for adding bulk operations like `bulk-verify`.
- **No breaking changes**: Existing applications continue to work without modification.
- **Clearer documentation**: The new routes more accurately express the relationship between the resource and the action.

---

### Upgrade Requirements

1. **Code Update**:
   - Replace `routes.php` with the version containing the new routes.
   - Replace the `verify`, `reject`, and `restore` methods in `Documents.php` with the updated versions (supporting `$id = null`).

2. **No New Migrations** – No database changes required.

3. **No Configuration Changes** – The `config.php` file remains unchanged.

4. **Update Internal API Documentation** (if any) to reflect the new routes.

5. **Compatibility Testing**:
   - Ensure that the old routes still work (if you have kept them).
   - Test the new routes with and without `id` in the path.

---

### Conclusion

Version **1.2.1** represents a significant improvement in the API organization of the `Nano3.Kyc` plugin. By restructuring the routes for non-standard operations, the API has become clearer and more extensible, while maintaining full backward compatibility with previous versions. This step lays the groundwork for adding more advanced features in the future, such as bulk operations.

---

### 📚 Reference Documentation

- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [DocumentType Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [Manager Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [Document Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)
