## توثيق كلاس `ReportQueryValidator`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### مقدمة
كلاس `ReportQueryValidator` هو المسؤول عن **التحقق الشامل من صحة تكوين التقرير (Query Configuration)** قبل تمريره إلى `ReportQueryConverter` للتنفيذ. يقوم بإجراء سلسلة من الفحوصات للتأكد من أن التكوين يتبع القواعد المحددة، وجميع المراجع (الجداول، الأعمدة، العلاقات) صحيحة وموجودة في السجل (`ReportSchemaRegistry`)، كما يقوم بإصدار تحذيرات تتعلق بالأداء والأمان. يهدف هذا الكلاس إلى منع الأخطاء المبكرة وضمان تجربة مستخدم سلسة.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$registry` | `ReportSchemaRegistry` | مرجع إلى سجل المخطط للوصول إلى معلومات الجداول والأعمدة والعلاقات. |
| `$errors` | `array` | مصفوفة لتجميع أخطاء التحقق (تمنع تنفيذ التقرير). |
| `$warnings` | `array` | مصفوفة لتجميع تحذيرات الأداء والأمان (لا تمنع التنفيذ). |

---

### أهم الطرق

#### 1. `__construct(ReportSchemaRegistry $registry)`
- يقوم بحقن `ReportSchemaRegistry` المستخدم في جميع عمليات التحقق.

#### 2. `validate(array $queryConfig): array`
**الهدف**: تنفيذ جميع خطوات التحقق على تكوين التقرير وإرجاع النتيجة.

**الخطوات**:
- تهيئة `$errors` و `$warnings`.
- التحقق من البنية الأساسية (`validateBasicStructure`). إذا فشل، يتوقف (لأن الأخطاء التالية قد تكون غير ذات معنى).
- التحقق من الجدول (`validateTable`).
- التحقق من الأعمدة (`validateColumns`).
- التحقق من الـ joins (`validateJoins`).
- التحقق من الشروط (`validateConditions`).
- التحقق من GROUP BY (`validateGroupBy`).
- التحقق من ORDER BY (`validateSortBy`).
- التحقق من LIMIT (`validateLimit`).
- إجراء فحوصات الأمان (`performSecurityChecks`).
- إجراء فحوصات الأداء (`performPerformanceChecks`).
- إنشاء ملخص (`generateValidationSummary`).

**الإرجاع**: مصفوفة تحتوي على:
- `valid`: boolean (true إذا كانت `$errors` فارغة).
- `errors`: مصفوفة من رسائل الأخطاء.
- `warnings`: مصفوفة من رسائل التحذير.
- `summary`: مصفوفة تحتوي على ملخص (مثل درجة التعقيد، عدد الأعمدة، إلخ).

#### 3. `validateBasicStructure(array $queryConfig)`
- يستخدم `Illuminate\Support\Facades\Validator` للتحقق من وجود المفاتيح الأساسية بالتنسيق الصحيح.
- **القواعد**:
  - `table` مطلوب وهو مصفوفة.
  - `table.name` مطلوب ونصي.
  - `columns` مطلوب ومصفوفة تحتوي على عنصر واحد على الأقل.
  - `joins` (اختياري) مصفوفة.
  - `conditions` (اختياري) مصفوفة.
  - `groupBy` (اختياري) مصفوفة.
  - `sortBy` (اختياري) مصفوفة.
  - `limit` (اختياري) عدد صحيح بين 1 و50000.

#### 4. `validateTable(array $queryConfig)`
- يتحقق من وجود الجدول في السجل (`$registry->hasTable($tableName)`).
- إذا كان الجدول غير موجود، يضيف خطأ.
- (معلق) التحقق من الصلاحيات إذا كانت موجودة في مخطط الجدول.
- إذا تم تحديد `limit`، يتحقق مما إذا كان يتجاوز `maxRows` المحدد للجدول (إذا وجد) ويضيف تحذيراً.

#### 5. `validateColumns(array $queryConfig)`
- يحصل على قائمة الأعمدة المتاحة من السجل (`$registry->getTableColumns($tableName)`).
- لكل عمود في التكوين، يستدعي `validateColumn`.
- إذا تجاوز عدد الأعمدة 50، يضيف تحذيراً.

#### 6. `validateColumn(array $column, int $index, array $availableColumns, string $tableName)`
- يتحقق من وجود `name` و `type` (اختياري) باستخدام Validator.
- يتحقق من أن اسم العمود موجود في `$availableColumns`.
- إذا تم تحديد `alias`، يتحقق من أنه يتبع نمط `[a-zA-Z_][a-zA-Z0-9_]*`.

#### 7. `validateJoins(array $queryConfig)`
- يحصل على قائمة العلاقات المتاحة للجدول الرئيسي (`$registry->getTableRelationships($mainTable)`).
- لكل join، يستدعي `validateJoin`.
- إذا تجاوز عدد الـ joins 5، يضيف تحذيراً.

#### 8. `validateJoin(array $join, int $index, array $availableRelationships, string $mainTable)`
- يتحقق من وجود `table`, `type` (left/right/inner), `local_key`, `foreign_key` باستخدام Validator.
- يتحقق من أن العلاقة موجودة في `$availableRelationships` (باستخدام مفتاح `$join['key']` أو اسم الجدول).
- يتحقق من أن الجدول المرتبط موجود في السجل (`$registry->hasTable($joinTable)`).
- إذا تم تحديد `selectedColumns`، يقوم بالتحقق من كل عمود باستخدام `validateColumn` مع الجدول المرتبط.

#### 9. `validateConditions(array $queryConfig)`
- يستدعي `validateConditionGroup` على مصفوفة الشروط.
- يحسب عدد الشروط باستخدام `countConditions`؛ إذا تجاوز 20، يضيف تحذيراً.

#### 10. `validateConditionGroup(array $conditions, array $queryConfig, string $path = 'conditions')`
- يتكرر عبر كل شرط. إذا كان الشرط يحتوي على مفتاح `conditions` (مجموعة متداخلة)، يستدعي نفسه. وإلا، يستدعي `validateCondition`.

#### 11. `validateCondition(array $condition, array $queryConfig, string $path)`
- يتحقق من وجود `field`, `operator`, `value` بالتنسيق الصحيح باستخدام Validator.
- يتحقق من أن الحقل (`field.name`) متاح عبر `isFieldAvailable`.
- يتحقق من أن العامل (`operator.value`) صالح عبر `isValidOperator`.
- يستدعي `validateConditionValue` للتحقق من القيمة حسب نوع العامل.

#### 12. `validateConditionValue(array $condition, string $path)`
- **للعوامل `in` / `not_in`**: يتأكد أن القيمة مصفوفة أو نص (يمكن تحويله إلى مصفوفة).
- **للعوامل `between` / `not_between`**: يتأكد أن القيمة مصفوفة وتحتوي على عنصرين.
- **للعوامل `is_null` / `is_not_null`**: لا تحتاج قيمة.
- للحالات الأخرى: إذا كانت القيمة فارغة، يضيف تحذيراً.

#### 13. `validateGroupBy(array $queryConfig)`
- لكل عنصر في `groupBy`، يتحقق من وجود `groupBy` ومصفوفة تحتوي على `name`، وأيضاً `aggregateFn` و `aggregateBy` إذا وجدا.
- يتحقق من أن دالة التجميع (`aggregateFn.value`) هي إحدى: `count, sum, avg, min, max, group_concat`.
- يتحقق من أن الحقل المجموع (`groupBy.name`) موجود عبر `isFieldAvailable`.
- إذا كان `aggregateBy.name` موجوداً، يتحقق من وجوده (مع السماح بـ `*` فقط مع COUNT).

#### 14. `validateSortBy(array $queryConfig)`
- لكل عنصر في `sortBy`، يتحقق من وجود `column` و `direction` بالتنسيق الصحيح.
- يتحقق من أن `direction.value` إما `asc` أو `desc`.
- يتحقق من أن الحقل (`column.name`) موجود عبر `isFieldAvailable`.

#### 15. `validateLimit(array $queryConfig)`
- إذا وجد `limit`، يتحقق من أنه رقم موجب.
- إذا تجاوز 50000، يضيف خطأ.
- إذا تجاوز 10000، يضيف تحذيراً.

#### 16. `isFieldAvailable(string $fieldName, string $tableName, array $queryConfig): bool`
- يتحقق مما إذا كان الحقل موجوداً في الجدول الرئيسي أو في أي من الجداول المرتبطة عبر الـ joins المحددة.
- يستخدم `$registry->getTableColumns($tableName)` للحصول على أسماء الأعمدة.

#### 17. `isValidOperator(string $operator): bool`
- يتحقق مما إذا كان العامل ضمن القائمة المسموحة:
  `eq`, `=`, `neq`, `!=`, `gt`, `>`, `gte`, `>=`, `lt`, `<`, `lte`, `<=`, `like`, `not_like`, `in`, `not_in`, `between`, `not_between`, `is_null`, `is_not_null`, `contains`, `starts_with`, `ends_with`.

#### 18. `countConditions(array $conditions): int`
- يحسب عدد الشروط بشكل متكرر (بما في ذلك الشروط المتداخلة).

#### 19. `calculateComplexity(array $queryConfig): string`
- يعطي درجة تعقيد (low, medium, high) بناءً على:
  - عدد الأعمدة.
  - عدد الـ joins × 3.
  - عدد الشروط × 2.
  - عدد GROUP BY × 4.
  - عدد ORDER BY.
- **low**: < 10 نقاط، **medium**: < 25، **high**: ≥ 25.

#### 20. `performSecurityChecks(array $queryConfig)`
- يستدعي `checkForSqlInjection` للبحث عن أنماط خطيرة (مثل `union select`، `drop table`).
- يستدعي `checkSensitiveDataAccess` للبحث عن أعمدة حساسة (password, token, secret, key, ssn, credit_card).
- يستدعي `checkResourceUsage` لتقدير استهلاك الموارد.

#### 21. `performPerformanceChecks(array $queryConfig)`
- يستدعي `checkIndexUsage` للتأكد من وجود شروط على أعمدة مفهرسة (فحص بسيط لأعمدة `id`, `uuid`, `created_at`, `updated_at`).
- يستدعي `checkCartesianProducts` للتحذير من الـ joins المتعددة التي قد تنتج منتج كارتيزي.

#### 22. `generateValidationSummary(array $queryConfig): array`
- يُنشئ مصفوفة تحتوي على:
  - `complexity`: نتيجة `calculateComplexity`.
  - `total_columns`: عدد الأعمدة.
  - `total_joins`: عدد الـ joins.
  - `total_conditions`: عدد الشروط.
  - `has_grouping`: هل يوجد GROUP BY؟
  - `has_sorting`: هل يوجد ORDER BY؟
  - `has_limit`: هل يوجد LIMIT؟
  - `estimated_performance`: تقدير الأداء (`estimatePerformance`).

#### 23. `estimatePerformance(array $queryConfig): string`
- يُقدر الأداء (`fast`, `moderate`, `slow`) بناءً على التعقيد وعدد الـ joins والشروط.

---

### أمثلة عملية

#### مثال 1: تكوين تقرير صحيح وبسيط
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'uuid', 'alias' => 'order_id'],
        ['name' => 'amount', 'type' => 'decimal'],
    ],
    'conditions' => [
        [
            'field' => ['name' => 'status'],
            'operator' => ['value' => '='],
            'value' => 'completed'
        ]
    ],
    'limit' => 100
];

$validator = new ReportQueryValidator($registry);
$result = $validator->validate($queryConfig);

// $result['valid'] = true
// $result['errors'] = []
// $result['warnings'] = []
// $result['summary'] = ['complexity' => 'low', 'estimated_performance' => 'fast', ...]
```

