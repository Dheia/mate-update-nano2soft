## 2026-06-01 - 2026-06-02

**Update to the `Nano.MediaApi` Plugin – Version 1.0.2**

### Improved Category Filtering in Images, Videos, and Files Controllers to Support Parent Categories

---

### Summary of Updates

Version **1.0.2** of the `Nano.MediaApi` plugin introduces an important improvement to the mechanism of filtering items by category (`categories_id`). Previously, filtering was limited to a direct match of the `categories_id` of each item (image, video, file). Now, the logic has been expanded to also include items that belong to categories where the provided category is the **parent category** (`parent_id`), via the relationships (`album` for images, `playlist` for videos, `filelist` for files).

**Key changes:**
- Modified the `index` method in the controllers:
  - `Nano\MediaApi\APIControllers\Images`
  - `Nano\MediaApi\APIControllers\Videos`
  - `Nano\MediaApi\APIControllers\Files`
- Support for passing `categories_id` as an array to filter by several categories.
- Use of `orWhereHas` to reach indirect categories through the relationship.
- Updated the Media Library API documentation to reflect the new behaviour.

---

### Release Objectives

- **Improve filtering accuracy:** enable developers to retrieve all images, videos, or files under a given category **and all its sub‑categories** without multiple calls.
- **Simplify integration:** do not force the user to know the full category tree; passing the parent category is sufficient.
- **Flexible array support:** allow passing several categories (parents or children) in a single request.
- **Meet user expectations:** in content management systems, selecting a category is often expected to also display content from all its sub‑categories.

---

### New Features and Improvements

#### 1. Expanded `categories_id` Filtering in `Images`, `Videos`, and `Files`

**Before version 1.0.2:**
```php
if($categories_id = Input::get('categories_id', false)) {
    $posts = $posts->where('categories_id', $categories_id);
}
```

**After the update:**
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

> **Note:** The same logic has been applied in the `Videos` controller using the `playlist` relationship, and in the `Files` controller using the `filelist` relationship.

#### 2. Support for Passing `categories_id` as an Array

It is now possible to pass several categories at once:

```
GET /api/v1/media/images?categories_id[]=3&categories_id[]=5
```

This will fetch images that belong to category 3 or 5 **or** that belong to any album whose album’s parent category is 3 or 5.

#### 3. Updated API Documentation

The previous documentation (`Docs-MediaApi-en.md`) has been updated to explain the new behaviour and to clarify that filtering by `categories_id` now includes **all items associated with the passed category, either directly or through relationships (albums/playlists/filelists)**.

---

### Practical Examples of the New Usage

#### 1. Fetch All Images Under a Main Category (including its sub‑categories)

Assume you have category `10` named "Nature", with sub‑categories "Mountains" and "Seas".  
By passing `categories_id=10`, you will receive all images that:
- have `categories_id = 10` directly, or
- belong to an album whose album’s `parent_id` is `10`.

```bash
curl -X GET "https://yourdomain.com/api/v1/media/images?categories_id=10" \
  -H "Authorization: Bearer <token>"
```

#### 2. Fetch Videos from Several Main Categories at Once

```bash
curl -X GET "https://yourdomain.com/api/v1/media/videos?categories_id[]=2&categories_id[]=7" \
  -H "Authorization: Bearer <token>"
```

#### 3. Fetch Files from a Specific Category including the Filelist

```bash
curl -X GET "https://yourdomain.com/api/v1/media/files?categories_id=15&include=filelist" \
  -H "Authorization: Bearer <token>"
```

---

### Benefits and Added Value

- **Reduces the number of API calls:** developers no longer need to first fetch all sub‑categories and then make separate calls for each.
- **Better consistency with UI logic:** in most media management applications, clicking on a main category displays content from all sub‑sections.
- **Increased flexibility:** array support allows combining unrelated category groups in a single request.
- **Full backward compatibility:** if you pass a single value for `categories_id`, the old (direct) behaviour still works, and the indirect results are added (which enhances compatibility).

---

### Upgrade Requirements (from 1.0.1 to 1.0.2)

1. **Update the code**:
   - Replace the following controller files:
     - `plugins/nano/mediaapi/APIControllers/Images.php`
     - `plugins/nano/mediaapi/APIControllers/Videos.php`
     - `plugins/nano/mediaapi/APIControllers/Files.php`

2. **Update the documentation** (optional but recommended):
   - Replace the `Docs-MediaApi-en.md` (or any API documentation file) with the updated version that explains the new filtering behaviour.

3. **Run any necessary commands** (no database migrations):
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

4. **Test the functionality**:
   - Ensure that `GET /api/v1/media/images?categories_id=X` returns images from albums whose `parent_id` equals `X`.
   - Test passing an array: `categories_id[]=1&categories_id[]=2`.
   - Test the same for `videos` and `files` endpoints.

---

### Conclusion

Version **1.0.2** of `Nano.MediaApi` is an important step towards improving the developer and end‑user experience when dealing with hierarchically classified media. By supporting parent categories and allowing arrays, the plugin has become more intelligent and flexible, while maintaining full backward compatibility. We recommend that all `Nano.MediaApi` projects upgrade to this version to benefit from the natural and expected filtering behaviour.

---

**Reference documentation**:
- [Media Library API Documentation (MediaApi)](./docs/MediaApi/Docs-MediaApi-en.md)
- [Media Library API Short Documentation (MediaApi)](./docs/MediaApi/Docs-MediaApi-Short-en.md)
- [Comprehensive Practical Examples – MediaApi Plugin](./docs/MediaApi/Docs-MediaApi-Examples-en.md)
- [Short Practical Examples – MediaApi Plugin](./docs/MediaApi/Docs-MediaApi-Short-Examples-en.md)

