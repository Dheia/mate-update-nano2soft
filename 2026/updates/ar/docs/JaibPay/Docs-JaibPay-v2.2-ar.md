# توثيق بوابة الدفع JaibPay (محفظة جيب) – دليل المطور

## 1. نظرة عامة

**JaibPay** هي بوابة دفع إلكترونية مدمجة ضمن حزمة `Nano.Yepayment` في تطبيقات نانوسوفت. تعتمد البوابة على واجهة برمجة تطبيقات **Jaib Wallet API** وتستخدم آلية **الدفع الفوري (Direct Payment)** من خطوة واحدة:

1. **تنفيذ عملية شراء عبر الكود** (Execute Buy Online By Code) – يقوم العميل بإدخال كود الشراء الذي أنشأه مسبقاً في تطبيق جيب، ويتم خصم المبلغ فوراً من رصيده دون حاجة إلى تأكيد إضافي أو رمز OTP.

بعد التحديثات الأخيرة، أصبحت البوابة تدعم أيضاً:
- **استرداد المبلغ (Refund)** – عبر `POST /api/v1/BuyOnline/RefoundBuy`
- **اختبار المنفذ** – دالة `testPortConnection` لتشخيص مشاكل الاتصال
- **معالجة موحدة للاستجابات** – دالة `normalizeApiResponse` لتقديم رسائل خطأ واضحة
- **جلب آمن للإعدادات المشفرة** – دالة `getSettingsByKey` لجلب كلمة المرور المخزنة بشكل مشفر

هذا الكلاس يمكن استخدامه مباشرة من خلال `Nano\Yepayment\PaymentTypes\JaibPay` أو عبر نظام المدفوعات الموحد `Nano\MicroCart\Classes\Payments\PaymentGateway`.

---

## 2. متطلبات التشغيل والإعدادات

### 2.1. المتطلبات الأساسية

- نظام نانوسوفت (Nano2Soft) الإصدار 2.0+
- إضافات مطلوبة:
  - `Nano.MicroCart` (>=2.0)
  - `Nano.Yepayment` (>=1.2)
  - `Nano.Helpers`
- بيانات الدخول إلى Jaib API (مقدمة من مشغل البوابة):
  - اسم المستخدم (`userName`)
  - كلمة المرور (`password`)
  - كود الوكيل (`agentCode` – القيمة الافتراضية `10004`)
  - رابط API الأساسي (مثال: `https://www.api2.e-jaib.com:5088`)

### 2.2. إعدادات البوابة في لوحة التحكم

عند تفعيل طريقة الدفع **"Jaib Pay (محفظة جيب)"**، تظهر الحقول التالية (تخزن في جدول `nano_microcart_payment_gateway_settings`):

| الحقل | المفتاح | الوصف |
|-------|---------|-------|
| رابط API الأساسي | `jaibpay_url` | عنوان API الخاص بـ Jaib (مثال: `https://www.api2.e-jaib.com:5088`) |
| رابط API للاختبار | `jaibpay_test_url` | (اختياري) عنوان بيئة الاختبار |
| اسم المستخدم | `jaibpay_username` | اسم مستخدم التاجر |
| كلمة المرور | `jaibpay_password` | كلمة مرور التاجر (تخزن مشفرة) |
| كود الوكيل (agentCode) | `jaibpay_agentcode` | رمز الوكيل المقدم من بوابة جيب (افتراضي `10004`) |
| العملة الافتراضية | `jaibpay_default_currency` | القيمة الافتراضية (YER, USD, SAR) |

يمكن الوصول إلى هذه الإعدادات داخل الكلاس عبر:
```php
$url = PaymentGatewaySettings::get('jaibpay_url', '');
$username = PaymentGatewaySettings::get('jaibpay_username', '');
$agentCode = PaymentGatewaySettings::get('jaibpay_agentcode', '10004');
```

