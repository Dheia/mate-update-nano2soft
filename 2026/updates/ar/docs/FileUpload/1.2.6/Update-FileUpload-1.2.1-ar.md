## 2026-04-10 – 2026-04-11

**تحديث إضافة `Nano.FileUpload` – الإصدار 1.2.1**

### إعادة هيكلة نظام التحقق من الصلاحيات وإضافة تحكم دقيق على مستوى النموذج والحقل

---

### ملخص التحديثات

في الإصدار **1.2.1**، تم إعادة هيكلة نظام التحقق من الصلاحيات في `FileUploadRegistry` بالكامل، مع إضافة طبقات متعددة للتحقق (عالمي، موديول، حقل) وإتاحة تعطيل عمليات محددة بشكل مستقل. كما تم تحسين `FileUploadService` لاستخدام دوال `validate` الجديدة، وإعادة كتابة دالة `attachTempFiles` بشكل كامل، وتحديث `FileUploadException` برموز خطأ جديدة وكود HTTP مناسب.

---

### أهداف الإصدار

- **توحيد وإعادة هيكلة دوال التحقق** في `FileUploadRegistry` لتصبح أكثر وضوحاً وتنظيماً.
- **إضافة آلية `disabled_operations`** على مستوى النموذج والحقل لتعطيل عمليات محددة (`add`, `edit`, `delete`, `view`) بشكل مستقل.
- **توفير دوال `validate` متخصصة** ترمي استثناءات مناسبة بدلاً من إرجاع `false`، لتبسيط كتابة الكود في طبقة الخدمة.
- **تحسين كود `FileUploadService`** للاستفادة من دوال `validate` الجديدة، مما يقلل من الكود المكرر ويوحد رسائل الأخطاء.
- **إضافة دالة `attachTempFiles` محسّنة بالكامل** مع هيكل استجابة موحد وأحداث WebSocket.
- **تحديث `FileUploadException`** برموز خطأ جديدة وتعديل كود HTTP لبعض الأخطاء لتجنب التعارض.

---

### الميزات الجديدة

#### 1. دوال التحقق المتخصصة في `FileUploadRegistry`

تم إضافة مجموعة من الدوال الجديدة التي تفصل مستويات التحقق:

| الدالة | الوصف |
|--------|-------|
| `canGlobal($operation)` | التحقق من تفعيل العملية عالمياً (عبر الإعدادات العامة `disable_upload`, `disable_edit`, `disable_delete`, `disable_get`). |
| `validateGlobal($operation)` | مثل `canGlobal` ولكن ترمي `FileUploadException` إذا كانت العملية معطلة. |
| `canModel($modelClass, $operation)` | التحقق من تفعيل العملية على مستوى النموذج (من خلال `disabled_operations` في تعريف النموذج). |
| `validateModel($modelClass, $operation)` | مثل `canModel` ولكن ترمي استثناء إذا كانت العملية معطلة. |
| `canField($modelClass, $field, $operation)` | التحقق من تفعيل العملية على مستوى الحقل (من خلال `disabled_operations` في تعريف الحقل). |
| `validateField($modelClass, $field, $operation)` | مثل `canField` ولكن ترمي استثناء. |
| `validate($modelClass, $operation, $userType, $user, $field)` | دالة متكاملة تجمع التحقق من المستوى العام، الموديول، الحقل، نوع المستخدم، والصلاحيات، وترمي الاستثناء المناسب عند أول فشل. |

**مثال استخدام `validate`:**
```php
// في FileUploadService::upload
$this->registry->validate($modelClass, $operation, $userType, $user, $field);
$this->registry->validateField($modelClass, $field, $operation);
```

#### 2. إضافة خاصية `disabled_operations`

أصبح من الممكن الآن تعطيل عمليات محددة على مستوى النموذج أو الحقل باستخدام المفتاح `disabled_operations` في تعريفات التسجيل:

```php
// في تعريف النموذج (Plugin::registerFileUploadFields)
\Nano\Shop\Models\Product::class => [
    'disabled_operations' => ['delete'], // تعطيل الحذف على مستوى المنتج
    'fields' => [
        'image' => [
            'type' => 'image',
            'disabled_operations' => ['edit'], // تعطيل استبدال الصورة فقط
        ],
    ],
];
```

هذا يسمح بمنع عمليات معينة بشكل دقيق دون الحاجة إلى تعطيل الصلاحيات الكاملة أو تعديل الإعدادات العامة.

#### 3. تحسين دالة `can()` في `FileUploadRegistry`

أعيدت كتابة دالة `can()` لتعتمد على سلسلة التحقق الجديدة:

1. التحقق من المستوى العام (`canGlobal`)
2. التحقق من مستوى الموديول (`canModel`)
3. التحقق من مستوى الحقل (`canField`) إن وُجد
4. التحقق من نوع المستخدم (`isUserTypeAllowed`)
5. التحقق من الصلاحيات المحددة (`permissions`)

هذا يضمن عدم السماح بأي عملية يتم تعطيلها في أي مستوى.

#### 4. رموز خطأ جديدة في `FileUploadException`

أضيفت الثوابت التالية:

- `ERR_EDIT_DISABLED_GLOBALLY` – عندما يكون التعديل معطلاً عالمياً (`disable_edit = true`)
- `ERR_MODEL_OPERATION_DISABLED` – عندما تكون العملية معطلة على مستوى النموذج
- `ERR_FIELD_OPERATION_DISABLED` – عندما تكون العملية معطلة على مستوى الحقل

**تغيير كود HTTP:**
- `ERR_PERMISSION_DENIED` تم تغيير كود HTTP من `403` إلى `422` (Unprocessable Entity) لتجنب التعارض مع حالة "حساب غير نشط" التي تستخدم `403`.
- `ERR_UPLOAD_DISABLED_GLOBALLY`, `ERR_EDIT_DISABLED_GLOBALLY`, `ERR_DELETE_DISABLED_GLOBALLY`, `ERR_GET_DISABLED_GLOBALLY` تستخدم كود `503` (Service Unavailable).
- `ERR_MODEL_OPERATION_DISABLED`, `ERR_FIELD_OPERATION_DISABLED` تستخدم كود `423` (Locked).

#### 5. تحسين دالة `upload` في `FileUploadService`

تم استبدال الكود اليدوي للتحقق من الصلاحيات باستدعاءات `validate`:

**قبل:**
```php
if (!$this->registry->can($modelClass, $operation, $userType, $user, $field)) {
    throw new FileUploadException(...);
}
```

**بعد:**
```php
$this->registry->validate($modelClass, $operation, $userType, $user, $field);
$this->registry->validateField($modelClass, $field, $operation);
```

كما تم إضافة خيار `skip_permission_check` لتجاوز التحقق من الصلاحيات (مفيد في الاختبارات الآلية).

#### 6. إعادة كتابة دالة `attachTempFiles` بالكامل

أصبحت الدالة تعيد هيكل استجابة موحد يحتوي على `code`, `status`, `message`, `data`, `error_code`, `process_data`, `debug`. وتقوم بـ:

- التحقق من تسجيل النموذج والحقل في `FileUploadRegistry`.
- التحقق من الصلاحيات (باستخدام عملية `edit` لأن الربط يعد تعديلاً على النموذج).
- التحقق من صحة المفتاح المؤقت باستخدام `validateTempSessionKey`.
- التحقق من تطابق `modelClass` و `field` مع بيانات المفتاح.
- التحقق من أن المستخدم الحالي هو من أنشأ الملفات المؤقتة (مقارنة `userId` و `userType`).
- البحث عن الملفات المؤقتة (`session_key`).
- ربط الملفات بالنموذج (تعيين `attachment_id`, `attachment_type`, وإلغاء `session_key`, `expires_at`).
- إضافة الملفات إلى علاقة النموذج (إن وجدت).
- إطلاق حدث `nano.fileupload.afterAttach`.
- إرسال إشعار WebSocket.

