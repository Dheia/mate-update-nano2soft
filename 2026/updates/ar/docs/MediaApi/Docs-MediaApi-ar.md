# توثيق إضافة `Nano.MediaApi`

## مقدمة

توفر إضافة **Nano.MediaApi** واجهة برمجة تطبيقات (API) متكاملة لإدارة محتوى الوسائط المتعددة داخل نظام NanoSoft APP، وذلك وفقاً لنمط `Nano.*Api`. تم تصميم الإضافة لتسهيل عملية جلب **الصور** و **الفيديوهات** و **الملفات** بتنظيم هرمي عبر التصنيفات (`Categories`) والحاويات الوسيطة (الألبومات `Albums`، قوائم التشغيل `Playlists`، حزم الملفات `Filelists`).  

جميع نقاط النهاية تعمل بطريقة `GET` فقط (قراءة البيانات)، وتدعم مجموعة واسعة من **الفلاتر**، **البحث**، **الترتيب**، **تضمين العلاقات** (`include`)، **استبعاد الحقول** (`exclude`)، والتخزين المؤقت الاختياري لتحسين الأداء.  

يجب أن يكون المستخدم **مسجل الدخول** (JWT token أو session) للوصول إلى معظم نقاط النهاية، حيث يتم تطبيق نطاقات الصلاحيات الخاصة بالشركة والقسم والمستخدم تلقائياً (حسب الإعدادات).  

---

## فهرس المحتويات

