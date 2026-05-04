# توثيق دوال معالجة فروق العملات: `checkExchangeDifferences` و `createExchangeDifferenceEntry`

## نظرة عامة

تمثل فروق العملات (Exchange Differences) الفروقات المالية الناتجة عن تغير أسعار الصرف بين تاريخ إجراء القيد المحاسبي وتاريخ ترحيله (أو أي تاريخ مقارنة آخر). يوفر النظام المحاسبي في نانوسوفت أداتين أساسيتين للتعامل مع هذه الفروقات:

- **`checkExchangeDifferences`**: دالة عامة لفحص قيد موجود، حساب الفروقات لكل تفصيل بعملة غير رئيسية، واتخاذ إجراء (إنشاء قيد فروق، إطلاق حدث، أو إرجاع المعلومات).
- **`createExchangeDifferenceEntry`**: دالة مساعدة عامة تنشئ قيداً محاسبياً واحداً (طرفين) لمعالجة فارق صرف محدد، وتربطه بالقيد الأصلي.

كلتا الدالتين موجودتان في السمة `TransactionsMultiHelper`، وبالتالي يمكن استدعاؤهما بشكل ثابت من `TransactionsHelper`.

---

## 1. دالة `checkExchangeDifferences`

### توقيع الدالة

```php
public static function checkExchangeDifferences($input, $options = []): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$input` | `int` أو `TransactionHeader` | معرف القيد المطلوب فحصه، أو كائن `TransactionHeader`. |
| `$options` | `array` | مصفوفة خيارات التحكم في الفحص والإجراء (انظر الجدول أدناه). |

### خيارات `$options`

| # | المفتاح | النوع | القيمة الافتراضية | الوصف |
|---|--------|------|-------------------|-------|
| 1 | `threshold` | `float` | `0.001` | الحد الأدنى للفرق في سعر الصرف الذي يتم اعتباره (يتجاوز التقريب). إذا كان الفرق أقل من أو يساوي هذه القيمة، يتم تجاهله. |
| 2 | `action` | `string` | `'entry'` | الإجراء المطلوب عند وجود فروق: `'entry'` (إنشاء قيد فروق)، `'event'` (إطلاق حدث فقط)، `'none'` (إرجاع معلومات فقط دون أي إجراء). |
| 3 | `force_check` | `bool` | `false` | إذا كان `true`، يتم الفحص حتى لو لم يتوفر تاريخ ترحيل أو كان مساوياً لتاريخ القيد. |
| 4 | `relay_date` | `Carbon`/`string`/`null` | `null` | تاريخ الترحيل (أو المقارنة) الصريح. إذا لم يُمرر، يُستخدم `relay_date_at` من كائن القيد. |
| 5 | `exchange_difference_account` | `string`/`null` | `Config::get('tss.accounts::exchange_difference_account')` | كود حساب فروق الصرف المستخدم في إنشاء القيد (إذا كان `action = 'entry'`). |
| 6 | `entry_options` | `array` | `[]` | خيارات إضافية يتم تمريرها إلى `createJournalEntry` عند إنشاء قيد الفروق (مثل `notes`, `transactions_type`...). |

### القيمة المرجعة

تعيد الدالة مصفوفة بنفس تنسيق `createJournalEntry`، مع إضافة مفتاح `exchange_entry` ومعلومات الفروقات في `data`.

| المفتاح | النوع | الوصف |
|--------|------|-------|
| `code` | `int` | `200` للنجاح، `400` للفشل. |
| `status` | `bool` | حالة العملية. |
| `message` | `string` | رسالة وصفية. |
| `error` | `string`/`null` | رسالة الخطأ. |
| `errors` | `array`/`null` | تفاصيل أخطاء التحقق. |
| `model` | `TransactionHeader`/`null` | كائن القيد الأصلي الذي تم فحصه. |
| `data` | `array` | معلومات الفروقات: `has_differences` (bool), `details` (مصفوفة الفروق), `total_difference` (float), `relay_date`, `entry_date`. |
| `exchange_entry` | `TransactionHeader`/`null` | كائن قيد الفروق إذا تم إنشاؤه (فقط عندما `action = 'entry'` وينجح الإنشاء). |
| `input_data` | `array` | الخيارات المدخلة. |
| `process_data` | `array` | البيانات الفعلية المستخدمة. |
| `debug` | `array`/`null` | معلومات التصحيح (في بيئة `debug`). |

