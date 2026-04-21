## Documentation of `ReportQueryErrorHandler` Class

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The `ReportQueryErrorHandler` class is responsible for **managing and standardizing error handling** in the dynamic reporting system (`Nano2\QueryBuilder\Classes\Reporting`). When any error occurs during report execution (whether a validation error, database error, timeout, or export issue), this class transforms the exception or situation into a structured response containing useful information for the user (with easy-to-understand messages) while logging all technical details to facilitate debugging.

---

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$errorCodes` | `array` | An array mapping error names to unique numeric codes (e.g., `'VALIDATION_FAILED' => 1001`). Used to identify the error type uniformly. |
| `$userFriendlyMessages` | `array` | An array mapping error names to user-friendly messages (e.g., `'TABLE_NOT_FOUND' => 'The specified table is not available for reports.'`). |

---

### Main Methods

#### 1. `handleError(\Throwable $exception, array $context = []): array`
**Purpose**: Handle any general exception (from a `try-catch` block) and transform it into a structured error response.

**Parameters**:
- `$exception`: The caught exception object.
- `$context`: An optional array containing additional context information (e.g., report configuration, user, etc.) to be logged.

**Steps**:
1. Determine the error code using `determineErrorCode()`.
2. Generate a unique error ID using `generateErrorId()`.
3. Log the error with all details using `logError()`.
4. Build a response containing:
   - `success: false`
   - `error`:
     - `code`: Textual error code (e.g., `'TIMEOUT'`).
     - `id`: Unique error identifier.
     - `message`: User-friendly message.
     - `details`: Limited technical details (e.g., exception type, file, line).
     - `suggestions`: List of suggestions to resolve the issue.
     - `timestamp`: Error time in ISO format.
   - `meta`: Additional information like `request_id`, `user_id`, `company_id`.

**Example Response**:
```json
{
  "success": false,
  "error": {
    "code": "TABLE_NOT_FOUND",
    "id": "ERR_ABC12345_1712345678",
    "message": "The specified table is not available for reports.",
    "details": {
      "type": "InvalidArgumentException",
      "file": "ReportQueryConverter.php",
      "line": 123
    },
    "suggestions": [
      "Check the table name.",
      "Ensure you have access permissions."
    ],
    "timestamp": "2025-04-06T10:30:00Z"
  },
  "meta": {
    "request_id": "req-xyz-789",
    "user_id": 42,
    "company_id": "comp-123"
  }
}
```

#### 2. `handleValidationError(array $validationResult, array $context = []): array`
**Purpose**: Handle validation errors returned by `ReportQueryValidator`.

**Parameters**:
- `$validationResult`: The result from `ReportQueryValidator::validate()` containing `errors` and `warnings`.
- `$context`: Additional context for logging.

**Steps**:
- Generate an error ID.
- Log a warning with the errors and warnings.
- Return a response containing `validation_errors`, `validation_warnings`, and `suggestions` extracted via `getValidationSuggestions()`.

**Example Response**:
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_FAILED",
    "id": "ERR_DEF45678_1712345678",
    "message": "Query validation failed",
    "validation_errors": [
      "Column 'xyz' does not exist in table 'orders'"
    ],
    "validation_warnings": [
      "Many conditions may impact performance"
    ],
    "suggestions": [
      "Check the existence of the specified columns.",
      "Reduce the number of conditions to improve performance."
    ],
    "timestamp": "2025-04-06T10:30:00Z"
  }
}
```

#### 3. `handleTimeoutError(float $executionTime, array $context = []): array`
**Purpose**: Handle timeout errors specifically.

**Parameters**:
- `$executionTime`: Actual execution time (in milliseconds) before the timeout occurred.
- `$context`: Additional context.

**Response**:
- Contains `execution_time` and specific suggestions for handling the timeout.

