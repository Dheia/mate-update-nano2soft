## توثيق متحكم `DynamicReports` – واجهة برمجية للتقارير الديناميكية والمسجلة

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** و **OctoberCMS** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

يوفر متحكم `DynamicReports` واجهة برمجية (API) موحدة للتعامل مع نظام التقارير، سواء كانت تقارير مسجلة مسبقاً (قوالب) أو تقارير ديناميكية يتم بناؤها أثناء الطلب.

### المصادقة والصلاحيات

- جميع نقاط النهاية محمية بواسطة middleware `oauth-users`، مما يعني أن الطلب يجب أن يحتوي على رمز مصادقة صالح (Access Token).
- يتم التحقق من صلاحيات المستخدم تلقائياً عند محاولة الوصول إلى تقرير معين أو جدول معين بناءً على الصلاحيات المسجلة في تعريفات الجداول والتقارير.

---

## نقاط النهاية (Endpoints)

### 1. قائمة التقارير المسجلة المتاحة

**GET** `/api/v1/querybuilder/dynamic-reports`

#### الوصف
يعيد قائمة بجميع التقارير المسجلة مسبقاً والتي يسمح للمستخدم الحالي بالوصول إليها.

#### المدخلات
لا توجد معاملات إضافية.

#### المخرجات
مصفوفة من كائنات التقارير المبسطة (بدون `query_config` الداخلي)، يحتوي كل منها على:
- `report_id`: معرف التقرير.
- `name`: اسم التقرير.
- `description`: وصف التقرير.
- `filters`: تعريف الفلاتر المتاحة.
- `export_options`: خيارات التصدير المسموحة.
- `meta`: بيانات إضافية (اختياري).

#### مثال للطلب
```http
GET /api/v1/querybuilder/dynamic-reports
Authorization: Bearer {token}
```

#### مثال للاستجابة
```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب التقارير المتاحة بنجاح",
    "data": [
        {
            "report_id": "products_with_units",
            "name": "منتجات لديها وحدات",
            "description": "المنتجات التي تحتوي على وحدات (is_units = 1) مع إمكانية التصفية حسب الشركة والفرع.",
            "filters": {
                "companys_id": {"type": "integer", "required": false, "label": "الشركة"},
                "departments_id": {"type": "integer", "required": false, "label": "الفرع"}
            },
            "export_options": {
                "formats": ["csv", "excel", "json"],
                "max_rows": 50000
            },
            "meta": []
        },
        {
            "report_id": "products_without_units",
            "name": "منتجات بدون وحدات",
            "description": "المنتجات التي لا تحتوي على وحدات (is_units = 0)",
            "filters": {
                "companys_id": {"type": "integer", "required": false},
                "departments_id": {"type": "integer", "required": false}
            },
            "export_options": {
                "formats": ["csv", "excel", "json"],
                "max_rows": 50000
            },
            "meta": []
        }
    ]
}
```

---

### 2. عرض تفاصيل تقرير مسجل

**GET** `/api/v1/querybuilder/dynamic-reports/{reportId}`

#### الوصف
يعيد تفاصيل تقرير معين (بما في ذلك `filters`، `export_options`، `meta`).

#### المدخلات
- `reportId` (في المسار): معرف التقرير.

#### المخرجات
كائن التقرير (بدون `query_config` الداخلي).

#### مثال للطلب
```http
GET /api/v1/querybuilder/dynamic-reports/products_with_units
Authorization: Bearer {token}
```

#### مثال للاستجابة
```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب تفاصيل التقرير بنجاح",
    "data": {
        "report_id": "products_with_units",
        "name": "منتجات لديها وحدات",
        "description": "المنتجات التي تحتوي على وحدات (is_units = 1) مع إمكانية التصفية حسب الشركة والفرع.",
        "filters": {
            "companys_id": {"type": "integer", "required": false, "label": "الشركة"},
            "departments_id": {"type": "integer", "required": false, "label": "الفرع"}
        },
        "export_options": {
            "formats": ["csv", "excel", "json"],
            "max_rows": 50000
        },
        "meta": []
    }
}
```

---

### 3. تشغيل تقرير مسجل

**POST** `/api/v1/querybuilder/dynamic-reports/run/{reportId}`

