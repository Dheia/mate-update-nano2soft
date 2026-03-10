## Documentation of `ReportSchemaRegistry` Class

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The `ReportSchemaRegistry` class is the **central registry** for the `Nano2.QueryBuilder.Reporting` dynamic reporting system. This class manages all `Table` objects registered in the system and provides a unified interface for querying tables, their columns, and relationships. It also supports caching for frequent queries to improve performance and allows developers to register new tables from multiple sources (such as different extensions).

Simply put, `ReportSchemaRegistry` is **the place where all table definitions live**, and it is the means by which other system components (like `ReportQueryValidator` and `ReportQueryConverter`) access schema information.

---

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$tables` | `array` | An array storing `Table` objects, keyed by table name. |
| `$cacheEnabled` | `bool` | Flag enabling or disabling caching (default `true`). |
| `$cacheTtl` | `int` | Default cache time-to-live in seconds (default `3600` = one hour). |

---

### Main Methods

#### 1. Registering Tables

##### `public function registerTable(Table $table): void`
- **Purpose**: Register a single `Table` object in the registry.
- **Parameters**: `$table` – the `Table` object to register.
- **Behavior**: Adds the table to `$tables` using `$table->getName()` as the key. Then calls `clearTableCache()` to clear any cached data for this table (to ensure data freshness).
- **Example**:
  ```php
  $registry->registerTable($ordersTable);
  ```

##### `public function registerTables(array $tables): void`
- **Purpose**: Register multiple tables at once.
- **Parameters**: `$tables` – an array of `Table` objects.
- **Behavior**: Iterates over the array and calls `registerTable()` for each element.
- **Example**:
  ```php
  $registry->registerTables([$ordersTable, $usersTable, $productsTable]);
  ```

#### 2. Getting Tables

##### `public function getTable(string $name): ?Table`
- **Purpose**: Get a `Table` object by name.
- **Parameters**: `$name` – the table name.
- **Returns**: `Table` object if found, otherwise `null`.
- **Example**:
  ```php
  $orders = $registry->getTable('orders');
  ```

##### `public function isTableRegistered(string $name): bool`
- **Purpose**: Check if a table is registered.
- **Parameters**: `$name` – the table name.
- **Returns**: `true` if exists, otherwise `false`.

##### `public function hasTable(string $name): bool`
- **Purpose**: Alias for `isTableRegistered()` (for convenience).

##### `public function getRegisteredTableNames(): array`
- **Purpose**: Get a list of all registered table names.

#### 3. Getting Table Information for the Frontend

##### `public function getAvailableTables(string $extension, ?string $category = null): array`
- **Purpose**: Return a list of tables available for the frontend, filtered by extension and category.
- **Parameters**:
  - `$extension`: The extension name (e.g., `core`, `fleet`).
  - `$category`: Table category (e.g., `sales`, `inventory`) – optional.
- **Behavior**:
  - Searches for tables matching `$extension` (and `$category` if provided).
  - For each matching table, builds an array containing: `name`, `label`, `description`, `category`, `extension`, `columns` (from `getTableColumns()`), `relationships` (from `getTableRelationships()`), `auto_join_columns` (from `getAutoJoinColumns()`), `supports_aggregates`, `max_rows`, `has_auto_joins`, `has_manual_joins`.
  - Uses caching (`Cache::get` / `Cache::put`) if `$cacheEnabled = true`, with a key like `report_tables_{extension}_{category}`.
- **Returns**: An array of tables ready to be sent as JSON.
- **Example**:
  ```php
  $coreSalesTables = $registry->getAvailableTables('core', 'sales');
  ```

#### 4. Getting Columns of a Specific Table

##### `public function getTableColumns(string $tableName): array`
- **Purpose**: Return all selectable columns from a given table (including columns from auto‑join relationships).
- **Parameters**: `$tableName` – the table name.
- **Behavior**:
  - Attempts to read from cache using the key `report_columns_{$tableName}`.
  - If not cached, obtains the `Table` object and calls `$table->getAllAvailableColumns()` to get the columns (with relationships expanded).
  - Converts each column to an array using `$column->toArray()`.
  - Stores the result in cache if caching is enabled.
- **Returns**: An array of columns (each column as an array).
- **Example**:
  ```php
  $columns = $registry->getTableColumns('orders');
  // Result: [['name' => 'uuid', 'label' => 'ID', ...], ['name' => 'customer.name', 'label' => 'Customer - Name', ...], ...]
  ```

#### 5. Getting Relationships of a Specific Table

##### `public function getTableRelationships(string $tableName): array`
- **Purpose**: Return all available relationships for a given table (for frontend use or validation).
- **Parameters**: `$tableName` – the table name.
- **Behavior**:
  - Attempts to read from cache using the key `report_relationships_{$tableName}`.
  - Obtains the `Table` object and calls `$table->getRelationships()`.
  - Converts each relationship to an array using `$relationship->toArray()`, and adds the property `available_for_manual_join` (always `true` currently).
  - Stores in cache.
- **Returns**: An array of relationships.

#### 6. Getting Auto‑Join Columns (Flattened)

##### `public function getAutoJoinColumns(string $tableName): array`
- **Purpose**: Return columns from auto‑join relationships in a flattened manner (useful for some special cases).
- **Parameters**: `$tableName` – the table name.
- **Behavior**:
  - Obtains the `Table` object and calls `$table->getAutoJoinRelationships()`.
  - For each relationship, calls `$relationship->getAllAvailableColumns()` and collects the columns.
  - Converts each column to an array, adding `auto_join_path`, `relationship`, and `column`.
- **Returns**: An array of columns (not fully nested, but with path information).

#### 7. Checking Column Validity

##### `public function isColumnAllowed(string $tableName, string $columnPath): bool`
- **Purpose**: Check whether a column with a given path (e.g., `payload.pickup.city`) exists and is allowed.
- **Parameters**:
  - `$tableName`: The base table name.
  - `$columnPath`: The column path (may contain dots).
- **Behavior**:
  - If the path contains no dots, directly checks the table using `$table->isColumnAllowed($columnPath)`.
  - If it contains dots, splits the path into segments (e.g., `['payload', 'pickup', 'city']`).
  - Obtains the `Table` object for the base table.
  - Traverses the segments (except the last) to find the chain of relationships:
    - The first segment looks for a relationship in the base table.
    - Subsequent segments look for nested relationships in the previous relationship.
    - If any segment fails or the relationship is not auto‑join, returns `false`.
  - After reaching the last relationship, checks whether the final column exists in that relationship's columns.
- **Returns**: `true` if the column is valid, otherwise `false`.
- **Example**:
  ```php
  if ($registry->isColumnAllowed('orders', 'payload.pickup.city')) {
      // The column is valid for use
  }
  ```

#### 8. Cache Management

##### `public function clearTableCache(string $tableName): void`
- **Purpose**: Clear all cache keys associated with a specific table.
- **Parameters**: `$tableName` – the table name.
- **Behavior**:
  - Obtains the `Table` object to extract `extension` and `category`.
  - Builds a list of keys to clear:
    - `report_columns_{$tableName}`
    - `report_relationships_{$tableName}`
    - `report_tables_{$extension}_all`
    - `report_tables_{$extension}_{$category}` (if exists)
  - Calls `Cache::forget()` for each key.
- **Example**:
  ```php
  $registry->clearTableCache('orders'); // after updating the table definition
  ```

##### `public function clearAllCache(): void`
- **Purpose**: Clear all cache keys for all registered tables.
- **Behavior**: Iterates over all tables and calls `clearTableCache()` for each.

##### `public function setCacheEnabled(bool $enabled): void`
- **Purpose**: Enable or disable caching.

##### `public function setCacheTtl(int $ttl): void`
- **Purpose**: Set the default cache time‑to‑live.

#### 9. Internal Helper Methods

##### `private function flattenRelationshipColumns($relationship, string $path, array $labelTrail): array`
- Used internally in `getTableColumns` to expand nested relationship columns and create paths like `payload.pickup.address`.
- Modifies the column name (`{$path}.{$column->getName()}`) and its label using `$labelTrail` to create a contextual label (e.g., "Pickup Address").

##### `private function shortRelationshipLabel(string $label): string`
- Used to generate a short prefix for the label (e.g., "Pickup" from "Pickup Location").

##### `private function getChildRelationship($parentRelationship, string $name)`
- Used in `isColumnAllowed` to search for a nested relationship inside a parent relationship.

---

### Comprehensive Practical Examples

#### Example 1: Registering Multiple Tables
```php
use Nano2\QueryBuilder\Classes\Reporting\ReportSchemaRegistry;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Table;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Relationship;

