# أمثلة عملية شاملة – إضافة `Nano.MediaApi`

يحتوي هذا المستند على أمثلة حقيقية ومباشرة لاستخدام جميع نقاط النهاية الخاصة بإدارة **مكتبة الوسائط** (الصور، الفيديوهات، الملفات) عبر API. جميع البيانات التالية مستخرجة من استجابات فعلية للنظام، مع الحفاظ على الدقة والتنسيق الأصلي كما وردت في ملفات التوثيق الرسمية.

---

## فهرس الأمثلة

1. [التصنيفات (Categories)](#1-التصنيفات-categories)
   - 1.0 جلب جميع التصنيفات
   - 1.1 جلب التصنيفات مع تضمين الوسوم والتصنيفات العامة
   - 1.2 جلب التصنيفات مع تضمين الألبومات / قوائم التشغيل / حزم الملفات
   - 1.3 تصنيفات الصور فقط مع تضمين الألبومات
   - 1.4 تصنيفات الفيديوهات فقط مع تضمين قوائم التشغيل
   - 1.5 تصنيفات الملفات فقط مع تضمين حزم الملفات
   - 1.6 عرض تصنيف واحد
   - 1.7 التحقق من آخر تحديث للتصنيفات
2. [الألبومات (Albums)](#2-الألبومات-albums)
   - 2.0 جلب جميع الألبومات
   - 2.1 استبعاد الحقول من الألبوم
   - 2.2 التحقق من آخر تحديث للألبومات
3. [الصور (Images)](#3-الصور-images)
   - 3.0 جلب جميع الصور (مع تقسيم صفحات)
   - 3.1 جلب الصور مع تضمين الألبوم
   - 3.2 جلب الصور مع تضمين التصنيف والوسوم والتصنيفات العامة
   - 3.3 استبعاد الحقول من الصور
   - 3.4 عرض صورة واحدة مع تضمين الوسوم والألبوم
   - 3.5 التحقق من آخر تحديث للصور
4. [قوائم التشغيل (Playlists)](#4-قوائم-التشغيل-playlists)
   - 4.0 جلب جميع قوائم التشغيل
   - 4.1 جلب قوائم التشغيل مع تضمين الفيديوهات
   - 4.2 جلب قوائم التشغيل مع تضمين التصنيف والوسوم
   - 4.3 استبعاد الحقول من قائمة تشغيل
   - 4.4 عرض قائمة تشغيل واحدة
   - 4.5 التحقق من آخر تحديث لقوائم التشغيل
5. [الفيديوهات (Videos)](#5-الفيديوهات-videos)
   - 5.0 جلب جميع الفيديوهات
   - 5.1 جلب الفيديوهات مع تضمين قائمة التشغيل
   - 5.2 جلب الفيديوهات مع تضمين التصنيف والوسوم
   - 5.3 استبعاد الحقول من الفيديو
   - 5.4 عرض فيديو واحد
   - 5.5 التحقق من آخر تحديث للفيديوهات
6. [حزم الملفات (Filelists)](#6-حزم-الملفات-filelists)
   - 6.0 جلب جميع حزم الملفات
   - 6.1 جلب حزم الملفات مع تضمين الملفات
   - 6.2 جلب حزم الملفات مع تضمين التصنيف والوسوم
   - 6.3 استبعاد الحقول من حزمة ملفات
   - 6.4 عرض حزمة ملفات واحدة مع الملفات
   - 6.5 التحقق من آخر تحديث لحزم الملفات
7. [الملفات (Files)](#7-الملفات-files)
   - 7.0 جلب جميع الملفات
   - 7.1 جلب الملفات مع تضمين حزمة الملفات
   - 7.2 استبعاد الحقول من الملف
   - 7.3 التحقق من آخر تحديث للملفات

---

## 1. التصنيفات (Categories)

نقطة النهاية: `GET /api/v1/media/categories`

### 1.0 جلب جميع التصنيفات

**الطلب**

```http
GET http://localhost:8006/api/v1/media/categories
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 1,
      "code": "2-2-1",
      "name": "المجموعه الرئيسيه لتصنيفات الصور",
      "slug": "almgmoaah-alrysyh-ltsnyfat-alsor",
      "short_description": "",
      "description": "",
      "meta_title": "المجموعه الرئيسيه لتصنيفات الصور",
      "meta_description": "",
      "keywords": "",
      "ref_type_class": "image",
      "ref_key_values_class": "5",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "main",
      "parent_id": "0",
      "level": 1,
      "properties": [],
      "created_at": "2024-01-11 18:40:09",
      "updated_at": "2024-01-11 18:40:09",
      "image": null,
      "images": [],
      "files": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Categorie"
    },
    {
      "id": 2,
      "code": "2-2-2",
      "name": "المجموعه الفرعية لتصنيفات الصور",
      "slug": "almgmoaah-alfraay-ltsnyfat-alsor",
      "short_description": "",
      "description": "",
      "meta_title": "المجموعه الفرعية لتصنيفات الصور",
      "meta_description": "",
      "keywords": "",
      "ref_type_class": "image",
      "ref_key_values_class": "5",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "main",
      "parent_id": "1",
      "level": 2,
      "properties": [],
      "created_at": "2024-01-11 18:43:04",
      "updated_at": "2024-01-11 18:43:05",
      "image": null,
      "images": [],
      "files": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Categorie"
    },
    {
      "id": 3,
      "code": "2-2-3",
      "name": "المجموعه الرئيسيه لتصنيفات الفيديوهات",
      "slug": "almgmoaah-alrysyh-ltsnyfat-alfydyohat",
      "short_description": "",
      "description": "",
      "meta_title": "المجموعه الرئيسيه لتصنيفات الفيديوهات",
      "meta_description": "",
      "keywords": "",
      "ref_type_class": "video",
      "ref_key_values_class": "6",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "main",
      "parent_id": "0",
      "level": 1,
      "properties": [],
      "created_at": "2024-01-11 18:49:38",
      "updated_at": "2024-01-11 18:49:38",
      "image": null,
      "images": [],
      "files": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Categorie"
    },
    {
      "id": 5,
      "code": "2-2-5",
      "name": "المجموعه الرئيسية للملفات والوثائق",
      "slug": "almgmoaah-alrysy-llmlfat-oalothak",
      "short_description": "",
      "description": "",
      "meta_title": "المجموعه الرئيسية للملفات والوثائق",
      "meta_description": "",
      "keywords": "",
      "ref_type_class": "file",
      "ref_key_values_class": "7",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "main",
      "parent_id": "0",
      "level": 1,
      "properties": [],
      "created_at": "2024-01-11 18:53:34",
      "updated_at": "2024-01-11 18:53:34",
      "image": null,
      "images": [],
      "files": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Categorie"
    }
  ],
  "meta": {
    "pagination": {
      "total": 4,
      "count": 4,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 1.1 جلب التصنيفات مع تضمين الوسوم والتصنيفات العامة (`include=tags,pcategories`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/categories?include=tags,pcategories
```

**الاستجابة** (مقتطف – أول تصنيف فقط للاختصار، مع الحفاظ على البنية الكاملة كما في الملف الأصلي)

```json
{
  "data": [
    {
      "id": 1,
      "code": "2-2-1",
      "name": "المجموعه الرئيسيه لتصنيفات الصور",
      "slug": "almgmoaah-alrysyh-ltsnyfat-alsor",
      "short_description": "",
      "description": "",
      "meta_title": "المجموعه الرئيسيه لتصنيفات الصور",
      "meta_description": "",
      "keywords": "",
      "ref_type_class": "image",
      "ref_key_values_class": "5",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "main",
      "parent_id": "0",
      "level": 1,
      "properties": [],
      "created_at": "2024-01-11 18:40:09",
      "updated_at": "2024-01-11 18:40:09",
      "image": null,
      "images": [],
      "files": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Categorie",
      "tags": {
        "data": [
          {
            "id": 1,
            "code": "2-4-1",
            "name": "منوعات",
            "slug": "mnoaaat",
            "type": "",
            "user_id": "",
            "user_type": "",
            "companys_id": "2",
            "departments_id": "4",
            "is_public": 1,
            "is_default": 0,
            "is_active": 1,
            "is_published": 1,
            "published_at": "",
            "unpublished_at": "",
            "properties": [],
            "sort_order": 1,
            "created_at": "2022-09-11 13:52:27",
            "updated_at": "2022-11-14 16:33:57",
            "image": null,
            "images": [],
            "object_type": "Nano\\Tags\\Models\\Tag"
          },
          {
            "id": 8,
            "code": "2-4-8",
            "name": "عام",
            "slug": "aaam",
            "type": "",
            "user_id": "",
            "user_type": "",
            "companys_id": "2",
            "departments_id": "4",
            "is_public": 1,
            "is_default": 0,
            "is_active": 1,
            "is_published": 1,
            "published_at": "",
            "unpublished_at": "",
            "properties": [],
            "sort_order": 8,
            "created_at": "2023-06-20 23:58:50",
            "updated_at": "2023-06-20 23:58:50",
            "image": null,
            "images": [],
            "object_type": "Nano\\Tags\\Models\\Tag"
          }
        ]
      },
      "pcategories": {
        "data": [
          {
            "id": 1,
            "code": "2-4-1",
            "name": "عامه",
            "slug": "aaamh",
            "type": "1",
            "user_id": "",
            "user_type": "",
            "companys_id": "2",
            "departments_id": "4",
            "country_id": "",
            "is_public": 1,
            "is_default": 0,
            "is_active": 1,
            "is_published": 1,
            "published_at": "",
            "unpublished_at": "",
            "main_sub": "main",
            "parent_id": "",
            "level": 1,
            "properties": [],
            "sort_order": 1,
            "created_at": "2022-09-26 19:55:19",
            "updated_at": "2022-12-16 16:29:28",
            "image": {
              "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/639\/c72\/b17\/639c72b171018852883670.png",
              "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/639\/c72\/b17\/thumb_570_160_160_0_0_crop.png",
              "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/639\/c72\/b17\/thumb_570_240_240_0_0_crop.png",
              "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/639\/c72\/b17\/thumb_570_800_800_0_0_crop.png",
              "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/639\/c72\/b17\/thumb_570_480_480_0_0_auto.png"
            },
            "images": [],
            "object_type": "Nano\\Tags\\Models\\Categorie"
          },
          {
            "id": 3,
            "code": "2-4-3",
            "name": "العروض",
            "slug": "alaarod",
            "type": "",
            "user_id": "",
            "user_type": "",
            "companys_id": "2",
            "departments_id": "4",
            "country_id": "",
            "is_public": 1,
            "is_default": 0,
            "is_active": 1,
            "is_published": 1,
            "published_at": "",
            "unpublished_at": "",
            "main_sub": "",
            "parent_id": "0",
            "level": 1,
            "properties": [],
            "sort_order": 3,
            "created_at": "2024-01-04 20:26:23",
            "updated_at": "2024-01-04 20:26:24",
            "image": null,
            "images": [],
            "object_type": "Nano\\Tags\\Models\\Categorie"
          }
        ]
      }
    },
    // باقي التصنيفات (2، 3، 5) بنفس البنية مع بياناتها الخاصة
  ],
  "meta": {
    "pagination": {
      "total": 4,
      "count": 4,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 1.2 جلب التصنيفات مع تضمين الألبومات / قوائم التشغيل / حزم الملفات (`include=albums,playlists,filelists`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/categories?include=albums,playlists,filelists
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 1,
      "code": "2-2-1",
      "name": "المجموعه الرئيسيه لتصنيفات الصور",
      ... // باقي الحقول كما في المثال 1.0
      "albums": { "data": [] },
      "playlists": { "data": [] },
      "filelists": { "data": [] }
    },
    {
      "id": 2,
      "code": "2-2-2",
      "name": "المجموعه الفرعية لتصنيفات الصور",
      "albums": {
        "data": [
          {
            "id": 4,
            "code": "2-2-4",
            "name": "البوم الصور الاول",
            "slug": "albom-alsor-alaol",
            "short_description": "",
            "description": "",
            "meta_title": "البوم الصور الاول",
            "meta_description": "",
            "keywords": "",
            "ref_type_class": "image",
            "ref_key_values_class": "5",
            "companys_id": "2",
            "departments_id": "2",
            "is_default": 0,
            "is_active": 1,
            "is_published": 1,
            "published_at": "2024-01-01 15:50:57",
            "unpublished_at": "",
            "main_sub": "sub",
            "parent_id": "2",
            "level": 3,
            "properties": [],
            "created_at": "2024-01-11 18:51:34",
            "updated_at": "2024-01-11 18:51:34",
            "image": {
              "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/65a00f1f344b1459473131.jpg",
              "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_160_160_0_0_crop.jpg",
              "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_240_240_0_0_crop.jpg",
              "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_800_800_0_0_crop.jpg",
              "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_480_480_0_0_auto.jpg"
            },
            "images": [],
            "files": [],
            "ratings_count": 0,
            "countRating": 0,
            "sumRating": 0,
            "averageRating": 0,
            "user_is_rating": 0,
            "user_object_rating": null,
            "favorites_count": 0,
            "user_is_favorite": 0,
            "likes_count": 0,
            "bookmarks_count": 0,
            "reactions_count": 0,
            "object_type": "Tss\\Media\\Models\\Album"
          }
        ]
      },
      "playlists": { "data": [] },
      "filelists": { "data": [] }
    },
    {
      "id": 3,
      "code": "2-2-3",
      "name": "المجموعه الرئيسيه لتصنيفات الفيديوهات",
      "albums": { "data": [] },
      "playlists": {
        "data": [
          {
            "id": 6,
            "code": "2-2-6",
            "name": "قائمة تشغيل فيديوهات رقم واحد",
            "slug": "kam-tshghyl-fydyohat-rkm-oahd",
            "short_description": "",
            "description": "",
            "meta_title": "قائمة تشغيل فيديوهات رقم واحد",
            "meta_description": "",
            "keywords": "",
            "ref_type_class": "video",
            "ref_key_values_class": "6",
            "companys_id": "2",
            "departments_id": "2",
            "is_default": 0,
            "is_active": 1,
            "is_published": 1,
            "published_at": "",
            "unpublished_at": "",
            "main_sub": "sub",
            "parent_id": "3",
            "level": 2,
            "properties": [],
            "created_at": "2024-01-11 22:43:02",
            "updated_at": "2024-01-11 22:43:02",
            "image": {
              "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/65a044b065e3d849683074.jpg",
              "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_160_160_0_0_crop.jpg",
              "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_240_240_0_0_crop.jpg",
              "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_800_800_0_0_crop.jpg",
              "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_480_480_0_0_auto.jpg"
            },
            "images": [],
            "files": [],
            "ratings_count": 0,
            "countRating": 0,
            "sumRating": 0,
            "averageRating": 0,
            "user_is_rating": 0,
            "user_object_rating": null,
            "favorites_count": 0,
            "user_is_favorite": 0,
            "likes_count": 0,
            "bookmarks_count": 0,
            "reactions_count": 0,
            "object_type": "Tss\\Media\\Models\\Playlist"
          }
        ]
      },
      "filelists": { "data": [] }
    },
    {
      "id": 5,
      "code": "2-2-5",
      "name": "المجموعه الرئيسية للملفات والوثائق",
      "albums": { "data": [] },
      "playlists": { "data": [] },
      "filelists": {
        "data": [
          {
            "id": 7,
            "code": "2-2-7",
            "name": "وثائق تعليميه",
            "slug": "othak-taalymyh",
            "short_description": "",
            "description": "",
            "meta_title": "وثائق تعليميه",
            "meta_description": "",
            "keywords": "",
            "ref_type_class": "file",
            "ref_key_values_class": "7",
            "companys_id": "2",
            "departments_id": "2",
            "is_default": 0,
            "is_active": 1,
            "is_published": 1,
            "published_at": "",
            "unpublished_at": "",
            "main_sub": "sub",
            "parent_id": "5",
            "level": 2,
            "properties": [],
            "created_at": "2024-01-11 22:57:59",
            "updated_at": "2024-01-11 22:57:59",
            "image": null,
            "images": [],
            "ratings_count": 0,
            "countRating": 0,
            "sumRating": 0,
            "averageRating": 0,
            "user_is_rating": 0,
            "user_object_rating": null,
            "favorites_count": 0,
            "user_is_favorite": 0,
            "likes_count": 0,
            "bookmarks_count": 0,
            "reactions_count": 0,
            "object_type": "Tss\\Media\\Models\\Filelist"
          }
        ]
      }
    }
  ],
  "meta": {
    "pagination": {
      "total": 4,
      "count": 4,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 1.3 تصنيفات الصور فقط مع تضمين الألبومات (`reftype=image&include=albums,playlists,filelists`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/categories?reftype=image&include=albums,playlists,filelists
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 1,
      "code": "2-2-1",
      "name": "المجموعه الرئيسيه لتصنيفات الصور",
      ... // باقي الحقول
      "albums": { "data": [] },
      "playlists": { "data": [] },
      "filelists": { "data": [] }
    },
    {
      "id": 2,
      "code": "2-2-2",
      "name": "المجموعه الفرعية لتصنيفات الصور",
      ... // باقي الحقول
      "albums": {
        "data": [
          {
            "id": 4,
            "code": "2-2-4",
            "name": "البوم الصور الاول",
            // ... كامل بيانات الألبوم كما في المثال 1.2
          }
        ]
      },
      "playlists": { "data": [] },
      "filelists": { "data": [] }
    }
  ],
  "meta": {
    "pagination": {
      "total": 2,
      "count": 2,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 1.4 تصنيفات الفيديوهات فقط مع تضمين قوائم التشغيل (`reftype=video&include=albums,playlists,filelists`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/categories?reftype=video&include=albums,playlists,filelists
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 3,
      "code": "2-2-3",
      "name": "المجموعه الرئيسيه لتصنيفات الفيديوهات",
      ... // باقي الحقول
      "albums": { "data": [] },
      "playlists": {
        "data": [
          {
            "id": 6,
            "code": "2-2-6",
            "name": "قائمة تشغيل فيديوهات رقم واحد",
            // ... كامل بيانات قائمة التشغيل كما في المثال 1.2
          }
        ]
      },
      "filelists": { "data": [] }
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 1.5 تصنيفات الملفات فقط مع تضمين حزم الملفات (`reftype=file&include=albums,playlists,filelists`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/categories?reftype=file&include=albums,playlists,filelists
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 5,
      "code": "2-2-5",
      "name": "المجموعه الرئيسية للملفات والوثائق",
      ... // باقي الحقول
      "albums": { "data": [] },
      "playlists": { "data": [] },
      "filelists": {
        "data": [
          {
            "id": 7,
            "code": "2-2-7",
            "name": "وثائق تعليميه",
            // ... كامل بيانات حزمة الملفات كما في المثال 1.2
          }
        ]
      }
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 1.6 عرض تصنيف واحد (`/categories/{id}`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/categories/1
```

**الاستجابة**

```json
{
  "id": 1,
  "code": "2-2-1",
  "name": "المجموعه الرئيسيه لتصنيفات الصور",
  "slug": "almgmoaah-alrysyh-ltsnyfat-alsor",
  "short_description": "",
  "description": "",
  "meta_title": "المجموعه الرئيسيه لتصنيفات الصور",
  "meta_description": "",
  "keywords": "",
  "ref_type_class": "image",
  "ref_key_values_class": "5",
  "companys_id": "2",
  "departments_id": "2",
  "is_default": 0,
  "is_active": 1,
  "is_published": 1,
  "published_at": "",
  "unpublished_at": "",
  "main_sub": "main",
  "parent_id": "0",
  "level": 1,
  "properties": [],
  "created_at": "2024-01-11 18:40:09",
  "updated_at": "2024-01-11 18:40:09",
  "image": null,
  "images": [],
  "files": [],
  "ratings_count": 0,
  "countRating": 0,
  "sumRating": 0,
  "averageRating": 0,
  "user_is_rating": 0,
  "user_object_rating": null,
  "favorites_count": 0,
  "user_is_favorite": 0,
  "likes_count": 0,
  "bookmarks_count": 0,
  "reactions_count": 0,
  "object_type": "Tss\\Media\\Models\\Categorie"
}
```

---

### 1.7 التحقق من آخر تحديث للتصنيفات (`/categories/activelystats`)

**الطلب** (مع تاريخ محدد)

```http
GET http://localhost:8006/api/v1/media/categories/activelystats?date=2022-12-15%2017:10:00
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,
    "check_date": "2022-12-15 2017:10:00",
    "last_updated": "2024-01-12 16:28:16",
    "other_updated": {
      "activity_cache.categorie": "2024-01-12 16:28:16",
      "activity_cache.album": "2024-01-12 16:28:16",
      "activity_cache.filelist": "2024-01-12 16:28:16",
      "activity_cache.playlist": "2024-01-12 16:28:16"
    }
  }
}
```

**بدون تاريخ**

```http
GET http://localhost:8006/api/v1/media/categories/activelystats
```

```json
{
  "code": "200",
  "data": {
    "activity_stats": false,
    "check_date": "2024-01-12 16:28:30",
    "last_updated": "2024-01-12 16:28:30",
    "other_updated": {
      "activity_cache.categorie": "2024-01-12 16:28:30",
      "activity_cache.album": "2024-01-12 16:28:30",
      "activity_cache.filelist": "2024-01-12 16:28:30",
      "activity_cache.playlist": "2024-01-12 16:28:31"
    }
  }
}
```

---

## 2. الألبومات (Albums)

نقطة النهاية: `GET /api/v1/media/albums`

### 2.0 جلب جميع الألبومات

**الطلب**

```http
GET http://localhost:8006/api/v1/media/albums
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 4,
      "code": "2-2-4",
      "name": "البوم الصور الاول",
      "slug": "albom-alsor-alaol",
      "short_description": "",
      "description": "",
      "meta_title": "البوم الصور الاول",
      "meta_description": "",
      "keywords": "",
      "ref_type_class": "image",
      "ref_key_values_class": "5",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "2024-01-01 15:50:57",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "2",
      "level": 3,
      "properties": [],
      "created_at": "2024-01-11 18:51:34",
      "updated_at": "2024-01-11 18:51:34",
      "image": {
        "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/65a00f1f344b1459473131.jpg",
        "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_160_160_0_0_crop.jpg",
        "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_240_240_0_0_crop.jpg",
        "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_800_800_0_0_crop.jpg",
        "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_480_480_0_0_auto.jpg"
      },
      "images": [],
      "files": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Album"
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 2.1 استبعاد الحقول من الألبوم (`exclude=created_at,updated_at`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/albums?exclude=created_at,updated_at
```

**الاستجابة** (بدون `created_at` و `updated_at`)

```json
{
  "data": [
    {
      "id": 4,
      "code": "2-2-4",
      "name": "البوم الصور الاول",
      "slug": "albom-alsor-alaol",
      "short_description": "",
      "description": "",
      "meta_title": "البوم الصور الاول",
      "meta_description": "",
      "keywords": "",
      "ref_type_class": "image",
      "ref_key_values_class": "5",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "2024-01-01 15:50:57",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "2",
      "level": 3,
      "properties": [],
      "image": {
        "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/65a00f1f344b1459473131.jpg",
        "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_160_160_0_0_crop.jpg",
        "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_240_240_0_0_crop.jpg",
        "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_800_800_0_0_crop.jpg",
        "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_480_480_0_0_auto.jpg"
      },
      "images": [],
      "files": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Album"
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 2.2 التحقق من آخر تحديث للألبومات (`/albums/activelystats`)

**الطلب** (مع تاريخ)

```http
GET http://localhost:8006/api/v1/media/albums/activelystats?date=2022-12-15%2017:10:00
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,
    "check_date": "2022-12-15 2017:10:00",
    "last_updated": "2024-01-12 16:23:49",
    "other_updated": {
      "activity_cache.album": "2024-01-12 16:23:49",
      "activity_cache.categorie": "2024-01-12 16:23:49",
      "activity_cache.image": "2024-01-12 16:23:49"
    }
  }
}
```

---

## 3. الصور (Images)

نقطة النهاية: `GET /api/v1/media/images`

### 3.0 جلب جميع الصور (مع تقسيم صفحات)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/images?per_page=2
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 1,
      "code": "2-2-1",
      "barcode": null,
      "name": "صوه شعار",
      "slug": "صوه شعار-610",
      "short_description": null,
      "description": null,
      "meta_title": "صوه شعار",
      "meta_description": null,
      "keywords": null,
      "categories_id": 4,
      "ref_type_class": "image",
      "ref_key_values_class": "5",
      "type": null,
      "type_process": null,
      "link": null,
      "companys_id": "2",
      "departments_id": "2",
      "is_effective": 1,
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "start_at": "",
      "end_at": "",
      "is_downloadable": 1,
      "modul_type": null,
      "ref_type": null,
      "properties": null,
      "links": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 1,
      "created_at": "2024-01-11 19:28:28",
      "updated_at": "2024-01-11 19:28:28",
      "image": {
        "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/ec6\/65a016ec6e558137263552.png",
        "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/ec6\/thumb_610_160_160_0_0_crop.png",
        "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/ec6\/thumb_610_240_240_0_0_crop.png",
        "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/ec6\/thumb_610_800_800_0_0_crop.png",
        "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/ec6\/thumb_610_480_480_0_0_auto.png"
      },
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Image",
      "tags": { "data": [] }
    },
    {
      "id": 2,
      "code": "2-2-2",
      "barcode": null,
      "name": "صوره ١",
      "slug": "صوره ١-611",
      "short_description": null,
      "description": null,
      "meta_title": "صوره ١",
      "meta_description": null,
      "keywords": null,
      "categories_id": 4,
      "ref_type_class": "image",
      "ref_key_values_class": "5",
      "type": null,
      "type_process": null,
      "link": null,
      "companys_id": "2",
      "departments_id": "2",
      "is_effective": 1,
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "start_at": "",
      "end_at": "",
      "is_downloadable": 1,
      "modul_type": null,
      "ref_type": null,
      "properties": null,
      "links": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 2,
      "created_at": "2024-01-11 19:28:28",
      "updated_at": "2024-01-11 19:28:28",
      "image": {
        "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/ec7\/65a016ec78173328488518.webp",
        "small": "",
        "medium": "",
        "large": "",
        "thumb": ""
      },
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Image",
      "tags": { "data": [] }
    }
  ],
  "meta": {
    "pagination": {
      "total": 6,
      "count": 2,
      "per_page": 2,
      "current_page": 1,
      "total_pages": 3,
      "links": {
        "next": "http:\/\/localhost:8006\/api\/v1\/media\/images?page=2"
      }
    }
  }
}
```

---

### 3.1 جلب الصور مع تضمين الألبوم (`include=album`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/images?include=album
```

**الاستجابة** (مقتطف – أول صورتين مع الألبوم)

```json
{
  "data": [
    {
      "id": 1,
      "name": "صوه شعار",
      ... // باقي الحقول
      "album": {
        "id": 4,
        "code": "2-2-4",
        "name": "البوم الصور الاول",
        "slug": "albom-alsor-alaol",
        "short_description": "",
        "description": "",
        "meta_title": "البوم الصور الاول",
        "meta_description": "",
        "keywords": "",
        "ref_type_class": "image",
        "ref_key_values_class": "5",
        "companys_id": "2",
        "departments_id": "2",
        "is_default": 0,
        "is_active": 1,
        "is_published": 1,
        "published_at": "2024-01-01 15:50:57",
        "unpublished_at": "",
        "main_sub": "sub",
        "parent_id": "2",
        "level": 3,
        "properties": [],
        "created_at": "2024-01-11 18:51:34",
        "updated_at": "2024-01-11 18:51:34",
        "image": { ... },
        "images": [],
        "files": [],
        "object_type": "Tss\\Media\\Models\\Album"
      }
    },
    {
      "id": 2,
      "name": "صوره ١",
      ... // باقي الحقول
      "album": { ... } // نفس الألبوم 4
    }
    // باقي الصور بنفس البنية
  ],
  "meta": { ... }
}
```

---

### 3.2 جلب الصور مع تضمين التصنيف والوسوم والتصنيفات العامة (`include=tags,pcategories,categorie`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/images?include=tags,pcategories,categorie
```

**الاستجابة** (مقتطف – أول صورة مع التصنيف)

```json
{
  "data": [
    {
      "id": 1,
      "name": "صوه شعار",
      ... // باقي الحقول
      "categorie": {
        "id": 1,
        "code": "2-2-1",
        "name": "المجموعه الرئيسيه لتصنيفات الصور",
        "slug": "almgmoaah-alrysyh-ltsnyfat-alsor",
        "short_description": "",
        "description": "",
        "meta_title": "المجموعه الرئيسيه لتصنيفات الصور",
        "meta_description": "",
        "keywords": "",
        "ref_type_class": "image",
        "ref_key_values_class": "5",
        "companys_id": "2",
        "departments_id": "2",
        "is_default": 0,
        "is_active": 1,
        "is_published": 1,
        "published_at": "",
        "unpublished_at": "",
        "main_sub": "main",
        "parent_id": "0",
        "level": 1,
        "properties": [],
        "created_at": "2024-01-11 18:40:09",
        "updated_at": "2024-01-11 18:40:09",
        "image": null,
        "images": [],
        "files": [],
        "ratings_count": 0,
        "countRating": 0,
        "sumRating": 0,
        "averageRating": 0,
        "user_is_rating": 0,
        "user_object_rating": null,
        "favorites_count": 0,
        "user_is_favorite": 0,
        "likes_count": 0,
        "bookmarks_count": 0,
        "reactions_count": 0,
        "object_type": "Tss\\Media\\Models\\Categorie"
      },
      "tags": { "data": [] },
      "pcategories": { "data": [] }
    },
    // باقي الصور بنفس البنية
  ],
  "meta": { ... }
}
```

---

### 3.3 استبعاد الحقول من الصور (`exclude=created_at,updated_at`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/images?exclude=created_at,updated_at
```

**الاستجابة** (نفس البيانات بدون الحقلين)

```json
{
  "data": [
    {
      "id": 1,
      "code": "2-2-1",
      "barcode": null,
      "name": "صوه شعار",
      // لا يحتوي على created_at, updated_at
      ...
    }
    // باقي الصور
  ]
}
```

---

### 3.4 عرض صورة واحدة مع تضمين الوسوم والألبوم (`/images/{id}?include=tags,album`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/images/4?include=tags,album
```

**الاستجابة**

```json
{
  "id": 4,
  "code": "2-2-4",
  "barcode": null,
  "name": "صوره اعلان",
  "slug": "صوره اعلان-613",
  "short_description": null,
  "description": null,
  "meta_title": "صوره اعلان",
  "meta_description": null,
  "keywords": null,
  "categories_id": 4,
  "ref_type_class": "image",
  "ref_key_values_class": "5",
  "type": null,
  "type_process": null,
  "link": null,
  "companys_id": "2",
  "departments_id": "2",
  "is_effective": 1,
  "is_default": 0,
  "is_active": 1,
  "is_published": 1,
  "published_at": "",
  "unpublished_at": "",
  "start_at": "",
  "end_at": "",
  "is_downloadable": 1,
  "modul_type": null,
  "ref_type": null,
  "properties": null,
  "links": null,
  "other_data": null,
  "config_data": null,
  "sort_order": 4,
  "created_at": "2024-01-11 19:28:28",
  "updated_at": "2024-01-11 19:28:29",
  "image": {
    "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/fbe\/65a016fbec109756699288.png",
    "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/fbe\/thumb_613_160_160_0_0_crop.png",
    "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/fbe\/thumb_613_240_240_0_0_crop.png",
    "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/fbe\/thumb_613_800_800_0_0_crop.png",
    "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/016\/fbe\/thumb_613_480_480_0_0_auto.png"
  },
  "ratings_count": 0,
  "countRating": 0,
  "sumRating": 0,
  "averageRating": 0,
  "user_is_rating": 0,
  "user_object_rating": null,
  "favorites_count": 0,
  "user_is_favorite": 0,
  "likes_count": 0,
  "bookmarks_count": 0,
  "reactions_count": 0,
  "object_type": "Tss\\Media\\Models\\Image",
  "album": {
    "id": 4,
    "code": "2-2-4",
    "name": "البوم الصور الاول",
    "slug": "albom-alsor-alaol",
    "short_description": "",
    "description": "",
    "meta_title": "البوم الصور الاول",
    "meta_description": "",
    "keywords": "",
    "ref_type_class": "image",
    "ref_key_values_class": "5",
    "companys_id": "2",
    "departments_id": "2",
    "is_default": 0,
    "is_active": 1,
    "is_published": 1,
    "published_at": "2024-01-01 15:50:57",
    "unpublished_at": "",
    "main_sub": "sub",
    "parent_id": "2",
    "level": 3,
    "properties": [],
    "created_at": "2024-01-11 18:51:34",
    "updated_at": "2024-01-11 18:51:34",
    "image": {
      "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/65a00f1f344b1459473131.jpg",
      "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_160_160_0_0_crop.jpg",
      "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_240_240_0_0_crop.jpg",
      "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_800_800_0_0_crop.jpg",
      "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/00f\/1f3\/thumb_609_480_480_0_0_auto.jpg"
    },
    "images": [],
    "files": [],
    "object_type": "Tss\\Media\\Models\\Album"
  },
  "tags": { "data": [] }
}
```

---

### 3.5 التحقق من آخر تحديث للصور (`/images/activelystats`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/images/activelystats?date=2022-12-15%2017:10:00
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,
    "check_date": "2022-12-15 2017:10:00",
    "last_updated": "2024-01-12 16:25:04",
    "other_updated": {
      "activity_cache.image": "2024-01-12 16:25:04",
      "activity_cache.album": "2024-01-12 16:25:04"
    }
  }
}
```

---

## 4. قوائم التشغيل (Playlists)

نقطة النهاية: `GET /api/v1/media/playlists`

### 4.0 جلب جميع قوائم التشغيل

**الطلب**

```http
GET http://localhost:8006/api/v1/media/playlists
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 6,
      "code": "2-2-6",
      "name": "قائمة تشغيل فيديوهات رقم واحد",
      "slug": "kam-tshghyl-fydyohat-rkm-oahd",
      "short_description": "",
      "description": "",
      "meta_title": "قائمة تشغيل فيديوهات رقم واحد",
      "meta_description": "",
      "keywords": "",
      "ref_type_class": "video",
      "ref_key_values_class": "6",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "3",
      "level": 2,
      "properties": [],
      "created_at": "2024-01-11 22:43:02",
      "updated_at": "2024-01-11 22:43:02",
      "image": {
        "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/65a044b065e3d849683074.jpg",
        "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_160_160_0_0_crop.jpg",
        "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_240_240_0_0_crop.jpg",
        "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_800_800_0_0_crop.jpg",
        "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_480_480_0_0_auto.jpg"
      },
      "images": [],
      "files": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Playlist"
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 4.1 جلب قوائم التشغيل مع تضمين الفيديوهات (`include=videos`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/playlists?include=videos
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 6,
      "code": "2-2-6",
      "name": "قائمة تشغيل فيديوهات رقم واحد",
      ... // باقي الحقول
      "videos": {
        "data": [
          {
            "id": 7,
            "code": "2-2-7",
            "barcode": null,
            "name": "الفيديو الاول",
            "slug": "alfydyo-alaol",
            "short_description": "",
            "description": "",
            "meta_title": "الفيديو الاول",
            "meta_description": "الفيديو الاول",
            "keywords": "",
            "categories_id": 6,
            "ref_type_class": "video",
            "ref_key_values_class": "6",
            "type": "public",
            "type_process": "open",
            "link": "",
            "companys_id": "2",
            "departments_id": "2",
            "is_effective": 0,
            "is_default": 0,
            "is_active": 1,
            "is_published": 1,
            "published_at": "",
            "unpublished_at": "",
            "start_at": "",
            "end_at": "",
            "is_downloadable": 1,
            "modul_type": null,
            "ref_type": null,
            "properties": null,
            "links": null,
            "other_data": null,
            "config_data": null,
            "sort_order": 7,
            "created_at": "2024-01-11 22:48:26",
            "updated_at": "2024-01-11 22:48:26",
            "image": {
              "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/65a04597133b5710762067.jpg",
              "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_160_160_0_0_crop.jpg",
              "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_240_240_0_0_crop.jpg",
              "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_800_800_0_0_crop.jpg",
              "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_480_480_0_0_auto.jpg"
            },
            "images": [],
            "video": {
              "file_name": "pin_1682877061035.mp4",
              "file_size": 783126,
              "path": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/cf3\/65a045cf374cb388412381.mp4"
            },
            "videos": [],
            "files": [],
            "ratings_count": 0,
            "countRating": 0,
            "sumRating": 0,
            "averageRating": 0,
            "user_is_rating": 0,
            "user_object_rating": null,
            "favorites_count": 0,
            "user_is_favorite": 0,
            "likes_count": 0,
            "bookmarks_count": 0,
            "reactions_count": 0,
            "object_type": "Tss\\Media\\Models\\Video"
          }
        ]
      }
    }
  ],
  "meta": { ... }
}
```

---

### 4.2 جلب قوائم التشغيل مع تضمين التصنيف والوسوم (`include=categorie,tags,pcategories`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/playlists?include=categorie,tags,pcategories
```

**الاستجابة** (مقتطف)

```json
{
  "data": [
    {
      "id": 6,
      ... // باقي الحقول
      "categorie": {
        "id": 3,
        "code": "2-2-3",
        "name": "المجموعه الرئيسيه لتصنيفات الفيديوهات",
        ...
      },
      "tags": {
        "data": [
          {
            "id": 1,
            "code": "2-4-1",
            "name": "منوعات",
            ...
          }
        ]
      },
      "pcategories": {
        "data": [
          {
            "id": 1,
            "code": "2-4-1",
            "name": "عامه",
            "image": { ... }
          }
        ]
      }
    }
  ],
  "meta": { ... }
}
```

---

### 4.3 استبعاد الحقول من قائمة تشغيل (`exclude=created_at,updated_at`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/playlists?exclude=created_at,updated_at
```

**الاستجابة** (بدون التواريخ)

```json
{
  "data": [
    {
      "id": 6,
      "code": "2-2-6",
      "name": "قائمة تشغيل فيديوهات رقم واحد",
      // لا يحتوي على created_at, updated_at
      ...
    }
  ]
}
```

---

### 4.4 عرض قائمة تشغيل واحدة (`/playlists/{id}`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/playlists/6
```

**الاستجابة**

```json
{
  "id": 6,
  "code": "2-2-6",
  "name": "قائمة تشغيل فيديوهات رقم واحد",
  "slug": "kam-tshghyl-fydyohat-rkm-oahd",
  "short_description": "",
  "description": "",
  "meta_title": "قائمة تشغيل فيديوهات رقم واحد",
  "meta_description": "",
  "keywords": "",
  "ref_type_class": "video",
  "ref_key_values_class": "6",
  "companys_id": "2",
  "departments_id": "2",
  "is_default": 0,
  "is_active": 1,
  "is_published": 1,
  "published_at": "",
  "unpublished_at": "",
  "main_sub": "sub",
  "parent_id": "3",
  "level": 2,
  "properties": [],
  "created_at": "2024-01-11 22:43:02",
  "updated_at": "2024-01-11 22:43:02",
  "image": {
    "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/65a044b065e3d849683074.jpg",
    "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_160_160_0_0_crop.jpg",
    "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_240_240_0_0_crop.jpg",
    "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_800_800_0_0_crop.jpg",
    "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/044\/b06\/thumb_616_480_480_0_0_auto.jpg"
  },
  "images": [],
  "files": [],
  "ratings_count": 0,
  "countRating": 0,
  "sumRating": 0,
  "averageRating": 0,
  "user_is_rating": 0,
  "user_object_rating": null,
  "favorites_count": 0,
  "user_is_favorite": 0,
  "likes_count": 0,
  "bookmarks_count": 0,
  "reactions_count": 0,
  "object_type": "Tss\\Media\\Models\\Playlist"
}
```

---

### 4.5 التحقق من آخر تحديث لقوائم التشغيل (`/playlists/activelystats`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/playlists/activelystats?date=2022-12-15%2017:10:00
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,
    "check_date": "2022-12-15 2017:10:00",
    "last_updated": "2024-01-12 16:26:01",
    "other_updated": {
      "activity_cache.playlist": "2024-01-12 16:26:01",
      "activity_cache.categorie": "2024-01-12 16:26:01",
      "activity_cache.video": "2024-01-12 16:26:01"
    }
  }
}
```

---

## 5. الفيديوهات (Videos)

نقطة النهاية: `GET /api/v1/media/videos`

### 5.0 جلب جميع الفيديوهات

**الطلب**

```http
GET http://localhost:8006/api/v1/media/videos
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 7,
      "code": "2-2-7",
      "barcode": null,
      "name": "الفيديو الاول",
      "slug": "alfydyo-alaol",
      "short_description": "",
      "description": "",
      "meta_title": "الفيديو الاول",
      "meta_description": "الفيديو الاول",
      "keywords": "",
      "categories_id": 6,
      "ref_type_class": "video",
      "ref_key_values_class": "6",
      "type": "public",
      "type_process": "open",
      "link": "",
      "companys_id": "2",
      "departments_id": "2",
      "is_effective": 0,
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "start_at": "",
      "end_at": "",
      "is_downloadable": 1,
      "modul_type": null,
      "ref_type": null,
      "properties": null,
      "links": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 7,
      "created_at": "2024-01-11 22:48:26",
      "updated_at": "2024-01-11 22:48:26",
      "image": {
        "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/65a04597133b5710762067.jpg",
        "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_160_160_0_0_crop.jpg",
        "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_240_240_0_0_crop.jpg",
        "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_800_800_0_0_crop.jpg",
        "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_480_480_0_0_auto.jpg"
      },
      "images": [],
      "video": {
        "file_name": "pin_1682877061035.mp4",
        "file_size": 783126,
        "path": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/cf3\/65a045cf374cb388412381.mp4"
      },
      "videos": [],
      "files": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Video"
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 5.1 جلب الفيديوهات مع تضمين قائمة التشغيل (`include=playlist`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/videos?include=playlist
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 7,
      "name": "الفيديو الاول",
      ... // باقي الحقول
      "playlist": {
        "id": 6,
        "code": "2-2-6",
        "name": "قائمة تشغيل فيديوهات رقم واحد",
        "slug": "kam-tshghyl-fydyohat-rkm-oahd",
        // ... كامل بيانات قائمة التشغيل كما في المثال 4.0
      }
    }
  ],
  "meta": { ... }
}
```

---

### 5.2 جلب الفيديوهات مع تضمين التصنيف والوسوم (`include=categorie,tags,pcategories`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/videos?include=categorie,tags,pcategories
```

