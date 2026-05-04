# توثيق شامل لدالة `createJournalEntry`

## نظرة عامة

تقع الدالة `createJournalEntry` ضمن السمة `TransactionsMultiHelper` التي يستخدمها الكلاس `TransactionsHelper`. تتيح هذه الدالة إنشاء قيود محاسبية متعددة الأطراف (Multi‑Leg Journal Entries) مع نظام تحقق شامل، معالجة متقدمة للعملات، فحص الأرصدة، وقواعد ديناميكية لفحص أنواع الحسابات، بالإضافة إلى إمكانية أتمتة معالجة فروق الصرف. تمثل هذه الدالة النواة الأساسية لإنشاء جميع أنواع القيود المتقدمة في النظام المالي لنانوسوفت.

---

## توقيع الدالة

```php
public static function createJournalEntry(array $options = [], bool $is_test_create = false): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$options` | `array` | مصفوفة خيارات إنشاء القيد (جميع المفاتيح اختيارية باستثناء `details`). |
| `$is_test_create` | `bool` | إذا كان `true`، يتم تنفيذ العملية داخل معاملة ثم التراجع عنها (لأغراض الاختبار). |

---

## تدفق تنفيذ الدالة

لفهم كيفية تأثير كل خيار على سلوك الدالة، من المهم معرفة تسلسل العمليات الداخلي:

1.  **التحقق من صحة المدخلات**: التحقق من أن `$options` مصفوفة غير فارغة.
2.  **`mergeDefaultCompanyValues`** – دمج جميع الخيارات المُمررة مع القيم الافتراضية الشاملة.
3.  **`preparePaperId`** – تجهيز `paper_id` (توليد تلقائي إذا كان `paper_id_auto = true`، إلغاء القيمة إذا كان `paper_id_allowed = false`).
4.  **`validatePaperIdRequired`** – التحقق من إلزامية `paper_id`.
5.  **`prepareDateAt`** – تجهيز `date_at` (تعيين التاريخ الحالي إذا كان `is_custome_date = false`).
6.  **`validatePaperId`** – التحقق من عدم تكرار `paper_id` إذا كان مطلوباً.
7.  **`validatePeriod`** – التحقق من صلاحية الفترة المحاسبية (التاريخ ضمن نطاق الفترة، الفترة مفتوحة).
8.  **`fireHook('beforeValidate', ...)`** – تنفيذ hook ما قبل التحقق.
9.  **`validateJournalEntry`** – التحقق الشامل من صحة القيد (توازن، عملات، قيود، حدود، أرصدة). تعيد التفاصيل بعد المعالجة.
10. **`fireHook('afterValidate', ...)`** – تنفيذ hook ما بعد التحقق.
11. **`calculateTotals`** – حساب مجاميع المدين والدائن الأساسية والأجنبية.
12. **إطلاق حدث `tss.accounts.beforeJournalEntry`**.
13. **`prepareHeaderOptions`** – تجهيز خيارات رأس القيد.
14. **إنشاء وحفظ كائن الرأس (`TransactionHeader`)**.
15. **حفظ التفاصيل** – لكل عنصر في `details`، يتم استدعاء `prepareDetailOptions` ثم `AccountHelper::getObjDetail` وحفظه.
16. **`handleExchangeDifferences`** – إذا كان `handle_exchange_differences = true`، يتم فحص فروق الصرف وإنشاء قيد فروق (أو إطلاق حدث).
17. **`Db::commit()`** (في حالة النجاح) أو **`Db::rollBack()`** (في حالة الخطأ أو وضع الاختبار).

---

## المصفوفة الكاملة للخيارات (`$options`)

### 1. حقول الرأس الأساسية

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 1.1 | `companys_id` | `int`/`string`/`null` | يُحسب تلقائياً من `getCheckCompanysId` | معرف الشركة. |
| 1.2 | `departments_id` | `int`/`string`/`null` | يُحسب تلقائياً من `getCheckDepartmentsId` | معرف القسم (الفرع). |
| 1.3 | `cost_centers_id` | `int`/`string`/`null` | يُحسب تلقائياً من `CostCenter::getPrimary` | معرف مركز التكلفة. |
| 1.4 | `periods_id` | `int`/`string`/`null` | يُحسب تلقائياً من `Period::getPrimary` | معرف الفترة المحاسبية. |
| 1.5 | `paper_id` | `mixed`/`null` | `null` | رقم المستند الورقي (نصي أو رقمي). يخضع لتحكم `is_paper_id`. |
| 1.6 | `employees_id` | `int`/`null` | `null` | معرف الموظف المسؤول عن القيد. |
| 1.7 | `date_at` | `Carbon`/`string`/`null` | `Carbon::now()` | تاريخ القيد. يخضع لتحكم `is_custome_date`. |
| 1.8 | `transactions_type` | `int` | `4` | نوع الحركة (من جدول `tss_accounts_transactions_types`). |
| 1.9 | `process_type` | `int`/`string` | `4` | نوع العملية (مثلاً `cash` أو `credit`). |
| 1.10| `patterns_id` | `int` | `4` | معرف النمط المحاسبي (`ConstraintsPattern`). |
| 1.11| `notes` | `string`/`null` | `null` | ملاحظات عامة على رأس القيد. |
| 1.12| `default_notes` | `string`/`null` | `null` | ملاحظة افتراضية تُستخدم إذا كانت `notes` فارغة. |
| 1.13| `type_header` | `string`/`null` | `null` | نوع رأس القيد (مثل `BondsDay::class`). |
| 1.14| `status` | `string` | `'active'` | حالة القيد (`active`, `inactive`, ...). |
| 1.15| `is_active` | `bool` | `true` | هل القيد نشط؟ |
| 1.16| `is_relay` | `bool` | `false` | هل تم ترحيله؟ |
| 1.17| `user_relay_id` | `int`/`null` | `null` | معرف المستخدم الذي قام بالترحيل. |
| 1.18| `relay_date_at` | `Carbon`/`null` | `null` | تاريخ الترحيل. |

