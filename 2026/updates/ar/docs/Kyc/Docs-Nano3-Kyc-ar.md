# توثيق إضافة Nano3.Kyc

## نظرة عامة

إضافة **Nano3.Kyc** هي حل برمجي متكامل لإدارة عمليات التحقق من الهوية (Know Your Customer - KYC) ضمن منصة NanoSoft App. توفر الإضافة نظاماً مرناً وآمناً لجمع وثائق الهوية الشخصية والتجارية، التحقق من صحتها، وتتبع حالتها وفقاً لأفضل ممارسات الامتثال التنظيمي.

تم تطوير الإضافة بواسطة **Nano2Soft** لتلبية احتياجات المؤسسات المالية، منصات التجارة الإلكترونية، والخدمات الرقمية التي تتطلب التحقق من هوية المستخدمين والعملاء.

### أهداف الإضافة

- **مركزية إدارة الوثائق**: توحيد مكان تخزين وإدارة جميع أنواع وثائق KYC.
- **مرونة عالية**: دعم تعريف أنواع وثائق غير محدودة بحقول ديناميكية قابلة للتخصيص.
- **التحقق والاعتماد**: توفير دورة حياة كاملة للوثيقة (قيد الانتظار ← تم التحقق ← مرفوض).
- **التكامل السلس**: واجهة API جاهزة للربط مع تطبيقات الجوال والويب.
- **الامتثال والأمان**: تتبع تواريخ الصلاحية، النسخ الورقية، وسجلات المراجعة.

---

## هيكل الإضافة ومكوناتها

تم تصميم الإضافة وفقاً لمعايير NanoSoft App وهيكلية `Nano3`، وتتكون من المكونات الرئيسية التالية:

```
plugins/nano3/kyc/
├── classes/
│   ├── Manager.php              # مدير العمليات (إنشاء، تحديث، حذف، تحقق)
│   └── DocumentType.php         # تعريف أنواع الوثائق وخصائصها
├── config/
│   └── document_types.php       # ملف تخصيص أنواع الوثائق
├── controllers/
│   └── Documents.php            # متحكم الواجهة الخلفية (Backend)
├── models/
│   └── Document.php             # موديل الوثيقة الرئيسي
│   └── document/                # السمات (Traits) المرتبطة بالموديل
├── updates/
│   ├── version.yaml             # سجل التحديثات والإصدارات
│   └── create_table_nano3_kyc_documents.php
├── lang/
│   └── ar/
│       └── lang.php             # ملف الترجمة العربية
├── apicontrollers/
│   └── Documents.php            # متحكم API RESTful
├── transformers/
│   └── DocumentTransformer.php  # محول بيانات API
└── routes.php                   # مسارات API
```

### المكونات الأساسية

| المكون | الوصف |
| :--- | :--- |
| `Manager` | الكلاس المسؤول عن جميع عمليات الوثائق (إنشاء، تحديث، حذف، اعتماد، رفض) مع دعم الاختبارات (`is_test_create`) والتحقق من التكرار والأحداث. |
| `DocumentType` | كلاس مركزي يعرف أكثر من 28 نوع وثيقة، خصائصها، حقولها، قواعد التحقق، وأوزان الموثوقية. يدعم تحميل الإعدادات من ملفات `config`. |
| `Document` | موديل Eloquent يمثل جدول `nano3_kyc_documents`. يستخدم مجموعة من السمات (Traits) لإضافة نطاقات البحث، خيارات القوائم، والتخزين المؤقت. |
| `Documents (APIController)` | متحكم API يوفر نقاط نهاية RESTful لإدارة الوثائق والتحقق منها. |
| `Documents (Controller)` | متحكم الواجهة الخلفية لإدارة الوثائق من لوحة تحكم NanoSoft App. |

---

## أنواع الوثائق المدعومة

تدعم الإضافة **28+ نوع وثيقة** موزعة على **6 فئات رئيسية**، مع إمكانية إضافة أنواع جديدة بسهولة عبر ملف الإعدادات.

