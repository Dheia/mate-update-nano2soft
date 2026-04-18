# توثيق عام: إدارة رفع الملفات في تطبيقات نانوسوفت (Nano.FileUpload)

**نبذة تعريفية عن حزمة `Nano.FileUpload` ووحدة إدارة رفع الملفات**

هذه المجموعة من الكلاسات (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager` وغيرها) هي جزء من **حزمة `Nano.FileUpload`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **نانوسوفت** بهدف توحيد وإدارة عمليات رفع الملفات عبر واجهات برمجة التطبيقات (API) بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano\FileUpload\Classes
```

توفر هذه الوحدة أدوات متقدمة لتسجيل النماذج (الموديولات) التي تحتوي على حقول رفع ملفات، والتحقق من صلاحيات المستخدمين (بما في ذلك أنواع المستخدمين المختلفة: `backend` و `frontend`)، وإدارة الملفات المرفوعة مؤقتًا قبل حفظ السجل، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة رفع ملفات قابلة للتوسع بسهولة وتلبية احتياجات التطبيقات المتقدمة دون المساس بأمان التطبيق أو أدائه.

---

## 1. مقدمة

**وحدة `Nano.FileUpload`** هي إضافة متخصصة مقدمة من **شركة نانوسوفت (NanoSoft)** لتطبيقات نانوسوفت و Laravel. تهدف هذه الوحدة إلى توفير **نظام متكامل لإدارة رفع الملفات** عبر واجهات برمجة التطبيقات (API) بطريقة آمنة ومرنة، مع دعم أنواع المستخدمين المختلفة (backend / frontend) وصلاحيات دقيقة على مستوى كل حقل وعملية.

#### لماذا نحتاج إلى نظام مركزي لرفع الملفات؟
في التطبيقات الإدارية المعقدة، تحتاج العديد من الإضافات (مثل المنتجات، المستخدمين، الطلبات) إلى رفع ملفات (صور، مستندات، إلخ). بدلاً من إعادة كتابة منطق الرفع والتحقق من الصلاحيات في كل إضافة، يوفر هذا النظام **طبقة مركزية** يمكن لأي إضافة التسجيل فيها بسهولة، مما يضمن:
- توحيد عملية الرفع عبر التطبيق.
- إدارة الصلاحيات بأنواع المستخدمين المختلفة.
- إمكانية رفع الملفات قبل حفظ النموذج (باستخدام مفاتيح جلسة مؤقتة).
- دعم استبدال الملفات (عملية `edit`) في العلاقات الأحادية مع الحفاظ على التكاملية باستخدام المعاملات.
- معالجة موحدة للأخطاء.
- توفير واجهة API جاهزة للاستخدام.

---

## 2. المكونات الرئيسية للنظام

يتكون النظام من مجموعة من الكلاسات المتخصصة، يمكن تقسيمها إلى عدة طبقات:

### أ. طبقة تسجيل النماذج (Registry Layer)
- **`FileUploadRegistry`**: السجل المركزي الذي يحتوي على تعريفات النماذج وحقول الرفع. يوفر دوال لتسجيل النماذج، الاستعلام عن الإعدادات، والتحقق من الصلاحيات وأنواع المستخدمين.
- **الخصائص الرئيسية**:
  - تسجيل النموذج مع `allowed_user_types` (backend/frontend).
  - تعيين صلاحيات لكل عملية (`add`, `edit`, `delete`, `view`) على مستوى النموذج أو الحقل.
  - **إضافة `disabled_operations`** (منذ الإصدار 1.2.1): تعطيل عمليات محددة على مستوى النموذج أو الحقل دون تعديل الصلاحيات.
  - **دوال التحقق المتكاملة** (منذ الإصدار 1.2.1): `canGlobal`, `validateGlobal`, `canModel`, `validateModel`, `canField`, `validateField`, `validate` – تسمح بالتحقق من الصلاحيات على مستويات متعددة ورمي استثناءات مناسبة.
  - دعم الإعدادات الافتراضية (`defaults`) لتقليل التكرار.
  - دالة `updateFieldConfig` لتحديث إعدادات حقل معين مع التطبيع ومسح الكاش.
  - دالة `updateModelConfig` محسنة لتطبيع النموذج بالكامل بعد التعديل.
  - إطلاق أحداث (`nano.api.fileupload.registerModels`, `nano.api.fileupload.modelRegistered`) لتوسيع النظام.
  - دعم التخزين المؤقت (Cache) لنتائج الاستعلامات.
  - إعدادات عامة للتحكم في عمليات API (`disable_upload`, `disable_delete`, `disable_get`, `disable_edit`).
  - إعدادات عامة للتحكم في التحويلات التلقائية (`disable_auto_resize`, `disable_auto_watermark`).

### ب. طبقة الخدمة (Service Layer)
- **`FileUploadService`**: المسؤول عن تنفيذ عمليات رفع الملفات وحذفها واسترجاعها. يعتمد على `FileUploadRegistry` للوصول إلى الإعدادات والتحقق من الصلاحيات، وعلى `FileUploadUserManager` لإدارة المستخدم الحالي.
- **الوظائف الرئيسية**:
  - رفع ملف واحد أو عدة ملفات (`upload`, `uploadMultiple`).
  - تحديد العملية تلقائياً (`add` أو `edit`) بناءً على وجود ملف مرتبط في علاقة `attachOne`.
  - استخدام `Db::transaction` لضمان تكاملية عملية الاستبدال (رفع الجديد، حذف القديم، ربط العلاقة).
  - حذف ملف مع التحقق من الصلاحية (`deleteFile`).
  - جلب الملفات المرتبطة بنموذج أو مؤقتة (`getFiles`).
  - توليد مفاتيح جلسة مؤقتة (`generateTempSessionKey`) مع توقيع HMAC-SHA256.
  - ربط الملفات المؤقتة بنموذج محفوظ (`attachTempFiles`) مع إرجاع استجابة موحدة.
  - **توحيد التحقق من المفاتيح المؤقتة** (منذ الإصدار 1.2.2): استخدام تريت `HasFileUploadsMatchTempKey` ودالة `validateAndMatchTempKey` للتحقق من صحة المفاتيح ومطابقتها مع النموذج والحقل والمستخدم في جميع الدوال (`upload`, `deleteFile`, `attachTempFiles`, `getFiles`).
  - التحقق من صحة الملف (حجم، نوع، قائمة سوداء) قبل الرفع (`validateFile`).
  - تطبيق التحويلات التلقائية (تحجيم، علامة مائية) على الصور (`applyAutoProcessing`).
  - تعيين قرص التخزين المخصص (`getStorageDiskForField`).
  - حساب SHA256 للمحتوى وتخزينه في `hash`.
  - تخزين أبعاد الصورة في `meta`.
  - تعيين تاريخ انتهاء الصلاحية للملفات المؤقتة (`expires_at`) وتعيين `session_key` للملفات المؤقتة.

### ج. طبقة إدارة المستخدمين والصلاحيات (User & Permission Layer)
- **`FileUploadUserManager`**: مدير المستخدمين والصلاحيات. يوفر واجهة موحدة للحصول على المستخدم الحالي ونوعه (`backend`/`frontend`/`guest`) والتحقق من صلاحياته.
- **الميزات**:
  - محلل افتراضي للمستخدم يدعم `BackendAuth`, `Auth`, `AuthHelpers`.
  - إمكانية تخصيص محلل المستخدم (`setUserResolver`).
  - إمكانية تخصيص دالة التحقق من الصلاحيات (`setPermissionChecker`).
  - دعم دوال `hasAccess` و `hasPermission` لمستخدمي backend، ودالة `canUpload` لمستخدمي frontend.

### د. طبقة واجهة برمجة التطبيقات (API Layer)
- **`FileUploadController`**: متحكم API يقدم نقاط نهاية RESTful لرفع الملفات (فردي/متعدد)، حذف ملف، جلب الملفات، بالإضافة إلى نقاط نهاية جديدة للاستعلام عن الموديولات والصلاحيات والتحقق من المفاتيح المؤقتة.
- **نقاط النهاية الأساسية**:
  - `POST /upload` – رفع ملف واحد.
  - `POST /upload-multiple` – رفع عدة ملفات.
  - `DELETE /delete/{id}` – حذف ملف.
  - `GET /files` – جلب الملفات المرتبطة.
  - `GET /tests` – تشغيل مجموعة الاختبارات (متاح فقط في بيئة التطوير).
- **نقاط النهاية الجديدة** (منذ الإصدار 1.2.3):
  - **الاستعلام عن الموديولات والحقول**:
    - `GET /models` – جلب قائمة الموديولات المسجلة (مرشحة حسب صلاحية المستخدم).
    - `GET /models/{modelClass}` – جلب إعدادات موديول معين.
    - `GET /models/{modelClass}/fields/{field}` – جلب إعدادات حقل معين.
    - `GET /models/{modelClass}/fields/{field}/constraints` – جلب قيود الحقل (الحجم، الأنواع، إلخ).
    - `GET /processing-options/{modelClass}/{field}` – جلب خيارات المعالجة المتقدمة (التخزين، التحجيم، العلامة المائية).
  - **التحقق من الصلاحيات**:
    - `GET /permissions/global/{operation}` – التحقق من تفعيل عملية عالمياً.
    - `GET /permissions/model/{modelClass}/{operation}` – التحقق من صلاحية عملية على مستوى موديول.
    - `GET /permissions/field/{modelClass}/{field}/{operation}` – التحقق من صلاحية عملية على مستوى حقل.
    - `POST /permissions/check` – التحقق المتكامل من الصلاحية باستخدام `validate()`.
  - **التحقق من المفاتيح المؤقتة**:
    - `POST /temp-key/validate` – التحقق من صحة مفتاح مؤقت ومطابقته باستخدام `validateAndMatchTempKey`.
- **المصادقة**: OAuth 2.0 عبر middleware `oauth-users`.
- **الاستجابات**: موحدة وفق الهيكل `{code, status, message, data, input_data, process_data, debug, error_code}`.

### هـ. طبقة معالجة الأخطاء والاستثناءات
- **`FileUploadException`**: كلاس استثناء مخصص يوفر أكواد خطأ فريدة (مثل `FILE_UPLOAD_FILE_SIZE_EXCEEDED`) وسياق إضافي للخطأ. يتم استخدامه في جميع أنحاء الخدمة لتوحيد التعامل مع الأخطاء.
- يقوم تلقائياً بتحويل كود الخطأ الداخلي إلى كود HTTP مناسب (401, 403, 404, 422, 500, 503, 423 إلخ) بناءً على خريطة محددة مسبقاً. **تم تعديل كود `ERR_PERMISSION_DENIED` من 403 إلى 422** (منذ الإصدار 1.2.1) لتجنب التعارض مع حالة "حساب غير نشط".

### و. طبقة الاختبارات (Testing Layer)
- **`FileUploadPlusTest`**: كلاس اختبار شامل يوفر تغطية كاملة لجميع كلاسات الإضافة.
- **الميزات**:
  - توحيد مخرجات الاختبارات بنفس هيكل API (`code`, `status`, `test_code`, `name`, `description`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`).
  - معالجة استثنائية متقدمة تمنع توقف الاختبارات الأخرى عند فشل اختبار واحد.
  - استخدام `Db::beginTransaction()` و `Db::rollBack()` في جميع الاختبارات التي تعدل قاعدة البيانات لضمان عدم ترك آثار.
  - اختبارات مخصصة لعملية التعديل (`testEditReplaceFile`, `testEditWithoutPermission`, `testEditWithGlobalDisable`, `testEditTriggersEvents`, `testEditKeepsOldFileOnFailure`).
  - دعم تشغيل الاختبارات عبر المتصفح باستخدام معامل `test_version=v1` أو `test_version=v2`.

---

## 3. كيف تعمل هذه الكلاسات معًا؟ (التدفق العام)

لنفترض أن إضافة `Nano.Shop` تريد رفع صورة رئيسية لمنتج جديد، ثم استبدالها لاحقاً. إليك كيف تعمل المكونات معًا:

1. **تسجيل النموذج (مرة واحدة)**:
   - في `Plugin.php` للإضافة، يتم تعريف دالة `registerFileUploadFields` التي تعيد مصفوفة تعريفات النماذج.
   - يقوم `FileUploadRegistry` تلقائيًا بجمع هذه التعريفات عبر `PluginManager` وتسجيلها، مع تطبيق الإعدادات الافتراضية (حسب نوع الملف) والتخزين المؤقت.
   - **منذ الإصدار 1.2.1** يمكن إضافة `disabled_operations` على مستوى النموذج أو الحقل لتعطيل عمليات محددة.

2. **رفع الملف الأول (عملية add)**:
   - يرسل العميل طلب `POST /upload` مع `model_class` و `field` والملف.
   - يتحقق `FileUploadController` من وجود النموذج والحقل في `FileUploadRegistry`.
   - يستدعي `FileUploadService::upload` مع البيانات.
   - يحدد `FileUploadService` العملية على أنها `add` (لعدم وجود ملف مرتبط).
   - **إذا تم تمرير `temp_session_key`، يتم التحقق من صحته ومطابقته باستخدام `validateAndMatchTempKey`** (منذ الإصدار 1.2.2).
   - يتحقق من الصلاحية عبر `validate($modelClass, 'add', ...)`.
   - يرفع الملف عبر `Base64::onUpload`، ويحسب `hash`, `meta`, ويضبط `expires_at` و `session_key` للملفات المؤقتة (إذا لم يكن هناك نموذج محفوظ).
   - يعيد `temp_session_key` في الاستجابة.

3. **ربط الملف بالنموذج بعد إنشاء المنتج**:
   - بعد حفظ المنتج، يستدعي التطبيق `FileUploadService::attachTempFiles` لربط الملف المؤقت بالنموذج.
   - **تستخدم `attachTempFiles` دالة `validateAndMatchTempKey`** للتحقق من المفتاح والمستخدم والنموذج والحقل في خطوة واحدة.

4. **استبدال الملف (عملية edit)**:
   - يرسل العميل طلب `POST /upload` مرة أخرى بنفس `model_class` و `field` ولكن مع `model_id` للمنتج الحالي.
   - يتحقق `FileUploadService` من وجود ملف مرتبط بالفعل في علاقة `attachOne`، فيحدد العملية على أنها `edit`.
   - يتحقق من صلاحية `edit` عبر `validate($modelClass, 'edit', ...)`.
   - يتم تنفيذ المعاملة (`Db::transaction`):
     - رفع الملف الجديد وحفظه.
     - إطلاق حدث `nano.fileupload.beforeEditDelete`.
     - حذف الملف القديم.
     - إطلاق حدث `nano.fileupload.afterEditDelete`.
     - إضافة الملف الجديد إلى علاقة النموذج.
   - إذا فشل أي جزء، يتم التراجع عن جميع التغييرات (لا يتم حذف الملف القديم).

5. **جلب الملفات**:
   - يمكن استدعاء `GET /files` مع `model_class` و `field` و `model_id` لجلب الملفات المرتبطة، أو مع `temp_session_key` لجلب الملفات المؤقتة.
   - **عند استخدام `temp_session_key`، يتم التحقق من صحته ومطابقته باستخدام `validateAndMatchTempKey`** قبل تنفيذ الاستعلام.

6. **استخدام نقاط النهاية الجديدة (منذ الإصدار 1.2.3)**:
   - يمكن للواجهات الأمامية استخدام `/models` لمعرفة الموديولات والحقول المتاحة.
   - يمكن استخدام `/permissions/check` للتحقق من صلاحية المستخدم قبل محاولة الرفع.
   - يمكن استخدام `/temp-key/validate` للتأكد من صحة المفتاح المؤقت قبل استخدامه.

---

## 4. حالات الاستخدام النموذجية

### أ. تسجيل نموذج منتج مع صورة رئيسية ومعرض صور (باستخدام الإعدادات المتقدمة و `disabled_operations`)
```php
// في Plugin.php
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'],
            'disabled_operations' => ['delete'], // تعطيل الحذف على مستوى المنتج
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'auto_resize' => true,
                    'resize_options' => ['width' => 800, 'height' => 600, 'mode' => 'crop'],
                    'auto_watermark' => true,
                    'watermark_options' => ['position' => 'bottom-right', 'resize_percentage' => 15],
                    'storage_disk' => 's3',
                    'disabled_operations' => ['edit'], // تعطيل استبدال الصورة فقط
                ],
                'gallery' => [
                    'type' => 'multiple',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'max_files' => 10,
                    'multiple' => true,
                ],
            ],
        ],
    ];
}
```

### ب. رفع صورة لمنتج جديد (باستخدام مفتاح جلسة مؤقت)
```php
// 1. رفع الصورة عبر API
$response = $api->post('/upload', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'image',
    'file_base64' => 'data:image/jpeg;base64,...',
]);
$tempKey = $response['temp_session_key'];

