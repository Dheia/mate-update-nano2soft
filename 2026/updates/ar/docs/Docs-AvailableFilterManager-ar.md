# توثيق كلاس `AvailableFilterManager`

## مقدمة

كلاس `AvailableFilterManager` هو كلاس عام (غير مرتبط بموديل معين) مصمم لإدارة وإعداد بيانات الفلاتر (available filters) بشكل موحد. يقوم الكلاس بالمهام التالية:

- تحميل الفلاتر من مصادر متعددة: مصفوفة (Array)، نص JSON، ملف إعدادات (Config).
- تطبيع بيانات الفلاتر: إكمال الحقول المفقودة بقيم افتراضية، وتحويل الحقول النصية إلى صيغ متعددة اللغات.
- التحقق من صحة الفلاتر والتأكد من مطابقتها للـ schema المطلوبة.
- دمج مجموعات فلاتر متعددة.
- ترتيب الفلاتر حسب `sort_order`.
- تصفية الفلاتر النشطة فقط (`is_active == true`).
- البحث عن فلتر بواسطة `id`.
- إعادة ترتيب الفلاتر.
- الكشف عن التكرارات في `id` أو `name`.
- جلب الخيارات (options) للفلاتر من مصادر البيانات الديناميكية (API، قاعدة بيانات، ثابتة، دوال).
- تحويل الفلاتر إلى JSON.
- الحصول على JSON schema الخاص بالفلاتر.

---

## طريقة الاستخدام الأساسية

### إنشاء كائن من الكلاس
```php
use Nano\Tags\Classes\AvailableFilterManager;

$manager = new AvailableFilterManager();
```
يمكن تمرير مصفوفة خيارات للـ constructor لتخصيص اللغات المدعومة أو مفتاح الإعدادات الافتراضي:
```php
$manager = new AvailableFilterManager([
    'supportedLanguages' => ['ar', 'en'],      // اللغات المدعومة
    'defaultConfigKey'   => 'custom.config.key' // مفتاح إعدادات مخصص
]);
```

### التحميل السريع باستخدام الدالة المساعدة `load`
```php
$filters = AvailableFilterManager::load($source);
```
حيث يمكن أن يكون `$source`:
- مصفوفة PHP.
- نص JSON.
- مفتاح إعدادات (string) مثل `'nano.tags::available_filter.shop'`.

---

## دوال التحميل

### 1. `fromArray(array $filters)`
**الوصف**: تحميل الفلاتر من مصفوفة عادية.

**المدخلات**:
- `$filters`: مصفوفة تحتوي على بيانات الفلاتر (يمكن أن تكون على شكل `[['id'=>'x', ...], ...]` أو `['available_filter' => [...]]`).

**المخرجات**: مصفوفة من الفلاتر بعد تطبيق عملية التطبيع (إكمال الحقول المفقودة، تحويل الحقول متعددة اللغات، إلخ).

**مثال**:
```php
$input = [
    ['id' => 'country_id', 'name' => 'country_id', 'label' => 'الدولة', 'type' => 'select']
];
$filters = $manager->fromArray($input);
// الناتج: مصفوفة مكتملة تحتوي على جميع الحقول الافتراضية (sort_order, placeholder, comment, ...)
```

### 2. `fromJson(string $json)`
**الوصف**: تحميل الفلاتر من نص JSON.

**المدخلات**:
- `$json`: نص JSON يمثل الفلاتر.

**المخرجات**: مصفوفة الفلاتر بعد التطبيع.

**مثال**:
```php
$json = '[{"id":"country_id","name":"country_id","label":"الدولة","type":"select"}]';
$filters = $manager->fromJson($json);
```

### 3. `fromConfig(string $key = null, string $type = null)`
**الوصف**: تحميل الفلاتر من ملف إعدادات Laravel.

