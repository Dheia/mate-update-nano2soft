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

See [docs/Docs-TagManager-en.md](./docs/Docs-TagManager-ar.md)


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

تم اطلاق وحدة برمجية جديدة متكاملة خاصة بإدارة الوثائق والمستندات القانونية حيث تضم الوحدة كافة المستندات القانونية كسياسة الخصوصية والاستخدام وسياسة الاسترداد والدفع وغيرها .وتم انشاء كلاسات تحترافية لتدير انواع الوثائق وتصنيفات الوثائق وانشاء نماذج الوثائق بشكل آلي حسب المواصفات العالمية . وتم انشاء جزء ال api الخاص بها بشكل كامل واحترافي وتم عمل التوثيق الخاص بها وتم انشاء دوال اختبار لاختبار الوحدة البرمجية .

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
 تصنيف انواع الوثائق كالتالي 
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

اما معرفات الوثائق الاساسية فهيا كالتالي 
```json
{
        "privacy_policy": "سياسة الخصوصية",
        "terms_of_service": "شروط الخدمة",
        "return_policy": "سياسة الإرجاع والاستبدال",
        "shipping_policy": "سياسة الشحن والتوصيل",
        "cookie_policy": "سياسة الكوكيز",
        "user_agreement": "اتفاقية المستخدم"
}
```

See [docs/docs-DocumentType-ar.md](./docs/docs-DocumentType-ar.md)


## 2026-2-15

**Update OrdersRequests And OrdersLists Api Version 2**

    - Update OrdersRequests And OrdersLists Api Version 2
    - Support Filter is_user_or_delivery_user_id  In OrdersLists And OrdersRequests Api Version 2
    - Support Permission Backend User In get OrdersRequests Api Version 2
    - Support Filter companys_id And departments_id In get OrdersRequests Api Version 2
تم تحديث جزء إدارة العروض الخاصة بطلبات التوصيل فى لوحة التحكم وفى ال api الخاص بها .


## 2026-2-16 - 2026-2-18 

**Support Filter is_has_product_options And is_has_product_options_value And product_options_values**

  -- Update Nano.Shop Product Models
  -- Update Nano.ShopApi Producs ApiController
  -- Update Nano.ShopApi Products Api Docs
تم تحديث جزء إدارة الاصناف وتم اضافة فلاتر لفلترة الاصناف حسب امتلكها لخيارات اضافية او قيم خيارات اضافية بحيث اصبح يمكن الفلترة حسب الخيارات والخصائص الاضافية للاصناف وتم تحديث جزء ال api وتم تحديث التوثيق مع اضافة امثلة توضيحية .

#### Example 3.3.2 get data Products Filter is_has_product_options or is_has_product_options_value

**يمكن فلترة الاصناف من خلال امتلكها لخيارات اضافية او لا وكذالك امتلكها لقيم خيارات اضافية ام لا من خلال الفلاتر التالية **

```json
{
  "is_has_product_options": "لجلب الاصناف حسب امتلكها لخيارات ام لا القيمة الافتراضية هي all ",
  "is_has_product_options_value": "لجلب الاصناف حسب امتلكها لخيارات وقيم للخيارات الاضافية القيمة الافتراضية هي all",
}
```

**لجلب الاصناف التي تمتلك خيارات اضافية نمرر البراميتر التالي **
```json
{
  "is_has_product_options": 1,
}
```

**لجلب الاصناف التي لا تمتلك خيارات اضافية نمرر البراميتر التالي **
```json
{
  "is_has_product_options": 0,
}
```

**لجلب الاصناف التي تمتلك خيارات اضافية وقيم للخيارات  نمرر البراميتر التالي **
```json
{
  "is_has_product_options_value": 1,
}
```

**لجلب الاصناف التي لا تمتلك خيارات اضافية وقيم للخيارات  نمرر البراميتر التالي **
```json
{
  "is_has_product_options_value": 0,
}
```

**فى المثال التالي سنقوم بجلب الاصناف التي لديها خيارات اضافية فقط وذالك من خلال الفلتر التالي **

```
GET http://localhost:8006/api/v1/shop/products?is_has_product_options_value=1
```

#### Example 3.3.3 get data Products Filter product_options_values=Black

**يمكن فلترة الاصناف من خلال امتلكها قيمة خيار اضافي كالتالي  **

```json
{
  "product_options_values": "قيمة الخيار المراد فلترة الاصناف حسبة ",
  "products_options_id": "معرف الخيار الاضافي ان وجد او يمكن تركة فارغ للبحث فى كافة الخيارات ",
}
```


