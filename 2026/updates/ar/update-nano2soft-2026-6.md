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

## 2026-06-05 - 2026-06-09

**Update to `Nano.Orders` – Version 2.2.12 and `Nano.OrdersApi` – Version 1.0.22**

### تحديث إضافة `Nano.Orders` – الإصدار 2.2.12  
### تحديث إضافة `Nano.OrdersApi` – الإصدار 1.0.22

---

### ملخص التحديثات

أضاف الإصدار **2.2.12** من `Nano.Orders` دعماً متكاملاً لإدارة الصور المرتبطة بالطلب، مع أربع علاقات جديدة من نوع `attachMany`، وسمة `StepPhotos` التي توفر دوال احترافية لرفع الصور وحذفها والتحقق من صلاحيات المستخدمين (المالك، الموصل، المدير) بناءً على حالة الطلب الحالية. تم أيضاً ربط النظام بـ `Nano.FileUpload` لتوحيد عملية رفع الملفات، مع إضافة إعدادات مرنة (`photo_rules`) للتحكم بكل حقل، وإطلاق أحداث جديدة، ودعم خيار `skip_photo_check` في دالة `updateOrderStatusAdvanced` لمنع تغيير الحالة دون الصور المطلوبة.

في المقابل، وفر الإصدار **1.0.22** من `Nano.OrdersApi` ثلاث نقاط نهاية API لرفع (فردي/متعدد) وحذف الصور، مع تكامل كامل مع `StepPhotos` والتحقق من الصلاحيات، وإرجاع روابط الصور والمصغرات في الاستجابة.

---

## Nano.Orders v2.2.12 – إدارة صور الطلبات (StepPhotos)

### أهداف الإصدار

- **تمكين رفع الصور** قبل وبعد التسليم من قبل كل من المستأجر (`user_id`) والمؤجر (`delivery_user_id`).
- **ربط الصور بحالات الطلب** مثل `NEW`, `PROCESSING`, `DELIVERY`, `COMPLETE` لضمان رفعها في الوقت المناسب.
- **منع تغيير حالة الطلب** (`DELIVERY` أو `COMPLETE`) ما لم تكن الصور المطلوبة قد رُفعت.
- **توفير دوال مرنة** تعتمد على `Nano.FileUpload` وتدعم رفع الملفات العادية أو `base64`.
- **إطلاق أحداث** لتوسيع السلوك (إشعارات، تسجيل، إلخ).
- **تمكين المطورين من تخصيص القواعد** (الحالات المسموحة، الأدوار، الحد الأقصى للصور، أنواع الملفات) عبر الإعدادات.

### الميزات الجديدة

#### 1. علاقات `attachMany` في نموذج `Order`

أضيفت أربع علاقات جديدة داخل `Order.php`:

```php
public $attachMany = [
    'user_before_delivery'     => 'System\Models\File',
    'user_after_delivery'      => 'System\Models\File',
    'delivery_before_delivery' => 'System\Models\File',
    'delivery_after_delivery'  => 'System\Models\File',
];
```

| العلاقة | الوصف | المستخدم المخوّل |
|---------|-------|------------------|
| `user_before_delivery` | صور المستأجر قبل التسليم | المالك (`user_id`) |
| `user_after_delivery`  | صور المستأجر بعد التسليم | المالك |
| `delivery_before_delivery` | صور المؤجر قبل التسليم | الموصل (`delivery_user_id`) |
| `delivery_after_delivery`  | صور المؤجر بعد التسليم | الموصل |

#### 2. تسجيل الحقول في `FileUploadRegistry`

تم تسجيل هذه الحقول تلقائياً في `Plugin.php` عبر دالة `registerOrderFileUploadFields`، مع الإعدادات التالية:

- نوع الحقل: `image` (يدعم الصور فقط)
- الحد الأقصى للحجم: 5 ميجابايت (قابل للتعديل)
- الأنواع المسموحة: `jpg, jpeg, png, gif`
- `multiple => true` (لأنها علاقات `attachMany`)
- `max_files => 10` (عدد الصور المسموحة)
- `is_public => false` (خاصة، يتم الوصول إليها عبر رابط مؤقت)

#### 3. ترايت `StepPhotos` (ملف `traits/steps/StepPhotos.php`)

هذا الترايت يضم جميع دوال إدارة الصور:

| الدالة | الوصف |
|--------|-------|
| `uploadOrderPhoto($field, $fileData, array $options)` | رفع صورة واحدة (دعم `UploadedFile` أو `base64`). |
| `uploadOrderPhotos($field, array $filesData, array $options)` | رفع عدة صور دفعة واحدة. |
| `deleteOrderPhoto($field, $fileId, array $options)` | حذف صورة موجودة. |
| `validatePhotoUploadPermission($field, $user, $order, $newStatus, array $options)` | التحقق من صلاحية المستخدم للرفع (الدور، الحالة المسموحة، الحد الأقصى). |
| `canUploadPhoto($field, $user, $order)` | فحص سريع بدون استثناء. |
| `canUploadPhotoToField($field, $user, $order, $newStatus)` | نفس السابق مع دعم الحالة الجديدة. |
| `enforcePhotoRequirementsOnStatusChange($oldStatus, $newStatus, $order, $actor, array $options)` | تُستدعى قبل تغيير الحالة للتأكد من وجود الصور المطلوبة. |
| `getOrderPhotoUrls($field, array $thumbSizes, $includeModel)` | جلب روابط الصور مع صور مصغرة مخصصة. |
| `hasRequiredPhotosForStatus($newStatus, $order, $role)` | فحص سريع للمتطلبات. |

**آلية التحقق من الصلاحية** (`validatePhotoUploadPermission`):
- يحدد دور المستخدم ( `user` / `delivery` / `admin` ) بالنسبة للطلب.
- يقرأ القواعد (`allowed_roles`, `allowed_statuses`, `max_files`) من الإعدادات أو الخيارات الممررة.
- يتحقق من أن الحقل يسمح بهذا الدور، ومن أن الحالة الحالية للطلب مسموحة للرفع، ومن أن عدد الصور الحالي لم يصل إلى الحد الأقصى.
- إذا تم تمرير `$newStatus`، يتحقق مما إذا كان الحقل مطلوباً للانتقال إلى تلك الحالة.

#### 4. دعم `skip_photo_check` في `updateOrderStatusAdvanced`

تم تعديل دالة `updateOrderStatusAdvanced` (في ترايت `StepStatus`) لدعم خيارين جديدين:

- `skip_photo_check` (bool، افتراضي `false`) – تجاوز فحص الصور.
- `photo_validation_rules` (array) – قواعد مخصصة لتجاوز القواعد الافتراضية.

عند محاولة تغيير الحالة، تستخرج الدالة الحالة الجديدة الفعلية، ثم تستدعي `enforcePhotoRequirementsOnStatusChange` ما لم يكن `skip_photo_check = true`. إذا كانت الصور المطلوبة مفقودة، تُرمى استثناء وتمنع التغيير.

#### 5. إعدادات `photo_rules` في ملف `config/nano/orders.php`

أضيف قسم كامل يمكن تخصيصه عبر متغيرات البيئة أو التعديل المباشر:

```php
'photo_rules' => [
    'user_before_delivery' => [
        'allowed_statuses' => ['NEW', 'PROCESSING'],
        'allowed_roles'    => ['user', 'admin'],
        'required_on_transition' => ['DELIVERY'],
        'max_files'        => 10,
        'allowed_types'    => 'jpg,jpeg,png,gif',
        'max_size'         => 5120,
        'description'      => 'صور المستأجر قبل التسليم',
    ],
    // ... باقي الحقول
],
```

متغيرات البيئة المقابلة:
```ini
NANO_ORDERS_PHOTO_USER_BEFORE_STATUSES="NEW,PROCESSING"
NANO_ORDERS_PHOTO_USER_BEFORE_ROLES="user,admin"
NANO_ORDERS_PHOTO_USER_BEFORE_REQUIRED="DELIVERY"
NANO_ORDERS_PHOTO_USER_BEFORE_MAX_FILES=10
NANO_ORDERS_PHOTO_USER_BEFORE_ALLOWED_TYPES="jpg,jpeg,png,gif"
NANO_ORDERS_PHOTO_USER_BEFORE_MAX_SIZE=5120
```

#### 6. الأحداث الجديدة

- `nano.orders.photo.uploaded` – يُطلق بعد رفع صورة بنجاح، يمرر `order`, `field`, `file`, `actor`.
- `nano.orders.photo.deleted` – يُطلق بعد حذف صورة بنجاح، يمرر `order`, `field`, `file_id`, `actor`.

#### 7. مفاتيح الترجمة

