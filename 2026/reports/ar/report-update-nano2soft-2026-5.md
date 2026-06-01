# التقرير الشهري لإنجازات فريق نانوسوفت للبرمجيات  
## مايو 2026  

### مقدمة  
يعرض هذا التقرير ملخصًا احترافيًا ودقيقًا لإنجازات فريق التطوير في شركة نانوسوفت للبرمجيات خلال الفترة من 1 مايو إلى 31 مايو 2026. تميز هذا الشهر بإصدارات وتحديثات ضخمة غير مسبوقة، حيث تم **إطلاق 5 واجهات برمجة تطبيقات جديدة (APIs)** كاملة (Nano.StudyyearApi, Nano.SchoolApi, Nano.AbsenceApi, Nano.HomeworkApi, Nano.LocationApi)، و**تطوير 4 بوابات دفع جديدة** (BasPay, CashPay, FloosakPay، وتحديث JaibPay)، بالإضافة إلى **تحسينات جوهرية في الأنظمة المحاسبية** (Tss.Accounts 1.0.38/1.0.39 و Nano.AccountsApi 1.1.0/1.1.1)، و**إعادة هيكلة شاملة لوحدات الطلبات** (Nano.Orders 2.2.10/2.2.11 و Nano.OrdersApi 1.0.20/1.0.21)، و**تطوير نظام متكامل لإدارة الصلاحيات** (AccessManager في Nano.AuthApi)، و**تحسينات متقدمة على سلوكيات الزيارات والبلاغات** (ProposalModel, VisitModel)، ودعم الوسائط المتعددة في الأنظمة الجغرافية والمحاسبية.

تعكس هذه الإنجازات نقلة نوعية في نضج المنصة، مع الالتزام الكامل بمعايير التوافق العكسي، وتوفير توثيق شامل (أكثر من 40 ملف توثيق)، وأمثلة عملية متقدمة.

---

## ملخص تنفيذي  
خلال شهر مايو 2026، أنجز الفريق **أكثر من 25 تحديثًا رئيسيًا**، موزعة على ثمانية محاور استراتيجية:

1. **إطلاق 5 واجهات برمجة تطبيقات متكاملة (APIs)** – تغطي السنوات الدراسية، المدرسة، الحضور والغياب، الواجبات المنزلية، والموقع الجغرافي، جميعها مبنية وفق أحدث المعايير.
2. **إضافة 4 بوابات دفع جديدة** – BasPay (منصة بس)، CashPay (الدفع النقدي الإلكتروني – OTP)، FloosakPay (محفظة فلوسك)، وتحديث شامل لـ JaibPay (محفظة جيب).
3. **تطوير شامل للنظام المحاسبي** – دعم القيود متعددة الأطراف، فروق العملات، والتحقق المتقدم (Tss.Accounts 1.0.38)، ثم دعم الوسائط المتعددة لجميع السندات (1.0.39) مع تكامل API كامل (Nano.AccountsApi 1.1.0/1.1.1).
4. **إعادة هيكلة نظام الطلبات** – إضافة نطاقات فلترة مالك المنتج، دالة تحديث الحالة بصلاحيات متدرجة، علاقات جديدة (مركبة الموصل، الدولة، المدينة، المطارات)، وتوسيع إعدادات التقارير.
5. **إطلاق نظام مركزي لإدارة الصلاحيات** – كلاس AccessManager متطور يدعم نطاقات (company, department, state, own, children) وتحليلاً ديناميكيًا للطلاب وأولياء الأمور، مع فلاتر متقدمة وتحكم مركزي عبر الإعدادات.
6. **تطوير سلوكيات عامة** – نطاقات متقدمة لاستعلامات الزيارات (VisitModel) والبلاغات (ProposalModel) مع إحصائيات وأعمدة محسوبة.
7. **دعم الوسائط المتعددة** – إضافة الصور والملفات للدول والمدن والمديريات (Nano.LocationApi 1.1.0) ولجميع نماذج الحسابات (UnifiedMorphClass).
8. **تحسينات داعمة** – تحديث Tss.Homework بإضافة وحدة Feedback كاملة، تحديث Tss.Student بإعادة هيكلة دوال getRecords ودعم AccessManager وAdvancedQueryHelper، وتحديث Nano.UserPlus بأنواع حسابات ديناميكية وأعمدة جديدة لمجموعات المستخدمين.

تمت جميع التحديثات مع الحفاظ على التوافق مع الإصدارات السابقة، وإضافة توثيق شامل باللغتين العربية والإنجليزية، وتوفير واجهات اختبار تفاعلية.

---

## جداول تلخيصية لأبرز الإنجازات

