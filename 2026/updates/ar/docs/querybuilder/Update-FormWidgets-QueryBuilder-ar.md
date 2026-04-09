## 2026-03-11 – 2026-04-09

### إضافة نظام بناء الاستعلامات البصرية (Query Builder) ومعالج الشروط المتقدم  
ضمن الوحدة البرمجية  `Nano2.QueryBuilder`


---

### 1. مقدمة

تم تطوير نظام **بناء الاستعلامات البصري** (Visual Query Builder) لتوفير واجهة تفاعلية سهلة الاستخدام تسمح للمستخدمين النهائيين والمطورين بإنشاء شروط استعلام معقدة دون الحاجة إلى كتابة SQL أو منطق برمجي يدوي. يعتمد النظام على مكتبة `jQuery QueryBuilder` من ناحية الواجهة الأمامية، ويقدّم خلفية متكاملة ضمن برمجيات نانوسوفت عبر `FormWidget` مخصص وكلاس مساعد لتحويل الشروط إلى استعلامات صالحة للتنفيذ على نماذج البيانات (Models).

يهدف هذا التحديث إلى تمكين حالات استخدام مثل:
- بناء تقارير ديناميكية بشروط متعددة المستويات (AND/OR) وبأنواع مختلفة من المُدخلات (نص، أرقام، تواريخ، قوائم منسدلة، خانات اختيار، إلخ).
- تطبيق الشروط مباشرة على `Model` أو `Builder` عبر كلاس `QueryBuilderParserHelper`.
- إعادة استخدام تعريف الفلاتر والعمليات عبر تكوين YAML أو عبر دوال في الموديل.
- دعم الإضافات (plugins) مثل الترتيب بالسحب والإفلات، عرض وصف الفلاتر، تخصيص الأيقونات، وغيرها.

---

### 2. المكونات المطورة

#### 2.1 `QueryBuilderParserHelper` – كلاس معالجة الشروط وتحويلها إلى استعلامات  
- **المسار**: `Nano2\QueryBuilder\Classes\QueryBuilderParserHelper`
- **الوظيفة**: كلاس مساعد (Helper) يقوم بتحليل كائن الشروط (القواعد) القادم من الـ FormWidget، وتطبيقها على استعلام Builder، مع دعم المتغيرات الكلية (macros) والجداول المتداخلة والمشغلين المتعددين.

**أبرز الدوال**:

| الدالة | الوصف |
|--------|--------|
| `parse($builder, $rules, $rows, $is_exception)` | الدالة الرئيسية: تستقبل `$builder` (موديل، سلسلة اسم موديل، أو كائن Builder) ومصفوفة الشروط `$rules` (عادةً من `getRules()` للـ QueryBuilder)، وتُعيد `Builder` مطبَّقًا عليه الشروط. |
| `setFields($rules)` | تستخرج أسماء الحقول المستخدمة في الشروط لتمريرها إلى مكتبة `timgws\QueryBuilderParser` (ضروري لتحديد الأعمدة المسموحة). |
| `decodeJSON($json)` | تحويل سلسلة JSON إلى كائن للتحقق من صحة الشروط. |
| `isNested($rule)` | تتحقق إذا كانت القاعدة تحتوي على مجموعة فرعية من القواعد (للتعامل مع الشروط المتداخلة). |
| `getQueryOrModel($obj, $is_model, $is_exception)` | تستقبل كائنًا وقد يكون `Model`، أو اسم كلاس، أو `Builder`، وتُعيد إما كائن `Builder` أو الموديل نفسه حسب الحاجة. |
| `toSql($builder, $expand)` | (مأخوذ من LibreNMS) تحويل الشروط إلى جملة SQL نصية للتصحيح أو العرض. |

