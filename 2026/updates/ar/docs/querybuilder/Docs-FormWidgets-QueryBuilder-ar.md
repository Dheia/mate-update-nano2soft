# توثيق `Nano2\QueryBuilder\FormWidgets\QueryBuilder`

**وحدة نانوسوفت البرمجية – أداة بناء الاستعلامات البصرية**

---

## 1. نظرة عامة

`QueryBuilder` هو عنصر واجهة (FormWidget) متقدم يسمح بإنشاء شروط استعلام معقدة بطريقة بصرية وسهلة الاستخدام، دون الحاجة إلى كتابة SQL أو منطق برمجي يدوي. يعتمد هذا العنصر على مكتبة `jQuery QueryBuilder` من ناحية الواجهة الأمامية، ويُخزّن القواعد بصيغة JSON داخل قاعدة البيانات، مع إمكانية تطبيقها مباشرة على استعلامات البيانات (Models) عبر كلاس `QueryBuilderParserHelper`.

يُستخدم هذا العنصر في:
- بناء تقارير ديناميكية بشروط متعددة المستويات (AND / OR).
- إنشاء شاشات بحث متقدمة.
- تخزين فلاتر قابلة لإعادة الاستخدام من قبل المستخدمين.
- دعم أنواع متعددة من المُدخلات (نص، أرقام، تواريخ، قوائم منسدلة، خانات اختيار، إلخ).

---

## 2. التثبيت والإعداد الأساسي

### 2.1. تضمين الوحدة
تأكد من تثبيت وحدة `Nano2.QueryBuilder` في مشروع نانوسوفت الخاص بك.

### 2.2. إضافة الحقل في ملف YAML الخاص بالنموذج
```yaml
# models/my_model/fields.yaml
fields:
    report_conditions:
        label: شروط التقرير
        type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
        span: full
```

### 2.3. العمود في قاعدة البيانات
يجب أن يكون العمود المرتبط من النوع `text` أو `longtext` لتخزين JSON الناتج:
```php
// في ملف تعريف الموديل
public $jsonable = ['report_conditions'];
```

---

## 3. الخصائص والتكوين (Configurable Properties)

يمكنك تخصيص سلوك العنصر عبر كتابة الخصائص في ملف YAML أو تعيينها مباشرة في الكلاس.

| الخاصية | النوع | الافتراضي | الوصف |
|----------|------|-----------|-------|
| `filters` | array / string | `[]` | تعريف الفلاتر المتاحة (يُفصّل لاحقاً). يمكن أن تكون مصفوفة، أو اسم دالة في الموديل. |
| `rules` | array | `[]` | القواعد الافتراضية التي تظهر عند تحميل الواجهة (بصيغة JSON أو مصفوفة). |
| `sort_filters` | bool | `false` | ترتيب الفلاتر أبجدياً حسب الترجمة. |
| `allow_groups` | int / bool | `-1` | الحد الأقصى لعدد المجموعات المتداخلة (`-1` = غير محدود، `0` = لا مجموعات، `true` = -1، `false` = 0). |
| `allow_empty` | bool | `false` | السماح بوجود قواعد فارغة (بدون أي شرط). |
| `display_errors` | bool | `true` | إظهار رسائل الخطأ بجانب القواعد (مثل الحقول المطلوبة أو التنسيق الخاطئ). |
| `conditions` | array / string | `['AND', 'OR']` | قائمة بالروابط المنطقية المسموحة. يمكن إرسال سلسلة واحدة `'AND'` أو `'OR'`. |
| `default_condition` | string | `'AND'` | الرابط الافتراضي للمجموعة الجذرية. |
| `inputs_separator` | string | `' , '` | الفاصل بين حقول الإدخال المتعددة (مثل BETWEEN). |
| `display_empty_filter` | bool | `true` | عرض خيار فارغ (`------`) في قائمة الفلاتر. |
| `select_placeholder` | string | `'------'` | النص المعروض عند عدم اختيار فلتر. |
| `default_filter` | string | `null` | معرف الفلتر الذي يتم اختياره تلقائياً عند إضافة قاعدة جديدة. |
| `plugins` | array | `[]` | قائمة الإضافات المفعَّلة (راجع قسم الإضافات). |
| `isShowParseSqlBtn` | bool | `false` | إظهار زر لترجمة القواعد إلى SQL (لأغراض التصحيح). |
| `isShowResetBtn` | bool | `true` | إظهار زر إعادة ضبط القواعد. |
| `isShowSetBtn` | bool | `true` | إظهار زر تحميل القواعد الافتراضية (rules). |
| `isShowGetBtn` | bool | `true` | إظهار زر تطبيق القواعد وحفظها في الحقل المخفي. |
| `isAllowNotfiy` | bool | `true` | عرض إشعار عند تطبيق القواعد بنجاح أو فشل. |

