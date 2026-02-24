# قائمة الإعدادات الكاملة لدالة `createJournalEntry` (الإصدار الموحد الشامل - محدث)

يتم تمرير هذه الإعدادات كمصفوفة `$options` إلى الدالة `createJournalEntry`. جميع المفاتيح اختيارية ولها قيم افتراضية كما هي معرفة في دالة `mergeDefaultCompanyValues`. القائمة أدناه تمثل جميع المفاتيح الممكنة مع شرح لكل منها، وهي مستخلصة من الملفين المرجعيين وتجمع كل الخيارات المتاحة، مع تحديث القيم الافتراضية لتطابق الكود الفعلي.

```php
<?php

return [
    /*
    |--------------------------------------------------------------------------
    | الإعدادات الافتراضية لإنشاء القيود المحاسبية (createJournalEntry)
    |--------------------------------------------------------------------------
    |
    | تستخدم هذه الإعدادات كقيم افتراضية عند إنشاء أي قيد محاسبي عبر الدالة
    | createJournalEntry. يمكن تخصيصها لكل نوع مستند (مثلاً BondsDay, CatchReceipt)
    | عبر ملف الإعدادات الخاص بذلك النوع.
    |
    */
    'journal_entry_defaults' => [

        // ----------------------------------------------------------------------
        // 1. الإعدادات الأساسية للرأس (تمرر إلى AccountHelper::getObjHeader)
        // ----------------------------------------------------------------------
        'periods_id'          => null,   // معرف الفترة المحاسبية (يُحدد لاحقاً)
        'companys_id'         => null,   // معرف الشركة (يُحدد لاحقاً)
        'departments_id'      => null,   // معرف الإدارة (الفرع) – يُحدد لاحقاً
        'cost_centers_id'     => null,   // معرف مركز التكلفة
        'paper_id'            => null,   // معرف المستند الورقي المرتبط (اختياري)
        'employees_id'        => null,   // معرف الموظف المسؤول
        'date_at'             => null,   // تاريخ القيد (كائن Carbon أو نص، إن لم يحدد يستخدم الوقت الحالي)
        'transactions_type'   => 4,      // نوع الحركة (رقم من جدول tss_accounts_transactions_types)
        'process_type'        => 4,      // نوع العملية (رقم أو نص، مثل "cash" أو "credit") - الافتراضي 4
        'patterns_id'         => 4,      // معرف النمط المحاسبي (ConstraintsPattern)
        'notes'               => null,   // ملاحظات عامة على رأس القيد
        'status'              => 'active', // حالة القيد (active, inactive, ...)
        'is_active'           => true,   // هل القيد نشط؟
        'is_relay'            => false,  // هل تم ترحيله؟
        'user_relay_id'       => null,   // معرف المستخدم الذي قام بالترحيل
        'relay_date_at'       => null,   // تاريخ الترحيل
        'main_currencys_id'   => null,   // العملة الأساسية للشركة (تُحدد لاحقاً)
        'currencys_id'        => null,   // عملة القيد (تُحدد لاحقاً)
        'rate'                => 1,      // سعر الصرف (رقم عشري)
        'type_header'         => null,   // نوع رأس القيد (اسم الكلاس، مثل BondsDay::class)
        'default_notes'       => null,   // ملاحظة افتراضية للرأس إذا لم تقدم

        // ----------------------------------------------------------------------
        // 2. الشخص على مستوى الرأس (Header Person)
        // ----------------------------------------------------------------------
        'header_person_id'    => null,   // معرف الشخص المرتبط بالرأس
        'header_person_type'  => null,   // نوع الشخص (Morph Class)
        'header_person'       => null,   // كائن الشخص (يمكن تمريره بدلاً من id/type)

        // ----------------------------------------------------------------------
        // 3. إعدادات paper_id (المستند الورقي)
        // ----------------------------------------------------------------------
        'is_paper_id'               => false,   // هل الحقل مستخدم في النموذج؟
        'paper_id_allowed'          => true,    // هل يسمح للمستخدم بإدخال قيمة؟
        'paper_id_auto'             => false,   // هل يتم توليد قيمة تلقائيًا إذا لم تُدخل؟
        'paper_id_required'         => false,   // هل الحقل إلزامي؟
        'paper_id_check_duplicate'  => true,    // هل نتحقق من عدم التكرار؟
        'paper_id_duplicate_scope'  => [],      // نطاق التحقق من التكرار (مصفوفة تحتوي على departments_id, periods_id, transactions_type)
        'validat_paper_id'          => 'no_rapet', // طريقة التحقق من التكرار (للتوافق مع الإصدارات القديمة)
        'type_rapet'                => [],      // شروط التحقق من التكرار القديمة

        // ----------------------------------------------------------------------
        // 4. إعدادات date_at (التاريخ)
        // ----------------------------------------------------------------------
        'is_custome_date'       => false,   // هل يسمح للمستخدم بإدخال التاريخ؟ (إذا false يتم تعيينه تلقائياً)
        'date_at_auto'          => false,   // هل يتم توليد التاريخ تلقائياً إذا لم يُدخل؟
        'date_at_required'      => false,   // هل التاريخ إلزامي؟

        // ----------------------------------------------------------------------
        // 5. إعدادات العملات وفروق الصرف
        // ----------------------------------------------------------------------
        'auto_convert_currency'         => false,   // تحويل المبالغ تلقائياً من العملة الأجنبية إلى الرئيسية
        'handle_exchange_differences'   => false,   // تفعيل معالجة فروق الصرف بعد إنشاء القيد
        'exchange_difference_action'    => 'event', // الإجراء عند وجود فروق: 'entry' (إنشاء قيد)، 'event' (إطلاق حدث)، 'none' (تجاهل)

        // ----------------------------------------------------------------------
        // 6. تفاصيل القيد (إجباري)
        // ----------------------------------------------------------------------
        'details'             => [],     // مصفوفة تحتوي على تفاصيل القيد (كل عنصر طرف)

        // ----------------------------------------------------------------------
        // 7. قيود عدد الأطراف (لكل جانب)
        // ----------------------------------------------------------------------
        'max_debit_entries'   => null,   // الحد الأقصى لعدد الأطراف المدينة (null = غير محدود)
        'max_credit_entries'  => null,   // الحد الأقصى لعدد الأطراف الدائنة (null = غير محدود)
        'allow_multiple_debit'  => true, // السماح بأكثر من طرف مدين (true/false)
        'allow_multiple_credit' => true, // السماح بأكثر من طرف دائن (true/false)

        // ----------------------------------------------------------------------
        // 8. إعدادات الشخص (person) – الجانب المدين
        // ----------------------------------------------------------------------
        'debit_person_allowed'        => true,   // السماح بربط شخص بالجانب المدين
        'debit_person_required'       => false,  // إلزامية وجود شخص في الجانب المدين
        'debit_person_allowed_types'  => null,   // أنواع الأشخاص المسموح بها (مصفوفة، مثل [User::class])

        // ----------------------------------------------------------------------
        // 9. إعدادات الشخص – الجانب الدائن
        // ----------------------------------------------------------------------
        'credit_person_allowed'       => true,   // السماح بربط شخص بالجانب الدائن
        'credit_person_required'      => false,  // إلزامية وجود شخص في الجانب الدائن
        'credit_person_allowed_types' => null,   // أنواع الأشخاص المسموح بها

        // ----------------------------------------------------------------------
        // 10. إعدادات الحساب – الجانب المدين
        // ----------------------------------------------------------------------
        'debit_account_allowed'   => true,   // السماح بتحديد حساب مدين
        'debit_account_required'  => true,   // إلزامية تحديد حساب مدين
        'debit_default_account'   => null,   // الحساب الافتراضي للمدين (كود الحساب)

        // ----------------------------------------------------------------------
        // 11. إعدادات الحساب – الجانب الدائن
        // ----------------------------------------------------------------------
        'credit_account_allowed'  => true,   // السماح بتحديد حساب دائن
        'credit_account_required' => true,   // إلزامية تحديد حساب دائن
        'credit_default_account'  => null,   // الحساب الافتراضي للدائن (كود الحساب)

        // ----------------------------------------------------------------------
        // 12. إعدادات الملاحظات – الجانب المدين
        // ----------------------------------------------------------------------
        'debit_notes_allowed'     => true,   // السماح بإدخال ملاحظة في الجانب المدين
        'debit_notes_required'    => false,  // إلزامية وجود ملاحظة في الجانب المدين
        'debit_default_notes'     => null,   // الملاحظة الافتراضية للمدين

        // ----------------------------------------------------------------------
        // 13. إعدادات الملاحظات – الجانب الدائن
        // ----------------------------------------------------------------------
        'credit_notes_allowed'    => true,   // السماح بإدخال ملاحظة في الجانب الدائن
        'credit_notes_required'   => false,  // إلزامية وجود ملاحظة في الجانب الدائن
        'credit_default_notes'    => null,   // الملاحظة الافتراضية للدائن

        // ----------------------------------------------------------------------
        // 14. حدود المبلغ الإجمالي
        // ----------------------------------------------------------------------
        'min_total_amount'        => null,   // الحد الأدنى للمبلغ الإجمالي (مجموع المدين = مجموع الدائن)
        'max_total_amount'        => null,   // الحد الأقصى للمبلغ الإجمالي
        'check_min_total'         => true,   // تفعيل فحص الحد الأدنى للمبلغ الإجمالي
        'check_max_total'         => true,   // تفعيل فحص الحد الأقصى للمبلغ الإجمالي

        // ----------------------------------------------------------------------
        // 15. حدود المبلغ – الجانب المدين
        // ----------------------------------------------------------------------
        'check_debit_min'         => false,  // تفعيل فحص الحد الأدنى لإجمالي الجانب المدين
        'check_debit_max'         => false,  // تفعيل فحص الحد الأقصى لإجمالي الجانب المدين
        'min_debit_amount'        => null,   // الحد الأدنى لإجمالي الجانب المدين
        'max_debit_amount'        => null,   // الحد الأقصى لإجمالي الجانب المدين

        // ----------------------------------------------------------------------
        // 16. حدود المبلغ – الجانب الدائن
        // ----------------------------------------------------------------------
        'check_credit_min'        => false,  // تفعيل فحص الحد الأدنى لإجمالي الجانب الدائن
        'check_credit_max'        => false,  // تفعيل فحص الحد الأقصى لإجمالي الجانب الدائن
        'min_credit_amount'       => null,   // الحد الأدنى لإجمالي الجانب الدائن
        'max_credit_amount'       => null,   // الحد الأقصى لإجمالي الجانب الدائن

        // ----------------------------------------------------------------------
        // 17. فحص الرصيد – الجانب المدين
        // ----------------------------------------------------------------------
        'debit_check_balance'          => false,   // تفعيل فحص الرصيد قبل الخصم من المدين
        'debit_balance_check_callback' => null,    // دالة مخصصة لفحص الرصيد (تتجاوز الفحص الافتراضي)

        // ----------------------------------------------------------------------
        // 18. فحص الرصيد – الجانب الدائن
        // ----------------------------------------------------------------------
        'credit_check_balance'         => false,   // تفعيل فحص الرصيد قبل الإضافة للدائن
        'credit_balance_check_callback'=> null,    // دالة مخصصة لفحص الرصيد

        // ----------------------------------------------------------------------
        // 19. فحص نوع الحساب بقواعد ديناميكية – الجانب المدين
        // ----------------------------------------------------------------------
        'debit_check_account_types'     => false,   // تفعيل فحص نوع الحساب في المدين
        'debit_rules_account_types'     => null,    // مصفوفة قواعد للتحقق من نوع الحساب (انظر الشرح)

        // ----------------------------------------------------------------------
        // 20. فحص نوع الحساب – الجانب الدائن
        // ----------------------------------------------------------------------
        'credit_check_account_types'    => false,   // تفعيل فحص نوع الحساب في الدائن
        'credit_rules_account_types'    => null,    // مصفوفة قواعد للتحقق من نوع الحساب

        // ----------------------------------------------------------------------
        // 21. الحقول الإضافية في رأس القيد (header)
        // ----------------------------------------------------------------------
        'header_extend_id_allowed'      => true,    // السماح باستخدام extend_id في الرأس
        'header_extend_id_required'     => false,   // إلزامية وجود extend_id في الرأس
        'header_extend_id_default'      => null,    // القيمة الافتراضية لـ extend_id في الرأس

        'header_modul_type_allowed'     => true,    // السماح باستخدام modul_type في الرأس
        'header_modul_type_required'    => false,   // إلزامية وجود modul_type في الرأس
        'header_modul_type_default'     => null,    // القيمة الافتراضية لـ modul_type في الرأس

        'header_ref_type_allowed'       => true,    // السماح باستخدام ref_type في الرأس
        'header_ref_type_required'      => false,   // إلزامية وجود ref_type في الرأس
        'header_ref_type_default'       => null,    // القيمة الافتراضية لـ ref_type في الرأس

        'header_ref_key_name_allowed'   => true,    // السماح باستخدام ref_key_name في الرأس
        'header_ref_key_name_required'  => false,   // إلزامية وجود ref_key_name في الرأس
        'header_ref_key_name_default'   => null,    // القيمة الافتراضية لـ ref_key_name في الرأس

        'header_ref_key_values_allowed' => true,    // السماح باستخدام ref_key_values في الرأس
        'header_ref_key_values_required'=> false,   // إلزامية وجود ref_key_values في الرأس
        'header_ref_key_values_default' => null,    // القيمة الافتراضية لـ ref_key_values في الرأس

        'header_batch_type_allowed'     => true,    // السماح باستخدام batch_type في الرأس
        'header_batch_type_required'    => false,   // إلزامية وجود batch_type في الرأس
        'header_batch_type_default'     => null,    // القيمة الافتراضية لـ batch_type في الرأس

        'header_batch_id_allowed'       => true,    // السماح باستخدام batch_id في الرأس
        'header_batch_id_required'      => false,   // إلزامية وجود batch_id في الرأس
        'header_batch_id_default'       => null,    // القيمة الافتراضية لـ batch_id في الرأس

        // ----------------------------------------------------------------------
        // 22. الحقول الإضافية في تفاصيل الجانب المدين (debit details)
        // ----------------------------------------------------------------------
        'debit_extend_id_allowed'       => true,
        'debit_extend_id_required'      => false,
        'debit_extend_id_default'       => null,

        'debit_modul_type_allowed'      => true,
        'debit_modul_type_required'     => false,
        'debit_modul_type_default'      => null,

        'debit_ref_type_allowed'        => true,
        'debit_ref_type_required'       => false,
        'debit_ref_type_default'        => null,

        'debit_ref_key_name_allowed'    => true,
        'debit_ref_key_name_required'   => false,
        'debit_ref_key_name_default'    => null,

        'debit_ref_key_values_allowed'  => true,
        'debit_ref_key_values_required' => false,
        'debit_ref_key_values_default'  => null,

        'debit_batch_type_allowed'      => true,
        'debit_batch_type_required'     => false,
        'debit_batch_type_default'      => null,

        'debit_batch_id_allowed'        => true,
        'debit_batch_id_required'       => false,
        'debit_batch_id_default'        => null,

        // ----------------------------------------------------------------------
        // 23. الحقول الإضافية في تفاصيل الجانب الدائن (credit details)
        // ----------------------------------------------------------------------
        'credit_extend_id_allowed'       => true,
        'credit_extend_id_required'      => false,
        'credit_extend_id_default'       => null,

        'credit_modul_type_allowed'      => true,
        'credit_modul_type_required'     => false,
        'credit_modul_type_default'      => null,

        'credit_ref_type_allowed'        => true,
        'credit_ref_type_required'       => false,
        'credit_ref_type_default'        => null,

        'credit_ref_key_name_allowed'    => true,
        'credit_ref_key_name_required'   => false,
        'credit_ref_key_name_default'    => null,

        'credit_ref_key_values_allowed'  => true,
        'credit_ref_key_values_required' => false,
        'credit_ref_key_values_default'  => null,

        'credit_batch_type_allowed'      => true,
        'credit_batch_type_required'     => false,
        'credit_batch_type_default'      => null,

        'credit_batch_id_allowed'        => true,
        'credit_batch_id_required'       => false,
        'credit_batch_id_default'        => null,

        // ----------------------------------------------------------------------
        // 24. إعدادات إضافية (Hooks وتعطيل القواعد)
        // ----------------------------------------------------------------------
        'auto_generate_custom_name' => true,   // توليد اسم مخصص تلقائياً للشخص إذا لم يُمرر
        'beforeValidate'            => null,   // دالة تُنفذ قبل التحقق من صحة البيانات (تستقبل $options و $details)
        'afterValidate'             => null,   // دالة تُنفذ بعد التحقق من صحة البيانات (تستقبل $options و $details)
        'disabled_rules'            => [],     // مصفوفة بأسماء القواعد التي يتم تعطيلها (مثل ['balance_check', 'multiplicity'])

        // ----------------------------------------------------------------------
        // 25. ملاحظة: يمكن تمرير `id` في بعض السياقات (للاستثناء عند التحقق من التكرار)
        // ----------------------------------------------------------------------
    ],
];
```

