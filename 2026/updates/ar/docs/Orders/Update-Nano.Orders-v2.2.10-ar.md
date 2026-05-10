## 2026-05-06 – 2026-05-08

**تحديثات إضافة `Nano.Orders` – الإصدار 2.2.10**  
**تحديثات إضافة `Nano.OrdersApi` – الإصدار 1.0.20**

### ملخص التحديثات

شهدت إضافة `Nano.Orders` تحديثاً محورياً في الإصدار **2.2.10** ركز على تمكين الفلترة المتقدمة للطلبات بناءً على مالك المنتج، وتطوير آلية احترافية لتحديث حالة الطلب بصلاحيات متدرجة وأدوار متعددة، إلى جانب تحسينات في `OrderHelper` لإدارة المستخدمين والموصلين.  
في المقابل، جاء الإصدار **1.0.20** من `Nano.OrdersApi` ليُتيح واجهة API موحّدة لتحديث حالة الطلب، مستفيداً من الدوال الجديدة في `OrderManager`، مع دعم كامل للخيارات المخصصة والصلاحيات.

---

## Nano.Orders v2.2.10 – فلترة مالك المنتج وتحديث حالة الطلب بصلاحيات متدرجة

### أهداف الإصدار

- **توفير نطاقات استعلام متقدمة** لفلترة الطلبات بناءً على ملكية المنتجات المرتبطة.
- **إنشاء دالة احترافية متكاملة** لتحديث حالة الطلب بأدوار مختلفة (مدير، مالك، موصل) مع قواعد انتقال مرنة وصلاحيات قابلة للتخصيص.
- **إضافة دوال مساعدة** لاستخراج الموصل من المستخدم والعكس.
- **دمج فلترة مالك المنتج** داخل `OrderHelper::getOrdersRecords` لتقارير أكثر ذكاءً.
- **دعم الإعدادات الخارجية** (`config`) للتحكم بقواعد الانتقال والصلاحيات.

### الميزات الجديدة

#### 1. ترايت `HasProductOwnerScopes` في نموذج `Order`

تم إنشاء الترايت في المسار `Nano\Orders\Models\Order\HasProductOwnerScopes` ويضم نطاقات ودوالاً متقدمة لفلترة الطلبات حسب مالك المنتج:

| النطاق / الدالة | الوصف |
|----------------|--------|
| `scopeWhereHasProductsByOwner` | نطاق رئيسي بمصفوفة خيارات شاملة: `user`, `count`, `not`, `boolean`, `conditions`, `itemConditions`. |
| `scopeWhereProductOwner` | اختصار للاستخدام السريع (وجود منتج واحد على الأقل). |
| `scopeHasProductsByOwner` | فلترة حسب عدد أدنى من المنتجات المملوكة. |
| `scopeDoesntHaveProductsByOwner` | فلترة الطلبات التي لا تحتوي أي منتج مملوك. |
| `scopeHasProductsByOwnerCount` | تحديد عدد دقيق (أو بمقارنة) للمنتجات المملوكة. |
| `scopeOrWhereHasProductsByOwner` | ربط النطاق مع `OR`. |
| `scopeWithProductsByOwnerCount` | إضافة عمود محسوب بعدد المنتجات المملوكة. |
| `scopeSortByProductsByOwnerCount` | ترتيب الطلبات حسب عدد المنتجات المملوكة. |
| `hasAnyProductByOwner` | دالة فحص على كائن الطلب الواحد. |
| `applyOwnerToProductQuery` | دالة مساعدة داخلية لتطبيق شرط المالك على استعلام منتج. |

جميع النطاقات تستخدم ثوابت أسماء الجداول (`Product::TABLE_NAME`) لضمان توافق دائم.

**مثال استخدام:**
```php
// طلبات تحتوي منتجين مملوكين للمستخدم الحالي
$orders = Order::whereHasProductsByOwner([
    'count' => 2,
    'countOperator' => '>='
])->get();
```

#### 2. دالة `updateOrderStatusAdvanced` في `StepStatus` ترايت

أُضيفت دالة متكاملة تُلبي جميع متطلبات تحديث حالة الطلب مع صلاحيات متدرجة:

| الميزة | الوصف |
|--------|--------|
| **تحديد الفاعل تلقائياً** | يستخدم المستخدم المسجّل دخوله (مدير باكند، مالك، موصل). |
| **ثلاث حقول للحالة** | `order_states_ref_type` (للإدارة)، `user_status` (للمالك)، `delivery_status` (للموصل). |
| **قواعد انتقال قابلة للتخصيص** | افتراضية + من الإعدادات + خاصة بالدور + مخصصة لكل استدعاء. |
| **منع تعديل المكتمل** | لا يُسمح بتغيير حالة طلب مكتمل (إلا بتجاوز إداري). |
| **مزامنة تلقائية** | عند تساوي `user_status` مع `delivery_status` تُحدّث الحالة الرئيسية. |
| **أسباب الإلغاء** | `because_cancel` (مالك) و `delivery_because_cancel` (موصل). |
| **خيارات تحكم شاملة** | `is_save`, `is_event`, `is_logs`, `skip_permission`, `admin_override`, `custom_message`. |
| **هيكل استجابة موحد** | يحتوي على `input_data`, `process_data`, `model`, `debug`. |

**أمثلة أدوار:**
- **المدير**: يمكنه تغيير أي حقل، مع إمكانية تجاوز قيود الانتقال (`admin_override`).
- **المالك**: يغير `user_status` فقط، ويمكنه إلغاء الطلب مع سبب.
- **الموصل**: يغير `delivery_status`، ويمكنه فقط تحريك `user_status` من `NEW` إلى `DELIVERY`.

#### 3. دوال مساعدة في `OrderHelper`

