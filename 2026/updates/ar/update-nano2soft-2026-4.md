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

## 2026-04-03 - 2026-04-05

**تحديثات إضافة `Nano.FileUpload` – الإصدارات 1.0.2 إلى 1.0.5**

### ملخص التحديثات

منذ الإصدار الأول للإضافة، واصلنا تطوير `Nano.FileUpload` بناءً على ملاحظات المستخدمين والمتطلبات الأمنية. تضمنت التحديثات في الإصدارات 1.0.2 حتى 1.0.5 تحسينات كبيرة في نظام الصلاحيات، وإعدادات افتراضية لأنواع الملفات، وأمان المفاتيح المؤقتة، والتخزين المؤقت، ودعم تعدد اللغات، وتسجيل الأخطاء، ومنع الملفات الخطيرة.

---

## الإصدار 1.0.2 – تحسينات الأمان والصلاحيات والإعدادات الافتراضية

### أهداف الإصدار

- توفير تحكم عام على مستوى API (تعطيل الرفع، الحذف، الجلب) عبر متغيرات البيئة.
- إضافة إعدادات افتراضية لأنواع الملفات (`image`, `audio`, `video`, `file`) لتقليل التكرار في تسجيل النماذج.
- تعزيز أمان مفاتيح الجلسة المؤقتة بتضمين `userType` والتوقيع باستخدام HMAC-SHA256.
- إضافة التحقق من صحة المفتاح المؤقت قبل ربط الملفات.

### الميزات الجديدة

#### 1. التحكم العام في عمليات API

أضفنا إعدادات عامة في `config.php` تسمح بتعطيل عمليات الرفع، الحذف، أو الجلب على مستوى API بالكامل، مع إمكانية التحكم عبر متغيرات البيئة:

```ini
NANO_FILE_UPLOAD_DISABLE_UPLOAD=false
NANO_FILE_UPLOAD_DISABLE_DELETE=false
NANO_FILE_UPLOAD_DISABLE_GET=false
```

أضفنا دوال مساعدة في `FileUploadRegistry`:
- `isUploadEnabledGlobally()`
- `isDeleteEnabledGlobally()`
- `isGetEnabledGlobally()`

وتضمنت دالة `can()` التحقق من هذه الإعدادات قبل أي عملية.

#### 2. الإعدادات الافتراضية لأنواع الملفات

أضفنا قسم `defaults` في `config.php` مع إعدادات لكل نوع من الملفات:
- `image`: الحجم الأقصى، الأنواع المسموحة، استخدام التسمية، خيارات الصور المصغرة.
- `audio`, `video`, `file` مع إعدادات مماثلة.

عند تسجيل حقل، إذا لم يحدد المطور `max_filesize` أو `allowed_types` أو `use_caption`، يتم استخدام الإعدادات الافتراضية تلقائياً حسب نوع الحقل.

#### 3. تعزيز أمان مفاتيح الجلسة المؤقتة

- تم تعديل `generateTempSessionKey()` لتضمين `userType` و `timestamp` في البيانات المشفرة.
- استخدام `hash_hmac('sha256', $data, $secret)` لتوقيع المفتاح، مما يمنع التلاعب أو التخمين.
- إضافة دالة `validateTempSessionKey()` للتحقق من صحة المفتاح وصلاحيته (مدة صلاحية افتراضية ساعة واحدة).
- في دالة `attachTempFiles()`، يتم التحقق من:
  - صحة المفتاح (التوقيع، الانتهاء).
  - تطابق `modelClass` و `field`.
  - تطابق `userId` و `userType` مع المستخدم الحالي.

هذا يمنع أي مستخدم من الوصول إلى ملفات مؤقتة خاصة بمستخدم آخر.

#### 4. تحديث `FileUploadRegistry::can()` لتضمين التحقق العالمي

أصبحت دالة `can()` تتحقق أولاً من الإعدادات العامة لكل عملية (`add`, `delete`, `view`) قبل التحقق من نوع المستخدم والصلاحيات المحددة.

#### 5. دوال مساعدة إضافية

- `getDefaultConfigForType($type)`: لجلب الإعدادات الافتراضية لنوع معين.
- `applyDefaultsToFieldConfig()`: لتطبيق الإعدادات الافتراضية على إعدادات الحقل.

### الفوائد

- إمكانية تعطيل عمليات API مؤقتاً دون تعديل الكود (مثلاً للصيانة).
- تقليل الكود المتكرر عند تسجيل النماذج.
- أمان أعلى للملفات المؤقتة، ومنع الوصول غير المصرح به.

---

## الإصدار 1.0.3 – تحسين أداء ومرونة `FileUploadRegistry`

### أهداف الإصدار

- إعادة هيكلة `FileUploadRegistry` لتحسين الأداء وقابلية التوسع.
- دعم التخزين المؤقت (Cache) لنتائج استعلامات الحقول والقيود.
- توفير واجهة تسجيل مرنة عبر `registerDefinition()` و `registerCallback()`.
- تطبيع تعريفات النماذج والحقول باستخدام دوال متخصصة.

### الميزات الجديدة

#### 1. فصل التعريفات الخام عن الكائنات الجاهزة

- `$rawDefinitions`: تخزين التعريفات كما هي من الإضافات.
- `$builtModels`: تخزين الكائنات الجاهزة بعد البناء (Lazy Loading).
- تحميل التعريفات من الإضافات مرة واحدة فقط (`$loaded` flag).

#### 2. دعم التخزين المؤقت (Cache)

أضفنا قسم `cache` في `config.php`:

```php
'registry' => [
    'cache' => [
        'enabled' => env('NANO_FILE_UPLOAD_REGISTRY_CACHE_ENABLED', true),
        'ttl' => env('NANO_FILE_UPLOAD_REGISTRY_CACHE_TTL', 3600),
    ],
],
```

ودوال:
- `getFieldConfig()` و `getFieldConstraints()` تستخدم `Cache::get()` و `Cache::put()`.
- `clearModelCache()` لمسح الكاش عند تحديث تعريف نموذج.
- `setCacheEnabled()`, `setCacheTtl()` للتحكم البرمجي.

#### 3. طرق تسجيل مرنة

- `registerDefinition($modelClass, array $definition)`: تسجيل تعريف خام.
- `registerDefinitions(array $definitions)`: تسجيل مجموعة.
- `registerCallback(callable $callback)`: تسجيل دالة رد تُنفذ عند التحميل (مثل `ReportsManager`).
- بقيت دالة `registerModel()` للتوافق مع الإصدارات السابقة.

#### 4. تطبيع التعريفات

- `normalizeModelDefinition()`: دمج الإعدادات الافتراضية.
- `normalizeFieldDefinition()`: تطبيق الإعدادات الافتراضية حسب نوع الحقل.
- `getDefaultConfigForType()`: جلب الإعدادات من ملف الإعدادات.

#### 5. تحسين دوال الاستعلام

- `isModelRegistered()` تستدعي `getRegisteredModels()` تلقائياً لتحميل التعريفات.
- `getModelConfig()` تستخدم `buildModel()` لبناء النموذج عند الحاجة.

### الفوائد

- أداء أفضل بفضل التخزين المؤقت (تجنب إعادة معالجة التعريفات في كل طلب).
- مرونة أكبر في تسجيل النماذج (عبر تعريفات خام أو دوال رد).
- هيكل كود أكثر نظافة وقابلية للصيانة.

---

## الإصدار 1.0.4 – تعزيز الأمان والتسجيل والاستثناءات

### أهداف الإصدار

- منع رفع الملفات الخطيرة (PHP, JS, HTML, EXE, إلخ) نهائياً حتى لو كانت في `allowed_types`.
- إنشاء قناة سجل منفصلة لتسجيل محاولات الرفع الفاشلة.
- إنشاء كلاس استثناء مخصص `FileUploadException` مع أكواد خطأ فريدة.
- تحسين رسائل الأخطاء وجعلها قابلة للترجمة (أساس للغة).

### الميزات الجديدة

#### 1. القائمة السوداء للملفات الخطيرة (Blacklist)

أضفنا ثابت `BLACKLISTED_EXTENSIONS` في `FileUploadService` يحتوي على امتدادات خطيرة مثل:
`php, js, html, exe, bat, sh, dll, ...`

ودالة `isBlacklistedExtension()` تتحقق من الامتداد قبل أي تحقق آخر. إذا كان الامتداد محظوراً، يتم رفض الملف فوراً مع رسالة مناسبة.

#### 2. تسجيل محاولات الرفع الفاشلة

- أضفنا قناة سجل جديدة `fileupload` في `config/logging.php` (تمت تهيئتها في `Plugin::boot()`).
- دالة `logFailedAttempt()` تسجل التفاصيل التالية:
  - المستخدم (ID, type, IP, User Agent)
  - النموذج والحقل
  - سبب الفشل (حجم، نوع، قائمة سوداء، صلاحية)
  - معلومات الملف (الاسم، الحجم، النوع، الامتداد)
- تُستدعى هذه الدالة عند حدوث أي خطأ في `validateFile()`.

#### 3. كلاس `FileUploadException`

تم إنشاء كلاس استثناء مخصص يرث من `Exception`، ويوفر:
- أكواد أخطاء رقمية (1000-1999 عام، 2000-2999 للملفات، 3000-3999 للصلاحيات، 4000-4999 للمفاتيح المؤقتة).
- توليد كود خطأ نصي فريد (مثل `FILE_UPLOAD_FILE_SIZE_EXCEEDED`).
- إمكانية إضافة سياق (`context`) للخطأ (مثل الحجم المسموح، الامتداد المرفوع).

#### 4. تحديث `FileUploadService::validateFile()` لتشمل:

- التحقق من القائمة السوداء أولاً.
- التحقق من الحجم والأنواع المسموحة (كما كان).
- رمي `FileUploadException` بدلاً من `ApplicationException`.
- استدعاء `logFailedAttempt()` قبل رمي الاستثناء.

#### 5. تحديث `FileUploadController`:

- دالة `handleException()` تتعرف على `FileUploadException` وتستخرج `errorCode`.
- دالة `errorResponse()` تقبل معامل `$errorCode` وتضيفه إلى الاستجابة.

### الفوائد

- أمان أعلى بمنع الملفات الخطيرة حتى لو تم تكوين `allowed_types` بشكل خاطئ.
- إمكانية تتبع الهجمات أو المشاكل من خلال سجل منفصل.
- توحيد التعامل مع الأخطاء وتوفير أكواد فريدة لتسهيل التعامل من الواجهات الأمامية.

---

## الإصدار 1.0.5 – دعم تعدد اللغات الكامل (Multilingual Support)

### أهداف الإصدار

- جعل جميع رسائل API (نجاح، خطأ) قابلة للترجمة إلى العربية والإنجليزية.
- إضافة مفاتيح ترجمة لكل عملية (`upload`, `uploadMultiple`, `delete`, `getFiles`).
- تحديث المتحكم والخدمة لاستخدام `trans()` بدلاً من النصوص الثابتة.

### الميزات الجديدة

#### 1. إضافة مفاتيح الترجمة إلى `lang.php`

أضفنا قسماً جديداً `public.helpers.upload` يحتوي على مفاتيح مثل:
- `msg_upload_success`
- `msg_upload_multiple_success`
- `msg_delete_success`
- `msg_get_success`
- `msg_permission_denied`
- `msg_validation_failed`
- `msg_upload_disabled`, `msg_delete_disabled`, `msg_get_disabled`
- وغيرها من رسائل الأخطاء الشائعة.

كما أضفنا مفاتيح في قسم `errors` للرسائل التفصيلية مثل `file_size_exceeded`, `file_type_not_allowed`, `file_type_blacklisted`.

#### 2. تحديث `FileUploadController`

- دوال `successResponse()` و `errorResponse()` تستخدم `trans()` مع القيم الافتراضية.
- دوال `upload()`, `uploadMultiple()`, `delete()`, `getFiles()` تستخدم `trans()` في كل رسالة (نجاح أو فشل).
- دالة `getSafeErrorMessage()` تستخدم `trans()` للرسائل العامة.
- في `uploadMultiple()`، يتم تمرير `success_count` و `total` إلى دالة الترجمة.

#### 3. تحديث `FileUploadService`

- استبدال جميع رسائل `ApplicationException` الثابتة بـ `trans()` باستخدام المفاتيح المناسبة.
- في `validateFile()`, يتم استخدام `trans()` في رسائل `FileUploadException` مع تمرير المعاملات (مثل `max_size`, `actual_size`, `extension`, `allowed`).

#### 4. دعم اللغتين العربية والإنجليزية

- ملفات الترجمة موجودة في `lang/ar/lang.php` و `lang/en/lang.php`.
- يمكن للمستخدم تبديل اللغة عبر `app.locale`.

### الفوائد

- تجربة مستخدم محسنة للمتحدثين بالعربية والإنجليزية.
- سهولة إضافة لغات جديدة مستقبلاً.
- توحيد جميع الرسائل في ملفات الترجمة، مما يسهل الصيانة والتعديل.

---

## ملخص الإصدارات (1.0.2 – 1.0.5)

| الإصدار | أبرز الميزات |
|---------|---------------|
| 1.0.2 | تحكم عام في API، إعدادات افتراضية لأنواع الملفات، تعزيز أمان المفاتيح المؤقتة. |
| 1.0.3 | إعادة هيكلة `FileUploadRegistry`، دعم التخزين المؤقت، تسجيل مرن عبر تعريفات ودوال رد. |
| 1.0.4 | قائمة سوداء للملفات الخطيرة، تسجيل محاولات الرفع الفاشلة، كلاس استثناء مخصص. |
| 1.0.5 | دعم كامل لتعدد اللغات (عربي/إنجليزي) لجميع رسائل API. |

---

## الخاتمة

بفضل هذه التحديثات، أصبحت إضافة `Nano.FileUpload` أكثر أماناً، أداءً، ومرونة. توفر الآن:
- تحكم دقيق في الصلاحيات على مستويات متعددة.
- إعدادات افتراضية ذكية تقلل من تكرار الكود.
- أمان عالٍ للملفات المؤقتة ولمنع الملفات الخطيرة.
- تسجيل شامل للأخطاء لمراقبة المحاولات الضارة.
- واجهة API موحدة مع رسائل قابلة للترجمة.

هذه التحسينات تجعل الإضافة جاهزة للاستخدام في أكبر المشاريع، مع إمكانية التوسع مستقبلاً.

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


## 2026-04-05 - 2026-04-07

**تحديثات إضافة `Nano.FileUpload` – الإصدارات 1.0.6 و 1.0.7**

### ملخص التحديثات

