# توثيق دالة `validateAndMatchTempKey`

## نظرة عامة

الدالة `validateAndMatchTempKey` هي دالة مساعدة (helper method) داخل كلاس `FileUploadService`، تهدف إلى توحيد عملية التحقق من صحة مفاتيح الجلسة المؤقتة (`temp_session_key`) ومطابقتها مع البيانات المقدمة (النموذج، الحقل، المستخدم). تم تصميم هذه الدالة لتكون مرنة وقابلة للتوسع، مع دعم الأحداث، التخزين المؤقت، تجميع الأخطاء، والتحكم الكامل في سلوك الاستثناءات.

**الغرض الرئيسي:**  
- تجنب تكرار كود التحقق في دوال متعددة مثل `deleteFile`, `getFiles`, `attachTempFiles`.
- توفير معلومات تفصيلية عن سبب فشل التحقق (للمطورين).
- إمكانية تخصيص منطق التحقق عبر الأحداث دون تعديل الكود الأساسي.

---

## التوقيع (Signature)

```php
protected function validateAndMatchTempKey(
    string|array $tempKey,
    string $modelClass,
    string $field,
    mixed $user = null,
    array $options = []
): array
```

---

## المعاملات (Parameters)

| المعامل | النوع | الوصف |
|---------|------|-------|
| `$tempKey` | `string\|array` | مفتاح الجلسة المؤقت (كنص) أو بيانات المفتاح بعد التحقق (كمصفوفة). إذا تم تمرير مصفوفة، يتم استخدامها مباشرة دون تحقق إضافي. |
| `$modelClass` | `string` | اسم الفئة الكامل للنموذج المتوقع (مثال: `Nano\Shop\Models\Product`). |
| `$field` | `string` | اسم الحقل المتوقع (مثال: `'image'`). |
| `$user` | `mixed` | (اختياري) كائن المستخدم الحالي. إذا لم يُمرر، يتم جلبه من `FileUploadUserManager`. |
| `$options` | `array` | مصفوفة خيارات إضافية (انظر الجدول أدناه). |

### خيارات `$options`

| الخيار | النوع | القيمة الافتراضية | الوصف |
|--------|------|-------------------|-------|
| `skip_user_check` | `bool` | `false` | تخطي التحقق من تطابق المستخدم (userId و userType). |
| `throw_on_failure` | `bool` | `true` | إذا كان `true`، ترمي الدالة استثناء عند الفشل. إذا `false`، تعيد مصفوفة النتيجة مع `status = false`. |
| `event_prefix` | `string` | `'nano.fileupload.tempKey'` | بادئة لأسماء الأحداث المطلقة (مثال: `nano.fileupload.tempKey.validating`). |
| `disable_events` | `bool` | `false` | تعطيل جميع الأحداث (لأغراض الأداء أو الاختبارات). |
| `strict_mode` | `bool` | `true` | وضع صارم: يتطلب تطابقاً تاماً للنموذج (`===`). إذا كان `false`، يسمح بالفئات الفرعية (`is_a`). |
| `cache_results` | `bool` | `false` | تخزين نتائج التحقق في ذاكرة مؤقتة (لكل طلب) لتجنب التحقق المتكرر من نفس المفتاح. |
| `retry_on_expiry` | `bool` | `false` | محاولة تجديد المفتاح إذا انتهت صلاحيته (يتطلب `regenerate_callback`). |
| `regenerate_callback` | `callable\|null` | `null` | دالة تُستدعى لتوليد مفتاح جديد عند انتهاء الصلاحية. تستقبل المفتاح المنتهي وتعيد مفتاحاً جديداً. |
| `stop_on_first_failure` | `bool` | `true` | إذا `true`، يتوقف عند أول خطأ ويرمي استثناء. إذا `false`، يجمع كل الأخطاء ويرمي استثناء واحداً في النهاية. |
| `allow_expired_key` | `bool` | `false` | السماح باستخدام المفاتيح منتهية الصلاحية ضمن فترة سماح. |
| `expiry_grace_period` | `int` | `300` | فترة السماح بالثواني (عند `allow_expired_key = true`). |
| `custom_validator` | `callable\|null` | `null` | دالة تحقق إضافية تُستدعى بعد التحقق الأساسي. تستقبل `$keyData, $modelClass, $field, $user, $options` وتعيد `bool`. |
| `key_data_overrides` | `array` | `[]` | تجاوزات لبيانات المفتاح. مثال: `['userId' => fn($v) => 'new_user_id']`. |
| `before_validation` | `callable\|null` | `null` | دالة تُستدعى قبل بدء عملية التحقق. تستقبل `$tempKey, $modelClass, $field, $options`. |
| `after_validation` | `callable\|null` | `null` | دالة تُستدعى بعد انتهاء التحقق. تستقبل `$result`. |
| `events_payload` | `array` | `[]` | بيانات إضافية تُمرر إلى جميع الأحداث. |
| `log_failures_only` | `bool` | `true` | إذا `true`، يتم تسجيل حالات الفشل فقط في السجل. إذا `false`، يتم تسجيل جميع المحاولات. |
| `collect_metadata` | `bool` | `false` | جمع بيانات أداء (مدة التنفيذ، الخطوات المنفذة) وإضافتها إلى `meta`. |