> **هام:** بالنسبة للإعدادات المشفرة مثل `jaibpay_password`، يجب استخدام دالة `getSettingsByKey()` بدلاً من `PaymentGatewaySettings::get()` المباشر لضمان جلب القيمة المفكوكة بشكل صحيح:
> ```php
> $password = $this->getSettingsByKey('jaibpay_password');
> ```

---

## 3. كلاس JaibPay – الطرق الأساسية

### 3.1. تعريف الكلاس

```php
namespace Nano\Yepayment\PaymentTypes;

use Nano\MicroCart\Classes\Payments\PaymentProvider;
use Nano\MicroCart\Classes\Payments\PaymentResult;
use Nano\MicroCart\Models\PaymentGatewaySettings;

class JaibPay extends PaymentProvider
{
    // ...
}
```

### 3.2. الخصائص الأساسية

| الخاصية | النوع | الوصف |
|----------|------|-------|
| `$order` | `Order` | كائن الطلب المرتبط بالدفع |
| `$data` | `array` | البيانات الواردة من المستخدم (كود الشراء، الجوال، المبلغ، إلخ) |
| `$success_url` | `string` | (غير مستخدم في JaibPay لأن الدفع مباشر بدون redirect) |
| `$cancel_url` | `string` | (غير مستخدم) |

### 3.3. الطرق الرئيسية

#### `public function identifier(): string`
يعيد معرف فريد لطريقة الدفع (`jaibpay`).

#### `public function name(): string`
يعيد الاسم المعروض (`Jaib Pay (محفظة جيب)`).

#### `public function process(PaymentResult $result): PaymentResult`
ينشئ دفعة جديدة (تنفيذ الدفع المباشر) عبر API.

**المدخلات المتوقعة في `$this->data` (من نموذج الدفع):**
- `purchase_code` – كود الشراء (مطلوب)
- `mobile` – رقم جوال العميل (مطلوب)
- `amount` – المبلغ (مطلوب)
- `currency` – العملة (اختياري، يستخدم الافتراضي إن لم يوجد)
- `notes` – ملاحظة (اختياري)

**الإجراءات:**
1. التحقق من صحة البيانات عبر `defineValidationRules()`.
2. الحصول على `accessToken` و `pinApi` عبر `getAuthToken()` (من Cache أو API).
3. توليد `requestID` فريد (UUID).
4. إرسال طلب POST إلى `/api/v1/BuyOnline/ExeBuy` مع البيانات.
5. إذا نجحت العملية، حفظ `requestID` في `order->payment_first_trans_id` و `referenceID` في `order->payment_trans_id`.
6. حفظ بيانات إضافية في `order->other_data['jaibpay']`.
7. تحديث حالة الطلب إلى `PaidState` عبر `$result->success()`.
8. إرجاع `PaymentResult` بنجاح.
9. **في حال الفشل:** يتم تعيين `$result->message` من رسالة الخطأ القادمة من Jaib (مثل "كود الشراء غير صحيح")، ثم استدعاء `$result->fail()`.

#### `public function complete(PaymentResult $result): PaymentResult`
غير مستخدم في هذا النوع (الدفع الفوري). يترك فارغاً أو يمكن استخدامه لاسترداد الأموال مستقبلاً.

#### `public function getAuthToken(): ?array`
يطلب `accessToken` و `pinApi` من Jaib API.

**نقطة النهاية:** `POST /api/v1/TokenAuth/LogAPI`  
**البيانات:** `{"userName": "...", "password": "...", "agentCode": "..."}`  
**الإرجاع:** مصفوفة تحتوي على `accessToken`, `pinApi`, `expire` أو `null` في حال الفشل.  
**التخزين المؤقت:** يتم تخزينها في Cache لمدة 86000 ثانية.  
**تحسينات:** تستخدم `normalizeApiResponse` لتقديم رسائل خطأ واضحة، وتستخدم `getSettingsByKey` لجلب كلمة المرور المشفرة.

#### `private function executeBuy(string $accessToken, string $pinApi, string $requestID): array`
ينفذ عملية الشراء عبر الكود.

