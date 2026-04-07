# توثيق الأمثلة المتقدمة والعملية لكلاس `FileUploadService`

**Namespace:** `Nano\FileUpload\Classes`  
**الهدف:** تقديم أمثلة تطبيقية شاملة لاستخدام `FileUploadService` في سيناريوهات متنوعة، بدءًا من العمليات الأساسية لرفع الملفات وصولًا إلى التعامل مع الملفات المؤقتة، والربط بالنماذج، والتحقق من الصلاحيات والقيود، بالإضافة إلى الميزات المتقدمة من الإصدارات 1.0.6 و 1.0.7: التخزين المتعدد، التحويل التلقائي للصور، خطافات الأحداث، WebSocket، حساب `hash` و `meta` و `expires_at`، والتعامل مع `FileUploadException`.

> **ملاحظة:** جميع الأمثلة تفترض أن الكلاس `FileUploadService` مهيأ عبر `FileUploadService::instance()`.

---

## مقدمة

يقدم هذا الوثيقة مجموعة من الأمثلة العملية التي تغطي مختلف حالات استخدام كلاس `FileUploadService`، الذي يمثل **طبقة الخدمة** المسؤولة عن تنفيذ عمليات رفع الملفات وحذفها واسترجاعها في تطبيقات نانوسوفت. يعتمد هذا الكلاس على `FileUploadRegistry` للوصول إلى إعدادات النماذج والتحقق من الصلاحيات، وعلى `FileUploadUserManager` لإدارة المستخدم الحالي والتحقق من صلاحياته.

من خلال هذه الأمثلة، ستتعلم كيفية:

- رفع ملف واحد أو عدة ملفات باستخدام بيانات `UploadedFile` أو base64.
- استخدام مفاتيح الجلسة المؤقتة (`temp_session_key`) لربط الملفات بالنماذج غير المحفوظة.
- ربط الملفات المؤقتة بنموذج بعد حفظه.
- جلب الملفات المرتبطة بنموذج مع صور مصغرة.
- حذف ملفات مع التحقق من الصلاحيات.
- التحقق من صحة الملف (حجم، نوع) قبل الرفع.
- التعامل مع الأخطاء المختلفة (صلاحيات، قيود، أخطاء الخادم).
- استخدام التخزين المتعدد (أقراص S3, FTP, Local).
- تطبيق التحويلات التلقائية (تحجيم وعلامة مائية) على الصور.
- الاستماع إلى الأحداث (`beforeUpload`, `afterUpload`, `beforeDelete`, `afterDelete`).
- إرسال إشعارات WebSocket عند اكتمال العمليات.
- التعامل مع أكواد الأخطاء الفريدة (`error_code`) من `FileUploadException`.
- الاستفادة من الحقول الجديدة (`hash`, `meta`, `expires_at`).

---

## المتطلبات الأساسية

- إضافة `Nano.FileUpload` مثبتة ومهيأة في تطبيقك (الإصدار 1.0.7 أو أحدث).
- الإضافة تعتمد على `Nano.Api` التي توفر `Base64` و `ApiController`.
- تم تسجيل النماذج وحقول الرفع في `FileUploadRegistry` كما هو موضح في [توثيق الأمثلة المتقدمة لكلاس FileUploadRegistry](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md).
- معرفة أساسية بكائنات `UploadedFile` وكيفية الحصول عليها من الطلبات (مثلاً عبر `$request->file('file')`).

### إعداد الكلاس في التطبيق

```php
use Nano\FileUpload\Classes\FileUploadService;

$service = FileUploadService::instance();
```

نظرًا لأن الكلاس يتبع نمط `Singleton`، يتم استدعاؤه عبر `instance()`.

---

## 1. رفع ملف واحد باستخدام بيانات UploadedFile (ربط مباشر بنموذج موجود)

**السيناريو:** منتج موجود بالفعل، نريد رفع صورة رئيسية له وربطها مباشرة.

```php
$product = Product::find(10);
$uploadedFile = $request->file('image'); // كائن UploadedFile

$result = $service->upload(Product::class, 'image', $uploadedFile, [
    'model' => $product, // الربط مباشر
    'title' => 'صورة المنتج',
    'description' => 'وصف الصورة',
]);

if ($result['status']) {
    echo "تم رفع الصورة بنجاح. معرف الملف: " . $result['data']['id'];
    echo "المسار: " . $result['data']['path'];
} else {
    echo "فشل الرفع: " . $result['error'] . " (كود: " . $result['error_code'] . ")";
}
```

