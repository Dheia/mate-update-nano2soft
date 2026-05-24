## 2026-05-18 – 2026-05-24

**إضافة وتحديث بوابة دفع FloosakPay (محفظة فلوسك) ضمن الوحدة البرمجية `Nano.Yepayment`**

---

## 1. مقدمة

تم تطوير طريقة دفع جديدة باسم **FloosakPay** وإضافتها إلى نظام المدفوعات الموحد في نانوسوفت ضمن الوحدة `Nano.Yepayment`. تأتي هذه الإضافة استجابة للحاجة إلى دعم بوابات الدفع عبر المحافظ الرقمية في اليمن، حيث تُعد **محفظة فلوسك (Floosak Wallet)** أحد الحلول المالية الرقمية الرائدة التي تتيح للمستخدمين إجراء المدفوعات الإلكترونية بسهولة وأمان.

تعتمد FloosakPay على واجهة برمجة تطبيقات **Floosak Wallet API** (الإصدار v1)، والتي تتيح:
- مصادقة التاجر عبر `/api/v1/auth/login` والحصول على `Bearer token`.
- إنشاء عملية شراء معلقة (`pending purchase`) عبر `/api/v1/merchant/p2mcl`.
- تأكيد الدفع باستخدام رمز OTP عبر `/api/v1/merchant/p2mcl/confirm`.
- الاستعلام عن حالة المعاملة عبر `/api/v1/merchant/check-status`.
- استرداد المبلغ (refund) عبر `/api/v1/merchant/p2mcl/refund`.

تم تصميم البوابة وفق **نمط الدفع من خطوتين (Two‑Step) مع تأكيد OTP**، حيث:
1. **الخطوة الأولى (`process`):** يتم إنشاء معاملة معلقة (`Send`)، وإرسال OTP إلى جوال العميل، وتُعاد بيانات المعاملة (`purchase_id`). **لا يتغير** الطلب إلى حالة مدفوع.
2. **الخطوة الثانية (`complete`):** بعد إدخال العميل للرمز، يتم تأكيد الدفع باستخدام OTP. إذا كان الرمز صحيحاً، يتم تحديث الطلب إلى `PaidState` وتفريغ السلة.

هذا التصميم يضمن التوافق الكامل مع `OrderManager` و `Checkout2` و `PaymentResult`، ويمنع تغيير حالة الطلب إلى مدفوع قبل تأكيد الرمز الصحيح من العميل.

تم تطوير البوابة وفق أعلى معايير الأمان والجودة، مع تخزين التوكنات في Cache، واستخدام `HttpHelper` الموحد لجميع طلبات API، وتوفير واجهات اختبار متكاملة ونقاط نهاية API مخصصة للمطورين والمسؤولين. كما تم دعم `RedirectHelper` لضمان التوافق مع تطبيقات الجوال التي تحتاج إلى روابط عودة بعد الدفع، وآلية `Idempotency` عبر `request_id` لمنع تكرار المعاملات، وتسجيل `PaymentLog` في المرحلة الأولى لتتبع المحاولات غير المكتملة.

---

## 2. المكونات المطورة

### 2.1. `FloosakPay` – كلاس بوابة الدفع الرئيسي

