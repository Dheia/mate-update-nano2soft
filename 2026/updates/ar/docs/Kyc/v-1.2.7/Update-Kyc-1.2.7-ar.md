## 2026-04-27 - 2026-04-28

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.7**

### ملخص التحديثات

بعد إطلاق نقاط النهاية الجديدة للتصنيفات والأنواع في الإصدار 1.2.6، نقدم في الإصدار 1.2.7 تحسينًا جوهريًا على **بنية حقول الإدخال (Form Fields)** الخاصة بأنواع الوثائق. يهدف هذا الإصدار إلى توحيد آلية تعريف الحقول وعرضها والتحقق منها عبر دمج `FormFieldsManager` – النظام الموحد لإدارة حقول النماذج في منصة نانوسوفت.

يركز هذا الإصدار على:

- **إعادة هيكلة تعريفات الحقول** في `DocumentType` لتتوافق مع معايير `FormFieldsManager` (مصفوفة متسلسلة بدلاً من الترابطية).
- **دوال تحويل ذكية** تضمن التوافق الكامل مع الإصدارات السابقة (Legacy Support) دون كسر أي وظائف موجودة.
- **تحسين أداء ودقة قواعد التحقق** عبر تفويض المهمة إلى `FormFieldsManager`.
- **إثراء استجابات API** بخيارات الحقول المحلولة (Resolved Options) للقوائم المنسدلة.

هذه التحسينات تجعل نظام KYC أكثر تكاملاً مع باقي مكونات المنصة، وتفتح الباب لاستخدام أدوات موحدة لبناء النماذج الديناميكية في لوحة التحكم وواجهات API.

---

### الإصدار 1.2.7 – توافق كامل مع `FormFieldsManager` وإعادة هيكلة حقول الوثائق

#### أهداف الإصدار

- تحويل جميع تعريفات الحقول في `DocumentType` وملف الإعدادات `document_types.php` إلى تنسيق `FormFieldsManager` (مصفوفة متسلسلة).
- توفير دوال تحويل تلقائية (`convertLegacyFieldsToNewFormat`، `convertLegacyValidation`، إلخ) لضمان عدم تعطل الأنظمة القائمة.
- تفويض عمليات التحقق من صحة الحقول إلى `FormFieldsManager` لتوحيد المنطق.
- تحسين نقطة النهاية `GET /document-fields/{type}` لإرجاع الحقول مع الخيارات المحلولة (مثلاً: قائمة الدول من API).
- إصلاح مشكلة تحميل ملف الإعدادات بسبب النقطة الزائدة في `CONFIG_KEY`.

#### الميزات الجديدة

##### 1. تحويل تعريفات الحقول إلى تنسيق `FormFieldsManager`

تم تغيير بنية الحقول في ملف `config/nano3/kyc/document_types.php` من:

```php
// القديم (مصفوفة ترابطية)
'fields' => [
    'document_number' => [
        'label' => 'رقم الوثيقة',
        'type' => 'text',
        'required' => true,
        'validation' => ['max_length' => 50],
    ],
]
```

إلى التنسيق الجديد المتسلسل المدعوم من `FormFieldsManager`:

```php
'fields' => [
    [
        'id' => 'document_number',
        'label' => ['ar' => 'رقم الوثيقة', 'en' => 'Document Number'],
        'type' => 'text',
        'required' => true,
        'validation' => [
            ['rule' => 'required'],
            ['rule' => 'max', 'value' => 50],
        ],
    ],
]
```

هذا التغيير يجعل تعريفات الحقول متوافقة مع أدوات إدارة النماذج الموحدة في المنصة.

##### 2. دوال تحويل ذكية للتوافق مع الإصدارات السابقة (Legacy Support)

لضمان عدم تأثر الأنظمة التي ما زالت تعتمد على التنسيق القديم، تمت إضافة دوال تحويل تلقائية في `DocumentType`:

| الدالة | الوصف |
| :--- | :--- |
| `convertLegacyFieldsToNewFormat($legacyFields)` | تحول مصفوفة الحقول الترابطية القديمة إلى المصفوفة المتسلسلة الجديدة. |
| `normalizeMultilingualValue($value)` | تحول أي نص إلى مصفوفة متعددة اللغات (`['ar' => ..., 'en' => ...]`). |
| `convertOptionsToDataSource($optionType)` | تحول الخيارات النصية القديمة (مثل `'countries'`) إلى تعريف `data_source` متوافق مع `FormFieldsManager`. |
| `convertLegacyValidation($legacyValidation)` | تحول قواعد التحقق من التنسيق الترابطي إلى تنسيق `FormFieldsManager` (مصفوفة من `['rule' => ..., 'value' => ...]`). |

بفضل هذه الدوال، يمكن لـ `DocumentType` قراءة كلا التنسيقين (القديم والجديد) وإرجاع الحقول دائمًا بالتنسيق الموحد.

##### 3. دوال مُحسَّنة وجديدة في `DocumentType`

