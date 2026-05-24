# توثيق بوابة الدفع CashPay – الدفع النقدي الإلكتروني (OTP)

## 1. نظرة عامة

**CashPay** هي بوابة دفع إلكترونية مدمجة ضمن حزمة `Nano.Yepayment` في تطبيقات نانوسوفت، وتستند إلى واجهة برمجة تطبيقات **Cash-Pay API**. تعمل البوابة وفق نمط **الدفع بخطوتين (Two‑Step Payment)** باستخدام رمز تأكيد لمرة واحدة (OTP) يتم إرساله إلى جوال العميل:

1. **`process()` – InitPayment:** يرسل التاجر طلب إنشاء معاملة إلى CashPay، ويستلم معرف فريد (`TransactionRef`). في هذه المرحلة **لا يتم خصم المبلغ**، ويتم حفظ المرجع في الطلب دون تغيير حالة الدفع.
2. **`complete()` – ConfirmPayment:** بعد أن يستلم العميل رمز OTP (عبر رسالة نصية أو تطبيق البنك)، يقوم التاجر بإرسال الرقم مع `TransactionRef` إلى CashPay لتأكيد الدفع. إذا كانت البيانات صحيحة، يتم خصم المبلغ وتحديث الطلب إلى `PaidState`.

تدعم البوابة العملات: الريال اليمني (`YER`)، الدولار الأمريكي (`USD`)، والريال السعودي (`SAR`). توفر أيضاً دوالاً إضافية للاستعلام عن حالة المعاملة (`OperationStatus`) وتغيير كلمة المرور (`ChangePass`) عند انتهاء صلاحيتها أو الحاجة إلى تحديثها.

هذه البوابة مبنية وفق معايير `Nano.Yepayment` وتتبع نمط `PaymentProvider` المشابه لـ `YottaPay` و `BasPay`.

---

## 2. متطلبات التشغيل والإعدادات

### 2.1. المتطلبات الأساسية

- نظام نانوسوفت (Nano2Soft) الإصدار 2.0+
- الإضافات المطلوبة:
  - `Nano.MicroCart` (>=2.0)
  - `Nano.Yepayment` (>=1.2)
  - `Nano.Helpers`
- بيانات الدخول إلى CashPay API (مقدمة من الشركة):
  - اسم المستخدم (`Username`)
  - كلمة المرور (`Password`)
  - مفتاح التشفير (`Encryption Key`) – يستخدم لتشفير كلمة المرور قبل إرسالها في الهيدر
  - رمز مزود الخدمة (`SpId`)
  - رابط API الأساسي (بيئة الإنتاج والاختبار)

### 2.2. إعدادات البوابة في لوحة التحكم

عند تفعيل طريقة الدفع **"CashPay (الدفع النقدي الإلكتروني)"**، تظهر الحقول التالية في صفحة إعدادات المدفوعات:

| الحقل | المفتاح | الوصف | القيمة الافتراضية |
|-------|---------|-------|-------------------|
| رابط API الأساسي | `cashpay_url` | عنوان API الخاص بـ CashPay | `https://api.cash-pay.com` |
| رابط API للاختبار | `cashpay_test_url` | عنوان API لبيئة الاختبار (UAT) | `https://test-api.cash-pay.com` |
| اسم المستخدم | `cashpay_username` | اسم مستخدم التاجر | - |
| كلمة المرور | `cashpay_password` | كلمة مرور التاجر (تُخزن مشفرة) | - |
| مفتاح التشفير (AES‑256‑CBC) | `cashpay_encryption_key` | المفتاح المستخدم لتشفير كلمة المرور في الهيدر | - |
| رمز العميل (SpId) | `cashpay_sp_id` | رمز مزود الخدمة المقدم من CashPay | - |
| العملة الافتراضية | `cashpay_default_currency_id` | العملة الافتراضية (تظهر كخيارات منسدلة) | `YER` |

> **ملاحظة أمنية:** يتم تخزين الحقول الحساسة (`cashpay_password`, `cashpay_encryption_key`) بشكل مشفر في قاعدة البيانات عبر `encryptedSettings()`.

