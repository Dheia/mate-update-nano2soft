# 📘 `StudentHelper` Class Documentation

**Version:** 1.0.13  
**Path:** `Tss\Student\Helpers\StudentHelper`  
**Purpose:** Provides a unified and simplified interface for handling student, parent, and academic record data, integrating advanced functions from `StudentRecordsHelper` and helper functions for linking users to various entities.

---

## 1. Introduction

The `StudentHelper` class is the main interface used by developers (in API or Backend) to interact with student, parent, and `StudentRecord` data. In version 1.0.13 it was restructured to rely on `StudentRecordsHelper` for query operations (`getRecords`) while maintaining full backward compatibility. The class also includes important helper functions such as:

- Linking a `User` to a `Student` or `Mparent`.
- Fetching a parent’s children.
- Extracting a student ID from a `StudentRecord`.
- Retrieving the current academic record of a student.

**Key features of the class:**

- **Full backward compatibility:** The old `getRecords` functions still work with the same signature.
- **Flexible querying:** You can use `getRecords`, `getParentRecords`, or `getStudentRecordRecords` as needed.
- **Linking helper functions:** Functions like `getStudentByUser`, `getUserByStudent`, `getMparentByUser`, `getUserByMparent` make it easy to move between entities.
- **Family relationship support:** Functions `getStudentIdsByMparent`, `getStudentsByMparent`, `getStudentsByUser` allow fetching a parent’s children easily.
- **Legacy helper functions:** `getStudentRecord`, `getResultRecord`, `setUpStudentRecord` (for promotion) are still available for old code.

---

## 2. Benefits and Use Cases

| Benefit | Description |
|---------|-------------|
| **Single interface for students and parents** | The class covers all basic operations related to students, parents, and their records. |
| **Easy to use in the API** | You can call `StudentHelper::getRecords` directly from API controllers without creating additional objects. |
| **Linking users to students and parents** | `get*ByUser` functions allow you to obtain the student or parent linked to the current (or any) user. |
| **Family scenario support** | Fetching a parent’s children or fetching students associated with a user (whether a parent or a student). |
| **Preserves legacy code** | Old functions like `getStudentRecord`, `getResultRecord`, `setUpStudentRecord` have not been deleted; the modern version of `getStudentRecord` was renamed to `getStudentRecordV1` to avoid conflicts. |
| **Integration with `StudentRecordsHelper`** | All new query functions benefit from caching, events, and advanced filters without re‑implementing the logic. |

**Typical use cases:**

- Fetching a list of students in the `StudentsApi::index` controller.
- Displaying a student’s personal data linked to the current user.
- In a parent’s interface: displaying the list of their registered children.
- Obtaining the current academic record of a student (class, year, fees).
- Converting a student record ID to the student ID itself.

---

## 3. Class Methods and Properties

### 3.1 Main Query Methods

| Method | Description |
|--------|-------------|
| `getRecords(array $options, bool $isException = false): array` | Calls `StudentRecordsHelper::getRecords` to fetch student data with full advanced filtering support. |
| `getParentRecords(array $options, bool $isException = false): array` | Fetches parent data with special filters such as `has_students`, `students_id`, `is_has_students_count`. |
| `getStudentRecordRecords(array $options, bool $isException = false): array` | Fetches `StudentRecord` data with special filters for fees, dates, and statuses. |

All these methods return a unified array containing `code`, `status`, `message`, `data`, `error`, `errors`, `input_data`, `process_data`, `debug`.

### 3.2 User / Student / Parent Linking Methods

| Method | Description |
|--------|-------------|
| `getStudentByUser($user = null, $isForce = false)` | Returns the student object linked to the user (via `user_id`/`user_type` or `ref_id`/`ref_type`). |
| `getUserByStudent($student = null, $isForce = false)` | Returns the user object linked to the student. |
| `getMparentByUser($user = null, $isForce = false)` | Returns the parent object linked to the user. |
| `getUserByMparent($mparent = null, $isForce = false)` | Returns the user object linked to the parent. |

### 3.3 Child‑Specific Methods (for `Mparent`)