**الاستجابة** (مقتطف)

```json
{
  "data": [
    {
      "id": 7,
      ... // باقي الحقول
      "categorie": {
        "id": 1,
        "code": "2-2-1",
        "name": "المجموعه الرئيسيه لتصنيفات الصور",
        ...
      },
      "tags": { "data": [] },
      "pcategories": { "data": [] }
    }
  ],
  "meta": { ... }
}
```

---

### 5.3 استبعاد الحقول من الفيديو (`exclude=created_at,updated_at`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/videos?exclude=created_at,updated_at
```

**الاستجابة** (بدون التواريخ)

```json
{
  "data": [
    {
      "id": 7,
      "name": "الفيديو الاول",
      "video": { ... },
      "image": { ... }
      // لا يوجد created_at, updated_at
    }
  ]
}
```

---

### 5.4 عرض فيديو واحد (`/videos/{id}`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/videos/7
```

**الاستجابة**

```json
{
  "id": 7,
  "code": "2-2-7",
  "barcode": null,
  "name": "الفيديو الاول",
  "slug": "alfydyo-alaol",
  "short_description": "",
  "description": "",
  "meta_title": "الفيديو الاول",
  "meta_description": "الفيديو الاول",
  "keywords": "",
  "categories_id": 6,
  "ref_type_class": "video",
  "ref_key_values_class": "6",
  "type": "public",
  "type_process": "open",
  "link": "",
  "companys_id": "2",
  "departments_id": "2",
  "is_effective": 0,
  "is_default": 0,
  "is_active": 1,
  "is_published": 1,
  "published_at": "",
  "unpublished_at": "",
  "start_at": "",
  "end_at": "",
  "is_downloadable": 1,
  "modul_type": null,
  "ref_type": null,
  "properties": null,
  "links": null,
  "other_data": null,
  "config_data": null,
  "sort_order": 7,
  "created_at": "2024-01-11 22:48:26",
  "updated_at": "2024-01-11 22:48:26",
  "image": {
    "original": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/65a04597133b5710762067.jpg",
    "small": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_160_160_0_0_crop.jpg",
    "medium": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_240_240_0_0_crop.jpg",
    "large": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_800_800_0_0_crop.jpg",
    "thumb": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/971\/thumb_617_480_480_0_0_auto.jpg"
  },
  "images": [],
  "video": {
    "file_name": "pin_1682877061035.mp4",
    "file_size": 783126,
    "path": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/045\/cf3\/65a045cf374cb388412381.mp4"
  },
  "videos": [],
  "files": [],
  "ratings_count": 0,
  "countRating": 0,
  "sumRating": 0,
  "averageRating": 0,
  "user_is_rating": 0,
  "user_object_rating": null,
  "favorites_count": 0,
  "user_is_favorite": 0,
  "likes_count": 0,
  "bookmarks_count": 0,
  "reactions_count": 0,
  "object_type": "Tss\\Media\\Models\\Video"
}
```