### 2. الشخص على مستوى الرأس (`Header Person`)

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 2.1 | `header_person_id` | `int`/`null` | `null` | معرف الشخص المرتبط برأس القيد. |
| 2.2 | `header_person_type` | `string`/`null` | `null` | نوع الشخص (Morph Class). |
| 2.3 | `header_person` | `object`/`null` | `null` | كائن الشخص (يُستخرج منه `id` و `type` تلقائياً). |

### 3. إعدادات `paper_id` (المستند الورقي)

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 3.1 | `is_paper_id` | `bool` | `false` | هل نظام `paper_id` مفعّل أصلاً؟ إذا كان `false`، يتم تجاهل جميع الإعدادات التالية. |
| 3.2 | `paper_id_allowed` | `bool` | `true` | هل يُسمح للمستخدم بإدخال قيمة `paper_id`؟ إذا كان `false`، تُحذف أي قيمة مدخلة. |
| 3.3 | `paper_id_auto` | `bool` | `false` | هل يتم توليد `paper_id` تلقائياً إذا لم يُدخل؟ |
| 3.4 | `paper_id_required` | `bool` | `false` | هل `paper_id` إلزامي؟ (يتم التحقق منه في `validatePaperIdRequired`). |
| 3.5 | `paper_id_check_duplicate` | `bool` | `true` | هل نتحقق من عدم تكرار `paper_id`؟ |
| 3.6 | `paper_id_duplicate_scope` | `array` | `[]` | نطاق التحقق من التكرار: مصفوفة من `'departments_id'`, `'periods_id'`, `'transactions_type'`. |
| 3.7 | `validat_paper_id` | `string` | `'no_rapet'` | (للتوافق) طريقة التحقق القديمة. |
| 3.8 | `type_rapet` | `array` | `[]` | (للتوافق) شروط التحقق القديمة (مثل `['rapet_other_periods', 'rapet_other_type']`). |

### 4. إعدادات `date_at` (التاريخ)

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 4.1 | `is_custome_date` | `bool` | `false` | هل يُسمح للمستخدم بإدخال التاريخ؟ إذا كان `false`، يتم تعيينه إلى `now()`. |
| 4.2 | `date_at_auto` | `bool` | `false` | (للتوافق) توليد التاريخ تلقائياً إذا لم يُدخل. |
| 4.3 | `date_at_required` | `bool` | `false` | هل التاريخ إلزامي؟ |

### 5. العملات وفروق الصرف

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 5.1 | `main_currencys_id` | `string`/`null` | يُحسب تلقائياً | العملة الأساسية للشركة. |
| 5.2 | `currencys_id` | `string`/`null` | يُحسب تلقائياً | عملة القيد. |
| 5.3 | `rate` | `float` | `1` | سعر الصرف (يُستخدم إذا كانت العملة أجنبية). |
| 5.4 | `auto_convert_currency` | `bool` | `false` | تحويل المبالغ تلقائياً من العملة الأجنبية إلى الرئيسية. |
| 5.5 | `handle_exchange_differences` | `bool` | `false` | تفعيل فحص وإنشاء فروق الصرف بعد حفظ القيد. |
| 5.6 | `exchange_difference_action` | `string` | `'event'` | الإجراء عند وجود فروق: `'entry'` (إنشاء قيد)، `'event'` (إطلاق حدث)، `'none'` (تجاهل). |

### 6. قيود عدد الأطراف (`Multiplicity`)

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 6.1 | `max_debit_entries` | `int`/`null` | `null` | الحد الأقصى لعدد الأطراف المدينة (`null` = غير محدود). |
| 6.2 | `max_credit_entries` | `int`/`null` | `null` | الحد الأقصى لعدد الأطراف الدائنة. |
| 6.3 | `allow_multiple_debit` | `bool` | `true` | السماح بأكثر من طرف مدين. |
| 6.4 | `allow_multiple_credit` | `bool` | `true` | السماح بأكثر من طرف دائن. |

### 7. إعدادات الشخص لكل جانب

#### 7.1 الجانب المدين (`Debit Person`)

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 7.1.1 | `debit_person_allowed` | `bool` | `true` | السماح بربط شخص بالجانب المدين. |
| 7.1.2 | `debit_person_required` | `bool` | `false` | إلزامية وجود شخص في الجانب المدين. |
| 7.1.3 | `debit_person_allowed_types` | `array`/`null` | `null` | أنواع الأشخاص المسموح بها (مصفوفة من Morph Classes). `null` = جميع الأنواع. |

#### 7.2 الجانب الدائن (`Credit Person`)

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 7.2.1 | `credit_person_allowed` | `bool` | `true` | السماح بربط شخص بالجانب الدائن. |
| 7.2.2 | `credit_person_required` | `bool` | `false` | إلزامية وجود شخص في الجانب الدائن. |
| 7.2.3 | `credit_person_allowed_types` | `array`/`null` | `null` | أنواع الأشخاص المسموح بها للدائن. |

