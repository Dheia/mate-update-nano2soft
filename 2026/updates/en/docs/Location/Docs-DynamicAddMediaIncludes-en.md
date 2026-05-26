# Documentation for `DynamicAddMediaIncludes` Behavior – `Nano.LocationApi` Plugin

## Overview

`DynamicAddMediaIncludes` is a custom Behavior designed to dynamically add multimedia relationships to any `Nano\API\Classes\Transformer`, especially those related to geographic models such as Country, State, and Directorate.

This Behavior aims to provide a unified and reusable way to include images, image galleries, videos, audio recordings, general files, and introductory files in API responses, with full support for controlling the return of metadata for each file.

**Path:** `plugins/nano/locationapi/behaviors/DynamicAddMediaIncludes.php`

---

## Purpose & Benefits

- **Unify media inclusion** across all geographic transformers.
- **Reduce code duplication** by providing a single Behavior that can be attached to any transformer.
- **Dynamically support includes** so developers can request them only when needed (`include=image,images`).
- **Control metadata** per file, reducing unnecessary response size.
- **Safely handle exceptions** and return default values if the relationship does not exist or an error occurs.
- **Compatible with `Nano.API` architecture** and uses the `image()`, `images()`, `file()`, `files()` methods from the base transformer.

---

## Structure

### Namespace and Inheritance

```php
namespace Nano\LocationApi\Behaviors;

use October\Rain\Extension\ExtensionBase;
use Nano\API\Classes\Transformer;
use League\Fractal\Resource\Primitive;
use League\Fractal\Resource\Item;
use League\Fractal\Resource\Collection;
```

- The Behavior extends `ExtensionBase` to be compatible with October CMS’s Extendable system.
- It depends on the base `Transformer` to provide file conversion methods.

### Properties

| Property | Type | Description |
| :--- | :--- | :--- |
| `$transformer` | `Transformer` | Reference to the transformer to which the Behavior is attached. |
| `$isMateData` | `bool` | (Internal) Determines whether metadata is required. |

### Methods

#### `__construct(Transformer $transformer)`

- **Description:** Constructor. Automatically adds media includes to `availableIncludes` in the transformer, and optionally to `defaultIncludes` based on configuration.
- **Parameters:**
  - `$transformer`: The associated transformer object.
- **Details:**
  - Checks if `availableIncludes` exists and adds `image`, `images`, `videos`, `audios`, `files`, `book_intro` if not already present.
  - If the configuration setting `nano.locationapi::default_media_includes` is enabled, also adds these includes to `defaultIncludes`.

#### `shouldReturnMateData(): bool`

- **Description:** Determines whether metadata should be returned globally.
- **Mechanism:**
  - Checks for the existence of the `is_mate_data_media` parameter in the Request and returns its value.
  - If not present, returns the value of the configuration setting `nano.locationapi::is_mate_data_media`.
- **Return:** `true` if metadata is requested, otherwise `false`.

#### `getMateDataForInclude(string $includeName): bool`

- **Description:** Determines whether metadata should be returned for a specific include, with the ability to override the global setting.
- **Parameters:**
  - `$includeName`: The name of the include (e.g., `image`, `files`).
- **Mechanism:**
  - Checks for a request parameter with the same name as the include containing `mate_data` (e.g., `image[mate_data]=1`).
  - If present, returns its value.
  - Otherwise returns the value of `shouldReturnMateData()`.
- **Return:** `true` if metadata is requested for this specific include, otherwise `false`.

---

#### Main Include Methods

Each of the following methods represents an include that can be requested via the `include` parameter in the API. All return either a `Primitive` (for single items) or `Collection` (for multiple items).

| Method | Include | Resource Type | Expected relationship on model | Default value on error |
| :--- | :--- | :--- | :--- | :--- |
| `includeImage($model)` | `image` | `Primitive` | `attachOne 'image'` | `null` |
| `includeImages($model)` | `images` | `Collection` | `attachMany 'images'` | `[]` |
| `includeVideos($model)` | `videos` | `Collection` | `attachMany 'videos'` | `[]` |
| `includeAudios($model)` | `audios` | `Collection` | `attachMany 'audios'` | `[]` |
| `includeFiles($model)` | `files` | `Collection` | `attachMany 'files'` | `[]` |
| `includeBookIntro($model)` | `book_intro` | `Primitive` | `attachOne 'book_intro'` | `null` |