---

## القيمة المُرجعة (Return Value)

الدالة تعيد **مصفوفة** بنفس هيكل الاستجابة الموحد في جميع دوال `FileUploadService`. تحتوي المصفوفة على المفاتيح التالية:

| المفتاح | النوع | الوصف |
|---------|------|-------|
| `code` | `int` | كود HTTP (200 للنجاح، 400/403/500 للفشل). |
| `status` | `bool` | `true` إذا نجح التحقق، `false` إذا فشل. |
| `message` | `string` | رسالة توضيحية (قابلة للترجمة). |
| `error` | `string\|null` | تفاصيل الخطأ (في حالة الفشل). |
| `errors` | `array` | مصفوفة بأخطاء إضافية (مثل سياق الاستثناء). |
| `data` | `array` | يحتوي على `key_data` (بيانات المفتاح) و `matched_data` (نتائج المطابقة لكل خطوة). |
| `meta` | `array` | بيانات وصفية (مثل مدة التنفيذ إذا طُلب). |
| `input_data` | `array` | نسخة من البيانات المدخلة (للتتبع). |
| `process_data` | `array` | معلومات داخلية عن عملية التحقق. |
| `debug` | `array\|null` | تفاصيل التصحيح (تظهر فقط عند `app.debug = true`). |

### هيكل `data` في حالة النجاح:

```php
'data' => [
    'key_data' => [
        'modelClass' => '...',
        'field' => '...',
        'userId' => '...',
        'userType' => '...',
        'timestamp' => 1234567890,
    ],
    'matched_data' => [
        'model_matched' => true,
        'field_matched' => true,
        'user_matched' => true,
        'user_id_matched' => true,
        'user_type_matched' => true,
    ]
]
```

### هيكل `data` في حالة الفشل (مع `throw_on_failure = false`):

```php
'data' => [
    'key_data' => [ /* قد يكون null أو بيانات جزئية */ ],
    'matched_data' => [
        'model_matched' => false,
        'field_matched' => true,  // قد تختلف القيم حسب الخطوة التي فشلت
        'user_matched' => false,
        // ...
    ]
]
```

---

## الأحداث (Events)

الدالة تطلق الأحداث التالية (يمكن الاستماع إليها لتوسيع السلوك):

| اسم الحدث | وقت الإطلاق | المعاملات |
|-----------|-------------|------------|
| `{prefix}.validating` | قبل بدء أي تحقق | `$tempKey, $modelClass, $field, $options, &$result, &$process_data, $eventsPayload` |
| `{prefix}.beforeUserCheck` | قبل التحقق من المستخدم (يمكن تعديل `$user` أو تجاوز التحقق) | `$keyData, $modelClass, $field, &$user, &$skipCheck, &$result, &$process_data, $eventsPayload` |
| `{prefix}.afterUserCheck` | بعد التحقق من المستخدم | `$keyData, $modelClass, $field, $userMatched, $userIdMatched, $userTypeMatched, &$result, &$process_data, $eventsPayload` |
| `{prefix}.validated` | عند نجاح التحقق | `$keyData, $modelClass, $field, $result, $eventsPayload` |
| `{prefix}.failed` | عند فشل التحقق (استثناء مخصص) | `$exception, $modelClass, $field, $result, $eventsPayload` |
| `{prefix}.error` | عند خطأ غير متوقع | `$exception, $modelClass, $field, $result, $eventsPayload` |