**نقطة النهاية:** `POST /api/v1/BuyOnline/ExeBuy`  
**البيانات:** `{"pinApi": "...", "mobile": "...", "requestID": "...", "code": "...", "amount": ..., "currencyCode": "...", "notes": "..."}`  
**الإرجاع:** مصفوفة تحتوي على `success`, `referenceID`, `requestID`, `msg`, `amount`, `currencyCode`.  
**تحسينات:** تستخدم `normalizeApiResponse` لتقديم رسائل خطأ دقيقة مثل "كود الشراء غير صحيح" أو "قد تم استخدام الكود مسبقاً".

#### `public function checkTransactionStatus(string $requestID): array`
يستعلم عن حالة معاملة باستخدام `requestID`.

**نقطة النهاية:** `POST /api/v1/BuyOnline/CheckProgress`  
**البيانات:** `{"pinApi": "...", "requestID": "..."}`  
**الإرجاع:** مصفوفة تحتوي على `success`, `request_id`, `reference_id`, `raw_response`.

#### `public function refund(string $referenceID, ?string $requestID = null, ?float $amount = null, ?string $currencyCode = null, ?string $notes = null): array`
**🆕 دالة جديدة** – تسترد مبلغ معاملة سابقة.

**نقطة النهاية:** `POST /api/v1/BuyOnline/RefoundBuy`  
**المعاملات:**
- `$referenceID` – رقم العملية المرجعي الأساسي (مطلوب)
- `$requestID` – معرف طلب فريد جديد (اختياري، يولّد تلقائياً)
- `$amount` – المبلغ المراد استرداده (اختياري، يُجلب من الطلب)
- `$currencyCode` – رمز العملة (اختياري)
- `$notes` – ملاحظات (اختياري)

**تأثير الدالة على الطلب:**
- عند النجاح: يتم تغيير `order->payment_state` إلى `RefundedState`.
- يتم تخزين بيانات الاسترداد في `order->other_data['jaibpay_refund']` كمصفوفة تحتوي على:
  ```php
  [
      'refund_referenceID'   => 'رقم مرجع الاسترداد من Jaib',
      'original_referenceID' => 'رقم العملية الأصلية',
      'requestID'            => 'معرف الطلب الفريد',
      'amount'               => 5000.0,
      'currencyCode'         => 'YER',
      'notes'                => 'ملاحظات الاسترداد',
      'msg'                  => 'تمت العملية بنجاح',
      'created_at'           => '2025-01-01 12:00:00'
  ];
  ```

#### `private function getApiUrl(string $type): string`
يبني روابط API بناءً على النوع (`login`, `buy`, `refund`, `check`).

#### `private function parseResponse($response): array`
يحول استجابة Guzzle إلى مصفوفة PHP.

---

## 4. الدوال المساعدة الجديدة

### 4.1. `normalizeApiResponse` – معالجة موحدة لاستجابات API

```php
private function normalizeApiResponse($response): array
```

**الوظيفة:** تستقبل استجابة HTTP (أو استثناء) وتستخرج بشكل موحد جميع المعلومات المهمة:

| المفتاح | الوصف |
|---------|-------|
| `success` | نجاح العملية (boolean) |
| `status_code` | رمز HTTP (مثل 200، 500) |
| `error_code` | كود الخطأ من Jaib (مثل `51`، `-100`، `-1026`) |
| `error_message` | رسالة الخطأ النصية من Jaib |
| `data` | بيانات النتيجة (المحتوى الكامل لـ `result`) |
| `raw` | الاستجابة الخام كاملة |

**الاستخدام:** تُستخدم في جميع دوال API (`getAuthToken`, `executeBuy`, `checkTransactionStatus`, `refund`) لتوحيد معالجة الردود وتقديم رسائل خطأ دقيقة للمستخدم.

### 4.2. `looksLikeJson` – التحقق من شكل JSON

