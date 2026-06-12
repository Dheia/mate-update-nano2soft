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

## 2026-06-03 – 2026-06-05

**تحديثات إضافة `Nano.TranslateExtended` – الإصدار 1.0.11**  
**تحسين شامل لاستدعاءات API للترجمات ومعالجة المحتوى القابل للترجمة**

### ملخص التحديثات

شهدت إضافة `Nano.TranslateExtended` تحديثاً رئيسياً في الإصدار **1.0.11** ركز على إصلاح مشكلة عدم ظهور الترجمات عند تضمين `translatable_fields` في استجابات API، وتوسيع مرونة جلب الترجمات عبر دعم تحديد الحقول واللغات بشكل ديناميكي من خلال `Input` والإعدادات والخصائص العامة. كما تم تحسين سلوك `TranslatableContentCaching` ليتيح التحكم البرمجي في الحقول القابلة للترجمة، وتصحيح منطق `fallback` للغة الافتراضية، ودعم فك تشفير JSON مع تسجيل الأخطاء، وتحسين التوافق مع هيكل بيانات `RainLab.Translate`.

---

## Nano.TranslateExtended v1.0.11 – تحسين API الترجمات وإصلاحات جوهرية

### أهداف الإصدار

- **إصلاح مشكلة عدم ظهور الترجمات** عند تضمين `translatable_fields` في استدعاءات API.
- **توفير مرونة كاملة في تحديد الحقول واللغات** المراد جلب ترجماتها عبر `Input` أو الإعدادات أو الخصائص العامة.
- **إضافة `includes` جديدة** مثل `translatable_attributes` و `translatable_dirty_locales` لدعم حالات الاستخدام المتقدمة.
- **تحسين `TranslatableContentCaching`** بإضافة دوال ديناميكية للتحكم بالحقول، وتصحيح منطق `fallback`، ودعم فك تشفير JSON مع تسجيل أخطاء.
- **توحيد واجهة تمرير الحقول** كسلسلة نصية مفصولة بفواصل (`title,content`) أو كلمة `all`/`*`.
- **تعزيز التوافق مع هيكل `RainLab.Translate`** في تخزين بيانات الترجمة.

### الميزات الجديدة والتحسينات

---

#### أولاً: تحديثات `DynamicAddIncludeTranslatableApiFields`

##### 1. دالة `includeTranslatableFields` المحسّنة

- **قراءة باراميترات مرنة** وفق الأولوية: الخاصية العامة ← الإعدادات ← `Input` ← القيمة الافتراضية.
- **دعم تحديد الحقول (`fields`) بأشكال متعددة**:
  - مصفوفة عادية.
  - سلسلة مفصولة بفواصل: `"title,content"`.
  - كلمة `"all"` أو `"*"` لجلب جميع الحقول القابلة للترجمة.
- **دعم تحديد اللغات (`locales`)** بنفس المرونة.
- **استدعاء `$item->transCollectFields()`** قبل استدعاء `getTranslationsInFormat()` لضمان تحديث قائمة الحقول ديناميكياً.
- **معالجة `exclude`** من `Input` أو الإعدادات لاستبعاد مفاتيح معينة من النتيجة.

##### 2. إضافة `includes` جديدة إلى الـ Transformer

- `translatable_attributes`: تعيد قائمة الحقول القابلة للترجمة في النموذج (`getTranslatableAttributes`).
- `translatable_dirty_locales` (معلقة اختيارياً): تعيد معلومات عن اللغات المتسخة والنسخ الأصلية.

##### 3. تحسين دالة `getConfigValue`

- إضافة دعم للقراءة من `Input` عبر المفتاح `translated_fields` كبديل عن `translatable_fields.fields`، لتوفير طريقة أبسط للمطورين.

##### 4. مثال استخدام في API

```json
GET /api/orders/orders/1?include=translatable_fields&translatable_fields.fields=title,content&translatable_fields.locales=en,ar
```

أو باستخدام الاختصار:
```json
GET /api/orders/orders/1?include=translatable_fields&translated_fields=title,content
```

---

#### ثانياً: تحديثات `TranslatableContentCaching`

##### 1. إضافة دوال برمجية للتحكم بالحقول

- `getTransCachedFields()`: إرجاع قائمة الحقول القابلة للترجمة الحالية.
- `setTransCachedFields($fields)`: تعيين القائمة ديناميكياً مع دعم السلسلة النصية المفصولة بفواصل أو `'all'`/`'*'`.

##### 2. تحسين دالة `getTranslated`

- إضافة معامل `$useInBackend` (افتراضي `false`) للسماح بترجمة الحقول حتى في بيئة الباكند عند الحاجة.
- تصحيح منطق `fallback` للغة الافتراضية: عند طلب `$locale` المطابق للغة الافتراضية، يتم تعيين `$fallback = true` لضمان الرجوع إلى القيمة الأصلية إذا لم توجد ترجمة.
- إضافة دالة مساعدة `processTranslatableJsonableValue` مع تسجيل أخطاء JSON في وضع التصحيح.

##### 3. تحسين دالة `getTranslatedAttributes`

- جعل البحث عن الترجمة أكثر توافقاً مع بنية `RainLab.Translate` التي تخزن `locale` داخل مصفوفة `attributes`.
- إضافة خطة احتياطية للبحث عبر الخاصية المباشرة `$item->locale`.

