# **TagManager - وثائق كاملة**

## **1. نظرة عامة**

`TagManager` هو كلاس لإدارة تكاملات `TagsModel` في نظام Nano2Soft. يعمل بنظام Singleton ويوفر واجهة مركزية للتحكم بكيفية تفاعل النماذج المختلفة مع نظام الوسوم (Tags).

## **2. التكاملات المتاحة**

| الثابت | الوصف | الوظيفة |
|--------|--------|----------|
| `INTEGRATION_BEHAVIOR` | حقن السلوك | إضافة `TagsModel` behavior للنموذج |
| `INTEGRATION_FORM_FIELD` | حقن حقل النموذج | إضافة حقل tags في واجهة Backend |
| `INTEGRATION_LIST_COLUMN` | حقن عمود القائمة | إضافة عمود tags في قوائم Backend |
| `INTEGRATION_LIST_FILTER` | حقن فلتر القائمة | إضافة فلتر tags في قوائم Backend |
| `INTEGRATION_QUERY` | حقن الاستعلامات | دعم تصفية tags في API و Reports |
| `INTEGRATION_TRANSFORMER` | حقن المحولات | دمج tags في API Transformers |

## **3. التكوين الأساسي**

### **هيكل تكوين النموذج**

```php
$config = [
    // 1. التحكم الأساسي
    'enabled' => true, // تفعيل/تعطيل التكامل للنموذج
    
    // 2. التحكم في أنواع التكامل
    'integrations' => [
        TagManager::INTEGRATION_BEHAVIOR => true,
        TagManager::INTEGRATION_FORM_FIELD => true,
        TagManager::INTEGRATION_LIST_COLUMN => true,
        TagManager::INTEGRATION_LIST_FILTER => true,
        TagManager::INTEGRATION_QUERY => true,
        TagManager::INTEGRATION_TRANSFORMER => true,
    ],
    
    // 3. الإشارات المرجعية
    'controller' => \App\Controllers\MyController::class, // مطلوب للتكاملات الأمامية
    'transformer' => \App\Transformers\MyTransformer::class, // اختياري للتكامل مع Transformers
    
    // 4. تكوينات نوعية
    'query_config' => [...],
    'transformer_config' => [...],
    'form_config' => [...],
    'list_config' => [...],
    'filter_config' => [...],
];
```

## **4. الطرق الأساسية**

### **4.1 التسجيل**

#### **طريقة 1: التسجيل الجماعي (في Plugin.php)**

```php
// في Plugin.php
public function registerTagIntegrations()
{
    return [
        \App\Models\Product::class => [
            'controller' => \App\Controllers\Products::class,
            'transformer' => \App\Transformers\ProductTransformer::class,
            'integrations' => [
                TagManager::INTEGRATION_BEHAVIOR => true,
                TagManager::INTEGRATION_FORM_FIELD => true,
                TagManager::INTEGRATION_QUERY => true,
            ],
        ],
    ];
}
```

#### **طريقة 2: التسجيل الفردي**

```php
$tagManager = TagManager::instance();

// تسجيل نموذج مع تكوين كامل
$tagManager->registerModel(
    \App\Models\Product::class,
    [
        'controller' => \App\Controllers\Products::class,
        'form_config' => [
            'tab' => 'app::lang.products.tab.tags',
        ],
    ]
);

// التسجيل السريع (مع إعدادات افتراضية)
$tagManager->quickRegister(
    \App\Models\Product::class,
    \App\Controllers\Products::class,
    [
        'form_tab' => 'app::lang.products.tab.basic',
        'with_column' => false, // لا نريد عمود في القائمة
        'with_filter' => true, // نريد فلتر
    ]
);
```

### **4.2 التحكم في التكاملات**

```php
$tagManager = TagManager::instance();

// تفعيل تكوين معين
$tagManager->enableIntegration(
    \App\Models\Product::class,
    TagManager::INTEGRATION_LIST_FILTER
);

// تعطيل تكوين معين
$tagManager->disableIntegration(
    \App\Models\Product::class,
    TagManager::INTEGRATION_LIST_COLUMN
);

// التحقق من حالة التكامل
if ($tagManager->isIntegrationEnabled(
    \App\Models\Product::class,
    TagManager::INTEGRATION_FORM_FIELD
)) {
    // الحقل مفعل
}

// تحديث تكوين موجود
$tagManager->updateModelConfig(
    \App\Models\Product::class,
    [
        'form_config' => [
            'cssClass' => 'col-sm-12', // تغيير عرض الحقل
        ],
    ]
);
```

