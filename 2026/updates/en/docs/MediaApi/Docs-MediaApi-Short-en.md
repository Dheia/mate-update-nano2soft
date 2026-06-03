# Media Library API Documentation (Short Version)

## Introduction

This document provides a full description of the Media Library API provided by **Smart Nano Soft**.  
The library is divided into three main sections:

- **Images** ← Categories ← Albums ← Images
- **Videos** ← Categories ← Playlists ← Videos
- **Files** ← Categories ← Filelists ← Files

All calls are `GET` requests, and parameters are used for filtering, inclusion, and search.

> **Note:** `{base_url}` is the base API URL, and all paths below are relative to it.

---

## 1. Images

### 1.1 Fetch Image Categories

```http
GET /api/v1/media/categories?reftype=image
```

**Example**  
```
GET {base_url}/api/v1/media/categories?reftype=image
```

---

### 1.2 Fetch Albums

You can fetch all albums or filter them by a specific category.

#### 1.2.1 Fetch all albums

```http
GET /api/v1/media/albums
```

#### 1.2.2 Fetch albums by a specific category

```http
GET /api/v1/media/albums?categories_id={category_id}
```

**Example**  
```
GET /api/v1/media/albums?categories_id=3
```

#### 1.2.3 Include the images belonging to each album

Use the `include` parameter with the value `photos`:

```http
GET /api/v1/media/albums?include=photos
```
or with category:
```http
GET /api/v1/media/albums?categories_id=3&include=photos
```

---

### 1.3 Fetch Images

```http
GET /api/v1/media/images
```

#### Optional Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `albums_id` | integer | Fetch images belonging to a specific album |
| `categories_id` | integer | Filter images by a specific category |
| `q` | string | Text search in images (name, description, etc.) |

**Examples**

```
GET /api/v1/media/images?albums_id=5
GET /api/v1/media/images?categories_id=3
GET /api/v1/media/images?q=nature
```

#### Image Record Structure

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Image name |
| `short_description` | text | Short description (plain text) |
| `description` | html | Detailed description (HTML) |
| `image` | object | Image file in several sizes (paths, dimensions) |

---

## 2. Videos

### 2.1 Fetch Video Categories

```http
GET /api/v1/media/categories?reftype=video
```

---

### 2.2 Fetch Playlists

#### 2.2.1 Fetch all playlists

```http
GET /api/v1/media/playlists
```

#### 2.2.2 Fetch playlists by a specific category

```http
GET /api/v1/media/playlists?categories_id={category_id}
```

**Example**  
```
GET /api/v1/media/playlists?categories_id=5
```

#### 2.2.3 Include the videos belonging to each playlist

```http
GET /api/v1/media/playlists?include=videos
```
or with category:
```http
GET /api/v1/media/playlists?categories_id=5&include=videos
```

---

### 2.3 Fetch Videos

```http
GET /api/v1/media/videos
```

#### Optional Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `playlists_id` | integer | Fetch videos belonging to a specific playlist |
| `categories_id` | integer | Filter videos by a specific category |
| `q` | string | Text search in videos (name, description, etc.) |

**Examples**

```
GET /api/v1/media/videos?playlists_id=8
GET /api/v1/media/videos?categories_id=5
GET /api/v1/media/videos?q=educational
```

#### Video Record Structure

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Video name |
| `short_description` | text | Short description (plain text) |
| `description` | html | Detailed description (HTML) |
| `image` | object | Cover image (if any) |
| `video` | object | Video file (path, type, size, etc.) |

---

## 3. Files

### 3.1 Fetch File Categories

```http
GET /api/v1/media/categories?reftype=file
```

---

### 3.2 Fetch Filelists

#### 3.2.1 Fetch all filelists

```http
GET /api/v1/media/filelists
```

#### 3.2.2 Fetch filelists by a specific category

```http
GET /api/v1/media/filelists?categories_id={category_id}
```

**Example**  
```
GET /api/v1/media/filelists?categories_id=6
```

#### 3.2.3 Include the files belonging to each filelist

```http
GET /api/v1/media/filelists?include=files
```
or with category:
```http
GET /api/v1/media/filelists?categories_id=6&include=files
```

---

### 3.3 Fetch Files

```http
GET /api/v1/media/files
```

#### Optional Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `filelists_id` | integer | Fetch files belonging to a specific filelist |
| `categories_id` | integer | Filter files by a specific category |
| `q` | string | Text search in files (name, description, etc.) |

**Examples**

```
GET /api/v1/media/files?filelists_id=10
GET /api/v1/media/files?categories_id=6
GET /api/v1/media/files?q=report
```

#### File Record Structure

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | File name |
| `short_description` | text | Short description (plain text) |
| `description` | html | Detailed description (HTML) |
| `image` | object | Cover image (if any) |
| `file` | object | File data and path (URL, extension, size, etc.) |

---

## General Notes

- All calls return data in JSON format.
- The `include` parameter is used to load related relationships directly (e.g., `photos`, `videos`, `files`) and avoids additional calls.
- The `q` parameter enables text search in textual fields (name, short description, detailed description).
- Additional documentation for response codes and errors is not provided here; standard HTTP codes (200, 400, 404, etc.) are assumed.

---

## Additional Documentation

- [Media Library API Documentation (MediaApi)](./Docs-MediaApi-en.md)
- [Media Library API Short Documentation (MediaApi)](./Docs-MediaApi-Short-en.md)
- [Comprehensive Practical Examples – MediaApi Plugin](./Docs-MediaApi-Examples-en.md)
- [Short Practical Examples – MediaApi Plugin](./Docs-MediaApi-Short-Examples-en.md)