##### 4. تحسين دالة `getTranslationsInFormat`

- دعم تمرير `$fields` كسلسلة نصية مفصولة بفواصل أو `'all'`/`'*'`، مع تحويلها إلى مصفوفة داخل الدالة.
- توحيد المنطق مع باقي أجزاء الإضافة.

##### 5. تحسين تسجيل النماذج ديناميكياً (`registerModel`)

- إضافة دالة ثابتة `isPropertyExists` للتحقق من وجود الخاصية قبل إضافتها.
- تجنب إعادة إضافة الخصائص الموجودة مسبقاً.

##### 6. دالة `isPropertyExists` المساعدة

- دالة ثابتة تتحقق من وجود خاصية في كائن مع دعم `propertyExists` (لتوسعات `October\Rain\Extension\ExtensionBase`).

---

### ثالثاً: تحديث `version.yaml`

تمت إضافة الإصدار `1.0.11` مع وصف موجز للتحديثات:

```yaml
1.0.11:
    - Improved translation API includes with flexible field/locale parameters
    - Added support for passing fields as comma-separated string in translatable_fields include
    - Added new includes: translatable_attributes and translatable_dirty_locales
    - Enhanced TranslatableContentCaching with dynamic field setter/getter
    - Improved fallback handling for default locale in getTranslated method
    - Better compatibility with RainLab.Translate attribute data structure
    - Added error logging for JSON decoding issues in debug mode
    - Optimized cache key generation and invalidation
```

---

### أمثلة تطبيقية

#### 1. استخدام النطاقات الجديدة في الـ API (عبر التضمين)

**طلب جلب ترجمات حقلين فقط مع لغة واحدة:**
```http
GET /api/products/products/1?include=translatable_fields&translatable_fields.fields=title,description&translatable_fields.locales=ar
```

**استخدام الاختصار `translated_fields`:**
```http
GET /api/products/products/1?include=translatable_fields&translated_fields=title
```

**جلب جميع الحقول القابلة للترجمة مع استبعاد حقل معين:**
```http
GET /api/products/products/1?include=translatable_fields&translatable_fields.fields=all&translatable_fields.exclude=seo_keywords
```

#### 2. استخدام `includeTranslatableAttributes` للحصول على أسماء الحقول فقط

```http
GET /api/products/products/1?include=translatable_attributes
```

**الاستجابة:**
```json
{
    "data": {
        "id": 1,
        "translatable_attributes": ["title", "description", "seo_title"]
    }
}
```

#### 3. التحكم البرمجي في `TranslatableContentCaching`

```php
// تعيين الحقول القابلة للترجمة ديناميكياً
$order = Order::find(1);
$order->setTransCachedFields('title,content,notes');

// الحصول على الترجمات بتنسيق مخصص
$translations = $order->getTranslationsInFormat(
    fields: 'title',
    format: TranslatableContentCaching::FORMAT_GROUP_BY_FIELD_UNDER_ALL,
    locales: ['en', 'ar'],
    includeOriginal: true,
    useFallback: false
);
```

#### 4. تفعيل الترجمة في الباكند (حالات خاصة)

```php
// فرض الترجمة حتى في بيئة الباكند
$value = $model->getTranslated('title', null, 'ar', true, true, true);
```

---

## ملخص الإصدار (1.0.11)

| الإصدار | أبرز الميزات |
|---------|---------------|
| **Nano.TranslateExtended 1.0.11** | إصلاح مشكلة `translatable_fields` في API، دعم تحديد الحقول واللغات بشكل مرن، إضافة `translatable_attributes` و `translatable_dirty_locales`، تحسين `TranslatableContentCaching` بدوال ديناميكية ومنطق `fallback` للغة الافتراضية، توافق أفضل مع `RainLab.Translate`، تسجيل أخطاء JSON. |

---

### متطلبات الترقية

1. **تحديث الملفات**:
   - `DynamicAddIncludeTranslatableApiFields.php`
   - `TranslatableContentCaching.php`
   - `version.yaml`

2. **لا توجد هجرات جديدة**: الإصدار لا يتطلب تغييرات في قاعدة البيانات.

3. **الاعتماديات**:
   - `RainLab.Translate` (الإصدار الأحدث)
   - `OctoberCMS` ≥ 2.0 أو `WinterCMS`

4. **إعدادات اختيارية**:
   - يمكن إضافة إعدادات مخصصة في ملف `config.php` للإضافة تحت مفتاح `nano.translateextended::translatable_fields` لتحديد قيم افتراضية لـ `fields`, `format`, `locales`, `includeOriginal`, `useFallback`, `useCache`, `exclude`.

5. **اختبار التوافق**:
   - اختبار استدعاء API مع `include=translatable_fields` ومختلف الباراميترات.
   - اختبار النماذج التي تستخدم `TranslatableContentCaching` والتحقق من عمل `getTranslated` في الواجهة والخلفية.
   - اختبار فك تشفير JSON للحقول القابلة للترجمة من نوع JSON.

---

### الخاتمة