بعد إصدار الإصدار 1.0.5 الذي ركز على دعم تعدد اللغات، واصلنا تطوير الإضافة بإضافة ميزات متقدمة تهدف إلى زيادة المرونة والتوسع وتحسين إدارة الملفات على مستوى قاعدة البيانات. تضمنت الإصدارات 1.0.6 و 1.0.7:

- دعم التخزين المتعدد (multi‑storage) عبر أقراص مختلفة (S3, FTP, Local).
- التحويل التلقائي للصور (تحجيم تلقائي وعلامة مائية) مع إمكانية التحكم العام عبر الإعدادات.
- خطافات الأحداث (event hooks) قبل وبعد عمليات الرفع والحذف.
- دعم إشعارات WebSocket الاختيارية.
- إضافة أعمدة جديدة إلى جدول `system_files`: `disk`, `hash`, `meta`, `expires_at`.
- تعبئة تلقائية للحقول الجديدة (SHA256 للمحتوى، أبعاد الصورة، تاريخ انتهاء الملفات المؤقتة).
- تحسينات عامة في الأداء والأمان.

---

## الإصدار 1.0.6 – دعم التخزين المتعدد، التحويل التلقائي للصور، خطافات الأحداث، وتحسين قاعدة البيانات

### أهداف الإصدار

- تمكين رفع الملفات إلى خدمات تخزين متعددة (AWS S3، FTP، Local) عبر تحديد قرص التخزين على مستوى الحقل.
- إضافة إمكانية التحويل التلقائي للصور (تحجيم وعلامة مائية) أثناء الرفع.
- توفير خطافات (hooks) للأحداث لتوسيع السلوك دون تعديل الكود الأساسي.
- دعم إشعارات WebSocket للتحديثات الفورية في الواجهات الأمامية.
- إضافة أعمدة جديدة إلى جدول `system_files` لدعم الميزات المتقدمة (التخزين، التكرار، البيانات الإضافية، انتهاء الصلاحية).

### الميزات الجديدة

#### 1. دعم التخزين المتعدد (Multi‑Storage)

أضفنا إمكانية تحديد قرص تخزين مخصص لكل حقل عبر إعداد `storage_disk` في تعريف الحقل. هذا يتجاوز القرص الافتراضي المحدد في إعدادات النظام.

- **إضافة عمود `disk`** إلى جدول `system_files` لتخزين اسم القرص المستخدم.
- **تعديل نموذج `System\Models\File`** (عبر `Plugin::extendSystemFile()`) لدعم خاصية `disk` وتجاوز دالة `getDisk()` لاستخدام القرص المخصص.
- **إضافة دالة `getStorageDiskForField()`** في `FileUploadService` لاسترجاع اسم القرص من إعدادات الحقل والتحقق من وجوده.
- عند الرفع، يتم تعيين `$file->disk = $diskName` ثم حفظ الملف على القرص المحدد.

#### 2. التحويل التلقائي للصور (Auto‑resize & Auto‑watermark)

أضفنا إعدادات جديدة في تعريف الحقل:
- `auto_resize` (bool): تفعيل تحجيم الصورة تلقائياً.
- `resize_options` (array): خيارات التحجيم (العرض، الارتفاع، الوضع).
- `auto_watermark` (bool): تفعيل إضافة علامة مائية تلقائياً.
- `watermark_options` (array): خيارات العلامة المائية (الموضع، نسبة التحجيم).

أضفنا دالة `applyAutoProcessing()` في `FileUploadService` التي:
- تتحقق من وجود الملف كصورة.
- تستخدم `October\Rain\Resize\Resizer` لتحجيم الصورة وفق الخيارات.
- تستخدم إضافة `Nano2.Watermark` (إن كانت موجودة ومفعّلة) لإضافة العلامة المائية.

#### 3. خطافات الأحداث (Event Hooks)

أضفنا الأحداث التالية لتوسيع السلوك بسهولة:

| الحدث | مكان الاستدعاء | المعاملات |
|-------|----------------|------------|
| `nano.fileupload.beforeUpload` | قبل بدء معالجة الرفع | `$modelClass, $field, $fileData, &$options` |
| `nano.fileupload.afterUpload` | بعد نجاح الرفع وقبل إرجاع النتيجة | `$file, $modelClass, $field, $options` |
| `nano.fileupload.beforeDelete` | قبل حذف الملف | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterDelete` | بعد حذف الملف | `$fileId, $modelClass, $field` |

يمكن للمطورين الاستماع لهذه الأحداث لإضافة منطق مخصص (مثل إرسال إشعارات، تسجيل إضافي، تعديل البيانات).

#### 4. دعم WebSocket للإشعارات الفورية

أضفنا إعدادات `websocket` في `config.php`:
- `enabled`: تفعيل/تعطيل الإشعارات.
- `channel`: اسم القناة المستخدمة.

أضفنا دالة `notifyWebSocket()` في `FileUploadService` تطلق حدث `nano.fileupload.websocket.notify` بعد نجاح الرفع أو الحذف. يمكن للمطور استخدام مكتبة WebSocket (مثل Laravel WebSockets أو Pusher) لالتقاط هذا الحدث وإرسال الإشعارات.

#### 5. إضافة أعمدة جديدة إلى جدول `system_files`

قمنا بإنشاء هجرة `add_columns_to_system_files.php` لإضافة الأعمدة التالية:

| العمود | النوع | الغرض |
|--------|-------|-------|
| `disk` | `string, nullable` | تخزين اسم قرص التخزين المخصص (مثل `s3`, `ftp`). |
| `hash` | `string, nullable, index` | تخزين SHA256 للمحتوى لمنع التكرار. |
| `meta` | `text, nullable` | تخزين بيانات إضافية بصيغة JSON (أبعاد الصورة، مدة الصوت، إلخ). |
| `expires_at` | `dateTime, nullable` | تاريخ انتهاء صلاحية الملفات المؤقتة لتنظيفها تلقائياً. |

### الفوائد

- إمكانية تخزين الملفات في خدمات سحابية متعددة حسب الحاجة.
- تحسين تجربة المستخدم عبر تحجيم الصور تلقائياً وإضافة علامة مائية دون تدخل يدوي.
- توسيع النظام بسهولة عبر الأحداث دون تعديل الكود الأساسي.
- إشعارات فورية للمستخدمين عند اكتمال الرفع أو الحذف.
- إدارة أفضل للملفات المكررة والبيانات الوصفية والملفات المؤقتة.

---

## الإصدار 1.0.7 – إكمال التعبئة التلقائية لحقول `hash`, `meta`, `expires_at` وإضافة تحكم عالمي في التحويلات

### أهداف الإصدار

- إكمال تنفيذ تعبئة الحقول الجديدة (`hash`, `meta`, `expires_at`) تلقائياً أثناء الرفع.
- إضافة إعدادات عامة للتحكم في تفعيل/تعطيل التحويلات التلقائية (`auto_resize`, `auto_watermark`) على مستوى API بالكامل.
- تحسين الأداء والأمان عبر حساب التجزئة وتخزين أبعاد الصورة وتحديد صلاحية الملفات المؤقتة.

### الميزات الجديدة

#### 1. تعبئة تلقائية لـ `hash` (SHA256)

أضفنا في دالة `upload()` بعد الحصول على كائن الملف:
```php
if (!$file->hash) {
    $content = file_get_contents($file->getLocalPath());
    $file->hash = hash('sha256', $content);
    $is_changed = true;
}
```
هذا يضمن حساب تجزئة فريدة للمحتوى، مما يساعد في منع تخزين نسخ مكررة (يمكن التحقق من `hash` قبل الحفظ).

#### 2. تعبئة تلقائية لـ `meta` بأبعاد الصورة

أضفنا:
```php
if ($file->isImage()) {
    $dimensions = getimagesize($file->getLocalPath());
    $file->meta = array_merge($file->meta ?: [], [
        'width'  => $dimensions[0],
        'height' => $dimensions[1],
        'mime'   => $dimensions['mime'],
    ]);
    $is_changed = true;
}
```
يتم تخزين الأبعاد ونوع MIME في حقل `meta` كـ JSON، مما يسهل استرجاعها لاحقاً دون الحاجة لقراءة الملف مرة أخرى.

#### 3. تعيين `expires_at` للملفات المؤقتة

عند استخدام `temp_session_key` (أي الملف غير مرتبط بنموذج بعد)، نضبط تاريخ انتهاء صلاحية تلقائي:
```php
if ($tempSessionKey) {
    if (!$file->expires_at && (!$file->attachment_type || !$file->attachment_id)) {
        $file->expires_at = Carbon::now()->addHours(24);
        $is_changed = true;
    }
}
```
يمكن تشغيل مهمة مجدولة (cron) لحذف الملفات منتهية الصلاحية وغير المرتبطة تلقائياً.

#### 4. إضافة تحكم عام في التحويلات التلقائية

أضفنا إعدادين في قسم `general` بملف `config.php`:
```php
'disable_auto_resize'    => env('NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE', true),
'disable_auto_watermark' => env('NANO_FILE_UPLOAD_DISABLE_AUTO_WATERMARK', true),
```
ودالتين في `FileUploadRegistry`:
- `isAutoResizeEnabledGlobally()`
- `isAutoWatermarkEnabledGlobally()`

ثم عدلنا دالة `applyAutoProcessing()` لتحترم هذه الإعدادات:
```php
if ($this->registry->isAutoResizeEnabledGlobally() && $procOptions['auto_resize']) { ... }
if ($this->registry->isAutoWatermarkEnabledGlobally() && $procOptions['auto_watermark'] && ...) { ... }
```
هذا يسمح للمسؤول بتعطيل جميع عمليات التحجيم أو العلامات المائية مؤقتاً عبر متغيرات البيئة دون تعديل تعريفات الحقول.

#### 5. تحسينات أخرى

- استخدام `Carbon::now()` بدلاً من `now()` لضمان التوافق.
- تصحيح اسم المتغير `$is_changed` (بدلاً من `$is_chage`).
- ضمان حفظ الملف مرة أخرى عند تغيير أي من الحقول (`hash`, `meta`, `expires_at`).

### الفوائد

- إدارة أفضل للملفات المكررة عبر التحقق من `hash`.
- تخزين بيانات وصفية مفيدة (أبعاد الصورة) لاستخدامها في الواجهات.
- تنظيف تلقائي للملفات المؤقتة عبر `expires_at`.
- تحكم مركزي في تفعيل/تعطيل التحويلات التلقائية.

---

## ملخص الإصدارات (1.0.6 و 1.0.7)

| الإصدار | أبرز الميزات |
|---------|---------------|
| 1.0.6 | دعم التخزين المتعدد (disk)، التحويل التلقائي للصور، خطافات الأحداث، WebSocket، إضافة أعمدة جديدة في قاعدة البيانات. |
| 1.0.7 | تعبئة تلقائية لـ `hash`, `meta`, `expires_at`، تحكم عام في التحويلات التلقائية، تحسينات في الأداء والأمان. |

---

## متطلبات الترقية

1. **تشغيل الهجرة**:
   ```bash
   php artisan plugin:refresh Nano.FileUpload
   ```
   أو تنفيذ الهجرة الجديدة `add_columns_to_system_files.php`.

2. **إضافة متغيرات البيئة** (اختياري) في ملف `.env`:
   ```ini
   # تعطيل التحويلات التلقائية (القيم الافتراضية: true)
   NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE=true
   NANO_FILE_UPLOAD_DISABLE_AUTO_WATERMARK=true

   # إعدادات WebSocket
   NANO_FILE_UPLOAD_WEBSOCKET_ENABLED=false
   NANO_FILE_UPLOAD_WEBSOCKET_CHANNEL=file-uploads

   # إعدادات التخزين المتعدد (على مستوى الحقل، وليس عالمياً)
   # (يتم تعيينها في تعريفات الحقول مباشرة)
   ```

3. **تحديث تعريفات الحقول** لإضافة `storage_disk`, `auto_resize`, `auto_watermark` حسب الحاجة.

4. **الاستماع للأحداث** لتوسيع السلوك:
   ```php
   Event::listen('nano.fileupload.afterUpload', function ($file, $modelClass, $field, $options) {
       // إرسال إشعار عبر WebSocket
   });
   ```

---

## الخاتمة

بفضل الإصدارين 1.0.6 و 1.0.7، أصبحت إضافة `Nano.FileUpload` أكثر قوة ومرونة من أي وقت مضى. يمكنها الآن:

- التعامل مع أقراص تخزين متعددة (S3، FTP، Local).
- تحويل الصور تلقائياً (تحجيم، علامة مائية).
- إطلاق أحداث مخصصة لتوسيع السلوك.
- إرسال إشعارات فورية عبر WebSocket.
- تخزين بيانات وصفية غنية (أبعاد الصورة، التجزئة).
- تنظيف الملفات المؤقتة تلقائياً.

هذه الميزات تجعل الإضافة جاهزة للاستخدام في المشاريع الكبيرة والمعقدة التي تتطلب أداءً عالياً وأماناً متقدماً.

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

## 2026-03-11 – 2026-04-09

### إضافة نظام بناء الاستعلامات البصرية (Query Builder) ومعالج الشروط المتقدم  
ضمن الوحدة البرمجية  `Nano2.QueryBuilder`


---

### 1. مقدمة

تم تطوير نظام **بناء الاستعلامات البصري** (Visual Query Builder) لتوفير واجهة تفاعلية سهلة الاستخدام تسمح للمستخدمين النهائيين والمطورين بإنشاء شروط استعلام معقدة دون الحاجة إلى كتابة SQL أو منطق برمجي يدوي. يعتمد النظام على مكتبة `jQuery QueryBuilder` من ناحية الواجهة الأمامية، ويقدّم خلفية متكاملة ضمن برمجيات نانوسوفت عبر `FormWidget` مخصص وكلاس مساعد لتحويل الشروط إلى استعلامات صالحة للتنفيذ على نماذج البيانات (Models).

يهدف هذا التحديث إلى تمكين حالات استخدام مثل:
- بناء تقارير ديناميكية بشروط متعددة المستويات (AND/OR) وبأنواع مختلفة من المُدخلات (نص، أرقام، تواريخ، قوائم منسدلة، خانات اختيار، إلخ).
- تطبيق الشروط مباشرة على `Model` أو `Builder` عبر كلاس `QueryBuilderParserHelper`.
- إعادة استخدام تعريف الفلاتر والعمليات عبر تكوين YAML أو عبر دوال في الموديل.
- دعم الإضافات (plugins) مثل الترتيب بالسحب والإفلات، عرض وصف الفلاتر، تخصيص الأيقونات، وغيرها.

---

### 2. المكونات المطورة

#### 2.1 `QueryBuilderParserHelper` – كلاس معالجة الشروط وتحويلها إلى استعلامات  
- **المسار**: `Nano2\QueryBuilder\Classes\QueryBuilderParserHelper`
- **الوظيفة**: كلاس مساعد (Helper) يقوم بتحليل كائن الشروط (القواعد) القادم من الـ FormWidget، وتطبيقها على استعلام Builder، مع دعم المتغيرات الكلية (macros) والجداول المتداخلة والمشغلين المتعددين.

**أبرز الدوال**:

| الدالة | الوصف |
|--------|--------|
| `parse($builder, $rules, $rows, $is_exception)` | الدالة الرئيسية: تستقبل `$builder` (موديل، سلسلة اسم موديل، أو كائن Builder) ومصفوفة الشروط `$rules` (عادةً من `getRules()` للـ QueryBuilder)، وتُعيد `Builder` مطبَّقًا عليه الشروط. |
| `setFields($rules)` | تستخرج أسماء الحقول المستخدمة في الشروط لتمريرها إلى مكتبة `timgws\QueryBuilderParser` (ضروري لتحديد الأعمدة المسموحة). |
| `decodeJSON($json)` | تحويل سلسلة JSON إلى كائن للتحقق من صحة الشروط. |
| `isNested($rule)` | تتحقق إذا كانت القاعدة تحتوي على مجموعة فرعية من القواعد (للتعامل مع الشروط المتداخلة). |
| `getQueryOrModel($obj, $is_model, $is_exception)` | تستقبل كائنًا وقد يكون `Model`، أو اسم كلاس، أو `Builder`، وتُعيد إما كائن `Builder` أو الموديل نفسه حسب الحاجة. |
| `toSql($builder, $expand)` | (مأخوذ من LibreNMS) تحويل الشروط إلى جملة SQL نصية للتصحيح أو العرض. |

**آلية العمل**:
1. يتم تحويل كائن `$rules` (عادةً مصفوفة أو JSON) إلى كائن stdClass باستخدام `decodeJSON`.
2. تُستخرج الحقول المستخدمة عبر `setFields`.
3. يُنشأ كائن من مكتبة `timgws\QueryBuilderParser` مع تمرير الحقول المسموحة.
4. يُستدعى `parse($json, $query)` لتطبيق جميع `where` و `orWhere` والمجموعات المنطقية.
5. يتم إرجاع كائن `Builder` المعدل.

**مثال استخدام**:
```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

