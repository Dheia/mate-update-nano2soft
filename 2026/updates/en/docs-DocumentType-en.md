# DocumentTypes Class Documentation

## 📋 Overview

`DocumentTypes` is a central and professional class for managing legal document types in the Nano2soft system (Nano3.Legal plugin). This class provides a structured and flexible framework for defining, categorizing, and retrieving different legal document types with full support for multilingual translation, in addition to providing ready-made default content for documents via the `DefaultContentDocumentTypes` trait.

## 🎯 Class Objectives

- **Unify Document Types**: Create a central reference for all document types in the system
- **Facilitate Maintenance**: Update and add new types in one place only
- **Improve Performance**: Reduce database queries
- **Enhance Experience**: Provide an easy-to-use interface for developers
- **Provide Default Content**: Create professionally ready documents

## ✨ Key Features

### 1. **Centralization and Organization**
- All document types in one place
- Divided into 8 main logical categories
- Easy access and use

### 2. **Maintainability**
- Easily add new types
- Update translations only once
- No duplication in definitions

### 3. **Security and Accuracy**
- Use constants to reduce errors
- Validate types
- Prevent unknown type entries

### 4. **Developer Experience**
- Intuitive API
- Multilingual support
- Return rich information for each type

### 5. **Integrated Default Content**
- Provide professional HTML content ready for the most common documents
- Customizable general templates for other types
- Dynamic dates and consistent formatting

## 📊 Statistics

| Item | Count |
|--------|--------|
| Total Document Types (ref_type) | 60 types |
| Number of Categories (document types) | 8 categories |
| E-commerce Documents | 10 documents |
| Apps & Rental Documents | 8 documents |
| Orders & Contracts Documents | 8 documents |
| Users & Registration Documents | 8 documents |
| Employment Documents | 5 documents |
| Security Documents | 5 documents |
| General Documents | 8 documents |
| Industry Documents | 8 documents |
| Process Types (process_type) | 4 types |
| Operation Types (operation_type) | 2 types |

## 🏗️ Organizational Structure

### Main Categories:
1. **E-commerce** – `TYPE_ECOMMERCE`
2. **Rental** – `TYPE_RENTAL`
3. **Contracts** – `TYPE_CONTRACT`
4. **Users** – `TYPE_USER`
5. **Employment** – `TYPE_EMPLOYMENT`
6. **Security** – `TYPE_SECURITY`
7. **General** – `TYPE_GENERAL`
8. **Industry** – `TYPE_INDUSTRY`

## 🛠️ Usage Methods

### 1. **Basic Usage**

```php
use Nano3\Legal\Classes\DocumentTypes;

// Get all types (as a simple array)
$allTypes = DocumentTypes::getAllTypes();

// Get types categorized by category
$categorizedTypes = DocumentTypes::getAllTypes(true);

// Get types by a specific category
$ecommerceTypes = DocumentTypes::getTypesByCategory(DocumentTypes::TYPE_ECOMMERCE);
```

### 2. **Dropdown Lists in Forms**

```php
// In FieldsOptions trait
public function getRefTypeOptions($value = null, $formData = null)
{
    return DocumentTypes::getRefTypeOptionsForSelect();
}

// Grouped by categories
public function getRefTypeOptionsGrouped($value = null, $formData = null)
{
    return DocumentTypes::getRefTypeOptionsForSelect(true);
}

// Document category options
public function getDocumentTypeOptions($value = null, $formData = null)
{
    return DocumentTypes::getTypeOptionsForSelect();
}
```

### 3. **Validation and Analytics**

```php
// Validate document type (ref_type)
if (DocumentTypes::isValidRefType('privacy_policy')) {
    // Type is valid
}

// Validate category (type)
if (DocumentTypes::isValidType('ecommerce')) {
    // Category is valid
}

// Get complete information about a document type
$typeInfo = DocumentTypes::getRefTypeInfo('privacy_policy');
// Result: ['code' => 'privacy_policy', 'name' => 'Privacy Policy', 'type' => 'ecommerce', ...]

// Get the category to which a type belongs
$category = DocumentTypes::getCategoryByRefType('privacy_policy');

// Get essential types (most used)
$essentialTypes = DocumentTypes::getEssentialRefTypes();
```

### 4. **Usage in Views**