يمثل الإصدار **Nano.TranslateExtended 1.0.11** نقلة نوعية في كيفية التعامل مع الترجمات عبر API وفي الكود البرمجي. لم يعد المطور بحاجة إلى كتابة استعلامات معقدة أو تمديد الـ Transformer يدوياً، بل أصبح بإمكانه استخدام `translatable_fields` مع تحديد الحقول واللغات المطلوبة مباشرة عبر الـ `Input`، والحصول على الترجمات بتنسيق نظيف ومرن. كما تم تصحيح العديد من المشاكل السلوكية في `TranslatableContentCaching` لضمان أن الترجمات تعمل كما هو متوقع في جميع السيناريوهات، بما فيها اللغة الافتراضية والرجوع إلى القيم الأصلية. الكود موثق بالكامل، ويتضمن تحسينات في الأداء والتوافق.

---

**الوثائق المرجعية**:
- [توثيق الإضافة `Nano.TranslateExtended`](./docs/TranslateExtended/Behaviors/Docs-Nano.TranslateExtended-ar.md)
- [توثيق سلوك `DynamicAddIncludeTranslatableApiFields`](./docs/TranslateExtended/Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-ar.md)
- [توثيق سلوك `TranslatableContentCaching`](./docs/TranslateExtended/Behaviors/Docs-TranslatableContentCaching-ar.md)
- [دليل استخدام الترجمات المحسّنة في API](./docs/TranslateExtended/Docs-API-Translations-Guide-ar.md)


## 2026-06-01 - 2026-06-06

**تحديثات إضافة `Nano2.Qrcodes` – الإصدار 1.0.3**

### ملخص التحديثات

يأتي الإصدار 1.0.3 ليقدّم نظاماً متكاملاً لإدارة الباركودات والرموز (1D و 2D) داخل برمجيات نانوسوفت (NanoSoft App). بعد أن كان الإصدار 1.0.2 مجرد هيكل أولي للجدول، يضيف هذا الإصدار:

- **موديل `Barcode` كامل** مع سمات متعددة تدعم النطاقات، الخيارات، الكاش، السجلات الافتراضية، والعلاقات المختلفة.
- **متحكم خلفي (Backend)** متكامل يدعم `FormController`، `ListController`، `ReorderController`، و `ImportExportController` مع صلاحيات دقيقة.
- **كلاس `Manager`** مركزي للتعامل مع الباركودات (إنشاء، تحديث، حذف، استخدام، تحقق، إحصائيات) مع دعم حدود الاستخدام اليومية/الأسبوعية/الشهرية عبر طريقتين (Cache و Database).
- **كلاس `BarcodeGenerator`** متطور يدعم 7 مكتبات مختلفة لتوليد الباركودات (Picqer، chillerlan، SimpleSoftwareIO، BaconQrCode، Endroid، Milon، PHP QR Code) مع مخرجات متعددة (PNG، SVG، HTML، Base64، إلخ).
- **واجهة API** RESTful كاملة (CRUD، تحقق، استخدام، إحصائيات، صورة الباركود) مع نظام صلاحيات متقدم عبر `AccessManager`.
- **دعم الاستيراد والتصدير** عبر ملفات CSV مع خيارات مرنة للقيم الافتراضية (قسم، مالك، منتج، وحدة، تخطي قواعد النوع).
- **إعدادات شاملة** عبر ملف `config.php` تشمل صلاحيات لكل عملية، حدود الاستخدام، فلاتر متقدمة، ومحلل ديناميكي لمستخدمي Frontend.

كل ذلك مع توثيق شامل ومفصل بالعربية والإنجليزية في ملفات `lang.php`.

---

### الإصدار 1.0.3 – نظام إدارة الباركودات المتكامل

#### أهداف الإصدار

- **بناء نموذج `Barcode` قوي** يدعم جميع أنواع الباركودات (1D و 2D) مع علاقات متعددة الأشكال (owner، verifier، subject، user) وحقول JSON ديناميكية.
- **توفير متحكم خلفي احترافي** يتبع معايير OctoberCMS ويوفر واجهة مستخدم سهلة لإدارة الباركودات.
- **إنشاء كلاس `Manager`** (Singleton) لتوحيد منطق الأعمال وإعادة استخدامه في الـ API و الـ Backend.
- **تطوير `BarcodeGenerator`** ليكون متعدد المكتبات، مرنًا وقابلًا للتوسع، ويدعم تنسيقات مخرجات مختلفة.
- **بناء واجهة API كاملة** مع دعم التصفية المتقدمة (`is_or`، `is_not`، `is_or_null`)، حدود الاستخدام، والتحقق من الصلاحيات.
- **تمكين استيراد وتصدير الباركودات** بمرونة عالية، مع إمكانية تحديد قيم افتراضية وتخطي قواعد التحقق حسب نوع الباركود.
- **توفير نظام حدود استخدام** (usage limits) للمستخدمين (يومي، ساعي، أسبوعي، شهري، أو عدد أيام مخصص) بطريقتين: Cache (سريع) و Database (دقيق).

#### الميزات الجديدة والتحسينات

##### 1. موديل `Barcode` المتكامل

تم إنشاء موديل `Nano2\Qrcodes\Models\Barcode` مع السمات التالية:

