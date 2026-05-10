# توثيق دالة `updateOrderStatusAdvanced`

**الكلاس:** `Nano\Orders\Traits\Steps\StepStatus`  
**الإضافة:** `Nano.Orders`  
**الإصدار:** 2.2.10  

---

## 📋 مقدمة

تُعدّ دالة `updateOrderStatusAdvanced` حجر الزاوية في إدارة حالات الطلب ضمن منصة Nano.soft. صُممت لتوفير آلية موحّدة وآمنة تسمح للأطراف الثلاثة (مدير النظام، مالك الطلب، وموصل الطلب) بتحديث حالة الطلب وفق صلاحيات متدرجة وقيود منطقية صارمة. تعتمد الدالة على نظام إعدادات هرمي (افتراضي ← إعدادات عامة ← إعدادات خاصة بالدور ← خيارات الاستدعاء المباشر) مما يمنح المطورين مرونة قصوى في تخصيص سلوكها دون المساس بالأمان.

---

## 📌 توقيع الدالة

```php
public function updateOrderStatusAdvanced(array $options = []): array
```

### المعاملات

| # | المعامل | النوع | إلزامي | الوصف |
|---|--------|-------|--------|-------|
| 1 | `$options` | `array` | لا | مصفوفة الخيارات التي تتحكم في سلوك الدالة (انظر الجدول التالي). |

---

## ⚙️ الخيارات الكاملة (Options)

| المفتاح | النوع | الافتراضي | الوصف |
|--------|--------|-----------|--------|
| `actor` | `User\|BackendUser\|null` | `null` | كائن المستخدم القائم بالتعديل. إذا لم يُمرّر، تُحاول الدالة جلبه من الجلسة الحالية (`AuthHelpers` ← `BackendAuth` ← `Auth`). |
| `order` | `Order\|null` | `null` | نموذج الطلب المراد تعديله. إذا لم يُمرّر، يُستخدم نموذج الطلب الحالي من `OrderManager`. |
| `order_states_ref_type` | `string\|null` | `null` | الحالة الرئيسية الجديدة (مثلاً `COMPLETE`، `CANCELLED`). **للإدارة فقط**. |
| `user_status` | `string\|null` | `null` | حالة الطلب من جهة العميل/المالك. |
| `delivery_status` | `string\|null` | `null` | حالة الطلب من جهة الموصل. |
| `because_cancel` | `string\|null` | `null` | سبب الإلغاء المُقدّم من المالك. |
| `delivery_because_cancel` | `string\|null` | `null` | سبب الإلغاء المُقدّم من الموصل. |
| `is_save` | `bool` | `true` | هل يتم حفظ التغييرات في قاعدة البيانات؟ (`false` مفيد للاختبار). |
| `is_event` | `bool` | `true` | هل يتم إطلاق أحداث النظام (`Event::fire`) مثل `nano.orders.newOrderStatus`؟ |
| `is_logs` | `bool` | `true` | هل يتم تسجيل تاريخ الحالة عبر `Nano2\StatusHistory` (إن وجدت)؟ |
| `skip_permission` | `bool` | `false` | تجاوز جميع فحوصات الصلاحيات والأدوار (يُعطي صلاحيات مدير كاملة). |
| `admin_override` | `bool` | `false` | يسمح للإدارة بتجاوز قيود انتقال الحالات (مثلاً إرجاع حالة `DELIVERY` إلى `PROCESSING`). |
| `custom_message` | `string\|null` | `null` | رسالة نجاح مخصصة تُستخدم بدلاً من رسالة الترجمة الافتراضية. |
| `custom_error` | `string\|null` | `null` | رسالة خطأ افتراضية مخصصة تُستخدم في حالة الفشل. |
| `allowed_transitions` | `array\|null` | `null` | قواعد انتقال مخصصة لهذا الاستدعاء فقط. تُدمج فوق كل الإعدادات الأخرى. |
| `role_allowed_transitions` | `callable\|null` | `null` | دالة لتوليد قواعد انتقال مخصصة حسب الدور. تستقبل `($role, $currentTransitions)` وتُعيد مصفوفة انتقالات. |

