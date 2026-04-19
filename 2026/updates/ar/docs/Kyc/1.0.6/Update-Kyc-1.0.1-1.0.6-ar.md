## 2026-04-19 – 2026-04-20

**تحديثات إضافة `Nano3.Kyc` – الإصدارات 1.0.1 إلى 1.0.6**

### ملخص التحديثات

تم تطوير إضافة `Nano3.Kyc` لتكون الحل المتكامل لإدارة عمليات التحقق من الهوية (KYC) ضمن برمجيات نانوسوفت (NanoSoft App). تضمنت التحديثات في الإصدارات من 1.0.1 إلى 1.0.6 بناء نظام متكامل يشمل:

- **إنشاء هيكل قاعدة البيانات** وجدول `nano3_kyc_documents` المصمم لاستيعاب جميع أنواع وثائق KYC.
- **تطوير كلاس `DocumentType`** الذي يعرف أكثر من 28 نوع وثيقة مع خصائصها وحقولها الديناميكية وقواعد التحقق.
- **تطوير كلاس `Manager`** المسؤول عن جميع عمليات إدارة الوثائق (إنشاء، تحديث، حذف، اعتماد، رفض) مع دعم المعاملات والأحداث والاختبارات.
- **بناء موديل `Document`** مع مجموعة متكاملة من السمات (Traits) لإدارة النطاقات، الخيارات، التخزين المؤقت، والفلاتر.
- **تطوير واجهة API كاملة** عبر متحكم `Nano3\Kyc\APIControllers\Documents` بنمط RESTful.
- **تطوير واجهة خلفية متكاملة** عبر متحكم `Nano3\Kyc\Controllers\Documents` مع قوائم ونماذج وفلاتر ديناميكية.
- **دعم كامل لتعدد اللغات** (العربية والإنجليزية) مع ملف ترجمة شامل.
- **نظام تخصيص مرن** عبر ملف إعدادات `config/nano3/kyc/document_types.php`.

---

### الإصدار 1.0.1 – الإطلاق الأولي وإنشاء جدول الوثائق

#### أهداف الإصدار

- إنشاء الإضافة الأساسية `Nano3.Kyc` ضمن برمجيات نانوسوفت.
- تصميم وإنشاء جدول `nano3_kyc_documents` ليكون العمود الفقري لتخزين جميع وثائق KYC.

#### الميزات الجديدة

##### 1. إنشاء جدول `nano3_kyc_documents`

تم تصميم جدول متكامل يدعم جميع أنواع وثائق KYC مع مرونة عالية:

| مجموعة الحقول | الوصف |
| :--- | :--- |
| **المعرفات الأساسية** | `document_type`، `document_category`، `code`، `barcode`، `document_number` |
| **التواريخ** | `issue_date`، `expiry_date` (نوع `date` للأداء)، `verified_at` |
| **البيانات الشخصية** | `full_name`، `date_of_birth`، `nationality`، `country_of_issue`، `religion` |
| **العنوان** | `address_line1`، `address_line2`، `address_city`، `address_state`، `address_postcode`، `address_country` |
| **بيانات الشركات** | `company_name`، `registration_number`، `tax_number`، `incorporation_date` |
| **العلاقات المتعددة** | `owner_id`/`owner_type`، `verifier_id`/`verifier_type`، `subject_id`/`subject_type` |
| **التحقق** | `is_verified`، `verification_score`، `verified_at` |
| **النسخ الورقية** | `is_physical_submitted`، `physical_copy_type`، `physical_received_at`، `physical_returned_at`، `physical_notes` |
| **الحالة والنشر** | `status`، `is_active`، `is_default`، `is_public`، `is_published`، `published_at`، `unpublished_at` |
| **بيانات مرنة (JSON)** | `fields_data`، `metadata`، `other_data`، `config_data` |
| **الملفات** | `file_ids`، `file_urls` |
| **التتبع** | `timestamps`، `softDeletes`، `created_by`، `updated_by`، `deleted_by` |

تمت إضافة فهارس على الحقول الأكثر استخداماً في البحث (`document_type`، `document_number`، `full_name`، `company_name`، `status`، `is_verified`، العلاقات المتعددة) لضمان أداء عالي.

##### 2. هيكل الإضافة الأساسي

