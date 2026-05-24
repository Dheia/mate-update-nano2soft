# توثيق بوابة الدفع FloosakPay (محفظة فلوسك) – دليل المطور

## 1. نظرة عامة

**FloosakPay** هي بوابة دفع إلكترونية مدمجة ضمن حزمة `Nano.Yepayment` في تطبيقات نانوسوفت، وتستند إلى واجهة برمجة تطبيقات **Floosak Wallet API** المقدمة من شركة فلوسك. تعمل البوابة وفق نمط **الدفع من خطوتين (Two‑Step) مع تأكيد برمز OTP**:

1. **`process()` – إنشاء معاملة معلقة (Send):**  
   يرسل التاجر طلباً إلى Floosak لإنشاء عملية شراء معلقة (`purchase`). يتم إرسال رمز OTP إلى جوال العميل، وتُعاد بيانات المعاملة بما فيها `purchase_id`. في هذه المرحلة **لا يتغير** الطلب إلى حالة "مدفوع"، ويتم فقط تخزين بيانات المعاملة وإعلام العميل بإدخال الرمز.

2. **`complete()` – تأكيد الدفع (Confirm):**  
   بعد إدخال العميل للرمز، يقوم النظام بإرسال طلب تأكيد يحوي `purchase_id` و `otp`. إذا كان الرمز صحيحاً، تصبح المعاملة `Completed` ويتم تحديث حالة الطلب إلى `PaidState`، وتفريغ السلة، وإطلاق الأحداث المرتبطة.

تدعم البوابة العملات التي يقدمها Floosak (الريال اليمني، الدولار الأمريكي، الريال السعودي). كما توفر آلية **إعادة المحاولة (Idempotency)** عبر `request_id`، وتسجيل `PaymentLog` في المرحلة الأولى لتتبع المحاولات غير المكتملة.

هذه البوابة مبنية وفق معايير `Nano.Yepayment` وتتبع النمط نفسه المستخدم في `CashPay` و `YottaPay`.

---

## 2. متطلبات التشغيل والإعدادات

### 2.1. المتطلبات الأساسية

- نظام نانوسوفت (Nano2Soft) الإصدار 2.0+
- الإضافات المطلوبة:
  - `Nano.MicroCart` (>=2.0)
  - `Nano.Yepayment` (>=1.2)
  - `Nano.Helpers`
- بيانات الدخول إلى Floosak Wallet API (مقدمة من مزود الخدمة):
  - رقم هاتف التاجر (`phone`)
  - كلمة المرور (`password`)
  - معرف المحفظة المصدر (`source_wallet_id`)
  - رابط API الأساسي (بيئة الإنتاج أو الاختبار)

### 2.2. إعدادات البوابة في لوحة التحكم

عند تفعيل طريقة الدفع **"Floosak Pay (محفظة فلوسك)"**، تظهر الحقول التالية في صفحة إعدادات المدفوعات:

| الحقل | المفتاح | الوصف | القيمة الافتراضية |
|-------|---------|-------|-------------------|
| رابط API الأساسي | `floosakpay_url` | عنوان API للإنتاج | `https://staging.fintech-expert.net` |
| رابط API للاختبار | `floosakpay_test_url` | عنوان API لبيئة الاختبار | `https://staging.fintech-expert.net` |
| رقم هاتف التاجر | `floosakpay_phone` | رقم الجوال المستخدم في المصادقة (مع رمز الدولة) | - |
| كلمة المرور | `floosakpay_password` | كلمة مرور حساب التاجر (تخزن مشفرة) | - |
| معرف المحفظة المصدر | `floosakpay_source_wallet_id` | معرف المحفظة التي سيتم السحب منها | - |
| الغرض الافتراضي | `floosakpay_default_purpose` | نص يُرسل كوصف للمعاملة | `دفع الطلب` |

> **ملاحظة أمنية:** يتم تخزين كلمة المرور بشكل مشفر عبر `encryptedSettings()`.

---

## 3. دورة حياة عملية الدفع (Payment Flow)

تعمل بوابة FloosakPay وفق نمط **Two‑Step**، وتتكون من مرحلتين منفصلتين.

### 3.1. المرحلة الأولى: إنشاء معاملة معلقة (`process`)

عند بدء الدفع لطلب معين، يتم استدعاء `process()` والتي تقوم بالخطوات التالية:

