# Documentation of the `Nano.MediaApi` Plugin

## Introduction

The **Nano.MediaApi** plugin provides a fully integrated API for managing multimedia content within the NanoSoft APP system, following the `Nano.*Api` pattern. The plugin is designed to facilitate the retrieval of **images**, **videos**, and **files** in a hierarchical structure via categories and intermediate containers (Albums, Playlists, Filelists).

All endpoints use only the `GET` method (read‑only) and support a wide range of **filters**, **search**, **ordering**, **including relationships** (`include`), **excluding fields** (`exclude`), and optional caching for performance.

The user must be **logged in** (JWT token or session) to access most endpoints, as company, department, and user access scopes are automatically applied (according to the settings).

---

## Table of Contents

1. [Media Library Structure](#media-library-structure)
2. [Common Endpoints and Shared Parameters](#common-endpoints-and-shared-parameters)
   - [Search `q`](#search-q)
   - [Ordering `orderBy` / `orderDirection`](#ordering-orderby--orderdirection)
   - [Pagination `page` / `per_page`](#pagination-page--per_page)
   - [Excluding Fields `exclude`](#excluding-fields-exclude)
   - [Including Relationships `include`](#including-relationships-include)
3. [Images Section](#images-section)
   - [1. Categories – `reftype=image`](#1-categories--reftypeimage)
   - [2. Albums](#2-albums)
   - [3. Images](#3-images)
4. [Videos Section](#videos-section)
   - [1. Categories – `reftype=video`](#1-categories--reftypevideo)
   - [2. Playlists](#2-playlists)
   - [3. Videos](#3-videos)
5. [Files Section](#files-section)
   - [1. Categories – `reftype=file`](#1-categories--reftypefile)
   - [2. Filelists](#2-filelists)
   - [3. Files](#3-files)
6. [Checking the Last Update (activelystats)](#checking-the-last-update-activelystats)
7. [General Notes](#general-notes)

---

## Media Library Structure

The plugin relies on the following hierarchy:

- **Images**  
  `Categories` ← `Albums` ← `Images`

- **Videos**  
  `Categories` ← `Playlists` ← `Videos`

- **Files**  
  `Categories` ← `Filelists` ← `Files`

Each of these elements has its own database object, with appropriate `belongsTo` / `hasMany` relationships. Data is automatically filtered according to the current user’s `companys_id` and `departments_id` (if present).

---

## Common Endpoints and Shared Parameters

All the following endpoints accept the same set of optional parameters, with some resource‑specific additions.

### Search `q`

The search text is applied to the fields: `name`, `short_description`, `description`, `meta_title`, `meta_description`, `keywords`.

```
GET /api/v1/media/images?q=nature
```

### Ordering `orderBy` / `orderDirection`

- `orderBy`: column name (e.g., `created_at`, `sort_order`, `name`).
- `orderDirection`: `asc` or `desc` (default `desc`).

If `orderBy` is not provided, the default ordering from the configuration file (`config.php`) is used (usually `sort_order asc`).

### Pagination `page` / `per_page`

- `per_page`: number of results per page (default 15).
- `page`: requested page number (default 1).

### Excluding Fields `exclude`

Pass comma‑separated field names to be excluded from the response:

```
GET /api/v1/media/images?exclude=created_at,updated_at
```

### Including Relationships `include`

Pass relationship names to be included in the same response (to avoid additional calls). Available relationships vary by resource (see tables below). Example:

```
GET /api/v1/media/albums?include=photos,categorie
```

---

## Images Section

### 1. Categories – `reftype=image`

**Endpoint:**  
`GET /api/v1/media/categories?reftype=image`

Fetches categories specific to images.

#### Specific Filters (in addition to common ones)

| Parameter | Type | Description |
|-----------|------|-------------|
| `parent_id` | integer | Fetch sub‑categories of a specific parent category. |
| `isActive` | boolean | Show only active (default `true`). |
| `isNotHidden` | boolean | Exclude hidden (default `true`). |
| `isHidden` | boolean | Show only hidden (default `false`). |
| `IsMain` | boolean | Only main categories (default `false`). |
| `IsSub` | boolean | Only sub‑categories (default `false`). |
| `companys_id` | integer | Filter by company. |
| `departments_id` | integer | Filter by department. |
| `country_id` | integer/string | Filter by country (via `Department` relationship). |
| `state_id` | integer/string | Filter by city (via `Department` relationship). |
| `categorysId` | string | Filter by general category IDs (pcategories). |
| `tagsId` | string | Filter by tag IDs. |
| `menus_id` | integer | Apply custom menu conditions. |
| `isFavorites` | boolean | Fetch categories favourited by the current user. |
| `isLikes` | boolean | Fetch categories liked by the user. |
| `isBookmarks` | boolean | Fetch categories bookmarked by the user. |
| `isReactions` | boolean | Fetch categories reacted to by the user. |
| `with_count` | boolean | Add counts of associated videos/albums (e.g., `videos_count`). |
| `isDisplayEmpty` | boolean | `false` to show only categories that contain videos (default `true`). |
| `displayTo` | integer | Minimum number of videos when `isDisplayEmpty=false` (default 1). |

#### Available `include` Relationships

| Relationship | Description |
|--------------|-------------|
| `companys` | Company data. |
| `department` | Department data. |
| `parent` | Parent category. |
| `children` | Sub‑categories (with pagination support). |
| `tags` | Associated tags. |
| `pcategories` | General categories (from `Nano.TagsApi`). |
| `albums` | Albums belonging to this category. |
| `playlists` | Playlists belonging to this category (for `video` categories). |
| `filelists` | Filelists belonging to this category (for `file` categories). |
| `videos` | Videos directly belonging to this category (rare). |
| `user_object_favorite` | User’s favourite object (if any). |
| `user_object_like` | User’s like object. |
| `user_object_bookmark` | User’s bookmark object. |
| `user_object_reaction` | User’s reaction object. |

#### Example: Fetch main image categories including albums

```http
GET /api/v1/media/categories?reftype=image&IsMain=1&include=albums
```

#### Example: Fetch categories that contain at least one video

```http
GET /api/v1/media/categories?reftype=video&isDisplayEmpty=false&displayTo=1
```

#### Example: Exclude some fields

```http
GET /api/v1/media/categories?exclude=created_at,updated_at,properties
```

---

### 2. Albums

**Endpoint:**  
`GET /api/v1/media/albums`

#### Specific Filters

| Parameter | Type | Description |
|-----------|------|-------------|
| `categories_id` or `parent_id` | integer | Fetch albums belonging to a specific category. |
| `isActive` | boolean | Show active albums. |
| `isPublished` | boolean | Show published albums. |
| `isNotHidden` / `isHidden` | boolean | Show non‑hidden / hidden. |
| `IsMain` / `IsSub` | boolean | Main / sub albums. |
| `companys_id` / `departments_id` | integer | Filter by company/department. |
| `country_id` / `state_id` | integer/string | Filter by country/city. |
| `isDisplayEmpty` | boolean | `false` to show only albums containing images. |
| `displayTo` | integer | Minimum number of images when `isDisplayEmpty=false`. |
| `isFavorites`, `isLikes`, `isBookmarks`, `isReactions` | boolean | Fetch based on user interactions. |
| `with_count` | boolean | Add `videos_count` (in Album, the name is kept). |

#### Available `include` Relationships

| Relationship | Description |
|--------------|-------------|
| `companys` | Company. |
| `department` | Department. |
| `categorie` | Associated category (parent). |
| `parent` | Parent album (if exists). |
| `children` | Sub‑albums (with pagination). |
| `tags` | Tags. |
| `pcategories` | General categories. |
| `photos` | Images belonging to this album (`Image` objects). |
| `user_object_*` | User interaction objects. |

#### Example: Fetch all albums with images

```http
GET /api/v1/media/albums?include=photos
```

#### Example: Fetch albums for a specific category (categories_id=2) with basic relationships

```http
GET /api/v1/media/albums?categories_id=2&include=categorie,tags,pcategories
```

#### Example: Fetch a single album by ID 4

```http
GET /api/v1/media/albums/4
```

---

### 3. Images

**Endpoint:**  
`GET /api/v1/media/images`

#### Specific Filters

| Parameter | Type | Description |
|-----------|------|-------------|
| `albums_id` | integer | Fetch images belonging to a specific album. |
| `categories_id` | integer | Filter images by category (supports **advanced filtering** – includes images whose album’s category parent matches). |
| `isActive` | boolean | Show active images. |
| `isPublished` | boolean | Show published images. |
| `companys_id` / `departments_id` | integer | Filter by company/department. |
| `country_id` / `state_id` | integer/string | Filter by country/city. |
| `categorysId` | string | Filter by general categories (pcategories). |
| `tagsId` | string | Filter by tags. |
| `menus_id` | integer | Apply custom menu conditions. |
| `isFavorites`, `isLikes`, `isBookmarks`, `isReactions` | boolean | Filter by user interactions. |
| `q` | string | Search in `name`, `short_description`, `description`, etc. |

#### Available `include` Relationships

| Relationship | Description |
|--------------|-------------|
| `companys` | Company. |
| `department` | Department. |
| `categorie` | Direct category of the image. |
| `album` | Album to which the image belongs. |
| `tags` | Tags. |
| `pcategories` | General categories. |
| `user_object_*` | User interaction objects (favourite, like, etc.). |

#### Example: Fetch images belonging to album 4

```http
GET /api/v1/media/images?albums_id=4
```

#### Example: Fetch images by category (includes sub‑categories)

```http
GET /api/v1/media/images?categories_id=2
```

> **Note:** Since version 1.0.2, `categories_id` also supports fetching images that belong to an album whose album’s `parent_id` equals the provided category.

#### Example: Search for images containing the word "logo"

```http
GET /api/v1/media/images?q=logo
```

#### Example: Fetch images with album and tags included

```http
GET /api/v1/media/images?include=album,tags
```

---

## Videos Section

### 1. Categories – `reftype=video`

Same endpoints as general categories but with `reftype=video`. All filters and relationships mentioned in the image categories section also apply to video categories, except that the available relationships may include `playlists` and `videos` instead of `albums`.

**Endpoint:**  
`GET /api/v1/media/categories?reftype=video`

#### Example: Fetch video categories with playlists included

```http
GET /api/v1/media/categories?reftype=video&include=playlists
```

---

### 2. Playlists

**Endpoint:**  
`GET /api/v1/media/playlists`

#### Specific Filters

| Parameter | Type | Description |
|-----------|------|-------------|
| `categories_id` or `parent_id` | integer | Playlists belonging to a specific category. |
| `isActive`, `isPublished` | boolean | Status. |
| `isNotHidden` / `isHidden` | boolean | Show/hide. |
| `IsMain` / `IsSub` | boolean | Main / sub. |
| `companys_id` / `departments_id` | integer | Filter. |
| `country_id` / `state_id` | integer/string | Geographic filter. |
| `isDisplayEmpty` | boolean | `false` to show only playlists containing videos. |
| `with_count` | boolean | Add `videos_count`. |
| `isFavorites`, etc. | boolean | User interactions. |

#### Available `include` Relationships

| Relationship | Description |
|--------------|-------------|
| `companys`, `department` | Company and department data. |
| `categorie` | Associated category. |
| `parent`, `children` | Tree structure (if any). |
| `tags`, `pcategories` | Tags and general categories. |
| `videos` | Videos belonging to this playlist. |
| `user_object_*` | User interaction objects. |

#### Example: Fetch playlists with videos

```http
GET /api/v1/media/playlists?include=videos
```

#### Example: Fetch playlists belonging to a main category (categories_id=3)

```http
GET /api/v1/media/playlists?categories_id=3
```

#### Example: Fetch a single playlist by ID 6

```http
GET /api/v1/media/playlists/6
```

---

### 3. Videos

**Endpoint:**  
`GET /api/v1/media/videos`

#### Specific Filters

| Parameter | Type | Description |
|-----------|------|-------------|
| `playlists_id` | integer | Videos belonging to a specific playlist. |
| `categories_id` | integer | Filter by category (supports parent categories via the `playlist` relationship). |
| `isActive`, `isPublished` | boolean | Status. |
| `companys_id` / `departments_id` | integer | Filter. |
| `country_id` / `state_id` | integer/string | Geographic filter. |
| `categorysId`, `tagsId` | string | Filter by general categories / tags. |
| `menus_id` | integer | Apply custom menu conditions. |
| `isFavorites`, etc. | boolean | User interactions. |
| `q` | string | Text search. |

#### Available `include` Relationships

| Relationship | Description |
|--------------|-------------|
| `companys`, `department` | Company and department. |
| `categorie` | Direct category. |
| `playlist` | Playlist to which the video belongs. |
| `tags`, `pcategories` | Tags and general categories. |
| `user_object_*` | Interaction objects. |

#### Example: Fetch videos belonging to playlist 6

```http
GET /api/v1/media/videos?playlists_id=6
```

#### Example: Fetch videos by category (with playlist included)

```http
GET /api/v1/media/videos?categories_id=3&include=playlist
```

#### Example: Search for a video with the word "educational"

```http
GET /api/v1/media/videos?q=educational
```

#### Simplified video response structure

```json
{
  "id": 7,
  "name": "First video",
  "description": "...",
  "image": { "original": "...", "thumb": "..." },
  "video": { "file_name": "...", "path": "...", "file_size": 783126 },
  "object_type": "Tss\\Media\\Models\\Video"
}
```

---

## Files Section

### 1. Categories – `reftype=file`

**Endpoint:**  
`GET /api/v1/media/categories?reftype=file`

Same filters and relationships as before, with support for `filelists` instead of `albums`/`playlists`.

#### Example: Fetch file categories with filelists included

```http
GET /api/v1/media/categories?reftype=file&include=filelists
```

---

### 2. Filelists

**Endpoint:**  
`GET /api/v1/media/filelists`

#### Specific Filters

| Parameter | Type | Description |
|-----------|------|-------------|
| `categories_id` or `parent_id` | integer | Filelists belonging to a specific category. |
| `isActive`, `isPublished` | boolean | Status. |
| `isNotHidden` / `isHidden` | boolean | Show/hide. |
| `IsMain` / `IsSub` | boolean | Main / sub. |
| `companys_id` / `departments_id` | integer | Filter. |
| `country_id` / `state_id` | integer/string | Geographic filter. |
| `isDisplayEmpty` | boolean | `false` to show only filelists containing files. |
| `with_count` | boolean | Add `files_count`. |
| `isFavorites`, etc. | boolean | User interactions. |

#### Available `include` Relationships

| Relationship | Description |
|--------------|-------------|
| `companys`, `department` | Company and department. |
| `categorie` | Associated category. |
| `parent`, `children` | Tree structure. |
| `tags`, `pcategories` | Tags and general categories. |
| `files` | Files belonging to this filelist. |
| `user_object_*` | Interaction objects. |

#### Example: Fetch filelists with files

```http
GET /api/v1/media/filelists?include=files
```

#### Example: Fetch filelists belonging to a main category

```http
GET /api/v1/media/filelists?categories_id=5
```

#### Example: Fetch a single filelist by ID 7

```http
GET /api/v1/media/filelists/7
```

---

### 3. Files

**Endpoint:**  
`GET /api/v1/media/files`

#### Specific Filters

| Parameter | Type | Description |
|-----------|------|-------------|
| `filelists_id` | integer | Files belonging to a specific filelist. |
| `categories_id` | integer | Filter by category (supports parent categories via the `filelist` relationship). |
| `isActive`, `isPublished` | boolean | Status. |
| `is_downloadable` | boolean | (In the data, not a direct filter) |
| `companys_id` / `departments_id` | integer | Filter. |
| `country_id` / `state_id` | integer/string | Geographic filter. |
| `categorysId`, `tagsId` | string | Filter. |
| `menus_id` | integer | Menu conditions. |
| `isFavorites`, etc. | boolean | User interactions. |
| `q` | string | Search. |

#### Available `include` Relationships

| Relationship | Description |
|--------------|-------------|
| `companys`, `department` | Company and department. |
| `categorie` | Direct category. |
| `filelist` | Filelist to which the file belongs. |
| `tags`, `pcategories` | Tags and general categories. |
| `user_object_*` | Interaction objects. |

#### Example: Fetch files belonging to filelist 7

```http
GET /api/v1/media/files?filelists_id=7
```

#### Example: Fetch files by a specific category with filelist included

```http
GET /api/v1/media/files?categories_id=5&include=filelist
```

#### Example: Search for a file named "guide"

```http
GET /api/v1/media/files?q=guide
```

#### Simplified file response structure

```json
{
  "id": 8,
  "name": "Educational document for NanoSoft system",
  "file": { "file_name": "document.pdf", "path": "https://...", "file_size": 39007 },
  "is_downloadable": 1,
  "object_type": "Tss\\Media\\Models\\File"
}
```

---

## Checking the Last Update (activelystats)

Every `index` endpoint supports an `activelystats` endpoint that allows checking whether data has changed after a given date. This is useful for client‑side caching.

**Endpoint:**  
`GET /api/v1/media/{resource}/activelystats`

**Parameter:**  
- `date` (optional): a date in `Y-m-d H:i:s` format. Default is the current time.

**Response:**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,      // true if there is an update after the passed date
    "check_date": "2024-01-12 16:24:45",
    "last_updated": "2024-01-12 16:25:04",
    "other_updated": {
      "activity_cache.image": "2024-01-12 16:25:04",
      "activity_cache.album": "2024-01-12 16:25:04"
    }
  }
}
```

**Example:**  
```http
GET /api/v1/media/images/activelystats?date=2024-01-01%2000:00:00
```

---

## General Notes

- All responses return JSON data, following the structure `{ "data": [...], "meta": { "pagination": {...} } }` when using `index`, or the object directly when using `show`.
- Caching at the plugin level is enabled by default if `nano.api::api_enable_cache` is `true`. The cache can be cleared via `php artisan cache:clear`.
- Filtering by `categories_id` in images, videos, and files **supports parent categories** via the relationships (`album`, `playlist`, `filelist`) since version 1.0.2. This means that passing `categories_id=2` will also fetch items whose sub‑category (belonging to the album/playlist/filelist) is 2.
- `include` and `exclude` can be used together. Exclusion takes precedence (if a field that appears in an included relationship is excluded, it may be removed only from the main data).
- There are no endpoints for creating, updating, or deleting media via this API (to maintain security and leave those operations to the admin panel or other tools). The plugin focuses on **read‑only** access.
- To support dynamic `exclude`, the `scopeExclude` scope has been added to all models (Categories, Album, Image, Playlist, Video, Filelist, File) inside `Plugin.php`.
- The default ordering for each resource can be customised via the `config.php` file or environment variables (e.g., `NANO_MEDIAAPI_IMAGES_ORDER_BY=sort_order`).

---

## Additional Documentation

- [Media Library API Documentation (MediaApi)](./Docs-MediaApi-en.md)
- [Media Library API Short Documentation (MediaApi)](./Docs-MediaApi-Short-en.md)
- [Comprehensive Practical Examples – MediaApi Plugin](./Docs-MediaApi-Examples-en.md)
- [Short Practical Examples – MediaApi Plugin](./Docs-MediaApi-Short-Examples-en.md)

**Last updated:** 2026-06-01  
**Documented version:** 1.0.2 (supports advanced parent‑category filtering)
