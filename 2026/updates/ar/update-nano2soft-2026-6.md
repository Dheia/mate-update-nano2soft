# Update 2026-6

**تحديثات شهر ستة يونيو **


## 2026-06-01

**تحديث إضافة `Nano2.ProposalsApi` – الإصدار 1.0.3**

### دعم تجاوز فلتر `User()` في `getRecords` لتمكين أولياء الأمور من استعراض مقترحات / شكاوي / بلاغات أبنائهم

---

### ملخص التحديثات

يقدم الإصدار **1.0.3** من إضافة `Nano2.ProposalsApi` تحسيناً مهماً في آلية جلب السجلات (المقترحات، الشكاوي، البلاغات) الخاصة بالمستخدم الحالي. سابقاً، كان يتم تطبيق الفلتر `User($user)` بشكل إلزامي في دالة `getRecords()`، مما كان يمنع أولياء الأمور من رؤية السجلات المتعلقة بأبنائهم (الطلاب) حتى لو تم تمرير `target_type` و `target_id` بشكل صحيح.

**أبرز التغييرات:**
- إضافة معامل جديد `is_stop_user` (من نوع boolean) إلى `Proposals@getRecords`.
- عند تفعيل `is_stop_user=1` وبشرط أن يكون نوع المستخدم الحالي طالباً أو ولي أمر، يتم تجاوز فلتر `User()` تماماً.
- السماح لولي الأمر بجلب جميع السجلات الخاصة بأبنائه من خلال تمرير `target_type` و `target_id` الخاص بالطالب.
- تحديث التوثيق (`docs.md`) بإضافة الأمثلة التوضيحية `1.8.1`، `1.8.2`، `1.8.3` التي تشرح كيفية استخدام `is_stop_user` مع أنواع السجلات المختلفة.

---

### أهداف الإصدار

- **تمكين أولياء الأمور**: إتاحة الوصول إلى مقترحات، شكاوي، وبلاغات أبنائهم عبر API دون الحاجة إلى صلاحيات إضافية معقدة.
- **مرونة الفلترة**: توفير طريقة مباشرة لتجاوز فلتر `User()` عند الحاجة (حالات التقارير والبلاغات الموجّهة لطلاب معينين).
- **الحفاظ على الأمان**: لا يتم تجاوز الفلتر إلا للنوعين `student` و `parent` وعند تمرير `is_stop_user=1` صراحةً.
- **توافق عكسي**: السلوك الافتراضي (`is_stop_user=0` أو عدم تمريره) يبقى كما هو (تطبيق فلتر `User()`).

---

### الميزات الجديدة والتحسينات

#### 1. إضافة معامل `is_stop_user` إلى `getRecords`

**قبل الإصدار 1.0.3:**
```php
$posts = $posts->User($user);
```

**بعد التحديث:**
```php
$allowed_ref_types = [
    'Tss\Student\Models\Student',
    'Tss\Student\Models\Mparent',
    'student',
    'parent',
];

$allowed_stop_user = [
    'Tss\Student\Models\Student',
    'student',
];

$is_stop_user = Input::get('is_stop_user', false);
if($is_stop_user && (!in_array($target_type, $allowed_stop_user) || !in_array($user->ref_type, $allowed_ref_types)))
    $is_stop_user = false;

if(! $is_stop_user)
    $posts = $posts->User($user);
```

#### 2. منطق عمل `is_stop_user`

- **يسمح بتجاوز فلتر `User()`** إذا تحققت الشروط التالية:
  - تم تمرير `is_stop_user=1` في الطلب.
  - `target_type` موجود ويشير إلى نموذج طالب (`Tss\Student\Models\Student` أو `student`).
  - المستخدم الحالي من نوع `student` أو `parent` (ولي أمر).

- **لا يُسمح** بتجاوز الفلتر لأنواع المستخدمين الأخرى (مثل مشرف، مدير) أو عند عدم وجود `target_type` صحيح.

- الهدف الأساسي: تمكين ولي الأمر من استعراض السجلات الخاصة بابنه عبر تمرير `target_type=...Student` و `target_id=id_الابن`.

#### 3. أمثلة توضيحية جديدة في التوثيق

تم إضافة الأمثلة التالية إلى ملف `docs.md`:

- **1.8.1** – جلب المقترحات المقدمة لطالب معين.
- **1.8.2** – جلب الشكاوي المقدمة لطالب معين.
- **1.8.3** – جلب البلاغات المقدمة عن طالب معين.

جميع الأمثلة تستخدم `is_stop_user=1` وتفترض أن المستخدم الحالي هو ولي أمر لذلك الطالب.

---

### أمثلة عملية على الاستخدام الجديد

#### 1. جلب مقترحات موجهة لطالب معين (بواسطة ولي الأمر)

```bash
curl -X GET "https://yourdomain.com/api/v1/proposals/proposals?type=proposals&target_type=Tss\\Student\\Models\\Student&target_id=8&is_stop_user=1" \
  -H "Authorization: Bearer <token>"
```

#### 2. جلب شكاوي مقدمة لطالب معين

```bash
curl -X GET "https://yourdomain.com/api/v1/proposals/proposals?type=complaints&target_type=Tss\\Student\\Models\\Student&target_id=8&is_stop_user=1"
```

#### 3. جلب بلاغات مرفوعة ضد طالب معين

```bash
curl -X GET "https://yourdomain.com/api/v1/proposals/proposals?type=reports&target_type=Tss\\Student\\Models\\Student&target_id=8&is_stop_user=1"
```

---