### **4.3 التطبيق اليدوي**

```php
// تطبيق جميع التكاملات لنموذج معين
TagManager::instance()->applyIntegrationsForModel(\App\Models\Product::class);

// تطبيق تكوين معين فقط
TagManager::instance()->applyBehaviorIntegration(\App\Models\Product::class);
TagManager::instance()->applyFormFieldIntegration(\App\Models\Product::class);
TagManager::instance()->applyQueryIntegration(\App\Models\Product::class);

// تطبيق جميع التكاملات لجميع النماذج المسجلة
TagManager::instance()->applyAllIntegrations();
```

## **5. أمثلة متقدمة**

### **المثال 1: منتج مع تكامل كامل**

```php
// في Plugin.php
public function registerTagIntegrations()
{
    return [
        \App\Models\Product::class => [
            'controller' => \App\Controllers\Products::class,
            'transformer' => \App\Transformers\ProductTransformer::class,
            
            // التحكم في التكاملات
            'integrations' => [
                TagManager::INTEGRATION_BEHAVIOR => true,
                TagManager::INTEGRATION_FORM_FIELD => true,
                TagManager::INTEGRATION_LIST_COLUMN => true,
                TagManager::INTEGRATION_LIST_FILTER => true,
                TagManager::INTEGRATION_QUERY => true,
                TagManager::INTEGRATION_TRANSFORMER => true,
            ],
            
            // تكوين الحقل في النموذج
            'form_config' => [
                'field_name' => 'product_tags',
                'label' => 'app::lang.products.tags',
                'tab' => 'app::lang.products.tab.additional',
                'type' => 'nanotaglist',
                'cssClass' => 'col-sm-12',
                'customTags' => false, // منع إنشاء tags جديدة
            ],
            
            // تكوين الاستعلامات
            'query_config' => [
                'param_keys' => ['tags', 'product_tags', 'filter_tags'],
                'wildcard_values' => ['*', 'all', 'any'],
                'scope_method' => 'whereHasProductTags',
            ],
            
            // تكوين المحول
            'transformer_config' => [
                'include_relation' => 'product_tags',
                'include_method' => 'includeProductTags',
            ],
        ],
    ];
}
```

### **المثال 2: المستخدمين (تكامل محدود)**

```php
// في Plugin.php
public function registerTagIntegrations()
{
    return [
        \RainLab\User\Models\User::class => [
            'controller' => \RainLab\User\Controllers\Users::class,
            
            // سلوك فقط + فلتر استعلامات
            'integrations' => [
                TagManager::INTEGRATION_BEHAVIOR => true,
                TagManager::INTEGRATION_FORM_FIELD => false, // لا حقل في النموذج
                TagManager::INTEGRATION_LIST_COLUMN => false, // لا عمود في القائمة
                TagManager::INTEGRATION_LIST_FILTER => false, // لا فلتر في القائمة
                TagManager::INTEGRATION_QUERY => true, // فلتر في API
                TagManager::INTEGRATION_TRANSFORMER => true, // تضمين في API
            ],
            
            // تكوين خاص للاستعلامات
            'query_config' => [
                'param_keys' => ['user_tags', 'user_tags_id'],
                'scope_method' => 'whereHasUserTags',
            ],
        ],
    ];
}
```

### **المثال 3: منشورات المدونة (تكامل مشروط)**

```php
// في Plugin.php
public function registerTagIntegrations()
{
    return [
        \RainLab\Blog\Models\Post::class => [
            'controller' => \RainLab\Blog\Controllers\Posts::class,
            
            // تكامل مشروط بحقل آخر
            'form_config' => [
                'field_name' => 'blog_tags',
                'tab' => 'rainlab.blog::lang.post.tab.categories',
                'trigger' => [
                    'action' => 'show',
                    'field' => 'enable_tags',
                    'condition' => 'checked',
                ],
            ],
            
            // فلتر مشروط
            'filter_config' => [
                'scope_name' => 'blog_tags_id',
                'label' => 'rainlab.blog::lang.post.tags',
                'type' => 'group',
                'modelClass' => 'Nano\Tags\Models\Tag',
                'options' => "listEnabled",
                'scope' => "whereHasBlogTags",
                'conditions' => "enable_tags = 1", // شرط إضافي
            ],
        ],
    ];
}
```

