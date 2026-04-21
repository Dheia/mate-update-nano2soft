## Documentation of `Column` Class

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The `Column` class is the fundamental unit in the Schema Definition Layer of the `Nano2.QueryBuilder.Reporting` dynamic reporting system. This class represents a **database column** or a **computed column** derived from an SQL expression. It contains all the necessary properties to describe the column and its capabilities (searching, sorting, filtering, aggregation) as well as a custom transformer function to format values before display.

`Column` is used inside the `Table` class to describe table columns, and inside the `Relationship` class to describe columns of the related table. It can also be used to create complex computed columns using allowed SQL functions.

---

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$name` | `string` | Actual column name in the database (e.g., `created_at`). |
| `$label` | `string` | Display label for the user (e.g., "Creation Date"), used in the user interface. |
| `$type` | `string` | Data type (string, integer, decimal, date, datetime, boolean, ...). |
| `$description` | `?string` | Optional description of the column clarifying its content. |
| `$format` | `?string` | Optional display format (e.g., `Y-m-d` for date, `#,##0.00` for number). |
| `$nullable` | `bool` | Can the value be NULL? (default `true`). |
| `$searchable` | `bool` | Can this column be searched? (default `true`). |
| `$sortable` | `bool` | Can results be sorted based on this column? (default `true`). |
| `$filterable` | `bool` | Can this column be used in filter conditions? (default `true`). |
| `$aggregatable` | `bool` | Can this column be used in aggregate functions (SUM, AVG, etc.)? (automatically determined for numbers). |
| `$hidden` | `bool` | Hide the column from the interface (remains available for querying). (default `false`). |
| `$computed` | `bool` | Is it a computed column (result of an SQL expression)? (default `false`). |
| `$computation` | `?string` | SQL expression used to compute the column if `computed = true`. |
| `$transformer` | `?\Closure` | PHP function to transform the value before display (applied to each value). |
| `$meta` | `array` | Additional key-value data (useful for extensions). |

---

### Main Methods

#### 1. Creating a Column Object

##### `public static function make(string $name, string $type = 'string'): self`
- **Purpose**: Create a new regular column.
- **Parameters**:
  - `$name`: Column name.
  - `$type`: Data type (default `string`).
- **Returns**: New `Column` object.
- **Example**:
  ```php
  $column = Column::make('status', 'string');
  ```

##### `public static function computed(string $name, string $computation, string $type = 'string', array $options = []): self`
- **Purpose**: Create a computed column with an SQL expression.
- **Parameters**:
  - `$name`: Column name (will appear with this name in results).
  - `$computation`: SQL expression (e.g., `"ROUND(amount * 1.14, 2)"`).
  - `$type`: Resulting data type.
  - `$options`: Array of additional options (e.g., `aggregatable`, `sortable`, `searchable`).
- **Example**:
  ```php
  $totalColumn = Column::computed('total_with_tax', 'ROUND(amount * 1.14, 2)', 'decimal')
      ->label('Total Including Tax');
  ```

#### 2. Aggregate Helper Methods
These methods provide shortcuts for creating columns representing SQL aggregate functions.

##### `public static function count(string $name, string $countField = '*'): self`
- **Purpose**: Create a COUNT column.
- **Parameters**:
  - `$name`: Column name.
  - `$countField`: Field to count (default `*`).
- **Example**:
  ```php
  $orderCount = Column::count('order_count', 'uuid')
      ->label('Number of Orders');
  ```

##### `public static function sum(string $name, string $sumField): self`
- **Purpose**: Create a SUM column.
- **Parameters**:
  - `$name`: Column name.
  - `$sumField`: Field to sum.
- **Example**:
  ```php
  $totalAmount = Column::sum('total_amount', 'amount')
      ->label('Total Amounts');
  ```

##### `public static function avg(string $name, string $avgField): self`
- **Purpose**: Create an AVG column.
- **Example**:
  ```php
  $averageAmount = Column::avg('avg_amount', 'amount')
      ->label('Average Amount');
  ```

##### `public static function max(string $name, string $maxField): self`
- **Purpose**: Create a MAX column.

##### `public static function min(string $name, string $minField): self`
- **Purpose**: Create a MIN column.

#### 3. Setting Properties (Fluent Setters)
All these methods return the object itself (`self`) to enable chaining.

##### `public function label(string $label): self`
- Set display label.
- **Example**: `$column->label('Status');`

##### `public function description(string $description): self`
- Set description.

##### `public function format(string $format): self`
- Set display format (e.g., `'Y-m-d'` for date, `'#,##0.00'` for numbers).

##### `public function nullable(bool $nullable = true): self`
- Specify whether the column accepts NULL values.

##### `public function searchable(bool $searchable = true): self`
- Specify searchability.

##### `public function sortable(bool $sortable = true): self`
- Specify sortability.

##### `public function filterable(bool $filterable = true): self`
- Specify filterability.

##### `public function aggregatable(bool $aggregatable = true): self`
- Specify aggregatability (used in functions like SUM, AVG).