```blade
<!-- In a document type selection form grouped by category -->
<select name="ref_type">
    @foreach(DocumentTypes::getRefTypeOptionsForSelect(true) as $category => $types)
        <optgroup label="{{ $category }}">
            @foreach($types as $value => $label)
                <option value="{{ $value }}">{{ $label }}</option>
            @endforeach
        </optgroup>
    @endforeach
</select>

<!-- Simple list -->
<select name="ref_type">
    @foreach(DocumentTypes::getAllTypes() as $value => $label)
        <option value="{{ $value }}">{{ $label }}</option>
    @endforeach
</select>
```

### 5. **Usage with Consents**

```php
// Get process types for consents
$processTypes = DocumentTypes::getProcessTypes();
// Result: ['add' => 'Add', 'remove' => 'Remove', ...]

// Get operation types
$operationTypes = DocumentTypes::getOperationTypes();
// Result: ['manual' => 'Manual', 'auto' => 'Automatic']
```

### 6. **Default Document Content**

```php
// Get default content for a specific type (latest version)
$content = DocumentTypes::getDefaultContentByRefType('privacy_policy');

// Get general content for any other type
$generalContent = DocumentTypes::getGeneralDocumentContent('some_other_type');

// Get a short description from language files
$description = DocumentTypes::getDefaultDescriptionsByRefType('privacy_policy', 'en');
```

## 📝 Available Constants

### Basic Document Types (REF_TYPE_*):
```php
// E-commerce
DocumentTypes::REF_TYPE_PRIVACY_POLICY          // Privacy Policy
DocumentTypes::REF_TYPE_TERMS_OF_SERVICE        // Terms of Service
DocumentTypes::REF_TYPE_RETURN_POLICY           // Return Policy
DocumentTypes::REF_TYPE_SHIPPING_POLICY         // Shipping Policy
DocumentTypes::REF_TYPE_CANCELLATION_POLICY     // Cancellation Policy
DocumentTypes::REF_TYPE_WARRANTY_POLICY         // Warranty Policy
DocumentTypes::REF_TYPE_PAYMENT_TERMS           // Payment Terms
DocumentTypes::REF_TYPE_COOKIE_POLICY           // Cookie Policy
DocumentTypes::REF_TYPE_AFFILIATE_AGREEMENT     // Affiliate Agreement
DocumentTypes::REF_TYPE_PARTNERSHIP_AGREEMENT   // Partnership Agreement

// ... and so on for other categories
```

### Document Categories (TYPE_*):
```php
DocumentTypes::TYPE_ECOMMERCE    // E-commerce
DocumentTypes::TYPE_RENTAL       // Rental
DocumentTypes::TYPE_CONTRACT     // Contracts
DocumentTypes::TYPE_USER         // Users
DocumentTypes::TYPE_EMPLOYMENT   // Employment
DocumentTypes::TYPE_SECURITY     // Security
DocumentTypes::TYPE_GENERAL      // General
DocumentTypes::TYPE_INDUSTRY     // Industry
```

### Process Types (PROCESS_TYPE_*):
```php
DocumentTypes::PROCESS_TYPE_ADD      // Add
DocumentTypes::PROCESS_TYPE_REMOVE   // Remove
DocumentTypes::PROCESS_TYPE_OUT      // Out
DocumentTypes::PROCESS_TYPE_RESET    // Reset
// (Note: PROCESS_TYPE_LEVEL_UP and PROCESS_TYPE_WINNER exist but are commented out)
```

### Operation Types (OPERATION_TYPE_*):
```php
DocumentTypes::OPERATION_TYPE_MANUAL  // Manual
DocumentTypes::OPERATION_TYPE_AUTO    // Automatic
```

## 🔧 Available Methods