أضيفت مفاتيح جديدة تحت `manager.photo` و `orders.photo` في ملفات اللغة (`lang/ar/lang.php`، `lang/en/lang.php`)، لتغطية رسائل النجاح والخطأ والتحقق.

---

## Nano.OrdersApi v1.0.22 – نقاط API لإدارة الصور

### أهداف الإصدار

- **توفير واجهة API لرفع وحذف الصور**، بنفس أسلوب `update-status`.
- **دمج كامل مع `StepPhotos`** و `OrderManager` لضمان تطبيق نفس قواعد الصلاحية والحالات.
- **دعم رفع الملفات عبر `multipart/form-data` أو `base64`**.
- **إرجاع روابط الصور والمصغرات** مباشرة في الاستجابة لتسهيل عرضها في التطبيقات.

### الميزات الجديدة

#### 1. نقاط النهاية الثلاث

| الطريقة | المسار | الوصف |
|---------|--------|-------|
| `POST` | `/api/v1/orders/orders/upload-photo` | رفع صورة واحدة. |
| `POST` | `/api/v1/orders/orders/upload-multiple-photos` | رفع عدة صور (مصفوفة). |
| `POST` | `/api/v1/orders/orders/delete-photo` | حذف صورة (تستقبل `order_id`, `field`, `file_id`). |
| `DELETE` | `/api/v1/orders/orders/delete-photo/{orderId}/{field}/{fileId}` | حذف صورة عبر معاملات المسار (بديل). |

#### 2. دوال المتحكم `Orders`

أضيفت الدوال التالية داخل `Orders.php` (بنفس نمط `updateStatus`):

- `onUploadPhoto($data, $is_array)`
- `onUploadMultiplePhotos($data, $is_array)`
- `onDeletePhoto($data, $is_array)`
- `uploadPhoto()` – نقطة عامة تستدعي `onUploadPhoto`.
- `uploadMultiplePhotos()`
- `deletePhoto()` – نقطة عامة تستدعي `onDeletePhoto`.

**معاملات رفع صورة واحدة**:
- `order_id` أو `id` (مطلوب)
- `field` (مطلوب، واحد من الحقول الأربعة)
- `file` (ملف) أو `file_base64` (نص base64)
- `title`, `description`, `is_public`, `auto_resize`, إلخ (اختيارية)

**استجابة النجاح**:
```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع الصورة بنجاح",
    "data": {
        "id": 456,
        "path": "https://.../image.jpg",
        "thumb": "https://.../thumb.jpg",
        "photos": {
            "thumb": "https://.../thumb.jpg",
            "medium": "https://.../medium.jpg",
            "large": "https://.../large.jpg"
        }
    }
}
```

#### 3. التعامل مع الملفات المرفوعة (`multipart/form-data`)

تم تحسين الدوال لاستخراج الملف مباشرة باستخدام `Input::hasFile()` و `Input::file()`، مع الاحتفاظ بدعم `base64` للبيئات التي لا ترفع ملفات مباشرة.

#### 4. دمج الصلاحيات بشكل كامل

قبل رفع أي صورة، يتم استدعاء `validatePhotoUploadPermission` (داخل `uploadOrderPhoto` في `StepPhotos`). إذا فشل التحقق (دور غير مسموح، حالة غير مسموحة، بلوغ الحد الأقصى)، تُرمى استثناء وتعرض رسالة خطأ مناسبة.

#### 5. إرجاع روابط الصور

بعد رفع الصورة، يتم استدعاء `getOrderPhotoUrls` لتوليد رابط الصورة الأصلية بالإضافة إلى صور مصغرة بأحجام افتراضية (`thumb`, `medium`, `large`). يمكن تخصيص الأحجام عبر `thumb_sizes` في الخيارات.

---

## أمثلة استخدام

### رفع صورة واحدة (multipart/form-data)

```http
POST /api/v1/orders/orders/upload-photo
Authorization: Bearer ...
Content-Type: multipart/form-data

order_id: 123
field: user_before_delivery
file: @/path/to/photo.jpg
title: صورة قبل التسليم
```

### رفع عدة صور (base64)

```json
POST /api/v1/orders/orders/upload-multiple-photos
{
    "order_id": 123,
    "field": "delivery_after_delivery",
    "files": [
        "data:image/jpeg;base64,...",
        "data:image/jpeg;base64,..."
    ]
}
```

### حذف صورة

```http
DELETE /api/v1/orders/orders/delete-photo/123/user_before_delivery/456
```

### محاولة تغيير حالة الطلب إلى DELIVERY دون رفع الصور المطلوبة

سيتم رفض الطلب واستقبال رسالة:
```json
{
    "code": 400,
    "status": false,
    "message": "يجب رفع الصور في حقل user_before_delivery قبل تغيير الحالة إلى DELIVERY"
}
```

### تجاوز فحص الصور (للمسؤول)

```php
$options = [
    'order' => $order,
    'order_states_ref_type' => 'COMPLETE',
    'skip_photo_check' => true,
    'admin_override' => true
];
$orderManager->updateOrderStatusAdvanced($options);
```

---

## متطلبات الترقية

### لإضافة `Nano.Orders`

1. **تحديث الملفات**:
   - `models/Order.php` – إضافة العلاقات الجديدة.
   - `traits/steps/StepPhotos.php` – إضافة الترايت بالكامل.
   - `Plugin.php` – إضافة `registerOrderFileUploadFields` واستدعائها في `boot()`.
   - `config/nano/orders.php` – إضافة قسم `photo_rules` (اختياري).
   - `lang/ar/lang.php` و `lang/en/lang.php` – إضافة مفاتيح الترجمة الجديدة.

2. **لا توجد هجرات جديدة** – العلاقات من نوع `attachMany` لا تحتاج إلى أعمدة إضافية في جدول `nano_orders_orders`.

3. **الاعتماديات**:
   - يجب أن تكون إضافة `Nano.FileUpload` مثبتة ومهيأة (الإصدار 1.0.7 أو أحدث).
   - إضافة `Nano.FileUpload` تعتمد على `Nano.Api` (لتوفير `AuthHelpers`).

4. **الإعدادات**:
   - يمكن ترك القيم الافتراضية أو تخصيصها عبر `.env` أو التعديل المباشر.

### لإضافة `Nano.OrdersApi`

1. **تحديث الملفات**:
   - `APIControllers/Orders.php` – إضافة الدوال الجديدة.
   - `routes.php` – إضافة المسارات الجديدة.
   - `lang/ar/lang.php` و `lang/en/lang.php` – إضافة مفاتيح الترجمة تحت `orders.photo`.

2. **الاعتماديات**:
   - تتطلب الإصدار 2.2.12 من `Nano.Orders` (أو أحدث).

3. **اختبار التوافق**:
   - اختبار رفع الصور باستخدام `multipart/form-data` و `base64`.
   - اختبار رفع عدة صور وتجاوز الحد الأقصى.
   - اختبار حذف الصور والتحقق من الصلاحية.
   - اختبار منع تغيير الحالة بدون الصور المطلوبة عبر `update-status`.

---

## الخاتمة

يمثل الإصداران **Nano.Orders 2.2.12** و **Nano.OrdersApi 1.0.22** إضافة قوية لإدارة صور الطلبات في نظام نانوسوفت. بفضل التكامل مع `Nano.FileUpload`، أصبح الرفع آمناً وموحداً، مع دعم للملفات العادية و base64. التحكم بالصلاحيات والأدوار (مالك / موصل / مدير) وحالات الطلب يضمن أن الصور ترفع في الوقت المناسب فقط من قبل الأشخاص المخولين. الخيار `skip_photo_check` يتيح للمسؤولين تجاوز المتطلبات عند الضرورة. كما أن واجهة API الجديدة تتيح للتطبيقات الخارجية الاستفادة من هذه الميزات بسهولة.

الكود موثّق، قابل للتوسع، ومتوافق مع الإصدارات السابقة.

---

**الوثائق المرجعية**:
- [توثيق `StepPhotos` الترايت](./docs/Orders/Traits/Steps/Docs-StepPhotos-Trait-ar.md)
- [توثيق `Nano.FileUpload`](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق واجهة برمجة التطبيقات (API) لإدارة الصور](./docs/OrdersApi/Docs-OrdersApi-Photos-ar.md)

## 2026-06-06 - 2026-06-09

**تحديثات إضافة `Nano2.Qrcodes` – الإصدار 1.0.4**

### ملخص التحديثات

يأتي الإصدار 1.0.4 ليُكمِّل البنية القوية التي أُسست في 1.0.3، ويركز على **تحسين تجربة الاستيراد**، **إضافة أوضاع الاختبار والنطاقات المخصصة**، **تدعيم مولد الباركودات بمكتبات إضافية**، **إصلاح أخطاء حرجة**، و **إثراء نظام الصلاحيات والحدود**. الهدف هو توفير بيئة استيراد أكثر أماناً وسرعة، وإتاحة أدوات محاكاة متقدمة، وتوسيع نطاق أنواع الباركودات المدعومة.

