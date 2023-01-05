# PHP SDK for ImageKit

[![PHP CI](https://github.com/imagekit-developer/imagekit-php/workflows/PHP%20CI/badge.svg)](https://github.com/imagekit-developer/imagekit-php/) [![Packagist](https://img.shields.io/packagist/v/imagekit/imagekit.svg)](https://packagist.org/packages/imagekit/imagekit) [![Packagist](https://img.shields.io/packagist/dt/imagekit/imagekit.svg)](https://packagist.org/packages/imagekit/imagekit) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Twitter Follow](https://img.shields.io/twitter/follow/imagekitio?label=Follow&style=social)](https://twitter.com/ImagekitIo)

PHP SDK for [ImageKit](https://imagekit.io/) that implements the new APIs and interface for performing different file
operations.

ImageKit is a complete image optimization and transformation solution that comes with an
[image CDN](https://imagekit.io/features/imagekit-infrastructure) and media storage. It can be integrated with your existing infrastructure - storage like AWS S3, web servers, your CDN, and custom domain names, allowing you to deliver optimize images in minutes with minimal code changes.

Table of contents -

-   [Installation](#Installation)
-   [Initialization](#Initialization)
-   [URL Generation](#URL-generation)
-   [File Upload](#File-upload)
-   [File Management](#File-management)
-   [Utility Functions](#Utility-functions)
-   [Sample Code Instruction](#Sample-Code-Instruction)
-   [Support](#Support)
-   [Links](#Links)

## Installation

Go to your terminal and type the following command

```sh
composer require imagekit/imagekit
```

## Initialization

```
use ImageKit\ImageKit;

$imageKit = new ImageKit(
    "your_public_key",
    "your_private_key",
    "your_url_endpoint"
);
```

## Usage

You can use this PHP SDK for 3 different kinds of methods - URL generation, file upload, and file management. The usage of the SDK has been explained below

## URL generation

**1. Using Image path and image hostname or endpoint**

This method allows you to create a URL using the path where the image exists and the URL
endpoint(url_endpoint) you want to use to access the image. You can refer to the documentation
[here](https://docs.imagekit.io/integration/url-endpoints) to read more about URL endpoints
in ImageKit and the section about [image origins](https://docs.imagekit.io/integration/configure-origin) to understand about paths with different kinds of origins.

```
$imageURL = $imageKit->url(array(
    "path" => "/default-image.jpg",
    "transformation" => array(
        array(
            "height" => "300",
            "width" => "400",
        )
    )
));
```

The result in a URL like

```
https://ik.imagekit.io/your_imagekit_id/endpoint/tr:h-300,w-400/default-image.jpg
```

**2.Using full image URL**
This method allows you to add transformation parameters to and existing, complete URL that is already mapped to ImageKit
using `src` parameter. This method should be used if you have the complete image URL mapped to ImageKit stored in your
database.

```
$imageURL = $imageKit->url(array(
    "src" => "https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg",
    "transformation" => array(
        array(
            "height" => "300",
            "width" => "400",
        )
    )
));

```

This results in a URL like

```
https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg?tr=h-300%2Cw-400
```

The `$imageKit->url()` method accepts the following parameters

| Option                | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| :-------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| urlEndpoint           | Optional. The base URL to be appended before the path of the image. If not specified, the URL Endpoint specified at the time of SDK initialization is used. For example, https://ik.imagekit.io/your_imagekit_id/endpoint/                                                                                                                                                                                                                                                                                                                                                               |
| path                  | Conditional. This is the path at which the image exists. For example, `/path/to/image.jpg`. Either the `path` or `src` parameter needs to be specified for URL generation.                                                                                                                                                                                                                                                                                                                                                                                                                |
| src                   | Conditional. This is the complete URL of an image already mapped to ImageKit. For example, `https://ik.imagekit.io/your_imagekit_id/endpoint/path/to/image.jpg`. Either the `path` or `src` parameter needs to be specified for URL generation.                                                                                                                                                                                                                                                                                                                                           |
| transformation        | Optional. An array of objects specifying the transformation to be applied in the URL. The transformation name and the value should be specified as a key-value pair in the object. Different steps of a [chained transformation](https://docs.imagekit.io/features/image-transformations/chained-transformations) can be specified as different objects of the array. The complete list of supported transformations in the SDK and some examples of using them are given later. If you use a transformation name that is not specified in the SDK, it gets applied as it is in the URL. |
| transformationPosition | Optional. The default value is `path` that places the transformation string as a path parameter in the URL. It can also be specified as `query` which adds the transformation string as the query parameter `tr` in the URL. If you use `src` parameter to create the URL, then the transformation string is always added as a query parameter.                                                                                                                                                                                                                                                 |
| queryParameters       | Optional. These are the other query parameters that you want to add to the final URL. These can be any query parameters and not necessarily related to ImageKit. Especially useful if you want to add some versioning parameter to your URLs.                                                                                                                                                                                                                                                                                                                                           |
| signed                | Optional. Boolean. The default value is `false`. If set to `true`, the SDK generates a signed image URL adding the image signature to the image URL. This can only be used if you are creating the URL with the `urlEndpoint` and `path` parameters, and not with the `src` parameter.                                                                                                                                                                                                                                                                                                             |
| expireSeconds         | Optional. Integer. Meant to be used along with the `signed` parameter to specify the time in seconds from now when the URL should expire. If specified, the URL contains the expiry timestamp in the URL and the image signature is modified accordingly.                                                                                                                                                                                                                                                                                                                                |

#### Examples of generating URLs

**1. Chained Transformations as a query parameter**

```
$imageURL = $imageKit->url(array(
    "path" => "/default-image.jpg",
    "url_endpoint" => "https://ik.imagekit.io/your_imagekit_id/endpoint/",
    "transformation" => array(
        array(
            "height" => "300",
            "width" => "400",
        ),
        array(
            "rotation" => 90
        ),
    ),
    "transformationPosition" => "query"
));
```

```
https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg?tr=h-300%2Cw-400%3Art-90
```

**2. Sharpening and contrast transforms with a progressive JPG image**

There are some transforms like [Sharpening](https://docs.imagekit.io/features/image-transformations/image-enhancement-and-color-manipulation) that can be added to the URL with or without any other value. To use such transforms without specifying a value, specify the value as "-" in the transformation object, otherwise, specify the value that you want to be added to this transformation.

```
$imageURL = $imageKit->url(array(
    "src" => "https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg",
    "transformation" => array(
        array(
            "format" => "jpg",
            "progressive" => true,
            "effectSharpen" => "-",
            "effectContrast" => "1"
        )
    )
));
```

```
//Note that because `src` parameter was used, the transformation string gets added as a query parameter `tr`
https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg?tr=f-jpg%2Cpr-true%2Ce-sharpen%2Ce-contrast-1
```

**3. Signed URL that expires in 300 seconds with the default URL endpoint and other query parameters**

```
 $imageURL = $imageKit->url(array(
    "path" => "/default-image.jpg",
    "queryParameters" => array(
        "v" => "123",
    ),
    "transformation" => array(
        array(
            "height" => "300",
            "width" => "400"
        ),
    ),
    "signed" => true,
    "expireSeconds" => 300,
));
```

```
https://ik.imagekit.io/your_imagekit_id/tr:h-300,w-400/default-image.jpg?v=123&ik-t=1567358667&ik-s=f2c7cdacbe7707b71a83d49cf1c6110e3d701054
```

#### List of supported transformations

The complete list of transformations supported and their usage in ImageKit can be found [here](https://docs.imagekit.io/features/image-transformations). The SDK gives a name to each transformation parameter, making the code simpler and readable. If a transformation is supported in ImageKit, but a name for it cannot be found in the table below, then use the transformation code from ImageKit docs as the name when using in the `url` function.

| Supported Transformation Name | Translates to parameter |
| ----------------------------- | ----------------------- |
| height                        | h                       |
| width                         | w                       |
| aspectRatio                   | ar                      |
| quality                       | q                       |
| crop                          | c                       |
| cropMode                      | cm                      |
| x                             | x                       |
| y                             | y                       |
| focus                         | fo                      |
| format                        | f                       |
| radius                        | r                       |
| background                    | bg                      |
| border                        | bo                      |
| rotation                      | rt                      |
| blur                          | bl                      |
| named                         | n                       |
| overlayImage                  | oi                      |
| overlayX                      | ox                      |
| overlayY                      | oy                      |
| overlayFocus                  | ofo                     |
| overlayHeight                 | oh                      |
| overlayWidth                  | ow                      |
| overlayText                   | ot                      |
| overlayTextFontSize           | ots                     |
| overlayTextFontFamily         | otf                     |
| overlayTextColor              | otc                     |
| overlayAlpha                  | oa                      |
| overlayTextTypography         | ott                     |
| overlayBackground             | obg                     |
| overlayImageTrim              | oit                     |
| progressive                   | pr                      |
| lossless                      | lo                      |
| trim                          | t                       |
| metadata                      | md                      |
| colorProfile                  | cp                      |
| defaultImage                  | di                      |
| dpr                           | dpr                     |
| effectSharpen                 | e-sharpen               |
| effectUSM                     | e-usm                   |
| effectContrast                | e-contrast              |
| effectGray                    | e-grayscale             |
| original                      | orig                    |

## File Upload

The SDK provides a simple interface using the `$imageKit->upload()` method to upload files to the ImageKit Media Library. It accepts all the parameters supported by the [ImageKit Upload API](https://docs.imagekit.io/api-reference/upload-file-api).

The `upload()` method requires at least the `file` and the `fileName` parameter to upload a file and returns a callback with the `error` and `result` as arguments. You can pass other parameters supported by the ImageKit upload API using the same parameter name as specified in the upload API documentation. For example, to specify tags for a file at the time of upload use the `tags` parameter as specified in the [documentation here](https://docs.imagekit.io/api-reference/upload-file-api/server-side-file-upload).

Sample usage

```
$imageKit->uploadFiles(array(
    "file" => "your_file", // required
    "fileName" => "your_file_name.jpg", // required
    "useUniqueFileName" => false, // optional
    "tags" => array("tag1","tag2"), // optional
    "folder" => "images/folder/", // optional
    "isPrivateFile" => false, // optional
    "customCoordinates" => "10,10,100,100", // optional
    "responseFields" => "tags,customCoordinates" // optional
));
```

If the upload succeed, `error` will be `null` and the `result` will be the same as what is received from ImageKit's servers.
If the upload fails, `error` will be the same as what is received from ImageKit's servers and the `result` will be null.

## File Management

The SDK provides a simple interface for all the [media APIs mentioned here](https://docs.imagekit.io/api-reference/media-api) to manage your files. You can use a callback function with all API interfaces. The first argument of the callback function is the error and the second is the result of the API call. The error will be `null` if the API succeeds.

**1. List & Search Files**

Accepts an object specifying the parameters to be used to list and search files. All parameters specified in the [documentation here](https://docs.imagekit.io/api-reference/media-api/list-and-search-files) can be passed as is with the correct values to get the results.

```
$imageKit->listFiles(
    array(
        "skip" => 10,
        "limit" => 10
    )
);
```

**2. Get File Details**

Accepts the file ID and fetches the details as per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/get-file-details).

```
$imageKit->getFileDetails("file_id");
```

**3. Get File Metadata**

Accepts the file ID and fetches the metadata as per the [API documentation here](https://docs.imagekit.io/api-reference/metadata-api/get-image-metadata-for-uploaded-media-files).

```
 $imageKit->getFileMetaData("file_id");
```

**4. Get File Metadata from remote URL**

Accepts the file ID and fetches image exif, pHash and other metadata from ImageKit.io powered remote URL as per the [API documentation here](https://docs.imagekit.io/api-reference/metadata-api/get-image-metadata-from-remote-url).

```
$imageKit->getFileMetadataFromRemoteURL("imagekit_remote_url")
```

**5. Update File Details**

Update parameters associated with the file as per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/update-file-details). The first argument to the `updateFileDetails` method is the file ID and the second argument is an object with the parameters to be updated.

```
$imageKit->updateFileDetails("file_id", array("tags" => array("image_tag")));
```

**6. Delete File**

Delete a file as per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/delete-file). The method accepts the file ID of the file that has to be deleted.

```
$imageKit->deleteFile("file_id");
```

**7. Delete Bulk Files by ID**

Deletes multiple files and all their transformations as per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/delete-files-bulk). The method accepts the array of file IDs that has to be deleted.

```
$imageKit->bulkFileDeleteByIds(array(
    "fileIds" => array("file_id_1", "file_id_2", ...)
));
```

**8. Purge Cache**

Programmatically issue a cache clear request as per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/purge-cache). Accepts the full URL of the file for which the cache has to be cleared.

```
$imageKit->purgeFileCacheApi("file_url");
```

**9. Purge Cache Status**

Get the purge cache request status using the request ID returned when a purge cache request gets submitted as per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/purge-cache-status)

```
$imageKit->purgeFileCacheApiStatus("cache_request_id");
```

## Utility functions

We have included following commonly used utility functions in this SDK.

### Authentication parameter generation

In case you are looking to implement client-side file upload, you are going to need a `token`, `expiry` timestamp and a valid `signature` for that upload. The SDK provides a simple method that you can use in your code to generate these authentication parameters for you.

_Note: The Private API Key should never be exposed in any client-side code. You must always generate these authentication parameters on the server-side_

```
$imageKit->getAuthenticationParameters($token = "", $expire = 0);
```

Returns

```
array(
    "token" => "unique_token",
    "expire" => "valid_expiry_timestamp",
    "signature" => "generated_signature",
);
```

Both the `token` and `expire` parameters are optional. If not specified, the SDK generates a random token and also generates a valid expiry timestamp internally. The value of the `token` and `expire` used to generate the signature are always returned in the response, no matter if they are provided as an input to this method or not.

### Distance calculation between two pHash values

Perceptual hashing allows you to construct a hash value that uniquely identifies an input image based on the contents of an image. [ImageKit.io metadata API](https://docs.imagekit.io/api-reference/metadata-api) returns the pHash value of an image in the response. You can use this value to find a duplicate (or similar) image by calculating the distance between pHash value of two images.

This SDK exposes `pHashDistance` function to calculate the distance between two pHash values. It accepts two pHash hexadecimal strings and returns a numeric value indicative of the level of difference between the two images.

```
  $imageKit->pHashDistance($firstHash ,$secondHash);
```

#### Distance calculation examples

```
$imageKit->pHashDistance('f06830ca9f1e3e90', 'f06830ca9f1e3e90');
// output: 0 (same image)

$imageKit->pHashDistance('2d5ad3936d2e015b', '2d6ed293db36a4fb');
// output: 17 (similar images)

$imageKit->pHashDistance('a4a65595ac94518b', '7838873e791f8400');
// output: 37 (dissimilar images)
```

## Sample Code Instruction

To run sample code go to the sample directory and run

```
php sample.php
```

## Support

For any feedback or to report any issues or general implementation support, please reach out to [support@imagekit.io](mailto:support@imagekit.io)

## Links

-   [Documentation](https://docs.imagekit.io)
-   [Main website](https://imagekit.io)

## License

Released under the MIT license.