**آلية العمل**:
1. يتم تحويل كائن `$rules` (عادةً مصفوفة أو JSON) إلى كائن stdClass باستخدام `decodeJSON`.
2. تُستخرج الحقول المستخدمة عبر `setFields`.
3. يُنشأ كائن من مكتبة `timgws\QueryBuilderParser` مع تمرير الحقول المسموحة.
4. يُستدعى `parse($json, $query)` لتطبيق جميع `where` و `orWhere` والمجموعات المنطقية.
5. يتم إرجاع كائن `Builder` المعدل.

**مثال استخدام**:
```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

$rules = [
    'condition' => 'AND',
    'rules' => [
        ['id' => 'price', 'operator' => 'greater', 'value' => 100],
        ['id' => 'category', 'operator' => 'equal', 'value' => 5]
    ]
];
$query = Product::query();
$finalQuery = QueryBuilderParserHelper::parse($query, $rules);
$products = $finalQuery->get();
```

---

#### 2.2 `QueryBuilder` – عنصر واجهة (FormWidget) لبناء الاستعلامات  
- **المسار**: `Nano2\QueryBuilder\FormWidgets\QueryBuilder`
- **الوظيفة**: يوفر حقلاً مخصصًا في نماذج نانوسوفت يعرض واجهة jQuery QueryBuilder للمستخدم، ويخزن القواعد بشكل JSON في قاعدة البيانات.

**الخصائص الرئيسية (قابلة للتكوين عبر YAML أو خاصيات الكلاس)**:

| الخاصية | الوصف | القيمة الافتراضية |
|---------|------|-------------------|
| `filters` | تعريف الفلاتر المتاحة (كائنات من نوع `Filter`). يمكن أن تكون مصفوفة أو سلسلة تشير إلى دالة في الموديل. | `[]` |
| `rules` | القواعد الافتراضية التي تظهر عند تحميل الواجهة. | `[]` |
| `sort_filters` | ترتيب الفلاتر أبجدياً حسب الترجمة. | `false` |
| `allow_groups` | الحد الأقصى لعدد المجموعات المتداخلة (`-1` = غير محدود، `0` = لا مجموعات). | `-1` |
| `allow_empty` | السماح بوجود قواعد فارغة. | `false` |
| `conditions` | قائمة بالروابط المنطقية المسموحة (`AND`, `OR`). | `['AND', 'OR']` |
| `default_condition` | الرابط الافتراضي للمجموعة الجذرية. | `'AND'` |
| `inputs_separator` | الفاصل بين حقول الإدخال المتعددة (مثل BETWEEN). | `' , '` |
| `display_empty_filter` | عرض خيار فارغ في قائمة الفلاتر. | `true` |
| `select_placeholder` | النص المعروض عند عدم اختيار فلتر. | `'------'` |
| `plugins` | قائمة الإضافات المفعَّلة (مثل `sortable`, `filter-description`). | `[]` |
| `isShowParseSqlBtn` | إظهار زر لترجمة القواعد إلى SQL. | `false` |
| `isShowResetBtn` / `isShowSetBtn` / `isShowGetBtn` | أزرار إعادة الضبط، تحميل القواعد، وجلب القواعد. | `true` |

**آلية العمل في النموذج**:
1. في دالة `init()` يتم تحميل الإعدادات من YAML أو من الخاصيات.
2. في `prepareVars()` يتم تجهيز مصفوفة `filters` بعد معالجتها (دعم الترجمة، واستدعاء دوال الموديل للحصول على الخيارات).
3. يتم تحويل `$value` (القواعد المخزنة JSON) إلى كائن مناسب للجافاسكريبت.
4. يتم إضافة الأصول (CSS/JS) الخاصة بالمكتبة والإضافات.
5. عند عرض القالب الجزئي (`_query_builder_basic.htm`)، يتم تهيئة الـ jQuery QueryBuilder باستخدام الخيارات والقواعد.
6. عند تغيير المستخدم للقواعد، يتم تحديث حقل خفي (`input[type=hidden]`) بقيمة JSON، وعند حفظ النموذج تُخزَّن القيمة في قاعدة البيانات.

