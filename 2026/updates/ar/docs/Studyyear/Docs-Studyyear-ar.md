# توثيق إضافة `Tss.Studyyear`

**الإصدار الحالي:** 1.0.7  
**المطور:** Dheia Ali  
**التبعيات:** RainLab.Translate, RainLab.Location, RainLab.User, Tss.Tools, Tss.BarcodeGenerator, Tss.Basic, Tss.School

---

## مقدمة

إضافة `Tss.Studyyear` هي المكون الأساسي لإدارة **السنوات الدراسية**، **الفصول الدراسية (الأترام)**، و**الأشهر الدراسية** ضمن نظام نانو سوفت المتكامل. توفر الإضافة بنية بيانات مرنة وقابلة للتوسع، مع واجهة مستخدم خلفية متكاملة، ونظام صلاحيات متقدم، ودعم كامل للترجمة (عربي/إنجليزي).

تمثل هذه الإضافة حجر الزاوية لأي نظام تعليمي، حيث تعتمد عليها العديد من الإضافات الأخرى مثل `Tss.School` (الطلاب والصفوف) و `Tss.Absence` (الحضور والغياب) و `Nano.HomeworkApi` (الواجبات) وغيرها.

---

## الميزات الرئيسية

- **إدارة السنوات الدراسية (Periods):**  
  إنشاء سنوات دراسية متعددة مع تواريخ بداية ونهاية، وإمكانية تعيين سنة افتراضية.

- **إدارة الفصول الدراسية (Semsters):**  
  كل سنة دراسية يمكن أن تحتوي على فصلين دراسيين (`semster1`, `semster2`)، مع تواريخ خاصة لكل فصل.

- **إدارة الأشهر الدراسية (Months):**  
  كل فصل دراسي يتكون من ثلاثة أشهر دراسية (`month_num` 1,2,3)، مع إمكانية تحديد أسماء الأشهر (يناير، فبراير…).

- **نظام نطاقات (Scopes) ذكي:**  
  - `IsCompany()`: تصفية حسب الشركة الحالية.  
  - `IsActive()`: عرض السجلات النشطة فقط.  
  - `IsCheckDate()`: التحقق من صلاحية التواريخ (لا تعرض السجلات منتهية الصلاحية) – قابل للتعطيل عبر متغيرات البيئة.  
  - `IsPublished()` / `IsNotPublished()`: تصفية السجلات المنشورة/غير المنشورة بناءً على `is_published` و `published_at` و `unpublished_at`.  
  - `scopeIsPublishedResults`: تصفية حسب `is_published_results`.

- **دعم النشر المؤقت:**  
  باستخدام حقول `is_published`، `published_at`، `unpublished_at` يمكن جدولة ظهور السجلات في الواجهات الأمامية والـ API.

- **حقل جديد `is_published_results`:**  
  يسمح بفصل نشر البيانات الأساسية عن نشر النتائج (مثل ظهور السنة الدراسية في تطبيق النتائج).

- **تخزين مؤقت (Caching) محسّن:**  
  جميع قوائم الخيارات والاستعلامات المتكررة مخزنة لتحسين الأداء.

- **دعم الترجمة:**  
  جميع النصوص قابلة للترجمة، مع ملفات جاهزة للعربية والإنجليزية.

- **صلاحيات مفصلة:**  
  لكل كيان (سنوات، فصول، أشهر) صلاحيات منفصلة للوصول، الإضافة، التعديل، الحذف، والتصدير.

- **سيد (Seeder) لتهيئة البيانات الأولية:**  
  يقوم بنقل البيانات من نموذج `Tss\School\Models\StudyYear` القديم إلى الجداول الجديدة مع إنشاء الفصول والأشهر تلقائياً.

---

## المتطلبات

- Smart Nano Soft App vبناء على الإصدار **3.x** أو **2.x** (مدعوم).
- PHP 7.4+.
- الإضافات التالية مثبتة ومفعّلة:
  - `RainLab.Translate`
  - `RainLab.Location`
  - `RainLab.User`
  - `Tss.Tools`
  - `Tss.BarcodeGenerator`
  - `Tss.Basic`
  - `Tss.School` (لتشغيل السيد – Seeder)

---

## التثبيت والترقية

### تثبيت جديد

1. انسخ مجلد `tss/studyyear` إلى `plugins/tss/studyyear`.
2. قم بتشغيل الأمر:
   ```bash
   php artisan plugin:refresh Tss.Studyyear
   ```
   ستُنشأ الجداول ويسجّل الإصدار.

