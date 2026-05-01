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

## 2026-04-18

**إضافة دعم مزودات SMS جديدة للإضافة Nano.SmsNotify**  

### مقدمة
تم تطوير إضافة **Nano.SmsNotify** لإدارة وإرسال الرسائل النصية القصيرة (SMS) عبر مزودات متعددة. الإضافة تدعم حاليًا العديد من مزودات الخدمة مثل Twilio، Nexmo، Alawaeltec، SmsSaudi، وغيرها. في هذا التحديث الأخير (الإصدار 1.0.13)، قمنا بإضافة خمسة مزودات SMS جديدة تغطي احتياجات مناطق إضافية وتوفر خيارات أكثر للمستخدمين.

### المزودات الجديدة
1. **BdBulkSms** – متخصص في خدمة بنغلاديش (GreenWeb).  
2. **ReveSms** – مزود يدعم الإرسال عبر API مع مصادقة باستخدام API Key و Secret Key.  
3. **SmsToday** – يوفر إرسال SMS بسهولة عبر endpoint بسيط.  
4. **TonkraSms** – يدعم الإرسال باستخدام Bearer token ويغطي عدة مناطق.  
5. **ReleansSms** – مزود عالمي يتطلب Bearer token و Sender ID.  

### الهدف من الإضافة
توسيع قاعدة العملاء الذين يحتاجون إلى استخدام مزودات محلية أو عالمية معينة، مما يزيد من مرونة النظام ويمكّن العملاء من اختيار الأنسب من حيث التكلفة والتغطية الجغرافية.

### تفاصيل التحديث

#### 1. إضافة كلاسات المزودات
تم إنشاء خمس كلاسات جديدة داخل namespace `Nano\SmsNotify\SmsChannels`:

- `BdBulkSms.php`  
- `ReveSms.php`  
- `SmsToday.php`  
- `TonkraSms.php`  
- `ReleansSms.php`

كل كلاس يرث من `BaseChannel` ويطبق دالة `send()` التي تستخدم cURL للتواصل مع API المزود، بالإضافة إلى تعريف حقول الإعدادات المطلوبة عبر `defineFormConfig()`.

#### 2. هيكل الملفات
تم وضع الكلاسات في المسار:
```
plugins/nano/smsnotify/smschannels/
```
كما تم إضافة ملفات partial للإرشادات (`_info.htm`) لكل مزود تشرح كيفية الحصول على البيانات المطلوبة (API keys، أرقام المرسل، إلخ).

#### 3. تسجيل المزودات في الإضافة
تم تحديث دالة `registerSmsChannels()` في `Plugin.php` بإضافة الأسطر التالية:
```php
'bdbulksms'    => \Nano\SmsNotify\SmsChannels\BdBulkSms::class,
'revesms'      => \Nano\SmsNotify\SmsChannels\ReveSms::class,
'smstoday'     => \Nano\SmsNotify\SmsChannels\SmsToday::class,
'tonkrasms'    => \Nano\SmsNotify\SmsChannels\TonkraSms::class,
'releanssms'   => \Nano\SmsNotify\SmsChannels\ReleansSms::class,
```
وبهذا تظهر هذه المزودات في واجهة إدارة القنوات داخل الإضافة.

#### 4. تحديث ملف الإصدارات (version.yaml)
تمت إضافة إصدار جديد `1.0.13` مع وصف التحديث:
```yaml
1.0.13:
    - Add new SMS channels: BdBulkSms, ReveSms, SmsToday, TonkraSms, ReleansSms
    - Extend SMS gateway support for additional regions
```

#### 5. اختبار المزودات
تم إجراء اختبارات أولية لكل مزود للتأكد من إمكانية الاتصال وإرسال رسائل اختبارية. تم التأكد من معالجة الأخطاء وإرجاع النتائج بشكل متوافق مع بنية `BaseChannel` (حالة النجاح/الفشل، معرف الرسالة، رسالة الخطأ، إلخ).

### الفوائد المتوقعة
- **توسيع قاعدة العملاء**: دعم مزودات تغطي أسواقًا جديدة (بنغلاديش، مناطق متعددة).  
- **زيادة المرونة**: إتاحة خيارات متعددة للعملاء تبعًا لتكلفة الإرسال والتغطية.  
- **تحسين تجربة المستخدم**: توفير تعليمات واضحة داخل الإضافة لتهيئة كل مزود بسهولة.

### الخطوات المستقبلية
- مراقبة أداء المزودات الجديدة بعد الإطلاق.  
- تحديث الوثائق لتشمل المزودات الجديدة.  
- إضافة المزيد من المزودات بناءً على طلبات العملاء.

## 2026-04-16 – 2026-04-18

**تحديثات إضافة `Nano.FileUpload` – الإصداران 1.2.4 و 1.2.5**

### ملخص التحديثات

شهدت الإضافة تطورين مهمين في الإصدارين **1.2.4** و **1.2.5**:

- **الإصدار 1.2.4** أضاف ميزة أمنية جديدة على مستوى الحقل: `allow_upload_only_when_model_exists`، والتي تمنع رفع الملفات إلى حقل معين إلا بعد أن يتم حفظ النموذج الأصلي (وجود `model_id` صالح). هذه الميزة مفيدة بشكل خاص للحقول المتعددة (`multiple`) مثل معارض الصور التي لا ينبغي رفع صورها قبل إنشاء السجل الرئيسي.

- **الإصدار 1.2.5** قام بإعادة هيكلة شاملة لنقاط نهاية API الخاصة بالاستعلام عن الموديولات والصلاحيات، وذلك بنقل معاملات `model_class` و `field` من المسار (route parameters) إلى معاملات الاستعلام (query parameters). هذا التغيير يحل مشكلة ترميز أسماء الموديولات التي تحتوي على خطوط مائلة عكسية (`\`) ويجعل المسارات أكثر نظافة وأماناً.

---

## الإصدار 1.2.4 – منع رفع الملفات قبل حفظ النموذج

### أهداف الإصدار

- **إضافة طبقة أمان إضافية** تمنع رفع الملفات إلى الحقول التي تتطلب وجود النموذج أولاً (مثل معارض الصور، المستندات المرتبطة بسجل محفوظ).
- **تحسين تجربة المطور** بتوفير خيار `allow_upload_only_when_model_exists` في تعريف الحقل، مما يسمح بمنع الرفع المؤقت (باستخدام `temp_session_key`) لهذا الحقل.
- **تجنب تراكم الملفات اليتيمة** (غير المرتبطة) في النظام.

### الميزات الجديدة

#### 1. إضافة خيار `allow_upload_only_when_model_exists` في تعريف الحقل

أصبح من الممكن الآن تعيين هذا الخيار في تعريف أي حقل (خاصة الحقول من نوع `multiple` أو `image` التي تتطلب وجود النموذج):

```php
'fields' => [
    'gallery' => [
        'type' => 'multiple',
        'max_filesize' => 2048,
        'allowed_types' => 'jpg,jpeg,png',
        'max_files' => 10,
        'allow_upload_only_when_model_exists' => true, // ⬅️ جديد
    ],
],
```

- **القيمة الافتراضية:** `false` (يسمح بالرفع المؤقت حتى للنماذج غير المحفوظة).
- **عند تفعيله (`true`)**: يمنع رفع الملفات لهذا الحقل إذا لم يتم توفير `model_id` صالح (أي النموذج غير موجود في قاعدة البيانات). سيتم رفض الطلب مع رسالة خطأ `FILE_UPLOAD_PERMISSION_DENIED` ورسالة `model_must_exist_before_upload`.

#### 2. التحقق في `FileUploadService::upload()`

أضفنا في دالة `upload()` منطقاً للتحقق من هذا الخيار:

- يتم التحقق من وجود النموذج (`model_exists`) إما من خلال `options['model']` (إذا كان موجوداً ومحفوظاً) أو عن طريق البحث عن `model_id` وجلب النموذج.
- إذا كان الخيار مفعلاً ولم يكن النموذج موجوداً، يتم رمي استثناء `FileUploadException` مع كود الخطأ المناسب.

#### 3. رسالة ترجمة جديدة

أضفنا مفتاح ترجمة `errors.model_must_exist_before_upload`:

```php
'model_must_exist_before_upload' => 'لا يمكن رفع الملفات لحقل ":field" إلا بعد حفظ ":model". يرجى حفظ :model أولاً ثم المحاولة مرة أخرى.',
```

### الفوائد

- **منع الملفات اليتيمة**: تضمن عدم وجود ملفات غير مرتبطة بأي سجل في قاعدة البيانات.
- **تحكم دقيق**: يمكن تفعيل الميزة على حقول محددة فقط (مثل المعارض) مع ترك الحقول الأخرى (مثل الصورة الرمزية) تسمح بالرفع المؤقت.
- **أمان محسن**: يمنع المستخدمين من رفع ملفات لحقول تتطلب وجود السجل الأصلي قبل إنشائه.

---

## الإصدار 1.2.5 – إعادة هيكلة مسارات API (Query Parameters بدلاً من Route Parameters)

### أهداف الإصدار

- **حل مشكلة ترميز أسماء الموديولات** التي تحتوي على خطوط مائلة عكسية (`\`) في المسارات (مثل `Nano\Shop\Models\Product`). كان تمريرها في المسار يتطلب ترميزاً (`Nano%5CShop%5CModels%5CProduct`) مما يجعل المسارات غير عملية وغير قابلة للقراءة.
- **تبسيط نقاط النهاية** باستخدام معاملات الاستعلام (`?model_class=...`) بدلاً من معاملات المسار.
- **الحفاظ على التوافق العكسي** عبر إبقاء المسارات القديمة (مع إهمالها) مع إتاحة المسارات الجديدة.

### الميزات الجديدة

#### 1. نقاط النهاية الجديدة (باستخدام query parameters)

| الوظيفة | المسار القديم (مهمل) | المسار الجديد |
|---------|---------------------|----------------|
| جلب إعدادات موديول | `GET /models/{modelClass}` | `GET /model/config?model_class=...` |
| جلب إعدادات حقل | `GET /models/{modelClass}/fields/{field}` | `GET /field/config?model_class=...&field=...` |
| جلب قيود الحقل | `GET /models/{modelClass}/fields/{field}/constraints` | `GET /field/constraints?model_class=...&field=...` |
| جلب خيارات المعالجة | `GET /processing-options/{modelClass}/{field}` | `GET /processing-options?model_class=...&field=...` |
| التحقق من صلاحية موديول | `GET /permissions/model/{modelClass}/{operation}` | `GET /permissions/model?model_class=...&operation=...` |
| التحقق من صلاحية حقل | `GET /permissions/field/{modelClass}/{field}/{operation}` | `GET /permissions/field?model_class=...&field=...&operation=...` |

**ملاحظة:** نقاط النهاية `GET /models` و `GET /permissions/global/{operation}` و `POST /permissions/check` و `POST /temp-key/validate` لم تتغير.

#### 2. تحديث دوال المتحكم `FileUploadController`

تم تعديل الدوال التالية لقراءة المعاملات من `Input::get()` (أو معاملات الاستعلام) بدلاً من معاملات المسار، مع دعم تمرير المعاملات عبر الطريقة القديمة كاحتياطي (للتوافق العكسي):

- `getModelConfig($modelClass = null)`
- `getFieldConfig($modelClass = null, $field = null)`
- `getFieldConstraints($modelClass = null, $field = null)`
- `getProcessingOptions($modelClass = null, $field = null)`
- `checkModelPermission($modelClass = null, $operation = null)`
- `checkFieldPermission($modelClass = null, $field = null, $operation = null)`

**مثال على الطريقة الجديدة للدالة `getModelConfig`:**

```php
public function getModelConfig($modelClass = null)
{
    $modelClass = $modelClass ?? Input::get('model_class');
    if (!$modelClass) {
        return $this->errorResponse('Missing model_class parameter', null, 400);
    }
    // ... باقي المنطق
}
```

#### 3. تحديث ملف `routes.php`

تم تعليق المسارات القديمة (باستخدام `/* ... */`) وإضافة المسارات الجديدة:

```php
// المسارات الجديدة
Route::get('model/config', [ 'as' => 'model.config', 'uses' => 'FileUploadController@getModelConfig' ]);
Route::get('field/config', [ 'as' => 'field.config', 'uses' => 'FileUploadController@getFieldConfig' ]);
Route::get('field/constraints', [ 'as' => 'field.constraints', 'uses' => 'FileUploadController@getFieldConstraints' ]);
Route::get('processing-options', [ 'as' => 'processing.options', 'uses' => 'FileUploadController@getProcessingOptions' ]);
Route::get('permissions/model', [ 'as' => 'permissions.model', 'uses' => 'FileUploadController@checkModelPermission' ]);
Route::get('permissions/field', [ 'as' => 'permissions.field', 'uses' => 'FileUploadController@checkFieldPermission' ]);
```

#### 4. أمثلة على الطلبات الجديدة

**جلب إعدادات موديول `Product`:**

```http
GET /api/v1/fileupload/model/config?model_class=Nano\Shop\Models\Product HTTP/1.1
Authorization: Bearer ...
```

**جلب قيود حقل `image`:**

```http
GET /api/v1/fileupload/field/constraints?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

**التحقق من صلاحية `edit` على حقل `image`:**

```http
GET /api/v1/fileupload/permissions/field?model_class=Nano\Shop\Models\Product&field=image&operation=edit HTTP/1.1
Authorization: Bearer ...
```

### الفوائد

- **مسارات نظيفة وقابلة للقراءة** – لا حاجة لترميز أسماء الموديولات.
- **سهولة الاستخدام من أي عميل HTTP** – تمرير `model_class` كقيمة عادية في query string.
- **توافق عكسي** – المسارات القديمة ما زالت تعمل (لكنها مهملة) وستتم إزالتها في إصدار مستقبلي.
- **أمان إضافي** – تجنب التعامل مع إدخال المستخدم كجزء من المسار (يقلل من مخاطر التوجيه الضار).

---

## متطلبات الترقية (من 1.2.3 إلى 1.2.5)

1. **تحديث الكود**:
   - استبدال `FileUploadController.php` بالنسخة التي تحتوي على الدوال المحدثة (تدعم query parameters).
   - استبدال `routes.php` بالنسخة التي تحتوي على المسارات الجديدة.
   - استبدال `FileUploadRegistry.php` بالنسخة التي تحتوي على خيار `allow_upload_only_when_model_exists` (تم تضمينه في الإصدار 1.2.4).
   - استبدال `FileUploadService.php` بالنسخة التي تطبق التحقق من الخيار الجديد.

2. **تحديث تعريفات النماذج** (اختياري):
   - إذا كنت ترغب في استخدام `allow_upload_only_when_model_exists`، أضفه إلى تعريفات الحقول المطلوبة.

3. **لا توجد هجرات جديدة** – لا تغييرات في قاعدة البيانات.

4. **لا توجد تغييرات في الإعدادات** – يظل ملف `config.php` كما هو.

5. **تحديث واجهات المستخدم (API clients)**:
   - إذا كنت تستخدم نقاط النهاية القديمة (ذات الأقواس المتعرجة)، يُنصح بالانتقال إلى المسارات الجديدة فوراً. المسارات القديمة ما زالت تعمل حالياً ولكنها مهملة وستتم إزالتها في الإصدار 1.3.0.

6. **اختبار التوافق**:
   - تأكد من أن جميع نقاط النهاية الجديدة تعمل بشكل صحيح مع `model_class` كمعامل استعلام.
   - اختبر حقل مع `allow_upload_only_when_model_exists = true` للتأكد من رفض الرفع قبل حفظ النموذج.

---

## الخاتمة

يمثل الإصداران **1.2.4** و **1.2.5** خطوة مهمة نحو تحسين أمان النظام وسهولة استخدام واجهة API. فبينما أضاف الإصدار 1.2.4 تحكماً دقيقاً في متى يُسمح برفع الملفات (منع الرفع المؤقت للحقول التي تتطلب وجود النموذج)، قدم الإصدار 1.2.5 إعادة هيكلة شاملة لنقاط النهاية لتكون أكثر نظافة وأماناً. هذه التحديثات تجعل الإضافة أكثر احترافية وتسهل دمجها مع التطبيقات الخارجية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)


## 2026-04-10 – 2026-04-18

**الإطلاق الرسمي لإضافة `Nano3.Kyc` – نظام ذكي ومتكامل للتحقق من الهوية (KYC) لبرمجيات نانوسوفت (NanoSoft App)**

---

### 🚀 تقديم الحزمة: لماذا `Nano3.Kyc`؟

في عالم الأعمال الرقمية المتسارع، أصبح التحقق من هوية العملاء والموردين (KYC) متطلبًا أساسيًا للامتثال التنظيمي وبناء الثقة. سواء كنت تدير منصة تجارة إلكترونية، أو مؤسسة مالية رقمية، أو سوقًا إلكترونيًا، فإن الحاجة إلى نظام مرن وآمن لجمع وإدارة وثائق الهوية باتت أمرًا لا غنى عنه.

هنا تأتي إضافة **`Nano3.Kyc`** – الحل المتكامل من **Nano2Soft** والمصمم خصيصًا لبيئة **NanoSoft App**. إنها ليست مجرد أداة لتخزين صور الوثائق، بل هي **منصة ذكية لإدارة دورة حياة التحقق من الهوية** بالكامل. بدءًا من تعريف أنواع الوثائق المطلوبة (بأكثر من 28 نوعًا جاهزًا) وصولاً إلى إدارة الحقول الديناميكية، والتحقق من الصلاحية، وتتبع النسخ الورقية، وتقديم واجهات برمجية (API) وواجهات إدارة خلفية متكاملة.

تم بناء `Nano3.Kyc` على أسس **المرونة والأمان**، مما يسمح لها بالتكيف مع أي نموذج عمل أو متطلبات تنظيمية متغيرة، دون الحاجة إلى تعديلات برمجية معقدة.

---

### 🎯 لماذا تم تطوير `Nano3.Kyc`؟ (الحاجة والمشكلة)

قبل `Nano3.Kyc`، كان المطورون في نانوسوفت يواجهون تحديات متكررة عند تنفيذ أنظمة KYC:

| التحدي | حل `Nano3.Kyc` |
| :--- | :--- |
| **عدم وجود هيكل موحد** | توفر الإضافة نموذج بيانات مرن (`Document`) مصمم لاستيعاب **أي نوع وثيقة**، مع أعمدة مفهرسة للحقول الشائعة وحقل JSON للحقول المتخصصة. |
| **تكرار كتابة كود التحقق** | يوفر كلاس `DocumentType` تعريفًا مركزيًا لكل نوع وثيقة مع قواعد تحقق جاهزة (حجم الملف، نوعه، تاريخ الصلاحية، إلخ). |
| **صعوبة إدارة الوثائق الورقية** | تتضمن الإضافة نظامًا متكاملاً لتتبع **النسخ الورقية المستلمة والمرتجعة** (`is_physical_submitted`, `physical_received_at`, ...). |
| **التحقق من صلاحية الوثائق بمرور الوقت** | دوال مدمجة للتحقق من صلاحية الوثائق (`isDocumentAcceptable`, `isDocumentExpiring`) تنبه تلقائيًا عند انتهاء صلاحية الوثيقة. |
| **الحاجة إلى واجهات إدارة مخصصة** | توفر الإضافة **واجهة خلفية جاهزة** (قوائم، نماذج، فلاتر) و **واجهة API RESTful** كاملة للربط مع تطبيقات الجوال. |
| **تعدد أنواع المستخدمين والصلاحيات** | تدعم الإضافة العلاقات المتعددة (`owner`, `verifier`, `subject`) مما يسمح بربط الوثيقة بمالكها (فرد/شركة) وبالمحقق، مع إمكانية تطبيق صلاحيات دقيقة. |

باختصار، وُلدت `Nano3.Kyc` لإنهاء التشتت وتوفير **حل مركزي واحد** لإدارة جميع متطلبات KYC في مشاريع نانوسوفت.

---

### 💎 المكونات الأساسية وقدرات الحزمة

تتألف حزمة `Nano3.Kyc` من عدة مكونات متكاملة تعمل بتناغم:

| المكون | الوظيفة |
| :--- | :--- |
| **`DocumentType`** | **العقل المدبر للإضافة**. كلاس مركزي يعرف أكثر من **28 نوع وثيقة** جاهزة (جواز سفر، هوية، فاتورة، رخصة تجارية، إلخ) موزعة على 6 فئات. يحتوي على مخططات الحقول (`fields schema`) وقواعد التحقق (`validation rules`) لكل نوع، ويدعم تحميل الإعدادات من ملف `config`. |
| **`Manager`** | **منسق العمليات**. يوفر واجهة برمجية موحدة لإنشاء الوثائق وتحديثها وحذفها واعتمادها ورفضها. يدعم المعاملات (`transactions`) والأحداث (`events`) ووضع الاختبار (`is_test_create`). |
| **`Document` (Model)** | **نموذج البيانات**. جدول `nano3_kyc_documents` مصمم ليكون العمود الفقري لتخزين جميع الوثائق. يستخدم مجموعة غنية من السمات (`Traits`) لتوفير نطاقات بحث (`Scopes`) وخيارات قوائم (`Options`) وتخزين مؤقت (`Cache`) وفلاتر متقدمة (`getRecords`). |
| **`Documents` (API Controller)** | **بوابة التكامل**. يوفر نقاط نهاية RESTful محمية بـ OAuth للتفاعل مع النظام من التطبيقات الخارجية. جميع الاستجابات تتبع هيكل `Nano.API` الموحد. |
| **`Documents` (Backend Controller)** | **واجهة المستخدم الإدارية**. يوفر قوائم ونماذج إدارة متكاملة داخل لوحة تحكم نانوسوفت، مع دعم الفلاتر المتقدمة والبحث والفرز. |
| **`DocumentTransformer`** | **منسق البيانات**. مسؤول عن تحويل بيانات الموديل إلى JSON متوافق مع API، مع دعم تضمين العلاقات وتنسيق الحقول. |
| **ملفات الترجمة (`lang.php`)** | **تجربة مستخدم محلية**. جميع النصوص والإشعارات قابلة للترجمة (العربية متوفرة بالكامل)، مما يضمن تجربة سلسة للمستخدمين. |

---

### ✨ أبرز مميزات الإصدار الأول (1.0.0)

يمثل هذا الإطلاق تتويجًا لأعمال تطوير مكثفة، ويضم مجموعة غنية من الميزات:

#### 1. تعريف مركزي لأكثر من 28 نوع وثيقة

- **6 فئات رئيسية**: هوية أساسية، هوية ثانوية، إثبات عنوان، وثائق شركات، المستفيد الحقيقي (UBO)، تحقق إضافي.
- **خصائص متقدمة لكل نوع**: هل تنتهي صلاحيته؟ (`has_expiry`)، العمر الأقصى بالأيام (`max_age_days`)، وزن الموثوقية (`verification_weight`)، أنواع الملفات المسموحة (`allowed_mime_types`)، الجهات المسموح لها (`allowed_for_entity`)، وغيرها.
- **قابلية التخصيص الكامل**: يمكن تعديل أي خاصية أو إضافة أنواع جديدة بالكامل عبر ملف `config/nano3/kyc/document_types.php`.

#### 2. حقول ديناميكية وذكية

- **توليد نماذج تلقائيًا**: دالة `getFieldsSchema()` تعيد مخطط الحقول الكامل لأي نوع وثيقة، مما يسمح ببناء واجهات إدخال ديناميكية بسهولة.
- **تخزين هجين**: الحقول الشائعة والمفهرسة (مثل `document_number`, `full_name`, `expiry_date`) تخزن في أعمدة منفصلة للأداء، بينما الحقول المتخصصة (مثل `passport_type`, `religion`) تخزن في حقل JSON واحد (`fields_data`).
- **تحويل تلقائي للبيانات**: دوال `mapInputToDocumentData` و `mapDocumentToOutput` تسهل التعامل مع هذا الهيكل المختلط.

#### 3. دورة حياة كاملة للوثيقة

- **حالات متعددة**: `pending` (قيد الانتظار)، `verified` (تم التحقق)، `rejected` (مرفوض)، `expired` (منتهي).
- **عمليات التحقق**: اعتماد الوثيقة (`verifyDocument`) يحدث الحالة ويسجل المحقق وتاريخ التحقق ودرجة الموثوقية.
- **رفض مع سبب**: إمكانية رفض الوثيقة مع تسجيل سبب الرفض في `metadata`.

#### 4. إدارة متقدمة للنسخ الورقية (Hard Copy Tracking)

- **تتبع دقيق**: حقول مخصصة لتسجيل استلام النسخة الورقية (`physical_received_at`) وإعادتها (`physical_returned_at`) ونوعها (أصلية/صورة).
- **ملاحظات**: حقل `physical_notes` لتوثيق أي معلومات إضافية.

#### 5. واجهة برمجة تطبيقات غنية (RESTful API)

- **نقاط نهاية شاملة**: إنشاء، عرض، تحديث، حذف، اعتماد، رفض، استعادة، وإحصائيات.
- **فلترة متقدمة**: يمكن تمرير معاملات مثل `document_type`, `status`, `owner_id`, `issue_date` لتصفية النتائج.
- **تنسيق موحد**: جميع الاستجابات تتبع هيكلًا موحدًا يسهل معالجته.

#### 6. واجهة خلفية سهلة الاستخدام

- **قائمة وثائق قابلة للتخصيص**: أعمدة قابلة للفرز والبحث، فلاتر سريعة، وإجراءات جماعية.
- **نموذج إدخال منظم**: مقسم إلى تبويبات منطقية (الأساسية، المالك، التحقق، النسخة الورقية، الملفات، النشر).
- **قوائم منسدلة ذكية**: تعتمد على بعضها (مثل `owner_id` يعتمد على `owner_type`).

---

### 🔧 التطلع إلى المستقبل

هذا الإطلاق هو مجرد البداية. تلتزم Nano2Soft بمواصلة تطوير `Nano3.Kyc` لتشمل:

- **التعرف الضوئي على الحروف (OCR)**: استخراج البيانات تلقائيًا من صور الوثائق.
- **التحقق التلقائي من الوثائق**: التكامل مع خدمات خارجية للتحقق من صحة الوثائق.
- **لوحات تحكم تحليلية**: عرض إحصائيات متقدمة عن حالة الوثائق وعمليات التحقق.
- **قوالب وثائق قابلة للتخصيص**: إمكانية إضافة حقول مخصصة عبر واجهة المستخدم دون الحاجة للبرمجة.

---

### 📚 الوثائق المرجعية

- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

---

**Nano3.Kyc** – تم تطويره بحب بواسطة فريق **Nano2Soft** 💙

## 2026-04-18 – 2026-04-19

**تحديث إضافة `Nano.FileUpload` – الإصدار 1.2.6**

### إصلاح مشكلة الحذف المزدوج للملفات عند الرفع على علاقات `AttachOne`

---

### ملخص التحديثات

في الإصدار **1.2.6**، تم معالجة خطأ برمجي خطير كان يتسبب في حذف الملفات الجديدة فور رفعها على الحقول من نوع `AttachOne` (مثل صورة الغلاف، صورة الملف الشخصي، إلخ). المشكلة كانت ناتجة عن استدعاء مزدوج لعملية ربط الملف (`$relation->add()`) مما أدى إلى تنفيذ استعلام `DELETE` بعد `INSERT` مباشرة، واختفاء الملف من قاعدة البيانات على الرغم من نجاح عملية الرفع الظاهري.

الحل تمثل في:
1. إضافة خيارات تحكم دقيقة في دالة `Base64::onUpload` للتحكم في تعيين الحقول (`field`, `attachment_id`, `attachment_type`) وفي تنفيذ الربط (`skip_relation_add`).
2. تعديل `FileUploadService::upload` لاستخدام هذه الخيارات، بحيث يتم حفظ الملف بدون ربطه في `Base64`، ثم يتم ربطه مرة واحدة فقط بعد معالجة الملف وحذف الملف القديم (إن وجد).
3. إزالة الاستدعاء المزدوج لـ `add()` مما أزال استعلامات `DELETE` غير المرغوب فيها.

---

### أهداف الإصدار

- **إصلاح عيب برمجي خطير** كان يمنع رفع الملفات بشكل صحيح على علاقات `AttachOne` (خاصة في بيئات الاختبار ذات المعاملات المتداخلة).
- **منع الحذف غير المقصود للملفات الجديدة** الذي كان يحدث بعد الإدراج مباشرة.
- **تحسين استقرار وموثوقية نظام الرفع** خاصة للحقول التي تقبل ملفًا واحدًا فقط.
- **توفير تحكم أدق** في سلوك `Base64::onUpload` من خلال خيارات جديدة تسمح بتخطي تعيين حقول معينة أو تخطي عملية الربط بالكامل.
- **ضمان نجاح الاختبارات الآلية** المتعلقة برفع الملفات (`FILE_CREATE_001`, `FILE_UPDATE_001`) والتي كانت تفشل باستمرار بسبب هذا الخطأ.

---

### المشكلة الأصلية (قبل الإصلاح)

#### وصف السلوك الخاطئ

عند رفع ملف جديد لحقل من نوع `AttachOne` (مثل `document_front`)، كان النظام يقوم بالخطوات التالية:

1. **`Base64::onUpload`**: يحفظ الملف في جدول `system_files` **مع تعيين `attachment_id` و `attachment_type`** ثم يستدعي `$fileRelation->add($file)` لربطه بالنموذج.
2. **`FileUploadService::upload`**: بعد استدعاء `onUpload`، يقوم **مرة أخرى** باستدعاء `$relation->add($newFile)`.

**النتيجة:** الاستدعاء الثاني لـ `add()` كان يحذف جميع الملفات المرتبطة بالحقل (بما فيها الملف الجديد الذي تم ربطه للتو) ثم يعيد ربطه. لكن في بعض السيناريوهات (خاصة مع المعاملات المتداخلة)، كان استعلام `DELETE` يحذف السجل نهائيًا دون إعادة إدراجه بشكل صحيح، مما يؤدي إلى اختفاء الملف من قاعدة البيانات.

