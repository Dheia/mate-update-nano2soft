## 2026-4-9 – 2026-4-25

### إضافة بوابة دفع YottaPay (Sabacash) إلى نظام المدفوعات في نانوسوفت

ضمن الوحدة البرمجية `Nano.Yepayment`  

---

### 1. مقدمة

تم تطوير طريقة دفع جديدة باسم **YottaPay** (التي تعمل تحت الاسم التجاري **Sabacash**) وإضافتها إلى نظام المدفوعات الموحد في نانوسوفت ضمن الوحدة `Nano.Yepayment`. تأتي هذه الإضافة استجابة للحاجة إلى دعم بوابات الدفع المحلية في اليمن، حيث تُعد **YottaGate** إحدى منصات الدفع الإلكتروني الرائدة التي تتيح قبول المدفوعات عبر التحويل المباشر من أرصدة العملاء باستخدام رمز OTP.

تهدف هذه الإضافة إلى:
- تمكين تجار نانوسوفت من قبول المدفوعات عبر نظام YottaGate (Sabacash) بخطوتين آمنتين (إنشاء معاملة ثم تأكيدها برمز OTP).
- توفير تجربة دفع سلسة دون الحاجة إلى إعادة توجيه المستخدم إلى صفحات خارجية (دفع داخل التطبيق).
- دمج كامل مع نظام `Nano.Yepayment` الحالي، بما في ذلك إدارة المعاملات والتحقق من الحالة وعمليات الاسترجاع.
- دعم بيئتي الاختبار (UAT) والإنتاج (Live) عبر بيانات دخول منفصلة.

---

### 2. المكونات المطورة

لتنفيذ هذا التكامل، تم إنشاء وتعديل الكلاسات التالية داخل الوحدة `Nano.Yepayment`:

#### 2.1 `YottaPay` – كلاس بوابة الدفع الرئيسي
- **المسار**: `Nano\Yepayment\PaymentTypes\YottaPay`
- **الوراثة**: يمتد من `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **الوظيفة**: مسؤول عن إنشاء معاملات الدفع، تأكيدها باستخدام OTP، الاستعلام عن حالة المعاملة، وتغيير كلمة المرور.

**الدوال الرئيسية**:

| الدالة | الوصف |
|--------|-------|
| `process(PaymentResult $result)` | إنشاء معاملة دفع جديدة عبر API YottaGate (طلب `POST /onLinePayment`). تخزين `adjustment.id` في `order.payment_first_trans_id` وإرجاع رسالة تطلب إدخال OTP. |
| `complete(PaymentResult $result)` | تأكيد الدفع باستخدام رمز OTP (طلب `PATCH /onLinePayment`). تحديث حالة الطلب إلى مدفوع عند النجاح. |
| `getAuthToken(): ?string` | طلب توكن OAuth 2.0 من YottaGate عبر `POST /login` باستخدام بيانات الاعتماد المخزنة. |
| `checkTransactionStatus($transactionId): array` | الاستعلام عن حالة معاملة باستخدام معرف المعاملة (Transaction ID) عبر `GET /checkAdjustmentByTransactionId`. |
| `changePassword($oldPassword, $newPassword): array` | تغيير كلمة مرور حساب التاجر عبر `PATCH /changeUserPassword`. |
| `getGateway(): Http` | إرجاع كائن `Http` مهيأ بالتوكن المناسب. |
| `settings(): array` | تعريف حقول الإعدادات في لوحة التحكم (URL، اسم المستخدم، كلمة المرور، التيرمينال الافتراضي، العملة الافتراضية). |
| `encryptedSettings(): array` | تحديد الحقول التي يتم تخزينها بشكل مشفر (`yottapay_password`). |

#### 2.2 `Http` – كلاس الاتصالات (CURL)
- **المسار**: `Nano\Yepayment\Classes\Http`
- **الوظيفة**: يوفر دوال منخفضة المستوى لإرسال طلبات HTTP (GET, POST, PATCH, JSON, Form-data) باستخدام cURL، مع دعم الوكيل وإعادة التوجيه. يستخدمه `YottaPay` لإرسال الطلبات إلى YottaGate.

**الدوال الرئيسية المستخدمة**:

| الدالة | الوصف |
|--------|-------|
| `postJsonData($url, $array_post, $headers, $basic)` | إرسال طلب POST مع `Content-Type: application/json`. |
| `getData($url, $headers)` | إرسال طلب GET وإرجاع النص الخام. |
| `getResponsData($response)` | تحويل الاستجابة (كائن أو سلسلة) إلى مصفوفة PHP. |

#### 2.3 `RedirectHelper` – كلاس مساعد لإعادة التوجيه
- **المسار**: `Nano\Yepayment\Classes\RedirectHelper`
- **الوظيفة**: يستخدم في نقاط نهاية الاختبار لإعادة التوجيه الذكية (دعم deeplinks والويب). غير مستخدم بشكل مباشر في `YottaPay` لأن الدفع لا يتطلب إعادة توجيه، لكنه مفيد لواجهات الاختبار.

#### 2.4 `yottapay.php` – ملف الإعدادات (اختياري)
- **المسار**: `config/yottapay.php` (يمكن إضافته مستقبلاً)
- **الوظيفة**: تخزين إعدادات الاتصال بـ YottaGate API، بما في ذلك عناوين URL لنقاط النهاية المختلفة. حالياً يتم استخدام `PaymentGatewaySettings` لتخزين الإعدادات.

#### 2.5 `Plugin.php` – تسجيل مزود الدفع
- **المسار**: `Nano\Yepayment\Plugin`
- **الوظيفة**: تسجيل `YottaPay` كمزود دفع في نظام `Nano.MicroCart` عبر دالة `registerPaymentProviders()` عند تفعيل خيار `allow_yemen_payment`.

#### 2.6 `routes.php` – نقاط نهاية API
- **المسار**: `routes.php`
- **الوظيفة**: توفير نقاط نهاية لاختبار بوابة YottaPay والاستعلام والإحصائيات.

**نقاط النهاية الخاصة بـ YottaPay**:

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/api/v1/yepayment/yottapay/test-auth` | POST | اختبار المصادقة مع YottaGate (الحصول على توكن) |
| `/api/v1/yepayment/yottapay/test-create-payment` | POST | إنشاء معاملة تجريبية (يحاكي `process`) |
| `/api/v1/yepayment/yottapay/test-confirm-payment` | POST | تأكيد معاملة باستخدام OTP |
| `/api/v1/yepayment/yottapay/test-check-status` | GET | الاستعلام عن حالة معاملة عبر `transaction_id` |
| `/api/v1/yepayment/yottapay/test-full-payment` | POST | اختبار شامل (إنشاء + تأكيد + استعلام) |
| `/api/v1/yepayment/yottapay/test-change-password` | POST | اختبار تغيير كلمة المرور |
| `/api/v1/yepayment/yottapay/stats` | GET | إحصائيات استخدام البوابة |
| `/api/v1/yepayment/yottapay/test-ui` | GET | واجهة ويب تفاعلية لاختبار جميع الوظائف |

#### 2.7 `_info.htm` – قالب معلومات الإعدادات
- **المسار**: `paymenttypes/yottapay/_info.htm`
- **الوظيفة**: عرض إرشادات الإعداد في لوحة التحكم، مع روابط الاختبار والوثائق.

#### 2.8 `_test_info.htm` – قالب أدوات الاختبار السريع
- **المسار**: `paymenttypes/yottapay/_test_info.htm`
- **الوظيفة**: تضمين أدوات الاختبار السريع (زر الاختبار، عرض الإحصائيات) ضمن صفحة إعدادات البوابة في لوحة التحكم.

#### 2.9 `yottapay-ui.htm` – واجهة اختبار متكاملة
- **المسار**: `views/yottapay-ui.htm`
- **الوظيفة**: واجهة HTML/JS كاملة لاختبار جميع وظائف YottaPay (اختبار يدوي، اختبار تلقائي، إحصائيات، سجلات).

---

### 3. دورة عمل الدفع (Payment Workflow)

يتم تنفيذ عملية الدفع عبر YottaPay وفق الخطوات التالية:

1. **إنشاء معاملة دفع**  
   عند استدعاء `process()`، يقوم الكلاس بـ:
   - التحقق من صحة المدخلات (رقم الهاتف، المبلغ، التيرمينال) عبر `checkValidate()`.
   - طلب توكن OAuth 2.0 عبر `getAuthToken()` باستخدام بيانات الاعتماد المخزنة في الإعدادات.
   - إرسال طلب `POST` إلى `{base_url}/api/accounts/v1/adjustment/onLinePayment` مع البيانات:
     ```json
     {
       "source": { "code": "771234567", "currencyId": "1" },
       "beneficiary": { "terminal": "1", "currencyId": "1" },
       "amount": "1000",
       "amountCurrencyId": "1",
       "note": "دفع الطلب #200"
     }
     ```
   - استلام `adjustment.id` من الاستجابة.
   - تخزين `adjustment.id` في `order.payment_first_trans_id` وفي `order.other_data['yottapay']`.
   - إرجاع `PaymentResult` مع `successful = true` ورسالة تطلب من المستخدم إدخال OTP (**بدون إعادة توجيه**).

