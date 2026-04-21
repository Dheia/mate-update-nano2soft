## Documentation of `Table` Class

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The `Table` class is one of the fundamental classes in the Schema Definition Layer of the `Nano2.QueryBuilder.Reporting` dynamic reporting system. This class represents a **database table** available for querying through the reporting system. It aggregates all information related to the table: its regular columns, computed columns, relationships with other tables, as well as descriptive properties such as category, extension, and permissions.

`Table` acts as a comprehensive container providing a unified interface to access all reportable data from this table, whether directly or through relationships.

---

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$name` | `string` | Actual table name in the database (e.g., `orders`). |
| `$label` | `string` | Display label for the table (e.g., "Orders") used in the user interface. |
| `$description` | `?string` | Optional description of the table clarifying its content or usage. |
| `$category` | `?string` | Table category (e.g., `sales`, `inventory`, `hr`) for organizing tables into groups. |
| `$extension` | `?string` | Name of the extension to which the table belongs (e.g., `core`, `fleet`). |
| `$columns` | `array` | Array of `Column` objects representing regular columns in the table. |
| `$computedColumns` | `array` | Array of `Column` objects representing computed columns (derived from an SQL expression). |
| `$relationships` | `array` | Array of `Relationship` objects representing relationships with other tables. |
| `$excludedColumns` | `array` | List of column names excluded from display in the user interface (remain available for querying). |
| `$supportsAggregates` | `bool` | Does the table support aggregate functions (e.g., COUNT, SUM, AVG)? |
| `$maxRows` | `?int` | Maximum number of rows allowed to be returned in any query on this table. |
| `$cacheable` | `bool` | Can the results of queries on this table be cached? |
| `$cacheTtl` | `int` | Cache duration (in seconds) if `$cacheable = true`. |
| `$permissions` | `array` | Array defining access permissions for the table (can be used later). |
| `$meta` | `array` | Additional key-value data (useful for extensions). |

---

### Main Methods

#### 1. Creating a Table Object

##### `public static function make(string $name): self`
- **Purpose**: Create a new `Table` object with the specified name.
- **Parameters**: `$name` – the table name in the database.
- **Returns**: New `Table` object.
- **Example**:
  ```php
  $ordersTable = Table::make('orders');
  ```

##### `public function label(string $label): self`
- **Purpose**: Set the display label for the table.
- **Example**:
  ```php
  $ordersTable->label('Orders');
  ```

##### `public function description(string $description): self`
- **Purpose**: Set a description for the table.
- **Example**:
  ```php
  $ordersTable->description('Main orders table');
  ```

##### `public function category(string $category): self`
- **Purpose**: Set the table category.
- **Example**:
  ```php
  $ordersTable->category('sales');
  ```

##### `public function extension(string $extension): self`
- **Purpose**: Set the extension providing the table.
- **Example**:
  ```php
  $ordersTable->extension('core');
  ```

#### 2. Adding Columns

##### `public function columns(array $columns): self`
- **Purpose**: Add a set of columns to the table. Can accept:
  - Ready-made `Column` objects.
  - Arrays containing `name` and `type` (will be converted to `Column`).
  - String column names (will be converted to `Column` with type `string`).
- **Example**:
  ```php
  $ordersTable->columns([
      'uuid',
      'created_at',
      ['name' => 'amount', 'type' => 'decimal'],
      Column::make('status', 'string')->label('Status'),
  ]);
  ```

##### `public function addColumn(Column $column): self`
- **Purpose**: Add a single column to the table.
- **Example**:
  ```php
  $ordersTable->addColumn(Column::make('notes')->label('Notes'));
  ```

##### `public function computedColumns(array $columns): self`
- **Purpose**: Add a set of computed columns (must be `Column` objects with `computed = true`).
- **Example**:
  ```php
  $ordersTable->computedColumns([
      Column::computed('total_with_tax', 'ROUND(amount * 1.14, 2)', 'decimal'),
  ]);
  ```

##### `public function addComputedColumn(Column $column): self`
- **Purpose**: Add a single computed column.

#### 3. Adding Relationships

##### `public function relationships(array $relationships): self`
- **Purpose**: Add a set of relationships (`Relationship` objects).
- **Example**:
  ```php
  $ordersTable->relationships([
      $customerRel,
      $payloadRel,
  ]);
  ```

##### `public function addRelationship(Relationship $relationship): self`
- **Purpose**: Add a single relationship.

#### 4. Managing Excluded Columns

##### `public function excludeColumns(array $columnNames): self`
- **Purpose**: Exclude specific columns from appearing in the user interface (they remain available for querying).
- **Example**:
  ```php
  $ordersTable->excludeColumns(['deleted_at', 'internal_notes']);
  ```

#### 5. Setting General Properties

##### `public function supportsAggregates(bool $supports = true): self`
- **Purpose**: Specify whether the table supports aggregate functions.

##### `public function maxRows(int $maxRows): self`
- **Purpose**: Set the maximum number of allowed rows.

##### `public function cacheable(bool $cacheable = true): self`
- **Purpose**: Enable/disable caching.

##### `public function cacheTtl(int $ttl): self`
- **Purpose**: Set the cache duration.

##### `public function permissions(array $permissions): self`
- **Purpose**: Set access permissions for the table.

##### `public function meta(string $key, $value): self`
- **Purpose**: Add metadata.

#### 6. Getters

##### `public function getName(): string`
##### `public function getLabel(): string`
##### `public function getDescription(): ?string`
##### `public function getCategory(): ?string`
##### `public function getExtension(): ?string`
##### `public function getColumns(): array`
- **Returns**: Array of regular columns.

##### `public function getComputedColumns(): array`
- **Returns**: Array of computed columns.

##### `public function getAllColumns(): array`
- **Returns**: Merged array of regular and computed columns.

##### `public function getRelationships(): array`
##### `public function getExcludedColumns(): array`
##### `public function getSupportsAggregates(): bool`
##### `public function getMaxRows(): ?int`
##### `public function isCacheable(): bool`
##### `public function getCacheTtl(): int`
##### `public function getPermissions(): array`
##### `public function getMeta(?string $key = null)`

#### 7. Validation and Advanced Query Methods

##### `public function getVisibleColumns(): array`
- **Purpose**: Returns columns displayable in the user interface. It excludes:
  - Hidden columns (`hidden = true`).
  - Columns present in `$excludedColumns`.
  - Columns that are foreign keys (ending with `_uuid` or `_id`).

##### `public function getAutoJoinRelationships(): array`
- **Purpose**: Returns only relationships with the `autoJoin = true` property.

##### `public function getManualJoinRelationships(): array`
- **Purpose**: Returns relationships without the `autoJoin` property.

##### `public function getRelationship(string $name): ?Relationship`
- **Purpose**: Find a relationship by name.

##### `public function getColumn(string $name): ?Column`
- **Purpose**: Find a column by name (searches in regular and computed columns).

##### `public function hasColumn(string $name): bool`
- **Purpose**: Check if a column with the given name exists.

##### `public function hasRelationship(string $name): bool`
- **Purpose**: Check if a relationship with the given name exists.

##### `public function isColumnAllowed(string $name): bool`
- **Purpose**: Check whether a column is allowed (i.e., exists and is not hidden or excluded).
- **Example**:
  ```php
  if ($table->isColumnAllowed('amount')) {
      // The column can be used
  }
  ```

##### `public function getAllAvailableColumns(): array`
- **Purpose**: Returns all selectable columns, including columns from auto-join relationships. This is done by:
  - Getting visible columns (`getVisibleColumns`).
  - For each auto-join relationship, fetching all its columns (including nested ones) using `getAllAvailableColumns` from `Relationship`, and adding `auto_join_path` to the meta.
- This is the primary source of columns that will appear to the user in the report builder interface.

##### `public function toArray(): array`
- **Purpose**: Convert the table to a serializable array (e.g., for sending via API). Contains all basic information.

##### `protected function isForeignKeyColumn(string $name): bool`
- **Purpose**: Check if a column name indicates a foreign key (ends with `_uuid` or `_id`).

##### `protected function generateLabel(string $name): string`
- **Purpose**: Generate a default label from the table name (convert underscores and dashes to spaces, then capitalize first letter).

---

### Comprehensive Practical Examples

#### Example 1: Creating a Complete Table (with columns and relationships)
Suppose we are building an `orders` table with columns and relationships `customer` and `payload` that contain `pickup` and `dropoff`.

```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Relationship;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Table;

