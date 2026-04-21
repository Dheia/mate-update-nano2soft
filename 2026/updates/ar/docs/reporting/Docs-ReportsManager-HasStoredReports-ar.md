## توثيق السمة `HasStoredReports` – نظام التقارير المسجلة مسبقاً

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

### مقدمة

**السمة `HasStoredReports`** هي إضافة متخصصة لكلاس `ReportsManager` (أو أي كلاس آخر) تتيح إمكانية **تسجيل وإدارة واستدعاء التقارير المسجلة مسبقاً** (قوالب التقارير). تسمح هذه السمة بتعريف تقارير ثابتة (مخزنة) تحتوي على تكوين استعلام أساسي، وفلاتر قابلة للتخصيص، وصلاحيات وصول، وخيارات تصدير. يمكن للمستخدمين (المصرح لهم) استعراض هذه التقارير وتشغيلها مع تمرير قيم الفلاتر ديناميكياً، مما يسهل بناء واجهات تقارير مخصصة دون الحاجة لإعادة تعريف الاستعلام في كل مرة.

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$storedReports` | `array` | مصفوفة تحتوي على جميع التقارير المسجلة، مفهرسة بواسطة `report_id`. |

### أهم الطرق (Methods)

#### 1. تسجيل التقارير

##### `public function registerReport(array $definition): void`
- **الهدف**: تسجيل تقرير واحد في النظام.
- **المعاملات**:
  - `$definition`: مصفوفة تحتوي على بيانات التقرير (انظر هيكل التقرير أدناه).
- **الاستثناءات**: ترمي `InvalidArgumentException` إذا كان `report_id` موجوداً مسبقاً أو كان التعريف غير صالح.
- **مثال**:
  ```php
  $manager->registerReport([
      'report_id' => 'inventory_summary',
      'name' => 'ملخص المخزون',
      'query_config' => [
          'table' => ['name' => 'tss_inventory_products'],
          'columns' => [['name' => 'name'], ['name' => 'quantity']],
      ],
  ]);
  ```

##### `public function registerReports(array $reports): void`
- **الهدف**: تسجيل مجموعة من التقارير دفعة واحدة.
- **المعاملات**: `$reports` مصفوفة من تعريفات التقارير (كل عنصر يمثل تقريراً).
- **مثال**:
  ```php
  $manager->registerReports([
      ['report_id' => 'rep1', 'name' => 'تقرير 1', 'query_config' => [...]],
      ['report_id' => 'rep2', 'name' => 'تقرير 2', 'query_config' => [...]],
  ]);
  ```

#### 2. استرجاع التقارير المسجلة

##### `public function getStoredReport(string $reportId): ?array`
- **الهدف**: الحصول على تعريف تقرير معين.
- **المعاملات**: `$reportId` معرف التقرير.
- **الإرجاع**: تعريف التقرير كمصفوفة، أو `null` إذا لم يوجد.

##### `public function getStoredReports(): array`
- **الهدف**: الحصول على جميع التقارير المسجلة.
- **الإرجاع**: مصفوفة مفهرسة بـ `report_id`.

##### `public function getAvailableStoredReports($user = null): array`
- **الهدف**: الحصول على التقارير المتاحة للمستخدم المحدد (أو المستخدم الحالي إذا لم يمرر) وفقاً للصلاحيات المسجلة.
- **المعاملات**: `$user` (اختياري) كائن المستخدم المراد التحقق من صلاحياته.
- **الإرجاع**: مصفوفة من تعريفات التقارير المبسطة (بدون تفاصيل داخلية) التي يسمح للمستخدم بالوصول إليها.
- **مثال**:
  ```php
  $availableReports = $manager->getAvailableStoredReports(BackendAuth::getUser());
  ```

#### 3. تشغيل تقرير مسجل

##### `public function runStoredReport(string $reportId, array $filterValues = [], array $options = []): array`
- **الهدف**: تنفيذ تقرير مسجل بعد دمج قيم الفلاتر.
- **المعاملات**:
  - `$reportId`: معرف التقرير.
  - `$filterValues`: مصفوفة من قيم الفلاتر (مفتاح = اسم الفلتر، قيمة = القيمة).
  - `$options`: خيارات إضافية تمرر إلى `runReport` (مثل `skip_validation`, `return_sql_only`).
- **الإرجاع**: نتيجة التقرير (نفس تنسيق `runReport`).
- **الاستثناءات**: ترمي `InvalidArgumentException` إذا كان التقرير غير موجود أو الفلاتر الإجبارية مفقودة.
- **مثال**:
  ```php
  $result = $manager->runStoredReport('inventory_summary', [
      'category' => 'electronics',
      'min_price' => 100
  ]);
  ```

### هيكل تعريف التقرير

يجب أن تحتوي مصفوفة تعريف التقرير على المفاتيح التالية:

| المفتاح | النوع | الوصف | إلزامي |
|--------|------|-------|--------|
| `report_id` | `string` | معرف فريد للتقرير | نعم |
| `name` | `string` | اسم التقرير (يظهر للمستخدم) | نعم |
| `query_config` | `array` | تكوين الاستعلام الأساسي (كما يمرر إلى `runReport`) | نعم |
| `description` | `string` | وصف التقرير (اختياري) | لا |
| `filters` | `array` | تعريف الفلاتر المتاحة (انظر أدناه) | لا |
| `permissions` | `array` | مصفوفة من صلاحيات الوصول للتقرير | لا |
| `export_options` | `array` | خيارات التصدير المسموحة (افتراضي: `['formats' => ['csv', 'excel', 'json'], 'max_rows' => 50000]`) | لا |
| `meta` | `array` | بيانات إضافية (اختياري) | لا |

#### هيكل الفلاتر (`filters`)

مصفوفة حيث المفتاح هو اسم الفلتر (يجب أن يطابق `variable` في شروط `query_config`)، والقيمة مصفوفة تحتوي على:

- `type`: نوع الفلتر (اختياري، للتحقق من النوع).
- `required`: `true` إذا كان الفلتر إجبارياً (افتراضي `false`).
- `options`: خيارات إضافية (مثل قيم منسدلة).

مثال:
```php
'filters' => [
    'branch_id' => ['type' => 'integer', 'required' => true],
    'min_price' => ['type' => 'decimal', 'required' => false],
]
```

### استخدام المتغيرات في `query_config`

لكي تتمكن السمة من استبدال قيم الفلاتر، يجب أن يحتوي `query_config` على شروط تحوي مفتاح `variable` بدلاً من `value`. مثال:

```php
'query_config' => [
    'table' => ['name' => 'tss_inventory_products'],
    'columns' => [['name' => 'name']],
    'conditions' => [
        ['field' => ['name' => 'branch_id'], 'operator' => ['value' => '='], 'variable' => 'branch_id'],
        ['field' => ['name' => 'price'], 'operator' => ['value' => '>'], 'variable' => 'min_price'],
    ],
]
```

عند تشغيل التقرير، يتم استبدال `variable` بالقيمة المقابلة من `$filterValues`.

### أمثلة عملية شاملة

#### مثال 1: تسجيل تقرير مع فلتر إجباري

```php
$manager->registerReport([
    'report_id' => 'products_by_branch',
    'name' => 'المنتجات حسب الفرع',
    'description' => 'يعرض المنتجات في فرع معين',
    'query_config' => [
        'table' => ['name' => 'tss_inventory_products'],
        'columns' => [
            ['name' => 'name', 'label' => 'الاسم'],
            ['name' => 'price', 'label' => 'السعر'],
        ],
        'conditions' => [
            ['field' => ['name' => 'branch_id'], 'operator' => ['value' => '='], 'variable' => 'branch_id'],
        ],
    ],
    'filters' => [
        'branch_id' => ['type' => 'integer', 'required' => true],
    ],
    'permissions' => ['inventory.reports.view'],
]);
```

#### مثال 2: الحصول على التقارير المتاحة للمستخدم الحالي

```php
public function listReports()
{
    $manager = ReportsManager::instance();
    $reports = $manager->getAvailableStoredReports();
    return response()->json($reports);
}
```

#### مثال 3: تشغيل تقرير مع تمرير الفلاتر

```php
public function runReport($reportId, Request $request)
{
    $manager = ReportsManager::instance();
    $result = $manager->runStoredReport($reportId, $request->input('filters', []));
    return response()->json($result);
}
```

#### مثال 4: تعريف تقرير في إضافة (Plugin)

```php
public function registerReports()
{
    return [
        'monthly_sales' => [
            'report_id' => 'monthly_sales',
            'name' => 'تقرير المبيعات الشهري',
            'query_config' => [
                'table' => ['name' => 'sales_orders'],
                'columns' => [
                    ['name' => 'created_at', 'label' => 'التاريخ'],
                    ['name' => 'customer.name', 'label' => 'العميل'],
                    ['name' => 'total_amount', 'label' => 'الإجمالي'],
                ],
                'conditions' => [
                    [
                        'field' => ['name' => 'created_at'],
                        'operator' => ['value' => '>='],
                        'variable' => 'start_date'
                    ],
                    [
                        'field' => ['name' => 'created_at'],
                        'operator' => ['value' => '<='],
                        'variable' => 'end_date'
                    ],
                ],
            ],
            'filters' => [
                'start_date' => ['type' => 'date', 'required' => true],
                'end_date' => ['type' => 'date', 'required' => true],
            ],
            'permissions' => ['sales.reports'],
        ],
    ];
}
```

### التفاعل مع الكلاسات الأخرى

- **`ReportsManager`**: السمة مصممة لتُستخدم داخل `ReportsManager`، وتعتمد على دواله مثل `runReport`, `handleReportError`, `getUserManager`.
- **`ReportUserManager`**: تستخدم السمة مدير المستخدم للتحقق من صلاحيات الوصول للتقارير (`permissions`).
- **`ReportQueryConverter`**: يتم تمرير `query_config` المدمج مع الفلاتر إلى `runReport` الذي يستخدم المحول لتنفيذ الاستعلام.

### الدوال المجردة (Abstract Methods)

لكي تعمل السمة بشكل صحيح، يجب أن يوفر الكلاس المستخدم (مثل `ReportsManager`) تنفيذ الدوال التالية:

- `abstract protected function getUserManager(): ReportUserManager;`
- `abstract protected function runReport(array $queryConfig, array $options = []): array;`
- `abstract protected function handleReportError(\Throwable $exception, array $context = []): array;`

### فوائد استخدام السمة

- **إعادة الاستخدام**: يمكن تعريف تقارير ثابتة مرة واحدة وتشغيلها بمرونة مع فلاتر مختلفة.
- **الأمان**: التحكم في الوصول عبر صلاحيات محددة لكل تقرير.
- **سهولة التكامل**: يمكن للإضافات تسجيل تقاريرها عبر دالة `registerReports` (مثل `registerReportTables`).
- **مرونة الفلاتر**: دعم المتغيرات في الشروط مع إمكانية تحديد الفلاتر الإجبارية والاختيارية.
- **توافق مع النظام الحالي**: لا تؤثر على وظائف `ReportsManager` الأخرى، وتعمل جنباً إلى جنب مع التقارير الديناميكية المباشرة.

### الاعتماديات والمتطلبات

- **PHP** (>= 8.0)
- **Laravel** (>= 8.x) أو **OctoberCMS** مع دعم `ReportsManager`.
- يتطلب وجود `ReportsManager` مع السمة `ReportsManagerQueryConverter` (التي توفر `getUserManager`, `runReport`, `handleReportError`).

### الخاتمة

تمثل السمة `HasStoredReports` إضافة قوية لنظام التقارير الديناميكي، حيث تسمح ببناء مكتبة من التقارير المسجلة مسبقاً مع تحكم كامل في الصلاحيات والفلاتر. بفضل تصميمها النمطي، يمكن للمطورين توفير تجربة تقارير متقدمة للمستخدمين دون تعقيد، مع الحفاظ على أمان وأداء النظام.