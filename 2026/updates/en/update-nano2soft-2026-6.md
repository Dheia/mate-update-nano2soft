# Update 2026-06

**June month updates**

## 2026-06-01

**Update to `Nano2.ProposalsApi` – Version 1.0.3**

### Support for Bypassing the `User()` Filter in `getRecords` to Enable Parents to View Their Children’s Proposals / Complaints / Reports

---

### Summary of Updates

Version **1.0.3** of the `Nano2.ProposalsApi` plugin introduces an important improvement in the mechanism for fetching records (proposals, complaints, reports) belonging to the current user. Previously, the `User($user)` filter was applied mandatorily in the `getRecords()` function, which prevented parents from seeing records related to their children (students) even when `target_type` and `target_id` were correctly passed.

**Key changes:**
- Added a new parameter `is_stop_user` (boolean) to `Proposals@getRecords`.
- When `is_stop_user=1` and the current user type is a student or parent, the `User()` filter is completely bypassed.
- Allows a parent to fetch all records related to their child by passing the child’s `target_type` and `target_id`.
- Updated the documentation (`docs.md`) with illustrative examples (1.8.1, 1.8.2, 1.8.3) explaining how to use `is_stop_user` with different record types.

---

### Release Objectives

- **Enable parents**: give parents access to their children’s proposals, complaints, and reports via the API without requiring complex additional permissions.
- **Filtering flexibility**: provide a straightforward way to bypass the `User()` filter when needed (for reports and complaints targeting specific students).
- **Maintain security**: the filter is bypassed only for `student` and `parent` user types and only when `is_stop_user=1` is explicitly passed.
- **Backward compatibility**: the default behaviour (`is_stop_user=0` or omitted) remains unchanged (the `User()` filter is applied).

---

### New Features and Improvements

#### 1. Added `is_stop_user` Parameter to `getRecords`

**Before version 1.0.3:**
```php
$posts = $posts->User($user);
```

**After the update:**
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

#### 2. Logic of `is_stop_user`

- **Allows bypassing the `User()` filter** if the following conditions are met:
  - `is_stop_user=1` is passed in the request.
  - `target_type` exists and refers to a student model (`Tss\Student\Models\Student` or `student`).
  - The current user is of type `student` or `parent` (parent).

- **Not allowed** for other user types (e.g., supervisor, admin) or when the correct `target_type` is missing.

- The primary goal: enable a parent to view their child’s records by passing `target_type=...Student` and `target_id=child_id`.

#### 3. New Illustrative Examples in the Documentation

The following examples were added to the `docs.md` file:

- **1.8.1** – Fetch proposals submitted to a specific student.
- **1.8.2** – Fetch complaints submitted against a specific student.
- **1.8.3** – Fetch reports submitted about a specific student.

All examples use `is_stop_user=1` and assume the current user is the parent of that student.

---

### Practical Examples of the New Usage

#### 1. Fetch Proposals Targeting a Specific Student (by parent)

```bash
curl -X GET "https://yourdomain.com/api/v1/proposals/proposals?type=proposals&target_type=Tss\\Student\\Models\\Student&target_id=8&is_stop_user=1" \
  -H "Authorization: Bearer <token>"
```

#### 2. Fetch Complaints Submitted Against a Specific Student

```bash
curl -X GET "https://yourdomain.com/api/v1/proposals/proposals?type=complaints&target_type=Tss\\Student\\Models\\Student&target_id=8&is_stop_user=1"
```

#### 3. Fetch Reports Filed About a Specific Student

```bash
curl -X GET "https://yourdomain.com/api/v1/proposals/proposals?type=reports&target_type=Tss\\Student\\Models\\Student&target_id=8&is_stop_user=1"
```

---

### Upgrade Requirements (from 1.0.2 to 1.0.3)

1. **Update the code**:
   - Replace the controller file:
     `plugins/nano2/proposalsapi/APIControllers/Proposals.php`

2. **Update the documentation (optional but recommended)**:
   - Replace the `docs.md` file with the version that contains the new examples (1.8.1 – 1.8.3).

3. **Run any necessary commands** (no database migrations):
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

4. **Test the functionality**:
   - Ensure that a parent user can fetch the associated student’s records when passing `is_stop_user=1`.
   - Ensure that a regular user (neither student nor parent) cannot bypass the `User()` filter even with `is_stop_user=1`.
   - Ensure that the default behaviour (without `is_stop_user`) has not changed.

---

### Benefits and Added Value

- **Improved parent experience**: parents can now follow all interactions (proposals, complaints, reports) related to their children through a single API.
- **Flexibility for application developers**: they no longer need to build complex solutions to aggregate data from multiple accounts.
- **Tight security**: the filter bypass is only allowed for explicitly authorised cases, preventing data leakage.
- **Full backward compatibility**: all existing applications that do not use `is_stop_user` will work exactly as before.

---

### Conclusion

Version **1.0.3** of `Nano2.ProposalsApi` is an important step towards supporting multi‑user scenarios (parent – student) in the proposal, complaint, and report management system. By adding the `is_stop_user` parameter, we have achieved an optimal balance between security and flexibility, allowing parents to view their children’s data without compromising data safety. We recommend that all `ProposalsApi` projects that deal with student and parent accounts upgrade to this version.

---

