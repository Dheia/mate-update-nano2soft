
# توثيق بوابة الدفع YottaPay (Sabacash) – إصدار نانوسوفت

## 1. نظرة عامة

**YottaPay** (التي تعمل تحت الاسم التجاري **Sabacash**) هي بوابة دفع إلكترونية متكاملة تتيح قبول المدفوعات عبر الإنترنت باستخدام واجهة برمجة تطبيقات **YottaGate**. تم بناء هذه البوابة خصيصاً لتطبيقات **Nano2Soft** (نانوسوفت) وتعمل كإضافة ضمن حزمة `Nano.Yepayment`. تعتمد البوابة على آلية من خطوتين لضمان أمان المعاملات:

1. **إنشاء معاملة دفع** (Create Adjustment) – يتم إرسال بيانات العميل والمبلغ والتيرمينال.
2. **تأكيد المعاملة** (Confirm Adjustment) – باستخدام رمز OTP يتم إرساله إلى العميل.

كما تدعم البوابة عمليات **استرجاع الأموال** (Money Return) والتحقق من حالة المعاملة عبر معرف المعاملة الفريد (Transaction ID). تستخدم المصادقة عبر **OAuth 2.0** (نوع `bearer token` يتم الحصول عليه عبر تسجيل الدخول بنظام YottaGate).

---

## 2. متطلبات التشغيل والتثبيت

- نظام **Nano2Soft App** الإصدار 2 فما فوق.
- إضافات مطلوبة:
  - `Nano.MicroCart` (>= 2.0) لإدارة السلة والطلبات.
  - `Nano.Yepayment` (>= 1.2) – تضم بوابة YottaPay.
  - `Nano.Helpers` لخدمة HTTP.
- بيانات الدخول إلى **YottaGate** (مقدمة من المشغل):
  - `username` (اسم المستخدم)
  - `password` (كلمة المرور)
  - `terminal` (رقم التيرمينال الافتراضي، مثل "1")
  - عنوان URL الأساسي للـ API (عادةً `https://api.sabacash.com:49901`)

---

## 3. إعدادات بوابة الدفع في لوحة التحكم

عند تفعيل طريقة الدفع **YottaPay (Sabacash)** في لوحة تحكم المتجر، تظهر الإعدادات التالية:

| الحقل | الوصف | مثال |
|-------|-------|------|
| **رابط API الأساسي** | عنوان URL لـ YottaGate | `https://api.sabacash.com:49901` |
| **اسم المستخدم** | اسم مستخدم التاجر في النظام | `712988875` |
| **كلمة المرور** | كلمة المرور الخاصة بالمستخدم | (تخزن مشفرة) |
| **رقم التيرمينال الافتراضي** | رقم التيرمينال الذي سيُستخدم في المعاملات | `1` |
| **معرف العملة الافتراضي** | معرف العملة (1 = ريال يمني، 2 = ريال سعودي، إلخ) | `1` |

> **ملاحظة:** يتم تخزين كلمة المرور بشكل مشفر في قاعدة البيانات. يمكن للمطور تعديل `encryptedSettings()` في الكلاس لإضافة أي حقول حساسة أخرى.

---

## 4. تدفق عملية الدفع (Payment Flow)

### 4.0 دورة عمل الدفع (Payment Workflow)

<img src="./images/sequenceDiagram-YottaGate-ar.jpg" alt="مخطط تدفق YottaGate" width="600">

### 4.1. إنشاء معاملة دفع جديدة (Create Transaction)

يتم استدعاء `process()` في كلاس `YottaPay` عند بدء الدفع. الخطوات:

1. **التحقق من صحة المدخلات** (مثل رقم الهاتف، المبلغ، التيرمينال) – يتم من خلال `checkValidate()`.
2. **طلب توكن OAuth 2.0** عبر `getAuthToken()` (يستخدم بيانات تسجيل الدخول المخزنة في الإعدادات).
3. **إرسال طلب `POST /api/accounts/v1/adjustment/onLinePayment`** إلى YottaGate مع البيانات التالية:
   ```json
   {
     "source": {
       "code": "771234567",
       "currencyId": "1"
     },
     "beneficiary": {
       "terminal": "1",
       "currencyId": "1"
     },
     "amount": "1000",
     "amountCurrencyId": "1",
     "note": "دفع قيمة الطلب #200"
   }
   ```
4. **تسجيل البيانات**:
   - حفظ `adjustment.id` في `order->payment_first_trans_id`.
   - حفظ `transactionId` (مثل `DF-01-...`) في `order->payment_trans_id` (إذا ورد).
   - تخزين بيانات إضافية في `order->other_data['yottapay']`.
