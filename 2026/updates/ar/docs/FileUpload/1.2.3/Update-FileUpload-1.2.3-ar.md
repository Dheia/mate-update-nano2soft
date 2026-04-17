## 2026-04-15 – 2026-04-16

**تحديث إضافة `Nano.FileUpload` – الإصدار 1.2.3**

### إضافة نقاط نهاية API جديدة للاستعلام عن الموديولات والصلاحيات والتحقق من المفاتيح المؤقتة

---

### ملخص التحديثات

في الإصدار **1.2.3**، تم إثراء واجهة برمجة التطبيقات (API) للإضافة بعدة نقاط نهاية جديدة تتيح للمطورين والأنظمة الخارجية:

- **الاستعلام عن الموديولات المسجلة** وحقولها مع مراعاة صلاحيات المستخدم.
- **جلب الإعدادات التفصيلية** لموديول معين أو حقل معين.
- **جلب قيود الحقل** (مثل الحجم الأقصى والأنواع المسموحة).
- **جلب خيارات المعالجة المتقدمة** (قرص التخزين، التحجيم التلقائي، العلامة المائية).
- **التحقق من الصلاحيات** على المستوى العالمي، مستوى الموديول، ومستوى الحقل.
- **التحقق المتكامل من الصلاحية** باستخدام دالة `validate()`.
- **التحقق من صحة المفتاح المؤقت ومطابقته** عبر واجهة API (باستخدام دالة `validateAndMatchTempKey` من التريت).

جميع نقاط النهاية الجديدة محمية بـ `oauth-users` وتتبع هيكل الاستجابة الموحد (`code`, `status`, `message`, `data`, ...).

---

### أهداف الإصدار

- **تمكين المطورين من استكشاف الإعدادات المسجلة** في `FileUploadRegistry` دون الحاجة للوصول المباشر إلى الكود أو قاعدة البيانات.
- **تسهيل دمج الواجهات الأمامية** (مثل تطبيقات React أو Vue) من خلال توفير معلومات ديناميكية حول الحقول المسموحة والقيود.
- **التحقق من الصلاحيات قبل محاولة الرفع** لتجنب أخطاء الرفض وتحسين تجربة المستخدم.
- **توفير نقطة نهاية للتحقق من صحة المفاتيح المؤقتة** مما يسمح للواجهات الأمامية بالتأكد من صلاحية المفتاح قبل استخدامه.
- **توحيد طريقة الاستعلام** عن معلومات النظام عبر API بدلاً من الاعتماد على أدوات داخلية.

---

### نقاط النهاية الجديدة

#### 1. جلب جميع الموديولات المسجلة (مع تصفية حسب الصلاحية)

- **المسار:** `GET /api/v1/fileupload/models`
- **الوصف:** يعيد قائمة الموديولات المسجلة التي يملك المستخدم الحالي صلاحية `view` عليها، مع معلومات مختصرة عن حقولها (الاسم، النوع، هل هو متعدد).
- **الاستجابة:** مصفوفة من الكائنات تحتوي على `class`, `label`, `enabled`, `allowed_user_types`, `fields`.

#### 2. جلب إعدادات موديول معين

- **المسار:** `GET /api/v1/fileupload/models/{modelClass}`
- **الوصف:** يعيد الإعدادات الكاملة لموديول معين (بعد التحقق من وجوده وصلاحية `view`).
- **الاستجابة:** تحتوي على `enabled`, `label`, `allowed_user_types`, `disabled_operations`, و `fields` (مع معلومات مختصرة لكل حقل).

#### 3. جلب إعدادات حقل معين

- **المسار:** `GET /api/v1/fileupload/models/{modelClass}/fields/{field}`
- **الوصف:** يعيد إعدادات حقل معين (مثل النوع، الحجم الأقصى، الأنواع المسموحة).
- **الاستجابة:** تشمل `type`, `label`, `multiple`, `required`, `max_filesize`, `allowed_types`, `use_caption`, `disabled_operations`.

#### 4. جلب قيود الحقل

- **المسار:** `GET /api/v1/fileupload/models/{modelClass}/fields/{field}/constraints`
- **الوصف:** يعيد قيود الحقل المستخدمة في عملية التحقق من صحة الملف (`max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`).
- **الفائدة:** يمكن للواجهات الأمامية تطبيق نفس القيود قبل رفع الملف.

#### 5. جلب خيارات المعالجة المتقدمة لحقل معين

- **المسار:** `GET /api/v1/fileupload/processing-options/{modelClass}/{field}`
- **الوصف:** يعيد خيارات المعالجة الخاصة بالحقل: `storage_disk`, `auto_resize`, `resize_options`, `auto_watermark`, `watermark_options`.
- **الفائدة:** يستخدم بشكل أساسي من قبل الواجهات المتقدمة التي تحتاج إلى معرفة إعدادات التحويل التلقائي.

#### 6. التحقق من تفعيل عملية معينة عالمياً

- **المسار:** `GET /api/v1/fileupload/permissions/global/{operation}`
- **المعاملات:** `operation` يمكن أن يكون `add`, `edit`, `delete`, `view`.
- **الاستجابة:** تعيد `allowed` (true/false) و `disabled_globally` (عكس `allowed`).

#### 7. التحقق من صلاحية عملية على مستوى موديول معين

- **المسار:** `GET /api/v1/fileupload/permissions/model/{modelClass}/{operation}`
- **الاستجابة:** تعيد `model_operation_enabled`, `user_type_allowed`, `can_proceed` (مجموع الشرطين).

