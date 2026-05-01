## 2026-04-29 – 2026-04-30

### إضافة بوابة دفع JaibPay (محفظة جيب) إلى نظام المدفوعات في نانوسوفت

ضمن الوحدة البرمجية `Nano.Yepayment`

---

### 1. مقدمة

تم تطوير طريقة دفع جديدة باسم **JaibPay** (التي تعمل تحت الاسم التجاري **محفظة جيب**) وإضافتها إلى نظام المدفوعات الموحد في نانوسوفت ضمن الوحدة `Nano.Yepayment`. تأتي هذه الإضافة استجابة للحاجة إلى دعم بوابات الدفع المحلية في اليمن، حيث تُعد **Jaib Pay** إحدى منصات الدفع الإلكتروني التي تتيح قبول المدفوعات عبر أكواد الشراء المولدة من تطبيق جيب.

تستخدم بوابة JaibPay طريقة دفع مباشرة (Direct Payment) من نوع **Execute Buy Online By Code**، حيث يقوم العميل بإدخال كود الشراء الذي أنشأه مسبقاً في تطبيق جيب، ويتم خصم المبلغ فوراً دون حاجة لإعادة توجيه أو تأكيد إضافي (بخلاف YottaPay التي تتطلب OTP). هذه الطريقة مناسبة للخدمات الفورية حيث يتم الدفع بنقرة واحدة.

تم تطوير البوابة وفق أعلى معايير الأمان، مع تخزين التوكنات في Cache، واستخدام `HttpHelper` الموحد لطلبات API، وتوفير واجهات اختبار متكاملة.

---

### 2. المكونات المطورة

#### 2.1. `JaibPay` – كلاس بوابة الدفع الرئيسي

- **المسار:** `Nano\Yepayment\PaymentTypes\JaibPay`
- **الوراثة:** يمتد من `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **الوظيفة:** مسؤول عن المصادقة مع Jaib API، تنفيذ الدفع المباشر، الاستعلام عن حالة المعاملة، وإدارة التوكنات.

**الدوال الرئيسية:**

| الدالة | الوصف |
|--------|-------|
| `process(PaymentResult $result)` | تنفيذ الدفع المباشر: الحصول على توكن، إرسال طلب `ExeBuy`، حفظ `requestID` و `referenceID`، وتحديث حالة الطلب إلى مدفوع. |
| `complete(PaymentResult $result)` | غير مستخدم في هذا النوع (دفع فوري)، لكنه موجود للتوافق مع الواجهة. |
| `getAuthToken(): ?array` | طلب توكن OAuth 2.0 من Jaib عبر `POST /api/v1/TokenAuth/LogAPI`، وتخزين `accessToken` و `pinApi` في Cache. |
| `executeBuy($accessToken, $pinApi, $requestID): array` | إرسال طلب `POST /api/v1/BuyOnline/ExeBuy` مع `code` (كود الشراء)، `mobile`، `amount`، `currencyCode`. |
| `checkTransactionStatus($requestID): array` | الاستعلام عن حالة المعاملة عبر `POST /api/v1/BuyOnline/CheckProgress`. |
| `getApiUrl($type): string` | بناء روابط API (login, buy, refund, check). |
| `parseResponse($response): array` | تحويل استجابة Guzzle إلى مصفوفة PHP. |
| `settings(): array` | تعريف حقول الإعدادات في لوحة التحكم (URL، اسم المستخدم، كلمة المرور، agentCode، العملة الافتراضية). |
| `encryptedSettings(): array` | تحديد الحقول التي يتم تخزينها بشكل مشفر (`jaibpay_password`). |

#### 2.2. ملفات جزئية (Partials) وإعدادات

| الملف | المسار | الوصف |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/jaibpay/_info.htm` | عرض إرشادات الإعداد ومعلومات البوابة في لوحة التحكم. |
| `_test_info.htm` | `paymenttypes/jaibpay/_test_info.htm` | أدوات اختبار سريع داخل صفحة الإعدادات (زر إنشاء دفعة، استعلام، اختبار شامل، مسح السجلات). |
| `jaibpay-ui.htm` | `views/jaibpay-ui.htm` | واجهة ويب تفاعلية متكاملة لاختبار جميع وظائف JaibPay (يدوي، تلقائي، إحصائيات، سجلات). |

#### 2.3. نقاط نهاية API (`routes.php`)