تم إنشاء الهيكل الأساسي للإضافة:
```
plugins/nano3/kyc/
├── Plugin.php              # ملف تعريف الإضافة
├── updates/
│   ├── version.yaml        # سجل الإصدارات
│   └── create_table_nano3_kyc_documents.php
└── lang/                   # مجلد الترجمة (سيتم ملؤه لاحقاً)
```

#### الفوائد

- قاعدة بيانات جاهزة لاستيعاب أي نوع وثيقة KYC حالية أو مستقبلية.
- مرونة عالية بفضل حقل `fields_data` (JSON) الذي يخزن الحقول المتخصصة لكل نوع وثيقة.
- فهارس محسنة للاستعلامات الشائعة.

---

### الإصدار 1.0.2 – تطوير كلاس `DocumentType` وموديل `Document` مع السمات الأساسية

#### أهداف الإصدار

- تطوير كلاس `DocumentType` ليكون المرجع المركزي لتعريف أنواع الوثائق وخصائصها.
- بناء موديل `Document` مع مجموعة السمات (Traits) الأساسية.

#### الميزات الجديدة

##### 1. كلاس `DocumentType`

تم تطوير كلاس `DocumentType` (`Nano3\Kyc\Classes\DocumentType`) الذي يوفر:

- **تعريف 28+ نوع وثيقة** موزعة على 6 فئات (هوية أساسية، هوية ثانوية، إثبات عنوان، وثائق شركات، UBO، تحقق إضافي).
- **خصائص متقدمة لكل نوع**: `has_expiry`، `max_age_days`، `verification_weight`، `allowed_mime_types`، `max_file_size_kb`، `allowed_for_entity`، `requires_back_side`، `requires_selfie_match`.
- **مخطط حقول ديناميكي (`fields`)**: تعريف كامل للحقول المطلوبة لكل نوع وثيقة مع خصائصها (`label`، `type`، `required`، `validation`، `order`، `extractable`).
- **دوال مساعدة للتحقق**: `getValidationRules()`، `isValidType()`، `isValidForPartyType()`، `isDocumentAcceptable()`، `isDocumentExpiring()`.
- **دوال لتحويل البيانات**: `mapInputToDocumentData()` و `mapDocumentToOutput()` لتسهيل التعامل مع هيكل التخزين المختلط (أعمدة ثابتة + JSON).
- **تحميل الإعدادات من ملف `config`**: مع إمكانية تجاوز القيم الافتراضية البرمجية.

**مثال على تعريف نوع وثيقة:**
```php
'passport' => [
    'category' => 'primary_id',
    'has_expiry' => true,
    'verification_weight' => 100,
    'allowed_mime_types' => ['image/jpeg', 'image/png', 'application/pdf'],
    'fields' => [
        'document_number' => ['type' => 'text', 'required' => true],
        'full_name' => ['type' => 'text', 'required' => true],
        'expiry_date' => ['type' => 'date', 'required' => true, 'validation' => ['after_or_equal' => 'today']],
        // ... إلخ
    ],
]
```

##### 2. موديل `Document` والسمات المرتبطة

تم بناء موديل `Document` (`Nano3\Kyc\Models\Document`) مع مجموعة سمات (Traits) تغطي جميع جوانب إدارة الوثائق:

| السمة | الوصف |
| :--- | :--- |
| `HasScopesModel` | نطاقات البحث الأساسية (`IsCompany`، `Departments`، `IsActive`، `IsPublished`، `WhereDocumentType`، ...). |
| `HasDefault` | إدارة السجل الافتراضي (`getDefault`، `makeDefault`). |
| `HasOwnerScopes` / `HasOwnerOptions` | نطاقات وخيارات فلترة المالك (`owner`). |
| `HasVerifierScopes` / `HasVerifierOptions` | نطاقات وخيارات فلترة المحقق (`verifier`). |
| `HasSubjectScopes` / `HasSubjectOptions` | نطاقات وخيارات فلترة الموضوع المرتبط (`subject`). |
| `HasRecordsOptions` | دالة `getRecords()` المتكاملة لجلب السجلات مع فلاتر مرنة وتنسيقات متعددة (مجموعة، paginator، query). |
| `ListObjects` / `ListOptions` | دوال مساعدة لجلب السجلات المخزنة مؤقتاً والقوائم المنسدلة. |
| `FieldsOptions` | دوال خيارات القوائم المنسدلة للنماذج (`getStatusOptions`، `getDocumentTypeOptions`، ...). |
| `HasCreateDefaultRecords` | دوال إنشاء السجلات الافتراضية (للتوافق مع هيكلية الإضافات الأخرى). |

