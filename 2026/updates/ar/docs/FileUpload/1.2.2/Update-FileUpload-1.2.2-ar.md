## 2026-04-12 – 2026-04-13

**تحديثات إضافة `Nano.FileUpload` – الإصدار 1.2.2**

### ملخص التحديثات

في إطار السعي المستمر لتعزيز أمان ومرونة نظام رفع الملفات، تم إصدار الإصدار 1.2.2 الذي يركز على توحيد وتبسيط عملية التحقق من مفاتيح الجلسة المؤقتة (temp session keys) عبر جميع دوال الخدمة. تم إنشاء تريت (Trait) جديد باسم `HasFileUploadsMatchTempKey` يحتوي على دالة متطورة `validateAndMatchTempKey` تجمع التحقق من صحة المفتاح، ومطابقة النموذج والحقل والمستخدم، مع خيارات متقدمة للتحكم في السلوك (الوضع الصارم، فترة السماح، التخزين المؤقت، إلخ). كما تمت إضافة دالة مساعدة `fireEventSafeInTrait` لضمان توافق إطلاق الأحداث مع كل من Laravel و OctoberCMS.

تم تحديث دوال `upload`، `deleteFile`، `attachTempFiles`، و `getFiles` في `FileUploadService` لاستخدام هذه الدالة الجديدة بدلاً من الكود المكرر، مما أدى إلى تقليل التعقيد وزيادة الأمان والتوحيد.

---

### أهداف الإصدار

- **توحيد منطق التحقق من المفاتيح المؤقتة**: إنشاء نقطة مركزية واحدة للتحقق من صحة المفاتيح ومطابقتها مع النموذج والحقل والمستخدم.
- **إزالة الكود المكرر**: استبدال عمليات التحقق اليدوية المتكررة في عدة دوال باستدعاء دالة واحدة.
- **تعزيز الأمان**: إضافة خيارات متقدمة مثل الوضع الصارم (strict mode)، التحقق من صلاحية المفتاح مع فترة سماح، ودعم إعادة المحاولة عند انتهاء الصلاحية.
- **تحسين التوافق**: ضمان إطلاق الأحداث بشكل صحيح في بيئات Laravel و OctoberCMS عبر دالة `fireEventSafeInTrait`.
- **تسهيل الاختبار**: توفير خيارات مثل `skip_user_check` و `cache_results` لتسهيل اختبار الوحدة.

---

### الميزات الجديدة

#### 1. تريت `HasFileUploadsMatchTempKey`

تم إنشاء التريت في المسار `Nano\FileUpload\Classes\FileUploadService\HasFileUploadsMatchTempKey` ويحتوي على:

##### أ. دالة `validateAndMatchTempKey`

هذه الدالة هي جوهر التحديث، وتقوم بالتحقق من صحة المفتاح المؤقت ومطابقته مع البيانات المتوقعة. توفر الدالة الخيارات التالية:

| الخيار | النوع | الوصف |
|--------|-------|-------|
| `throw_on_failure` | bool | إطلاق استثناء عند الفشل (true) أو إرجاع نتيجة خطأ كمصفوفة (false) |
| `skip_user_check` | bool | تخطي التحقق من المستخدم (للاستخدام الداخلي أو الاختبارات) |
| `strict_mode` | bool | وضع صارم: يتطلب تطابق النموذج والحقل بالكامل (true)، أو يسمح بعدم التطابق (false) |
| `stop_on_first_failure` | bool | إيقاف عند أول خطأ (true) أو تجميع الأخطاء (false) |
| `allow_expired_key` | bool | السماح بالمفاتيح منتهية الصلاحية ضمن فترة سماح |
| `expiry_grace_period` | int | فترة السماح بالثواني (افتراضي 300) |
| `custom_validator` | callable | دالة تحقق إضافية (تستقبل `$keyData` وتعيد bool) |
| `cache_results` | bool | تخزين نتائج التحقق مؤقتاً في نفس الطلب |
| `collect_metadata` | bool | جمع بيانات أداء (مدة التنفيذ، الخطوات المنفذة) |

**مثال على الاستدعاء الأساسي:**
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

