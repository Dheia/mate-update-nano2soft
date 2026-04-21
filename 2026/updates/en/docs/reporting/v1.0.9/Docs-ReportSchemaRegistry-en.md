## Documentation of Dynamic Reporting System Classes in NanoSoft App - Nano2.QueryBuilder Package

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The set of classes `ReportSchemaRegistry`, `Table`, `Relationship`, and `Column` aims to build a **Schema Definition Layer** for dynamic reporting systems in NanoSoft App applications, as part of the **Nano2.QueryBuilder** package developed by **NanoSoft Company**. This layer allows registering tables available for reporting along with their columns and relationships, and provides a unified interface for querying this information, making it easier to build user interfaces and dynamic SQL queries in a secure and organized manner.

---

## 1. `Column` Class (Column Representation)

### Purpose
Represents a column in a database table or a computed column derived from an SQL expression. It contains all necessary properties to describe the column and its capabilities (searching, sorting, filtering, aggregation) in addition to a custom transformer function.

### Basic Properties
- **name**: Actual column name in the database (e.g., `created_at`).
- **label**: Display label for the user (e.g., "Creation Date").
- **type**: Data type (string, integer, decimal, date, datetime, boolean, ...).
- **description**: Optional description of the column.
- **format**: Optional display format (e.g., `Y-m-d` for date).
- **nullable**: Can the value be null?
- **searchable, sortable, filterable**: Column capabilities.
- **aggregatable**: Can it be used in aggregate functions (SUM, AVG)?
- **hidden**: Hide the column from the interface (remains available for querying).
- **computed**: Is it a computed column?
- **computation**: SQL expression for calculation (if computed).
- **transformer**: PHP function to transform the value before display.
- **meta**: Additional key-value data.

### Main Methods

#### Creating a Regular Column
```php
public static function make(string $name, string $type = 'string'): self
```
Example:
```php
$column = Column::make('status', 'string')
    ->label('Status')
    ->searchable(true)
    ->sortable(true)
    ->filterable(true);
```

#### Setting Properties
- `label(string $label)`
- `description(string $description)`
- `format(string $format)`
- `nullable(bool $nullable = true)`
- `searchable(bool $searchable = true)`
- `sortable(bool $sortable = true)`
- `filterable(bool $filterable = true)`
- `aggregatable(bool $aggregatable = true)`
- `hidden(bool $hidden = true)`
- `meta(string $key, $value)`
- `setMeta(array $meta)`

#### Creating Computed Columns
```php
public static function computed(string $name, string $computation, string $type = 'string', array $options = []): self
```
Example:
```php
$totalColumn = Column::computed('total_with_tax', 'ROUND(amount * 1.14, 2)', 'decimal')
    ->label('Total Including Tax')
    ->aggregatable(true);
```

#### Helper Methods for Aggregate Columns
- `Column::count(string $name, string $countField = '*')`: COUNT column.
- `Column::sum(string $name, string $sumField)`: SUM column.
- `Column::avg(string $name, string $avgField)`: AVG column.
- `Column::max(string $name, string $maxField)`: MAX column.
- `Column::min(string $name, string $minField)`: MIN column.

Example:
```php
$orderCount = Column::count('order_count', 'uuid')
    ->label('Number of Orders');
```

#### Transformer Methods
You can register a function to transform the column value before sending it to the interface:
```php
public function transformer(\Closure|callable $transformer): self
```
Example:
```php
$column->transformer(function ($value) {
    return $value ? 'Active' : 'Inactive';
});
```

#### Getters
- `getName()`, `getLabel()`, `getType()`, `getDescription()`, `getFormat()`
- `isNullable()`, `isSearchable()`, `isSortable()`, `isFilterable()`, `isAggregatable()`, `isHidden()`, `isComputed()`
- `getComputation()`, `getTransformer()`, `getMeta()`

#### Additional Methods
- `isForeignKey()`: Is the column a foreign key (ends with `_uuid` or `_id`)?
- `transformValue($value)`: Apply the transformer to a value.
- `toArray()`: Convert the column to an array (for sending via API).
- `copyWith(array $overrides)`: Create a modified copy of the column (useful for nested relationships).