أبرز ما في هذا الإصدار:

- **وضع الاختبار (`is_test_import`)** في الاستيراد لمحاكاة العملية دون حفظ البيانات.
- **الاستيراد بنطاق مخصص (`is_custom_import`)** مع دعم صيغ مرنة (first, last, نطاقات أرقام، عربية/إنجليزية).
- **تحسين أداء الاستيراد** عبر تعطيل مسح الكاش وتوليد الصور أثناء الجري، ثم مسح الكاش مرة واحدة بعد الانتهاء.
- **دعم المكتبات** `chillerlan\QRCode`، `PHP QR Code`، `BaconQrCode`، `Endroid\QrCode` داخل `BarcodeGenerator`.
- **إضافة `validateBarcode`** الشاملة التي تتحقق من صحة قيم الباركود لكل الأنواع (EAN, UPC, ISBN, Code 128/39/93, I25, POSTNET، وغيرها).
- **تطبيق قواعد النوع ديناميكياً** عبر `applyBarcodeTypeRules` لضمان توافق الباركود مع النوع المختار.
- **إصلاح خطأ فادح** في `Manager::createBarcode` كان يعين كائن النموذج في حقل `barcode`.
- **إصلاح كود لا يمكن الوصول إليه** في `getBarcodeImageUrl`.
- **تحسين حدود الاستخدام** بإضافة التتبع عبر قاعدة البيانات (`tracking_type=database`) وفترات مرنة (ساعة، أسبوع، شهر، أيام مخصصة).
- **إضافة `frontend_resolver`** و `advanced_filters` في صلاحيات API لتعزيز المرونة.
- **توسيع قائمة أنواع الباركودات** في `getBarcodeTypeOptions` لتشمل جميع الأنواع المدعومة من `BarcodeGenerator`.

كل ذلك مع ترجمة كاملة للمصطلحات الجديدة وتوثيق عملي.

---

### الإصدار 1.0.4 – النضج في الاستيراد والتحقق والتوسع

#### أهداف الإصدار

- **تمكين المستخدم من اختبار الاستيراد** قبل تنفيذه فعلياً (`is_test_import`).
- **إتاحة تحديد نطاق معين من السجلات** في ملف الاستيراد دون الحاجة لتحرير الملف (`is_custom_import` و `custom_import_range`).
- **تحسين أداء الاستيراد** لتجنب التأخير في الملفات الكبيرة (آلاف الصفوف).
- **توسيع قدرات `BarcodeGenerator`** لاستيعاب أكبر عدد من المكتبات وأنواع المخرجات.
- **إضافة طبقة تحقق صارمة ومتعددة** للباركودات حسب نوعها.
- **إصلاح الأخطاء البرمجية** التي تؤثر على إنشاء الباركودات وعرض الصور.
- **تخصيص حدود الاستخدام** لتناسب متطلبات العمل (يومي، ساعي، أسبوعي، شهري، أو أيام مخصصة).
- **جعل نظام الصلاحيات أكثر ديناميكية** لدعم الفلاتر المتقدمة وحل البيانات من المستخدم (frontend resolver).

#### الميزات الجديدة والتحسينات

##### 1. أوضاع الاستيراد المتقدمة: الاختبار والنطاق المخصص

أضفنا إلى نموذج `BarcodeImport` ثلاثة حقول جديدة في واجهة الاستيراد:

- **`is_test_import`**: وضع الاختبار (Test Mode). عند تفعيله، لا يتم حفظ البيانات في قاعدة البيانات، بل يتم استدعاء دوال `Manager::createBarcode` و `updateBarcode` مع `$is_test_create = true`، ثم يتم عرض نتائج المحاكاة (نجاح/فشل) عبر رسائل تحذيرية. يمكن للمستخدم مراجعة الأخطاء قبل الاستيراد الفعلي.

- **`is_custom_import`**: تفعيل الاستيراد بنطاق مخصص. عند تفعيله، يظهر حقل `custom_import_range`.

- **`custom_import_range`**: حقل نصي يدعم صيغاً متعددة لتحديد السجلات المراد استيرادها:
  - `5` – سجل واحد فقط (رقم 5).
  - `5-10` – من السجل 5 إلى 10.
  - `1,3,5,7` – سجلات محددة مفصولة بفواصل.
  - `1-5,10,15-20` – مزيج من نطاقات وأرقام.
  - `أول 10` / `first 10` – أول 10 سجلات.
  - `آخر 5` / `last 5` – آخر 5 سجلات.
  - `من 5 إلى 10` / `السجلات من 5 إلى 10` – نطاق بالعربية.

تمت إضافة دالة `parseRange` و `filterResultsByRange` في `CustomImportRange` trait لتحليل هذه الصيغ وتطبيقها على مصفوفة النتائج.

**تأثير الأداء**: أثناء الاستيراد، يتم تعطيل مسح الكاش (`skip_cache_clear = true`) وتوليد الصور (`generate_image = false`) لتسريع العملية، ويُمسح الكاش مرة واحدة بعد انتهاء الحلقة.

##### 2. تحسين `Manager::createBarcode` و `updateBarcode` لدعم الاختبار وتخطي الكاش

- أُضيف خيار `skip_cache_clear` (افتراضي `false`) يمكن تمريره من الاستيراد لمنع استدعاء `Barcode::clearCache` أثناء الحفظ.
- أُضيف خيار `skip_barcode_type_rules` (يقبل `null`، `true`، `false`) لتجاوز قواعد النوع عند الحاجة (مثل استيراد بيانات قديمة).
- تم إصلاح **الخطأ الفادح**: كان السطر `$barcode->barcode = $barcode;` يعين كائن النموذج في حقل النص. أصبح الآن `$barcodeModel->barcode = $barcode;` (مع تغيير اسم المتغير لتجنب التعارض).
- تم إصلاح **الكود الذي لا يمكن الوصول إليه** في `getBarcodeImageUrl` (كان هناك `return null;` قبل إنشاء الرابط).

##### 3. توسيع `BarcodeGenerator` بمكتبات إضافية

أضفنا محولات للمكتبات التالية مع كشف تلقائي:

- **chillerlan\QRCode** (مكتبة QR متقدمة، تدعم إعدادات دقيقة).
- **PHP QR Code** (دالة `qrcode` – تستخدم في بيئات لا تتوفر فيها مكتبات أخرى).
- **BaconQrCode** (مكتبة QR خفيفة).
- **Endroid\QrCode** (مكتبة QR حديثة).

أصبحت قائمة المكتبات المدعومة:

| المكتبة | الأولوية | الأنواع المدعومة |
|---------|----------|------------------|
| Picqer\Barcode | 1 | 1D & 2D |
| chillerlan\QRCode | 2 | QR فقط |
| SimpleSoftwareIO\QrCode | 3 | QR فقط |
| PHP QR Code (qrcode) | 4 | QR فقط |
| BaconQrCode | 5 | QR فقط |
| Endroid\QrCode | 6 | QR فقط |
| Milon\Barcode | 7 | 1D & 2D |
| Fallback (GD) | الأخير | نص بسيط |

كما تم إضافة دالة `generateBarcodeImage` التي تُعد واجهة مبسطة لإنشاء صور PNG.

##### 4. التحقق من صحة الباركود `validateBarcode`

تمت كتابة دالة شاملة `validateBarcode($barcode, $type)` تدعم:

- **2D**: أي نوع (QR, PDF417, Datamatrix, Aztec, MaxiCode) – فقط تتأكد من أن الطول ≤ 4096 حرفاً.
- **EAN-13, EAN-8, UPC-A, UPC-E**: تتحقق من الطول وخوارزمية الرقم الاختباري (mod 10).
- **ISBN-10, ISBN-13, ISSN**: تتحقق من خوارزميات الأرقام الاختبارية الخاصة بها.
- **Code 128, Code 39, Code 93, Codabar, RMS4CC, KIX, IMB**: تتحقق من الأحرف المسموحة (ASCII القابلة للطباعة أو مجموعة محددة).
- **Interleaved 2 of 5 (I25)**: تتحقق من الأرقام والطول الزوجي.
- **POSTNET, PLANET**: تتحقق من الأطوال المسموحة (5,9,11 و 12,14 على التوالي).
- **MSI, CODE11, PHARMA, PHARMA2T**: أرقام فقط.

هذه الدالة مفيدة جداً قبل إنشاء أو استيراد الباركودات للتأكد من صحة البيانات.