$rules = [
    'condition' => 'AND',
    'rules' => [
        ['id' => 'price', 'operator' => 'greater', 'value' => 100],
        ['id' => 'category', 'operator' => 'equal', 'value' => 5]
    ]
];
$query = Product::query();
$finalQuery = QueryBuilderParserHelper::parse($query, $rules);
$products = $finalQuery->get();
```

---

#### 2.2 `QueryBuilder` – عنصر واجهة (FormWidget) لبناء الاستعلامات  
- **المسار**: `Nano2\QueryBuilder\FormWidgets\QueryBuilder`
- **الوظيفة**: يوفر حقلاً مخصصًا في نماذج نانوسوفت يعرض واجهة jQuery QueryBuilder للمستخدم، ويخزن القواعد بشكل JSON في قاعدة البيانات.

**الخصائص الرئيسية (قابلة للتكوين عبر YAML أو خاصيات الكلاس)**:

| الخاصية | الوصف | القيمة الافتراضية |
|---------|------|-------------------|
| `filters` | تعريف الفلاتر المتاحة (كائنات من نوع `Filter`). يمكن أن تكون مصفوفة أو سلسلة تشير إلى دالة في الموديل. | `[]` |
| `rules` | القواعد الافتراضية التي تظهر عند تحميل الواجهة. | `[]` |
| `sort_filters` | ترتيب الفلاتر أبجدياً حسب الترجمة. | `false` |
| `allow_groups` | الحد الأقصى لعدد المجموعات المتداخلة (`-1` = غير محدود، `0` = لا مجموعات). | `-1` |
| `allow_empty` | السماح بوجود قواعد فارغة. | `false` |
| `conditions` | قائمة بالروابط المنطقية المسموحة (`AND`, `OR`). | `['AND', 'OR']` |
| `default_condition` | الرابط الافتراضي للمجموعة الجذرية. | `'AND'` |
| `inputs_separator` | الفاصل بين حقول الإدخال المتعددة (مثل BETWEEN). | `' , '` |
| `display_empty_filter` | عرض خيار فارغ في قائمة الفلاتر. | `true` |
| `select_placeholder` | النص المعروض عند عدم اختيار فلتر. | `'------'` |
| `plugins` | قائمة الإضافات المفعَّلة (مثل `sortable`, `filter-description`). | `[]` |
| `isShowParseSqlBtn` | إظهار زر لترجمة القواعد إلى SQL. | `false` |
| `isShowResetBtn` / `isShowSetBtn` / `isShowGetBtn` | أزرار إعادة الضبط، تحميل القواعد، وجلب القواعد. | `true` |

**آلية العمل في النموذج**:
1. في دالة `init()` يتم تحميل الإعدادات من YAML أو من الخاصيات.
2. في `prepareVars()` يتم تجهيز مصفوفة `filters` بعد معالجتها (دعم الترجمة، واستدعاء دوال الموديل للحصول على الخيارات).
3. يتم تحويل `$value` (القواعد المخزنة JSON) إلى كائن مناسب للجافاسكريبت.
4. يتم إضافة الأصول (CSS/JS) الخاصة بالمكتبة والإضافات.
5. عند عرض القالب الجزئي (`_query_builder_basic.htm`)، يتم تهيئة الـ jQuery QueryBuilder باستخدام الخيارات والقواعد.
6. عند تغيير المستخدم للقواعد، يتم تحديث حقل خفي (`input[type=hidden]`) بقيمة JSON، وعند حفظ النموذج تُخزَّن القيمة في قاعدة البيانات.

**دعم مصادر الخيارات الديناميكية**:
يمكن تعريف `values` في الفلتر كسلسلة نصية تشير إلى:
- اسم دالة في الموديل الحالي (مثل `getCategoryOptions`).
- اسم دالة ثابتة بصيغة `ClassName::method`.
- يتم استدعاؤها أثناء تحضير الفلاتر وتُمرر القيم الحالية والسياق.

**مثال تكوين YAML**:
```yaml
query_builder:
    type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
    filters:
        category:
            id: category
            label: التصنيف
            type: integer
            input: select
            values:
                1: كتب
                2: أفلام
                3: موسيقى
            operators: ['equal', 'not_equal', 'in']
        price:
            id: price
            label: السعر
            type: double
            validation:
                min: 0
                step: 0.01
    rules: 
        condition: AND
        rules: []
    allow_groups: 3
    sort_filters: true
    plugins:
        sortable: null
        filter-description: { mode: 'popover' }
```

---

### 3. الخيارات المتقدمة في الفلاتر (Filters)

يتبع كل فلتر نفس البنية المحددة في مكتبة jQuery QueryBuilder مع دعم إضافات نانوسوفت:

| الخاصية | الوصف |
|---------|------|
| `id` | معرف فريد للفلتر. |
| `label` | النص المعروض. يدعم الترجمة كمصفوفة أو كسلسلة. |
| `type` | نوع البيانات: `string`, `integer`, `double`, `date`, `time`, `datetime`, `boolean`. |
| `input` | نوع عنصر الإدخال: `text`, `number`, `textarea`, `radio`, `checkbox`, `select`. |
| `values` | مصفوفة من الخيارات (للـ `select`, `radio`, `checkbox`). يمكن أن تكون دالة أو مصفوفة ثابتة. |
| `operators` | قائمة بالمشغلات المسموحة لهذا الفلتر (مثل `equal`, `contains`, `between`). |
| `default_operator` | المشغل الافتراضي. |
| `validation` | قواعد التحقق من صحة القيمة: `min`, `max`, `step`, `format` (regex أو تنسيق تاريخ)، `allow_empty_value`. |
| `placeholder` | نص توجيهي داخل حقل الإدخال. |
| `plugin` | اسم إضافة jQuery (مثل `datepicker`, `selectize`). |
| `plugin_config` | إعدادات خاصة بالإضافة. |

**طريقتان لتزويد الفلاتر**:
1. **مباشرة في ملف YAML** (كما في المثال أعلاه).
2. **عبر دالة في الموديل**:
   - دالة عامة: `getQueryBuilderFiltersOptions($columnName)`
   - دالة خاصة باسم الحقل: `get{FieldName}QueryBuilderFiltersOptions($fieldName, $columnName)`

**مثال لدالة في الموديل**:
```php
public function getQueryBuilderFiltersOptions($columnName)
{
    if ($columnName == 'filters') {
        return [
            ['id' => 'status', 'label' => 'الحالة', 'type' => 'integer', 'input' => 'select', 
             'values' => [1 => 'نشط', 0 => 'غير نشط']],
            // ... 
        ];
    }
    if ($columnName == 'rules') {
        return ['condition' => 'AND', 'rules' => []];
    }
    return [];
}
```

---

### 4. دعم الإضافات (Plugins) في الـ FormWidget

يمكن تفعيل مجموعة من الإضافات التي توفرها مكتبة jQuery QueryBuilder مع إعدادات مخصصة:

| اسم الإضافة | الوصف | الخيارات |
|-------------|-------|-----------|
| `sortable` | تمكين السحب والإفلات لإعادة ترتيب القواعد والمجموعات. | `icon`, `inherit_no_sortable` |
| `filter-description` | عرض وصف توضيحي للفلتر (بـ Inline, Popover, أو Bootbox). | `mode`, `icon` |
| `bt-tooltip-errors` | عرض أخطاء التحقق كـ Tooltip من Bootstrap. | `placement` |
| `bt-checkbox` | تنسيق عناصر `checkbox` و `radio` باستخدام Bootstrap. | `color` |
| `unique-filter` | منع استخدام نفس الفلتر أكثر من مرة (عالمياً أو داخل المجموعة). | – |
| `not-group` | إضافة خيار "NOT" لعكس منطق المجموعة. | `icon_checked`, `icon_unchecked` |
| `invert` | زر قلب الشرط (تحويل `=` إلى `!=`، و `AND` إلى `OR`...). | `recursive`, `invert_rules` |

**طريقة التفعيل في YAML**:
```yaml
plugins:
    sortable: { icon: 'bi-sort-down' }
    filter-description: { mode: 'bootbox', icon: 'bi-info-circle' }
    bt-tooltip-errors: { placement: 'top' }
```

---

### 5. مثال متكامل لاستخدام FormWidget وParserHelper

#### 5.1 تعريف الحقل في نموذج (model fields.yaml)
```yaml
fields:
    query_conditions:
        label: شروط التقرير
        type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
        filters:
            name: { id: name, label: الاسم, type: string }
            created_at: { id: created_at, label: تاريخ الإنشاء, type: date, input: datepicker, plugin: datepicker }
        allow_groups: 2
        sort_filters: true
        plugins: { 'sortable': null, 'filter-description': { mode: 'inline' } }
```

#### 5.2 حفظ القواعد من المتحكم (Controller)
عند حفظ النموذج، سيتم تخزين القيمة كـ JSON في العمود المرتبط بالحقل.

#### 5.3 استخدام القواعد في الاستعلام
```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

$reportRules = $model->query_conditions; // JSON string or array
if (!empty($reportRules)) {
    $query = Product::query();
    $query = QueryBuilderParserHelper::parse($query, $reportRules);
    $results = $query->get();
}
```

---

### 6. القيمة المضافة

- **للمطورين**: اختصار كبير في وقت تطوير تقارير وشاشات البحث؛ لم يعد هناك حاجة لكتابة شروط `where` معقدة يدويًا. توحيد طريقة تعريف الفلاتر وإعادة استخدامها عبر المشاريع في بيئة نانوسوفت.
- **للمستخدمين النهائيين**: واجهة بديهية وسهلة لبناء استعلامات مخصصة دون الحاجة إلى معرفة SQL أو بنية البيانات الخلفية.
- **للنظام**: فصل منطق بناء الاستعلام عن واجهة المستخدم، مع إمكانية التوسع بإضافات جديدة وتكامل سلس مع نظام نانوسوفت (ترجمة، صلاحيات، دوال موديل ديناميكية).

---

### 7. الخاتمة

يمثل تحديث `Nano2.QueryBuilder` إضافة نوعية لمنصة نانوسوفت، حيث يزود المطورين بأداة قوية ومرنة لإنشاء شروط استعلام معقدة بطريقة بصرية وآمنة. بفضل الدمج بين مكتبة `jQuery QueryBuilder` القوية من جهة، وكلاس `QueryBuilderParserHelper` الذي يعالج الشروط ويطبقها على `Builder` من جهة أخرى، أصبح بالإمكان بناء أنظمة تقارير وتصفية متقدمة بسرعة وجودة عالية. كما أن التصميم المعياري للـ `FormWidget` يسمح بإعادة استخدام المكون في أي نموذج مع دعم كامل لتخصيص الفلاتر والإضافات.

---

## الوثائق المرجعية

- [توثيق عنصر الواجهة `QueryBuilder`](./docs/querybuilder/Docs-FormWidgets-QueryBuilder-ar.md)
- [توثيق كلاس `QueryBuilderParserHelper`](./docs/querybuilder/Docs-QueryBuilderParserHelper-ar.md)

## 2026-04-07 – 2026-04-10

**تحديثات إضافة `Nano.FileUpload` – الإصدارات 1.0.8 إلى 1.2.0**

### ملخص التحديثات

بعد الإصدارات السابقة التي ركزت على دعم التخزين المتعدد، التحويل التلقائي للصور، وخطافات الأحداث، واصلنا تطوير الإضافة لتعزيز الأمان والمرونة وإضافة ميزات متقدمة لإدارة الملفات. تضمنت التحديثات في الإصدارات 1.0.8 حتى 1.2.0:

- **إضافة نظام اختبار شامل (Testing Suite)** للتحقق من صحة جميع كلاسات الإضافة.
- **إضافة عمود `session_key`** إلى جدول `system_files` لدعم الرفع المؤقت والربط المتأخر.
- **دعم عملية التعديل (Edit)** لاستبدال الملفات في علاقات `attachOne` مع صلاحيات مخصصة.
- **إضافة دوال لإدارة التحديثات على مستوى الحقل** في `FileUploadRegistry`.
- **تحسين أداء وموثوقية عمليات الرفع والحذف** باستخدام المعاملات والأحداث الجديدة.
- **إعادة هيكلة كلاس الاختبارات** إلى `FileUploadPlusTest` بنظام موحد للمخرجات ومعالجة متقدمة للاستثناءات.

---

### الإصدار 1.0.8 – نظام اختبار شامل وتحسين أمان الحذف

#### أهداف الإصدار

- إنشاء كلاس اختبار متكامل (`FileUploadTest`) يغطي جميع كلاسات الإضافة (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`).
- تحسين أمان دالة `deleteFile` بحيث تستخرج `modelClass` و `field` تلقائياً من سجل الملف إذا لم يتم تمريرهما.
- دمج الاختبارات في `Plugin.php` بحيث يمكن تشغيلها عبر معامل `test_fileupload` في بيئة التطوير.