### 1. الأنظمة والإضافات الرئيسية المطلقة (جديدة كلياً)

| النظام / الإضافة | الوصف | الإصدار / التاريخ | أبرز المميزات |
| :--- | :--- | :--- | :--- |
| **Nano.StudyyearApi** | API لإدارة السنوات والفصول والأشهر الدراسية | 1.0.0 (6 – 7 مايو) | 3 متحكمات (Periods, Semsters, Months)، نظام صلاحيات متعدد المستويات، محولات بيانات، دعم القيم الافتراضية (السنة الدراسية الأساسية). |
| **Nano.SchoolApi** | API لإدارة بيانات المدرسة الأساسية (14 مورداً) | 1.0.0 (8 – 10 مايو) | صفوف، مراحل، مواد، شعب، توزيعات، رسوم، وثائق، علامات، دعم صلاحيات لكل مورد، توثيق شامل. |
| **Nano.AbsenceApi** | API لإدارة الحضور والغياب والكشوفات | 1.0.0 ثم 1.1.0 (11 – 12 مايو ثم 26 – 27 مايو) | عمليات CRUD لسجلات الغياب، دعم إنشاء وتحديث عبر API، تكامل مع AccessManager، فلاتر متقدمة. |
| **Nano.HomeworkApi** | API لإدارة الواجبات والتسليمات والتصنيفات | 1.0.0 (27 – 30 مايو) | 5 متحكمات، دعم الواجبات العامة (student_id = '*')، صلاحيات مرنة، محلل ديناميكي للطلاب/أولياء الأمور. |
| **Nano.LocationApi (الإصدار 1.1.0)** | دعم الوسائط المتعددة في الموقع الجغرافي | 1.1.0 (20 – 21 مايو) | Behavior DynamicAddMediaIncludes، تضمين الصور والملفات في استجابات API، تحكم في بيانات الميتا. |
| **Nano.AuthApi (إصدارات 1.0.19–1.0.21)** | تطوير AccessManager والصلاحيات المتقدمة | 1.0.19 (15-16 مايو)، 1.0.20 (24-25 مايو)، 1.0.21 (25-26 مايو) | نقاط نهاية لخيارات الحقول، متحكم UserGroups، AccessManager مع محلل ديناميكي وفلاتر متقدمة ودعم الرجوع. |

### 2. بوابات الدفع المضافة والمحدثة (ضمن Nano.Yepayment)

| البوابة | المنطقة / الجهة | نوع الدفع | أبرز الميزات | الإصدار / التاريخ |
| :--- | :--- | :--- | :--- | :--- |
| **BasPay** | اليمن (منصة بس) | خطوتين (Two-Step) | OAuth2، توقيع مشفر AES-256-CBC، إنشاء معاملة ثم تأكيد، واجهة اختبار متكاملة. | 6 – 18 مايو |
| **CashPay** | اليمن (الدفع النقدي الإلكتروني – OTP) | خطوتين مع OTP | تشفير كلمة المرور في الهيدر، InitPayment، ConfirmPayment، تغيير كلمة المرور، واجهة اختبار. | 20 – 22 مايو |
| **FloosakPay** | اليمن (محفظة فلوسك) | خطوتين مع OTP | OAuth2، Idempotency، تسجيل PaymentLog في المرحلة الأولى، استرداد المبالغ، واجهة اختبار. | 18 – 24 مايو |
| **JaibPay (تحديث شامل)** | اليمن (محفظة جيب) | مباشر (Direct) | إصلاح المصادقة، دالة refund، تحسين معالجة الاستجابات، إصلاح حفظ الإعدادات المشفرة، واجهة اختبار. | 4 – 10 مايو |

### 3. التحديثات الرئيسية للإضافات القائمة