- **السمات الأساسية**: `Validation`، `SoftDelete`، `Purgeable`، `Sortable`.
- **سمات متخصصة**: `HasScopesModel`، `HasDefault`، `HasUserScopes`، `HasUserOptions`، `HasSubjectScopes`، `HasSubjectOptions`، `HasOwnerScopes`، `HasOwnerOptions`، `HasVerifierScopes`، `HasVerifierOptions`، `HasRecordsOptions`، `ListObjects`، `ListOptions`، `FieldsOptions`، `HasCreateDefaultRecords`.
- **العلاقات**:
  - `belongsTo` مع `Company`، `Department`، `Template`، `Product`، `Unit`، والمستخدمين (`Created_by`، `Updated_by`، `Deleted_by`).
  - `morphTo` مع `owner`، `verifier`، `subject`، `user`.
  - `attachOne` و `attachMany` للملفات والصور.
- **حقول JSON**: `fields_data`، `metadata`، `other_data`، `config_data`، `additional_data`.
- **دوال مساعدة**:
  - `isExpired()`، `isExpiringSoon()`، `canUse()`، `use($user)`، `verify($verifier, $score)`.
  - `generateUniqueCode()` – توليد كود فريد بالصيغة `{companys_id}-{departments_id}-{تاريخ_وقت_عشوائي}`.
  - `prepareDuplicate()` – منع التكرار حسب الإعدادات.
  - `applyBarcodeTypeRules()` – تطبيق قواعد تحقق دقيقة لكل نوع باركود (أطوال ثابتة، أرقام فقط، طول زوجي، إلخ).
- **دعم تخطي قواعد النوع**: خاصية `skipBarcodeTypeRules` مع دوال `skipBarcodeTypeRules()`، `enableBarcodeTypeRules()`، `disableBarcodeTypeRules()`، `isBarcodeTypeRulesSkipped()`.
- **نظام الكاش** المتكامل لتحسين الأداء.

##### 2. متحكم الخلفية `Barcodes` (Backend)

تم إنشاء متحكم `Nano2\Qrcodes\Controllers\Barcodes` مع الميزات التالية:

- **السلوكيات**:
  - `FormController`، `ListController`، `ReorderController`، `ImportExportController`.
- **الصلاحيات**: `access`، `access_all`، `add`، `edit`، `delete`، `verify`، `use`، `generate`، `print`، `export`، `import`.
- **دوال CRUD الأساسية**: `onCreate`، `onUpdate`، `onDelete`.
- **الإجراءات الجماعية**:
  - `onActivateSelected` / `onDeactivateSelected` – تفعيل/تعطيل مجموعة.
  - `onVerifySelected` – التحقق الإداري من مجموعة.
  - `onUseSelected` – تسجيل استخدام لمجموعة.
- **الإنشاء الجماعي**:
  - صفحة `generate` مع نموذج لتحديد العدد، البادئة، نوع الباركود، مدة الصلاحية، المنتج، القسم.
  - دالة `onGenerate` لتوليد عدة باركودات دفعة واحدة (حتى 1000).
- **الطباعة**:
  - `onPrint` و `onPrintSelected` لعرض نافذة طباعة تحتوي على الباركود مع بيانات المنتج.
- **التصدير والاستيراد**:
  - `onExport` لتصدير الباركودات إلى CSV.
  - دعم `ImportExportController` مع نموذجي `BarcodeImport` و `BarcodeExport`.
- **خيارات القوائم المنسدلة**:
  - `getStatusOptions`، `getBarcodeTypeOptions`، `getDepartmentsIdOptions`، `getCompanysIdOptions`، `getProductIdOptions`، `getUnitIdOptions`، `getCurrencysIdOptions`، إلخ.
- **السجلات الافتراضية**: `index_onCreateDefaultRecords` و `index_onRestDefaultRecords`.
- **القوائم والإعدادات**: دوال `bootBackendNavigation` و `registerBackendPermissions` لتسجيل القائمة والصلاحيات عبر الأحداث.

##### 3. كلاس `Manager` (إدارة العمليات)

تم إنشاء كلاس `Nano2\Qrcodes\Classes\Manager` (Singleton) ليكون المسؤول الوحيد عن منطق الأعمال:

- **عمليات CRUD**:
  - `createBarcode($options)` – إنشاء باركود مع دعم خيارات متقدمة (تخطي التحقق، تخطي قواعد النوع، توليد صورة، إلخ).
  - `updateBarcode($id, $options)` – تحديث باركود مع منع تعديل المستخدم إن تم استخدامه.
  - `deleteBarcode($id)` – حذف ناعم (soft delete).
  - `restoreBarcode($id)` – استعادة باركود محذوف.
- **الاستخدام والتحقق**:
  - `scanAndUseBarcode($barcodeValue, $user)` – استخدام باركود مع التحقق من صلاحيته وحدود المستخدم.
  - `verifyBarcode($barcodeValue)` – التحقق من الصلاحية دون استخدام.
  - `verifyBarcodeById($id, $verifier, $score)` – توثيق إداري.
- **الإنشاء الجماعي**:
  - `generateMultipleBarcodes($count, $baseOptions)` – إنشاء عدة باركودات.
  - `generateBatch($batchOptions)` – إنشاء دفعة واحدة برقم دفعة موحد.
- **حدود الاستخدام (Usage Limits)**:
  - دعم طريقتين للتتبع: `cache` (سريع) و `database` (دقيق).
  - فترات مرنة: `hour`، `day`، `week`، `month`، أو عدد أيام مخصص.
  - دوال `checkUserUsageLimit` و `incrementUserUsage` و `getUserUsageCountFromDatabase` و `getUsageLimitValue`.