#### الميزات الجديدة

##### 1. كلاس `FileUploadTest` للاختبار الشامل

تم إنشاء كلاس اختبار يقع في `Nano\FileUpload\Tests\FileUploadTest` ويحتوي على دوال لاختبار:

- **FileUploadRegistry**: تسجيل النماذج، الحصول على إعدادات الحقول، التحقق من وجود النموذج، الإعدادات الافتراضية، خيارات المعالجة، تحديث النموذج، التحكم العام في عمليات API، وآلية التخزين المؤقت.
- **FileUploadUserManager**: جلب المستخدم، تحديد نوعه، التحقق من الصلاحيات، تخصيص محلل المستخدم، وتخصيص دالة التحقق من الصلاحيات.
- **FileUploadService**: توليد مفاتيح مؤقتة، التحقق من صحتها، رفع ملف مباشر، رفع base64، رفع متعدد، التحقق من الحجم والنوع والقائمة السوداء، رفع مؤقت، ربط الملفات المؤقتة، جلب الملفات، حذف الملفات، التحجيم التلقائي، العلامة المائية، قرص التخزين المخصص، حساب `hash` و `meta`، صلاحية الملفات المؤقتة، الأحداث، WebSocket، وتعطيل العمليات عالمياً.

يستخدم الاختبار بيئة معزولة (`Storage::fake('local')`) ونموذج وهمي (`TestModel`) لضمان عدم التأثير على البيانات الحقيقية.

##### 2. تحسين أمان دالة `deleteFile`

أصبحت دالة `deleteFile` تتعامل مع ثلاث حالات للتحقق من الصلاحية:

- **الحالة 1:** تم تمرير `modelClass` و `field` – يتم التحقق من الصلاحية باستخدامهما.
- **الحالة 2:** الملف مرتبط بنموذج محفوظ (`attachment_type` و `attachment_id`) – يتم استخراج `modelClass` و `field` من سجل الملف والتحقق من الصلاحية.
- **الحالة 3:** الملف مؤقت (`session_key`) – يتم التحقق من صحة المفتاح وأن المستخدم الحالي هو من أنشأ الملف المؤقت.

هذا يضمن عدم قدرة أي مستخدم على حذف ملف لا يملك صلاحية عليه، حتى لو لم يمرر `modelClass` و `field` بشكل صريح.

##### 3. دمج الاختبارات في `Plugin.php`

أضفنا دالة `testFileUpload()` في `Plugin.php` تُستدعى عند وجود معامل `is_testFileUpload=true` في الطلب. تقوم بإنشاء كائن من `FileUploadTest` وتنفيذ جميع الاختبارات وعرض النتائج. هذا يسمح للمطورين بتشغيل الاختبارات بسهولة عبر المتصفح.

#### الفوائد

- ضمان جودة الكود من خلال اختبار آلي لجميع الوظائف الأساسية.
- اكتشاف الأخطاء مبكراً أثناء التطوير.
- توثيق حي لسلوك الكلاسات من خلال الاختبارات.
- زيادة الثقة في استقرار الإضافة عند إضافة ميزات جديدة.

---

### الإصدار 1.0.9 – دعم `session_key` في جدول الملفات

#### أهداف الإصدار

- إضافة عمود `session_key` إلى جدول `system_files` لدعم الربط المؤقت (deferred binding) بشكل كامل.
- تمكين تصفية الملفات المؤقتة التي تخص جلسة مستخدم معينة.

#### الميزات الجديدة

##### 1. هجرة `add_session_key_to_system_files`

تم إنشاء ملف هجرة يضيف العمود `session_key` من النوع `string` و `nullable` مع فهرس (`index`) بعد عمود `field`. هذا العمود يُستخدم لتخزين مفتاح الجلسة المؤقت عند رفع ملف دون ربطه بنموذج محفوظ.

##### 2. تحديث `FileUploadService::upload`

أثناء الرفع، إذا تم استخدام مفتاح مؤقت (`tempSessionKey`)، يتم تعيين `$file->session_key = $tempSessionKey` قبل حفظ الملف. هذا يضمن إمكانية العثور على الملفات المؤقتة لاحقاً باستخدام هذا المفتاح.

##### 3. تحديث `FileUploadService::attachTempFiles`

تعتمد دالة `attachTempFiles` على عمود `session_key` للبحث عن الملفات المؤقتة المراد ربطها بالنموذج المحفوظ. بعد الربط، يتم مسح `session_key` وتعيين `attachment_id` و `attachment_type`.

##### 4. تحديث `FileUploadService::deleteFile`

في حالة الملف المؤقت، يتم التحقق من `session_key` للتأكد من أن المستخدم الحالي هو من أنشأ الملف قبل السماح بالحذف.

#### الفوائد

- دعم كامل للرفع المؤقت والربط المتأخر (deferred binding).
- إمكانية تنظيف الملفات المؤقتة منتهية الصلاحية عبر مهمة مجدولة باستخدام `expires_at` و `session_key`.
- أمان أعلى: لا يمكن الوصول إلى ملفات مؤقتة خاصة بمستخدم آخر.

---

### الإصدار 1.1.0 – دعم عملية التعديل (Edit) واستبدال الملفات

#### أهداف الإصدار

- تمكين استبدال ملف موجود في علاقة `attachOne` برفع ملف جديد (عملية `edit`).
- إضافة إعداد عام `disable_edit` للتحكم في تفعيل/تعطيل استبدال الملفات على مستوى API.
- إضافة دالة `updateFieldConfig` في `FileUploadRegistry` لتحديث إعدادات حقل معين مع التطبيع ومسح الكاش.
- تحسين `updateModelConfig` لتطبيع النموذج بأكمله بعد الدمج.
- نقل كلاس `Base64` من `Nano.API` إلى داخل الإضافة لتقليل الاعتماديات.
- إضافة أحداث جديدة `beforeEditDelete` و `afterEditDelete` لدورة حياة التعديل.
- تحسين دالة `attachTempFiles` لإرجاع استجابة موحدة.

#### الميزات الجديدة

##### 1. تحديد عملية الرفع (add/edit)

في `FileUploadService::upload`، يتم التحقق مما إذا كان النموذج موجوداً (`exists`) وله علاقة `attachOne` بالحقل. إذا كان هناك ملف مرتبط بالفعل، يتم تعيين `$operation = 'edit'`، وإلا `'add'`.

##### 2. التحقق من الصلاحية لعملية `edit`

أضفنا في `FileUploadRegistry`:

- دالة `isEditEnabledGlobally()` تتحقق من `disable_edit` في الإعدادات العامة.
- دالة `can()` تدعم الآن العملية `'edit'` وتتحقق من الإعداد العام وصلاحية الحقل/النموذج.
- في تعريفات النماذج والحقول الافتراضية، أضفنا مفتاح `'edit' => null` في مصفوفة `permissions`.

##### 3. استخدام المعاملة (Transaction) في عملية `upload`

أصبحت عملية الرفع محاطة بـ `Db::transaction` لضمان التكاملية:

- يتم رفع الملف الجديد وحفظه أولاً.
- إذا كانت العملية `edit` ونجح الرفع، يتم حذف الملف القديم.
- ثم يتم إضافة الملف الجديد إلى علاقة النموذج.
- إذا فشل أي جزء، يتم التراجع عن جميع التغييرات (بما في ذلك حذف الملف القديم).

هذا يمنع فقدان الملف القديم في حالة فشل رفع الملف الجديد.

##### 4. أحداث جديدة لعملية التعديل

- `nano.fileupload.beforeEditDelete`: يُطلق قبل حذف الملف القديم (عند `operation === 'edit'`).
- `nano.fileupload.afterEditDelete`: يُطلق بعد حذف الملف القديم.

هذا يسمح للمطورين بتنفيذ منطق مخصص (مثل إرسال إشعار، تسجيل، أو نسخ احتياطي) قبل وبعد استبدال الملف.

##### 5. دالة `updateFieldConfig` في `FileUploadRegistry`

توفر هذه الدالة وسيلة محدثة لتعديل إعدادات حقل معين دون الحاجة لإعادة تسجيل النموذج بأكمله. تقوم بـ:

- التحقق من وجود النموذج والحقل.
- دمج الإعدادات الجديدة مع الحالية باستخدام `array_replace_recursive`.
- تطبيع الحقل عبر `normalizeFieldDefinition`.
- تحديث `rawDefinitions` ومسح النموذج المبني (`builtModels`).
- مسح الكاش الخاص بهذا الحقل فقط عبر `clearFieldCache`.

##### 6. تحسين `updateModelConfig`

أصبحت الدالة تعيد تطبيع النموذج بأكمله بعد دمج الإعدادات الجديدة، لضمان اكتمال جميع الحقول الافتراضية (مثل `is_public`, `thumb_options`).

##### 7. نقل كلاس `Base64` إلى الإضافة

تم نقل كلاس `Base64` من `Nano\API\Classes\Base64` إلى `Nano\FileUpload\Classes\Base64` مع إضافة دوال متقدمة للتعامل مع أحجام الملفات (`normalizeUploadFileSize`, `getUploadMaxFilesize`, `checkUploadMaxFilesize`). هذا يلغي الاعتماد على `Nano.API` ويجعل الإضافة مستقلة.

##### 8. تحسين `attachTempFiles`

أصبحت الدالة تعيد مصفوفة موحدة (مثل `upload`) تحتوي على `code`, `status`, `message`, `data`, `process_data`, `error`, `debug`، وتقبل خيار `skip_permission_check` لتجاوز الصلاحية في الاختبارات. كما تضيف حدث `afterAttach` وإشعار WebSocket.

#### الفوائد

- إمكانية استبدال الملفات في علاقات `attachOne` بشكل آمن (باستخدام المعاملات).
- تحكم دقيق في صلاحية التعديل عبر إعداد عام أو صلاحيات مخصصة.
- مرونة أكبر في تحديث إعدادات الحقول دون إعادة تسجيل النموذج.
- تقليل الاعتماديات بنقل `Base64` داخل الإضافة.
- توحيد مخرجات `attachTempFiles` مع بقية دوال الخدمة.

---

### الإصدار 1.2.0 – إعادة هيكلة نظام الاختبارات (Testing Overhaul)

#### أهداف الإصدار

- إعادة كتابة كلاس الاختبارات بالكامل إلى `FileUploadPlusTest` بهيكل موحد للمخرجات.
- توحيد مخرجات كل اختبار لتشمل: `code`, `status`, `test_code`, `name`, `description`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`.
- إضافة معالجة متقدمة للاستثناءات بحيث لا يؤدي فشل اختبار واحد إلى إيقاف تنفيذ بقية الاختبارات.
- استخدام المعاملات (`Db::beginTransaction()`, `Db::rollBack()`) في جميع الاختبارات التي تعدل قاعدة البيانات.
- إزالة أي اعتماد على حزم اختبار خارجية (مثل `assert`).
- إضافة اختبارات جديدة لعملية التعديل (`testEditReplaceFile`, `testEditWithoutPermission`, `testEditWithGlobalDisable`, `testEditTriggersEvents`, `testEditKeepsOldFileOnFailure`).

#### الميزات الجديدة

##### 1. توحيد مخرجات الاختبارات

أضفنا دالة `recordResult(array $result)` تقبل مصفوفة بالهيكل المطلوب وتدمجها مع القيم الافتراضية. هذا يضمن أن جميع نتائج الاختبارات متسقة وسهلة المعالجة آلياً.

مثال على مخرجات اختبار ناجح:
```json
{
    "code": 200,
    "status": true,
    "test_code": "UPLOAD_DIRECT_001",
    "name": "رفع ملف مباشر (UploadedFile) مع مفتاح مؤقت",
    "description": "يتحقق من رفع ملف صورة باستخدام كائن UploadedFile...",
    "message": "تم رفع الملف بنجاح",
    "error": null,
    "errors": [],
    "data": { ... },
    "input_data": { ... },
    "process_data": { ... },
    "debug": null
}
```

##### 2. معالجة الاستثناءات

كل اختبار محاط بـ `try-catch`. في حالة حدوث استثناء، يتم تعيين `status` إلى `false`، و `code` إلى `500`، وتخزين رسالة الخطأ في `error`، وتفاصيل التصحيح في `debug` (إذا كان `app.debug` مفعلاً). هذا يضمن استمرار تنفيذ الاختبارات الأخرى.

##### 3. استخدام المعاملات في الاختبارات

جميع الاختبارات التي تُجري تغييرات في قاعدة البيانات (رفع، حذف، تحديث، ربط) تستخدم `Db::beginTransaction()` و `Db::rollBack()` في كتلة `finally`. هذا يضمن عدم ترك أي آثار بعد انتهاء الاختبارات.

##### 4. إضافة اختبارات عملية التعديل

أضفنا 5 اختبارات جديدة لتغطية سيناريوهات استبدال الملفات:

- `testEditReplaceFile`: اختبار نجاح استبدال ملف في علاقة `attachOne`.
- `testEditWithoutPermission`: اختبار رفض الاستبدال عند عدم وجود صلاحية `edit`.
- `testEditWithGlobalDisable`: اختبار رفض الاستبدال عند تعطيل التعديل عالمياً (`disable_edit = true`).
- `testEditTriggersEvents`: اختبار إطلاق الأحداث `beforeEditDelete` و `afterEditDelete`.
- `testEditKeepsOldFileOnFailure`: اختبار عدم حذف الملف القديم عند فشل رفع الملف الجديد (بفضل المعاملة).

##### 5. إعادة هيكلة الاختبارات الحالية

تم إعادة كتابة جميع الاختبارات القديمة (43 اختباراً) لتتوافق مع الهيكل الجديد. كما تم إضافة `test_code` فريد لكل اختبار لتسهيل التعرف عليه.

##### 6. تحديث `FileUploadController::tests()`

أصبح المتحكم يدعم تشغيل كلا الإصدارين من الاختبارات (`FileUploadTest` للإصدار القديم و `FileUploadPlusTest` للإصدار الجديد) عبر معامل `test_version=v1` أو `test_version=v2` (الافتراضي هو `v2`). هذا يسمح بالانتقال التدريجي.

#### الفوائد

- نتائج اختبارات موحدة وسهلة القراءة والفهم.
- إمكانية دمج الاختبارات في أنظمة CI/CD بسهولة.
- عزل كامل للاختبارات عن البيانات الحقيقية.
- تغطية كاملة لعمليات التعديل (edit) التي تمت إضافتها في الإصدار 1.1.0.
- سهولة إضافة اختبارات جديدة في المستقبل باتباع النمط نفسه.

---

### ملخص الإصدارات (1.0.8 – 1.2.0)

| الإصدار | أبرز الميزات |
|---------|---------------|
| 1.0.8 | نظام اختبار شامل (`FileUploadTest`)، تحسين أمان `deleteFile`، دمج الاختبارات في `Plugin.php`. |
| 1.0.9 | إضافة عمود `session_key` إلى `system_files` لدعم الرفع المؤقت الكامل. |
| 1.1.0 | دعم استبدال الملفات (edit)، إضافة `disable_edit`، دالة `updateFieldConfig`، تحسين `updateModelConfig`، نقل `Base64`، أحداث `beforeEditDelete/afterEditDelete`، تحسين `attachTempFiles`. |
| 1.2.0 | إعادة هيكلة نظام الاختبارات إلى `FileUploadPlusTest` بتوحيد المخرجات، معالجة استثناءات متقدمة، استخدام المعاملات، وإضافة اختبارات التعديل. |

---

### متطلبات الترقية

1. **تشغيل الهجرات الجديدة** (لإضافة عمود `session_key`):
   ```bash
   php artisan plugin:refresh Nano.FileUpload
   ```
   أو تنفيذ الهجرة `add_session_key_to_system_files.php` يدوياً.

2. **إضافة متغير البيئة** (اختياري) للتحكم في عملية التعديل:
   ```ini
   NANO_FILE_UPLOAD_DISABLE_EDIT=false
   ```

3. **تحديث تعريفات الحقول** لإضافة صلاحية `edit` إذا لزم الأمر:
   ```php
   'permissions' => [
       'add'    => 'some.permission',
       'edit'   => 'some.edit.permission', // جديد
       'delete' => 'some.delete.permission',
       'view'   => 'some.view.permission',
   ]
   ```

4. **الاستماع للأحداث الجديدة** (اختياري):
   ```php
   Event::listen('nano.fileupload.beforeEditDelete', function ($file, $modelClass, $field, $model) {
       // نسخ احتياطي أو تسجيل
   });
   ```

5. **تشغيل الاختبارات** (في بيئة التطوير):
   - عبر المتصفح: `https://yourdomain.com/api/v1/fileupload/tests?test_version=v2`
   - أو عبر سطر الأوامر (بعد إضافة أمر مخصص).