// 2. إنشاء المنتج وحفظه
$product = new Product();
$product->name = 'منتج جديد';
$product->save();

// 3. ربط الصورة بالمنتج (يتم من الخادم)
FileUploadService::instance()->attachTempFiles($product, 'image', $tempKey);
```

### ج. استبدال صورة منتج موجود (عملية edit)
```php
// رفع الصورة الجديدة مع تمرير model_id
$response = $api->post('/upload', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'image',
    'model_id' => 10,
    'file_base64' => 'data:image/jpeg;base64,...',
]);
// ستحل الصورة الجديدة محل القديمة تلقائياً، وسيتم حذف القديمة
```

### د. رفع عدة صور لمعرض منتج موجود
```php
$response = $api->post('/upload-multiple', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'gallery',
    'model_id' => 10,
    'files' => [$file1, $file2],
]);
```

### هـ. جلب صور معرض منتج مع صور مصغرة
```php
$files = $api->get('/files', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'gallery',
    'model_id' => 10,
    'with_thumbs' => true,
    'thumb_sizes' => [
        'small' => [150, 150, 'crop'],
        'medium' => [300, 300, 'crop'],
    ],
]);
```

### و. الاستماع للأحداث (مثل إرسال إشعار WebSocket عند استبدال ملف)
```php
Event::listen('nano.fileupload.afterEditDelete', function ($oldFile, $modelClass, $field, $model) {
    // إرسال إشعار بأن الملف القديم تم حذفه واستبداله بآخر جديد
    broadcast(new FileReplacedEvent($oldFile, $model));
});
```

### ز. استخدام نقاط النهاية الجديدة للاستعلام عن الموديولات (منذ الإصدار 1.2.3)
```php
// جلب جميع الموديولات المتاحة للمستخدم الحالي
$models = $api->get('/models');