### Complete Example Using Column
```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;

$columns = [
    Column::make('uuid', 'string')->label('ID')->hidden(true),
    Column::make('created_at', 'datetime')
        ->label('Creation Date')
        ->format('Y-m-d H:i')
        ->sortable(true),
    Column::make('status', 'string')
        ->label('Status')
        ->filterable(true)
        ->transformer(fn($v) => $v === 'completed' ? 'Completed' : 'Pending'),
    Column::computed('total', 'SUM(amount)', 'decimal')
        ->label('Total')
        ->aggregatable(true),
];
```

---

## 2. `Relationship` Class (Relationship Representation)

### Purpose
Represents a relationship between two tables, such as `belongsTo` or `hasMany`. It provides information about the related table, join keys, join type, and available columns from that relationship. Supports **nested relationships** and the **auto-join** feature, which automatically adds the join when columns from the relationship are selected.

### Basic Properties
- **name**: Relationship name (e.g., `customer`, `payload`).
- **table**: Name of the related table in the database.
- **label**: Display label for the relationship (e.g., "Customer").
- **type**: Join type (`left`, `right`, `inner`).
- **localKey**: Local key in the current table (default `{name}_uuid`).
- **foreignKey**: Foreign key in the related table (default `uuid`).
- **enabled**: Is the relationship enabled?
- **autoJoin**: Is it automatically added when columns from it are selected?
- **description**: Optional description.
- **columns**: Array of `Column` objects representing columns of the related table.
- **nestedRelationships**: Array of `Relationship` objects representing relationships inside the related table (e.g., `payload.pickup`).
- **meta**: Additional data.

### Main Methods

#### Creating a Relationship
```php
public static function make(string $name, string $table, string $type = 'left'): self
```
Example:
```php
$customerRel = Relationship::make('customer', 'customers')
    ->label('Customer')
    ->localKey('customer_uuid')
    ->foreignKey('uuid')
    ->joinType('left');
```

#### Relationship Helper Methods
- `Relationship::belongsTo(string $name, string $table)`: Alias for `make` with type = 'left'.
- `Relationship::hasMany(string $name, string $table)`: Alias.
- `Relationship::hasOne(string $name, string $table)`: Alias.
- `Relationship::hasAutoJoin(string $name, string $table, string $type = 'left')`: To create a relationship with auto-join enabled.

#### Setting Keys
```php
public function localKey(string $localKey): self
public function foreignKey(string $foreignKey): self
```

#### Adding Columns to the Relationship
```php
public function columns(array $columns): self
public function addColumn(Column $column): self
```
Example:
```php
$customerRel->columns([
    Column::make('name')->label('Name'),
    Column::make('email')->label('Email'),
]);
```

#### Adding Nested Relationships
```php
public function with(array $relationships): self
public function addNestedRelationship(Relationship $relationship): self
```
Example:
```php
$payloadRel = Relationship::hasAutoJoin('payload', 'order_payloads')
    ->columns(['weight', 'dimensions'])
    ->with([
        Relationship::hasAutoJoin('pickup', 'addresses')->columns(['address', 'city'])
    ]);
```

#### Enabling Auto-Join
```php
public function autoJoin(bool $autoJoin = true): self
// Or use hasAutoJoin at creation
```

#### Getters
- `getName()`, `getTable()`, `getLabel()`, `getType()`, `getLocalKey()`, `getForeignKey()`
- `isEnabled()`, `isAutoJoin()`
- `getColumns()`, `getNestedRelationships()`
- `getAllAvailableColumns()`: Returns all columns including those from nested relationships (with path prefix).
- `getAutoJoinRelationships()`: Returns nested relationships that have auto-join enabled.
- `hasNestedRelationships()`

#### Convert to Array
```php
public function toArray(): array
```

