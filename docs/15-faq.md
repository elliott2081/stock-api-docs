# Stock API Frequently Asked Questions

<!-- MarkdownTOC -->

- [General](#general)
    - [What thumbnail preview sizes are available?](#what-thumbnail-preview-sizes-are-available)
    - [How do I download a comp image?](#how-do-i-download-a-comp-image)
- [Licensing](#licensing)
    - [How do I add license references?](#how-do-i-add-license-references)
- [Print on Demand](#print-on-demand)
    - [How do you license assets more than once?](#how-do-you-license-assets-more-than-once)
    - [Why do I see Premium and Video in my search results if I don't have credits?](#why-do-i-see-premium-and-video-in-my-search-results-if-i-dont-have-credits)
    - [How do I filter out Premium content?](#how-do-i-filter-out-premium-content)
    - [How do I filter for high-resolution images only?](#how-do-i-filter-for-high-resolution-images-only)
    - [What type of image quota do I have?](#what-type-of-image-quota-do-i-have)

<!-- /MarkdownTOC -->

<a id="general"></a>
## General

<a id="what-thumbnail-preview-sizes-are-available"></a>
### What thumbnail preview sizes are available?

<ul>
<li><code>110</code>: Small (110 px)
<li><code>160</code>: Medium (160 px)
<li><code>240</code>: Large (240 px)
<li><code>500</code>: Extra large (XL) (500 px). Returned with watermark. (default)
<li><code>1000</code>: Extra-extra large (XXL) (1000 px). Returned with watermark.</li>
</ul>

See [Search API reference](api/11-search-reference.md).

<a id="how-do-i-download-a-comp-image"></a>
### How do I download a comp image?
There are two kinds of preview images available: cached thumbnail images from the CDN, and non-cached comp images which need to be downloaded from the API. The first type of images are most common, and recommended for most applications. This is a sample URL:
[https://t4.ftcdn.net/jpg/00/84/66/63/240_F_84666330_LoeYCZ5LCobNwWePKbykqEfdQOZ6fipq.jpg](https://t4.ftcdn.net/jpg/00/84/66/63/240_F_84666330_LoeYCZ5LCobNwWePKbykqEfdQOZ6fipq.jpg)

For best performance, use this type of image when possible. In some circumstances, however, you may need the "comp" image version instead. This image requires a different workflow. First you must get the URL from the API, and then download it, often with an authentication header.

- Get comp URL from media ID
```
  GET /Rest/Media/1/Search/Files?search_parameters[media_id]=143738171&result_columns[]=comp_url HTTP/1.1
  Host: stock.adobe.io
  X-Product: MySampleApp/1.0
  x-api-key: MyApiKey
  Authorization: Bearer MyAccessToken
```

- Result
```json
"files": [
    {
        "comp_url": "https://stock.adobe.com/Rest/Libraries/Watermarked/Download/143738171/5"
    }
```

- Curl download request for comp image
```shell
curl "https://stock.adobe.com/Rest/Libraries/Watermarked/Download/143738171/5" \
  -H "x-api-key: YourApiKeyHere" \
  -H "x-product: MySampleApp/1.0" \
  -H "authorization: Bearer AccessTokenHere"
```

<a id="licensing"></a>
## Licensing

<a id="how-do-i-add-license-references"></a>
### How do I add license references?
License references are extra metadata that you can add to a license record at the time you license the asset. They can be used to track the customer, purchase order, project code, etc., and can be made mandatory or optional. If mandatory, you _must_ include those fields when licensing the asset or you will receive an error.

There are two steps involved:
1. Get a list of required and optional fields from a Member/Profile request
2. Send a POST request to Content/License, including the fields as JSON

In step 1, you call Member/Profile as described in [Licensing assets and stuff](getting-started/apps/06-licensing-assets.md)

```http
  GET /Rest/Libraries/1/Member/Profile?content_id=172563501&license=Standard HTTP/1.1
  Host: stock.adobe.io
  X-Product: MySampleApp/1.0
  x-api-key: MyApiKey
  Authorization: Bearer MyAccessToken
```

If license references have been enabled, the response will include a `cce_agency` array. In the example below, `id:2` ("project reference") is required, while `id:4` ("client reference") is not. Therefore, you must at a minimum include a value for `id:2`.

```json
    "cce_agency": [
        {
            "id": 2,
            "text": "Enter project reference...",
            "required": true
        },
        {
            "id": 4,
            "text": "Enter client reference...",
            "required": false
        }
    ]
```

Instead of calling `GET` Content/License, your application will `POST` Content/License, and set the content type to `application/json`. The body of the message will include your license reference array as shown below.

```http
  POST /Rest/Libraries/1/Content/License?content_id=172563501 HTTP/1.1
  Host: stock.adobe.io
  Content-Type: application/json
  X-Product: MySampleApp
  x-api-key: <API KEY>
  Authorization: Bearer <TOKEN>

  {
    "cce_agency": [
      { "id": "2", "value": "Project Banana" },
      { "id": "4", "value": "King Kong Co" }
    ]
  }
```

To learn how to add or edit license reference fields, see [Edit a product profile for Adobe Stock](https://helpx.adobe.com/enterprise/using/adobe-stock-enterprise.html#CreateeditaproductprofileforAdobeStock).

<a id="print-on-demand"></a>
## Print on Demand

<a id="how-do-you-license-assets-more-than-once"></a>
### How do you license assets more than once?
Or, "if my contract requires me to license the same asset again, will this happen automatically in the API?"
No, currently you must set your application to use the `license_again=true` flag. Best practice in this case is to use this flag _every time_ you license an asset, even if it has not been licensed before. The Stock API will only deduct one license from your pool of credits, even if this is the first time you are licensing this asset.

```shell
curl "https://stock.adobe.io/Rest/Libraries/1/Content/License?content_id=112670342&license=Standard&license_again=true" \
  -H "x-api-key: YourApiKeyHere" \
  -H "x-product: MySampleApp/1.0" \
  -H "authorization: Bearer AccessTokenHere"
```

If using the Stock SDK for PHP, add `license_again` to the request object.

```php
    $license_request = new LicenseRequest();
    $license_request->setLicenseState('STANDARD');
    $license_request->setContentId(112670342);
    $license_request->license_again = true;
    $adobe_stock_client = new AdobeStock($api_key, $app_name, 'PROD', $http_client);
    $license_response = $adobe_stock_client->getContentLicense($license_request, $access_token);
```

<a id="why-do-i-see-premium-and-video-in-my-search-results-if-i-dont-have-credits"></a>
### Why do I see Premium and Video in my search results if I don't have credits?
Premium assets are included in search results by default. In fact, the default API search includes _all asset types_ (except Editorial--see below). Further, to increase performance, search requests are designed to be anonymous, not requiring authentication. Therefore it is not possible for the Search API to know all your contract details and entitlements when you perform a search--this would slow down the search. Therefore, if you don't have rights to these assets and don't want to see them in search results, you will need to filter them out. See next question, below.

<a id="how-do-i-filter-out-premium-content"></a>
### How do I filter out Premium content?
The Search API includes all asset types by default, therefore you must (1) tell the API not to include Premium assets, and (2) tell it what kind of assets you _do_ want to include. Keep in mind that Video, Templates, and 3D are not considered "Premium," and therefore would still appear in the results.

The following request will exclude Premium assets (`[premium]=false`), and limit results to photos, vectors and illustrations.

```http
GET /Rest/Media/1/Search/Files?locale=en_US
&search_parameters[filters][premium]=false&search_parameters[filters][content_type:photo]=1&search_parameters[filters][content_type:illustration]=1&search_parameters[filters][content_type:vector]=1 HTTP/1.1
Host: stock.adobe.io
X-Product: MySampleApp/1.0
X-API-Key: YourApiKeyHere
```

See [Search API reference](api/11-search-reference.md).

<a id="how-do-i-filter-for-high-resolution-images-only"></a>
### How do I filter for high-resolution images only?
All Adobe Stock images _should_ be high resolution by default. Per the Stock Contributor [content requirements](https://helpx.adobe.com/stock/contributor/help/photography-illustrations.html), the minimum image resolution is 4MP (megapixels). However, if you need to guarantee a minimum resolution you can do this using the `area_pixels` search filter. 

The first value is the minimum number of MP you require, and the optional second parameter is the maximum MP.

```
search_parameters[filters][area_pixels] = min int [- max int]
```

To search on images that are at least 25MP (e.g., 5000x5000), use:
```
search_parameters[filters][area_pixels]=25000000
```

In addition, you can use the `orientation` filter to approximate an aspect ratio.

```
search_parameters[filters][orientation] = horizontal | vertical | square | all
```

In the example below, search for all photographs of hippos that have a horizontal/landscape orientation, and have a minimum pixel area of 20 megapixels. For example, this will yield images 5000x4000 or 10000x2000, but _not_ 4000x5000 or 2000x10000.

```http
GET /Rest/Media/1/Search/Files?local=en_US&search_parameters[words]=hippos& search_parameters[filters][content_type:photo]=1&search_parameters[filters][orientation]=horizontal&search_parameters[filters][area_pixels]=20000000 HTTP/1.1
Host: stock.adobe.io
X-Product: MySampleApp/1.0
X-API-Key: YourApiKeyHere
```

For more details, see [Search API reference](api/11-search-reference.md).

<a id="what-type-of-image-quota-do-i-have"></a>
### What type of image quota do I have?
This question only applies to POD Small and Medium Businesses (SMB) customers, not to Enterprise customers.

In the SMB model for Print on Demand, Individual and Team customers must purchase credit packs _instead of_ image subscriptions, and use one credit per image printed. Furthermore, if they already have a Stock image subscription on their account, they must not combine both quota types on the same account; their POD account must use credit packs only. If they combine both types, Stock will automatically deduct image licenses from a subscription instead of the credit pack--there is no way to override this behavior.

The two types of quota are easy to distinguish in the Adobe Stock web UI, but are slightly more confusing as displayed in the Stock API. In the Stock web UI, image quota is displayed as "Images," and credit quota as "Credits." As noted above, SMB customers must only have credits to use Stock for Print on Demand.

In the examples below, only user #3 has the proper type of quota, because user #1 has an image subscription only, and user #2 has both a subscription and credit pack.

![Sample Stock quota types](images/web_pod-quota-types.png)

When using the Stock API, the quota type will be returned by the [Member/Profile API](api/12-licensing-reference.md) as part of the JSON object `available_entitlement.full_entitlement_quota`.

```JavaScript
"available_entitlement": {
    "quota": 0,         <== Image subscription quota (ignore)
    "license_type_id": 1,
    "has_credit_model": false,
    "has_agency_model": false,
    "is_cce": false,
    "full_entitlement_quota": {
       ...               <== Quota types listed here (see below)
    }
},
```

Individual and Team customers can see two types of `full_entitlement_quota`:
- `image_quota`: The available images available in an image subscription.
- `individual_universal_credits_quota`: The available credits available from a credit pack.

The top-level `quota` attribute can be ignored, as it only applies to image subscriptions in this case and is not applicable to Print on Demand.

Using the previous screenshot example, the `Member/Profile` responses would look like this when returned by the API:

**BAD** (image subscription only)
```JavaScript
"available_entitlement": {
    "quota": 3,
    "license_type_id": 1,
    "has_credit_model": false,
    "has_agency_model": false,
    "is_cce": false,
    "full_entitlement_quota": {
        "image_quota": 3
    }
},
```

**BAD** (image subscriptions + credit pack in same account)
```JavaScript
"available_entitlement": {
    "quota": 10,
    "license_type_id": 1,
    "has_credit_model": false,
    "has_agency_model": false,
    "is_cce": false,
    "full_entitlement_quota": {
        "image_quota": 10,
        "individual_universal_credits_quota": 40
    }
},
```

**GOOD** (credit pack only)
```JavaScript
"available_entitlement": {
    "quota": 0,
    "license_type_id": 5,
    "has_credit_model": false,
    "has_agency_model": false,
    "is_cce": false,
    "full_entitlement_quota": {
        "individual_universal_credits_quota": 93
    }
},
```

If you are building an integration that allows the POD user to sign in, your application must check that the user only has `individual_universal_credits_quota`, or else licensing cannot happen for printed goods.