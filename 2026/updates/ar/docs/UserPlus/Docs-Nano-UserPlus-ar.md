# توثيق إضافة Nano.UserPlus

## نظرة عامة

إضافة `Nano.UserPlus` هي إضافة شاملة لمنصة OctoberCMS تقوم بتوسيع وظائف إدارة المستخدمين التي توفرها إضافة `RainLab.User` بشكل جذري. تهدف الإضافة إلى تحويل نظام المستخدمين الأساسي إلى نظام متكامل يدعم:

- **ملفات شخصية غنية** تتضمن عشرات الحقول الإضافية (الموقع، الاتصال، العمل، الروابط الاجتماعية).
- **أنواع حسابات متعددة ومرنة** (مستخدم عادي، عامل توصيل، موظف، طالب، مورد...) يمكن ربطها بكائنات أخرى عبر علاقات بوليمورفيك.
- **تكامل متقدم مع نظام الإشعارات** `RainLab.Notify` عبر شروط وأحداث مخصصة.
- **أدوات استيراد وتصدير** متقدمة للمستخدمين مع صلاحيات منفصلة.
- **فلاتر متطورة** في لوحة التحكم لتسهيل البحث عن المستخدمين.
- **إدارة تفضيلات** المستخدمين (الواجهة الأمامية) بنظام مشابه لتفضيلات backend.
- **توافق كامل** مع الإصدارات الحديثة من RainLab.User (دعم `first_name`/`last_name`).
- **تحكم متقدم بمجموعات المستخدمين** (تفعيل/تعطيل، ترتيب، مجموعة افتراضية).

## المتطلبات

- `RainLab.User`
- `Nano.Location`
- `RainLab.Notify`

## المكونات الرئيسية

| المكون | الوصف |
| :--- | :--- |
| `Plugin.php` | الملف الرئيسي، يحتوي على دوال `boot` و `register` التي تقوم بعمليات الحقن والتوسيع. |
| `UserModelBehavior` | سلوك يُضاف إلى `UserModel` ليوفر علاقات إضافية، نطاقات استعلام (Scopes)، ودوال مساعدة. |
| `Preference` | موديل يحاكي `Backend\Models\Preference` لإدارة تفضيلات مستخدمي الواجهة الأمامية. |
| `UserPreference` | موديل فعلي يخزن التفضيلات في جدول `frontend_user_preferences`. |
| `UserPreferencesModel` | سلوك (Behavior) يُضاف إلى `Preference` ليربطه بسجل المستخدم الحالي. |
| `RainlabUser2CompatibilityHandler` | معالج حدث يضمن توافق الحقول القديمة (`name`, `surname`) مع الجديدة (`first_name`, `last_name`). |
| `UserLocationAttributeCondition` | شرط إشعارات مخصص يعتمد على موقع المستخدم. |
| `Components\Notifications` | مكون لعرض إشعارات المستخدم في الواجهة الأمامية. |

## الحقول الموسعة في جدول `users`

تقوم الإضافة بإضافة الحقول التالية إلى نموذج المستخدم عبر `addFillable`:

| الحقل | النوع | الوصف |
| :--- | :--- | :--- |
| `phone` | string | رقم الهاتف |
| `mobile` | string | رقم الجوال (إجباري) |
| `company` | string | اسم الشركة |
| `street_addr`, `city`, `zip` | string | عنوان الشارع، المدينة، الرمز البريدي |
| `country`, `country_id` | relation | الدولة (علاقة مع `RainLab\Location\Models\Country`) |
| `state`, `state_id` | relation | الولاية/المدينة |
| `directorate`, `directorate_id` | string/int | المديرية |
| `address_1`, `address_2`, `street_name`, `street_number` | string | تفاصيل العنوان |
| `latitude`, `longitude` | float | الإحداثيات الجغرافية |
| `radius`, `geo_components`, `postcode`, `country_long`, `formataddress`, `vicinity` | mixed | بيانات موقع إضافية |
| `companys_id`, `departments_id`, `employees_id` | int | معرفات الشركة/القسم/الموظف |
| `ref_type`, `ref_id` | string/int | نوع الحساب ومعرفه (لعلاقات polymorphic) |
| `referral_id` | int | معرف الدعوة |
| `barcode`, `manual_code`, `accounts_id` | string/int | رموز تعريفية |
| `gender` | string | الجنس (ذكر/أنثى) |
| `date_of_birth` | datetime | تاريخ الميلاد |
| `created_by`, `updated_by` | int | معرف المنشئ/المعدل |
| `is_blocked_orders` | boolean | حظر الطلبات |
| `business_number`, `vat_number` | string | الرقم التجاري والضريبي |
| `language` | string | لغة المستخدم |
| `nationality` | int | معرف الجنسية |
| `bio` | text | نبذة عن المستخدم |
| `links` | json | روابط اجتماعية (JSON) |
| `website` | json | مواقع ويب (JSON) |

