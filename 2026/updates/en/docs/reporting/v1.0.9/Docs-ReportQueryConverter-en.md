## Documentation of `ReportQueryConverter` Class

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The `ReportQueryConverter` class is the core component responsible for converting a report configuration (Query Configuration) into an executable Laravel Query Builder query, then executing it and returning the results along with metadata. This class receives the `queryConfig` that typically comes from the frontend (after validation) and dynamically builds an SQL query taking into account relationships (joins), computed columns, aggregations, conditions, and sorting.

### Main Objectives
- Build a secure and efficient SQL query based on user configuration.
- Apply security constraints such as company scope.
- Automatically handle auto-joins when using columns from relationships.
- Integrate computed columns and aggregations.
- Return results with additional information such as execution time, the executed query, and the selected columns.

---

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$registry` | `ReportSchemaRegistry` | Reference to the schema registry to access table and relationship information. |
| `$queryConfig` | `array` | The report configuration as received from the frontend. |
| `$autoJoins` | `array` | Records information about applied auto-joins (for tracking). |
| `$manualJoins` | `array` | Records information about manual joins from the configuration. |
| `$joinAliases` | `array` | A dictionary mapping a relationship path (e.g., `payload.pickup`) to the table alias used in the query. |
| `$emittedAggregates` | `array` | Records information about aggregate columns that were added (for reporting in results). |
| `$aliasCounter` | `int` | Counter for generating unique aliases (rarely used). |

---

### General Flow of the Class

1. **Construction**: The object is created by passing `$registry` and `$queryConfig`. During construction, `extractComputedColumnsFromAggregates()` is called to extract computed columns from `groupBy` if present.
2. **Execution**: The `execute()` method is the main entry point. It performs basic validation, builds the query, executes it, processes the results, and returns the response.
3. **Query Building**: The `buildQuery()` method builds a `Builder` object step by step:
   - Determine the base table.
   - Apply company scope (`applyCompanyScope`).
   - Process auto-joins (`processAutoJoins`).
   - Process manual joins (`processManualJoins`).
   - Build the SELECT clause (`buildSelectClause`).
   - Build the WHERE clause (`buildWhereClause`).
   - Build the GROUP BY clause (`buildGroupByClause`).
   - Build the ORDER BY clause (`buildOrderByClause`).
   - Build the LIMIT clause (`buildLimitClause`).
4. **Result Processing**: The `processResults` method (currently simple) may be used later to transform data.
5. **Response Return**: Includes `success`, `data`, `columns`, `meta` (execution time, SQL, etc.).

---

### Detailed Method Explanations

#### 1. `__construct(ReportSchemaRegistry $registry, array $queryConfig)`
- Stores `$registry` and `$queryConfig`.
- Calls `extractComputedColumnsFromAggregates()`.

#### 2. `extractComputedColumnsFromAggregates()`
**Purpose**: Extract computed columns from the `groupBy` array if they exist, and add them to `queryConfig['computed_columns']`. This allows defining computed columns within the same aggregation object without duplication.

**Logic**:
- If no `groupBy` exists, exit.
- Ensure `computed_columns` exists in `queryConfig` (add if not).
- For each group in `groupBy`, check `aggregateBy`, and if it has `computed` true, take its `computation` and `name` and add them to `computed_columns` while avoiding duplicates.

#### 3. `execute(): array`
**Steps**:
- Record start time.
- Validate basic configuration (presence of table and columns) via `validateQueryConfig()`.
- Build the query via `buildQuery()`.
- Execute `$query->get()->toArray()`.
- Process results via `processResults()`.
- Calculate execution time.
- Return an array containing `success`, `data`, `columns` (from `getSelectedColumns()`), and `meta`.

In case of error, the exception is caught and a response with `success: false` and an error message is returned.

#### 4. `buildQuery(): Builder`
**Steps**:
- Obtain a `Table` object from `$registry` using the base table name.
- Start the query with `DB::table($tableName)`.
- Apply company scope (`applyCompanyScope`).
- Process auto-joins (`processAutoJoins`).
- Process manual joins (`processManualJoins`).
- Build SELECT clause (`buildSelectClause`).
- Build WHERE clause (`buildWhereClause`).
- Build GROUP BY clause (`buildGroupByClause`).
- Build ORDER BY clause (`buildOrderByClause`).
- Build LIMIT clause (`buildLimitClause`).

#### 5. `applyCompanyScope(Builder $query)`
- Adds a `company_uuid` condition on the base table using the company ID from the session (obtained via `resolveCompanyUuid()`).
- Throws an error if no company is found.

#### 6. `resolveCompanyUuid(): ?string`
- Attempts to extract the company ID from `session('company')`, whether it is a string, an object, or an array.
- Returns `null` if not found.

#### 7. `processAutoJoins(Builder $query, string $tableName)`
**Purpose**: Extract auto-join paths from selected columns, conditions, groupings, and computed columns, then apply them.

**Steps**:
- Collect paths from:
  - `columns` (from `auto_join_path` or from column names containing dots).
  - `conditions` (using `collectAutoJoinPathsFromConditions`).
  - `groupBy` (using `collectAutoJoinPathsFromGroupBy`).
  - `sortBy` (using `collectAutoJoinPathsFromSortBy`).
  - `computed_columns` (using `extractRelationshipPathsFromExpression`).
- Remove duplicates and sort paths by the number of dots (shortest first) to ensure joins are applied in the correct order.
- For each path, call `applyAutoJoinPath`.

#### 8. `collectAutoJoinPathsFromConditions` (and similar helpers)
- Helper methods to extract paths from condition arrays, groupings, and sorting. They look for `auto_join_path` or column names containing dots, and extract the path (all parts except the last).

#### 9. `collectAutoJoinPathsFromComputedColumns`
- Calls `extractRelationshipPathsFromExpression` to extract relationship paths from the computed column expression.

#### 10. `applyAutoJoinPath(Builder $query, string $rootTable, string $fullPath)`
- Ensures the same path has not been applied before (using `$joinAliases`).
- Splits the path into segments (e.g., `payload.pickup`).
- Iterates through each segment, at each step:
  - Obtains the relationship object from the current context (using `getRelationshipFromContext`).
  - If the relationship does not exist or is not auto-joinable, stops.
  - Generates an alias for the table (e.g., `orders_payload_pickup`).
  - Performs a `join` using the specified type (`left`, `right`, `inner`) and local/foreign keys.
  - Records the join in `$autoJoins` and `$joinAliases`.
  - Updates the current context to the aliased table and the next relationship.

#### 11. `getRelationshipFromContext($ctx, string $name)`
- Attempts to obtain the relationship object by name from `$ctx` (which could be a `Table` or a `Relationship`) using methods `getRelationship` or `getAutoJoinRelationships`.

#### 12. `getChildContext($ctx, string $name)`
- Returns the same as `getRelationshipFromContext` (since the new context is the relationship object itself).

#### 13. `generateAliasChain(string $root, array $segments)`
- Generates an alias from the root and segments, e.g., `orders_payload_pickup`.

#### 14. `resolveAliasAndColumn(string $rootTable, string $columnPath)`
- **Purpose**: Convert a column path (e.g., `payload.pickup.address`) into the table alias and the actual column name.
- If the path has no dots, returns `[$rootTable, $columnPath]`.
- If it has dots, separates the last part (the column) from the rest (the relationship path).
- Searches `$joinAliases` for the full path; if found, uses the corresponding alias.
- If not found, walks through the segments step by step, using the last alias found (or falling back to the base table if none found).
- Returns `[$tableAlias, $column]`.

#### 15. `processManualJoins` and `applyManualJoin`
- Applies joins manually specified in `queryConfig['joins']` (if any). For each join, adds a join of the specified type with keys, and records it in `$manualJoins` and `$joinAliases`.

#### 16. `buildSelectClause(Builder $query)`
**Purpose**: Build the SELECT part of the query.

**Logic**:
- Determines `$hasGrouping` (whether there is a `groupBy`).
- If no grouping:
  - Builds an array `$selects` from regular columns (using `resolveAliasAndColumn` for each column) and adds them as `table_alias.column as alias`.
  - Calls `buildComputedColumns` to add computed columns.
- If grouping exists:
  - Adds `groupBy` columns as selected columns (with aliases).
  - For each item in `groupBy`, checks for `aggregateFn`, and builds the aggregate expression:
    - If the aggregate field is `'*'` (for COUNT), uses `COUNT(*)`.
    - If the aggregate field is a computed column (present in `computed_columns`), uses the computed expression after resolving it (`resolveComputedColumnReferences`).
    - Otherwise, uses `table_alias.column` with the aggregate function (COUNT, SUM, AVG, MIN, MAX).
  - Records the aggregates in `$emittedAggregates` with their type and alias.
  - **Note**: In aggregation mode, computed columns are not added as regular columns (as they would violate `ONLY_FULL_GROUP_BY`), but are only used inside aggregate functions.

#### 17. `buildComputedColumns(Builder $query, array &$selects)`
- For each computed column in `queryConfig['computed_columns']`:
  - Validates the expression using `ComputedColumnValidator` (called here).
  - Resolves column references in the expression using `resolveComputedColumnReferences`.
  - Adds the expression to `$selects` as `({$resolvedExpression}) as `{$name}``.

#### 18. `resolveComputedColumnReferences(string $expression, string $rootTable)`
**Purpose**: Convert a computed column expression into a valid query expression by replacing column names with `alias.column` references, while protecting string literals and function names from incorrect replacement.

**Steps**:
1. Recursively expand any references to other computed columns (max 10 times) to avoid infinite loops.
2. Protect string literals (between single or double quotes) by replacing them with temporary placeholders.
3. Protect function names (a word followed by `(`) by replacing them with temporary placeholders.
4. Replace column names (words) with `alias.column` references using `resolveAliasAndColumn`.
5. Restore the protected functions and string literals.

This is a complex process but ensures that parts of the expression that should not be replaced (like string literals or function names) are not altered.

#### 19. `buildWhereClause`, `processConditions`, and `applySingleCondition`
- Builds the WHERE clause by recursively traversing the conditions array.
- For each individual condition:
  - Resolves the field name to `alias.column` using `resolveAliasAndColumn`.
  - Applies the condition based on the operator type (`=`, `>`, `like`, `in`, `null`, `between`, etc.).

#### 20. `buildGroupByClause`
- If `groupBy` exists, collects the grouped columns after converting them to `alias.column`, and adds them to the query's `groupBy`.

#### 21. `buildOrderByClause`
- If `sortBy` exists:
  - In aggregation mode, only allows ordering by column aliases (e.g., `count_uuid` or `customer_name`) that are present in the SELECT to avoid errors.
  - In normal mode, uses `resolveAliasAndColumn` and adds `orderBy` on `alias.column`.

#### 22. `buildLimitClause`
- If `limit` exists, adds `limit` and `offset` (if provided).

#### 23. `getSelectedColumns(): array`
- Builds an array containing information about all columns that will be returned in the results (regular columns, computed columns, aggregate columns). This is used later for displaying results in the frontend (determining labels and types).

#### 24. `deriveAggregateAlias(array $g): string`
- Generates an alias for an aggregate function based on the function name and field (e.g., `count_uuid` or `sum_amount`).

#### 25. `validateQueryConfig()`
- Checks for the existence of table name and columns.
- Verifies that the table is registered in `$registry`.
- Validates each column (using `$registry->isColumnAllowed`).
- If there is a `groupBy`, ensures that non-aggregate columns are included in the `groupBy` set, otherwise throws an error.

#### 26. `extractRelationshipPathsFromExpression(string $expression, string $rootTable): array`
- Extracts relationship paths (e.g., `payload.pickup`) from a computed column expression by looking for column names containing dots, then removing the last part (the column) to obtain the path.
- Used in `processAutoJoins` to determine necessary joins for computed columns.

#### 27. `createJoinsForComputedColumn` (old, unused)
- An old function kept for compatibility, but it does nothing (No-op) because joins are now handled in `processAutoJoins`.

---

### Comprehensive Practical Examples

#### Example 1: Simple Report Without Aggregation
**Report Configuration:**
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'uuid', 'label' => 'Order ID'],
        ['name' => 'created_at', 'label' => 'Creation Date'],
        ['name' => 'customer.name', 'label' => 'Customer Name'],
    ],
    'conditions' => [
        [
            'field' => ['name' => 'status'],
            'operator' => ['value' => '='],
            'value' => 'completed'
        ]
    ],
    'sortBy' => [
        ['column' => ['name' => 'created_at'], 'direction' => ['value' => 'desc']]
    ],
    'limit' => 100
];
```

**Resulting Query:**
```sql
SELECT 
    orders.uuid as `uuid`,
    orders.created_at as `created_at`,
    customers.name as `customer_name`
FROM orders 
LEFT JOIN customers as orders_customer ON orders.customer_uuid = orders_customer.uuid
WHERE orders.status = 'completed' AND orders.company_uuid = 'abc-123'
ORDER BY orders.created_at DESC
LIMIT 100
```

**Result:** Array of 100 orders with customer name.

#### Example 2: Report with Group By and Computed Columns
**Report Configuration:**
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'customer.name', 'label' => 'Customer'],
    ],
    'computed_columns' => [
        ['name' => 'total_with_tax', 'expression' => 'ROUND(amount * 1.14, 2)', 'type' => 'decimal']
    ],
    'groupBy' => [
        [
            'groupBy' => ['name' => 'customer.name', 'label' => 'Customer'],
            'aggregateFn' => ['value' => 'count'],
            'aggregateBy' => ['name' => 'uuid']
        ],
        [
            'groupBy' => ['name' => 'customer.name'],
            'aggregateFn' => ['value' => 'sum'],
            'aggregateBy' => ['name' => 'total_with_tax']  // using computed column
        ]
    ],
    'sortBy' => [
        ['column' => ['alias' => 'count_uuid'], 'direction' => ['value' => 'desc']]
    ]
];
```

**Resulting Query:**
```sql
SELECT 
    customers.name as `customer_name`,
    COUNT(orders.uuid) as `count_uuid`,
    SUM(ROUND(orders.amount * 1.14, 2)) as `sum_total_with_tax`