**المخرجات المتوقعة (في حالة النجاح):**

```php
[
    'code' => 200,
    'status' => true,
    'message' => 'تم رفع الملف بنجاح',
    'data' => [
        'id' => 123,
        'path' => '/storage/app/uploads/public/...',
        'thumb' => '/storage/app/uploads/public/...',
    ],
    'process_data' => [
        'model_class' => 'Nano\Shop\Models\Product',
        'field' => 'image',
        'temp_session_key' => null,
        'user_type' => 'backend',
        'storage_disk' => null,
        // ... بيانات إضافية
    ],
]
```

**ملاحظات:**  
- عندما يتم تمرير `model` موجود ومحفوظ (`$product->exists`)، يتم الربط مباشرة دون استخدام مفتاح جلسة مؤقت.  
- يمكن تخصيص `title` و `description` لإضافتها للملف.

---

## 2. رفع ملف واحد باستخدام base64 (نموذج غير محفوظ)

**السيناريو:** إنشاء منتج جديد، نريد رفع صورته قبل حفظ المنتج، ثم ربطها لاحقًا.

```php
$base64String = 'data:image/jpeg;base64,/9j/4AAQSkZJRg...';

$modelClass = Product::class;
$field = 'image';

// توليد مفتاح جلسة مؤقت (الخدمة ستضيف userType و timestamp تلقائياً)
$tempKey = $service->generateTempSessionKey($modelClass, $field);

// رفع الصورة
$result = $service->upload($modelClass, $field, $base64String, [
    'temp_session_key' => $tempKey,
]);

if ($result['status']) {
    // حفظ المنتج
    $product = new Product();
    $product->name = 'منتج جديد';
    $product->save();

    // ربط الصورة بالمنتج
    $service->attachTempFiles($product, $field, $tempKey);

    echo "تم رفع الصورة وربطها بالمنتج رقم {$product->id}";
} else {
    echo "فشل الرفع: " . $result['error'];
}
```

**المخرجات:**  
- `upload` يعيد `temp_session_key` في `$result['temp_session_key']` إذا تم استخدامه.  
- `attachTempFiles` يعيد `true` إذا تم ربط ملفات، وإلا `false`.  
- **سيتم تلقائياً:** حساب `hash` (SHA256) للمحتوى، وتخزين أبعاد الصورة في `meta`، وتعيين `expires_at` للملف المؤقت (24 ساعة).

**ملاحظات:**  
- `generateTempSessionKey` ليس إجباريًا؛ إذا لم تمرر `temp_session_key`، سيقوم `upload` بتوليد واحد تلقائيًا ويعيده في النتيجة.  
- يجب حفظ `temp_session_key` في جلسة المستخدم أو قاعدة بيانات مؤقتة لاستخدامه لاحقًا.

---

## 3. رفع صورة مع تحجيم تلقائي وعلامة مائية (حسب إعدادات الحقل)

**السيناريو:** بعد تسجيل الحقل مع `auto_resize` و `auto_watermark` (كما في مثال `FileUploadRegistry`)، يتم تطبيق التحويلات تلقائياً أثناء الرفع.

```php
// تعريف الحقل في registry (مرة واحدة)
// 'image' => [
//     'type' => 'image',
//     'auto_resize' => true,
//     'resize_options' => ['width' => 800, 'height' => 600, 'mode' => 'crop'],
//     'auto_watermark' => true,
//     'watermark_options' => ['position' => 'bottom-right', 'resize_percentage' => 15],
// ]

$result = $service->upload(Product::class, 'image', $uploadedFile, [
    'model' => $product,
]);
// بعد الرفع، ستكون الصورة محجمة وموقعة بالعلامة المائية.
```

**التحقق من تطبيق التحويلات:**

يمكنك مراقبة السجلات (`storage/logs/laravel.log` أو `fileupload.log`) للتأكد من نجاح العملية. في حالة الفشل (مثل عدم وجود مكتبة GD)، سيظهر خطأ في السجل مع `Auto-resize failed`.

