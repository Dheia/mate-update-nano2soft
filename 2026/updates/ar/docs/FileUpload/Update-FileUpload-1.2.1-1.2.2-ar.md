## 2026-04-10 – 2026-04-14

**تحديثات إضافة `Nano.FileUpload` – الإصداران 1.2.1 و 1.2.2**

### ملخص التحديثات

شهدت إضافة `Nano.FileUpload` تطورين متتاليين رئيسيين في الإصدارين **1.2.1** و **1.2.2**، ركز الأول منهما على إعادة هيكلة متكاملة لنظام التحقق من الصلاحيات وإضافة آليات تحكم دقيقة على مستوى النموذج والحقل، بينما اهتم الثاني بتوحيد وتبسيط منطق التحقق من مفاتيح الجلسة المؤقتة وإنشاء تريت متخصص يضم دوالاً متقدمة لهذا الغرض.

---

## الإصدار 1.2.1 – إعادة هيكلة نظام التحقق من الصلاحيات

### أهداف الإصدار

- **توحيد وإعادة هيكلة دوال التحقق** في `FileUploadRegistry` لتصبح أكثر وضوحاً وتنظيماً.
- **إضافة آلية `disabled_operations`** على مستوى النموذج والحقل لتعطيل عمليات محددة (`add`, `edit`, `delete`, `view`) بشكل مستقل.
- **توفير دوال `validate` متخصصة** ترمي استثناءات مناسبة بدلاً من إرجاع `false`، لتبسيط كتابة الكود في طبقة الخدمة.
- **تحسين كود `FileUploadService`** للاستفادة من دوال `validate` الجديدة، مما يقلل من الكود المكرر ويوحد رسائل الأخطاء.
- **إضافة دالة `attachTempFiles` محسّنة بالكامل** مع هيكل استجابة موحد وأحداث WebSocket.
- **تحديث `FileUploadException`** برموز خطأ جديدة وتعديل كود HTTP لبعض الأخطاء لتجنب التعارض.

### الميزات الجديدة

#### 1. دوال التحقق المتخصصة في `FileUploadRegistry`

أضيفت مجموعة من الدوال الجديدة التي تفصل مستويات التحقق:

| الدالة | الوصف |
|--------|-------|
| `canGlobal($operation)` | التحقق من تفعيل العملية عالمياً (عبر الإعدادات العامة). |
| `validateGlobal($operation)` | مثل `canGlobal` ولكن ترمي `FileUploadException` إذا كانت العملية معطلة. |
| `canModel($modelClass, $operation)` | التحقق من تفعيل العملية على مستوى النموذج (`disabled_operations`). |
| `validateModel($modelClass, $operation)` | مثل `canModel` ولكن ترمي استثناء إذا كانت العملية معطلة. |
| `canField($modelClass, $field, $operation)` | التحقق من تفعيل العملية على مستوى الحقل. |
| `validateField($modelClass, $field, $operation)` | مثل `canField` ولكن ترمي استثناء. |
| `validate($modelClass, $operation, $userType, $user, $field)` | دالة متكاملة تجمع التحقق من المستوى العام، الموديول، الحقل، نوع المستخدم، والصلاحيات، وترمي الاستثناء المناسب عند أول فشل. |

#### 2. إضافة خاصية `disabled_operations`

أصبح من الممكن الآن تعطيل عمليات محددة على مستوى النموذج أو الحقل باستخدام المفتاح `disabled_operations` في تعريفات التسجيل:

```php
// على مستوى النموذج
'disabled_operations' => ['delete', 'edit']

// على مستوى الحقل
'disabled_operations' => ['view']
```

هذا يسمح بمنع عمليات معينة بشكل دقيق دون الحاجة إلى تعطيل الصلاحيات الكاملة.

#### 3. تحسين دالة `can()` في `FileUploadRegistry`

أعيدت كتابة دالة `can()` لتعتمد على سلسلة التحقق الجديدة: العالمية ← الموديول ← الحقل ← نوع المستخدم ← الصلاحيات. هذا يضمن عدم السماح بأي عملية يتم تعطيلها في أي مستوى.