**المدخلات**:
- `$key`: مفتاح الإعدادات (إذا كان `null` يستخدم `defaultConfigKey`).
- `$type`: جزء إضافي من المفتاح (مثل 'shop' أو 'products').

**المخرجات**: مصفوفة الفلاتر بعد التطبيع.

**مثال**:
```php
// تحميل الفلاتر من config/nano/tags/available_filter.php تحت مفتاح 'shop'
$filters = $manager->fromConfig('nano.tags::available_filter', 'shop');
```

### 4. `static load($source, array $options = [])`
**الوصف**: دالة مساعدة لتحميل الفلاتر من مصدر غير محدد النوع.

**المدخلات**:
- `$source`: مصفوفة، نص JSON، أو مفتاح إعدادات.
- `$options`: مصفوفة خيارات لتهيئة الكلاس (اختياري).

**المخرجات**: مصفوفة الفلاتر بعد التطبيع.

**مثال**:
```php
// من مصفوفة
$filters1 = AvailableFilterManager::load($array);

// من JSON
$filters2 = AvailableFilterManager::load($jsonString);

// من إعدادات
$filters3 = AvailableFilterManager::load('nano.tags::available_filter.shop');
```

---

## دوال معالجة الفلاتر

### 5. `normalizeFilters($filters)`
**الوصف**: تطبيع الفلاتر (تحويلها إلى مصفوفة من السجلات الكاملة). تستخدم داخلياً في دوال التحميل.

**المدخلات**:
- `$filters`: بيانات الفلاتر (مصفوفة أو أي شيء آخر).

**المخرجات**: مصفوفة من الفلاتر بعد إنشاء كل سجل باستخدام `createRecord()` وترتيبها.

### 6. `createRecord(array $data)`
**الوصف**: إنشاء سجل فلتر كامل بالاستناد إلى البيانات المدخلة وإضافة القيم الافتراضية والتحقق من الصحة.

**المدخلات**:
- `$data`: مصفوفة تحتوي على بعض خصائص الفلتر (مثل `id`, `name`, `type`, ...).

**المخرجات**: مصفوفة كاملة تمثل الفلتر بعد إضافة جميع الحقول الافتراضية وتطبيق التحقق.

**مثال**:
```php
$record = $manager->createRecord(['id' => 'price', 'type' => 'number']);
/* الناتج:
[
    'id' => 'price',
    'name' => 'price',
    'type' => 'number',
    'label' => [],
    'placeholder' => null,
    'comment' => null,
    'commentPosition' => 'below',
    'commentHtml' => false,
    'default' => 0,               // تمت إضافته تلقائياً لأن type=number
    'dependsOn' => null,
    'trigger' => null,
    'is_active' => true,
    'showSearch' => false,
    'sort_order' => 0,
    'options' => null,
    'data_source' => null,
    ...
]
*/
```

### 7. `validateRecord(array $record, $throw = true)`
**الوصف**: التحقق من صحة سجل فلتر واحد.

**المدخلات**:
- `$record`: مصفوفة تمثل الفلتر.
- `$throw`: إذا كان `true` يقوم برمي استثناء عند وجود أخطاء.

**المخرجات**: مصفوفة `['valid' => bool, 'errors' => array]` (إذا كان `$throw = false`).

**مثال**:
```php
$validation = $manager->validateRecord($record, false);
if (!$validation['valid']) {
    print_r($validation['errors']);
}
```

### 8. `validateAll(array $filters)`
**الوصف**: التحقق من صحة مجموعة كاملة من الفلاتر.

**المدخلات**:
- `$filters`: مصفوفة الفلاتر.

**المخرجات**: مصفوفة `['valid' => bool, 'errors' => array]` حيث المفاتيح تشير إلى فهرس العنصر المخالف.

**مثال**:
```php
$result = $manager->validateAll($filters);
if (!$result['valid']) {
    foreach ($result['errors'] as $index => $errors) {
        echo "خطأ في الفلتر $index: " . implode(', ', $errors);
    }
}
```