---

### الخاتمة

بفضل الإصدارات 1.0.8 إلى 1.2.0، أصبحت إضافة `Nano.FileUpload` أكثر نضجاً وموثوقية من أي وقت مضى. يمكنها الآن:

- **اختبار نفسها تلقائياً** من خلال نظام اختبار متكامل ومتوافق مع CI/CD.
- **دعم استبدال الملفات** في علاقات `attachOne` بأمان باستخدام المعاملات والأحداث المخصصة.
- **تحديث إعدادات الحقول** بمرونة دون إعادة تعريف النموذج بأكمله.
- **إدارة الملفات المؤقتة** بشكل كامل عبر عمود `session_key`.
- **تقديم نتائج اختبارات موحدة** لتسهيل التصحيح والتكامل مع الأدوات الخارجية.

هذه التحسينات تجعل الإضافة جاهزة للاستخدام في أكبر المشاريع وأكثرها تعقيداً، مع ضمان الاستقرار والأمان وسهولة الصيانة.

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

## 2026-04-10 – 2026-04-11

**تحديث إضافة `Nano.FileUpload` – الإصدار 1.2.1**

### إعادة هيكلة نظام التحقق من الصلاحيات وإضافة تحكم دقيق على مستوى النموذج والحقل

---

### ملخص التحديثات

في الإصدار **1.2.1**، تم إعادة هيكلة نظام التحقق من الصلاحيات في `FileUploadRegistry` بالكامل، مع إضافة طبقات متعددة للتحقق (عالمي، موديول، حقل) وإتاحة تعطيل عمليات محددة بشكل مستقل. كما تم تحسين `FileUploadService` لاستخدام دوال `validate` الجديدة، وإعادة كتابة دالة `attachTempFiles` بشكل كامل، وتحديث `FileUploadException` برموز خطأ جديدة وكود HTTP مناسب.

---

### أهداف الإصدار

- **توحيد وإعادة هيكلة دوال التحقق** في `FileUploadRegistry` لتصبح أكثر وضوحاً وتنظيماً.
- **إضافة آلية `disabled_operations`** على مستوى النموذج والحقل لتعطيل عمليات محددة (`add`, `edit`, `delete`, `view`) بشكل مستقل.
- **توفير دوال `validate` متخصصة** ترمي استثناءات مناسبة بدلاً من إرجاع `false`، لتبسيط كتابة الكود في طبقة الخدمة.
- **تحسين كود `FileUploadService`** للاستفادة من دوال `validate` الجديدة، مما يقلل من الكود المكرر ويوحد رسائل الأخطاء.
- **إضافة دالة `attachTempFiles` محسّنة بالكامل** مع هيكل استجابة موحد وأحداث WebSocket.
- **تحديث `FileUploadException`** برموز خطأ جديدة وتعديل كود HTTP لبعض الأخطاء لتجنب التعارض.

---

### الميزات الجديدة

#### 1. دوال التحقق المتخصصة في `FileUploadRegistry`

تم إضافة مجموعة من الدوال الجديدة التي تفصل مستويات التحقق:

| الدالة | الوصف |
|--------|-------|
| `canGlobal($operation)` | التحقق من تفعيل العملية عالمياً (عبر الإعدادات العامة `disable_upload`, `disable_edit`, `disable_delete`, `disable_get`). |
| `validateGlobal($operation)` | مثل `canGlobal` ولكن ترمي `FileUploadException` إذا كانت العملية معطلة. |
| `canModel($modelClass, $operation)` | التحقق من تفعيل العملية على مستوى النموذج (من خلال `disabled_operations` في تعريف النموذج). |
| `validateModel($modelClass, $operation)` | مثل `canModel` ولكن ترمي استثناء إذا كانت العملية معطلة. |
| `canField($modelClass, $field, $operation)` | التحقق من تفعيل العملية على مستوى الحقل (من خلال `disabled_operations` في تعريف الحقل). |
| `validateField($modelClass, $field, $operation)` | مثل `canField` ولكن ترمي استثناء. |
| `validate($modelClass, $operation, $userType, $user, $field)` | دالة متكاملة تجمع التحقق من المستوى العام، الموديول، الحقل، نوع المستخدم، والصلاحيات، وترمي الاستثناء المناسب عند أول فشل. |

**مثال استخدام `validate`:**
```php
// في FileUploadService::upload
$this->registry->validate($modelClass, $operation, $userType, $user, $field);
$this->registry->validateField($modelClass, $field, $operation);
```

#### 2. إضافة خاصية `disabled_operations`

أصبح من الممكن الآن تعطيل عمليات محددة على مستوى النموذج أو الحقل باستخدام المفتاح `disabled_operations` في تعريفات التسجيل:

```php
// في تعريف النموذج (Plugin::registerFileUploadFields)
\Nano\Shop\Models\Product::class => [
    'disabled_operations' => ['delete'], // تعطيل الحذف على مستوى المنتج
    'fields' => [
        'image' => [
            'type' => 'image',
            'disabled_operations' => ['edit'], // تعطيل استبدال الصورة فقط
        ],
    ],
];
```

هذا يسمح بمنع عمليات معينة بشكل دقيق دون الحاجة إلى تعطيل الصلاحيات الكاملة أو تعديل الإعدادات العامة.

#### 3. تحسين دالة `can()` في `FileUploadRegistry`

أعيدت كتابة دالة `can()` لتعتمد على سلسلة التحقق الجديدة:

1. التحقق من المستوى العام (`canGlobal`)
2. التحقق من مستوى الموديول (`canModel`)
3. التحقق من مستوى الحقل (`canField`) إن وُجد
4. التحقق من نوع المستخدم (`isUserTypeAllowed`)
5. التحقق من الصلاحيات المحددة (`permissions`)

هذا يضمن عدم السماح بأي عملية يتم تعطيلها في أي مستوى.

#### 4. رموز خطأ جديدة في `FileUploadException`

أضيفت الثوابت التالية:

- `ERR_EDIT_DISABLED_GLOBALLY` – عندما يكون التعديل معطلاً عالمياً (`disable_edit = true`)
- `ERR_MODEL_OPERATION_DISABLED` – عندما تكون العملية معطلة على مستوى النموذج
- `ERR_FIELD_OPERATION_DISABLED` – عندما تكون العملية معطلة على مستوى الحقل

**تغيير كود HTTP:**
- `ERR_PERMISSION_DENIED` تم تغيير كود HTTP من `403` إلى `422` (Unprocessable Entity) لتجنب التعارض مع حالة "حساب غير نشط" التي تستخدم `403`.
- `ERR_UPLOAD_DISABLED_GLOBALLY`, `ERR_EDIT_DISABLED_GLOBALLY`, `ERR_DELETE_DISABLED_GLOBALLY`, `ERR_GET_DISABLED_GLOBALLY` تستخدم كود `503` (Service Unavailable).
- `ERR_MODEL_OPERATION_DISABLED`, `ERR_FIELD_OPERATION_DISABLED` تستخدم كود `423` (Locked).

#### 5. تحسين دالة `upload` في `FileUploadService`

تم استبدال الكود اليدوي للتحقق من الصلاحيات باستدعاءات `validate`:

**قبل:**
```php
if (!$this->registry->can($modelClass, $operation, $userType, $user, $field)) {
    throw new FileUploadException(...);
}
```

**بعد:**
```php
$this->registry->validate($modelClass, $operation, $userType, $user, $field);
$this->registry->validateField($modelClass, $field, $operation);
```

كما تم إضافة خيار `skip_permission_check` لتجاوز التحقق من الصلاحيات (مفيد في الاختبارات الآلية).

#### 6. إعادة كتابة دالة `attachTempFiles` بالكامل

أصبحت الدالة تعيد هيكل استجابة موحد يحتوي على `code`, `status`, `message`, `data`, `error_code`, `process_data`, `debug`. وتقوم بـ:

- التحقق من تسجيل النموذج والحقل في `FileUploadRegistry`.
- التحقق من الصلاحيات (باستخدام عملية `edit` لأن الربط يعد تعديلاً على النموذج).
- التحقق من صحة المفتاح المؤقت باستخدام `validateTempSessionKey`.
- التحقق من تطابق `modelClass` و `field` مع بيانات المفتاح.
- التحقق من أن المستخدم الحالي هو من أنشأ الملفات المؤقتة (مقارنة `userId` و `userType`).
- البحث عن الملفات المؤقتة (`session_key`).
- ربط الملفات بالنموذج (تعيين `attachment_id`, `attachment_type`, وإلغاء `session_key`, `expires_at`).
- إضافة الملفات إلى علاقة النموذج (إن وجدت).
- إطلاق حدث `nano.fileupload.afterAttach`.
- إرسال إشعار WebSocket.

**مثال الاستجابة الناجحة:**
```json
{
    "code": 200,
    "status": true,
    "message": "تم ربط 3 ملفات بنجاح",
    "data": [
        {"id": 10, "title": "image.jpg", "path": "/storage/...", "size": 1024, "content_type": "image/jpeg"}
    ],
    "process_data": {
        "field_config": {...},
        "key_data": {...},
        "attached_count": 3,
        "temp_session_key": "tmp_xxx",
        "user_type": "backend"
    }
}
```

#### 7. تحسين دالة `getFiles`

تم تحسين استخراج `morphClass` باستخدام `app($modelClass)->getMorphClass()` بدلاً من `$modelClass::getMorphClass()` لتفادي الأخطاء في بعض السياقات (مثل عندما لا يكون النموذج محملاً مسبقاً). كما تم توحيد التحقق من صلاحية `view` باستخدام `$this->registry->validate($modelClass, 'view', ...)`.

#### 8. تحسين دالة `deleteFile`

تمت إعادة هيكلة منطق التحقق من الصلاحية ليشمل أربع حالات:

| الحالة | الوصف | طريقة التحقق |
|--------|-------|---------------|
| 1 | تم تمرير `modelClass` و `field` | استخدام `registry->can()` مع المعطيات الممررة |
| 2 | الملف مرتبط بنموذج محفوظ (`attachment_type`, `attachment_id`, `field`) | استخراج القيم من الملف واستخدام `registry->can()` |
| 3 | الملف مؤقت (`session_key`) | التحقق من صحة المفتاح ومطابقة المستخدم |
| 4 | لا توجد معلومات كافية | رفض الحذف مباشرة |

**التحسين الخاص بالملفات المؤقتة:**
```php
elseif ($file->session_key) {
    $keyData = $this->validateTempSessionKey($file->session_key);
    if (!$keyData) throw ...;
    $currentUserId = $user->getKey() ?? 'guest';
    if ($keyData['userId'] != $currentUserId) throw ...;
    $usedModelClass = $keyData['modelClass'];
    $usedField = $keyData['field'];
    $isAuthorized = true;
}
```

#### 9. تحسين دالة `logFailedAttempt`

تم تحسين استخراج `userId` لدعم أنواع متعددة من كائنات المستخدم:

```php
$userId = 'guest';
if ($user) {
    if (method_exists($user, 'getKey')) $userId = $user->getKey();
    elseif (property_exists($user, 'id')) $userId = $user->id;
    elseif (method_exists($user, 'getId')) $userId = $user->getId();
    else $userId = 'unknown';
}
```

هذا يقلل الأخطاء عند وجود أنواع مختلفة من المستخدمين (backend, frontend, مخصص).

#### 10. إضافة خيار `skip_permission_check` في دوال متعددة

أضيف خيار `skip_permission_check` في دوال `upload`, `uploadMultiple`, `attachTempFiles` لتجاوز التحقق من الصلاحيات بالكامل. هذا الخيار مفيد جداً في بيئات الاختبار الآلي (مثل `FileUploadPlusTest`) حيث لا يكون هناك مستخدم حقيقي أو صلاحيات محددة.

**مثال الاستخدام:**
```php
$result = $service->upload($modelClass, $field, $file, [
    'skip_permission_check' => true
]);
```

---

### الفوائد والقيمة المضافة

