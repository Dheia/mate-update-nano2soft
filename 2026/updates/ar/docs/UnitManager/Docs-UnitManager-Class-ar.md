# توثيق الكلاس `UnitManager`

**Namespace:** `Tss\Inventory\Classes\UnitManager`

---

## 1. مقدمة

كلاس `UnitManager` هو الواجهة البرمجية الموحدة (Facade) للتعامل مع نظام الوحدات في حزمة `Tss.Inventory`. يوفر هذا الكلاس دوالاً ثابتة (static methods) تغطي معظم العمليات المتعلقة بالوحدات: إنشاء وحدات جديدة، التحقق من صحة الوحدات (خاصة كروت الشحن)، استرجاع قوائم الوحدات مصنفة، طباعة تقارير، توليد أكواد فريدة، وأدوات مساعدة للاستعلامات.

الهدف من هذا الكلاس هو تبسيط استخدام النظام وتوحيد الوصول إلى الوظائف المختلفة، مما يقلل من التكرار ويسهل صيانة الكود.

---

## 2. الدوال المتوفرة

### 2.1 دوال إنشاء وإدارة الوحدات

#### `createUnit(array $options = [], bool $is_test_create = false): array`

**الوصف:**  
إنشاء وحدة جديدة في قاعدة البيانات. تعتمد هذه الدالة على `UnitCreator::createUnit` وتقوم بتمرير الخيارات إليها.

**المعاملات:**
- `$options` : مصفوفة تحتوي على بيانات الوحدة. من أهم الخيارات:
  - `name` (string) : اسم الوحدة (مطلوب).
  - `unit_symbol` (string) : رمز الوحدة (مثل 'kg', 'm', 'dozen').
  - `type` (string) : نوع الوحدة (أحد ثوابت `UnitType`).
  - `companys_id` (int) : معرف الشركة.
  - `departments_id` (int) : معرف القسم.
  - `conversion_factor` (float) : عامل التحويل (بالنسبة للوحدة الأساسية).
  - `is_base_unit` (bool) : هل هي الوحدة الأساسية؟
  - `is_default` (bool) : هل هي افتراضية؟
  - `is_active` (bool) : حالة التفعيل.
  - وغيرها من الحقول الموجودة في نموذج `Unit`.
- `$is_test_create` : إذا كانت `true`، يتم إجراء العملية في معاملة (transaction) ثم التراجع (لأغراض الاختبار).

**القيمة المُرجعة:**  
مصفوفة تحتوي على:
- `status` (bool) : نجاح العملية أو فشلها.
- `message` (string) : رسالة توضيحية.
- `error` (string|null) : نص الخطأ إن وجد.
- `model` (object|null) : كائن الوحدة المُنشأة.
- `data` (array) : بيانات الوحدة (نسخة `toArray()`).
- `input_data`, `process_data` (array) : معلومات إضافية للتتبع.

**مثال:**
```php
$result = UnitManager::createUnit([
    'name' => 'كيلوجرام',
    'unit_symbol' => 'kg',
    'type' => UnitType::WEIGHT,
    'is_base_unit' => true,
    'conversion_factor' => 1,
    'companys_id' => 1,
    'departments_id' => 2,
]);

if ($result['status']) {
    echo "تم إنشاء الوحدة: " . $result['model']->name;
} else {
    echo "خطأ: " . $result['error'];
}
```

---

#### `checkUnit(array $options = []): array`

**الوصف:**  
التحقق من صحة وحدة (عادةً ما تستخدم للتحقق من كروت الشحن أو الرموز). تتحقق الدالة من وجود الوحدة، حالتها، رصيدها، ومطابقة المستخدم (إذا طُلب).

**المعاملات:**
- `$options` : مصفوفة تحتوي على معايير البحث والتحقق. من أهم الخيارات:
  - `unit_symbol` (string) : رمز الوحدة المراد التحقق منها (إجباري).
  - `companys_id` (mixed) : معرف الشركة (يمكن أن يكون `'*'` للبحث في الكل).
  - `departments_id` (mixed) : معرف القسم.
  - `total` (float) : القيمة المطلوبة (للتحقق من كفاية الرصيد).
  - `is_force_user` (bool) : هل يجب أن يكون المستخدم مطابقاً؟
  - `user` , `user_id` , `user_type` : بيانات المستخدم.
  - `is_model` (bool) : إذا كانت `true`، يتم إرجاع كائن الوحدة في النتيجة.

