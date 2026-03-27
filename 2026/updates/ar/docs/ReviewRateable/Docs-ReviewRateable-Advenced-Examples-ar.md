# أمثلة عملية متقدمة لاستخدام سلوك `ReviewRateable`

يقدم هذا القسم مجموعة من الأمثلة المتكاملة التي توضح كيفية استخدام سلوك `ReviewRateable` في سيناريوهات حقيقية، بدءاً من العمليات الأساسية وصولاً إلى الاستعلامات المعقدة والإحصائيات المتقدمة.

---

### 1. العمليات الأساسية

#### 1.1 إضافة تقييم جديد مع بيانات مخصصة

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Reviews\Models\Review;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة تقييم عادي (5 نجوم، مراجعة نصية)
$review = $product->addRating([
    'rating' => 5,
    'title' => 'منتج رائع',
    'content' => 'جودة عالية وسعر مناسب',
    'approved' => true,
    'is_positive' => true,
], $user);

// إضافة تقييم غير معتمد (يحتاج موافقة المشرف)
$review = $product->addRating([
    'rating' => 2,
    'title' => 'غير راضٍ',
    'content' => 'المنتج لا يطابق الوصف',
    'approved' => false,
    'is_positive' => false,
], $user);
```

#### 1.2 تحديث تقييم موجود

```php
$user = AuthHelpers::getCurrentUser();
$review = $product->getRating($user);

if ($review) {
    $product->updateRating($review->id, [
        'rating' => 4,
        'content' => 'بعد فترة من الاستخدام، أصبح أفضل',
    ]);
}
```

#### 1.3 التبديل بين التقييم والإلغاء (Toggle)

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إذا كان المستخدم قد قيم، يقوم بتحديث التقييم (يمكن تمرير بيانات جديدة)
// إذا لم يقيم، يقوم بإضافة تقييم جديد
$product->toggleRating(['rating' => 5, 'content' => 'تجربة ممتازة'], $user);
```

#### 1.4 حذف تقييم (Soft Delete)

```php
$user = AuthHelpers::getCurrentUser();
$review = $product->getRating($user);

if ($review) {
    $product->deleteRating($review->id);
}
```

---

### 2. التصفية حسب المستخدمين

#### 2.1 جلب جميع المنتجات التي قيمها المستخدم الحالي

```php
$myRatedProducts = Product::ratedByUser()->get();

// مع تضمين التقييمات غير النشطة فقط
$myRatedProducts = Product::ratedByUser(null, false)->get();

// مع تضمين التقييمات المعتمدة فقط
$myRatedProducts = Product::ratedByUser(null, null, false, true)->get();
```

#### 2.2 جلب المنتجات التي لم يقيمها المستخدم الحالي (لاقتراحها)

```php
$unratedProducts = Product::notRatedByUser()->paginate(20);
```

#### 2.3 جلب المنتجات التي قيمها مستخدم معين بتقييم إيجابي

```php
$user = User::find(2);
$products = Product::ratedByUser($user, null, false, null, true)->get();
```

---

### 3. الترتيب والتصنيف المتقدم

#### 3.1 ترتيب المنتجات حسب عدد التقييمات (الأكثر تقييماً)

```php
// باستخدام النطاق المباشر
$topProducts = Product::sortByCountReviews('DESC', 'reviews_count', true)->take(10)->get();

// باستخدام الاختصار topRatedByCount
$topProducts = Product::topRatedByCount(10)->get();

// ترتيب حسب عدد التقييمات المعتمدة فقط
$topProducts = Product::sortByCountReviews('DESC', 'reviews_count', true, false, true)->get();
```

#### 3.2 ترتيب المنتجات حسب متوسط التقييم (الأعلى تقييماً)

```php
// باستخدام النطاق المباشر
$topProducts = Product::sortByAvgRating('DESC', 'avg_rating', true, false, true)->get();

// باستخدام الاختصار topRated
$topProducts = Product::topRated(10)->get();

// ترتيب حسب متوسط التقييمات الإيجابية فقط
$topProducts = Product::sortByAvgRating('DESC', 'avg_rating', null, false, null, true)->get();
```

#### 3.3 إضافة عمود عدد التقييمات إلى النتائج (بدون ترتيب)

```php
$products = Product::addCountReviews()->get();
foreach ($products as $product) {
    echo $product->name . ' - عدد التقييمات: ' . $product->reviews_count;
}
```

#### 3.4 إضافة عمود متوسط التقييم مع الترتيب

```php
$products = Product::withAvgRating('DESC', 'avg_rating', true, false, true)->get();
foreach ($products as $product) {
    echo $product->name . ' - متوسط التقييم: ' . $product->avg_rating;
}
```

#### 3.5 ترتيب حسب أحدث تقييم (آخر مستخدم قام بالتقييم)

```php
$products = Product::sortByLatestReview('DESC')->get();
```

#### 3.6 ترتيب حسب أقدم تقييم (أول من قيم)

```php
$products = Product::sortByEarliestReview('ASC')->get();
```

---

### 4. الإحصائيات والتقارير

#### 4.1 إجمالي التقييمات لمنتج معين (مجموع قيم التقييم)