- **إحصائيات**:
  - `getBarcodeStats($options)` – إحصائيات عامة (إجمالي، نشط، منتهي، حسب النوع، إلخ).
  - `getUserBarcodeUsageStats($user)` – إحصائيات استخدام المستخدم.
- **عمليات مساعدة**:
  - `getBarcodeRecords($options)` – استرجاع السجلات مع خيارات تصفية.
  - `exportBarcodes($options)` – تصدير البيانات.
  - `importBarcodes($data, $options)` – استيراد البيانات.
  - `getBarcodeImageUrl($barcode)` – الحصول على رابط صورة الباركود.
  - `formatBarcodeForApi($barcode)` – تنسيق البيانات للـ API.

##### 4. كلاس `BarcodeGenerator` (توليد الباركودات)

تم إنشاء كلاس متطور يدعم سبع مكتبات مختلفة مع كشف تلقائي:

| المكتبة | الأنواع المدعومة | الأولوية |
| :--- | :--- | :--- |
| Picqer\Barcode | 1D و 2D | 1 (الأعلى) |
| chillerlan\QRCode | QR فقط | 2 |
| SimpleSoftwareIO\QrCode | QR فقط | 3 |
| PHP QR Code (دالة `qrcode`) | QR فقط | 4 |
| BaconQrCode | QR فقط | 5 |
| Endroid\QrCode | QR فقط | 6 |
| Milon\Barcode (Tss) | 1D و 2D | 7 |
| Fallback (GD) | نص بسيط | الأخير |

**الميزات**:
- **تنسيقات المخرجات**: `png`، `jpg`، `jpeg`، `webp`، `svg`، `html`، `base64`، `data-uri`، `gd`، `response`، `file`.
- **خيارات متقدمة**: العرض، الارتفاع، اللون، لون الخلفية، الهامش، مستوى تصحيح الخطأ، الجودة، الشفافية، الكاش.
- **دوال خاصة**:
  - `generate($data, $type, $options)` – الدالة الرئيسية.
  - `generateBarcodeImage($data, $type, $options)` – توليد صورة PNG.
  - `generateAndSaveBarcodeImage($barcode, $path)` – حفظ الصورة في التخزين وتحديث النموذج.
  - `validateBarcode($barcode, $type)` – التحقق من صحة قيمة الباركود حسب النوع (يدعم EAN، UPC، ISBN، Code 128/39/93، I25، POSTNET، إلخ).
- **دوال مساعدة**: `hexToRgb`، `hexToGdColor`، `getAvailableLibraries`، `selectLibrary`.

##### 5. واجهة API (RESTful)

تم إنشاء متحكم `Nano2\Qrcodes\ApiControllers\Barcodes` مع نقاط النهاية التالية:

| الطريقة | المسار | الوصف |
| :--- | :--- | :--- |
| GET | `/barcodes` | جلب قائمة الباركودات مع تصفية وترقيم. |
| GET | `/barcodes/activelystats` | آخر توقيت تحديث (للكاش). |
| POST | `/barcodes` | إنشاء باركود جديد. |
| PUT | `/barcodes/{id}` | تحديث باركود موجود. |
| DELETE | `/barcodes/{id}` | حذف باركود (ناعم). |
| GET | `/barcodes/{id}` | عرض تفاصيل باركود. |
| POST | `/barcodes/verify` | التحقق من صحة باركود دون استخدام. |
| POST | `/barcodes/use` | مسح واستخدام باركود (تسجيل استخدام). |
| POST | `/barcodes/verify/{id}` | التحقق الإداري (تعيين `is_verified`). |
| POST | `/barcodes/generate` | توليد دفعة من الباركودات (للمشرفين فقط). |
| GET | `/barcodes/stats` | إحصائيات عامة للباركودات. |
| GET | `/barcodes/user-stats` | إحصائيات استخدام المستخدم الحالي. |
| GET | `/barcodes/image/{id}` | الحصول على صورة الباركود (PNG). |

**نظام الصلاحيات**:
- كل عملية (`list`، `show`، `create`، `update`، `delete`، `verify`، `use`، `generate`، `stats`، `userStats`) لها إعدادات صلاحيات مستقلة في `config.php`.
- تدعم الفلاتر المتقدمة (`is_or`، `is_not`، `is_or_null`) عبر `advanced_filters`.
- محلل ديناميكي (frontend resolver) لتعبئة `user_id` و `user_type` تلقائياً لمستخدمي frontend.

**الاستجابات**:
- تتبع هيكل `Nano\API\Classes\ApiController` مع الحقول: `code`، `status`، `message`، `error`، `errors`، `data`، `input_data`، `process_data`، `debug`.

##### 6. الاستيراد والتصدير

تم إضافة نموذجين:

- **`BarcodeImport`**:
  - يدعم خيارات: `update_existing`، `skip_barcode_type_rules` (ثلاث حالات: `null`، `true`، `false`)، `default_status`، `default_barcode_type`، `default_departments_id`، `default_owner_type` و `default_owner_id`، `default_product_id` و `default_product_name`، `default_unit_id`.
  - دوال خيارات لبناء واجهة الاستيراد (`getSkipBarcodeTypeRulesOptions`، `getDefaultDepartmentsIdOptions`، إلخ).
  - دالة `importData` التي تقوم بمعالجة كل صف وتستدعي `Manager::createBarcode` أو `updateBarcode` مع تمرير خيار `skip_barcode_type_rules`.