**Reference documentation**:
- [Documentation of `Nano2.ProposalsApi`](./docs/ProposalsApi/Docs-ProposalsApi-en.md)
- [Proposals API Short Documentation (Nano2.ProposalsApi)](./docs/ProposalsApi/Docs-ProposalsApi-Short-en.md)
- [Comprehensive Practical Examples – ProposalsApi](./docs/ProposalsApi/Docs-ProposalsApi-Examples-en.md)
- [Nano API Plugin Development Guide (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)

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

## 2026-05-30 – 2026-06-03

**Update to the `Tss.Studyyear` Plugin – Versions 1.0.6 and 1.0.7**  
**and Update to the `Nano.StudyyearApi` Plugin – Version 1.0.1**

### Adding the `is_published_results` Column and Supporting Publishing and Advanced Filtering for Academic Years, Semesters, and Months

---

### Summary of Updates

Two consecutive updates have been made to the `Tss.Studyyear` plugin (versions 1.0.6 and 1.0.7) along with an update to the `Nano.StudyyearApi` plugin (version 1.0.1) with the following objectives:

- **Add a new column** `is_published_results` to the academic year, semester, and month tables.
- **Enable control over data publishing** via the `is_published`, `published_at`, and `unpublished_at` fields in the backend administration interface.
- **Provide advanced filters** for published, unpublished, and result‑published records.
- **Support these fields in the API** so that external applications can benefit from the publication status.

These updates come in response to the need for schools and educational centres to dynamically control the visibility of academic years, semesters, and months to users (especially in results and reporting applications).

---

### Release Objectives

- **Add flexibility in publishing data**: Administrators can determine whether an academic year, semester, or month is published on the site or available for API queries.
- **Separate basic data publishing from results publishing**: Using the new `is_published_results` field, the visibility of report and results data for students and parents can be controlled independently.
- **Support date‑based publishing**: Through `published_at` and `unpublished_at`, publication can be scheduled to start and end automatically.
- **Improve application performance**: By providing advanced filters in both the backend and the API, the amount of transmitted data is reduced.
- **Compatibility with environment variables**: Allow disabling the expiry date check (Check Date) for each entity individually via environment variables.

---

### New Features and Improvements

#### 1. Adding the `is_published_results` Column to the Tables

A new migration file (`builder_table_add_is_published_results_columns.php`) has been created to add a `boolean` column named `is_published_results` to the following tables:

- `tss_studyyear_periods`
- `tss_studyyear_semsters`
- `tss_studyyear_months`

**Meaning of the column:**  
Determines whether this entity’s data is allowed to be published in a “results” context (e.g., appearing to students in a results application). It can be used independently of `is_published`.

#### 2. Supporting Publishing Fields in Forms (`fields.yaml`)

The `fields.yaml` files for `Periods`, `Semsters`, and `Months` have been updated to include the following fields under the `options` tab:

| Field | Type | Description |
|-------|------|-------------|
| `is_published` | Switch | Publish the item on the site / API |
| `published_at` | Datepicker | Publication start date (optional) |
| `unpublished_at` | Datepicker | Publication end date (optional) |
| `is_published_results` | Switch | Publish the item in report results |

A `trigger` logic has been added so that the date fields appear only when `is_published` is enabled.

#### 3. Supporting the Fields in Lists (`columns.yaml`)

The following columns have been added to the `columns.yaml` files for each entity:

- `is_published_results` (text / key)
- `is_published` (text / key)
- `published_at` (datetime)
- `unpublished_at` (datetime)

They are set to `invisible: true` by default to keep the list simple, but can be shown as desired.

#### 4. Adding New Scopes in the Models

The following scopes have been added to the `Period`, `Semster`, and `Month` models:

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
    // Logical negation of isPublished
}
```

The `scopeIsPublishedResults` scope has also been added to check `is_published_results`.

#### 5. Advanced Filters in the Backend Interface (`config_filter.yaml`)

The `config_filter.yaml` files for `Periods`, `Semsters`, and `Months` have been updated with the following filters:

| Filter Name | Type | Description |
|-------------|------|-------------|
| `is_published` | Checkbox | Shows only published records (uses `scopeIsPublished`) |
| `is_not_published` | Checkbox | Shows only unpublished records (uses `scopeIsNotPublished`) |
| `is_published_results` | Switch | Filters based on `is_published_results` (0/1) |
| `is_default` | Switch | Filters default records |
| `is_active` | Switch | Filters active records |

Additionally, a `departments_id` (group) filter, `study_year_id`, `semsters_id`, and `month_num` filters have been added for the appropriate entities.

#### 6. Environment Variables for Date Validity Checking

The `config.php` file of `Tss.Studyyear` has been updated to:

```php
return [
    'check_date_year' => env('TSS_STUDYYEAR_CHECK_DATE_YEAR', true),
    'check_date_semsters' => env('TSS_STUDYYEAR_CHECK_DATE_SEMSTERS', true),
    'check_date_months' => env('TSS_STUDYYEAR_CHECK_DATE_MONTHS', true),
];
```

These variables control whether the date validity check (`from_date`, `to_date`) is applied when retrieving data. If disabled (false), the `to_date > now()` condition is removed, allowing expired items to be displayed.

#### 7. Update to the `Nano.StudyyearApi` Plugin (Version 1.0.1)

- **Added the `is_published_results` column** to the transformers (`PeriodTransformer`, `SemsterTransformer`, `MonthTransformer`).
- **Added new filters** in the `getRecords` functions for `Periods`, `Semsters`, and `Months`:
  - `is_published_results` (0/1)
  - `is_published` (0/1)
  - `is_default` (0/1)
- **Updated the retrieval logic** in the API so that if the user does not pass `is_published_results`, it is treated as `'*'` or `null` (no filtering).
- **Improved date handling** in the transformers using `formatDate`.

**Example from `Periods.php` (API):**

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

#### 8. Updating Version Files (`version.yaml`)

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

### Usage Examples (Backend)

#### 1. Adding a New Academic Year with Scheduled Publishing

- Go to `Academic Years > Academic Year`.
- Create a new year, e.g., `2026-2027`.
- Enable `Publish on site`.
- Set `Publication start date` = `2026-06-01` and `Publication end date` = `2026-07-15`.
- Enable `Publish results in the application` if you want it to appear in the results app.
- Save.

The year will only appear to front‑end interfaces and the API during the specified period.

#### 2. Using the `is_published` Filter in the Semester List

- In the `Semesters` list, use the `Published` (checkbox) filter to show only published semesters.

#### 3. API Query to Fetch Only Months Published for Results

```bash
GET /api/v1/studyyear/months?study_year_id=5&is_published_results=1&include=period,semster
```

---

### Upgrade Requirements (from previous versions)

#### For those using `Tss.Studyyear` (versions older than 1.0.6)

1. **Update the code**: Replace all changed files (models, controllers, fields.yaml, columns.yaml, config_filter.yaml, config.php, version.yaml).
2. **Run the new migration**:
   ```bash
   php artisan plugin:refresh Tss.Studyyear
   ```
   - The `is_published_results` columns will be added to the three tables.
3. **Re‑assign permissions** (optional): No new permissions are required.
4. **Clear cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

#### For those using `Nano.StudyyearApi` (versions older than 1.0.1)

1. **Update the code**: Replace the files `Periods.php`, `Semsters.php`, `Months.php` and the transformers (`PeriodTransformer`, `SemsterTransformer`, `MonthTransformer`).
2. **Update `version.yaml`**.
3. **Clear cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

**Note:** There are no database changes in `Nano.StudyyearApi`, only code improvements.

---

### Backward Compatibility

- **All existing API endpoints** still work unchanged (the new fields are optional in filters).
- **The backend interfaces** continue to work even if the new fields are not used.
- **Existing data**: The `is_published_results` column will automatically be set to `false` for existing rows (since the migration adds it with `default(false)`).

---

### Benefits and Added Value

- **Dynamic content control**: You can now schedule the appearance of academic years, semesters, and months without manually deleting or disabling them.
- **Separation of general publishing from results publishing**: Using `is_published_results`, you can show data to results applications while keeping it hidden from regular visitors.
- **Reduced server load**: Advanced filters reduce the amount of data retrieved and improve response time.
- **Customisation flexibility**: Date checking can be disabled via environment variables, allowing old years to be displayed in admin panels as needed.

---

### Conclusion

Updates **1.0.6** and **1.0.7** of `Tss.Studyyear` and **1.0.1** of `Nano.StudyyearApi` represent a significant step forward in managing academic years, semesters, and months. By adding the `is_published_results` field and supporting temporary publishing and advanced filters, it is now easy to schedule and precisely control content. We recommend all users upgrade to benefit from these features, especially in educational environments that rely on periodic reports and results.

---

**Reference documentation**:
- [`Tss.Studyyear` plugin documentation](./docs/Studyyear/Docs-Studyyear-en.md)
- [`Nano.StudyyearApi` plugin documentation](./docs/StudyyearApi/Docs-StudyyearApi-en.md)
- [API Plugin Development Guide (Nano-Api-SKILL.md)](./Nano-Api-SKILL.md)

## 2026-06-03 – 2026-06-05

**Update to the `Nano.StudyyearApi` Plugin – Version 1.0.2**

### Complete Restructuring of Controllers According to the `Nano-Api-SKILL.md` Guide, Integration of `AccessManager`, and Advanced Features

---

### Summary of Updates

Version **1.0.2** of the `Nano.StudyyearApi` plugin introduces a complete restructuring of the controllers (`Periods`, `Semsters`, `Months`) to fully comply with the `Nano-Api-SKILL.md` guide. The manual `validationList` function has been removed and replaced by `AccessManager` for centralised permission control. Support for advanced filters (`is_or`, `is_not`, `is_force`, `is_or_null`) has been added via `AdvancedQueryHelper`. The `getRecords` function has been unified to support multiple output options, events, caching, and access scope application (company, department). Smart dependency resolution between the academic year and semester has been added in the `Months` controller, and the `access_scope` settings for frontend users have been corrected.

**Key changes:**
- Removed `validationList` functions and replaced them with `AccessManager::checkByResource` and `checkWithFallback`.
- Added support for advanced filters on all fields using `AdvancedQueryManager::scopeWhereField`.
- Unified the `getRecords` function to support various output options (`is_query`, `is_first`, `is_collection`, `is_paginator`, `is_to_sql`).
- Added events `api.list.extendQueryBefore` and `api.list.extendQuery` with resource‑specific event names.
- Supported caching via `$this->cached()` and `getLastUpdateAt`.
- Applied access scopes (`applyAccessScope`) for company and department.
- Centralised permission settings in `config.php` with environment variables.
- Added smart logic in the `Months` controller to infer `study_year_id` and `semsters_id` (if the user does not provide a semester, the default semester for the year is used).
- Corrected `access_scope` from `'own'` to `'all'` for the `list` operations on `periods`, `semsters`, and `months`, because these are master data records not tied to a specific user.

---

### Release Objectives

- **Standardise development practices** across all API controllers in the Nano ecosystem.
- **Simplify permission management** by centralising settings in `config.php` and using `AccessManager`.
- **Improve security** through advanced restricted filters and permission checks for every operation.
- **Increase flexibility** by supporting multiple output options and extensible events.
- **Facilitate maintenance** by restructuring the code to be consistent with `Nano-Api-SKILL.md`.
- **Improve developer experience** by using `checkWithFallback` to avoid duplicating permission settings for `show`.

---

### New Features and Improvements

#### 1. Replaced `validationList` with `AccessManager`

- **Before version 1.0.2**: each controller had a `validationList` function that manually checked settings, leading to duplicated logic.
- **After the update**: permission checks are performed using `AccessManager::checkByResource` and `checkWithFallback` based on the `config.php` settings.

**Example from `Periods.php`:**
```php
$access = AccessManager::checkByResource('nano.studyyearapi::periods.list', $user);
if (!$access['allowed']) {
    return $this->errorUnauthorized($access['message']);
}
```

#### 2. Support for Advanced Filters (`advanced_filters`)

Full support for `is_or`, `is_not`, `is_force`, `is_or_null` has been added for the main fields using `AdvancedQueryManager::scopeWhereField`. These filters are controlled via the `advanced_filters` settings in `config.php`.

**Example – `calendar_type` filter with `is_or`:**
```php
if ($options['calendar_type'] && $options['calendar_type'] !== '*') {
    $query = AdvancedQueryManager::scopeWhereField(
        $query, 'calendar_type', $options['calendar_type'],
        $options['is_or_calendar_type'], $table,
        $options['is_not_calendar_type'], $options['is_force_calendar_type']
    );
}
```

#### 3. Unified `getRecords` Function

The `getRecords` function has become identical across all controllers and supports:

- **Excluding columns** (`exclude`) to reduce data size.
- **Including relationships** (`custom_with`).
- **Text search** (`q`) with advanced search support (ArPhpHelper).
- **Ordering** (`orderBy`, `orderDirection`) and grouping (`group_by`) with `having`.
- **Output options**: `is_query`, `is_first`, `is_model`, `is_collection`, `is_paginator`, `is_to_sql`.
- **Events**: `api.list.extendQueryBefore` and `api.list.extendQuery` to extend queries.
- **Caching** via `$this->cached()`.

#### 4. Applying Access Scopes (`applyAccessScope`)

Company and department scopes are now automatically applied using `applyAccessScope`, with the ability to easily disable specific scopes:

```php
if (!empty($options['access_result']) && $options['access_result']['allowed']) {
    $query = AccessManager::instance()->applyAccessScope($query, $options['access_result'], [
        'company_field'    => 'companys_id',
        'department_field' => 'departments_id',
    ]);
}
```

#### 5. Improved `show` Function Using `checkWithFallback`

To avoid duplicating permission settings, `show` now uses `checkWithFallback` to fall back to `list` settings if no `show`‑specific settings exist:

```php
$access = AccessManager::checkWithFallback(
    'nano.studyyearapi::periods.show',
    'nano.studyyearapi::periods.list',
    $user
);
```

#### 6. Centralised Settings in `config.php`

All permission and advanced filter settings have been moved to `config.php`, allowing control via environment variables:

```php
'periods' => [
    'list' => [
        'permission' => [
            'is_allow' => env('NANO_STUDYYEARAPI_PERIODS_LIST_IS_ALLOW', true),
            'backend' => [...],
            'frontend' => [
                'allow' => env('NANO_STUDYYEARAPI_PERIODS_LIST_FRONTEND_ALLOW', true),
                'allowed_ref_types' => ['student', 'parent'],
                'access_scope' => 'all',  // corrected from 'own' to 'all'
            ],
            'advanced_filters' => [...],
        ],
    ],
],
```

#### 7. Smart Logic in the `Months` Controller

Advanced handling for year and semester IDs has been added:

- If `study_year_id` is not provided → use the default year (`Period::getPrimary()`).
- If `semsters_id` is not provided but `study_year_id` is provided → fetch the default semester for that year (`Semster::where('study_year_id', $study_year_id)->where('is_default', true)->first()`).
- If `semsters_id` is provided but `study_year_id` is missing → infer `study_year_id` from the semester.
- Allow `semsters_id = '*'` to skip filtering by semester.

#### 8. Corrected `access_scope` for Master Data

In previous versions, the `frontend` settings for `list` used `access_scope = 'own'`, which is inappropriate for master data (years, semesters, months) because these records are not tied to a specific user. They have been corrected to `access_scope = 'all'` to match the nature of the data.

---

### Practical Examples of the New Usage

#### 1. Fetch Published Academic Years with an Advanced Filter

```bash
GET /api/v1/studyyear/periods?is_published=1&calendar_type=years&is_or_calendar_type=false
```

#### 2. Fetch Months for a Specific Year and Semester, with the Option to Ignore the Semester

```bash
GET /api/v1/studyyear/months?study_year_id=3&semsters_id=5
```
Or to fetch all months of a year (without filtering by semester):
```bash
GET /api/v1/studyyear/months?study_year_id=3&semsters_id=*
```

#### 3. Using `is_to_sql` for Debugging

```php
$result = $controller->getRecords([
    'study_year_id' => 3,
    'is_to_sql' => true,
]);
// The SQL query will be printed in trace_log
```

---

### Backward Compatibility

- **All existing endpoints** (routes) have not changed, so any application consuming the API will not be affected.
- **Outputs** remain compatible with the previous structure (with `input_data`, `process_data`, `debug` added only when needed).
- **No new database migrations**.
- **The new configuration is optional**: you can continue using the old settings (without `advanced_filters` or `frontend_resolver`), and the plugin will work with default behaviour.

---

### Upgrade Requirements (from 1.0.1 to 1.0.2)

1. **Update the code**:
   - Replace all controller files (`Periods.php`, `Semsters.php`, `Months.php`) with the new versions.
   - Replace the `config.php` file with the new version, which contains the `permission`, `advanced_filters`, and `frontend_resolver` sections (even if they are empty).

2. **Update the `version.yaml` file**:
   - Add version `1.0.2` as shown at the beginning of this document.

3. **Run migrations** (none).

4. **Clear cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

5. **Test the functionality**:
   - Verify that all endpoints (`periods`, `semsters`, `months`) work as expected.
   - Test different access permissions (backend, frontend student, frontend parent, guest).
   - Verify that advanced filters (`is_or`, `is_not`, `is_force`) work on the fields allowed in `config.php`.

---

### Benefits and Added Value

- **Improved security**: permission scopes are automatically applied and cannot be easily bypassed.
- **High flexibility**: the behaviour of each operation can be configured independently without modifying the code.
- **Extensibility**: thanks to events, other plugins can dynamically modify the queries.
- **Better performance**: with support for caching and excluding unnecessary columns.
- **Better developer experience**: all controllers follow the same pattern, making the code easier to learn and maintain.
- **Better data accuracy**: the smart logic in `Months` ensures that the correct months are fetched even if the user does not provide the semester.

---

### Conclusion

Version **1.0.2** of `Nano.StudyyearApi` represents a significant achievement in standardising API standards within the Nano ecosystem. Thanks to the complete restructuring and integration of `AccessManager`, the plugin is now more secure, maintainable, and extensible. All controllers are now compliant with the `Nano-Api-SKILL.md` guide and offer advanced features such as advanced filters, events, and caching.

We recommend that all developers upgrade to this version and take advantage of the centralised settings in `config.php` to customise the API behaviour according to their needs.

---

**Reference documentation**:
- [`Nano.StudyyearApi` plugin documentation](./docs/StudyyearApi/Docs-StudyyearApi-en.md)
- [`Tss.Studyyear` plugin documentation](./docs/Studyyear/Docs-Studyyear-en.md)
- [API Plugin Development Guide (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)
- [`AccessManager` class documentation](./docs/AuthApi/Docs-AccessManager-en.md)
- [`AdvancedQueryHelper` class documentation](./docs/querybuilder/Docs-AdvancedQueryHelper.md)

## 2026-06-03 – 2026-06-05

**Updates to the `Nano.TranslateExtended` plugin – Version 1.0.11**  
**Comprehensive improvement of API translation calls and handling of translatable content**

### Summary of Updates

The `Nano.TranslateExtended` plugin underwent a major update in version **1.0.11** that focused on fixing the issue where translations did not appear when including `translatable_fields` in API responses, and expanding the flexibility of fetching translations by supporting dynamic field and language selection via `Input`, configuration, and public properties. It also improved the `TranslatableContentCaching` behaviour to allow programmatic control over translatable fields, corrected the fallback logic for the default locale, added support for decoding JSON with error logging, and improved compatibility with the `RainLab.Translate` data structure.

---

## Nano.TranslateExtended v1.0.11 – Improved API Translations and Core Fixes

### Release Objectives

- **Fix the issue of translations not appearing** when `translatable_fields` is included in API calls.
- **Provide full flexibility in specifying the fields and languages** for which translations should be fetched via `Input`, configuration, or public properties.
- **Add new `includes`** such as `translatable_attributes` and `translatable_dirty_locales` to support advanced use cases.
- **Improve `TranslatableContentCaching`** by adding dynamic methods to control fields, correcting the fallback logic, and supporting JSON decoding with error logging.
- **Unify the interface for passing fields** as a comma‑separated string (`title,content`) or the word `all`/`*`.
- **Enhance compatibility with `RainLab.Translate`'s translation storage structure.**

### New Features and Improvements

---

#### First: Updates to `DynamicAddIncludeTranslatableApiFields`

##### 1. Enhanced `includeTranslatableFields` method

- **Flexible parameter reading** according to the priority: public property ← configuration ← `Input` ← default value.
- **Support for specifying fields (`fields`) in multiple formats**:
  - Plain array.
  - Comma‑separated string: `"title,content"`.
  - The word `"all"` or `"*"` to fetch all translatable fields.
- **Support for specifying languages (`locales`)** with the same flexibility.
- **Call `$item->transCollectFields()`** before calling `getTranslationsInFormat()` to ensure the field list is updated dynamically.
- **Handle `exclude`** from `Input` or configuration to exclude certain keys from the result.

##### 2. Added new `includes` to the Transformer

- `translatable_attributes`: returns the list of translatable field names in the model (`getTranslatableAttributes`).
- `translatable_dirty_locales` (optionally commented out): returns information about dirty locales and originals.

##### 3. Improved `getConfigValue` method

- Added support for reading from `Input` via the key `translated_fields` as an alternative to `translatable_fields.fields`, providing a simpler way for developers.

##### 4. Example API usage

```json
GET /api/orders/orders/1?include=translatable_fields&translatable_fields.fields=title,content&translatable_fields.locales=en,ar
```

Or using the shortcut:
```json
GET /api/orders/orders/1?include=translatable_fields&translated_fields=title,content
```

---

#### Second: Updates to `TranslatableContentCaching`

##### 1. Added programmatic methods to control fields

- `getTransCachedFields()`: returns the current list of translatable fields.
- `setTransCachedFields($fields)`: dynamically sets the list, supporting comma‑separated strings or `'all'`/`'*'`.

##### 2. Improved `getTranslated` method

- Added the `$useInBackend` parameter (default `false`) to allow translation even in the backend environment when needed.
- Corrected the fallback logic for the default locale: when the requested `$locale` matches the default locale, `$fallback` is set to `true` to ensure fallback to the original value if no translation exists.
- Added a helper function `processTranslatableJsonableValue` with JSON error logging in debug mode.

##### 3. Improved `getTranslatedAttributes` method

- Made the search for the translation more compatible with `RainLab.Translate`’s structure, which stores `locale` inside an `attributes` array.
- Added a fallback to search via the direct property `$item->locale`.

##### 4. Improved `getTranslationsInFormat` method

- Supports passing `$fields` as a comma‑separated string or `'all'`/`'*'`, converting it to an array inside the method.
- Unified logic with the rest of the plugin.

##### 5. Improved dynamic model registration (`registerModel`)

- Added a static function `isPropertyExists` to check for property existence before adding it.
- Avoid re‑adding existing properties.

##### 6. Helper function `isPropertyExists`

- A static function that checks whether a property exists on an object, with support for `propertyExists` (for `October\Rain\Extension\ExtensionBase` extensions).

---

### Third: Updated `version.yaml`

Version `1.0.11` was added with a brief description of the updates:

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

### Application Examples

#### 1. Using the new includes in the API (via inclusion)

**Request to fetch translations for only two fields with one language:**
```http
GET /api/products/products/1?include=translatable_fields&translatable_fields.fields=title,description&translatable_fields.locales=ar
```

**Using the shortcut `translated_fields`:**
```http
GET /api/products/products/1?include=translatable_fields&translated_fields=title
```

**Fetch all translatable fields and exclude a specific field:**
```http
GET /api/products/products/1?include=translatable_fields&translatable_fields.fields=all&translatable_fields.exclude=seo_keywords
```

#### 2. Using `includeTranslatableAttributes` to get only the field names

```http
GET /api/products/products/1?include=translatable_attributes
```

**Response:**
```json
{
    "data": {
        "id": 1,
        "translatable_attributes": ["title", "description", "seo_title"]
    }
}
```

#### 3. Programmatic control in `TranslatableContentCaching`

```php
// Dynamically set the translatable fields
$order = Order::find(1);
$order->setTransCachedFields('title,content,notes');