---

## شرح مفصل لـ `debit_rules_account_types` (ونظيره للدائن `credit_rules_account_types`)

يمكن أن يأتي هذا الخيار بصيغتين:

### صيغة مبسطة (مصفوفة قواعد مع عامل AND تلقائي)
```php
$rules = [
    ['field' => 'reports', 'operator' => '=', 'value' => '1'],
    ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
];
```

### صيغة متقدمة مع تحديد العامل المنطقي
```php
$rules = [
    'operator' => 'OR',   // أو 'AND'
    'rules'    => [
        ['field' => 'reports', 'operator' => '=', 'value' => '1'],
        ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
        ['field' => 'account_number', 'operator' => 'REGEXP', 'value' => '/^1253/'],
    ],
];
```

### الحقول المدعومة من جدول `accounts`:
- `reports` (1: ميزانية، 2: أرباح وخسائر)
- `final_account_id`
- `modul_type` (مثل "Boxe", "Suppler", "Customer")
- `account_normal` (0 مدين، 1 دائن)
- `groups_accounts_id`
- `ref_type`
- `person_type`
- `account_number`
- `parent_id`
- `level`
- `main_sub` (main / sub)
- وغيرها من أعمدة جدول `accounts`.

### المشغلات المدعومة:
- `=`, `==`, `!=`, `<>`
- `>`, `>=`, `<`, `<=`
- `LIKE`, `NOT LIKE`
- `IN`, `NOT IN` (مع مصفوفة)
- `REGEXP` (تعبير نمطي)