---

## 4. رفع ملفات متعددة لحقل متعدد (gallery)

**السيناريو:** رفع مجموعة صور لمعرض منتج موجود.

```php
$product = Product::find(10);
$files = $request->file('gallery'); // مصفوفة من UploadedFile

$result = $service->uploadMultiple(Product::class, 'gallery', $files, [
    'model' => $product, // الربط مباشر
]);

if ($result['status']) {
    echo "تم رفع " . $result['process_data']['success_count'] . " من " . $result['process_data']['total'] . " صور بنجاح.";
    foreach ($result['data'] as $index => $fileResult) {
        if ($fileResult['status']) {
            echo "الصورة $index: معرف {$fileResult['data']['id']}\n";
        } else {
            echo "الصورة $index فشلت: {$fileResult['error']} (كود: {$fileResult['error_code']})\n";
        }
    }
} else {
    echo "فشل رفع جميع الصور: " . $result['error'];
}
```

**المخرجات المتوقعة (جزء من `$result`):**

```php
[
    'code' => 200,
    'status' => true,
    'message' => 'تم رفع 2 من 3 ملف بنجاح',
    'data' => [
        0 => ['status' => true, 'data' => ['id' => 101, ...]],
        1 => ['status' => true, 'data' => ['id' => 102, ...]],
        2 => ['status' => false, 'error' => 'نوع الملف غير مسموح به', 'error_code' => 'FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED'],
    ],
    'process_data' => [
        'success_count' => 2,
        'total' => 3,
    ],
]
```

---

## 5. جلب ملفات حقل معين مع صور مصغرة

**السيناريو:** عرض جميع صور معرض منتج مع صور مصغرة بأحجام مختلفة.

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'small' => [150, 150, 'crop'],
        'medium' => [300, 300, 'crop'],
        'large' => [800, 600, 'auto'],
    ],
]);

if ($result['status']) {
    foreach ($result['data'] as $image) {
        echo "<img src='{$image['small']}' alt='{$image['title']}'>";
        echo "<a href='{$image['large']}'>عرض بحجم كبير</a>";
    }
} else {
    echo "خطأ: " . $result['error'];
}
```

**المخرجات (مثال لملف واحد):**

```php
[
    'id' => 101,
    'title' => 'صورة المنتج',
    'description' => null,
    'path' => '/storage/app/uploads/public/.../original.jpg',
    'size' => 123456,
    'content_type' => 'image/jpeg',
    'small' => '/storage/app/uploads/public/.../small.jpg',
    'medium' => '/storage/app/uploads/public/.../medium.jpg',
    'large' => '/storage/app/uploads/public/.../large.jpg',
]
```

---

## 6. جلب الملفات المؤقتة (غير المرتبطة) مع التحقق الأمني

**السيناريو:** بعد رفع الملفات باستخدام مفتاح جلسة مؤقت، نريد عرضها قبل حفظ النموذج. الخدمة تتحقق من صحة المفتاح ومطابقة المستخدم والنموذج والحقل.

```php
$tempKey = $request->input('temp_session_key');
$result = $service->getFiles(Product::class, 'gallery', null, [
    'temp_session_key' => $tempKey,
]);

if ($result['status']) {
    echo "عدد الملفات المؤقتة: " . count($result['data']);
    foreach ($result['data'] as $file) {
        echo "<img src='{$file['path']}'>";
    }
} else {
    echo "فشل جلب الملفات المؤقتة: " . $result['error'];
}
```

**ملاحظات أمنية:**  
- إذا كان المفتاح منتهي الصلاحية أو تم التلاعب به، سترفض الخدمة الطلب مع رسالة خطأ.  
- إذا حاول مستخدم آخر استخدام مفتاح مستخدم آخر، سترفض الخدمة الطلب مع كود `FILE_UPLOAD_TEMP_KEY_MISMATCH`.

---

## 7. حذف ملف مع التحقق من الصلاحية والأحداث

**السيناريو:** مستخدم يريد حذف صورة رئيسية لمنتج، نتحقق من صلاحيته أولاً، ونستخدم الأحداث لتسجيل العملية.

```php
Event::listen('nano.fileupload.beforeDelete', function ($fileId, $modelClass, $field) {
    \Log::info("محاولة حذف ملف: {$fileId} من النموذج {$modelClass}");
});