// جلب إعدادات حقل معين
$fieldConfig = $api->get('/models/Nano%5CShop%5CModels%5CProduct/fields/image');

// التحقق من صلاحية المستخدم لاستبدال صورة منتج
$permission = $api->post('/permissions/check', [
    'model_class' => 'Nano\Shop\Models\Product',
    'operation' => 'edit',
    'field' => 'image',
]);
if ($permission['data']['allowed']) {
    // عرض واجهة استبدال الصورة
}

// التحقق من صحة مفتاح مؤقت قبل استخدامه
$tempKeyValidation = $api->post('/temp-key/validate', [
    'temp_key' => $tempKey,
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'image',
]);
if ($tempKeyValidation['data']['valid']) {
    // المفتاح صالح، يمكن استخدامه
}
```

---

## 5. الفوائد والمزايا

- **الأمان**:
  - التحقق من صلاحيات المستخدم (backend/frontend) قبل كل عملية.
  - دعم أنواع المستخدمين المختلفة، مع إمكانية تخصيص التحقق.
  - استخدام `FileUploadUserManager` الموحد للحصول على المستخدم الحالي.
  - التحقق من حجم ونوع الملف قبل الرفع عبر `validateFile` (بما في ذلك القائمة السوداء للملفات الخطيرة).
  - مفاتيح جلسة مؤقتة موقعة بـ HMAC-SHA256 تحتوي على `userType` و `timestamp` لمنع التلاعب.
  - **توحيد التحقق من المفاتيح المؤقتة** (منذ الإصدار 1.2.2) عبر دالة `validateAndMatchTempKey` المستخدمة في جميع الدوال.
  - إمكانية تعطيل عمليات API بالكامل (`disable_upload`, `disable_delete`, `disable_get`, `disable_edit`).
  - إمكانية تعطيل عمليات محددة على مستوى النموذج أو الحقل عبر `disabled_operations` (منذ الإصدار 1.2.1).

- **المرونة**:
  - تسجيل أي نموذج مع إعدادات مخصصة لكل حقل.
  - إمكانية تحديد `allowed_user_types` لكل نموذج أو حقل.
  - دعم الصلاحيات لكل عملية (`add`, `edit`, `delete`, `view`) على مستوى النموذج أو الحقل.
  - إمكانية تخصيص محلل المستخدم ودالة التحقق من الصلاحيات.
  - دعم رفع الملفات بصيغ متعددة (multipart, base64).
  - دعم الرفع المؤقت للنماذج غير المحفوظة.
  - دعم التخزين المتعدد (أقراص S3, FTP, Local) عبر `storage_disk`.
  - دعم التحويل التلقائي للصور (تحجيم، علامة مائية).
  - دعم WebSocket عبر الأحداث.
  - دالة `updateFieldConfig` لتحديث إعدادات حقل معين دون إعادة تسجيل النموذج.
  - **نقاط نهاية API جديدة للاستعلام عن الموديولات والصلاحيات والتحقق من المفاتيح المؤقتة** (منذ الإصدار 1.2.3).

- **الأداء**:
  - استخدام مفاتيح جلسة مؤقتة لتجنب حفظ الملفات بشكل غير مرتبط.
  - إمكانية جلب الملفات مع صور مصغرة مخصصة.
  - تخزين إعدادات النماذج في الذاكرة بعد التحميل.
  - تخزين مؤقت لنتائج `getFieldConfig` و `getFieldConstraints` لتقليل استعلامات قاعدة البيانات.
  - استخدام المعاملات (`Db::transaction`) في عمليات الاستبدال لضمان التكاملية وتجنب فقدان البيانات.

- **قابلية التوسع**:
  - يمكن تسجيل نماذج من أي إضافة عبر دالة `registerFileUploadFields` أو حدث `nano.api.fileupload.registerModels`.
  - إمكانية توسيع المنطق عبر الأحداث (`modelRegistered`, `beforeUpload`, `afterUpload`, `beforeEditDelete`, `afterEditDelete`, إلخ).
  - فصل واضح بين طبقات التسجيل، الخدمة، المستخدمين، وواجهة API.
  - دعم الحقول الجديدة في جدول `system_files` (`disk`, `hash`, `meta`, `expires_at`, `session_key`) لتمكين ميزات متقدمة.

- **تجربة المطور**:
  - واجهة برمجية بسيطة وموحدة.
  - توثيق شامل باللغة العربية.
  - دعم عمليات الرفع الفردية والمتعددة.
  - إدارة الملفات المؤقتة تلقائيًا.
  - أكواد خطأ فريدة ورسائل مترجمة.
  - نظام اختبار متكامل (`FileUploadPlusTest`) يمكن تشغيله بسهولة عبر API للتحقق من صحة التثبيت.
  - **نقاط نهاية API للاستعلام** تسهل بناء واجهات أمامية ديناميكية.

---

## 6. مثال عملي متكامل (كود كامل مع ميزات متقدمة)

```php
<?php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadRegistry;
use Nano\FileUpload\Classes\FileUploadUserManager;
use Nano\Shop\Models\Product;