2. **إدخال المستخدم لرمز OTP**  
   يتلقى العميل رسالة نصية تحتوي على رمز OTP من YottaGate (يتم إرساله تلقائياً بعد إنشاء المعاملة). يقوم المستخدم بإدخال الرمز في نموذج الدفع.

3. **تأكيد الدفع**  
   يتم استدعاء `complete()` مع إرسال رمز OTP:
   - إرسال طلب `PATCH` إلى نفس المسار `/onLinePayment` مع البيانات:
     ```json
     { "id": "816613", "otp": "4320", "note": "تأكيد الدفع" }
     ```
   - التحقق من `completed == true` في الاستجابة.
   - تحديث `order.payment_state` إلى `PaidState` وحفظ `transactionId` في `order.payment_trans_id`.
   - تسجيل الدفع كـ **ناجح** عبر `PaymentResult::success()`.

4. **التحقق من حالة المعاملة (اختياري)**  
   يمكن استدعاء `checkTransactionStatus($transactionId)` في أي وقت لإرسال طلب `GET` إلى `/checkAdjustmentByTransactionId` والاطلاع على `statusCode` (completed, not-completed, not-exist).

5. **استرجاع الأموال (Refund)**  
   (يمكن إضافته مستقبلاً) – تدعم YottaGate عمليات الاسترجاع عبر `POST /onlineMoneyReturn` و `PATCH /onlineMoneyReturn`، ويمكن تنفيذها عبر دوال إضافية في الكلاس.

---

### 4. الإعدادات والتهيئة (Configuration)

لتفعيل طريقة الدفع YottaPay، يجب إضافة الإعدادات التالية في واجهة إعدادات بوابة الدفع في نظام نانوسوفت (`Nano\MicroCart\Models\PaymentGatewaySettings`)، أو عبر متغيرات البيئة (`.env`) إذا تم دعمها:

```ini
# تفعيل YottaPay
YOTTAPAY_ENABLED=true
YOTTAPAY_URL=https://api.sabacash.com:49901
YOTTAPAY_USERNAME=712988875
YOTTAPAY_PASSWORD=your_password
YOTTAPAY_DEFAULT_TERMINAL=1
YOTTAPAY_DEFAULT_CURRENCY=1
```

**الإعدادات في لوحة التحكم** (حقل `settings()` في `YottaPay`):

| الإعداد | المفتاح | الوصف |
|---------|---------|-------|
| رابط API الأساسي | `yottapay_url` | عنوان YottaGate (مثال: `https://api.sabacash.com:49901`) |
| اسم المستخدم | `yottapay_username` | اسم مستخدم التاجر |
| كلمة المرور | `yottapay_password` | كلمة مرور التاجر (تخزن مشفرة) |
| رقم التيرمينال الافتراضي | `yottapay_default_terminal` | القيمة الافتراضية (مثلاً `"1"`) |
| معرف العملة الافتراضي | `yottapay_default_currency` | القيمة الافتراضية (مثلاً `"1"` للريال اليمني، `"2"` للريال السعودي) |

**ملاحظة**: يتم تخزين `yottapay_password` بشكل مشفر عبر `encryptedSettings()`.

---

### 5. الأمثلة التوضيحية (Usage Examples)

#### 5.1. إنشاء معاملة دفع لطلب موجود (ضمن تطبيق نانوسوفت)

```php
use Nano\Yepayment\PaymentTypes\YottaPay;
use Nano\Orders\Models\Order;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$provider = new YottaPay($order, [
    'source_phone' => '771234567',
    'amount'       => 1000,
    'terminal'     => '1',
    'currency_id'  => '1',
    'note'         => 'دفع الطلب #200',
]);

$result = new PaymentResult($provider, $order);
$processed = $provider->process($result);

if ($processed->successful) {
    // تخزين adjustment_id في الجلسة أو إرساله إلى الواجهة الأمامية
    $adjustmentId = $order->payment_first_trans_id;
    // عرض نموذج إدخال OTP للمستخدم
    return view('payment.otp_form', ['adjustment_id' => $adjustmentId]);
} else {
    return back()->withError($processed->message);
}
```