**القيمة المُرجعة:**  
مصفوفة تحتوي على:
- `status` (bool) : `true` إذا كانت الوحدة صالحة.
- `message` (string) : رسالة توضيحية.
- `error` (string|null) : نص الخطأ إن وجد.
- `model` (object|null) : كائن الوحدة (إذا طُلب `is_model`).
- `input_data`, `process_data` (array) : معلومات إضافية.

**أمثلة:**

*التحقق من وجود كرت شحن برمز معين:*
```php
$check = UnitManager::checkUnit([
    'unit_symbol' => 'CARD12345',
    'is_active' => true,
]);

if ($check['status']) {
    echo "الكرت صالح.";
} else {
    echo "الكرت غير صالح: " . $check['error'];
}
```

*التحقق من كرت مع التأكد من أن المستخدم هو مالكه:*
```php
$user = Auth::getUser();
$check = UnitManager::checkUnit([
    'unit_symbol' => 'CARD12345',
    'is_force_user' => true,
    'user' => $user,
]);

if (!$check['status']) {
    throw new Exception($check['error']);
}
```

*التحقق من كرت مع وجود رصيد كافٍ:*
```php
$check = UnitManager::checkUnit([
    'unit_symbol' => 'CARD12345',
    'total' => 100, // نريد استخدام 100 وحدة
]);

if ($check['status']) {
    echo "الرصيد كافٍ.";
} else {
    echo "رصيد غير كافٍ أو كرت غير صالح.";
}
```

---

### 2.2 دوال استعلام وإحصائيات

#### `getAllUnitsGrouped(bool $groupByMainType = true, string $lang = 'ar'): array`

**الوصف:**  
إرجاع جميع الوحدات المدعومة في النظام مصنفة حسب النوع (مع تسميات). تستخدم داخلياً `UnitConverter::getAllUnitsGrouped`.

**المعاملات:**
- `$groupByMainType` : إذا كانت `true`، يتم التجميع حسب الأنواع الرئيسية (مثل `length`, `weight`). إذا كانت `false`، يتم تضمين الأنواع الفرعية للكميات.
- `$lang` : اللغة (`'ar'` أو `'en'`).

**القيمة المُرجعة:**  
مصفوفة ترابطية حيث المفتاح هو معرف النوع، والقيمة مصفوفة تحتوي على:
- `label` : تسمية النوع.
- `units` : مصفوفة من رموز الوحدات وتسمياتها.
- `type` : معرف النوع.

**مثال:**
```php
$grouped = UnitManager::getAllUnitsGrouped(true, 'ar');
foreach ($grouped as $type => $info) {
    echo $info['label'] . ":\n";
    foreach ($info['units'] as $symbol => $label) {
        echo "  - $symbol : $label\n";
    }
}
```

---

#### `getUnitsForType($type, bool $includeLabels = true, string $lang = 'ar'): array`

**الوصف:**  
إرجاع قائمة بالوحدات التابعة لنوع معين.

**المعاملات:**
- `$type` : نوع الوحدة (مثل `UnitType::LENGTH`).
- `$includeLabels` : إذا كانت `true`، تُرجع مصفوفة ترابطية `[الرمز => التسمية]`؛ وإلا تُرجع قائمة عادية من الرموز.
- `$lang` : اللغة.

**القيمة المُرجعة:**  
مصفوفة من الرموز (أو مصفوفة ترابطية حسب `$includeLabels`).

**مثال:**
```php
$weightUnits = UnitManager::getUnitsForType(UnitType::WEIGHT, true, 'ar');
// ['mg' => 'مليجرام', 'g' => 'جرام', 'kg' => 'كيلوجرام', ...]
```

---

#### `getRentalTimeUnits(string $lang = 'ar', bool $is_key = false): array`

**الوصف:**  
إرجاع وحدات الزمن المستخدمة في سياق التأجير (ساعة، يوم، أسبوع، شهر).

**المعاملات:**
- `$lang` : اللغة.
- `$is_key` : إذا كانت `true`، تُرجع فقط قائمة الرموز (`['h','day','week','month']`). إذا كانت `false`، تُرجع مصفوفة ترابطية `[الرمز => التسمية]`.