##### `public function hidden(bool $hidden = true): self`
- Hide the column from the user interface.

##### `public function meta(string $key, $value): self`
- Add a single metadata key-value pair.

##### `public function setMeta(array $meta): self`
- Set a set of metadata.

#### 4. Transformer Methods

##### `public function transformer(\Closure|callable $transformer): self`
- **Purpose**: Register a function to transform the column value before sending it to the interface.
- **Parameters**: A PHP function (either Closure or callable) that receives the value and returns the transformed value.
- **Example**:
  ```php
  $column->transformer(function ($value) {
      return $value ? 'Active' : 'Inactive';
  });
  ```

##### `public function transformValue($value)`
- **Purpose**: Apply the transformer to a given value. If there is no transformer, the value is returned as is.

#### 5. Getters

| Method | Returns |
|--------|---------|
| `getName(): string` | Column name |
| `getLabel(): string` | Label |
| `getType(): string` | Type |
| `getDescription(): ?string` | Description |
| `getFormat(): ?string` | Format |
| `isNullable(): bool` | Accepts NULL? |
| `isSearchable(): bool` | Is searchable? |
| `isSortable(): bool` | Is sortable? |
| `isFilterable(): bool` | Is filterable? |
| `isAggregatable(): bool` | Is aggregatable? |
| `isHidden(): bool` | Is hidden? |
| `isComputed(): bool` | Is computed? |
| `getComputation(): ?string` | Computation expression (if computed) |
| `getTransformer(): ?\Closure` | Transformer function |
| `hasTransformer(): bool` | Does a transformer exist? |
| `getMeta(?string $key = null)` | Metadata (all or a specific key) |

#### 6. Additional Methods

##### `public function isForeignKey(): bool`
- **Purpose**: Check if the column is a foreign key (ends with `_uuid` or `_id`).
- **Example**:
  ```php
  if ($column->isForeignKey()) {
      // This column is used in relationships
  }
  ```

##### `public function toArray(): array`
- **Purpose**: Convert the column to a serializable array (for sending via API). Contains all public properties.
- **Example**:
  ```php
  $array = $column->toArray();
  // Can be returned as JSON
  ```

##### `public function copyWith(array $overrides): self`
- **Purpose**: Create a modified copy of the column. Used especially in nested relationships to change the column name or label while preserving other properties.
- **Parameters**: `$overrides` array containing keys to change (`name`, `label`, `type`, `description` ...).
- **Example**:
  ```php
  $newColumn = $originalColumn->copyWith([
      'name' => 'new_name',
      'label' => 'New Label'
  ]);
  ```

##### `protected function generateLabel(string $name): string`
- **Purpose**: Generate a default label from the column name (convert underscores and dashes to spaces, then capitalize first letter).

##### `protected function determineAggregatable(string $type): bool`
- **Purpose**: Determine the default value for the `aggregatable` property based on the data type (numbers and dates are aggregatable, strings are not).

---

### Comprehensive Practical Examples

#### Example 1: Creating various regular columns
```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;

$columns = [
    // Regular text column
    Column::make('status', 'string')
        ->label('Status')
        ->searchable(true)
        ->sortable(true)
        ->filterable(true),

    // Date column with formatting
    Column::make('created_at', 'datetime')
        ->label('Creation Date')
        ->format('Y-m-d H:i')
        ->sortable(true),

    // Numeric column with aggregation support
    Column::make('amount', 'decimal')
        ->label('Amount')
        ->aggregatable(true)
        ->format('#,##0.00'),

    // Hidden column (for internal use)
    Column::make('internal_code')
        ->label('Internal Code')
        ->hidden(true),

    // Column with transformer
    Column::make('is_active', 'boolean')
        ->label('Active')
        ->transformer(fn($v) => $v ? 'Yes' : 'No')
];
```

#### Example 2: Creating computed columns
```php
// Computed column concatenating first and last name
$fullNameColumn = Column::computed('full_name', "CONCAT(first_name, ' ', last_name)", 'string')
    ->label('Full Name')
    ->searchable(true); // Searching on computed columns is possible (requires database support)

// Computed column calculating tax
$taxColumn = Column::computed('tax_amount', "ROUND(amount * 0.14, 2)", 'decimal')
    ->label('Tax')
    ->aggregatable(true);

// Computed column using CASE
$statusTextColumn = Column::computed('status_text', 
    "CASE WHEN status = 'completed' THEN 'Completed' WHEN status = 'pending' THEN 'Pending' ELSE 'Other' END", 
    'string'
)->label('Status Text');
```

#### Example 3: Using aggregate helper methods
```php
$orderCount = Column::count('order_count', 'uuid')
    ->label('Number of Orders');

$totalRevenue = Column::sum('total_revenue', 'amount')
    ->label('Total Revenue');

$averageRating = Column::avg('avg_rating', 'rating')
    ->label('Average Rating');

$maxPrice = Column::max('max_price', 'price')
    ->label('Highest Price');

$minPrice = Column::min('min_price', 'price')
    ->label('Lowest Price');
```