#### استعلامات SQL المرصودة (دليل على المشكلة)

```
INSERT INTO `system_files` (...) VALUES (...)  -- تم إدراج الملف الجديد (ID=7556)
DELETE FROM `system_files` WHERE `attachment_id` = ... AND `field` = ...  -- حذف الملف الجديد!
```

نتيجة هذا السلوك: `file_exists_in_db = false` وفشل اختبارات رفع الملفات.

---

### الحلول المطبقة في الإصدار 1.2.6

#### 1. إضافة خيارات تحكم جديدة في `Base64::onUpload`

أضفنا أربعة خيارات جديدة إلى مصفوفة `$options` في دالة `onUpload`:

| الخيار | النوع | القيمة الافتراضية | الوصف |
|--------|-------|-------------------|-------|
| `skip_set_field` | `bool` | `false` | إذا كان `true`، لا يتم تعيين `field` في نموذج `File`. |
| `skip_set_attachment_type` | `bool` | `false` | إذا كان `true`، لا يتم تعيين `attachment_type`. |
| `skip_set_attachment_id` | `bool` | `false` | إذا كان `true`، لا يتم تعيين `attachment_id`. |
| `skip_relation_add` | `bool` | `false` | إذا كان `true`، لا يتم استدعاء `$fileRelation->add()` (أي لا يتم ربط الملف). |

**الكود المضاف في `onUpload`:**

```php
$d_options = array_merge([
    // ... الخيارات السابقة
    'skip_set_field' => false,
    'skip_set_attachment_type' => false,
    'skip_set_attachment_id' => false,
    'skip_relation_add' => false,
], $options);

// ...

if (!$skip_set_field && $field) {
    $file->field = $field;
}
if (!$skip_set_attachment_type && $attachment_type) {
    $file->attachment_type = $attachment_type;
}
if (!$skip_set_attachment_id && $attachment_id) {
    $file->attachment_id = $attachment_id;
}

// ...

if (!$skip_relation_add && is_object($fileRelation)) {
    // ... تنفيذ الربط
}
```

#### 2. تعديل `FileUploadService::upload` لاستخدام الخيارات الجديدة

قمنا بتعديل خيارات `$uploadOptions` التي يتم تمريرها إلى `Base64::onUpload`:

```php
$uploadOptions = [
    // ... الخيارات السابقة
    'skip_set_field'           => false,
    'skip_set_attachment_type' => false,
    'skip_set_attachment_id'   => true,   // ⬅️ لا تعيّن attachment_id في Base64
    'skip_relation_add'        => true,   // ⬅️ لا تقم بالربط في Base64
];
```

ثم بعد استدعاء `onUpload` والحصول على `$newFile`، نقوم بالخطوات التالية **داخل `FileUploadService`**:

1. تعيين `field` إذا لم يكن موجوداً.
2. حساب `hash` و `meta` وحفظ التغييرات.
3. حذف الملف القديم (`$existingFile`) إذا كانت العملية `edit`.
4. **ربط الملف الجديد مرة واحدة فقط** باستخدام `$relation->add($newFile)`.

**الكود النهائي في `FileUploadService::upload`:**

```php
Db::transaction(function () use (...) {
    $uploadResult = Base64::onUpload($uploadOptions, $uploadedFile);
    $newFile = $uploadResult['model'];

    // تعيين الحقول الإضافية وحفظها
    if (!$newFile->field) {
        $newFile->field = $field;
    }
    // ... حساب hash و meta و disk ...

    $newFile->save();

    // حذف الملف القديم (إذا كانت العملية edit)
    if ($operation === 'edit' && $existingFile) {
        $existingFile->delete();
    }

    // ربط الملف الجديد مرة واحدة فقط
    if ($model && $model->exists && $model->hasRelation($field)) {
        $relation = $model->{$field}();
        if ($relation instanceof \October\Rain\Database\Relations\AttachOne) {
            $relation->add($newFile);
        }
    }
});
```

---

### النتائج بعد الإصلاح

- **اختفاء استعلام `DELETE` غير المرغوب فيه** بعد `INSERT`.
- **بقاء الملف في قاعدة البيانات** (`file_exists_in_db = true`).
- **نجاح اختبارات `FILE_CREATE_001` و `FILE_UPDATE_001`** بشكل موثوق.
- **عدم الحاجة إلى حلول التفافية في الاختبارات** (مثل `savepoint` أو التنظيف اليدوي).

---

### متطلبات الترقية (من 1.2.5 إلى 1.2.6)

1. **تحديث الكود**:
   - استبدال `Base64.php` بالنسخة التي تحتوي على الخيارات الجديدة (`skip_set_field`, `skip_set_attachment_type`, `skip_set_attachment_id`, `skip_relation_add`).
   - استبدال `FileUploadService.php` بالنسخة التي تستخدم هذه الخيارات وتنفذ الربط مرة واحدة فقط.

2. **لا توجد هجرات جديدة** – لا تغييرات في قاعدة البيانات.

3. **لا توجد تغييرات في الإعدادات** – يظل ملف `config.php` كما هو.

4. **لا توجد تغييرات في واجهة API** – جميع نقاط النهاية تعمل كما هي دون تعديل.

5. **اختبار التوافق**:
   - يُنصح بتشغيل اختبارات `KycPlusTest` (أو `FileUploadPlusTest`) للتأكد من نجاح عمليات الرفع والتحديث.
   - يجب أن تختفي أي أخطاء متعلقة بـ `FILE_CREATE_001` أو `FILE_UPDATE_001`.

---

### الفوائد والقيمة المضافة

- **إصلاح خطأ برمجي حرج** كان يؤثر على أي حقل `AttachOne` (مثل الصور الرمزية، صور الغلاف، الملفات الفردية).
- **زيادة موثوقية النظام** خاصة في بيئات الاختبار ذات المعاملات المتداخلة.
- **تحسين جودة الكود** بفصل مسؤوليات `Base64` (حفظ الملف فقط) عن `FileUploadService` (التحكم الكامل في دورة حياة الرفع والربط).
- **توفير خيارات تحكم دقيقة** يمكن استخدامها في سيناريوهات متقدمة أخرى.
- **تمهيد الطريق لإصدارات مستقبلية** قد تضيف ميزات مثل التحميل المسبق للملفات دون ربط فوري.

---

### الخاتمة

يمثل الإصدار **1.2.6** إصلاحًا جوهريًا لمشكلة كانت تؤثر سلبًا على استقرار نظام رفع الملفات ونتائج الاختبارات الآلية. من خلال إعادة هيكلة منطق الربط بين `Base64` و `FileUploadService` وإضافة خيارات تحكم جديدة، أصبحت عملية رفع الملفات للحقول الفردية (`AttachOne`) أكثر موثوقية وأقل عرضة للأخطاء. هذه التحسينات تعزز من قوة الإضافة وتجعلها مناسبة للاستخدام في بيئات الإنتاج والتطوير على حد سواء.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `Base64`](./docs/FileUpload/Docs-Base64-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)

## 2026-04-19 – 2026-04-20

**تحديثات إضافة `Nano3.Kyc` – الإصدارات 1.0.1 إلى 1.0.6**

### ملخص التحديثات

تم تطوير إضافة `Nano3.Kyc` لتكون الحل المتكامل لإدارة عمليات التحقق من الهوية (KYC) ضمن برمجيات نانوسوفت (NanoSoft App). تضمنت التحديثات في الإصدارات من 1.0.1 إلى 1.0.6 بناء نظام متكامل يشمل:

- **إنشاء هيكل قاعدة البيانات** وجدول `nano3_kyc_documents` المصمم لاستيعاب جميع أنواع وثائق KYC.
- **تطوير كلاس `DocumentType`** الذي يعرف أكثر من 28 نوع وثيقة مع خصائصها وحقولها الديناميكية وقواعد التحقق.
- **تطوير كلاس `Manager`** المسؤول عن جميع عمليات إدارة الوثائق (إنشاء، تحديث، حذف، اعتماد، رفض) مع دعم المعاملات والأحداث والاختبارات.
- **بناء موديل `Document`** مع مجموعة متكاملة من السمات (Traits) لإدارة النطاقات، الخيارات، التخزين المؤقت، والفلاتر.
- **تطوير واجهة API كاملة** عبر متحكم `Nano3\Kyc\APIControllers\Documents` بنمط RESTful.
- **تطوير واجهة خلفية متكاملة** عبر متحكم `Nano3\Kyc\Controllers\Documents` مع قوائم ونماذج وفلاتر ديناميكية.
- **دعم كامل لتعدد اللغات** (العربية والإنجليزية) مع ملف ترجمة شامل.
- **نظام تخصيص مرن** عبر ملف إعدادات `config/nano3/kyc/document_types.php`.

---

### الإصدار 1.0.1 – الإطلاق الأولي وإنشاء جدول الوثائق

#### أهداف الإصدار

- إنشاء الإضافة الأساسية `Nano3.Kyc` ضمن برمجيات نانوسوفت.
- تصميم وإنشاء جدول `nano3_kyc_documents` ليكون العمود الفقري لتخزين جميع وثائق KYC.

#### الميزات الجديدة

##### 1. إنشاء جدول `nano3_kyc_documents`

تم تصميم جدول متكامل يدعم جميع أنواع وثائق KYC مع مرونة عالية:

| مجموعة الحقول | الوصف |
| :--- | :--- |
| **المعرفات الأساسية** | `document_type`، `document_category`، `code`، `barcode`، `document_number` |
| **التواريخ** | `issue_date`، `expiry_date` (نوع `date` للأداء)، `verified_at` |
| **البيانات الشخصية** | `full_name`، `date_of_birth`، `nationality`، `country_of_issue`، `religion` |
| **العنوان** | `address_line1`، `address_line2`، `address_city`، `address_state`، `address_postcode`، `address_country` |
| **بيانات الشركات** | `company_name`، `registration_number`، `tax_number`، `incorporation_date` |
| **العلاقات المتعددة** | `owner_id`/`owner_type`، `verifier_id`/`verifier_type`، `subject_id`/`subject_type` |
| **التحقق** | `is_verified`، `verification_score`، `verified_at` |
| **النسخ الورقية** | `is_physical_submitted`، `physical_copy_type`، `physical_received_at`، `physical_returned_at`، `physical_notes` |
| **الحالة والنشر** | `status`، `is_active`، `is_default`، `is_public`، `is_published`، `published_at`، `unpublished_at` |
| **بيانات مرنة (JSON)** | `fields_data`، `metadata`، `other_data`، `config_data` |
| **الملفات** | `file_ids`، `file_urls` |
| **التتبع** | `timestamps`، `softDeletes`، `created_by`، `updated_by`، `deleted_by` |

تمت إضافة فهارس على الحقول الأكثر استخداماً في البحث (`document_type`، `document_number`، `full_name`، `company_name`، `status`، `is_verified`، العلاقات المتعددة) لضمان أداء عالي.

##### 2. هيكل الإضافة الأساسي

تم إنشاء الهيكل الأساسي للإضافة:
```
plugins/nano3/kyc/
├── Plugin.php              # ملف تعريف الإضافة
├── updates/
│   ├── version.yaml        # سجل الإصدارات
│   └── create_table_nano3_kyc_documents.php
└── lang/                   # مجلد الترجمة (سيتم ملؤه لاحقاً)
```

#### الفوائد

- قاعدة بيانات جاهزة لاستيعاب أي نوع وثيقة KYC حالية أو مستقبلية.
- مرونة عالية بفضل حقل `fields_data` (JSON) الذي يخزن الحقول المتخصصة لكل نوع وثيقة.
- فهارس محسنة للاستعلامات الشائعة.

---

### الإصدار 1.0.2 – تطوير كلاس `DocumentType` وموديل `Document` مع السمات الأساسية

#### أهداف الإصدار

- تطوير كلاس `DocumentType` ليكون المرجع المركزي لتعريف أنواع الوثائق وخصائصها.
- بناء موديل `Document` مع مجموعة السمات (Traits) الأساسية.

#### الميزات الجديدة

##### 1. كلاس `DocumentType`

تم تطوير كلاس `DocumentType` (`Nano3\Kyc\Classes\DocumentType`) الذي يوفر:

- **تعريف 28+ نوع وثيقة** موزعة على 6 فئات (هوية أساسية، هوية ثانوية، إثبات عنوان، وثائق شركات، UBO، تحقق إضافي).
- **خصائص متقدمة لكل نوع**: `has_expiry`، `max_age_days`، `verification_weight`، `allowed_mime_types`، `max_file_size_kb`، `allowed_for_entity`، `requires_back_side`، `requires_selfie_match`.
- **مخطط حقول ديناميكي (`fields`)**: تعريف كامل للحقول المطلوبة لكل نوع وثيقة مع خصائصها (`label`، `type`، `required`، `validation`، `order`، `extractable`).
- **دوال مساعدة للتحقق**: `getValidationRules()`، `isValidType()`، `isValidForPartyType()`، `isDocumentAcceptable()`، `isDocumentExpiring()`.
- **دوال لتحويل البيانات**: `mapInputToDocumentData()` و `mapDocumentToOutput()` لتسهيل التعامل مع هيكل التخزين المختلط (أعمدة ثابتة + JSON).
- **تحميل الإعدادات من ملف `config`**: مع إمكانية تجاوز القيم الافتراضية البرمجية.

**مثال على تعريف نوع وثيقة:**
```php
'passport' => [
    'category' => 'primary_id',
    'has_expiry' => true,
    'verification_weight' => 100,
    'allowed_mime_types' => ['image/jpeg', 'image/png', 'application/pdf'],
    'fields' => [
        'document_number' => ['type' => 'text', 'required' => true],
        'full_name' => ['type' => 'text', 'required' => true],
        'expiry_date' => ['type' => 'date', 'required' => true, 'validation' => ['after_or_equal' => 'today']],
        // ... إلخ
    ],
]
```

##### 2. موديل `Document` والسمات المرتبطة

تم بناء موديل `Document` (`Nano3\Kyc\Models\Document`) مع مجموعة سمات (Traits) تغطي جميع جوانب إدارة الوثائق:

| السمة | الوصف |
| :--- | :--- |
| `HasScopesModel` | نطاقات البحث الأساسية (`IsCompany`، `Departments`، `IsActive`، `IsPublished`، `WhereDocumentType`، ...). |
| `HasDefault` | إدارة السجل الافتراضي (`getDefault`، `makeDefault`). |
| `HasOwnerScopes` / `HasOwnerOptions` | نطاقات وخيارات فلترة المالك (`owner`). |
| `HasVerifierScopes` / `HasVerifierOptions` | نطاقات وخيارات فلترة المحقق (`verifier`). |
| `HasSubjectScopes` / `HasSubjectOptions` | نطاقات وخيارات فلترة الموضوع المرتبط (`subject`). |
| `HasRecordsOptions` | دالة `getRecords()` المتكاملة لجلب السجلات مع فلاتر مرنة وتنسيقات متعددة (مجموعة، paginator، query). |
| `ListObjects` / `ListOptions` | دوال مساعدة لجلب السجلات المخزنة مؤقتاً والقوائم المنسدلة. |
| `FieldsOptions` | دوال خيارات القوائم المنسدلة للنماذج (`getStatusOptions`، `getDocumentTypeOptions`، ...). |
| `HasCreateDefaultRecords` | دوال إنشاء السجلات الافتراضية (للتوافق مع هيكلية الإضافات الأخرى). |

##### 3. دالة `getRecords` المتكاملة

تدعم دالة `getRecords` في موديل `Document` خيارات فلترة شاملة:
- `id`، `document_type`، `document_category`، `document_number`، `full_name`، `company_name`، `status`، `is_verified`، `is_active`، `is_published`
- `owner_id`/`owner_type`، `verifier_id`/`verifier_type`، `subject_id`/`subject_type`
- نطاقات تاريخ (`issue_date`، `expiry_date`، `created_at`، `updated_at`)
- البحث النصي (`q`)
- دعم `is_paginator`، `is_collection`، `is_query`، `is_first`

#### الفوائد

- تعريف مركزي لجميع أنواع الوثائق يسهل الصيانة والتوسع.
- حقول ديناميكية تسمح ببناء نماذج إدخال تكيفية حسب نوع الوثيقة.
- تحويل تلقائي للبيانات بين المدخلات وقاعدة البيانات.
- موديل `Document` قوي مع نطاقات جاهزة للاستخدام في القوائم والفلاتر.
- تخزين مؤقت (Cache) لتحسين الأداء في `ListObjects` و `ListOptions`.

---

### الإصدار 1.0.3 – تطوير كلاس `Manager` (مدير العمليات)

#### أهداف الإصدار

- تطوير كلاس `Manager` (`Nano3\Kyc\Classes\Manager`) ليكون المسؤول عن جميع عمليات إدارة الوثائق.
- توفير دوال موحدة لإنشاء، تحديث، حذف، اعتماد، ورفض الوثائق مع دعم المعاملات، الأحداث، والاختبارات.

#### الميزات الجديدة

##### 1. كلاس `Manager` – مدير عمليات KYC

يوفر كلاس `Manager` الدوال التالية:

| الدالة | الوصف |
| :--- | :--- |
| `createDocument(array $options, bool $is_test_create)` | إنشاء وثيقة جديدة مع التحقق من الصلاحية، التكرار، والأحداث. |
| `updateDocument($documentId, array $options, bool $is_test_create)` | تحديث وثيقة موجودة. |
| `deleteDocument($documentId, bool $is_test_create)` | حذف وثيقة (Soft Delete). |
| `restoreDocument($documentId)` | استعادة وثيقة محذوفة. |
| `verifyDocument($documentId, $verifier, $verification_score)` | اعتماد وثيقة (تحديث الحالة إلى `verified`). |
| `rejectDocument($documentId, $reason)` | رفض وثيقة مع إمكانية إضافة سبب. |
| `checkDuplicateDocument(array $options)` | التحقق من وجود وثيقة مكررة لنفس المالك. |
| `getDocumentRecords($options)` | جلب سجلات الوثائق باستخدام دالة `getRecords` من الموديل. |
| `getDocumentStats($options)` | جلب إحصائيات عن الوثائق. |

##### 2. هيكل استجابة موحد

جميع دوال `Manager` تعيد هيكل استجابة موحد:

```php
[
    'code' => 200,
    'status' => true,
    'message' => 'رسالة النجاح',
    'error' => null,
    'errors' => null,
    'model' => $document,
    'data' => $document->toArray(),
    'input_data' => [...],
    'process_data' => [...]
]
```

##### 3. دعم الاختبارات (`is_test_create`)

جميع الدوال تدعم معامل `is_test_create` الذي يسمح بتنفيذ العملية ضمن معاملة يتم التراجع عنها (`rollback`) بعد التحقق من نجاحها، مما يسهل اختبار الوظائف دون التأثير على البيانات الحقيقية.

##### 4. دعم الأحداث والتحكم بها

- يمكن إيقاف إطلاق الأحداث عبر خيار `is_stop_event`.
- الأحداث المدعومة: `nano3.kyc.document.created`، `updated`، `deleted`، `restored`، `verified`، `rejected`.

##### 5. التحقق من التكرار

دالة `checkDuplicateDocument` تتحقق من وجود وثيقة مطابقة بناءً على حقول قابلة للتخصيص (`document_number`، `owner_id`، `owner_type`، `document_type`).

##### 6. دوال مساعدة للتوافق

- `getQueryDate()`: تطبيق فلتر تاريخ مرن على الاستعلامات.
- `checkValueIsNotAll()`: التحقق من أن القيمة ليست عامة (`*` أو `all`).
- `scopeWhereField()`: نطاق عام لتطبيق شروط `WHERE` مع دعم `is_or` و `is_not` و `is_force`.

#### الفوائد

- واجهة موحدة لجميع عمليات إدارة الوثائق.
- أمان عالي مع التحقق من الصلاحيات والتكرار.
- دعم كامل للمعاملات يضمن تكامل البيانات.
- سهولة الاختبار بفضل `is_test_create`.
- مرونة في تخصيص سلوك العمليات عبر الأحداث.

---

### الإصدار 1.0.4 – تطوير متحكم API ومتحكم الواجهة الخلفية

#### أهداف الإصدار

- تطوير متحكم API (`Nano3\Kyc\APIControllers\Documents`) لتوفير نقاط نهاية RESTful لإدارة الوثائق.
- تطوير متحكم الواجهة الخلفية (`Nano3\Kyc\Controllers\Documents`) لإدارة الوثائق من لوحة التحكم.

#### الميزات الجديدة

##### 1. متحكم API (`Documents`)

يوفر المتحكم نقاط النهاية التالية (المسار الأساسي: `/api/v1/kyc`):

| المسار | الطريقة | الوصف |
| :--- | :--- | :--- |
| `/documents` | `GET` | عرض قائمة الوثائق مع فلترة وبحث وتصفية. |
| `/documents` | `POST` | إنشاء وثيقة جديدة. |
| `/documents/{id}` | `GET` | عرض تفاصيل وثيقة. |
| `/documents/{id}` | `PUT` | تحديث وثيقة. |
| `/documents/{id}` | `DELETE` | حذف وثيقة. |
| `/documents/{id}/verify` | `POST` | اعتماد وثيقة. |
| `/documents/{id}/reject` | `POST` | رفض وثيقة. |
| `/documents/{id}/restore` | `POST` | استعادة وثيقة محذوفة. |
| `/document-types` | `GET` | جلب أنواع الوثائق المدعومة (مسطحة أو مجمعة). |
| `/document-types/{id}` | `GET` | جلب تفاصيل نوع وثيقة محدد. |
| `/document-fields/{type}` | `GET` | جلب حقول نوع وثيقة (لبناء نماذج ديناميكية). |
| `/documents/stats` | `GET` | جلب إحصائيات سريعة عن الوثائق. |
| `/documents/activelystats` | `GET` | التحقق من وجود تحديثات جديدة (للتخزين المؤقت). |

**مميزات متحكم API:**
- استخدام `Manager` لتنفيذ العمليات.
- دعم التخزين المؤقت (Cache) لتحسين الأداء.
- هيكل استجابة موحد متوافق مع `Nano.API`.
- معالجة أخطاء احترافية مع رسائل قابلة للترجمة.

##### 2. محول البيانات `DocumentTransformer`

تم تطوير `DocumentTransformer` (`Nano3\Kyc\Transformers\DocumentTransformer`) لتنسيق استجابات API، مع دعم:
- تضمين العلاقات (`owner`، `verifier`، `subject`، `company`، `department`).
- تنسيق التواريخ والحالات.
- معالجة حقول JSON (`fields_data`، `metadata`).
- استبعاد الحقول حسب الطلب (`exclude`).

##### 3. متحكم الواجهة الخلفية (`Documents`)

يوفر المتحكم إدارة كاملة للوثائق من لوحة التحكم:
- **قائمة الوثائق**: مع إمكانية البحث، التصفية، الفرز، والإجراءات الجماعية.
- **نموذج إنشاء/تعديل**: مقسم إلى تبويبات منظمة (أساسي، المالك، التحقق، النسخة الورقية، الملفات، النشر، العلاقات، التدقيق).
- **فلاتر متقدمة**: عبر ملف `config_filter.yaml` تشمل جميع حقول الجدول المهمة.
- **أعمدة قابلة للتخصيص**: عبر `columns.yaml`.

##### 4. ملفات التكوين للواجهة الخلفية

| الملف | الوصف |
| :--- | :--- |
| `config_filter.yaml` | تعريف فلاتر القائمة (نوع الوثيقة، الحالة، المالك، التواريخ، ...). |
| `columns.yaml` | تعريف أعمدة القائمة مع إمكانية البحث والفرز. |
| `fields.yaml` | تعريف حقول نموذج الإدخال مع تقسيمها إلى تبويبات. |

#### الفوائد

- واجهة API جاهزة للتكامل مع تطبيقات الجوال والويب.
- واجهة خلفية سهلة الاستخدام لإدارة الوثائق يدوياً.
- تنسيق موحد للبيانات عبر `DocumentTransformer`.
- فلاتر وأعمدة قابلة للتخصيص حسب احتياجات كل مشروع.

---

### الإصدار 1.0.5 – تحسين ملفات الترجمة والقوائم والنماذج

#### أهداف الإصدار

- إكمال ملف الترجمة العربية `lang.php` ليشمل جميع المفاتيح المستخدمة في المتحكمات والقوائم والنماذج.
- تحسين ملفات `columns.yaml` و `fields.yaml` لتوفير تجربة مستخدم محسنة في الواجهة الخلفية.

#### الميزات الجديدة

##### 1. ملف الترجمة الشامل `lang.php`

تم إعداد ملف ترجمة عربي كامل يحتوي على:

- **الأيقونات**: تعريف أيقونات القوائم والحالات.
- **القوائم**: عناوين القوائم الرئيسية والفرعية مع أوصافها.
- **الصلاحيات**: مفاتيح ونصوص الصلاحيات لجميع العمليات.
- **الرسائل**: رسائل النجاح والخطأ والتحذير لجميع العمليات.
- **ترجمات المتحكمات**: ترجمات خاصة بمتحكم `Documents` تشمل التبويبات، الأزرار، الرسائل، الفلاتر، حقول النماذج، وأعمدة القوائم.
- **ترجمات أنواع الوثائق**: فئات، أنواع، أوصاف، وحقول كل نوع وثيقة.

##### 2. تحسين `columns.yaml`

تم تحسين ملف أعمدة القائمة ليشمل:
- **أعمدة مرئية افتراضياً**: `id`، `document_type`، `document_number`، `full_name`، `company_name`، `owner`، `status`، `is_verified`، `issue_date`، `expiry_date`، `created_at`.
- **أعمدة مخفية قابلة للإظهار**: `code`، `document_category`، `verifier`، `verification_score`، `is_physical_submitted`، `departments_id`، `subject_name`، `updated_at`.
- **تنسيقات مخصصة**: استخدام `formatter` لعرض اسم نوع الوثيقة والمالك والمحقق بشكل مناسب.
- **دعم البحث والفرز** لمعظم الأعمدة.

##### 3. تحسين `fields.yaml`

تم تنظيم حقول نموذج الإدخال في تبويبات:

| التبويب | الحقول |
| :--- | :--- |
| **الأساسية** | `document_category`، `document_type`، `document_number`، `issuing_authority`، `issue_date`، `expiry_date` |
| **المالك** | `owner_type`، `owner_id` (مع دعم `@create` و `@update` للتحكم في قابلية التعديل) |
| **التحقق** | `verifier_type`، `verifier_id`، `is_verified`، `verification_score`، `verified_at` |
| **النسخة الورقية** | `is_physical_submitted`، `physical_copy_type`، `physical_received_at`، `physical_returned_at`، `physical_notes` |
| **الملفات** | `document_front`، `document_back`، `signature_image`، `image`، `files` |
| **النشر والحالة** | `status`، `is_published`، `published_at`، `unpublished_at`، `is_active`، `is_default`، `is_public` |
| **علاقات إضافية** | `subject_type`، `subject_id`، `extend_id`، `departments_id` |
| **التدقيق** | `created_by`، `updated_by`، `created_at`، `updated_at` (للقراءة فقط) |

**مميزات إضافية:**
- استخدام `dependsOn` لربط القوائم المنسدلة (مثلاً `owner_id` يعتمد على `owner_type`).
- استخدام `trigger` لإظهار/إخفاء حقول `published_at` و `unpublished_at` بناءً على حالة `is_published`.
- تنسيق متجاوب باستخدام `span` و `cssClass`.

#### الفوائد

- واجهة خلفية معربة بالكامل وجاهزة للاستخدام.
- تنظيم منطقي للحقول يسهل عملية إدخال البيانات.
- تجربة مستخدم سلسة مع القوائم المنسدلة المترابطة والحقول المشروطة.

---

### الإصدار 1.0.6 – تحسينات نهائية وتجهيز للإطلاق

#### أهداف الإصدار

- إجراء تحسينات نهائية على الكود والهيكل.
- إعداد ملفات `composer.json` و `README.md` للتوثيق والتوزيع.
- توحيد المخرجات والتأكد من جاهزية الإضافة للاستخدام الإنتاجي.

#### الميزات الجديدة

##### 1. تحسينات على `Manager` و `DocumentType`

- تحسين دالة `createDocument` لتشمل معالجة الملفات (`files`) وبيانات الحقول (`fields`) بشكل منفصل.
- إضافة دالة `prepareDocumentOptions` لتجهيز القيم الافتراضية للشركة والفرع.
- تحسين معالجة الأخطاء وإضافة تفاصيل التصحيح (`debug`) في بيئة التطوير.

##### 2. ملف `composer.json`

تم إنشاء ملف `composer.json` بالإعدادات التالية:
- **الاسم**: `nano3/kyc`
- **النوع**: `nano-extension` (خاص ببرمجيات نانوسوفت)
- **الاعتماديات**: `php >= 8.0`، `nano/api`، `tss/basic`
- **الترخيص**: `proprietary`

##### 3. ملف `README.md`

تم إعداد ملف `README.md` احترافي يحتوي على:
- وصف عام للإضافة ومميزاتها.
- قائمة أنواع الوثائق المدعومة.
- متطلبات النظام والتثبيت.
- أمثلة استخدام لكل من API وكلاسات `Manager` و `DocumentType`.
- شرح هيكل قاعدة البيانات.
- معلومات الترخيص والدعم.

##### 4. توثيق التحديثات

تم إعداد ملف التوثيق هذا (`Update-Nano3-Kyc-ar.md`) ليغطي جميع الإصدارات من 1.0.1 إلى 1.0.6 بتنسيق احترافي متوافق مع معايير توثيق برمجيات نانوسوفت.

#### الفوائد

