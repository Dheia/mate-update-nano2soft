# `UnitCreator` Class Documentation

**Namespace:** `Tss\Inventory\Classes\Units`

---

## 📋 Table of Contents

1. [Introduction](#introduction)
2. [Internal Helper Functions](#internal-helper-functions)
   - [getAndCheckDepartmentsId](#getandcheckdepartmentsid)
   - [getAndCheckCompanysId](#getandcheckcompanysid)
3. [Main Functions](#main-functions)
   - [createUnit](#createunit)
   - [createUnitFromSymbol](#createunitfromsymbol)
   - [createStandardUnitsForType](#createstandardunitsfortype)
   - [createAllStandardUnits](#createallstandardunits)
   - [attachUnitToProduct](#attachunittoproduct)
   - [createProductUnitPrice](#createproductunitprice)
   - [createProductUnitsStructure](#createproductunitsstructure)
   - [createUnitConversion](#createunitconversion)
   - [importUnitsFromSymbols](#importunitsfromsymbols)
   - [createStandardQuantityUnits](#createstandardquantityunits)
   - [firstOrCreateFromSymbol](#firstorcreatefromsymbol)
4. [Result Array Structure](#result-array-structure)
5. [Comprehensive Examples](#comprehensive-examples)

---

## Introduction

The `UnitCreator` class is responsible for creating and managing units in the database, as well as linking units to products and prices. This class provides safe functions that use transactions and handle errors in an organized manner. It relies on models such as `Unit`, `ProductsUnit`, and `ProductsPricesUnit` to perform operations.

The goal of this class is to provide a unified programming interface for creating units (whether from complete data or from standard symbols), linking them to products, and creating associated prices, while ensuring data integrity and avoiding duplication.

---

## Internal Helper Functions

### `getAndCheckDepartmentsId`

```php
public static function getAndCheckDepartmentsId($departments_id = null)
```

Description:
Validate the department ID and return the correct value based on system settings (e.g., using the default company, main/sub department type). Used internally to unify the logic for obtaining the department.

Parameters:

· $departments_id : The input department ID (optional).

Returns:
The correct department ID (integer).

---

getAndCheckCompanysId

```php
public static function getAndCheckCompanysId($departments_id = null, $companys_id = null)
```

Description:
Validate the company ID and return the correct value. If departments_id is passed, the associated company is extracted. Otherwise, it falls back to default settings.

Parameters:

· $departments_id : The department ID (optional).
· $companys_id : The input company ID (optional).

Returns:
The correct company ID (integer).

---

Main Functions

createUnit

```php
public static function createUnit(array $options = [], bool $is_test_create = false): array
```

Description:
Create a new unit in the database. The function validates the input, ensures no duplicate unit exists, and then creates the record in the units table with transaction support.

Parameters:

· $options : An array containing unit data. Key options include:
  · name (string) : Unit name (required).
  · unit_symbol (string) : Unit symbol (e.g., 'kg', 'm').
  · type (string) : Unit type (one of the UnitType constants). Defaults to UnitType::QUANTITY_GENERAL.
  · companys_id (int) : Company ID.
  · departments_id (int) : Department ID.
  · main_sub (string) : 'main' or 'sub' (default 'main').
  · parent_id (int) : Parent ID (if the unit is a sub-unit).
  · conversion_factor (float) : Conversion factor (default 1.0).
  · value (float) : Unit value (usually the same as conversion_factor).
  · is_active (bool) : Activation status (default true).
  · is_default (bool) : Is it the default unit? (default false).
  · is_force_create (bool) : If true, bypasses the duplication error (creates a duplicate unit).
  · status (string) : Status (default 'active').
  · date_at (Carbon) : Creation date (default now).
  · And other fields present in the Unit model.
· $is_test_create : If true, the operation is performed within a transaction and then rolled back (for testing purposes).

Returns:
An array containing:

· status (bool) : Success or failure of the operation.
· message (string) : Explanatory message.
· error (string|null) : Error text if any.
· errors (array|null) : Error details (if any).
· model (object|null) : The created unit object (Unit model).
· data (array) : Unit data (toArray() copy).
· input_data (array) : A copy of the input options.
· process_data (array) : Additional information about the operation.
· code (int) : Status code (200 for success, 400 for error).

Example:

```php
$result = UnitCreator::createUnit([
    'name' => 'Kilogram',
    'unit_symbol' => 'kg',
    'type' => UnitType::WEIGHT,
    'is_base_unit' => true,
    'conversion_factor' => 1,
    'companys_id' => 1,
    'departments_id' => 2,
]);

if ($result['status']) {
    $unit = $result['model'];
    echo "Unit created: " . $unit->name;
} else {
    echo "Error: " . $result['error'];
}
```

---

createUnitFromSymbol

```php
public static function createUnitFromSymbol(array $options = [], bool $is_test_create = false): array
```

Description:
Create a unit from a known symbol (e.g., 'kg', 'm', 'dozen'). The function queries unit information using UnitNameHelper and fills the data automatically (name, type, short description). Then it calls createUnit to create the unit.

Parameters:

· $options : An array containing:
  · symbol (string) : Unit symbol (required).
  · make_default (bool) : Whether to make the unit default? (default false).
  · companys_id (int) : Company ID.
  · departments_id (int) : Department ID.
  · additional_data (array) : Additional data that can be merged with the unit data (e.g., main_sub, parent_id, ...).
· $is_test_create : For testing.

Returns:
Same structure as createUnit.

Example:

```php
$result = UnitCreator::createUnitFromSymbol([
    'symbol' => 'dozen',
    'make_default' => true,
    'companys_id' => 1,
    'departments_id' => 2,
    'additional_data' => [
        'is_base_unit' => true,
    ],
]);

if ($result['status']) {
    echo "Dozen unit created: " . $result['model']->name;
}
```

---

createStandardUnitsForType

```php
public static function createStandardUnitsForType(array $options = [], bool $is_test_create = false): array
```

Description:
Create all standard units for a specific type (e.g., all length units). Uses UnitConverter::getUnitsByType to get the list of symbols, then creates each unit using createUnitFromSymbol. Returns an array containing the created units and errors.

Parameters:

· $options : An array containing:
  · type (string) : Unit type (required, from UnitType constants).
  · additional_data (array) : Additional data passed to each unit.
  · companys_id (int) : Company ID.
  · departments_id (int) : Department ID.
· $is_test_create : For testing.

Returns:
An array containing:

· status, message, error, errors as usual.
· data (array) : List of created units (Unit objects).
· models (array) : Same as data.
· process_data : Contains created_count and errors (error details per symbol).

Example:

```php
$result = UnitCreator::createStandardUnitsForType([
    'type' => UnitType::LENGTH,
    'companys_id' => 1,
    'departments_id' => 2,
]);

echo "Created " . $result['process_data']['created_count'] . " units.";
if (!empty($result['process_data']['errors'])) {
    print_r($result['process_data']['errors']);
}
```

---

createAllStandardUnits

```php
public static function createAllStandardUnits(array $options = [], bool $is_test_create = false): array
```

Description:
Create all standard units for all main types. Calls createStandardUnitsForType for each type in UnitType::getMainTypes().

Parameters:

· $options : An array containing:
  · additional_data (array) : Additional data.
  · companys_id (int) : Company ID.
  · departments_id (int) : Department ID.
· $is_test_create : For testing.

Returns:
An array containing:

· data (array) : Associative array [type => [created units]].
· process_data : Contains created_count and errors_count.

Example:

```php
$result = UnitCreator::createAllStandardUnits([
    'companys_id' => 1,
    'departments_id' => 2,
]);

echo "Total created units: " . $result['process_data']['created_count'];
```

---

attachUnitToProduct

```php
public static function attachUnitToProduct(array $options = [], bool $is_test_create = false): array
```

Description:
Attach a unit to a product (create a record in the products_units table). Checks for duplication, and ensures no duplicate main unit if requested.

Parameters:

· $options : An array containing:
  · products_id (int) : Product ID (required).
  · units_id (int) : Unit ID (required).
  · is_main (bool) : Is this the main unit for the product? (default false).
  · conversion_factor (float) : Conversion factor (default 1.0).
  · value (float) : Unit value (default 1.0).
  · barcode (string) : Barcode (optional).
  · is_active (bool) : Activation status (default true).
  · is_force_create (bool) : Bypass duplication error.
  · type (string) : Operation type ('add' or 'edit').

Returns:
An array containing model (ProductsUnit object) and data.

Example:

```php
$result = UnitCreator::attachUnitToProduct([
    'products_id' => 123,
    'units_id' => 5,
    'is_main' => true,
    'conversion_factor' => 1,
]);

if ($result['status']) {
    echo "Unit attached to product.";
}
```

---

createProductUnitPrice

```php
public static function createProductUnitPrice(array $options = [], bool $is_test_create = false): array
```

Description:
Create a price for a product unit (in the products_prices_units table). If the unit is not yet attached to the product, it can be attached automatically if is_allow_add_unit = true.

Parameters:

· $options : An array containing:
  · products_id (int) : Product ID (required).
  · units_id (int) : Unit ID (required).
  · prices_id (int) : Price list ID (required).
  · currencys_id (int) : Currency ID (if not given, tries to extract it from the product or default currency).
  · value (float) : Selling price.
  · old_price (float) : Old price.
  · value_purchas (float) : Purchase price.
  · costed_amount (float) : Cost.
  · manage_stock (bool) : Manage stock.
  · shop_stock (float) : Available quantity.
  · is_show_old_price (bool) : Show old price.
  · is_parleying (bool) : Negotiable.
  · is_default (bool) : Default price.
  · is_active (bool) : Active.
  · process_type (string) : 'add', 'edit', or 'add_or_edit'.
  · And others.

Returns:
An array containing model (ProductsPricesUnit object).

Example:

```php
$result = UnitCreator::createProductUnitPrice([
    'products_id' => 123,
    'units_id' => 5,
    'prices_id' => 1,
    'value' => 100.50,
    'currencys_id' => 1,
    'is_default' => true,
]);

if ($result['status']) {
    echo "Price created.";
}
```

---

createProductUnitsStructure

```php
public static function createProductUnitsStructure(array $options = [], bool $is_test_create = false): array
```

Description:
Create a complete structure of units and prices for a specific product. Accepts an array of unit data and price data and creates everything in one batch.

Parameters:

· $options : An array containing:
  · products_id (int) : Product ID.
  · units_data (array) : An array of unit data. Each element can contain units_id or unit_symbol, plus is_main, conversion_factor, value, barcode.
  · prices_data (array) : An array of price data linked to the same order as units_data. Each element is an array of prices for that unit.
  · companys_id, departments_id optional.

Returns:
An array containing:

· data : ['units' => [...], 'prices' => [...]]
· errors : List of errors that occurred during the operation.

Example:

```php
$result = UnitCreator::createProductUnitsStructure([
    'products_id' => 123,
    'units_data' => [
        ['unit_symbol' => 'kg', 'is_main' => true],
        ['unit_symbol' => 'g', 'conversion_factor' => 0.001],
    ],
    'prices_data' => [
        [ // Prices for the first unit
            ['prices_id' => 1, 'value' => 10],
        ],
        [ // Prices for the second unit
            ['prices_id' => 1, 'value' => 0.01],
        ],
    ],
]);
```

---

createUnitConversion

```php
public static function createUnitConversion(array $options = [], bool $is_test_create = false): array
```

Description:
Create a conversion relationship between two units (for future use). Checks that both units are of the same type, then links the target unit to the product.

Parameters:

· $options : An array containing:
  · products_id (int) : Product ID.
  · from_units_id (int) : Source unit ID.
  · to_units_id (int) : Target unit ID.
  · conversion_factor (float) : Conversion factor.

Returns:
An array containing the attachment result.

---

importUnitsFromSymbols

```php
public static function importUnitsFromSymbols(array $options = [], bool $is_test_create = false): array
```

Description:
Import units from a list of symbols. Checks for existing units and creates only new ones.

Parameters:

· $options : An array containing:
  · unit_symbols (array) : List of unit symbols.
  · common_data (array) : Common data added to each unit.
  · companys_id, departments_id.

Returns:
An array containing:

· data : ['created' => [...], 'skipped' => [...], 'errors' => [...]]

Example:

```php
$result = UnitCreator::importUnitsFromSymbols([
    'unit_symbols' => ['kg', 'g', 'lb'],
    'common_data' => ['is_active' => true],
    'companys_id' => 1,
]);
```

---

createStandardQuantityUnits

```php
public static function createStandardQuantityUnits(array $options = [], bool $is_test_create = false): array
```

Description:
Create standard quantity units (e.g., 'pcs', 'dozen', 'box', ...). Uses a predefined list.

Parameters:

· $options : Contains common_data, companys_id, departments_id.

Returns:
An array containing the created units.

---

firstOrCreateFromSymbol

```php
public static function firstOrCreateFromSymbol(array $options = [], bool $is_test_create = false): array
```

Description:
Find a unit by a given symbol, and create it if it doesn't exist.

Parameters:

· $options : Contains symbol, additional_data, companys_id, departments_id.

Returns:
An array containing model (the existing or new unit) and process_data['is_existing'] indicating whether it existed before.

Example:

```php
$result = UnitCreator::firstOrCreateFromSymbol([
    'symbol' => 'kg',
    'companys_id' => 1,
]);

if ($result['process_data']['is_existing']) {
    echo "Unit already exists.";
} else {
    echo "Unit created.";
}
```

---

Result Array Structure

Most UnitCreator functions return an array with the following structure:

```php
[
    'code' => 200,                     // Status code
    'status' => true,                   // Success/failure
    'message' => 'Explanatory message', // General message
    'error' => null,                     // Error text (if any)
    'errors' => null,                    // Error details (array)
    'data' => null,                      // Requested data (could be an array or object)
    'model' => null,                      // Model object (if any)
    'input_data' => [],                   // Copy of the inputs
    'process_data' => [],                 // Additional information about the operation
    'debug' => []                         // Debug information (in debug mode)
]
```

---

Comprehensive Examples

Example 1: Creating a New Unit, Linking it to a Product, and Adding a Price

```php
use Tss\Inventory\Classes\Units\UnitCreator;

// 1. Create the unit
$unitResult = UnitCreator::createUnitFromSymbol([
    'symbol' => 'dozen',
    'make_default' => true,
    'companys_id' => 1,
    'departments_id' => 2,
]);

if (!$unitResult['status']) {
    throw new Exception($unitResult['error']);
}

$unit = $unitResult['model'];

// 2. Attach the unit to the product (product ID 123)
$attachResult = UnitCreator::attachUnitToProduct([
    'products_id' => 123,
    'units_id' => $unit->id,
    'is_main' => true,
    'conversion_factor' => 12,
]);

if (!$attachResult['status']) {
    throw new Exception($attachResult['error']);
}

// 3. Create a price for the unit
$priceResult = UnitCreator::createProductUnitPrice([
    'products_id' => 123,
    'units_id' => $unit->id,
    'prices_id' => 1,
    'value' => 50.00,
    'currencys_id' => 1,
    'is_default' => true,
]);

if ($priceResult['status']) {
    echo "Unit created, attached to product, and price added successfully.";
}
```

Example 2: Importing Units from a List of Symbols

```php
$symbols = ['kg', 'g', 'lb', 'oz'];
$result = UnitCreator::importUnitsFromSymbols([
    'unit_symbols' => $symbols,
    'common_data' => [
        'is_active' => true,
        'is_base_unit' => false,
    ],
    'companys_id' => 1,
    'departments_id' => 2,
]);

echo "Created: " . count($result['data']['created']) . "\n";
echo "Skipped (already exist): " . count($result['data']['skipped']) . "\n";
echo "Errors: " . count($result['data']['errors']) . "\n";
```

Example 3: Creating All Length and Weight Units

```php
foreach ([UnitType::LENGTH, UnitType::WEIGHT] as $type) {
    $result = UnitCreator::createStandardUnitsForType([
        'type' => $type,
        'companys_id' => 1,
        'departments_id' => 2,
    ]);
    echo "Type $type: created {$result['process_data']['created_count']} units.\n";
}
```

Example 4: Using firstOrCreateFromSymbol

```php
$result = UnitCreator::firstOrCreateFromSymbol([
    'symbol' => 'kg',
    'additional_data' => ['is_base_unit' => true],
    'companys_id' => 1,
    'departments_id' => 2,
]);

$unit = $result['model'];
if ($result['process_data']['is_existing']) {
    echo "Unit found: " . $unit->name;
} else {
    echo "Unit created: " . $unit->name;
}
```

---

This covers all functions of the UnitCreator class with detailed explanations and practical examples. This class is the main tool for creating and managing units in the database, and it is an essential part of the unit management system.