### أمثلة متنوعة على `debit_rules_account_types`:

```php
// السماح فقط بحسابات الخزينة (modul_type = 'Boxe')
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    ['field' => 'modul_type', 'operator' => '=', 'value' => 'Boxe']
]

// السماح بحسابات العملاء أو الموردين (ref_type = 14 للعملاء، 13 للموردين)
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    'operator' => 'OR',
    'rules' => [
        ['field' => 'ref_type', 'operator' => '=', 'value' => '14'],
        ['field' => 'ref_type', 'operator' => '=', 'value' => '13']
    ]
]

// حسابات الأصول (reports = 1) وليست رئيسية (main_sub = 'sub')
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    ['field' => 'reports', 'operator' => '=', 'value' => '1'],
    ['field' => 'main_sub', 'operator' => '=', 'value' => 'sub']
]

// حسابات تبدأ بادئة معينة في account_number (مثلاً 1253 للخزائن)
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    ['field' => 'account_number', 'operator' => 'LIKE', 'value' => '1253%']
]
// أو باستخدام REGEXP:
// ['field' => 'account_number', 'operator' => 'REGEXP', 'value' => '/^1253/']

// حسابات من مجموعة معينة (groups_accounts_id = 5) أو ذات طبيعة مدينة (account_normal = 0)
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    'operator' => 'OR',
    'rules' => [
        ['field' => 'groups_accounts_id', 'operator' => '=', 'value' => '5'],
        ['field' => 'account_normal', 'operator' => '=', 'value' => '0']
    ]
]

// حسابات ليس نوعها 'Suppler' (أي ليست حسابات موردين)
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    ['field' => 'modul_type', 'operator' => '!=', 'value' => 'Suppler']
]

// قواعد معقدة باستخدام IN مع قيم متعددة (مثلاً modul_type في ['Boxe', 'Bank'])
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Bank']]
]
```

