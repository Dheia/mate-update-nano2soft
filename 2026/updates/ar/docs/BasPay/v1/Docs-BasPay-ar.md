# توثيق بوابة الدفع BasPay – منصة بس

## 1. نظرة عامة

**BasPay** هي بوابة دفع مباشر (Immediate/Direct Payment) مدمجة ضمن حزمة `Nano.Yepayment` في تطبيقات نانوسوفت. تعتمد البوابة على واجهة برمجة تطبيقات **منصة بس (BAS)** وتتيح تنفيذ عمليات الدفع إلكترونياً بشكل فوري دون حاجة إلى إعادة توجيه المستخدم أو خطوات تأكيد إضافية.

تعمل البوابة وفق آلية:
1. **مصادقة OAuth 2.0** – الحصول على رمز دخول (Bearer Token) باستخدام بيانات التاجر.
2. **إنشاء معاملة دفع فورية** – إرسال بيانات الطلب (المبلغ، العملة، رقم الطلب) مع توقيع مشفر AES-256-CBC إلى API منصة بس، واستلام `trxToken` فورياً.

تدعم البوابة العملات: الريال اليمني (`YER`)، الدولار الأمريكي (`USD`)، والريال السعودي (`SAR`). كما تمتاز البوابة بدعم اختياري لروابط إعادة التوجيه (Callback URLs) باستخدام `RedirectHelper` لتتوافق مع تطبيقات الجوال.

هذه البوابة مبنية وفق معايير `Nano.Yepayment` وتتبع نمط `PaymentProvider` مثل `YottaPay` و `QasemiPay`.

---

## 2. متطلبات التشغيل والإعدادات

### 2.1. المتطلبات الأساسية

- نظام نانوسوفت (Nano2Soft) الإصدار 2.0+
- الإضافات المطلوبة:
  - `Nano.MicroCart` (>=2.0)
  - `Nano.Yepayment` (>=1.2)
  - `Nano.Helpers`
- بيانات الدخول إلى منصة بس (مقدمة من الدعم الفني):
  - `Client ID`
  - `Client Secret`
  - `App ID`
  - `Merchant Key` (للتوقيع المشفر)
  - `IV` – قيمة المتجه الأولي (16 بايت)، غالباً القيمة الافتراضية `@@@@&&&&####$$$$`
  - رابط API الأساسي (مثال: `https://api.basgate.com`)

### 2.2. إعدادات البوابة في لوحة التحكم

عند تفعيل طريقة الدفع **"BAS Payment Gateway"**، تظهر الحقول التالية في صفحة إعدادات المدفوعات:

| الحقل | المفتاح | الوصف | القيمة الافتراضية |
|-------|---------|-------|-------------------|
| رابط API الأساسي | `baspay_url` | عنوان API منصة بس | `https://api.basgate.com` |
| Client ID | `baspay_client_id` | معرف العميل | - |
| Client Secret | `baspay_client_secret` | المفتاح السري للعميل (مخزن مشفر) | - |
| App ID | `baspay_app_id` | معرف التطبيق | - |
| Merchant Key | `baspay_merchant_key` | مفتاح التاجر للتوقيع المشفر (مخزن مشفر) | - |
| IV | `baspay_iv` | قيمة المتجه الأولي للتشفير (16 بايت) | `@@@@&&&&####$$$$` |
| العملة الافتراضية | `baspay_default_currency` | العملة المستخدمة عند عدم تحديدها | `YER` |

> **ملاحظة أمنية:** يتم تخزين الحقول الحساسة (`Client Secret`, `Merchant Key`) بشكل مشفر في قاعدة البيانات عبر `encryptedSettings()`.

---

## 3. دورة حياة عملية الدفع (Payment Flow)

### 3.1. تنفيذ الدفع الفوري (`process`)

عند بدء الدفع لطلب معين، يتم استدعاء `process()` والتي تقوم بالخطوات التالية:

1. **التحقق من حالة الطلب** – إذا كان الطلب مدفوع مسبقاً (`PaidState`) يتم إيقاف العملية.
2. **الحصول على رمز الدخول (Access Token)** عبر OAuth 2.0 باستخدام `client_credentials` (يتم تخزينه في الـ Cache لمدة 3500 ثانية).
3. **توليد توقيع مشفر (Signature)** وفق آلية BAS:
   - إنشاء سلسلة عشوائية `salt` طولها 4 أحرف.
   - حساب `SHA256` للنص المطلوب توقيعه مع `salt`.
   - تشفير الناتج باستخدام `AES-256-CBC` بمفتاح التاجر والـ `IV`.
4. **إرسال طلب إنشاء معاملة** إلى `/api/v1/merchant/sdk-payment/initiate-transaction` مع البيانات التالية:
   ```json
   {
     "head": {
       "signature": "<التوقيع>",
       "requestTimestamp": 1696000000000
     },
     "body": {
       "amount": { "value": 100, "currency": "YER" },
       "ordertype": "PayBill",
       "orderId": "200",
       "requestTimestamp": 1696000000000,
       "appId": "<app_id>"
     }
   }
   ```