##### 5. تطبيق قواعد النوع ديناميكياً `applyBarcodeTypeRules`

أصبحت دالة `beforeValidate` في موديل `Barcode` تستدعي `applyBarcodeTypeRules` التي تعدل قواعد التحقق (`$rules['barcode']`) استناداً إلى `barcode_type`. تشمل القواعد:

- الأطوال الثابتة (EAN13: 13 رقم، EAN8: 8، UPCA: 12، UPCE: 8).
- الأنواع الرقمية بطول متغير (I25, S25, MSI, CODE11, POSTNET, PLANET, PHARMA, PHARMA2T) مع دعم أطوال زوجية أو أطوال مسموحة.
- الأنواع الأبجدية الرقمية (C128, C39, C93, CODABAR, RMS4CC, KIX, IMB) مع تحديد مجموعة أحرف مسموحة.
- الأنواع 2D (QR, PDF417, DATAMATRIX, AZTEC, MAXICODE) مع حد أقصى 2000 حرف.
- قاعدة عامة لبقية الأنواع.

هذا يضمن أن الباركود المخزّن يتوافق مع المعايير المعتمدة لنوعه.

##### 6. تحسين حدود الاستخدام (Usage Limits)

تم إضافة إمكانيات جديدة في `config.php`:

```php
'usage_limits' => [
    'enabled' => true,
    'daily_limit' => 10,
    'hourly_limit' => 3,
    'tracking_type' => 'database',   // أو 'cache'
    'tracking_period' => 'day',      // 'hour', 'day', 'week', 'month', أو عدد أيام
    'tracking_custom_days' => null,
    'use_soft_limit' => false,
    'limit_message' => null,
],
```

- **`tracking_type = database`**: يقوم بحساب عدد الاستخدامات من جدول الباركودات مباشرة (حقل `used_at`)، مما يضمن دقة مطلقة ولا يعتمد على ذاكرة التخزين المؤقت.
- **`tracking_period`**: يمكن ضبطها على `hour`، `day`، `week`، `month`، أو عدد صحيح (مثل 10 أيام). يتم حساب بداية الفترة باستخدام `Carbon` وفلترة السجلات التي استخدمت بعد ذلك الوقت.
- **`use_soft_limit`**: إذا كان `true`، يتم السماح بتجاوز الحد مع إصدار تحذير بدلاً من رفض العملية (سيُطبق لاحقاً).
- **`limit_message`**: تخصيص رسالة الخطأ عند تجاوز الحد.

تم إضافة دوال مساعدة: `getPeriodStartTime`, `getUserUsageCountFromDatabase`, `getUserCurrentUsageCount`, `getUsageLimitValue` (مع مراعاة الأولويات).

##### 7. تعزيز صلاحيات API (Frontend Resolver & Advanced Filters)

- **`frontend_resolver`**: يسمح بتحديد كيفية تعبئة حقول مثل `user_id` و `user_type` تلقائياً للمستخدمين المسجلين. يمكن إعداد قواعد لكل `ref_type` (مثل `user`, `student`, `parent`). يتم استدعاء `resolveDynamicFrontendOptions` في المتحكم API قبل تنفيذ الاستعلام.

- **`advanced_filters`**: يتحكم في إمكانية استخدام الفلاتر المتقدمة (`is_or_*`, `is_not_*`, `is_or_null_*`) لكل عملية. يحدد الحقول المسموحة لكل نوع مستخدم (backend/frontend/guest) وقواعد خاصة بالحقول. يتم استدعاء `filterAdvancedOptions` لتطهير الخيارات من الفلاتر غير المسموحة.

##### 8. توسيع قائمة أنواع الباركودات في `getBarcodeTypeOptions`

أصبحت الدالة تعرض جميع الأنواع المعرفة في `BarcodeGenerator::TYPE_1D` و `TYPE_2D` مع أسماء مقروءة:

```php
$types1D = [
    'C128'      => 'Code 128',
    'C39'       => 'Code 39',
    'C93'       => 'Code 93',
    'EAN13'     => 'EAN-13',
    'EAN8'      => 'EAN-8',
    'UPCA'      => 'UPC-A',
    'UPCE'      => 'UPC-E',
    'I25'       => 'Interleaved 2 of 5',
    'S25'       => 'Standard 2 of 5',
    'CODABAR'   => 'Codabar',
    'MSI'       => 'MSI',
    'POSTNET'   => 'POSTNET',
    'PLANET'    => 'PLANET',
    'RMS4CC'    => 'RMS4CC (Royal Mail)',
    'KIX'       => 'KIX',
    'IMB'       => 'Intelligent Mail',
    'CODE11'    => 'Code 11',
    'PHARMA'    => 'Pharmacode',
    'PHARMA2T'  => 'Pharmacode 2-Track',
    'QR'        => 'QR Code',
    'PDF417'    => 'PDF417',
    'DATAMATRIX' => 'Data Matrix',
    'AZTEC'     => 'Aztec',
    'MAXICODE'  => 'MaxiCode',
];
```

##### 9. إصلاحات الأخطاء (Bug Fixes)

- **إصلاح خطأ `TypeError` في `Manager::createBarcode`**: كان تعيين `$barcode->barcode = $barcode;` يعين كائن النموذج بدلاً من القيمة النصية. تم تغيير اسم المتغير إلى `$barcodeModel` لتجنب التعارض.
- **إصلاح كود لا يمكن الوصول إليه في `getBarcodeImageUrl`**: كان هناك `return null;` قبل محاولة إنشاء الرابط، مما يمنع الوصول إلى `route()`. تم إعادة ترتيب المنطق.
- **إصلاح خطأ نحوي في `BarcodeImport::importData`**: تم تصحيح السطر `if($this->skip_barcode_type_rules, ==='null')` (فاصلة زائدة) إلى `if($this->skip_barcode_type_rules === 'null')`.
- **إصلاح مشكلة توليد الصورة في API**: تمت إضافة دالة `generateBarcodeImage` في `BarcodeGenerator` لتستخدمها نقطة `image` في المتحكم.

##### 10. تحسينات إضافية في `Barcode` model

- إضافة دالة `prepareDepartmentAndCompanys` لتوحيد حساب `departments_id` و `companys_id`.
- إضافة دالة `skipCacheClear` مع العلم `skipCacheClear`، وتعديل `afterSave` لاحترامه.
- إضافة دالة `prepareCode` لضمان وجود `code` قبل الحفظ.
- تعديل `getPublicDefault` لتطبيق القيم الافتراضية بشكل أفضل.

---

### أمثلة عملية

#### 1. اختبار الاستيراد قبل التنفيذ (وضع الاختبار)

في واجهة استيراد الباركودات، قم بتفعيل `is_test_import` ثم ارفع ملف CSV. سترى رسائل تحذيرية لكل صف توضح البيانات التي ستُستورد وحالة العملية (نجاح/فشل) دون تغيير قاعدة البيانات.

#### 2. استيراد أول 50 سجل فقط

- فعّل `is_custom_import`.
- اكتب في `custom_import_range`: `أول 50` (أو `first 50`).
- سيتم استيراد أول 50 صفاً فقط من الملف.

#### 3. استخدام `Manager::createBarcode` مع تخطي قواعد النوع والكاش

```php
$result = Manager::createBarcode([
    'barcode' => '123',
    'barcode_type' => 'EAN13',  // غير صالح (يجب 13 رقمًا)
    'skip_barcode_type_rules' => true,
    'skip_cache_clear' => true,
], false); // false = ليس اختبارًا
```

#### 4. التحقق من صحة باركود قبل الاستيراد

```php
if (BarcodeGenerator::validateBarcode($barcodeValue, $barcodeType)) {
    // صالح
} else {
    // غير صالح
}
```

---

### متطلبات الترقية

1. **تحديث قاعدة البيانات**  
   لا توجد تغييرات هيكلية في الجداول، لذا لا حاجة لتشغيل ترحيلات جديدة.

2. **تحديث الكود**  
   يجب استبدال الملفات التالية بالنسخ الجديدة:
   - `models/Barcode.php`
   - `models/BarcodeImport.php`
   - `models/barcodeimport/fields.yaml`
   - `models/barcodeimport/CustomImportRange.php` (إنشاء جديد)
   - `classes/Manager.php`
   - `classes/BarcodeGenerator.php`
   - `config/config.php`
   - `lang/ar/lang.php` و `lang/en/lang.php`
   - `apicontrollers/Barcodes.php` (لإضافة التعديلات إن وجدت)
   - `updates/version.yaml`

3. **تسجيل السمة `CustomImportRange`**  
   تأكد من أن `BarcodeImport` يستخدم `use \Nano2\Qrcodes\Models\BarcodeImport\CustomImportRange;` (موجود بالفعل).

