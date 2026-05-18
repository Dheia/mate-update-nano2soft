# أمثلة عملية متقدمة لاستخدام سلوك `ProposalModel`

يقدم هذا القسم مجموعة من الأمثلة المتكاملة التي توضح كيفية استخدام سلوك `ProposalModel` في سيناريوهات حقيقية، بدءاً من العمليات الأساسية وصولاً إلى الاستعلامات المعقدة والإحصائيات المتقدمة.

---

### 1. العمليات الأساسية

#### 1.1 إضافة بلاغ بأنواع مختلفة

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano2\Proposals\Models\Proposal;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة بلاغ (report) عن منتج
$report = $product->addProposal([
    'type' => 'reports',
    'content' => 'المنتج لا يطابق الوصف',
    'is_active' => true,
], $user);

// إضافة مقترح (proposal)
$proposal = $product->addProposal([
    'type' => 'proposals',
    'content' => 'أقترح إضافة خاصية الدفع بالتقسيط',
], $user);

// إضافة شكوى (complaint)
$complaint = $product->addProposal([
    'type' => 'complaints',
    'content' => 'تأخر الشحن عن الموعد المحدد',
    'is_active' => true,
], $user);

// إضافة بلاغ مع ميتاداتا إضافية (عبر `other_data` أو `config_data`)
$report = $product->addProposal([
    'type' => 'reports',
    'content' => 'سعر غير منطقي',
    'other_data' => ['priority' => 'high', 'order_id' => 123],
], $user);
```

#### 1.2 تحديث بلاغ موجود

```php
$user = AuthHelpers::getCurrentUser();
$report = $product->getUserProposal($user);

if ($report) {
    $product->updateProposal($report->id, [
        'content' => 'تحديث المحتوى بعد التحقق',
        'is_active' => true,
    ]);
}
```

#### 1.3 التبديل بين الإضافة والإزالة (Toggle)

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إذا كان المستخدم قد أرسل بلاغاً، يقوم بحذفه؛ وإلا يضيف بلاغاً جديداً
$product->toggleProposal([
    'type' => 'reports',
    'content' => 'بلاغ جديد',
], $user);
```

#### 1.4 حذف بلاغ (Soft Delete)

```php
$user = AuthHelpers::getCurrentUser();
$report = $product->getUserProposal($user);

if ($report) {
    $product->deleteProposal($report->id);
}
```

---

### 2. التصفية حسب المستخدمين

#### 2.1 جلب المنتجات التي أرسل لها المستخدم الحالي بلاغاً (جانب target)

```php
// جميع المنتجات التي أرسل لها المستخدم الحالي بلاغاً (أي نوع)
$myReportedProducts = Product::proposedByUser()->get();

// فقط المنتجات التي أرسل لها المستخدم الحالي بلاغاً من نوع 'complaints'
$myComplaints = Product::proposedByUser(null, 'complaints')->get();
```

#### 2.2 جلب المنتجات التي لم يرسل لها المستخدم الحالي بلاغاً

```php
$unreportedProducts = Product::notProposedByUser()->paginate(20);
```

#### 2.3 جلب المستخدمين الذين أرسلوا بلاغات (جانب user)

```php
// المستخدمون الذين أرسلوا بلاغات (أي نوع)
$usersWhoReported = User::proposedToUser()->get();

// المستخدمون الذين أرسلوا بلاغات من نوع 'reports'
$usersWithReports = User::proposedToUser(null, 'reports')->get();
```

#### 2.4 جلب المنتجات التي أرسل لها مستخدم معين بلاغاً مع تحميل تفاصيل البلاغ

```php
$user = User::find(2);
$products = Product::proposedByUser($user)
    ->with(['all_proposals' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

---

### 3. الترتيب والتصنيف المتقدم

#### 3.1 ترتيب المنتجات حسب عدد البلاغات (الأكثر بلاغاً)

```php
// باستخدام النطاق المباشر
$topProducts = Product::sortByCountProposals('DESC')->take(10)->get();

// باستخدام الاختصار topProposed
$topProducts = Product::topProposed(10)->get();