| Method | Description | Example |
|---------|-------------|---------|
| `getAllRefTypes($includeCategories = false)` | All document types (ref_type) | `DocumentTypes::getAllRefTypes(true)` |
| `getAllTypes($includeCategories = false)` | Alias for getAllRefTypes | `DocumentTypes::getAllTypes()` |
| `findRefType($id, $locale = null)` | Find label for a specific type | `DocumentTypes::findRefType('privacy_policy')` |
| `getAllTypesOptions($locale = null, $is_key_val = false)` | Options for all types | `DocumentTypes::getAllTypesOptions()` |
| `groupByCategories($refTypes)` | Group types by category | `DocumentTypes::groupByCategories($types)` |
| `getTypesByCategory($category)` | Types by category | `DocumentTypes::getTypesByCategory('ecommerce')` |
| `getDocumentTypes()` | All document categories | `DocumentTypes::getDocumentTypes()` |
| `findDocumentType($id, $locale = null)` | Find category label | `DocumentTypes::findDocumentType('ecommerce')` |
| `getAllDocumentTypes($locale = null, $is_key_val = false)` | All categories | `DocumentTypes::getAllDocumentTypes()` |
| `getCategoryName($category)` | Translated category name | `DocumentTypes::getCategoryName('ecommerce')` |
| `getCategoryOptions()` | Category options for dropdowns | `DocumentTypes::getCategoryOptions()` |
| `getProcessTypes()` | Process types for consents | `DocumentTypes::getProcessTypes()` |
| `getOperationTypes()` | Operation types | `DocumentTypes::getOperationTypes()` |
| `getRefTypeInfo($refType)` | Complete info about a document type | `DocumentTypes::getRefTypeInfo('privacy_policy')` |
| `getRefTypeConstantName($refType)` | Constant name for the type | `DocumentTypes::getRefTypeConstantName('privacy_policy')` |
| `getCategoryByRefType($refType)` | Category the type belongs to | `DocumentTypes::getCategoryByRefType('privacy_policy')` |
| `getEssentialTypes()` | Essential types (old) | `DocumentTypes::getEssentialTypes()` |
| `getEssentialRefTypes()` | Essential types (updated) | `DocumentTypes::getEssentialRefTypes()` |
| `getEssentialRefTypesOptions($locale = null, $is_key_val = false)` | Options for essential types | `DocumentTypes::getEssentialRefTypesOptions()` |
| `isValidRefType($refType)` | Validate document type | `DocumentTypes::isValidRefType('privacy_policy')` |
| `isValidType($type)` | Validate category | `DocumentTypes::isValidType('ecommerce')` |
| `getRefTypeOptionsForSelect($groupByCategory = false)` | Options for dropdowns | `DocumentTypes::getRefTypeOptionsForSelect(true)` |
| `getTypeOptionsForSelect()` | Category options | `DocumentTypes::getTypeOptionsForSelect()` |
| `getProcessTypeOptionsForSelect()` | Process type options | `DocumentTypes::getProcessTypeOptionsForSelect()` |
| `getOperationTypeOptionsForSelect()` | Operation type options | `DocumentTypes::getOperationTypeOptionsForSelect()` |
| `getRefTypesByCategory($category)` | Document types by category | `DocumentTypes::getRefTypesByCategory('ecommerce')` |
| `getOptionsForSelect($groupByCategory = false)` | Alias for getRefTypeOptionsForSelect | `DocumentTypes::getOptionsForSelect(true)` |
| `getRefTypesCount()` | Number of document types | `DocumentTypes::getRefTypesCount()` |
| `getConstantName($ref_type)` | Constant name | `DocumentTypes::getConstantName('privacy_policy')` |
| `getDefaultDescriptionsByRefType($refType, $locale = null, $defaultValue = null)` | Default short description | `DocumentTypes::getDefaultDescriptionsByRefType('privacy_policy', 'en')` |
| `getDefaultDescriptionsLangByRefType($refType, $locale = null, $defaultValue = null)` | Description in a specific language | `DocumentTypes::getDefaultDescriptionsLangByRefType('privacy_policy', 'en')` |
| `getDefaultConditionsByRefType($refType)` | Default brief conditions | `DocumentTypes::getDefaultConditionsByRefType('privacy_policy')` |

### Default Content Methods (from DefaultContentDocumentTypes trait)

| Method | Description |
|---------|-------------|
| `getDefaultContentByRefTypeV1($refType)` | Old default content (basic) |
| `getDefaultContentByRefType($refType)` | Updated default content (professional) |
| `getPrivacyPolicyContent()` | Privacy Policy content |
| `getTermsOfServiceContent()` | Terms of Service content |
| `getReturnPolicyContent()` | Return Policy content |
| `getShippingPolicyContent()` | Shipping Policy content |
| `getUserAgreementContent()` | User Agreement content |
| `getCookiePolicyContent()` | Cookie Policy content |
| `getGeneralDocumentContent($refType)` | General content for any other type |

