# توثيق كلاس RepeaterFieldsData

## مقدمة

كلاس `RepeaterFieldsData` هو جزء من إضافة `Tss\Basic` في OctoberCMS، ويهدف إلى تسهيل التعامل مع الحقول القابلة للتكرار (repeater fields) من نوع `jsonable`. يدعم الكلاس خمسة حقول رئيسية هي: `phone`, `email`, `website`, `properties`, `links`، والتي تُستخدم بكثرة في نماذج البيانات لتخزين معلومات متعددة مثل قائمة أرقام الهواتف، عناوين البريد الإلكتروني، روابط المواقع، إلخ.

نظراً لأن هذه الحقول قد تُمرر إلى API بصيغ مختلفة (نص مفرد، مصفوفة نصوص، كائن واحد، مصفوفة كائنات، خليط) فإن الكلاس يقوم بتطبيع (normalize) هذه المدخلات إلى بنية موحدة قابلة للحفظ في قاعدة البيانات عبر خاصية `jsonable`، كما يوفر دوال لتنسيق الإخراج والتحقق والدمج والاختبار.

---

## الفوائد والمزايا

- **توحيد البنية** – يحول جميع صيغ الإدخال المتنوعة إلى مصفوفة متسقة العناصر، مما يضمن سلامة البيانات المخزنة.
- **مرونة عالية** – يقبل النصوص المفردة، المصفوفات النصية، الكائنات المنفردة، ومصفوفات الكائنات، بل وحتى الخلط بينها.
- **الاعتماد على التعريفات (YAML)** – يقرأ الكلاس ملفات تعريف الحقول (`repeater_fields_*.yaml`) لاستخراج القيم الافتراضية وأنواع الحقول، مما يجعله قابلاً للتعديل دون تغيير الكود.
- **معالجة أنواع الحقول المختلفة** – يتعامل مع `switch`، `dropdown`، `mediafinder` بطريقة ذكية (تحويل القيم المنطقية إلى `'1'`/`'0'`، وتحويل مسارات الوسائط إلى روابط كاملة).
- **دوال متكاملة** – يوفر دوال للإدخال (`processInput`)، الإخراج (`formatOutput`)، استخراج القيم الرئيسية (`extractMainValues`)، الدمج (`mergeData`)، التحقق (`validate`)، والاختبار (`runTests`).
- **جاهز للاستخدام في API** – يضمن تنسيق البيانات بالشكل المناسب للردود، مع إمكانية تصفية العناصر غير الظاهرة.
- **تحسين الأداء** – يخبئ تعريفات الحقول بعد تحميلها من YAML لمنع القراءة المتكررة من القرص.

---

## الاستخدامات

- في الـ API عند استقبال بيانات من العميل لحقول قابلة للتكرار.
- عند حفظ النماذج (models) التي تحتوي على خصائص `jsonable` من نوع repeater.
- عند عرض البيانات في واجهات API أو في القوالب بعد تنسيقها.
- عند تحديث جزئي للبيانات (مثل دمج قوائم جديدة مع القديمة).
- للتحقق من صحة البيانات قبل الحفظ.
- لاختبار مدى مرونة المعالجة عبر دوال الاختبار المدمجة.

---

## وظائف الكلاس وخصائصه

### 1. `processInput(string $fieldName, mixed $input): array`
- **الوصف**: تحويل المدخلات المختلفة إلى مصفوفة موحدة قابلة للحفظ.
- **المعاملات**:
  - `$fieldName`: اسم الحقل (phone, email, website, properties, links).
  - `$input`: البيانات المدخلة (نص، مصفوفة، كائن، null).
- **الإرجاع**: مصفوفة تحتوي على عناصر طبيعية مع القيم الافتراضية وإضافة `_group`.