### **المثال 4: التسجيل الديناميكي أثناء التشغيل**

```php
// في أي مكان في التطبيق
public function boot()
{
    // التسجيل الديناميكي بناءً على إعدادات النظام
    if (Config::get('app.enable_product_tags')) {
        $this->registerProductTags();
    }
}

protected function registerProductTags()
{
    $tagManager = TagManager::instance();
    
    $config = [
        'controller' => \App\Controllers\Products::class,
        'integrations' => [
            TagManager::INTEGRATION_BEHAVIOR => true,
            TagManager::INTEGRATION_FORM_FIELD => true,
            TagManager::INTEGRATION_QUERY => Config::get('app.enable_tag_filtering'),
        ],
    ];
    
    // تكوين مختلف لكل بيئة
    if (App::environment('production')) {
        $config['form_config']['customTags'] = false; // منع إنشاء tags جديدة في Production
    }
    
    $tagManager->registerModel(\App\Models\Product::class, $config);
}
```

### **المثال 5: تعديل تكوينات حالية**

```php
// في Service Provider
public function boot()
{
    // تعديل تكوين موجود من Plugin آخر
    $tagManager = TagManager::instance();
    
    $currentConfig = $tagManager->getModelConfig(\Tss\Inventory\Models\Product::class);
    
    if ($currentConfig) {
        // إضافة CSS class إضافية
        $tagManager->updateModelConfig(
            \Tss\Inventory\Models\Product::class,
            [
                'form_config' => [
                    'cssClass' => $currentConfig['form_config']['cssClass'] . ' custom-tags-field',
                ],
            ]
        );
        
        // تعطيل الفلتر في بيئة معينة
        if (App::environment('testing')) {
            $tagManager->disableIntegration(
                \Tss\Inventory\Models\Product::class,
                TagManager::INTEGRATION_LIST_FILTER
            );
        }
    }
}
```

## **6. تكوينات Query المتقدمة**

### **دعم تصفية متعددة**

```php
'query_config' => [
    'param_keys' => ['tags', 'tags_id', 'tag_ids', 'filter[tags]'],
    'wildcard_values' => ['*', 'all', 'any', 'none'],
    'scope_method' => 'whereHasTags',
    
    // معالجة خاصة للقيم
    'value_processors' => [
        'none' => function($query) {
            return $query->doesntHave('tags');
        },
        'any' => function($query) {
            return $query->has('tags');
        },
    ],
    
    // دعم AND/OR منطقي
    'logic_operator' => 'AND', // أو 'OR'
    
    // معالجة متعددة القيم
    'multiple_value_handler' => 'array', // array, comma, json
],
```

### **تكوين Transformer متقدم**

```php
'transformer_config' => [
    'include_relation' => 'tags',
    'include_method' => 'includeTags',
    'default_include' => true, // تضمين افتراضي
    'eager_load' => true, // تحميل مسبق
    'transformer_class' => \Nano\Tags\Transformers\TagTransformer::class,
    
    // حقول إضافية في Transformer
    'additional_fields' => [
        'tags_count' => function($model) {
            return $model->tags()->count();
        },
        'primary_tag' => function($model) {
            return $model->tags()->orderBy('sort_order')->first();
        },
    ],
],
```

## **7. الأحداث (Events) المخصصة**

يمكنك الاستماع لأحداث TagManager:

```php
// قبل تطبيق التكاملات
Event::listen('nano.tags.beforeApplyIntegrations', function($modelClass, $config) {
    // تعديل التكوين قبل التطبيق
});

// بعد تطبيق التكاملات
Event::listen('nano.tags.afterApplyIntegrations', function($modelClass, $config) {
    // تنفيذ إجراءات إضافية
});

// عند تسجيل نموذج جديد
Event::listen('nano.tags.modelRegistered', function($modelClass, $config) {
    Log::info("Model {$modelClass} registered with tags");
});
```

## **8. أفضل الممارسات**

