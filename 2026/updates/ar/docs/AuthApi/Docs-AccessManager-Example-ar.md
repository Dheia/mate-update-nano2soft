## 12. مثال كامل لإعدادات متحكم API (العمليات list, show, create, update)

نقدم هنا مثالاً متكاملاً لإعدادات متحكم افتراضي باسم `items`، يوضح كيفية تكوين الصلاحيات لكل عملية باستخدام متغيرات البيئة. الهيكل نفسه ينطبق على أي مورد (مثل `absences`, `students`, `parents`).

### 12.1 هيكل ملف `config.php`

```php
<?php

return [
    'items' => [
        // ------------------------------
        // عملية list (جلب القائمة)
        // ------------------------------
        'list' => [
            'permission' => [
                'is_allow' => env('NANO_ITEMSAPI_ITEMS_LIST_IS_ALLOW', true),

                'backend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_LIST_BACKEND_ALLOW', true),
                    'check_permission'   => env('NANO_ITEMSAPI_ITEMS_LIST_CHECK_PERMISSION', true),
                    'permissions'        => ['tss.items.access_all', 'tss.items.access'],
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_LIST_BACKEND_SCOPE', 'all'),
                    'company_field'      => env('NANO_ITEMSAPI_ITEMS_LIST_COMPANY_FIELD', 'companys_id'),
                    'department_field'   => env('NANO_ITEMSAPI_ITEMS_LIST_DEPARTMENT_FIELD', 'departments_id'),
                    'state_field'        => env('NANO_ITEMSAPI_ITEMS_LIST_STATE_FIELD', 'state_id'),
                    'created_by_field'   => env('NANO_ITEMSAPI_ITEMS_LIST_CREATED_BY_FIELD', 'created_by'),
                ],

                'frontend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_LIST_FRONTEND_ALLOW', true),
                    'allowed_ref_types'  => explode(',', env('NANO_ITEMSAPI_ITEMS_LIST_ALLOWED_REF_TYPES', 'student,parent')),
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_LIST_FRONTEND_SCOPE', 'own'),
                    'children_relation'  => env('NANO_ITEMSAPI_ITEMS_LIST_CHILDREN_RELATION', 'students'),
                ],

                'guest' => [
                    'allow'         => env('NANO_ITEMSAPI_ITEMS_LIST_GUEST_ALLOW', false),
                    'access_scope'  => env('NANO_ITEMSAPI_ITEMS_LIST_GUEST_SCOPE', 'none'),
                ],

                'advanced_filters' => [
                    'enabled'                => env('NANO_ITEMSAPI_ITEMS_ADVANCED_FILTERS_ENABLED', true),
                    'default_for_backend'    => env('NANO_ITEMSAPI_ITEMS_ADV_FILTERS_BACKEND_DEFAULT', true),
                    'default_for_frontend'   => env('NANO_ITEMSAPI_ITEMS_ADV_FILTERS_FRONTEND_DEFAULT', false),
                    'allowed_fields_backend' => explode(',', env('NANO_ITEMSAPI_ITEMS_ADV_FILTERS_BACKEND_FIELDS', '*')),
                    'allowed_fields_frontend' => explode(',', env('NANO_ITEMSAPI_ITEMS_ADV_FILTERS_FRONTEND_FIELDS', '')),
                    'field_specific_rules'   => [
                        'student_id' => [
                            'backend' => true,
                            'frontend' => [
                                '*' => false,
                                'parent' => true,
                            ],
                        ],
                        'record_id' => [
                            'backend' => true,
                            'frontend' => [
                                '*' => false,
                                'parent' => true,
                            ],
                        ],
                    ],
                ],
            ],
        ],

        // ------------------------------
        // عملية show (عرض عنصر واحد)
        // ------------------------------
        'show' => [
            'permission' => [
                'is_allow' => env('NANO_ITEMSAPI_ITEMS_SHOW_IS_ALLOW', true),

                'backend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_SHOW_BACKEND_ALLOW', true),
                    'check_permission'   => env('NANO_ITEMSAPI_ITEMS_SHOW_CHECK_PERMISSION', true),
                    'permissions'        => ['tss.items.access', 'tss.items.access_all'],
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_SHOW_BACKEND_SCOPE', 'all'),
                    'company_field'      => env('NANO_ITEMSAPI_ITEMS_SHOW_COMPANY_FIELD', 'companys_id'),
                    'department_field'   => env('NANO_ITEMSAPI_ITEMS_SHOW_DEPARTMENT_FIELD', 'departments_id'),
                    'state_field'        => env('NANO_ITEMSAPI_ITEMS_SHOW_STATE_FIELD', 'state_id'),
                    'created_by_field'   => env('NANO_ITEMSAPI_ITEMS_SHOW_CREATED_BY_FIELD', 'created_by'),
                ],

                'frontend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_SHOW_FRONTEND_ALLOW', true),
                    'allowed_ref_types'  => explode(',', env('NANO_ITEMSAPI_ITEMS_SHOW_ALLOWED_REF_TYPES', 'student,parent')),
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_SHOW_FRONTEND_SCOPE', 'own'),
                    'children_relation'  => env('NANO_ITEMSAPI_ITEMS_SHOW_CHILDREN_RELATION', 'students'),
                ],

                'guest' => [
                    'allow'         => env('NANO_ITEMSAPI_ITEMS_SHOW_GUEST_ALLOW', false),
                    'access_scope'  => env('NANO_ITEMSAPI_ITEMS_SHOW_GUEST_SCOPE', 'none'),
                ],
            ],
        ],

        // ------------------------------
        // عملية create (إنشاء جديد)
        // ------------------------------
        'create' => [
            'permission' => [
                'is_allow' => env('NANO_ITEMSAPI_ITEMS_CREATE_IS_ALLOW', true),

                'backend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_CREATE_BACKEND_ALLOW', true),
                    'check_permission'   => env('NANO_ITEMSAPI_ITEMS_CREATE_CHECK_PERMISSION', true),
                    'permissions'        => ['tss.items.add'],
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_CREATE_BACKEND_SCOPE', 'all'),
                ],

                'frontend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_CREATE_FRONTEND_ALLOW', true),
                    'allowed_ref_types'  => explode(',', env('NANO_ITEMSAPI_ITEMS_CREATE_ALLOWED_REF_TYPES', 'student,parent')),
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_CREATE_FRONTEND_SCOPE', 'own'),
                ],

                'guest' => [
                    'allow'         => env('NANO_ITEMSAPI_ITEMS_CREATE_GUEST_ALLOW', false),
                    'access_scope'  => env('NANO_ITEMSAPI_ITEMS_CREATE_GUEST_SCOPE', 'none'),
                ],
            ],
        ],

        // ------------------------------
        // عملية update (تعديل)
        // ------------------------------
        'update' => [
            'permission' => [
                'is_allow' => env('NANO_ITEMSAPI_ITEMS_UPDATE_IS_ALLOW', true),

                'backend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_UPDATE_BACKEND_ALLOW', true),
                    'check_permission'   => env('NANO_ITEMSAPI_ITEMS_UPDATE_CHECK_PERMISSION', true),
                    'permissions'        => ['tss.items.edit'],
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_UPDATE_BACKEND_SCOPE', 'all'),
                ],

                'frontend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_UPDATE_FRONTEND_ALLOW', true),
                    'allowed_ref_types'  => explode(',', env('NANO_ITEMSAPI_ITEMS_UPDATE_ALLOWED_REF_TYPES', 'student,parent')),
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_UPDATE_FRONTEND_SCOPE', 'own'),
                ],

                'guest' => [
                    'allow'         => env('NANO_ITEMSAPI_ITEMS_UPDATE_GUEST_ALLOW', false),
                    'access_scope'  => env('NANO_ITEMSAPI_ITEMS_UPDATE_GUEST_SCOPE', 'none'),
                ],
            ],
        ],

        // ------------------------------
        // الإعدادات العامة للمورد (اختيارية)
        // ------------------------------
        'order_by'       => env('NANO_ITEMSAPI_ITEMS_ORDER_BY', 'sort_order'),
        'order_dir'      => env('NANO_ITEMSAPI_ITEMS_ORDER_DIR', 'asc'),
        'per_page'       => env('NANO_ITEMSAPI_ITEMS_PER_PAGE', 15),
        'exclude'        => env('NANO_ITEMSAPI_ITEMS_EXCLUDE', ''),
    ],
];
```