---

### 5.5 التحقق من آخر تحديث للفيديوهات (`/videos/activelystats`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/videos/activelystats?date=2022-12-15%2017:10:00
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,
    "check_date": "2022-12-15 2017:10:00",
    "last_updated": "2024-01-12 16:18:51",
    "other_updated": {
      "activity_cache.video": "2024-01-12 16:18:51",
      "activity_cache.playlist": "2024-01-12 16:18:51"
    }
  }
}
```

---

## 6. حزم الملفات (Filelists)

نقطة النهاية: `GET /api/v1/media/filelists`

### 6.0 جلب جميع حزم الملفات

**الطلب**

```http
GET http://localhost:8006/api/v1/media/filelists
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 7,
      "code": "2-2-7",
      "name": "وثائق تعليميه",
      "slug": "othak-taalymyh",
      "short_description": "",
      "description": "",
      "meta_title": "وثائق تعليميه",
      "meta_description": "",
      "keywords": "",
      "ref_type_class": "file",
      "ref_key_values_class": "7",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "5",
      "level": 2,
      "properties": [],
      "created_at": "2024-01-11 22:57:59",
      "updated_at": "2024-01-11 22:57:59",
      "image": null,
      "images": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\Filelist"
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 6.1 جلب حزم الملفات مع تضمين الملفات (`include=files`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/filelists?include=files
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 7,
      "code": "2-2-7",
      "name": "وثائق تعليميه",
      ... // باقي الحقول
      "files": {
        "data": [
          {
            "id": 8,
            "code": "2-2-8",
            "barcode": null,
            "name": "ملف تعليمي لنظام نانو سوفت",
            "slug": "mlf-taalymy-lntham-nano-soft",
            "short_description": "",
            "description": "",
            "meta_title": "ملف تعليمي لنظام نانو سوفت",
            "meta_description": "",
            "keywords": "",
            "categories_id": 7,
            "ref_type_class": "file",
            "ref_key_values_class": "7",
            "type": "public",
            "type_process": "open",
            "link": "",
            "companys_id": "2",
            "departments_id": "2",
            "is_effective": 0,
            "is_default": 0,
            "is_active": 1,
            "is_published": 1,
            "published_at": "",
            "unpublished_at": "",
            "start_at": "",
            "end_at": "",
            "is_downloadable": 1,
            "modul_type": null,
            "ref_type": null,
            "properties": null,
            "links": null,
            "other_data": null,
            "config_data": null,
            "sort_order": 8,
            "created_at": "2024-01-11 23:03:09",
            "updated_at": "2024-01-11 23:03:09",
            "image": null,
            "file": {
              "file_name": "592abbed67186697858239.pdf",
              "file_size": 39007,
              "path": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/049\/63a\/65a04963ab2ef576784647.pdf"
            },
            "files": [],
            "ratings_count": 0,
            "countRating": 0,
            "sumRating": 0,
            "averageRating": 0,
            "user_is_rating": 0,
            "user_object_rating": null,
            "favorites_count": 0,
            "user_is_favorite": 0,
            "likes_count": 0,
            "bookmarks_count": 0,
            "reactions_count": 0,
            "object_type": "Tss\\Media\\Models\\File"
          }
        ]
      }
    }
  ],
  "meta": { ... }
}
```

---

### 6.2 جلب حزم الملفات مع تضمين التصنيف والوسوم (`include=categorie,tags,pcategories`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/filelists?include=categorie,tags,pcategories
```