كما يتم إضافة العلاقات التالية:
- `belongsTo['Nationalitys']`: مع موديل `Nano\Location\Models\Nationality`.
- `morphMany['notifications']`: مع `RainLab\Notify\Models\Notification`.
- `morphTo['ref']`: علاقة بوليمورفيك حسب `ref_type`.
- `hasMany['preferences']`, `user_preferences`, `user_preferences_location`: مع `UserPreference` (مضافة عبر `UserModelBehavior`).

## أنواع الحسابات (`ref_type`)

توفر الإضافة نظام أنواع حسابات مرن يسمح بربط المستخدم بكائنات مختلفة في المنصة. يمكن التحكم بالأنواع المتاحة عبر ملف `config.php`:

```php
'ref_type' => [
    'delivery' => true,
    'department' => true,
    'employee' => false,
    'student' => true,
    // ...
]
```

الأنواع المدعومة (حسب الإضافات المثبتة):
- `user` (افتراضي)
- `delivery` (Nano.Deliverys)
- `department` (Tss.Basic)
- `employee` (Tss.Basic)
- `customer` (Tss.SalesAndMarketing)
- `suppler` (Tss.Purchasing)
- `syndical` (Tss.Purchasing)
- `partner` (Tss.Accounts)
- `student` (Tss.Student)
- `mparent` (Tss.Student)
- `author` (Nano.Authors)

كل نوع يستخدم `getMorphClass()` للحصول على اسم العلاقة polymorphic الصحيح.

## مجموعات المستخدمين (`user_groups`)

بدءاً من الإصدار 1.1.7، تم توسيع جدول `user_groups` بالحقول:

| الحقل | الوصف |
| :--- | :--- |
| `is_new_user_default` | تعيين المجموعة كافتراضية للمستخدمين الجدد |
| `is_active` | تفعيل/تعطيل المجموعة |
| `sort_order` | ترتيب المجموعة في القوائم |

ويتم حقن هذه الحقول تلقائياً في نموذج وقائمة `RainLab\User\Controllers\UserGroups`.

## التوافق مع RainLab.User 2.x

عبر الكلاس `RainlabUser2CompatibilityHandler`، يتم ربط الحقول القديمة (`name`, `surname`) مع الجديدة (`first_name`, `last_name`)، مما يضمن عدم تعطل أي كود قديم.

## الصلاحيات

| الصلاحية | الوصف |
| :--- | :--- |
| `nano.userplus.users.import` | استيراد المستخدمين |
| `nano.userplus.users.export` | تصدير المستخدمين |

## الأحداث (Events)

- `backend.list.extendColumns`: لتوسيع أعمدة قوائم المستخدمين والمجموعات.
- `rainlab.user.register`: يمكن استخدامه لربط المستخدم الجديد بالمجموعة الافتراضية.

## الترجمة

جميع النصوص قابلة للترجمة عبر ملف `lang.php` في مجلد `lang`. اللغة الافتراضية مدعومة بالعربية.

## ملاحظات هامة

- قاعدة `unique` على حقل `mobile` يتم ضبطها ديناميكياً لاستثناء السجل الحالي عند التحديث.
- حقول `password`، `email`، `mobile`، `ref_type`، `ref_id` يمكن جعلها للقراءة فقط عند تعديل مستخدم موجود حسب إعدادات `config.php`.
- الحقول المخفية (مثل `last_login`، `ref_type`) في قوائم المستخدمين يمكن إظهارها عبر إعدادات القائمة.