### متطلبات الترقية (من 1.0.2 إلى 1.0.3)

1. **تحديث الكود**:
   - استبدال ملف المتحكم:
     `plugins/nano2/proposalsapi/APIControllers/Proposals.php`

2. **تحديث التوثيق (اختياري ولكن موصى به)**:
   - استبدال ملف `docs.md` بالنسخة التي تحتوي على الأمثلة الجديدة (1.8.1 - 1.8.3).

3. **تنفيذ أي أوامر ضرورية** (لا توجد هجرات قاعدة بيانات):
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

4. **اختبار الوظائف**:
   - تأكد من أن مستخدم ولي أمر يمكنه جلب سجلات الطالب المرتبط به عند تمرير `is_stop_user=1`.
   - تأكد من أن مستخدم عادي (ليس طالباً ولا ولي أمر) لا يستطيع تجاوز فلتر `User()` حتى مع `is_stop_user=1`.
   - تأكد من أن السلوك الافتراضي (بدون `is_stop_user`) لم يتغير.

---

### الفوائد والقيمة المضافة

- **تحسين تجربة أولياء الأمور**: يمكنهم الآن متابعة كافة التفاعلات (مقترحات، شكاوي، بلاغات) المتعلقة بأبنائهم من خلال واجهة برمجة واحدة.
- **مرونة مطوري التطبيقات**: لم يعودوا مضطرين لبناء حلول معقدة لتجميع البيانات من عدة حسابات.
- **أمان محكم**: لا يمكن تجاوز الفلتر إلا للحالات المصرح بها صراحة، مما يمنع تسرّب البيانات.
- **توافق عكسي كامل**: جميع التطبيقات الحالية التي لا تستخدم `is_stop_user` ستعمل بنفس الطريقة السابقة.

---

### الخاتمة

يمثل الإصدار **1.0.3** من `Nano2.ProposalsApi` خطوة مهمة نحو دعم سيناريوهات المستخدمين المتعددة (ولي الأمر – الطالب) في نظام إدارة المقترحات والشكاوي والبلاغات. من خلال إضافة معامل `is_stop_user`، تمكنا من تحقيق توازن مثالي بين الأمان والمرونة، مما يسمح لأولياء الأمور بالاطلاع على بيانات أبنائهم دون التضحية بسلامة البيانات. نوصي جميع مشاريع `ProposalsApi` التي تتعامل مع حسابات الطلاب وأولياء الأمور بالترقية إلى هذا الإصدار.

---