### 9. `sort(array $filters)`
**الوصف**: ترتيب الفلاتر تصاعدياً حسب `sort_order`.

**المدخلات**:
- `$filters`: مصفوفة الفلاتر.

**المخرجات**: مصفوفة الفلاتر مرتبة.

### 10. `merge(array $filters1, array $filters2, $override = true)`
**الوصف**: دمج مجموعتي فلاتر. إذا وجد فلتر بنفس `id` في المجموعتين، يتم استبداله بقيمة `$filters2` إذا كان `$override = true`.

**المدخلات**:
- `$filters1`, `$filters2`: مصفوفتا الفلاتر.
- `$override`: استبدال الفلاتر الموجودة.

**المخرجات**: مصفوفة الفلاتر المدمجة (مرتبة).

**مثال**:
```php
$merged = $manager->merge($defaultFilters, $userFilters);
```

### 11. `getActiveOnly(array $filters)`
**الوصف**: إرجاع الفلاتر النشطة فقط (حيث `is_active == true`).

**المدخلات**:
- `$filters`: مصفوفة الفلاتر.

**المخرجات**: مصفوفة تحتوي على الفلاتر النشطة.

### 12. `findById(array $filters, $id)`
**الوصف**: البحث عن فلتر بواسطة `id`.

**المدخلات**:
- `$filters`: مصفوفة الفلاتر.
- `$id`: قيمة `id` المطلوب.

**المخرجات**: مصفوفة الفلتر إذا وُجد، أو `null` إذا لم يوجد.

### 13. `reorder(array $filters)`
**الوصف**: إعادة ترتيب الفلاتر بتعيين `sort_order` تصاعدياً جديداً (1,2,3,...).

**المدخلات**:
- `$filters`: مصفوفة الفلاتر.

**المخرجات**: مصفوفة الفلاتر بعد التعديل.

### 14. `checkDuplicates(array $filters)`
**الوصف**: الكشف عن التكرارات في `id` أو `name`.

**المدخلات**:
- `$filters`: مصفوفة الفلاتر.

**المخرجات**: مصفوفة `['valid' => bool, 'duplicates' => ['id' => [...], 'name' => [...]]]`.

**مثال**:
```php
$result = $manager->checkDuplicates($filters);
if (!$result['valid']) {
    echo "تكرار في id: " . implode(', ', $result['duplicates']['id']);
    echo "تكرار في name: " . implode(', ', $result['duplicates']['name']);
}
```

---

## دوال الخيارات ومصادر البيانات

### 15. `getResolvedOptions(array $filter)`
**الوصف**: الحصول على الخيارات المحلولة للفلتر. إذا كان الفلتر يحتوي على `options` يتم إرجاعها، وإذا كان يحتوي على `data_source` يتم جلب البيانات من المصدر.

**المدخلات**:
- `$filter`: مصفوفة تمثل الفلتر.

**المخرجات**: مصفوفة من الخيارات (كل خيار يحتوي على `id` و `name`).

**مثال**:
```php
$options = $manager->getResolvedOptions($filter);
```

### 16. `enrichWithOptions(array $filter)`
**الوصف**: إثراء الفلتر بإضافة حقل `resolved_options` يحتوي على الخيارات المحلولة.

**المدخلات**:
- `$filter`: مصفوفة الفلتر.

**المخرجات**: نسخة من الفلتر مع الحقل الجديد `resolved_options`.

### 17. `fetchOptionsFromDataSource(array $dataSource)`
**الوصف**: جلب الخيارات من مصدر البيانات (يُستخدم داخلياً). يدعم الأنواع: `api`, `database`, `static`, `function`.

**المدخلات**:
- `$dataSource`: مصفوفة تحتوي على تفاصيل مصدر البيانات.

**المخرجات**: مصفوفة الخيارات.

