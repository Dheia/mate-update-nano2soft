# توثيق سلوك `FollowableModel` 

---

## 1. مقدمة

`FollowableModel` هو سلوك متقدم يضيف نظام متابعة متعدد الأشكال (polymorphic) إلى أي موديل في تطبيقات نانوسوفت. يتيح هذا السلوك للمستخدمين متابعة كائنات متنوعة (مثل المقالات، المنتجات، المستخدمين، إلخ) مع دعم شامل لخيارات القبول والنشاط واستعادة المتابعات المحذوفة. يأتي السلوك مزوداً بمجموعة غنية من النطاقات والدوال المساعدة التي تسهل بناء أنظمة اجتماعية متقدمة بمرونة وأداء عالٍ.

---

## 2. الفوائد والميزات الرئيسية

- **نظام متابعة متعدد الأشكال** – يمكن جعل أي موديل قابلاً للمتابعة.
- **مرونة في حالات المتابعة** – دعم حقول `is_accepted` (قبول المتابعة) و `is_active` (نشاط المتابعة) لتمثيل حالات متعددة.
- **دعم المتابعات المحذوفة (Soft Deletes)** – إمكانية تضمين المتابعات المحذوفة في الاستعلامات حسب الحاجة.
- **نطاقات ترتيب متقدمة** – ترتيب الكائنات حسب عدد المتابعات، أحدث متابعة، أو أقدم متابعة، مع خيارات تصفية القبول والنشاط.
- **نطاقات تصفية** – تصفية الكائنات التي يتابعها مستخدم معين، أو التي لها متابعون، أو التي لا متابعين لها.
- **إضافة عمود `is_followed_by_user`** – يظهر في نتائج الاستعلامات ما إذا كان المستخدم الحالي يتابع الكائن، دون الحاجة إلى تحميل العلاقة.
- **واجهة متسقة** – دوال مشابهة لسلوكي المفضلات والتقييمات (FavoriteableModel و ReviewRateable) لتجربة تطوير موحدة.
- **تحسين الأداء** – استخدام استعلامات فرعية (subqueries) فعالة وتجنب تحميل العلاقات بالكامل حيثما أمكن.
- **توافق عكسي** – الاحتفاظ بأسماء النطاقات القديمة (`addSortByCountFollows`، `withSortByLatestAtFollows`، إلخ) لضمان عدم كسر الكود القديم.

---

## 3. متطلبات الاستخدام

1. وجود موديل `Nano\Follows\Models\Follow` (الذي يمثل سجل المتابعة) مع جدول `nano_follows_follows`.
2. الموديل المراد جعله قابلاً للمتابعة يجب أن يكون من نوع Eloquent (أو `October\Rain\Database\Model` في حالة استخدام OctoberCMS).
3. أن يكون المستخدمون (مثل موديل `User`) قادرين على استخدام نظام المتابعة (أي أنهم يستوفون واجهة `Authenticatable`).
4. اختياري: استخدام `SoftDeletes` في موديل `Follow` للاستفادة من خيار `withTrashed`.

---

## 4. طريقة التثبيت والإعداد

### المتطلبات الأساسية
- نظام نانوسوفت الأساسي (NanoSoft System) الإصدار 3 أو أعلى.
- موديل `Nano\Follows\Models\Follow` مع جدول `nano_follows_follows`.
- الموديل المراد جعله قابلاً للمتابعة يجب أن يستخدم نظام `ExtensionBase` (أو يكون من نوع `Model` في OctoberCMS).

### خطوات الإضافة
1. قم بإضافة السلوك إلى الموديل عبر خاصية `$implement`:

```php
class Product extends Model
{
    public $implement = [
        \Nano\Follows\Behaviors\FollowableModel::class
    ];
}
```

2. تأكد من أن جدول المتابعات (`nano_follows_follows`) يحتوي على الحقول التالية على الأقل:
   - `followable_id`, `followable_type` (للعلاقة متعددة الأشكال)
   - `user_id`, `user_type` (للعلاقة بالمستخدم)
   - `is_accepted`, `is_active`, `accepted_at`, `re_follow_at` (لحالات المتابعة)
   - `deleted_at` (إذا كنت تستخدم Soft Deletes)

3. لا حاجة لإضافة أي حقول إضافية في الموديل؛ العلاقة `follows` تُضاف تلقائياً.

---

## 5. الهيكل الأساسي

بعد إضافة السلوك، يصبح للموديل العلاقة التالية:

```php
$product->follows() // يعيد علاقة morphMany تربط سجلات Follow بهذا الموديل
```

كما يتوفر دالة مختصرة للتحقق من متابعة المستخدم الحالي:

```php
if ($product->isUserFollow()) {
    // المستخدم يتابع هذا المنتج
}
```

---

## 6. الدوال والنطاقات المتاحة

### 6.1 دوال بناء الاستعلامات الفرعية (داخلية)

#### `protected function buildFollowCountSubQuery($onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يبني استعلام فرعي لحساب عدد المتابعات الخاصة بالكائن الحالي، مع خيارات تصفية.
- **المعاملات**:
  - `$onlyAccepted` (bool|null): `true` للمتابعات المقبولة فقط، `false` للمتابعات غير المقبولة فقط، `null` للكل.
  - `$onlyActive` (bool|null): `true` للمتابعات النشطة فقط، `false` للمتابعات غير النشطة فقط، `null` للكل.
  - `$withTrashed` (bool): تضمين المتابعات المحذوفة.
- **المخرجات**: استعلام فرعي من نوع `\Illuminate\Database\Query\Builder`.

#### `protected function buildFollowLatestDateSubQuery($field = 'created_at', $onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يبني استعلام فرعي لأحدث تاريخ متابعة (MAX).
- **المعاملات**:
  - `$field`: اسم الحقل الزمني (`created_at`، `updated_at`، `accepted_at`، `re_follow_at`).
  - باقي المعاملات كالسابق.

#### `protected function buildFollowFirstDateSubQuery($field = 'created_at', $onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يبني استعلام فرعي لأول تاريخ متابعة (MIN).

### 6.2 نطاقات عدد المتابعات

#### `scopeAddCountFollows(Builder $query, $orderDirection = 'DESC', $columnName = 'follows_count', $onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يضيف عموداً يحتوي على عدد المتابعات إلى نتيجة الاستعلام (بدون ترتيب).
- **مثال**:
  ```php
  $products = Product::addCountFollows()->get();
  foreach ($products as $product) {
      echo $product->follows_count; // عدد متابعي المنتج
  }
  ```

#### `scopeSortByCountFollows(Builder $query, $orderDirection = 'DESC', $columnName = 'follows_count', $onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يرتب النتائج حسب عدد المتابعات (دون إضافة العمود).
- **مثال**:
  ```php
  // ترتيب المنتجات تنازلياً حسب عدد المتابعات المقبولة
  $products = Product::sortByCountFollows('DESC', 'follows_count', true)->get();
  ```

#### `scopeWithCountFollows(Builder $query, $orderDirection = 'DESC', $columnName = 'follows_count', $onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يجمع بين `addCountFollows` و `sortByCountFollows` (إضافة العمود والترتيب معاً).
- **مثال**:
  ```php
  $products = Product::withCountFollows('DESC', 'follows_count', true)->get();
  ```

### 6.3 نطاقات أحدث تاريخ متابعة

#### `scopeAddLatestFollow(Builder $query, $orderDirection = 'DESC', $columnName = 'latest_follow_at', $onlyAccepted = null, $onlyActive = null, $withTrashed = false, $field = 'created_at')`
- **الوصف**: يضيف عموداً بأحدث تاريخ متابعة.

#### `scopeSortByLatestFollow(...)` – ترتيب حسب أحدث تاريخ متابعة (دون إضافة عمود).

#### `scopeWithLatestFollow(...)` – إضافة العمود والترتيب معاً.

**مثال**:
```php
// ترتيب المنتجات حسب أحدث متابعة (منشأة) مع تضمين المتابعات غير المقبولة
$products = Product::sortByLatestFollow('DESC', 'latest_follow_at', null, null, false, 'created_at')->get();
```

### 6.4 نطاقات أول تاريخ متابعة

#### `scopeAddEarliestFollow(...)` – يضيف عموداً بأقدم تاريخ متابعة.
#### `scopeSortByEarliestFollow(...)` – ترتيب حسب أقدم تاريخ متابعة.
#### `scopeWithEarliestFollow(...)` – إضافة العمود والترتيب.

**مثال**:
```php
$products = Product::withEarliestFollow('ASC', 'first_follow_at', true, true)->get(); // أول متابعة مقبولة ونشطة
```

### 6.5 نطاقات التصفية حسب المستخدم

#### `scopeFollowedByUser(Builder $query, $user = null, $isAccepted = true, $isActive = null, $withTrashed = false)`
- **الوصف**: تصفية الكائنات التي يتابعها مستخدم معين (إذا لم يُمرر `$user`، يستخدم المستخدم الحالي).
- **مثال**:
  ```php
  // المنتجات التي يتابعها المستخدم الحالي
  $products = Product::followedByUser()->get();
  
  // المنتجات التي يتابعها مستخدم محدد (حتى لو كانت المتابعة غير مقبولة)
  $products = Product::followedByUser($user, false)->get();
  ```