**لجلب الاصناف التي قيمة احد الخيارات الاضافية لها تساوي Blackنقوم بتمرير البراميتر التالي **

```json
{
  "product_options_values": "Black",
}
```

**فى المثال التالي سنقوم بجلب الاصناف التي لديها خيارات اضافية وبشرط ان تكون قيمة الخيار الاضافي تساوي Black **

```
GET http://localhost:8006/api/v1/shop/products?product_options_values=Black
```
or
```
GET http://localhost:8006/api/v1/shop/products?product_options_values=Black&products_options_id=4,8
```


## 2026-2-18 - 2026-2-19

**Update Google Merchant Validator Version 2**

    - Support DescriptiveTextValidator Class
    - Support NoPromotionalTextValidator Class
    - Support LinkProcessor Class

تم تحديث الوحدة البرمجية الخاصة بالتحقق من صحة ملف جوجل ميرشنت حيث تم اضافة قواعد تحقق جديدة وتم تحسين الكود بشكل كبير وتم اضافة كلاسات مساعدة جديدة لتحسين التحقق من الكلمات الوصفية بشكل احترافي وبحيث يمكن التحكم باعدادات الكلاس الخاص بذالك وايضا بشكل يدعم التحقق من الكلمات الوصفية بعدة لغات 
. وتم انشاء كلاس منفصل خاص بالتحقق من الكلمات الترويجية بشكل احترافي ومرن ويدعم تعدد اللغات ايضاً. والعديد من التحسينات الاخري في الكود .

الرابط العام للاطلاع على واجهة هذه الوحده 

https://account.now-ye.com/nano2/googlemerchant/validator


## 2026-2-20 - 2026-2-23

**Nano3.Redactor – أداة تعمية البيانات الحساسة لتطبيقات نانوسوفت**

### لمحة عامة

إضافة **Nano3.Redactor** هي أداة متطورة مصممة لاكتشاف وتعمية (إخفاء) البيانات الحساسة بشكل تلقائي داخل التطبيقات المبنية على منصة **نانوسوفت للبرمجيات**. تعمل الإضافة على تحليل مختلف أنواع البيانات (نصوص، مصفوفات، كائنات) واستبدال المعلومات الخاصة مثل كلمات المرور، رموز API، البريد الإلكتروني، أرقام البطاقات الائتمانية، وغيرها بنص بديل آمن (مثل `[REDACTED]`)، وذلك قبل تسجيلها في السجلات أو عرضها أو تصديرها.

### كيف تعمل؟

تعتمد الإضافة على نظام **استراتيجيات متخصصة** (Strategies) يتم تنفيذها بترتيب أولوي محدد. كل استراتيجية مسؤولة عن نوع معين من البيانات الحساسة:

- **استراتيجية المفاتيح الآمنة** – تحافظ على المفاتيح غير الحساسة (مثل `id`, `created_at`).
- **استراتيجية المفاتيح المحظورة** – تخفي المفاتيح المعروفة مثل `password`, `token`.
- **استراتيجية الأنماط النصية** – تبحث عن أنماط مثل البريد الإلكتروني، رقم الضمان الاجتماعي، رقم البطاقة الائتمانية.
- **استراتيجية التعمية بالإنتروبيا (Shannon Entropy)** – تكشف السلاسل العشوائية عالية الإنتروبيا مثل مفاتيح API وجلسات JWT.
- **استراتيجية الكائنات الكبيرة** – تخفي المصفوفات أو الكائنات التي يتجاوز حجمها حداً معيناً.
- **استراتيجية النصوص الطويلة** – تخفي النصوص التي يتجاوز طولها حداً معيناً.

يمكن تخصيص هذه الاستراتيجيات عبر **ملفات تعريف (Profiles)** متعددة، مما يتيح سلوكاً مختلفاً للتعمية حسب السياق: ملف تعريف افتراضي (متوازن)، ملف صارم (للبيانات شديدة الحساسية)، ملف أداء (سريع مع حد أدنى من التعمية). كما تدعم الإضافة أنماط **wildcard** لتحديد المفاتيح المحظورة بشكل مرن.

### الفوائد الرئيسية