## 🚀 Practical Use Cases

### 1. **E-commerce System**
```php
// When creating a new document
$ecommerceDocs = DocumentTypes::getTypesByCategory(DocumentTypes::TYPE_ECOMMERCE);
// Result: Privacy Policy, Terms of Service, Return Policy, etc. (10 documents)

// Create a document with default content
$document = new Document;
$document->ref_type = DocumentTypes::REF_TYPE_PRIVACY_POLICY;
$document->content = DocumentTypes::getDefaultContentByRefType(DocumentTypes::REF_TYPE_PRIVACY_POLICY);
$document->save();
```

### 2. **Contract Management System**
```php
// Display available contract types
$contractDocs = DocumentTypes::getTypesByCategory(DocumentTypes::TYPE_CONTRACT);
// Result: Purchase Agreement, Sales Agreement, Invoice Terms, etc. (8 documents)

// Validate uploaded contract
if (DocumentTypes::isValidRefType($input->ref_type)) {
    // Process contract
}
```

### 3. **User Management System**
```php
// User registration documents
$userDocs = DocumentTypes::getTypesByCategory(DocumentTypes::TYPE_USER);
// Result: User Agreement, Registration Terms, Code of Conduct, etc. (8 documents)

// Show only essential types for a new user
$essentialDocs = DocumentTypes::getEssentialRefTypesOptions();
```

### 4. **Consents System**
```php
// Create a new consent record
$consent = new Consent;
$consent->ref_type = DocumentTypes::REF_TYPE_PRIVACY_POLICY;
$consent->process_type = DocumentTypes::PROCESS_TYPE_ADD; // Add
$consent->operation_type = DocumentTypes::OPERATION_TYPE_MANUAL; // Manual
$consent->save();
```

### 5. **Export Analytical Reports**
```php
public function getDocumentStats()
{
    $stats = [];
    $categories = DocumentTypes::getDocumentTypes();
    
    foreach (array_keys($categories) as $category) {
        $stats[$category] = [
            'category_name' => DocumentTypes::getCategoryName($category),
            'count' => count(DocumentTypes::getRefTypesByCategory($category)),
            'types' => DocumentTypes::getRefTypesByCategory($category)
        ];
    }
    
    return $stats;
}
```

## 📈 Performance Benefits

1. **Reduce Database Queries**: No need for a `SELECT DISTINCT` query every time
2. **Improve Memory**: Cache types in memory
3. **Response Speed**: Instant data retrieval
4. **Error Reduction**: Using constants prevents typos

## 🔄 How to Add New Types

### Step 1: Add the constant in `DocumentTypes.php`
```php
const REF_TYPE_NEW_DOCUMENT = 'new_document';
```

### Step 2: Add the translation in the language file (e.g., `lang/en/lang.php`)
```php
'document_types' => [
    'ref_types' => [
        // ... existing types
        'new_document' => 'New Document',
    ],
    'descriptions' => [
        // ... existing descriptions
        'new_document' => 'Brief description of the new document',
    ],
],
```

### Step 3: Add to the appropriate category in the `groupByCategories` method
```php
self::TYPE_GENERAL => [
    // ... existing types
    self::REF_TYPE_NEW_DOCUMENT => $types[self::REF_TYPE_NEW_DOCUMENT],
],
```

### Step 4: (Optional) Add default content in `DefaultContentDocumentTypes`
```php
public static function getDefaultContentByRefType($refType)
{
    $defaultContent = [
        // ... existing types
        self::REF_TYPE_NEW_DOCUMENT => self::getNewDocumentContent(),
    ];
    // ...
}

public static function getNewDocumentContent()
{
    return '<h1>New Document</h1>...';
}
```

## 🧪 Testing and Validation

