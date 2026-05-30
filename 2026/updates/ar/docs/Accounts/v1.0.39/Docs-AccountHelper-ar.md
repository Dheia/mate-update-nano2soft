# توثيق كلاس `AccountHelper` – مساعد الحسابات

## نظرة عامة

يقع كلاس `AccountHelper` في المسار `Tss\Accounts\Helpers\AccountHelper`، وهو أحد المكونات الأساسية في إضافة `Tss.Accounts` ضمن برمجيات نانوسوفت (NanoSoft App). يعمل الكلاس كطبقة مساعدة شاملة للتعامل مع الحسابات المالية والأرصدة والحركات والتفاصيل والمراجع والمستندات المحاسبية، ويقدم مجموعة متكاملة من الدوال الثابتة (static) التي تُسهل على المطورين تنفيذ العمليات المحاسبية اليومية دون الحاجة إلى كتابة استعلامات معقدة.

تشمل مسؤوليات الكلاس ما يلي:

- **جلب الأرصدة**: حساب رصيد حساب معين أو شخص معين (مدين، دائن، صافي) حسب شروط مرنة (قسم، فترة، عملة، إلخ).
- **جلب تفاصيل الحركات**: استعلام متقدم عن سجلات `TransactionsDetail` مع دعم تصفية حسب الحساب، الشخص، التاريخ، نوع الحركة، وغيرها.
- **إنشاء كائنات الرأس والتفصيل**: دوال مساعدة لتجهيز كائنات `TransactionHeader` و `TransactionsDetail` بأنواعها المختلفة (BondsDay، CatchReceipt، إلخ) بناءً على نوع المستند.
- **إدارة المراجع والحسابات الأم**: البحث عن مرجع (`Reference`) أو حساب أم (`Parent Account`) لنموذج معين.
- **إنشاء حسابات تلقائية**: إنشاء حساب فرعي جديد لنموذج (مثل موظف، طالب) بشكل تلقائي بناءً على إعدادات المرجع والحساب الأم.
- **تطبيع تفاصيل الحركات**: دمج وتجميع تفاصيل الحركات حسب `transactions_id` لتسهيل العرض.
- **معالجة تواريخ الاستعلام**: دالة مرنة لتطبيق شروط التاريخ (`date_at`، `created_at`) على استعلامات قاعدة البيانات.
- **تصفية حسب نوع المستخدم**: دوال لتصفية السجلات بناءً على `ref_type` الخاص بالمستخدمين (مثل سائق، عميل).

يتميز الكلاس بأن جميع دواله ثابتة (`static`) مما يسمح باستدعائها مباشرة من أي مكان في التطبيق دون الحاجة إلى إنشاء كائن. كما أنه يعتمد بشكل كبير على الإعدادات المخزنة في ملفات `Config` (مثل `tss.accounts::`) لتوفير سلوك افتراضي مرن يمكن تخصيصه لكل مشروع.

---

## هيكل الاستجابة للدوال التي تعيد كائنات

بعض الدوال مثل `getRefAccount` و `getParentAccount` و `createAccountToModel` تعيد مصفوفة بنية موحدة تحتوي على حالة العملية والرسالة والكائن (إن وجد). هذه البنية تسهل التحقق من نجاح العملية والتعامل مع الأخطاء.

| المفتاح | النوع | الوصف |
| :--- | :--- | :--- |
| `status` | `bool` | حالة العملية (`true` للنجاح، `false` للفشل). |
| `message` | `string` | رسالة وصفية (بالإنجليزية). |
| `model` | `object\|null` | الكائن الذي تم العثور عليه أو إنشاؤه (إن وجد). |

الدوال التي تعيد هذا الهيكل تدعم أيضاً معامل `$is_model` الذي إذا كان `true`، يتم إرجاع الكائن مباشرة بدلاً من المصفوفة.

---

## الدوال العامة (Public API)

### 1. `getBalances`