// 1. Create regular columns
$columns = [
    Column::make('uuid')->label('ID')->hidden(true),
    Column::make('created_at', 'datetime')->label('Creation Date')->sortable(true),
    Column::make('amount', 'decimal')->label('Amount')->aggregatable(true),
    Column::make('status', 'string')->label('Status')->filterable(true),
];

// 2. Create computed columns
$computedColumns = [
    Column::computed('total_with_tax', 'ROUND(amount * 1.14, 2)', 'decimal')
        ->label('Total Including Tax'),
];

// 3. Create customer relationship (auto-join)
$customerRel = Relationship::hasAutoJoin('customer', 'customers')
    ->label('Customer')
    ->localKey('customer_uuid')
    ->foreignKey('uuid')
    ->columns([
        Column::make('name')->label('Name'),
        Column::make('email')->label('Email'),
    ]);

// 4. Create payload relationship (auto-join) with nested relationships
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
        Column::make('weight', 'decimal')->label('Weight'),
        Column::make('dimensions')->label('Dimensions'),
    ])
    ->with([$pickupRel, $dropoffRel]);

// 5. Create the table
$ordersTable = Table::make('orders')
    ->label('Orders')
    ->description('Main orders table')
    ->category('sales')
    ->extension('core')
    ->columns($columns)
    ->computedColumns($computedColumns)
    ->relationships([$customerRel, $payloadRel])
    ->excludeColumns(['deleted_at', 'internal_note'])
    ->supportsAggregates(true)
    ->maxRows(50000)
    ->cacheable(true)
    ->cacheTtl(3600); // one hour