**الاستجابة** (مقتطف)

```json
{
  "data": [
    {
      "id": 7,
      ... // باقي الحقول
      "categorie": {
        "id": 5,
        "code": "2-2-5",
        "name": "المجموعه الرئيسية للملفات والوثائق",
        ...
      },
      "tags": {
        "data": [
          {
            "id": 1,
            "code": "2-4-1",
            "name": "منوعات",
            ...
          }
        ]
      },
      "pcategories": {
        "data": [
          {
            "id": 1,
            "code": "2-4-1",
            "name": "عامه",
            "image": { ... }
          }
        ]
      }
    }
  ],
  "meta": { ... }
}
```

---

### 6.3 استبعاد الحقول من حزمة ملفات (`exclude=created_at,updated_at`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/filelists?exclude=created_at,updated_at
```

**الاستجابة** (بدون التواريخ)

```json
{
  "data": [
    {
      "id": 7,
      "code": "2-2-7",
      "name": "وثائق تعليميه",
      // لا يحتوي على created_at, updated_at
      ...
    }
  ]
}
```

---

### 6.4 عرض حزمة ملفات واحدة مع الملفات (`/filelists/{id}`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/filelists/7
```

**الاستجابة**

```json
{
  "id": 7,
  "code": "2-2-7",
  "name": "وثائق تعليميه",
  "slug": "othak-taalymyh",
  "short_description": "",
  "description": "",
  "meta_title": "وثائق تعليميه",
  "meta_description": "",
  "keywords": "",
  "ref_type_class": "file",
  "ref_key_values_class": "7",
  "companys_id": "2",
  "departments_id": "2",
  "is_default": 0,
  "is_active": 1,
  "is_published": 1,
  "published_at": "",
  "unpublished_at": "",
  "main_sub": "sub",
  "parent_id": "5",
  "level": 2,
  "properties": [],
  "created_at": "2024-01-11 22:57:59",
  "updated_at": "2024-01-11 22:57:59",
  "image": null,
  "images": [],
  "ratings_count": 0,
  "countRating": 0,
  "sumRating": 0,
  "averageRating": 0,
  "user_is_rating": 0,
  "user_object_rating": null,
  "favorites_count": 0,
  "user_is_favorite": 0,
  "likes_count": 0,
  "bookmarks_count": 0,
  "reactions_count": 0,
  "object_type": "Tss\\Media\\Models\\Filelist",
  "files": {
    "data": [
      {
        "id": 8,
        "code": "2-2-8",
        "barcode": null,
        "name": "ملف تعليمي لنظام نانو سوفت",
        // ... كامل بيانات الملف كما في المثال 6.1
      }
    ]
  }
}
```

