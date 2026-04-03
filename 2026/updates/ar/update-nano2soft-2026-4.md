# Update 2026-4

**تحديثات شهر اربعة أبريل **

## 2026-3-31 - 2026-4-1

### إضافة دعم القيم الافتراضية للدولة والمدينة والمديرية في إضافة العناوين عبر API (الإصدار الثاني)

**تم تزويد وحدة `Nano.LocationApi` بخيارات جديدة تسمح باستخدام القيم الافتراضية للدولة (Country)، والمدينة (State)، والمديرية (Directorate) عند إنشاء عنوان جديد عبر واجهة البرمجة (API) في الإصدار الثاني (`api_version=2` أو `v2`).**

سابقاً، كان على المطور تمرير معرفات (IDs) هذه الكيانات بشكل إلزامي أثناء إضافة العنوان، وإلا كان الطلب يفشل. بعد هذا التحديث، يمكن تفعيل إمكانية تعيين القيم الافتراضية تلقائياً في حال عدم توفيرها، مما يسهل تكامل التطبيقات مع النظام ويقلل من الأخطاء المرتبطة بالبيانات المكانية.

تمت إضافة ثلاث خيارات جديدة في ملف الإعدادات `config.php`، وتم تعديل دالة `addV2` في المتحكم `Address` لتفعيل هذا السلوك وفقاً للإعدادات.

---

### 1. المكونات المطورة

| السلوك | المكون | الوصف |
|--------|--------|-------|
| إضافة العنوان (الإصدار الثاني) | `Address@addV2` | تم إضافة منطق لاستخدام القيم الافتراضية عند عدم تمرير الدولة/المدينة/المديرية. |
| الإعدادات | `config.php` (قسم `address`) | إضافة مفاتيح جديدة للتحكم في تفعيل القيم الافتراضية. |

#### الإعدادات الجديدة

| المفتاح | متغير البيئة | الوصف |
|---------|--------------|-------|
| `is_allow_default_country_in_add` | `NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD` | إذا كان `true`، سيتم تعيين الدولة الافتراضية (من `RainLab\Location\Models\Country::getDefault()`) عند عدم تمرير `country_id`. |
| `is_allow_default_state_in_add` | `NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_STATE_IN_ADD` | إذا كان `true`، سيتم تعيين المدينة الافتراضية (من `RainLab\Location\Models\State::getDefault()` أو أول مدينة تابعة للدولة المحددة) عند عدم تمرير `state_id`. |
| `is_allow_default_directorate_in_add` | `NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_DIRECTORATE_IN_ADD` | إذا كان `true`، سيتم تعيين المديرية الافتراضية (من `RainLab\Location\Models\Directorate::getDefault()` أو أول مديرية تابعة للمدينة المحددة) عند عدم تمرير `directorate_id`. |

جميع الإعدادات افتراضياً بقيمة `false` للحفاظ على التوافق العكسي.

---

### 2. تفاصيل التحديثات البرمجية

#### 2.1. منطق تعيين القيم الافتراضية في `addV2`

تم إضافة الكود التالي بعد تحضير بيانات العنوان `$data_add` وقبل إنشاء السجل:

```php
// تعيين الدولة الافتراضية إذا كان مفعلاً ولم يتم تمرير country_id
if(\Config::get('nano.locationapi::address.is_allow_default_country_in_add',false) && (! isset($data_add['country_id']) || empty($data_add['country_id'])))
{
    $def = \RainLab\Location\Models\Country::getDefault();
    if($def) {
        $data_add['country_id'] = $def->id;
    }
}

// تعيين المدينة الافتراضية إذا كان مفعلاً وتم تمرير country_id (أو تم تعيينه أعلاه) ولم يتم تمرير state_id
if(\Config::get('nano.locationapi::address.is_allow_default_state_in_add',false) && (isset($data_add['country_id']) && !empty($data_add['country_id'])) && (! isset($data_add['state_id']) || empty($data_add['state_id'])))
{
    $def = \RainLab\Location\Models\State::getDefault();
    $def_state_id = null;
    if($def) {
        $def_state_id = $def->id;
    }
    
    $nameList = \RainLab\Location\Models\State::getNameList($data_add['country_id']);
    
    if(!empty($nameList)) {
        if($def_state_id && isset($nameList[$def_state_id]))
            $data_add['state_id'] = $def_state_id;
        else
            $data_add['state_id'] = key($nameList);
    }
}

// تعيين المديرية الافتراضية إذا كان مفعلاً وتم تمرير country_id و state_id (أو تم تعيينهما أعلاه) ولم يتم تمرير directorate_id
if(\Config::get('nano.locationapi::address.is_allow_default_directorate_in_add',false) && (isset($data_add['country_id']) && !empty($data_add['country_id'])) && (isset($data_add['state_id']) && !empty($data_add['state_id'])) && (! isset($data_add['directorate_id']) || empty($data_add['directorate_id'])))
{
    $def = \RainLab\Location\Models\Directorate::getDefault();
    $def_directorate_id = null;
    if($def) {
        $def_directorate_id = $def->id;
    }
    
    $nameList = \RainLab\Location\Models\Directorate::getNameList($data_add['state_id']);
    
    if(!empty($nameList)) {
        if($def_directorate_id && isset($nameList[$def_directorate_id]))
            $data_add['directorate_id'] = $def_directorate_id;
        else
            $data_add['directorate_id'] = key($nameList);
    }
}
```

2.2. ترتيب الأولويات

يتم تطبيق القيم الافتراضية بالتسلسل:

1. الدولة الافتراضية – تعتمد على Country::getDefault().
2. المدينة الافتراضية – تُستخدم State::getDefault() إن وُجدت، وإلا يتم اختيار أول مدينة تابعة للدولة المحددة.
3. المديرية الافتراضية – تُستخدم Directorate::getDefault() إن وُجدت، وإلا يتم اختيار أول مديرية تابعة للمدينة المحددة.

2.3. التوافق مع الإصدارات السابقة

· الإعدادات الجديدة مغلقة افتراضياً (false) بحيث لا تتأثر التطبيقات الحالية.
· التغييرات قُصرت على دالة addV2 (الإصدار الثاني من API) ولا تؤثر على دالة addV1.
· يمكن للمطور تفعيل الميزات حسب الحاجة عبر ملف .env أو تغيير قيم الإعدادات مباشرة.

---

3. أمثلة تطبيقية

3.1. إضافة عنوان مع تفعيل الدولة الافتراضية فقط

```env
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD=true
```

```json
POST /api/v1/location/address/add?api_version=v2
{
    "address_1": "شارع النيل",
    "address_2": "بجوار البنك"
}
```

في هذه الحالة، سيتم تعيين country_id تلقائياً إلى الدولة الافتراضية.

3.2. إضافة عنوان مع تفعيل الدولة والمدينة الافتراضية

```env
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD=true
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_STATE_IN_ADD=true
```

```json
POST /api/v1/location/address/add?api_version=v2
{
    "address_1": "شارع النيل",
    "address_2": "بجوار البنك"
}
```

سيتم تعيين country_id و state_id بالقيم الافتراضية تلقائياً.

3.3. إضافة عنوان مع تمرير الدولة فقط وتفعيل المدينة الافتراضية

```json
POST /api/v1/location/address/add?api_version=v2
{
    "address_1": "شارع النيل",
    "country_id": 1
}
```

إذا كانت المدينة الافتراضية مفعلة، سيتم اختيار المدينة المناسبة للدولة رقم 1.

3.4. تعطيل جميع الإعدادات (السلوك القديم)

```env
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_COUNTRY_IN_ADD=false
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_STATE_IN_ADD=false
NANO_LOCATIONAPI_ADDRESS_IS_ALLOW_DEFAULT_DIRECTORATE_IN_ADD=false
```

في هذه الحالة، أي طلب لا يحتوي على country_id أو state_id أو directorate_id سيفشل كما كان سابقاً.

---

4. القيمة المضافة

· للمطورين: تقليل الحاجة إلى جلب معرفات الدولة والمدينة والمديرية قبل إضافة العنوان، مما يبسط الكود في التطبيقات العميلة (تطبيقات الهاتف، واجهات المستخدم).
· للمستخدمين النهائيين: تجربة أكثر سلاسة عند إضافة عنوان، حيث يمكنهم ترك الحقول الاختيارية دون الحاجة لإدخالها يدوياً.
· للنظام: تحسين مرونة واجهة البرمجة وجعلها أكثر ذكاءً، مع الاحتفاظ بالتوافق العكسي.
· قابلية التوسع: إضافة الإعدادات عبر ملف config.php يسمح بتخصيص السلوك لكل مشروع دون تعديل الكود الأساسي.

---

5. الخاتمة

يمثل هذا التحديث إضافة مهمة لوحدة Nano.LocationApi، حيث أصبح بإمكان المطورين التحكم في استخدام القيم الافتراضية للكيانات المكانية (الدولة، المدينة، المديرية) عند إنشاء عنوان عبر API. من خلال إعدادات مرنة وتنفيذ دقيق في دالة addV2، تم تحقيق توازن بين السهولة والمرونة، مع الحفاظ على استقرار المشاريع القائمة التي لا تحتاج إلى هذه الميزة.

---

ملاحظة: لمزيد من التفاصيل حول إعدادات الوحدة وكيفية استخدام الإصدار الثاني من API، يُرجى الرجوع إلى ملف التوثيق الخاص بـ Nano.LocationApi أو مراجعة الأمثلة المقدمة أعلاه.

## 2025-9-17 - 2026-4-2 

### إضافة بوابة دفع ThawaniPay إلى نظام المدفوعات في نانوسوفت

ضمن الوحدة البرمجية `Nano.Yepayment`  

---

### 1. مقدمة

تم تطوير طريقة دفع جديدة باسم **ThawaniPay** وإضافتها إلى نظام المدفوعات الموحد في نانوسوفت ضمن الوحدة `Nano.Yepayment`. تأتي هذه الإضافة استجابةً للحاجة إلى دعم بوابات الدفع الإقليمية في سلطنة عُمان، حيث تُعد **Thawani Pay** أول بوابة دفع إلكترونية مرخصة من البنك المركزي العُماني (CBO).

تهدف هذه الإضافة إلى:
- تمكين تجار نانوسوفت من قبول المدفوعات عبر بطاقات الائتمان والخصم المباشر من خلال بوابة ثواني.
- توفير تجربة دفع سلسة وآمنة مع إعادة توجيه المستخدم إلى صفحة دفع مستضافة.
- دمج كامل مع نظام `Nano.Yepayment` الحالي، بما في ذلك إدارة المعاملات والمبالغ المستردة والاشتراكات الدورية.
- دعم بيئتي الاختبار (UAT) والإنتاج (Live) عبر مفاتيح API منفصلة.

---

### 2. المكونات المطورة

لتنفيذ هذا التكامل، تم إنشاء وتعديل الكلاسات التالية داخل الوحدة `Nano.Yepayment`:

#### 2.1 `ThawaniPay` – كلاس بوابة الدفع الرئيسي
- **المسار**: `Nano\Yepayment\PaymentTypes\ThawaniPay`
- **الوراثة**: يمتد من `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **الوظيفة**: مسؤول عن إنشاء جلسات الدفع، معالجة الإشعارات، الاستعلام عن حالة الدفع، واسترداد المبالغ.

**الدوال الرئيسية**:

| الدالة | الوصف |
|--------|-------|
| `process(PaymentResult $result)` | إنشاء جلسة دفع جديدة عبر API ثواني، تخزين `session_id` في `order.payment_first_trans_id`، وإعادة رابط إعادة التوجيه. |
| `complete(PaymentResult $result)` | تأكيد الدفع بعد عودة المستخدم من بوابة ثواني (تُستخدم حالياً للتحقق والتحديث). |
| `createPaymentThawani($order, $is_test)` | دالة ثابتة لإنشاء جلسة دفع بناءً على كائن الطلب أو بيانات الفاتورة. |
| `checkSessionPay($options)` | دالة ثابتة للاستعلام عن حالة جلسة دفع (`session_id`، `invoice`، أو `reference`). |
| `checkAndCompletePay($options)` | دالة ثابتة تجمع بين فحص الجلسة وتحديث حالة الطلب إذا كانت مدفوعة. |
| `createRedirectUrl($session_id, $is_test)` | توليد رابط إعادة التوجيه إلى بوابة ثواني بصيغة `https://checkout.thawani.om/pay/{session_id}?key={publishable_key}`. |
| `createRequestDataByOrder($order, $is_test_mod)` | بناء بيانات الطلب (products, metadata, success_url, cancel_url) من كائن Order. |
| `createRequestProcessDataByOrder($order, $is_test_mod)` | معالجة بيانات الطلب (تحويل العملة إلى أصغر وحدة، حساب total_thawani). |

#### 2.2 `Http` – كلاس الاتصالات (CURL)
- **المسار**: `Nano\Yepayment\Classes\Http`
- **الوظيفة**: يوفر دوال منخفضة المستوى لإرسال طلبات HTTP (GET, POST, JSON, Form-data) باستخدام cURL، مع دعم الوكيل (proxy) وإعادة توجيه.

**الدوال الرئيسية**:

| الدالة | الوصف |
|--------|-------|
| `getData($url, $headers)` | إرسال طلب GET وإرجاع النص الخام أو رسالة الخطأ. |
| `postJsonData($url, $array_post, $headers, $basic)` | إرسال طلب POST مع `Content-Type: application/json`. |
| `postData($url, $array_post, $headers, $basic)` | إرسال طلب POST مع `application/x-www-form-urlencoded`. |
| `getResponsData($response)` | تحويل الاستجابة (كائن أو سلسلة) إلى مصفوفة PHP مع دعم تنسيقات متعددة (JSON، Guzzle Response، إلخ). |

#### 2.3 `RedirectHelper` – كلاس مساعد لإعادة التوجيه
- **المسار**: `Nano\Yepayment\Classes\RedirectHelper`
- **الوظيفة**: يوفر آليات متقدمة لإعادة التوجيه إلى التطبيقات (deeplinks) أو المواقع، مع دعم الترميز باستخدام token، تصفية البيانات الحساسة، والتحقق من صحة الروابط.

**الدوال الرئيسية**:

| الدالة | الوصف |
|--------|-------|
| `redirectToApp($callbackUrl, $data, $forceJson, $queryParams, $deepLinkSchemes, $is_force_deep_list)` | إعادة توجيه ذكية تدعم deeplinks و JSON والويب. |
| `isValidDeepLink($url, $deepLinkSchemes, $is_force_deep_list)` | التحقق مما إذا كان الرابط من نوع deeplink (مخطط غير http/https). |
| `filterSensitiveData($data, $sensitiveKeys)` | إزالة الحقول الحساسة (passwords, tokens, secrets) من البيانات قبل الإرسال. |
| `redirectUrlWithToken($callbackUrl, $data, $token, $queryParams)` | حفظ البيانات الكبيرة في Cache وإرجاع رابط مع token. |
| `getNestedValue($array, $key, $default, $delimiter)` | استخراج قيمة من مصفوفة متداخلة باستخدام تدوين النقاط (مثل 'thawani.callback_success_url'). |

#### 2.4 `thawanipay.php` – ملف الإعدادات
- **المسار**: `config/thawanipay.php` (ضمن الوحدة)
- **الوظيفة**: تخزين إعدادات الاتصال بـ Thawani API، بما في ذلك مفاتيح البيئة (test/live)، وعناوين URL لنقاط النهاية المختلفة.

**المحتوى الرئيسي**:
```php
return [
    'api_key' => env('THAWANIPAY_API_KEY', '...'),
    'secret_key' => env('THAWANIPAY_SECRET_KEY', '...'),
    'publishable_key' => env('THAWANIPAY_PUBLISHABLE_KEY', '...'),
    'url' => [
        'base' => 'https://checkout.thawani.om/api/v1',
        'test' => 'https://uatcheckout.thawani.om/api/v1/',
        'payment' => 'checkout/session',
        'redirect_pay' => 'https://checkout.thawani.om/pay/',
        'redirect_pay_test' => 'https://uatcheckout.thawani.om/pay/',
    ],
    'test' => [...],
];
```

#### 2.5 `Plugin.php` – تسجيل مزود الدفع
- **المسار**: `Nano\Yepayment\Plugin`
- **الوظيفة**: تسجيل `ThawaniPay` كمزود دفع في نظام `Nano.MicroCart` عبر دالة `registerPaymentProviders()` عند تفعيل خيار `allow_oman_payment`.