1. **التحقق من صحة البيانات** – التحقق من وجود `target_phone`، `amount`، `purpose`.
2. **التأكد من أن الطلب غير مدفوع مسبقاً** – إذا كان `payment_state` يساوي `PaidState` يتم إيقاف العملية.
3. **التحقق من وجود معاملة سابقة** – يتم البحث عن `request_id` مخزن في `payment_first_trans_id` أو `other_data['floosakpay']['request_id']`. إذا وُجد، يتم الاستعلام عن حالته عبر `checkTransactionStatus()`:
   - إذا كانت الحالة `Completed` → يتم استدعاء `$result->success()` مباشرة لتحديث الطلب إلى `PaidState`.
   - إذا كانت الحالة `Pending` → يتم إعلام المستخدم بانتظار إدخال OTP.
   - إذا كانت الحالة غير معروفة أو فاشلة → يتم إنشاء معاملة جديدة.
4. **الحصول على Access Token** عبر `POST /api/v1/auth/login` – يتم تخزينه في Cache لمدة 3500 ثانية.
5. **إرسال طلب إنشاء الدفع** `POST /api/v1/merchant/p2mcl` مع البيانات:
   - `source_wallet_id`
   - `request_id` (UUID فريد)
   - `target_phone`
   - `amount`
   - `purpose`
6. **استلام `purchase_id` و `reference_id`** من الاستجابة.
7. **حفظ بيانات المعاملة** في الطلب:
   - `order.payment_first_trans_id` = `request_id`
   - `order.payment_trans_id` = `purchase_id`
   - تخزين البيانات الإضافية في `order.other_data['floosakpay']` (رقم الهاتف، المبلغ، الغرض، روابط العودة، إلخ)
   - **لا يتم** تغيير حالة الدفع إلى `PaidState` في هذه المرحلة.
8. **تسجيل سجل الدفع** في جدول `PaymentLog` عبر `$result->logSuccessfulPayment()` بحالة `'initiated'` (حتى لو لم يكتمل الدفع بعد).
9. **إرجاع نتيجة ناجحة مع رسالة "يرجى إدخال رمز التأكيد"** – بدون استدعاء `$result->success()`.

### 3.2. المرحلة الثانية: تأكيد الدفع (`complete`)

بعد أن يُدخل العميل رمز OTP المستلم، يقوم التطبيق باستدعاء نقطة النهاية `/floosakpay/success` (أو مباشرة `complete()`). تقوم `complete()` بالخطوات التالية:

1. **استخراج `purchase_id`** من `order.payment_trans_id`.
2. **الحصول على Access Token** من جديد (أو من الكاش).
3. **إرسال طلب التأكيد** `POST /api/v1/merchant/p2mcl/confirm` مع:
   - `otp` (الرمز الذي أدخله العميل)
   - `purchase_id`
4. **تحليل الاستجابة**:
   - إذا كان `is_success = true` و `status.en = "Completed"` → يتم استدعاء `$result->success()` التي تقوم بتحديث الطلب إلى `PaidState`، وإطلاق الأحداث، وتفريغ السلة.
   - إذا كان `status.en = "Pending"` → يتم إرجاع رسالة بأن الرمز غير صحيح أو العملية لا تزال معلقة.
5. **إرجاع النتيجة النهائية** (نجاح أو فشل).

### 3.3. روابط إعادة التوجيه الاختيارية (Callbacks)

تدعم FloosakPay تخزين روابط `callback_success_url` و `callback_error_url` إذا تم تمريرها ضمن بيانات الدفع (مثلاً من تطبيق جوال). بعد تأكيد الدفع، يمكن توجيه المستخدم عبر المسارين:

- `GET /api/v1/yepayment/floosakpay/success` – يسترجع رابط العودة عند النجاح ويستخدم `RedirectHelper` لإعادة التوجيه إلى التطبيق أو الموقع.
- `GET /api/v1/yepayment/floosakpay/cancel` – مشابه لحالة الإلغاء.

هذا يضمن توافقاً كاملاً مع تطبيقات الجوال التي تحتاج إلى Deep Links بعد الدفع.

---

## 4. هيكل البيانات المخزنة في `order->other_data['floosakpay']`

