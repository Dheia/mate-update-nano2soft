# `PersonHelper` Class Documentation – Persons and Entities Helper

## Overview

The `PersonHelper` class is located at `Tss\Accounts\Helpers\PersonHelper` and is the core component responsible for managing “person” (or “entity”) entities within the `Tss.Accounts` add‑on. In the accounting system, any financial transaction (debit or credit) can be linked to a “person” to represent the customer, supplier, employee, student, store, or any other entity with a financial relationship. This class is the bridge that connects the accounting system with the various other models of the system via Morph relationships.

The class responsibilities include:

- **Converting class names to Morph values and vice versa**: Functions such as `getMorphClassNano` and `getMorphedModelNano` make it easier to handle Eloquent polymorphic relationships.
- **Providing user interface options**: Functions such as `getPersonTypeLabelOptions` and `getPersonSymbolLabelOptions` generate ready‑to‑use arrays for dropdown lists in forms and reports.
- **Fetching person objects**: The `getPersonObj` function retrieves the actual person object (e.g., an `Employee` or `Student` object) based on `person_type` and `person_id`.
- **Extracting names and contact information**: Functions `getPersonObjName` and `getPersonObjArray` extract the name, mobile number, and email address from any person object in a unified way.
- **Managing short symbols**: A system to link short symbols (e.g., `'TTYP'`, `'EMPL'`, `'STUD'`) to each person type, facilitating storage and display in user interfaces.

All methods of this class are `static`, allowing them to be called easily from anywhere in the application without needing to instantiate an object. It also relies on internal caching (`$listPersonTypeLabel` and `$listPersonSymbolLabel`) to improve performance when the same lists are requested repeatedly.

---

## Core Data Structure: Supported Person Types

The class supports 15 person types by default, which can be enabled or disabled via the configuration file (`tss.accounts::person_type.*`). Each type has a symbol and a descriptive label used throughout the system.

| Value (Morph Class) | Class | Symbol | Description |
| :--- | :--- | :--- | :--- |
| `TTYP` | `Nano\Tags\Models\Type` | `TTYP` | Types of tags |
| `TAGS` | `Nano\Tags\Models\Tag` | `TAGS` | Tags |
| `TCAT` | `Nano\Tags\Models\Categorie` | `TCAT` | Tag categories |
| `DEPT` | `Tss\Basic\Models\Department` | `DEPT` | Departments / branches |
| `PROD` | `Nano\Shop\Models\Product` | `PROD` | Products |
| `EMPL` | `Tss\Basic\Models\Employee` | `EMPL` | Employees |
| `CUST` | `Tss\SalesAndMarketing\Models\Customer` | `CUST` | Customers |
| `FUSE` | `RainLab\User\Models\User` | `FUSE` | Front‑end users |
| `BUSE` | `Backend\Models\User` | `BUSE` | Backend users |
| `DELI` | `Nano\Deliverys\Models\Delivery` | `DELI` | Delivery couriers |
| `SUPP` | `Tss\Purchasing\Models\Suppler` | `SUPP` | Suppliers |
| `SYND` | `Tss\Purchasing\Models\Syndical` | `SYND` | Agents / syndicates |
| `STUD` | `Tss\Student\Models\Student` | `STUD` | Students |
| `MPAR` | `Tss\Student\Models\Mparent` | `MPAR` | Parents |
| `PART` | `Tss\Accounts\Models\Partner` | `PART` | Business partners |

**Notes about the activation system (Config):**

- **`getMorphToClassNameOptions`** and **`getPersonTypeLabelOptions`** consult the `Config` settings (`tss.accounts::person_type.*`). Some types are enabled by default (e.g., `tags_type`, `department`) while others are not (`employee`, `customer`…).
- **`getMorphToSymbolOptions`** and **`getClassNameToSymbolOptions`** use the `$stop_check` parameter. If `$stop_check = true`, all types are included regardless of the settings. This is useful in scenarios where you need all possibilities without referring to `Config`.

---

## Public Methods (Public API)

### Group One: Morph ↔ Class Conversion Functions

#### 1.1 `getMorphClassNano`

Converts a class name (or an object) into its corresponding Morph Class value. Used to get the short value stored in the `*_type` column in the database.

```php
public static function getMorphClassNano($class): ?string
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$class` | `string\|object` | The class name (e.g., `\RainLab\User\Models\User`) or an object. |

**Return value:** The Morph value (e.g., `'FUSE'`, `'EMPL'`…) or `null` if it cannot be found.

