## Documentation of `ReportQueryValidator` Class

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The `ReportQueryValidator` class is responsible for **comprehensive validation of the report configuration (Query Configuration)** before passing it to the `ReportQueryConverter` for execution. It performs a series of checks to ensure that the configuration follows the defined rules, all references (tables, columns, relationships) are correct and exist in the registry (`ReportSchemaRegistry`), and also issues warnings related to performance and security. This class aims to prevent early errors and ensure a smooth user experience.

---

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$registry` | `ReportSchemaRegistry` | Reference to the schema registry to access table, column, and relationship information. |
| `$errors` | `array` | Array to accumulate validation errors (prevent report execution). |
| `$warnings` | `array` | Array to accumulate performance and security warnings (do not prevent execution). |

---

### Main Methods

#### 1. `__construct(ReportSchemaRegistry $registry)`
- Injects the `ReportSchemaRegistry` used in all validation operations.

#### 2. `validate(array $queryConfig): array`
**Purpose**: Performs all validation steps on the report configuration and returns the result.

**Steps**:
- Initialize `$errors` and `$warnings`.
- Validate the basic structure (`validateBasicStructure`). If it fails, stop (subsequent errors may be meaningless).
- Validate the table (`validateTable`).
- Validate columns (`validateColumns`).
- Validate joins (`validateJoins`).
- Validate conditions (`validateConditions`).
- Validate GROUP BY (`validateGroupBy`).
- Validate ORDER BY (`validateSortBy`).
- Validate LIMIT (`validateLimit`).
- Perform security checks (`performSecurityChecks`).
- Perform performance checks (`performPerformanceChecks`).
- Generate a validation summary (`generateValidationSummary`).

**Returns**: An array containing:
- `valid`: boolean (true if `$errors` is empty).
- `errors`: array of error messages.
- `warnings`: array of warning messages.
- `summary`: array containing a summary (e.g., complexity score, column count, etc.).

#### 3. `validateBasicStructure(array $queryConfig)`
- Uses `Illuminate\Support\Facades\Validator` to check for the presence of required keys with the correct format.
- **Rules**:
  - `table` is required and must be an array.
  - `table.name` is required and must be a string.
  - `columns` is required and must be an array containing at least one element.
  - `joins` (optional) must be an array.
  - `conditions` (optional) must be an array.
  - `groupBy` (optional) must be an array.
  - `sortBy` (optional) must be an array.
  - `limit` (optional) must be an integer between 1 and 50000.

#### 4. `validateTable(array $queryConfig)`
- Checks if the table exists in the registry (`$registry->hasTable($tableName)`).
- If the table does not exist, adds an error.
- (Commented) Check permissions if they exist in the table schema.
- If `limit` is specified, checks if it exceeds the table's `maxRows` (if defined) and adds a warning.

#### 5. `validateColumns(array $queryConfig)`
- Gets the list of available columns from the registry (`$registry->getTableColumns($tableName)`).
- For each column in the configuration, calls `validateColumn`.
- If the number of columns exceeds 50, adds a warning.

#### 6. `validateColumn(array $column, int $index, array $availableColumns, string $tableName)`
- Checks for the presence of `name` and optional `type` using Validator.
- Verifies that the column name exists in `$availableColumns`.
- If `alias` is specified, verifies that it follows the pattern `[a-zA-Z_][a-zA-Z0-9_]*`.

#### 7. `validateJoins(array $queryConfig)`
- Gets the list of available relationships for the main table (`$registry->getTableRelationships($mainTable)`).
- For each join, calls `validateJoin`.
- If the number of joins exceeds 5, adds a warning.

#### 8. `validateJoin(array $join, int $index, array $availableRelationships, string $mainTable)`
- Checks for the presence of `table`, `type` (left/right/inner), `local_key`, `foreign_key` using Validator.
- Verifies that the relationship exists in `$availableRelationships` (using the key `$join['key']` or the table name).
- Verifies that the related table exists in the registry (`$registry->hasTable($joinTable)`).
- If `selectedColumns` is specified, validates each column using `validateColumn` with the related table.

#### 9. `validateConditions(array $queryConfig)`
- Calls `validateConditionGroup` on the conditions array.
- Counts the number of conditions using `countConditions`; if it exceeds 20, adds a warning.

#### 10. `validateConditionGroup(array $conditions, array $queryConfig, string $path = 'conditions')`
- Iterates through each condition. If a condition contains a `conditions` key (nested group), it calls itself. Otherwise, it calls `validateCondition`.

#### 11. `validateCondition(array $condition, array $queryConfig, string $path)`
- Checks for the presence of `field`, `operator`, `value` with the correct format using Validator.
- Verifies that the field (`field.name`) is available via `isFieldAvailable`.
- Verifies that the operator (`operator.value`) is valid via `isValidOperator`.
- Calls `validateConditionValue` to validate the value based on the operator type.

#### 12. `validateConditionValue(array $condition, string $path)`
- **For `in` / `not_in` operators**: Ensures the value is an array or a string (can be converted to an array).
- **For `between` / `not_between` operators**: Ensures the value is an array containing two elements.
- **For `is_null` / `is_not_null` operators**: No value needed.
- For other cases: If the value is empty, adds a warning.

#### 13. `validateGroupBy(array $queryConfig)`
- For each item in `groupBy`, checks for the presence of `groupBy` (an array containing `name`), and also `aggregateFn` and `aggregateBy` if they exist.
- Verifies that the aggregate function (`aggregateFn.value`) is one of: `count, sum, avg, min, max, group_concat`.
- Verifies that the grouped field (`groupBy.name`) is available via `isFieldAvailable`.
- If `aggregateBy.name` exists, verifies its existence (allowing `*` only with COUNT).

#### 14. `validateSortBy(array $queryConfig)`
- For each item in `sortBy`, checks for the presence of `column` and `direction` with the correct format.
- Verifies that `direction.value` is either `asc` or `desc`.
- Verifies that the field (`column.name`) is available via `isFieldAvailable`.

#### 15. `validateLimit(array $queryConfig)`
- If `limit` exists, checks that it is a positive number.
- If it exceeds 50000, adds an error.
- If it exceeds 10000, adds a warning.

#### 16. `isFieldAvailable(string $fieldName, string $tableName, array $queryConfig): bool`
- Checks if the field exists in the main table or in any of the related tables via the specified joins.
- Uses `$registry->getTableColumns($tableName)` to get column names.

#### 17. `isValidOperator(string $operator): bool`
- Checks if the operator is in the allowed list:
  `eq`, `=`, `neq`, `!=`, `gt`, `>`, `gte`, `>=`, `lt`, `<`, `lte`, `<=`, `like`, `not_like`, `in`, `not_in`, `between`, `not_between`, `is_null`, `is_not_null`, `contains`, `starts_with`, `ends_with`.

#### 18. `countConditions(array $conditions): int`
- Recursively counts the number of conditions (including nested conditions).

#### 19. `calculateComplexity(array $queryConfig): string`
- Assigns a complexity level (low, medium, high) based on:
  - Number of columns.
  - Number of joins × 3.
  - Number of conditions × 2.
  - Number of GROUP BY × 4.
  - Number of ORDER BY.
- **low**: < 10 points, **medium**: < 25, **high**: ≥ 25.

#### 20. `performSecurityChecks(array $queryConfig)`
- Calls `checkForSqlInjection` to search for dangerous patterns (e.g., `union select`, `drop table`).
- Calls `checkSensitiveDataAccess` to search for sensitive columns (password, token, secret, key, ssn, credit_card).
- Calls `checkResourceUsage` to estimate resource consumption.

#### 21. `performPerformanceChecks(array $queryConfig)`
- Calls `checkIndexUsage` to check for conditions on indexed columns (a simple check for common columns like `id`, `uuid`, `created_at`, `updated_at`).
- Calls `checkCartesianProducts` to warn about multiple joins that might produce a Cartesian product.

#### 22. `generateValidationSummary(array $queryConfig): array`
- Creates an array containing:
  - `complexity`: result of `calculateComplexity`.
  - `total_columns`: number of columns.
  - `total_joins`: number of joins.
  - `total_conditions`: number of conditions.
  - `has_grouping`: whether GROUP BY exists.
  - `has_sorting`: whether ORDER BY exists.
  - `has_limit`: whether LIMIT exists.
  - `estimated_performance`: performance estimate (`estimatePerformance`).

#### 23. `estimatePerformance(array $queryConfig): string`
- Estimates performance (`fast`, `moderate`, `slow`) based on complexity, number of joins, and conditions.

---

### Practical Examples

#### Example 1: Simple and Correct Report Configuration
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'uuid', 'alias' => 'order_id'],
        ['name' => 'amount', 'type' => 'decimal'],
    ],
    'conditions' => [
        [
            'field' => ['name' => 'status'],
            'operator' => ['value' => '='],
            'value' => 'completed'
        ]
    ],
    'limit' => 100
];

$validator = new ReportQueryValidator($registry);
$result = $validator->validate($queryConfig);

// $result['valid'] = true
// $result['errors'] = []
// $result['warnings'] = []
// $result['summary'] = ['complexity' => 'low', 'estimated_performance' => 'fast', ...]
```