---

## 3. دورة حياة عملية الدفع (Payment Flow)

تعمل بوابة CashPay وفق نمط **الدفع بخطوتين (Two‑Step)** باستخدام OTP.

### 3.1. المرحلة الأولى: إنشاء المعاملة (`process`)

عند بدء الدفع لطلب معين، يتم استدعاء `process()` والتي تقوم بالخطوات التالية:

1. **التحقق من صحة البيانات** – التأكد من وجود رقم الجوال (`target_msisdn`)، رمز الدفع النقدي (`customer_cash_pay_code`)، المبلغ، والعملة.
2. **التأكد من أن الطلب غير مدفوع مسبقاً** – إذا كان `payment_state` يساوي `PaidState` يتم إيقاف العملية.
3. **التحقق من وجود معاملة سابقة** – يتم البحث عن `TransactionRef` مخزن في `payment_first_trans_id` أو `other_data['cashpay']['transaction_ref']`. إذا وُجد:
   - يتم استدعاء `operationStatus()` للاستعلام عن الحالة.
   - إذا كانت الحالة `success` → يتم استدعاء `$result->success()` مباشرة (الطلب مدفوع مسبقاً).
   - إذا كانت الحالة `pending` → يتم إرجاع رسالة "أدخل OTP" دون إنشاء معاملة جديدة.
4. **إنشاء معاملة جديدة عبر `InitPayment`** – إرسال طلب `POST /api/CashPay/InitPayment` مع البيانات المطلوبة.
5. **استلام `TransactionRef`** – حفظه في `order.payment_first_trans_id` و `order.payment_trans_id`.
6. **حفظ بيانات المعاملة** في `order.other_data['cashpay']` (المبلغ، العملة، رقم الجوال، رمز الدفع، تاريخ الإنشاء، روابط العودة).
7. **تسجيل سجل الدفع** في جدول `PaymentLog` عبر `$result->logSuccessfulPayment()` (حتى لو لم يُكمل).
8. **إرجاع نجاح جزئي** مع رسالة تطلب من المستخدم إدخال OTP.

> **ملاحظة للمطور:** في هذه المرحلة، حالة الطلب **لا تتغير** (`processed = false`). النظام ينتظر إدخال OTP من العميل.

### 3.2. المرحلة الثانية: تأكيد الدفع (`complete`)

عند حصول التاجر على OTP من العميل (يُدخل في واجهة الدفع)، يتم استدعاء `complete()` والتي تقوم بالخطوات التالية:

1. **استخراج `TransactionRef`** من `order.payment_first_trans_id`.
2. **الحصول على OTP** من البيانات المدخلة (`$this->data['otp']`).
3. **إرسال طلب `ConfirmPayment`** إلى `POST /api/CashPay/ConfirmPayment` مع `TransactionRef` و `TRCode = md5(TransactionRef + OTP)`.
4. **تحليل الاستجابة**:
   - إذا كان `ResultCode == 1` → يتم استدعاء `$result->success()` التي تقوم بتحديث الطلب إلى `PaidState`، وإطلاق الأحداث، وتفريغ السلة.
   - إذا كان `ResultCode == 6022` → يعني ضرورة تغيير كلمة المرور (يُرسل خطأ مناسب).
   - أي رمز آخر → يُعتبر فشلاً في الدفع.
5. **إرجاع النتيجة النهائية** (نجاح أو فشل).

### 3.3. دوال إضافية

#### `operationStatus(string $requestId, string $type = 'InitOP'): array`
يستعلم عن حالة معاملة باستخدام `RequestID` (الذي يُرسل في `InitPayment`). النوع `InitOP` للمعاملة الأولية، و `PayOP` للتأكيد. يُستخدم داخلياً للتحقق من المعاملات السابقة.

