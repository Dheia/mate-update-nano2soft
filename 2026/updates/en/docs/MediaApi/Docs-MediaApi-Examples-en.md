# Comprehensive Practical Examples – `Nano.MediaApi` Plugin

This document contains real, direct examples of using all endpoints for managing the **Media Library** (images, videos, files) via the API. All the following data are extracted from actual system responses, preserving the original accuracy and formatting as they appear in the official documentation files.

---

## Table of Examples

1. [Categories](#1-categories)
   - 1.0 Fetch all categories
   - 1.1 Fetch categories with tags and general categories
   - 1.2 Fetch categories with albums / playlists / filelists
   - 1.3 Image categories only with albums included
   - 1.4 Video categories only with playlists included
   - 1.5 File categories only with filelists included
   - 1.6 View a single category
   - 1.7 Check last update of categories
2. [Albums](#2-albums)
   - 2.0 Fetch all albums
   - 2.1 Exclude fields from an album
   - 2.2 Check last update of albums
3. [Images](#3-images)
   - 3.0 Fetch all images (with pagination)
   - 3.1 Fetch images with album included
   - 3.2 Fetch images with category, tags, and general categories
   - 3.3 Exclude fields from images
   - 3.4 View a single image with tags and album
   - 3.5 Check last update of images
4. [Playlists](#4-playlists)
   - 4.0 Fetch all playlists
   - 4.1 Fetch playlists with videos included
   - 4.2 Fetch playlists with category and tags
   - 4.3 Exclude fields from a playlist
   - 4.4 View a single playlist
   - 4.5 Check last update of playlists
5. [Videos](#5-videos)
   - 5.0 Fetch all videos
   - 5.1 Fetch videos with playlist included
   - 5.2 Fetch videos with category and tags
   - 5.3 Exclude fields from a video
   - 5.4 View a single video
   - 5.5 Check last update of videos
6. [Filelists](#6-filelists)
   - 6.0 Fetch all filelists
   - 6.1 Fetch filelists with files included
   - 6.2 Fetch filelists with category and tags
   - 6.3 Exclude fields from a filelist
   - 6.4 View a single filelist with files
   - 6.5 Check last update of filelists
7. [Files](#7-files)
   - 7.0 Fetch all files
   - 7.1 Fetch files with filelist included
   - 7.2 Exclude fields from a file
   - 7.3 Check last update of files

---

## 1. Categories

Endpoint: `GET /api/v1/media/categories`

### 1.0 Fetch all categories

**Request**

```http
GET http://localhost:8006/api/v1/media/categories
```

**Response**

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

### 1.1 Fetch categories with tags and general categories (`include=tags,pcategories`)

**Request**

```http
GET http://localhost:8006/api/v1/media/categories?include=tags,pcategories
```

**Response** (excerpt – first category only, preserving the full structure as in the original file)

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
    // remaining categories (2, 3, 5) with the same structure and their own data
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

### 1.2 Fetch categories with albums / playlists / filelists (`include=albums,playlists,filelists`)

**Request**

```http
GET http://localhost:8006/api/v1/media/categories?include=albums,playlists,filelists
```

**Response**

```json
{
  "data": [
    {
      "id": 1,
      "code": "2-2-1",
      "name": "المجموعه الرئيسيه لتصنيفات الصور",
      ... // remaining fields as in example 1.0
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

### 1.3 Image categories only with albums included (`reftype=image&include=albums,playlists,filelists`)

**Request**

```http
GET http://localhost:8006/api/v1/media/categories?reftype=image&include=albums,playlists,filelists
```

**Response**

```json
{
  "data": [
    {
      "id": 1,
      "code": "2-2-1",
      "name": "المجموعه الرئيسيه لتصنيفات الصور",
      ... // remaining fields
      "albums": { "data": [] },
      "playlists": { "data": [] },
      "filelists": { "data": [] }
    },
    {
      "id": 2,
      "code": "2-2-2",
      "name": "المجموعه الفرعية لتصنيفات الصور",
      ... // remaining fields
      "albums": {
        "data": [
          {
            "id": 4,
            "code": "2-2-4",
            "name": "البوم الصور الاول",
            // ... full album data as in example 1.2
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

### 1.4 Video categories only with playlists included (`reftype=video&include=albums,playlists,filelists`)

**Request**

```http
GET http://localhost:8006/api/v1/media/categories?reftype=video&include=albums,playlists,filelists
```

**Response**

```json
{
  "data": [
    {
      "id": 3,
      "code": "2-2-3",
      "name": "المجموعه الرئيسيه لتصنيفات الفيديوهات",
      ... // remaining fields
      "albums": { "data": [] },
      "playlists": {
        "data": [
          {
            "id": 6,
            "code": "2-2-6",
            "name": "قائمة تشغيل فيديوهات رقم واحد",
            // ... full playlist data as in example 1.2
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

### 1.5 File categories only with filelists included (`reftype=file&include=albums,playlists,filelists`)

**Request**

```http
GET http://localhost:8006/api/v1/media/categories?reftype=file&include=albums,playlists,filelists
```

**Response**

```json
{
  "data": [
    {
      "id": 5,
      "code": "2-2-5",
      "name": "المجموعه الرئيسية للملفات والوثائق",
      ... // remaining fields
      "albums": { "data": [] },
      "playlists": { "data": [] },
      "filelists": {
        "data": [
          {
            "id": 7,
            "code": "2-2-7",
            "name": "وثائق تعليميه",
            // ... full filelist data as in example 1.2
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

### 1.6 View a single category (`/categories/{id}`)

**Request**

```http
GET http://localhost:8006/api/v1/media/categories/1
```

**Response**

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

### 1.7 Check last update of categories (`/categories/activelystats`)

**Request** (with a specific date)

```http
GET http://localhost:8006/api/v1/media/categories/activelystats?date=2022-12-15%2017:10:00
```

**Response**

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

**Without a date**

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

## 2. Albums

Endpoint: `GET /api/v1/media/albums`

### 2.0 Fetch all albums

**Request**

```http
GET http://localhost:8006/api/v1/media/albums
```

**Response**

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

### 2.1 Exclude fields from an album (`exclude=created_at,updated_at`)

**Request**

```http
GET http://localhost:8006/api/v1/media/albums?exclude=created_at,updated_at
```

**Response** (same data but without `created_at` and `updated_at`)

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

### 2.2 Check last update of albums (`/albums/activelystats`)

**Request** (with a date)

```http
GET http://localhost:8006/api/v1/media/albums/activelystats?date=2022-12-15%2017:10:00
```

**Response**

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

## 3. Images

Endpoint: `GET /api/v1/media/images`

### 3.0 Fetch all images (with pagination)

**Request**

```http
GET http://localhost:8006/api/v1/media/images?per_page=2
```

**Response**

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

### 3.1 Fetch images with album included (`include=album`)

**Request**

```http
GET http://localhost:8006/api/v1/media/images?include=album
```

**Response** (excerpt – first two images with album)

```json
{
  "data": [
    {
      "id": 1,
      "name": "صوه شعار",
      ... // remaining fields
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
      ... // remaining fields
      "album": { ... } // same album 4
    }
    // remaining images with the same structure
  ],
  "meta": { ... }
}
```

---

### 3.2 Fetch images with category, tags, and general categories (`include=tags,pcategories,categorie`)

**Request**

```http
GET http://localhost:8006/api/v1/media/images?include=tags,pcategories,categorie
```

**Response** (excerpt – first image with category)

```json
{
  "data": [
    {
      "id": 1,
      "name": "صوه شعار",
      ... // remaining fields
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
    // remaining images with the same structure
  ],
  "meta": { ... }
}
```

---

### 3.3 Exclude fields from images (`exclude=created_at,updated_at`)

**Request**

```http
GET http://localhost:8006/api/v1/media/images?exclude=created_at,updated_at
```

**Response** (same data without the two fields)

```json
{
  "data": [
    {
      "id": 1,
      "code": "2-2-1",
      "barcode": null,
      "name": "صوه شعار",
      // does not contain created_at, updated_at
      ...
    }
    // remaining images
  ]
}
```

---

### 3.4 View a single image with tags and album (`/images/{id}?include=tags,album`)

**Request**

```http
GET http://localhost:8006/api/v1/media/images/4?include=tags,album
```

**Response**

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

### 3.5 Check last update of images (`/images/activelystats`)

**Request**

```http
GET http://localhost:8006/api/v1/media/images/activelystats?date=2022-12-15%2017:10:00
```

**Response**

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

## 4. Playlists

Endpoint: `GET /api/v1/media/playlists`

### 4.0 Fetch all playlists

**Request**

```http
GET http://localhost:8006/api/v1/media/playlists
```

**Response**

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

### 4.1 Fetch playlists with videos included (`include=videos`)

**Request**

```http
GET http://localhost:8006/api/v1/media/playlists?include=videos
```

**Response** (playlist contains one video)

```json
{
  "data": [
    {
      "id": 6,
      "code": "2-2-6",
      "name": "قائمة تشغيل فيديوهات رقم واحد",
      ... // remaining fields
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

### 4.2 Fetch playlists with category and tags (`include=categorie,tags,pcategories`)

**Request**

```http
GET http://localhost:8006/api/v1/media/playlists?include=categorie,tags,pcategories
```

**Response** (excerpt)

```json
{
  "data": [
    {
      "id": 6,
      ... // remaining fields
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

### 4.3 Exclude fields from a playlist (`exclude=created_at,updated_at`)

**Request**

```http
GET http://localhost:8006/api/v1/media/playlists?exclude=created_at,updated_at
```

**Response** (without the dates)

```json
{
  "data": [
    {
      "id": 6,
      "code": "2-2-6",
      "name": "قائمة تشغيل فيديوهات رقم واحد",
      // does not contain created_at, updated_at
      ...
    }
  ]
}
```

---

### 4.4 View a single playlist (`/playlists/{id}`)

**Request**

```http
GET http://localhost:8006/api/v1/media/playlists/6
```

**Response**

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

### 4.5 Check last update of playlists (`/playlists/activelystats`)

**Request**

```http
GET http://localhost:8006/api/v1/media/playlists/activelystats?date=2022-12-15%2017:10:00
```

**Response**

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

## 5. Videos

Endpoint: `GET /api/v1/media/videos`

### 5.0 Fetch all videos

**Request**

```http
GET http://localhost:8006/api/v1/media/videos
```

**Response**

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

### 5.1 Fetch videos with playlist included (`include=playlist`)

**Request**

```http
GET http://localhost:8006/api/v1/media/videos?include=playlist
```

**Response**

```json
{
  "data": [
    {
      "id": 7,
      "name": "الفيديو الاول",
      ... // remaining fields
      "playlist": {
        "id": 6,
        "code": "2-2-6",
        "name": "قائمة تشغيل فيديوهات رقم واحد",
        "slug": "kam-tshghyl-fydyohat-rkm-oahd",
        // ... full playlist data as in example 4.0
      }
    }
  ],
  "meta": { ... }
}
```

---

### 5.2 Fetch videos with category and tags (`include=categorie,tags,pcategories`)

**Request**

```http
GET http://localhost:8006/api/v1/media/videos?include=categorie,tags,pcategories
```

**Response** (excerpt)

```json
{
  "data": [
    {
      "id": 7,
      ... // remaining fields
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

### 5.3 Exclude fields from a video (`exclude=created_at,updated_at`)

**Request**

```http
GET http://localhost:8006/api/v1/media/videos?exclude=created_at,updated_at
```

**Response** (without the dates)

```json
{
  "data": [
    {
      "id": 7,
      "name": "الفيديو الاول",
      "video": { ... },
      "image": { ... }
      // does not contain created_at, updated_at
    }
  ]
}
```

---

### 5.4 View a single video (`/videos/{id}`)

**Request**

```http
GET http://localhost:8006/api/v1/media/videos/7
```

**Response**

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

### 5.5 Check last update of videos (`/videos/activelystats`)

**Request**

```http
GET http://localhost:8006/api/v1/media/videos/activelystats?date=2022-12-15%2017:10:00
```

**Response**

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

## 6. Filelists

Endpoint: `GET /api/v1/media/filelists`

### 6.0 Fetch all filelists

**Request**

```http
GET http://localhost:8006/api/v1/media/filelists
```

**Response**

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

### 6.1 Fetch filelists with files included (`include=files`)

**Request**

```http
GET http://localhost:8006/api/v1/media/filelists?include=files
```

**Response**

```json
{
  "data": [
    {
      "id": 7,
      "code": "2-2-7",
      "name": "وثائق تعليميه",
      ... // remaining fields
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

### 6.2 Fetch filelists with category and tags (`include=categorie,tags,pcategories`)

**Request**

```http
GET http://localhost:8006/api/v1/media/filelists?include=categorie,tags,pcategories
```

**Response** (excerpt)

```json
{
  "data": [
    {
      "id": 7,
      ... // remaining fields
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

### 6.3 Exclude fields from a filelist (`exclude=created_at,updated_at`)

**Request**

```http
GET http://localhost:8006/api/v1/media/filelists?exclude=created_at,updated_at
```

**Response** (without the dates)

```json
{
  "data": [
    {
      "id": 7,
      "code": "2-2-7",
      "name": "وثائق تعليميه",
      // does not contain created_at, updated_at
      ...
    }
  ]
}
```

---

### 6.4 View a single filelist with files (`/filelists/{id}`)

**Request**

```http
GET http://localhost:8006/api/v1/media/filelists/7
```

**Response**

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
        // ... full file data as in example 6.1
      }
    ]
  }
}
```

---

### 6.5 Check last update of filelists (`/filelists/activelystats`)

**Request**

```http
GET http://localhost:8006/api/v1/media/filelists/activelystats?date=2022-12-15%2017:10:00
```

**Response**

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

## 7. Files

Endpoint: `GET /api/v1/media/files`

### 7.0 Fetch all files

**Request**

```http
GET http://localhost:8006/api/v1/media/files
```

**Response**

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

### 7.1 Fetch files with filelist included (`include=filelist`)

**Request**

```http
GET http://localhost:8006/api/v1/media/files?include=filelist
```

**Response**

```json
{
  "data": [
    {
      "id": 8,
      "name": "ملف تعليمي لنظام نانو سوفت",
      ... // remaining fields
      "filelist": {
        "id": 7,
        "code": "2-2-7",
        "name": "وثائق تعليميه",
        // ... full filelist data
      }
    }
  ],
  "meta": { ... }
}
```

---

### 7.2 Exclude fields from a file (`exclude=created_at,updated_at`)

**Request**

```http
GET http://localhost:8006/api/v1/media/files?exclude=created_at,updated_at
```

**Response** (without the dates)

```json
{
  "data": [
    {
      "id": 8,
      "code": "2-2-8",
      "name": "ملف تعليمي لنظام نانو سوفت",
      "file": { ... }
      // does not contain created_at, updated_at
    }
  ]
}
```

---

### 7.3 Check last update of files (`/files/activelystats`)

**Request**

```http
GET http://localhost:8006/api/v1/media/files/activelystats?date=2022-12-15%2017:10:00
```

**Response**

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

**Date of examples:** 2026-06-01  
**Based on actual responses from version 1.0.2 of `Nano.MediaApi`**