- **المسار:** `Nano\Yepayment\PaymentTypes\FloosakPay`
- **الوراثة:** يمتد من `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **الوظيفة:** مسؤول عن المصادقة عبر `/api/v1/auth/login`، إنشاء معاملة معلقة (Send)، تأكيد الدفع (Confirm)، الاستعلام عن الحالة (Check Status)، واسترداد المبلغ (Refund).

**الدوال الرئيسية:**

| الدالة | الوصف |
|--------|-------|
| `process(PaymentResult $result)` | **الخطوة الأولى:** التحقق من بيانات الإدخال (`target_phone`, `amount`, `purpose`)، التحقق من وجود معاملة سابقة (Idempotency)، الحصول على توكن، إرسال طلب Send، حفظ `request_id` و `purchase_id` في الطلب، تسجيل `PaymentLog` بحالة `initiated`، ثم إرجاع `successful = true` مع رسالة تطلب OTP (**لا تستدعي `$result->success()`**). |
| `complete(PaymentResult $result)` | **الخطوة الثانية:** استخراج `purchase_id`، الحصول على توكن جديد، إرسال طلب Confirm مع OTP، إذا نجحت وكانت الحالة `Completed` يتم استدعاء `$result->success()` لتحديث الطلب إلى `PaidState`. |
| `reverse(string $refNo, string $reason = ''): array` | استرداد مبلغ معاملة مكتملة عبر `/api/v1/merchant/p2mcl/refund`. عند النجاح، تغيير حالة الطلب إلى `RefundedState` وتسجيل بيانات الاسترداد في `other_data`. |
| `checkTransactionStatus(string $requestId): array` | الاستعلام عن حالة معاملة باستخدام `request_id` عبر `/api/v1/merchant/check-status`. تُستخدم في منع التكرار وأدوات الاختبار. |
| `getTransactionStatusForOrder(string $orderId): array` | دالة مساعدة لاستعلام الحالة باستخدام `order_id` مباشرة. |
| `getAuthToken(bool $useCache = true): ?string` | الحصول على `Bearer token` عبر `POST /api/v1/auth/login` (مع `x-channel: merchant`). يتم تخزين التوكن في Cache لمدة 3500 ثانية. |
| `sendPayment(string $token): array` | إرسال طلب `POST /api/v1/merchant/p2mcl` لإنشاء معاملة معلقة. تولد `request_id` كـ UUID. تعيد `purchase_id`, `reference_id`, `fee`, `gross`. |
| `confirmPayment(string $token, int $purchaseId, string $otp): array` | إرسال طلب `POST /api/v1/merchant/p2mcl/confirm` لتأكيد الدفع. تتحقق من أن `status.en == "Completed"`. |
| `refundPayment(string $token, int $transactionId, string $requestId, float $amount, string $reason): array` | إرسال طلب `POST /api/v1/merchant/p2mcl/refund`. |
| `settings(): array` | تعريف حقول الإعدادات في لوحة التحكم (URL, phone, password, source_wallet_id, default_purpose). |
| `encryptedSettings(): array` | تحديد الحقول التي تخزن بشكل مشفر (`floosakpay_password`). |
| `defineValidationRules(): array` | قواعد التحقق من `target_phone`, `amount`, `purpose`. |
| `handleSpecificErrors(array $responseData, string $defaultMessage): array` | استخراج رسائل الخطأ من استجابة Floosak (تدعم `errors` و `message`). |

### 2.2. آلية Idempotency (منع تكرار المعاملات)

- في `process()`، يتم البحث عن معاملة سابقة عبر `payment_first_trans_id` أو `other_data['floosakpay']['request_id']`.
- إذا وُجدت، يتم استدعاء `checkTransactionStatus()`:
  - إذا كانت الحالة `Completed` → يتم استدعاء `$result->success()` مباشرة (الطلب مدفوع مسبقاً).
  - إذا كانت الحالة `Pending` → يتم إعلام المستخدم بانتظار OTP دون إنشاء معاملة جديدة.
  - إذا كانت الحالة غير معروفة أو فاشلة → يتم إنشاء معاملة جديدة.
- هذا يضمن عدم إنشاء معاملات مكررة عند إعادة محاولة المستخدم لعملية الدفع.

### 2.3. تسجيل `PaymentLog` في المرحلة الأولى

- بعد إنشاء المعاملة بنجاح، يتم تسجيل `PaymentLog` يدوياً عبر:
  ```php
  $paymentLog = $result->logSuccessfulPayment($sendResult, null, 'initiated');
  $this->order->payment_id = $paymentLog->id;
  $this->order->save();
  ```
- هذا يسمح بتتبع محاولات الدفع حتى لو لم يكمل العميل إدخال OTP، مما يساعد في تحليل معدل الإكمال.

### 2.4. ملفات جزئية (Partials) وإعدادات

| الملف | المسار | الوصف |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/floosakpay/_info.htm` | عرض إرشادات الإعداد ومعلومات البوابة في لوحة التحكم. |
| `_test_info.htm` | `paymenttypes/floosakpay/_test_info.htm` | أدوات اختبار سريع داخل صفحة الإعدادات (أزرار المصادقة، إنشاء معاملة، تأكيد OTP، استعلام، استرداد). |
| `floosakpay-ui.htm` | `views/floosakpay-ui.htm` | واجهة ويب تفاعلية متكاملة لاختبار جميع وظائف FloosakPay (يدوي، تلقائي، إحصائيات، سجلات). |

### 2.5. نقاط نهاية API (`routes.php`)

