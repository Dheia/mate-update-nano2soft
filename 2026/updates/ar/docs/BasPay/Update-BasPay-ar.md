## 2026-05-06 – 2026-05-18

### إضافة وتحديث بوابة دفع BasPay (منصة بس) في نظام المدفوعات بنانوسوفت

ضمن الوحدة البرمجية `Nano.Yepayment`

---

### 1. مقدمة

تم تطوير طريقة دفع جديدة باسم **BasPay** وإضافتها إلى نظام المدفوعات الموحد في نانوسوفت ضمن الوحدة `Nano.Yepayment`. تأتي هذه الإضافة استجابة للحاجة إلى دعم بوابات الدفع المحلية في اليمن، حيث تُعد **منصة بس (BAS)** إحدى منصات الدفع الإلكتروني التي تتيح قبول المدفوعات عبر واجهة API مباشرة مع توقيع مشفر AES-256-CBC.

بعد الفحص الدقيق لتدفق الدفع في منصة بس، **تم تحديث البوابة** لتعمل وفق نمط **الدفع بخطوتين (Two‑Step Payment)**، وهو النمط الذي يتوافق مع طبيعة عمل المنصة حيث:
1. **الخطوة الأولى (process):** يتم إنشاء معاملة والحصول على `trxToken`، دون خصم المبلغ بعد.
2. **الخطوة الثانية (complete):** بعد أن يكمل العميل الدفع عبر تطبيق بس، يتم التحقق من حالة المعاملة وتحديث الطلب إلى "مدفوع".

هذا التحديث جعل البوابة متوافقة تماماً مع `OrderManager` و `Checkout2` و `PaymentResult`، ويمنع تغيير حالة الطلب إلى مدفوع قبل الأوان.

تم تطوير البوابة وفق أعلى معايير الأمان والجودة، مع تخزين التوكنات في Cache، واستخدام `HttpHelper` الموحد لجميع طلبات API، وتوفير آلية توليد توقيع مطابقة تماماً لمتطلبات منصة بس (SHA256 + AES-256-CBC)، بالإضافة إلى واجهات اختبار متكاملة ونقاط نهاية API مخصصة للمطورين والمسؤولين. كما تم دعم `RedirectHelper` لضمان التوافق مع تطبيقات الجوال التي تحتاج إلى روابط عودة بعد الدفع.

---

### 2. المكونات المطورة

#### 2.1. `BasPay` – كلاس بوابة الدفع الرئيسي

