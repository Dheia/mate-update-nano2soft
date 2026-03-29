# أمثلة عملية متقدمة لاستخدام سلوك `ReactionableModel`

يقدم هذا القسم مجموعة من الأمثلة المتكاملة التي توضح كيفية استخدام سلوك `ReactionableModel` في سيناريوهات حقيقية، بدءاً من العمليات الأساسية وصولاً إلى الاستعلامات المعقدة والإحصائيات المتقدمة.

---

### 1. العمليات الأساسية

#### 1.1 إضافة تفاعل بأنواع مختلفة

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Reaction;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة تفاعل من نوع 'heart'
$reaction = Reaction::add($product, $user, 'heart');

// إضافة تفاعل من نوع 'wow'
$reaction = Reaction::add($product, $user, 'wow', ['notes' => 'مذهل!']);

// إضافة تفاعل من نوع 'like'
$reaction = Reaction::add($product, $user, 'like');

// إضافة تفاعل مع ميتاداتا إضافية
$reaction = Reaction::add($product, $user, 'angry', [
    'reason' => 'جودة منخفضة',
    'priority' => 'high'
]);
```

#### 1.2 إزالة تفاعل

```php
// إزالة التفاعل (أي نوع)
Reaction::remove($product, $user);

// إزالة تفاعل من نوع 'heart' فقط
Reaction::remove($product, $user, 'heart');
```

#### 1.3 التبديل بين التفاعلات (Toggle)

```php
// إذا كان المستخدم قد تفاعل من قبل، يزيل التفاعل؛ وإلا يضيف تفاعلاً
$result = Reaction::toggle($product, $user, 'heart');

// تبديل تفاعل من نوع 'wow'
$result = Reaction::toggle($product, $user, 'wow');
```

#### 1.4 الحصول على سجل التفاعل للمستخدم الحالي

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

$reaction = $product->getReacted($user);
if ($reaction) {
    echo "نوع التفاعل: " . $reaction->value; // heart, wow, like, angry
    echo "بيانات إضافية: " . $reaction->metadata['reason'] ?? '';
}
```

---

### 2. التصفية حسب المستخدمين

#### 2.1 جلب جميع المنتجات التي تفاعل معها المستخدم الحالي

```php
// جميع التفاعلات (أي نوع)
$myReactedProducts = Product::reactedByUser()->get();

// فقط التفاعلات من نوع 'heart'
$myHearts = Product::reactedByUser(null, 'heart')->get();

// فقط التفاعلات من نوع 'wow'
$myWow = Product::reactedByUser(null, 'wow')->get();
```

#### 2.2 جلب المنتجات التي لم يتفاعل معها المستخدم الحالي (لاقتراحها)

```php
$suggestedProducts = Product::notReactedByUser()->paginate(20);
```

#### 2.3 جلب المنتجات التي تفاعل معها مستخدم معين مع تحميل تفاصيل التفاعل

```php
$user = User::find(2);
$products = Product::reactedByUser($user)
    ->with(['reactions' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

#### 2.4 استخدام الدوال القديمة `whereHasReaction` مع التحكم في السلوك عند عدم وجود مستخدم

```php
// السلوك الافتراضي (لا يعيد نتائج عند عدم وجود مستخدم)
$products = Product::whereHasReaction(null, 'heart')->get();

// تعطيل القسر (يعيد جميع المنتجات عند عدم وجود مستخدم)
$products = Product::whereHasReaction(null, 'heart', false)->get();
```

---

### 3. الترتيب والتصنيف المتقدم

#### 3.1 ترتيب المنتجات حسب عدد التفاعلات (الأكثر تفاعلاً)

```php
// باستخدام النطاق المباشر
$topProducts = Product::sortByCountReactions('DESC', 'reactions_count')->take(10)->get();

// باستخدام الاختصار topReacted
$topProducts = Product::topReacted(10)->get();

// ترتيب حسب عدد التفاعلات من نوع 'heart' فقط
$topHearts = Product::sortByCountReactions('DESC', 'hearts_count', 'heart')->get();