Event::listen('nano.fileupload.afterDelete', function ($fileId, $modelClass, $field) {
    \Log::info("تم حذف الملف: {$fileId}");
});

$result = $service->deleteFile(123, Product::class, 'image');

if ($result['status']) {
    echo "تم حذف الملف بنجاح";
} else {
    echo "فشل الحذف: " . $result['message'] . " (كود: " . $result['error_code'] . ")";
}
```

**ملاحظات:**  
- إذا لم يتم تمرير `$modelClass` و `$field`، لن يتم التحقق من الصلاحية.  
- التحقق يستخدم صلاحية `delete` من إعدادات الحقل أو النموذج، بالإضافة إلى الإعدادات العامة `disable_delete`.

---

## 8. التحقق من صحة الملف قبل الرفع (يكتشف الملفات الخطيرة)

**السيناريو:** قبل رفع الملف، نتحقق من حجمه ونوعه وقائمة الامتدادات الخطيرة. إذا كان الملف من نوع PHP، سيتم رفضه مع كود خطأ محدد.

```php
$uploadedFile = $request->file('image');

try {
    $service->validateFile(Product::class, 'image', $uploadedFile);
    // الملف صالح
    $result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);
} catch (FileUploadException $e) {
    echo "الملف غير صالح: " . $e->getMessage();
    echo "كود الخطأ: " . $e->getErrorCode(); // مثلاً FILE_UPLOAD_FILE_TYPE_BLACKLISTED
    echo "السياق: " . print_r($e->getContext(), true);
}
```

**الاستثناءات الشائعة:**  
- `FILE_UPLOAD_FILE_SIZE_EXCEEDED` – حجم الملف أكبر من الحد المسموح.  
- `FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED` – امتداد الملف غير مسموح.  
- `FILE_UPLOAD_FILE_TYPE_BLACKLISTED` – امتداد خطير (PHP, JS, HTML, EXE, إلخ) – لا يمكن تجاوزه.

---

## 9. التعامل مع أخطاء الصلاحية والإعدادات العامة

**السيناريو:** محاولة رفع ملف دون صلاحية أو عندما يكون الرفع معطلاً عالمياً.

```php
$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);

if (!$result['status']) {
    switch ($result['error_code']) {
        case 'FILE_UPLOAD_PERMISSION_DENIED':
            echo "لا تملك صلاحية رفع الملفات لهذا الحقل";
            break;
        case 'FILE_UPLOAD_UPLOAD_DISABLED_GLOBALLY':
            echo "عمليات رفع الملفات معطلة حالياً";
            break;
        case 'FILE_UPLOAD_FIELD_NOT_REGISTERED':
            echo "الحقل غير مسجل";
            break;
        default:
            echo "خطأ: " . $result['error'];
    }
}
```

**كود HTTP المستخدمة:**  
- `400`: بيانات غير صالحة أو قيود الحقل.  
- `403`: صلاحية ممنوعة (بما في ذلك تعطيل العمليات عالمياً).  
- `404`: النموذج أو الحقل غير مسجل.  
- `500`: خطأ داخلي (يظهر تفاصيل في debug).

---

## 10. استخدام الأحداث `beforeUpload` و `afterUpload` لتوسيع السلوك

**السيناريو:** تسجيل محاولات الرفع، تعديل الخيارات، أو إرسال إشعارات.

```php
// قبل الرفع
Event::listen('nano.fileupload.beforeUpload', function ($modelClass, $field, &$fileData, &$options) {
    \Log::info("محاولة رفع ملف: {$modelClass} - {$field}");
    // إضافة خيار إضافي
    $options['custom_flag'] = true;
    // يمكن تعديل $fileData أو $options حسب الحاجة
});

// بعد الرفع
Event::listen('nano.fileupload.afterUpload', function ($file, $modelClass, $field, $options) {
    \Log::info("تم رفع ملف: {$file->id} للنموذج {$modelClass}");
    // إرسال إشعار عبر البريد أو WebSocket
    broadcast(new FileUploadedEvent($file));
});