- **`BarcodeExport`**:
  - يقوم بتصدير البيانات حسب الأعمدة المحددة في `columns.yaml`، مع فلترة اختيارية حسب صلاحية المستخدم.

##### 7. الإعدادات المركزية (config.php)

تم إنشاء ملف `config.php` غني بالإعدادات:

- **الإعدادات العامة**:
  - `allow_debug_any` – تفعيل وضع التصحيح عبر GET parameters.
  - `api.enable_cache` و `api.cache_ttl` – تفعيل الكاش للـ API.
- **إعدادات `barcodes`**:
  - `is_default_company`، `department_type`، `is_check_duplicate`، `is_show_create_default`، `is_stop_show_menu`، إلخ.
  - `image_width`، `image_height`، `image_color`، `image_format`، `storage_disk` – إعدادات صورة الباركود.
  - **`usage_limits`**:
    - `enabled`، `daily_limit`، `hourly_limit`، `reset_at_midnight`.
    - `tracking_type` (`cache` أو `database`).
    - `tracking_period` (`hour`، `day`، `week`، `month`، أو عدد أيام).
    - `tracking_custom_days`، `use_soft_limit`، `limit_message`.
  - **إعدادات الصلاحيات لكل عملية** (`list`، `show`، `create`، `update`، `delete`، `verify`، `use`، `generate`، `stats`، `userStats`):
    - `permission.is_allow`، `permission.backend`، `permission.frontend`، `permission.guest`.
    - `advanced_filters` و `frontend_resolver` لكل عملية (قابلة للتخصيص عبر متغيرات البيئة).

##### 8. الترجمة والدعم متعدد اللغات

تم إضافة ملفات `lang/ar/lang.php` و `lang/en/lang.php` تغطي:

- الأيقونات، القوائم، الصلاحيات، الرسائل، الأخطاء، الفلاتر، حقول النماذج، أعمدة القوائم، مساعدات الاستيراد/التصدير، إلخ.
- مفاتيح خاصة بخيارات `skip_barcode_type_rules` في قسم الاستيراد.
- ترجمة لأنواع الباركودات المتعددة المستخدمة في `getBarcodeTypeOptions`.

---

### أمثلة عملية

#### 1. إنشاء باركود عبر API

```bash
curl -X POST "https://domain.com/api/v1/qrcodes/barcodes" \
  -H "Content-Type: application/json" \
  -d '{
    "barcode": "5901234123457",
    "barcode_type": "EAN13",
    "product_name": "منتج تجريبي",
    "price": 99.99
  }'
```

#### 2. مسح واستخدام باركود

```bash
curl -X POST "https://domain.com/api/v1/qrcodes/barcodes/use" \
  -H "Content-Type: application/json" \
  -d '{"barcode": "BC000101"}'
```

#### 3. الاستيراد مع تخطي قواعد النوع (في واجهة الخلفية)

في نموذج استيراد الباركودات، اختر من قائمة `skip_barcode_type_rules` القيمة `true` إذا كنت تستورد باركودات EAN-13 ليست بطول 13 رقمًا، أو `false` لتطبيق القواعد، أو `null` للإعداد الافتراضي.

#### 4. استخدام `Manager` مباشرة

```php
use Nano2\Qrcodes\Classes\Manager;

// إنشاء باركود جديد
$result = Manager::createBarcode([
    'barcode' => '1234567890',
    'barcode_type' => 'C128',
    'product_name' => 'جهاز جديد',
]);

if ($result['status']) {
    $barcode = $result['model'];
    echo $barcode->code;
}

// استخدام باركود
$useResult = Manager::scanAndUseBarcode('1234567890', $currentUser);

// إحصائيات المستخدم
$stats = Manager::getUserBarcodeUsageStats($currentUser);
```

#### 5. توليد صورة باركود باستخدام `BarcodeGenerator`

```php
use Nano2\Qrcodes\Classes\BarcodeGenerator;

$pngData = BarcodeGenerator::generateBarcodeImage('5901234123457', 'EAN13');
header('Content-Type: image/png');
echo $pngData;
```

---

### متطلبات الترقية

1. **تحديث قاعدة البيانات**  
   يجب تشغيل الترحيلات الجديدة (إن لم تكن مشغلة):

   ```bash
   php artisan october:migrate
   ```

   وهذا سيُنشئ جدول `nano2_qrcodes_barcodes` (وقد يكون موجوداً مسبقاً في الإصدار 1.0.2).

2. **تحديث الكود**  
   استبدل الملفات التالية بالنسخ الجديدة:
   - `models/Barcode.php` (مع جميع السمات في مجلد `barcode/`).
   - `controllers/Barcodes.php` (الخلفي).
   - `apicontrollers/Barcodes.php` (API).
   - `transformers/BarcodeTransformer.php`.
   - `classes/Manager.php`.
   - `classes/BarcodeGenerator.php`.
   - `classes/QrcodeManagement.php` (إن وجد).
   - `config/config.php`.
   - `routes.php`.
   - `updates/version.yaml`.
   - ملفات اللغة `lang/ar/lang.php` و `lang/en/lang.php`.
   - ملفات YAML الخاصة بالنماذج (`models/barcode/fields.yaml`, `columns.yaml`، إلخ).
   - ملفات العرض والتهيئة الخاصة بالمتحكم الخلفي (`controllers/barcodes/*.htm`, `config_*.yaml`).