---

## شرح مفصل لـ `debit_balance_check_callback` و `credit_balance_check_callback`

هذه دوال مخصصة لفحص الرصيد قبل تنفيذ الحركة، وتتجاوز الفحص الافتراضي. تستقبل الدالة الوسائط التالية:
- `$account`: كائن الحساب (Account)
- `$amount`: المبلغ المطلوب
- `$balance`: الرصيد الحالي للحساب (يُحسب وفقاً للسياق)
- `$personId`: معرف الشخص المرتبط (قد يكون null)
- `$personType`: نوع الشخص (Morph Class)
- `$context`: مصفوفة تحتوي على سياق إضافي (مثل `transactions_type`, `process_type`، إلخ)

### أمثلة:

```php
// دالة بسيطة تطبع تحذيراً في السجل ولا تمنع العملية
'debit_check_balance' => true,
'debit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    if ($balance < $amount) {
        Log::warning("رصيد الحساب {$account->code} غير كافٍ: الرصيد {$balance}، المطلوب {$amount}");
    }
    return true; // السماح بالعملية مع تسجيل تحذير
}

// دالة تسمح بالسحب حتى حد ائتماني معين (10,000)
'credit_check_balance' => true,
'credit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    $creditLimit = 10000; // حد ائتماني 10,000
    if ($balance + $creditLimit < $amount) {
        throw new ApplicationException("تجاوز الحد الائتماني للحساب {$account->code}");
    }
    return true;
}

// دالة تستخدم معلومات الشخص لتحديد الحد المسموح (إذا كان الشخص موجوداً)
'debit_check_balance' => true,
'debit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    if ($personId && $personType) {
        $person = \Tss\Accounts\Helpers\PersonHelper::getPersonObj($personType, $personId);
        if ($person && property_exists($person, 'credit_limit') && $person->credit_limit) {
            if ($balance + $person->credit_limit < $amount) {
                throw new ApplicationException("الحد الائتماني للشخص غير كافٍ");
            }
        }
    }
    // الفحص الافتراضي
    if ($balance < $amount) {
        throw new ApplicationException("رصيد الحساب غير كافٍ");
    }
    return true;
}

// دالة تسمح بالسحب فقط إذا كان المبلغ أقل من 50,000 بغض النظر عن الرصيد (لحسابات معينة)
'credit_check_balance' => true,
'credit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    // إذا كان الحساب من نوع الخزينة، نسمح بمبالغ كبيرة
    if ($account->modul_type == 'Boxe') {
        if ($amount > 100000) {
            throw new ApplicationException("المبلغ يتجاوز الحد المسموح للخزينة");
        }
        return true;
    }
    // باقي الحسابات تخضع للفحص العادي
    if ($balance < $amount) {
        throw new ApplicationException("رصيد الحساب غير كافٍ");
    }
    return true;
}

// دالة تستخدم سياق إضافي (مثل نوع الحركة) لتحديد سلوك الفحص
'credit_check_balance' => true,
'credit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    // إذا كانت الحركة من نوع 'cash' نسمح بمرونة أكبر
    if (($context['transactions_type'] ?? null) == 2) { // 2 = سند قبض مثلاً
        if ($balance + 5000 < $amount) {
            throw new ApplicationException("رصيد غير كافٍ حتى مع المرونة");
        }
    } else {
        if ($balance < $amount) {
            throw new ApplicationException("رصيد الحساب غير كافٍ");
        }
    }
    return true;
}
```

