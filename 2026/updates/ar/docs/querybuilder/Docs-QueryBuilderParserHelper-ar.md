## توثيق كلاس `QueryBuilderParserHelper`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder`**

كلاس `QueryBuilderParserHelper` هو جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

يقع هذا الكلاس ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes
```

### مقدمة
كلاس `QueryBuilderParserHelper` هو أداة مساعدة (Helper) لتحليل قواعد الاستعلامات القادمة من مكتبات مثل **jQuery QueryBuilder** وتطبيقها على `Eloquent Builder` أو `Query Builder` في Laravel. يقوم الكلاس بتحويل كائن JSON يمثل مجموعة من القواعد (Rules) إلى شروط `WHERE` و `JOIN` مناسبة، مع دعم العمليات المنطقية (`AND` / `OR`)، والعوامل المختلفة (مثل `=`, `>`, `LIKE`, `IN`, `BETWEEN`, إلخ)، والتعامل مع العلاقات عبر النقاط (dot notation) مثل `customer.name`.

يستند الكلاس بشكل أساسي إلى مكتبة `timgws/QueryBuilderParser` ولكنه يضيف عليها مجموعة من التحسينات والدوال المساعدة مثل:
- استخراج الجداول المستخدمة من القواعد (`getTables`).
- تحويل القواعد إلى استعلام SQL (`toSql`).
- توسيع الماكرو (Macros) المخصصة (`expandMacro`).
- تطبيق القواعد مباشرة على كائن Query Builder (`parse`).
- التعامل مع المدخلات المختلفة (نموذج، Builder، نص JSON).

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$legacy_operators` | `array` | (ثابت) مصفوفة تحويل العوامل القديمة (مثل `=`, `!=`) إلى أسماء العوامل في النظام الجديد. |
| `$operators` | `array` | (ثابت) مصفوفة تربط أسماء العوامل (مثل `equal`) بالعامل الفعلي في SQL (مثل `=`). |
| `$values` | `array` | (ثابت) مصفوفة تحدد كيفية تنسيق القيم لبعض العوامل الخاصة (مثل `between`, `contains`). |

---

### أهم الطرق (Methods)

#### 1. `getTables`
```php
public static function getTables($builder): array
```
**الهدف**: استخراج جميع أسماء الجداول المستخدمة في قواعد الاستعلام (بما في ذلك الجداول الناتجة عن الماكرو).

**المعاملات**:
- `$builder`: مصفوفة القواعد (rules) كما هي قادمة من jQuery QueryBuilder (يجب أن تحتوي على مفتاح `rules` و `condition`).

**الإرجاع**: مصفوفة من أسماء الجداول (بدون تكرار).

**مثال**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'orders.amount', 'operator' => 'greater', 'value' => 100],
        ['field' => 'customer.name', 'operator' => 'begins_with', 'value' => 'A']
    ]
];
$tables = QueryBuilderParserHelper::getTables($rules);
// $tables = ['orders', 'customer']
```

---

#### 2. `findTablesRecursive` (protected)
```php
protected static function findTablesRecursive($rules): array
```
**الهدف**: دالة مساعدة تستخدمها `getTables` للبحث عن الجداول بشكل متكرر في القواعد المتداخلة.

---

#### 3. `toSql`
```php
public static function toSql($builder, $expand = true): ?string
```
**الهدف**: تحويل قواعد الاستعلام إلى جملة SQL قابلة للتنفيذ (جزء WHERE) مع إمكانية توسيع الماكرو.

**المعاملات**:
- `$builder`: مصفوفة القواعد.
- `$expand`: (اختياري) إذا كان `true` يتم توسيع الماكرو (افتراضي `true`).

**الإرجاع**: نص SQL يمثل شرط WHERE (قد يكون `null` إذا كانت القواعد فارغة).

**ملاحظة**: هذه الدالة لا تولد جملة `SELECT` كاملة، بل تولد فقط الجزء الخاص بالشروط.

**مثال**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'orders.amount', 'operator' => 'greater', 'value' => 100],
        ['field' => 'customer.name', 'operator' => 'begins_with', 'value' => 'A']
    ]
];
$sql = QueryBuilderParserHelper::toSql($rules);
// الناتج: (orders.amount > 100 AND customer.name LIKE 'A%')
```

---

#### 4. `parseGroup` (protected)
```php
private static function parseGroup($rule, $expand = false, $wrap = true): string
```
**الهدف**: تحليل مجموعة من القواعد (مجموعة متداخلة) وتحويلها إلى نص SQL.