**How each method works:**

1. Checks that `$model` is valid and that the relationship method exists (e.g., `method_exists($model, 'image')`).
2. Calls the appropriate method from the base transformer (`$this->transformer->image()`, `$this->transformer->images()`, `$this->transformer->file()`, or `$this->transformer->files()`).
3. Passes the `$isMate` parameter obtained from `getMateDataForInclude()`.
4. In case of any exception/throwable, logs it (only in development environment) and returns the safe default value.

---

## Integration with Transformers

To use this Behavior in a specific transformer, attach it via the `extend` method as in the following example:

```php
use Nano\LocationApi\Behaviors\DynamicAddMediaIncludes;

class CountryTransformer extends Transformer
{
    public $implement = [
        DynamicAddMediaIncludes::class,
    ];
    
    // ... rest of the code
}
```

Or via dynamic extension inside `Plugin.php` (as implemented in `Nano.LocationApi`):

```php
protected function extendLocationApiTransformersWithMedia()
{
    $transformers = [
        \Nano\LocationApi\Transformers\CountryTransformer::class,
        \Nano\LocationApi\Transformers\StateTransformer::class,
        \Nano\LocationApi\Transformers\DirectorateTransformer::class,
    ];

    foreach ($transformers as $transformerClass) {
        $transformerClass::extend(function ($transformer) {
            if (!$transformer->isClassExtendedWith('Nano\LocationApi\Behaviors\DynamicAddMediaIncludes')) {
                $transformer->extendClassWith('Nano\LocationApi\Behaviors\DynamicAddMediaIncludes');
            }
        });
    }
}
```

---

## Configuration

The Behavior’s behaviour can be customised via the plugin’s configuration file `config/nano/locationapi/config.php` (published using `php artisan config:publish Nano.LocationApi`).

| Key | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `is_mate_data_media` | `bool` | `false` | Return metadata by default when not explicitly requested. |
| `default_media_includes` | `array|false` | `false` | If an array, these includes are added to `defaultIncludes` in the transformer. Example: `['image', 'images']`. |

**Example configuration file:**

```php
<?php

return [
    'is_mate_data_media' => false,
    'default_media_includes' => ['image'], // include the main image by default
];
```

---

## Usage Examples

### 1. Request a Country with only the Main Image

**Request:**
```http
GET /api/v1/countries/1?include=image
```

**Response:**
```json
{
  "data": {
    "id": 1,
    "name": "Yemen",
    "image": {
      "original": "https://domain.com/storage/flags/ye.png",
      "small": "https://domain.com/storage/flags/ye_small.png"
    }
  }
}
```

### 2. Request a City with the Gallery and General Metadata

**Request:**
```http
GET /api/v1/states/5?include=images&is_mate_data_media=1
```

**Response:**
```json
{
  "data": {
    "id": 5,
    "name": "Sana'a",
    "images": [
      {
        "original": "https://domain.com/storage/gallery/1.jpg",
        "mate_data": {
          "id": 101,
          "title": "Old City",
          "file_name": "1.jpg",
          "file_size": 204800
        }
      }
    ]
  }
}
```

### 3. Request a Directorate with Videos and Files, Requesting Metadata only for Files

**Request:**
```http
GET /api/v1/directorates/10?include=videos,files&files[mate_data]=1
```

**Response (excerpt):**
```json
{
  "data": {
    "id": 10,
    "name": "Al-Sabeen",
    "videos": [
      {
        "file_name": "intro.mp4",
        "path": "https://domain.com/storage/videos/intro.mp4"
      }
    ],
    "files": [
      {
        "file_name": "report.pdf",
        "path": "https://domain.com/storage/files/report.pdf",
        "mate_data": {
          "id": 202,
          "title": "Annual Report",
          "file_size": 1048576
        }
      }
    ]
  }
}
```

