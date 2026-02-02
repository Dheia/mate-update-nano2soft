# Update 2025-10

**تحديثات شهر أكتوبر 2025**

## 2025-10-1 – 2025-10-6 Support Blocked Api And Docs
تم اضاقة آلية الحظر الجديدة فى ال api بحيث يمكن من خلال ال api عمل حظر لاى كائن فى النظام من قبال المستخدمين سوى كان حظر دائم او حظر مؤقت مع امكانية الاطلاع على الكائنات التي قام بحظرها والغاء الحظر عنها . وتم عمل التوثيق الخاص بهذه الوحدة مع عدة امثله لكل حالة من الحالات . يمكن مشاهدة الفيديو والصور للتوضيح .

## 2025-10-7 – 2025-10-10 Support Public Docs Link And Api Collection JSON In Api Routes And Docs
تم تحديث الجزء الخاص بالتعامل مع توثيق ال api حيث تم تحسين هذ الجزء بشكل كبير واصبح بالامكان تحميل ملف collection_api.json لاي جزء فى النظام كما يمكن نسخ رابط الملف بشكل مباشر لاستخدامة فى وكلا الذكاء الاصطناعي بدون تحميل الملف هذه التحديثات تساعد المطورين بشكل كبير فى التعامل ال api . لتفعيل هذه الميزات يجب ان تكون الاعدادات التالية مفعلة .

```env
NANO_ROUTESBROWSER_IS_ALLOW_PUBLIC_URL= true
NANO_ROUTESBROWSER_IS_ALLOW_DOCS_IN_THUNDER= true
```

كما اصبح بالامكان استخدام اداة اختبار ال api من خلال محرر الاكواد Vs Code عن طريق تحميل ملف collection json واستخدامة مع ألاداة Thunder Client
رابط شرح استخدام الاداة:
```
https://docs.thunderclient.com
```
يمكن مشاهدة الفيديو للتوضيح .

## 2025-10-11 – 2025-10-12 Support Report Widgets OPCACHE MEMORY USAGE
تحديثات متنوعة فى نظام إدارة الكاش والذكرة المؤقتة حيث تم اضاقة اداة لمعرفة نسبة الكاش المستخدمة فى السيرفر بشكل مفصل كتقرير مع امكانية حذفها وتفريغها حيث تكمن اهمية هذه الاداة فى مرحلة التطوير .

## 2025-10-12 – 2025-10-13 Fixed the problem of not being able to add new items to the cart due to the image feature
تحديثات فى جزء السلة واصلاح مشكلة عدم قدرة اضافة الاصناف الجديدة الى السلة بسبب خاصية الصور حيث تم تحديث الكلاس العام الخاص معالجة الصور فى ال api ApiHelper.

## 2025-10-13 – 2025-10-14 Update Orders Types Support Interactions Reviews comments Favorites
- Support Reviews Api Version 2
- Support Comments Api Version 2
- Support Favorites Api Version 2
- Support Likes or DisLikes Api Version 2
- Support Bookmarks Api Version 2
- Support Reactions Api Version 2

تم تحديث جزء انواع الطلبات لتدعم التفاعل معها بشكل مباشر كاتقييمها او التعليق عليها او الاعجاب بها وغيرها .

## 2025-10-14 – 2025-10-16 Update RTL Style in All Version
- تحديثات متنوعه فى جزء الوحدة الخاصة بدعم تنسيقات اللغة العربية حيث اصبحة تدعم الاصدار الاول والثاني والثالث والرابع . وتم تحديث انشاء ال slug باللغة العربية فى جزء المدونة . كما تم تحسين انشاء وتوليد الروابط بحسب اللغة المحددة .

## 2025-10-17 – 2025-10-19 Update SitemapHelpers in Seo
- Support extendSitemapFromModelToStormInItrms in SitemapHelpers class
- Support extendSitemapFromImagesToStormInItrms in SitemapHelpers class
- Support extendSitemapFromVideosToStormInItrms in SitemapHelpers class