```php
[
    'request_id'         => '550e8400-e29b-41d4-a716-446655440000',   // معرف الطلب الفريد (UUID)
    'purchase_id'        => 1699240,                                   // معرف الشراء من Floosak
    'source_wallet_id'   => 10000,                                     // معرف المحفظة المصدر
    'target_phone'       => '967771234567',                            // رقم جوال العميل
    'amount'             => 100.00,                                    // المبلغ
    'purpose'            => 'دفع الطلب #200',                          // الغرض
    'fee'                => 1.00,                                      // رسوم الخدمة (إن وُجدت)
    'gross'              => 101.00,                                    // الإجمالي شامل الرسوم
    'reference_id'       => '4633395571756870',                        // المرجع من Floosak
    'requires_otp'       => true,
    'created_at'         => '2025-01-01T12:00:00Z',
    'callback_success_url' => 'myapp://pay/success',                   // (اختياري)
    'callback_error_url'   => 'myapp://pay/error'                      // (اختياري)
]
```

---

## 5. سيناريو متكامل لاستخدام FloosakPay في تطبيقك

يوضّح هذا السيناريو الخطوات التي يقوم بها مطور التطبيق أو المتجر للتعامل مع FloosakPay من خلال واجهة `Checkout2` API.

### 5.1. المتطلبات الأساسية
- تطبيق جوال أو موقع يستخدم `Nano.Yepayment` و `Nano.ShopApi`.
- تم إعداد بوابة FloosakPay في لوحة التحكم (مع phone، password، source_wallet_id).
- العميل يملك حساباً في محفظة فلوسك ورقمه مسجل مسبقاً.

### 5.2. الخطوات من وجهة نظر المطور

#### **الخطوة 1: يختار المستخدم طريقة الدفع FloosakPay**
يرسل التطبيق طلباً إلى نقطة النهاية `checkout` مع `step=pay` ومعرف طريقة الدفع `floosakpay`.

**مثال الطلب:**
```http
POST /api/v1/shop/checkout
Content-Type: application/json

{
  "step": "pay",
  "payment_method_id": 5,
  "target_phone": "967771234567",
  "amount": 100,
  "purpose": "دفع الطلب",
  "callback_success_url": "myapp://checkout/success",
  "callback_error_url": "myapp://checkout/error"
}
```

#### **الخطوة 2: يعالج النظام الدفع عبر `Checkout2` و `OrderManager`**
- يستدعي النظام `OrderManager->setStepPayments()` الذي بدوره:
  1. يتحقق من صحة عناوين الشحن والكوبونات.
  2. ينشئ `PaymentService` و `PaymentGateway` بناءً على طريقة الدفع المختارة (FloosakPay).
  3. يستدعي `FloosakPay->process()`.

#### **الخطوة 3: `FloosakPay->process()` تنشئ المعاملة المعلقة**
- تتواصل البوابة مع Floosak API وتُنشئ عملية شراء معلقة، وتستلم `purchase_id`.
- يقوم النظام بحفظ `request_id` و `purchase_id` في الطلب (دون تغيير حالة الدفع).
- يستلم المطور في الـ API Response:

```json
{
  "status": true,
  "step_status": true,
  "redirect": false,
  "processed": false,
  "process_data": {
    "paymentResult": {
      "successful": true,
      "message": "يرجى إدخال رمز التأكيد المرسل إلى هاتفك",
      "api_data": {
        "purchase_id": 1699240,
        "request_id": "550e8400-..."
      }
    }
  }
}
```

- **المهم هنا:** `processed` لا تزال `false`، والتطبيق يعرض واجهة إدخال OTP.

#### **الخطوة 4: يدخل العميل رمز OTP ويؤكده**
يرسل التطبيق طلباً إلى نقطة نهاية مخصصة (مثلاً `POST /api/v1/shop/confirm-payment`) مع `order_id` و `otp`.

**مثال الطلب:**
```json
{
  "order_id": 200,
  "otp": "123456"
}
```

#### **الخطوة 5: يستدعي النظام `FloosakPay->complete()`**
- يتصل النظام بـ Floosak API لتأكيد الدفع.
- إذا نجحت العملية، يتم تحديث `order.payment_state` إلى `PaidState` وتفريغ السلة.
- يعاد إلى التطبيق استجابة النجاح.