---

## 🧰 آلية عمل الدالة (خطوة بخطوة)

### 1. دمج الإعدادات الافتراضية
تدمج الدالة القيم الافتراضية (الموضحة أعلاه) مع أي قيم قادمة من ملفات `config` تحت المسار `nano.orders::manager.edit_status`، ثم أخيراً الخيارات المباشرة المُمرّرة في `$options` (أولوية الخيارات المباشرة هي الأعلى).

### 2. تحديد الفاعل
إذا لم يُمرّر `actor` صراحةً، تبحث الدالة عن مستخدم مسجّل الدخول:
1. `AuthHelpers::getCurrentUser()`
2. `BackendAuth::getUser()`
3. `Auth::getUser()`

### 3. تصنيف الأدوار
- **مدير** (`admin`): كائن من `Backend\Models\User`.
- **مالك** (`user`): مستخدم أمامي (`RainLab\User\Models\User`) ومعرّفه يطابق `order->user_id`.
- **موصل** (`delivery`): مستخدم أمامي ومعرّفه يطابق `order->delivery_user_id`.
- إذا كان `skip_permission = true`، يُعامل الجميع كمدير.

### 4. منع تعديل الطلبات المكتملة
إذا كانت `order_states_ref_type = COMPLETE`، تُرفض أي تغييرات ما لم يكن الفاعل مديراً مع `admin_override = true`.

### 5. بناء قواعد الانتقال (هرمي)
1. **القواعد الأساسية** (الصلبة في الكود):
   ```
   CART       → [NEW, CANCELLED]
   NEW        → [PROCESSING, DELIVERY, CANCELLED]
   PROCESSING → [DELIVERY, CANCELLED]
   DELIVERY   → [COMPLETE, CANCELLED]
   ```
2. تُدمج مع `config('nano.orders::manager.edit_status.allowed_transitions')`.
3. تُدمج مع إعدادات الدور من `config("nano.orders::manager.edit_status.{admin|user|delivery}.allowed_transitions")`.
4. إذا وُجد `role_allowed_transitions` (callable)، يُستدعى و تُدمج نتيجته.
5. أخيراً تُدمج `allowed_transitions` من خيارات الاستدعاء المباشر.

### 6. تطبيق التغييرات حسب الدور

#### المدير (`admin`)
- يمكنه تغيير `order_states_ref_type` (مع الالتزام بقواعد الانتقال أو تجاوزها إذا `admin_override`).
- يمكنه تغيير `user_status` و `delivery_status` بشكل منفصل.
- يمكنه تسجيل أسباب الإلغاء (`because_cancel`, `delivery_because_cancel`).
- عند تعيين الحالة الرئيسية إلى `COMPLETE` أو `CANCELLED`، تُنسخ تلقائياً إلى `user_status` و `delivery_status`.

#### المالك (`user`)
- **لا يمكنه** تغيير `order_states_ref_type` أو `delivery_status`.
- يمكنه تغيير `user_status` فقط (مع قواعد الانتقال).
- إذا اختار `CANCELLED`، يمكنه تسجيل `because_cancel`.
- إذا تساوت `user_status` مع `delivery_status` بعد التعديل، تُحدّث `order_states_ref_type` تلقائياً.

#### الموصل (`delivery`)
- **لا يمكنه** تغيير `order_states_ref_type`.
- يمكنه تغيير `delivery_status` (مع قواعد الانتقال).
- يمكنه تغيير `user_status` **فقط** من `NEW` إلى `DELIVERY` (بدء التوصيل).
- إذا اختار `CANCELLED` لـ `delivery_status`، يمكنه تسجيل `delivery_because_cancel`.
- المزامنة التلقائية تطبق أيضاً.

