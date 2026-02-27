# توثيق كلاس `FormFieldsManager`

## مقدمة

كلاس `FormFieldsManager` هو كلاس عام (غير مرتبط بموديل معين) مصمم لإدارة وإعداد بيانات حقول النماذج (form fields) بشكل موحد ومستقل. يقوم الكلاس بالمهام التالية:

- تحميل الحقول من مصادر متعددة: مصفوفة (Array)، نص JSON، ملف إعدادات (Config).
- تطبيع بيانات الحقول: إكمال الحقول المفقودة بقيم افتراضية، وتحويل الحقول النصية إلى صيغ متعددة اللغات.
- التحقق من صحة الحقول والتأكد من مطابقتها للـ schema المطلوبة.
- دمج مجموعات حقول متعددة.
- ترتيب الحقول حسب `sort_order`.
- تصفية الحقول النشطة فقط (`is_active == true`).
- البحث عن حقل بواسطة `id`.
- إعادة ترتيب الحقول.
- الكشف عن التكرارات في `id` أو `name`.
- جلب الخيارات (options) للحقول من مصادر البيانات الديناميكية (API، قاعدة بيانات، ثابتة، دوال).
- تحويل الحقول إلى JSON.
- الحصول على JSON schema الخاص بالحقول.

تم بناء هذا الكلاس ليكون مرنًا وقابلاً للتوسع، ويمكن استخدامه في أي جزء من تطبيق OctoberCMS أو Laravel مثل الـ APIs، أوامر الكونسول، أو حتى داخل الموديلات بعد تعديل بسيط.

---

## الخصائص (Properties)

الكلاس يحتوي على الخصائص التالية (يمكن تعديلها عبر الـ constructor):

| الخاصية | النوع | الوصف | القيمة الافتراضية |
|----------|------|------|-------------------|
| `$supportedLanguages` | `array` | اللغات المدعومة للحقول متعددة اللغات. | `['ar', 'en', 'fr']` |
| `$defaultConfigKey` | `string` | المفتاح الافتراضي في ملف الإعدادات لتحميل الحقول. | `'nano.tags::form_fields'` |
| `$defaultRecord` | `array` | الهيكل الافتراضي لسجل الحقل (يُملأ تلقائياً). | (يتم تعيينه في `getDefaultRecordStructure`) |

---

## دوال التحميل (Loading Methods)

### 1. `__construct(array $options = [])`
**الوصف**: مُنشئ الكلاس، يسمح بتخصيص اللغات المدعومة ومفتاح الإعدادات الافتراضي.

**المدخلات**:
- `$options`: مصفوفة اختيارية تحتوي على:
  - `supportedLanguages`: مصفوفة اللغات المطلوبة.
  - `defaultConfigKey`: مفتاح الإعدادات الافتراضي.

**مثال**:
```php
$manager = new FormFieldsManager([
    'supportedLanguages' => ['ar', 'en'],
    'defaultConfigKey'   => 'myplugin::custom_fields'
]);
```

---

### 2. `static load($source, array $options = [])`
**الوصف**: دالة مساعدة لتحميل الحقول من مصدر غير محدد النوع. تقوم بإنشاء كائن مؤقت من الكلاس وتحميل البيانات.

**المدخلات**:
- `$source`: مصدر البيانات، يمكن أن يكون:
  - `array`: مصفوفة PHP.
  - `string`: نص JSON أو مفتاح إعدادات (إذا كان النص يبدأ بـ `{` أو `[` يُعتبر JSON، وإلا يُعتبر مفتاح config).
- `$options`: خيارات لتمريرها إلى المُنشئ (اختياري).

**المخرجات**: مصفوفة الحقول بعد التطبيع.

**مثال**:
```php
// من مصفوفة
$fields = FormFieldsManager::load($array);

// من JSON
$json = '[{"id":"title","label":"العنوان","type":"text"}]';
$fields = FormFieldsManager::load($json);

// من إعدادات
$fields = FormFieldsManager::load('nano.tags::form_fields.shop');
```

---

### 3. `fromArray(array $fields)`
**الوصف**: تحميل الحقول من مصفوفة PHP عادية.

