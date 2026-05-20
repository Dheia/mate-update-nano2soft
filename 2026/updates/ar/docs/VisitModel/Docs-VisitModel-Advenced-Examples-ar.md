# أمثلة عملية متقدمة لاستخدام سلوك `VisitModel`

يقدم هذا القسم مجموعة من الأمثلة المتكاملة التي توضح كيفية استخدام سلوك `VisitModel` في سيناريوهات حقيقية، بدءاً من العمليات الأساسية وصولاً إلى الاستعلامات المعقدة والإحصائيات المتقدمة.

---

### 1. العمليات الأساسية

#### 1.1 تسجيل زيارة جديدة مع بيانات مخصصة

```php
use Nano2\Visitors\Classes\Manager;
use Nano\AuthApi\Classes\AuthHelpers;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// تسجيل زيارة عادية (نوع add، عملية in)
Manager::createVisits([
    'visitable' => $product,
    'user' => $user,
    'type' => 'add',
    'process_type' => 'in',
    'event_type' => 'product_view',
    'visits' => 1,
    'views' => 1,
]);

// تسجيل زيارة مع بيانات إضافية (مثلاً سبب الزيارة في حقل other_data)
Manager::createVisits([
    'visitable' => $product,
    'user' => $user,
    'type' => 'add',
    'process_type' => 'in',
    'event_type' => 'special_offer',
    'visits' => 1,
    'views' => 2,
    'other_data' => json_encode(['source' => 'email_campaign']),
]);
```

#### 1.2 تحديث إحصائيات الزيارة (إضافة مشاهدات أو زيارات)

```php
$product = Product::find(1);
$user = AuthHelpers::getCurrentUser();

// جلب سجل الزيارة الحالي
$visit = $product->getVisit($user);

if ($visit) {
    // زيادة عدد المشاهدات
    $visit->views += 1;
    $visit->save();
}
```

#### 1.3 حذف سجل زيارة (Soft Delete)

```php
$visit = Visit::find($visitId);
if ($visit) {
    $visit->delete();
}
```

---

### 2. التصفية حسب المستخدمين

#### 2.1 جلب جميع المنتجات التي زارها المستخدم الحالي

```php
$myVisitedProducts = Product::visitedByUser()->get();

// مع تضمين الزيارات غير النشطة فقط
$myVisitedProducts = Product::visitedByUser(null, false)->get();

// مع تضمين الزيارات المحذوفة (Soft Deletes)
$myVisitedProducts = Product::visitedByUser(null, null, true)->get();
```

#### 2.2 جلب المنتجات التي لم يزرها المستخدم الحالي (لاقتراحها)

```php
$suggestedProducts = Product::notVisitedByUser()->paginate(20);
```

#### 2.3 استخدام النطاق القديم `whereHasVisit` (للتوافق العكسي)

```php
$products = Product::whereHasVisit($user)->get();
```

#### 2.4 جلب المنتجات التي زارها مستخدم معين مع تحميل تفاصيل الزيارة

```php
$user = User::find(2);
$products = Product::visitedByUser($user)
    ->with(['visits' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

---

### 3. الترتيب والتصنيف المتقدم

#### 3.1 ترتيب المنتجات حسب عدد الزيارات (عدد السجلات)

```php
// باستخدام النطاق المباشر
$topProducts = Product::sortByCountVisits('DESC', 'visits_count', true)->take(10)->get();

// باستخدام الاختصار topVisited
$topProducts = Product::topVisited(10)->get();

// ترتيب حسب عدد الزيارات النشطة فقط من نوع معين
$topActive = Product::sortByCountVisits('DESC', 'visits_count', true, false, 'add')->get();
```

#### 3.2 ترتيب المنتجات حسب مجموع الزيارات (حقل `visits`)

```php
$products = Product::sortBySumVisits('DESC', 'total_visits', true)->get();
```

#### 3.3 ترتيب المنتجات حسب مجموع المشاهدات (حقل `views`)

```php
$products = Product::sortBySumViews('DESC', 'total_views')->get();
```

#### 3.4 ترتيب حسب آخر تاريخ زيارة (الأحدث أولاً)

```php
$products = Product::sortByLatestVisit('DESC')->get();
```

#### 3.5 إضافة عمود آخر تاريخ زيارة مع الترتيب

```php
$products = Product::withLatestVisit('DESC', 'last_visit_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - آخر زيارة: ' . $product->last_visit_date;
}
```

---

### 4. الإحصائيات والتقارير

#### 4.1 إجمالي عدد الزيارات والمشاهدات لمنتج معين

```php
$product = Product::find(1);

// كل الزيارات
$totalVisits = $product->getTotalVisits();

