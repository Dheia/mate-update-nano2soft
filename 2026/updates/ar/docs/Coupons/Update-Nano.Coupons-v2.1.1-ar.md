## 2026-05-02 – 2026-05-08

**تحديث إضافة `Nano.Coupons` – الإصدار 2.1.1 – تحسينات شاملة على شرط "حد المنتج"**

---

### ملخص التحديثات

يقدم الإصدار **2.1.1** تحسينات جوهرية على ميزة "حد المنتج" التي تم إطلاقها في الإصدار 2.1.0. تم إعادة هيكلة منطق التحقق بالكامل عبر إضافة كلاس جديد ومستقل `ProductLimitValidator`، مما يوفر مرونة غير مسبوقة في التعامل مع قيود المنتج. يشمل التحديث دعم سيناريوهات متقدمة مثل تقييد عدد الطلبات لأنواع طلب محددة (بغض النظر عن المنتج)، ورسائل خطأ تفصيلية متعددة اللغات تُظهر متى يمكن للمستخدم إعادة الطلب، وتحسينات كبيرة في الأداء عبر التخزين المؤقت ومنع الاستعلامات المتكررة.

---

### 1. أهداف الإصدار

- **فصل منطق التحقق** عن شرط السلة إلى كلاس مستقل `ProductLimitValidator` لتسهيل إعادة الاستخدام والصيانة.
- **دعم سيناريوهات جديدة**: تحديد عدد الطلبات المسموحة لنوع طلب معين خلال فترة، بدون الحاجة إلى تقييد منتج محدد.
- **تحسين تجربة المستخدم** عبر رسائل خطأ واضحة ومفصلة تشمل الحد الأقصى، العدد الحالي، الفترة، وتاريخ إمكانية الطلب التالي.
- **تحسين كبير في الأداء** من خلال التخزين المؤقت للقيود (`Cache`) ومنع تكرار استعلامات قاعدة البيانات المكلفة.
- **توسيع واجهة الإدارة** لتشمل البحث عن المنتج (`recordfinder`) ودعم اختيار الوحدة للمنتج المحدد.
- **إضافة نطاقات جديدة** `scopeOnProductLimit` و `scopeNotOnProductLimit` في نموذج الكوبون.

---

### 2. المكونات المطورة

#### 2.1 كلاس `ProductLimitValidator` (جديد)

الموقع: `plugins/nano/coupons/classes/ProductLimitValidator.php`

كلاس مستقل ومتخصص في فحص قيود حد المنتج. يقبل المستخدم وخيارات التهيئة (`use_cache`, `cache_ttl`) ويوفر دوال عامة شاملة:

| الدالة | الوصف |
|--------|-------|
| `checkOrFail(...)` | فحص شامل يرمي استثناءً مع رسالة خطأ مفصلة عند تجاوز الحدود. |
| `check(...)` | فحص صامت يعيد `true/false`. |
| `getLimits(...)` | جلب القيود المطبقة مع استراتيجيات متعددة (`auto`, `cache`, `fresh_db`, `direct_query`). |
| `getLimitsForProduct(...)` | اختصار لجلب قيود منتج محدد. |
| `checkOrderTypeLimitOrFail(...)` | فحص عدد طلبات نوع طلب معين. |
| `getOrderTypeLimits(...)` | جلب القيود الخاصة بنوع طلب معين. |
| `clearCache()` | مسح الكاش يدوياً. |
| `setUser(User $user)` | تعيين المستخدم. |

**استراتيجيات الجلب المدعومة:**
- `auto`: استخدام الكاش تلقائياً إذا كان مفعلاً.
- `cache`: فرض تحميل القيود من الكاش.
- `fresh_db`: تجاهل الكاش وتحميل مباشر من قاعدة البيانات.
- `direct_query`: تنفيذ استعلام مباشر على جدول الكوبونات بالشروط المحددة.

**أبرز التحسينات على المنطق:**

- **الحالة الأولى (منتج محدد):**
  - إذا كان `max_quantity > 0` يتم فحص الكمية الإجمالية (العربة + التاريخية + المضافة حديثاً).
  - إذا كان `max_orders > 0` يتم فحص عدد الطلبات التي تحتوي على هذا المنتج (أو وحدته).
  - احترام `units_id` و `order_restriction` و `shop_restriction` و `period_days`.