3. (اختياري) لتعبئة البيانات من `Tss.School`، شغّل السيد:
   ```bash
   php artisan tss:seed Tss.Studyyear
   ```

### الترقية من إصدار سابق (قبل 1.0.6)

1. استبدل جميع الملفات بالنسخة الجديدة.
2. قم بتشغيل:
   ```bash
   php artisan plugin:refresh Tss.Studyyear
   ```
   ستتم إضافة عمود `is_published_results` إلى الجداول ولن تُمسح البيانات الحالية.
3. مسح الكاش:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

## النماذج (Models)

### 1. `Period` – السنة الدراسية

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `name` | string | اسم السنة (مثال: 2025-2026) |
| `calendar_type` | string | نوع التقويم (`years`) |
| `from_date` | timestamp | تاريخ بداية السنة |
| `to_date` | timestamp | تاريخ نهاية السنة |
| `is_published` | boolean | نشر السنة على الموقع |
| `published_at` | timestamp | تاريخ بدء النشر |
| `unpublished_at` | timestamp | تاريخ انتهاء النشر |
| `is_published_results` | boolean | نشر السنة في نتائج التقارير |
| `is_default` | boolean | سنة افتراضية (تستخدم تلقائياً في الـ API) |
| `is_active` | boolean | تفعيل السنة |

**العلاقات:**
- `hasMany` → `semsters`, `months`, `studentRecords`
- `belongsTo` → `Company`, `Department`

**الدوال الأساسية:**
- `getPrimary()`: إرجاع السنة الافتراضية (الأولى حسب `is_default` والنشطة).
- `getNameList()`: قائمة بالسنوات النشطة والمنشورة والمتحقق من صلاحيتها.
- `clearCache()`: مسح الكاش بعد أي تعديل.

### 2. `Semster` – الفصل الدراسي

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `ref_type` | string | `semster1` أو `semster2` |
| `study_year_id` | integer | معرف السنة الدراسية (ربط مع `Period`) |
| `from_date`, `to_date` | timestamp | تواريخ بداية ونهاية الفصل (اختيارية) |
| `is_published_results` | boolean | نشر نتائج الفصل |

**العلاقات:**
- `belongsTo` → `Period` (السنوات)
- `hasMany` → `months`

### 3. `Month` – الشهر الدراسي

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `month_num` | integer | رقم الشهر ضمن الفصل (1، 2، 3) |
| `month_number` | string | رمز الشهر (JAN, FEB, …) |
| `semsters_id` | integer | معرف الفصل |
| `study_year_id` | integer | معرف السنة |
| `from_date`, `to_date` | timestamp | تواريخ بداية ونهاية الشهر (يتم احتسابها تلقائياً عند الإنشاء) |

**العلاقات:**
- `belongsTo` → `Semster`, `Period`

**ملاحظة:** يتم حساب `to_date` تلقائياً بإضافة 31 يوماً على `from_date` (يمكن تعديله يدوياً).

---

## واجهة المستخدم الخلفية

### القوائم (Menus)

تظهر الإضافة في لوحة التحكم تحت بند **السنوات الدراسية** (قائمة رئيسية) وفي القائمة الجانبية للإعدادات.

**المسارات:**
- السنوات: `backend/tss/studyyear/periods`
- الفصول: `backend/tss/studyyear/semsters`
- الأشهر: `backend/tss/studyyear/months`

### القوائم (Lists)

#### `columns.yaml` لكل كيان

| العمود | الوصف | مرئي افتراضياً |
|--------|-------|----------------|
| `id` | المعرف | لا |
| `code` | الكود | لا |
| `name` | الاسم | نعم |
| `companys_id` | الشركة | لا |
| `departments_id` | الفرع | لا |
| `calendar_type` | نوع التقويم | نعم |
| `from_date` | من تاريخ | نعم |
| `to_date` | إلى تاريخ | نعم |
| `is_published_results` | نشر النتائج | لا |
| `is_published` | منشور | لا |
| `published_at` | تاريخ النشر | لا |
| `unpublished_at` | تاريخ انتهاء النشر | لا |
| `is_default` | افتراضي | نعم |
| `is_active` | نشط | نعم |
| `status` | الحالة | لا |
| `sort_order` | ترتيب | لا |

### نماذج الإدخال (Forms)

#### `fields.yaml` يدعم:

- **تبويبات**: Basic, Details, Options.
- **حقول منسدلة تعتمد على أخرى (`dependsOn`):**  
  - اختيار الفرع → تحديد السنة → تحديد الفصل.
