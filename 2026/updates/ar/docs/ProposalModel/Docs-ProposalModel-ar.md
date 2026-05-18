# توثيق سلوك `ProposalModel` 

---

## 1. مقدمة

`ProposalModel` هو سلوك متقدم يضيف نظام بلاغات ومقترحات وشكاوى متعدد الأشكال (polymorphic) إلى أي موديل في تطبيقات نانوسوفت. يتيح هذا السلوك للمستخدمين إرسال بلاغات (reports) أو مقترحات (proposals) أو شكاوى (complaints) حول كائنات متنوعة (مثل المنتجات، المقالات، المستخدمين)، مع دعم كامل للجانبين: **الهدف (target)** الذي يُرسل البلاغ ضده، و**المرسل (user)** الذي يرسل البلاغ. يأتي السلوك مزوداً بمجموعة غنية من النطاقات والدوال المساعدة التي تسهل بناء أنظمة بلاغات متطورة بمرونة وأداء عالٍ.

### الفوائد الرئيسية

- **نظام بلاغات متعدد الأشكال** – يمكن جعل أي موديل قابلاً لتلقي البلاغات (كهدف) أو إرسالها (كمستخدم).
- **دعم أنواع متعددة** – يمكن تمييز البلاغات عبر حقل `type` (proposals, reports, complaints).
- **مرونة في الحالات** – دعم حقول `is_active` (نشاط البلاغ) و `deleted_at` (الحذف الناعم).
- **نطاقات ترتيب متقدمة** – ترتيب الكائنات حسب عدد البلاغات، أحدث بلاغ، أو أقدم بلاغ، مع تصفية حسب النوع والنشاط.
- **نطاقات تصفية** – تصفية الكائنات التي أرسل لها المستخدم بلاغاً (كهدف)، أو التي أرسلت بلاغاً (كمستخدم)، أو التي لها بلاغات، أو التي لا بلاغات لها.
- **إضافة عمود حالة للمستخدم الحالي** – `is_proposed_by_user` (هل أرسل المستخدم بلاغاً للكائن) و `is_proposed_to_user` (هل الكائن هو هدف بلاغ أرسله المستخدم).
- **نطاق متقدم** (`scopeWhereHasProposalAdvanced`) – يوفر مرونة كاملة للتصفية حسب الجانب (target/user)، ونوع العلاقة (has, orHas, doesntHave, orDoesntHave)، وشروط مرنة (مصفوفة أو دالة إغلاق)، مع دعم `withTrashed`/`onlyTrashed`.
- **دوال إحصائية مساعدة** – للحصول على إجمالي البلاغات، توزيعها حسب نوع المستخدم، وقائمة المستخدمين المبلّغين.
- **واجهة متسقة** – دوال مشابهة لسلوكيات المفضلات والإعجابات والتفاعلات (FavoriteableModel, LikeableModel, ReactionableModel, BookmarkableModel) لتجربة تطوير موحدة.
- **تحسين الأداء** – استخدام استعلامات فرعية (subqueries) فعالة وتجنب تحميل العلاقات بالكامل حيثما أمكن.
- **توافق عكسي** – الاحتفاظ بأسماء النطاقات القديمة (`sortByCountProposalsOld`, `withSortByCountProposals`, `whereHasProposal`، إلخ) لضمان عدم كسر الكود القديم.

---

## 2. متطلبات الاستخدام

1. وجود موديل `Nano2\Proposals\Models\Proposal` (الذي يمثل سجل البلاغ) مع جدول `tss_proposals_proposals`.
2. الموديل المراد جعله قابلاً للبلاغات يجب أن يكون من نوع Eloquent (أو `October\Rain\Database\Model` في حالة استخدام OctoberCMS).
3. أن يكون المستخدمون (مثل موديل `User`) قادرين على استخدام نظام البلاغات (أي أنهم يستوفون واجهة `Authenticatable`).
4. اختياري: استخدام `SoftDeletes` في موديل `Proposal` للاستفادة من خيار `withTrashed`.

---

## 3. إضافة السلوك إلى موديل

يمكن إضافة السلوك إلى أي موديل باستخدام خاصية `$implement` (في سياق نانوسوفت). السلوك يتضمن Trait يحمل النطاقات والدوال المساعدة، لذا بعد الإضافة يصبح الموديل قادراً على استخدام جميع الميزات.