**القيمة المُرجعة:**  
مصفوفة حسب `$is_key`.

**مثال:**
```php
$rentalUnits = UnitManager::getRentalTimeUnits('ar');
// ['h' => 'ساعة', 'day' => 'يوم', 'week' => 'أسبوع', 'month' => 'شهر']
```

---

### 2.3 دوال مساعدة للاستعلامات

#### `getQueryDate($records, array $options = [])`

**الوصف:**  
تطبيق شروط تاريخية على استعلام (Builder). تُستخدم داخلياً لإضافة فلاتر زمنية.

**المعاملات:**
- `$records` : كائن الاستعلام (Builder).
- `$options` : مصفوفة تحتوي على خيارات التاريخ، مثل:
  - `field_date` : اسم حقل التاريخ.
  - `date_at` : قيمة التاريخ (أو التاريخ من).
  - `to_date_at` : التاريخ إلى (للنطاق).
  - `date_at_opration` : العملية (`=`, `>=`, `between`, `notbetween` ...).
  - `is_or` : إذا كانت `true` تستخدم `orWhere`.

**القيمة المُرجعة:**  
كائن الاستعلام المعدل.

**مثال:**
```php
$query = Unit::query();
$query = UnitManager::getQueryDate($query, [
    'field_date' => 'created_at',
    'date_at' => '2026-03-01',
    'to_date_at' => '2026-03-18',
    'date_at_opration' => 'between',
]);
$units = $query->get();
```

---

#### `scopeWhereField($query, $field, $value = null, $is_or = false, $table = null, $is_not = false, $is_force = false)`

**الوصف:**  
تطبيق شرط `where` أو `whereIn` على حقل معين، مع دعم القيم الخاصة مثل `'*'` أو `'all'` (يتم تجاهلها إذا لم تكن `is_force`). تُستخدم في النطاقات (scopes).

**المعاملات:**
- `$query` : كائن الاستعلام.
- `$field` : اسم الحقل.
- `$value` : القيمة (قد تكون نصاً أو مصفوفة).
- `$is_or` : إذا كانت `true` تستخدم `orWhere`.
- `$table` : اسم الجدول (يُسبق للحقل).
- `$is_not` : إذا كانت `true` تستخدم `whereNotIn` أو `!=`.
- `$is_force` : إذا كانت `true`، يتم تطبيق الشرط حتى لو كانت القيمة `'*'` أو `'all'`.

**القيمة المُرجعة:**  
كائن الاستعلام المعدل.

**مثال:**
```php
$query = Unit::query();
$query = UnitManager::scopeWhereField($query, 'type', ['weight', 'length'], true, 'tss_inventory_units');
// ينتج: where (type in ('weight','length')) (بدون or لأن is_or = false)
```

---

### 2.4 دوال طباعة التقارير

#### `printReports(array $options = [])`

**الوصف:**  
طباعة تقرير عن الوحدات باستخدام نظام `Tss\Reports`. تُعيد محتوى HTML للتقرير.

**المعاملات:**
- `$options` : نفس الخيارات التي تقبلها `Unit::getRecords` (تحدد نطاق الوحدات المطلوب طباعتها).

**القيمة المُرجعة:**  
نص HTML يحتوي على التقرير.

**مثال:**
```php
$html = UnitManager::printReports([
    'type' => UnitType::WEIGHT,
    'is_active' => true,
    'orderBy' => 'name',
]);
echo $html; // عرض التقرير في المتصفح
```

---

### 2.5 دوال توليد الأكواد والمعرفات الفريدة

#### `generateDigital(int $length = 6): int`

**الوصف:**  
توليد رقم عشوائي بطول معين (لأكواد الكروت مثلاً).

**المعاملات:**
- `$length` : عدد الأرقام المطلوبة (افتراضي 6).

**القيمة المُرجعة:**  
رقم صحيح عشوائي.

**مثال:**
```php
$code = UnitManager::generateDigital(8); // مثال: 83472915
```

---

#### `getGuid(): string`

**الوصف:**  
توليد UUID (معرّف فريد عام) عشوائي بدون الأقواس `{}`.

