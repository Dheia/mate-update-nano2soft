# `UnitsInformation` Class Documentation

**Namespace:** `Tss\Inventory\Classes\Units`

---

## 📋 Table of Contents

1. [Introduction](#introduction)
2. [Public Static Methods](#public-static-methods)
   - [displayAllUnitsInfoV1](#displayallunitsinfov1)
   - [displayAllUnitsInfoBootstrap](#displayallunitsinfobootstrap)
   - [displayAllUnitsInfo](#displayallunitsinfo)
   - [displayTypeInfoBootstrap](#displaytypeinfobootstrap)
   - [displayTypeInfo](#displaytypeinfo)
3. [Private Static Methods](#private-static-methods)
4. [Comprehensive Examples](#comprehensive-examples)

---

## Introduction

The `UnitsInformation` class is responsible for generating HTML pages that display comprehensive information about all supported units in the system, including statistics, informational cards for each unit, and practical conversion examples. Its purpose is to provide a documentation and demonstration interface for developers and users to explore the capabilities of the unit system. Two display versions are available: one with a custom design (V1) and one compatible with Bootstrap (default).

---

## Public Static Methods

### `displayAllUnitsInfoV1`

```php
public static function displayAllUnitsInfoV1(): string
```

Description:
Generate a complete HTML page with a custom design (not reliant on Bootstrap) displaying all units categorized by type, including statistics, a search box, and cards containing unit information and conversion examples.

Returns:
HTML string for the complete page.

Example:

```php
echo UnitsInformation::displayAllUnitsInfoV1();
```

---

displayAllUnitsInfoBootstrap

```php
public static function displayAllUnitsInfoBootstrap(): string
```

Description:
Generate an HTML page with a Bootstrap design displaying all categorized units, including statistics, a search box, and collapsible cards containing information and examples. This version uses Bootstrap 5 and is responsive across different devices.

Returns:
HTML string for the page.

Example:

```php
echo UnitsInformation::displayAllUnitsInfoBootstrap();
```

---

displayAllUnitsInfo

```php
public static function displayAllUnitsInfo(): string
```

Description:
A backward-compatible function that calls displayAllUnitsInfoBootstrap().

Returns:
HTML string.

Example:

```php
echo UnitsInformation::displayAllUnitsInfo();
```

---

displayTypeInfoBootstrap

```php
public static function displayTypeInfoBootstrap(string $typeKey): string
```

Description:
Display information for a specific unit type only, with simple cards for each unit.

Parameters:

· $typeKey : Type key (e.g., 'length').

Returns:
HTML string containing cards for units of that type.

Example:

```php
echo UnitsInformation::displayTypeInfoBootstrap('weight');
```

---

displayTypeInfo

```php
public static function displayTypeInfo(string $typeKey): string
```

Description:
A backward-compatible function that calls displayTypeInfoBootstrap.

Parameters:

· $typeKey : Type key.

Returns:
HTML string.

Example:

```php
echo UnitsInformation::displayTypeInfo('length');
```

---

Private Static Methods

These functions are used internally and are not intended to be called directly, but they are listed here for clarification:

· generateStatistics(): Generates statistics (number of types and units).
· generateSearchBox(): Generates the search box.
· generateAllUnitsInfo(): Generates all unit sections (for the custom view).
· generateUnitCard(): Generates a unit card for the custom view.
· getUnitIcon(): Returns the appropriate icon for the unit.
· generateConversionExample(): Generates a random conversion example.
· getContextLabel(): Gets the context label.
· generateBootstrapStatistics(): Bootstrap statistics.
· generateBootstrapSearchBox(): Bootstrap search box.
· generateBootstrapAllUnitsInfo(): Generates Bootstrap unit sections.
· generateBootstrapUnitCard(): Bootstrap unit card.
· getTypeIcon(): Type icon.
· getContextBadge(): Context badge.
· getSearchScript(): Search script.
· generateSimpleUnitCard(): Simple card.

---

Comprehensive Examples

Example 1: Displaying the Full Units Page in a Controller

```php
public function index()
{
    return UnitsInformation::displayAllUnitsInfo();
}
```

Example 2: Displaying Only Weight Units Within an Existing Template

```php
<div class="container">
    <h2>Weight Units</h2>
    <?= UnitsInformation::displayTypeInfo('weight') ?>
</div>
```

Example 3: Integrating with Custom Bootstrap

```php
<!DOCTYPE html>
<html>
<head>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <?= UnitsInformation::displayAllUnitsInfoBootstrap() ?>
</body>
</html>
```

---

Notes

· The display requires the Font Awesome library for icons.
· The Bootstrap version needs Bootstrap JS if you want the collapsible buttons (collapse) to work.
· All text and labels are in Arabic.

---

This concludes the comprehensive documentation for the UnitsInformation class.