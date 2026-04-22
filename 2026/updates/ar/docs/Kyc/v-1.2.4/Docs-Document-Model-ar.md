# توثيق موديل `Document` – نموذج وثائق KYC

## نظرة عامة

يقع موديل `Document` في المسار `Nano3\Kyc\Models\Document`، وهو نموذج Eloquent المسؤول عن تمثيل وإدارة جميع وثائق التحقق من الهوية (KYC) في قاعدة البيانات. يرتبط الموديل بجدول `nano3_kyc_documents` ويستخدم مجموعة غنية من السمات (Traits) لتوفير وظائف متقدمة مثل:

- نطاقات البحث والتصفية (Scopes).
- إدارة السجلات الافتراضية.
- خيارات القوائم المنسدلة للنماذج والفلاتر.
- دالة `getRecords` المتكاملة لجلب السجلات مع فلترة مرنة.
- التخزين المؤقت للكائنات والقوائم.
- علاقات متعددة الأشكال (Polymorphic) مع المالك (`owner`) والمحقق (`verifier`) والموضوع المرتبط (`subject`).

---

## هيكل الجدول `nano3_kyc_documents`

| مجموعة الحقول | الحقول الرئيسية |
| :--- | :--- |
| **المعرفات** | `id`, `code`, `barcode` |
| **نوع الوثيقة** | `document_type`, `document_category` |
| **البيانات الأساسية** | `document_number`, `issuing_authority`, `issue_date`, `expiry_date` |
| **البيانات الشخصية** | `full_name`, `date_of_birth`, `nationality`, `country_of_issue`, `religion` |
| **العنوان** | `address_line1`, `address_line2`, `address_city`, `address_state`, `address_postcode`, `address_country` |
| **بيانات الشركات** | `company_name`, `registration_number`, `tax_number`, `incorporation_date` |
| **العلاقات** | `owner_id`/`owner_type`, `verifier_id`/`verifier_type`, `subject_id`/`subject_type` |
| **التحقق والحالة** | `is_verified`, `verified_at`, `verification_score`, `status` |
| **النسخ الورقية** | `is_physical_submitted`, `physical_copy_type`, `physical_received_at`, `physical_returned_at`, `physical_notes` |
| **النشر** | `is_published`, `published_at`, `unpublished_at`, `is_public`, `is_active`, `is_default` |
| **بيانات مرنة** | `fields_data`, `metadata`, `other_data`, `config_data` (JSON) |
| **ملفات** | `file_ids`, `file_urls` |
| **التتبع** | `created_by`, `updated_by`, `deleted_by`, `timestamps`, `softDeletes` |
| **متعددة الشركات** | `companys_id`, `departments_id` |

---

## السمات (Traits) المستخدمة في الموديل

يستخدم موديل `Document` السمات التالية لتنظيم الكود وإضافة الوظائف:

| السمة | الوصف |
| :--- | :--- |
| `HasScopesModel` | نطاقات البحث والتصفية الأساسية (مثل `isActive`, `whereDocumentType`). |
| `HasDefault` | إدارة السجل الافتراضي (`getDefault`, `makeDefault`). |
| `HasUserScopes` / `HasUserOptions` | نطاقات وخيارات فلترة المستخدم (للتوافق مع إضافات أخرى). |
| `HasSubjectScopes` / `HasSubjectOptions` | نطاقات وخيارات فلترة الموضوع المرتبط (`subject`). |
| `HasOwnerScopes` / `HasOwnerOptions` | نطاقات وخيارات فلترة المالك (`owner`). |
| `HasVerifierScopes` / `HasVerifierOptions` | نطاقات وخيارات فلترة المحقق (`verifier`). |
| `HasRecordsOptions` | دالة `getRecords` المتكاملة لجلب السجلات مع فلترة شاملة. |
| `ListObjects` | دوال التخزين المؤقت للكائنات (`getRecordObj`, `clearCacheObjectKeyModel`). |
| `ListOptions` | دوال مساعدة لجلب القوائم المنسدلة (مثل `listAvailableDepartments`). |
| `FieldsOptions` | دوال خيارات القوائم المنسدلة للنماذج الخلفية (`getStatusOptions`, `getDocumentTypeOptions`). |