### 8. إعدادات الحساب لكل جانب

#### 8.1 الجانب المدين (`Debit Account`)

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 8.1.1 | `debit_account_allowed` | `bool` | `true` | السماح بتحديد حساب مدين. |
| 8.1.2 | `debit_account_required` | `bool` | `true` | إلزامية تحديد حساب مدين. |
| 8.1.3 | `debit_default_account` | `string`/`null` | `null` | الحساب الافتراضي للمدين (كود الحساب) إذا لم يُحدد. |

#### 8.2 الجانب الدائن (`Credit Account`)

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 8.2.1 | `credit_account_allowed` | `bool` | `true` | السماح بتحديد حساب دائن. |
| 8.2.2 | `credit_account_required` | `bool` | `true` | إلزامية تحديد حساب دائن. |
| 8.2.3 | `credit_default_account` | `string`/`null` | `null` | الحساب الافتراضي للدائن. |

### 9. إعدادات الملاحظات لكل جانب

#### 9.1 الجانب المدين (`Debit Notes`)

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 9.1.1 | `debit_notes_allowed` | `bool` | `true` | السماح بإدخال ملاحظة للمدين. |
| 9.1.2 | `debit_notes_required` | `bool` | `false` | إلزامية وجود ملاحظة للمدين. |
| 9.1.3 | `debit_default_notes` | `string`/`null` | `null` | الملاحظة الافتراضية للمدين. |

#### 9.2 الجانب الدائن (`Credit Notes`)

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 9.2.1 | `credit_notes_allowed` | `bool` | `true` | السماح بإدخال ملاحظة للدائن. |
| 9.2.2 | `credit_notes_required` | `bool` | `false` | إلزامية وجود ملاحظة للدائن. |
| 9.2.3 | `credit_default_notes` | `string`/`null` | `null` | الملاحظة الافتراضية للدائن. |

### 10. حدود المبالغ

#### 10.1 إجمالي القيد

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 10.1.1 | `min_total_amount` | `float`/`null` | `null` | الحد الأدنى للمبلغ الإجمالي. |
| 10.1.2 | `max_total_amount` | `float`/`null` | `null` | الحد الأقصى للمبلغ الإجمالي. |
| 10.1.3 | `check_min_total` | `bool` | `true` | تفعيل فحص الحد الأدنى. |
| 10.1.4 | `check_max_total` | `bool` | `true` | تفعيل فحص الحد الأقصى. |

#### 10.2 الجانب المدين

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 10.2.1 | `check_debit_min` | `bool` | `false` | تفعيل فحص الحد الأدنى للمدين. |
| 10.2.2 | `check_debit_max` | `bool` | `false` | تفعيل فحص الحد الأقصى للمدين. |
| 10.2.3 | `min_debit_amount` | `float`/`null` | `null` | الحد الأدنى لإجمالي المدين. |
| 10.2.4 | `max_debit_amount` | `float`/`null` | `null` | الحد الأقصى لإجمالي المدين. |

#### 10.3 الجانب الدائن

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 10.3.1 | `check_credit_min` | `bool` | `false` | تفعيل فحص الحد الأدنى للدائن. |
| 10.3.2 | `check_credit_max` | `bool` | `false` | تفعيل فحص الحد الأقصى للدائن. |
| 10.3.3 | `min_credit_amount` | `float`/`null` | `null` | الحد الأدنى لإجمالي الدائن. |
| 10.3.4 | `max_credit_amount` | `float`/`null` | `null` | الحد الأقصى لإجمالي الدائن. |

### 11. فحص الرصيد

#### 11.1 الجانب المدين

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 11.1.1 | `debit_check_balance` | `bool` | `false` | تفعيل فحص رصيد الحساب المدين قبل التنفيذ. |
| 11.1.2 | `debit_balance_check_callback` | `callable`/`null` | `null` | دالة مخصصة لفحص الرصيد (تستقبل `$account`, `$amount`, `$balance`, `$personId`, `$personType`, `$context`). |

#### 11.2 الجانب الدائن

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 11.2.1 | `credit_check_balance` | `bool` | `false` | تفعيل فحص رصيد الحساب الدائن. |
| 11.2.2 | `credit_balance_check_callback` | `callable`/`null` | `null` | دالة مخصصة لفحص رصيد الدائن. |

### 12. فحص نوع الحساب بقواعد ديناميكية

#### 12.1 الجانب المدين

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 12.1.1 | `debit_check_account_types` | `bool` | `false` | تفعيل فحص نوع الحساب في المدين. |
| 12.1.2 | `debit_rules_account_types` | `array`/`null` | `null` | قواعد التحقق (انظر قسم شرح القواعد أدناه). |

#### 12.2 الجانب الدائن

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 12.2.1 | `credit_check_account_types` | `bool` | `false` | تفعيل فحص نوع الحساب في الدائن. |
| 12.2.2 | `credit_rules_account_types` | `array`/`null` | `null` | قواعد التحقق للدائن. |

### 13. الحقول الإضافية (`Extra Fields`)

الحقول الإضافية هي: `extend_id`, `modul_type`, `ref_type`, `ref_key_name`, `ref_key_values`, `batch_type`, `batch_id`.
لكل حقل من هذه الحقول، توجد ثلاث مجموعات من الإعدادات: على مستوى الرأس (`header`)، على مستوى تفاصيل المدين (`debit`)، على مستوى تفاصيل الدائن (`credit`). كل مجموعة تحتوي على ثلاث خيارات:

| # | المفتاح (نمط) | النوع | القيمة الافتراضية | الوصف |
|---|--------------|------|-------------------|-------|
| 13.1 | `header_{field}_allowed` | `bool` | `true` | السماح باستخدام الحقل في رأس القيد. |
| 13.2 | `header_{field}_required` | `bool` | `false` | إلزامية الحقل في الرأس. |
| 13.3 | `header_{field}_default` | `mixed` | `null` | القيمة الافتراضية للحقل في الرأس. |
| 13.4 | `debit_{field}_allowed` | `bool` | `true` | السماح باستخدام الحقل في الجانب المدين. |
| 13.5 | `debit_{field}_required` | `bool` | `false` | إلزامية الحقل في الجانب المدين. |
| 13.6 | `debit_{field}_default` | `mixed` | `null` | القيمة الافتراضية للحقل في المدين. |
| 13.7 | `credit_{field}_allowed` | `bool` | `true` | السماح باستخدام الحقل في الجانب الدائن. |
| 13.8 | `credit_{field}_required` | `bool` | `false` | إلزامية الحقل في الجانب الدائن. |
| 13.9 | `credit_{field}_default` | `mixed` | `null` | القيمة الافتراضية للحقل في الدائن. |

### 14. إعدادات إضافية

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 14.1 | `auto_generate_custom_name` | `bool` | `true` | توليد اسم مخصص تلقائياً للشخص إذا لم يُمرر. |
| 14.2 | `beforeValidate` | `callable`/`null` | `null` | دالة تُنفذ قبل التحقق من صحة البيانات (تستقبل `&$options`, `&$details`). |
| 14.3 | `afterValidate` | `callable`/`null` | `null` | دالة تُنفذ بعد التحقق (تستقبل `&$options`, `&$details`). |
| 14.4 | `disabled_rules` | `string[]` | `[]` | أسماء القواعد المطلوب تعطيلها. القيم المدعومة: `'balance_check'`, `'multiplicity'`, `'amount_limits'`. |

---

## هيكل `details`

المفتاح `details` هو **المفتاح الإجباري الوحيد** لإنشاء قيد متعدد الأطراف. يحتوي على مصفوفة من عناصر التفاصيل، كل عنصر يمثل طرفاً محاسبياً واحداً.

### هيكل العنصر الواحد من `details`

| المفتاح | النوع | الوصف |
|--------|------|-------|
| `accounts_id` | `string` | **إجباري**. كود الحساب (مثل `'2-2-1253010001'`). |
| `debit` | `float` | المبلغ المدين (يجب أن يكون > 0 إذا كان مديناً، وإلا 0). |
| `credit` | `float` | المبلغ الدائن (يجب أن يكون > 0 إذا كان دائناً، وإلا 0). |
| `notes` | `string`/`null` | ملاحظة خاصة بهذا الطرف (اختياري). |
| `person_id` | `int`/`null` | معرف الشخص المرتبط (اختياري). |
| `person_type` | `string`/`null` | نوع الشخص (Morph Class). |
| `person` | `object`/`null` | كائن الشخص (يُستخرج منه `id` و `type` تلقائياً). |
| `custom_name_accounts` | `string`/`null` | اسم مخصص للحساب (اختياري). |
| `currencys_id` | `string`/`null` | عملة هذا الطرف (إذا اختلفت عن عملة الرأس). |
| `main_currencys_id` | `string`/`null` | العملة الأساسية (عادة تُترك لتُحدد تلقائياً). |
| `rate` | `float`/`null` | سعر الصرف لهذا الطرف (اختياري). |
| `extend_id` | `mixed`/`null` | (اختياري) معرف خارجي. |
| `modul_type` | `mixed`/`null` | (اختياري) نوع الوحدة. |
| `ref_type` | `mixed`/`null` | (اختياري) نوع المرجع. |
| `ref_key_name` | `mixed`/`null` | (اختياري) اسم مفتاح المرجع. |
| `ref_key_values` | `mixed`/`null` | (اختياري) قيمة مفتاح المرجع. |
| `batch_type` | `mixed`/`null` | (اختياري) نوع الدُفعة. |
| `batch_id` | `mixed`/`null` | (اختياري) معرف الدُفعة. |

**ملاحظات هامة:**
- يجب أن يكون مجموع قيم `debit` عبر جميع التفاصيل مساوياً لمجموع قيم `credit` (بتسامح 0.001).
- إذا كان `auto_convert_currency = true` وكانت عملة التفصيل تختلف عن العملة الرئيسية، يتم تحويل المبالغ تلقائياً وتخزين القيم الأصلية في `debit_alien` و `credit_alien`.
- لا يمكن أن يكون `debit` و `credit` كلاهما > 0 في نفس التفصيل.

---

## شرح الدوال المساعدة المرتبطة بالخيارات

### `applyAccountRules` وفحص نوع الحساب

عند تفعيل `debit_check_account_types` أو `credit_check_account_types`، يتم استدعاء الدالة `applyAccountRules` التي تقارن خصائص الحساب مع مجموعة من القواعد. تدعم القواعد بنيتين:

**أ. صيغة مبسطة (AND تلقائي):**
```php
$rules = [
    ['field' => 'reports', 'operator' => '=', 'value' => '1'],
    ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
];
```