5. **استلام الاستجابة** – إذا كانت `status = 1` و `code = '1111'` فإن المعاملة ناجحة، ويتم استخراج `trxToken` من `response.body.trxToken`.
6. **تحديث بيانات الطلب**:
   - تعيين `order.payment_first_trans_id` و `order.payment_trans_id` بقيمة `trxToken`.
   - تخزين معلومات العملية في `order.other_data['baspay']` (الـ token، المبلغ، العملة، الطابع الزمني).
   - تغيير حالة الدفع إلى `PaidState`.
7. **تسجيل سجل الدفع** في جدول `PaymentLog`.
8. إرجاع `PaymentResult` بنجاح (`successful = true`).

### 3.2. روابط إعادة التوجيه الاختيارية (اختياري)

مع أن BasPay هي بوابة دفع مباشر، إلا أنها تدعم تخزين روابط `callback_success_url` و `callback_error_url` إن تم تمريرها ضمن بيانات الدفع (مثلاً من تطبيق جوال). يمكن للمطور بعد تنفيذ الدفع توجيه المستخدم باستخدام المسارين:

- `GET /api/v1/yepayment/baspay/success` – يسترجع رابط العودة عند النجاح ويستخدم `RedirectHelper` لإعادة التوجيه إلى التطبيق أو الموقع.
- `GET /api/v1/yepayment/baspay/cancel` – مشابه لحالة الإلغاء.

هذا يضمن توافقاً كاملاً مع تطبيقات الجوال التي تحتاج إلى Deep Links بعد الدفع.

---

## 4. هيكل البيانات المخزنة في `order->other_data['baspay']`

```php
[
    'trx_token' => 'abc123...',             // المعرف الفريد للمعاملة من BAS
    'amount'    => 100.00,                  // المبلغ المدفوع
    'currency'  => 'YER',                   // العملة
    'order_id'  => '200',                   // رقم الطلب
    'timestamp' => '2025-01-01T12:00:00Z',  // وقت التنفيذ
    'callback_success_url' => 'myapp://...',// (اختياري) رابط العودة عند النجاح
    'callback_error_url'   => 'myapp://...' // (اختياري) رابط العودة عند الخطأ
]
```

---

## 5. واجهة اختبار API (Test UI)

توفر البوابة واجهة ويب متكاملة لاختبار جميع الوظائف، يمكن الوصول إليها عبر:

```
/api/v1/yepayment/baspay/test-ui
```

### 5.1. الميزات

- **اختبار المصادقة** (Auth) – التحقق من صحة بيانات Client ID/Secret عبر `/baspay/test-auth`.
- **إنشاء دفعة فورية** – إدخال رقم الطلب، المبلغ، العملة لتُنفذ دفعة جديدة.
- **التحقق من الحالة** – الاستعلام عن معاملة باستخدام `trxToken`.
- **اختبار شامل** – يجمع بين إنشاء الدفعة والتحقق من الحالة في خطوة واحدة.
- **اختبار تلقائي متكرر** – إمكانية تحديد عدد مرات التكرار لقياس الاستقرار.
- **إحصائيات فورية** – عدد الطلبات، نسبة النجاح، آخر السجلات.
- **سجلات محلية** – تخزن نتائج الاختبارات في `localStorage` للمتصفح.

### 5.2. نقاط نهاية API المستخدمة في الاختبار

| الغرض | الطريقة | المسار |
|-------|--------|--------|
| المصادقة | POST | `/baspay/test-auth` |
| إنشاء دفعة | POST | `/baspay/test-create-payment` |
| التحقق من الحالة | POST | `/baspay/test-check-status` (body: `{trx_token}`) |
| اختبار شامل | POST | `/baspay/test-full-payment` |
| إحصائيات | GET | `/baspay/stats` |

---

## 6. أمثلة على الطلبات والاستجابات

### 6.1. إنشاء دفعة جديدة (ناجحة)

**طلب:**
```http
POST /api/v1/yepayment/baspay/test-create-payment
Content-Type: application/json

{
    "order_id": 200,
    "amount": 100,
    "currency": "YER"
}
```

**استجابة:**
```json
{
    "success": true,
    "message": "Payment completed",
    "trx_token": "bas_trx_a1b2c3d4...",
    "api_data": {
        "success": true,
        "trx_token": "bas_trx_a1b2c3d4...",
        "amount": 100,
        "currency": "YER",
        "order_id": "200",
        "raw_response": { ... }
    }
}
```

### 6.2. التحقق من حالة معاملة

**طلب:**
```http
POST /api/v1/yepayment/baspay/test-check-status
Content-Type: application/json

{
    "trx_token": "bas_trx_a1b2c3d4..."
}
```