#### الوصف
يقوم بتنفيذ تقرير مسجل بعد تمرير قيم الفلاتر (إن وجدت) وخيارات إضافية (مثل التصفح والتصدير).

#### المدخلات (JSON Body)
- `filters` (اختياري): كائن يحتوي على قيم الفلاتر المطلوبة. يجب أن تطابق المفاتيح أسماء الفلاتر المعرفة في التقرير.
- `options` (اختياري): كائن يحتوي على خيارات إضافية:
  - `page` (عدد صحيح): رقم الصفحة (للبحث المقسّم).
  - `per_page` (عدد صحيح): عدد السجلات في كل صفحة.
  - `limit` (عدد صحيح): عدد السجلات المطلوبة (بديل عن `per_page`).
  - `offset` (عدد صحيح): نقطة البداية (بديل عن `page`).
  - `expose_meta_details` (قيمة منطقية): إذا كانت `true`، يتم إرجاع تفاصيل إضافية في `meta` مثل استعلام SQL والـ joins المستخدمة (يُستخدم للتصحيح). القيمة الافتراضية تتبع `app.debug`.

#### المخرجات
- `data`: مصفوفة من السجلات الناتجة عن التقرير.
- `columns`: معلومات عن الأعمدة المحددة.
- `meta`: بيانات وصفية تشمل وقت التنفيذ وإجمالي الصفوف ومعلومات التصفح (إذا طُلب).

#### مثال للطلب
```http
POST /api/v1/querybuilder/dynamic-reports/run/products_with_units
Authorization: Bearer {token}
Content-Type: application/json

{
    "filters": {
        "companys_id": 1
    },
    "options": {
        "page": 2,
        "per_page": 50
    }
}
```

#### مثال للاستجابة
```json
{
    "code": 200,
    "status": true,
    "message": "تم تنفيذ التقرير بنجاح",
    "data": {
        "success": true,
        "data": [
            {
                "id": 51,
                "name": "منتج 51",
                "price": "250.00",
                "brand_name": null,
                "units_units_id": 3
            },
            ...
        ],
        "columns": [
            {"name": "id", "column_name": "id", "label": "المعرف", "type": "string"},
            {"name": "name", "column_name": "name", "label": "الاسم", "type": "string"},
            {"name": "price", "column_name": "price", "label": "السعر", "type": "string"},
            {"name": "brand_name", "column_name": "brand.name", "label": "العلامة التجارية", "type": "string"},
            {"name": "units_units_id", "column_name": "units.units_id", "label": "معرف الوحدة", "type": "string"}
        ],
        "meta": {
            "total_rows": 50,
            "execution_time_ms": 4.21,
            "pagination": {
                "total": 1240,
                "count": 50,
                "per_page": 50,
                "current_page": 2,
                "total_pages": 25,
                "links": {
                    "previous": "https://example.com/api/v1/querybuilder/dynamic-reports/run/products_with_units?page=1&per_page=50",
                    "next": "https://example.com/api/v1/querybuilder/dynamic-reports/run/products_with_units?page=3&per_page=50"
                }
            }
        }
    }
}
```

---

### 4. تشغيل تقرير ديناميكي (مخصص)

**POST** `/api/v1/querybuilder/dynamic-reports/execute`

#### الوصف
يقوم بتنفيذ تقرير ديناميكي يتم تعريفه بالكامل في الطلب، دون الحاجة لتسجيله مسبقاً.

#### المدخلات (JSON Body)
- `query` (إلزامي): كائن يمثل تكوين الاستعلام، بنفس هيكل `query_config` المستخدم في التقارير المسجلة. يجب أن يحتوي على:
  - `table`: كائن يحتوي على `name` (اسم الجدول الأساسي).
  - `columns`: مصفوفة من الأعمدة المحددة.
  - `conditions` (اختياري): مصفوفة من الشروط.
  - `computed_columns` (اختياري): مصفوفة من الأعمدة المحسوبة.
  - `joins` (اختياري): مصفوفة من الـ joins اليدوية.
  - `groupBy` (اختياري): مصفوفة من التجميعات.
  - `sortBy` (اختياري): مصفوفة من الترتيب.
  - `limit` (اختياري): عدد السجلات.
  - `offset` (اختياري): نقطة البداية.