**ب. صيغة متقدمة مع عامل منطقي:**
```php
$rules = [
    'operator' => 'OR',   // أو 'AND'
    'rules'    => [
        ['field' => 'reports', 'operator' => '=', 'value' => '1'],
        ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
    ],
];
```

**الحقول المدعومة:** أي عمود من جدول `tss_accounts_accounts` (`reports`, `final_account_id`, `modul_type`, `account_normal`, `groups_accounts_id`, `ref_type`, `person_type`, `account_number`, `parent_id`, `level`, `main_sub`، وغيرها).

**المشغلات المدعومة:** `=`, `==`, `!=`, `<>`, `>`, `>=`, `<`, `<=`, `LIKE`, `NOT LIKE`, `IN`, `NOT IN`, `REGEXP`.

### دوال الـ `Callback` لفحص الرصيد

الدوال `debit_balance_check_callback` و `credit_balance_check_callback` تستقبل ست معاملات:

| # | المعامل | النوع | الوصف |
|---|--------|------|-------|
| 1 | `$account` | `Account` | كائن الحساب الذي يتم فحصه. |
| 2 | `$amount` | `float` | المبلغ المطلوب (مدين أو دائن). |
| 3 | `$balance` | `float` | الرصيد الحالي للحساب (وفقاً للسياق). |
| 4 | `$personId` | `int`/`null` | معرف الشخص المرتبط بالتفصيل. |
| 5 | `$personType` | `string`/`null` | نوع الشخص. |
| 6 | `$context` | `array` | الخيارات العامة للقيد (نفس `$options`). |

إذا تم تمرير `callback`، يتم استدعاؤه **بدلاً** من الفحص الافتراضي. يمكن للدالة أن ترمي `ApplicationException` لمنع القيد، أو `return` للسماح.

### `handleExchangeDifferences`

تعمل فقط إذا تم تعيين `handle_exchange_differences = true`. تقوم لكل تفصيل بعملة غير رئيسية بحساب الفرق بين `rate` المستخدم والسعر الرسمي في `relay_date`. بناءً على `exchange_difference_action`:

- `'entry'`: إنشاء قيد فروق صرف جديد باستخدام `createExchangeDifferenceEntry`.
- `'event'`: إطلاق حدث `tss.accounts.exchangeDifference`.
- `'none'`: لا تفعل شيئاً.

---

## أمثلة عملية متكاملة

### مثال 1: قيد يومي بسيط (طرفان)

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 5000],
        ['accounts_id' => '2-2-1231010001', 'credit' => 5000],
    ],
    'notes' => 'إيداع نقدي',
    'type_header' => BondsDay::class,
]);

if ($result['status']) {
    echo "تم إنشاء القيد بنجاح، رقم القيد: " . $result['model']->id;
}
```

### مثال 2: قيد متعدد الأطراف مع أشخاص وفحص نوع الحساب

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        [
            'accounts_id' => '2-2-1253010001',
            'debit' => 3000,
            'person' => $cashier, // كائن أمين الصندوق
            'notes' => 'استلام نقدي'
        ],
        [
            'accounts_id' => '2-2-1231010001',
            'credit' => 2000,
            'person' => $customer,
        ],
        [
            'accounts_id' => '2-2-4111010001',
            'credit' => 1000,
            'notes' => 'إيرادات خدمات'
        ],
    ],
    'debit_check_account_types' => true,
    'debit_rules_account_types' => [
        ['field' => 'modul_type', 'operator' => '=', 'value' => 'Boxe']
    ],
    'credit_person_required' => true,
]);

if (!$result['status']) {
    echo "فشل الإنشاء: " . $result['error'];
}
```

### مثال 3: إنشاء قيد بحدود مبالغ وفحص رصيد مع `Callback`

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1231010001', 'debit' => 5000],
        ['accounts_id' => '2-2-1252010001', 'credit' => 5000],
    ],
    'min_total_amount' => 100,
    'max_total_amount' => 50000,
    'debit_check_balance' => true,
    'debit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
        $creditLimit = 2000; // حد ائتماني
        if ($balance + $creditLimit < $amount) {
            throw new ApplicationException("رصيد الحساب {$account->code} غير كافٍ حتى مع الحد الائتماني.");
        }
    },
]);
```

### مثال 4: استخدام `beforeValidate` لتعديل التفاصيل تلقائياً

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 1500],
        ['accounts_id' => '2-2-1231010001', 'credit' => 1500],
    ],
    'beforeValidate' => function(&$options, &$details) {
        // إضافة ضريبة 15% على المبلغ المدين
        foreach ($details as &$d) {
            if ($d['debit'] > 0) {
                $d['debit'] = round($d['debit'] * 1.15, 2);
            }
        }
    },
]);
```

### مثال 5: إنشاء قيد بعملة أجنبية مع معالجة فروق الصرف

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        [
            'accounts_id' => '2-2-1253010001',
            'debit' => 1000,
            'currencys_id' => 'USD', // عملة أجنبية
            'rate' => 3.67,
        ],
        [
            'accounts_id' => '2-2-1231010001',
            'credit' => 1000,
            'currencys_id' => 'USD',
            'rate' => 3.67,
        ],
    ],
    'main_currencys_id' => 'SAR',
    'currencys_id' => 'USD',
    'auto_convert_currency' => true, // تحويل المبالغ إلى SAR
    'handle_exchange_differences' => true,
    'relay_date_at' => '2026-06-01', // تاريخ ترحيل مختلف
    'exchange_difference_action' => 'entry',
]);

