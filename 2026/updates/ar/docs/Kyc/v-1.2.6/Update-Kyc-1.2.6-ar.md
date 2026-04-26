## 2026-04-25 - 2026-04-26

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.6**

### ملخص التحديثات

بعد تعزيز أمان وثائق KYC في الإصدار 1.2.5، نقدم في الإصدار 1.2.6 تحسينات شاملة على **واجهة برمجة التطبيقات (API)** لجلب بيانات التصنيفات وأنواع الوثائق. تهدف هذه التحديثات إلى توفير بيانات منظمة ومرنة تناسب تطبيقات الواجهة الأمامية (Frontend)، مع دعم الترجمة وخيارات تضمين متقدمة.

يركز هذا الإصدار على:

- **نقاط نهاية جديدة** لجلب التصنيفات (`document-categories-list`) والأنواع (`document-types-list`).
- **دوال متقدمة في `DocumentType`** لإرجاع بيانات مهيكلة تدعم الترجمة وتضمين العلاقات (الأنواع، التفاصيل، الحقول، قواعد التحقق).
- **تحسينات في معالجة الأخطاء** داخل `FileUploadDocuments` لإرجاع معلومات تصحيحية مفصلة.
- **توحيد مخرجات API** باستخدام Fractal في عمليات التحقق (verify, reject, restore).

هذه التحسينات تجعل تطوير واجهات المستخدم الخاصة بـ KYC أكثر سهولة وفعالية.

---

### الإصدار 1.2.6 – نقاط نهاية جديدة للتصنيفات والأنواع وتحسينات API

#### أهداف الإصدار

- توفير نقاط نهاية API احترافية لجلب تصنيفات وأنواع وثائق KYC ببيانات منظمة.
- دعم خيارات تضمين مرنة (`include_types`, `include_details`, `include_fields`, `include_rules`, `include_category`).
- تحسين دوال `DocumentType` لتوليد استجابات متوافقة مع معايير تطبيقات الواجهة الأمامية.
- توحيد استجابات API لعمليات (verify, reject, restore) باستخدام Fractal Transformer.
- تحسين تقارير الأخطاء في رفع الملفات لتسهيل عملية التصحيح.

#### الميزات الجديدة

##### 1. نقاط نهاية جديدة للتصنيفات والأنواع

تم إضافة مسارين جديدين إلى `routes.php`:

| المسار | الوصف |
| :--- | :--- |
| `GET /api/v1/kyc/document-categories-list` | جلب قائمة تصنيفات الوثائق مع خيارات تضمين متقدمة. |
| `GET /api/v1/kyc/document-categories-list/{id}` | جلب تفاصيل تصنيف محدد مع أنواعه. |
| `GET /api/v1/kyc/document-types-list` | جلب قائمة أنواع الوثائق مع خيارات تضمين متقدمة. |
| `GET /api/v1/kyc/document-types-list/{id}` | جلب تفاصيل نوع وثيقة محدد. |

##### 2. دوال جديدة ومُحسَّنة في `DocumentType`

| الدالة | الوصف |
| :--- | :--- |
| `isValidCategory($category)` | التحقق من صحة كود تصنيف. |
| `getCategoriesList($locale, $options)` | جلب قائمة التصنيفات مع دعم تضمين الأنواع والتفاصيل. |
| `getCategoryItem($code, $locale, $options)` | جلب عنصر تصنيف واحد مع خيار تضمين أنواعه. |
| `getDocumentTypeList($locale, $options)` | جلب قائمة أنواع الوثائق مع دعم الفلترة حسب التصنيف وخيارات تضمين متقدمة. |
| `getDocumentTypeItem($code, $locale, $options)` | جلب عنصر نوع وثيقة واحد مع تفاصيله الكاملة. |

**خيارات التضمين المدعومة (`$options`):**

| الخيار | الوصف |
| :--- | :--- |
| `include_types` | تضمين أنواع الوثائق داخل كل تصنيف. |
| `include_details` | تضمين تفاصيل النوع (الوزن، الحقول المطلوبة، الملفات المسموحة...). |
| `include_fields` | تضمين مخطط الحقول (`fields`) الكامل للنوع. |
| `include_rules` | تضمين قواعد التحقق (`rules`) الخاصة بكل حقل. |
| `include_category` | تضمين كائن التصنيف الأب داخل عنصر النوع. |
| `category_filter` | فلترة الأنواع حسب تصنيف محدد (مثلاً `primary_id`). |