| الفئة | الأنواع |
| :--- | :--- |
| **هوية أساسية** | جواز سفر، بطاقة هوية وطنية، رخصة قيادة، تصريح إقامة، وثيقة لاجئ |
| **هوية ثانوية** | بطاقة ناخب، هوية عسكرية، هوية حكومية |
| **إثبات عنوان** | فاتورة مرافق، كشف حساب بنكي، مستند ضريبي، عقد إيجار، كشف رهن عقاري |
| **وثائق شركات** | شهادة تأسيس، عقد تأسيس، رخصة تجارية، تسجيل ضريبي، سجل المساهمين، قرار مجلس إدارة، قوائم مالية، إثبات عنوان تجاري |
| **المستفيد الحقيقي** | إقرار UBO، هيكل ملكية، صك ائتمان |
| **تحقق إضافي** | صورة شخصية (سيلفي)، فحص حيوية، تحقق بالفيديو، نموذج توقيع |

---

## الميزات الرئيسية

### 1. حقول ديناميكية مرنة

لكل نوع وثيقة مجموعة حقول خاصة به (مثل: `full_name`, `passport_type`, `religion`, `trustee_name`). يتم تخزين الحقول المشتركة والمفهرسة في أعمدة منفصلة، بينما تخزن الحقول المتخصصة في حقل JSON واحد (`fields_data`). هذا يضمن:

- **أداء عالي** في البحث والتصفية.
- **مرونة مطلقة** لاستيعاب أي نوع وثيقة مستقبلي دون تعديل هيكل الجدول.

**مثال على تعريف حقل ديناميكي:**
```php
'passport_type' => [
    'label'      => 'passport_type',
    'type'       => 'select',
    'required'   => false,
    'options'    => ['P' => 'Personal', 'D' => 'Diplomatic', 'O' => 'Official'],
    'order'      => 350,
],
```

### 2. دورة حياة الوثيقة والتحقق

تمر الوثيقة بمراحل محددة:
- **`pending`**: قيد الانتظار (الحالة الافتراضية عند الإنشاء).
- **`verified`**: تم التحقق والاعتماد.
- **`rejected`**: مرفوضة مع إمكانية إضافة سبب الرفض.
- **`expired`**: منتهية الصلاحية (يتم تحديثها تلقائياً عند انتهاء تاريخ الصلاحية).

يمكن للمحقق (مثل موظف أو نظام) اعتماد الوثيقة عبر `Manager::verifyDocument()` مما يؤدي إلى تسجيل:
- تاريخ التحقق (`verified_at`)
- معرف المحقق (`verifier_id` / `verifier_type`)
- درجة الموثوقية (`verification_score`)

### 3. إدارة النسخ الورقية (Hard Copy Tracking)

تدعم الإضافة تتبع الوثائق الورقية المستلمة من العميل (مثال: أصل جواز السفر). الحقول المخصصة:

| الحقل | الوصف |
| :--- | :--- |
| `is_physical_submitted` | هل تم تسليم نسخة ورقية؟ |
| `physical_copy_type` | نوع النسخة (`original` أو `copy`) |
| `physical_received_at` | تاريخ استلام النسخة الورقية |
| `physical_returned_at` | تاريخ إعادتها لصاحبها |
| `physical_notes` | ملاحظات حول النسخة الورقية |

### 4. واجهة API متكاملة

جميع العمليات متاحة عبر API بنمط RESTful، مما يسمح بالتكامل مع تطبيقات الجوال والويب.

**أمثلة على الطلبات:**

```http
GET /api/v1/kyc/documents?document_type=passport&status=pending
POST /api/v1/kyc/documents/{id}/verify
POST /api/v1/kyc/documents
Content-Type: multipart/form-data

{
  "document_type": "national_id",
  "owner_id": 123,
  "owner_type": "RainLab\\User\\Models\\User",
  "fields[full_name]": "أحمد محمد",
  "fields[document_number]": "1234567890",
  "files[document_front]": (binary)
}
```