#### 4. رموز خطأ جديدة في `FileUploadException`

أضيفت الثوابت التالية:

- `ERR_EDIT_DISABLED_GLOBALLY`
- `ERR_MODEL_OPERATION_DISABLED`
- `ERR_FIELD_OPERATION_DISABLED`

كما تم تغيير كود HTTP الخاص بـ `ERR_PERMISSION_DENIED` من `403` إلى `422` (Unprocessable Entity) لتجنب التعارض مع حالة `403` المستخدمة عادةً لحساب غير نشط.

وتم تخصيص كود HTTP `503` للأخطاء العالمية المعطلة (`UPLOAD_DISABLED_GLOBALLY`، إلخ) وكود `423` (Locked) للأخطاء على مستوى الموديول أو الحقل.

#### 5. تحسين دالة `upload` في `FileUploadService`

تم استبدال الكود اليدوي للتحقق من الصلاحيات:

```php
// الكود القديم (متعدد الأسطر)
if (!$this->registry->can($modelClass, $operation, $userType, $user, $field)) {
    throw new FileUploadException(...);
}

// الكود الجديد (بسيط وموحد)
$this->registry->validate($modelClass, $operation, $userType, $user, $field);
$this->registry->validateField($modelClass, $field, $operation);
```

#### 6. إعادة كتابة دالة `attachTempFiles` بالكامل

أصبحت الدالة تعيد هيكل استجابة موحد يحتوي على `code`, `status`, `message`, `data`, `error_code`, `process_data`، وتقوم بـ:

- التحقق من تسجيل النموذج والحقل.
- التحقق من الصلاحيات (عملية `edit`).
- التحقق من صحة المفتاح المؤقت ومطابقة النموذج والحقل والمستخدم.
- البحث عن الملفات المؤقتة وربطها بالنموذج.
- إلغاء صلاحية الملفات (`expires_at = null`) ومسح `session_key`.
- إطلاق حدث `nano.fileupload.afterAttach`.
- إرسال إشعار WebSocket.

#### 7. إضافة خيار `skip_permission_check`

أضيف خيار `skip_permission_check` في دوال `upload`, `uploadMultiple`, `attachTempFiles` لتجاوز التحقق من الصلاحيات، وهو مفيد جداً في بيئات الاختبار الآلي.

#### 8. تحسين دالة `getFiles`

تم تحسين استخراج `morphClass` باستخدام `app($modelClass)->getMorphClass()` بدلاً من `$modelClass::getMorphClass()` لتفادي الأخطاء في بعض السياقات. كما تم توحيد التحقق من الصلاحيات باستخدام `validate()`.

#### 9. تحسين دالة `deleteFile`

تم تغطية أربع حالات للتحقق من الصلاحية:

1. تمرير `modelClass` و `field` بشكل صريح.
2. الملف مرتبط بنموذج محفوظ (استخراج `attachment_type` و `field`).
3. الملف مؤقت (`session_key`) – يتم التحقق من المفتاح ومطابقة المستخدم.
4. في حال عدم توفر معلومات كافية، يتم رفض الحذف.

كما تم استخدام `registry->can()` في الحالتين الأولى والثانية لضمان التحقق من الصلاحية على مستوى النموذج والحقل.

#### 10. تحسين دالة `logFailedAttempt`

تم تحسين استخراج `userId` لدعم أنواع متعددة من كائنات المستخدم: `getKey()`, `id`, `getId()`، مما يقلل الأخطاء عند وجود أنواع مختلفة من المستخدمين (backend, frontend, مخصص).

### الفوائد

- **كود أكثر نظافة**: دوال `validate` تلغي الحاجة إلى كتابة كتل شرطية متكررة للتحقق من الصلاحيات.
- **تحكم دقيق**: `disabled_operations` يسمح بتعطيل عمليات معينة دون المساس بالصلاحيات الكلية.
- **توافق أفضل مع أنظمة المصادقة**: تغيير كود HTTP لـ `403` يتجنب الخلط بين "غير مصرح" و "حساب غير نشط".
- **دالة `attachTempFiles` احترافية**: أصبحت الدالة كاملة ومتكاملة مع هيكل الاستجابة الموحد والأحداث.
- **مرونة الاختبارات**: خيار `skip_permission_check` يسهل كتابة اختبارات وحدة لا تعتمد على وجود مستخدم حقيقي.