- **قبل التحديث**: كان يتم قبول أي مفتاح مؤقت يتم تمريره من العميل دون التحقق من صحته أو مطابقته للنموذج والحقل.
- **بعد التحديث**: عند وجود `temp_session_key` في الخيارات، يتم استدعاء `validateAndMatchTempKey` للتحقق من أن المفتاح يخص نفس النموذج والحقل والمستخدم، وإلا يتم رمي استثناء مناسب.

**الكود الجديد:**
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

- **قبل التحديث**: كان يتم التحقق يدوياً من صحة المفتاح ثم مقارنة `userId` بشكل منفصل.
- **بعد التحديث**: استخدام `validateAndMatchTempKey` مع `strict_mode = false` (لأن الملف المؤقت قد لا يكون له `attachment_type`) لتوحيد المنطق.

**الكود الجديد:**
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

- **قبل التحديث**: كان يتم استدعاء `validateTempSessionKey` ثم مقارنة `modelClass` و `field` و `userId` و `userType` بشكل منفصل (أكثر من 20 سطراً من الكود).
- **بعد التحديث**: استبدال كل هذا المنطق باستدعاء واحد للدالة `validateAndMatchTempKey`.

**الكود الجديد:**
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

- **قبل التحديث**: كان يتم التحقق يدوياً من صحة المفتاح ثم مقارنة `userId` و `userType` و `modelClass` و `field` بشكل منفصل.
- **بعد التحديث**: استخدام `validateAndMatchTempKey` للتحقق من المفتاح قبل استخدامه في الاستعلام.

**الكود الجديد:**
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

تم استبدال جميع استدعاءات `Event::fire` المباشرة داخل `validateAndMatchTempKey` باستدعاءات `$this->fireEventSafeInTrait`، مما يضمن توافق الأحداث مع بيئات Laravel و OctoberCMS، ويتيح الاستفادة من دالة `fireEventSafe` الموجودة في `FileUploadService` إن وجدت.

---

### الفوائد

- **تقليل الكود المكرر**: إزالة أكثر من 50 سطراً من الكود المتكرر من الدوال المختلفة.
- **زيادة الأمان**: التحقق الموحد يضمن عدم وجود ثغرات ناتجة عن نسيان التحقق من مطابقة النموذج أو المستخدم في أي دالة.
- **مرونة عالية**: الخيارات المتعددة في `validateAndMatchTempKey` تسمح بتخصيص السلوك حسب الحالة (وضع صارم، فترة سماح، تخزين مؤقت، إلخ).
- **توافق محسن**: دالة `fireEventSafeInTrait` تضمن عمل الأحداث في كافة البيئات.
- **سهولة الاختبار**: خيار `skip_user_check` و `cache_results` و `collect_metadata` يسهل اختبار الوحدة وتحليل الأداء.
- **توثيق مدمج**: الدالة الجديدة تحتوي على توثيق شامل لجميع الخيارات.

---

### متطلبات الترقية

1. **تحديث الكود**: استبدال ملف `FileUploadService.php` بالنسخة المحدثة التي تستخدم التريت الجديد.
2. **إضافة التريت**: إنشاء ملف `HasFileUploadsMatchTempKey.php` في المسار `classes/FileUploadService/` بالمحتوى المطلوب.
3. **لا توجد هجرات جديدة**: هذا الإصدار لا يتطلب تغييرات في قاعدة البيانات.
4. **لا توجد تغييرات في الإعدادات**: يظل ملف `config.php` كما هو.
5. **اختبار التوافق**: يُنصح بتشغيل اختبارات `FileUploadPlusTest` للتأكد من أن جميع العمليات تعمل بشكل صحيح.

---

### الخاتمة

يمثل الإصدار 1.2.2 خطوة مهمة نحو توحيد وتبسيط منطق التحقق من المفاتيح المؤقتة في نظام رفع الملفات. من خلال إدخال تريت `HasFileUploadsMatchTempKey` ودالة `validateAndMatchTempKey`، أصبح من الممكن الآن تنفيذ تحقق آمن وشامل من المفاتيح المؤقتة في أي دالة تحتاج إليها، مع خيارات متقدمة تتيح التحكم الدقيق في السلوك. التحديثات التي تم إجراؤها على دوال `upload`, `deleteFile`, `attachTempFiles`, `getFiles` تجعل الكود أكثر نظافة وأقل عرضة للأخطاء، مع الحفاظ على التوافق الكامل مع الإصدارات السابقة.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)