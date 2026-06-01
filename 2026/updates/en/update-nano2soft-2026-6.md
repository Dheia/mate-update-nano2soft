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