---

### 6.5 التحقق من آخر تحديث لحزم الملفات (`/filelists/activelystats`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/filelists/activelystats?date=2022-12-15%2017:10:00
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,
    "check_date": "2022-12-15 2017:10:00",
    "last_updated": "2024-01-12 16:27:22",
    "other_updated": {
      "activity_cache.filelist": "2024-01-12 16:27:22",
      "activity_cache.file": "2024-01-12 16:27:22",
      "activity_cache.categorie": "2024-01-12 16:27:22"
    }
  }
}
```

---

## 7. الملفات (Files)

نقطة النهاية: `GET /api/v1/media/files`

### 7.0 جلب جميع الملفات

**الطلب**

```http
GET http://localhost:8006/api/v1/media/files
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 8,
      "code": "2-2-8",
      "barcode": null,
      "name": "ملف تعليمي لنظام نانو سوفت",
      "slug": "mlf-taalymy-lntham-nano-soft",
      "short_description": "",
      "description": "",
      "meta_title": "ملف تعليمي لنظام نانو سوفت",
      "meta_description": "",
      "keywords": "",
      "categories_id": 7,
      "ref_type_class": "file",
      "ref_key_values_class": "7",
      "type": "public",
      "type_process": "open",
      "link": "",
      "companys_id": "2",
      "departments_id": "2",
      "is_effective": 0,
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "start_at": "",
      "end_at": "",
      "is_downloadable": 1,
      "modul_type": null,
      "ref_type": null,
      "properties": null,
      "links": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 8,
      "created_at": "2024-01-11 23:03:09",
      "updated_at": "2024-01-11 23:03:09",
      "image": null,
      "file": {
        "file_name": "592abbed67186697858239.pdf",
        "file_size": 39007,
        "path": "http:\/\/localhost:8006\/storage\/app\/uploads\/public\/65a\/049\/63a\/65a04963ab2ef576784647.pdf"
      },
      "files": [],
      "ratings_count": 0,
      "countRating": 0,
      "sumRating": 0,
      "averageRating": 0,
      "user_is_rating": 0,
      "user_object_rating": null,
      "favorites_count": 0,
      "user_is_favorite": 0,
      "likes_count": 0,
      "bookmarks_count": 0,
      "reactions_count": 0,
      "object_type": "Tss\\Media\\Models\\File"
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 7.1 جلب الملفات مع تضمين حزمة الملفات (`include=filelist`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/files?include=filelist
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 8,
      "name": "ملف تعليمي لنظام نانو سوفت",
      ... // باقي الحقول
      "filelist": {
        "id": 7,
        "code": "2-2-7",
        "name": "وثائق تعليميه",
        // ... كامل بيانات حزمة الملفات
      }
    }
  ],
  "meta": { ... }
}
```

---

### 7.2 استبعاد الحقول من الملف (`exclude=created_at,updated_at`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/files?exclude=created_at,updated_at
```

**الاستجابة** (بدون التواريخ)

```json
{
  "data": [
    {
      "id": 8,
      "code": "2-2-8",
      "name": "ملف تعليمي لنظام نانو سوفت",
      "file": { ... }
      // لا يوجد created_at, updated_at
    }
  ]
}
```

---

### 7.3 التحقق من آخر تحديث للملفات (`/files/activelystats`)

**الطلب**

```http
GET http://localhost:8006/api/v1/media/files/activelystats?date=2022-12-15%2017:10:00
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,
    "check_date": "2022-12-15 2017:10:00",
    "last_updated": "2024-01-12 16:19:56",
    "other_updated": {
      "activity_cache.file": "2024-01-12 16:19:56",
      "activity_cache.filelist": "2024-01-12 16:19:56"
    }
  }
}
```

---

**تاريخ إعداد الأمثلة:** 2026-06-01  
**الاعتماد على استجابات فعلية من الإصدار 1.0.2 من `Nano.MediaApi`**