```php
$product = Product::find(1);

// كل التقييمات
$total = $product->getTotalRatings();

// التقييمات المعتمدة فقط
$totalApproved = $product->getTotalRatings(null, false, true);

// التقييمات الإيجابية المعتمدة فقط
$totalPositive = $product->getTotalRatings(null, false, true, true);
```

#### 4.2 متوسط التقييم لمنتج معين

```php
$product = Product::find(1);

// متوسط كل التقييمات
$avg = $product->getAverageRating();

// متوسط التقييمات المعتمدة فقط (مقرب لمنزلتين عشريتين)
$avgApproved = $product->getAverageRating(null, false, true, null, null, 2);
```

#### 4.3 توزيع التقييمات حسب نوع المستخدم

```php
$counts = $product->getRatingsCountByType(true, false, true);
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]
```

#### 4.4 جلب قائمة بأسماء المستخدمين الذين قيموا منتجاً معيناً

```php
$users = $product->getRatersUsers(true, false, true);
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.5 إحصائيات التقييمات في نطاق زمني (باستخدام النطاقات مع where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// المنتجات التي قيمت خلال الأسبوع الماضي
$products = Product::whereHas('ratings', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. الجمع بين النطاقات لاستعلامات معقدة

#### 5.1 المنتجات الأعلى تقييماً (متوسط التقييم) والتي قيمها المستخدم الحالي أيضاً

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topRated(20)
    ->ratedByUser($user)
    ->withIsRatedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (متوسط التقييم: ' . $product->avg_rating . ') - ';
    echo $product->is_rated_by_user ? 'قيمته' : 'لم تقيمه';
}
```

#### 5.2 المنتجات التي لها أكثر من 5 تقييمات معتمدة، مرتبة حسب متوسط التقييم (مع إضافة العمود)

```php
$products = Product::hasRatings(5, true, false, true)
    ->withAvgRating('DESC', 'avg_rating')
    ->get();
```

#### 5.3 المنتجات النشطة التي قيمها المستخدم الحالي، مع إضافة عمود متوسط التقييم وعدد التقييمات

```php
$products = Product::active()  // افتراض وجود نطاق active في الموديل
    ->ratedByUser()
    ->withCountReviews('DESC', 'reviews_count')
    ->withAvgRating('DESC', 'avg_rating')
    ->get();
```

#### 5.4 المنتجات التي حصلت على تقييم 5 نجوم من مستخدمين معينين

```php
$users = [1, 2, 3]; // معرفات المستخدمين
$products = Product::whereHas('ratings', function($q) use ($users) {
    $q->whereIn('user_id', $users)
      ->where('rating', 5);
})->get();
```

#### 5.5 المنتجات التي ليس لها تقييمات معتمدة

```php
$products = Product::hasNoRatings(true, false, true)->get();
```

---

### 6. استخدام الدوال المساعدة في القوالب (مثال Twig/Blade)

#### 6.1 عرض متوسط التقييم وعدد التقييمات لمنتج

```php
@php
$product = Product::find(1);
$avgRating = $product->getAverageRating(null, false, true);
$count = $product->getTotalRatings(null, false, true);
$isRated = $product->isRating();
@endphp

<div class="product-rating">
    <span class="stars">@for($i=1;$i<=5;$i++)★@endfor</span>
    <span class="avg">{{ number_format($avgRating, 1) }}</span>
    <span class="count">({{ $count }} تقييم)</span>
    @if($isRated)
        <span class="rated">لقد قيمت هذا المنتج</span>
    @else
        <button class="rate-btn">قيم المنتج</button>
    @endif
</div>
```

#### 6.2 عرض قائمة التقييمات في صفحة المنتج

```php
$ratings = $product->ratings()
    ->where('approved', true)
    ->with('user')
    ->latest()
    ->take(5)
    ->get();
?>

<div class="reviews-list">
    <h4>أحدث التقييمات</h4>
    @foreach($ratings as $review)
        <div class="review-item">
            <div class="user-info">
                <img src="{{ $review->user->avatar }}" width="40">
                <strong>{{ $review->user->name }}</strong>
                <span>{{ $review->created_at->diffForHumans() }}</span>
            </div>
            <div class="rating">التقييم: {{ $review->rating }}/5</div>
            <div class="title">{{ $review->title }}</div>
            <div class="content">{{ $review->content }}</div>
        </div>
    @endforeach
</div>
```

---

### 7. أمثلة على واجهات API (JSON)

#### 7.1 إرجاع تفاصيل المنتج مع إحصائيات التقييمات

```php
public function show($id)
{
    $product = Product::withIsRatedByUser()
        ->withCountReviews('DESC', 'reviews_count')
        ->withAvgRating('DESC', 'avg_rating')
        ->find($id);
    
    return response()->json($product);
}
```

النتيجة:
```json
{
    "id": 1,
    "name": "هاتف ذكي",
    "reviews_count": 45,
    "avg_rating": 4.7,
    "is_rated_by_user": 1
}
```

#### 7.2 قائمة المنتجات الأعلى تقييماً مع إضافة عمود التقييم للمستخدم

