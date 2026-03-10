## توثيق عام: نظام التقارير الديناميكي `Nano2.QueryBuilder.Reporting`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### 1. مقدمة

**وحدة `Nano2\QueryBuilder\Classes\Reporting`** هي جزء من حزمة `Nano2.QueryBuilder`، وهي إضافة متخصصة مقدمة من **شركة نانوسوفت (NanoSoft)** لتطبيقات NanoSoft App و Laravel. تهدف هذه الوحدة إلى توفير **نظام متكامل لبناء وتنفيذ وإدارة التقارير الديناميكية** بشكل آمن ومرن وقابل للتوسع.

#### لماذا نحتاج إلى نظام تقارير ديناميكي؟
في التطبيقات الإدارية المعقدة (مثل أنظمة إدارة الأساطيل، أو إدارة الموارد، أو أنظمة تخطيط الموارد)، يحتاج المستخدمون النهائيون (غير المبرمجين) إلى القدرة على:
- استعراض بيانات مخصصة من جداول متعددة.
- تطبيق فلاتر وشروط معقدة.
- تجميع البيانات وحساب الإحصائيات (المجاميع، المتوسطات، إلخ).
- تصدير النتائج بصيغ متعددة (Excel, PDF, CSV...).

بدلاً من كتابة استعلامات SQL يدويًا أو طلب تقارير مخصصة من فريق التطوير في كل مرة، يوفر هذا النظام **طبقة تعريفية (Schema)** تسمح للواجهة الأمامية ببناء تكوين (Configuration) يمثل التقرير المطلوب، ثم يتم تحويل هذا التكوين إلى استعلام SQL آمن وتنفيذه، وإرجاع النتائج بشكل منظم. يتولى `ReportsManager` عملية جمع تعريفات الجداول من الإضافات وإدارة دورة حياتها، مما يبسط عملية التكامل.

---

### 2. المكونات الرئيسية للنظام

يتكون النظام من مجموعة من الكلاسات المتخصصة، يمكن تقسيمها إلى عدة طبقات:

#### أ. طبقة تعريف المخطط (Schema Definition Layer)
هذه الطبقة مسؤولة عن وصف هيكل البيانات المتاح للتقارير (الجداول، الأعمدة، العلاقات) بطريقة برمجية.

- **`Column`**: يمثل عمودًا في جدول (أو عمودًا محسوبًا). يحدد اسمه، نوع بياناته، تسميته للعرض، وإمكانياته (بحث، ترتيب، تصفية، تجميع). يمكن أن يكون عاديًا أو محسوبًا (Computed) بتعبير SQL.
- **`Relationship`**: يمثل علاقة بين جدولين (مثل `belongsTo`، `hasMany`). يحدد اسم العلاقة، الجدول المرتبط، مفاتيح الربط، ونوع الـ JOIN. يدعم خاصية `autoJoin` التي تسمح بإضافة الجوين تلقائيًا عند اختيار أعمدة من العلاقة. كما يدعم العلاقات المتداخلة (nested relationships) مثل `payload.pickup`.
- **`Table`**: يمثل جدولاً رئيسياً. يجمع الأعمدة العادية والمحسوبة والعلاقات. يوفر دوال للوصول إلى الأعمدة القابلة للعرض والتحقق من صلاحيتها.
- **`ReportSchemaRegistry`**: سجل مركزي يُسجل فيه جميع كائنات `Table`. يوفر واجهة للاستعلام عن الجداول المسجلة، وأعمدةها، وعلاقاتها، مع دعم التخزين المؤقت (Cache) لتحسين الأداء. يلعب دور "الذاكرة الدائمة" للنظام.

#### ب. طبقة إدارة المخطط (Schema Management Layer)
هذه الطبقة مسؤولة عن جمع تعريفات الجداول من مصادر متعددة (الإضافات، التسجيل المباشر، الـ callbacks) وتحويلها إلى الكائنات المناسبة وتسليمها إلى `ReportSchemaRegistry`. توفر واجهة موحدة لتسجيل الجداول وإدارتها، بالإضافة إلى وظائف متقدمة لتشغيل التقارير.

