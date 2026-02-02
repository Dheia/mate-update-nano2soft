# Update 2025-12

**تحديثات شهر ديسمبر 2025**

## 2025-12-1 – 2025-12-2 Support Is Normalize metaData In Cart Api
تم تحديث ال api الخاص بالسلة وتم اضافة خيار فى جزء إعدادات السلة للتحكم بشكل بيانات ال metaData ضمن السلة بالاضافة الى تحسينات عديدة فى جزء التعامل مع السلة .وكذالك تم تحديث ال api الخاص بإدارة العروض المقدمة للطلبات حيث تم اصلاح بعض المشاكل واضافة فلاتر لفلترة العروض حسب الحالة وغيرها . يمكن مشاهدة الفيديو والصور للتوضيح .

## 2025-12-2 – 2025-12-4 Support theme settings when activating multiple stores
دعم خاصية تعدد المتاجر فى جزء الاعدادات العامة فى ال api بحيث يتم ارجاع اسم وشعار وخصائص المتجر النشط فى ال api الخاص بالاعدادات وبذالك يكون لكل متجر اعداداته الخاصة بالنسبه للموقع .
ال api المتاثر هو
```
api/v1/basic/settings
```
لتفعيل هذا الجزء يجب ان يكون الخيار التالي مفعل فى الاعدادات علما ان القيمة الافتراضية لهذا الخيار هى false

```env
#ارجاع الاعداتد بحسب المتجر  يتم تفعيل هذه الختصية فى حال كان لكل متجر ثيم مختلف
NANO_BASICAPI_SETTINGS_IS_ALLOW_MULTI_SHOP=false
```

## 2025-12-5 – 2025-12-6 Support relationships favorites and likes or dislikes and bookmark and reaction in Reviews
- Support relationships favorites and likes or dislikes and bookmark and reaction in Reviews
- Support relationships favorites and likes or dislikes and bookmark and reaction in Reviews Api Version 2 .

تم تحديث جزء التقييمات حيث تم اضافة امكانية التفاعل مع التقييمات بالايكات وغيرها وتم اضافة api خاص بجلب تقييم المستخدم الحالي لكائن معين وغيرها وتم تحديث التوثيق مع اضافة امثلة على ذالك . يمكن مشاهدة الفيديو للتوضيح .

**في ما يلي توضيح لبعض البراميترلرت التي يمكن استخدامها لفلترة التقييمات عند جلبها بحسب المستخدم الحالي**

```json
{
  "isFavorites": "لجلب التقييمات فى قائمة تفضيلات المستخدم الحالي ",
  "isLikes": "لجلب التقييمات فى قائمة اعجابات المستخدم الحالي ",
  "isBookmarks": "لجلب التقييمات فى قائمة محفوظات المستخدم الحالي ",
  "isReactions": "لجلب التقييمات فى قائمة تفاعلات المستخدم الحالي "
}
```

وتم اضافة العلاقات التالية

| Relation Name              | Type      | Description                                      |
| -------------------------- | --------- | ------------------------------------------------ |
| `favorites_count`          | `integer` | The get count favorites default value false      |
| `user_is_favorite`         | `hasOne`  | The get has user favorite default value true     |
| `user_object_favorite`     | `hasOne`  | The get user object favorite data                |
| `likes_count`              | `integer` | The get count like default value false           |
| `dislikes_count`           | `integer` | The get count dislike default value false        |
| `user_is_like`             | `hasOne`  | The get has user like default value false        |
| `user_is_dislike`          | `hasOne`  | The get has user dislike default value false     |
| `user_object_like`         | `hasOne`  | The get user object like data                    |
| `user_is_bookmark`         | `hasOne`  | The get has user like default value false        |
| `bookmarks_count`          | `integer` | The get count bookmarks default value false      |
| `user_object_bookmark`     | `hasOne`  | The get user object bookmark data                |
| `user_is_reaction`         | `hasOne`  | The get has user reaction default value false    |
| `reactions_count`          | `integer` | The get count reaction default value false       |
| `user_object_reaction`     | `hasOne`  | The get user object reaction data                |

## 2025-11-1 – 2025-12-12 Support ApiHiddenFieldsManager
1.1.1:
    - Support ApiHelper Class.
1.1.2:
    - Support ErrorHandler And ExceptionHandler In API.
1.1.3:
    - Support DynamicInclude In Transformer API.
    - Support NestedChildrenTransformer.
1.1.4:
    - Support FractalHelper In API.
