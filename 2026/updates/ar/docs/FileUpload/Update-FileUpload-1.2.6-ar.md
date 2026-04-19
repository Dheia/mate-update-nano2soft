## 2026-04-18 – 2026-04-19

**تحديث إضافة `Nano.FileUpload` – الإصدار 1.2.6**

### إصلاح مشكلة الحذف المزدوج للملفات عند الرفع على علاقات `AttachOne`

---

### ملخص التحديثات

في الإصدار **1.2.6**، تم معالجة خطأ برمجي خطير كان يتسبب في حذف الملفات الجديدة فور رفعها على الحقول من نوع `AttachOne` (مثل صورة الغلاف، صورة الملف الشخصي، إلخ). المشكلة كانت ناتجة عن استدعاء مزدوج لعملية ربط الملف (`$relation->add()`) مما أدى إلى تنفيذ استعلام `DELETE` بعد `INSERT` مباشرة، واختفاء الملف من قاعدة البيانات على الرغم من نجاح عملية الرفع الظاهري.

الحل تمثل في:
1. إضافة خيارات تحكم دقيقة في دالة `Base64::onUpload` للتحكم في تعيين الحقول (`field`, `attachment_id`, `attachment_type`) وفي تنفيذ الربط (`skip_relation_add`).
2. تعديل `FileUploadService::upload` لاستخدام هذه الخيارات، بحيث يتم حفظ الملف بدون ربطه في `Base64`، ثم يتم ربطه مرة واحدة فقط بعد معالجة الملف وحذف الملف القديم (إن وجد).
3. إزالة الاستدعاء المزدوج لـ `add()` مما أزال استعلامات `DELETE` غير المرغوب فيها.

---

### أهداف الإصدار

- **إصلاح عيب برمجي خطير** كان يمنع رفع الملفات بشكل صحيح على علاقات `AttachOne` (خاصة في بيئات الاختبار ذات المعاملات المتداخلة).
- **منع الحذف غير المقصود للملفات الجديدة** الذي كان يحدث بعد الإدراج مباشرة.
- **تحسين استقرار وموثوقية نظام الرفع** خاصة للحقول التي تقبل ملفًا واحدًا فقط.
- **توفير تحكم أدق** في سلوك `Base64::onUpload` من خلال خيارات جديدة تسمح بتخطي تعيين حقول معينة أو تخطي عملية الربط بالكامل.
- **ضمان نجاح الاختبارات الآلية** المتعلقة برفع الملفات (`FILE_CREATE_001`, `FILE_UPDATE_001`) والتي كانت تفشل باستمرار بسبب هذا الخطأ.

---

### المشكلة الأصلية (قبل الإصلاح)

#### وصف السلوك الخاطئ

عند رفع ملف جديد لحقل من نوع `AttachOne` (مثل `document_front`)، كان النظام يقوم بالخطوات التالية:

1. **`Base64::onUpload`**: يحفظ الملف في جدول `system_files` **مع تعيين `attachment_id` و `attachment_type`** ثم يستدعي `$fileRelation->add($file)` لربطه بالنموذج.
2. **`FileUploadService::upload`**: بعد استدعاء `onUpload`، يقوم **مرة أخرى** باستدعاء `$relation->add($newFile)`.

**النتيجة:** الاستدعاء الثاني لـ `add()` كان يحذف جميع الملفات المرتبطة بالحقل (بما فيها الملف الجديد الذي تم ربطه للتو) ثم يعيد ربطه. لكن في بعض السيناريوهات (خاصة مع المعاملات المتداخلة)، كان استعلام `DELETE` يحذف السجل نهائيًا دون إعادة إدراجه بشكل صحيح، مما يؤدي إلى اختفاء الملف من قاعدة البيانات.

#### استعلامات SQL المرصودة (دليل على المشكلة)

```
INSERT INTO `system_files` (...) VALUES (...)  -- تم إدراج الملف الجديد (ID=7556)
DELETE FROM `system_files` WHERE `attachment_id` = ... AND `field` = ...  -- حذف الملف الجديد!
```

نتيجة هذا السلوك: `file_exists_in_db = false` وفشل اختبارات رفع الملفات.

---

### الحلول المطبقة في الإصدار 1.2.6

#### 1. إضافة خيارات تحكم جديدة في `Base64::onUpload`

أضفنا أربعة خيارات جديدة إلى مصفوفة `$options` في دالة `onUpload`:

| الخيار | النوع | القيمة الافتراضية | الوصف |
|--------|-------|-------------------|-------|
| `skip_set_field` | `bool` | `false` | إذا كان `true`، لا يتم تعيين `field` في نموذج `File`. |
| `skip_set_attachment_type` | `bool` | `false` | إذا كان `true`، لا يتم تعيين `attachment_type`. |
| `skip_set_attachment_id` | `bool` | `false` | إذا كان `true`، لا يتم تعيين `attachment_id`. |
| `skip_relation_add` | `bool` | `false` | إذا كان `true`، لا يتم استدعاء `$fileRelation->add()` (أي لا يتم ربط الملف). |

**الكود المضاف في `onUpload`:**

