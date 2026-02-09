# Update 2026-2

**تحديثات شهر اثنين**

## 2026-02-01 - 2026-02-02

**Support Nationalitys In Nano.Location**

    - Created table nano_location_nationalities
    - create_nano_location_nationalities_table.php
    - Support NationalityHelper Class
    - Support ReligionsHelper Class
    - Support Interface Nationalitys In Backend 
    - Support Auto Seed Nationalitys In Interface

تم تحديث الوحدة البرمجية الخاصة بإدارة المناطق الجغرافية وتم اضافة جدول جديد خاص بالجنسيات وتم انشاء واجهات إدارة الجنسيات فى لوحة تحكم النظام مع آلية لانشاء بيانات كافة الجنسيات من الوجهة بضغطة زر وتم انشاء كلاسات احترافية لإدارة الجنسيات والديانات . 

## 2026-2-3

** Support Nationalitys In Nano.LocationAPI **

    - Support Nationalitys Api
    - Support Docs Nationalitys Api
    - Support Nationalitys Endpoint Api And Docs Api
تم تحديث الوحدة البرمجية Nano.LocationAPI وتم اضافة ال api الخاص بالتعامل مع الجنسيات وتم كتابة التوثيق بشكل كامل مع عدة امثلة توضيحية .

```
GET /api/v1/location/nationalities
GET /api/v1/location/nationalities/{id}
GET /api/v1/location/nationalities/activelystats
```

## 2026-2-5 

** Support Filter By Targets is_has_targets target_type target_id In Adverts in API Version 2 **

تم تحديث ال api الخاص بالاعلانات وتم اضافة فلاتر خاص بفلترة الاعلانات حسب نوع الكائن المستهدف بحيث اصبح بالامكان جلب الإعلانات حسب قسم معين او تصنيف معين وما الى ذالك وتم تحديث التوثيق واضافة عدة امثلة للتوضيح 


**لفرض ارجاع الاعلانات التي تستهدف كائنات معين فى النظام كاصنف معين او متجر معين او حسب فئات معينة او حسب تصنيفات معينه او حسب نوع طلب معين نستخدم البراميترات التالية  **

```json
{
    "is_has_targets": 1,
    "target_type": "هنا يتم تمرير نوع الكائن المرتبط بالاعلان ",
    "target_id": "هنا يتم تمرير معرف الكائن المرتبط بالاعلان ",
}
```

**انواع الكائنات التي من الممكن ان ترتبط بالاعلانات والتي يمكن تمريرها ضمن البراميتر target_type هى كالتالي **

```json
{
    "Nano\Orders\Models\OrdersType": "انواع الطلبات ",
    "Nano\Tags\Models\Type": "الفئات الرئيسية ",
    "Tss\Basic\Models\Department": "المتاجر والفروع ",
    "Nano\Tags\Models\Categorie": "التصنيفات",
    "Nano\Tags\Models\Tag": "الهاشتجات",
    "Nano\Shop\Models\Product": "الاصناف",
}
```

**للتوضيح فكرة التعامل مع الكائنات المستهدفة انظر الامثلة التالية**

Example 1.2.3
Example 1.2.4
Example 1.2.5
Example 1.2.6


## 2026-2-3 -2026-2-6 

**Update Nano.Location And Nano.LocationApi And Nano.UserPlus And Nano.AuthApi**

    - Support LocaleHelper Class
    - Support LanguagesHelper Class
    - Update Address Behaviors Class
    - Support Languages Endpoins Api And Docs Api
    - Support Religions Endpoins Api And Docs Api
    - Support Relation belongsTo Nationalitys in Frontend User Model.
    - Support dropdown field nationality and language In Backend Interface
تم تحديث جزء إداة مستخدمين الفرونت اند وتم تحسين واجهات إدارة المستخدمين فى الباك اند وتم اضافة فلاتر لفلترة المستخدمين حسب الديانه او الجنسية او اللغة وتم تحسين حقول ادخال هذه البيانات وتم اضافة علاقات جديدة الي ملف المستخدم خاصة ببيانات الجنسية وتم انشاء ال api التالي مع التوثيق الكامل لكل شي 

ال api المضاف كالتالي 

```
GET /api/v1/location/religions
GET /api/v1/location/religions/{id}
GET /api/v1/location/religions/activelystats

GET /api/v1/location/languages
GET /api/v1/location/languages/{id}
GET /api/v1/location/languages/activelystats
```


## 2026-2-7 

**Update RoutesBrowser Api To v3**

قمنا بتحديث وترقية الجزء الخاص بتصفح واختبار ال api ليتوافق مع الاصدار الثالث من النظام بالاضافة الى امكانية تصدير ملف collection-api.json مع التوثيق او بدون التوثيق من اجل استراد ال api فى جهات خارجية كا postmain وغيرها .

## 2026-2-8 

**Update FormWidgets Map In Backend**

تم تحسين وتحديث حقول الادخال الخاصة بنظام الخرائط فى جزء الباك اند .

##2026-2-9

**Support VisitModel In OrdersType**

Support VisitModel In OrdersType
    - Update Nano2.Visitors Support extendOrdersTypeModel And Config
    - Update Nano.OrdersApi
    - Development of the Order Types System Adding Behaviors VisitModel In OrdersType Models And OrdersTypeTransformer
    - Support Visitors In Order Types
    - Support Visit Event Api In Order Types

تم تحديث جزء انوع الطلبات بحيث تم دعم خاصية تسجيل المشاهدات والزيارة على مستوي انواع الطلبات بحيث يمكن معرفة عدد زيارة كل نوع طلب وتم تحديث ال api الخاص بهذا الجزء وتم تحديث التوثيق الخاص به . 
 ولتفعيل او ايقاف هذه الميزة تم اضافة الاعدادات التالية 

```
#السماح بتسجيل الزيارات عند الدخول الى السجل فى نوع الطلب
NANO2_VISITORS_IS_SUPPORT_ORDERS_TYPES_SHOW= true
#السماح بتسجيل الزيارات عند جلب السجل فى القائمة انوع الطلبات 
NANO2_VISITORS_IS_SUPPORT_ORDERS_TYPES_LIST= true
```