**المدخلات**:
- `$fields`: مصفوفة تحتوي على بيانات الحقول. يمكن أن تكون:
  - مصفوفة من السجلات مثل `[['id'=>'x', ...], ...]`.
  - مصفوفة تحتوي على مفتاح `'form_fields'` وقيمته المصفوفة السابقة.

**المخرجات**: مصفوفة من الحقول بعد تطبيق التطبيع والترتيب.

**مثال**:
```php
$input = [
    ['id' => 'title', 'name' => 'title', 'label' => 'العنوان', 'type' => 'text'],
    ['id' => 'content', 'name' => 'content', 'label' => 'المحتوى', 'type' => 'textarea']
];
$fields = $manager->fromArray($input);
```

---

### 4. `fromJson(string $json)`
**الوصف**: تحميل الحقول من نص JSON.

**المدخلات**:
- `$json`: نص JSON صالح.

**المخرجات**: مصفوفة الحقول بعد التطبيع.

**رمي استثناء**: إذا كان JSON غير صالح.

**مثال**:
```php
$json = '[{"id":"title","label":"العنوان","type":"text"}]';
$fields = $manager->fromJson($json);
```

---

### 5. `fromConfig(string $key = null, string $type = null)`
**الوصف**: تحميل الحقول من ملف إعدادات Laravel (config).

**المدخلات**:
- `$key`: مفتاح الإعدادات (إذا كان `null` يستخدم `$defaultConfigKey`).
- `$type`: جزء إضافي من المفتاح (مثل `'shop'` أو `'products'`) ليصبح المفتاح الكامل `key.type`.

**المخرجات**: مصفوفة الحقول بعد التطبيع.

**مثال**:
```php
// تحميل من config/nano/tags/form_fields.php
$fields = $manager->fromConfig();

// تحميل من config/nano/tags/form_fields.php تحت مفتاح 'shop'
$fields = $manager->fromConfig('nano.tags::form_fields', 'shop');
```

---

## دوال إنشاء السجلات (Record Creation)

### 6. `normalizeFields($fields)`
**الوصف**: دالة داخلية تستخدم لتطبيع الحقول. تقوم بتحويل المدخلات إلى مصفوفة من السجلات الكاملة باستخدام `createRecord`، وتضبط `sort_order` إذا لم يكن موجوداً.

**المدخلات**:
- `$fields`: بيانات الحقول (قد تكون مصفوفة أو أي شيء آخر).

**المخرجات**: مصفوفة مرتبة من السجلات الكاملة.

---

### 7. `createRecord(array $data)`
**الوصف**: إنشاء سجل حقل كامل بالاستناد إلى البيانات المدخلة. يتم دمج البيانات مع الهيكل الافتراضي، ثم تطبيق القيم الإلزامية والخاصة بالنوع، وتنظيف القيم، وأخيراً التحقق من الصحة.

**المدخلات**:
- `$data`: مصفوفة تحتوي على بعض خصائص الحقل (مثل `id`, `name`, `type`, `label`, ...).

**المخرجات**: مصفوفة كاملة تمثل الحقل.

**رمي استثناء**: إذا فشل التحقق (`ApplicationException`).

**مثال**:
```php
$record = $manager->createRecord([
    'id' => 'price',
    'type' => 'number',
    'label' => 'السعر'
]);
/* النتيجة تحتوي على:
    'id' => 'price',
    'name' => 'price', (تم تعيينه تلقائياً)
    'type' => 'number',
    'label' => ['ar' => 'السعر', 'en' => 'السعر', 'fr' => 'السعر'],
    'default' => 0, (خاص بالنوع number)
    ...
*/
```

---

### 8. `applyRequired(array &$record)`
**الوصف**: دالة داخلية لتطبيق القيم الإلزامية ومعالجة الحقول متعددة اللغات.

- تضبط `id` إذا كان فارغاً.
- تضبط `name` من `id` إذا كان فارغاً.
- تحول الحقول (`label`, `placeholder`, `comment`, `emptyOption`) إلى مصفوفة لغات إذا لم تكن كذلك.
- تضبط `sort_order` ليكون رقماً.

---