1. **حماية خصوصية البيانات** – تمنع تسرب المعلومات الحساسة عبر السجلات أو واجهات API، مما يقلل من مخاطر الاختراق والامتثال لمعايير الخصوصية مثل GDPR وPCI DSS.
2. **توفير وقت المطورين** – لا حاجة لكتابة كود تعمية يدوي لكل حقل؛ يكفي تهيئة الإضافة مرة واحدة.
3. **مرونة عالية** – يمكن تخصيص الاستراتيجيات، إضافة استراتيجيات مخصصة، وإنشاء ملفات تعريف تناسب كل بيئة (تطوير، إنتاج، تدقيق).
4. **تحسين جودة السجلات** – مع استخدام `CustomLogTap`، يتم تعمية سياق السجلات تلقائياً، مما يبقي السجلات مفيدة للتحليل دون كشف بيانات حساسة.
5. **أداة فحص ملفات** – توفر الإضافة أمر Artisan لفحص الملفات والمجلدات بحثاً عن بيانات حساسة، مما يساعد في مراجعة الأمان واكتشاف الثغرات قبل النشر.
6. **سهولة التكامل** – يمكن استخدامها عبر Facade بسيط، أو عبر خدمة الحاوية، أو بالإنشاء المباشر، وتعمل مع أي نوع بيانات (مصفوفات، كائنات Eloquent، كائنات عادية).
7. **دعم شامل للغات والتشفير** – تعمل مع النصوص العربية والترميزات المختلفة، وتدعم حساب الإنتروبيا لاكتشاف السلاسل العشوائية حتى في النصوص غير الإنجليزية.

### حالات استخدام نموذجية

- **تسجيل الأحداث (Logging)** – تعمية بيانات المستخدم الحساسة قبل كتابتها في سجل النظام.
- **تصدير البيانات (Export)** – إنشاء نسخ احتياطية أو تقارير تحتوي على بيانات منزوعة الهوية.
- **استجابات API** – إخفاء الحقول الخاصة عند إرجاع معلومات التصحيح (debug) أو بيانات المستخدم.
- **فحص أمني** – استخدام أداة فحص الملفات للبحث عن مفاتيح API أو كلمات مرور منسية في قاعدة الشيفرات.
- **الامتثال PCI** – تعمية أرقام البطاقات الائتمانية تلقائياً قبل تخزينها أو تسجيلها.

### باختصار

**Nano3.Redactor** هي الإضافة المثالية لأي مشروع يعمل على منصة **نانوسوفت للبرمجيات** ويحتاج إلى مستوى عالٍ من الأمان والخصوصية دون تعقيد. تمنحك راحة البال بأن بياناتك الحساسة لن تظهر أبداً في السجلات أو المخرجات غير المصرح بها، مع الحفاظ على سهولة الاستخدام والمرونة الكاملة.


## 2026-2-23 -2026-2-24

**Support AvailableFilterManager Class In Nano.Tags**

**Update Nano.Tags**
1.0.16:
    - Support AvailableFilterManager Class In Nano.Tags
    - Update Config available_filter shop and products
    
تم انشاء آلية جديدة خاصة بالتحكم بالفلاتر وانشائها بشكل احترافي بحيث يمكن التحكم بالفلاتر التي تعرض فى صفحة المنتجات فى المتاجر الالكترونية والتطبيقات مع انشاء توثيق كامل للكلاسات المسئولة عن ذالك .

### الخلاصة

كلاس `AvailableFilterManager` هو كلاس عام (غير مرتبط بموديل معين) مصمم لإدارة وإعداد بيانات الفلاتر (available filters) بشكل موحد. يقوم الكلاس بالمهام التالية:

- تحميل الفلاتر من مصادر متعددة: مصفوفة (Array)، نص JSON، ملف إعدادات (Config).
- تطبيع بيانات الفلاتر: إكمال الحقول المفقودة بقيم افتراضية، وتحويل الحقول النصية إلى صيغ متعددة اللغات.
- التحقق من صحة الفلاتر والتأكد من مطابقتها للـ schema المطلوبة.
- دمج مجموعات فلاتر متعددة.
- ترتيب الفلاتر حسب `sort_order`.
- تصفية الفلاتر النشطة فقط (`is_active == true`).
- البحث عن فلتر بواسطة `id`.
- إعادة ترتيب الفلاتر.
- الكشف عن التكرارات في `id` أو `name`.
- جلب الخيارات (options) للفلاتر من مصادر البيانات الديناميكية (API، قاعدة بيانات، ثابتة، دوال).
- تحويل الفلاتر إلى JSON.
- الحصول على JSON schema الخاص بالفلاتر.
- التأكد من اكتمال بياناتها وتوافقها مع الهيكل المطلوب.
- معالجتها بسهولة (دمج، ترتيب، تصفية).
- الحصول على خياراتها ديناميكياً.
- تحويلها إلى JSON للاستخدام في الـ API أو واجهات المستخدم.


