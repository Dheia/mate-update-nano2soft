## 2026-05-20 – 2026-05-22

**تحديث بوابة الدفع CashPay (الدفع النقدي الإلكتروني – OTP) في نظام المدفوعات بنانوسوفت**

### إضافة بوابة دفع CashPay (الدفع النقدي الإلكتروني – OTP)

ضمن الوحدة البرمجية `Nano.Yepayment`

---

## 1. مقدمة

تم تطوير طريقة دفع جديدة باسم **CashPay** وإضافتها إلى نظام المدفوعات الموحد في نانوسوفت ضمن الوحدة `Nano.Yepayment`. تأتي هذه الإضافة استجابة للحاجة إلى دعم بوابات الدفع الإلكترونية التي تعتمد على تأكيد الدفع عبر رمز OTP (One‑Time Password)، حيث تُعد **CashPay** إحدى منصات الدفع التي تتيح قبول المدفوعات عبر واجهة API مباشرة مع تشفير AES‑256‑CBC لكلمة المرور وآلية تبادل آمنة.

تعمل CashPay وفق نمط **الدفع بخطوتين (Two‑Step Payment) باستخدام OTP**، وهو النمط الذي يتوافق مع طبيعة عمل المنصة حيث:
1. **الخطوة الأولى (process):** يتم إنشاء معاملة (InitPayment) والحصول على `TransactionRef`، دون خصم المبلغ بعد. في هذه المرحلة يُطلب من العميل إدخال رمز OTP الذي سيصله لاحقاً.
2. **الخطوة الثانية (complete):** بعد أن يدخل العميل رمز OTP، يتم إرسال طلب تأكيد الدفع (ConfirmPayment) إلى CashPay. إذا كان الرمز صحيحاً، يتم خصم المبلغ وتحديث الطلب إلى "مدفوع".

هذا التصميم جعل البوابة متوافقة تماماً مع `OrderManager` و `Checkout2` و `PaymentResult`، ويمنع تغيير حالة الطلب إلى مدفوع قبل إدخال OTP الصحيح.

تم تطوير البوابة وفق أعلى معايير الأمان والجودة، مع استخدام `HttpHelper` الموحد لجميع طلبات API، وتشفير كلمة المرور في الهيدر باستخدام AES‑256‑CBC، وتوفير واجهات اختبار متكاملة ونقاط نهاية API مخصصة للمطورين والمسؤولين. كما تم دعم `RedirectHelper` لضمان التوافق مع تطبيقات الجوال التي تحتاج إلى روابط عودة بعد الدفع.

---

## 2. المكونات المطورة

### 2.1. `CashPay` – كلاس بوابة الدفع الرئيسي