**مثال الاستجابة الناجحة:**
```json
{
    "code": 200,
    "status": true,
    "message": "تم ربط 3 ملفات بنجاح",
    "data": [
        {"id": 10, "title": "image.jpg", "path": "/storage/...", "size": 1024, "content_type": "image/jpeg"}
    ],
    "process_data": {
        "field_config": {...},
        "key_data": {...},
        "attached_count": 3,
        "temp_session_key": "tmp_xxx",
        "user_type": "backend"
    }
}
```

#### 7. تحسين دالة `getFiles`

تم تحسين استخراج `morphClass` باستخدام `app($modelClass)->getMorphClass()` بدلاً من `$modelClass::getMorphClass()` لتفادي الأخطاء في بعض السياقات (مثل عندما لا يكون النموذج محملاً مسبقاً). كما تم توحيد التحقق من صلاحية `view` باستخدام `$this->registry->validate($modelClass, 'view', ...)`.

#### 8. تحسين دالة `deleteFile`

تمت إعادة هيكلة منطق التحقق من الصلاحية ليشمل أربع حالات:

| الحالة | الوصف | طريقة التحقق |
|--------|-------|---------------|
| 1 | تم تمرير `modelClass` و `field` | استخدام `registry->can()` مع المعطيات الممررة |
| 2 | الملف مرتبط بنموذج محفوظ (`attachment_type`, `attachment_id`, `field`) | استخراج القيم من الملف واستخدام `registry->can()` |
| 3 | الملف مؤقت (`session_key`) | التحقق من صحة المفتاح ومطابقة المستخدم |
| 4 | لا توجد معلومات كافية | رفض الحذف مباشرة |

**التحسين الخاص بالملفات المؤقتة:**
```php
elseif ($file->session_key) {
    $keyData = $this->validateTempSessionKey($file->session_key);
    if (!$keyData) throw ...;
    $currentUserId = $user->getKey() ?? 'guest';
    if ($keyData['userId'] != $currentUserId) throw ...;
    $usedModelClass = $keyData['modelClass'];
    $usedField = $keyData['field'];
    $isAuthorized = true;
}
```

#### 9. تحسين دالة `logFailedAttempt`

تم تحسين استخراج `userId` لدعم أنواع متعددة من كائنات المستخدم:

```php
$userId = 'guest';
if ($user) {
    if (method_exists($user, 'getKey')) $userId = $user->getKey();
    elseif (property_exists($user, 'id')) $userId = $user->id;
    elseif (method_exists($user, 'getId')) $userId = $user->getId();
    else $userId = 'unknown';
}
```

هذا يقلل الأخطاء عند وجود أنواع مختلفة من المستخدمين (backend, frontend, مخصص).

#### 10. إضافة خيار `skip_permission_check` في دوال متعددة

أضيف خيار `skip_permission_check` في دوال `upload`, `uploadMultiple`, `attachTempFiles` لتجاوز التحقق من الصلاحيات بالكامل. هذا الخيار مفيد جداً في بيئات الاختبار الآلي (مثل `FileUploadPlusTest`) حيث لا يكون هناك مستخدم حقيقي أو صلاحيات محددة.

**مثال الاستخدام:**
```php
$result = $service->upload($modelClass, $field, $file, [
    'skip_permission_check' => true
]);
```

---

### الفوائد والقيمة المضافة

- **كود أكثر نظافة**: دوال `validate` تلغي الحاجة إلى كتابة كتل شرطية متكررة للتحقق من الصلاحيات.
- **تحكم دقيق**: `disabled_operations` يسمح بتعطيل عمليات معينة على مستوى النموذج أو الحقل دون المساس بالصلاحيات الكلية أو الإعدادات العامة.
- **توافق أفضل مع أنظمة المصادقة**: تغيير كود HTTP لـ `ERR_PERMISSION_DENIED` من `403` إلى `422` يتجنب الخلط بين "غير مصرح" و "حساب غير نشط".
- **دالة `attachTempFiles` احترافية**: أصبحت الدالة كاملة ومتكاملة مع هيكل الاستجابة الموحد والأحداث وإشعارات WebSocket.
- **مرونة الاختبارات**: خيار `skip_permission_check` يسهل كتابة اختبارات وحدة لا تعتمد على وجود مستخدم حقيقي.
- **أمان محسن**: التحقق من الصلاحية في `deleteFile` يغطي جميع الحالات الممكنة، مما يمنع الحذف غير المصرح به.

