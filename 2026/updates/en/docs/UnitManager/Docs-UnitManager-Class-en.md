# `UnitManager` Class Documentation

**Namespace:** `Tss\Inventory\Classes\UnitManager`

---

## 1. Introduction

The `UnitManager` class is the unified programming interface (Facade) for handling the unit system in the `Tss.Inventory` package. This class provides static methods that cover most operations related to units: creating new units, validating units (especially recharge cards), retrieving categorized unit lists, printing reports, generating unique codes, and helper tools for queries.

The goal of this class is to simplify the use of the system and unify access to various functions, reducing redundancy and making code maintenance easier.

---

## 2. Available Functions

### 2.1 Unit Creation and Management Functions

#### `createUnit(array $options = [], bool $is_test_create = false): array`

**Description:**  
Create a new unit in the database. This function relies on `UnitCreator::createUnit` and passes the options to it.

**Parameters:**
- `$options` : An array containing unit data. Key options include:
  - `name` (string) : Unit name (required).
  - `unit_symbol` (string) : Unit symbol (e.g., 'kg', 'm', 'dozen').
  - `type` (string) : Unit type (one of the `UnitType` constants).
  - `companys_id` (int) : Company ID.
  - `departments_id` (int) : Department ID.
  - `conversion_factor` (float) : Conversion factor (relative to the base unit).
  - `is_base_unit` (bool) : Is it the base unit?
  - `is_default` (bool) : Is it default?
  - `is_active` (bool) : Activation status.
  - And other fields present in the `Unit` model.
- `$is_test_create` : If `true`, the operation is performed within a transaction and then rolled back (for testing purposes).

**Return Value:**  
An array containing:
- `status` (bool) : Success or failure of the operation.
- `message` (string) : Explanatory message.
- `error` (string|null) : Error text if any.
- `model` (object|null) : The created unit object.
- `data` (array) : Unit data (`toArray()` copy).
- `input_data`, `process_data` (array) : Additional information for tracking.

**Example:**
```php
$result = UnitManager::createUnit([
    'name' => 'Kilogram',
    'unit_symbol' => 'kg',
    'type' => UnitType::WEIGHT,
    'is_base_unit' => true,
    'conversion_factor' => 1,
    'companys_id' => 1,
    'departments_id' => 2,
]);

if ($result['status']) {
    echo "Unit created: " . $result['model']->name;
} else {
    echo "Error: " . $result['error'];
}
```

---

#### `checkUnit(array $options = []): array`

**Description:**  
Validate a unit (usually used for checking recharge cards or codes). The function checks the unit's existence, status, balance, and user match (if requested).

**Parameters:**
- `$options` : An array containing search and validation criteria. Key options include:
  - `unit_symbol` (string) : The symbol of the unit to be validated (required).
  - `companys_id` (mixed) : Company ID (can be `'*'` to search all).
  - `departments_id` (mixed) : Department ID.
  - `total` (float) : Required value (to check for sufficient balance).
  - `is_force_user` (bool) : Must the user match?
  - `user` , `user_id` , `user_type` : User data.
  - `is_model` (bool) : If `true`, the unit object is returned in the result.

**Return Value:**  
An array containing:
- `status` (bool) : `true` if the unit is valid.
- `message` (string) : Explanatory message.
- `error` (string|null) : Error text if any.
- `model` (object|null) : The unit object (if `is_model` is requested).
- `input_data`, `process_data` (array) : Additional information.

**Examples:**

*Check if a recharge card with a specific code exists:*
```php
$check = UnitManager::checkUnit([
    'unit_symbol' => 'CARD12345',
    'is_active' => true,
]);

if ($check['status']) {
    echo "Card is valid.";
} else {
    echo "Card is invalid: " . $check['error'];
}
```

*Check a card and ensure the user is its owner:*
```php
$user = Auth::getUser();
$check = UnitManager::checkUnit([
    'unit_symbol' => 'CARD12345',
    'is_force_user' => true,
    'user' => $user,
]);

if (!$check['status']) {
    throw new Exception($check['error']);
}
```

*Check a card with sufficient balance:*
```php
$check = UnitManager::checkUnit([
    'unit_symbol' => 'CARD12345',
    'total' => 100, // We want to use 100 units
]);

if ($check['status']) {
    echo "Balance is sufficient.";
} else {
    echo "Insufficient balance or invalid card.";
}
```

