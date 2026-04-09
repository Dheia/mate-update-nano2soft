## `QueryBuilderParserHelper` Class Documentation

**Overview of the `Nano2.QueryBuilder` Package**

The `QueryBuilderParserHelper` class is part of the **`Nano2.QueryBuilder`** package, an integrated software package provided by **NanoSoft**, designed specifically for **Laravel** applications to facilitate building and managing complex queries and dynamic reports in a secure and flexible manner.

This class resides under the following namespace:

```
Nano2\QueryBuilder\Classes
```

### Introduction
The `QueryBuilderParserHelper` class is a helper tool for parsing query rules coming from libraries such as **jQuery QueryBuilder** and applying them to Laravel's `Eloquent Builder` or `Query Builder`. The class converts a JSON object representing a set of rules into appropriate `WHERE` and `JOIN` conditions, supporting logical operations (`AND` / `OR`), various operators (e.g., `=`, `>`, `LIKE`, `IN`, `BETWEEN`, etc.), and handling relationships via dot notation (e.g., `customer.name`).

The class is primarily based on the `timgws/QueryBuilderParser` library but adds a set of enhancements and helper methods such as:
- Extracting used tables from rules (`getTables`).
- Converting rules to an SQL query (`toSql`).
- Expanding custom macros (`expandMacro`).
- Applying rules directly to a Query Builder object (`parse`).
- Handling different inputs (model, Builder, JSON string).

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$legacy_operators` | `array` | (static) Array mapping legacy operators (e.g., `=`, `!=`) to new operator names. |
| `$operators` | `array` | (static) Array mapping operator names (e.g., `equal`) to actual SQL operators (e.g., `=`). |
| `$values` | `array` | (static) Array defining how to format values for special operators (e.g., `between`, `contains`). |

---

### Important Methods

#### 1. `getTables`
```php
public static function getTables($builder): array
```
**Purpose**: Extract all table names used in the query rules (including tables resulting from macros).

**Parameters**:
- `$builder`: The rules array as coming from jQuery QueryBuilder (must contain `rules` and `condition` keys).

**Returns**: Array of table names (without duplicates).

**Example**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'orders.amount', 'operator' => 'greater', 'value' => 100],
        ['field' => 'customer.name', 'operator' => 'begins_with', 'value' => 'A']
    ]
];
$tables = QueryBuilderParserHelper::getTables($rules);
// $tables = ['orders', 'customer']
```

---

#### 2. `findTablesRecursive` (protected)
```php
protected static function findTablesRecursive($rules): array
```
**Purpose**: A helper function used by `getTables` to recursively search for tables in nested rules.

---

#### 3. `toSql`
```php
public static function toSql($builder, $expand = true): ?string
```
**Purpose**: Convert query rules to an executable SQL string (WHERE part) with optional macro expansion.

**Parameters**:
- `$builder`: The rules array.
- `$expand`: (optional) If `true`, expands macros (default `true`).

**Returns**: SQL string representing the WHERE condition (may be `null` if rules are empty).

**Note**: This function does not generate a full `SELECT` statement, only the condition part.