- **حقول التاريخ:** `from_date`, `to_date` مع حساب تلقائي لـ `to_date` عند تغيير `from_date`.
- **مفاتيح التبديل (Switch):** `is_default`, `is_active`, `is_published`, `is_published_results`.
- **حقول التاريخ المؤقت:** `published_at`, `unpublished_at` تظهر فقط عند تفعيل `is_published`.

### الفلاتر (Filters)

#### `config_filter.yaml` متقدم:

| الفلتر | النوع | الوصف |
|--------|-------|-------|
| `departments_id` | مجموعة (Dropdown) | تصفية حسب الفرع |
| `study_year_id` | مجموعة | يعتمد على `departments_id` |
| `semsters_id` | مجموعة (للأشهر) | يعتمد على `study_year_id` |
| `month_num` | مجموعة | تصفية رقم الشهر |
| `ref_type` | مجموعة (للفصول) | `semster1` / `semster2` |
| `is_active` | مفتاح تبديل | نشط / غير نشط |
| `is_published` | مربع اختيار | يعرض فقط السجلات المنشورة (يستخدم `scopeIsPublished`) |
| `is_not_published` | مربع اختيار | السجلات غير المنشورة |
| `is_published_results` | مفتاح تبديل | 0/1 |
| `is_default` | مفتاح تبديل | الافتراضي فقط |
| `id` | نص | بحث حسب المعرف |
| `created_at`, `updated_at` | نطاق تاريخي | تصفية حسب تاريخ الإنشاء أو التحديث |

---

## الصلاحيات (Permissions)

تم تعريف الصلاحيات لكل من `periods`، `semsters`، `months` في `Plugin::registerPermissions()`.

**مثال – صلاحيات السنوات الدراسية:**

| المفتاح | الوصف |
|---------|-------|
| `tss.studyyear.periods.access` | الوصول إلى قائمة السنوات |
| `tss.studyyear.periods.add` | إضافة سنة جديدة |
| `tss.studyyear.periods.edit` | تعديل سنة |
| `tss.studyyear.periods.delete` | حذف سنة |
| `tss.studyyear.periods.export` | تصدير بيانات السنوات |

نفس الهيكل ينطبق على `semsters` و `months`.

> **ملاحظة:** يتم تعيين الصلاحيات لمجموعة "مطور" (developer) افتراضياً، يمكن للمسؤول توزيعها على أدوار أخرى.

---

## الإعدادات (Config)

يوجد ملف `config.php` في `plugins/tss/studyyear/config/config.php` يدعم متغيرات البيئة التالية:

| المتغير | القيمة الافتراضية | الوصف |
|---------|------------------|-------|
| `TSS_STUDYYEAR_CHECK_DATE_YEAR` | `true` | تفعيل فحص صلاحية تاريخ السنة (عدم عرض السنوات المنتهية) |
| `TSS_STUDYYEAR_CHECK_DATE_SEMSTERS` | `true` | تفعيل فحص صلاحية تاريخ الفصل |
| `TSS_STUDYYEAR_CHECK_DATE_MONTHS` | `true` | تفعيل فحص صلاحية تاريخ الشهر |

يمكن تعطيل الفحص لوضع `false` في ملف `.env`:

```ini
TSS_STUDYYEAR_CHECK_DATE_YEAR=false
```

---

## قاعدة البيانات (الجداول)

### `tss_studyyear_periods`
- الحقول الأساسية: `id`, `code`, `barcode`, `name`, `short_description`, `description`, `calendar_type`, `from_date`, `to_date`
- حقول النشر: `is_published`, `published_at`, `unpublished_at`, `is_published_results`
- حقول الإعدادات: `is_default`, `is_active`, `status`, `sort_order`
- حقول العلاقات: `companys_id`, `departments_id`
- حقول التدقيق: `created_by`, `updated_by`, `deleted_by`, `created_at`, `updated_at`, `deleted_at`

### `tss_studyyear_semsters`
- إضافات: `study_year_id`, `ref_type`, `month_num` (غير موجود)، `month_number` (غير موجود) – هذه الحقول خاصة بـ `months`.
- باقي الحقول مشابهة لـ `periods` مع `ref_type` إجباري.

### `tss_studyyear_months`
- إضافات: `study_year_id`, `semsters_id`, `month_num`, `month_number`
- باقي الحقول مشابهة.

جميع الجداول تستخدم `InnoDB` وتدعم `SoftDelete`.

---

## أدوات السيد (Seeder)

### `SeederSchoolStudyyears`

يقوم هذا السيد بنقل البيانات من نموذج `Tss\School\Models\StudyYear` (الإضافة القديمة `Tss.School`) إلى نظام `Tss.Studyyear`.