- **كود أكثر نظافة**: دوال `validate` تلغي الحاجة إلى كتابة كتل شرطية متكررة للتحقق من الصلاحيات.
- **تحكم دقيق**: `disabled_operations` يسمح بتعطيل عمليات معينة على مستوى النموذج أو الحقل دون المساس بالصلاحيات الكلية أو الإعدادات العامة.
- **توافق أفضل مع أنظمة المصادقة**: تغيير كود HTTP لـ `ERR_PERMISSION_DENIED` من `403` إلى `422` يتجنب الخلط بين "غير مصرح" و "حساب غير نشط".
- **دالة `attachTempFiles` احترافية**: أصبحت الدالة كاملة ومتكاملة مع هيكل الاستجابة الموحد والأحداث وإشعارات WebSocket.
- **مرونة الاختبارات**: خيار `skip_permission_check` يسهل كتابة اختبارات وحدة لا تعتمد على وجود مستخدم حقيقي.
- **أمان محسن**: التحقق من الصلاحية في `deleteFile` يغطي جميع الحالات الممكنة، مما يمنع الحذف غير المصرح به.

---

### التغييرات التي قد تؤثر على التوافق

1. **تغيير كود HTTP لـ `ERR_PERMISSION_DENIED`**:
   - إذا كان لديك كود يعتمد على استلام `403` لهذا الخطأ، فسيتلقى الآن `422`. يُنصح بتحديث أي معالجة للخطأ تعتمد على الكود.

2. **دالة `attachTempFiles` أصبحت تعيد هيكلاً مختلفاً**:
   - كانت تعيد سابقاً `true/false` أو مصفوفة بسيطة. الآن تعيد مصفوفة موحدة تحتوي على `code`, `status`, `message`, `data`, إلخ. إذا كنت تستخدم هذه الدالة مباشرة، فستحتاج إلى تحديث الكود للتعامل مع الهيكل الجديد.

3. **إضافة `disabled_operations`**:
   - لا تؤثر على الإعدادات الحالية ما لم تقم بإضافتها صراحة. الإعدادات الافتراضية تسمح بجميع العمليات.

4. **دوال `validate` الجديدة**:
   - تم استخدامها داخل `FileUploadService`، ولكن إذا كنت تستدعي `can()` مباشرة، فستظل تعمل كما هي (مع تحسين السلسلة الجديدة).

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال `FileUploadRegistry.php` بالنسخة التي تحتوي على دوال `validate` الجديدة وخاصية `disabled_operations`.
   - استبدال `FileUploadService.php` بالنسخة المحسّنة التي تستخدم `validate` وتحتوي على `attachTempFiles` الجديدة.
   - استبدال `FileUploadException.php` بالنسخة التي تحتوي على رموز الخطأ الجديدة.

2. **لا توجد هجرات جديدة**:
   - هذا الإصدار لا يتطلب تغييرات في قاعدة البيانات أو إضافة أعمدة جديدة.

3. **لا توجد تغييرات في الإعدادات**:
   - يظل ملف `config.php` كما هو. يمكن استخدام متغيرات البيئة الموجودة (`disable_upload`, `disable_edit`, `disable_delete`, `disable_get`) بنفس الطريقة.

4. **مراجعة تعريفات النماذج** (اختياري):
   - إذا كنت ترغب في استخدام `disabled_operations`، أضف المفتاح إلى تعريفات النماذج أو الحقول كما هو موضح في الأمثلة أعلاه.

5. **اختبار التوافق**:
   - يُنصح بتشغيل اختبارات `FileUploadPlusTest` (الإصدار 1.2.0) للتأكد من أن جميع العمليات تعمل بشكل صحيح مع التغييرات الجديدة.
   - التحقق من أن صلاحيات `edit` تعمل كما هو متوقع عند استبدال الملفات (في حال استخدامها).

6. **تحديث أي كود يعتمد على `attachTempFiles`**:
   - إذا كنت تستدعي `attachTempFiles` مباشرة، فستحتاج إلى تعديل الكود ليتعامل مع هيكل الاستجابة الجديد.

---

### الخاتمة

يمثل الإصدار **1.2.1** نقلة نوعية في نضج إدارة الصلاحيات داخل إضافة `Nano.FileUpload`. من خلال إضافة دوال `validate` المتخصصة، وخاصية `disabled_operations`، وتحسين دالتَي `attachTempFiles` و `deleteFile`، أصبح النظام أكثر تنظيماً وأماناً ومرونة. هذه التغييرات تجعل الإضافة قادرة على التعامل مع سيناريوهات معقدة مثل استبدال الملفات، وتعطيل عمليات معينة مؤقتاً، والتحقق من الصلاحيات متعددة المستويات، مع الحفاظ على التوافق مع الإصدارات السابقة قدر الإمكان.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)

## 2026-04-12 – 2026-04-14

**تحديثات إضافة `Nano.FileUpload` – الإصدار 1.2.2**

### ملخص التحديثات

في إطار السعي المستمر لتعزيز أمان ومرونة نظام رفع الملفات، تم إصدار الإصدار 1.2.2 الذي يركز على توحيد وتبسيط عملية التحقق من مفاتيح الجلسة المؤقتة (temp session keys) عبر جميع دوال الخدمة. تم إنشاء تريت (Trait) جديد باسم `HasFileUploadsMatchTempKey` يحتوي على دالة متطورة `validateAndMatchTempKey` تجمع التحقق من صحة المفتاح، ومطابقة النموذج والحقل والمستخدم، مع خيارات متقدمة للتحكم في السلوك (الوضع الصارم، فترة السماح، التخزين المؤقت، إلخ). كما تمت إضافة دالة مساعدة `fireEventSafeInTrait` لضمان توافق إطلاق الأحداث مع كل من Laravel و OctoberCMS.

تم تحديث دوال `upload`، `deleteFile`، `attachTempFiles`، و `getFiles` في `FileUploadService` لاستخدام هذه الدالة الجديدة بدلاً من الكود المكرر، مما أدى إلى تقليل التعقيد وزيادة الأمان والتوحيد.

---

### أهداف الإصدار

- **توحيد منطق التحقق من المفاتيح المؤقتة**: إنشاء نقطة مركزية واحدة للتحقق من صحة المفاتيح ومطابقتها مع النموذج والحقل والمستخدم.
- **إزالة الكود المكرر**: استبدال عمليات التحقق اليدوية المتكررة في عدة دوال باستدعاء دالة واحدة.
- **تعزيز الأمان**: إضافة خيارات متقدمة مثل الوضع الصارم (strict mode)، التحقق من صلاحية المفتاح مع فترة سماح، ودعم إعادة المحاولة عند انتهاء الصلاحية.
- **تحسين التوافق**: ضمان إطلاق الأحداث بشكل صحيح في بيئات Laravel و OctoberCMS عبر دالة `fireEventSafeInTrait`.
- **تسهيل الاختبار**: توفير خيارات مثل `skip_user_check` و `cache_results` لتسهيل اختبار الوحدة.

---

### الميزات الجديدة

#### 1. تريت `HasFileUploadsMatchTempKey`

تم إنشاء التريت في المسار `Nano\FileUpload\Classes\FileUploadService\HasFileUploadsMatchTempKey` ويحتوي على:

##### أ. دالة `validateAndMatchTempKey`

هذه الدالة هي جوهر التحديث، وتقوم بالتحقق من صحة المفتاح المؤقت ومطابقته مع البيانات المتوقعة. توفر الدالة الخيارات التالية:

| الخيار | النوع | الوصف |
|--------|-------|-------|
| `throw_on_failure` | bool | إطلاق استثناء عند الفشل (true) أو إرجاع نتيجة خطأ كمصفوفة (false) |
| `skip_user_check` | bool | تخطي التحقق من المستخدم (للاستخدام الداخلي أو الاختبارات) |
| `strict_mode` | bool | وضع صارم: يتطلب تطابق النموذج والحقل بالكامل (true)، أو يسمح بعدم التطابق (false) |
| `stop_on_first_failure` | bool | إيقاف عند أول خطأ (true) أو تجميع الأخطاء (false) |
| `allow_expired_key` | bool | السماح بالمفاتيح منتهية الصلاحية ضمن فترة سماح |
| `expiry_grace_period` | int | فترة السماح بالثواني (افتراضي 300) |
| `custom_validator` | callable | دالة تحقق إضافية (تستقبل `$keyData` وتعيد bool) |
| `cache_results` | bool | تخزين نتائج التحقق مؤقتاً في نفس الطلب |
| `collect_metadata` | bool | جمع بيانات أداء (مدة التنفيذ، الخطوات المنفذة) |

**مثال على الاستدعاء الأساسي:**
```php
$keyData = $this->validateAndMatchTempKey(
    $tempSessionKey,
    $modelClass,
    $field,
    $user,
    ['throw_on_failure' => true, 'strict_mode' => true]
);
```

##### ب. دالة `manualValidateTempSessionKeyWithGrace`

دالة مساعدة داخلية تُستخدم عند تفعيل خيار `allow_expired_key`، وتقوم بالتحقق من المفتاح المنتهي مع فترة سماح دون تعديل الدالة الأصلية `validateTempSessionKey`.

##### ج. دالة `fireEventSafeInTrait`

دالة مساعدة لإطلاق الأحداث بشكل آمن ومتوافق مع كل من Laravel و OctoberCMS. تتحقق أولاً مما إذا كان الكائن المستخدم (مثل `FileUploadService`) يمتلك دالة `fireEventSafe`، وتستخدمها إن وجدت، وإلا تطبق منطقاً مكافئاً يدعم `Event::fire` و `Event::dispatch` و `event()` helper.

#### 2. تحديث دالة `upload` في `FileUploadService`

- **قبل التحديث**: كان يتم قبول أي مفتاح مؤقت يتم تمريره من العميل دون التحقق من صحته أو مطابقته للنموذج والحقل.
- **بعد التحديث**: عند وجود `temp_session_key` في الخيارات، يتم استدعاء `validateAndMatchTempKey` للتحقق من أن المفتاح يخص نفس النموذج والحقل والمستخدم، وإلا يتم رمي استثناء مناسب.

**الكود الجديد:**
```php
} elseif (isset($options['temp_session_key']) && !empty($options['temp_session_key'])) {
    $tempSessionKey = $options['temp_session_key'];
    $user = $this->userManager->getUser();
    $this->validateAndMatchTempKey($tempSessionKey, $modelClass, $field, $user, [
        'throw_on_failure' => true,
        'strict_mode' => true,
        'skip_user_check' => false,
    ]);
}
```

#### 3. تحديث دالة `deleteFile` في `FileUploadService`

- **قبل التحديث**: كان يتم التحقق يدوياً من صحة المفتاح ثم مقارنة `userId` بشكل منفصل.
- **بعد التحديث**: استخدام `validateAndMatchTempKey` مع `strict_mode = false` (لأن الملف المؤقت قد لا يكون له `attachment_type`) لتوحيد المنطق.

**الكود الجديد:**
```php
elseif ($file->session_key) {
    $keyData = $this->validateAndMatchTempKey(
        $file->session_key,
        $file->attachment_type ?: '',
        $file->field,
        $user,
        ['throw_on_failure' => true, 'strict_mode' => false, 'skip_user_check' => false]
    );
    $usedModelClass = $keyData['modelClass'];
    $usedField = $keyData['field'];
    $isAuthorized = true;
}
```

#### 4. تحديث دالة `attachTempFiles` في `FileUploadService`

- **قبل التحديث**: كان يتم استدعاء `validateTempSessionKey` ثم مقارنة `modelClass` و `field` و `userId` و `userType` بشكل منفصل (أكثر من 20 سطراً من الكود).
- **بعد التحديث**: استبدال كل هذا المنطق باستدعاء واحد للدالة `validateAndMatchTempKey`.

**الكود الجديد:**
```php
$keyData = $this->validateAndMatchTempKey(
    $tempSessionKey,
    $modelClass,
    $field,
    $user,
    ['throw_on_failure' => true, 'strict_mode' => true, 'skip_user_check' => false]
);
$process_data['key_data'] = $keyData;
```

#### 5. تحديث دالة `getFiles` في `FileUploadService`

- **قبل التحديث**: كان يتم التحقق يدوياً من صحة المفتاح ثم مقارنة `userId` و `userType` و `modelClass` و `field` بشكل منفصل.
- **بعد التحديث**: استخدام `validateAndMatchTempKey` للتحقق من المفتاح قبل استخدامه في الاستعلام.

**الكود الجديد:**
```php
} elseif (isset($options['temp_session_key'])) {
    $user = $this->userManager->getUser();
    $this->validateAndMatchTempKey(
        $options['temp_session_key'],
        $modelClass,
        $field,
        $user,
        ['throw_on_failure' => true, 'strict_mode' => true, 'skip_user_check' => false]
    );
    $query->where('session_key', $options['temp_session_key']);
}
```

#### 6. تحسين إطلاق الأحداث في التريت

تم استبدال جميع استدعاءات `Event::fire` المباشرة داخل `validateAndMatchTempKey` باستدعاءات `$this->fireEventSafeInTrait`، مما يضمن توافق الأحداث مع بيئات Laravel و OctoberCMS، ويتيح الاستفادة من دالة `fireEventSafe` الموجودة في `FileUploadService` إن وجدت.

---

### الفوائد

- **تقليل الكود المكرر**: إزالة أكثر من 50 سطراً من الكود المتكرر من الدوال المختلفة.
- **زيادة الأمان**: التحقق الموحد يضمن عدم وجود ثغرات ناتجة عن نسيان التحقق من مطابقة النموذج أو المستخدم في أي دالة.
- **مرونة عالية**: الخيارات المتعددة في `validateAndMatchTempKey` تسمح بتخصيص السلوك حسب الحالة (وضع صارم، فترة سماح، تخزين مؤقت، إلخ).
- **توافق محسن**: دالة `fireEventSafeInTrait` تضمن عمل الأحداث في كافة البيئات.
- **سهولة الاختبار**: خيار `skip_user_check` و `cache_results` و `collect_metadata` يسهل اختبار الوحدة وتحليل الأداء.
- **توثيق مدمج**: الدالة الجديدة تحتوي على توثيق شامل لجميع الخيارات.

---

### متطلبات الترقية

1. **تحديث الكود**: استبدال ملف `FileUploadService.php` بالنسخة المحدثة التي تستخدم التريت الجديد.
2. **إضافة التريت**: إنشاء ملف `HasFileUploadsMatchTempKey.php` في المسار `classes/FileUploadService/` بالمحتوى المطلوب.
3. **لا توجد هجرات جديدة**: هذا الإصدار لا يتطلب تغييرات في قاعدة البيانات.
4. **لا توجد تغييرات في الإعدادات**: يظل ملف `config.php` كما هو.
5. **اختبار التوافق**: يُنصح بتشغيل اختبارات `FileUploadPlusTest` للتأكد من أن جميع العمليات تعمل بشكل صحيح.

