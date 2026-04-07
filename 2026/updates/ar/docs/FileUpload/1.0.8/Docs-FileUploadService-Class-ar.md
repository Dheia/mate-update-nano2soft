# توثيق كلاس `FileUploadService`

**نبذة تعريفية عن حزمة `Nano.FileUpload` ووحدة إدارة رفع الملفات**

هذه المجموعة من الكلاسات (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager` وغيرها) هي جزء من **حزمة `Nano.FileUpload`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **نانوسوفت** بهدف توحيد وإدارة عمليات رفع الملفات عبر واجهات برمجة التطبيقات (API) بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano\FileUpload\Classes
```

توفر هذه الوحدة أدوات متقدمة لتسجيل النماذج (الموديولات) التي تحتوي على حقول رفع ملفات، والتحقق من صلاحيات المستخدمين (بما في ذلك أنواع المستخدمين المختلفة: `backend` و `frontend`)، وإدارة الملفات المرفوعة مؤقتًا قبل حفظ السجل، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة رفع ملفات قابلة للتوسع بسهولة وتلبية احتياجات التطبيقات المتقدمة دون المساس بأمان التطبيق أو أدائه.

> **ملاحظة:** يغطي هذا المستند الإصدارات حتى 1.0.7، ويشمل الميزات المتقدمة مثل التخزين المتعدد، التحويل التلقائي للصور، خطافات الأحداث، WebSocket، وتحسينات الأمان والحقول الجديدة في قاعدة البيانات.

---

### مقدمة

كلاس `FileUploadService` هو **طبقة الخدمة (Service Layer)** المسؤولة عن تنفيذ عمليات رفع الملفات وحذفها واسترجاعها في تطبيقات نانوسوفت. يعتمد هذا الكلاس على `FileUploadRegistry` للوصول إلى إعدادات النماذج والتحقق من الصلاحيات، وعلى `FileUploadUserManager` لإدارة المستخدم الحالي والتحقق من صلاحياته.

ببساطة، `FileUploadService` هو **المحرك الذي ينفذ عمليات الرفع الفعلية**، ويتأكد من أن كل عملية تتم وفقًا للإعدادات المسجلة والصلاحيات المحددة، مع دعم الرفع المؤقت (عبر مفتاح جلسة مؤقت) لربط الملفات بالنماذج غير المحفوظة بعد.

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

#### 2. التحقق من صحة مفتاح جلسة مؤقت

##### `public function validateTempSessionKey(string $tempSessionKey): ?array`
- **الهدف**: التحقق من صحة مفتاح الجلسة المؤقت (التوقيع، الانتهاء، البنية) واستخراج بياناته.
- **المعاملات**: `$tempSessionKey` – المفتاح المراد التحقق منه.
- **الإرجاع**: مصفوفة تحتوي على `modelClass`, `field`, `userId`, `userType`, `timestamp`، أو `null` إذا كان غير صالح.
- **السلوك**:
  - يتحقق من البداية `tmp_`.
  - يفك الترميز ويتحقق من التوقيع.
  - يتحقق من صلاحية المفتاح (الافتراضي 3600 ثانية، قابل للتعديل عبر `security.temp_key_ttl`).

#### 3. رفع ملف واحد

##### `public function upload(string $modelClass, string $field, $fileData, array $options = []): array`
- **الهدف**: رفع ملف واحد لحقل معين.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$fileData`: بيانات الملف (كائن `UploadedFile` أو سلسلة base64).
  - `$options`: مصفوفة خيارات إضافية:
    - `model`: كائن النموذج (إذا كان موجودًا ومحفوظًا).
    - `temp_session_key`: مفتاح جلسة مؤقت (لتجاوز التوليد التلقائي).
    - `title`, `description`: لتحديد عنوان ووصف الملف.
    - `...`: أي خيارات إضافية تُمرر إلى `Base64::onUpload`.
- **السلوك**:
  - يتحقق من تسجيل النموذج والحقل في `FileUploadRegistry`.
  - يتحقق من صلاحية `add` للمستخدم الحالي (مع مراعاة الإعدادات العامة مثل `disable_upload`).
  - يطلق حدث `nano.fileupload.beforeUpload`.
  - يتحقق من صحة الملف (القائمة السوداء، الحجم، الأنواع) عبر `validateFile`.
  - يحدد ما إذا كان النموذج موجودًا ومحفوظًا (لربط مباشر) أو يحتاج إلى مفتاح جلسة مؤقت.
  - يستدعي `Base64::onUpload` لتنفيذ الرفع الفعلي.
  - **يطبق التحويلات التلقائية** (تحجيم، علامة مائية) إذا كان الحقل من نوع `image` والإعدادات مفعلة.
  - **يحسب SHA256** للمحتوى ويخزنه في `hash` (إذا لم يكن موجوداً).
  - **يخزن أبعاد الصورة** في `meta` (كـ JSON) إذا كان الملف صورة.
  - **يضبط `expires_at`** للملفات المؤقتة (24 ساعة) إذا كان الملف غير مرتبط.
  - **يعين قرص التخزين المخصص** (إذا تم تحديد `storage_disk` في إعدادات الحقل).
  - يحفظ الملف (إذا لم يتم حفظه بعد أو حدثت تغييرات).
  - يُضيف `temp_session_key` إلى النتيجة إذا تم استخدامه.
  - يطلق حدث `nano.fileupload.afterUpload` ويرسل إشعار WebSocket (إذا مفعل).
- **الإرجاع**: مصفوفة بنفس هيكل `Base64::onUpload` مع حقول إضافية: `code`, `status`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`, `error_code`.
- **مثال**:
  ```php
  $result = $service->upload(
      'Nano\Shop\Models\Product',
      'image',
      $uploadedFile,
      ['temp_session_key' => $tempKey]
  );
  if ($result['status']) {
      echo "تم رفع الملف بنجاح. ID: " . $result['data']['id'];
  } else {
      echo "فشل: " . $result['error'] . " (كود: " . $result['error_code'] . ")";
  }
  ```

#### 4. رفع عدة ملفات (لحقل متعدد)

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

#### 5. حذف ملف

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
  - إذا تم تمرير `$modelClass` و `$field`، يتحقق من صلاحية `delete` للمستخدم الحالي (مع مراعاة `disable_delete`).
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

#### 6. جلب الملفات

##### `public function getFiles(string $modelClass, string $field, ?int $modelId = null, array $options = []): array`
- **الهدف**: استرجاع الملفات المرتبطة بنموذج وحقل معين أو ملفات مؤقتة.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$modelId`: (اختياري) معرف النموذج المحفوظ.
  - `$options`: خيارات إضافية:
    - `temp_session_key`: مفتاح جلسة مؤقت (لجلب الملفات غير المرتبطة).
    - `with_thumbs`: `bool` لتضمين الصور المصغرة.
    - `thumb_sizes`: مصفوفة بأحجام الصور المصغرة (مثل `['thumb' => [100,100]]`).
- **السلوك**:
  - يتحقق من تسجيل الحقل وصلاحية `view` (مع مراعاة `disable_get`).
  - يبني استعلامًا على `File` مع تصفية حسب `field` و (`attachment_id` + `attachment_type` إذا كان `modelId` موجودًا) أو `session_key` إذا كان `temp_session_key` موجودًا.
  - **عند استخدام `temp_session_key`**، يتحقق من صحة المفتاح (التوقيع، الانتهاء، مطابقة المستخدم والنموذج والحقل) قبل تنفيذ الاستعلام.
  - يعيد الملفات مع البيانات الأساسية (id, title, description, path, size, content_type) وصور مصغرة إذا طُلب.
- **الإرجاع**: مصفوفة تحتوي على `code`, `status`, `message`, `data`.
- **مثال**:
  ```php
  $files = $service->getFiles('Nano\Shop\Models\Product', 'image', 456, ['with_thumbs' => true]);
  foreach ($files['data'] as $file) {
      echo "<img src='{$file['thumb']}'>";
  }
  ```

#### 7. ربط الملفات المؤقتة بنموذج محفوظ

##### `public function attachTempFiles(Model $model, string $field, string $tempSessionKey): bool`
- **الهدف**: نقل الملفات التي تم رفعها باستخدام مفتاح جلسة مؤقت إلى نموذج محفوظ.
- **المعاملات**:
  - `$model`: كائن النموذج المحفوظ.
  - `$field`: اسم الحقل.
  - `$tempSessionKey`: مفتاح الجلسة المؤقت المستخدم أثناء الرفع.
- **السلوك**:
  - يتحقق من صحة المفتاح (التوقيع، الانتهاء).
  - يتحقق من تطابق `modelClass` و `field` بين المفتاح والنموذج.
  - يتحقق من أن المستخدم الحالي هو نفس المستخدم الذي أنشأ المفتاح (نفس `userId` و `userType`).
  - يبحث عن الملفات التي تحمل `session_key = $tempSessionKey` و `field = $field`.
  - لكل ملف، يحدّث `attachment_id` و `attachment_type` لربطه بالنموذج ويزيل `session_key`.
  - إذا كان للنموذج علاقة بهذا الحقل، يستخدم `add()`، وإلا يعيّن الحقل مباشرة.
- **الإرجاع**: `true` إذا تم ربط ملفات، وإلا `false`.
- **مثال**:
  ```php
  $product = new Product();
  $product->name = 'منتج جديد';
  $product->save();
  $service->attachTempFiles($product, 'image', $tempKey);
  ```

#### 8. التحقق من صحة الملف

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

#### 9. دوال مساعدة (التخزين والتحويل والإشعارات)

##### `protected function getStorageDiskForField($modelClass, $field): ?string`
- يحدد اسم قرص التخزين المخصص للحقل (من `getProcessingOptions`) ويتحقق من وجوده.

##### `protected function applyAutoProcessing($file, $modelClass, $field): void`
- يطبق التحجيم التلقائي والعلامة المائية على الصور وفق خيارات الحقل والإعدادات العامة.

##### `protected function notifyWebSocket($event, $data): void`
- يطلق حدث `nano.fileupload.websocket.notify` إذا كان WebSocket مفعلاً في الإعدادات.

---

### الأحداث (Events)

| اسم الحدث | مكان الاستدعاء | المعاملات |
|-----------|----------------|------------|
| `nano.fileupload.beforeUpload` | بداية `upload` | `$modelClass, $field, $fileData, &$options` |
| `nano.fileupload.afterUpload` | نهاية `upload` (بعد النجاح) | `$file, $modelClass, $field, $options` |
| `nano.fileupload.beforeDelete` | بداية `deleteFile` | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterDelete` | نهاية `deleteFile` (بعد الحذف) | `$fileId, $modelClass, $field` |
| `nano.fileupload.websocket.notify` | بعد `upload` أو `delete` (إذا مفعل WebSocket) | `$channel, $event, $data` |

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

// 4. ربط الصورة بالمنتج (الخدمة تتحقق من المفتاح والمستخدم)
$service->attachTempFiles($product, $field, $tempKey);

echo "تم رفع الصورة وربطها بالمنتج رقم {$product->id}";
```

#### مثال 2: رفع عدة صور لمعرض منتج موجود (مع WebSocket)

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

#### مثال 3: جلب جميع صور معرض منتج مع صور مصغرة مخصصة

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

#### مثال 4: حذف صورة رئيسية بعد التحقق من الصلاحية (مع الأحداث)

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

#### مثال 5: التحقق من صحة الملف قبل الرفع (يكتشف ملف PHP خطير)

```php
$maliciousFile = $request->file('upload'); // ملف php
try {
    $service->validateFile(Product::class, 'image', $maliciousFile);
    // لن يصل إلى هنا
} catch (FileUploadException $e) {
    echo $e->getErrorCode(); // FILE_UPLOAD_FILE_TYPE_BLACKLISTED
    echo $e->getMessage();   // "نوع الملف php غير مسموح به لأسباب أمنية."
}
```

#### مثال 6: استخدام حدث `afterUpload` لإرسال بريد إلكتروني

```php
Event::listen('nano.fileupload.afterUpload', function ($file, $modelClass, $field, $options) {
    // إرسال إشعار للمسؤول
    Mail::to('admin@example.com')->send(new FileUploadedNotification($file));
});
```

#### مثال 7: دمج `FileUploadService` مع واجهة برمجة تطبيقات (API) باستخدام أكواد الأخطاء

في متحكم API:

```php
public function uploadImage(Request $request)
{
    $modelClass = $request->input('model_class');
    $field = $request->input('field');
    $file = $request->file('file');

    $service = FileUploadService::instance();
    $result = $service->upload($modelClass, $field, $file, [
        'temp_session_key' => $request->input('temp_session_key')
    ]);

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

#### مثال 8: استخدام WebSocket للإشعارات الفورية

في ملف `.env`:
```ini
NANO_FILE_UPLOAD_WEBSOCKET_ENABLED=true
NANO_FILE_UPLOAD_WEBSOCKET_CHANNEL=file-uploads
```

ثم الاستماع للحدث:
```php
Event::listen('nano.fileupload.websocket.notify', function ($channel, $event, $data) {
    broadcast(new \App\Events\FileUploadWebsocketEvent($channel, $event, $data));
});
```

---

### التفاعل مع الكلاسات الأخرى

- **مع `FileUploadRegistry`**: يستخدم `$registry` للحصول على إعدادات الحقول (`getFieldConfig`, `getFieldConstraints`, `getProcessingOptions`) والتحقق من الصلاحيات (`can`).
- **مع `FileUploadUserManager`**: يستخدم `$userManager` للحصول على المستخدم الحالي ونوعه (`getUser`, `getUserType`, `getId`).
- **مع `Base64` (من `Nano.Api`)**: يستدعي `Base64::onUpload`, `Base64::is_base64`, `Base64::base64ToUploadedFile`, `Base64::getBase64ImageSize`, `Base64::getDataInBase64`.
- **مع `October\Rain\Resize\Resizer`**: لتحجيم الصور (إذا كان `auto_resize` مفعلاً).
- **مع `Nano2\Watermark\Classes\WatermarkHelper`**: للعلامة المائية (إذا كانت الإضافة موجودة و`auto_watermark` مفعلاً).

---

### أفضل الممارسات

1. **استخدم مفتاح جلسة مؤقت دائمًا للنماذج الجديدة**: لتجنب فقدان الملفات إذا لم يتم حفظ النموذج بعد.
2. **تحقق من صحة الملف قبل الرفع باستخدام `validateFile`**: لتقديم رسائل خطأ واضحة للمستخدم (الخدمة تفعل ذلك تلقائياً، لكن يمكنك استدعاؤها يدوياً أيضاً).
3. **سجل معرفات المعاملات (transaction IDs)** في قاعدة البيانات لتتبع عمليات الرفع.
4. **استخدم `uploadMultiple` للحقول المتعددة** بدلاً من استدعاء `upload` في حلقة، لأن `uploadMultiple` يجمع الأخطاء ويقدم إحصاءات.
5. **لا تعتمد على `temp_session_key` فقط**: بعد حفظ النموذج، استخدم `attachTempFiles` لنقل الملكية.
6. **تعامل مع الأخطاء حسب الكود**: 400 (طلب خاطئ)، 403 (صلاحية ممنوعة)، 404 (غير موجود)، 500 (خطأ داخلي) – واستخدم `error_code` للتمييز الدقيق.
7. **فعّل `app.debug` في بيئة التطوير** للحصول على تفاصيل الأخطاء الكاملة.
8. **استخدم `with_thumbs` فقط عند الحاجة** لتقليل حجم الاستجابة وزيادة الأداء.
9. **احتفظ بمفاتيح الجلسة المؤقتة في الجلسة (session) أو قاعدة بيانات مؤقتة** لتجنب فقدانها.
10. **اختبر الصلاحيات جيدًا**: تأكد من أن المستخدمين غير المصرح لهم لا يمكنهم رفع أو حذف الملفات.
11. **استخدم التحويلات التلقائية (`auto_resize`, `auto_watermark`) بحكمة**: قد تستهلك موارد الخادم، لذا فعّلها فقط عند الحاجة.
12. **راقب سجل `fileupload.log`** للكشف عن محاولات رفع الملفات الخطيرة أو الأخطاء المتكررة.
13. **أنشئ مهمة مجدولة (cron)** لتنظيف الملفات منتهية الصلاحية (`expires_at`) وغير المرتبطة.

---

### الأخطاء الشائعة وحلولها

| الخطأ | السبب | الحل |
|-------|-------|------|
| `الحقل ... غير مسجل للنموذج ...` | الحقل غير مسجل في `FileUploadRegistry`. | أضف الحقل في التسجيل. |
| `لا تملك صلاحية رفع الملفات لهذا الحقل` | المستخدم لا يمتلك الصلاحية `add` للحقل أو `disable_upload` مفعل. | تحقق من إعدادات `permissions` في Registry أو الإعدادات العامة، ومن صلاحيات المستخدم. |
| `بيانات الملف غير صالحة` | البيانات الممررة ليست `UploadedFile` ولا base64 صالح. | تأكد من صيغة البيانات. |
| `الملف غير موجود` (عند الحذف) | معرف الملف غير موجود. | تأكد من صحة المعرف. |
| `حجم الملف يتجاوز الحد المسموح به` | حجم الملف أكبر من `max_filesize`. | قم بضغط الملف أو زيادة `max_filesize` في إعدادات الحقل أو الإعدادات الافتراضية. |
| `نوع الملف غير مسموح به` | امتداد الملف غير مدرج في `allowed_types`. | استخدم نوع مسموح أو عدل `allowed_types`. |
| `نوع الملف ... غير مسموح به لأسباب أمنية` | الامتداد موجود في القائمة السوداء (`BLACKLISTED_EXTENSIONS`). | لا يمكن تجاوز هذا الحظر لأسباب أمنية، استخدم أنواع ملفات آمنة. |
| `مفتاح جلسة مؤقت غير صالح أو منتهي الصلاحية` | المفتاح منتهي (أكثر من `temp_key_ttl`) أو تم التلاعب به. | أعد رفع الملف أو استخدم مفتاحاً جديداً. |
| `غير مصرح لك بالوصول إلى هذه الملفات المؤقتة` | محاولة الوصول إلى ملفات مؤقتة لمستخدم آخر أو بنموذج/حقل مختلف. | تأكد من أن المفتاح يخص المستخدم الحالي والنموذج/الحقل الصحيح. |
| `Auto-resize failed` | مكتبة GD أو Imagick غير مثبتة، أو مسار الملف غير صالح. | ثبّت `php-gd` أو `php-imagick`، وتحقق من صلاحيات الملف. |
| `Auto-watermark failed` | إضافة `Nano2.Watermark` غير مثبتة أو مسار الشعار غير صحيح. | ثبّت الإضافة وتأكد من وجود ملف الشعار في المسار المحدد. |
| `Storage disk 's3' not found` | لم يتم تكوين قرص `s3` في `config/filesystems.php`. | أضف إعدادات القرص المناسبة. |

---

### الخلاصة

كلاس `FileUploadService` هو العمود الفقري لعمليات رفع الملفات في تطبيقات نانوسوفت. يوفر واجهة موحدة وآمنة لرفع وحذف واسترجاع الملفات، مع دعم كامل لأنواع المستخدمين المختلفة والصلاحيات الدقيقة، والربط المؤقت للنماذج غير المحفوظة، والتحويلات التلقائية للصور، والتخزين المتعدد، وخطافات الأحداث، وإشعارات WebSocket. بفضل اعتماده على `FileUploadRegistry` و `FileUploadUserManager`، يمكن للمطورين بناء أنظمة رفع ملفات قوية وقابلة للتوسع دون القلق بشأن التفاصيل التنفيذية أو أمان البيانات.

للاطلاع على أمثلة إضافية متقدمة، راجع [توثيق الأمثلة المتقدمة لكلاس `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-ar.md).

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