- `options` (اختياري): نفس خيارات `options` الموضحة في نقطة النهاية السابقة (بما في ذلك `page`، `per_page`، `expose_meta_details`).

#### المخرجات
نفس مخرجات تقرير مسجل.

#### مثال للطلب
```http
POST /api/v1/querybuilder/dynamic-reports/execute
Authorization: Bearer {token}
Content-Type: application/json

{
    "query": {
        "table": {"name": "tss_inventory_products"},
        "columns": [
            {"name": "id", "label": "المعرف"},
            {"name": "name", "label": "الاسم"},
            {"name": "price", "label": "السعر"},
            {"name": "brand.name", "label": "العلامة التجارية"}
        ],
        "conditions": [
            {"field": {"name": "is_active"}, "operator": {"value": "="}, "value": 1},
            {"field": {"name": "companys_id"}, "operator": {"value": "="}, "value": 1}
        ],
        "limit": 20
    },
    "options": {
        "page": 1,
        "per_page": 10
    }
}
```

#### مثال للاستجابة
```json
{
    "code": 200,
    "status": true,
    "message": "تم تنفيذ التقرير بنجاح",
    "data": {
        "success": true,
        "data": [...],
        "columns": [...],
        "meta": {
            "total_rows": 10,
            "execution_time_ms": 3.12,
            "pagination": {
                "total": 45,
                "count": 10,
                "per_page": 10,
                "current_page": 1,
                "total_pages": 5,
                "links": {
                    "next": "https://example.com/api/v1/querybuilder/dynamic-reports/execute?page=2&per_page=10"
                }
            }
        }
    }
}
```

---

### 5. تصدير نتائج تقرير

**POST** `/api/v1/querybuilder/dynamic-reports/export`

#### الوصف
يقوم بتشغيل تقرير (مسجل أو ديناميكي) ثم تصدير النتائج بصيغة محددة (مثل CSV، Excel، JSON).

#### المدخلات (JSON Body)
يجب توفير إما `report_id` (لتقرير مسجل) أو `query` (لتقرير ديناميكي).

- `report_id` (اختياري): معرف التقرير المسجل.
- `query` (اختياري): تكوين الاستعلام (كما في `execute`).
- `filters` (اختياري): قيم الفلاتر (إذا تم استخدام `report_id`).
- `format` (إلزامي): صيغة التصدير المدعومة (csv, excel, json, pdf, xml).
- `export_options` (اختياري): خيارات إضافية للتصدير (مثل `delimiter` لـ CSV، `sheet_name` لـ Excel).

#### المخرجات
كائن يحتوي على معلومات الملف المُصدَّر:
- `success`: true.
- `format`: الصيغة المستخدمة.
- `filename`: اسم الملف.
- `filepath`: المسار الكامل للملف على الخادم.
- `size`: حجم الملف بالبايت.
- `url`: رابط تحميل الملف.
- `download_url`: رابط تحميل بديل (إذا تم تعريف route).

#### مثال للطلب (تصدير تقرير مسجل)
```http
POST /api/v1/querybuilder/dynamic-reports/export
Authorization: Bearer {token}
Content-Type: application/json

{
    "report_id": "products_with_units",
    "filters": {
        "companys_id": 1
    },
    "format": "excel",
    "export_options": {
        "sheet_name": "منتجات بوحدات"
    }
}
```

#### مثال للاستجابة
```json
{
    "code": 200,
    "status": true,
    "message": "تم تصدير التقرير بنجاح",
    "data": {
        "success": true,
        "format": "excel",
        "filename": "report-products_with_units-2025-04-13-153045.xlsx",
        "filepath": "/var/www/html/storage/app/exports/report-products_with_units-2025-04-13-153045.xlsx",
        "size": 25600,
        "rows": 1240,
        "url": "https://example.com/v1/reports/export-download/report-products_with_units-2025-04-13-153045.xlsx",
        "download_url": "https://example.com/reports/download/report-products_with_units-2025-04-13-153045.xlsx"
    }
}
```

---

### 6. الحصول على مخطط جدول (الأعمدة المتاحة)

**GET** `/api/v1/querybuilder/dynamic-reports/schema/{tableName}`