---

#### 5. `parseRule` (protected)
```php
protected static function parseRule($rule, $expand = false): string
```
**الهدف**: تحليل قاعدة واحدة وتحويلها إلى نص SQL.

---

#### 6. `expandMacro`
```php
protected static function expandMacro($subject, $tables_only = false, $depth_limit = 20)
```
**الهدف**: توسيع الماكرو (Macros) المخصصة في أسماء الحقول أو القيم. الماكرو هي اختصارات معرفة في ملف الإعدادات `nano2.querybuilder::macros.rule` يتم استبدالها بتعبيرات SQL.

**المعاملات**:
- `$subject`: النص الذي قد يحتوي على ماكرو (مثل `%macros.total_with_tax`).
- `$tables_only`: إذا كان `true`، تُعيد قائمة الجداول المستخدمة في الماكرو بدلاً من النص الموسع.
- `$depth_limit`: الحد الأقصى لعدد مرات التوسع التكراري (لمنع الحلقات اللانهائية).

**الإرجاع**: النص بعد توسيع الماكرو، أو مصفوفة من أسماء الجداول إذا كان `tables_only = true`.

**مثال (افتراضي)**:
```php
// بافتراض أن الإعدادات تحتوي على:
// Config::set('nano2.querybuilder::macros.rule', ['total_with_tax' => 'ROUND(amount * 1.14, 2)']);
$expanded = QueryBuilderParserHelper::expandMacro('%macros.total_with_tax');
// الناتج: (ROUND(amount * 1.14, 2))
```

---

#### 7. `runMacros`
```php
public static function runMacros($rule, $x = 1)
```
**الهدف**: طريقة بديلة لتوسيع الماكرو (تستخدم تكراراً محدوداً). تقوم باستبدال جميع `%macros.` بالقيم المقابلة من الإعدادات.

**المعاملات**:
- `$rule`: النص الذي يحتوي على الماكرو.
- `$x`: عداد لمنع التكرار اللانهائي.

**الإرجاع**: النص بعد التوسيع، أو `false` إذا تجاوز الحد الأقصى.

---

#### 8. `parseRuleToQuery`
```php
public static function parseRuleToQuery($query, $rule, $condition)
```
**الهدف**: تطبيق قاعدة واحدة على كائن Query Builder (مثل `Eloquent\Builder`) مع تحديد الشرط المنطقي (`AND` / `OR`).

**المعاملات**:
- `$query`: كائن Builder (يمكن أن يكون `EloquentBuilder` أو `Query\Builder`).
- `$rule`: مصفوفة القاعدة (تحتوي على `field`, `operator`, `value`).
- `$condition`: `'and'` أو `'or'` (يُستخدم في ربط الشروط).

**الإرجاع**: كائن Builder بعد إضافة الشرط.

**مثال**:
```php
$query = DB::table('orders');
$rule = ['field' => 'amount', 'operator' => 'greater', 'value' => 100];
$query = QueryBuilderParserHelper::parseRuleToQuery($query, $rule, 'and');
// يضيف: where 'amount' > 100
```

---