1. [هيكلية مكتبة الوسائط](#هيكلية-مكتبة-الوسائط)
2. [نقاط النهاية العامة والمعاملات المشتركة](#نقاط-النهاية-العامة-والمعاملات-المشتركة)
   - [البحث `q`](#البحث-q)
   - [الترتيب `orderBy` / `orderDirection`](#الترتيب-orderby--orderdirection)
   - [تقسيم الصفحات `page` / `per_page`](#تقسيم-الصفحات-page--per_page)
   - [استبعاد الحقول `exclude`](#استبعاد-الحقول-exclude)
   - [تضمين العلاقات `include`](#تضمين-العلاقات-include)
3. [جزء الصور](#جزء-الصور)
   - [1. التصنيفات (Categories) – `reftype=image`](#1-التصنيفات-categories--reftypeimage)
   - [2. الألبومات (Albums)](#2-الألبومات-albums)
   - [3. الصور (Images)](#3-الصور-images)
4. [جزء الفيديوهات](#جزء-الفيديوهات)
   - [1. التصنيفات (Categories) – `reftype=video`](#1-التصنيفات-categories--reftypevideo)
   - [2. قوائم التشغيل (Playlists)](#2-قوائم-التشغيل-playlists)
   - [3. الفيديوهات (Videos)](#3-الفيديوهات-videos)
5. [جزء الملفات](#جزء-الملفات)
   - [1. التصنيفات (Categories) – `reftype=file`](#1-التصنيفات-categories--reftypefile)
   - [2. حزم الملفات (Filelists)](#2-حزم-الملفات-filelists)
   - [3. الملفات (Files)](#3-الملفات-files)
6. [التحقق من آخر تحديث (activelystats)](#التحقق-من-آخر-تحديث-activelystats)
7. [ملاحظات عامة](#ملاحظات-عامة)

---

## هيكلية مكتبة الوسائط

تعتمد الإضافة على التسلسل الهرمي التالي:

- **الصور**  
  `تصنيفات (Categories)` ← `ألبومات (Albums)` ← `صور (Images)`

- **الفيديوهات**  
  `تصنيفات (Categories)` ← `قوائم تشغيل (Playlists)` ← `فيديو (Videos)`

- **الملفات**  
  `تصنيفات (Categories)` ← `حزم ملفات (Filelists)` ← `ملف (Files)`

كل عنصر من هذه العناصر له كائن خاص به في قاعدة البيانات، مع وجود علاقات `belongsTo` / `hasMany` مناسبة. يتم تصفية البيانات تلقائياً بحسب `companys_id` و `departments_id` للمستخدم الحالي (إن وجد).

---

## نقاط النهاية العامة والمعاملات المشتركة

جميع نقاط النهاية التالية تستقبل نفس مجموعة المعاملات (parameters) الاختيارية، مع بعض الإضافات الخاصة بكل مورد.

### البحث `q`

نص البحث يطبق على الحقول: `name`، `short_description`، `description`، `meta_title`، `meta_description`، `keywords`.

```
GET /api/v1/media/images?q=طبيعة
```

### الترتيب `orderBy` / `orderDirection`

- `orderBy`: اسم العمود (مثل `created_at`، `sort_order`، `name`).
- `orderDirection`: `asc` أو `desc` (افتراضي `desc`).

إذا لم يتم تمرير `orderBy`، يستخدم الترتيب الافتراضي من ملف الإعدادات (`config.php`) والذي عادة ما يكون `sort_order asc`.

### تقسيم الصفحات `page` / `per_page`

- `per_page`: عدد النتائج في الصفحة (افتراضي 15).
- `page`: رقم الصفحة المطلوبة (افتراضي 1).

### استبعاد الحقول `exclude`

يتم تمرير أسماء الحقول المراد استبعادها من الاستجابة مفصولة بفواصل:

```
GET /api/v1/media/images?exclude=created_at,updated_at
```

### تضمين العلاقات `include`

يتم تمرير أسماء العلاقات المراد تضمينها في نفس الاستجابة (لتجنب استدعاءات إضافية). العلاقات المتاحة تختلف حسب المورد (انظر الجداول أدناه). مثال:

```
GET /api/v1/media/albums?include=photos,categorie
```

---

## جزء الصور

### 1. التصنيفات (Categories) – `reftype=image`

**نقطة النهاية:**  
`GET /api/v1/media/categories?reftype=image`

يتم جلب التصنيفات الخاصة بالصور فقط.

#### الفلاتر الخاصة (بالإضافة إلى العامة)

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `parent_id` | integer | جلب التصنيفات الفرعية لتصنيف أب محدد. |
| `isActive` | boolean | `true` لعرض النشط فقط (افتراضي `true`). |
| `isNotHidden` | boolean | `true` لاستبعاد المخفي (افتراضي `true`). |
| `isHidden` | boolean | عرض المخفي فقط (افتراضي `false`). |
| `IsMain` | boolean | تصنيفات رئيسية فقط (افتراضي `false`). |
| `IsSub` | boolean | تصنيفات فرعية فقط (افتراضي `false`). |
| `companys_id` | integer | فلترة حسب الشركة. |
| `departments_id` | integer | فلترة حسب القسم. |
| `country_id` | integer/string | فلترة حسب البلد (عبر العلاقة `Department`). |
| `state_id` | integer/string | فلترة حسب المدينة (عبر العلاقة `Department`). |
| `categorysId` | string | فلترة حسب معرفات تصنيفات عامة (pcategories). |
| `tagsId` | string | فلترة حسب معرفات الوسوم (tags). |
| `menus_id` | integer | تطبيق شروط قائمة مخصصة (Menu). |
| `isFavorites` | boolean | جلب التصنيفات المفضلة لدى المستخدم الحالي. |
| `isLikes` | boolean | جلب التصنيفات التي أعجب بها المستخدم. |
| `isBookmarks` | boolean | جلب التصنيفات المحفوضة (Bookmarks). |
| `isReactions` | boolean | جلب التصنيفات التي تفاعل معها المستخدم. |
| `with_count` | boolean | إضافة عدد الفيديوهات/الألبومات المرتبطة (مثل `videos_count`). |
| `isDisplayEmpty` | boolean | `false` لعرض التصنيفات التي تحتوي على فيديوهات فقط (افتراضي `true`). |
| `displayTo` | integer | الحد الأدنى لعدد الفيديوهات عند تفعيل `isDisplayEmpty=false` (افتراضي 1). |

#### العلاقات القابلة للتضمين (`include`)

| العلاقة | الوصف |
|---------|-------|
| `companys` | بيانات الشركة. |
| `department` | بيانات القسم. |
| `parent` | التصنيف الأب. |
| `children` | التصنيفات الفرعية (مع دعم الصفحات). |
| `tags` | الوسوم المرتبطة. |
| `pcategories` | التصنيفات العامة (من إضافة `Nano.TagsApi`). |
| `albums` | الألبومات التابعة لهذا التصنيف. |
| `playlists` | قوائم التشغيل التابعة (للتصنيفات من نوع `video`). |
| `filelists` | حزم الملفات التابعة (للتصنيفات من نوع `file`). |
| `videos` | الفيديوهات التابعة مباشرة (نادراً). |
| `user_object_favorite` | كائن الإعجاب الخاص بالمستخدم (إن وجد). |
| `user_object_like` | كائن الإعجاب (Like). |
| `user_object_bookmark` | كائن الحفظ (Bookmark). |
| `user_object_reaction` | كائن التفاعل (Reaction). |

#### مثال: جلب التصنيفات الرئيسية للصور مع تضمين الألبومات

```http
GET /api/v1/media/categories?reftype=image&IsMain=1&include=albums
```

#### مثال: جلب التصنيفات التي تحتوي على فيديوهات على الأقل

```http
GET /api/v1/media/categories?reftype=video&isDisplayEmpty=false&displayTo=1
```

#### مثال: استبعاد بعض الحقول

```http
GET /api/v1/media/categories?exclude=created_at,updated_at,properties
```

---

### 2. الألبومات (Albums)

**نقطة النهاية:**  
`GET /api/v1/media/albums`

#### الفلاتر الخاصة

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `categories_id` أو `parent_id` | integer | جلب الألبومات التابعة لتصنيف معين. |
| `isActive` | boolean | `true` للألبومات النشطة. |
| `isPublished` | boolean | `true` للمنشورة. |
| `isNotHidden` / `isHidden` | boolean | عرض غير المخفي/المخفي. |
| `IsMain` / `IsSub` | boolean | ألبومات رئيسية/فرعية. |
| `companys_id` / `departments_id` | integer | فلترة حسب الشركة/القسم. |
| `country_id` / `state_id` | integer/string | فلترة حسب البلد/المدينة. |
| `isDisplayEmpty` | boolean | `false` لعرض الألبومات التي تحتوي على صور فقط. |
| `displayTo` | integer | الحد الأدنى لعدد الصور عند `isDisplayEmpty=false`. |
| `isFavorites`, `isLikes`, `isBookmarks`, `isReactions` | boolean | جلب بناءً على تفاعلات المستخدم. |
| `with_count` | boolean | إضافة `videos_count` (في الألبوم، الاسم محفوظ). |

#### العلاقات القابلة للتضمين (`include`)

| العلاقة | الوصف |
|---------|-------|
| `companys` | الشركة. |
| `department` | القسم. |
| `categorie` | التصنيف المرتبط (الجد). |
| `parent` | الألبوم الأب (إذا وجد). |
| `children` | الألبومات الفرعية (مع دعم الصفحات). |
| `tags` | الوسوم. |
| `pcategories` | التصنيفات العامة. |
| `photos` | الصور التابعة لهذا الألبوم (كائن `Image`). |
| `user_object_*` | كائنات تفاعل المستخدم. |

#### مثال: جلب كل الألبومات مع الصور

```http
GET /api/v1/media/albums?include=photos
```

#### مثال: جلب ألبومات تصنيف معين (categories_id=2) مع تضمين العلاقات الأساسية

```http
GET /api/v1/media/albums?categories_id=2&include=categorie,tags,pcategories
```

#### مثال: جلب ألبوم واحد بالمعرف 4

```http
GET /api/v1/media/albums/4
```

---

### 3. الصور (Images)

**نقطة النهاية:**  
`GET /api/v1/media/images`

#### الفلاتر الخاصة

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `albums_id` | integer | جلب الصور التابعة لألبوم معين. |
| `categories_id` | integer | فلترة الصور حسب التصنيف (يدعم **الفلترة المتقدمة** – يشمل الصور التي ينتمي ألبومها إلى تصنيف أب). |
| `isActive` | boolean | `true` للصور النشطة. |
| `isPublished` | boolean | `true` للمنشورة. |
| `companys_id` / `departments_id` | integer | فلترة حسب الشركة/القسم. |
| `country_id` / `state_id` | integer/string | فلترة حسب البلد/المدينة. |
| `categorysId` | string | فلترة حسب التصنيفات العامة (pcategories). |
| `tagsId` | string | فلترة حسب الوسوم. |
| `menus_id` | integer | تطبيق شروط قائمة مخصصة. |
| `isFavorites`, `isLikes`, `isBookmarks`, `isReactions` | boolean | فلترة بناءً على تفاعلات المستخدم. |
| `q` | string | بحث في `name`, `short_description`, `description`, إلخ. |

#### العلاقات القابلة للتضمين (`include`)

| العلاقة | الوصف |
|---------|-------|
| `companys` | الشركة. |
| `department` | القسم. |
| `categorie` | التصنيف المباشر للصورة. |
| `album` | الألبوم الذي تنتمي إليه الصورة. |
| `tags` | الوسوم. |
| `pcategories` | التصنيفات العامة. |
| `user_object_*` | كائنات تفاعل المستخدم (مفضل، إعجاب، إلخ). |

#### مثال: جلب الصور التابعة لألبوم رقم 4

```http
GET /api/v1/media/images?albums_id=4
```

#### مثال: جلب الصور حسب تصنيف (يشمل التصنيفات الفرعية للألبوم)

```http
GET /api/v1/media/images?categories_id=2
```

> **ملاحظة:** منذ الإصدار 1.0.2، أصبح `categories_id` يدعم أيضاً جلب الصور التي تتبع ألبوماً يكون `parent_id` الخاص بألبومها مساوياً للتصنيف الممرر.

#### مثال: البحث عن صور تحتوي كلمة "شعار"

```http
GET /api/v1/media/images?q=شعار
```

#### مثال: جلب صور مع تضمين الألبوم والوسوم

```http
GET /api/v1/media/images?include=album,tags
```

---

## جزء الفيديوهات

### 1. التصنيفات (Categories) – `reftype=video`

نفس نقاط نهاية التصنيفات العامة ولكن بتمرير `reftype=video`. جميع الفلاتر والعلاقات المذكورة في قسم تصنيفات الصور تنطبق أيضاً على الفيديو، مع اختلاف أن العلاقات المتاحة قد تشمل `playlists` و `videos` بدلاً من `albums`.

**نقطة النهاية:**  
`GET /api/v1/media/categories?reftype=video`

#### مثال: جلب التصنيفات الخاصة بالفيديوهات مع تضمين قوائم التشغيل

```http
GET /api/v1/media/categories?reftype=video&include=playlists
```

---

### 2. قوائم التشغيل (Playlists)

**نقطة النهاية:**  
`GET /api/v1/media/playlists`

#### الفلاتر الخاصة

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `categories_id` أو `parent_id` | integer | قوائم التشغيل التابعة لتصنيف معين. |
| `isActive`, `isPublished` | boolean | الحالة. |
| `isNotHidden` / `isHidden` | boolean | إظهار/إخفاء. |
| `IsMain` / `IsSub` | boolean | رئيسية/فرعية. |
| `companys_id` / `departments_id` | integer | فلترة. |
| `country_id` / `state_id` | integer/string | فلترة جغرافية. |
| `isDisplayEmpty` | boolean | `false` لعرض القوائم التي تحتوي على فيديوهات فقط. |
| `with_count` | boolean | إضافة `videos_count`. |
| `isFavorites`، إلخ | boolean | تفاعلات المستخدم. |

#### العلاقات القابلة للتضمين (`include`)

| العلاقة | الوصف |
|---------|-------|
| `companys`, `department` | بيانات الشركة والقسم. |
| `categorie` | التصنيف المرتبط. |
| `parent`, `children` | هيكل الشجرة (إن وجد). |
| `tags`, `pcategories` | الوسوم والتصنيفات العامة. |
| `videos` | الفيديوهات التابعة لهذه القائمة. |
| `user_object_*` | كائنات تفاعل المستخدم. |

#### مثال: جلب قوائم التشغيل مع الفيديوهات

```http
GET /api/v1/media/playlists?include=videos
```

#### مثال: جلب قائمة تشغيل تابعة لتصنيف رئيسي (categories_id=3)

```http
GET /api/v1/media/playlists?categories_id=3
```

#### مثال: جلب قائمة تشغيل واحدة بالمعرف 6

```http
GET /api/v1/media/playlists/6
```

---

### 3. الفيديوهات (Videos)

**نقطة النهاية:**  
`GET /api/v1/media/videos`

#### الفلاتر الخاصة

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `playlists_id` | integer | الفيديوهات التابعة لقائمة تشغيل معينة. |
| `categories_id` | integer | فلترة حسب التصنيف (يدعم التصنيفات الأم عبر علاقة `playlist`). |
| `isActive`, `isPublished` | boolean | الحالة. |
| `companys_id` / `departments_id` | integer | فلترة. |
| `country_id` / `state_id` | integer/string | فلترة جغرافية. |
| `categorysId`, `tagsId` | string | فلترة حسب التصنيفات/الوسوم. |
| `menus_id` | integer | تطبيق شروط قائمة. |
| `isFavorites`، إلخ | boolean | تفاعلات المستخدم. |
| `q` | string | بحث في النصوص. |

#### العلاقات القابلة للتضمين (`include`)

| العلاقة | الوصف |
|---------|-------|
| `companys`, `department` | الشركة والقسم. |
| `categorie` | التصنيف المباشر. |
| `playlist` | قائمة التشغيل التي ينتمي إليها الفيديو. |
| `tags`, `pcategories` | الوسوم والتصنيفات العامة. |
| `user_object_*` | كائنات التفاعل. |

#### مثال: جلب الفيديوهات التابعة لقائمة تشغيل رقم 6

```http
GET /api/v1/media/videos?playlists_id=6
```

#### مثال: جلب فيديوهات حسب تصنيف (مع تضمين قائمة التشغيل)

```http
GET /api/v1/media/videos?categories_id=3&include=playlist
```

#### مثال: البحث عن فيديو بكلمة "تعليمي"

```http
GET /api/v1/media/videos?q=تعليمي
```

#### هيكل استجابة الفيديو (مختصر)

```json
{
  "id": 7,
  "name": "الفيديو الاول",
  "description": "...",
  "image": { "original": "...", "thumb": "..." },
  "video": { "file_name": "...", "path": "...", "file_size": 783126 },
  "object_type": "Tss\\Media\\Models\\Video"
}
```

---

## جزء الملفات

### 1. التصنيفات (Categories) – `reftype=file`

**نقطة النهاية:**  
`GET /api/v1/media/categories?reftype=file`

نفس الفلاتر والعلاقات السابقة، مع دعم `filelists` بدلاً من `albums`/`playlists`.

#### مثال: جلب تصنيفات الملفات مع تضمين حزم الملفات

```http
GET /api/v1/media/categories?reftype=file&include=filelists
```

---

### 2. حزم الملفات (Filelists)

**نقطة النهاية:**  
`GET /api/v1/media/filelists`

#### الفلاتر الخاصة

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `categories_id` أو `parent_id` | integer | حزم الملفات التابعة لتصنيف معين. |
| `isActive`, `isPublished` | boolean | الحالة. |
| `isNotHidden` / `isHidden` | boolean | إظهار/إخفاء. |
| `IsMain` / `IsSub` | boolean | رئيسية/فرعية. |
| `companys_id` / `departments_id` | integer | فلترة. |
| `country_id` / `state_id` | integer/string | فلترة جغرافية. |
| `isDisplayEmpty` | boolean | `false` لعرض الحزم التي تحتوي على ملفات فقط. |
| `with_count` | boolean | إضافة `files_count`. |
| `isFavorites`، إلخ | boolean | تفاعلات المستخدم. |

#### العلاقات القابلة للتضمين (`include`)

| العلاقة | الوصف |
|---------|-------|
| `companys`, `department` | الشركة والقسم. |
| `categorie` | التصنيف المرتبط. |
| `parent`, `children` | الشجرة. |
| `tags`, `pcategories` | الوسوم والتصنيفات العامة. |
| `files` | الملفات التابعة للحزمة. |
| `user_object_*` | كائنات التفاعل. |

#### مثال: جلب حزم الملفات مع تضمين الملفات

```http
GET /api/v1/media/filelists?include=files
```

#### مثال: جلب حزمة ملفات تابعة لتصنيف رئيسي

```http
GET /api/v1/media/filelists?categories_id=5
```

#### مثال: جلب حزمة واحدة بالمعرف 7

```http
GET /api/v1/media/filelists/7
```

---

### 3. الملفات (Files)

**نقطة النهاية:**  
`GET /api/v1/media/files`

#### الفلاتر الخاصة

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `filelists_id` | integer | جلب الملفات التابعة لحزمة معينة. |
| `categories_id` | integer | فلترة حسب التصنيف (يدعم التصنيفات الأم عبر علاقة `filelist`). |
| `isActive`, `isPublished` | boolean | الحالة. |
| `is_downloadable` | boolean | (ضمن البيانات) ليس فلتراً مباشراً. |
| `companys_id` / `departments_id` | integer | فلترة. |
| `country_id` / `state_id` | integer/string | فلترة جغرافية. |
| `categorysId`, `tagsId` | string | فلترة. |
| `menus_id` | integer | شروط قائمة. |
| `isFavorites`، إلخ | boolean | تفاعلات المستخدم. |
| `q` | string | بحث. |

#### العلاقات القابلة للتضمين (`include`)

| العلاقة | الوصف |
|---------|-------|
| `companys`, `department` | الشركة والقسم. |
| `categorie` | التصنيف المباشر. |
| `filelist` | حزمة الملفات التي ينتمي إليها الملف. |
| `tags`, `pcategories` | الوسوم والتصنيفات العامة. |
| `user_object_*` | كائنات التفاعل. |

#### مثال: جلب الملفات التابعة لحزمة رقم 7

```http
GET /api/v1/media/files?filelists_id=7
```

#### مثال: جلب ملفات حسب تصنيف معين مع تضمين الحزمة

```http
GET /api/v1/media/files?categories_id=5&include=filelist
```

#### مثال: البحث عن ملف باسم "دليل"

```http
GET /api/v1/media/files?q=دليل
```

#### هيكل استجابة الملف (مختصر)

```json
{
  "id": 8,
  "name": "ملف تعليمي لنظام نانو سوفت",
  "file": { "file_name": "document.pdf", "path": "https://...", "file_size": 39007 },
  "is_downloadable": 1,
  "object_type": "Tss\\Media\\Models\\File"
}
```

---

## التحقق من آخر تحديث (activelystats)

كل نقطة نهاية من نقاط `index` تدعم نقطة `activelystats` التي تسمح بمعرفة ما إذا كانت البيانات قد تغيرت بعد تاريخ معين. هذا مفيد للتخزين المؤقت على مستوى التطبيق العميل.

**نقطة النهاية:**  
`GET /api/v1/media/{resource}/activelystats`

**المعامل:**  
- `date` (اختياري): تاريخ بصيغة `Y-m-d H:i:s`. الافتراضي هو الوقت الحالي.

**الاستجابة:**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,      // true إذا كان هناك تحديث بعد التاريخ الممرر
    "check_date": "2024-01-12 16:24:45",
    "last_updated": "2024-01-12 16:25:04",
    "other_updated": {
      "activity_cache.image": "2024-01-12 16:25:04",
      "activity_cache.album": "2024-01-12 16:25:04"
    }
  }
}
```

**مثال:**  
```http
GET /api/v1/media/images/activelystats?date=2024-01-01%2000:00:00
```

---

## ملاحظات عامة

- جميع الاستجابات تعيد بيانات بصيغة JSON، وفق هيكل `{ "data": [...], "meta": { "pagination": {...} } }` عند استخدام `index`، أو الكائن مباشرة عند استخدام `show`.
- التخزين المؤقت على مستوى الإضافة مفعل افتراضياً إذا كان `nano.api::api_enable_cache` يساوي `true`. يمكن مسح الكاش عبر `php artisan cache:clear`.
- الفلترة بـ `categories_id` في الصور والفيديوهات والملفات تدعم **التصنيفات الأم** عبر العلاقات (`album`، `playlist`، `filelist`) منذ الإصدار 1.0.2، مما يعني أن تمرير `categories_id=2` سيجلب أيضاً العناصر التي يكون تصنيفها الفرعي (التابع للألبوم/قائمة التشغيل) هو 2.
- يمكن استخدام `include` مع `exclude` معاً. الأولوية للاستبعاد (إذا تم استبعاد حقل يظهر في علاقة مضمنة، فقد يُزال من البيانات الرئيسية فقط).
- لا توجد نقاط نهاية لإنشاء أو تعديل أو حذف الوسائط عبر هذا الـ API (للحفاظ على الأمان وترك هذه العمليات للوحة التحكم أو أدوات أخرى). الإضافة تركز على **القراءة فقط**.
- لدعم `exclude` ديناميكياً، تم إضافة `scopeExclude` لجميع النماذج (Categories, Album, Image, Playlist, Video, Filelist, File) داخل `Plugin.php`.
- يمكن تخصيص الترتيب الافتراضي لكل مورد عبر ملف الإعدادات `config.php` أو متغيرات البيئة (مثل `NANO_MEDIAAPI_IMAGES_ORDER_BY=sort_order`).

---

## وثائق إضافية 

- [توثيق مكتبة الوسائط API (MediaApi)](./docs/MediaApi/Docs-MediaApi-ar.md)
- [توثيق مختصر مكتبة الوسائط API (MediaApi)](./docs/MediaApi/Docs-MediaApi-Short-ar.md)
- [أمثلة عملية شاملة – إضافة MediaApi](./docs/MediaApi/Docs-MediaApi-Examples-ar.md)
- [أمثلة عملية مختصرة – إضافة MediaApi](./docs/MediaApi/Docs-MediaApi-Short-Examples-ar.md)

**آخر تحديث:** 2026-06-01  
**الإصدار الموثق:** 1.0.2 (يدعم الفلترة المتقدمة للتصنيفات الأم)