---

## السمة `HasScopesModel` – نطاقات البحث الأساسية

توفر هذه السمة مجموعة من النطاقات (Scopes) الجاهزة للاستخدام في استعلامات Eloquent.

### النطاقات المتاحة

| النطاق | الوصف | مثال الاستخدام |
| :--- | :--- | :--- |
| `IsCompany($companys_id)` | تصفية حسب الشركة. | `Document::IsCompany(5)->get()` |
| `IsDepartment($departments_id)` | تصفية حسب الفرع (مع دعم الأقسام الرئيسية والفرعية). | `Document::IsDepartment(10)->get()` |
| `Departments($departments_id)` | تصفية مرنة حسب الفرع. | `Document::Departments(10)->get()` |
| `IsActive()` | السجلات النشطة فقط (`is_active = true`). | `Document::IsActive()->get()` |
| `IsNotActive()` | السجلات غير النشطة. | `Document::IsNotActive()->get()` |
| `IsPublished()` | السجلات المنشورة والتي تقع ضمن نطاق تواريخ النشر. | `Document::IsPublished()->get()` |
| `IsNotPublished()` | السجلات غير المنشورة أو خارج نطاق التواريخ. | `Document::IsNotPublished()->get()` |
| `WhereDocumentType($value, $is_or, $is_not, $is_force)` | فلترة حسب نوع الوثيقة. | `Document::WhereDocumentType('passport')->get()` |
| `WhereDocumentCategory($value, ...)` | فلترة حسب فئة الوثيقة. | `Document::WhereDocumentCategory('primary_id')->get()` |
| `WhereDocumentNumber($value, ...)` | فلترة حسب رقم الوثيقة. | `Document::WhereDocumentNumber('A123')->get()` |
| `WhereFullName($value, ...)` | فلترة حسب الاسم الكامل. | `Document::WhereFullName('John')->get()` |
| `WhereCompanyName($value, ...)` | فلترة حسب اسم الشركة. | `Document::WhereCompanyName('Nano')->get()` |
| `WhereStatus($value, ...)` | فلترة حسب الحالة. | `Document::WhereStatus('pending')->get()` |
| `WhereIsVerified($value)` | فلترة حسب حالة التحقق. | `Document::WhereIsVerified(true)->get()` |
| `IsExpired()` | الوثائق المنتهية الصلاحية (`expiry_date < NOW()`). | `Document::IsExpired()->get()` |
| `IsExpiringSoon($days)` | الوثائق التي ستنتهي خلال عدد معين من الأيام. | `Document::IsExpiringSoon(30)->get()` |

**ملاحظة:** النطاقات التي تحتوي على معاملات `$is_or`, `$is_not`, `$is_force` تسمح بتحكم دقيق في سلوك الشرط. `$is_force` يجبر تطبيق الشرط حتى لو كانت القيمة `*` أو `all`.

---

## السمة `HasRecordsOptions` – دالة `getRecords` المتكاملة

تعتبر دالة `getRecords` من أقوى الأدوات في الموديل، حيث تسمح بجلب السجلات مع خيارات فلترة شاملة وتنسيقات متعددة للإخراج.

### توقيع الدالة

```php
public static function getRecords(array $options = [], bool $isException = false): array
```

### هيكل الاستجابة

تعيد الدالة مصفوفة بالهيكل التالي:

```php
[
    'code' => 200,
    'status' => true,
    'message' => '...',
    'error' => null,
    'errors' => null,
    'data' => $result, // قد يكون Collection أو Paginator أو Query Builder
    'input_data' => [...],
    'process_data' => [...]
]
```

### الخيارات المدعومة (`$options`)