// ترتيب حسب عدد البلاغات من نوع 'complaints' فقط
$topComplaints = Product::sortByCountProposals('DESC', 'complaints')->get();
```

#### 3.2 إضافة عمود عدد البلاغات إلى النتائج (بدون ترتيب)

```php
$products = Product::addCountProposals()->get();
foreach ($products as $product) {
    echo $product->name . ' - عدد البلاغات: ' . $product->proposals_count;
}
```

#### 3.3 ترتيب حسب أحدث بلاغ (آخر مستخدم أرسل بلاغاً)

```php
// ترتيب حسب تاريخ إنشاء آخر بلاغ (الأحدث أولاً)
$products = Product::sortByLatestProposal('DESC')->get();

// استخدام حقل updated_at بدلاً من created_at
$products = Product::sortByLatestProposal('DESC', 'latest_proposal_at', null, null, null, 'updated_at')->get();
```

#### 3.4 إضافة عمود آخر تاريخ بلاغ مع الترتيب

```php
$products = Product::withLatestProposal('DESC', 'last_proposal_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - آخر بلاغ: ' . $product->last_proposal_date;
}
```

#### 3.5 ترتيب حسب أقدم بلاغ (أول من أرسل بلاغاً)

```php
$products = Product::sortByEarliestProposal('ASC')->get();
```

---

### 4. الإحصائيات والتقارير

#### 4.1 إجمالي عدد البلاغات لمنتج معين

```php
$product = Product::find(1);

// كل البلاغات (جميع الأنواع)
$total = $product->getTotalProposals();

// البلاغات من نوع 'complaints' فقط
$totalComplaints = $product->getTotalProposals('complaints');

// البلاغات النشطة من نوع 'reports'
$totalActiveReports = $product->getTotalProposals('reports', true);
```

#### 4.2 توزيع البلاغات حسب نوع المستخدم

```php
$counts = $product->getProposalsCountByType('reports');
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]

