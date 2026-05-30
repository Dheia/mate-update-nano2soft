## 2026-05-29 – 2026-05-31

### Updates to the `Tss.Accounts` Plugin and the `Nano.AccountsApi` Plugin

### Version 1.0.39 – `Tss.Accounts` and Version 1.1.1 – `Nano.AccountsApi`

**Versions:**  
- `Tss.Accounts`: 1.0.39  
- `Nano.AccountsApi`: 1.1.1

---

### Summary of Updates

This release enhances the accounting system in NanoSoft software with full support for multimedia (images, documents, files) on two levels:

1. **Backend (`Tss.Accounts`)** – by adding `attachOne` and `attachMany` relationships to all major accounting models (catch receipts, payment receipts, journal entries, transfers, etc.), with support for displaying and downloading attachments in admin lists and forms.
2. **API (`Nano.AccountsApi`)** – by creating a new Behavior (`DynamicAddMediaIncludes`) that adds dynamic includes to all accounting Transformers, allowing media data to be included in API responses using the same `include` mechanism.

The polymorphic type for files attached to documents has been unified using the `UnifiedMorphClass` trait to ensure seamless integration with the file system and avoid complexity in polymorphic relationships. The update is fully backward‑compatible and does not require any significant changes to the database.

---

### Version 1.0.39 – Detailed Updates in `Tss.Accounts`

#### Release Objectives

- **Add media relationships** to all basic accounting models using `attachOne` and `attachMany`.
- **Unify the polymorphic type** for files attached to accounting documents via the `UnifiedMorphClass` trait.
- **Extend the backend** by adding columns and fields for files in admin lists and forms.
- **Ensure compatibility** with `extendMorphMapTransactionHeader` for handling `attachment_type` in the `File` model.

#### New Features in `Tss.Accounts`

The following models in the accounts part have been extended:

- **Account**
- **Reference**
- **CostCenter**
- **Group**
- **Period**
- **Boxe**
- **Bank**
- **Partner**
- **TransactionsDetail**
- **TransactionHeader** (the parent)
- **BindingBond**
- **OpeningBalance**
- **Bonds2Day**
- **BondsDay**
- **CatchReceipt**
- **PayReceipt**
- **CreditNote**
- **DebitNote**
- **Expense**
- **Income**
- **Relay**
- **Transfer**
- **CreditDelivery** (from the Tss.Cards plugin)

##### 1. `UnifiedMorphClass` Trait

A new trait was created at `Tss\Accounts\Traits\UnifiedMorphClass` to unify the polymorphic name for all accounting models. The trait overrides `getMorphClass` to return `\Tss\Accounts\Models\TransactionHeader::class`:

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

This trait has been applied to the following models (dynamically via `Plugin.php`):

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

##### 2. Multimedia Support in Accounting Models

All of the above models have been extended with the following relationships (dynamically via `extendAllModelsAccountsWithMedia`):

```php
public $attachOne = [
    'image'      => \System\Models\File::class,
    'book_intro' => \System\Models\File::class,
];

public $attachMany = [
    'images' => \System\Models\File::class,
    'videos' => \System\Models\File::class,
    'audios' => \System\Models\File::class,
    'files'  => \System\Models\File::class,
];
```

> **Note:** Currently only `image`, `images`, and `files` are activated in the backend; the others can be easily enabled.

##### 3. Backend Improvements

###### a. Added List Columns

The following columns have been added to the lists of **all models** via the `backend.list.extendColumns` event:

| Column | Type | Visible by default | Description |
| :--- | :--- | :--- | :--- |
| `image` | `simpleimage` | Yes | Displays the main image as a thumbnail |
| `images` | `simpleimages` | No | Shows an icon indicating the presence of an image gallery |
| `files` | `partial` | No | Link to download attached files (via a custom partial) |

The partial files are located at: `plugins/tss/accounts/partials/media/_link_files.htm`.

###### b. Added Form Fields

A new tab named `Attachments` (Media) has been added with the following fields via the `backend.form.extendFields` event:

| Field | Type | Options | Tab |
| :--- | :--- | :--- | :--- |
| `image` | `fileupload` | mode: image, imageWidth: 120, imageHeight: 120, useCaption: true | Attachments |
| `images` | `fileupload` | mode: image, multiple: true | Attachments |
| `files` | `fileupload` | mode: file, multiple: true | Attachments |

##### 4. Handling Polymorphic Relations

The function `extendMorphMapTransactionHeader` in `Plugin.php` has been updated to ensure that all attached files use the same `attachment_type` (`Tss\Accounts\Models\TransactionHeader`). This simplifies queries and prevents duplication of attachment types.

---

### Version 1.1.1 – Detailed Updates in `Nano.AccountsApi`

#### Release Objectives

