# قائمة الإعدادات الكاملة لدالة `createJournalEntry`

يتم تمرير هذه الإعدادات كمصفوفة `$options` إلى الدالة `createJournalEntry`. جميع المفاتيح اختيارية ولها قيم افتراضية كما هي معرفة في دالة `mergeDefaultCompanyValues`.

---

## 1. حقول الرأس الأساسية (تمرر إلى `AccountHelper::getObjHeader`)

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `companys_id` | int/string | `null` (يُحسب لاحقاً) | معرف الشركة |
| `departments_id` | int/string | `null` (يُحسب لاحقاً) | معرف الإدارة (الفرع) |
| `cost_centers_id` | int/string | `null` (يُحسب لاحقاً) | معرف مركز التكلفة |
| `periods_id` | int/string | `null` (يُحسب لاحقاً) | معرف الفترة المحاسبية |
| `paper_id` | mixed | `null` | رقم المستند الورقي (يمكن أن يكون نصياً أو رقماً) |
| `employees_id` | int | `null` | معرف الموظف المسؤول |
| `date_at` | Carbon/string | `Carbon::now()` | تاريخ القيد (يتم ضبطه حسب إعدادات `is_custome_date` و `date_at_auto`) |
| `transactions_type` | int | `4` | نوع الحركة (من جدول `transactions_types`) |
| `process_type` | string | `'4'` | نوع العملية (مثل "cash") |
| `patterns_id` | int | `4` | معرف النمط المحاسبي (ConstraintsPattern) |
| `notes` | string | `null` | ملاحظات عامة على رأس القيد |
| `default_notes` | string | `null` | ملاحظة افتراضية إذا لم تُعطى `notes` |
| `type_header` | string | `null` | نوع رأس القيد (اسم الكلاس، مثل `BondsDay::class`) |
| `status` | string | `'active'` | حالة القيد |
| `is_active` | bool | `true` | هل القيد نشط؟ |
| `is_relay` | bool | `false` | هل تم ترحيله؟ |
| `user_relay_id` | int | `null` | معرف المستخدم الذي قام بالترحيل |
| `relay_date_at` | Carbon | `null` | تاريخ الترحيل |

---

## 2. الشخص على مستوى الرأس (Header Person)

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `header_person_id` | int | `null` | معرف الشخص المرتبط بالرأس |
| `header_person_type` | string | `null` | نوع الشخص (Morph Class) |
| `header_person` | object | `null` | كائن الشخص (يمكن تمريره بدلاً من id/type) |

---

## 3. إعدادات `paper_id`

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `is_paper_id` | bool | `false` | هل الحقل مستخدم في النموذج؟ |
| `paper_id_allowed` | bool | `true` | هل يسمح للمستخدم بإدخال قيمة؟ |
| `paper_id_auto` | bool | `false` | هل يتم توليد قيمة تلقائياً إذا لم تُدخل؟ |
| `paper_id_required` | bool | `false` | هل الحقل إلزامي؟ |
| `paper_id_check_duplicate` | bool | `true` | هل نتحقق من عدم التكرار؟ |
| `paper_id_duplicate_scope` | array | `[]` | نطاق التحقق من التكرار: يمكن أن يحتوي على `'departments_id'`, `'periods_id'`, `'transactions_type'` |
| `validat_paper_id` | string | `'no_rapet'` | (للتوافق) طريقة التحقق من التكرار |
| `type_rapet` | array | `[]` | (للتوافق) شروط التحقق القديمة |

---

## 4. إعدادات `date_at`

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `is_custome_date` | bool | `false` | هل يسمح للمستخدم بإدخال التاريخ؟ (إذا `false` يتم تعيينه تلقائياً) |
| `date_at_auto` | bool | `false` | هل يتم توليد التاريخ تلقائياً إذا لم يُدخل؟ |
| `date_at_required` | bool | `false` | هل التاريخ إلزامي؟ |

---