حساب رصيد حساب معين أو شخص معين (أو كليهما) بناءً على مجموعة من الشروط. تعتبر هذه الدالة العمود الفقري لجميع عمليات الاستعلام عن الأرصدة في النظام.

```php
public static function getBalances($code, $options = [])
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$code` | `string\|array\|object\|null` | كود الحساب (أو مصفوفة من الأكواد، أو كائن الشخص). إذا كان `null`، يمكن الاستعلام عن طريق الشخص فقط. |
| `$options` | `array` | مصفوفة خيارات الفلترة (انظر الجدول أدناه). |

#### خيارات `$options` المدعومة

| المفتاح | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `periods_id` | `string` | `*` (الفترة الأساسية تُحسب تلقائياً) | معرف الفترة المحاسبية. |
| `departments_id` | `string` | `*` | معرف القسم / الفرع. |
| `cost_centers_id` | `string` | `*` | معرف مركز التكلفة. |
| `is_relay` | `bool` | `false` | هل يتم تضمين القيود المرحلة فقط؟ |
| `currencys_id` | `string` | `*` (العملة الأساسية تُحسب تلقائياً) | معرف العملة. |
| `person` | `object\|null` | `null` | كائن الشخص (يتم استخراج `id` و `type` منه). |
| `person_id` | `int\|string` | `null` | معرف الشخص. |
| `person_type` | `string` | `null` | نوع الشخص (Morph Class). |
| `return_value` | `string` | `'stock'` | القيمة المطلوب إرجاعها: `'stock'`، `'debit'`، `'credit'`، `'stock_alien'`، `'debit_alien'`، `'credit_alien'`، أو أي مفتاح آخر موجود في النتيجة. |
| `with_currency` | `bool` | `false` | هل يتم إرجاع معلومات العملة مع النتيجة؟ |
| `add_select` | `array` | `[]` | أعمدة إضافية لإضافتها إلى `SELECT`. |
| `groub_by` | `array` | `[]` | أعمدة للتجميع حسبها (`GROUP BY`). |
| `field_date` | `string\|null` | `null` | حقل التاريخ المطلوب الفلترة عليه (`'date_at'`، `'created_at'`، أو `'*'`). |
| `date_at` | `string\|Carbon\|null` | `null` | تاريخ البداية للفلترة. |
| `date_at_opration` | `string\|null` | `'='` | عامل المقارنة للتاريخ (مثل `=`, `>=`, `<=`). |
| `date_at_no_format` | `bool\|null` | `null` | إذا كان `true`، يتم استخدام التاريخ كما هو بدون تنسيق. |
| `to_date_at` | `string\|Carbon\|null` | `null` | تاريخ النهاية للفلترة. |
| `is_stop_force_periods` | `bool` | `false` | إذا كان `true`، يتم تجاهل فلترة الفترة التلقائية. |

#### القيمة المرجعة

تعيد الدالة مصفوفة تحتوي على المفاتيح التالية:

| المفتاح | النوع | الوصف |
| :--- | :--- | :--- |
| `debit` | `float` | إجمالي المبالغ المدينة (بالعملة الرئيسية). |
| `credit` | `float` | إجمالي المبالغ الدائنة (بالعملة الرئيسية). |
| `stock` | `float` | صافي الرصيد (`debit - credit`). |
| `debit_alien` | `float` | إجمالي المبالغ المدينة بالعملة الأجنبية. |
| `credit_alien` | `float` | إجمالي المبالغ الدائنة بالعملة الأجنبية. |
| `stock_alien` | `float` | صافي الرصيد بالعملة الأجنبية. |

بالإضافة إلى أي أعمدة إضافية تم طلبها عبر `add_select`.

إذا تم تمرير `return_value` محدد (مثل `'stock'`)، يتم إرجاع قيمة مفردة (`float`) بدلاً من المصفوفة.

إذا كان `with_currency = true`، تحتوي النتيجة أيضاً على المفتاح `currency` الذي يحمل معلومات العملة.