3. **تسجيل القوائم والصلاحيات**  
   تأكد من أن `Plugin.php` يستدعي `Barcodes::registerBackendPermissions()` و `Barcodes::bootBackendNavigation()` داخل دالة `boot()`.

4. **إعداد متغيرات البيئة** (اختياري)  
   يمكن وضع المتغيرات التالية في ملف `.env` لتخصيص السلوك:

   ```ini
   NANO2_QRCODES_BARCODES_USAGE_LIMITS_ENABLED=true
   NANO2_QRCODES_BARCODES_USAGE_LIMITS_DAILY_LIMIT=10
   NANO2_QRCODES_BARCODES_USAGE_LIMITS_TRACKING_TYPE=database
   NANO2_QRCODES_BARCODES_LIST_BACKEND_ALLOW=true
   NANO2_QRCODES_API_ENABLE_CACHE=false
   NANO2_QRCODES_BARCODE_IMAGE_WIDTH=300
   ```

5. **اختبار الوظائف**  
   - تحقق من ظهور قائمة `الباركودات` في لوحة التحكم.
   - جرب إنشاء باركود جديد وتعديله وحذفه.
   - اختبر الإنشاء الجماعي والطباعة والتصدير.
   - جرب نقاط API باستخدام Postman أو `curl`.
   - تأكد من أن حدود الاستخدام تعمل كما هو مطلوب (يمكن اختبارها عبر `use` endpoint).

6. **مسح الكاش**  
   لتطبيق الإعدادات الجديدة:

   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### الخاتمة

الإصدار 1.0.3 هو إصدار أساسي ومتكامل لإدارة الباركودات في منصة نانوسوفت. يوفّر موديلاً غنياً، متحكماً خلفياً كاملاً، واجهة API حديثة، كلاس `Manager` مرن، ومولد باركودات متعدد المكتبات. كما يدعم الاستيراد والتصدير بمرونة عالية، ونظام حدود استخدام متقدم يمكن تخصيصه حسب متطلبات العمل.

نشكركم على متابعتكم، ونرحب بملاحظاتكم واقتراحاتكم لتحسين الإضافة في الإصدارات القادمة.

---

**الوثائق المرجعية** 
- [التوثيق العام للإضافة](./docs/Qrcodes/Docs-Nano2-Qrcodes-ar.md)
- [توثيق موديل `Barcode`](./docs/Qrcodes/Docs-Barcode-Model-ar.md)
- [توثيق كلاس `Manager`](./docs/Qrcodes/Docs-Manager-Class-ar.md)
- [توثيق كلاس `BarcodeGenerator`](./docs/Qrcodes/Docs-BarcodeGenerator-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Qrcodes/Docs-API-Documentation-ar.md)

## 2026-06-07 - 2026-06-08

### تحديث الإصدار: Tss.Webbasic (v1.0.15) و Nano.BasicApi (v1.0.22)

**المطور:** Dheia Al-Shami  

---

### ملخص التحديث

تم إضافة حقل جديد من نوع `repeater` باسم `app_links` في إعدادات الثيم (Theme Settings) وذلك لإدارة روابط التطبيقات (مثل متجر جوجل، آب ستور، وغيرها). كما تم دعم هذا الحقل في واجهة برمجة التطبيقات (API) الخاصة بـ `Nano.BasicApi` بحيث يتم إرجاع قيمة الحقل مع معالجة حقل الصورة (`image`) لتحويله إلى رابط كامل باستخدام `MediaLibrary`.

---

### التغييرات في Tss.Webbasic (v1.0.15)

#### 1. إضافة حقل `app_links` إلى واجهة إعدادات الثيم

- **الموقع:** `CmsManager.php` (دالة `extendCmsThemeConfig`)
- **نوع الحقل:** `repeater`
- **الحقول الفرعية:**
  - `name` (نص، مطلوب)
  - `link` (نص، مطلوب)
  - `icon` (أيقونات من Awesome Icons)
  - `image` (ملف وسائط عبر MediaFinder)
  - `is_active` (مفتاح تشغيل/إيقاف)

- **التبويب:** `tss.webbasic::lang.theme.tab.shop`

#### 2. إضافة مفاتيح الترجمة (lang)

تم إضافة مفاتيح جديدة في `lang/ar/lang.php` و `lang/en/lang.php`:

##### اللغة العربية (`ar`)

```php
'theme' => [
    'form' => [
        'app_links_label' => 'روابط التطبيقات',
        'app_links_prompt' => 'إضافة رابط جديد',
        'app_links_name_label' => 'الاسم',
        'app_links_name_placeholder' => 'أدخل اسم الرابط',
        'app_links_link_label' => 'الرابط',
        'app_links_link_placeholder' => 'https://...',
        'app_links_icon_label' => 'الأيقونة',
        'app_links_icon_placeholder' => 'اختر الأيقونة',
        'app_links_image_label' => 'الصورة',
        'app_links_is_active' => 'مفعل',
    ],
    'tab' => [
        'shop' => 'إعدادات المتجر', // إذا لم يكن موجوداً
    ],
],
```