5. **تسجيل محاولة الدفع** في جدول `nano_microcart_payment_logs` (الحالة: غير مؤكد).
6. إرجاع `PaymentResult` مع `successful = true` ورسالة تفيد بضرورة إدخال OTP، دون إعادة توجيه (لأن الدفع يتم عبر API مباشرة).

### 4.2. تأكيد الدفع باستخدام OTP (Confirm Transaction)

يتم استدعاء `complete()` بعد حصول العميل على رمز OTP (يُرسل عبر SMS من قبل YottaGate). الخطوات:

1. استرداد `adjustmentId` من `order->payment_first_trans_id`.
2. استرداد `otp` من بيانات الطلب (مثل `Request::input('otp')`).
3. إرسال طلب `PATCH /api/accounts/v1/adjustment/onLinePayment` إلى YottaGate:
   ```json
   {
     "id": "816613",
     "otp": "4320",
     "note": "تأكيد الدفع"
   }
   ```
4. التحقق من نجاح العملية (حقل `completed: true`).
5. تحديث `order->payment_state` إلى `PaidState`.
6. حفظ `transactionId` (من الاستجابة) في `order->payment_trans_id` إن لم يكن موجوداً.
7. تسجيل الدفع كـ **ناجح** عبر `$result->success()`.

### 4.3. استرجاع الأموال (Online Money Return)

تدعم البوابة عمليات الاسترجاع (refund) عبر API منفصل. يتم استخدامه بشكل أساسي من قبل النظام أو التاجر:

**طلب إنشاء استرجاع:**
```http
POST /api/accounts/v1/adjustment/onlineMoneyReturn
{
  "id": "816613",   // adjustment ID الأصلي
  "source": { "currencyId": "1", "isTerminal": true },
  "beneficiary": { "code": "711592626", "currencyId": "1" },
  "amount": "100",
  "amountCurrencyId": "1",
  "transactionId": "5697302956",
  "note": "استرجاع أموال"
}
```

**تأكيد الاسترجاع (Patch):**
```http
PATCH /api/accounts/v1/adjustment/onlineMoneyReturn
{
  "id": 816614,
  "note": "تأكيد استرجاع"
}
```

> **ملاحظة:** في حال عدم كفاية رصيد التيرمينال، يمكن تعيين `isTerminal: false` ليتم الاسترجاع من رصيد التاجر الرئيسي.

### 4.4. التحقق من حالة المعاملة

يمكن استدعاء `checkTransactionStatus($transactionId)` في أي وقت لإرسال طلب GET إلى:
```
GET /api/accounts/v1/adjustment/checkAdjustmentByTransactionId?transactionId=DF-01-...
```
الاستجابة تعيد `statusCode` والذي يمكن أن يكون:
- `"completed"` – معاملة ناجحة ومكتملة.
- `"not-completed"` – معلقة (لم تؤكد بعد).
- `"not-exist"` – غير موجودة.

---

## 5. هيكل البيانات المخزنة في `order->other_data['yottapay']`

```php
[
    'adjustment_id'      => '816613',            // id من استجابة إنشاء المعاملة
    'transaction_id'     => 'DF-01-...',         // معرف المعاملة النهائي
    'source_phone'       => '771234567',
    'terminal'           => '1',
    'amount'             => 1000,
    'currency_id'        => '1',
    'note'               => 'دفع الطلب #200',
    'requires_otp'       => true,
    'created_at'         => '2025-01-01T12:00:00Z',
    'callback_success_url' => '...',            // اختياري
    'callback_error_url'   => '...',
]
```

---

## 6. واجهة اختبار API (Test UI)

توفر البوابة واجهة ويب متكاملة لاختبار جميع الوظائف، يمكن الوصول إليها عبر:

```
/api/v1/yepayment/yottapay/test-ui
```

### 6.1. الميزات

- **اختبار المصادقة** – الحصول على توكن API.
- **إنشاء معاملة تجريبية** – مع إمكانية إدخال رقم الهاتف، المبلغ، التيرمينال.
- **تأكيد المعاملة** – باستخدام OTP تجريبي (مثلاً `1234`).
- **التحقق من الحالة** – عبر معرف المعاملة.
- **اختبار شامل** – ينفذ الخطوات الثلاث تلقائياً ويعرض النتائج.
- **اختبار تكراري** – لتقييم استقرارية البوابة (حتى 10 مرات).
- **إحصائيات فورية** – عدد الطلبات، نسبة النجاح، آخر السجلات.
- **سجلات الاختبارات** – تخزين محلي (LocalStorage) مع إمكانية التصدير والمسح.