---

### 2.2 Query and Statistics Functions

#### `getAllUnitsGrouped(bool $groupByMainType = true, string $lang = 'en'): array`

**Description:**  
Return all supported units in the system, categorized by type (with labels). Internally uses `UnitConverter::getAllUnitsGrouped`.

**Parameters:**
- `$groupByMainType` : If `true`, grouping is done by main types (e.g., `length`, `weight`). If `false`, quantity subtypes are included.
- `$lang` : Language (`'en'` or `'ar'`).

**Return Value:**  
An associative array where the key is the type identifier, and the value is an array containing:
- `label` : Type label.
- `units` : An array of unit symbols and their labels.
- `type` : Type identifier.

**Example:**
```php
$grouped = UnitManager::getAllUnitsGrouped(true, 'en');
foreach ($grouped as $type => $info) {
    echo $info['label'] . ":\n";
    foreach ($info['units'] as $symbol => $label) {
        echo "  - $symbol : $label\n";
    }
}
```

---

#### `getUnitsForType($type, bool $includeLabels = true, string $lang = 'en'): array`

**Description:**  
Return a list of units belonging to a specific type.

**Parameters:**
- `$type` : Unit type (e.g., `UnitType::LENGTH`).
- `$includeLabels` : If `true`, returns an associative array `[symbol => label]`; otherwise returns a simple list of symbols.
- `$lang` : Language.

**Return Value:**  
An array of symbols (or an associative array depending on `$includeLabels`).

**Example:**
```php
$weightUnits = UnitManager::getUnitsForType(UnitType::WEIGHT, true, 'en');
// ['mg' => 'Milligram', 'g' => 'Gram', 'kg' => 'Kilogram', ...]
```

---

#### `getRentalTimeUnits(string $lang = 'en', bool $is_key = false): array`

**Description:**  
Return time units used in the rental context (hour, day, week, month).

**Parameters:**
- `$lang` : Language.
- `$is_key` : If `true`, returns only the list of symbols (`['h','day','week','month']`). If `false`, returns an associative array `[symbol => label]`.

**Return Value:**  
An array according to `$is_key`.

**Example:**
```php
$rentalUnits = UnitManager::getRentalTimeUnits('en');
// ['h' => 'Hour', 'day' => 'Day', 'week' => 'Week', 'month' => 'Month']
```

---

### 2.3 Query Helper Functions

#### `getQueryDate($records, array $options = [])`

**Description:**  
Apply date conditions to a query (Builder). Used internally to add time filters.

**Parameters:**
- `$records` : The query object (Builder).
- `$options` : An array containing date options, such as:
  - `field_date` : The name of the date field.
  - `date_at` : The date value (or date from).
  - `to_date_at` : The date to (for range).
  - `date_at_opration` : The operation (`=`, `>=`, `between`, `notbetween` ...).
  - `is_or` : If `true`, uses `orWhere`.

**Return Value:**  
The modified query object.

**Example:**
```php
$query = Unit::query();
$query = UnitManager::getQueryDate($query, [
    'field_date' => 'created_at',
    'date_at' => '2026-03-01',
    'to_date_at' => '2026-03-18',
    'date_at_opration' => 'between',
]);
$units = $query->get();
```

---

#### `scopeWhereField($query, $field, $value = null, $is_or = false, $table = null, $is_not = false, $is_force = false)`

**Description:**  
Apply a `where` or `whereIn` condition on a specific field, with support for special values like `'*'` or `'all'` (ignored if `is_force` is false). Used in scopes.

**Parameters:**
- `$query` : The query object.
- `$field` : The field name.
- `$value` : The value (could be a string or an array).
- `$is_or` : If `true`, uses `orWhere`.
- `$table` : The table name (prepended to the field).
- `$is_not` : If `true`, uses `whereNotIn` or `!=`.
- `$is_force` : If `true`, the condition is applied even if the value is `'*'` or `'all'`.

**Return Value:**  
The modified query object.

**Example:**
```php
$query = Unit::query();
$query = UnitManager::scopeWhereField($query, 'type', ['weight', 'length'], false, 'tss_inventory_units');
// Produces: where (type in ('weight','length'))
```

---

### 2.4 Report Printing Functions

#### `printReports(array $options = [])`

