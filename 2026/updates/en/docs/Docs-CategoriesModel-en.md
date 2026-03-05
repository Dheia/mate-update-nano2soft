# Documentation for `CategoriesModel` Behavior

## Introduction

`CategoriesModel` is an advanced behavior designed to add a polymorphic many-to-many relationship with categories to any model in OctoberCMS. It allows any model (e.g., products, posts, users, etc.) to be linked to categories stored in the `Nano\Tags\Models\Categorie` model, while providing a rich set of methods and scopes for handling these categories flexibly and efficiently.

### Key Benefits

- **Unified category relationship** – Easily add categories to any model.
- **Slug support** – Use slugs instead of numeric IDs in all methods.
- **Mixed arrays** – Accept a mixture of numbers and text (slugs) in the same array.
- **Hierarchical filtering** – Include ancestors and descendants when filtering (using `HasAdvancedTree`).
- **Multiple match types** – Support `any` (any of the categories) and `all` (all of the categories).
- **Performance optimization** – Caching of prepared IDs to avoid repetitive queries.
- **Helper check methods** – Verify the existence of categories in various ways.
- **Ordering by categories** – Sort results by category name or category count.

---

## Requirements

1. The `Nano\Tags\Models\Categorie` model must exist and implement the necessary traits (especially `HasAdvancedTree` to benefit from hierarchical methods).
2. The pivot table defined in `Categorie::PIVOT_TABLE` (default `nano_tags_cateables`) must exist.
3. The target model must be an Eloquent model (or `October\Rain\Database\Model`).

---

## Adding the Behavior to a Model

You can add the behavior to any model using the `$implement` property (in OctoberCMS) or manually via `extend()`.

### Method 1 (in OctoberCMS using `$implement`)

```php
class Product extends Model
{
    public $implement = [
        'Nano\Tags\Behaviors\CategoriesModel'
    ];
}
```

### Method 2 (manually using `extend`)

```php
Product::extend(function($model) {
    $model->implement[] = 'Nano\Tags\Behaviors\CategoriesModel';
});
```

After adding the behavior, the model gains the following relationship:

```php
$product->pcategories // returns a collection of associated categories (of type Categorie)
```

---

## Available Methods and Scopes

### 1. ID Preparation Methods (internal)

#### `prepareCategoryIds($categories, $is_force_all = false, $useCache = true)`
- **Description**: Convert various input types into an array of numeric IDs.
- **Parameters**:
  - `$categories`: Can be a number, string (ID, slug, or comma-separated string), mixed array, `Categorie` object, or `Collection`.
  - `$is_force_all`: Ignore the "all" value check.
  - `$useCache`: Use caching.
- **Returns**: Array of unique integer IDs.

### 2. Basic Filtering Scopes

#### `scopeWhereCategories($query, $categories, $matchType = 'any', $not = false, $forceAll = false, $useCache = true)`
- **Description**: The main scope for filtering by categories.
- **Parameters**:
  - `$categories`: Required categories (supports all types).
  - `$matchType`: `'any'` (any of the categories) or `'all'` (all of the categories).
  - `$not`: Invert the condition (does not have these categories).
  - `$forceAll`: Ignore the "all" value.
  - `$useCache`: Use caching.
- **Example**:
  ```php
  // Products that have any of categories 1, 5, or the one with slug "electronics"
  $products = Product::whereCategories([1, 5, 'electronics'])->get();
  
  // Products that have both categories 1 and "clothing"
  $products = Product::whereCategories([1, 'clothing'], 'all')->get();
  ```

#### `scopeWhereAnyCategories($query, $categories, $not = false, $forceAll = false, $useCache = true)`
- Shortcut for `whereCategories` with `matchType = 'any'`.

#### `scopeWhereAllCategories($query, $categories, $not = false, $forceAll = false, $useCache = true)`
- Shortcut for `whereCategories` with `matchType = 'all'`.

#### `scopeOrWhereCategories($query, $categories, $forceAll = false, $useCache = true)`
- Add an OR condition for categories.

#### `scopeWhereNotCategories($query, $categories, $forceAll = false, $useCache = true)`
- Filter results that do not have the specified categories.