### 12.2 شرح المتغيرات البيئية المستخدمة

| المتغير البيئي | الغرض | القيم الممكنة (افتراضي) |
|----------------|--------|--------------------------|
| `NANO_ITEMSAPI_ITEMS_LIST_IS_ALLOW` | تفعيل عملية `list` | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_LIST_BACKEND_ALLOW` | السماح لمستخدمي Backend | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_LIST_CHECK_PERMISSION` | تفعيل فحص الصلاحيات لـ Backend | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_LIST_BACKEND_SCOPE` | نطاق الوصول لـ Backend | `all`, `own`, `company`, `department`, `state` |
| `NANO_ITEMSAPI_ITEMS_LIST_FRONTEND_ALLOW` | السماح لمستخدمي Frontend | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_LIST_ALLOWED_REF_TYPES` | أنواع `ref_type` المسموحة (مفصولة بفواصل) | `student,parent` |
| `NANO_ITEMSAPI_ITEMS_LIST_FRONTEND_SCOPE` | نطاق الوصول لـ Frontend | `own`, `children` |
| `NANO_ITEMSAPI_ITEMS_LIST_CHILDREN_RELATION` | اسم العلاقة لجلب الأبناء (لـ `children`) | `students` |
| `NANO_ITEMSAPI_ITEMS_LIST_GUEST_ALLOW` | السماح للضيوف | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_ADVANCED_FILTERS_ENABLED` | تفعيل الفلاتر المتقدمة | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_ADV_FILTERS_BACKEND_DEFAULT` | السماح الافتراضي لـ Backend | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_ADV_FILTERS_FRONTEND_DEFAULT` | السماح الافتراضي لـ Frontend | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_ADV_FILTERS_BACKEND_FIELDS` | الحقول المسموحة لـ Backend (مفصولة بفواصل) | `*` أو `field1,field2` |
| `NANO_ITEMSAPI_ITEMS_ADV_FILTERS_FRONTEND_FIELDS` | الحقول المسموحة لـ Frontend | `""` أو `field1,field2` |
| `NANO_ITEMSAPI_ITEMS_SHOW_IS_ALLOW` | تفعيل عملية `show` | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_CREATE_IS_ALLOW` | تفعيل عملية `create` | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_UPDATE_IS_ALLOW` | تفعيل عملية `update` | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_ORDER_BY` | حقل الترتيب الافتراضي | `sort_order` |
| `NANO_ITEMSAPI_ITEMS_ORDER_DIR` | اتجاه الترتيب | `asc` أو `desc` |
| `NANO_ITEMSAPI_ITEMS_PER_PAGE` | عدد النتائج الافتراضي لكل صفحة | عدد صحيح |
| `NANO_ITEMSAPI_ITEMS_EXCLUDE` | الأعمدة المستبعدة افتراضياً | `""` أو أسماء أعمدة مفصولة بفواصل |

---

### 12.3 كيفية استخدام هذه الإعدادات في متحكم `Items`

في متحكم `Items`، يمكنك استخدام دوال `AccessManager` المبسطة كما يلي:

```php
public function index()
{
    $user = $this->current_user;
    $access = AccessManager::checkByResource('nano.itemsapi::items.list', $user);
    // ...
}

