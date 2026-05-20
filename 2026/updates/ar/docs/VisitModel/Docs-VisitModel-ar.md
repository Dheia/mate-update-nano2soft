# توثيق سلوك `VisitModel` 

---

## 1. مقدمة

`VisitModel` هو سلوك متقدم يضيف نظام تتبع الزيارات والمشاهدات متعدد الأشكال (polymorphic) إلى أي موديل في تطبيقات نانوسوفت. يتيح هذا السلوك للمطورين تسجيل وتحليل زيارات المستخدمين للكائنات المختلفة (مثل المقالات، المنتجات، الصفحات) مع دعم شامل لخيارات التصفية المتقدمة (النوع، العملية، الحدث) والحالات (نشاط السجل، الحذف الناعم). يأتي السلوك مزوداً بمجموعة غنية من النطاقات والدوال المساعدة التي تسهل بناء أنظمة تحليلات متطورة بمرونة وأداء عالٍ.

### الفوائد الرئيسية

- **نظام تتبع متعدد الأشكال** – يمكن تتبع الزيارات لأي موديل.
- **دعم أنواع متعددة** – حقول `type` (نوع الزيارة: add, out, remove, reset, level_up) و `process_type` (in/out) و `event_type` (نوع الحدث) لتمثيل حالات وسياقات متنوعة.
- **دعم السجلات المحذوفة (Soft Deletes)** – إمكانية تضمين الزيارات المحذوفة في الاستعلامات حسب الحاجة.
- **نطاقات ترتيب متقدمة** – ترتيب الكائنات حسب عدد الزيارات، مجموع الزيارات، مجموع المشاهدات، أو التواريخ (أحدث/أقدم زيارة)، مع خيارات تصفية مرنة.
- **نطاقات تصفية** – تصفية الكائنات التي زارها مستخدم معين، أو التي لها زيارات، أو التي لا زيارات لها.
- **إضافة عمود `is_visited_by_user`** – يظهر في نتائج الاستعلامات ما إذا كان المستخدم الحالي قد زار الكائن، دون الحاجة إلى تحميل العلاقة.
- **واجهة متسقة** – دوال مشابهة لسلوك المتابعة (`FollowableModel`) لتجربة تطوير موحدة.
- **تحسين الأداء** – استخدام استعلامات فرعية (subqueries) فعالة وتجنب تحميل العلاقات بالكامل حيثما أمكن.
- **توافق عكسي** – الاحتفاظ بأسماء النطاقات القديمة (`sortByCountVisitsOld`، `withSortBySumVisits`، إلخ) لضمان عدم كسر الكود القديم.

---

## 2. متطلبات الاستخدام

1. وجود موديل `Nano2\Visitors\Models\Visit` (الذي يمثل سجل الزيارة) مع جدول `nano2_visitors_visits`.
2. الموديل المراد تتبع زياراته يجب أن يكون من نوع Eloquent (أو `October\Rain\Database\Model` في حالة استخدام OctoberCMS).
3. أن يكون المستخدمون (مثل موديل `User`) قادرين على استخدام نظام الزيارات (أي أنهم يستوفون واجهة `Authenticatable`).
4. اختياري: استخدام `SoftDeletes` في موديل `Visit` للاستفادة من خيار `withTrashed`.

---

## 3. إضافة السلوك إلى موديل

يمكن إضافة السلوك إلى أي موديل باستخدام خاصية `$implement` (في سياق نانوسوفت). السلوك يتضمن Trait يحمل النطاقات والدوال المساعدة، لذا بعد الإضافة يصبح الموديل قادراً على استخدام جميع الميزات.

### مثال: جعل موديل `Product` قابلًا لتتبع الزيارات

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano2\Visitors\Behaviors\VisitModel;