```php
class DocumentTypesTest extends \TestCase
{
    public function testAllTypesCount()
    {
        $types = DocumentTypes::getAllRefTypes();
        $this->assertCount(60, $types); // 60 types
    }
    
    public function testEssentialTypes()
    {
        $essential = DocumentTypes::getEssentialRefTypes();
        $this->assertContains('privacy_policy', $essential);
        $this->assertContains('terms_of_service', $essential);
    }
    
    public function testTypeInfo()
    {
        $info = DocumentTypes::getRefTypeInfo('privacy_policy');
        $this->assertEquals('Privacy Policy', $info['name']);
        $this->assertEquals('ecommerce', $info['type']);
    }
    
    public function testCategoryName()
    {
        $this->assertEquals('E-commerce', DocumentTypes::getCategoryName('ecommerce'));
    }
    
    public function testValidRefType()
    {
        $this->assertTrue(DocumentTypes::isValidRefType('privacy_policy'));
        $this->assertFalse(DocumentTypes::isValidRefType('invalid_type'));
    }
}
```

## 📚 Advanced Examples

### Example 1: Create a Dynamic Document with Default Content
```php
class DocumentManager
{
    public function createDocument($refType, $title = null)
    {
        if (!DocumentTypes::isValidRefType($refType)) {
            throw new \InvalidArgumentException("Invalid document type: $refType");
        }
        
        $document = new Document;
        $document->ref_type = $refType;
        $document->title = $title ?: DocumentTypes::getRefTypeInfo($refType)['name'];
        $document->content = DocumentTypes::getDefaultContentByRefType($refType);
        $document->type = DocumentTypes::getCategoryByRefType($refType);
        $document->save();
        
        return $document;
    }
}
```

### Example 2: Filter Documents by Category with Advanced Options
```php
class DocumentFilter
{
    public function getOptionsForBusiness($businessType)
    {
        $options = [];
        
        switch ($businessType) {
            case 'ecommerce':
                $categories = [DocumentTypes::TYPE_ECOMMERCE, DocumentTypes::TYPE_CONTRACT];
                break;
            case 'saas':
                $categories = [DocumentTypes::TYPE_RENTAL, DocumentTypes::TYPE_USER, DocumentTypes::TYPE_SECURITY];
                break;
            default:
                $categories = [DocumentTypes::TYPE_GENERAL];
        }
        
        foreach ($categories as $category) {
            $options[DocumentTypes::getCategoryName($category)] = DocumentTypes::getRefTypesByCategory($category);
        }
        
        return $options;
    }
}
```

### Example 3: Integration with Consents System
```php
class ConsentLogger
{
    public function logConsent($userId, $refType, $processType)
    {
        if (!DocumentTypes::isValidRefType($refType)) {
            throw new \Exception("Unknown document type");
        }
        
        if (!array_key_exists($processType, DocumentTypes::getProcessTypes())) {
            throw new \Exception("Invalid process type");
        }
        
        $consent = new Consent;
        $consent->user_id = $userId;
        $consent->ref_type = $refType;
        $consent->process_type = $processType;
        $consent->operation_type = DocumentTypes::OPERATION_TYPE_AUTO;
        $consent->save();
        
        return $consent;
    }
}
```

## 🎨 Best Practices

1. **Always use constants**: `DocumentTypes::REF_TYPE_PRIVACY_POLICY` instead of direct strings
2. **Validate**: `DocumentTypes::isValidRefType($refType)` before processing
3. **Use grouping**: `DocumentTypes::getRefTypeOptionsForSelect(true)` for complex dropdowns
4. **Cache results**: Cache frequently used results (e.g., `getAllRefTypes`)
5. **Regular updates**: Review types and add new ones with proper documentation
6. **Leverage default content**: Use `getDefaultContentByRefType` as a starting point for new documents
7. **Consistent naming**: Adhere to a uniform naming pattern for constants (`REF_TYPE_*`, `TYPE_*`)

## 🏆 Conclusion

`DocumentTypes` is a vital and essential class in the legal document management system, providing:

- **Central system** for managing document types (60 types across 8 categories)
- **Organized structure** that facilitates extension and maintenance
- **Intuitive interface** for developers with many helper methods
- **Excellent performance** with full translation support
- **High flexibility** for different systems (e-commerce, contracts, users, etc.)
- **Ready default content** to speed up development and standardize formatting
- **Support for consents** through various process types

This class is an ideal solution for projects that need comprehensive management of legal documents while maintaining organization and ease of maintenance, and together with the `DefaultContentDocumentTypes` trait, it provides an integrated package for creating and managing legal documents with high efficiency.

---