### 9. `applyTypeSpecific(array &$record)`
**الوصف**: دالة داخلية تضيف قيماً افتراضية خاصة بنوع الحقل (مثل `placeholder` للنصوص، `options` للقوائم، `default` للأرقام، إلخ).

**ملاحظة**: هذه الدالة تغطي جميع أنواع الحقول المدعومة وتضبط الخصائص المناسبة.

---

### 10. `cleanValues(array &$record)`
**الوصف**: دالة داخلية لتنظيف القيم غير الضرورية:
- تحول القيم الفارغة إلى `null` باستثناء بعض الحقول.
- تحول `validation` من نص JSON إلى مصفوفة إذا لزم الأمر.
- تحول `attributes` و `containerAttributes` إلى `null` إذا كانت مصفوفة فارغة.

---

## دوال التحقق (Validation)

### 11. `validateRecord(array $record, $throw = true)`
**الوصف**: التحقق من صحة سجل حقل واحد.

**المدخلات**:
- `$record`: مصفوفة الحقل.
- `$throw`: إذا كان `true` يرمي استثناء عند وجود أخطاء، وإلا يعيد مصفوفة النتائج.

**المخرجات**: مصفوفة `['valid' => bool, 'errors' => array]` (إذا كان `$throw = false`).

**مثال**:
```php
$validation = $manager->validateRecord($record, false);
if (!$validation['valid']) {
    foreach ($validation['errors'] as $error) {
        echo $error;
    }
}
```

---

### 12. `getSupportedFieldTypes()`
**الوصف**: دالة داخلية تعيد قائمة بجميع أنواع الحقول المدعومة (حسب الـ schema).

**المخرجات**: `array` من الأسماء.

---

### 13. `validateAll(array $fields)`
**الوصف**: التحقق من صحة مجموعة كاملة من الحقول.

**المدخلات**:
- `$fields`: مصفوفة الحقول.

**المخرجات**: مصفوفة `['valid' => bool, 'errors' => array]` حيث مفتاح كل خطأ هو فهرس العنصر المخالف.

**مثال**:
```php
$result = $manager->validateAll($fields);
if (!$result['valid']) {
    foreach ($result['errors'] as $index => $errors) {
        echo "خطأ في الحقل $index: " . implode(', ', $errors);
    }
}
```

---

## دوال معالجة الحقول (Utilities)

### 14. `sort(array $fields)`
**الوصف**: ترتيب الحقول تصاعدياً حسب `sort_order` (القيمة الافتراضية `9999` إذا لم توجد).

**المدخلات**:
- `$fields`: مصفوفة الحقول.

**المخرجات**: مصفوفة الحقول مرتبة.

---

### 15. `merge(array $fields1, array $fields2, $override = true)`
**الوصف**: دمج مجموعتي حقول. يتم الدمج بناءً على `id` لكل حقل. إذا وجد حقل بنفس `id` في المجموعتين، يتم استبدال قيمته بقيمة `$fields2` إذا كان `$override = true`، وإلا تُحتفظ بالقيمة الأصلية.

**المدخلات**:
- `$fields1`, `$fields2`: مصفوفتا الحقول.
- `$override`: استبدال الحقول الموجودة.

**المخرجات**: مصفوفة الحقول المدمجة (مرتبة).

**مثال**:
```php
$merged = $manager->merge($defaultFields, $userFields, true);
```

---

### 16. `getActiveOnly(array $fields)`
**الوصف**: إرجاع الحقول النشطة فقط حيث `is_active == true`.

**المدخلات**:
- `$fields`: مصفوفة الحقول.

**المخرجات**: مصفوفة تحتوي على الحقول النشطة.

---

### 17. `findById(array $fields, $id)`
**الوصف**: البحث عن حقل بواسطة `id`.

**المدخلات**:
- `$fields`: مصفوفة الحقول.
- `$id`: قيمة `id` المطلوبة.

**المخرجات**: مصفوفة الحقل إذا وُجد، أو `null` إذا لم يوجد.

---

### 18. `reorder(array $fields)`
**الوصف**: إعادة ترتيب الحقول بتعيين `sort_order` تصاعدياً جديداً (1, 2, 3, ...).

