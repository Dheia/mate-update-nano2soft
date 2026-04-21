## Documentation of `ComputedColumnValidator` Class

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The `ComputedColumnValidator` class is responsible for **validating the safety and correctness of computed column expressions** in the dynamic reporting system. It is used to ensure that any SQL expression provided by the user (or coming from the frontend) does not contain forbidden keywords, uses only allowed functions, and references columns that actually exist in the database (with support for relationships). This prevents SQL injection attacks and errors resulting from invalid references.

---

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$allowedFunctions` | `array` | List of functions allowed to be used in the expression (e.g., `CONCAT`, `ROUND`, `DATEDIFF`). |
| `$allowedOperators` | `array` | List of allowed mathematical and logical operators (e.g., `+`, `-`, `*`, `/`, `AND`, `OR`). |
| `$forbiddenKeywords` | `array` | List of forbidden keywords that indicate dangerous operations (e.g., `DROP`, `DELETE`, `UNION`). |
| `$registry` | `ReportSchemaRegistry` | Reference to the schema registry to verify the existence of columns and relationships. |

---

### Main Methods

#### 1. `__construct(ReportSchemaRegistry $registry)`
- Injects the `ReportSchemaRegistry` that will be used to verify the existence of columns and relationships.

#### 2. `validate(string $expression, string $tableName, array $computedColumns = []): array`
**Purpose**: Executes all validation steps on a computed column expression and returns the result.

**Parameters**:
- `$expression`: The SQL expression to be validated (e.g., `"CONCAT(first_name, ' ', last_name)"`).
- `$tableName`: The name of the base table (against which the query will be executed).
- `$computedColumns`: An array of other computed columns (optional) to allow cross-references between them.

**Steps**:
1. Check for forbidden keywords (`checkForbiddenKeywords`).
2. Verify that all used functions are within the allowed list (`validateFunctions`).
3. Ensure the used operators are safe (`validateOperators`).
4. Verify that all column references exist in the table or in other computed columns (`validateColumnReferences`).
5. Aggregate errors and return an array containing `valid` (boolean) and `errors` (array of messages).

#### 3. `checkForbiddenKeywords(string $expression): array`
- Searches the expression for any word from `$forbiddenKeywords` (respecting word boundaries using `\b`). If found, returns an error.

#### 4. `validateFunctions(string $expression): array`
- Extracts function names from the expression (using a regex that looks for the pattern `WORD(`) and ensures each function is present in `$allowedFunctions`.

#### 5. `validateOperators(string $expression): array`
- Removes string literals from the expression (to avoid false positives) and then looks for dangerous operators like `;`, `--`, `/*`. If found, returns an error.

#### 6. `validateColumnReferences(string $expression, string $tableName, array $computedColumns = []): array`
**Purpose**: Ensures that every column reference (e.g., `amount` or `customer.name`) points to an existing column.

**Steps**:
- Obtains a `Table` object from `$registry` using `$tableName`.
- Builds a list of other computed column names (to allow cross-references).
- Removes string literals from the expression.
- Extracts all words that could be column names (using `\b[a-z_][a-z0-9_]*\b`).
- For each word, checks if it is a reserved word (function, operator, keyword) or a number, and ignores it.
- If the word is present in the `$computedColumns` list, it is considered valid (reference to another computed column).
- Otherwise, calls `isValidColumnReference` to check if the column exists in the table or its relationships.

#### 7. `isValidColumnReference(string $columnRef, Table $table): bool`
- Determines whether a column reference (e.g., `amount` or `payload.pickup.city`) is valid.
- If the reference is without a dot, checks for its existence in the table using `columnExistsInTable`.
- If the reference contains dots (e.g., `customer.name`), attempts to:
  - Check if the first part (`customer`) is an existing JSON column.
  - Or look for a relationship named `customer` in the table (using `getRelationships`). If the relationship exists, the reference is considered valid (the validation could be improved to also check the columns of the related table).
- If the reference has more than two dots (e.g., `payload.pickup.city`), it is considered valid by default (the logic can be extended later).

#### 8. `columnExistsInTable(string $columnName, Table $table): bool`
- Checks whether a column named `$columnName` exists in the table (by searching in `getColumns()`). It also supports JSON columns where the name might be like `details.address` – it checks the prefix `details`.

#### 9. `getAllowedFunctions()` and `getAllowedOperators()`
- Accessor methods to return the full lists of allowed functions and operators (may be useful for the frontend).

---

### Practical Examples

#### Example 1: Simple and correct expression
```php
use Nano2\QueryBuilder\Classes\Reporting\ComputedColumnValidator;
use Nano2\QueryBuilder\Classes\Reporting\ReportSchemaRegistry;

// Assuming $registry is registered with a 'users' table having columns first_name and last_name
$validator = new ComputedColumnValidator($registry);

$expression = "CONCAT(first_name, ' ', last_name) AS full_name";
$result = $validator->validate($expression, 'users');

if ($result['valid']) {
    echo "Expression is valid!";
} else {
    echo "Errors: " . implode(', ', $result['errors']);
}
// Output: "Expression is valid!"
```

#### Example 2: Using a disallowed function
```php
$expression = "EXEC sp_help"; // Dangerous function
$result = $validator->validate($expression, 'users');
// $result['valid'] = false
// $result['errors'] = ["Function 'EXEC' is not allowed. Allowed functions: ..."]
```

#### Example 3: Forbidden keyword (DROP)
```php
$expression = "SELECT * FROM users; DROP TABLE users; --";
$result = $validator->validate($expression, 'users');
// $result['errors'] = ["Expression contains forbidden SQL keyword: DROP"]
```

#### Example 4: Non-existent column reference
```php
$expression = "first_name + last_name + non_existent_column";
$result = $validator->validate($expression, 'users');
// $result['errors'] = ["Column reference 'non_existent_column' does not exist in table 'users' or its relationships"]
```

#### Example 5: Expression with a relationship (auto-join)
```php
// Assuming the 'users' table has an auto-join relationship named 'profile' containing a column 'age'
$expression = "profile.age * 2 AS double_age";
$result = $validator->validate($expression, 'users');
// Valid because profile.age refers to an existing relationship.
```

#### Example 6: Expression with nested relationship
```php
// Assuming the 'orders' table has a relationship 'payload', which in turn has a relationship 'pickup' with a column 'city'
$expression = "UPPER(payload.pickup.city) AS pickup_city_upper";
$result = $validator->validate($expression, 'orders');
// Valid (as long as the path payload.pickup.city exists in the registry).
```

#### Example 7: Reference to another computed column
```php
$computedColumns = [
    ['name' => 'full_name', 'expression' => "CONCAT(first_name, ' ', last_name)"]
];
$expression = "LENGTH(full_name) AS name_length";
$result = $validator->validate($expression, 'users', $computedColumns);
// Valid because full_name is present in the list of computed columns.
```

#### Example 8: Dangerous operator (SQL comment)
```php
$expression = "amount * 10 -- this is a comment";
$result = $validator->validate($expression, 'orders');
// $result['errors'] = ["Operator '--' is not allowed"]
```

---

### List of Allowed Functions (`$allowedFunctions`)
Includes common and safe functions such as:
- **Date functions**: `DATEDIFF`, `DATE_ADD`, `NOW`, `YEAR`, `MONTH`, `DAY`, `DATE_FORMAT`, etc.
- **String functions**: `CONCAT`, `SUBSTRING`, `REPLACE`, `UPPER`, `LOWER`, `TRIM`, `LENGTH`.
- **Numeric functions**: `ROUND`, `ABS`, `CEIL`, `FLOOR`, `MOD`, `POWER`, `SQRT`.
- **Conditional functions**: `CASE`, `IF`, `IFNULL`, `COALESCE`.
- **Aggregate functions**: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` (allowed only in aggregation context, but the class does not prevent them).
- **Conversion functions**: `CAST`, `CONVERT`.

### List of Allowed Operators (`$allowedOperators`)
- Mathematical: `+`, `-`, `*`, `/`, `%`
- Comparison: `=`, `!=`, `<>`, `<`, `>`, `<=`, `>=`
- Logical: `AND`, `OR`, `NOT`
- Others: `IS`, `NULL`

### List of Forbidden Keywords (`$forbiddenKeywords`)
Includes any word that could alter data or database structure: `DROP`, `DELETE`, `UPDATE`, `INSERT`, `TRUNCATE`, `ALTER`, `CREATE`, `GRANT`, `REVOKE`, `EXEC`, `UNION`, `INTO`, `INFORMATION_SCHEMA`, `LOAD_FILE`, `OUTFILE`, `DUMPFILE`, `BENCHMARK`, `SLEEP`.

---

### Important Notes
- The class **does not execute the expression**, it only checks its syntactic and security soundness.
- Validation of column existence in relationships is done in a simplified manner (just checking the relationship name); it can be extended to also verify the final column.
- Computed columns can reference each other, but they must be passed in `$computedColumns` to avoid errors.
- String literals are removed before checking operators and forbidden keywords to prevent false positives (e.g., having `--` inside plain text).

---

### Summary
`ComputedColumnValidator` is an essential tool for ensuring the security and correctness of computed columns in the reporting system. Thanks to the lists of allowed functions and forbidden keywords, and the verification of column existence via `ReportSchemaRegistry`, developers can allow users to write custom SQL expressions without risking application security or data integrity.