// ترتيب حسب عدد التفاعلات من نوع 'wow' فقط
$topWow = Product::sortByCountReactions('DESC', 'wow_count', 'wow')->get();
```

#### 3.2 إضافة عمود عدد التفاعلات إلى النتائج (بدون ترتيب)

```php
$products = Product::addCountReactions()->get();
foreach ($products as $product) {
    echo $product->name . ' - عدد التفاعلات: ' . $product->reactions_count;
}
```

#### 3.3 ترتيب حسب أحدث تفاعل (آخر مستخدم تفاعل مع المنتج)

```php
// ترتيب حسب تاريخ إنشاء آخر تفاعل (الأحدث أولاً)
$products = Product::sortByLatestReaction('DESC')->get();

// استخدام حقل updated_at بدلاً من created_at
$products = Product::sortByLatestReaction('DESC', 'latest_reaction_at', null, 'updated_at')->get();
```

#### 3.4 إضافة عمود آخر تاريخ تفاعل مع الترتيب

```php
$products = Product::withLatestReaction('DESC', 'last_reaction_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - آخر تفاعل: ' . $product->last_reaction_date;
}
```

#### 3.5 ترتيب حسب أقدم تفاعل (أول من تفاعل)

```php
$products = Product::sortByEarliestReaction('ASC')->get();
```

---

### 4. الإحصائيات والتقارير

#### 4.1 إجمالي عدد التفاعلات لمنتج معين

```php
$product = Product::find(1);

// كل التفاعلات (جميع الأنواع)
$total = $product->getTotalReactions();

// التفاعلات من نوع 'heart' فقط
$totalHearts = $product->getTotalReactions('heart');

// التفاعلات من نوع 'wow' فقط
$totalWow = $product->getTotalReactions('wow');
```

#### 4.2 توزيع التفاعلات حسب نوع المستخدم

```php
$counts = $product->getReactionsCountByType('heart');
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]