##### 3. دالة `getRecords` المتكاملة

تدعم دالة `getRecords` في موديل `Document` خيارات فلترة شاملة:
- `id`، `document_type`، `document_category`، `document_number`، `full_name`، `company_name`، `status`، `is_verified`، `is_active`، `is_published`
- `owner_id`/`owner_type`، `verifier_id`/`verifier_type`، `subject_id`/`subject_type`
- نطاقات تاريخ (`issue_date`، `expiry_date`، `created_at`، `updated_at`)
- البحث النصي (`q`)
- دعم `is_paginator`، `is_collection`، `is_query`، `is_first`

#### الفوائد

- تعريف مركزي لجميع أنواع الوثائق يسهل الصيانة والتوسع.
- حقول ديناميكية تسمح ببناء نماذج إدخال تكيفية حسب نوع الوثيقة.
- تحويل تلقائي للبيانات بين المدخلات وقاعدة البيانات.
- موديل `Document` قوي مع نطاقات جاهزة للاستخدام في القوائم والفلاتر.
- تخزين مؤقت (Cache) لتحسين الأداء في `ListObjects` و `ListOptions`.

---

### الإصدار 1.0.3 – تطوير كلاس `Manager` (مدير العمليات)

#### أهداف الإصدار

- تطوير كلاس `Manager` (`Nano3\Kyc\Classes\Manager`) ليكون المسؤول عن جميع عمليات إدارة الوثائق.
- توفير دوال موحدة لإنشاء، تحديث، حذف، اعتماد، ورفض الوثائق مع دعم المعاملات، الأحداث، والاختبارات.

#### الميزات الجديدة

##### 1. كلاس `Manager` – مدير عمليات KYC

يوفر كلاس `Manager` الدوال التالية:

| الدالة | الوصف |
| :--- | :--- |
| `createDocument(array $options, bool $is_test_create)` | إنشاء وثيقة جديدة مع التحقق من الصلاحية، التكرار، والأحداث. |
| `updateDocument($documentId, array $options, bool $is_test_create)` | تحديث وثيقة موجودة. |
| `deleteDocument($documentId, bool $is_test_create)` | حذف وثيقة (Soft Delete). |
| `restoreDocument($documentId)` | استعادة وثيقة محذوفة. |
| `verifyDocument($documentId, $verifier, $verification_score)` | اعتماد وثيقة (تحديث الحالة إلى `verified`). |
| `rejectDocument($documentId, $reason)` | رفض وثيقة مع إمكانية إضافة سبب. |
| `checkDuplicateDocument(array $options)` | التحقق من وجود وثيقة مكررة لنفس المالك. |
| `getDocumentRecords($options)` | جلب سجلات الوثائق باستخدام دالة `getRecords` من الموديل. |
| `getDocumentStats($options)` | جلب إحصائيات عن الوثائق. |

##### 2. هيكل استجابة موحد

جميع دوال `Manager` تعيد هيكل استجابة موحد:

```php
[
    'code' => 200,
    'status' => true,
    'message' => 'رسالة النجاح',
    'error' => null,
    'errors' => null,
    'model' => $document,
    'data' => $document->toArray(),
    'input_data' => [...],
    'process_data' => [...]
]
```

##### 3. دعم الاختبارات (`is_test_create`)

جميع الدوال تدعم معامل `is_test_create` الذي يسمح بتنفيذ العملية ضمن معاملة يتم التراجع عنها (`rollback`) بعد التحقق من نجاحها، مما يسهل اختبار الوظائف دون التأثير على البيانات الحقيقية.

##### 4. دعم الأحداث والتحكم بها

- يمكن إيقاف إطلاق الأحداث عبر خيار `is_stop_event`.
- الأحداث المدعومة: `nano3.kyc.document.created`، `updated`، `deleted`، `restored`، `verified`، `rejected`.

##### 5. التحقق من التكرار

دالة `checkDuplicateDocument` تتحقق من وجود وثيقة مطابقة بناءً على حقول قابلة للتخصيص (`document_number`، `owner_id`، `owner_type`، `document_type`).

##### 6. دوال مساعدة للتوافق

- `getQueryDate()`: تطبيق فلتر تاريخ مرن على الاستعلامات.
- `checkValueIsNotAll()`: التحقق من أن القيمة ليست عامة (`*` أو `all`).
- `scopeWhereField()`: نطاق عام لتطبيق شروط `WHERE` مع دعم `is_or` و `is_not` و `is_force`.

