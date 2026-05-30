# `Tss.Accounts` Plugin Documentation

**Version:** 1.0.39  
**Author:** Dheia Ali
**License:** Proprietary to NanoSoft  
**Last updated:** 2026-05-30

---

## Table of Contents

1. [Introduction](#introduction)
2. [Requirements](#requirements)
3. [Installation and Upgrade](#installation-and-upgrade)
4. [Main Features](#main-features)
5. [Database Structure](#database-structure)
6. [Models](#models)
7. [Traits](#traits)
8. [Behaviors](#behaviors)
9. [Backend Interface](#backend-interface)
10. [Plugin Settings](#plugin-settings)
11. [Permissions](#permissions)
12. [Notification Rules](#notification-rules)
13. [Print Templates](#print-templates)
14. [Report Widgets](#report-widgets)
15. [Form Widgets](#form-widgets)
16. [Custom Twig Tags](#custom-twig-tags)
17. [Helper Functions](#helper-functions)
18. [Events](#events)
19. [Practical Examples](#practical-examples)
20. [Troubleshooting](#troubleshooting)
21. [References](#references)

---

## Introduction

The `Tss.Accounts` plugin is a comprehensive accounting system for NanoSoft APP, providing all basic and advanced accounting operations. It is built on a strong structure that includes a hierarchical chart of accounts, cost centers, financial periods, receipts and payments, journal entries, inter‑account transfers, debit and credit notes, opening balances, relays, and full support for multimedia (images, documents) on all accounting documents.

The plugin is designed to suit companies, institutions, and schools (integrates with `Tss.School` and `Tss.Student`) and offers high flexibility through customisable settings and fine‑grained permissions for each document type.

---

## Requirements

- **NanoSoft APP** version 3.x or 2.x (with some limitations).
- **PHP** >= 8.0.
- **Required plugins** (declared in `$require`):
  - `Rainlab.Translate`
  - `Rainlab.Location`
  - `Rainlab.User`
  - `Tss.Tools`
  - `Tss.BarcodeGenerator`
  - `Tss.Basic`
  - `Tss.Currency`
- **Recommended plugins**:
  - `Nano.MicroCart` (to support accounting payment methods).
  - `Nano.Orders` (to integrate orders with commissions).
  - `Tss.School` (to support student and parent accounts).
  - `Tss.Cards` (to support `CreditDelivery` documents).
  - `Nano.AccountsApi` (to provide a complete accounting API).

---

## Installation and Upgrade

### Fresh installation

```bash
php artisan plugin:install Tss.Accounts
php artisan nano:up
```

### Upgrade from a previous version

```bash
composer update tss/accounts
php artisan nano:up
php artisan cache:clear
```

### Publish configuration file (optional)

```bash
php artisan config:publish Tss.Accounts
```

This will create the configuration file at `config/tss/accounts/config.php`.

---

## Main Features

| Feature | Description |
| :--- | :--- |
| **Hierarchical Chart of Accounts** | Main and sub‑accounts with hierarchical numbering, supports unlimited levels, account nature (debit/credit), and report types (balance sheet / profit & loss). |
| **Cost Centers** | Hierarchical structure for cost centers, can be linked to accounts and departments. |
| **Financial Periods** | Manage financial years with date ranges, validate document dates. |
| **Journal Entries (BondsDays / Bonds2Days)** | Multi‑party journal entries with support for periodic and reverse entries. |
| **Catch Receipts** | Record cash or bank receipts from customers, suppliers, or students. |
| **Pay Receipts** | Record cash or bank payments. |
| **Transfers** | Transfer funds between accounts (cash/bank) with optional bank charges. |
| **Debit Notes / Credit Notes** | Record adjustments to customer or supplier accounts without cash movement. |
| **Expenses / Incomes** | Record various expenses and incomes. |
| **Relays** | Transfer documents in bulk or individually to later periods. |
| **Opening Balances** | Enter opening balances for accounts at the start of a financial period. |
| **Binding Bonds** | General journal entries (multi‑party). |
| **Partners** | Manage partner, customer, supplier, employee, and student accounts linked to accounting accounts. |
| **Boxes / Banks** | Manage cash boxes and bank accounts, automatically creating their accounts in the chart of accounts. |
| **Groups / References** | Classify and group accounts for reporting purposes. |
| **Multimedia Support (Media)** | Attach images and documents to all document types (added in version 1.0.39). |
| **Unified Morph Type** | All attached files use `TransactionHeader` as the unified type (`UnifiedMorphClass`). |
| **Paper ID Management** | Support for checking duplicate reference numbers based on flexible settings (by financial period or document type). |
| **Automatic Document Numbering** | Generate sequential numbers for documents based on type and department settings. |
| **Fine‑grained Permissions** | Separate permissions for each document type (access, add, edit, delete, print, email, sms, export, search). |
| **Custom Print Templates** | Support for multiple print templates per document type (integrated with the `Tss.Tools` reporting system). |
| **Notifications** | Fire events that can be used with `RainLab.Notify` (e.g., adding balance to a delivery person). |
| **Integration with the Educational System** | Special support for students and parents (student accounts, payments, balance top‑up). |

---

## Database Structure

### Main Tables Created by the Plugin

| Table | Description |
| :--- | :--- |
| `tss_accounts_accounts` | Chart of accounts (supports Nested Tree). |
| `tss_accounts_cost_centers` | Cost centers (supports Nested Tree). |
| `tss_accounts_periods` | Financial periods. |
| `tss_accounts_transaction_headers` | Transaction headers (all document types). |
| `tss_accounts_transactions_details` | Transaction details (debit/credit). |
| `tss_accounts_groups` | Account groups. |
| `tss_accounts_groups_accounts` | Linking accounts to groups. |
| `tss_accounts_references` | Account references (for various references such as account types). |
| `tss_accounts_boxes` | Cash boxes. |
| `tss_accounts_banks` | Banks. |
| `tss_accounts_partners` | Partners (customers, suppliers, employees, students, etc.). |
| `tss_accounts_currency_conversions` | Currency conversions. |
| `tss_accounts_payments_company` | Allowed payment methods. |
| `tss_accounts_relays` | (Uses the same table for document relays). |

> **Note:** All document models use the `tss_accounts_transaction_headers` table with the `ref_type` column to identify the document type, and `transactions_type` to identify the accounting transaction type. The details table `tss_accounts_transactions_details` links each document to its account and debit/credit.

### Auxiliary Tables from Other Plugins

| Table | Usage |
| :--- | :--- |
| `tss_basic_departments` | Departments (stores). |
| `tss_basic_companies` | Companies. |
| `tss_currency_currencies` | Currencies. |
| `tss_basic_employees` | Employees. |
| `tss_student_students` / `tss_student_mparents` | Students and parents. |

---

## Models

### 1. `Account` – Chart of Accounts

```php
class Account extends Model
{
    use \October\Rain\Database\Traits\Validation;
    use \October\Rain\Database\Traits\SoftDelete;
    use \October\Rain\Database\Traits\NestedTree;
    use \October\Rain\Database\Traits\Purgeable;
    use \Tss\Accounts\Traits\UnifiedMorphClass;

    public $table = 'tss_accounts_accounts';
    public $fillable = ['code', 'name', 'account_number', 'account_normal', 'reports', 'parent_id', ...];
    public $belongsTo = [
        'parent' => [Account::class],
        'currency' => [Currency::class],
    ];
    public $hasMany = ['children' => [Account::class, 'key' => 'parent_id']];
}
```

**Key properties:**

- `account_normal`: `debit` or `credit`.
- `reports`: `mezinah` (balance sheet) or `arbeh` (profit & loss).
- `main_sub`: `main` or `sub`.
- Supports unlimited levels thanks to `NestedTree`.

### 2. `TransactionHeader` – Transaction Header (the parent for all documents)

All document models (CatchReceipt, PayReceipt, BondsDay, …) either extend this model or use the same table with a `ref_type` column. It contains:

```php
public $table = 'tss_accounts_transaction_headers';
public $hasMany = ['TransactionDetail' => [TransactionsDetail::class]];
public $belongsTo = [
    'Period' => [Period::class],
    'Department' => [Department::class],
    'CostCenter' => [CostCenter::class],
    'TransactionsType' => [TransactionsType::class],
    'Currency' => [Currency::class],
    'Employee' => [Employee::class],
];
public $attachOne = ['image', 'book_intro'];
public $attachMany = ['images', 'videos', 'audios', 'files'];
```

### 3. `TransactionsDetail` – Transaction Detail

```php
class TransactionsDetail extends Model
{
    public $belongsTo = [
        'TransactionHeader' => [TransactionHeader::class],
        'Account' => [Account::class],
    ];
    // fields: debit, credit, person_type, person_id, notes, ...
}
```

### 4. `CatchReceipt` – Catch Receipt (example)

```php
class CatchReceipt extends TransactionHeader
{
    // Uses the same table with ref_type = 2 for example
    // Adds special helper methods like getAccountTypeOptions, getAccountIdOptions, ...
}
```

### 5. `Period` – Financial Period

```php
class Period extends Model
{
    public $dates = ['from_date', 'to_date'];
    // Helper methods: isValidDate($date), isOpen() ...
}
```

### 6. `CostCenter` – Cost Center

```php
class CostCenter extends Model
{
    use NestedTree;
}
```

### 7. `Partner` – Partner (customer, supplier, employee, student)

```php
class Partner extends Model
{
    public $morphTo = ['partnerable']; // polymorphic relationship to the real model (Employee, Customer, ...)
    public $belongsTo = ['Account' => [Account::class]];
    public $attachOne = ['image'];
    public $attachMany = ['files', 'images'];
}
```

### 8. `Boxe` / `Bank` – Cash Boxes and Banks

```php
class Boxe extends Model
{
    public $belongsTo = ['Account' => [Account::class], 'Currency' => [Currency::class]];
    public $attachOne = ['image'];
}
```

### 9. `Group` / `Reference` – Groups and References

```php
class Group extends Model
{
    public $belongsToMany = ['Accounts' => [Account::class, 'table' => 'tss_accounts_groups_accounts']];
}
```

---

## Traits

| Trait | Description |
| :--- | :--- |
| `HasSequenceDocsNumber` | Generate a sequential document number (docs_number) based on settings per document type. |
| `OptionsPerson` | Provide helper methods for person types and their dropdown lists in forms. |
| `OptionsToPerson` / `OptionsFromPerson` | Copies for parties used on the credit/debit side (specific to transfers). |
| `UnifiedMorphClass` | Unify the morph class for attached files to `TransactionHeader::class` (added in 1.0.39). |
| `HasTimestamps` (from `Tss.Basic`) | Automatically add `created_by`, `updated_by`, `deleted_by` fields. |
| `BasicScope` (from `Tss.Basic`) | Basic scopes such as `IsCompany()`, `Departments()`, `IsActive()`. |

---

## Behaviors

### `Tss\Accounts\Behaviors\TransactionHeaderController`

(Inside controllers) Provides shared methods for document operations.

### `Tss\Accounts\Behaviors\TransactionHeaderImportExport`

Supports importing/exporting documents from/to Excel.

---

## Backend Interface

### Main Menus

| Menu | Path | Description |
| :--- | :--- | :--- |
| Accounts | `/backend/tss/accounts/accounts` | Chart of accounts (tree). |
| Catch Receipts | `/backend/tss/accounts/catchreceipts` | Manage catch receipts. |
| Pay Receipts | `/backend/tss/accounts/payreceipts` | Manage payment receipts. |
| Journal Entries | `/backend/tss/accounts/bondsdays` | Manage journal entries. |
| Transfers | `/backend/tss/accounts/transfers` | Manage inter‑account transfers. |
| Opening Balances | `/backend/tss/accounts/openingbalances` | Enter opening balances. |
| Expenses | `/backend/tss/accounts/expenses` | Manage expenses. |
| Incomes | `/backend/tss/accounts/incomes` | Manage incomes. |
| Debit Notes | `/backend/tss/accounts/debitnotes` | Debit notes. |
| Credit Notes | `/backend/tss/accounts/creditnotes` | Credit notes. |
| Relays | `/backend/tss/accounts/relays` | Relay documents. |
| Cost Centers | `/backend/tss/accounts/costcenters` | Manage cost centers. |
| Financial Periods | `/backend/tss/accounts/periods` | Manage financial periods. |
| Boxes | `/backend/tss/accounts/boxes` | Manage cash boxes. |
| Banks | `/backend/tss/accounts/banks` | Manage bank accounts. |
| Partners | `/backend/tss/accounts/partners` | Manage partner accounts. |
| Chart of Accounts Configuration | `/backend/tss/accounts/configaccounts` | Account settings. |

### Media Management Lists (from version 1.0.39)

The following columns have been added to the lists of **all documents and accounting entities**:

- `image` column (type `simpleimage`) to display the main image.
- `images` column (type `simpleimages`) for the image gallery.
- `files` column (type `partial`) to display links to attached files.

Also, an `Attachments` tab has been added in the input forms, containing file upload fields for images and files.

---

## Plugin Settings

The plugin provides several Settings models accessible from `Settings > Accounting Settings`:

| Settings Model | Description |
| :--- | :--- |
| `AccountsSetting` | General chart of accounts settings (maximum level, capital and profit/loss accounts, etc.). |
| `AccountsReportsSetting` | Settings for accounting report headers. |
| `CatchReceiptsSetting` | Catch receipt settings (reference numbering, allow date modification, force notes, etc.). |
| `PayReceiptsSetting` | Payment receipt settings. |
| `BondsDaysSetting` | Journal entry settings. |
| `Bonds2DaysSetting` | Journal entry settings (second version). |
| `TransfersSetting` | Transfer settings. |
| `CreditNotesSetting` | Credit note settings. |
| `DebitNotesSetting` | Debit note settings. |
| `ExpensesSetting` | Expense settings. |

### Available Settings (examples)

- `is_custome_date`: allow manual editing of the document date.
- `is_force_notes`: make notes mandatory.
- `is_paper_id` / `validat_paper_id`: control the use of a reference number and check for duplicates.
- `type_rapet`: repetition options (number can repeat across periods or across other document types).
- `default_notes`: default note text.
- `is_auto_paper_id`: automatically generate a sequential reference number.
- `is_custom_employees`: allow selecting the employee linked to the document.
- `is_relays`: automatically relay the document upon creation.

---

## Permissions

Permissions are divided by document type, allowing very fine‑grained management. Example for `CatchReceipts`:

| Permission | Key | Description |
| :--- | :--- | :--- |
| Access | `tss.accounts.catch_receipts.access` | Access to catch receipts (user‑specific). |
| Access All | `tss.accounts.catch_receipts.access_all` | Access to all catch receipts. |
| Add | `tss.accounts.catch_receipts.add` | Add a new catch receipt. |
| Edit | `tss.accounts.catch_receipts.edit` | Edit a catch receipt. |
| Delete | `tss.accounts.catch_receipts.delete` | Delete a catch receipt. |
| Print | `tss.accounts.catch_receipts.print` | Print a catch receipt. |
| Send SMS | `tss.accounts.catch_receipts.sms` | Send a catch receipt via SMS. |
| Send Email | `tss.accounts.catch_receipts.email` | Send a catch receipt via email. |
| Search | `tss.accounts.catch_receipts.search` | Search catch receipts. |
| Export | `tss.accounts.catch_receipts.export` | Export catch receipt data. |

The same structure applies to `PayReceipts`, `BondsDays`, `Transfers`, `DebitNotes`, `CreditNotes`, `Expenses`, `OpeningBalances`, etc.

There are also permissions for managing the chart of accounts, groups, cost centers, boxes, banks, and partners.

---

## Notification Rules

The plugin fires the following events that can be used with the `RainLab.Notify` system:

- `tss.accounts.afterAddMonyDelivery` – after adding balance to a delivery person.
- `tss.accounts.afterAddOpeningBalanceDelivery` – after adding an opening balance for a delivery person.
- `tss.accounts.afterBrokerage` – after deducting a commission from a delivery person.

---

## Print Templates

The following templates have been registered in the `Tss.Tools` reporting system:

- `OpeningBalances`
- `BondsDays`
- `CatchReceipt`
- `PayReceipt`
- `Expense`
- `Income`
- `BindingBonds`
- `Transfers`
- `DebitNotes`

And the main format `ReportAccounts`.

---

## Report Widgets

- `Tss\Accounts\ReportWidgets\WelcomeAccounts` – welcome to accounts.
- `Tss\Accounts\ReportWidgets\WelcomeConfigAccounts` – welcome to account settings.

---

## Form Widgets

- `Tss\Accounts\Formwidgets\Price` – price field with currency support.
- `Tss\Accounts\Formwidgets\Repeatertss` – enhanced repeater.

---

## Custom Twig Tags

| Function | Description |
| :--- | :--- |
| `get_old_trans_report($data)` | Display previous documents for a given financial transaction. |
| `get_accounts_person_type_label($person_type)` | Display the label of a party type (customer, supplier, etc.). |
| `get_accounts_person_obj_label($custom_name, $person_type, $person_id)` | Display the name of a party based on its type and ID. |
| `get_accounts_person_symbol_name($person_type)` | Display the symbol of a party type. |

---

## Helper Functions

### `Tss\Accounts\Helpers\AccountHelper`

Functions for handling accounts, including:

- `getBalances($accountCode, $options)` – fetch the balance of an account considering period, department, cost centre, currency, and party.
- `getAccountsTree()` – get the chart of accounts tree.
- `getAccountByNumber($number)` – find an account by its number.

### `Tss\Accounts\Helpers\PersonHelper`

- `getPersonTypeLabel($type, $default)` – translate the party type.
- `getPersonObjName($type, $id)` – get the real name of the party (customer, student, etc.).
- `getPersonSymbolName($type)` – get the party symbol.

### `Tss\Accounts\Helpers\TransactionsHelper`

(Exists in recent versions; provides advanced functions for creating journal entries, checking balances, currency differences, etc.)

---

## Events

The plugin fires the following events (can be listened to via `Event::listen`):

| Event name | Parameters | Description |
| :--- | :--- | :--- |
| `tss.accounts.beforeSaveTransaction` | `$header, $details` | Before saving any financial transaction (header and details). |
| `tss.accounts.afterSaveTransaction` | `$header, $details` | After saving the transaction. |
| `tss.accounts.beforeRelayTransaction` | `$header` | Before relaying a document. |
| `tss.accounts.afterRelayTransaction` | `$header` | After relaying a document. |
| `tss.accounts.beforeDeleteTransaction` | `$header` | Before deleting a document. |
| `tss.accounts.afterDeleteTransaction` | `$header` | After deleting a document. |

---

## Practical Examples

### 1. Create a Journal Entry (BondsDay) via Code

```php
$data = [
    'departments_id' => 1,
    'periods_id' => 3,
    'cost_centers_id' => 2,
    'date_at' => Carbon::now(),
    'notes' => 'Purchase furniture',
    'bonds' => [
        ['accounts_id' => '2-3-1211010001', 'process_type' => 'debit', 'mony' => 5000],
        ['accounts_id' => '2-3-1253010001', 'process_type' => 'credit', 'mony' => 5000],
    ]
];
$bond = new BondsDay($data);
$bond->save(); // Automatically generates the document number and checks the balance
```

### 2. Add a Catch Receipt for a Student

```php
$receipt = new CatchReceipt();
$receipt->departments_id = 1;
$receipt->periods_id = 3;
$receipt->account_type = 'current_student'; // or '24' for students
$receipt->account_id = 1232010001; // student account number
$receipt->mony = 1000;
$receipt->process_type = 'cach';
$receipt->ac_boxs = '2-2-1253010001'; // cash account
$receipt->notes = 'March payment';
$receipt->save();
```

### 3. Fetch the Balance of a Specific Account

```php
$options = [
    'departments_id' => 1,
    'periods_id' => 3,
    'currencys_id' => 1,
];
$balance = AccountHelper::getBalances('2-2-1232010001', $options);
echo "Balance: {$balance}";
```

### 4. Attach an Image to a Catch Receipt (Backend)

In the form, the `Attachments` tab appears where you can upload an image. After saving, you can access it via `$receipt->image`.

### 5. Use the `UnifiedMorphClass` Trait in a Custom Model

```php
use Tss\Accounts\Traits\UnifiedMorphClass;

class MyCustomVoucher extends Model
{
    use UnifiedMorphClass;
    // ...
}
```

---

## Troubleshooting

### Problem: Error "Transaction is unbalanced"

**Cause:** The sum of debit amounts does not equal the sum of credit amounts.

**Solution:** Ensure the `bonds` array contains at least two entries and that the total debit equals total credit.

### Problem: Error "Date is outside the financial period"

**Cause:** The document date is not within the selected financial period’s date range.

**Solution:** Check the `from_date` and `to_date` of the period, or modify the document date.

### Problem: Image columns do not appear in the list

**Cause:** The plugin has not been upgraded to 1.0.39 or the migrations have not been run.

**Solution:** Run `php artisan nano:up` and clear the cache.

### Problem: Error "Class 'UnifiedMorphClass' not found"

**Cause:** The trait is missing or not included.

**Solution:** Ensure the file `traits/UnifiedMorphClass.php` exists in the correct path, and add `use Tss\Accounts\Traits\UnifiedMorphClass;` in the model.

### Problem: File upload fails

**Cause:** Storage folder permissions or `System\Models\File` configuration.

**Solution:** Ensure that `storage/app/uploads` is writable, and check the `cms.upload_dir` setting.

---

## Conclusion

The `Tss.Accounts` plugin is a complete, extensible accounting system suitable for small and medium‑sized enterprises, schools, and companies. Thanks to its support for many document types, fine‑grained permissions, flexible settings, print templates, and multimedia, it provides a comprehensive solution for managing financial and accounting operations within the NanoSoft APP environment. The latest update (1.0.39) adds robust support for files and attachments, enhancing users’ ability to document every accounting operation with the necessary documents.

---

**References**:

- [Documentation of `Tss.Accounts`](./Docs-Tss-Accounts-en.md)
- [`AccountHelper` class documentation](./Docs-AccountHelper-en.md)
- [`PersonHelper` class documentation](./Docs-PersonHelper-en.md)
- [`TransactionsHelper` class documentation](./Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` documentation](./Docs-TransactionsHelper-Advanced-en.md)
- [Index of `TransactionsHelper` functions](./Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive documentation of the `createJournalEntry` function](./Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced practical examples of the `createJournalEntry` function](./Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` settings (comprehensive)](./Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` settings (compatible)](./Docs-createJournalEntry-config-v1-en.md)
- [Documentation of exchange difference functions](./Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical examples of exchange difference functions](./Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)
- [`UnifiedMorphClass` trait documentation](./Docs-UnifiedMorphClass-en.md)
- [`Nano.AccountsApi` plugin documentation](./docs/AccountsApi/Docs-Nano-AccountsApi-en.md)
- [`DynamicAddMediaIncludes` Behavior documentation](./docs/AccountsApi/Docs-DynamicAddMediaIncludes-en.md)