- جاهزية الإضافة للتوزيع والاستخدام في المشاريع الإنتاجية.
- توثيق شامل يسهل على المطورين فهم الإضافة واستخدامها.
- هيكل متكامل يدعم الصيانة والتطوير المستقبلي.

---

### ملخص الإصدارات (1.0.1 – 1.0.6)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.0.1 | إنشاء جدول `nano3_kyc_documents` والهيكل الأساسي للإضافة. |
| 1.0.2 | تطوير كلاس `DocumentType` وموديل `Document` مع السمات الأساسية. |
| 1.0.3 | تطوير كلاس `Manager` (مدير العمليات) مع دعم المعاملات والأحداث والاختبارات. |
| 1.0.4 | تطوير متحكم API ومتحكم الواجهة الخلفية مع ملفات التكوين. |
| 1.0.5 | تحسين ملفات الترجمة والقوائم والنماذج (`lang.php`، `columns.yaml`، `fields.yaml`). |
| 1.0.6 | تحسينات نهائية، إعداد `composer.json` و `README.md`، وتوثيق التحديثات. |

---

### متطلبات الترقية

1. **تشغيل الهجرات**:
   ```bash
   php artisan nano:up
   ```
   أو تنفيذ هجرة `create_table_nano3_kyc_documents.php` يدوياً.

2. **نشر ملف الإعدادات** (اختياري):
   ```bash
   php artisan config:publish nano3.kyc
   ```
   لتخصيص خصائص أنواع الوثائق عبر `config/nano3/kyc/document_types.php`.

3. **إعداد الصلاحيات**:
   - منح الصلاحيات المناسبة للأدوار (`documents.access`، `documents.verify`، إلخ) من لوحة التحكم.

4. **تحديث تعريفات API** (إذا كنت تستخدم `Nano.API`):
   - تأكد من أن مسارات API مضمنة في ملف التوجيه الرئيسي.

---

### الخاتمة

تم تطوير إضافة `Nano3.Kyc` لتكون حلاً متكاملاً ومرناً لإدارة عمليات التحقق من الهوية ضمن برمجيات نانوسوفت. بفضل هيكلها القابل للتوسع، ودعمها لأكثر من 28 نوع وثيقة، وواجهة API المتكاملة، وواجهة الإدارة السهلة، يمكن للمطورين دمجها بسرعة في أي مشروع يتطلب التحقق من هوية المستخدمين أو الشركات.

نتطلع إلى مواصلة تطوير الإضافة بناءً على ملاحظات المستخدمين والمتطلبات المتطورة، مع خطط مستقبلية تشمل:
- دعم التعرف الضوئي على الحروف (OCR) لاستخراج البيانات تلقائياً.
- تكامل مع خدمات التحقق من الوثائق الخارجية.
- لوحات تحكم تحليلية متقدمة.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

## 2026-04-20

**تحديثات إضافة `Nano3.Kyc` – الإصدارات 1.0.7 إلى 1.1.0**

### ملخص التحديثات

بعد الإطلاق الرسمي للإضافة في الإصدارات السابقة، واصلنا تطوير `Nano3.Kyc` لإضافة ميزات متقدمة تمكن المطورين من تقييم حالة KYC للمستخدمين والجهات المختلفة بسهولة ومرونة. تضمنت التحديثات في الإصدارات من 1.0.7 إلى 1.1.0:

- **إضافة دوال تقييم KYC شاملة** في `Manager` (`assessKycStatus` و `assessKycStatusByCategory`).
- **تطوير سلوك `DynamicAddIncludeKyc`** لحقن بيانات KYC مباشرة في استجابات API عبر `include`.
- **توسيع دعم أنواع المستخدمين** في `DocumentType` ليشمل مستخدمي `backend` و `frontend` كأفراد.
- **تحسين آلية تحديد الوثائق الإلزامية** بناءً على التصنيف (`category`) ونوع الوثيقة (`document_type`).

---

### الإصدار 1.0.7 – إضافة دوال تقييم حالة KYC في `Manager`

#### أهداف الإصدار

- توفير آلية متكاملة لتقييم حالة KYC لمالك معين (فرد/شركة).
- حساب نسبة الاكتمال، الوثائق المتحقق منها، المفقودة، المنتهية، والدرجة الإجمالية.
- دعم فلترة حسب الفئة (`category`) ونوع الوثيقة (`document_type`) ومستوى المخاطرة (`risk_level`).

#### الميزات الجديدة

##### 1. دالة `assessKycStatus` في `Manager`

تمت إضافة دالة `Manager::assessKycStatus` التي تقوم بتقييم شامل لحالة KYC لمالك معين.

**توقيع الدالة:**
```php
public static function assessKycStatus($owner, ?string $ownerType = null, ?int $ownerId = null, array $options = []): array
```

**الخيارات المدعومة:**
| الخيار | الوصف | الافتراضي |
| :--- | :--- | :--- |
| `category` | فلترة حسب فئة وثيقة محددة (`primary_id`, `address`, `corporate`, ...). | `null` |
| `document_type` | فلترة حسب نوع وثيقة محدد (`passport`, `utility_bill`, ...). | `null` |
| `risk_level` | مستوى المخاطرة لتحديد المتطلبات الإلزامية (`low`, `medium`, `high`). | `medium` |
| `include_expired` | تضمين الوثائق المنتهية في النتائج. | `true` |
| `include_pending` | تضمين الوثائق قيد الانتظار. | `true` |
| `include_rejected` | تضمين الوثائق المرفوضة. | `false` |

**هيكل الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "is_compliant": false,
        "completion_percentage": 0,
        "overall_score": 0,
        "total_documents_required": 0,
        "verified_documents_count": 0,
        "pending_documents_count": 0,
        "expired_documents_count": 0,
        "rejected_documents_count": 0,
        "verified_documents": [],
        "pending_documents": [],
        "expired_documents": [],
        "rejected_documents": [],
        "missing_required_types": [],
        "recommendations": []
    }
}
```

##### 2. آلية تحديد الوثائق الإلزامية والموصى بها

تستخدم الدالة دوال `DocumentType` التالية:
- `getMandatoryTypesForParty($ownerType, $riskLevel)`: لتحديد أنواع الوثائق الإلزامية بناءً على نوع المالك ومستوى المخاطرة.
- `getRecommendedTypes($ownerType, $riskLevel)`: لتحديد أنواع الوثائق الموصى بها.

##### 3. التحقق من صلاحية الوثائق

تقوم الدالة بالتحقق من صلاحية كل وثيقة متحقق منها (`verified`) باستخدام:
- `DocumentType::hasExpiry()` و `DocumentType::isDocumentAcceptable()` للوثائق ذات تاريخ انتهاء.
- `DocumentType::getMaxAgeDays()` و `DocumentType::isDocumentAcceptable()` للوثائق محدودة العمر.

#### الفوائد

- تقييم موحد وشامل لحالة KYC يمكن استخدامه في عمليات اتخاذ القرار (مثل السماح بإجراء معاملات).
- مرونة عالية في تخصيص التقييم حسب الفئة أو نوع الوثيقة أو مستوى المخاطرة.
- توصيات تلقائية بالوثائق المفقودة أو التي تحتاج تجديد.

---

### الإصدار 1.0.8 – سلوك `DynamicAddIncludeKyc` لحقن بيانات KYC في API

#### أهداف الإصدار

- تمكين استجابات API من تضمين بيانات KYC المرتبطة بالكائن (مستخدم، طلب، منتج) بسهولة عبر `include`.
- توفير آلية مرنة لحقن دوال `include` في أي `Transformer` دون تعديل الكود الأصلي.
- دعم التقييم (`kyc_status`) وقائمة الوثائق (`kyc_documents`) وعدد الوثائق (`kyc_documents_count`).

#### الميزات الجديدة

##### 1. سلوك `DynamicAddIncludeKyc`

تم إنشاء كلاس `DynamicAddIncludeKyc` (`Nano3\Kyc\Behaviors\DynamicAddIncludeKyc`) الذي يقوم بحقن الدوال التالية في أي `Transformer`:

| الدالة | الوصف |
| :--- | :--- |
| `includeKycStatus` | تقييم حالة KYC للمالك (تستخدم `Manager::assessKycStatus`). |
| `includeKycDocuments` | قائمة وثائق المالك مع دعم pagination. |
| `includeKycDocumentsCount` | عدد وثائق المالك. |

**طريقة الاستخدام في طلب API:**
```
GET /api/v1/me?include=kyc_status,kyc_documents,kyc_documents_count
```

##### 2. دعم خيارات متقدمة عبر query parameters

يمكن تمرير خيارات إضافية لكل `include`:

```
?include=kyc_status&kyc_status[risk_level]=high&kyc_status[category]=primary_id
?include=kyc_documents&kyc_documents[per_page]=20&kyc_documents[status]=verified
```

##### 3. حقن السلوك في Transformers متعددة

تم تحديث `Plugin::boot()` ليشمل حقن السلوك في Transformers التالية:
- `UserTransformer` (مستخدمي الموقع)
- `AdminTransformer` (مستخدمي لوحة التحكم)
- `DeliveryTransformer` (مندوبي التوصيل)
- `ParentTransformer` (أولياء الأمور)
- `StudentTransformer` (الطلاب)
- `DepartmentTransformer` (الفروع والأقسام)

#### الفوائد

- إثراء استجابات API ببيانات KYC دون الحاجة لطلبات منفصلة.
- تكامل سلس مع نظام `Nano.API` و `Fractal`.
- قابلية التوسع لإضافة Transformers جديدة بسهولة.

---

### الإصدار 1.0.9 – إضافة `assessKycStatusByCategory` وتحديث السلوك

#### أهداف الإصدار

- توفير دالة تقييم KYC تعتمد مباشرة على تصنيف الوثائق (`category`) دون الحاجة لاستخدام `risk_level` أو `getMandatoryTypesForParty`.
- إضافة `include` جديد باسم `kyc_status_by_category` في السلوك لاستخدام هذه الدالة.
- تحسين المرونة في تقييم فئات محددة من الوثائق.

#### الميزات الجديدة

##### 1. دالة `assessKycStatusByCategory` في `Manager`

تمت إضافة دالة جديدة تسمح بتقييم KYC بناءً على تصنيف الوثائق مباشرة.

**توقيع الدالة:**
```php
public static function assessKycStatusByCategory(
    $owner,
    ?string $ownerType = null,
    ?int $ownerId = null,
    ?string $category = null,
    $documentType = null,
    array $options = []
): array
```

**المعاملات:**
| المعامل | الوصف |
| :--- | :--- |
| `$category` | تصنيف الوثائق (إلزامي، افتراضي `CATEGORY_PRIMARY_ID`). |
| `$documentType` | نوع وثيقة محدد أو مصفوفة من الأنواع (اختياري). |

**آلية العمل:**
- تبني قائمة الوثائق المطلوبة مباشرة من جميع أنواع الوثائق التي تنتمي إلى `$category`.
- إذا تم تمرير `$documentType`، يتم تضييق النطاق ليشمل فقط الأنواع المحددة.
- لا تعتمد على `risk_level` أو `getMandatoryTypesForParty`.

##### 2. تحديث `DynamicAddIncludeKyc` بإضافة `includeKycStatusByCategory`

تمت إضافة دالة `includeKycStatusByCategory` إلى السلوك، والتي تستخدم `Manager::assessKycStatusByCategory`.

**مثال للاستخدام:**
```
GET /api/v1/me?include=kyc_status_by_category&kyc_category=address&kyc_document_type[]=utility_bill&kyc_document_type[]=bank_statement
```

**الخيارات المدعومة:**
| الخيار | الوصف |
| :--- | :--- |
| `category` | تصنيف الوثائق (مطلوب، افتراضي `primary_id`). |
| `document_type` | نوع وثيقة محدد أو مصفوفة. |
| `include_expired` | تضمين الوثائق المنتهية. |
| `include_pending` | تضمين الوثائق قيد الانتظار. |
| `include_rejected` | تضمين الوثائق المرفوضة. |

#### الفوائد

- تقييم دقيق لفئة معينة من الوثائق (مثل "إثبات العنوان") دون الحاجة لتحديد مستوى مخاطرة.
- مثالية للتحقق من متطلبات محددة (مثل: "هل لدى المستخدم إثبات عنوان ساري؟").
- تكامل كامل مع نظام `include` في API.

---

### الإصدار 1.1.0 – تحسينات على `DocumentType` ودعم أنواع المستخدمين

#### أهداف الإصدار

- توسيع دالة `getMandatoryTypesForParty` لدعم مستخدمي `backend` و `frontend` كأفراد (`individual`).
- ضمان أن تقييم KYC يعمل بشكل صحيح لجميع أنواع المستخدمين في النظام.
- تحسينات عامة على الكود وتوحيد المخرجات.

#### الميزات الجديدة

##### 1. تحديث `DocumentType::getMandatoryTypesForParty`

تم تعديل الدالة لتشمل تحويل أنواع المستخدمين التالية إلى `individual`:
- `RainLab\User\Models\User` (مستخدمي الموقع)
- `Backend\Models\User` (مستخدمي لوحة التحكم)

**الكود قبل التحديث:**
```php
if ($partyType === 'individual') {
    $mandatory[] = self::TYPE_NATIONAL_ID;
}
```

**الكود بعد التحديث:**
```php
if (in_array($partyType, ['RainLab\User\Models\User', 'Backend\Models\User'])) {
    $partyType = 'individual';
}

