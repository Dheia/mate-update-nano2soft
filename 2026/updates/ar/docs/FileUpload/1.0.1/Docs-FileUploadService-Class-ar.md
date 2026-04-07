## توثيق كلاس `FileUploadService`

**نبذة تعريفية عن حزمة `Nano.FileUpload` ووحدة إدارة رفع الملفات**

هذه المجموعة من الكلاسات (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager` وغيرها) هي جزء من **حزمة `Nano.FileUpload`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **نانوسوفت** بهدف توحيد وإدارة عمليات رفع الملفات عبر واجهات برمجة التطبيقات (API) بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano\FileUpload\Classes
```

توفر هذه الوحدة أدوات متقدمة لتسجيل النماذج (الموديولات) التي تحتوي على حقول رفع ملفات، والتحقق من صلاحيات المستخدمين (بما في ذلك أنواع المستخدمين المختلفة: `backend` و `frontend`)، وإدارة الملفات المرفوعة مؤقتًا قبل حفظ السجل، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة رفع ملفات قابلة للتوسع بسهولة وتلبية احتياجات التطبيقات المتقدمة دون المساس بأمان التطبيق أو أدائه.

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

---

### أهم الطرق (Methods)

#### 1. إنشاء مفتاح جلسة مؤقت

##### `public function generateTempSessionKey(string $modelClass, string $field, $userId = null): string`
- **الهدف**: توليد مفتاح جلسة مؤقت فريد لربط الملفات قبل حفظ النموذج.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$userId`: (اختياري) معرف المستخدم، إذا لم يُمرر يستخدم المستخدم الحالي.
- **السلوك**:
  - يحصل على معرف المستخدم الحالي أو يستخدم `guest`.
  - يُنشئ مفتاحًا بتنسيق `tmp_{md5(modelClass+field+userId+time+rand)}`.
- **الإرجاع**: مفتاح الجلسة المؤقت.
- **مثال**:
  ```php
  $tempKey = $service->generateTempSessionKey('Nano\Shop\Models\Product', 'image');
  ```

#### 2. رفع ملف واحد

##### `public function upload(string $modelClass, string $field, $fileData, array $options = []): array`
- **الهدف**: رفع ملف واحد لحقل معين.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$fileData`: بيانات الملف (كائن `UploadedFile` أو سلسلة base64).
  - `$options`: مصفوفة خيارات إضافية:
    - `model`: كائن النموذج (إذا كان موجودًا ومحفوظًا).
    - `temp_session_key`: مفتاح جلسة مؤقت (لتجاوز التوليد التلقائي).
    - `...`: أي خيارات إضافية تُمرر إلى `Base64::onUpload`.
- **السلوك**:
  - يتحقق من تسجيل النموذج والحقل في `FileUploadRegistry`.
  - يتحقق من صلاحية `add` للمستخدم الحالي.
  - يحدد ما إذا كان النموذج موجودًا ومحفوظًا (لربط مباشر) أو يحتاج إلى مفتاح جلسة مؤقت.
  - يستدعي `Base64::onUpload` لتنفيذ الرفع الفعلي.
  - يُضيف `temp_session_key` إلى النتيجة إذا تم استخدامه.
  - يعيد مصفوفة بنفس هيكل `Base64::onUpload` (مع `status`, `data`, `message`, ...).
- **الإرجاع**: مصفوفة تحتوي على نتيجة العملية.
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
  }
  ```

#### 3. رفع عدة ملفات (لحقل متعدد)

##### `public function uploadMultiple(string $modelClass, string $field, array $filesData, array $options = []): array`
- **الهدف**: رفع عدة ملفات لحقل متعدد في طلب واحد.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$filesData`: مصفوفة من بيانات الملفات (كل عنصر كما في `upload`).
  - `$options`: نفس خيارات `upload`.
- **السلوك**:
  - يتكرر على `$filesData` ويستدعي `upload()` لكل ملف.
  - يجمع النتائج ويحسب عدد النجاحات.
  - يعيد مصفوفة تحتوي على `data` (نتائج كل ملف) و `process_data` بإحصائيات.
