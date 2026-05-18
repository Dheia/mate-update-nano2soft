# SKILL: إنشاء طريقة دفع جديدة ضمن إضافة `Nano\Yepayment`

## 📖 نظرة عامة

يتيح نظام `Nano.Yepayment` إضافة بوابات دفع جديدة باتباع نمط موحد يضمن التكامل السلس مع `Nano.MicroCart` و `Nano.Orders`. تتيح هذه المهارة للمطور إنشاء أي مزود دفع باتباع الخطوات والمعايير المحددة، مع دعم ثلاثة أنواع رئيسية من تدفقات الدفع.

---

## 🧩 أنواع طرق الدفع (Payment Flows)

| النوع | الوصف | مثال | دوال رئيسية |
|-------|-------|------|--------------|
| **نوع Redirect** | يقوم بإعادة توجيه المستخدم إلى بوابة خارجية (البنك، PayPal، إلخ) لإتمام الدفع، ثم يعود إلى `success_url` أو `cancel_url`. | `ThawaniPay` | `process()` → `$result->redirect()`<br>مسار `success` يكمل الدفع. |
| **نوع Two-Step** | ينشئ معاملة أولاً (process)، ثم يتطلب تأكيداً لاحقاً (complete) عبر إدخال OTP أو رمز من العميل. | `YottaPay` | `process()` يعيد `successful = true` ورسالة "أدخل OTP".<br>`complete()` يؤكد الدفع. |
| **نوع Direct (فوري)** | يتم الدفع فوراً عبر API دون حاجة لإعادة توجيه أو تأكيد إضافي. | `QasemiPay` (في حال كان التأكيد تلقائياً) | `process()` ينفذ الدفع ويعيد `$result->success()` مباشرة. |

> **ملاحظة:** يمكن لبوابة واحدة أن تجمع بين أكثر من نمط (مثل إنشاء معاملة ثم إعادة توجيه، أو تأكيد لاحق). اختر النمط المناسب حسب وثائق API للبوابة.

---

## 🏗️ هيكل كلاس طريقة الدفع

**الموقع:** `plugins/nano/yepayment/paymenttypes/NewPay.php`

```php
<?php namespace Nano\Yepayment\PaymentTypes;

use Nano\MicroCart\Classes\Payments\PaymentProvider;
use Nano\MicroCart\Classes\Payments\PaymentResult;
use Nano\MicroCart\Models\PaymentGatewaySettings;
use Nano\Helpers\Classes\Helpers\HttpHelper;
use Nano\Yepayment\Classes\RedirectHelper; // عند الحاجة لإعادة التوجيه
use Config, Request, Validator, Throwable, ApplicationException;

class NewPay extends PaymentProvider
{
    // خصائص إلزامية
    public $success_url = 'api/v1/yepayment/newpay/success';
    public $cancel_url  = 'api/v1/yepayment/newpay/cancel';
    public $is_test_mod = false;

    // دوال مجردة يجب تنفيذها
    public function name(): string { return 'New Payment Gateway'; }
    public function identifier(): string { return 'newpay'; }
    public function validate(): bool { return true; }
    public function settings(): array { /* ... */ }
    public function process(PaymentResult $result): PaymentResult { /* ... */ }
    public function complete(PaymentResult $result): PaymentResult { /* ... */ }
    public function encryptedSettings(): array { return ['newpay_password']; }

    // دوال مساعدة (حسب الحاجة)
    public function defineValidationRules(): array { /* ... */ }
    public function getFieldNames(): array { /* ... */ }
    private function getAuthToken(): ?string { /* ... */ }
    private function createPayment($token): array { /* ... */ }
    private function confirmPayment($token, $id, $code): array { /* ... */ }
    public function checkTransactionStatus($refId): array { /* ... */ }
    private function getApiUrl(string $type): string { /* ... */ }
    private function parseResponse($response): array { /* ... */ }
}
```

---

## 📝 خطوات الإنشاء التفصيلية

### 1. إنشاء الكلاس الرئيسي

- قم بإنشاء ملف PHP جديد في `paymenttypes/`.
- تأكد من أن الكلاس يمتد `PaymentProvider`.
- حدد `$success_url` و `$cancel_url` حسب مجموعة التوجيه المستخدمة (`yepayment` أو `ompayment`).
- عرّف `$is_test_mod` للتبديل بين بيئات الاختبار والإنتاج.

### 2. تنفيذ الدوال المجردة

#### `name(): string`
- أعد الاسم التجاري للبوابة كما سيظهر للمستخدم.

#### `identifier(): string`
- أعد معرف فريد (أحرف لاتينية صغيرة، بدون مسافات). يُستخدم لتخزين السجلات وتمييز البوابة.