- **`ReportsManager`**: الكلاس الأساسي في هذه الطبقة. يقوم بتجميع تعريفات الجداول الخام (arrays) من الإضافات (عبر دالة `registerReportTables`)، وتخزينها، ثم تحويلها إلى كائنات `Table` عند الحاجة وتسجيلها في `ReportSchemaRegistry`. يدعم التحميل البطيء، والتخزين المؤقت للكائنات الجاهزة، وفلترة الجداول حسب الصلاحيات.
- **`ReportsManagerQueryConverter` (trait)**: سمة (Trait) تُضاف إلى `ReportsManager` لتزويده بوظائف متكاملة لتشغيل التقارير:
  - التحقق من صحة تكوين التقرير (`validateReportQuery`).
  - تنفيذ الاستعلام (`executeReportQuery`).
  - معالجة الأخطاء (`handleReportError`).
  - تصدير النتائج (`runReportAndExport`).
  - تخزين النتائج مؤقتًا (`runReportWithCache`).
  - تقدير تكلفة الاستعلام (`estimateQueryCost`).
  - خيارات متقدمة (إرجاع SQL فقط، dry run).

#### ج. طبقة التحقق (Validation Layer)
قبل بناء أي استعلام، يجب التأكد من أن تكوين التقرير المقدم من المستخدم صحيح وآمن.

- **`ComputedColumnValidator`**: يتحقق من صحة تعبيرات الأعمدة المحسوبة. يتأكد من:
  - عدم وجود كلمات محظورة (SQL Injection).
  - أن جميع الدوال المستخدمة ضمن قائمة مسموحة.
  - أن مراجع الأعمدة موجودة في الجدول أو في علاقاته أو في أعمدة محسوبة أخرى.
- **`ReportQueryValidator`**: يتحقق من تكوين التقرير بالكامل. يشمل:
  - البنية الأساسية (وجود جدول وأعمدة).
  - وجود الجدول والأعمدة المحددة في `ReportSchemaRegistry`.
  - صحة الشروط (العوامل، القيم).
  - صحة التجميعات (`groupBy`) والترتيب (`sortBy`).
  - الحدود المسموحة (`limit`).
  - تحذيرات الأداء (تعقيد الاستعلام، عدد الـ joins، الشروط الكثيرة).
  - فحوصات أمان إضافية.

#### د. طبقة تنفيذ الاستعلام (Query Execution Layer)
هذه الطبقة تحول التكوين الصحيح إلى استعلام SQL حقيقي.

- **`ReportQueryConverter`**: الكلاس الأساسي الذي يقوم ببناء استعلام Laravel Query Builder وتنفيذه. يقوم بـ:
  - تطبيق نطاق الشركة (Company Scope) تلقائيًا.
  - تحليل المسارات (مثل `payload.pickup.address`) وتطبيق الـ auto-joins اللازمة باستخدام معلومات العلاقات من `Table` و `Relationship`.
  - بناء جملة SELECT مع الأعمدة العادية والمحسوبة والتجميعية.
  - بناء WHERE من الشروط المتداخلة.
  - بناء GROUP BY و ORDER BY و LIMIT.
  - تنفيذ الاستعلام وإرجاع النتائج مع بيانات وصفية (وقت التنفيذ، الاستعلام المنفذ، إلخ).

#### هـ. طبقة التصدير (Export Layer)
بعد الحصول على النتائج، يمكن تصديرها بصيغ متعددة.

- **`ReportQueryExporter`**: يقوم بتصدير البيانات (`$data`) إلى ملفات بصيغ مختلفة:
  - **CSV**: مع خيارات (فاصل، ترميز).
  - **Excel (XLSX)**: باستخدام PhpSpreadsheet مع تنسيق متقدم (ألوان، حدود، تنسيق أرقام/تواريخ، ورقة Metadata).
  - **JSON**: مع أو بدون Pretty Print.
  - **PDF**: (تطوير مستقبلي) حاليًا ينشئ HTML.
  - **XML**: هيكل منظم.

