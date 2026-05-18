# توثيق بوابة الدفع BasPay – منصة بس (مُحدَّث)

## 1. نظرة عامة

**BasPay** هي بوابة دفع تعمل وفق نمط **الدفع بخطوتين (Two‑Step Payment)** ومدمجة ضمن حزمة `Nano.Yepayment` في تطبيقات نانوسوفت. تعتمد البوابة على واجهة برمجة تطبيقات **منصة بس (BAS)**، وتتيح تنفيذ المدفوعات الإلكترونية من خلال إنشاء معاملة أولاً، ثم تأكيدها لاحقاً بعد أن يُتم العميل عملية الدفع عبر تطبيق بس.

على عكس البوابات الفورية، **لا تقوم BasPay بخصم المبلغ مباشرة** عند استدعاء واجهة برمجة التطبيقات، بل تتبع تدفقاً مكوناً من مرحلتين:

1. **`process()` – إنشاء المعاملة:** يرسل التاجر طلب إنشاء معاملة إلى منصة بس، ويستلم معرّفاً فريداً للمعاملة (`trxToken`). في هذه المرحلة يتم حفظ المعرّف في الطلب، ولكن **لا يتغير** الطلب إلى حالة "مدفوع".
2. **`complete()` – تأكيد المعاملة:** بعد أن يكمل العميل الدفع في تطبيق بس، يستعلم النظام عن حالة المعاملة باستخدام `trxToken`. إذا كانت الحالة ناجحة (`SUCCESS`)، يتم تحديث الطلب إلى حالة "مدفوع" وإطلاق الإجراءات المرتبطة بذلك.

تدعم البوابة العملات: الريال اليمني (`YER`)، الدولار الأمريكي (`USD`)، والريال السعودي (`SAR`). كما توفر دعماً اختيارياً لروابط إعادة التوجيه (Callback URLs) باستخدام `RedirectHelper` لتتوافق مع تطبيقات الجوال.

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

تعمل بوابة BasPay على مرحلتين منفصلتين، وهو ما يتوافق مع نمط "الدفع بخطوتين" في نظام `Nano.Yepayment`.

### 3.1. المرحلة الأولى: إنشاء المعاملة (`process`)

عند بدء الدفع لطلب معين، يتم استدعاء `process()` والتي تقوم بالخطوات التالية:

1. **التحقق من حالة الطلب** – إذا كان الطلب مدفوع مسبقاً (`PaidState`) يتم إيقاف العملية.
2. **الحصول على رمز الدخول (Access Token)** عبر OAuth 2.0 باستخدام `client_credentials` (يتم تخزينه في الـ Cache لمدة 3500 ثانية).
3. **توليد توقيع مشفر (Signature)** وفق آلية BAS:
   - إنشاء سلسلة عشوائية `salt` طولها 4 أحرف.
   - حساب `SHA256` للنص المطلوب توقيعه مع `salt`.
   - تشفير الناتج باستخدام `AES-256-CBC` بمفتاح التاجر والـ `IV`.
4. **إرسال طلب إنشاء معاملة** إلى `/api/v1/merchant/sdk-payment/initiate-transaction` مع بيانات الطلب والتوقيع.
5. **استلام الاستجابة** – إذا كانت `status = 1` و `code = '1111'` فإن المعاملة ناجحة، ويتم استخراج `trxToken` من `response.body.trxToken`.
6. **حفظ بيانات المعاملة** في الطلب دون تغيير حالة الدفع:
   - تعيين `order.payment_first_trans_id` و `order.payment_trans_id` بقيمة `trxToken`.
   - تخزين معلومات العملية في `order.other_data['baspay']` (الـ token، المبلغ، العملة، الطابع الزمني، روابط العودة).
   - **لا يتم** تغيير حالة الدفع إلى `PaidState` في هذه المرحلة.
7. **تسجيل سجل الدفع** في جدول `PaymentLog`.
8. إرجاع `PaymentResult` بنجاح جزئي (`successful = true`) مع رسالة تفيد بضرورة تأكيد الدفع (`confirmation_required`).

> **ملاحظة للمطوّر:** في هذه المرحلة، يعتبر النظام أن الطلب **معلق التأكيد**، وينتظر إكمال العميل للدفع عبر تطبيق بس.

### 3.2. المرحلة الثانية: تأكيد المعاملة (`complete`)

بعد أن يُكمل العميل عملية الدفع في تطبيق بس، يجب على النظام التحقق من حالة المعاملة وإتمام الطلب. يتم ذلك عبر دالة `complete()` (أو الدالة المساعدة `checkAndCompletePay`) والتي تقوم بالخطوات التالية:

1. **استخراج `trxToken`** من بيانات الطلب.
2. **الحصول على رمز الدخول** من جديد (أو من الكاش).
3. **إرسال طلب التحقق من الحالة** إلى `/api/v1/merchant/sdk-payment/get-transaction-status` مع `trxToken`.
4. **تحليل الاستجابة**:
   - إذا كانت `trxStatus` تساوي `SUCCESS` أو `COMPLETED`: يتم استدعاء `$result->success()` التي تقوم بتحديث الطلب إلى `PaidState`، وإطلاق الأحداث (مثل `nano.orders.paymentProcessed`)، وتفريغ السلة.
   - إذا كانت `PENDING` أو `PROCESSING`: يتم استدعاء `$result->pending()` وتبقى حالة الطلب معلقة.
   - أي حالة أخرى: يتم إرجاع فشل العملية.
5. **إرجاع النتيجة النهائية** لتقوم وحدات النظام الأخرى (مثل `RedirectHelper`) بإعادة التوجيه المناسبة.

### 3.3. روابط إعادة التوجيه الاختيارية (Callbacks)

تدعم BasPay تخزين روابط `callback_success_url` و `callback_error_url` إن تم تمريرها ضمن بيانات الدفع (مثلاً من تطبيق جوال). بعد تأكيد الدفع، يمكن للمطور توجيه المستخدم باستخدام المسارين:

- `GET /api/v1/yepayment/baspay/success` – يسترجع رابط العودة عند النجاح ويستخدم `RedirectHelper` لإعادة التوجيه إلى التطبيق أو الموقع.
- `GET /api/v1/yepayment/baspay/cancel` – مشابه لحالة الإلغاء.

هذا يضمن توافقاً كاملاً مع تطبيقات الجوال التي تحتاج إلى Deep Links بعد الدفع.

---

## 4. هيكل البيانات المخزنة في `order->other_data['baspay']`

```php
[
    'trx_token'             => 'bas_trx_a1b2c3d4...',  // المعرف الفريد للمعاملة من BAS
    'amount'                => 100.00,                  // المبلغ
    'currency'              => 'YER',                   // العملة
    'order_id'              => '200',                   // رقم الطلب
    'requires_confirmation' => true,                    // هل يتطلب تأكيداً (دائماً true)
    'created_at'            => '2025-01-01T12:00:00Z', // وقت إنشاء المعاملة
    'callback_success_url'  => 'myapp://pay/success',  // (اختياري) رابط العودة عند النجاح
    'callback_error_url'    => 'myapp://pay/error'     // (اختياري) رابط العودة عند الخطأ
]
```

---

## 5. سيناريو متكامل لاستخدام BasPay في تطبيقك

يوضّح هذا السيناريو الخطوات التي يقوم بها مطور التطبيق أو المتجر للتعامل مع BasPay من خلال واجهة `Checkout2` API.

### 5.1. المتطلبات الأساسية
- تطبيق جوال أو موقع يستخدم `Nano.Yepayment` و `Nano.ShopApi`.
- تم إعداد بوابة BasPay في لوحة التحكم.
- يمتلك العميل حساباً في منصة بس (لتأكيد الدفع لاحقاً).

### 5.2. الخطوات من وجهة نظر المطور

#### **الخطوة 1: يختار المستخدم طريقة الدفع BasPay**
يرسل التطبيق طلباً إلى نقطة النهاية `checkout` مع `step=pay` ومعرف طريقة الدفع `baspay`.

**مثال الطلب:**
```http
POST /api/v1/shop/checkout
Content-Type: application/json

{
  "step": "pay",
  "payment_method_id": 5,
  "callback_success_url": "myapp://checkout/success",
  "callback_error_url": "myapp://checkout/error"
}
```

#### **الخطوة 2: يعالج النظام الدفع عبر `Checkout2` و `OrderManager`**
- يستدعي النظام `OrderManager->setStepPayments()` الذي بدوره:
  1. يتحقق من صحة عناوين الشحن والكوبونات.
  2. ينشئ `PaymentService` و `PaymentGateway` بناءً على طريقة الدفع المختارة (BasPay).
  3. يستدعي `BasPay->process()`.