4. **إعداد متغيرات البيئة الجديدة** (اختياري)  
   أضف إلى `.env`:
   ```ini
   NANO2_QRCODES_BARCODES_USAGE_LIMITS_TRACKING_TYPE=database
   NANO2_QRCODES_BARCODES_USAGE_LIMITS_TRACKING_PERIOD=week
   ```

5. **اختبار الاستيراد**  
   - جرب وضع الاختبار (`is_test_import`) مع ملف صغير للتأكد من ظهور النتائج.
   - جرب وضع النطاق المخصص (`is_custom_import`) مع صيغ مختلفة.
   - تأكد من أن الاستيراد الفعلي يعمل بكفاءة مع ملف كبير (2000+ سجل).

6. **مسح الكاش**  
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### الخاتمة

الإصدار 1.0.4 يضيف طبقة متقدمة من التحكم والمرونة في استيراد الباركودات، ويحسن الأداء بشكل ملحوظ، ويدعم مزيداً من المكتبات وأنواع الباركودات، ويصلح أخطاء حرجة. أصبحت الإضافة الآن أداة قوية وجاهزة للاستخدام في بيئات الإنتاج التي تتطلب استيراد كميات كبيرة من الباركودات مع إمكانية الاختبار المسبق وتحديد النطاقات. كما أن نظام حدود الاستخدام أصبح أكثر دقة بفضل خيار `database`، ونظام الصلاحيات أصبح أكثر ديناميكية بفضل `frontend_resolver` و `advanced_filters`.

نشكركم على دعمكم المستمر، ونرحب بملاحظاتكم لمواصلة تحسين الإضافة.

---

**الوثائق المرجعية** 

- [التوثيق العام للإضافة](./docs/Qrcodes/Docs-Nano2-Qrcodes-ar.md)
- [توثيق الاستيراد والتصدير](./docs/Qrcodes/Docs-Import-Export-ar.md)
- [توثيق مولد الباركودات](./docs/Qrcodes/Docs-BarcodeGenerator-ar.md)
- [توثيق واجهة برمجة التطبيقات](./docs/Qrcodes/Docs-API-Documentation-ar.md)

## 2026-06-08 - 2026-06-10 

**إطلاق إضافة `Nano.SchoolControlApi` – الإصدار 1.0.1**

### إنشاء واجهة برمجة تطبيقات أساسية للتحكم المدرسي (الأعمال الشهرية، العلامات الفصلية، النتائج)

---

### ملخص التحديثات

يقدم الإصدار الأول **1.0.1** من إضافة `Nano.SchoolControlApi` واجهة برمجة تطبيقات أساسية للتفاعل مع ثلاثة كيانات رئيسية في نظام `Tss.SchoolControl`:

- **الأعمال الشهرية للطلاب** (`StudentMonthlyWork`) – إدارة درجات الأعمال الشهرية لكل مادة.
- **العلامات الفصلية والنهائية** (`StudentMark`) – إدارة درجات الفصلين الأول والثاني.
- **نتائج الطلاب** (`Result`) – إدارة النتائج بجميع أنواعها (شهرية، نصفية، نهائية).

تم بناء الإضافة وفقاً لأحدث معايير `Nano-Api-SKILL.md`، مع الاستفادة من `AccessManager` للصلاحيات، و`AdvancedQueryHelper` للتصفية الديناميكية، ومحولات البيانات (Transformers)، ونظام تخزين مؤقت وأحداث موحد.

---

### أهداف الإصدار

- **توفير API أساسي** لإدارة الأعمال الشهرية والعلامات الفصلية والنتائج.
- **تمكين التطبيقات الخارجية** من قراءة وكتابة بيانات الدرجات والنتائج.
- **دعم الفلاتر الأساسية** مثل `student_id`, `record_id`, `year_id`, `class_id`, `group_id`, `subject_id`.
- **تطبيق نظام صلاحيات أولي** للتحكم في الوصول حسب نوع المستخدم (backend/frontend/guest).
- **توفير توثيق أولي** للنقاط النهائية الأساسية.

---

### الميزات الجديدة

#### 1. ثلاثة متحكمات RESTful أساسية

| المتحكم | النموذج المستهدف | العمليات المدعومة |
|---------|-----------------|-------------------|
| `StudentMonthlyWorks` | `StudentMonthlyWork` | قائمة، عرض، إنشاء، تحديث، حذف |
| `StudentMarks` | `StudentMark` | قائمة، عرض، إنشاء، تحديث، حذف |
| `Results` | `Result` | قائمة، عرض، إنشاء، تحديث، حذف |

**كل متحكم يدعم:**
- `index()`: جلب قائمة مع تصفية وبحث وترتيب وتقسيم صفحات.
- `show($id)`: عرض تفاصيل عنصر مع تضمين العلاقات.
- `getRecords(array $options)`: دالة أساسية لبناء استعلامات معقدة.
- `store()` و `update()` و `destroy()`: عمليات الكتابة.
- `activelystats()`: نقطة نهاية لإرجاع آخر تحديث للتخزين المؤقت.

#### 2. دالة `getRecords` موحدة

تم تطبيق دالة `getRecords` موحدة في كل متحكم باستخدام التريت `HasRecordsHelper`، وتدعم:
- **فلاتر متقدمة** عبر `AdvancedQueryManager::scopeWhereField` مع `is_or`, `is_not`, `is_force`, `is_or_null`.
- **نطاق الوصول** (`applyAccessScope`) للشركة والقسم والولاية ومنشئ السجل.
- **استبعاد الأعمدة** (`exclude`) لتقليل حجم البيانات.
- **تضمين العلاقات** (`custom_with`).
- **البحث النصي** (`q`) مع دعم البحث المتقدم (ArPhpHelper).
- **الترتيب** (`orderBy`, `orderDirection`) والمجموعات (`group_by`) و`having`.
- **خيارات الإخراج**: `is_query`, `is_first`, `is_model`, `is_collection`, `is_paginator`, `is_to_sql`.
- **الأحداث**: `api.list.extendQueryBefore` و `api.list.extendQuery` لتوسيع الاستعلامات.

#### 3. نظام صلاحيات مركزي أولي

تم تعريف إعدادات الصلاحيات الأساسية في ملف `config.php` لكل عملية ولكل مورد، مع إمكانية التحكم عبر متغيرات البيئة.

**مثال من إعدادات `results.list`:**
```php
'results' => [
    'list' => [
        'permission' => [
            'is_allow' => env('NANO_SCHOOLCONTROLAPI_RESULTS_LIST_IS_ALLOW', true),
            'backend' => [
                'allow' => true,
                'check_permission' => true,
                'permissions' => ['tss.schoolcontrol.resultalls.access_all', 'tss.schoolcontrol.resultalls.access'],
                'access_scope' => 'all',
            ],
            'frontend' => [
                'allow' => true,
                'allowed_ref_types' => ['student', 'parent'],
                'access_scope' => 'own',
            ],
        ],
    ],
],
```

يتم التحقق من الصلاحيات عبر `AccessManager::checkByResource()`.

#### 4. محولات البيانات (Transformers)

تم إنشاء ثلاثة محولات أساسية:
- **`StudentMonthlyWorkTransformer`**: يعرض بيانات الأعمال الشهرية مع العلاقات (student, subject, teacher, class, group, studyYear, department).
- **`StudentMarkTransformer`**: يعرض بيانات العلامات الفصلية مع العلاقات.
- **`ResultTransformer`**: يعرض بيانات النتائج مع العلاقات ودعم تضمين الدرجات (`marks`) حسب نوع النتيجة.

كل المحولات تدعم تنسيق التواريخ واستبعاد الحقول عبر `exclude`.

#### 5. استجابة موحدة لعمليات الإنشاء والتحديث

```json
{
  "data": {
    "code": 200,
    "status": true,
    "message": "Result created successfully.",
    "data": { ... },
    "model": { ... },
    "input_data": { ... },
    "process_data": { ... }
  }
}
```

#### 6. تسجيل نطاق `scopeExclude`

تم تسجيل النطاق الديناميكي `scopeExclude` للنماذج الثلاثة في `Plugin::boot()`:
```php
\Tss\SchoolControl\Models\StudentMonthlyWork::extend(function($model) {
    $model->addDynamicMethod('scopeExclude', function($query, $columns) use($model) { ... });
});
// وكذلك لـ StudentMark و Result
```

---

### نقاط النهاية المدعومة

| المورد | الطرق المدعومة | المسارات |
|--------|---------------|----------|
| **StudentMonthlyWorks** | GET, POST, PUT, DELETE | `/student-monthly-works`, `/student-monthly-works/{id}`, `/student-monthly-works/activelystats` |
| **StudentMarks** | GET, POST, PUT, DELETE | `/student-marks`, `/student-marks/{id}`, `/student-marks/activelystats` |
| **Results** | GET, POST, PUT, DELETE | `/results`, `/results/{id}`, `/results/activelystats` |