#### `validate(): bool`
- يمكن إرجاع `true` واستخدام `defineValidationRules()` بشكل منفصل، أو تنفيذ التحقق يدوياً.

#### `settings(): array`
- عرّف حقول الإعدادات التي ستظهر في لوحة التحكم (URL، مفاتيح API، اسم المستخدم، إلخ).
- استخدم `type => 'partial'` لتضمين ملفات `_info.htm` و `_test_info.htm`.
- استخدم `type => 'password'` للحقول الحساسة، وأضفها إلى `encryptedSettings()`.

#### `process(PaymentResult $result): PaymentResult`
هذا هو قلب منطق الدفع. الخطوات النموذجية:

```php
public function process(PaymentResult $result): PaymentResult
{
    try {
        // 1. التحقق من صحة بيانات الإدخال (إن وجدت)
        $validator = Validator::make($this->data, $this->defineValidationRules());
        if ($validator->fails()) throw new ValidationException($validator);

        // 2. التأكد من أن الطلب غير مدفوع مسبقاً
        if ($this->order->payment_state == PaidState::class) {
            $result->successful = false;
            $result->message = trans('nano.yepayment::lang.public.newpay.errors.order_already_paid');
            return $result;
        }

        // 3. الحصول على توكن المصادقة (إن لزم)
        $token = $this->getAuthToken();
        if (!$token) throw new ApplicationException('Auth failed');

        // 4. إنشاء معاملة الدفع عبر API
        $paymentResult = $this->createPayment($token);
        if (!$paymentResult['success']) throw new ApplicationException($paymentResult['message']);

        // 5. حفظ بيانات المعاملة في order->other_data و payment_first_trans_id
        $this->order->payment_first_trans_id = $paymentResult['transaction_id'];
        $this->order->payment_trans_id = $paymentResult['ref_no'] ?? null;
        $other = $this->order->other_data;
        $other['newpay'] = [ /* بيانات إضافية */ ];
        $this->order->other_data = $other;
        $this->order->save();

        // 6. تحديد مسار الدفع بناءً على نوع البوابة
        if (isset($paymentResult['redirect_url'])) {
            return $result->redirect($paymentResult['redirect_url']);
        } elseif ($paymentResult['requires_confirmation'] ?? false) {
            $result->successful = true;
            $result->message = 'يرجى إدخال رمز التأكيد';
            $result->api_data = $paymentResult;
            return $result;
        } else {
            // دفع فوري
            $this->order->payment_state = PaidState::class;
            $this->order->save();
            return $result->success($paymentResult, null);
        }
    } catch (Throwable $e) {
        return $result->fail([], $e);
    }
}
```

#### `complete(PaymentResult $result): PaymentResult`
- يُستخدم في حالة النوع **Two-Step** أو عند الحاجة إلى تأكيد إضافي بعد عودة المستخدم من بوابة خارجية.
- يجب استرداد `transaction_id` من `payment_first_trans_id` أو من `other_data`.
- طلب تأكيد الدفع من API، ثم تحديث حالة الطلب إلى `PaidState` عبر `$result->success()`.

### 3. استخدام `HttpHelper` لطلبات API

يجب استخدام الكلاس `Nano\Helpers\Classes\Helpers\HttpHelper` لتنفيذ جميع طلبات HTTP (GET, POST, PUT, PATCH, DELETE). هذا الكلاس يوفر واجهة موحدة لـ Guzzle ويدعم JSON، Form Data، Multipart.

**استيراد الكلاس:**
```php
use Nano\Helpers\Classes\Helpers\HttpHelper;
```

**أمثلة عملية من البوابات الحالية:**

#### أ. طلب JSON (مثل YottaPay – إنشاء معاملة):
```php
$response = HttpHelper::sendJson([
    'method' => 'POST',
    'url' => $this->getApiUrl('payment'),
    'json' => $payload,
    'options' => [
        'headers' => [
            'Authorization' => 'Bearer ' . $token,
            'Content-Type' => 'application/json',
        ],
        'timeout' => 30,
    ],
]);
```

#### ب. طلب Form Data (مثل QasemiPay – الحصول على التوكن):
```php
$response = HttpHelper::sendForm([
    'method' => 'POST',
    'url' => $this->getApiUrl('token'),
    'form_params' => [
        'client_id' => $clientId,
        'client_secret' => $clientSecret,
        'grant_type' => 'password',
        // ...
    ],
    'options' => [
        'headers' => [
            'Content-Type' => 'application/x-www-form-urlencoded',
        ],
    ],
]);
```