---

### الخاتمة

يمثل الإصدار 1.2.2 خطوة مهمة نحو توحيد وتبسيط منطق التحقق من المفاتيح المؤقتة في نظام رفع الملفات. من خلال إدخال تريت `HasFileUploadsMatchTempKey` ودالة `validateAndMatchTempKey`، أصبح من الممكن الآن تنفيذ تحقق آمن وشامل من المفاتيح المؤقتة في أي دالة تحتاج إليها، مع خيارات متقدمة تتيح التحكم الدقيق في السلوك. التحديثات التي تم إجراؤها على دوال `upload`, `deleteFile`, `attachTempFiles`, `getFiles` تجعل الكود أكثر نظافة وأقل عرضة للأخطاء، مع الحفاظ على التوافق الكامل مع الإصدارات السابقة.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)


## 2026-04-15 – 2026-04-16

**تحديث إضافة `Nano.FileUpload` – الإصدار 1.2.3**

### إضافة نقاط نهاية API جديدة للاستعلام عن الموديولات والصلاحيات والتحقق من المفاتيح المؤقتة

---

### ملخص التحديثات

في الإصدار **1.2.3**، تم إثراء واجهة برمجة التطبيقات (API) للإضافة بعدة نقاط نهاية جديدة تتيح للمطورين والأنظمة الخارجية:

- **الاستعلام عن الموديولات المسجلة** وحقولها مع مراعاة صلاحيات المستخدم.
- **جلب الإعدادات التفصيلية** لموديول معين أو حقل معين.
- **جلب قيود الحقل** (مثل الحجم الأقصى والأنواع المسموحة).
- **جلب خيارات المعالجة المتقدمة** (قرص التخزين، التحجيم التلقائي، العلامة المائية).
- **التحقق من الصلاحيات** على المستوى العالمي، مستوى الموديول، ومستوى الحقل.
- **التحقق المتكامل من الصلاحية** باستخدام دالة `validate()`.
- **التحقق من صحة المفتاح المؤقت ومطابقته** عبر واجهة API (باستخدام دالة `validateAndMatchTempKey` من التريت).

جميع نقاط النهاية الجديدة محمية بـ `oauth-users` وتتبع هيكل الاستجابة الموحد (`code`, `status`, `message`, `data`, ...).

---

### أهداف الإصدار

- **تمكين المطورين من استكشاف الإعدادات المسجلة** في `FileUploadRegistry` دون الحاجة للوصول المباشر إلى الكود أو قاعدة البيانات.
- **تسهيل دمج الواجهات الأمامية** (مثل تطبيقات React أو Vue) من خلال توفير معلومات ديناميكية حول الحقول المسموحة والقيود.
- **التحقق من الصلاحيات قبل محاولة الرفع** لتجنب أخطاء الرفض وتحسين تجربة المستخدم.
- **توفير نقطة نهاية للتحقق من صحة المفاتيح المؤقتة** مما يسمح للواجهات الأمامية بالتأكد من صلاحية المفتاح قبل استخدامه.
- **توحيد طريقة الاستعلام** عن معلومات النظام عبر API بدلاً من الاعتماد على أدوات داخلية.

---

### نقاط النهاية الجديدة

#### 1. جلب جميع الموديولات المسجلة (مع تصفية حسب الصلاحية)

- **المسار:** `GET /api/v1/fileupload/models`
- **الوصف:** يعيد قائمة الموديولات المسجلة التي يملك المستخدم الحالي صلاحية `view` عليها، مع معلومات مختصرة عن حقولها (الاسم، النوع، هل هو متعدد).
- **الاستجابة:** مصفوفة من الكائنات تحتوي على `class`, `label`, `enabled`, `allowed_user_types`, `fields`.

#### 2. جلب إعدادات موديول معين

- **المسار:** `GET /api/v1/fileupload/models/{modelClass}`
- **الوصف:** يعيد الإعدادات الكاملة لموديول معين (بعد التحقق من وجوده وصلاحية `view`).
- **الاستجابة:** تحتوي على `enabled`, `label`, `allowed_user_types`, `disabled_operations`, و `fields` (مع معلومات مختصرة لكل حقل).

#### 3. جلب إعدادات حقل معين

- **المسار:** `GET /api/v1/fileupload/models/{modelClass}/fields/{field}`
- **الوصف:** يعيد إعدادات حقل معين (مثل النوع، الحجم الأقصى، الأنواع المسموحة).
- **الاستجابة:** تشمل `type`, `label`, `multiple`, `required`, `max_filesize`, `allowed_types`, `use_caption`, `disabled_operations`.

#### 4. جلب قيود الحقل

- **المسار:** `GET /api/v1/fileupload/models/{modelClass}/fields/{field}/constraints`
- **الوصف:** يعيد قيود الحقل المستخدمة في عملية التحقق من صحة الملف (`max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`).
- **الفائدة:** يمكن للواجهات الأمامية تطبيق نفس القيود قبل رفع الملف.

#### 5. جلب خيارات المعالجة المتقدمة لحقل معين

- **المسار:** `GET /api/v1/fileupload/processing-options/{modelClass}/{field}`
- **الوصف:** يعيد خيارات المعالجة الخاصة بالحقل: `storage_disk`, `auto_resize`, `resize_options`, `auto_watermark`, `watermark_options`.
- **الفائدة:** يستخدم بشكل أساسي من قبل الواجهات المتقدمة التي تحتاج إلى معرفة إعدادات التحويل التلقائي.

#### 6. التحقق من تفعيل عملية معينة عالمياً

- **المسار:** `GET /api/v1/fileupload/permissions/global/{operation}`
- **المعاملات:** `operation` يمكن أن يكون `add`, `edit`, `delete`, `view`.
- **الاستجابة:** تعيد `allowed` (true/false) و `disabled_globally` (عكس `allowed`).

#### 7. التحقق من صلاحية عملية على مستوى موديول معين

- **المسار:** `GET /api/v1/fileupload/permissions/model/{modelClass}/{operation}`
- **الاستجابة:** تعيد `model_operation_enabled`, `user_type_allowed`, `can_proceed` (مجموع الشرطين).

#### 8. التحقق من صلاحية عملية على مستوى حقل معين

- **المسار:** `GET /api/v1/fileupload/permissions/field/{modelClass}/{field}/{operation}`
- **الاستجابة:** تعيد `field_operation_enabled`, `user_has_full_permission`, `can_proceed`.

#### 9. التحقق المتكامل من الصلاحية (باستخدام `validate`)

- **المسار:** `POST /api/v1/fileupload/permissions/check`
- **البيانات (JSON):**
  ```json
  {
    "model_class": "Nano\\Shop\\Models\\Product",
    "operation": "edit",
    "field": "image"
  }
  ```
- **الاستجابة:** تعيد `allowed` (true/false) ورسالة توضيحية.
- **السلوك:** يستخدم دالة `$this->registry->validate()` التي تتحقق من المستوى العالمي، الموديول، الحقل، نوع المستخدم، والصلاحيات المحددة، وتعيد النتيجة دون رمي استثناء (يتم التقاط الاستثناء وتحويله إلى استجابة).

#### 10. التحقق من صحة مفتاح مؤقت ومطابقته

- **المسار:** `POST /api/v1/fileupload/temp-key/validate`
- **البيانات (JSON):**
  ```json
  {
    "temp_key": "tmp_xxxx:yyyy",
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "strict_mode": true,
    "allow_expired_key": false,
    "expiry_grace_period": 300
  }
  ```
- **الوصف:** يستدعي دالة `validateAndMatchTempKey` من التريت `HasFileUploadsMatchTempKey` ويعيد النتيجة بشكل JSON.
- **الاستجابة:** في حالة النجاح تعيد `valid: true` و `key_data` و `matched_data`، وفي حالة الفشل تعيد رسالة الخطأ مع `error_code`.

---

### أمثلة على الاستخدام

#### جلب الموديولات المتاحة للمستخدم الحالي

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/models" \
  -H "Authorization: Bearer <token>"
```

#### جلب إعدادات حقل `image` في موديول `Product`

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/models/Nano%5CShop%5CModels%5CProduct/fields/image" \
  -H "Authorization: Bearer <token>"
```

#### التحقق من صلاحية `edit` على حقل معين

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/permissions/field/Nano%5CShop%5CModels%5CProduct/image/edit" \
  -H "Authorization: Bearer <token>"
```

#### التحقق من صحة مفتاح مؤقت

```bash
curl -X POST "https://yourdomain.com/api/v1/fileupload/temp-key/validate" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "temp_key": "tmp_eyJtb2RlbENsYXNzIjoiTmFub1xTaG9wXE1vZGVsc1xQcm9kdWN0Iiw...",
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "strict_mode": true
  }'
```

---

### الفوائد والقيمة المضافة

- **تمكين الواجهات الأمامية**: يمكن الآن للتطبيقات الأمامية (مثل Single Page Applications) معرفة ديناميكياً الحقول المتاحة والقيود والصلاحيات، مما يسمح ببناء نماذج رفع ملفات تكيفية.
- **تقليل الاعتماد على الخادم**: يمكن للعميل التحقق من صلاحياته قبل محاولة الرفع، مما يقلل من طلبات الرفض وتحسين تجربة المستخدم.
- **توثيق حي للإعدادات**: المطورون يمكنهم استكشاف إعدادات الإضافة عبر API بدلاً من البحث في الكود.
- **أمان إضافي**: نقاط نهاية الصلاحيات لا تعرض معلومات حساسة (مثل مفاتيح التخزين) وتخضع لنفس صلاحيات المستخدم.
- **تكامل سهل مع أدوات خارجية**: يمكن استخدام هذه النقاط في أنظمة CI/CD أو أدوات إدارة المحتوى.

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال `FileUploadController.php` بالنسخة التي تحتوي على الدوال الجديدة.
   - تحديث `routes.php` بإضافة المسارات الجديدة.

2. **لا توجد هجرات جديدة**:
   - لا يتطلب هذا الإصدار تغييرات في قاعدة البيانات.

3. **لا توجد تغييرات في الإعدادات**:
   - يظل ملف `config.php` كما هو.

4. **صلاحيات API**:
   - جميع النقاط الجديدة محمية بـ `oauth-users`، لذا يجب أن يمتلك المستخدم توكن صالح للوصول إليها.

5. **اختبار التوافق**:
   - يُنصح بتشغيل اختبارات `FileUploadPlusTest` للتأكد من أن الإصدار الجديد لا يؤثر على الوظائف الحالية.
   - يمكن اختبار النقاط الجديدة باستخدام أدوات مثل Postman أو cURL.

---

### الخاتمة

يمثل الإصدار **1.2.3** نقلة نوعية في إمكانية الوصول إلى معلومات نظام رفع الملفات عبر API. من خلال إضافة نقاط نهاية للاستعلام عن الموديولات والحقول والصلاحيات والتحقق من المفاتيح المؤقتة، أصبحت الإضافة أكثر انفتاحاً وتكاملاً مع الأنظمة الخارجية والواجهات الحديثة. هذه الميزات تجعل من السهل بناء تطبيقات غنية تعتمد على `Nano.FileUpload` كخلفية لإدارة الملفات، مع الحفاظ على أعلى مستويات الأمان والتحكم.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)


## 2026-04-16 - 2026-04-18

**تحديثات إضافة `Nano.TagsApi` – الإصدار 1.0.10**

### ملخص التحديثات

في هذا الإصدار، تم إجراء تحسينات شاملة على إدارة `TagsApi` لتكون أكثر مرونة وتكاملاً مع الإعدادات العامة للنظام، مع دعم خيارات جديدة في `config.php`، وتحسين متحكمات `Categories`، `Tags`، `Types`، و`Menus`. تم أيضاً إضافة أحداث جديدة لتوسيع الاستعلامات، وتحسين آلية الصفحات (pagination) والترتيب (ordering)، ودعم الفلاتر المتقدمة مثل `is_has_products`، `is_has_shops`، و`is_has_cateables` في `Categories`.

تشمل أبرز التحديثات:

- **إضافة خيارات إعدادات جديدة** في `config.php` (مثل `include`, `exclude`, `per_page`, `is_public`) لكل الأقسام.
- **تحسين متحكم `Categories`** لدعم `mainCategoriesId`، `is_has_*` مع `leftJoin` معقد، و`is_to_sql` للتصحيح.
- **إضافة أحداث `extendQueryBefore` و `extendQuery`** في جميع المتحكمات.
- **دعم قراءة `is_public`، `per_page`، `page` من الإعدادات** مع إمكانية التجاوز عبر `Input::get()`.
- **إضافة خيار `is_departments`** في `Types` للتحكم في فلترة الأقسام.
- **إضافة خيار `is_stop_order_type` و `is_stop_order_type_departments`** للتحكم في دمج `OrderHelper`.
- **تحسين دوال `availablefilters` و `formfields`** بإضافة `input_data` و `process_data` في الردود.

---

### الإصدار 1.0.10 – تحسين شامل للإعدادات والمتحكمات

#### أهداف الإصدار

- جعل جميع المتحكمات تعتمد على الإعدادات المخزنة في `config.php` بدلاً من القيم الثابتة أو `Input::get()` فقط.
- إضافة مرونة أكبر في فلترة التصنيفات والفئات والوسوم بناءً على العلاقات (مثل المنتجات، المتاجر، الكائنات المرتبطة).
- تحسين أداء الاستعلامات المعقدة في `Categories` عبر استخدام `leftJoin` مع `nano_tags_cateables`.
- توفير أدوات تصحيح (`is_to_sql`) لتسهيل تطوير الاستعلامات.
- توحيد طريقة التعامل مع `page` و `per_page` عبر جميع المتحكمات.

#### الميزات الجديدة

##### 1. توسيع ملف الإعدادات `config.php`

تم إضافة مفاتيح جديدة لجميع الأقسام (`types`, `tags`, `categories`, `menus`):

| المفتاح | الوصف | مثال القيمة |
|--------|------|-------------|
| `include` | العلاقات المضمنة افتراضياً | `'children,products'` |
| `exclude` | الأعمدة المستبعدة | `'password,remember_token'` |
| `per_page` | عدد النتائج لكل صفحة | `50` |
| `is_public` | فلترة السجلات العامة | `true` |
| `is_has_cateables` (لـ categories) | تصفية التصنيفات المرتبطة بأي كائن | `true` |
| `is_has_shops` (لـ categories) | تصفية التصنيفات المرتبطة بمتاجر | `true` |
| `is_has_products` (لـ categories) | تصفية التصنيفات المرتبطة بمنتجات | `true` |
| `is_departments` (لـ types) | فلترة الأنواع التي تمثل أقساماً | `true` |

**مثال من `config.php` الجديد:**
```php
'categories' => [
    'include' => env('NANO_TAGSAPI_CATEGORIES_INCLUDE', ''),
    'exclude' => env('NANO_TAGSAPI_CATEGORIES_EXCLUDE', ''),
    'is_display_empty' => env('NANO_TAGSAPI_CATEGORIES_IS_DISPLAY_EMPTY', true),
    'is_has_cateables' => env('NANO_TAGSAPI_CATEGORIES_IS_HAS_CATEABLES', false),
    'is_has_shops' => env('NANO_TAGSAPI_CATEGORIES_IS_HAS_SHOP', false),
    'is_has_products' => env('NANO_TAGSAPI_CATEGORIES_IS_HAS_PRODUCTS', false),
    'is_public' => env('NANO_TAGSAPI_CATEGORIES_IS_PUBLIC', true),
    'per_page' => env('NANO_TAGSAPI_CATEGORIES_PER_PAGE', 50),
],
```

##### 2. تحسين متحكم `Categories` (فئة `Categories.php`)

- **قراءة `isPublic` من الإعدادات** إذا لم يتم تمريرها في الطلب.
- **دعم `mainCategoriesId`** لدمجها مع `parent_id` (مفيد لعرض تصنيفات رئيسية محددة بالإضافة إلى تصنيفات فرعية معينة).
- **إضافة خيارات `is_has_products`، `is_has_shops`، `is_has_cateables`**:
  - عند تفعيل أي من هذه الخيارات، يتم بناء استعلام معقد باستخدام `leftJoin` مع جداول `nano_tags_categories as child` و `nano_tags_cateables`.
  - يتم حساب عدد الكائنات المرتبطة (`cateables_count`, `sub_cateables_count`) وتصفية النتائج بناءً على `cateables_count`.
- **إضافة خيار `is_to_sql`** لطباعة استعلام SQL النهائي في `trace_log` لتسهيل التصحيح.
- **تحسين التعامل مع `orderBy` و `orderDirection`**:
  ```php
  if ($orderBy && $orderDirection) {
      $posts->orderBy(\Db::raw($orderBy), \Db::raw($orderDirection));
  } else {
      $posts->orderBy(\Db::raw(\Config::get('nano.tagsapi::categories.order_by','sort_order')), 
                     \Db::raw(\Config::get('nano.tagsapi::categories.order_dir','asc')));
  }
  ```
- **إضافة `$CategorieTransformer->defaultIncludes`** بناءً على قيمة `$include`.

##### 3. إضافة أحداث `api.list.extendQueryBefore` و `api.list.extendQuery`

تم إضافة هذين الحدثين في جميع المتحكمات (`Categories`, `Tags`, `Types`, `Menus`) لتمكين المطورين من تعديل الاستعلام قبل وبعد تطبيق الفلاتر الأساسية.

**مثال الاستخدام:**
```php
Event::listen('api.list.extendQuery', function ($query, $controller, $model, $options) {
    $query->where('is_featured', true);
});
```

##### 4. تحسين متحكم `Types`

- **إضافة دعم `is_departments`**:
  ```php
  $is_departments = Input::get('isDepartments', \Config::get('nano.tagsapi::types.is_departments','*'));
  if($is_departments !== null && $is_departments !== '*' && $is_departments !== 'all' && is_bool($is_departments)){
      if($is_departments) $posts = $posts->where('is_departments',true);
      else $posts = $posts->where('is_departments',false);
  }
  ```
- **إضافة خيارين للتحكم في دمج `OrderHelper`**:
  - `is_stop_order_type`: لإيقاف تطبيق `OrderHelper` على مستوى `types`.
  - `is_stop_order_type_departments`: لإيقاف تطبيقه على مستوى الأقسام المرتبطة.
- **تحسين دوال `availablefilters` و `formfields`**:
  - إضافة `input_data` و `process_data` في الردود لتسهيل تتبع البيانات المدخلة والمعالجة.
  - تحسين التعامل مع الاستثناءات وإضافة تفاصيل `debug` في بيئة التطوير.

##### 5. تحسين متحكمي `Tags` و `Menus`

- **قراءة `per_page` و `page` من الإعدادات** مع إمكانية التجاوز عبر `Input::get()`.
- **إضافة `$src_options` واستدعاء الأحداث** بنفس طريقة `Categories`.
- **دعم `isPublic` من الإعدادات**:
  ```php
  if(Input::get('isPublic', \Config::get('nano.tagsapi::tags.is_public', true))){
      $posts = $posts->where('is_public', true);
  }
  ```

##### 6. توحيد التعامل مع الصفحات (Pagination)

في جميع المتحكمات، أصبح الكود التالي هو المعيار:
```php
$per_page = Input::get('per_page', null);
if(is_null($per_page))
    $per_page = \Config::get('nano.tagsapi::categories.per_page', 15);
