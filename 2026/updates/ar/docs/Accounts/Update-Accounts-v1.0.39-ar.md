## 2026-05-29 -2026-05-31

### تحديثات إضافة `Tss.Accounts` وإضافة `Nano.AccountsApi`

### الإصدار 1.0.39 – `Tss.Accounts` والإصدار 1.1.1 – `Nano.AccountsApi`


**الإصدارات:**  
- `Tss.Accounts`: 1.0.39  
- `Nano.AccountsApi`: 1.1.1

---

### ملخص التحديثات

يأتي هذا الإصدار ليعزز النظام المحاسبي في برمجيات نانوسوفت بدعم كامل للوسائط المتعددة (الصور، المستندات، الملفات) على مستويين:

1. **الواجهة الخلفية (Tss.Accounts)** – من خلال إضافة علاقات `attachOne` و `attachMany` إلى جميع نماذج الحسابات الرئيسية (سندات قبض، صرف، قيود يومية، تحويلات، إلخ)، مع دعم عرض وتحميل المرفقات في قوائم ونماذج الإدارة.
2. **واجهة برمجة التطبيقات (Nano.AccountsApi)** – عبر إنشاء Behavior جديد (`DynamicAddMediaIncludes`) يضيف includes ديناميكية إلى جميع Transformers الخاصة بالحسابات، مما يسمح بتضمين بيانات الميديا في استجابات الـ API بنفس آلية `include`.

تم توحيد النوع `polymorphic` للملفات المرتبطة بالسندات باستخدام السمة `UnifiedMorphClass` لضمان تكامل سلس مع نظام الملفات وتجنب تعقيدات العلاقات المتعددة الأشكال. التحديث متوافق تماماً مع الإصدارات السابقة ولا يتطلب أي تغييرات جوهرية في قاعدة البيانات.

---

### الإصدار 1.0.39 – تفاصيل التحديث في `Tss.Accounts`

#### أهداف الإصدار

- **إضافة علاقات الميديا** إلى جميع نماذج الحسابات الأساسية باستخدام `attachOne` و `attachMany`.
- **توحيد النوع polymorphic** للملفات المرتبطة بالسندات المحاسبية من خلال سمة `UnifiedMorphClass`.
- **توسيع الواجهة الخلفية** بإضافة أعمدة وحقول للملفات في قوائم ونماذج الإدارة.
- **ضمان التوافق** مع `extendMorphMapTransactionHeader` لمعالجة `attachment_type` في نموذج `File`.

#### الميزات الجديدة في `Tss.Accounts`


تم توسيع النماذج التالية فى جزء الحسابات 

- **Account**
- **Reference**
- **CostCenter**
- **Group**
- **Period**
- **Boxe**
- **Bank**
- **Partner**
- **TransactionsDetail**
- **TransactionHeader** (الأصل)
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
- **CreditDelivery** (من إضافة Tss.Cards)


##### 1. سمة `UnifiedMorphClass`

تم إنشاء سمة جديدة في `Tss\Accounts\Traits\UnifiedMorphClass` لتوحيد اسم الـ polymorphic لجميع نماذج الحسابات. السمة تعيد `getMorphClass` إلى `\Tss\Accounts\Models\TransactionHeader::class`:

```php
<?php namespace Tss\Accounts\Traits;

trait UnifiedMorphClass
{
    /**
     * إرجاع اسم polymorphic موحد لجميع سندات الحسابات.
     *
     * @return string
     */
    public function getMorphClass()
    {
        return \Tss\Accounts\Models\TransactionHeader::class;
    }
}
```

تم تطبيق هذه السمة على النماذج التالية (ديناميكياً عبر `Plugin.php`):

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
- `CreditDelivery` (من إضافة `Tss.Cards`)

##### 2. دعم الوسائط المتعددة في نماذج الحسابات

تم توسيع جميع النماذج المذكورة أعلاه بإضافة العلاقات التالية (ديناميكياً عبر `extendAllModelsAccountsWithMedia`):

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

> **ملاحظة:** حالياً تم تفعيل `image`, `images`, `files` فقط في الواجهة الخلفية، ويمكن تفعيل الباقي بسهولة.

##### 3. تحسين الواجهة الخلفية (Backend)

###### أ. إضافة أعمدة في قوائم الإدارة (List Columns)

تمت إضافة الأعمدة التالية إلى قوائم **جميع النماذج** عبر حدث `backend.list.extendColumns`:

| العمود | النوع | مرئي افتراضياً | الوصف |
| :--- | :--- | :--- | :--- |
| `image` | `simpleimage` | نعم | عرض الصورة الرئيسية كمصغرة |
| `images` | `simpleimages` | لا | عرض أيقونة تشير إلى وجود معرض صور |
| `files` | `partial` | لا | رابط لتحميل الملفات المرفقة (عبر partial مخصص) |

تم وضع ملفات الـ partial في المسار: `plugins/tss/accounts/partials/media/_link_files.htm`.

###### ب. إضافة حقول في نماذج الإدارة (Form Fields)

تمت إضافة تبويب جديد باسم `المرفقات` (Media) يحتوي على الحقول التالية عبر حدث `backend.form.extendFields`:

| الحقل | النوع | الخيارات | التبويب |
| :--- | :--- | :--- | :--- |
| `image` | `fileupload` | mode: image, imageWidth: 120, imageHeight: 120, useCaption: true | المرفقات |
| `images` | `fileupload` | mode: image, multiple: true | المرفقات |
| `files` | `fileupload` | mode: file, multiple: true | المرفقات |

##### 4. معالجة الـ Polymorphic Relations

تم تحديث دالة `extendMorphMapTransactionHeader` في `Plugin.php` لضمان أن جميع الملفات المرفقة تستخدم نفس `attachment_type` (`Tss\Accounts\Models\TransactionHeader`). هذا يبسط الاستعلامات ويمنع ازدواجية أنواع المرفقات.

---

### الإصدار 1.1.1 – تفاصيل التحديث في `Nano.AccountsApi`

#### أهداف الإصدار

- **إضافة Behavior `DynamicAddMediaIncludes`** لتضمين علاقات الميديا (image, images, files, videos, audios, book_intro) في Transformers الخاصة بالحسابات.
- **توسيع جميع Transformers** الخاصة بـ `Nano.AccountsApi` لدعم الـ includes الجديدة ديناميكياً.
- **توفير مرونة كاملة** للتحكم في إرجاع بيانات الميتا (metadata) لكل include على حدة أو بشكل عام.
- **دمج دعم `is_mate_data_media`** مع إمكانية تجاوزها لكل include عبر الطلب أو الإعدادات.

#### الميزات الجديدة في `Nano.AccountsApi`

##### 1. Behavior `DynamicAddMediaIncludes`

تم إنشاء Behavior جديد في `Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes` يوفر الدوال التالية:

- `includeImage` – إرجاع الصورة الرئيسية (attachOne).
- `includeImages` – إرجاع معرض الصور (attachMany).
- `includeFiles` – إرجاع الملفات العامة (attachMany).
- `includeVideos` – إرجاع الفيديوهات (attachMany) – معطل حالياً، يمكن تفعيله.
- `includeAudios` – إرجاع التسجيلات الصوتية (attachMany) – معطل حالياً.
- `includeBookIntro` – إرجاع الملف التمهيدي (attachOne) – معطل حالياً.

**مميزات الـ Behavior:**

- يضيف تلقائياً الـ includes المحددة إلى `availableIncludes` في الـ Transformer.
- يدعم إضافة بعضها إلى `defaultIncludes` عبر إعداد `default_media_includes` (من `config.php` أو متغيرات البيئة).
- يستخدم دوال مساعدة من `Nano\Api\Helpers\ApiHelper` (`image()`, `images()`, `files()`) لتنسيق البيانات بشكل موحد.
- يدعم تحديد `is_mate_data_media` عبر:
  - المعامل العام `is_mate_data_media` في الطلب.
  - المعامل الخاص بكل include (مثال: `image[mate_data]=1`).
  - إعدادات خاصة بـ Transformer عبر `nano.accountsapi::config.{transformer_name}.is_mate_data_{include}`.
  - الإعداد العام `nano.accountsapi::is_mate_data_media`.

**آلية استخراج الإعدادات الخاصة بـ Transformer:**

يحتوي الـ Behavior على دالة `getDefaultIncludeForConfig` التي تحاول جلب الإعدادات بناءً على اسم الـ Transformer بعد إزالة لاحقة "Transformer" وتحويله إلى snake_case. مثال:

- Transformer باسم `CatchReceiptTransformer` → يبحث في الإعدادات عن `nano.accountsapi::config.catch_receipt.include` و `nano.accountsapi::config.catch_receipt.is_mate_data_image`.

##### 2. توسيع جميع Transformers الخاصة بـ `Nano.AccountsApi`