// Get translations in a custom format
$translations = $order->getTranslationsInFormat(
    fields: 'title',
    format: TranslatableContentCaching::FORMAT_GROUP_BY_FIELD_UNDER_ALL,
    locales: ['en', 'ar'],
    includeOriginal: true,
    useFallback: false
);
```

#### 4. Enabling translation in the backend (special cases)

```php
// Force translation even in the backend environment
$value = $model->getTranslated('title', null, 'ar', true, true, true);
```

---

## Version Summary (1.0.11)

| Version | Key Features |
|---------|---------------|
| **Nano.TranslateExtended 1.0.11** | Fixed the `translatable_fields` issue in the API, added flexible field/language specification, added `translatable_attributes` and `translatable_dirty_locales`, improved `TranslatableContentCaching` with dynamic methods and fallback logic for the default locale, better compatibility with `RainLab.Translate`, JSON error logging. |

---

### Upgrade Requirements

1. **Update the files**:
   - `DynamicAddIncludeTranslatableApiFields.php`
   - `TranslatableContentCaching.php`
   - `version.yaml`

2. **No new migrations**: This version does not require database changes.

3. **Dependencies**:
   - `RainLab.Translate` (latest version)
   - `OctoberCMS` ≥ 2.0 or `WinterCMS`

4. **Optional settings**:
   - You can add custom settings in the plugin’s `config.php` under the key `nano.translateextended::translatable_fields` to set default values for `fields`, `format`, `locales`, `includeOriginal`, `useFallback`, `useCache`, `exclude`.

5. **Compatibility testing**:
   - Test API calls with `include=translatable_fields` and various parameters.
   - Test models using `TranslatableContentCaching` and verify that `getTranslated` works in both frontend and backend.
   - Test JSON decoding for translatable JSON fields.

---

### Conclusion

Version **Nano.TranslateExtended 1.0.11** represents a significant leap in how translations are handled both via the API and in code. Developers no longer need to write complex queries or manually extend the Transformer; they can now use `translatable_fields` with the ability to specify fields and languages directly via `Input`, and obtain translations in a clean, flexible format. Many behavioural issues in `TranslatableContentCaching` have been fixed to ensure translations work as expected in all scenarios, including the default locale and fallback to original values. The code is fully documented and includes performance and compatibility improvements.

---

**Reference documentation**:
- [`Nano.TranslateExtended` plugin documentation](./docs/TranslateExtended/Behaviors/Docs-Nano.TranslateExtended-en.md)
- [`DynamicAddIncludeTranslatableApiFields` behaviour documentation](./docs/TranslateExtended/Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-en.md)
- [`TranslatableContentCaching` behaviour documentation](./docs/TranslateExtended/Behaviors/Docs-TranslatableContentCaching-en.md)
- [Enhanced Translations API Usage Guide](./docs/TranslateExtended/Docs-API-Translations-Guide-en.md)


## 2026-06-01 - 2026-06-06

**Updates to the `Nano2.Qrcodes` Plugin – Version 1.0.3**

### Summary of Updates

Version 1.0.3 introduces a complete system for managing barcodes and codes (1D and 2D) within NanoSoft software (NanoSoft App). Whereas version 1.0.2 was only a preliminary table structure, this version adds:

- A **complete `Barcode` model** with multiple traits supporting scopes, options, caching, default records, and various relationships.
- An **integrated backend controller** supporting `FormController`, `ListController`, `ReorderController`, and `ImportExportController` with fine‑grained permissions.
- A **central `Manager` class** for handling barcodes (create, update, delete, use, verify, statistics) with support for daily/weekly/monthly usage limits via two methods (Cache and Database).
- An **advanced `BarcodeGenerator` class** supporting 7 different barcode generation libraries (Picqer, chillerlan, SimpleSoftwareIO, BaconQrCode, Endroid, Milon, PHP QR Code) with multiple output formats (PNG, SVG, HTML, Base64, etc.).
- A **complete RESTful API** (CRUD, verify, use, statistics, barcode image) with an advanced permission system via `AccessManager`.
- **Import and export support** via CSV files with flexible options for default values (department, owner, product, unit, skip type rules).
- **Comprehensive configuration** via `config.php` including permissions for each operation, usage limits, advanced filters, and a dynamic resolver for Frontend users.

All of this is accompanied by thorough documentation in Arabic and English within the `lang.php` files.

---

### Version 1.0.3 – Integrated Barcode Management System

#### Release Objectives

- **Build a robust `Barcode` model** supporting all barcode types (1D and 2D) with polymorphic relationships (owner, verifier, subject, user) and dynamic JSON fields.
- **Provide a professional backend controller** following OctoberCMS standards and offering an easy‑to‑use interface for managing barcodes.
- **Create a `Manager` class** (Singleton) to unify business logic and reuse it in both the API and the backend.
- **Develop a `BarcodeGenerator`** that is multi‑library, flexible, extensible, and supports various output formats.
- **Build a complete API** with support for advanced filtering (`is_or`, `is_not`, `is_or_null`), usage limits, and permission checks.
- **Enable high‑flexibility import/export** of barcodes, with the ability to set default values and skip validation rules per barcode type.
- **Provide a usage limits system** (usage limits) for users (daily, hourly, weekly, monthly, or a custom number of days) using two methods: Cache (fast) and Database (accurate).

#### New Features and Improvements

##### 1. Integrated `Barcode` Model

A model `Nano2\Qrcodes\Models\Barcode` has been created with the following traits:

- **Basic traits**: `Validation`, `SoftDelete`, `Purgeable`, `Sortable`.
- **Specialised traits**: `HasScopesModel`, `HasDefault`, `HasUserScopes`, `HasUserOptions`, `HasSubjectScopes`, `HasSubjectOptions`, `HasOwnerScopes`, `HasOwnerOptions`, `HasVerifierScopes`, `HasVerifierOptions`, `HasRecordsOptions`, `ListObjects`, `ListOptions`, `FieldsOptions`, `HasCreateDefaultRecords`.
- **Relationships**:
  - `belongsTo` with `Company`, `Department`, `Template`, `Product`, `Unit`, and users (`Created_by`, `Updated_by`, `Deleted_by`).
  - `morphTo` with `owner`, `verifier`, `subject`, `user`.
  - `attachOne` and `attachMany` for files and images.
- **JSON fields**: `fields_data`, `metadata`, `other_data`, `config_data`, `additional_data`.
- **Helper methods**:
  - `isExpired()`, `isExpiringSoon()`, `canUse()`, `use($user)`, `verify($verifier, $score)`.
  - `generateUniqueCode()` – generates a unique code in the format `{companys_id}-{departments_id}-{random_date_time}`.
  - `prepareDuplicate()` – prevents duplication according to settings.
  - `applyBarcodeTypeRules()` – applies precise validation rules for each barcode type (fixed lengths, digits only, even length, etc.).
- **Skip type rules support**: property `skipBarcodeTypeRules` with methods `skipBarcodeTypeRules()`, `enableBarcodeTypeRules()`, `disableBarcodeTypeRules()`, `isBarcodeTypeRulesSkipped()`.
- **Integrated caching system** to improve performance.

##### 2. Backend Controller `Barcodes`

A controller `Nano2\Qrcodes\Controllers\Barcodes` has been created with the following features:

- **Behaviors**:
  - `FormController`, `ListController`, `ReorderController`, `ImportExportController`.
- **Permissions**: `access`, `access_all`, `add`, `edit`, `delete`, `verify`, `use`, `generate`, `print`, `export`, `import`.
- **Basic CRUD methods**: `onCreate`, `onUpdate`, `onDelete`.
- **Bulk actions**:
  - `onActivateSelected` / `onDeactivateSelected` – activate/deactivate a group.
  - `onVerifySelected` – administrative verification of a group.
  - `onUseSelected` – record usage for a group.
- **Batch creation**:
  - `generate` page with a form to specify count, prefix, barcode type, expiry duration, product, department.
  - `onGenerate` function to generate multiple barcodes at once (up to 1000).
- **Printing**:
  - `onPrint` and `onPrintSelected` to open a print window containing the barcode with product data.
- **Export and import**:
  - `onExport` to export barcodes to CSV.
  - Support for `ImportExportController` with `BarcodeImport` and `BarcodeExport` models.
- **Dropdown options**:
  - `getStatusOptions`, `getBarcodeTypeOptions`, `getDepartmentsIdOptions`, `getCompanysIdOptions`, `getProductIdOptions`, `getUnitIdOptions`, `getCurrencysIdOptions`, etc.
- **Default records**: `index_onCreateDefaultRecords` and `index_onRestDefaultRecords`.
- **Lists and settings**: `bootBackendNavigation` and `registerBackendPermissions` functions to register the menu and permissions via events.

##### 3. `Manager` Class (Business Logic)

A class `Nano2\Qrcodes\Classes\Manager` (Singleton) has been created as the sole entity responsible for business logic:

- **CRUD operations**:
  - `createBarcode($options)` – create a barcode with advanced options (skip validation, skip type rules, generate image, etc.).
  - `updateBarcode($id, $options)` – update a barcode, preventing modification of the user if already used.
  - `deleteBarcode($id)` – soft delete.
  - `restoreBarcode($id)` – restore a soft‑deleted barcode.
- **Usage and verification**:
  - `scanAndUseBarcode($barcodeValue, $user)` – scan and use a barcode, checking its validity and user limits.
  - `verifyBarcode($barcodeValue)` – verify validity without using.
  - `verifyBarcodeById($id, $verifier, $score)` – administrative verification.
- **Batch creation**:
  - `generateMultipleBarcodes($count, $baseOptions)` – create several barcodes.
  - `generateBatch($batchOptions)` – create a batch with a unified batch number.
- **Usage limits**:
  - Two tracking methods: `cache` (fast) and `database` (accurate).
  - Flexible periods: `hour`, `day`, `week`, `month`, or a custom number of days.
  - Functions `checkUserUsageLimit`, `incrementUserUsage`, `getUserUsageCountFromDatabase`, `getUsageLimitValue`.
- **Statistics**:
  - `getBarcodeStats($options)` – general statistics (total, active, expired, by type, etc.).
  - `getUserBarcodeUsageStats($user)` – user usage statistics.
- **Helper operations**:
  - `getBarcodeRecords($options)` – retrieve records with filtering options.
  - `exportBarcodes($options)` – export data.
  - `importBarcodes($data, $options)` – import data.
  - `getBarcodeImageUrl($barcode)` – get the barcode image URL.
  - `formatBarcodeForApi($barcode)` – format data for the API.

##### 4. `BarcodeGenerator` Class (Barcode Generation)

An advanced class has been created that supports seven different libraries with automatic detection:

| Library | Supported Types | Priority |
| :--- | :--- | :--- |
| Picqer\Barcode | 1D & 2D | 1 (highest) |
| chillerlan\QRCode | QR only | 2 |
| SimpleSoftwareIO\QrCode | QR only | 3 |
| PHP QR Code (`qrcode` function) | QR only | 4 |
| BaconQrCode | QR only | 5 |
| Endroid\QrCode | QR only | 6 |
| Milon\Barcode (Tss) | 1D & 2D | 7 |
| Fallback (GD) | plain text | last |

**Features**:
- **Output formats**: `png`, `jpg`, `jpeg`, `webp`, `svg`, `html`, `base64`, `data-uri`, `gd`, `response`, `file`.
- **Advanced options**: width, height, colour, background colour, margin, error correction level, quality, transparency, caching.
- **Special functions**:
  - `generate($data, $type, $options)` – main function.
  - `generateBarcodeImage($data, $type, $options)` – generate a PNG image.
  - `generateAndSaveBarcodeImage($barcode, $path)` – save the image to storage and update the model.
  - `validateBarcode($barcode, $type)` – validate a barcode value according to its type (supports EAN, UPC, ISBN, Code 128/39/93, I25, POSTNET, etc.).
- **Helper functions**: `hexToRgb`, `hexToGdColor`, `getAvailableLibraries`, `selectLibrary`.

##### 5. RESTful API

An API controller `Nano2\Qrcodes\ApiControllers\Barcodes` has been created with the following endpoints:

| Method | Path | Description |
| :--- | :--- | :--- |
| GET | `/barcodes` | Fetch a list of barcodes with filtering and pagination. |
| GET | `/barcodes/activelystats` | Latest update timestamp (for caching). |
| POST | `/barcodes` | Create a new barcode. |
| PUT | `/barcodes/{id}` | Update an existing barcode. |
| DELETE | `/barcodes/{id}` | Soft‑delete a barcode. |
| GET | `/barcodes/{id}` | View barcode details. |
| POST | `/barcodes/verify` | Verify a barcode without using it. |
| POST | `/barcodes/use` | Scan and use a barcode (record usage). |
| POST | `/barcodes/verify/{id}` | Administrative verification (set `is_verified`). |
| POST | `/barcodes/generate` | Generate a batch of barcodes (admin only). |
| GET | `/barcodes/stats` | General barcode statistics. |
| GET | `/barcodes/user-stats` | Statistics for the current user. |
| GET | `/barcodes/image/{id}` | Get the barcode image (PNG). |

**Permission system**:
- Each operation (`list`, `show`, `create`, `update`, `delete`, `verify`, `use`, `generate`, `stats`, `userStats`) has independent permission settings in `config.php`.
- Supports advanced filters (`is_or`, `is_not`, `is_or_null`) via `advanced_filters`.
- Dynamic frontend resolver to automatically populate `user_id` and `user_type` for frontend users.

**Responses**:
- Follow the `Nano\API\Classes\ApiController` structure with fields: `code`, `status`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`.

