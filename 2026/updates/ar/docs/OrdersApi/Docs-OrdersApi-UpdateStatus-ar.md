# Update Order Status – API Reference

تحديث حالة الطلب مع صلاحيات متدرجة (المالك، الموصل، أو مدير النظام). تعتمد هذه النهاية على الدالة الاحترافية `updateOrderStatusAdvanced` الموجودة في `OrderManager`.

---

## Endpoint

```
POST /api/v1/orders/orders/update-status
```

---

## Permissions & Roles

| الدور | الحقول المسموح بها | ملاحظات |
|-------|-------------------|---------|
| **مدير النظام** (`Backend\Models\User`) | `order_states_ref_type`, `user_status`, `delivery_status`, `because_cancel`, `delivery_because_cancel` | يمكنه تجاوز قواعد الانتقال باستخدام `admin_override: true`. عند تعيين الحالة الرئيسية إلى `COMPLETE` أو `CANCELLED`، تُنسخ تلقائياً إلى `user_status` و `delivery_status`. |
| **مالك الطلب** (`user_id`) | `user_status`, `because_cancel` | لا يمكنه تغيير `order_states_ref_type` أو `delivery_status`. إذا تساوت `user_status` مع `delivery_status` بعد التحديث، تُحدّث الحالة الرئيسية تلقائياً. |
| **موصل الطلب** (`delivery_user_id`) | `delivery_status`, `delivery_because_cancel`، و `user_status` فقط من `NEW` → `DELIVERY` | يمكن للموصل بدء التوصيل بتغيير حالة العميل من `NEW` إلى `DELIVERY`. نفس مبدأ المزامنة التلقائية ينطبق هنا. |

> إذا لم يتم تمرير أي مستخدم، تستخدم النهاية المستخدم المسجّل دخوله حالياً (مدير، أو مستخدم أمامي، أو موصل).

---

## Request Body (JSON)

| المفتاح | النوع | مطلوب | الوصف |
|--------|-------|--------|-------|
| `id` أو `order_id` | `integer` | **نعم** | معرّف الطلب |
| `order_states_ref_type` | `string` | لا | الحالة الرئيسية الجديدة: `NEW`, `PROCESSING`, `DELIVERY`, `CANCELLED`, `COMPLETE` |
| `user_status` | `string` | لا | حالة الطلب من جهة العميل |
| `delivery_status` | `string` | لا | حالة الطلب من جهة الموصل |
| `because_cancel` | `string` | لا | سبب الإلغاء من المالك |
| `delivery_because_cancel` | `string` | لا | سبب الإلغاء من الموصل |
| `is_save` | `boolean` | لا | حفظ التغييرات في قاعدة البيانات (افتراضي `true`). اجعله `false` للتجربة بدون حفظ. |
| `is_event` | `boolean` | لا | إطلاق الأحداث مثل `nano.orders.newOrderStatus` (افتراضي `true`). |
| `is_logs` | `boolean` | لا | تسجيل تاريخ الحالة في `StatusHistory` (افتراضي `true`). |
| `skip_permission` | `boolean` | لا | تجاوز جميع فحوصات الصلاحيات (يعامل المستخدم كمدير). |
| `admin_override` | `boolean` | لا | للمدير فقط: تجاوز قيود انتقال الحالات (مثلاً إرجاع حالة). |
| `custom_message` | `string` | لا | رسالة نجاح مخصصة. |
| `custom_error` | `string` | لا | رسالة خطأ مخصصة. |
| `allowed_transitions` | `object` | لا | مصفوفة انتقالات مخصصة لهذا الطلب، تُدمج فوق القواعد العامة. |
| `role_allowed_transitions` | `callable` (يُمرر عبر الكود فقط) | لا | دالة لتوليد قواعد انتقال حسب الدور. |

---

## Response Structure

### Success (200)

```json
{
    "code": 200,
    "status": true,
    "message": "تم تحديث حالة الطلب بنجاح.",
    "error": null,
    "errors": null,
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

### Error (400 / 401)

```json
{
    "code": 400,
    "status": false,
    "message": "فشل تحديث حالة الطلب.",
    "error": "لا يمكن الانتقال من COMPLETE إلى NEW.",
    "errors": null,
    "data": [],
    "input_data": { ... },
    "process_data": []
}
```

---

## Examples

### 1. مدير يلغي طلباً مع تجاوز القيود

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <admin_token>
Content-Type: application/json

{
    "order_id": 125,
    "order_states_ref_type": "CANCELLED",
    "because_cancel": "مخالف للشروط",
    "admin_override": true
}
```

#### Response

```html
Status: 200 OK
```