#### أمثلة

**جلب رصيد حساب معين:**
```php
$balance = AccountHelper::getBalances('2-2-1231010001');
echo "الرصيد: " . $balance['stock'];
// أو مباشرة:
$stock = AccountHelper::getBalances('2-2-1231010001', ['return_value' => 'stock']);
```

**جلب رصيد عميل معين في فرع و فترة محددين:**
```php
$balance = AccountHelper::getBalances(null, [
    'person_type' => 'RainLab\User\Models\User',
    'person_id'   => 15,
    'departments_id' => 3,
    'periods_id'  => 5,
    'return_value' => 'stock',
]);
```

**جلب رصيد مع تفاصيل العملة:**
```php
$result = AccountHelper::getBalances('2-2-1231010001', [
    'with_currency' => true,
]);
print_r($result['currency']); // معلومات العملة
echo $result['stock'];        // الرصيد
```

---

### 2. `getTransactionsDetails`

جلب تفاصيل الحركات (`TransactionsDetail`) مع إمكانية تطبيق مجموعة واسعة من الفلاتر. تستخدم هذه الدالة بشكل أساسي في تقارير الحركات وفي `AccountsHelpers` في `Nano.AccountsApi`.

```php
public static function getTransactionsDetails($code, $options = [], $is_sql = false)
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$code` | `string\|array\|object\|null` | كود الحساب (أو مصفوفة أكواد، أو كائن الشخص). |
| `$options` | `array` | مصفوفة خيارات الفلترة (انظر الجدول أدناه). |
| `$is_sql` | `bool` | إذا كان `true`، يتم إرجاع استعلام Eloquent بدون تنفيذه (للتصحيح). |

#### خيارات `$options` المدعومة

| المفتاح | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `periods_id` | `string` | `*` (تُحسب تلقائياً) | معرف الفترة. |
| `departments_id` | `string` | `*` | معرف القسم. |
| `cost_centers_id` | `string` | `*` | معرف مركز التكلفة. |
| `is_relay` | `bool` | `false` | تضمين المرحّل فقط؟ |
| `currencys_id` | `string` | `*` | معرف العملة. |
| `person` | `object\|null` | `null` | كائن الشخص. |
| `person_id` | `int\|string` | `null` | معرف الشخص. |
| `person_type` | `string` | `null` | نوع الشخص. |
| `accounts_type` | `string` | `'custom_accounts'` | نوع البحث عن الحساب: `'custom_accounts'`، `'parent_accounts'`، `'custom_type'`. |
| `details_type` | `string\|null` | `null` | نوع التفصيل: `'debit'` أو `'credit'`. |
| `transactions_type` | `string\|array\|null` | `null` | نوع الحركة (أو مصفوفة). |
| `transactions_id` | `int\|null` | `null` | معرف رأس الحركة. |
| `not_transactions_id` | `int\|null` | `null` | استبعاد رأس حركة محدد. |
| `details_id` | `int\|null` | `null` | معرف تفصيل محدد. |
| `paper_id` | `string\|null` | `null` | رقم المستند الورقي. |
| `created_by` | `int\|null` | `null` | معرف المنشئ. |
| `notes` | `string\|null` | `null` | نص الملاحظات (بحث جزئي). |
| `field_date` | `string\|null` | `null` | حقل التاريخ للفلترة. |
| `date_at` | `string\|Carbon\|null` | `null` | تاريخ البداية. |
| `to_date_at` | `string\|Carbon\|null` | `null` | تاريخ النهاية. |
| `date_at_opration` | `string\|null` | `'='` | عامل المقارنة. |
| `date_at_no_format` | `bool\|null` | `null` | استخدام التاريخ بدون تنسيق. |
| `custome_opration` | `string\|null` | `null` | عامل مقارنة مخصص للمبلغ (`=`, `>`, `<`). |
| `custome_value` | `float\|null` | `null` | قيمة المبلغ للتخصيص. |
| `custom_with` | `array\|string\|null` | `null` | علاقات إضافية للتحميل (Eager Load). |
| `with_count` | `bool` | `false` | تحميل عدد العلاقات. |
| `take_limet` | `int\|null` | `null` | عدد السجلات المطلوبة. |
| `orderBy` | `string` | `'created_at'` | ترتيب حسب. |
| `orderDirection` | `string` | `'asc'` | اتجاه الترتيب. |
| `is_stop_force_periods` | `bool` | `false` | تجاهل فلترة الفترة التلقائية. |