**الوثائق المرجعية**:
- [توثيق إضافة `Nano2.ProposalsApi`](./docs/ProposalsApi/Docs-ProposalsApi-ar.md)
- [توثيق مختصر إضافة ProposalsApi API (Nano2.ProposalsApi)](./docs/ProposalsApi/Docs-ProposalsApi-Short-ar.md)
- [أمثلة عملية شاملة – إضافة ProposalsApi](./docs/ProposalsApi/Docs-ProposalsApi-Examples-ar.md)
- [دليل تطوير إضافات Nano API (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)

## 2026-06-01 - 2026-06-02

**تحديث إضافة `Nano.MediaApi` – الإصدار 1.0.2**

### تحسين فلترة التصنيفات في متحكمات الصور والفيديوهات والملفات لدعم التصنيفات الأم (Parent Categories)

---

### ملخص التحديثات

يقدم الإصدار **1.0.2** من إضافة `Nano.MediaApi` تحسيناً مهماً في آلية فلترة العناصر حسب التصنيف (`categories_id`). سابقاً، كانت الفلترة تقتصر على مطابقة `categories_id` المباشر لكل عنصر (صورة، فيديو، ملف). أما الآن، فتم توسيع المنطق ليشمل أيضاً العناصر التي تنتمي إلى تصنيفات يكون التصنيف الممرر هو **التصنيف الأب (parent_id)** لها، وذلك عبر العلاقات (`album` للصور، `playlist` للفيديوهات، `filelist` للملفات).

**أبرز التغييرات:**
- تعديل دالة `index` في المتحكمات:
  - `Nano\MediaApi\APIControllers\Images`
  - `Nano\MediaApi\APIControllers\Videos`
  - `Nano\MediaApi\APIControllers\Files`
- دعم تمرير `categories_id` كمصفوفة (array) للفلترة بعدة تصنيفات.
- استخدام `orWhereHas` للوصول إلى التصنيفات غير المباشرة عبر العلاقة.
- تحديث التوثيق الخاص بمكتبة الوسائط (Media Library API) ليعكس السلوك الجديد.

---

### أهداف الإصدار

- **تحسين دقة الفلترة**: تمكين المطورين من جلب جميع الصور أو الفيديوهات أو الملفات الموجودة تحت تصنيف معين **وجميع تصنيفاته الفرعية** دون الحاجة إلى استدعاءات متعددة.
- **تبسيط التكامل**: عدم إجبار المستخدم على معرفة الشجرة الكاملة للتصنيفات؛ يكفي تمرير التصنيف الأب.
- **دعم مرن للمصفوفات**: السماح بتمرير عدة تصنيفات (أباء أو أبناء) في طلب واحد.
- **التوافق مع توقعات المستخدمين**: في أنظمة إدارة المحتوى، غالباً ما يُتوقع أن اختيار تصنيف يعرض محتوى جميع التصنيفات الفرعية أيضاً.

---

### الميزات الجديدة والتحسينات

#### 1. توسيع منطق فلترة `categories_id` في `Images`، `Videos`، `Files`

**قبل الإصدار 1.0.2:**
```php
if($categories_id = Input::get('categories_id', false)) {
    $posts = $posts->where('categories_id', $categories_id);
}
```

**بعد التحديث:**
```php
if($categories_id = Input::get('categories_id', false)) {
    $posts = $posts->where(function ($query) use ($categories_id) {
        if(is_array($categories_id)) {
            $query->whereIn('categories_id', $categories_id)
                ->orWhereHas('album', function ($query2) use ($categories_id) {
                    $query2->whereIn('tss_media_categories.parent_id', $categories_id);
                });
        } else {
            $query->where('categories_id', $categories_id)
                ->orWhereHas('album', function ($query2) use ($categories_id) {
                    $query2->where('tss_media_categories.parent_id', $categories_id);
                });
        }
    });
}
```

> **ملاحظة:** تم تطبيق نفس المنطق في متحكم `Videos` باستخدام العلاقة `playlist`، وفي متحكم `Files` باستخدام العلاقة `filelist`.

#### 2. دعم تمرير `categories_id` كمصفوفة

أصبح من الممكن الآن تمرير عدة تصنيفات دفعة واحدة:

```
GET /api/v1/media/images?categories_id[]=3&categories_id[]=5
```

سيؤدي ذلك إلى جلب الصور التي تتبع التصنيف 3 أو 5 **أو** التي تتبع أي ألبوم يكون التصنيف الأب لألبومها هو 3 أو 5.

#### 3. تحديث توثيق API

تم تحديث التوثيق السابق (`Docs-MediaApi-ar.md`) ليشمل شرحاً للسلوك الجديد وإيضاح أن فلترة `categories_id` تشمل الآن **جميع العناصر المرتبطة بالتصنيف الممرر سواء بشكل مباشر أو عبر العلاقات (الألبومات/قوائم التشغيل/حزم الملفات)**.

---

### أمثلة عملية على الاستخدام الجديد

#### 1. جلب جميع الصور تحت تصنيف رئيسي (مع تصنيفاته الفرعية)

افترض أن لديك التصنيف رقم `10` باسم "الطبيعة"، وتحته تصنيفات فرعية "جبال" و"بحار".  
بتمرير `categories_id=10`، ستحصل على جميع الصور التي:
- `categories_id = 10` مباشرة، أو
- تنتمي إلى ألبوم يكون `parent_id` لذلك الألبوم هو `10`.

```bash
curl -X GET "https://yourdomain.com/api/v1/media/images?categories_id=10" \
  -H "Authorization: Bearer <token>"
```

#### 2. جلب الفيديوهات من عدة تصنيفات رئيسية في وقت واحد

```bash
curl -X GET "https://yourdomain.com/api/v1/media/videos?categories_id[]=2&categories_id[]=7" \
  -H "Authorization: Bearer <token>"
```

#### 3. جلب الملفات من تصنيف محدد مع تضمين حزمة الملفات

```bash
curl -X GET "https://yourdomain.com/api/v1/media/files?categories_id=15&include=filelist" \
  -H "Authorization: Bearer <token>"
```

---

### الفوائد والقيمة المضافة

- **تقليل عدد استدعاءات الـ API**: لم يعد المطور بحاجة إلى جلب جميع التصنيفات الفرعية أولاً ثم إجراء استدعاءات منفصلة لكل منها.
- **تناسق أفضل مع منطق واجهة المستخدم**: في معظم تطبيقات إدارة الوسائط، النقر على تصنيف رئيسي يعرض محتوى جميع الأقسام الفرعية.
- **مرونة متزايدة**: دعم المصفوفات يسمح بدمج مجموعات تصنيفات غير مترابطة في طلب واحد.
- **توافق عكسي كامل**: إذا كنت تمرر `categories_id` كقيمة واحدة، فسيظل السلوك القديم (المباشر) صحيحاً، مع إضافة النتائج غير المباشرة (وهو ما يعزز التوافق).

---

### متطلبات الترقية (من 1.0.1 إلى 1.0.2)

1. **تحديث الكود**:
   - استبدال ملفات المتحكمات التالية:
     - `plugins/nano/mediaapi/APIControllers/Images.php`
     - `plugins/nano/mediaapi/APIControllers/Videos.php`
     - `plugins/nano/mediaapi/APIControllers/Files.php`

2. **تحديث التوثيق** (اختياري ولكن موصى به):
   - استبدال ملف `Docs-MediaApi-ar.md` (أو أي ملف توثيق API) بالنسخة المحدثة التي توضح السلوك الجديد للفلترة.

3. **تنفيذ أي أوامر ضرورية** (لا توجد هجرات قاعدة بيانات):
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

4. **اختبار الوظائف**:
   - تأكد من أن طلب `GET /api/v1/media/images?categories_id=X` يعيد الصور من الألبومات التي يكون `parent_id` لها يساوي `X`.
   - اختبر تمرير مصفوفة: `categories_id[]=1&categories_id[]=2`.
   - اختبر بقية النقاط (`videos`, `files`) بنفس الطريقة.

---

### الخاتمة

يمثل الإصدار **1.0.2** من `Nano.MediaApi` خطوة مهمة نحو تحسين تجربة المطورين والمستخدمين النهائيين عند التعامل مع الوسائط المصنفة هرمياً. من خلال دعم التصنيفات الأم والسماح بالمصفوفات، أصبحت الإضافة أكثر ذكاءً ومرونة، مع الحفاظ على التوافق الكامل مع الإصدارات السابقة. نوصي جميع مشاريع `Nano.MediaApi` بالترقية إلى هذا الإصدار للاستفادة من سلوك الفلترة الطبيعي والمتوقع.

---

**الوثائق المرجعية**:
- [توثيق مكتبة الوسائط API (MediaApi)](./docs/MediaApi/Docs-MediaApi-ar.md)
- [توثيق مختصر مكتبة الوسائط API (MediaApi)](./docs/MediaApi/Docs-MediaApi-Short-ar.md)
- [أمثلة عملية شاملة – إضافة MediaApi](./docs/MediaApi/Docs-MediaApi-Examples-ar.md)
- [أمثلة عملية مختصرة – إضافة MediaApi](./docs/MediaApi/Docs-MediaApi-Short-Examples-ar.md)

## 2026-05-30 - 2026-06-03


**تحديث إضافة `Tss.Studyyear` – الإصداران 1.0.6 و 1.0.7**  
**وتحديث إضافة `Nano.StudyyearApi` – الإصدار 1.0.1**

### إضافة عمود `is_published_results` ودعم النشر والفلترة المتقدمة للسنوات والفصول والأشهر الدراسية

---

### ملخص التحديثات

تم إجراء تحديثين متتاليين على إضافة `Tss.Studyyear` (الإصداران 1.0.6 و 1.0.7) بالإضافة إلى تحديث إضافة `Nano.StudyyearApi` (الإصدار 1.0.1) بهدف:

- **إضافة عمود جديد** `is_published_results` إلى جداول السنوات والفصول والأشهر الدراسية.
- **تمكين التحكم في نشر البيانات** عبر حقول `is_published`, `published_at`, `unpublished_at` في واجهة الإدارة الخلفية.
- **توفير فلاتر متقدمة** لتصفية السجلات المنشورة وغير المنشورة والمخصصة لنشر النتائج.
- **دعم هذه الحقول في واجهة برمجة التطبيقات (API)** لتتمكن التطبيقات الخارجية من الاستفادة من حالة النشر.

تأتي هذه التحديثات استجابة لحاجة المدارس والمراكز التعليمية في التحكم بظهور السنوات والفصول والأشهر الدراسية للمستخدمين (خاصة في تطبيقات النتائج والتقارير) بشكل ديناميكي.

---

### أهداف التحديثات

- **إضافة مرونة في نشر البيانات**: يمكن للمسؤول تحديد ما إذا كانت السنة الدراسية أو الفصل أو الشهر منشوراً على الموقع أو متاحاً للاستعلامات عبر API.
- **فصل نشر البيانات الأساسية عن نشر النتائج**: باستخدام الحقل الجديد `is_published_results` يمكن التحكم بشكل منفصل في ظهور بيانات التقارير والنتائج للطلاب وأولياء الأمور.
- **دعم التواريخ المؤقتة للنشر**: من خلال `published_at` و `unpublished_at` يمكن جدولة النشر والانتهاء منه تلقائياً.
- **تحسين أداء التطبيقات**: عبر توفير فلاتر متقدمة في الخلفية والـ API لتقليل كمية البيانات المنقولة.
- **التوافق مع متغيرات البيئة**: إتاحة تعطيل فحص تواريخ الصلاحية (Check Date) لكل كيان على حدة عبر متغيرات البيئة.

---

### الميزات الجديدة والتحسينات

#### 1. إضافة عمود `is_published_results` إلى الجداول

تم إنشاء ملف هجرة جديد (`builder_table_add_is_published_results_columns.php`) يضيف عموداً من النوع `boolean` باسم `is_published_results` إلى الجداول التالية:

- `tss_studyyear_periods`
- `tss_studyyear_semsters`
- `tss_studyyear_months`

**دلالة العمود:**  
تحديد ما إذا كانت بيانات هذا الكيان مسموحاً بنشرها في سياق "النتائج" (مثل ظهورها للطلاب في تطبيق النتائج). يمكن استخدامه بشكل منفصل عن `is_published`.

#### 2. دعم حقول النشر في النماذج (`fields.yaml`)

تم تحديث ملفات `fields.yaml` الخاصة بـ `Periods`, `Semsters`, `Months` لتشمل الحقول التالية ضمن تبويب `options`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `is_published` | Switch | نشر العنصر على الموقع / الـ API |
| `published_at` | Datepicker | تاريخ بدء النشر (اختياري) |
| `unpublished_at` | Datepicker | تاريخ انتهاء النشر (اختياري) |
| `is_published_results` | Switch | نشر العنصر في نتائج التقارير |

تمت إضافة منطق `trigger` بحيث يظهر حقلا التاريخ فقط عند تفعيل `is_published`.

#### 3. دعم الحقول في القوائم (`columns.yaml`)

أضيفت الأعمدة التالية إلى ملفات `columns.yaml` الخاصة بكل كيان:

- `is_published_results` (نص / مفتاح)
- `is_published` (نص / مفتاح)
- `published_at` (تاريخ ووقت)
- `unpublished_at` (تاريخ ووقت)

مع جعلها `invisible: true` افتراضياً لإبقاء القائمة بسيطة، ولكن يمكن إظهارها حسب الرغبة.

#### 4. إضافة نطاقات (Scopes) جديدة في النماذج

تم إضافة النطاقات التالية في نماذج `Period`, `Semster`, `Month`:

```php
public function scopeIsPublished($query)
{
    $now = Carbon::now();
    return $query->where('is_published', true)
        ->where(function ($q) use ($now) {
            $q->where('published_at', '<=', $now)
              ->orWhereNull('published_at');
        })
        ->where(function ($q) use ($now) {
            $q->where('unpublished_at', '>=', $now)
              ->orWhereNull('unpublished_at');
        });
}

public function scopeIsNotPublished($query)
{
    // النفي المنطقي لـ isPublished
}
```

كما تمت إضافة نطاق `scopeIsPublishedResults` للتحقق من `is_published_results`.

#### 5. فلاتر متقدمة في واجهة الخلفية (`config_filter.yaml`)

تم تحديث ملفات `config_filter.yaml` لكل من `Periods`, `Semsters`, `Months` بإضافة الفلاتر التالية:

| اسم الفلتر | النوع | الوصف |
|-----------|-------|-------|
| `is_published` | Checkbox | يعرض السجلات المنشورة فقط (يستخدم `scopeIsPublished`) |
| `is_not_published` | Checkbox | يعرض السجلات غير المنشورة فقط (يستخدم `scopeIsNotPublished`) |
| `is_published_results` | Switch | تصفية بناءً على `is_published_results` (0/1) |
| `is_default` | Switch | تصفية السجلات الافتراضية |
| `is_active` | Switch | تصفية النشطة |

كما تمت إضافة فلتر `departments_id` (مجموعة) و `study_year_id` و `semsters_id` و `month_num` للكائنات المناسبة.

#### 6. دعم متغيرات البيئة للتحقق من صلاحية التواريخ

تم تحديث ملف `config.php` الخاص بـ `Tss.Studyyear` ليصبح:

```php
return [
    'check_date_year' => env('TSS_STUDYYEAR_CHECK_DATE_YEAR', true),
    'check_date_semsters' => env('TSS_STUDYYEAR_CHECK_DATE_SEMSTERS', true),
    'check_date_months' => env('TSS_STUDYYEAR_CHECK_DATE_MONTHS', true),
];
```

هذه المتغيرات تتحكم في تفعيل فحص تواريخ الصلاحية (`from_date`, `to_date`) عند جلب البيانات. إذا تم تعطيلها (false) يتم إلغاء شرط `to_date > now()` مما يسمح بعرض العناصر منتهية الصلاحية.

#### 7. تحديث واجهة برمجة التطبيقات `Nano.StudyyearApi` (الإصدار 1.0.1)

- **إضافة عمود `is_published_results`** في المحولات (`PeriodTransformer`, `SemsterTransformer`, `MonthTransformer`).
- **دعم فلاتر جديدة** في دوال `getRecords` لكل من `Periods`, `Semsters`, `Months`:
  - `is_published_results` (0/1)
  - `is_published` (0/1)
  - `is_default` (0/1)
- **تحديث منطق الجلب** في الـ API بحيث إذا لم يمرر المستخدم `is_published_results`، يتم التعامل مع `'*'` أو `null` لعدم التصفية.
- **تحسين معالجة التواريخ** في المحولات باستخدام `formatDate`.

**مثال من `Periods.php` (API):**

```php
if ($options['is_published_results'] !== null && $options['is_published_results'] !== '*') {
    $posts->where($table . '.is_published_results', (bool)$options['is_published_results']);
}
if ($options['is_published'] !== null && $options['is_published'] !== '*') {
    $options['is_published'] ? $posts->isPublished() : $posts->isNotPublished();
}
if ($options['is_default'] !== null && $options['is_default'] !== '*') {
    $posts->where($table . '.is_default', (bool)$options['is_default']);
}
```

#### 8. تحديث ملفات الإصدارات (`version.yaml`)

**Tss.Studyyear**:
```yaml
1.0.6:
    - 'Add is_published_results columns to all table tss_studyyear '
    - builder_table_add_is_published_results_columns.php
1.0.7:
    - 'Support fields (is_published,published_at,unpublished_at and is_published_results) In Backend Interface'
    - 'Support filter (is_published,published_at,unpublished_at and is_published_results) In Backend Interface'
    - 'Support list columns (is_published,is_not_published and is_published_results) In Backend Interface'
```

**Nano.StudyyearApi**:
```yaml
1.0.0:
    - Plugin initialization Nano.StudyyearApi.
1.0.1:
    - 'Support is_published_results column in PeriodTransformer,SemsterTransformer And MonthTransformer'
    - 'Support filter is_published_results ,is_published and is_default in Periods,Semsters And Months APIControllers'
```

---

### أمثلة على الاستخدام (Backend)

#### 1. إضافة سنة دراسية جديدة مع جدولة النشر

- انتقل إلى `السنوات الدراسية > السنة الدراسية`.
- أنشئ سنة جديدة، مثلاً `2026-2027`.
- فعّل خيار `نشر على الموقع`.
- حدد `تاريخ ابتداء النشر` = `2026-06-01` وتاريخ انتهاء النشر = `2026-07-15`.
- فعّل خيار `نشر النتائج في التطبيق` (إذا أردت ظهورها في تطبيق النتائج).
- احفظ.

ستظهر السنة فقط للواجهات والـ API خلال الفترة المحددة.

#### 2. استخدام الفلتر `is_published` في قائمة الفصول

- في قائمة `الفصول الدراسية`، استخدم فلتر `منشور` (checkbox) لعرض الفصول المنشورة فقط.

#### 3. استعلام API لجلب الأشهر المنشورة للنتائج فقط

```bash
GET /api/v1/studyyear/months?study_year_id=5&is_published_results=1&include=period,semster
```

---

### متطلبات الترقية (من الإصدارات السابقة)

#### لمن يستخدم `Tss.Studyyear` (إصدارات أقدم من 1.0.6)

1. **تحديث الكود**: استبدال جميع الملفات المتغيرة (models, controllers, fields.yaml, columns.yaml, config_filter.yaml, config.php, version.yaml).
2. **تشغيل الهجرة الجديدة**:
   ```bash
   php artisan plugin:refresh Tss.Studyyear
   ```
   - ستتم إضافة الأعمدة `is_published_results` إلى الجداول الثلاثة.
3. **إعادة تعيين الأذونات** (اختياري): لا توجد أذونات جديدة.
4. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

#### لمن يستخدم `Nano.StudyyearApi` (إصدارات أقدم من 1.0.1)

1. **تحديث الكود**: استبدال ملفات `Periods.php`, `Semsters.php`, `Months.php` والمحولات (`PeriodTransformer`, `SemsterTransformer`, `MonthTransformer`).
2. **تحديث `version.yaml`**.
3. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

**ملاحظة:** لا توجد تغييرات على قاعدة البيانات في `Nano.StudyyearApi`، فقط تحسينات في الكود.

---

### التوافق مع الإصدارات السابقة

- **جميع نقاط النهاية الحالية** في الـ API لا تزال تعمل دون تغيير (الحقول الجديدة اختيارية في الفلاتر).
- **الواجهات الخلفية** تستمر في العمل حتى لو لم يتم استخدام الحقول الجديدة.
- **البيانات الحالية**: سيتم تعيين القيمة الافتراضية `false` لعمود `is_published_results` تلقائياً للصفوف الموجودة (لأنه تم إضافته مع `default(false)` في الهجرة).

---

### الفوائد والقيمة المضافة

- **تحكم ديناميكي في المحتوى**: يمكن الآن جدولة ظهور السنوات والفصول والأشهر الدراسية دون الحاجة إلى حذفها أو تعطيلها يدوياً.
- **فصل النشر العام عن نشر النتائج**: باستخدام `is_published_results` يمكن إظهار البيانات لتطبيقات النتائج مع إبقائها مخفية عن الزوار العاديين.
- **تقليل التحميل على الخادم**: الفلاتر المتقدمة تقلل كمية البيانات المسترجعة وتحسن زمن الاستجابة.
- **مرونة التخصيص**: يمكن تعطيل فحص التواريخ (Check Date) عبر متغيرات البيئة، مما يسمح بعرض السنوات القديمة في لوحات التحكم حسب الحاجة.

---

### الخاتمة

تمثل التحديثات **1.0.6** و **1.0.7** من `Tss.Studyyear` و **1.0.1** من `Nano.StudyyearApi` نقلة نوعية في إدارة السنوات والفصول والأشهر الدراسية. من خلال إضافة حقل `is_published_results` ودعم النشر المؤقت والفلاتر المتقدمة، أصبح من السهل جدولة المحتوى والتحكم فيه بشكل دقيق. نوصي جميع المستخدمين بالترقية للاستفادة من هذه الميزات، خاصة في بيئات التعليم التي تعتمد على التقارير الدورية والنتائج.

---

**الوثائق المرجعية**:
- [توثيق إضافة `Tss.Studyyear` ](./docs/Studyyear/Docs-Studyyear-ar.md)
- [توثيق إضافة `Nano.StudyyearApi`](./docs/StudyyearApi/Docs-StudyyearApi-ar.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL.md)](./Nano-Api-SKILL.md)

