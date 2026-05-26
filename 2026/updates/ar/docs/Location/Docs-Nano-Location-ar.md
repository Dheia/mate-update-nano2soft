# توثيق إضافة `Nano.Location`

**الإصدار:** 1.0.16  
**المؤلف:** Dheia Ali  
**الرخصة:** خاصة بـ Nano2Soft  
**آخر تحديث:** 2025-04-30

---

## المحتويات

1. [مقدمة](#مقدمة)
2. [المتطلبات](#المتطلبات)
3. [التثبيت والترقية](#التثبيت-والترقية)
4. [الميزات الرئيسية](#الميزات-الرئيسية)
5. [هيكل قاعدة البيانات](#هيكل-قاعدة-البيانات)
6. [النماذج (Models)](#النماذج-models)
7. [السلوكيات (Behaviors)](#السلوكيات-behaviors)
8. [السمات (Traits)](#السمات-traits)
9. [الواجهة الخلفية (Backend)](#الواجهة-الخلفية-backend)
10. [إعدادات الإضافة](#إعدادات-الإضافة)
11. [الصلاحيات](#الصلاحيات)
12. [المكونات (Components)](#المكونات-components)
13. [دوال المساعدة (Helpers)](#دوال-المساعدة-helpers)
14. [الأحداث (Events)](#الأحداث-events)
15. [الترجمة (Internationalization)](#الترجمة-internationalization)
16. [أمثلة عملية](#أمثلة-عملية)
17. [استكشاف الأخطاء وإصلاحها](#استكشاف-الأخطاء-وإصلاحها)
18. [المراجع](#المراجع)

---

## مقدمة

إضافة `Nano.Location` هي إضافة متقدمة لأنظمة October CMS تهدف إلى توفير بنية جغرافية متكاملة تشمل **الدول (Countries)**، **المدن/الولايات (States)**، **المناطق/المديريات (Directorates)**، بالإضافة إلى **العناوين (Addresses)**، **الجنسيات (Nationalities)**، **اللغات (Languages)**، **الأديان (Religions)**، وتتبع المسارات (Traceroutes). تم تصميم الإضافة لتكون مرنة وقابلة للتوسع، مع دعم كامل للواجهة الخلفية (Backend) وواجهة برمجة التطبيقات (API) من خلال إضافة `Nano.LocationApi` المصاحبة.

تعتمد الإضافة على `RainLab.Location` كمكوّن أساسي، ولكنها تمددها بإضافة:
- **المديريات (Directorates)**: مستوى ثالث من التقسيم الجغرافي (مثل الأحياء، العزل، المديريات).
- **العناوين (Addresses)**: نموذج مرن لإدارة العناوين لعدة أنواع من الكائنات (متعدد الأشكال).
- **الجنسيات (Nationalities)**: قائمة بالجنسيات مع أكوادها وعلمها.
- **تتبع المسارات (Traceroutes)**: تسجيل حركة المستخدمين الجغرافية.
- **دعم الوسائط المتعددة**: إضافة الصور، الفيديوهات، الملفات الصوتية، والمستندات للكيانات الجغرافية (أُضيف في الإصدار 1.0.16).
- **دعم تعدد اللغات**: عبر `RainLab.Translate`.

---

## المتطلبات

- **October CMS** الإصدار 3.x أو 2.x (مع بعض القيود).
- **PHP** >= 8.0.
- **الإضافات المطلوبة** (معلنة في `$require`):
  - `RainLab.Location`
  - `RainLab.User`
- **الإضافات الموصى بها**:
  - `RainLab.Translate` (لترجمة أسماء الدول والمدن والمحافظات).
  - `Nano.LocationApi` (لتوفير واجهة API متكاملة).
  - `Nano.ActivityLog` (لتسجيل النشاطات على الكيانات الجغرافية).

---

## التثبيت والترقية

### تثبيت جديد

```bash
php artisan plugin:install Nano.Location
php artisan nano:up
```

### الترقية من إصدار سابق

```bash
composer update nano/location
php artisan nano:up
php artisan cache:clear
```

### نشر ملف الإعدادات (اختياري)

```bash
php artisan config:publish Nano.Location
```

سيتم إنشاء ملف الإعدادات في `config/nano/location/config.php`.

---

## الميزات الرئيسية

| الميزة | الوصف |
| :--- | :--- |
| **الدول (Countries)** | توسيع نموذج `RainLab\Location\Models\Country` بإضافة حقول: `wikiDataId`, `latitude`, `longitude`, `sort_order`, وعلاقات وسائط متعددة. |
| **المدن/الولايات (States)** | توسيع نموذج `RainLab\Location\Models\State` بإضافة `wikiDataId`, `latitude`, `longitude`, `sort_order`، وعلاقة `hasMany` مع `Directorate`. |
| **المديريات (Directorates)** | نموذج جديد يمثل المستوى الثالث من التقسيم الإداري، مع ترجمة وتفعيل وفرز. |
| **العناوين (Addresses)** | نموذج `Addres` (ملاحظ: الاسم به خطأ إملائي متعمد للتوافق) يدعم العلاقات متعددة الأشكال مع أي نموذج آخر، مع حقول جغرافية متكاملة وإمكانية تعيين عنوان افتراضي. |
| **الجنسيات (Nationalities)** | نموذج `Nationality` يحتوي على: `name`, `code`, `nationality_code`, `country_code`, `iso_code`, `phone_code`, `region`, `subregion`, `flag_emoji`, مع دعم الصورة الرئيسية. |
| **تتبع المسارات (Traceroutes)** | نموذج `Traceroute` يسجل زيارات المستخدمين مع بيانات الموقع الجغرافي (IP، البلد، المدينة، الإحداثيات). |
| **اللغات والأديان (Languages & Religions)** | دوال مساعدة `LanguagesHelper` و `ReligionsHelper` لتوفير قوائم جاهزة للغات والأديان المعروفة. |
| **الوسائط المتعددة (Media)** | دعم `attachOne` و `attachMany` للصور، الفيديوهات، الصوت، والملفات في الدول والمدن والمديريات (من الإصدار 1.0.16). |
| **الواجهة الخلفية** | قوائم ونماذج متكاملة لإدارة جميع الكيانات، مع دعم الأعمدة المخصصة، الفلاتر، والصلاحيات. |
| **صلاحيات متقدمة** | صلاحيات منفصلة للعناوين (access, add, edit, delete, access_all). |
| **دعم تعدد اللغات** | ترجمة أسماء الدول، المدن، المديريات، والجنسيات. |
| **تسجيل النشاطات** | تكامل مع `Nano.ActivityLog` لتتبع التغييرات على الكيانات الجغرافية. |

---

## هيكل قاعدة البيانات

### الجداول التي تنشئها الإضافة

| الجدول | الوصف |
| :--- | :--- |
| `nano_location_directorates` | المديريات (مرتبطة بـ `country_id` و `state_id`) |
| `nano_location_address` | العناوين (مع أعمدة `addressable_type`, `addressable_id`) |
| `nano_location_traceroutes` | تتبع المسارات |
| `nano_location_nationalities` | الجنسيات |

### الجداول الممتدة من إضافات أخرى

| الجدول الأصلي | الحقول المضافة |
| :--- | :--- |
| `rainlab_location_countries` | `wikiDataId`, `latitude`, `longitude`, `sort_order` |
| `rainlab_location_states` | `wikiDataId`, `latitude`, `longitude`, `sort_order` |

### مخطط مبسط للعلاقات

```
countries (RainLab)
    ├── states (hasMany)
    │       └── directorates (hasMany)
    └── nationalities (hasMany?)

address (Nano)
    ├── addressable (morphTo)
    ├── country (belongsTo)
    ├── state (belongsTo)
    └── directorate (belongsTo)

traceroute
    ├── user (morphTo)
    └── geolocation data
```

---

## النماذج (Models)

### 1. `Country` (ممتد من `RainLab\Location\Models\Country`)

تمت إضافة الخصائص والعلاقات التالية:

```php
// حقول قابلة للتعبئة
$fillable = ['wikiDataId', 'latitude', 'longitude', 'sort_order'];

// علاقات الوسائط (من الإصدار 1.0.16)
public $attachOne = ['image', 'book_intro'];
public $attachMany = ['images', 'videos', 'audios', 'files'];

// حدث قبل الحفظ لتعيين sort_order تلقائياً
public function beforeSave()
{
    if ($this->id && !$this->sort_order) {
        $this->sort_order = $this->id;
    }
}
```

### 2. `State` (ممتد من `RainLab\Location\Models\State`)

```php
// حقول إضافية
$fillable = ['wikiDataId', 'latitude', 'longitude', 'sort_order'];

// علاقة بالمديريات
public $hasMany = ['directorates' => Directorate::class];

// علاقات الوسائط
public $attachOne = ['image', 'book_intro'];
public $attachMany = ['images', 'videos', 'audios', 'files'];

// حدث beforeSave لـ sort_order
```

### 3. `Directorate`

نموذج المديريات الكامل:

```php
class Directorate extends Model
{
    use \October\Rain\Database\Traits\Validation;
    use \Nano\Location\Traits\HasTimestamps;
    use \October\Rain\Database\Traits\Sortable;
    use \RainLab\Translate\Behaviors\TranslatableModel;
    
    public $table = 'nano_location_directorates';
    public $translatable = ['name'];
    public $fillable = ['name', 'code', 'is_enabled', 'sort_order'];
    
    public $belongsTo = [
        'country' => [Country::class],
        'state'   => [State::class],
    ];
    
    // دوال مساعدة
    public static function getNameList($countryId);
    public static function formSelect($name, $countryId = null, $selectedValue = null);
    public static function getDefault();
}
```

### 4. `Addres` (ملاحظ: اسم النموذج "Addres" وليس "Address" للتوافق القديم)

```php
class Addres extends Model
{
    use \October\Rain\Database\Traits\Validation;
    use \Nano\Location\Traits\HasTimestamps;
    use \October\Rain\Database\Traits\Sortable;
    use \RainLab\Translate\Behaviors\TranslatableModel;
    
    public $table = 'nano_location_address';
    public $translatable = ['address_1', 'address_2', 'street_name', 'city'];
    
    // العلاقات متعددة الأشكال
    public $morphTo = ['addressable'];
    
    public $belongsTo = [
        'country'    => [Country::class],
        'state'      => [State::class],
        'directorate'=> [Directorate::class],
    ];
    
    // حقول JSON
    protected $jsonable = ['config_data', 'other_data'];
    
    // دوال مساعدة
    public static function getDefaultAddress($addressable);
    public function scopeIsDefault($query);
}
```

### 5. `Nationality`

```php
class Nationality extends Model
{
    use \October\Rain\Database\Traits\Validation;
    use \Nano\Location\Traits\HasTimestamps;
    use \October\Rain\Database\Traits\Sortable;
    
    public $table = 'nano_location_nationalities';
    public $fillable = [
        'name', 'code', 'nationality_code', 'country_code', 'iso_code',
        'phone_code', 'region', 'subregion', 'flag_emoji', 'description',
        'is_active', 'is_default', 'sort_order', 'metadata'
    ];
    
    public $attachOne = ['image' => \System\Models\File::class];
}
```

### 6. `Traceroute`

```php
class Traceroute extends Model
{
    use \October\Rain\Database\Traits\Validation;
    
    public $table = 'nano_location_traceroutes';
    public $fillable = [
        'ip', 'country_code', 'country_name', 'region_code', 'region_name',
        'city', 'zip_code', 'latitude', 'longitude', 'timezone', 'user_agent',
        'user_id', 'user_type', 'geo_point'
    ];
    
    // علاقات متعددة الأشكال للمستخدم
    public $morphTo = ['user'];
}
```

---

## السلوكيات (Behaviors)

### `Nano\Location\Behaviors\LocationModel`

هذا السلوك يضيف حقول الموقع (country_id, state_id, directorate_id) إلى أي نموذج يدعمه. يتم استخدامه في `ProductModel` و `UserModel` و `Delivery` وغيرها.

```php
// إضافة السلوك إلى نموذج
class MyModel extends Model
{
    public $implement = ['Nano\Location\Behaviors\LocationModel'];
}
```

### `Nano\Location\Behaviors\Address`

يضيف علاقات العنوان (address_1, address_2, latitude, longitude, إلخ) ويوفر دوال للتعامل مع العناوين.

```php
// الاستخدام
$model->extendClassWith('Nano\Location\Behaviors\Address');
```

---

## السمات (Traits)

### `Nano\Location\Traits\HasTimestamps`

تضيف حقول `created_at`, `updated_at`, `created_by`, `updated_by`, `deleted_by` مع معالجة تلقائية عند حفظ النموذج.

```php
use Nano\Location\Traits\HasTimestamps;

class MyModel extends Model
{
    use HasTimestamps;
}
```

---

## الواجهة الخلفية (Backend)

### قائمة المديريات (Directorates)

- المسار: `/backend/nano/location/directorates`
- الأعمدة: `id`, `name`, `code`, `country`, `state`, `is_enabled`, `sort_order`
- الفلاتر: `country`, `state`, `is_enabled`
- الإجراءات الجماعية: تمكين/تعطيل، حذف

### قائمة العناوين (Addresses)

- المسار: `/backend/nano/location/address`
- يظهر للمستخدمين الذين لديهم صلاحية `nano.location.address.access` (عناوينهم الخاصة) أو `nano.location.address.access_all` (جميع العناوين).
- الأعمدة: `address_1`, `country`, `state`, `directorate`, `user`, `addressable`, `is_default`, `created_at`

### قائمة الجنسيات (Nationalities)

- المسار: `/backend/nano/location/nationalitys`
- دعم استيراد تلقائي للجنسيات الافتراضية عبر زر في الواجهة.

### إعدادات الإضافة

- المسار: `الإعدادات > الموقع والجغرافيا > Location settings` (إذا تم تفعيلها في `registerSettings` - ملاحظ: معظم الإعدادات حالياً معلقة).

---

## إعدادات الإضافة

يمكن تخصيص الإضافة عبر ملف الإعدادات `config/nano/location/config.php` بعد نشره:

```php
return [
    'address' => [
        'is_allow_default_country_in_add' => true,
        'is_allow_default_state_in_add' => true,
        'is_allow_default_directorate_in_add' => true,
    ],
    'models' => [
        'use_soft_delete' => true,
    ],
];
```

**الإعدادات المتاحة في الكود (بعضها غير مفعل بعد):**

| المفتاح | القيمة الافتراضية | الوصف |
| :--- | :--- | :--- |
| `address.is_allow_default_country_in_add` | `true` | استخدام الدولة الافتراضية عند إنشاء عنوان جديد إذا لم يحدد المستخدم دولة. |
| `address.is_allow_default_state_in_add` | `true` | استخدام المدينة الافتراضية عند إنشاء عنوان جديد. |
| `address.is_allow_default_directorate_in_add` | `true` | استخدام المديرية الافتراضية عند إنشاء عنوان جديد. |

---

## الصلاحيات

| الصلاحية | التبويب | الوصف |
| :--- | :--- | :--- |
| `nano.location.address.access` | العناوين | الوصول إلى العناوين الخاصة بالمستخدم فقط. |
| `nano.location.address.access_all` | العناوين | الوصول إلى جميع العناوين (إدارة شاملة). |
| `nano.location.address.add` | العناوين | إضافة عنوان جديد. |
| `nano.location.address.edit` | العناوين | تعديل عنوان موجود. |
| `nano.location.address.delete` | العناوين | حذف عنوان. |

**ملاحظة:** صلاحيات الجنسيات والمديريات تُدار عبر `rainlab.location.access_settings` في الوقت الحالي.

---

## المكونات (Components)

### `countrySelect`

مكون لعرض قائمة منسدلة بالدول في الواجهة الأمامية.

**الاستخدام في الصفحة (CMS):**

```twig
[countrySelect]
inputName = "country"
inputId = "country_id"
inputClass = "form-control"
defaultValueSessionKey = "user_country"
selectedValue = "YE"
==
{% component 'countrySelect' %}
```

**الخصائص:**

| الخاصية | الوصف |
| :--- | :--- |
| `inputName` | اسم الحقل في الـ POST |
| `inputId` | معرف العنصر HTML |
| `inputClass` | كلاس CSS |
| `inputRequired` | هل الحقل مطلوب |
| `defaultValueSessionKey` | مفتاح الجلسة لجلب القيمة الافتراضية |
| `selectedValue` | القيمة المحددة مسبقاً (رمز الدولة) |
| `defaultLanguage` | اللغة الافتراضية للعرض |

---

## دوال المساعدة (Helpers)

توفر الإضافة عدة دوال مساعدة في مجلد `helpers/`:

### `country_helper.php`

```php
// الحصول على معرف الدولة من رمزها (YE => 1)
Nano\Location\Classes\Country::getIdByCountryCode($code);

// الحصول على رمز الدولة من معرفها
Nano\Location\Classes\Country::getCountryCodeById($id);
```

### `nationality_helper.php`

```php
// الحصول على قائمة الجنسيات
NationalityHelper::getList();

// الحصول على جنسية تلقائياً بناءً على الدولة
NationalityHelper::getDefaultForCountry($countryCode);
```

### `religions_helper.php`

```php
// قائمة الأديان المعروفة
ReligionsHelper::getList(); // ['muslim', 'christian', 'jewish', ...]
```

### `languages_helper.php`

```php
// قائمة اللغات المعروفة مع رموزها
LanguagesHelper::getList(); // ['ar' => 'Arabic', 'en' => 'English', ...]
```

### `locale_helper.php`

```php
// دوال مساعدة للتعامل مع الإعدادات الإقليمية
LocaleHelper::getCurrentLocale();
LocaleHelper::getDirection(); // 'rtl' or 'ltr'
```

---

## الأحداث (Events)

ترسل الإضافة الأحداث التالية والتي يمكن الاستماع إليها:

| اسم الحدث | المعاملات | الوصف |
| :--- | :--- | :--- |
| `nano.location.beforeSaveAddress` | `$address` | قبل حفظ العنوان |
| `nano.location.afterSaveAddress` | `$address` | بعد حفظ العنوان |
| `nano.location.beforeDeleteAddress` | `$address` | قبل حذف العنوان |
| `nano.location.afterGetDefaultAddress` | `$address, $addressable` | بعد جلب العنوان الافتراضي |

**مثال على الاستماع:**

```php
Event::listen('nano.location.beforeSaveAddress', function($address) {
    if (empty($address->address_1)) {
        throw new \Exception('Address line 1 is required');
    }
});
```

---

## الترجمة (Internationalization)

الإضافة تدعم الترجمة الكاملة للواجهة الخلفية ولنماذج `Country`, `State`, `Directorate`, `Addres`, `Nationality` عبر `RainLab.Translate`.

**ملفات الترجمة:**

| الملف | المحتوى |
| :--- | :--- |
| `lang/ar/lang.php` | العربية |
| `lang/en/lang.php` | الإنجليزية |

**مفاتيح الترجمة الرئيسية:**

```php
// في lang.php
'plugin' => [
    'name' => 'Location Directorates',
    'description' => 'Location based features...',
],
'menus' => [...],
'permissions' => [...],
'public' => [
    'country_label' => 'Country',
    'state_label' => 'City/State',
    'directorate_label' => 'District/Directorate',
    // ...
],
'models' => [ // أضيف في الإصدار 1.0.16
    'media_tab' => 'Media',
    'image' => 'Main Image',
    // ...
],
```

---

## أمثلة عملية

### 1. إضافة علاقة العنوان إلى نموذج مخصص

```php
use Nano\Location\Behaviors\Address;

class MyPost extends Model
{
    public $implement = [Address::class];
    
    // ستتم إضافة حقول: country_id, state_id, directorate_id, address_1, address_2, ...
}
```

### 2. جلب العنوان الافتراضي لمستخدم

```php
$user = User::find(1);
$defaultAddress = Addres::getDefaultAddress($user);
if ($defaultAddress) {
    echo $defaultAddress->address_1 . ', ' . $defaultAddress->directorate->name;
}
```

### 3. إنشاء عنوان جديد باستخدام الـ API (عبر `Nano.LocationApi`)

```http
POST /api/v2/addresses
{
    "addressable_type": "RainLab\\User\\Models\\User",
    "addressable_id": 1,
    "country_id": 1,
    "state_id": 5,
    "directorate_id": 12,
    "address_1": "شارع تعز",
    "is_default": true
}
```

### 4. إضافة صورة رئيسية لدولة في الكود

```php
$country = Country::find(1);
$country->image = Input::file('flag');
$country->save();
```

### 5. الحصول على قائمة المديريات لدولة معينة (للاستخدام في الواجهة الأمامية)

```php
$directorates = Directorate::whereHas('state.country', function($q) {
    $q->where('code', 'YE');
})->get();
```

---

## استكشاف الأخطاء وإصلاحها

### المشكلة: عدم ظهور حقول العناوين في نموذج المستخدم

**السبب:** عدم تطبيق السلوك `Address` أو `LocationModel` على النموذج.

**الحل:** تأكد من استدعاء `extendBackendUserModel()` و `extendFrontendUserModel()` في `Plugin.php` بشكل صحيح.

### المشكلة: خطأ `Class 'Nano\Location\Models\Directorate' not found`

**السبب:** لم يتم تشغيل هجرات الإضافة.

**الحل:**
```bash
php artisan nano:up
```

### المشكلة: عدم ظهور الصورة في قائمة الدول

**السبب:** لم يتم تعيين `image` في النموذج أو أن نوع العمود `simpleimage` غير مدعوم.

**الحل:** تأكد من رفع صورة وتخزينها، وتحقق من أن ملف `system/models/file/columns.yaml` يحتوي على تعريف `simpleimage`.

### المشكلة: تعارض مع إضافة `RainLab.Location` الأصلية

**السبب:** بعض التوسيعات قد لا تعمل إذا تم تحميل `RainLab.Location` بعد `Nano.Location`.

**الحل:** أعلن التبعية في `$require = ['RainLab.Location', ...]` وتأكد من ترتيب التحميل.

### المشكلة: خطأ في دالة `getNameList` للمديريات

**السبب:** تمرير `$countryId` بدلاً من `$stateId` في بعض الأحيان.

**الحل:** استخدم `Directorate::getNameList($stateId)` وليس `$countryId`.

---

## المراجع

- [GitHub - Nano.Location](https://github.com/nano/location-plugin)
- [توثيق RainLab.Location](https://github.com/rainlab/location-plugin)
- [توثيق October CMS Models](https://docs.octobercms.com/3.x/database/model.html)
- [توثيق Behavior DynamicAddMediaIncludes](./Docs-DynamicAddMediaIncludes-ar.md)
- [توثيق إضافة Nano.LocationApi](./Docs-Nano-LocationApi-ar.md)

---

## ملخص

إضافة `Nano.Location` هي حجر الزاوية لإدارة البيانات الجغرافية في مشاريع نانوسوفت. بفضل هيكلها المرن، وتوسيعها لـ `RainLab.Location`، ودعمها للعناوين متعددة الأشكال، والجنسيات، والمديريات، وتتبع المسارات، والوسائط المتعددة، يمكن للمطورين بناء تطبيقات غنية بالموقع بسرعة وكفاءة. الإضافة متكاملة مع `Nano.LocationApi` لتوفير واجهة RESTful، و `Nano.ActivityLog` لتتبع النشاطات.

**التحديثات الأخيرة (1.0.16):**
- دعم الوسائط المتعددة في الدول والمدن والمديريات.
- إضافة أعمدة وحقول في الواجهة الخلفية للميديا.
- دعم الترجمة للحقول الجديدة.

نوصي بالترقية إلى أحدث إصدار للاستفادة من هذه الميزات.

---

**تاريخ التوثيق:** 2025-04-30  
**المؤلف:** فريق تطوير نانوسوفت (Nano2Soft Team)  
**الترخيص:** حقوق النشر محفوظة لشركة نانوسوفت