### 4. Request a Country with All Media (Multiple Includes)

**Request:**
```http
GET /api/v1/countries/1?include=image,images,videos,audios,files,book_intro&is_mate_data_media=1
```

---

## Error Handling

- **Checking for method existence:** Before accessing any relationship, the Behaviour checks whether the method exists on the model (`method_exists`). If not, it returns the default value immediately without attempting to access it.
- **Exception handling:** All include methods are wrapped in `try-catch`. In case of any exception, it is logged (only in development environment) using `trace_log`, and the safe default value is returned.
- **Error logging:** Errors are logged in the trace log when `app.debug = true`, helping debugging during development without affecting the end user.

**Example of a logged error:**
```
DynamicAddMediaIncludes::includeImage - Call to undefined method App\Models\Country::image()
```

---

## Extensibility

### Adding a New Include

To add a new include (e.g., `documents`), follow these steps:

1. Ensure the relationship (`attachMany 'documents'`) exists on the model.
2. Add a new method in the Behavior:

```php
public function includeDocuments($model)
{
    try {
        if (!$model || !method_exists($model, 'documents')) {
            return $this->collection([]);
        }
        $files = $model->documents;
        $isMate = $this->getMateDataForInclude('documents');
        $result = $this->transformer->files($files, $isMate);
        return $this->collection($result);
    } catch (Exception|Throwable $e) {
        $this->logError(__FUNCTION__, $e);
        return $this->collection([]);
    }
}
```

3. Add `'documents'` to the `$mediaIncludes` array in the constructor.

### Overriding Behaviour in a Custom Transformer

You can override any of the include methods in your custom transformer if you need a different format:

```php
class MyCountryTransformer extends CountryTransformer
{
    public function includeImage($model)
    {
        // custom image formatting
        $image = $model->image;
        return $this->primitive([
            'url' => $image ? $image->path : null,
            'caption' => $image ? $image->title : null,
        ]);
    }
}
```

---

## Dependencies

- **Nano.API:** Provides the base transformer (`Nano\API\Classes\Transformer`) and helper methods (`image()`, `images()`, `file()`, `files()`).
- **Nano.Location:** Provides the geographic models that contain the added media relationships.
- **System\Models\File:** The core file management system in October CMS.

---

## Testing

To test the Behavior after attaching it to a specific transformer, you can use the following code in a test environment:

```php
use Nano\LocationApi\Transformers\CountryTransformer;
use Nano\LocationApi\Behaviors\DynamicAddMediaIncludes;

// Create a transformer with the Behavior
$transformer = new CountryTransformer();

// Simulate an API request with a specific include
Input::merge(['include' => 'image', 'is_mate_data_media' => 1]);

// Get a country model that has an image
$country = Country::find(1);

// Call the include method directly
$behavior = new DynamicAddMediaIncludes($transformer);
$result = $behavior->includeImage($country);

// Verify the result
assert($result instanceof Primitive);
assert(!is_null($result->getData()));
```

---

## Summary

`DynamicAddMediaIncludes` is a powerful and flexible Behavior that simplifies the integration of multimedia into geographic APIs. Thanks to its modular design and safe exception handling, it can be relied upon in production while ensuring stable responses even when models are incomplete. It follows the same architectural pattern as `DynamicAddIncludeKyc` from the `Nano3.Kyc` plugin, ensuring code consistency across NanoSoft projects.

---

**References:**
- [Nano.LocationApi Documentation](./Docs-Nano-LocationApi-en.md)
- [Nano.Location Documentation](./Docs-Nano-Location-en.md)
- [October CMS Extendable System](https://docs.octobercms.com/3.x/extend/classes/extendable.html)
- [`System\Models\File` Documentation](https://docs.octobercms.com/3.x/api/system/file.html)

---

**Last updated:** 2025-04-30  
**Version:** 1.1.0  
**Author:** Nano2Soft Team

---