#### Example 4: Using copyWith in nested relationships
Suppose we have a nested relationship like `payload.pickup`, and we want to create an `address` column for this level with an appropriate label:

```php
// Original column in the addresses table
$originalAddressColumn = Column::make('address')->label('Address');

// When integrating it into a pickup relationship, we want to change the label to "Pickup Address"
$pickupAddressColumn = $originalAddressColumn->copyWith([
    'label' => 'Pickup Address'
]);

// Or change the name to include the path (this is done automatically in Relationship::getAllAvailableColumns)
$pickupAddressColumnWithPath = $originalAddressColumn->copyWith([
    'name' => 'payload.pickup.address',
    'label' => 'Pickup Address'
]);
```

#### Example 5: Using transformer to transform values
```php
// Convert boolean value to Arabic text (or any language)
$activeColumn = Column::make('is_active', 'boolean')
    ->label('Status')
    ->transformer(function ($value) {
        return $value ? 'Active' : 'Inactive';
    });

// Transform date to a different format
$dateColumn = Column::make('created_at', 'datetime')
    ->label('Date')
    ->transformer(function ($value) {
        return $value ? \Carbon\Carbon::parse($value)->format('d/m/Y') : '';
    });

// Convert number to currency
$amountColumn = Column::make('amount', 'decimal')
    ->label('Amount')
    ->transformer(function ($value) {
        return number_format($value, 2) . ' KWD';
    });
```

#### Example 6: Using properties in building a table
```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Table;

$ordersTable = Table::make('orders')
    ->columns([
        Column::make('uuid')->hidden(true),
        Column::make('created_at', 'datetime')->label('Order Date')->sortable(true),
        Column::make('amount', 'decimal')->label('Amount')->aggregatable(true),
        Column::make('status', 'string')->label('Status')->filterable(true),
    ])
    ->computedColumns([
        Column::computed('total_with_tax', 'ROUND(amount * 1.14, 2)', 'decimal')
            ->label('Total Including Tax'),
    ]);
```

#### Example 7: Accessing properties and checking
```php
$column = Column::make('email', 'string')
    ->label('Email')
    ->searchable(true)
    ->sortable(true)
    ->nullable(false);

echo $column->getName(); // email
echo $column->getLabel(); // Email
echo $column->isSearchable() ? 'Yes' : 'No'; // Yes
echo $column->isNullable() ? 'Yes' : 'No'; // No
echo $column->isAggregatable() ? 'Yes' : 'No'; // No (because it's string)

if ($column->isForeignKey()) {
    // No, because it doesn't end with _uuid or _id
}
```

#### Example 8: Converting column to array
```php
$column = Column::make('status', 'string')
    ->label('Status')
    ->filterable(true)
    ->meta('custom_key', 'custom_value');

$array = $column->toArray();
/*
[
    'name' => 'status',
    'label' => 'Status',
    'type' => 'string',
    'description' => null,
    'format' => null,
    'nullable' => true,
    'searchable' => true,
    'sortable' => true,
    'filterable' => true,
    'aggregatable' => false,
    'hidden' => false,
    'computed' => false,
    'computation' => null,
    'transformer' => false,
    'meta' => ['custom_key' => 'custom_value']
]
*/
```

---

### Interaction with Other Classes

- **With `Table`**: `Column` is used inside `Table` to define table columns via the `columns()` and `computedColumns()` methods. `Table` also uses methods like `getColumn()` and `getAllAvailableColumns()` which rely on `Column`.
- **With `Relationship`**: `Column` is used inside `Relationship` to define columns of the related table via the `columns()` method. Also, `Relationship::getAllAvailableColumns()` returns modified copies of columns (using `copyWith`) to include the relationship path in the name.
- **With `ReportQueryConverter`**: When building the query, the converter transforms the column name into an `alias.column` reference using join information. It also uses `ComputedColumnValidator` to validate computed column expressions.

---

### Best Practices

1. **Use clear labels**: Make labels understandable to the end user.
2. **Specify properties accurately**: Do not make all columns searchable and sortable if not needed; this improves interface performance.
3. **Use `hidden` for technical columns**: Such as `uuid`, `created_at` might be useful for querying but the user does not need to see them.
4. **Set `aggregatable` only for numeric columns**: Avoid using aggregation on text columns.
5. **Use `transformer` to format values instead of changing them in the interface**: This maintains separation of concerns.
6. **When creating computed columns, ensure the expression uses allowed functions and is compatible with the SQL of the database in use**.
7. **Use `copyWith` with caution**: It is typically used internally in `Relationship`, but can be used if you need a modified copy.

---

### Summary
The `Column` class is the fundamental unit for describing data in the dynamic reporting system. It provides a flexible and powerful way to define column properties, whether regular or computed, with the ability to customize every aspect of their behavior (search, sort, filter, aggregate, transform). By using it with `Table` and `Relationship`, an integrated definition layer can be built that enables users to create complex reports easily and securely.