#### 9. `expandRule` (protected)
```php
protected static function expandRule($rule): array
```
**الهدف**: توسيع الحقل والقيمة في القاعدة إذا كانا يحتويان على ماكرو أو قيمة خام (محاطة بـ `` ` ``). تُعيد مصفوفة `[$field, $op, $value]` بعد التوسيع.

---

#### 10. `parse` (الدالة الرئيسية)
```php
public static function parse($queryBuilder, $rules, $rows, $is_exception = true)
```
**الهدف**: تطبيق مجموعة كاملة من القواعد على كائن Builder باستخدام مكتبة `timgws\QueryBuilderParser`. هذه الدالة هي الواجهة الرئيسية للاستخدام.

**المعاملات**:
- `$queryBuilder`: يمكن أن يكون:
  - كائن Builder (`EloquentBuilder` أو `Query\Builder`).
  - كائن Model (Eloquent) – سيتم تحويله إلى Builder.
  - اسم كلاس (string) – سيتم إنشاء كائن منه.
- `$rules`: القواعد كما هي قادمة من jQuery QueryBuilder. يمكن أن تكون:
  - مصفوفة (array).
  - نص JSON.
- `$rows`: مصفوفة بأسماء الحقول المسموح بها (يجب تمريرها للمكتبة). إذا كانت فارغة، سيتم استخراجها تلقائياً من القواعد باستخدام `setFields`.
- `$is_exception`: إذا كان `true`، سيتم رمي استثناء عند حدوث خطأ (مثل عدم صحة المدخلات). إذا كان `false`، يُعيد المدخل الأصلي دون تعديل.

**الإرجاع**: كائن Builder بعد إضافة جميع الشروط.

**مثال**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'status', 'operator' => 'equal', 'value' => 'completed'],
        ['field' => 'amount', 'operator' => 'greater', 'value' => 100]
    ]
];
$query = DB::table('orders');
$query = QueryBuilderParserHelper::parse($query, $rules, []);
// الناتج: استعلام مع where status = 'completed' and amount > 100
```

---

#### 11. `setFields`
```php
public static function setFields($rules, $is_encode = true, $is_decode = true): array
```
**الهدف**: استخراج أسماء الحقول (الـ keys) المستخدمة في القواعد. تُستخدم لإنشاء قائمة الحقول المسموحة لمكتبة `QueryBuilderParser`.

**المعاملات**:
- `$rules`: القواعد (يمكن أن تكون مصفوفة أو JSON).
- `$is_encode`: إذا كان `true` وكانت `$rules` مصفوفة، يتم تحويلها إلى JSON أولاً.
- `$is_decode`: إذا كان `true` وكانت `$rules` نصاً، يتم فك تشفير JSON.

**الإرجاع**: مصفوفة associative بأسماء الحقول (المفتاح والقيمة نفس الاسم).

**مثال**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'status', 'operator' => 'equal', 'value' => 'completed'],
        ['field' => 'amount', 'operator' => 'greater', 'value' => 100]
    ]
];
$fields = QueryBuilderParserHelper::setFields($rules);
// $fields = ['status' => 'status', 'amount' => 'amount']
```

---

#### 12. `decodeJSON`
```php
public static function decodeJSON($json): object
```
**الهدف**: فك تشفير نص JSON إلى كائن مع معالجة الأخطاء.

**المعاملات**:
- `$json`: النص المراد فك تشفيره.

**الإرجاع**: كائن `stdClass`.

**الاستثناءات**: يرمي `Exception` إذا كان JSON غير صالح.

---

#### 13. `isNested`
```php
public static function isNested($rule): bool
```
**الهدف**: التحقق مما إذا كانت القاعدة تحتوي على قواعد متداخلة (أي أنها مجموعة وليس قاعدة بسيطة).

**المعاملات**:
- `$rule`: كائن أو مصفوفة تمثل قاعدة (قد تحتوي على مفتاح `rules`).

**الإرجاع**: `true` إذا كانت مجموعة، وإلا `false`.

---

#### 14. `getQueryOrModel`
```php
public static function getQueryOrModel($obj, $is_model = false, $is_exception = true)
```
**الهدف**: تحويل المدخل (قد يكون اسم كلاس، كائن Model، أو كائن Builder) إلى كائن Builder (أو Model حسب الطلب). تُستخدم لتوحيد التعامل مع المدخلات المختلفة.

**المعاملات**:
- `$obj`: المدخل (string, Model, Builder).
- `$is_model`: إذا كان `true`، تُعيد كائن Model بدلاً من Builder.
- `$is_exception`: إذا كان `true`، ترمي استثناء عند فشل التحويل.

**الإرجاع**: كائن Builder أو Model (حسب `$is_model`).

**مثال**:
```php
$builder = QueryBuilderParserHelper::getQueryOrModel('App\Models\Order'); // يُعيد Builder لجدول orders
$model = QueryBuilderParserHelper::getQueryOrModel('App\Models\Order', true); // يُعيد كائن Order جديد
```

---

#### 15. `isModel`
```php
public static function isModel($item): bool
```
**الهدف**: التحقق مما إذا كان الكائن نموذج Eloquent (من Laravel أو OctoberCMS).

---

#### 16. `isBuilder`
```php
public static function isBuilder($item): bool
```
**الهدف**: التحقق مما إذا كان الكائن Builder (من Laravel Query Builder أو Eloquent Builder أو October builders).

---

### أمثلة عملية شاملة

#### مثال 1: تطبيق قواعد بسيطة على استعلام
**السيناريو**: لدينا استعلام على جدول `orders` ونريد تطبيق شرطين: `status = 'completed'` و `amount > 100`.

**الكود**:
```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'status', 'operator' => 'equal', 'value' => 'completed'],
        ['field' => 'amount', 'operator' => 'greater', 'value' => 100]
    ]
];