## 2026-06-03 -2026-06-05

**تحديث إضافة `Nano.StudyyearApi` – الإصدار 1.0.2**

### إعادة هيكلة كاملة للمتحكمات وفقاً لدليل `Nano-Api-SKILL.md` وتكامل `AccessManager` والميزات المتقدمة

---

### ملخص التحديثات

يقدم الإصدار **1.0.2** من إضافة `Nano.StudyyearApi` إعادة هيكلة كاملة للمتحكمات (`Periods`, `Semsters`, `Months`) لتتوافق بشكل كامل مع دليل `Nano-Api-SKILL.md`. تم الاستغناء عن دالة `validationList` اليدوية والاعتماد على `AccessManager` للتحكم المركزي بالصلاحيات، مع إضافة دعم الفلاتر المتقدمة (`is_or`, `is_not`, `is_force`, `is_or_null`) عبر `AdvancedQueryHelper`، وتوحيد دالة `getRecords` لدعم خيارات الإخراج المتعددة، والأحداث، والتخزين المؤقت، وتطبيق نطاقات الوصول (الشركة، القسم). كما تمت إضافة منطق ذكي لحل التبعيات بين السنة والترم في متحكم `Months`، وتصحيح إعدادات `access_scope` للمستخدمين الأماميين.

**أبرز التغييرات:**
- إزالة دوال `validationList` واستبدالها بـ `AccessManager::checkByResource` و `checkWithFallback`.
- إضافة دعم الفلاتر المتقدمة لكل الحقول باستخدام `AdvancedQueryManager::scopeWhereField`.
- توحيد دالة `getRecords` لدعم خيارات الإخراج المتنوعة (`is_query`, `is_first`, `is_collection`, `is_paginator`, `is_to_sql`).
- إضافة أحداث `api.list.extendQueryBefore` و `api.list.extendQuery` وقوائم خاصة لكل مورد.
- دعم التخزين المؤقت عبر `$this->cached()` و `getLastUpdateAt`.
- تطبيق نطاق الوصول (`applyAccessScope`) للشركة والقسم.
- مركزية إعدادات الصلاحيات في `config.php` مع متغيرات البيئة.
- إضافة منطق ذكي لاستنتاج `study_year_id` و `semsters_id` في متحكم `Months` (إذا لم يُمرر المستخدم الترم، يتم جلب الترم الافتراضي للسنة).
- تصحيح `access_scope` من `'own'` إلى `'all'` لأقسام `list` في `periods`, `semsters`, `months` لأن هذه البيانات عامة (Master Data) ولا ترتبط بمستخدم معين.

