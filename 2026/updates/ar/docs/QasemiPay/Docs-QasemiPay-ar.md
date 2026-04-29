# توثيق بوابة الدفع QasemiPay – مصرف قاسمي الإسلامي

## 1. نظرة عامة

**QasemiPay** هي بوابة دفع متكاملة تتيح قبول المدفوعات عبر خدمة **Purchase Code Service** المقدمة من مصرف قاسمي الإسلامي.  
تعتمد البوابة على واجهة برمجة تطبيقات (API) تستخدم بروتوكول **OAuth 2.0** للمصادقة، وتتم عمليات الدفع عبر خطوتين أساسيتين:

1. **إنشاء معاملة شراء** (Create Transaction) – يتم إرسال بيانات العميل ورمز الشراء.
2. **تأكيد المعاملة** (Confirm Transaction) – يتم باستخدام `concurrencyStamp` لإتمام العملية.

كما توفر البوابة إمكانية **التحقق من حالة المعاملة** في أي وقت عبر معرف المرجع (`refId`).

هذه البوابة مبنية خصيصاً لـ **Nano2Soft App** وتعمل كإضافة ضمن `Nano.Yepayment`، وتتبع نفس نمط بوابات الدفع الأخرى مثل `YottaPay` و `ThawaniPay`.

---

## 2. متطلبات التثبيت والتشغيل

- نظام Nano2Soft App الإصدار 2+.
- إضافة `Nano.MicroCart` لتسيير سلة المشتريات والطلبات.
- إضافة `Nano.Yepayment` (التي تضم بوابة QasemiPay).
- بيانات الوصول إلى API من مصرف قاسمي:
  - `Client ID`
  - `Client Secret`
  - `Username`
  - `Password`
  - `Scope` (افتراضي: `purchase_code_service`)

---

## 3. إعدادات بوابة الدفع (Backend Settings)

عند تفعيل طريقة الدفع **"Qasemi Islamic Bank"** في لوحة التحكم، ستظهر الإعدادات التالية:

| الحقل | الوصف | القيمة الافتراضية |
|-------|-------|-------------------|
| **رابط API الأساسي** | عنوان URL لواجهة API الخاصة بالمصرف | `https://api.qasemi-bank.com` |
| **رابط API للاختبار** | عنوان URL للبيئة التجريبية (اختياري) | `https://test-api.qasemi-bank.com` |
| **Client ID** | معرف العميل من المصرف | - |
| **Client Secret** | المفتاح السري للعميل | - |
| **Username** | اسم مستخدم التكامل | - |
| **Password** | كلمة مرور المستخدم | - |
| **Scope** | نطاق الوصول المطلوب | `purchase_code_service` |
| **العملة الافتراضية** | العملة التي تستخدم إذا لم يحددها العميل | `YER` (ريال يمني) أو `SAR` |
| **المنطقة الافتراضية** | المنطقة الجغرافية للتاجر | `Z1` |
| **الوكيل الافتراضي** | معرف الوكيل (رقمي) | `0` |
| **الفرع الافتراضي** | معرف الفرع (نصي) | فارغ |

> **ملاحظة:** الحقول `Client Secret` و `Password` يتم تخزينها بشكل مشفر في قاعدة البيانات.

---

## 4. تدفق عملية الدفع (Payment Flow)

### 4.1. إنشاء معاملة جديدة (Create Transaction)

يقوم النظام باستدعاء `process()` عند بدء الدفع. الخطوات:

1. **التحقق من صحة المدخلات** – `purchase_code`, `mobile_number`, `amount`, `currency` إلزامية.
2. **طلب توكن OAuth 2.0** – باستخدام بيانات الاعتماد المسجلة.
3. **إرسال طلب إنشاء معاملة** إلى `/api/purchase-code-management/purchase-transactions` مع البيانات التالية:
   ```json
   {
     "purchaseCode": "PC-987654321",
     "currency": "YER",
     "mobileNumber": "+967700000000",
     "amount": 100,
     "refId": "uuid-v4",
     "refNo": "uuid-v4",
     "notes": "دفع قيمة الطلب #200",
     "region": {
       "zone": "Z1",
       "user": "اسم المستخدم",
       "agent": 0,
       "branch": ""
     }
   }
   ```
4. **حفظ بيانات المعاملة** داخل `order->other_data['qasemi']`:
   - `transaction_id` (المعرف الداخلي للمعاملة)
   - `concurrency_stamp` (ختم التزامن)
   - `ref_id` و `ref_no`
5. **تسجيل محاولة الدفع** في جدول `nano_microcart_payment_logs` (الحالة: غير مؤكد بعد).
6. إرجاع `PaymentResult` مع `successful = true` ورسالة **"يجب تأكيد الدفع"**، دون إعادة توجيه.

