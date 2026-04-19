## 2026-04-16 – 2026-04-18

**تحديثات إضافة `Nano.FileUpload` – الإصداران 1.2.4 و 1.2.5**

### ملخص التحديثات

شهدت الإضافة تطورين مهمين في الإصدارين **1.2.4** و **1.2.5**:

- **الإصدار 1.2.4** أضاف ميزة أمنية جديدة على مستوى الحقل: `allow_upload_only_when_model_exists`، والتي تمنع رفع الملفات إلى حقل معين إلا بعد أن يتم حفظ النموذج الأصلي (وجود `model_id` صالح). هذه الميزة مفيدة بشكل خاص للحقول المتعددة (`multiple`) مثل معارض الصور التي لا ينبغي رفع صورها قبل إنشاء السجل الرئيسي.

- **الإصدار 1.2.5** قام بإعادة هيكلة شاملة لنقاط نهاية API الخاصة بالاستعلام عن الموديولات والصلاحيات، وذلك بنقل معاملات `model_class` و `field` من المسار (route parameters) إلى معاملات الاستعلام (query parameters). هذا التغيير يحل مشكلة ترميز أسماء الموديولات التي تحتوي على خطوط مائلة عكسية (`\`) ويجعل المسارات أكثر نظافة وأماناً.

---

## الإصدار 1.2.4 – منع رفع الملفات قبل حفظ النموذج

### أهداف الإصدار

- **إضافة طبقة أمان إضافية** تمنع رفع الملفات إلى الحقول التي تتطلب وجود النموذج أولاً (مثل معارض الصور، المستندات المرتبطة بسجل محفوظ).
- **تحسين تجربة المطور** بتوفير خيار `allow_upload_only_when_model_exists` في تعريف الحقل، مما يسمح بمنع الرفع المؤقت (باستخدام `temp_session_key`) لهذا الحقل.
- **تجنب تراكم الملفات اليتيمة** (غير المرتبطة) في النظام.

### الميزات الجديدة

#### 1. إضافة خيار `allow_upload_only_when_model_exists` في تعريف الحقل

أصبح من الممكن الآن تعيين هذا الخيار في تعريف أي حقل (خاصة الحقول من نوع `multiple` أو `image` التي تتطلب وجود النموذج):

```php
'fields' => [
    'gallery' => [
        'type' => 'multiple',
        'max_filesize' => 2048,
        'allowed_types' => 'jpg,jpeg,png',
        'max_files' => 10,
        'allow_upload_only_when_model_exists' => true, // ⬅️ جديد
    ],
],
```

- **القيمة الافتراضية:** `false` (يسمح بالرفع المؤقت حتى للنماذج غير المحفوظة).
- **عند تفعيله (`true`)**: يمنع رفع الملفات لهذا الحقل إذا لم يتم توفير `model_id` صالح (أي النموذج غير موجود في قاعدة البيانات). سيتم رفض الطلب مع رسالة خطأ `FILE_UPLOAD_PERMISSION_DENIED` ورسالة `model_must_exist_before_upload`.

#### 2. التحقق في `FileUploadService::upload()`

أضفنا في دالة `upload()` منطقاً للتحقق من هذا الخيار:

- يتم التحقق من وجود النموذج (`model_exists`) إما من خلال `options['model']` (إذا كان موجوداً ومحفوظاً) أو عن طريق البحث عن `model_id` وجلب النموذج.
- إذا كان الخيار مفعلاً ولم يكن النموذج موجوداً، يتم رمي استثناء `FileUploadException` مع كود الخطأ المناسب.

#### 3. رسالة ترجمة جديدة

أضفنا مفتاح ترجمة `errors.model_must_exist_before_upload`:

```php
'model_must_exist_before_upload' => 'لا يمكن رفع الملفات لحقل ":field" إلا بعد حفظ ":model". يرجى حفظ :model أولاً ثم المحاولة مرة أخرى.',
```

### الفوائد

- **منع الملفات اليتيمة**: تضمن عدم وجود ملفات غير مرتبطة بأي سجل في قاعدة البيانات.
- **تحكم دقيق**: يمكن تفعيل الميزة على حقول محددة فقط (مثل المعارض) مع ترك الحقول الأخرى (مثل الصورة الرمزية) تسمح بالرفع المؤقت.
- **أمان محسن**: يمنع المستخدمين من رفع ملفات لحقول تتطلب وجود السجل الأصلي قبل إنشائه.

---

## الإصدار 1.2.5 – إعادة هيكلة مسارات API (Query Parameters بدلاً من Route Parameters)

### أهداف الإصدار

- **حل مشكلة ترميز أسماء الموديولات** التي تحتوي على خطوط مائلة عكسية (`\`) في المسارات (مثل `Nano\Shop\Models\Product`). كان تمريرها في المسار يتطلب ترميزاً (`Nano%5CShop%5CModels%5CProduct`) مما يجعل المسارات غير عملية وغير قابلة للقراءة.
- **تبسيط نقاط النهاية** باستخدام معاملات الاستعلام (`?model_class=...`) بدلاً من معاملات المسار.
- **الحفاظ على التوافق العكسي** عبر إبقاء المسارات القديمة (مع إهمالها) مع إتاحة المسارات الجديدة.

### الميزات الجديدة

#### 1. نقاط النهاية الجديدة (باستخدام query parameters)

| الوظيفة | المسار القديم (مهمل) | المسار الجديد |
|---------|---------------------|----------------|
| جلب إعدادات موديول | `GET /models/{modelClass}` | `GET /model/config?model_class=...` |
| جلب إعدادات حقل | `GET /models/{modelClass}/fields/{field}` | `GET /field/config?model_class=...&field=...` |
| جلب قيود الحقل | `GET /models/{modelClass}/fields/{field}/constraints` | `GET /field/constraints?model_class=...&field=...` |
| جلب خيارات المعالجة | `GET /processing-options/{modelClass}/{field}` | `GET /processing-options?model_class=...&field=...` |
| التحقق من صلاحية موديول | `GET /permissions/model/{modelClass}/{operation}` | `GET /permissions/model?model_class=...&operation=...` |
| التحقق من صلاحية حقل | `GET /permissions/field/{modelClass}/{field}/{operation}` | `GET /permissions/field?model_class=...&field=...&operation=...` |

**ملاحظة:** نقاط النهاية `GET /models` و `GET /permissions/global/{operation}` و `POST /permissions/check` و `POST /temp-key/validate` لم تتغير.

#### 2. تحديث دوال المتحكم `FileUploadController`

تم تعديل الدوال التالية لقراءة المعاملات من `Input::get()` (أو معاملات الاستعلام) بدلاً من معاملات المسار، مع دعم تمرير المعاملات عبر الطريقة القديمة كاحتياطي (للتوافق العكسي):

- `getModelConfig($modelClass = null)`
- `getFieldConfig($modelClass = null, $field = null)`
- `getFieldConstraints($modelClass = null, $field = null)`
- `getProcessingOptions($modelClass = null, $field = null)`
- `checkModelPermission($modelClass = null, $operation = null)`
- `checkFieldPermission($modelClass = null, $field = null, $operation = null)`

**مثال على الطريقة الجديدة للدالة `getModelConfig`:**

```php
public function getModelConfig($modelClass = null)
{
    $modelClass = $modelClass ?? Input::get('model_class');
    if (!$modelClass) {
        return $this->errorResponse('Missing model_class parameter', null, 400);
    }
    // ... باقي المنطق
}
```

#### 3. تحديث ملف `routes.php`

تم تعليق المسارات القديمة (باستخدام `/* ... */`) وإضافة المسارات الجديدة:

```php
// المسارات الجديدة
Route::get('model/config', [ 'as' => 'model.config', 'uses' => 'FileUploadController@getModelConfig' ]);
Route::get('field/config', [ 'as' => 'field.config', 'uses' => 'FileUploadController@getFieldConfig' ]);
Route::get('field/constraints', [ 'as' => 'field.constraints', 'uses' => 'FileUploadController@getFieldConstraints' ]);
Route::get('processing-options', [ 'as' => 'processing.options', 'uses' => 'FileUploadController@getProcessingOptions' ]);
Route::get('permissions/model', [ 'as' => 'permissions.model', 'uses' => 'FileUploadController@checkModelPermission' ]);
Route::get('permissions/field', [ 'as' => 'permissions.field', 'uses' => 'FileUploadController@checkFieldPermission' ]);
```

#### 4. أمثلة على الطلبات الجديدة

**جلب إعدادات موديول `Product`:**

```http
GET /api/v1/fileupload/model/config?model_class=Nano\Shop\Models\Product HTTP/1.1
Authorization: Bearer ...
```

**جلب قيود حقل `image`:**

```http
GET /api/v1/fileupload/field/constraints?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

**التحقق من صلاحية `edit` على حقل `image`:**

```http
GET /api/v1/fileupload/permissions/field?model_class=Nano\Shop\Models\Product&field=image&operation=edit HTTP/1.1
Authorization: Bearer ...
```

### الفوائد

- **مسارات نظيفة وقابلة للقراءة** – لا حاجة لترميز أسماء الموديولات.
- **سهولة الاستخدام من أي عميل HTTP** – تمرير `model_class` كقيمة عادية في query string.
- **توافق عكسي** – المسارات القديمة ما زالت تعمل (لكنها مهملة) وستتم إزالتها في إصدار مستقبلي.
- **أمان إضافي** – تجنب التعامل مع إدخال المستخدم كجزء من المسار (يقلل من مخاطر التوجيه الضار).

---

## متطلبات الترقية (من 1.2.3 إلى 1.2.5)

1. **تحديث الكود**:
   - استبدال `FileUploadController.php` بالنسخة التي تحتوي على الدوال المحدثة (تدعم query parameters).
   - استبدال `routes.php` بالنسخة التي تحتوي على المسارات الجديدة.
   - استبدال `FileUploadRegistry.php` بالنسخة التي تحتوي على خيار `allow_upload_only_when_model_exists` (تم تضمينه في الإصدار 1.2.4).
   - استبدال `FileUploadService.php` بالنسخة التي تطبق التحقق من الخيار الجديد.

2. **تحديث تعريفات النماذج** (اختياري):
   - إذا كنت ترغب في استخدام `allow_upload_only_when_model_exists`، أضفه إلى تعريفات الحقول المطلوبة.

3. **لا توجد هجرات جديدة** – لا تغييرات في قاعدة البيانات.

4. **لا توجد تغييرات في الإعدادات** – يظل ملف `config.php` كما هو.

5. **تحديث واجهات المستخدم (API clients)**:
   - إذا كنت تستخدم نقاط النهاية القديمة (ذات الأقواس المتعرجة)، يُنصح بالانتقال إلى المسارات الجديدة فوراً. المسارات القديمة ما زالت تعمل حالياً ولكنها مهملة وستتم إزالتها في الإصدار 1.3.0.

6. **اختبار التوافق**:
   - تأكد من أن جميع نقاط النهاية الجديدة تعمل بشكل صحيح مع `model_class` كمعامل استعلام.
   - اختبر حقل مع `allow_upload_only_when_model_exists = true` للتأكد من رفض الرفع قبل حفظ النموذج.

---

## الخاتمة

يمثل الإصداران **1.2.4** و **1.2.5** خطوة مهمة نحو تحسين أمان النظام وسهولة استخدام واجهة API. فبينما أضاف الإصدار 1.2.4 تحكماً دقيقاً في متى يُسمح برفع الملفات (منع الرفع المؤقت للحقول التي تتطلب وجود النموذج)، قدم الإصدار 1.2.5 إعادة هيكلة شاملة لنقاط النهاية لتكون أكثر نظافة وأماناً. هذه التحديثات تجعل الإضافة أكثر احترافية وتسهل دمجها مع التطبيقات الخارجية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)