#### `scopeNotFollowedByUser(Builder $query, $user = null, $isAccepted = true, $isActive = null, $withTrashed = false)`
- **الوصف**: تصفية الكائنات التي **لا** يتابعها مستخدم معين.

#### `scopeWhereHasFollow($builder, $user = null, $is_accepted = true)` – (قديم، للتوافق العكسي)
- **الوصف**: يُرجع الكائنات التي يتابعها المستخدم المحدد مع إمكانية تحديد حالة القبول. لا يدعم خيارات `is_active` أو `withTrashed`. يُفضل استخدام `FollowedByUser` للحصول على مرونة أكبر.
- **مثال**:
  ```php
  $products = Product::whereHasFollow($user, true)->get();
  ```

#### `scopeWhereHasFollows($builder, $user = null, $is_accepted = true)` – (قديم، للتوافق العكسي)
- **الوصف**: مرادف لـ `whereHasFollow` (يوجد لأسباب تاريخية). يعمل بنفس الطريقة.

### 6.6 نطاقات التصفية حسب وجود متابعين

#### `scopeHasFollowers(Builder $query, $minCount = 1, $isAccepted = null, $isActive = null, $withTrashed = false)`
- **الوصف**: تصفية الكائنات التي لديها على الأقل `$minCount` من المتابعين (مع خيارات التصفية).
- **مثال**:
  ```php
  // منتجات لها 5 متابعين مقبولين على الأقل
  $products = Product::hasFollowers(5, true)->get();
  ```

#### `scopeHasNoFollowers(Builder $query, $isAccepted = null, $isActive = null, $withTrashed = false)`
- **الوصف**: تصفية الكائنات التي لا متابعين لها.

### 6.7 نطاق إضافة عمود حالة المتابعة للمستخدم الحالي

#### `scopeWithIsFollowedByUser(Builder $query, $user = null, $columnName = 'is_followed_by_user', $isAccepted = true, $isActive = null, $withTrashed = false)`
- **الوصف**: يضيف عموداً منطقياً (`1` أو `0`) إلى نتيجة الاستعلام يبين ما إذا كان المستخدم المحدد يتابع الكائن.
- **مثال**:
  ```php
  $products = Product::withIsFollowedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_followed_by_user) {
          echo "أنت تتابع هذا المنتج";
      }
  }
  ```

### 6.8 نطاقات خاصة

#### `scopeTopFollowed(Builder $query, $limit = 10, $orderDirection = 'DESC', $onlyAccepted = true, $onlyActive = null)`
- **الوصف**: يجلب الكائنات الأكثر متابعة (عدد متابعين مقبولين) مع تحديد الحد.
- **مثال**:
  ```php
  $topProducts = Product::topFollowed(5)->get();
  ```

#### `scopeSortByAcceptedFollowsCount(Builder $query, $orderDirection = 'DESC', $columnName = 'accepted_follows_count')`
- اختصار لـ `sortByCountFollows` مع `$onlyAccepted = true`.

#### `scopeSortByActiveFollowsCount(Builder $query, $orderDirection = 'DESC', $columnName = 'active_follows_count')`
- اختصار لـ `sortByCountFollows` مع `$onlyActive = true`.

### 6.9 دوال إحصائية مساعدة

#### `getFollowersCountByType($onlyAccepted = null, $onlyActive = null)`
- **الوصف**: يعيد عدد المتابعين مقسمين حسب `user_type` (نوع نموذج المستخدم).
- **المخرجات**: `Collection` حيث المفتاح هو `user_type` والقيمة هي العدد.
- **مثال**:
  ```php
  $counts = $product->getFollowersCountByType(true, null);
  // ['RainLab\User\Models\User' => 42, 'App\Models\Admin' => 5]
  ```

#### `getTotalFollowers($onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: إجمالي عدد المتابعين (أو حسب التصفية).
- **المخرجات**: عدد صحيح.

#### `getFollowersUsers($onlyAccepted = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يعيد مجموعة من المستخدمين الذين يتابعون الكائن (مع تحميل العلاقة `user`).
- **المخرجات**: `Collection` تحتوي على كائنات المستخدمين.

### 6.10 دوال أساسية في السلوك (غير موجودة في الـ Trait)

#### `followedBy()`
- **الوصف**: يعيد مجموعة المستخدمين الذين يتابعون الكائن الحالي.
- **المخرجات**: `Collection` من المستخدمين (مع تحميل `user`).
- **مثال**:
  ```php
  $followers = $product->followedBy();
  foreach ($followers as $user) {
      echo $user->name;
  }
  ```

