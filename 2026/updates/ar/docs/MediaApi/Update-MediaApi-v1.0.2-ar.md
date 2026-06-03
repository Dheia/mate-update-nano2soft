## 2026-06-01 - 2026-06-02

**تحديث إضافة `Nano.MediaApi` – الإصدار 1.0.2**

### تحسين فلترة التصنيفات في متحكمات الصور والفيديوهات والملفات لدعم التصنيفات الأم (Parent Categories)

---

### ملخص التحديثات

يقدم الإصدار **1.0.2** من إضافة `Nano.MediaApi` تحسيناً مهماً في آلية فلترة العناصر حسب التصنيف (`categories_id`). سابقاً، كانت الفلترة تقتصر على مطابقة `categories_id` المباشر لكل عنصر (صورة، فيديو، ملف). أما الآن، فتم توسيع المنطق ليشمل أيضاً العناصر التي تنتمي إلى تصنيفات يكون التصنيف الممرر هو **التصنيف الأب (parent_id)** لها، وذلك عبر العلاقات (`album` للصور، `playlist` للفيديوهات، `filelist` للملفات).

**أبرز التغييرات:**
- تعديل دالة `index` في المتحكمات:
  - `Nano\MediaApi\APIControllers\Images`
  - `Nano\MediaApi\APIControllers\Videos`
  - `Nano\MediaApi\APIControllers\Files`
- دعم تمرير `categories_id` كمصفوفة (array) للفلترة بعدة تصنيفات.
- استخدام `orWhereHas` للوصول إلى التصنيفات غير المباشرة عبر العلاقة.
- تحديث التوثيق الخاص بمكتبة الوسائط (Media Library API) ليعكس السلوك الجديد.

---

### أهداف الإصدار

- **تحسين دقة الفلترة**: تمكين المطورين من جلب جميع الصور أو الفيديوهات أو الملفات الموجودة تحت تصنيف معين **وجميع تصنيفاته الفرعية** دون الحاجة إلى استدعاءات متعددة.
- **تبسيط التكامل**: عدم إجبار المستخدم على معرفة الشجرة الكاملة للتصنيفات؛ يكفي تمرير التصنيف الأب.
- **دعم مرن للمصفوفات**: السماح بتمرير عدة تصنيفات (أباء أو أبناء) في طلب واحد.
- **التوافق مع توقعات المستخدمين**: في أنظمة إدارة المحتوى، غالباً ما يُتوقع أن اختيار تصنيف يعرض محتوى جميع التصنيفات الفرعية أيضاً.

---

### الميزات الجديدة والتحسينات

#### 1. توسيع منطق فلترة `categories_id` في `Images`، `Videos`، `Files`

**قبل الإصدار 1.0.2:**
```php
if($categories_id = Input::get('categories_id', false)) {
    $posts = $posts->where('categories_id', $categories_id);
}
```

**بعد التحديث:**
```php
if($categories_id = Input::get('categories_id', false)) {
    $posts = $posts->where(function ($query) use ($categories_id) {
        if(is_array($categories_id)) {
            $query->whereIn('categories_id', $categories_id)
                ->orWhereHas('album', function ($query2) use ($categories_id) {
                    $query2->whereIn('tss_media_categories.parent_id', $categories_id);
                });
        } else {
            $query->where('categories_id', $categories_id)
                ->orWhereHas('album', function ($query2) use ($categories_id) {
                    $query2->where('tss_media_categories.parent_id', $categories_id);
                });
        }
    });
}
```

> **ملاحظة:** تم تطبيق نفس المنطق في متحكم `Videos` باستخدام العلاقة `playlist`، وفي متحكم `Files` باستخدام العلاقة `filelist`.

#### 2. دعم تمرير `categories_id` كمصفوفة

أصبح من الممكن الآن تمرير عدة تصنيفات دفعة واحدة:

```
GET /api/v1/media/images?categories_id[]=3&categories_id[]=5
```

سيؤدي ذلك إلى جلب الصور التي تتبع التصنيف 3 أو 5 **أو** التي تتبع أي ألبوم يكون التصنيف الأب لألبومها هو 3 أو 5.

#### 3. تحديث توثيق API

تم تحديث التوثيق السابق (`Docs-MediaApi-ar.md`) ليشمل شرحاً للسلوك الجديد وإيضاح أن فلترة `categories_id` تشمل الآن **جميع العناصر المرتبطة بالتصنيف الممرر سواء بشكل مباشر أو عبر العلاقات (الألبومات/قوائم التشغيل/حزم الملفات)**.

