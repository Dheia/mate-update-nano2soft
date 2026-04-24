# توثيق كلاس `NflowService`

**الإصدار:** 1.0  

**Namespace:** `Nano3\TelecomRecharge\Classes\Services`  
**الهدف:** التكامل مع بوابة الدفع الإلكترونية nflow.tech لإدارة عمليات الشحن الرقمي، واستعلام أرصدة شركات الاتصالات اليمنية، وتفعيل الباقات، وشحن الألعاب وبطاقات الهدايا.

---

## جدول المحتويات

1. [مقدمة](#مقدمة)
2. [المتطلبات والإعداد](#المتطلبات-والإعداد)
   - 2.1 [تسجيل حساب في nflow.tech](#21-تسجيل-حساب-في-nflowtech)
   - 2.2 [إضافة البيانات إلى ملف البيئة](#22-إضافة-البيانات-إلى-ملف-البيئة)
   - 2.3 [استخدام الكلاس بدون Laravel](#23-استخدام-الكلاس-بدون-laravel)
   - 2.4 [تسجيل الدخول قبل أي عملية](#24-تسجيل-الدخول-قبل-أي-عملية)
3. [هيكل الاستجابة](#هيكل-الاستجابة)
4. [العمليات العامة (Global Operations)](#العمليات-العامة-global-operations)
5. [الدوال العامة المرنة (Flexible Generic Methods)](#الدوال-العامة-المرنة-flexible-generic-methods)
   - 5.1 [`executeService`](#51-executeservice)
   - 5.2 [`queryBalance`](#52-querybalance)
   - 5.3 [`chargeBalanceGeneric`](#53-chargebalancegeneric)
   - 5.4 [`activateOfferGeneric`](#54-activateoffergeneric)
6. [دوال الشبكات المحددة (Network-Specific Shortcuts)](#دوال-الشبكات-المحددة-network-specific-shortcuts)
   - 6.1 [استعلامات الرصيد](#61-استعلامات-الرصيد)
   - 6.2 [دوال الشحن الأصلية (للتوافق)](#62-دوال-الشحن-الأصلية-للتوافق)
7. [دوال البيانات والمساعدة](#دوال-البيانات-والمساعدة)
8. [دعم Webhook](#دعم-webhook)
9. [اختبار الاتصال](#اختبار-الاتصال)
10. [معالجة الأخطاء](#معالجة-الأخطاء)
11. [أمثلة تطبيقية](#أمثلة-تطبيقية)
12. [ملخص](#ملخص)

---

## مقدمة

كلاس `NflowService` هو أداة متكاملة تهدف إلى تمكين **التكامل مع منصة nflow.tech** (ماستر هوست) لإدارة جميع خدمات الشحن الرقمي. يوفر الكلاس واجهة موحدة لتنفيذ عمليات مثل:

- استعلام رصيد الوكيل.
- استعلام حالة المعاملة.
- استعلام رصيد العملاء (إيداعات الوكيل).
- جلب فئات الألعاب وبطاقات الهدايا.
- استعلام رصيد شركات الاتصالات (يمن موبايل، ADSL، هاتف أرضي، Yemen 4G، عدن نت).
- شحن رصيد مباشر لأي شبكة (يمن موبايل، سبافون، يو، واي تيليكوم، ADSL، هاتف أرضي، Yemen 4G، سبافون ساوث، عدن نت).
- تفعيل باقات الإنترنت والخدمات (يمن موبايل، يو، سبافون، واي تيليكوم، Yemen 4G، سبافون ساوث).
- شحن الألعاب وبطاقات الهدايا (بوبجي، يويو شات، وغيرها).

يتميز الكلاس بـ:

- **دوال عامة مرنة** لتنفيذ أي خدمة باستخدام رقم الشبكة ورقم الخدمة.
- **دوال مختصرة** لكل شبكة لتسهيل الاستخدام.
- **معالجة احترافية للأخطاء** مع دعم Webhook لتحديثات الحالة.
- **دوال مساعدة** للتحقق من صحة أرقام الجوالات، وتنسيقها، والحصول على قوائم الشبكات والخدمات.
- **دعم كامل لـ Laravel Environment** عبر متغيرات البيئة.

---

## المتطلبات والإعداد

### 2.1 تسجيل حساب في nflow.tech

للحصول على بيانات الاعتماد، يجب التسجيل في الموقع الرسمي:  
[https://nflow.tech](https://nflow.tech)

بعد التسجيل، يمكنك الحصول على:
- **UserName** (اسم المستخدم)
- **Password** (كلمة المرور)
- **AccountNumber** (رقم الحساب)
- **API Token** (رمز API)

> **ملاحظة أمنية:** احرص على عدم مشاركة هذه البيانات مع أي شخص غير مخول، واستخدم HTTPS دائمًا.

### 2.2 إضافة البيانات إلى ملف البيئة (.env)

أضف الأسطر التالية إلى ملف `.env` الخاص بتطبيقك (إذا كنت تستخدم Laravel أو October CMS):

```ini
# YemenRecharge By Nflow API Configuration
NANO3_TELECOM_RECHARGE_NFLOW_BASE_URL=https://nflow.tech
NANO3_TELECOM_RECHARGE_NFLOW_USERNAME=your_username_here
NANO3_TELECOM_RECHARGE_NFLOW_PASSWORD=your_password_here
NANO3_TELECOM_RECHARGE_NFLOW_ACCOUNT_NUMBER=your_account_number_here
NANO3_TELECOM_RECHARGE_NFLOW_API_TOKEN=your_api_token_here
```

### 2.3 استخدام الكلاس بدون Laravel (أو مع إعدادات مخصصة)

يمكن تمرير البيانات مباشرة عند إنشاء الكائن:

```php
$api = new NflowService(
    'https://nflow.tech',
    'your_username',
    'your_password',
    'your_account_number',
    'your_api_token'
);
```

### 2.4 تسجيل الدخول قبل أي عملية

يجب تنفيذ `login()` أولاً للحصول على `accessToken`:

```php
$login = $api->login();
if (!$login['success']) {
    die('فشل تسجيل الدخول: ' . json_encode($login));
}
```

---

## هيكل الاستجابة

جميع الدوال تعيد مصفوفة بنفس الهيكل:

```php
[
    'success'     => bool,      // نجاح الطلب (HTTP 200 والبيانات)
    'status_code' => int,       // كود HTTP
    'data'        => mixed,     // البيانات المستلمة (مصفوفة أو نص)
    'error'       => string,    // رسالة الخطأ (إذا وجد)
    'message'     => mixed,     // الرسالة الأصلية من API
]
```

**مثال لاستجابة ناجحة:**

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

**مثال لاستجابة فاشلة:**

```php
[
    'success' => false,
    'status_code' => 402,
    'error' => 'رصيد غير كافٍ',
    'message' => ['status' => false, 'message' => 'Insufficient balance']
]
```

---

## العمليات العامة (Global Operations)

هذه الدوال تخص حساب الوكيل وتستخدم `NetworkNumber = 0`.

| الدالة | الوصف | مثال الطلب |
|--------|-------|-----------|
| `getAccountBalance()` | استعلام رصيد الوكيل | `$api->getAccountBalance();` |
| `getOperationStatus($transactionId)` | استعلام حالة معاملة سابقة | `$api->getOperationStatus('TXN_123');` |
| `getFeedClientsBalance()` | عرض إيداعات الوكيل | `$api->getFeedClientsBalance();` |
| `getGamesAndGiftCardsCategories()` | جلب فئات الألعاب والبطاقات | `$api->getGamesAndGiftCardsCategories();` |

**مثال:**

```php
$balance = $api->getAccountBalance();
if ($balance['success']) {
    echo "الرصيد الحالي: " . $balance['data']['agentBalance'] . " ريال";
}
```

---

## الدوال العامة المرنة (Flexible Generic Methods)

هذه الدوال تسمح بتنفيذ أي خدمة مع تحديد رقم الشبكة ورقم الخدمة.

### 5.1 `executeService`

```php
public function executeService($networkNumber, $serviceNumber, $additionalFields = [], $transactionId = null)
```

تنفيذ خدمة مخصصة بأي رقم شبكة وخدمة.  
**المعاملات:**
- `$networkNumber`: رقم الشبكة (مثال: 1 ليمن موبايل)
- `$serviceNumber`: رقم الخدمة (مثال: 101 لاستعلام رصيد يمن موبايل)
- `$additionalFields`: مصفوفة بالحقول الإضافية (مثل `MobileNumber`, `Amount`, `OfferCode`)
- `$transactionId`: معرف معاملة (يُولَّد تلقائياً إذا لم يُمرر)

**مثال:**

```php
$result = $api->executeService(3, 301, [
    'MobileNumber' => '777777777',
    'Amount' => 100
], 'TXN_123');
```

### 5.2 `queryBalance`

```php
public function queryBalance($networkNumber, $mobileNumber, $transactionId = null)
```

استعلام رصيد لأي شبكة تدعم الخدمة (تحدد رقم الخدمة تلقائياً).  
**المعاملات:**
- `$networkNumber`: رقم الشبكة (1,5,6,7,13)
- `$mobileNumber`: رقم الجوال أو الخط
- `$transactionId`: معرف معاملة (اختياري)

**مثال:**

```php
$result = $api->queryBalance(1, '777777777');
```

### 5.3 `chargeBalanceGeneric`

```php
public function chargeBalanceGeneric($networkNumber, $mobileNumber, $amount, $transactionId = null, $webHookUrl = null, $webHookCode = null)
```

شحن رصيد لأي شبكة تدعم الخدمة (تحدد رقم الخدمة تلقائياً).  
**المعاملات:**
- `$networkNumber`: رقم الشبكة
- `$mobileNumber`: رقم الجوال
- `$amount`: المبلغ (للشبكات التي تستخدم وحدات، يُمثل عدد الوحدات)
- `$transactionId`: معرف معاملة (يُولَّد تلقائياً)
- `$webHookUrl`: رابط WebHook لتلقي تحديثات الحالة (اختياري)
- `$webHookCode`: كود أمان للـ WebHook (اختياري)

**مثال:**

```php
$result = $api->chargeBalanceGeneric(2, '777777777', 50, 'TXN_456');
```

### 5.4 `activateOfferGeneric`

```php
public function activateOfferGeneric($networkNumber, $mobileNumber, $offerCode, $transactionId = null, $webHookUrl = null, $webHookCode = null)
```

تفعيل باقة لأي شبكة تدعم الخدمة (تحدد رقم الخدمة تلقائياً).  
**المعاملات:**
- `$networkNumber`: رقم الشبكة
- `$mobileNumber`: رقم الجوال
- `$offerCode`: كود الباقة
- `$transactionId`: معرف معاملة (يُولَّد تلقائياً)
- `$webHookUrl`: رابط WebHook (اختياري)
- `$webHookCode`: كود WebHook (اختياري)

**مثال:**

```php
$result = $api->activateOfferGeneric(2, '777777777', 'PREWhatsApp', 'TXN_789');
```

---

## دوال الشبكات المحددة (Network-Specific Shortcuts)

### 6.1 استعلامات الرصيد

| الدالة | الشبكة | مثال |
|--------|--------|------|
| `queryYemenMobileBalance($mobileNumber, $transactionId)` | يمن موبايل | `$api->queryYemenMobileBalance('777777777');` |
| `queryYemenMobileLoan($mobileNumber, $transactionId)` | يمن موبايل (قرض) | `$api->queryYemenMobileLoan('777777777');` |
| `queryYemenMobileOffers($mobileNumber, $transactionId)` | يمن موبايل (الباقات) | `$api->queryYemenMobileOffers('777777777');` |
| `queryAdslBalance($mobileNumber, $transactionId)` | ADSL | `$api->queryAdslBalance('12345678');` |
| `queryLandPhoneBalance($mobileNumber, $transactionId)` | هاتف أرضي | `$api->queryLandPhoneBalance('012345678');` |
| `queryYemen4GBalance($mobileNumber, $transactionId)` | Yemen 4G | `$api->queryYemen4GBalance('777777777');` |
| `queryAdenNetBalance($mobileNumber, $transactionId)` | عدن نت | `$api->queryAdenNetBalance('770000000');` |

**مثال:**

```php
$result = $api->queryYemenMobileBalance('777777777');
if ($result['success']) {
    echo "رصيد العميل: " . $result['data']['mobileBalance'];
    echo "نوع الخط: " . $result['data']['mobileTypeName'];
}
```

### 6.2 دوال الشحن الأصلية (للتوافق)

| الدالة | الوصف |
|--------|-------|
| `chargeBalance($networkNumber, $serviceNumber, $mobileNumber, $amount, $transactionId, $webHookUrl, $webHookCode)` | شحن رصيد مع تحديد رقم الخدمة يدوياً. |
| `activateOffer($networkNumber, $serviceNumber, $mobileNumber, $offerCode, $transactionId, $webHookUrl, $webHookCode)` | تفعيل باقة مع تحديد رقم الخدمة يدوياً. |
| `chargeGameOrGiftCard($linkCode, $fields, $quantity, $transactionId, $mobileNumber, $webHookUrl, $webHookCode)` | شحن لعبة أو بطاقة هدايا. |

**مثال:**

```php
// شحن رصيد سبافون (خدمة 301)
$result = $api->chargeBalance(3, 301, '777777777', 100, 'TXN_123');

// تفعيل باقة يو تيليكوم (خدمة 202)
$result = $api->activateOffer(2, 202, '777777777', 'PREWhatsApp', 'TXN_456');

// شحن لعبة بوبجي (60 شدة)
$fields = ['PlayerID' => '123456789'];
$result = $api->chargeGameOrGiftCard('pubg_60', $fields, 1, 'TXN_789');
```

---

## دوال البيانات والمساعدة

هذه الدوال تساعد في الحصول على معلومات حول الشبكات والخدمات والتحقق من صحة البيانات.

| الدالة | الوصف |
|--------|-------|
| `getNetworksList()` | إرجاع مصفوفة بأرقام الشبكات وأسمائها. |
| `getNetworkName($networkNumber)` | الحصول على اسم الشبكة من رقمها. |
| `isValidNetwork($networkNumber)` | التحقق من صحة رقم الشبكة. |
| `getAllServices()` | إرجاع قائمة كاملة بالخدمات لكل شبكة. |
| `getServiceName($networkNumber, $serviceNumber)` | الحصول على وصف الخدمة من أرقام الشبكة والخدمة. |
| `getServicesForNetwork($networkNumber)` | الحصول على قائمة الخدمات لشبكة معينة. |
| `isValidService($networkNumber, $serviceNumber)` | التحقق من صحة رقم الخدمة لشبكة معينة. |
| `getBillBalanceServiceNumber($networkNumber)` | رقم خدمة شحن الرصيد للشبكة. |
| `getQueryBalanceServiceNumber($networkNumber)` | رقم خدمة استعلام الرصيد للشبكة. |
| `getQueryOffersServiceNumber($networkNumber)` | رقم خدمة الباقات للشبكة. |
| `supportsBalanceQuery($networkNumber)` | هل الشبكة تدعم استعلام الرصيد؟ |
| `supportsBalanceBill($networkNumber)` | هل الشبكة تدعم شحن الرصيد المباشر؟ |
| `supportsOffers($networkNumber)` | هل الشبكة تدعم الباقات؟ |
| `isValidMobileNumber($mobileNumber, $networkNumber)` | التحقق من صحة رقم الجوال حسب الشبكة (اختياري). |
| `formatMobileNumber($mobileNumber)` | تنسيق الرقم (إزالة الأصفار الزائدة). |
| `getOperationStatusText($operationStatus)` | تحويل رمز حالة العملية إلى نص عربي. |

**أمثلة:**

```php
// قائمة الشبكات
$networks = $api->getNetworksList();
print_r($networks);

// التحقق من دعم الشبكة
if ($api->supportsBalanceQuery(1)) {
    echo "يمن موبايل تدعم استعلام الرصيد";
}

// التحقق من رقم جوال
if ($api->isValidMobileNumber('777777777', 1)) {
    echo "رقم صحيح ليمن موبايل";
}

// تنسيق رقم
$formatted = $api->formatMobileNumber('0 77 7777777'); // -> 777777777
```

---

## دعم Webhook

عند إجراء عمليات الدفع (شحن، باقات، ألعاب)، يمكن تمرير `WebHookURL` و `WebHookCode` اختياريًا. ستقوم المنصة بإرسال تحديثات حالة المعاملة إلى هذا الرابط باستخدام طريقة GET مع المعاملات التالية:

| المعامل | الوصف |
|---------|-------|
| `OperationStatus` | 1: نجاح، 0: فشل |
| `WebHookCode` | الكود الذي أرسلته مسبقًا |
| `TransactionID` | معرف المعاملة من نظامك |
| `ReferenceID` | معرف المعاملة في نظام nflow.tech |
| `price` | السعر الذي تم خصمه من رصيدك |
| `message` | رسالة توضيحية |

**مثال لاستقبال Webhook في Laravel:**

```php
Route::get('/webhook/yepayment', function (Request $request) {
    $status = $request->get('OperationStatus');
    $txnId = $request->get('TransactionID');
    // ... حفظ الحالة في قاعدة البيانات
});
```

**إرسال الطلب مع Webhook:**

```php
$result = $api->chargeBalanceGeneric(
    1, '777777777', 100, 'TXN_123',
    'https://your-site.com/webhook/yepayment',
    'SECURE_CODE'
);
```

---

## اختبار الاتصال

للتأكد من صحة البيانات وإمكانية الاتصال بالخادم:

```php
$test = $api->testConnection();
if ($test['success']) {
    echo "الاتصال ناجح. التوكن موجود: " . ($test['token'] ? 'نعم' : 'لا');
} else {
    echo "فشل الاتصال: " . $test['error'];
}
```

---

## معالجة الأخطاء

جميع الدوال تعيد مصفوفة بها `success`، ويمكن التعامل مع الأخطاء كالتالي:

```php
$result = $api->queryYemenMobileBalance('777777777');
if ($result['success']) {
    // نجاح
} else {
    switch ($result['status_code']) {
        case 401:
            echo "فشل المصادقة، تأكد من بيانات الاعتماد.";
            break;
        case 402:
            echo "رصيد غير كافٍ.";
            break;
        case 404:
            echo "الخدمة أو الرقم غير موجود.";
            break;
        case 423:
            echo "الخدمة مقفلة مؤقتًا.";
            break;
        default:
            echo "خطأ: " . ($result['error'] ?? 'غير معروف');
    }
}
```

---

## أمثلة تطبيقية

### مثال 1: شحن رصيد يمن موبايل

```php
$txnId = 'TXN_' . uniqid();
$result = $api->chargeBalanceGeneric(1, '777777777', 500, $txnId);
if ($result['success']) {
    echo "تم شحن الرصيد بنجاح. المرجع: " . $result['data']['referenceID'];
}
```

### مثال 2: تفعيل باقة YOU Telecom

```php
$result = $api->activateOfferGeneric(2, '777777777', 'PREWhatsApp', 'TXN_123');
```

### مثال 3: استعلام رصيد ADSL

```php
$result = $api->queryAdslBalance('12345678');
if ($result['success']) {
    echo "الرصيد المتبقي: " . $result['data']['mobileBalance'];
}
```

### مثال 4: شحن لعبة بوبجي

```php
$fields = ['PlayerID' => '123456789'];
$result = $api->chargeGameOrGiftCard('pubg_60', $fields, 1, 'TXN_456');
```

---

## ملخص

كلاس `NflowService` هو أداة متكاملة للتعامل مع خدمات nflow.tech، ويوفر:

- **واجهة موحدة** لكل من العمليات العامة والاستعلامات والدفع.
- **دوال عامة مرنة** `executeService` لتنفيذ أي خدمة بأرقام الشبكة والخدمة.
- **دوال مختصرة** لكل شبكة لتسهيل الاستخدام اليومي.
- **دعم كامل لـ Webhook** لتحديثات الحالة الآلية.
- **دوال مساعدة** للتحقق من صحة البيانات والحصول على معلومات الشبكات والخدمات.
- **معالجة أخطاء احترافية** مع إرجاع أكواد HTTP والرسائل المناسبة.

سواء كنت مطورًا تحتاج إلى دمج هذه الخدمات في تطبيقك، أو مسؤولًا يقوم بتحديثات جماعية، فإن `NflowService` يوفر الأدوات اللازمة بأمان وكفاءة.

---

**ملاحظة:** لمزيد من التفاصيل حول واجهة API الأصلية، راجع [https://nflow.tech/api-documentation](https://nflow.tech/api-documentation).  
للاطلاع على الأمثلة المتقدمة والسيناريوهات العملية، راجع [Docs-NflowService-Advanced-Examples-ar.md](./Docs-NflowService-Advanced-Examples-ar.md).