// توزيع البلاغات من نوع 'complaints'
$complaintsByType = $product->getProposalsCountByType('complaints');
```

#### 4.3 جلب قائمة بأسماء المستخدمين الذين أرسلوا بلاغات لمنتج معين

```php
$users = $product->getProposersUsers('reports');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 إحصائيات البلاغات في نطاق زمني (باستخدام النطاقات مع where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// المنتجات التي أرسل لها بلاغات خلال الأسبوع الماضي
$products = Product::whereHas('all_proposals', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. الجمع بين النطاقات لاستعلامات معقدة

#### 5.1 المنتجات الأكثر بلاغاً والتي أرسل لها المستخدم الحالي بلاغاً أيضاً

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topProposed(20)
    ->proposedByUser($user)
    ->withIsProposedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (عدد البلاغات: ' . $product->proposals_count . ') - ';
    echo $product->is_proposed_by_user ? 'أرسلت بلاغاً' : 'لم ترسل بلاغاً';
}
```

#### 5.2 المنتجات التي لها أكثر من 5 بلاغات من نوع 'complaints'، مرتبة حسب أحدث بلاغ

```php
$products = Product::hasProposals(5, 'complaints')
    ->withLatestProposal('DESC', 'last_complaint_date')
    ->get();
```

#### 5.3 المنتجات النشطة التي أرسل لها المستخدم الحالي بلاغاً، مع عدد البلاغات

```php
$products = Product::active()  // افتراض وجود نطاق active في الموديل
    ->proposedByUser()
    ->withCountProposals('DESC', 'proposals_count')
    ->get();
```

#### 5.4 المنتجات التي أرسل لها المستخدم الحالي بلاغاً مع إضافة عمود الحالة (سيكون 1 دائماً)

```php
$products = Product::proposedByUser()
    ->withIsProposedByUser()
    ->get();
// كل المنتجات ستحوي is_proposed_by_user = 1
```

#### 5.5 المنتجات التي لها بلاغات من نوع 'reports' فقط، واستبعاد المنتجات التي لا بلاغات لها

```php
$products = Product::hasProposals(1, 'reports')
    ->sortByCountProposals('DESC', 'reports_count', 'reports')
    ->get();
```

---

### 6. استخدام الدوال المساعدة في القوالب (مثال Twig/Blade)

#### 6.1 عرض عدد البلاغات للمنتج وإمكانية إضافة بلاغ

```php
@php
$product = Product::find(1);
$hasReported = $product->isUserProposal(null, 'reports');
$reportCount = $product->getTotalProposals('reports');
@endphp

<div class="proposal-buttons">
    <button class="report-btn" data-type="reports" data-id="{{ $product->id }}">
        @if($hasReported)
            إلغاء البلاغ
        @else
            إبلاغ عن مشكلة
        @endif
    </button>
    <span class="report-count">{{ $reportCount }} بلاغ</span>
</div>
```

#### 6.2 عرض قائمة المبلّغين في صفحة المنتج

```php
$reporters = $product->getProposersUsers('reports');
?>

<div class="reporters-list">
    <h4>أبرز المبلّغين</h4>
    @foreach($reporters->take(5) as $user)
        <div class="reporter">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

#### 6.3 عرض أزرار لأنواع البلاغات المختلفة

```php
@php
$product = Product::find(1);
$isReported = $product->isUserProposal(null, 'reports');
$isComplaint = $product->isUserProposal(null, 'complaints');
@endphp

<button class="report-btn" data-type="reports">
    {{ $isReported ? 'إلغاء البلاغ' : 'بلاغ' }}
</button>
<button class="complaint-btn" data-type="complaints">
    {{ $isComplaint ? 'إلغاء الشكوى' : 'شكوى' }}
</button>
```

---

### 7. أمثلة على واجهات API (JSON)

#### 7.1 إرجاع تفاصيل المنتج مع معلومات البلاغات

```php
public function show($id)
{
    $product = Product::withIsProposedByUser()
        ->withCountProposals('DESC', 'proposals_count')
        ->find($id);
    
    return response()->json($product);
}
```

النتيجة:
```json
{
    "id": 1,
    "name": "هاتف ذكي",
    "proposals_count": 245,
    "is_proposed_by_user": 1
}
```

#### 7.2 قائمة المنتجات الأكثر بلاغاً مع إضافة عمود البلاغ للمستخدم

```php
public function topReported()
{
    $products = Product::topProposed(10, 'DESC', 'reports')
        ->withIsProposedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API لإضافة أو إزالة بلاغ

```php
public function toggleReport($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    $type = $request->input('type', 'reports');
    
    $result = $product->toggleProposal([
        'type' => $type,
        'content' => $request->input('content'),
    ], $user);
    
    return response()->json([
        'success' => true,
        'is_proposed' => $product->isUserProposal($user, $type),
        'proposals_count' => $product->getTotalProposals($type)
    ]);
}
```

---

### 8. التوافق العكسي: استخدام النطاقات القديمة

إذا كنت تعتمد على كود قديم يستخدم النطاقات القديمة، يمكنك الاستمرار في استخدامها، لكن يُنصح بالترقية تدريجياً.

```php
// قديم
$products = Product::sortByCountProposalsOld('DESC')->get();

// جديد
$products = Product::sortByCountProposals('DESC')->get();
```

```php
// قديم
$products = Product::withSortByCountProposals('DESC', 'reports')->get();

// جديد
$products = Product::withCountProposals('DESC', 'proposals_count', 'reports')->get();
```

```php
// قديم
$products = Product::sortByCreatedAtProposals('DESC', 'complaints')->get();

// جديد
$products = Product::sortByLatestProposal('DESC', 'latest_proposal_at', 'complaints')->get();
```

```php
// قديم
$products = Product::whereHasProposal($user, 'reports')->get();

// جديد
$products = Product::proposedByUser($user, 'reports')->get();
```

---

### 9. نصائح وحيل متقدمة

#### 9.1 استخدام النطاق المتقدم `scopeWhereHasProposalAdvanced` لشروط معقدة

```php
// المنتجات التي لديها بلاغات من نوع 'reports' أو 'complaints' (OR)
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'reports']
])->orWhereHasProposalAdvanced([
    'conditions' => ['type' => 'complaints']
])->get();

// المنتجات التي لديها بلاغات من نوع 'reports' تم إنشاؤها خلال الأسبوع الماضي
$products = Product::whereHasProposalAdvanced([
    'conditions' => function($q) {
        $q->where('type', 'reports')
          ->where('created_at', '>=', Carbon::now()->subWeek());
    }
])->get();

// المنتجات التي لديها بلاغات من نوع 'reports' مع تصنيف معين (categories_id)
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => 'reports',
        'categories_id' => 5,
    ]
])->get();