تم إضافة مجموعة كاملة من المسارات ضمن مجموعة `yepayment` لدعم الاختبار والمراقبة:

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/floosakpay/test-auth` | POST | اختبار المصادقة والحصول على Bearer Token. |
| `/floosakpay/test-create-payment` | POST | إنشاء معاملة معلقة (محاكاة `process`). |
| `/floosakpay/test-confirm-payment` | POST | تأكيد الدفع باستخدام OTP (محاكاة `complete`). |
| `/floosakpay/test-check-status` | GET | الاستعلام عن حالة معاملة باستخدام `request_id` أو `order_id`. |
| `/floosakpay/test-refund` | POST | استرداد مبلغ معاملة مكتملة. |
| `/floosakpay/test-full-payment` | POST | اختبار شامل (إنشاء + تأكيد + استعلام) في خطوة واحدة. |
| `/floosakpay/stats` | GET | إحصائيات استخدام البوابة (عدد الطلبات، نسبة النجاح، السجلات الحديثة). |
| `/floosakpay/logs` | GET | سجلات المعاملات من قاعدة البيانات (للمسؤول). |
| `/floosakpay/test-ui` | GET | واجهة الاختبار المتكاملة (HTML). |
| `/floosakpay/success` | GET | نقطة نهاية لتأكيد الدفع بعد إدخال OTP (تستخدم `checkAndCompletePay`). |
| `/floosakpay/cancel` | GET | نقطة نهاية اختيارية لإعادة التوجيه عند الإلغاء. |

جميع المسارات (باستثناء success/cancel) محمية بـ `BackendAuth::getUser()` لضمان وصول المسؤولين فقط.

### 2.6. `Plugin.php` – تسجيل مزود الدفع

تمت إضافة السطر التالي في دالة `registerPaymentProviders()` ضمن قسم `allow_yemen_payment`:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\FloosakPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\FloosakPay();
}
```

### 2.7. ملفات اللغة (Translation)

تم إضافة مفاتيح الترجمة في `lang/ar/lang.php` و `lang/en/lang.php` لكل من الإعدادات ورسائل العموم والأخطاء (مثل `auth_success`, `confirmation_required`, `payment_success`, `order_already_paid`, `auth_failed`, `missing_credentials`, `missing_wallet_id`).

---

## 3. دورة عمل الدفع (Payment Workflow) – نمط Two‑Step (OTP)

تعمل FloosakPay وفق تدفق **Two‑Step** مع تأكيد OTP:

### 3.1. المصادقة (Authentication)

يتم استدعاء `getAuthToken()`:
- التحقق من وجود `access_token` في Cache (صلاحية 3500 ثانية).
- إذا لم يكن موجوداً، يتم إرسال طلب `POST /api/v1/auth/login` مع `phone` و `password` و `x-channel: merchant`.
- يتم تخزين التوكن في Cache لمدة أقل بقليل من صلاحيته الفعلية.

### 3.2. الخطوة الأولى – إنشاء معاملة معلقة (`process`)

- التحقق من صحة البيانات (`target_phone`, `amount`, `purpose`) باستخدام `Validator`.
- التأكد من أن الطلب ليس مدفوعاً مسبقاً (`payment_state != PaidState`).
- **التحقق من وجود معاملة سابقة (Idempotency):**
  - البحث عن `request_id` في `payment_first_trans_id` أو `other_data['floosakpay']['request_id']`.
  - إذا وُجد، استدعاء `checkTransactionStatus()`:
    - إذا `Completed` → استدعاء `$result->success()` (الطلب مدفوع مسبقاً).
    - إذا `Pending` → إرجاع رسالة تطلب OTP بدون إنشاء معاملة جديدة.
    - إذا غير ذلك → متابعة الإنشاء.
- **لا توجد معاملة سابقة أو الحالة غير صالحة:**
  - الحصول على `access_token`.
  - إرسال طلب `POST /api/v1/merchant/p2mcl` مع:
    - `source_wallet_id` (من الإعدادات)
    - `request_id` (UUID فريد)
    - `target_phone`, `amount`, `purpose`
  - استلام `purchase_id`, `reference_id`, `fee`, `gross`.
  - حفظ `request_id` في `order.payment_first_trans_id`.
  - حفظ `purchase_id` في `order.payment_trans_id`.
  - تخزين البيانات الإضافية في `order.other_data['floosakpay']` (الرقم، المبلغ، الغرض، روابط العودة، إلخ).
  - **لا يتم استدعاء `$result->success()`**، ولا يتغير الطلب إلى `PaidState`.
  - تسجيل `PaymentLog` بحالة `'initiated'`.
  - إرجاع `$result->successful = true` مع رسالة "يرجى إدخال رمز التأكيد".