### هيكل عنصر `data['details']` (الفروقات)

كل عنصر في مصفوفة `details` يحتوي على:

| المفتاح | النوع | الوصف |
|--------|------|-------|
| `detail_id` | `int` | معرف تفصيل القيد الأصلي. |
| `accounts_id` | `string` | كود حساب التفصيل. |
| `currencys_id` | `string` | كود العملة الأجنبية. |
| `original_rate` | `float` | سعر الصرف المستخدم في القيد الأصلي. |
| `official_rate` | `float` | سعر الصرف الرسمي في تاريخ الترحيل. |
| `rate_difference` | `float` | فرق السعر (`official_rate - original_rate`). |
| `alien_amount` | `float` | المبلغ بالعملة الأجنبية (`debit_alien` أو `credit_alien`). |
| `difference` | `float` | قيمة الفرق المالي (`alien_amount * rate_difference`). |
| `person_id` | `int`/`null` | معرف الشخص المرتبط بالتفصيل. |
| `person_type` | `string`/`null` | نوع الشخص. |

### تدفق تنفيذ الدالة

1.  **تحميل القيد**: إذا كان `$input` رقماً، يتم تحميل `TransactionHeader` مع العلاقة `TransactionDetail`. إذا كان كائناً، يتم التأكد من تحميل التفاصيل.
2.  **دمج الخيارات الافتراضية**: مع الخيارات المُمررة.
3.  **تحديد تاريخ الترحيل (`$relayDate`)**: يُستخدم `options['relay_date']` أو `$header->relay_date_at`. إذا لم يوجد وكان `force_check = false`، يتم إرجاع نتيجة بعدم وجود فروق.
4.  **التكرار على تفاصيل القيد**: لكل تفصيل حيث `currencys_id != main_currencys_id`:
    - جلب السعر الرسمي للعملة في تاريخ الترحيل باستخدام `CurrencyHelper::getRate`.
    - حساب فرق السعر (`$rateDifference = $officialRate - $originalRate`).
    - إذا تجاوز الفرق `threshold`، يتم حساب الفرق المالي (`$difference = $alienAmount * $rateDifference`) وإضافته إلى قائمة الفروقات.
5.  **بناء وإرجاع معلومات الفروقات** في `data`.
6.  **تنفيذ الإجراء** بناءً على `action`:
    - `'none'`: إرجاع المعلومات فوراً.
    - `'event'`: إطلاق الحدث `tss.accounts.exchangeDifference` وإرجاع المعلومات.
    - `'entry'`: بناء خيارات قيد فروق جامع (`$entryOptions`) يضم كل الفروقات (كل فرق ينتج عنه طرفين: حساب الفروقات والحساب الأصلي). استدعاء `createJournalEntry` بهذه الخيارات. في حالة النجاح، يتم وضع القيد الناتج في `exchange_entry`.
7.  **إرجاع النتيجة النهائية**.

### أمثلة

#### مثال 1: فحص فروق قيد وطباعة النتيجة بدون إنشاء

```php
$result = TransactionsHelper::checkExchangeDifferences(123, [
    'action' => 'none',
]);
print_r($result['data']);
```

#### مثال 2: فحص فروق قيد مع إنشاء قيد فروق تلقائي

```php
$result = TransactionsHelper::checkExchangeDifferences(123, [
    'action' => 'entry',
    'threshold' => 0.01,
    'relay_date' => '2026-06-01',
]);

if ($result['exchange_entry']) {
    echo "تم إنشاء قيد فروق رقم: " . $result['exchange_entry']->id;
}
```

#### مثال 3: إطلاق حدث فقط لمعالجة خارجية

```php
TransactionsHelper::checkExchangeDifferences($header, [
    'action' => 'event',
]);
// سيتم الاستماع للحدث tss.accounts.exchangeDifference في مكان آخر
```

---

## 2. دالة `createExchangeDifferenceEntry`

تقوم بإنشاء قيد محاسبي بسيط (طرفين) يمثل تسوية فارق صرف واحد. تُستخدم عادة داخلياً من `handleExchangeDifferences` (بعد إنشاء قيد) أو يمكن استدعاؤها مباشرة.

### توقيع الدالة