#### مثال 2: عمود غير موجود في الجدول
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'non_existent_column'],
    ],
];

$result = $validator->validate($queryConfig);
// $result['valid'] = false
// $result['errors'] = ["Column 'non_existent_column' does not exist in table 'orders'"]
```

#### مثال 3: استخدام alias غير صالح
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'uuid', 'alias' => 'order id'], // مسافة غير مسموحة
    ],
];

$result = $validator->validate($queryConfig);
// $result['errors'] = ["Column 0: Invalid alias format 'order id' for column 'uuid'"]
```

#### مثال 4: شرط مع عامل غير صالح
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [['name' => 'uuid']],
    'conditions' => [
        [
            'field' => ['name' => 'amount'],
            'operator' => ['value' => 'invalid_operator'],
            'value' => 100
        ]
    ],
];

$result = $validator->validate($queryConfig);
// $result['errors'] = ["conditions[0]: Invalid operator 'invalid_operator'"]
```

#### مثال 5: شرط `in` بقيمة غير مصفوفة
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [['name' => 'uuid']],
    'conditions' => [
        [
            'field' => ['name' => 'status'],
            'operator' => ['value' => 'in'],
            'value' => 'completed,pending' // نص وليس مصفوفة
        ]
    ],
];

$result = $validator->validate($queryConfig);
// لا يوجد خطأ هنا لأن الدالة تقبل نصاً (يتم تحويله لاحقاً)، لكن قد يكون هناك تحذير إذا كانت القيمة غير مناسبة.
```