### 3.3. الخطوة الثانية – تأكيد الدفع (`complete`)

- تُستدعى من مسار `floosakpay/success` بعد أن يدخل العميل OTP.
- استخراج `purchase_id` من `order.payment_trans_id`.
- الحصول على `access_token` جديد (قد يكون منتهياً).
- إرسال طلب `POST /api/v1/merchant/p2mcl/confirm` مع `otp` و `purchase_id`.
- تحليل الاستجابة:
  - إذا كان `is_success == true` و `status.en == "Completed"`:
    - يتم استدعاء `$result->success()` التي:
      - تغير `payment_state` إلى `PaidState`.
      - تضع `processed = true`.
      - تُخطر النظام بالأحداث المرتبطة (إرسال إيميل، تحديث المخزون، تفريغ السلة).
  - إذا كان `status.en == "Pending"` → إرجاع رسالة بأن الرمز غير صحيح أو المعاملة لا تزال معلقة.
- أي حالة أخرى تعتبر فشلاً.

### 3.4. روابط العودة الاختيارية (Callbacks)

عند تمرير `callback_success_url` أو `callback_error_url` في بيانات الدفع، يتم تخزينها في `other_data`. بعد التأكيد، يمكن للمطور استخدام `RedirectHelper` في مسار `success` لإعادة توجيه المستخدم بذكاء (Deep Link أو ويب) مع تمرير نتيجة الدفع.

---

## 4. الإعدادات والتهيئة (Configuration)

