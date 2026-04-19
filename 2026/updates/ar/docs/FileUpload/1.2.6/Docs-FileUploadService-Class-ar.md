## توثيق كلاس `FileUploadService` – الإصدار 1.2.3

**نبذة تعريفية عن حزمة `Nano.FileUpload` ووحدة إدارة رفع الملفات**

هذه المجموعة من الكلاسات (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager` وغيرها) هي جزء من **حزمة `Nano.FileUpload`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **نانوسوفت** بهدف توحيد وإدارة عمليات رفع الملفات عبر واجهات برمجة التطبيقات (API) بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano\FileUpload\Classes
```

توفر هذه الوحدة أدوات متقدمة لتسجيل النماذج (الموديولات) التي تحتوي على حقول رفع ملفات، والتحقق من صلاحيات المستخدمين (بما في ذلك أنواع المستخدمين المختلفة: `backend` و `frontend`)، وإدارة الملفات المرفوعة مؤقتًا قبل حفظ السجل، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة رفع ملفات قابلة للتوسع بسهولة وتلبية احتياجات التطبيقات المتقدمة دون المساس بأمان التطبيق أو أدائه.

> **ملاحظة:** يغطي هذا المستند الإصدارات حتى 1.2.3، ويشمل الميزات المتقدمة مثل التخزين المتعدد، التحويل التلقائي للصور، خطافات الأحداث، WebSocket، تحسينات الأمان، دعم عملية `edit` (استبدال الملفات)، واستخدام المعاملات (`Db::transaction`) لضمان التكاملية، بالإضافة إلى توحيد منطق التحقق من المفاتيح المؤقتة عبر تريت `HasFileUploadsMatchTempKey`.

---

### مقدمة

كلاس `FileUploadService` هو **طبقة الخدمة (Service Layer)** المسؤولة عن تنفيذ عمليات رفع الملفات وحذفها واسترجاعها في تطبيقات نانوسوفت. يعتمد هذا الكلاس على `FileUploadRegistry` للوصول إلى إعدادات النماذج والتحقق من الصلاحيات، وعلى `FileUploadUserManager` لإدارة المستخدم الحالي والتحقق من صلاحياته.

ببساطة، `FileUploadService` هو **المحرك الذي ينفذ عمليات الرفع الفعلية**، ويتأكد من أن كل عملية تتم وفقًا للإعدادات المسجلة والصلاحيات المحددة، مع دعم الرفع المؤقت (عبر مفتاح جلسة مؤقت) لربط الملفات بالنماذج غير المحفوظة بعد. بدءاً من الإصدار 1.1.0، يدعم الكلاس أيضاً عملية **استبدال الملفات (edit)** في علاقات `attachOne` باستخدام المعاملات (`Db::transaction`) لضمان عدم فقدان البيانات. وفي الإصدار 1.2.2، تم توحيد منطق التحقق من المفاتيح المؤقتة عبر تريت `HasFileUploadsMatchTempKey`، مما يزيد الأمان ويقلل تكرار الكود.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$registry` | `FileUploadRegistry` | مرجع إلى سجل النماذج المسجلة. |
| `$userManager` | `FileUploadUserManager` | مرجع إلى مدير المستخدمين والصلاحيات. |
| `BLACKLISTED_EXTENSIONS` | `array` | قائمة الامتدادات الخطيرة الممنوعة نهائياً (PHP, JS, HTML, EXE, إلخ). |

---

### أهم الطرق (Methods)

#### 1. إنشاء مفتاح جلسة مؤقت

##### `public function generateTempSessionKey(string $modelClass, string $field, $userId = null, $userType = null): string`
- **الهدف**: توليد مفتاح جلسة مؤقت فريد وآمن لربط الملفات قبل حفظ النموذج.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$userId`: (اختياري) معرف المستخدم، إذا لم يُمرر يستخدم المستخدم الحالي.
  - `$userType`: (اختياري) نوع المستخدم (`backend`/`frontend`/`guest`).
- **السلوك**:
  - يحصل على معرف المستخدم الحالي ونوعه أو يستخدم القيم الممررة.
  - يُنشئ مفتاحًا بتنسيق `tmp_{base64(data)}:{hash}` حيث `data = modelClass|field|userId|userType|timestamp` ويُوقع بـ HMAC-SHA256 باستخدام `app.key`.
- **الإرجاع**: مفتاح الجلسة المؤقت.
- **مثال**:
  ```php
  $tempKey = $service->generateTempSessionKey('Nano\Shop\Models\Product', 'image');
  ```

#### 2. التحقق من صحة مفتاح جلسة مؤقت (الأساسي)

##### `public function validateTempSessionKey(string $tempSessionKey): ?array`
- **الهدف**: التحقق من صحة مفتاح الجلسة المؤقت (التوقيع، الانتهاء، البنية) واستخراج بياناته.
- **المعاملات**: `$tempSessionKey` – المفتاح المراد التحقق منه.
- **الإرجاع**: مصفوفة تحتوي على `modelClass`, `field`, `userId`, `userType`, `timestamp`، أو `null` إذا كان غير صالح.
- **السلوك**:
  - يتحقق من البداية `tmp_`.
  - يفك الترميز ويتحقق من التوقيع.
  - يتحقق من صلاحية المفتاح (الافتراضي 3600 ثانية، قابل للتعديل عبر `security.temp_key_ttl`).

#### 3. دالة متقدمة للتحقق من المفتاح المؤقت ومطابقته (من التريت)

##### `public function validateAndMatchTempKey($tempKey, string $modelClass, string $field, $user = null, array $options = []): array`
- **الهدف**: دالة متكاملة تجمع التحقق من صحة المفتاح ومطابقته مع النموذج والحقل والمستخدم، مع خيارات متقدمة للتحكم في السلوك.
- **المعاملات**:
  - `$tempKey`: مفتاح الجلسة المؤقت (نص) أو بياناته (مصفوفة).
  - `$modelClass`: اسم النموذج المتوقع.
  - `$field`: اسم الحقل المتوقع.
  - `$user`: المستخدم الحالي (اختياري، يُستخدم للحصول على userId و userType).
  - `$options`: مصفوفة خيارات متقدمة (انظر الجدول أدناه).
- **الخيارات المتقدمة**:
  | الخيار | النوع | الوصف |
  |--------|-------|-------|
  | `throw_on_failure` | bool | إطلاق استثناء عند الفشل (`true`) أو إرجاع نتيجة خطأ كمصفوفة (`false`). |
  | `skip_user_check` | bool | تخطي التحقق من المستخدم. |
  | `strict_mode` | bool | وضع صارم: يتطلب تطابق النموذج والحقل بالكامل. |
  | `stop_on_first_failure` | bool | إيقاف عند أول خطأ (`true`) أو تجميع الأخطاء. |
  | `allow_expired_key` | bool | السماح بالمفاتيح منتهية الصلاحية ضمن فترة سماح. |
  | `expiry_grace_period` | int | فترة السماح بالثواني (افتراضي 300). |
  | `custom_validator` | callable | دالة تحقق إضافية. |
  | `cache_results` | bool | تخزين نتائج التحقق مؤقتاً في نفس الطلب. |
  | `collect_metadata` | bool | جمع بيانات أداء. |
- **الإرجاع**: مصفوفة تحتوي على `status`, `message`, `data` (بيانات المفتاح ونتائج المطابقة), `process_data`, `error_code`, إلخ.
- **الاستخدام الداخلي**: تستخدمها دوال `upload`, `deleteFile`, `attachTempFiles`, `getFiles` للتحقق من المفاتيح المؤقتة المقدمة من العميل.
- **مثال**:
  ```php
  $result = $service->validateAndMatchTempKey($tempKey, 'Product', 'image', $user, [
      'throw_on_failure' => false,
      'strict_mode' => true,
  ]);
  if ($result['status']) {
      echo "المفتاح صالح ويطابق النموذج والحقل والمستخدم";
  } else {
      echo "فشل التحقق: " . $result['message'];
  }
  ```

> **ملاحظة:** هذه الدالة جزء من تريت `HasFileUploadsMatchTempKey` الذي يستخدمه `FileUploadService`. تم إضافتها في الإصدار 1.2.2.

#### 4. رفع ملف واحد

##### `public function upload(string $modelClass, string $field, $fileData, array $options = []): array`
- **الهدف**: رفع ملف واحد لحقل معين، مع دعم عمليات `add` (رفع جديد) و `edit` (استبدال ملف موجود في علاقة `attachOne`).
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$fileData`: بيانات الملف (كائن `UploadedFile` أو سلسلة base64).
  - `$options`: مصفوفة خيارات إضافية:
    - `model`: كائن النموذج (إذا كان موجودًا ومحفوظًا).
    - `temp_session_key`: مفتاح جلسة مؤقت (لتجاوز التوليد التلقائي). **منذ الإصدار 1.2.2، يتم التحقق من صحة هذا المفتاح ومطابقته باستخدام `validateAndMatchTempKey` قبل استخدامه.**
    - `skip_permission_check`: `bool` لتجاوز التحقق من الصلاحية (مفيد للاختبارات).
    - `title`, `description`: لتحديد عنوان ووصف الملف.
    - `...`: أي خيارات إضافية تُمرر إلى `Base64::onUpload`.
- **السلوك**:
  - يتحقق من تسجيل النموذج والحقل في `FileUploadRegistry`.
  - **يحدد العملية**:
    - إذا كان `options['model']` موجودًا ومحفوظًا (`exists`) وله علاقة `attachOne` بالحقل وتم العثور على ملف مرتبط، تصبح العملية `edit` (استبدال).
    - وإلا تصبح العملية `add` (رفع جديد).
  - **إذا تم تمرير `temp_session_key`**، يتم استدعاء `validateAndMatchTempKey` للتحقق من أن المفتاح يخص نفس النموذج والحقل والمستخدم. إذا فشل التحقق، يتم رمي استثناء.
  - يتحقق من صلاحية العملية (`add` أو `edit`) للمستخدم الحالي (مع مراعاة الإعدادات العامة مثل `disable_upload`, `disable_edit`).
  - يطلق حدث `nano.fileupload.beforeUpload` (مع تمرير العملية).
  - يتحقق من صحة الملف (القائمة السوداء، الحجم، الأنواع) عبر `validateFile`.
  - **يستخدم معاملة قاعدة البيانات (`Db::transaction`)** لضمان التكاملية:
    - رفع الملف الجديد وحفظه (بما في ذلك تعيين `session_key`, `expires_at`, `hash`, `meta`, `disk`).
    - **إذا كانت العملية `edit` ونجح رفع الجديد**:
      - يطلق حدث `nano.fileupload.beforeEditDelete`.
      - يحذف الملف القديم.
      - يطلق حدث `nano.fileupload.afterEditDelete`.
    - يضيف الملف الجديد إلى علاقة النموذج (إذا كان موجوداً).
  - يطلق حدث `nano.fileupload.afterUpload` ويرسل إشعار WebSocket (إذا مفعل).
- **الإرجاع**: مصفوفة بنفس هيكل `Base64::onUpload` مع حقول إضافية: `code`, `status`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`, `error_code`. وتحتوي `process_data` على `operation` (`add`/`edit`) و `existing_file_deleted` (في حالة `edit`).
- **مثال**:
  ```php
  // رفع جديد (add) مع مفتاح مؤقت (سيتم التحقق منه)
  $result = $service->upload('Product', 'image', $file, ['temp_session_key' => $tempKey]);
  
  // استبدال ملف (edit)
  $product = Product::find(10);
  $result = $service->upload('Product', 'image', $newFile, ['model' => $product]);
  
  if ($result['status']) {
      echo "تم رفع الملف بنجاح. العملية: " . $result['process_data']['operation'];
  } else {
      echo "فشل: " . $result['error'] . " (كود: " . $result['error_code'] . ")";
  }
  ```

#### 5. رفع عدة ملفات (لحقل متعدد)

##### `public function uploadMultiple(string $modelClass, string $field, array $filesData, array $options = []): array`
- **الهدف**: رفع عدة ملفات لحقل متعدد في طلب واحد.
- **المعاملات**: نفس `upload`، لكن `$filesData` مصفوفة من بيانات الملفات.
- **السلوك**: يتكرر على `$filesData` ويستدعي `upload()` لكل ملف، ثم يجمع النتائج ويحسب عدد النجاحات.
- **الإرجاع**: مصفوفة تحتوي على `code`, `status`, `message`, `data` (نتائج كل ملف), `process_data` (عدد النجاحات/الإجمالي).
- **مثال**:
  ```php
  $files = [ $file1, $file2, $file3 ];
  $result = $service->uploadMultiple('Product', 'gallery', $files, ['model' => $product]);
  echo "نجح رفع " . $result['process_data']['success_count'] . " من " . $result['process_data']['total'];
  ```

#### 6. حذف ملف

##### `public function deleteFile(int $fileId, ?string $modelClass = null, ?string $field = null, $user = null): array`
- **الهدف**: حذف ملف محدد مع التحقق من الصلاحية.
- **المعاملات**:
  - `$fileId`: معرف الملف.
  - `$modelClass`: (اختياري) اسم الفئة للتحقق من صلاحية الحذف.
  - `$field`: (اختياري) اسم الحقل للتحقق من صلاحية الحذف.
  - `$user`: (اختياري) كائن المستخدم.
- **السلوك**:
  - يطلق حدث `nano.fileupload.beforeDelete`.
  - يبحث عن الملف باستخدام `File::find()`.
  - يتحقق من الصلاحية بناءً على ثلاث حالات:
    1. تم تمرير `modelClass` و `field`: يتحقق من صلاحية `delete` للمستخدم الحالي (مع مراعاة `disable_delete`).
    2. الملف مرتبط بنموذج محفوظ: يستخرج `modelClass` و `field` من `attachment_type` و `field` في سجل الملف.
    3. الملف مؤقت (`session_key` موجود): **يستخدم `validateAndMatchTempKey` للتحقق من صحة المفتاح وأن المستخدم الحالي هو من أنشأه (مع `strict_mode = false` لأن `attachment_type` قد يكون null).**
  - إذا كان المستخدم غير مصرح له، يرمي استثناء.
  - يحذف الملف.
  - يطلق حدث `nano.fileupload.afterDelete` ويرسل إشعار WebSocket.
- **الإرجاع**: مصفوفة تحتوي على `code`, `status`, `message`, `error`, `error_code`, إلخ.
- **مثال**:
  ```php
  $result = $service->deleteFile(123, 'Nano\Shop\Models\Product', 'image');
  if ($result['status']) {
      echo "تم حذف الملف بنجاح";
  }
  ```

#### 7. جلب الملفات

##### `public function getFiles(string $modelClass, string $field, ?int $modelId = null, array $options = []): array`
- **الهدف**: استرجاع الملفات المرتبطة بنموذج وحقل معين أو ملفات مؤقتة.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$modelId`: (اختياري) معرف النموذج المحفوظ.
  - `$options`: خيارات إضافية:
    - `temp_session_key`: مفتاح جلسة مؤقت (لجلب الملفات غير المرتبطة). **منذ الإصدار 1.2.2، يتم التحقق من صحة هذا المفتاح ومطابقته باستخدام `validateAndMatchTempKey` قبل استخدامه.**
    - `with_thumbs`: `bool` لتضمين الصور المصغرة.
    - `thumb_sizes`: مصفوفة بأحجام الصور المصغرة (مثل `['thumb' => [100,100]]`).
- **السلوك**:
  - يتحقق من تسجيل الحقل وصلاحية `view` (مع مراعاة `disable_get`).
  - يبني استعلامًا على `File` مع تصفية حسب `field` و (`attachment_id` + `attachment_type` إذا كان `modelId` موجودًا) أو `session_key` إذا كان `temp_session_key` موجودًا.
  - **عند استخدام `temp_session_key`**، يستدعي `validateAndMatchTempKey` للتحقق من صحة المفتاح ومطابقته (النموذج، الحقل، المستخدم) قبل تنفيذ الاستعلام. إذا فشل التحقق، يتم رمي استثناء.
  - يعيد الملفات مع البيانات الأساسية (id, title, description, path, size, content_type) وصور مصغرة إذا طُلب.
- **الإرجاع**: مصفوفة تحتوي على `code`, `status`, `message`, `data`.
- **مثال**:
  ```php
  $files = $service->getFiles('Product', 'image', 456, ['with_thumbs' => true]);
  foreach ($files['data'] as $file) {
      echo "<img src='{$file['thumb']}'>";
  }
  ```

#### 8. ربط الملفات المؤقتة بنموذج محفوظ

##### `public function attachTempFiles(Model $model, string $field, string $tempSessionKey, array $options = []): array`
- **الهدف**: نقل الملفات التي تم رفعها باستخدام مفتاح جلسة مؤقت إلى نموذج محفوظ.
- **المعاملات**:
  - `$model`: كائن النموذج المحفوظ.
  - `$field`: اسم الحقل.
  - `$tempSessionKey`: مفتاح الجلسة المؤقت المستخدم أثناء الرفع.
  - `$options`: خيارات إضافية (مثل `skip_permission_check` للاختبارات).
- **السلوك**:
  - **يستخدم `validateAndMatchTempKey`** للتحقق من صحة المفتاح ومطابقته مع النموذج والحقل والمستخدم في خطوة واحدة متكاملة (بدلاً من الكود اليدوي المكرر سابقاً). إذا فشل التحقق، يتم رمي استثناء.
  - يتحقق من صلاحية `edit` (لأن الربط يعتبر تعديلاً على النموذج) ما لم يتم تجاوزها بـ `skip_permission_check`.
  - يبحث عن الملفات التي تحمل `session_key = $tempSessionKey` و `field = $field`.
  - لكل ملف، يحدّث `attachment_id` و `attachment_type` لربطه بالنموذج ويزيل `session_key` و `expires_at`.
  - إذا كان للنموذج علاقة بهذا الحقل، يستخدم `add()`.
  - يطلق حدث `nano.fileupload.afterAttach` ويرسل إشعار WebSocket.
- **الإرجاع**: مصفوفة موحدة تحتوي على `code`, `status`, `message`, `data` (الملفات المرتبطة), `process_data` (عدد الملفات المرتبطة, إلخ).
- **مثال**:
  ```php
  $product = new Product();
  $product->name = 'منتج جديد';
  $product->save();
  $result = $service->attachTempFiles($product, 'image', $tempKey);
  if ($result['status']) {
      echo "تم ربط " . $result['process_data']['attached_count'] . " ملفات";
  }
  ```

#### 9. التحقق من صحة الملف

##### `public function validateFile(string $modelClass, string $field, $fileData): void`
- **الهدف**: التحقق من أن الملف يطابق قيود الحقل المسجل (الحجم الأقصى، الأنواع المسموحة) بالإضافة إلى القائمة السوداء.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$fileData`: بيانات الملف (كائن `UploadedFile` أو سلسلة base64).
- **السلوك**:
  - يستدعي `getFieldConstraints()` من `FileUploadRegistry`.
  - **يتحقق من القائمة السوداء** أولاً (امتدادات خطيرة).
  - يتحقق من حجم الملف (يقارن مع `max_filesize`).
  - يتحقق من امتداد الملف مقابل `allowed_types`.
- **الإرجاع**: لا شيء، لكنه يرمي `FileUploadException` مع كود خطأ مناسب إذا فشل التحقق، ويسجل المحاولة الفاشلة في `fileupload.log`.
- **مثال**:
  ```php
  try {
      $service->validateFile('Product', 'image', $uploadedFile);
      // الملف صالح
  } catch (FileUploadException $e) {
      echo "الملف غير صالح: " . $e->getMessage() . " (كود: " . $e->getErrorCode() . ")";
  }
  ```

#### 10. دوال مساعدة (التخزين والتحويل والإشعارات)

##### `protected function getStorageDiskForField($modelClass, $field): ?string`
- يحدد اسم قرص التخزين المخصص للحقل (من `getProcessingOptions`) ويتحقق من وجوده.

##### `protected function applyAutoProcessing($file, $modelClass, $field): void`
- يطبق التحجيم التلقائي والعلامة المائية على الصور وفق خيارات الحقل والإعدادات العامة (`disable_auto_resize`, `disable_auto_watermark`).

##### `protected function notifyWebSocket($event, $data): void`
- يطلق حدث `nano.fileupload.websocket.notify` إذا كان WebSocket مفعلاً في الإعدادات.

##### `protected function fireEventSafe(string $eventName, $params = [], $halt = false)`
- إطلاق حدث بشكل آمن متوافق مع Laravel و OctoberCMS. تستخدمه دوال الخدمة والأحداث داخل التريت `HasFileUploadsMatchTempKey` عبر `fireEventSafeInTrait`.

---

### الأحداث (Events)

| اسم الحدث | مكان الاستدعاء | المعاملات |
|-----------|----------------|------------|
| `nano.fileupload.beforeUpload` | بداية `upload` (قبل المعاملة) | `$modelClass, $field, $fileData, &$options, $operation` |
| `nano.fileupload.afterUpload` | نهاية `upload` (بعد النجاح) | `$file, $modelClass, $field, $options, $operation` |
| `nano.fileupload.beforeEditDelete` | أثناء `upload` (عملية `edit`، قبل حذف القديم) | `$existingFile, $modelClass, $field, $model` |
| `nano.fileupload.afterEditDelete` | أثناء `upload` (عملية `edit`، بعد حذف القديم) | `$existingFile, $modelClass, $field, $model` |
| `nano.fileupload.beforeDelete` | بداية `deleteFile` | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterDelete` | نهاية `deleteFile` (بعد الحذف) | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterAttach` | نهاية `attachTempFiles` (بعد الربط) | `$model, $field, $attachedFiles, $tempSessionKey` |
| `nano.fileupload.websocket.notify` | بعد `upload`, `deleteFile`, `attachTempFiles` (إذا مفعل WebSocket) | `$channel, $event, $data` |

يمكن للمطورين الاستماع لهذه الأحداث لتوسيع السلوك (مثل إرسال إشعارات، تسجيل إضافي، تعديل البيانات).

---

### أمثلة عملية شاملة

#### مثال 1: رفع صورة رئيسية لمنتج جديد باستخدام مفتاح جلسة مؤقت (مع تحجيم تلقائي وعلامة مائية)

```php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadUserManager;
use Nano\Shop\Models\Product;

$service = FileUploadService::instance();
$userManager = FileUploadUserManager::instance();

$modelClass = Product::class;
$field = 'image';

// 1. إنشاء مفتاح جلسة مؤقت (يتضمن userType و timestamp)
$tempKey = $service->generateTempSessionKey($modelClass, $field, $userManager->getId(), $userManager->getUserType());

// 2. رفع الصورة (ستُطبق التحويلات التلقائية حسب إعدادات الحقل)
$uploadResult = $service->upload($modelClass, $field, $uploadedFile, [
    'temp_session_key' => $tempKey,
    'title' => 'صورة المنتج',
    'description' => 'وصف الصورة',
]);

if (!$uploadResult['status']) {
    die("فشل الرفع: " . $uploadResult['error'] . " (كود: " . $uploadResult['error_code'] . ")");
}

// 3. إنشاء المنتج وحفظه
$product = new Product();
$product->name = 'منتج جديد';
$product->save();

// 4. ربط الصورة بالمنتج (الخدمة تتحقق من المفتاح والمستخدم عبر validateAndMatchTempKey)
$attachResult = $service->attachTempFiles($product, $field, $tempKey);
if (!$attachResult['status']) {
    die("فشل الربط: " . $attachResult['error']);
}

echo "تم رفع الصورة وربطها بالمنتج رقم {$product->id}";
```

#### مثال 2: استبدال صورة منتج موجود (عملية `edit`)

```php
$product = Product::find(10); // منتج موجود
$newImageFile = $request->file('new_image');

$result = $service->upload(Product::class, 'image', $newImageFile, [
    'model' => $product, // تمرير النموذج المحفوظ
]);

if ($result['status']) {
    echo "تم استبدال الصورة بنجاح. العملية: " . $result['process_data']['operation']; // edit
    echo "تم حذف الملف القديم رقم: " . $result['process_data']['existing_file_deleted'];
} else {
    echo "فشل الاستبدال: " . $result['error'];
}
```

#### مثال 3: رفع عدة صور لمعرض منتج موجود (مع WebSocket)

```php
$product = Product::find(10); // منتج موجود

// رفع مجموعة صور
$filesData = [$file1, $file2, $file3];
$result = $service->uploadMultiple(Product::class, 'gallery', $filesData, [
    'model' => $product, // ربط مباشر
]);

if ($result['status']) {
    echo "تم رفع " . $result['process_data']['success_count'] . " صور بنجاح";
    // سيتم إطلاق حدث afterUpload لكل ملف، ويمكن للـ WebSocket إشعار الواجهة
} else {
    echo "فشل رفع بعض الصور: " . $result['error'];
}
```

#### مثال 4: جلب جميع صور معرض منتج مع صور مصغرة مخصصة

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'small' => [150, 150, 'crop'],
        'medium' => [300, 300, 'crop'],
    ],
]);

