# Comprehensive Unit Management and Conversion System Documentation

## 📋 Table of Contents

1. [General Introduction](#general-introduction)
2. [Architecture Overview](#architecture-overview)
3. [Main Classes](#main-classes)
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
4. [Updated Models](#updated-models)
5. [Integrated Examples](#integrated-examples)
6. [Best Practices and Error Handling](#best-practices-and-error-handling)
7. [Conclusion](#conclusion)

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

Tss\Inventory\Classes
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

Main Functions

Function Description
getMainTypesWithLabels($lang) Returns main types with their labels (['length'=>'Length', ...])
getAllTypesWithLabels($lang) Returns all types (including subtypes) with their labels
getQuantitySubtypesWithLabels($lang) Returns only quantity subtypes with labels
getTransLabel($type, $lang) Get the translated label for a specific type
isValid($type) Check if the type is defined
isQuantitySubtype($type) Check if the type is a quantity subtype
isCustomType($type) Check if the type is custom
getMainType($type) Return the main type (for subtypes)
getAllTypes() Return a list of all types
getMainTypes() Return a list of main types only
getSelectOptions($selected, $includeEmpty, $onlyMainTypes) Generate HTML <select> options for types

Examples

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

UnitNameHelper - Unit Name Helper

This class provides centralized access to unit information: their names, symbols, types, contexts, and searching.

Main Functions

Function Description
getAllUnitsWithLabels($lang) Returns all units with their labels (['mm'=>'Millimeter', ...])
getUnitsByTypeWithLabels($type, $lang, $isBaseUnit) Returns units of a specific type, with optional filtering for base units only
getUnitLabel($unit, $lang) Get the label for a specific unit
getEnglishLabel($unit) Get the English label for the unit
getCategorizedUnits($lang) Returns units categorized by type (for structuring lists)
getSelectOptions($type, $selected, $includeEmpty, $lang) Generate HTML <select> options for units of a specific type
getGroupedSelectOptions($selected, $includeEmpty, $lang) Generate <select> options grouped by type
searchUnits($searchText, $withLabels, $lang) Simple search for units containing the text
searchUnitAdvanced($searchText, $type, $lang) Advanced search returning information of the first matching unit (by symbol, Arabic name, English name)
getUnitType($unit) Return the unit type
isValidUnit($unit) Validate unit symbol
getSimilarUnits($unit, $lang) Return units of the same type
getBaseUnits($lang) Return base units for each type (['length'=>['unit'=>'m','label'=>'Meter']])
getUnitInfo($unit, $lang) Return basic information about the unit (symbol, label, type, ...)
getUnitFullInfo($unit, $lang) Return complete information about the unit (including unit_id, unit_name_ar, unit_name_en, type_id, type_name_ar, type_name_en, is_base_unit, base_unit, context, is_metric, is_imperial, display_name, full_name)
isBaseUnitForItsType($unit) Check if the unit is base for its type
getBaseUnitByType($type) Return the base unit symbol for a given type

Examples

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

UnitConverter - Unit Converter

The heart of the conversion system, containing conversion data for all units and performing mathematical conversions.

Main Functions

Function Description
convert($value, $from, $to, $precision, $strictContext) Convert a value from one unit to another. $strictContext enables strict context checking for quantity units.
convertBatch(array $values, $from, $to, $precision) Convert a batch of values at once.
describeConversion($value, $from, $to, $precision) Return a textual description of the conversion (e.g., "1 meter = 100 centimeters").
isUnitSupported($unit) Check if the unit is supported.
getUnitType($unit) Return the unit type.
getArabicLabel($unit) Return the Arabic label (shortcut for UnitNameHelper::getUnitLabel).
getUnitsByType($type, $withLabels, $lang) Return units of a specific type (with or without labels).
getAllUnitsGrouped($groupByMainType, $lang) Return all units grouped by main type or full type.
getUnitInfo($unit, $lang) Return basic information about the unit (including conversion factor).
canConvert($from, $to) Check if conversion between two units is possible (without exception).
getSupportedUnits($type, $includeSubtypes) Return a list of all supported unit symbols, or for a specific type including subtypes.
getSupportedUnitsWithLabels($type, $includeSubtypes, $lang) Same as above but with labels.
getSupportedConversionTypes($includeSubtypes) Return a list of supported conversion types.
getSupportedConversionTypesWithLabels($includeSubtypes, $lang) Same as above with labels.
isTypeSupported($type) Check if the unit type is supported.
getConversionFactor($unit, $type) Return the conversion factor for the unit relative to the base unit of the type.

Examples

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

UnitHelper - Unit Helper

General helper functions to facilitate working with units in applications.

Main Functions

Function Description
safeConvert($value, $from, $to, $precision, $validateLogic) Safe conversion with error handling and optional logical validation (returns null on failure).
formatWithUnit($value, $unit, $decimals, $lang) Format a value with the unit label (e.g., "100.00 Kilograms").
getBestDisplayUnit($value, $currentUnit, $preferredUnits, $lang) Find the best unit to display a value (so that the value is between 1 and 1000).
generateSelectOptions($type, $selected, $includeEmpty, $lang) Generate <select> options for units of a specific type.
generateGroupedSelectOptions($selected, $includeEmpty, $lang) Generate <select> options grouped by type.
getRentalTimeUnits($lang, $is_key) Return time units used in rentals (hour, day, week, month).
getConversionFactor($unit, $defaultValue) Return the conversion factor for a unit (with a default value in case of error).
getConversionFactorFormatted($unit, $defaultValue, $decimals, $isFormatDecimal) Return the conversion factor formatted (without trailing zeros).
formatDecimal($value, $decimals) Format a decimal number without trailing zeros.

Examples

```php
// Safe conversion
$result = UnitHelper::safeConvert(1, 'dozen', 'box', 2, true); // 0.50

// Format value
echo UnitHelper::formatWithUnit(2.5, 'kg', 3, 'en'); // "2.500 Kilograms"

// Get best display unit
$best = UnitHelper::getBestDisplayUnit(1500, 'm', ['km','m'], 'en'); // 'km'

// Get rental units
$rentalUnits = UnitHelper::getRentalTimeUnits('en'); // ['h'=>'Hour', 'day'=>'Day', ...]

// Format decimal number
echo UnitHelper::formatDecimal(12.3000, 4); // "12.3"
```

---

UnitContextValidator - Unit Context Validator

Responsible for verifying the logicality of converting quantity units based on their context (packaging, paper, pairs, large quantities).

Main Functions

Function Description
isConversionLogical($from, $to) Check if a conversion between two units is logical (within the same context or via pcs).
getConversionReason($from, $to) Return the reason for logicality or lack thereof (text).
getCompatibleUnits($unit) Return a list of logically compatible units with a given unit (each is an array containing unit and label).

Rules

· pcs (piece) is a universal unit that can be converted from and to any quantity unit.
· Specific rules defined in CONVERSION_RULES determine logical relationships between units (e.g., dozen ↔ box).
· If no rule exists, conversion is allowed if the context matches.

Examples

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

SmartUnitConverter - Smart Converter

Provides conversions with warnings and strict logical validation.

Main Functions

Function Description
safeConvert($value, $from, $to, $precision) Safe conversion with logical validation; throws an exception if the conversion is illogical.
convertWithWarning($value, $from, $to, $precision) Conversion returning an array containing the value, is_logical flag, and a warning if not logical.

Examples

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

UnitsInformation - Units Information

A class to display unit information in formatted HTML (for documentation and demonstration purposes).

Main Functions

Function Description
displayAllUnitsInfo() Display all units in a full HTML page with statistics and cards (using Bootstrap).
displayTypeInfo($typeKey) Display units of a specific type only.
displayAllUnitsInfoBootstrap() Same as displayAllUnitsInfo (alias).
displayTypeInfoBootstrap($typeKey) Same as displayTypeInfo.

Example

```php
echo UnitsInformation::displayAllUnitsInfo();
```

---

ConversionForm - Conversion Form

Creates interactive HTML forms for testing unit conversions.

Main Functions

Function Description
renderAdvancedForm() Advanced form with options (conversion type, units, value, strict mode, precision).
renderForm() Simple form.
testQuantityConversions() Test function to try quantity conversions (prints results to the command line).

Example

```php
$form = new ConversionForm();
echo $form->renderAdvancedForm();
```

---

UnitFactory - Unit Factory

Creates Unit objects from unit symbols or types.

Main Functions

Function Description
createFromString($unitString) Create a Unit object from a unit symbol (e.g., 'm').
createUnitsByType($type) Create an array of Unit objects for a specific type.
createAllUnits() Create an array of Unit objects for all units.
createBaseUnits() Create an array of Unit objects for base units of each type (associated with the type).
createUnitByContext($context) Create a base unit for a specific context (e.g., packaging → dozen).
createCompatibleUnits($unitSymbol) Create an array of Unit objects compatible with a given unit.

Examples

```php
$meter = UnitFactory::createFromString('m');
echo $meter->convertTo(100, 'cm'); // 10000

$lengthUnits = UnitFactory::createUnitsByType('length');
foreach ($lengthUnits as $unit) {
    echo $unit->symbol . ' - ' . $unit->label . "\n";
}
```

---

Unit - Unit Object

Represents a single unit of measurement with its properties and operations.

Properties

Property Description
symbol Unit symbol (e.g., 'kg')
type Unit type (e.g., 'weight')
label Arabic label (e.g., 'كيلوجرام')
context Context (for quantity units)

Main Functions

Function Description
convertTo($value, $toUnit, $precision) Convert a value to another unit.
convertToBase($value, $precision) Convert to the base unit of the type.
isCompatibleWith($otherUnit) Check compatibility with another unit (same main type).
getConversionFactor() Return the conversion factor for the unit.
isBaseUnit() Check if the unit is base.
getEnglishLabel() Return the English label.
getInfo() Return complete information about the unit (using UnitNameHelper).
getContext() Return the context.
getCompatibleUnits() Return compatible units (of the same type).
canConvertTo($toUnit) Check if conversion to another unit is possible.
formatValue($value, $decimals) Format a value with the unit (e.g., "2.50 Kilograms").
__toString() String representation of the unit (e.g., "kg (Kilogram)").
equals(Unit $other) Compare with another unit.
copy() Create a new copy.

Examples

```php
$kg = UnitFactory::createFromString('kg');
echo $kg->convertTo(5, 'g'); // 5000
echo $kg->formatValue(2.5, 2); // "2.50 Kilograms"
if ($kg->canConvertTo('lb')) { ... }
```

---

UnitCreator - Database Unit Creator

A specialized class for creating and updating units and linking them to products and prices in the database.

Main Functions

Function Description
createUnit(array $options, $is_test_create) Create a new unit in the database.
createUnitFromSymbol(array $options, $is_test_create) Create a unit from a known symbol (e.g., 'kg') with automatic data completion.
createStandardUnitsForType(array $options, $is_test_create) Create all standard units for a specific type (e.g., all length units).
createAllStandardUnits(array $options, $is_test_create) Create all standard units for all types.
attachUnitToProduct(array $options, $is_test_create) Attach a unit to a product (create a record in products_units).
createProductUnitPrice(array $options, $is_test_create) Create a price for a product unit (in products_prices_units).
createProductUnitsStructure(array $options, $is_test_create) Create a complete structure of units and prices for a specific product.
createUnitConversion(array $options, $is_test_create) Create a conversion relationship between two units (for future use).
importUnitsFromSymbols(array $options, $is_test_create) Import units from a list of symbols.
createStandardQuantityUnits(array $options, $is_test_create) Create standard quantity units (dozen, ream, pair, ...).
firstOrCreateFromSymbol(array $options, $is_test_create) Find a unit by a given symbol, and create it if it doesn't exist.

Common $options Structure

· companys_id, departments_id: To specify the company and department.
· name: Unit name.
· unit_symbol: Unit symbol.
· type: Unit type (from UnitType).
· conversion_factor, value: Conversion factor (usually the same value).
· is_base_unit: Is it the base unit?
· is_default: Is it the default?
· is_active: Activation status.

Examples

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

UnitManager - Unified Unit Manager

A unified programming interface that combines most system functions and provides easy-to-use static functions.

Main Functions

Function Description
createUnit($options, $is_test_create) Create a new unit (relies on UnitCreator::createUnit).
checkUnit($options) Validate a unit (its symbol, balance, user) - used for validating recharge cards, for example.
getQueryDate($records, $options) Apply date conditions to a query (helper function).
scopeWhereField($query, $field, $value, $is_or, $table, $is_not, $is_force) Helper scope for applying conditions on fields with support for * and all.
printReports($options) Print unit reports (using the reporting system).
getAllUnitsGrouped($groupByMainType, $lang) Get all units categorized (passes to UnitConverter::getAllUnitsGrouped).
getUnitsForType($type, $includeLabels, $lang) Get units for a specific type (passes to UnitConverter::getUnitsForType).
getRentalTimeUnits($lang, $is_key) Get time units used in rentals.
formatDecimal($value, $decimals) Format a decimal number without trailing zeros.
generateDigital($length) Generate a random number of a given length (for card codes).
getGuid() Generate a random UUID.
getVoucherCode() Generate a 13-digit voucher code.
humanUuid($options) Generate a human-friendly UUID (format YYYYMMDD-HHMM-SSMM-MMMM...).
nanoUid($options) Generate a unique nanosecond-based identifier.

Examples

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

Updated Models

Tss\Inventory\Models\Unit

New columns added to support the system:

· unit_symbol (string, index)
· conversion_factor (decimal, 20,10)
· value (decimal, 20,10)
· is_base_unit (boolean)
· is_public (boolean)
· is_published (boolean)
· published_at (timestamp)
· unpublished_at (timestamp)

Additionally, Traits have been added to provide advanced functionality:

· HasBaseUnit: Handling the base unit for the type.
· HasDefault: Making a unit default.
· HasRecordsOptions: getRecords function for flexibly retrieving records.
· ListObjects: Temporary caching of objects.
· ListOptions: Option lists (for interfaces).
· FieldsOptions: Field options (for interfaces).
· HasScopesModel: Advanced query scopes.

Tss\Inventory\Models\ProductsUnit

· Added unit_symbol column to speed up queries.
· Added type column for classification.
· Improved duplicate checking.

Tss\Inventory\Models\ProductsPricesUnit

· Added unit_symbol column.
· Improved makePrimary and makeIsActive functions to correctly update records.

---

Integrated Examples

Example 1: Creating a New Unit and Linking it to a Product

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

Example 2: Using the Conversion System in a Sale Process

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

Example 3: Searching for a Unit and Displaying Its Information

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

Example 4: Creating All Standard Units for the Weight Type

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

---

Best Practices and Error Handling

1. Use SmartUnitConverter::safeConvert when you need to ensure conversion logicality in business contexts (e.g., converting dozens to reams is illogical).
2. Use UnitNameHelper::isValidUnit before using any unit symbol to ensure it is supported.
3. When creating units in the database, make sure to pass the appropriate companys_id and departments_id.
4. To get complete information about a unit, use UnitNameHelper::getUnitFullInfo instead of getUnitInfo if you need detailed data.
5. For displaying units in user interfaces, use UnitHelper::generateGroupedSelectOptions to get a categorized dropdown list.
6. Error Handling:
   · Most UnitCreator functions return an array containing status, error, and message.
   · Conversion functions may throw InvalidArgumentException exceptions, so use try-catch.

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

Conclusion

The Comprehensive Unit Management and Conversion System provides a complete and flexible solution for all unit handling needs in TSS Inventory applications. Thanks to the modular design and specialized classes, developers can:

· Accurately convert values between any two supported units.
· Obtain detailed information about units.
· Easily create and manage units in the database.
· Link units to products and prices.
· Verify the logicality of conversions in business contexts.
· Provide rich user interfaces (dropdowns, formatted information, conversion forms).

The system is easily extensible: new units can be added by updating UnitConverter::CONVERSION_DATA, new logical rules can be added in UnitContextValidator::CONVERSION_RULES, and new languages can be supported via translation files.

---

This covers all aspects of the system with practical examples for each part. For more details, please refer to the source code and inline comments.