if ($partyType === 'individual') {
    $mandatory[] = self::TYPE_NATIONAL_ID;
    if (in_array($riskLevel, ['high', 'very_high'])) {
        $mandatory[] = self::TYPE_PASSPORT;
    }
}
```

##### 2. تأثير التحديث على تقييم KYC

- **قبل التحديث**: مستخدمو `backend` لم تكن لديهم أي وثائق إلزامية (`total_documents_required = 0`).
- **بعد التحديث**: يتم التعامل معهم كأفراد، وبالتالي يُطلب منهم `national_id` كحد أدنى (و `passport` في حالات المخاطرة العالية).

##### 3. تحسينات عامة

- توحيد هيكل الاستجابة في جميع دوال `Manager`.
- إضافة تفاصيل تصحيح (`debug`) في بيئة التطوير.
- تحسين معالجة الأخطاء في `DynamicAddIncludeKyc`.

#### الفوائد

- تغطية شاملة لجميع أنواع المستخدمين في النظام.
- سلوك متوقع ومنطقي لتقييم KYC.
- جاهزية الإضافة للاستخدام في سيناريوهات واقعية متنوعة.

---

### ملخص الإصدارات (1.0.7 – 1.1.0)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.0.7 | إضافة `assessKycStatus` في `Manager` لتقييم شامل لحالة KYC. |
| 1.0.8 | تطوير سلوك `DynamicAddIncludeKyc` لحقن بيانات KYC في Transformers API. |
| 1.0.9 | إضافة `assessKycStatusByCategory` ودعم `kyc_status_by_category` في API. |
| 1.1.0 | تحديث `DocumentType` لدعم مستخدمي `backend` و `frontend` كأفراد في التقييم. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملفات `Manager.php`، `DocumentType.php`، `DynamicAddIncludeKyc.php`، و `Plugin.php` بالنسخ الجديدة.

2. **لا توجد هجرات جديدة**:
   - لا يتطلب هذا الإصدار تغييرات في قاعدة البيانات.

3. **تحديث تعريفات API** (اختياري):
   - للاستفادة من `include` الجديدة، تأكد من أن العملاء (Clients) يستخدمون المسارات المحدثة.

4. **اختبار التوافق**:
   - يُنصح باختبار دوال التقييم مع أنواع المستخدمين المختلفة للتأكد من النتائج المتوقعة.

---

### الخاتمة

مع الإصدارات 1.0.7 إلى 1.1.0، أصبحت إضافة `Nano3.Kyc` أكثر ذكاءً وتكاملاً. يمكنها الآن:

- **تقييم حالة KYC** بشكل شامل مع توصيات تلقائية.
- **التكامل مع API** بسلاسة عبر `include` دون الحاجة لطلبات منفصلة.
- **دعم جميع أنواع المستخدمين** في النظام بشكل موحد.
- **توفير مرونة عالية** في تقييم فئات محددة من الوثائق.

هذه التحسينات تجعل الإضافة أداة قوية لفرض سياسات KYC واتخاذ القرارات المستندة إلى حالة التحقق من الهوية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

## 2026-04-20

**تحديثات إضافة `Nano3.Kyc` – الإصدارات 1.1.1 إلى 1.2.0**

### ملخص التحديثات

بعد إضافة دوال تقييم KYC ونقاط نهاية التقييم في الإصدارات السابقة، ركزت التحديثات الجديدة على **تمكين رفع الملفات وإدارتها** عبر التكامل الكامل مع حزمة `Nano.FileUpload`، بالإضافة إلى **تطوير نظام اختبار شامل** يضمن استقرار الإضافة ويسهل اكتشاف الأخطاء. تشمل التحديثات في الإصدارات من 1.1.1 إلى 1.2.0:

- **دمج رفع الملفات في واجهة API**: دعم رفع `document_front`, `document_back`, `signature_image`, `image`, `files` أثناء إنشاء وتحديث الوثائق.
- **حماية الملفات بعد الاعتماد**: منع تغيير أو حذف ملفات الوثيقة بعد اعتمادها (`is_verified = true`).
- **تسجيل موديل `Document` في `FileUploadRegistry`**: لتمكين صلاحيات الرفع المتقدمة وإدارة الملفات المؤقتة.
- **إضافة كلاس اختبار شامل `KycPlusTest`**: يغطي اختبارات `DocumentType`، `Manager`، رفع الملفات، تقييم KYC، وصلاحيات API.
- **نقطة نهاية `GET /tests`**: لتشغيل مجموعة الاختبارات في بيئة التطوير وعرض النتائج بشكل موحد.
- **تحسينات عامة على معالجة الأخطاء ورسائل الترجمة**.

---

### الإصدار 1.1.1 – تحسينات طفيفة على نقاط نهاية التقييم

#### أهداف الإصدار

- تثبيت نقاط النهاية `assessKycStatus` و `assessKycStatusByCategory` التي أضيفت في 1.1.0.
- تحسين معالجة خيارات `include_expired`, `include_pending`, `include_rejected` مع دعم القيم النصية (`"true"`/`"false"`).

#### الميزات الجديدة

- تحويل تلقائي لخيارات التضمين من نصوص إلى قيم منطقية، مما يسمح باستخدامها بسهولة في query strings.

---

### الإصدار 1.1.2 – دعم رفع الملفات في API مع حماية الوثائق المعتمدة

#### أهداف الإصدار

- تمكين المستخدمين من رفع ملفات الهوية (صور، PDF، توقيع) مباشرة عبر نقاط النهاية `create` و `update`.
- منع أي تعديل على ملفات الوثيقة بعد اعتمادها (`is_verified = true`)، حفاظاً على سلامة بيانات KYC.
- تسجيل نموذج `Document` في `FileUploadRegistry` بشكل آلي.

#### الميزات الجديدة

##### 1. سمة `FileUploadDocuments`

تم إنشاء سمة جديدة `Nano3\Kyc\APIControllers\Documents\FileUploadDocuments` تحتوي على الدوال:

| الدالة | الوصف |
| :--- | :--- |
| `extractUploadedFiles()` | استخراج الملفات من بيانات الطلب (سواء `UploadedFile` أو base64). |
| `attachFilesToDocument()` | رفع الملفات عبر `FileUploadService` وربطها بالوثيقة. |
| `documentHasFile()` | التحقق من وجود ملف مرتبط بحقل معين. |
| `registerDocumentModel()` | تسجيل موديل `Document` في `FileUploadRegistry`. |

##### 2. تحديث دالتي `create` و `update`

- **`create`**: بعد إنشاء الوثيقة، يتم رفع الملفات المرفقة وربطها. في حالة فشل رفع بعض الملفات، لا تفشل العملية كاملة، بل يتم إرجاع تحذير مع تفاصيل الأخطاء.
- **`update`**: قبل السماح برفع ملفات جديدة، يتم التحقق من حالة الوثيقة. إذا كانت `is_verified = true` ولديها ملف مرتبط مسبقاً، يتم رفض الطلب مع رسالة خطأ مناسبة.

##### 3. تسجيل `Document` في `FileUploadRegistry`

في `Plugin::boot()`، يتم استدعاء `registerDocumentInFileUpload()` التي تسجل الحقول التالية:
- `document_front`, `document_back`, `signature_image`, `image` (نوع `image`، حد أقصى 10MB للصور).
- `files` (نوع `multiple`، يسمح بعدة ملفات).

هذا يمكن `FileUploadService` من تطبيق صلاحيات الرفع والتحقق من القيود تلقائياً.

#### الفوائد

- تجربة مستخدم موحدة لرفع ملفات KYC دون الحاجة لاستدعاء منفصل لواجهة `FileUpload`.
- حماية قوية للوثائق المعتمدة من التلاعب.
- تكامل سلس مع نظام الصلاحيات والقيود الخاص بـ `Nano.FileUpload`.

---

### الإصدار 1.1.3 – تحسينات على رفع الملفات ورسائل الترجمة

#### أهداف الإصدار

- تحسين التعامل مع الملفات المرفوعة بصيغة base64.
- إضافة مفاتيح ترجمة لرسائل النجاح الجزئي وحماية الملفات المعتمدة.
- معالجة أفضل للأخطاء في رفع الملفات.

#### الميزات الجديدة

- دعم الحقول التي تنتهي بـ `_base64` لاستقبال الملفات المشفرة.
- مفاتيح ترجمة جديدة:
  - `public.helpers.create_document.msg_files_partial`
  - `public.helpers.update_document.msg_files_partial`
  - `public.helpers.update_document.msg_cannot_change_verified_files`
- إرجاع تفاصيل أخطاء الرفع في `process_data.file_errors`.

---

### الإصدار 1.2.0 – نظام اختبار متكامل (`KycPlusTest`)

#### أهداف الإصدار

- توفير آلية اختبار شاملة تغطي جميع مكونات الإضافة (DocumentType, Manager, FileUpload, API).
- تمكين المطورين من تشغيل الاختبارات بسهولة عبر واجهة API في بيئة التطوير.
- ضمان استقرار الإضافة عند إجراء تغييرات مستقبلية.

#### الميزات الجديدة

##### 1. كلاس `KycPlusTest`

كلاس اختبار يقع في `Nano3\Kyc\Tests\KycPlusTest` ويغطي:

- **DocumentType**: تسجيل الأنواع، جلب الحقول، قواعد التحقق.
- **Manager**: إنشاء، تحديث، اعتماد، رفض، حذف، استعادة، جلب السجلات، التحقق من التكرار.
- **رفع الملفات**: إنشاء وتحديث وثائق مع ملفات، منع تغيير ملفات الوثائق المعتمدة.
- **تقييم KYC**: `assessKycStatus` و `assessKycStatusByCategory`.
- **الصلاحيات والعزل**: التحقق من مصادقة API وعزل بيانات المالك.

**هيكل نتائج الاختبارات:**
كل اختبار يعيد مصفوفة تحتوي على:
```json
{
    "code": 200,
    "status": true,
    "test_code": "DOCTYPE_REG_001",
    "name": "تسجيل أنواع الوثائق",
    "description": "...",
    "message": "Test passed",
    "data": {...},
    "input_data": {...},
    "process_data": {...}
}
```

##### 2. نقطة نهاية `GET /api/v1/kyc/tests`

تتيح تشغيل مجموعة الاختبارات وعرض النتائج. **متاحة فقط** عندما يكون `app.debug = true` والمستخدم من نوع `backend`.

**معاملات الاستعلام:**
| المعامل | الوصف |
| :--- | :--- |
| `test_version` | `v1` أو `v2` (الافتراضي `v2`). |
| `filter` | `all`, `passed`, `failed` (الافتراضي `all`). |

**مثال للطلب:**
```
GET /api/v1/kyc/tests?test_version=v2&filter=passed
```

**الاستجابة:** تحتوي على `data` (نتائج الاختبارات المصفاة) و `meta.summary` بإحصائيات (عدد الناجح، الفاشل، نسبة النجاح).

##### 3. عزل بيئة الاختبار

تستخدم الاختبارات `Db::beginTransaction()` و `Db::rollBack()` لضمان عدم تأثيرها على قاعدة البيانات الحقيقية. كما يتم تنظيف الملفات المرفوعة مؤقتاً.

#### الفوائد

- اكتشاف الأخطاء مبكراً قبل النشر.
- توثيق حي لسلوك الإضافة من خلال الاختبارات.
- سهولة التحقق من صحة التثبيت في بيئات مختلفة.

---

### ملخص الإصدارات (1.1.1 – 1.2.0)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.1.1 | تحسين نقاط نهاية التقييم. |
| 1.1.2 | دعم رفع الملفات في API، منع تغيير ملفات الوثائق المعتمدة، تسجيل Document في FileUploadRegistry. |
| 1.1.3 | تحسين base64 ورسائل الترجمة. |
| 1.2.0 | إضافة نظام اختبار شامل (`KycPlusTest`) ونقطة نهاية `/tests`. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال الملفات التالية بالنسخ الجديدة:
     - `Plugin.php`
     - `apicontrollers/Documents.php`
     - `apicontrollers/documents/FileUploadDocuments.php`
     - `tests/KycPlusTest.php` (جديد)

2. **إضافة مسار الاختبارات** في `routes.php`:
   ```php
   Route::get('tests', 'Documents@tests');
   ```

3. **تحديث ملف الترجمة** `lang.php` بالمفاتيح الجديدة (انظر الإصدار 1.1.3).

4. **تشغيل الاختبارات** (في بيئة التطوير):
   ```
   GET /api/v1/kyc/tests
   ```
   للتأكد من أن الترقية تمت بنجاح.

5. **لا توجد هجرات جديدة** في هذه الإصدارات.

---

### الخاتمة

مع الإصدارات 1.1.1 إلى 1.2.0، أصبحت إضافة `Nano3.Kyc` متكاملة تماماً مع نظام رفع الملفات `Nano.FileUpload`، مما يوفر تجربة سلسة لإدارة وثائق KYC مع حماية متقدمة للبيانات المعتمدة. كما أن إضافة نظام اختبار شامل يعزز الثقة في استقرار الإضافة ويسهل صيانتها وتطويرها مستقبلاً. هذه التحسينات تجعل الإضافة جاهزة للاستخدام في المشاريع الإنتاجية مع ضمان أعلى معايير الجودة.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)
- [توثيق السلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)

## 2026-03-10 - 2026-04-20

### Support Dynamic Reporting API 

**تحديث نظام التقارير الديناميكي: إطلاق الإصدار 2.0 مع دعم التقارير المسجلة مسبقاً وواجهة API متكاملة**

الإصدار المُحسَّن من نظام التقارير الديناميكي – إضافة التقارير المسجلة مسبقاً، واجهة برمجية متكاملة، وتحسينات الأداء والأمان  

---

### 1. مقدمة

تم إتمام المرحلة الثانية من تطوير مشروع **نظام التقارير الديناميكي** `Nano2.QueryBuilder.Reporting`، والتي تضمنت إضافات جوهرية وتوسعات كبيرة في النظام، ليرتقي إلى مصاف أدوات التقارير الاحترافية. استجابةً لملاحظات العملاء والمطورين الداخليين، ركزنا في هذه المرحلة على تمكين **التقارير المسجلة مسبقاً** (Predefined Reports / Templates)، وتوفير **واجهة برمجية (API) متكاملة** للتعامل معها، مع تحسين الأداء والأمان وتجربة المستخدم.

يأتي هذا التحديث ليضيف طبقة جديدة فوق البنية الأساسية السابقة، مما يجعل النظام أكثر مرونة وقابلية للتوسع، ويفتح الباب أمام المطورين لبناء تطبيقات تقارير متقدمة بسرعة وأمان. تم تطوير جميع المكونات وفق أفضل الممارسات في تصميم البرمجيات، مع الحفاظ على التوافقية مع الإصدارات السابقة.

---

### 2. أهداف التحديثات

- **تمكين تسجيل التقارير الجاهزة**: إضافة آلية تسمح للإضافات (Plugins) بتسريب تقارير ثابتة (قوالب) مرة واحدة، ثم استخدامها عبر واجهات برمجية موحدة.
- **توفير واجهة برمجية (API) شاملة**: إنشاء متحكم `DynamicReports` يوفر نقاط نهاية RESTful لإدارة وتنفيذ التقارير (مسجلة أو ديناميكية) مع دعم الفلاتر والتصفح والتصدير.
- **إضافة دعم التصفح (Pagination)** للتقارير المسجلة والديناميكية، مع إمكانية تحديد عدد السجلات لكل صفحة والحدود القصوى.
- **تحسين نظام الأمان** على مستوى الوصول إلى الجداول والتقارير، باستخدام كلاس موحد `ReportUserManager` يدعم أنظمة المصادقة المختلفة (OctoberCMS Backend, RainLab.User, Laravel Auth).
- **تحسين معالجة الأخطاء** لعرض رسائل آمنة في بيئة الإنتاج مع إخفاء تفاصيل SQL الحساسة، مع الاحتفاظ بمعلومات التصحيح الكاملة في وضع التطوير.
- **توفير مرونة في تحديد حقل الشركة** عبر إضافة خاصية `company_key` في تعريفات الجداول، مما يلغي افتراض `company_uuid` الثابت.
- **توثيق كامل** للمتحكم `DynamicReports` مع أمثلة عملية باللغتين العربية والإنجليزية.

---

### 3. المكونات المطورة والمحسّنة

#### 3.1 السمة `HasStoredReports`
تمت إضافة سمة جديدة إلى `ReportsManager` تسمح بتسجيل التقارير الجاهزة وإدارتها. توفر هذه السمة الدوال التالية:

- `registerReport(array $definition)` – تسجيل تقرير واحد.
- `registerReports(array $reports)` – تسجيل مجموعة تقارير.
- `getStoredReport(string $reportId)` – الحصول على تعريف تقرير.
- `getAvailableStoredReports($user = null)` – الحصول على التقارير المتاحة للمستخدم مع مراعاة الصلاحيات.
- `runStoredReport(string $reportId, array $filterValues, array $options)` – تشغيل تقرير مسجل بعد دمج الفلاتر وخيارات التصفح.

كما تضمنت السمة آليات لتطبيع تعريف التقرير، والتحقق من الفلاتر الإجبارية، وتطبيق إعدادات التصفح.

#### 3.2 متحكم `DynamicReports`
تم إنشاء متحكم جديد ضمن مسار `api/v1/querybuilder` يوفر نقاط النهاية التالية:

| المسار | الطريقة | الوصف |
|--------|--------|--------|
| `dynamic-reports` | GET | قائمة التقارير المسجلة المتاحة للمستخدم الحالي |
| `dynamic-reports/{reportId}` | GET | عرض تفاصيل تقرير معين (بما في ذلك الأعمدة والفلاتر) |
| `dynamic-reports/run/{reportId}` | POST | تشغيل تقرير مسجل مع تمرير الفلاتر وخيارات التصفح |
| `dynamic-reports/execute` | POST | تشغيل تقرير ديناميكي (غير مسجل) بتكوين مخصص |
| `dynamic-reports/export` | POST | تصدير نتائج تقرير (مسجل أو ديناميكي) بصيغة محددة (CSV, Excel, JSON, PDF, XML) |
| `dynamic-reports/schema/{tableName}` | GET | الحصول على مخطط جدول (الأعمدة المتاحة مع مسارات العلاقات) |
| `dynamic-reports/validate` | GET/POST | التحقق من صحة تكوين تقرير دون تنفيذه |
| `dynamic-reports/estimate` | GET/POST | تقدير تكلفة استعلام (التعقيد، الأداء المتوقع) |

تم بناء المتحكم على نفس نمط متحكمات API الأخرى (مثل `CatchReceipts`) مع دوال `init`، `successResponse`، `errorResponse`، ومعالجة استثناءات موحدة.

#### 3.3 تحسينات في `ReportQueryConverter`
- **دعم حقل الشركة الديناميكي**: أصبح `applyCompanyScope` يقرأ اسم عمود الشركة من كائن `Table` (عبر `getCompanyKey()`)، مما يسمح باستخدام حقول مثل `companys_id` بدلاً من `company_uuid`.
- **إضافة `exposeMetaDetails`**: معامل جديد في الـ constructor يتحكم في عرض التفاصيل الحساسة (استعلام SQL، bindings، joins) في `meta`. يتم تعيينه تلقائياً من `ReportsManager` أو يمكن تمريره عبر `options`.
- **حساب العدد الإجمالي للصفحات**: تمت إضافة دالة `getTotalCount` لحساب إجمالي الصفوف لاستعلامات التصفح باستخدام `COUNT DISTINCT` على المفتاح الأساسي.
- **بناء معلومات التصفح**: دالة `buildPaginationMeta` تولد كائن pagination يحتوي على `total`, `count`, `per_page`, `current_page`, `total_pages`, `links` (للصفحة السابقة والتالية).

#### 3.4 كلاس `ReportUserManager`
تم تطوير كلاس موحد لإدارة المستخدمين والصلاحيات، يدعم:

- الحصول على المستخدم الحالي من أنظمة متعددة (BackendAuth, Auth, auth helper).
- استخراج معرف المستخدم واسمه وبريده ورقم هاتفه.
- التحقق من الصلاحيات عبر دوال `hasAnyAccess` و `hasAccess` مع دعم الخصائص القابلة للاستدعاء (callable properties) والطرق المدمجة في كائنات المستخدم.
- الحصول على معرف الشركة (`getCompanyId`) المستخدم في نطاق الشركة.

#### 3.5 تحسينات في `ReportsManager`
- إضافة دالة `loadReports()` لتحميل التقارير المسجلة من الإضافات عبر دالة `registerReports` في `Plugin.php`.
- إعادة تعريف دوال `getStoredReport`, `getAvailableStoredReports`, `runStoredReport` لاستدعاء `loadReports()` أولاً.
- إضافة دالة `setExposeMetaDetails` للتحكم في عرض التفاصيل على مستوى المدير.
- تحسين `registerDefinition` لتطبيق التعريف فوراً إذا كان الـ registry مهيأً (دعم التسجيل الديناميكي بعد التهيئة).

#### 3.6 تحسينات في `HasStoredReports`
- إضافة دالة `applyPaginationToQueryConfig` لدمج خيارات التصفح (`page`, `per_page`, `limit`, `offset`) في `query_config`.
- تحديث دالة `runStoredReport` لاستخدام هذه الدالة مع الحفاظ على إعدادات التصفح الخاصة بكل تقرير (enabled, default_per_page, max_per_page).
- تعديل `sanitizeReportDefinitionForOutput` لإرجاع الأعمدة (`columns`) والأعمدة المحسوبة (`computed_columns`) إلى جانب الفلاتر، مما يتيح للواجهة الأمامية معرفة الحقول التي يعرضها كل تقرير.

#### 3.7 تحسينات معالجة الأخطاء
- في `ReportQueryErrorHandler`، تم تعديل دالة `handleError` لتعيد هيكل خطأ موحد مع إمكانية إضافة `debug` عند تفعيل `app.debug`.
- في المتحكم `DynamicReports`، تمت إضافة دالة `formatErrorResponseFromResult` لتنسيق الأخطاء القادمة من `runReport` بشكل صحيح، مع إخفاء التفاصيل الحساسة في الإنتاج.
- دالة `getSafeErrorMessage` تم توسيعها لتغطية أنواع مختلفة من الاستثناءات (QueryException, ModelNotFoundException, ValidationException, AuthorizationException, ApplicationException, SystemException, AuthException) مع رسائل مناسبة.

#### 3.8 دعم التصفح في التقارير المسجلة
- تمت إضافة مفتاح `pagination` في تعريف التقرير (يتم دمجه افتراضياً في `normalizeReportDefinition`).
- يمكن تحديد `enabled`, `default_per_page`, `max_per_page`, `allow_offset`.
- في `runStoredReport`، يتم دمج `page` و `per_page` من `options` وتحويلها إلى `limit` و `offset` مع احترام الحدود القصوى.

---

### 4. آلية العمل (التدفق الجديد)

1. **تسجيل التقارير**: تقوم الإضافات بتعريف تقاريرها عبر دالة `registerReports` في `Plugin.php`، بنفس آلية `registerReportTables`.
2. **تحميل التقارير**: عند تهيئة `ReportsManager` (أول استدعاء لـ `getRegistry`)، يتم استدعاء `loadReports()` التي تجمع التقارير من الإضافات وتسجلها في `storedReports`.
3. **استعلام قائمة التقارير**: تستدعي الواجهة `GET /dynamic-reports`، فيقوم المتحكم باستدعاء `getAvailableStoredReports()` الذي يراعي صلاحيات المستخدم ويعيد قائمة مبسطة.
4. **تشغيل تقرير مسجل**: يرسل المستخدم `POST /dynamic-reports/run/{reportId}` مع `filters` و `options`. يقوم `runStoredReport` بدمج الفلاتر وخيارات التصفح مع `query_config` الأصلي، ثم يستدعي `runReport` (الموجود في `ReportsManagerQueryConverter`).
5. **تنفيذ الاستعلام**: يستخدم `ReportQueryConverter` معلومات الجدول المسجل (من `ReportSchemaRegistry`) لبناء الاستعلام، مع تطبيق نطاق الشركة عبر `company_key` وإضافة auto-joins حسب الحاجة.
6. **حساب التصفح**: إذا كان هناك `limit`، يتم حساب العدد الإجمالي (`getTotalCount`) وإضافة معلومات pagination إلى `meta`.
7. **إرجاع النتيجة**: يتم إرجاع النتائج مع `columns` و `meta`، وفي حالة تفعيل `expose_meta_details` تظهر تفاصيل الاستعلام.

---

### 5. أبرز الإنجازات والميزات

- **نظام التقارير المسجلة (Templates)**: يمكن الآن حفظ تقارير ثابتة في الكود، مع تحديد الأعمدة والفلاتر والصلاحيات وخيارات التصدير، واستدعاؤها بمرونة مع إمكانية تمرير قيم الفلاتر.
- **واجهة API متكاملة**: 8 نقاط نهاية تغطي جميع احتياجات الواجهة الأمامية (قائمة، تفاصيل، تنفيذ، تصدير، مخطط، تحقق، تقدير).
- **دعم التصفح الكامل**: يمكن للمستخدمين التنقل بين صفحات النتائج مع معلومات دقيقة عن العدد الإجمالي والصفحات.
- **تحكم دقيق في صلاحيات الجدول والتقرير**: عبر `permissions` في تعريف الجدول والتقرير، وكلاس `ReportUserManager` للتحقق المرن.
- **إخفاء البيانات الحساسة في الإنتاج**: استعلامات SQL لا تظهر للعميل النهائي، ولكن تبقى متاحة للتطوير عبر `expose_meta_details` أو `app.debug`.
- **توثيق شامل**: ملفات توثيق تفصيلية بالعربية والإنجليزية للمتحكم `DynamicReports` مع أمثلة عملية لكل نقطة نهاية.
- **دعم حقول الشركة المختلفة**: عبر `company_key`، يمكن للنظام العمل مع أي اسم عمود (companys_id، company_id، company_uuid، إلخ).

---

### 6. الفوائد والقيمة المضافة

- **للمطورين**:
  - إمكانية تعريف تقارير معقدة مرة واحدة واستخدامها في عدة أماكن.
  - واجهة برمجية نظيفة وموثقة للتعامل مع التقارير.
  - توفير وقت التطوير من خلال إعادة استخدام التقارير الجاهزة.
  - سهولة التوسع بإضافة تقارير جديدة عبر دالة `registerReports` فقط.

- **للمستخدمين النهائيين**:
  - تجربة سلسة في إنشاء التقارير عبر واجهة موحدة.
  - إمكانية تصدير التقارير بصيغ متعددة.
  - تصفح النتائج عبر صفحات مع معلومات دقيقة.
  - رسائل خطأ واضحة تساعد في تصحيح الإدخالات.

- **للنظام ككل**:
  - أمان محسن عبر الفصل بين تعريف الجداول والتقارير وصلاحيات المستخدم.
  - أداء عالي بفضل التخزين المؤقت وحساب العدد الإجمالي بكفاءة.
  - مرونة في التوسع لدعم أعمال جديدة.

---

### 7. خطط التطوير المستقبلية

- إكمال دعم PDF باستخدام مكتبة متخصصة (مثل Dompdf أو wkhtmltopdf).
- تطوير واجهة مستخدم رسومية (GUI) لإدارة تعريفات الجداول والتقارير.
- دمج إمكانية جدولة التقارير وإرسالها بالبريد الإلكتروني.
- دعم مصادر بيانات متعددة (API، قواعد بيانات أخرى) عبر واجهات موحدة.
- تحسينات إضافية على أداء استعلامات التجميع للبيانات الضخمة.

---

### 8. الخاتمة

يمثل تحديث نظام التقارير الديناميكي `Nano2.QueryBuilder.Reporting` نقلة نوعية في قدرات المنصة على توفير حلول تحليل بيانات متقدمة. من خلال إضافة نظام التقارير المسجلة مسبقاً (Predefined Reports) وتوفير واجهة برمجية (API) متكاملة، تم تمكين المطورين من بناء تطبيقات تقارير معقدة بسرعة وأمان، مع منح المستخدمين النهائيين تجربة مرنة وسلسة في استعراض وتحليل البيانات.

ساهم هذا الإصدار في تعزيز أمان النظام عبر آليات صلاحيات متعددة المستويات، وتحسين الأداء من خلال دعم التخزين المؤقت وحساب التصفح بكفاءة، بالإضافة إلى معالجة الأخطاء بطريقة احترافية تحمي المعلومات الحساسة في بيئات الإنتاج. كما أن إضافة `company_key` الديناميكي عززت مرونة النظام في التكيف مع هياكل قواعد البيانات المختلفة.

مع الانتهاء من هذه المرحلة، يصبح النظام جاهزاً للتوسع المستقبلي ليشمل دعم تصدير PDF، وجدولة التقارير، والتكامل مع مصادر بيانات متعددة، مما يضمن استمراره كأداة أساسية في تطبيقات نانوسوفت. نشكر جميع المساهمين في هذا الإنجاز، ونتطلع إلى مواصلة التطوير بناءً على ملاحظات المستخدمين والمتطلبات المتطورة.

## 2026-04-20

**تحديثات إضافة `Nano3.Kyc` – الإصدارات 1.1.1 إلى 1.2.0**

### ملخص التحديثات

بعد إضافة دوال تقييم KYC ونقاط نهاية التقييم في الإصدارات السابقة، ركزت التحديثات الجديدة على **تمكين رفع الملفات وإدارتها** عبر التكامل الكامل مع حزمة `Nano.FileUpload`، بالإضافة إلى **تطوير نظام اختبار شامل** يضمن استقرار الإضافة ويسهل اكتشاف الأخطاء. تشمل التحديثات في الإصدارات من 1.1.1 إلى 1.2.0:

- **دمج رفع الملفات في واجهة API**: دعم رفع `document_front`, `document_back`, `signature_image`, `image`, `files` أثناء إنشاء وتحديث الوثائق.
- **حماية الملفات بعد الاعتماد**: منع تغيير أو حذف ملفات الوثيقة بعد اعتمادها (`is_verified = true`).
- **تسجيل موديل `Document` في `FileUploadRegistry`**: لتمكين صلاحيات الرفع المتقدمة وإدارة الملفات المؤقتة.
- **إضافة كلاس اختبار شامل `KycPlusTest`**: يغطي اختبارات `DocumentType`، `Manager`، رفع الملفات، تقييم KYC، وصلاحيات API.
- **نقطة نهاية `GET /tests`**: لتشغيل مجموعة الاختبارات في بيئة التطوير وعرض النتائج بشكل موحد.
- **تحسينات عامة على معالجة الأخطاء ورسائل الترجمة**.

---

### الإصدار 1.1.1 – تحسينات طفيفة على نقاط نهاية التقييم

#### أهداف الإصدار

- تثبيت نقاط النهاية `assessKycStatus` و `assessKycStatusByCategory` التي أضيفت في 1.1.0.
- تحسين معالجة خيارات `include_expired`, `include_pending`, `include_rejected` مع دعم القيم النصية (`"true"`/`"false"`).

#### الميزات الجديدة

- تحويل تلقائي لخيارات التضمين من نصوص إلى قيم منطقية، مما يسمح باستخدامها بسهولة في query strings.

---

### الإصدار 1.1.2 – دعم رفع الملفات في API مع حماية الوثائق المعتمدة

#### أهداف الإصدار

- تمكين المستخدمين من رفع ملفات الهوية (صور، PDF، توقيع) مباشرة عبر نقاط النهاية `create` و `update`.
- منع أي تعديل على ملفات الوثيقة بعد اعتمادها (`is_verified = true`)، حفاظاً على سلامة بيانات KYC.
- تسجيل نموذج `Document` في `FileUploadRegistry` بشكل آلي.

#### الميزات الجديدة

##### 1. سمة `FileUploadDocuments`

تم إنشاء سمة جديدة `Nano3\Kyc\APIControllers\Documents\FileUploadDocuments` تحتوي على الدوال:

| الدالة | الوصف |
| :--- | :--- |
| `extractUploadedFiles()` | استخراج الملفات من بيانات الطلب (سواء `UploadedFile` أو base64). |
| `attachFilesToDocument()` | رفع الملفات عبر `FileUploadService` وربطها بالوثيقة. |
| `documentHasFile()` | التحقق من وجود ملف مرتبط بحقل معين. |
| `registerDocumentModel()` | تسجيل موديل `Document` في `FileUploadRegistry`. |

##### 2. تحديث دالتي `create` و `update`

- **`create`**: بعد إنشاء الوثيقة، يتم رفع الملفات المرفقة وربطها. في حالة فشل رفع بعض الملفات، لا تفشل العملية كاملة، بل يتم إرجاع تحذير مع تفاصيل الأخطاء.
- **`update`**: قبل السماح برفع ملفات جديدة، يتم التحقق من حالة الوثيقة. إذا كانت `is_verified = true` ولديها ملف مرتبط مسبقاً، يتم رفض الطلب مع رسالة خطأ مناسبة.

##### 3. تسجيل `Document` في `FileUploadRegistry`

في `Plugin::boot()`، يتم استدعاء `registerDocumentInFileUpload()` التي تسجل الحقول التالية:
- `document_front`, `document_back`, `signature_image`, `image` (نوع `image`، حد أقصى 10MB للصور).
- `files` (نوع `multiple`، يسمح بعدة ملفات).

هذا يمكن `FileUploadService` من تطبيق صلاحيات الرفع والتحقق من القيود تلقائياً.

#### الفوائد

- تجربة مستخدم موحدة لرفع ملفات KYC دون الحاجة لاستدعاء منفصل لواجهة `FileUpload`.
- حماية قوية للوثائق المعتمدة من التلاعب.
- تكامل سلس مع نظام الصلاحيات والقيود الخاص بـ `Nano.FileUpload`.

---

### الإصدار 1.1.3 – تحسينات على رفع الملفات ورسائل الترجمة

#### أهداف الإصدار

- تحسين التعامل مع الملفات المرفوعة بصيغة base64.
- إضافة مفاتيح ترجمة لرسائل النجاح الجزئي وحماية الملفات المعتمدة.
- معالجة أفضل للأخطاء في رفع الملفات.

#### الميزات الجديدة

- دعم الحقول التي تنتهي بـ `_base64` لاستقبال الملفات المشفرة.
- مفاتيح ترجمة جديدة:
  - `public.helpers.create_document.msg_files_partial`
  - `public.helpers.update_document.msg_files_partial`
  - `public.helpers.update_document.msg_cannot_change_verified_files`
- إرجاع تفاصيل أخطاء الرفع في `process_data.file_errors`.

---

### الإصدار 1.2.0 – نظام اختبار متكامل (`KycPlusTest`)

#### أهداف الإصدار

- توفير آلية اختبار شاملة تغطي جميع مكونات الإضافة (DocumentType, Manager, FileUpload, API).
- تمكين المطورين من تشغيل الاختبارات بسهولة عبر واجهة API في بيئة التطوير.
- ضمان استقرار الإضافة عند إجراء تغييرات مستقبلية.

#### الميزات الجديدة

##### 1. كلاس `KycPlusTest`

كلاس اختبار يقع في `Nano3\Kyc\Tests\KycPlusTest` ويغطي:

- **DocumentType**: تسجيل الأنواع، جلب الحقول، قواعد التحقق.
- **Manager**: إنشاء، تحديث، اعتماد، رفض، حذف، استعادة، جلب السجلات، التحقق من التكرار.
- **رفع الملفات**: إنشاء وتحديث وثائق مع ملفات، منع تغيير ملفات الوثائق المعتمدة.
- **تقييم KYC**: `assessKycStatus` و `assessKycStatusByCategory`.
- **الصلاحيات والعزل**: التحقق من مصادقة API وعزل بيانات المالك.

**هيكل نتائج الاختبارات:**
كل اختبار يعيد مصفوفة تحتوي على:
```json
{
    "code": 200,
    "status": true,
    "test_code": "DOCTYPE_REG_001",
    "name": "تسجيل أنواع الوثائق",
    "description": "...",
    "message": "Test passed",
    "data": {...},
    "input_data": {...},
    "process_data": {...}
}
```

##### 2. نقطة نهاية `GET /api/v1/kyc/tests`

تتيح تشغيل مجموعة الاختبارات وعرض النتائج. **متاحة فقط** عندما يكون `app.debug = true` والمستخدم من نوع `backend`.

**معاملات الاستعلام:**
| المعامل | الوصف |
| :--- | :--- |
| `test_version` | `v1` أو `v2` (الافتراضي `v2`). |
| `filter` | `all`, `passed`, `failed` (الافتراضي `all`). |

**مثال للطلب:**
```
GET /api/v1/kyc/tests?test_version=v2&filter=passed
```

**الاستجابة:** تحتوي على `data` (نتائج الاختبارات المصفاة) و `meta.summary` بإحصائيات (عدد الناجح، الفاشل، نسبة النجاح).

##### 3. عزل بيئة الاختبار

تستخدم الاختبارات `Db::beginTransaction()` و `Db::rollBack()` لضمان عدم تأثيرها على قاعدة البيانات الحقيقية. كما يتم تنظيف الملفات المرفوعة مؤقتاً.

#### الفوائد

- اكتشاف الأخطاء مبكراً قبل النشر.
- توثيق حي لسلوك الإضافة من خلال الاختبارات.
- سهولة التحقق من صحة التثبيت في بيئات مختلفة.

---

### ملخص الإصدارات (1.1.1 – 1.2.0)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.1.1 | تحسين نقاط نهاية التقييم. |
| 1.1.2 | دعم رفع الملفات في API، منع تغيير ملفات الوثائق المعتمدة، تسجيل Document في FileUploadRegistry. |
| 1.1.3 | تحسين base64 ورسائل الترجمة. |
| 1.2.0 | إضافة نظام اختبار شامل (`KycPlusTest`) ونقطة نهاية `/tests`. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال الملفات التالية بالنسخ الجديدة:
     - `Plugin.php`
     - `apicontrollers/Documents.php`
     - `apicontrollers/documents/FileUploadDocuments.php`
     - `tests/KycPlusTest.php` (جديد)

2. **إضافة مسار الاختبارات** في `routes.php`:
   ```php
   Route::get('tests', 'Documents@tests');
   ```

3. **تحديث ملف الترجمة** `lang.php` بالمفاتيح الجديدة (انظر الإصدار 1.1.3).

4. **تشغيل الاختبارات** (في بيئة التطوير):
   ```
   GET /api/v1/kyc/tests
   ```
   للتأكد من أن الترقية تمت بنجاح.

5. **لا توجد هجرات جديدة** في هذه الإصدارات.

---

### الخاتمة

مع الإصدارات 1.1.1 إلى 1.2.0، أصبحت إضافة `Nano3.Kyc` متكاملة تماماً مع نظام رفع الملفات `Nano.FileUpload`، مما يوفر تجربة سلسة لإدارة وثائق KYC مع حماية متقدمة للبيانات المعتمدة. كما أن إضافة نظام اختبار شامل يعزز الثقة في استقرار الإضافة ويسهل صيانتها وتطويرها مستقبلاً. هذه التحسينات تجعل الإضافة جاهزة للاستخدام في المشاريع الإنتاجية مع ضمان أعلى معايير الجودة.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)
- [توثيق السلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)

## 2026-04-20 - 2026-04-21

**تحديث إضافة `Nano3.Kyc` – الإصدار 1.2.1**

### إعادة هيكلة مسارات API للتحقق والرفض والاستعادة إلى نمط أكثر توافقاً مع REST

---

### ملخص التحديثات

في الإصدار **1.2.1**، تم تحسين مسارات API الخاصة بالعمليات غير القياسية (التحقق `verify`، الرفض `reject`، والاستعادة `restore`) لتصبح أكثر وضوحاً وتنظيماً. بدلاً من النمط السابق `POST documents/{id}/verify`، تم اعتماد النمط الجديد `POST documents/verify/{id?}` مع جعل معرّف الوثيقة اختيارياً في المسار (يدعم أيضاً إرساله ضمن جسم الطلب). هذا التغيير يحقق عدة أهداف:

- **توافق أفضل مع مبادئ REST**: الإجراء المخصص (`verify`) يأتي تحت نطاق المورد (`documents`) قبل المعرّف، مما يوضح أن الإجراء ينتمي إلى مجموعة `documents`.
- **مرونة في تمرير المعرّف**: يمكن إرسال `id` كجزء من المسار أو كمعامل في جسم الطلب (`id` أو `document_id`)، مما يحافظ على التوافق العكسي مع أي عميل قديم يستخدم الطريقة السابقة.
- **سهولة التوسع**: يسهل إضافة إجراءات جديدة مثل `documents/bulk-verify` مستقبلاً ضمن نفس النطاق.

تم تحديث دوال المتحكم `verify` و `reject` و `restore` لدعم هذا النمط الجديد مع الاحتفاظ بكامل المنطق الداخلي دون تغيير. كما تم تحديث ملفات التوثيق لتعكس المسارات الجديدة.

---

### أهداف الإصدار

- **تحسين تنظيم المسارات** وجعلها أكثر وضوحاً للمطورين المستخدمين للـ API.
- **زيادة المرونة** في تمرير معرّف الوثيقة دون التقيد بموقعه في المسار.
- **الحفاظ على التوافق العكسي** بحيث لا تتعطل التطبيقات الحالية التي تستخدم المسارات القديمة (التي ما زالت مدعومة).
- **تسهيل الصيانة المستقبلية** للمتحكم والمسارات.

---

### الميزات الجديدة

#### 1. مسارات API الجديدة

تم استبدال المسارات القديمة:

| المسار القديم (ما زال مدعوماً) | المسار الجديد (الموصى به) |
| :--- | :--- |
| `POST documents/{id}/verify` | `POST documents/verify/{id?}` |
| `POST documents/{id}/reject` | `POST documents/reject/{id?}` |
| `POST documents/{id}/restore` | `POST documents/restore/{id?}` |

المسارات الجديدة تجعل الإجراء (`verify`, `reject`, `restore`) جزءاً من مسار المورد قبل المعرّف الاختياري.

#### 2. تحديث دوال المتحكم

تم تعديل تواقيع الدوال في `Documents` controller لتقبل `$id` كمعامل اختياري من المسار، مع الاحتفاظ بمنطق القراءة من `Input::get()` كبديل:

```php
public function verify($id = null)
{
    if (!$id) {
        $id = Input::get('id') ?? Input::get('document_id');
    }
    // ... باقي المنطق
}
```

هذا التغيير يسمح باستدعاء الدالة عبر المسار الجديد `POST documents/verify/123` أو عبر المسار القديم `POST documents/123/verify` أو حتى عبر إرسال `id` في جسم الطلب مع `POST documents/verify`.

#### 3. تحديث ملف `routes.php`

تم تعديل ملف التوجيه ليضم المسارات الجديدة مع الإبقاء على المسارات القديمة معلقة (للتوافق العكسي):

```php
// المسارات الجديدة (الموصى بها)
Route::post('documents/verify/{id?}', 'Documents@verify');
Route::post('documents/reject/{id?}', 'Documents@reject');
Route::post('documents/restore/{id?}', 'Documents@restore');

// المسارات القديمة (للتوافق العكسي - مهملة)
// Route::post('documents/{id}/verify', 'Documents@verify');
// Route::post('documents/{id}/reject', 'Documents@reject');
// Route::post('documents/{id}/restore', 'Documents@restore');
```

يمكن إزالة المسارات القديمة في إصدار مستقبلي بعد إشعار المستخدمين.

#### 4. تحديث ملف `version.yaml`

تم إضافة إصدار جديد `1.2.1` يوثق هذه التغييرات.

#### 5. تحديث التوثيق

تم تحديث توثيق API ليعكس المسارات الجديدة مع ذكر أن المسارات القديمة ما زالت تعمل لأغراض التوافق.

---

### مقارنة بين الأنماط

| المعيار | النمط القديم `documents/{id}/verify` | النمط الجديد `documents/verify/{id?}` |
| :--- | :--- | :--- |
| **الوضوح** | جيد (إجراء على مورد) | أفضل (الإجراء تحت نطاق المورد) |
| **التوافق مع REST** | مقبول (RPC-style) | أفضل تجميعاً |
| **قابلية التوسع** | صعوبة في إضافة إجراءات مجمعة | سهولة إضافة `documents/bulk-verify` |
| **التوافق العكسي** | مدعوم (بالإبقاء على المسار القديم) | مدعوم (معامل اختياري + قراءة من Input) |
| **تأثير على الكود** | لا تغيير | تغيير طفيف في توقيع الدوال |

---

### أمثلة على الاستخدام

#### 1. استخدام المسار الجديد مع `id` في المسار

```bash
curl -X POST "https://api.example.com/api/v1/kyc/documents/verify/123" \
  -H "Authorization: Bearer <token>"
```

#### 2. استخدام المسار الجديد مع `id` في الجسم (اختياري)

```bash
curl -X POST "https://api.example.com/api/v1/kyc/documents/verify" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"id": 123}'
```

#### 3. استخدام المسار القديم (ما زال يعمل)

```bash
curl -X POST "https://api.example.com/api/v1/kyc/documents/123/verify" \
  -H "Authorization: Bearer <token>"