#### مثال 6: تكوين مع GROUP BY وأعمدة غير مجمعة
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'customer_name'], // هذا العمود غير موجود في groupBy
        ['name' => 'amount'],
    ],
    'groupBy' => [
        [
            'groupBy' => ['name' => 'customer_name'],
            'aggregateFn' => ['value' => 'sum'],
            'aggregateBy' => ['name' => 'amount']
        ]
    ],
];

$result = $validator->validate($queryConfig);
// في validateQueryConfig (التي تُستدعى لاحقاً في المحول) سيرمي خطأ، لكن في هذا الكلاس لا يوجد فحص لذلك.
// لكن validateGroupBy سيتحقق من وجود الحقول.
```

#### مثال 7: تجاوز حد LIMIT
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [['name' => 'uuid']],
    'limit' => 60000,
];

$result = $validator->validate($queryConfig);
// $result['errors'] = ["Limit cannot exceed 50,000 rows"]
// $result['warnings'] قد تحتوي على تحذير تجاوز 10000.
```

#### مثال 8: شروط متداخلة كثيرة (تحذير أداء)
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [['name' => 'uuid']],
    'conditions' => [
        ['field' => ['name' => 'status'], 'operator' => ['value' => '='], 'value' => 'a'],
        ['field' => ['name' => 'status'], 'operator' => ['value' => '='], 'value' => 'b'],
        // ... 25 شرطاً
    ],
];

