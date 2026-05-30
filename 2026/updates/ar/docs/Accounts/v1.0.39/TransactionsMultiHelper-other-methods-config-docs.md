**ملف توثيق الدوال المساعدة في `TransactionsMultiHelper`**  

هذا الملف يشرح الدوال المهمة الأخرى في السمة `TransactionsMultiHelper`، خاصة تلك المتعلقة بفروق العملات والتحقق من القيود المحاسبية.

---

## 1. `checkExchangeDifferences`

فحص قيد محاسبي موجود لتحديد ما إذا كان يحتاج إلى قيد فروق عملات، مع إمكانية إنشاء القيد تلقائياً أو إطلاق حدث.

### التوقيع
```php
public static function checkExchangeDifferences($input, $options = [])
```

### المدخلات
| الباراميتر | النوع | الوصف |
|------------|-------|--------|
| `$input` | `int` \| `TransactionHeader` | معرف القيد (رقم) أو كائن القيد نفسه |
| `$options` | `array` | مصفوفة خيارات للتحكم في السلوك |

### خيارات `$options`

| الخيار | النوع | الافتراضي | الوصف |
|--------|------|----------|-------|
| `threshold` | `float` | `0.001` | الحد الأدنى للفرق المالي (قيمة مطلقة) لقبوله كفرق صرف. أي فرق أقل من هذه القيمة يُعتبر مهملاً. |
| `action` | `string` | `'entry'` | الإجراء الذي سيتم اتخاذه عند وجود فروق:<br>`'entry'` – إنشاء قيد فروق باستخدام `createExchangeDifferenceEntry`<br>`'event'` – إطلاق حدث `tss.accounts.exchangeDifference` فقط<br>`'none'` – فقط إرجاع معلومات عن الفروق (لا إنشاء ولا حدث) |
| `force_check` | `bool` | `false` | إذا كان `true`، يتم فحص الفروق حتى لو لم يكن هناك تاريخ ترحيل (`relay_date`) أو كان مساوياً لتاريخ القيد. |
| `relay_date` | `Carbon`\|`null` | `null` | تاريخ محدد للفحص بدلاً من تاريخ الترحيل الموجود في القيد. إذا لم يُمرر، يستخدم `$header->relay_date_at`. |
| `exchange_difference_account` | `string`\|`null` | `Config::get('tss.accounts::exchange_difference_account')` | حساب فروق الصرف الذي سيستخدم إذا كان `action = 'entry'`. إذا لم يُمرر، يستخدم القيمة من ملف الإعدادات. |
| `entry_options` | `array` | `[]` | خيارات إضافية تمرر إلى `createJournalEntry` عند إنشاء قيد الفروق. يمكن استخدامها لتخصيص القيد الناتج (مثل `notes`, `transactions_type`، إلخ). |

### قيمة الإرجاع
مصفوفة بنفس تنسيق `createJournalEntry`:
```php
[
    'code'          => 200,        // كود الحالة (200 للنجاح، 400 للخطأ)
    'status'        => true,       // نجاح العملية
    'message'       => string,     // رسالة توضيحية
    'error'         => null,       // نص الخطأ إن وجد
    'errors'        => null,       // تفاصيل الأخطاء (إن وجدت)
    'input_data'    => [],         // البيانات المدخلة (للتصحيح)
    'process_data'  => [],         // بيانات المعالجة (للتصحيح)
    'model'         => TransactionHeader, // القيد الأصلي
    'data'          => [           // معلومات تفصيلية عن الفروق
        'has_differences' => bool,
        'details'         => array, // مصفوفة من الفروق لكل تفصيل
        'total_difference' => float,
        'relay_date'      => string,
        'entry_date'      => string,
    ],
    'exchange_entry' => TransactionHeader|null // قيد الفروق إذا تم إنشاؤه
]
```

### مثال
```php
$result = TransactionsMultiHelper::checkExchangeDifferences(123, [
    'threshold' => 0.01,
    'action' => 'entry',
    'relay_date' => Carbon::parse('2023-12-31'),
    'entry_options' => [
        'notes' => 'قيد فروق يدوي',
        'transactions_type' => 10,
    ]
]);

if ($result['status']) {
    echo "تم الفحص، إجمالي الفروق: " . $result['data']['total_difference'];
    if ($result['exchange_entry']) {
        echo "تم إنشاء قيد فروق رقم: " . $result['exchange_entry']->id;
    }
}
```

---

## 2. `createExchangeDifferenceEntry`

