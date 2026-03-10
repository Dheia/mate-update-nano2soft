## توثيق كلاس `ReportQueryConverter`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### مقدمة
كلاس `ReportQueryConverter` هو المكون الأساسي المسؤول عن تحويل تكوين التقرير (Query Configuration) إلى استعلام Laravel Query Builder قابل للتنفيذ، ثم تنفيذه وإرجاع النتائج مع بيانات وصفية. يستقبل هذا الكلاس `queryConfig` الذي يأتي عادةً من الواجهة الأمامية (بعد التحقق من صحته)، ويقوم ببناء استعلام SQL ديناميكي مع مراعاة العلاقات (joins)، الأعمدة المحسوبة، التجميعات، الشروط، والترتيب.

### الهدف الرئيسي
- بناء استعلام SQL آمن وفعال بناءً على تكوين المستخدم.
- تطبيق قيود الأمان مثل نطاق الشركة (company scope).
- معالجة auto-joins تلقائياً عند استخدام أعمدة من علاقات.
- دمج الأعمدة المحسوبة والتجميعات.
- إرجاع النتائج مع معلومات إضافية مثل وقت التنفيذ، الاستعلام المُنفذ، والأعمدة المحددة.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$registry` | `ReportSchemaRegistry` | مرجع إلى سجل المخطط للوصول إلى معلومات الجداول والعلاقات. |
| `$queryConfig` | `array` | تكوين التقرير كما هو وارد من الواجهة. |
| `$autoJoins` | `array` | يسجل معلومات الـ auto-joins التي تم تطبيقها (للتتبع). |
| `$manualJoins` | `array` | يسجل معلومات الـ joins اليدوية من التكوين. |
| `$joinAliases` | `array` | قاموس يربط مسار العلاقة (مثل `payload.pickup`) بالاسم المستعار للجدول (alias) المُستخدم في الاستعلام. |
| `$emittedAggregates` | `array` | يسجل معلومات الأعمدة التجميعية التي تم إضافتها (للإبلاغ في النتائج). |
| `$aliasCounter` | `int` | عداد لتوليد أسماء مستعارة فريدة (نادر الاستخدام). |

---

### التدفق العام للكلاس

1. **الإنشاء**: يتم إنشاء الكائن بتمرير `$registry` و `$queryConfig`. في البناء، يتم استدعاء `extractComputedColumnsFromAggregates()` لاستخراج الأعمدة المحسوبة من `groupBy` إذا كانت موجودة.
2. **التنفيذ**: دالة `execute()` هي المدخل الرئيسي. تقوم بالتحقق الأساسي، بناء الاستعلام، التنفيذ، معالجة النتائج، وإرجاع الرد.
3. **بناء الاستعلام**: دالة `buildQuery()` تقوم ببناء كائن `Builder` خطوة بخطوة:
   - تحديد الجدول الأساسي.
   - تطبيق نطاق الشركة (`applyCompanyScope`).
   - معالجة auto-joins (`processAutoJoins`).
   - معالجة manual joins (`processManualJoins`).
   - بناء SELECT (`buildSelectClause`).
   - بناء WHERE (`buildWhereClause`).
   - بناء GROUP BY (`buildGroupByClause`).
   - بناء ORDER BY (`buildOrderByClause`).
   - بناء LIMIT (`buildLimitClause`).
4. **معالجة النتائج**: دالة `processResults` (حالياً بسيطة) قد تستخدم لاحقاً لتحويل البيانات.
5. **إرجاع الرد**: يتضمن `success`, `data`, `columns`, `meta` (وقت التنفيذ، SQL، إلخ).

---

### شرح مفصل للدوال

#### 1. `__construct(ReportSchemaRegistry $registry, array $queryConfig)`
- يحفظ `$registry` و `$queryConfig`.
- يستدعي `extractComputedColumnsFromAggregates()`.

#### 2. `extractComputedColumnsFromAggregates()`
**الهدف**: استخراج الأعمدة المحسوبة من مجموعة `groupBy` إذا كانت موجودة، وإضافتها إلى `queryConfig['computed_columns']`. هذا يسمح بتعريف الأعمدة المحسوبة في نفس كائن التجميع دون الحاجة لتكرارها.

**المنطق**:
- إذا لم يوجد `groupBy`، يخرج.
- يتأكد من وجود `computed_columns` في `queryConfig` (يُضعه إن لم يكن).
- لكل مجموعة `groupBy`، يتحقق من `aggregateBy`، وإذا كان `computed` true، يأخذ `computation` و `name` ويضيفهما إلى `computed_columns` مع تجنب التكرار.

#### 3. `execute(): array`
**الخطوات**:
- تسجيل وقت البدء.
- التحقق من صحة التكوين الأساسي (وجود جدول وأعمدة) عبر `validateQueryConfig()`.
- بناء الاستعلام عبر `buildQuery()`.
- تنفيذ `$query->get()->toArray()`.
- معالجة النتائج عبر `processResults()`.
- حساب وقت التنفيذ.
- إرجاع مصفوفة تحتوي على `success`, `data`, `columns` (من `getSelectedColumns()`)، و `meta`.

في حالة الخطأ، يتم اصطياد الاستثناء وإرجاع `success: false` مع رسالة الخطأ.

#### 4. `buildQuery(): Builder`
**الخطوات**:
- الحصول على كائن `Table` من `$registry` باستخدام اسم الجدول الأساسي.
- بدء الاستعلام بـ `DB::table($tableName)`.
- تطبيق نطاق الشركة (`applyCompanyScope`).
- معالجة auto-joins (`processAutoJoins`).
- معالجة manual joins (`processManualJoins`).
- بناء SELECT (`buildSelectClause`).
- بناء WHERE (`buildWhereClause`).
- بناء GROUP BY (`buildGroupByClause`).
- بناء ORDER BY (`buildOrderByClause`).
- بناء LIMIT (`buildLimitClause`).

#### 5. `applyCompanyScope(Builder $query)`
- يضيف شرط `company_uuid` على الجدول الأساسي باستخدام معرف الشركة من الجلسة (من `resolveCompanyUuid()`).
- يرمي خطأ إذا لم يتم العثور على شركة.

#### 6. `resolveCompanyUuid(): ?string`
- يحاول استخراج معرف الشركة من `session('company')`، سواء كان نصاً أو كائنًا أو مصفوفة.
- يُعيد `null` إذا لم يُعثر.

#### 7. `processAutoJoins(Builder $query, string $tableName)`
**الهدف**: استخراج مسارات auto-join من الأعمدة المحددة والشروط والتجميعات والأعمدة المحسوبة، ثم تطبيقها.

**الخطوات**:
- جمع المسارات من:
  - `columns` (من `auto_join_path` أو من اسم العمود الذي يحتوي على نقاط).
  - `conditions` (باستخدام `collectAutoJoinPathsFromConditions`).
  - `groupBy` (باستخدام `collectAutoJoinPathsFromGroupBy`).
  - `sortBy` (باستخدام `collectAutoJoinPathsFromSortBy`).
  - `computed_columns` (باستخدام `extractRelationshipPathsFromExpression`).
- إزالة التكرار وترتيب المسارات حسب عدد النقاط (الأقصر أولاً) لضمان تنفيذ الـ joins بالتسلسل الصحيح.
- لكل مسار، استدعاء `applyAutoJoinPath`.

#### 8. `collectAutoJoinPathsFromConditions` (وما يشابهها)
- دوال مساعدة لاستخراج المسارات من مصفوفات الشروط والتجميعات والترتيب. تبحث عن `auto_join_path` أو اسم العمود الذي يحتوي على نقاط، وتستخرج المسار (كل الأجزاء ما عدا الأخير).

#### 9. `collectAutoJoinPathsFromComputedColumns`
- تستدعي `extractRelationshipPathsFromExpression` لاستخراج مسارات العلاقات من تعبير العمود المحسوب.

#### 10. `applyAutoJoinPath(Builder $query, string $rootTable, string $fullPath)`
- يتأكد من عدم تطبيق نفس المسار مسبقاً (باستخدام `$joinAliases`).
- يقسم المسار إلى أجزاء (مثل `payload.pickup`).
- يمر بكل جزء، وفي كل خطوة:
  - يحصل على كائن العلاقة من السياق الحالي (باستخدام `getRelationshipFromContext`).
  - إذا لم تكن العلاقة موجودة أو لم تكن auto-joinable، يتوقف.
  - يولد اسم مستعار للجدول (مثل `orders_payload_pickup`).
  - ينفذ `join` باستخدام النوع (`left`, `right`, `inner`) والمفاتيح المحلية والأجنبية.
  - يسجل الـ join في `$autoJoins` و `$joinAliases`.
  - يحدث السياق الحالي إلى الجدول المستعار والعلاقة التالية.

#### 11. `getRelationshipFromContext($ctx, string $name)`
- يحاول الحصول على كائن العلاقة بالاسم من `$ctx` (سواء كان `Table` أو `Relationship`) عن طريق دوال `getRelationship` أو `getAutoJoinRelationships`.

#### 12. `getChildContext($ctx, string $name)`
- يُعيد نفس ما تعيده `getRelationshipFromContext` (لأن السياق الجديد هو كائن العلاقة نفسها).

#### 13. `generateAliasChain(string $root, array $segments)`
- يولد اسم مستعار من الجذر والأجزاء، مثل: `orders_payload_pickup`.

#### 14. `resolveAliasAndColumn(string $rootTable, string $columnPath)`
- **الهدف**: تحويل مسار عمود (مثل `payload.pickup.address`) إلى اسم الجدول المستعار (alias) واسم العمود الفعلي.
- إذا كان المسار بدون نقاط، يُعيد `[$rootTable, $columnPath]`.
- إذا كان به نقاط، يفصل الجزء الأخير (العمود) والباقي (مسار العلاقة).
- يبحث في `$joinAliases` عن المسار الكامل؛ إذا وجد، يستخدم alias المطابق.
- إذا لم يجد، يمشي عبر الأجزاء خطوة بخطوة، ويستخدم آخر alias تم العثور عليه (أو يبقى على الجدول الأساسي إذا لم يُعثر).
- يُعيد `[$tableAlias, $column]`.

#### 15. `processManualJoins` و `applyManualJoin`
- تطبق الـ joins المحددة يدوياً في `queryConfig['joins']` (إذا وجدت). لكل join، يتم إضافة join من النوع المحدد مع المفاتيح، وتسجيله في `$manualJoins` و `$joinAliases`.

#### 16. `buildSelectClause(Builder $query)`
**الهدف**: بناء جزء SELECT من الاستعلام.

**المنطق**:
- يحصل على `$hasGrouping` (ما إذا كان هناك `groupBy`).
- إذا لم يكن هناك تجميع:
  - يبني مصفوفة `$selects` من الأعمدة العادية (باستخدام `resolveAliasAndColumn` لكل عمود) ويضيفها بصيغة `table_alias.column as alias`.
  - يستدعي `buildComputedColumns` لإضافة الأعمدة المحسوبة.
- إذا كان هناك تجميع:
  - يضيف أعمدة `groupBy` كأعمدة محددة (مع أسماء مستعارة).
  - لكل عنصر في `groupBy`، يتحقق من وجود `aggregateFn`، ويبني تعبير التجميع:
    - إذا كان الحقل التجميعي `'*'` (لـ COUNT)، يستخدم `COUNT(*)`.
    - إذا كان الحقل التجميعي عموداً محسوباً (موجود في `computed_columns`)، يستخدم التعبير المحسوب بعد حله (`resolveComputedColumnReferences`).
    - وإلا، يستخدم `table_alias.column` مع دالة التجميع (COUNT, SUM, AVG, MIN, MAX).
  - يسجل التجميعات في `$emittedAggregates` مع نوعها وتسميتها.
  - **ملاحظة**: في وضع التجميع، لا تتم إضافة الأعمدة المحسوبة كأعمدة عادية (لأنها ستخالف `ONLY_FULL_GROUP_BY`)، بل تستخدم فقط داخل دوال التجميع.

#### 17. `buildComputedColumns(Builder $query, array &$selects)`
- لكل عمود محسوب في `queryConfig['computed_columns']`:
  - يتحقق من صحة التعبير باستخدام `ComputedColumnValidator` (يتم استدعاؤها هنا).
  - يحل مراجع الأعمدة في التعبير باستخدام `resolveComputedColumnReferences`.
  - يضيف التعبير إلى `$selects` بصيغة `({$resolvedExpression}) as `{$name}``.

#### 18. `resolveComputedColumnReferences(string $expression, string $rootTable)`
**الهدف**: تحويل تعبير العمود المحسوب إلى تعبير صالح للاستعلام مع استبدال أسماء الأعمدة بمراجع `alias.column`، مع حماية السلاسل النصية وأسماء الدوال من الاستبدال الخاطئ.

**الخطوات**:
1. توسيع أي إشارات إلى أعمدة محسوبة أخرى بشكل متكرر (بحد أقصى 10 مرات) لتجنب التكرار اللانهائي.
2. حماية السلاسل النصية (بين علامات الاقتباس المفردة والمزدوجة) باستبدالها بمكان مؤقت.
3. حماية أسماء الدوال (كلمة متبوعة بـ `(`) باستبدالها بمكان مؤقت.
4. استبدال أسماء الأعمدة (كلمات) بمراجع `alias.column` باستخدام `resolveAliasAndColumn`.
5. استعادة الدوال والسلاسل النصية المحمية.

هذه عملية معقدة ولكنها تضمن عدم استبدال أجزاء من التعبير لا ينبغي استبدالها (مثل السلاسل النصية أو أسماء الدوال).

#### 19. `buildWhereClause` و `processConditions` و `applySingleCondition`
- بناء WHERE من خلال تمرير مصفوفة الشروط بشكل متكرر.
- لكل شرط فردي:
  - يحل اسم الحقل إلى `alias.column` باستخدام `resolveAliasAndColumn`.
  - يطبق الشرط بناءً على نوع العامل (`=`, `>`, `like`, `in`, `null`, `between`, إلخ).

#### 20. `buildGroupByClause`
- إذا كان هناك `groupBy`، يقوم بجمع الأعمدة المجموعة بعد تحويلها إلى `alias.column`، ويضيفها إلى `groupBy` في الاستعلام.

#### 21. `buildOrderByClause`
- إذا كان هناك `sortBy`:
  - في وضع التجميع، يسمح فقط بالترتيب باستخدام أسماء الأعمدة المستعارة (aliases) الموجودة في SELECT (مثل `count_uuid` أو `customer_name`) لمنع الأخطاء.
  - في الوضع العادي، يستخدم `resolveAliasAndColumn` ويضيف `orderBy` على `alias.column`.

#### 22. `buildLimitClause`
- إذا وجد `limit`، يضيف `limit` و `offset` (إن وجد).

#### 23. `getSelectedColumns(): array`
- يبني مصفوفة تحتوي على معلومات جميع الأعمدة التي سيتم إرجاعها في النتائج (الأعمدة العادية، الأعمدة المحسوبة، أعمدة التجميع). يُستخدم هذا لاحقاً في عرض النتائج في الواجهة الأمامية (تحديد التسميات والأنواع).

#### 24. `deriveAggregateAlias(array $g): string`
- يولد اسم مستعار لدالة التجميع بناءً على اسم الدالة والحقل (مثل `count_uuid` أو `sum_amount`).

#### 25. `validateQueryConfig()`
- التحقق من وجود اسم الجدول والأعمدة.
- التحقق من تسجيل الجدول في `$registry`.
- التحقق من صلاحية كل عمود (باستخدام `$registry->isColumnAllowed`).
- إذا كان هناك `groupBy`، التأكد من أن الأعمدة المحددة (غير التجميعية) موجودة في مجموعة `groupBy`، وإلا رمي خطأ.

#### 26. `extractRelationshipPathsFromExpression(string $expression, string $rootTable): array`
- يستخرج مسارات العلاقات (مثل `payload.pickup`) من تعبير العمود المحسوب عن طريق البحث عن أسماء أعمدة تحتوي على نقاط، ثم إزالة الجزء الأخير (العمود) للحصول على المسار.
- يُستخدم في `processAutoJoins` لتحديد الـ joins اللازمة للأعمدة المحسوبة.

#### 27. `createJoinsForComputedColumn` (قديم، غير مستخدم)
- دالة قديمة تم الاحتفاظ بها للتوافق، لكنها لا تفعل شيئاً (No-op) لأن الـ joins تُعالج الآن في `processAutoJoins`.

---

### أمثلة عملية شاملة

#### مثال 1: تقرير بسيط بدون تجميع
**تكوين التقرير:**
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'uuid', 'label' => 'رقم الطلب'],
        ['name' => 'created_at', 'label' => 'تاريخ الإنشاء'],
        ['name' => 'customer.name', 'label' => 'اسم العميل'],
    ],
    'conditions' => [
        [
            'field' => ['name' => 'status'],
            'operator' => ['value' => '='],
            'value' => 'completed'
        ]
    ],
    'sortBy' => [
        ['column' => ['name' => 'created_at'], 'direction' => ['value' => 'desc']]
    ],
    'limit' => 100
];
```

**الاستعلام الناتج:**
```sql
SELECT 
    orders.uuid as `uuid`,
    orders.created_at as `created_at`,
    customers.name as `customer_name`
FROM orders 
LEFT JOIN customers as orders_customer ON orders.customer_uuid = orders_customer.uuid
WHERE orders.status = 'completed' AND orders.company_uuid = 'abc-123'
ORDER BY orders.created_at DESC
LIMIT 100
```

**النتيجة:** مصفوفة من 100 طلب مع اسم العميل.

#### مثال 2: تقرير مع تجميع (Group By) وأعمدة محسوبة
**تكوين التقرير:**
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'customer.name', 'label' => 'العميل'],
    ],
    'computed_columns' => [
        ['name' => 'total_with_tax', 'expression' => 'ROUND(amount * 1.14, 2)', 'type' => 'decimal']
    ],
    'groupBy' => [
        [
            'groupBy' => ['name' => 'customer.name', 'label' => 'العميل'],
            'aggregateFn' => ['value' => 'count'],
            'aggregateBy' => ['name' => 'uuid']
        ],
        [
            'groupBy' => ['name' => 'customer.name'],
            'aggregateFn' => ['value' => 'sum'],
            'aggregateBy' => ['name' => 'total_with_tax']  // استخدام العمود المحسوب
        ]
    ],
    'sortBy' => [
        ['column' => ['alias' => 'count_uuid'], 'direction' => ['value' => 'desc']]
    ]
];
```

**الاستعلام الناتج:**
```sql
SELECT 
    customers.name as `customer_name`,
    COUNT(orders.uuid) as `count_uuid`,
    SUM(ROUND(orders.amount * 1.14, 2)) as `sum_total_with_tax`