#### `changePassword(string $newPassword): array`
يغيّر كلمة المرور الخاصة بالتاجر لدى CashPay. يجب استدعاؤها عند تلقي رمز الخطأ `6022` (كلمة المرور منتهية الصلاحية). تُشفّر كلمة المرور الجديدة باستخدام `encryptAes256Cbc()` ثم تُرسل إلى `POST /api/CashPay/ChangePass`.

### 3.4. روابط إعادة التوجيه الاختيارية (Callbacks)

تدعم CashPay تخزين روابط `callback_success_url` و `callback_error_url` إذا تم تمريرها ضمن بيانات الدفع (مثلاً من تطبيق جوال). بعد تأكيد الدفع، يمكن للمطور استخدام المسارين:

- `GET /api/v1/yepayment/cashpay/success` – يسترجع رابط العودة عند النجاح ويستخدم `RedirectHelper` لإعادة التوجيه إلى التطبيق أو الموقع.
- `GET /api/v1/yepayment/cashpay/cancel` – مشابه لحالة الإلغاء.

هذا يضمن توافقاً كاملاً مع تطبيقات الجوال التي تحتاج إلى Deep Links بعد الدفع.

---

## 4. هيكل البيانات المخزنة في `order->other_data['cashpay']`

```php
[
    'transaction_ref'     => '1234567890',                  // المرجع الفريد من CashPay
    'amount'              => 100.00,                        // المبلغ
    'currency_id'         => 2,                             // رقم العملة (حسب النظام)
    'currency_code'       => 'YER',                         // رمز العملة (ISO 4217)
    'target_msisdn'       => '771234567',                   // رقم جوال العميل
    'customer_cash_pay_code' => 555,                        // رمز الدفع النقدي
    'desc'                => 'دفع الطلب رقم 200',           // وصف المنتجات
    'created_at'          => '2025-01-01T12:00:00Z',       // وقت الإنشاء
    'requires_otp'        => true,                          // يتطلب OTP
    'callback_success_url'=> 'myapp://pay/success',         // (اختياري) رابط العودة عند النجاح
    'callback_error_url'  => 'myapp://pay/error'            // (اختياري) رابط العودة عند الخطأ
]
```

---

## 5. سيناريو متكامل لاستخدام CashPay في تطبيقك

### 5.1. المتطلبات الأساسية
- تطبيق جوال أو موقع يستخدم `Nano.Yepayment` و `Nano.ShopApi`.
- تم إعداد بوابة CashPay في لوحة التحكم (مع Username, Password, Encryption Key, SpId).
- العميل مسجل في نظام CashPay ولديه رمز دفع نقدي (`CashPay Code`) يتم إدخاله أثناء الدفع.

### 5.2. الخطوات من وجهة نظر المطور

#### **الخطوة 1: يختار المستخدم طريقة الدفع CashPay**
يرسل التطبيق طلباً إلى نقطة النهاية `checkout` مع `step=pay` ومعرف طريقة الدفع `cashpay`.

**مثال الطلب:**
```http
POST /api/v1/shop/checkout
Content-Type: application/json

{
  "step": "pay",
  "payment_method_id": 5,
  "target_msisdn": "771234567",
  "customer_cash_pay_code": 555,
  "amount": 100,
  "currency": "YER",
  "desc": "دفع الطلب",
  "callback_success_url": "myapp://checkout/success",
  "callback_error_url": "myapp://checkout/error"
}
```

#### **الخطوة 2: يعالج النظام الدفع عبر `Checkout2` و `OrderManager`**
- يستدعي النظام `OrderManager->setStepPayments()` الذي بدوره:
  1. يتحقق من صحة عناوين الشحن والكوبونات.
  2. ينشئ `PaymentService` و `PaymentGateway` بناءً على طريقة الدفع المختارة (CashPay).
  3. يستدعي `CashPay->process()`.