| المفتاح | النوع | الوصف |
| :--- | :--- | :--- |
| `id` | `mixed` | فلترة حسب المعرف. |
| `is_force_id` | `bool` | إجبار فلترة `id` حتى لو كانت `*`. |
| `companys_id` | `string` | فلترة حسب الشركة. |
| `departments_id` | `string` | فلترة حسب الفرع. |
| `document_type` | `mixed` | فلترة حسب نوع الوثيقة. |
| `is_or_document_type` | `bool` | استخدام `OR` لشرط `document_type`. |
| `is_not_document_type` | `bool` | نفي شرط `document_type`. |
| `is_force_document_type` | `bool` | إجبار الفلترة. |
| `document_category` | `mixed` | فلترة حسب فئة الوثيقة (مع مفاتيح `is_or`, `is_not`, `is_force`). |
| `document_number` | `mixed` | فلترة حسب رقم الوثيقة. |
| `full_name` | `mixed` | فلترة حسب الاسم الكامل. |
| `company_name` | `mixed` | فلترة حسب اسم الشركة. |
| `status` | `mixed` | فلترة حسب الحالة. |
| `is_verified` | `mixed` | فلترة حسب حالة التحقق (يمكن أن تكون `true`, `false`, `'*'`). |
| `is_active` | `mixed` | فلترة حسب النشاط. |
| `is_default` | `mixed` | فلترة حسب الافتراضي. |
| `is_public` | `mixed` | فلترة حسب العام. |
| `is_published` | `mixed` | فلترة حسب المنشور. |
| `owner` / `owner_id` / `owner_type` | `mixed` | فلترة حسب المالك (يدعم كائن أو معرف ونوع). |
| `verifier` / `verifier_id` / `verifier_type` | `mixed` | فلترة حسب المحقق. |
| `subject` / `subject_id` / `subject_type` | `mixed` | فلترة حسب الموضوع المرتبط. |
| `issue_date` / `expiry_date` | `string\|array` | فلترة حسب تاريخ الإصدار/الانتهاء (يدعم نطاق). |
| `q` | `string` | بحث نصي في (`document_number`, `full_name`, `company_name`, `code`). |
| `orderBy` | `string` | اسم العمود للترتيب (الافتراضي: `created_at`). |
| `orderDirection` | `string` | اتجاه الترتيب (`asc` أو `desc`). |
| `is_paginator` | `bool` | إرجاع `Paginator` بدلاً من `Collection`. |
| `is_collection` | `bool` | إرجاع `Collection`. |
| `is_query` | `bool` | إرجاع كائن `Builder` للاستعلام. |
| `is_first` | `bool` | إرجاع أول سجل فقط. |
| `per_page` | `int` | عدد السجلات لكل صفحة (عند التصفح). |
| `page` | `int` | رقم الصفحة. |
| `withTrashed` | `bool` | تضمين السجلات المحذوفة. |
| `onlyTrashed` | `bool` | السجلات المحذوفة فقط. |
| `exclude` | `string` | أسماء أعمدة مفصولة بفواصل لاستبعادها من النتائج. |

### أمثلة على استخدام `getRecords`

#### 1. جلب جميع الوثائق النشطة من نوع "جواز سفر"

```php
$result = Document::getRecords([
    'document_type' => 'passport',
    'is_active' => true,
    'is_paginator' => true,
    'per_page' => 20,
]);

if ($result['status']) {
    $documents = $result['data']; // LengthAwarePaginator
    foreach ($documents as $doc) {
        echo $doc->document_number;
    }
}
```

#### 2. البحث عن وثائق لمستخدم معين مع فلترة تاريخ

```php
$result = Document::getRecords([
    'owner' => $user, // كائن المستخدم
    'issue_date' => ['2024-01-01', '2024-12-31'],
    'status' => 'verified',
    'orderBy' => 'created_at',
    'orderDirection' => 'desc',
]);

$documents = $result['data']; // Collection
```

#### 3. جلب استعلام جاهز للتعديل

```php
$result = Document::getRecords([
    'is_verified' => false,
    'is_query' => true,
]);

if ($result['status']) {
    $query = $result['data']; // Builder
    $query->where('verification_score', '>', 50);
    $docs = $query->get();
}
```