**المدخلات**:
- `$fields`: مصفوفة الحقول.

**المخرجات**: مصفوفة الحقول بعد التعديل (تُعدل المصفوفة الأصلية).

---

### 19. `checkDuplicates(array $fields)`
**الوصف**: الكشف عن التكرارات في `id` أو `name` بين الحقول.

**المدخلات**:
- `$fields`: مصفوفة الحقول.

**المخرجات**: مصفوفة `['valid' => bool, 'duplicates' => ['id' => [...], 'name' => [...]]]`.

**مثال**:
```php
$result = $manager->checkDuplicates($fields);
if (!$result['valid']) {
    echo "تكرار في id: " . implode(', ', $result['duplicates']['id']);
    echo "تكرار في name: " . implode(', ', $result['duplicates']['name']);
}
```

---

## دوال الخيارات ومصادر البيانات (Options & Data Sources)

### 20. `getResolvedOptions(array $field, $isResolvedField = true)`
**الوصف**: الحصول على الخيارات المحلولة للحقل. إذا كان الحقل يحتوي على `options` يتم إرجاعها مباشرة. إذا كان يحتوي على `data_source` يتم جلب البيانات من المصدر باستخدام `fetchOptionsFromDataSource`.

**المدخلات**:
- `$field`: مصفوفة الحقل.
- `$isResolvedField`: إذا كان `true` (افتراضي) يقوم بتحويل البيانات القادمة من المصدر إلى صيغة `['id' => ..., 'name' => ...]`، وإلا يعيد البيانات كما هي (خام).

**المخرجات**: مصفوفة من الخيارات (قد تكون فارغة).

**مثال**:
```php
$options = $manager->getResolvedOptions($field);
```

---

### 21. `enrichWithOptions(array $field, $isResolvedField = true)`
**الوصف**: إثراء الحقل بإضافة حقل جديد `resolved_options` يحتوي على الخيارات المحلولة.

**المدخلات**:
- `$field`: مصفوفة الحقل.
- `$isResolvedField`: نفس المعنى السابق.

**المخرجات**: نسخة من الحقل مع الحقل الإضافي.

**مثال**:
```php
$enrichedField = $manager->enrichWithOptions($field);
```

---

### 22. `fetchOptionsFromDataSource(array $dataSource, $isResolvedField = false)`
**الوصف**: دالة داخلية لجلب الخيارات من مصدر البيانات. تدعم الأنواع التالية:

- **`api`**: يقوم بطلب HTTP GET إلى `endpoint` مع `params`، ويدعم تحويل الرابط النسبي إلى كامل باستخدام `url()`. يحاول استخدام `Illuminate\Support\Facades\Http` إن وجد، أو `October\Rain\Network\Http` كبديل.
- **`database`**: يستعلم من نموذج Eloquent مع إمكانية إضافة `conditions` و `order_by`.
- **`static`**: يعيد `options` مباشرة.
- **`function`**: يستدعي دالة (أو دالة داخل كلاس) مع `params`.

**المدخلات**:
- `$dataSource`: مصفوفة تحتوي على `type` والبيانات المطلوبة حسب النوع.
- `$isResolvedField`: إذا كان `true` يحول بيانات API إلى صيغة `['id', 'name']` باستخدام `value_field` و `label_field`.

**المخرجات**: مصفوفة الخيارات.

---

## دوال التحويل والتوثيق (Conversion & Schema)

### 23. `toJson(array $fields, $pretty = false)`
**الوصف**: تحويل مصفوفة الحقول إلى نص JSON.

**المدخلات**:
- `$fields`: مصفوفة الحقول.
- `$pretty`: إذا كان `true` يُنسق JSON بشكل مقروء.

**المخرجات**: نص JSON.

**مثال**:
```php
$json = $manager->toJson($fields, true);
echo $json;
```

---

### 24. `getSchema($format = 'array')`
**الوصف**: الحصول على JSON schema الخاص بالحقول. يحاول قراءة الملف `form-fields-settings-schema-no-default.json` من مسار `plugins/nano/tags/models/type/`. إذا لم يوجد، يعيد schema افتراضياً مبسطاً.

