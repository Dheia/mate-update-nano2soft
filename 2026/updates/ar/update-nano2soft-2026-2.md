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

## 2026-2-2 

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