$result = $service->upload(Product::class, 'image', $fileData);
```

---

## 11. استخدام WebSocket للإشعارات الفورية

**السيناريو:** تفعيل WebSocket في الإعدادات، ثم الاستماع لحدث `nano.fileupload.websocket.notify` لإرسال إشعارات للواجهة.

**الإعدادات في `.env`:**
```ini
NANO_FILE_UPLOAD_WEBSOCKET_ENABLED=true
NANO_FILE_UPLOAD_WEBSOCKET_CHANNEL=file-uploads
```

**الكود في التطبيق:**
```php
Event::listen('nano.fileupload.websocket.notify', function ($channel, $event, $data) {
    // استخدم مكتبة WebSocket مثل Pusher أو Laravel WebSockets
    broadcast(new \App\Events\FileUploadWebsocketEvent($channel, $event, $data));
});

$result = $service->upload(Product::class, 'image', $uploadedFile);
// بعد نجاح الرفع، سيتم إطلاق الحدث وإرسال الإشعار.
```

---

## 12. رفع ملف مؤقت واستخدامه في عدة نماذج

**السيناريو:** ملف يتم رفعه مبدئيًا (مثل صورة شخصية) ثم ربطه بأكثر من نموذج (مثلاً مستخدم ومقال).

```php
// الخطوة 1: رفع الصورة بمفتاح مؤقت
$tempKey = $service->generateTempSessionKey(User::class, 'avatar');
$result = $service->upload(User::class, 'avatar', $uploadedFile, ['temp_session_key' => $tempKey]);

if (!$result['status']) die('فشل الرفع');

// الخطوة 2: حفظ المستخدم والمقال وربط نفس الصورة
$user = new User();
$user->name = 'أحمد';
$user->save();

$article = new Article();
$article->title = 'مقال جديد';
$article->save();

// ربط الصورة بالمستخدم
$service->attachTempFiles($user, 'avatar', $tempKey);

// ربط الصورة نفسها بالمقال (إذا كان المقال له حقل image)
$service->attachTempFiles($article, 'image', $tempKey);

// ملاحظة: attachTempFiles يبحث عن جميع الملفات التي تحمل المفتاح، وسيربط جميع الملفات الموجودة.
// إذا أردت ربط نفس الملف بكلا النموذجين، سيتم تكرار الرابط (يجب التأكد من أن الملفات مستقلة).
```

---

## 13. استخدام التخزين المتعدد (Multi-Storage) عبر `storage_disk`

**السيناريو:** رفع ملف مباشرة إلى قرص S3 (أو FTP) بدلاً من القرص المحلي.

```php
// تعريف الحقل مع storage_disk (في registry)
// 'image' => ['type' => 'image', 'storage_disk' => 's3']

$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);
// سيتم تعيين $file->disk = 's3' قبل الحفظ، وسيستخدم القرص المخصص.
```

**ملاحظة:** يجب أن يكون القرص `s3` معرفاً مسبقاً في `config/filesystems.php`.

---

## 14. رفع ملفات متعددة باستخدام بيانات base64 من طلب واحد

**السيناريو:** واجهة API تستقبل مصفوفة من سلاسل base64 وترفعها كلها.

```php
$base64Array = $request->input('images'); // ['data:image/...', 'data:image/...', ...]

$result = $service->uploadMultiple(Product::class, 'gallery', $base64Array, [
    'model' => $product,
]);

if ($result['status']) {
    echo "تم رفع " . $result['process_data']['success_count'] . " صور.";
}
```

---

## 15. إضافة صور مصغرة مخصصة عند جلب الملفات

**السيناريو:** نريد أحجامًا مخصصة للصور المصغرة تختلف عن الأحجام الافتراضية.

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'icon' => [50, 50, 'crop'],
        'preview' => [200, 150, 'auto'],
        'full' => [800, 600, 'auto'],
    ],
]);

// النتيجة تحتوي على المفاتيح icon, preview, full لكل ملف.
```

---

## 16. التعامل مع أخطاء الحذف (ملف غير موجود)

```php
$result = $service->deleteFile(999999);

if (!$result['status']) {
    switch ($result['error_code']) {
        case 'FILE_UPLOAD_FILE_NOT_FOUND':
            echo "الملف غير موجود";
            break;
        default:
            echo "خطأ: " . $result['message'];
    }
}
```