| Method | Description |
|--------|-------------|
| `getStudentIdsByMparent($mparent, bool $useCache = true): array` | Returns an array of student IDs belonging to a specific parent. |
| `getStudentsByMparent($mparent, array $options = [])` | Returns student objects belonging to a specific parent (supports the same options as `getRecords`). |
| `getStudentsByUser($user = null, array $options = [])` | Fetches students associated with a specific user. If the user is a parent, returns their children; if a student, returns only that student’s data. |

### 3.4 `StudentRecord`‑Specific Methods

| Method | Description |
|--------|-------------|
| `getStudentIdFromRecord($record): ?int` | Extracts the student ID from a `StudentRecord` object, record ID, or an array containing `student_id`. |
| `getCurrentStudentRecord($student, array $options = []): ?StudentRecord` | Retrieves the student’s current academic record (usually with `status_class = 'continue'`), with support for specifying the year and class. |

### 3.5 Legacy Methods (for backward compatibility)

| Method | Description |
|--------|-------------|
| `getStudentRecordV1(array $options = [])` | The old version of `getStudentRecord` (renamed). Fetches student records based on limited options. |
| `getResultRecord(array $options = [])` | Fetches result records (from `Tss\SchoolControl\Models\Result`). |
| `setUpStudentRecord(array $options = [])` | Performs student promotion (changes `status_class`) based on their results and control options. |
| `getQueryDate($records , $options = [])` | Helper function for date filtering using `AdvancedQueryHelper`. |
| `checkValueIsNotAll($value = null)` | Checks whether a value is not `*` or `'all'`. |
| `scopeWhereField(...)` | Applies a `where` condition on a field with support for `is_or`, `is_not`, `is_force`. |
| `sanitizeDateFields(...)`, `parseCarbon`, `makeCarbon`, `w3cDatetime`, `toDatetime` | Helper functions for date handling (used internally). |

---

## 4. Complete and Detailed Examples

### 4.1 Fetching a List of Students with Advanced Filtering

```php
$result = StudentHelper::getRecords([
    'gender' => 'male,female',
    'is_or_gender' => true,
    'is_active' => true,
    'q' => 'Mohamed',
    'orderBy' => 'full_name',
    'orderDirection' => 'asc',
    'per_page' => 20,
    'cache_enabled' => true,
]);

if ($result['status']) {
    $paginator = $result['data'];
    foreach ($paginator as $student) {
        echo $student->full_name . '<br>';
    }
} else {
    Log::error($result['error']);
}
```

### 4.2 Fetching Parents Who Have More Than Two Children

```php
$result = StudentHelper::getParentRecords([
    'is_has_students_count' => ['>', 2],
    'orderBy' => 'full_name',
]);

if ($result['status']) {
    $parents = $result['data'];
    foreach ($parents as $parent) {
        echo $parent->full_name . ' - number of children: ' . $parent->students_count . '<br>';
    }
}
```

### 4.3 Fetching Student Records for a Specific Academic Year and `continue` Status

```php
$result = StudentHelper::getStudentRecordRecords([
    'year_id' => 3,
    'status_class' => 'continue',
    'with_count' => ['student', 'class'],
]);

if ($result['status']) {
    $records = $result['data']['data']; // in default mode returns a Paginator with transformer
    foreach ($records as $record) {
        echo "Student: {$record['student']['full_name']}, Class: {$record['class']['name']}<br>";
    }
}
```

### 4.4 Obtaining the Student Linked to the Current User

```php
$user = BackendAuth::getUser(); // or AuthHelpers::getCurrentUser()
$student = StudentHelper::getStudentByUser($user);
if ($student) {
    echo "Welcome {$student->full_name}";
} else {
    echo "You are not registered as a student in the system.";
}
```

### 4.5 Fetching a Specific Parent’s Children

```php
$mparent = Mparent::find(10);
$students = StudentHelper::getStudentsByMparent($mparent, [
    'is_active' => true,
    'orderBy' => 'full_name',
]);

foreach ($students as $student) {
    echo $student->full_name . '<br>';
}
```

### 4.6 Fetching Students Associated with a User (whether Parent or Student)

```php
$user = AuthHelpers::getCurrentUser();
$students = StudentHelper::getStudentsByUser($user, ['is_active' => true]);

if ($students) {
    foreach ($students as $student) {
        echo $student->full_name . '<br>';
    }
}
```