| الإضافة | الإصدار / التاريخ | أبرز التحديثات |
| :--- | :--- | :--- |
| **Tss.Accounts** | 1.0.38 (17 فبراير – 1 مايو)، ثم 1.0.39 (29 – 31 مايو) | 1.0.38: ترايت TransactionsMultiHelper (قيود متعددة الأطراف، فروق عملات، تحقق متقدم). 1.0.39: دعم الوسائط المتعددة لجميع نماذج الحسابات، توحيد MorphClass (UnifiedMorphClass). |
| **Nano.AccountsApi** | 1.1.0 (20 مارس – 5 مايو)، ثم 1.1.1 (29 – 31 مايو) | 1.1.0: BaseDocumentController، إضافة 5 أنواع مستندات جديدة (BondsDays, PayReceipts, DebitNotes, CreditNotes, Transfers) و CatchReceiptsV2. 1.1.1: Behavior DynamicAddMediaIncludes لتضمين الميديا في API. |
| **Nano.Orders** | 2.2.10 (6 – 8 مايو)، ثم 2.2.11 (26 – 29 مايو) | 2.2.10: HasProductOwnerScopes (فلترة حسب مالك المنتج)، updateOrderStatusAdvanced (صلاحيات متدرجة). 2.2.11: علاقات جديدة (delivery_vehicle_type, delivery_car, country, state, departuredestination, arrivaldestination)، توسيع إعدادات تقرير الطلب. |
| **Nano.OrdersApi** | 1.0.20 (6 – 8 مايو)، ثم 1.0.21 (26 – 29 مايو) | 1.0.20: نقطة API update-status. 1.0.21: دعم العلاقات الجديدة في OrderTransformer. |
| **Nano.UserPlus** | 1.1.7 (13 – 14 مايو) | أنواع حسابات ديناميكية، توافق مع RainLab.User 2.x (first_name/last_name)، أعمدة جديدة في user_groups (is_new_user_default, is_active, sort_order)، توسيع واجهة إدارة المجموعات. |
| **Tss.Student** | 1.0.13 (24 – 26 مايو) | إعادة هيكلة دوال getRecords، إضافة StudentRecordsHelper، دمج AccessManager و AdvancedQueryHelper، دوال ربط بين المستخدم والطالب/ولي الأمر. |
| **Tss.Homework** | 1.0.9 (27 – 28 مايو) | إضافة وحدة Feedback (التسليمات والتقييمات) بشكل كامل مع نموذج وواجهة إدارة وصلاحيات وقائمة جانبية. |
| **Nano.Coupons** | 2.1.1 (2 – 8 مايو) | إعادة هيكلة شرط "حد المنتج" في كلاس ProductLimitValidator، دعم سيناريوهات متقدمة (تقييد عدد الطلبات بدون منتج)، تحسين الأداء (Cache). |
| **Nano.Location** | 1.0.16 (20 – 21 مايو) | إضافة علاقات الميديا (attachOne/attachMany) لدولة، مدينة، مديرية، وتوسيع الواجهة الخلفية بحقول وأعمدة للميديا. |

### 4. تطوير سلوكيات عامة (Behaviors)

| السلوك | الإصدار / التاريخ | أبرز النطاقات والدوال الجديدة |
| :--- | :--- | :--- |
| **ProposalModel (بلاغات)** | 19 – 20 مايو | نطاقات: عدد البلاغات، أحدث/أقدم بلاغ، فلترة حسب المستخدم (target/user)، إضافة عمود is_proposed_by_user، نطاق متقدم scopeWhereHasProposalAdvanced، دوال إحصائية. |
| **VisitModel (زيارات)** | 20 – 21 مايو | نطاقات: عدد الزيارات، مجموع الزيارات، مجموع المشاهدات، أحدث/أقدم زيارة، فلترة حسب المستخدم، عمود is_visited_by_user، دوال إحصائية (getTotalVisits, getVisitorsUsers). |

---

## تفاصيل الإنجازات والتحديثات

### 1. إطلاق واجهات برمجة تطبيقات متكاملة (APIs)

#### 1.1. Nano.StudyyearApi (الإصدار 1.0.0)  
**الفترة:** 6 – 7 مايو  
**الوصف:** إنشاء واجهة برمجة تطبيقات RESTful لإدارة السنوات والفصول والأشهر الدراسية، بناءً على إضافة Tss.Studyyear.  
**المتحكمات:**  
- `Periods` (السنوات الدراسية)  
- `Semsters` (الفصول الدراسية)  
- `Months` (الأشهر الدراسية)  
**الميزات:**  
- دعم الصلاحيات لكل مورد (backend/frontend) عبر إعدادات config.php.  
- محولات بيانات (Transformers) مع تضمين العلاقات.  
- دعم القيم الافتراضية (السنة الدراسية الافتراضية عبر Period::getPrimary()).  
- دعم التخزين المؤقت والبحث النصي.  
**نقاط النهاية:**  
- `GET /api/v1/studyyear/periods` ، `GET /api/v1/studyyear/semsters` ، `GET /api/v1/studyyear/months`  
- `GET /api/v1/studyyear/periods/{id}` ونظائرها.

#### 1.2. Nano.SchoolApi (الإصدار 1.0.0)  
**الفترة:** 8 – 10 مايو  
**الوصف:** API شامل لجميع كيانات إضافة Tss.School (14 مورداً).  
**المتحكمات:**  
`Classes`, `Stages`, `Subjects`, `Groups`, `ClassGroups`, `ClassSubjects`, `TeacherSubjects`, `ClassCostCenters`, `FeesTypes`, `DocumentTypes`, `OutSchools`, `StudentDocuments`, `StudentsFees`, `LastStudentMarks`.  
**الميزات:**  
- صلاحيات منفصلة لكل مورد.  
- دعم القيمة الافتراضية للسنة الدراسية للموارد المرتبطة بها.  
- نطاق scopeExclude ديناميكي.  
- توثيق شامل مع أمثلة.