// الزيارات النشطة فقط
$totalActiveVisits = $product->getTotalVisits(true);

// الزيارات النشطة من نوع add فقط
$totalAddVisits = $product->getTotalVisits(true, false, 'add');

// مجموع المشاهدات
$totalViews = $product->getTotalViews(true);
```

#### 4.2 توزيع الزيارات حسب نوع المستخدم

```php
$counts = $product->getVisitsCountByType(true);
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]
```

#### 4.3 جلب قائمة بأسماء المستخدمين الذين زاروا منتجاً معيناً

```php
$users = $product->getVisitorsUsers(true);
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 إحصائيات الزيارات في نطاق زمني (باستخدام النطاقات مع where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// المنتجات التي زُورت خلال الأسبوع الماضي
$products = Product::whereHas('visits', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. الجمع بين النطاقات لاستعلامات معقدة

#### 5.1 المنتجات الأكثر زيارة (عدد السجلات) والتي زارها المستخدم الحالي أيضاً

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topVisited(20)
    ->visitedByUser($user)
    ->withIsVisitedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (عدد الزيارات: ' . $product->visits_count . ') - ';
    echo $product->is_visited_by_user ? 'لقد زُرتَه' : 'لم تزره';
}
```

#### 5.2 المنتجات التي لها أكثر من 5 زيارات نشطة، مرتبة حسب أحدث زيارة (مع إضافة العمود)

```php
$products = Product::hasVisits(5, true)
    ->withLatestVisit('DESC', 'last_visit_date')
    ->get();
```

#### 5.3 المنتجات النشطة التي زارها المستخدم الحالي، مع إضافة عمود مجموع الزيارات

```php
$products = Product::active()  // افتراض وجود نطاق active في الموديل
    ->visitedByUser()
    ->withSumVisits('DESC', 'total_visits')
    ->get();
```

#### 5.4 المنتجات التي زارها المستخدم الحالي من نوع معين (add) في حدث محدد

```php
$products = Product::visitedByUser(null, null, false, 'add', null, 'contest')->get();
```

#### 5.5 المنتجات التي ليس لها زيارات نشطة، مرتبة حسب الاسم

```php
$products = Product::hasNoVisits(true)->orderBy('name')->get();
```

---

### 6. استخدام الدوال المساعدة في القوالب (مثال Twig/Blade)

#### 6.1 عرض عدد الزيارات والمشاهدات لمنتج

```php
@php
$product = Product::find(1);
$totalVisits = $product->getTotalVisits(true);
$totalViews = $product->getTotalViews(true);
$isVisited = $product->is_visits;
@endphp

<div class="product-stats">
    <span>عدد الزيارات: {{ $totalVisits }}</span>
    <span>عدد المشاهدات: {{ $totalViews }}</span>
    <span>{{ $isVisited ? 'تمت زيارته' : 'لم تتم زيارته' }}</span>
</div>
```

#### 6.2 عرض قائمة المنتجات الأكثر زيارة مع مؤشر للمستخدم الحالي

```php
$products = Product::topVisited(10)
    ->withIsVisitedByUser()
    ->get();
?>

<div class="top-products">
    @foreach($products as $product)
        <div class="product-item">
            <h3>{{ $product->name }}</h3>
            <p>الزيارات: {{ $product->visits_count }}</p>
            <span class="visit-status">
                {{ $product->is_visited_by_user ? 'زرته' : 'لم تزره' }}
            </span>
        </div>
    @endforeach
</div>
```

---

### 7. أمثلة على واجهات API (JSON)

#### 7.1 إرجاع تفاصيل المنتج مع إحصائيات الزيارات

```php
public function show($id)
{
    $product = Product::withIsVisitedByUser()
        ->withSumVisits('DESC', 'total_visits')
        ->withSumViews('DESC', 'total_views')
        ->find($id);
    
    return response()->json($product);
}
```

النتيجة:
```json
{
    "id": 1,
    "name": "هاتف ذكي",
    "total_visits": 245,
    "total_views": 1230,
    "is_visited_by_user": 1
}
```

#### 7.2 قائمة المنتجات الأكثر زيارة مع إضافة عمود الزيارة للمستخدم

```php
public function topVisited()
{
    $products = Product::topVisited(10)
        ->withIsVisitedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API لتسجيل زيارة (عبر Manager)

```php
public function recordVisit($productId)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    
    $result = Manager::createVisits([
        'visitable' => $product,
        'user' => $user,
        'type' => 'add',
        'process_type' => 'in',
        'event_type' => 'api_view',
        'visits' => 1,
        'views' => 1,
    ]);
    
    return response()->json([
        'success' => $result['status'],
        'message' => $result['message'],
    ]);
}
```

---

### 8. التوافق العكسي: استخدام النطاقات القديمة

إذا كنت تعتمد على كود قديم يستخدم النطاقات القديمة، يمكنك الاستمرار في استخدامها، لكن يُنصح بالترقية تدريجياً.

```php
// قديم
$products = Product::sortByCountVisitsOld('DESC')->get();

// جديد
$products = Product::sortByCountVisits('DESC')->get();
```

```php
// قديم
$products = Product::withSortBySumVisits('DESC')->get();

// جديد
$products = Product::withSumVisits('DESC')->get();
```

```php
// قديم (whereHasVisit)
$products = Product::whereHasVisit($user)->get();

// جديد (visitedByUser)
$products = Product::visitedByUser($user)->get();
```

---

### 9. نصائح وحيل متقدمة

#### 9.1 استخدام `withTrashed` في الإحصائيات لتضمين الزيارات المحذوفة

```php
$totalWithDeleted = $product->getTotalVisits(null, true);
```

#### 9.2 الحصول على عدد الزيارات حسب نوع العملية

```php
$inVisits = $product->getTotalVisits(true, false, null, 'in');
$outVisits = $product->getTotalVisits(true, false, null, 'out');
```

#### 9.3 استخدام النطاق `hasVisits` مع حد أدنى متغير من التحكم

```php
$minCount = request('min_visits', 5);
$products = Product::hasVisits($minCount, true)->get();
```

#### 9.4 تخصيص اسم العمود في النطاقات

```php
$products = Product::withCountVisits('DESC', 'my_visits_column', true)->get();
echo $products->first()->my_visits_column;
```

#### 9.5 الجمع بين `visitedByUser` و `notVisitedByUser` في استعلام واحد (باستخدام union)

```php
$visited = Product::visitedByUser()->get();
$notVisited = Product::notVisitedByUser()->get();
$all = $visited->merge($notVisited);
```

---

### 10. حالات استخدام متقدمة في نظام اجتماعي

#### 10.1 اقتراح منتجات للمستخدم بناءً على زيارات أصدقائه

```php
$user = AuthHelpers::getCurrentUser();

// المستخدمون الذين يتابعهم المستخدم الحالي (افترض وجود نظام متابعة)
$followedUsers = $user->followedBy()->pluck('id');

// المنتجات التي زارها هؤلاء المستخدمون
$suggestedProducts = Product::whereHas('visits', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers);
})->whereNotIn('id', $user->visitedBy()->pluck('id')) // استبعاد ما زاره المستخدم
  ->withCountVisits('DESC', 'visits_count', true)
  ->limit(10)
  ->get();
```

#### 10.2 إشعارات الزيارات الجديدة لمنتج يتابعه المستخدم

```php
// بعد تسجيل زيارة لمنتج
$product = Product::find($productId);
$visitor = AuthHelpers::getCurrentUser();

// إرسال إشعار للمستخدمين الذين يتابعون المنتج (افترض وجود نظام متابعة)
$followers = $product->followedBy()->get();
foreach ($followers as $follower) {
    if ($follower->id != $visitor->id) {
        Notification::send($follower, new ProductVisitedNotification($product, $visitor));
    }
}
```

#### 10.3 عرض عدد الزيارات والمشاهدات في قائمة المنتجات مع التحميل البطيء

```php
$products = Product::withCountVisits('DESC', 'visits_count', true)
    ->withSumVisits('DESC', 'total_visits')
    ->withSumViews('DESC', 'total_views')
    ->paginate(20);
```

#### 10.4 تحليل سلوك المستخدمين (الأكثر تفاعلاً)

```php
// المستخدمون الذين زاروا أكثر من 10 منتجات
$activeUsers = User::has('visits', '>=', 10)->get();

// المنتجات الأكثر مشاهدة من قبل المستخدمين النشطين
$topProducts = Product::whereHas('visits', function($q) use ($activeUsers) {
    $q->whereIn('user_id', $activeUsers->pluck('id'));
})->withSumViews('DESC', 'total_views')
  ->limit(10)
  ->get();
```

---

### الخلاصة

يوفر سلوك `VisitModel` مجموعة غنية من الأدوات التي تمكنك من بناء نظام تتبع زيارات ومشاهدات متكامل بسهولة. من خلال الأمثلة المقدمة، يمكنك تطبيق هذه الميزات في مشاريعك لإنشاء تجارب مستخدم تفاعلية، وتحليل سلوك المستخدمين، وتحسين أداء التطبيقات. استخدم النطاقات المتقدمة والدوال الإحصائية لتقليل التعقيد وزيادة الإنتاجية.