## توثيق كلاس `ComputedColumnValidator`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### مقدمة
كلاس `ComputedColumnValidator` هو المسؤول عن **التحقق من صحة وأمان تعبيرات الأعمدة المحسوبة (Computed Columns)** في نظام التقارير الديناميكي. يتم استخدامه للتأكد من أن أي تعبير SQL يقدمه المستخدم (أو يأتي من الواجهة الأمامية) لا يحتوي على كلمات محظورة، ويستخدم دوالاً مسموحة فقط، ويشير إلى أعمدة موجودة فعلياً في قاعدة البيانات (مع دعم العلاقات). هذا يمنع هجمات SQL Injection والأخطاء الناتجة عن مراجع غير صالحة.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$allowedFunctions` | `array` | قائمة بالدوال المسموح استخدامها في التعبير (مثل `CONCAT`, `ROUND`, `DATEDIFF`). |
| `$allowedOperators` | `array` | قائمة بالعوامل الرياضية والمنطقية المسموح بها (مثل `+`, `-`, `*`, `/`, `AND`, `OR`). |
| `$forbiddenKeywords` | `array` | قائمة بالكلمات المحظورة التي تشير إلى عمليات خطيرة (مثل `DROP`, `DELETE`, `UNION`). |
| `$registry` | `ReportSchemaRegistry` | مرجع إلى سجل المخطط للتحقق من وجود الأعمدة والعلاقات. |

---

### أهم الطرق

#### 1. `__construct(ReportSchemaRegistry $registry)`
- يقوم بحقن `ReportSchemaRegistry` الذي سيستخدم للتحقق من وجود الأعمدة والعلاقات.

#### 2. `validate(string $expression, string $tableName, array $computedColumns = []): array`
**الهدف**: تنفيذ جميع خطوات التحقق على تعبير العمود المحسوب وإرجاع النتيجة.

**المعاملات**:
- `$expression`: تعبير SQL المراد التحقق منه (مثل `"CONCAT(first_name, ' ', last_name)"`).
- `$tableName`: اسم الجدول الأساسي (الذي سيتم إجراء الاستعلام عليه).
- `$computedColumns`: مصفوفة من الأعمدة المحسوبة الأخرى (اختياري) للسماح بالإشارات المتبادلة بينها.

**الخطوات**:
1. التحقق من عدم وجود كلمات محظورة (`checkForbiddenKeywords`).
2. التحقق من أن جميع الدوال المستخدمة ضمن القائمة المسموحة (`validateFunctions`).
3. التحقق من أن العوامل المستخدمة آمنة (`validateOperators`).
4. التحقق من أن جميع مراجع الأعمدة موجودة في الجدول أو في الأعمدة المحسوبة الأخرى (`validateColumnReferences`).
5. تجميع الأخطاء وإرجاع مصفوفة تحتوي على `valid` (boolean) و `errors` (مصفوفة من الرسائل).

#### 3. `checkForbiddenKeywords(string $expression): array`
- يبحث في التعبير عن أي كلمة من `$forbiddenKeywords` (مع مراعاة حدود الكلمات باستخدام `\b`). إذا وجد، يُرجع خطأ.

#### 4. `validateFunctions(string $expression): array`
- يستخرج أسماء الدوال من التعبير (باستخدام regex يبحث عن نمط `WORD(`) ويتأكد من أن كل دالة موجودة في `$allowedFunctions`.

#### 5. `validateOperators(string $expression): array`
- يزيل السلاسل النصية من التعبير (لتجنب الاكتشافات الخاطئة) ثم يبحث عن عوامل خطيرة مثل `;`, `--`, `/*`. إذا وجد، يُرجع خطأ.

#### 6. `validateColumnReferences(string $expression, string $tableName, array $computedColumns = []): array`
**الهدف**: التأكد من أن كل مرجع لعمود (مثل `amount` أو `customer.name`) يشير إلى عمود موجود بالفعل.

**الخطوات**:
- يحصل على كائن `Table` من `$registry` باستخدام `$tableName`.
- يبني قائمة بأسماء الأعمدة المحسوبة الأخرى (للسماح بالإشارات المتبادلة).
- يزيل السلاسل النصية من التعبير.
- يستخرج جميع الكلمات التي قد تكون أسماء أعمدة (باستخدام `\b[a-z_][a-z0-9_]*\b`).
- لكل كلمة، يتحقق مما إذا كانت كلمة محجوزة (دالة، عامل، كلمة مفتاحية) أو رقماً، ثم يتجاهلها.
- إذا كانت الكلمة موجودة في قائمة `$computedColumns`، تعتبر صالحة (إشارة إلى عمود محسوب آخر).
- بخلاف ذلك، يستدعي `isValidColumnReference` للتحقق من وجود العمود في الجدول أو في علاقاته.

#### 7. `isValidColumnReference(string $columnRef, Table $table): bool`
- يحدد ما إذا كان مرجع العمود (مثل `amount` أو `payload.pickup.city`) صالحاً.
- إذا كان المرجع بدون نقطة، يتحقق من وجوده في الجدول باستخدام `columnExistsInTable`.
- إذا كان المرجع بنقطتين (مثل `customer.name`)، يحاول:
  - التحقق مما إذا كان الجزء الأول (`customer`) هو عمود JSON موجود.
  - أو البحث عن علاقة بالاسم `customer` في الجدول (باستخدام `getRelationships`). إذا وُجدت العلاقة، يعتبر المرجع صالحاً (يمكن تحسين التحقق ليشمل أعمدة الجدول المرتبط).
- إذا كان المرجع بأكثر من نقطتين (مثل `payload.pickup.city`)، يعتبر صالحاً افتراضياً (يمكن توسيع المنطق لاحقاً).

#### 8. `columnExistsInTable(string $columnName, Table $table): bool`
- يتحقق من وجود عمود بالاسم `$columnName` في الجدول (بالبحث في `getColumns()`). كما يدعم أعمدة JSON حيث قد يكون الاسم مثل `details.address` فيتحقق من البادئة `details`.

#### 9. `getAllowedFunctions()` و `getAllowedOperators()`
- دوال وصول لإرجاع القوائم الكاملة للدوال والعوامل المسموحة (قد تكون مفيدة للواجهة الأمامية).

---

### أمثلة عملية

#### مثال 1: تعبير بسيط وصحيح
```php
use Nano2\QueryBuilder\Classes\Reporting\ComputedColumnValidator;
use Nano2\QueryBuilder\Classes\Reporting\ReportSchemaRegistry;

// على افتراض أن $registry مسجل به جدول users بعمودي first_name و last_name
$validator = new ComputedColumnValidator($registry);

$expression = "CONCAT(first_name, ' ', last_name) AS full_name";
$result = $validator->validate($expression, 'users');

if ($result['valid']) {
    echo "التعبير صحيح!";
} else {
    echo "الأخطاء: " . implode(', ', $result['errors']);
}
// الناتج: "التعبير صحيح!"
```

#### مثال 2: استخدام دالة غير مسموحة
```php
$expression = "EXEC sp_help"; // دالة خطيرة
$result = $validator->validate($expression, 'users');
// $result['valid'] = false
// $result['errors'] = ["Function 'EXEC' is not allowed. Allowed functions: ..."]
```

#### مثال 3: كلمة محظورة (DROP)
```php
$expression = "SELECT * FROM users; DROP TABLE users; --";
$result = $validator->validate($expression, 'users');
// $result['errors'] = ["Expression contains forbidden SQL keyword: DROP"]
```

#### مثال 4: مرجع عمود غير موجود
```php
$expression = "first_name + last_name + non_existent_column";
$result = $validator->validate($expression, 'users');
// $result['errors'] = ["Column reference 'non_existent_column' does not exist in table 'users' or its relationships"]
```

#### مثال 5: تعبير مع علاقة (auto-join)
```php
// بافتراض أن جدول users لديه علاقة auto-join باسم 'profile' تحتوي على عمود 'age'
$expression = "profile.age * 2 AS double_age";
$result = $validator->validate($expression, 'users');
// صحيح لأن profile.age يشير إلى علاقة موجودة.
```

#### مثال 6: تعبير مع علاقة متداخلة
```php
// بافتراض أن جدول orders لديه علاقة 'payload'، والتي بدورها لديها علاقة 'pickup' بعمود 'city'
$expression = "UPPER(payload.pickup.city) AS pickup_city_upper";
$result = $validator->validate($expression, 'orders');
// صحيح (طالما أن المسار payload.pickup.city موجود في السجل).
```

#### مثال 7: الإشارة إلى عمود محسوب آخر
```php
$computedColumns = [
    ['name' => 'full_name', 'expression' => "CONCAT(first_name, ' ', last_name)"]
];
$expression = "LENGTH(full_name) AS name_length";
$result = $validator->validate($expression, 'users', $computedColumns);
// صحيح لأن full_name موجود في قائمة الأعمدة المحسوبة.
```

#### مثال 8: عامل خطير (تعليق SQL)
```php
$expression = "amount * 10 -- هذا تعليق";
$result = $validator->validate($expression, 'orders');
// $result['errors'] = ["Operator '--' is not allowed"]
```

---

### قائمة الدوال المسموحة (`$allowedFunctions`)
تتضمن الدوال الشائعة والآمنة مثل:
- **الدوال التاريخية**: `DATEDIFF`, `DATE_ADD`, `NOW`, `YEAR`, `MONTH`, `DAY`, `DATE_FORMAT`, إلخ.
- **الدوال النصية**: `CONCAT`, `SUBSTRING`, `REPLACE`, `UPPER`, `LOWER`, `TRIM`, `LENGTH`.
- **الدوال الرقمية**: `ROUND`, `ABS`, `CEIL`, `FLOOR`, `MOD`, `POWER`, `SQRT`.
- **الدوال الشرطية**: `CASE`, `IF`, `IFNULL`, `COALESCE`.
- **دوال التجميع**: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` (يُسمح بها فقط في سياق التجميع، لكن الكلاس لا يمنعها).
- **دوال التحويل**: `CAST`, `CONVERT`.

### قائمة العوامل المسموحة (`$allowedOperators`)
- الرياضية: `+`, `-`, `*`, `/`, `%`
- المقارنة: `=`, `!=`, `<>`, `<`, `>`, `<=`, `>=`
- المنطقية: `AND`, `OR`, `NOT`
- أخرى: `IS`, `NULL`

### قائمة الكلمات المحظورة (`$forbiddenKeywords`)
تتضمن أي كلمة قد تؤدي إلى تغيير البيانات أو هيكل قاعدة البيانات: `DROP`, `DELETE`, `UPDATE`, `INSERT`, `TRUNCATE`, `ALTER`, `CREATE`, `GRANT`, `REVOKE`, `EXEC`, `UNION`, `INTO`, `INFORMATION_SCHEMA`, `LOAD_FILE`, `OUTFILE`, `DUMPFILE`, `BENCHMARK`, `SLEEP`.

---

### ملاحظات مهمة
- الكلاس **لا ينفذ التعبير**، بل يتحقق فقط من سلامته النحوية والأمنية.
- التحقق من وجود الأعمدة في العلاقات يتم بطريقة مبسطة (يكفي وجود اسم العلاقة)، ويمكن توسيعه للتحقق من العمود النهائي أيضاً.
- الأعمدة المحسوبة يمكن أن تشير إلى بعضها البعض، ولكن يجب تمريرها في `$computedColumns` لتجنب الأخطاء.
- يتم إزالة السلاسل النصية قبل التحقق من العوامل والكلمات المحظورة لمنع الاكتشافات الخاطئة (مثل وجود `--` داخل نص عادي).

---

### الخلاصة
`ComputedColumnValidator` هو أداة أساسية لضمان أمان وصحة الأعمدة المحسوبة في نظام التقارير. بفضل قوائم الدوال المسموحة والكلمات المحظورة، والتحقق من وجود الأعمدة عبر `ReportSchemaRegistry`، يمكن للمطورين السماح للمستخدمين بكتابة تعبيرات SQL مخصصة دون المخاطرة بأمان التطبيق أو سلامة البيانات.