تم تعديل `Plugin.php` في `Nano.AccountsApi` بإضافة دالة `extendAccountsApiTransformersWithMedia()` التي تطبق الـ Behavior على الـ Transformers التالية:

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
- وأيضاً الـ Transformer الأساسي (`Nano\AccountsApi\Transformers\Transformer`).

##### 3. إعدادات `config.php`

تم إضافة إعدادات جديدة في ملف `config.php` الخاص بـ `Nano.AccountsApi`:

```php
'default_media_includes' => env('NANO_ACCOUNTSAPI_DEFAULT_MEDIA_INCLUDES', null),
'is_mate_data_media' => env('NANO_ACCOUNTSAPI_IS_MATE_DATA_MEDIA', false),
```

يمكن تخصيص الإعدادات لكل Transformer على حدة باستخدام نفس الهيكل، مثل:

```php
'catch_receipts_v2' => [
    'is_mate_data_image' => true,
    'include' => ['image', 'files'],
],
```

##### 4. دمج `scopeExclude` في نماذج الحسابات (للاستخدام في API)

تمت إضافة نطاق `scopeExclude` ديناميكياً إلى النماذج التالية: `TransactionsType`, `Account`, `TransactionHeader`, `TransactionsDetail` لتمكين استبعاد أعمدة معينة من الاستجابة عبر معامل `exclude` في API.

---

### أمثلة عملية

#### 1. رفع صورة رئيسية لسند قبض (الواجهة الخلفية)

1. انتقل إلى `الحسابات > سندات القبض`.
2. اختر سنداً أو أنشئ سنداً جديداً.
3. في تبويب `المرفقات`، ارفع صورة في حقل `الصورة الرئيسية`.
4. احفظ السند.
5. ستظهر الصورة المصغرة في قائمة سندات القبض.

#### 2. طلب API لسند قبض مع تضمين الصورة والملفات

**الطلب:**
```http
GET /api/v1/accounts/catch-receipts/1?include=image,files&is_mate_data_media=1
```

**الاستجابة (مقتطف):**
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

#### 3. طلب API لقيد يومي مع تضمين الصور فقط دون بيانات الميتا

**الطلب:**
```http
GET /api/v1/accounts/bonds-days/5?include=images&images[mate_data]=0
```

#### 4. إضافة صورة رئيسية لقيد يومي عبر API (رفع ملف)

**الطلب:**
```http
POST /api/v1/accounts/bonds-days/5/image
Content-Type: multipart/form-data

file: (binary)
```

(يتم التعامل مع رفع الملفات عبر نقطة نهاية منفصلة إن وجدت، أو عبر تحديث النموذج مع إرسال الملف في حقل `image`.)

#### 5. استخدام السمة `UnifiedMorphClass` في نموذج مخصص

```php
use Tss\Accounts\Traits\UnifiedMorphClass;

class CustomVoucher extends Model
{
    use UnifiedMorphClass;
    // ...
}
```

---

### التحسينات والإصلاحات

| المجال | التحسين |
| :--- | :--- |
| **توحيد الـ Morph Type** | جميع الملفات المرفقة بالسندات المحاسبية أصبحت `attachment_type = Tss\Accounts\Models\TransactionHeader`، مما يبسط إدارة الملفات والاستعلامات. |
| **معالجة آمنة للاستثناءات** | في حالة عدم وجود علاقة ميديا في النموذج، يتم إرجاع قيم فارغة (`null` أو `[]`) بدلاً من إلقاء خطأ، مما يضمن عدم تعطل API. |
| **توافق عكسي كامل** | لا يتم تضمين الوسائط تلقائياً في استجابات API الحالية (فقط عند طلبها صراحة عبر `include`)، ولا تؤثر على قاعدة البيانات الحالية. |
| **سهولة التوسع** | يمكن إضافة حقول وسائط جديدة (`videos`, `audios`, `book_intro`) بمجرد إلغاء التعليق في الـ Behavior وإضافة العلاقات في النماذج. |
| **مرونة عالية في الإعدادات** | إمكانية التحكم في `default_media_includes` و `is_mate_data_media` لكل Transformer على حدة، أو بشكل عام، أو عبر متغيرات البيئة. |

---

### متطلبات الترقية

#### للإضافة `Tss.Accounts`

1. **تحديث الكود**:
   - استبدال ملف `plugins/tss/accounts/Plugin.php` بالنسخة الجديدة.
   - إضافة مجلد `partials/media/` في `plugins/tss/accounts/` ووضع ملفات `_link_files.htm` وغيرها.
   - إضافة ملف `traits/UnifiedMorphClass.php` في `plugins/tss/accounts/traits/`.
