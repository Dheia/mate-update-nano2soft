## شرح آلية عمل الدالة خطوة بخطوة

الدالة `updateOrderStatusAdvanced` تمر بعدة مراحل متسلسلة تضمن الأمان، المرونة، والتوافق مع مختلف السيناريوهات. فيما يلي تفصيل دقيق لكل خطوة.

---

### 🔹 الخطوة 1: تهيئة هيكل الاستجابة

يتم إنشاء مصفوفة `$result` بالبنية الموحدة (code, status, message, error, errors, model, data) بالإضافة إلى متغيرات الرسائل الافتراضية ومصفوفات `input_data` و `process_data`.

---

### 🔹 الخطوة 2: تجهيز الإعدادات (الدمج الهرمي)

تتم قراءة الإعدادات من مستويات متعددة، مع إعطاء أولوية للأعلى:

1. **القيم المبدئية** (المشفّرة في الدالة نفسها).
2. **الإعدادات العامة** من `config('nano.orders::manager.edit_status')` مثل `is_save`، `is_event`، `allowed_transitions`.
3. **خيارات الاستدعاء المباشر** التي يمررها المطور في المصفوفة `$options`.

تستخدم الدالة المساعدة `$mergeConfig` لمقارنة القيم بالترتيب: الخيارات المباشرة ← الإعدادات العامة ← القيمة الافتراضية.

يتم استخراج جميع المتغيرات الداخلية (مثل `$actor`, `$order`, `$newOrderState`, `$isSave` ...) من الخيارات أو الإعدادات أو القيم الافتراضية.

---

### 🔹 الخطوة 3: تحديد الفاعل (Actor)

إذا لم يتم تمرير كائن المستخدم `actor` صراحةً، تبحث الدالة عن مستخدم مسجل الدخول حالياً بالترتيب التالي:
1. `\Nano\AuthApi\Classes\AuthHelpers::getCurrentUser()`
2. `\BackendAuth::getUser()`
3. `\Auth::getUser()`

هذا يضمن أن الدالة تعمل بشكل تلقائي دون الحاجة لتمرير المستخدم في كل مرة.

---

### 🔹 الخطوة 4: تصنيف الأدوار

بعد تحديد الفاعل، يتم تصنيفه إلى أحد الأدوار:

- **مدير (Admin)**: إذا كان الكائن من نوع `Backend\Models\User`.
- **مالك (Owner)**: إذا كان من نوع `RainLab\User\Models\User` ومعرفه يساوي `order->user_id`.
- **موصل (Courier)**: إذا كان من نوع `RainLab\User\Models\User` ومعرفه يساوي `order->delivery_user_id`.
- إذا كان المستخدم هو نفسه المالك والموصل معاً، يُعامل كـ **مالك** فقط.
- إذا كان الخيار `skip_permission` مفعّلاً، يُعامل الفاعل كمدير كامل الصلاحيات.

---

### 🔹 الخطوة 5: منع تعديل الطلبات المكتملة

إذا كانت الحالة الرئيسية للطلب هي `COMPLETE`، يتم رفض أي تغيير ما لم يكن الفاعل **مديراً** مع تفعيل خيار `admin_override`.

---

### 🔹 الخطوة 6: بناء قواعد الانتقال المسموحة (هرمي)

يتم بناء المصفوفة النهائية لقواعد الانتقال عبر دمج الطبقات التالية:

1. **القواعد الأساسية** (Hardcoded):
   ```
   CART       → [NEW, CANCELLED]
   NEW        → [PROCESSING, DELIVERY, CANCELLED]
   PROCESSING → [DELIVERY, CANCELLED]
   DELIVERY   → [COMPLETE, CANCELLED]
   ```

2. **الإعدادات العامة**: المفتاح `nano.orders::manager.edit_status.allowed_transitions` في ملف `config`.

3. **إعدادات الدور**: المفتاح `nano.orders::manager.edit_status.{role}.allowed_transitions` (حيث `role` هو `admin`, `user`, أو `delivery`).

4. **الدالة المخصصة**: إذا وُجد الخيار `role_allowed_transitions` وهو callable، يتم استدعاؤه ويمرر إليه `(role, currentTransitions)`، وتُدمج النتيجة.

5. **الخيارات المباشرة**: المفتاح `allowed_transitions` في `$options`.

يتم الدمج باستخدام `array_replace_recursive` للحفاظ على هيكل متعدد المستويات.

---

### 🔹 الخطوة 7: دالة التحقق من الانتقال