#### الوصف
يعيد قائمة بجميع الأعمدة القابلة للاستعلام من جدول معين (بما في ذلك أعمدة العلاقات auto-join).

#### المدخلات
- `tableName` (في المسار): اسم الجدول (مثل `tss_inventory_products`).

#### المخرجات
مصفوفة من الأعمدة، كل عمود يحتوي على:
- `name`: اسم العمود الفعلي (مع المسار إذا كان من علاقة).
- `label`: تسمية عرض.
- `type`: نوع البيانات.
- `auto_join_path`: مسار العلاقة (إن وجد).

#### مثال للطلب
```http
GET /api/v1/querybuilder/dynamic-reports/schema/tss_inventory_products
Authorization: Bearer {token}
```

#### مثال للاستجابة
```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب مخطط الجدول بنجاح",
    "data": [
        {"name": "id", "label": "Id", "type": "integer", "auto_join_path": null},
        {"name": "name", "label": "Name", "type": "string", "auto_join_path": null},
        {"name": "price", "label": "Price", "type": "decimal", "auto_join_path": null},
        {"name": "brand.name", "label": "Brand Name", "type": "string", "auto_join_path": "brand"},
        {"name": "currency.name", "label": "Currency Name", "type": "string", "auto_join_path": "currency"},
        {"name": "categories.name", "label": "Categories Name", "type": "string", "auto_join_path": "categories"}
    ]
}
```

---

### 7. التحقق من صحة تكوين تقرير

**GET** أو **POST** `/api/v1/querybuilder/dynamic-reports/validate`

#### الوصف
يقوم بالتحقق من صحة تكوين تقرير (سواء كان ديناميكياً أو جزءاً من تقرير مسجل) دون تنفيذه.

#### المدخلات
يمكن تمرير تكوين الاستعلام كـ JSON في الجسم (لطلب POST) أو كمعامل `query` في سلسلة الاستعلام (لطلب GET).

- `query` (إلزامي): كائن تكوين الاستعلام.

#### المخرجات
- `valid`: true إذا كان التكوين صالحاً.
- `errors`: مصفوفة من الأخطاء (إن وجدت).
- `warnings`: مصفوفة من التحذيرات.
- `summary`: ملخص يتضمن التعقيد والأداء المتوقع.

#### مثال للطلب (POST)
```http
POST /api/v1/querybuilder/dynamic-reports/validate
Authorization: Bearer {token}
Content-Type: application/json

{
    "query": {
        "table": {"name": "tss_inventory_products"},
        "columns": [{"name": "id"}, {"name": "name"}, {"name": "brand.name"}],
        "conditions": [
            {"field": {"name": "is_active"}, "operator": {"value": "="}, "value": 1}
        ]
    }
}
```

#### مثال للاستجابة
```json
{
    "code": 200,
    "status": true,
    "message": "تكوين التقرير صحيح",
    "data": {
        "valid": true,
        "errors": [],
        "warnings": [],
        "summary": {
            "complexity": "low",
            "total_columns": 3,
            "total_joins": 1,
            "total_conditions": 1,
            "has_grouping": false,
            "has_sorting": false,
            "has_limit": false,
            "estimated_performance": "fast"
        }
    }
}
```

---

### 8. تقدير تكلفة استعلام

**GET** أو **POST** `/api/v1/querybuilder/dynamic-reports/estimate`

#### الوصف
يقوم بتقدير تكلفة تنفيذ استعلام (تعقيد، أداء متوقع) دون تنفيذه فعلياً.

#### المدخلات
نفس مدخلات `validate`.

#### المخرجات
- `success`: true إذا كان التكوين صالحاً.
- `complexity`: درجة التعقيد (low, medium, high).
- `estimated_performance`: الأداء المتوقع (fast, moderate, slow).
- `total_columns`, `total_joins`, `total_conditions`: إحصائيات.
- `warnings`: تحذيرات.
- `errors`: أخطاء (إن وجدت).

#### مثال للطلب (POST)
```http
POST /api/v1/querybuilder/dynamic-reports/estimate
Authorization: Bearer {token}
Content-Type: application/json

{
    "query": {
        "table": {"name": "tss_inventory_products"},
        "columns": [{"name": "id"}, {"name": "name"}],
        "joins": [
            {"table": "tss_inventory_companys_products", "type": "left", "local_key": "companys_products_id", "foreign_key": "id"}
        ]
    }
}
```