---

### أمثلة عملية على الاستخدام الجديد

#### 1. جلب جميع الصور تحت تصنيف رئيسي (مع تصنيفاته الفرعية)

افترض أن لديك التصنيف رقم `10` باسم "الطبيعة"، وتحته تصنيفات فرعية "جبال" و"بحار".  
بتمرير `categories_id=10`، ستحصل على جميع الصور التي:
- `categories_id = 10` مباشرة، أو
- تنتمي إلى ألبوم يكون `parent_id` لذلك الألبوم هو `10`.

```bash
curl -X GET "https://yourdomain.com/api/v1/media/images?categories_id=10" \
  -H "Authorization: Bearer <token>"
```

#### 2. جلب الفيديوهات من عدة تصنيفات رئيسية في وقت واحد

```bash
curl -X GET "https://yourdomain.com/api/v1/media/videos?categories_id[]=2&categories_id[]=7" \
  -H "Authorization: Bearer <token>"
```

#### 3. جلب الملفات من تصنيف محدد مع تضمين حزمة الملفات

```bash
curl -X GET "https://yourdomain.com/api/v1/media/files?categories_id=15&include=filelist" \
  -H "Authorization: Bearer <token>"
```

---

### الفوائد والقيمة المضافة

- **تقليل عدد استدعاءات الـ API**: لم يعد المطور بحاجة إلى جلب جميع التصنيفات الفرعية أولاً ثم إجراء استدعاءات منفصلة لكل منها.
- **تناسق أفضل مع منطق واجهة المستخدم**: في معظم تطبيقات إدارة الوسائط، النقر على تصنيف رئيسي يعرض محتوى جميع الأقسام الفرعية.
- **مرونة متزايدة**: دعم المصفوفات يسمح بدمج مجموعات تصنيفات غير مترابطة في طلب واحد.
- **توافق عكسي كامل**: إذا كنت تمرر `categories_id` كقيمة واحدة، فسيظل السلوك القديم (المباشر) صحيحاً، مع إضافة النتائج غير المباشرة (وهو ما يعزز التوافق).

---

### متطلبات الترقية (من 1.0.1 إلى 1.0.2)

1. **تحديث الكود**:
   - استبدال ملفات المتحكمات التالية:
     - `plugins/nano/mediaapi/APIControllers/Images.php`
     - `plugins/nano/mediaapi/APIControllers/Videos.php`
     - `plugins/nano/mediaapi/APIControllers/Files.php`

2. **تحديث التوثيق** (اختياري ولكن موصى به):
   - استبدال ملف `Docs-MediaApi-ar.md` (أو أي ملف توثيق API) بالنسخة المحدثة التي توضح السلوك الجديد للفلترة.

3. **تنفيذ أي أوامر ضرورية** (لا توجد هجرات قاعدة بيانات):
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

4. **اختبار الوظائف**:
   - تأكد من أن طلب `GET /api/v1/media/images?categories_id=X` يعيد الصور من الألبومات التي يكون `parent_id` لها يساوي `X`.
   - اختبر تمرير مصفوفة: `categories_id[]=1&categories_id[]=2`.
   - اختبر بقية النقاط (`videos`, `files`) بنفس الطريقة.

---

### الخاتمة

يمثل الإصدار **1.0.2** من `Nano.MediaApi` خطوة مهمة نحو تحسين تجربة المطورين والمستخدمين النهائيين عند التعامل مع الوسائط المصنفة هرمياً. من خلال دعم التصنيفات الأم والسماح بالمصفوفات، أصبحت الإضافة أكثر ذكاءً ومرونة، مع الحفاظ على التوافق الكامل مع الإصدارات السابقة. نوصي جميع مشاريع `Nano.MediaApi` بالترقية إلى هذا الإصدار للاستفادة من سلوك الفلترة الطبيعي والمتوقع.

---

**الوثائق المرجعية**:
- [توثيق مكتبة الوسائط API (MediaApi)](./docs/MediaApi/Docs-MediaApi-ar.md)
- [توثيق مختصر مكتبة الوسائط API (MediaApi)](./docs/MediaApi/Docs-MediaApi-Short-ar.md)
- [أمثلة عملية شاملة – إضافة MediaApi](./docs/MediaApi/Docs-MediaApi-Examples-ar.md)
- [أمثلة عملية مختصرة – إضافة MediaApi](./docs/MediaApi/Docs-MediaApi-Short-Examples-ar.md)