#### Example 2: Column Does Not Exist in the Table
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'non_existent_column'],
    ],
];

$result = $validator->validate($queryConfig);
// $result['valid'] = false
// $result['errors'] = ["Column 'non_existent_column' does not exist in table 'orders'"]
```

#### Example 3: Invalid Alias Format
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'uuid', 'alias' => 'order id'], // space not allowed
    ],
];

$result = $validator->validate($queryConfig);
// $result['errors'] = ["Column 0: Invalid alias format 'order id' for column 'uuid'"]
```

#### Example 4: Condition with Invalid Operator
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [['name' => 'uuid']],
    'conditions' => [
        [
            'field' => ['name' => 'amount'],
            'operator' => ['value' => 'invalid_operator'],
            'value' => 100
        ]
    ],
];

$result = $validator->validate($queryConfig);
// $result['errors'] = ["conditions[0]: Invalid operator 'invalid_operator'"]
```

#### Example 5: `in` Condition with Non-Array Value
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [['name' => 'uuid']],
    'conditions' => [
        [
            'field' => ['name' => 'status'],
            'operator' => ['value' => 'in'],
            'value' => 'completed,pending' // string, not array
        ]
    ],
];

$result = $validator->validate($queryConfig);
// No error here because the function accepts a string (it will be converted later), but there might be a warning if the value is inappropriate.
```

