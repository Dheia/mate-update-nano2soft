# Comprehensive Unit Management and Conversion System Documentation

## 📋 Table of Contents

1.  [General Introduction](#general-introduction)
2.  [Architecture Overview](#architecture-overview)
3.  [Main Classes](#main-classes)
    - [UnitType - Unit Type Definition](#unittype---unit-type-definition)
    - [UnitNameHelper - Unit Name Helper](#unitnamehelper---unit-name-helper)
    - [UnitConverter - Unit Converter](#unitconverter---unit-converter)
    - [UnitHelper - Unit Helper](#unithelper---unit-helper)
    - [UnitContextValidator - Unit Context Validator](#unitcontextvalidator---unit-context-validator)
    - [SmartUnitConverter - Smart Converter](#smartunitconverter---smart-converter)
    - [UnitsInformation - Units Information](#unitsinformation---units-information)
    - [ConversionForm - Conversion Form](#conversionform---conversion-form)
    - [UnitFactory - Unit Factory](#unitfactory---unit-factory)
    - [Unit - Unit Object](#unit---unit-object)
    - [UnitCreator - Database Unit Creator](#unitcreator---database-unit-creator)
    - [UnitManager - Unified Unit Manager](#unitmanager---unified-unit-manager)
4.  [Updated Models](#updated-models)
5.  [Detailed Explanation of Models and Traits](#detailed-explanation-of-models-and-traits)
    - [Unit Model](#unit-model)
    - [ProductsUnit Model](#productsunit-model)
    - [ProductsPricesUnit Model](#productspricesunit-model)
6.  [Caching Mechanisms](#caching-mechanisms)
7.  [The `getRecords` Function in Detail](#the-getrecords-function-in-detail)
8.  [Available Scopes in `Unit`](#available-scopes-in-unit)
9.  [Configuration (Config) Affecting Behavior](#configuration-config-affecting-behavior)
10. [Localization](#localization)
11. [Unique Code Generation Functions in `UnitManager`](#unique-code-generation-functions-in-unitmanager)
12. [Printing Reports (`printReports`)](#printing-reports-printreports)
13. [Performance Notes and Query Optimization](#performance-notes-and-query-optimization)
14. [Upgrade Guide](#upgrade-guide)
15. [Frequently Asked Questions (FAQ)](#frequently-asked-questions-faq)
16. [Troubleshooting](#troubleshooting)
17. [Integrated Examples](#integrated-examples)
18. [Best Practices and Error Handling](#best-practices-and-error-handling)
19. [Conclusion](#conclusion)

---

## General Introduction

The Comprehensive Unit Management and Conversion System is an integrated PHP package within `Tss.Inventory` aimed at providing a complete and flexible solution for handling all types of standard and commercial units. The system supports over 16 main unit types (length, weight, volume, area, temperature, time, speed, pressure, energy, power, data, angles, quantities with various contexts) and provides tools for conversion, searching, validation, and linking with products.

The system has been completely restructured to include:
- **Helper Classes** for handling unit names and conversions.
- **Context Validator** to prevent illogical conversions between different quantity units.
- **Unit Factory** to create unit objects.
- **Unit Creator** to create units in the database and link them to products.
- **Unified Manager** that provides a simplified programming interface for all operations.

The goal is to enable developers to:
- Convert values between units with high precision.
- Obtain complete information about any unit.
- Create and manage units in the database.
- Link units to products and prices.
- Verify the logicality of conversions in practical applications.

---

## Architecture Overview

```
Tss\Inventory\Classes\
│
├── UnitManager.php                      # Unified Unit Manager (main programming interface)
│
└── Units\                                # Conversion system classes
    ├── UnitType.php                      # Unit type definitions
    ├── UnitNameHelper.php                 # Unit names and classifications
    ├── UnitConverter.php                  # Basic unit converter
    ├── UnitHelper.php                      # General helper functions
    ├── UnitContextValidator.php            # Quantity context validator
    ├── SmartUnitConverter.php              # Smart converter with warnings
    ├── UnitsInformation.php                # Display unit information (HTML)
    ├── ConversionForm.php                  # Conversion forms (for demonstration)
    ├── UnitFactory.php                      # Unit object factory
    ├── Unit.php                             # Unit object
    └── UnitCreator.php                      # Create units in the database
```

---

## Main Classes

### UnitType - Unit Type Definition

A static class containing constants for defining various unit types, along with helper functions for validation and translation.

#### Basic Constants

```php
UnitType::LENGTH;          // 'length'   - Length
UnitType::WEIGHT;          // 'weight'   - Weight
UnitType::VOLUME;          // 'volume'   - Volume
UnitType::AREA;            // 'area'     - Area
UnitType::TEMPERATURE;     // 'temperature' - Temperature
UnitType::TIME;            // 'time'     - Time
UnitType::SPEED;           // 'speed'    - Speed
UnitType::PRESSURE;        // 'pressure' - Pressure
UnitType::ENERGY;          // 'energy'   - Energy
UnitType::POWER;           // 'power'    - Power
UnitType::DATA;            // 'data'     - Data
UnitType::ANGLE;           // 'angle'    - Angles
UnitType::QUANTITY;        // 'quantity' - Quantities (main type)

// Quantity subtypes
UnitType::QUANTITY_GENERAL;    // 'quantity_general'   - General quantities
UnitType::QUANTITY_PACKAGING;  // 'quantity_packaging' - Packaging quantities
UnitType::QUANTITY_PAPER;      // 'quantity_paper'     - Paper quantities
UnitType::QUANTITY_PAIRS;      // 'quantity_pairs'     - Pair quantities
UnitType::QUANTITY_LARGE;      // 'quantity_large'     - Large quantities
UnitType::CUSTOM;              // 'custom'             - Custom units
```

#### Main Functions

| Function | Description |
|--------|-------|
| `getMainTypesWithLabels($lang)` | Returns main types with their labels (`['length'=>'Length', ...]`) |
| `getAllTypesWithLabels($lang)` | Returns all types (including subtypes) with their labels |
| `getQuantitySubtypesWithLabels($lang)` | Returns only quantity subtypes with labels |
| `getTransLabel($type, $lang)` | Get the translated label for a specific type |
| `isValid($type)` | Check if the type is defined |
| `isQuantitySubtype($type)` | Check if the type is a quantity subtype |
| `isCustomType($type)` | Check if the type is custom |
| `getMainType($type)` | Return the main type (for subtypes) |
| `getAllTypes()` | Return a list of all types |
| `getMainTypes()` | Return a list of main types only |
| `getSelectOptions($selected, $includeEmpty, $onlyMainTypes)` | Generate HTML `<select>` options for types |

#### Examples

```php
// Get type label
$label = UnitType::getTransLabel('length', 'en'); // 'Length'

// Check type
if (UnitType::isValid('weight')) { ... }

// Check subtype
if (UnitType::isQuantitySubtype('quantity_packaging')) { ... }

// Generate select options
echo UnitType::getSelectOptions('length', true, false);
```

---

### UnitNameHelper - Unit Name Helper

This class provides centralized access to unit information: their names, symbols, types, contexts, and searching.

#### Main Functions

| Function | Description |
|--------|-------|
| `getAllUnitsWithLabels($lang)` | Returns all units with their labels (`['mm'=>'Millimeter', ...]`) |
| `getUnitsByTypeWithLabels($type, $lang, $isBaseUnit)` | Returns units of a specific type, with optional filtering for base units only |
| `getUnitLabel($unit, $lang)` | Get the label for a specific unit |
| `getEnglishLabel($unit)` | Get the English label for the unit |
| `getCategorizedUnits($lang)` | Returns units categorized by type (for structuring lists) |
| `getSelectOptions($type, $selected, $includeEmpty, $lang)` | Generate HTML `<select>` options for units of a specific type |
| `getGroupedSelectOptions($selected, $includeEmpty, $lang)` | Generate `<select>` options grouped by type |
| `searchUnits($searchText, $withLabels, $lang)` | Simple search for units containing the text |
| `searchUnitAdvanced($searchText, $type, $lang)` | Advanced search returning information of the first matching unit (by symbol, Arabic name, English name) |
| `getUnitType($unit)` | Return the unit type |
| `isValidUnit($unit)` | Validate unit symbol |
| `getSimilarUnits($unit, $lang)` | Return units of the same type |
| `getBaseUnits($lang)` | Return base units for each type (`['length'=>['unit'=>'m','label'=>'Meter']]`) |
| `getUnitInfo($unit, $lang)` | Return basic information about the unit (symbol, label, type, ...) |
| `getUnitFullInfo($unit, $lang)` | Return complete information about the unit (including `unit_id`, `unit_name_ar`, `unit_name_en`, `type_id`, `type_name_ar`, `type_name_en`, `is_base_unit`, `base_unit`, `context`, `is_metric`, `is_imperial`, `display_name`, `full_name`) |
| `isBaseUnitForItsType($unit)` | Check if the unit is base for its type |
| `getBaseUnitByType($type)` | Return the base unit symbol for a given type |

#### Examples

```php
// Get unit label
echo UnitNameHelper::getUnitLabel('kg', 'en'); // "Kilogram"

// Advanced unit search
$info = UnitNameHelper::searchUnitAdvanced('kilo');
if ($info) {
    echo $info['unit_name_en']; // "Kilogram"
}

// Get full information
$full = UnitNameHelper::getUnitFullInfo('dozen', 'en');
/*
[
    'unit_id' => 'dozen',
    'unit_name_ar' => 'درزن',
    'unit_name_en' => 'Dozen',
    'type_id' => 'quantity_packaging',
    'type_name_ar' => 'كميات التعبئة',
    'is_base_unit' => true,
    'base_unit' => 'dozen',
    'context' => 'packaging',
    ...
]
*/

// Generate grouped select options
echo UnitNameHelper::getGroupedSelectOptions('kg', true, 'en');
```

---

### UnitConverter - Unit Converter

The heart of the conversion system, containing conversion data for all units and performing mathematical conversions.

#### Main Functions

| Function | Description |
|--------|-------|
| `convert($value, $from, $to, $precision, $strictContext)` | Convert a value from one unit to another. `$strictContext` enables strict context checking for quantity units. |
| `convertBatch(array $values, $from, $to, $precision)` | Convert a batch of values at once. |
| `describeConversion($value, $from, $to, $precision)` | Return a textual description of the conversion (e.g., "1 meter = 100 centimeters"). |
| `isUnitSupported($unit)` | Check if the unit is supported. |
| `getUnitType($unit)` | Return the unit type. |
| `getArabicLabel($unit)` | Return the Arabic label (shortcut for `UnitNameHelper::getUnitLabel`). |
| `getUnitsByType($type, $withLabels, $lang)` | Return units of a specific type (with or without labels). |
| `getAllUnitsGrouped($groupByMainType, $lang)` | Return all units grouped by main type or full type. |
| `getUnitInfo($unit, $lang)` | Return basic information about the unit (including conversion factor). |
| `canConvert($from, $to)` | Check if conversion between two units is possible (without exception). |
| `getSupportedUnits($type, $includeSubtypes)` | Return a list of all supported unit symbols, or for a specific type including subtypes. |
| `getSupportedUnitsWithLabels($type, $includeSubtypes, $lang)` | Same as above but with labels. |
| `getSupportedConversionTypes($includeSubtypes)` | Return a list of supported conversion types. |
| `getSupportedConversionTypesWithLabels($includeSubtypes, $lang)` | Same as above with labels. |
| `isTypeSupported($type)` | Check if the unit type is supported. |
| `getConversionFactor($unit, $type)` | Return the conversion factor for the unit relative to the base unit of the type. |

#### Examples

```php
// Simple conversion
$result = UnitConverter::convert(5, 'kg', 'lb', 4); // 11.0231

// Temperature conversion
$result = UnitConverter::convert(100, 'c', 'f', 2); // 212.00

// Quantity unit conversion
$result = UnitConverter::convert(1, 'dozen', 'pcs'); // 12

// Check conversion possibility
if (UnitConverter::canConvert('dozen', 'box')) { ... }

// Get all weight units with labels
$weightUnits = UnitConverter::getSupportedUnitsWithLabels('weight', false, 'en');
```

---

### UnitHelper - Unit Helper

General helper functions to facilitate working with units in applications.

#### Main Functions

| Function | Description |
|--------|-------|
| `safeConvert($value, $from, $to, $precision, $validateLogic)` | Safe conversion with error handling and optional logical validation (returns `null` on failure). |
| `formatWithUnit($value, $unit, $decimals, $lang)` | Format a value with the unit label (e.g., "100.00 Kilogram"). |
| `getBestDisplayUnit($value, $currentUnit, $preferredUnits, $lang)` | Find the best unit to display a value (so that the value is between 1 and 1000). |
| `generateSelectOptions($type, $selected, $includeEmpty, $lang)` | Generate `<select>` options for units of a specific type. |
| `generateGroupedSelectOptions($selected, $includeEmpty, $lang)` | Generate `<select>` options grouped by type. |
| `getRentalTimeUnits($lang, $is_key)` | Return time units used in rentals (hour, day, week, month). |
| `getConversionFactor($unit, $defaultValue)` | Return the conversion factor for a unit (with a default value in case of error). |
| `getConversionFactorFormatted($unit, $defaultValue, $decimals, $isFormatDecimal)` | Return the conversion factor formatted (without trailing zeros). |
| `formatDecimal($value, $decimals)` | Format a decimal number without trailing zeros. |

#### Examples

```php
// Safe conversion
$result = UnitHelper::safeConvert(1, 'dozen', 'box', 2, true); // 0.50

// Format value
echo UnitHelper::formatWithUnit(2.5, 'kg', 3, 'en'); // "2.500 Kilogram"

// Get best display unit
$best = UnitHelper::getBestDisplayUnit(1500, 'm', ['km','m'], 'en'); // 'km'

// Get rental units
$rentalUnits = UnitHelper::getRentalTimeUnits('en'); // ['h'=>'Hour', 'day'=>'Day', ...]

// Format decimal number
echo UnitHelper::formatDecimal(12.3000, 4); // "12.3"
```

---

### UnitContextValidator - Unit Context Validator

Responsible for verifying the logicality of converting quantity units based on their context (packaging, paper, pairs, large quantities).

#### Main Functions

| Function | Description |
|--------|-------|
| `isConversionLogical($from, $to)` | Check if a conversion between two units is logical (within the same context or via `pcs`). |
| `getConversionReason($from, $to)` | Return the reason for logicality or lack thereof (text). |
| `getCompatibleUnits($unit)` | Return a list of logically compatible units with a given unit (each is an array containing `unit` and `label`). |

#### Rules

- `pcs` (piece) is a universal unit that can be converted from and to any quantity unit.
- Specific rules defined in `CONVERSION_RULES` determine logical relationships between units (e.g., `dozen` ↔ `box`).
- If no rule exists, conversion is allowed if the context matches.

#### Examples

```php
// Check logicality
if (UnitContextValidator::isConversionLogical('dozen', 'box')) {
    // Allowed (both are packaging)
}

if (!UnitContextValidator::isConversionLogical('dozen', 'ream')) {
    // Illogical (packaging to paper)
}

// Get compatible units
$compatible = UnitContextValidator::getCompatibleUnits('dozen');
```

---

### SmartUnitConverter - Smart Converter

Provides conversions with warnings and strict logical validation.

#### Main Functions

| Function | Description |
|--------|-------|
| `safeConvert($value, $from, $to, $precision)` | Safe conversion with logical validation; throws an exception if the conversion is illogical. |
| `convertWithWarning($value, $from, $to, $precision)` | Conversion returning an array containing the value, `is_logical` flag, and a warning if not logical. |

#### Examples

```php
try {
    $result = SmartUnitConverter::safeConvert(1, 'dozen', 'box', 2);
    // Result 0.50
} catch (InvalidArgumentException $e) {
    echo $e->getMessage();
}

// Convert with warning
$result = SmartUnitConverter::convertWithWarning(1, 'dozen', 'ream', 2);
if (!$result['is_logical']) {
    echo $result['warning']; // Warning: conversion is illogical
}
```

---

### UnitsInformation - Units Information

A class to display unit information in formatted HTML (for documentation and demonstration purposes).

#### Main Functions

| Function | Description |
|--------|-------|
| `displayAllUnitsInfo()` | Display all units in a full HTML page with statistics and cards (using Bootstrap). |
| `displayTypeInfo($typeKey)` | Display units of a specific type only. |
| `displayAllUnitsInfoBootstrap()` | Same as `displayAllUnitsInfo` (alias). |
| `displayTypeInfoBootstrap($typeKey)` | Same as `displayTypeInfo`. |

#### Example

```php
echo UnitsInformation::displayAllUnitsInfo();
```

---

### ConversionForm - Conversion Form

Creates interactive HTML forms for testing unit conversions.

#### Main Functions

| Function | Description |
|--------|-------|
| `renderAdvancedForm()` | Advanced form with options (conversion type, units, value, strict mode, precision). |
| `renderForm()` | Simple form. |
| `testQuantityConversions()` | Test function to try quantity conversions (prints results to the command line). |

#### Example

```php
$form = new ConversionForm();
echo $form->renderAdvancedForm();
```

---

### UnitFactory - Unit Factory

Creates `Unit` objects from unit symbols or types.

#### Main Functions

| Function | Description |
|--------|-------|
| `createFromString($unitString)` | Create a `Unit` object from a unit symbol (e.g., 'm'). |
| `createUnitsByType($type)` | Create an array of `Unit` objects for a specific type. |
| `createAllUnits()` | Create an array of `Unit` objects for all units. |
| `createBaseUnits()` | Create an array of `Unit` objects for base units of each type (associated with the type). |
| `createUnitByContext($context)` | Create a base unit for a specific context (e.g., `packaging` → `dozen`). |
| `createCompatibleUnits($unitSymbol)` | Create an array of `Unit` objects compatible with a given unit. |

#### Examples

```php
$meter = UnitFactory::createFromString('m');
echo $meter->convertTo(100, 'cm'); // 10000

$lengthUnits = UnitFactory::createUnitsByType('length');
foreach ($lengthUnits as $unit) {
    echo $unit->symbol . ' - ' . $unit->label . "\n";
}
```

---

### Unit - Unit Object

Represents a single unit of measurement with its properties and operations.

#### Properties

| Property | Description |
|---------|-------|
| `symbol` | Unit symbol (e.g., 'kg') |
| `type` | Unit type (e.g., 'weight') |
| `label` | Arabic label (e.g., 'كيلوجرام') |
| `context` | Context (for quantity units) |

#### Main Functions

| Function | Description |
|--------|-------|
| `convertTo($value, $toUnit, $precision)` | Convert a value to another unit. |
| `convertToBase($value, $precision)` | Convert to the base unit of the type. |
| `isCompatibleWith($otherUnit)` | Check compatibility with another unit (same main type). |
| `getConversionFactor()` | Return the conversion factor for the unit. |
| `isBaseUnit()` | Check if the unit is base. |
| `getEnglishLabel()` | Return the English label. |
| `getInfo()` | Return complete information about the unit (using `UnitNameHelper`). |
| `getContext()` | Return the context. |
| `getCompatibleUnits()` | Return compatible units (of the same type). |
| `canConvertTo($toUnit)` | Check if conversion to another unit is possible. |
| `formatValue($value, $decimals)` | Format a value with the unit (e.g., "2.50 Kilogram"). |
| `__toString()` | String representation of the unit (e.g., "kg (Kilogram)"). |
| `equals(Unit $other)` | Compare with another unit. |
| `copy()` | Create a new copy. |

#### Examples

```php
$kg = UnitFactory::createFromString('kg');
echo $kg->convertTo(5, 'g'); // 5000
echo $kg->formatValue(2.5, 2); // "2.50 Kilogram"
if ($kg->canConvertTo('lb')) { ... }
```

---

### UnitCreator - Database Unit Creator

A specialized class for creating and updating units and linking them to products and prices in the database.

#### Main Functions

| Function | Description |
|--------|-------|
| `createUnit(array $options, $is_test_create)` | Create a new unit in the database. |
| `createUnitFromSymbol(array $options, $is_test_create)` | Create a unit from a known symbol (e.g., 'kg') with automatic data completion. |
| `createStandardUnitsForType(array $options, $is_test_create)` | Create all standard units for a specific type (e.g., all length units). |
| `createAllStandardUnits(array $options, $is_test_create)` | Create all standard units for all types. |
| `attachUnitToProduct(array $options, $is_test_create)` | Attach a unit to a product (create a record in `products_units`). |
| `createProductUnitPrice(array $options, $is_test_create)` | Create a price for a product unit (in `products_prices_units`). |
| `createProductUnitsStructure(array $options, $is_test_create)` | Create a complete structure of units and prices for a specific product. |
| `createUnitConversion(array $options, $is_test_create)` | Create a conversion relationship between two units (for future use). |
| `importUnitsFromSymbols(array $options, $is_test_create)` | Import units from a list of symbols. |
| `createStandardQuantityUnits(array $options, $is_test_create)` | Create standard quantity units (dozen, ream, pair, ...). |
| `firstOrCreateFromSymbol(array $options, $is_test_create)` | Find a unit by a given symbol, and create it if it doesn't exist. |

#### Common `$options` Structure

- `companys_id`, `departments_id`: To specify the company and department.
- `name`: Unit name.
- `unit_symbol`: Unit symbol.
- `type`: Unit type (from `UnitType`).
- `conversion_factor`, `value`: Conversion factor (usually the same value).
- `is_base_unit`: Is it the base unit?
- `is_default`: Is it the default?
- `is_active`: Activation status.

#### Examples

```php
// Create a new unit
$result = UnitCreator::createUnit([
    'name' => 'Dozen',
    'unit_symbol' => 'dozen',
    'type' => UnitType::QUANTITY_PACKAGING,
    'is_base_unit' => true,
    'conversion_factor' => 12,
]);

if ($result['status']) {
    $unit = $result['model'];
}

// Attach a unit to a product
$result = UnitCreator::attachUnitToProduct([
    'products_id' => 123,
    'units_id' => 5,
    'is_main' => true,
    'conversion_factor' => 1,
]);

// Create a price for a product unit
$result = UnitCreator::createProductUnitPrice([
    'products_id' => 123,
    'units_id' => 5,
    'prices_id' => 1,
    'value' => 100.50,
    'currencys_id' => 1,
]);
```

---

### UnitManager - Unified Unit Manager

A unified programming interface that combines most system functions and provides easy-to-use static functions.

#### Main Functions

| Function | Description |
|--------|-------|
| `createUnit($options, $is_test_create)` | Create a new unit (relies on `UnitCreator::createUnit`). |
| `checkUnit($options)` | Validate a unit (its symbol, balance, user) - used for validating recharge cards, for example. |
| `getQueryDate($records, $options)` | Apply date conditions to a query (helper function). |
| `scopeWhereField($query, $field, $value, $is_or, $table, $is_not, $is_force)` | Helper scope for applying conditions on fields with support for `*` and `all`. |
| `printReports($options)` | Print unit reports (using the reporting system). |
| `getAllUnitsGrouped($groupByMainType, $lang)` | Get all units categorized (passes to `UnitConverter::getAllUnitsGrouped`). |
| `getUnitsForType($type, $includeLabels, $lang)` | Get units for a specific type (passes to `UnitConverter::getUnitsForType`). |
| `getRentalTimeUnits($lang, $is_key)` | Get time units used in rentals. |
| `formatDecimal($value, $decimals)` | Format a decimal number without trailing zeros. |
| `generateDigital($length)` | Generate a random number of a given length (for card codes). |
| `getGuid()` | Generate a random UUID. |
| `getVoucherCode()` | Generate a 13-digit voucher code. |
| `humanUuid($options)` | Generate a human-friendly UUID (format `YYYYMMDD-HHMM-SSMM-MMMM...`). |
| `nanoUid($options)` | Generate a unique nanosecond-based identifier. |

#### Examples

```php
use Tss\Inventory\Classes\UnitManager;

// Create a unit
$result = UnitManager::createUnit([
    'name' => 'Kilogram',
    'unit_symbol' => 'kg',
    'type' => UnitType::WEIGHT,
    'is_base_unit' => true,
]);

// Check a recharge card
$check = UnitManager::checkUnit(['unit_symbol' => 'CARD123']);

// Get rental time units
$rentalUnits = UnitManager::getRentalTimeUnits('en');

// Print a report
UnitManager::printReports($options);

// Format a number
echo UnitManager::formatDecimal(12.3000, 4); // "12.3"
```

---

## Updated Models

### `Tss\Inventory\Models\Unit`

New columns added to support the system:

- `unit_symbol` (string, index)
- `conversion_factor` (decimal, 20,10)
- `value` (decimal, 20,10)
- `is_base_unit` (boolean)
- `is_public` (boolean)
- `is_published` (boolean)
- `published_at` (timestamp)
- `unpublished_at` (timestamp)

### `Tss\Inventory\Models\ProductsUnit`

- Added `unit_symbol` column to speed up queries.
- Added `type` column for classification.
- Improved duplicate checking.

### `Tss\Inventory\Models\ProductsPricesUnit`

- Added `unit_symbol` column.
- Improved `makePrimary` and `makeIsActive` functions to correctly update records.

---

## Detailed Explanation of Models and Traits

### Unit Model

#### New Columns and Their Uses

| Column | Type | Description |
|--------|------|-------|
| `unit_symbol` | string | International unit symbol (e.g., 'kg', 'm', 'dozen'), indexed for fast searching |
| `conversion_factor` | decimal(20,10) | Conversion factor to the base unit (1 for the base unit) |
| `value` | decimal(20,10) | Unit value (synonym for conversion_factor) |
| `is_base_unit` | boolean | Is this unit the base unit for its type? |
| `is_public` | boolean | Is the unit public (visible to everyone)? |
| `is_published` | boolean | Is the unit published? |
| `published_at` | timestamp | Publication date |
| `unpublished_at` | timestamp | Unpublication date |

#### Added Traits

##### 1. `HasBaseUnit`
- **Description**: Handling the base unit for the type.
- **Functions**:
  - `getBaseUnit($type = null, $departments_id = null)`: Get the base unit for a specific type.
  - `makeBaseUnit()`: Make the current unit the base unit for its type (and update other units to be non-base).

##### 2. `HasDefault`
- **Description**: Making a unit default.
- **Functions**:
  - `getDefault($departments_id = null)`: Get the default unit for a specific department.
  - `makeDefault()`: Make the current unit default (and update other units).

##### 3. `HasRecordsOptions`
- **Description**: The `getRecords` function for retrieving records flexibly (detailed in a later section).

##### 4. `ListObjects`
- **Description**: Temporary caching of objects to reduce database queries.
- **Functions**:
  - `getRecordObj($key = null, $is_exception = true, $keyName = null)`: Get a cached unit object using a key.
  - `clearCacheListObjectKeyByDepartments()`: Clear the cache for a specific department.
  - `clearCacheObjectKeyModel()`: Clear the cache for a specific object.

##### 5. `ListOptions`
- **Description**: Providing option lists for interfaces (e.g., `listAvailable`, `listEnabled`).
- **Functions**:
  - `listAvailableDepartments()`, `listEnabledDepartments()`: Lists of available/active units by department.
  - `clearCacheListByCompanys()`, `clearCacheListByDepartments()`: Clear list caches.

##### 6. `FieldsOptions`
- **Description**: Providing options for fields in admin interfaces.
- **Functions**:
  - `getStatusOptions()`: Status options (active/inactive).
  - `getTypeOptions()`: Unit type options.
  - `getUnitSymbolOptions()`: Unit symbol options.
  - `getMainSubOptions()`: Main/sub options.
  - `getParentOptions()`: Parent options.

##### 7. `HasScopesModel`
- **Description**: Advanced query scopes.
- **Functions**: (See the Scopes section later)

### ProductsUnit Model

- **Relationships**: `belongsTo` with `Unit` and `Product`.
- **Duplicate Check**: In `beforeCreate`, it checks that the same unit is not duplicated for the same product, and that the main unit is not repeated.

### ProductsPricesUnit Model

- **Relationships**: `belongsTo` with `ProductsUnit`, `Price`, `Currency`.
- **Important Functions**:
  - `makePrimary()`: Make this price the default for this unit and product.
  - `makeIsActive()`: Make this price active and deactivate others.

---

## Caching Mechanisms

### Object Caching

Unit objects are stored in `Unit::$objectCache` after being retrieved from the database, reducing the number of queries when the same unit is requested multiple times.

```php
// First time: database query
$unit1 = Unit::getRecordObj(5);

// Second time: from cache (no query)
$unit2 = Unit::getRecordObj(5);
```

### Database Caching (Laravel Cache)

`Cache::rememberForever` is used to store unit arrays (results of `toArray()`) to speed up frequent queries.

```php
$item = Cache::rememberForever(static::getCacheListObjectKey().'._'.$cacheKey, function () use ($key,$keyName) {
    // Database query
});
```

### Cache Clearing Functions

- `clearCache()`: Clear all cache keys related to units.
- `clearCacheListByCompanys($companys_id)`: Clear list cache for a specific company.
- `clearCacheListByDepartments($departments_id)`: Clear list cache for a specific department.
- `clearCacheListObjectKeyByDepartments()`: Clear object cache for a specific department.
- `clearCacheObjectKeyModel()`: Clear cache for a specific unit object.

### Importance of Caching

- Significantly improves performance when fetching units frequently.
- Reduces load on the database.
- Ensures data consistency upon update (cache is automatically cleared after saving).

---

## The `getRecords` Function in Detail

Found in `HasRecordsOptions` and used to retrieve unit records with high flexibility.

### Supported Options

| Option | Type | Description |
|--------|------|-------|
| `id` | mixed | Unit ID (number, array, or `*`) |
| `companys_id` | mixed | Company ID |
| `departments_id` | mixed | Department ID |
| `type` | string | Unit type |
| `with_type` | string | Alias for `type` |
| `name` | string | Unit name |
| `unit_symbol` | string | Unit symbol |
| `parent_id` | mixed | Parent ID |
| `level` | int | Level |
| `status` | string | Status |
| `is_active` | bool | Active/Inactive |
| `is_default` | bool | Default/Not default |
| `is_base_unit` | bool | Base/Not base |
| `is_published` | bool | Published/Not published |
| `is_public` | bool | Public/Private |
| `is_rental_units` | bool | Rental units/Not rental |
| `q` | string | Text to search in name, description, and symbol |
| `orderBy` | string | Order by field |
| `orderDirection` | string | Order direction (`asc`/`desc`) |
| `withTrashed` | bool | Include trashed |
| `onlyTrashed` | bool | Only trashed |
| `page` | int | Page number (for paginator) |
| `per_page` | int | Items per page |
| `is_query` | bool | Return the query object (without executing) |
| `is_first` | bool | Return only the first record |
| `is_collection` | bool | Return a Collection |
| `is_paginator` | bool | Return paginated results |
| `is_to_sql` | bool | Print the SQL query (for debugging) |

### Examples

```php
// Get all active units of type weight
$result = Unit::getRecords([
    'type' => UnitType::WEIGHT,
    'is_active' => true,
    'is_collection' => true,
]);

// Search for a unit by symbol
$result = Unit::getRecords([
    'unit_symbol' => 'kg',
    'is_first' => true,
]);

// Paginated units
$result = Unit::getRecords([
    'departments_id' => 5,
    'is_paginator' => true,
    'per_page' => 20,
]);

// Recently updated units
$result = Unit::getRecords([
    'orderBy' => 'updated_at',
    'orderDirection' => 'desc',
    'take_limet' => 10,
]);
```

---

## Available Scopes in `Unit`

All scopes are in `HasScopesModel` and are used in queries.

| Scope | Description |
|--------|-------|
| `isActive()` | Only active units |
| `isNotActive()` | Only inactive units |
| `isPublic()` | Only public units |
| `isNotPublic()` | Only non-public units |
| `isDefault()` | Only default units |
| `isNotDefault()` | Only non-default units |
| `isBaseUnit()` | Only base units |
| `isNotBaseUnit()` | Only non-base units |
| `isPublished()` | Only published units (considering `published_at` and `unpublished_at`) |
| `isNotPublished()` | Only unpublished units |
| `isHidden()` | (Alias for `isPublished`) |
| `isNotHidden()` | (Alias for `isNotPublished`) |
| `whereType($value, $is_or, $is_not, $is_force)` | Filter by type |
| `withType($value, $is_or, $is_not, $is_force)` | Alias for `whereType` |
| `whereUnitSymbol($value, $is_or, $is_not, $is_force)` | Filter by unit symbol |
| `whereName($value, $is_or, $is_not, $is_force)` | Filter by name |
| `whereParentId($value, $is_or, $is_not, $is_force)` | Filter by parent ID |
| `whereLevel($value, $is_or, $is_not, $is_force)` | Filter by level |
| `whereStatus($value, $is_or, $is_not, $is_force)` | Filter by status |
| `isRentalUnits()` | Rental units (hour, day, week, month) |
| `isNotRentalUnits()` | Non-rental units |
| `whereLastChange($date_at, $interval)` | Units changed during the period (using `created_at` or `updated_at`) |

### Examples

```php
// All base units of type weight
$baseWeightUnits = Unit::isBaseUnit()->whereType(UnitType::WEIGHT)->get();

// Active rental units
$rentalUnits = Unit::isRentalUnits()->isActive()->get();

// Units modified in the last 5 minutes
$recentUnits = Unit::whereLastChange(now(), 5)->get();
```

---

## Configuration (Config) Affecting Behavior

Settings are defined in the `config.php` file under the `tss.inventory::unit` key.

| Key | Description | Default Value |
|---------|-------|-------------------|
| `is_default_company` | Use the default company | false |
| `department_type` | Type of department used (`main` or `sub`) | `sub` |
| `check_unit_symbol` | Enable unit symbol validation | null (can be `*`, `all`, `create`, `update`) |
| `check_unit_type` | Enable unit type validation | null |
| `default_main_sub` | Default value for the `main_sub` field | `sub` |
| `auto_sub_parent_id` | Automatically assign parent for sub-units | false |
| `default_parent_id` | Default parent ID | null |
| `order_by` | Default order field in `getRecords` | `created_at` |
| `order_dir` | Default order direction | `desc` |
| `per_page` | Default items per page | 15 |

### Example of Using Config

```php
// Anywhere in the code
if (Config::get('tss.inventory::unit.is_default_company')) {
    // Use the default company
}
```

---

## Localization

The system uses translation files in `tss.inventory::lang.units.*`.

### Adding a Label for a New Unit

1.  Add the unit symbol to `UnitNameHelper::getAllUnitsWithLabels`.
2.  Add the translation in the language files:

```php
// en/lang.php
'units' => [
    'units_name' => [
        'myunit' => 'My Unit Name',
    ],
],
```

### Translation Functions

- `UnitType::getTransLabel($type, $lang)`
- `UnitNameHelper::getUnitLabel($unit, $lang)`
- `UnitNameHelper::getEnglishLabel($unit)`

---

## Unique Code Generation Functions in `UnitManager`

These functions are used to generate unique identifiers for cards, vouchers, etc.

| Function | Description | Example |
|--------|-------|-------|
| `generateDigital($length)` | Generate a random number of a given length | `UnitManager::generateDigital(6)` // 483912 |
| `getGuid()` | Generate a random UUID (without braces) | `UnitManager::getGuid()` // "f47ac10b-58cc-4372-a567-0e02b2c3d479" |
| `getVoucherCode()` | Generate a 13-digit voucher code (time-based) | `UnitManager::getVoucherCode()` // 4829117456382 |
| `humanUuid($options)` | Human-friendly UUID in `YYYYMMDD-HHMM-SSMM-MMMM...` format | `UnitManager::humanUuid(['useDashes'=>true])` // "20260318-1430-1234-5678-123456789012" |
| `nanoUid($options)` | Unique nanosecond-based identifier | `UnitManager::nanoUid()` // "20260318143022564321" |

---

## Printing Reports (`printReports`)

The `UnitManager::printReports($options)` function is used to print unit reports.

### Mechanism

- Relies on the `Tss\Reports` system to generate reports.
- Calls `Unit::getRecords` to fetch data.
- Uses the `ReportsUnits` report template.

### Example

```php
$options = [
    'type' => UnitType::WEIGHT,
    'is_active' => true,
    'orderBy' => 'name',
];
$html = UnitManager::printReports($options);
echo $html; // Display the report in the browser
```

---

## Performance Notes and Query Optimization

1.  **Use Indexes**:
    - Ensure indexes exist on frequently used columns: `unit_symbol`, `type`, `is_base_unit`, `companys_id`, `departments_id`.
    - Indexes have been added in migration files.

2.  **Use Caching**:
    - Use `Unit::getRecordObj` instead of `Unit::find` when the same unit is needed frequently.
    - Use `listEnabledDepartments` for frequently used options.

3.  **Avoid Fetching Large Amounts**:
    - When using `getRecords` with `is_paginator`, set an appropriate `per_page`.
    - Use `take_limet` if you only need a limited number.

4.  **Complex Queries**:
    - When using `whereLastChange`, ensure there is an index on `created_at` and `updated_at`.
    - Use `is_to_sql` to review the query during development.

---

## Upgrade Guide

### From Older Versions (pre-1.1.13)

1.  **Run Updates**:
    ```bash
    php artisan october:up
    ```
    This will run all migration files from 1.1.13 to 1.1.25.

2.  **Database Changes**:
    - Columns `unit_symbol`, `conversion_factor`, `value`, `is_base_unit`, `is_public`, `is_published`, `published_at`, `unpublished_at` were added to `tss_inventory_units`.
    - `unit_symbol` and `type` were added to `tss_inventory_products_units`.
    - `unit_symbol` was added to `tss_inventory_products_prices_units`.

3.  **Code Changes**:
    - Use `unit_symbol` instead of relying only on `id` in linking operations.
    - Use `UnitNameHelper` and `UnitConverter` functions instead of custom logic.
    - Update any code that uses old type names (`Numerical` → `quantity`, `Mass` → `weight`, `Length` → `length`, `Size` → `volume`).

4.  **Data Seeding**:
    - If you have old units without `unit_symbol`, you can use `UnitCreator::firstOrCreateFromSymbol` to assign appropriate symbols.

---

## Frequently Asked Questions (FAQ)

### 1. How do I add a new unit not present in the system?

Add the symbol and conversion factor to `UnitConverter::CONVERSION_DATA`, add the label in `UnitNameHelper::getAllUnitsWithLabels` and the translation files. Then use `UnitCreator::createUnitFromSymbol` to create it in the database.

### 2. Why can't I convert from `dozen` to `ream`?

Because they are in different contexts: `dozen` is in the packaging context and `ream` is in the paper context. The conversion is illogical in practical applications. You can use `UnitContextValidator::getConversionReason` to find out why.

### 3. How do I link a unit to a product with a different price?

First use `UnitCreator::attachUnitToProduct` to link the unit, then `UnitCreator::createProductUnitPrice` to set the price.

### 4. How do I get a list of units compatible with a given unit?

Use `UnitContextValidator::getCompatibleUnits($unitSymbol)`.

### 5. What does `strictContext` mean in `UnitConverter::convert`?

When enabled (`true`), it checks for exact context matching (even within the same type) before allowing conversion. This prevents conversions between different subtypes like `dozen` to `pair`.

### 6. How do I handle errors when creating a duplicate unit?

Use `firstOrCreateFromSymbol` which searches first and creates if not found. Or use `is_force_create` in `createUnit` to bypass the error.

---

## Troubleshooting

### Common Error Messages

| Error | Cause | Solution |
|-------|-------|------|
| `Unit unknown: X` | Unit symbol not supported | Check the symbol with `UnitNameHelper::isValidUnit` |
| `Cannot convert from ... to ...` | Units are of different types or conversion is illogical | Ensure type compatibility or use `canConvert` first |
| `Duplicate unit` | Attempting to create a unit with the same symbol, type, and department | Use `firstOrCreateFromSymbol` or `is_force_create` |
| `Cannot find unit` | Unit does not exist or was deleted | Check `id` or symbol, use `withTrashed` if necessary |

### Cache Issues

If you encounter issues with updated data not appearing, clear the cache manually:

```php
Unit::clearCache(); // Clear all cache
// Or to clear cache for a specific unit
Unit::clearCacheObjectKeyModel($unitId);
```

### Illogical Conversion Issues

If you want to allow illogical conversions despite warnings, use `UnitConverter::convert` with `strictContext = false` or use `UnitHelper::safeConvert` with `$validateLogic = false`.

---

## Integrated Examples

### Example 1: Creating a New Unit and Linking it to a Product

```php
use Tss\Inventory\Classes\UnitManager;
use Tss\Inventory\Classes\Units\UnitCreator;

// 1. Create a unit
$unitResult = UnitManager::createUnit([
    'name' => 'Dozen',
    'unit_symbol' => 'dozen',
    'type' => UnitType::QUANTITY_PACKAGING,
    'is_base_unit' => true,
    'conversion_factor' => 12,
]);

if (!$unitResult['status']) {
    throw new Exception($unitResult['error']);
}

$unit = $unitResult['model'];

// 2. Attach the unit to a product (products_id = 123)
$attachResult = UnitCreator::attachUnitToProduct([
    'products_id' => 123,
    'units_id' => $unit->id,
    'is_main' => false,
    'conversion_factor' => 12,
]);

if (!$attachResult['status']) {
    throw new Exception($attachResult['error']);
}

// 3. Create a price for this unit
$priceResult = UnitCreator::createProductUnitPrice([
    'products_id' => 123,
    'units_id' => $unit->id,
    'prices_id' => 1,
    'value' => 50.00,
    'currencys_id' => 1,
    'is_default' => false,
]);

echo "Unit created and linked to product successfully.";
```

### Example 2: Using the Conversion System in a Sale Process

```php
use Tss\Inventory\Classes\Units\UnitConverter;
use Tss\Inventory\Classes\Units\UnitHelper;

// Product is sold by the dozen, but the customer wants 50 pieces
$quantityInDozen = UnitConverter::convert(50, 'pcs', 'dozen', 2); // 4.17 dozen

// Show the price to the customer in the requested unit
$pricePerPiece = 5; // Price per piece
$totalPrice = $quantityInDozen * $pricePerPiece * 12; // Or another method

// Format the display
echo UnitHelper::formatWithUnit($quantityInDozen, 'dozen', 2, 'en'); // "4.17 dozen"
```

### Example 3: Searching for a Unit and Displaying Its Information

```php
use Tss\Inventory\Classes\Units\UnitNameHelper;

$searchTerm = 'kilo';
$info = UnitNameHelper::searchUnitAdvanced($searchTerm, null, 'en');

if ($info) {
    echo "Unit: {$info['unit_name_en']} ({$info['unit_id']})\n";
    echo "Type: {$info['type_name_en']}\n";
    echo "Is base? " . ($info['is_base_unit'] ? 'Yes' : 'No') . "\n";
    if ($info['context']) {
        echo "Context: {$info['context']}\n";
    }
}
```

### Example 4: Creating All Standard Units for the Weight Type

```php
use Tss\Inventory\Classes\Units\UnitCreator;

$result = UnitCreator::createStandardUnitsForType([
    'type' => UnitType::WEIGHT,
    'companys_id' => 1,
    'departments_id' => 2,
]);

if ($result['status']) {
    echo "Successfully created " . count($result['data']) . " units.\n";
    foreach ($result['data'] as $unit) {
        echo "- {$unit->name} ({$unit->unit_symbol})\n";
    }
}
```

### Example 5: Using `getRecords` with Multiple Options

```php
$result = Unit::getRecords([
    'type' => UnitType::WEIGHT,
    'is_base_unit' => true,
    'is_active' => true,
    'orderBy' => 'name',
    'orderDirection' => 'asc',
    'is_collection' => true,
]);

if ($result['status']) {
    foreach ($result['data'] as $unit) {
        echo $unit->name . ' (' . $unit->unit_symbol . ")\n";
    }
}
```

---

## Best Practices and Error Handling

1.  **Use `SmartUnitConverter::safeConvert`** when you need to ensure conversion logicality in business contexts (e.g., converting dozens to reams is illogical).
2.  **Use `UnitNameHelper::isValidUnit`** before using any unit symbol to ensure it is supported.
3.  **When creating units in the database**, make sure to pass the appropriate `companys_id` and `departments_id`.
4.  **To get complete information about a unit**, use `UnitNameHelper::getUnitFullInfo` instead of `getUnitInfo` if you need detailed data.
5.  **For displaying units in user interfaces**, use `UnitHelper::generateGroupedSelectOptions` to get a categorized dropdown list.
6.  **Error Handling**:
    - Most `UnitCreator` functions return an array containing `status`, `error`, and `message`.
    - Conversion functions may throw `InvalidArgumentException` exceptions, so use `try-catch`.

```php
try {
    $result = SmartUnitConverter::safeConvert($value, $from, $to);
} catch (InvalidArgumentException $e) {
    // Illogical conversion or unsupported unit
    log_error($e->getMessage());
    return back()->withError($e->getMessage());
}
```

---

## Conclusion

The Comprehensive Unit Management and Conversion System provides a complete and flexible solution for all unit handling needs in TSS Inventory applications. Thanks to the modular design and specialized classes, developers can:

- Accurately convert values between any two supported units.
- Obtain detailed information about units.
- Easily create and manage units in the database.
- Link units to products and prices.
- Verify the logicality of conversions in business contexts.
- Provide rich user interfaces (dropdowns, formatted information, conversion forms).

The system is easily extensible: new units can be added by updating `UnitConverter::CONVERSION_DATA`, new logical rules can be added in `UnitContextValidator::CONVERSION_RULES`, and new languages can be supported via translation files.

---

This covers all aspects of the system with practical examples for each part. For more details, please refer to the source code and inline comments.