```

---

### الفوائد والقيمة المضافة

- **تنظيم أفضل للمسارات**: يسهل فهم الـ API وتصفحه.
- **مرونة أكبر للمطورين**: يمكنهم اختيار الطريقة الأنسب لتمرير المعرّف.
- **جاهزية للتوسع**: تمهيد لإضافة عمليات جماعية مثل `bulk-verify`.
- **لا كسر للتوافق العكسي**: التطبيقات الحالية تستمر بالعمل دون تغيير.
- **توثيق أوضح**: المسارات الجديدة تعبر بشكل أدق عن العلاقة بين المورد والإجراء.

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال `routes.php` بالنسخة التي تحتوي على المسارات الجديدة.
   - استبدال دوال `verify`, `reject`, `restore` في `Documents.php` بالنسخ المحدثة (تدعم `$id = null`).

2. **لا توجد هجرات جديدة** – لا تغييرات في قاعدة البيانات.

3. **لا توجد تغييرات في الإعدادات** – يظل ملف `config.php` كما هو.

4. **تحديث وثائق API الداخلية** (إن وجدت) لتعكس المسارات الجديدة.

5. **اختبار التوافق**:
   - تأكد من أن المسارات القديمة ما زالت تعمل (إذا كنت قد أبقيتها).
   - اختبر المسارات الجديدة مع وبدون `id` في المسار.

---

### الخاتمة

يمثل الإصدار **1.2.1** تحسيناً هاماً في تنظيم واجهة API لإضافة `Nano3.Kyc`. من خلال إعادة هيكلة مسارات العمليات غير القياسية، أصبحت الـ API أكثر وضوحاً وقابلية للتوسع، مع الحفاظ الكامل على التوافق مع الإصدارات السابقة. هذه الخطوة تمهد الطريق لإضافة ميزات أكثر تقدماً في المستقبل مثل العمليات المجمعة (Bulk Operations).

---

### 📚 الوثائق المرجعية

- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

## 2026-04-21

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.2**

### تحسين جذري لمنطق تقييم KYC عبر نظام المجموعات البديلة وإعادة هيكلة الكود

---

### ملخص التحديثات

في الإصدار **1.2.2**، تم تنفيذ تحسين جوهري على آلية تقييم حالة KYC، وذلك بمعالجة قيود المنطق السابق الذي كان يتطلب وثائق محددة بعينها (مثل اشتراط `national_id` رغم وجود `passport`). التحديث يقدم **نظام المجموعات البديلة (Alternative Groups)** الذي يتيح اعتبار أي وثيقة من ضمن مجموعة معينة (مثل وثائق الهوية الأساسية) كافية لاستيفاء المتطلب الإلزامي. كما تم إعادة هيكلة كود التقييم بالكامل ونقله إلى سمة مستقلة (`HasAssessKycStatus`) لتعزيز قابلية الصيانة والتوسع.

تشمل التحسينات الرئيسية:

- **نظام المجموعات البديلة**: إمكانية تعريف مجموعات مثل `primary_identity` (جواز سفر، هوية وطنية، رخصة قيادة...إلخ) و`address_verification` (فاتورة مرافق، كشف حساب...إلخ)، حيث يكفي تقديم وثيقة واحدة من المجموعة.
- **منطق تقييم أكثر دقة**: تم تعديل دوال `assessKycStatus` و `assessKycStatusByCategory` للتعامل مع المجموعات البديلة، مما ينتج تقييمات أكثر واقعية ومطابقة للمعايير العملية لأنظمة KYC.
- **إعادة هيكلة الكود**: تم استخراج دوال التقييم ومساعداتها إلى سمة `HasAssessKycStatus`، مما يقلل من حجم كلاس `Manager` ويسهل اختبار وتعديل منطق KYC بشكل مستقل.
- **تحسين دالة `getMandatoryTypesForParty`**: أصبحت تعيد متطلبات على شكل نصوص خاصة مثل `group:primary_identity` بدلاً من أنواع محددة، مما يسمح بمرونة عالية في تعريف المتطلبات حسب مستوى المخاطرة ونوع الطرف.
- **إصلاح التقييم حسب الفئة (`assessKycStatusByCategory`)**: تم تعديل جلب الوثائق ليشمل جميع وثائق الفئة، مما يضمن ظهور الوثائق الموجودة في نتائج التقييم، بدلاً من اقتصارها على الأنواع المطلوبة فقط.

---

### أهداف الإصدار

- **جعل تقييم KYC أكثر مرونة وواقعية** من خلال السماح ببدائل للوثائق الإلزامية، وهو ما يتماشى مع أفضل الممارسات في أنظمة الامتثال.
- **تحسين دقة حسابات نسبة الاكتمال (`completion_percentage`) والدرجة الإجمالية (`overall_score`)** لتعكس بدقة حالة المستخدم بناءً على المجموعات المستوفاة.
- **تسهيل صيانة وتطوير منطق KYC** عبر فصله في سمة مستقلة (`HasAssessKycStatus`).
- **إصلاح مشكلة عدم ظهور الوثائق في تقييم الفئة** (مثال: جواز السفر لم يظهر سابقًا في `kyc_status_by_category` عند تقييم `primary_id`).
- **توفير توصيات أوضح** للمستخدمين حول المتطلبات المفقودة، مع الإشارة إلى أمثلة من الوثائق المطلوبة.

---

### الميزات الجديدة

#### 1. سمة `HasAssessKycStatus`

تم إنشاء سمة جديدة `Nano3\Kyc\Classes\Manager\HasAssessKycStatus` تحتوي على كامل منطق تقييم KYC:

| الدالة | الوصف |
| :--- | :--- |
| `assessKycStatus()` | تقييم حالة KYC العامة لمالك معين (فرد/شركة). |
| `assessKycStatusByCategory()` | تقييم حالة KYC حسب تصنيف وثائق محدد (`primary_id`, `address`, ...). |
| `getEmptyAssessmentData()` | هيكل بيانات فارغ للتقييم. |
| `processMandatoryRequirements()` | معالجة المتطلبات الإلزامية وفصل المجموعات عن الأنواع الفردية. |
| `analyzeDocuments()` | تحليل مجموعة الوثائق واستخراج الإحصائيات والبيانات. |
| `determineSatisfiedGroups()` | تحديد المجموعات البديلة التي تم استيفاؤها. |
| `determineMissingRequirements()` | تحديد المتطلبات المفقودة (سواء أنواع فردية أو مجموعات). |
| `buildRecommendations()` | بناء قائمة التوصيات بناءً على نتائج التحليل. |

تم تضمين السمة في كلاس `Manager` باستخدام `use \Nano3\Kyc\Classes\Manager\HasAssessKycStatus;`، مما يحافظ على توافق واجهة `Manager` العامة دون تغيير.

#### 2. نظام المجموعات البديلة في `DocumentType`

أضفنا دوال جديدة في `DocumentType` لدعم المجموعات البديلة:

| الدالة | الوصف |
| :--- | :--- |
| `getAlternativeGroups()` | تعيد مصفوفة المجموعات البديلة، مثل `primary_identity` و `address_verification`. |
| `isInAlternativeGroup($type, $groupKey)` | تتحقق مما إذا كانت وثيقة معينة تنتمي لمجموعة محددة. |
| `getAlternativeGroupKey($type)` | تعيد مفتاح المجموعة البديلة التي تنتمي إليها الوثيقة. |

**تعريف المجموعات الافتراضي:**

```php
'primary_identity' => [
    TYPE_PASSPORT, TYPE_NATIONAL_ID, TYPE_DRIVERS_LICENSE,
    TYPE_RESIDENCE_PERMIT, TYPE_REFUGEE_DOCUMENT
],
'address_verification' => [
    TYPE_UTILITY_BILL, TYPE_BANK_STATEMENT, TYPE_TAX_DOCUMENT,
    TYPE_RENTAL_AGREEMENT, TYPE_MORTGAGE_STATEMENT
]
```

يمكن توسيع هذه المجموعات أو تجاوزها عبر ملف الإعدادات `config/nano3/kyc/document_types.php` مستقبلاً.

#### 3. تحسين `DocumentType::getMandatoryTypesForParty`

تم تحديث الدالة لتعيد متطلبات تعتمد على المجموعات البديلة حسب مستوى المخاطرة:

```php
if ($partyType === 'individual') {
    $mandatory[] = 'group:primary_identity';
    if (in_array($riskLevel, ['medium', 'high', 'very_high'])) {
        $mandatory[] = 'group:address_verification';
    }
    if (in_array($riskLevel, ['high', 'very_high'])) {
        $mandatory[] = self::TYPE_PASSPORT;
    }
}
```

هذا يعني:
- المستوى `low`: مطلوب فقط وثيقة هوية أساسية واحدة.
- المستوى `medium`: مطلوب وثيقة هوية أساسية + وثيقة إثبات عنوان.
- المستوى `high` / `very_high`: مطلوب وثيقة هوية أساسية + جواز سفر تحديداً + وثيقة إثبات عنوان.

#### 4. تحسينات في دالتي التقييم

- **`assessKycStatus`**: أصبحت تحسب `completion_percentage` بناءً على عدد المتطلبات (مجموعات وأنواع) المستوفاة من إجمالي المتطلبات الإلزامية. وتحسب `overall_score` بناءً على أوزان الوثائق الفردية التي تشكل المتطلبات.
- **`assessKycStatusByCategory`**: تم إصلاح جلب الوثائق بحيث يشمل جميع وثائق الفئة المحددة (`whereDocumentCategory` فقط، بدون `whereDocumentType`)، مما يضمن ظهور جميع الوثائق ذات الصلة في النتائج. كما تم استخدام نفس منطق المجموعات البديلة بعد تصفيته حسب الفئة.

#### 5. توصيات محسّنة

عند وجود متطلب `group:address_verification` غير مستوفى، ستظهر التوصية بمثال عن وثيقة مقبولة (مثل `utility_bill`) بدلاً من عرض `group:address_verification` كنص خام، مما يجعل الرسالة أكثر وضوحاً للمستخدم.

---

### مقارنة النتائج قبل وبعد التحديث

لنفس المستخدم (يمتلك جواز سفر معتمد فقط):

| المؤشر | قبل الإصدار 1.2.2 | بعد الإصدار 1.2.2 |
| :--- | :--- | :--- |
| **`kyc_status.is_compliant`** | `false` (يفتقد `national_id`) | `false` (يفتقد إثبات عنوان، لكن الهوية الأساسية مكتملة) |
| **`kyc_status.completion_percentage`** | `0%` | `50%` |
| **`kyc_status.total_documents_required`** | `1` | `2` |
| **`kyc_status.missing_required_types`** | `["national_id"]` | `["utility_bill"]` |
| **`kyc_status_by_category.is_compliant`** (فئة `primary_id`) | `false` | `true` |
| **`kyc_status_by_category.completion_percentage`** | `20%` | `100%` |
| **`kyc_status_by_category.verified_documents`** | `[]` (فارغة) | تحتوي على جوازات السفر |

هذه النتائج تعكس واقعية أكبر: المستخدم قد استوفى متطلبات الهوية، ولكنه بحاجة إلى إثبات عنوان، والتقييم حسب الفئة يظهر اكتمال متطلبات الهوية الأساسية.

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال الملفات التالية بالنسخ الجديدة:
     - `classes/Manager.php` (لإضافة `use HasAssessKycStatus`)
     - `classes/Manager/HasAssessKycStatus.php` (جديد)
     - `classes/DocumentType.php` (لإضافة دوال المجموعات البديلة وتعديل `getMandatoryTypesForParty`)

2. **لا توجد هجرات جديدة** – لا تغييرات في قاعدة البيانات.

3. **لا توجد تغييرات في الإعدادات** – يظل ملف `config.php` كما هو (يمكن تخصيص المجموعات عبر الإعدادات مستقبلاً).

4. **تحديث ملف `version.yaml`**:
   أضفنا الإصدار `1.2.2` مع وصف التغييرات.

5. **اختبار التوافق**:
   - تأكد من أن نقاط النهاية `/assess` و `/assess/{category}` تعمل بشكل صحيح مع النتائج الجديدة.
   - تحقق من سلوك التضمين `kyc_status` و `kyc_status_by_category` في `UserTransformer`.

6. **تشغيل الاختبارات** (في بيئة التطوير):
   ```
   GET /api/v1/kyc/tests
   ```
   للتأكد من أن مجموعة اختبارات `KycPlusTest` ما زالت تمر بنجاح.

---

### الفوائد والقيمة المضافة

- **مرونة عالية**: يمكن تخصيص متطلبات KYC بسهولة عبر تعديل المجموعات البديلة دون تغيير المنطق الأساسي.
- **دقة أفضل**: التقييم يعكس الوضع الفعلي للمستخدم، مما يقلل من الرفض الخاطئ أو طلب وثائق غير ضرورية.
- **كود أنظف وأكثر قابلية للصيانة**: بفضل نقل منطق KYC إلى سمة مستقلة، أصبح كلاس `Manager` أكثر تنظيماً وتركيزاً على مسؤولياته الأساسية.
- **تجربة مستخدم محسّنة**: التوصيات والمتطلبات المفقودة أصبحت أكثر وضوحاً.
- **جاهزية للتوسع**: يمكن إضافة مجموعات بديلة جديدة (مثل `secondary_identity`) بسهولة.

---

### الخاتمة

يمثل الإصدار **1.2.2** قفزة نوعية في دقة ومرونة نظام تقييم KYC داخل إضافة `Nano3.Kyc`. من خلال إدخال مفهوم المجموعات البديلة وإعادة هيكلة الكود، أصبحت الإضافة قادرة على محاكاة متطلبات الامتثال الواقعية بشكل أفضل، مع الحفاظ على أداء عالٍ وسهولة في التخصيص. هذه التحسينات تجعل `Nano3.Kyc` خياراً أكثر قوة وجاهزية للمشاريع التي تتطلب إدارة متقدمة للتحقق من الهوية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

## 2026-04-21 - 2026-04-22

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.3**

### ملخص التحديثات

بعد إعادة هيكلة منطق تقييم KYC في الإصدار 1.2.2 وإدخال مفهوم "المجموعات البديلة" للوثائق، نقدم في الإصدار 1.2.3 تحسينات جذرية على أداء وسرعة التحقق من حالة KYC. تم نقل دوال التحقق السريع إلى سمة مستقلة `HasValidKycDocuments` مع دعم متقدم للتخزين المؤقت والأقفال الذرية، مما يجعل عمليات الفحص المتكررة (مثل التحقق من وجود هوية أو إثبات عنوان) خفيفة للغاية على قاعدة البيانات ومناسبة للاستخدام في Middleware وPolicies وكل طلب API.

---

### الإصدار 1.2.3 – تحسينات الأداء والتخزين المؤقت لدوال التحقق السريع

#### أهداف الإصدار

- نقل منطق `hasValidKycDocumentByCategory` إلى سمة مستقلة `HasValidKycDocuments` لتحسين قابلية الصيانة.
- دعم **المجموعات البديلة** (`group:primary_identity`, `group:address_verification`) داخل دوال التحقق السريع.
- تنفيذ **تخزين مؤقت متعدد المستويات** مع دعم Cache Tags (لـ Redis/Memcached) ومجموعات مفاتيح احتياطية.
- إضافة **أقفال ذرية (Atomic Locks)** لمنع "تسابق الطلبات" (Cache Stampede) في أوقات الذروة.
- مسح الكاش تلقائياً عند أي تغيير في وثائق المستخدم (إنشاء، تحديث، حذف، اعتماد، رفض).
- توفير دالة مختصرة `hasValidKycDocument()` للتحقق من وجود أي وثيقة صالحة.

#### الميزات الجديدة

##### 1. سمة `HasValidKycDocuments`

تم نقل جميع دوال التحقق السريع إلى سمة جديدة `Nano3\Kyc\Classes\Manager\HasValidKycDocuments` والتي توفر:

| الدالة | الوصف |
| :--- | :--- |
| `hasValidKycDocumentByCategory($owner, $ownerType, $ownerId, $category, $documentType, $options)` | التحقق من وجود وثيقة صالحة ضمن تصنيف وأنواع محددة مع خيارات متقدمة. |
| `hasValidKycDocument($owner, $ownerType, $ownerId)` | دالة مختصرة للتحقق من وجود **أي** وثيقة KYC صالحة للمالك. |
| `clearKycCheckCache($ownerType, $ownerId)` | مسح جميع نتائج الكاش المرتبطة بمالك معين (تُستدعى تلقائياً عند تغيير الوثائق). |

##### 2. دعم المجموعات البديلة في التحقق السريع

يمكن الآن تمرير `group:primary_identity` أو `group:address_verification` ضمن `$documentType`، وسيتم توسيعها تلقائياً إلى قائمة الأنواع الفردية المناسبة. هذا يضمن الاتساق مع منطق `assessKycStatus` الجديد.

**مثال:**
```php
// التحقق من وجود أي هوية أساسية (جواز سفر، هوية وطنية، رخصة قيادة)
$hasId = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    'group:primary_identity'
);
```

##### 3. تخزين مؤقت متعدد المستويات

لتقليل الحمل على قاعدة البيانات في الاستدعاءات المتكررة (مثل `include=kyc_status` لكل مستخدم)، تم تطبيق آلية تخزين مؤقت ذكية:

- **Cache Tags (للأنظمة الداعمة)**: يتم تجميع مفاتيح الكاش تحت وسم `owner_{hash}`، مما يسمح بمسحها جميعاً دفعة واحدة عند الحاجة.
- **مجموعة مفاتيح احتياطية (Fallback)**: إذا كان مزود الكاش لا يدعم Tags (مثل `file`)، يتم تخزين المفاتيح في مصفوفة منفصلة (`owner_keys`) تُستخدم للمسح اليدوي.
- **مدة صلاحية افتراضية 5 دقائق** (قابلة للتعديل عبر `cache_ttl`).

##### 4. أقفال ذرية (Atomic Locks)

في حالات التزامن العالي (عدة طلبات لنفس المستخدم في نفس اللحظة)، قد تحاول عدة عمليات إعادة بناء نفس نتيجة الكاش مما يهدر الموارد. تمت إضافة قفل ذري (`Cache::lock`) يضمن أن عملية واحدة فقط هي التي تنفذ الاستعلام الفعلي، بينما تنتظر العمليات الأخرى وتستخدم النتيجة المخزنة.

**الخيارات الجديدة في `$options`:**
| الخيار | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `use_cache` | `bool` | `true` | تفعيل/تعطيل التخزين المؤقت. |
| `cache_ttl` | `int` | `300` | مدة صلاحية الكاش بالثواني. |
| `use_atomic_lock` | `bool` | `true` | تفعيل/تعطيل القفل الذري. |

##### 5. مسح الكاش التلقائي عند تغيير الوثائق

تم تعديل موديل `Document` (`afterSave` و `afterDelete`) لاستدعاء `Manager::clearKycCheckCache()` تلقائياً عند أي تغيير يطرأ على وثيقة (إنشاء، تحديث، حذف، اعتماد، رفض). هذا يضمن أن نتائج التحقق السريع تعكس دائماً أحدث حالة للمستخدم دون الحاجة إلى تدخل يدوي.

##### 6. تحسينات أداء الاستعلامات

- استخدام `select()` لتحديد الأعمدة الضرورية فقط (`id`, `document_type`, `status`, `issue_date`, `expiry_date`, `verification_score`).
- استخدام `exists()` بدلاً من `get()` عندما لا تكون هناك حاجة للتحقق من الصلاحية (`check_validity = false` و `require_all_types = false`).
- ترتيب `$requiredTypes` لضمان اتساق مفاتيح الكاش.

---

### أمثلة عملية

#### 1. التحقق من وجود هوية أساسية صالحة (مع استخدام الكاش)

```php
$hasId = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    'group:primary_identity',
    ['use_cache' => true, 'cache_ttl' => 600] // 10 دقائق
);