### مثال: جعل موديل `Product` قابلاً لتلقي البلاغات (كهدف) وإرسالها (كمستخدم)

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano2\Proposals\Behaviors\ProposalModel;

class Product extends Model
{
    public $implement = [
        ProposalModel::class
    ];
}
```

**ملاحظة:** إذا كان الموديل يستخدم نظام `ExtensionBase` (كما هو الحال في تطبيقات نانوسوفت)، فإن الإضافة تتم عبر `$implement` كما هو موضح.

---

## 4. الهيكل الأساسي

بعد إضافة السلوك، يصبح للموديل العلاقات التالية:

| العلاقة | الوصف |
|---------|-------|
| `all_proposals` | جميع سجلات البلاغات (الواردة إلى الكائن كهدف) |
| `proposals` | البلاغات من نوع `proposals` فقط (الواردة) |
| `reports` | البلاغات من نوع `reports` فقط (الواردة) |
| `complaints` | البلاغات من نوع `complaints` فقط (الواردة) |
| `all_user_proposals` | جميع سجلات البلاغات (المرسلة من الكائن كمستخدم) |
| `user_proposals` | البلاغات من نوع `proposals` المرسلة |
| `user_reports` | البلاغات من نوع `reports` المرسلة |
| `user_complaints` | البلاغات من نوع `complaints` المرسلة |

كما يتوفر خصائص محسوبة ديناميكية (تُضاف تلقائياً):

- `$product->proposals_count` – عدد البلاغات (يمكن تمرير `type` عبر الباراميتر).
- `$product->is_proposed_by_user()` – هل المستخدم الحالي أرسل بلاغاً لهذا الكائن (كهدف).
- `$product->is_proposed_to_user()` – هل هذا الكائن (كمستخدم) أرسل بلاغاً.
- `$product->getUserProposal()` – إرجاع سجل البلاغ الخاص بالمستخدم الحالي (كهدف).

---

## 5. الدوال والنطاقات المتاحة

### 5.1 دوال بناء الاستعلامات الفرعية (داخلية)

#### `protected function buildProposalCountSubQuery($type = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يبني استعلام فرعي لحساب عدد سجلات البلاغات (COUNT) للكائن الحالي كهدف.
- **المعاملات**:
  - `$type` (string|null): تصفية حسب نوع البلاغ (`proposals`, `reports`, `complaints`).
  - `$onlyActive` (bool|null): `true` للبلاغات النشطة فقط، `false` للغير نشطة، `null` للكل.
  - `$withTrashed` (bool): تضمين البلاغات المحذوفة.

#### `protected function buildProposalLatestDateSubQuery(...)` – تستخدم `MAX(field)` للحصول على أحدث تاريخ بلاغ.
#### `protected function buildProposalEarliestDateSubQuery(...)` – تستخدم `MIN(field)` للحصول على أقدم تاريخ بلاغ.

### 5.2 نطاقات عدد البلاغات (COUNT)

#### `scopeAddCountProposals(Builder $query, $orderDirection = 'DESC', $columnName = 'proposals_count', $type = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يضيف عموداً يحتوي على عدد سجلات البلاغات إلى نتيجة الاستعلام (بدون ترتيب).
- **مثال**:
  ```php
  $products = Product::addCountProposals()->get();
  foreach ($products as $product) {
      echo $product->proposals_count;
  }
  ```

#### `scopeSortByCountProposals(...)` – يرتب النتائج حسب عدد البلاغات (دون إضافة العمود).
#### `scopeWithCountProposals(...)` – يجمع بين `addCountProposals` و `sortByCountProposals` (إضافة العمود والترتيب معاً).

**مثال**:
```php
$products = Product::withCountProposals('DESC', 'proposals_count', 'reports')->get(); // بلاغات من نوع reports فقط
```

### 5.3 نطاقات تاريخ البلاغ

#### `scopeAddLatestProposal(...)` – يضيف عموداً بأحدث تاريخ بلاغ (اختيارياً تحديد الحقل عبر `$field`).
#### `scopeSortByLatestProposal(...)` – ترتيب حسب أحدث تاريخ بلاغ.
#### `scopeWithLatestProposal(...)` – إضافة العمود والترتيب معاً.

**مثال**:
```php
$products = Product::withLatestProposal('DESC', 'last_proposal', 'complaints')->get();
```

#### `scopeAddEarliestProposal(...)` – يضيف عموداً بأقدم تاريخ بلاغ.
#### `scopeSortByEarliestProposal(...)` – ترتيب حسب أقدم تاريخ بلاغ.
#### `scopeWithEarliestProposal(...)` – إضافة العمود والترتيب معاً.

### 5.4 نطاقات التصفية حسب المستخدم (جانب الهدف – target)

#### `scopeProposedByUser(Builder $query, $user = null, $type = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: تصفية الكائنات التي أرسل لها المستخدم المحدد بلاغاً (كهدف). إذا لم يُمرر `$user`، يستخدم المستخدم الحالي.
- **مثال**:
  ```php
  // المنتجات التي أرسل لها المستخدم الحالي بلاغاً
  $products = Product::proposedByUser()->get();
  
  // المنتجات التي أرسل لها مستخدم محدد بلاغاً من نوع 'complaints'
  $products = Product::proposedByUser($user, 'complaints')->get();
  ```

#### `scopeNotProposedByUser(...)` – تصفية الكائنات التي **لم** يرسل لها المستخدم بلاغاً.

### 5.5 نطاقات التصفية حسب المستخدم (جانب المرسل – user)

#### `scopeProposedToUser(Builder $query, $user = null, $type = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: تصفية الكائنات التي أرسلت بلاغاً (كمستخدم). إذا لم يُمرر `$user`، يستخدم المستخدم الحالي.
- **مثال**:
  ```php
  // المستخدمون الذين أرسلوا بلاغات
  $users = User::proposedToUser()->get();
  
  // المستخدمون الذين أرسلوا بلاغات من نوع 'reports'
  $users = User::proposedToUser(null, 'reports')->get();
  ```

#### `scopeNotProposedToUser(...)` – تصفية الكائنات التي **لم** ترسل بلاغاً.

### 5.6 نطاقات التصفية حسب وجود بلاغات

#### `scopeHasProposals(Builder $query, $minCount = 1, $type = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: تصفية الكائنات التي لديها على الأقل `$minCount` من البلاغات (مع خيارات التصفية).
- **مثال**:
  ```php
  // منتجات لها 5 بلاغات من نوع 'complaints' على الأقل
  $products = Product::hasProposals(5, 'complaints')->get();
  ```

#### `scopeHasNoProposals(...)` – تصفية الكائنات التي لا بلاغات لها.

### 5.7 نطاقات إضافة عمود الحالة للمستخدم الحالي

#### `scopeWithIsProposedByUser(Builder $query, $user = null, $columnName = 'is_proposed_by_user', $type = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يضيف عموداً منطقياً (`1` أو `0`) إلى نتيجة الاستعلام يبين ما إذا كان المستخدم المحدد قد أرسل بلاغاً للكائن.
- **مثال**:
  ```php
  $products = Product::withIsProposedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_proposed_by_user) {
          echo "لقد أرسلت بلاغاً لهذا المنتج";
      }
  }
  ```

#### `scopeWithIsProposedToUser(...)` – يضيف عموداً يوضح ما إذا كان المستخدم المحدد هو هدف بلاغ أرسله الكائن.

### 5.8 نطاق متقدم `scopeWhereHasProposalAdvanced`

هذا النطاق هو الأقوى والأكثر مرونة، ويسمح بالتصفية حسب البلاغات مع خيارات غير محدودة. يتم تمرير مصفوفة خيارات تحتوي على:

| الخيار | النوع | الوصف |
|--------|-------|-------|
| `side` | string | `target` (بلاغات واردة إلى الكائن) أو `user` (بلاغات صادرة من الكائن). افتراضي `target`. |
| `relation` | string | اسم العلاقة المراد استخدامها (تحدد تلقائياً إن لم تُمرر). |
| `type` | string | نوع العلاقة: `has`, `orHas`, `doesntHave`, `orDoesntHave`. افتراضي `has`. |
| `conditions` | array\|callable | شروط إضافية على جدول البلاغات. يمكن أن تكون مصفوفة من الحقول والقيم، أو دالة إغلاق، أو مصفوفة تدعم الاستعلامات المتقدمة (مثل `['whereIn', [1,2,3]]`). |
| `user` | Model\|null | المستخدم المحدد (لجانب user). |
| `target` | Model\|null | الهدف المحدد (لجانب target). |
| `getUserFromHelpers` | bool | هل نحاول جلب المستخدم الحالي إذا كان `user = null`؟ افتراضي `true`. |
| `getTargetFromHelpers` | bool | هل نحاول جلب الهدف من الموديل الحالي إذا كان `target = null`؟ افتراضي `true`. |
| `userSource` | string | مصدر المستخدم: `'current_user'` (من AuthHelpers) أو `'current_model'` (الموديل الحالي). يقرأ من الإعدادات. |
| `targetSource` | string | مصدر الهدف: `'current_user'` أو `'current_model'`. يقرأ من الإعدادات. |
| `isForceUser` | bool\|null | عند عدم وجود مستخدم: `true` يضيف شرطاً مستحيلاً (لا نتائج)، `false` لا يضيف شرطاً. |
| `isForceTarget` | bool\|null | عند عدم وجود هدف: `true` يضيف شرطاً مستحيلاً، `false` لا يضيف شرطاً. |
| `useUserConditions` | bool\|array | تطبيق شروط المستخدم (`user_id`, `user_type`). يمكن أن يكون `true` (الكل)، `false` (لا شيء)، أو مصفوفة تحتوي على أسماء الحقول المطلوبة (مثل `['user_type']`). |
| `useTargetConditions` | bool\|array | تطبيق شروط الهدف (`target_id`, `target_type`). |

**مثال**:
```php
// المنتجات التي لديها بلاغات من نوع 'reports' (أي مستخدم)
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'reports']
])->get();

// المنتجات التي لديها بلاغات نشطة من نوع 'complaints' للمستخدم الحالي
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'complaints', 'is_active' => true]
])->get();

// المنتجات التي ليس لها بلاغات من نوع 'proposals'
$products = Product::whereHasProposalAdvanced([
    'type' => 'doesntHave',
    'conditions' => ['type' => 'proposals']
])->get();

// استخدام دالة إغلاق
$products = Product::whereHasProposalAdvanced([
    'conditions' => function($q) {
        $q->where('type', 'reports')
          ->where('created_at', '>=', Carbon::now()->subWeek());
    }
])->get();
```

### 5.9 نطاقات خاصة

#### `scopeTopProposed(Builder $query, $limit = 10, $orderDirection = 'DESC', $type = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يجلب الكائنات الأكثر بلاغاً (حسب عدد البلاغات) مع تحديد الحد.
- **مثال**:
  ```php
  $topProducts = Product::topProposed(10, 'DESC', 'complaints')->get();
  ```

### 5.10 دوال إحصائية مساعدة

#### `getTotalProposals($type = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: إجمالي عدد البلاغات (مجموع `COUNT`) حسب خيارات التصفية.
- **المخرجات**: عدد صحيح.

#### `getProposalsCountByType($type = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: توزيع البلاغات حسب نوع المستخدم (`user_type`).
- **المخرجات**: `Collection` حيث المفتاح هو `user_type` والقيمة هي العدد.

#### `getProposersUsers($type = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: قائمة المستخدمين الذين أرسلوا بلاغات للكائن (مع تحميل العلاقة `user`).
- **المخرجات**: `Collection` من كائنات المستخدمين.

### 5.11 دوال أساسية في السلوك (غير موجودة في الـ Trait)

#### `proposaledBy()`
- **الوصف**: يعيد مجموعة المستخدمين الذين أرسلوا بلاغات للكائن الحالي (كهدف).
- **المخرجات**: `Collection` من المستخدمين (مع تحميل `user`).

#### `isUserProposal($user = null)`
- **الوصف**: يتحقق مما إذا كان المستخدم (أو المستخدم الحالي) قد أرسل بلاغاً للكائن.
- **المخرجات**: `bool`.

#### `getUserProposal($user = null)`
- **الوصف**: يعيد سجل البلاغ الخاص بالمستخدم (إن وجد).
- **المخرجات**: كائن `Proposal` أو `null`.

#### `addProposal($data, $user = null, $parent = null): Proposal`
- **الوصف**: إضافة بلاغ جديد للكائن. يمكن تمرير `$data` كـ `array` أو كـ `string` (سيتم تفسيره كـ `content`).
- **المخرجات**: كائن `Proposal` المُنشأ.

#### `toggleProposal($data, $user = null, $parent = null)`
- **الوصف**: إذا كان المستخدم قد أرسل بلاغاً، يقوم بحذفه (أو استعادته إذا كان محذوفاً)؛ وإلا يضيف بلاغاً جديداً.
- **المخرجات**: كائن `Proposal` أو `null` عند الحذف.

#### `updateProposal($id, $data): Proposal`
- **الوصف**: تحديث بيانات بلاغ موجود.

#### `deleteProposal($id): ?bool`
- **الوصف**: حذف بلاغ معين (Soft delete).

#### `countProposal(bool $onlyAccepted = false)`
- **الوصف**: عدد البلاغات (اختيارياً المقبولة فقط – غير مستخدم حالياً).

---

## 6. أمثلة عملية شاملة

### مثال 1: إضافة بلاغ والتحقق

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano2\Proposals\Models\Proposal;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة بلاغ من نوع 'reports'
$report = $product->addProposal([
    'type' => 'reports',
    'content' => 'هذا المنتج يحتوي على معلومات خاطئة',
    'is_active' => true,
], $user);

// إضافة مقترح
$proposal = $product->addProposal([
    'type' => 'proposals',
    'content' => 'أقترح إضافة خاصية جديدة',
], $user);

// التحقق مما إذا كان المستخدم قد أرسل بلاغاً لهذا المنتج
if ($product->isUserProposal()) {
    echo "لقد أرسلت بلاغاً لهذا المنتج";
}
```

### مثال 2: ترتيب الكائنات حسب عدد البلاغات

```php
$topProducts = Product::topProposed(10)->get();
```

### مثال 3: إضافة عمود حالة البلاغ للمستخدم الحالي

```php
$products = Product::withIsProposedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_proposed_by_user ? 'أرسلت بلاغاً' : 'لم ترسل بلاغاً');
}
```

### مثال 4: تصفية الكائنات التي أرسل لها المستخدم بلاغاً (جانب target)

```php
$myReportedProducts = Product::proposedByUser()->get();
```

### مثال 5: تصفية الكائنات التي أرسلت بلاغاً (جانب user)

```php
$usersWhoReported = User::proposedToUser()->get();
```

### مثال 6: الإحصائيات

```php
$product = Product::find(1);
echo "إجمالي البلاغات من نوع 'reports': " . $product->getTotalProposals('reports');
print_r($product->getProposalsCountByType('reports')->toArray());
$users = $product->getProposersUsers('reports');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### مثال 7: الجمع بين النطاقات