```php
public static function createExchangeDifferenceEntry(
    TransactionHeader $originalHeader,
    array $originalDetail,
    float $difference,
    float $officialRate,
    array $originalOptions = []
): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$originalHeader` | `TransactionHeader` | كائن رأس القيد الأصلي الذي نتج عنه الفرق. |
| `$originalDetail` | `array` | بيانات التفصيل الأصلي بعد المعالجة (نفس بنية عنصر `details` المُعالجة). |
| `$difference` | `float` | قيمة الفرق المالي (موجب أو سالب). |
| `$officialRate` | `float` | سعر الصرف الرسمي في تاريخ الترحيل. |
| `$originalOptions` | `array` | الخيارات الأصلية المُستخدمة لإنشاء القيد الأصلي (اختياري). |

### القيمة المرجعة

مصفوفة بنفس تنسيق `createJournalEntry`، حيث `model` هو كائن قيد الفروق المنشأ.

### آلية العمل

1.  **التحقق من وجود حساب فروق الصرف**: يتم قراءة `tss.accounts::exchange_difference_account` من الإعدادات. إذا لم يُعرف، يتم رمي استثناء.
2.  **تجهيز خيارات القيد الجديد**:
    - نسخ القيم التنظيمية من القيد الأصلي (`companys_id`, `departments_id`...).
    - تعيين التاريخ من `relay_date_at` (في `$originalOptions`) أو تاريخ ترحيل القيد الأصلي أو الوقت الحالي.
    - تعيين ملاحظات افتراضية تصف القيد الأصلي ومبلغ الفرق.
    - ربط القيد الجديد بالأصلي عبر `modul_type = 'exchange_difference'`، `extend_id = originalHeader->id`، و `batch_type/id`.
    - تعطيل قواعد فحص الرصيد وتعدد الأطراف.
3.  **بناء تفاصيل القيد (طرفين)**:
    - **الطرف الأول: حساب فروق الصرف**:
        - إذا كان الفرق موجباً (القيد الأصلي سجل قيمة أقل من الواقع بالعملة الرئيسية) ← **مدين**.
        - إذا كان الفرق سالباً (القيد الأصلي سجل قيمة أعلى) ← **دائن**.
        - المبلغ = القيمة المطلقة للفرق.
    - **الطرف الثاني: نفس الحساب الأصلي (عكس الإشارة)**:
        - إذا كان الطرف الأول مديناً ← الحساب الأصلي **دائن**.
        - إذا كان الطرف الأول دائناً ← الحساب الأصلي **مدين**.
        - يتم نسخ `person_id`, `person_type`, `custom_name_accounts` من التفصيل الأصلي.
    - العملة الأساسية لجميع الأطراف هي العملة الرئيسية.
4.  **استدعاء `mergeDefaultCompanyValues`** ثم **`createJournalEntry`** لإنشاء القيد وحفظه.
5.  **إرجاع نتيجة `createJournalEntry`**.

### مثال

```php
// إنشاء قيد فروق مباشر بعد حساب الفرق
$diff = 15.75; // قيمة موجبة (القيد الأصلي أقل من الواقع)
$result = TransactionsHelper::createExchangeDifferenceEntry(
    $originalHeader,
    $processedDetail,
    $diff,
    3.75, // السعر الرسمي
    $originalCreateOptions
);

if ($result['status']) {
    echo "تم إنشاء قيد فروق الصرف رقم: " . $result['model']->id;
}
```

### ملاحظات حول الإشارة والمبلغ

- **الفرق الموجب** (`$difference > 0`): القيمة المسجلة في القيد الأصلي أقل من القيمة الفعلية بالعملة الرئيسية. لذا نحتاج إلى **زيادة** رصيد حساب فروق الصرف (مدين) وتقليل الحساب الأصلي (دائن).
- **الفرق السالب** (`$difference < 0`): القيمة المسجلة أكبر من الفعلية، لذا نخفض حساب فروق الصرف (دائن) ونزيد الحساب الأصلي (مدين).

---

## 3. العلاقة بين الدالتين ومعالجة الفروق الآلية

