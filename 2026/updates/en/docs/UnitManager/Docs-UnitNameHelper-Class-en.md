# `UnitNameHelper` Class Documentation

**Namespace:** `Tss\Inventory\Classes\Units`

---

## 📋 Table of Contents

1. [Introduction](#introduction)
2. [Constants](#constants)
3. [Main Functions](#main-functions)
   - [getAllUnitsWithLabels](#getallunitswithlabels)
   - [getAllUnitsFilterWithLabels](#getallunitsfilterwithlabels)
   - [getUnitsByTypeWithLabels](#getunitsbytypewithlabels)
   - [getUnitLabel](#getunitlabel)
   - [getEnglishLabel](#getenglishlabel)
   - [getCategorizedUnits](#getcategorizedunits)
   - [getSelectOptions](#getselectoptions)
   - [getGroupedSelectOptions](#getgroupedselectoptions)
   - [searchUnits](#searchunits)
   - [searchUnitAdvanced](#searchunitadvanced)
   - [getUnitType](#getunittype)
   - [isValidUnit](#isvalidunit)
   - [getSimilarUnits](#getsimilarunits)
   - [getUnitsByContext](#getunitsbycontext)
   - [getBaseUnits](#getbaseunits)
   - [getBaseUnitByType](#getbaseunitbytype)
   - [getBaseUnitInfoByType](#getbaseunitinfobytype)
   - [getBaseUnitWithLabel](#getbaseunitwithlabel)
   - [getAllBaseUnitsForMainTypes](#getallbaseunitsformaintypes)
   - [getBaseUnitsForQuantitySubtypes](#getbaseunitsforquantitysubtypes)
   - [isBaseUnitForItsType](#isbaseunitforitstype)
   - [filterBaseUnits](#filterbaseunits)
   - [convertToBaseUnit](#converttobaseunit)
   - [getBaseUnitsList](#getbaseunitslist)
   - [getUnitInfo](#getunitinfo)
   - [getUnitFullInfo](#getunitfullinfo)
4. [Internal Helper Functions](#internal-helper-functions)
5. [Comprehensive Examples](#comprehensive-examples)

---

## Introduction

The `UnitNameHelper` class is the central manager for unit names and labels in the `Tss.Inventory` system. It provides a unified programming interface for accessing information about supported units, such as obtaining Arabic and English labels, searching for units, categorizing them by type, and extracting detailed information (symbol, type, context, whether it's a base unit, ...). This class relies on internal data (arrays) and supports translation via language files, making it an essential tool for building user interfaces and validating input.

---

## Constants

| Constant | Value | Description |
|--------|--------|-------|
| `LOCALIZATION_KEY` | `'tss.inventory::lang.units.'` | Language file key for units |

---

## Main Functions

### `getAllUnitsWithLabels`

```php
public static function getAllUnitsWithLabels($lang = null): array
```

Description:
Returns a list of all supported units in the system with their labels according to the requested language.

Parameters:

· $lang : (Optional) Language code ('en' or 'ar'). If not passed, the current application language is used.

Returns:
An associative array [unit_symbol => label].

Example:

```php
$allUnits = UnitNameHelper::getAllUnitsWithLabels('en');
// ['mm' => 'Millimeter', 'cm' => 'Centimeter', 'm' => 'Meter', ...]
```

---

getAllUnitsFilterWithLabels

```php
public static function getAllUnitsFilterWithLabels($isBaseUnit = null, $lang = null): array
```

Description:
Returns a list of all units with the option to filter them by whether they are base units or not.

Parameters:

· $isBaseUnit : (Optional) If true, returns only base units. If false, returns non-base units. If null, returns all.
· $lang : Language.

Returns:
An associative array [unit_symbol => label] after filtering.

Example:

```php
$baseUnits = UnitNameHelper::getAllUnitsFilterWithLabels(true, 'en');
// Only base units (e.g., 'm', 'kg', 'pcs')
```

---

getUnitsByTypeWithLabels

```php
public static function getUnitsByTypeWithLabels(string $type, $lang = null, $isBaseUnit = null): array
```

Description:
Returns a list of units belonging to a specific type (e.g., length, weight), with optional filtering for base units.

Parameters:

· $type : Unit type (one of the UnitType constants).
· $lang : Language.
· $isBaseUnit : (Optional) Filter by base status.

Returns:
An associative array [unit_symbol => label].

Example:

```php
$lengthUnits = UnitNameHelper::getUnitsByTypeWithLabels(UnitType::LENGTH, 'en');
/*
[
    'mm' => 'Millimeter',
    'cm' => 'Centimeter',
    'm' => 'Meter',
    'km' => 'Kilometer',
    'in' => 'Inch',
    'ft' => 'Foot',
    'yd' => 'Yard',
    'mile' => 'Mile',
    'nmi' => 'Nautical Mile'
]
*/
```

---

getUnitLabel

```php
public static function getUnitLabel(string $unit, $lang = null): string
```

Description:
Get the label (by language) for a given unit symbol.

Parameters:

· $unit : Unit symbol (e.g., 'kg').
· $lang : Language.

Returns:
The unit label, or the symbol itself if the label is not found.

Example:

```php
echo UnitNameHelper::getUnitLabel('kg', 'en'); // "Kilogram"
echo UnitNameHelper::getUnitLabel('kg', 'ar'); // "كيلوجرام"
```

---

getEnglishLabel

```php
public static function getEnglishLabel(string $unit): string
```

Description:
Returns the English label for the unit (from the internal list, regardless of translation).

Parameters:

· $unit : Unit symbol.

Returns:
The English label.

Example:

```php
echo UnitNameHelper::getEnglishLabel('dozen'); // "Dozen"
```

---

getCategorizedUnits

```php
public static function getCategorizedUnits($lang = null): array
```

Description:
Returns all units categorized by type, with additional information such as label (type label) and has_subtypes (for quantities).

Parameters:

· $lang : Language.

Returns:
A complex associative array, where the key is the unit type, and the value is an array containing:

· label : Type label.
· units : Array [symbol => label] of units of that type.
· has_subtypes : bool (specific to quantities).
· subtypes : (if present) Array of quantity subtypes.

Example:

```php
$categorized = UnitNameHelper::getCategorizedUnits('en');
foreach ($categorized as $type => $info) {
    echo $info['label'] . ":\n";
    foreach ($info['units'] as $symbol => $label) {
        echo "  $symbol : $label\n";
    }
}
```

---

getSelectOptions

```php
public static function getSelectOptions(?string $type = null, string $selectedUnit = '', bool $includeEmpty = true, $lang = null): string
```

Description:
Generates HTML <option> code for creating a dropdown list (select) containing units of a specific type (or all units).

Parameters:

· $type : Unit type (optional). If null, all units are shown.
· $selectedUnit : The symbol of the unit that should be pre-selected (selected attribute added).
· $includeEmpty : Add an empty option at the beginning.
· $lang : Language.

Returns:
HTML string for the options.

Example:

```php
echo '<select name="unit">';
echo UnitNameHelper::getSelectOptions('weight', 'kg', true, 'en');
echo '</select>';
// Output: <option value="">-- Select unit --</option>
//         <option value="mg">Milligram</option>
//         <option value="g">Gram</option>
//         <option value="kg" selected>Kilogram</option>
//         ...
```

---

getGroupedSelectOptions

```php
public static function getGroupedSelectOptions(string $selectedUnit = '', bool $includeEmpty = true, $lang = null): string
```

Description:
Generates <select> options grouped by type using <optgroup>.

Parameters:

· $selectedUnit : The selected unit.
· $includeEmpty : Add an empty option.
· $lang : Language.

Returns:
HTML string for options with grouping by type.

Example:

```php
echo '<select name="unit">';
echo UnitNameHelper::getGroupedSelectOptions('kg', true, 'en');
echo '</select>';
// Output: <option value="">-- Select unit --</option>
//         <optgroup label="Length">
//             <option value="mm">Millimeter</option> ...
//         </optgroup>
//         <optgroup label="Weight">
//             <option value="mg">Milligram</option>
//             <option value="g">Gram</option>
//             <option value="kg" selected>Kilogram</option> ...
//         </optgroup>
```

---

searchUnits

```php
public static function searchUnits(string $searchText, bool $withLabels = true, $lang = null)
```

Description:
Simple search for units that contain the given text in the symbol, Arabic label, or English label.

Parameters:

· $searchText : Search text.
· $withLabels : If true, returns an associative array [symbol => label]; otherwise returns a list of symbols only.
· $lang : Language.

Returns:
An array (depending on $withLabels).

Example:

```php
$results = UnitNameHelper::searchUnits('kilo', true, 'en');
// ['kg' => 'Kilogram', 'km' => 'Kilometer', 'kcal' => 'Kilocalorie', ...]
```

---

searchUnitAdvanced

```php
public static function searchUnitAdvanced(string $searchText, ?string $type = null, $lang = null): ?array
```

Description:
Advanced search that returns complete information about the first unit matching the text (in priority order: exact symbol match, then Arabic name, then English name, then partial search).

Parameters:

· $searchText : Search text.
· $type : (Optional) Restrict search to a specific type.
· $lang : Language.

Returns:
An array containing complete unit information (as in getUnitFullInfo) or null if nothing is found.

Example:

```php
$info = UnitNameHelper::searchUnitAdvanced('kilo');
if ($info) {
    echo $info['unit_name_en']; // "Kilogram" (first matching unit)
}
```

---

getUnitType

```php
public static function getUnitType(string $unit): ?string
```

Description:
Get the unit type (e.g., 'length').

Parameters:

· $unit : Unit symbol.

Returns:
The unit type as a string, or null if unknown.

Example:

```php
$type = UnitNameHelper::getUnitType('kg'); // 'weight'
```

---

isValidUnit

```php
public static function isValidUnit(string $unit): bool
```

Description:
Check if a unit symbol is known and supported in the system.

Parameters:

· $unit : Unit symbol.

Returns:
true if the unit exists, otherwise false.

Example:

```php
if (UnitNameHelper::isValidUnit('kg')) {
    echo "Unit is supported";
}
```

---

getSimilarUnits

```php
public static function getSimilarUnits(string $unit, $lang = null): array
```

Description:
Returns a list of similar units (of the same type) for a given unit.

Parameters:

· $unit : Unit symbol.
· $lang : Language.

Returns:
An associative array [symbol => label] of units of the same type (including the unit itself? - the code returns all, but it's better to exclude itself).

Example:

```php
$similar = UnitNameHelper::getSimilarUnits('m', 'en');
// ['mm' => 'Millimeter', 'cm' => 'Centimeter', 'm' => 'Meter', 'km' => 'Kilometer', ...]
```

---

getUnitsByContext

```php
public static function getUnitsByContext(string $context, bool $withLabels = true, $lang = null)
```

Description:
Returns units belonging to a specific context (for quantity units only).

Parameters:

· $context : Context ('general', 'packaging', 'paper', 'pairs', 'large').
· $withLabels : If true, returns an associative array; otherwise a list of symbols.
· $lang : Language.

Returns:
An array according to $withLabels.

Example:

```php
$packagingUnits = UnitNameHelper::getUnitsByContext('packaging', true, 'en');
// ['dozen' => 'Dozen', 'half_dozen' => 'Half Dozen', 'box' => 'Box', ...]
```

---

getBaseUnits

```php
public static function getBaseUnits($lang = null): array
```

Description:
Returns the base units for each type (e.g., m for length, kg for weight) with their labels.

Parameters:

· $lang : Language.

Returns:
An associative array [type => ['unit' => symbol, 'label' => label]].

Example:

```php
$baseUnits = UnitNameHelper::getBaseUnits('en');
// [
//     'length' => ['unit' => 'm', 'label' => 'Meter'],
//     'weight' => ['unit' => 'kg', 'label' => 'Kilogram'],
//     ...
// ]
```

---

getBaseUnitByType

```php
public static function getBaseUnitByType(string $type): ?string
```

Description:
Get the base unit symbol for a specific type.

Parameters:

· $type : Unit type.

Returns:
The base unit symbol or null if not found.

Example:

```php
$base = UnitNameHelper::getBaseUnitByType(UnitType::LENGTH); // 'm'
```

---

getBaseUnitInfoByType

```php
public static function getBaseUnitInfoByType(string $type, $lang = null): ?array
```

Description:
Get complete information about the base unit for a specific type.

Parameters:

· $type : Unit type.
· $lang : Language.

Returns:
An array containing symbol, label, english_label, type, type_label or null.

---

getBaseUnitWithLabel

```php
public static function getBaseUnitWithLabel(string $type, $lang = null): array
```

Description:
Returns an array [symbol => label] for the base unit of a specific type.

Parameters:

· $type : Unit type.
· $lang : Language.

Returns:
An array containing one element (or empty).

Example:

```php
$base = UnitNameHelper::getBaseUnitWithLabel(UnitType::LENGTH, 'en'); // ['m' => 'Meter']
```

---

getAllBaseUnitsForMainTypes

```php
public static function getAllBaseUnitsForMainTypes($lang = null): array
```

Description:
Returns all base units for main types only (excluding subtypes).

Parameters:

· $lang : Language.

Returns:
An associative array [type => ['symbol' => symbol, 'label' => label, 'type_label' => type label]].

---

getBaseUnitsForQuantitySubtypes

```php
public static function getBaseUnitsForQuantitySubtypes($lang = null): array
```

Description:
Returns base units for quantity subtypes only.

Parameters:

· $lang : Language.

Returns:
An associative array similar to the previous one.

---

isBaseUnitForItsType

```php
public static function isBaseUnitForItsType(string $unit): bool
```

Description:
Checks if a unit is the base unit for its type.

Parameters:

· $unit : Unit symbol.

Returns:
true if it is base for its type.

Example:

```php
UnitNameHelper::isBaseUnitForItsType('m');   // true
UnitNameHelper::isBaseUnitForItsType('cm');  // false
```

---

filterBaseUnits

```php
public static function filterBaseUnits(array $units): array
```

Description:
Filters a list of unit symbols to keep only base units.

Parameters:

· $units : An array of unit symbols.

Returns:
An array containing only base symbols.

Example:

```php
$filtered = UnitNameHelper::filterBaseUnits(['m', 'cm', 'kg']); // ['m', 'kg']
```

---

convertToBaseUnit

```php
public static function convertToBaseUnit(string $unit): string
```

Description:
Converts a unit symbol to the base unit of its type. If the type is not found, returns the symbol itself.

Parameters:

· $unit : Unit symbol.

Returns:
The base unit symbol.

Example:

```php
$base = UnitNameHelper::convertToBaseUnit('cm'); // 'm'
```

---

getBaseUnitsList

```php
public static function getBaseUnitsList($lang = null): array
```

Description:
Returns a list of base units (each unit once) with their complete information.

Parameters:

· $lang : Language.

Returns:
An associative array [symbol => complete_info].

---

getUnitInfo

```php
public static function getUnitInfo(string $unit, $lang = null): ?array
```

Description:
Get basic information about a unit (symbol, label, type, whether it's base, context, conversion factor).

Parameters:

· $unit : Unit symbol.
· $lang : Language.

Returns:
An array containing symbol, label, english_label, type, type_label, is_base_unit, context, conversion_factor, conversion_factor_formatted or null.

Example:

```php
$info = UnitNameHelper::getUnitInfo('dozen', 'en');
/*
[
    'symbol' => 'dozen',
    'label' => 'Dozen',
    'english_label' => 'Dozen',
    'type' => 'quantity_packaging',
    'type_label' => 'Packaging Quantities',
    'is_base_unit' => true,
    'context' => 'packaging',
    'conversion_factor' => 12.0,
    'conversion_factor_formatted' => '12'
]
*/
```

---

getUnitFullInfo

```php
public static function getUnitFullInfo(string $unit, $lang = null): ?array
```

Description:
Get complete and detailed information about a unit, including type names, metric/imperial system, and display names.

Parameters:

· $unit : Unit symbol.
· $lang : Language.

Returns:
An array containing:

· unit_id : Unit symbol.
· unit_name_ar, unit_name_en : Names.
· type_id, type_name_ar, type_name_en : Type information.
· is_base_unit, base_unit, context.
· is_metric, is_imperial.
· display_name, full_name.

Example:

```php
$full = UnitNameHelper::getUnitFullInfo('kg', 'en');
/*
[
    'unit_id' => 'kg',
    'unit_name_ar' => 'كيلوجرام',
    'unit_name_en' => 'Kilogram',
    'type_id' => 'weight',
    'type_name_ar' => 'الوزن',
    'type_name_en' => 'Weight',
    'is_base_unit' => true,
    'base_unit' => 'kg',
    'context' => null,
    'is_metric' => true,
    'is_imperial' => false,
    'display_name' => 'Kilogram',
    'full_name' => 'Kilogram (Kilogram)'
]
*/
```

---

Internal Helper Functions

· getTransLabel(string $unit, string $defaultLabel, $lang): Gets the translated label from language files, otherwise returns the default label.
· getUnitContext(string $unit): Determines the unit context (for quantity units) based on an internal list.
· isMetricUnit(string $unit), isImperialUnit(string $unit): Checks the measurement system of the unit.

---

Comprehensive Examples

Example 1: Building a Grouped Dropdown List for a User Interface

```php
$html = '<form><select name="unit">';
$html .= UnitNameHelper::getGroupedSelectOptions('kg', true, 'en');
$html .= '</select></form>';
echo $html;
```

Example 2: Searching for a Unit and Displaying Its Information

```php
$search = 'dozen';
$info = UnitNameHelper::searchUnitAdvanced($search, null, 'en');
if ($info) {
    echo "Unit: {$info['unit_name_en']} ({$info['unit_id']})\n";
    echo "Type: {$info['type_name_en']}\n";
    echo "Is base? " . ($info['is_base_unit'] ? 'Yes' : 'No') . "\n";
    echo "Context: " . ($info['context'] ?? 'None') . "\n";
}
```

Example 3: Getting All Base Weight Units

```php
$weightBaseUnits = UnitNameHelper::getUnitsByTypeWithLabels(UnitType::WEIGHT, 'en', true);
// ['kg' => 'Kilogram'] (if kilogram is the base)
```

Example 4: Validating a Unit Symbol Before Use

```php
$unitCode = 'xyz';
if (!UnitNameHelper::isValidUnit($unitCode)) {
    throw new Exception("Unit not supported");
}
```

Example 5: Getting Complete Unit Information and Displaying It

```php
$unit = 'dozen';
$info = UnitNameHelper::getUnitFullInfo($unit, 'en');
if ($info) {
    echo $info['full_name'] . "\n"; // "Dozen (Dozen)"
    echo "Type: {$info['type_name_en']}\n";
    echo "System: " . ($info['is_metric'] ? 'Metric' : 'Imperial') . "\n";
}
```

---

This covers all functions of the UnitNameHelper class with detailed explanations and practical examples. This class is the primary source for everything related to unit names and classifications in the system.