1.1.5:
    - Support withFractalInputBag In API Controller.
    - Support hasFractalInputBag In API Controller.
    - Support getFractalInputBag In API Controller.
    - Support clearFractalInputBag In API Controller.
    - Support fractalizeInputBag In API Controller.
1.1.6:
    - Support prepareStatusCode In API Controller.
    - Support invalidStatusCode In API Controller.
1.1.7:
    - Support withForceArrayOutput In API Controller.
    - Support withForceContentOutput In API Controller.
    - Support withData In API Controller.
    - Support withInput In API Controller.
1.1.8:
    - Support cached And cachedTags In API Controller.
    - Support withForceInvalidateCache And withCacheInvalidate And getCacheInvalidate In API Controller.
    - Support withOtherHashDataAnd withAllowedHashData And getHashedPayload In API Controller.
    - Support cached In API Controller.
1.1.9:
    - Support FiltersHiddenFields In API Controller.
    - Support FiltersHiddenFields In API Transformer.
1.1.10:
    - Support ApiHiddenFieldsManager.

تم تحديث الوحدة البرمجية الخاصة بال api بشكل كامل لتشمل التحديثات العديد من الجوانب كالتحكم بالحقول الراجعة فى كل api والتحكم بالتوثيق لكل جزء بشكل اكثر احترافية وبحيث يمكن تخصيص ال api حسب الحاجة مثلا بالامكان عمل نسخة خاصة بالتدريب تحتوي على اجزاء معينه فقط وايضا تحتوي هذه الاجزء على حقول وخصائص معينه فقط وذالك من اجل الحماية وبدل من تدريب المبرمجين الجدد على كامل ال api يتم تدريبهم على جزء مخصص فقط . وشمل التحديث ايضا جزء التوثيق الخاص بال api بحيث يتم اعادة تشكيل التوثيق بناء على الاعدادات بدلاً من اعادة كتابة التوثيق لكل جزء .

## 2025-12-13 Update Import User
تم تحديث واجهات استيراد وتصدير المستخدمين حيث تم اضاقة خصائص وفلاتر لواجهة عملية استيراد المستخدمين بحيث اصبح بالامكان تحديد عدد السجلات المراد استيرادها بعدة طرق واصبح بالامكان تفعيل وضع الاختبار لاختبار وتجربة عملية الاستيراد قبل التنفيذ الفعلي للعملية .

## 2025-12-14 Support line.sa whatsapp channel
تم اضافة مزود جديد لخدمة رسائل الواتساب فى النظام باسم line.sa التابع لشركة سعودية.

## 2025-12-14 – 2025-12-16 Update Sms Notify And Support SmsSaudi SMS Channel
- Update Sms Notify
- Update Sms Channel Model
- Support applyConfigDataToAttributes to Channel Model
- Support SmsSaudi SMS Channel

تم تحديث الوحدة الخاصة بالتعامل مع الرسائل النصية لتدعم مختلف اصدارات النظام من الاصدار الاول الى الاصدار الرابع . وتم اضافة مزود رسائل جديد تابع لشركة SmsSaudi .

## 2025-12-19 Support Translation Propertiers Producs
دعم خاصية تعدد اللغات للخاصية خصائص الصنف فى الواجهات وفى ال api وفى المواقع حيث تستخدم هذه الخصائص لاضافة تفاصيل او نقاط الى الاصناف او الخدمات لكي يتم عرضها مع تفاصيل الصنف او الخدمة بشكل مناسب .

## 2025-12-20 Update Nano.AuthApi Version V2+V4
- Updating the Nano.AuthApi module with significant code improvements and resolving numerous issues, such as standardizing response statuses for cases like an inactive account (403) or unauthenticated access (401), among others.
- Fixing issues in account creation, such as redirecting the user to the account activation code entry interface and incorrect phone number entry.

تحديث الواحدة البرمجية Nano.AuthApi وتحسين الكود بشكل كبير ومعالج العديد من المشاكل كتوحيد حالة الطلب الراجعة بسبب عدم تنشيط الحساب 403 او بسبب عدم المصادقة 401 وما الى ذالك . وتم إصلاح مشاكل فى انشاء الحساب كنقل المستخدم الى واجهة ادخال كود تنشيط االحساب ورقم الهاتف غير صحيح وغيرها من التحسينات الاخرى .

## 2025-12-6 – 2025-12-21 Support Fields Selection Manager
تحديثات فى جزء ال api حيث تم دعم خاصية تحديد الحقول المطلوبة بعدة طرق كالتالي