if ($result['status']) {
    echo "تم إنشاء القيد. إذا وجدت فروق صرف، تم إنشاء قيد إضافي.";
    // $result['exchange_entry'] قد يحتوي على قيد الفروق إذا تم إنشاؤه عبر checkExchangeDifferences
}
```

### مثال 6: استخدام `disabled_rules` لتعطيل بعض الفحوصات

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 5000],
        ['accounts_id' => '2-2-1231010001', 'credit' => 5000],
    ],
    'disabled_rules' => ['balance_check', 'multiplicity'], // تعطيل فحص الرصيد وعدد الأطراف
    'debit_check_balance' => true, // سيتم تجاهله بسبب disabled_rules
]);
```

---

## الاختبار (Testing Mode)

يمكن استخدام المعامل `$is_test_create = true` لمحاكاة العملية بالكامل ثم التراجع عنها. يعيد كائن القيد مع `test_mode => true`.

```php
$test = TransactionsHelper::createJournalEntry([
    'details' => [...],
], true);

if ($test['status']) {
    echo "القيد صالح ويمكن إنشاؤه.";
    // $test['model'] يحتوي على الكائن الذي كان سيُحفظ
    // لم يطرأ أي تغيير على قاعدة البيانات
}
```

---

## هيكل الاستجابة

| المفتاح | النوع | الوصف |
|--------|------|-------|
| `code` | `int` | `200` نجاح، `400` فشل. |
| `status` | `bool` | حالة العملية. |
| `message` | `string` | رسالة وصفية. |
| `error` | `string`/`null` | رسالة الخطأ. |
| `errors` | `array`/`null` | تفاصيل أخطاء التحقق. |
| `model` | `TransactionHeader`/`null` | كائن رأس القيد المُنشأ. |
| `input_data` | `array` | الخيارات المدخلة. |
| `process_data` | `array` | البيانات الفعلية المستخدمة (بعد الدمج). |
| `debug` | `array`/`null` | معلومات التصحيح (في بيئة `debug`). |


---

## آلية معالجة العملات تفصيلياً

### التحويل التلقائي (`auto_convert_currency`)
عند تعيين `auto_convert_currency = true`، تقوم الدالة المساعدة `processCurrency` (التي تُستدعى داخل `validateSingleDetail`) بالخطوات التالية:

1.  **تحديد العملة الرئيسية**: من `$context['main_currencys_id']` أو `CurrencyHelper::primaryCode()`.
2.  **تحديد عملة التفصيل**: إذا وُجد `currencys_id` في التفصيل يُستخدم، وإلا تُستخدم عملة الرأس (`$context['currencys_id']`)، ثم العملة الرئيسية.
3.  **إذا كانت العملة أجنبية**: تُنسخ قيم `debit`/`credit` المُدخلة إلى `debit_alien`/`credit_alien`، ثم تُحوّل القيم الأساسية بضربها في `rate`:  
    `$debit = $debit * $rate;`
4.  **إذا كانت العملة رئيسية أو كان `auto_convert_currency = false`**: لا يحدث تحويل، وتُترك القيم كما هي. (في حالة `false` والعملة أجنبية، يفترض أن `beforeSave` في الموديل سيتولى التحويل بناءً على `rate` و `debit_alien`/`credit_alien`).

**مثال عملي:**
```php
// إدخال قيد بعملة USD (أجنبية) مع تفعيل التحويل التلقائي
$options = [
    'main_currencys_id' => 'SAR',
    'currencys_id'      => 'USD',
    'auto_convert_currency' => true,
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 1000, 'rate' => 3.75],
        ['accounts_id' => '2-2-1231010001', 'credit' => 1000, 'rate' => 3.75],
    ],
];
// النتيجة في قاعدة البيانات:
// debit = 1000 * 3.75 = 3750 SAR,  debit_alien = 1000 USD
// credit = 3750 SAR, credit_alien = 1000 USD
```

### معالجة فروق الصرف (`handleExchangeDifferences`)
تعمل هذه الميزة بعد حفظ القيد بالكامل، وتتحقق من وجود `relay_date_at` مختلف عن `date_at`. لكل تفصيل بعملة أجنبية:

1.  تستدعي `CurrencyHelper::getRate` للحصول على السعر الرسمي في `relay_date`.
2.  تحسب الفرق: `$difference = $alienAmount * ($officialRate - $originalRate)`.
3.  إذا تجاوز الفرق قيمة `threshold` (افتراضياً 0.001)، يتم اتخاذ إجراء بناءً على `exchange_difference_action`:
    *   `'entry'`: إنشاء قيد فروق جديد عبر `createExchangeDifferenceEntry`.
    *   `'event'`: إطلاق حدث `tss.accounts.exchangeDifference` للمعالجة الخارجية.
    *   `'none'`: لا شيء.

يمكن أيضاً فحص الفروق بشكل مستقل لأي قيد موجود باستخدام الدالة العامة `TransactionsHelper::checkExchangeDifferences`.

---

## نظام الدمج مع القيم الافتراضية (`mergeDefaultCompanyValues`)

هذه الدالة هي المسؤولة عن ملء أي خيارات لم تُمرر بقيم افتراضية ذكية. تعتمد على:

*   **الإعدادات العامة**: `tss.accounts::use_default_company` يتحكم في ما إذا كانت القيم الافتراضية للشركة والقسم تُفرض بقوة.
*   **الدوال المساعدة**: `getCheckCompanysId`, `getCheckDepartmentsId`, `Period::getPrimary`, `Currency::getPrimary` لاستخراج القيم من قاعدة البيانات بناءً على السياق.
*   **مصفوفة ثوابت**: تحتوي على جميع المفاتيح المذكورة أعلاه بقيمها الافتراضية المذكورة.

**نقطة هامة:** يمكن تخصيص هذه القيم لمشروعك عن طريق إنشاء ملف إعدادات (مثلاً `config/nano/accountsapi.php`) يحتوي على مفتاح `journal_entry_defaults` وتمرير قيمك المخصصة. يتم دمجها مع القيم الافتراضية في `createJournalEntry` (يُفضل دمجها قبل الاستدعاء).

---

## دوال التحليل والاختبار

### `testJournalEntry`
لاختبار قيد قبل إنشائه فعلياً، استخدم `testJournalEntry`:

```php
$testResult = TransactionsHelper::testJournalEntry($options);
print_r($testResult['analysis']);
/*
[
    'header_prepared' => [...],
    'details_prepared' => [...],
    'totals' => ['total_debit_base' => 5000, 'total_credit_base' => 5000, ...],
    'currency_analysis' => ['SAR' => ['debit' => 5000, 'credit' => 5000]],
    'test_passed' => true,
    'rule_violations' => [],
    'warnings' => [],
]
*/
```

### `validateJournalEntry`
للتحقق فقط دون إنشاء (تُستخدم داخلياً وأيضاً خارجياً):

```php
try {
    $validation = TransactionsHelper::validateJournalEntry($options);
    // القيد صالح
} catch (ApplicationException $e) {
    echo $e->getMessage();
}
```

---

## إدارة الحقول الإضافية (`Extra Fields`) بشكل كامل

الحقول الإضافية (`extend_id`, `modul_type`, `ref_type`, `ref_key_name`, `ref_key_values`, `batch_type`, `batch_id`) هي أداة قوية لربط القيود بوحدات أعمال أخرى (طلبات، فواتير، موظفين). يسمح النظام بالتحكم في وجودها وإلزاميتها على ثلاثة مستويات: **رأس القيد**، **تفاصيل المدين**، **تفاصيل الدائن**.

**مثال: إلزامية وجود `batch_id` في رأس القيد، ومنع `modul_type` في التفاصيل المدينة:**

```php
$options = [
    'details' => [...],
    // الرأس: batch_id إلزامي
    'header_batch_id_required' => true,
    'batch_id' => 'BATCH-001',
    // المدين: modul_type غير مسموح
    'debit_modul_type_allowed' => false,
];
```
إذا حاول أي تفصيل مدين إرسال `modul_type`، سيتم رفض القيد. وإذا لم يتم إرسال `batch_id` في الخيارات الرئيسية، سيتم رفض القيد لأن `header_batch_id_required = true`.

---

## الأخطاء الشائعة وكيفية تجنبها

1.  **"Call to undefined method"**: يحدث إذا لم يتم تحديث `TransactionsHelper` ليستخدم السمة `use TransactionsMultiHelper`. تأكد من وجود السطر في الكلاس.
2.  **"The given data was invalid" بدون تفاصيل كافية**: استخدم `testJournalEntry` لترى بالضبط أي القواعد تم انتهاكها (`rule_violations`).
3.  **عدم توازن القيد بسبب العملات**: إذا كانت العملات مختلطة وتستخدم `auto_convert_currency`، تأكد من أن `rate` لكل تفصيل صحيح. استخدم `testJournalEntry` لفحص مجاميع `debit_alien` و `credit_alien`.
4.  **عدم عمل `handle_exchange_differences`**: تأكد من أن `relay_date_at` مُمرر في الخيارات أو موجود في كائن القيد، وأنه مختلف عن `date_at`. أيضاً تأكد من وجود حساب فروق صرف معرف في `tss.accounts::exchange_difference_account`.
5.  **تجاهل `disabled_rules`**: تذكر أن تعطيل `'balance_check'` مثلاً يعني تجاهل `debit_check_balance` حتى لو كان `true`.

---

## أفضل الممارسات

*   **اختبر قبل التنفيذ**: استخدم `testJournalEntry` أو `createJournalEntry($opts, true)` في بيئة التطوير.
*   **مركز الإعدادات**: بدلاً من تمرير خيارات مثل `debit_account_required` في كل استدعاء، عرّف قيماً افتراضية لمشروعك في ملف `config` وادمجها قبل الاستدعاء.
*   **استخدم `type_header` الصحيح**: دائماً استخدم `BondsDay::class` أو ما شابهها بدلاً من النصوص العادية لضمان استخدام النموذج الصحيح في `AccountHelper`.
*   **تعطيل الفحوصات بحذر**: `disabled_rules` قوي لكن أسيء استخدامه قد يُدخل بيانات غير متسقة. استخدمه فقط في حالات استثنائية (مثل ترحيل بيانات قديمة).
*   **راقب الأداء**: `debit_check_balance` ينفذ استعلام `getBalances` لكل تفصيل. للقيود الكبيرة جداً، قد يكون الأداء أفضل بفحص الرصيد في طبقة الأعمال قبل استدعاء الدالة.
*   **استفد من الأحداث**: الأحداث `beforeJournalEntry` و `afterJournalEntry` تسمح بربط منطق مخصص (إشعارات، تدقيق) دون تعديل الكود الأساسي.