- **المسار:** `Nano\Yepayment\PaymentTypes\BasPay`
- **الوراثة:** يمتد من `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **الوظيفة:** مسؤول عن المصادقة مع منصة بس عبر OAuth2، تنفيذ الخطوة الأولى (إنشاء معاملة)، والخطوة الثانية (التحقق من الحالة وتأكيد الدفع)، وتوليد التوقيع المشفر.

**الدوال الرئيسية:**

| الدالة | الوصف |
|--------|-------|
| `process(PaymentResult $result)` | **الخطوة الأولى:** إنشاء معاملة عبر `initiateTransaction`، استلام `trxToken`، وحفظه في الطلب دون تغيير حالة الدفع. تُعيد نجاحاً مع رسالة تأكيد. |
| `complete(PaymentResult $result)` | **الخطوة الثانية:** التحقق من حالة المعاملة عبر `checkTransactionStatusByToken`، وفي حال كانت `SUCCESS` يتم استدعاء `$result->success()` لتحديث الطلب إلى `PaidState`. |
| `checkAndCompletePay(array $options): array` | دالة `static` عامة تُستخدم في مسار `baspay/success` لربط العودة من التطبيق بعملية التأكيد. |
| `getAuthToken(): ?string` | طلب توكن OAuth 2.0 من BAS عبر `POST /api/v1/auth/token` (grant_type: client_credentials)، وتخزينه في Cache لمدة 3500 ثانية. |
| `initiateTransaction($token): array` | إرسال طلب `POST /api/v1/merchant/sdk-payment/initiate-transaction` مع جسم الطلب والتوقيع. |
| `checkTransactionStatusByToken($token, $trxToken): array` | الاستعلام عن حالة المعاملة عبر `POST /api/v1/merchant/sdk-payment/get-transaction-status` (تُستخدم داخلياً). |
| `checkTransactionStatus($trxToken): array` | دالة عامة للاستعلام عن حالة معاملة (للاستخدام الخارجي). |
| `generateSignature($paramsString): string` | توليد توقيع BAS: salt عشوائي (4 أحرف)، SHA256، ثم تشفير AES-256-CBC باستخدام Merchant Key و IV. |
| `encryptAes256Cbc($input, $key, $iv): string` | دالة التشفير AES-256-CBC باستخدام OpenSSL. |
| `parseResponse($response): array` | تحويل استجابة HTTP إلى مصفوفة PHP. |
| `getCommonHeaders(): array` | إرجاع الهيدرات المطلوبة لكل طلب (x-client-id, x-app-id, x-sdk-version, ...). |
| `settings(): array` | تعريف حقول الإعدادات في لوحة التحكم (URL, Client ID, Client Secret, App ID, Merchant Key, IV, العملة الافتراضية). |
| `encryptedSettings(): array` | تحديد الحقول التي تخزن بشكل مشفر (`baspay_client_secret`, `baspay_merchant_key`). |

#### 2.2. ملفات جزئية (Partials) وإعدادات

| الملف | المسار | الوصف |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/baspay/_info.htm` | عرض إرشادات الإعداد ومعلومات البوابة في لوحة التحكم. |
| `_test_info.htm` | `paymenttypes/baspay/_test_info.htm` | أدوات اختبار سريع داخل صفحة الإعدادات (أزرار المصادقة، إنشاء دفعة، التحقق من الحالة). |
| `baspay-ui.htm` | `views/baspay-ui.htm` | واجهة ويب تفاعلية متكاملة لاختبار جميع وظائف BasPay (يدوي، تلقائي، إحصائيات، سجلات). |

#### 2.3. نقاط نهاية API (`routes.php`)

تم إضافة مجموعة كاملة من المسارات ضمن مجموعة `yepayment` لدعم الاختبار والمراقبة:

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/baspay/test-auth` | POST | اختبار المصادقة والحصول على Bearer Token. |
| `/baspay/test-create-payment` | POST | إنشاء معاملة جديدة (الخطوة الأولى). |
| `/baspay/test-check-status` | POST | التحقق من حالة معاملة باستخدام `trx_token`. |
| `/baspay/test-full-payment` | POST | اختبار شامل (إنشاء معاملة + التحقق من الحالة). |
| `/baspay/stats` | GET | إحصائيات استخدام البوابة (عدد الطلبات، نسبة النجاح). |
| `/baspay/test-ui` | GET | واجهة الاختبار المتكاملة (HTML). |
| `/baspay/success` | GET | نقطة نهاية لتأكيد الدفع بعد عودة المستخدم من تطبيق بس. |
| `/baspay/cancel` | GET | نقطة نهاية اختيارية لإعادة التوجيه عند الإلغاء. |

جميع المسارات (باستثناء success/cancel) محمية بـ `BackendAuth::getUser()` لضمان وصول المسؤولين فقط.

#### 2.4. `Plugin.php` – تسجيل مزود الدفع

تمت إضافة السطر التالي في دالة `registerPaymentProviders()` ضمن قسم `allow_yemen_payment`:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\BasPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\BasPay();
}
```

#### 2.5. ملفات اللغة (Translation)

تم إضافة مفاتيح الترجمة في `lang/ar/lang.php` و `lang/en/lang.php` لكل من الإعدادات ورسائل العموم والأخطاء (مثل `payment_success`, `auth_failed`, `order_already_paid`).

---

### 3. دورة عمل الدفع (Payment Workflow) - نمط الخطوتين

تعمل BasPay وفق تدفق **Two‑Step (خطوتين)**:

1. **المصادقة (Authentication)**  
   يتم استدعاء `getAuthToken()`:
   - التحقق من وجود `access_token` في Cache.
   - إذا لم يكن موجوداً، يتم إرسال طلب `POST /api/v1/auth/token` مع `client_id`، `client_secret`، و `grant_type=client_credentials`.
   - يتم تخزين التوكن في Cache لمدة 3500 ثانية.