**ملاحظة**: يجب تنفيذ طلب HTTP للـ API بنفسك (غير مضمن في الكلاس). الكلاس يعيد مصفوفة فارغة حالياً ويترك لك التوسع.

---

## دوال التحويل والتوثيق

### 18. `toJson(array $filters, $pretty = false)`
**الوصف**: تحويل الفلاتر إلى نص JSON.

**المدخلات**:
- `$filters`: مصفوفة الفلاتر.
- `$pretty`: إذا كان `true` يُنسق JSON بطريقة مقروءة.

**المخرجات**: نص JSON.

**مثال**:
```php
$json = $manager->toJson($filters, true);
```

### 19. `getSchema($format = 'array')`
**الوصف**: الحصول على JSON schema الخاص بالفلاتر (يُقرأ من ملف `available-filter-schema.json` إن وجد، وإلا يعيد schema افتراضياً).

**المدخلات**:
- `$format`: `'array'` لإرجاع مصفوفة، `'json'` لإرجاع نص JSON.

**المخرجات**: مصفوفة أو نص JSON يمثل الـ schema.

---

## مثال شامل

لنفترض أن لدينا مصفوفة فلاتر بسيطة ونريد تحميلها وتجهيزها للاستخدام.

```php
use Nano\Tags\Classes\AvailableFilterManager;

// بيانات أولية من مصدر ما (قد تكون من POST أو قاعدة بيانات)
$input = [
    [
        'id' => 'country_id',
        'name' => 'country_id',
        'label' => 'الدولة',
        'type' => 'select',
        'data_source' => [
            'type' => 'api',
            'endpoint' => '/api/countries'
        ]
    ],
    [
        'id' => 'price',
        'name' => 'price',
        'label' => 'السعر',
        'type' => 'number',
        'sort_order' => 5
    ]
];

// إنشاء المدير
$manager = new AvailableFilterManager();

// 1. تحميل وتطبيع الفلاتر
$filters = $manager->fromArray($input);

// 2. التحقق من الصحة
$validation = $manager->validateAll($filters);
if (!$validation['valid']) {
    // معالجة الأخطاء
    exit;
}

// 3. إثراء الفلاتر بالخيارات المحلولة (اختياري)
$enriched = [];
foreach ($filters as $filter) {
    $enriched[] = $manager->enrichWithOptions($filter);
}

// 4. تحويل إلى JSON للإرسال إلى واجهة المستخدم أو API
$jsonOutput = $manager->toJson($enriched, true);

echo $jsonOutput;
```

**مخرجات المثال** (مختصرة):
```json
[
    {
        "id": "country_id",
        "name": "country_id",
        "type": "select",
        "label": {
            "ar": "الدولة",
            "en": "الدولة",
            "fr": "الدولة"
        },
        "sort_order": 0,
        "data_source": {
            "type": "api",
            "endpoint": "/api/countries"
        },
        "resolved_options": []  // ستكون الخيارات الفعلية إذا تم جلبها من API
    },
    {
        "id": "price",
        "name": "price",
        "type": "number",
        "label": {
            "ar": "السعر",
            "en": "السعر",
            "fr": "السعر"
        },
        "sort_order": 5,
        "default": 0,
        "resolved_options": []
    }
]
```

---

## خاتمة

كلاس `AvailableFilterManager` يوفر واجهة موحدة وقوية للتعامل مع بيانات الفلاتر في تطبيقات Laravel. باستخدامه يمكنك:

- تحميل الفلاتر من أي مصدر.
- التأكد من اكتمال بياناتها وتوافقها مع الهيكل المطلوب.
- معالجتها بسهولة (دمج، ترتيب، تصفية).
- الحصول على خياراتها ديناميكياً.
- تحويلها إلى JSON للاستخدام في الـ API أو واجهات المستخدم.

يمكنك توسيع الكلاس حسب الحاجة، خاصة في دوال جلب الخيارات من API أو قاعدة البيانات.