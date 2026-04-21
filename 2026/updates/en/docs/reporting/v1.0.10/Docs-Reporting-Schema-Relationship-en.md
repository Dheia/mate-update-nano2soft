## Documentation of `Relationship` Class

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The `Relationship` class is one of the fundamental classes in the Schema Definition Layer of the `Nano2.QueryBuilder.Reporting` dynamic reporting system. This class represents a **relationship between two tables** in the database, such as `belongsTo`, `hasMany`, or `hasOne`. It provides comprehensive information about the relationship: relationship name, related table, join keys, JOIN type, and available columns from the related table. It also supports **nested relationships** and the **Auto-Join** feature, which allows automatic addition of a JOIN when columns from the relationship are selected.

`Relationship` is primarily used inside the `Table` class to describe how the current table relates to other tables, enabling the reporting system to build complex queries involving multiple tables without manually writing JOINs.

---

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$name` | `string` | Relationship name (e.g., `customer`, `payload`). Used as an identifier in paths (e.g., `customer.name`). |
| `$table` | `string` | Name of the related table in the database (e.g., `customers`, `order_payloads`). |
| `$label` | `string` | Display label for the relationship (e.g., "Customer", "Payload Data") used in the user interface. |
| `$type` | `string` | JOIN type (`left`, `right`, `inner`). Default `left`. |
| `$localKey` | `string` | Local key in the current table that links to the relationship (default `{name}_uuid`). |
| `$foreignKey` | `string` | Foreign key in the related table (default `uuid`). |
| `$enabled` | `bool` | Is the relationship enabled? (default `true`). Can be temporarily disabled without deletion. |
| `$autoJoin` | `bool` | Should this relationship be automatically joined when columns from it are selected? (default `false`). |
| `$description` | `?string` | Optional description of the relationship. |
| `$columns` | `array` | Array of `Column` objects representing available columns from the related table. |
| `$nestedRelationships` | `array` | Array of `Relationship` objects representing relationships inside the related table (e.g., `payload.pickup`). |
| `$meta` | `array` | Additional key-value data (useful for extensions). |

---

### Main Methods

#### 1. Creating a Relationship Object

##### `public static function make(string $name, string $table, string $type = 'left'): self`
- **Purpose**: Create a new relationship with the specified name and table.
- **Parameters**:
  - `$name`: Relationship name (e.g., `customer`).
  - `$table`: Name of the related table.
  - `$type`: JOIN type (default `left`).
- **Returns**: New `Relationship` object.
- **Example**:
  ```php
  $customerRel = Relationship::make('customer', 'customers');
  ```

##### Relationship Helper Methods
These methods provide shortcuts for creating common relationship types:

- **`public static function belongsTo(string $name, string $table): self`**  
  Alias for `make` with `type = 'left'` (default).  
  Example: `Relationship::belongsTo('user', 'users')`

- **`public static function hasMany(string $name, string $table): self`**  
  Alias for `make` with `type = 'left'`.  
  Example: `Relationship::hasMany('posts', 'posts')`

- **`public static function hasOne(string $name, string $table): self`**  
  Alias for `make` with `type = 'left'`.  
  Example: `Relationship::hasOne('profile', 'profiles')`

- **`public static function hasAutoJoin(string $name, string $table, string $type = 'left'): self`**  
  Create a relationship with the `autoJoin` property enabled.  
  Example: `Relationship::hasAutoJoin('payload', 'order_payloads')`

#### 2. Setting Properties (Fluent Setters)

##### `public function label(string $label): self`
- Set display label for the relationship.
- **Example**: `$customerRel->label('Customer');`

##### `public function description(string $description): self`
- Set relationship description.

##### `public function localKey(string $localKey): self`
- Set the local key (column in the current table).
- **Example**: `$customerRel->localKey('customer_uuid');`

##### `public function foreignKey(string $foreignKey): self`
- Set the foreign key (column in the related table).
- **Example**: `$customerRel->foreignKey('uuid');`

##### `public function joinType(string $type): self`
- Set the JOIN type (`left`, `right`, `inner`).

##### `public function enabled(bool $enabled = true): self`
- Enable or disable the relationship.

##### `public function autoJoin(bool $autoJoin = true): self`
- Enable or disable the auto-join feature.

##### `public function columns(array $columns): self`
- Add a set of columns to the relationship. Can accept:
  - Ready-made `Column` objects.
  - Arrays containing `name` and `type` (will be converted to `Column`).
  - String column names (will be converted to `Column` with type `string`).
- **Example**:
  ```php
  $customerRel->columns([
      'name',
      'email',
      Column::make('phone')->label('Phone Number'),
  ]);
  ```

##### `public function addColumn(Column $column): self`
- Add a single column to the relationship.

##### `public function with(array $relationships): self`
- Add a set of nested relationships. Must be `Relationship` objects.
- **Example**:
  ```php
  $payloadRel->with([
      $pickupRel,
      $dropoffRel,
  ]);
  ```

##### `public function addNestedRelationship(Relationship $relationship): self`
- Add a single nested relationship.

##### `public function meta(string $key, $value): self`
- Add metadata.

#### 3. Getters

| Method | Returns |
|--------|---------|
| `getName(): string` | Relationship name |
| `getTable(): string` | Related table name |
| `getLabel(): string` | Label |
| `getType(): string` | JOIN type |
| `getLocalKey(): string` | Local key |
| `getForeignKey(): string` | Foreign key |
| `getDescription(): ?string` | Description |
| `isEnabled(): bool` | Is relationship enabled? |
| `isAutoJoin(): bool` | Is it auto-join? |
| `getColumns(): array` | Direct columns (without nested) |
| `getNestedRelationships(): array` | Nested relationships |
| `getMeta(?string $key = null)` | Metadata |

#### 4. Advanced Methods

##### `public function getAllAvailableColumns(): array`
- **Purpose**: Returns all available columns from this relationship, including columns from nested relationships. This is done by:
  - Starting with direct columns (`$this->columns`).
  - For each nested relationship, fetching its columns using its own `getAllAvailableColumns()`, then modifying each column using `copyWith()` to prefix the path (e.g., `pickup.`) to the column name and adjust the label to reflect the context.
- **Returns**: Array of `Column` objects (modified copies).
- **Example**:
  ```php
  $allColumns = $payloadRel->getAllAvailableColumns();
  // Contains columns like: weight, dimensions, pickup.address, pickup.city, dropoff.address, ...
  ```

##### `public function getNestedRelationship(string $name): ?Relationship`
- **Purpose**: Search for a nested relationship by the given name.
- **Example**:
  ```php
  $pickup = $payloadRel->getNestedRelationship('pickup');
  ```

##### `public function getAutoJoinRelationships(): array`
- **Purpose**: Return nested relationships that have `autoJoin = true`.

##### `public function getManualJoinRelationships(): array`
- **Purpose**: Return nested relationships that do not have `autoJoin`.

##### `public function hasNestedRelationships(): bool`
- **Purpose**: Check if there are any nested relationships.

##### `public function toArray(): array`
- **Purpose**: Convert the relationship to a serializable array (for sending via API). Contains all basic properties.

#### 5. Protected Internal Methods

##### `protected function setAutoJoin(bool $autoJoin): self`
- **Purpose**: Set the auto-join property (used internally).

##### `protected function generateLabel(string $name): string`
- **Purpose**: Generate a default label from the relationship name (convert underscores and dashes to spaces, then capitalize first letter).

##### `protected function generateLocalKey(string $name): string`
- **Purpose**: Generate a default local key with the format `{name}_uuid` (because the Fleetbase system uses `_uuid` as a suffix for foreign keys).

---

### Comprehensive Practical Examples

#### Example 1: Creating a simple relationship (belongsTo)
```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Relationship;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;