#### **الخطوة 3: `BasPay->process()` تنشئ المعاملة وتُعيد `trxToken`**
- تتواصل البوابة مع BAS API وتُنشئ معاملة جديدة.
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
        "message": "يرجى تأكيد الدفع من خلال تطبيق بس",
        "api_data": {
          "trx_token": "bas_trx_a1b2c3d4..."
        }
      }
    }
  }
  ```
- **المهم هنا:** `processed` لا تزال `false`، مما يعني أن الدفع لم يكتمل.

#### **الخطوة 4: يطلب المطور من العميل إكمال الدفع في تطبيق بس**
- يعرض التطبيق رسالة للعميل: "يرجى فتح تطبيق بس وإكمال الدفع".
- يمكن أن يشتمل ذلك على زر "افتح بس" أو Deep Link للتطبيق.
- في هذه الأثناء، المطور لديه `trxToken` (موجود في `payment_first_trans_id` و `other_data`).

#### **الخطوة 5: يتأكد المطور من إتمام الدفع**
بعد أن يُكمل العميل الدفع في تطبيق بس، لدى المطور طريقتين:

**الطريقة أ: استخدام مسار `baspay/success` مع روابط العودة (موصى به لتطبيقات الجوال):**
- عندما يعود المستخدم من تطبيق بس إلى تطبيقك عبر Deep Link `myapp://checkout/success?order_id=200`، يقوم التطبيق باستدعاء:
  ```http
  GET /api/v1/yepayment/baspay/success?order_id=200
  ```
- يستجيب المسار باستخدام `RedirectHelper` ويعيد توجيه المستخدم إلى الصفحة الصحيحة مع نتيجة الدفع.

**الطريقة ب: الاستعلام المباشر عن الحالة (للتطبيقات بدون Deep Links):**
- يمكن للمطور استدعاء `POST /api/v1/yepayment/baspay/test-check-status` (للمسؤولين) أو إنشاء نقطة نهاية مخصصة تستخدم `BasPay::checkAndCompletePay()`.
- مثال على الاستدعاء اليدوي:
  ```php
  $result = \Nano\Yepayment\PaymentTypes\BasPay::checkAndCompletePay(['order_id' => 200]);
  if ($result['success']) {
      // تم الدفع، تحديث واجهة المستخدم
  }
  ```

#### **الخطوة 6: يتغير الطلب إلى مدفوع**
- بمجرد نجاح `complete()`، يستدعي النظام `$result->success()` الذي:
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
/api/v1/yepayment/baspay/test-ui
```

### 6.1. الميزات

- **اختبار المصادقة** (Auth) – التحقق من صحة بيانات Client ID/Secret عبر `/baspay/test-auth`.
- **إنشاء معاملة (Initiate)** – إدخال رقم الطلب، المبلغ، العملة لإنشاء معاملة جديدة واستلام `trxToken`.
- **التحقق من الحالة** – الاستعلام عن معاملة باستخدام `trxToken` ومعرفة حالتها.
- **اختبار شامل (Full Flow)** – يجمع بين إنشاء المعاملة والتحقق من الحالة في خطوة واحدة (يحاكي المرحلتين).
- **اختبار تلقائي متكرر** – إمكانية تحديد عدد مرات التكرار لقياس الاستقرار.
- **إحصائيات فورية** – عدد الطلبات، نسبة النجاح، آخر السجلات.
- **سجلات محلية** – تخزن نتائج الاختبارات في `localStorage` للمتصفح.

### 6.2. نقاط نهاية API المستخدمة في الاختبار

| الغرض | الطريقة | المسار |
|-------|--------|--------|
| المصادقة | POST | `/baspay/test-auth` |
| إنشاء معاملة | POST | `/baspay/test-create-payment` |
| التحقق من الحالة | POST | `/baspay/test-check-status` (body: `{trx_token}`) |
| اختبار شامل (المرحلتين) | POST | `/baspay/test-full-payment` |
| إحصائيات | GET | `/baspay/stats` |

---

## 7. أمثلة على الطلبات والاستجابات

### 7.1. إنشاء معاملة جديدة (المرحلة الأولى)

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
    "message": "يرجى تأكيد الدفع من خلال تطبيق بس",
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

### 7.2. التحقق من حالة معاملة (المرحلة الثانية)

**طلب:**
```http
POST /api/v1/yepayment/baspay/test-check-status
Content-Type: application/json

{
    "trx_token": "bas_trx_a1b2c3d4..."
}
```

**استجابة (مثال – نجاح):**
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

### 7.3. اختبار شامل (المرحلتين معاً)

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
            "trxStatus": "SUCCESS"
        }
    }
}
```

---

## 8. دمج البوابة في مشروعك (Developer Guide)

### 8.1. تسجيل طريقة الدفع في `Plugin.php`

```php
if (config('nano.microcart::paymenttypes.allow_yemen_payment', true)) {
    // ... المزودون الآخرون ...
    if (class_exists(\Nano\Yepayment\PaymentTypes\BasPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\BasPay();
    }
}
```

### 8.2. استخدام البوابة عبر `PaymentGateway` (المرحلة الأولى فقط)

```php
use Nano\MicroCart\Classes\Payments\DefaultPaymentGateway;
use Nano\Orders\Models\Order;

$order = Order::find($orderId);
$gateway = new DefaultPaymentGateway();
$gateway->init($paymentMethod, ['callback_success_url' => 'myapp://pay/success']);
$result = $gateway->process($order);
if ($result->successful) {
    // تم إنشاء المعاملة – يجب توجيه العميل لتطبيق بس
    $trxToken = $order->payment_first_trans_id;
}
```