#### 1.3. Nano.AbsenceApi (الإصدار 1.0.0 ثم 1.1.0)  
**الفترة:** 11 – 12 مايو (الإصدار 1.0.0)، ثم 26 – 27 مايو (الإصدار 1.1.0)  
**الوصف:** API لإدارة الحضور والغياب وسجلات الكشوفات.  
**المتحكمات:** `Absences`, `Files`, `ClassTypes`, `ClassTypeCompanies`.  
**ميزات الإصدار 1.0.0:**  
- دعم عمليات `store` و `update` لسجلات الغياب.  
- تحقق من الصلاحيات لكل عملية.  
- قيم افتراضية ذكية (companys_id, departments_id, year_id).  
**ميزات الإصدار 1.1.0:**  
- إعادة هيكلة كاملة وفق معايير Nano-Api-SKILL.md.  
- استخدام AccessManager::checkByResource و checkWithFallback.  
- دعم الفلاتر المتقدمة (is_or, is_not, is_force, is_or_null).  
- محلل ديناميكي للطلاب وأولياء الأمور (frontend_resolver).  
- توحيد دالة getRecords مع أحداث وتخزين مؤقت.

#### 1.4. Nano.HomeworkApi (الإصدار 1.0.0)  
**الفترة:** 27 – 30 مايو  
**الوصف:** API متكامل لإدارة الواجبات المنزلية والتسليمات والتصنيفات.  
**المتحكمات:** `Homeworks`, `Feedbacks`, `Categories`, `ClassTypes`, `ClassTypeCompanies`.  
**الميزات:**  
- دعم الواجبات العامة لكل الصف (student_id = '*') أو لكل الشعبة (group_id = '*').  
- صلاحيات متعددة المستويات (backend/frontend/guest) مع إعدادات مركزية.  
- محلل ديناميكي لـ student_id و record_id للطلاب وأولياء الأمور.  
- دالة getRecords موحدة مع فلاتر متقدمة وأحداث.  
- محولات بيانات (HomeworkTransformer, FeedbackTransformer, إلخ).  
- دعم عمليات CRUD كاملة للواجبات، والتسليمات (إنشاء وتحديث فقط للـ Feedbacks).

#### 1.5. Nano.LocationApi (الإصدار 1.1.0)  
**الفترة:** 20 – 21 مايو  
**الوصف:** إضافة دعم الوسائط المتعددة إلى API الموقع الجغرافي.  
**الميزات:**  
- Behavior `DynamicAddMediaIncludes` يضيف includes ديناميكية (image, images, files, videos, audios, book_intro) إلى CountryTransformer, StateTransformer, DirectorateTransformer.  
- دعم التحكم في بيانات الميتا عبر `is_mate_data_media` عام أو خاص بكل include.  
- إضافة أعمدة وحقول الميديا في الواجهة الخلفية لـ Country, State, Directorate.  
- استخدام سمة `UnifiedMorphClass` لتوحيد polymorphic type للملفات (اختياري، لكن تم تطبيقه في الحسابات).

---

### 2. بوابات الدفع الجديدة والمحدثة

#### 2.1. BasPay (منصة بس)  
**الفترة:** 6 – 18 مايو  
**الوصف:** بوابة دفع لليمن تعمل بنمط الخطوتين (Two-Step) مع توقيع مشفر AES-256-CBC.  
**المكونات:** كلاس BasPay يمتد من PaymentProvider، دوال process (إنشاء معاملة)، complete (تأكيد)، getAuthToken (OAuth2)، generateSignature (AES-256-CBC)، checkTransactionStatus.  
**نقاط نهاية الاختبار:** `/baspay/test-auth`, `/baspay/test-create-payment`, `/baspay/test-ui`.  
**الإعدادات:** baspay_url, client_id, client_secret, app_id, merchant_key, iv.

#### 2.2. CashPay (الدفع النقدي الإلكتروني – OTP)  
**الفترة:** 20 – 22 مايو  
**الوصف:** بوابة دفع تعتمد على OTP، بنمط خطوتين مع تشفير كلمة المرور في الهيدر.  
**المكونات:** كلاس CashPay، دوال process (InitPayment)، complete (ConfirmPayment)، operationStatus، changePassword، encryptAes256Cbc.  
**نقاط نهاية الاختبار:** `/cashpay/test-auth`, `/cashpay/test-create-payment`, `/cashpay/test-confirm-payment`, `/cashpay/test-ui`.  
**الإعدادات:** cashpay_url, username, password (مشفر), encryption_key, sp_id.