### 7. الحفظ، الأحداث، والسجلات
إذا كان `is_save = true`:
- تُحدّث التواريخ تلقائياً (`shipped_at`, `delivery_at`, `end_date`).
- يُحفظ الطلب داخل معاملة `DB::transaction`.
- إذا `is_logs = true`، تُسجّل تغييرات الحالة في `StatusHistory`.
- إذا `is_event = true`، تُطلق الأحداث:
  - `nano.orders.newOrderStatus`
  - `nano.orders.newUserOrderStatus`
  - `nano.orders.newDeliveryOrderStatus`

---

## 📦 هيكل الاستجابة

النجاح:
```json
{
    "code": 200,
    "status": true,
    "message": "تم تحديث حالة الطلب بنجاح.",
    "error": null,
    "errors": null,
    "model": "<Order object>",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "CANCELLED",
        "delivery_status": "CANCELLED",
        "because_cancel": "طلب مكرر",
        "delivery_because_cancel": null
    },
    "input_data": { ... },
    "process_data": { ... }
}
```

الفشل:
```json
{
    "code": 400,
    "status": false,
    "message": "فشل تحديث حالة الطلب.",
    "error": "لا يمكن الانتقال من COMPLETE إلى CANCELLED.",
    "errors": null,
    "model": null,
    "data": null
}
```

---

## 🧪 أمثلة عملية

### مثال 1: مدير يلغي طلباً (مع تجاوز القيود)
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'                 => $order,
    'actor'                 => $adminUser,
    'order_states_ref_type' => 'CANCELLED',
    'because_cancel'        => 'منتهي الصلاحية',
    'admin_override'        => true,  // يسمح بالانتقال حتى من COMPLETE
]);
```

### مثال 2: مالك الطلب يلغي طلبه
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'         => $order,
    'user_status'   => 'CANCELLED',
    'because_cancel'=> 'غيرت رأيي',
]);
// إذا كان delivery_status أيضاً CANCELLED، تصبح order_states_ref_type = CANCELLED تلقائياً
```

### مثال 3: موصل يبدأ التوصيل
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'        => $order,
    'user_status'  => 'DELIVERY',     // من NEW -> DELIVERY فقط
    'actor'        => $courierUser,
]);
```

### مثال 4: موصل يكمل التوصيل
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'            => $order,
    'delivery_status'  => 'COMPLETE',
    'actor'            => $courierUser,
]);
// إذا كان user_status أيضاً COMPLETE، تصبح order_states_ref_type = COMPLETE
```

### مثال 5: اختبار بدون حفظ (تجربة)
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'        => $order,
    'user_status'  => 'CANCELLED',
    'is_save'      => false,   // لن يحفظ في قاعدة البيانات
]);
echo $result['data']['user_status']; // "CANCELLED"
```

### مثال 6: تخصيص قواعد الانتقال لكل دور عبر callable
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'                    => $order,
    'user_status'              => 'ON_HOLD',
    'role_allowed_transitions' => function ($role, $current) {
        if ($role === 'user') {
            $current['NEW'][] = 'ON_HOLD';
        }
        return $current;
    },
]);
```

### مثال 7: تعطيل الأحداث والسجلات (عملية صامتة)
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'    => $order,
    'user_status' => 'CANCELLED',
    'is_event' => false,
    'is_logs'  => false,
]);
```

---

## 🌐 استخدام الدالة عبر API

### نقطة النهاية
```
POST /api/v1/orders/orders/update-status
```

### الصلاحيات
- **مالك الطلب** (`user_id`) – يمكنه تحديث `user_status` فقط.
- **موصل الطلب** (`delivery_user_id`) – يمكنه تحديث `delivery_status` و بدء التوصيل (`user_status` من `NEW` إلى `DELIVERY`).
- **مستخدم لوحة التحكم** (`Backend\Models\User`) – صلاحيات كاملة.

### مثال طلب (مالك يلغي)
```bash
curl -X POST https://yourdomain.com/api/v1/orders/orders/update-status \
  -H "Authorization: Bearer {user_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 125,
    "user_status": "CANCELLED",
    "because_cancel": "طلب خاطئ"
  }'
```

### مثال طلب (موصل يكمل التوصيل)
```bash
curl -X POST https://yourdomain.com/api/v1/orders/orders/update-status \
  -H "Authorization: Bearer {courier_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 125,
    "delivery_status": "COMPLETE"
  }'