✅ الطريقة الأولى (Bracket Notation):

```
api/v1/shop/products?include=department(id,name)
```

✅ الطريقة الثانية (Fieldsets):

```
api/v1/shop/products?include=department&fields[department]=id,name
```

✅ الطريقة الثالثة (Dot Notation):

```
api/v1/shop/products?include=department.id,department.name
```

## 2025-12-20 – 2025-12-23 Update: Bulk SMS Messaging via Contact File Import

**Feature Overview:**
The messaging module has been enhanced to support sending SMS messages to multiple recipients simultaneously by importing contact lists directly from external files. This feature streamlines large-scale communication and eliminates the need for manual entry.

**Supported File Formats:**
You can now import contacts from the following file types:
- CSV (Comma-Separated Values)
- XLS / XLSX (Microsoft Excel formats)
- VCF (vCard contact files)
- AUTO (automatically detected format, typically structured text or system-specific lists)

**Key Benefits:**
- Time Efficiency: Reach hundreds of contacts in minutes.
- Accuracy: Reduce errors from manual data entry.
- Flexibility: Compatible with popular contact management tools (Excel, Outlook, etc.).
- Personalization: Support for merge tags to customize each message.

**Use Cases:**
- Marketing campaigns and promotional alerts.
- Event reminders or scheduling updates.
- Notifications for organizations, schools, or community groups.

**Notes:**
- Ensure phone numbers are correctly formatted before import.
- The system validates numbers and provides error logs for failed entries.
- Supports scheduling for future delivery.

This update significantly enhances the module’s scalability and usability for team-based or mass communication needs.

## 2025-12-24 – 2025-12-25 Update to Nano.SmsNotify Module – Scheduled and Monitored Messaging System

**ملخص التحديث**

تم تطوير وتحسين آلية إرسال الرسائل النصية (...,WhatsApp, SMS) من خلال نقل عملية الإرسال إلى نظام معالجة في الخلفية (Background Job) حيث تم استبدال النمط المتزامن التقليدي بنظام يعتمد على الوظائف الخلفية (Background Jobs)، مما يمكن من معالجة الرسائل بشكل غير متزامن ومُجدول.
وقد رافق هذا التحول تطوير لوحة مراقبة آنية تتيح تتبع تقدم عمليات الإرسال وعرض مؤشرات أداء دقيقة تشمل: عدد الرسائل المرسلة والفاشلة، ونسبة الإنجاز، والحالة التشغيلية الحالية.

كما تم تعزيز النظام بآلية ذكية لإعادة المحاولة، تدعم تخصيص حجم الدفعات، والفترات الزمنية بين المحاولات، وعدد مرات إعادة الإرسال، مما يرفع مستوى الموثوقية ويقلل من نسبة الفقد.
وتتميز الواجهة الجديدة بوظائف تحكم ديناميكية تشمل: الإيقاف المؤقت، الاستئناف، الإلغاء، وإعادة المحاولة، مما يتيح إدارة مرنة ومباشرة للحملات النشطة.

النتيجة: تحسين أداء النظام، ورفع قدرته على التعامل مع الأحمال الكبيرة، وتوفير رؤية شاملة وعمليات إدارة فعّالة لحملات الرسائل النصية الجماعية.

---

**الميزات والتغييرات الرئيسية**

1. **التحول إلى نظام إرسال غير متزامن (Asynchronous Processing)**
   - تم استبدال الإرسال المباشر Synchronous بـ Background Job باستخدام نظام الصفوف (Queues).
   - يدعم النظام إرسال دفعات (Batches) من الرسائل مع إمكانية تحديد حجم الدفعة.
   - يحسن استجابة النظام ويقلل من تأثير الإرسال الجماعي على أداء التطبيق.

2. **نظام مراقبة وإحصائيات في الوقت الفعلي**
   - إضافة لوحة عرض إحصائية تعرض:
     - إجمالي المستلمين والرسائل المرسلة والفاشلة.
     - نسبة الإنجاز (Completion Percentage).
     - الحالة الحالية (مثل: قيد المعالجة، مكتمل، متوقف).
     - وقت وتاريخ بدء الإرسال.
   - تحديث البيانات ديناميكيًا دون الحاجة إلى إعادة تحميل الصفحة.