if ($hasId) {
    // السماح بالوصول
}
```

#### 2. التحقق من وجود أي وثيقة عنوان (بما فيها المعلقة)

```php
$hasAddress = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_ADDRESS,
    null,
    ['include_pending' => true]
);
```

#### 3. استخدام الدالة المختصرة للتحقق من أي وثيقة

```php
if (Manager::hasValidKycDocument($user)) {
    echo "المستخدم لديه وثيقة KYC واحدة صالحة على الأقل.";
}
```

#### 4. استخدام في Middleware لحماية المسارات

```php
class RequireKycMiddleware
{
    public function handle($request, Closure $next)
    {
        $user = Auth::getUser();
        if (!Manager::hasValidKycDocumentByCategory($user, null, null, DocumentType::CATEGORY_PRIMARY_ID, 'group:primary_identity')) {
            return response()->json(['error' => 'KYC verification required'], 403);
        }
        return $next($request);
    }
}
```

---

### ملخص الإصدارات (1.2.2 – 1.2.3)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.2.2 | إعادة هيكلة منطق التقييم إلى سمة `HasAssessKycStatus`، إضافة المجموعات البديلة للوثائق، تحسين دقة حسابات التقييم. |
| 1.2.3 | نقل دوال التحقق السريع إلى سمة `HasValidKycDocuments`، تخزين مؤقت متعدد المستويات، أقفال ذرية، دعم المجموعات البديلة في التحقق السريع، مسح تلقائي للكاش، تحسينات أداء شاملة. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملفات `Manager.php`، `Document.php`، وإضافة الملفات الجديدة:
     - `classes/Manager/HasValidKycDocuments.php`
     - `classes/Manager/HasAssessKycStatus.php` (إن لم يكن موجوداً)

2. **تشغيل هجرات قاعدة البيانات** (لا توجد تغييرات في هذا الإصدار).

3. **تحديث تعريفات API** (لا توجد تغييرات في نقاط النهاية).

4. **اختبار الأداء**:
   - راقب أداء نقاط النهاية التي تستخدم `include=kyc_status` بعد التفعيل؛ يجب أن تشهد تحسناً ملحوظاً بسبب الكاش.

5. **مسح الكاش يدوياً (اختياري)**:
   - إذا كنت ترغب في تطبيق الكاش فوراً، يمكنك مسح الكاش العام للتطبيق: `php artisan cache:clear`.

---

### الخاتمة

يمثل الإصدار 1.2.3 نقلة نوعية في أداء عمليات التحقق من KYC. بفضل التخزين المؤقت الذكي والأقفال الذرية، أصبحت الدوال التي تُستدعى آلاف المرات يومياً (مثل فحص حالة الهوية) سريعة جداً ولا تثقل كاهل قاعدة البيانات. كما أن مسح الكاش التلقائي يضمن دقة النتائج في جميع الأوقات. هذه التحسينات تجعل إضافة `Nano3.Kyc` جاهزة للاستخدام في المشاريع الضخمة ذات الحركة المرورية العالية دون أي قلق بشأن الأداء.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

## 2026-04-22 - 2026-04-23

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.4**

### ملخص التحديثات

بعد تحسين أداء دوال التحقق السريع في الإصدار 1.2.3 وإضافة التخزين المؤقت والأقفال الذرية، نقدم في الإصدار 1.2.4 تحسينات شاملة على سلوك `DynamicAddIncludeKyc` – المسؤول عن حقن علاقات KYC داخل `Transformers` الخاصة بـ `Nano.API`. يركز هذا الإصدار على:

- **تحسين جذري للأداء** عند طلب عدة علاقات KYC معاً من خلال تخزين مؤقت على مستوى الطلب.
- **إضافة 7 علاقات جديدة** تشمل ملخص حالة التحقق (`kyc_verification_summary`) وفحص سريع لكل فئة من فئات الوثائق الست (`is_verifier_primary_id`، `is_verifier_address`، ...).
- **دعم خيارات متقدمة** مثل `expiring_within_days` للكشف عن الوثائق التي على وشك الانتهاء.
- **توثيق محدث وشامل** يغطي جميع العلاقات الجديدة مع أمثلة عملية متكاملة.

هذا الإصدار يجعل دمج بيانات KYC في استجابات API أكثر قوة ومرونة، ويوفر للمطورين أدوات دقيقة للتحكم في سلوك الفحص حسب احتياجات التطبيق.

---

### الإصدار 1.2.4 – تحسينات شاملة لسلوك `DynamicAddIncludeKyc` وعلاقات KYC جديدة

#### أهداف الإصدار

- تحسين أداء سلوك `DynamicAddIncludeKyc` من خلال مشاركة استعلامات قاعدة البيانات عبر تخزين مؤقت على مستوى الطلب (`$documentsCache` و `$categoryCheckCache`).
- إضافة علاقة `kyc_verification_summary` التي تقدم ملخصاً سريعاً لحالة جميع فئات الوثائق الست مع إحصائيات (عدد المنتهية، المعلقة، المرفوضة).
- إضافة فاحصات سريعة (Boolean Includes) لكل فئة وثائق: `is_verifier_primary_id`، `is_verifier_secondary_id`، `is_verifier_address`، `is_verifier_corporate`، `is_verifier_ubo`، `is_verifier_additional`.
- دعم خيارات متقدمة لجميع الفاحصات السريعة وملخص التحقق، تشمل `include_pending`، `include_expired`، `check_validity`، و `expiring_within_days` (للكشف عن الوثائق التي ستنتهي خلال عدد أيام محدد).
- تحسين آلية تسجيل العلاقات (`includes`) داخل `__construct` بحيث يتم إضافتها إلى `secretIncludes` فقط إذا كان `Transformer` الأصلي يدعمها، وإلا تُضاف إلى `availableIncludes` العامة.
- تحديث شامل لملفي التوثيق `Docs-DynamicAddIncludeKyc-Behaviors-ar.md` و `Docs-API-Documentation-ar.md` لتغطية كافة الميزات الجديدة مع أمثلة عملية.

#### الميزات الجديدة

##### 1. تخزين مؤقت على مستوى الطلب (Request-Level Caching)

في السابق، كان طلب عدة علاقات KYC في استدعاء API واحد (مثل `?include=is_verifier_primary_id,is_verifier_address,kyc_verification_summary`) يؤدي إلى تنفيذ استعلامات قاعدة بيانات منفصلة ومتكررة لنفس المالك. في هذا الإصدار، تمت إضافة آليتين للتخزين المؤقت على مستوى الطلب:

| الخاصية | الوصف |
| :--- | :--- |
| `$documentsCache` | تخزن جميع وثائق المالك المسترجعة من قاعدة البيانات. يتم جلبها مرة واحدة فقط لكل طلب، بغض النظر عن عدد العلاقات المطلوبة. |
| `$categoryCheckCache` | تخزن نتائج فحص الفئات (`true`/`false`) مع مراعاة الخيارات الممررة. يمنع إعادة حساب نفس الفئة بنفس الخيارات خلال الطلب الواحد. |

**النتيجة:** تحسن كبير في زمن الاستجابة وتقليل الحمل على قاعدة البيانات عند استخدام عدة علاقات KYC معاً.

##### 2. علاقة `kyc_verification_summary` – ملخص حالة التحقق

توفر هذه العلاقة كائناً يحتوي على حالة جميع فئات الوثائق الست (`primary_id`، `secondary_id`، `address`، `corporate`، `ubo`، `additional`) بالإضافة إلى إحصائيات سريعة:

| الحقل | الوصف |
| :--- | :--- |
| `primary_id` | `true` إذا كان لدى المالك وثيقة هوية أساسية صالحة. |
| `secondary_id` | `true` إذا كان لدى المالك وثيقة هوية ثانوية صالحة. |
| `address` | `true` إذا كان لدى المالك وثيقة إثبات عنوان صالحة. |
| `corporate` | `true` إذا كان لدى المالك وثيقة شركات صالحة. |
| `ubo` | `true` إذا كان لدى المالك وثيقة مالك مستفيد صالحة. |
| `additional` | `true` إذا كان لدى المالك وثيقة إضافية صالحة. |
| `total_verified` | عدد الوثائق المعتمدة. |
| `total_pending` | عدد الوثائق قيد الانتظار. |
| `total_expired` | عدد الوثائق المنتهية. |
| `total_rejected` | عدد الوثائق المرفوضة. |

**خيارات مدعومة:** `include_pending`، `include_expired`، `check_validity`، `expiring_within_days`.

##### 3. فاحصات سريعة لكل فئة (Boolean Includes)

تمت إضافة 6 علاقات جديدة من النوع المنطقي (`true`/`false`) تتيح التحقق السريع من وجود وثيقة صالحة ضمن فئة محددة:

| العلاقة | الوصف |
| :--- | :--- |
| `is_verifier_primary_id` | هل يوجد وثيقة هوية أساسية صالحة؟ |
| `is_verifier_secondary_id` | هل يوجد وثيقة هوية ثانوية صالحة؟ |
| `is_verifier_address` | هل يوجد وثيقة إثبات عنوان صالحة؟ |
| `is_verifier_corporate` | هل يوجد وثيقة شركات صالحة؟ |
| `is_verifier_ubo` | هل يوجد وثيقة مالك مستفيد صالحة؟ |
| `is_verifier_additional` | هل يوجد وثيقة إضافية صالحة؟ |

جميع هذه العلاقات تدعم نفس الخيارات المتقدمة (`include_pending`، `include_expired`، `check_validity`، `expiring_within_days`).

> **ملاحظة:** العلاقات `is_verifier_secondary_id` حتى `is_verifier_additional` تُضاف إلى `secretIncludes` (لا تظهر في التوثيق التلقائي للـ API ولكن يمكن استخدامها)، بينما `is_verifier_primary_id` و `kyc_verification_summary` علاقات عامة تُضاف إلى `availableIncludes`.

##### 4. دعم خيار `expiring_within_days`

يمكن للمطورين الآن تمرير خيار `expiring_within_days` (عدد صحيح موجب) إلى أي من الفاحصات السريعة أو ملخص التحقق. عند تحديده، سيعيد الفحص `true` إذا كانت هناك وثيقة **صالحة حالياً** ولكنها ستنتهي صلاحيتها خلال العدد المحدد من الأيام. هذا مفيد لتنبيه المستخدمين لتجديد وثائقهم قبل انتهائها.

**مثال:** التحقق مما إذا كان لدى المستخدم هوية ستنتهي خلال 30 يوماً:
```
?include=is_verifier_primary_id&is_verifier_primary_id[expiring_within_days]=30
```

##### 5. تحسين آلية تسجيل العلاقات (`__construct`)

تم تحسين دالة البناء (`__construct`) لسلوك `DynamicAddIncludeKyc` لتكون أكثر ذكاءً وتوافقاً مع جميع أنواع `Transformers`:

- يتم التحقق من وجود الخاصية `secretIncludes` في الكلاس الأصلي (`property_exists`).
- إذا كانت الخاصية موجودة، يتم إضافة العلاقات الجديدة إلى `secretIncludes` (باستثناء العلاقات العامة التي تضاف أيضاً إلى `availableIncludes`).
- إذا لم تكن الخاصية موجودة (أي أن الـ `Transformer` لا يدعم العلاقات السرية)، يتم إضافة جميع العلاقات إلى `availableIncludes` مباشرة.

هذا يضمن سلوكاً متوقعاً في جميع البيئات دون أخطاء.

##### 6. تحديث التوثيق

تم تحديث ملفي التوثيق الرئيسيين ليعكسا كافة الميزات الجديدة:

| الملف | التحديثات |
| :--- | :--- |
| `Docs-DynamicAddIncludeKyc-Behaviors-ar.md` | إعادة كتابة كاملة لتشمل شرحاً مفصلاً لجميع العلاقات الجديدة، آلية التخزين المؤقت على مستوى الطلب، الخيارات المتقدمة، وأمثلة عملية متكاملة. |
| `Docs-API-Documentation-ar.md` | إضافة قسم مخصص بعنوان "تضمين بيانات KYC ضمن استجابات API" يشرح العلاقات المتاحة، كيفية استخدامها، الخيارات المدعومة، مع أمثلة عملية. |

---

### أمثلة عملية

#### 1. الحصول على ملخص سريع لحالة KYC مع فحص الوثائق التي ستنتهي خلال 30 يوماً

**الطلب:**
```http
GET /api/v1/me?include=kyc_verification_summary&kyc_verification_summary[include_pending]=true&kyc_verification_summary[expiring_within_days]=30 HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مقتطف):**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "kyc_verification_summary": {
        "primary_id": true,
        "secondary_id": false,
        "address": true,
        "corporate": false,
        "ubo": false,
        "additional": false,
        "total_verified": 2,
        "total_pending": 1,
        "total_expired": 0,
        "total_rejected": 0
    }
}
```

#### 2. استخدام عدة فاحصات سريعة معاً في طلب واحد

**الطلب:**
```http
GET /api/v1/me?include=is_verifier_primary_id,is_verifier_address,is_verifier_corporate HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مقتطف):**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "is_verifier_primary_id": true,
    "is_verifier_address": false,
    "is_verifier_corporate": false
}
```

> **ملاحظة الأداء:** في هذا المثال، يتم جلب وثائق المستخدم من قاعدة البيانات **مرة واحدة فقط** ويتم تخزينها في `$documentsCache`، ثم يتم فحص كل فئة من نفس مجموعة البيانات المخزنة مؤقتاً.

#### 3. فحص وثيقة هوية أساسية مع تضمين الوثائق قيد الانتظار

**الطلب:**
```http
GET /api/v1/me?include=is_verifier_primary_id&is_verifier_primary_id[include_pending]=true HTTP/1.1
Authorization: Bearer ...
```

في هذه الحالة، سيعيد `is_verifier_primary_id` القيمة `true` حتى لو كانت الوثيقة لا تزال في حالة `pending` (قيد المراجعة).

---

### ملخص الإصدارات (1.2.3 – 1.2.4)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.2.3 | نقل دوال التحقق السريع إلى سمة `HasValidKycDocuments`، تخزين مؤقت متعدد المستويات، أقفال ذرية، مسح تلقائي للكاش. |
| 1.2.4 | تحسين أداء سلوك `DynamicAddIncludeKyc` عبر تخزين مؤقت على مستوى الطلب، إضافة 7 علاقات جديدة (`kyc_verification_summary` و 6 فاحصات فئات)، دعم `expiring_within_days`، تحسين آلية تسجيل العلاقات، تحديث شامل للتوثيق. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `behaviors/DynamicAddIncludeKyc.php` بالنسخة الجديدة.
   - تأكد من وجود السمة `HasValidKycDocuments.php` (تمت إضافتها في الإصدار 1.2.3).

2. **تشغيل هجرات قاعدة البيانات** (لا توجد تغييرات في هذا الإصدار).

3. **تحديث تعريفات API** (لا توجد تغييرات في نقاط النهاية).

4. **اختبار العلاقات الجديدة**:
   - جرب طلب `?include=kyc_verification_summary` على `/api/v1/me` للتأكد من عمل الملخص بشكل صحيح.
   - جرب طلب عدة فاحصات سريعة معاً ولاحظ تحسن زمن الاستجابة.

5. **مراجعة التوثيق الجديد**:
   - اطلع على قسم "تضمين بيانات KYC ضمن استجابات API" في `Docs-API-Documentation-ar.md` لفهم كافة الخيارات المتاحة.

---

### الخاتمة

مع الإصدار 1.2.4، نكون قد أكملنا دورة تحسين أداء وكفاءة نظام KYC من جميع الزوايا: من دوال التحقق الخلفية (الإصدار 1.2.3) إلى واجهة API وتضمين البيانات (الإصدار 1.2.4). أصبح بإمكان المطورين الآن:

- استخدام علاقات KYC بكثافة داخل استجابات API دون القلق بشأن أداء قاعدة البيانات.
- الحصول على ملخصات سريعة أو فحوصات دقيقة حسب الحاجة.
- تخصيص سلوك الفحص بدقة عبر خيارات متقدمة مثل `expiring_within_days`.

نتطلع إلى ملاحظاتكم ومقترحاتكم لمواصلة تطوير هذه الإضافة الحيوية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)


## 2026-03-27 – 2026-04-23

**إطلاق إضافة `Nano3.TelecomRecharge` – منصة موحدة للتعامل مع مزودي خدمات الشحن الرقمي في اليمن **

### إطلاق إضافة `Nano3.TelecomRecharge` – منصة موحدة للتعامل مع مزودي خدمات الشحن المحليين

تم إطلاق إضافة **`Nano3.TelecomRecharge`** كإضافة برمجية مستقلة ضمن نظام نانوسوفت، تهدف إلى توفير حل موحد وقابل للتوسع للتعامل مع مزودي خدمات الشحن الرقمي في اليمن. الإصدار الأول من الإضافة يقدم تكاملاً كاملاً مع منصة **nflow.tech** (ماستر هوست)، ويوفر واجهة برمجية متكاملة لإدارة عمليات شحن رصيد شركات الاتصالات اليمنية، وتفعيل الباقات، وشحن الألعاب وبطاقات الهدايا.

تم تصميم الإضافة لتدعم إضافة مزودين جدد (مثل Sadad، YouGotaGift، وغيرهم) مستقبلاً دون الحاجة إلى تعديل الكود الأساسي، مما يجعلها منصة مرنة لخدمات الدفع الإلكتروني في اليمن.

---

### 1. مقدمة

استجابةً للحاجة المتزايدة لتكامل تطبيقات نانوسوفت مع بوابات الدفع الإلكتروني المحلية، تم تطوير إضافة `Nano3.TelecomRecharge` كأداة شاملة للتعامل مع واجهة برمجة التطبيقات (API) الخاصة بمنصة **nflow.tech** (ماستر هوست). يوفر الإصدار الأول واجهة موحدة لتنفيذ جميع العمليات الموثقة، بدءًا من استعلام رصيد الوكيل وصولاً إلى شحن الألعاب وتفعيل الباقات، مع دعم كامل لآلية المصادقة Webhook ومعالجة الأخطاء.

تم تصميم الإضافة لتعتمد على بنية قابلة للتوسع، مع فصل المسؤوليات عبر سمات (Traits) متخصصة، ودعم كامل لبيئة Laravel عبر متغيرات البيئة، وتوفير متحكم API متكامل مع توثيق شامل.

---

### 2. أهداف التحديثات

- **إنشاء إضافة برمجية مستقلة**: توفير إضافة `Nano3.TelecomRecharge` كحل موحد للتعامل مع مزودي الشحن الرقمي المحليين.
- **تكامل كامل مع nflow.tech**: تغطية جميع نقاط النهاية الموثقة في وثائق API الرسمية.
- **فصل المسؤوليات عبر السمات**: تقسيم الكود إلى سمات متخصصة (`NflowEndpoints`، `NflowExtendedEndpoints`، `NflowDataEndpoints`) لتسهيل الصيانة والتوسع.
- **دعم الدوال العامة المرنة**: إضافة دوال عامة (`executeService`، `queryBalance`، `chargeBalanceGeneric`، `activateOfferGeneric`) تسمح بتنفيذ أي خدمة بأرقام الشبكة والخدمة.
- **توفير متحكم API احترافي**: إنشاء `NflowController` مع نقاط نهاية RESTful محمية بـ OAuth.
- **توثيق شامل**: إعداد ملفي توثيق شاملين باللغة العربية (وثيقة الكلاس، أمثلة متقدمة، توثيق API).
- **دعم Webhook**: تمكين استقبال تحديثات الحالة عبر GET إلى رابط محدد.
- **معالجة أخطاء احترافية**: عرض رسائل آمنة للمستخدم في الإنتاج مع إمكانية تسجيل التفاصيل الكاملة في التطوير.

---

### 3. المكونات المطورة

#### 3.1 إضافة `Nano3.TelecomRecharge`

- **الاسم:** `Nano3.TelecomRecharge`
- **النطاق:** `Nano3\TelecomRecharge`
- **الهدف:** توفير واجهة موحدة للتعامل مع خدمات الشحن الرقمي المحلية (اليمن) عبر مزودين متعددين، مع إصدار أول مخصص لمنصة nflow.tech.
- **الهيكل الأساسي:**
  - `classes/services/NflowService.php` – الكلاس الرئيسي
  - `classes/services/NflowEndpoints.php` – العمليات العامة
  - `classes/services/NflowExtendedEndpoints.php` – الدوال العامة المرنة والاختصارات
  - `classes/services/NflowDataEndpoints.php` – بيانات الشبكات والخدمات والمساعدة
  - `controllers/NflowController.php` – متحكم API
  - `routes.php` – نقاط النهاية
  - `config.php` – الإعدادات ومتغيرات البيئة
  - `lang.php` – ملف اللغة
  - `docs/TelecomRecharge/` – التوثيق بالعربية

#### 3.2 كلاس `NflowService` (النواة)

يقع الكلاس في المسار `Nano3\TelecomRecharge\Classes\Services\NflowService`، ويستخدم ثلاث سمات:

- `NflowEndpoints`: العمليات العامة (رصيد الوكيل، حالة المعاملة، إيداعات الوكيل، فئات الألعاب).
- `NflowExtendedEndpoints`: الدوال العامة المرنة ودوال الاختصار للشبكات.
- `NflowDataEndpoints`: بيانات الشبكات والخدمات، ودوال التحقق المساعدة.

**الوظائف الأساسية:**

| الدالة | الوصف |
|--------|-------|
| `__construct()` | تهيئة الكلاس مع بيانات الاعتماد (من المعاملات أو من ملف البيئة). |
| `generateToken()` | توليد التوكن المؤقت باستخدام صيغة MD5 الموثقة. |
| `login()` | تسجيل الدخول والحصول على `accessToken`، مع إمكانية تحديد نقطة النهاية. |
| `post()` | تنفيذ طلب POST إلى `/rest-api/` مع إضافة `api-token` في الهيدر. |
| `parseResponse()` | تحليل الاستجابة JSON وإرجاع مصفوفة موحدة تحتوي على `success` و `data` و `status_code`. |
| `handleException()` | معالجة استثناءات Guzzle وإرجاع رسائل خطأ آمنة حسب بيئة التشغيل. |
| `testConnection()` | اختبار الاتصال بالخادم والتحقق من صحة بيانات الاعتماد. |

#### 3.3 سمة `NflowEndpoints` (العمليات العامة)

| الدالة | الوصف |
|--------|-------|
| `getAccountBalance()` | استعلام رصيد الوكيل (`NetworkNumber=0`, `ServiceNumber=1`) |
| `getOperationStatus($transactionId)` | استعلام حالة معاملة سابقة (`ServiceNumber=2`) |
| `getFeedClientsBalance()` | عرض إيداعات الوكيل (`ServiceNumber=3`) |
| `getGamesAndGiftCardsCategories()` | جلب فئات الألعاب والبطاقات (`ServiceNumber=4`) |

#### 3.4 سمة `NflowExtendedEndpoints` (الدوال العامة والاختصارات)

##### أ. الدوال العامة المرنة

| الدالة | الوصف |
|--------|-------|
| `executeService($networkNumber, $serviceNumber, $additionalFields, $transactionId)` | تنفيذ أي خدمة بأرقام الشبكة والخدمة يدوياً. |
| `queryBalance($networkNumber, $mobileNumber, $transactionId)` | استعلام رصيد لأي شبكة تدعم الخدمة (تحدد رقم الخدمة تلقائياً). |
| `chargeBalanceGeneric($networkNumber, $mobileNumber, $amount, $transactionId, $webHookUrl, $webHookCode)` | شحن رصيد لأي شبكة تدعم الخدمة (تحدد رقم الخدمة تلقائياً). |
| `activateOfferGeneric($networkNumber, $mobileNumber, $offerCode, $transactionId, $webHookUrl, $webHookCode)` | تفعيل باقة لأي شبكة تدعم الخدمة (تحدد رقم الخدمة تلقائياً). |

##### ب. دوال الاختصار للشبكات

| الدالة | الشبكة |
|--------|--------|
| `queryYemenMobileBalance($mobileNumber, $transactionId)` | يمن موبايل – استعلام الرصيد |
| `queryYemenMobileLoan($mobileNumber, $transactionId)` | يمن موبايل – استعلام القرض |
| `queryYemenMobileOffers($mobileNumber, $transactionId)` | يمن موبايل – استعلام الباقات |
| `queryAdslBalance($mobileNumber, $transactionId)` | ADSL |
| `queryLandPhoneBalance($mobileNumber, $transactionId)` | هاتف أرضي |
| `queryYemen4GBalance($mobileNumber, $transactionId)` | Yemen 4G |
| `queryAdenNetBalance($mobileNumber, $transactionId)` | عدن نت |

##### ج. دوال الدفع الأصلية (للتوافق)

| الدالة | الوصف |
|--------|-------|
| `chargeBalance($networkNumber, $serviceNumber, $mobileNumber, $amount, $transactionId, $webHookUrl, $webHookCode)` | شحن رصيد مع تحديد رقم الخدمة يدوياً. |
| `activateOffer($networkNumber, $serviceNumber, $mobileNumber, $offerCode, $transactionId, $webHookUrl, $webHookCode)` | تفعيل باقة مع تحديد رقم الخدمة يدوياً. |
| `chargeGameOrGiftCard($linkCode, $fields, $quantity, $transactionId, $mobileNumber, $webHookUrl, $webHookCode)` | شحن لعبة أو بطاقة هدايا. |

#### 3.5 سمة `NflowDataEndpoints` (البيانات والمساعدة)

| الدالة | الوصف |
|--------|-------|
| `getNetworksList()` | قائمة الشبكات المتاحة (مع أرقامها وأسمائها). |
| `getNetworkName($networkNumber)` | اسم الشبكة من رقمها. |
| `isValidNetwork($networkNumber)` | التحقق من صحة رقم الشبكة. |
| `getAllServices()` | قائمة كاملة بالخدمات لكل شبكة (كما في جدول Services Data). |
| `getServiceName($networkNumber, $serviceNumber)` | وصف الخدمة من أرقام الشبكة والخدمة. |
| `getServicesForNetwork($networkNumber)` | قائمة الخدمات لشبكة معينة. |
| `isValidService($networkNumber, $serviceNumber)` | التحقق من صحة رقم الخدمة لشبكة معينة. |
| `getBillBalanceServiceNumber($networkNumber)` | رقم خدمة شحن الرصيد للشبكة. |
| `getQueryBalanceServiceNumber($networkNumber)` | رقم خدمة استعلام الرصيد للشبكة. |
| `getQueryOffersServiceNumber($networkNumber)` | رقم خدمة الباقات للشبكة. |
| `supportsBalanceQuery($networkNumber)` | هل الشبكة تدعم استعلام الرصيد؟ |
| `supportsBalanceBill($networkNumber)` | هل الشبكة تدعم شحن الرصيد المباشر؟ |
| `supportsOffers($networkNumber)` | هل الشبكة تدعم الباقات؟ |
| `isValidMobileNumber($mobileNumber, $networkNumber)` | التحقق من صحة رقم الجوال حسب الشبكة. |
| `formatMobileNumber($mobileNumber)` | تنسيق الرقم (إزالة الأصفار الزائدة). |
| `getOperationStatusText($operationStatus)` | تحويل رمز حالة العملية إلى نص عربي. |
| `getAvailableServicesWithDetails($networkNumber)` | قائمة الخدمات مع التصنيف (استعلام، دفع، باقة). |

#### 3.6 متحكم `NflowController`

يقع المتحكم في المسار `Nano3\TelecomRecharge\APIControllers\NflowController`، ويوفر نقاط نهاية RESTful محمية بـ OAuth. جميع الدوال تستدعي `NflowService` وتتعامل مع الاستجابات بشكل موحد.

**الهيكل العام للمتحكم:**

- دالة `getNflowServiceObj()`: تهيئة الخدمة وتسجيل الدخول.
- دوال عامة: `executeService`, `queryBalance`, `chargeBalanceGeneric`, `activateOfferGeneric`.
- دوال العمليات العامة: `getAccountBalance`, `getOperationStatus`, `getFeedClientsBalance`, `getGamesAndGiftCardsCategories`.
- دوال استعلامات الشبكات (اختصارات): `queryYemenMobileBalance`, `queryYemenMobileLoan`, `queryYemenMobileOffers`, `queryAdslBalance`, `queryLandPhoneBalance`, `queryYemen4GBalance`, `queryAdenNetBalance`.
- دوال شحن (اختصار): `chargeYemenMobileBalance`.
- دوال تفعيل الباقات (اختصار): `activateYouOffer`, `activateSabafonOffer`.
- دوال الألعاب: `chargeGameOrGiftCard`.
- دوال البيانات: `getNetworksList`, `getAllServices`, `testConnection`.
- دوال مساعدة: `handleApiResponse`, `errorResponse`, `errorWrongArgs`, `errorNotFound`.

#### 3.7 ملف التوجيه (routes.php)

تم إنشاء ملف توجيه متكامل يوفر جميع نقاط النهاية مع استخدام مجموعتين:

- المجموعة الخارجية: `prefix = api/v1/telecomrecharge/nflow`، وتحتوي على `cors`, `api`, `web`.
- المجموعة الداخلية: `middleware = oauth-users`، `as = nflow.`، وتحتوي على جميع المسارات.

**نقاط النهاية الرئيسية:**

| المسار | الطريقة | الاسم | الوصف |
|--------|--------|-------|-------|
| `execute` | POST | `nflow.execute` | تنفيذ خدمة مخصصة |
| `balance/query` | GET | `nflow.balance.query` | استعلام رصيد أي شبكة |
| `balance/charge` | POST | `nflow.balance.charge` | شحن رصيد أي شبكة |
| `offer/activate` | POST | `nflow.offer.activate` | تفعيل باقة أي شبكة |
| `account-balance` | GET | `nflow.account-balance` | رصيد الوكيل |
| `operation-status` | GET | `nflow.operation-status` | حالة معاملة |
| `feed-clients-balance` | GET | `nflow.feed-clients-balance` | إيداعات الوكيل |
| `games-categories` | GET | `nflow.games-categories` | فئات الألعاب |
| `yemen-mobile/balance` | GET | `nflow.yemen-mobile.balance` | رصيد يمن موبايل |
| `yemen-mobile/loan` | GET | `nflow.yemen-mobile.loan` | قرض يمن موبايل |
| `yemen-mobile/offers` | GET | `nflow.yemen-mobile.offers` | باقات يمن موبايل |
| `adsl/balance` | GET | `nflow.adsl.balance` | رصيد ADSL |
| `land-phone/balance` | GET | `nflow.land-phone.balance` | رصيد هاتف أرضي |
| `yemen-4g/balance` | GET | `nflow.yemen-4g.balance` | رصيد Yemen 4G |
| `aden-net/balance` | GET | `nflow.aden-net.balance` | رصيد عدن نت |
| `yemen-mobile/charge` | POST | `nflow.yemen-mobile.charge` | شحن يمن موبايل |
| `you/offer/activate` | POST | `nflow.you.offer.activate` | تفعيل باقة YOU |
| `sabafon/offer/activate` | POST | `nflow.sabafon.offer.activate` | تفعيل باقة سبافون |
| `game-charge` | POST | `nflow.game-charge` | شحن لعبة أو بطاقة |
| `networks` | GET | `nflow.networks` | قائمة الشبكات |
| `services` | GET | `nflow.services.list` | قائمة الخدمات |
| `test-connection` | GET | `nflow.test-connection` | اختبار الاتصال |

#### 3.8 التوثيق

تم إنشاء ملفين توثيقيين باللغة العربية ضمن مسار `docs/TelecomRecharge/`:

1. **`Docs-NflowService-Class-ar.md`**: وثيقة شاملة للكلاس `NflowService` تتضمن:
   - مقدمة عن الكلاس وأهدافه.
   - المتطلبات والإعداد (بيانات الاعتماد، متغيرات البيئة).
   - هيكل الاستجابة.
   - شرح جميع الدوال مع أمثلة.
   - دعم Webhook.
   - معالجة الأخطاء.
   - ملخص نهائي.

2. **`Docs-NflowService-Advanced-Examples-ar.md`**: وثيقة الأمثلة المتقدمة وتشمل:
   - مقدمة عن الأمثلة.
   - المتطلبات الأساسية.
   - أمثلة للعمليات العامة (رصيد الوكيل، حالة المعاملة، إيداعات الوكيل).
   - أمثلة لاستعلامات الرصيد (يمن موبايل، ADSL، هاتف أرضي، Yemen 4G، عدن نت).
   - أمثلة لشحن الرصيد (باستخدام الدوال العامة والاختصارات).
   - أمثلة لتفعيل الباقات.
   - أمثلة لشحن الألعاب وبطاقات الهدايا.
   - أمثلة لاستخدام الدوال العامة المرنة (`executeService`).
   - أمثلة لاستخدام Webhook.
   - أمثلة للتحقق من صحة البيانات باستخدام دوال المساعدة.
   - أمثلة لمعالجة الأخطاء.
   - أفضل الممارسات.
   - اختبار الاتصال.

3. **`Nflow-API-Documentation-ar.md`**: توثيق واجهة برمجة التطبيقات (نقاط النهاية) مع أمثلة باستخدام cURL.

---

### 4. آلية العمل (التدفق الجديد)

1. **تهيئة الكلاس**: يتم إنشاء كائن `NflowService` إما بتمرير بيانات الاعتماد مباشرة أو باستخدام متغيرات البيئة.
2. **توليد التوكن المؤقت**: يتم استدعاء `generateToken()` لحساب `Token` باستخدام MD5.
3. **تسجيل الدخول**: يتم استدعاء `login()` مع نقطة النهاية المناسبة للحصول على `accessToken`.
4. **تنفيذ العمليات**: يتم استدعاء الدوال المناسبة (مثل `queryBalance` أو `chargeBalanceGeneric`) التي تقوم بتجميع البيانات واستدعاء `executeService`.
5. **إرسال الطلب**: دالة `post()` تقوم بإرسال الطلب إلى `/rest-api/` مع إضافة `api-token` في الهيدر.
6. **معالجة الاستجابة**: دالة `parseResponse()` تحلل JSON وتعيد مصفوفة موحدة تحتوي على `success`, `data`, `status_code`.
7. **معالجة الأخطاء**: في حالة حدوث استثناء، يتم استدعاء `handleException()` التي تعيد رسالة خطأ آمنة حسب بيئة التشغيل.

---

### 5. أبرز الإنجازات والميزات

- **إضافة برمجية متكاملة**: تم إطلاق إضافة `Nano3.TelecomRecharge` كحل موحد للتعامل مع مزودي الشحن الرقمي المحليين.
- **تغطية كاملة لنقاط النهاية**: جميع العمليات الموثقة في وثائق nflow.tech مغطاة بدوال محددة.
- **دوال عامة مرنة**: يمكن تنفيذ أي خدمة بأرقام الشبكة والخدمة عبر `executeService`، مما يسهل إضافة شبكات جديدة مستقبلاً.
- **دعم Webhook**: إمكانية تمرير `WebHookURL` و `WebHookCode` في عمليات الدفع، واستقبال تحديثات الحالة عبر GET.
- **معالجة أخطاء احترافية**: عرض رسائل آمنة للمستخدم في الإنتاج، مع تسجيل التفاصيل الكاملة في بيئة التطوير.
- **دعم متغيرات البيئة**: يمكن ضبط بيانات الاعتماد عبر `.env` لتسهيل التهيئة.
- **فصل المسؤوليات**: استخدام السمات (Traits) جعل الكود أكثر تنظيماً وقابلية للصيانة.
- **متحكم API متكامل**: مع نقاط نهاية محمية بـ OAuth وتنسيق استجابة موحد.
- **توثيق شامل**: ملفات توثيق تفصيلية باللغة العربية تغطي جميع الجوانب.

---

### 6. الفوائد والقيمة المضافة

- **للمطورين**:
  - إمكانية دمج خدمات nflow.tech في تطبيقات نانوسوفت بسهولة.
  - واجهة برمجية نظيفة وموثقة.
  - توفير وقت التطوير من خلال دوال مختصرة وعامة.
  - إمكانية التوسع بإضافة مزودين جدد دون تعديل الكود الأساسي.

- **للمستخدمين النهائيين**:
  - تجربة سلسة في شحن الرصيد وتفعيل الباقات.
  - تحديثات فورية عبر Webhook.
  - رسائل خطأ واضحة تساعد في تصحيح الإدخالات.

- **للنظام ككل**:
  - أمان محسن عبر المصادقة المزدوجة (OAuth + API Token).
  - مرونة عالية في التكامل مع بوابات الدفع المختلفة.
  - هيكل قابل للتوسع لدعم شبكات جديدة في المستقبل.

---

### 7. خطط التطوير المستقبلية

- **دعم مزودين إضافيين**: إضافة دعم لمزودي خدمات محليين آخرين (مثل Sadad، YouGotaGift) عبر تطوير كلاسات متخصصة ضمن نفس الإضافة.
- **واجهة إدارة رسومية**: تطوير واجهة إدارة لمراقبة العمليات وسجل المعاملات.
- **دعم عمليات إضافية**: إضافة دوال لعمليات مثل استعلام رصيد الوكيل بالتفصيل، وإدارة الفواتير.
- **تحسين نظام التخزين المؤقت**: تخزين نتائج الاستعلامات المتكررة لتقليل استهلاك API.
- **دعم التراجع (Undo)**: إمكانية التراجع عن عمليات الشحن الأخيرة (إذا أمكن من خلال API).

---

### 8. الخاتمة

يمثل إطلاق إضافة `Nano3.TelecomRecharge` خطوة مهمة في دمج خدمات الشحن الرقمي المحلي في نظام نانوسوفت. من خلال توفير كلاس `NflowService`، ومتحكم `NflowController`، ونظام متكامل من السمات والدوال المساعدة، أصبح بإمكان المطورين دمج خدمات nflow.tech في تطبيقاتهم بسرعة وأمان. الإضافة مصممة لتكون قابلة للتوسع، مما يسمح بإضافة مزودين جدد مستقبلاً دون الحاجة إلى إعادة هيكلة الكود الأساسي.

مع الانتهاء من هذه المرحلة، تصبح الإضافة جاهزة للاستخدام في مشاريع نانوسوفت المختلفة، ونتطلع إلى مواصلة التطوير بناءً على ملاحظات المستخدمين والمتطلبات المتطورة.

**الوثائق المرجعية**:
- [التوثيق الداخلي للكلاس NflowService](./docs/TelecomRecharge/Docs-NflowService-Class-ar.md)
- [أمثلة عملية متقدمة](./docs/TelecomRecharge/Docs-NflowService-Advanced-Examples-ar.md)
- [توثيق واجهة API (نقاط النهاية)](./docs/TelecomRecharge/Nflow-API-Documentation-ar.md)

## 2026-04-23 - 2026-04-25

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.5**

### ملخص التحديثات

بعد تحسين أداء علاقات KYC في API (الإصدار 1.2.4) وإضافة آليات تخزين مؤقت متقدمة (الإصدار 1.2.3)، نركز في الإصدار 1.2.5 على **تعزيز أمان وموثوقية الوثائق** من خلال نظام حماية متكامل للوثائق المعتمدة. يهدف هذا الإصدار إلى:

- **منع تعديل الوثائق بعد اعتمادها** إلا وفق صلاحيات محددة.
- **توفير تحكم دقيق** في الصلاحيات عبر إعدادات مرنة (مالك، مستخدم backend، صلاحيات خاصة).
- **حساب درجة التحقق تلقائياً** عند الاعتماد.
- **تعيين المحقّق تلقائياً** بناءً على المستخدم الحالي.
- **منع تكرار نفس نوع الوثيقة لنفس المالك** (اختياري).

هذه التحسينات تجعل نظام KYC أكثر أماناً وجاهزية للاستخدام في بيئات تتطلب امتثالاً تنظيمياً صارماً، مع الحفاظ على مرونة عالية للمطورين.

---

### الإصدار 1.2.5 – نظام حماية الوثائق المعتمدة وتحسينات الأمان

#### أهداف الإصدار

- تطوير نظام حماية متكامل يمنع تعديل الوثائق المعتمدة (`is_verified = true`) وغير المنتهية.
- توفير دوال مساعدة احترافية للتحقق من صلاحية التعديل (`canModifyDocument`، `shouldProtectVerifiedDocument`).
- دعم إعدادات مرنة للتحكم في سلوك الحماية (`protect_verified` في `config.php`).
- دمج نظام صلاحيات `Backend` لمنح استثناءات للمستخدمين الإداريين.
- تحسين دالة `prepareChangesIsVerified` لحساب `verification_score` تلقائياً.
- إضافة دالة `preperSetVerifier` لتعيين المحقّق تلقائياً عند الاعتماد.
- تفعيل خيار منع تكرار نفس نوع الوثيقة للمالك (`is_check_duplicate`).
- إضافة دالة `isDocumentValid` لفحص صلاحية الوثيقة (انتهاء/عمر أقصى).

#### الميزات الجديدة

##### 1. نظام منع تعديل الوثائق المعتمدة (`protect_verified`)

تم إضافة قسم جديد في ملف الإعدادات `config.php` باسم `protect_verified` يتيح تحكماً كاملاً في سياسة تعديل الوثائق المعتمدة:

```php
'protect_verified' => [
    'is_check' => true,                     // تفعيل/تعطيل الحماية كلياً
    'allow_owner_modify_verified' => false, // السماح لمالك الوثيقة بتعديلها
    'backend_permissions' => 'nano3.kyc.documents.edit_verified', // صلاحية خاصة لـ Backend
    'allowed_fields' => [                   // الحقول المسموح تعديلها للوثائق المعتمدة
        'verification_score',
        'is_verified',
        'status',
        'metadata',
        'other_data',
        'config_data',
    ],
],
```

| الإعداد | الوصف | القيمة الافتراضية |
| :--- | :--- | :--- |
| `is_check` | تفعيل الحماية بالكامل. إذا `false`، يمكن تعديل أي وثيقة. | `true` |
| `allow_owner_modify_verified` | السماح للمالك (صاحب الوثيقة) بتعديلها حتى لو كانت معتمدة. | `false` |
| `backend_permissions` | صلاحية (أو صلاحيات) مطلوبة من مستخدم `Backend` لتجاوز الحماية. يمكن أن تكون نصاً واحداً أو مفصولة بفواصل. | `'nano3.kyc.documents.edit_verified'` |
| `allowed_fields` | الحقول الوحيدة المسموح بتعديلها عند تطبيق الحماية (للمستخدمين غير المصرح لهم). | قائمة بالحقول الآمنة مثل `metadata` و `status`. |

**آلية العمل:**

1. عند محاولة حفظ وثيقة (تحديث)، تُستدعى دالة `protectVerifiedDocument()` داخل `beforeSave`.
2. تتحقق الدالة من `canModifyDocument()` لمعرفة ما إذا كان المستخدم الحالي مسموحاً له بالتعديل.
3. إذا كان مسموحاً، يُسمح بالتعديل الكامل.
4. إذا كان ممنوعاً، يتم منع التعديل بشكل كامل أو السماح فقط بالحقول المحددة في `allowed_fields` (حسب نوع المستخدم).

##### 2. دوال جديدة للحماية والتحقق

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `canModifyDocument(?Document $doc, $user)` | `public static` | تحدد ما إذا كان المستخدم (أو المستخدم الحالي) مسموحاً له بتعديل الوثيقة. تراعي الإعدادات والصلاحيات. |
| `shouldProtectVerifiedDocument(?Document $doc)` | `public static` | فحص سريع لمعرفة ما إذا كان يجب تطبيق الحماية على الوثيقة (بغض النظر عن المستخدم). |
| `isDocumentValid(Document $doc)` | `public static` | تتحقق من صلاحية الوثيقة (تاريخ الانتهاء مستقبلي، ضمن العمر الأقصى). |
| `checkBackendPermissions($user, $permissions)` | `protected static` | تتحقق من صلاحيات مستخدم `Backend` باستخدام نظام صلاحيات OctoberCMS. |
| `protectVerifiedDocument()` | `protected` | تطبق الحماية أثناء `beforeSave`، وتمنع التعديل غير المصرح به. |
| `prepareDuplicate()` | `public` | تمنع تكرار نفس نوع الوثيقة لنفس المالك (إذا فُعّل `is_check_duplicate`). |
| `prepareChangesIsVerified()` | `public` | تعالج تغييرات `is_verified`، تحسب `verification_score` تلقائياً، وتعين `verified_at`. |
| `preperSetVerifier($user)` | `public` | تعين `verifier_id` و `verifier_type` تلقائياً من المستخدم الحالي عند الاعتماد. |

##### 3. تحسين عملية الاعتماد (`verification`)

- عند تعيين `is_verified = true`، تقوم `prepareChangesIsVerified` بما يلي:
  - تعيين `verified_at` إلى الوقت الحالي.
  - استدعاء `preperSetVerifier()` لتعيين المحقّق تلقائياً (إذا لم يكن محدداً).
  - حساب `verification_score` من `DocumentType::getVerificationWeight()` إذا لم يتم تحديده يدوياً.
  - تغيير `status` إلى `verified` إذا كانت الحالة `pending` أو `rejected` أو `draft`.

##### 4. منع تكرار الوثائق (`is_check_duplicate`)

تم تحسين دالة `prepareDuplicate()` لتعمل وفق الإعداد `is_check_duplicate`. عند تفعيله، يتم منع إنشاء وثيقة جديدة من نفس `document_type` لنفس `owner` إذا كانت موجودة مسبقاً (باستثناء نفس السجل عند التحديث). هذا يضمن عدم ازدواجية الوثائق الأساسية.

##### 5. تكامل مع نظام الصلاحيات

- يمكن لمستخدمي `Backend` الذين يملكون الصلاحية `nano3.kyc.documents.edit_verified` (أو المحددة في الإعدادات) تعديل الوثائق المعتمدة دون قيود.
- إذا لم تُحدد أي صلاحية، يُسمح لجميع مستخدمي `Backend` بتعديل الوثائق المعتمدة (للاستخدام الداخلي).
- يمكن أيضاً السماح لمالك الوثيقة (مستخدم `frontend`) بتعديل وثيقته المعتمدة عبر `allow_owner_modify_verified`.

---

### أمثلة عملية

#### 1. محاولة مستخدم عادي (frontend) تعديل وثيقة معتمدة

لدى المستخدم `أحمد` وثيقة جواز سفر معتمدة (`is_verified = true`). يحاول تحديث حقل `document_number`.

**النتيجة:**  
يتم رفض الطلب مع رسالة خطأ:  
`"لا يمكن تعديل وثيقة معتمدة وصالحة."`

> إذا كان `allow_owner_modify_verified = true`، سيُسمح لأحمد بالتعديل.

#### 2. مستخدم Backend لديه صلاحية `nano3.kyc.documents.edit_verified`

يحاول نفس المستخدم `Backend` (يملك الصلاحية) تعديل الوثيقة.

**النتيجة:**  
يتم قبول التعديل دون قيود.

#### 3. محاولة إنشاء وثيقة مكررة (مع `is_check_duplicate = true`)

لدى المستخدم `أحمد` بالفعل وثيقة `passport`. يحاول إنشاء وثيقة `passport` أخرى.

**النتيجة:**  
يتم رفض الطلب مع رسالة خطأ:  
`"لا يمكن تكرار نفس نوع الوثيقة لنفس المالك."`

#### 4. اعتماد وثيقة جديدة – حساب `verification_score` تلقائياً

عند اعتماد وثيقة `passport` دون تحديد `verification_score`:

```php
$document->is_verified = true;
$document->save();
```

**النتيجة:**  
يتم تعيين `verification_score = 100` (وزن جواز السفر)، ويتم تعيين `verifier` بالمستخدم الحالي، وتصبح الحالة `verified`.

---

### ملخص الإصدارات (1.2.4 – 1.2.5)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.2.4 | تحسين أداء سلوك `DynamicAddIncludeKyc`، إضافة 7 علاقات جديدة (`kyc_verification_summary` و 6 فاحصات فئات)، دعم `expiring_within_days`، تحديث شامل للتوثيق. |
| 1.2.5 | نظام حماية متكامل للوثائق المعتمدة، منع التكرار، تحسينات عملية الاعتماد (حساب درجة التحقق، تعيين المحقق تلقائياً)، تكامل مع صلاحيات Backend، إعدادات مرنة في `config.php`. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `models/Document.php` بالنسخة الجديدة.
   - تحديث ملف `config/config.php` لإضافة قسم `protect_verified`.

2. **تشغيل هجرات قاعدة البيانات** (لا توجد تغييرات في هذا الإصدار).

3. **تحديث ملف اللغة**:
   - إضافة المفاتيح التالية إلى `lang.php`:
     ```php
     'msg.cannot_modify_verified_document' => 'لا يمكن تعديل وثيقة معتمدة وصالحة.',
     'msg.cannot_modify_verified_document_field' => 'لا يمكن تعديل الحقل ":field" لوثيقة معتمدة وصالحة.',
     ```

4. **مراجعة الصلاحيات**:
   - تأكد من منح صلاحية `nano3.kyc.documents.edit_verified` للمستخدمين الإداريين الذين يحتاجون تعديل الوثائق المعتمدة.

5. **اختبار الحماية**:
   - جرب تعديل وثيقة معتمدة كمستخدم `frontend` للتأكد من رفض العملية.
   - جرب نفس العملية كمستخدم `Backend` بالصلاحية المناسبة.

---

### الخاتمة

مع الإصدار 1.2.5، نكون قد أضفنا طبقة أمان متكاملة تمنع التلاعب بالوثائق بعد اعتمادها، مع الحفاظ على مرونة كاملة للمطورين عبر إعدادات دقيقة. كما تم تحسين تجربة الاعتماد بحيث تتم العمليات الروتينية (حساب الدرجة، تعيين المحقق) تلقائياً. هذه التحسينات تجعل إضافة `Nano3.Kyc` مناسبة لأكثر السيناريوهات تعقيداً وتطلباً للأمان والامتثال.

نتطلع إلى ملاحظاتكم ومقترحاتكم لمواصلة تطوير هذه الإضافة الحيوية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

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

- [YottaGate API Documentation ](./docs/YottaPay/Docs-YottaPay-ar.md)
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


## 2026-04-25 - 2026-04-26

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.6**

### ملخص التحديثات

بعد تعزيز أمان وثائق KYC في الإصدار 1.2.5، نقدم في الإصدار 1.2.6 تحسينات شاملة على **واجهة برمجة التطبيقات (API)** لجلب بيانات التصنيفات وأنواع الوثائق. تهدف هذه التحديثات إلى توفير بيانات منظمة ومرنة تناسب تطبيقات الواجهة الأمامية (Frontend)، مع دعم الترجمة وخيارات تضمين متقدمة.

يركز هذا الإصدار على:

- **نقاط نهاية جديدة** لجلب التصنيفات (`document-categories-list`) والأنواع (`document-types-list`).
- **دوال متقدمة في `DocumentType`** لإرجاع بيانات مهيكلة تدعم الترجمة وتضمين العلاقات (الأنواع، التفاصيل، الحقول، قواعد التحقق).
- **تحسينات في معالجة الأخطاء** داخل `FileUploadDocuments` لإرجاع معلومات تصحيحية مفصلة.
- **توحيد مخرجات API** باستخدام Fractal في عمليات التحقق (verify, reject, restore).

هذه التحسينات تجعل تطوير واجهات المستخدم الخاصة بـ KYC أكثر سهولة وفعالية.

---

### الإصدار 1.2.6 – نقاط نهاية جديدة للتصنيفات والأنواع وتحسينات API

#### أهداف الإصدار

- توفير نقاط نهاية API احترافية لجلب تصنيفات وأنواع وثائق KYC ببيانات منظمة.
- دعم خيارات تضمين مرنة (`include_types`, `include_details`, `include_fields`, `include_rules`, `include_category`).
- تحسين دوال `DocumentType` لتوليد استجابات متوافقة مع معايير تطبيقات الواجهة الأمامية.
- توحيد استجابات API لعمليات (verify, reject, restore) باستخدام Fractal Transformer.
- تحسين تقارير الأخطاء في رفع الملفات لتسهيل عملية التصحيح.

#### الميزات الجديدة

##### 1. نقاط نهاية جديدة للتصنيفات والأنواع

تم إضافة مسارين جديدين إلى `routes.php`:

| المسار | الوصف |
| :--- | :--- |
| `GET /api/v1/kyc/document-categories-list` | جلب قائمة تصنيفات الوثائق مع خيارات تضمين متقدمة. |
| `GET /api/v1/kyc/document-categories-list/{id}` | جلب تفاصيل تصنيف محدد مع أنواعه. |
| `GET /api/v1/kyc/document-types-list` | جلب قائمة أنواع الوثائق مع خيارات تضمين متقدمة. |
| `GET /api/v1/kyc/document-types-list/{id}` | جلب تفاصيل نوع وثيقة محدد. |

##### 2. دوال جديدة ومُحسَّنة في `DocumentType`

| الدالة | الوصف |
| :--- | :--- |
| `isValidCategory($category)` | التحقق من صحة كود تصنيف. |
| `getCategoriesList($locale, $options)` | جلب قائمة التصنيفات مع دعم تضمين الأنواع والتفاصيل. |
| `getCategoryItem($code, $locale, $options)` | جلب عنصر تصنيف واحد مع خيار تضمين أنواعه. |
| `getDocumentTypeList($locale, $options)` | جلب قائمة أنواع الوثائق مع دعم الفلترة حسب التصنيف وخيارات تضمين متقدمة. |
| `getDocumentTypeItem($code, $locale, $options)` | جلب عنصر نوع وثيقة واحد مع تفاصيله الكاملة. |

**خيارات التضمين المدعومة (`$options`):**

| الخيار | الوصف |
| :--- | :--- |
| `include_types` | تضمين أنواع الوثائق داخل كل تصنيف. |
| `include_details` | تضمين تفاصيل النوع (الوزن، الحقول المطلوبة، الملفات المسموحة...). |
| `include_fields` | تضمين مخطط الحقول (`fields`) الكامل للنوع. |
| `include_rules` | تضمين قواعد التحقق (`rules`) الخاصة بكل حقل. |
| `include_category` | تضمين كائن التصنيف الأب داخل عنصر النوع. |
| `category_filter` | فلترة الأنواع حسب تصنيف محدد (مثلاً `primary_id`). |

##### 3. تحسينات في معالجة الأخطاء لرفع الملفات

تم تحسين دوال `create` و `update` في `FileUploadDocuments` لتشمل:

- إرجاع تفاصيل أخطاء التحقق (`errors`) من `Manager`.
- تضمين معلومات التصحيح (`debug`) عند تفعيل `app.debug`.
- استخدام رسائل استثناء أكثر دقة.

##### 4. توحيد مخرجات API لعمليات التحقق

تم تعديل دوال `verify`, `reject`, `restore` في `Documents` API Controller لاستخدام `DocumentTransformer` عبر Fractal، مما يضمن تناسق شكل البيانات مع باقي نقاط النهاية.

---

### أمثلة عملية

#### 1. جلب قائمة التصنيفات فقط

**الطلب:**
```http
GET /api/v1/kyc/document-categories-list HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب بيانات التصنيفات بنجاح",
    "data": [
        {
            "code": "primary_id",
            "name_ar": "هوية أساسية",
            "name_en": "Primary ID",
            "name": "هوية أساسية"
        },
        ...
    ]
}
```

#### 2. جلب تصنيف محدد مع أنواعه وتفاصيلها

**الطلب:**
```http
GET /api/v1/kyc/document-categories-list/primary_id?include_types=1&include_details=1 HTTP/1.1
```

**الاستجابة (مقتطف):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "code": "primary_id",
        "name": "هوية أساسية",
        "types": [
            {
                "code": "passport",
                "name": "جواز سفر",
                "details": {
                    "verification_weight": 100,
                    "has_expiry": true,
                    "required_fields": ["document_front", "document_number", ...]
                }
            }
        ]
    }
}
```