**استجابة نموذجية:**
```json
{
  "code": 200,
  "status": true,
  "message": "تم إنشاء الوثيقة بنجاح.",
  "data": {
    "id": 1,
    "document_type": "national_id",
    "status": "pending",
    ...
  }
}
```

### 5. واجهة خلفية متكاملة (Backend UI)

توفر الإضافة قوائم ونماذج إدارة كاملة داخل لوحة تحكم NanoSoft App:

- **قائمة الوثائق**: مع إمكانية البحث، التصفية، والفرز حسب جميع الحقول.
- **نموذج إنشاء/تعديل**: مقسم إلى تبويبات (أساسي، المالك، التحقق، النسخة الورقية، الملفات، النشر).
- **فلاتر متقدمة**: عبر ملف `config_filter.yaml` تشمل نوع الوثيقة، الحالة، تاريخ الإصدار، المالك، وغيرها.
- **أزرار إجراءات جماعية**: اعتماد، رفض، حذف، تصدير.

---

## متطلبات النظام

| المتطلب | الإصدار |
| :--- | :--- |
| PHP | >= 8.0 |
| NanoSoft App | 2.x أو 3.x |
| إضافة `Nano.API` | ^1.0 |
| إضافة `Tss.Basic` | ^1.0 |

---

## التثبيت والإعداد

### 1. تثبيت الإضافة

قم بنسخ مجلد الإضافة إلى المسار `plugins/nano3/kyc`، ثم نفذ أمر التحديث:

```bash
php artisan october:up
```

سيتم إنشاء جدول `nano3_kyc_documents` تلقائياً.

### 2. (اختياري) نشر ملف الإعدادات

لتخصيص خصائص أنواع الوثائق، انشر ملف الإعدادات:

```bash
php artisan config:publish nano3.kyc
```

سيتم إنشاء الملف `config/nano3/kyc/document_types.php` والذي يمكنك تعديله حسب احتياجاتك.

### 3. إعداد الصلاحيات

بعد التثبيت، انتقل إلى **الإعدادات → المسؤولون → الصلاحيات** ومنح الصلاحيات المناسبة للأدوار (مثل `documents.access`, `documents.verify`).

---

## أمثلة عملية للمطورين

### إنشاء وثيقة باستخدام `Manager`

```php
use Nano3\Kyc\Classes\Manager;

$result = Manager::createDocument([
    'document_type' => 'passport',
    'owner'         => $user, // كائن المستخدم
    'fields'        => [
        'document_number' => 'P12345678',
        'full_name'       => 'John Doe',
        'issue_date'      => '2020-01-01',
        'expiry_date'     => '2030-01-01',
        'country_of_issue'=> 'US',
        'nationality'     => 'US',
    ],
    'files' => [
        'document_front' => Input::file('front_image')
    ],
    'is_stop_duplicate' => true, // منع تكرار نفس رقم الوثيقة لنفس المالك
]);

if ($result['status']) {
    $document = $result['model'];
    echo "تم إنشاء الوثيقة برقم: " . $document->id;
} else {
    echo "خطأ: " . $result['error'];
}
```

### جلب حقول نوع وثيقة لبناء نموذج ديناميكي

```php
use Nano3\Kyc\Classes\DocumentType;

$fields = DocumentType::getFieldsSchema('drivers_license');

foreach ($fields as $fieldCode => $fieldDef) {
    echo "<label>" . DocumentType::getFieldLabel('drivers_license', $fieldCode) . "</label>";
    if ($fieldDef['type'] === 'text') {
        echo "<input type='text' name='fields[$fieldCode]' />";
    }
    // ... أنواع أخرى
}
```

### التحقق من صلاحية وثيقة

```php
$document = Document::find(1);
if (DocumentType::hasExpiry($document->document_type)) {
    if (!DocumentType::isDocumentAcceptable($document->document_type, $document->expiry_date)) {
        // الوثيقة منتهية الصلاحية
    }
}
```

