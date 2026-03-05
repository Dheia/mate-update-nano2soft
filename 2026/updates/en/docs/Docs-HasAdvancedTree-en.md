# Documentation for `HasAdvancedTree` Trait

## Introduction

`HasAdvancedTree` is an advanced trait for managing hierarchical (tree) data in Eloquent models for Laravel / OctoberCMS. It provides efficient methods for handling ancestors and descendants with significant performance improvements using recursive queries (CTE) and caching.

This trait is designed to work independently or alongside other tree traits such as `SimpleTree` and `NestedTree` without any conflicts, thanks to unique method names prefixed with `Node` (e.g., `getNodeAncestors`, `getNodeDescendants`, etc.).

---

## Key Features

- **High Performance** – Uses recursive SQL queries (CTE) to fetch ancestors and descendants in a single query instead of iterating through relationships (supported in MySQL 8+, PostgreSQL, SQLite).
- **Caching** – Provides methods to cache ancestor and descendant results to avoid repeated queries, with the ability to manually clear the cache.
- **SoftDeletes Support** – Handles soft-deleted records flexibly via the `$withTrashed` parameter in most methods.
- **Practical Helper Methods** – Node path (breadcrumb), relationship checks, nested tree building, flat lists, and more.
- **Full Compatibility** – Can be used with `SimpleTree` or `NestedTree` without conflicts due to unique method names.
- **High Flexibility** – Works with any model that has a `parent_id` column and `children`/`parent` relationships.

---

## Requirements

- The model's table must have a `parent_id` column (the column name can be changed by defining a `PARENT_ID` constant in the model).
- The model must have `children` (hasMany) and `parent` (belongsTo) relationships.  
  **Note:** The trait automatically creates these relationships if they don't exist when the model is initialized (via `initializeHasAdvancedTree`).

---

## Installation

1. Copy the `HasAdvancedTree.php` file to the appropriate path in your project (e.g., `nano/tags/traits/`).
2. In the model you want to add the trait to, use `HasAdvancedTree`:

```php
use Nano\Tags\Traits\HasAdvancedTree;

class Categorie extends Model
{
    use HasAdvancedTree;

    // (Optional) Change the parent_id column name
    const PARENT_ID = 'parent_id';
}
```

3. Ensure the following relationships exist in the model (the trait adds them automatically, but you can also define them manually):

```php
public $hasMany = [
    'children' => [self::class, 'key' => 'parent_id']
];

public $belongsTo = [
    'parent' => [self::class, 'key' => 'parent_id']
];
```

---

## Compatibility with SimpleTree and NestedTree

You can use `HasAdvancedTree` alongside `SimpleTree` or `NestedTree` in the same model without issues because all `HasAdvancedTree` methods are prefixed with `Node` (e.g., `getNodeAncestors`), while the original trait methods have generic names (e.g., `getParent`). In case of conflicts with helper methods (like `getParentColumnName`), you can use `insteadof` to specify which method to use:

```php
use October\Rain\Database\Traits\NestedTree;
use Nano\Tags\Traits\HasAdvancedTree;

class Categorie extends Model
{
    use NestedTree, HasAdvancedTree {
        NestedTree::getParentColumnName insteadof HasAdvancedTree;
        NestedTree::getParentId insteadof HasAdvancedTree;
    }
}
```

---

## Available Methods

### 1. Ancestor Methods

| Method | Description |
|--------|-------------|
| `getNodeAncestors($columns = ['*'], $withTrashed = false)` | Returns all ancestors from parent up to the root. |
| `getNodeAncestorsWithTrashed($columns = ['*'])` | Returns ancestors including soft-deleted ones. |
| `getNodeSelfAndAncestors($columns = ['*'], $withTrashed = false)` | Returns the current node along with its ancestors. |

### 2. Descendant Methods

| Method | Description |
|--------|-------------|
| `getNodeDescendants($columns = ['*'], $withTrashed = false)` | Returns all descendants (at any depth). Uses CTE if the database supports it, otherwise falls back to iteration. |
| `getNodeDescendantsWithTrashed($columns = ['*'])` | Returns descendants including soft-deleted ones. |
| `getNodeSelfAndDescendants($columns = ['*'], $withTrashed = false)` | Returns the current node along with its descendants. |
| `getNodeDescendantsUpToDepth(int $depth, array $columns = ['*'], $withTrashed = false)` | Returns descendants up to a specified depth (1 = immediate children, 2 = children and grandchildren, ...). |
| `getNodeDescendantsWith($relations, $columns = ['*'], $withTrashed = false)` | Returns descendants with the specified relationships eager loaded (avoids N+1 problem). |