#### 4. استخدام البحث النصي

```php
$result = Document::getRecords([
    'q' => 'NanoSoft',
    'is_paginator' => true,
]);

// يبحث في document_number, full_name, company_name, code
```

---

## سمات `HasOwnerScopes` و `HasOwnerOptions`

### `HasOwnerScopes` – نطاقات المالك

توفر هذه السمة نطاقات متقدمة للفلترة حسب المالك (`owner`).

| النطاق | الوصف |
| :--- | :--- |
| `WhereOwnerType($value, $is_or, $is_not, $is_force)` | فلترة حسب نوع المالك. |
| `WhereOwnerId($value, ...)` | فلترة حسب معرف المالك. |
| `ByOwner($query, ...$params)` | **نطاق متقدم** يقبل كائنًا أو مصفوفة خيارات مفصلة. |

#### النطاق المتقدم `byOwner`

يمكن استدعاؤه بعدة طرق:

```php
// 1. تمرير كائن مباشرة
Document::byOwner($user)->get();

// 2. تمرير مصفوفة خيارات مفصلة
Document::byOwner([
    'owner_type' => ['value' => 'RainLab\User\Models\User', 'is_force' => true],
    'owner_id'   => ['value' => 5, 'is_or' => true],
    'global_is_or' => false,
    'logic_between' => 'OR',
])->get();

// 3. معاملات منفصلة (للتوافق)
Document::byOwner($user, null, null, false, false, true)->get();
```

يدعم `byOwner` الخصائص التالية على مستوى الحقل أو المجموعة:

- `is_or`: يجعل الشرط `OR` بدلاً من `AND`.
- `is_not`: ينفي الشرط.
- `is_force`: يجبر تطبيق الشرط حتى لو كانت القيمة `*` أو `all`.
- `logic_between`: العلاقة بين `owner_type` و `owner_id` (`AND` أو `OR`).

نطاقات أخرى:

- `WhereNotNullOwner()` / `WhereNullOwner()`: فلترة السجلات التي لها مالك أو لا تملك.
- `WhereHasOwnerTrashed()`: جلب السجلات التي تم حذف مالكها.

### `HasOwnerOptions` – خيارات المالك

توفر دوال لتوليد خيارات القوائم المنسدلة في النماذج والفلاتر:

| الدالة | الوصف |
| :--- | :--- |
| `getOwnerTypeOptions()` | قائمة بأنواع المالكين المسجلة في النظام. |
| `getOwnerIdOptions()` | قائمة بمعرفات المالكين (تعتمد على `owner_type`). |
| `getOwnerTypeFilterOptions($scopes)` | خيارات فلتر نوع المالك. |
| `getOwnerIdFilterOptions($scopes)` | خيارات فلتر معرف المالك. |
| `getOwnerNameAttribute()` | محوّل (Accessor) لإرجاع اسم المالك. |

**مثال في `fields.yaml`:**

```yaml
owner_type:
    label: نوع المالك
    type: dropdown
    options: getOwnerTypeOptions

owner_id:
    label: المالك
    type: dropdown
    options: getOwnerIdOptions
    dependsOn: owner_type
```

---

## باقي السمات بشكل موجز

### `HasSubjectScopes` / `HasSubjectOptions`

نطاقات وخيارات مشابهة لـ `Owner` ولكن للعلاقة `subject`. تدعم نفس الخيارات المتقدمة في `bySubject`.

### `HasVerifierScopes` / `HasVerifierOptions`

نطاقات وخيارات للعلاقة `verifier` (المحقق). تستخدم للفلترة حسب من قام بالتحقق من الوثيقة.

### `HasDefault`

إدارة السجل الافتراضي في كل فرع. الدوال الرئيسية:

- `getDefault($departments_id)`: جلب الوثيقة الافتراضية.
- `makeDefault()`: تعيين الوثيقة الحالية كافتراضية (مع إلغاء الافتراضية عن باقي الوثائق من نفس النوع والمالك).

### `ListObjects`