**قاعدة المسار الأساسية:** `/api/v1/schoolcontrol`

---

### أمثلة على الاستخدام

#### 1. جلب نتائج شهرية لطالب معين

```bash
curl -X GET "https://yourdomain.com/api/v1/schoolcontrol/results?student_id=8&record_id=3&ref_type=monthly&semster=semster1&month_num=1&include=student" \
  -H "Authorization: Bearer <token>"
```

#### 2. إنشاء علامة فصلية جديدة

```bash
curl -X POST "https://yourdomain.com/api/v1/schoolcontrol/student-marks" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": 8,
    "record_id": 3,
    "subject_id": 1,
    "year_id": 3,
    "class_id": 1,
    "s1_month1_mark": 12,
    "s1_month2_mark": 14,
    "s1_month3_mark": 13,
    "exam1_mark": 18
  }'
```

#### 3. تحديث درجة عمل شهري

```bash
curl -X PUT "https://yourdomain.com/api/v1/schoolcontrol/student-monthly-works/2" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"exam_mark": 16}'
```

---

### التغييرات التقنية

#### ملفات جديدة

| المسار | الوصف |
|--------|-------|
| `apicontrollers/StudentMonthlyWorks.php` | متحكم الأعمال الشهرية. |
| `apicontrollers/StudentMarks.php` | متحكم العلامات الفصلية. |
| `apicontrollers/Results.php` | متحكم النتائج. |
| `traits/HasRecordsHelper.php` | التريت الموحد لبناء الاستعلامات. |
| `helpers/StudentMonthlyWorkRecordsHelper.php` | مساعد الأعمال الشهرية. |
| `helpers/StudentMarkRecordsHelper.php` | مساعد العلامات الفصلية. |
| `helpers/ResultRecordsHelper.php` | مساعد النتائج. |
| `transformers/StudentMonthlyWorkTransformer.php` | محول الأعمال الشهرية. |
| `transformers/StudentMarkTransformer.php` | محول العلامات الفصلية. |
| `transformers/ResultTransformer.php` | محول النتائج. |
| `config/config.php` | إعدادات الصلاحيات والفلاتر. |
| `lang/ar/lang.php` | ترجمة عربية. |
| `lang/en/lang.php` | ترجمة إنجليزية. |
| `routes.php` | تعريف المسارات. |
| `Plugin.php` | تسجيل الإضافة و `scopeExclude`. |
| `version.yaml` | تعريف الإصدار. |

---

### متطلبات التثبيت

1. **تثبيت الإضافة**: انسخ مجلد `nano/schoolcontrolapi` إلى `plugins/nano/schoolcontrolapi`.
2. **تسجيل الإضافة**:
   ```bash
   php artisan plugin:refresh Nano.SchoolControlApi
   ```
3. **إعدادات البيئة**: أضف متغيرات البيئة المطلوبة في ملف `.env` (راجع `config/config.php`).
4. **الصلاحيات**: تأكد من أن الأدوار المناسبة تمتلك الصلاحيات مثل `tss.schoolcontrol.resultalls.access` إلخ.
5. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### الخاتمة

يمثل الإصدار **1.0.1** من `Nano.SchoolControlApi` الأساس المتين لإدارة الأعمال الشهرية والعلامات الفصلية والنتائج عبر API. يوفر هذا الإصدار واجهة آمنة ومرنة، مع دعم الفلاتر الأساسية ونظام صلاحيات أولي، مما يسمح للمطورين ببناء تطبيقات تعليمية متكاملة.

---