3. **آلية متقدمة لإعادة المحاولة (Retry Mechanism)**
   - إمكانية إعادة إرسال الرسائل الفاشلة تلقائيًا.
   - قابلية ضبط:
     - حجم الدفعة لإعادة الإرسال.
     - التأخير بين الرسائل (بالثواني).
     - التأخير قبل بدء العملية.
     - الحد الأقصى لعدد المحاولات لكل رسالة.
   - تدوين سبب إعادة المحاولة لأغراض التدقيق والتتبع.

4. **واجهة تحكم ديناميكية في عملية الإرسال**
   - أزرار تحكم تسمح بإدارة العملية أثناء التنفيذ:
     - إيقاف مؤقت / استئناف.
     - إلغاء العملية بالكامل.
     - إعادة محاولة للرسائل الفاشلة.
     - تحديث حالة الإرسال يدويًا.

5. **تحسين إدارة الحالات (State Management)**
   - دعم حالات متعددة ومعرّفة بوضوح:
     - قيد المعالجة
     - مكتمل
     - مكتمل مع أخطاء
     - فاشل
     - متوقف مؤقتًا
   - كل حالة توفر إمكانيات تحكم مختلفة حسب سياق العملية.

---

**الفوائد التقنية**
- زيادة الموثوقية: تقليل فقدان الرسائل عبر آلية إعادة المحاولة.
- تحسين الأداء: عدم حجب واجهة المستخدم أثناء الإرسال.
- قابلية التوسع: إرسال آلاف الرسائل بكفاءة.
- سهولة الصيانة: فصل منطق الإرسال عن الطبقة التقديمية.
- تتبع أفضل: تسجيل كامل لسير العملية والأخطاء.

---

**كيفية الاستخدام**
1. إنشاء حملة إرسال جديدة.
2. تحديد المستلمين والمحتوى.
3. بدء الإرسال الذي يعمل في الخلفية.
4. مراقبة التقدم عبر لوحة الإحصائيات.
5. التحكم في العملية باستخدام الأزرار المتاحة.

---

**ملاحظات هامة**
- النظام يدعم Idempotency لمنع إرسال مكرر.
- يمكن تكامل النظام مع أنظمة التنبيهات (Notifications) للإخطار بحالة الإرسال.
- تم تحسين تسجيل الأحداث (Logging) لسهولة التحليل والتدقيق.

---

يعمل هذا التحديث على تحويل وحدة الإرسال من عملية مباشرة إلى نظام إرسال ذكي، خاضع للمراقبة، وقابل للإدارة، مما يلبي احتياجات الحملات الكبيرة والمعقدة في بيئة الإنتاج.

## 2025-9-17 – 2025-12-29 Support Json Editor Form Widgets
تم انشاء وايدجت باسم JsonEditor يمكن استخدمها فى اى وحدة اخرى لدعم JSON Editor لتمكين إنشاء نماذج إدخال بيانات ديناميكية ومرنة تعتمد على مخططات JSON Schema، مما يسمح بتوليد واجهات مستخدم غنية تلقائياً، ويوفر تحققاً قوياً من البيانات في الوقت الفعلي، ويقلل بشكل كبير من وقت التطوير والحاجة للكود الثابت، مع تقديم تجربة مستخدم محسنة وموحدة.

**تحديث: دمج مكتبة JSON Editor لإنشاء نماذج ديناميكية احترافية**

يتمحور هذا التحديث حول دمج مكتبة JSON Editor الشهيرة كأداة أساسية (Widget) داخل النظام ضمن الوحدة البرمجية Nano.JsonDB. هذا ليس مجرد محرر نصوص لـ JSON، بل هو محرر نماذج يعتمد على Schema، مما يفتح آفاقًا جديدة تمامًا لإنشاء واجهات إدخال بيانات معقدة ومرنة.

**الفكرة الأساسية:**

بدلاً من كتابة كود HTML/JS ثابت لكل نموذج، يمكنك الآن تعريف مخطط بيانات (JSON Schema) يصف هيكل البيانات المطلوب (الحقول، أنواعها، شروطها). المكتبة ستولد تلقائيًا واجهة مستخدم كاملة ونظيفة وديناميكية بناءً على هذا المخطط.

**المزايا الرئيسية لهذا الدمج:**

1. **النماذج الديناميكية بالكامل:**
   - يمكن تغيير هيكل النموذج (إضافة/إزالة حقول) بتعديل ملف schema فقط، دون الحاجة لتغيير كود الواجهة الأمامية أو الخلفية في معظم الحالات.
   - مثالي للإعدادات الديناميكية، والسمات المخصصة (Custom Attributes)، والتكوينات المعقدة.