#### **الخطوة 3: `CashPay->process()` تنشئ المعاملة وتُعيد `TransactionRef`**
- تتواصل البوابة مع CashPay API وتُنشئ معاملة جديدة.
- يستلم المطور في الـ API Response:
  ```json
  {
    "status": true,
    "step_status": true,
    "payment_state": null,
    "processed": false,
    "process_data": {
      "paymentResult": {
        "successful": true,
        "message": "يرجى إدخال رمز التأكيد (OTP) المرسل إلى جوالك",
        "api_data": {
          "transaction_ref": "1234567890",
          "requires_otp": true
        }
      }
    }
  }
  ```
- **المهم هنا:** `processed` لا تزال `false`، والطلب لم يُدفع.

#### **الخطوة 4: يطلب التطبيق من العميل إدخال OTP**
- يعرض التطبيق حقل إدخال OTP ويطلب من العميل إدخال الرقم الذي تلقاه على جواله.
- بعد إدخال OTP، يرسل التطبيق طلباً إلى نقطة نهاية مخصصة (أو يستخدم `complete` مباشرة).

#### **الخطوة 5: تأكيد الدفع باستخدام OTP**
يرسل التطبيق طلباً إلى `/api/v1/yepayment/cashpay/test-confirm-payment` أو نقطة نهاية مخصصة تستدعي `complete()`:

**مثال الطلب:**
```http
POST /api/v1/yepayment/cashpay/confirm
Content-Type: application/json
{
  "order_id": 200,
  "otp": "1234"
}
```

#### **الخطوة 6: يتغير الطلب إلى مدفوع**
- إذا نجحت عملية `ConfirmPayment`، يستدعي `$result->success()` الذي:
  - يغير `payment_state` إلى `PaidState`.
  - يضع `processed = true`.
  - يُخطر النظام بالأحداث المرتبطة (إرسال إيميل، تحديث المخزون، إلخ).
  - يفرغ سلة المستخدم.

#### **الخطوة 7: إعادة التوجيه النهائية**
- يتم توجيه المستخدم إلى صفحة نجاح الدفع (أو إلى الـ Deep Link `myapp://checkout/success`) مع بيانات الطلب.

---

## 6. واجهة اختبار API (Test UI)

توفر البوابة واجهة ويب متكاملة لاختبار جميع الوظائف، يمكن الوصول إليها عبر:

```
/api/v1/yepayment/cashpay/test-ui
```

### 6.1. الميزات

- **اختبار المصادقة (Auth)** – التحقق من صحة الإعدادات (Username, Password, Encryption Key, SpId).
- **إنشاء معاملة (InitPayment)** – إدخال رقم الطلب، رقم الجوال، رمز الدفع، المبلغ، العملة لإنشاء معاملة واستلام `TransactionRef`.
- **تأكيد الدفع (ConfirmPayment)** – إدخال OTP لتأكيد الدفع وتحديث الطلب إلى مدفوع.
- **الاستعلام عن الحالة (OperationStatus)** – التحقق من حالة معاملة باستخدام `RequestID`.
- **تغيير كلمة المرور (ChangePass)** – تغيير كلمة المرور عند الحاجة.
- **اختبار شامل (Full Flow)** – يجمع بين إنشاء المعاملة وتأكيدها في خطوة واحدة.
- **إحصائيات فورية** – عدد الطلبات، نسبة النجاح، آخر السجلات.
- **سجلات محلية** – تخزن نتائج الاختبارات في `localStorage` للمتصفح.

### 6.2. نقاط نهاية API المستخدمة في الاختبار

| الغرض | الطريقة | المسار |
|-------|--------|--------|
| المصادقة | POST | `/cashpay/test-auth` |
| إنشاء معاملة | POST | `/cashpay/test-create-payment` |
| تأكيد الدفع | POST | `/cashpay/test-confirm-payment` |
| الاستعلام عن الحالة | POST | `/cashpay/test-check-status` |
| تغيير كلمة المرور | POST | `/cashpay/test-change-password` |
| اختبار شامل | POST | `/cashpay/test-full-payment` |
| إحصائيات | GET | `/cashpay/stats` |

---

## 7. أمثلة على الطلبات والاستجابات

### 7.1. إنشاء معاملة جديدة (المرحلة الأولى – InitPayment)