- **Add the `DynamicAddMediaIncludes` Behavior** to include media relationships (image, images, files, videos, audios, book_intro) in accounting Transformers.
- **Extend all `Nano.AccountsApi` Transformers** to support the new includes dynamically.
- **Provide full flexibility** to control the return of metadata for each include individually or globally.
- **Integrate `is_mate_data_media` support** with the ability to override it per include via request or settings.

#### New Features in `Nano.AccountsApi`

##### 1. `DynamicAddMediaIncludes` Behavior

A new Behavior was created at `Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes` providing the following methods:

- `includeImage` – returns the main image (attachOne).
- `includeImages` – returns the image gallery (attachMany).
- `includeFiles` – returns general files (attachMany).
- `includeVideos` – returns videos (attachMany) – currently disabled, can be enabled.
- `includeAudios` – returns audio recordings (attachMany) – currently disabled.
- `includeBookIntro` – returns the introductory file (attachOne) – currently disabled.

**Features of the Behavior:**

- Automatically adds the specified includes to the Transformer’s `availableIncludes`.
- Supports adding some of them to `defaultIncludes` via the `default_media_includes` setting (from `config.php` or environment variables).
- Uses helper functions from `Nano\Api\Helpers\ApiHelper` (`image()`, `images()`, `files()`) to format data uniformly.
- Supports specifying `is_mate_data_media` via:
  - The global request parameter `is_mate_data_media`.
  - An include‑specific parameter (e.g., `image[mate_data]=1`).
  - Transformer‑specific settings via `nano.accountsapi::config.{transformer_name}.is_mate_data_{include}`.
  - The global setting `nano.accountsapi::is_mate_data_media`.

**Mechanism for extracting transformer‑specific settings:**

The Behavior contains a function `getDefaultIncludeForConfig` that attempts to retrieve settings based on the transformer’s name after removing the "Transformer" suffix and converting it to snake_case. Example:

- Transformer named `CatchReceiptTransformer` → looks for `nano.accountsapi::config.catch_receipt.include` and `nano.accountsapi::config.catch_receipt.is_mate_data_image`.

##### 2. Extending All Transformers in `Nano.AccountsApi`

The `Plugin.php` in `Nano.AccountsApi` was modified to add a function `extendAccountsApiTransformersWithMedia()` that applies the Behavior to the following Transformers:

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
- as well as the base transformer (`Nano\AccountsApi\Transformers\Transformer`).

##### 3. Configuration (`config.php`)

New settings were added in the `config.php` file of `Nano.AccountsApi`:

```php
'default_media_includes' => env('NANO_ACCOUNTSAPI_DEFAULT_MEDIA_INCLUDES', null),
'is_mate_data_media' => env('NANO_ACCOUNTSAPI_IS_MATE_DATA_MEDIA', false),
```

Settings can be customised per transformer using the same structure, for example:

```php
'catch_receipts_v2' => [
    'is_mate_data_image' => true,
    'include' => ['image', 'files'],
],
```

##### 4. Integrating `scopeExclude` into Accounting Models (for API use)

The `scopeExclude` scope was dynamically added to the following models: `TransactionsType`, `Account`, `TransactionHeader`, `TransactionsDetail` to enable excluding certain columns from the response via the `exclude` parameter in the API.

---

### Practical Examples

#### 1. Upload a Main Image for a Catch Receipt (Backend)

1. Go to `Accounts > Catch Receipts`.
2. Select an existing receipt or create a new one.
3. In the `Attachments` tab, upload an image in the `Main Image` field.
4. Save the receipt.
5. The thumbnail will appear in the catch receipts list.

#### 2. API Request for a Catch Receipt Including Image and Files

**Request:**
```http
GET /api/v1/accounts/catch-receipts/1?include=image,files&is_mate_data_media=1
```

**Response (excerpt):**
```json
{
  "data": {
    "id": 1,
    "code": "CR-001",
    "image": {
      "original": "https://domain.com/storage/receipts/image.jpg",
      "small": "https://domain.com/storage/receipts/image_small.jpg",
      "mate_data": {
        "id": 101,
        "file_name": "image.jpg",
        "file_size": 204800
      }
    },
    "files": [
      {
        "file_name": "invoice.pdf",
        "path": "https://domain.com/storage/receipts/invoice.pdf",
        "mate_data": { ... }
      }
    ]
  }
}
```

#### 3. API Request for a Journal Entry Including Only Images (no metadata)

**Request:**
```http
GET /api/v1/accounts/bonds-days/5?include=images&images[mate_data]=0
```

#### 4. Add a Main Image to a Journal Entry via API (file upload)

**Request:**
```http
POST /api/v1/accounts/bonds-days/5/image
Content-Type: multipart/form-data

file: (binary)
```

(File uploads are handled via a separate endpoint if available, or by updating the model with the `image` field.)

#### 5. Use the `UnifiedMorphClass` Trait in a Custom Model