**مثال تكوين متقدم في YAML**:
```yaml
report_conditions:
    type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
    label: شروط البحث
    sort_filters: true
    allow_groups: 3
    allow_empty: false
    conditions: ['AND', 'OR']
    default_condition: 'AND'
    inputs_separator: ' | '
    display_empty_filter: false
    select_placeholder: 'اختر الحقل'
    default_filter: 'name'
    plugins:
        sortable: null
        filter-description: { mode: 'popover' }
    isShowParseSqlBtn: true
```

---

## 4. تعريف الفلاتر (Filters)

الفلاتر هي اللبنات الأساسية التي تسمح للمستخدم ببناء الشروط. كل فلتر يحدد حقل بيانات، نوعه، طريقة إدخال القيمة، والمشغلات المسموحة.

### 4.1. خصائص الفلتر

| الخاصية | النوع | إلزامي | الوصف |
|----------|------|--------|-------|
| `id` | string | نعم | معرف فريد للفلتر (يُستخدم في القواعد). |
| `label` | string / object | نعم | النص المعروض. يدعم الترجمة كمصفوفة `{'en':'Name', 'ar':'الاسم'}`. |
| `type` | string | نعم | نوع البيانات: `string`, `integer`, `double`, `date`, `time`, `datetime`, `boolean`. |
| `input` | string | لا | نوع عنصر الإدخال: `text`, `number`, `textarea`, `radio`, `checkbox`, `select`. الافتراضي حسب النوع. |
| `field` | string | لا | اسم الحقل الفعلي في قاعدة البيانات (إذا كان مختلفاً عن `id`). |
| `operators` | array | لا | قائمة المشغلات المسموحة لهذا الفلتر (راجع الجدول أدناه). إذا لم تُحدد، ستُستخدم جميع المشغلات المناسبة للنوع. |
| `default_operator` | string | لا | المشغل الافتراضي (يجب أن يكون ضمن `operators`). |
| `values` | array / string | مُشروط | مطلوب لـ `radio`, `checkbox`, `select`. مصفوفة من الخيارات أو اسم دالة لجلبها ديناميكياً. |
| `validation` | object | لا | قواعد التحقق: `min`, `max`, `step`, `format`, `allow_empty_value`, `messages`. |
| `placeholder` | string | لا | نص توجيهي داخل الحقل (لـ `text`, `textarea`, `number`). |
| `size` | int | لا | عرض الحقل (لـ `text`, `textarea`). |
| `rows` | int | لا | ارتفاع الحقل (لـ `textarea`). |
| `multiple` | bool | لا | السماح بتحديد عدة قيم (لـ `select`). |
| `vertical` | bool | لا | عرض الخيارات عمودياً (لـ `radio`, `checkbox`). |
| `plugin` | string | لا | اسم إضافة jQuery (مثل `datepicker`, `selectize`). |
| `plugin_config` | object | لا | إعدادات الإضافة. |
| `default_value` | mixed | لا | قيمة افتراضية تُعبأ تلقائياً عند إنشاء قاعدة جديدة. |
| `value_separator` | string | لا | الفاصل المستخدم لتحويل القيم المتعددة إلى سلسلة (للمشغلات `in`, `not_in`). |
| `icon` | string | لا | أيقونة (فئة Bootstrap Icons) تظهر بجانب الفلتر في القائمة. |
| `optgroup` | string | لا | اسم المجموعة التي ينتمي إليها الفلتر (لتنظيم القائمة المنسدلة). |
| `description` | string / function | لا | نص وصفي يظهر حسب الإضافة `filter-description`. |
| `data` | object | لا | بيانات مخصصة إضافية تُضاف إلى كائن القاعدة عند التصدير. |

