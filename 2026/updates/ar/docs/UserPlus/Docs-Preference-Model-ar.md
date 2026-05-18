# توثيق نظام تفضيلات المستخدم (Preference & UserPreference)

## نظرة عامة

يوفر نظام التفضيلات في `Nano.UserPlus` آلية مشابهة تماماً لتفضيلات المستخدمين في لوحة التحكم (`Backend\Models\Preference`)، لكنها موجهة لمستخدمي **الواجهة الأمامية**. يتكون النظام من ثلاثة مكونات رئيسية:

| المكون | الدور |
| :--- | :--- |
| `Preference` (موديل) | واجهة المطور: تحدد إعدادات التفضيلات (الحقول، القيم الافتراضية، التحقق). |
| `UserPreference` (موديل) | التخزين الفعلي في قاعدة البيانات (جدول `frontend_user_preferences`). |
| `UserPreferencesModel` (سلوك) | الرابط بين `Preference` وسجل المستخدم الحالي. |

## هيكل قاعدة البيانات

جدول `frontend_user_preferences`:

| العمود | الوصف |
| :--- | :--- |
| `id` | معرف فريد |
| `user_id` | معرف المستخدم (مرتبط بـ `users`) |
| `namespace` | نطاق الإعدادات (مثلاً `frontend`) |
| `group` | مجموعة الإعدادات (مثلاً `frontend`) |
| `item` | عنصر الإعدادات (مثلاً `preferences`) |
| `value` | قيمة JSON تخزن جميع الإعدادات |

**المفتاح الفريد:** `namespace + group + item + user_id`

## موديل `Preference`

### الوصف

هذا الموديل مخصص لاستخدام المطورين لإنشاء صفحات إعدادات مخصصة للمستخدمين. يرث من `Model` ويستخدم سلوك `UserPreferencesModel`.

### الخصائص الهامة

- `$settingsCode`: `'frontend::frontend.preferences'` – يحدد المفتاح الفريد للإعدادات.
- `$settingsFields`: `'fields.yaml'` – ملف YAML لتعريف حقول النموذج.
- `$implement`: `[UserPreferencesModel::class]` – يربط النموذج بسجل المستخدم.

### الدوال الرئيسية

#### `initSettingsData()`

تستدعى عند أول مرة يتم فيها الوصول إلى الإعدادات (إن لم تكن موجودة). تقوم بتعيين قيم افتراضية:

- `locale`: من `app.locale`.
- `timezone`: من `backend.timezone` أو `app.timezone`.
- إعدادات المحرر (font_size, word_wrap, theme...).

#### `useDefaults()` (خاصة)

إذا كانت الإعدادات موجودة ولكن بعض المفاتيح مفقودة (بسبب تحديث)، تقوم بدمج القيم الافتراضية من `BasicHelper::getPreferenceInitSettingsData()` عند `afterFetch()`، شرط أن يكون `nano.userplus::preference.use_defaults` مفعلاً.

#### `afterFetch()`

حدث بعد جلب السجل من قاعدة البيانات. يستدعي `useDefaults()` إذا لزم الأمر.

#### `formSelectRefType($name, $selectedValue, $options)`

دالة مساعدة لتوليد حقل `select` لقائمة أنواع الحسابات (`ref_type`). تستخدم داخلياً `getRefTypeList()`.

#### `formSelectGender($name, $selectedValue, $options)`

مشابهة لكن لقائمة الجنس (`gender`).

#### `getRefTypeList()` (static)

ترجع مصفوفة بجميع أنواع الحسابات المدعومة. **الأنواع تعتمد على إعدادات `config.php` ووجود الإضافات ذات الصلة.**

**مثال:**
```php
$types = Preference::getRefTypeList();
// ['user' => 'مستخدم عادى', 'delivery_morph_key' => 'عامل توصيل', ...]
```

#### `getGenderOptions()` (static)

ترجع خيارات الجنس:
```php
[
    'male' => 'ذكر',
    'female' => 'أنثى',
]
```

#### `setAppLocale()` / `setAppFallbackLocale()`

تضبط لغة التطبيق ولغة الاحتياط بناءً على تفضيل المستخدم (تستخدم `Session` للتخزين المؤقت).

#### `applyConfigValues()`

تطبق إعدادات المستخدم على `Config` (مثل `app.locale`).

#### `resetDefault()`

تعيد ضبط الإعدادات إلى القيم الافتراضية وتمسح الجلسة.

#### `getLocaleOptions()`