// توزيع تفاعلات 'wow'
$wowCounts = $product->getReactionsCountByType('wow');
```

#### 4.3 جلب قائمة بأسماء المستخدمين الذين تفاعلوا مع منتج معين

```php
$users = $product->getReactorsUsers('heart');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 إحصائيات التفاعلات في نطاق زمني (باستخدام النطاقات مع where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// المنتجات التي تم التفاعل معها خلال الأسبوع الماضي
$products = Product::whereHas('reactions', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. الجمع بين النطاقات لاستعلامات معقدة

#### 5.1 المنتجات الأكثر تفاعلاً والتي تفاعل معها المستخدم الحالي أيضاً

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topReacted(20)
    ->reactedByUser($user)
    ->withIsReactedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (عدد التفاعلات: ' . $product->reactions_count . ') - ';
    echo $product->is_reacted_by_user ? 'تفاعلت' : 'لم تتفاعل';
}
```

#### 5.2 المنتجات التي لها أكثر من 5 تفاعلات من نوع 'heart'، مرتبة حسب أحدث تفاعل

```php
$products = Product::hasReactions(5, 'heart')
    ->withLatestReaction('DESC', 'last_heart_date')
    ->get();
```

#### 5.3 المنتجات النشطة التي تفاعل معها المستخدم الحالي، مع عدد التفاعلات

```php
$products = Product::active()  // افتراض وجود نطاق active في الموديل
    ->reactedByUser()
    ->withCountReactions('DESC', 'reactions_count')
    ->get();
```

#### 5.4 المنتجات التي تفاعل معها المستخدم الحالي مع إضافة عمود الحالة (سيكون 1 دائماً)

```php
$products = Product::reactedByUser()
    ->withIsReactedByUser()
    ->get();
// كل المنتجات ستحوي is_reacted_by_user = 1
```

#### 5.5 المنتجات التي لها تفاعلات من نوع 'wow' فقط، واستبعاد المنتجات التي لا تفاعلات لها

```php
$products = Product::hasReactions(1, 'wow')
    ->sortByCountReactions('DESC', 'wow_count', 'wow')
    ->get();
```

---

### 6. استخدام الدوال المساعدة في القوالب (مثال Twig/Blade)

#### 6.1 عرض أزرار تفاعل متعددة

```php
@php
$product = Product::find(1);
$isHeart = $product->isReacted(null, 'heart');
$isWow = $product->isReacted(null, 'wow');
$heartCount = $product->getTotalReactions('heart');
$wowCount = $product->getTotalReactions('wow');
@endphp

<button class="reaction-btn" data-type="heart" data-id="{{ $product->id }}">
    @if($isHeart)
        ❤️ إلغاء
    @else
        🤍 قلب
    @endif
</button>
<button class="reaction-btn" data-type="wow" data-id="{{ $product->id }}">
    @if($isWow)
        😲 إلغاء
    @else
        😲 واو
    @endif
</button>
<span class="heart-count">{{ $heartCount }}</span>
<span class="wow-count">{{ $wowCount }}</span>
```

#### 6.2 عرض قائمة المتفاعلين في صفحة المنتج

```php
$reactors = $product->getReactorsUsers('heart');
?>

<div class="reactors-list">
    <h4>أبرز المعجبين (قلب)</h4>
    @foreach($reactors->take(5) as $user)
        <div class="reactor">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

---

### 7. أمثلة على واجهات API (JSON)

#### 7.1 إرجاع تفاصيل المنتج مع معلومات التفاعلات

```php
public function show($id)
{
    $product = Product::withIsReactedByUser()
        ->withCountReactions('DESC', 'reactions_count')
        ->find($id);
    
    return response()->json($product);
}
```

النتيجة:
```json
{
    "id": 1,
    "name": "هاتف ذكي",
    "reactions_count": 245,
    "is_reacted_by_user": 1
}
```

#### 7.2 قائمة المنتجات الأكثر تفاعلاً مع إضافة عمود التفاعل للمستخدم

```php
public function topReacted()
{
    $products = Product::topReacted(10)
        ->withIsReactedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API لإضافة أو إزالة تفاعل

```php
public function toggleReaction($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    $value = $request->input('value', 'heart');
    
    $result = Reaction::toggle($product, $user, $value);
    
    return response()->json([
        'success' => true,
        'is_reacted' => $product->isReacted($user, $value),
        'reactions_count' => $product->getTotalReactions($value)
    ]);
}
```

---

### 8. التوافق العكسي: استخدام النطاقات القديمة

إذا كنت تعتمد على كود قديم يستخدم النطاقات القديمة، يمكنك الاستمرار في استخدامها، لكن يُنصح بالترقية تدريجياً.

```php
// قديم
$products = Product::sortByCountReactionsOld('DESC')->get();

// جديد
$products = Product::sortByCountReactions('DESC')->get();
```

```php
// قديم
$products = Product::withSortByCountReactions('DESC', 'heart')->get();

// جديد
$products = Product::withCountReactions('DESC', 'reactions_count', 'heart')->get();
```

```php
// قديم
$products = Product::sortByCreatedAtReactions('DESC', 'heart')->get();

// جديد
$products = Product::sortByLatestReaction('DESC', 'latest_reaction_at', 'heart')->get();
```

```php
// قديم
$products = Product::whereHasReaction($user, 'heart')->get();

// جديد
$products = Product::reactedByUser($user, 'heart')->get();
```

---

### 9. نصائح وحيل متقدمة

#### 9.1 استخدام أنواع متعددة من التفاعلات

```php
// تعيين الأنواع المسموحة في الإعدادات
// config/nano/markable.php
return [
    'allowed_values' => [
        'reaction' => ['like', 'heart', 'wow', 'angry', 'sad'],
    ],
];

// الاستعلام عن منتجات تم التفاعل معها كنوع 'wow'
$wowProducts = Product::reactedByUser(null, 'wow')->get();
```

#### 9.2 الحصول على عدد التفاعلات لكل نوع

```php
$product = Product::find(1);

$reactionsByType = [
    'heart' => $product->getTotalReactions('heart'),
    'wow' => $product->getTotalReactions('wow'),
    'like' => $product->getTotalReactions('like'),
    'angry' => $product->getTotalReactions('angry'),
];
```

#### 9.3 استخدام النطاق `hasReactions` مع حد أدنى متغير من التحكم

```php
$minCount = request('min_reactions', 5);
$products = Product::hasReactions($minCount, 'heart')->get();
```

#### 9.4 تخصيص اسم العمود في النطاقات

```php
$products = Product::withCountReactions('DESC', 'my_heart_column', 'heart')->get();
echo $products->first()->my_heart_column;
```

#### 9.5 الجمع بين `reactedByUser` و `notReactedByUser` في استعلام واحد (باستخدام union)

```php
$reacted = Product::reactedByUser()->get();
$notReacted = Product::notReactedByUser()->get();
$all = $reacted->merge($notReacted);
```

---

### 10. حالات استخدام متقدمة في نظام اجتماعي

#### 10.1 اقتراح منتجات للمستخدم بناءً على تفاعلات أصدقائه

```php
$user = AuthHelpers::getCurrentUser();

// المستخدمون الذين يتابعهم المستخدم الحالي (افترض وجود نظام متابعة)
$followedUsers = $user->followedBy()->pluck('id');

// المنتجات التي تفاعل معها هؤلاء المستخدمون (نوع heart)
$suggestedProducts = Product::whereHas('reactions', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers)
      ->where('value', 'heart');
})->whereNotIn('id', $user->reactedBy()->pluck('id')) // استبعاد ما تفاعل معه المستخدم
  ->withCountReactions('DESC', 'hearts_count', 'heart')
  ->limit(10)
  ->get();