```

### مثال طلب (مدير يلغي مع تجاوز)
```bash
curl -X POST https://yourdomain.com/api/v1/orders/orders/update-status \
  -H "Authorization: Bearer {admin_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 125,
    "order_states_ref_type": "CANCELLED",
    "admin_override": true,
    "because_cancel": "مخالف للشروط"
  }'
```

### مثال استجابة
```json
{
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "CANCELLED",
        "delivery_status": "CANCELLED",
        "because_cancel": "مخالف للشروط",
        "delivery_because_cancel": null
    }
}
```

---

## ❗ معالجة الأخطاء

| الخطأ | السبب |
|-------|--------|
| `لا يمكن تعديل حالة طلب مكتمل.` | `order_states_ref_type` تساوي `COMPLETE` ولا يوجد `admin_override`. |
| `لا يمكن الانتقال من X إلى Y.` | الانتقال غير موجود في قواعد الانتقال. |
| `ليس لديك صلاحية تغيير الحالة الرئيسية للطلب.` | مالك أو موصل يحاول تغيير `order_states_ref_type`. |
| `ليس لديك صلاحية تغيير حالة الموصل.` | مالك يحاول تغيير `delivery_status`. |
| `يمكنك فقط تغيير حالة العميل من NEW إلى DELIVERY.` | موصل يحاول تغيير `user_status` لغير هاتين الحالتين. |
| `ليس لديك صلاحية لتعديل هذا الطلب.` | مستخدم غير مرتبط بالطلب وليس مديراً. |

---

## 🛠️ التخصيص عبر الإعدادات (Config)

يمكن تخصيص قواعد الانتقال والصلاحيات لكل دور عبر ملف `config.php`:

```php
'nano.orders::manager' => [
    'edit_status' => [
        // انتقالات عامة (تُدمج مع القواعد الأساسية)
        'allowed_transitions' => [
            'PROCESSING' => ['DELIVERY', 'CANCELLED', 'ON_HOLD'],
        ],

        // انتقالات خاصة بالمدير
        'admin' => [
            'allowed_transitions' => [
                'DELIVERY' => ['COMPLETE', 'CANCELLED', 'PROCESSING'], // يسمح بالإرجاع
            ],
        ],

        // انتقالات خاصة بالمالك
        'user' => [
            'allowed_transitions' => [
                'NEW' => ['CANCELLED'],
            ],
        ],

        // انتقالات خاصة بالموصل
        'delivery' => [
            'allowed_transitions' => [
                'NEW' => ['DELIVERY'],
            ],
        ],
    ],
],
```

---

## ✅ أفضل الممارسات

- استخدم `is_save = false` أثناء الاختبارات.
- مرر `actor` صراحةً في العمليات الخلفية (Cron, Jobs) لتجنب الاعتماد على الجلسة.
- استفد من `role_allowed_transitions` لإضافة حالات مخصصة دون تعديل الكود الأساسي.
- تأكد من وجود مفاتيح الترجمة (`nano.orders::lang.manager.update_status.*`) برسائل واضحة.
- استخدم `admin_override` بحذر، ويفضل تسجيل سبب في `admin_notes`.

---

## 🔚 الخاتمة

دالة `updateOrderStatusAdvanced` تغطي كامل الاحتياجات المتعلقة بتحديث حالة الطلب في نظام متعدد الأطراف. بفضل تصميمها الهرمي للإعدادات وخياراتها الواسعة، يمكن تطويعها لتناسب أي سيناريو عمل، مع الحفاظ على أمان وكفاءة عاليين. الدمج مع API عبر نقطة النهاية الموحّدة يجعلها جاهزة للاستخدام الفوري في تطبيقات الموبايل ولوحات التحكم.

**الوثائق المرجعية**:
- [توثيق `OrderManager` وسماته](./docs/Orders/Classes/Docs-OrderManager-Class-ar.md)
- [توثيق السمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-ar.md)
- [توثيق متقدم للسمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-ar.md)