#### **الخطوة 6: إعادة التوجيه النهائية**
- يتم توجيه المستخدم إلى صفحة نجاح الدفع (أو إلى الـ Deep Link `myapp://checkout/success`) مع بيانات الطلب.

---

## 6. كلاس FloosakPay – الطرق الأساسية

### 6.1. تعريف الكلاس

```php
namespace Nano\Yepayment\PaymentTypes;

use Nano\MicroCart\Classes\Payments\PaymentProvider;
use Nano\MicroCart\Classes\Payments\PaymentResult;
use Nano\MicroCart\Models\PaymentGatewaySettings;

class FloosakPay extends PaymentProvider
{
    // ...
}
```

### 6.2. الخصائص الأساسية

| الخاصية | النوع | الوصف |
|----------|------|-------|
| `$order` | `Order` | كائن الطلب المرتبط بالدفع |
| `$data` | `array` | البيانات الواردة من المستخدم (`target_phone`, `amount`, `purpose`, `otp`, إلخ) |
| `$success_url` | `string` | مسار النجاح (`api/v1/yepayment/floosakpay/success`) |
| `$cancel_url` | `string` | مسار الإلغاء (`api/v1/yepayment/floosakpay/cancel`) |
| `$is_test_mod` | `bool` | وضع الاختبار (يُقرأ من إعدادات البوابة) |

### 6.3. الطرق الرئيسية

| الطريقة | الوصف |
|---------|-------|
| `identifier(): string` | يعيد `'floosakpay'` |
| `name(): string` | يعيد `'Floosak Pay (محفظة فلوسك)'` |
| `process(PaymentResult $result): PaymentResult` | ينشئ معاملة معلقة (Send) ويعيد رسالة تطلب OTP. |
| `complete(PaymentResult $result): PaymentResult` | يؤكد الدفع باستخدام OTP ويحدّث حالة الطلب إلى مدفوع. |
| `reverse(string $refNo, string $reason = ''): array` | يسترد مبلغ معاملة مكتملة (Refund). |
| `checkTransactionStatus(string $requestId): array` | يستعلم عن حالة معاملة باستخدام `request_id`. |
| `getAuthToken(bool $useCache = true): ?string` | يحصل على توكن المصادقة (Bearer) من `/api/v1/auth/login`. |
| `validateSettings(): void` | يتحقق من اكتمال الإعدادات الأساسية. |
| `isAvailable(): bool` | يتحقق من صحة بيانات الاعتماد (محاولة الحصول على توكن). |

### 6.4. دوال مساعدة مهمة

| الدالة | الوصف |
|--------|-------|
| `sendPayment(string $token): array` | ينفذ طلب `POST /api/v1/merchant/p2mcl`. |
| `confirmPayment(string $token, int $purchaseId, string $otp): array` | ينفذ طلب `POST /api/v1/merchant/p2mcl/confirm`. |
| `refundPayment(string $token, int $transactionId, string $requestId, float $amount, string $reason): array` | ينفذ طلب `POST /api/v1/merchant/p2mcl/refund`. |
| `getApiUrl(string $type): string` | يبني الرابط الكامل للـ API (login, send, confirm, refund, status). |
| `getHeaders(string $token): array` | يعيد مصفوفة الهيدرز المطلوبة (`Authorization`, `x-channel: merchant`, إلخ). |
| `handleSpecificErrors(array $responseData, string $defaultMessage): array` | يستخرج رسائل الخطأ من استجابة Floosak. |
| `getSettingsByKey(string $key, $default = null)` | يجلب الإعدادات مع دعم الحقول المشفرة. |

---

## 7. نقاط نهاية الاختبار المضمنة في `routes.php`

ضمن ملف `routes.php` الخاص بـ `Nano.Yepayment`، تم توفير مجموعة من نقاط النهاية المساعدة تحت المجموعة `/api/v1/yepayment`، والمخصصة للمطورين والمسؤولين لاختبار البوابة.

