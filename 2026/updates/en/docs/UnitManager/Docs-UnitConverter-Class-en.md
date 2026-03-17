# Conversion and Validation Classes Documentation

## 📋 Table of Contents

1.  [UnitConverter - Basic Unit Converter](#unitconverter---basic-unit-converter)
    - [Introduction](#introduction-unitconverter)
    - [Constants](#constants-unitconverter)
    - [Main Functions](#main-functions-unitconverter)
    - [Comprehensive Examples](#comprehensive-examples-unitconverter)
2.  [UnitContextValidator - Unit Context Validator](#unitcontextvalidator---unit-context-validator)
    - [Introduction](#introduction-unitcontextvalidator)
    - [Constants](#constants-unitcontextvalidator)
    - [Main Functions](#main-functions-unitcontextvalidator)
    - [Examples](#examples-unitcontextvalidator)
3.  [SmartUnitConverter - Smart Converter](#smartunitconverter---smart-converter)
    - [Introduction](#introduction-smartunitconverter)
    - [Main Functions](#main-functions-smartunitconverter)
    - [Examples](#examples-smartunitconverter)
4.  [Conclusion](#conclusion)

---

## UnitConverter - Basic Unit Converter

**Namespace:** `Tss\Inventory\Classes\Units`

### Introduction {#introduction-unitconverter}

The `UnitConverter` class is the heart of the unit conversion system. It contains conversion data for all supported units (over 80 units spread across 16 types) and performs mathematical conversions with high precision. It supports conversion between units of the same type, with special handling for temperatures that require specific formulas. It also provides helper functions to query supported units, get conversion factors, and check conversion feasibility.

### Constants {#constants-unitconverter}

#### `CONVERSION_DATA`
An array containing conversion data for all unit types. Each main key represents a unit type (e.g., `'length'`, `'weight'`) and its value is an associative array of unit symbols and their conversion factors relative to the base unit of that type.

**Code Example:**
```php
private const CONVERSION_DATA = [
    'length' => [
        'mm'    => 0.001,
        'cm'    => 0.01,
        'm'     => 1.0,
        'km'    => 1000.0,
        // ...
    ],
    'weight' => [
        'mg'    => 0.000001,
        'g'     => 0.001,
        'kg'    => 1.0,
        // ...
    ],
    // ... remaining types
];
```

#### `UNIT_CONTEXTS`
An array that defines the context for each quantity unit (e.g., `'dozen'` in the `'quantity_packaging'` context). Used to verify conversion logicality in strict mode.

```php
private const UNIT_CONTEXTS = [
    'pcs'         => 'quantity_general',
    'dozen'       => 'quantity_packaging',
    'half_dozen'  => 'quantity_packaging',
    'box'         => 'quantity_packaging',
    // ...
];
```

### Main Functions {#main-functions-unitconverter}

#### `convert(float $value, string $from, string $to, int $precision = 6, bool $strictContext = false): float`

**Description:**  
Convert a value from one unit to another. This is the core function of the class.

**Parameters:**
- `$value` : The numeric value to convert.
- `$from` : Source unit symbol.
- `$to` : Target unit symbol.
- `$precision` : Number of decimal places for the result (default 6).
- `$strictContext` : If `true`, applies strict context checking for quantity units (prevents conversion between different contexts).

**Returns:**  
The converted value (float).

**Exceptions:**
- `InvalidArgumentException` if the unit is unknown, conversion is not possible (different types), or conversion is illogical in strict mode.

**Examples:**
```php
// Convert 5 kilograms to grams
$result = UnitConverter::convert(5, 'kg', 'g'); // 5000

// Convert 1 meter to centimeters with 2 decimal places
$result = UnitConverter::convert(1, 'm', 'cm', 2); // 100.00

// Temperature conversion
$result = UnitConverter::convert(100, 'c', 'f', 2); // 212.00

// Convert quantity units (dozen to pieces)
$result = UnitConverter::convert(2, 'dozen', 'pcs'); // 24

// Conversion with strict mode (will prevent conversion from dozen to ream as they are in different contexts)
try {
    $result = UnitConverter::convert(1, 'dozen', 'ream', 6, true);
} catch (InvalidArgumentException $e) {
    echo $e->getMessage(); // "Illogical conversion: dozen to ream"
}
```

#### `convertBatch(array $values, string $from, string $to, int $precision = 6): array`

**Description:**  
Convert a batch of values from one unit to another.

**Parameters:**
- `$values` : An array of numeric values.
- `$from` : Source unit symbol.
- `$to` : Target unit symbol.
- `$precision` : Number of decimal places.

**Returns:**  
An array of converted values in the same order as the input.

**Example:**
```php
$values = [1, 2, 3, 4, 5];
$results = UnitConverter::convertBatch($values, 'm', 'cm', 2);
// [100.00, 200.00, 300.00, 400.00, 500.00]
```

#### `describeConversion(float $value, string $from, string $to, int $precision = 6): string`

**Description:**  
Return a textual description of the conversion suitable for user display.

**Parameters:**
- `$value` : The value to convert.
- `$from` : Source unit symbol.
- `$to` : Target unit symbol.
- `$precision` : Number of decimal places.

**Returns:**  
A string like `"1 meter = 100.00 centimeter"`.

**Example:**
```php
echo UnitConverter::describeConversion(1, 'm', 'cm', 2);
// "1 meter = 100.00 centimeter"
```

#### `isUnitSupported(string $unit): bool`

**Description:**  
Check if a unit is supported in the system.

**Parameters:**
- `$unit` : Unit symbol.

**Returns:**  
`true` if the unit exists in the conversion data.

**Example:**
```php
if (UnitConverter::isUnitSupported('kg')) {
    echo "Supported";
}
```

#### `getUnitType(string $unit): string`

**Description:**  
Get the unit type (e.g., `'length'`, `'weight'`).

**Parameters:**
- `$unit` : Unit symbol.

**Returns:**  
The unit type as a string.

**Exceptions:**  
`InvalidArgumentException` if the unit is unknown.

**Example:**
```php
$type = UnitConverter::getUnitType('kg'); // 'weight'
```

#### `getConversionFactor(string $unit, string $type): float`

**Description:**  
Get the conversion factor for a unit relative to the base unit of its type. This is an internal function used by `convert`.

**Parameters:**
- `$unit` : Unit symbol.
- `$type` : Unit type.

**Returns:**  
Conversion factor (float).

#### `getConversionFactorUnit(string $unit): float`

**Description:**  
Get the conversion factor for a unit (relative to the base unit of its type). This function is safer because it automatically determines the type.

**Parameters:**
- `$unit` : Unit symbol.

**Returns:**  
Conversion factor.

**Exceptions:**
- If the unit is unknown.
- If the unit is a temperature unit (no fixed conversion factor).
- If the conversion factor is invalid.

**Example:**
```php
$factor = UnitConverter::getConversionFactorUnit('km'); // 1000.0
```

#### `canConvert(string $from, string $to): bool`

**Description:**  
Check if conversion between two units is possible without throwing an exception.

**Parameters:**
- `$from` : Source unit symbol.
- `$to` : Target unit symbol.

**Returns:**  
`true` if conversion is possible, `false` otherwise.

**Example:**
```php
if (UnitConverter::canConvert('dozen', 'box')) {
    echo "Conversion possible";
}
```

#### `getUnitsByType(string $type, bool $withLabels = false, $lang = 'en'): array`

**Description:**  
Return a list of units belonging to a specific type.

**Parameters:**
- `$type` : Unit type (one of the `UnitType` constants).
- `$withLabels` : If `true`, returns an associative array `[symbol => label]`; otherwise returns a simple list of symbols.
- `$lang` : Language used for labels (if requested).

**Returns:**  
An array of symbols or labels.

**Example:**
```php
$lengthUnits = UnitConverter::getUnitsByType(UnitType::LENGTH, true, 'en');
/*
[
    'mm' => 'Millimeter',
    'cm' => 'Centimeter',
    'm' => 'Meter',
    ...
]
*/
```

#### `getAllUnitsGrouped(bool $groupByMainType = true, $lang = 'en'): array`

**Description:**  
Return all units grouped by type, with labels. Useful for building user interfaces.

**Parameters:**
- `$groupByMainType` : If `true`, grouping is done by main types (e.g., `length`, `weight`). If `false`, subtypes are included.
- `$lang` : Language.

**Returns:**  
An associative array where the key is the type, and the value is an array containing `label`, `units`, and `type`.

**Example:**
```php
$grouped = UnitConverter::getAllUnitsGrouped(true, 'en');
foreach ($grouped as $type => $info) {
    echo $info['label'] . ":\n";
    foreach ($info['units'] as $symbol => $label) {
        echo "  $symbol : $label\n";
    }
}
```

#### `getUnitInfo(string $unit, $lang = 'en'): array`

**Description:**  
Get complete information about a specific unit, including symbol, type, label, conversion factor, context, and whether it's a temperature or quantity unit.

**Parameters:**
- `$unit` : Unit symbol.
- `$lang` : Language.

**Returns:**  
An array containing:
- `symbol` : Unit symbol.
- `type` : Unit type.
- `label` : Label according to language.
- `base_factor` : Conversion factor.
- `context` : Context (if any).
- `is_temperature` : Is it a temperature unit.
- `is_quantity` : Is it a quantity unit.
- Plus information from `UnitNameHelper::getUnitInfo`.

**Exceptions:**  
`InvalidArgumentException` if the unit is unsupported.

**Example:**
```php
$info = UnitConverter::getUnitInfo('dozen', 'en');
/*
[
    'symbol' => 'dozen',
    'type' => 'quantity_packaging',
    'label' => 'Dozen',
    'base_factor' => 12.0,
    'context' => 'quantity_packaging',
    'is_temperature' => false,
    'is_quantity' => true,
    ...
]
*/
```

#### `getSupportedUnits($type = null, bool $includeSubtypes = false, $lang = 'en'): array`

**Description:**  
Return a list of all supported unit symbols, or for a specific type with the option to include subtypes.

**Parameters:**
- `$type` : Unit type (if `null`, returns all units).
- `$includeSubtypes` : If `true`, includes subtypes (e.g., `quantity_packaging`). If `false`, `QUANTITY` is treated as the aggregate of subtypes.
- `$lang` : Language (used only when calling other functions, but not directly used here).

**Returns:**  
An array of unit symbols.

**Examples:**
```php
// All units
$all = UnitConverter::getSupportedUnits();

// Length units
$lengthUnits = UnitConverter::getSupportedUnits('length');

// Quantity units (all subtypes)
$quantityUnits = UnitConverter::getSupportedUnits(UnitType::QUANTITY, false);
// ['pcs', 'dozen', 'half_dozen', 'box', ...]
```

#### `getSupportedUnitsWithLabels($type = null, bool $includeSubtypes = false, $lang = 'en'): array`

**Description:**  
Same as `getSupportedUnits` but with labels.

**Returns:**  
An associative array `[symbol => label]`.

#### `getSupportedConversionTypes(bool $includeSubtypes = false): array`

**Description:**  
Return a list of supported conversion types (type names).

**Parameters:**
- `$includeSubtypes` : Include subtypes or not.

**Returns:**  
An array of type names.

#### `getSupportedConversionTypesWithLabels(bool $includeSubtypes = false, $lang = 'en'): array`

**Description:**  
Same as above with labels.

#### `isTypeSupported(string $type): bool`

**Description:**  
Check if a unit type is supported.

#### `getUnitContext(string $unit): ?string`

**Description:**  
Get the context of a unit (for quantity units). Returns `null` if the unit has no context.

**Example:**
```php
$context = UnitConverter::getUnitContext('dozen'); // 'quantity_packaging'
```

### Internal Helper Functions

- `buildUnitLookupCache()`: Builds a cache for quick unit lookup.
- `validateUnit(string $unit)`: Validates a unit.
- `validateCompatibleTypes()`: Validates type compatibility in normal mode.
- `validateStrictConversion()`: Validates compatibility in strict mode.
- `convertTemperature()`: Handles temperature conversions.
- `isTemperatureUnit()`: Checks if a unit is a temperature unit.
- `hasQuantityUnits()`: Checks for presence of quantity units.

### Comprehensive Examples {#comprehensive-examples-unitconverter}

#### Example 1: Various Conversions
```php
// Length
echo UnitConverter::convert(10, 'km', 'm'); // 10000

// Weight
echo UnitConverter::convert(2.5, 'kg', 'lb', 2); // 5.51

// Volume
echo UnitConverter::convert(3, 'l', 'ml'); // 3000

// Area
echo UnitConverter::convert(1, 'acre', 'm2', 2); // 4046.86

// Speed
echo UnitConverter::convert(100, 'km/h', 'm/s', 2); // 27.78

// Pressure
echo UnitConverter::convert(1, 'bar', 'psi', 2); // 14.50

// Energy
echo UnitConverter::convert(500, 'cal', 'kj', 4); // 2.0920

// Power
echo UnitConverter::convert(10, 'hp', 'kw', 2); // 7.46

// Data
echo UnitConverter::convert(1, 'gb', 'mb'); // 1024

// Angle
echo UnitConverter::convert(180, 'deg', 'rad', 4); // 3.1416
```

#### Example 2: Checking Conversion Feasibility
```php
$from = 'dozen';
$to = 'box';

if (UnitConverter::canConvert($from, $to)) {
    $result = UnitConverter::convert(2, $from, $to);
    echo "2 dozen = $result box";
} else {
    echo "Conversion not possible";
}
```

#### Example 3: Getting Unit Information
```php
$unit = 'kg';
if (UnitConverter::isUnitSupported($unit)) {
    $info = UnitConverter::getUnitInfo($unit, 'en');
    echo "Unit: {$info['label']}\n";
    echo "Type: {$info['type']}\n";
    echo "Conversion factor: {$info['base_factor']}\n";
}
```

#### Example 4: Using convertBatch
```php
$kilograms = [1, 2, 3, 4, 5];
$pounds = UnitConverter::convertBatch($kilograms, 'kg', 'lb', 2);
// [2.20, 4.41, 6.61, 8.82, 11.02]
```

#### Example 5: Getting All Units of a Specific Type
```php
$weightUnits = UnitConverter::getSupportedUnitsWithLabels('weight', false, 'en');
foreach ($weightUnits as $symbol => $label) {
    echo "$symbol : $label\n";
}
```

---

## UnitContextValidator - Unit Context Validator

**Namespace:** `Tss\Inventory\Classes\Units`

### Introduction {#introduction-unitcontextvalidator}

The `UnitContextValidator` class is responsible for verifying the logicality of converting quantity units based on their context. It prevents illogical conversions such as converting dozens (packaging) to reams (paper). It relies on the fundamental rule that the `pcs` (piece) unit is a universal unit that can be converted from and to any quantity unit.

### Constants {#constants-unitcontextvalidator}

#### `UNIVERSAL_UNIT`
```php
private const UNIVERSAL_UNIT = 'pcs';
```
The universal unit that can be converted to any other quantity unit.

#### `CONVERSION_RULES`
An array of specific rules defining logical relationships between specific pairs of units. Each rule contains:
- Source unit
- Target unit
- `true` if the conversion is logical, `false` if illogical
- A descriptive reason

```php
private const CONVERSION_RULES = [
    ['dozen', 'box', true, 'Packaging to packaging'],
    ['box', 'dozen', true, 'Packaging to packaging'],
    ['pair', 'dozen', true, 'Pairs to packaging'],
    ['dozen', 'pair', true, 'Packaging to pairs'],
    ['ream', 'dozen', false, 'Paper to packaging (illogical)'],
    // ...
];
```

### Main Functions {#main-functions-unitcontextvalidator}

#### `isConversionLogical(string $from, string $to): bool`

**Description:**  
Check if a conversion between two units is logical.

**Parameters:**
- `$from` : Source unit symbol.
- `$to` : Target unit symbol.

**Returns:**  
`true` if the conversion is logical, `false` if illogical.

**Workflow:**
1. If one of the units is `pcs`, the conversion is always considered logical.
2. Searches `CONVERSION_RULES` for an exact match of the pair `($from, $to)`. If found, returns the recorded value.
3. If not found, extracts the context of each unit via `UnitNameHelper::getUnitInfo`. If the contexts match, the conversion is considered logical. Otherwise, it is illogical.

**Examples:**
```php
// Logical conversions
UnitContextValidator::isConversionLogical('dozen', 'box'); // true
UnitContextValidator::isConversionLogical('pair', 'dozen'); // true
UnitContextValidator::isConversionLogical('pcs', 'ream'); // true (because pcs is universal)

// Illogical conversions
UnitContextValidator::isConversionLogical('dozen', 'ream'); // false
UnitContextValidator::isConversionLogical('ream', 'pair'); // false
```

#### `getConversionReason(string $from, string $to): string`

**Description:**  
Return the reason for the logicality or illogicality of a conversion as a descriptive text.

**Parameters:**
- `$from` : Source unit symbol.
- `$to` : Target unit symbol.

**Returns:**  
A string explaining the reason.

**Examples:**
```php
echo UnitContextValidator::getConversionReason('dozen', 'box');
// "Same context: Packaging quantities"

echo UnitContextValidator::getConversionReason('dozen', 'ream');
// "Conversion between different contexts: Packaging quantities to Paper quantities"

echo UnitContextValidator::getConversionReason('pcs', 'dozen');
// "Piece is the base unit for quantities"
```

#### `getCompatibleUnits(string $unit): array`

**Description:**  
Return a list of logically compatible units with a given unit (excluding the unit itself). Each element in the array is an array containing `unit` (symbol) and `label`.

**Parameters:**
- `$unit` : Unit symbol.

**Returns:**  
An array of compatible units.

**Example:**
```php
$compatible = UnitContextValidator::getCompatibleUnits('dozen');
foreach ($compatible as $item) {
    echo "{$item['unit']} : {$item['label']}\n";
}
// box : Box
// carton : Carton
// half_dozen : Half Dozen
// pack : Pack
// tray : Tray
// pair : Pair
// pcs : Piece
// ...
```

### Internal Helper Functions

- `getUnitContext(string $unit): ?string` : Gets the unit context using `UnitNameHelper`.
- `getContextLabel(?string $context): string` : Gets the Arabic label for the context.

### Examples {#examples-unitcontextvalidator}

#### Example 1: Checking Conversion Logicality Before Performing It
```php
$from = 'dozen';
$to = 'ream';

if (UnitContextValidator::isConversionLogical($from, $to)) {
    $result = UnitConverter::convert(1, $from, $to);
    echo "Result: $result";
} else {
    echo "Conversion is illogical: " . UnitContextValidator::getConversionReason($from, $to);
}
```

#### Example 2: Getting Compatible Units for Display in UI
```php
$unit = 'box';
$compatible = UnitContextValidator::getCompatibleUnits($unit);

echo "<select name='to_unit'>";
foreach ($compatible as $item) {
    echo "<option value='{$item['unit']}'>{$item['label']}</option>";
}
echo "</select>";
```

#### Example 3: Using Custom Rules
```php
// New rules can be added in future extensions
```

---

## SmartUnitConverter - Smart Converter

**Namespace:** `Tss\Inventory\Classes\Units`

### Introduction {#introduction-smartunitconverter}

The `SmartUnitConverter` class provides a safer and smarter interface for conversion. It checks conversion logicality before performing it and can return warnings instead of throwing exceptions. Suitable for use in applications that need to guide the user or prevent illogical conversions.

### Main Functions {#main-functions-smartunitconverter}

#### `safeConvert(float $value, string $from, string $to, int $precision = 6): float`

**Description:**  
Safe conversion with logical validation. If the conversion is illogical, an exception is thrown.

**Parameters:**
- `$value` : The value to convert.
- `$from` : Source unit symbol.
- `$to` : Target unit symbol.
- `$precision` : Number of decimal places.

**Returns:**  
The converted value (float).

**Exceptions:**  
`InvalidArgumentException` if the conversion is illogical (according to `UnitContextValidator::isConversionLogical`).

**Example:**
```php
try {
    $result = SmartUnitConverter::safeConvert(2, 'dozen', 'box');
    echo "2 dozen = $result box";
} catch (InvalidArgumentException $e) {
    echo "Error: " . $e->getMessage();
}
```

#### `convertWithWarning(float $value, string $from, string $to, int $precision = 6): array`

**Description:**  
Convert and return a warning if the conversion is illogical. Does not throw an exception; instead returns an array containing the result and additional information.

**Parameters:**
- `$value` : The value to convert.
- `$from` : Source unit symbol.
- `$to` : Target unit symbol.
- `$precision` : Number of decimal places.

**Returns:**  
An array containing:
- `value` : The converted value.
- `unit` : Target unit symbol.
- `unit_label` : Target unit label.
- `is_logical` : `true` if the conversion is logical, `false` if illogical.
- `warning` : Warning text (if conversion is illogical) or `null`.
- `description` : Conversion description from `UnitConverter::describeConversion`.

**Example:**
```php
$result = SmartUnitConverter::convertWithWarning(1, 'dozen', 'ream', 2);

echo $result['description']; // "1 dozen = 0.04 ream"
if (!$result['is_logical']) {
    echo "Warning: " . $result['warning'];
}
// Warning: ⚠️ Warning: Converting from dozen to ream may not be logical in practical applications
```

### Examples {#examples-smartunitconverter}

#### Example 1: Using safeConvert with Validation
```php
function convertForUser($value, $from, $to) {
    try {
        $result = SmartUnitConverter::safeConvert($value, $from, $to, 2);
        return [
            'success' => true,
            'result' => $result,
            'message' => UnitConverter::describeConversion($value, $from, $to, 2)
        ];
    } catch (InvalidArgumentException $e) {
        return [
            'success' => false,
            'message' => $e->getMessage()
        ];
    }
}

$output = convertForUser(3, 'dozen', 'box');
if ($output['success']) {
    echo $output['message'];
} else {
    echo "Sorry, this conversion cannot be performed: " . $output['message'];
}
```

#### Example 2: Using convertWithWarning to Provide Recommendations
```php
$from = 'dozen';
$to = 'ream';
$value = 5;

$result = SmartUnitConverter::convertWithWarning($value, $from, $to, 2);

if ($result['is_logical']) {
    echo "Result: {$result['description']}";
} else {
    echo "Caution: {$result['warning']}\n";
    echo "However, you can perform the conversion if you wish: {$result['description']}\n";
    echo "Logically compatible units: ";
    $compatible = UnitContextValidator::getCompatibleUnits($from);
    $labels = array_column($compatible, 'label', 'unit');
    echo implode(', ', array_slice($labels, 0, 3)) . "...";
}
```

#### Example 3: Combining with UnitConverter and UnitContextValidator
```php
class ConversionService
{
    public function convert($value, $from, $to, $mode = 'safe')
    {
        if (!UnitConverter::isUnitSupported($from) || !UnitConverter::isUnitSupported($to)) {
            throw new InvalidArgumentException("Unsupported unit");
        }
        
        if ($mode === 'safe') {
            return SmartUnitConverter::safeConvert($value, $from, $to);
        } elseif ($mode === 'warning') {
            return SmartUnitConverter::convertWithWarning($value, $from, $to);
        } else {
            return UnitConverter::convert($value, $from, $to);
        }
    }
}
```

---

## Conclusion

- **`UnitConverter`** is the basic converter that contains all conversion data and performs the mathematical operations. It provides comprehensive functions for conversion and querying.
- **`UnitContextValidator`** adds a layer of logicality to conversions between quantity units, preventing illogical conversions and helping build smarter applications.
- **`SmartUnitConverter`** offers an easy and safe interface that combines the power of `UnitConverter` with the logicality of `UnitContextValidator`, with options to handle warnings instead of exceptions.

By using these classes together, you can build an integrated unit conversion system that supports high precision, logicality, and flexibility in handling various use cases.