$customerRel = Relationship::belongsTo('customer', 'customers')
    ->label('Customer')
    ->localKey('customer_uuid')
    ->foreignKey('uuid')
    ->columns([
        Column::make('name')->label('Name'),
        Column::make('email')->label('Email'),
        Column::make('phone')->label('Phone Number'),
    ]);
```

#### Example 2: Creating an auto-join relationship with columns
```php
$payloadRel = Relationship::hasAutoJoin('payload', 'order_payloads')
    ->label('Payload Data')
    ->columns([
        Column::make('weight', 'decimal')->label('Weight'),
        Column::make('dimensions')->label('Dimensions'),
        Column::make('notes')->label('Notes'),
    ]);
```

#### Example 3: Creating nested relationships
Suppose the `order_payloads` table has multiple relationships with the `addresses` table (pickup and dropoff):

```php
// Common columns for addresses
$addressColumns = [
    Column::make('address')->label('Address'),
    Column::make('city')->label('City'),
    Column::make('country')->label('Country'),
    Column::make('postal_code')->label('Postal Code'),
];

// Pickup relationship
$pickupRel = Relationship::hasAutoJoin('pickup', 'addresses')
    ->label('Pickup Location')
    ->localKey('payload_uuid')  // Local key in addresses table pointing to payload
    ->foreignKey('uuid')
    ->columns($addressColumns)
    ->autoJoin(true);

// Dropoff relationship
$dropoffRel = Relationship::hasAutoJoin('dropoff', 'addresses')
    ->label('Dropoff Location')
    ->localKey('payload_uuid')
    ->foreignKey('uuid')
    ->columns($addressColumns)
    ->autoJoin(true);