#### 2.6 `routes.php` – نقاط نهاية API
- **المسار**: `routes.php`
- **الوظيفة**: توفير نقاط نهاية لمعالجة ردود بوابة ثواني (success, cancel)، والاستعلام (retrieve)، والاختبار (test).

**نقاط النهاية**:
| المسار | الوصف |
|--------|-------|
| `GET /api/v1/thawani/success` | معالجة الدفع الناجح: استدعاء `checkAndCompletePay`، ثم إعادة التوجيه إلى `callback_success_url` (إن وُجد) أو عرض رسالة. |
| `GET /api/v1/thawani/cancel` | معالجة إلغاء الدفع: إعادة التوجيه إلى `callback_error_url` أو عرض رسالة. |
| `GET /api/v1/thawani/retrieve` | الاستعلام عن حالة جلسة دفع (معاملات: `session_id`, `type`, `is_test`). |
| `GET /api/v1/thawani/test` | واجهة اختبار متكاملة لإنشاء جلسة دفع تجريبية مع خيار إعادة التوجيه أو إرجاع JSON. |

#### 2.7 `_info.htm` – قالب معلومات الإعدادات
- **المسار**: `paymenttypes/thawanipay/_info.htm`
- **الوظيفة**: عرض إرشادات الإعداد في لوحة التحكم، مع روابط الاختبار والوثائق وبطاقات الاختبار.

---

### 3. دورة عمل الدفع (Payment Workflow)

يتم تنفيذ عملية الدفع عبر ThawaniPay وفق الخطوات التالية:

1. **إنشاء جلسة دفع**  
   عند استدعاء `process()`، يقوم الكلاس بـ:
   - التحقق من صحة الطلب (عدم وجود دفع سابق).
   - استدعاء `createRequestDataByOrder()` لبناء بيانات الفاتورة (المنتجات، المبلغ، العملة، عناوين النجاح والإلغاء).
   - استدعاء `createSessionPay()` لإرسال طلب `POST` إلى `https://.../checkout/session` مع `thawani-api-key`.
   - استلام `session_id` من الاستجابة.
   - تخزين `session_id` في `order.payment_first_trans_id` وفي `order.other_data['thawani']`.
   - إرجاع رابط إعادة التوجيه عبر `createRedirectUrl()`.

2. **إعادة التوجيه إلى بوابة ثواني**  
   يتم توجيه المستخدم إلى `https://checkout.thawani.om/pay/{session_id}?key={publishable_key}`.

3. **إتمام الدفع**  
   يقوم المستخدم بإدخال بيانات بطاقته على صفحة ثواني الآمنة.

4. **معالجة الإشعار (Webhook عبر نقطة النهاية)**  
   بعد إتمام الدفع، تُعيد ثواني توجيه المستخدم إلى `success_url` (المحددة مسبقاً).  
   نقطة النهاية `/thawani/success`:
   - تستقبل معامل `session_id` (أو تستخرجه من الرابط).
   - تستدعي `ThawaniPay::checkAndCompletePay()` التي تقوم بـ:
     - استدعاء `retrieveSessionPayTest()` للاستعلام عن الجلسة عبر API.
     - التحقق من `payment_status == 'paid'`.
     - تحديث حالة الطلب إلى "مدفوع" عبر `PaymentResult::success()`.
     - تسجيل سجل الدفع في `PaymentLog`.
   - إذا وُجد `callback_success_url` في `order.other_data`، يتم إعادة التوجيه إليه مع تمرير البيانات (عبر deeplink أو query string).  
   - خلاف ذلك، يتم عرض رسالة نجاح مباشرة.

5. **إعادة التوجيه إلى صفحة الفشل**  
   إذا ألغى المستخدم الدفع أو فشل، يتم توجيهه إلى `/thawaniy/cancel`، ويتم إعادة التوجيه إلى `callback_error_url` إن وُجد.

---

### 4. الإعدادات والتهيئة (Configuration)

لتفعيل طريقة الدفع ThawaniPay، يجب إضافة الإعدادات التالية في متغيرات البيئة (`.env`) أو من خلال واجهة إعدادات بوابة الدفع في نظام نانوسوفت (`Nano\MicroCart\Models\PaymentGatewaySettings`):

```ini
# تفعيل ThawaniPay
THAWANI_ENABLED=true
THAWANI_MODE=test   # test / live

# مفاتيح بيئة الاختبار (UAT)
THAWANI_TEST_API_KEY=rRQ26GcsZzoEHZvLYDbn9C9et
THAWANI_TEST_SECRET_KEY=rRQ26GcsZzP2HZvLYDbn9C9et
THAWANI_TEST_PUBLISHABLE_KEY=HGvJghr9tlN9gr4DVYt0qyBy
THAWANI_TEST_SUCCESS_URL=https://yourdomain.com/api/v1/thawani/success
THAWANI_TEST_CANCEL_URL=https://yourdomain.com/api/v1/thawani/cancel

# مفاتيح بيئة الإنتاج (Live)
THAWANI_LIVE_API_KEY=...
THAWANI_LIVE_SECRET_KEY=...
THAWANI_LIVE_PUBLISHABLE_KEY=...
THAWANI_LIVE_SUCCESS_URL=...
THAWANI_LIVE_CANCEL_URL=...
```

**الإعدادات في لوحة التحكم** (حقل `settings()` في `ThawaniPay`):
| الإعداد | الوصف |
|---------|-------|
| `thawanipay_url` | رابط API الأساسي (افتراضي: `https://checkout.thawani.om/api/v1`). |
| `thawanipay_api_key` | مفتاح API (مطلوب). |
| `thawanipay_secret_key` | المفتاح السري. |
| `thawanipay_publishable_key` | المفتاح القابل للنشر (يُستخدم في رابط إعادة التوجيه). |