#### القيمة المرجعة

- إذا كان `$is_sql = true`، يتم إرجاع كائن `Query Builder` (بدون تنفيذ).
- في الحالة الطبيعية، يتم إرجاع مجموعة (Collection) من كائنات `TransactionsDetail` مع العلاقات `Account`، `Currency`، `TransactionsType`.

#### أمثلة

**جلب تفاصيل حركات حساب معين في شهر محدد:**
```php
$details = AccountHelper::getTransactionsDetails('2-2-1231010001', [
    'field_date' => 'date_at',
    'date_at'    => '2026-01-01',
    'to_date_at' => '2026-01-31',
    'orderBy'    => 'date_at',
    'orderDirection' => 'desc',
]);
foreach ($details as $detail) {
    echo $detail->debit . ' | ' . $detail->credit;
}
```

**جلب تفاصيل حركات عميل معين:**
```php
$details = AccountHelper::getTransactionsDetails(null, [
    'person_type' => 'RainLab\User\Models\User',
    'person_id'   => 42,
]);
```

---

### 3. `getTotalAcc`

دالة مشابهة لـ `getBalances` ولكنها أبسط وأقل مرونة. تحسب إجمالي الحساب (`debit` أو `credit` أو `stock`) لحساب واحد فقط. يفضل استخدام `getBalances` في التطويرات الجديدة.

```php
public static function getTotalAcc($code, $options = [])
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$code` | `string` | كود الحساب. |
| `$options` | `array` | خيارات الفلترة (مشابهة لـ `getBalances` ولكن أبسط). |

#### خيارات `$options`

- `periods_id`، `departments_id`، `cost_centers_id`، `is_relay`، `is_debit`، `currencys_id`، `return_value`.

#### مثال

```php
$total = AccountHelper::getTotalAcc('2-2-1231010001', [
    'return_value' => 'stock',
]);
```

---

### 4. `getQueryDate`

دالة مساعدة داخلية لتطبيق شروط التاريخ على استعلام معين. تدعم ثلاثة أنماط من الفلترة: على `date_at` فقط، على `created_at` فقط، أو على كليهما (`*`). تستخدم بشكل واسع داخل `getBalances` و `getTransactionsDetails` و `getTransactionHeader`.

```php
public static function getQueryDate($records, $options = [])
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$records` | `Query Builder` | استعلام Eloquent المطلوب تطبيق الفلتر عليه. |
| `$options` | `array` | خيارات التاريخ: `field_date`، `date_at`، `date_at_opration`، `date_at_no_format`، `to_date_at`. |

#### القيمة المرجعة

الاستعلام بعد تطبيق شروط التاريخ.

#### مثال

```php
$query = TransactionsDetail::query();
$query = AccountHelper::getQueryDate($query, [
    'field_date' => 'date_at',
    'date_at'    => '2026-01-01',
    'to_date_at' => '2026-01-31',
]);
// الآن يمكن تنفيذ $query->get()
```

---

### 5. `getObjHeader`

إنشاء كائن رأس حركة (`TransactionHeader`) بالنوع المناسب (بناءً على `type_header`) معبأً بالقيم الافتراضية والخيارات الممررة. هذه الدالة هي المدخل الأساسي لإنشاء أي رأس قيد أو سند أو إشعار.