- **المسار:** `Nano\Yepayment\PaymentTypes\CashPay`
- **الوراثة:** يمتد من `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **الوظيفة:** مسؤول عن المصادقة عبر الهيدر المشفر (`encPassword`)، تنفيذ الخطوة الأولى (InitPayment)، والخطوة الثانية (ConfirmPayment)، والاستعلام عن حالة المعاملة (OperationStatus)، وتغيير كلمة المرور (ChangePass).

**الدوال الرئيسية:**

| الدالة | الوصف |
|--------|-------|
| `process(PaymentResult $result)` | **الخطوة الأولى:** التحقق من وجود معاملة سابقة، إنشاء معاملة جديدة عبر `InitPayment`، استلام `TransactionRef`، حفظه في الطلب دون تغيير حالة الدفع. تُعيد نجاحاً مع رسالة "أدخل OTP". |
| `complete(PaymentResult $result)` | **الخطوة الثانية:** التحقق من `TransactionRef` و `OTP`، إرسال طلب `ConfirmPayment`، وعند النجاح استدعاء `$result->success()` لتحديث الطلب إلى `PaidState`. |
| `operationStatus(string $requestId, string $type): array` | الاستعلام عن حالة معاملة باستخدام `RequestID` (نوع `InitOP` للمعاملة الأولية، `PayOP` للتأكيد). |
| `changePassword(string $newPassword): array` | تغيير كلمة المرور الخاصة بالتاجر لدى CashPay (عند انتهاء صلاحيتها أو الحاجة إلى تحديثها). |
| `getAuthToken(): ?string` | (غير مستخدم في CashPay لأن المصادقة تعتمد على `encPassword` في الهيدر. لكن الدالة موجودة للتوافق). |
| `initPayment(): array` | بناء طلب `InitPayment` وإرساله إلى `/api/CashPay/InitPayment`، مع إرجاع `TransactionRef`. |
| `confirmPayment(string $transactionRef, string $otp): array` | بناء طلب `ConfirmPayment` مع `TRCode = md5(TransactionRef + OTP)` وإرساله إلى `/api/CashPay/ConfirmPayment`. |
| `encryptAes256Cbc(string $data, string $key): string` | تشفير نص (كلمة المرور) باستخدام AES‑256‑CBC للحصول على `encPassword`. |
| `getEncryptedPassword(): string` | تشفير كلمة المرور المخزنة باستخدام مفتاح التشفير من الإعدادات. |
| `buildHeaders(): array` | بناء هيدرات الطلبات (`encPassword`, `unixtimestamp`, `Content-Type`). |
| `settings(): array` | تعريف حقول الإعدادات في لوحة التحكم (URL, Username, Password, Encryption Key, SpId, العملة الافتراضية). |
| `encryptedSettings(): array` | تحديد الحقول التي تخزن بشكل مشفر (`cashpay_password`, `cashpay_encryption_key`). |
| `getSupportedCurrencies(): array` | قائمة العملات المدعومة مع أرقامها (YER=2, USD=1, SAR=3). |
| `getCurrencyId($currency): ?int` | تحويل رمز العملة أو رقمها إلى الرقم الصحيح المدعوم. |
| `isCurrencySupported($currency): bool` | التحقق من دعم عملة معينة. |

### 2.2. ملفات جزئية (Partials) وإعدادات

| الملف | المسار | الوصف |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/cashpay/_info.htm` | عرض إرشادات الإعداد ومعلومات البوابة في لوحة التحكم. |
| `_test_info.htm` | `paymenttypes/cashpay/_test_info.htm` | أدوات اختبار سريع داخل صفحة الإعدادات (أزرار المصادقة، إنشاء معاملة، تأكيد، استعلام، تغيير كلمة المرور، اختبار شامل). |
| `cashpay-ui.htm` | `views/cashpay-ui.htm` | واجهة ويب تفاعلية متكاملة لاختبار جميع وظائف CashPay (يدوي، تلقائي، إحصائيات، سجلات). |

### 2.3. نقاط نهاية API (`routes.php`)

تم إضافة مجموعة كاملة من المسارات ضمن مجموعة `yepayment` لدعم الاختبار والمراقبة:

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/cashpay/test-auth` | POST | اختبار صحة الإعدادات (Username, Password, Encryption Key, SpId). |
| `/cashpay/test-create-payment` | POST | إنشاء معاملة جديدة (InitPayment). |
| `/cashpay/test-confirm-payment` | POST | تأكيد الدفع (ConfirmPayment) باستخدام OTP. |
| `/cashpay/test-check-status` | POST | الاستعلام عن حالة معاملة (OperationStatus). |
| `/cashpay/test-change-password` | POST | تغيير كلمة المرور (ChangePass). |
| `/cashpay/test-full-payment` | POST | اختبار شامل (InitPayment + ConfirmPayment). |
| `/cashpay/stats` | GET | إحصائيات استخدام البوابة (عدد الطلبات، نسبة النجاح). |
| `/cashpay/test-ui` | GET | واجهة الاختبار المتكاملة (HTML). |
| `/cashpay/success` | GET | نقطة نهاية لإعادة التوجيه عند نجاح الدفع (تستخدم `RedirectHelper`). |
| `/cashpay/cancel` | GET | نقطة نهاية لإعادة التوجيه عند الإلغاء. |

جميع المسارات (باستثناء success/cancel) محمية بـ `BackendAuth::getUser()` لضمان وصول المسؤولين فقط.

### 2.4. `Plugin.php` – تسجيل مزود الدفع

تمت إضافة السطر التالي في دالة `registerPaymentProviders()` ضمن قسم `allow_yemen_payment`:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\CashPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\CashPay();
}
```