يتم إنشاء closure داخلي `$isValidTransition` الذي:
- إذا كانت القيمة الجديدة فارغة → يسمح (لا تغيير).
- إذا كان الفاعل مديراً و `admin_override` مفعّل → يسمح دائماً.
- عدا ذلك، يتحقق من وجود الحالة الجديدة ضمن القيم المسموحة للحالة الحالية.

---

### 🔹 الخطوة 8: تطبيق التغييرات حسب الدور

يتم فحص الدور وتنفيذ المنطق الخاص به:

#### أ. المدير (Admin)
- يمكنه تغيير الحالة الرئيسية `order_states_ref_type`.
- إذا كانت الحالة الجديدة `COMPLETE` أو `CANCELLED`، يتم نسخها فوراً إلى `user_status` و `delivery_status`.
- إذا لم يمرر حالة رئيسية، يمكنه تغيير `user_status` أو `delivery_status` بشكل منفصل.
- يمكنه تسجيل أسباب الإلغاء (`because_cancel` و `delivery_because_cancel`).

#### ب. المالك (Owner)
- **ممنوع** تغيير الحالة الرئيسية أو `delivery_status`.
- يمكنه فقط تغيير `user_status`.
- إذا ألغى الطلب (`CANCELLED`) وقدم سبباً، يسجل السبب في `because_cancel`.
- بعد التعديل، إذا أصبح `user_status` مطابقاً لـ `delivery_status`، يتم تحديث الحالة الرئيسية تلقائياً.

#### ج. الموصل (Courier)
- **ممنوع** تغيير الحالة الرئيسية.
- يمكنه تغيير `delivery_status`.
- يمكنه تغيير `user_status` **فقط** من `NEW` إلى `DELIVERY` (بدء التوصيل).
- إذا ألغى (`CANCELLED`) وقدم سبباً، يسجل في `delivery_because_cancel`.
- بعد التعديل، إذا تساوت `user_status` و `delivery_status`، تُحدّث الحالة الرئيسية تلقائياً.

#### د. أدوار أخرى
- أي فاعل لا ينطبق عليه أي من الأدوار أعلاه، يتم رفض العملية.

---

### 🔹 الخطوة 9: الحفظ، السجلات، والأحداث

إذا كان `is_save` مفعلاً وحدث تغيير فعلي في الطلب:

1. **تحديث التواريخ تلقائياً**:
   - إذا تغيرت `order_states_ref_type` إلى `DELIVERY` → يتم تعيين `shipped_at` و `delivery_at` إن لم تكونا موجودتين.
   - إذا أصبحت `COMPLETE` أو `CANCELLED` → يتم تعيين `end_date`.

2. **حفظ الطلب** داخل معاملة قاعدة بيانات لضمان التكامل.

3. **تسجيل تاريخ الحالة** (إذا `is_logs` مفعّل والكلاس `Nano2\StatusHistory\Classes\Manager` موجود):
   - يتم تسجيل تغيير كل حقل من الحقول الثلاثة (`order_states_ref_type`, `user_status`, `delivery_status`) بشكل منفصل مع ربطه بالفاعل.

4. **إطلاق الأحداث** (إذا `is_event` مفعّل):
   - `nano.orders.newOrderStatus` عند تغيير الحالة الرئيسية.
   - `nano.orders.newUserOrderStatus` عند تغيير `user_status`.
   - `nano.orders.newDeliveryOrderStatus` عند تغيير `delivery_status`.

---

### 🔹 الخطوة 10: تجهيز المخرجات

تتم تعبئة `$output_process_data` بالحالة النهائية للطلب، وتوضع في `$result['data']` و `$result['model']`. كما يتم وضع `$output_input_data` في `$result['input_data']`.

---

### 🔹 الخطوة 11: معالجة الأخطاء

أي استثناء يتم التقاطه في الكتلة `catch (\Throwable $e)`:

- يتم التراجع عن المعاملة (rollback) إذا كانت نشطة.
- تملأ الحقول: `code`, `status`, `message`, `error`.
- في بيئة التطوير (`debug = true`)، تضاف تفاصيل التتبع (`debug`).

---

### 🔹 الخطوة 12: الإرجاع النهائي

تُرجع الدالة المصفوفة `$result` مهما كانت النتيجة.

**الوثائق المرجعية**:
- [توثيق `OrderManager` وسماته](./docs/Orders/Classes/Docs-OrderManager-Class-ar.md)
- [توثيق السمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-ar.md)
- [توثيق متقدم للسمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-ar.md)