---

## هيكل تنظيمي بديل (باستخدام المفاتيح المتداخلة)

يمكن أيضاً تنظيم الإعدادات بشكل متداخل لتجميع الخيارات المتعلقة بكل جانب، وهذا الهيكل مدعوم كبديل. المثال أدناه يوضح هذا التنظيم:

```php
'journal_entry_defaults' => [
    // الإعدادات الأساسية للرأس (كما هي)
    'periods_id' => null,
    // ... باقي إعدادات الرأس

    // إعدادات الجانب المدين
    'debit' => [
        'max_entries'           => null,
        'allow_multiple'        => true,
        'person_allowed'        => true,
        'person_required'       => false,
        'person_allowed_types'  => null,
        'account_allowed'       => true,
        'account_required'      => true,
        'default_account'       => null,
        'notes_allowed'         => true,
        'notes_required'        => false,
        'default_notes'         => null,
        'check_balance'         => false,
        'balance_check_callback'=> null,
        'check_account_types'   => false,
        'account_type_rules'    => null,
        'extend_id_allowed'     => true,
        // ... باقي الحقول الإضافية للجانب المدين
    ],

    // إعدادات الجانب الدائن (بنفس الهيكل)
    'credit' => [ ... ],

    // إعدادات رأس القيد للحقول الإضافية
    'header' => [ ... ],

    // حدود المبالغ
    'amount_limits' => [
        'total' => ['min' => null, 'max' => null, 'check_min' => true, 'check_max' => true],
        'debit' => ['min' => null, 'max' => null, 'check_min' => false, 'check_max' => false],
        'credit' => ['min' => null, 'max' => null, 'check_min' => false, 'check_max' => false],
    ],
];
```

