# توثيق الأمثلة المتقدمة والعملية لكلاس `NflowService`

**Namespace:** `Nano3\TelecomRecharge\Classes\Services`  
**الهدف:** تقديم أمثلة تطبيقية شاملة لاستخدام `NflowService` في سيناريوهات متنوعة، من العمليات الأساسية إلى المتقدمة، مع توضيح المدخلات والمخرجات المتوقعة، وأفضل الممارسات.

---

## مقدمة

يقدم هذا الوثيقة مجموعة من الأمثلة العملية التي تغطي مختلف حالات استخدام كلاس `NflowService`، بدءًا من أبسط عمليات الاستعلام والشحن وصولًا إلى استخدام الدوال العامة المرنة، وإدارة Webhook، والتحقق من صحة البيانات، ومعالجة الأخطاء.

> **ملاحظة:** الأمثلة هنا تتعامل مباشرة مع الكلاس `NflowService`. إذا كنت تستخدم متحكم `NflowController`، فإن الاستجابة ستكون موحدة وفق الهيكل الموصوف في [توثيق API](./Nflow-API-Documentation-ar.md). الكلاس نفسه يعيد مصفوفة تحتوي على `success`, `status_code`, `data`, `error`, `message`.

كل مثال يحتوي على:

- **السيناريو**: شرح مختصر للهدف.
- **الكود**: الكود المستخدم لتنفيذ العملية.
- **المدخلات**: تفصيل الخيارات الممرة.
- **المخرجات المتوقعة**: شكل النتيجة التي تعيدها الدالة، مع شرح تأثيرها على النظام.
- **ملاحظات**: توضيحات حول حالات خاصة أو نصائح.

---

## المتطلبات الأساسية