### 4.2. المشغلات (Operators) المدعومة

| المشغل | الوصف | ينطبق على الأنواع |
|--------|-------|-------------------|
| `equal` | يساوي | string, number, datetime, boolean |
| `not_equal` | لا يساوي | string, number, datetime, boolean |
| `in` | ضمن مجموعة | string, number, datetime |
| `not_in` | ليس ضمن مجموعة | string, number, datetime |
| `less` | أقل من | number, datetime |
| `less_or_equal` | أقل أو يساوي | number, datetime |
| `greater` | أكبر من | number, datetime |
| `greater_or_equal` | أكبر أو يساوي | number, datetime |
| `between` | بين (قيمتين) | number, datetime |
| `not_between` | ليس بين | number, datetime |
| `begins_with` | يبدأ بـ | string |
| `not_begins_with` | لا يبدأ بـ | string |
| `contains` | يحتوي على | string |
| `not_contains` | لا يحتوي على | string |
| `ends_with` | ينتهي بـ | string |
| `not_ends_with` | لا ينتهي بـ | string |
| `is_empty` | فارغ (يساوي `''`) | string |
| `is_not_empty` | غير فارغ | string |
| `is_null` | يساوي NULL | جميع الأنواع |
| `is_not_null` | لا يساوي NULL | جميع الأنواع |

### 4.3. أمثلة لتعريف الفلاتر

#### مثال 1: فلتر نص بسيط
```yaml
name:
    id: name
    label: الاسم
    type: string
    operators: ['equal', 'contains', 'begins_with']
    placeholder: أدخل الاسم
```

#### مثال 2: فلتر رقم مع تحقق
```yaml
price:
    id: price
    label: السعر
    type: double
    validation:
        min: 0
        step: 0.01
    default_value: 0
```

#### مثال 3: فلتر قائمة منسدلة
```yaml
category:
    id: category_id
    label: التصنيف
    type: integer
    input: select
    values:
        1: كتب
        2: أفلام
        3: موسيقى
    operators: ['equal', 'not_equal', 'in']
    default_operator: 'in'
```

#### مثال 4: فلتر تاريخ مع إضافة Datepicker
```yaml
created_at:
    id: created_at
    label: تاريخ الإنشاء
    type: date
    input: text
    plugin: datepicker
    plugin_config:
        format: 'yyyy-mm-dd'
    validation:
        format: 'yyyy-mm-dd'
```

#### مثال 5: فلتر خيارات متعددة (Checkbox)
```yaml
status:
    id: status
    label: الحالة
    type: integer
    input: checkbox
    values:
        1: نشط
        0: غير نشط
    vertical: true
    operators: ['equal']
```

#### مثال 6: فلتر يعتمد على دالة في الموديل لجلب الخيارات
```yaml
user:
    id: user_id
    label: المستخدم
    type: integer
    input: select
    values: getUsersOptions   # اسم دالة في الموديل
```

---

## 5. تعريف القواعد الافتراضية (Rules)

يمكنك تعيين قواعد مبدئية تظهر عند تحميل الواجهة عبر خاصية `rules`. الصيغة كالتالي:

```yaml
rules:
    condition: AND
    rules:
        - id: price
          operator: greater
          value: 100
        - condition: OR
          rules:
              - id: category
                operator: equal
                value: 1
              - id: category
                operator: equal
                value: 2
```

**ملاحظة**: يمكن أيضاً تمرير القواعد كمصفوفة PHP في ملف YAML (سيتم تحويلها إلى JSON تلقائياً).

---

## 6. الإضافات (Plugins)

الإضافات تمدد وظائف الواجهة وتضيف ميزات تفاعلية. تُفعَّل عبر خاصية `plugins` كمصفوفة.