لتفعيل FloosakPay، يجب إدخال الإعدادات التالية في واجهة إعدادات بوابة الدفع في نظام نانوسوفت (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| الإعداد | المفتاح | الوصف | القيمة الافتراضية |
|---------|---------|-------|-------------------|
| رابط API الأساسي | `floosakpay_url` | عنوان API لبيئة الإنتاج | `https://staging.fintech-expert.net` |
| رابط API للاختبار | `floosakpay_test_url` | عنوان API لبيئة الاختبار (اختياري) | `https://staging.fintech-expert.net` |
| رقم هاتف التاجر | `floosakpay_phone` | رقم الجوال المستخدم في المصادقة (مع رمز الدولة) | - |
| كلمة المرور | `floosakpay_password` | كلمة مرور حساب التاجر (تخزن مشفرة) | - |
| معرف المحفظة المصدر | `floosakpay_source_wallet_id` | معرف المحفظة التي سيتم السحب منها | - |
| الغرض الافتراضي | `floosakpay_default_purpose` | نص يُرسل كوصف للمعاملة | `دفع الطلب` |

**ملاحظات أمنية:**
- `floosakpay_password` يتم تخزينه مشفراً عبر `encryptedSettings()`.
- لا يتم تسجيل هذه القيم الحساسة في السجلات.

---

## 5. الأمثلة التوضيحية (Usage Examples)

### 5.1. بدء الدفع – الخطوة الأولى (إنشاء معاملة معلقة)

```php
use Nano\Yepayment\PaymentTypes\FloosakPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$floosak = new FloosakPay($order, [
    'target_phone' => '967771234567',
    'amount'       => 100,
    'purpose'      => 'دفع الطلب #200',
    'callback_success_url' => 'myapp://pay/success',
]);
$result = new PaymentResult($floosak, $order);
$processResult = $floosak->process($result);

if ($processResult->successful && !$processResult->redirect) {
    // عرض واجهة إدخال OTP للمستخدم
    $purchaseId = $order->payment_trans_id;
    $requestId  = $order->payment_first_trans_id;
}
```

### 5.2. تأكيد الدفع – الخطوة الثانية (إدخال OTP)

بعد أن يدخل العميل الرمز، يمكن تأكيد العملية كالتالي:

```php
$floosak = new FloosakPay($order, ['otp' => '123456']);
$result = new PaymentResult($floosak, $order);
$completeResult = $floosak->complete($result);

if ($completeResult->successful) {
    // تم تأكيد الدفع وتحديث الطلب إلى PaidState
}
```

أو باستخدام الدالة الثابتة `checkAndCompletePay`:

```php
$result = \Nano\Yepayment\PaymentTypes\FloosakPay::checkAndCompletePay(['order_id' => 200, 'otp' => '123456']);
if ($result['success']) {
    // نجاح
}
```

### 5.3. الاستعلام المباشر عن حالة معاملة

```php
$floosak = new FloosakPay();
$status = $floosak->checkTransactionStatus('550e8400-...');
if ($status['success'] && $status['completed']) {
    echo "المعاملة مكتملة";
}
```

### 5.4. استرداد مبلغ معاملة

```php
$order = Order::find(200);
$floosak = new FloosakPay($order);
$refundResult = $floosak->reverse($order->payment_first_trans_id, 'استرداد بناءً على طلب العميل');
if ($refundResult['success']) {
    // تم الاسترداد، حالة الطلب أصبحت RefundedState
}
```

### 5.5. استخدام واجهة الاختبار المتكاملة

بعد تسجيل الدخول إلى لوحة التحكم كمسؤول، افتح الرابط:
```
https://yourdomain.com/api/v1/yepayment/floosakpay/test-ui
```

الواجهة تتيح:
- اختبار المصادقة (التحقق من صحة phone, password, source_wallet_id).
- إنشاء معاملة معلقة (محاكاة `process`) وعرض `purchase_id` و `request_id`.
- تأكيد الدفع باستخدام OTP (يدوياً أو افتراضي `123456` في بيئة الاختبار).
- الاستعلام عن حالة معاملة.
- استرداد المبلغ.
- اختبار شامل (إنشاء + تأكيد + استعلام) في خطوة واحدة.
- اختبار تلقائي بعدد مرات محدد (حتى 5 مرات).
- إحصائيات الاستخدام (عدد الطلبات، نسبة النجاح، السجلات الحديثة).
- سجلات محلية للاختبارات (تخزن في `localStorage`).

---

## 6. التعامل مع بيئة الاختبار (Sandbox)

- في بيئة الاختبار، يتم تعيين `x-channel: merchant` كما في الإنتاج.
- قيمة OTP ثابتة: **`123456`** لأغراض الاختبار (لا يتم إرسال رسالة SMS حقيقية).
- يمكن استخدام رابط الاختبار `https://staging.fintech-expert.net` (يُحدد في حقل `floosakpay_test_url`).
- تأكد من أن `source_wallet_id` صالح في بيئة الاختبار.

---

## 7. القيمة المضافة

- **للمطورين:** نموذج متكامل لبوابة دفع من نوع **Two‑Step** باستخدام OTP، مع آلية Idempotency، وتخزين Cache للتوكنات، وتسجيل PaymentLog في المرحلة الأولى. يُظهر كيفية التعامل مع واجهات API التي تتطلب `x-channel` وجسم JSON قياسي.
- **للتجار:** دعم محفظة فلوسك كطريقة دفع رقمية، مما يتيح للعملاء الدفع مباشرة من محافظهم الإلكترونية دون الحاجة إلى بطاقات ائتمان.
- **للمستخدمين النهائيين:** تجربة دفع سلسة عبر OTP، حيث يتلقى العميل رمزاً على هاتفه ويؤكده داخل التطبيق، مما يعزز الأمان.

---

## 8. اختبار التكامل (Integration Testing)

تم توفير عدة طبقات للاختبار:

- **بيئة الاختبار:** يمكن استخدام رابط `staging.fintech-expert.net` (يُحدد في حقل `floosakpay_test_url`). OTP ثابت `123456`.
- **واجهة الاختبار السريع:** من خلال صفحة إعدادات البوابة (الملف الجزئي `_test_info.htm`) يمكن اختبار المصادقة، إنشاء معاملة، تأكيد، واستعلام مباشرة.
- **واجهة الاختبار المتكاملة:** `/api/v1/yepayment/floosakpay/test-ui` توفر جميع الأدوات اللازمة لاختبار البوابة بشكل كامل (يدوي، تلقائي، إحصائيات، سجلات).
- **نقاط نهاية API مستقلة:** يمكن للمطورين استخدام أدوات مثل Postman أو cURL للاتصال المباشر بالنقاط مثل `/floosakpay/test-create-payment` و `/floosakpay/test-confirm-payment`.

**خطوات اختبار سريعة:**
1. تأكد من إعداد `phone`, `password`, `source_wallet_id` في صفحة إعدادات البوابة.
2. افتح `/api/v1/yepayment/floosakpay/test-ui`.
3. انقر على "اختبار المصادقة" للتحقق من صحة البيانات.
4. أدخل رقم طلب ورقم جوال العميل ومبلغ، ثم انقر "إنشاء معاملة".
5. سيظهر `purchase_id` ورسالة "يرجى إدخال OTP". استخدم OTP `123456`.
6. انقر "تأكيد الدفع" – يجب أن يعود بنجاح وتتغير حالة الطلب إلى مدفوع.
7. جرب "الاستعلام عن الحالة" و "استرداد المبلغ".

---

## 9. ملاحظات للمطورين (Developer Notes)

- **نمط Two‑Step:** `process()` **لا** يُكمل الدفع، بل ينشئ معاملة معلقة ويعيد رسالة تطلب OTP. يجب استدعاء `complete()` (أو `checkAndCompletePay`) بعد إدخال العميل للرمز لتأكيد الدفع وتحديث الطلب إلى `PaidState`.
- **Idempotency:** يتم التحقق من وجود معاملة سابقة في بداية `process()` لمنع تكرار الإنشاء. هذا مهم جداً عند إعادة تحميل صفحة الدفع أو الضغط على زر "ادفع" عدة مرات.
- **التخزين المؤقت للتوكن:** يتم تخزين Bearer Token في Cache لمدة 3500 ثانية (أقل من صلاحية التوكن الفعلية البالغة 3600 ثانية) لتجنب استخدام توكن منتهي.
- **استخدام `HttpHelper`:** جميع طلبات API تستخدم `HttpHelper::sendJson()` مع الهيدرز المناسبة (`x-channel: merchant`).
- **تسجيل `PaymentLog`:** يتم تسجيل محاولة الدفع في المرحلة الأولى (حالة `initiated`) حتى لو لم يكمل العميل OTP، مما يساعد في تتبع وتحليل معدل الإكمال.
- **دعم `RedirectHelper`:** تم تضمين إمكانية إعادة التوجيه بشكل كامل، مما يجعل البوابة مناسبة لتطبيقات الجوال التي تستخدم Deep Links.
- **اختبار OTP:** في بيئة الاختبار، OTP ثابت `123456`. لا يتم إرسال رسالة حقيقية.
- **قابلية التوسع:** يمكن إضافة دعم Webhook أو واجهات إضافية مستقبلاً باستخدام نقاط النهاية المتاحة.

---

## 10. إصلاحات الأخطاء (Bug Fixes)

لا يوجد – هذا الإصدار مخصص لإضافة ميزة جديدة (FloosakPay).

---

## 11. فترة التطوير والاختبار

تم تطوير بوابة FloosakPay خلال الفترة من **18 مايو 2026** وحتى **24 مايو 2026**. تضمنت هذه الفترة:
- كتابة الكود الأساسي لكلاس `FloosakPay` (يمتد من `PaymentProvider`).
- تنفيذ دوال `process()`, `complete()`, `reverse()`, `getAuthToken()`, `sendPayment()`, `confirmPayment()`, `checkTransactionStatus()`.
- إضافة آلية Idempotency (التحقق من معاملة سابقة).
- إضافة تسجيل `PaymentLog` في `process()` بحالة `initiated`.
- إنشاء ملفات الواجهة الجزئية (`_info.htm`, `_test_info.htm`) وواجهة الاختبار المتكاملة (`floosakpay-ui.htm`).
- كتابة نقاط نهاية API اللازمة للاختبار والمراقبة في `routes.php` (11 نقطة نهاية).
- إضافة مفاتيح الترجمة في ملفات اللغة.
- تسجيل البوابة في `Plugin.php`.
- إجراء اختبارات تكامل شاملة للتحقق من صحة العمل مع بيئة Floosak (محاكاة باستخدام Postman و UI).

---

## 12. روابط ذات صلة

- [توثيق FloosakPay (دليل المطور)](./docs/FloosakPay/Docs-FloosakPay-ar.md)
- [وثيقة SKILL-v2.5 لإنشاء بوابات الدفع](./docs/FloosakPay/SKILL-v2.5.md)
- [وثيقة API الرسمية من Floosak](./docs/FloosakPay/external/Merchant_Wallet_Integration_API_Documentation_RefStyle.pdf)
- [مجموعة Postman لاختبار FloosakPay](./docs/FloosakPay/external/Marchant%20Last%20Version.json)
- [كلاس FloosakPay.php](./Nano/Yepayment/PaymentTypes/FloosakPay.php)
- [ملف routes.php الخاص بـ Nano.Yepayment (قسم FloosakPay)](./routes.php)
- [قناة الدعم الفني](https://nano2soft.com)

---

**تم إعداد هذا التحديث بواسطة:**  
فريق تطوير نانوسوفت – قسم المدفوعات الإلكترونية  
**المراجع:** Dheia Ali, Nano2Soft