### Complete Example of Building a Relationship with Nesting
```php
$addressColumns = [
    Column::make('address')->label('Address'),
    Column::make('city')->label('City'),
    Column::make('country')->label('Country'),
];

$pickupRel = Relationship::hasAutoJoin('pickup', 'addresses')
    ->label('Pickup Location')
    ->columns($addressColumns);

$dropoffRel = Relationship::hasAutoJoin('dropoff', 'addresses')
    ->label('Dropoff Location')
    ->columns($addressColumns);

$payloadRel = Relationship::hasAutoJoin('payload', 'order_payloads')
    ->label('Payload Data')
    ->columns([
        Column::make('weight')->label('Weight')->type('decimal'),
        Column::make('dimensions')->label('Dimensions'),
    ])
    ->with([$pickupRel, $dropoffRel]);

$orderRel = Relationship::belongsTo('customer', 'customers')
    ->columns([
        Column::make('name')->label('Name'),
        Column::make('email')->label('Email'),
    ]);
```

---

## 3. `Table` Class (Table Representation)

### Purpose
Represents a main table in the database available for reporting. It combines regular columns, computed columns, and relationships with other tables. Provides methods for accessing visible columns and validating their permissions.

### Basic Properties
- **name**: Table name in the database.
- **label**: Display label for the table.
- **description**: Optional description.
- **category**: Table category (e.g., `sales`, `inventory`).
- **extension**: Extension to which the table belongs (e.g., `core`, `fleet`).
- **columns**: Array of `Column` for regular columns.
- **computedColumns**: Array of `Column` for computed columns (appear only when used in aggregations or as computed fields).
- **relationships**: Array of `Relationship`.
- **excludedColumns**: List of column names excluded from display (remain available for querying).
- **supportsAggregates**: Does it support aggregate functions?
- **maxRows**: Maximum allowed rows.
- **cacheable**: Can query results be cached?
- **cacheTtl**: Cache duration.
- **permissions**: Access permissions for the table.
- **meta**: Additional data.

### Main Methods

#### Creating a Table
```php
public static function make(string $name): self
```
Example:
```php
$ordersTable = Table::make('orders')
    ->label('Orders')
    ->category('sales')
    ->extension('core');
```

#### Adding Columns
```php
public function columns(array $columns): self
public function addColumn(Column $column): self
```
You can pass an array of `Column` objects or string column names (automatically converted to Column objects with type string).

Example:
```php
$ordersTable->columns([
    'uuid',   // becomes Column::make('uuid')
    'created_at',
    Column::make('amount', 'decimal')->label('Amount'),
]);
```

#### Adding Computed Columns
```php
public function computedColumns(array $columns): self
public function addComputedColumn(Column $column): self
```

#### Adding Relationships
```php
public function relationships(array $relationships): self
public function addRelationship(Relationship $relationship): self
```
Example:
```php
$ordersTable->relationships([
    $customerRel,
    $payloadRel,
]);
```

#### Excluding Columns from Display
```php
public function excludeColumns(array $columnNames): self
```
Example:
```php
$ordersTable->excludeColumns(['deleted_at', 'internal_note']);
```

#### Setting General Properties
- `supportsAggregates(bool $supports = true)`
- `maxRows(int $maxRows)`
- `cacheable(bool $cacheable = true)`
- `cacheTtl(int $ttl)`
- `permissions(array $permissions)`
- `meta(string $key, $value)`

#### Getters
- `getName()`, `getLabel()`, `getDescription()`, `getCategory()`, `getExtension()`
- `getColumns()`, `getComputedColumns()`, `getAllColumns()`
- `getRelationships()`, `getAutoJoinRelationships()`, `getManualJoinRelationships()`
- `getExcludedColumns()`
- `getSupportsAggregates()`, `getMaxRows()`, `isCacheable()`, `getCacheTtl()`
- `getPermissions()`, `getMeta()`

#### Methods for Checking Columns and Relationships
- `getVisibleColumns()`: Displayable columns (not hidden, not excluded, and not foreign keys).
- `getAllAvailableColumns()`: All selectable columns, including those from auto-join relationships (with paths).
- `getRelationship(string $name)`: Get a relationship by name.
- `getColumn(string $name)`: Get a column by name.
- `hasColumn(string $name)`
- `hasRelationship(string $name)`
- `isColumnAllowed(string $name)`: Check if a column is allowed (exists, not hidden, not excluded).