#### و. طبقة معالجة الأخطاء (Error Handling Layer)
- **`ReportQueryErrorHandler`**: يوفر واجهة موحدة لمعالجة الأخطاء والاستثناءات. يقوم بـ:
  - تحديد رمز الخطأ المناسب (مثل `TABLE_NOT_FOUND`، `TIMEOUT`).
  - إنشاء معرف فريد للخطأ لتتبعه.
  - إرجاع رسائل خطأ سهلة للمستخدم مع اقتراحات.
  - تسجيل كافة التفاصيل التقنية في سجلات (Logs) للتشخيص.

---

### 3. كيف تعمل هذه الكلاسات معًا؟ (التدفق العام)

لنفترض أن مستخدمًا في الواجهة الأمامية يريد إنشاء تقرير عن الطلبات مع اسم العميل ومدينة الاستلام.

1. **تسجيل المخطط (مرة واحدة)**:
   - في ملف `Plugin.php` لأي إضافة، يتم تعريف دالة `registerReportTables` التي تُعيد مصفوفة من تعريفات الجداول.
   - يقوم `ReportsManager` (وهو كلاس Singleton) تلقائيًا باكتشاف هذه الإضافات عبر `PluginManager`، وجمع التعريفات، وتخزينها بشكل خام (`$rawDefinitions`). كما يمكن التسجيل يدويًا عبر `registerDefinition` أو `registerCallback`.
   - عند أول استدعاء لـ `getRegistry()`، يتم تهيئة `ReportSchemaRegistry`، وبناء كائنات `Table` من التعريفات الخام، وتسجيلها في الـ registry. يتم أيضًا تخزين هذه الكائنات في `$builtTables` للتسريع.

2. **طلب الواجهة الأمامية للمخطط**:
   - تستدعي الواجهة API مثل `/api/reports/schema/orders`.
   - يُستخدم `ReportsManager::getTableColumns('orders')` الذي يمرر الطلب إلى `ReportSchemaRegistry::getTableColumns()`. يمكن فلترة الأعمدة حسب صلاحيات المستخدم لاحقًا.

3. **بناء تكوين التقرير**:
   - يحدد المستخدم الأعمدة، الشروط، التجميعات، الترتيب، الحد.
   - ترسل الواجهة كائن JSON يمثل `queryConfig` (مطابق لهيكل متوقع).

4. **التحقق والتنفيذ عبر `ReportsManager`**:
   - بدلاً من إنشاء `ReportQueryValidator` و `ReportQueryConverter` يدويًا، يتم استخدام دوال `ReportsManager` المضافة بواسطة الترايت:
     ```php
     $manager = ReportsManager::instance();
     $result = $manager->runReport($queryConfig);
     ```
   - دالة `runReport` تقوم بالتحقق (`validateReportQuery`) ثم التنفيذ (`executeReportQuery`) مع معالجة الأخطاء تلقائيًا. كما تدعم خيارات مثل `skip_validation`, `return_sql_only`, `dry_run`.

5. **التصدير (اختياري)**:
   - يمكن استخدام `runReportAndExport` لتنفيذ التقرير وتصديره مباشرة.

6. **معالجة الأخطاء**:
   - أي استثناء يتم اصطياده ومعالجته عبر `handleReportError` الذي يستخدم `ReportQueryErrorHandler` داخليًا.

---

### 4. حالات الاستخدام النموذجية

#### أ. تقرير بسيط بدون تجميع
- **السيناريو**: مدير المبيعات يريد عرض جميع الطلبات المكتملة مع تاريخ الإنشاء واسم العميل.
- **التكوين**: جدول `orders`، أعمدة `created_at` و `customer.name`، شرط `status = 'completed'`، ترتيب حسب التاريخ تنازليًا.
- **الكلاسات المعنية**: `ReportsManager` (للتسجيل والتنفيذ)، `ReportQueryValidator`، `ReportQueryConverter`، `ReportSchemaRegistry`.