- **الحالة الثانية (بدون منتج – تقييد عدد الطلبات فقط):**
  - إذا كان `product_id` فارغاً، يتم تطبيق فحص `max_orders` بناءً على `order_types` و `shop_ids` و `period_days`.

#### 2.2 تحديثات شرط `ProductLimit` في السلة

الموقع: `plugins/nano/coupons/cartconditions/ProductLimit.php`

- تم تبسيط الكلاس ليكون وسيطاً بين نظام السلة (`CartCondition`) و `ProductLimitValidator`.
- دالة `validateCartItem` أصبحت تستخدم `ProductLimitValidator` مباشرة.
- تم إيقاف العمل بـ `beforeApply` مؤقتاً لمنع أي عبء إضافي أثناء حساب إجمالي السلة.
- الاعتماد على الكاش عبر `Cache::remember` في `loadAllProductLimits` مع تخزين `is_expired` لتجنب مشاكل serialization.

#### 2.3 تحديثات نموذج `Coupon` و `Coupons_model`

- إضافة خصائص الكاست `'product_id' => 'integer'`, `'units_id' => 'integer'`, `'max_quantity' => 'integer'`, `'max_orders' => 'integer'`, `'period_days' => 'integer'`.
- إضافة دالة `getProductIdOptions` لجلب قائمة المنتجات النشطة.
- توسيع دالة `getUnitsIdOptions` لجلب وحدات منتج محدد بشكل ديناميكي.
- إضافة النطاقات `scopeOnProductLimit` و `scopeNotOnProductLimit` لفلترة الكوبونات.
- في `afterSave` يتم مسح الكاش `nano.coupons.product_limits` عند تعديل/إضافة كوبون `product_limit`.

#### 2.4 تحديثات واجهة الإدارة (`fields.yaml`)

- تغيير `product_id` من `type: dropdown` إلى `type: recordfinder` للبحث المتقدم عن المنتجات.
- إضافة دعم الترجمة لخصائص `placeholder`, `emptyOption`, `title`, `prompt`.
- تحديث `units_id` ليعمل بـ `dependsOn: [product_id]` لتحديث قائمة الوحدات ديناميكياً.
- إظهار حقول `product_id`, `units_id`, `max_quantity`, `max_orders`, `period_days` فقط عند اختيار `apply_coupon_on = product_limit` عبر `trigger`.

#### 2.5 ملفات الترجمة (`lang/ar/lang.php`)

- مفاتيح جديدة لرسائل حد المنتج مع بارامترات:
  - `max_quantity_exceeded` مع `:max` و `:count`.
  - `max_orders_exceeded` مع `:max`, `:count`, `:days`, `:next_date`.
  - `unlimited` وعناصر تسميات الحقول الجديدة.

#### 2.6 ملف التهيئة (`config.php`)

- إضافة المفتاح `is_support_product_limit` مع القيمة الافتراضية `false` (للتحكم في إظهار النوع في القوائم).

#### 2.7 تحديثات `Plugin.php` (Nano.Coupons)

- تسجيل شرط `ProductLimit` في `registerCartConditions` فقط عند وجود الكلاس.
- تعديل `bindCouponsEvent` لإضافة `notOnProductLimit()` لتجنب تطبيق الكوبونات التلقائية على سلة فيها قيد منتج.

---

### 3. آلية العمل (التدفق الجديد)

1. **الإعداد (مرة واحدة):** ينشئ المسؤول كوبون `product_limit`، ويختار إما منتج معين (ووحدة اختيارية) أو يتركه فارغاً لتقييد عدد الطلبات حسب نوع الطلب.
2. **تحميل القيود:** عند تحميل العربة، يتم تحميل القيود من الكاش (أو قاعدة البيانات) مرة واحدة.
3. **فحص فوري عند إضافة منتج:** `ProductLimit` يستدعي `ProductLimitValidator::checkOrFail` باستخدام الحدث `nano.cart.adding`.
4. **فحص الكمية / عدد الطلبات:**
   - للمنتج: الكمية الإجمالية + الكمية الجديدة ≤ `max_quantity`.
   - للمنتج أو نوع الطلب: عدد الطلبات السابقة < `max_orders`.