### 2.5. ملفات اللغة (Translation)

تم إضافة مفاتيح الترجمة في `lang/ar/lang.php` و `lang/en/lang.php` لكل من الإعدادات ورسائل العموم والأخطاء (مثل `payment_success`, `auth_success`, `confirmation_required`, `order_already_paid`).

---

## 3. دورة عمل الدفع (Payment Workflow) – نمط الخطوتين (OTP)

تعمل CashPay وفق تدفق **Two‑Step (خطوتين) مع OTP**:

1. **المصادقة (Authentication)**  
   لا توجد جلسة توكن منفصلة. يتم إرسال كل طلب مع هيدر `encPassword` وهو كلمة مرور التاجر مشفرة بـ AES‑256‑CBC باستخدام مفتاح التشفير المقدم.  
   تُستخدم أيضاً `unixtimestamp` (بالمللي ثانية) لمنع إعادة الهجمات.

2. **الخطوة الأولى – إنشاء المعاملة (`process`)**  
   - التحقق من أن الطلب غير مدفوع مسبقاً.
   - التحقق من وجود معاملة سابقة مرتبطة بالطلب عبر `payment_first_trans_id` أو `other_data['cashpay']['transaction_ref']`:
     - إذا وُجدت، يتم استدعاء `operationStatus()` للاستعلام عن حالتها.
     - إذا كانت الحالة `success` → يتم استدعاء `$result->success()` (الطلب مدفوع مسبقاً).
     - إذا كانت الحالة `pending` → يتم إرجاع رسالة "أدخل OTP" دون إنشاء معاملة جديدة.
   - **لا توجد معاملة سابقة أو الحالة غير صالحة:**  
     - بناء جسم الطلب (`RequestID`, `UserName`, `SpId`, `MDToken`, `TargetMSISDN`, `CustomerCashPayCode`, `Amount`, `CurrencyId`, `Desc`).
     - إرسال طلب `POST /api/CashPay/InitPayment`.
     - استلام `TransactionRef` وتخزينه في `order.payment_first_trans_id` و `order.payment_trans_id`.
     - تخزين البيانات الإضافية في `order.other_data['cashpay']` (المبلغ، العملة، رقم الجوال، رمز الدفع، تاريخ الإنشاء، روابط العودة).
     - **لا يتم استدعاء `$result->success()`**، ولا يتغير الطلب إلى `PaidState`. بدلاً من ذلك، تُرجع البوابة `successful = true` مع رسالة `confirmation_required`.
     - تسجيل `PaymentLog` عبر `$result->logSuccessfulPayment()`.

3. **الخطوة الثانية – تأكيد الدفع (`complete`)**  
   - تُستدعى بعد أن يدخل العميل OTP (يُمرر في `$this->data['otp']`).
   - تُستخرج `TransactionRef` من الطلب.
   - تُحسب `TRCode = md5(TransactionRef . OTP)`.
   - يُرسل طلب `POST /api/CashPay/ConfirmPayment` مع `TransactionRef` و `TRCode`.
   - إذا كان `ResultCode == 1`:
     - يتم استدعاء `$result->success()` التي **تغير حالة الطلب إلى `PaidState`** وتُشغل الأحداث (مثل `nano.orders.paymentProcessed`) وتُفرغ السلة.
   - إذا كان `ResultCode == 6022`: يتم إرجاع خطأ "يجب تغيير كلمة المرور".
   - أي رمز آخر يُعتبر فشلاً.