#### ج. طلب GET مع Query Parameters:
```php
$response = HttpHelper::get([
    'url' => $this->getApiUrl('check_status') . '/' . $refId,
    'options' => [
        'query' => ['param' => 'value'],
        'headers' => ['Accept' => 'application/json'],
    ],
]);
```

#### د. طلب PATCH (مثل YottaPay – تأكيد الدفع):
```php
$response = HttpHelper::sendJson([
    'method' => 'PATCH',
    'url' => $this->getApiUrl('payment'),
    'json' => ['id' => $id, 'otp' => $otp],
    'options' => [...],
]);
```

**معالجة الاستجابة:**  
بعد تنفيذ الطلب، استخدم الدالة المساعدة `parseResponse()` لتحويل الاستجابة إلى مصفوفة (كما في `YottaPay`, `QasemiPay`, `ThawaniPay`):

```php
private function parseResponse($response): array
{
    $body = $response->getBody()->getContents();
    $contentType = $response->getHeaderLine('content-type');
    if (strpos($contentType, 'application/json') !== false) {
        return json_decode($body, true) ?: [];
    }
    return ['raw_body' => $body, 'status_code' => $response->getStatusCode()];
}
```

### 4. إدارة التوكن (Token)

- إذا كانت البوابة تتطلب توكن (OAuth، JWT)، قم بإنشاء دالة `getAuthToken()`.
- يُفضل تخزين التوكن في Cache لتجنب طلبات المصادقة المتكررة:

```php
use Illuminate\Support\Facades\Cache;

private function getAuthToken(): ?string
{
    return Cache::remember('newpay_token', 3500, function () {
        // طلب التوكن من API باستخدام HttpHelper
        $response = HttpHelper::sendForm([...]);
        return $response['access_token'] ?? null;
    });
}
```

### 5. التعامل مع إعادة التوجيه (Redirect) و `RedirectHelper`

للبوابات من نوع **Redirect** (مثل `ThawaniPay`)، يجب استخدام الكلاس `Nano\Yepayment\Classes\RedirectHelper` لإعادة التوجيه إلى التطبيقات أو المواقع بعد الدفع. هذا الكلاس:

- يستخرج روابط `callback_success_url` و `callback_error_url` من `order->other_data`.
- يدعم Deep Links (مثل `myapp://`) والروابط العادية.
- يتعامل مع البيانات الكبيرة عبر تخزينها في Cache باستخدام Token.
- يضمن الأمان عبر التحقق من النطاقات المسموحة وتصفية البيانات الحساسة.

**استيراد الكلاس:**
```php
use Nano\Yepayment\Classes\RedirectHelper;
```

**أمثلة عملية من `routes.php` (ThawaniPay):**

#### أ. استخراج رابط إعادة التوجيه من بيانات الطلب:
```php
$callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, ['callback_success_url', 'thawani.callback_success_url']);
```

#### ب. إعادة التوجيه إلى التطبيق أو الويب:
```php
return RedirectHelper::redirectToApp($callbackUrl, $data, $forceJson, $queryParams, $deepLinkSchemes, $is_force_deep_list);
```
- `$callbackUrl`: الرابط المستخرج (قد يكون `https://...` أو `myapp://...`).
- `$data`: البيانات التي تريد إرسالها (حالة الدفع، رقم الطلب، إلخ).
- `$forceJson`: إذا كان `true`، يُرجع JSON بدلاً من إعادة التوجيه.
- `$queryParams`: مصفوفة بأسماء الحقول التي تريد إضافتها كـ query parameters (مثل `['order_id', 'payment_method_id']`).
- `$deepLinkSchemes`: قائمة بـ schemes المسموحة للتطبيقات (مثال `['myapp', 'app', '*']`). `'*'` يعني أي scheme ليس http/https.
- `$is_force_deep_list`: إذا كان `true`، يُلزم باستخدام القائمة المحددة فقط.

#### ج. حالات بديلة عند عدم وجود رابط:
```php
if (!$callbackUrl) {
    return Response::make('تم الدفع بنجاح. يمكنك العودة إلى التطبيق.', 200);
}
```

#### د. دالة عامة للتحقق وإتمام الدفع (مثل `checkAndCompletePay` في ThawaniPay):
يمكنك إنشاء دالة `static` في كلاس البوابة تستدعي `RedirectHelper` داخلياً لتوحيد المنطق.

**ملاحظة:** يجب إضافة مسارات `success` و `cancel` في `routes.php` لاستقبال العودة من البوابة الخارجية، ثم استخدام `RedirectHelper` داخلها كما في `thawanipay/success`.

### 6. التعامل مع `PaymentResult` وتأثيره على حالة الطلب والتكامل مع النظام