تم إضافة مجموعة كاملة من المسارات ضمن مجموعة `yepayment`:

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/jaibpay/test-auth` | POST | اختبار المصادقة (التحقق من صحة username/password/agentCode). |
| `/jaibpay/test-create-payment` | POST | إنشاء دفعة جديدة (تنفيذ الدفع المباشر). |
| `/jaibpay/test-check-status` | GET | الاستعلام عن حالة معاملة باستخدام `request_id`. |
| `/jaibpay/test-full-payment` | POST | اختبار شامل (إنشاء دفعة + استعلام). |
| `/jaibpay/stats` | GET | إحصائيات استخدام البوابة (عدد الطلبات، نسبة النجاح، آخر السجلات). |
| `/jaibpay/test-ui` | GET | واجهة الاختبار المتكاملة (HTML). |

جميع المسارات محمية بـ `BackendAuth::getUser()` لضمان عدم وصول غير المصرح لهم.

#### 2.4. `Plugin.php` – تسجيل مزود الدفع

تمت إضافة السطر التالي في دالة `registerPaymentProviders()` ضمن قسم `allow_yemen_payment`:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\JaibPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\JaibPay();
}
```

#### 2.5. ملفات اللغة (Translation)

تم إضافة مفاتيح الترجمة في `lang/ar/lang.php` و `lang/en/lang.php` لكل من الإعدادات ورسائل العموم والأخطاء.

---

### 3. دورة عمل الدفع (Payment Workflow)

يعتمد JaibPay على تدفق **Direct (فوري)** بدون إعادة توجيه أو تأكيد إضافي:

1. **المصادقة (Authentication)**  
   يتم استدعاء `getAuthToken()`:
   - التحقق من وجود `accessToken` و `pinApi` في Cache.
   - إذا لم يكن موجوداً، يتم إرسال طلب `POST /api/v1/TokenAuth/LogAPI` مع `userName` و `password` و `agentCode`.
   - يتم تخزين `accessToken` و `pinApi` في Cache لمدة 86000 ثانية (أقل بقليل من صلاحية التوكن الفعلية 86400).

2. **إنشاء دفعة (Execute Buy)**  
   عند استدعاء `process()`:
   - التحقق من صحة المدخلات (`purchase_code`, `mobile`, `amount`, `currency`).
   - توليد `requestID` فريد (UUID v4).
   - إرسال طلب `POST /api/v1/BuyOnline/ExeBuy` مع:
     ```json
     {
       "pinApi": "tIcI4zm",
       "mobile": "774760761",
       "requestID": "550e8400-e29b-41d4-a716-446655440000",
       "code": "3719",
       "amount": 5000,
       "currencyCode": "YER",
       "notes": "دفع الطلب #200"
     }
     ```
   - استلام `referenceID` و `msg` ("تمت العملية بنجاح") من الاستجابة.
   - حفظ `requestID` في `order.payment_first_trans_id` و `referenceID` في `order.payment_trans_id`.
   - تحديث حالة الطلب إلى `PaidState` عبر `$result->success()`.

3. **الاستعلام عن الحالة (Check Progress)**  
   يمكن استدعاء `checkTransactionStatus($requestID)` في أي وقت:
   - إرسال طلب `POST /api/v1/BuyOnline/CheckProgress` مع `pinApi` و `requestID`.
   - استلام `referenceID` إذا كانت المعاملة موجودة.

4. **استرجاع الأموال (Refund)** – لم يتم تنفيذه في هذا الإصدار، لكنه مدعوم في API عبر `/RefoundBuy` ويمكن إضافته مستقبلاً.

---

### 4. الإعدادات والتهيئة (Configuration)