### 3. Check and Property Methods

| Method | Description |
|--------|-------------|
| `isNodeRoot()` | Checks if the node is a root (has no parent). |
| `isNodeLeaf()` | Checks if the node is a leaf (has no children). |
| `getNodeDepth($withTrashed = false)` | Returns the node's depth (number of ancestors). |
| `getNodeRoot($withTrashed = false)` | Returns the tree root for this node (highest ancestor). |
| `getNodeSiblings($withTrashed = false)` | Returns all sibling nodes (excluding itself). |
| `getNodePath(string $separator = ' > ', string $attribute = 'name', $withTrashed = false): string` | Builds a breadcrumb string. |
| `isNodeAncestorOf(self $node, $withTrashed = false): bool` | Checks if the current node is an ancestor of another node. |
| `isNodeDescendantOf(self $node, $withTrashed = false): bool` | Checks if the current node is a descendant of another node. |
| `getNodeDescendantsCount($withTrashed = false)` | Returns the number of descendants (without loading them). |

### 4. Scopes

| Scope | Description |
|-------|-------------|
| `scopeWhereNodeDescendantOf($query, $id, $withTrashed = false)` | Filters nodes that are descendants of a given node (including the node itself). |
| `scopeWhereNodeAncestorOf($query, $id, $withTrashed = false)` | Filters nodes that are ancestors of a given node (including the node itself). |
| `scopeWithNodeDepth($query, $depthColumn = 'depth')` | Adds a computed depth column to the results (supports CTE). |
| `scopeWhereNodeDepth($query, $operator, $value, $depthColumn = 'depth')` | Filters nodes by depth (e.g., `whereNodeDepth('>', 2)`). |

### 5. General Static Helpers

| Method | Description |
|--------|-------------|
| `buildFlatNodeTree($items, $labelColumn = 'name', $keyColumn = 'id', $indent = '&nbsp;&nbsp;&nbsp;')` | Builds a flat list with indentation from any collection of nodes (useful for dropdowns). |
| `toFlatNodeTree($labelColumn = 'name', $keyColumn = 'id', $indent = '&nbsp;&nbsp;&nbsp;')` | Returns a flat array of all nodes in the tree (ordered by depth). |
| `buildNestedNodeTree($items, $parentId = null)` | Builds a nested tree (with `children` populated) from a flat collection of nodes. |
| `getNodeRoots($withTrashed = false)` | Returns all root nodes. |
| `getNodeHierarchy($withTrashed = false)` | Returns a complete hierarchical representation (roots with nested children). |

### 6. Caching Methods

| Method | Description |
|--------|-------------|
| `getCachedNodeAncestors($columns = ['*'])` | Returns ancestors from cache (TTL defined in `$cacheTtl`). |
| `getCachedNodeDescendants($columns = ['*'])` | Returns descendants from cache. |
| `clearNodeTreeCache()` | Clears the cache for the current node. |

---

## Practical Examples

Assume we have a `Categorie` model with the following data:

| id | name          | parent_id |
|----|---------------|-----------|
| 1  | Electronics   | null      |
| 2  | Phones        | 1         |
| 3  | iPhone        | 2         |
| 4  | Samsung       | 2         |
| 5  | Tablets       | 1         |
| 6  | iPad          | 5         |
| 7  | Clothing      | null      |
| 8  | Men           | 7         |
| 9  | Women         | 7         |

### Example 1: Get all ancestors of "iPhone"

```php
$iphone = Categorie::find(3);
$ancestors = $iphone->getNodeAncestors(['id', 'name']);
```

**Output** (collection of objects):
```
Collection {
    0 => Categorie { id: 2, name: "Phones" },
    1 => Categorie { id: 1, name: "Electronics" }
}
```

### Example 2: Get all descendants of "Electronics"

```php
$electronics = Categorie::find(1);
$descendants = $electronics->getNodeDescendants(['id', 'name']);
```

**Output** (flat collection):
```
Collection {
    0 => Categorie { id: 2, name: "Phones" },
    1 => Categorie { id: 3, name: "iPhone" },
    2 => Categorie { id: 4, name: "Samsung" },
    3 => Categorie { id: 5, name: "Tablets" },
    4 => Categorie { id: 6, name: "iPad" }
}
```

### Example 3: Get descendants up to depth 1 (immediate children only)

```php
$descendantsDepth1 = $electronics->getNodeDescendantsUpToDepth(1, ['id', 'name']);
```

**Output**:
```
Collection {
    0 => Categorie { id: 2, name: "Phones" },
    1 => Categorie { id: 5, name: "Tablets" }
}
```