### 4.2. تأكيد المعاملة (Confirm Transaction)

يتم استدعاء `complete()` بعد الحصول على `concurrencyStamp` (يمكن أن يُطلب من المستخدم إدخال OTP أو يتم استدعاؤها تلقائياً من واجهة التاجر). الخطوات:

1. استرداد `transaction_id` و `concurrencyStamp` من `order->other_data`.
2. طلب توكن جديد (أو استخدام مخزّن).
3. إرسال طلب تأكيد إلى `/api/purchase-code-management/purchase-transactions/confirm`:
   ```json
   {
     "id": "d625aefc-7204-f951-c33c-3a1d09bcc8da",
     "concurrencyStamp": "stamp_string_here"
   }
   ```
4. التحقق من `status == 2` (Succeeded).
5. تحديث `order->payment_state` إلى `PaidState`، وحفظ `ref_no` في `payment_trans_id`.
6. تسجيل الدفع كـ **ناجح** عبر `$result->success()`.

### 4.3. التحقق من حالة المعاملة

يمكن استدعاء `checkTransactionStatus($refId)` في أي وقت لمعرفة حالة المعاملة.  
ترميز الحالات حسب وثائق المصرف:

| Status Code | الحالة        |
|-------------|---------------|
| 2           | Succeeded (ناجحة) |
| 3           | Failed (فشلت)     |
| 4           | Pending (معلقة)   |
| 6           | Refunded (مستردة) |
| 8           | Cancelled (ملغاة) |

---

## 5. هيكل البيانات المخزنة في `order->other_data['qasemi']`

```php
[
    'order_total' => 100.00,                     // المبلغ الإجمالي للطلب
    'transaction_id' => 'd625aefc-7204-...',     // id من استجابة إنشاء المعاملة
    'concurrency_stamp' => 'stamp...',           // ختم التزامن من الاستجابة
    'ref_id' => 'uuid-v4',                       // معرف المرجع المرسل في الطلب
    'ref_no' => 'uuid-v4',                       // رقم مرجع آخر
    'requires_confirmation' => true,             // يشير إلى أن الدفع ينتظر التأكيد
    'created_at' => '2025-01-01T12:00:00Z',      // وقت الإنشاء
    'callback_success_url' => '...',             // رابط العودة عند النجاح (اختياري)
    'callback_error_url' => '...'                // رابط العودة عند الخطأ (اختياري)
]
```

---

## 6. واجهة اختبار API (Test UI)

توفر البوابة واجهة ويب متكاملة لاختبار جميع الوظائف، يمكن الوصول إليها عبر:

```
/api/v1/yepayment/qasemipay/test-ui
```

### 6.1. الميزات

- **اختبار المصادقة** (Auth) – التحقق من صحة بيانات Client ID/Secret و username/password.
- **إنشاء معاملة** – إدخال بيانات رمز الشراء، رقم الجوال، المبلغ، إلخ.
- **تأكيد المعاملة** – باستخدام `transaction_id` و `concurrencyStamp`.
- **التحقق من الحالة** – عبر `refId`.
- **اختبار شامل** – ينفذ الخطوات الثلاث تلقائياً ويعرض النتائج.
- **اختبار تلقائي بتكرار متعدد** – لقياس استقرارية البوابة.
- **إحصائيات فورية** – عدد الطلبات، نسبة النجاح، آخر السجلات.
- **سجلات الاختبارات** – تخزين محلي (LocalStorage) لنتائج الاختبارات.

### 6.2. نقاط نهاية API المستخدمة في الاختبار

| الغرض | الطريقة | المسار |
|-------|--------|--------|
| المصادقة | POST | `/qasemipay/test-auth` |
| إنشاء معاملة | POST | `/qasemipay/test-create-payment` |
| تأكيد معاملة | POST | `/qasemipay/test-confirm-payment` |
| التحقق من الحالة | GET | `/qasemipay/test-check-status?ref_id=...` |
| اختبار شامل | POST | `/qasemipay/test-full-payment` |
| إحصائيات | GET | `/qasemipay/stats` |

---

## 7. أمثلة على الطلبات والاستجابات

### 7.1. إنشاء معاملة ناجحة

**طلب:**
```http
POST /api/v1/yepayment/qasemipay/test-create-payment
Content-Type: application/json

{
    "order_id": 200,
    "purchase_code": "PC-987654321",
    "mobile_number": "+967700000000",
    "amount": 100,
    "currency": "YER",
    "zone": "Z1"
}
```

**استجابة:**
```json
{
    "success": true,
    "message": "تم إنشاء المعاملة بنجاح",
    "data": {
        "transaction_id": "d625aefc-7204-f951-c33c-3a1d09bcc8da",
        "ref_no": "87c5dfeb-d57f-494e-86ad-96ed5a6d814b",
        "requires_confirmation": true,
        "concurrency_stamp": "stamp_string_here",
        "ref_id": "123e4567-e89b-12d3-a456-426614174000"
    }
}
```