ترجع قائمة اللغات المدعومة مع الأعلام (مصفوفة ضخمة من `ar`, `en`, `fr`...).

#### `getTimezoneOptions()`

ترجع قائمة المناطق الزمنية بتنسيق `(UTC +03:00) Asia/Riyadh`.

---

## موديل `UserPreference`

### الوصف

النموذج الفعلي الذي يتعامل مع جدول `frontend_user_preferences`. يرث من `October\Rain\Auth\Models\PreferencesBase` ويخصصه لمستخدمي الواجهة الأمامية.

### الخصائص

- `$table`: `'frontend_user_preferences'`
- `$timestamps`: `false`

### الدوال الرئيسية

#### `resolveUser($user)`

تحل المستخدم: إذا لم يمرر، تجلب المستخدم الحالي من `FrontendAuth`. ترمي `SystemException` إذا لم يكن هناك مستخدم مسجل دخول.

#### `scopeOnNamespace($query, $name)`

نطاق للبحث حسب `namespace`.

#### `scopeOnGroup($query, $name)`

نطاق للبحث حسب `group`.

#### `scopeOnItem($query, $name)`

نطاق للبحث حسب `item`.

#### `scopeIsUserPreference($query)`

نطاق مُعد مسبقاً: `namespace=frontend, group=frontend, item=preferences`.

#### `scopeIsLocationPreference($query)`

نطاق مُعد مسبقاً: `namespace=nano.sitemanager, group=location, item=manager`.

#### `getAllConfigUser($user = null)` (static)

تجلب جميع إعدادات مستخدم معين (أو الحالي) وتنظمها في مصفوفة متعددة المستويات:
```
[
    'namespace_group_item' => { value object },
    ...
]
```

#### `getConfigByUserId($userId)` (static)

مثل السابقة لكن بمعرف مستخدم محدد.

---

## سلوك `UserPreferencesModel`

### الوصف

سلوك يُضاف إلى أي موديل (مثل `Preference`) ليجعله يتعامل مع تفضيلات المستخدم الحالي. يرث من `System\Behaviors\SettingsModel` لكنه يعيد توجيه التخزين إلى `UserPreference`.

### الدوال الملغاة (Overridden)

| الدالة | الوصف |
| :--- | :--- |
| `instance()` | تجلب أو تنشئ سجل التفضيلات للمستخدم الحالي. تخزن مؤقتاً في `$instances`. |
| `isConfigured()` | تتحقق إذا كان قد تم إعداد التفضيلات مسبقاً. |
| `getSettingsRecord()` | تجلب سجل `UserPreference` المطابق للمفتاح (`settingsCode`) والمستخدم الحالي. تستخدم `remember(1440)` للتخزين المؤقت. |
| `beforeModelSave()` | قبل الحفظ، تعين `item`, `group`, `namespace`, `user_id` وتحول `fieldValues` إلى `value` (JSON). |
| `isKeyAllowed()` | تسمح بمرور `namespace` و `group` بالإضافة إلى مفاتيح الحقول العادية. |
| `getCacheKey()` | تولد مفتاح كاش فريد يجمع `recordCode` و `user_id`. |

---

## أمثلة عملية

### 1. إنشاء صفحة إعدادات مخصصة للمستخدم

```php
// موديل MyUserSettings.php
class MyUserSettings extends Model
{
    public $implement = [\Nano\UserPlus\Behaviors\UserPreferencesModel::class];
    public $settingsCode = 'myplugin::settings';
    public $settingsFields = 'fields.yaml';

    public function initSettingsData()
    {
        $this->theme = 'dark';
    }
}
```

### 2. الوصول إلى إعدادات المستخدم الحالي

```php
$settings = MyUserSettings::instance();
echo $settings->theme; // 'dark'
$settings->theme = 'light';
$settings->save();
```

### 3. جلب تفضيلات موقع المستخدم

```php
$locationPref = UserPreference::forUser()
    ->scopeIsLocationPreference($query)
    ->first();
```

---

## ملاحظات هامة

- التخزين المؤقت: `getSettingsRecord` تستخدم `remember(1440)` لتقليل الاستعلامات.
- `useDefaults()` في `Preference` تضمن أن أي حقل جديد يضاف لاحقاً سيحصل على قيمة افتراضية دون الحاجة لإعادة تعيين الإعدادات.
- دالة `getAllConfigUser` مفيدة لعرض جميع إعدادات المستخدم في لوحة التحكم أو للتصدير.