- **الإرجاع**: مصفوفة تحتوي على `code`, `status`, `message`, `data`, `process_data`.
- **مثال**:
  ```php
  $files = [ $file1, $file2, $file3 ];
  $result = $service->uploadMultiple('Product', 'gallery', $files, ['temp_session_key' => $tempKey]);
  echo "نجح رفع " . $result['process_data']['success_count'] . " من " . $result['process_data']['total'];
  ```

#### 4. حذف ملف

##### `public function deleteFile(int $fileId, ?string $modelClass = null, ?string $field = null, $user = null): array`
- **الهدف**: حذف ملف محدد.
- **المعاملات**:
  - `$fileId`: معرف الملف.
  - `$modelClass`: (اختياري) اسم الفئة للتحقق من صلاحية الحذف.
  - `$field`: (اختياري) اسم الحقل للتحقق من صلاحية الحذف.
  - `$user`: (اختياري) كائن المستخدم، إذا لم يُمرر يستخدم الحالي.
- **السلوك**:
  - يبحث عن الملف باستخدام `File::find()`.
  - إذا تم تمرير `$modelClass` و `$field`، يتحقق من صلاحية `delete` للمستخدم الحالي.
  - يحذف الملف.
- **الإرجاع**: مصفوفة تحتوي على `code`, `status`, `message`, `error`.
- **مثال**:
  ```php
  $result = $service->deleteFile(123, 'Nano\Shop\Models\Product', 'image');
  if ($result['status']) {
      echo "تم حذف الملف بنجاح";
  }
  ```

#### 5. جلب الملفات

##### `public function getFiles(string $modelClass, string $field, ?int $modelId = null, array $options = []): array`
- **الهدف**: استرجاع الملفات المرتبطة بنموذج وحقل معين.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$modelId`: (اختياري) معرف النموذج المحفوظ.
  - `$options`: خيارات إضافية:
    - `temp_session_key`: مفتاح جلسة مؤقت للبحث عن الملفات غير المرتبطة.
    - `with_thumbs`: `bool` لتضمين الصور المصغرة.
    - `thumb_sizes`: مصفوفة بأحجام الصور المصغرة (مثل `['thumb' => [100,100]]`).
- **السلوك**:
  - يتحقق من تسجيل الحقل وصلاحية `view`.
  - يبني استعلامًا على `File` مع تصفية حسب `field` و (`attachment_id` + `attachment_type` إذا كان `modelId` موجودًا) أو `session_key` إذا كان `temp_session_key` موجودًا.
  - يعيد الملفات مع البيانات الأساسية (id, title, description, path, size, content_type) وصور مصغرة إذا طُلب.
- **الإرجاع**: مصفوفة تحتوي على `code`, `status`, `message`, `data`.
- **مثال**:
  ```php
  $files = $service->getFiles('Nano\Shop\Models\Product', 'image', 456);
  foreach ($files['data'] as $file) {
      echo "<img src='{$file['path']}'>";
  }
  ```

#### 6. ربط الملفات المؤقتة بنموذج محفوظ

##### `public function attachTempFiles(Model $model, string $field, string $tempSessionKey): bool`
- **الهدف**: نقل الملفات التي تم رفعها باستخدام مفتاح جلسة مؤقت إلى نموذج محفوظ.
- **المعاملات**:
  - `$model`: كائن النموذج المحفوظ.
  - `$field`: اسم الحقل.
  - `$tempSessionKey`: مفتاح الجلسة المؤقت المستخدم أثناء الرفع.
- **السلوك**:
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

#### 7. التحقق من صحة الملف (حجم، نوع)

##### `public function validateFile(string $modelClass, string $field, $fileData): bool`
- **الهدف**: التحقق من أن الملف يطابق قيود الحقل المسجل (الحجم الأقصى، الأنواع المسموحة).
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$field`: اسم الحقل.
  - `$fileData`: بيانات الملف (كائن `UploadedFile` أو سلسلة base64).