#### 2.3. FloosakPay (محفظة فلوسك)  
**الفترة:** 18 – 24 مايو  
**الوصف:** بوابة دفع لليمن تعمل بنمط خطوتين مع OTP، مع آلية Idempotency وتخزين مؤقت للتوكنات.  
**المكونات:** كلاس FloosakPay، دوال process (sendPayment)، complete (confirmPayment)، reverse (refund)، getAuthToken، checkTransactionStatus.  
**الميزات:**  
- التحقق من معاملة سابقة لمنع التكرار.  
- تسجيل PaymentLog بحالة 'initiated' في المرحلة الأولى.  
- واجهة اختبار متكاملة.  
**نقاط نهاية الاختبار:** `/floosakpay/test-auth`, `/floosakpay/test-create-payment`, `/floosakpay/test-confirm-payment`, `/floosakpay/test-ui`.

#### 2.4. JaibPay (تحديث شامل)  
**الفترة:** 4 – 10 مايو  
**الوصف:** تحسينات جوهرية على بوابة JaibPay (محفظة جيب) التي تم إطلاقها في أبريل.  
**الإصلاحات والتحسينات:**  
- دالة normalizeApiResponse لتوحيد معالجة الاستجابات.  
- إصلاح حفظ الإعدادات المشفرة في PaymentGatewaySettings (تجاوز __set).  
- إضافة دالة refund لاسترداد المبالغ.  
- دالة testPortConnection لاختبار المنفذ.  
- تحسين رسائل الخطأ (مثل "كود الشراء غير صحيح").  
- واجهة اختبار محدثة.

---

### 3. التحديثات المحاسبية المتقدمة

#### 3.1. Tss.Accounts الإصدار 1.0.38  
**الفترة:** 17 فبراير – 1 مايو (تم تضمينه في تقرير مايو لأنه اكتمل في مايو)  
**الميزات الرئيسية:**  
- ترايت `TransactionsMultiHelper` لإنشاء قيود متعددة الأطراف (3+ أطراف).  
- دوال فروق العملات: `checkExchangeDifferences`, `createExchangeDifferenceEntry`, `handleExchangeDifferences`.  
- دالة `testJournalEntry` لاختبار القيد قبل الإنشاء.  
- إعدادات شاملة للتحكم (حدود المبالغ، فحص الرصيد، قواعد ديناميكية لأنواع الحسابات، إدارة paper_id).  
- تكامل مع `TransactionsHelper` بحيث تصبح دوال الترايت متاحة.

#### 3.2. Nano.AccountsApi الإصدار 1.1.0  
**الفترة:** 20 مارس – 5 مايو  
**الميزات:**  
- كلاس أساسي `BaseDocumentController` لتوحيد منطق CRUD للمستندات المالية.  
- إضافة 5 متحكمات جديدة: `BondsDays` (قيود يومية)، `PayReceipts` (سندات صرف)، `DebitNotes` (إشعارات مدين)، `CreditNotes` (إشعارات دائن)، `Transfers` (تحويلات).  
- نسخة محسنة `CatchReceiptsV2` (سندات قبض).  
- محولات (Transformers) لكل نوع.  
- توسيع إعدادات config.php والمسارات.

#### 3.3. Tss.Accounts الإصدار 1.0.39 و Nano.AccountsApi 1.1.1 (دعم الوسائط المتعددة)  
**الفترة:** 29 – 31 مايو  
**الميزات:**  
- إضافة علاقات `attachOne` و `attachMany` (image, images, files, videos, audios, book_intro) إلى جميع نماذج الحسابات (TransactionHeader, CatchReceipt, PayReceipt, BondsDay, Transfer، إلخ) باستخدام سمة `UnifiedMorphClass` لتوحيد الـ polymorphic type.  
- توسيع الواجهة الخلفية: أعمدة (image, images, files) في القوائم، وتبويب "المرفقات" في نماذج الإدارة.  
- Behavior `DynamicAddMediaIncludes` في Nano.AccountsApi لتضمين الميديا في استجابات API عبر `include=image,images,files`.  
- دعم `is_mate_data_media` للتحكم في إرجاع بيانات الميتا.

---

### 4. تحديثات نظام الطلبات (Nano.Orders & Nano.OrdersApi)

