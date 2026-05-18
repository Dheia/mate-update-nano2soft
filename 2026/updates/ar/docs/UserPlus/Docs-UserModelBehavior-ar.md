# توثيق سلوك UserModelBehavior

## نظرة عامة

`Nano\UserPlus\Behaviors\UserModelBehavior` هو سلوك (Behavior) يتم إضافته ديناميكياً إلى موديل `RainLab\User\Models\User` عبر الإضافة `Nano.UserPlus`. يهدف هذا السلوك إلى إثراء نموذج المستخدم بعلاقات إضافية مع نظام التفضيلات، ونطاقات استعلام (Scopes) مفيدة، ودوال مساعدة لخيارات القوائم المنسدلة.

لا يتم تحميل هذا السلوك افتراضياً في `RainLab.User`، بل تقوم `Nano.UserPlus` بتمديد نموذج المستخدم به عبر `$model->extendClassWith(...)` في دالة `boot()`.

## العلاقات المضافة

يقوم السلوك في دالة `__construct` بإضافة العلاقات التالية إلى نموذج المستخدم:

| العلاقة | النوع | الموديل المرتبط | الوصف |
| :--- | :--- | :--- | :--- |
| `preferences` | `hasMany` | `Nano\UserPlus\Models\UserPreference` | جميع تفضيلات المستخدم (بدون نطاق). |
| `user_preferences` | `hasMany` | `Nano\UserPlus\Models\UserPreference` | تفضيلات المستخدم من نوع `frontend::frontend.preferences` (نطاق `isUserPreference`). |
| `user_preferences_location` | `hasMany` | `Nano\UserPlus\Models\UserPreference` | تفضيلات موقع المستخدم من نوع `nano.sitemanager::location.manager` (نطاق `isLocationPreference`). |

## نطاقات الاستعلام (Query Scopes)

### `scopeWhereUserPreferenceStates($query, $state_id = null)`

يبحث عن المستخدمين الذين لديهم تفضيل موقع يطابق `state_id` محدد.

- إذا لم يُمرر `$state_id`، يحاول جلبه من `\SiteLocation::getEditState()`.
- يستخدم العلاقة `user_preferences_location` للبحث في قيمة JSON المخزنة (`value->editState_id`).

**مثال:**
```php
$usersInState = User::whereUserPreferenceStates(5)->get();
```

### `scopeWhereUserPreferenceStatesFilter($query, $state_id = null)`

دالة مغلفة لـ `scopeWhereUserPreferenceStates`، لكنها تتعامل مع المصفوفات (في حال أتت من فلتر مجموعة).

- إذا كان `$state_id` مصفوفة، تأخذ القيمة الأولى.

**مثال:**
```php
// استخدام في الفلاتر
User::whereUserPreferenceStatesFilter([5])->get();
```

### `scopeIsSysCompanys($query)`

نطاق معقد لتصفية المستخدمين حسب الصلاحيات التنظيمية:

- يستبعد البريد الإلكتروني `info@nano2soft.com`.
- إذا كان المستخدم الحالي لا يملك صلاحية الوصول لكل الشركات (`BasicHelper::checkAccessAllCompanys`)، يطبق نطاق `IsCompany`.
- إذا كان لا يملك صلاحية الوصول لكل الأقسام (`BasicHelper::checkAccessAllDepartments`)، يطبق نطاق `IsDepartment`.

### `scopeIsSuspendedV1($query, $value = true)`

يبحث عن المستخدمين المعلقة حساباتهم عبر علاقة `throttles` (يجب أن تكون موجودة).

- `$value = true`: يبحث عن المعلقة.
- `$value = false`: يبحث عن غير المعلقة.

### `scopeIsBannedV1($query, $value = true)`

مشابه للتعليق لكن للحظر (محظور/غير محظور).

### `getInetisIsBannedAttributeV1()`

وسيط وصول (Accessor) مخصص لفحص حالة الحظر للمستخدم الحالي عبر `AuthManager`.

### `setInetisIsBannedAttributeV1($value)`

وسيط تعيين (Mutator) لتغيير حالة الحظر. يمنع المستخدم من حظر نفسه ويرمي `ApplicationException` في هذه الحالة.

## دوال مساعدة لخيارات القوائم

### `getEditStateListOptions()`

ترجع قائمة الولايات/المدن بناءً على إعدادات الموقع الحالي أو الافتراضي.

**المنطق:**
1. تتحقق من وجود `RainLab\Location\Models\State`.
2. تحاول جلب `SiteLocation::getModelState()`.
3. إذا لم توجد، تستخدم الدولة والمدينة الافتراضيتين.
4. ترجع `State::getNameList($country_id)` مع إضافة المدينة الحالية في أعلى القائمة إن لم تكن موجودة.

**مثال:**
```php
$stateOptions = $user->getEditStateListOptions();
// ['1' => 'صنعاء', '2' => 'عدن', ...]
```

### `getPhoneTypeOptions($value, $formData)`

ترجع أنواع الهواتف المدعومة (mobile, home, work, fax...). إذا كانت إضافة `Tss.Basic` مثبتة، تفوض لها.

### `getSortShowOptions($value, $formData)`

ترجع خيارات ترتيب العرض من 1 إلى 10. تفوض لـ `Tss.Basic` إن وجدت.

### `getWebsiteTypeOptions($value, $formData)`

ترجع أنواع المواقع (website, facebook). تفوض لـ `Tss.Basic` إن وجدت.

## ملاحظات هامة

- جميع الدوال المساعدة لخيارات القوائم تتحقق من وجود `Tss\Basic\Classes\RepeaterFieldsData` لإعطاء الأولوية لإعدادات المنصة.
- نطاقات `IsSuspendedV1` و `IsBannedV1` تفترض وجود علاقة `throttles` في نموذج المستخدم (قد تحتاج إلى إضافتها يدوياً).
- دالة `getInetisIsBannedAttributeV1` تستخدم وسيط وصول مخصص (وليس `getIsBannedAttribute`) لتفادي التعارض مع أي إضافة أخرى.