#### Convert to Array
```php
public function toArray(): array
```

### Complete Example of Building a Table
```php
// Create columns
$columns = [
    Column::make('uuid')->label('ID')->hidden(true),
    Column::make('created_at', 'datetime')->label('Creation Date')->sortable(true),
    Column::make('status', 'string')->label('Status')->filterable(true),
    Column::make('amount', 'decimal')->label('Amount')->aggregatable(true),
];

// Create relationships
$customerRel = Relationship::hasAutoJoin('customer', 'customers')
    ->columns([
        Column::make('name')->label('Name'),
        Column::make('email')->label('Email'),
    ]);

$payloadRel = Relationship::hasAutoJoin('payload', 'order_payloads')
    ->columns(['weight', 'dimensions'])
    ->with([
        Relationship::hasAutoJoin('pickup', 'addresses')
            ->columns(['address', 'city'])
            ->label('Pickup Location')
    ]);

// Create the table
$ordersTable = Table::make('orders')
    ->label('Orders')
    ->description('Main orders table')
    ->category('sales')
    ->extension('core')
    ->columns($columns)
    ->relationships([$customerRel, $payloadRel])
    ->supportsAggregates(true)
    ->maxRows(20000)
    ->cacheable(true)
    ->cacheTtl(1800); // 30 minutes
```

---

## 4. `ReportSchemaRegistry` Class (Schema Registry)

### Purpose
Represents a central registry for all tables registered in the system. Provides an interface for registering tables and retrieving column and relationship information, with cache support for performance.

### Basic Properties
- **$tables**: Array of `Table` objects indexed by table names.
- **$cacheEnabled**: Is caching enabled?
- **$cacheTtl**: Default cache time-to-live (seconds).

### Main Methods

#### Registering Tables
```php
public function registerTable(Table $table): void
public function registerTables(array $tables): void
```
Example:
```php
$registry = new ReportSchemaRegistry();
$registry->registerTable($ordersTable);
$registry->registerTable($usersTable);
```

#### Getting a Table
```php
public function getTable(string $name): ?Table
public function isTableRegistered(string $name): bool
public function hasTable(string $name): bool // alias
public function getRegisteredTableNames(): array
```

#### Getting Tables Available for the Interface
```php
public function getAvailableTables(string $extension, ?string $category = null): array
```
Returns an array of tables with their information (name, label, description, category, columns, relationships). Supports caching.

Example:
```php
$tables = $registry->getAvailableTables('core', 'sales');
```

#### Getting Table Columns
```php
public function getTableColumns(string $tableName): array
```
Returns an array of selectable columns (with auto-join relationship expansion). The result is cached.

Example:
```php
$columns = $registry->getTableColumns('orders');
// Result contains columns like: ['uuid', 'created_at', 'customer.name', 'payload.pickup.address', ...]
```

#### Getting Table Relationships
```php
public function getTableRelationships(string $tableName): array
```
Returns an array of relationships (including nested ones).

#### Getting Auto-Join Columns (Flattened)
```php
public function getAutoJoinColumns(string $tableName): array
```
Returns columns from auto-join relationships in a flattened manner.

#### Checking Column Validity
```php
public function isColumnAllowed(string $tableName, string $columnPath): bool
```
Checks if the specified column (with a path like `payload.pickup.city`) exists and is allowed. Used for validation before building the query.

Example:
```php
if ($registry->isColumnAllowed('orders', 'payload.pickup.city')) {
    // Column is valid
}
```

#### Cache Management Methods
```php
public function clearTableCache(string $tableName): void
public function clearAllCache(): void
public function setCacheEnabled(bool $enabled): void
public function setCacheTtl(int $ttl): void
```

### Practical Examples of Using the Registry

#### Registering Multiple Tables
```php
$registry = new ReportSchemaRegistry();

$ordersTable = Table::make('orders')...;
$usersTable = Table::make('users')...;
$registry->registerTables([$ordersTable, $usersTable]);
```