##### 6. Import and Export

Two models have been added:

- **`BarcodeImport`**:
  - Supports options: `update_existing`, `skip_barcode_type_rules` (three states: `null`, `true`, `false`), `default_status`, `default_barcode_type`, `default_departments_id`, `default_owner_type` and `default_owner_id`, `default_product_id` and `default_product_name`, `default_unit_id`.
  - Option functions to build the import interface (`getSkipBarcodeTypeRulesOptions`, `getDefaultDepartmentsIdOptions`, etc.).
  - `importData` function that processes each row and calls `Manager::createBarcode` or `updateBarcode` while passing the `skip_barcode_type_rules` option.
- **`BarcodeExport`**:
  - Exports data according to the columns defined in `columns.yaml`, with optional filtering based on user permissions.

##### 7. Central Configuration (config.php)

A rich `config.php` file has been created:

- **General settings**:
  - `allow_debug_any` – enable debug mode via GET parameters.
  - `api.enable_cache` and `api.cache_ttl` – enable API caching.
- **`barcodes` settings**:
  - `is_default_company`, `department_type`, `is_check_duplicate`, `is_show_create_default`, `is_stop_show_menu`, etc.
  - `image_width`, `image_height`, `image_color`, `image_format`, `storage_disk` – barcode image settings.
  - **`usage_limits`**:
    - `enabled`, `daily_limit`, `hourly_limit`, `reset_at_midnight`.
    - `tracking_type` (`cache` or `database`).
    - `tracking_period` (`hour`, `day`, `week`, `month`, or a number of days).
    - `tracking_custom_days`, `use_soft_limit`, `limit_message`.
  - **Permission settings for each operation** (`list`, `show`, `create`, `update`, `delete`, `verify`, `use`, `generate`, `stats`, `userStats`):
    - `permission.is_allow`, `permission.backend`, `permission.frontend`, `permission.guest`.
    - `advanced_filters` and `frontend_resolver` for each operation (customisable via environment variables).