// 1. تسجيل النموذج (في Plugin.php)
public function registerFileUploadFields()
{
    return [
        Product::class => [
            'allowed_user_types' => ['backend'],
            'permissions' => [
                'add'    => 'products.create',
                'edit'   => 'products.update',
                'delete' => 'products.delete',
                'view'   => 'products.view',
            ],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'auto_resize' => true,
                    'resize_options' => ['width' => 800, 'height' => 600],
                    'auto_watermark' => true,
                    'storage_disk' => 's3',
                ],
            ],
        ],
    ];
}

// 2. في متحكم API لإنشاء منتج
public function createProduct(Request $request)
{
    $base64Image = $request->input('image_base64');
    $productName = $request->input('name');

    // رفع الصورة مؤقتًا
    $service = FileUploadService::instance();
    $tempKey = $service->generateTempSessionKey(Product::class, 'image');
    $uploadResult = $service->upload(Product::class, 'image', $base64Image, [
        'temp_session_key' => $tempKey,
    ]);

    if (!$uploadResult['status']) {
        return response()->json(['error' => $uploadResult['error'], 'error_code' => $uploadResult['error_code']], $uploadResult['code']);
    }

    // إنشاء المنتج
    $product = new Product();
    $product->name = $productName;
    $product->save();

    // ربط الصورة بالمنتج
    $attachResult = $service->attachTempFiles($product, 'image', $tempKey);
    if (!$attachResult['status']) {
        // معالجة الخطأ
    }

    return response()->json(['product_id' => $product->id]);
}