### 7.1. قائمة نقاط النهاية

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/floosakpay/test-auth` | POST | اختبار المصادقة (الحصول على توكن) |
| `/floosakpay/test-create-payment` | POST | إنشاء معاملة معلقة (الخطوة الأولى) |
| `/floosakpay/test-confirm-payment` | POST | تأكيد الدفع باستخدام OTP (الخطوة الثانية) |
| `/floosakpay/test-check-status` | GET | الاستعلام عن حالة معاملة (بـ `request_id` أو `order_id`) |
| `/floosakpay/test-refund` | POST | استرداد مبلغ معاملة مكتملة |
| `/floosakpay/test-full-payment` | POST | اختبار شامل (إنشاء + تأكيد + استعلام) |
| `/floosakpay/stats` | GET | إحصائيات استخدام البوابة |
| `/floosakpay/logs` | GET | سجلات المعاملات من قاعدة البيانات |
| `/floosakpay/test-ui` | GET | واجهة ويب تفاعلية لاختبار جميع الوظائف |
| `/floosakpay/success` | GET | مسار النجاح (للعودة من التطبيق) |
| `/floosakpay/cancel` | GET | مسار الإلغاء |

### 7.2. شرح نقاط النهاية

#### `POST /floosakpay/test-auth`
لا تحتاج إلى بيانات إدخال (تستخدم الإعدادات المخزنة).  
**الاستجابة:**
```json
{
  "success": true,
  "message": "تم الحصول على التوكن بنجاح",
  "token_length": 123
}
```

#### `POST /floosakpay/test-create-payment`
**بيانات الطلب (JSON):**
```json
{
  "order_id": 200,
  "target_phone": "967771234567",
  "amount": 100,
  "purpose": "دفع الطلب"
}
```
**الاستجابة:**
```json
{
  "success": true,
  "message": "يرجى إدخال رمز التأكيد المرسل إلى هاتفك",
  "requires_otp": true,
  "purchase_id": 1699240,
  "api_data": { ... }
}
```

#### `POST /floosakpay/test-confirm-payment`
**بيانات الطلب:**
```json
{
  "order_id": 200,
  "otp": "123456"
}
```
**الاستجابة:**
```json
{
  "success": true,
  "message": "تم تأكيد الدفع بنجاح",
  "api_data": { ... }
}
```

#### `GET /floosakpay/test-check-status?request_id=...` أو `?order_id=...`
**الاستجابة:**
```json
{
  "success": true,
  "status": "Completed",
  "completed": true,
  "raw_response": { ... }
}
```

#### `POST /floosakpay/test-refund`
**بيانات الطلب:**
```json
{
  "order_id": 200,
  "reason": "استرداد بناءً على طلب العميل"
}
```
**الاستجابة:**
```json
{
  "success": true,
  "message": "Refund processed successfully",
  "refund_id": 620493
}
```

#### `POST /floosakpay/test-full-payment`
يقوم بتنفيذ خطوتين تلقائياً (إنشاء + تأكيد) مع إمكانية تمرير OTP (إذا لم يُمرر يستخدم `123456` افتراضياً).  
**الاستجابة:** تحتوي على `results` و `summary`.

#### `GET /floosakpay/stats`
**الاستجابة:**
```json
{
  "success": true,
  "stats": {
    "total_orders": 1500,
    "floosakpay_orders": 120,
    "successful_payments": 115,
    "pending_payments": 3,
    "failed_payments": 2,
    "success_rate": 95.83,
    "recent_logs": [ ... ]
  },
  "settings": {
    "url": "https://staging.fintech-expert.net",
    "phone": "9677XXXXXXX",
    "source_wallet_id": "10000"
  }
}
```

#### `GET /floosakpay/test-ui`
يعرض واجهة HTML متكاملة تحتوي على:
- اختبار يدوي (إنشاء، تأكيد، استعلام، استرداد).
- اختبار تلقائي بعدد مرات قابل للتحديد.
- إحصائيات فورية.
- سجلات الاختبارات المخزنة في LocalStorage.
- أدوات إضافية (تصدير السجلات، مسح السجلات).

---

## 8. أمثلة عملية لاستخدام الكلاس في كود مخصص

### 8.1. إنشاء دفعة جديدة (المرحلة الأولى)

```php
use Nano\Yepayment\PaymentTypes\FloosakPay;
use Nano\Orders\Models\Order;

$order = Order::find(200);
$floosak = new FloosakPay($order, [
    'target_phone' => '967771234567',
    'amount'       => 100,
    'purpose'      => 'دفع الطلب #200',
]);

$paymentResult = new \Nano\MicroCart\Classes\Payments\PaymentResult($floosak, $order);
$processResult = $floosak->process($paymentResult);