#### 8. التحقق من صلاحية عملية على مستوى حقل معين

- **المسار:** `GET /api/v1/fileupload/permissions/field/{modelClass}/{field}/{operation}`
- **الاستجابة:** تعيد `field_operation_enabled`, `user_has_full_permission`, `can_proceed`.

#### 9. التحقق المتكامل من الصلاحية (باستخدام `validate`)

- **المسار:** `POST /api/v1/fileupload/permissions/check`
- **البيانات (JSON):**
  ```json
  {
    "model_class": "Nano\\Shop\\Models\\Product",
    "operation": "edit",
    "field": "image"
  }
  ```
- **الاستجابة:** تعيد `allowed` (true/false) ورسالة توضيحية.
- **السلوك:** يستخدم دالة `$this->registry->validate()` التي تتحقق من المستوى العالمي، الموديول، الحقل، نوع المستخدم، والصلاحيات المحددة، وتعيد النتيجة دون رمي استثناء (يتم التقاط الاستثناء وتحويله إلى استجابة).

#### 10. التحقق من صحة مفتاح مؤقت ومطابقته

- **المسار:** `POST /api/v1/fileupload/temp-key/validate`
- **البيانات (JSON):**
  ```json
  {
    "temp_key": "tmp_xxxx:yyyy",
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "strict_mode": true,
    "allow_expired_key": false,
    "expiry_grace_period": 300
  }
  ```
- **الوصف:** يستدعي دالة `validateAndMatchTempKey` من التريت `HasFileUploadsMatchTempKey` ويعيد النتيجة بشكل JSON.
- **الاستجابة:** في حالة النجاح تعيد `valid: true` و `key_data` و `matched_data`، وفي حالة الفشل تعيد رسالة الخطأ مع `error_code`.

---

### أمثلة على الاستخدام

#### جلب الموديولات المتاحة للمستخدم الحالي

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/models" \
  -H "Authorization: Bearer <token>"
```

#### جلب إعدادات حقل `image` في موديول `Product`

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/models/Nano%5CShop%5CModels%5CProduct/fields/image" \
  -H "Authorization: Bearer <token>"
```

#### التحقق من صلاحية `edit` على حقل معين

```bash
curl -X GET "https://yourdomain.com/api/v1/fileupload/permissions/field/Nano%5CShop%5CModels%5CProduct/image/edit" \
  -H "Authorization: Bearer <token>"
```

#### التحقق من صحة مفتاح مؤقت

```bash
curl -X POST "https://yourdomain.com/api/v1/fileupload/temp-key/validate" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "temp_key": "tmp_eyJtb2RlbENsYXNzIjoiTmFub1xTaG9wXE1vZGVsc1xQcm9kdWN0Iiw...",
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "strict_mode": true
  }'
```

---

### الفوائد والقيمة المضافة

- **تمكين الواجهات الأمامية**: يمكن الآن للتطبيقات الأمامية (مثل Single Page Applications) معرفة ديناميكياً الحقول المتاحة والقيود والصلاحيات، مما يسمح ببناء نماذج رفع ملفات تكيفية.
- **تقليل الاعتماد على الخادم**: يمكن للعميل التحقق من صلاحياته قبل محاولة الرفع، مما يقلل من طلبات الرفض وتحسين تجربة المستخدم.
- **توثيق حي للإعدادات**: المطورون يمكنهم استكشاف إعدادات الإضافة عبر API بدلاً من البحث في الكود.
- **أمان إضافي**: نقاط نهاية الصلاحيات لا تعرض معلومات حساسة (مثل مفاتيح التخزين) وتخضع لنفس صلاحيات المستخدم.
- **تكامل سهل مع أدوات خارجية**: يمكن استخدام هذه النقاط في أنظمة CI/CD أو أدوات إدارة المحتوى.

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال `FileUploadController.php` بالنسخة التي تحتوي على الدوال الجديدة.
   - تحديث `routes.php` بإضافة المسارات الجديدة.

2. **لا توجد هجرات جديدة**:
   - لا يتطلب هذا الإصدار تغييرات في قاعدة البيانات.

3. **لا توجد تغييرات في الإعدادات**:
   - يظل ملف `config.php` كما هو.

4. **صلاحيات API**:
   - جميع النقاط الجديدة محمية بـ `oauth-users`، لذا يجب أن يمتلك المستخدم توكن صالح للوصول إليها.

5. **اختبار التوافق**:
   - يُنصح بتشغيل اختبارات `FileUploadPlusTest` للتأكد من أن الإصدار الجديد لا يؤثر على الوظائف الحالية.
   - يمكن اختبار النقاط الجديدة باستخدام أدوات مثل Postman أو cURL.

---

### الخاتمة

يمثل الإصدار **1.2.3** نقلة نوعية في إمكانية الوصول إلى معلومات نظام رفع الملفات عبر API. من خلال إضافة نقاط نهاية للاستعلام عن الموديولات والحقول والصلاحيات والتحقق من المفاتيح المؤقتة، أصبحت الإضافة أكثر انفتاحاً وتكاملاً مع الأنظمة الخارجية والواجهات الحديثة. هذه الميزات تجعل من السهل بناء تطبيقات غنية تعتمد على `Nano.FileUpload` كخلفية لإدارة الملفات، مع الحفاظ على أعلى مستويات الأمان والتحكم.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)