5. **رسالة خطأ مفصلة:** "تجاوزت الحد المسموح لعدد الطلبات (الحد الأقصى 3). عدد الطلبات الحالي: 3. الفترة: 7 أيام. يمكنك الطلب مرة أخرى بعد 2026-05-10."

---

### 4. أبرز التحسينات والإنجازات

- **أداء ممتاز:** استخدام `Cache` للقيود مع إمكانية مسحه تلقائياً عند تعديل الكوبون.
- **استعلامات مجمعة:** دوال `getBulkHistoricalQuantities` و `getBulkHistoricalOrderCounts` تجمع البيانات في استعلام واحد.
- **مرونة عالية:**
  - فحص منتج معين أو نوع طلب معين أو كليهما.
  - فحص كمية أو عدد طلبات أو كليهما.
  - إمكانية تحديد `unit_id` اختيارياً.
- **رسائل خطأ احترافية:** تعرض الحد الأقصى، الاستخدام الحالي، الفترة، وتاريخ السماح بإعادة الطلب.
- **دعم كامل للترجمة** (عربي/إنجليزي).
- **فصل الاهتمامات:** `ProductLimitValidator` قابل لإعادة الاستخدام في أي مكان (API، CLI، الخ) بدون الاعتماد على نظام السلة.

---

### 5. المتطلبات والترقية

- **تحديث الملفات:**
  - `plugins/nano/coupons/classes/ProductLimitValidator.php` (جديد)
  - `plugins/nano/coupons/cartconditions/ProductLimit.php`
  - `plugins/nano/coupons/models/Coupon.php`
  - `plugins/nano/coupons/models/Coupons_model.php`
  - `plugins/nano/coupons/models/coupon/fields.yaml`
  - `plugins/nano/coupons/lang/ar/lang.php`
  - `plugins/nano/coupons/Plugin.php`
  - `plugins/nano/coupons/config/config.php`
- **تشغيل الهجرة:** لا توجد هجرات جديدة في هذا الإصدار (الأعمدة أضيفت في 2.1.0).
- **مسح الكاش:** يوصى بتنفيذ `php artisan cache:clear` بعد الترقية لضمان تحميل ملفات الترجمة الجديدة.

---

### 6. ملاحظات إضافية

- يمكن تفعيل/تعطيل ميزة `product_limit` من ملف البيئة `.env` عبر المفتاح `NANO_COUPONS_IS_SUPPORT_PRODUCT_LIMIT`.
- للحصول على أداء أفضل مع عدد كبير من الكوبونات، يُنصح بضبط `cache_ttl` إلى مدة مناسبة (الافتراضي 300 ثانية).
- جميع الاستعلامات الآمنة ضد SQL Injection وتستخدم ربط المعاملات (Parameter Binding).

---

### 7. خطط التطوير المستقبلية

- دعم تحديد عدة منتجات في كوبون `product_limit` واحد.
- تطوير واجهة تقارير لمتابعة استهلاك العملاء لحدود المنتجات.
- إضافة القدرة على تعطيل القيد لفئات عملاء محددة.

---

### 8. الخاتمة

يُكمل الإصدار 2.1.1 بناء نظام "حد المنتج" ويجعله أكثر قوة ومرونة واحترافية. بفضل `ProductLimitValidator` المنفصل، يمكن للمطورين دمج فحوصات الحدود في أي مكان بسهولة، بينما تضمن تحسينات الأداء تجربة سلسة للمستخدم النهائي حتى مع وجود عدد كبير من القيود النشطة.


**الوثائق المرجعية**:

- [توثيق كلاس `ProductLimitValidator`](./docs/Coupons/Classes/Docs-ProductLimitValidator-Class-ar.md)
- [توثيق شرط السلة `ProductLimit`](./docs/Coupons/CartConditions/Docs-ProductLimit-CartCondition-ar.md)
- [توثيق شرط السلة `Coupon`](./docs/Coupons/CartConditions/Docs-Coupon-CartCondition-ar.md)
- [توثيق شرط السلة `AutoCoupon`](./docs/Coupons/CartConditions/Docs-AutoCoupon-CartCondition-ar.md)
- [توثيق شرط السلة `AutoCouponShipping`](./docs/Coupons/CartConditions/Docs-AutoCouponShipping-CartCondition-ar.md)