---

### التغييرات التي قد تؤثر على التوافق

1. **تغيير كود HTTP لـ `ERR_PERMISSION_DENIED`**:
   - إذا كان لديك كود يعتمد على استلام `403` لهذا الخطأ، فسيتلقى الآن `422`. يُنصح بتحديث أي معالجة للخطأ تعتمد على الكود.

2. **دالة `attachTempFiles` أصبحت تعيد هيكلاً مختلفاً**:
   - كانت تعيد سابقاً `true/false` أو مصفوفة بسيطة. الآن تعيد مصفوفة موحدة تحتوي على `code`, `status`, `message`, `data`, إلخ. إذا كنت تستخدم هذه الدالة مباشرة، فستحتاج إلى تحديث الكود للتعامل مع الهيكل الجديد.

3. **إضافة `disabled_operations`**:
   - لا تؤثر على الإعدادات الحالية ما لم تقم بإضافتها صراحة. الإعدادات الافتراضية تسمح بجميع العمليات.

4. **دوال `validate` الجديدة**:
   - تم استخدامها داخل `FileUploadService`، ولكن إذا كنت تستدعي `can()` مباشرة، فستظل تعمل كما هي (مع تحسين السلسلة الجديدة).

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال `FileUploadRegistry.php` بالنسخة التي تحتوي على دوال `validate` الجديدة وخاصية `disabled_operations`.
   - استبدال `FileUploadService.php` بالنسخة المحسّنة التي تستخدم `validate` وتحتوي على `attachTempFiles` الجديدة.
   - استبدال `FileUploadException.php` بالنسخة التي تحتوي على رموز الخطأ الجديدة.

2. **لا توجد هجرات جديدة**:
   - هذا الإصدار لا يتطلب تغييرات في قاعدة البيانات أو إضافة أعمدة جديدة.

3. **لا توجد تغييرات في الإعدادات**:
   - يظل ملف `config.php` كما هو. يمكن استخدام متغيرات البيئة الموجودة (`disable_upload`, `disable_edit`, `disable_delete`, `disable_get`) بنفس الطريقة.

4. **مراجعة تعريفات النماذج** (اختياري):
   - إذا كنت ترغب في استخدام `disabled_operations`، أضف المفتاح إلى تعريفات النماذج أو الحقول كما هو موضح في الأمثلة أعلاه.

5. **اختبار التوافق**:
   - يُنصح بتشغيل اختبارات `FileUploadPlusTest` (الإصدار 1.2.0) للتأكد من أن جميع العمليات تعمل بشكل صحيح مع التغييرات الجديدة.
   - التحقق من أن صلاحيات `edit` تعمل كما هو متوقع عند استبدال الملفات (في حال استخدامها).

6. **تحديث أي كود يعتمد على `attachTempFiles`**:
   - إذا كنت تستدعي `attachTempFiles` مباشرة، فستحتاج إلى تعديل الكود ليتعامل مع هيكل الاستجابة الجديد.

---

### الخاتمة

يمثل الإصدار **1.2.1** نقلة نوعية في نضج إدارة الصلاحيات داخل إضافة `Nano.FileUpload`. من خلال إضافة دوال `validate` المتخصصة، وخاصية `disabled_operations`، وتحسين دالتَي `attachTempFiles` و `deleteFile`، أصبح النظام أكثر تنظيماً وأماناً ومرونة. هذه التغييرات تجعل الإضافة قادرة على التعامل مع سيناريوهات معقدة مثل استبدال الملفات، وتعطيل عمليات معينة مؤقتاً، والتحقق من الصلاحيات متعددة المستويات، مع الحفاظ على التوافق مع الإصدارات السابقة قدر الإمكان.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)