```

#### Example 2: Using Getters
```php
// Get all visible columns (excluding foreign keys)
$visibleColumns = $ordersTable->getVisibleColumns();
foreach ($visibleColumns as $column) {
    echo $column->getName() . ' - ' . $column->getLabel() . PHP_EOL;
}
// Output: uuid - ID, created_at - Creation Date, amount - Amount, status - Status

// Get all available columns (including relationship columns)
$allAvailable = $ordersTable->getAllAvailableColumns();
foreach ($allAvailable as $column) {
    echo $column->getName() . ' (' . $column->getMeta('auto_join_path') . ')' . PHP_EOL;
}
// Output: uuid, created_at, amount, status, customer.name (customer), customer.email (customer), payload.weight (payload), payload.dimensions (payload), payload.pickup.address (payload.pickup), payload.pickup.city (payload.pickup), ...

// Check if a specific column exists
if ($ordersTable->hasColumn('amount')) {
    echo "Column amount exists";
}

// Get a specific relationship
$customer = $ordersTable->getRelationship('customer');
if ($customer) {
    echo $customer->getLabel(); // "Customer"
}
```

#### Example 3: Checking Column Validity
```php
// Check if a particular column is allowed for use in reports
if ($ordersTable->isColumnAllowed('amount')) {
    // It can be added to the report
}

if (!$ordersTable->isColumnAllowed('deleted_at')) {
    // This column is excluded
}

if ($ordersTable->isColumnAllowed('payload.pickup.city')) {
    // Although this column is not direct, it appears in getAllAvailableColumns
    // (The check here relies on getColumn which does not handle paths; the Registry is needed)
    // So it is better to use isColumnAllowed from the Registry for paths.
}
```

#### Example 4: Interaction with ReportSchemaRegistry
After creating the table, it must be registered in the `ReportSchemaRegistry`:

```php
$registry = app(ReportSchemaRegistry::class);
$registry->registerTable($ordersTable);

// Later, the table can be retrieved
$retrieved = $registry->getTable('orders');
if ($retrieved) {
    $columns = $retrieved->getAllAvailableColumns();
}
```

#### Example 5: Using toArray to Display Table Information
```php
$array = $ordersTable->toArray();
// This JSON can be returned to the frontend
return response()->json($array);
```

---

### Interaction with Other Classes

- **`Column`**: `Table` uses `Column` objects to represent columns. `Table` provides methods to add and retrieve columns.
- **`Relationship`**: `Table` uses `Relationship` objects to represent relationships. It provides methods to filter relationships by `autoJoin`.
- **`ReportSchemaRegistry`**: `Table` is registered in the `Registry`, and it queries the `Registry` for other tables when needed (e.g., to verify existence of columns in relationships).

---

### Best Practices

1. **Consistent Naming**: Use consistent names for tables and relationships (e.g., use `snake_case`).
2. **Use `autoJoin` Wisely**: Make relationships `autoJoin` only when you expect users will frequently need columns from that relationship. This prevents unnecessary JOINs.
3. **Use `excludeColumns` to Hide Sensitive Columns**: Such as `password`, `api_token`, `remember_token`.
4. **Set `maxRows`**: Put a reasonable limit to protect system performance.
5. **Leverage Caching**: Adjust `cacheable` and `cacheTtl` according to table update frequency.

---

### Summary
The `Table` class is the cornerstone of the schema definition system for reporting. It provides an organized and powerful way to describe any database table, with all its details and relationships. Through its clear interface, developers can build an integrated definition layer that feeds the dynamic reporting system with all necessary data, enabling end users to create complex reports easily and securely.