See [docs/Docs-AvailableFilterManager-ar.md](./docs/Docs-AvailableFilterManager-ar.md)


## 2026-2-24 - 2026-2-25

**Support api availablefilters In TagsAPI Version 2**

**Update Nano.TagsApi**
1.0.7:
    - Support api/v1/tags/availablefilters In API Version 2.
    - Support api/v1/tags/availablefilters?filter_id In API Version 2.
    - Create tags/availablefilters Docs Api Version 2.
    - Support Relation children_count And sub_children_count In CategorieTransformer API Version 2.
    - Update tags/types Docs Api Version 2.
    - Update tags/categories Docs Api Version 2.

تم انشاء apiخاص بجلب اعدادات الفلاتر مع امكانية جلب اعدادات حقول الفلاتر مع قيم الخيارات للفلاتر التي من نوع خيارات او قائمة خيارات  وتم انشاء التوثيق الخاص بهذا الجزء بشكل كامل .

https://account.now-ye.com/api/v1/thunder/docs?method=GET&uri=api/v1/tags/availablefilters


## 2026-2-25 - 2026-2-27

**Update tags types api **

**Update Nano.TagsApi**
1.0.8:
    - Support Config NANO_TAGSAPI_TYPES_IS_STOP_CONFIG_DATA_STEP_FIELDS In tags/types Api Version 2.
    - Support Relation step_fields In TypeTransformer API Version 2.
    - Update tags/types Docs Api Version 2.

تم تحديث الجزء الخاص بالفئات الرئيسية للمواقع والمتاحر api/v1/tags/types
حيث تم تحسين طريقة ارجاع اعدادات حقول اضاقة الاصناف حيث اصبح يتعامل معها كعلاقة وتم تحسين هذا الجزء بشكل كبير وكذالك تم تحديث التوثيق بشكل كامل .

### Notes 

**ملاحظة مهمة فى الاصدارات الحديث تم ايقافة ارجاع اعدادات حقول اضافة الاصناف ضمن الحقل config_data->step_fields وبدلا من ذالك يمكن تضمين العلاقة مباشرة اذا اردنا ارجاع حقول اعدادات اضافة الاصناف **

**الاعدادات المسئولة عن ايقاف ارجاع اعدادات الحقول ضمن الحقل config_data->step_fields هى كالتالي **

```env
#القيمة الافتراضية لارجاع اعدادات حقول اضافة الاصناف او الاعلانات
NANO_TAGSAPI_TYPES_IS_CONFIG_DATA_STEP_FIELDS= true


#تم استخدمه فى الاصدارت الحديثة 
# يستخدم في حال اردنا ايقاف ارجاع اعدادات حقول اضافة الاصناف ضمن الحقل 
#لايقاف ارجاع إعدادات حقول اضافة الاصناف ضمن الحقل config_data
NANO_TAGSAPI_TYPES_IS_STOP_CONFIG_DATA_STEP_FIELDS= true

#config_data.step_fields
```

**الامثلة التالية فى الاصدار القديم **

```
Example 1.1.2.1 get List Types is_config_data_step_fields=1
Example 1.1.3.1 get List Types is_config_data_step_fields=0
Example 1.1.4.1 get List Types exclude=config_data
```

**الامثلة التالية فى الاصدار الحديث بالاعتماد على تضمين العلاقة include=step_fields **

```
Example 1.1.2.2 get List Types include=step_fields
Example 1.1.3.2 get List Types exclude=step_fields
Example 1.1.4.2 get List Types include=step_fields and exclude=config_data
```

## 2026-2-24 - 2026-2-28

** إضافة مدير الحقول `FormFieldsManager` وتوفير API متكامل لجلب الحقول المتاحة ** 

---

**نص التحديث الكامل:**
إصدار تحديثين رئيسيين لوحدات `Nano.Tags` و `Nano.TagsApi`، يهدفان إلى تحسين إدارة حقول النماذج وتوحيد طريقة التعامل معها عبر جميع برمجيات الشركة، بالإضافة إلى توفير واجهة برمجية (API) متكاملة لجلب الحقول المتاحة ديناميكياً.

---

### **أولاً: تحديث وحدة Nano.Tags (الإصدار 1.0.17)**

#### **1. إضافة كلاس FormFieldsManager**  
تم تطوير كلاس عام مستقل (`Nano\Tags\Classes\FormFieldsManager`) يعمل كمركز موحد لإدارة بيانات حقول النماذج. يقوم الكلاس بالمهام التالية:  

- **تحميل الحقول من مصادر متعددة**  
  دعم تحميل الحقول من مصفوفة PHP، نص JSON، أو ملف إعدادات (config) مع توحيد الناتج النهائي.  