**هذا القسم يشرح العلاقة الدقيقة بين البوابة وحالة الطلب، وكيفية انسجامها مع `OrderManager` و`Checkout2`.**

#### 6.1 دوال `PaymentResult` وتأثيرها الكامل

| الدالة | متى تُستخدم | التأثير الكامل |
|--------|--------------|----------------|
| `$result->success($data, $response)` | **فقط** عندما تتأكد البوابة أن عملية الدفع قد تمت بشكل نهائي ولا رجعة فيه. | 1. يعين `successful = true`.<br>2. **يغير حالة الطلب إلى `PaidState`**.<br>3. يضع `processed = true`.<br>4. يسجل `PaymentLog` ناجحاً.<br>5. **يُشغل حدث `nano.orders.paymentProcessed`** (إرسال الإيميلات، تحديث المخزون، إلخ).<br>6. **يُفرغ سلة المستخدم** تلقائياً. |
| `$result->fail($data, $response)` | عند فشل العملية. | 1. يعين `successful = false`.<br>2. **يغير حالة الطلب إلى `FailedState`**.<br>3. يسجل `PaymentLog` فاشلاً.<br>4. لا يتم تفريغ السلة. |
| `$result->redirect($url)` | لإعادة التوجيه إلى بوابة خارجية. | 1. يعين `redirect = true` و `redirectUrl`.<br>2. **لا يغير حالة الطلب**. |
| `$result->pending($data, $response)` | عندما تكون المعاملة معلقة وغير مكتملة (مثلاً في انتظار تأكيد خارجي). | 1. يعين `successful = true`.<br>2. **يغير حالة الطلب إلى `PendingState`** (لا يصبح مدفوعاً).<br>3. يضع `processed = true` لكن الحالة ليست مدفوعة.<br>4. يسجل `PaymentLog` ناجحاً.<br>5. **لا يُشغل حدث `paymentProcessed`**.<br>6. **يُفرغ السلة** رغم أن الطلب لم يُدفع. |

> ⚠️ **قاعدة ذهبية:** لا تستدعي `$result->success()` داخل `process()` إذا كانت البوابة من النوع **Two‑Step**. استخدمها فقط داخل `complete()` بعد التحقق من نجاح الدفع. أما في النوع **Direct**، فيمكن استخدام `$result->success()` مباشرةً في `process()` لأن الدفع فوري.

#### 6.2 تفاعل البوابة مع `OrderManager` و `Checkout2` (انسجام الكلاس مع باقي النظام)

عندما يصل المستخدم لخطوة الدفع (`step=pay`) في تطبيق العميل، يكون تدفق الاستدعاء كالتالي:
- `Checkout2` يستدعي `$this->orderManager->setStepPayments($data)`.
- `OrderManager` بدوره:
  1. يتحقق من صحة الخطوات السابقة (العناوين، الشحن، الكوبونات).
  2. ينشئ `PaymentGateway` و `PaymentService`.
  3. يستدعي `$paymentService->process()` الذي بدوره يستدعي `$gateway->process($order)` الذي يمرر الاستدعاء إلى `BasPay->process()` (أو أي بوابة أخرى).
  4. **بعد عودة `PaymentResult` من البوابة**، يتصرف `OrderManager` بناءً على حالته:
     - إذا كانت `$order->isPaymentProcessed()` ترجع `true` (أي أن الطلب انتقل إلى `PaidState`) → يعتبر الدفع مكتملاً.
     - إذا كانت ترجع `false` (كما في حالة BasPay بعد `process` فقط) → يعرض رسالة للمستخدم بضرورة إكمال الدفع (مثل "يرجى تأكيد الدفع من تطبيق بس").

**النتيجة:** في البوابات Two‑Step، بعد `process()` مباشرةً، يكون `processed = false` والطلب **ليس** مدفوعاً، مما يسمح للمستخدم بإكمال الخطوة التالية. عدم استدعاء `$result->success()` داخل `process()` هو ما يمنع تحويل الطلب إلى `PaidState` قبل الأوان.

#### 6.3 إكمال الدفع في البوابات Two‑Step: `complete()` و `checkAndCompletePay`

لإتمام الدفع، يجب استدعاء `complete()` عند عودة المستخدم من التأكيد. الطريقة المثلى هي توفير **دالة `static` عامة** باسم `checkAndCompletePay` تُستخدم في مسار `success`. مثال:

```php
public static function checkAndCompletePay(array $options): array
{
    $orderId = $options['order_id'] ?? null;
    $order = Order::find($orderId);
    if (!$order) return ['success' => false, 'message' => 'Order not found'];
    
    $bas = new self($order);
    $result = new PaymentResult($bas, $order);
    $completeResult = $bas->complete($result);

    return [
        'success'  => $completeResult->successful,
        'message'  => $completeResult->message,
        'data'     => $completeResult->api_data,
        'order_id' => $orderId,
    ];
}
```

ثم في `routes.php`:

```php
Route::get('baspay/success', function () {
    $options = Input::get();
    $result = BasPay::checkAndCompletePay($options);
    $order_id = $result['order_id'] ?? null;
    if ($order_id) {
        $callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, ['callback_success_url', 'baspay.callback_success_url']);
        if ($callbackUrl) {
            return RedirectHelper::redirectToApp($callbackUrl, $result, false, ['order_id', 'payment_method_id'], ['*'], false);
        }
    }
    return response()->json($result);
});
```

> **ملاحظة:** `complete()` يجب أن تستدعي `$result->success()` فقط عندما تكون المعاملة ناجحة، وإلا تستخدم `$result->pending()` أو `$result->fail()` حسب الحالة.

---

## 🧩 إنشاء الملفات الجزئية (Partials)

### 1. `_info.htm`
**المسار:** `paymenttypes/newpay/_info.htm`  
يعرض معلومات تعريفية عن البوابة في صفحة الإعدادات.

```html
<div class="callout callout-info">
    <h4>New Payment Gateway</h4>
    <p>بوابة دفع متكاملة تتيح قبول المدفوعات عبر الإنترنت.</p>
    <hr>
    <strong>متطلبات الإعداد:</strong>
    <ol>
        <li>الحصول على API Key من بوابة الدفع.</li>
        <li>تحديد عنوان API الأساسي (Live/Sandbox).</li>
    </ol>
</div>
```

### 2. `_test_info.htm`
**المسار:** `paymenttypes/newpay/_test_info.htm`  
يوفر أداة اختبار سريع داخل صفحة الإعدادات.

**ملاحظات هامة:**
- جميع المعرفات (IDs) وأسماء الدوال يجب أن تكون فريدة باستخدام بادئة البوابة (مثل `newpay-`) لتجنب التعارض مع بوابات أخرى.
- استخدام `const newpayApiBaseUrl = '/api/v1/yepayment';` (أو `/ompayment` حسب المجموعة).
- الأزرار تستمع إلى الأحداث عبر `addEventListener`.
- عرض النتائج في `div` مخصص.

**نموذج مبسط:**
```html
<div id="newpay-test-widget">
    <div class="row">
        <div class="col-md-6">
            <div class="form-group">
                <label>رقم الطلب:</label>
                <input type="text" id="newpay-test-order-id" class="form-control" value="200">
            </div>
            <div class="form-group">
                <label>المبلغ:</label>
                <input type="number" id="newpay-test-amount" class="form-control" value="100">
            </div>
        </div>
        <div class="col-md-6">
            <div class="form-group">
                <label>رمز التأكيد:</label>
                <input type="text" id="newpay-test-code" class="form-control" value="1234">
            </div>
        </div>
    </div>
    <div class="text-center">
        <button id="newpay-btn-auth" class="btn btn-info">اختبار المصادقة</button>
        <button id="newpay-btn-create" class="btn btn-primary">إنشاء معاملة</button>
        <button id="newpay-btn-confirm" class="btn btn-success">تأكيد الدفع</button>
        <button id="newpay-btn-status" class="btn btn-warning">التحقق من الحالة</button>
    </div>
    <div id="newpay-test-results" style="display:none; margin-top:15px;">
        <div class="alert" id="newpay-test-alert"></div>
        <pre id="newpay-test-details"></pre>
    </div>
</div>
<script>
    const newpayApi = '/api/v1/yepayment';
    document.getElementById('newpay-btn-auth')?.addEventListener('click', () => {
        fetch(`${newpayApi}/newpay/test-auth`, { method: 'POST' })
            .then(r => r.json())
            .then(d => showResult(d));
    });
    function showResult(data) {
        const div = document.getElementById('newpay-test-results');
        div.style.display = 'block';
        document.getElementById('newpay-test-details').textContent = JSON.stringify(data, null, 2);
    }
    // باقي الدوال بنفس النمط مع بادئة newpay-
</script>
```

---

## 🖥️ إنشاء صفحة الاختبار المتكاملة (UI)

**المسار:** `views/newpay-ui.htm`

- استخدم نفس هيكل `yottapay-ui.htm` أو `thawanipay-ui.htm`.
- قم بتغيير كل `yottapay` إلى `newpay` في المعرفات وأسماء الدوال و endpoints.
- تأكد من وجود علامات تبويب (يدوي، تلقائي، إحصائيات، سجلات).
- أضف سجلات محلية باستخدام `localStorage` بمفتاح فريد (`newpay_test_logs`).