**Example**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'orders.amount', 'operator' => 'greater', 'value' => 100],
        ['field' => 'customer.name', 'operator' => 'begins_with', 'value' => 'A']
    ]
];
$sql = QueryBuilderParserHelper::toSql($rules);
// Output: (orders.amount > 100 AND customer.name LIKE 'A%')
```

---

#### 4. `parseGroup` (protected)
```php
private static function parseGroup($rule, $expand = false, $wrap = true): string
```
**Purpose**: Parse a group of rules (nested group) and convert it to SQL string.

---

#### 5. `parseRule` (protected)
```php
protected static function parseRule($rule, $expand = false): string
```
**Purpose**: Parse a single rule and convert it to SQL string.

---

#### 6. `expandMacro`
```php
protected static function expandMacro($subject, $tables_only = false, $depth_limit = 20)
```
**Purpose**: Expand custom macros in field names or values. Macros are shortcuts defined in the configuration file `nano2.querybuilder::macros.rule` that are replaced with SQL expressions.

**Parameters**:
- `$subject`: The text that may contain a macro (e.g., `%macros.total_with_tax`).
- `$tables_only`: If `true`, returns the list of tables used in the macro instead of the expanded text.
- `$depth_limit`: Maximum number of recursive expansions (to prevent infinite loops).

**Returns**: The expanded text, or an array of table names if `tables_only = true`.

**Example (hypothetical)**:
```php
// Assuming configuration contains:
// Config::set('nano2.querybuilder::macros.rule', ['total_with_tax' => 'ROUND(amount * 1.14, 2)']);
$expanded = QueryBuilderParserHelper::expandMacro('%macros.total_with_tax');
// Output: (ROUND(amount * 1.14, 2))
```

---

#### 7. `runMacros`
```php
public static function runMacros($rule, $x = 1)
```
**Purpose**: An alternative method to expand macros (uses limited recursion). Replaces all `%macros.` with corresponding values from configuration.

**Parameters**:
- `$rule`: The text containing macros.
- `$x`: Counter to prevent infinite recursion.

**Returns**: Expanded text, or `false` if the depth limit is exceeded.

---

#### 8. `parseRuleToQuery`
```php
public static function parseRuleToQuery($query, $rule, $condition)
```
**Purpose**: Apply a single rule to a Query Builder object (e.g., `Eloquent\Builder`) with the specified logical condition (`AND` / `OR`).

**Parameters**:
- `$query`: Builder object (can be `EloquentBuilder` or `Query\Builder`).
- `$rule`: Rule array (containing `field`, `operator`, `value`).
- `$condition`: `'and'` or `'or'` (used to link conditions).

**Returns**: Builder object after adding the condition.

**Example**:
```php
$query = DB::table('orders');
$rule = ['field' => 'amount', 'operator' => 'greater', 'value' => 100];
$query = QueryBuilderParserHelper::parseRuleToQuery($query, $rule, 'and');
// Adds: where 'amount' > 100
```

---

#### 9. `expandRule` (protected)
```php
protected static function expandRule($rule): array
```
**Purpose**: Expand the field and value in the rule if they contain a macro or a raw value (surrounded by `` ` ``). Returns an array `[$field, $op, $value]` after expansion.

---

#### 10. `parse` (Main Method)
```php
public static function parse($queryBuilder, $rules, $rows, $is_exception = true)
```
**Purpose**: Apply a full set of rules to a Builder object using the `timgws\QueryBuilderParser` library. This is the main interface for usage.

**Parameters**:
- `$queryBuilder`: Can be:
  - A Builder object (`EloquentBuilder` or `Query\Builder`).
  - An Eloquent Model – will be converted to a Builder.
  - A class name (string) – a new instance will be created.
- `$rules`: Rules as coming from jQuery QueryBuilder. Can be:
  - An array.
  - A JSON string.
- `$rows`: Array of allowed field names (must be passed to the library). If empty, it will be automatically extracted from the rules using `setFields`.
- `$is_exception`: If `true`, an exception will be thrown on error (e.g., invalid input). If `false`, returns the original input unchanged.

**Returns**: Builder object with all conditions added.

**Example**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'status', 'operator' => 'equal', 'value' => 'completed'],
        ['field' => 'amount', 'operator' => 'greater', 'value' => 100]
    ]
];
$query = DB::table('orders');
$query = QueryBuilderParserHelper::parse($query, $rules, []);
// Output: query with where status = 'completed' and amount > 100
```

---

#### 11. `setFields`
```php
public static function setFields($rules, $is_encode = true, $is_decode = true): array
```
**Purpose**: Extract field names (keys) used in the rules. Used to create the list of allowed fields for the `QueryBuilderParser` library.

**Parameters**:
- `$rules`: The rules (can be array or JSON).
- `$is_encode`: If `true` and `$rules` is an array, it is first converted to JSON.
- `$is_decode`: If `true` and `$rules` is a string, it is JSON decoded.

**Returns**: Associative array with field names (key and value are the same name).

**Example**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'status', 'operator' => 'equal', 'value' => 'completed'],
        ['field' => 'amount', 'operator' => 'greater', 'value' => 100]
    ]
];
$fields = QueryBuilderParserHelper::setFields($rules);
// $fields = ['status' => 'status', 'amount' => 'amount']
```

---

#### 12. `decodeJSON`
```php
public static function decodeJSON($json): object
```
**Purpose**: Decode a JSON string to an object with error handling.

**Parameters**:
- `$json`: The string to decode.

**Returns**: `stdClass` object.

**Exceptions**: Throws `Exception` if JSON is invalid.

---

