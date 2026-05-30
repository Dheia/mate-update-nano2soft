# Documentation of the `UnifiedMorphClass` Trait

**Version:** 1.0.39 (`Tss.Accounts` plugin)  
**Author:** Dheia Ali  
**License:** Proprietary to NanoSoft  
**Last updated:** 2026-05-30

---

## Table of Contents

1. [Introduction](#introduction)
2. [Purpose of the Trait](#purpose-of-the-trait)
3. [Installation and Dependencies](#installation-and-dependencies)
4. [Structure and Usage](#structure-and-usage)
5. [How It Works](#how-it-works)
6. [Integration with `System\Models\File`](#integration-with-systemmodelsfile)
7. [Dynamic Extension via `Plugin.php`](#dynamic-extension-via-pluginphp)
8. [Benefits and Advantages](#benefits-and-advantages)
9. [Practical Examples](#practical-examples)
10. [Troubleshooting](#troubleshooting)
11. [References](#references)

---

## Introduction

`UnifiedMorphClass` is a trait designed specifically for the `Tss.Accounts` plugin to unify the polymorphic class name (morph class) for all accounting document models that may have attached files via `attachOne` or `attachMany` relationships with `System\Models\File`. This trait ensures that all files attached to accounting documents (catch receipts, payment receipts, journal entries, transfers, etc.) are stored in the `system_files` table with the same `attachment_type` value: `Tss\Accounts\Models\TransactionHeader`. This unification simplifies queries for files associated with accounting documents, prevents duplication of attachment types, and facilitates filtering and management via backend interfaces and APIs.

---

## Purpose of the Trait

- **Unify `attachment_type`:** Make all document files use the same reference class (`TransactionHeader`), regardless of the actual document type (CatchReceipt, PayReceipt, BondsDay, etc.).
- **Simplify file queries:** When needing to fetch all files associated with any accounting document, you can query directly on `attachment_type = TransactionHeader::class`.
- **Avoid complex polymorphic relations:** Prevent files from being scattered across different `attachment_type` values, making them harder to track and manage.
- **Support `extendMorphMapTransactionHeader`:** Provide a solid foundation for the extended function that redirects the `morphMap` to `TransactionHeader`.

---

## Installation and Dependencies

This trait is part of the `Tss.Accounts` plugin (version 1.0.39 or later). It does not require separate installation. To use it, ensure:

- `Tss.Accounts` is installed via `composer` or the marketplace.
- The plugin is updated to at least version 1.0.39.
- Database migrations have been run (if any updates are associated).

No additional dependencies are required beyond `System\Models\File`, which is already present in NanoSoft APP.

---

## Structure and Usage

### Basic Usage

You can apply the trait to any model you want to unify the `morphClass` for:

```php
<?php namespace Tss\Accounts\Models;

use Model;
use Tss\Accounts\Traits\UnifiedMorphClass;

class MyCustomVoucher extends Model
{
    use UnifiedMorphClass;
    
    // Rest of the model definition
}
```

### Usage in Standard `Tss.Accounts` Models

All of the following models have been dynamically extended (without modifying their original files) using `Plugin::extend`:

- `TransactionHeader`
- `BindingBond`
- `OpeningBalance`
- `Bonds2Day`
- `BondsDay`
- `CatchReceipt`
- `PayReceipt`
- `CreditNote`
- `DebitNote`
- `Expense`
- `Income`
- `Relay`
- `Transfer`
- `CreditDelivery` (from the `Tss.Cards` plugin)

Therefore, when using any of these models, the trait has already been applied automatically.

### Checking That the Trait Is Applied

```php
$receipt = CatchReceipt::find(1);
echo $receipt->getMorphClass(); // prints: Tss\Accounts\Models\TransactionHeader
```

---

## How It Works

The trait contains only one function, which is an override of `getMorphClass()`:

```php
<?php namespace Tss\Accounts\Traits;

trait UnifiedMorphClass
{
    /**
     * Return a unified polymorphic name for all accounting documents.
     *
     * @return string
     */
    public function getMorphClass()
    {
        return \Tss\Accounts\Models\TransactionHeader::class;
    }
}
```

When this trait is used in a model, any `morphTo`, `morphMany`, `morphOne`, or `morphToMany` relationship will rely on the value returned by `getMorphClass()` to determine the type.

**Illustrative example:**

Without the trait, a `CatchReceipt` model returns `CatchReceipt::class` as its `morphClass`. When a file is attached via `$receipt->image = $file`, the `attachment_type` is stored as `CatchReceipt`. This leads to files being scattered under multiple types.

With the trait, `CatchReceipt` returns `TransactionHeader::class`, so the `attachment_type` is unified. As a result, all files attached to any document type share the same `attachment_type`.

### Integration with `extendMorphMapTransactionHeader`

In the `Plugin.php` file of the `Tss.Accounts` plugin, there is a function `extendMorphMapTransactionHeader` that redirects the `morphMap` for all these models to `TransactionHeader`. This function ensures that any search using `TransactionHeader` will retrieve files associated with any document, even if the original model is different.

---

## Integration with `System\Models\File`

When a file is uploaded to a model that uses `UnifiedMorphClass`, `System\Models\File` automatically records the `attachment_type` as the value returned by `getMorphClass()` of the original model. This behaviour comes from the NanoSoft APP and does not require modification.

**Example:**  
If you upload an image to a `CatchReceipt` via `$receipt->image = $uploadedFile`, then `System\Models\File` will store `attachment_type = Tss\Accounts\Models\TransactionHeader`. Consequently, you can retrieve all files for any accounting document regardless of its type using:

```php
$files = File::where('attachment_type', TransactionHeader::class)->get();
```

---

## Dynamic Extension via `Plugin.php`

The `Tss.Accounts` plugin uses dynamic extension to apply the trait to all the aforementioned models without needing to modify the original model files. The following code is present in `Plugin.php`:

```php
public function extendMorphMapTransactionHeader()
{
    $models = [
        TransactionHeader::class,
        BindingBond::class,
        OpeningBalance::class,
        Bonds2Day::class,
        BondsDay::class,
        CatchReceipt::class,
        PayReceipt::class,
        CreditNote::class,
        DebitNote::class,
        Expense::class,
        Income::class,
        Relay::class,
        Transfer::class,
        CreditDelivery::class,
    ];

    foreach ($models as $modelClass) {
        $modelClass::extend(function ($model) {
            $model->addDynamicMethod('getMorphClass', function () {
                return TransactionHeader::class;
            });
        });
    }
}
```

Thus, the trait is applied without the models directly inheriting it, keeping the original code clean.

---

## Benefits and Advantages

| Benefit | Description |
| :--- | :--- |
| **Simplifies file queries** | All document files (receipts, payments, journal entries) can be fetched with a single query on `attachment_type = TransactionHeader::class`. |
| **Compatible with `Media Manager`** | Makes file management easier via admin interfaces that rely on `attachment_type`. |
| **Reduces `morphMap` complexity** | No need to register each document type in the `morphMap`; registering `TransactionHeader` is sufficient. |
| **Eases relay operations** | When documents are relayed, the files remain linked to the original document without losing the relationship. |
| **Supports `extendMorphMapTransactionHeader`** | Provides a solid foundation for the function that maintains compatibility when querying `File` with `withTrashed` and other options. |
| **No need to modify models** | Applied dynamically, making future updates easier. |

---

## Practical Examples

### 1. Upload a File to a Catch Receipt and Verify `attachment_type`

```php
$receipt = CatchReceipt::find(1);
$file = new \System\Models\File();
$file->data = Input::file('document');
$file->save();

$receipt->files()->add($file);

// The file now has attachment_type = TransactionHeader::class
echo $file->attachment_type; // Tss\Accounts\Models\TransactionHeader
```

### 2. Fetch All Files for Catch Receipts, Payment Receipts, and Journal Entries

```php
use Tss\Accounts\Models\TransactionHeader;
use System\Models\File;

$allAccountFiles = File::where('attachment_type', TransactionHeader::class)->get();
```

### 3. Use `morphTo` in a Custom Model to Retrieve the Original Document

```php
class CustomLog extends Model
{
    public $morphTo = ['transaction'];
}

$log = CustomLog::find(1);
$transaction = $log->transaction; // Will be of type TransactionHeader (not CatchReceipt)
```

### 4. Manually Add the Trait to a New Model

```php
<?php namespace MyPlugin\Models;

use Model;
use Tss\Accounts\Traits\UnifiedMorphClass;

class CustomVoucher extends Model
{
    use UnifiedMorphClass;
    
    public $attachMany = [
        'files' => \System\Models\File::class
    ];
}
```

---

## Troubleshooting

| Problem | Likely Cause | Solution |
| :--- | :--- | :--- |
| `Class 'UnifiedMorphClass' not found` | The trait is missing or not included | Ensure `Tss.Accounts` is updated to 1.0.39 and that the file exists at `traits/UnifiedMorphClass.php`. |
| `getMorphClass()` does not return the unified name | The trait is not applied to the model | Check that you have used `use UnifiedMorphClass;` or that the dynamic extension is working. |
| Attached files store the old `attachment_type` (e.g., `CatchReceipt`) | The trait was not activated before uploading the file | Ensure the plugin load order and clear the cache (`php artisan cache:clear`). |
| Conflict with another custom `morphMap` | Another plugin redefines the `morphMap` | Review plugin compatibility and possibly adjust the `$require` order in `Plugin.php`. |
| Error `Call to undefined method getMorphClass` | The trait is not applied | Apply the trait manually or use the dynamic extension. |

---

## Conclusion

The `UnifiedMorphClass` trait is the cornerstone for unifying attached file management in the `Tss.Accounts` accounting system. Thanks to this trait, developers can rely on `TransactionHeader` as a single type for all files attached to accounting documents, simplifying queries, improving system performance, and enhancing maintainability. The trait is applied dynamically to all major document models and can also be used in any custom model that wishes to share the same file namespace.

---

**References**:
- [Documentation of `Tss.Accounts` (version 1.0.39)](./Update-Accounts-v1.0.39-en.md)
- [Documentation of the `DynamicAddMediaIncludes` Behavior (for API)](./docs/AccountsApi/Docs-DynamicAddMediaIncludes-en.md)
- [Polymorphic Relations Guide](https://laravel.com/docs/11.x/eloquent-relationships#polymorphic-relations)