**ملاحظة:** يمكن تغيير البادئة (`{prefix}`) عبر خيار `event_prefix`.

---

## أمثلة عملية

### 1. الاستخدام الأساسي في `deleteFile` (مع رمي استثناء عند الفشل)

```php
public function deleteFile($fileId, $modelClass = null, $field = null, $user = null): array
{
    // ... (إعداد النتيجة)
    try {
        // ... (البحث عن الملف)

        // التحقق من المفتاح المؤقت (إذا كان الملف مؤقتاً)
        if ($file->session_key) {
            $validation = $this->validateAndMatchTempKey(
                $file->session_key,
                $modelClass ?? $file->attachment_type,
                $field ?? $file->field,
                $user,
                ['throw_on_failure' => true] // الافتراضي
            );
            // في حال النجاح، $validation['data']['key_data'] يحتوي على بيانات المفتاح
        }
        // ...
    } catch (FileUploadException $e) {
        // معالجة الاستثناء
    }
}
```

### 2. استخدام مع `throw_on_failure = false` (تحقيق الأخطاء دون توقف)

```php
public function attachTempFiles(Model $model, string $field, string $tempSessionKey, array $options = []): array
{
    // ...
    try {
        $validation = $this->validateAndMatchTempKey(
            $tempSessionKey,
            get_class($model),
            $field,
            null,
            ['throw_on_failure' => false, 'stop_on_first_failure' => false]
        );

        if (!$validation['status']) {
            // فشل التحقق، نرجع النتيجة مباشرة
            return $validation;
        }

        $keyData = $validation['data']['key_data'];
        // ... متابعة عملية الربط
    } catch (Exception $e) {
        // معالجة الاستثناءات غير المتوقعة
    }
}
```

### 3. استخدام التخزين المؤقت لتجنب التحقق المتكرر

```php
// في دالة getFiles، قد يُستدعى نفس المفتاح عدة مرات في طلب واحد
$validation = $this->validateAndMatchTempKey(
    $tempKey,
    $modelClass,
    $field,
    null,
    ['cache_results' => true]
);
```

### 4. السماح بالمفاتيح منتهية الصلاحية ضمن فترة سماح

```php
$validation = $this->validateAndMatchTempKey(
    $tempKey,
    $modelClass,
    $field,
    null,
    [
        'allow_expired_key' => true,
        'expiry_grace_period' => 600, // 10 دقائق
    ]
);
```

### 5. استخدام `custom_validator` للتحقق من صلاحية إضافية (مثل عنوان IP)

```php
$validation = $this->validateAndMatchTempKey($tempKey, $modelClass, $field, null, [
    'custom_validator' => function($keyData, $modelClass, $field, $user) {
        // التحقق من أن المفتاح صدر من نفس عنوان IP الحالي
        $allowedIp = $keyData['ip'] ?? null;
        return $allowedIp === request()->ip();
    }
]);
```

### 6. تجاوز بيانات المفتاح (`key_data_overrides`)

```php
// في حالة ترحيل المستخدمين، نحتاج إلى تعيين userId جديد للمفاتيح القديمة
$validation = $this->validateAndMatchTempKey($tempKey, $modelClass, $field, null, [
    'key_data_overrides' => [
        'userId' => function($oldUserId) {
            return UserMigration::getNewId($oldUserId);
        }
    ]
]);
```

### 7. الاستماع إلى حدث `beforeUserCheck` لتخصيص منطق المستخدم

```php
// في `boot()` من Plugin.php
Event::listen('nano.fileupload.tempKey.beforeUserCheck', function ($keyData, $modelClass, $field, &$user, &$skipCheck, &$result, &$processData) {
    if ($modelClass === 'Nano\Shop\Models\Product' && $field === 'image') {
        $currentUser = $user ?? BackendAuth::getUser();
        if ($currentUser && $currentUser->hasAccess('nano.shop.manage_all_products')) {
            // السماح للمدير بالوصول إلى ملفات أي منتج
            $skipCheck = true;
            $processData['user_check_overridden'] = true;
            return true;
        }
    }
    return false;
});
```

### 8. تجميع الأخطاء لمعرفة جميع أسباب الفشل (للواجهات الأمامية)