---

## 17. دمج `FileUploadService` مع `FileUploadUserManager` لتخصيص صلاحيات frontend

**السيناريو:** مستخدم frontend يريد رفع صورة شخصية، نستخدم `FileUploadUserManager` للتحقق من صلاحية مخصصة.

```php
$userManager = FileUploadUserManager::instance();
$user = $userManager->getUser();

if (!$user) {
    return response()->json(['error' => 'غير مصرح'], 401);
}

// افتراض أن نموذج المستخدم مسجل في Registry مع صلاحية add للـ avatar
$result = $service->upload(User::class, 'avatar', $uploadedFile, ['model' => $user]);
```

---

## 18. استخدام `validateFile` مع بيانات base64 في واجهة API

**السيناريو:** قبل رفع الصورة عبر base64، نتحقق من صحتها.

```php
$base64 = $request->input('image_base64');

try {
    $service->validateFile(Product::class, 'image', $base64);
    // الملف صالح
    $result = $service->upload(Product::class, 'image', $base64, ['model' => $product]);
} catch (FileUploadException $e) {
    return response()->json([
        'error' => $e->getMessage(),
        'error_code' => $e->getErrorCode(),
    ], 400);
}
```

---

## 19. الحصول على الملفات المؤقتة مع التحقق الأمني الكامل

**السيناريو:** بعد رفع الملفات المؤقتة، نريد عرضها قبل حفظ النموذج، مع التأكد من أن المستخدم الحالي هو صاحب المفتاح.

```php
$tempKey = $request->input('temp_session_key');
$result = $service->getFiles(Product::class, 'gallery', null, ['temp_session_key' => $tempKey]);

if ($result['status']) {
    // الملفات تنتمي للمستخدم الحالي
    foreach ($result['data'] as $file) {
        echo "ملف: {$file['path']}<br>";
    }
} else {
    echo "خطأ: " . $result['error']; // قد يكون بسبب مفتاح غير صالح أو منتهي أو لمستخدم آخر
}
```

---

## 20. أفضل الممارسات للإصدارات 1.0.6+

1. **استخدم مفاتيح جلسة مؤقتة للنماذج الجديدة**: لتجنب فقدان الملفات إذا لم يتم حفظ النموذج.
2. **تحقق من صحة الملف قبل الرفع باستخدام `validateFile`**: الخدمة تفعل ذلك تلقائياً، لكن يمكنك استدعاؤها يدوياً أيضاً.
3. **سجل معرفات المعاملات (transaction IDs)** في قاعدة البيانات لتتبع عمليات الرفع.
4. **استخدم `uploadMultiple` للحقول المتعددة** بدلاً من استدعاء `upload` في حلقة، لأن `uploadMultiple` يجمع الأخطاء ويقدم إحصاءات.
5. **لا تعتمد على `temp_session_key` فقط**: بعد حفظ النموذج، استخدم `attachTempFiles` لنقل الملكية.
6. **تعامل مع الأخطاء حسب الكود والـ `error_code`**: قدم للمستخدم رسائل مناسبة لكل حالة.
7. **فعّل `app.debug` في بيئة التطوير** للحصول على تفاصيل الأخطاء الكاملة.
8. **استخدم `with_thumbs` فقط عند الحاجة** لتقليل حجم الاستجابة وزيادة الأداء.
9. **احتفظ بمفاتيح الجلسة المؤقتة في الجلسة (session) أو قاعدة بيانات مؤقتة** لتجنب فقدانها.
10. **اختبر الصلاحيات جيدًا**، تأكد من أن المستخدمين غير المصرح لهم لا يمكنهم رفع أو حذف الملفات.
11. **استخدم التحويلات التلقائية (`auto_resize`, `auto_watermark`) بحكمة** – قد تستهلك موارد الخادم.
12. **راقب سجل `fileupload.log`** للكشف عن محاولات رفع الملفات الخطيرة أو الأخطاء المتكررة.
13. **أنشئ مهمة مجدولة (cron)** لتنظيف الملفات منتهية الصلاحية (`expires_at`) وغير المرتبطة.

---