#### ب. تقرير مع تجميع وأعمدة محسوبة
- **السيناريو**: مدير المخزون يريد معرفة إجمالي المبيعات لكل مدينة مع حساب الضريبة.
- **التكوين**:
  - `groupBy` على `payload.pickup.city`.
  - `aggregateFn` = `sum` على عمود `amount` (للمبلغ).
  - `aggregateFn` = `avg` على عمود `amount` (لمتوسط المبلغ).
  - عمود محسوب: `total_with_tax` = `ROUND(amount * 1.14, 2)`.
- **الكلاسات المعنية**: `ComputedColumnValidator`، `ReportQueryConverter`، `ReportsManager`.

#### ج. تقرير مع علاقات متداخلة عميقة
- **السيناريو**: تقرير عن الشحنات يتضمن عنوان الاستلام (من `payload.pickup.address`) ومدينة التسليم (من `payload.dropoff.city`).
- **التكوين**: أعمدة `payload.pickup.address`، `payload.dropoff.city`.
- **الكلاسات المعنية**: `ReportSchemaRegistry::isColumnAllowed`، `ReportQueryConverter::resolveAliasAndColumn`، `ReportsManager`.

#### د. تصدير تقرير
- **السيناريو**: بعد عرض التقرير، يريد المستخدم تحميله كملف Excel.
- **الإجراء**: استخدام `ReportsManager::runReportAndExport($queryConfig, 'excel')`.
- **النتيجة**: ملف Excel منسق.

#### هـ. تقدير التكلفة قبل التنفيذ
- **السيناريو**: قبل تشغيل تقرير معقد، يريد النظام تحذير المستخدم إذا كان الاستعلام ثقيلاً.
- **الإجراء**: `ReportsManager::estimateQueryCost($queryConfig)` تُرجع درجة التعقيد والأداء المتوقع.

---

### 5. الفوائد والمزايا

- **الأمان**:
  - نطاق الشركة (`company_uuid`) يُطبق تلقائيًا على كل الاستعلامات.
  - `ComputedColumnValidator` يمنع استخدام كلمات SQL خطيرة.
  - التحقق من وجود الأعمدة في المخطط قبل بناء الاستعلام يمنع الأخطاء.
  - استخدام Prepared Statements عبر Laravel Query Builder.
  - `ReportsManager` يتحقق من صلاحيات المستخدم قبل عرض الجداول أو تنفيذ التقارير.

- **المرونة**:
  - يمكن تعريف أي جدول مع أي عدد من العلاقات المتداخلة.
  - دعم الأعمدة المحسوبة بتعبيرات SQL محدودة وآمنة.
  - التسجيل عبر `registerReportTables` في الإضافات، أو عبر `registerDefinition`، أو `registerCallback`.
  - إمكانية تمرير خيارات متقدمة لـ `runReport` (تخطي التحقق، إرجاع SQL فقط، إلخ).

- **الأداء**:
  - استخدام `auto-join` فقط عند الحاجة (تجنب JOINs غير الضرورية).
  - التخزين المؤقت للمخطط في `ReportSchemaRegistry` يقلل من استعلامات قاعدة البيانات المتكررة.
  - إمكانية تحديد `maxRows` وحدود الصفحات.
  - دعم التخزين المؤقت لنتائج التقارير عبر `runReportWithCache`.

- **قابلية التوسع**:
  - فصل واضح بين المسؤوليات (Schema, Management, Validation, Conversion, Export, Error Handling).
  - يمكن استبدال أو توسيع أي جزء (مثل إضافة مصدر بيانات جديد، أو صيغة تصدير جديدة) بسهولة.
  - `ReportsManager` يجمع كل التعريفات في مكان واحد، مما يسهل تتبعها وتحديثها.