لتفعيل JaibPay، يجب إضافة الإعدادات التالية في واجهة إعدادات بوابة الدفع في نظام نانوسوفت (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| الإعداد | المفتاح | الوصف |
|---------|---------|-------|
| رابط API الأساسي | `jaibpay_url` | عنوان Jaib API (مثال: `https://www.api2.e-jaib.com:5088`) |
| رابط API للاختبار | `jaibpay_test_url` | (اختياري) رابط بيئة الاختبار |
| اسم المستخدم | `jaibpay_username` | اسم مستخدم التاجر |
| كلمة المرور | `jaibpay_password` | كلمة مرور التاجر (تخزن مشفرة) |
| كود الوكيل (agentCode) | `jaibpay_agentcode` | القيمة الافتراضية `10004` |
| العملة الافتراضية | `jaibpay_default_currency` | `YER` / `USD` / `SAR` (افتراضي `YER`) |

**ملاحظات**:
- حقل `jaibpay_password` يتم تخزينه بشكل مشفر تلقائياً عبر `encryptedSettings()`.
- يمكن أيضاً تعيين هذه القيم عبر متغيرات البيئة في `.env` إذا تم دعمها.

---

### 5. الأمثلة التوضيحية (Usage Examples)

#### 5.1. إنشاء دفعة لطلب موجود (ضمن تطبيق نانوسوفت)

```php
use Nano\Yepayment\PaymentTypes\JaibPay;
use Nano\Orders\Models\Order;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$provider = new JaibPay($order, [
    'purchase_code' => '3719',
    'mobile'        => '774760761',
    'amount'        => 5000,
    'currency'      => 'YER',
    'notes'         => 'دفع الطلب #200',
]);

$result = new PaymentResult($provider, $order);
$processed = $provider->process($result);

if ($processed->successful) {
    // الدفع تم بنجاح
    $requestID = $order->payment_first_trans_id;
    $referenceID = $order->payment_trans_id;
    return redirect()->route('order.success', $order->id);
} else {
    return back()->withError($processed->message);
}
```

#### 5.2. الاستعلام عن حالة معاملة

```php
$jaib = new JaibPay();
$status = $jaib->checkTransactionStatus('550e8400-e29b-41d4-a716-446655440000');
if ($status['success']) {
    echo "Reference ID: " . $status['reference_id'];
}
```

#### 5.3. اختبار الدفع عبر واجهة API (للمطورين)

```bash
# اختبار المصادقة
curl -X POST "https://yourdomain.com/api/v1/yepayment/jaibpay/test-auth" \
  -H "Authorization: Bearer <admin_token>"

# إنشاء دفعة جديدة
curl -X POST "https://yourdomain.com/api/v1/yepayment/jaibpay/test-create-payment" \
  -H "Authorization: Bearer <admin_token>" \
  -H "Content-Type: application/json" \
  -d '{"order_id":200,"purchase_code":"3719","mobile":"774760761","amount":5000,"currency":"YER"}'

# الاستعلام عن الحالة
curl -X GET "https://yourdomain.com/api/v1/yepayment/jaibpay/test-check-status?request_id=550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer <admin_token>"
```

#### 5.4. استخدام واجهة الاختبار المتكاملة

بعد تسجيل الدخول إلى لوحة التحكم كمسؤول، افتح الرابط:
```
https://yourdomain.com/api/v1/yepayment/jaibpay/test-ui
```

تحتوي الواجهة على:
- تبويب "يدوي": إنشاء دفعة، استعلام، اختبار شامل.
- تبويب "تلقائي": اختبار تكراري (حتى 10 مرات) مع عرض نسبة النجاح.
- تبويب "إحصائيات": عرض إجمالي الطلبات، طلبات JaibPay، المدفوعات الناجحة، نسبة النجاح، آخر السجلات.
- تبويب "سجلات": تخزين محلي لنتائج الاختبارات مع إمكانية التصدير والمسح.

---

### 6. التعامل مع عمليات إعادة التوجيه (Deeplinks & Callbacks)

بما أن JaibPay من نوع **Direct** (دفع فوري بدون إعادة توجيه)، فإن `success_url` و `cancel_url` غير مستخدمين في الدفع الأساسي. ومع ذلك، تم توفير مسار `success` و `cancel` في الكلاس (يمكن إضافتهما مستقبلاً إذا تطلب API ذلك)، لكنهما غير مستخدمين حالياً.

في نقاط نهاية الاختبار، يتم استخدام `RedirectHelper` بشكل محدود فقط لإعادة التوجيه إلى صفحة الاختبار نفسها. للتكامل مع تطبيقات الجوال، يمكن للتاجر توفير `callback_success_url` في `order.other_data` (عبر `RedirectHelper::getCallbackSuccessUrlByOrder()`).

---

### 7. القيمة المضافة

- **للمطورين:** نموذج إضافي لبوابة دفع من نوع **Direct** (بدون إعادة توجيه ولا OTP)، مما يثري مكتبة الأمثلة في `SKILL.md`. استخدام `Cache` لتخزين التوكنات، و`HttpHelper` الموحد، ونقاط نهاية اختبار كاملة.
- **للتجار:** دعم بوابة دفع محلية في اليمن (Jaib Pay) تعتمد على أكواد الشراء، مما يسمح للعملاء بالدفع بسرعة دون الحاجة لإدخال بيانات بطاقة ائتمان أو انتظار رسالة OTP (الدفع فوري).
- **للمستخدمين النهائيين:** تجربة دفع سلسة وسريعة باستخدام كود الشراء من تطبيق جيب، مع دعم العملات (YER, USD, SAR).

---

### 8. اختبار التكامل (Integration Testing)

يوفر النظام نقاط نهاية مخصصة لاختبار التكامل:

- **بيئة الاختبار:** يمكن استخدام نفس رابط الإنتاج (`https://www.api2.e-jaib.com:5088`) مع بيانات دخول تجريبية من Jaib (غير متاحة للعموم، يجب التنسيق مع المشغل). إذا وفرت Jaib بيئة اختبار (UAT)، يمكن إدخال رابطها في إعداد `jaibpay_test_url`.
- **أكواد شراء تجريبية:** وفقاً لوثائق Jaib، الكود `3719` ورقم الجوال `774760761` يُستخدمان للاختبار.
- **واجهة الاختبار المتكاملة:** `/api/v1/yepayment/jaibpay/test-ui` توفر جميع الأدوات اللازمة لاختبار البوابة.

**خطوات اختبار سريعة**:
1. تأكد من صحة بيانات الدخول (username/password/agentCode) في إعدادات البوابة.
2. افتح واجهة الاختبار: `/api/v1/yepayment/jaibpay/test-ui`.
3. انقر على "اختبار المصادقة" للتأكد من صلاحية البيانات.
4. أدخل كود شراء تجريبي (`3719`) ورقم جوال (`774760761`) والمبلغ المطلوب (`5000`).
5. انقر على "تنفيذ الدفع" – سيتم إنشاء دفعة جديدة.
6. انسخ `Request ID` واستخدمه في حقل الاستعلام للتحقق من الحالة.
7. يمكنك أيضاً استخدام "اختبار شامل" لتنفيذ الخطوتين معاً.

---

### 9. التوافق (Compatibility)

- **NanoSoft App:** v2.x
- **PHP:** 8.0 / 8.1 / 8.2
- **إضافات نانوسوفت:**
  - `Nano.MicroCart` (>=2.0)
  - `Nano.Helpers` (>=1.2)
  - `Nano.Orders` (>=1.5)
- **قواعد البيانات:** يدعم MySQL، PostgreSQL، SQLite (من خلال Eloquent)

---

### 10. ملاحظات للمطورين (Developer Notes)

- **لا تعتمد JaibPay على إعادة التوجيه**: لذلك تم إهمال `success_url` و `cancel_url` في الدفع الأساسي. يمكن إضافتهما لاحقاً إذا دعت الحاجة.
- **تخزين التوكنات في Cache**: يتم تخزين `accessToken` و `pinApi` لمدة 86000 ثانية (أقل من صلاحية التوكن الفعلية) لتجنب طلبات المصادقة المتكررة.
- **استخدام `HttpHelper`**: جميع طلبات API تستخدم `HttpHelper::sendJson`، مما يسهل تتبع الأخطاء وتوسيع البوابة.
- **توليد `requestID`**: يتم استخدام `Str::uuid()` لضمان تفرد المعرف لكل معاملة.
- **الاختبار الآلي**: يمكن استخدام نقطة النهاية `/jaibpay/test-full-payment` مع خيار `iterations` في واجهة الاختبار لاختبار استقرارية البوابة تحت ضغط.
- **توسيع الدوال**: يمكن إضافة دالة `refund()` مستقبلاً باستخدام مسار `/RefoundBuy` الموجود بالفعل في `getApiUrl()`.

---

### 11. إصلاحات الأخطاء (Bug Fixes)

لا يوجد – هذا الإصدار مخصص لإضافة ميزة جديدة فقط.

---

### 12. روابط ذات صلة

- [JaibPay API Documentation ](./Docs-JaibPay-ar.md)
- [وثيقة API الخاصة بـ Jaib Pay – تسجيل الدخول (Login.pdf)](./Login.pdf)
- [وثيقة API الخاصة بـ Jaib Pay – تنفيذ واستعلام الدفع (Jaib Wallet Pay API.pdf)](./Jaib%20Wallet%20Pay%20API.pdf)
- [مجموعة Postman لاختبار API Jaib Pay](./Jaib%20Pay%20API.postman_collection.json)
- [دليل تطوير بوابات الدفع – نانوسوفت](https://docs.nano2soft.com/payment-gateways)
- [قناة الدعم الفني](https://nano2soft.com)

---

**تم إعداد هذا التحديث بواسطة:**  
فريق تطوير نانوسوفت – قسم المدفوعات الإلكترونية  
**المراجع:** Dheia Ali, Nano2Soft

---