### **8.1 التنظيم في Plugins كبيرة**

```php
// في Plugin كبير مع عدة نماذج
public function registerTagIntegrations()
{
    $integrations = [];
    
    // 1. جمع كل التكاملات في مكان واحد
    $integrations = array_merge($integrations, $this->getProductTagIntegrations());
    $integrations = array_merge($integrations, $this->getUserTagIntegrations());
    $integrations = array_merge($integrations, $this->getOrderTagIntegrations());
    
    return $integrations;
}

protected function getProductTagIntegrations()
{
    $configs = [];
    
    if (class_exists(\App\Models\Product::class)) {
        $configs[\App\Models\Product::class] = [
            'controller' => \App\Controllers\Products::class,
            'form_config' => [
                'tab' => 'app::lang.products.tab.tags',
            ],
        ];
    }
    
    if (class_exists(\App\Models\ProductVariant::class)) {
        $configs[\App\Models\ProductVariant::class] = [
            'controller' => \App\Controllers\ProductVariants::class,
            'integrations' => [
                TagManager::INTEGRATION_BEHAVIOR => true,
                TagManager::INTEGRATION_FORM_FIELD => true,
                // ... باقي التكاملات
            ],
        ];
    }
    
    return $configs;
}
```

### **8.2 التكوين الشرطي**

```php
// بناءً على إعدادات المستخدم
public function registerTagIntegrations()
{
    $integrations = [];
    
    $enableAdvancedTags = Config::get('app.enable_advanced_tags', false);
    
    $integrations[\App\Models\Product::class] = [
        'controller' => \App\Controllers\Products::class,
        'integrations' => [
            TagManager::INTEGRATION_BEHAVIOR => true,
            TagManager::INTEGRATION_FORM_FIELD => true,
            TagManager::INTEGRATION_QUERY => true,
            TagManager::INTEGRATION_TRANSFORMER => $enableAdvancedTags,
        ],
        'form_config' => [
            'type' => $enableAdvancedTags ? 'nanotaglist' : 'taglist',
            'customTags' => $enableAdvancedTags,
        ],
    ];
    
    return $integrations;
}
```

### **8.3 الترحيل من النظام القديم**

```php
// دالة مساعدة للترحيل التدريجي
protected function migrateOldTagSystem()
{
    $tagManager = TagManager::instance();
    
    // قائمة النماذج القديمة
    $oldModels = [
        'product' => [
            'model' => \Tss\Inventory\Models\Product::class,
            'controller' => \Tss\Inventory\Controllers\Products::class,
        ],
        // ... نماذج أخرى
    ];
    
    foreach ($oldModels as $key => $data) {
        if (class_exists($data['model'])) {
            // التسجيل بالنظام الجديد
            $tagManager->registerModel($data['model'], [
                'controller' => $data['controller'],
                'form_config' => [
                    'field_name' => "{$key}_tags", // للحفاظ على التوافق
                ],
            ]);
            
            // تعطيل النظام القديم لهذا النموذج
            $this->disableOldTagIntegration($key);
        }
    }
}
```

## **9. استكشاف الأخطاء**

### **تمكين الـ Debugging**

```php
// في AppServiceProvider أو أي مكان
if (Config::get('app.debug_tags', false)) {
    // تسجيل كل عمليات TagManager
    Event::listen('nano.tags.*', function($event, $data) {
        Log::debug('TagManager Event: ' . $event, $data);
    });
    
    // التحقق من تكوينات النماذج
    $models = TagManager::instance()->getRegisteredModels();
    Log::debug('Registered Tag Models:', array_keys($models));
}
```

### **فحص تكوين نموذج**

```php
// في أي مكان
public function checkTagConfig($modelClass)
{
    $tagManager = TagManager::instance();
    
    if ($tagManager->isModelRegistered($modelClass)) {
        $config = $tagManager->getModelConfig($modelClass);
        
        // التحقق من التكاملات المفعلة
        $enabledIntegrations = array_filter($config['integrations']);
        $enabledNames = array_keys($enabledIntegrations);
        
        return [
            'model' => $modelClass,
            'enabled_integrations' => $enabledNames,
            'has_controller' => !empty($config['controller']),
            'has_transformer' => !empty($config['transformer']),
        ];
    }
    
    return null;
}
```