### 6.1. قائمة الإضافات المدعومة

| اسم الإضافة | الوصف | الخيارات |
|-------------|-------|-----------|
| `sortable` | تمكين السحب والإفلات لإعادة ترتيب القواعد والمجموعات. | `icon` (مثل `bi-sort-down`), `inherit_no_sortable`, `inherit_no_drop` |
| `filter-description` | عرض وصف توضيحي للفلتر. | `mode` (`inline`, `popover`, `bootbox`), `icon` |
| `bt-tooltip-errors` | عرض أخطاء التحقق كـ Tooltip من Bootstrap. | `placement` |
| `bt-checkbox` | تنسيق عناصر `checkbox` و `radio` باستخدام Bootstrap. | `color` |
| `unique-filter` | منع استخدام نفس الفلتر أكثر من مرة (عالمياً أو داخل المجموعة). | – |
| `not-group` | إضافة زر "NOT" لعكس منطق المجموعة. | `icon_checked`, `icon_unchecked` |
| `invert` | زر قلب الشرط (تحويل `=` إلى `!=`، و `AND` إلى `OR`). | `recursive`, `invert_rules`, `display_rules_button`, `icon` |

### 6.2. مثال لتفعيل الإضافات
```yaml
plugins:
    sortable: { icon: 'bi-arrows-move' }
    filter-description: { mode: 'popover', icon: 'bi-question-circle' }
    bt-tooltip-errors: { placement: 'top' }
    unique-filter: null
    not-group: { icon_unchecked: 'bi-square', icon_checked: 'bi-check-square' }
```

---

## 7. جلب الخيارات الديناميكية من الموديل

لتزويد الفلاتر بخيارات ديناميكية (مثل قائمة المستخدمين، الأقسام، المنتجات)، يمكنك تعريف دالة في الموديل الخاص بك. هناك طريقتان:

### 7.1. دالة عامة لجميع حقول QueryBuilder
```php
public function getQueryBuilderFiltersOptions($columnName)
{
    if ($columnName == 'filters') {
        return [
            [
                'id' => 'user_id',
                'label' => 'المستخدم',
                'type' => 'integer',
                'input' => 'select',
                'values' => \App\Models\User::pluck('name', 'id')->toArray()
            ],
            // ...
        ];
    }
    return [];
}
```

### 7.2. دالة خاصة باسم الحقل
إذا كان اسم حقل QueryBuilder هو `report_conditions`، فاسم الدالة سيكون:
```php
public function getReportConditionsQueryBuilderFiltersOptions($fieldName, $columnName)
{
    // $fieldName = 'report_conditions', $columnName = 'filters' أو 'rules'
    if ($columnName == 'filters') {
        return [...];
    }
}
```

**ملاحظة**: يمكن للدالة أن ترجع القواعد الافتراضية أيضاً عبر التحقق من `$columnName == 'rules'`.

---

## 8. أمثلة عملية متكاملة

### 8.1. نموذج منتج مع فلتر تصنيف ديناميكي

**ملف fields.yaml**:
```yaml
fields:
    search_rules:
        label: شروط البحث المتقدم
        type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
        sort_filters: true
        allow_groups: 2
        filters:
            name:
                id: name
                label: اسم المنتج
                type: string
                operators: ['contains', 'begins_with', 'equal']
            price:
                id: price
                label: السعر
                type: double
                validation: { min: 0, step: 0.01 }
            category_id:
                id: category_id
                label: التصنيف
                type: integer
                input: select
                values: getCategoryOptions   # دالة في الموديل
                operators: ['equal', 'in']
        rules:
            condition: AND
            rules:
                - id: price
                  operator: greater
                  value: 10
```

**في الموديل (Product.php)**:
```php
public function getCategoryOptions()
{
    return Category::pluck('name', 'id')->toArray();
}
```

### 8.2. تطبيق الشروط على استعلام باستخدام QueryBuilderParserHelper

بعد تخزين القواعد في قاعدة البيانات (كـ JSON)، يمكنك استخدامها لتصفية النتائج:

```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

// $product = Product::find($id);
$rules = $product->search_rules; // مصفوفة أو نص JSON

if (!empty($rules)) {
    $query = Product::query();
    $filteredQuery = QueryBuilderParserHelper::parse($query, $rules);
    $results = $filteredQuery->get();
} else {
    $results = Product::all();
}
```

### 8.3. مثال كامل لوحدة تحكم (Controller)

```php
public function filterProducts(Request $request)
{
    $rules = $request->input('rules'); // JSON من الواجهة
    $query = Product::query();
    
    if ($rules) {
        $query = QueryBuilderParserHelper::parse($query, json_decode($rules, true));
    }
    
    $products = $query->paginate(20);
    return view('products.index', compact('products'));
}
```

---

## 9. التعامل مع القيم المخزنة

عند حفظ النموذج، يقوم `QueryBuilder` بتخزين القواعد بصيغة JSON في العمود المرتبط. يمكنك الوصول إليها كـ مصفوفة عبر جعل العمود `jsonable` في الموديل:

```php
class Report extends Model
{
    protected $jsonable = ['query_conditions'];
}
```

ثم استخدامها كالتالي:
```php
$report = Report::find(1);
$rules = $report->query_conditions; // مصفوفة PHP جاهزة
```

إذا لم تستخدم `$jsonable`، يمكنك تحويلها يدوياً:
```php
$rules = json_decode($report->query_conditions, true);
```

---

## 10. الأسئلة الشائعة

### س1: كيف يمكنني ترجمة تسميات الفلاتر والمشغلات؟
يمكنك استخدام صيغة المصفوفة في خاصية `label`:
```yaml
label:
    en: 'Product Name'
    ar: 'اسم المنتج'
```
أيضاً يمكنك استخدام مفاتيح الترجمة في نظام نانوسوفت (عبر `Lang::get`).

### س2: كيف أمنع المستخدم من إضافة أكثر من فلتر معين؟
استخدم إضافة `unique-filter` واجعل الفلتر فريداً:
```yaml
filters:
    category:
        id: category
        label: التصنيف
        unique: true   # أو unique: 'group' ليكن فريداً داخل المجموعة فقط
```

### س3: كيف أتحقق من صحة القيم قبل التطبيق؟
يمكنك الاعتماد على خاصية `validation` في الفلتر، أو تنفيذ التحقق يدوياً قبل استدعاء `QueryBuilderParserHelper`:
```php
$builder = QueryBuilderParserHelper::parse($query, $rules);
// $builder سينفذ الاستعلام فقط إذا كانت القيم صالحة، وإلا سيرمي استثناء.
```

### س4: هل يمكن استخدام هذا العنصر خارج النماذج (مثلاً في واجهة تحكم مستقلة)؟
نعم، يمكنك إنشاء الحقل في أي قالب باستخدام `FormWidget` مباشرة، مع توفير البيانات اللازمة. راجع وثائق نانوسوفت حول استخدام `FormWidget` خارج السياق النموذجي.

### س5: كيف أستدعي القواعد من قاعدة البيانات وأعرضها مرة أخرى في الواجهة؟
عند تحميل النموذج، سيقرأ `FormWidget` القيمة المخزنة تلقائياً ويعيد بناء الواجهة بنفس القواعد. لا تحتاج إلى أي كود إضافي.

---

## 11. الخلاصة

يُعتبر `Nano2\QueryBuilder\FormWidgets\QueryBuilder` أداة قوية ومرنة تتيح للمطورين والمستخدمين النهائيين بناء استعلامات معقدة بسهولة. بفضل دعمه لأنواع متعددة من المُدخلات، والإضافات، والتكامل السلس مع `QueryBuilderParserHelper`، يمكنك إنشاء أنظمة تقارير وبحث متقدمة في وقت قياسي. استخدم هذا التوثيق كمرجع شامل لاستكشاف جميع إمكانيات العنصر والبدء في تطبيقه في مشاريع نانوسوفت الخاصة بك.

---

## التوثيق الإضافي

- [توثيق كلاس `QueryBuilderParserHelper`](./Docs-QueryBuilderParserHelper-ar.md)