```php
private function looksLikeJson($string): bool
```

دالة مساعدة تتحقق مما إذا كان النص المُعطى يشبه JSON (يبدأ بـ `{` أو `[`). تُستخدم داخلياً في `normalizeApiResponse`.

### 4.3. `testPortConnection` – اختبار المنفذ

```php
public static function testPortConnection(string $url, ?int $port = null, int $timeout = 5): array
```

**الوظيفة:** تختبر ما إذا كان منفذ معين مفتوحاً على خادم. تدعم:
- استخراج المنفذ تلقائياً من الرابط (مثل `https://www.api2.e-jaib.com:5088`)
- أولوية المنفذ الممرر Parameter على المنفذ الموجود في الرابط
- استخدام المنفذ الافتراضي حسب scheme (443 لـ HTTPS، 80 لـ HTTP)
- فحص DNS وتحديد ما إذا كانت المشكلة في DNS أم في المنفذ
- قياس زمن الاستجابة بالمللي ثانية

**مثال استدعاء:**
```php
$result = JaibPay::testPortConnection('https://www.api2.e-jaib.com:5088');
if ($result['success']) {
    echo "المنفذ مفتوح، زمن الاستجابة: " . $result['details']['elapsed_ms'] . "ms";
} else {
    echo "فشل الاتصال: " . $result['message'];
}
```

### 4.4. `getSettingsByKey` – جلب آمن للإعدادات

```php
public function getSettingsByKey($key, $defaultValue = null)
```

**الوظيفة:** تجلب قيمة إعداد من `PaymentGatewaySettings` بطريقة صحيحة، مع دعم الحقول المشفرة:
- إذا كان المفتاح ضمن `encryptedSettings()`، تستخدم `getEncryptableValue()` لفك التشفير.
- وإلا تستخدم `get()` العادية.

**الاستخدام الأساسي:** جلب كلمة المرور المخزنة بشكل مشفر (`jaibpay_password`).

---

## 5. آلية الدفع خطوة بخطوة (للمطور)

### 5.1. تدفق العملية الكامل

1. **المصادقة:** `getAuthToken()` → `POST /api/v1/TokenAuth/LogAPI` ← `{ accessToken, pinApi, expire }`
2. **تنفيذ الدفع:** `executeBuy()` → `POST /api/v1/BuyOnline/ExeBuy` ← `{ referenceID, requestID, msg }`
3. **حفظ البيانات:** تخزين `requestID` في `order->payment_first_trans_id` و `referenceID` في `order->payment_trans_id` وتحديث الطلب إلى `PaidState`
4. **الاستعلام (اختياري):** `checkTransactionStatus()` → `POST /api/v1/BuyOnline/CheckProgress`
5. **استرداد المبلغ (جديد):** `refund()` → `POST /api/v1/BuyOnline/RefoundBuy` ← تغيير حالة الطلب إلى `RefundedState`


### 5.2. مخطط تدفق العملية الكامل

<img src="./images/sequenceDiagram-JaibPay-ar.jpg" alt="JaibPay Flow Diagram" width="600">


### 5.3. دمج البوابة في واجهة برمجة تطبيقات (API) مخصصة

#### أ. تنفيذ دفعة جديدة (بدء الدفع)

**نقطة نهاية مخصصة في `routes/api.php`:**

```php
Route::post('/payment/jaibpay/create', function (Request $request) {
    $order = Order::find($request->order_id);
    $jaib = new JaibPay($order, [
        'purchase_code' => $request->purchase_code,
        'mobile'        => $request->mobile,
        'amount'        => $request->amount,
        'currency'      => $request->currency ?? 'YER',
        'notes'         => $request->notes,
    ]);
    $result = new PaymentResult($jaib, $order);
    $processResult = $jaib->process($result);
    return response()->json([
        'success'      => $processResult->successful,
        'request_id'   => $order->payment_first_trans_id,
        'reference_id' => $order->payment_trans_id,
        'message'      => $processResult->message,
    ]);
});
```