```php
public function topRated()
{
    $products = Product::topRated(10)
        ->withIsRatedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API لإضافة أو تحديث تقييم

```php
public function rate($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    
    $review = $product->toggleRating([
        'rating' => $request->rating,
        'title' => $request->title,
        'content' => $request->content,
        'approved' => false, // تحتاج موافقة المشرف
        'is_positive' => $request->rating >= 4,
    ], $user);
    
    return response()->json([
        'success' => true,
        'review' => $review,
        'avg_rating' => $product->getAverageRating(null, false, true),
        'reviews_count' => $product->getTotalRatings(null, false, true)
    ]);
}
```

---

### 8. التوافق العكسي: استخدام النطاقات القديمة

إذا كنت تعتمد على كود قديم يستخدم النطاقات القديمة، يمكنك الاستمرار في استخدامها، لكن يُنصح بالترقية تدريجياً.

```php
// قديم
$products = Product::sortByRating('DESC')->get();

// جديد
$products = Product::sortByAvgRating('DESC')->get();
```

```php
// قديم
$products = Product::withRating('DESC')->get();

// جديد
$products = Product::withAvgRating('DESC')->get();
```

```php
// قديم
$products = Product::sortByCreatedAtReviews('DESC')->get();

// جديد
$products = Product::sortByLatestReview('DESC')->get();
```

```php
// قديم
$products = Product::withSortByCountReviews('DESC')->get();

// جديد
$products = Product::withCountReviews('DESC')->get();
```

---

### 9. نصائح وحيل متقدمة

#### 9.1 استخدام `withTrashed` في الإحصائيات لتضمين التقييمات المحذوفة

```php
$totalWithDeleted = $product->getTotalRatings(null, true);
```

#### 9.2 الحصول على عدد التقييمات الإيجابية فقط

```php
$positiveCount = $product->getRatingsCountByType(null, false, null, true)->sum();
```

#### 9.3 استخدام النطاق `hasRatings` مع حد أدنى متغير من التحكم

```php
$minCount = request('min_ratings', 5);
$products = Product::hasRatings($minCount, true, false, true)->get();
```

#### 9.4 تخصيص اسم العمود في النطاقات

```php
$products = Product::withCountReviews('DESC', 'my_ratings_column', true)->get();
echo $products->first()->my_ratings_column;
```

#### 9.5 الحصول على متوسط التقييم حسب كل مستخدم (تحليل تفصيلي)

```php
$userRatings = $product->ratings()
    ->select('user_id', 'user_type', DB::raw('AVG(rating) as avg'))
    ->groupBy('user_id', 'user_type')
    ->get();
```

---

### 10. حالات استخدام متقدمة في نظام اجتماعي

#### 10.1 اقتراح منتجات للمستخدم بناءً على تقييمات أصدقائه

```php
$user = AuthHelpers::getCurrentUser();

// المستخدمون الذين يتابعهم المستخدم الحالي (افترض وجود نظام متابعة)
$followedUsers = $user->followedBy()->pluck('id');

// المنتجات التي قيمها هؤلاء المستخدمون بتقييم مرتفع (أكثر من 4)
$suggestedProducts = Product::whereHas('ratings', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers)
      ->where('rating', '>=', 4);
})->whereNotIn('id', $user->ratedBy()->pluck('id')) // استبعاد ما قيمه المستخدم
  ->withAvgRating('DESC', 'avg_rating')
  ->limit(10)
  ->get();
```

#### 10.2 إشعارات التقييمات الجديدة (عند الموافقة على تقييم)

```php
// بعد الموافقة على تقييم من قبل المشرف
$review->approved = true;
$review->save();

if ($review->wasChanged('approved')) {
    // إرسال إشعار للمستخدم الذي كتب التقييم
    Notification::send($review->user, new ReviewApprovedNotification($product, $review));
    
    // إشعار لمالك المنتج
    Notification::send($product->owner, new NewReviewNotification($product, $review));
}
```

#### 10.3 عرض متوسط التقييم في قائمة المنتجات مع التحميل البطيء

```php
$products = Product::withAvgRating('DESC', 'avg_rating')
    ->withCountReviews('DESC', 'reviews_count')
    ->paginate(20);
```

#### 10.4 تحليل اتجاهات التقييمات (تغير متوسط التقييم مع الوقت)

```php
$product = Product::find(1);

$ratingsByMonth = $product->ratings()
    ->select(DB::raw('DATE_FORMAT(created_at, "%Y-%m") as month'), DB::raw('AVG(rating) as avg_rating'))
    ->where('approved', true)
    ->groupBy('month')
    ->orderBy('month')
    ->get();
```

---

### الخلاصة

يوفر سلوك `ReviewRateable` مجموعة غنية من الأدوات التي تمكنك من بناء نظام تقييم متكامل بسهولة. من خلال الأمثلة المقدمة، يمكنك تطبيق هذه الميزات في مشاريعك لإنشاء تجارب مستخدم تفاعلية، وتحليل آراء العملاء، وتحسين المنتجات بناءً على التقييمات. استخدم النطاقات المتقدمة والدوال الإحصائية لتقليل التعقيد وزيادة الإنتاجية.