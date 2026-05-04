## 2026-03-20 – 2026-05-05

**`Nano.AccountsApi` Add‑on Update – Version 1.1.0**

### Comprehensive Restructuring and Addition of Integrated Accounting Document Types

---

### Summary of Updates

In version **1.1.0**, a major architectural leap was made in the `Nano.AccountsApi` add‑on. An abstract base class `BaseDocumentController` was extracted to manage common operations for different types of accounting documents. Full support was added for five new types of financial documents (daily entries, payment vouchers, debit notes, credit notes, transfers) with their own models, transformers and settings, together with an improved version of receipt vouchers (`CatchReceiptsV2`). The configuration file and routes were extended to support these new types, with multiple improvements to existing controllers and transformers.

---

### Version Goals

- **Unify operation logic** through `BaseDocumentController` to eliminate code duplication and make it easy to add new document types.
- **Add common accounting document types** to cover most daily financial operations via the API.
- **Provide flexible settings and separate permissions** for each document type, supported by environment variables to simplify customisation.
- **Improve developer experience** by providing a unified API interface for CRUD operations and permission checks.
- **Maintain backward compatibility** with older controllers and transformers, which continue to work unchanged.

---

### New Features and Improvements

#### 1. Unified Abstract Base Class `BaseDocumentController`

An abstract base class named `BaseDocumentController` was created, inheriting from `ApiController`, providing the following shared functionality:

- **Management of the current user** and permission checks (`checkAddPermission`, `checkListPermission`, `checkViewPermission`, `checkUpdatePermission`, `checkDeletePermission`).
- **Data preparation** through functions such as `prepareCreateData`, `prepareListOptions`, `validateInput`.
- **Operation execution** via `performCreate`, `performList`, `performShow`, `performUpdate`, `performDelete`, which return a unified result array.
- **Automatic transformer application** to API responses.

Any new financial document controller can now inherit this class and define only `$modelClass`, `$transformerClass`, `$configKey`, and `$typeHeader` to obtain full CRUD functionality.

#### 2. Addition of New Document Types

Five new controllers were added, each based on `BaseDocumentController`, with its own model, transformer and dedicated settings:

| Controller | Model | Transformer | typeHeader | Description |
|------------|-------|-------------|------------|-------------|
| `BondsDays` | `BondsDay` | `BondsDayTransformer` | `BondsDay::class` | Daily entries (multi‑party) with debit/credit balance checking |
| `PayReceipts` | `PayReceipt` | `PayReceiptTransformer` | `PayReceipt::class` | Payment vouchers (cheque, cash) |
| `DebitNotes` | `DebitNote` | `DebitNoteTransformer` | `DebitNote::class` | Debit notes |
| `CreditNotes` | `CreditNote` | `CreditNoteTransformer` | `CreditNote::class` | Credit notes |
| `Transfers` | `Transfer` | `TransferTransformer` | `Transfer::class` | Transfers between accounts |

An improved version of receipt vouchers was also added:

- `CatchReceiptsV2` (based on `BaseDocumentController` and the `CatchReceipt` model), while the old controller `CatchReceipts` remains unchanged.

**Validation rules specific to each type:**

- **BondsDays:** Checks that `details` exists with at least two parties, that debit equals credit, and prevents zero amounts.
- **CatchReceiptsV2 / PayReceipts:** Supports fields such as `payment_method`, `cheque_number`, `cheque_date`, `bank_name`, `beneficiary`.
- **DebitNotes / CreditNotes:** Fields `invoice_number` and `reason`.
- **Transfers:** Verifies that the two accounts are different, with fields `reference_number` and `reason`.

#### 3. New Transformer System

- A `BondsDayTransformer` was created, inheriting from `TransactionHeaderTransformer`, adding the total debit/credit for each entry.
- Transformers `CatchReceiptTransformer`, `PayReceiptTransformer`, `DebitNoteTransformer`, `CreditNoteTransformer`, `TransferTransformer` were created, all inheriting from `TransactionHeaderTransformer` and adding their own fields (e.g. `payment_method`, `cheque_number` …).
- `TransactionHeaderTransformer` was updated to accept all the new model types through union types in the signature of the `data()` function, allowing it to serve as a base transformer for any financial transaction header.

#### 4. Expanded Settings and Permissions

New sections were added in the `config.php` file for each document type, with support for environment variables to control:

- **Operational settings:** `order_by`, `order_dir`, `transactions_type`, `process_type`, `patterns_id`, `notes`.
- **Separate permissions:** `is_allow_add`, `is_allow_add_backend`, `is_check_add_permission`, `is_allow_add_frontend` (and similarly for list, view, update, delete).
- **Transformer settings:** `is_stop_to_array` for each type.

The added sections are: `bonds_days`, `catch_receipts_v2`, `pay_receipts`, `debit_notes`, `credit_notes`, `transfers`.

#### 5. Expanded API Routes

New routes were added within the `oauth-users` group in `routes.php`:

```
// Daily entries
POST  bonds/add → BondsDays@create
POST  bonds/test → BondsDays@test
GET   bonds → BondsDays@index
GET   bonds/{id} → BondsDays@show

// Payment vouchers
POST  payreceipts/add → PayReceipts@create
GET   payreceipts → PayReceipts@index
GET   payreceipts/{id} → PayReceipts@show

// Debit notes
POST  debitnotes/add → DebitNotes@create
GET   debitnotes → DebitNotes@index
GET   debitnotes/{id} → DebitNotes@show

// Credit notes
POST  creditnotes/add → CreditNotes@create
GET   creditnotes → CreditNotes@index
GET   creditnotes/{id} → CreditNotes@show

// Transfers
POST  transfers/add → Transfers@create
GET   transfers → Transfers@index
GET   transfers/{id} → Transfers@show
```

Note: Routes for update and delete (`PUT` and `DELETE`) were added as comments, ready to be activated in the future.

#### 6. Additional Improvements

- **The `show` function in `Accounts.php`** now supports looking up an account by its `code` in addition to its `id`, giving developers more flexibility.
- **The `test` function in `BondsDays`** allows testing a daily entry via `TransactionsHelper::testJournalEntry` before saving, with special permissions for backend users.
- **`TransactionHeaderTransformer`** was extended to cover all financial document types, while keeping its core functionality unchanged.

---

### Usage Examples

#### Create a multi‑party daily entry

```bash
curl -X POST "https://yourdomain.com/api/v1/accounts/bonds/add" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "date_at": "2026-04-20",
    "notes": "Settlement entry",
    "details": [
      {"accounts_id": "2-2-1253010001", "debit": 1500},
      {"accounts_id": "2-2-1231010001", "credit": 1000},
      {"accounts_id": "2-2-4111010001", "credit": 500}
    ]
  }'
```

#### List receipt vouchers (new version)

```bash
curl -X GET "https://yourdomain.com/api/v1/accounts/catchreceipts" \
  -H "Authorization: Bearer <token>"
```

#### Show an account using its code

```bash
curl -X GET "https://yourdomain.com/api/v1/accounts/accounts/2-2-1231010001" \
  -H "Authorization: Bearer <token>"
```

---

### Benefits and Added Value

- **Faster development**: Any new financial document type can be added by writing just a few lines of code, simply by inheriting `BaseDocumentController`.
- **Reduced errors**: Unifying permission‑checking logic and data preparation reduces the chance of human error.
- **Customisation flexibility**: Each document type has its own settings and separate permissions, allowing the add‑on to be adapted to the requirements of each project.
- **Enhanced user experience**: Full API support for all core financial operations allows building rich front‑end applications.
- **Stronger security**: Uses the same permission system already present in the base add‑on (`Tss.Accounts`) via the API.
- **Future‑proof**: The new architecture makes it easy to add update and delete operations (technically ready, waiting to be activated).

---

### Upgrade Requirements

1. **Update the code**:
   - Replace the core files with the new versions:
     - `APIControllers/BaseDocumentController.php`
     - `APIControllers/BondsDays.php`
     - `APIControllers/PayReceipts.php`
     - `APIControllers/DebitNotes.php`
     - `APIControllers/CreditNotes.php`
     - `APIControllers/Transfers.php`
     - `APIControllers/CatchReceiptsV2.php`
     - `Transformers/BondsDayTransformer.php`
     - `Transformers/CatchReceiptTransformer.php`
     - `Transformers/PayReceiptTransformer.php`
     - `Transformers/DebitNoteTransformer.php`
     - `Transformers/CreditNoteTransformer.php`
     - `Transformers/TransferTransformer.php`
   - Update the following files:
     - `config.php`
     - `routes.php`
     - `TransactionHeaderTransformer.php`
     - `Accounts.php` (the `show` improvement)
2. **No new database migrations**.
3. **API permissions**:
   - The new endpoints are protected by `oauth-users`, so the user must have a valid token, and their permissions must match the settings defined for each type.
4. **Test connectivity**:
   - Make sure the environment variables for the new settings are set correctly (especially `is_allow_add_backend` and the transaction type `transactions_type`).
   - Try creating a simple daily entry using `bonds/add` and check the response.
5. **Clear the cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### Conclusion

Version **1.1.0** represents a qualitative leap for the `Nano.AccountsApi` add‑on, moving it from a collection of individual controllers to a small framework for managing financial documents via the API. Thanks to `BaseDocumentController`, adding new document types has become quick and safe, opening the door to future expansions without complexity. The comprehensive coverage of entry types, vouchers, notes and transfers makes the add‑on suitable for a wide range of accounting applications, while maintaining backward compatibility with previous versions.

---

### Additional Documentation

**Reference Documentation**:
- [`AccountHelper` Class Documentation](./Docs-AccountHelper-en.md)
- [`PersonHelper` Class Documentation](./Docs-PersonHelper-en.md)
- [`TransactionsHelper` Class Documentation](./Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` Documentation](./Docs-TransactionsHelper-Advanced-en.md)
- [`TransactionsHelper` Function Index](./Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive `createJournalEntry` Documentation](./Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced Practical Examples for `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical Examples for Exchange Differences Functions](./Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)