$query = DB::table('orders');
$optimizedQuery = QueryBuilderParserHelper::parse($query, $rules, []);
$results = $optimizedQuery->get();
```

**الاستعلام الناتج**:
```sql
select * from orders where status = 'completed' and amount > 100
```

---

#### مثال 2: استخدام علاقات (dot notation)
**السيناريو**: نريد البحث في الطلبات حيث اسم العميل يبدأ بحرف "A".

**الكود**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'customer.name', 'operator' => 'begins_with', 'value' => 'A']
    ]
];

$query = DB::table('orders');
$optimizedQuery = QueryBuilderParserHelper::parse($query, $rules, []);
// سيقوم تلقائياً بعمل join مع جدول customers إذا كان العلاقة معرفة
```

**الاستعلام الناتج (تقريبي)**:
```sql
select * from orders inner join customers on orders.customer_id = customers.id where customers.name like 'A%'
```

---

#### مثال 3: استخدام قواعد متداخلة (Nested Conditions)
**السيناريو**: (status = 'completed' OR status = 'pending') AND amount > 100

**الكود**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'condition' => 'OR',
            'rules' => [
                ['field' => 'status', 'operator' => 'equal', 'value' => 'completed'],
                ['field' => 'status', 'operator' => 'equal', 'value' => 'pending']
            ]
        ],
        ['field' => 'amount', 'operator' => 'greater', 'value' => 100]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**الاستعلام الناتج**:
```sql
select * from orders where (status = 'completed' or status = 'pending') and amount > 100
```

---

#### مثال 4: استخدام عامل `IN`
**السيناريو**: الحصول على الطلبات ذات الحالات المحددة (completed, pending, cancelled).

**الكود**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'status',
            'operator' => 'in',
            'value' => 'completed,pending,cancelled' // يمكن أيضاً مصفوفة
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**الاستعلام الناتج**:
```sql
select * from orders where status in ('completed', 'pending', 'cancelled')
```

---

#### مثال 5: استخدام عامل `BETWEEN`
**السيناريو**: الطلبات التي مبلغها بين 100 و 500.

**الكود**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'amount',
            'operator' => 'between',
            'value' => [100, 500]
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**الاستعلام الناتج**:
```sql
select * from orders where amount between 100 and 500
```

---

#### مثال 6: استخدام عامل `LIKE` مع `contains`
**السيناريو**: البحث عن الطلبات التي تحتوي ملاحظاتها على كلمة "urgent".

**الكود**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'notes',
            'operator' => 'contains',
            'value' => 'urgent'
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**الاستعلام الناتج**:
```sql
select * from orders where notes like '%urgent%'
```

---

#### مثال 7: استخدام عامل `IS NULL`
**السيناريو**: الطلبات التي ليس لها تاريخ حذف (deleted_at = null).

**الكود**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'deleted_at',
            'operator' => 'is_null',
            'value' => null // القيمة لا تُستخدم
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**الاستعلام الناتج**:
```sql
select * from orders where deleted_at is null
```

---

#### مثال 8: استخدام الماكرو (Macros) المخصصة
**السيناريو**: لدينا ماكرو معرف باسم `total_with_tax` يقوم بحساب المبلغ شامل الضريبة. نريد البحث عن الطلبات حيث `total_with_tax` أكبر من 500.

**الإعدادات المسبقة** (في ملف config أو Service Provider):
```php
Config::set('nano2.querybuilder::macros.rule', [
    'total_with_tax' => 'ROUND(amount * 1.14, 2)'
]);
```

**الكود**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'macros.total_with_tax', // استخدام الماكرو في الحقل
            'operator' => 'greater',
            'value' => 500
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**الاستعلام الناتج**:
```sql
select * from orders where ROUND(amount * 1.14, 2) > 500
```

---

#### مثال 9: استخدام القيمة المحاطة بـ `` ` `` للإشارة إلى عمود آخر
**السيناريو**: البحث عن الطلبات حيث `amount` أكبر من `tax` (عمود آخر).

**الكود**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        [
            'field' => 'amount',
            'operator' => 'greater',
            'value' => '`tax`' // القيمة محاطة بـ ` لتدل على عمود
        ]
    ]
];