---

## تكامل `createEntry` الموحدة

الدالة `createEntry` (الموجودة أيضاً في نفس السمة `TransactionsMultiHelper`) هي واجهة مبسطة تختار تلقائياً بين `createSingle` (لطرفين) و `createJournalEntry` (لأكثر من طرفين). يمكنك استخدامها بنفس الخيارات دون القلق بشأن أي دالة ستُستدعى:

```php
// ستستخدم createSingle تلقائياً (طرفان)
TransactionsHelper::createEntry([
    'mony' => 100,
    'debit_accounts_id' => '...',
    'credit_accounts_id' => '...',
]);

// ستستخدم createJournalEntry تلقائياً لأن عدد الأطراف > 2
TransactionsHelper::createEntry([
    'details' => [
        ['accounts_id' => '1', 'debit' => 500],
        ['accounts_id' => '2', 'credit' => 300],
        ['accounts_id' => '3', 'credit' => 200],
    ],
]);
```
لإجبار استخدام `createJournalEntry` حتى مع وجود طرفين، استخدم الخيار `'forceJournal' => true`.

---

## أفضل الممارسات والنصائح

1.  **اختبر قبل التنفيذ**: استخدم `testJournalEntry` أو `createJournalEntry($opts, true)` في بيئة التطوير للتأكد من صحة القيد ومنطق عملك قبل الالتزام بالبيانات.
2.  **مركز الإعدادات**: بدلاً من تمرير خيارات مثل `debit_account_required` في كل استدعاء، عرّف قيماً افتراضية لمشروعك في ملف `config` وادمجها قبل الاستدعاء. هذا يجعل الكود أنظف وأكثر اتساقاً.
3.  **استخدم `type_header` الصحيح**: استخدم `BondsDay::class` أو `CatchReceipt::class` بدلاً من النصوص العادية لضمان استخدام النموذج الصحيح في `AccountHelper::getObjHeaderFromType`.
4.  **تعطيل الفحوصات بحذر**: ميزة `disabled_rules` قوية لكن إساءة استخدامها قد تؤدي إلى بيانات مالية غير متسقة. استخدمها فقط في حالات استثنائية (مثل ترحيل بيانات تاريخية) ويفضل تسجيل سبب التعطيل في `notes`.
5.  **مراقبة الأداء**: خيار `debit_check_balance` ينفذ استعلام قاعدة بيانات (`AccountHelper::getBalances`) لكل تفصيل مدين. للقيود الكبيرة جداً (مئات الأطراف)، قد يكون من الأفضل إجراء فحص الرصيد في طبقة الأعمال بشكل مجمع قبل استدعاء الدالة.
6.  **استفد من الأحداث**: الأحداث `beforeJournalEntry` و `afterJournalEntry` و `tss.accounts.exchangeDifference` تسمح بربط منطق مخصص (إشعارات، تدقيق، تكامل مع أنظمة خارجية) دون تعديل الكود الأساسي.
7.  **توثيق القيد**: استخدم الحقول الإضافية (خاصة `batch_id`، `batch_type`، `extend_id`) بتوسّع لربط القيود بالكيانات التجارية التي أنشأتها (أوامر الشراء، الفواتير، العقود). هذا يسهل تتبع الحركات وتدقيقها.

---

## ملاحظات ختامية

- **القيم الافتراضية للمفاتيح التنظيمية**: معرفات الشركة (`companys_id`)، القسم (`departments_id`)، مركز التكلفة (`cost_centers_id`)، والفترة (`periods_id`) يتم جلبها تلقائياً من الإعدادات وسياق المستخدم. إذا كان إعداد `use_default_company` مفعلاً، يتم فرض القيم الافتراضية للشركة الأم ويتجاهل أي قيم مُمررة.
- **التعامل مع الأخطاء**: في حالة حدوث أي خطأ أثناء التحقق أو الحفظ، يتم التراجع عن كامل العملية (`rollback`) وإرجاع مصفوفة تحتوي على `status => false` ورسالة الخطأ في `error`. في وضع `debug`، يضاف مفتاح `debug` بمعلومات تفصيلية.
- **قابلية التوسع**: يمكن للمطورين إضافة قواعد تحقق جديدة أو تعديل سلوك الدالة من خلال نظام `beforeValidate`/`afterValidate` hooks أو بالاستماع للأحداث المطلقة.

بهذا نكون قد غطينا كافة جوانب دالة `createJournalEntry` بدءاً من الخيارات الأساسية وحتى آليات العمل الداخلية وأفضل الممارسات. يمكن للمطور الآن استخدام هذه الدالة بكامل إمكانياتها لبناء نظام محاسبي قوي ومرن.



## التوثيق الإضافي

**الوثائق المرجعية**:
- [توثيق كلاس `AccountHelper`](./Docs-AccountHelper-ar.md)
- [توثيق كلاس `PersonHelper`](./Docs-PersonHelper-ar.md)
- [توثيق كلاس `TransactionsHelper`](./Docs-TransactionsHelper-ar.md)
- [توثيق متقدم لـ `TransactionsHelper`](./Docs-TransactionsHelper-Advanced-ar.md)
- [فهرس دوال `TransactionsHelper`](./Docs-TransactionsHelper-Reference-Function-ar.md)
- [أمثلة عملية متقدمة لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)