```php
// المنتجات التي لها أكثر من 5 بلاغات من نوع 'complaints'، مرتبة حسب أحدث بلاغ
$products = Product::hasProposals(5, 'complaints')
    ->sortByLatestProposal('DESC', 'latest_complaint_at')
    ->get();
```

### مثال 8: استخدام النطاق المتقدم `whereHasProposalAdvanced`

```php
// المنتجات التي لديها بلاغات من نوع 'reports' (أي مستخدم)
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'reports']
])->get();

// المنتجات التي لديها بلاغات نشطة من نوع 'complaints' للمستخدم الحالي
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'complaints', 'is_active' => true]
])->get();

// المنتجات التي ليس لها بلاغات من نوع 'proposals'
$products = Product::whereHasProposalAdvanced([
    'type' => 'doesntHave',
    'conditions' => ['type' => 'proposals']
])->get();

// استخدام دالة إغلاق مع شروط تاريخية
$products = Product::whereHasProposalAdvanced([
    'conditions' => function($q) {
        $q->where('type', 'reports')
          ->where('created_at', '>=', Carbon::now()->subWeek());
    }
])->get();

// استخدام whereIn على نوع البلاغ
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => ['whereIn', ['reports', 'complaints']],
        'is_active' => true,
    ]
])->get();

// تضمين البلاغات المحذوفة
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => 'complaints',
        'withTrashed' => true,
    ]
])->get();

// التحكم في مصدر المستخدم (جانب user)
$users = User::whereHasProposalAdvanced([
    'side' => 'user',
    'userSource' => 'current_model', // استخدام الموديل الحالي كمستخدم
])->get();

// تعطيل شروط المستخدم (البحث عن أي بلاغات بغض النظر عن المستخدم)
$products = Product::whereHasProposalAdvanced([
    'useUserConditions' => false,
    'conditions' => ['type' => 'reports']
])->get();
```