### 7.2. تأكيد المعاملة ناجح

**طلب:**
```http
POST /api/v1/yepayment/qasemipay/test-confirm-payment
{
    "order_id": 200,
    "transaction_id": "d625aefc-7204-f951-c33c-3a1d09bcc8da",
    "concurrency_stamp": "stamp_string_here"
}
```

**استجابة:**
```json
{
    "success": true,
    "message": "تم تأكيد الدفع بنجاح",
    "data": {
        "ref_no": "028030a3-d748-4ead-babd-89a43caef612",
        "completed": true
    }
}
```

### 7.3. التحقق من الحالة

**طلب:**
```http
GET /api/v1/yepayment/qasemipay/test-check-status?ref_id=123e4567-e89b-12d3-a456-426614174000
```

**استجابة:**
```json
{
    "success": true,
    "message": "تم التحقق من حالة المعاملة",
    "data": {
        "status_code": 2,
        "status": "Succeeded",
        "ref_id": "123e4567...",
        "ref_no": "028030a3...",
        "operation_number": 483287097411584
    }
}
```

---

## 8. دمج البوابة في مشروعك (Developer Guide)

### 8.1. تسجيل طريقة الدفع في `Plugin.php`

```php
public function registerPaymentProviders()
{
    $providers = [];
    // ... البوابات الأخرى ...
    if (class_exists(\Nano\Yepayment\PaymentTypes\QasemiPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\QasemiPay();
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
$gateway->init($paymentMethod, $userInputData);
$result = $gateway->process($order);
if ($result->successful && $result->redirect) {
    return redirect()->to($result->redirectUrl);
}
```

### 8.3. تأكيد الدفع بعد عودة المستخدم

```php
$paymentProvider = new QasemiPay();
$paymentProvider->setOrder($order);
$result = new PaymentResult($paymentProvider, $order);
$completeResult = $paymentProvider->complete($result);
if ($completeResult->successful) {
    // الدفع تم بنجاح
}
```

---

## 9. ملفات الكود الخاصة بالبوابة

| الملف | الوصف |
|-------|-------|
| `QasemiPay.php` | كلاس البوابة الرئيسي |
| `qasemi-ui.htm` | واجهة اختبار HTML/JS |
| `_info.htm` | معلومات العرض في الإعدادات |
| `_test_info.htm` | أدوات الاختبار السريع في الإعدادات |

**موقع الملفات داخل الإضافة:**  
`plugins/nano/yepayment/paymenttypes/qasemipay/`

---

## 10. استكشاف الأخطاء (Troubleshooting)

| المشكلة | السبب المحتمل | الحل |
|---------|---------------|------|
| فشل المصادقة | Client ID/Secret أو username/password غير صحيحة | تأكد من البيانات في الإعدادات |
| `concurrencyStamp` مفقود | لم يتم حفظه بعد إنشاء المعاملة | تأكد من نجاح `process()` قبل استدعاء `complete()` |
| الحالة `Failed` عند التأكيد | `concurrencyStamp` قديم أو المعاملة غير قابلة للتأكيد | أعد إنشاء معاملة جديدة |
| `refId` غير موجود | لم يتم توليده أثناء الإنشاء | تحقق من استجابة API، قد يكون الحقل مختلفاً |
| خطأ `403 Forbidden` | التوكن منتهي الصلاحية أو صلاحيات غير كافية | أعد طلب توكن جديد (يقوم به الكلاس تلقائياً) |

---

## 11. ملاحظات إضافية

- **لا تحتوي وثيقة API المرفقة على آلية OTP** – لذلك فإن خطوة التأكيد تعتمد فقط على `concurrencyStamp`. إذا كان المصرف يطلب OTP خارجياً، يمكن تعديل `complete()` لقراءة `otp` من `Request::input('otp')` وإرساله في جسم التأكيد إن كان مدعوماً.
- **تخزين التوكن مؤقتاً** – لتحسين الأداء، يمكن إضافة طبقة Cache لتخزين التوكن لمدة 3600 ثانية بدلاً من طلبه في كل مرة.
- **التوافق مع المطورين** – كلاس `QasemiPay` مستقل ويمكن استخدامه خارج `Nano.MicroCart` إذا تم توفير كائن `Order` مناسب.


---

**تم إعداد هذا التوثيق لدعم المطورين في دمج واستخدام بوابة QasemiPay بسهولة وكفاءة.**  
للاستفسارات أو الدعم الفني، يرجى التواصل عبر الموقع الرسمي nano2soft.com.