```php
$d_options = array_merge([
    // ... الخيارات السابقة
    'skip_set_field' => false,
    'skip_set_attachment_type' => false,
    'skip_set_attachment_id' => false,
    'skip_relation_add' => false,
], $options);

// ...

if (!$skip_set_field && $field) {
    $file->field = $field;
}
if (!$skip_set_attachment_type && $attachment_type) {
    $file->attachment_type = $attachment_type;
}
if (!$skip_set_attachment_id && $attachment_id) {
    $file->attachment_id = $attachment_id;
}

// ...

if (!$skip_relation_add && is_object($fileRelation)) {
    // ... تنفيذ الربط
}
```

#### 2. تعديل `FileUploadService::upload` لاستخدام الخيارات الجديدة

قمنا بتعديل خيارات `$uploadOptions` التي يتم تمريرها إلى `Base64::onUpload`:

```php
$uploadOptions = [
    // ... الخيارات السابقة
    'skip_set_field'           => false,
    'skip_set_attachment_type' => false,
    'skip_set_attachment_id'   => true,   // ⬅️ لا تعيّن attachment_id في Base64
    'skip_relation_add'        => true,   // ⬅️ لا تقم بالربط في Base64
];
```

ثم بعد استدعاء `onUpload` والحصول على `$newFile`، نقوم بالخطوات التالية **داخل `FileUploadService`**:

1. تعيين `field` إذا لم يكن موجوداً.
2. حساب `hash` و `meta` وحفظ التغييرات.
3. حذف الملف القديم (`$existingFile`) إذا كانت العملية `edit`.
4. **ربط الملف الجديد مرة واحدة فقط** باستخدام `$relation->add($newFile)`.

**الكود النهائي في `FileUploadService::upload`:**

```php
Db::transaction(function () use (...) {
    $uploadResult = Base64::onUpload($uploadOptions, $uploadedFile);
    $newFile = $uploadResult['model'];

    // تعيين الحقول الإضافية وحفظها
    if (!$newFile->field) {
        $newFile->field = $field;
    }
    // ... حساب hash و meta و disk ...

    $newFile->save();

    // حذف الملف القديم (إذا كانت العملية edit)
    if ($operation === 'edit' && $existingFile) {
        $existingFile->delete();
    }

    // ربط الملف الجديد مرة واحدة فقط
    if ($model && $model->exists && $model->hasRelation($field)) {
        $relation = $model->{$field}();
        if ($relation instanceof \October\Rain\Database\Relations\AttachOne) {
            $relation->add($newFile);
        }
    }
});
```

---

### النتائج بعد الإصلاح

- **اختفاء استعلام `DELETE` غير المرغوب فيه** بعد `INSERT`.
- **بقاء الملف في قاعدة البيانات** (`file_exists_in_db = true`).
- **نجاح اختبارات `FILE_CREATE_001` و `FILE_UPDATE_001`** بشكل موثوق.
- **عدم الحاجة إلى حلول التفافية في الاختبارات** (مثل `savepoint` أو التنظيف اليدوي).

---

### متطلبات الترقية (من 1.2.5 إلى 1.2.6)

1. **تحديث الكود**:
   - استبدال `Base64.php` بالنسخة التي تحتوي على الخيارات الجديدة (`skip_set_field`, `skip_set_attachment_type`, `skip_set_attachment_id`, `skip_relation_add`).
   - استبدال `FileUploadService.php` بالنسخة التي تستخدم هذه الخيارات وتنفذ الربط مرة واحدة فقط.

2. **لا توجد هجرات جديدة** – لا تغييرات في قاعدة البيانات.

3. **لا توجد تغييرات في الإعدادات** – يظل ملف `config.php` كما هو.

4. **لا توجد تغييرات في واجهة API** – جميع نقاط النهاية تعمل كما هي دون تعديل.

5. **اختبار التوافق**:
   - يُنصح بتشغيل اختبارات `KycPlusTest` (أو `FileUploadPlusTest`) للتأكد من نجاح عمليات الرفع والتحديث.
   - يجب أن تختفي أي أخطاء متعلقة بـ `FILE_CREATE_001` أو `FILE_UPDATE_001`.

---

### الفوائد والقيمة المضافة

- **إصلاح خطأ برمجي حرج** كان يؤثر على أي حقل `AttachOne` (مثل الصور الرمزية، صور الغلاف، الملفات الفردية).
- **زيادة موثوقية النظام** خاصة في بيئات الاختبار ذات المعاملات المتداخلة.
- **تحسين جودة الكود** بفصل مسؤوليات `Base64` (حفظ الملف فقط) عن `FileUploadService` (التحكم الكامل في دورة حياة الرفع والربط).
- **توفير خيارات تحكم دقيقة** يمكن استخدامها في سيناريوهات متقدمة أخرى.
- **تمهيد الطريق لإصدارات مستقبلية** قد تضيف ميزات مثل التحميل المسبق للملفات دون ربط فوري.

---

### الخاتمة

يمثل الإصدار **1.2.6** إصلاحًا جوهريًا لمشكلة كانت تؤثر سلبًا على استقرار نظام رفع الملفات ونتائج الاختبارات الآلية. من خلال إعادة هيكلة منطق الربط بين `Base64` و `FileUploadService` وإضافة خيارات تحكم جديدة، أصبحت عملية رفع الملفات للحقول الفردية (`AttachOne`) أكثر موثوقية وأقل عرضة للأخطاء. هذه التحسينات تعزز من قوة الإضافة وتجعلها مناسبة للاستخدام في بيئات الإنتاج والتطوير على حد سواء.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `Base64`](./docs/FileUpload/Docs-Base64-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)