**Example**:
```json
{
  "success": false,
  "error": {
    "code": "TIMEOUT",
    "id": "ERR_GHI78901_1712345678",
    "message": "Query execution timed out",
    "execution_time": 30500,
    "suggestions": [
      "Reduce the number of selected columns.",
      "Add more specific filters.",
      "Increase the allowed limit."
    ],
    "timestamp": "2025-04-06T10:30:00Z"
  }
}
```

#### 4. `handleExportError(\Throwable $exception, string $format, array $context = []): array`
**Purpose**: Handle errors that occur during result export (using `ReportQueryExporter`).

**Parameters**:
- `$exception`: The caught exception.
- `$format`: The requested export format (csv, excel, json, pdf, xml).
- `$context`: Additional context.

**Response**:
- Contains `format` and export-specific suggestions.

**Example**:
```json
{
  "success": false,
  "error": {
    "code": "EXPORT_FAILED",
    "id": "ERR_JKL01234_1712345678",
    "message": "Export to pdf format failed",
    "format": "pdf",
    "suggestions": [
      "Try another export format.",
      "Reduce the amount of exported data.",
      "Check storage space."
    ],
    "timestamp": "2025-04-06T10:30:00Z"
  }
}
```

#### 5. `formatErrorForOutput(array $error, string $outputType = 'json'): mixed`
**Purpose**: Format the error for different outputs (JSON, HTML, plain text). Currently supports JSON by default, with helper functions `formatErrorAsHtml` and `formatErrorAsText` (can be extended later).

**Parameters**:
- `$error`: The error array (as returned by the previous methods).
- `$outputType`: The output type (`'json'`, `'html'`, `'text'`).

**Usage**: Useful if the system supports multiple response types (e.g., an API that also handles Ajax requests that might expect HTML to display the error directly).

---

### Protected Helper Methods

#### `determineErrorCode(\Throwable $exception): string`
- Analyzes the exception message to determine the most appropriate error code (e.g., `TABLE_NOT_FOUND`, `COLUMN_NOT_FOUND`, `PERMISSION_DENIED`, `TIMEOUT`, `MEMORY_LIMIT`, `CONNECTION_ERROR`, `VALIDATION_FAILED`, or default `QUERY_EXECUTION_FAILED`).

#### `getUserFriendlyMessage(string $errorCode): string`
- Returns the appropriate user message from `$userFriendlyMessages`, or a default message if not found.

#### `getErrorDetails(\Throwable $exception, string $errorCode): array`
- Returns limited technical details (exception type, file name, line number). Adds additional details based on the error code (e.g., `timeout_limit`, `memory_limit`, `database`).

#### `getErrorSuggestions(string $errorCode): array`
- Returns a list of suggestions for resolving the issue based on the error code.

#### `getValidationSuggestions(array $errors): array`
- Analyzes validation errors and generates specific suggestions based on the content of each error (e.g., presence of words like "table", "column", "join", "condition").

#### `logError(\Throwable $exception, string $errorCode, string $errorId, array $context): void`
- Logs the error to the error log (`Log::error`) with all details (exception, context, user, company, request, IP, User-Agent).

#### `generateErrorId(): string`
- Generates a unique error ID in the format `ERR_` + 8 random characters + `_` + timestamp. Example: `ERR_AB12CD34_1712345678`.

#### `isRecoverable(string $errorCode): bool`
- Determines whether the error is recoverable (i.e., can be retried later). Recoverable errors: `TIMEOUT`, `MEMORY_LIMIT`, `RATE_LIMIT_EXCEEDED`, `CONNECTION_ERROR`.

#### `getRetrySuggestions(string $errorCode): array`
- Returns retry suggestions (wait time, message) for recoverable errors.

---

### Practical Examples of Using the Class

#### Scenario 1: Using `handleError` in a `try-catch` block
```php
use Nano2\QueryBuilder\Classes\Reporting\ReportQueryErrorHandler;

class ReportController extends Controller
{
    public function generateReport(Request $request, ReportQueryErrorHandler $errorHandler)
    {
        try {
            // Execute the report...
            $converter = new ReportQueryConverter($registry, $queryConfig);
            $result = $converter->execute();
            return response()->json($result);
        } catch (\Throwable $e) {
            // Gather context for debugging
            $context = [
                'query_config' => $queryConfig,
                'user_id' => auth()->id(),
                'company_id' => session('company'),
            ];
            
            $errorResponse = $errorHandler->handleError($e, $context);
            return response()->json($errorResponse, 500);
        }
    }
}
```