#### Example 6: Configuration with GROUP BY and Non-Aggregated Columns
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'customer_name'], // This column is not in groupBy
        ['name' => 'amount'],
    ],
    'groupBy' => [
        [
            'groupBy' => ['name' => 'customer_name'],
            'aggregateFn' => ['value' => 'sum'],
            'aggregateBy' => ['name' => 'amount']
        ]
    ],
];

$result = $validator->validate($queryConfig);
// In validateQueryConfig (which is called later in the converter) an error will be thrown, but in this class there is no check for that.
// However, validateGroupBy will check for the existence of the fields.
```

#### Example 7: Exceeding the LIMIT Limit
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [['name' => 'uuid']],
    'limit' => 60000,
];

$result = $validator->validate($queryConfig);
// $result['errors'] = ["Limit cannot exceed 50,000 rows"]
// $result['warnings'] might contain a warning for exceeding 10000.
```

#### Example 8: Many Nested Conditions (Performance Warning)
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [['name' => 'uuid']],
    'conditions' => [
        ['field' => ['name' => 'status'], 'operator' => ['value' => '='], 'value' => 'a'],
        ['field' => ['name' => 'status'], 'operator' => ['value' => '='], 'value' => 'b'],
        // ... 25 conditions
    ],
];

$result = $validator->validate($queryConfig);
// $result['warnings'] = ["Many conditions (25) may impact query performance"]
```

#### Example 9: Presence of Forbidden Keyword (SQL Injection)
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [['name' => 'uuid']],
    'conditions' => [
        [
            'field' => ['name' => 'note'],
            'operator' => ['value' => 'like'],
            'value' => "%'; DROP TABLE users; --" // injection attempt
        ]
    ],
];

$result = $validator->validate($queryConfig);
// $result['errors'] = ["Potential SQL injection detected: value"]
```

#### Example 10: Accessing a Sensitive Column (Warning)
```php
$queryConfig = [
    'table' => ['name' => 'users'],
    'columns' => [
        ['name' => 'email'],
        ['name' => 'password_hash'], // column might be sensitive
    ],
];

$result = $validator->validate($queryConfig);
// $result['warnings'] = ["Accessing potentially sensitive column: password_hash"]
```

---

### Important Notes
- The class **does not modify the configuration**; it only issues judgments about it.
- Some checks rely on the existence of `$registry` with all tables and columns registered.
- Column validation in conditions, joins, and group by is done via `isFieldAvailable`, which searches in the main table and the related tables specified in `joins`. This means a column from a relationship not added as a join will be considered unavailable (since it cannot be referenced without a join).
- The performance check (`checkIndexUsage`) is very basic and relies on common column names; it could be improved by querying the actual database indexes.
- The SQL injection check (`checkForSqlInjection`) looks for known patterns but is not a substitute for the Prepared Statements settings provided by Laravel itself.
- Developers can extend the class by adding custom checks that suit their application.

---

### Summary
`ReportQueryValidator` is the first line of defense before building and executing any report. By validating the configuration, it ensures that the queries to be generated are syntactically correct and secure, and it provides valuable warnings to the user about potential performance. This helps in building a robust and reliable reporting system.