#### 4.1. Nano.Orders الإصدار 2.2.10  
**الفترة:** 6 – 8 مايو  
**الميزات:**  
- ترايت `HasProductOwnerScopes` في نموذج Order:  
  - نطاقات فلترة متقدمة حسب مالك المنتج (whereHasProductsByOwner, hasProductsByOwner, doesntHaveProductsByOwner، إلخ).  
  - دعم count, not, conditions, itemConditions.  
  - إضافة عمود products_owner_count والترتيب بناءً عليه.  
- دالة `updateOrderStatusAdvanced` (ضمن ترايت StepStatus):  
  - تحديث حالة الطلب بصلاحيات متدرجة (مدير، مالك، موصل).  
  - قواعد انتقال قابلة للتخصيص عبر config.  
  - دعم because_cancel, delivery_because_cancel.  
  - خيارات is_save, is_event, is_logs, skip_permission, admin_override.  
- دوال مساعدة في OrderHelper: `getDeliveryByUser`, `getUserByDelivery`.  
- دعم فلترة مالك المنتج في `OrderHelper::getOrdersRecords`.

#### 4.2. Nano.OrdersApi الإصدار 1.0.20  
**الفترة:** 6 – 8 مايو  
**الميزات:**  
- نقطة نهاية `POST /api/v1/orders/orders/update-status` لتحديث حالة الطلب.  
- دمج مع `updateOrderStatusAdvanced`، دعم جميع الخيارات.

#### 4.3. Nano.Orders الإصدار 2.2.11  
**الفترة:** 26 – 29 مايو  
**الميزات:**  
- إضافة علاقات جديدة في نموذج Order:  
  - `delivery_vehicle_type` و `delivery_vehicle_ref_type` (نوع مركبة الموصل).  
  - `delivery_car` (مركبة الموصل).  
  - `country` و `state` (الدولة والمدينة).  
  - `departuredestination` و `arrivaldestination` (مطارات المغادرة والوصول).  
- توسيع إعدادات تقرير الطلب (OrdersSetting) بأقسام جديدة: بيانات العميل التفصيلية، بيانات الحجز والرحلة، بيانات الحمولة، إلخ.  
- إعادة كتابة قالب التقرير `ReportsOrders` ليعرض جميع الحقول بشكل منظم مع التحكم عبر الإعدادات.

#### 4.4. Nano.OrdersApi الإصدار 1.0.21  
**الفترة:** 26 – 29 مايو  
**الميزات:**  
- إضافة العلاقات الجديدة إلى `availableIncludes` في OrderTransformer.  
- دوال include منفصلة: `includeDeliveryVehicleType`, `includeDeliveryCar`, `includeCountry`, `includeState`, `includeDeparturedestination`, `includeArrivaldestination`.

---

### 5. تطوير نظام إدارة الصلاحيات المركزية (AccessManager)

#### 5.1. Nano.AuthApi الإصدار 1.0.19 (15 – 16 مايو)  
**الميزات:**  
- نقاط نهاية لجلب خيارات الحقول:  
  - `GET /api/v1/user/options` مع بارامتر `fields`.  
  - `GET /api/v1/user/frontend-options` و `backend-options`.  
- متحكم `UserGroups` (قائمة، عرض) مع دعم فلاتر (is_active, is_new_user_default، بحث).  
- تحديث UserGroupTransformer ليشمل الأعمدة الجديدة (is_new_user_default, is_active, sort_order).  
- إضافة هجرة لإضافة تلك الأعمدة إلى جدول user_groups.

#### 5.2. Nano.AuthApi الإصدار 1.0.20 (24 – 25 مايو)  
**الميزات:**  
- كلاس `AccessManager` (Singleton) لإدارة الصلاحيات والوصول بشكل مركزي.  
**الوظائف الأساسية:**  
  - `check($operation, $config, $user)` تعيد مصفوفة (allowed, scope, message, applied_scope_details).  
  - دوال تطبيق النطاق: `applyAccessScope`, `applyCompanyScope`, `applyDepartmentScope`, `applyStateScope`, `applyCreatedByScope`, `applyChildrenScope`.  
  - دوال مساعدة: `canAccessAllCompanies`, `canAccessAllDepartments`, `canAccessAllStates`.  
  - دعم أنواع المستخدمين: backend, frontend, guest.  
  - دعم نطاقات: all, own, created_by, company, department, state, children.  
  - دمج `BasicHelper::checkAccessAllCompanys` وغيره.