```json
{
    "code": 200,
    "status": true,
    "message": "تم تحديث حالة الطلب بنجاح.",
    "error": null,
    "errors": null,
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

### 2. مالك الطلب يلغي طلبه

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <user_token>
Content-Type: application/json

{
    "order_id": 125,
    "user_status": "CANCELLED",
    "because_cancel": "غيرت رأيي"
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "تم تحديث حالة الطلب بنجاح.",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "CANCELLED",
        "delivery_status": "CANCELLED",
        "because_cancel": "غيرت رأيي",
        "delivery_because_cancel": null
    }
}
```

> تم تحديث الحالة الرئيسية تلقائياً لأن `user_status` تساوت مع `delivery_status` بعد التعديل.

---

### 3. موصل يبدأ التوصيل

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <courier_token>
Content-Type: application/json

{
    "order_id": 125,
    "user_status": "DELIVERY"
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "تم تحديث حالة الطلب بنجاح.",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "DELIVERY",
        "user_status": "DELIVERY",
        "delivery_status": "COMPLETE",
        "because_cancel": null,
        "delivery_because_cancel": null
    }
}
```

---

### 4. موصل يكمل التوصيل

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <courier_token>
Content-Type: application/json

{
    "order_id": 125,
    "delivery_status": "COMPLETE"
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "تم تحديث حالة الطلب بنجاح.",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "COMPLETE",
        "user_status": "COMPLETE",
        "delivery_status": "COMPLETE",
        "because_cancel": null,
        "delivery_because_cancel": null
    }
}
```

---

### 5. محاولة غير مصرح بها (مالك يحاول تغيير حالة الموصل)

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <user_token>
Content-Type: application/json

{
    "order_id": 125,
    "delivery_status": "COMPLETE"
}
```

#### Response

```html
Status: 400 Bad Request
```

```json
{
    "code": 400,
    "status": false,
    "message": "فشل تحديث حالة الطلب.",
    "error": "ليس لديك صلاحية تغيير حالة الموصل.",
    "errors": null,
    "data": []
}
```

---

### 6. انتقال غير قانوني (موصل يحاول إرجاع الحالة)

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <courier_token>
Content-Type: application/json

{
    "order_id": 125,
    "delivery_status": "NEW"
}
```

#### Response

```json
{
    "code": 400,
    "status": false,
    "message": "فشل تحديث حالة الطلب.",
    "error": "لا يمكن الانتقال من DELIVERY إلى NEW.",
    "errors": null,
    "data": []
}
```

---

### 7. إلغاء مع سبب من الموصل

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <courier_token>
Content-Type: application/json

{
    "order_id": 125,
    "delivery_status": "CANCELLED",
    "delivery_because_cancel": "العميل لا يجيب"
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "تم تحديث حالة الطلب بنجاح.",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "NEW",
        "delivery_status": "CANCELLED",
        "because_cancel": null,
        "delivery_because_cancel": "العميل لا يجيب"
    }
}
```

---

### 8. تجربة بدون حفظ (Dry-Run)

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <admin_token>
Content-Type: application/json

{
    "order_id": 125,
    "order_states_ref_type": "COMPLETE",
    "is_save": false
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "تم تحديث حالة الطلب بنجاح.",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "COMPLETE",
        "user_status": "COMPLETE",
        "delivery_status": "COMPLETE",
        "because_cancel": null,
        "delivery_because_cancel": null
    }
}
```

> لم يتم حفظ التغييرات فعلياً في قاعدة البيانات.

---

### 9. مستخدم غير مسجّل الدخول

```bash
POST /api/v1/orders/orders/update-status
Content-Type: application/json

{
    "order_id": 125,
    "user_status": "CANCELLED"
}
```

#### Response

```html
Status: 401 Unauthorized
```

```json
{
    "code": 401,
    "status": false,
    "message": "فشل تحديث حالة الطلب.",
    "error": "User Not found",
    "errors": null,
    "data": []
}
```

---

## ملاحظات هامة

- جميع الحقول النصية للحالات (`order_states_ref_type`, `user_status`, `delivery_status`) تُقبل بالقيم الظاهرة أعلاه.
- قواعد الانتقال الافتراضية تمنع الرجوع للخلف (مثلاً من `DELIVERY` إلى `NEW`)، ولكن يمكن للمدير تجاوزها باستخدام `admin_override: true`.
- عند تفعيل `is_save: false` يمكن استخدام النهاية لتجربة المنطق دون تأثير فعلي.
- الأحداث (`nano.orders.newOrderStatus` إلخ) تُطلق تلقائياً ما لم يتم تعطيلها بـ `is_event: false`.
- تسجيل التاريخ (`StatusHistory`) يتم بشكل افتراضي ويمكن تعطيله بـ `is_logs: false`.
- يمكن تخصيص قواعد الانتقال عبر الإعدادات `nano.orders::manager.edit_status.allowed_transitions` وكذلك حسب الدور.
- هيكل الاستجابة يشمل `input_data` (المدخلات المستلمة) و `process_data` (بيانات ما بعد المعالجة) لتسهيل تتبع العمليات.