// Main payload relationship
$payloadRel = Relationship::hasAutoJoin('payload', 'order_payloads')
    ->label('Payload Data')
    ->columns([
        Column::make('weight', 'decimal')->label('Weight'),
        Column::make('dimensions')->label('Dimensions'),
    ])
    ->with([$pickupRel, $dropoffRel]); // Adding nested relationships
```

#### Example 4: Using getAllAvailableColumns
```php
// After defining relationships as in the previous example
$allColumns = $payloadRel->getAllAvailableColumns();

foreach ($allColumns as $column) {
    echo $column->getName() . ' - ' . $column->getLabel() . PHP_EOL;
}

// Output:
// weight - Weight
// dimensions - Dimensions
// pickup.address - Pickup Location - Address
// pickup.city - Pickup Location - City
// pickup.country - Pickup Location - Country
// pickup.postal_code - Pickup Location - Postal Code
// dropoff.address - Dropoff Location - Address
// dropoff.city - Dropoff Location - City
// dropoff.country - Dropoff Location - Country
// dropoff.postal_code - Dropoff Location - Postal Code
```

Notice how column names have been modified by adding the path prefix (`pickup.`, `dropoff.`) and labels have been adjusted to reflect the context.

#### Example 5: Using relationships in building a table
```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Table;

$ordersTable = Table::make('orders')
    ->label('Orders')
    ->columns([
        Column::make('uuid')->hidden(true),
        Column::make('created_at', 'datetime')->label('Creation Date'),
        Column::make('amount', 'decimal')->label('Amount'),
    ])
    ->relationships([
        $customerRel,  // customer relationship
        $payloadRel,   // payload relationship (which contains pickup and dropoff internally)
    ]);
```

Now, when calling `$ordersTable->getAllAvailableColumns()`, columns like `customer.name`, `payload.weight`, `payload.pickup.city`, etc., will appear.

#### Example 6: Searching for a nested relationship
```php
$pickup = $payloadRel->getNestedRelationship('pickup');
if ($pickup) {
    echo $pickup->getLabel(); // "Pickup Location"
    $pickupColumns = $pickup->getColumns(); // only pickup columns
}
```

#### Example 7: Checking relationship properties
```php
if ($payloadRel->isAutoJoin()) {
    echo "This relationship will be automatically added when needed";
}

if ($payloadRel->hasNestedRelationships()) {
    echo "Contains nested relationships: " . count($payloadRel->getNestedRelationships());
}
```

#### Example 8: Converting the relationship to an array
```php
$array = $payloadRel->toArray();
// Can be sent to the frontend to display relationship information
```

---

### Interaction with Other Classes

- **With `Table`**: `Relationship` is used inside `Table` via the `relationships()` and `addRelationship()` methods. `Table` provides methods like `getRelationships()`, `getAutoJoinRelationships()`, `getRelationship(name)` that rely on `Relationship`.
- **With `Column`**: `Relationship` contains an array of `Column` objects representing columns of the related table. Also, `getAllAvailableColumns()` uses the `copyWith()` method from `Column` to modify column copies.
- **With `ReportQueryConverter`**: When building the query, the converter uses relationship information (keys, JOIN type, table) to create JOINs. It also uses `getAllAvailableColumns()` (via `Table`) to determine available columns.

---

### Best Practices

1. **Use clear and consistent relationship names**: Such as `customer`, `payload`, `pickup`, `dropoff`. This makes paths (e.g., `customer.name`) easy to understand.
2. **Enable `autoJoin` wisely**: Make relationships `autoJoin` only when you expect users will frequently need columns from that relationship. Excessive auto-join may lead to unnecessary JOINs.
3. **Explicitly specify keys**: Do not rely on default values (`{name}_uuid`) if keys differ. Use `localKey()` and `foreignKey()` to specify them accurately.
4. **Organize nested relationships hierarchically**: When multiple levels of relationships exist (e.g., `payload.pickup`), build them sequentially using `with()`.
5. **Use `copyWith` cautiously in nested columns**: This is done automatically in `getAllAvailableColumns()`, so there is no need to use it manually except in special cases.
6. **Avoid cyclic relationships**: Do not create nested relationships that cause a cycle (e.g., A → B → A), as they may cause issues in query building.
7. **Use `enabled(false)` to temporarily disable relationships**: Instead of deleting them, they can be disabled if needed.

---

### Summary
The `Relationship` class is the backbone for representing relationships between tables in the dynamic reporting system. With its support for nested relationships and the auto-join feature, it can build complex data schemas that accurately reflect the real database structure. By providing formatted columns with clear paths, it makes it easy for end users to select data from any relationship level without needing to understand JOIN details. `Relationship` integrates closely with `Table` and `Column` to form a powerful and flexible definition layer.