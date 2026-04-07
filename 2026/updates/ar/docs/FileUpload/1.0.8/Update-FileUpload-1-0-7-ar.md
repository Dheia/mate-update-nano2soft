## 2026-04-04 - 2026-04-05

**تحديثات إضافة `Nano.FileUpload` – الإصدارات 1.0.6 و 1.0.7**

### ملخص التحديثات

بعد إصدار الإصدار 1.0.5 الذي ركز على دعم تعدد اللغات، واصلنا تطوير الإضافة بإضافة ميزات متقدمة تهدف إلى زيادة المرونة والتوسع وتحسين إدارة الملفات على مستوى قاعدة البيانات. تضمنت الإصدارات 1.0.6 و 1.0.7:

- دعم التخزين المتعدد (multi‑storage) عبر أقراص مختلفة (S3, FTP, Local).
- التحويل التلقائي للصور (تحجيم تلقائي وعلامة مائية) مع إمكانية التحكم العام عبر الإعدادات.
- خطافات الأحداث (event hooks) قبل وبعد عمليات الرفع والحذف.
- دعم إشعارات WebSocket الاختيارية.
- إضافة أعمدة جديدة إلى جدول `system_files`: `disk`, `hash`, `meta`, `expires_at`.
- تعبئة تلقائية للحقول الجديدة (SHA256 للمحتوى، أبعاد الصورة، تاريخ انتهاء الملفات المؤقتة).
- تحسينات عامة في الأداء والأمان.

---

## الإصدار 1.0.6 – دعم التخزين المتعدد، التحويل التلقائي للصور، خطافات الأحداث، وتحسين قاعدة البيانات

### أهداف الإصدار

- تمكين رفع الملفات إلى خدمات تخزين متعددة (AWS S3، FTP، Local) عبر تحديد قرص التخزين على مستوى الحقل.
- إضافة إمكانية التحويل التلقائي للصور (تحجيم وعلامة مائية) أثناء الرفع.
- توفير خطافات (hooks) للأحداث لتوسيع السلوك دون تعديل الكود الأساسي.
- دعم إشعارات WebSocket للتحديثات الفورية في الواجهات الأمامية.
- إضافة أعمدة جديدة إلى جدول `system_files` لدعم الميزات المتقدمة (التخزين، التكرار، البيانات الإضافية، انتهاء الصلاحية).

### الميزات الجديدة

#### 1. دعم التخزين المتعدد (Multi‑Storage)

أضفنا إمكانية تحديد قرص تخزين مخصص لكل حقل عبر إعداد `storage_disk` في تعريف الحقل. هذا يتجاوز القرص الافتراضي المحدد في إعدادات النظام.

- **إضافة عمود `disk`** إلى جدول `system_files` لتخزين اسم القرص المستخدم.
- **تعديل نموذج `System\Models\File`** (عبر `Plugin::extendSystemFile()`) لدعم خاصية `disk` وتجاوز دالة `getDisk()` لاستخدام القرص المخصص.
- **إضافة دالة `getStorageDiskForField()`** في `FileUploadService` لاسترجاع اسم القرص من إعدادات الحقل والتحقق من وجوده.
- عند الرفع، يتم تعيين `$file->disk = $diskName` ثم حفظ الملف على القرص المحدد.

#### 2. التحويل التلقائي للصور (Auto‑resize & Auto‑watermark)

أضفنا إعدادات جديدة في تعريف الحقل:
- `auto_resize` (bool): تفعيل تحجيم الصورة تلقائياً.
- `resize_options` (array): خيارات التحجيم (العرض، الارتفاع، الوضع).
- `auto_watermark` (bool): تفعيل إضافة علامة مائية تلقائياً.
- `watermark_options` (array): خيارات العلامة المائية (الموضع، نسبة التحجيم).

أضفنا دالة `applyAutoProcessing()` في `FileUploadService` التي:
- تتحقق من وجود الملف كصورة.
- تستخدم `October\Rain\Resize\Resizer` لتحجيم الصورة وفق الخيارات.
- تستخدم إضافة `Nano2.Watermark` (إن كانت موجودة ومفعّلة) لإضافة العلامة المائية.

#### 3. خطافات الأحداث (Event Hooks)

أضفنا الأحداث التالية لتوسيع السلوك بسهولة:

| الحدث | مكان الاستدعاء | المعاملات |
|-------|----------------|------------|
| `nano.fileupload.beforeUpload` | قبل بدء معالجة الرفع | `$modelClass, $field, $fileData, &$options` |
| `nano.fileupload.afterUpload` | بعد نجاح الرفع وقبل إرجاع النتيجة | `$file, $modelClass, $field, $options` |
| `nano.fileupload.beforeDelete` | قبل حذف الملف | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterDelete` | بعد حذف الملف | `$fileId, $modelClass, $field` |

يمكن للمطورين الاستماع لهذه الأحداث لإضافة منطق مخصص (مثل إرسال إشعارات، تسجيل إضافي، تعديل البيانات).

#### 4. دعم WebSocket للإشعارات الفورية

أضفنا إعدادات `websocket` في `config.php`:
- `enabled`: تفعيل/تعطيل الإشعارات.
- `channel`: اسم القناة المستخدمة.

أضفنا دالة `notifyWebSocket()` في `FileUploadService` تطلق حدث `nano.fileupload.websocket.notify` بعد نجاح الرفع أو الحذف. يمكن للمطور استخدام مكتبة WebSocket (مثل Laravel WebSockets أو Pusher) لالتقاط هذا الحدث وإرسال الإشعارات.

#### 5. إضافة أعمدة جديدة إلى جدول `system_files`

قمنا بإنشاء هجرة `add_columns_to_system_files.php` لإضافة الأعمدة التالية:

| العمود | النوع | الغرض |
|--------|-------|-------|
| `disk` | `string, nullable` | تخزين اسم قرص التخزين المخصص (مثل `s3`, `ftp`). |
| `hash` | `string, nullable, index` | تخزين SHA256 للمحتوى لمنع التكرار. |
| `meta` | `text, nullable` | تخزين بيانات إضافية بصيغة JSON (أبعاد الصورة، مدة الصوت، إلخ). |
| `expires_at` | `dateTime, nullable` | تاريخ انتهاء صلاحية الملفات المؤقتة لتنظيفها تلقائياً. |

### الفوائد

- إمكانية تخزين الملفات في خدمات سحابية متعددة حسب الحاجة.
- تحسين تجربة المستخدم عبر تحجيم الصور تلقائياً وإضافة علامة مائية دون تدخل يدوي.
- توسيع النظام بسهولة عبر الأحداث دون تعديل الكود الأساسي.
- إشعارات فورية للمستخدمين عند اكتمال الرفع أو الحذف.
- إدارة أفضل للملفات المكررة والبيانات الوصفية والملفات المؤقتة.

---

## الإصدار 1.0.7 – إكمال التعبئة التلقائية لحقول `hash`, `meta`, `expires_at` وإضافة تحكم عالمي في التحويلات

### أهداف الإصدار

- إكمال تنفيذ تعبئة الحقول الجديدة (`hash`, `meta`, `expires_at`) تلقائياً أثناء الرفع.
- إضافة إعدادات عامة للتحكم في تفعيل/تعطيل التحويلات التلقائية (`auto_resize`, `auto_watermark`) على مستوى API بالكامل.
- تحسين الأداء والأمان عبر حساب التجزئة وتخزين أبعاد الصورة وتحديد صلاحية الملفات المؤقتة.

### الميزات الجديدة

#### 1. تعبئة تلقائية لـ `hash` (SHA256)

أضفنا في دالة `upload()` بعد الحصول على كائن الملف:
```php
if (!$file->hash) {
    $content = file_get_contents($file->getLocalPath());
    $file->hash = hash('sha256', $content);
    $is_changed = true;
}
```
هذا يضمن حساب تجزئة فريدة للمحتوى، مما يساعد في منع تخزين نسخ مكررة (يمكن التحقق من `hash` قبل الحفظ).

#### 2. تعبئة تلقائية لـ `meta` بأبعاد الصورة

أضفنا:
```php
if ($file->isImage()) {
    $dimensions = getimagesize($file->getLocalPath());
    $file->meta = array_merge($file->meta ?: [], [
        'width'  => $dimensions[0],
        'height' => $dimensions[1],
        'mime'   => $dimensions['mime'],
    ]);
    $is_changed = true;
}
```
يتم تخزين الأبعاد ونوع MIME في حقل `meta` كـ JSON، مما يسهل استرجاعها لاحقاً دون الحاجة لقراءة الملف مرة أخرى.

#### 3. تعيين `expires_at` للملفات المؤقتة

عند استخدام `temp_session_key` (أي الملف غير مرتبط بنموذج بعد)، نضبط تاريخ انتهاء صلاحية تلقائي:
```php
if ($tempSessionKey) {
    if (!$file->expires_at && (!$file->attachment_type || !$file->attachment_id)) {
        $file->expires_at = Carbon::now()->addHours(24);
        $is_changed = true;
    }
}
```
يمكن تشغيل مهمة مجدولة (cron) لحذف الملفات منتهية الصلاحية وغير المرتبطة تلقائياً.

#### 4. إضافة تحكم عام في التحويلات التلقائية

أضفنا إعدادين في قسم `general` بملف `config.php`:
```php
'disable_auto_resize'    => env('NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE', true),
'disable_auto_watermark' => env('NANO_FILE_UPLOAD_DISABLE_AUTO_WATERMARK', true),
```
ودالتين في `FileUploadRegistry`:
- `isAutoResizeEnabledGlobally()`
- `isAutoWatermarkEnabledGlobally()`

ثم عدلنا دالة `applyAutoProcessing()` لتحترم هذه الإعدادات:
```php
if ($this->registry->isAutoResizeEnabledGlobally() && $procOptions['auto_resize']) { ... }
if ($this->registry->isAutoWatermarkEnabledGlobally() && $procOptions['auto_watermark'] && ...) { ... }
```
هذا يسمح للمسؤول بتعطيل جميع عمليات التحجيم أو العلامات المائية مؤقتاً عبر متغيرات البيئة دون تعديل تعريفات الحقول.

#### 5. تحسينات أخرى

- استخدام `Carbon::now()` بدلاً من `now()` لضمان التوافق.
- تصحيح اسم المتغير `$is_changed` (بدلاً من `$is_chage`).
- ضمان حفظ الملف مرة أخرى عند تغيير أي من الحقول (`hash`, `meta`, `expires_at`).

### الفوائد

- إدارة أفضل للملفات المكررة عبر التحقق من `hash`.
- تخزين بيانات وصفية مفيدة (أبعاد الصورة) لاستخدامها في الواجهات.
- تنظيف تلقائي للملفات المؤقتة عبر `expires_at`.
- تحكم مركزي في تفعيل/تعطيل التحويلات التلقائية.

---

## ملخص الإصدارات (1.0.6 و 1.0.7)

| الإصدار | أبرز الميزات |
|---------|---------------|
| 1.0.6 | دعم التخزين المتعدد (disk)، التحويل التلقائي للصور، خطافات الأحداث، WebSocket، إضافة أعمدة جديدة في قاعدة البيانات. |
| 1.0.7 | تعبئة تلقائية لـ `hash`, `meta`, `expires_at`، تحكم عام في التحويلات التلقائية، تحسينات في الأداء والأمان. |

---

## متطلبات الترقية

1. **تشغيل الهجرة**:
   ```bash
   php artisan plugin:refresh Nano.FileUpload
   ```
   أو تنفيذ الهجرة الجديدة `add_columns_to_system_files.php`.

2. **إضافة متغيرات البيئة** (اختياري) في ملف `.env`:
   ```ini
   # تعطيل التحويلات التلقائية (القيم الافتراضية: true)
   NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE=true
   NANO_FILE_UPLOAD_DISABLE_AUTO_WATERMARK=true

   # إعدادات WebSocket
   NANO_FILE_UPLOAD_WEBSOCKET_ENABLED=false
   NANO_FILE_UPLOAD_WEBSOCKET_CHANNEL=file-uploads

   # إعدادات التخزين المتعدد (على مستوى الحقل، وليس عالمياً)
   # (يتم تعيينها في تعريفات الحقول مباشرة)
   ```

3. **تحديث تعريفات الحقول** لإضافة `storage_disk`, `auto_resize`, `auto_watermark` حسب الحاجة.

4. **الاستماع للأحداث** لتوسيع السلوك:
   ```php
   Event::listen('nano.fileupload.afterUpload', function ($file, $modelClass, $field, $options) {
       // إرسال إشعار عبر WebSocket
   });
   ```

---

## الخاتمة

بفضل الإصدارين 1.0.6 و 1.0.7، أصبحت إضافة `Nano.FileUpload` أكثر قوة ومرونة من أي وقت مضى. يمكنها الآن:

- التعامل مع أقراص تخزين متعددة (S3، FTP، Local).
- تحويل الصور تلقائياً (تحجيم، علامة مائية).
- إطلاق أحداث مخصصة لتوسيع السلوك.
- إرسال إشعارات فورية عبر WebSocket.
- تخزين بيانات وصفية غنية (أبعاد الصورة، التجزئة).
- تنظيف الملفات المؤقتة تلقائياً.

هذه الميزات تجعل الإضافة جاهزة للاستخدام في المشاريع الكبيرة والمعقدة التي تتطلب أداءً عالياً وأماناً متقدماً.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)