2. **الخطوة الأولى – إنشاء المعاملة (`process`)**  
   - التحقق من أن الطلب غير مدفوع مسبقاً.
   - بناء جسم الطلب (`amount`، `currency`، `orderId`، `appId`، `requestTimestamp`).
   - توليد التوقيع عبر `generateSignature()`.
   - إرسال طلب `POST /api/v1/merchant/sdk-payment/initiate-transaction`.
   - استلام `trxToken` وتخزينه في `order.payment_first_trans_id` و `order.other_data['baspay']`.
   - **لا يتم استدعاء `$result->success()`**، ولا يتغير الطلب إلى `PaidState`. بدلاً من ذلك، تُرجع البوابة `successful = true` مع رسالة `confirmation_required`.

3. **الخطوة الثانية – تأكيد الدفع (`complete`)**  
   - تُستدعى من مسار `baspay/success` بعد عودة المستخدم من تطبيق بس.
   - تستخرج `trxToken` من الطلب، وتتواصل مع BAS عبر `POST /api/v1/merchant/sdk-payment/get-transaction-status`.
   - إذا كانت `trxStatus` تساوي `SUCCESS` أو `COMPLETED`:
     - يتم استدعاء `$result->success()` التي **تغير حالة الطلب إلى `PaidState`** وتُشغل الأحداث (مثل `nano.orders.paymentProcessed`).
   - إذا كانت `PENDING`: يتم استدعاء `$result->pending()` وتبقى حالة الطلب معلقة.

4. **روابط العودة الاختيارية**  
   عند تمرير `callback_success_url` أو `callback_error_url`، يتم تخزينها في `other_data`. بعد التأكيد، يمكن للمطور استخدام `RedirectHelper` لإعادة توجيه المستخدم بذكاء (Deep Link أو ويب).

---

### 4. الإعدادات والتهيئة (Configuration)