2. **تشغيل هجرات قاعدة البيانات** (غير مطلوبة – لا تغيير في الهيكل).
3. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
4. **التحقق من صلاحيات رفع الملفات**:
   - التأكد من أن مجلد `storage/app/uploads` قابل للكتابة.

#### للإضافة `Nano.AccountsApi`

1. **تحديث الكود**:
   - استبدال ملف `plugins/nano/accountsapi/Plugin.php` بالنسخة الجديدة.
   - إضافة مجلد `behaviors/` في `plugins/nano/accountsapi/` ووضع ملف `DynamicAddMediaIncludes.php`.
   - تحديث ملف `config.php` (نشره) إذا أردت تخصيص الإعدادات.
2. **تحديث ملف `version.yaml`** (تم بالفعل في الإصدار 1.1.1):
   ```yaml
   1.1.1:
       - 'Add DynamicAddMediaIncludes behavior for All Tss.Accounts Transformer'
       - 'Support dynamic media includes (image, images, videos, audios, files, book_intro) in Accounts API'
       - 'Add is_mate_data_media parameter to control returning file metadata in API responses'
       - 'Extend availableIncludes automatically for media relations'
   ```
3. **تشغيل أمر نشر الإعدادات (اختياري)**:
   ```bash
   php artisan config:publish Nano.AccountsApi
   ```
4. **اختبار نقاط النهاية**:
   - جرب `GET /api/v1/accounts/catch-receipts/1?include=image,files` وتأكد من ظهور بيانات الميديا.

---

### الخاتمة

يمثل هذا التحديث المزدوج (الإصدار 1.0.39 من `Tss.Accounts` والإصدار 1.1.1 من `Nano.AccountsApi`) نقلة نوعية في إدارة المرفقات المحاسبية ضمن برمجيات نانوسوفت. بفضل السمة `UnifiedMorphClass`، تم توحيد العلاقات متعددة الأشكال للملفات، مما يبسط التكامل مع `System\Models\File`. كما أن إضافة Behavior `DynamicAddMediaIncludes` يتبع نفس النمط المعماري المستخدم في إضافات `Nano.Location` و `Nano3.Kyc`، مما يضمن تجربة مطور متسقة عبر جميع الإضافات.

**الفوائد الرئيسية للمستخدمين والمطورين:**

- **للمستخدمين (الواجهة الخلفية):** إمكانية إرفاق صور ومستندات بأي سند محاسبي، وعرضها مباشرة في القوائم والنماذج.
- **للمطورين (API):** إمكانية تضمين الملفات والصور في استجابات API بكل سهولة، مع تحكم دقيق في بيانات الميتا وحجم الاستجابة.

---

**الوثائق المرجعية**:

- [توثيق إضافة `Tss.Accounts`](./docs/Accounts/Docs-Tss-Accounts-ar.md)
- [توثيق كلاس `AccountHelper`](./docs/Accounts/Docs-AccountHelper-ar.md)
- [توثيق كلاس `PersonHelper`](./docs/Accounts/Docs-PersonHelper-ar.md)
- [توثيق كلاس `TransactionsHelper`](./docs/Accounts/Docs-TransactionsHelper-ar.md)
- [توثيق متقدم لـ `TransactionsHelper`](./docs/Accounts/Docs-TransactionsHelper-Advanced-ar.md)
- [فهرس دوال `TransactionsHelper`](./docs/Accounts/Docs-TransactionsHelper-Reference-Function-ar.md)
- [توثيق شامل لدالة `createJournalEntry`](./docs/Accounts/Docs-TransactionsHelper-createJournalEntry-ar.md)
- [أمثلة عملية متقدمة لدالة `createJournalEntry`](./docs/Accounts/Docs-TransactionsHelper-createJournalEntry-Example-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./docs/Accounts/Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./docs/Accounts/Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./docs/Accounts/Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./docs/Accounts/Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)
- [توثيق سمة `UnifiedMorphClass`](./docs/Accounts/Docs-UnifiedMorphClass-ar.md)
- [توثيق إضافة `Nano.AccountsApi`](./docs/AccountsApi/Docs-Nano-AccountsApi-ar.md)
- [توثيق Behavior `DynamicAddMediaIncludes`](./docs/AccountsApi/Docs-DynamicAddMediaIncludes-ar.md)