#### `isUserFollow($user = null)`
- **الوصف**: يتحقق مما إذا كان المستخدم (أو المستخدم الحالي) يتابع الكائن.
- **المخرجات**: `bool`.

#### `getUserFollow($user = null)`
- **الوصف**: يعيد سجل المتابعة الخاص بالمستخدم (إن وجد).
- **المخرجات**: كائن `Follow` أو `null`.

#### `addFollow($data, $user = null, $parent = null): Follow`
- **الوصف**: إضافة متابعة جديدة للكائن. يمكن تمرير `$data` كـ `array` أو كـ `string` (سيتم تفسيره كـ `content`).
- **المخرجات**: كائن `Follow` المُنشأ.

#### `toggleFollow($data, $user = null, $parent = null)`
- **الوصف**: إذا كان المستخدم يتابع، يقوم بحذف المتابعة (أو استعادتها إذا كانت محذوفة)؛ وإلا يضيف متابعة جديدة.
- **المخرجات**: كائن `Follow` أو `null` عند الحذف.

#### `deleteFollow($id, $data = null): ?bool`
- **الوصف**: حذف متابعة معينة (Soft delete).
- **المخرجات**: `true` إذا تم الحذف، `false` إذا فشل، `null` في بعض الحالات.

#### `updateFollow($id, $data): Follow`
- **الوصف**: تحديث بيانات متابعة موجودة.

#### `countFollow(bool $onlyAccepted = false)`
- **الوصف**: عدد المتابعات (اختيارياً المقبولة فقط).
- **المخرجات**: عدد صحيح.

#### `getSuggestedFriends()`
- **الوصف**: (خاص بالكائن المستخدم) يعيد الأصدقاء المقترحين بناءً على متابعات الكائن.
- **المخرجات**: مجموعة من المستخدمين.

---

## 7. جدول ملخص للدوال والنطاقات حسب الفئة

| الفئة | الدوال/النطاقات |
|-------|-----------------|
| **إضافة المتابعات** | `addFollow()`, `toggleFollow()`, `deleteFollow()`, `updateFollow()` |
| **الاستعلام عن المتابعات** | `followedBy()`, `isUserFollow()`, `getUserFollow()` |
| **نطاقات التصفية حسب المستخدم** | `followedByUser()`, `notFollowedByUser()`, `whereHasFollow()` (قديم) |
| **نطاقات التصفية حسب وجود متابعين** | `hasFollowers()`, `hasNoFollowers()` |
| **نطاقات الترتيب حسب العدد** | `sortByCountFollows()`, `addCountFollows()`, `withCountFollows()` |
| **نطاقات الترتيب حسب التاريخ** | `sortByLatestFollow()`, `addLatestFollow()`, `withLatestFollow()`, `sortByEarliestFollow()`, `addEarliestFollow()`, `withEarliestFollow()` |
| **نطاقات خاصة** | `topFollowed()`, `sortByAcceptedFollowsCount()`, `sortByActiveFollowsCount()` |
| **دوال إحصائية** | `getFollowersCountByType()`, `getTotalFollowers()`, `getFollowersUsers()` |
| **دوال إضافية** | `countFollow()`, `getSuggestedFriends()` |
| **إضافة عمود حالة المتابعة** | `withIsFollowedByUser()` |

---

## 8. أمثلة عملية شاملة

### مثال 1: إضافة متابعة وإلغاؤها

```php
$user = Auth::getUser();
$product = Product::find(1);

// إضافة متابعة
$follow = $product->addFollow(['is_accepted' => false], $user);
// الآن $user يتابع المنتج ولكن بانتظار القبول

// التحقق من المتابعة
if ($product->isUserFollow($user)) {
    echo "يتبع هذا المنتج";
}

// إلغاء المتابعة (حذف)
$product->toggleFollow([], $user);
// بعد التبديل، تم الحذف
```

### مثال 2: ترتيب المنتجات حسب عدد المتابعين

```php
// المنتجات الأكثر متابعة (مقبولة فقط)
$products = Product::topFollowed(10, 'DESC', true)->get();

// المنتجات مرتبة حسب عدد المتابعات النشطة (مع تضمين المحذوفة)
$products = Product::sortByCountFollows('DESC', 'follows_count', null, true, true)->get();
```

### مثال 3: تصفية المنتجات التي يتابعها المستخدم الحالي

```php
$myFollowedProducts = Product::followedByUser()->get();

// مع إمكانية تضمين المتابعات غير المقبولة
$allFollows = Product::followedByUser(null, false)->get();
```

### مثال 4: إضافة عمود `is_followed_by_user` في قائمة المنتجات