#### 5.2. تأكيد الدفع باستخدام OTP

```php
$order = Order::find(200);
$provider = new YottaPay($order, ['otp' => '4320']);
$result = new PaymentResult($provider, $order);
$confirmed = $provider->complete($result);

if ($confirmed->successful) {
    // الدفع تم بنجاح
    return redirect()->route('order.success', $order->id);
} else {
    return back()->withError('رمز OTP غير صحيح أو منتهي الصلاحية');
}
```

#### 5.3. الاستعلام عن حالة معاملة

```php
$yotta = new YottaPay();
$status = $yotta->checkTransactionStatus('DF-01-0123456-0123456789');
if ($status['success'] && $status['statusCode'] === 'completed') {
    echo "المعاملة مكتملة";
}
```

#### 5.4. اختبار الدفع عبر واجهة API (للمطورين)

يمكن استخدام نقاط النهاية المخصصة للاختبار (تتطلب صلاحيات مسؤول):

```bash
# اختبار المصادقة
curl -X POST "https://yourdomain.com/api/v1/yepayment/yottapay/test-auth" \
  -H "Authorization: Bearer <admin_token>"

# إنشاء معاملة تجريبية
curl -X POST "https://yourdomain.com/api/v1/yepayment/yottapay/test-create-payment" \
  -H "Authorization: Bearer <admin_token>" \
  -H "Content-Type: application/json" \
  -d '{"order_id":200,"source_phone":"771234567","amount":1000}'

# تأكيد المعاملة
curl -X POST "https://yourdomain.com/api/v1/yepayment/yottapay/test-confirm-payment" \
  -H "Authorization: Bearer <admin_token>" \
  -H "Content-Type: application/json" \
  -d '{"order_id":200,"adjustment_id":"816613","otp":"4320"}'

# الاستعلام عن الحالة
curl -X GET "https://yourdomain.com/api/v1/yepayment/yottapay/test-check-status?transaction_id=DF-01-..." \
  -H "Authorization: Bearer <admin_token>"
```

#### 5.5. استخدام واجهة الاختبار المتكاملة

يمكن فتح واجهة الاختبار في المتصفح (بعد تسجيل الدخول إلى لوحة التحكم كمسؤول):
```
https://yourdomain.com/api/v1/yepayment/yottapay/test-ui
```

تحتوي الواجهة على:
- اختبار يدوي خطوة بخطوة (المصادقة، إنشاء معاملة، تأكيد بـ OTP، التحقق من الحالة).
- اختبار تلقائي شامل مع إمكانية تحديد عدد مرات التكرار.
- إحصائيات فورية (عدد الطلبات، نسبة النجاح، آخر السجلات).
- سجلات الاختبارات المخزنة محلياً (LocalStorage) مع إمكانية التصدير والمسح.

---

### 6. التعامل مع عمليات إعادة التوجيه (Deeplinks & Callbacks)

بما أن YottaPay لا تعتمد على إعادة توجيه المستخدم إلى صفحات خارجية، فإن `success_url` و `cancel_url` غير مستخدمين في الدفع الأساسي. ومع ذلك، في نقاط نهاية الاختبار (`/yottapay/test-create-payment` وغيرها)، يتم استخدام `RedirectHelper` لإعادة التوجيه إلى واجهة الاختبار أو عرض النتائج بتنسيق JSON.

بالنسبة لسيناريوهات التكامل مع تطبيقات الجوال، يمكن للتاجر توفير `callback_success_url` و `callback_error_url` داخل `order.other_data` (على سبيل المثال عبر `Nano\Yepayment\Classes\RedirectHelper::getCallbackSuccessUrlByOrder()`) لتوجيه المستخدم بعد إتمام الدفع. لكن هذا غير مطلوب أساساً لأن الدفع يتم بالكامل داخل التطبيق دون مغادرة.

---

### 7. القيمة المضافة