#### الفوائد

- واجهة موحدة لجميع عمليات إدارة الوثائق.
- أمان عالي مع التحقق من الصلاحيات والتكرار.
- دعم كامل للمعاملات يضمن تكامل البيانات.
- سهولة الاختبار بفضل `is_test_create`.
- مرونة في تخصيص سلوك العمليات عبر الأحداث.

---

### الإصدار 1.0.4 – تطوير متحكم API ومتحكم الواجهة الخلفية

#### أهداف الإصدار

- تطوير متحكم API (`Nano3\Kyc\APIControllers\Documents`) لتوفير نقاط نهاية RESTful لإدارة الوثائق.
- تطوير متحكم الواجهة الخلفية (`Nano3\Kyc\Controllers\Documents`) لإدارة الوثائق من لوحة التحكم.

#### الميزات الجديدة

##### 1. متحكم API (`Documents`)

يوفر المتحكم نقاط النهاية التالية (المسار الأساسي: `/api/v1/kyc`):

| المسار | الطريقة | الوصف |
| :--- | :--- | :--- |
| `/documents` | `GET` | عرض قائمة الوثائق مع فلترة وبحث وتصفية. |
| `/documents` | `POST` | إنشاء وثيقة جديدة. |
| `/documents/{id}` | `GET` | عرض تفاصيل وثيقة. |
| `/documents/{id}` | `PUT` | تحديث وثيقة. |
| `/documents/{id}` | `DELETE` | حذف وثيقة. |
| `/documents/{id}/verify` | `POST` | اعتماد وثيقة. |
| `/documents/{id}/reject` | `POST` | رفض وثيقة. |
| `/documents/{id}/restore` | `POST` | استعادة وثيقة محذوفة. |
| `/document-types` | `GET` | جلب أنواع الوثائق المدعومة (مسطحة أو مجمعة). |
| `/document-types/{id}` | `GET` | جلب تفاصيل نوع وثيقة محدد. |
| `/document-fields/{type}` | `GET` | جلب حقول نوع وثيقة (لبناء نماذج ديناميكية). |
| `/documents/stats` | `GET` | جلب إحصائيات سريعة عن الوثائق. |
| `/documents/activelystats` | `GET` | التحقق من وجود تحديثات جديدة (للتخزين المؤقت). |

**مميزات متحكم API:**
- استخدام `Manager` لتنفيذ العمليات.
- دعم التخزين المؤقت (Cache) لتحسين الأداء.
- هيكل استجابة موحد متوافق مع `Nano.API`.
- معالجة أخطاء احترافية مع رسائل قابلة للترجمة.

##### 2. محول البيانات `DocumentTransformer`

تم تطوير `DocumentTransformer` (`Nano3\Kyc\Transformers\DocumentTransformer`) لتنسيق استجابات API، مع دعم:
- تضمين العلاقات (`owner`، `verifier`، `subject`، `company`، `department`).
- تنسيق التواريخ والحالات.
- معالجة حقول JSON (`fields_data`، `metadata`).
- استبعاد الحقول حسب الطلب (`exclude`).

##### 3. متحكم الواجهة الخلفية (`Documents`)

يوفر المتحكم إدارة كاملة للوثائق من لوحة التحكم:
- **قائمة الوثائق**: مع إمكانية البحث، التصفية، الفرز، والإجراءات الجماعية.
- **نموذج إنشاء/تعديل**: مقسم إلى تبويبات منظمة (أساسي، المالك، التحقق، النسخة الورقية، الملفات، النشر، العلاقات، التدقيق).
- **فلاتر متقدمة**: عبر ملف `config_filter.yaml` تشمل جميع حقول الجدول المهمة.
- **أعمدة قابلة للتخصيص**: عبر `columns.yaml`.

##### 4. ملفات التكوين للواجهة الخلفية

| الملف | الوصف |
| :--- | :--- |
| `config_filter.yaml` | تعريف فلاتر القائمة (نوع الوثيقة، الحالة، المالك، التواريخ، ...). |
| `columns.yaml` | تعريف أعمدة القائمة مع إمكانية البحث والفرز. |
| `fields.yaml` | تعريف حقول نموذج الإدخال مع تقسيمها إلى تبويبات. |

#### الفوائد