$query = DB::table('orders');
$result = QueryBuilderParserHelper::parse($query, $rules, [])->get();
```

**الاستعلام الناتج**:
```sql
select * from orders where amount > tax
```

---

#### مثال 10: استخراج الجداول المستخدمة
**السيناريو**: قبل تنفيذ الاستعلام، نريد معرفة الجداول التي ستتأثر.

**الكود**:
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['field' => 'orders.amount', 'operator' => 'greater', 'value' => 100],
        ['field' => 'customer.name', 'operator' => 'begins_with', 'value' => 'A']
    ]
];
$tables = QueryBuilderParserHelper::getTables($rules);
// $tables = ['orders', 'customer']
```

---

#### مثال 11: تحويل القواعد إلى SQL (بدون تنفيذ)
**السيناريو**: الحصول على نص SQL للشروط فقط.

**الكود**:
```php
$sqlWhere = QueryBuilderParserHelper::toSql($rules);
// $sqlWhere = "(orders.amount > 100 AND customer.name LIKE 'A%')"
```

---

#### مثال 12: استخدام `parse` مع Model مباشرة
**السيناريو**: تمرير اسم كلاس Model بدلاً من Builder.

**الكود**:
```php
$rules = [ /* ... */ ];
$query = QueryBuilderParserHelper::parse('App\Models\Order', $rules, []);
// يُعيد Builder جاهز للتطبيق
$orders = $query->get();
```

---

#### مثال 13: التعامل مع JSON قادم من الطلب (Request)
**السيناريو**: واجهة API تستقبل قواعد بصيغة JSON.

**الكود**:
```php
public function search(Request $request)
{
    $rulesJson = $request->input('rules'); // نص JSON
    $query = DB::table('orders');
    $query = QueryBuilderParserHelper::parse($query, $rulesJson, []);
    return $query->paginate();
}
```

---

#### مثال 14: استخدام `setFields` لاستخراج الحقول المسموحة
**السيناريو**: نريد الحصول على قائمة الحقول التي استخدمها المستخدم في القواعد.

**الكود**:
```php
$rules = [ /* ... */ ];
$fields = QueryBuilderParserHelper::setFields($rules);
// يمكن استخدامها للتحقق من الصلاحيات أو لتغذية مكتبة QueryBuilderParser.
```

---

#### مثال 15: التحقق من نوع المدخلات
**السيناريو**: التأكد من أن كائن معين هو Builder قبل استخدامه.

**الكود**:
```php
if (QueryBuilderParserHelper::isBuilder($someObject)) {
    // يمكن تطبيق القواعد
}
```

---

### ملاحظات مهمة

- الكلاس يعتمد على مكتبة `timgws/QueryBuilderParser`. يجب تثبيتها عبر Composer: `composer require timgws/querybuilderparser`
- الماكرو (Macros) تُعرّف في ملف الإعدادات `nano2.querybuilder::macros.rule`. يمكن إضافتها عبر `Config::set` أو من خلال ملفات الإعدادات الخاصة بالتطبيق.
- عند استخدام الحقول ذات النقاط (مثل `customer.name`)، يفترض الكلاس أن الجزء الأول هو اسم علاقة (Relationship) وسيقوم تلقائياً بعمل `join` إذا كانت العلاقة معرفة في النموذج. هذا يتطلب وجود علاقة في النموذج بنفس الاسم.
- دالة `parse` تستخدم `setFields` لاستخراج الحقول المسموحة إذا لم تمرر `$rows`. هذا يضمن أن المكتبة لن تسمح بحقول غير موجودة في القواعد.
- في حالة وجود أخطاء في JSON أو في المدخلات، يمكن التحكم في رمي الاستثناء عبر `$is_exception`.

---

### الخلاصة
كلاس `QueryBuilderParserHelper` يقدم واجهة متكاملة وسهلة الاستخدام لتحليل قواعد jQuery QueryBuilder وتطبيقها على استعلامات Laravel. بفضل دعمه للعلاقات، الماكرو، والعوامل المتنوعة، يمكن للمطورين بناء واجهات بحث متقدمة وتقارير ديناميكية بسرعة وأمان. يتكامل الكلاس بشكل وثيق مع باقي مكونات حزمة `Nano2.QueryBuilder`، مثل `ReportQueryConverter` و `ReportsManager`، مما يجعله أداة قوية في بناء أنظمة التقارير.

---

## التوثيق الإضافي

- [توثيق عنصر الواجهة `QueryBuilder`](./Docs-FormWidgets-QueryBuilder-ar.md)