إنشاء قيد محاسبي خاص بفروق العملات لفارق واحد (عادةً ما يُستدعى من `handleExchangeDifferences`).

### التوقيع
```php
public static function createExchangeDifferenceEntry($originalHeader, $originalDetail, $difference, $officialRate, $originalOptions = [])
```

### المدخلات
| الباراميتر | النوع | الوصف |
|------------|-------|--------|
| `$originalHeader` | `TransactionHeader` | القيد الأصلي الذي نتجت عنه الفروق |
| `$originalDetail` | `array` | بيانات التفصيل الأصلي (بعد المعالجة) الذي تسبب في الفرق |
| `$difference` | `float` | قيمة الفرق (قد تكون موجبة أو سالبة) |
| `$officialRate` | `float` | سعر الصرف الرسمي في تاريخ الترحيل |
| `$originalOptions` | `array` | الخيارات الأصلية التي استخدمت لإنشاء القيد الأصلي (اختياري) |

### خيارات `$originalOptions` (مهمة للقيد الجديد)
- `disabled_rules` – يتم دمجه مع `['balance_check', 'multiplicity']` لتعطيل بعض القواعد
- `relay_date_at` – يستخدم كتاريخ للقيد الجديد إن وجد

### قيمة الإرجاع
نفس نتيجة `createJournalEntry` (مصفوفة تحتوي على `model` وغيره).

### ملاحظات
- تعتمد على وجود حساب فروق صرف معرف في الإعدادات (`exchange_difference_account`).
- تنشئ قيداً بسيطاً من طرفين: حساب الفروق (مدين/دائن) والحساب الأصلي (عكس الإشارة).
- تربط القيد الجديد بالقيد الأصلي عبر `modul_type`, `extend_id`, `batch_type`, `batch_id`.

---

## 3. `validateJournalEntry`

التحقق من صحة بيانات القيد (بدون إنشائه). تستخدم داخلياً في `createJournalEntry`.

### التوقيع
```php
public static function validateJournalEntry($options = [])
```

### المدخلات
نفس خيارات `createJournalEntry` (لأنها تحتاج السياق الكامل).

### قيمة الإرجاع
```php
[
    'details'       => array,  // التفاصيل بعد المعالجة والتحقق
    'header_person' => array|null // بيانات الشخص على الرأس (person_id, person_type)
]
```

### استثناءات
ترمي `ApplicationException` إذا كان هناك خطأ في البيانات (مثل عدم توازن القيد، أو حساب غير موجود، إلخ).

---

## 4. `handleExchangeDifferences`

دالة داخلية تُستدعى من `createJournalEntry` إذا كان `handle_exchange_differences = true`. تقوم بفحص القيد بعد إنشائه ومعالجة فروق الصرف حسب الإعدادات.

### التوقيع
```php
protected static function handleExchangeDifferences($header, $details, $options)
```

### المدخلات
- `$header` – كائن `TransactionHeader` الذي تم إنشاؤه
- `$details` – مصفوفة التفاصيل بعد المعالجة
- `$options` – الخيارات الأصلية للقيد

### الإجراء
- تحدد تاريخ الترحيل (`relay_date`) من الخيارات أو من القيد.
- إذا كان هناك فرق زمني، تفحص كل تفصيل بعملة غير رئيسية.
- تحسب الفرق باستخدام السعر الرسمي في تاريخ الترحيل.
- بناءً على `exchange_difference_action`، إما تنشئ قيداً (`entry`) أو تطلق حدثاً (`event`).

---

## 5. `testJournalEntry`

محاكاة إنشاء قيد دون حفظ في قاعدة البيانات، مع تحليل مفصل للنتيجة.

### التوقيع
```php
public static function testJournalEntry($options = [])
```

### المدخلات
نفس خيارات `createJournalEntry`.

### قيمة الإرجاع
نفس نتيجة `createJournalEntry` مع إضافة مفتاح `analysis`:
```php
[
    // ... حقول result العادية ...
    'analysis' => [
        'header_prepared'    => array,  // الرأس بعد التجهيز
        'details_prepared'   => array,  // التفاصيل بعد المعالجة
        'model_preview'      => array,  // تمثيل للكائن الذي كان سيُنشأ
        'totals'             => array,  // المجاميع (total_debit_base, total_credit_base, ...)
        'currency_analysis'  => array,  // تحليل العملات المستخدمة
        'rule_violations'    => array,  // مخالفات للقواعد إن وجدت
        'warnings'           => array,  // تحذيرات
        'test_passed'        => bool,   // هل اجتاز الاختبار؟
    ],
    'original_options' => $originalOptions // الخيارات الأصلية
]
```