$result = $validator->validate($queryConfig);
// $result['warnings'] = ["Many conditions (25) may impact query performance"]
```

#### مثال 9: وجود كلمة محظورة (SQL Injection)
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [['name' => 'uuid']],
    'conditions' => [
        [
            'field' => ['name' => 'note'],
            'operator' => ['value' => 'like'],
            'value' => "%'; DROP TABLE users; --" // محاولة حقن
        ]
    ],
];

$result = $validator->validate($queryConfig);
// $result['errors'] = ["Potential SQL injection detected: value"]
```

#### مثال 10: الوصول إلى عمود حساس (تحذير)
```php
$queryConfig = [
    'table' => ['name' => 'users'],
    'columns' => [
        ['name' => 'email'],
        ['name' => 'password_hash'], // عمود قد يكون حساساً
    ],
];

$result = $validator->validate($queryConfig);
// $result['warnings'] = ["Accessing potentially sensitive column: password_hash"]
```

---

### ملاحظات مهمة
- الكلاس **لا يقوم بتعديل التكوين**، بل فقط بإصدار أحكام عليه.
- بعض الفحوصات تعتمد على وجود `$registry` المسجل به جميع الجداول والأعمدة.
- فحص الأعمدة في الشروط والـ joins والـ group by يتم عبر `isFieldAvailable` التي تبحث في الجدول الرئيسي والجداول المرتبطة المحددة في `joins`. هذا يعني أن العمود من علاقة لم تتم إضافتها كـ join سيُعتبر غير متاح (لأنه لا يمكن الإشارة إليه دون join).
- فحص الأداء (`checkIndexUsage`) مبسط للغاية ويعتمد على أسماء أعمدة شائعة؛ يمكن تحسينه بالاستعلام عن فهارس قاعدة البيانات الفعلية.
- فحص SQL Injection (`checkForSqlInjection`) يبحث عن أنماط معروفة ولكنه ليس بديلاً عن إعدادات Prepared Statements التي توفرها Laravel نفسها.
- يمكن للمطورين توسيع الكلاس بإضافة فحوصات مخصصة تناسب تطبيقهم.

---

### الخلاصة
`ReportQueryValidator` هو خط الدفاع الأول قبل بناء وتنفيذ أي تقرير. من خلال التحقق من صحة التكوين، يضمن أن الاستعلامات التي سيتم إنشاؤها صحيحة نحويًا وآمنة، كما يقدم تحذيرات قيّمة للمستخدم حول الأداء المحتمل. هذا يساعد في بناء نظام تقارير قوي وموثوق.