FROM orders 
LEFT JOIN customers as orders_customer ON orders.customer_uuid = orders_customer.uuid
WHERE orders.company_uuid = 'abc-123'
GROUP BY customers.name
ORDER BY `count_uuid` DESC
```

**النتيجة:** قائمة بالعملاء مع عدد الطلبات لكل عميل ومجموع المبالغ بعد الضريبة.

#### مثال 3: تقرير مع علاقات متداخلة (Nested Relationships)
**تكوين التقرير:**
```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'uuid'],
        ['name' => 'payload.pickup.address', 'label' => 'عنوان الاستلام'],
        ['name' => 'payload.dropoff.city', 'label' => 'مدينة التسليم'],
    ],
    'conditions' => [
        [
            'field' => ['name' => 'payload.pickup.city'],
            'operator' => ['value' => '='],
            'value' => 'القاهرة'
        ]
    ]
];
```

**الاستعلام الناتج:**
```sql
SELECT 
    orders.uuid as `uuid`,
    orders_payload_pickup.address as `payload_pickup_address`,
    orders_payload_dropoff.city as `payload_dropoff_city`
FROM orders 
LEFT JOIN order_payloads as orders_payload ON orders.uuid = orders_payload.order_uuid
LEFT JOIN addresses as orders_payload_pickup ON orders_payload.uuid = orders_payload_pickup.payload_uuid AND orders_payload_pickup.type = 'pickup'
LEFT JOIN addresses as orders_payload_dropoff ON orders_payload.uuid = orders_payload_dropoff.payload_uuid AND orders_payload_dropoff.type = 'dropoff'
WHERE orders_payload_pickup.city = 'القاهرة' AND orders.company_uuid = 'abc-123'
```

**النتيجة:** طلبات حيث مدينة الاستلام هي القاهرة، مع عرض عنوان الاستلام ومدينة التسليم.

---

### التعامل مع الأخطاء
- `validateQueryConfig` ترمي `InvalidArgumentException` إذا كان التكوين غير صالح.
- أثناء بناء الاستعلام، إذا فشل أي جزء (مثل علاقة غير موجودة)، قد يتم تخطي الـ join أو رمي خطأ.
- في `execute`، يتم اصطياد أي `Exception` وإرجاع رد خطأ منظم مع وقت التنفيذ.

### ملاحظات مهمة
- يتم تطبيق نطاق الشركة تلقائياً على جميع الاستعلامات لضمان الأمان.
- الـ auto-joins تُطبق فقط للعلاقات التي تحمل خاصية `autoJoin = true`.
- يتم استخدام `resolveAliasAndColumn` للحصول على الاسم المستعار الصحيح للجدول، مما يضمن عدم تضارب الأسماء عند استخدام عدة joins.
- الأعمدة المحسوبة تخضع للتحقق عبر `ComputedColumnValidator` لمنع الحقن والأخطاء.

---

### خاتمة
كلاس `ReportQueryConverter` يقدم حلاً قوياً ومرناً لتحويل تكوينات التقارير الديناميكية إلى استعلامات SQL قابلة للتنفيذ. يدعم العلاقات المعقدة، الأعمدة المحسوبة، التجميعات، والعديد من عوامل التصفية، مع الحفاظ على الأمان والأداء. هذا الكلاس هو العمود الفقري لنظام التقارير في تطبيقات مثل Fleetbase، حيث يحتاج المستخدمون إلى بناء تقارير مخصصة دون الحاجة إلى معرفة تفاصيل SQL.