```php
public static function getObjHeader($options = [])
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$options` | `array` | خيارات رأس الحركة. |

#### الخيارات الأساسية

| المفتاح | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `type_header` | `string\|null` | `null` | نوع رأس الحركة (مثل `'BondsDay'` أو `BondsDay::class`). إذا لم يحدد، يتم إنشاء `TransactionHeader` عام. |
| `periods_id` | `string` | الفترة الأساسية | معرف الفترة. |
| `departments_id` | `string\|null` | `null` | معرف القسم. |
| `cost_centers_id` | `string\|null` | `null` | معرف مركز التكلفة. |
| `employees_id` | `int\|null` | الموظف المرتبط بالمستخدم الحالي | معرف الموظف. |
| `date_at` | `Carbon` | `Carbon::now()` | تاريخ الحركة. |
| `transactions_type` | `int` | `4` | نوع الحركة. |
| `process_type` | `int\|string` | `4` | نوع العملية. |
| `patterns_id` | `int` | `4` | معرف النمط. |
| `status` | `string` | `'active'` | الحالة. |
| `is_active` | `bool` | `true` | هل هو نشط؟ |
| `is_relay` | `bool` | `false` | هل تم ترحيله؟ |

#### مثال

```php
$header = AccountHelper::getObjHeader([
    'type_header'      => BondsDay::class,
    'departments_id'   => 2,
    'transactions_type'=> 1,
    'notes'            => 'قيد يومي تجريبي',
]);
$header->save();
```

---

### 6. `getObjHeaderFromType`

دالة مساعدة لتحويل اسم نوع المستند (سلسلة نصية أو اسم كلاس) إلى كائن من النوع المناسب. تستخدم داخلياً في `getObjHeader`.

```php
public static function getObjHeaderFromType($type_header = null)
```

الأنواع المدعومة:

| القيمة (سلسلة أو كلاس) | الكائن المنشأ |
| :--- | :--- |
| `'BondsDay'` / `BondsDay::class` | `new BondsDay()` |
| `'Bonds2Day'` / `Bonds2Day::class` | `new Bonds2Day()` |
| `'OpeningBalance'` / `OpeningBalance::class` | `new OpeningBalance()` |
| `'CatchReceipt'` / `CatchReceipt::class` | `new CatchReceipt()` |
| `'PayReceipt'` / `PayReceipt::class` | `new PayReceipt()` |
| `'DebitNote'` / `DebitNote::class` | `new DebitNote()` |
| `'CreditNote'` / `CreditNote::class` | `new CreditNote()` |
| `'Income'` / `Income::class` | `new Income()` |
| `'Expense'` / `Expense::class` | `new Expense()` |
| `'Transfer'` / `Transfer::class` | `new Transfer()` |
| أي قيمة أخرى | `new TransactionHeader()` |

#### مثال

```php
$payReceipt = AccountHelper::getObjHeaderFromType('PayReceipt');
// أو
$payReceipt = AccountHelper::getObjHeaderFromType(PayReceipt::class);
```

---

### 7. `getObjDetail`

إنشاء كائن تفصيل حركة (`TransactionsDetail`) معبأً بالقيم الافتراضية والخيارات الممررة. تستخدم لإنشاء أطراف القيد.

```php
public static function getObjDetail($options = [])
```

#### الخيارات الأساسية

| المفتاح | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `currencys_id` | `string` | العملة الأساسية | معرف العملة. |
| `rate` | `float` | `1` | سعر الصرف. |
| ... | ... | ... | (باقي الخيارات مشابهة لـ `getObjHeader`) |

#### مثال

```php
$detail = AccountHelper::getObjDetail([
    'accounts_id'      => '2-2-1231010001',
    'debit'            => 1000,
    'currencys_id'     => 1,
    'transactions_id'  => 55,
]);
$detail->save();
```

---

### 8. `getRefAccount`

