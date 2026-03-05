## 2026-3-3 - 2026-3-6

**Advanced Traits for Hierarchical Data and Improvements to the Categorization System**

### Summary of Achievement

An integrated package for managing hierarchical data and categories in NanoSoft systems has been developed, including:

1. **`HasAdvancedTree` Trait** – For managing tree relationships (ancestors and descendants) with high efficiency.
2. **`CategoriesModel` Behavior Update** – To enhance handling of categories and leverage the advantages of the new trait.
3. **Comprehensive Documentation** – In both Arabic and English explaining usage and examples.

### Details of Achievement

#### First: `HasAdvancedTree` Trait

**Objective**: Provide advanced functions for handling hierarchical models (those containing `parent_id`) without the need for additional columns, while improving performance using recursive queries (CTE) and caching.

**Key Features**:
- **High Performance**: Uses recursive queries (CTE) to fetch ancestors and descendants in a single query instead of iterating through relationships.
- **Soft Delete Support**: Ability to include or exclude soft-deleted records via the `$withTrashed` parameter.
- **Caching**: Functions `getCachedNodeAncestors` and `getCachedNodeDescendants` to cache results and avoid repeated queries.
- **Full Compatibility**: Functions are prefixed with `Node` (e.g., `getNodeAncestors`, `isNodeRoot`) to work alongside any other traits without conflict.
- **Practical Helper Functions**: Build a nested tree (`buildNestedNodeTree`), a flat list for dropdowns (`toFlatNodeTree`), breadcrumb path (`getNodePath`), and more.

**Additional Improvements**:
- The `databaseSupportsRecursive` function was developed to check the actual database version (MySQL 8+, SQLite 3.8.3+, PostgreSQL) to ensure support for recursive queries before using them, ensuring compatibility with older versions.

**Documentation**: A comprehensive documentation file (`Docs-HasAdvancedTree-en.md`) has been prepared explaining all functions with practical examples and their outputs.

#### Second: `CategoriesModel` Behavior Update

**Objective**: Improve the behavior responsible for linking any model (e.g., products, articles) to categories stored in `Nano\Tags\Models\Categorie`, and leverage the capabilities of `HasAdvancedTree`.

**New Features**:
- **`prepareCategoryIds` Function**: Processes various inputs (numbers, slugs, comma-separated strings, mixed arrays) and converts them into an array of numeric IDs with caching support.
- **Enhanced Scopes**:
  - `scopeWhereCategories` with support for `any` (any category) and `all` (all categories).
  - `scopeWhereHasCategoryWithDescendants` to include all descendants (using `getNodeDescendants`).
  - `scopeWhereHasCategoryWithAncestors` to include ancestors.
  - `scopeWhereHasCategoryUpToDepth` for filtering up to a specific depth.
- **Ordering Scopes**: `orderByCategoryName` and `orderByCategoriesCount`.
- **Validation Functions**: `hasCategory` with option to include ancestors/descendants, `hasAnyCategory`, `hasAllCategories`.
- **Helper Functions**: `getCategoriesHierarchy` to build a tree of associated categories, `getCategoriesFlatList` for dropdowns.

**Documentation**: A documentation file (`CategoriesModel.md`) has been prepared explaining all new and improved functions with multiple examples.

### Benefits Achieved

1. **Higher Efficiency**: Using recursive queries significantly reduces the number of queries, especially with large trees.
2. **Greater Flexibility**: Support for mixed inputs (numbers and slugs) makes it easier to use categories in user interfaces.
3. **Faster Development**: Helper functions like `getNodePath` and `getCategoriesFlatList` save development time for common tasks.
4. **Integrated Documentation**: Helps the team quickly understand and use the new tools.
5. **Compatibility**: The trait works with older database versions via automatic fallback to the iterative method.

### Next Steps

- Integrate the trait and behavior into current projects and test performance.
- Add automated tests to verify the correctness of functions in various scenarios.
- Study the possibility of adding new functions such as moving nodes (`moveNode`) within the tree using recursive queries.