public function show($id)
{
    $user = $this->current_user;
    // show قد تستخدم نفس إعدادات list إذا لم تُخصص، أو تستخدم checkWithFallback
    $access = AccessManager::checkWithFallback(
        'nano.itemsapi::items.show',
        'nano.itemsapi::items.list',
        $user
    );
    // ...
}

public function store()
{
    $user = $this->current_user;
    $access = AccessManager::checkByResource('nano.itemsapi::items.create', $user);
    // ...
}

public function update($id)
{
    $user = $this->current_user;
    $access = AccessManager::checkByResource('nano.itemsapi::items.update', $user);
    // ...
}
```

> **ملاحظة:** `checkByResource` تتوقع وجود مفتاح `permission` داخل الإعدادات. لذلك فإن هيكل `config.php` الموضح أعلاه يتوافق تماماً مع هذه الدوال.

---

### 12.4 استراتيجية تجاوز الإعدادات (Overrides)

يمكن تجاوز أي إعداد عن طريق تمرير `$overrideConfig` إلى دوال مثل `checkByResource` أو `getOperationByConfig`. مثلاً:

```php
$override = [
    'backend' => [
        'allow' => false,
    ],
];
$access = AccessManager::checkByResource('nano.itemsapi::items.list', $user, $override);
```

هذا مفيد عندما تريد تغيير السلوك لسياق معين دون تعديل ملف `config.php`.

---

## 13. خلاصة

- **جميع إعدادات الصلاحيات** يجب أن توضع تحت مفتاح `permission` لكل عملية داخل `config.php`.
- **استخدم متغيرات البيئة** لتجنب القيم الثابتة ولتسهيل التبديل بين البيئات (تطوير، اختبار، إنتاج).
- **نطاقات الوصول (scopes)** مثل `company`, `department`, `state`, `children` يتم التعامل معها تلقائياً بواسطة `AccessManager` عند تطبيق `applyAccessScope`.
- **الفلاتر المتقدمة** تمنح تحكماً دقيقاً في الحقول التي يسمح للمستخدمين باستخدام `is_or`, `is_not`, `is_or_null` بها.
- **دوال المبسطة** (`checkByResource`, `getOperationByConfig`, `checkWithFallback`) تجعل استخدام `AccessManager` نظيفاً وموجهاً بالكامل نحو الإعدادات.

بهذا الإطار، يمكنك بناء أي API plugin مع نظام صلاحيات مرن وقابل للتوسع بسهولة.