## 5. إعدادات العملات وفروق الصرف

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `main_currencys_id` | string | `null` (يُحسب لاحقاً) | العملة الأساسية |
| `currencys_id` | string | `null` (يُحسب لاحقاً) | عملة القيد |
| `rate` | float | `1` | سعر الصرف |
| `auto_convert_currency` | bool | `false` | تحويل المبالغ تلقائياً من العملة الأجنبية إلى الرئيسية |
| `handle_exchange_differences` | bool | `false` | تفعيل معالجة فروق الصرف بعد إنشاء القيد |
| `exchange_difference_action` | string | `'entry'` | الإجراء عند وجود فروق: `'entry'` (إنشاء قيد)، `'event'` (إطلاق حدث)، `'none'` (تجاهل) |

---

## 6. قيود عدد الأطراف (Multiplicity Controls)

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `max_debit_entries` | int/null | `null` | الحد الأقصى لعدد الأطراف المدينة (null = غير محدود) |
| `max_credit_entries` | int/null | `null` | الحد الأقصى لعدد الأطراف الدائنة (null = غير محدود) |
| `allow_multiple_debit` | bool | `true` | السماح بأكثر من طرف مدين |
| `allow_multiple_credit` | bool | `true` | السماح بأكثر من طرف دائن |

---

## 7. إعدادات الشخص (Person) لكل جانب

### الجانب المدين (Debit)

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `debit_person_allowed` | bool | `true` | السماح بربط شخص بالجانب المدين |
| `debit_person_required` | bool | `false` | إلزامية وجود شخص في الجانب المدين |
| `debit_person_allowed_types` | array/null | `null` | أنواع الأشخاص المسموح بها (مصفوفة من أسماء الكلاسات) |

### الجانب الدائن (Credit)

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `credit_person_allowed` | bool | `true` | السماح بربط شخص بالجانب الدائن |
| `credit_person_required` | bool | `false` | إلزامية وجود شخص في الجانب الدائن |
| `credit_person_allowed_types` | array/null | `null` | أنواع الأشخاص المسموح بها |

---

## 8. إعدادات الحساب (Account) لكل جانب

### الجانب المدين

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `debit_account_allowed` | bool | `true` | السماح بتحديد حساب مدين |
| `debit_account_required` | bool | `true` | إلزامية تحديد حساب مدين |
| `debit_default_account` | string/null | `null` | الحساب الافتراضي للمدين (كود الحساب) |

### الجانب الدائن

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `credit_account_allowed` | bool | `true` | السماح بتحديد حساب دائن |
| `credit_account_required` | bool | `true` | إلزامية تحديد حساب دائن |
| `credit_default_account` | string/null | `null` | الحساب الافتراضي للدائن (كود الحساب) |

---

## 9. إعدادات الملاحظات (Notes) لكل جانب

### الجانب المدين

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `debit_notes_allowed` | bool | `true` | السماح بإدخال ملاحظة في الجانب المدين |
| `debit_notes_required` | bool | `false` | إلزامية وجود ملاحظة في الجانب المدين |
| `debit_default_notes` | string/null | `null` | الملاحظة الافتراضية للمدين |

### الجانب الدائن

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `credit_notes_allowed` | bool | `true` | السماح بإدخال ملاحظة في الجانب الدائن |
| `credit_notes_required` | bool | `false` | إلزامية وجود ملاحظة في الجانب الدائن |
| `credit_default_notes` | string/null | `null` | الملاحظة الافتراضية للدائن |

---

## 10. حدود المبالغ (Amount Limits)

### المبلغ الإجمالي

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `min_total_amount` | float/null | `null` | الحد الأدنى للمبلغ الإجمالي (مجموع المدين = مجموع الدائن) |
| `max_total_amount` | float/null | `null` | الحد الأقصى للمبلغ الإجمالي |
| `check_min_total` | bool | `true` | تفعيل فحص الحد الأدنى للمبلغ الإجمالي |
| `check_max_total` | bool | `true` | تفعيل فحص الحد الأقصى للمبلغ الإجمالي |

### الجانب المدين

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `min_debit_amount` | float/null | `null` | الحد الأدنى لإجمالي الجانب المدين |
| `max_debit_amount` | float/null | `null` | الحد الأقصى لإجمالي الجانب المدين |
| `check_debit_min` | bool | `false` | تفعيل فحص الحد الأدنى للمدين |
| `check_debit_max` | bool | `false` | تفعيل فحص الحد الأقصى للمدين |