```php
$products = Product::withIsFollowedByUser()->paginate(20);

foreach ($products as $product) {
    echo $product->name . ' - ';
    echo $product->is_followed_by_user ? 'متابَع' : 'غير متابَع';
}
```

### مثال 5: استخدام النطاقات المتقدمة

```php
// المنتجات التي لها أكثر من 5 متابعين مقبولين، مرتبة حسب أحدث متابعة (عبر حقل created_at)
$products = Product::hasFollowers(5, true)
    ->sortByLatestFollow('DESC', 'latest_follow_at', true, null, false, 'created_at')
    ->get();
```

### مثال 6: إحصائيات المتابعات

```php
$product = Product::find(1);

echo "إجمالي المتابعين: " . $product->getTotalFollowers(true, true); // مقبولين ونشطين
echo "متابعون حسب النوع: ";
print_r($product->getFollowersCountByType(true, true)->toArray());

$users = $product->getFollowersUsers(true, true);
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### مثال 7: الجمع بين `followedByUser` و `withIsFollowedByUser`

```php
// الحصول على المنتجات التي يتابعها المستخدم الحالي، مع إضافة عمود is_followed_by_user
// (العمود سيكون 1 لكل المنتجات في النتيجة لأنها جميعاً متابعة)
$products = Product::followedByUser()
    ->withIsFollowedByUser()
    ->get();
```

---

## 9. دوال التوافق العكسي

للحفاظ على الكود القديم الذي يستخدم أسماء النطاقات السابقة (`addSortByCountFollows`، `withSortByLatestAtFollows`، إلخ)، تم توفير دوال تغليف تُعيد التوجيه إلى الدوال الجديدة. هذه الدوال موصوفة بـ `@deprecated` وينصح باستخدام البدائل الجديدة.

**قائمة الدوال القديمة المدعومة:**

- `scopeAddSortByCountFollows` → `scopeAddCountFollows`
- `scopeWithSortByCountFollows` → `scopeWithCountFollows`
- `scopeAddSortByLatestAtFollows` → `scopeAddLatestFollow`
- `scopeSortByLatestAtFollows` → `scopeSortByLatestFollow`
- `scopeWithSortByLatestAtFollows` → `scopeWithLatestFollow`
- `scopeAddSortByFirstAtFollows` → `scopeAddEarliestFollow`
- `scopeSortByFirstAtFollows` → `scopeSortByEarliestFollow`
- `scopeWithSortByFirstAtFollows` → `scopeWithEarliestFollow`

**ملاحظة:** `scopeSortByCountFollows` لم يتغير اسمه، لذا لا حاجة لدالة تغليف.

**ملاحظة إضافية:** النطاقات القديمة `whereHasFollow` و `whereHasFollows` لا تزال موجودة في الكود ولكن يُنصح باستخدام `FollowedByUser` لمرونة أكبر.

---

## 10. تحسين الأداء والتخزين المؤقت

السلوك يعتمد على استعلامات فرعية (subqueries) محسنة لتجنب تحميل العلاقات بالكامل. لا يوجد نظام تخزين مؤقت مدمج بشكل افتراضي، لكن يمكنك الاعتماد على آلية التخزين المؤقت للموديلات نفسها أو استخدام ذاكرة التخزين المؤقت للتطبيق إذا احتجت لتكرار استعلامات ثقيلة.

- عند استخدام `withCountFollows` أو `sortByCountFollows`، يتم تنفيذ استعلام فرعي واحد فقط.
- استخدام `withIsFollowedByUser` يضيف استعلاماً فرعياً لكل صف، لذا يُنصح باستخدامه بحذر في القوائم الكبيرة، لكنه عادةً ما يكون مقبولاً بسبب فاعلية استعلامات SQL.

**ملاحظة حول `withTrashed`:** عند استخدام `withTrashed = true` في أي من النطاقات أو الدوال الإحصائية، تأكد من أن موديل `Nano\Follows\Models\Follow` يستخدم الترايت `SoftDeletes`؛ وإلا ستظهر أخطاء في الاستعلام.

---

## 11. الخاتمة

`FollowableModel` هو حل متكامل وقوي لإدارة نظام المتابعة في تطبيقات نانوسوفت. بفضل تنوع النطاقات والدوال المساعدة، يمكن للمطورين بناء أنظمة اجتماعية متقدمة بسهولة مع الاحتفاظ بمرونة عالية وقابلية للتوسع. التوافق العكسي يضمن انتقالاً سلساً من الإصدارات السابقة. نأمل أن يكون هذا التوثيق مرجعاً شاملاً لجميع إمكانيات السلوك.