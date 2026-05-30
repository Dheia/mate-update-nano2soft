# توثيق إضافة `Nano.AccountsApi`

**الإصدار:** 1.1.1  
**المؤلف:** Dheia Ali  
**الرخصة:** خاصة بـ NanoSoft  
**آخر تحديث:** 2026-05-30

---

## المحتويات

1. [مقدمة](#مقدمة)
2. [المتطلبات](#المتطلبات)
3. [التثبيت والترقية](#التثبيت-والترقية)
4. [هيكل الـ API](#هيكل-الـ-api)
5. [نقاط النهاية (Endpoints)](#نقاط-النهاية-endpoints)
   - [الحسابات (Accounts)](#الحسابات-accounts)
   - [أنواع الحركات (TransactionsTypes)](#أنواع-الحركات-transactionstypes)
   - [رؤوس الحركات (TransactionHeaders)](#رؤوس-الحركات-transactionheaders)
   - [تفاصيل الحركات (TransactionsDetails)](#تفاصيل-الحركات-transactionsdetails)
   - [سندات القبض (CatchReceipts)](#سندات-القبض-catchreceipts)
   - [القيود اليومية (BondsDays)](#القيود-اليومية-bondadays)
   - [سندات الدفع (PayReceipts)](#سندات-الدفع-payreceipts)
   - [أشعار مدين (DebitNotes)](#أشعار-مدين-debitnotes)
   - [أشعار دائن (CreditNotes)](#أشعار-دائن-creditnotes)
   - [التحويلات (Transfers)](#التحويلات-transfers)
   - [الطلاب (Students)](#الطلاب-students)
   - [دوال مساعدة (AccountsHelpers)](#دوال-مساعدة-accountshellpers)
6. [المحولات (Transformers)](#المحولات-transformers)
7. [Behavior: DynamicAddMediaIncludes](#behavior-dynamicaddmediaincludes)
8. [إعدادات الإضافة (Config)](#إعدادات-الإضافة-config)
9. [الصلاحيات والأمان](#الصلاحيات-والأمان)
10. [أمثلة عملية](#أمثلة-عملية)
11. [استكشاف الأخطاء وإصلاحها](#استكشاف-الأخطاء-وإصلاحها)
12. [المراجع](#المراجع)

---

## مقدمة

إضافة `Nano.AccountsApi` توفر واجهة برمجة تطبيقات (API) متكاملة لنظام الحسابات `Tss.Accounts` ضمن برمجيات نانوسوفت. تم تصميم الـ API بنمط RESTful وتدعم الإصدار `v1` (مع إمكانية التوسع للإصدار 2). تتيح الإضافة عمليات CRUD على جميع أنواع السندات المحاسبية (سندات قبض، صرف، قيود يومية، تحويلات، أشعار مدينة ودائنة) بالإضافة إلى جلب الحسابات، أنواع الحركات، مراكز التكلفة، الفترات المالية، الصناديق، البنوك، الشركاء، والطلاب.

تم تزويد الـ API بدعم كامل للوسائط المتعددة (الصور، الملفات) عبر Behavior `DynamicAddMediaIncludes` الذي يضيف `includes` ديناميكية لجميع Transformers، مما يسمح بتضمين بيانات الصور والمستندات في استجابات API بنفس آلية `include` المستخدمة في `Nano.API`.

---

## المتطلبات

- **NanoSoft APP** الإصدار 3.x أو 2.x
- **PHP** >= 8.0
- **الإضافات المطلوبة**:
  - `Nano.API`
  - `Tss.Accounts` (الإصدار 1.0.39 أو أحدث)
- **الإضافات الموصى بها**:
  - `Nano.OAuth2` أو `Nano.AuthApi` (لتوثيق المستخدمين عبر `oauth-users` middleware)
  - `Tss.School` (لدعم نقاط نهاية الطلاب)

---

## التثبيت والترقية

### تثبيت جديد

```bash
php artisan plugin:install Nano.AccountsApi
php artisan nano:up
```

### الترقية من إصدار سابق

```bash
composer update nano/accountsapi
php artisan nano:up
php artisan cache:clear
```

### نشر ملف الإعدادات (اختياري)

```bash
php artisan config:publish Nano.AccountsApi
```

سيتم إنشاء ملف الإعدادات في `config/nano/accountsapi/config.php`.

---

## هيكل الـ API

- **المسار الأساسي:** `/api/v1/accounts`
- **التنسيق المدعوم:** JSON (افتراضي)، مع إمكانية `application/x-yaml` و `application/xml` حسب `Accept` header.
- **التوثيق:** تستخدم نقاط النهاية الحساسة (التي تغير البيانات) `middleware: oauth-users`، مما يتطلب إرسال `Bearer token` صالح من OAuth2.
- **نقاط النهاية العامة** (مثل `GET /transactionstypes`) لا تتطلب توثيقاً.
- **الترقيم:** تدعم جميع نقاط النهاية التي تعيد قائمة معاملات `page`, `per_page`, `orderBy`, `orderDirection`، وتعيد استجابة متوافقة مع `IlluminatePaginatorAdapter`.

---

## نقاط النهاية (Endpoints)

### الحسابات (Accounts)

| الطريقة | المسار | الوصف | التوثيق |
| :--- | :--- | :--- | :--- |
| `GET` | `/accounts` | قائمة الحسابات (مع خيارات fltering) | oauth-users |
| `GET` | `/accounts/{id}` | عرض حساب محدد (id أو code) | oauth-users |

**معاملات GET /accounts:**

- `orderBy`: ترتيب حسب (`sort_order`, `code`, `name`, `created_at`) – افتراضي `sort_order`
- `orderDir`: `asc` أو `desc`
- `main_sub`: `main`, `sub`, `*` (الكل)
- `ref_type`: نوع المرجع (رقم)
- `is_active`: `true/false`
- `include`: تضمين علاقات مثل `children`, `parent`, `currency`, `image`, `files`, ...

**مثال:**

```http
GET /api/v1/accounts/accounts?include=children,image&main_sub=sub&orderBy=code&per_page=20
```

### أنواع الحركات (TransactionsTypes)

| الطريقة | المسار | الوصف |
| :--- | :--- | :--- |
| `GET` | `/transactionstypes` | قائمة أنواع الحركات (بدون توثيق) |
| `GET` | `/transactionstypes/activelystats` | التحقق من وجود تحديثات جديدة (للتخزين المؤقت) |
| `GET` | `/transactionstypes/{id}` | عرض نوع حركة محدد |

### رؤوس الحركات (TransactionHeaders)

| الطريقة | المسار | الوصف | التوثيق |
| :--- | :--- | :--- | :--- |
| `GET` | `/transactions` | قائمة رؤوس الحركات (جميع الأنواع) | oauth-users |
| `GET` | `/transactions/{id}` | عرض رأس حركة محدد | oauth-users |
| `POST` | `/transactions/check` | التحقق من صحة حركة مالية (بدون حفظ) | oauth-users |
| `POST` | `/transactions/allowpay` | التحقق من إمكانية دفع مبلغ معين لجهة | oauth-users |
| `POST` | `/transactions/add` | إنشاء قيد مفرد (حركة بسيطة) | oauth-users |

**مثال `POST /transactions/add`:**

```json
{
    "departments_id": 1,
    "periods_id": 3,
    "cost_centers_id": 2,
    "date_at": "2026-05-30",
    "notes": "شراء مستلزمات",
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

### تفاصيل الحركات (TransactionsDetails)

| الطريقة | المسار | الوصف | التوثيق |
| :--- | :--- | :--- | :--- |
| `GET` | `/transactionsdetails` | قائمة تفاصيل الحركات | oauth-users |
| `GET` | `/transactionsdetails/{id}` | عرض تفصيل حركة محدد | oauth-users |
| `POST` | `/transactions/details` | جلب تفاصيل الحركات (مع فلاتر متقدمة) | oauth-users |

### سندات القبض (CatchReceipts)

| الطريقة | المسار | الوصف | التوثيق |
| :--- | :--- | :--- | :--- |
| `GET` | `/catchreceipts` | قائمة سندات القبض | oauth-users |
| `GET` | `/catchreceipts/{id}` | عرض سند قبض محدد | oauth-users |
| `POST` | `/catchreceipts/add` | إنشاء سند قبض جديد | oauth-users |

**مثال `POST /catchreceipts/add`:**

```json
{
    "departments_id": 1,
    "periods_id": 3,
    "cost_centers_id": 2,
    "date_at": "2026-05-30",
    "notes": "دفعة من عميل",
    "mony": 1000,
    "process_type": "cach",
    "ac_boxs": "2-2-1253010001",
    "account_type": "14",
    "account_id": "2-2-1231010001",
    "person_type": "customer",
    "person_id": 5
}
```

### القيود اليومية (BondsDays)

| الطريقة | المسار | الوصف | التوثيق |
| :--- | :--- | :--- | :--- |
| `GET` | `/bonds` | قائمة القيود اليومية | oauth-users |
| `GET` | `/bonds/{id}` | عرض قيد يومي محدد | oauth-users |
| `POST` | `/bonds/add` | إنشاء قيد يومي (متعدد الأطراف) | oauth-users |
| `POST` | `/bonds/test` | اختبار القيد (بدون حفظ) | oauth-users |

**مثال `POST /bonds/add`:**

```json
{
    "departments_id": 1,
    "periods_id": 3,
    "date_at": "2026-05-30",
    "notes": "قيد يومي تجريبي",
    "bonds": [
        {"account_code": "2-2-1211010001", "process_type": "debit", "mony": 500},
        {"account_code": "2-2-1253010001", "process_type": "credit", "mony": 500}
    ]
}
```

### سندات الدفع (PayReceipts)

| الطريقة | المسار | الوصف | التوثيق |
| :--- | :--- | :--- | :--- |
| `GET` | `/payreceipts` | قائمة سندات الدفع | oauth-users |
| `GET` | `/payreceipts/{id}` | عرض سند دفع محدد | oauth-users |
| `POST` | `/payreceipts/add` | إنشاء سند دفع جديد | oauth-users |

### أشعار مدين (DebitNotes)

| الطريقة | المسار | الوصف | التوثيق |
| :--- | :--- | :--- | :--- |
| `GET` | `/debitnotes` | قائمة أشعار مدين | oauth-users |
| `GET` | `/debitnotes/{id}` | عرض إشعار مدين محدد | oauth-users |
| `POST` | `/debitnotes/add` | إنشاء إشعار مدين | oauth-users |

### أشعار دائن (CreditNotes)

| الطريقة | المسار | الوصف | التوثيق |
| :--- | :--- | :--- | :--- |
| `GET` | `/creditnotes` | قائمة أشعار دائن | oauth-users |
| `GET` | `/creditnotes/{id}` | عرض إشعار دائن محدد | oauth-users |
| `POST` | `/creditnotes/add` | إنشاء إشعار دائن | oauth-users |

### التحويلات (Transfers)

| الطريقة | المسار | الوصف | التوثيق |
| :--- | :--- | :--- | :--- |
| `GET` | `/transfers` | قائمة التحويلات بين الحسابات | oauth-users |
| `GET` | `/transfers/{id}` | عرض تحويل محدد | oauth-users |
| `POST` | `/transfers/add` | إنشاء تحويل بين حسابين | oauth-users |

### الطلاب (Students)

| الطريقة | المسار | الوصف | التوثيق |
| :--- | :--- | :--- | :--- |
| `POST` | `/students/balances` | جلب رصيد طالب | oauth-users |
| `POST` | `/students/payment` | إضافة دفعة مالية لحساب طالب | oauth-users |
| `GET` | `/students/transactions` | جلب حركات طالب (مع فلاتر) | oauth-users |
| `POST` | `/students/details` | جلب تفاصيل حركات طالب | oauth-users |

**مثال `POST /students/payment`:**

```json
{
    "student_id": 10,
    "amount": 2000,
    "notes": "دفعة شهر مايو",
    "payment_method": "cach",
    "box_code": "2-2-1253010001"
}
```

### دوال مساعدة (AccountsHelpers)

| الطريقة | المسار | الوصف |
| :--- | :--- | :--- |
| `POST` | `/balances` | جلب رصيد حساب معين (مع خيارات الفرع والفترة والمركز) |
| `POST` | `/transactions/details` | جلب تفاصيل الحركات (مرشحة) |

**مثال `POST /balances`:**

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

## المحولات (Transformers)

جميع الـ Transformers في `Nano.AccountsApi` تم توسيعها باستخدام Behavior `DynamicAddMediaIncludes` وتدعم `availableIncludes` و `defaultIncludes` التالية:

- `image` – الصورة الرئيسية (attachOne)
- `images` – معرض الصور (attachMany)
- `files` – الملفات المرفقة (attachMany)
- `videos` – الفيديوهات (attachMany، معطل افتراضياً)
- `audios` – التسجيلات الصوتية (attachMany، معطل افتراضياً)
- `book_intro` – الملف التمهيدي (attachOne، معطل افتراضياً)

بالإضافة إلى العلاقات الأصلية لكل كيان (مثل `children`, `parent`, `currency`, `department`, `period`, `cost_center`، إلخ).

**قائمة Transformers المدعومة:**

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

يقع هذا الـ Behavior في `Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes` وهو مسؤول عن إضافة `includes` ديناميكية لوسائط الميديا إلى أي Transformer.

### الميزات الرئيسية

- **إضافة تلقائية للـ includes:** يضيف `image`, `images`, `files` إلى `availableIncludes` لكل Transformer.
- **التحكم في default includes:** يمكن تحديد `default_media_includes` في `config.php` أو عبر متغير بيئة `NANO_ACCOUNTSAPI_DEFAULT_MEDIA_INCLUDES` (مثال: `image,files`).
- **التحكم في بيانات الميتا:** يمكن طلب بيانات الميتا (حجم الملف، الاسم، التاريخ، إلخ) عبر:
  - المعامل العام: `is_mate_data_media=1`
  - المعامل الخاص بكل include: `image[mate_data]=1`
  - الإعداد الخاص بـ Transformer: `nano.accountsapi::config.{transformer_name}.is_mate_data_{include}`
- **استخدام `ApiHelper`:** يعتمد على دوال `Nano\Api\Helpers\ApiHelper::image()`, `images()`, `files()` لتنسيق البيانات بشكل موحد مع جميع إضافات Nano.
- **معالجة آمنة للاستثناءات:** في حالة عدم وجود علاقة أو حدوث خطأ، يتم إرجاع `null` أو `[]` بدلاً من تعطل API.

### طريقة الاستخدام في الطلب

```http
GET /api/v1/accounts/catchreceipts/1?include=image,files&is_mate_data_media=1
```

أو مع تخصيص الميتا لكل include:

```http
GET /api/v1/accounts/accounts/5?include=image,images&image[mate_data]=1&images[mate_data]=0
```

---

## إعدادات الإضافة (Config)

ملف الإعدادات `config/nano/accountsapi/config.php` (بعد نشره) يدعم الخيارات التالية:

### إعدادات عامة

| المفتاح | الوصف | القيمة الافتراضية |
| :--- | :--- | :--- |
| `default_media_includes` | قائمة `includes` التي تضاف تلقائياً إلى `defaultIncludes` | `null` |
| `is_mate_data_media` | إرجاع بيانات الميتا للميديا افتراضياً | `false` |

### إعدادات لكل نوع كيان (مثال لـ `catch_receipts_v2`)

```php
'catch_receipts_v2' => [
    'order_by' => 'created_at',
    'order_dir' => 'desc',
    'is_stop_to_array' => true,
    'transactions_type' => 2,
    'is_allow_add' => true,
    'is_allow_add_backend' => true,
    'is_allow_add_frontend' => false,
    // إعدادات الميديا
    'is_mate_data_image' => true,   // خاص بالـ Transformer
    'include' => ['image', 'files'], // default_media_includes خاص بهذا النوع
],
```

يمكن تكرار هذا الهيكل لـ `bonds_days`, `pay_receipts`, `debit_notes`, `credit_notes`, `transfers`.

### إعدادات متغيرات البيئة

يمكن التحكم بجميع الإعدادات عبر متغيرات البيئة (`.env`) باستخدام البادئة `NANO_ACCOUNTSAPI_`. أمثلة:

```
NANO_ACCOUNTSAPI_DEFAULT_MEDIA_INCLUDES=image,files
NANO_ACCOUNTSAPI_IS_MATE_DATA_MEDIA=true
NANO_ACCOUNTSAPI_CATCH_RECEIPTS_IS_ALLOW_ADD=true
NANO_ACCOUNTSAPI_BONDS_DAYS_TRANSACTIONS_TYPE=1
```

---

## الصلاحيات والأمان

- **مصادقة المستخدم:** جميع نقاط النهاية التي تغير البيانات تستخدم `middleware: oauth-users` وبالتالي تتطلب `Bearer token` صالح. يمكن الحصول على التوكن من إضافة `Nano.OAuth2` أو `Nano.AuthApi`.
- **الصلاحيات على مستوى السندات:** تتحقق النقاط الخلفية (Controllers) من صلاحيات المستخدم وفقاً لنوع السند (مثل `tss.accounts.catch_receipts.add`، `tss.accounts.catch_receipts.access_all`، إلخ). هذه الصلاحيات مستوردة من `Tss.Accounts`.
- **نطاق `scopeExclude`:** تم إضافة نطاق `exclude` ديناميكياً لبعض النماذج (`Account`, `TransactionHeader`, `TransactionsDetail`) لتمكين استبعاد أعمدة معينة من الاستجابة عبر معامل `exclude` في الطلب.

**مثال لطلب يستبعد أعمدة معينة:**

```http
GET /api/v1/accounts/accounts/1?exclude=created_at,updated_at
```

---

## أمثلة عملية

### 1. جلب قائمة سندات القبض مع الصور والملفات

```http
GET /api/v1/accounts/catchreceipts?include=image,files&per_page=10&orderBy=created_at&orderDir=desc
```

### 2. إنشاء سند قبض عبر API (مع إرفاق صورة)

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
    "notes": "دفعة من عميل",
    "image": "base64_encoded_image_data"  # اختياري
  }'
```

### 3. جلب رصيد حساب معين مع خيارات متقدمة

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

**الرد:**

```json
{
    "data": {
        "balance": 12500.00,
        "currency_code": "YER",
        "account_name": "عملاء"
    }
}
```

### 4. إنشاء قيد يومي (مفرد) باستخدام `transactions/add`

```http
POST /api/v1/accounts/transactions/add
{
    "departments_id": 1,
    "periods_id": 3,
    "date_at": "2026-05-30",
    "notes": "تسديد فاتورة كهرباء",
    "details": [
        {"account_code": "2-2-4111010001", "type": "debit", "amount": 300},
        {"account_code": "2-2-1253010001", "type": "credit", "amount": 300}
    ]
}
```

### 5. جلب حركات طالب مع ترشيح التاريخ

```http
POST /api/v1/accounts/students/details
{
    "student_id": 10,
    "from_date": "2026-01-01",
    "to_date": "2026-05-30",
    "per_page": 20
}
```

### 6. استخدام `include` لاسترجاع الصورة الرئيسية لحساب

```http
GET /api/v1/accounts/accounts/2-2-1231010001?include=image&image[mate_data]=1
```

---

## استكشاف الأخطاء وإصلاحها

| المشكلة | السبب المحتمل | الحل |
| :--- | :--- | :--- |
| خطأ 401 (Unauthorized) | عدم إرسال `Bearer token` أو انتهاء صلاحيته | تأكد من تمرير التوكن بشكل صحيح في `Authorization` header |
| خطأ 403 (Forbidden) | المستخدم لا يملك الصلاحية المطلوبة (مثل `tss.accounts.catch_receipts.add`) | قم بمنح الصلاحية المناسبة للمستخدم عبر لوحة التحكم |
| خطأ "القيد غير متزن" | مجموع المبالغ لا يساوي صفر | تأكد من أن مجموع `debit` يساوي مجموع `credit` |
| عدم ظهور بيانات `image` أو `files` في الاستجابة | لم تضمّن الـ `include` المطلوب أو العلاقة غير موجودة | أضف `include=image,files` إلى الطلب، وتأكد من أن النموذج يحتوي على ملفات محفوظة |
| خطأ "Class 'ApiHelper' not found" | إضافة `Nano.API` غير مثبتة | تأكد من تثبيت `Nano.API` وتشغيل `composer update` |
| خطأ `Column not found` عند استخدام `exclude` | العمود غير موجود في الجدول | تحقق من أسماء الأعمدة الصحيحة |

---

## الخاتمة

إضافة `Nano.AccountsApi` توفر طبقة API قوية ومرنة لنظام `Tss.Accounts`، مما يتيح للمطورين دمج العمليات المحاسبية بسهولة في تطبيقات الجوال، متاجر الويب، وأنظمة الطرف الثالث. بفضل التكامل مع `Nano.API`، تدعم جميع ميزات التضمين (`include`)، استبعاد الأعمدة (`exclude`)، والترقيم (pagination). كما أن Behavior `DynamicAddMediaIncludes` يتبع نفس المعيار المستخدم في إضافات `Nano.LocationApi` و `Nano3.Kyc`، مما يضمن تجربة متسقة عبر جميع واجهات API في نانوسوفت.

نوصي بالاطلاع على [توثيق إضافة `Tss.Accounts`](./Update-Accounts-v1.0.39-ar.md) لفهم أعمق للنماذج والصلاحيات والإعدادات الأساسية.

---

**المراجع**:
- [توثيق إضافة `Tss.Accounts` (الإصدار 1.0.39)](./Update-Accounts-v1.0.39-ar.md)
- [توثيق Behavior `DynamicAddMediaIncludes`](./Docs-DynamicAddMediaIncludes-ar.md)
