---
title: "How-to: Migrate your application from Form Recognizer v2.1 to v3.0."
titleSuffix: Azure Applied AI Services
description: In this how-to guide, you'll learn the differences between Form Recognizer API v2.1 and v3.0. You'll also learn the changes you need to move to the newer version of the API.
author: vkurpad
manager: nitinme
ms.service: applied-ai-services
ms.subservice: forms-recognizer
ms.topic: how-to
ms.date: 10/07/2021
ms.author: vikurpad
recommendations: false
---

# Form Recognizer v3.0 migration | Preview

> [!IMPORTANT]
>
> Form Recognizer REST API v3.0 introduces breaking changes in the REST API request and analyze response JSON.

Form Recognizer v3.0 (preview) introduces several new features and capabilities:

* [Form Recognizer REST API](quickstarts/try-v3-rest-api.md) has been redesigned for better usability.
* [**General document (v3.0)**](concept-general-document.md) model is a new API that extracts text, tables, structure, key-value pairs, and named entities from forms and documents.
* [**Receipt (v3.0)**](concept-receipt.md) model supports single-page hotel receipt processing.
* [**ID document (v3.0)**](concept-id-document.md) model supports endorsements, restrictions, and vehicle classification extraction from US driver's licenses.
* [**Custom model API (v3.0)**](concept-custom.md) supports signature detection for custom forms.

In this article, you'll learn the differences between Form Recognizer v2.1 and v3.0 and how to move to the newer version of the API.

## Changes to the REST API endpoints

 The v3.0 REST API combines the analysis operations for layout analysis, prebuilt models, and custom models into a single pair of operations by assigning **`documentModels`**  and  **`modelId`** to layout analysis (prebuilt-layout) and prebuilt models.

### POST request

```http
https://{your-form-recognizer-endpoint}/formrecognizer/documentModels/{modelId}?api-version=2021-07-30-preview

```

### GET request

```http
https://{your-form-recognizer-endpoint}/formrecognizer/documentModels/{modelId}/AnalyzeResult/{resultId}?api-version=2021-07-30-preview
```

### Analyze operation