يمكنك استخدام أي من الهيكلين (المسطح أو المتداخل) حسب تفضيلك، على أن يتم دعمهما في الكود.

---

## ملاحظات ختامية

- القيم الافتراضية لمعرفات الشركة والفرع ومركز التكلفة والفترة يتم جلبها تلقائياً من الدوال المساعدة (`getCheckCompanysId`, `getCheckDepartmentsId`، إلخ) بناءً على سياق المستخدم أو الإعدادات العامة.
- إذا تم تفعيل `use_default_company` في إعدادات الإضافة، سيتم تجاوز القيم المدخلة بالقيم الافتراضية للشركة.
- يمكن تمرير أي من هذه الخيارات بشكل اختياري، وسيتم دمجها مع القيم الافتراضية المعرفة في `mergeDefaultCompanyValues`.
- حقل `disabled_rules` يمكن استخدامه لتعطيل قواعد تحقق معينة، مثل `'balance_check'` أو `'multiplicity'` أو `'account_type_rules'`.
- الدوال `beforeValidate` و `afterValidate` تتيح لك تنفيذ كود مخصص قبل وبعد عملية التحقق، وهي تستقبل `$options` و `$details` (بعد معالجتها) ويمكنها تعديلها أو إضافة تحقق إضافي.
- في بعض السياقات (مثل التحديث)، يمكن تمرير مفتاح `id` لاستبعاد القيد الحالي عند التحقق من تكرار `paper_id`.