```php
$validation = $this->validateAndMatchTempKey($tempKey, $modelClass, $field, null, [
    'throw_on_failure' => false,
    'stop_on_first_failure' => false,
]);

if (!$validation['status']) {
    $errors = $validation['errors']; // مصفوفة تحتوي على كل الأخطاء
    foreach ($errors as $error) {
        if ($error['step'] === 'model') {
            echo "النموذج غير متطابق: متوقع {$error['expected']}، موجود {$error['actual']}";
        } elseif ($error['step'] === 'user_id') {
            echo "معرف المستخدم غير متطابق";
        }
    }
}
```

---

## مثال متكامل في `FileUploadService::attachTempFiles`

```php
public function attachTempFiles(Model $model, string $field, string $tempSessionKey, array $options = []): array
{
    $input_data = $options;
    $process_data = [];
    $result = [
        'code' => 200,
        'status' => true,
        'message' => trans('nano.fileupload::lang.public.helpers.upload.msg_attach_success'),
        'error' => null,
        'errors' => [],
        'data' => null,
        'meta' => null,
        'input_data' => $input_data,
        'process_data' => [],
        'debug' => null,
    ];

    try {
        // التحقق من صحة المفتاح ومطابقته للنموذج والحقل والمستخدم
        $validation = $this->validateAndMatchTempKey(
            $tempSessionKey,
            get_class($model),
            $field,
            null,
            [
                'throw_on_failure' => true,
                'cache_results' => true,
                'event_prefix' => 'nano.fileupload.attachTempFiles',
            ]
        );

        $keyData = $validation['data']['key_data'];
        $process_data = array_merge($process_data, $validation['process_data']);

        // البحث عن الملفات المؤقتة
        $files = File::where('field', $field)
                     ->where('session_key', $tempSessionKey)
                     ->get();

        if ($files->isEmpty()) {
            throw new FileUploadException(
                trans('nano.fileupload::lang.public.helpers.upload.no_temp_files_found'),
                FileUploadException::ERR_FILE_NOT_FOUND,
                ['temp_session_key' => $tempSessionKey, 'field' => $field]
            );
        }

        // ربط الملفات
        $attachedFiles = [];
        foreach ($files as $file) {
            $file->attachment_id = $model->getKey();
            $file->attachment_type = $model->getMorphClass();
            $file->session_key = null;
            $file->expires_at = null;
            $file->save();

            if ($model->hasRelation($field)) {
                $model->{$field}()->add($file);
            }

            $attachedFiles[] = $file->toArray();
        }

        $result['data'] = $attachedFiles;
        $result['process_data'] = array_merge($process_data, [
            'attached_count' => count($attachedFiles),
            'temp_session_key' => $tempSessionKey,
        ]);
        $result['message'] = trans('nano.fileupload::lang.public.helpers.upload.msg_attach_success', ['count' => count($attachedFiles)]);

        Event::fire('nano.fileupload.afterAttach', [$model, $field, $attachedFiles, $tempSessionKey]);

    } catch (FileUploadException $e) {
        // معالجة موحدة
        $result['code'] = $e->getCode() ?: 400;
        $result['status'] = false;
        $result['message'] = $e->getMessage();
        $result['error'] = $e->getMessage();
        $result['error_code'] = $e->getErrorCode();
        $result['errors'] = $e->getContext() ?: [];
        // ... إضافة debug إذا لزم الأمر
    } catch (Throwable $e) {
        // معالجة الأخطاء العامة
        $result['code'] = 500;
        $result['status'] = false;
        $result['message'] = trans('nano.fileupload::lang.errors.general_error');
        $result['error'] = $e->getMessage();
        // ... إضافة debug
    }

    return $result;
}
```

---

## ملخص

الدالة `validateAndMatchTempKey` توفر طريقة موحدة ومرنة للتحقق من صحة مفاتيح الجلسة المؤقتة. بفضل الخيارات المتعددة والأحداث القابلة للتوسع، يمكن للمطورين:

- تجنب تكرار الكود.
- تخصيص منطق التحقق حسب احتياجات التطبيق (مثل السماح للمديرين بالوصول إلى ملفات أي مستخدم).
- جمع معلومات تفصيلية عن أسباب الفشل.
- تحسين الأداء باستخدام التخزين المؤقت.

تستخدم هذه الدالة داخل `FileUploadService` في دوال `deleteFile`, `getFiles`, `attachTempFiles` لضمان أمان وسلامة العمليات.

## Additional Documentation

- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-ar.md)
- [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advanced-Examples-ar.md)