#### 13. `isNested`
```php
public static function isNested($rule): bool
```
**Purpose**: Check whether a rule contains nested rules (i.e., it is a group, not a simple rule).

**Parameters**:
- `$rule`: Object or array representing a rule (may contain a `rules` key).

**Returns**: `true` if it is a group, otherwise `false`.

---

#### 14. `getQueryOrModel`
```php
public static function getQueryOrModel($obj, $is_model = false, $is_exception = true)
```
**Purpose**: Convert the input (which may be a class name, Model object, or Builder object) to a Builder (or Model as requested). Used to unify handling of different inputs.

**Parameters**:
- `$obj`: Input (string, Model, Builder).
- `$is_model`: If `true`, returns a Model object instead of a Builder.
- `$is_exception`: If `true`, throws an exception on conversion failure.

**Returns**: Builder or Model object (depending on `$is_model`).

**Example**:
```php
$builder = QueryBuilderParserHelper::getQueryOrModel('App\Models\Order'); // Returns Builder for orders table
$model = QueryBuilderParserHelper::getQueryOrModel('App\Models\Order', true); // Returns new Order object
```

---

#### 15. `isModel`
```php
public static function isModel($item): bool
```
**Purpose**: Check if the object is an Eloquent Model (from Laravel or OctoberCMS).

---

#### 16. `isBuilder`
```php
public static function isBuilder($item): bool
```
**Purpose**: Check if the object is a Builder (from Laravel Query Builder, Eloquent Builder, or October builders).

---

### Comprehensive Practical Examples

#### Example 1: Applying Simple Rules to a Query
**Scenario**: We have a query on the `orders` table and want to apply two conditions: `status = 'completed'` and `amount > 100`.

**Code**:
```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'status', 'operator' => 'equal', 'value' => 'completed'],
        ['field' => 'amount', 'operator' => 'greater', 'value' => 100]
    ]
];

$query = DB::table('orders');
$optimizedQuery = QueryBuilderParserHelper::parse($query, $rules, []);
$results = $optimizedQuery->get();
```

**Resulting SQL**:
```sql
select * from orders where status = 'completed' and amount > 100
```

---

#### Example 2: Using Relationships (Dot Notation)
**Scenario**: Search for orders where the customer name begins with "A".

**Code**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'customer.name', 'operator' => 'begins_with', 'value' => 'A']
    ]
];

$query = DB::table('orders');
$optimizedQuery = QueryBuilderParserHelper::parse($query, $rules, []);
// Will automatically join with the customers table if the relationship is defined
```

**Resulting SQL (approximate)**:
```sql
select * from orders inner join customers on orders.customer_id = customers.id where customers.name like 'A%'
```

---

#### Example 3: Using Nested Conditions
**Scenario**: (status = 'completed' OR status = 'pending') AND amount > 100

**Code**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'condition' => 'OR',
            'rules' => [
                ['field' => 'status', 'operator' => 'equal', 'value' => 'completed'],
                ['field' => 'status', 'operator' => 'equal', 'value' => 'pending']
            ]
        ],
        ['field' => 'amount', 'operator' => 'greater', 'value' => 100]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**Resulting SQL**:
```sql
select * from orders where (status = 'completed' or status = 'pending') and amount > 100
```

---

#### Example 4: Using the `IN` Operator
**Scenario**: Get orders with specific statuses (completed, pending, cancelled).

**Code**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'status',
            'operator' => 'in',
            'value' => 'completed,pending,cancelled' // can also be an array
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**Resulting SQL**:
```sql
select * from orders where status in ('completed', 'pending', 'cancelled')
```

---

#### Example 5: Using the `BETWEEN` Operator
**Scenario**: Orders with amount between 100 and 500.

**Code**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'amount',
            'operator' => 'between',
            'value' => [100, 500]
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**Resulting SQL**:
```sql
select * from orders where amount between 100 and 500
```

---

#### Example 6: Using the `LIKE` Operator with `contains`
**Scenario**: Search for orders where notes contain the word "urgent".

**Code**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'notes',
            'operator' => 'contains',
            'value' => 'urgent'
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**Resulting SQL**:
```sql
select * from orders where notes like '%urgent%'
```

---

#### Example 7: Using the `IS NULL` Operator
**Scenario**: Orders that have no deletion date (deleted_at = null).

**Code**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'deleted_at',
            'operator' => 'is_null',
            'value' => null // value is not used
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**Resulting SQL**:
```sql
select * from orders where deleted_at is null
```

---

#### Example 8: Using Custom Macros
**Scenario**: We have a macro named `total_with_tax` that calculates the amount including tax. We want to search for orders where `total_with_tax` is greater than 500.

**Prerequisite configuration** (in config file or Service Provider):
```php
Config::set('nano2.querybuilder::macros.rule', [
    'total_with_tax' => 'ROUND(amount * 1.14, 2)'
]);
```

**Code**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'macros.total_with_tax', // using the macro in the field
            'operator' => 'greater',
            'value' => 500
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**Resulting SQL**:
```sql
select * from orders where ROUND(amount * 1.14, 2) > 500
```

---

#### Example 9: Using a Value Surrounded by `` ` `` to Reference Another Column
**Scenario**: Search for orders where `amount` is greater than `tax` (another column).

