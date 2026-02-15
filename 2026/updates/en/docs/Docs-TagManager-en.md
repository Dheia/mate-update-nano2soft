# **TagManager - Complete Documentation**

## **1. Overview**

`TagManager` is a class for managing `TagsModel` integrations in the Nano2Soft system. It operates as a Singleton and provides a centralized interface for controlling how different models interact with the tagging system.

## **2. Available Integrations**

| Constant | Description | Function |
|--------|--------|----------|
| `INTEGRATION_BEHAVIOR` | Behavior injection | Adds `TagsModel` behavior to the model |
| `INTEGRATION_FORM_FIELD` | Form field injection | Adds a tags field in the Backend interface |
| `INTEGRATION_LIST_COLUMN` | List column injection | Adds a tags column in Backend lists |
| `INTEGRATION_LIST_FILTER` | List filter injection | Adds a tags filter in Backend lists |
| `INTEGRATION_QUERY` | Query injection | Supports tags filtering in API and Reports |
| `INTEGRATION_TRANSFORMER` | Transformer injection | Merges tags into API Transformers |

## **3. Basic Configuration**

### **Model Configuration Structure**

```php
$config = [
    // 1. Basic control
    'enabled' => true, // Enable/disable integration for the model
    
    // 2. Integration type control
    'integrations' => [
        TagManager::INTEGRATION_BEHAVIOR => true,
        TagManager::INTEGRATION_FORM_FIELD => true,
        TagManager::INTEGRATION_LIST_COLUMN => true,
        TagManager::INTEGRATION_LIST_FILTER => true,
        TagManager::INTEGRATION_QUERY => true,
        TagManager::INTEGRATION_TRANSFORMER => true,
    ],
    
    // 3. References
    'controller' => \App\Controllers\MyController::class, // Required for frontend integrations
    'transformer' => \App\Transformers\MyTransformer::class, // Optional for Transformer integration
    
    // 4. Specific configurations
    'query_config' => [...],
    'transformer_config' => [...],
    'form_config' => [...],
    'list_config' => [...],
    'filter_config' => [...],
];
```

## **4. Basic Methods**

### **4.1 Registration**

#### **Method 1: Bulk Registration (in Plugin.php)**

```php
// In Plugin.php
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

#### **Method 2: Individual Registration**

```php
$tagManager = TagManager::instance();

// Register a model with full configuration
$tagManager->registerModel(
    \App\Models\Product::class,
    [
        'controller' => \App\Controllers\Products::class,
        'form_config' => [
            'tab' => 'app::lang.products.tab.tags',
        ],
    ]
);

// Quick registration (with default settings)
$tagManager->quickRegister(
    \App\Models\Product::class,
    \App\Controllers\Products::class,
    [
        'form_tab' => 'app::lang.products.tab.basic',
        'with_column' => false, // We don't want a column in the list
        'with_filter' => true, // We want a filter
    ]
);
```

### **4.2 Integration Control**

```php
$tagManager = TagManager::instance();

// Enable a specific integration
$tagManager->enableIntegration(
    \App\Models\Product::class,
    TagManager::INTEGRATION_LIST_FILTER
);

// Disable a specific integration
$tagManager->disableIntegration(
    \App\Models\Product::class,
    TagManager::INTEGRATION_LIST_COLUMN
);

// Check integration status
if ($tagManager->isIntegrationEnabled(
    \App\Models\Product::class,
    TagManager::INTEGRATION_FORM_FIELD
)) {
    // The field is active
}

// Update existing configuration
$tagManager->updateModelConfig(
    \App\Models\Product::class,
    [
        'form_config' => [
            'cssClass' => 'col-sm-12', // Change field width
        ],
    ]
);
```

### **4.3 Manual Application**

```php
// Apply all integrations for a specific model
TagManager::instance()->applyIntegrationsForModel(\App\Models\Product::class);

// Apply only a specific integration
TagManager::instance()->applyBehaviorIntegration(\App\Models\Product::class);
TagManager::instance()->applyFormFieldIntegration(\App\Models\Product::class);
TagManager::instance()->applyQueryIntegration(\App\Models\Product::class);