| الدالة | الوصف |
| :--- | :--- |
| `getFieldsSchema($type)` | **مُحسَّنة**: تكتشف تنسيق الحقول تلقائيًا (قديم/جديد) وتعيدها دائمًا بالتنسيق المتسلسل بعد التطبيع عبر `FormFieldsManager`. |
| `getFieldDefinition($type, $field)` | **مُحسَّنة**: تبحث في المصفوفة المتسلسلة عن الحقل باستخدام المفتاح `id`. |
| `getValidationRules($type)` | **مُحسَّنة**: تستخدم `FormFieldsManager` لتوليد قواعد التحقق بدلاً من المنطق المخصص. |
| `getDefaultFields()` | **مُعاد كتابتها**: ترجع الحقول الافتراضية بالتنسيق المتسلسل الجديد. |
| `buildFieldsForType($customFields, $overrideDefaults)` | **مُعاد كتابتها**: تتعامل مع المصفوفات المتسلسلة وتطبق التجاوزات بناءً على `id`. |
| `mapInputToDocumentData($type, $input)` | **مُحسَّنة**: تستخدم `id` من تعريف الحقل لاستخراج البيانات. |
| `mapDocumentToOutput($type, $document)` | **مُحسَّنة**: تعيد بناء المخرجات باستخدام `id` الحقول. |
| `getFieldLabel($type, $field, $locale)` | **مُحسَّنة**: تستخرج النص المطلوب من مصفوفة `label` متعددة اللغات. |
| `getFormFieldsManager()` | **جديدة**: تنشئ نسخة من `FormFieldsManager` مع اللغات المدعومة (`ar`, `en`). |

##### 4. تحسين نقطة نهاية `GET /document-fields/{type}`

تم تحديث الدالة `Documents::getDocumentFields` لتقوم بما يلي:

- جلب الحقول المطبعة من `DocumentType::getFieldsSchema`.
- إثراء الحقول بالخيارات المحلولة (`resolved_options`) باستخدام `FormFieldsManager::enrichWithOptions`.
- إرجاع البيانات بتنسيق موحد يدعمه `FormFieldsManager`.

هذا يسمح لتطبيقات الواجهة الأمامية بالحصول على قوائم منسدلة جاهزة (مثل قائمة الدول والجنسيات) مباشرة ضمن تعريف الحقل.

##### 5. إصلاح تحميل ملف الإعدادات

تم تصحيح دالة `loadConfig` لإزالة النقطة الزائدة من `CONFIG_KEY` (`nano3.kyc::document_types.` ← `nano3.kyc::document_types`) مما يضمن تحميل ملف `document_types.php` بشكل صحيح.

---

### أمثلة عملية

#### 1. الحصول على حقول جواز السفر مع الخيارات المحلولة

**الطلب:**
```http
GET /api/v1/kyc/document-fields/passport HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مقتطف):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "type": "passport",
        "type_name": "جواز سفر",
        "fields": [
            {
                "id": "document_front",
                "type": "fileupload",
                "label": { "ar": "وجه الوثيقة", "en": "Document Front" },
                "required": true,
                "validation": [
                    { "rule": "required" },
                    { "rule": "mimes", "value": "jpeg,jpg,png,pdf" }
                ]
            },
            {
                "id": "country_of_issue",
                "type": "select",
                "label": { "ar": "بلد الإصدار", "en": "Country of Issue" },
                "data_source": {
                    "type": "api",
                    "endpoint": "api/v1/location/countries"
                },
                "resolved_options": [
                    { "id": "YE", "name": "اليمن" },
                    { "id": "SA", "name": "السعودية" },
                    ...
                ]
            }
        ]
    }
}
```

#### 2. استخدام التنسيق القديم (يُحول تلقائيًا)

إذا كان لديك تعريف حقل قديم (مصفوفة ترابطية) في إصدار سابق، فإن `DocumentType` سيحوله تلقائيًا إلى التنسيق الجديد دون أي تدخل من المطور.

```php
// تعريف قديم
$oldFields = [
    'document_number' => ['type' => 'text', 'required' => true]
];

// يستخدم داخليًا convertLegacyFieldsToNewFormat()
$newFields = DocumentType::getFieldsSchema('some_type');
```

---

### ملخص الإصدارات (1.2.6 – 1.2.7)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.2.6 | نقاط نهاية جديدة للتصنيفات والأنواع، دوال `DocumentType` متقدمة، تحسين معالجة الأخطاء، توحيد مخرجات API. |
| 1.2.7 | إعادة هيكلة حقول الوثائق لتتوافق مع `FormFieldsManager`، دوال تحويل تلقائية للتوافق مع الإصدارات السابقة، تحسين قواعد التحقق، إثراء حقول API بالخيارات المحلولة، إصلاح تحميل الإعدادات. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `classes/DocumentType.php` بالنسخة الجديدة.
   - استبدال ملف `apicontrollers/Documents.php` بالنسخة الجديدة.
   - تحديث ملف `config/nano3/kyc/document_types.php` إلى التنسيق الجديد (اختياري لكن موصى به للاستفادة الكاملة من الميزات).

2. **تشغيل هجرات قاعدة البيانات** (لا توجد تغييرات في هذا الإصدار).

3. **اختبار التوافق**:
   - تأكد من أن نقاط النهاية الحالية (`GET /document-fields/{type}`) تعيد البيانات بالتنسيق الجديد.
   - تحقق من أن عمليات إنشاء وتحديث الوثائق ما زالت تعمل بشكل صحيح (حيث تستخدم `mapInputToDocumentData`).

4. **مسح الكاش** (اختياري):
   - بعد تحديث ملف الإعدادات، قد تحتاج إلى مسح كاش التطبيق: `php artisan cache:clear`

---

### الخاتمة

مع الإصدار 1.2.7، نكون قد وحَّدنا آلية إدارة حقول KYC مع نظام `FormFieldsManager` المعتمد في باقي أجزاء المنصة. هذا يفتح آفاقًا جديدة مثل بناء نماذج ديناميكية في لوحة التحكم، وتوليد قواعد تحقق موحدة، واستخدام مصادر بيانات خارجية بسهولة. كما أن دوال التحويل التلقائية تضمن سلاسة الترقية دون أي تأثير سلبي على الأنظمة القائمة.

نتطلع إلى ملاحظاتكم ومقترحاتكم لمواصلة تطوير هذه الإضافة الحيوية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)