### الجانب الدائن

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `min_credit_amount` | float/null | `null` | الحد الأدنى لإجمالي الجانب الدائن |
| `max_credit_amount` | float/null | `null` | الحد الأقصى لإجمالي الجانب الدائن |
| `check_credit_min` | bool | `false` | تفعيل فحص الحد الأدنى للدائن |
| `check_credit_max` | bool | `false` | تفعيل فحص الحد الأقصى للدائن |

---

## 11. فحص الرصيد (Balance Check) لكل جانب

### الجانب المدين

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `debit_check_balance` | bool | `false` | تفعيل فحص الرصيد قبل الخصم من المدين |
| `debit_balance_check_callback` | callable/null | `null` | دالة مخصصة لفحص الرصيد في المدين (تتجاوز الفحص الافتراضي) |

### الجانب الدائن

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `credit_check_balance` | bool | `false` | تفعيل فحص الرصيد قبل الإضافة للدائن |
| `credit_balance_check_callback` | callable/null | `null` | دالة مخصصة لفحص الرصيد في الدائن |

---

## 12. فحص نوع الحساب بقواعد ديناميكية (Account Type Rules) لكل جانب

### الجانب المدين

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `debit_check_account_types` | bool | `false` | تفعيل فحص نوع الحساب في المدين باستخدام القواعد |
| `debit_rules_account_types` | array/null | `null` | قواعد التحقق من نوع الحساب في المدين (انظر الشرح أدناه) |

### الجانب الدائن

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `credit_check_account_types` | bool | `false` | تفعيل فحص نوع الحساب في الدائن باستخدام القواعد |
| `credit_rules_account_types` | array/null | `null` | قواعد التحقق من نوع الحساب في الدائن |

---

## 13. الحقول الإضافية (Extra Fields)

تنطبق هذه الإعدادات على الحقول: `extend_id`, `modul_type`, `ref_type`, `ref_key_name`, `ref_key_values`, `batch_type`, `batch_id`.

لكل حقل من هذه الحقول، هناك ثلاث مجموعات من الإعدادات:

### على مستوى الرأس (Header)

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `header_{field}_allowed` | bool | `true` | السماح باستخدام الحقل في الرأس |
| `header_{field}_required` | bool | `false` | إلزامية وجود الحقل في الرأس |
| `header_{field}_default` | mixed | `null` | القيمة الافتراضية للحقل في الرأس |

### على مستوى الجانب المدين (Debit Detail)

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `debit_{field}_allowed` | bool | `true` | السماح باستخدام الحقل في الجانب المدين |
| `debit_{field}_required` | bool | `false` | إلزامية وجود الحقل في الجانب المدين |
| `debit_{field}_default` | mixed | `null` | القيمة الافتراضية للحقل في الجانب المدين |

### على مستوى الجانب الدائن (Credit Detail)

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `credit_{field}_allowed` | bool | `true` | السماح باستخدام الحقل في الجانب الدائن |
| `credit_{field}_required` | bool | `false` | إلزامية وجود الحقل في الجانب الدائن |
| `credit_{field}_default` | mixed | `null` | القيمة الافتراضية للحقل في الجانب الدائن |

**قائمة الحقول:**
- `extend_id`
- `modul_type`
- `ref_type`
- `ref_key_name`
- `ref_key_values`
- `batch_type`
- `batch_id`

---

## 14. إعدادات إضافية (Hooks & Disabled Rules)

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `auto_generate_custom_name` | bool | `true` | توليد اسم مخصص تلقائياً للشخص إذا لم يُمرر |
| `beforeValidate` | callable/null | `null` | دالة تُنفذ قبل التحقق من صحة البيانات (تستقبل `$options` و `$details`) |
| `afterValidate` | callable/null | `null` | دالة تُنفذ بعد التحقق من صحة البيانات (تستقبل `$options` و `$details`) |
| `disabled_rules` | array | `[]` | مصفوفة بأسماء القواعد التي يتم تعطيلها (مثل `['balance_check', 'multiplicity']`) |

---