- إضافة `Nano3.TelecomRecharge` مثبتة.
- بيانات اعتماد صالحة من [nflow.tech](https://nflow.tech) (اسم المستخدم، كلمة المرور، رقم الحساب، رمز API).
- الاتصال بالإنترنت للوصول إلى واجهة API.

### إعداد الكلاس في Laravel / October CMS

أضف بيانات الاعتماد في ملف `.env`:

```ini
NANO3_TELECOM_RECHARGE_NFLOW_BASE_URL=https://nflow.tech
NANO3_TELECOM_RECHARGE_NFLOW_USERNAME=your_username
NANO3_TELECOM_RECHARGE_NFLOW_PASSWORD=your_password
NANO3_TELECOM_RECHARGE_NFLOW_ACCOUNT_NUMBER=your_account_number
NANO3_TELECOM_RECHARGE_NFLOW_API_TOKEN=your_api_token

NANO3_TELECOM_RECHARGE_NFLOW_LOGIN_ENDPOINT='/rest-api/login'
```

ثم قم بإنشاء الكائن:

```php
use Nano3\TelecomRecharge\Classes\Services\NflowService;

$api = new NflowService();
$login = $api->login();
if (!$login['success']) {
    die('فشل تسجيل الدخول: ' . json_encode($login));
}
```

**ملاحظة:** في جميع الأمثلة، نفترض أن المتغير `$api` هو كائن جاهز مع تسجيل دخول ناجح.

---

## 1. العمليات الأساسية (استعلامات الوكيل والعمليات العامة)

### 1.1 استعلام رصيد الوكيل

**السيناريو:** التحقق من الرصيد الحالي في حسابك.

```php
$balance = $api->getAccountBalance();
if ($balance['success']) {
    echo "الرصيد الحالي: " . $balance['data']['agentBalance'] . " ريال يمني";
} else {
    echo "خطأ: " . $balance['error'];
}
```

**المخرجات المتوقعة (مثال):**

```php
[
    'success' => true,
    'status_code' => 200,
    'data' => [
        'status' => true,
        'agentBalance' => 1500.50,
        'message' => 'Agent Balance Query Success',
        'transactionID' => 0
    ]
]
```

### 1.2 استعلام حالة عملية سابقة

**السيناريو:** متابعة حالة معاملة أرسلت مسبقًا باستخدام معرف المعاملة الخاص بك.

```php
$status = $api->getOperationStatus('TXN_12345');
if ($status['success']) {
    $data = $status['data'];
    $statusText = $api->getOperationStatusText($data['operationStatus']);
    echo "حالة المعاملة: $statusText\n";
    if ($data['operationStatus'] == 1) {
        echo "تمت العملية بنجاح. رقم المرجع: " . $data['referenceID'];
    }
}
```

**المخرجات المتوقعة (في حالة النجاح):**

```php
[
    'success' => true,
    'data' => [
        'status' => true,
        'operationStatus' => 1,
        'mobileNumber' => '777777777',
        'price' => 100,
        'message' => 'جاهزة',
        'transactionID' => 'TXN_12345',
        'referenceID' => 98765
    ]
]
```

### 1.3 الحصول على قائمة إيداعات الوكيل

**السيناريو:** عرض الإيداعات التي تمت في حسابك (مثل إضافة رصيد).

```php
$feed = $api->getFeedClientsBalance();
if ($feed['success']) {
    foreach ($feed['data']['data'] as $voucher) {
        echo "التاريخ: {$voucher['Date']} - المبلغ: {$voucher['Amount']} {$voucher['Currency']}\n";
    }
}
```

---

## 2. استعلامات الرصيد للشبكات المختلفة

### 2.1 استعلام رصيد يمن موبايل

**السيناريو:** معرفة رصيد خط عميل يمن موبايل.

```php
$result = $api->queryYemenMobileBalance('777777777');
if ($result['success']) {
    $data = $result['data'];
    echo "الرصيد: {$data['mobileBalance']} ريال\n";
    echo "نوع الخط: {$data['mobileTypeName']}\n";
} else {
    echo "فشل الاستعلام: " . $result['error'];
}
```

**المخرجات المتوقعة (مثال):**

```php
[
    'success' => true,
    'data' => [
        'status' => true,
        'mobileBalance' => 154.15,
        'mobileType' => 1,
        'mobileTypeName' => 'دفع مسبق',
        'message' => 'Balance Query Success'
    ]
]
```

### 2.2 استعلام رصيد ADSL (خط إنترنت أرضي)

**السيناريو:** معرفة البيانات المتبقية من باقة ADSL.

```php
$result = $api->queryAdslBalance('12345678');
if ($result['success']) {
    $data = $result['data'];
    echo "الرصيد المتبقي: {$data['mobileBalance']}\n";
    echo "تاريخ انتهاء الباقة: {$data['expiredDate']}\n";
}
```

**المخرجات المتوقعة:**

```php
[
    'success' => true,
    'data' => [
        'status' => true,
        'mobileBalance' => '44.18 جيجابايت',
        'expiredDate' => '21/10/2024',
        'offerAmount' => '12600',
        'message' => 'Balance Query Success'
    ]
]
```

### 2.3 استعلام رصيد هاتف أرضي

```php
$result = $api->queryLandPhoneBalance('012345678');
if ($result['success']) {
    echo "الرصيد: {$result['data']['mobileBalance']} ريال";
}
```

### 2.4 استعلام رصيد Yemen 4G

```php
$result = $api->queryYemen4GBalance('777777777');
if ($result['success']) {
    $data = $result['data'];
    echo "الرصيد: {$data['mobileBalance']}\n";
    echo "تاريخ الانتهاء: {$data['expiredDate']}\n";
}
```

### 2.5 استعلام رصيد عدن نت (Aden Net)

```php
$result = $api->queryAdenNetBalance('770000000');
if ($result['success']) {
    echo "الرصيد: {$result['data']['mobileBalance']} ريال";
}
```

### 2.6 استعلام قرض يمن موبايل

**السيناريو:** التحقق من إمكانية الحصول على قرض (سلفة) للعميل.

```php
$result = $api->queryYemenMobileLoan('777777777');
if ($result['success']) {
    $data = $result['data'];
    echo "حالة القرض: {$data['loanStatusString']}\n";
}
```

**المخرجات المتوقعة:**

```php
[
    'success' => true,
    'data' => [
        'status' => true,
        'loanStatus' => false,
        'loanStatusString' => 'الرقم غير متسلف',
        'message' => 'Balance Query Success'
    ]
]
```

### 2.7 استعلام باقات يمن موبايل

**السيناريو:** الحصول على قائمة الباقات المشترك بها العميل.

```php
$result = $api->queryYemenMobileOffers('777777777');
if ($result['success']) {
    foreach ($result['data']['data'] as $offer) {
        echo "الباقة: {$offer['offerName']} (من {$offer['offerStartDate']} إلى {$offer['offerEndDate']})\n";
    }
}
```

---

## 3. عمليات الشحن (إضافة رصيد)

### 3.1 شحن رصيد يمن موبايل باستخدام الدالة العامة `chargeBalanceGeneric`

**السيناريو:** إضافة 500 ريال لرقم يمن موبايل.

```php
$txnId = 'TXN_' . uniqid();
$result = $api->chargeBalanceGeneric(1, '777777777', 500, $txnId);
if ($result['success']) {
    echo "تم شحن الرصيد بنجاح.\n";
    echo "الرصيد المتبقي في حسابك: " . $result['data']['agentBalance'];
    echo "رقم المرجع: " . $result['data']['referenceID'];
} else {
    echo "فشل الشحن: " . $result['error'];
}
```

**المخرجات المتوقعة:**

```php
[
    'success' => true,
    'data' => [
        'status' => true,
        'operationStatus' => 1,
        'agentBalance' => 1500.50,
        'price' => 500,
        'message' => 'Payment success',
        'transactionID' => 'TXN_xxx',
        'referenceID' => 123456
    ]
]
```

### 3.2 شحن رصيد سبافون (بوحدات)

**السيناريو:** شحن 5 وحدات لرقم سبافون (يتم ضربها بسعر الوحدة في النظام).

```php
$result = $api->chargeBalanceGeneric(3, '777777777', 5, 'TXN_' . uniqid());
```

**ملاحظات:**  
- الشبكات (2) YOU, (3) Sabafon, (4) Y Telecom تستخدم قيمة `Amount` كعدد وحدات، وليس مبلغًا نقديًا مباشرًا.

### 3.3 شحن رصيد ADSL بقيمة 1000 ريال

```php
$result = $api->chargeBalanceGeneric(5, '12345678', 1000, 'TXN_' . uniqid());
```

### 3.4 شحن رصيد هاتف أرضي

```php
$result = $api->chargeBalanceGeneric(6, '012345678', 500, 'TXN_' . uniqid());
```

### 3.5 شحن رصيد باستخدام الدالة الأصلية `chargeBalance` (لتحديد رقم الخدمة يدويًا)

**السيناريو:** شحن رصيد يمن موبايل (ServiceNumber = 103) مع تمرير WebHook.

```php
$result = $api->chargeBalance(
    1, 103, '777777777', 200, 'TXN_789',
    'https://your-site.com/webhook', 'SECURE_CODE'
);
```

---

## 4. تفعيل الباقات (Offers)

### 4.1 تفعيل باقة يمن موبايل (باستخدام الدالة العامة)

**السيناريو:** تفعيل باقة معينة لرقم يمن موبايل باستخدام كود الباقة.

```php
$result = $api->activateOfferGeneric(1, '777777777', 'OFFER_CODE', 'TXN_' . uniqid());
if ($result['success']) {
    echo "تم تفعيل الباقة بنجاح.";
}
```

**ملاحظات:**  
- كود الباقة (`OfferCode`) يجب الحصول عليه من النظام (غالبًا يكون معروفًا أو يتم استلامه من استعلام الباقات).
- لشبكة يمن موبايل، خدمة تفعيل الباقات هي `104` (Re New Yemen Mobile Offer)، لكن الدالة العامة تستخدم `107` للاستعلام وليس التفعيل. انتبه: حسب الوثيقة، خدمة `104` هي `Re New Yemen Mobile Offer` (تجديد باقة)، و`107` هي `Query Yemen Mobile Offers`. إذا كنت تريد تفعيل باقة جديدة، قد تحتاج إلى استخدام `109` (New Yemen Mobile Offer). لذلك في الدالة `activateOfferGeneric` افترضنا أن رقم الخدمة للباقات هو `202` لشبكة YOU و `303` لسبافون و `402` لواي تيليكوم و `704` لـ Yemen 4G و `1202` لسبافون ساوث. بالنسبة ليمن موبايل، نستخدم `104` في الدالة المخصصة. لذا يفضل استخدام الدوال المختصرة إن وجدت.

### 4.2 تفعيل باقة يمن موبايل باستخدام الدالة المخصصة

```php
$result = $api->activateOffer(1, 104, '777777777', 'OFFER_CODE', 'TXN_' . uniqid());
```

### 4.3 تفعيل باقة YOU Telecom

```php
$result = $api->activateOfferGeneric(2, '777777777', 'PREWhatsApp', 'TXN_' . uniqid());
```

### 4.4 تفعيل باقة سبافون

```php
$result = $api->activateOfferGeneric(3, '777777777', 'SABAFON_OFFER', 'TXN_' . uniqid());
```

### 4.5 تفعيل باقة سبافون ساوث

```php
$result = $api->activateOfferGeneric(12, '777777777', 'SOUTH_OFFER', 'TXN_' . uniqid());
```

---

## 5. شحن الألعاب وبطاقات الهدايا

### 5.1 جلب فئات الألعاب المتاحة

**السيناريو:** الحصول على قائمة الألعاب المدعومة مع أسعارها والبيانات المطلوبة.

```php
$games = $api->getGamesAndGiftCardsCategories();
if ($games['success']) {
    foreach ($games['data']['data'] as $game) {
        echo "اللعبة: {$game['ServiceName']} - {$game['CategoryName']}\n";
        echo "السعر: {$game['LocalPrice']} {$game['LocalCurrencyName']}\n";
        echo "الحد الأدنى: {$game['MinQuantity']} - الحد الأقصى: {$game['MaxQuantity']}\n";
        echo "الحقول المطلوبة: ";
        foreach ($game['RequiredFields'] as $field) {
            echo "{$field['FieldName']} ({$field['FieldCode']}) ";
        }
        echo "\n\n";
    }
}
```

### 5.2 شحن بوبجي (60 شدة)

**السيناريو:** شحن 60 شدة لـ PUBG باستخدام `PlayerID`.

```php
$fields = ['PlayerID' => '1234567890'];
$result = $api->chargeGameOrGiftCard('pubg_60', $fields, 1, 'TXN_' . uniqid());
if ($result['success']) {
    echo "تم شحن اللعبة بنجاح.";
}
```

### 5.3 شحن يويو شات (كمية حرة)

**السيناريو:** شحن 500 عملة لـ Yoyo Chat (يسمح بالكمية الحرة).

```php
$fields = ['PlayerID' => 'user@example.com']; // أو أي معرف مطلوب
$result = $api->chargeGameOrGiftCard('YoyoChat', $fields, 500, 'TXN_' . uniqid());
```

---

## 6. استخدام الدوال العامة المرنة (`executeService`)

هذه الدالة تسمح بتنفيذ أي خدمة برقم شبكة وخدمة معينين، مما يوفر مرونة كاملة.

### 6.1 تنفيذ خدمة غير مغطاة بدالة مختصرة (مثل إزالة باقة يمن موبايل)

**السيناريو:** إزالة باقة معينة من رقم عميل.

```php
$result = $api->executeService(1, 108, [
    'MobileNumber' => '777777777',
    'OfferCode' => 'OFFER_ID'
], 'TXN_' . uniqid());
```

### 6.2 تنفيذ خدمة شحن رصيد مع WebHook باستخدام `executeService`

```php
$result = $api->executeService(3, 301, [
    'MobileNumber' => '777777777',
    'Amount' => 100,
    'WebHookURL' => 'https://your-site.com/webhook',
    'WebHookCode' => 'SECURE'
], 'TXN_' . uniqid());
```

### 6.3 تنفيذ خدمة استعلام رصيد معينة (مثل استعلام رصيد ADSL) دون استخدام الدالة المختصرة

```php
$result = $api->executeService(5, 501, ['MobileNumber' => '12345678'], 'TXN_' . uniqid());
```

---

## 7. دعم Webhook

### 7.1 إرسال طلب مع WebHook

**السيناريو:** شحن رصيد وإشعار نظامك بتحديثات الحالة عبر WebHook.

```php
$webhookUrl = 'https://your-app.com/api/yepayment/webhook';
$webhookCode = 'SECURE_' . uniqid();

$result = $api->chargeBalanceGeneric(1, '777777777', 100, 'TXN_' . uniqid(), $webhookUrl, $webhookCode);
```

### 7.2 استقبال WebHook في Laravel (مثال على مسار)

```php
Route::get('/api/yepayment/webhook', function (Request $request) {
    $operationStatus = $request->get('OperationStatus'); // 1 نجاح، 0 فشل
    $txnId = $request->get('TransactionID');
    $referenceId = $request->get('ReferenceID');
    $price = $request->get('price');
    $message = $request->get('message');

    // تحديث حالة المعاملة في قاعدة البيانات
    // ...
});
```

---

## 8. التحقق من صحة البيانات باستخدام دوال المساعدة

### 8.1 التحقق من صحة رقم الجوال حسب الشبكة

**السيناريو:** التأكد من أن الرقم المدخل صالح ليمن موبايل قبل إرسال الطلب.

```php
$mobile = '777777777';
if ($api->isValidMobileNumber($mobile, 1)) {
    echo "الرقم صحيح ليمن موبايل.\n";
} else {
    echo "الرقم غير صالح.\n";
}
```

### 8.2 تنسيق رقم الجوال (إزالة الأصفار الزائدة)

```php
$raw = '0 77 7777777';
$formatted = $api->formatMobileNumber($raw); // 777777777
```

### 8.3 الحصول على وصف الشبكة

```php
$networkName = $api->getNetworkName(1); // "Yemen Mobile"
```

### 8.4 التحقق من دعم الشبكة لخدمة معينة

```php
if ($api->supportsBalanceQuery(1)) {
    $api->queryBalance(1, '777777777');
}
```

### 8.5 الحصول على قائمة الخدمات لشبكة معينة

```php
$services = $api->getServicesForNetwork(1);
foreach ($services as $serviceNumber => $description) {
    echo "$serviceNumber: $description\n";
}
```

---

## 9. معالجة الأخطاء المتقدمة

### 9.1 التعامل مع أخطاء محددة حسب كود HTTP

**السيناريو:** التعامل مع رصيد غير كافٍ.

```php
$result = $api->chargeBalanceGeneric(1, '777777777', 1000, 'TXN_' . uniqid());
if (!$result['success']) {
    switch ($result['status_code']) {
        case 402:
            echo "رصيد غير كافٍ. يرجى شحن حسابك.";
            break;
        case 401:
            echo "فشل المصادقة. تأكد من بيانات الاعتماد.";
            break;
        case 404:
            echo "الرقم أو الخدمة غير موجودة.";
            break;
        case 423:
            echo "الخدمة مقفلة مؤقتًا.";
            break;
        default:
            echo "خطأ: " . $result['error'];
    }
}
```

### 9.2 استخدام وضع الاختبار (محاكاة التغيير دون حفظ)

على عكس `PriceHelper`، لا يحتوي `NflowService` على دالة `previewUpdate` مدمجة، ولكن يمكن تحقيق معاينة من خلال تنفيذ عملية واستخدام `is_test` غير موجود. لكن يمكن محاكاة ذلك باستخدام `executeService` مع معاملة وهمية وعدم الاعتماد على النتيجة. **بديل**: يمكنك استخدام `getOperationStatus` بعد العملية لتأكيد نجاحها.

---

## 10. أفضل الممارسات

1. **استخدم الدوال العامة المرنة** (`executeService`, `queryBalance`, `chargeBalanceGeneric`, `activateOfferGeneric`) لتقليل الكود المكرر.
2. **قم بتسجيل جميع المعاملات** في قاعدة البيانات المحلية (معرف المعاملة، الشبكة، المبلغ، الحالة) لتتبع العمليات.
3. **استخدم WebHook** للحصول على تحديثات فورية بدلاً من الاستعلام الدوري.
4. **تحقق من صحة الرقم** قبل إرسال الطلب لتجنب أخطاء 406.
5. **راقب رصيد الوكيل** قبل العمليات الكبيرة لتجنب أخطاء 402.
6. **احتفظ بـ `transactionId` فريدًا** (مثل `TXN_` + uniqid()) لتجنب تكرار المعاملات.
7. **في بيئة التطوير، استخدم بيانات اختبار** لتجنب استهلاك الرصيد.
8. **توثيق الأخطاء** باستخدام `Log::error` في Laravel لتتبع المشاكل.

---

## 11. اختبار الاتصال والتحقق من صحة الإعدادات

**السيناريو:** التأكد من صحة بيانات الاعتماد وإمكانية الاتصال.

```php
$test = $api->testConnection();
if ($test['success']) {
    echo "الاتصال ناجح. التوكن موجود: " . ($test['token'] ? 'نعم' : 'لا');
} else {
    echo "فشل الاتصال: " . $test['error'];
}
```

---

## 12. دمج `NflowService` مع نظام التتبع المحلي (مثال)

**السيناريو:** حفظ كل معاملة شحن في قاعدة بيانات محلية.

```php
use Nano3\TelecomRecharge\Classes\Services\NflowService;

class NflowController extends Controller
{
    public function charge(Request $request)
    {
        $api = new NflowService();
        $login = $api->login();
        if (!$login['success']) {
            return response()->json(['error' => 'فشل تسجيل الدخول'], 500);
        }

        $txnId = 'TXN_' . uniqid();
        $result = $api->chargeBalanceGeneric(
            $request->network,
            $request->mobile,
            $request->amount,
            $txnId,
            route('webhook.yepayment')
        );

        // حفظ في قاعدة البيانات المحلية
        Transaction::create([
            'transaction_id' => $txnId,
            'network' => $request->network,
            'mobile' => $request->mobile,
            'amount' => $request->amount,
            'status' => $result['success'] ? 'pending' : 'failed',
            'response' => json_encode($result)
        ]);

        return response()->json($result);
    }
}
```

---

## الخلاصة

كلاس `NflowService` يوفر مجموعة متكاملة من الأدوات للتعامل مع خدمات nflow.tech، من خلال دوال عامة مرنة ودوال مختصرة سهلة الاستخدام. من خلال الأمثلة المذكورة أعلاه، يمكن للمطورين تنفيذ أي سيناريو يتعلق بشحن الرصيد، استعلام الرصيد، تفعيل الباقات، أو شحن الألعاب، مع مراعاة الأمان والموثوقية. نوصي باستخدام الدوال العامة (`executeService`، `queryBalance`، `chargeBalanceGeneric`، `activateOfferGeneric`) للحصول على أقصى مرونة، والاستفادة من دوال المساعدة للتحقق من صحة البيانات.

للاطلاع على هيكل الاستجابة الموحد عند استخدام متحكم `NflowController`، راجع [توثيق واجهة API](./Nflow-API-Documentation-ar.md).