**مقتطفات أساسية:**

- عرض الإعدادات الحالية: `{{ apiBaseUrl }}`, `{{ settings.url }}`, إلخ.
- نموذج إدخال البيانات (رقم الطلب، المبلغ، رمز الشراء، إلخ).
- أزرار: إنشاء معاملة، تأكيد، استعلام، اختبار شامل، اختبار تلقائي.
- عرض النتائج بتنسيق JSON.
- إحصائيات (عدد الطلبات، نسبة النجاح، آخر السجلات) عبر endpoint `/stats`.

---

## 🛣️ إضافة المسارات في `routes.php`

يجب إضافة المسارات داخل المجموعة الرئيسية المناسبة (`yepayment` أو `ompayment`). المسارات المطلوبة:

| الطريقة | المسار | الوصف |
|----------|--------|-------|
| `POST` | `/newpay/test-auth` | اختبار المصادقة |
| `POST` | `/newpay/test-create-payment` | إنشاء معاملة تجريبية |
| `POST` | `/newpay/test-confirm-payment` | تأكيد المعاملة (إن لزم) |
| `GET` | `/newpay/test-check-status` | الاستعلام عن حالة معاملة |
| `POST` | `/newpay/test-full-payment` | اختبار شامل |
| `GET` | `/newpay/stats` | إحصائيات البوابة |
| `GET` | `/newpay/test-ui` | عرض صفحة الاختبار |
| `GET` | `/newpay/success` | (لنوع redirect) مسار النجاح |
| `GET` | `/newpay/cancel` | (لنوع redirect) مسار الإلغاء |

**ملاحظات هامة:**

- جميع مسارات الاختبار محمية بـ `if(!BackendAuth::getUser())` لمنع الوصول غير المصرح به.
- استخدم `HttpHelper` داخل المسارات (إذا لزم الأمر) أو استدعِ دوال الكلاس مباشرة.
- في مسارات `success`/`cancel`، استخدم `RedirectHelper` لإعادة التوجيه إلى التطبيق (كما في `thawanipay/success`).

**مثال لمسار `success`:**
```php
Route::get('newpay/success', function () {
    $options = Input::get();
    $result = NewPay::checkAndCompletePay($options); // دالة عامة للتحقق
    $order_id = $result['data']['order_id'] ?? null;
    if ($order_id) {
        $callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, ['callback_success_url', 'newpay.callback_success_url']);
        if ($callbackUrl) {
            return RedirectHelper::redirectToApp($callbackUrl, $result, false, ['order_id', 'payment_method_id'], ['*'], false);
        }
    }
    return response()->json($result);
});
```

**مثال لمسار `test-create-payment`:**
```php
Route::post('newpay/test-create-payment', function (Request $request) {
    if (!BackendAuth::getUser()) return Response::make('Access Denied', 404);
    $order = Order::find($request->input('order_id'));
    if (!$order) return response()->json(['success' => false, 'message' => 'Order not found']);
    $newpay = new \Nano\Yepayment\PaymentTypes\NewPay($order, $request->all());
    $result = new PaymentResult($newpay, $order);
    $processResult = $newpay->process($result);
    return response()->json(['success' => $processResult->successful, 'data' => $processResult->api_data]);
});
```

---

## 📦 تسجيل البوابة في `Plugin.php`

```php
public function registerPaymentProviders()
{
    $providers = [];
    // ... البوابات الأخرى
    if (class_exists(\Nano\Yepayment\PaymentTypes\NewPay::class)) {
        $providers[] = new \Nano\Yepayment\PaymentTypes\NewPay();
    }
    return $providers;
}
```

ضعها في القسم الجغرافي المناسب (`allow_yemen_payment` أو `allow_oman_payment`).

---

## 🌐 إضافة مفاتيح الترجمة

**الملف:** `lang/ar/lang.php` و `lang/en/lang.php`