// 3. في متحكم API لتحديث صورة المنتج (استبدال)
public function updateProductImage(Request $request, $id)
{
    $product = Product::findOrFail($id);
    $base64Image = $request->input('image_base64');

    $service = FileUploadService::instance();
    $result = $service->upload(Product::class, 'image', $base64Image, [
        'model' => $product,
        // 'skip_permission_check' => true // فقط للاختبارات
    ]);

    if (!$result['status']) {
        return response()->json(['error' => $result['error'], 'error_code' => $result['error_code']], $result['code']);
    }

    return response()->json(['message' => 'تم استبدال الصورة بنجاح']);
}

// 4. مثال لاستخدام نقاط النهاية الجديدة للتحقق من الصلاحية قبل عرض النموذج
public function showProductEditForm($id)
{
    $product = Product::findOrFail($id);
    // التحقق من صلاحية المستخدم لاستبدال الصورة
    $client = new \GuzzleHttp\Client();
    $response = $client->post(env('API_URL') . '/permissions/check', [
        'headers' => ['Authorization' => 'Bearer ' . $request->bearerToken()],
        'json' => [
            'model_class' => Product::class,
            'operation' => 'edit',
            'field' => 'image',
        ],
    ]);
    $permission = json_decode($response->getBody(), true);
    if (!$permission['data']['allowed']) {
        return view('product.edit', ['product' => $product, 'can_edit_image' => false]);
    }
    return view('product.edit', ['product' => $product, 'can_edit_image' => true]);
}
```

---

## 7. الاعتماديات والمتطلبات

- **PHP** (>= 8.0).
- **Laravel / OctoberCMS** (أي إطار عمل متوافق مع `System\Models\File`).
- **إضافة `Nano.Api`** (لتوفير `ApiController`).
- **Composer** لتثبيت الحزمة.
- (اختياري) **إضافة `Nano2.Watermark`** للعلامة المائية التلقائية.
- (اختياري) **مكتبة WebSocket** للإشعارات الفورية.

**ملاحظة**: تم نقل كلاس `Base64` من `Nano.API` إلى داخل الإضافة ( `Nano\FileUpload\Classes\Base64` ) بدءاً من الإصدار 1.1.0، لذا لم يعد هناك اعتماد إلزامي على `Nano.API` باستثناء `ApiController`.

---

## 8. الخاتمة

تمثل كلاسات `Nano.FileUpload` حلاً متكاملاً واحترافياً لإدارة رفع الملفات في تطبيقات نانوسوفت. بفضل التصميم الطبقي (Registry, Service, UserManager, API)، يمكن للمطورين بناء أنظمة رفع ملفات قوية وآمنة بسرعة وسهولة. مع إضافة `FileUploadService`، أصبح النظام أكثر تنظيماً وسهولة في الاستخدام، حيث يوفر نقطة دخول واحدة لتنفيذ عمليات الرفع والحذف والاسترجاع، مع دعم خيارات متقدمة مثل الرفع المؤقت، الصور المصغرة، والتحقق من الصلاحيات بأنواع المستخدمين المختلفة. كما أن دعم عملية `edit` (استبدال الملفات) باستخدام المعاملات والأحداث المخصصة يضمن تكاملية البيانات وأمانها. الإصدارات الأخيرة (1.2.1، 1.2.2، 1.2.3) أضافت تحسينات كبيرة: دوال تحقق متكاملة (`validate`, `canGlobal`, ...)، إمكانية تعطيل عمليات محددة (`disabled_operations`)، توحيد منطق التحقق من المفاتيح المؤقتة عبر تريت `HasFileUploadsMatchTempKey`، ونقاط نهاية API جديدة للاستعلام عن الموديولات والصلاحيات والتحقق من المفاتيح. هذا التصميم يضمن قابلية التوسع والصيانة، ويلبي احتياجات التطبيقات المتقدمة دون المساس بأمان النظام أو أدائه.

---

## 9. التوثيق الإضافي

للحصول على تفاصيل أكثر عن كل كلاس، يُرجى مراجعة المستندات التالية:

- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