### 6.2. نقاط نهاية API الخاصة بالاختبار

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/yottapay/test-auth` | POST | التحقق من صحة بيانات الدخول والحصول على توكن (تجريبي). |
| `/yottapay/test-create-payment` | POST | إنشاء معاملة دفع تجريبية (يحاكي `process`). |
| `/yottapay/test-confirm-payment` | POST | تأكيد الدفع باستخدام OTP. |
| `/yottapay/test-check-status` | GET | الاستعلام عن حالة معاملة. |
| `/yottapay/test-full-payment` | POST | اختبار شامل لجميع الخطوات. |
| `/yottapay/test-change-password` | POST | اختبار تغيير كلمة المرور (للحساب). |
| `/yottapay/stats` | GET | إحصائيات استخدام البوابة. |

> **ملاحظة:** جميع نقاط النهاية هذه محمية بـ `BackendAuth` (تتطلب تسجيل دخول مسؤول) ولا تُستخدم في بيئة الإنتاج.

---

## 7. أمثلة عملية للطلبات والاستجابات

### 7.1. إنشاء معاملة دفع ناجحة

**طلب (من داخل `YottaPay::process`):**
```http
POST https://api.sabacash.com:49901/api/accounts/v1/adjustment/onLinePayment
Authorization: Bearer <token>
Content-Type: application/json

{
  "source": { "code": "771234567", "currencyId": "1" },
  "beneficiary": { "terminal": "1", "currencyId": "1" },
  "amount": "1000",
  "amountCurrencyId": "1",
  "note": "طلب #200"
}
```

**استجابة:**
```json
{
  "destination": {
    "code": "771234567",
    "exchangePrice": 1,
    "isRegistered": true,
    "viral": false,
    "currencyId": "1"
  },
  "adjustment": {
    "amount": 1000,
    "paymentTypeId": 0,
    "sourceFeeAmount": 0,
    "thirdPartyFeeAmount": 0,
    "id": 816613,
    "makerChecker": false,
    "destinationFeeAmount": 0,
    "adjustmentTypeId": 620,
    "transactionId": "DF-01-0123456-0123456789"
  },
  "source": {
    "code": "771234567",
    "exchangePrice": 1,
    "isRegistered": true,
    "viral": false,
    "currencyId": "1"
  }
}
```

### 7.2. تأكيد الدفع باستخدام OTP

**طلب:**
```http
PATCH https://api.sabacash.com:49901/api/accounts/v1/adjustment/onLinePayment
Authorization: Bearer <token>
Content-Type: application/json

