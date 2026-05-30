# `Nano.AccountsApi` Plugin Documentation

**Version:** 1.1.1  
**Author:** Dheia Ali  
**License:** Proprietary to NanoSoft  
**Last updated:** 2026-05-30

---

## Table of Contents

1. [Introduction](#introduction)
2. [Requirements](#requirements)
3. [Installation and Upgrade](#installation-and-upgrade)
4. [API Structure](#api-structure)
5. [Endpoints](#endpoints)
   - [Accounts](#accounts)
   - [Transactions Types](#transactions-types)
   - [Transaction Headers](#transaction-headers)
   - [Transaction Details](#transaction-details)
   - [Catch Receipts](#catch-receipts)
   - [Bonds Days](#bonds-days)
   - [Pay Receipts](#pay-receipts)
   - [Debit Notes](#debit-notes)
   - [Credit Notes](#credit-notes)
   - [Transfers](#transfers)
   - [Students](#students)
   - [Helper Functions (AccountsHelpers)](#helper-functions-accountshellpers)
6. [Transformers](#transformers)
7. [Behavior: DynamicAddMediaIncludes](#behavior-dynamicaddmediaincludes)
8. [Plugin Configuration (Config)](#plugin-configuration-config)
9. [Permissions and Security](#permissions-and-security)
10. [Practical Examples](#practical-examples)
11. [Troubleshooting](#troubleshooting)
12. [References](#references)

---

## Introduction

The `Nano.AccountsApi` plugin provides a complete RESTful API for the `Tss.Accounts` accounting system within NanoSoft software. The API is built in a RESTful style and supports version `v1` (with the possibility of future expansion to version 2). It allows CRUD operations on all types of accounting documents (receipts, payments, journal entries, transfers, debit and credit notes) as well as fetching accounts, transaction types, cost centers, financial periods, boxes, banks, partners, and students.

The API is fully equipped with multimedia support (images, files) via the `DynamicAddMediaIncludes` Behavior, which adds dynamic `includes` to all Transformers, allowing media data to be included in API responses using the same `include` mechanism as in `Nano.API`.

---

## Requirements

- **NanoSoft APP** version 3.x or 2.x
- **PHP** >= 8.0
- **Required plugins**:
  - `Nano.API`
  - `Tss.Accounts` (version 1.0.39 or later)
- **Recommended plugins**:
  - `Nano.OAuth2` or `Nano.AuthApi` (for user authentication via `oauth-users` middleware)
  - `Tss.School` (to support student endpoints)

---

## Installation and Upgrade

### Fresh installation

```bash
php artisan plugin:install Nano.AccountsApi
php artisan nano:up
```

### Upgrade from a previous version

```bash
composer update nano/accountsapi
php artisan nano:up
php artisan cache:clear
```

### Publish configuration file (optional)

```bash
php artisan config:publish Nano.AccountsApi
```

This will create the configuration file at `config/nano/accountsapi/config.php`.

---

## API Structure

- **Base path:** `/api/v1/accounts`
- **Supported formats:** JSON (default), with the possibility of `application/x-yaml` and `application/xml` depending on the `Accept` header.
- **Authentication:** Sensitive endpoints (those that modify data) use the `oauth-users` middleware, requiring a valid `Bearer token` from OAuth2.
- **Public endpoints** (e.g., `GET /transactionstypes`) do not require authentication.
- **Pagination:** All endpoints that return a list support `page`, `per_page`, `orderBy`, `orderDirection`, and return a response compatible with `IlluminatePaginatorAdapter`.

---

## Endpoints

### Accounts

| Method | Path | Description | Authentication |
| :--- | :--- | :--- | :--- |
| `GET` | `/accounts` | List of accounts (with filtering options) | oauth-users |
| `GET` | `/accounts/{id}` | View a specific account (id or code) | oauth-users |

**Parameters for `GET /accounts`:**

- `orderBy`: sort by (`sort_order`, `code`, `name`, `created_at`) – default `sort_order`
- `orderDir`: `asc` or `desc`
- `main_sub`: `main`, `sub`, `*` (all)
- `ref_type`: reference type (number)
- `is_active`: `true/false`
- `include`: include relationships such as `children`, `parent`, `currency`, `image`, `files`, ...

**Example:**

```http
GET /api/v1/accounts/accounts?include=children,image&main_sub=sub&orderBy=code&per_page=20
```

### Transactions Types

| Method | Path | Description |
| :--- | :--- | :--- |
| `GET` | `/transactionstypes` | List of transaction types (no authentication) |
| `GET` | `/transactionstypes/activelystats` | Check for new updates (for caching) |
| `GET` | `/transactionstypes/{id}` | View a specific transaction type |

### Transaction Headers

| Method | Path | Description | Authentication |
| :--- | :--- | :--- | :--- |
| `GET` | `/transactions` | List of transaction headers (all types) | oauth-users |
| `GET` | `/transactions/{id}` | View a specific transaction header | oauth-users |
| `POST` | `/transactions/check` | Validate a financial transaction (without saving) | oauth-users |
| `POST` | `/transactions/allowpay` | Check whether a certain amount can be paid to a party | oauth-users |
| `POST` | `/transactions/add` | Create a single‑entry journal entry (simple transaction) | oauth-users |

**Example `POST /transactions/add`:**

```json
{
    "departments_id": 1,
    "periods_id": 3,
    "cost_centers_id": 2,
    "date_at": "2026-05-30",
    "notes": "Purchase supplies",
    "details": [
        {
            "account_code": "2-2-1211010001",
            "type": "debit",
            "amount": 500
        },
        {
            "account_code": "2-2-1253010001",
            "type": "credit",
            "amount": 500
        }
    ]
}
```

### Transaction Details

| Method | Path | Description | Authentication |
| :--- | :--- | :--- | :--- |
| `GET` | `/transactionsdetails` | List of transaction details | oauth-users |
| `GET` | `/transactionsdetails/{id}` | View a specific transaction detail | oauth-users |
| `POST` | `/transactions/details` | Fetch transaction details (with advanced filters) | oauth-users |

### Catch Receipts

| Method | Path | Description | Authentication |
| :--- | :--- | :--- | :--- |
| `GET` | `/catchreceipts` | List of catch receipts | oauth-users |
| `GET` | `/catchreceipts/{id}` | View a specific catch receipt | oauth-users |
| `POST` | `/catchreceipts/add` | Create a new catch receipt | oauth-users |

**Example `POST /catchreceipts/add`:**

```json
{
    "departments_id": 1,
    "periods_id": 3,
    "cost_centers_id": 2,
    "date_at": "2026-05-30",
    "notes": "Payment from customer",
    "mony": 1000,
    "process_type": "cach",
    "ac_boxs": "2-2-1253010001",
    "account_type": "14",
    "account_id": "2-2-1231010001",
    "person_type": "customer",
    "person_id": 5
}
```

### Bonds Days

| Method | Path | Description | Authentication |
| :--- | :--- | :--- | :--- |
| `GET` | `/bonds` | List of journal entries | oauth-users |
| `GET` | `/bonds/{id}` | View a specific journal entry | oauth-users |
| `POST` | `/bonds/add` | Create a multi‑party journal entry | oauth-users |
| `POST` | `/bonds/test` | Test the journal entry (without saving) | oauth-users |

**Example `POST /bonds/add`:**

```json
{
    "departments_id": 1,
    "periods_id": 3,
    "date_at": "2026-05-30",
    "notes": "Test journal entry",
    "bonds": [
        {"account_code": "2-2-1211010001", "process_type": "debit", "mony": 500},
        {"account_code": "2-2-1253010001", "process_type": "credit", "mony": 500}
    ]
}
```

### Pay Receipts

| Method | Path | Description | Authentication |
| :--- | :--- | :--- | :--- |
| `GET` | `/payreceipts` | List of payment receipts | oauth-users |
| `GET` | `/payreceipts/{id}` | View a specific payment receipt | oauth-users |
| `POST` | `/payreceipts/add` | Create a new payment receipt | oauth-users |

### Debit Notes

| Method | Path | Description | Authentication |
| :--- | :--- | :--- | :--- |
| `GET` | `/debitnotes` | List of debit notes | oauth-users |
| `GET` | `/debitnotes/{id}` | View a specific debit note | oauth-users |
| `POST` | `/debitnotes/add` | Create a debit note | oauth-users |

### Credit Notes

| Method | Path | Description | Authentication |
| :--- | :--- | :--- | :--- |
| `GET` | `/creditnotes` | List of credit notes | oauth-users |
| `GET` | `/creditnotes/{id}` | View a specific credit note | oauth-users |
| `POST` | `/creditnotes/add` | Create a credit note | oauth-users |

### Transfers

| Method | Path | Description | Authentication |
| :--- | :--- | :--- | :--- |
| `GET` | `/transfers` | List of transfers between accounts | oauth-users |
| `GET` | `/transfers/{id}` | View a specific transfer | oauth-users |
| `POST` | `/transfers/add` | Create a transfer between two accounts | oauth-users |

### Students

| Method | Path | Description | Authentication |
| :--- | :--- | :--- | :--- |
| `POST` | `/students/balances` | Fetch a student’s balance | oauth-users |
| `POST` | `/students/payment` | Add a payment to a student’s account | oauth-users |
| `GET` | `/students/transactions` | Fetch student transactions (with filters) | oauth-users |
| `POST` | `/students/details` | Fetch details of student transactions | oauth-users |

**Example `POST /students/payment`:**

```json
{
    "student_id": 10,
    "amount": 2000,
    "notes": "May payment",
    "payment_method": "cach",
    "box_code": "2-2-1253010001"
}
```

### Helper Functions (AccountsHelpers)

| Method | Path | Description |
| :--- | :--- | :--- |
| `POST` | `/balances` | Fetch the balance of a specific account (with options for department, period, and centre) |
| `POST` | `/transactions/details` | Fetch transaction details (filtered) |

**Example `POST /balances`:**

```json
{
    "account_code": "2-2-1232010001",
    "departments_id": 1,
    "periods_id": 3,
    "currencys_id": 1,
    "include_children": true
}
```

---

## Transformers

All Transformers in `Nano.AccountsApi` have been extended with the `DynamicAddMediaIncludes` Behavior and support the following `availableIncludes` and `defaultIncludes`:

- `image` – main image (attachOne)
- `images` – image gallery (attachMany)
- `files` – attached files (attachMany)
- `videos` – videos (attachMany, disabled by default)
- `audios` – audio recordings (attachMany, disabled by default)
- `book_intro` – introductory file (attachOne, disabled by default)

In addition to the original relationships of each entity (e.g., `children`, `parent`, `currency`, `department`, `period`, `cost_center`, etc.).

**List of supported Transformers:**

- `AccountTransformer`
- `ReferenceTransformer`
- `GroupTransformer`
- `CostCenterTransformer`
- `PeriodTransformer`
- `BoxeTransformer`
- `BankTransformer`
- `PartnerTransformer`
- `TransactionHeaderTransformer`
- `TransactionsDetailTransformer`
- `BondsDayTransformer`
- `CatchReceiptTransformer`
- `PayReceiptTransformer`
- `CreditNoteTransformer`
- `DebitNoteTransformer`
- `TransferTransformer`
- `ExpenseTransformer`
- `IncomeTransformer`
- `OpeningBalanceTransformer`
- `BindingBondTransformer`
- `RelayTransformer`
- `Bonds2DayTransformer`
- `CreditDeliveryTransformer`

---

## Behavior: `DynamicAddMediaIncludes`

This Behavior is located at `Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes` and is responsible for dynamically adding media `includes` to any Transformer.

### Main Features

- **Automatic addition of includes:** Adds `image`, `images`, `files` to `availableIncludes` for each Transformer.
- **Control over default includes:** `default_media_includes` can be set in `config.php` or via the environment variable `NANO_ACCOUNTSAPI_DEFAULT_MEDIA_INCLUDES` (e.g., `image,files`).
- **Control over metadata:** Metadata (file size, name, date, etc.) can be requested via:
  - Global parameter: `is_mate_data_media=1`
  - Include‑specific parameter: `image[mate_data]=1`
  - Transformer‑specific setting: `nano.accountsapi::config.{transformer_name}.is_mate_data_{include}`
- **Uses `ApiHelper`:** Relies on `Nano\Api\Helpers\ApiHelper::image()`, `images()`, `files()` to format data consistently with all Nano plugins.
- **Safe exception handling:** If a relationship does not exist or an error occurs, `null` or `[]` is returned instead of breaking the API.

### How to Use in a Request

```http
GET /api/v1/accounts/catchreceipts/1?include=image,files&is_mate_data_media=1
```

Or with custom metadata per include:

```http
GET /api/v1/accounts/accounts/5?include=image,images&image[mate_data]=1&images[mate_data]=0
```

---

## Plugin Configuration (Config)

The configuration file `config/nano/accountsapi/config.php` (after publishing) supports the following options:

### Global Settings

| Key | Description | Default |
| :--- | :--- | :--- |
| `default_media_includes` | List of `includes` automatically added to `defaultIncludes` | `null` |
| `is_mate_data_media` | Return metadata for media by default | `false` |

### Per‑Entity Settings (example for `catch_receipts_v2`)

```php
'catch_receipts_v2' => [
    'order_by' => 'created_at',
    'order_dir' => 'desc',
    'is_stop_to_array' => true,
    'transactions_type' => 2,
    'is_allow_add' => true,
    'is_allow_add_backend' => true,
    'is_allow_add_frontend' => false,
    // Media settings
    'is_mate_data_image' => true,   // transformer‑specific
    'include' => ['image', 'files'], // default_media_includes for this type
],
```

This structure can be repeated for `bonds_days`, `pay_receipts`, `debit_notes`, `credit_notes`, `transfers`.

### Environment Variables

All settings can be controlled via environment variables (`.env`) using the prefix `NANO_ACCOUNTSAPI_`. Examples:

```
NANO_ACCOUNTSAPI_DEFAULT_MEDIA_INCLUDES=image,files
NANO_ACCOUNTSAPI_IS_MATE_DATA_MEDIA=true
NANO_ACCOUNTSAPI_CATCH_RECEIPTS_IS_ALLOW_ADD=true
NANO_ACCOUNTSAPI_BONDS_DAYS_TRANSACTIONS_TYPE=1
```

---

## Permissions and Security

- **User authentication:** All endpoints that modify data use the `oauth-users` middleware, thus requiring a valid `Bearer token`. The token can be obtained from the `Nano.OAuth2` or `Nano.AuthApi` plugin.
- **Document‑level permissions:** The backend controllers check user permissions according to the document type (e.g., `tss.accounts.catch_receipts.add`, `tss.accounts.catch_receipts.access_all`, etc.). These permissions are imported from `Tss.Accounts`.
- **`scopeExclude` scope:** A dynamic `exclude` scope has been added to some models (`Account`, `TransactionHeader`, `TransactionsDetail`) to allow excluding certain columns from the response via the `exclude` request parameter.

**Example request that excludes certain columns:**

```http
GET /api/v1/accounts/accounts/1?exclude=created_at,updated_at
```

---

## Practical Examples

### 1. Fetch a List of Catch Receipts with Images and Files

```http
GET /api/v1/accounts/catchreceipts?include=image,files&per_page=10&orderBy=created_at&orderDir=desc
```

### 2. Create a Catch Receipt via API (with an attached image)

```bash
curl -X POST https://yourdomain.com/api/v1/accounts/catchreceipts/add \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "departments_id": 1,
    "periods_id": 3,
    "date_at": "2026-05-30",
    "mony": 1500,
    "process_type": "bank",
    "ac_boxs": "2-2-1252010001",
    "account_type": "14",
    "account_id": "2-2-1231010001",
    "notes": "Payment from customer",
    "image": "base64_encoded_image_data"  # optional
  }'
```

### 3. Fetch the Balance of a Specific Account with Advanced Options

```http
POST /api/v1/accounts/balances
Content-Type: application/json

{
    "account_code": "2-2-1232010001",
    "departments_id": 1,
    "periods_id": 3,
    "currencys_id": 1,
    "include_children": false
}
```

**Response:**

```json
{
    "data": {
        "balance": 12500.00,
        "currency_code": "YER",
        "account_name": "Customers"
    }
}
```

### 4. Create a Single‑Entry Journal Entry using `transactions/add`

```http
POST /api/v1/accounts/transactions/add
{
    "departments_id": 1,
    "periods_id": 3,
    "date_at": "2026-05-30",
    "notes": "Pay electricity bill",
    "details": [
        {"account_code": "2-2-4111010001", "type": "debit", "amount": 300},
        {"account_code": "2-2-1253010001", "type": "credit", "amount": 300}
    ]
}
```

### 5. Fetch Student Transactions with Date Filtering

```http
POST /api/v1/accounts/students/details
{
    "student_id": 10,
    "from_date": "2026-01-01",
    "to_date": "2026-05-30",
    "per_page": 20
}
```

### 6. Use `include` to Retrieve an Account’s Main Image

```http
GET /api/v1/accounts/accounts/2-2-1231010001?include=image&image[mate_data]=1
```

---

## Troubleshooting

| Problem | Likely Cause | Solution |
| :--- | :--- | :--- |
| Error 401 (Unauthorized) | Missing or expired `Bearer token` | Ensure the token is passed correctly in the `Authorization` header. |
| Error 403 (Forbidden) | User does not have the required permission (e.g., `tss.accounts.catch_receipts.add`) | Grant the appropriate permission to the user via the control panel. |
| Error "Transaction is unbalanced" | The sum of debits does not equal the sum of credits | Ensure the total debit equals the total credit. |
| `image` or `files` data does not appear in the response | The required `include` was not added, or the relationship does not exist | Add `include=image,files` to the request, and ensure the model has saved files. |
| Error "Class 'ApiHelper' not found" | The `Nano.API` plugin is not installed | Ensure `Nano.API` is installed and run `composer update`. |
| Error "Column not found" when using `exclude` | The column does not exist in the table | Check the correct column names. |

---

## Conclusion

The `Nano.AccountsApi` plugin provides a powerful and flexible API layer for the `Tss.Accounts` system, allowing developers to easily integrate accounting operations into mobile applications, web stores, and third‑party systems. Thanks to its integration with `Nano.API`, it supports all the standard features such as `include`, `exclude`, and pagination. Furthermore, the `DynamicAddMediaIncludes` Behavior follows the same pattern used in `Nano.LocationApi` and `Nano3.Kyc`, ensuring a consistent experience across all NanoSoft APIs.

We recommend reading the [documentation of the `Tss.Accounts` plugin](./Update-Accounts-v1.0.39-en.md) for a deeper understanding of the models, permissions, and core settings.

---

**References**:
- [Documentation of `Tss.Accounts` (version 1.0.39)](./Update-Accounts-v1.0.39-en.md)
- [Documentation of the `DynamicAddMediaIncludes` Behavior](./Docs-DynamicAddMediaIncludes-en.md)