---

### أهداف الإصدار

- **توحيد معايير التطوير** بين جميع متحكمات API في النظام البيئي Nano.
- **تبسيط إدارة الصلاحيات** عبر مركزية الإعدادات في `config.php` واستخدام `AccessManager`.
- **تحسين الأمان** من خلال الفلاتر المتقدمة المقيدة والتحقق من الصلاحيات لكل عملية.
- **زيادة المرونة** عبر دعم خيارات إخراج متعددة وأحداث قابلة للتوسع.
- **تسهيل الصيانة** بإعادة هيكلة الكود بحيث يكون متسقاً مع `Nano-Api-SKILL.md`.
- **تحسين تجربة المطور** باستخدام `checkWithFallback` لتجنب تكرار إعدادات الصلاحيات لـ `show`.

---

### الميزات الجديدة والتحسينات

#### 1. استبدال `validationList` بـ `AccessManager`

- **قبل الإصدار 1.0.2**: كان كل متحكم يحتوي على دالة `validationList` تتحقق من الإعدادات يدوياً، مما أدى إلى تكرار المنطق.
- **بعد التحديث**: يتم التحقق من الصلاحيات باستخدام `AccessManager::checkByResource` و `checkWithFallback` بناءً على إعدادات `config.php`.

**مثال من `Periods.php`:**
```php
$access = AccessManager::checkByResource('nano.studyyearapi::periods.list', $user);
if (!$access['allowed']) {
    return $this->errorUnauthorized($access['message']);
}
```