### 8.3. تأكيد الدفع يدوياً (المرحلة الثانية)

بعد أن يؤكد العميل الدفع في تطبيق بس، يمكنك استخدام الدالة المساعدة:

```php
use Nano\Yepayment\PaymentTypes\BasPay;

$result = BasPay::checkAndCompletePay(['order_id' => 200]);
if ($result['success']) {
    // تم الدفع بنجاح
}
```

---

## 9. استكشاف الأخطاء (Troubleshooting)

| المشكلة | السبب المحتمل | الحل |
|---------|---------------|------|
| فشل المصادقة (`auth_failed`) | Client ID/Secret غير صحيح | تحقق من الإعدادات المدخلة في لوحة التحكم |
| `payment_creation_failed` مع استجابة خطأ من BAS | `trxToken` مفقود أو `code` لا يساوي `1111` | افحص الـ logs للاستجابة الكاملة؛ قد يكون المبلغ أقل من الحد الأدنى أو العملة غير مدعومة |
| توقيع غير صحيح | Merchant Key أو IV غير متطابقين | تأكد من مطابقة القيم مع تلك المقدمة من منصة بس |
| الطلب مدفوع مسبقاً | `order.payment_state == PaidState` | لا يمكن الدفع مرة أخرى لنفس الطلب |
| خطأ `cURL error` | مشكلة في الاتصال بـ API | تحقق من صحة الرابط ووجود اتصال إنترنت |
| حالة المعاملة غير معروفة عند التحقق | `trxToken` غير صحيح | تأكد من أن التوكن مستخدم في طلب التحقق هو نفسه الناتج عن `initiateTransaction` |
| لم يتغير الطلب إلى مدفوع بعد `complete` | `trxStatus` ليست `SUCCESS` | تحقق من حالة المعاملة في لوحة تحكم بس؛ قد يكون العميل لم يؤكد الدفع بعد |

---

## 10. ملاحظات إضافية

- **البوابة تتبع نمط Two‑Step**، وبالتالي لا يتم الدفع مباشرة. يجب استخدام `complete()` أو `checkAndCompletePay()` لتأكيد الدفع.
- التوقيع المشفر يعتمد على نفس الخوارزمية المستخدمة في **BAS SDK** (`AES-256-CBC + SHA256 + salt`)، مما يضمن التوافق التام مع متطلبات منصة بس.
- يتم تخزين `Access Token` في الـ Cache لمدة 3500 ثانية لتقليل عدد طلبات المصادقة.
- يمكن تمديد الكلاس لإضافة خاصية **Webhook** أو استرداد الأموال مستقبلاً دون كسر البنية الحالية.
- **دالة `checkAndCompletePay`** هي دالة `static` عامة تُستخدم لربط مرحلة التأكيد مع مسار `baspay/success`.

---

## 11. ملفات الكود الخاصة بالبوابة

| الملف | الوصف |
|-------|-------|
| `BasPay.php` | الكلاس الرئيسي للبوابة (يمتد من `PaymentProvider`) |
| `_info.htm` | قالب معلومات الإعدادات في لوحة التحكم |
| `_test_info.htm` | أدوات الاختبار السريع مع أزرار تفاعلية |
| `baspay-ui.htm` | واجهة الاختبار الكاملة (Test UI) |
| `routes.php` (قسم BasPay) | تعريف نقاط نهاية الاختبار والإحصائيات وواجهة المستخدم |
| `Plugin.php` (إضافة) | تسجيل البوابة كمزود دفع |

---

## 12. روابط ذات صلة

- [وثيقة SKILL-ar.md لإنشاء بوابات الدفع](./SKILL-ar.md)
- [دليل منصة بس API](https://basgate.apidog.io)
- [دليل منصة بس API](./external/BAS/README-ar.md)
- [مستودع وثائق BasGate على GitHub](https://github.com/basgate/basgate.github.io)
- [حزمة Laravel Payment SDK على GitHub](https://github.com/basgate/laravel-payment-sdk)
- [مستودع BasPaymentFlutter على GitHub](https://github.com/BasPlatform/BasPaymentFlutter.git)
- [حزمة bas_php_sdk على GitHub](https://github.com/basgate/bas_php_sdk)
- [حزمة bas-laravel-sdk على GitHub](https://github.com/basgate/bas-laravel-sdk)


**تم إعداد هذا التوثيق لدعم المطورين في دمج واستخدام بوابة BasPay بسهولة وكفاءة.**  
للاستفسارات أو الدعم الفني، يرجى التواصل عبر الموقع الرسمي [nano2soft.com](https://nano2soft.com).