## **10. المرجع السريع**

### **التسجيل السريع**

```php
// استخدام واحد لكل شيء
TagManager::instance()->quickRegister(
    \App\Models\Product::class,
    \App\Controllers\Products::class
);

// مع خيارات مخصصة
TagManager::instance()->quickRegister(
    \App\Models\Product::class,
    \App\Controllers\Products::class,
    [
        'form_tab' => 'app::lang.tab.tags',
        'form_field_name' => 'custom_tags',
        'with_column' => false,
        'with_filter' => true,
        'with_query' => true,
        'with_transformer' => true,
        'transformer_class' => \App\Transformers\ProductTransformer::class,
    ]
);
```

### **التحقق من التوفر**

```php
// التحقق مما إذا كان النموذج يدعم tags
if (TagManager::instance()->isModelRegistered($modelClass)) {
    // النموذج مسجل
    $config = TagManager::instance()->getModelConfig($modelClass);
    
    // التحقق من ميزة معينة
    if ($config['integrations'][TagManager::INTEGRATION_FORM_FIELD]) {
        // يدعم حقل النموذج
    }
    
    if ($config['integrations'][TagManager::INTEGRATION_QUERY]) {
        // يدعم تصفية الاستعلامات
    }
}
```

## **11. الميزات المتقدمة لمعالجة الاستعلامات**

### 11.1 معالجات القيم (Value Processors)

تتيح معالجات القيم معالجة قيم خاصة في الاستعلامات:

```php
'value_processors' => [
    'none' => function($query, $modelClass) {
        return $query->doesntHave('tags');
    },
    'any' => function($query, $modelClass) {
        return $query->has('tags');
    },
    'empty' => function($query, $modelClass) {
        return $query->whereDoesntHave('tags');
    },
]
```

**استخدام:** `?tags_id=none` أو `?tags_id=any`

### 11.2 المعاملات المنطقية (Logic Operators)

تدعم `AND` و `OR` للمنطق في الاستعلامات:

```php
'logic_operator' => 'AND', // أو 'OR'
```

**تأثير:**
- `AND` (الافتراضي): `?tags_id=1,2,3` = WHERE tag_id IN (1,2,3)
- `OR`: `?tags_id=1,2,3` = WHERE tag_id=1 OR tag_id=2 OR tag_id=3

### 11.3 معالجات القيم المتعددة (Multiple Value Handlers)

تدعم عدة طرق لمعالجة القيم المتعددة:

```php
'multiple_value_handler' => 'array', // array, comma, json, pipe
```

**الأنواع المتاحة:**

| النوع | الوصف | مثال |
|-------|-------|------|
| `array` | معالجة كمصفوفة | `[1, 2, 3]` |
| `comma` | فصل بفواصل | `1,2,3` |
| `json` | تنسيق JSON | `[1, 2, 3]` أو `{"tags": [1,2,3]}` |
| `pipe` | فصل بأنابيب (OR منطق) | `1\|2\|3` |

### 11.4 استعلامات معقدة

يمكن دمج المعاملات المنطقية:

```php
// استعلام معقد: (الوسم 1 أو 2) و (الوسم 3 أو 4)
'?tags_id=1,2|3,4'

// وهذا يترجم إلى:
// WHERE (tag_id=1 OR tag_id=2) AND (tag_id=3 OR tag_id=4)
```

### 11.5 إنشاء استعلامات متقدمة برمجياً

```php
use Nano\Tags\Classes\TagManager;

// إنشاء استعلام متقدم
$tagManager = TagManager::instance();
$query = $tagManager->createAdvancedQuery(
    \App\Models\Product::class,
    [
        'tags_id' => '1,2|3,4',
        'status' => 'active'
    ],
    [
        'with_tags_count' => true,
        'with_tags' => true,
        'tags_sort' => ['column' => 'name', 'direction' => 'asc']
    ]
);

// الحصول على النتائج
$products = $query->paginate(20);
```

### 11.6 أحداث مخصصة للاستعلامات

```php
// الاستماع لحدث الاستعلام المتقدم
Event::listen('nano.tags.extendQuery', function ($query, $params, $context) {
    // يمكنك تعديل الاستعلام هنا
    if ($context === 'api') {
        $query->where('is_active', true);
    }
});

// تشغيل الحدث يدوياً
Event::fire('nano.tags.extendQuery', [$query, $params, 'custom_context']);
```

