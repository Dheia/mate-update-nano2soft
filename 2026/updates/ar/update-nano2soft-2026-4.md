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