#### 3. جلب جميع أنواع الوثائق مع التصنيف الأب والحقول والقواعد

**الطلب:**
```http
GET /api/v1/kyc/document-types-list?include_category=1&include_fields=1&include_rules=1 HTTP/1.1
```

#### 4. جلب تفاصيل نوع وثيقة محدد (جواز السفر) مع كل البيانات

**الطلب:**
```http
GET /api/v1/kyc/document-types-list/passport?include_category=1&include_details=1&include_fields=1&include_rules=1 HTTP/1.1
```

---

### ملخص الإصدارات (1.2.5 – 1.2.6)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.2.5 | نظام حماية متكامل للوثائق المعتمدة، منع التكرار، تحسينات عملية الاعتماد، تكامل مع صلاحيات Backend. |
| 1.2.6 | نقاط نهاية جديدة للتصنيفات والأنواع (`document-categories-list`, `document-types-list`)، دوال `DocumentType` متقدمة، تحسين معالجة الأخطاء، توحيد مخرجات API. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `classes/DocumentType.php` بالنسخة الجديدة.
   - استبدال ملف `apicontrollers/Documents.php` و `apicontrollers/documents/FileUploadDocuments.php`.
   - تحديث ملف `routes.php` لإضافة المسارات الجديدة.

2. **تشغيل هجرات قاعدة البيانات** (لا توجد تغييرات في هذا الإصدار).

3. **اختبار نقاط النهاية الجديدة**:
   - جرب `GET /api/v1/kyc/document-categories-list` مع خيارات تضمين مختلفة.
   - جرب `GET /api/v1/kyc/document-types-list/passport?include_fields=1`.

4. **مراجعة التوثيق**:
   - تم تحديث ملفات التوثيق `docs.md` الخاصة بنقاط النهاية الجديدة مع أمثلة كاملة.

---

### الخاتمة

مع الإصدار 1.2.6، نكون قد أكملنا بناء واجهة API احترافية ومرنة لنظام KYC، حيث يمكن للمطورين الآن جلب بيانات التصنيفات والأنواع بسهولة مع تحكم كامل في البيانات المضمنة. هذا يسهل بناء نماذج ديناميكية وواجهات مستخدم غنية دون الحاجة إلى طلبات متعددة أو معالجة يدوية للبيانات.

نتطلع إلى ملاحظاتكم ومقترحاتكم لمواصلة تطوير هذه الإضافة الحيوية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

## 2026-04-25 – 2026-04-27

**تحديث إضافة `Nano.Coupons` – إضافة ميزة "حد المنتج" (Product Limit) ونظام تقييد كميات الطلبات**

---

### ملخص التحديثات

تم تطوير وإطلاق ميزة جديدة بالكامل في إضافة `Nano.Coupons` باسم **"حد المنتج" (`Product Limit`)**، تتيح للمسؤولين إنشاء قواعد تقييد تلقائية على كميات منتجات محددة أو عدد مرات طلبها خلال فترة زمنية معينة. القاعدة تعمل بشكل فوري: عند محاولة العميل إضافة منتج مخالف للعربة، يتم رفض الإضافة مع رسالة خطأ واضحة، كما يتم التحقق مرة أخرى عند محاولة إتمام الطلب.

الميزة مبنية على البنية الحالية للكوبونات ونظام الشروط (`Cart Conditions`) في `Nano.Cart`، وتم تمديدها بإضافة نوع جديد `product_limit` إلى جانب الأنواع السابقة (`whole_cart`, `delivery_fee`, `menu_items`). تم توفير واجهة إدارة متكاملة عبر الحقول الجديدة في نموذج الكوبون، مع دعم الترجمة وظهور/إخفاء الحقول بناءً على النوع المختار. النظام يراعي جميع القيود الأخرى مثل تاريخ صلاحية الكوبون، نوع الطلب، والمتجر المحدد.

---

### 1. أهداف التحديثات

- **تمكين إنشاء قواعد تقييد تلقائية للمنتجات** دون الحاجة إلى كود مخصص أو تدخل يدوي من المطور.
- **منع تجاوز الكميات القصوى** للمنتجات أو الوحدات خلال فترة زمنية محددة، أو تجاوز عدد مرات الطلب.
- **تكامل سلس مع نظام العربة** عبر شرط `ProductLimit` الذي يستمع إلى حدث `nano.cart.adding` ويمنع الإضافة الفورية.
- **استغلال نموذج الكوبون الحالي** لتخزين القواعد، مما يسمح بإعادة استخدام صلاحيات التاريخ، أنواع الطلبات، والمتاجر.
- **واجهة إدارة سهلة** عبر حقل `recordfinder` لاختيار المنتج، وخيارات رقمية للحدود.
- **توثيق كامل** للتعديلات البرمجية وللمفاتيح الترجمية المستخدمة.

---

### 2. المكونات المطورة

#### 2.1 نموذج `Coupon` و `Coupons_model`

- **إضافة النوع الجديد `product_limit`** إلى دالة `getApplyCouponOnOptions`.
- **إضافة الخصائص الجديدة** إلى مصفوفة `$casts` لدعم القيم الصحيحة.
- **إضافة الأعمدة الجديدة** إلى قاعدة البيانات عبر Migration:
  - `product_id` (int) – رقم المنتج
  - `units_id` (int, nullable) – رقم الوحدة (اختياري)
  - `max_quantity` (int, default 0) – الحد الأقصى للكمية
  - `max_orders` (int, default 0) – الحد الأقصى لعدد الطلبات
  - `period_days` (int, default 0) – الفترة بالأيام
- **توفير دوال الخيارات المنسدلة** (`getProductIdOptions`, `getUnitsIdOptions`) لدعم واجهة الإدارة.

#### 2.2 شرط العربة `ProductLimit` (Cart Condition)

الملف: `plugins/nano/coupons/cartconditions/ProductLimit.php`

- **كلاس جديد** يمتد من `CartCondition` ويسجل نفسه تلقائيًا.
- **تحميل القيود النشطة** من جميع كوبونات `product_limit` السارية مرة واحدة عند أول استخدام.
- **مستمع لحدث `nano.cart.adding`** للتحقق الفوري من أي صنف يضاف إلى العربة، ورفضه إن خالف القواعد.
- **فحص إضافي عند `beforeApply`** للتحقق من العربة كاملة قبل إتمام الطلب (يعالج حالة استعادة عربة محفوظة).
- **مراعاة القيود الإضافية**: نوع الطلب (`order_restriction`)، المتجر (`shop_restriction`)، تاريخ صلاحية الكوبون، وعدد مرات الاستخدام العام والخاص.
- **استعلامات دقيقة** لجمع الكميات التاريخية وعدد الطلبات السابقة من جدولي `nano_orders_orders` و `nano_orders_items` مع فلترة بالحالة (باستثناء `CART` و `CANCELLED`).
- **دعم الترجمة** عبر مفاتيح `Lang::get` للرسائل مع تمرير المتغيرات.
- **معالجة أمان** لعدم تكرار تسجيل المستمعين ولمنع الأخطاء في حال غياب `OrderManager`.

#### 2.3 واجهة الإدارة (fields.yaml)

- **إضافة الحقول الجديدة**: `product_id`, `units_id`, `max_quantity`, `max_orders`, `period_days`.
- **استخدام `trigger`** لإظهار هذه الحقول فقط عند اختيار `apply_coupon_on = product_limit`.
- **حقل `product_id`** من نوع `recordfinder` مع `useRelation: false` و `modelClass` للبحث عن المنتجات.
- **دعم الترجمة** لجميع خصائص الحقول: `label`, `commentAbove`, `placeholder`, `emptyOption`, `title`, `prompt`.

#### 2.4 ملفات الترجمة

- **إضافة مفاتيح ترجمة عربية وإنجليزية** لرسائل الخطأ (`max_quantity_exceeded`, `max_orders_exceeded`) ولتسميات الحقول في واجهة الإدارة.
- **توسيع مفاتيح `apply_coupon_on_options`** لتشمل `product_limit`.

#### 2.5 ملف الهجرة (Migration)

- `add_product_limit_fields.php` لإضافة الأعمدة الخمسة إلى جدول `nano_coupons_coupons` مع فحص `Schema::hasColumn` لتجنب التكرار.

---

### 3. آلية العمل (التدفق)

1. **الإعداد (مرة واحدة)**: يقوم المسؤول بإنشاء كوبون جديد من نوع `product_limit`، يحدد فيه المنتج، الوحدة (اختياري)، الحد الأقصى للكمية، الحد الأقصى للطلبات، الفترة بالأيام، وأنواع الطلبات والمتاجر المسموح بها.

2. **تفعيل الشرط**: عند تحميل نظام العربة، يقوم `ProductLimit` بتحميل جميع الكوبونات النشطة من هذا النوع وتخزينها مؤقتًا. يسجل مستمعًا للحدث `nano.cart.adding`.

3. **إضافة منتج للعربة (فحص فوري)**: عند محاولة المستخدم إضافة منتج:
   - يتحقق الشرط مما إذا كان المنتج والوحدة يتطابقان مع أي قاعدة نشطة (مع مراعاة نوع الطلب والمتجر الحالي).
   - يحسب الكمية الحالية في العربة + الكمية التاريخية من الطلبات السابقة + الكمية الجديدة.
   - إذا تجاوز المجموع `max_quantity`، يتم رمي `ApplicationException` وتظهر رسالة خطأ للمستخدم.
   - بالمثل، إذا كان عدد الطلبات التاريخية قد بلغ `max_orders`، يتم الرفض.

4. **إتمام الطلب (فحص إضافي)**: عند استدعاء `beforeApply` (حساب إجمالي العربة)، يتم تكرار نفس الفحوصات للتأكد من أن العربة الحالية لا تحتوي على كميات مخالفة (مثلاً بسبب استعادة عربة محفوظة).

5. **رسائل الخطأ**: تظهر للمستخدم رسائل واضحة مثل "تجاوزت الحد المسموح للكمية (الحد الأقصى 5)." أو "تجاوزت الحد المسموح لعدد الطلبات (الحد الأقصى 3)." مع إمكانية الترجمة.

---

### 4. أبرز الإنجازات والميزات

- **إضافة كاملة وتلقائية**: القاعدة تعمل فورًا بعد إنشاء كوبون `product_limit` نشط، دون أي كود إضافي.
- **تكامل عميق مع منظومة الكوبونات**: الاستفادة من صلاحيات التاريخ، أنواع الطلبات، والمتاجر دون إعادة اختراعها.
- **رفض فوري عند الإضافة**: معالجة فورية تمنع وصول منتجات مخالفة إلى العربة، بدلاً من الانتظار حتى مرحلة الدفع.
- **تحسين الأداء**: تحميل القيود مرة واحدة فقط لكل طلب HTTP مع تخزينها في متغير ثابت.
- **مرونة عالية**: يمكن تقييد منتج معين بغض النظر عن الوحدة، أو تقييد وحدة محددة منه.
- **دعم التعدد**: يمكن أن يكون هناك عدة قواعد لمنتجات مختلفة تعمل في نفس الوقت.
- **معالجة أخطاء آمنة**: جميع الاستعلامات داخل `try-catch` ولا تؤثر على استقرار العربة في حال فشل استعلام.
- **توثيق كامل** في ملف التحديث مع أمثلة للترجمة والحقول.

---

### 5. المتطلبات والترقية

- **ترقية الإضافة**: استبدال الملفات التالية:
  - `Nano/Coupons/Models/Coupon.php` (أو `Coupons_model.php`)
  - `Nano/Coupons/CartConditions/ProductLimit.php` (جديد)
  - `Nano/Coupons/updates/add_product_limit_fields.php` (جديد)
  - `Nano/Coupons/models/coupon/fields.yaml`
  - `Nano/Coupons/lang/ar/lang.php` و `lang/en/lang.php`
- **تشغيل الهجرة**: `php artisan october:up` لتطبيق الأعمدة الجديدة.
- **تحديث السجلات القديمة**: لا يؤثر على الكوبونات الموجودة (تظل `product_id` وغيرها بقيم افتراضية).
- **اختبار التوافق**: ينصح بإنشاء كوبون `product_limit` تجريبي واختبار إضافة منتج بعربة مستخدم لديه طلبات سابقة.

---

### 6. الفوائد والقيمة المضافة

- **لأصحاب المتاجر**: التحكم في كميات المنتجات المباعة للمستخدم الواحد، ومنع الاستغلال أو بيع منتجات محدودة بكميات كبيرة لنفس الزبون.
- **للمطورين**: إضافة مرنة يمكن تخصيصها لتناسب أي منتج أو وحدة، وتستفيد من البنية القوية للكوبونات دون بدء من الصفر.
- **للمستخدمين النهائيين**: تجربة شراء عادلة مع رسائل واضحة عند محاولة تجاوز الحدود المسموحة.

---

### 7. خطط التطوير المستقبلية

- إضافة دعم لتقييد عدة منتجات في كوبون واحد (بدلاً من منتج واحد).
- إمكانية تقييد الكمية الإجمالية عبر عدة منتجات (مثل "لا يزيد عدد المنتجات من الفئة س عن كذا").
- واجهة تقارير لمراقبة استهلاك العملاء لحدود المنتجات.
- دمج الذكاء الاصطناعي لاقتراح حدود مثالية بناءً على سلوك الشراء.

---

### 8. الخاتمة

يمثل هذا التحديث خطوة هامة نحو تمكين المتاجر من إدارة كميات المنتجات بشكل أكثر ذكاءً وتلقائية. من خلال إضافة `ProductLimit`، يمكن الآن وضع سياسات بيع دقيقة دون الحاجة إلى تعديلات برمجية، مما يعزز من مرونة النظام وقوته. نتطلع إلى مواصلة تطوير هذه الميزة بناءً على اقتراحاتكم واستخداماتكم العملية.

## 2026-04-27 - 2026-04-28

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.7**

### ملخص التحديثات

بعد إطلاق نقاط النهاية الجديدة للتصنيفات والأنواع في الإصدار 1.2.6، نقدم في الإصدار 1.2.7 تحسينًا جوهريًا على **بنية حقول الإدخال (Form Fields)** الخاصة بأنواع الوثائق. يهدف هذا الإصدار إلى توحيد آلية تعريف الحقول وعرضها والتحقق منها عبر دمج `FormFieldsManager` – النظام الموحد لإدارة حقول النماذج في منصة نانوسوفت.

يركز هذا الإصدار على:

- **إعادة هيكلة تعريفات الحقول** في `DocumentType` لتتوافق مع معايير `FormFieldsManager` (مصفوفة متسلسلة بدلاً من الترابطية).
- **دوال تحويل ذكية** تضمن التوافق الكامل مع الإصدارات السابقة (Legacy Support) دون كسر أي وظائف موجودة.
- **تحسين أداء ودقة قواعد التحقق** عبر تفويض المهمة إلى `FormFieldsManager`.
- **إثراء استجابات API** بخيارات الحقول المحلولة (Resolved Options) للقوائم المنسدلة.