لتفعيل BasPay، يجب إدخال الإعدادات التالية في واجهة إعدادات بوابة الدفع في نظام نانوسوفت (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| الإعداد | المفتاح | الوصف | القيمة الافتراضية |
|---------|---------|-------|-------------------|
| رابط API الأساسي | `baspay_url` | عنوان API منصة بس | `https://api.basgate.com` |
| Client ID | `baspay_client_id` | معرف العميل من منصة بس | - |
| Client Secret | `baspay_client_secret` | المفتاح السري (مخزن مشفر) | - |
| App ID | `baspay_app_id` | معرف التطبيق | - |
| Merchant Key | `baspay_merchant_key` | مفتاح التاجر للتوقيع (مخزن مشفر) | - |
| IV | `baspay_iv` | قيمة المتجه الأولي (16 بايت) | `@@@@&&&&####$$$$` |
| العملة الافتراضية | `baspay_default_currency` | العملة المستخدمة إذا لم يحدد الطلب عملة | `YER` |

**ملاحظات أمنية:**
- `baspay_client_secret` و `baspay_merchant_key` يتم تخزينهما مشفرين عبر `encryptedSettings()`.
- لا يتم تسجيل هذه القيم الحساسة في السجلات.

---

### 5. الأمثلة التوضيحية (Usage Examples)

#### 5.1. بدء الدفع – الخطوة الأولى (إنشاء المعاملة)

```php
use Nano\Yepayment\PaymentTypes\BasPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$bas = new BasPay($order, [
    'callback_success_url' => 'myapp://pay/success',   // اختياري
]);
$result = new PaymentResult($bas, $order);
$processResult = $bas->process($result);

if ($processResult->successful) {
    // تم إنشاء المعاملة وحفظ trxToken، لكن الدفع لم يكتمل بعد
    $trxToken = $order->payment_first_trans_id;
    // يجب توجيه المستخدم لتطبيق بس لإكمال الدفع
}
```

#### 5.2. تأكيد الدفع – الخطوة الثانية (التحقق من الحالة)

بعد أن يكمل العميل الدفع في تطبيق بس، يمكن تأكيد العملية كالتالي:

```php
$result = \Nano\Yepayment\PaymentTypes\BasPay::checkAndCompletePay(['order_id' => 200]);
if ($result['success']) {
    // تم تأكيد الدفع وتحديث الطلب إلى PaidState
}
```

#### 5.3. الاستعلام المباشر عن حالة معاملة

```php
$bas = new BasPay();
$status = $bas->checkTransactionStatus('bas_trx_a1b2c3d4...');
if ($status['success']) {
    // $status['data'] يحتوي على تفاصيل المعاملة
}
```

#### 5.4. استخدام واجهة الاختبار المتكاملة

بعد تسجيل الدخول إلى لوحة التحكم كمسؤول، افتح الرابط:
```
https://yourdomain.com/api/v1/yepayment/baspay/test-ui
```

الواجهة تتيح:
- إنشاء معاملة جديدة (Initiate).
- التحقق من الحالة.
- اختبار تلقائي بعدد مرات محدد (حتى 10).
- عرض إحصائيات الاستخدام (عدد الطلبات، نسبة النجاح).
- سجلات محلية للاختبارات.

---

### 6. التعامل مع عمليات إعادة التوجيه (Deeplinks & Callbacks)

تم دمج دعم **اختياري** لروابط إعادة التوجيه باستخدام `RedirectHelper`، لضمان التوافق مع تطبيقات الجوال التي تحتاج إلى فتح شاشة معينة بعد الدفع.

- عند تمرير `callback_success_url` أو `callback_error_url` في بيانات الدفع، يتم تخزينهما في `order.other_data['baspay']`.
- بعد تأكيد الدفع عبر `complete()`، يمكن للمطور توجيه المستخدم إلى `/baspay/success` (مع `order_id`) لتقوم البوابة باستخراج رابط العودة، وتنفيذ إعادة التوجيه الذكي (deeplink أو ويب) مع تمرير بيانات الحالة.
- نفس الآلية تنطبق على `/baspay/cancel`.

---

### 7. القيمة المضافة

- **للمطورين:** نموذج شامل لبوابة دفع من نوع **Two-Step** يستخدم OAuth2 مع توقيع مشفر AES-256-CBC، مما يوضح كيفية بناء بوابات تتطلب تأكيداً خارجياً دون SDK. كما يُظهر التكامل الكامل مع `OrderManager` و `PaymentResult` و `RedirectHelper`.
- **للتجار:** دعم منصة بس المحلية في اليمن، مما يفتح المجال لقبول المدفوعات الإلكترونية بسهولة عبر حساب التاجر في بس.
- **للمستخدمين النهائيين:** تجربة دفع موثوقة تتم على خطوتين: تأكيد الطلب في المتجر، ثم إكمال الدفع في تطبيق بس.

---

### 8. اختبار التكامل (Integration Testing)

تم توفير عدة طبقات للاختبار:

- **بيئة الاختبار:** يمكن استخدام نفس رابط الإنتاج (`https://api.basgate.com`) مع بيانات اعتماد تجريبية من منصة بس. إذا توفرت بيئة اختبار مستقبلية، يمكن إدخال رابطها في حقل `baspay_url`.
- **واجهة الاختبار السريع:** من خلال صفحة إعدادات البوابة (الملف الجزئي `_test_info.htm`) يمكن اختبار المصادقة، إنشاء معاملة، والتحقق من الحالة مباشرة.
- **واجهة الاختبار المتكاملة:** `/api/v1/yepayment/baspay/test-ui` توفر جميع الأدوات اللازمة لاختبار البوابة بشكل كامل (بما في ذلك محاكاة الخطوتين)، مع إمكانية الاختبار التكراري (Automated Testing).
- **نقاط نهاية API مستقلة:** يمكن للمطورين استخدام أدوات مثل Postman أو cURL للاتصال المباشر بالنقاط مثل `/baspay/test-full-payment`.

**خطوات اختبار سريعة:**
1. تأكد من إعداد Client ID، Client Secret، App ID، Merchant Key، IV في صفحة إعدادات البوابة.
2. افتح `/api/v1/yepayment/baspay/test-ui`.
3. انقر على "اختبار المصادقة" للتحقق من صحة البيانات.
4. أدخل رقم طلب ومبلغ، ثم انقر "إنشاء معاملة جديدة".
5. ستظهر `trxToken` – يمكنك استخدامها في "التحقق من الحالة" لمحاكاة الخطوة الثانية.

---

### 9. ملاحظات للمطورين (Developer Notes)

- **نمط الخطوتين:** `process()` **لا** يُكمل الدفع، بل ينشئ المعاملة فقط. يجب استدعاء `complete()` أو `checkAndCompletePay` لتأكيد الدفع وتحديث الطلب إلى `PaidState`.
- **التخزين المؤقت للتوكن:** يتم تخزين Bearer Token في Cache لمدة 3500 ثانية (أقل من صلاحية التوكن الفعلية المقدّرة بـ 3600 ثانية).
- **التوقيع المشفر:** تم تنفيذ آلية التوقيع يدوياً باستخدام OpenSSL دون الاعتماد على حزمة خارجية، وذلك لضمان التوافق مع البيئة الخاصة بنانوسوفت ومنع التعارضات.
- **استخدام `HttpHelper`:** جميع طلبات API (JSON، Form) تستخدم `HttpHelper` الموحد، مما يسهل تتبع الأخطاء وتوسيع البوابة.
- **دعم `RedirectHelper`:** تم تضمين إمكانية إعادة التوجيه بشكل كامل، مما يجعل البوابة مناسبة لتطبيقات الجوال التي تستخدم Deep Links.
- **قابلية التوسع:** يمكن إضافة دعم Webhook أو استرداد المبالغ مستقبلاً عبر دوال إضافية تستخدم نقاط نهاية BAS المختلفة.

---

### 10. إصلاحات الأخطاء (Bug Fixes)

لا يوجد – هذا الإصدار مخصص لإضافة ميزة جديدة وتحديثها فقط (BasPay).

---

### 11. فترة التطوير والاختبار

تم تطوير بوابة BasPay في البداية كدفع مباشر، ثم **تم تحديثها** إلى نمط الخطوتين خلال الفترة من **6 مايو 2026** وحتى **18 مايو 2026**. تضمنت هذه الفترة كتابة الكود الأساسي للكلاس BasPay وتطبيق آلية التوقيع المشفر AES-256-CBC، وإنشاء ملفات الواجهة الجزئية (_info.htm, _test_info.htm) وواجهة الاختبار المتكاملة (baspay-ui.htm)، بالإضافة إلى كتابة نقاط نهاية API اللازمة للاختبار والمراقبة، وتعديل `process()` و`complete()` لتتوافق مع تدفق الخطوتين. تم أيضاً إجراء اختبارات تكامل شاملة للتأكد من صحة عمل البوابة مع بيئة منصة بس، وتم التحقق من توافق التوقيع مع الردود المستلمة.

---

### 12. روابط ذات صلة

- [توثيق BasPay (دليل المطور)](./docs/BasPay/Docs-BasPay-ar.md)
- [وثيقة SKILL-ar.md لإنشاء بوابات الدفع](./docs/BasPay/SKILL-ar.md)
- [دليل منصة بس API](https://basgate.apidog.io)
- [دليل منصة بس API](./docs/BasPay/external/BAS/README-ar.md)
- [مستودع وثائق BasGate على GitHub](https://github.com/basgate/basgate.github.io)
- [حزمة Laravel Payment SDK على GitHub](https://github.com/basgate/laravel-payment-sdk)
- [مستودع BasPaymentFlutter على GitHub](https://github.com/BasPlatform/BasPaymentFlutter.git)
- [حزمة bas_php_sdk على GitHub](https://github.com/basgate/bas_php_sdk)
- [حزمة bas-laravel-sdk على GitHub](https://github.com/basgate/bas-laravel-sdk)
- [قناة الدعم الفني](https://nano2soft.com)

---

**تم إعداد هذا التحديث بواسطة:**  
فريق تطوير نانوسوفت – قسم المدفوعات الإلكترونية  
**المراجع:** Dheia Ali, Nano2Soft