- **تطبيع الحقول (Normalization)**  
  إكمال القيم المفقودة وفق هيكل افتراضي، وتحويل النصوص إلى صيغ متعددة اللغات تلقائياً بناءً على اللغات المدعومة (`ar`, `en`, `fr`).  
- **التحقق من الصحة (Validation)**  
  التحقق من اكتمال البيانات المطلوبة وصحة أنواع الحقول والقيم المسموح بها.  
- **معالجة متقدمة للحقول**  
  دمج مجموعات حقول، ترتيبها حسب `sort_order`، البحث عن حقل بواسطة `id`، واكتشاف التكرارات في `id` أو `name`.  
- **إدارة الخيارات الديناميكية (Options)**  
  جلب الخيارات من مصادر خارجية مثل APIs، قاعدة البيانات (Eloquent Models)، بيانات ثابتة، أو دوال PHP مخصصة.  
- **التحويل إلى JSON**  
  تحويل الحقول إلى JSON (عادي أو منسق) وإرجاع JSON schema الخاص بها.  

#### **2. إضافة Trait مساعد: SettingsFormFields**  
تم إنشاء Trait باسم `SettingsFormFields` يمكن استخدامه في أي كلاس (خاص بالإعدادات أو غيره) لتمكينه من الاستفادة من وظائف `FormFieldsManager` بسهولة، مما يعزز إعادة استخدام الكود ويقلل من التكرار.

#### **3. تحديث ملفات الإعدادات (Config)**  
تم تحديث ملف `config/nano/tags/form_fields.php` ليشمل حقولاً نموذجية جاهزة للمتجر (`shop`) والمنتجات (`products`) مع تغطية شاملة لأنواع الحقول المختلفة (نصوص، قوائم، تواريخ، أقسام، إلخ)، مما يوفر نموذجاً عملياً للمطورين.

---

### **ثانياً: تحديث وحدة Nano.TagsApi (الإصدار 1.0.9)**

#### **1. إضافة نقطة نهاية API جديدة**  
تم توفير نقطة نهاية (endpoint) في الإصدار الثاني من الـ API الخاص ببرمجيات نانوسوفت:  
```
api/v1/tags/availableformfields
```
تعيد هذه النقطة مصفوفة من الحقول المتاحة بعد تطبيق جميع قواعد التطبيع والتحقق، لاستخدامها مباشرة في التطبيقات الأمامية (Frontend) مثل تطبيقات الجوال أو واجهات Vue/React.

#### **2. دعم التصفية بواسطة filter_id**  
يمكن استدعاء الـ API مع الباراميتر `?filter_id` لجلب حقل واحد محدد فقط، مما يتيح تحديث جزء معين من النموذج دون الحاجة لتحميل جميع الحقول مرة أخرى.

#### **3. إنشاء توثيق شامل للـ API**  
تم إعداد توثيق مفصل بالإصدار الثاني من API الخاص بالحقول المتاحة، يشمل شرحاً لطريقة الاستخدام، أمثلة على الطلبات والاستجابات، وقائمة بجميع أنواع الحقول المدعومة. هذا التوثيق متاح الآن لفريق التطوير وسيتم نشره في بوابة المطورين الداخلية قريباً.

---

### **الفوائد المتوقعة والأثر على سير العمل:**

- **توحيد معالجة الحقول**  
  الاعتماد على `FormFieldsManager` في جميع الوحدات يضمن اتساق البيانات وسهولة الصيانة وتقليل الأخطاء.  
- **تسريع تطوير النماذج المعقدة**  
  إمكانية جلب الخيارات ديناميكياً من مصادر خارجية تسمح ببناء نماذج محدثة تلقائياً دون تعديل الكود.  
- **تحسين تجربة المطورين**  
  وجود Trait مساعد وتوثيق واضح يقلل من منحنى التعلم ويسرع دمج الميزات الجديدة.  
- **مرونة أكبر للواجهات الأمامية**  
  مع API جديد، يمكن للتطبيقات الأمامية طلب الحوامل وعرضها حسب السياق، مما يعزز التفاعلية ويحسن تجربة المستخدم.

---

يمثل هذا التحديث خطوة استراتيجية نحو بناء نظام موحد وقوي لإدارة حقول النماذج في جميع برمجيات نانوسوفت، مما يسهم في رفع جودة المنتجات وتسريع وتيرة التطوير. 


See [docs/Docs-FormFieldsManager-ar.md](./docs/Docs-FormFieldsManager-ar.md)