```

#### 10.2 إشعارات التفاعلات الجديدة (عند تفاعل مستخدم مع منتج)

```php
// بعد إضافة التفاعل
$reaction = Reaction::add($product, $user, 'heart');

// إرسال إشعار لمالك المنتج
if ($product->owner_id != $user->id) {
    Notification::send($product->owner, new NewReactionNotification($product, $user, 'heart'));
}
```

#### 10.3 عرض عدد التفاعلات في قائمة المنتجات مع التحميل البطيء

```php
$products = Product::withCountReactions('DESC', 'reactions_count')
    ->paginate(20);
```

#### 10.4 تحليل سلوك المستخدمين (الأكثر تفاعلاً)

```php
// المستخدمون الذين تفاعلوا مع أكثر من 10 منتجات
$activeUsers = User::has('reactions', '>=', 10)->get();

// المنتجات الأكثر تفاعلاً من قبل المستخدمين النشطين (نوع heart)
$topProducts = Product::whereHas('reactions', function($q) use ($activeUsers) {
    $q->whereIn('user_id', $activeUsers->pluck('id'))
      ->where('value', 'heart');
})->withCountReactions('DESC', 'hearts_count', 'heart')
  ->limit(10)
  ->get();
```

#### 10.5 إنشاء نظام تحليل المشاعر (تفاعلات إيجابية/سلبية)

```php
$product = Product::find(1);

$positiveReactions = ['heart', 'wow', 'like'];
$negativeReactions = ['angry', 'sad'];

$positiveCount = 0;
$negativeCount = 0;

foreach ($positiveReactions as $type) {
    $positiveCount += $product->getTotalReactions($type);
}
foreach ($negativeReactions as $type) {
    $negativeCount += $product->getTotalReactions($type);
}

$total = $positiveCount + $negativeCount;
$positiveRatio = $total > 0 ? round(($positiveCount / $total) * 100, 1) : 0;

echo "نسبة التفاعلات الإيجابية: {$positiveRatio}%";
```

---

### الخلاصة

يوفر سلوك `ReactionableModel` مجموعة غنية من الأدوات التي تمكنك من بناء نظام تفاعلات متكامل بسهولة. من خلال الأمثلة المقدمة، يمكنك تطبيق هذه الميزات في مشاريعك لإنشاء تجارب مستخدم تفاعلية، وتحليل مشاعر المستخدمين، وتحسين أداء التطبيقات. استخدم النطاقات المتقدمة والدوال الإحصائية لتقليل التعقيد وزيادة الإنتاجية.