#### 2. دعم الفلاتر المتقدمة (`advanced_filters`)

تمت إضافة دعم كامل لـ `is_or`, `is_not`, `is_force`, `is_or_null` للحقول الرئيسية باستخدام `AdvancedQueryManager::scopeWhereField`. يتم التحكم في هذه الفلاتر عبر إعدادات `advanced_filters` في `config.php`.

**مثال – فلتر `calendar_type` مع `is_or`:**
```php
if ($options['calendar_type'] && $options['calendar_type'] !== '*') {
    $query = AdvancedQueryManager::scopeWhereField(
        $query, 'calendar_type', $options['calendar_type'],
        $options['is_or_calendar_type'], $table,
        $options['is_not_calendar_type'], $options['is_force_calendar_type']
    );
}
```

#### 3. توحيد دالة `getRecords`

أصبحت دالة `getRecords` متطابقة في جميع المتحكمات وتدعم:

- **استبعاد الأعمدة** (`exclude`) لتقليل حجم البيانات.
- **تضمين العلاقات** (`custom_with`).
- **البحث النصي** (`q`) مع دعم البحث المتقدم (ArPhpHelper).
- **الترتيب** (`orderBy`, `orderDirection`) والمجموعات (`group_by`) و `having`.
- **خيارات الإخراج**: `is_query`, `is_first`, `is_model`, `is_collection`, `is_paginator`, `is_to_sql`.
- **الأحداث**: `api.list.extendQueryBefore` و `api.list.extendQuery` لتوسيع الاستعلامات.
- **التخزين المؤقت** عبر `$this->cached()`.