**طلب:**
```http
POST /api/v1/yepayment/cashpay/test-create-payment
Content-Type: application/json

{
    "order_id": 200,
    "target_msisdn": "771234567",
    "customer_cash_pay_code": 555,
    "amount": 100,
    "currency": "YER",
    "desc": "دفع الطلب"
}
```

**استجابة (نجاح):**
```json
{
    "success": true,
    "message": "يرجى إدخال رمز التأكيد (OTP) المرسل إلى جوالك",
    "data": {
        "transaction_ref": "1234567890",
        "requires_otp": true,
        "api_data": { ... }
    }
}
```

### 7.2. تأكيد الدفع (المرحلة الثانية – ConfirmPayment)

**طلب:**
```http
POST /api/v1/yepayment/cashpay/test-confirm-payment
Content-Type: application/json

{
    "order_id": 200,
    "otp": "1234"
}
```

**استجابة (نجاح):**
```json
{
    "success": true,
    "message": "تم تأكيد الدفع بنجاح",
    "data": { ... }
}
```

### 7.3. الاستعلام عن حالة معاملة (OperationStatus)

**طلب:**
```http
POST /api/v1/yepayment/cashpay/test-check-status
Content-Type: application/json

{
    "request_id": "1234567890",
    "type": "InitOP"
}
```

**استجابة (نجاح – معاملة مكتملة):**
```json
{
    "success": true,
    "status": "success",
    "result_code": 1,
    "message": "Success",
    "data": { ... }
}
```

### 7.4. تغيير كلمة المرور (ChangePass)

**طلب:**
```http
POST /api/v1/yepayment/cashpay/test-change-password
Content-Type: application/json

{
    "new_password": "NewStrongPass123"
}
```

**استجابة (نجاح):**
```json
{
    "success": true,
    "message": "تم تغيير كلمة المرور بنجاح",
    "data": { ... }
}
```

---

## 8. دمج البوابة في مشروعك (Developer Guide)

### 8.1. تسجيل طريقة الدفع في `Plugin.php`

```php
if (config('nano.microcart::paymenttypes.allow_yemen_payment', true)) {
    // ... المزودون الآخرون ...
    if (class_exists(\Nano\Yepayment\PaymentTypes\CashPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\CashPay();
    }
}
```

### 8.2. استخدام البوابة عبر `PaymentGateway` (المرحلة الأولى فقط)

```php
use Nano\MicroCart\Classes\Payments\DefaultPaymentGateway;
use Nano\Orders\Models\Order;

$order = Order::find(200);
$gateway = new DefaultPaymentGateway();
$gateway->init($paymentMethod, [
    'target_msisdn' => '771234567',
    'customer_cash_pay_code' => 555,
    'amount' => 100,
    'currency' => 'YER',
    'desc' => 'دفع الطلب #200',
    'callback_success_url' => 'myapp://pay/success',
]);
$result = $gateway->process($order);
if ($result->successful) {
    // تم إنشاء المعاملة – نطلب من المستخدم إدخال OTP
    $transactionRef = $order->payment_first_trans_id;
}
```

### 8.3. تأكيد الدفع يدوياً (المرحلة الثانية)

بعد أن يدخل المستخدم OTP، يمكن استدعاء `complete()` مباشرة:

```php
$cash = new \Nano\Yepayment\PaymentTypes\CashPay($order, ['otp' => '1234']);
$result = new PaymentResult($cash, $order);
$completeResult = $cash->complete($result);
if ($completeResult->successful) {
    // تم الدفع بنجاح
}
```

### 8.4. الاستعلام عن حالة معاملة

```php
$cash = new CashPay();
$status = $cash->operationStatus('1234567890', 'InitOP');
if ($status['success'] && $status['status'] === 'success') {
    // المعاملة مكتملة
}
```

---

## 9. استكشاف الأخطاء (Troubleshooting)