#### مثال للاستجابة
```json
{
    "code": 200,
    "status": true,
    "message": "تم تقدير تكلفة الاستعلام بنجاح",
    "data": {
        "success": true,
        "complexity": "low",
        "estimated_performance": "fast",
        "total_columns": 2,
        "total_joins": 1,
        "total_conditions": 0,
        "warnings": [],
        "errors": []
    }
}
```

---

## أمثلة متكاملة

### مثال 1: بناء تقرير ديناميكي بسيط وجلب الصفحة الأولى

**الطلب**:
```http
POST /api/v1/querybuilder/dynamic-reports/execute
Authorization: Bearer {token}
Content-Type: application/json

{
    "query": {
        "table": {"name": "tss_inventory_products"},
        "columns": [
            {"name": "id", "label": "المعرف"},
            {"name": "name", "label": "الاسم"},
            {"name": "price", "label": "السعر"}
        ],
        "limit": 10
    }
}
```

**الاستجابة**:
```json
{
    "code": 200,
    "status": true,
    "message": "تم تنفيذ التقرير بنجاح",
    "data": {
        "success": true,
        "data": [
            {"id": 1, "name": "منتج 1", "price": "100.00"},
            {"id": 2, "name": "منتج 2", "price": "150.00"}
        ],
        "columns": [
            {"name": "id", "column_name": "id", "label": "المعرف", "type": "string"},
            {"name": "name", "column_name": "name", "label": "الاسم", "type": "string"},
            {"name": "price", "column_name": "price", "label": "السعر", "type": "string"}
        ],
        "meta": {
            "total_rows": 2,
            "execution_time_ms": 2.34,
            "pagination": {
                "total": 500,
                "count": 2,
                "per_page": 10,
                "current_page": 1,
                "total_pages": 50
            }
        }
    }
}
```

### مثال 2: استخدام تقرير مسجل مع فلتر وطلب الصفحة الثانية

**الطلب**:
```http
POST /api/v1/querybuilder/dynamic-reports/run/products_with_units
Authorization: Bearer {token}
Content-Type: application/json

{
    "filters": {
        "companys_id": 2
    },
    "options": {
        "page": 2,
        "per_page": 25
    }
}
```

**الاستجابة** (مشابهة للمثال السابق لكن مع تغيير البيانات).

### مثال 3: تصدير تقرير مسجل إلى CSV

**الطلب**:
```http
POST /api/v1/querybuilder/dynamic-reports/export
Authorization: Bearer {token}
Content-Type: application/json

{
    "report_id": "products_without_units",
    "filters": {
        "companys_id": 1,
        "departments_id": 2
    },
    "format": "csv",
    "export_options": {
        "delimiter": ";",
        "include_bom": true
    }
}
```

**الاستجابة**:
```json
{
    "code": 200,
    "status": true,
    "message": "تم تصدير التقرير بنجاح",
    "data": {
        "success": true,
        "format": "csv",
        "filename": "report-products_without_units-2025-04-13-160215.csv",
        "filepath": "...",
        "size": 15200,
        "rows": 320,
        "url": "...",
        "download_url": "..."
    }
}
```

---

## ملاحظات هامة

- في بيئة الإنتاج (`app.debug = false`)، يتم إخفاء تفاصيل استعلام SQL والمعلومات الحساسة من `meta` ومن رسائل الخطأ.
- في حالة حدوث خطأ، يتم إرجاع رسالة مناسبة مع رمز الخطأ ومعرف فريد لتتبعه.
- يمكن التحكم في عرض التفاصيل الإضافية في `meta` عبر خيار `expose_meta_details` في `options`، والذي يُستخدم بشكل أساسي في بيئة التطوير.
- جميع التواريخ والأرقام ترجع بتنسيق مناسب (التواريخ بصيغة ISO، الأرقام العشرية كسلاسل نصية للحفاظ على الدقة).

بهذا نكون قد غطينا جميع جوانب متحكم `DynamicReports` مع أمثلة عملية توضح الاستخدام المتوقع.