جلب المرجع (`Reference`) المرتبط بنموذج معين. تعتمد على جدول `tss_accounts_references` للعثور على المرجع المناسب حسب اسم الجدول، الكود، أو المعرف.

```php
public static function getRefAccount($model, $options = [], $is_model = false)
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$model` | `string\|object` | مسار الكلاس أو كائن النموذج. |
| `$options` | `array` | خيارات إضافية (`companys_id`، `ref_id`، `ref_code`، `ref_table_name`، `ref_code_name`). |
| `$is_model` | `bool` | إذا كان `true`، يتم إرجاع الكائن مباشرة. |

#### القيمة المرجعة

- إذا كان `$is_model = false` (الافتراضي): مصفوفة تحتوي على `status`, `message`, `model` (كائن `Reference` أو `null`).
- إذا كان `$is_model = true`: كائن `Reference` أو `null`.

#### مثال

```php
$ref = AccountHelper::getRefAccount(\Tss\Basic\Models\Employee::class, [], true);
if ($ref) {
    echo "اسم الكود: " . $ref->code_name;
}
```

---

### 9. `getParentAccount`

جلب الحساب الأم (`Account`) المرتبط بنموذج معين، وذلك بالاعتماد على المرجع (`Reference`) أو إعدادات `config` (مثل `tss.accounts::accounts.employees_account`). هذه الدالة تستخدم بشكل كبير في إنشاء الحسابات الفرعية تلقائياً.

```php
public static function getParentAccount($model_path, $options = [], $is_model = false)
```

#### مثال

```php
$parentAcc = AccountHelper::getParentAccount(Employee::class, [], true);
if ($parentAcc) {
    echo "الحساب الأم: " . $parentAcc->name;
}
```

---

### 10. `createAccountToModel`

إنشاء حساب فرعي جديد (`Account`) لنموذج معين (مثل موظف، طالب) وربطه تلقائياً بالمرجع والحساب الأم. يتم تعبئة الحقول تلقائياً (الاسم، الوصف، `modul_type`، `ref_type`، إلخ) وحفظ الحساب. إذا كان الخيار `is_update_model_accounts_id` مفعلاً (الافتراضي)، يتم تحديث حقل `accounts_id` في النموذج بكود الحساب الجديد.

```php
public static function createAccountToModel(Model $model, $parentAcc = null, $options = [], $is_model = false)
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$model` | `Model` | كائن النموذج المراد إنشاء حساب له. |
| `$parentAcc` | `Account\|null` | الحساب الأم (إذا لم يُمرر، يتم البحث عنه تلقائياً). |
| `$options` | `array` | خيارات إضافية (مثل `acc_name`، `acc_prefix`). |
| `$is_model` | `bool` | إذا كان `true`، يتم إرجاع الحساب مباشرة. |

#### مثال

```php
$employee = Employee::find(10);
$result = AccountHelper::createAccountToModel($employee);
if ($result['status']) {
    echo "تم إنشاء الحساب: " . $result['model']->code;
    echo "تم تحديث الموظف: " . $employee->accounts_id;
}
```

---

### 11. `getNormalizDetailTrans`

تطبيع (تجميع) تفاصيل حركات متعددة إلى مصفوفة واحدة تجمع الحركات حسب `transactions_id`. مفيدة عند عرض كشف حساب حيث يتم تجميع `debit` و `credit` لكل حركة.

```php
public static function getNormalizDetailTrans($detailsObj)
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$detailsObj` | `Collection` | مجموعة من كائنات `TransactionsDetail`. |

#### القيمة المرجعة

مصفوفة ارتباطية مفتاحها `transactions_id`، وقيمتها مصفوفة تحتوي على:
- `transactions_id`, `date_at`, `created_at`
- `debit`, `credit`, `debit_alien`, `credit_alien` (مجمعة)
- `is_alien` (هل العملة أجنبية؟)
- `mony` (المبلغ النهائي بالعملة المناسبة)

#### مثال