---

## الإصدار 1.2.2 – توحيد التحقق من مفاتيح الجلسة المؤقتة

### أهداف الإصدار

- **إنشاء تريت متخصص** (`HasFileUploadsMatchTempKey`) يحتوي على دالة متطورة للتحقق من صحة المفاتيح المؤقتة ومطابقتها مع النموذج والحقل والمستخدم.
- **توحيد منطق التحقق** المستخدم في أربع دوال مختلفة (`upload`, `deleteFile`, `attachTempFiles`, `getFiles`) في مكان واحد.
- **إزالة الكود المكرر** الذي كان يزيد عن 50 سطراً من عمليات التحقق اليدوية المتفرقة.
- **إضافة خيارات متقدمة** للتحكم في سلوك التحقق: الوضع الصارم، فترة السماح للمفاتيح منتهية الصلاحية، التخزين المؤقت للنتائج، وجمع بيانات الأداء.
- **تحسين توافق الأحداث** عبر إضافة دالة `fireEventSafeInTrait` التي تدعم Laravel و OctoberCMS.

### الميزات الجديدة

#### 1. تريت `HasFileUploadsMatchTempKey`

تم إنشاء التريت في المسار `Nano\FileUpload\Classes\FileUploadService\HasFileUploadsMatchTempKey` ويضم المكونات التالية:

##### أ. دالة `validateAndMatchTempKey` (الدالة الأساسية)

هذه الدالة تجمع كل ما تحتاجه عملية التحقق من المفتاح المؤقت في مكان واحد. تستقبل المعاملات التالية:

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `$tempKey` | `string|array` | مفتاح الجلسة المؤقت (نص) أو بياناته (مصفوفة) |
| `$modelClass` | `string` | اسم النموذج المتوقع |
| `$field` | `string` | اسم الحقل المتوقع |
| `$user` | `mixed|null` | المستخدم الحالي (اختياري) |
| `$options` | `array` | خيارات إضافية للتحكم في السلوك |

وتقدم الخيارات التالية (مع قيم افتراضية):

| الخيار | النوع | الوصف |
|--------|-------|-------|
| `throw_on_failure` | `bool` | إطلاق استثناء عند الفشل (`true`) أو إرجاع نتيجة خطأ كمصفوفة (`false`) |
| `skip_user_check` | `bool` | تخطي التحقق من المستخدم (للاستخدام الداخلي أو الاختبارات) |
| `strict_mode` | `bool` | وضع صارم: يتطلب تطابق النموذج والحقل بالكامل (`true`) |
| `stop_on_first_failure` | `bool` | إيقاف عند أول خطأ (`true`) أو تجميع الأخطاء (`false`) |
| `allow_expired_key` | `bool` | السماح بالمفاتيح منتهية الصلاحية ضمن فترة سماح |
| `expiry_grace_period` | `int` | فترة السماح بالثواني (افتراضي `300`) |
| `custom_validator` | `callable|null` | دالة تحقق إضافية (تستقبل `$keyData` وتعيد `bool`) |
| `cache_results` | `bool` | تخزين نتائج التحقق مؤقتاً في نفس الطلب |
| `collect_metadata` | `bool` | جمع بيانات أداء (مدة التنفيذ، الخطوات المنفذة) |
| `key_data_overrides` | `array` | تجاوزات لبيانات المفتاح (مثلاً تغيير `userId`) |
| `before_validation` | `callable|null` | دالة تُستدعى قبل بدء التحقق |
| `after_validation` | `callable|null` | دالة تُستدعى بعد انتهاء التحقق |
| `log_failures_only` | `bool` | تسجيل الفشل فقط (`true`) أو تسجيل الكل (`false`) |