2. **التحقق من الصحة المدمج (Built-in Validation):**
   - تستخدم المكتبة JSON Schema القياسي للتحقق من صحة البيانات. يمكن تحديد الحقول الإجبارية، الأنماط (مثل نمط البريد الإلكتروني أو الرابط)، النطاقات الرقمية، طول النصوص، وغيرها.
   - تظهر أخطاء التحقق للمستخدم فوريًا وبدقة عالية.

3. **واجهة مستخدم غنية وقابلة للتخصيص:**
   - تقدم المكتبة مجموعة واسعة من أنواع الحقول المتقدمة أكثر من مجرد input و textarea:
     - select مع خيارات ديناميكية.
     - array (قوائم) مع إمكانية إضافة/حذف/إعادة ترتيب العناصر بسهولة.
     - object متداخل (مثل شجرة الخصائص).
     - مشغلات (toggles)، منتقي ألوان، منتقي تواريخ، إلخ.
   - يمكن تخصيص مظهر كل حقل (مثل استخدام textarea بدلاً من input للنص الطويل).

4. **كفاءة في التطوير:**
   - يقلل الكود المكرر: لا حاجة لكتابة HTML/JS منفصل لكل نموذج.
   - الفصل بين الواجهة والمنطق: يعمل فريق المطورين الخلفيين (Backend) على تعريف schema البيانات، وواجهة المستخدم تتولد تلقائيًا وبشكل متسق.
   - سهولة الصيانة: تغيير قاعدة البيانات أو قواعد التحقق ينعكس تلقائيًا على الواجهة إذا تم تحديث الـ schema.

5. **تجربة مستخدم محسنة:**
   - النماذج المولدة نظيفة وسهلة الاستخدام.
   - التوجيه والتلميحات: إمكانية إضافة وصف (description) لكل حقل يظهر للمساعدة.
   - ترتيب منطقي: الحقول تظهر بالترتيب المحدد في المخطط.

**أمثلة على حالات الاستخدام داخل المنصة:**
- لوحة إعدادات وحدة معقدة: بدلاً من 20 إعدادًا ثابتًا، يمكن إنشاء schema واحد يولد واجهة قابلة للطي والتنظيم.
- منشئ محتوى ديناميكي: لمحرري المحتوى لتعريف كتل محتوى ذات هياكل متغيرة (مثل صفحة منتج بها مواصفات تقنية قابلة للتخصيص).
- نموذج تسجيل بيانات مرن: لجمع بيانات تختلف من حملة لأخرى دون إعادة برمجة.
- واجهة إدخال لبيانات API: لتكوين طلبات API معقدة بشكل مرئي وآمن.

**الخلاصة:**

دمج مكتبة JSON Editor ليس مجرد "ميزة إضافية"، بل هو تحول في فلسفة بناء واجهات إدخال البيانات. فهو يحول عملية الإنشاء من الترميز اليدوي الثابت إلى التصميع الديناميكي القائم على البيانات والمخططات (Schema-Driven). هذا يمنح النظام مرونة هائلة، ويقلل وقت التطوير، ويضمن جودة واتساق واجهات المستخدم، مما يخدم كلًا من المطورين والمستخدمين النهائيين على حد سواء.

## 2025-12-26 – 2025-12-30 Update AuthApi And SSO And Support Build Socialite Providers Icon
- Update Nano.AuthApi
- Update SSO Socialite Providers
- Support Build Socialite Providers Icon

تم تحديث الواحدة البرمجية الخاصة بنظام المصادقة Nano.AuthApi وتم اضافة ميزات جديدة الى الكود كما تم دعم العلاقة التالية ضمن ملف بروفايل المستخدمين `shopping_preferences` تختص بتفضيلات المستخدم المتعلقة بالتسوق كالمدينة ونوع الطلب والمتجر وغيرها حيث تفيد فى تحديد سلوك الواجهات بالنسبة للمستخدم . وتم تحديث الواحدة البرمجية الخاصة بتسجيل الدخول من خلال وسائل التواصل الاجتماعي SSO . وتم انشاء إداة لتوليد ايقونات المزودين باكثر من 23 تنسبق لكل ايقونه مع امكانية تحميل ملف مضغوط مباشرة بالايقونات حسب تخصيص المستخدم .

رابط ادات توليد الايقونات التي تم انشائها:
```
https://account.now-ye.com/sso/build-icon
```

مرفق ملفات مضغوطة بالايقونات التي تم توليدها بإكثر من 23 تنسيق لكل إيقونة بالصيغتين png and svg.