**المدخلات**:
- `$format`: `'array'` لإرجاع مصفوفة، `'json'` لإرجاع نص JSON.

**المخرجات`: مصفوفة أو نص JSON.

**مثال**:
```php
$schemaArray = $manager->getSchema('array');
$schemaJson = $manager->getSchema('json');
```

---

### 25. `deepMerge(array $array1, array $array2)`
**الوصف**: دالة داخلية لدمج عميق لمصفوفتين (تستخدم في الدمج الداخلي).

---

### 26. `getDefaultRecordStructure()`
**الوصف**: دالة داخلية تُعيد الهيكل الافتراضي الكامل لسجل الحقل، والذي يحتوي على جميع الخصائص الممكنة مع قيمها الافتراضية (كما هو موصوف في الـ schema).

---

## مثال شامل

لنفترض أننا نريد بناء واجهة برمجة تطبيقات (API) تقبل بيانات حقول النموذج من المستخدم، وتقوم بتطبيعها والتحقق منها، ثم إرجاعها مع الخيارات المحلولة.

```php
use Nano\Tags\Classes\FormFieldsManager;

// البيانات المرسلة من العميل (قد تكون JSON أو مصفوفة)
$input = [
    [
        'id' => 'title',
        'label' => 'العنوان',
        'type' => 'text',
        'required' => true,
        'sort_order' => 1
    ],
    [
        'id' => 'country',
        'label' => 'الدولة',
        'type' => 'select',
        'data_source' => [
            'type' => 'api',
            'endpoint' => '/api/countries',
            'value_field' => 'code',
            'label_field' => 'name_ar'
        ],
        'sort_order' => 2
    ]
];

// 1. إنشاء المدير
$manager = new FormFieldsManager();

// 2. تحميل وتطبيع الحقول
$fields = $manager->fromArray($input);

// 3. التحقق من الصحة
$validation = $manager->validateAll($fields);
if (!$validation['valid']) {
    return response()->json([
        'error' => 'Invalid fields data',
        'details' => $validation['errors']
    ], 422);
}

// 4. إثراء الحقول بالخيارات (لإرسالها للواجهة)
$enriched = [];
foreach ($fields as $field) {
    $enriched[] = $manager->enrichWithOptions($field);
}

// 5. تحويل إلى JSON
$output = $manager->toJson($enriched, true);

// 6. إرجاع الرد
return response($output, 200)->header('Content-Type', 'application/json');
```

**النتيجة المتوقعة (JSON)**:
```json
[
    {
        "id": "title",
        "name": "title",
        "type": "text",
        "label": {
            "ar": "العنوان",
            "en": "العنوان",
            "fr": "العنوان"
        },
        "required": true,
        "sort_order": 1,
        "placeholder": {
            "ar": "أدخل العنوان",
            "en": "Enter العنوان",
            "fr": "Entrez العنوان"
        },
        "resolved_options": []
    },
    {
        "id": "country",
        "name": "country",
        "type": "select",
        "label": {
            "ar": "الدولة",
            "en": "الدولة",
            "fr": "الدولة"
        },
        "data_source": {
            "type": "api",
            "endpoint": "/api/countries",
            "value_field": "code",
            "label_field": "name_ar"
        },
        "sort_order": 2,
        "emptyOption": {
            "ar": "اختر",
            "en": "Select",
            "fr": "Sélectionner"
        },
        "showSearch": true,
        "resolved_options": [
            { "id": "SA", "name": "السعودية" },
            { "id": "EG", "name": "مصر" },
            ...
        ]
    }
]
```

---

## خاتمة

كلاس `FormFieldsManager` يوفر واجهة برمجية متكاملة وقوية للتعامل مع بيانات حقول النماذج في تطبيقات OctoberCMS و Laravel. باستخدامه يمكنك:

- توحيد طريقة تحميل الحقول من مصادر متعددة.
- التأكد من اكتمال بياناتها وصلاحيتها.
- معالجتها بسهولة (دمج، ترتيب، تصفية).
- الحصول على خياراتها ديناميكياً من APIs أو قاعدة البيانات.
- تحويلها إلى JSON للاستخدام في الـ API أو التخزين.