**مثال الاستخدام الأساسي:**
```php
$keyData = $this->validateAndMatchTempKey(
    $tempSessionKey,
    $modelClass,
    $field,
    $user,
    ['throw_on_failure' => true, 'strict_mode' => true]
);
```

##### ب. دالة `manualValidateTempSessionKeyWithGrace`

دالة مساعدة داخلية تُستخدم عند تفعيل خيار `allow_expired_key`، وتقوم بالتحقق من المفتاح المنتهي مع فترة سماح دون تعديل الدالة الأصلية `validateTempSessionKey`.

##### ج. دالة `fireEventSafeInTrait`

دالة مساعدة لإطلاق الأحداث بشكل آمن ومتوافق مع كل من Laravel و OctoberCMS. تتحقق أولاً مما إذا كان الكائن المستخدم (مثل `FileUploadService`) يمتلك دالة `fireEventSafe`، وتستخدمها إن وجدت، وإلا تطبق منطقاً مكافئاً يدعم `Event::fire` و `Event::dispatch` و `event()` helper.

#### 2. تحديث دالة `upload` في `FileUploadService`

عند وجود `temp_session_key` في الخيارات، يتم الآن استدعاء `validateAndMatchTempKey` للتحقق من أن المفتاح يخص نفس النموذج والحقل والمستخدم قبل استخدامه.

```php
} elseif (isset($options['temp_session_key']) && !empty($options['temp_session_key'])) {
    $tempSessionKey = $options['temp_session_key'];
    $user = $this->userManager->getUser();
    $this->validateAndMatchTempKey($tempSessionKey, $modelClass, $field, $user, [
        'throw_on_failure' => true,
        'strict_mode' => true,
        'skip_user_check' => false,
    ]);
}
```

#### 3. تحديث دالة `deleteFile` في `FileUploadService`

تم استبدال الكود اليدوي الخاص بالملفات المؤقتة باستدعاء `validateAndMatchTempKey` مع `strict_mode = false` (لأن الملف المؤقت قد لا يكون له `attachment_type`).

```php
elseif ($file->session_key) {
    $keyData = $this->validateAndMatchTempKey(
        $file->session_key,
        $file->attachment_type ?: '',
        $file->field,
        $user,
        ['throw_on_failure' => true, 'strict_mode' => false, 'skip_user_check' => false]
    );
    $usedModelClass = $keyData['modelClass'];
    $usedField = $keyData['field'];
    $isAuthorized = true;
}
```

#### 4. تحديث دالة `attachTempFiles` في `FileUploadService`

تم استبدال أكثر من 20 سطراً من عمليات التحقق اليدوية (التحقق من المفتاح، مقارنة `modelClass`، مقارنة `field`، مقارنة `userId`، مقارنة `userType`) باستدعاء واحد:

```php
$keyData = $this->validateAndMatchTempKey(
    $tempSessionKey,
    $modelClass,
    $field,
    $user,
    ['throw_on_failure' => true, 'strict_mode' => true, 'skip_user_check' => false]
);
$process_data['key_data'] = $keyData;
```

#### 5. تحديث دالة `getFiles` في `FileUploadService`

تم استبدال الكود اليدوي للتحقق من المفتاح قبل استخدامه في الاستعلام:

```php
} elseif (isset($options['temp_session_key'])) {
    $user = $this->userManager->getUser();
    $this->validateAndMatchTempKey(
        $options['temp_session_key'],
        $modelClass,
        $field,
        $user,
        ['throw_on_failure' => true, 'strict_mode' => true, 'skip_user_check' => false]
    );
    $query->where('session_key', $options['temp_session_key']);
}
```

#### 6. تحسين إطلاق الأحداث في التريت

تم استبدال جميع استدعاءات `Event::fire` المباشرة داخل `validateAndMatchTempKey` باستدعاءات `$this->fireEventSafeInTrait`، مما يضمن عمل الأحداث في بيئات Laravel و OctoberCMS بشكل موحد.

### الفوائد