##### 8. Translation and Multi‑language Support

`lang/ar/lang.php` and `lang/en/lang.php` files have been added covering:

- Icons, menus, permissions, messages, errors, filters, form fields, list columns, import/export helpers, etc.
- Special keys for `skip_barcode_type_rules` options in the import section.
- Translation of multiple barcode types used in `getBarcodeTypeOptions`.

---

### Practical Examples

#### 1. Create a Barcode via API

```bash
curl -X POST "https://domain.com/api/v1/qrcodes/barcodes" \
  -H "Content-Type: application/json" \
  -d '{
    "barcode": "5901234123457",
    "barcode_type": "EAN13",
    "product_name": "Test Product",
    "price": 99.99
  }'
```

#### 2. Scan and Use a Barcode

```bash
curl -X POST "https://domain.com/api/v1/qrcodes/barcodes/use" \
  -H "Content-Type: application/json" \
  -d '{"barcode": "BC000101"}'
```

#### 3. Import with Skip Type Rules (in the backend)

In the barcode import form, choose the value `true` from the `skip_barcode_type_rules` dropdown if you are importing EAN‑13 barcodes that are not exactly 13 digits long, or `false` to apply the rules, or `null` for the default setting.

#### 4. Use `Manager` Directly

```php
use Nano2\Qrcodes\Classes\Manager;

// Create a new barcode
$result = Manager::createBarcode([
    'barcode' => '1234567890',
    'barcode_type' => 'C128',
    'product_name' => 'New device',
]);

if ($result['status']) {
    $barcode = $result['model'];
    echo $barcode->code;
}

// Use a barcode
$useResult = Manager::scanAndUseBarcode('1234567890', $currentUser);

// User statistics
$stats = Manager::getUserBarcodeUsageStats($currentUser);
```