**Code**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'amount',
            'operator' => 'greater',
            'value' => '`tax`' // value surrounded by ` indicates a column
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**Resulting SQL**:
```sql
select * from orders where amount > tax
```

---

#### Example 10: Extracting Used Tables
**Scenario**: Before executing the query, we want to know which tables will be affected.

**Code**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'orders.amount', 'operator' => 'greater', 'value' => 100],
        ['field' => 'customer.name', 'operator' => 'begins_with', 'value' => 'A']
    ]
];
$tables = QueryBuilderParserHelper::getTables($rules);
// $tables = ['orders', 'customer']
```

---

#### Example 11: Converting Rules to SQL (Without Execution)
**Scenario**: Get the SQL string of the conditions only.

**Code**:
```php
$sqlWhere = QueryBuilderParserHelper::toSql($rules);
// $sqlWhere = "(orders.amount > 100 AND customer.name LIKE 'A%')"
```

---

#### Example 12: Using `parse` Directly with a Model
**Scenario**: Pass a Model class name instead of a Builder.

**Code**:
```php
$rules = [ /* ... */ ];
$query = QueryBuilderParserHelper::parse('App\Models\Order', $rules, []);
// Returns a ready-to-use Builder
$orders = $query->get();
```

---

#### Example 13: Handling JSON Coming from a Request
**Scenario**: An API endpoint receives rules as JSON.

**Code**:
```php
public function search(Request $request)
{
    $rulesJson = $request->input('rules'); // JSON string
    $query = DB::table('orders');
    $query = QueryBuilderParserHelper::parse($query, $rulesJson, []);
    return $query->paginate();
}
```

---

#### Example 14: Using `setFields` to Extract Allowed Fields
**Scenario**: We want to get the list of fields the user used in the rules.

**Code**:
```php
$rules = [ /* ... */ ];
$fields = QueryBuilderParserHelper::setFields($rules);
// Can be used for permission checks or to feed the QueryBuilderParser library.
```

---

#### Example 15: Checking Input Type
**Scenario**: Verify that a given object is a Builder before using it.

**Code**:
```php
if (QueryBuilderParserHelper::isBuilder($someObject)) {
    // Can apply rules
}
```

---

### Important Notes

- The class depends on the `timgws/QueryBuilderParser` library. Install it via Composer: `composer require timgws/querybuilderparser`
- Macros are defined in the configuration file `nano2.querybuilder::macros.rule`. They can be added via `Config::set` or through the application's configuration files.
- When using dot notation fields (e.g., `customer.name`), the class assumes the first part is a relationship name and will automatically perform a `join` if the relationship is defined in the model. This requires that a relationship with the same name exists in the model.
- The `parse` method uses `setFields` to extract allowed fields if `$rows` is not passed. This ensures that the library will not allow fields not present in the rules.
- In case of JSON or input errors, exception throwing can be controlled via `$is_exception`.

---

### Conclusion

The `QueryBuilderParserHelper` class provides an integrated and easy-to-use interface for parsing jQuery QueryBuilder rules and applying them to Laravel queries. Thanks to its support for relationships, macros, and various operators, developers can build advanced search interfaces and dynamic reports quickly and securely. The class integrates closely with other components of the `Nano2.QueryBuilder` package, such as `ReportQueryConverter` and `ReportsManager`, making it a powerful tool for building reporting systems.

---

## Additional Documentation

- [`QueryBuilder` FormWidget Documentation](./Docs-FormWidgets-QueryBuilder-en.md)