**طلب مثال:**
```json
POST /api/payment/jaibpay/create
{
    "order_id": 200,
    "purchase_code": "3719",
    "mobile": "774760761",
    "amount": 5000,
    "currency": "YER",
    "notes": "دفع الطلب #200"
}
```

**استجابة:**
```json
{
    "success": true,
    "request_id": "550e8400-e29b-41d4-a716-446655440000",
    "reference_id": "16986110064345",
    "message": "تمت عملية الدفع بنجاح"
}
```

#### ب. الاستعلام عن حالة المعاملة

```php
Route::get('/payment/jaibpay/status', function (Request $request) {
    $jaib = new JaibPay();
    $status = $jaib->checkTransactionStatus($request->request_id);
    return response()->json($status);
});
```

**طلب:**
```
GET /api/payment/jaibpay/status?request_id=550e8400-e29b-41d4-a716-446655440000
```

**استجابة:**
```json
{
    "success": true,
    "request_id": "550e8400-e29b-41d4-a716-446655440000",
    "reference_id": "16986110064345",
    "raw_response": { ... }
}
```

#### ج. استرداد المبلغ (جديد)

```php
Route::post('/payment/jaibpay/refund', function (Request $request) {
    $order = Order::find($request->order_id);
    $jaib = new JaibPay($order, [
        'amount'   => $request->amount,
        'currency' => $request->currency ?? 'YER',
        'notes'    => $request->notes ?? 'استرداد مبلغ',
    ]);
    $refundResult = $jaib->refund($request->reference_id);
    return response()->json($refundResult);
});
```

---

## 6. نقاط نهاية الاختبار المضمنة في `routes.php`

ضمن ملف `routes.php` الخاص بـ `Nano.Yepayment`، تم توفير مجموعة من نقاط النهاية المساعدة تحت المجموعة `/api/v1/yepayment`، والمخصصة للمطورين والمسؤولين لاختبار البوابة.

