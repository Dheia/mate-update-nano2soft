## 2026-05-27 – 2026-05-28

**Update to the `Tss.Homework` Plugin – Version 1.0.9**

### Full Addition of the `Feedback` Module (Submissions and Evaluations)

---

### Summary of Updates

Version **1.0.9** of the `Tss.Homework` plugin introduces the fully integrated **`Feedback`** module, which had been structurally present since version 1.0.7 (table `tss_homework_feedbacks`) but is now fully supported in the backend:

- **Model (`Feedback`)** with all relationships, scopes, and events.
- **Fully integrated administration interface** (Controller, Model, Config) within the `Backend`.
- **List (`columns.yaml`)** to display submissions in a table.
- **Integrated input form (`fields.yaml`)** with tabs and components (dropdowns, rich editor, file uploads).
- **Custom permissions** (`permissions`) to manage submissions.
- **Sidebar menu item** (`menus`) under the main `homework` menu.
- **Full translation** (`lang/ar/lang.php` and `lang/en/lang.php`) for all elements.
- **Relationships with homeworks (`homeworks`)**, students, classes, groups, subjects, and teachers.

This update completes the homework system, allowing teachers to record student submissions and evaluations, supervisors to monitor performance, and future use via the API.

---

### Release Objectives

- **Complete the homework ecosystem** by linking submissions and evaluations to assignments.
- **Enable teachers to record student grades** for each assignment via an easy backend interface.
- **Provide basic reports** on submission status (received, grade, evaluation, notes).
- **Support nested relationships** with classes, groups, subjects, and academic records.
- **Prepare for API usage** (`Nano.HomeworkApi`) which depends on this model.

---

### New Features and Improvements

#### 1. Full‑Featured `Feedback` Model (`models/Feedback.php`)

The model was created following OctoberCMS best practices:

- **Table**: `tss_homework_feedbacks` (existing since 1.0.7).
- **Traits**: `Validation`, `SoftDelete`, `Sortable`.
- **Behaviors**: `TranslatableModel`, `BasicModel`, `BasicScope`.
- **JSONable fields**: `keywords`, `config_data`, `properties`, `other_data`, `ref_key_data`.
- **`belongsTo` relationships**: with `Homework`, `Student`, `StudentRecord`, `ClassTypeCompany`, `TbClass`, `Group`, `Subject`, `Employee` (teacher, educator), and others.
- **File relationships**: `attachOne` (image), `attachMany` (images, files).
- **Custom scopes**: `scopeRefTypeClass`, `scopeStatusHomework`, `scopeEvaluation`, `scopeIsRecive`.
- **Before/after save events**: automatic `code` generation, setting `ref_key_name_class` and `ref_key_values_class` based on `ref_type_class`, setting a default `published_at`.

#### 2. Integrated Administration Interface

##### **Submissions List (`feedback/columns.yaml`)**
- Basic columns: `id`, `code`, `name`, `homeworks_id` (with link to the assignment), `student_name`, `class`, `subject`, `evaluation`, `status_homework`, `degree`, `is_recive`, `is_active`, `created_at`.
- Supports search, sorting, and filtering.

##### **Input Form (`feedback/fields.yaml`)**
- **Tabs**:
  - `basic`: basic data (assignment, student, academic record, class, group, subject, teacher, semester, month, etc.).
  - `evaluation`: evaluation, status, grade, completion percentage, receipt.
  - `notes`: teacher notes, supervisor notes, parent notes, automatic note.
  - `details`: short description, formatted description, solution.
  - `media`: main image, image gallery, files.
  - `seo`: SEO titles and description.
  - `properties`: additional properties (Repeater).
  - `options`: publication options, status, order.
- **Dropdown fields** that depend on each other (`dependsOn`) such as: `class_id`, `group_id`, `student_id`, `subject_id`, `teacher_id`.
- **Components**: `fileupload`, `richeditor`, `taglist`, `datepicker`, `repeater`.

#### 3. New Permissions (`permissions`)

A `feedbacks` section was added in `registerPermissions()`:

```php
'feedbacks' => [
    'title' => 'Submissions and Evaluations Permissions',
    'access_all' => 'Access to all submissions',
    'access' => 'Access to submissions',
    'add' => 'Add a submission',
    'edit' => 'Edit a submission',
    'delete' => 'Delete a submission',
],
```

#### 4. Sidebar Menu (`menus`)

A `feedbacks` item was added as a sub‑menu under `homework`:

```php
'feedbacks' => [
    'label' => 'Submissions and Evaluations',
    'url' => Backend::url('tss/homework/feedbacks'),
    'icon' => 'icon-check-square-o',
    'permissions' => ['tss.homework.feedbacks.*'],
    'order' => 4,
],
```

#### 5. Full Translation (`lang/ar/lang.php` and `lang/en/lang.php`)