#### 5. Generate a Barcode Image Using `BarcodeGenerator`

```php
use Nano2\Qrcodes\Classes\BarcodeGenerator;

$pngData = BarcodeGenerator::generateBarcodeImage('5901234123457', 'EAN13');
header('Content-Type: image/png');
echo $pngData;
```

---

### Upgrade Requirements

1. **Update the database**  
   Run the new migrations (if not already run):

   ```bash
   php artisan october:migrate
   ```

   This will create the `nano2_qrcodes_barcodes` table (it may already exist from version 1.0.2).

2. **Update the code**  
   Replace the following files with the new versions:
   - `models/Barcode.php` (with all traits in the `barcode/` folder).
   - `controllers/Barcodes.php` (backend).
   - `apicontrollers/Barcodes.php` (API).
   - `transformers/BarcodeTransformer.php`.
   - `classes/Manager.php`.
   - `classes/BarcodeGenerator.php`.
   - `classes/QrcodeManagement.php` (if present).
   - `config/config.php`.
   - `routes.php`.
   - `updates/version.yaml`.
   - Language files `lang/ar/lang.php` and `lang/en/lang.php`.
   - YAML files for the model (`models/barcode/fields.yaml`, `columns.yaml`, etc.).
   - View and configuration files for the backend controller (`controllers/barcodes/*.htm`, `config_*.yaml`).

3. **Register menus and permissions**  
   Ensure that `Plugin.php` calls `Barcodes::registerBackendPermissions()` and `Barcodes::bootBackendNavigation()` inside the `boot()` function.

4. **Set environment variables (optional)**  
   You can place the following variables in the `.env` file to customise behaviour:

   ```ini
   NANO2_QRCODES_BARCODES_USAGE_LIMITS_ENABLED=true
   NANO2_QRCODES_BARCODES_USAGE_LIMITS_DAILY_LIMIT=10
   NANO2_QRCODES_BARCODES_USAGE_LIMITS_TRACKING_TYPE=database
   NANO2_QRCODES_BARCODES_LIST_BACKEND_ALLOW=true
   NANO2_QRCODES_API_ENABLE_CACHE=false
   NANO2_QRCODES_BARCODE_IMAGE_WIDTH=300
   ```

5. **Test the functionality**  
   - Verify that the `Barcodes` menu appears in the control panel.
   - Try creating, editing, and deleting a barcode.
   - Test batch creation, printing, and export.
   - Test API endpoints using Postman or `curl`.
   - Ensure that usage limits work as required (can be tested via the `use` endpoint).

6. **Clear cache**  
   To apply the new settings:

   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### Conclusion

Version 1.0.3 is a foundational, integrated release for barcode management on the NanoSoft platform. It provides a rich model, a complete backend controller, a modern API, a flexible `Manager` class, and a multi‑library barcode generator. It also supports flexible import/export and an advanced usage limits system that can be customised according to business requirements.

Thank you for your attention, and we welcome your feedback and suggestions to improve the plugin in future releases.

---

**Reference documentation**

- [General plugin documentation](./docs/Qrcodes/Docs-Nano2-Qrcodes-en.md)
- [`Barcode` model documentation](./docs/Qrcodes/Docs-Barcode-Model-en.md)
- [`Manager` class documentation](./docs/Qrcodes/Docs-Manager-Class-en.md)
- [`BarcodeGenerator` class documentation](./docs/Qrcodes/Docs-BarcodeGenerator-Class-en.md)
- [API documentation](./docs/Qrcodes/Docs-API-Documentation-en.md)