// Create registry
$registry = new ReportSchemaRegistry();

// Create orders table
$ordersTable = Table::make('orders')
    ->label('Orders')
    ->category('sales')
    ->extension('core')
    ->columns([
        Column::make('uuid')->hidden(true),
        Column::make('created_at', 'datetime')->label('Creation Date'),
        Column::make('amount', 'decimal')->label('Amount'),
    ]);

// Create users table
$usersTable = Table::make('users')
    ->label('Users')
    ->category('users')
    ->extension('core')
    ->columns([
        Column::make('uuid')->hidden(true),
        Column::make('name')->label('Name'),
        Column::make('email')->label('Email'),
    ]);

// Register tables
$registry->registerTables([$ordersTable, $usersTable]);
```

#### Example 2: Getting Columns of a Specific Table (for the Frontend)
```php
// In a Controller
public function getTableColumns(Request $request, ReportSchemaRegistry $registry)
{
    $tableName = $request->input('table');
    
    if (!$registry->hasTable($tableName)) {
        return response()->json(['error' => 'Table not found'], 404);
    }
    
    $columns = $registry->getTableColumns($tableName);
    return response()->json($columns);
}
```

#### Example 3: Getting Available Tables for a Specific Extension
```php
$coreTables = $registry->getAvailableTables('core');
$fleetTables = $registry->getAvailableTables('fleet');