**استجابة (مثال):**
```json
{
    "success": true,
    "data": {
        "head": { ... },
        "body": {
            "trxToken": "bas_trx_a1b2c3d4...",
            "trxStatus": "SUCCESS",
            "order": { "orderId": "200" }
        }
    }
}
```

### 6.3. اختبار شامل (إنشاء + فحص)

**طلب:**
```http
POST /api/v1/yepayment/baspay/test-full-payment
{
    "order_id": 200,
    "amount": 100,
    "currency": "YER"
}
```

**استجابة:**
```json
{
    "success": true,
    "results": {
        "step1_create": {
            "success": true,
            "trx_token": "bas_trx_..."
        },
        "step2_status": {
            "success": true,
            "data": { ... }
        }
    }
}
```

---

## 7. دمج البوابة في مشروعك (Developer Guide)

### 7.1. تسجيل طريقة الدفع في `Plugin.php`

```php
if (config('nano.microcart::paymenttypes.allow_yemen_payment', true)) {
    // ... المزودون الآخرون ...
    if (class_exists(\Nano\Yepayment\PaymentTypes\BasPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\BasPay();
    }
}
```

### 7.2. استخدام البوابة عبر `PaymentGateway`

```php
use Nano\MicroCart\Classes\Payments\DefaultPaymentGateway;
use Nano\Orders\Models\Order;

$order = Order::find($orderId);
$gateway = new DefaultPaymentGateway();
$gateway->init($paymentMethod, ['callback_success_url' => 'myapp://pay/success']);
$result = $gateway->process($order);
if ($result->successful) {
    // تم الدفع بنجاح – يمكن إعادة التوجيه إلى success_url إن وجد
}
```

### 7.3. تنفيذ الدفع مباشرة (بدون PaymentGateway)

```php
use Nano\Yepayment\PaymentTypes\BasPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$bas = new BasPay($order, [
    'callback_success_url' => 'myapp://pay/success', // اختياري
]);
$paymentResult = new PaymentResult($bas, $order);
$processResult = $bas->process($paymentResult);

if ($processResult->successful) {
    $trxToken = $order->payment_first_trans_id;
    // إعادة التوجيه إن لزم
}
```

---

## 8. استكشاف الأخطاء (Troubleshooting)

| المشكلة | السبب المحتمل | الحل |
|---------|---------------|------|
| فشل المصادقة (`auth_failed`) | Client ID/Secret غير صحيح | تحقق من الإعدادات المدخلة في لوحة التحكم |
| `payment_creation_failed` مع استجابة خطأ من BAS | `trxToken` مفقود أو `code` لا يساوي `1111` | افحص الـ logs للاستجابة الكاملة؛ قد يكون المبلغ أقل من الحد الأدنى أو العملة غير مدعومة |
| توقيع غير صحيح | Merchant Key أو IV غير متطابقين | تأكد من مطابقة القيم مع تلك المقدمة من منصة بس |
| الطلب مدفوع مسبقاً | `order.payment_state == PaidState` | لا يمكن الدفع مرة أخرى لنفس الطلب |
| خطأ `cURL error` | مشكلة في الاتصال بـ API | تحقق من صحة الرابط ووجود اتصال إنترنت |
| حالة المعاملة غير معروفة عند التحقق | `trxToken` غير صحيح | تأكد من أن التوكن مستخدم في طلب التحقق هو نفسه الناتج عن `createPayment` |

---

## 9. ملاحظات إضافية

- بوابة **BasPay** لا تحتاج إلى `complete()` لأن الدفع فوري؛ الدالة موجودة للتوافق مع الواجهة فقط.
- التوقيع المشفر يعتمد على نفس الخوارزمية المستخدمة في **BAS SDK** (`AES-256-CBC + SHA256 + salt`)، مما يضمن التوافق التام مع متطلبات منصة بس.
- يتم تخزين `Access Token` في الـ Cache لمدة 3500 ثانية لتقليل عدد طلبات المصادقة.
- يمكن تمديد الكلاس لإضافة خاصية **Webhook** أو استرداد الأموال مستقبلاً دون كسر البنية الحالية.

---

## 10. ملفات الكود الخاصة بالبوابة

| الملف | الوصف |
|-------|-------|
| `BasPay.php` | الكلاس الرئيسي للبوابة (يمتد من `PaymentProvider`) |
| `_info.htm` | قالب معلومات الإعدادات في لوحة التحكم |
| `_test_info.htm` | أدوات الاختبار السريع مع أزرار تفاعلية |
| `baspay-ui.htm` | واجهة الاختبار الكاملة (Test UI) |
| `routes.php` (قسم BasPay) | تعريف نقاط نهاية الاختبار والإحصائيات وواجهة المستخدم |
| `Plugin.php` (إضافة) | تسجيل البوابة كمزود دفع |

---

**تم إعداد هذا التوثيق لدعم المطورين في دمج واستخدام بوابة BasPay بسهولة وكفاءة.**  
للاستفسارات أو الدعم الفني، يرجى التواصل عبر الموقع الرسمي [nano2soft.com](https://nano2soft.com).