---

## 7. دوال التوافق العكسي

للحفاظ على الكود القديم الذي يستخدم أسماء النطاقات السابقة، تم توفير دوال تغليف في الكلاس الرئيسي. هذه الدوال موصوفة بـ `@deprecated` وينصح باستخدام البدائل الجديدة.

**قائمة الدوال القديمة المدعومة:**

| الدالة القديمة | الدالة الجديدة |
|----------------|----------------|
| `scopeSortByCountProposalsOld` | `scopeSortByCountProposals` |
| `scopeWithSortByCountProposals` | `scopeWithCountProposals` |
| `scopeAddSortByCountProposals` | `scopeAddCountProposals` |
| `scopeSortByCreatedAtProposals` | `scopeSortByLatestProposal` |
| `scopeAddSortByCreatedAtProposals` | `scopeAddLatestProposal` |
| `scopeWithSortByCreatedAtProposals` | `scopeWithLatestProposal` |
| `scopeWhereHasProposal` | `scopeWhereHasProposalAdvanced` (جانب target) |
| `scopeWhereHasProposals` | `scopeWhereHasProposalAdvanced` |
| `scopeWhereHasUserProposal` | `scopeWhereHasProposalAdvanced` (جانب user) |
| `scopeWhereHasUserProposals` | `scopeWhereHasProposalAdvanced` |