// With a specific category
$salesTables = $registry->getAvailableTables('core', 'sales');
```

#### Example 4: Checking Column Validity (Before Building a Query)
```php
function validateColumn($tableName, $columnPath, ReportSchemaRegistry $registry) {
    if (!$registry->isColumnAllowed($tableName, $columnPath)) {
        throw new \Exception("Column {$columnPath} is not allowed");
    }
}

// Example:
validateColumn('orders', 'payload.pickup.city', $registry); // passes if path is correct
validateColumn('orders', 'non_existent', $registry); // throws an error
```

#### Example 5: Using isColumnAllowed with Different Paths
```php
// Direct path
$registry->isColumnAllowed('orders', 'amount'); // true (if exists)

// Single‑relationship path
$registry->isColumnAllowed('orders', 'customer.name'); // true (if customer relationship exists and has name column)

// Nested two‑relationship path
$registry->isColumnAllowed('orders', 'payload.pickup.city'); // true (if paths are correct)

// Non‑existent path
$registry->isColumnAllowed('orders', 'payload.invalid.city'); // false
```

#### Example 6: Cache Management
```php
// Temporarily disable caching (e.g., during development)
$registry->setCacheEnabled(false);

// Change TTL to 30 minutes
$registry->setCacheTtl(1800);

// After modifying a table definition, clear its cache
$registry->clearTableCache('orders');

// Or clear everything
$registry->clearAllCache();
```

#### Example 7: Using the Registry in ReportQueryValidator
```php
class ReportQueryValidator
{
    protected ReportSchemaRegistry $registry;
    
    public function __construct(ReportSchemaRegistry $registry)
    {
        $this->registry = $registry;
    }
    
    public function validate(array $queryConfig)
    {
        // Check if table exists
        if (!$this->registry->hasTable($queryConfig['table']['name'])) {
            $this->errors[] = "Table not found";
        }
        
        // Check columns
        foreach ($queryConfig['columns'] as $column) {
            if (!$this->registry->isColumnAllowed($queryConfig['table']['name'], $column['name'])) {
                $this->errors[] = "Column {$column['name']} is not allowed";
            }
        }
        
        // ... rest of validation
    }
}
```

#### Example 8: Registering Tables from an Extension Using the ReportSchema Interface
```php
// Class implementing ReportSchema interface
class CoreReportSchema implements \Nano2\QueryBuilder\Classes\Reporting\Contracts\ReportSchema
{
    public function registerReportSchema(ReportSchemaRegistry $registry): void
    {
        $ordersTable = Table::make('orders')...;
        $usersTable = Table::make('users')...;
        
        $registry->registerTables([$ordersTable, $usersTable]);
    }
}

// In Service Provider
public function boot()
{
    $registry = $this->app->make(ReportSchemaRegistry::class);
    
    // Call all classes tagged with 'report-schemas'
    foreach ($this->app->tagged('report-schemas') as $schema) {
        $schema->registerReportSchema($registry);
    }
}
```

#### Example 9: Getting All Registered Table Names
```php
$tableNames = $registry->getRegisteredTableNames();
// ['orders', 'users', 'products', ...]
```

---

### Interaction with Other Classes

- **With `Table`**: `ReportSchemaRegistry` manages a collection of `Table` objects. It uses `Table` methods such as `getName()`, `getAllAvailableColumns()`, `getRelationships()`, `isColumnAllowed()`, `getAutoJoinRelationships()`.
- **With `Relationship`**: When expanding columns in `getTableColumns()`, it calls `Relationship` methods like `getAllAvailableColumns()`, `getName()`, and `getLabel()`.
- **With `Column`**: Converts columns to arrays using `toArray()`, and internally uses `copyWith()` in `flattenRelationshipColumns()`.
- **With `ReportQueryValidator` and `ReportQueryConverter`**: Both depend on `ReportSchemaRegistry` to access schema information and validate data.

---

### Best Practices

1. **Register tables in a Service Provider**: Register all tables once when the application starts (in `boot()` or `register()`).
2. **Use the `ReportSchema` interface**: To standardise how tables are registered from different extensions.
3. **Enable caching in production**: With an appropriate TTL (e.g., 3600 seconds) to reduce database load.
4. **Clear the cache after updating any table definition**: To avoid serving stale data.
5. **Use `isColumnAllowed` for pre‑query validation**: This prevents errors and improves security.
6. **Organise tables using `category` and `extension`**: To simplify filtering in the frontend.

---

### Summary
`ReportSchemaRegistry` is the heart of the dynamic reporting system. It centralises all the system's knowledge about available tables and their relationships, and provides a clean and efficient programmatic interface to access this information. With its support for caching and column validity checking, developers can build powerful and secure reporting systems without worrying about performance or data correctness. This class integrates closely with `Table`, `Relationship`, and `Column` to form a complete definition layer that empowers end‑users to create complex reports with ease.