// استخدام whereIn
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => ['whereIn', ['reports', 'complaints']],
        'is_active' => true,
    ]
])->get();

// تضمين البلاغات المحذوفة (soft deleted)
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => 'reports',
        'withTrashed' => true,
    ]
])->get();
```

#### 9.2 البحث عن البلاغات التي تم الرد عليها (حقل reply_at)

```php
$products = Product::whereHasProposalAdvanced([
    'conditions' => function($q) {
        $q->whereNotNull('reply_at');
    }
])->get();
```

#### 9.3 التحكم في مصدر المستخدم والهدف عبر الإعدادات

```php
// افتراضيًا، `userSource` للإعدادات هو 'current_user' و `targetSource` هو 'current_model'
// يمكن تغيير ذلك في الاستدعاء:
$users = User::whereHasProposalAdvanced([
    'side' => 'user',
    'userSource' => 'current_model', // استخدام الموديل الحالي كمستخدم
])->get();
```

#### 9.4 تخصيص اسم العمود في النطاقات

```php
$products = Product::withCountProposals('DESC', 'my_reports_column', 'reports')->get();
echo $products->first()->my_reports_column;
```

---

### 10. حالات استخدام متقدمة في نظام اجتماعي

#### 10.1 اقتراح منتجات للمستخدم بناءً على بلاغات أصدقائه

```php
$user = AuthHelpers::getCurrentUser();

// المستخدمون الذين يتابعهم المستخدم الحالي (افترض وجود نظام متابعة)
$followedUsers = $user->followedBy()->pluck('id');

// المنتجات التي أرسل لها هؤلاء المستخدمون بلاغات من نوع 'complaints'
$suggestedProducts = Product::whereHas('all_proposals', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers)
      ->where('type', 'complaints');
})->whereNotIn('id', $user->proposedBy()->pluck('id')) // استبعاد ما أرسل له المستخدم بلاغاً
  ->withCountProposals('DESC', 'complaints_count', 'complaints')
  ->limit(10)
  ->get();
```

#### 10.2 إشعارات البلاغات الجديدة (عند إضافة بلاغ لمنتج يتابعه المستخدم)

```php
// بعد إضافة البلاغ
$report = $product->addProposal(['type' => 'reports', 'content' => '...'], $user);

// إرسال إشعار لمالك المنتج
if ($product->owner_id != $user->id) {
    Notification::send($product->owner, new NewReportNotification($product, $user));
}
```

#### 10.3 عرض عدد البلاغات في قائمة المنتجات مع التحميل البطيء

```php
$products = Product::withCountProposals('DESC', 'proposals_count')
    ->paginate(20);
```

#### 10.4 تحليل سلوك المستخدمين (الأكثر بلاغاً)

```php
// المستخدمون الذين أرسلوا أكثر من 10 بلاغات
$activeReporters = User::has('all_user_proposals', '>=', 10)->get();

// المنتجات الأكثر بلاغاً من قبل المستخدمين النشطين (نوع reports)
$topProducts = Product::whereHas('all_proposals', function($q) use ($activeReporters) {
    $q->whereIn('user_id', $activeReporters->pluck('id'))
      ->where('type', 'reports');
})->withCountProposals('DESC', 'reports_count', 'reports')
  ->limit(10)
  ->get();
```

#### 10.5 إنشاء نظام تصنيف للبلاغات حسب الأولوية

```php
// إضافة بلاغ بأولوية عالية عبر `other_data`
$product->addProposal([
    'type' => 'reports',
    'content' => 'خطير',
    'other_data' => ['priority' => 'high'],
], $user);

// جلب البلاغات ذات الأولوية العالية
$highPriorityReports = Proposal::where('type', 'reports')
    ->whereJsonContains('other_data', ['priority' => 'high'])
    ->get();
```

---

### الخلاصة

يوفر سلوك `ProposalModel` مجموعة غنية من الأدوات التي تمكنك من بناء نظام بلاغات ومقترحات وشكاوى متكامل بسهولة. من خلال الأمثلة المقدمة، يمكنك تطبيق هذه الميزات في مشاريعك لإنشاء تجارب مستخدم تفاعلية، وتحليل سلوك المستخدمين، وتحسين أداء التطبيقات. استخدم النطاقات المتقدمة والدوال الإحصائية لتقليل التعقيد وزيادة الإنتاجية.