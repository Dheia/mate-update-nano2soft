# `UnitHelper` Class Documentation

**Namespace:** `Tss\Inventory\Classes\Units`

---

## 📋 Table of Contents

1. [Introduction](#introduction)
2. [Main Functions](#main-functions)
   - [safeConvert](#safeconvert)
   - [formatWithUnit](#formatwithunit)
   - [getBestDisplayUnit](#getbestdisplayunit)
   - [generateSelectOptions](#generateselectoptions)
   - [generateGroupedSelectOptions](#generategroupedselectoptions)
   - [getRentalTimeUnits](#getrentaltimeunits)
   - [getConversionFactor](#getconversionfactor)
   - [getConversionFactorFormatted](#getconversionfactorformatted)
   - [formatDecimal](#formatdecimal)
3. [Comprehensive Examples](#comprehensive-examples)

---

## Introduction

The `UnitHelper` class is a helper class that provides general functions to facilitate working with units in applications. This class offers functions for safe conversion, formatting values with units, selecting the best display unit, generating simplified dropdown list options, and handling rental units. The class relies on other classes (`UnitNameHelper`, `UnitConverter`, `UnitContextValidator`) to provide integrated functionality.

---

## Main Functions

### `safeConvert`

```php
public static function safeConvert(
    float $value,
    string $from,
    string $to,
    int $precision = 6,
    bool $validateLogic = true
): ?float
```

Description:
Convert a value from one unit to another with error handling and the option to validate conversion logicality. It does not throw exceptions but returns null on failure.

Parameters:

· $value : The value to be converted.
· $from : Source unit symbol.
· $to : Target unit symbol.
· $precision : Number of decimal places (default 6).
· $validateLogic : If true, checks the logicality of the conversion using UnitContextValidator::isConversionLogical. If the conversion is illogical, returns null.

Returns:
The converted value (float) or null in case of an error (unsupported unit, impossible conversion, or illogical conversion).

Example:

```php
$result = UnitHelper::safeConvert(5, 'kg', 'lb', 2, true);
if ($result !== null) {
    echo "Result: $result";
} else {
    echo "Conversion cannot be performed";
}
```

---

formatWithUnit

```php
public static function formatWithUnit(float $value, string $unit, int $decimals = 2, $lang = 'en'): string
```

Description:
Format a numeric value with the appropriate unit label (by language) to obtain a ready-to-display string.

Parameters:

· $value : The numeric value.
· $unit : Unit symbol.
· $decimals : Number of decimal places (default 2).
· $lang : Language ('en' or 'ar').

Returns:
A string like "100.00 Kilogram" or "2.50 Meter".

Example:

```php
echo UnitHelper::formatWithUnit(2.5, 'kg', 3, 'en'); // "2.500 Kilogram"
echo UnitHelper::formatWithUnit(100, 'cm', 0, 'en'); // "100 Centimeter"
```

---

getBestDisplayUnit

```php
public static function getBestDisplayUnit(float $value, string $currentUnit, array $preferredUnits = [], $lang = 'en'): string
```

Description:
Find the best unit to display a given value, such that the converted value is between 1 and 1000. Useful for choosing the appropriate unit (e.g., displaying 1500 meters as 1.5 km).

Parameters:

· $value : The value in $currentUnit.
· $currentUnit : The current unit symbol.
· $preferredUnits : An array of preferred unit symbols (optional). If provided, the search is limited to them.
· $lang : Language (used only to fetch unit information, but the result is the unit symbol).

Returns:
The best unit symbol for display. If no suitable unit is found, returns $currentUnit.

Example:

```php
$best = UnitHelper::getBestDisplayUnit(1500, 'm', ['km', 'm'], 'en');
echo $best; // 'km'
```

---

generateSelectOptions

```php
public static function generateSelectOptions(string $type, string $selected = '', bool $includeEmptyOption = true, $lang = 'en'): string
```

Description:
Generate HTML <option> code for a dropdown list containing units of a specific type.

Parameters:

· $type : Unit type (one of the UnitType constants).
· $selected : The symbol of the unit that should be pre-selected (selected attribute added).
· $includeEmptyOption : Add an empty option (<option value="">-- Select unit --</option>) at the beginning.
· $lang : Language.

Returns:
HTML string containing <option> elements.

Example:

```php
echo '<select name="weight_unit">';
echo UnitHelper::generateSelectOptions(UnitType::WEIGHT, 'kg', true, 'en');
echo '</select>';
```

---

generateGroupedSelectOptions

```php
public static function generateGroupedSelectOptions(string $selected = '', bool $includeEmptyOption = true, $lang = 'en'): string
```

Description:
Generate <select> options grouped by type using <optgroup>, suitable for displaying all units in an organized manner.

Parameters:

· $selected : The selected unit symbol.
· $includeEmptyOption : Add an empty option.
· $lang : Language.

Returns:
HTML string with <optgroup> groups.

Example:

```php
echo '<select name="unit">';
echo UnitHelper::generateGroupedSelectOptions('kg', true, 'en');
echo '</select>';
```

---

getRentalTimeUnits

```php
public static function getRentalTimeUnits($lang = 'en', $is_key = false): array
```

Description:
Return time units used in the rental context (hour, day, week, month). These units are the most common in rental systems.

Parameters:

· $lang : Language.
· $is_key : If true, returns only the list of symbols (['h','day','week','month']). If false, returns an associative array [symbol => label].

Returns:
An array according to $is_key.

Example:

```php
$units = UnitHelper::getRentalTimeUnits('en');
// ['h' => 'Hour', 'day' => 'Day', 'week' => 'Week', 'month' => 'Month']

$keys = UnitHelper::getRentalTimeUnits('en', true);
// ['h', 'day', 'week', 'month']
```

---

getConversionFactor

```php
public static function getConversionFactor(string $unit, $defaultValue = 0.0): float
```

Description:
Get the conversion factor for a unit (relative to the base unit of its type). In case of an error, returns the default value.

Parameters:

· $unit : Unit symbol.
· $defaultValue : Default value (default 0.0).

Returns:
Conversion factor as a float.

Example:

```php
$factor = UnitHelper::getConversionFactor('km'); // 1000.0
$factor = UnitHelper::getConversionFactor('xyz', 1); // 1.0 (because the unit is unknown)
```

---

getConversionFactorFormatted

```php
public static function getConversionFactorFormatted(string $unit, $defaultValue = 0.0, int $decimals = 10, $isFormatDecimal = true): string
```

Description:
Get the conversion factor formatted as a string, with trailing zeros removed (optional). Suitable for displaying in input fields.

Parameters:

· $unit : Unit symbol.
· $defaultValue : Default value.
· $decimals : Number of decimal places (default 10).
· $isFormatDecimal : If true, formatDecimal is applied to the output.

Returns:
A string representing the conversion factor.

Example:

```php
echo UnitHelper::getConversionFactorFormatted('kg'); // "1"
echo UnitHelper::getConversionFactorFormatted('lb', 0, 6); // "0.453592"
```

---

formatDecimal

```php
public static function formatDecimal($value, int $decimals = 12): string
```

Description:
Format a decimal number by removing trailing zeros from the right, and removing the decimal point if no digits remain. Useful for displaying numbers cleanly.

Parameters:

· $value : The number (float or string).
· $decimals : Maximum number of decimal places allowed (default 12).

Returns:
A string without trailing zeros.

Example:

```php
echo UnitHelper::formatDecimal(12.3000, 4); // "12.3"
echo UnitHelper::formatDecimal(5.0, 2);      // "5"
echo UnitHelper::formatDecimal(3.14159, 3);  // "3.142"
```

---

Comprehensive Examples

Example 1: Safe Conversion and Displaying the Result

```php
$value = 100;
$from = 'cm';
$to = 'm';

$result = UnitHelper::safeConvert($value, $from, $to, 2, true);
if ($result !== null) {
    echo UnitHelper::formatWithUnit($result, $to, 2, 'en');
} else {
    echo "Cannot convert $value $from to $to";
}
```

Example 2: Choosing the Best Unit to Display Length

```php
$lengthInMeters = 3456; // meters
$bestUnit = UnitHelper::getBestDisplayUnit($lengthInMeters, 'm', ['km', 'm'], 'en');
$converted = UnitConverter::convert($lengthInMeters, 'm', $bestUnit, 2);
echo "Best display: " . UnitHelper::formatWithUnit($converted, $bestUnit, 2, 'en');
// Output for example: "3.46 km"
```

Example 3: Generating a Dropdown List for Rental Units

```php
$rentalUnits = UnitHelper::getRentalTimeUnits('en');
echo '<select name="rental_period">';
foreach ($rentalUnits as $unit => $label) {
    echo "<option value=\"$unit\">$label</option>";
}
echo '</select>';
```

Example 4: Using formatDecimal in Input Fields

```php
$conversionFactor = UnitHelper::getConversionFactor('kg'); // 1.0
echo '<input type="text" value="' . UnitHelper::formatDecimal($conversionFactor, 5) . '">';
// <input type="text" value="1">
```

---

This covers all functions of the UnitHelper class with detailed explanations and practical examples. This class helps simplify day-to-day unit-related operations, especially in user interfaces and safe conversions.