- **إزالة الكود المكرر**: تم التخلص من أكثر من 50 سطراً من عمليات التحقق اليدوية المتكررة.
- **زيادة الأمان**: التحقق الموحد يضمن عدم وجود ثغرات ناتجة عن نسيان التحقق من مطابقة النموذج أو المستخدم في أي دالة.
- **مرونة عالية**: الخيارات المتعددة في `validateAndMatchTempKey` تسمح بتخصيص السلوك حسب الحالة (وضع صارم، فترة سماح، تخزين مؤقت، إلخ).
- **توافق محسن**: دالة `fireEventSafeInTrait` تضمن عمل الأحداث في كافة البيئات.
- **سهولة الاختبار**: خيار `skip_user_check` و `cache_results` و `collect_metadata` يسهل اختبار الوحدة وتحليل الأداء.
- **توثيق مدمج**: الدالة الجديدة تحتوي على توثيق شامل لجميع الخيارات والمعاملات.

---

## ملخص الإصدارات (1.2.1 و 1.2.2)

| الإصدار | أبرز الميزات |
|---------|---------------|
| **1.2.1** | إعادة هيكلة نظام التحقق من الصلاحيات بإضافة دوال `validate` المتخصصة، إضافة `disabled_operations` على مستوى النموذج والحقل، تحسين دالة `can()`، إضافة رموز خطأ جديدة، تحسين دوال `upload`، `deleteFile`، `attachTempFiles`، `getFiles`، وإضافة خيار `skip_permission_check`. |
| **1.2.2** | إنشاء تريت `HasFileUploadsMatchTempKey` بدالة `validateAndMatchTempKey` المتطورة، توحيد منطق التحقق من المفاتيح المؤقتة في جميع الدوال، إضافة خيارات متقدمة (وضع صارم، فترة سماح، تخزين مؤقت، إلخ)، وإضافة دالة `fireEventSafeInTrait` لتحسين توافق الأحداث. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `FileUploadService.php` بالنسخة المحدثة (الإصدار 1.2.2).
   - إضافة تريت `HasFileUploadsMatchTempKey` في المسار `classes/FileUploadService/HasFileUploadsMatchTempKey.php`.
   - تحديث `FileUploadRegistry.php` بالنسخة التي تحتوي على دوال `validate` الجديدة (الإصدار 1.2.1).
   - تحديث `FileUploadException.php` بالنسخة التي تحتوي على رموز الخطأ الجديدة.

2. **لا توجد هجرات جديدة**: كلا الإصدارين لا يتطلبان تغييرات في قاعدة البيانات.

3. **لا توجد تغييرات في الإعدادات**: يظل ملف `config.php` كما هو.

4. **اختبار التوافق**:
   - يُنصح بتشغيل اختبارات `FileUploadPlusTest` للتأكد من أن جميع العمليات تعمل بشكل صحيح.
   - التحقق من أن صلاحيات `edit` تعمل كما هو متوقع عند استبدال الملفات.
   - التأكد من أن المفاتيح المؤقتة غير الصالحة تسبب أخطاء مناسبة.

5. **مراجعة تعريفات النماذج** (اختياري):
   - إذا كنت تستخدم `disabled_operations`، تأكد من صياغتها بشكل صحيح في تعريفات الحقول.
   - إذا كنت تستخدم صلاحية `edit`، تأكد من إضافتها في مصفوفة `permissions`.

---

### الخاتمة

يمثل الإصداران **1.2.1** و **1.2.2** قفزة نوعية في نضج إضافة `Nano.FileUpload`. فبينما ركز الأول على إعادة هيكلة نظام التحقق من الصلاحيات ليكون أكثر تنظيماً ودقة، قدم الثاني آلية موحدة ومتقدمة للتحقق من مفاتيح الجلسة المؤقتة، مما جعل الكود أكثر نظافة وأماناً. مع هذه التحديثات، أصبحت الإضافة قادرة على التعامل مع سيناريوهات معقدة مثل استبدال الملفات، والتحقق من الصلاحيات متعددة المستويات، وإدارة الملفات المؤقتة بمرونة عالية، مع الحفاظ على توافق كامل مع بيئات Laravel و OctoberCMS.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)