### Example 4: Breadcrumb path for "iPad"

```php
$ipad = Categorie::find(6);
$path = $ipad->getNodePath(' / ', 'name');
```

**Output**:
```
"Electronics / Tablets / iPad"
```

### Example 5: Relationship checks

```php
$electronics = Categorie::find(1);
$iphone = Categorie::find(3);

if ($electronics->isNodeAncestorOf($iphone)) {
    echo "Electronics is an ancestor of iPhone";
}
// Output: Electronics is an ancestor of iPhone

if ($iphone->isNodeDescendantOf($electronics)) {
    echo "iPhone is a descendant of Electronics";
}
// Output: iPhone is a descendant of Electronics
```

### Example 6: Get sibling nodes

```php
$iphone = Categorie::find(3);
$siblings = $iphone->getNodeSiblings(['id', 'name']);
```

**Output**:
```
Collection {
    0 => Categorie { id: 4, name: "Samsung" }
}
```

### Example 7: Using scopes

```php
// All categories that are descendants of "Electronics"
$descendantCategories = Categorie::whereNodeDescendantOf(1)->get();

// All categories with depth greater than 1 (neither roots nor immediate children)
$deepCategories = Categorie::whereNodeDepth('>', 1)->get();
```

### Example 8: Build a nested tree

```php
$allCategories = Categorie::all();
$nestedTree = Categorie::buildNestedNodeTree($allCategories);

foreach ($nestedTree as $root) {
    echo $root->name . "\n";
    foreach ($root->children as $child) {
        echo "  " . $child->name . "\n";
    }
}
```

**Output**:
```
Electronics
  Phones
  Tablets
Clothing
  Men
  Women
```

### Example 9: Get a flat list with indentation (for dropdowns)

```php
$flatList = Categorie::toFlatNodeTree('name', 'id', '--');
```

**Output**:
```php
[
    1 => 'Electronics',
    2 => '--Phones',
    3 => '----iPhone',
    4 => '----Samsung',
    5 => '--Tablets',
    6 => '----iPad',
    7 => 'Clothing',
    8 => '--Men',
    9 => '--Women'
]
```

### Example 10: Using caching

```php
// First time: query executed and cached
$ancestors = $iphone->getCachedNodeAncestors();

// Second time: fetched from cache (faster)
$ancestorsAgain = $iphone->getCachedNodeAncestors();

// Clear cache for this node
$iphone->clearNodeTreeCache();
```

### Example 11: Fetch descendants with eager loading

Assume `Categorie` has a `products` relationship. We want all descendants of a category with their products loaded:

```php
$electronics = Categorie::find(1);
$descendantsWithProducts = $electronics->getNodeDescendantsWith('products', ['id', 'name']);
```

This first fetches the descendant IDs only, then loads them with the `products` relationship in a single query, avoiding N+1.

### Example 12: Get all root nodes

```php
$roots = Categorie::getNodeRoots();
foreach ($roots as $root) {
    echo $root->name; // Electronics, Clothing
}
```

### Example 13: Get full hierarchy

```php
$hierarchy = Categorie::getNodeHierarchy();
// Returns a collection of roots with children populated for all levels
```

---

## Important Notes

- The trait does not require any additional columns beyond `parent_id`. It relies solely on the `parent` and `children` relationships.
- When using `getNodeDescendants` with very large sets, consider using `getNodeDescendantsCount` first to estimate size, or use `getNodeDescendantsUpToDepth` if you need a specific depth.
- CTE methods (like `getNodeDescendantsUsingCte`) work only with databases that support recursive queries (MySQL 8+, PostgreSQL, SQLite). If unsupported, the trait automatically falls back to the iterative method.
- When `withTrashed = true` is used, the trait always falls back to the iterative method to ensure accurate results with soft-deleted records, as combining soft deletes with CTE is complex.
- The trait automatically creates the `children` and `parent` relationships if they are missing during model initialization (in `initializeHasAdvancedTree`). This is useful in OctoberCMS.
- The cache TTL can be changed by modifying the `$cacheTtl` property (in minutes) in the model using the trait.
- All methods that handle ancestors and descendants support specifying the required columns (`$columns`) to reduce loaded data.

---

## Conclusion

`HasAdvancedTree` is a powerful addition to any model that needs to manage hierarchical data in Laravel or OctoberCMS. With its high performance, flexibility, and compatibility with other traits, it can simplify many common tasks such as building nested menus, displaying breadcrumbs, and complex tree queries. We hope this documentation helps you use the trait efficiently.