Translations were added for all elements:
- `feedbacks` → `tab`, `form`, `list`, `filter`, `pages`.
- Option values: `evaluation_options`, `status_homework_options`, `option_month_num`, etc.
- Error and success messages (to be used by the API in future versions).

#### 6. Data Fetching Support (`getStudentIdOptions`, `getClassIdOptions`, etc.)

Helper methods were implemented in the model to generate dynamic dropdown lists, taking into account:
- Company scope (`IsCompany`).
- Dependency on previously selected options (e.g., `departments_id`, `class_id`, `year_id`).
- Hierarchical display (for classes and branches).

#### 7. Integration with `Tss\Homework\Traits\Homeworkble`

The `Homeworkble` trait was used, which adds a global scope `scopeHomeworkble` to filter records by `ref_type_class` (in the case of `Feedback`, it could be used to distinguish types, but according to the current design, `Feedback` does not use `Homeworkble` by default because `feedback` is not a `homeworkble` medium; it is left for future use).

---

### Usage Examples in the Backend

#### 1. Add a New Submission

- Go to `Assignments > Submissions and Evaluations`.
- Click **"Add new"**.
- Select the **assignment** from the popup (depends on department, branch, and classification type).
- Select the **student** (the list updates automatically based on class, group, and year).
- Enter the **grade** (e.g., `9.5`).
- Choose the **evaluation** (`Excellent`, `Good`, `Average`, `Poor`).
- Choose the **submission status** (`Passed`, `Failed`).
- Upload solution files if any.
- Save.

#### 2. Update a Submission’s Grade from the List

- From the submissions list, directly click the grade you want to modify (if the column is editable) or open the form.
- Change the value and save.

#### 3. Filter Submissions by Assignment

- Use the `homeworks_id` filter at the top of the list to display only submissions for a specific assignment.

---

### Detailed Technical Changes

#### New Files

| Path | Description |
|------|-------------|
| `models/Feedback.php` | Complete `Feedback` model. |
| `models/feedback/columns.yaml` | List column definitions. |
| `models/feedback/fields.yaml` | Form field definitions. |
| `controllers/Feedbacks.php` | Backend controller for managing submissions. |
| `controllers/feedbacks/config_list.yaml` | List configuration. |
| `controllers/feedbacks/config_form.yaml` | Form configuration. |
| `controllers/feedbacks/config_reorder.yaml` | Reorder configuration (optional). |
| `lang/ar/lang.php` | Updated with the `feedbacks` section. |
| `lang/en/lang.php` | Updated with the `feedbacks` section. |

#### Modifications to Existing Files

- **`Plugin.php`**:
  - Added `registerPermissions()` for the `feedbacks` section.
  - Added `bootBackendNavigation()` to add the new sidebar menu item.
- **`version.yaml`**:
  - Added version `1.0.9` with the description `Support Feedbacks`.

#### No Database Changes

- The table `tss_homework_feedbacks` already existed since version 1.0.7, so no additional migrations are required.

---

### Upgrade Requirements (from 1.0.8 to 1.0.9)

1. **Update the code**: Replace all the files mentioned above.
2. **Update `version.yaml`**:
   ```yaml
   1.0.9:
     - 'Support Feedbacks'
   ```
3. **Run migrations**:
   ```bash
   php artisan plugin:refresh Tss.Homework
   ```
   (Only the version will be updated; no new migrations.)
4. **Re‑assign permissions (optional)**: You can assign the new permissions to the appropriate groups in `Settings > User Management > Groups`.
5. **Clear cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **Verify the sidebar**: The `Submissions and Evaluations` item should appear under the `Assignments` menu.

---

### Benefits and Added Value

- **Integrated with the homework system**: Each submission is linked to a specific assignment.
- **Easy data entry**: Smart dependent dropdowns reduce errors.
- **Instant reporting**: Teachers and supervisors can see each student’s status.
- **API‑ready**: The model is ready to be used by `Nano.HomeworkApi` (the `Feedbacks` controller).
- **Multi‑language support**: Ready for interface translation according to the user’s language.
- **Extensibility**: Additional fields or relationships can be added in the future.

---

### Conclusion

Version **1.0.9** of the `Tss.Homework` plugin is an important addition that completes the homework ecosystem by providing a fully integrated backend interface for submissions and evaluations. With this update, schools and educational centres have a powerful tool to monitor student performance on daily and monthly assignments, with the possibility of future expansion using the dedicated API (`Nano.HomeworkApi`).

We recommend all users of the plugin upgrade to benefit from these features, while ensuring that the appropriate permissions are assigned to teachers and supervisors.

---

**Related documentation**:
- [`Tss.Homework` User Guide (in preparation)](./docs/User-Guide-en.md)
- [`Nano.HomeworkApi` documentation](./docs/HomeworkApi/Docs-HomeworkApi-en.md)
- [`Nano.HomeworkApi` update version 1.0.0](./docs/HomeworkApi/Update-HomeworkApi-v1.0.1-en.md)
