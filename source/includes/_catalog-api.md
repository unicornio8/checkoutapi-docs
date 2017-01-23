# Catalog API

The Catalog API is the component that clients should use to obtain prices, stock, and additional details on our articles. That information can come in two formats: a catalog feed; article details for a given Config SKU.

This component serves two main purposes: allowing our partners' Customer Facing Apps (mobile or web) to keep their Zalando catalog up to date (we're mostly talking about feeds, but not exclusively); provide the most recent information on stock and prices to fill in during the checkout process.

More information on the description of the different endpoints.

<aside class="notice">
Don't forget to check the API spec!</a>
</aside>

## Getting Feeds

> Sample Request (all feeds)

```shell
curl -v -X GET
 -H "Accept: application/x.zalando.catalog-feeds+json,application/x.problem+json"
 -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
 "https://{Catalog API URL}/api/feeds"
```

> Sample Response

```
[
  {
    "sales_channel": "16a40c35-5ac3-4e39-a3e6-0d0d6047a6ae",
    "catalog_url": "ftp://user:passwd@ftp.server.de/zalando_client_X_de.csv.gz",
    "last_modified": "2016-08-12T10:00:16Z"
  },
  {
    "sales_channel": "16b43f36-5ef9-0a25-3f4b-b00b5007b3de",
    "catalog_url": "ftp://user:passwd@ftp.server.de/zalando_client_X_fr.csv.gz",
    "last_modified": "2016-08-12T10:00:16Z"
  }
]
```

> Sample Request (specific feed)

```shell
curl -v -X GET
 -H "Accept: application/x.zalando.catalog-feeds+json,application/x.problem+json"
 -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
 "https://{Catalog API URL}/api/feeds?sales_channels=16a40c35-5ac3-4e39-a3e6-0d0d6047a6ae"
```

> Sample Response

```json
[
  {
    "sales_channel": "16a40c35-5ac3-4e39-a3e6-0d0d6047a6ae",
    "catalog_url": "ftp://ftp-10393-148801272:8570771f@ftp.semtrack.de/zalando_client_X_de.csv.gz",
    "last_modified": "2016-08-12T10:00:16Z"
  }
]
```

Feed is short for Catalog Feed. Concretely, it is a CSV file with the complete catalog of products for a given Sales Channel assigned to the partner that owns the Sales Channel. This endpoint will not stream or download that CSV file, but will instead provide the link for it.

A Feed is identified by a Sales Channel. So, a partner has the option to either get the Feed for a specific Sales Channel, or obtain all the Feeds for each Sales Channels held by this partner. This gives our partners the option to process those CSV files in whatever way they choose, as the sheer size of your average catalog is not something that should be queried on demand.

### Security

The Feeds endpoint is protected by OAuth2. A Service-To-Service token is required to perform the request.

### Request

`GET /api/feeds?sales_channels={sales-channel-1,sales-channel-2}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
sales_channels | Query | List (comma separated) of sales channels. | No
OAuth token | Header | Service token used to authenticate the request. The token also helps validate that the sales channels in the request belong to the client making the request. | Yes

## Getting Article Details

> Sample Request

```shell
curl -X GET
    -H "Accept: application/x.zalando.articles+json;charset=UTF-8,application/x.problem+json;charset=UTF-8"
    "https://{Catalog API URL}/api/articles/su221d0pi-m11?sales_channel=16b43f36-5ef9-0a25-3f4b-b00b5007b3de&client_id=client_HS23eDa2"
```

> Sample Response

```
[
  {
    "id": "SU221D0PI-M11000S000",
    "size": "S",
    "price": {
      "amount": 19.95,
      "currency": "EUR"
    },
    "original_price": {
      "amount": 19.95,
      "currency": "EUR"
    },
    "available": true,
    "stock": 3,
    "partner": {
      "id": "3293",
      "name": "Superdry",
      "detail_url": "https://www.zalando.de/superdry-top-khaki-twist-su221d0pi-m11.html"
    }
  },
  {
    "id": "SU221D0PI-M11000L000",
    "size": "L",
    "price": {
      "amount": 19.95,
      "currency": "EUR"
    },
    "original_price": {
      "amount": 19.95,
      "currency": "EUR"
    },
    "available": true,
    "stock": 3,
    "partner": {
      "id": "3293",
      "name": "Superdry",
      "detail_url": "https://www.zalando.de/superdry-top-khaki-twist-su221d0pi-m11.html"
    }
  }
]
```

If you used the above endpoint to get the feeds, then you now have a catalog of Zalando products. You will notice that the information about those products is not very complete. It is enough to show to the customer what is available, but it's missing some information, such as: pricing, sizes or updated stock. This is where the Article Details come in.

Using the Config SKU clients can get the aforementioned details on the individual products (Simple SKUs). This is mainly used to show the customer the sizes available when he or she opens the product for more details.

Speaking of the checkout process, this is the endpoint to get an up to date stock availability count, as well as the price to be submitted.

Also note that the Feed is not mandatory to use this endpoint. A Config SKU is what is required, and that can come from different places.

### Security

The Articles endpoint is public. Nevertheless, the origin (partner wise) of the requests is identified by the `client_id` parameter. This allows us to get some control over the endpoint to prevent abuses.

### Request

`GET /api/articles/{config-sku}?sales_channel={sales-channel}&client_id={client-id}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
config_sku | Path | The config SKU of the article. | Yes
client_id | Query | Client ID for the client making the request. Used to track the origin of the request. | Yes
sales_channel | Query | Sales Channel under which the article is being sold. The sales channel gives us the country and currency to show the price. | Yes


## Getting Article Recommendation

> Sample Request

```shell
curl -X GET
    -H "Accept: application/x.zalando.article.recommendation+json;charset=UTF-8,application/x.problem+json;charset=UTF-8"
    "https://{Catalog API URL}/api/articles/lop21ddff-k12/recommendation?sales_channel=16b43f36-5ef9-0a25-3f4b-b00b5007b3de&client_id=client_HS23eDa2"
```

> Sample Response

```
[
  {
    "id": "SU221D0PI-M11000S000",
    "size": "S",
    "price": {
      "amount": 19.95,
      "currency": "EUR"
    },
    "original_price": {
      "amount": 19.95,
      "currency": "EUR"
    },
    "available": true,
    "stock": 3,
    "partner": {
      "id": "3293",
      "name": "Superdry",
      "detail_url": "https://www.zalando.de/superdry-top-khaki-twist-su221d0pi-m11.html"
    }
  },
  {
    "id": "ADB123456-H12000L000",
    "size": "L",
    "price": {
      "amount": 99.88,
      "currency": "EUR"
    },
    "original_price": {
      "amount": 12.95,
      "currency": "EUR"
    },
    "available": true,
    "stock": 3,
    "partner": {
      "id": "3293",
      "name": "Test",
      "detail_url": "https://www.zalando.de/something-ADB123456-H12.html"
    }
  }
]
```

With this call, you will get similar articles, which could then be presented on the product detail page;
thereby, allowing customers to see other products without searching for them.

### Security

The Articles endpoint is public. Nevertheless, the origin (partner wise) of the requests is identified by the `client_id` parameter. This allows us to get some control over the endpoint to prevent abuses.

### Request

`GET /api/articles/{config-sku}/recommendation?sales_channel={sales-channel}&client_id={client-id}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
config_sku | Path | The config SKU of the article. | Yes
client_id | Query | Client ID for the client making the request. Used to track the origin of the request. | Yes
sales_channel | Query | Sales Channel under which the article is being sold. The sales channel gives us the country and currency to show the price. | Yes