- **السلوك**:
  - يستدعي `getFieldConstraints()` من `FileUploadRegistry`.
  - يقارن حجم الملف مع `max_filesize`.
  - يقارن امتداد الملف مع `allowed_types`.
- **الإرجاع**: `true` إذا كان الملف صالحًا، وإلا يلقي `ApplicationException`.
- **مثال**:
  ```php
  try {
      $service->validateFile('Product', 'image', $uploadedFile);
      // يمكن رفع الملف
  } catch (ApplicationException $e) {
      echo "الملف غير صالح: " . $e->getMessage();
  }
  ```

---

### الأحداث (Events)

| اسم الحدث | الوصف |
|----------|-------|
| `nano.api.fileupload.beforeUpload` | (اختياري، غير مطبق حاليًا) يُطلق قبل تنفيذ الرفع، يمكن استخدامه لتعديل البيانات أو تسجيل. |
| `nano.api.fileupload.afterUpload` | (اختياري، غير مطبق حاليًا) يُطلق بعد نجاح الرفع، لتحديث بيانات إضافية. |

---

### أمثلة عملية شاملة

#### مثال 1: رفع صورة رئيسية لمنتج جديد باستخدام مفتاح جلسة مؤقت

```php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadUserManager;
use Nano\Shop\Models\Product;

$service = FileUploadService::instance();
$userManager = FileUploadUserManager::instance();

$modelClass = Product::class;
$field = 'image';

// 1. إنشاء مفتاح جلسة مؤقت
$tempKey = $service->generateTempSessionKey($modelClass, $field, $userManager->getId());

// 2. رفع الصورة (بافتراض أن $uploadedFile هو كائن UploadedFile)
$uploadResult = $service->upload($modelClass, $field, $uploadedFile, [
    'temp_session_key' => $tempKey,
    'title' => 'صورة المنتج', // اختياري
    'description' => 'وصف الصورة',
]);

if (!$uploadResult['status']) {
    die("فشل الرفع: " . $uploadResult['error']);
}

// 3. إنشاء المنتج وحفظه
$product = new Product();
$product->name = 'منتج جديد';
$product->save();

// 4. ربط الصورة بالمنتج
$service->attachTempFiles($product, $field, $tempKey);

echo "تم رفع الصورة وربطها بالمنتج رقم {$product->id}";
```

#### مثال 2: رفع عدة صور لمعرض منتج موجود

```php
$product = Product::find(10); // منتج موجود

// رفع مجموعة صور
$filesData = [
    $file1,
    $file2,
    $file3,
];

$result = $service->uploadMultiple(Product::class, 'gallery', $filesData, [
    'model' => $product, // ربط مباشر لأن المنتج موجود
]);

if ($result['status']) {
    echo "تم رفع " . $result['process_data']['success_count'] . " صور بنجاح";
} else {
    echo "فشل رفع بعض الصور: " . $result['error'];
}
```

#### مثال 3: جلب جميع صور معرض منتج مع صور مصغرة

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

#### مثال 4: حذف صورة رئيسية بعد التحقق من الصلاحية

```php
$result = $service->deleteFile(123, Product::class, 'image');

if ($result['status']) {
    echo "تم حذف الصورة بنجاح";
} else {
    echo "خطأ: " . $result['message'];
}
```

#### مثال 5: التحقق من صحة الملف قبل الرفع

```php
try {
    $service->validateFile(Product::class, 'image', $uploadedFile);
    // الملف صالح
} catch (ApplicationException $e) {
    echo "الملف غير صالح: " . $e->getMessage();
    return;
}

$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);
```

#### مثال 6: استخدام `validateFile` مع بيانات base64

```php
$base64String = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...';
$fileData = $base64String;

try {
    $service->validateFile(Product::class, 'image', $fileData);
    echo "الملف صالح";
} catch (ApplicationException $e) {
    echo "الملف غير صالح: " . $e->getMessage();
}
```

#### مثال 7: التعامل مع أخطاء الرفع