- واجهة API جاهزة للتكامل مع تطبيقات الجوال والويب.
- واجهة خلفية سهلة الاستخدام لإدارة الوثائق يدوياً.
- تنسيق موحد للبيانات عبر `DocumentTransformer`.
- فلاتر وأعمدة قابلة للتخصيص حسب احتياجات كل مشروع.

---

### الإصدار 1.0.5 – تحسين ملفات الترجمة والقوائم والنماذج

#### أهداف الإصدار

- إكمال ملف الترجمة العربية `lang.php` ليشمل جميع المفاتيح المستخدمة في المتحكمات والقوائم والنماذج.
- تحسين ملفات `columns.yaml` و `fields.yaml` لتوفير تجربة مستخدم محسنة في الواجهة الخلفية.

#### الميزات الجديدة

##### 1. ملف الترجمة الشامل `lang.php`

تم إعداد ملف ترجمة عربي كامل يحتوي على:

- **الأيقونات**: تعريف أيقونات القوائم والحالات.
- **القوائم**: عناوين القوائم الرئيسية والفرعية مع أوصافها.
- **الصلاحيات**: مفاتيح ونصوص الصلاحيات لجميع العمليات.
- **الرسائل**: رسائل النجاح والخطأ والتحذير لجميع العمليات.
- **ترجمات المتحكمات**: ترجمات خاصة بمتحكم `Documents` تشمل التبويبات، الأزرار، الرسائل، الفلاتر، حقول النماذج، وأعمدة القوائم.
- **ترجمات أنواع الوثائق**: فئات، أنواع، أوصاف، وحقول كل نوع وثيقة.

##### 2. تحسين `columns.yaml`

تم تحسين ملف أعمدة القائمة ليشمل:
- **أعمدة مرئية افتراضياً**: `id`، `document_type`، `document_number`، `full_name`، `company_name`، `owner`، `status`، `is_verified`، `issue_date`، `expiry_date`، `created_at`.
- **أعمدة مخفية قابلة للإظهار**: `code`، `document_category`، `verifier`، `verification_score`، `is_physical_submitted`، `departments_id`، `subject_name`، `updated_at`.
- **تنسيقات مخصصة**: استخدام `formatter` لعرض اسم نوع الوثيقة والمالك والمحقق بشكل مناسب.
- **دعم البحث والفرز** لمعظم الأعمدة.

##### 3. تحسين `fields.yaml`

تم تنظيم حقول نموذج الإدخال في تبويبات:

| التبويب | الحقول |
| :--- | :--- |
| **الأساسية** | `document_category`، `document_type`، `document_number`، `issuing_authority`، `issue_date`، `expiry_date` |
| **المالك** | `owner_type`، `owner_id` (مع دعم `@create` و `@update` للتحكم في قابلية التعديل) |
| **التحقق** | `verifier_type`، `verifier_id`، `is_verified`، `verification_score`، `verified_at` |
| **النسخة الورقية** | `is_physical_submitted`، `physical_copy_type`، `physical_received_at`، `physical_returned_at`، `physical_notes` |
| **الملفات** | `document_front`، `document_back`، `signature_image`، `image`، `files` |
| **النشر والحالة** | `status`، `is_published`، `published_at`، `unpublished_at`، `is_active`، `is_default`، `is_public` |
| **علاقات إضافية** | `subject_type`، `subject_id`، `extend_id`، `departments_id` |
| **التدقيق** | `created_by`، `updated_by`، `created_at`، `updated_at` (للقراءة فقط) |

**مميزات إضافية:**
- استخدام `dependsOn` لربط القوائم المنسدلة (مثلاً `owner_id` يعتمد على `owner_type`).
- استخدام `trigger` لإظهار/إخفاء حقول `published_at` و `unpublished_at` بناءً على حالة `is_published`.
- تنسيق متجاوب باستخدام `span` و `cssClass`.

#### الفوائد

- واجهة خلفية معربة بالكامل وجاهزة للاستخدام.
- تنظيم منطقي للحقول يسهل عملية إدخال البيانات.
- تجربة مستخدم سلسة مع القوائم المنسدلة المترابطة والحقول المشروطة.

---

### الإصدار 1.0.6 – تحسينات نهائية وتجهيز للإطلاق

#### أهداف الإصدار

- إجراء تحسينات نهائية على الكود والهيكل.
- إعداد ملفات `composer.json` و `README.md` للتوثيق والتوزيع.
- توحيد المخرجات والتأكد من جاهزية الإضافة للاستخدام الإنتاجي.