## 2026-06-07 - 2026-06-08

### Update Version: Tss.Webbasic (v1.0.15) and Nano.BasicApi (v1.0.22)

**Developer:** Dheia Al-Shami  

---

### Update Summary

A new `repeater` field named `app_links` has been added to the theme settings to manage application links (such as Google Play Store, App Store, etc.). This field is also supported in the `Nano.BasicApi` API endpoint, so the field value is returned with the `image` field processed into a full URL using `MediaLibrary`.

---

### Changes in Tss.Webbasic (v1.0.15)

#### 1. Adding the `app_links` Field to the Theme Settings Interface

- **Location:** `CmsManager.php` (`extendCmsThemeConfig` function)
- **Field Type:** `repeater`
- **Subfields:**
  - `name` (text, required)
  - `link` (text, required)
  - `icon` (Awesome Icons)
  - `image` (media file via MediaFinder)
  - `is_active` (switch)

- **Tab:** `tss.webbasic::lang.theme.tab.shop`

#### 2. Adding Translation Keys (lang)

New keys have been added in `lang/ar/lang.php` and `lang/en/lang.php`:

##### Arabic (`ar`)

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
        'shop' => 'إعدادات المتجر', // if not already present
    ],
],
```

##### English (`en`)

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
        'shop' => 'Shop Settings', // if not already present
    ],
],
```

#### 3. Updating the `version.yaml` File

```yaml
1.0.15:
    - 'Add app_links repeater field to theme settings with translation support'
    - 'Add Arabic and English translations for app_links fields'
```

---

### Changes in Nano.BasicApi (v1.0.22)

#### 1. Supporting the `app_links` Field in the API

- **Modified File:** `APIControllers/Themes.php`
- **Function:** `index()`

##### a. Adding `app_links` Processing When Present in Theme Settings

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

##### b. Updating the `getUrlMediaLibrary` Function

The function already exists and converts a media path to a full URL. It has not been modified but is used to process images in `app_links`.

#### 2. Updating the `version.yaml` File

```yaml
1.0.22:
    - 'Add support for app_links field in theme settings API endpoint'
    - 'Process image field in app_links using MediaLibrary URL'
```

---

### How to Use

#### In the Administration Panel (Theme Settings)

1. Go to `Customize → Theme` (or `Settings → Theme` depending on language).
2. Go to the **"Shop Settings"** tab.
3. You will find a new field named **"App Links"**.
4. You can add multiple links, each containing:
   - Name (required)
   - URL (required)
   - Icon (optional)
   - Image (optional)
   - Active/Inactive toggle

#### Via API

- **Endpoint:** `GET /api/v1/basic/settings`
- **Response:** `app_links` will appear as an array of objects, with the `image` field processed into a full URL.

**Example Response:**

```json
{
    "app_links": [
        {
            "name": "Google Play Store Download Link",
            "link": "https://play.google.com/store/apps/details?id=com.nano2soft.now",
            "icon": "fab fa-google-play",
            "image": "https://example.com/storage/app/media/GooglePlay.svg",
            "is_active": "1"
        }
    ]
}
```

---

### Compatibility

- **Requires** `GinoPane.AwesomeSocialLinks` to support the `awesomeiconslist` field type (optional; if not present, an error will appear in the interface).
- **Requires NanoSoft App v2.x or v3.x**
- **Minimum PHP version:** 7.2

---

### Notes for Developers

- The `app_links` field is stored as JSON in the `ThemeData` model, so no new database table is needed.
- To ensure the field works properly, make sure `app_links` is added to the `jsonable` array in the `ThemeData` model (can be added via `extend` in `boot()`).
- The same image processing method used for `social_links` has been applied.

---

### Previous Version History

- **Tss.Webbasic v1.0.14:** Support for SiteSearch on posts and projects content.
- **Nano.BasicApi v1.0.21:** (if a previous version exists)

---

### Suggested Update for Existing Projects

1. Update the plugins using the command:
   ```bash
   php artisan plugin:refresh Tss.Webbasic
   php artisan plugin:refresh Nano.BasicApi
   ```
2. Clear the cache:
   ```bash
   php artisan cache:clear
   ```
3. If you are using your own theme, ensure that the theme settings interface supports the new field.

---

**Updated by:** Dheia Al-Shami  
**For inquiries:** info@nano2soft.com

## 2026-06-05 - 2026-06-09

**Update to `Nano.Orders` – Version 2.2.12 and `Nano.OrdersApi` – Version 1.0.22**

### Update to `Nano.Orders` – Version 2.2.12  

### Update to `Nano.OrdersApi` – Version 1.0.22

---

### Summary of Updates

Version **2.2.12** of `Nano.Orders` added full support for managing photos associated with orders, including four new `attachMany` relationships and the `StepPhotos` trait, which provides professional functions for uploading, deleting, and checking user permissions (owner, delivery, admin) based on the current order status. The system was also integrated with `Nano.FileUpload` to unify the file upload process, added flexible settings (`photo_rules`) to control each field, fired new events, and introduced the `skip_photo_check` option in `updateOrderStatusAdvanced` to prevent status changes without the required photos.

In parallel, version **1.0.22** of `Nano.OrdersApi` provided three API endpoints for uploading (single/multiple) and deleting photos, fully integrated with `StepPhotos` and permission checks, returning photo URLs and thumbnails in the response.

---

## Nano.Orders v2.2.12 – Order Photo Management (StepPhotos)

### Release Objectives

- **Enable uploading photos** before and after delivery by both the lessee (`user_id`) and the lessor (`delivery_user_id`).
- **Link photos to order statuses** such as `NEW`, `PROCESSING`, `DELIVERY`, `COMPLETE` to ensure they are uploaded at the appropriate time.
- **Prevent changing the order status** (`DELIVERY` or `COMPLETE`) unless the required photos have been uploaded.
- **Provide flexible functions** that rely on `Nano.FileUpload` and support regular files or base64.
- **Fire events** to extend behaviour (notifications, logging, etc.).
- **Allow developers to customise rules** (allowed statuses, roles, maximum number of photos, allowed file types) via configuration.

### New Features

#### 1. `attachMany` relationships in the `Order` model

Four new relationships were added inside `Order.php`:

```php
public $attachMany = [
    'user_before_delivery'     => 'System\Models\File',
    'user_after_delivery'      => 'System\Models\File',
    'delivery_before_delivery' => 'System\Models\File',
    'delivery_after_delivery'  => 'System\Models\File',
];
```

| Relationship | Description | Authorised User |
|--------------|-------------|-----------------|
| `user_before_delivery` | Lessee photos before delivery | Owner (`user_id`) |
| `user_after_delivery`  | Lessee photos after delivery | Owner |
| `delivery_before_delivery` | Lessor photos before delivery | Delivery person (`delivery_user_id`) |
| `delivery_after_delivery`  | Lessor photos after delivery | Delivery person |

#### 2. Registering fields in `FileUploadRegistry`

These fields were automatically registered in `Plugin.php` via the `registerOrderFileUploadFields` function, with the following settings:

- Field type: `image` (only images supported)
- Maximum size: 5 MB (adjustable)
- Allowed types: `jpg, jpeg, png, gif`
- `multiple => true` (because they are `attachMany` relationships)
- `max_files => 10` (allowed number of photos)
- `is_public => false` (private, accessed via temporary link)

#### 3. `StepPhotos` trait (file `traits/steps/StepPhotos.php`)

This trait contains all photo management functions:

| Function | Description |
|----------|-------------|
| `uploadOrderPhoto($field, $fileData, array $options)` | Upload a single photo (supports `UploadedFile` or base64). |
| `uploadOrderPhotos($field, array $filesData, array $options)` | Upload multiple photos at once. |
| `deleteOrderPhoto($field, $fileId, array $options)` | Delete an existing photo. |
| `validatePhotoUploadPermission($field, $user, $order, $newStatus, array $options)` | Check user permission for upload (role, allowed status, maximum count). |
| `canUploadPhoto($field, $user, $order)` | Quick check without exception. |
| `canUploadPhotoToField($field, $user, $order, $newStatus)` | Same as above with support for new status. |
| `enforcePhotoRequirementsOnStatusChange($oldStatus, $newStatus, $order, $actor, array $options)` | Called before status change to ensure required photos exist. |
| `getOrderPhotoUrls($field, array $thumbSizes, $includeModel)` | Fetch photo URLs with custom thumbnails. |
| `hasRequiredPhotosForStatus($newStatus, $order, $role)` | Quick check for requirements. |