```php
$details = TransactionsDetail::whereIn('transactions_id', [10,11,12])->get();
$normalized = AccountHelper::getNormalizDetailTrans($details);
foreach ($normalized as $transId => $data) {
    echo "الحركة $transId: مدين {$data['debit']} | دائن {$data['credit']}";
}
```

---

### 12. دوال تصفية المستخدمين

**`getQueryWhereHasMorphPersonUsersRefType`**: تصفية سجلات `TransactionsDetail` التي يكون فيها `person` من نوع `User` ولديه `ref_type` محدد (مثل `delivery`).

**`getQueryWhereUsersRefType`**: تصفية سجلات المستخدمين مباشرة بناءً على `ref_type`.

```php
public static function getQueryWhereHasMorphPersonUsersRefType($records, $users_ref_type)
public static function getQueryWhereUsersRefType($records, $users_ref_type, $columnName = 'users.ref_type')
```

#### مثال

```php
$query = TransactionsDetail::query();
$query = AccountHelper::getQueryWhereHasMorphPersonUsersRefType($query, 'delivery');
// يجلب تفاصيل الحركات التي يكون فيها الشخص سائق توصيل فقط
```

---

### 13. `checkShopReport` و `checkShopReportData`

دوال خاصة بفحص تقارير المتاجر في حالة تعدد الفروع (مطاعم، متاجر). تحدد ما إذا كان يجب تطبيق فلترة إضافية على التقارير بناءً على إعدادات `tss.accounts::check_shop_report`.

```php
public static function checkShopReport($type = 'sub', $is_force = false)
public static function checkShopReportData($data = [], $type = 'sub')
```

#### مثال

```php
$reportFilters = AccountHelper::checkShopReportData([
    'departments_id' => 5,
]);
// الآن يمكن استخدام $reportFilters في استعلام التقرير
```

---

## الأحداث (Events)

كلاس `AccountHelper` لا يطلق أحداثاً بشكل مباشر، ولكنه يعتمد على أحداث `TransactionsHelper` (مثل `tss.accounts.beforeJournalEntry`) عند استخدامه مع الترايت الجديد `TransactionsMultiHelper`.

---

## الاعتماديات (Dependencies)

يعتمد `AccountHelper` على المكونات التالية:

- **`Tss\Accounts\Models\*`**: جميع النماذج المحاسبية (Account, TransactionHeader, TransactionsDetail, BondsDay, CatchReceipt, إلخ).
- **`Tss\Currency\Models\Currency`**: لإدارة العملات.
- **`Tss\Accounts\Helpers\PersonHelper`**: لمعالجة أنواع الأشخاص (Morph Classes).
- **`Tss\Basic\Helpers\BasicHelper`**: لتجهيز معرفات الشركة والفرع الافتراضية.
- **`Illuminate\Support\Facades\DB`**: لبناء الاستعلامات الخام.
- **`Carbon\Carbon`**: لمعالجة التواريخ.
- **`BackendAuth`**: للحصول على المستخدم الخلفي الحالي.
- **`Config`**: لقراءة الإعدادات الافتراضية.

---

## ملاحظات هامة

1. **استخدام القيم الافتراضية التلقائية**: في دوال `getBalances` و `getTransactionsDetails`، إذا لم يتم تمرير `periods_id` أو `currencys_id`، يتم جلبهما تلقائياً من الإعدادات أو من `getPrimary`. يمكن تعطيل ذلك عبر `is_stop_force_periods = true`.

2. **التعامل مع `use_default_company`**: إذا كان إعداد `tss.accounts::use_default_company` مفعلاً، يتم تجاهل القسم والشركة الممررين واستخدام القيم الافتراضية للشركة الأم. هذا سلوك هام يجب مراعاته.

3. **دعم العملات الأجنبية**: تحتوي النتائج على مفاتيح `_alien` (`debit_alien`, `credit_alien`, `stock_alien`) تمثل القيم بالعملة الأجنبية. عند طلب `return_value` محدد، يتم إرجاع القيمة بالعملة المناسبة تلقائياً (أجنبية إذا كانت العملة المحددة غير أساسية).