#### الميزات الجديدة

##### 1. تحسينات على `Manager` و `DocumentType`

- تحسين دالة `createDocument` لتشمل معالجة الملفات (`files`) وبيانات الحقول (`fields`) بشكل منفصل.
- إضافة دالة `prepareDocumentOptions` لتجهيز القيم الافتراضية للشركة والفرع.
- تحسين معالجة الأخطاء وإضافة تفاصيل التصحيح (`debug`) في بيئة التطوير.

##### 2. ملف `composer.json`

تم إنشاء ملف `composer.json` بالإعدادات التالية:
- **الاسم**: `nano3/kyc`
- **النوع**: `nano-extension` (خاص ببرمجيات نانوسوفت)
- **الاعتماديات**: `php >= 8.0`، `nano/api`، `tss/basic`
- **الترخيص**: `proprietary`

##### 3. ملف `README.md`

تم إعداد ملف `README.md` احترافي يحتوي على:
- وصف عام للإضافة ومميزاتها.
- قائمة أنواع الوثائق المدعومة.
- متطلبات النظام والتثبيت.
- أمثلة استخدام لكل من API وكلاسات `Manager` و `DocumentType`.
- شرح هيكل قاعدة البيانات.
- معلومات الترخيص والدعم.

##### 4. توثيق التحديثات

تم إعداد ملف التوثيق هذا (`Update-Nano3-Kyc-ar.md`) ليغطي جميع الإصدارات من 1.0.1 إلى 1.0.6 بتنسيق احترافي متوافق مع معايير توثيق برمجيات نانوسوفت.

#### الفوائد

- جاهزية الإضافة للتوزيع والاستخدام في المشاريع الإنتاجية.
- توثيق شامل يسهل على المطورين فهم الإضافة واستخدامها.
- هيكل متكامل يدعم الصيانة والتطوير المستقبلي.

---

### ملخص الإصدارات (1.0.1 – 1.0.6)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.0.1 | إنشاء جدول `nano3_kyc_documents` والهيكل الأساسي للإضافة. |
| 1.0.2 | تطوير كلاس `DocumentType` وموديل `Document` مع السمات الأساسية. |
| 1.0.3 | تطوير كلاس `Manager` (مدير العمليات) مع دعم المعاملات والأحداث والاختبارات. |
| 1.0.4 | تطوير متحكم API ومتحكم الواجهة الخلفية مع ملفات التكوين. |
| 1.0.5 | تحسين ملفات الترجمة والقوائم والنماذج (`lang.php`، `columns.yaml`، `fields.yaml`). |
| 1.0.6 | تحسينات نهائية، إعداد `composer.json` و `README.md`، وتوثيق التحديثات. |

---

### متطلبات الترقية

1. **تشغيل الهجرات**:
   ```bash
   php artisan nano:up
   ```
   أو تنفيذ هجرة `create_table_nano3_kyc_documents.php` يدوياً.

2. **نشر ملف الإعدادات** (اختياري):
   ```bash
   php artisan config:publish nano3.kyc
   ```
   لتخصيص خصائص أنواع الوثائق عبر `config/nano3/kyc/document_types.php`.

3. **إعداد الصلاحيات**:
   - منح الصلاحيات المناسبة للأدوار (`documents.access`، `documents.verify`، إلخ) من لوحة التحكم.

4. **تحديث تعريفات API** (إذا كنت تستخدم `Nano.API`):
   - تأكد من أن مسارات API مضمنة في ملف التوجيه الرئيسي.

---

### الخاتمة

تم تطوير إضافة `Nano3.Kyc` لتكون حلاً متكاملاً ومرناً لإدارة عمليات التحقق من الهوية ضمن برمجيات نانوسوفت. بفضل هيكلها القابل للتوسع، ودعمها لأكثر من 28 نوع وثيقة، وواجهة API المتكاملة، وواجهة الإدارة السهلة، يمكن للمطورين دمجها بسرعة في أي مشروع يتطلب التحقق من هوية المستخدمين أو الشركات.

نتطلع إلى مواصلة تطوير الإضافة بناءً على ملاحظات المستخدمين والمتطلبات المتطورة، مع خطط مستقبلية تشمل:
- دعم التعرف الضوئي على الحروف (OCR) لاستخراج البيانات تلقائياً.
- تكامل مع خدمات التحقق من الوثائق الخارجية.
- لوحات تحكم تحليلية متقدمة.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)