* The request payload and call pattern remain unchanged.
* The Analyze operation specifies the input document and content-specific configurations, it returns the analyze result URL via the Operation-Location header in the response.
* Poll this Analyze Result URL, via a GET request to check the status of the analyze operation (minimum recommended interval between requests is 1 second).
* Upon success, status is set to succeeded and [analyzeResult](#changes-to-analyze-result) is returned in the response body. If errors are encountered, status will be set to failed and an error will be returned.

| Model | v2.1 | v3.0 |
|:--| :--| :--|
| **Request URL prefix**| **https://{your-form-recognizer-endpoint}/formrecognizer/v2.1**  | **https://{your-form-recognizer-endpoint}/formrecognizer** |
|🆕 **General document**|N/A|/documentModels/prebuilt-document:analyze |
| **Layout**| /layout/analyze |/documentModels/prebuilt-layout:analyze|
|**Custom**| /custom/{modelId}/analyze    |/documentModels/{modelId}:analyze |
| **Invoice** | /prebuilt/invoice/analyze    | /documentModels/prebuilt-invoice:analyze |
| **Receipt** | /prebuilt/receipt/analyze    | /documentModels/prebuilt-receipt:analyze |
| **ID document** | /prebuilt/idDocument/analyze |  /documentModels/prebuilt-idDocument:analyze |
|**Business card**| /prebuilt/businessCard/analyze| /documentModels/prebuilt-businessCard:analyze|

### Analyze request body

The content to be analyzed is provided via the request body. Either the URL or base64 encoded data can be user to construct the request.

  To specify a publicly accessible web URL, set  Content-Type to **application/json** and send the following JSON body:

  ```json
  {
    "urlSource": "{urlPath}"
  }
  ```

Base64 encoding is also supported in Form Recognizer v3.0:

```json
{
  "base64Source": "{base64EncodedContent}"
}
```

### Additional parameters

Parameters that continue to be supported:

* pages
* locale

Parameters no longer supported: 

* includeTextDetails

The new response format is more compact and the full output is always returned.

## Changes to analyze result

Analyze response has been refactored to the following top-level results to support multi page elements.
* pages
* tables
* keyValuePairs
* entities
* styles
* documents

> [!NOTE]
>
> The analyzeResult response changes includes a number of changes like moving up from a property of pages to a top lever property within analyzeResult.

```json

{
// Basic analyze result metadata
"apiVersion": "2021-07-30-preview", // REST API version used
"modelId": "prebuilt-invoice", // ModelId used
"stringIndexType": "textElements", // Character unit used for string offsets and lengths:
// textElements, unicodeCodePoint, utf16CodeUnit // Concatenated content in global reading order across pages.
// Words are generally delimited by space, except CJK (Chinese, Japanese, Korean) characters.
// Lines and selection marks are generally delimited by newline character.
// Selection marks are represented in Markdown emoji syntax (:selected:, :unselected:).
"content": "CONTOSO LTD.\nINVOICE\nContoso Headquarters...", "pages": [ // List of pages analyzed
{
// Basic page metadata
"pageNumber": 1, // 1-indexed page number
"angle": 0, // Orientation of content in clockwise direction (degree)
"width": 0, // Page width
"height": 0, // Page height
"unit": "pixel", // Unit for width, height, and bounding box coordinates
"spans": [ // Parts of top-level content covered by page
{
"offset": 0, // Offset in content
"length": 7 // Length in content
}
], // List of words in page
"words": [
{
"text": "CONTOSO", // Equivalent to $.content.Substring(span.offset, span.length)
"boundingBox": [ ... ], // Position in page
"confidence": 0.99, // Extraction confidence
"span": { ... } // Part of top-level content covered by word
}, ...
], // List of selectionMarks in page
"selectionMarks": [
{
"state": "selected", // Selection state: selected, unselected
"boundingBox": [ ... ], // Position in page
"confidence": 0.95, // Extraction confidence
"span": { ... } // Part of top-level content covered by selection mark
}, ...
], // List of lines in page
"lines": [
{
"content": "CONTOSO LTD.", // Concatenated content of line (may contain both words and selectionMarks)
"boundingBox": [ ... ], // Position in page
"spans": [ ... ], // Parts of top-level content covered by line
}, ...
]
}, ...
], // List of extracted tables
"tables": [
{
"rowCount": 1, // Number of rows in table
"columnCount": 1, // Number of columns in table
"boundingRegions": [ // Bounding boxes potentially across pages covered by table
{
"pageNumber": 1, // 1-indexed page number
"boundingBox": [ ... ], // Bounding box
}
],
"spans": [ ... ], // Parts of top-level content covered by table // List of cells in table
"cells": [
{
"kind": "stub", // Cell kind: content (default), rowHeader, columnHeader, stub, description
"rowIndex": 0, // 0-indexed row position of cell
"columnIndex": 0, // 0-indexed column position of cell
"rowSpan": 1, // Number of rows spanned by cell (default=1)
"columnSpan": 1, // Number of columns spanned by cell (default=1)
"content": "SALESPERSON", // Concatenated content of cell
"boundingRegions": [ ... ], // Bounding regions covered by cell
"spans": [ ... ] // Parts of top-level content covered by cell
}, ...
]
}, ...
], // List of extracted key-value pairs
"keyValuePairs": [
{
"key": { // Extracted key
"content": "INVOICE:", // Key content
"boundingRegions": [ ... ], // Key bounding regions
"spans": [ ... ] // Key spans
},
"value": { // Extracted value corresponding to key, if any
"content": "INV-100", // Value content
"boundingRegions": [ ... ], // Value bounding regions
"spans": [ ... ] // Value spans
},
"confidence": 0.95 // Extraction confidence
}, ...
], // List of extracted entities
"entities": [
{
"category": "DateTime", // Primary entity category
"subCategory": "Date", // Secondary entity category
"content": "11/15/2019", // Entity content
"boundingRegions": [ ... ], // Entity bounding regions
"spans": [ ... ], // Entity spans
"confidence": 0.99 // Extraction confidence
}, ...
], // List of extracted styles
"styles": [
{
"isHandwritten": true, // Is content in this style handwritten?
"spans": [ ... ], // Spans covered by this style
"confidence": 0.95 // Detection confidence
}, ...
], // List of extracted documents
"documents": [
{
"docType": "prebuilt-invoice", // Classified document type (model dependent)
"boundingRegions": [ ... ], // Document bounding regions
"spans": [ ... ], // Document spans
"confidence": 0.99, // Document splitting/classification confidence // List of extracted fields
"fields": {
"VendorName": { // Field name (docType dependent)
"type": "string", // Field value type: string, number, array, object, ...
"valueString": "CONTOSO LTD.",// Normalized field value
"content": "CONTOSO LTD.", // Raw extracted field content
"boundingRegions": [ ... ], // Field bounding regions
"spans": [ ... ], // Field spans
"confidence": 0.99 // Extraction confidence
}, ...
}
}, ...
]
}



```

## Build or Train model

The model object has two updates in the new API
* ```modelId``` is now a property that can be set on a model for a human readable name.
* ```modelName``` has been renamed to ```description```

The ```build``` operation is invoked to train a model. The request payload and call pattern remain unchanged. The build operation specifies the model and training dataset, it returns the result via the Operation-Location header in the response. Poll this model operation URL, via a GET request to check the status of the build operation (minimum recommended interval between requests is 1 second). Unlike v2.1, this URL is not the resource location of the model. Instead, the model URL can be constructed from the given modelId, also retrieved from the resourceLocation property in the response. Upon success, status is set to ```succeeded``` and result contains the custom model info. If errors are encountered, status is set to ```failed``` and the error is returned.

The following code is a sample build request using a SAS token. Note the trailing slash when setting the prefix or folder path.

```json
POST https://{your-form-recognizer-endpoint}/formrecognizer/documentModels:build?api-version=2021-09-30-preview

{
  "modelId": {modelId},
  "description": "Sample model",
  "azureBlobSource": {
    "containerUrl": "https://{storageAccount}.blob.core.windows.net/{containerName}?{sasToken}",
    "prefix": "{folderName/}"
  }
}
```
## Changes to compose model

Model compose is now limited to single level of nesting. Composed models are now consistent with custom models with the addition of ```modelId``` and ```description``` properties.

```json
POST https://{your-form-recognizer-endpoint}/formrecognizer/documentModels:compose?api-version=2021-09-30-preview
{
  "modelId": "{composedModelId}",
  "description": "{composedModelDescription}",
  "componentModels": [
    { "modelId": "{modelId1}" },
    { "modelId": "{modelId2}" },
  ]
}

```

## Changes to copy model

The call pattern for copy model remains unchanged:

* Authorize the copy operation with the target resource calling ```authorizeCopy```. Now a POST request.
* Submit the authorization to the source resource to copy the model calling ```copy-to```
* Poll the returned operation to validate the operation completed successfully

The only changes to the copy model function are:

* HTTP action on the ```authorizeCopy``` is now a POST request.
* The authorization payload contains all the information needed to submit the copy request.

Authorize the copy.
```json
POST https://{targetHost}/formrecognizer/documentModels:authorizeCopy?api-version=2021-09-30-preview
{
  "modelId": "{targetModelId}",
  "description": "{targetModelDescription}",
}
```
Use the response body from the authorize action to construct the request for the copy.

```json
POST https://{sourceHost}/formrecognizer/documentModels/{sourceModelId}:copy-to?api-version=2021-09-30-preview
{
  "targetResourceId": "{targetResourceId}",
  "targetResourceRegion": "{targetResourceRegion}",
  "targetModelId": "{targetModelId}",
  "targetModelLocation": "https://{targetHost}/formrecognizer/documentModels/{targetModelId}",
  "accessToken": "{accessToken}",
  "expirationDateTime": "2021-08-02T03:56:11Z"
}
```

## Changes to List models

List models have been extended to now return prebuilt and custom models. All prebuilt model names start with ```prebuilt-```. Only models with a status of succeeded are returned. To list models that either failed or are in progress, see [List Operations](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v3-0-preview-1/operations/GetModels).

Sample list models request

```json
GET https://{your-form-recognizer-endpoint}/formrecognizer/documentModels?api-version=2021-09-30-preview
```

## Change to get model

As get model now includes prebuilt models, the get operation returns a ```docTypes``` dictionary. Each document type is described by its name, optional description, field schema, and optional field confidence. The field schema describes the list of fields potentially returned with the document type.

```json
GET https://{your-form-recognizer-endpoint}/formrecognizer/documentModels/{modelId}?api-version=2021-09-30-preview
```

## New get info operation
The ```info``` operation on the service returns the custom model count and custom model limit.

```json
GET https://{your-form-recognizer-endpoint}/formrecognizer/info? api-version=2021-09-30-preview
```
Sample response
```json
{
  "customDocumentModels": {
    "count": 5,
    "limit": 100
  }
}
```

## Next steps

In this migration guide, you've learned how to upgrade your existing Form Recognizer application to use the v3.0 APIs. Continue to use the 2.1 API for all GA features and use the 3.0 API for any of the preview features.

* [Review the new REST API](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v3-0-preview-1/operations/AnalyzeDocument)
* [What is Form Recognizer?](overview.md)
* [Form Recognizer quickstart](./quickstarts/try-sdk-rest-api.md)