تم تحديث جزء توليد خرائط الموقع بشكل ديناميكي حيث اصبح بالامكان حقن خرائط الموقع باى كائن مع تمرير نوع الخريطة وخصائصها لكي يقوم النظام بانشاء الخريطة لكافة صفحات الكائن بكل سهولة .

## 2025-10-20 – 2025-10-22 Update Cards And DBQueueManager
- Support UniqueIdGenerator class In Cards Manager
- Update clearCache In All Models Cards
- Support Filters in Job

تحديثات متنوعه تخص جزء إدارة كروت الشحن حيث تم تحسين جزء توليد ارقام كروت الشحن وتم معالجة مسئلة الكاش فى هذا الجزء . وتم تحديث جزء إدارة المهام حيث تم اضافة فلاتر الى جزء المهام وتم اضافة زر لتشغيل طابور المهام وزر اخر لفحص هل يعمل طابور المهام ام لا وغيرها من التحسينات الاخرى .

## 2025-10-23 – 2025-10-24 Improved name matching verification algorithm to avoid duplicate records
- Support normalizeArabicText
- Support splitNameToSyllables
- Support getSyllableCount
- Support scopeHasSyllableCount
- Support scopeHasCharCount
- Support scopeSearchFullName

تحسين خوارزمية التحقق من مطابقة الاسماء لتجنب تكرار السجلات فى بيانات الطلاب والعملاء والموصلين والموردين والموظفين والمستخدمين وغيرها ...

## 2025-10-25 Solve the problem of the Delivery not being linked to a user account and add advanced filters to filter Deliverys based on account data
حل مشكلة عدم ارتباط الموصل بحساب مستخدم واضافة فلاتر متقدمة لفلترة الموصلين حسب بيانات الحساب

## 2025-10-26 – 2025-10-28 Support HtmlOptimizer In SEO
تم تحديث جزء تحسين السيو لمعالجة وتحسين محتوي صفحات الموقع بشكل ديناميكي مع امكانية التحكم بتفعيل او ايقاف الميزات المضافة من خلال الاعدادات

1. **تحويل الأنماط المضمنة إلى فئات inlineCss**

```php
// تحويل style="color: red;" إلى class="page_speed_123456"
$optimizedHtml = HtmlOptimizer::inlineCss($html);
```

2. **إضافة وسوم DNS Prefetch**

```php
// إضافة وسوم prefetch للنطاقات الخارجية
$optimizedHtml = HtmlOptimizer::insertDnsPrefetch($html);
```

3. **ضغط المسافات البيضاء**

```php
// إزالة المسافات الزائدة والأسطر الفارغة collapseWhitespace
$optimizedHtml = HtmlOptimizer::collapseWhitespace($html);
```

4. **إزالة الوسوم الفارغة removeAllEmptyTags**

```php
// إزالة وسوم script و style الفارغة
$optimizedHtml = HtmlOptimizer::removeAllEmptyTags($html);
```

5. **تبسيط السمات elideAttributes**

```php
// تبسيط سمات HTML مثل method="get"
$optimizedHtml = HtmlOptimizer::elideAttributes($html);
```

**الفوائد العملية من التحديث:**
- تحسين سرعة الموقع: تقليل وقت التحميل 1-3 ثواني
- تحسين تجربة المستخدم: صفحات تستجيب بسرعة
- تحسين SEO: صفحات أسرع = ترتيب أفضل في البحث
- توفير النطاق الترددي: تقليل حجم البيانات المتبادلة

## 2025-10-29 – 2025-10-31 Update LogManager V2
- Support LogsFiles
- Support LogsOverview And Reports
- Support LogsBrowser

تم تحديث جزء تصفح ملفات الاحداث والاخطاء بطريقة اكثر احترافية وتم مراعات مسائل الإداء والسرعة فى معالجة ملفات الاخطاء مع دعم كامل لفلترة الاحداث والاخطاء فى ملفات ال log مع اضافة احصائيات بالاخطاء والاحداث حسب نوعها وخلال فترات معينه من جميع ملفات الاحداث .log فى النظام .