$per_page = intval($per_page);
if(!$per_page) $per_page = 15;

$page = intval(Input::get('page', 1));
if(!$page) $page = 1;

$paginator = $posts->paginate($per_page, $page);
```

---

### الفوائد والقيمة المضافة

- **مرونة أكبر**: يمكن الآن التحكم في سلوك الـ API بالكامل عبر متغيرات البيئة أو ملف `config.php` دون تعديل الكود.
- **أداء محسن**: الاستعلامات المعقدة في `Categories` أصبحت أكثر كفاءة بفضل استخدام `leftJoin` المحسّن و`groupBy` المناسب.
- **سهولة التصحيح**: إضافة `is_to_sql` تسمح للمطورين بمراجعة الاستعلامات النهائية مباشرة.
- **توافقية مع الإضافات الأخرى**: الأحداث الجديدة `extendQueryBefore/After` تتيح لبقية الإضافات تعديل الاستعلامات بسهولة.
- **دعم أفضل للعلاقات**: إمكانية تصفية التصنيفات بناءً على وجود منتجات أو متاجر أو أي كائن مرتبط عبر `cateables`.

---

### متطلبات الترقية إلى الإصدار 1.0.10

1. **تحديث ملف `config.php`**:
   - أضف المتغيرات الجديدة في ملف الإعدادات الخاص بك أو في `.env` حسب الحاجة.
   - مثال متغيرات البيئة الجديدة:
     ```ini
     NANO_TAGSAPI_CATEGORIES_IS_HAS_PRODUCTS=true
     NANO_TAGSAPI_CATEGORIES_IS_HAS_SHOP=true
     NANO_TAGSAPI_TYPES_IS_DEPARTMENTS=true
     NANO_TAGSAPI_TYPES_PER_PAGE=50
     NANO_TAGSAPI_CATEGORIES_PER_PAGE=30
     ```

2. **تحديث أي كود مخصص يعتمد على الـ API**:
   - إذا كنت تستخدم `Input::get('per_page')` بشكل مباشر في التطبيق، فسيظل يعمل، لكن القيمة الافتراضية أصبحت من الإعدادات.
   - إذا كنت تعتمد على القيمة الافتراضية `15` للصفحات، فتأكد من أن إعدادات `per_page` في `config.php` مناسبة.

3. **الاستفادة من الأحداث الجديدة**:
   - يمكنك الآن إضافة مستمعين للأحداث `api.list.extendQueryBefore` و `api.list.extendQuery` في `boot()` الخاص بأي إضافة.

4. **اختبار الاستعلامات المعقدة**:
   - إذا قمت بتفعيل `is_has_products` أو `is_has_shops`، تأكد من أن جداول `nano_tags_cateables` تحتوي على البيانات الصحيحة.
   - استخدم `is_to_sql=true` في بيئة التطوير لمراجعة الاستعلام النهائي.

---

### متغيرات البيئة (Environment Variables) المدعومة في `Nano.TagsApi`

يمكنك تخصيص سلوك الإضافة بالكامل عبر متغيرات البيئة في ملف `.env`، دون الحاجة لتعديل ملف `config.php` مباشرة. فيما يلي جميع المتغيرات المتاحة مع وصفها وقيمها الافتراضية:

#### 1. إعدادات `types`

| المتغير | الوصف | القيمة الافتراضية |
|---------|------|-------------------|
| `NANO_TAGSAPI_TYPES_INCLUDE` | العلاقات المضمنة افتراضياً (مفصولة بفواصل) | `''` |
| `NANO_TAGSAPI_TYPES_EXCLUDE` | الأعمدة المستبعدة من الاستعلام | `''` |
| `NANO_TAGSAPI_TYPES_IS_DEPARTMENTS` | فلترة الأنواع التي تمثل أقساماً (`true`/`false`) | `true` |
| `NANO_TAGSAPI_TYPES_IS_PUBLIC` | عرض الأنواع العامة فقط | `true` |
| `NANO_TAGSAPI_TYPES_PER_PAGE` | عدد النتائج لكل صفحة | `15` |
| `NANO_TAGSAPI_TYPES_ORDER_BY` | عمود الترتيب الافتراضي | `sort_order` |
| `NANO_TAGSAPI_TYPES_ORDER_DIR` | اتجاه الترتيب الافتراضي (`asc`/`desc`) | `asc` |
| `NANO_TAGSAPI_TYPES_IS_STOP_ORDER_TYPE` | إيقاف دمج `OrderHelper` على مستوى `types` | `false` |
| `NANO_TAGSAPI_TYPES_IS_STOP_ORDER_TYPE_DEPARTMENTS` | إيقاف دمج `OrderHelper` على مستوى الأقسام المرتبطة | `false` |

#### 2. إعدادات `tags`

| المتغير | الوصف | القيمة الافتراضية |
|---------|------|-------------------|
| `NANO_TAGSAPI_TAGS_INCLUDE` | العلاقات المضمنة افتراضياً | `''` |
| `NANO_TAGSAPI_TAGS_EXCLUDE` | الأعمدة المستبعدة | `''` |
| `NANO_TAGSAPI_TAGS_IS_PUBLIC` | عرض الوسوم العامة فقط | `true` |
| `NANO_TAGSAPI_TAGS_PER_PAGE` | عدد النتائج لكل صفحة | `15` |
| `NANO_TAGSAPI_TAGS_ORDER_BY` | عمود الترتيب الافتراضي | `sort_order` |
| `NANO_TAGSAPI_TAGS_ORDER_DIR` | اتجاه الترتيب الافتراضي | `asc` |

#### 3. إعدادات `categories`

| المتغير | الوصف | القيمة الافتراضية |
|---------|------|-------------------|
| `NANO_TAGSAPI_CATEGORIES_INCLUDE` | العلاقات المضمنة افتراضياً | `''` |
| `NANO_TAGSAPI_CATEGORIES_EXCLUDE` | الأعمدة المستبعدة | `''` |
| `NANO_TAGSAPI_CATEGORIES_IS_DISPLAY_EMPTY` | عرض التصنيفات الفارغة (بدون محتوى مرتبط) | `true` |
| `NANO_TAGSAPI_CATEGORIES_IS_HAS_CATEABLES` | تصفية التصنيفات المرتبطة بأي كائن عبر `cateables` | `false` |
| `NANO_TAGSAPI_CATEGORIES_IS_HAS_SHOP` | تصفية التصنيفات المرتبطة بمتاجر | `false` |
| `NANO_TAGSAPI_CATEGORIES_IS_HAS_PRODUCTS` | تصفية التصنيفات المرتبطة بمنتجات | `false` |
| `NANO_TAGSAPI_CATEGORIES_IS_PUBLIC` | عرض التصنيفات العامة فقط | `true` |
| `NANO_TAGSAPI_CATEGORIES_PER_PAGE` | عدد النتائج لكل صفحة | `15` |
| `NANO_TAGSAPI_CATEGORIES_ORDER_BY` | عمود الترتيب الافتراضي | `sort_order` |
| `NANO_TAGSAPI_CATEGORIES_ORDER_DIR` | اتجاه الترتيب الافتراضي | `asc` |

#### 4. إعدادات `menus`

| المتغير | الوصف | القيمة الافتراضية |
|---------|------|-------------------|
| `NANO_TAGSAPI_MENUS_INCLUDE` | العلاقات المضمنة افتراضياً | `''` |
| `NANO_TAGSAPI_MENUS_EXCLUDE` | الأعمدة المستبعدة | `''` |
| `NANO_TAGSAPI_MENUS_IS_PUBLIC` | عرض القوائم العامة فقط | `true` |
| `NANO_TAGSAPI_MENUS_PER_PAGE` | عدد النتائج لكل صفحة | `15` |
| `NANO_TAGSAPI_MENUS_ORDER_BY` | عمود الترتيب الافتراضي | `sort_order` |
| `NANO_TAGSAPI_MENUS_ORDER_DIR` | اتجاه الترتيب الافتراضي | `asc` |

---

#### مثال لملف `.env`:

```ini
# إعدادات Nano.TagsApi
NANO_TAGSAPI_CATEGORIES_IS_HAS_PRODUCTS=true
NANO_TAGSAPI_CATEGORIES_IS_HAS_SHOP=true
NANO_TAGSAPI_CATEGORIES_PER_PAGE=30
NANO_TAGSAPI_TYPES_IS_DEPARTMENTS=true
NANO_TAGSAPI_TYPES_PER_PAGE=50
NANO_TAGSAPI_TAGS_PER_PAGE=20
NANO_TAGSAPI_CATEGORIES_INCLUDE=children,parent
NANO_TAGSAPI_TYPES_EXCLUDE=password,remember_token
```

---

#### ملاحظات هامة

- جميع المتغيرات اختيارية. إذا لم يتم تعيينها، فسيتم استخدام القيم الافتراضية المدمجة في `config.php`.
- يمكن تجاوز أي من هذه الإعدادات عبر تمرير نفس المعامل في طلب API (مثل `?per_page=10`)، حيث أن `Input::get()` له الأولوية على الإعدادات.
- يُنصح باستخدام متغيرات البيئة في بيئة الإنتاج 

---

### الخاتمة

يمثل الإصدار `1.0.10` من إضافة `Nano.TagsApi` نقلة نوعية في مرونة النظام وقابليته للتوسع. من خلال الاعتماد على الإعدادات المركزية، وإضافة أحداث التوسع، وتحسين استعلامات `Categories`، أصبح من السهل الآن دمج `TagsApi` مع متطلبات أي مشروع معقد. كما أن تحسين التعامل مع الصفحات والترتيب يضمن اتساق السلوك عبر جميع نقاط النهاية.

نوصي جميع المشاريع التي تستخدم `Nano.TagsApi` بالترقية إلى هذا الإصدار للاستفادة من التحسينات والأداء الأفضل.

---