class Product extends Model
{
    public $implement = [
        VisitModel::class
    ];
}
```

**ملاحظة:** إذا كان الموديل يستخدم نظام `ExtensionBase` (كما هو الحال في تطبيقات نانوسوفت)، فإن الإضافة تتم عبر `$implement` كما هو موضح.

---

## 4. الهيكل الأساسي

بعد إضافة السلوك، يصبح للموديل العلاقة التالية:

```php
$product->visits() // يعيد علاقة morphMany تربط سجلات Visit بهذا الموديل
```

كما يتوفر خصائص محسوبة لسهولة الوصول إلى إحصائيات الزيارات:

```php
$product->visits_count   // عدد سجلات الزيارات
$product->sum_visits     // مجموع الزيارات (حقل visits)
$product->sum_views      // مجموع المشاهدات (حقل views)
$product->is_visits      // هل يوجد أي زيارات
```

---

## 5. الدوال والنطاقات المتاحة

### 5.1 دوال بناء الاستعلامات الفرعية (داخلية)

#### `protected function buildVisitCountSubQuery($onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **الوصف**: يبني استعلام فرعي لحساب عدد سجلات الزيارات (COUNT).
- **المعاملات**:
  - `$onlyActive` (bool|null): `true` للزيارات النشطة فقط، `false` للزيارات غير النشطة فقط، `null` للكل.
  - `$withTrashed` (bool): تضمين الزيارات المحذوفة.
  - `$type` (string|null): تصفية حسب نوع الزيارة (`add`, `out`, `remove`, ...).
  - `$processType` (string|null): تصفية حسب نوع العملية (`in`, `out`).
  - `$eventType` (string|null): تصفية حسب نوع الحدث.

#### `protected function buildVisitSumSubQuery(...)` – نفس المعاملات، لكن تستخدم `SUM(visits)`.

#### `protected function buildViewSumSubQuery(...)` – تستخدم `SUM(views)`.

#### `protected function buildVisitLatestDateSubQuery($field = 'created_at', ...)` – تستخدم `MAX(field)` للحصول على أحدث تاريخ.

#### `protected function buildVisitEarliestDateSubQuery($field = 'created_at', ...)` – تستخدم `MIN(field)` للحصول على أقدم تاريخ.

### 5.2 نطاقات عدد الزيارات (COUNT)

#### `scopeAddCountVisits(Builder $query, $orderDirection = 'DESC', $columnName = 'visits_count', $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **الوصف**: يضيف عموداً يحتوي على عدد سجلات الزيارات إلى نتيجة الاستعلام (بدون ترتيب).
- **مثال**:
  ```php
  $products = Product::addCountVisits()->get();
  foreach ($products as $product) {
      echo $product->visits_count;
  }
  ```

#### `scopeSortByCountVisits(...)` – يرتب النتائج حسب عدد الزيارات (دون إضافة العمود).

#### `scopeWithCountVisits(...)` – يجمع بين `addCountVisits` و `sortByCountVisits` (إضافة العمود والترتيب معاً).

**مثال**:
```php
$products = Product::withCountVisits('DESC', 'visits_count', true)->get(); // الزيارات النشطة فقط
```

### 5.3 نطاقات مجموع الزيارات (SUM visits)

#### `scopeAddSumVisits(...)` – يضيف عموداً بمجموع الزيارات (حقل `visits`).
#### `scopeSortBySumVisits(...)` – ترتيب حسب مجموع الزيارات.
#### `scopeWithSumVisits(...)` – إضافة العمود والترتيب معاً.

**مثال**:
```php
$products = Product::sortBySumVisits('DESC', 'total_visits', true)->get();
```

### 5.4 نطاقات مجموع المشاهدات (SUM views)

#### `scopeAddSumViews(...)` – يضيف عموداً بمجموع المشاهدات (حقل `views`).
#### `scopeSortBySumViews(...)` – ترتيب حسب مجموع المشاهدات.
#### `scopeWithSumViews(...)` – إضافة العمود والترتيب معاً.

### 5.5 نطاقات أحدث تاريخ زيارة

#### `scopeAddLatestVisit(...)` – يضيف عموداً بأحدث تاريخ زيارة (اختيارياً تحديد الحقل الزمني عبر `$field`).
#### `scopeSortByLatestVisit(...)` – ترتيب حسب أحدث تاريخ زيارة.
#### `scopeWithLatestVisit(...)` – إضافة العمود والترتيب معاً.

**مثال**:
```php
$products = Product::withLatestVisit('DESC', 'last_visit', true, false, 'add', 'in', 'contest', 'last_visit_at')->get();
```

### 5.6 نطاقات أقدم تاريخ زيارة

#### `scopeAddEarliestVisit(...)` – يضيف عموداً بأقدم تاريخ زيارة.
#### `scopeSortByEarliestVisit(...)` – ترتيب حسب أقدم تاريخ زيارة.
#### `scopeWithEarliestVisit(...)` – إضافة العمود والترتيب معاً.

### 5.7 نطاقات التصفية حسب المستخدم

#### `scopeVisitedByUser(Builder $query, $user = null, $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **الوصف**: تصفية الكائنات التي زارها مستخدم معين (إذا لم يُمرر `$user`، يستخدم المستخدم الحالي).
- **مثال**:
  ```php
  // المنتجات التي زارها المستخدم الحالي
  $products = Product::visitedByUser()->get();
  
  // المنتجات التي زارها مستخدم محدد في حدث معين
  $products = Product::visitedByUser($user, null, false, null, null, 'contest')->get();
  ```

#### `scopeNotVisitedByUser(...)` – تصفية الكائنات التي **لم** يزرها مستخدم معين.

### 5.8 نطاقات التصفية حسب وجود زيارات

#### `scopeHasVisits(Builder $query, $minCount = 1, $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **الوصف**: تصفية الكائنات التي لديها على الأقل `$minCount` من الزيارات (مع خيارات التصفية).
- **مثال**:
  ```php
  // منتجات لها 5 زيارات نشطة على الأقل
  $products = Product::hasVisits(5, true)->get();
  ```

#### `scopeHasNoVisits(...)` – تصفية الكائنات التي لا زيارات لها.

### 5.9 نطاق إضافة عمود حالة الزيارة للمستخدم الحالي

#### `scopeWithIsVisitedByUser(Builder $query, $user = null, $columnName = 'is_visited_by_user', $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **الوصف**: يضيف عموداً منطقياً (`1` أو `0`) إلى نتيجة الاستعلام يبين ما إذا كان المستخدم المحدد قد زار الكائن.
- **مثال**:
  ```php
  $products = Product::withIsVisitedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_visited_by_user) {
          echo "لقد زرت هذا المنتج";
      }
  }
  ```

### 5.10 نطاقات خاصة

#### `scopeTopVisited(Builder $query, $limit = 10, $orderDirection = 'DESC', $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **الوصف**: يجلب الكائنات الأكثر زيارة (حسب عدد السجلات) مع تحديد الحد.
- **مثال**:
  ```php
  $topProducts = Product::topVisited(5, 'DESC', true)->get();
  ```

#### `scopeTopSumVisits(Builder $query, $limit = 10, $orderDirection = 'DESC', $onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **الوصف**: يجلب الكائنات الأكثر زيارة (حسب مجموع `visits`) مع تحديد الحد.

### 5.11 دوال إحصائية مساعدة

#### `getTotalVisits($onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **الوصف**: إجمالي عدد الزيارات (مجموع `visits`) حسب خيارات التصفية.
- **المخرجات**: عدد صحيح.

#### `getTotalViews(...)` – إجمالي عدد المشاهدات (مجموع `views`).

#### `getVisitsCountByType($onlyActive = null, $withTrashed = false, $type = null, $processType = null, $eventType = null)`
- **الوصف**: توزيع الزيارات حسب نوع المستخدم (`user_type`).
- **المخرجات**: `Collection` حيث المفتاح هو `user_type` والقيمة هي العدد.
- **مثال**:
  ```php
  $counts = $product->getVisitsCountByType(true);
  // ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]
  ```

#### `getVisitorsUsers(...)`
- **الوصف**: قائمة المستخدمين الذين زاروا الكائن (مع تحميل العلاقة `user`).
- **المخرجات**: `Collection` من كائنات المستخدمين.

---

## 6. أمثلة عملية شاملة

### مثال 1: تسجيل زيارة جديدة

عبر الكلاس `Manager` (المرفق)، يمكن تسجيل زيارة للمنتج:

```php
use Nano2\Visitors\Classes\Manager;

$product = Product::find(1);
$user = Auth::getUser();

Manager::createVisits([
    'visitable' => $product,
    'user' => $user,
    'type' => 'add',
    'process_type' => 'in',
    'event_type' => 'product_view',
    'visits' => 1,
    'views' => 1,
]);
```

### مثال 2: ترتيب المنتجات حسب عدد الزيارات

```php
$topProducts = Product::topVisited(10, 'DESC', true)->get();
```

### مثال 3: إضافة عمود `is_visited_by_user` في قائمة المنتجات

```php
$products = Product::withIsVisitedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_visited_by_user ? 'زُرتَه' : 'لم تزره');
}
```

### مثال 4: تصفية المنتجات التي زارها المستخدم الحالي

```php
$myVisitedProducts = Product::visitedByUser()->get();
```

### مثال 5: استخدام التصفية حسب النوع والعملية والحدث

```php
// المنتجات التي زارها المستخدم الحالي من نوع 'add' (دخول) في حدث 'contest'
$products = Product::visitedByUser(null, null, false, 'add', 'in', 'contest')->get();
```

### مثال 6: إحصائيات الزيارات لمنتج معين

```php
$product = Product::find(1);

echo "إجمالي الزيارات: " . $product->getTotalVisits(true);
echo "إجمالي المشاهدات: " . $product->getTotalViews(true);
echo "توزيع الزيارات حسب نوع المستخدم: ";
print_r($product->getVisitsCountByType(true)->toArray());

$visitors = $product->getVisitorsUsers(true);
foreach ($visitors as $user) {
    echo $user->name . "\n";
}
```

### مثال 7: الجمع بين النطاقات المتقدمة

```php
// المنتجات التي لها أكثر من 5 زيارات نشطة من نوع 'add'، مرتبة حسب أحدث زيارة
$products = Product::hasVisits(5, true, false, 'add')
    ->sortByLatestVisit('DESC', 'last_visit')
    ->get();
```

---

## 7. دوال التوافق العكسي

للحفاظ على الكود القديم الذي يستخدم أسماء النطاقات السابقة، تم توفير دوال تغليف في الكلاس الرئيسي. هذه الدوال موصوفة بـ `@deprecated` وينصح باستخدام البدائل الجديدة.

**قائمة الدوال القديمة المدعومة:**

| الدالة القديمة | الدالة الجديدة |
|----------------|----------------|
| `scopeSortByCountVisitsOld` | `scopeSortByCountVisits` |
| `scopeWithSortByCountVisits` | `scopeWithCountVisits` |
| `scopeSortBySumVisitsOld` | `scopeSortBySumVisits` |
| `scopeWithSortBySumVisits` | `scopeWithSumVisits` |
| `scopeSortBySumViewsOld` | `scopeSortBySumViews` |
| `scopeWithSortBySumViews` | `scopeWithSumViews` |
| `scopeWhereHasVisit` | `scopeVisitedByUser` |
| `scopeWhereHasVisits` | `scopeVisitedByUser` |

**ملاحظة:** الدوال القديمة `scopeWhereHasVisit` و `scopeWhereHasVisits` تم تحسينها لدعم معامل `$isForceUser` للتحكم في سلوك الاستعلام عند عدم وجود مستخدم، مع الحفاظ على السلوك الأصلي (عدم إرجاع نتائج) كقيمة افتراضية.

---

## 8. تحسين الأداء والتخزين المؤقت

السلوك يعتمد على استعلامات فرعية (subqueries) محسنة لتجنب تحميل العلاقات بالكامل. لا يوجد نظام تخزين مؤقت مدمج بشكل افتراضي، لكن يمكن الاعتماد على آلية التخزين المؤقت للموديلات نفسها أو استخدام ذاكرة التخزين المؤقت للتطبيق إذا احتجت لتكرار استعلامات ثقيلة.

- عند استخدام `withCountVisits` أو `sortByCountVisits`، يتم تنفيذ استعلام فرعي واحد فقط.
- استخدام `withIsVisitedByUser` يضيف استعلاماً فرعياً لكل صف، لذا يُنصح باستخدامه بحذر في القوائم الكبيرة، لكنه عادةً ما يكون مقبولاً بسبب فاعلية استعلامات SQL.

**ملاحظة حول `withTrashed`:** عند استخدام `withTrashed = true` في أي من النطاقات أو الدوال الإحصائية، تأكد من أن موديل `Nano2\Visitors\Models\Visit` يستخدم الترايت `SoftDeletes`؛ وإلا ستظهر أخطاء في الاستعلام.

---

## 9. الخاتمة

`VisitModel` هو حل متكامل وقوي لإدارة تتبع الزيارات والمشاهدات في تطبيقات نانوسوفت. بفضل تنوع النطاقات والدوال المساعدة، يمكن للمطورين بناء أنظمة تحليلات متقدمة بسهولة مع الاحتفاظ بمرونة عالية وقابلية للتوسع. التوافق العكسي يضمن انتقالاً سلساً من الإصدارات السابقة. نأمل أن يكون هذا التوثيق مرجعاً شاملاً لجميع إمكانيات السلوك.