**ملاحظة**: يتم تخزين القيم الحساسة مشفرة عبر `encryptedSettings()`.

---

### 5. الأمثلة التوضيحية (Usage Examples)

#### 5.1. إنشاء جلسة دفع لطلب موجود

```php
use Nano\Yepayment\PaymentTypes\ThawaniPay;
use Nano\Orders\Models\Order;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(123);
$provider = new ThawaniPay($order, [
    'is_test' => true,  // استخدام بيئة الاختبار
]);

$result = new PaymentResult($provider, $order);
$processed = $provider->process($result);

if ($processed->successful && $processed->redirect) {
    // إعادة توجيه المستخدم إلى بوابة ثواني
    return redirect()->away($processed->redirectUrl);
} else {
    // عرض رسالة الخطأ
    return back()->withError($processed->message);
}
```

#### 5.2. الاستعلام عن حالة جلسة دفع

```php
$options = [
    'session_id' => 'checkout_xxxxx',
    'type' => 'session',   // session, invoice, reference
    'is_test' => true,
];
$result = ThawaniPay::checkSessionPay($options);

if ($result['status'] && ThawaniPay::checkPaidSessionId($result)) {
    // الدفع تم بنجاح
}
```

#### 5.3. إكمال الدفع بعد العودة من بوابة ثواني (في نقطة النهاية)

```php
// success
$options = Input::get();
$options['type'] = 'session';
$result = ThawaniPay::checkAndCompletePay($options);

if ($result['status'] && $result['data']['is_complete_pay_order']) {
    // تحديث حالة الطلب تلقائياً
    $callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order->id);
    return RedirectHelper::redirectToApp($callbackUrl, $result, false, ['order_id'], ['*']);
}
```

#### 5.4. اختبار الدفع عبر واجهة API

يمكن استخدام نقاط النهاية المخصصة للاختبار:

```bash
# إنشاء جلسة دفع تجريبية وإعادة التوجيه
GET /api/v1/thawani/test?is_test=1&is_redirect=1&order_id=200

# الاستعلام عن حالة جلسة
GET /api/v1/retrieve?is_test=1&type=session&session_id=checkout_xxxxx
```

---

### 6. التعامل مع عمليات إعادة التوجيه (Deeplinks & Callbacks)

يستخدم `RedirectHelper` لإعادة التوجيه إلى التطبيقات الخارجية (مثل تطبيقات الجوال) عبر `deeplink` أو إلى صفحات ويب. يتم استخراج عنوان `callback_success_url` و `callback_error_url` من `order.other_data` باستخدام دوال مثل:

```php
$callbackUrl = RedirectHelper::getCallbackSuccessUrlByOrder($order_id, [
    'callback_success_url',
    'thawani.callback_success_url'
]);
```

إذا كان الرابط من نوع deeplink (مخطط غير `http`/`https`)، يتم بناء رابط مع معاملات الحالة (`status`, `code`, `message`, `token`) وإعادة التوجيه. أما إذا كان رابط ويب عادي، فيتم إعادة التوجيه مع تمرير البيانات كمعاملات في URL أو باستخدام token إذا كانت البيانات كبيرة.

**مميزات `RedirectHelper`**:
- تصفية البيانات الحساسة تلقائياً (`password`, `token`, `secret`).
- دعم إعادة التوجيه عبر `token` للبيانات الكبيرة (تخزين مؤقت في Cache لمدة 5 دقائق).
- التحقق من صحة الروابط ضد نطاقات مسموحة أو مخططات deeplink مخصصة.

---

### 7. القيمة المضافة

- **للمطورين**: إضافة بوابات دفع جديدة عبر وراثة `PaymentProvider` مع دوال موحدة (`process`, `complete`). استخدام كلاس `Http` و `RedirectHelper` يقلل من تكرار كود الاتصال وإعادة التوجيه.
- **للتجار**: دعم بوابة دفع محلية مرخصة في سلطنة عُمان، مع إمكانية التبديل بين بيئة الاختبار والإنتاج بسهولة عبر متغيرات البيئة أو الإعدادات.
- **للمستخدمين النهائيين**: تجربة دفع سلسة وآمنة دون الحاجة إلى مغادرة الموقع لفترة طويلة، مع دعم بطاقات الائتمان والخصم المحلية والدولية.

---

### 8. اختبار التكامل (Integration Testing)

يوفر النظام نقاط نهاية مخصصة لاختبار التكامل دون الحاجة إلى طلب حقيقي:

- **بطاقات اختبار ثواني**: يمكن استخدام بطاقات من [وثائق ثواني](https://thawani-technologies.stoplight.io/docs/thawani-ecommerce-api/7c0f75e1668d7-thawani-test-card) (مثل `4242 4242 4242 4242` مع أي تاريخ مستقبلي و CVV عشوائي).

---

### 9. الخاتمة

يمثل إضافة طريقة الدفع `ThawaniPay` إلى نظام `Nano.Yepayment` خطوة مهمة في توسيع نطاق حلول المدفوعات التي توفرها نانوسوفت لتشمل بوابات الدفع الإقليمية المعتمدة في سلطنة عُمان. بفضل التصميم المعياري القائم على `PaymentProvider`، والكلاسات المساعدة القوية (`Http`, `RedirectHelper`)، ونقاط النهاية الجاهزة للاستخدام، يمكن للتجار والمطورين دمج ThawaniPay بسلاسة في مشاريعهم مع ضمان تجربة دفع موثوقة وآمنة. مع استمرار التطوير، سيتم دعم المزيد من بوابات الدفع الإقليمية والدولية لتلبية احتياجات الأسواق المختلفة.


### فترة التطوير والاختبار

استغرق تطوير طريقة الدفع ThawaniPay وإدماجها بالكامل في نظام `Nano.Yepayment` فترة تمتد من **17 سبتمبر 2025** وحتى **2 أبريل 2026**. لم تقتصر هذه الفترة على كتابة الكود فقط، بل شملت مراحل متعددة من الاختبارات المتكاملة (Integration Testing) في بيئة الاختبار (UAT) والتحقق من صحة تدفق الدفع بالكامل، بما في ذلك إنشاء الجلسات، ومعالجة الإشعارات (Webhooks)، وإعادة التوجيه عبر التطبيقات (Deeplinks)، والتعامل مع حالات الفشل والاسترداد. كما تضمنت عملية التطوير تحسينات مستمرة على كلاس `Http` لدعم الاتصال الآمن، وتطوير `RedirectHelper` لإعادة التوجيه الذكية، وبناء نقاط نهاية API متكاملة للاختبار والمراقبة. تم التأكد من استقرار البوابة في بيئة الإنتاج قبل اعتمادها رسمياً، وجرى توثيق جميع حالات الاستخدام لضمان تجربة دفع سلسة وآمنة.

## 2026-3-15 - 2026-04-03

**إطلاق إضافة `Nano.FileUpload` – نظام مركزي لإدارة رفع الملفات في تطبيقات نانوسوفت**

### إطلاق إضافة `Nano.FileUpload` – نظام مركزي لإدارة رفع الملفات

تم إطلاق إضافة **`Nano.FileUpload`** كإضافة برمجية مستقلة ضمن نظام نانوسوفت، تهدف إلى توحيد وإدارة عمليات رفع الملفات عبر واجهات برمجة التطبيقات (API) في جميع إضافات النظام. توفر الإضافة بنية متكاملة لتسجيل النماذج (الموديولات) التي تحتوي على حقول رفع، والتحقق من صلاحيات المستخدمين بأنواعهم المختلفة (backend/frontend)، وإدارة الملفات المؤقتة قبل حفظ السجلات، وتوفير نقاط نهاية API جاهزة للاستخدام مع هيكل استجابة موحد.

تم تصميم الإضافة لتعمل بشكل منفصل عن التطبيقات، بحيث يمكن لأي إضافة أخرى التسجيل فيها بسهولة عبر دوال مخصصة أو عبر الأحداث، مما يضمن إعادة استخدام منطق الرفع والصلاحيات دون تكرار الكود.

---

### 1. مقدمة

استجابةً للحاجة المتكررة لإضافة حقول رفع ملفات في العديد من إضافات نانوسوفت (مثل المنتجات، المستخدمين، الطلبات، وغيرها)، تم تطوير إضافة `Nano.FileUpload` كحل مركزي يدير:

- تسجيل النماذج وحقول الرفع مع إعدادات (الحجم الأقصى، الأنواع المسموحة، إلخ).
- تحديد أنواع المستخدمين المسموح لهم (backend / frontend) على مستوى النموذج أو الحقل.
- تعيين صلاحيات دقيقة لكل عملية (add, edit, delete, view) عبر نظام صلاحيات متكامل.
- دعم الرفع المؤقت باستخدام مفاتيح جلسة مؤقتة (temp session keys) لربط الملفات بالنماذج غير المحفوظة.
- توفير واجهة برمجة تطبيقات RESTful مع مصادقة OAuth 2.0.

تم بناء الإضافة على مبادئ فصل المسؤوليات باستخدام كلاسات متخصصة (Registry, Service, UserManager, Controller)، مما يجعلها قابلة للتوسع والصيانة بسهولة.

---

### 2. أهداف التحديثات

- **إنشاء إضافة برمجية مستقلة**: توفير `Nano.FileUpload` كحزمة قابلة لإعادة الاستخدام في جميع مشاريع نانوسوفت.
- **تسجيل مركزي للنماذج**: إتاحة تسجيل أي نموذج مع إعدادات حقوله عبر دالة `registerFileUploadFields` في الإضافات أو عبر حدث `nano.api.fileupload.registerModels`.
- **دعم أنواع المستخدمين والصلاحيات**: التعامل مع مستخدمي `backend` و `frontend` بشكل منفصل، مع إمكانية تخصيص صلاحيات لكل عملية.
- **الرفع المؤقت**: تمكين رفع الملفات قبل حفظ النموذج باستخدام مفاتيح جلسة مؤقتة، ثم ربطها بعد الحفظ.
- **واجهة API موحدة**: توفير نقاط نهاية للرفع الفردي والمتعدد، والحذف، والجلب، مع هيكل استجابة موحد وأمان عبر OAuth.
- **توثيق شامل**: إعداد توثيق كامل باللغة العربية يغطي الكلاسات والأمثلة ونقاط النهاية.

---

### 3. المكونات المطورة

#### 3.1 إضافة `Nano.FileUpload`

- **الاسم:** `Nano.FileUpload`
- **النطاق:** `Nano\FileUpload`
- **الهدف:** توفير نظام مركزي لإدارة رفع الملفات في تطبيقات نانوسوفت مع دعم الصلاحيات وأنواع المستخدمين المختلفة.
- **الهيكل الأساسي:**
  - `classes/FileUploadRegistry.php` – سجل النماذج المسجلة
  - `classes/FileUploadService.php` – طبقة الخدمة لتنفيذ عمليات الرفع والحذف والجلب
  - `classes/FileUploadUserManager.php` – إدارة المستخدم الحالي والتحقق من الصلاحيات
  - `controllers/FileUploadController.php` – متحكم API
  - `routes.php` – نقاط النهاية
  - `Plugin.php` – تهيئة الإضافة وتسجيل المسارات والأحداث
  - `lang/ar/lang.php` – ملف اللغة العربية
  - `docs/FileUpload/` – التوثيق بالعربية

#### 3.2 كلاس `FileUploadRegistry`

يقع الكلاس في المسار `Nano\FileUpload\Classes\FileUploadRegistry`، وهو السجل المركزي الذي يدير تعريفات النماذج وحقول الرفع.

**الوظائف الأساسية:**

| الدالة | الوصف |
|--------|-------|
| `registerModel($modelClass, $config)` | تسجيل نموذج مع إعداداته. |
| `getRegisteredModels()` | جلب جميع النماذج المسجلة. |
| `isModelRegistered($modelClass)` | التحقق من وجود نموذج مسجل. |
| `getModelConfig($modelClass)` | جلب إعدادات النموذج. |
| `getFieldConfig($modelClass, $field)` | جلب إعدادات حقل معين. |
| `isUserTypeAllowed($modelClass, $userType)` | التحقق من نوع المستخدم. |
| `can($modelClass, $operation, $userType, $user, $field)` | التحقق من صلاحية عملية معينة. |
| `getFieldConstraints($modelClass, $field)` | الحصول على قيود الحقل (الحجم، الأنواع). |
| `updateModelConfig($modelClass, $config)` | تحديث إعدادات نموذج مسجل. |

**بنية إعدادات النموذج:**

```php
[
    'enabled' => true,
    'allowed_user_types' => ['backend', 'frontend'],
    'permissions' => [
        'add'    => null,
        'edit'   => null,
        'delete' => null,
        'view'   => null,
    ],
    'fields' => [
        'image' => [
            'type' => 'image', // file, image, multiple
            'label' => 'الصورة الرئيسية',
            'max_filesize' => 2048,
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
            'multiple' => false,
            'permissions' => [ /* تجاوز صلاحيات النموذج */ ]
        ],
    ],
    'defaults' => [ /* إعدادات افتراضية */ ]
]
```

#### 3.3 كلاس `FileUploadService`

يقع في `Nano\FileUpload\Classes\FileUploadService`، وهو المسؤول عن تنفيذ عمليات رفع الملفات وحذفها واسترجاعها.

**الوظائف الأساسية:**

| الدالة | الوصف |
|--------|-------|
| `generateTempSessionKey($modelClass, $field, $userId)` | توليد مفتاح جلسة مؤقت. |
| `upload($modelClass, $field, $fileData, $options)` | رفع ملف واحد. |
| `uploadMultiple($modelClass, $field, $filesData, $options)` | رفع عدة ملفات. |
| `deleteFile($fileId, $modelClass, $field, $user)` | حذف ملف مع التحقق من الصلاحية. |
| `getFiles($modelClass, $field, $modelId, $options)` | جلب الملفات المرتبطة. |
| `attachTempFiles($model, $field, $tempSessionKey)` | ربط الملفات المؤقتة بنموذج محفوظ. |
| `validateFile($modelClass, $field, $fileData)` | التحقق من صحة الملف (حجم، نوع). |

#### 3.4 كلاس `FileUploadUserManager`

يقع في `Nano\FileUpload\Classes\FileUploadUserManager`، وهو مدير المستخدمين والصلاحيات الموحد.

**الوظائف الأساسية:**

| الدالة | الوصف |
|--------|-------|
| `setUserResolver(callable $resolver)` | تعيين محلل مستخدم مخصص. |
| `setUser($user)` | تعيين المستخدم مباشرة. |
| `getUser()` | جلب المستخدم الحالي (مع تخزين مؤقت). |
| `clearUser()` | إعادة تعيين المستخدم المخبأ. |
| `setPermissionChecker(callable $checker)` | تعيين دالة تحقق مخصصة. |
| `getUserType()` | تحديد نوع المستخدم (backend/frontend/guest). |
| `getId()` | جلب معرف المستخدم. |
| `checkPermission($operation, $permissions, $user)` | التحقق من الصلاحية. |

#### 3.5 متحكم `FileUploadController`

يقع في `Nano\FileUpload\APIControllers\FileUploadController`، ويوفر نقاط نهاية RESTful محمية بـ OAuth. جميع الدوال تستدعي `FileUploadService` وتعيد استجابات بهيكل موحد.

**نقاط النهاية الرئيسية:**

| المسار | الطريقة | الوصف |
|--------|--------|-------|
| `/upload` | POST | رفع ملف واحد (multipart أو base64). |
| `/upload-multiple` | POST | رفع عدة ملفات. |
| `/delete/{id}` | DELETE | حذف ملف. |
| `/files` | GET | جلب الملفات المرتبطة (بنموذج أو بمفتاح مؤقت). |

#### 3.6 ملف التوجيه (routes.php)

تم إنشاء ملف توجيه متكامل يوفر جميع نقاط النهاية ضمن مجموعة خارجية تحدد المسار الأساسي `/api/v1/fileupload`، ومجموعة داخلية محمية بـ middleware `oauth-users`.

---

### 4. آلية العمل (التدفق الجديد)

1. **تسجيل النموذج (مرة واحدة)**:
   - في إضافة أخرى، يتم تنفيذ دالة `registerFileUploadFields()` أو الاستماع لحدث `nano.api.fileupload.registerModels`.
   - يقوم `FileUploadRegistry` بجمع التعريفات وتسجيلها في الذاكرة.

2. **رفع ملف (عبر API)**:
   - يرسل العميل طلب `POST /upload` مع `model_class`، `field`، والملف (أو base64).
   - `FileUploadController` يتحقق من وجود النموذج والحقل.
   - يستدعي `FileUploadService::upload` مع الخيارات.
   - يتحقق `FileUploadService` من الصلاحية عبر `FileUploadRegistry::can` باستخدام المستخدم الحالي من `FileUploadUserManager`.
   - يتم رفع الملف عبر `Base64::onUpload` (من `Nano.Api`).
   - إذا لم يتم تمرير نموذج محفوظ، يتم توليد مفتاح جلسة مؤقت وإعادته مع الاستجابة.

3. **ربط الملفات المؤقتة**:
   - بعد حفظ النموذج (مثلاً منتج جديد)، يستدعي المطور `FileUploadService::attachTempFiles` لنقل الملفات المؤقتة إلى النموذج.

4. **جلب الملفات**:
   - يمكن استدعاء `GET /files` مع `model_id` لجلب الملفات المرتبطة بنموذج، أو مع `temp_session_key` لجلب الملفات المؤقتة.

---

### 5. أبرز الإنجازات والميزات

- **إضافة برمجية متكاملة**: تم إطلاق `Nano.FileUpload` كحل مركزي لإدارة رفع الملفات في جميع إضافات نانوسوفت.
- **تسجيل مرن للنماذج**: دعم التسجيل عبر دوال الإضافات أو عبر الأحداث، مع إعدادات تفصيلية لكل حقل.
- **دعم أنواع المستخدمين**: التعامل مع مستخدمي `backend` و `frontend` بشكل منفصل، مع إمكانية تحديد `allowed_user_types` لكل نموذج أو حقل.
- **صلاحيات دقيقة**: صلاحيات لكل عملية (`add`, `edit`, `delete`, `view`) على مستوى النموذج أو الحقل، مع دمج نظام صلاحيات `backend` (hasAccess/hasPermission) وإمكانية تخصيص التحقق لـ `frontend`.
- **الرفع المؤقت**: إمكانية رفع الملفات قبل حفظ النموذج باستخدام مفاتيح جلسة مؤقتة (`temp_session_key`)، ثم ربطها لاحقًا عبر `attachTempFiles`.
- **واجهة API جاهزة**: نقاط نهاية RESTful مع مصادقة OAuth، وهيكل استجابة موحد (`code`, `status`, `message`, `data`, ...).
- **دعم تنسيقات متعددة**: رفع الملفات عبر `multipart/form-data` أو عبر سلسلة base64.
- **معالجة أخطاء احترافية**: عرض رسائل آمنة للمستخدم في الإنتاج، مع تسجيل التفاصيل الكاملة في بيئة التطوير.
- **توثيق شامل**: ملفات توثيق تفصيلية باللغة العربية تغطي جميع الكلاسات والأمثلة ونقاط النهاية.

---

### 6. الفوائد والقيمة المضافة

- **للمطورين**:
  - إمكانية إضافة حقول رفع ملفات في أي إضافة ببساطة عبر التسجيل.
  - توحيد منطق الرفع والصلاحيات يقلل من تكرار الكود.
  - دعم الرفع المؤقت يسهل إنشاء النماذج متعددة الخطوات.
  - واجهة API موحدة ومحمية بأمان.

- **للمستخدمين النهائيين**:
  - تجربة رفع ملفات سلسة ومتسقة عبر التطبيق.
  - رسائل خطأ واضحة تساعد في تصحيح الإدخالات.

- **للنظام ككل**:
  - أمان محسن عبر التحقق من الصلاحيات قبل كل عملية.
  - مرونة عالية في إدارة أنواع المستخدمين.
  - هيكل قابل للتوسع لإضافة مزيد من الميزات مستقبلاً (مثل الضغط التلقائي، تحسين الصور).

---

### 7. خطط التطوير المستقبلية

- **دعم أنواع ملفات إضافية**: إضافة معالجة خاصة لأنواع معينة (PDF، مستندات).
- **تحسين معالجة الصور**: دعم ضغط الصور تلقائياً، وإضافة علامات مائية.
- **واجهة إدارة رسومية**: تطوير واجهة إدارة لمراقبة الملفات المرفوعة وإدارة الصلاحيات.
- **تكامل مع خدمات سحابية**: دعم رفع الملفات مباشرة إلى خدمات التخزين السحابي (AWS S3، إلخ).
- **إشعارات عبر Webhook**: إرسال إشعارات بعد نجاح رفع الملفات.

---

### 8. الخاتمة

يمثل إطلاق إضافة `Nano.FileUpload` نقلة نوعية في كيفية إدارة عمليات رفع الملفات في تطبيقات نانوسوفت. من خلال توفير كلاسات `FileUploadRegistry`، `FileUploadService`، `FileUploadUserManager`، ومتحكم `FileUploadController`، أصبح بإمكان المطورين إضافة حقول رفع ملفات في أي إضافة بسرعة وأمان، مع التحكم الكامل في الصلاحيات وأنواع المستخدمين. الإضافة مصممة لتكون قابلة للتوسع، مما يسمح بإضافة ميزات متقدمة مستقبلاً دون التأثير على الاستقرار الحالي.

مع الانتهاء من هذه المرحلة، تصبح الإضافة جاهزة للاستخدام في مشاريع نانوسوفت المختلفة، ونتطلع إلى مواصلة التطوير بناءً على ملاحظات المستخدمين والمتطلبات المتطورة.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)