##### اللغة الإنجليزية (`en`)

```php
'theme' => [
    'form' => [
        'app_links_label' => 'App Links',
        'app_links_prompt' => 'Add new link',
        'app_links_name_label' => 'Name',
        'app_links_name_placeholder' => 'Enter link name',
        'app_links_link_label' => 'URL',
        'app_links_link_placeholder' => 'https://...',
        'app_links_icon_label' => 'Icon',
        'app_links_icon_placeholder' => 'Select icon',
        'app_links_image_label' => 'Image',
        'app_links_is_active' => 'Active',
    ],
    'tab' => [
        'shop' => 'Shop Settings', // إذا لم يكن موجوداً
    ],
],
```

#### 3. تحديث ملف الإصدارات `version.yaml`

```yaml
1.0.15:
    - 'Add app_links repeater field to theme settings with translation support'
    - 'Add Arabic and English translations for app_links fields'
```

---

### التغييرات في Nano.BasicApi (v1.0.22)

#### 1. دعم حقل `app_links` في واجهة API

- **الملف المعدل:** `APIControllers/Themes.php`
- **الدالة:** `index()`

##### أ. إضافة معالجة `app_links` عند وجوده في إعدادات الثيم

```php
if(isset($customData['app_links']) && !empty($customData['app_links'])) {
    $app_links = $customData['app_links'];
    $app_links = array_map(function($item) {
        $item['image'] = isset($item['image']) ? self::getUrlMediaLibrary($item['image']) : null;
        return $item;
    }, $app_links);
    
    $customData['app_links'] = $app_links;
}

if(!isset($customData['app_links']))
    $customData['app_links'] = null;
```

##### ب. تحديث دالة `getUrlMediaLibrary`

الدالة موجودة مسبقاً وتقوم بتحويل مسار الوسائط إلى رابط كامل. لم يتم تعديلها ولكنها تستخدم لمعالجة الصور في `app_links`.

#### 2. تحديث ملف الإصدارات `version.yaml`

```yaml
1.0.22:
    - 'Add support for app_links field in theme settings API endpoint'
    - 'Process image field in app_links using MediaLibrary URL'
```

---

### كيفية الاستخدام

#### في لوحة التحكم (إعدادات الثيم)

1. انتقل إلى `Customize → Theme` (أو `الإعدادات → الثيم` حسب اللغة).
2. اذهب إلى تبويب **"إعدادات المتجر"** (أو `Shop Settings`).
3. ستجد حقل جديد باسم **"روابط التطبيقات"** أو `App Links`.
4. يمكنك إضافة عدة روابط، كل رابط يحتوي على:
   - الاسم (مطلوب)
   - الرابط URL (مطلوب)
   - أيقونة (اختياري)
   - صورة (اختياري)
   - تفعيل/إلغاء التفعيل

#### عبر API

- **نقطة النهاية:** `GET /api/v1/basic/settings`
- **الاستجابة:** ستظهر `app_links` كمصفوفة من الكائنات، مع معالجة حقل `image` ليكون رابطاً كاملاً.

**مثال للاستجابة:**

```json
{
    "app_links": [
        {
            "name": "رابط تحميل التطبيق من متجر جوجل",
            "link": "https://play.google.com/store/apps/details?id=com.nano2soft.now",
            "icon": "fab fa-google-play",
            "image": "https://example.com/storage/app/media/GooglePlay.svg",
            "is_active": "1"
        }
    ]
}
```

---

### التوافق

- **يتطلب وجود** `GinoPane.AwesomeSocialLinks` لدعم نوع الحقل `awesomeiconslist` (اختياري، إذا لم يكن موجوداً سيظهر خطأ في الواجهة).
- **يتطلب NanoSoft App v2.x أو v3.x**
- **الحد الأدنى لإصدار PHP:** 7.2

---

### ملاحظات للمطورين

- حقل `app_links` يُخزن بصيغة JSON في نموذج `ThemeData`، لذا لا حاجة لإنشاء جدول جديد.
- لضمان عمل الحقل بشكل صحيح، تأكد من إضافة `app_links` إلى قائمة `jsonable` في نموذج `ThemeData` (يمكن إضافته عبر `extend` في `boot()`).
- تم استخدام نفس أسلوب معالجة الصور المطبق على `social_links`.

---

### تاريخ الإصدارات السابقة

- **Tss.Webbasic v1.0.14:** دعم SiteSearch لمحتوى المنشورات والمشاريع.
- **Nano.BasicApi v1.0.21:** (في حال وجود إصدار سابق)

---

### التحديث المقترح للمشاريع القائمة

1. قم بتحديث الإضافات عبر الأمر:
   ```bash
   php artisan plugin:refresh Tss.Webbasic
   php artisan plugin:refresh Nano.BasicApi
   ```
2. مسح ذاكرة التخزين:
   ```bash
   php artisan cache:clear
   ```
3. إذا كنت تستخدم الثيم الخاص بك، تأكد من أن واجهة إعدادات الثيم تدعم الحقل الجديد.

---

**تم التحديث بواسطة:** Dheia Al-Shami  
**للاستفسارات:** info@nano2soft.com