4. **روابط العودة الاختيارية**  
   عند تمرير `callback_success_url` أو `callback_error_url` في بيانات الدفع، يتم تخزينها في `other_data`. بعد التأكيد، يمكن للمطور استخدام `RedirectHelper` لإعادة التوجيه بذكاء (Deep Link أو ويب).

---

## 4. الإعدادات والتهيئة (Configuration)

لتفعيل CashPay، يجب إدخال الإعدادات التالية في واجهة إعدادات بوابة الدفع في نظام نانوسوفت (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| الإعداد | المفتاح | الوصف | القيمة الافتراضية |
|---------|---------|-------|-------------------|
| رابط API الأساسي | `cashpay_url` | عنوان API لبيئة الإنتاج | `https://api.cash-pay.com` |
| رابط API للاختبار | `cashpay_test_url` | عنوان API لبيئة UAT (اختياري) | `https://test-api.cash-pay.com` |
| اسم المستخدم | `cashpay_username` | اسم مستخدم التاجر | - |
| كلمة المرور | `cashpay_password` | كلمة مرور التاجر (تُخزن مشفرة) | - |
| مفتاح التشفير (AES‑256‑CBC) | `cashpay_encryption_key` | المفتاح المستخدم لتشفير كلمة المرور في الهيدر | - |
| رمز العميل (SpId) | `cashpay_sp_id` | رمز مزود الخدمة المقدم من CashPay | - |
| العملة الافتراضية | `cashpay_default_currency_id` | العملة الافتراضية (تظهر كخيارات منسدلة) | `YER` |

**ملاحظات أمنية:**
- `cashpay_password` و `cashpay_encryption_key` يتم تخزينهما مشفرين عبر `encryptedSettings()`.
- لا يتم تسجيل هذه القيم الحساسة في السجلات.

---

## 5. الأمثلة التوضيحية (Usage Examples)

### 5.1. بدء الدفع – الخطوة الأولى (InitPayment)

```php
use Nano\Yepayment\PaymentTypes\CashPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$cash = new CashPay($order, [
    'target_msisdn' => '771234567',
    'customer_cash_pay_code' => 555,
    'amount' => 100,
    'currency' => 'YER',
    'desc' => 'دفع الطلب #200',
    'callback_success_url' => 'myapp://pay/success',
]);
$result = new PaymentResult($cash, $order);
$processResult = $cash->process($result);

if ($processResult->successful) {
    // تم إنشاء المعاملة وحفظ TransactionRef، ينتظر النظام إدخال OTP
    $transactionRef = $order->payment_first_trans_id;
    // إظهار حقل إدخال OTP للمستخدم
}
```

### 5.2. تأكيد الدفع – الخطوة الثانية (ConfirmPayment)

بعد أن يدخل المستخدم OTP، يمكن استدعاء `complete()` مباشرة:

```php
$cash = new CashPay($order, ['otp' => '1234']);
$result = new PaymentResult($cash, $order);
$completeResult = $cash->complete($result);

if ($completeResult->successful) {
    // تم تأكيد الدفع وتحديث الطلب إلى PaidState
}
```

أو باستخدام دالة مساعدة ثابتة (عبر مسار `success`):

```php
$result = \Nano\Yepayment\PaymentTypes\CashPay::checkAndCompletePay(['order_id' => 200, 'otp' => '1234']);
if ($result['success']) {
    // تم الدفع بنجاح
}
```

### 5.3. الاستعلام المباشر عن حالة معاملة

```php
$cash = new CashPay();
$status = $cash->operationStatus('1234567890', 'InitOP');
if ($status['success']) {
    echo "الحالة: " . $status['status']; // success, pending, failed
}
```

### 5.4. تغيير كلمة المرور (ChangePass)

عند استلام رمز خطأ 6022، يجب تغيير كلمة المرور:

```php
$cash = new CashPay();
$result = $cash->changePassword('NewStrongPass123');
if ($result['success']) {
    // تم تغيير كلمة المرور بنجاح، يجب تحديث الإعدادات في لوحة التحكم أيضاً
}
```

### 5.5. استخدام واجهة الاختبار المتكاملة

بعد تسجيل الدخول إلى لوحة التحكم كمسؤول، افتح الرابط:
```
https://yourdomain.com/api/v1/yepayment/cashpay/test-ui
```

الواجهة تتيح:
- اختبار المصادقة (التحقق من صحة الإعدادات).
- إنشاء معاملة (InitPayment) وعرض `TransactionRef`.
- تأكيد الدفع (ConfirmPayment) باستخدام OTP.
- الاستعلام عن الحالة (OperationStatus) باستخدام `RequestID`.
- تغيير كلمة المرور (ChangePass).
- اختبار شامل (إنشاء + تأكيد) في خطوة واحدة.
- اختبار تلقائي بعدد مرات محدد (حتى 5 مرات).
- إحصائيات الاستخدام (عدد الطلبات، نسبة النجاح).
- سجلات محلية للاختبارات.

---

## 6. التعامل مع رموز الأخطاء وأكواد الاستجابة

| كود | المعنى | الإجراء المناسب |
|-----|--------|------------------|
| 1 | نجاح العملية | متابعة التدفق الطبيعي. |
| 35 | بيانات الإدخال غير صالحة | التحقق من الحقول المرسلة. |
| 6022 | كلمة المرور منتهية الصلاحية أو غير صالحة | استدعاء `changePassword()` لتغيير كلمة المرور. |
| 6025 | رصيد العميل غير كافٍ | إبلاغ العميل بشحن الرصيد. |
| 9999 | خطأ داخلي عام | إعادة المحاولة أو الاتصال بالدعم. |

> **ملاحظة:** أي رمز خطأ آخر يتم عرضه كما هو من CashPay عبر `ResultMessage`.

---

## 7. اختبار التكامل (Integration Testing)

تم توفير عدة طبقات للاختبار:

- **بيئة الاختبار:** يمكن استخدام رابط UAT المقدم من CashPay (يُحدد في حقل `cashpay_test_url`).
- **واجهة الاختبار السريع:** من خلال صفحة إعدادات البوابة (الملف الجزئي `_test_info.htm`) يمكن اختبار المصادقة، إنشاء معاملة، تأكيد، استعلام، وتغيير كلمة المرور.
- **واجهة الاختبار المتكاملة:** `/api/v1/yepayment/cashpay/test-ui` توفر جميع الأدوات اللازمة لاختبار البوابة بشكل كامل (يدوي، تلقائي، إحصائيات).
- **نقاط نهاية API مستقلة:** يمكن للمطورين استخدام أدوات مثل Postman أو cURL للاتصال المباشر بالنقاط مثل `/cashpay/test-create-payment`.

**خطوات اختبار سريعة:**
1. تأكد من إعداد Username, Password, Encryption Key, SpId في صفحة إعدادات البوابة.
2. افتح `/api/v1/yepayment/cashpay/test-ui`.
3. انقر على "اختبار المصادقة" للتحقق من صحة البيانات.
4. أدخل رقم طلب، رقم جوال، رمز الدفع، المبلغ، ثم انقر "إنشاء معاملة".
5. سيظهر `TransactionRef` – أدخل OTP الصحيح (يُرسل إلى الجوال) في حقل OTP وانقر "تأكيد الدفع".
6. ستتغير حالة الطلب إلى مدفوع.

---

## 8. ملاحظات للمطورين (Developer Notes)

- **نمط الخطوتين:** `process()` **لا** يُكمل الدفع، بل ينشئ المعاملة فقط. يجب استدعاء `complete()` مع OTP صحيح لتأكيد الدفع وتحديث الطلب إلى `PaidState`.
- **تشفير كلمة المرور:** يتم تشفير كلمة المرور في هيدر كل طلب باستخدام `encryptAes256Cbc()` ومفتاح التشفير المخصص. تأكد من أن المفتاح مطابق لما هو معطى من CashPay.
- **تغيير كلمة المرور:** عند ظهور رمز خطأ `6022`، يجب استدعاء `changePassword()` فوراً (يمكن إظهار رسالة للمسؤول أو توجيهه لتغيير كلمة المرور في لوحة التحكم).
- **التحقق من المعاملة السابقة:** في `process()`، يتم التحقق أولاً من وجود معاملة سابقة مرتبطة بالطلب. إذا كانت حالتها `success` يتم إكمال الطلب فوراً، مما يمنع إنشاء معاملات مكررة.
- **استخدام `HttpHelper`:** جميع طلبات API تستخدم `HttpHelper` الموحد، مما يسهل تتبع الأخطاء وتوسيع البوابة.
- **دعم العملات:** تم تضمين دوال `getSupportedCurrencies()`, `getCurrencyId()`, `isCurrencySupported()` للتحقق من العملات المدعومة وتحويل الرموز إلى الأرقام الصحيحة.
- **دعم `RedirectHelper`:** تم تضمين إمكانية إعادة التوجيه بشكل كامل، مما يجعل البوابة مناسبة لتطبيقات الجوال التي تستخدم Deep Links.
- **قابلية التوسع:** يمكن إضافة دعم Webhook أو استرداد المبالغ مستقبلاً عبر دوال إضافية تستخدم نقاط نهاية CashPay المختلفة.

---

## 9. إصلاحات الأخطاء (Bug Fixes)

لا يوجد – هذا الإصدار مخصص لإضافة ميزة جديدة (CashPay).

---

## 10. فترة التطوير والاختبار

تم تطوير بوابة CashPay خلال الفترة من **15 مايو 2026** وحتى **20 مايو 2026**. تضمنت هذه الفترة:
- كتابة الكود الأساسي لكلاس `CashPay` (يمتد من `PaymentProvider`).
- تنفيذ دوال `process()`, `complete()`, `initPayment()`, `confirmPayment()`, `operationStatus()`, `changePassword()`.
- إضافة التحقق من المعاملة السابقة في `process()` لمنع التكرار.
- تنفيذ آلية تشفير كلمة المرور باستخدام AES‑256‑CBC.
- إنشاء ملفات الواجهة الجزئية (`_info.htm`, `_test_info.htm`) وواجهة الاختبار المتكاملة (`cashpay-ui.htm`).
- كتابة نقاط نهاية API اللازمة للاختبار والمراقبة في `routes.php` (10 نقاط نهاية).
- إضافة مفاتيح الترجمة في ملفات اللغة.
- تسجيل البوابة في `Plugin.php`.
- إجراء اختبارات تكامل شاملة للتحقق من صحة العمل مع بيئة CashPay (محاكاة باستخدام Postman و UI).

---

## 11. روابط ذات صلة

- [توثيق CashPay (دليل المطور)](./docs/CashPay/Docs-CashPay-ar.md)
- [وثيقة SKILL-ar.md لإنشاء بوابات الدفع](./docs/CashPay/SKILL-ar.md)
- [وثيقة Cash-Pay API](./docs/CashPay/external/Cash-Pay%20API%20Doc2.pdf)
- [كلاس CashPay.php](./Nano/Yepayment/PaymentTypes/CashPay.php)
- [ملف routes.php الخاص بـ Nano.Yepayment](./routes.php)
- [قناة الدعم الفني](https://nano2soft.com)

---

**تم إعداد هذا التحديث بواسطة:**  
فريق تطوير نانوسوفت – قسم المدفوعات الإلكترونية  
**المراجع:** Dheia Ali, Nano2Soft