هذه التحسينات تجعل نظام KYC أكثر تكاملاً مع باقي مكونات المنصة، وتفتح الباب لاستخدام أدوات موحدة لبناء النماذج الديناميكية في لوحة التحكم وواجهات API.

---

### الإصدار 1.2.7 – توافق كامل مع `FormFieldsManager` وإعادة هيكلة حقول الوثائق

#### أهداف الإصدار

- تحويل جميع تعريفات الحقول في `DocumentType` وملف الإعدادات `document_types.php` إلى تنسيق `FormFieldsManager` (مصفوفة متسلسلة).
- توفير دوال تحويل تلقائية (`convertLegacyFieldsToNewFormat`، `convertLegacyValidation`، إلخ) لضمان عدم تعطل الأنظمة القائمة.
- تفويض عمليات التحقق من صحة الحقول إلى `FormFieldsManager` لتوحيد المنطق.
- تحسين نقطة النهاية `GET /document-fields/{type}` لإرجاع الحقول مع الخيارات المحلولة (مثلاً: قائمة الدول من API).
- إصلاح مشكلة تحميل ملف الإعدادات بسبب النقطة الزائدة في `CONFIG_KEY`.

#### الميزات الجديدة

##### 1. تحويل تعريفات الحقول إلى تنسيق `FormFieldsManager`

تم تغيير بنية الحقول في ملف `config/nano3/kyc/document_types.php` من:

```php
// القديم (مصفوفة ترابطية)
'fields' => [
    'document_number' => [
        'label' => 'رقم الوثيقة',
        'type' => 'text',
        'required' => true,
        'validation' => ['max_length' => 50],
    ],
]
```

إلى التنسيق الجديد المتسلسل المدعوم من `FormFieldsManager`:

```php
'fields' => [
    [
        'id' => 'document_number',
        'label' => ['ar' => 'رقم الوثيقة', 'en' => 'Document Number'],
        'type' => 'text',
        'required' => true,
        'validation' => [
            ['rule' => 'required'],
            ['rule' => 'max', 'value' => 50],
        ],
    ],
]
```

هذا التغيير يجعل تعريفات الحقول متوافقة مع أدوات إدارة النماذج الموحدة في المنصة.

##### 2. دوال تحويل ذكية للتوافق مع الإصدارات السابقة (Legacy Support)

لضمان عدم تأثر الأنظمة التي ما زالت تعتمد على التنسيق القديم، تمت إضافة دوال تحويل تلقائية في `DocumentType`:

| الدالة | الوصف |
| :--- | :--- |
| `convertLegacyFieldsToNewFormat($legacyFields)` | تحول مصفوفة الحقول الترابطية القديمة إلى المصفوفة المتسلسلة الجديدة. |
| `normalizeMultilingualValue($value)` | تحول أي نص إلى مصفوفة متعددة اللغات (`['ar' => ..., 'en' => ...]`). |
| `convertOptionsToDataSource($optionType)` | تحول الخيارات النصية القديمة (مثل `'countries'`) إلى تعريف `data_source` متوافق مع `FormFieldsManager`. |
| `convertLegacyValidation($legacyValidation)` | تحول قواعد التحقق من التنسيق الترابطي إلى تنسيق `FormFieldsManager` (مصفوفة من `['rule' => ..., 'value' => ...]`). |

بفضل هذه الدوال، يمكن لـ `DocumentType` قراءة كلا التنسيقين (القديم والجديد) وإرجاع الحقول دائمًا بالتنسيق الموحد.

##### 3. دوال مُحسَّنة وجديدة في `DocumentType`

| الدالة | الوصف |
| :--- | :--- |
| `getFieldsSchema($type)` | **مُحسَّنة**: تكتشف تنسيق الحقول تلقائيًا (قديم/جديد) وتعيدها دائمًا بالتنسيق المتسلسل بعد التطبيع عبر `FormFieldsManager`. |
| `getFieldDefinition($type, $field)` | **مُحسَّنة**: تبحث في المصفوفة المتسلسلة عن الحقل باستخدام المفتاح `id`. |
| `getValidationRules($type)` | **مُحسَّنة**: تستخدم `FormFieldsManager` لتوليد قواعد التحقق بدلاً من المنطق المخصص. |
| `getDefaultFields()` | **مُعاد كتابتها**: ترجع الحقول الافتراضية بالتنسيق المتسلسل الجديد. |
| `buildFieldsForType($customFields, $overrideDefaults)` | **مُعاد كتابتها**: تتعامل مع المصفوفات المتسلسلة وتطبق التجاوزات بناءً على `id`. |
| `mapInputToDocumentData($type, $input)` | **مُحسَّنة**: تستخدم `id` من تعريف الحقل لاستخراج البيانات. |
| `mapDocumentToOutput($type, $document)` | **مُحسَّنة**: تعيد بناء المخرجات باستخدام `id` الحقول. |
| `getFieldLabel($type, $field, $locale)` | **مُحسَّنة**: تستخرج النص المطلوب من مصفوفة `label` متعددة اللغات. |
| `getFormFieldsManager()` | **جديدة**: تنشئ نسخة من `FormFieldsManager` مع اللغات المدعومة (`ar`, `en`). |

##### 4. تحسين نقطة نهاية `GET /document-fields/{type}`

تم تحديث الدالة `Documents::getDocumentFields` لتقوم بما يلي:

- جلب الحقول المطبعة من `DocumentType::getFieldsSchema`.
- إثراء الحقول بالخيارات المحلولة (`resolved_options`) باستخدام `FormFieldsManager::enrichWithOptions`.
- إرجاع البيانات بتنسيق موحد يدعمه `FormFieldsManager`.

هذا يسمح لتطبيقات الواجهة الأمامية بالحصول على قوائم منسدلة جاهزة (مثل قائمة الدول والجنسيات) مباشرة ضمن تعريف الحقل.

##### 5. إصلاح تحميل ملف الإعدادات

تم تصحيح دالة `loadConfig` لإزالة النقطة الزائدة من `CONFIG_KEY` (`nano3.kyc::document_types.` ← `nano3.kyc::document_types`) مما يضمن تحميل ملف `document_types.php` بشكل صحيح.

---

### أمثلة عملية

#### 1. الحصول على حقول جواز السفر مع الخيارات المحلولة

**الطلب:**
```http
GET /api/v1/kyc/document-fields/passport HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مقتطف):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "type": "passport",
        "type_name": "جواز سفر",
        "fields": [
            {
                "id": "document_front",
                "type": "fileupload",
                "label": { "ar": "وجه الوثيقة", "en": "Document Front" },
                "required": true,
                "validation": [
                    { "rule": "required" },
                    { "rule": "mimes", "value": "jpeg,jpg,png,pdf" }
                ]
            },
            {
                "id": "country_of_issue",
                "type": "select",
                "label": { "ar": "بلد الإصدار", "en": "Country of Issue" },
                "data_source": {
                    "type": "api",
                    "endpoint": "api/v1/location/countries"
                },
                "resolved_options": [
                    { "id": "YE", "name": "اليمن" },
                    { "id": "SA", "name": "السعودية" },
                    ...
                ]
            }
        ]
    }
}
```

#### 2. استخدام التنسيق القديم (يُحول تلقائيًا)

إذا كان لديك تعريف حقل قديم (مصفوفة ترابطية) في إصدار سابق، فإن `DocumentType` سيحوله تلقائيًا إلى التنسيق الجديد دون أي تدخل من المطور.

```php
// تعريف قديم
$oldFields = [
    'document_number' => ['type' => 'text', 'required' => true]
];

// يستخدم داخليًا convertLegacyFieldsToNewFormat()
$newFields = DocumentType::getFieldsSchema('some_type');
```

---

### ملخص الإصدارات (1.2.6 – 1.2.7)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.2.6 | نقاط نهاية جديدة للتصنيفات والأنواع، دوال `DocumentType` متقدمة، تحسين معالجة الأخطاء، توحيد مخرجات API. |
| 1.2.7 | إعادة هيكلة حقول الوثائق لتتوافق مع `FormFieldsManager`، دوال تحويل تلقائية للتوافق مع الإصدارات السابقة، تحسين قواعد التحقق، إثراء حقول API بالخيارات المحلولة، إصلاح تحميل الإعدادات. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `classes/DocumentType.php` بالنسخة الجديدة.
   - استبدال ملف `apicontrollers/Documents.php` بالنسخة الجديدة.
   - تحديث ملف `config/nano3/kyc/document_types.php` إلى التنسيق الجديد (اختياري لكن موصى به للاستفادة الكاملة من الميزات).

2. **تشغيل هجرات قاعدة البيانات** (لا توجد تغييرات في هذا الإصدار).

3. **اختبار التوافق**:
   - تأكد من أن نقاط النهاية الحالية (`GET /document-fields/{type}`) تعيد البيانات بالتنسيق الجديد.
   - تحقق من أن عمليات إنشاء وتحديث الوثائق ما زالت تعمل بشكل صحيح (حيث تستخدم `mapInputToDocumentData`).

4. **مسح الكاش** (اختياري):
   - بعد تحديث ملف الإعدادات، قد تحتاج إلى مسح كاش التطبيق: `php artisan cache:clear`

---

### الخاتمة

مع الإصدار 1.2.7، نكون قد وحَّدنا آلية إدارة حقول KYC مع نظام `FormFieldsManager` المعتمد في باقي أجزاء المنصة. هذا يفتح آفاقًا جديدة مثل بناء نماذج ديناميكية في لوحة التحكم، وتوليد قواعد تحقق موحدة، واستخدام مصادر بيانات خارجية بسهولة. كما أن دوال التحويل التلقائية تضمن سلاسة الترقية دون أي تأثير سلبي على الأنظمة القائمة.

نتطلع إلى ملاحظاتكم ومقترحاتكم لمواصلة تطوير هذه الإضافة الحيوية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

## 2026-04-28 - 2026-04-29

**تحديث إضافة Yepayment – طريقة دفع جديدة QasemiPay**

**نوع التحديث:** إضافة ميزة رئيسية (Major Feature)

---

### 1. نظرة عامة

يضيف هذا التحديث طريقة دفع جديدة بالكامل باسم **QasemiPay**، وهي بوابة دفع متكاملة مع **مصرف قاسمي الإسلامي** عبر خدمة **Purchase Code Service API**.  
تم بناء البوابة وفق أعلى معايير الأمان والمرونة، وتدعم خطوتين لإتمام الدفع (إنشاء المعاملة ثم تأكيدها)، بالإضافة إلى التحقق الفوري من حالة المعاملة.  
كما تم توفير واجهة اختبار تفاعلية وأدوات تطوير متكاملة، لضمان سهولة التكامل والاختبار.

---

### 2. الميزات الجديدة

#### 2.1. إضافة بوابة دفع QasemiPay

- **المعرف:** `qasemi`
- **الاسم المعروض:** Qasemi Islamic Bank
- **آلية المصادقة:** OAuth 2.0 (grant_type=password)  
- **عمليات الدعم:**
  - إنشاء معاملة شراء (Create Purchase Transaction)
  - تأكيد المعاملة باستخدام `concurrencyStamp`
  - الاستعلام عن حالة المعاملة عبر `refId`
- **حقول الإدخال المطلوبة من العميل:**
  - `purchase_code` – رمز الشراء
  - `mobile_number` – رقم الجوال
  - `amount` – المبلغ
  - `currency` – العملة (YER/SAR)
  - `zone` – المنطقة (اختياري)
  - `agent` – الوكيل (اختياري)
  - `branch` – الفرع (اختياري)
  - `notes` – ملاحظات (اختياري)

#### 2.2. إعدادات متقدمة في لوحة التحكم

- **رابط API الأساسي** (وضع الإنتاج والاختبار)
- **بيانات المصادقة:** Client ID, Client Secret, Username, Password
- **النطاق (Scope)** – قيمة مرنة، افتراضياً `purchase_code_service`
- **الإعدادات الافتراضية:** العملة، المنطقة، الوكيل، الفرع
- **تخزين مشفر للحساسات:** `client_secret` و `password`

#### 2.3. واجهة اختبار متكاملة (Test UI)

- مسار الوصول: `/api/v1/yepayment/qasemipay/test-ui`
- **الميزات:**
  - اختبار المصادقة (Auth)
  - إنشاء معاملة تجريبية مع جميع الحقول
  - تأكيد المعاملة باستخدام `concurrencyStamp`
  - الاستعلام عن حالة المعاملة
  - اختبار شامل لجميع الخطوات تلقائياً
  - اختبار تكراري (حتى 10 مرات) لقياس الاستقرارية
  - إحصائيات مباشرة (عدد الطلبات، نسبة النجاح، آخر السجلات)
  - سجل محلي للاختبارات مع إمكانية التصدير والمسح

#### 2.4. نقاط نهاية API جديدة للاختبار والتكامل الخارجي

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/qasemipay/test-auth` | POST | التحقق من صحة بيانات المصادقة |
| `/qasemipay/test-create-payment` | POST | إنشاء معاملة تجريبية |
| `/qasemipay/test-confirm-payment` | POST | تأكيد معاملة |
| `/qasemipay/test-check-status` | GET | الاستعلام عن حالة معاملة |
| `/qasemipay/test-full-payment` | POST | اختبار شامل (إنشاء + تأكيد + استعلام) |
| `/qasemipay/stats` | GET | إحصائيات استخدام البوابة |

---

### 3. التغييرات التقنية (Technical Changes)

#### 3.1. الكلاسات والملفات المضافة

```
plugins/nano/yepayment/
├── paymenttypes/
│   └── qasemipay/
│       ├── QasemiPay.php                 ## الكلاس الرئيسي لبوابة الدفع
│       ├── _info.htm                     ## معلومات العرض في الإعدادات
│       ├── _test_info.htm                ## أدوات الاختبار السريع ضمن الإعدادات
│       └── qasemi-ui.htm                 ## واجهة الاختبار المتكاملة (HTML/JS)
├── views/
│   └── qasemi-ui.htm                     (نسخة احتياطية – اختيارية)
└── lang/
    ├── ar/lang.php                       ## إضافة مفاتيح الترجمة العربية
    └── en/lang.php                       ## إضافة مفاتيح الترجمة الإنجليزية
```

#### 3.2. التعديلات على الملفات القائمة

- **`Plugin.php`**  
  تم إضافة تسجيل `QasemiPay` في دالة `registerPaymentProviders()` ضمن القسم الخاص بـ `allow_yemen_payment` (أو `allow_oman_payment` حسب الرغبة).

- **`routes.php`**  
  تم إضافة مجموعة كاملة من نقاط النهاية الخاصة بـ `qasemi` تحت مجموعة `yepayment`، مع تطبيق نفس ميدلوير `cors` و `api` و `web` المستخدمة في البوابات الأخرى.

#### 3.3. التبعيات (Dependencies)

- **مطلوب:** إصدار 2.0+ من `Nano.MicroCart`
- **مطلوب:** `Nano.Helpers` (خاص بـ `HttpHelper`)
- **مطلوب:** `Illuminate\Support\Str` (لإنشاء UUID)
- **موصى به:** `nano.yepayment::lang` ملفات اللغة محدثة

---

### 4. دليل الترقية (Upgrade Guide)

#### 4.1. للمطورين (رفع الحزمة)

```bash
php artisan plugin:refresh Nano.Yepayment
```

أو إعادة تثبيت الإضافة من السوق الداخلي.

#### 4.2. للتجار (إعداد البوابة)

1. انتقل إلى **لوحة التحكم ← إعدادات المتجر ← طرق الدفع**.
2. أضف طريقة دفع جديدة واختر **Qasemi Islamic Bank**.
3. أدخل بيانات API المقدمة من مصرف قاسمي:
   - Client ID / Client Secret
   - Username / Password
   - رابط API (يفضل استخدام رابط الاختبار أولاً)
4. احفظ الإعدادات.

#### 4.3. اختبار التكامل

- استخدم واجهة الاختبار: `/api/v1/yepayment/qasemipay/test-ui`
- تأكد من نجاح المصادقة ثم قم بإنشاء معاملة تجريبية.
- بعد التأكد من عمل كل الخطوات، غيّر إلى رابط الإنتاج.

---

### 5. التوافق (Compatibility)

- **NanoSoft App:** v2.x
- **PHP:** 8.0 / 8.1 / 8.2
- **إضافات نانوسوفت:**  
  - `Nano.MicroCart` (>=2.0)  
  - `Nano.Helpers` (>=1.2)  
  - `Nano.Orders` (>=1.5)
- **قواعد البيانات:** يدعم MySQL، PostgreSQL، SQLite (من خلال Eloquent)

---

### 6. تحسينات أخرى في هذا الإصدار

- **تحديث ملفات اللغة** – إضافة ترجمات كاملة لبوابة QasemiPay بالعربية والإنجليزية.
- **تحسين أمان تخزين التوكن** – إمكانية إضافة cache للتوكن مستقبلاً (لم يتم تفعيلها افتراضياً، لكن البنية مهيأة).
- **توثيق كامل** – تم إضافة ملف توثيق شامل لبوابة QasemiPay (`QasemiPay-Documentation.md`).

---

### 7. ملاحظات للمطورين (Developer Notes)

- لا تعتمد بوابة QasemiPay على إعادة توجيه المستخدم إلى صفحة خارجية، لذلك تم إهمال `success_url` و `cancel_url` (يمكن إضافتهما لاحقاً إذا دعت الحاجة).
- في حال رغبت في تفعيل تخزين التوكن مؤقتاً، أضف طبقة Cache داخل `getAuthToken()` مع مدة صلاحية 3600 ثانية.
- جميع طلبات API تستخدم `HttpHelper` الموحد، مما يسهل تتبع الأخطاء وتوسيع البوابة.

---

### 8. إصلاحات الأخطاء (Bug Fixes)

لا يوجد – هذا الإصدار مخصص لإضافة ميزة جديدة فقط.

---

### 9. روابط ذات صلة

- [QasemiPay API Documentation ](./docs/QasemiPay/Docs-QasemiPay-ar.md)
- [وثيقة API الخاصة بخدمة رمز الشراء – مصرف قاسمي (PDF)](Purchase%20Code%20Service%20API%20Documentation%20(Draft).pdf)
- [دليل تطوير بوابات الدفع – نانوسوفت](https://docs.nano2soft.com/payment-gateways)
- [قناة الدعم الفني](https://nano2soft.com)

---

**تم إعداد هذا التحديث بواسطة:**  
فريق تطوير نانوسوفت – قسم المدفوعات الإلكترونية -مصرف القاسمي 
**المراجع:** Dheia Ali, Nano2Soft

---

## 2026-04-30

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.8**

### ملخص التحديثات

يأتي الإصدار 1.2.8 كإصلاح شامل لآلية دمج الحقول وتوليد قواعد التحقق في كلاس `DocumentType`. رغم النجاح الذي حققه الإصدار 1.2.7 في توحيد بنية الحقول مع `FormFieldsManager`، إلا أن الاختبارات العملية كشفت عن مشكلات في طريقة دمج الحقول الافتراضية مع التجاوزات (overrides) ومع إعدادات ملف `document_types.php`. أدت هذه المشكلات إلى ظهور حقول غير مرغوب فيها في بعض أنواع الوثائق (مثل ظهور حقول الشركة في جواز السفر)، وتكرار بعض الحقول، ووجود قواعد تحقق خاطئة أو مشوهة.

يركز هذا الإصدار على **إصلاح جذري لخوارزميات الدمج** لضمان:

- حذف الحقول غير المرغوب فيها نهائياً عند تحديدها بقيمة `null` في التجاوزات.
- دمج قواعد التحقق (`validation`) بذكاء مع إمكانية استبدال القواعد المتكررة بدلاً من تراكمها.
- استبدال الحقول بالكامل عند تعريفها في ملف الإعدادات (`document_types.php`) بدلاً من دمجها مع الحقول الافتراضية.
- توليد قواعد تحقق (`rules`) صحيحة ومتوافقة تماماً مع صيغة Laravel، جاهزة للاستخدام المباشر في `Validator::make`.

كل ذلك مع الحفاظ على التوافق التام مع الإصدارات السابقة وعدم كسر أي واجهات برمجية (API).

---

### الإصدار 1.2.8 – إصلاح دمج الحقول وتوليد قواعد التحقق

#### أهداف الإصدار

- **إصلاح دالة `mergeFieldsWithOverrides`** لدعم حذف الحقول (`null`) ودمج قواعد التحقق بشكل صحيح.
- **إضافة دالة `mergeValidation`** لاستبدال القواعد المكررة بدلاً من تراكمها، مما يحل مشكلة القيم المشوهة في `validation`.
- **إصلاح دالة `loadConfig`** لجعل ملف الإعدادات هو المصدر الوحيد للحقول عند تعريفها، مما يمنع تسرب الحقول الافتراضية إلى الأنواع المخصصة.
- **إزالة التكرارات** في الحقول (مثل `religion` و `passport_type`).
- **تصحيح قواعد التحقق** في جميع أنواع الوثائق لتعكس القيم الصحيحة (مثل `max:50` بدلاً من `max:100` أو قيم غير مقصودة).

#### التحسينات والإصلاحات

##### 1. إعادة كتابة دالة `mergeFieldsWithOverrides`

تم استبدال التنفيذ السابق الذي كان يعتمد على `array_merge` و `unset` (الذي لم يكن يحذف الحقول بشكل صحيح في بعض الحالات) بتنفيذ جديد يعتمد على **الفهرسة بالمفتاح `id`**:

```php
protected static function mergeFieldsWithOverrides(array $baseFields, array $extraFields = [], array $overrides = []): array
{
    // تحويل baseFields إلى associative بمفتاح id
    $fieldsAssoc = [];
    foreach ($baseFields as $field) {
        $id = $field['id'] ?? null;
        if ($id) {
            $fieldsAssoc[$id] = $field;
        }
    }

    // extraFields تستبدل الحقل بالكامل إذا كان id موجوداً
    foreach ($extraFields as $extraField) {
        $id = $extraField['id'] ?? null;
        if ($id) {
            $fieldsAssoc[$id] = $extraField;
        } else {
            $fieldsAssoc[] = $extraField;
        }
    }

    // overrides تطبق التعديلات أو الحذف (null)
    foreach ($overrides as $fieldId => $override) {
        if ($override === null) {
            unset($fieldsAssoc[$fieldId]);   // حذف نهائي
            continue;
        }

        if (isset($fieldsAssoc[$fieldId])) {
            $base = $fieldsAssoc[$fieldId];
            foreach ($override as $key => $value) {
                if ($key === 'validation') {
                    $base['validation'] = self::mergeValidation($base['validation'] ?? [], $value);
                } else {
                    $base[$key] = $value;
                }
            }
            $fieldsAssoc[$fieldId] = $base;
        } else {
            $fieldsAssoc[$fieldId] = $override;
        }
    }

    $fields = array_values($fieldsAssoc);
    usort($fields, fn($a, $b) => ($a['sort_order'] ?? 999) <=> ($b['sort_order'] ?? 999));
    return $fields;
}
```

**النتائج**:
- حذف الحقول المحددة بقيمة `null` في `overrides` يعمل الآن بشكل موثوق.
- دمج الخصائص يتم بشكل دقيق مع معالجة خاصة لـ `validation`.

##### 2. دالة `mergeValidation` لدمج قواعد التحقق بذكاء

تمت إضافة دالة مساعدة جديدة تقوم بفهرسة القواعد الحالية باستخدام `rule` كمفتاح، ثم تستبدل القاعدة إذا تكرر اسمها، مما يمنع تراكم القواعد المكررة وظهور قيم مشوهة (مثل `rule: "required", value: 100`).

```php
protected static function mergeValidation(array $baseValidation, array $newValidation): array
{
    $indexed = [];
    foreach ($baseValidation as $rule) {
        if (isset($rule['rule'])) {
            $indexed[$rule['rule']] = $rule;
        } else {
            $indexed[] = $rule;
        }
    }

    foreach ($newValidation as $rule) {
        if (isset($rule['rule'])) {
            $indexed[$rule['rule']] = $rule;   // استبدال
        } else {
            $indexed[] = $rule;
        }
    }

    return array_values($indexed);
}
```

##### 3. إصلاح دالة `loadConfig` – استبدال الحقول عند تعريفها في ملف الإعدادات

في السابق، كان ملف `document_types.php` يدمج حقوله مع الحقول الافتراضية، مما يؤدي إلى ظهور حقول غير مرغوب فيها. الآن، إذا تم تعريف المفتاح `fields` في ملف الإعدادات، فإنه **يستبدل** الحقول الافتراضية بالكامل:

```php
protected static function loadConfig(): array
{
    // ...
    foreach ($fileConfig as $type => $settings) {
        if (isset($merged[$type])) {
            // إذا كان الملف يحتوي على 'fields'، نستبدل الحقول بالكامل
            if (isset($settings['fields'])) {
                $merged[$type]['fields'] = $settings['fields'];
                unset($settings['fields']);
            }
            $merged[$type] = array_replace_recursive($merged[$type], $settings);
        } else {
            $merged[$type] = $settings;
        }
    }
    // ...
}
```

##### 4. إزالة تكرار الحقول

تم حذف الحقل `religion` من `getDefaultFields` لأنه يُضاف يدويًا عند الحاجة عبر `extraFields` أو `overrides`. كما تم ضبط `overrides` لمنع ظهور حقول مكررة مثل `passport_type`.

##### 5. تصحيح قواعد التحقق في جميع الأنواع

بفضل `mergeValidation` الجديدة، أصبحت قواعد التحقق تعكس القيم الصحيحة المحددة في `overrides` أو ملف الإعدادات. على سبيل المثال:

- `document_number` → `['required', 'max:50']` (بدلاً من القيم المشوهة السابقة).
- `issue_date` → `['required', 'date', 'before_or_equal:today']` (بدون `value` إضافية).
- `country_of_issue` → `['required']` (بدون قواعد `date` الخاطئة).

---

### أمثلة عملية

#### 1. حقل `signature_specimen` (نموذج التوقيع) – قبل وبعد

**قبل الإصلاح (الإصدار 1.2.7)**:
كانت استجابة `GET /document-types-list/signature_specimen` تُرجع **جميع الحقول الافتراضية** (مثل `issue_date`، `address_line1`، `company_name`، إلخ) بالإضافة إلى حقل `signature_image`.

**بعد الإصلاح (الإصدار 1.2.8)**:
الاستجابة تقتصر على الحقول المعرفة في `document_types.php` فقط:

```json
{
    "fields": [
        {
            "id": "signature_image",
            "type": "fileupload",
            "label": { "ar": "نموذج التوقيع", "en": "Signature Specimen" },
            "required": true,
            "validation": [
                { "rule": "required" },
                { "rule": "mimes", "value": "jpg,jpeg,png" },
                { "rule": "max", "value": 2048 }
            ]
        }
    ],
    "rules": {
        "signature_image": ["required", "file", "mimes:jpg,jpeg,png", "max:2048"]
    }
}
```

#### 2. حقل `passport` – قواعد تحقق نظيفة

بعد الإصلاحات، أصبحت قواعد التحقق لحقل `passport` كما يلي (مقتطف):

```json
"rules": {
    "document_front": ["required", "file", "mimes:jpg,jpeg,png,pdf", "max:10240"],
    "document_number": ["required", "max:50"],
    "full_name": ["required", "max:100"],
    "issue_date": ["required", "date", "before_or_equal:today"],
    "expiry_date": ["required", "date", "after_or_equal:today"],
    "country_of_issue": ["required"],
    "nationality": ["required"]
}
```

لاحظ اختفاء القواعد الخاطئة مثل `date:today` المكررة أو `before_or_equal:today - 16 years` على `country_of_issue`.

---

### ملخص الإصدارات (1.2.7 – 1.2.8)

| الإصدار | أبرز الميزات والإصلاحات |
| :--- | :--- |
| 1.2.7 | توافق كامل مع `FormFieldsManager`، تحويل تلقائي للحقول القديمة، إثراء API بالخيارات المحلولة. |
| 1.2.8 | **إصلاح جذري لدمج الحقول وقواعد التحقق**، حذف الحقول غير المرغوب فيها (`null`)، دمج قواعد التحقق بذكاء، منع تسرب الحقول الافتراضية، إزالة التكرارات، تصحيح جميع قواعد التحقق. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `classes/DocumentType.php` بالنسخة الجديدة (يتضمن الدوال المحسنة `mergeFieldsWithOverrides` و `mergeValidation` و `loadConfig`).
   - التأكد من تحديث `getDefaultFields` بإزالة حقل `religion` إن وُجد.

2. **اختبار نقاط النهاية**:
   - جرب `GET /api/v1/kyc/document-types-list/{type}?include_fields=1&include_rules=1` وتأكد من أن الحقول المعروضة تطابق المتوقع.
   - تحقق من عمليات `POST /documents` و `PUT /documents/{id}` للتأكد من أن قواعد التحقق تعمل بشكل صحيح.

3. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### الخاتمة

الإصدار 1.2.8 هو إصدار استقرار مهم يعالج مشكلات أساسية في آلية دمج الحقول التي ظهرت بعد الانتقال إلى `FormFieldsManager`. بفضل هذه الإصلاحات، أصبح بإمكان المطورين الاعتماد على تعريفات الحقول في ملف الإعدادات بثقة تامة، وستعمل قواعد التحقق في `Manager` كما هو متوقع دون أخطاء.

نشكركم على ملاحظاتكم القيمة التي ساعدت في اكتشاف هذه المشكلات ومعالجتها. نلتزم بمواصلة تحسين إضافة KYC لتكون قوية ومرنة وسهلة الاستخدام.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

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

- [JaibPay API Documentation ](./docs/JaibPay/Docs-JaibPay-ar.md)
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

