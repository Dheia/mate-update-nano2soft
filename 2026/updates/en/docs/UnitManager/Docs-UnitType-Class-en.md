# Units System Classes Documentation

## 📋 Table of Contents

1.  [UnitType - Unit Type Definition](#unittype---unit-type-definition)
    - [Constants](#constants)
    - [Main Functions for UnitType](#main-functions-for-unittype)
    - [UnitType Examples](#unittype-examples)
2.  [Unit - Unit Object](#unit---unit-object)
    - [Properties](#properties)
    - [Main Functions for Unit](#main-functions-for-unit)
    - [Unit Examples](#unit-examples)
3.  [UnitFactory - Unit Factory](#unitfactory---unit-factory)
    - [Main Functions for UnitFactory](#main-functions-for-unitfactory)
    - [UnitFactory Examples](#unitfactory-examples)

---

## UnitType - Unit Type Definition

**Namespace:** `Tss\Inventory\Classes\Units`

### Introduction
A static class used to define and classify the unit types supported in the system. It provides constants for main and subtype units, as well as helper functions to validate types and retrieve their translated labels.

### Constants

| Constant | Value | Description |
|--------|--------|-------|
| `LENGTH` | `'length'` | Length |
| `WEIGHT` | `'weight'` | Weight |
| `VOLUME` | `'volume'` | Volume |
| `AREA` | `'area'` | Area |
| `TEMPERATURE` | `'temperature'` | Temperature |
| `TIME` | `'time'` | Time |
| `SPEED` | `'speed'` | Speed |
| `PRESSURE` | `'pressure'` | Pressure |
| `ENERGY` | `'energy'` | Energy |
| `POWER` | `'power'` | Power |
| `DATA` | `'data'` | Data |
| `ANGLE` | `'angle'` | Angles |
| `QUANTITY` | `'quantity'` | Quantities (main type) |
| `QUANTITY_GENERAL` | `'quantity_general'` | General quantities |
| `QUANTITY_PACKAGING` | `'quantity_packaging'` | Packaging quantities |
| `QUANTITY_PAPER` | `'quantity_paper'` | Paper quantities |
| `QUANTITY_PAIRS` | `'quantity_pairs'` | Pair quantities |
| `QUANTITY_LARGE` | `'quantity_large'` | Large quantities |
| `CUSTOM` | `'custom'` | Custom units |

### Main Functions for UnitType

#### `getMainTypesWithLabels($lang = null): array`
Returns main types with their labels.

- **Parameters:**
  - `$lang` : Language (`'en'` or `'ar'`). If not passed, the current language is used.
- **Returns:** Associative array `[type => label]`.

**Example:**
```php
$types = UnitType::getMainTypesWithLabels('en');
/*
[
    'length' => 'Length',
    'weight' => 'Weight',
    'volume' => 'Volume',
    ...
]
*/
```

#### `getAllTypesWithLabels($lang = null): array`
Returns all types (including subtypes) with their labels.

#### `getQuantitySubtypesWithLabels($lang = null): array`
Returns only quantity subtypes with labels.

#### `getTypesWithLabels(array $types, $lang = null): array`
Returns labels for specific types.

- **Parameters:**
  - `$types` : An array of type names.
- **Returns:** Associative array `[type => label]` for existing types only.

#### `getCategorizedTypes($lang = null): array`
Returns types categorized by class (main, quantity, subtypes, custom).

**Example:**
```php
$categorized = UnitType::getCategorizedTypes('en');
/*
[
    'main' => ['length' => 'Length', ...],
    'quantity' => ['quantity' => 'Quantities'],
    'quantity_subtypes' => ['quantity_general' => 'General Quantities', ...],
    'custom' => ['custom' => 'Custom Units'],
]
*/
```

#### `isValid(string $type): bool`
Checks if the type is defined (one of the constants).

- **Returns:** `true` if the type is valid.

#### `isQuantitySubtype(string $type): bool`
Checks if the type is a quantity subtype (starts with `quantity_`).

#### `isCustomType(string $type): bool`
Checks if the type is custom (equals `custom` or starts with `custom_`).

#### `getMainType(string $type): string`
Returns the main type for the given type.

- For quantity subtypes, returns `QUANTITY`.
- For custom types, returns `CUSTOM`.
- Otherwise, returns the type itself.

#### `getArabicLabel(string $type): string`
Returns the default Arabic label for the type (without translation from files).

#### `getTransLabel(string $type, $lang = null): string`
Returns the translated label for the type based on the language. It searches translation files first, then falls back to the default Arabic or English label.

#### `getEnglishLabel(string $type): string`
Returns the default English label.

#### `getAllTypes(): array`
Returns a list of all types (constant values).

#### `getMainTypes(): array`
Returns a list of main types only.

#### `getSelectOptions(string $selectedType = '', bool $includeEmpty = true, bool $onlyMainTypes = false): string`
Generates HTML `<select>` options for types.

- **Parameters:**
  - `$selectedType` : The pre-selected type (gets `selected` attribute).
  - `$includeEmpty` : Add an empty option at the beginning.
  - `$onlyMainTypes` : Show only main types (if `false`, shows all including subtypes).
- **Returns:** HTML code for the options.

#### `getQuantitySubtypesSelectOptions(string $selectedType = '', bool $includeEmpty = true): string`
Generates HTML options for quantity subtypes only.

#### `filterByPrefix(string $prefix, bool $withLabels = true, $lang = null)`
Returns types that start with a given prefix.

- **Parameters:**
  - `$prefix` : The prefix (e.g., `'quantity_'`).
  - `$withLabels` : If `true`, returns an associative array `[type => label]`; otherwise returns a simple list.
- **Returns:** An array.

### UnitType Examples

```php
// 1. Check type
if (UnitType::isValid('weight')) {
    echo "Valid type";
}

// 2. Get translated label
echo UnitType::getTransLabel('length', 'en'); // "Length"

// 3. Get main type for a subtype
$main = UnitType::getMainType('quantity_packaging'); // "quantity"

// 4. Create a dropdown list for main types
echo '<select name="type">';
echo UnitType::getSelectOptions('weight', true, true);
echo '</select>';
```

---

## Unit - Unit Object

**Namespace:** `Tss\Inventory\Classes\Units`

### Introduction
This object represents a single unit of measurement with its properties (symbol, type, label, context) and provides functions for conversion, comparison, and querying. It is typically created via `UnitFactory`.

### Properties

| Property | Type | Description |
|---------|-------|-------|
| `symbol` | `string` | Unit symbol (e.g., `'kg'`) |
| `type` | `string` | Unit type (one of the `UnitType` constants) |
| `label` | `string` | Arabic label for the unit |
| `context` | `string\|null` | Context (for quantity units: `general`, `packaging`, ...) |

All properties are read-only.

### Main Functions for Unit

#### `__construct(string $symbol, string $type, string $label, ?string $context = null)`
Creates a new unit object. Internally calls `UnitNameHelper::getUnitInfo` to store additional information.

#### `convertTo(float $value, string $toUnit, int $precision = 6): float`
Converts a value from this unit to another unit.

- **Parameters:**
  - `$value` : The value to be converted.
  - `$toUnit` : The target unit symbol.
  - `$precision` : Number of decimal places (default 6).
- **Returns:** The converted value.
- **Exceptions:** Throws `InvalidArgumentException` if the target unit is unsupported or conversion is not possible.

**Example:**
```php
$kg = UnitFactory::createFromString('kg');
$grams = $kg->convertTo(5, 'g', 2); // 5000.00
```

#### `convertToBase(float $value, int $precision = 6): float`
Converts the value to the base unit of this type.

- **Returns:** The value in the base unit.
- **Exceptions:** Throws `InvalidArgumentException` if no base unit exists for the type.

**Example:**
```php
$dozen = UnitFactory::createFromString('dozen');
$pieces = $dozen->convertToBase(2, 2); // 24.00 (because the base is pcs)
```

#### `isCompatibleWith(string $otherUnit): bool`
Checks if this unit is compatible with another unit (i.e., can be logically converted between them).

- **Parameters:**
  - `$otherUnit` : The other unit's symbol.
- **Returns:** `true` if both units are of the same main type.

**Example:**
```php
$meter = UnitFactory::createFromString('m');
if ($meter->isCompatibleWith('cm')) {
    echo "Conversion possible";
}
```

#### `getConversionFactor(): float`
Returns the conversion factor for this unit relative to the base unit of its type.

**Example:**
```php
$km = UnitFactory::createFromString('km');
echo $km->getConversionFactor(); // 1000.0
```

#### `isBaseUnit(): bool`
Checks if this unit is the base unit for its type.

#### `getEnglishLabel(): string`
Returns the English label for the unit.

#### `getInfo(): array`
Returns complete information about the unit (by calling `UnitNameHelper::getUnitInfo`).

#### `getContext(): ?string`
Returns the context (for quantity units).

#### `getCompatibleUnits(): array`
Returns an array of compatible unit symbols (of the same type) with their labels.

- **Returns:** Associative array `[symbol => label]`.

#### `canConvertTo(string $toUnit): bool`
Checks if conversion to a specific unit is possible (without performing the actual conversion).

#### `formatValue(float $value, int $decimals = 2): string`
Formats the value with the Arabic unit label.

- **Returns:** A formatted string like `"5.00 Kilogram"`.

#### `__toString(): string`
String representation of the unit in the format `"symbol (label)"`.

**Example:**
```php
echo $kg; // "kg (Kilogram)"
```

#### `equals(Unit $other): bool`
Compares this unit with another unit (based on symbol and type).

#### `copy(): self`
Creates a new copy of the unit.

### Unit Examples

```php
// Create a unit object via the factory
$meter = UnitFactory::createFromString('m');

// Convert 100 meters to centimeters
$cm = $meter->convertTo(100, 'cm'); // 10000

// Check compatibility
if ($meter->isCompatibleWith('km')) {
    echo "Yes";
}

// Get full information
$info = $meter->getInfo();
echo $info['base_unit']; // 'm'

// Format a value
echo $meter->formatValue(3.5, 2); // "3.50 Meter"

// Use the unit in a calculation
$lengthInKm = $meter->convertToBase(5000) / 1000; // 5 km
```

---

## UnitFactory - Unit Factory

**Namespace:** `Tss\Inventory\Classes\Units`

### Introduction
A class responsible for creating `Unit` objects in various ways: from a unit symbol, from a type, or from a context. It ensures that the objects it creates are valid (validates the symbol).

### Main Functions for UnitFactory

#### `createFromString(string $unitString): Unit`
Creates a `Unit` object from a unit symbol.

- **Parameters:**
  - `$unitString` : The unit symbol (e.g., `'kg'`, `'m'`, `'dozen'`).
- **Returns:** A `Unit` object.
- **Exceptions:** Throws `InvalidArgumentException` if the symbol is unsupported.

**Example:**
```php
$kg = UnitFactory::createFromString('kg');
echo $kg->label; // "Kilogram"
```

#### `createUnitsByType(string $type): array`
Creates an array of `Unit` objects for all units belonging to a specific type.

- **Parameters:**
  - `$type` : The unit type (one of the `UnitType` constants).
- **Returns:** An array of `Unit` objects.

**Example:**
```php
$weightUnits = UnitFactory::createUnitsByType(UnitType::WEIGHT);
foreach ($weightUnits as $unit) {
    echo $unit->symbol . ' - ' . $unit->label . "\n";
}
```

#### `createAllUnits(): array`
Creates an array of `Unit` objects for all supported units in the system.

#### `createBaseUnits(): array`
Creates an array of `Unit` objects for the base units of each type (e.g., `m` for length, `kg` for weight). The array is associative `[type => Unit]`.

**Example:**
```php
$baseUnits = UnitFactory::createBaseUnits();
$baseLength = $baseUnits[UnitType::LENGTH]; // Meter unit
```

#### `createUnitByContext(string $context): ?Unit`
Creates a base unit for a specific quantity context.

- **Parameters:**
  - `$context` : The context (`'general'`, `'packaging'`, `'paper'`, `'pairs'`, `'large'`).
- **Returns:** A `Unit` object or `null` if the context is unknown.

**Supported Contexts:**
- `'general'` ← `pcs` (piece)
- `'packaging'` ← `dozen` (dozen)
- `'paper'` ← `ream` (ream)
- `'pairs'` ← `pair` (pair)
- `'large'` ← `hundred` (hundred)

**Example:**
```php
$packagingUnit = UnitFactory::createUnitByContext('packaging');
echo $packagingUnit->symbol; // "dozen"
```

#### `createCompatibleUnits(string $unitSymbol): array`
Creates an array of `Unit` objects for units compatible with a given unit (of the same type, excluding the unit itself).

- **Parameters:**
  - `$unitSymbol` : The unit symbol.
- **Returns:** An array of `Unit` objects.

**Example:**
```php
$compatible = UnitFactory::createCompatibleUnits('m');
foreach ($compatible as $unit) {
    echo $unit->symbol . ' '; // cm, km, mm, ...
}
```

### UnitFactory Examples

```php
// 1. Create a unit from a symbol
try {
    $unit = UnitFactory::createFromString('xyz');
} catch (InvalidArgumentException $e) {
    echo "Unsupported unit";
}

// 2. Create all length units
$lengthUnits = UnitFactory::createUnitsByType(UnitType::LENGTH);
echo count($lengthUnits); // Number of length units

// 3. Create a unit from context
$pairUnit = UnitFactory::createUnitByContext('pairs');
echo $pairUnit->label; // "Pair"

// 4. Create units compatible with kilogram
$kg = UnitFactory::createFromString('kg');
$compatible = UnitFactory::createCompatibleUnits($kg->symbol);
```

---

## Best Practices

1.  **Use `UnitFactory` to create unit objects** instead of manually instantiating them with `new Unit(...)`, because the factory validates the symbol.
2.  **When multiple conversions are needed**, create a `Unit` object once and reuse it instead of creating a new object for each conversion.
3.  **To get detailed information about a unit**, use `getInfo()` instead of relying only on the basic properties.
4.  **Before converting**, you can use `canConvertTo()` to check if conversion is possible without throwing an exception.
5.  **Use `isCompatibleWith()`** when you want to check logical compatibility (same main type) before performing a conversion.

---

This covers the essential aspects of the three classes. For more details, please refer to the source code and inline comments.