دوال التخزين المؤقت للكائنات لتسريع الأداء:

- `getRecordObj($key)`: جلب كائن وثيقة من الكاش أو قاعدة البيانات (يدعم `id` أو `code`).
- `clearCacheObjectKeyModel($key)`: مسح الكاش لسجل محدد.
- `clearCacheListObjectKeyByDepartments()`: مسح الكاش لجميع سجلات فرع معين.

### `ListOptions`

دوال مساعدة لجلب قوائم منسدلة (للشركات والأقسام) مع تخزين مؤقت:

- `listAvailableCompanys()` / `listEnabledCompanys()`
- `listAvailableDepartments()` / `listEnabledDepartments()`
- `clearCacheListByCompanys()` / `clearCacheListByDepartments()`

### `FieldsOptions`

دوال توليد خيارات القوائم المنسدلة المستخدمة في نماذج الإدارة الخلفية:

- `getStatusOptions()`
- `getDocumentTypeOptions()`
- `getDocumentCategoryOptions()`
- `getPhysicalCopyTypeOptions()`
- `getDepartmentsIdOptions()`
- `getPartyTypeOptions()`

---

## أحداث الموديل (Model Events)

يرتبط الموديل بالأحداث التالية خلال دورة حياته:

| الحدث | الوصف |
| :--- | :--- |
| `beforeValidate` | قبل التحقق من صحة البيانات. |
| `beforeSave` | قبل الحفظ. يتم فيها ضبط `companys_id` و `departments_id` وتحديث `verified_at` عند تغيير `is_verified`. |
| `beforeCreate` | قبل الإنشاء. تعيين `created_by`. |
| `beforeUpdate` | قبل التحديث. تعيين `updated_by`. |
| `beforeDelete` | قبل الحذف. تعيين `deleted_by`. |
| `afterSave` | بعد الحفظ. تحديث الكاش وإدارة السجل الافتراضي. |
| `afterDelete` | بعد الحذف. |

---

## محولات (Accessors) مفيدة

| المحول | الوصف |
| :--- | :--- |
| `getDocumentTypeNameAttribute()` | إرجاع الاسم المعرّب لنوع الوثيقة. |
| `getStatusColorAttribute()` | لون Bootstrap المناسب للحالة. |
| `getStatusIconAttribute()` | أيقونة Font Awesome المناسبة للحالة. |
| `isExpired()` | هل الوثيقة منتهية الصلاحية؟ |
| `isExpiringSoon($days)` | هل ستنتهي صلاحيتها خلال `$days` أيام؟ |

---

## مثال متكامل: إنشاء وتحديث وجلب وثيقة

```php
use Nano3\Kyc\Models\Document;
use Nano3\Kyc\Classes\Manager;

// إنشاء وثيقة
$result = Manager::createDocument([
    'document_type' => 'national_id',
    'owner'         => Auth::getUser(),
    'fields'        => [
        'document_number' => '987654321',
        'full_name'       => 'أحمد محمد',
        'issue_date'      => '2022-05-10',
        'expiry_date'     => '2032-05-10',
    ],
    'files' => ['document_front' => $uploadedFile],
]);

if ($result['status']) {
    $document = $result['model'];
}

// جلب الوثائق النشطة لمستخدم معين
$docs = Document::byOwner(Auth::getUser())
    ->IsActive()
    ->orderBy('created_at', 'desc')
    ->get();

// استخدام getRecords مع فلترة متقدمة
$result = Document::getRecords([
    'document_type' => 'passport',
    'is_verified' => true,
    'expiry_date' => ['2025-01-01', '2025-12-31'],
    'is_paginator' => true,
]);
```

---

## الاعتماديات

- `Tss\Basic\Helpers\BasicHelper` – لإدارة الشركات والأقسام.
- `Nano3\Kyc\Classes\DocumentType` – للحصول على معلومات أنواع الوثائق.
- `Nano3\Kyc\Classes\Manager` – لبعض الدوال المساعدة (`scopeWhereField`, `getQueryDate`).

---


## التوثيق الإضافي

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