### 6.1. قائمة نقاط النهاية

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/jaibpay/test-auth` | POST | اختبار المصادقة مع Jaib API (الحصول على accessToken و pinApi) |
| `/jaibpay/test-create-payment` | POST | إنشاء دفعة جديدة (تنفيذ الدفع المباشر) |
| `/jaibpay/test-check-status` | GET | الاستعلام عن حالة معاملة باستخدام `request_id` |
| `/jaibpay/test-full-payment` | POST | اختبار شامل (إنشاء دفعة + استعلام) |
| `/jaibpay/test-refund` | POST | **🆕** استرداد مبلغ معاملة |
| `/jaibpay/test-port` | GET | **🆕** اختبار المنفذ (يستخدم `JaibPay::testPortConnection`) |
| `/jaibpay/stats` | GET | إحصائيات استخدام البوابة (عدد الطلبات، نسبة النجاح) |
| `/jaibpay/test-ui` | GET | واجهة ويب تفاعلية لاختبار جميع الوظائف |

### 6.2. شرح نقاط النهاية الجديدة

#### `POST /jaibpay/test-refund`
**بيانات الطلب (JSON):**
```json
{
    "order_id": 200,
    "reference_id": "16986110064345",
    "amount": 5000,
    "currency": "YER",
    "notes": "استرداد"
}
```
> إذا لم يتم تمرير `reference_id` وتم تمرير `order_id`، يحاول النظام جلبه تلقائياً من بيانات الطلب. كما يتحقق من أن الطلب مدفوع وأنه تم عبر JaibPay.  
**الاستجابة:** `{ success, data: { reference_id, request_id, api_data } }`

#### `GET /jaibpay/test-port?url=...&port=...`
**الاستجابة:** `{ success, message, details: { host, port, elapsed_ms, dns_resolved, ... } }`

### 6.3. شرح كل نقطة نهاية

#### `POST /jaibpay/test-auth`
لا تحتاج إلى بيانات إدخال (تستخدم الإعدادات المخزنة).  
**الاستجابة:** `{ success, data: { access_token_length, pin_api, expire } }`

#### `POST /jaibpay/test-create-payment`
**بيانات الطلب (JSON):**
```json
{
    "order_id": 200,
    "purchase_code": "3719",
    "mobile": "774760761",
    "amount": 5000,
    "currency": "YER",
    "notes": "اختبار"
}
```
**الاستجابة:** `{ success, data: { request_id, reference_id, api_data }, order_data }`

#### `GET /jaibpay/test-check-status?request_id=...`
**الاستجابة:** `{ success, request_id, reference_id, raw_response }`

#### `POST /jaibpay/test-full-payment`
يقوم بتنفيذ خطوتين تلقائياً (إنشاء دفعة + استعلام).  
**الاستجابة:** تحتوي على `results` (نتائج كل خطوة) و `summary`.

#### `GET /jaibpay/test-ui`
يعرض واجهة HTML متكاملة تحتوي على:
- اختبار يدوي خطوة بخطوة (مصادقة، إنشاء دفعة، استعلام، استرداد)
- اختبار تلقائي بعدد مرات قابل للتحديد (1-10 مرات)
- إحصائيات فورية (عدد الطلبات، نسبة النجاح، آخر السجلات)
- سجلات الاختبارات المخزنة في LocalStorage
- أدوات إضافية (اختبار المنفذ، تصدير السجلات، إعادة التعيين)

#### `GET /jaibpay/stats`
**الاستجابة:** إحصائيات مثل `total_orders`, `jaibpay_orders`, `successful_payments`, `success_rate`, إعدادات البوابة.

---

## 7. التعامل مع البوابة عبر API خارجي (للتطبيقات الأخرى)

إذا كنت تطور تطبيقاً خارجياً (مثلاً تطبيق جوال أو متجر إلكتروني مستقل) وترغب في دمج JaibPay دون استخدام كلاس `JaibPay` مباشرة، يمكنك الاتصال بـ **نقاط النهاية العامة** التي يوفرها النظام (المذكورة أعلاه) بعد المصادقة عبر `oauth-users`.

### 7.1. المصادقة المسبقة

يجب أن يكون لديك توكن OAuth 2.0 صالح (يمكن الحصول عليه من نظام نانوسوفت عبر نقطة نهاية تسجيل الدخول المعتادة). ثم ترسل التوكن في الهيدر:
```
Authorization: Bearer <token>
```

### 7.2. مثال متكامل باستخدام cURL

#### أ. إنشاء دفعة جديدة
```bash
curl -X POST "https://yourdomain.com/api/v1/yepayment/jaibpay/test-create-payment" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 200,
    "purchase_code": "3719",
    "mobile": "774760761",
    "amount": 5000,
    "currency": "YER",
    "notes": "شراء منتج"
  }'
```

#### ب. الاستعلام عن حالة الدفعة
```bash
curl -X GET "https://yourdomain.com/api/v1/yepayment/jaibpay/test-check-status?request_id=550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer <token>"
```

#### ج. استرداد المبلغ
```bash
curl -X POST "https://yourdomain.com/api/v1/yepayment/jaibpay/test-refund" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 200,
    "reference_id": "16986110064345",
    "amount": 5000,
    "currency": "YER"
  }'
```

#### د. اختبار المنفذ
```bash
curl -X GET "https://yourdomain.com/api/v1/yepayment/jaibpay/test-port?url=https://www.api2.e-jaib.com:5088" \
  -H "Authorization: Bearer <token>"