### 3. Slug-Based Filtering Scopes

#### `scopeWhereCategorySlug($query, $slug, $matchType = 'any', $not = false, $forceAll = false)`
- Filter by a category slug.

#### `scopeOrWhereCategorySlug($query, $slug, $forceAll = false)`
- OR condition for a category slug.

### 4. Active Categories Filtering

#### `scopeWhereCategoriesActive($query, $categories, $matchType = 'any', $not = false, $forceAll = false, $useCache = true)`
- Filter by only active categories (`is_active = true`).

### 5. Hierarchical Filtering Scopes (using `HasAdvancedTree`)

#### `scopeWhereHasCategoryWithDescendants($query, $categoryId, $includeDescendants = true, $matchType = 'any', $not = false, $forceAll = false, $useCache = true)`
- Include all descendants of the specified category.
- Example: `Product::whereHasCategoryWithDescendants('electronics')->get()` fetches all products in the "electronics" category or any subcategory under it.

#### `scopeWhereHasCategoryWithAncestors($query, $categoryId, $includeAncestors = true, $matchType = 'any', $not = false, $forceAll = false, $useCache = true)`
- Include all ancestors of the specified category.

#### `scopeWhereHasCategoryUpToDepth($query, $categoryId, $depth, $matchType = 'any', $not = false, $forceAll = false, $useCache = true)`
- Include descendants up to a specific depth (1 = immediate children, 2 = children and grandchildren, ...).

### 6. Ordering Scopes

#### `scopeOrderByCategoryName($query, $direction = 'asc')`
- Order results by the name of the first associated category.

#### `scopeOrderByCategoriesCount($query, $direction = 'asc')`
- Order results by the count of associated categories.

### 7. Computed Property Methods

#### `getIsCategoriesAttribute()`
- **Description**: Check if the model has any categories.
- **Usage**: `$product->is_categories` (returns true/false).

#### `getCategoriesCountAttribute()`
- **Description**: Number of associated categories.
- **Usage**: `$product->categories_count`.

#### `getPcategoriesCountAttribute()`
- Alias for the above.

### 8. Helper Methods for Fetching Categories

#### `getCategories($relations = [], $activeOnly = false)`
- **Description**: Fetch all associated categories.
- **Parameters**:
  - `$relations`: Relations to eager load.
  - `$activeOnly`: Fetch only active categories.
- **Returns**: `Collection` of `Categorie` models.
- **Example**: `$product->getCategories(['parent'])` fetches categories with parent loaded.

#### `getCategoriesHierarchy($activeOnly = false)`
- **Description**: Fetch categories as a nested tree (with `children` populated).
- **Returns**: `Collection` of roots with `children` filled.

#### `getCategoriesFlatList($labelColumn = 'name', $keyColumn = 'id', $indent = '&nbsp;&nbsp;&nbsp;', $activeOnly = false)`
- **Description**: Fetch a flat list with indentation (suitable for dropdowns).
- **Returns**: Array `[id => indent + name]`.

#### `getCategoryIds()`
- **Description**: Fetch an array of associated category IDs.

### 9. Verification Methods

#### `hasCategory($categoryId, $relationType = 'self', $forceAll = false, $useCache = true)`
- **Description**: Check for the existence of a specific category or its relations.
- **`$relationType`**:
  - `'self'`: the category itself.
  - `'descendants'`: the category or its descendants.
  - `'ancestors'`: the category or its ancestors.
  - `'both'`: the category, its ancestors, or its descendants.
- **Example**: `$product->hasCategory('electronics', 'descendants')`.

#### `hasAnyCategory(array $categoryIds)`
- Check if any of the specified categories exist.

#### `hasAllCategories(array $categoryIds)`
- Check if all of the specified categories exist.

---

## Comprehensive Practical Examples

### Example 1: Product Categorization

Assume we have a `Product` model with the behavior attached. We want to:

```php
// Create a product and attach categories
$product = Product::find(1);
$product->pcategories()->attach([1, 2, 3]); // attach by IDs
$product->pcategories()->attach(Categorie::where('slug', 'electronics')->first()); // attach by object

// Fetch all products in the "electronics" category
$products = Product::whereCategorySlug('electronics')->get();

// Fetch products in "electronics" or any subcategory
$products = Product::whereHasCategoryWithDescendants('electronics')->get();

// Fetch products that have both "electronics" and "sale"
$products = Product::whereAllCategories(['electronics', 'sale'])->get();

// Fetch products that do not have the "out-of-stock" category
$products = Product::whereNotCategories('out-of-stock')->get();
```

### Example 2: Using Mixed Arrays

```php
// Comma-separated string
$products = Product::whereCategories("1,5,electronics,clothing")->get();

// Mixed array
$products = Product::whereCategories([1, 'electronics', 7, 'sale'])->get();
// The function will extract IDs 1,7 and look up slugs "electronics" and "sale", converting them to IDs

// Using forceAll to treat "all" as a slug
$products = Product::whereCategories(['all', 'electronics'], 'any', false, true)->get();
// forceAll = true means "all" is treated as a slug, not a special value
```

### Example 3: Hierarchical Filtering

```php
// All products in "electronics" or any subcategory
$products = Product::whereHasCategoryWithDescendants('electronics')->get();

// All products in "phones" or any ancestor (parent categories)
$products = Product::whereHasCategoryWithAncestors('phones')->get();

// All products in "iphone" or its children up to depth 2
$products = Product::whereHasCategoryUpToDepth('iphone', 2)->get();
```

### Example 4: Ordering and Computed Properties

```php
// Order products by category name
$products = Product::orderByCategoryName()->get();

// Order by category count (descending)
$products = Product::orderByCategoriesCount('desc')->get();

// Using computed properties
foreach ($products as $product) {
    echo $product->name . ' - Categories: ' . $product->categories_count . "\n";
    if ($product->is_categories) {
        echo "Has categories\n";
    }
}
```

### Example 5: Fetching Categories in Various Ways

```php
$product = Product::find(1);

// Fetch categories with relations loaded
$categories = $product->getCategories(['parent', 'children']);

// Fetch as a tree
$tree = $product->getCategoriesHierarchy();
foreach ($tree as $root) {
    echo $root->name . "\n";
    foreach ($root->children as $child) {
        echo " - " . $child->name . "\n";
    }
}

// Fetch a flat dropdown list
$list = $product->getCategoriesFlatList('name', 'id', '--');
// Output: [1 => 'Electronics', 2 => '--Phones', 3 => '----iPhone', ...]
```

### Example 6: Advanced Verification

```php
$product = Product::find(1);

if ($product->hasCategory('electronics')) {
    echo "Product is in the electronics category";
}

if ($product->hasCategory('electronics', 'descendants')) {
    echo "Product is in electronics or any subcategory";
}

if ($product->hasAnyCategory([1, 5, 7])) {
    echo "Product has at least one of these categories";
}

if ($product->hasAllCategories([2, 3])) {
    echo "Product has both of these categories";
}
```

---

## Performance and Caching

The behavior uses caching by default when preparing IDs (`prepareCategoryIds`) to avoid repeated slug lookups. You can control this with the `$useCache` parameter.

- Cache TTL: one hour (adjustable by changing the `$cacheTtl` property).
- A unique cache key is generated based on the input.

You can temporarily disable caching:

```php
$products = Product::whereCategories($categories, 'any', false, false, false); // last false disables cache
```

---

## Backward Compatibility

Legacy methods (`scopeWhereHasCategory`, `scopeWhereHasCategorie`, `scopeWhereHasPcategorie`) are kept for compatibility with older code. They call the new `whereAnyCategories` scope with default values.

It is recommended to use the new methods (`whereCategories`, `whereAnyCategories`, `whereAllCategories`) in new development.

---

## Summary

`CategoriesModel` is a powerful and flexible tool for managing categories in any model. With its support for slugs, mixed arrays, hierarchical filtering, and caching, developers can build advanced categorization systems easily and efficiently. We hope this documentation helps you explore the full capabilities of the behavior.