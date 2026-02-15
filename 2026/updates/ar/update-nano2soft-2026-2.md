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

## 2025-10-25 - 2026-2-10

**Support Nano3.Packer Tools Version V2**

تم تطوير وحدة برمجية متكاملة بشكل احترافي لمعالجة وتشفير الكود وتجهيز الكود لمرحلة الانتاج حيث من خلال هذه الوحدة يمكن تشفير حزمة برمجية كامل بعدة طرق  من خلال ملف مضغوط او من خلال تحديد الوحدة المراد تشفيرها فى النظام . 

## 2026-2-11

**Update Nano.Tags And Nano.TagsApi**

    - Support TagManager Class
    - Support TagIntegrationHandler EventHandlers Class
    - Support DynamicAddIncludeTags Transformer Class
    - Support tags Relation In Types
    - Support tags Relation In Categories
    - Support Tags Relation And tagsId Filter in api/v1/tags/types API Version 2.
    - Support Tags Relation And tagsId Filter in api/v1/tags/categories API Version 2.
    - Update Docs Api Version 2.
تم تحديث الوحدة البرمجية الخاصة بإدارة الهاشتجات العامة حيث تم تحسين آلية إدارة الهاشتجات وطريقة حقن الكائنات والواجهات والاستعلامات وال api بشكل احترافي من خلال كلاس باسم TagManager يتولي ربط اي كائن او واجهة بالهشتاجات بشكل مرن مع امكانية التحكم بخصائص كل كائن من حيث الواجهات وغيرها وتم انشاء توثيق كامل لطريقة التعامل مع الآلية الجديدة . وتم تحديث ال api والتوثيق الخاص به . كما اصبح بالامكان فلترة الفئات العامة والتصنيفات العامة بحسب الهاشتاجات .

### Example 1.1.5 get List Types include=tags And where tagsId=43,54
### Example 1.1.12 get List Categories include=tags And where tagsId=1,2

**فى المثال التالي سنقوم بجلب التصنيفات حسب الهاشتجات مع تضمين العلاقة الخاصة بالهاشتجات كالتالي     **

 
 GET http://localhost:8006/api/v1/tags/categories?tagsId=1,2&include=tags

**البراميترات الممررة كالتالي **

```json
{
  "tagsId": "1,2",
  "include": tags,
}
```

## 2026-2-8 - 2026-2-12

**Support Google Merchant Validator Version 2**

    - Support ValidationLibrary Class
    - Support GoogleMerchantFields Class
    - Support GoogleMerchantValidator Class
    - Support Interface Google Merchant Validator
تم انشاء وحدة برمجية جديدة خاصة بالتحقق من صحة محتوي عناصر ملف جوجل ميرشانت بطريقة احترافية حيث يتم التحقق من كافة القواعد لكل صنف فى الملف .
الرابط العام للاطلاع على واجهة هذه الوحده 

https://account.now-ye.com/nano2/googlemerchant/validator





## 2026-2-12
**Update GDPR**
تم تحديث الوحدة البرمجية الخاصة بالكوكيز وطلب الموافقة علية من قبل المتصفح او الزائر للموقع وتم اضافة api تجريبي لهذا الجزء .
نقاط النهاية التي تم اضافتها فى ال api هى كالتالي 

```json

POST /api/v1/gdpr/cookie/accept-defaults
قبول الكوكيز الافتراضية (مثل cookieBanner->onAccept)
)
GET api/v1/gdpr/cookie/check/{code}/{level?}
التحقق من صلاحية كوكي معين
GET api/v1/gdpr/cookie/consent
الحصول على حالة موافقات المستخدم الحالية
POST api/v1/gdpr/cookie/decline-all
رفض جميع الكوكيز (مثل cookieBanner->onDecline)
POST api/v1/gdpr/cookie/reset
إعادة تعيين جميع الموافقات
GET api/v1/gdpr/cookie/settings
الحصول على جميع مجموعات الكوكيز والإعدادات
GET api/v1/gdpr/cookie/status
الحصول على حالة الموافقة العامة
POST api/v1/gdpr/cookie/update
تحديث تفضيلات الكوكيز (مثل CookieManager)
GET api/v1/gdpr/data-retention/policies
الحصول على جميع سياسات الاحتفاظ بالبيانات
```

## 2026-2-6 - 2026-2-15 

**Support Nano3.Legal**

تم اطلاق وحدة برمجية جديدة متكاملة خاصة بإدارة الوثائق والمستندات القانونية حيث تضم الوحدة كافة المستندات القانونية كسياسة الخصوصية والاستخدام وسياسة الاستدار والدفع وغيره . وتم انشاء جزء ال api الخاص بها بشكل كامل واحترافي وتم عمل التوثيق الخاص بها وتم تصنيف انواع الوثائق كالتالي 
```json
{
        "ecommerce": "التجارة الإلكترونية",
        "rental": "التأجير والتطبيقات",
        "contract": "العقود والطلبات",
        "user": "المستخدمين والتسجيل",
        "employment": "التوظيف والعمل",
        "security": "الأمن والحماية",
        "general": "عامة",
        "industry": "خاصة بالصناعة"
}
```

نقاط النهاية الخاصة بال api هى كالتالي 
```
GET api/v1/legal/support/ref-types/{id?}
GET api/v1/legal/support/types/{id?}
GET api/v1/legal/documents
GET api/v1/legal/documents/activelystats
GET api/v1/legal/documents/{id}
GET api/v1/legal/consents
GET api/v1/legal/consents/activelystats
GET api/v1/legal/consents/{id}
```

## 2026-2-15

**Update OrdersRequests And OrdersLists Api Version 2**

    - Update OrdersRequests And OrdersLists Api Version 2
    - Support Filter is_user_or_delivery_user_id  In OrdersLists And OrdersRequests Api Version 2
    - Support Permission Backend User In get OrdersRequests Api Version 2
    - Support Filter companys_id And departments_id In get OrdersRequests Api Version 2
تم تحديث جزء إدارة العروض الخاصة بطلبات التوصيل فى لوحة التحكم وفى ال api الخاص بها .