if ($processResult->successful) {
    $purchaseId = $order->payment_trans_id;
    // عرض واجهة إدخال OTP للمستخدم
} else {
    // عرض رسالة الخطأ
}
```

### 8.2. تأكيد الدفع (المرحلة الثانية)

```php
$floosak = new FloosakPay($order, ['otp' => '123456']);
$result = new PaymentResult($floosak, $order);
$completeResult = $floosak->complete($result);

if ($completeResult->successful) {
    // الدفع مكتمل – تحديث واجهة المستخدم
} else {
    // فشل التأكيد – عرض رسالة الخطأ
}
```

### 8.3. الاستعلام عن حالة معاملة

```php
$floosak = new FloosakPay();
$status = $floosak->checkTransactionStatus('550e8400-...');
if ($status['success'] && $status['completed']) {
    echo "المعاملة مكتملة";
}
```

### 8.4. استرداد المبلغ

```php
$order = Order::find(200);
$floosak = new FloosakPay($order);
$refundResult = $floosak->reverse($order->payment_first_trans_id, 'استرداد');

if ($refundResult['success']) {
    // تم الاسترداد، حالة الطلب أصبحت RefundedState
}
```

---

## 9. رموز الأخطاء الشائعة والحلول

| كود HTTP | الخطأ (من Floosak API) | السبب والحل |
|----------|------------------------|--------------|
| 401 | Unauthorized | فشل المصادقة – تحقق من `phone` و `password` في الإعدادات. |
| 400 | `"target_phone must be a valid phone number"` | رقم الهاتف غير صحيح – تأكد من إرساله مع رمز الدولة (مثال: `9677...`). |
| 400 | `"source_wallet_id not found"` | معرف المحفظة المصدر غير صحيح – تحقق من الإعدادات. |
| 400 | `"insufficient balance"` | رصيد محفظة التاجر لا يكفي لتغطية المبلغ + الرسوم. |
| 400 | `"OTP is incorrect"` | رمز التأكيد غير صحيح – اطلب من العميل إعادة المحاولة. |
| 400 | `"transaction already completed"` | تم تأكيد المعاملة مسبقاً – لا حاجة لإعادة التأكيد. |
| 404 | `"purchase not found"` | `purchase_id` غير موجود – تحقق من أن `payment_trans_id` صحيح. |
| 500 | Internal Server Error | مشكلة في خادم Floosak – حاول لاحقاً أو اتصل بالدعم. |

---

## 10. استكشاف الأخطاء (Troubleshooting)

| المشكلة | السبب المحتمل | الحل |
|---------|---------------|------|
| فشل المصادقة (`auth_failed`) | `phone` أو `password` غير صحيحين | تحقق من الإعدادات المدخلة في لوحة التحكم. |
| `target_phone` غير مقبول | الرقم لا يبدأ بـ `967` أو لا يحوي رمز الدولة | تأكد من إرسال الرقم بالصيغة الدولية الصحيحة (مثال: `967771234567`). |
| `purchase_id` غير موجود في الطلب | لم يتم حفظ `payment_trans_id` أثناء `process()` | تحقق من أن `sendPayment` نجحت وأرجعت `data.id`. |
| الطلب لا يتغير إلى مدفوع بعد التأكيد | `complete()` لم تُستدعَ أو أن OTP غير صحيح | تأكد من استدعاء نقطة النهاية `/floosakpay/success` مع `order_id` و `otp`. |
| `request_id` مكرر (Duplicate) | تم إرسال نفس `request_id` لمعاملة معلقة | استخدم UUID جديد لكل عملية إنشاء. |
| خطأ `cURL error 60` (SSL) | شهادة SSL غير موثوقة في بيئة الاختبار | أضف `'verify' => false` في `HttpHelper` (للاستخدام في الاختبار فقط). |
| `PaymentLog` لا يُسجل | استثناء أثناء حفظ السجل | افحص الـ logs: `trace_log('FloosakPay: failed to log...')`. |

---

## 11. دمج البوابة في API خارجي (للتطبيقات الأخرى)

إذا كنت تطور تطبيقاً خارجياً (مثلاً تطبيق جوال) وترغب في دمج FloosakPay دون استخدام كلاس `FloosakPay` مباشرة، يمكنك الاتصال بـ **نقاط النهاية العامة** المذكورة أعلاه بعد المصادقة عبر `oauth-users`.

### 11.1. المصادقة المسبقة

يجب أن يكون لديك توكن OAuth 2.0 صالح (يمكن الحصول عليه من نظام نانوسوفت عبر نقطة نهاية تسجيل الدخول المعتادة). ثم ترسل التوكن في الهيدر:
```
Authorization: Bearer <token>
```

### 11.2. مثال متكامل باستخدام cURL

#### أ. إنشاء معاملة معلقة
```bash
curl -X POST "https://yourdomain.com/api/v1/yepayment/floosakpay/test-create-payment" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 200,
    "target_phone": "967771234567",
    "amount": 100,
    "purpose": "دفع الطلب"
  }'