#### 5.3. Nano.AuthApi الإصدار 1.0.21 (25 – 26 مايو)  
**الميزات المتقدمة:**  
- محلل ديناميكي للطلاب وأولياء الأمور (`resolveDynamicFrontendOptions`) يقوم بتعبئة `student_id` و `record_id` تلقائياً بناءً على `ref_type` وقواعد من config.  
- التحكم في الفلاتر المتقدمة (`advanced_filters`): تحديد الحقول التي يسمح فيها بـ `is_or`, `is_not`, `is_or_null` لكل نوع مستخدم.  
- دوال مبسطة: `checkByResource`, `getOperationByConfig`, `checkWithFallback`.  
- تحميل مركزي لإعدادات الصلاحيات من config.php مع دمج القيم الافتراضية.  
- دعم الرجوع (fallback) بين العمليات (مثل show تعتمد على list).

---

### 6. تطوير سلوكيات عامة (ProposalModel و VisitModel)

#### 6.1. ProposalModel (نظام البلاغات والمقترحات) – 19 – 20 مايو  
**النطاقات الجديدة:**  
- `scopeAddCountProposals` / `scopeSortByCountProposals` / `scopeWithCountProposals`.  
- `scopeAddLatestProposal` / `scopeSortByLatestProposal` / `scopeWithLatestProposal` (وأقدم).  
- `scopeProposedByUser` / `scopeNotProposedByUser` (الكائن كهدف).  
- `scopeProposedToUser` / `scopeNotProposedToUser` (الكائن كمرسل).  
- `scopeHasProposals` / `scopeHasNoProposals`.  
- `scopeWithIsProposedByUser` / `scopeWithIsProposedToUser`.  
- `scopeWhereHasProposalAdvanced` (نطاق متقدم يدعم side, type, conditions, useUserConditions, withTrashed).  
- دوال إحصائية: `getTotalProposals`, `getProposalsCountByType`, `getProposersUsers`.

#### 6.2. VisitModel (نظام الزيارات والمشاهدات) – 20 – 21 مايو  
**النطاقات الجديدة:**  
- `scopeAddCountVisits` / `scopeSortByCountVisits` / `scopeWithCountVisits`.  
- `scopeAddSumVisits` / `scopeSortBySumVisits` / `scopeWithSumVisits`.  
- `scopeAddSumViews` / `scopeSortBySumViews` / `scopeWithSumViews`.  
- `scopeAddLatestVisit` / `scopeSortByLatestVisit` / `scopeWithLatestVisit` (وأقدم).  
- `scopeVisitedByUser` / `scopeNotVisitedByUser`.  
- `scopeHasVisits` / `scopeHasNoVisits`.  
- `scopeWithIsVisitedByUser`.  
- دوال إحصائية: `getTotalVisits`, `getTotalViews`, `getVisitsCountByType`, `getVisitorsUsers`.  
- دعم `withTrashed` لتضمين الزيارات المحذوفة.

---

### 7. تحديثات داعمة وتحسينات أخرى

#### 7.1. Tss.Student (الإصدار 1.0.13) – 24 – 26 مايو  
**الميزات:**  
- إعادة هيكلة دوال `getRecords` في كلاس `StudentRecordsHelper` الجديد.  
- دالة `getParentRecords` (لأولياء الأمور) و `getStudentRecordRecords` (للسجلات الدراسية).  
- دمج `AccessManager` لتطبيق نطاقات الصلاحيات تلقائياً.  
- دمج `AdvancedQueryHelper` للفلاتر الديناميكية مع دعم is_or, is_not, is_force, is_or_null.  
- دوال ربط: `getStudentByUser`, `getUserByStudent`, `getMparentByUser`, `getUserByMparent`, `getStudentsByMparent`, `getStudentsByUser`, `getCurrentStudentRecord`.  
- دعم التخزين المؤقت والأحداث وهيكل استجابة موحد.

#### 7.2. Tss.Homework (الإصدار 1.0.9) – 27 – 28 مايو  
**الميزات:**  
- إضافة وحدة `Feedback` (التسليمات والتقييمات) بشكل كامل:  
  - نموذج Feedback مع علاقات (homework, student, student_record, class, group, subject, teacher).  
  - واجهة إدارة (متحكم Feedbacks، قوائم، نموذج إدخال مع تبويبات).  
  - صلاحيات مخصصة (access_all, access, add, edit, delete).  
  - قائمة جانبية تحت قائمة homework.  
  - ترجمة كاملة.  
- لم يتم إجراء هجرات جديدة (جدول tss_homework_feedbacks موجود منذ 1.0.7).

#### 7.3. Nano.UserPlus (الإصدار 1.1.7) – 13 – 14 مايو  
**الميزات:**  
- أنواع حسابات ديناميكية (`ref_type`) عبر config.php (قائمة ref_type.delivery, ref_type.department، إلخ).  
- معالج توافق مع RainLab.User 2.x (RainlabUser2CompatibilityHandler) لربط first_name/last_name مع name/surname.  
- إضافة أعمدة جديدة لجدول user_groups: `is_new_user_default` (افتراضي للمستخدمين الجدد)، `is_active`، `sort_order`.  
- توسيع واجهة إدارة مجموعات المستخدمين (أعمدة القائمة وحقول النموذج).  
- ترجمة للمفاتيح الجديدة.