### 4.7 Extracting the Student ID from a Record

```php
$record = StudentRecord::find(100);
$studentId = StudentHelper::getStudentIdFromRecord($record); // or pass 100 directly
if ($studentId) {
    $student = Student::find($studentId);
    echo $student->full_name;
}
```

### 4.8 Retrieving the Current Academic Record of a Student

```php
$student = Student::find(5);
$currentRecord = StudentHelper::getCurrentStudentRecord($student, [
    'year_id' => 3, // current academic year (optional, will be auto‑fetched)
]);

if ($currentRecord) {
    echo "Class: " . $currentRecord->class_id;
    echo "Fees: " . $currentRecord->study_fees;
}
```

### 4.9 Using `getResultRecord` to Fetch Student Results

```php
$results = StudentHelper::getResultRecord([
    'year_id' => 3,
    'class_id' => 2,
    'result_status' => 'success', // only successful students
]);
```

### 4.10 Promoting Students (Changing Their Record Statuses)

```php
$upgradeResult = StudentHelper::setUpStudentRecord([
    'departments_id' => 1,
    'year_id' => 3,
    'class_id' => 2,
    'change_status_class' => 'auto',
    'record_up_to' => 'success', // only successful ones
]);

if ($upgradeResult['status']) {
    echo "Successfully promoted {$upgradeResult['count_success']} student(s).";
} else {
    echo $upgradeResult['error'];
}
```

---

## 5. Backward Compatibility

- **`getRecords`:** now calls `StudentRecordsHelper::getRecords` instead of the direct implementation, but its signature and output have not changed. All code that used `StudentHelper::getRecords` will continue to work.
- **`getStudentRecord`:** was renamed to `getStudentRecordV1` to avoid conflicts with `StudentRecordsHelper` methods. If you are using it in your project, replace it with `getStudentRecordV1` or upgrade to use `getStudentRecordRecords`.
- **Other functions (`getResultRecord`, `setUpStudentRecord`, date helpers):** remain unchanged to preserve compatibility.

---

## 6. Summary

| Feature | Details |
|---------|---------|
| **Unified interface for students and parents** | `getRecords`, `getParentRecords`, `getStudentRecordRecords` cover the basic needs. |
| **User linking** | `get*ByUser` and `getUserBy*` functions make it easy to move between entities. |
| **Family relationship support** | `getStudentsByMparent`, `getStudentsByUser`, and `getStudentIdsByMparent`. |
| **Academic record support** | `getCurrentStudentRecord`, `getStudentIdFromRecord`. |
| **Backward compatibility** | Old functions are still available (except `getStudentRecord`, which was renamed). |
| **Integration with `StudentRecordsHelper`** | Modern query functions benefit from caching, events, and advanced filters. |

**Recommended usage:**

- For new projects: use `StudentHelper::getRecords`, `getParentRecords`, `getStudentRecordRecords` with the advanced options.
- For user‑related relationships: use `getStudentByUser`, `getMparentByUser`, `getStudentsByUser`.
- To obtain the current academic record: use `getCurrentStudentRecord`.
- If you need promotion logic: use `setUpStudentRecord`.

---

## 7. Conclusion

The `StudentHelper` class is the main entry point for all read and linking operations related to students and parents in the Nano ecosystem. Thanks to the restructuring in version 1.0.13, the class has become more powerful and flexible while maintaining full backward compatibility. We recommend relying on this class in all new applications and upgrading old applications to benefit from the advanced features (especially `getParentRecords` and `getStudentRecordRecords`).

**References:**

- [Full documentation of the `Tss.Student` plugin](./Docs-Studenten.md)
- [`StudentRecordsHelper` class](./Docs-StudentRecordsHelper-Class-en.md)
- [`AccessManager` class (Nano.AuthApi)](../AuthApi/Docs-AccessManager-en.md)
- [`AdvancedQueryHelper` class (Nano2.QueryBuilder)](../querybuilder/Docs-AdvancedQueryHelper.md)
- [API Plugin Development Guide (Nano-Api-SKILL)](../mcp/Nano-Api-SKILL.md)

---
