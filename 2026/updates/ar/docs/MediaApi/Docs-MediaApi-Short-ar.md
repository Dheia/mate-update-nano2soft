# توثيق مختصر مكتبة الوسائط (Media Library API)

## مقدمة

يوفّر هذا التوثيق وصفًا كاملًا لأجزاء API الخاصة بمكتبة الوسائط من شركة **Smart Nano Soft**.  
تنقسم المكتبة إلى ثلاثة أقسام رئيسية:

- **الصور** ← تصنيفات ← ألبومات ← صور
- **الفيديوهات** ← تصنيفات ← قوائم تشغيل ← فيديو
- **الملفات** ← تصنيفات ← حزم ملفات (Filelists) ← ملف

جميع الاستدعاءات تكون بصيغة `GET`، وتُستخدم المعاملات (parameters) للتصفية، والتضمين، والبحث.

> **ملاحظة:** يمكن اعتبار `{base_url}` هو الرابط الأساسي لواجهة الـ API، وجميع المسارات أدناه مبنية بالنسبة له.

---

## 1. الصور (Images)

### 1.1 جلب تصنيفات الصور

```http
GET /api/v1/media/categories?reftype=image
```

**مثال**  
```
GET {base_url}/api/v1/media/categories?reftype=image
```

---

### 1.2 جلب الألبومات

يمكن جلب جميع الألبومات، أو فلترتها حسب تصنيف معين.

#### 1.2.1 جلب كل الألبومات

```http
GET /api/v1/media/albums
```

#### 1.2.2 جلب الألبومات حسب تصنيف محدد

```http
GET /api/v1/media/albums?categories_id={category_id}
```

**مثال**  
```
GET /api/v1/media/albums?categories_id=3
```

#### 1.2.3 تضمين الصور التابعة لكل ألبوم

استخدم معامل `include` بقيمة `photos`:

```http
GET /api/v1/media/albums?include=photos
```
أو مع التصنيف:
```http
GET /api/v1/media/albums?categories_id=3&include=photos
```

---

### 1.3 جلب الصور

```http
GET /api/v1/media/images
```

#### المعاملات الاختيارية

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `albums_id` | integer | جلب الصور التابعة لألبوم معين |
| `categories_id` | integer | فلترة الصور حسب تصنيف معين |
| `q` | string | نص البحث في الصور (اسم، وصف، إلخ) |

**أمثلة**

```
GET /api/v1/media/images?albums_id=5
GET /api/v1/media/images?categories_id=3
GET /api/v1/media/images?q=طبيعة
```

#### هيكل سجل الصورة

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `name` | string | اسم الصورة |
| `short_description` | text | وصف مختصر (نص عادي) |
| `description` | html | وصف تفصيلي (نص HTML) |
| `image` | object | ملف الصورة بعدة أحجام (تفاصيل المسارات والأبعاد) |

---

## 2. الفيديوهات (Videos)

### 2.1 جلب تصنيفات الفيديوهات

```http
GET /api/v1/media/categories?reftype=video
```

---

### 2.2 جلب قوائم التشغيل (Playlists)

#### 2.2.1 جلب جميع قوائم التشغيل

```http
GET /api/v1/media/playlists
```

#### 2.2.2 جلب قوائم التشغيل حسب تصنيف

```http
GET /api/v1/media/playlists?categories_id={category_id}
```

**مثال**  
```
GET /api/v1/media/playlists?categories_id=5
```

#### 2.2.3 تضمين الفيديوهات التابعة لكل قائمة تشغيل

```http
GET /api/v1/media/playlists?include=videos
```
أو مع التصنيف:
```http
GET /api/v1/media/playlists?categories_id=5&include=videos
```

---

### 2.3 جلب الفيديوهات

```http
GET /api/v1/media/videos
```

#### المعاملات الاختيارية

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `playlists_id` | integer | جلب الفيديوهات التابعة لقائمة تشغيل معينة |
| `categories_id` | integer | فلترة الفيديوهات حسب تصنيف معين |
| `q` | string | نص البحث في الفيديوهات (الاسم، الوصف، إلخ) |

**أمثلة**

```
GET /api/v1/media/videos?playlists_id=8
GET /api/v1/media/videos?categories_id=5
GET /api/v1/media/videos?q=تعليمي
```

#### هيكل سجل الفيديو

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `name` | string | اسم الفيديو |
| `short_description` | text | وصف مختصر (نص عادي) |
| `description` | html | وصف تفصيلي (نص HTML) |
| `image` | object | صورة الغلاف (إن وجدت) |
| `video` | object | ملف الفيديو (المسار، النوع، الحجم، إلخ) |

---

## 3. الملفات (Files)

### 3.1 جلب تصنيفات الملفات

```http
GET /api/v1/media/categories?reftype=file
```

---

### 3.2 جلب حزم الملفات (Filelists)

#### 3.2.1 جلب جميع الحزم

```http
GET /api/v1/media/filelists
```

#### 3.2.2 جلب الحزم حسب تصنيف معين

```http
GET /api/v1/media/filelists?categories_id={category_id}
```

**مثال**  
```
GET /api/v1/media/filelists?categories_id=6
```

#### 3.2.3 تضمين الملفات التابعة لكل حزمة

```http
GET /api/v1/media/filelists?include=files
```
أو مع التصنيف:
```http
GET /api/v1/media/filelists?categories_id=6&include=files
```

---

### 3.3 جلب الملفات

```http
GET /api/v1/media/files
```

#### المعاملات الاختيارية

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `filelists_id` | integer | جلب الملفات التابعة لحزمة معينة |
| `categories_id` | integer | فلترة الملفات حسب تصنيف معين |
| `q` | string | نص البحث في الملفات (الاسم، الوصف، إلخ) |

**أمثلة**

```
GET /api/v1/media/files?filelists_id=10
GET /api/v1/media/files?categories_id=6
GET /api/v1/media/files?q=تقرير
```

#### هيكل سجل الملف

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `name` | string | اسم الملف |
| `short_description` | text | وصف مختصر (نص عادي) |
| `description` | html | وصف تفصيلي (نص HTML) |
| `image` | object | صورة الغلاف (إن وجدت) |
| `file` | object | بيانات الملف ومساره (الرابط، الامتداد، الحجم، إلخ) |

---

## ملاحظات عامة

- جميع الاستدعاءات تعيد البيانات بصيغة JSON.
- معامل `include` يُستخدم لتحميل العلاقات المرتبطة مباشرة (مثل `photos`, `videos`, `files`) ويمنع الحاجة إلى استدعاءات إضافية.
- معامل `q` يتيح البحث النصي في الحقول النصية (الاسم، الوصف المختصر، الوصف التفصيلي).
- توثيق إضافي لرموز الاستجابة والأخطاء غير مذكور هنا، ويفترض التعامل مع الأكواد المعيارية HTTP (200، 400، 404، إلخ).

---

## وثائق إضافية 

- [توثيق مكتبة الوسائط API (MediaApi)](./docs/MediaApi/Docs-MediaApi-ar.md)
- [توثيق مختصر مكتبة الوسائط API (MediaApi)](./docs/MediaApi/Docs-MediaApi-Short-ar.md)
- [أمثلة عملية شاملة – إضافة MediaApi](./docs/MediaApi/Docs-MediaApi-Examples-ar.md)
- [أمثلة عملية مختصرة – إضافة MediaApi](./docs/MediaApi/Docs-MediaApi-Short-Examples-ar.md)