**المنطق:**
- لكل سنة دراسية موجودة في `tss_school_studyyears`، يتم إنشاء سجل في `tss_studyyear_periods`.
- ثم يتم إنشاء فصلين دراسيين (`semster1`, `semster2`) لكل سنة.
- لكل فصل، يتم إنشاء ثلاثة أشهر دراسية (`month_num` 1,2,3) مع تعيين `month_number` تلقائياً (JAN, FEB, MAR للفصل الأول، و APR, MAY, JUN للفصل الثاني).

**التشغيل:**
```bash
php artisan tss:seed Tss.Studyyear
```

> **ملاحظة:** يتطلب وجود إضافة `Tss.School` مثبتة ومفعّلة، وإلا سيرمي السيد استثناءً.

---

## التكامل مع الـ API – `Nano.StudyyearApi`

توفر إضافة `Nano.StudyyearApi` واجهة RESTful للوصول إلى بيانات السنوات والفصول والأشهر. تعتمد على نماذج `Tss.Studyyear` وتدعم:

- نقاط النهاية: `periods`, `semsters`, `months`.
- الفلاتر: `is_published`, `is_published_results`, `is_default`, `study_year_id`, `semsters_id`, `month_num` … إلخ.
- التضمين (includes): `company`, `department`, `period`, `semster`.
- التخزين المؤقت والترقيم.

**مثال طلب لجلب الأشهر المنشورة للنتائج فقط:**
```bash
GET /api/v1/studyyear/months?is_published_results=1&study_year_id=5
```

لمزيد من التفاصيل، راجع [توثيق `Nano.StudyyearApi`](./Docs-StudyyearApi-ar.md).

---

## أمثلة الاستخدام

### إنشاء سنة دراسية جديدة عبر واجهة الخلفية

1. اذهب إلى `السنوات الدراسية > السنة الدراسية`.
2. اضغط **إضافة جديد**.
3. أدخل الاسم: `2026-2027`.
4. اختر تاريخ البداية: `2026-09-01`، تاريخ النهاية: `2027-08-31`.
5. فعّل `نشر على الموقع`، وحدد `published_at` = `2026-09-01`.
6. فعّل `نشر النتائج في التطبيق` إذا أردت ظهورها في تطبيق النتائج.
7. احفظ.

### تعيين سنة افتراضية

- من قائمة السنوات، افتح السنة المرغوبة.
- فعّل خيار `افتراضي`.
- احفظ. ستُلغى الخاصية الافتراضية عن السنوات الأخرى تلقائياً.

### استخدام النطاقات في الكود

```php
// جلب السنة الافتراضية النشطة والمنشورة
$defaultPeriod = Period::getPrimary();

// جلب جميع الفصول النشطة للسنة 5
$semsters = Semster::IsCompany()->where('study_year_id', 5)->isActive()->get();

// جلب الأشهر المنشورة فقط (مع مراعاة التواريخ)
$months = Month::isPublished()->get();
```

---

## الإصدارات وتاريخ التحديثات

| الإصدار | التاريخ | التغييرات |
|---------|---------|------------|
| 1.0.1 | – | تهيئة الإضافة |
| 1.0.2 | – | إنشاء جدول `tss_studyyear_periods` |
| 1.0.3 | – | إنشاء جدول `tss_studyyear_semsters` |
| 1.0.4 | – | إنشاء جدول `tss_studyyear_months` |
| 1.0.5 | – | سيد لنقل البيانات من `Tss.School` |
| **1.0.6** | 2026-05-30 | إضافة عمود `is_published_results` لجميع الجداول |
| **1.0.7** | 2026-05-30 | دعم حقول النشر المؤقت (`is_published`, `published_at`, `unpublished_at`) في واجهة المستخدم، دعم الفلاتر المتقدمة، إضافة `is_published_results` إلى القوائم والنماذج والفلاتر |

---

## الخلاصة

إضافة `Tss.Studyyear` هي العمود الفقري لإدارة البنية الزمنية للعام الدراسي في نظام نانو سوفت. بفضل تصميمها المرن، ونظام النطاقات القوي، ودعم النشر المؤقت وفصل نتائج التقارير، يمكن لأي نظام تعليمي الاعتماد عليها لبناء جداول زمنية دقيقة وقابلة للتوسع.

توفر الإضافة أيضاً تكاملاً سلساً مع الـ API (`Nano.StudyyearApi`) مما يسمح للتطبيقات الخارجية بالاستفادة من بيانات السنوات والفصول والأشهر بشكل آمن وسريع.

---

**المراجع:**
- [توثيق `Nano.StudyyearApi`](./Docs-StudyyearApi-ar.md)