**Error Scenario (Table Not Found):**
- Exception: `new \InvalidArgumentException("Table 'non_existent' not found")`.
- Resulting response (as in the previous example) with code `TABLE_NOT_FOUND` and appropriate message.

#### Scenario 2: Using `handleValidationError` after validation
```php
$validator = new ReportQueryValidator($registry);
$validation = $validator->validate($queryConfig);

if (!$validation['valid']) {
    $errorResponse = $errorHandler->handleValidationError($validation, ['query_config' => $queryConfig]);
    return response()->json($errorResponse, 422);
}
```

#### Scenario 3: Handling a timeout error
```php
$startTime = microtime(true);
// ... execute a long query ...
$executionTime = microtime(true) - $startTime;

if ($executionTime > 30) { // assume max is 30 seconds
    $errorResponse = $errorHandler->handleTimeoutError($executionTime * 1000, ['query_config' => $queryConfig]);
    return response()->json($errorResponse, 500);
}
```

#### Scenario 4: Handling an export error
```php
try {
    $exporter = new ReportQueryExporter($data, $columns, $metadata, $tableName);
    $exportResult = $exporter->export($format);
    return response()->json($exportResult);
} catch (\Throwable $e) {
    $errorResponse = $errorHandler->handleExportError($e, $format, ['table' => $tableName]);
    return response()->json($errorResponse, 500);
}
```

#### Scenario 5: Getting retry suggestions
```php
if ($errorHandler->isRecoverable($errorCode)) {
    $retryInfo = $errorHandler->getRetrySuggestions($errorCode);
    // $retryInfo = ['wait_time' => 30, 'message' => 'Try again in 30 seconds...']
    // Can be added to the response or shown to the user.
}
```

---

### Error Codes List

| Code | Meaning | User Message |
|------|---------|--------------|
| `VALIDATION_FAILED` | Configuration validation failed | The report configuration contains errors. Please review. |
| `TABLE_NOT_FOUND` | Table not found | The specified table is not available for reports. |
| `COLUMN_NOT_FOUND` | Column not found | One or more columns are not available. |
| `PERMISSION_DENIED` | Insufficient permissions | You do not have permission to access this data. |
| `QUERY_EXECUTION_FAILED` | Query execution failed | Unable to execute the query. Please simplify it. |
| `EXPORT_FAILED` | Export failed | Export could not be completed. Please try again. |
| `TIMEOUT` | Timeout | The query took too long. Try narrowing the data scope. |
| `MEMORY_LIMIT` | Memory limit exceeded | The query consumes too much memory. Try limiting the results. |
| `INVALID_CONFIGURATION` | Invalid configuration | The report configuration is invalid. |
| `SCHEMA_ERROR` | Schema error | An error occurred in the data schema. |
| `CONNECTION_ERROR` | Connection error | Unable to connect to the database. |
| `RATE_LIMIT_EXCEEDED` | Rate limit exceeded | Too many requests. Please wait a moment. |

---

### Important Notes
- The class focuses on **standardizing responses** and providing safe user messages while hiding sensitive technical details (like full file paths, long stack traces) but logs them for debugging.
- The `logError` function logs a large amount of information (including trace), which aids in debugging.
- The class can be extended by adding new output formats (e.g., XML) by modifying `formatErrorForOutput` and adding helper methods.
- `getValidationSuggestions` analyzes textual errors to provide customized suggestions, improving user experience.

---

### Summary
`ReportQueryErrorHandler` is an essential tool for building a robust and user-friendly reporting system. By standardizing error handling, it ensures users receive clear and helpful messages, while developers retain all necessary details for debugging. This contributes to improving system stability and the end-user experience.