**Mechanism:**
1.  If `$class` is an object, it is converted to a class name.
2.  Tries to call `app($className)->getMorphClass()` directly.
3.  If that fails (the class name is not a valid Morph value), it tries to find the full class name from the morph name and then get the `getMorphClass` value from a new object.

**Example:**
```php
$morph = PersonHelper::getMorphClassNano(\RainLab\User\Models\User::class); // 'FUSE'
$morph = PersonHelper::getMorphClassNano($userObject); // 'FUSE'
```

---

#### 1.2 `getMorphedModelNano`

Converts a Morph Class value (e.g., `'FUSE'`) into the full class name (e.g., `'RainLab\User\Models\User'`). The direct inverse of `getMorphClassNano`.

```php
public static function getMorphedModelNano($class): ?string
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$class` | `string` | The short Morph Class value. |

**Mechanism:**
1.  Tries to use `\Illuminate\Database\Eloquent\Relations\Relation::getMorphedModel`.
2.  If that fails, tries to use `\October\Rain\Database\Relations\Relation::getMorphedModel` (for different October copies).

**Example:**
```php
$fullClass = PersonHelper::getMorphedModelNano('FUSE'); // 'RainLab\User\Models\User'
```

---

#### 1.3 `getMorphedModelObjectNano`

Creates a new object of the specified Morph Class.

```php
public static function getMorphedModelObjectNano($class): ?object
```

**Example:**
```php
$object = PersonHelper::getMorphedModelObjectNano('FUSE'); // new RainLab\User\Models\User()
```

---

#### 1.4 `getPersonTypeClassName`

A comprehensive conversion function: accepts a Morph Class value (e.g., `'FUSE'`) and returns the full class name. Internally uses `getClassNameToMorphOptions` and `getMorphedModelNano`.

```php
public static function getPersonTypeClassName($person_type): ?string
```

**Example:**
```php
$class = PersonHelper::getPersonTypeClassName('FUSE'); // 'RainLab\User\Models\User'
$class = PersonHelper::getPersonTypeClassName('EMPL'); // 'Tss\Basic\Models\Employee'
```

---

### Group Two: Options and Lists Functions

#### 2.1 `getPersonTypeLabelOptions`

Generates an array of options with translatable person type names, for use in dropdown lists.

```php
public static function getPersonTypeLabelOptions(): array
```

**Return value:** An array of the form `['FUSE' => 'Frontend User', 'EMPL' => 'Employee', ...]`.

Uses internal caching (`$listPersonTypeLabel`).

**Example:**
```php
$options = PersonHelper::getPersonTypeLabelOptions();
// [
//     ""    => "Choose person type",
//     "TTYP" => "Tag Types",
//     "DEPT" => "Departments",
//     "EMPL" => "Employees",
//     ...
// ]
```

---

#### 2.2 `getPersonSymbolLabelOptions`

Generates an array of the short symbols (e.g., `'TTYP'`, `'EMPL'`).

```php
public static function getPersonSymbolLabelOptions(): array
```

**Return value:** An array of the form `['FUSE' => 'FUSE', 'EMPL' => 'EMPL', ...]`.

**Example:**
```php
$symbols = PersonHelper::getPersonSymbolLabelOptions();
// ["TTYP" => "TTYP", "DEPT" => "DEPT", "EMPL" => "EMPL", ...]
```

---

#### 2.3 `getMorphToClassNameOptions`

Reverse mapping: key = Morph Class value, value = full class name.

```php
public static function getMorphToClassNameOptions(): array
```

---

#### 2.4 `getClassNameToMorphOptions`

The opposite of the previous: key = full class name, value = Morph Class value.

```php
public static function getClassNameToMorphOptions(): array
```

---

#### 2.5 `getMorphToSymbolOptions`

Mapping from Morph Class to short symbol (the `$stop_check` parameter controls whether activation settings are ignored).

```php
public static function getMorphToSymbolOptions($stop_check = true): array
```

#### 2.6 `getClassNameToSymbolOptions`

Same as above, but the key is the full class name.

```php
public static function getClassNameToSymbolOptions($stop_check = true): array
```

---

### Group Three: Fetching Person Objects and Their Data

#### 3.1 `getPersonObj`

Fetches the actual person object based on its type and ID.

```php
public static function getPersonObj($person_type = null, $person_id = null, $auto_fill = true): ?object
```

- If `$person_id` is not passed, an empty new object of the specified type is returned.

**Example:**
```php
$user = PersonHelper::getPersonObj('FUSE', 15);
$newEmployee = PersonHelper::getPersonObj('EMPL'); // empty object
```