#### 7.4. Nano.Coupons (الإصدار 2.1.1) – 2 – 8 مايو  
**الميزات:**  
- كلاس `ProductLimitValidator` مستقل لفحص قيود حد المنتج.  
- دعم سيناريوهات جديدة: تقييد عدد الطلبات لنوع طلب معين (بدون منتج محدد).  
- تحسين الأداء عبر التخزين المؤقت للقيود (Cache).  
- رسائل خطأ مفصلة مع بارامترات (الحد الأقصى، العدد الحالي، الفترة، تاريخ الطلب التالي).  
- تحديث واجهة الإدارة: product_id من نوع recordfinder، units_id يعتمد على product_id.  
- نطاقات `scopeOnProductLimit` و `scopeNotOnProductLimit`.

---

## الاستنتاج والتوصيات

### النجاحات المحققة  
1. **إنجازات غير مسبوقة من حيث الكم والنوع** – تم إطلاق 5 واجهات APIs كاملة، و4 بوابات دفع، وتحديثات جوهرية في الأنظمة المحاسبية والطلبات والصلاحيات، مما يعكس قدرة الفريق على التسليم بجودة عالية في وقت قياسي.  
2. **توحيد المعمارية** – اعتماد معايير `Nano-Api-SKILL.md` في جميع الـ APIs الجديدة (AccessManager، AdvancedQueryHelper، هيكل getRecords الموحد) يضمن سهولة الصيانة والتوسع.  
3. **تغطية قطاعات جديدة** – دعم بوابات دفع محلية متعددة (BasPay, CashPay, FloosakPay) يفتح أسواقاً جديدة في اليمن، ودعم ThawaniPay سابقاً يُكمّل التغطية الإقليمية.  
4. **تحسين أمني كبير** – إطلاق AccessManager يوفر تحكماً دقيقاً في الصلاحيات ونطاقات الوصول، بالإضافة إلى حماية الوثائق المعتمدة في KYC (أبريل) وتعمية البيانات الحساسة (Redactor في فبراير).  
5. **توثيق استثنائي** – تم إنتاج أكثر من 40 ملف توثيق (عربي/إنجليزي) خلال مايو، مع أمثلة عملية متقدمة وشاملة.  
6. **مرونة عالية** – إمكانية تخصيص الفلاتر المتقدمة (is_or, is_not) والصلاحيات (AccessManager) عبر ملفات config.php فقط، دون تعديل الكود.

### التوصيات للمرحلة القادمة  
1. **نشر الوعي الداخلي والخارجي** – تنظيم ورش عمل وعروض تقديمية للفرق (التطوير، ضمان الجودة، الدعم الفني) حول الـ APIs الجديدة و AccessManager وطرق الدفع الجديدة، حيث أن هذه الأنظمة تمثل نقلة نوعية في المنصة.  
2. **دمج الأنظمة في المشاريع القائمة** – وضع خطة مرحلية لترقية المشاريع (حراج، تيسير، سوق عقار، المنصة التعليمية) للاستفادة من APIs المدارس والحضور والواجبات، وكذلك من نظام رفع الملفات المركزي و KYC.  
3. **مراقبة أداء بوابات الدفع الجديدة** – تتبع معدلات نجاح الدفع وزمن الاستجابة لبوابات BasPay, CashPay, FloosakPay في بيئة الإنتاج، وإعداد تقارير دورية.  
4. **إكمال دعم الوسائط المتعددة** – تفعيل includes `videos`, `audios`, `book_intro` في DynamicAddMediaIncludes للحسابات والموقع الجغرافي، وإضافة الواجهات الخلفية المناسبة.  
5. **الاستمرار في تحسين التوثيق المرئي** – إنتاج فيديوهات تعليمية قصيرة (2-5 دقائق) لكل API جديد ولـ AccessManager، لتسريع عملية التبني من قبل المطورين الخارجيين.  
6. **إعداد خطة اختبار أداء** – خاصة لاستعلامات getRecords المتقدمة مع الفلاتر والأحداث والتخزين المؤقت، للتأكد من كفاءة الاستعلامات عند التعامل مع آلاف السجلات.

---

**معد التقرير:** فريق التوثيق التقني  
**تاريخ الإصدار:** 31 مايو 2026  
**التصنيف:** تقرير إنجازات داخلي