FROM orders 
LEFT JOIN customers as orders_customer ON orders.customer_uuid = orders_customer.uuid
WHERE orders.company_uuid = 'abc-123'
GROUP BY customers.name
ORDER BY `count_uuid` DESC
```

**Result:** List of customers with order count per customer and total amounts after tax.

#### Example 3: Report with Nested Relationships
**Report Configuration:**
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'uuid'],
        ['name' => 'payload.pickup.address', 'label' => 'Pickup Address'],
        ['name' => 'payload.dropoff.city', 'label' => 'Dropoff City'],
    ],
    'conditions' => [
        [
            'field' => ['name' => 'payload.pickup.city'],
            'operator' => ['value' => '='],
            'value' => 'Cairo'
        ]
    ]
];
```

**Resulting Query:**
```sql
SELECT 
    orders.uuid as `uuid`,
    orders_payload_pickup.address as `payload_pickup_address`,
    orders_payload_dropoff.city as `payload_dropoff_city`
FROM orders 
LEFT JOIN order_payloads as orders_payload ON orders.uuid = orders_payload.order_uuid
LEFT JOIN addresses as orders_payload_pickup ON orders_payload.uuid = orders_payload_pickup.payload_uuid AND orders_payload_pickup.type = 'pickup'
LEFT JOIN addresses as orders_payload_dropoff ON orders_payload.uuid = orders_payload_dropoff.payload_uuid AND orders_payload_dropoff.type = 'dropoff'
WHERE orders_payload_pickup.city = 'Cairo' AND orders.company_uuid = 'abc-123'
```

**Result:** Orders where pickup city is Cairo, displaying pickup address and dropoff city.

---

### Error Handling
- `validateQueryConfig` throws `InvalidArgumentException` if the configuration is invalid.
- During query building, if any part fails (e.g., a missing relationship), the join may be skipped or an error thrown.
- In `execute`, any `Exception` is caught and a structured error response is returned with execution time.

### Important Notes
- Company scope is automatically applied to all queries to ensure security.
- Auto-joins are only applied to relationships with the `autoJoin = true` property.
- `resolveAliasAndColumn` is used to obtain the correct table alias, ensuring no name collisions when multiple joins are used.
- Computed columns are validated via `ComputedColumnValidator` to prevent injection and errors.

---

### Conclusion
The `ReportQueryConverter` class provides a robust and flexible solution for converting dynamic report configurations into executable SQL queries. It supports complex relationships, computed columns, aggregations, and numerous filter operators, while maintaining security and performance. This class is the backbone of the reporting system in applications like Fleetbase, where users need to build custom reports without needing to know SQL details.