### 2. `formatOutput(string $fieldName, mixed $storedValue, bool $filterHidden = false, bool $resolveMedia = true): array`
- **الوصف**: تنسيق البيانات المخزنة للإخراج (عرض API).
- **المعاملات**:
  - `$fieldName`: اسم الحقل.
  - `$storedValue`: القيمة المخزنة (عادة ما تكون مصفوفة).
  - `$filterHidden`: إذا كان `true`، يتم استبعاد العناصر التي `is_show = '0'`.
  - `$resolveMedia`: إذا كان `true`، يتم تحويل مسارات `website_icon` إلى روابط كاملة باستخدام `MediaLibrary`.
- **الإرجاع**: مصفوفة مهيأة للإخراج.

### 3. `extractMainValues(string $fieldName, array $data, bool $includeDefaults = false): array`
- **الوصف**: استخراج القيم الرئيسية فقط من البيانات (مثل أرقام الهواتف أو عناوين البريد).
- **المعاملات**:
  - `$fieldName`: اسم الحقل.
  - `$data`: مصفوفة البيانات (بعد `processInput` أو من المخزن).
  - `$includeDefaults`: إذا كان `true`، يتم تضمين العناصر الافتراضية فقط (`is_default = '1'`).
- **الإرجاع**: مصفوفة من القيم (نصوص).

### 4. `mergeData(string $fieldName, array $oldData, array $newData, bool $replace = false): array`
- **الوصف**: دمج بيانات جديدة مع القديمة (لتحديث جزئي).
- **المعاملات**:
  - `$fieldName`: اسم الحقل.
  - `$oldData`: البيانات القديمة (مصفوفة).
  - `$newData`: البيانات الجديدة (يمكن أن تكون بأي صيغة).
  - `$replace`: إذا كان `true`، يتم استبدال القديم بالكامل بالجديد.
- **الإرجاع**: مصفوفة البيانات بعد الدمج.

### 5. `validate(string $fieldName, array $data): array`
- **الوصف**: التحقق من صحة البيانات وفقًا لتعريفات الحقول (الحقول المطلوبة، قيم dropdown المسموحة).
- **المعاملات**:
  - `$fieldName`: اسم الحقل.
  - `$data`: البيانات المراد التحقق منها (مصفوفة).
- **الإرجاع**: مصفوفة تحتوي على `valid` (boolean) و `errors` (مصفوفة أخطاء).

### 6. `runTests(): array`
- **الوصف**: تشغيل مجموعة اختبارات على جميع الحقول المدعومة لضمان عمل الكلاس بشكل صحيح.
- **الإرجاع**: تقرير مفصل يحتوي على ملخص ونتائج كل حالة اختبار.

### 7. `getSupportedFields(): array`
- **الوصف**: إرجاع قائمة بأسماء الحقول المدعومة.
- **الإرجاع**: مصفوفة من السلاسل النصية.

---

## أمثلة توضيحية لكل حقل

في جميع الأمثلة نفترض أن الكلاس مستورد كالتالي:
```php
use Tss\Basic\Classes\RepeaterFieldsData;
$repeater = new RepeaterFieldsData();
```

### حقل phone

#### مثال 1: نص مفرد
**الإدخال:**
```php
$input = "770529482";
$result = $repeater->processInput('phone', $input);
```
**المخرجات (بعد processInput):**
```php
[
    [
        'phone_label'  => '',
        'phone_number' => '770529482',
        'phone_type'   => null,
        'sort_show'    => null,
        'is_default'   => '1', // القيمة الافتراضية من YAML
        'is_show'      => '1',
        'phone_note'   => '',
        '_group'       => 'phone',
    ]
]
```

#### مثال 2: مصفوفة نصوص
**الإدخال:**
```php
$input = ["770529482", "780505400"];
$result = $repeater->processInput('phone', $input);
```
**المخرجات:**
```php
[
    ['phone_number' => '770529482', '_group' => 'phone', ...],
    ['phone_number' => '780505400', '_group' => 'phone', ...]
]
```