| المشكلة | السبب المحتمل | الحل |
|---------|---------------|------|
| فشل المصادقة (`auth_failed`) | Username/Password/EncryptionKey/SpId غير صحيحة | تحقق من الإعدادات المدخلة في لوحة التحكم |
| `payment_creation_failed` مع رمز خطأ 6022 | كلمة المرور منتهية الصلاحية | استخدم دالة `changePassword()` لتغيير كلمة المرور |
| رمز الخطأ 6022 في أي طلب آخر | كلمة المرور غير صالحة | غيّر كلمة المرور عبر `changePassword()` |
| فشل تأكيد الدفع (رمز 35 أو غيره) | `OTP` غير صحيح أو `TransactionRef` غير صحيح | تأكد من OTP المرسل للعميل و `TransactionRef` المخزن |
| الاستعلام عن الحالة يعيد `failed` | المعاملة لم تُنشأ أو منتهية الصلاحية | تحقق من `RequestID` ونوع العملية (`InitOP` / `PayOP`) |
| `cURL error` أو `HttpHelper` استثناء | مشكلة في الاتصال بـ CashPay API | تحقق من صحة الرابط ووجود اتصال إنترنت |
| `missing_encryption_key` | مفتاح التشفير غير موجود في الإعدادات | تأكد من إدخال `cashpay_encryption_key` في لوحة التحكم |

---

## 10. ملاحظات إضافية

- **البوابة تتبع نمط Two‑Step (OTP)**، وبالتالي لا يتم خصم المبلغ مباشرة. يجب استدعاء `complete()` مع OTP صحيح لإتمام الدفع.
- **يتم تخزين `TransactionRef` في `order.payment_first_trans_id` و `order.payment_trans_id`** لاستخدامه في مرحلة التأكيد والاستعلام.
- **يتم تخزين روابط العودة (`callback_success_url`, `callback_error_url`)** في `other_data['cashpay']` واستخدامها بواسطة `RedirectHelper` في مسارات `success`/`cancel`.
- **إذا تلقيت رمز الخطأ 6022** في أي استجابة، يجب استدعاء `changePassword()` فوراً (يمكن إظهار رسالة للمسؤول أو توجيهه لتغيير كلمة المرور).
- **يتم تشفير كلمة المرور في هيدر الطلبات** باستخدام `encryptAes256Cbc()` ومفتاح التشفير المخصص. تأكد من أن المفتاح مطابق لما هو معطى من CashPay.
- **تُستخدم دالة `getSupportedCurrencies()` و `getCurrencyId()` و `isCurrencySupported()`** للتحقق من العملات المدعومة وتحويل رمز العملة (مثل `YER`) إلى الرقم الصحيح (مثل `2`).

---

## 11. ملفات الكود الخاصة بالبوابة

| الملف | الوصف |
|-------|-------|
| `CashPay.php` | الكلاس الرئيسي للبوابة (يمتد من `PaymentProvider`) |
| `_info.htm` | قالب معلومات الإعدادات في لوحة التحكم |
| `_test_info.htm` | أدوات الاختبار السريع مع أزرار تفاعلية (مضمنة في صفحة الإعدادات) |
| `cashpay-ui.htm` | واجهة الاختبار الكاملة (Test UI) |
| `routes.php` (قسم CashPay) | تعريف نقاط نهاية الاختبار والإحصائيات وواجهة المستخدم |
| `Plugin.php` (إضافة) | تسجيل البوابة كمزود دفع |

---

## 12. روابط ذات صلة

- [وثيقة SKILL-ar.md لإنشاء بوابات الدفع](./SKILL-ar.md)
- [وثيقة Cash-Pay API (PDF مرفق)](./Cash-Pay%20API%20Doc2.pdf)
- [كلاس CashPay.php](./CashPay.php)
- [ملف routes.php الخاص بـ Nano.Yepayment](./routes.php)

---

**تم إعداد هذا التوثيق لدعم المطورين في دمج واستخدام بوابة CashPay بسهولة وكفاءة.**  
للاستفسارات أو الدعم الفني، يرجى التواصل عبر الموقع الرسمي [nano2soft.com](https://nano2soft.com).