```

> **ملاحظة:** نقاط النهاية هذه محمية بـ `BackendAuth` أيضاً (تتطلب أن يكون المستخدم مسؤولاً). إذا كنت تريد توفيرها للعملاء العاديين، يجب تعديل `routes.php` لإزالة فحص `BackendAuth` أو إضافة middleware مخصص.

---

## 8. رموز الأخطاء الشائعة والحلول

| كود HTTP | الخطأ (من Jaib API) | السبب والحل |
|----------|----------------------|--------------|
| 401 | Unauthorized | فشل المصادقة – تحقق من `userName`/`password`/`agentCode` في الإعدادات. |
| 400 | "رقم الكود غير صحيح" (code 51) | كود الشراء غير صحيح أو منتهي الصلاحية – تأكد من الكود المدخل. |
| 400 | "قد تم استخدام الكود مسبقاً" (code -1026) | تم استهلاك الكود مسبقاً – استخدم كود جديد. |
| 400 | "رصيد غير كاف" | رصيد العميل في محفظة جيب لا يكفي للمبلغ المطلوب. |
| 400 | "المعاملة غير موجودة" | `requestID` غير صحيح – تحقق من المعرف المخزن. |
| 400 | "رسم مرجع العملية غير صحيح" | `referenceID` غير صحيح عند الاسترداد. |
| 500 | Internal Server Error | مشكلة في الاتصال بـ Jaib API – تأكد من الرابط والمنفذ، أو حاول لاحقاً. |
| cURL 28 | Connection timed out | المنفذ 5088 غير مفتوح من جهة الاستضافة – استخدم `testPortConnection` للتشخيص. |

---

## 9. هيكل البيانات المخزنة في الطلب

### 9.1. بيانات الدفع – `order->other_data['jaibpay']`

```php
[
    'request_id'      => '550e8400-e29b-41d4-a716-446655440000',  // معرف الطلب الفريد
    'reference_id'    => '16986110064345',      // رقم العملية المرجعي من Jaib
    'pin_api'         => 'tIcI4zm',             // رمز PIN API المستخدم
    'amount'          => 5000.0,                // المبلغ
    'currency_code'   => 'YER',                 // رمز العملة
    'mobile'          => '774760761',           // رقم جوال العميل
    'purchase_code'   => '3719',                // كود الشراء المستخدم
    'created_at'      => '2025-01-01T12:00:00Z' // وقت إنشاء المعاملة
]
```

### 9.2. بيانات الاسترداد – `order->other_data['jaibpay_refund']`

```php
[
    'refund_referenceID'   => '16986138584353',  // رقم مرجع الاسترداد من Jaib
    'original_referenceID' => '16986110064345',  // رقم العملية الأصلية
    'requestID'            => 'uuid-...',        // معرف طلب الاسترداد
    'amount'               => 5000.0,            // المبلغ المسترد
    'currencyCode'         => 'YER',             // رمز العملة
    'notes'                => 'استرداد مبلغ',    // ملاحظات
    'msg'                  => 'تمت العملية بنجاح', // رسالة من Jaib
    'created_at'           => '2025-01-01T12:30:00Z' // وقت الاسترداد
]
```

---

## 10. أمثلة عملية لاستخدام الكلاس في كود مخصص

### 10.1. إنشاء دفعة جديدة بدون استخدام `PaymentGateway`

```php
use Nano\Yepayment\PaymentTypes\JaibPay;
use Nano\Orders\Models\Order;

$order = Order::find(200);
$jaib = new JaibPay($order, [
    'purchase_code' => '3719',
    'mobile'        => '774760761',
    'amount'        => 5000,
    'currency'      => 'YER',
    'notes'         => 'دفع الطلب #200',
]);

$paymentResult = new \Nano\MicroCart\Classes\Payments\PaymentResult($jaib, $order);
$processResult = $jaib->process($paymentResult);

if ($processResult->successful) {
    $requestID = $order->payment_first_trans_id;
    $referenceID = $order->payment_trans_id;
    // توجيه المستخدم إلى صفحة النجاح
}
```

### 10.2. الاستعلام عن حالة دفعة

```php
$jaib = new JaibPay();
$status = $jaib->checkTransactionStatus('550e8400-e29b-41d4-a716-446655440000');
if ($status['success']) {
    echo "Reference ID: " . $status['reference_id'];
}
```

### 10.3. استرداد مبلغ معاملة

```php
$order = Order::find(200);
$jaib = new JaibPay($order, [
    'amount'   => 5000,
    'currency' => 'YER',
    'notes'    => 'استرداد كامل المبلغ',
]);
$refundResult = $jaib->refund('16986110064345');