4. **فلترة التاريخ المرنة**: دالة `getQueryDate` تدعم البحث بتاريخ محدد (`=`) أو نطاق (`>=...<=`) مع تجاهل التنسيق (`date_at_no_format`) مما يسمح ببحث دقيق حسب الوقت أيضاً.

5. **إنشاء الحسابات التلقائية**: `createAccountToModel` تعتمد على وجود `ref_type` و `parent_id` معرفين بشكل صحيح. تأكد من وجود المراجع (`References`) والحسابات الأم قبل استخدامها.

6. **الثبات (Static)**: جميع الدوال ثابتة، مما يعني أنها لا تحتفظ بحالة داخلية ويمكن استدعاؤها من أي مكان. هذا يسهل الاختبار ولكنه يعني أيضاً أنه لا يمكن تجاوزها بسهولة. للتوسع، يمكن إنشاء كلاسات مساعدة إضافية بدلاً من تعديل هذا الكلاس.

---

## أمثلة متكاملة

### السيناريو 1: حساب رصيد عميل وعرضه في واجهة المستخدم

```php
$user = Auth::getUser();
$balance = AccountHelper::getBalances(null, [
    'person_type' => $user->getMorphClass(),
    'person_id'   => $user->getKey(),
    'return_value' => 'stock',
]);
echo "رصيدك الحالي: " . number_format($balance, 2) . " ريال";
```

### السيناريو 2: إنشاء حساب فرعي تلقائي عند تسجيل طالب جديد

```php
$student = new Student();
$student->fill($request->all());
$student->save();

$result = AccountHelper::createAccountToModel($student);
if ($result['status']) {
    Log::info('تم إنشاء حساب للطالب: ' . $student->name . ' برقم ' . $student->accounts_id);
}
```

### السيناريو 3: تجهيز رأس وتفاصيل قيد يومي وحفظها

```php
$header = AccountHelper::getObjHeader([
    'type_header' => BondsDay::class,
    'transactions_type' => 1,
    'notes' => 'قيد تسوية',
]);
$header->save();

$detail1 = AccountHelper::getObjDetail([
    'accounts_id' => '2-2-1253010001',
    'debit'       => 500,
    'transactions_id' => $header->id,
]);
$detail1->save();

$detail2 = AccountHelper::getObjDetail([
    'accounts_id' => '2-2-1231010001',
    'credit'      => 500,
    'transactions_id' => $header->id,
]);
$detail2->save();
```

---

## الخاتمة

يمثل كلاس `AccountHelper` أداة لا غنى عنها في النظام المحاسبي لنانوسوفت، حيث يقدم مجموعة شاملة من الدوال المساعدة التي تبسط التعامل مع الحسابات والأرصدة والحركات والمراجع. بفضل قدرته على التعامل مع مختلف أنواع المستندات المالية، ودعمه للعملات المتعددة، ومرونته في تطبيق الفلاتر، يمكن للمطورين بناء تطبيقات محاسبية قوية معتمدين على هذا الكلاس كقاعدة صلبة. نأمل أن يكون هذا التوثيق دليلاً شاملاً يساعدك في تحقيق أقصى استفادة من هذه الأداة.


## التوثيق الإضافي

**الوثائق المرجعية**:
- [توثيق كلاس `AccountHelper`](./Docs-AccountHelper-ar.md)
- [توثيق كلاس `PersonHelper`](./Docs-PersonHelper-ar.md)
- [توثيق متقدم لـ `TransactionsHelper`](./Docs-TransactionsHelper-Advanced-ar.md)
- [فهرس دوال `TransactionsHelper`](./Docs-TransactionsHelper-Reference-Function-ar.md)
- [توثيق شامل لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-ar.md)
- [أمثلة عملية متقدمة لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)

