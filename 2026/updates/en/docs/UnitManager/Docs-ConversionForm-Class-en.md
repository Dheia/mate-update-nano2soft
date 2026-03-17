# `ConversionForm` Class Documentation

**Namespace:** `Tss\Inventory\Classes\Units`

---

## 📋 Table of Contents

1. [Introduction](#introduction)
2. [Main Functions](#main-functions)
   - [getConversionTypes](#getconversiontypes)
   - [getUnitsForSelectedType](#getunitsforselectedtype)
   - [renderAdvancedForm](#renderadvancedform)
   - [renderForm](#renderform)
   - [testQuantityConversions](#testquantityconversions)
3. [Internal Functions](#internal-functions)
   - [processConversion](#processconversion)
   - [getFormScript](#getformscript)
4. [Comprehensive Examples](#comprehensive-examples)

---

## Introduction

The `ConversionForm` class is responsible for generating interactive HTML forms for testing and converting units. It provides a simple user interface that allows the user to select the unit type, source unit, target unit, and value, then displays the conversion result with additional options such as strict mode and result precision. This class is primarily intended for demonstration and testing purposes, not for direct use in production environments, as it relies directly on `$_POST`.

---

## Main Functions

### `getConversionTypes`

```php
public function getConversionTypes(bool $includeSubtypes = false): array
```

Description:
Get a list of available conversion types (unit types) with their Arabic labels.

Parameters:

· $includeSubtypes : If true, returns all types including quantity subtypes. If false, returns only main types.

Returns:
An associative array [type => label].

Example:

```php
$form = new ConversionForm();
$types = $form->getConversionTypes(true);
// ['length' => 'الطول', 'weight' => 'الوزن', 'quantity_packaging' => 'كميات التعبئة', ...]
```

---

getUnitsForSelectedType

```php
public function getUnitsForSelectedType(string $selectedType): array
```

Description:
Get a list of units belonging to a specific type, with their Arabic labels.

Parameters:

· $selectedType : Unit type (e.g., 'length').

Returns:
An associative array [symbol => label].

Example:

```php
$units = $form->getUnitsForSelectedType('length');
// ['mm' => 'مليمتر', 'cm' => 'سنتيمتر', 'm' => 'متر', ...]
```

---

renderAdvancedForm

```php
public function renderAdvancedForm(): string
```

Description:
Generate HTML code for an advanced conversion form, containing:

· A dropdown list for selecting the conversion type (split into groups: main types and quantity types).
· Two dropdown lists for selecting the source unit and target unit.
· A value input field.
· Additional options: strict mode (checkbox) and result precision (select).
· A convert button.
· A result display area (appears after submission).

Returns:
Complete HTML string for the form.

Example:

```php
$form = new ConversionForm();
echo $form->renderAdvancedForm();
```

Notes:

· The form relies on $_POST to display pre-selected values and the result.
· Includes JavaScript code for swapping units when the arrow is clicked, and for updating the form when the conversion type changes.

---

renderForm

```php
public function renderForm(): string
```

Description:
Generate HTML code for a simple conversion form, containing:

· A dropdown list for selecting the conversion type (without grouping).
· Two dropdown lists for selecting units.
· A value input field.
· A convert button.

Returns:
HTML string for the simple form.

Example:

```php
$form = new ConversionForm();
echo $form->renderForm();
```

---

testQuantityConversions

```php
public static function testQuantityConversions(): void
```

Description:
A test function (static) that runs a set of predefined quantity conversions and prints the results to the command line (or HTTP output). Useful for verifying conversion data accuracy.

Tests include:

· dozen → pcs
· gross → dozen
· box → dozen
· tray → box
· pair → half_dozen
· hundred → thousand
· million → thousand
· carton → dozen
· pallet → pcs
· ream → pcs
· pack → dozen
  And others.

Example:

```php
ConversionForm::testQuantityConversions();
// Test results will be printed to the screen.
```

---

Internal Functions

processConversion

```php
private function processConversion(): string
```

Description:
Process the form data after submission. It validates the inputs, performs the conversion using UnitConverter (considering strict mode via UnitContextValidator), and returns HTML code to display the result or error message.

Returns:
HTML string containing the conversion result (success or error).

getFormScript

```php
private function getFormScript(): string
```

Description:
Returns the JavaScript code required for the advanced form, which performs the following:

· Automatically submits the form when the conversion type changes.
· Swaps unit values when the arrow between the two dropdowns is clicked.
· Automatically converts negative values to positive.

Returns:
JavaScript string inside a <script> tag.

---

Comprehensive Examples

Example 1: Embedding the Advanced Form in a Web Page

```php
<?php
use Tss\Inventory\Classes\Units\ConversionForm;
?>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>Unit Converter</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
</head>
<body>
    <div class="container mt-5">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header bg-primary text-white">
                        <h4>Comprehensive Unit Converter</h4>
                    </div>
                    <div class="card-body">
                        <?php
                        $form = new ConversionForm();
                        echo $form->renderAdvancedForm();
                        ?>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

Example 2: Using the Simple Form in a Dashboard

```php
$form = new ConversionForm();
echo $form->renderForm();
```

Example 3: Running Quantity Tests

```php
// Can be called from anywhere, such as the command line or a test page
ConversionForm::testQuantityConversions();
```

---

Important Notes

· This class is intended for demonstration and testing purposes, and it heavily relies on $_POST, making it unsuitable for production use without additional security handling.
· The forms use Bootstrap 5 and Font Awesome for styling, so these libraries must be included in the page.
· The testQuantityConversions function prints results directly, which is useful for developers to verify conversion data accuracy.