**الوثائق المرجعية**:
- [دليل تطوير إضافات API (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)
- [توثيق كلاس `AccessManager`](./docs/AuthApi/Docs-AccessManager-ar.md)
- [توثيق كلاس `AdvancedQueryHelper`](./docs/querybuilder/Docs-AdvancedQueryHelper.md)

---

## 2026-06-12

**تحديث إضافة `Nano.SchoolControlApi` – الإصدار 1.0.2**

### إضافة أرقام الجلوس، الفترات الامتحانية، واللجان الامتحانية

---

### ملخص التحديثات

يضيف الإصدار **1.0.2** من إضافة `Nano.SchoolControlApi` دعم ثلاثة كيانات إضافية في نظام `Tss.SchoolControl`:

- **أرقام الجلوس** (`Seating`) – إدارة أرقام الجلوس الامتحانية.
- **الفترات الامتحانية** (`ExamPeriod`) – إدارة فترات الامتحانات.
- **اللجان الامتحانية** (`Committee`) – إدارة اللجان والقاعات الامتحانية.

تم اتباع نفس المعايير والنمط المعماري المعتمد في الإصدار 1.0.1، مع إضافة تحسينات في دالة `getDefaultOptions` لدعم الحقول الخاصة بهذه الكيانات.

---

### أهداف الإصدار

- **توسيع نطاق API** ليشمل إدارة أرقام الجلوس والفترات واللجان الامتحانية.
- **تمكين التطبيقات من تخصيص أرقام الجلوس** لكل طالب حسب الفترة واللجنة.
- **دعم الفلاتر المتقدمة** مثل `sitting_number`, `pin_number`, `exam_periods_id`, `committees_id`.
- **توفير نفس مستوى الصلاحيات والأمان** للموارد الجديدة.

---

### الميزات الجديدة

#### 1. ثلاثة متحكمات إضافية

| المتحكم | النموذج المستهدف | العمليات المدعومة |
|---------|-----------------|-------------------|
| `Seatings` | `Seating` | قائمة، عرض، إنشاء، تحديث، حذف |
| `ExamPeriods` | `ExamPeriod` | قائمة، عرض، إنشاء، تحديث، حذف |
| `Committees` | `Committee` | قائمة، عرض، إنشاء، تحديث، حذف |

#### 2. دالة `getDefaultOptions` محدّثة

تم إضافة مفاتيح جديدة في `HasRecordsHelper::getDefaultOptions()` لدعم الحقول الخاصة بالكيانات الجديدة:
- `sitting_number`, `pin_number`, `exam_periods_id`, `committees_id` (لـ `Seating`).
- `name`, `start_at`, `end_at` (لـ `ExamPeriod` و `Committee`).

#### 3. فلاتر متقدمة خاصة بأرقام الجلوس

يمكن تصفية أرقام الجلوس حسب:
- `sitting_number`: رقم الجلوس (يدعم مقارنات).
- `pin_number`: الرقم السري.
- `exam_periods_id`: الفترة الامتحانية.
- `committees_id`: اللجنة الامتحانية.

#### 4. دعم العلاقات في Transformers الجديدة

- **`SeatingTransformer`**: يدعم تضمين `student`, `record`, `class`, `group`, `studyYear`, `examPeriod`, `committee`, `department`.
- **`ExamPeriodTransformer`**: يدعم تضمين `department`, `studyYear`.
- **`CommitteeTransformer`**: يدعم تضمين `department`, `studyYear`.

#### 5. إعدادات صلاحيات مخصصة

تم إضافة إعدادات `permission` لكل من `seatings`, `exam_periods`, `committees` في `config.php`، مع دعم متغيرات البيئة.

**مثال:**
```php
'seatings' => [
    'list' => [
        'permission' => [
            'is_allow' => env('NANO_SCHOOLCONTROLAPI_SEATINGS_LIST_IS_ALLOW', true),
            'backend' => [
                'allow' => true,
                'permissions' => ['tss.schoolcontrol.seatings.access_all', 'tss.schoolcontrol.seatings.access'],
            ],
            'frontend' => [
                'allow' => true,
                'allowed_ref_types' => ['student', 'parent'],
            ],
        ],
    ],
],
```

#### 6. تسجيل `scopeExclude` للنماذج الجديدة

تم إضافة النماذج `Seating`, `ExamPeriod`, `Committee` إلى حلقة `extendModelsScopeExclude` في `Plugin.php`.

---

### نقاط النهاية الجديدة

| المورد | الطرق المدعومة | المسارات |
|--------|---------------|----------|
| **Seatings** | GET, POST, PUT, DELETE | `/seatings`, `/seatings/{id}`, `/seatings/activelystats` |
| **ExamPeriods** | GET, POST, PUT, DELETE | `/exam-periods`, `/exam-periods/{id}`, `/exam-periods/activelystats` |
| **Committees** | GET, POST, PUT, DELETE | `/committees`, `/committees/{id}`, `/committees/activelystats` |

---

### أمثلة على الاستخدام

#### 1. جلب أرقام جلوس طالب معين

```bash
curl -X GET "https://yourdomain.com/api/v1/schoolcontrol/seatings?student_id=8&record_id=3" \
  -H "Authorization: Bearer <token>"
```

#### 2. إنشاء لجنة امتحانية جديدة

```bash
curl -X POST "https://yourdomain.com/api/v1/schoolcontrol/committees" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "اللجنة الأولى",
    "departments_id": "4",
    "year_id": "3",
    "semster": "semster2",
    "ref_type": "finality"
  }'
```

#### 3. تحديث فترة امتحانية

```bash
curl -X PUT "https://yourdomain.com/api/v1/schoolcontrol/exam-periods/5" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"start_at": "2026-06-20 08:00:00"}'
```

---

### التغييرات التقنية

#### ملفات جديدة

| المسار | الوصف |
|--------|-------|
| `apicontrollers/Seatings.php` | متحكم أرقام الجلوس. |
| `apicontrollers/ExamPeriods.php` | متحكم الفترات الامتحانية. |
| `apicontrollers/Committees.php` | متحكم اللجان الامتحانية. |
| `helpers/SeatingRecordsHelper.php` | مساعد أرقام الجلوس. |
| `helpers/ExamPeriodRecordsHelper.php` | مساعد الفترات الامتحانية. |
| `helpers/CommitteeRecordsHelper.php` | مساعد اللجان الامتحانية. |
| `transformers/SeatingTransformer.php` | محول أرقام الجلوس. |
| `transformers/ExamPeriodTransformer.php` | محول الفترات الامتحانية. |
| `transformers/CommitteeTransformer.php` | محول اللجان الامتحانية. |

#### ملفات محدّثة

- `traits/HasRecordsHelper.php`: إضافة مفاتيح جديدة في `getDefaultOptions()`.
- `config/config.php`: إضافة إعدادات `seatings`, `exam_periods`, `committees`.
- `Plugin.php`: إضافة النماذج الجديدة إلى `extendModelsScopeExclude`.
- `routes.php`: إضافة المسارات الجديدة.

---

### متطلبات الترقية (من 1.0.1 إلى 1.0.2)

1. **تحديث الكود**: استبدال جميع الملفات المذكورة أعلاه.
2. **تحديث `version.yaml`**:
   ```yaml
   1.0.2:
     - 'Added API endpoints for seatings (أرقام الجلوس), exam periods (الفترات الامتحانية), and committees (اللجان الامتحانية).'
   ```
3. **تنفيذ هجرات**: لا توجد هجرات جديدة (تعتمد على جداول `Tss.SchoolControl` الموجودة).
4. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### الخاتمة

يكمل الإصدار **1.0.2** من `Nano.SchoolControlApi` تغطية كافة الكيانات الأساسية في نظام التحكم المدرسي، مما يوفر منصة متكاملة لإدارة النتائج والدرجات وأرقام الجلوس والفترات واللجان الامتحانية عبر API واحد. هذا الإصدار يمهد الطريق للإصدار 1.0.3 الذي سيركز على تحسينات دعم ولي الأمر والحقول المحسوبة والتوثيق.

---

**الوثائق المرجعية**:
- [دليل تطوير إضافات API (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)

## 2026-06-10 - 2026-06-13

**تحديثات إضافة `Nano.TranslateExtended` – الإصدار 1.0.12**  
**إضافة نظام متكامل لتجاوز الترجمات (Translation Override) مع دعم متعدد المسارات والكاش والإعدادات عبر الواجهة الخلفية ومتغيرات البيئة**

### ملخص التحديثات

يُضيف الإصدار **1.0.12** نظاماً متكاملاً ومرناً لتجاوز أي ترجمة في أي إضافة أو في النظام الأساسي، دون الحاجة إلى تعديل ملفات الترجمة الأصلية. يعتمد النظام على آلية `TranslationOverrideManager` التي تقرأ الترجمات من مجلدات مخصصة، وتخزنها في الكاش، وتدمجها في نظام الترجمة عبر الدالة `set()` من `October\Rain\Translation` لضمان التجاوز الفعال. يتم التحكم في النظام من خلال واجهة إدارة جديدة ضمن إعدادات الإضافة، أو عبر متغيرات البيئة، أو ملف الإعدادات `config.php`.

---

## Nano.TranslateExtended v1.0.12 – نظام تجاوز الترجمات (Translation Override)

### أهداف الإصدار

- **توفير آلية احترافية لتجاوز الترجمات** دون تعديل الملفات الأصلية للإضافات أو النظام.
- **دعم مسارات متعددة لملفات التجاوز** مع إمكانية تحديد الأولوية (آخر مسار له الأسبقية).
- **دمج نظام التجاوز مع إعدادات الإضافة** بحيث يمكن تفعيله / تعطيله وتعديل مدته ومساراته عبر واجهة المستخدم.
- **دعم كامل لمتغيرات البيئة** للتحكم بكل إعداد دون الحاجة إلى قاعدة البيانات.
- **تخزين مؤقت ذكي** لتجنب إعادة قراءة الملفات في كل طلب، مع إمكانية تعطيل الكاش للتطوير.
- **معالجة استثنائية شاملة** وتسجيل الأخطاء في `system.log` لتسهيل التصحيح.
- **توافق كامل مع `October\Rain\Translation::set()`** لضمان استبدال الترجمات بشكل صحيح.

### الميزات الجديدة والتحسينات

---

#### أولاً: `TranslationOverrideManager` – مدير التجاوزات

##### 1. قراءة الملفات من مسارات متعددة

- البحث في المسارات المحددة (مثل `lang/override` و `plugins/nano/translateextended/override/lang`).
- دعم المسارات المطلقة والنسبية (تُحول تلقائياً إلى `base_path()`).
- دمج جميع ملفات PHP الموجودة في المجلد (أي اسم ملف) وفي المجلدات الفرعية.
- تحويل المصفوفات المتداخلة إلى صيغة `dot.notation` (مثلاً `['a' => ['b' => 'c']]` تصبح `['a.b' => 'c']`).

##### 2. دمج الترجمات باستخدام `set()`

- استخدام الدالة `$translator->set($key, $value, $locale)` التي توفرها OctoberCMS لضمان تجاوز الترجمة حتى لو كانت قد حملت مسبقاً.
- دعم المفاتيح العادية (`messages.welcome`) والمفاتيح ذات مساحة الاسم (`nano.plugin::lang.key`).
- تسجيل الأخطاء في حال فشل إضافة أي ترجمة.

##### 3. التخزين المؤقت (Cache)

- تخزين الترجمات المقروءة من الملفات لمدة قابلة للتكوين (افتراضياً 60 دقيقة).
- إمكانية تعطيل الكاش بوضع `override_cache_minutes = 0`.
- مسح الكاش تلقائياً عند حفظ إعدادات الإضافة.
- مسح الكاش يدوياً عبر دالة `clearCache()` أو مع الحدث `locale.changed`.

##### 4. دوال مساعدة عامة

- `loadAll($locale = null)`: تحميل ترجمات كل اللغات المتاحة أو لغة واحدة.
- `loadForLocale($locale)`: تحميل الترجمة للغة معينة ودمجها.
- `reload($locale = null)`: مسح الكاش وإعادة تحميل اللغة.
- `addPath($path)`: إضافة مسار إضافي ديناميكياً.
- `getPaths()` و `isEnabled()` و `getCacheMinutes()` للتشخيص.

##### 5. معالجة استثنائية وتسجيل أخطاء

- كل دالة محاطة بـ `try/catch` مع تسجيل الخطأ في `Log::error` أو `Log::warning`.
- التعامل مع `Throwable` لالتقاط جميع أنواع الأخطاء.
- تسجيل معلومات عند فشل تضمين ملف ترجمة أو عند وجود ملف لا يعيد مصفوفة.

---

#### ثانياً: تحديثات نموذج `Settings`

##### 1. إضافة حقول جديدة في `fields.yaml`

- `enable_overrides` (switch): تفعيل / تعطيل نظام التجاوز.
- `override_cache_minutes` (number): مدة التخزين المؤقت (بالدقائق).
- `override_paths` (textarea): قائمة المسارات (كل مسار في سطر جديد).

##### 2. دوال مساعدة جديدة في `Settings`

- `getWithEnvPriority($key, $envVarName, $default)`: استرجاع القيمة وفق الأولوية (env > database > config).
- `getOverridePaths()`: إرجاع مصفوفة المسارات بعد تحويل النسبي إلى مطلق ودعم سلاسل متعددة الأسطر.
- تحسين معالجة القيم المنطقية من `env` (مثل `false` و `true` و `null`).

##### 3. تحسين التوافق مع PHP 7.4+

- استبدال `str_starts_with()` بـ `strpos() === 0` لضمان العمل على الإصدارات الأقدم.

---

#### ثالثاً: إعدادات `config.php` الجديدة

أضيفت المفاتيح التالية إلى ملف `config.php` داخل الإضافة:

```php
'enable_overrides'          => env('NANO_TRANSLATE_ENABLE_OVERRIDES', true),
'override_paths'            => [
    base_path('lang/override'),
    plugins_path('nano/translateextended/override/lang'),
],
'override_cache_minutes'    => env('NANO_TRANSLATE_CACHE_MINUTES', 60),
'forceDefaultLocale'        => env('NANO_TRANSLATE_FORCE_LOCALE', null),
'prefixDefaultLocale'       => env('NANO_TRANSLATE_PREFIX_LOCALE', true),
'disableLocalePrefixRoutes' => env('NANO_TRANSLATE_DISABLE_PREFIX_ROUTES', false),
```

---

#### رابعاً: متغيرات البيئة (`.env`) المدعومة

| المتغير | الغرض | مثال |
|---------|-------|------|
| `NANO_TRANSLATE_ENABLE_OVERRIDES` | تفعيل / تعطيل النظام | `true` / `false` |
| `NANO_TRANSLATE_CACHE_MINUTES` | مدة الكاش بالدقائق | `120` |
| `NANO_TRANSLATE_OVERRIDE_PATHS` | مسارات التجاوز (سطر واحد، مفصولة بفواصل أو أسطر) | `lang/override,plugins/nano/translateextended/override/lang` |
| `NANO_TRANSLATE_FORCE_LOCALE` | فرض لغة ثابتة على الموقع | `ar` |
| `NANO_TRANSLATE_PREFIX_LOCALE` | إضافة بادئة اللغة للروابط (حتى للغة الافتراضية) | `true` / `false` |
| `NANO_TRANSLATE_DISABLE_PREFIX_ROUTES` | تعطيل توجيه اللغة عبر الرابط بشكل كامل | `true` / `false` |

---

#### خامساً: ربط الأحداث (Event Listeners)

- **`backend.form.beforeSave`**: مسح كاش التجاوزات عند حفظ إعدادات الإضافة.
- **`locale.changed`**: إعادة تحميل ترجمات اللغة الجديدة عند تغيير اللغة.
- (اختياري) **`rainlab.translate.beforeResolveLocale`** و **`LocaleMiddleware::$prefixDefaultLocale`** تم دعمهما في `applyLocaleSettings` لتطبيق `forceDefaultLocale` و `prefixDefaultLocale`.

---

#### سادساً: أمثلة ملفات التجاوز

**ملف `lang/override/ar/custom.php`**

```php
<?php return [
    'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'إدارة متجر آخر',
    'tss.inventory::lang.products.form.use_case_options.new' => 'جديد',
    'tss.inventory::lang.products.form.use_case_options.excellent' => 'ممتاز',
];
```

**ملف `lang/override/en/custom.php`**

```php
<?php return [
    'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'Manage Another Store',
    'tss.inventory::lang.products.form.use_case_options.new' => 'New',
    'tss.inventory::lang.products.form.use_case_options.excellent' => 'Excellent',
];
```

---

### كيفية الاستخدام (للمطورين)

#### 1. تفعيل النظام

- من لوحة التحكم: `الإعدادات → Translate Extended → تفعيل تجاوز الترجمات`.
- أو عبر `.env`: `NANO_TRANSLATE_ENABLE_OVERRIDES=true`.

#### 2. إنشاء ملفات التجاوز

- أنشئ مجلد `lang/override/{locale}/` في جذر المشروع.
- ضع أي ملف `.php` يعيد مصفوفة المفاتيح والقيم المطلوب تجاوزها.

#### 3. مسح الكاش

- احفظ إعدادات الإضافة (يتم تلقائياً).
- أو نفذ أمر `php artisan cache:clear`.
- أو استخدم دالة `$manager->clearCache()` برمجياً.

#### 4. استخدام البرمجة

```php
$manager = app('translateextended.translationOverride');
$manager->addPath(base_path('my/extra/path'));
$manager->reload('ar');
```

---

### الترقية من الإصدارات السابقة

1. تحديث ملفات الإضافة عبر `php artisan plugin:refresh Nano.TranslateExtended` أو نسخ الملفات يدوياً.
2. تشغيل `php artisan cache:clear` لمسح الكاش القديم.
3. (اختياري) إضافة متغيرات البيئة الجديدة إلى `.env`.
4. الدخول إلى صفحة إعدادات الإضافة وضبط القيم المطلوبة.

> **ملاحظة:** لا توجد تغييرات في هيكل قاعدة البيانات، الترقية آمنة ولا تؤثر على البيانات الحالية.

---

### معالجة الأخطاء الشائعة

| المشكلة | الحل |
|---------|------|
| الترجمات لا تتجاوز رغم وجود الملفات | تأكد من تفعيل `enable_overrides` ومن صحة المسارات. امسح الكاش. |
| خطأ `Too few arguments to function ...` | حدث `locale.changed` يحتاج إلى `$oldLocale = null` (تم إصلاحه في هذا الإصدار). |
| تظهر الترجمة القديمة بعد التعديل مباشرة | امسح الكاش أو استخدم `$manager->reload()`. |
| استخدام `str_starts_with` يسبب خطأ في PHP 7.4 | تم استبدالها بـ `strpos() === 0`. |

---

### ملخص الإصدار (1.0.12)

| الإصدار | أبرز الميزات |
|---------|---------------|
| **Nano.TranslateExtended 1.0.12** | نظام متكامل لتجاوز الترجمات، دعم مسارات متعددة، كاش قابل للتكوين، إعدادات عبر واجهة المستخدم ومتغيرات البيئة، توافق كامل مع `October\Rain\Translation::set()`، معالجة استثنائية وتسجيل أخطاء. |

---

### متطلبات الترقية

1. **تحديث الملفات**:
   - `Plugin.php`
   - `classes/TranslationOverrideManager.php`
   - `models/Settings.php`
   - `config/config.php`
   - `fields.yaml`
   - `version.yaml`

2. **لا توجد هجرات قاعدة بيانات جديدة** – فقط إضافة حقول إعدادات اختيارية.

3. **الاعتماديات**:
   - `RainLab.Translate` (أي إصدار حديث)
   - `OctoberCMS` ≥ 2.0 أو `WinterCMS` ≥ 1.2

4. **إعدادات إضافية موصى بها في `.env`**:
   ```
   NANO_TRANSLATE_ENABLE_OVERRIDES=true
   NANO_TRANSLATE_CACHE_MINUTES=60
   NANO_TRANSLATE_OVERRIDE_PATHS="lang/override,plugins/nano/translateextended/override/lang"
   ```

5. **اختبار التوافق**:
   - إنشاء ملف تجاوز بسيط والتحقق من ظهور الترجمة الجديدة.
   - اختبار تغيير اللغة (`locale.changed`) وتحميل الترجمات تلقائياً.
   - اختبار تفعيل / تعطيل النظام وحفظ الإعدادات.

---

### الخاتمة

يمثل الإصدار **1.0.12** نقلة نوعية في إمكانيات الإضافة، حيث يمنح المطورين القدرة على تعديل أي ترجمة في النظام دون لمس الملفات الأصلية. النظام مصمم ليكون مرناً، عالي الأداء، وسهل الاستخدام، مع دعم كامل لأدوات OctoberCMS المعتادة (الإعدادات، الأحداث، الكاش). يمكن استخدامه لتخصيص واجهات المستخدم، تصحيح ترجمات خاطئة، أو حتى إنشاء ترجمات خاصة بالعميل دون التعارض مع تحديثات الإضافات.

---

**الوثائق المرجعية**:
- [توثيق الإضافة `Nano.TranslateExtended`](./docs/TranslateExtended/Docs-Nano.TranslateExtended-ar.md)
- [توثيق سلوك `DynamicAddIncludeTranslatableApiFields`](./docs/TranslateExtended/Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-ar.md)
- [توثيق سلوك `TranslatableContentCaching`](./docs/TranslateExtended/Behaviors/Docs-TranslatableContentCaching-ar.md)
- [دليل استخدام الترجمات المحسّنة في API](./docs/TranslateExtended/Docs-API-Translations-Guide-ar.md)
- [دليل استخدام تجاوز الترجمات](./docs/TranslateExtended/Docs-Translation-Override-Guide-ar.md)
- [توثيق كلاس `TranslationOverrideManager`](./docs/TranslateExtended/Classes/Docs-TranslationOverrideManager-Class-ar.md)
- [ملف الإعدادات `config.php`](./plugins/nano/translateextended/config/config.php)
- [نموذج الإعدادات `Settings`](./plugins/nano/translateextended/models/Settings.php)