## 15. تفاصيل القيد (Details)

هذا المفتاح إجباري ويجب تمريره لإنشاء القيد.

| المفتاح | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `details` | array | `[]` | مصفوفة من عناصر التفاصيل، كل عنصر هو مصفوفة تمثل طرفاً محاسبياً. |

### هيكل عنصر `details` الواحد:

| المفتاح | النوع | الوصف |
|--------|------|-------|
| `accounts_id` | string | كود الحساب (إجباري) |
| `debit` | float | المبلغ المدين (يجب أن يكون > 0 إذا كان هذا الطرف مديناً، وإلا 0) |
| `credit` | float | المبلغ الدائن (يجب أن يكون > 0 إذا كان هذا الطرف دائناً، وإلا 0) |
| `notes` | string | ملاحظة خاصة بهذا الطرف (اختياري) |
| `person_id` | int | معرف الشخص المرتبط (اختياري) |
| `person_type` | string | نوع الشخص (Morph Class) (اختياري) |
| `custom_name_accounts` | string | اسم مخصص للحساب (اختياري، يتم توليده تلقائياً إذا كان مفعلاً) |
| `currencys_id` | string | عملة هذا الطرف (إذا اختلفت عن عملة الرأس) |
| `main_currencys_id` | string | العملة الرئيسية (عادة تُترك لتُحدد تلقائياً) |
| `rate` | float | سعر الصرف لهذا الطرف (إذا اختلف عن سعر الرأس) |
| `extend_id` | mixed | (اختياري) |
| `modul_type` | mixed | (اختياري) |
| `ref_type` | mixed | (اختياري) |
| `ref_key_name` | mixed | (اختياري) |
| `ref_key_values` | mixed | (اختياري) |
| `batch_type` | mixed | (اختياري) |
| `batch_id` | mixed | (اختياري) |
| `person` | object | (اختياري) يمكن تمرير كائن الشخص بدلاً من id/type |

---

## شرح `debit_rules_account_types` و `credit_rules_account_types`

يمكن أن تأتي القواعد بصيغتين:

### صيغة مبسطة (AND تلقائي)
```php
$rules = [
    ['field' => 'reports', 'operator' => '=', 'value' => '1'],
    ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
];
```

### صيغة متقدمة مع `operator`
```php
$rules = [
    'operator' => 'OR',   // أو 'AND'
    'rules'    => [
        ['field' => 'reports', 'operator' => '=', 'value' => '1'],
        ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
    ],
];
```

### الحقول المدعومة من جدول `accounts`:
- `reports`, `final_account_id`, `modul_type`, `account_normal`, `groups_accounts_id`, `ref_type`, `person_type`, `account_number`, `parent_id`, `level`, `main_sub`, وغيرها.

### المشغلات المدعومة:
- `=`, `==`, `!=`, `<>`, `>`, `>=`, `<`, `<=`, `LIKE`, `NOT LIKE`, `IN`, `NOT IN`, `REGEXP`.

---

## أمثلة على `balance_check_callback`

دالة callback تستقبل المعاملات التالية:
- `$account`: كائن `Account`
- `$amount`: المبلغ المطلوب
- `$balance`: الرصيد الحالي
- `$personId`: معرف الشخص (إن وجد)
- `$personType`: نوع الشخص (إن وجد)
- `$context`: الخيارات العامة للقيد

**مثال:**
```php
'debit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    if ($balance < $amount) {
        throw new ApplicationException("رصيد الحساب غير كافٍ");
    }
}
```

---

## ملاحظات ختامية

- القيم الافتراضية لمعرفات الشركة والفرع ومركز التكلفة والفترة يتم جلبها تلقائياً من الدوال المساعدة (`getCheckCompanysId`, `getCheckDepartmentsId`، إلخ) بناءً على سياق المستخدم أو الإعدادات العامة.
- إذا تم تفعيل `use_default_company` في إعدادات الإضافة، سيتم تجاوز القيم المدخلة بالقيم الافتراضية للشركة.
- يمكن تمرير أي من هذه الخيارات بشكل اختياري، وسيتم دمجها مع القيم الافتراضية المعرفة في `mergeDefaultCompanyValues`.