if ($refundResult['success']) {
    // تم الاسترداد بنجاح
    // حالة الطلب أصبحت RefundedState تلقائياً
    // بيانات الاسترداد مخزنة في $order->other_data['jaibpay_refund']
}
```

### 10.4. استخدام دالة المصادقة للحصول على التوكن فقط

```php
$jaib = new JaibPay();
$auth = $jaib->getAuthToken(); // يعيد مصفوفة تحتوي على accessToken, pinApi
if ($auth) {
    echo "Token: " . $auth['accessToken'];
}
```

### 10.5. اختبار المنفذ

```php
$result = JaibPay::testPortConnection('https://www.api2.e-jaib.com:5088');
if ($result['success']) {
    echo "المنفذ مفتوح، زمن الاستجابة: " . $result['details']['elapsed_ms'] . "ms";
} else {
    echo "فشل الاتصال: " . $result['message'];
    if ($result['details']['dns_resolved']) {
        echo " (تم تحليل DNS بنجاح – المشكلة في المنفذ نفسه)";
    }
}
```

---

## 11. ملخص نقاط النهاية في `routes.php` (مرجع سريع)

| المسار الكامل | الطريقة | الاستخدام |
|---------------|---------|-----------|
| `/api/v1/yepayment/jaibpay/test-auth` | POST | اختبار بيانات الدخول |
| `/api/v1/yepayment/jaibpay/test-create-payment` | POST | إنشاء دفعة جديدة |
| `/api/v1/yepayment/jaibpay/test-check-status` | GET | الاستعلام عن حالة معاملة |
| `/api/v1/yepayment/jaibpay/test-full-payment` | POST | اختبار شامل (إنشاء + استعلام) |
| `/api/v1/yepayment/jaibpay/test-refund` | POST | 🆕 استرداد مبلغ معاملة |
| `/api/v1/yepayment/jaibpay/test-port` | GET | 🆕 اختبار المنفذ |
| `/api/v1/yepayment/jaibpay/stats` | GET | إحصائيات البوابة |
| `/api/v1/yepayment/jaibpay/test-ui` | GET | واجهة اختبار ويب |

> **ملاحظة:** جميع هذه النقاط تتطلب أن يكون المستخدم الحالي مديراً (`BackendAuth`). لإتاحتها لعملاء API عاديين، قم بتعديل `routes.php` أو أضف middleware مخصص.

---

## 12. المراجع

- [كلاس JaibPay.php](./JaibPay.php) – الكود الكامل للبوابة.
- [ملف routes.php](./routes.php) – تعريف نقاط النهاية الخاصة بـ JaibPay.
- [وثيقة تحديثات JaibPay (مايو 2026)](./Update-JaibPay-ar.md) – سجل التحديثات والتحسينات.
- [دليل تطوير بوابات الدفع – نانوسوفت](./SKILL.md)
- [وثيقة API الخاصة بـ Jaib Pay – تسجيل الدخول (Login.pdf)](./Login.pdf)
- [وثيقة API الخاصة بـ Jaib Pay – تنفيذ واستعلام الدفع (Jaib Wallet Pay API.pdf)](./Jaib%20Wallet%20Pay%20API.pdf)
- [مجموعة Postman لاختبار API Jaib Pay](./Jaib%20Pay%20API.postman_collection.json)

---

**تم إعداد هذا التوثيق لمساعدة المطورين على دمج واستخدام بوابة JaibPay (محفظة جيب) بسهولة وفعالية.**  
للاستفسارات أو الدعم الفني، يرجى التواصل عبر الموقع الرسمي [nano2soft.com](https://nano2soft.com).