// Apply all integrations for all registered models
TagManager::instance()->applyAllIntegrations();
```

## **5. Advanced Examples**

### **Example 1: Product with Full Integration**

```php
// In Plugin.php
public function registerTagIntegrations()
{
    return [
        \App\Models\Product::class => [
            'controller' => \App\Controllers\Products::class,
            'transformer' => \App\Transformers\ProductTransformer::class,
            
            // Integration control
            'integrations' => [
                TagManager::INTEGRATION_BEHAVIOR => true,
                TagManager::INTEGRATION_FORM_FIELD => true,
                TagManager::INTEGRATION_LIST_COLUMN => true,
                TagManager::INTEGRATION_LIST_FILTER => true,
                TagManager::INTEGRATION_QUERY => true,
                TagManager::INTEGRATION_TRANSFORMER => true,
            ],
            
            // Form field configuration
            'form_config' => [
                'field_name' => 'product_tags',
                'label' => 'app::lang.products.tags',
                'tab' => 'app::lang.products.tab.additional',
                'type' => 'nanotaglist',
                'cssClass' => 'col-sm-12',
                'customTags' => false, // Prevent creating new tags
            ],
            
            // Query configuration
            'query_config' => [
                'param_keys' => ['tags', 'product_tags', 'filter_tags'],
                'wildcard_values' => ['*', 'all', 'any'],
                'scope_method' => 'whereHasProductTags',
            ],
            
            // Transformer configuration
            'transformer_config' => [
                'include_relation' => 'product_tags',
                'include_method' => 'includeProductTags',
            ],
        ],
    ];
}
```

### **Example 2: Users (Limited Integration)**

```php
// In Plugin.php
public function registerTagIntegrations()
{
    return [
        \RainLab\User\Models\User::class => [
            'controller' => \RainLab\User\Controllers\Users::class,
            
            // Behavior only + query filter
            'integrations' => [
                TagManager::INTEGRATION_BEHAVIOR => true,
                TagManager::INTEGRATION_FORM_FIELD => false, // No field in the form
                TagManager::INTEGRATION_LIST_COLUMN => false, // No column in the list
                TagManager::INTEGRATION_LIST_FILTER => false, // No filter in the list
                TagManager::INTEGRATION_QUERY => true, // Filter in API
                TagManager::INTEGRATION_TRANSFORMER => true, // Include in API
            ],
            
            // Special query configuration
            'query_config' => [
                'param_keys' => ['user_tags', 'user_tags_id'],
                'scope_method' => 'whereHasUserTags',
            ],
        ],
    ];
}
```

### **Example 3: Blog Posts (Conditional Integration)**

```php
// In Plugin.php
public function registerTagIntegrations()
{
    return [
        \RainLab\Blog\Models\Post::class => [
            'controller' => \RainLab\Blog\Controllers\Posts::class,
            
            // Conditional integration based on another field
            'form_config' => [
                'field_name' => 'blog_tags',
                'tab' => 'rainlab.blog::lang.post.tab.categories',
                'trigger' => [
                    'action' => 'show',
                    'field' => 'enable_tags',
                    'condition' => 'checked',
                ],
            ],
            
            // Conditional filter
            'filter_config' => [
                'scope_name' => 'blog_tags_id',
                'label' => 'rainlab.blog::lang.post.tags',
                'type' => 'group',
                'modelClass' => 'Nano\Tags\Models\Tag',
                'options' => "listEnabled",
                'scope' => "whereHasBlogTags",
                'conditions' => "enable_tags = 1", // Additional condition
            ],
        ],
    ];
}
```

### **Example 4: Dynamic Registration at Runtime**

```php
// Anywhere in the application
public function boot()
{
    // Dynamic registration based on system settings
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
    
    // Different configuration per environment
    if (App::environment('production')) {
        $config['form_config']['customTags'] = false; // Prevent creating new tags in Production
    }
    
    $tagManager->registerModel(\App\Models\Product::class, $config);
}
```

### **Example 5: Modifying Existing Configurations**

```php
// In a Service Provider
public function boot()
{
    // Modify an existing configuration from another Plugin
    $tagManager = TagManager::instance();
    
    $currentConfig = $tagManager->getModelConfig(\Tss\Inventory\Models\Product::class);
    
    if ($currentConfig) {
        // Add extra CSS class
        $tagManager->updateModelConfig(
            \Tss\Inventory\Models\Product::class,
            [
                'form_config' => [
                    'cssClass' => $currentConfig['form_config']['cssClass'] . ' custom-tags-field',
                ],
            ]
        );
        
        // Disable the filter in a specific environment
        if (App::environment('testing')) {
            $tagManager->disableIntegration(
                \Tss\Inventory\Models\Product::class,
                TagManager::INTEGRATION_LIST_FILTER
            );
        }
    }
}
```

## **6. Advanced Query Configurations**

### **Multi-Filter Support**

```php
'query_config' => [
    'param_keys' => ['tags', 'tags_id', 'tag_ids', 'filter[tags]'],
    'wildcard_values' => ['*', 'all', 'any', 'none'],
    'scope_method' => 'whereHasTags',
    
    // Special handling for values
    'value_processors' => [
        'none' => function($query) {
            return $query->doesntHave('tags');
        },
        'any' => function($query) {
            return $query->has('tags');
        },
    ],
    
    // Support logical AND/OR
    'logic_operator' => 'AND', // or 'OR'
    
    // Multi-value handling
    'multiple_value_handler' => 'array', // array, comma, json
],
```

### **Advanced Transformer Configuration**

```php
'transformer_config' => [
    'include_relation' => 'tags',
    'include_method' => 'includeTags',
    'default_include' => true, // Default inclusion
    'eager_load' => true, // Eager load
    'transformer_class' => \Nano\Tags\Transformers\TagTransformer::class,
    
    // Additional fields in the Transformer
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

## **7. Custom Events**

You can listen to TagManager events:

```php
// Before applying integrations
Event::listen('nano.tags.beforeApplyIntegrations', function($modelClass, $config) {
    // Modify the configuration before application
});

// After applying integrations
Event::listen('nano.tags.afterApplyIntegrations', function($modelClass, $config) {
    // Perform additional actions
});

// When a new model is registered
Event::listen('nano.tags.modelRegistered', function($modelClass, $config) {
    Log::info("Model {$modelClass} registered with tags");
});
```

## **8. Best Practices**

### **8.1 Organization in Large Plugins**

```php
// In a large Plugin with multiple models
public function registerTagIntegrations()
{
    $integrations = [];
    
    // 1. Collect all integrations in one place
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
                // ... other integrations
            ],
        ];
    }
    
    return $configs;
}
```

### **8.2 Conditional Configuration**

```php
// Based on user settings
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

### **8.3 Migration from Legacy System**

```php
// Helper function for gradual migration
protected function migrateOldTagSystem()
{
    $tagManager = TagManager::instance();
    
    // List of old models
    $oldModels = [
        'product' => [
            'model' => \Tss\Inventory\Models\Product::class,
            'controller' => \Tss\Inventory\Controllers\Products::class,
        ],
        // ... other models
    ];
    
    foreach ($oldModels as $key => $data) {
        if (class_exists($data['model'])) {
            // Register with the new system
            $tagManager->registerModel($data['model'], [
                'controller' => $data['controller'],
                'form_config' => [
                    'field_name' => "{$key}_tags", // To maintain compatibility
                ],
            ]);
            
            // Disable the old system for this model
            $this->disableOldTagIntegration($key);
        }
    }
}
```

## **9. Troubleshooting**

### **Enable Debugging**

```php
// In AppServiceProvider or anywhere
if (Config::get('app.debug_tags', false)) {
    // Log all TagManager operations
    Event::listen('nano.tags.*', function($event, $data) {
        Log::debug('TagManager Event: ' . $event, $data);
    });
    
    // Check model configurations
    $models = TagManager::instance()->getRegisteredModels();
    Log::debug('Registered Tag Models:', array_keys($models));
}
```

### **Check Model Configuration**

```php
// Anywhere
public function checkTagConfig($modelClass)
{
    $tagManager = TagManager::instance();
    
    if ($tagManager->isModelRegistered($modelClass)) {
        $config = $tagManager->getModelConfig($modelClass);
        
        // Check enabled integrations
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

## **10. Quick Reference**

### **Quick Registration**

```php
// One-liner for everything
TagManager::instance()->quickRegister(
    \App\Models\Product::class,
    \App\Controllers\Products::class
);

// With custom options
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

### **Availability Check**

```php
// Check if a model supports tags
if (TagManager::instance()->isModelRegistered($modelClass)) {
    // The model is registered
    $config = TagManager::instance()->getModelConfig($modelClass);
    
    // Check a specific feature
    if ($config['integrations'][TagManager::INTEGRATION_FORM_FIELD]) {
        // Supports form field
    }
    
    if ($config['integrations'][TagManager::INTEGRATION_QUERY]) {
        // Supports query filtering
    }
}
```

## **11. Advanced Query Processing Features**

### 11.1 Value Processors

Value processors allow handling special values in queries:

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

**Usage:** `?tags_id=none` or `?tags_id=any`

### 11.2 Logic Operators

Supports `AND` and `OR` logic in queries:

```php
'logic_operator' => 'AND', // or 'OR'
```

**Effect:**
- `AND` (default): `?tags_id=1,2,3` = WHERE tag_id IN (1,2,3)
- `OR`: `?tags_id=1,2,3` = WHERE tag_id=1 OR tag_id=2 OR tag_id=3

### 11.3 Multiple Value Handlers

Supports multiple methods for handling multiple values:

```php
'multiple_value_handler' => 'array', // array, comma, json, pipe
```

**Available types:**

| Type | Description | Example |
|-------|-------|------|
| `array` | Handle as an array | `[1, 2, 3]` |
| `comma` | Comma-separated | `1,2,3` |
| `json` | JSON format | `[1, 2, 3]` or `{"tags": [1,2,3]}` |
| `pipe` | Pipe-separated (OR logic) | `1\|2\|3` |

### 11.4 Complex Queries

You can combine logical operators:

```php
// Complex query: (tag 1 or 2) and (tag 3 or 4)
'?tags_id=1,2|3,4'

// This translates to:
// WHERE (tag_id=1 OR tag_id=2) AND (tag_id=3 OR tag_id=4)
```

### 11.5 Creating Advanced Queries Programmatically

```php
use Nano\Tags\Classes\TagManager;

// Create an advanced query
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

// Get results
$products = $query->paginate(20);
```

### 11.6 Custom Query Events

```php
// Listen to the advanced query event
Event::listen('nano.tags.extendQuery', function ($query, $params, $context) {
    // You can modify the query here
    if ($context === 'api') {
        $query->where('is_active', true);
    }
});

// Fire the event manually
Event::fire('nano.tags.extendQuery', [$query, $params, 'custom_context']);
```

### 11.7 Practical Examples

#### Example 1: Filtering with OR Logic
```php
// Find products that contain tag 1 or 2
// URL: ?tags_id=1|2
$config = [
    'query_config' => [
        'logic_operator' => 'OR',
        'multiple_value_handler' => 'pipe',
    ]
];
```

#### Example 2: JSON Handling
```php
// Send JSON values in the request
// URL: ?tags_id=[1,2,3]
$config = [
    'query_config' => [
        'multiple_value_handler' => 'json',
    ]
];
```

#### Example 3: Complex Query
```php
// Find products that:
// - Contain (tag 1 or 2) and (tag 3 or 4)
// - Do not contain tag 5
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

### 11.8 Control Settings

```php
'advanced_features' => [
    'value_processors_enabled' => true,
    'logic_operator_enabled' => true,
    'multiple_handlers_enabled' => true,
    'complex_handling_enabled' => true, // For complex queries
    'strict_mode' => false, // If true, fails on errors
    'debug_mode' => false, // Log processing operations
],
```

### 11.9 Best Practices

1. **Use `comma` for simple values** from the user interface
2. **Use `json` for advanced applications** and APIs
3. **Enable `complex_handling`** only when complex queries are needed
4. **Choose `logic_operator`** based on search requirements
5. **Add custom `value_processors`** for specific needs

## 3. Usage Examples

### Example 1: Registering a Model with Advanced Processors

```php
// In Plugin.php
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

### Example 2: Advanced Usage in a Controller

```php
// In any controller
public function listProducts(Request $request)
{
    $tagManager = TagManager::instance();
    
    // Create an advanced query
    $query = $tagManager->createAdvancedQuery(
        \App\Models\Product::class,
        $request->all(),
        [
            'with_tags' => true,
            'with_tags_count' => true,
        ]
    );
    
    // Apply additional filters
    if ($request->has('category_id')) {
        $query->where('category_id', $request->get('category_id'));
    }
    
    // Get results
    $products = $query->paginate($request->get('per_page', 20));
    
    return response()->json($products);
}
```

## **Summary**

`TagManager` provides a flexible and extensible system for managing TagsModel integrations with full control over every aspect of the integration. It can be used in both small and large projects while maintaining performance and flexibility.