### مثال
```php
$test = TransactionsMultiHelper::testJournalEntry([
    'details' => [...]
]);
if ($test['analysis']['test_passed']) {
    echo "القيد صالح ويمكن إنشاؤه.";
} else {
    print_r($test['analysis']['rule_violations']);
}
```

---

## 6. `processCurrency`

دالة داخلية لمعالجة العملة والـ rate لكل تفصيل.

### التوقيع
```php
protected static function processCurrency($detail, $context)
```

### المدخلات
- `$detail` – مصفوفة بيانات التفصيل
- `$context` – الخيارات العامة (تحتوي على `currencys_id`, `main_currencys_id`, `auto_convert_currency`)

### الإجراء
- تحدد العملة (`currencys_id`) والعملة الرئيسية (`main_currencys_id`) والـ `rate`.
- إذا كان `auto_convert_currency = true` والعملة مختلفة، تقوم بتحويل المبلغ من العملة الأجنبية إلى الرئيسية وتخزن المبلغ الأصلي في `debit_alien`/`credit_alien`.
- تعيد التفصيل بعد تحديث الحقول.

### قيمة الإرجاع
مصفوفة التفصيل بعد المعالجة.

---

## 7. `applyAccountRules`

تطبيق مجموعة من القواعد على كائن الحساب للتحقق من نوعه أو خصائصه.

### التوقيع
```php
protected static function applyAccountRules($account, $rules, $side, $index)
```

### المدخلات
- `$account` – كائن `Account`
- `$rules` – مصفوفة القواعد (انظر الشرح أدناه)
- `$side` – `'debit'` أو `'credit'` (لرسائل الخطأ)
- `$index` – رقم التفصيل (لرسائل الخطأ)

### هيكل `$rules`
يمكن أن يكون إما:
1. **قائمة بسيطة من القواعد** (يتم دمجها بـ `AND`):
   ```php
   [
       ['field' => 'reports', 'operator' => '=', 'value' => '1'],
       ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe','Cash']],
   ]
   ```
2. **مصفوفة مع `operator` و `rules`**:
   ```php
   [
       'operator' => 'OR',
       'rules' => [
           ['field' => 'reports', 'operator' => '=', 'value' => '1'],
           ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe','Cash']],
       ]
   ]
   ```
حيث `operator`可以是 `'AND'` أو `'OR'` (افتراضي `'AND'`).

### العوامل المدعومة
`=`, `==`, `!=`, `<>`, `>`, `>=`, `<`, `<=`, `LIKE`, `NOT LIKE`, `IN`, `NOT IN`, `REGEXP`.

---

## 8. `handleException`

معالجة موحدة للاستثناءات وإرجاع مصفوفة النتيجة مع تفاصيل التصحيح إذا كان `app.debug = true`.

### التوقيع
```php
protected static function handleException($e, $result, $default_msg_error, $options, $is_log = false)
```

### المدخلات
- `$e` – كائن الاستثناء
- `$result` – مصفوفة النتيجة الحالية
- `$default_msg_error` – الرسالة الافتراضية للخطأ
- `$options` – الخيارات الأصلية (تُستخدم للتسجيل)
- `$is_log` – إذا كان `true`، يسجل الخطأ في ملف `Log`

### قيمة الإرجاع
مصفوفة `$result` محدثة.

---

## 9. `fireHook`

إطلاق دالة ربط مخصصة (إذا كانت موجودة في `$options`) وحدث نظام.

### التوقيع
```php
protected static function fireHook($hookName, &$options, &$details = null)
```

### المدخلات
- `$hookName` – اسم الخطاف (مثل `'beforeValidate'`)
- `$options` – الخيارات (تسمح بتعديلها)
- `$details` – التفاصيل (إن وجدت)

### الإجراء
- إذا كان `$options[$hookName]` دالة قابلة للاستدعاء، يتم تنفيذها.
- يتم إطلاق حدث `tss.accounts.$hookName`.

---

## 10. `getMaxPaperId`, `generatePaperId`, `preparePaperId`, `validatePaperId`, `validatePaperIdRequired`

مجموعة دوال مساعدة للتحكم في `paper_id`. راجع توثيق `createJournalEntry` للخيارات المتعلقة بهذه الدوال.

---

## 11. `prepareDateAt`, `validateDateAtRequired`

دوال مساعدة للتحكم في `date_at`. راجع توثيق `createJournalEntry` للخيارات المتعلقة بهذه الدوال.

---