#### Using in a Controller to Fetch Table Columns
```php
class ReportController extends Controller
{
    public function getTableColumns(Request $request)
    {
        $tableName = $request->input('table');
        $registry = ReportSchemaRegistry::instance(); // or via dependency injection
        
        if (!$registry->hasTable($tableName)) {
            return response()->json(['error' => 'Table not found'], 404);
        }
        
        $columns = $registry->getTableColumns($tableName);
        return response()->json($columns);
    }
}
```

#### Validating a Column Before Adding It to a Query
```php
function validateColumn($tableName, $columnPath, ReportSchemaRegistry $registry) {
    if (!$registry->isColumnAllowed($tableName, $columnPath)) {
        throw new \Exception("Column {$columnPath} not allowed");
    }
}
```

#### Getting All Available Tables for a Specific Extension
```php
$coreTables = $registry->getAvailableTables('core');
$fleetTables = $registry->getAvailableTables('fleet');
```

---

## Summary of Relationships Between Classes

- **Column** is the basic unit for describing a field.
- **Relationship** builds on Column to add columns from another table, and can contain other **Relationship** objects (nesting).
- **Table** combines Columns and Relationships to describe a complete table.
- **ReportSchemaRegistry** manages a collection of Tables and provides a query interface with caching.

This design allows building a dynamic and scalable reporting schema, usable in advanced NanoSoft systems where multi-level relationships exist (e.g., `orders.payload.pickup.address`).

### How the Classes Work Together (Typical Scenario)

1. **Definition:** The developer (or extension system) creates `Column`, `Relationship`, and `Table` objects to describe the data structure.
2. **Registration:** `Table` objects are registered in the `ReportSchemaRegistry` once (usually in a Service Provider).
3. **Querying:** When the frontend needs to display available columns for a new report, it calls `$registry->getTableColumns('orders')`, which in turn collects regular columns and relationship columns (`getAllAvailableColumns`) and returns them as a flat list with their paths (e.g., `customer.name`).
4. **Validation:** Before executing a custom report query, `$registry->isColumnAllowed('orders', 'payload.pickup.city')` is used to ensure each column in the report configuration is valid and allowed.
5. **Query Building:** `ReportQueryConverter` (another class in the same package) uses the same `$registry` to convert column paths into an SQL query with appropriate joins, relying on the stored relationship information.

### Benefits of This Design

- **Separation of definition and usage:** Data structure is explicitly defined in separate objects, making the system maintainable and extensible.
- **Reusability:** Relationship definitions can be reused across multiple tables.
- **Safety and security:** `ReportSchemaRegistry` provides a central validation point to ensure that columns requested by the user exist and are allowed, preventing unauthorized access attempts or invalid SQL queries.
- **Performance:** Caching the outputs of `getTableColumns` and `getAvailableTables` reduces repeated database queries and speeds up API responses.
- **Flexibility:** Support for nested relationships and dot‑notation paths (e.g., `payload.pickup.address`) allows building complex reports that reach multiple data depths.

### Best Practices When Using These Classes

1. **Consistent naming:** Use consistent names for relationships and columns to avoid confusion (e.g., using `_uuid` as a suffix for foreign keys).
2. **Use `autoJoin` carefully:** Enable `autoJoin` only for relationships whose columns are frequently needed in reports, to avoid fetching unnecessary data.
3. **Leverage `excludeColumns`:** Exclude sensitive or technical columns (e.g., `password`, `remember_token`) from public display.
4. **Register tables in a timely manner:** Register tables in the application's or extensions' Service Providers to ensure availability before any request.
5. **Use `copyWith` in nested relationships:** When creating deep nested relationships, you may need to modify a copy of a column (e.g., change its label) using `copyWith` to preserve context.

## Conclusion

These four classes (`Column`, `Relationship`, `Table`, `ReportSchemaRegistry`) form the backbone of the reporting schema layer in the **Nano2.QueryBuilder** package from **NanoSoft Company**. Together, they provide an organized and powerful way to describe any complex database structure, with easy extensibility by new extensions. By separating definition from execution and providing validation and caching mechanisms, this design ensures dynamic reports are built safely and efficiently, meeting advanced user needs without compromising system performance or integrity.