**Description:**  
Print a report about units using the `Tss\Reports` system. Returns the HTML content of the report.

**Parameters:**
- `$options` : Same options accepted by `Unit::getRecords` (determine the scope of units to be printed).

**Return Value:**  
HTML string containing the report.

**Example:**
```php
$html = UnitManager::printReports([
    'type' => UnitType::WEIGHT,
    'is_active' => true,
    'orderBy' => 'name',
]);
echo $html; // Display the report in the browser
```

---

### 2.5 Unique Code and Identifier Generation Functions

#### `generateDigital(int $length = 6): int`

**Description:**  
Generate a random number of a given length (e.g., for card codes).

**Parameters:**
- `$length` : The required number of digits (default 6).

**Return Value:**  
A random integer.

**Example:**
```php
$code = UnitManager::generateDigital(8); // Example: 83472915
```

---

#### `getGuid(): string`

**Description:**  
Generate a random UUID (Universally Unique Identifier) without curly braces `{}`.

**Return Value:**  
A string in the format `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

**Example:**
```php
$uuid = UnitManager::getGuid(); // "f47ac10b-58cc-4372-a567-0e02b2c3d479"
```

---

#### `getVoucherCode(): string`

**Description:**  
Generate a 13-digit voucher code based on time and random numbers.

**Return Value:**  
A 13-digit numeric string.

**Example:**
```php
$voucher = UnitManager::getVoucherCode(); // "4829117456382"
```

---

#### `humanUuid(array $options = []): string`

**Description:**  
Generate a human-friendly UUID in the format `YYYYMMDD-HHMM-SSMM-MMMMRRRRRRRRRRRR` (32 characters). Dashes can be added.

**Parameters:**
- `$options` : Can pass `'useDashes' => true` to include dashes.

**Return Value:**  
A string.

**Example:**
```php
$id = UnitManager::humanUuid(['useDashes' => true]); // "20260318-1430-1234-5678-123456789012"
```

---

#### `nanoUid(array $options = []): string`

**Description:**  
Generate a highly precise unique identifier based on nanoseconds, in the format `YYYYMMDD-HHMMSS-MMMMMM-NNN` (23 characters).

**Parameters:**
- `$options` : Can pass `'useDashes' => true`.

**Return Value:**  
A string.

**Example:**
```php
$uid = UnitManager::nanoUid(); // "202603181430225643210"
```

---

### 2.6 Other Helper Functions

#### `getProductRecords(array $options = [])`

**Description:**  
Redirects to `Product::getRecords`. (For compatibility).

#### `newProductModel()`

**Description:**  
Create a new `Product` object.

#### `newInventoryLotModel()` and `newUnitModel()`

**Description:**  
Create new objects from specific models (for internal use).

#### `checkValueIsNotAll($value = null): bool`

**Description:**  
Check if the value does not represent "all" (`'*'` or `'all'`). Used in filtering functions.

**Example:**
```php
if (UnitManager::checkValueIsNotAll($value)) {
    // The value is specific and not 'all'
}
```

#### `getAllContexts(): array`

**Description:**  
Return all available contexts for quantity units (from `UnitConverter::getAllContexts`).

#### `getAllUnitLabels(): array`

**Description:**  
Return all unit labels (from `UnitNameHelper::getAllUnitsWithLabels`).

---

## 3. Best Practices

- **Use `createUnit` instead of directly interacting with `UnitCreator`** for unified error handling.
- **When validating recharge cards**, use `checkUnit` with appropriate parameters (`is_force_user`, `total`).
- **For querying units**, use `getAllUnitsGrouped` and `getUnitsForType` instead of building arrays manually.
- **For generating unique codes**, choose the appropriate function based on precision and length requirements (e.g., `nanoUid` for high precision, `generateDigital` for simple numeric codes).
- **When creating reports**, pass filtering options through `printReports` instead of building the query and then the report manually.

---

## 4. Conclusion

The `UnitManager` class provides a comprehensive and easy-to-use programming interface for all unit-related operations in the TSS Inventory system. Using its static methods, developers can:

- Create and manage units.
- Validate units (especially recharge cards).
- Obtain organized lists of units.
- Apply advanced conditions to queries.
- Print professional reports.
- Generate unique codes and identifiers.

Thanks to this design, redundancy is reduced and maintainability is increased, while providing high flexibility to meet the needs of various applications.