## 21. الأخطاء الشائعة وحلولها (محدث)

| الخطأ | السبب | الحل |
|-------|-------|------|
| `ApplicationException: الحقل ... غير مسجل للنموذج ...` | الحقل غير مسجل في `FileUploadRegistry`. | أضف الحقل في التسجيل. |
| `لا تملك صلاحية رفع الملفات لهذا الحقل` | المستخدم لا يمتلك الصلاحية `add` للحقل أو `disable_upload` مفعل. | تحقق من إعدادات `permissions` في Registry أو الإعدادات العامة، ومن صلاحيات المستخدم. |
| `بيانات الملف غير صالحة` | البيانات الممررة ليست `UploadedFile` ولا base64 صالح. | تأكد من صيغة البيانات. |
| `الملف غير موجود` (عند الحذف) | معرف الملف غير موجود. | تأكد من صحة المعرف. |
| `حجم الملف يتجاوز الحد المسموح به` | حجم الملف أكبر من `max_filesize`. | قم بضغط الملف أو زيادة `max_filesize`. |
| `نوع الملف غير مسموح به` | امتداد الملف غير مدرج في `allowed_types`. | استخدم نوع مسموح أو عدل `allowed_types`. |
| `نوع الملف ... غير مسموح به لأسباب أمنية` | الامتداد موجود في القائمة السوداء (`BLACKLISTED_EXTENSIONS`). | لا يمكن تجاوزه، استخدم أنواع ملفات آمنة. |
| `مفتاح جلسة مؤقت غير صالح أو منتهي الصلاحية` | المفتاح منتهي أو تم التلاعب به. | أعد رفع الملف أو استخدم مفتاحاً جديداً. |
| `غير مصرح لك بالوصول إلى هذه الملفات المؤقتة` | محاولة الوصول إلى ملفات مؤقتة لمستخدم آخر أو بنموذج/حقل مختلف. | تأكد من أن المفتاح يخص المستخدم الحالي والنموذج/الحقل الصحيح. |
| `Auto-resize failed` | مكتبة GD أو Imagick غير مثبتة، أو مسار الملف غير صالح. | ثبّت `php-gd` أو `php-imagick`، وتحقق من صلاحيات الملف. |
| `Auto-watermark failed` | إضافة `Nano2.Watermark` غير مثبتة أو مسار الشعار غير صحيح. | ثبّت الإضافة وتأكد من وجود ملف الشعار. |
| `Storage disk 's3' not found` | لم يتم تكوين قرص `s3` في `config/filesystems.php`. | أضف إعدادات القرص المناسبة. |

---

## 22. دمج مع نظام الصلاحيات المخصص

في تطبيقات نانوسوفت، يمكن تخصيص دالة التحقق من الصلاحيات لمستخدمي frontend عبر `FileUploadUserManager`:

```php
FileUploadUserManager::instance()->setPermissionChecker(function ($user, $operation, $permissions) {
    // منطق مخصص: مثلاً التحقق من جدول صلاحيات مخصص
    if ($user->hasRole('editor')) {
        return true;
    }
    return $user->hasPermission($permissions);
});
```

---

## الخلاصة

كلاس `FileUploadService` يقدم واجهة قوية وموحدة لإدارة رفع الملفات في تطبيقات نانوسوفت. من خلال الأمثلة المذكورة أعلاه، يمكن للمطورين تنفيذ أي سيناريو متعلق برفع الملفات: من الرفع البسيط مع الربط المباشر إلى الرفع المؤقت مع النماذج غير المحفوظة، والتحقق من الصلاحيات والقيود، وجلب الملفات مع صور مصغرة، وحذفها بأمان. بالإضافة إلى ذلك، تدعم الخدمة الميزات المتقدمة مثل التخزين المتعدد، التحويل التلقائي للصور، خطافات الأحداث، WebSocket، وأكواد الأخطاء الفريدة. نوصي باستخدام هذه الخدمة كطبقة وحيدة للتعامل مع الملفات في جميع أنحاء التطبيق لضمان الاتساق والأمان.

للاطلاع على تفاصيل هيكل الاستجابة عند استخدام متحكم `FileUploadController`، راجع [توثيق واجهة API](./Docs-API-Documentation-ar.md).

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