#### مثال 3: كائن واحد كامل
**الإدخال:**
```php
$input = [
    'phone_label'  => 'وتس',
    'phone_number' => '+967780505400',
    'phone_type'   => 'mobile',
    'sort_show'    => '1',
    'is_default'   => '1',
    'is_show'      => '1',
    'phone_note'   => 'ملاحظة',
];
$result = $repeater->processInput('phone', $input);
```
**المخرجات:**
```php
[
    [
        'phone_label'  => 'وتس',
        'phone_number' => '+967780505400',
        'phone_type'   => 'mobile',
        'sort_show'    => '1',
        'is_default'   => '1',
        'is_show'      => '1',
        'phone_note'   => 'ملاحظة',
        '_group'       => 'phone',
    ]
]
```

#### مثال 4: مصفوفة كائنات
**الإدخال:**
```php
$input = [
    ['phone_number' => '111111111', 'is_default' => '0'],
    ['phone_number' => '222222222', 'is_default' => '1']
];
$result = $repeater->processInput('phone', $input);
```
**المخرجات:**
```php
[
    ['phone_number' => '111111111', 'is_default' => '0', '_group' => 'phone', ...],
    ['phone_number' => '222222222', 'is_default' => '1', '_group' => 'phone', ...]
]
```

#### مثال 5: خليط (نصوص + كائنات)
**الإدخال:**
```php
$input = [
    "333333333",
    ['phone_number' => '444444444', 'is_default' => '0'],
    "555555555"
];
$result = $repeater->processInput('phone', $input);
```
**المخرجات:** ثلاثة عناصر (العناصر النصية تتحول إلى كائنات بالحقل الرئيسي phone_number).

---

### حقل email

#### مثال: نص مفرد
**الإدخال:** `"test@example.com"`
**المخرجات:** عنصر واحد مع `email_text = test@example.com` وباقي القيم الافتراضية.

#### مثال: مصفوفة كائنات جزئية
**الإدخال:**
```php
[
    ['email_text' => 'one@site.com'],
    ['email_text' => 'two@site.com', 'is_default' => '0']
]
```
**المخرجات:** عنصرين مع دمج القيم الافتراضية.

---

### حقل website

#### مثال: مع أيقونة (mediafinder)
**الإدخال:**
```php
[
    'website_label' => 'فيسبوك',
    'website_url'   => 'https://fb.com/page',
    'website_icon'  => 'icons/fb.png'
]
```
**المخرجات (بعد processInput):** يتم الاحتفاظ بالمسار النسبي.
**عند formatOutput مع resolveMedia = true:**
```php
// يتم تحويل icons/fb.png إلى الرابط الكامل لوسائط الموقع
'website_icon' => 'https://example.com/storage/app/media/icons/fb.png'
```

---

### حقل properties

#### مثال: نص مفرد (يعامل كقيمة للحقل الرئيسي value)
**الإدخال:** `"قيمة تجريبية"`
**المخرجات:**
```php
[
    [
        'title'      => null,   // القيم الافتراضية من YAML
        'code'       => null,
        'value'      => 'قيمة تجريبية',
        'is_default' => '1',
        'is_show'    => '1',
        'sort_show'  => null,
        '_group'     => 'properties',
    ]
]
```

#### مثال: كائن كامل
**الإدخال:**
```php
[
    'title' => 'اللون',
    'code'  => 'color',
    'value' => 'أحمر'
]
```
**المخرجات:** عنصر واحد مع القيم المعطاة.

---

### حقل links

#### مثال: نص مفرد (url)
**الإدخال:** `"https://example.com"`
**المخرجات:** عنصر مع `url = https://example.com` وباقي القيم الافتراضية (مثل `target = _blank`).

#### مثال: مصفوفة كائنات مع target مختلف
**الإدخال:**
```php
[
    ['title' => 'رابط1', 'url' => 'https://link1.com'],
    ['title' => 'رابط2', 'url' => 'https://link2.com', 'target' => '_self']
]
```
**المخرجات:** عنصرين، الأول target افتراضي `_blank` والثاني `_self`.

---

## أمثلة متكاملة متقدمة

### مثال 1: استخدام الكلاس في Controller لاستقبال بيانات من API وحفظها في موديل

لنفترض أن لدينا موديل `Department` به خاصية `jsonable = ['phone', 'email', 'website']`.