### 11.7 أمثلة عملية

#### مثال 1: فلترة بواسطة OR منطقي
```php
// البحث عن المنتجات التي تحتوي على الوسم 1 أو 2
// URL: ?tags_id=1|2
$config = [
    'query_config' => [
        'logic_operator' => 'OR',
        'multiple_value_handler' => 'pipe',
    ]
];
```

#### مثال 2: معالجة JSON
```php
// إرسال قيم JSON في الطلب
// URL: ?tags_id=[1,2,3]
$config = [
    'query_config' => [
        'multiple_value_handler' => 'json',
    ]
];
```

#### مثال 3: استعلام معقد
```php
// البحث عن المنتجات التي:
// - تحتوي على (الوسم 1 أو 2) و (الوسم 3 أو 4)
// - ولا تحتوي على الوسم 5
// URL: ?tags_id=1,2|3,4&exclude_tags=5
$config = [
    'query_config' => [
        'value_processors' => [
            'exclude' => function($query, $modelClass, $excludedIds) {
                return $query->whereDoesntHave('tags', function($q) use ($excludedIds) {
                    $q->whereIn('id', (array)$excludedIds);
                });
            }
        ]
    ]
];
```

### 11.8 إعدادات التحكم

```php
'advanced_features' => [
    'value_processors_enabled' => true,
    'logic_operator_enabled' => true,
    'multiple_handlers_enabled' => true,
    'complex_handling_enabled' => true, // للاستعلامات المعقدة
    'strict_mode' => false, // إذا كان true، يفشل عند وجود أخطاء
    'debug_mode' => false, // تسجيل عمليات المعالجة
],
```

### 11.9 أفضل الممارسات

1. **استخدم `comma` للقيم البسيطة** من واجهة المستخدم
2. **استخدم `json` للتطبيقات المتقدمة** والـ APIs
3. **فعل `complex_handling`** فقط عند الحاجة للاستعلامات المعقدة
4. **اختر `logic_operator`** بناءً على متطلبات البحث
5. **أضف `value_processors` مخصصة** للاحتياجات الخاصة
```

## 3. أمثلة للاستخدام

### مثال 1: تسجيل نموذج مع معالجات متقدمة

```php
// في Plugin.php
public function registerTagIntegrations()
{
    return [
        \App\Models\Product::class => [
            'controller' => \App\Controllers\Products::class,
            'query_config' => [
                'param_keys' => ['tags', 'tag_ids', 'filter[tags]'],
                'logic_operator' => 'OR',
                'multiple_value_handler' => 'comma',
                'value_processors' => [
                    'featured' => function($query, $modelClass) {
                        return $query->whereHas('tags', function($q) {
                            $q->where('is_featured', true);
                        });
                    },
                    'popular' => function($query, $modelClass) {
                        return $query->whereHas('tags', function($q) {
                            $q->where('usage_count', '>', 100);
                        });
                    }
                ],
                'advanced_features' => [
                    'complex_handling_enabled' => true,
                    'debug_mode' => Config::get('app.debug'),
                ],
            ],
        ],
    ];
}
```

### مثال 2: استخدام متقدم في الكنترولر

```php
// في أي كنترولر
public function listProducts(Request $request)
{
    $tagManager = TagManager::instance();
    
    // إنشاء استعلام متقدم
    $query = $tagManager->createAdvancedQuery(
        \App\Models\Product::class,
        $request->all(),
        [
            'with_tags' => true,
            'with_tags_count' => true,
        ]
    );
    
    // تطبيق فلاتر إضافية
    if ($request->has('category_id')) {
        $query->where('category_id', $request->get('category_id'));
    }
    
    // الحصول على النتائج
    $products = $query->paginate($request->get('per_page', 20));
    
    return response()->json($products);
}
```

## **خلاصة**

`TagManager` يوفر نظاماً مرناً وقابلاً للتوسع لإدارة تكاملات TagsModel مع التحكم الكامل في كل جانب من جوانب التكامل. يمكن استخدامه في مشاريع صغيرة وكبيرة مع الحفاظ على الأداء والمرونة.