- **للمطورين**: إضافة بوابات دفع جديدة عبر وراثة `PaymentProvider` مع دوال موحدة (`process`, `complete`). استخدام كلاس `Http` الموحد يقلل من تكرار كود الاتصال. توفر نقاط نهاية الاختبار وإحصائيات الأداء تسهل عملية التكامل والمراقبة.
- **للتجار**: دعم بوابة دفع محلية في اليمن (Sabacash) مع آلية دفع آمنة من خطوتين (OTP)، دون الحاجة إلى إعادة توجيه المستخدم إلى موقع خارجي، مما يحسن تجربة المستخدم ويقلل من معدلات التخلي عن الدفع.
- **للمستخدمين النهائيين**: تجربة دفع سلسة وسريعة باستخدام رمز OTP يُرسل إلى جوالهم، مع دعم العملات المحلية (الريال اليمني والريال السعودي).

---

### 8. اختبار التكامل (Integration Testing)

يوفر النظام نقاط نهاية مخصصة لاختبار التكامل دون الحاجة إلى طلب حقيقي:

- **بيئة الاختبار (UAT)**: يمكن استخدام بيانات دخول تجريبية من YottaGate (غير متاحة حالياً للعموم، يجب التنسيق مع المشغل).
- **بطاقات اختبار**: لا حاجة لبطاقات ائتمان لأن الدفع يعتمد على رصيد العميل ورمز OTP.
- **واجهة الاختبار المتكاملة**: `/api/v1/yepayment/yottapay/test-ui` توفر جميع الأدوات اللازمة لاختبار البوابة بشكل كامل.

**خطوات اختبار سريعة**:
1. تأكد من صحة بيانات الدخول (username/password) في إعدادات البوابة.
2. افتح واجهة الاختبار: `/api/v1/yepayment/yottapay/test-ui`.
3. انقر على "اختبار المصادقة" للتأكد من صلاحية البيانات.
4. أدخل رقم هاتف تجريبي (مثل `771234567`) والمبلغ المطلوب.
5. انقر على "إنشاء معاملة" – سيتم إنشاء معاملة وهمية.
6. استخدم رمز OTP تجريبياً (مثل `1234`) وانقر على "تأكيد الدفع".
7. تحقق من حالة المعاملة عبر "التحقق من الحالة".

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

- **لا تعتمد YottaPay على إعادة التوجيه**: لذلك تم إهمال `success_url` و `cancel_url` في الدفع الأساسي. يمكن إضافتهما لاحقاً إذا دعت الحاجة.
- **تخزين التوكن**: حالياً يتم طلب توكن جديد لكل عملية دفع. لتحسين الأداء، يمكن إضافة طبقة Cache لتخزين التوكن لمدة 3600 ثانية داخل `getAuthToken()`.
- **الاعتماد على `HttpHelper`**: جميع طلبات API تستخدم كلاس `Http` الموحد، مما يسهل تتبع الأخطاء وتوسيع البوابة.
- **الاختبار الآلي**: يمكن استخدام نقاط النهاية `/yottapay/test-full-payment` مع خيار `iterations` لاختبار استقرارية البوابة تحت ضغط.
- **توسيع الدوال**: يمكن إضافة دوال `refund()` و `getTransactionDetails()` مستقبلاً وفقاً لوثائق YottaGate API.

---

### 11. إصلاحات الأخطاء (Bug Fixes)

لا يوجد – هذا الإصدار مخصص لإضافة ميزة جديدة فقط.

---

### 12. روابط ذات صلة

- [وثيقة API الخاصة بـ YottaGate – تسجيل الدخول وتغيير كلمة المرور](./yottaPay%20Login%20-%20Change%20My%20Password%20API%20Documentation%20V.1.1.pdf)
- [وثيقة API الخاصة بـ YottaGate – إنشاء وتأكيد الدفع (Online Payments v1.3)](./API%20Online%20Payments%20v%201.3.pdf)
- [وثيقة API الخاصة بـ YottaGate – استرجاع الأموال (Online Payment Return v1.4)](./API%20Online%20Payment%20Return%20v%201.4.pdf)
- [وثيقة API الخاصة بـ YottaGate – التحقق من الحالة (Online Status Check v1.3)](./API%20Online%20Status%20Check%20v1.3.pdf)
- [مجموعة Postman لاختبار API Sabacash](./Sabacash%20online%20payment.postman_collection.json)
- [دليل تطوير بوابات الدفع – نانوسوفت](https://docs.nano2soft.com/payment-gateways)
- [قناة الدعم الفني](https://nano2soft.com)

---

**تم إعداد هذا التحديث بواسطة:**  
فريق تطوير نانوسوفت – قسم المدفوعات الإلكترونية  
**المراجع:** Dheia Ali, Nano2Soft

---