---

## هيكل جدول `nano3_kyc_documents`

| الحقل | النوع | الوصف |
| :--- | :--- | :--- |
| `id` | bigint | معرف تلقائي |
| `document_type` | string(50) | كود نوع الوثيقة (مثال: `passport`) |
| `document_category` | string(50) | فئة الوثيقة (مثال: `primary_id`) |
| `code` / `barcode` | string(100) | رموز مرجعية |
| `document_number` | string(100) | رقم الوثيقة (مفهرس) |
| `issue_date` / `expiry_date` | date | تواريخ الإصدار والانتهاء |
| `full_name` | string(200) | الاسم الكامل (مفهرس) |
| `company_name` | string(200) | اسم الشركة (مفهرس) |
| `owner_id` / `owner_type` | polymorphic | المالك (فرد/شركة) |
| `verifier_id` / `verifier_type` | polymorphic | المحقق |
| `subject_id` / `subject_type` | polymorphic | كيان مرتبط (طلب/معاملة) |
| `is_verified` | boolean | حالة التحقق |
| `verified_at` | timestamp | تاريخ التحقق |
| `verification_score` | tinyint | درجة الموثوقية (0-100) |
| `status` | string(50) | الحالة (`pending`, `verified`, `rejected`, `expired`) |
| `fields_data` | json | جميع الحقول الديناميكية الخاصة بنوع الوثيقة |
| `metadata` / `other_data` / `config_data` | json | بيانات مرنة إضافية |
| `is_physical_submitted` | boolean | نسخة ورقية مسلمة |
| `physical_copy_type` | string(20) | `original` أو `copy` |
| `physical_received_at` / `physical_returned_at` | timestamp | تواريخ الاستلام والإعادة |
| `companys_id` / `departments_id` | string(190) | دعم تعدد الشركات والفروع |
| `timestamps` / `softDeletes` | - | تتبع زمني وحذف ناعم |

---

## التخصيص عبر ملف الإعدادات

يمكن تجاوز أي من خصائص أنواع الوثائق الافتراضية عبر ملف `config/nano3/kyc/document_types.php`. القيم الجديدة تدمج تلقائياً مع القيم الافتراضية.

**مثال: تخصيص جواز السفر**
```php
// config/nano3/kyc/document_types.php
return [
    'passport' => [
        'max_file_size_kb' => 5120, // 5MB بدلاً من 10MB
        'requires_selfie_match' => false,
        'fields' => [
            'expiry_date' => ['required' => true],
            'religion'    => ['required' => true], // جعل الديانة إلزامية
        ],
    ],
];
```

---

## الأحداث (Events)

يمكن للمطورين ربط دوال الاستماع (Listeners) بالأحداث التالية:

| الحدث | الوصف |
| :--- | :--- |
| `nano3.kyc.document.created` | عند إنشاء وثيقة جديدة |
| `nano3.kyc.document.updated` | عند تحديث وثيقة |
| `nano3.kyc.document.deleted` | عند حذف وثيقة (Soft Delete) |
| `nano3.kyc.document.restored` | عند استعادة وثيقة محذوفة |
| `nano3.kyc.document.verified` | عند اعتماد وثيقة |
| `nano3.kyc.document.rejected` | عند رفض وثيقة |

---

## الاعتماديات والإضافات المطلوبة

- **Nano.API**: توفر البنية الأساسية لـ API (المتحكمات، المحولات، التخزين المؤقت).
- **Tss.Basic**: توفر دوال مساعدة للشركات والأقسام (`BasicHelper`).

---

## الترخيص والدعم

هذه الإضافة ملكية خاصة بشركة **Nano2Soft**. جميع الحقوق محفوظة © 2025.

للاستفسارات والدعم الفني:
- الموقع: [https://nano2soft.com](https://nano2soft.com)
- البريد الإلكتروني: support@nano2soft.com


## التوثيق الإضافي

**الوثائق المرجعية**:
- [توثيق كلاس `DocumentType`](./Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