**ملاحظة حول الدوال القديمة `whereHasProposal` ونظيراتها:** تم تحسينها لتوجه الاستدعاءات إلى النطاق المتقدم `scopeWhereHasProposalAdvanced` مع الحفاظ على السلوك الأصلي.

---

## 8. تحسين الأداء والتخزين المؤقت

السلوك يعتمد على استعلامات فرعية (subqueries) محسنة لتجنب تحميل العلاقات بالكامل. لا يوجد نظام تخزين مؤقت مدمج بشكل افتراضي، لكن يمكن الاعتماد على آلية التخزين المؤقت للموديلات نفسها أو استخدام ذاكرة التخزين المؤقت للتطبيق إذا احتجت لتكرار استعلامات ثقيلة.

- عند استخدام `withCountProposals` أو `sortByCountProposals`، يتم تنفيذ استعلام فرعي واحد فقط.
- استخدام `withIsProposedByUser` أو `withIsProposedToUser` يضيف استعلاماً فرعياً لكل صف، لذا يُنصح باستخدامه بحذر في القوائم الكبيرة، لكنه عادةً ما يكون مقبولاً بسبب فاعلية استعلامات SQL.

**ملاحظة حول `withTrashed`:** عند استخدام `withTrashed = true` في أي من النطاقات أو الدوال الإحصائية، تأكد من أن موديل `Nano2\Proposals\Models\Proposal` يستخدم الترايت `SoftDeletes`؛ وإلا ستظهر أخطاء في الاستعلام.

---

## 9. الخاتمة

`ProposalModel` هو حل متكامل وقوي لإدارة نظام البلاغات والمقترحات والشكاوى في تطبيقات نانوسوفت. بفضل تنوع النطاقات والدوال المساعدة، يمكن للمطورين بناء أنظمة بلاغات متطورة بسهولة مع الاحتفاظ بمرونة عالية وقابلية للتوسع. النطاق المتقدم `scopeWhereHasProposalAdvanced` يوفر أقصى درجات التحكم في الاستعلامات المعقدة. التوافق العكسي يضمن انتقالاً سلساً من الإصدارات السابقة. نأمل أن يكون هذا التوثيق مرجعاً شاملاً لجميع إمكانيات السلوك.