{
  "id": "816613",
  "otp": "4320",
  "note": "تأكيد دفع الطلب #200"
}
```

**استجابة ناجحة:**
```json
{
  "id": 816613,
  "completed": true,
  "transactionId": "DF-01-0123456-0123456789"
}
```

### 7.3. التحقق من حالة معاملة

**طلب:**
```http
GET https://api.sabacash.com:49901/api/accounts/v1/adjustment/checkAdjustmentByTransactionId?transactionId=DF-01-0123456-0123456789
Authorization: Bearer <token>
```

**استجابة ناجحة:**
```json
{
  "amount": "1000.000000",
  "transactionDate": "1655714348044",
  "transactionId": "DF-01-0123456-0123456789",
  "statusCode": "completed"
}
```

---

## 8. دليل المطور (Integration Guide)

### 8.1. تسجيل طريقة الدفع في `Plugin.php`

```php
public function registerPaymentProviders()
{
    $providers = [];
    if (class_exists(\Nano\Yepayment\PaymentTypes\YottaPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\YottaPay();
    }
    return $providers;
}
```

### 8.2. استخدام البوابة عبر `PaymentGateway`

```php
use Nano\MicroCart\Classes\Payments\DefaultPaymentGateway;
use Nano\Orders\Models\Order;

$order = Order::find($orderId);
$gateway = new DefaultPaymentGateway();
$gateway->init($paymentMethod, [
    'source_phone' => '771234567',
    'amount'       => 1000,
    // ... بيانات أخرى
]);
$result = $gateway->process($order);

if ($result->successful && $result->redirect) {
    // في حالة YottaPay لا توجد إعادة توجيه، ولكن نعرض رسالة تطلب إدخال OTP
    return view('payment.otp_form', ['adjustment_id' => $order->payment_first_trans_id]);
}
```

### 8.3. تأكيد الدفع بعد إدخال OTP من العميل

```php
$paymentProvider = new YottaPay();
$paymentProvider->setOrder($order);
$result = new PaymentResult($paymentProvider, $order);

// تعيين OTP من الطلب
$paymentProvider->data['otp'] = Request::input('otp');
$completeResult = $paymentProvider->complete($result);

if ($completeResult->successful) {
    // الدفع تم بنجاح
}
```

### 8.4. استدعاء الاسترجاع (Refund) برمجياً

```php
$yottaPay = new YottaPay();
$refundResult = $yottaPay->refund($adjustmentId, $amount, $note, $isTerminal = true);
// $refundResult['success'] => true/false
```

> **ملاحظة:** دالة `refund` غير موجودة في الكلاس الحالي، يمكن إضافتها باتباع وثائق API الخاصة بالاسترجاع.

---

## 9. رموز الأخطاء الشائعة (Error Codes)

| كود HTTP | رسالة الخطأ (مثال) | المعنى والحل |
|----------|-------------------|---------------|
| 401 | Unauthorized | فشل المصادقة – تحقق من اسم المستخدم/كلمة المرور في الإعدادات. |
| 400 | entity of mobile [7xxxx] is not exists | رقم الهاتف المصدر غير مسجل في نظام YottaGate. |
| 400 | Sender do not has sufficient balance | رصيد المصدر غير كافٍ لإجراء المعاملة. |
| 400 | can not do transaction, you Already did transaction for this Adjustment | تم تأكيد المعاملة سابقاً – لا يمكن إعادة التأكيد. |
| 400 | Operation expired, this Operation Timed out and Closed | انتهت صلاحية المعاملة (عادة بعد فترة زمنية محددة) – يجب إنشاء معاملة جديدة. |
| 400 | anonymous users can not perform any transaction | تم استخدام طلب بدون توكن أو بتوكن غير صالح. |
| 404 | Entered id [xxx] does not exist in [adjustment] | معرف التعديل غير موجود – تحقق من `payment_first_trans_id`. |

---

## 10. استكشاف الأخطاء وإصلاحها (Troubleshooting)

| المشكلة | السبب المحتمل | الحل |
|---------|---------------|------|
| فشل `process()` برسالة "Invalid credentials" | اسم المستخدم أو كلمة المرور غير صحيحة | تأكد من بيانات الدخول في إعدادات البوابة. |
| يتم إنشاء المعاملة لكن التأكيد يفشل برسالة "Operation expired" | مر وقت طويل بين الإنشاء والتأكيد | أعد إنشاء معاملة جديدة وأكّدها خلال فترة الصلاحية (عادة 5-10 دقائق). |
| رمز OTP غير صحيح | أدخل العميل رقماً خاطئاً | أعد طلب OTP (إذا كانت البوابة تدعمه) أو أعد إنشاء المعاملة. |
| الاسترجاع يفشل برسالة "Terminal balance is not enough" | رصيد التيرمينال غير كافٍ | أعد الاسترجاع مع `isTerminal: false` ليسحب من رصيد التاجر الرئيسي. |
| لا تظهر واجهة الاختبار `test-ui` | المستخدم الحالي ليس مديراً (backend) | تأكد من تسجيل الدخول إلى لوحة التحكم كمسؤول. |

---

## 11. إضافة تحسينات مستقبلية (مقترحات للمطور)

1. **تخزين التوكن مؤقتاً** – يمكن إضافة طبقة Cache لتخزين `access_token` لمدة 3600 ثانية لتقليل عدد طلبات تسجيل الدخول.
2. **دعم Webhooks** – لاستلام إشعارات فورية عند تأكيد الدفع أو انتهاء صلاحية المعاملة.
3. **إعادة محاولة تلقائية** – في حال فشل تأكيد الدفع بسبب انتهاء الصلاحية، يمكن إعادة إنشاء المعاملة تلقائياً.
4. **توسيع واجهة الاختبار** – إضافة اختبار لاسترجاع الأموال وتغيير كلمة المرور بشكل كامل.

---

## 12. المراجع والروابط

- [وثيقة تسجيل الدخول وتغيير كلمة المرور (YottaPay)](./yottaPay%20Login%20-%20Change%20My%20Password%20API%20Documentation%20V.1.1.pdf)
- [وثيقة إنشاء وتأكيد الدفع (Online Payments v1.3)](./API%20Online%20Payments%20v%201.3.pdf)
- [وثيقة استرجاع الأموال (Online Payment Return v1.4)](./API%20Online%20Payment%20Return%20v%201.4.pdf)
- [وثيقة التحقق من الحالة (Online Status Check v1.3)](./API%20Online%20Status%20Check%20v1.3.pdf)
- [مجموعة Postman لاختبار API](./Sabacash%20online%20payment.postman_collection.json)
- [كلاس YottaPay في نانوسوفت](./YottaPay.php)
- [ملف التوجيه (routes) مع نقاط نهاية YottaPay](./routes.php)

---

**تم إعداد هذا التوثيق لدعم المطورين في دمج واستخدام بوابة YottaPay بسهولة وكفاءة.**  
للاستفسارات أو الدعم الفني، يرجى التواصل عبر الموقع الرسمي [nano2soft.com](https://nano2soft.com) أو عبر قنوات الدعم المخصصة.