- **تجربة المستخدم**:
  - رسائل خطأ واضحة مع اقتراحات.
  - دعم تصدير متعدد الصيغ.
  - إمكانية بناء تقارير معقدة دون الحاجة لمعرفة SQL.
  - واجهة برمجية بسيطة عبر `ReportsManager::runReport()`.

---

### 6. مثال عملي سريع (كود متكامل)

باستخدام `ReportsManager`، يصبح الكود في الـ Controller أبسط وأكثر وضوحًا:

```php
<?php namespace Nano\Reports\Http\Controllers;

use Illuminate\Routing\Controller;
use Illuminate\Http\Request;
use Nano2\QueryBuilder\Classes\Reporting\ReportsManager;

class ReportController extends Controller
{
    public function generate(Request $request)
    {
        $queryConfig = $request->input('query');
        $manager = ReportsManager::instance();

        // تشغيل التقرير مع خيارات (مثل تجاوز التحقق)
        $result = $manager->runReport($queryConfig, [
            'skip_validation' => false
        ]);

        if (!$result['success']) {
            return response()->json($result, 400);
        }

        // إذا طلب التصدير
        if ($request->input('export')) {
            $export = $manager->runReportAndExport(
                $queryConfig, 
                $request->input('format', 'csv')
            );
            return response()->json($export);
        }

        return response()->json($result);
    }

    public function estimate(Request $request)
    {
        $queryConfig = $request->input('query');
        $cost = ReportsManager::instance()->estimateQueryCost($queryConfig);
        return response()->json($cost);
    }

    public function schema($tableName)
    {
        $manager = ReportsManager::instance();
        $columns = $manager->getTableColumns($tableName);
        return response()->json($columns);
    }
}
```

**مثال لتسجيل الجداول من إضافة** (في `Plugin.php`):

```php
public function registerReportTables()
{
    return [
        'orders' => [
            'label' => 'الطلبات',
            'category' => 'sales',
            'extension' => 'orders',
            'columns' => [
                'uuid' => ['label' => 'المعرف', 'hidden' => true],
                'created_at' => ['type' => 'datetime', 'label' => 'تاريخ الإنشاء'],
                'amount' => ['type' => 'decimal', 'label' => 'المبلغ', 'aggregatable' => true],
            ],
            'relationships' => [
                'customer' => [
                    'table' => 'customers',
                    'label' => 'العميل',
                    'auto_join' => true,
                    'columns' => [
                        'name' => ['label' => 'الاسم'],
                        'email' => ['label' => 'البريد'],
                    ]
                ]
            ],
            'permissions' => ['nano.orders.access_reports'],
        ]
    ];
}
```

---

### 7. الاعتماديات والمتطلبات

- **Laravel** (>= 8.x) مع دعم Query Builder.
- **PHP** (>= 8.0).
- **OctoberCMS** (اختياري، لكن النظام مصمم للعمل معه).
- **Composer** لتثبيت الحزمة.
- **PhpOffice/PhpSpreadsheet** (لتصدير Excel) – يمكن إضافته اختياريًا.
- **Redis/Memcached** (اختياري) للتخزين المؤقت في `ReportSchemaRegistry` و `ReportsManager`.

---

### 8. الخاتمة

تمثل كلاسات `Nano2\QueryBuilder\Classes\Reporting` حلاً متكاملاً واحترافياً لبناء نظام تقارير ديناميكي في تطبيقات NanoSoft App و Laravel. بفضل التصميم الطبقي (Schema, Management, Validation, Conversion, Export, Error Handling)، يمكن للمطورين بناء تقارير معقدة بسرعة وأمان. مع إضافة `ReportsManager`، أصبح النظام أكثر تنظيماً وسهولة في الاستخدام، حيث يوفر نقطة دخول واحدة لإدارة تعريفات الجداول وتنفيذ التقارير، مع دعم خيارات متقدمة مثل التخزين المؤقت، التصدير، وتقدير التكلفة. هذا التصميم يضمن قابلية التوسع والصيانة، ويلبي احتياجات المستخدمين المتقدمة دون المساس بأداء النظام أو سلامته.