foreach ($result['data'] as $image) {
    echo "<img src='{$image['small']}' alt='{$image['title']}'>";
}
```

#### مثال 5: حذف صورة رئيسية بعد التحقق من الصلاحية (مع الأحداث)

```php
Event::listen('nano.fileupload.beforeDelete', function ($fileId, $modelClass, $field) {
    \Log::info("محاولة حذف ملف {$fileId}");
});

$result = $service->deleteFile(123, Product::class, 'image');

if ($result['status']) {
    echo "تم حذف الصورة بنجاح";
} else {
    echo "خطأ: " . $result['error'] . " (كود: " . $result['error_code'] . ")";
}
```

#### مثال 6: استخدام `validateAndMatchTempKey` مباشرة للتحقق من مفتاح مؤقت (على سبيل المثال في واجهة API مخصصة)

```php
$tempKey = $request->input('temp_key');
$user = auth()->user();

$result = $service->validateAndMatchTempKey($tempKey, 'Product', 'image', $user, [
    'throw_on_failure' => false, // نريد استجابة JSON بدلاً من استثناء
    'strict_mode' => true,
]);

if ($result['status']) {
    return response()->json(['valid' => true, 'data' => $result['data']]);
} else {
    return response()->json(['valid' => false, 'error' => $result['message']], $result['code']);
}
```

#### مثال 7: استخدام حدث `afterEditDelete` لتسجيل عملية الاستبدال

```php
Event::listen('nano.fileupload.afterEditDelete', function ($oldFile, $modelClass, $field, $model) {
    \Log::info("تم استبدال الملف القديم {$oldFile->id} بملف جديد للموديل {$model->id}");
});
```

#### مثال 8: دمج `FileUploadService` مع واجهة برمجة تطبيقات (API) باستخدام أكواد الأخطاء

في متحكم API:

```php
public function uploadImage(Request $request)
{
    $modelClass = $request->input('model_class');
    $field = $request->input('field');
    $file = $request->file('file');
    $modelId = $request->input('model_id');
    
    $service = FileUploadService::instance();
    
    $options = [];
    if ($modelId) {
        $model = $modelClass::find($modelId);
        if (!$model) {
            return response()->json(['error' => 'Model not found'], 404);
        }
        $options['model'] = $model;
    } else {
        $tempKey = $service->generateTempSessionKey($modelClass, $field);
        $options['temp_session_key'] = $tempKey;
    }
    
    $result = $service->upload($modelClass, $field, $file, $options);
    
    if ($result['status']) {
        return response()->json($result, 200);
    } else {
        return response()->json([
            'error' => $result['error'],
            'error_code' => $result['error_code'],
            'message' => $result['message']
        ], $result['code']);
    }
}
```

---

### التفاعل مع الكلاسات الأخرى

- **مع `FileUploadRegistry`**: يستخدم `$registry` للحصول على إعدادات الحقول (`getFieldConfig`, `getFieldConstraints`, `getProcessingOptions`) والتحقق من الصلاحيات (`can`, `validate`).
- **مع `FileUploadUserManager`**: يستخدم `$userManager` للحصول على المستخدم الحالي ونوعه (`getUser`, `getUserType`, `getId`).
- **مع `Base64` (المنقول إلى داخل الإضافة)**: يستدعي `Base64::onUpload`, `Base64::is_base64`, `Base64::base64ToUploadedFile`, `Base64::getBase64ImageSize`, `Base64::getDataInBase64`.
- **مع تريت `HasFileUploadsMatchTempKey`**: يستخدم الكلاس هذا التريت للاستفادة من دالة `validateAndMatchTempKey` و `fireEventSafeInTrait`. هذه الدوال متاحة كأنها جزء من `FileUploadService`.
- **مع `October\Rain\Resize\Resizer`**: لتحجيم الصور (إذا كان `auto_resize` مفعلاً).
- **مع `Nano2\Watermark\Classes\WatermarkHelper`**: للعلامة المائية (إذا كانت الإضافة موجودة و`auto_watermark` مفعلاً).

---

### أفضل الممارسات

1. **استخدم مفتاح جلسة مؤقت دائمًا للنماذج الجديدة**: لتجنب فقدان الملفات إذا لم يتم حفظ النموذج بعد.
2. **للاستبدال (edit)، استخدم `model` في الخيارات** بدلاً من المفتاح المؤقت، واترك الخدمة تتعامل مع المعاملة (`Db::transaction`) وحذف الملف القديم.
3. **لا تثق بالمفاتيح المؤقتة المقدمة من العميل دون تحقق**: الخدمة تقوم بذلك تلقائياً عبر `validateAndMatchTempKey`، ولكن تأكد من أنك لا تتجاوز هذه الخطوة.
4. **تحقق من صحة الملف قبل الرفع باستخدام `validateFile`**: لتقديم رسائل خطأ واضحة للمستخدم (الخدمة تفعل ذلك تلقائياً، لكن يمكنك استدعاؤها يدوياً أيضاً).
5. **سجل معرفات المعاملات (transaction IDs)** في قاعدة البيانات لتتبع عمليات الرفع.
6. **استخدم `uploadMultiple` للحقول المتعددة** بدلاً من استدعاء `upload` في حلقة، لأن `uploadMultiple` يجمع الأخطاء ويقدم إحصاءات.
7. **لا تعتمد على `temp_session_key` فقط**: بعد حفظ النموذج، استخدم `attachTempFiles` لنقل الملكية.
8. **تعامل مع الأخطاء حسب الكود**: 400 (طلب خاطئ)، 403 (صلاحية ممنوعة)، 404 (غير موجود)، 422 (فشل التحقق)، 500 (خطأ داخلي) – واستخدم `error_code` للتمييز الدقيق.
9. **فعّل `app.debug` في بيئة التطوير** للحصول على تفاصيل الأخطاء الكاملة.
10. **استخدم `with_thumbs` فقط عند الحاجة** لتقليل حجم الاستجابة وزيادة الأداء.
11. **احتفظ بمفاتيح الجلسة المؤقتة في الجلسة (session) أو قاعدة بيانات مؤقتة** لتجنب فقدانها.
12. **اختبر الصلاحيات جيدًا**: تأكد من أن المستخدمين غير المصرح لهم لا يمكنهم رفع أو حذف أو استبدال الملفات.
13. **استخدم التحويلات التلقائية (`auto_resize`, `auto_watermark`) بحكمة**: قد تستهلك موارد الخادم، لذا فعّلها فقط عند الحاجة.
14. **راقب سجل `fileupload.log`** للكشف عن محاولات رفع الملفات الخطيرة أو الأخطاء المتكررة.
15. **أنشئ مهمة مجدولة (cron)** لتنظيف الملفات منتهية الصلاحية (`expires_at`) وغير المرتبطة.
16. **استخدم `skip_permission_check => true` فقط في الاختبارات**، وليس في بيئة الإنتاج.

---

### الأخطاء الشائعة وحلولها

| الخطأ | السبب | الحل |
|-------|-------|------|
| `الحقل ... غير مسجل للنموذج ...` | الحقل غير مسجل في `FileUploadRegistry`. | أضف الحقل في التسجيل. |
| `لا تملك صلاحية رفع الملفات لهذا الحقل` | المستخدم لا يمتلك الصلاحية `add` للحقل أو `disable_upload` مفعل. | تحقق من إعدادات `permissions` في Registry أو الإعدادات العامة، ومن صلاحيات المستخدم. |
| `لا تملك صلاحية استبدال الملف لهذا الحقل` | محاولة `edit` دون صلاحية `edit` أو `disable_edit` مفعل. | تحقق من صلاحية `edit` في Registry والإعدادات العامة. |
| `بيانات الملف غير صالحة` | البيانات الممررة ليست `UploadedFile` ولا base64 صالح. | تأكد من صيغة البيانات. |
| `الملف غير موجود` (عند الحذف) | معرف الملف غير موجود. | تأكد من صحة المعرف. |
| `حجم الملف يتجاوز الحد المسموح به` | حجم الملف أكبر من `max_filesize`. | قم بضغط الملف أو زيادة `max_filesize` في إعدادات الحقل أو الإعدادات الافتراضية. |
| `نوع الملف غير مسموح به` | امتداد الملف غير مدرج في `allowed_types`. | استخدم نوع مسموح أو عدل `allowed_types`. |
| `نوع الملف ... غير مسموح به لأسباب أمنية` | الامتداد موجود في القائمة السوداء (`BLACKLISTED_EXTENSIONS`). | لا يمكن تجاوز هذا الحظر لأسباب أمنية، استخدم أنواع ملفات آمنة. |
| `مفتاح جلسة مؤقت غير صالح أو منتهي الصلاحية` | المفتاح منتهي (أكثر من `temp_key_ttl`) أو تم التلاعب به أو فشل التحقق في `validateAndMatchTempKey`. | أعد رفع الملف أو استخدم مفتاحاً جديداً. |
| `المفتاح المؤقت غير مخصص لهذا النموذج/الحقل` | فشل مطابقة النموذج أو الحقل في `validateAndMatchTempKey`. | تأكد من أن المفتاح يخص نفس النموذج والحقل. |
| `المفتاح المؤقت يخص مستخدم آخر` | فشل مطابقة المستخدم في `validateAndMatchTempKey`. | تأكد من أن المفتاح يخص المستخدم الحالي. |
| `Auto-resize failed` | مكتبة GD أو Imagick غير مثبتة، أو مسار الملف غير صالح. | ثبّت `php-gd` أو `php-imagick`، وتحقق من صلاحيات الملف. |
| `Auto-watermark failed` | إضافة `Nano2.Watermark` غير مثبتة أو مسار الشعار غير صحيح. | ثبّت الإضافة وتأكد من وجود ملف الشعار في المسار المحدد. |
| `Storage disk 's3' not found` | لم يتم تكوين قرص `s3` في `config/filesystems.php`. | أضف إعدادات القرص المناسبة. |
| `تم حذف الملف القديم على الرغم من فشل رفع الجديد` | (نادر) خطأ في المعاملة أو إعدادات قاعدة البيانات. | تأكد من استخدام محرك InnoDB لدعم المعاملات. |

---

### الخلاصة

كلاس `FileUploadService` هو العمود الفقري لعمليات رفع الملفات في تطبيقات نانوسوفت. يوفر واجهة موحدة وآمنة لرفع وحذف واسترجاع الملفات، مع دعم كامل لأنواع المستخدمين المختلفة والصلاحيات الدقيقة، والربط المؤقت للنماذج غير المحفوظة، والتحويلات التلقائية للصور، والتخزين المتعدد، وخطافات الأحداث، وإشعارات WebSocket. بدءاً من الإصدار 1.1.0، يدعم أيضاً عملية استبدال الملفات (edit) في علاقات `attachOne` باستخدام المعاملات (`Db::transaction`) والأحداث المخصصة. وفي الإصدار 1.2.2، تم إضافة تريت `HasFileUploadsMatchTempKey` الذي يوفر دالة `validateAndMatchTempKey` لتوحيد منطق التحقق من المفاتيح المؤقتة، مما يزيد الأمان ويقلل تكرار الكود في دوال `upload`, `deleteFile`, `attachTempFiles`, `getFiles`. بفضل اعتماده على `FileUploadRegistry` و `FileUploadUserManager`، يمكن للمطورين بناء أنظمة رفع ملفات قوية وقابلة للتوسع دون القلق بشأن التفاصيل التنفيذية أو أمان البيانات.

للاطلاع على أمثلة إضافية متقدمة، راجع [توثيق الأمثلة المتقدمة لكلاس `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-ar.md).

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-ar.md)
-- [validateAndMatchTempKey Function for `FileUploadService` Class](./Docs-FileUploadService-validateAndMatchTempKey-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