```php
use Tss\Basic\Classes\RepeaterFieldsData;
use Tss\Basic\Models\Department;

class DepartmentController
{
    public function store()
    {
        $data = input(); // بيانات الطلب

        $repeater = new RepeaterFieldsData();

        // تجهيز الحقول القابلة للتكرار
        $department = new Department();
        $department->name = $data['name'];
        $department->phone = $repeater->processInput('phone', $data['phone'] ?? null);
        $department->email = $repeater->processInput('email', $data['email'] ?? null);
        $department->website = $repeater->processInput('website', $data['website'] ?? null);

        $department->save();

        return response()->json([
            'success' => true,
            'data'    => $department
        ]);
    }
}
```

### مثال 2: تحديث جزئي (دمج بيانات جديدة مع القديمة)

```php
public function update($id)
{
    $department = Department::find($id);
    $data = input();

    $repeater = new RepeaterFieldsData();

    if (isset($data['phone'])) {
        // دمج الهواتف الجديدة مع القديمة دون استبدال كامل
        $oldPhones = $department->phone ?? [];
        $department->phone = $repeater->mergeData('phone', $oldPhones, $data['phone']);
    }

    if (isset($data['email'])) {
        // استبدال كامل للبريد الإلكتروني
        $department->email = $repeater->processInput('email', $data['email']);
    }

    $department->save();

    return response()->json($department);
}
```

### مثال 3: تنسيق الإخراج مع تصفية العناصر غير الظاهرة وحل مسارات الوسائط

```php
public function show($id)
{
    $department = Department::with('someRelation')->find($id);

    $repeater = new RepeaterFieldsData();

    $response = [
        'id'   => $department->id,
        'name' => $department->name,
        'phone' => $repeater->formatOutput('phone', $department->phone, true), // إخفاء غير الظاهر
        'email' => $repeater->formatOutput('email', $department->email, false), // عرض الكل
        'website' => $repeater->formatOutput('website', $department->website, true, true), // إخفاء + حل الوسائط
    ];

    return response()->json($response);
}
```

### مثال 4: استخراج أرقام الهواتف الافتراضية فقط

```php
$phones = $department->phone; // البيانات المخزنة
$defaultPhoneNumbers = $repeater->extractMainValues('phone', $phones, true);
// يعيد مصفوفة بأرقام الهواتف التي is_default = 1
```

### مثال 5: التحقق من صحة البيانات قبل الحفظ

```php
$data = input('website'); // قد تكون بيانات غير صحيحة
$processed = $repeater->processInput('website', $data);

$validation = $repeater->validate('website', $processed);
if (!$validation['valid']) {
    return response()->json(['errors' => $validation['errors']], 422);
}

// إذا صحيح، نحفظ
$department->website = $processed;
```

### مثال 6: تشغيل الاختبارات للتأكد من سلامة الكلاس

يمكن للمطور تشغيل الاختبارات يدويًا (مثلاً في سكريبت تجريبي أو في وحدة تحكم خاصة):

```php
Route::get('test-repeater', function() {
    $repeater = new RepeaterFieldsData();
    $report = $repeater->runTests();
    return response()->json($report);
});
```

هذا سيعرض تقريرًا بجميع حالات الاختبار ونتائجها، مما يساعد في التحقق من أن الكلاس يعمل كما هو متوقع بعد أي تعديل.

---

## خاتمة

كلاس `RepeaterFieldsData` يوفر حلاً موحداً ومرناً للتعامل مع الحقول القابلة للتكرار في OctoberCMS، مما يقلل من التعقيد عند بناء واجهات API أو نماذج تحتوي على بيانات متعددة. بفضل تصميمه المعتمد على التعريفات (YAML) ودواله المتعددة، يمكن للمطورين الاعتماد عليه بشكل كامل في مشاريعهم دون الحاجة لإعادة اختراع العجلة.

**ملاحظة:** تأكد من وجود ملفات تعريف YAML في المسارات المحددة (`$/tss/basic/classes/partials/`) قبل استخدام الكلاس، وقم بتعديل المسارات إذا لزم الأمر.