- `getDeliveryByUser($user)` – استخراج كائن الموصل من المستخدم.
- `getUserByDelivery($delivery)` – استخراج كائن المستخدم من الموصل.

#### 4. دعم فلترة مالك المنتج في `OrderHelper::getOrdersRecords`

أُضيفت الخيارات التالية:
- `is_has_products_by_owner` (تفعيل الفلتر)
- `products_by_owner_*` (جميع خيارات النطاق الرئيسي)
- `is_has_products_by_owner_or_delivery` لدمج الفلتر مع شروط التوصيل بـ `OR`.

#### 5. الإعدادات الجديدة في `config.php`

```php
'nano.orders::manager.edit_status.allowed_transitions'
'nano.orders::manager.edit_status.admin.allowed_transitions'
'nano.orders::manager.edit_status.user.allowed_transitions'
'nano.orders::manager.edit_status.delivery.allowed_transitions'
```

---

## Nano.OrdersApi v1.0.20 – نقطة API لتحديث حالة الطلب

### أهداف الإصدار

- **توفير واجهة API موحدة** لتحديث حالة الطلب من قبل المالك، الموصل، أو المدير.
- **دمج كامل مع `updateOrderStatusAdvanced`** لضمان اتساق الصلاحيات والمنطق.
- **دعم الخيارات المخصصة** عبر الطلب JSON.
- **الحفاظ على هيكل استجابة متسق** مع باقي API.

### الميزات الجديدة

#### 1. نقطة النهاية `POST /api/v1/orders/orders/update-status`

تستقبل الطلب JSON بالحقول التالية (جميعها اختيارية وتعتمد على الدور):
- `order_id` أو `id`
- `order_states_ref_type`
- `user_status`
- `delivery_status`
- `because_cancel`
- `delivery_because_cancel`
- `is_save`, `is_event`, `is_logs`
- `skip_permission`, `admin_override`
- `custom_message`, `custom_error`

#### 2. دالة `updateStatus` في المتحكم `Orders`

تستخرج الطلب، تجهز الخيارات، وتستدعي `OrderManager::updateOrderStatusAdvanced()` مع تمرير جميع المُعامِلات، وتُعيد استجابة موحدة تحتوي على الحالة الجديدة.

**مثال استجابة ناجحة:**
```json
{
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "CANCELLED",
        "delivery_status": "CANCELLED",
        "because_cancel": "طلب مكرر"
    }
}
```

---

## ملخص الإصدارات (2.2.10 و 1.0.20)

| الإصدار | أبرز الميزات |
|---------|---------------|
| **Nano.Orders 2.2.10** | `HasProductOwnerScopes` (نطاقات فلترة مالك المنتج)، `updateOrderStatusAdvanced` (تحديث حالة بصلاحيات متدرجة)، دوال `getDeliveryByUser` / `getUserByDelivery`، دعم فلترة المالك في `OrderHelper::getOrdersRecords`، إعدادات مرنة لقواعد الانتقال. |
| **Nano.OrdersApi 1.0.20** | نقطة API `POST orders/update-status`، دمج مع الدالة الاحترافية، دعم جميع الخيارات المخصصة، استجابة موحدة. |

---

### متطلبات الترقية

1. **تحديث الملفات**:
   - إضافة ترايت `HasProductOwnerScopes` في `models/Order/HasProductOwnerScopes.php`.
   - إضافة ترايت `StepStatus` في `traits/steps/StepStatus.php`.
   - تحديث `OrderHelper.php` بالدوال الجديدة ودعم فلترة المالك.
   - تحديث `OrderManager.php` لاستخدام `StepStatus`.
   - تحديث `Orders.php` في `OrdersApi` لإضافة دالة `updateStatus`.
   - تحديث `routes.php` في `OrdersApi` لإضافة المسار الجديد.

2. **لا توجد هجرات جديدة**: الإصداران لا يتطلبان تغييرات في قاعدة البيانات.

3. **ملفات الترجمة**:
   - أضف ملف `lang/ar/manager/update_status.php` في `Nano.Orders` بالمفاتيح المطلوبة.

4. **الإعدادات الاختيارية**:
   - يمكن إضافة قواعد انتقال مخصصة في `config.php` تحت `nano.orders::manager.edit_status`.

5. **اختبار التوافق**:
   - اختبار النطاقات الجديدة مع استعلامات مختلفة.
   - اختبار تحديث الحالة من الأدوار الثلاثة.
   - اختبار API من تطبيقات الموبايل (باستخدام توكن المستخدم المناسب).

---

### الخاتمة

يمثل الإصداران **Nano.Orders 2.2.10** و **Nano.OrdersApi 1.0.20** نقلة نوعية في مرونة نظام الطلبات. فمن جهة، أتاحت نطاقات `HasProductOwnerScopes` فلترة غير مسبوقة للطلبات بناءً على ملكية المنتجات، مما يلبي احتياجات الأسواق متعددة البائعين. ومن جهة أخرى، قدمت دالة `updateOrderStatusAdvanced` ونقطة API المدمجة معها نظاماً موحداً وآمناً لتحديث حالة الطلب من قبل جميع الأطراف (مدير، مالك، موصل) مع إعدادات قابلة للتخصيص بالكامل. الكود نظيف، موثّق، وقابل للتوسع بسهولة.

---

**الوثائق المرجعية**:
- [توثيق `OrderManager` وسماته](./docs/Orders/Classes/Docs-OrderManager-Class-ar.md)
- [توثيق السمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-ar.md)
- [توثيق متقدم للسمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-ar.md)
- [توثيق نطاقات `HasProductOwnerScopes`](./docs/Orders/Models/orders/Docs-HasProductOwnerScopes-ar.md)
- [توثيق API الطلبات](./docs/OrdersApi/Docs-OrdersApi-ar.md)