- عند إنشاء قيد عبر `createJournalEntry` مع الخيار `handle_exchange_differences = true`، يتم استدعاء الدالة المحمية `handleExchangeDifferences` تلقائياً بعد حفظ القيد. هذه الدالة بدورها تقوم بفحص التفاصيل واستدعاء `createExchangeDifferenceEntry` لكل فرق يتجاوز العتبة (إذا كان `exchange_difference_action = 'entry'`).
- يمكن للمطور أيضاً استخدام `checkExchangeDifferences` بشكل مستقل لفحص أي قيد في أي وقت (مثلاً عبر مهام مجدولة أو أوامر كونسول).
- الحدث `tss.accounts.exchangeDifference` يُطلق سواء تم إنشاء قيد أم لا (إذا كان `action = 'event'`)، ويمكن للمطورين الاستماع له لتنفيذ إجراءات مخصصة.

---

## 4. الأخطاء الشائعة واستكشاف الأخطاء

1.  **"الحساب غير معرف" (`account_not_defined`)**: تأكد من تعيين `tss.accounts::exchange_difference_account` في ملف الإعدادات.
2.  **عدم وجود فروق**: تحقق من أن `relay_date` مختلف عن `date_at`، وأن القيود تحتوي على عملات أجنبية. استخدم `force_check = true` لتجاوز شروط التاريخ.
3.  **فشل إنشاء قيد الفروق**: تحقق من `entry_options` المُمررة، فقد تتعارض بعض القواعد (مثل حدود المبالغ) مع قيود الفروق. يمكنك إضافة `disabled_rules` إلى `entry_options` لتجاوزها.

---

## 5. أفضل الممارسات

- **فحص دوري**: قم بجدولة استدعاء `checkExchangeDifferences` على القيود غير المرحلة في نهاية كل فترة.
- **استخدام العتبة (threshold)**: لتجنب إنشاء قيود بقروش بسبب تقريب سعر الصرف.
- **الربط بالقيود الأصلية**: استفد من `batch_id` و `extend_id` لتتبع قيود الفروق المرتبطة بقيد أصلي محدد.
- **اختبار السيناريوهات**: استخدم `testJournalEntry` مع خيارات تحاكي وجود فروق للتأكد من منطق عملك.
- **تجنب الحلقات غير الضرورية**: إذا كنت تفحص عدداً كبيراً من القيود، تأكد من تحميل العلاقة `TransactionDetail` مسبقاً (`with('TransactionDetail')`) لتجنب مشكلة N+1.

---

## 6. سيناريوهات عملية متكاملة

### سيناريو 1: معالجة فروق العملات لجميع القيود غير المرحلة نهاية الشهر

```php
$unrelayedHeaders = TransactionHeader::with('TransactionDetail')
    ->where('is_relay', false)
    ->whereHas('TransactionDetail', function($q) {
        $q->whereColumn('currencys_id', '!=', 'main_currencys_id');
    })
    ->get();

foreach ($unrelayedHeaders as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'    => 'entry',
        'relay_date'=> now()->endOfMonth(),
        'threshold' => 0.01,
    ]);

    if ($result['exchange_entry']) {
        Log::info("تم إنشاء قيد فروق للقيد {$header->id}");
    }
}
```

### سيناريو 2: استخدام حدث لمراجعة قبل الإنشاء

```php
Event::listen('tss.accounts.exchangeDifference', function ($payload) {
    $header     = $payload['header'];
    $differences= $payload['differences'];
    $total      = $payload['total_difference'];

    // يمكن إرسال إشعار للمراجع المالي
    if (abs($total) > 1000) {
        Notification::send($auditor, new LargeExchangeDifferenceFound($header, $total));
    }
});

// ثم فحص مع action = 'event'
TransactionsHelper::checkExchangeDifferences($header, ['action' => 'event']);
```

---

## 7. خاتمة

توفر دالتا `checkExchangeDifferences` و `createExchangeDifferenceEntry` حلاً متكاملاً ومرناً لإدارة فروق العملات في النظام المحاسبي. من خلال الجمع بين الفحص الدقيق، العتبات القابلة للتخصيص، والإجراءات المتعددة (إنشاء، إطلاق حدث، أو الإرجاع فقط)، يمكن للمطورين أتمتة معالجة فروق الصرف أو دمجها مع سير عمل يدوي للمراجعة حسب الحاجة. الاستخدام الصحيح لهاتين الدالتين يضمن دقة الحسابات واكتمال القيود المحاسبية في بيئة متعددة العملات.

بذلك نكون قد أكملنا توثيق دوال فروق العملات بشكل شامل. يمكن للمطور الآن الرجوع إلى هذا الدليل لفهم آلية عمل كل خيار والاستفادة من الأمثلة العملية لتطبيقها في مشاريعه.