**القيمة المُرجعة:**  
سلسلة نصية بصيغة `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

**مثال:**
```php
$uuid = UnitManager::getGuid(); // "f47ac10b-58cc-4372-a567-0e02b2c3d479"
```

---

#### `getVoucherCode(): string`

**الوصف:**  
توليد رمز قسيمة مكون من 13 رقمًا بناءً على الوقت وأرقام عشوائية.

**القيمة المُرجعة:**  
سلسلة رقمية مكونة من 13 رقمًا.

**مثال:**
```php
$voucher = UnitManager::getVoucherCode(); // "4829117456382"
```

---

#### `humanUuid(array $options = []): string`

**الوصف:**  
توليد UUID صديق للإنسان بصيغة `YYYYMMDD-HHMM-SSMM-MMMMRRRRRRRRRRRR` (32 خانة). يمكن إضافة شرطات.

**المعاملات:**
- `$options` : يمكن تمرير `'useDashes' => true` لإدراج الشرطات.

**القيمة المُرجعة:**  
سلسلة نصية.

**مثال:**
```php
$id = UnitManager::humanUuid(['useDashes' => true]); // "20260318-1430-1234-5678-123456789012"
```

---

#### `nanoUid(array $options = []): string`

**الوصف:**  
توليد معرف فريد بالنانوثانية، دقيق جداً، بصيغة `YYYYMMDD-HHMMSS-MMMMMM-NNN` (23 خانة).

**المعاملات:**
- `$options` : يمكن تمرير `'useDashes' => true`.

**القيمة المُرجعة:**  
سلسلة نصية.

**مثال:**
```php
$uid = UnitManager::nanoUid(); // "202603181430225643210"
```

---

### 2.6 دوال مساعدة أخرى

#### `getProductRecords(array $options = [])`

**الوصف:**  
إعادة توجيه إلى `Product::getRecords`. (للتوافق).

#### `newProductModel()`

**الوصف:**  
إنشاء كائن جديد من `Product`.

#### `newInventoryLotModel()` و `newUnitModel()`

**الوصف:**  
إنشاء كائنات جديدة من النماذج الخاصة (للإستخدام الداخلي).

#### `checkValueIsNotAll($value = null): bool`

**الوصف:**  
التحقق مما إذا كانت القيمة لا تمثل الكل (`'*'` أو `'all'`). تُستخدم في دوال الفلترة.

**مثال:**
```php
if (UnitManager::checkValueIsNotAll($value)) {
    // القيمة محددة وليست الكل
}
```

#### `getAllContexts(): array`

**الوصف:**  
إرجاع جميع السياقات المتاحة لوحدات الكميات (من `UnitConverter::getAllContexts`).

#### `getAllUnitLabels(): array`

**الوصف:**  
إرجاع جميع تسميات الوحدات (من `UnitNameHelper::getAllUnitsWithLabels`).

---

## 3. أفضل الممارسات

- **استخدم `createUnit` بدلاً من التعامل المباشر مع `UnitCreator`** للحصول على معالجة موحدة للأخطاء.
- **عند التحقق من كروت الشحن**، استخدم `checkUnit` مع المعاملات المناسبة (`is_force_user`, `total`).
- **للاستعلام عن الوحدات**، استخدم دوال `getAllUnitsGrouped` و `getUnitsForType` بدلاً من بناء المصفوفات يدوياً.
- **لتوليد أكواد فريدة**، اختر الدالة المناسبة حسب متطلبات الدقة والطول (مثلاً `nanoUid` للدقة العالية، `generateDigital` للأكواد الرقمية البسيطة).
- **عند إنشاء تقارير**، مرر خيارات التصفية عبر `printReports` بدلاً من بناء الاستعلام ثم التقرير يدوياً.

---

## 4. الخلاصة

كلاس `UnitManager` يقدم واجهة برمجية متكاملة وسهلة الاستخدام لجميع العمليات المتعلقة بالوحدات في نظام TSS Inventory. باستخدام دواله الثابتة، يمكن للمطورين:

- إنشاء وإدارة الوحدات.
- التحقق من صحة الوحدات (خاصة كروت الشحن).
- الحصول على قوائم منظمة للوحدات.
- تطبيق شروط متقدمة على الاستعلامات.
- طباعة تقارير احترافية.
- توليد أكواد ومعرفات فريدة.

بفضل هذا التصميم، يتم تقليل التكرار وزيادة قابلية الصيانة، مع توفير مرونة عالية لتلبية احتياجات التطبيقات المختلفة.