#### 4. تطبيق نطاق الوصول (`applyAccessScope`)

يتم الآن تطبيق نطاقات الشركة والقسم تلقائياً باستخدام `applyAccessScope` مع إمكانية تعطيل نطاقات معينة بسهولة:

```php
if (!empty($options['access_result']) && $options['access_result']['allowed']) {
    $query = AccessManager::instance()->applyAccessScope($query, $options['access_result'], [
        'company_field'    => 'companys_id',
        'department_field' => 'departments_id',
    ]);
}
```

#### 5. تحسين دالة `show` باستخدام `checkWithFallback`

لتجنب تكرار إعدادات الصلاحيات، تستخدم `show` الآن `checkWithFallback` لاستخدام إعدادات `list` إذا لم توجد إعدادات خاصة بـ `show`:

```php
$access = AccessManager::checkWithFallback(
    'nano.studyyearapi::periods.show',
    'nano.studyyearapi::periods.list',
    $user
);
```

#### 6. إعدادات مركزية في `config.php`

تم نقل جميع إعدادات الصلاحيات والفلاتر المتقدمة إلى `config.php`، مما يسمح بالتحكم عبر متغيرات البيئة:

```php
'periods' => [
    'list' => [
        'permission' => [
            'is_allow' => env('NANO_STUDYYEARAPI_PERIODS_LIST_IS_ALLOW', true),
            'backend' => [...],
            'frontend' => [
                'allow' => env('NANO_STUDYYEARAPI_PERIODS_LIST_FRONTEND_ALLOW', true),
                'allowed_ref_types' => ['student', 'parent'],
                'access_scope' => 'all',  // تم التصحيح من 'own' إلى 'all'
            ],
            'advanced_filters' => [...],
        ],
    ],
],
```

#### 7. منطق ذكي في متحكم `Months`

تمت إضافة معالجة متقدمة لمعرف السنة والترم:

- إذا لم يُمرر `study_year_id` → استخدام السنة الافتراضية (`Period::getPrimary()`).
- إذا لم يُمرر `semsters_id` ولكن `study_year_id` موجود → جلب الترم الافتراضي لتلك السنة (`Semster::where('study_year_id', $study_year_id)->where('is_default', true)->first()`).
- إذا أُمرر `semsters_id` ولكن `study_year_id` غير موجود → استنتاج `study_year_id` من الترم.
- السماح بـ `semsters_id = '*'` لعدم التصفية حسب الترم.

#### 8. تصحيح `access_scope` للبيانات العامة

في الإصدارات السابقة، كانت إعدادات `frontend` في `list` تستخدم `access_scope = 'own'`، وهو غير مناسب للبيانات العامة (السنوات، الفصول، الأشهر) لأن هذه البيانات لا ترتبط بمستخدم معين. تم تصحيحها إلى `access_scope = 'all'` لتتناسب مع طبيعة البيانات.

---

### أمثلة عملية على الاستخدام الجديد

#### 1. جلب السنوات الدراسية المنشورة مع استخدام فلتر متقدم

```bash
GET /api/v1/studyyear/periods?is_published=1&calendar_type=years&is_or_calendar_type=false
```

#### 2. جلب أشهر سنة معينة وترم معين مع إمكانية تجاهل الترم

```bash
GET /api/v1/studyyear/months?study_year_id=3&semsters_id=5
```
أو لجلب كل أشهر السنة (دون فلترة حسب الترم):
```bash
GET /api/v1/studyyear/months?study_year_id=3&semsters_id=*
```

#### 3. استخدام `is_to_sql` لتصحيح الأخطاء

```php
$result = $controller->getRecords([
    'study_year_id' => 3,
    'is_to_sql' => true,
]);
// ستتم طباعة استعلام SQL في trace_log
```

---

### توافق الإصدارات السابقة

- **جميع نقاط النهاية الحالية** (routes) لم تتغير، لذا لن يتأثر أي تطبيق يستهلك الـ API.
- **المخرجات** لا تزال متوافقة مع الهيكل السابق (مع إضافة `input_data`, `process_data`, `debug` فقط عند الحاجة).
- **لا توجد هجرات قاعدة بيانات** جديدة.
- **التكوين الجديد** اختياري: يمكنك الاستمرار في استخدام الإعدادات القديمة (دون `advanced_filters` أو `frontend_resolver`) وستعمل الإضافة بالسلوك الافتراضي.

---

### متطلبات الترقية (من 1.0.1 إلى 1.0.2)

1. **تحديث الكود**:
   - استبدال جميع ملفات المتحكمات (`Periods.php`, `Semsters.php`, `Months.php`) بالنسخ الجديدة.
   - استبدال ملف `config.php` بالإصدار الجديد الذي يحتوي على أقسام `permission`, `advanced_filters`, `frontend_resolver` (حتى لو كانت فارغة).

2. **تحديث `version.yaml`**:
   - أضف الإصدار `1.0.2` كما هو موضح في بداية هذا المستند.

3. **تنفيذ الهجرات** (لا توجد).

4. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

5. **اختبار الوظائف**:
   - تحقق من أن جميع نقاط النهاية (`periods`, `semsters`, `months`) تعمل كما هو متوقع.
   - اختبر صلاحيات وصول مختلفة (backend, frontend student, frontend parent, guest).
   - تحقق من أن الفلاتر المتقدمة (`is_or`, `is_not`, `is_force`) تعمل على الحقول المحددة في `config.php`.

---

### الفوائد والقيمة المضافة

- **أمان محسّن**: تطبيق نطاقات الصلاحيات تلقائياً وعدم إمكانية تجاوزها بسهولة.
- **مرونة عالية**: يمكن تكوين سلوك كل عملية بشكل مستقل دون تعديل الكود.
- **قابلية التوسع**: بفضل الأحداث، يمكن للإضافات الأخرى تعديل الاستعلامات ديناميكياً.
- **أداء أفضل**: مع دعم التخزين المؤقت واستبعاد الأعمدة غير الضرورية.
- **تجربة مطور أفضل**: جميع المتحكمات تتبع نفس النمط، مما يسهل تعلم وصيانة الكود.
- **دقة أكبر في البيانات**: المنطق الذكي في `Months` يضمن جلب الأشهر الصحيحة حتى لو لم يمرر المستخدم الترم.

---

### الخاتمة

يمثل الإصدار **1.0.2** من `Nano.StudyyearApi` إنجازاً كبيراً نحو توحيد معايير API في نظام Nano البيئي. بفضل إعادة الهيكلة الكاملة وتكامل `AccessManager`، أصبحت الإضافة أكثر أماناً وقابلية للصيانة والتوسع. جميع المتحكمات الآن متوافقة مع دليل `Nano-Api-SKILL.md` وتقدم ميزات متقدمة مثل الفلاتر المتقدمة، الأحداث، والتخزين المؤقت.

نوصي جميع المطورين بالترقية إلى هذا الإصدار والاستفادة من الإعدادات المركزية في `config.php` لتخصيص سلوك API حسب احتياجاتهم.

---

**الوثائق المرجعية**:
- [توثيق إضافة `Nano.StudyyearApi`](./docs/StudyyearApi/Docs-StudyyearApi-ar.md)
- [توثيق إضافة `Tss.Studyyear` ](./docs/Studyyear/Docs-Studyyear-ar.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)
- [توثيق كلاس `AccessManager`](./docs/AuthApi/Docs-AccessManager-ar.md)
- [توثيق كلاس `AdvancedQueryHelper`](./docs/querybuilder/Docs-AdvancedQueryHelper.md)