```

#### ب. تأكيد الدفع
```bash
curl -X POST "https://yourdomain.com/api/v1/yepayment/floosakpay/test-confirm-payment" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 200,
    "otp": "123456"
  }'
```

#### ج. الاستعلام عن الحالة
```bash
curl -X GET "https://yourdomain.com/api/v1/yepayment/floosakpay/test-check-status?order_id=200" \
  -H "Authorization: Bearer <token>"
```

#### د. استرداد المبلغ
```bash
curl -X POST "https://yourdomain.com/api/v1/yepayment/floosakpay/test-refund" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"order_id": 200, "reason": "استرداد"}'
```

> **ملاحظة:** نقاط النهاية هذه محمية بـ `BackendAuth` (تتطلب أن يكون المستخدم مسؤولاً). إذا كنت تريد توفيرها للعملاء العاديين، يجب تعديل `routes.php` لإزالة فحص `BackendAuth` أو إضافة middleware مخصص.

---

## 12. ملخص نقاط النهاية (مرجع سريع)

| المسار الكامل | الطريقة | الاستخدام |
|---------------|---------|-----------|
| `/api/v1/yepayment/floosakpay/test-auth` | POST | اختبار المصادقة |
| `/api/v1/yepayment/floosakpay/test-create-payment` | POST | إنشاء معاملة معلقة (Send) |
| `/api/v1/yepayment/floosakpay/test-confirm-payment` | POST | تأكيد الدفع (Confirm) |
| `/api/v1/yepayment/floosakpay/test-check-status` | GET | الاستعلام عن حالة معاملة |
| `/api/v1/yepayment/floosakpay/test-refund` | POST | استرداد المبلغ |
| `/api/v1/yepayment/floosakpay/test-full-payment` | POST | اختبار شامل (إنشاء + تأكيد) |
| `/api/v1/yepayment/floosakpay/stats` | GET | إحصائيات البوابة |
| `/api/v1/yepayment/floosakpay/logs` | GET | سجلات المعاملات |
| `/api/v1/yepayment/floosakpay/test-ui` | GET | واجهة اختبار ويب |
| `/api/v1/yepayment/floosakpay/success` | GET | مسار النجاح (للعودة من التطبيق) |
| `/api/v1/yepayment/floosakpay/cancel` | GET | مسار الإلغاء |

> **ملاحظة:** جميع هذه النقاط تتطلب أن يكون المستخدم الحالي مديراً (`BackendAuth`). لإتاحتها لعملاء API عاديين، قم بتعديل `routes.php` أو أضف middleware مخصص.

---

## 13. المراجع

- [كلاس FloosakPay.php](./FloosakPay.php) – الكود الكامل للبوابة.
- [ملف routes.php (قسم FloosakPay)](./routes.php) – تعريف نقاط النهاية الخاصة بـ FloosakPay.
- [وثيقة API الرسمية (PDF)](./Merchant_Wallet_Integration_API_Documentation_RefStyle.pdf)
- [مجموعة Postman لاختبار FloosakPay](./Marchant%20Last%20Version.json)
- [دليل تطوير بوابات الدفع – نانوسوفت (SKILL-v2.5)](./SKILL-v2.5.md)
- [ملخص المعرفة (SKILL-Summary)](./SKILL-Summary.md)

---

**تم إعداد هذا التوثيق لمساعدة المطورين على دمج واستخدام بوابة FloosakPay (محفظة فلوسك) بسهولة وكفاءة.**  
للاستفسارات أو الدعم الفني، يرجى التواصل عبر الموقع الرسمي [nano2soft.com](https://nano2soft.com).