**دعم مصادر الخيارات الديناميكية**:
يمكن تعريف `values` في الفلتر كسلسلة نصية تشير إلى:
- اسم دالة في الموديل الحالي (مثل `getCategoryOptions`).
- اسم دالة ثابتة بصيغة `ClassName::method`.
- يتم استدعاؤها أثناء تحضير الفلاتر وتُمرر القيم الحالية والسياق.

**مثال تكوين YAML**:
```yaml
query_builder:
    type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
    filters:
        category:
            id: category
            label: التصنيف
            type: integer
            input: select
            values:
                1: كتب
                2: أفلام
                3: موسيقى
            operators: ['equal', 'not_equal', 'in']
        price:
            id: price
            label: السعر
            type: double
            validation:
                min: 0
                step: 0.01
    rules: 
        condition: AND
        rules: []
    allow_groups: 3
    sort_filters: true
    plugins:
        sortable: null
        filter-description: { mode: 'popover' }
```

---

### 3. الخيارات المتقدمة في الفلاتر (Filters)

يتبع كل فلتر نفس البنية المحددة في مكتبة jQuery QueryBuilder مع دعم إضافات نانوسوفت:

| الخاصية | الوصف |
|---------|------|
| `id` | معرف فريد للفلتر. |
| `label` | النص المعروض. يدعم الترجمة كمصفوفة أو كسلسلة. |
| `type` | نوع البيانات: `string`, `integer`, `double`, `date`, `time`, `datetime`, `boolean`. |
| `input` | نوع عنصر الإدخال: `text`, `number`, `textarea`, `radio`, `checkbox`, `select`. |
| `values` | مصفوفة من الخيارات (للـ `select`, `radio`, `checkbox`). يمكن أن تكون دالة أو مصفوفة ثابتة. |
| `operators` | قائمة بالمشغلات المسموحة لهذا الفلتر (مثل `equal`, `contains`, `between`). |
| `default_operator` | المشغل الافتراضي. |
| `validation` | قواعد التحقق من صحة القيمة: `min`, `max`, `step`, `format` (regex أو تنسيق تاريخ)، `allow_empty_value`. |
| `placeholder` | نص توجيهي داخل حقل الإدخال. |
| `plugin` | اسم إضافة jQuery (مثل `datepicker`, `selectize`). |
| `plugin_config` | إعدادات خاصة بالإضافة. |

**طريقتان لتزويد الفلاتر**:
1. **مباشرة في ملف YAML** (كما في المثال أعلاه).
2. **عبر دالة في الموديل**:
   - دالة عامة: `getQueryBuilderFiltersOptions($columnName)`
   - دالة خاصة باسم الحقل: `get{FieldName}QueryBuilderFiltersOptions($fieldName, $columnName)`

**مثال لدالة في الموديل**:
```php
public function getQueryBuilderFiltersOptions($columnName)
{
    if ($columnName == 'filters') {
        return [
            ['id' => 'status', 'label' => 'الحالة', 'type' => 'integer', 'input' => 'select', 
             'values' => [1 => 'نشط', 0 => 'غير نشط']],
            // ... 
        ];
    }
    if ($columnName == 'rules') {
        return ['condition' => 'AND', 'rules' => []];
    }
    return [];
}
```

---

### 4. دعم الإضافات (Plugins) في الـ FormWidget

يمكن تفعيل مجموعة من الإضافات التي توفرها مكتبة jQuery QueryBuilder مع إعدادات مخصصة:

| اسم الإضافة | الوصف | الخيارات |
|-------------|-------|-----------|
| `sortable` | تمكين السحب والإفلات لإعادة ترتيب القواعد والمجموعات. | `icon`, `inherit_no_sortable` |
| `filter-description` | عرض وصف توضيحي للفلتر (بـ Inline, Popover, أو Bootbox). | `mode`, `icon` |
| `bt-tooltip-errors` | عرض أخطاء التحقق كـ Tooltip من Bootstrap. | `placement` |
| `bt-checkbox` | تنسيق عناصر `checkbox` و `radio` باستخدام Bootstrap. | `color` |
| `unique-filter` | منع استخدام نفس الفلتر أكثر من مرة (عالمياً أو داخل المجموعة). | – |
| `not-group` | إضافة خيار "NOT" لعكس منطق المجموعة. | `icon_checked`, `icon_unchecked` |
| `invert` | زر قلب الشرط (تحويل `=` إلى `!=`، و `AND` إلى `OR`...). | `recursive`, `invert_rules` |

**طريقة التفعيل في YAML**:
```yaml
plugins:
    sortable: { icon: 'bi-sort-down' }
    filter-description: { mode: 'bootbox', icon: 'bi-info-circle' }
    bt-tooltip-errors: { placement: 'top' }
```

---

### 5. مثال متكامل لاستخدام FormWidget وParserHelper

#### 5.1 تعريف الحقل في نموذج (model fields.yaml)
```yaml
fields:
    query_conditions:
        label: شروط التقرير
        type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
        filters:
            name: { id: name, label: الاسم, type: string }
            created_at: { id: created_at, label: تاريخ الإنشاء, type: date, input: datepicker, plugin: datepicker }
        allow_groups: 2
        sort_filters: true
        plugins: { 'sortable': null, 'filter-description': { mode: 'inline' } }
```

#### 5.2 حفظ القواعد من المتحكم (Controller)
عند حفظ النموذج، سيتم تخزين القيمة كـ JSON في العمود المرتبط بالحقل.

#### 5.3 استخدام القواعد في الاستعلام
```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

$reportRules = $model->query_conditions; // JSON string or array
if (!empty($reportRules)) {
    $query = Product::query();
    $query = QueryBuilderParserHelper::parse($query, $reportRules);
    $results = $query->get();
}
```

---

### 6. القيمة المضافة

- **للمطورين**: اختصار كبير في وقت تطوير تقارير وشاشات البحث؛ لم يعد هناك حاجة لكتابة شروط `where` معقدة يدويًا. توحيد طريقة تعريف الفلاتر وإعادة استخدامها عبر المشاريع في بيئة نانوسوفت.
- **للمستخدمين النهائيين**: واجهة بديهية وسهلة لبناء استعلامات مخصصة دون الحاجة إلى معرفة SQL أو بنية البيانات الخلفية.
- **للنظام**: فصل منطق بناء الاستعلام عن واجهة المستخدم، مع إمكانية التوسع بإضافات جديدة وتكامل سلس مع نظام نانوسوفت (ترجمة، صلاحيات، دوال موديل ديناميكية).

---

### 7. الخاتمة

يمثل تحديث `Nano2.QueryBuilder` إضافة نوعية لمنصة نانوسوفت، حيث يزود المطورين بأداة قوية ومرنة لإنشاء شروط استعلام معقدة بطريقة بصرية وآمنة. بفضل الدمج بين مكتبة `jQuery QueryBuilder` القوية من جهة، وكلاس `QueryBuilderParserHelper` الذي يعالج الشروط ويطبقها على `Builder` من جهة أخرى، أصبح بالإمكان بناء أنظمة تقارير وتصفية متقدمة بسرعة وجودة عالية. كما أن التصميم المعياري للـ `FormWidget` يسمح بإعادة استخدام المكون في أي نموذج مع دعم كامل لتخصيص الفلاتر والإضافات.

---

## الوثائق المرجعية

- [توثيق عنصر الواجهة `QueryBuilder`](./docs/querybuilder/Docs-FormWidgets-QueryBuilder-ar.md)
- [توثيق كلاس `QueryBuilderParserHelper`](./docs/querybuilder/Docs-QueryBuilderParserHelper-ar.md)