---

#### 3.2 `getPersonObjArray`

Extracts the person’s data (name, mobile, email) as an array or a single value. It looks for the name in `name`, `full_name`, `first_name`, `username`, `login` in that order.

```php
public static function getPersonObjArray($person_type = null, $person_id = null, $filed = 'all', $auto_fill = true): mixed
```

**Examples:**
```php
$data = PersonHelper::getPersonObjArray('FUSE', 15);
// ['name' => 'Ahmed', 'mobile' => '0123456789', 'email' => 'a@b.com']

$name = PersonHelper::getPersonObjArray($userObject, null, 'name');
```

---

#### 3.3 `getPersonObjLabel`

A wrapper around `getPersonObjArray` to retrieve a single field (`name`, `mobile`, `email`).

```php
public static function getPersonObjLabel($person_type = null, $person_id = null, $filed = 'name', $auto_fill = false): mixed
```

#### 3.4 `getPersonObjName`

A shortcut to fetch just the name. Used extensively in `TransactionsHelper`.

```php
public static function getPersonObjName($person_type = null, $person_id = null, $auto_fill = false): mixed
```

**Example:**
```php
$name = PersonHelper::getPersonObjName('STUD', 120); // 'Mohammed Ali'
```

---

### Group Four: Short Symbol Functions

#### 4.1 `getPersonSymbolName`

Gets the short symbol for a given person type.

```php
public static function getPersonSymbolName($class_name, $defaultValue = null, $stop_check = true): ?string
```

#### 4.2 `getPersonTypeLabel`

Gets the translatable descriptive label for a given person type.

```php
public static function getPersonTypeLabel($class_name, $defaultValue = null): ?string
```

---

## Internal Caching

The class uses two static properties to improve performance:

```php
public static $listPersonTypeLabel = null;
public static $listPersonSymbolLabel = null;
```

The first time `getPersonTypeLabelOptions` or `getPersonSymbolLabelOptions` is called, the arrays are built and stored. Subsequent calls return the cached value. To refresh the lists after changing settings within the same request:

```php
PersonHelper::$listPersonTypeLabel = null;
PersonHelper::$listPersonSymbolLabel = null;
```

---

## Usage Scenarios

### Scenario 1: Build a dropdown list for choosing a person type

```php
$options = PersonHelper::getPersonTypeLabelOptions();
echo Form::select('person_type', $options, null, ['class' => 'form-control']);
```

### Scenario 2: Display the name of the party associated with an accounting entry

```php
$detail = TransactionsDetail::find(100);
$name = PersonHelper::getPersonObjName($detail->person_type, $detail->person_id);
echo "Party: " . $name;
```

### Scenario 3: Validate a submitted `person_type` value

```php
$validTypes = PersonHelper::getPersonTypeLabelOptions();
if (!array_key_exists($request->input('person_type'), $validTypes)) {
    throw new ApplicationException('Invalid person type.');
}
```

### Scenario 4: Adding a new person type (for developers)

1.  Add the type to all six option functions (`getMorphToClassNameOptions`, `getMorphToSymbolOptions`, `getClassNameToSymbolOptions`, `getPersonTypeLabelOptions`, `getPersonSymbolLabelOptions`).
2.  Add an activation key in `Config` (`tss.accounts::person_type.new_type`).
3.  Ensure that the corresponding model supports `MorphMany`/`MorphTo`.

---

## Dependencies

- `Illuminate\Database\Eloquent\Relations\Relation` and `October\Rain\Database\Relations\Relation`
- `Tss\Accounts\Models\*` (Period, Account, Reference, Currency, TransactionHeader, TransactionsDetail)
- `Config` (for the `tss.accounts::person_type.*` settings)
- `Lang` / `trans` (for translation)

---

## Conclusion

The `PersonHelper` class is the cornerstone of managing entities linked to financial transactions. Through its unified system for converting between Morph types, its ready‑to‑use interface options, and its intelligent data extraction mechanism, the class simplifies linking any system entity (user, student, employee, store) to accounting operations. This documentation provides a comprehensive guide for using and extending the class.

---

## Additional Documentation

**Reference Documentation**:
- [`AccountHelper` Class Documentation](./Docs-AccountHelper-en.md)
- [`TransactionsHelper` Class Documentation](./Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` Documentation](./Docs-TransactionsHelper-Advanced-en.md)
- [`TransactionsHelper` Function Index](./Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive `createJournalEntry` Documentation](./Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced Practical Examples for `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical Examples for Exchange Differences Functions](./Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)