```php
use Tss\Accounts\Traits\UnifiedMorphClass;

class CustomVoucher extends Model
{
    use UnifiedMorphClass;
    // ...
}
```

---

### Improvements and Fixes

| Area | Improvement |
| :--- | :--- |
| **Unified Morph Type** | All files attached to accounting documents now have `attachment_type = Tss\Accounts\Models\TransactionHeader`, simplifying file management and queries. |
| **Safe exception handling** | If a media relationship does not exist on the model, empty values (`null` or `[]`) are returned instead of throwing an error, ensuring the API does not break. |
| **Full backward compatibility** | Media is not automatically included in existing API responses (only when explicitly requested via `include`), and there is no effect on the existing database. |
| **Easy extensibility** | New media fields (`videos`, `audios`, `book_intro`) can be added simply by uncommenting them in the Behavior and adding the relationships to the models. |
| **High configuration flexibility** | `default_media_includes` and `is_mate_data_media` can be controlled per transformer, globally, or via environment variables. |

---

### Upgrade Requirements

#### For the `Tss.Accounts` plugin

1. **Update the code**:
   - Replace the file `plugins/tss/accounts/Plugin.php` with the new version.
   - Add the folder `partials/media/` in `plugins/tss/accounts/` and place the `_link_files.htm` and other partial files.
   - Add the file `traits/UnifiedMorphClass.php` in `plugins/tss/accounts/traits/`.
2. **Run database migrations** (not required – no structural changes).
3. **Clear cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
4. **Check file upload permissions**:
   - Ensure that the `storage/app/uploads` folder is writable.

#### For the `Nano.AccountsApi` plugin

1. **Update the code**:
   - Replace the file `plugins/nano/accountsapi/Plugin.php` with the new version.
   - Add the `behaviors/` folder in `plugins/nano/accountsapi/` and place the file `DynamicAddMediaIncludes.php`.
   - Update the `config.php` file (publish it) if you wish to customise the settings.
2. **Update the `version.yaml` file** (already done in version 1.1.1):
   ```yaml
   1.1.1:
       - 'Add DynamicAddMediaIncludes behavior for All Tss.Accounts Transformer'
       - 'Support dynamic media includes (image, images, videos, audios, files, book_intro) in Accounts API'
       - 'Add is_mate_data_media parameter to control returning file metadata in API responses'
       - 'Extend availableIncludes automatically for media relations'
   ```
3. **Run the config publish command (optional)**:
   ```bash
   php artisan config:publish Nano.AccountsApi
   ```
4. **Test the endpoints**:
   - Try `GET /api/v1/accounts/catch-receipts/1?include=image,files` and verify that media data appears.

---

### Conclusion

This dual update (version 1.0.39 of `Tss.Accounts` and version 1.1.1 of `Nano.AccountsApi`) represents a major step forward in managing accounting attachments within NanoSoft software. Thanks to the `UnifiedMorphClass` trait, polymorphic file relationships have been unified, simplifying integration with `System\Models\File`. Additionally, the `DynamicAddMediaIncludes` Behavior follows the same architectural pattern used in the `Nano.Location` and `Nano3.Kyc` plugins, ensuring a consistent developer experience across all extensions.

**Key benefits for users and developers:**

- **For users (backend):** Ability to attach images and documents to any accounting document, and view them directly in lists and forms.
- **For developers (API):** Ability to include files and images in API responses with ease, with precise control over metadata and response size.

---

**Reference documentation**:

- [`Tss.Accounts` plugin documentation](./docs/Accounts/Docs-Tss-Accounts-en.md)
- [`AccountHelper` class documentation](./docs/Accounts/Docs-AccountHelper-en.md)
- [`PersonHelper` class documentation](./docs/Accounts/Docs-PersonHelper-en.md)
- [`TransactionsHelper` class documentation](./docs/Accounts/Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` documentation](./docs/Accounts/Docs-TransactionsHelper-Advanced-en.md)
- [Index of `TransactionsHelper` functions](./docs/Accounts/Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive documentation of the `createJournalEntry` function](./docs/Accounts/Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced practical examples of the `createJournalEntry` function](./docs/Accounts/Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` settings (comprehensive)](./docs/Accounts/Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` settings (compatible)](./docs/Accounts/Docs-createJournalEntry-config-v1-en.md)
- [Documentation of exchange difference functions](./docs/Accounts/Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical examples of exchange difference functions](./docs/Accounts/Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)
- [`UnifiedMorphClass` trait documentation](./docs/Accounts/Docs-UnifiedMorphClass-en.md)
- [`Nano.AccountsApi` plugin documentation](./docs/AccountsApi/Docs-Nano-AccountsApi-en.md)
- [`DynamicAddMediaIncludes` Behavior documentation](./docs/AccountsApi/Docs-DynamicAddMediaIncludes-en.md)