##### 3. تحسينات في معالجة الأخطاء لرفع الملفات

تم تحسين دوال `create` و `update` في `FileUploadDocuments` لتشمل:

- إرجاع تفاصيل أخطاء التحقق (`errors`) من `Manager`.
- تضمين معلومات التصحيح (`debug`) عند تفعيل `app.debug`.
- استخدام رسائل استثناء أكثر دقة.

##### 4. توحيد مخرجات API لعمليات التحقق

تم تعديل دوال `verify`, `reject`, `restore` في `Documents` API Controller لاستخدام `DocumentTransformer` عبر Fractal، مما يضمن تناسق شكل البيانات مع باقي نقاط النهاية.

---

### أمثلة عملية

#### 1. جلب قائمة التصنيفات فقط

**الطلب:**
```http
GET /api/v1/kyc/document-categories-list HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب بيانات التصنيفات بنجاح",
    "data": [
        {
            "code": "primary_id",
            "name_ar": "هوية أساسية",
            "name_en": "Primary ID",
            "name": "هوية أساسية"
        },
        ...
    ]
}
```

#### 2. جلب تصنيف محدد مع أنواعه وتفاصيلها

**الطلب:**
```http
GET /api/v1/kyc/document-categories-list/primary_id?include_types=1&include_details=1 HTTP/1.1
```

**الاستجابة (مقتطف):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "code": "primary_id",
        "name": "هوية أساسية",
        "types": [
            {
                "code": "passport",
                "name": "جواز سفر",
                "details": {
                    "verification_weight": 100,
                    "has_expiry": true,
                    "required_fields": ["document_front", "document_number", ...]
                }
            }
        ]
    }
}
```

#### 3. جلب جميع أنواع الوثائق مع التصنيف الأب والحقول والقواعد

**الطلب:**
```http
GET /api/v1/kyc/document-types-list?include_category=1&include_fields=1&include_rules=1 HTTP/1.1
```

#### 4. جلب تفاصيل نوع وثيقة محدد (جواز السفر) مع كل البيانات

**الطلب:**
```http
GET /api/v1/kyc/document-types-list/passport?include_category=1&include_details=1&include_fields=1&include_rules=1 HTTP/1.1
```

---

### ملخص الإصدارات (1.2.5 – 1.2.6)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.2.5 | نظام حماية متكامل للوثائق المعتمدة، منع التكرار، تحسينات عملية الاعتماد، تكامل مع صلاحيات Backend. |
| 1.2.6 | نقاط نهاية جديدة للتصنيفات والأنواع (`document-categories-list`, `document-types-list`)، دوال `DocumentType` متقدمة، تحسين معالجة الأخطاء، توحيد مخرجات API. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `classes/DocumentType.php` بالنسخة الجديدة.
   - استبدال ملف `apicontrollers/Documents.php` و `apicontrollers/documents/FileUploadDocuments.php`.
   - تحديث ملف `routes.php` لإضافة المسارات الجديدة.

2. **تشغيل هجرات قاعدة البيانات** (لا توجد تغييرات في هذا الإصدار).

3. **اختبار نقاط النهاية الجديدة**:
   - جرب `GET /api/v1/kyc/document-categories-list` مع خيارات تضمين مختلفة.
   - جرب `GET /api/v1/kyc/document-types-list/passport?include_fields=1`.

4. **مراجعة التوثيق**:
   - تم تحديث ملفات التوثيق `docs.md` الخاصة بنقاط النهاية الجديدة مع أمثلة كاملة.

---

### الخاتمة

مع الإصدار 1.2.6، نكون قد أكملنا بناء واجهة API احترافية ومرنة لنظام KYC، حيث يمكن للمطورين الآن جلب بيانات التصنيفات والأنواع بسهولة مع تحكم كامل في البيانات المضمنة. هذا يسهل بناء نماذج ديناميكية وواجهات مستخدم غنية دون الحاجة إلى طلبات متعددة أو معالجة يدوية للبيانات.

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