**Permission validation mechanism (`validatePhotoUploadPermission`):**
- Determines the user’s role (`user` / `delivery` / `admin`) relative to the order.
- Reads the rules (`allowed_roles`, `allowed_statuses`, `max_files`) from settings or passed options.
- Checks that the field allows this role, that the current order status is allowed for upload, and that the current number of photos has not reached the maximum.
- If `$newStatus` is passed, checks whether the field is required for transitioning to that status.

#### 4. Support for `skip_photo_check` in `updateOrderStatusAdvanced`

The `updateOrderStatusAdvanced` function (in the `StepStatus` trait) was modified to support two new options:

- `skip_photo_check` (bool, default `false`) – bypass photo check.
- `photo_validation_rules` (array) – custom rules to override the default rules.

When attempting to change the status, the function extracts the actual new status, then calls `enforcePhotoRequirementsOnStatusChange` unless `skip_photo_check = true`. If required photos are missing, an exception is thrown and the change is prevented.

#### 5. `photo_rules` settings in `config/nano/orders.php`

A complete section was added that can be customised via environment variables or direct editing:

```php
'photo_rules' => [
    'user_before_delivery' => [
        'allowed_statuses' => ['NEW', 'PROCESSING'],
        'allowed_roles'    => ['user', 'admin'],
        'required_on_transition' => ['DELIVERY'],
        'max_files'        => 10,
        'allowed_types'    => 'jpg,jpeg,png,gif',
        'max_size'         => 5120,
        'description'      => 'Lessee photos before delivery',
    ],
    // ... remaining fields
],
```

Corresponding environment variables:
```ini
NANO_ORDERS_PHOTO_USER_BEFORE_STATUSES="NEW,PROCESSING"
NANO_ORDERS_PHOTO_USER_BEFORE_ROLES="user,admin"
NANO_ORDERS_PHOTO_USER_BEFORE_REQUIRED="DELIVERY"
NANO_ORDERS_PHOTO_USER_BEFORE_MAX_FILES=10
NANO_ORDERS_PHOTO_USER_BEFORE_ALLOWED_TYPES="jpg,jpeg,png,gif"
NANO_ORDERS_PHOTO_USER_BEFORE_MAX_SIZE=5120
```

#### 6. New Events

- `nano.orders.photo.uploaded` – fired after a photo is successfully uploaded, passes `order`, `field`, `file`, `actor`.
- `nano.orders.photo.deleted` – fired after a photo is successfully deleted, passes `order`, `field`, `file_id`, `actor`.

#### 7. Translation keys

New keys were added under `manager.photo` and `orders.photo` in the language files (`lang/ar/lang.php`, `lang/en/lang.php`) to cover success, error, and validation messages.

---

## Nano.OrdersApi v1.0.22 – API Endpoints for Photo Management

### Release Objectives

- **Provide an API for uploading and deleting photos**, following the same style as `update-status`.
- **Fully integrate with `StepPhotos` and `OrderManager`** to ensure the same permission and status rules are applied.
- **Support file uploads via `multipart/form-data` or `base64`**.
- **Return photo and thumbnail URLs** directly in the response to facilitate display in client applications.

### New Features

#### 1. Three Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/orders/orders/upload-photo` | Upload a single photo. |
| `POST` | `/api/v1/orders/orders/upload-multiple-photos` | Upload multiple photos (array). |
| `POST` | `/api/v1/orders/orders/delete-photo` | Delete a photo (accepts `order_id`, `field`, `file_id`). |
| `DELETE` | `/api/v1/orders/orders/delete-photo/{orderId}/{field}/{fileId}` | Delete a photo via path parameters (alternative). |

#### 2. Controller Functions in `Orders`

The following functions were added inside `Orders.php` (following the same pattern as `updateStatus`):

- `onUploadPhoto($data, $is_array)`
- `onUploadMultiplePhotos($data, $is_array)`
- `onDeletePhoto($data, $is_array)`
- `uploadPhoto()` – public endpoint calling `onUploadPhoto`.
- `uploadMultiplePhotos()`
- `deletePhoto()` – public endpoint calling `onDeletePhoto`.

**Parameters for single photo upload**:
- `order_id` or `id` (required)
- `field` (required, one of the four fields)
- `file` (file) or `file_base64` (base64 string)
- `title`, `description`, `is_public`, `auto_resize`, etc. (optional)

**Success response**:
```json
{
    "code": 200,
    "status": true,
    "message": "Photo uploaded successfully",
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

#### 3. Handling Uploaded Files (`multipart/form-data`)

The functions were improved to extract files directly using `Input::hasFile()` and `Input::file()`, while retaining support for `base64` for environments that do not upload files directly.

#### 4. Full Permission Integration

Before any photo upload, `validatePhotoUploadPermission` (inside `uploadOrderPhoto` in `StepPhotos`) is called. If the check fails (disallowed role, disallowed status, maximum reached), an exception is thrown and an appropriate error message is displayed.

#### 5. Returning Photo URLs

After uploading the photo, `getOrderPhotoUrls` is called to generate the URL of the original photo as well as default thumbnail sizes (`thumb`, `medium`, `large`). Sizes can be customised via `thumb_sizes` in the options.

---

## Usage Examples

### Upload a single photo (multipart/form-data)

```http
POST /api/v1/orders/orders/upload-photo
Authorization: Bearer ...
Content-Type: multipart/form-data

order_id: 123
field: user_before_delivery
file: @/path/to/photo.jpg
title: Photo before delivery
```

### Upload multiple photos (base64)

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

### Delete a photo

```http
DELETE /api/v1/orders/orders/delete-photo/123/user_before_delivery/456
```

### Attempt to change the order status to DELIVERY without the required photos

The request will be rejected with the message:
```json
{
    "code": 400,
    "status": false,
    "message": "You must upload photos in field user_before_delivery before changing the status to DELIVERY"
}
```

### Bypass photo check (admin)

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

## Upgrade Requirements

### For `Nano.Orders`

1. **Update the files**:
   - `models/Order.php` – add the new relationships.
   - `traits/steps/StepPhotos.php` – add the entire trait.
   - `Plugin.php` – add `registerOrderFileUploadFields` and call it in `boot()`.
   - `config/nano/orders.php` – add the `photo_rules` section (optional).
   - `lang/ar/lang.php` and `lang/en/lang.php` – add the new translation keys.

2. **No new migrations** – `attachMany` relationships do not require additional columns in the `nano_orders_orders` table.

3. **Dependencies**:
   - The `Nano.FileUpload` plugin must be installed and configured (version 1.0.7 or later).
   - `Nano.FileUpload` depends on `Nano.Api` (to provide `AuthHelpers`).

4. **Configuration**:
   - You can leave the default values or customise via `.env` or direct editing.

### For `Nano.OrdersApi`

1. **Update the files**:
   - `APIControllers/Orders.php` – add the new functions.
   - `routes.php` – add the new routes.
   - `lang/ar/lang.php` and `lang/en/lang.php` – add the new translation keys under `orders.photo`.

2. **Dependencies**:
   - Requires version 2.2.12 of `Nano.Orders` (or later).

3. **Compatibility testing**:
   - Test uploading photos using `multipart/form-data` and `base64`.
   - Test uploading multiple photos and exceeding the maximum limit.
   - Test deleting photos and permission checks.
   - Test preventing status changes without required photos via `update-status`.

---

## Conclusion

Versions **Nano.Orders 2.2.12** and **Nano.OrdersApi 1.0.22** add powerful photo management capabilities to the NanoSoft orders system. Thanks to the integration with `Nano.FileUpload`, uploading is secure and unified, supporting both regular files and base64. Controlling permissions and roles (owner / delivery / admin) and order statuses ensures that photos are uploaded only at the appropriate time by authorised persons. The `skip_photo_check` option allows administrators to bypass requirements when necessary. The new API endpoints allow external applications to easily benefit from these features.

The code is documented, extensible, and compatible with previous versions.

---

**Reference documentation**:
- [`StepPhotos` trait documentation](./docs/Orders/Traits/Steps/Docs-StepPhotos-Trait-en.md)
- [`Nano.FileUpload` documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [Photo Management API documentation](./docs/OrdersApi/Docs-OrdersApi-Photos-en.md)