```php
return [
    'payment_gateway_settings' => [
        'newpay' => [
            'url' => 'رابط API الأساسي',
            'test_url' => 'رابط API للاختبار',
            'api_key' => 'مفتاح API',
            'secret_key' => 'المفتاح السري',
            'username' => 'اسم المستخدم',
            'password' => 'كلمة المرور',
            'default_currency' => 'العملة الافتراضية',
        ],
    ],
    'public' => [
        'newpay' => [
            'payment_success' => 'تم الدفع بنجاح',
            'confirmation_required' => 'يرجى إدخال رمز التأكيد',
            'transaction_created' => 'تم إنشاء المعاملة',
            'default_note' => 'تأكيد الدفع',
            'yes' => 'نعم',
            'no' => 'لا',
            'errors' => [
                'auth_failed' => 'فشلت المصادقة',
                'order_already_paid' => 'الطلب مدفوع مسبقاً',
                'missing_credentials' => 'بيانات الاعتماد غير مكتملة',
                'payment_creation_failed' => 'فشل إنشاء معاملة الدفع',
                'payment_confirmation_failed' => 'فشل تأكيد الدفع',
                'incomplete_payment_data' => 'بيانات الدفع غير مكتملة',
            ],
        ],
    ],
];
```

---

## 🧪 نصائح إضافية لتطوير بوابات احترافية

### 1. استخدام إلزامي لـ `HttpHelper` و `RedirectHelper`
- **جميع طلبات API** يجب أن تتم عبر `HttpHelper`، وليس عبر `curl` المباشر أو `Guzzle` بدون تغليف.
- **جميع عمليات إعادة التوجيه بعد الدفع** (خاصة للبوابات من نوع Redirect) يجب أن تستخدم `RedirectHelper` لضمان التوافق مع التطبيقات والويب.

### 2. دعم Webhooks
- أضف مساراً لاستقبال الإشعارات من البوابة (POST).
- تحقق من التوقيع (signature) إن وُجد.
- قم بتحديث حالة الطلب تلقائياً.

### 3. إعادة محاولة الطلبات الفاشلة (Retry)
- أضف حلقة `for` مع `try-catch` لتنفيذ الطلب عدة مرات قبل الفشل.

### 4. دعم الـ Idempotency
- أرسل `Idempotency-Key` فريد (مثل `order_id_uuid`) في طلبات إنشاء المعاملة.

### 5. دالة `checkTransactionStatus` عامة
- يمكن استدعاؤها من `stats` أو من واجهة المستخدم لعرض حالة المعاملة.

### 6. دالة `isAvailable()` للتحقق من صحة بيانات الإعدادات
```php
public function isAvailable(): bool
{
    try {
        $token = $this->getAuthToken();
        return !empty($token);
    } catch (\Exception $e) {
        return false;
    }
}
```

### 7. تسجيل أخطاء تفصيلي
- استخدم `trace_log()` أو `logger()` لتسجيل الطلبات والاستجابات عند حدوث أخطاء.

### 8. دالة `checkAndCompletePay` (لنوع redirect)
أضف دالة static في كلاس البوابة لتوحيد منطق التحقق وإتمام الدفع بعد العودة من البوابة الخارجية. يمكن الاستفادة من `RedirectHelper` داخلها.

### 9. التعامل مع جلسة الدفع وسجل العمليات (Payment Lifecycle)

- **تسجيل محاولة الدفع في `process`:** حتى لو لم يكتمل الدفع بعد (في النوع Two‑Step)، يجب استدعاء `$result->logSuccessfulPayment()` لحفظ السجل الأولي. هذا ما يفعله `YottaPay` و `BasPay`.
- **استخدام `complete` في `PaymentRedirector`:** عند العودة من بوابة خارجية (أو من تطبيق الجوال)، يستدعي `PaymentRedirector::handleOffSiteReturn` دالة `complete` تلقائياً إذا كانت الجلسة (session) تحتوي على callback. تأكد من أن `complete` جاهزة لهذا السيناريو.
- **تخزين `callback_success_url`:** في البوابات التي تحتاج إلى دعم الجوال، قم بتخزين الروابط في `order->other_data` كما في `BasPay` و `ThawaniPay`، واستخدم `RedirectHelper` لاستخراجها وإعادة التوجيه إليها بعد اكتمال الدفع.
- **معالجة `processed`:** متغير `processed` في الطلب يتحول إلى `true` فقط عندما يتم استدعاء `$result->success()` (الذي يعيّن الحالة إلى `PaidState`). في حالة استخدام `$result->pending()`، يكون `processed = true` لكن الحالة `PendingState` وليست `PaidState`. اعتمد على `isPaymentProcessed()` للتحقق من اكتمال الدفع الفعلي.

---

## ✅ قائمة مراجعة (Checklist) لإنشاء بوابة جديدة