```php
$result = $service->upload(Product::class, 'image', $fileData, []);

if (!$result['status']) {
    switch ($result['code']) {
        case 400:
            echo "طلب غير صالح: " . $result['error'];
            break;
        case 401:
            echo "غير مصرح: " . $result['error'];
            break;
        case 403:
            echo "صلاحية ممنوعة: " . $result['error'];
            break;
        default:
            echo "خطأ غير معروف: " . $result['error'];
    }
}
```

#### مثال 8: دمج `FileUploadService` مع واجهة برمجة تطبيقات (API)

في متحكم API:

```php
public function uploadImage(Request $request)
{
    $modelClass = $request->input('model_class');
    $field = $request->input('field');
    $file = $request->file('file');

    $service = FileUploadService::instance();

    // التحقق من الصلاحيات والقيود
    $result = $service->upload($modelClass, $field, $file, [
        'temp_session_key' => $request->input('temp_session_key')
    ]);

    if ($result['status']) {
        return response()->json([
            'success' => true,
            'data' => $result['data'],
            'temp_session_key' => $result['temp_session_key'] ?? null,
        ]);
    } else {
        return response()->json([
            'success' => false,
            'error' => $result['error'],
            'code' => $result['code'],
        ], $result['code']);
    }
}
```

---

### التفاعل مع الكلاسات الأخرى

- **مع `FileUploadRegistry`**: يستخدم `$registry` للحصول على إعدادات الحقول (`getFieldConfig`, `getFieldConstraints`) والتحقق من الصلاحيات (`can`).
- **مع `FileUploadUserManager`**: يستخدم `$userManager` للحصول على المستخدم الحالي ونوعه (`getUser`, `getUserType`).
- **مع `Base64` (من `Nano.Api`)**: يستدعي `Base64::onUpload`, `Base64::is_base64`, `Base64::base64ToUploadedFile`, `Base64::getBase64ImageSize`, `Base64::getDataInBase64`.

---

### أفضل الممارسات

1. **استخدم مفتاح جلسة مؤقت دائمًا للنماذج الجديدة**: لتجنب فقدان الملفات إذا لم يتم حفظ النموذج بعد.
2. **تحقق من صحة الملف قبل الرفع باستخدام `validateFile`**: لتقديم رسائل خطأ واضحة للمستخدم.
3. **سجل معرفات المعاملات (transaction IDs)**: لتتبع عمليات الرفع وحالة الملفات.
4. **استخدم `uploadMultiple` للحقول المتعددة**: بدلاً من استدعاء `upload` في حلقة، لأن `uploadMultiple` يعالج الأخطاء ويقدم إحصاءات.
5. **لا تعتمد على `temp_session_key` فقط**: بعد حفظ النموذج، استخدم `attachTempFiles` لنقل الملكية.
6. **تعامل مع الأخطاء حسب الكود**: 400 (طلب خاطئ)، 401 (غير مصرح)، 403 (ممنوع)، 404 (غير موجود)، 500 (خطأ داخلي).
7. **فعّل `app.debug` في بيئة التطوير**: للحصول على تفاصيل الأخطاء الكاملة.
8. **استخدم `with_thumbs` فقط عند الحاجة**: لتقليل حجم الاستجابة وزيادة الأداء.
9. **احتفظ بمفاتيح الجلسة المؤقتة في الجلسة (session) أو قاعدة بيانات مؤقتة**: لتجنب فقدانها.
10. **اختبر الصلاحيات جيدًا**: تأكد من أن المستخدمين غير المصرح لهم لا يمكنهم رفع أو حذف الملفات.

---

### الخلاصة

كلاس `FileUploadService` هو العمود الفقري لعمليات رفع الملفات في تطبيقات نانوسوفت. يوفر واجهة موحدة وآمنة لرفع وحذف واسترجاع الملفات، مع دعم كامل لأنواع المستخدمين المختلفة والصلاحيات الدقيقة، والربط المؤقت للنماذج غير المحفوظة. بفضل اعتماده على `FileUploadRegistry` و `FileUploadUserManager`، يمكن للمطورين بناء أنظمة رفع ملفات قوية وقابلة للتوسع دون القلق بشأن التفاصيل التنفيذية أو أمان البيانات.

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)