- [ ] إنشاء كلاس `XxxPay.php` يمتد `PaymentProvider`.
- [ ] تنفيذ الدوال: `name`, `identifier`, `validate`, `settings`, `process`, `complete`.
- [ ] إضافة الدوال المساعدة: `getAuthToken`, `createPayment`, `confirmPayment`, `checkTransactionStatus`, `getApiUrl`, `parseResponse`.
- [ ] تعريف `defineValidationRules` و `getFieldNames`.
- [ ] إضافة خصائص `$success_url`, `$cancel_url`, `$is_test_mod`.
- [ ] إنشاء `_info.htm` و `_test_info.htm` (مع معرفات فريدة).
- [ ] إنشاء `views/xxxpay-ui.htm` (صفحة اختبار كاملة).
- [ ] إضافة مسارات الاختبار في `routes.php` (مع حماية `BackendAuth`).
- [ ] إضافة مسارات `success`/`cancel` (لنوع redirect) باستخدام `RedirectHelper`.
- [ ] تسجيل الكلاس في `Plugin.php` تحت القسم المناسب.
- [ ] إضافة مفاتيح الترجمة في `lang/ar/lang.php` و `lang/en/lang.php`.
- [ ] اختبار البوابة عبر `/api/v1/yepayment/xxxpay/test-ui`.
- [ ] توثيق أي سلوك خاص (Webhooks, إلخ) في `_info.htm` أو ملف منفصل.
- [ ] **تأكد من أن `process()` في النوع Two‑Step لا تستدعي `$result->success()`**، بل تعيد نجاحاً مع رسالة تأكيد.
- [ ] **تأكد من أن `complete()` تستدعي `$result->success()` فقط بعد التحقق من نجاح العملية**.
- [ ] **وفر دالة `static checkAndCompletePay`** لاستخدامها في مسار `success` (خاصة لبوابات Two‑Step و Redirect).
- [ ] **تأكد من تخزين روابط العودة** (`callback_success_url` و `callback_error_url`) في `other_data` إذا تم تمريرها.
- [ ] **اختبر أن `OrderManager` يُبقي `processed = false`** بعد `process()` مباشرة في البوابات Two‑Step.
- [ ] **تحقق من أن `PaymentResult::success` يؤدي إلى `PaidState`** وتفعيل الأحداث وتفريغ السلة.

---

## 📚 أمثلة مرجعية (من الكود المرفق)

- **YottaPay**: Two‑Step (OTP). `process()` تعيد `successful=true` مع `requires_confirmation`، و `complete()` تستخدم OTP المرسل. يُظهر استخدام `HttpHelper` في طلبات JSON و PATCH، وتخزين التوكن.
- **BasPay**: Two‑Step (تأكيد عبر تطبيق منفصل). `process()` تنشئ معاملة وتُعيد `trxToken`، و `complete()` تتحقق من حالة المعاملة وتُحدّث الطلب إلى `PaidState` فقط عند النجاح. يُظهر استخدام `HttpHelper::sendForm` للحصول على توكن OAuth، ودالة `checkAndCompletePay` الثابتة للتأكيد عبر مسار `success`.
- **QasemiPay**: Direct مع تأكيد اختياري (concurrencyStamp). `process()` و `complete()` يتبعان نمطاً مشابهاً، ويُظهر استخدام `HttpHelper::sendForm` للحصول على توكن OAuth.
- **ThawaniPay**: Redirect مع إعادة توجيه إلى بوابة خارجية. `process()` تعيد توجيه، و `complete()` تُستدعى بعد العودة. يُظهر استخدام `RedirectHelper` في مسارات `success`/`cancel`، ودالة `checkAndCompletePay` للتأكيد.

---

## 🏁 الخلاصة

باتباع هذا الدليل يمكنك إنشاء أي طريقة دفع جديدة في نظام `Nano.Yepayment` بغض النظر عن تعقيد API الخاص بها. استخدم النمط المناسب (Redirect, Two‑Step, Direct) وفقاً لتدفق الدفع، ولا تنسَ:

- استخدام `HttpHelper` لجميع طلبات API.
- استخدام `RedirectHelper` لإعادة التوجيه بعد الدفع (حتى في البوابات غير المباشرة التي تدعم الجوال).
- فهم تأثير دوال `PaymentResult` (`success`, `pending`, `fail`) على حالة الطلب وتفريغ السلة والأحداث.
- توفير دالة `static checkAndCompletePay` لتسهيل التأكيد من مسارات العودة.
- تخزين روابط العودة (`callback_success_url`) في `other_data`.
- تذكر أن `OrderManager` و `Checkout2` يعتمدان على حالة `isPaymentProcessed()` لتحديد الخطوة التالية، فلا تستعجل في استدعاء `$result->success()`.
- توفير أدوات الاختبار اللازمة (`_test_info.htm` و `xxxpay-ui.htm`) لتسهيل التطوير والصيانة.

🚀 **ابدأ الآن في إضافة بوابتك الجديدة!**
