# Checkout API

This is the component that allows partners to share the Zalando shopping experience in their apps or websites. With it partners have everything that they need to create a Zalando order, without the customer navigating away from the place where they became inspired.

This component provides endpoints that grant its clients the ability to:

 * Manage addresses
 * Verify addresses
 * Manage carts
 * Manage checkouts
 * Create and read orders

Everything is in place to give to our customers all the possibilities of the Zalando shop (coupons, different shipping options, different destinations). Even if some of those feature are not available at the moment, adding them should not include any breaking changes to the API.

More information on the description of the different endpoints.

<aside class="notice">
Don't forget to check the API spec!</a>
</aside>

## Happy Path

One of the goals of team Atlas is to provide the most streamlined shopping experience possible. Shopping should be about getting what you want, and not about filling in some forms. With that in mind we make it possible to create an order with only 3 requests:

 1. [Create a cart](#creating-a-cart)
 2. [Create a checkout](#creating-a-checkout)
 3. [Create an order](#creating-an-order)

The above steps are, of course, from the API's perspective. It excludes the browsing and adding of individual articles to the cart. It also assumes that the customer has a valid token. Otherwise he or she would have to login.

What makes this flow possible is having some default data for addresses and payment method. If a customer already set his or her default address for billing and shipping, then no address selection or input is required. The same is valid for payment. Especially if we're talking about Invoice or Pre-payment, that do not require further inputs (Credit Card can ask for an extra token, even if all details were already submitted).

After choosing the article(s), a customer needs only to confirm the default data to create the order. And that is it!

Let's take a look at a more visual example of the Happy Path.

### Press the 'Buy Now' button

<img src="images/press-buy-button.png" >

### Select your size

<img src="images/select-size.png" >

### Confirm default selections

<img src="images/confirm-checkout.png" >

### Sit back and wait for your order to be delivered

<img src="images/order-confirmed.png" >

## Long Path (aka Complete Checkout Process)

The flow described in the above section is the ideal one. As was mentioned, some default selections need to be there already. For active Zalando customers this should all be in place already. For new customers this won't be the case. A customer that just registered won't have any defaults and will have to submit all of the required data. For those, here is the expected path:

 1. Register a new account
 2. [Login](#customer-authentication)
 3. Give Consent
 4. [Create a cart](#creating-a-cart)
 5. [Create an address](#create-address) (for simplicity let's create one that is valid for billing and shipping)
 6. [Create a checkout](#creating-a-checkout)
 7. [Select your payment method](#payment) (depending on the payment method you can have more steps)
 8. [Create an order](#creating-an-order)

A lot more work compared with the previous flow, right? Fortunately, after this first experience the customer can expect the [happy path](#happy-path) in subsequent visits to the app or website.

Let's take a look at a more visual example of the Happy Path.

### Press the 'Buy Now' button

<img src="images/press-buy-button.png" >

### Select your size

<img src="images/select-size.png" >

### Connect with Zalando

<img src="images/no-selections.png" >

### Login

<img src="images/login.png" >

### Or register

<img src="images/register-customer.png" >

### Give consent

<img src="images/give-consent.png" >

### Input addresses

<img src="images/input-address.png" >

### Select payment method

<img src="images/select-payment-method.png" >

### Confirm selections

<img src="images/confirm-checkout.png" >

### Sit back and wait for your order to be delivered

<img src="images/order-confirmed.png" >

## Address Operations

Below we have the different operations that can be done on Addresses, which are basically CRUD operations. All of these operations have their place in the checkout process.

### Security

All Address endpoints are protected by OAuth2. A Customer token is required to perform the requests. The token will also identify the customer performing the requests.

## Reading addresses

> Sample Request

```shell
curl -X GET
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Accept: application/x.zalando.customer.addresses+json; application/x.problem+json"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    "https://{Checkout API URL}/api/addresses?client_id=client_HS23eDa2"
```

> Sample Response

```json
[
  {
    "id": "1",
    "customer_number": "1",
    "gender": "MALE",
    "first_name": "John",
    "last_name": "Doe",
    "street": "Mollstr. 1",
    "additional": "EG",
    "zip": "10178",
    "city": "Berlin",
    "country_code": "DE",
    "pack_station": false,
    "default_billing": true,
    "default_shipping": false
  }
]
```

This endpoint retrieves addresses that belong to the customer that owns the OAuth2 token sent with the request. The addresses are also filtered with the help of the sales channel ID. That makes the endpoint return only the addresses that are in the same countries the sales channel is good for. For example, if the sales channel is only for France, then the response will contain only French addresses.
An empty array is returned if there are no addresses for that customer, in the countries specified by the sales channel.

This endpoint is mostly useful during checkout when the customer would like to select one of its existing addresses to set as billing or shipping.

### Request

`GET /api/addresses`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. This is relevant to obtain addresses filtered by the countries the sales channel operates in. | Yes
OAuth token | Header | Customer token used to authenticate the request. It will also identify the customer issuing the request, and with it the addresses to return | Yes

## Reading a single address

> Sample Request

```shell
curl -X GET
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Accept: application/x.zalando.customer.address+json; application/x.problem+json"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    "https://{Checkout API URL}/api/addresses/1"
```

> Sample Response

```json
{
  "id": 1,
  "customer_number": 1,
  "gender": "MALE",
  "first_name": "John",
  "last_name": "Doe",
  "street": "Mollstr. 1",
  "additional": "EG",
  "zip": 10178,
  "city": "Berlin",
  "country_code": "DE",
  "default_billing": true,
  "default_shipping": false
}
```

This endpoint retrieves a specific address that belongs to the customer that owns the OAuth2 token sent with the request. Address is specified by the address ID. In order for the address to be returned it must share the country with the sales channel specified in the request.

This endpoint will find its use across the checkout process. Whenever an address ID is specified, but its details are not present, this is the endpoint that will be used.

### Request

`GET /api/addresses/{address_id}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. This is relevant to obtain addresses filtered by the countries the sales channel operates in. | Yes
OAuth token | Header | Customer token used to authenticate the request. The token also helps validate that the requested address belongs to the user making the request. | Yes
address_id | Path | ID of the customer address | Yes

## Creating an address <a name="create-address"></a>

> Sample Request

```shell
curl -X POST
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Content-Type: application/x.zalando.customer.address.create+json;charset=UTF-8"
    -H "Accept: application/x.zalando.customer.address.create.response+json;charset=UTF-8, application/x.problem+json;charset=UTF-8"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    -d '{
         "customer_number": 1,
         "gender": "MALE",
         "first_name": "John",
         "last_name": "Doe",
         "street": "Mollstr. 1",
         "additional": "EG",
         "zip": 10178,
         "city": "Berlin",
         "country_code": "DE",
         "default_billing": true,
         "default_shipping": false
       }'
    "https://{Checkout API URL}/api/addresses"
```

> Sample Response

```json
{
  "id": 1,
  "customer_number": 1,
  "gender": "MALE",
  "first_name": "John",
  "last_name": "Doe",
  "street": "Mollstr. 1",
  "additional": "EG",
  "zip": 10178,
  "city": "Berlin",
  "country_code": "DE",
  "default_billing": true,
  "default_shipping": false
}
```

Creating addresses is done with this endpoint. There are some rules that should be made clear when performing this request. First, when an address is created it is not assigned to either billing or shipping, unless the `default_billing` or `default_shipping` flags are set. The addresses of a customer are like an address book that can be used for one purpose or the other.

There are, however, some constraints. A Pick up point type address cannot be used as a billing address. You can see an example Pick up point on the [right](#pick-up-point-example).
Speaking of Pick up points, we currently differentiate them between Packstation, and non-Packstations. The reason for this is that Packstation do not need a Street, whereas the remaining Pick up points do. For a Packstation, the combination of zip code + city + Packstation ID is enough to identify the Packstation.

> Example of a Pick up point address <a name="pick-up-point-example"></a>

```json
{
  "id": 1,
  "customer_number": 1,
  "gender" : "FEMALE",
  "first_name" : "Jim",
  "last_name" : "Bean",
  "zip" : "10178",
  "city" : "Berlin",
  "country_code" : "DE",
  "default_billing": false,
  "default_shipping": true,
  "pickup_point": {
      "name": "PACKSTATION",
      "id": "802",
      "member_id": "111222333"
  }
}
```

Whether because a new customer is going through the checkout process or an existing customer wants to create a new address for billing or shipping, a time will come where addresses will need to be created. That's where this endpoint comes in handy.

### Request

`POST /api/addresses`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. This is relevant to validate whether the submitted address is for a country the sales channel operates in. | Yes
OAuth token | Header | Customer token used to authenticate the request. The token specifies under which user the address will be created. | Yes
Address creation request object | Body | Address details | Yes

## Changing an address

> Sample Request

```shell
curl -X PUT
    -H "Content-Type: application/x.zalando.customer.address.update+json;charset=UTF-8"
    -H "Accept: application/x.zalando.customer.address.update.response+json;charset=UTF-8, application/x.problem+json;charset=UTF-8"
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "If-Match: *"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    -d '{
        "id": "1",
        "customer_number": 1,
        "gender": "FEMALE",
        "first_name": "Jim",
        "last_name": "Bean",
        "street" : "Tamara-Danz-Str. 1",
        "zip" : "10178",
        "city" : "München",
        "country_code": "DE",
        "default_billing": true,
        "default_shipping": true
      }'
    "https://{Checkout API URL}/api/addresses/1"
```

> Sample Response

```json
{
    "id": "1",
    "customer_number": 1,
    "gender": "FEMALE",
    "first_name": "Jim",
    "last_name": "Bean",
    "street" : "Tamara-Danz-Str. 1",
    "zip" : "10178",
    "city" : "München",
    "country_code": "DE",
    "default_billing": true,
    "default_shipping": true
}
```

Except for the request URI, changing addresses is in everything similar to creating them. The same rules apply to build the request body (the address object).

Also like the create address endpoint, they are useful when the customer is picking which addresses to use in the checkout, and needs to make small adjustments to one of them.

### Request

`PUT /api/addresses/{address_id}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. This is relevant to validate whether the submitted address is for a country the sales channel operates in. | Yes
OAuth token | Header | Customer token used to authenticate the request. Used to validate if the address to be changed belongs to the customer issuing the request. | Yes
address_id | Path | ID of the customer address | Yes
Address update request object | Body | Address details | Yes

## Deleting an address

> Sample Request

```shell
curl -X DELETE
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    "https://{Checkout API URL}/api/addresses/1"
```

Although it might seem that this functionality has little use in the checkout process, it was still included with the user experience in mind. The previous endpoints allow a customer to manage his or her address book, to a point. Makes sense to make it feature complete and provide the delete functionality. This way a customer can get rid of some old addresses and make the use of the address book on a mobile screen (for example) much simpler.

### Request

`DELETE /api/addresses/{address_id}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. This is relevant to validate whether the submitted address is for a country the sales channel operates in. | Yes
OAuth token | Header | Customer token used to authenticate the request. Used to validate if the address to be deleted belongs to the customer issuing the request. | Yes
address_id | Path | ID of the customer address | Yes

## Address Check

> Sample Request

```shell
curl -X POST
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Content-Type: application/x.zalando.address-check.create+json"
    -d '{
            "address": {
                "street": "Mollstraße 1",
                "additional": "EG",
                "zip": 10178,
                "city": "Berlin",
                "country_code": "DE"
            }
        }'
    "https://{Checkout API URL}/api/address-checks"
```

> Sample Response

```json
{
  "status": "CORRECT",
  "normalized_address": {
    "street": "Mollstr. 1",
    "additional": "EG",
    "zip": 10178,
    "city": "Berlin",
    "country_code": "DE"
  }
}
```

The last address operation, and the only request that is not directly related to customer addresses, Address Checks are a useful tool to help ensure that Zalando orders reach our customers. Similarly to creating or updating addresses, there are some rules when it comes to Pick up points. Fortunately, they are the same rules as the above operations.

With address checks, one can submit an address for validation. The request will tell you whether the address is properly written - and if it isn't it will try to provide an alternative -, or if it is blacklisted. Suggestions provided by this endpoint are not mandatory. They should be presented to the customer, and suggested to use the alternative as it will increase the chances of getting the order. The reason why the suggestion is not mandatory is because the inputted address can be of a recent building, which is not yet recognized by our services. This operation should be performed with every new address created, or changes to an address.

Some additional information about the status returned in the response object (you can also find it in the API spec):

 * `CORRECT`: Address is correct and valid for use
 * `NORMALIZED`: Address is valid, but the normalized form (field `normalized_address` in the AddressCheckResponse type) should be suggested to the user to safeguard the processing of the order
 * `NOT_CORRECT`: Address was not recognized. User should be notified of this, since it might impact the processing of the order, but the choice to progress should still be given (address might be of a new street or building that is not yet recognised).

### Request

`POST /address-checks`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
OAuth token | Header | Customer token used to authenticate the request. | Yes

## Cart Operations

At the start of the checkout process we have the Cart. This is where the customers will put all the articles that they want to buy. Similar to the Zalando shop experience, a cart can be managed by an unauthenticated customer (aka anonymous cart).

Following operations allow management of a Cart with the exception of deleting the cart, which wouldn't make sense from the point of view of our clients, or customers. A customer only owns a cart per platform, and does not see the carts for the other platforms. For example: Customer Y buys Zalando articles in app X and app Y; the carts for the two apps will be different, and are only available for their respective app.

### Security

All Cart endpoints are protected by OAuth2. But unlike most of the other endpoints, these accept service tokens as well as customer tokens. The reason for this is to allow the creation of an anonymous cart. For example, a customer could browse a catalog without being logged in. When he or she would like to complete the order they could login and continue the checkout process using a customer token. At this point, the items in the anonymous cart would be moved to the cart of that customer. When using a customer token, it will also identify the customer performing the requests. An anonymous cart cannot proceed to the checkout step because it lacks that information.

## Creating a cart

> Sample Request

```shell
curl -X POST
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Accept: application/x.zalando.cart.create.response+json;charset=UTF-8, application/x.problem+json;charset=UTF-8"
    -H "Content-Type: application/x.zalando.cart.create+json;charset=UTF-8"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    -d '{
            "items" : [
                {
                    "sku" : "GU1L13R38-K135784412",
                    "quantity" : 1
                },
                {
                    "sku" : "D123GSAR1-1JL579242",
                    "quantity" : 2
                }
            ],
            "replace_items": true
        }'
    "https://{Checkout API URL}/api/carts"
```

> Sample Response

```json
{
  "id": "b13fce609f084bbaac3e14265b9805dda309d7ca660311e68b7786f30ca893d3",
  "items": [
    {
        "sku" : "D123GSAR1-1JL579242",
        "quantity" : 2
    }
  ],
  "items_out_of_stock": ["GU1L13R38-K135784412"],
  "delivery": {
    "earliest": "2016-08-22T11:52:48.622Z",
    "latest": "2016-08-23T11:52:48.622Z"
  },
  "gross_total": {
    "amount": 90.95,
    "currency": "EUR"
  },
  "tax_total": {
    "amount": 14.52,
    "currency": "EUR"
  }
}
```

This endpoint will provide our Checkout API with the items the customer wishes to buy. It is also here that the prices will be locked for the customer.

The flag `replace_items`, when set to true (its default value), will clear the cart of existing items, if any. If set to false, it will add the items of the request to the ones already present in the cart. A valid use case for this field is when a customer with an anonymous cart logs in, and the API client can merge the items of the anonymous cart with the items that a customer may, or may not, have in his or her cart.

The starting point for any checkout process. Or so it should be used. A client could hold the cart ID and use it in future checkouts by removing all the items from it, but we strongly recommend to always start a checkout with this endpoint. Even if the customer will proceed adding items to cart, and move to the checkout step later.

### Request

`POST /api/carts`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. This is relevant to specify the sales channel under which the cart will be created, and the content ultimately sold. | Yes
OAuth token | Header | Customer token used to authenticate the request. | Yes
Cart creation request object | Body | Specifies the content of the cart, and the sales channel where the items are being sold. | Yes

## Managing items in a cart

> Sample Request

```shell
curl -X PUT
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Accept: application/x.zalando.cart.items.update.response+json;charset=UTF-8, application/x.problem+json;charset=UTF-8"
    -H "Content-Type: application/x.zalando.cart.items.update+json;charset=UTF-8"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    -d '[
            {
                "sku": "D123GSAR1-1JL579242",
                "quantity": "1"
            }
        ]'
    "https://{Checkout API URL}/api/carts/b13fce609f084bbaac3e14265b9805dda309d7ca660311e68b7786f30ca893d3/items"
```

> Sample Response

```json
{
  "id": "b13fce609f084bbaac3e14265b9805dda309d7ca660311e68b7786f30ca893d3",
  "items": [
    {
        "sku" : "D123GSAR1-1JL579242",
        "quantity" : 1
    }
  ],
  "items_out_of_stock": [],
  "delivery": {
    "earliest": "2016-08-22T11:52:48.622Z",
    "latest": "2016-08-23T11:52:48.622Z"
  },
  "gross_total": {
    "amount": 45.95,
    "currency": "EUR"
  },
  "tax_total": {
    "amount": 7.52,
    "currency": "EUR"
  }
}
```

Assuming a typical use of the cart endpoints, the first article added by the customer will trigger the creation of the cart. Subsequent articles will be added with this endpoint.

It is important to notice that the body for this request should include all the items that should be in the cart, and not only the items being added or removed. This may seem like overhead, but it makes it clear at all times (or requests) what is supposed to be the content of the cart.

### Request

`PUT api/carts/{cart_id}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. This is important to validate whether the cart was created under the same sales channel | Yes
OAuth token | Header | Customer token used to authenticate the request. Used to validate if the cart to be changed belongs to the customer issuing the request. | Yes
cart_id | Path | The ID of the Cart | Yes
Cart update request object | Body | Specifies the content of the cart. After creation, only the items can be changed. | Yes

## Getting the cart

> Sample Request

```shell
curl -X GET
    -H "Accept: application/x.zalando.cart+json"
    -H "If-None-Match: *"
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    "https://{Checkout API URL}/api/carts/b13fce609f084bbaac3e14265b9805dda309d7ca660311e68b7786f30ca893d3"
```

> Sample Response

```json
{
  "id": "b13fce609f084bbaac3e14265b9805dda309d7ca660311e68b7786f30ca893d3",
  "items": [
    {
        "sku" : "D123GSAR1-1JL579242",
        "quantity" : 1
    }
  ],
  "items_out_of_stock": [],
  "delivery": {
    "earliest": "2016-08-22T11:52:48.622Z",
    "latest": "2016-08-23T11:52:48.622Z"
  },
  "gross_total": {
    "amount": 45.95,
    "currency": "EUR"
  },
  "tax_total": {
    "amount": 7.52,
    "currency": "EUR"
  }
}
```

This endpoint returns the same information than the ones above. Its usefulness comes from the navigation capabilities it provides to clients of the API. Like with the case of [deleting addresses](#deleting-an-address), it was added to the API with the user experience in mind, as well as the flexibility it provides our partners. For example, a customer could browse the catalog for a bit after adding some articles to the cart, and then initiate the checkout process; at that point this endpoint could be called to open the cart view.

### Request

`GET api/carts/{cart_id}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. This is important to validate whether the cart was created under the same sales channel | Yes
OAuth token | Header | Customer token used to authenticate the request. The token also helps validate that the requested cart belongs to the user making the request. | Yes
cart_id | Path | The ID of the Cart | Yes

## Checkout Operations

At the checkout step, customers will specify the addresses for billing and shipping, delivery options (which for the time being are limited to Standard delivery only), and the payment method. The payment selection, as well as any input regarding payment, is done outside of our components. For more information please check the [Payment section](#payment).

Remaining purpose of the checkout operation is to confirm all the selections done by the customer, and make the necessary adjustments. Said adjustments are limited to the addresses and payment method. Changes to the articles being ordered requires starting all over. Two reasons for this are stock and price verification. Even for articles that were already in the cart, verifying stock and price is required. Not to mention recalculating the total cart value.

Operations below allow for everything mentioned in the above paragraphs, which translates into creation, read, and update of checkouts.

### Security

All Checkout endpoints are protected by OAuth2. A Customer token is required to perform the requests. The token will also identify the customer performing the requests.

## Creating a checkout

> Sample Request

```shell
curl -X POST
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Content-Type: application/x.zalando.customer.checkout.create+json;charset=UTF-8"
    -H "Accept: application/x.zalando.customer.checkout.create.response+json;charset=UTF-8,application/x.problem+json;charset=UTF-8"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    -d '{
            "cart_id":"b13fce609f084bbaac3e14265b9805dda309d7ca660311e68b7786f30ca893d3",
            "shipping_address_id": "1",
            "billing_address_id": "2"
        }'
    "https://{Checkout API URL}/api/checkouts"
```

> Sample Response

```json
{
  "id": "VOJX0xgdfGbuziS6W9i1FUCeBkRWzaH6CAL5T0dEabc",
  "customer_number": 1,
  "cart_id": "b13fce609f084bbaac3e14265b9805dda309d7ca660311e68b7786f30ca893d3",
  "billing_address": {
    "id": 1,
    "gender": "MALE",
    "first_name": "John",
    "last_name": "Doe",
    "street": "Mollstr. 1",
    "additional": "EG",
    "zip": 10178,
    "city": "Berlin",
    "country_code": "DE"
  },
  "shipping_address": {
    "id": 2,
    "gender": "MALE",
    "first_name": "John",
    "last_name": "Doe",
    "zip": 10178,
    "city": "Berlin",
    "country_code": "DE",
    "pickup_point": {
      "name": "PACKSTATION",
      "id": 123,
      "member_id": 12345678
    }
  },
  "delivery": {
    "earliest": "2015-04-21T13:27:31+01:00",
    "latest": "2015-04-23T13:27:31+01:00"
  },
  "payment": {
    "selected": {
      "method": "CREDIT_CARD",
      "metadata": {
        "key_1": "value_1",
        "key_2": "value_2"
      },
      "external_payment": true
    },
    "selection_page_url": "https://{Payment Selection URL}/ae45f19"
  }
}
```

Creation of the checkout object requires only the cart ID and the addresses to be used. And the latter is actually optional. When not present, the default addresses for billing and shipping will be used. As for the articles being checked out, everything is taken from the cart.

Notice that the response also does not include the articles being ordered. You can get that using the endpoint to get the [cart content](#getting-the-cart).

Assuming that everything is OK with the articles and addresses, we are left with the payment step. There are two options here: the customer already has the default payment method selected (look into the `payment` field); customer has no default payment method, or wishes to use a different one. For the first case the checkout process can proceed to order creation. For the second case the customer must be redirected to the payment selection page indicated by the `selection_page_url` field.

The payment selection is done outside of our components. Using the link in the `selection_page_url` field, customers are directed to a page where the selection of the payment method is done. After they're finished, customers will be redirected back to the checkout. This will require a redirect URL provided by the client of our API, which is sent by our component to the payment component. To know which payment method was selected clients of the API need to [get the checkout object](#getting-the-checkout) to get an updated version.

Worth noticing is the `payment.external_payment` flag. When set to true, clients of the API can already provide the customer with the information that they will be redirected to an external page after order creation (for example, for payments with PayPal). More details on payment in the [Payment section](#payment).

### Request

`POST /api/checkouts`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. The cart specified in the body will have to belong to the same sales channel | Yes
X-Forwarded-For | Header | IP of the customer device issuing the request. If not present, it will be taken from the request that our components get. If the request IP does not match the geographical area where the customer is making his or her order, then there is the possibility that risk assessment will raise some red flags and cancel the order | No
OAuth token | Header | Customer token used to authenticate the request. The token also helps validate that the cart and addresses being used belong to the user making the request | Yes
Checkout creation request object | Body | Input that clients can/need to provide for checkout creation | Yes

## Changing the checkout

> Sample Request

```shell
curl -X PUT
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Content-Type: application/x.zalando.customer.checkout.update+json;charset=UTF-8"
    -H "Accept: application/x.zalando.customer.checkout.update.response+json;charset=UTF-8,application/x.problem+json;charset=UTF-8"
    -d '{
            "shipping_address_id": "4",
            "billing_address_id": "3"
        }'
    "https://{Checkout API URL}/api/checkouts/VOJX0xgdfGbuziS6W9i1FUCeBkRWzaH6CAL5T0dEabc"
```

> Sample Response

```json
{
  "id": "VOJX0xgdfGbuziS6W9i1FUCeBkRWzaH6CAL5T0dEabc",
  "customer_number": 1,
  "cart_id": "b13fce609f084bbaac3e14265b9805dda309d7ca660311e68b7786f30ca893d3",
  "billing_address": {
    "id": 3,
    "gender": "MALE",
    "first_name": "John",
    "last_name": "Doe",
    "street": "Tamara-Danz-Str. 1",
    "additional": "EG",
    "zip": 10243,
    "city": "Berlin",
    "country_code": "DE"
  },
  "shipping_address": {
    "id": 4,
    "gender": "MALE",
    "first_name": "John",
    "last_name": "Doe",
    "street": "Neue Bahnhoffstr. 10",
    "zip": 10245,
    "city": "Berlin",
    "country_code": "DE"
  },
  "delivery": {
    "earliest": "2015-04-21T13:27:31+01:00",
    "latest": "2015-04-23T13:27:31+01:00"
  },
  "payment": {
    "selected": {
      "method": "CREDIT_CARD",
      "metadata": {
        "key_1": "value_1",
        "key_2": "value_2"
      },
      "external_payment": true
    },
    "selection_page_url": "https://{Payment Selection URL}/ae45f19"
  }
}
```

Of all the elements of the checkout, addresses are the only ones that can be changed without impact on the overall checkout process. In the future we will support different shipping methods, which will be available for changing after checkout creation.

The response is the same object than in checkout creation, so the same considerations taken for creation are valid here.

### Request

`PUT /api/checkouts/{checkout_id}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. The cart originally used to create the checkout will have to belong to the same sales channel | Yes
X-Forwarded-For | Header | IP of the customer device issuing the request. If not present, it will be taken from the request that our components get. If the request IP does not match the geographical area where the customer is making his or her order, then there is the possibility that risk assessment will raise some red flags and cancel the order | No
OAuth token | Header | Customer token used to authenticate the request. The token also helps validate that the cart and addresses being used belong to the user making the request | Yes
checkout_id | Path | ID of the checkout | Yes
Checkout update request object | Body | Input that clients can change in the checkout | Yes

## Getting the checkout

> Sample Request

```shell
curl -X GET
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Accept: application/x.zalando.customer.checkout+json;charset=UTF-8, application/x.problem+json;charset=UTF-8"
    "https://{Checkout API URL}/api/checkouts/VOJX0xgdfGbuziS6W9i1FUCeBkRWzaH6CAL5T0dEabc"
```

> Sample Response

```json
{
  "id": "VOJX0xgdfGbuziS6W9i1FUCeBkRWzaH6CAL5T0dEabc",
  "customer_number": 1,
  "cart_id": "b13fce609f084bbaac3e14265b9805dda309d7ca660311e68b7786f30ca893d3",
  "billing_address": {
    "id": 3,
    "gender": "MALE",
    "first_name": "John",
    "last_name": "Doe",
    "street": "Tamara-Danz-Str. 1",
    "additional": "EG",
    "zip": 10243,
    "city": "Berlin",
    "country_code": "DE"
  },
  "shipping_address": {
    "id": 4,
    "gender": "MALE",
    "first_name": "John",
    "last_name": "Doe",
    "street": "Neue Bahnhoffstr. 10",
    "zip": 10245,
    "city": "Berlin",
    "country_code": "DE"
  },
  "delivery": {
    "earliest": "2015-04-21T13:27:31+01:00",
    "latest": "2015-04-23T13:27:31+01:00"
  },
  "payment": {
    "selected": {
      "method": "CREDIT_CARD",
      "metadata": {
        "key_1": "value_1",
        "key_2": "value_2"
      },
      "external_payment": true
    },
    "selection_page_url": "https://{Payment Selection URL}/ae45f19"
  }
}
```

All operations done on the checkout object can (and should) be done in the checkout step, so there is no real point in navigating away from the checkout step. Given that, the use of this endpoint can raise some questions. First, checkout and all its components are valid for a short period of time, so it is still possible to recover the current state. This can be useful in case of problems in the web site or app of our partners. Second, it might be necessary to get a new checkout object after the payment step. Payment is done outside of our components, which means that additional information regarding payment is obtained in the checkout response under the `payment` field. For example:

 * A customer has no payment method selected, so there won't be any values in the `payment.selected` field
 * The customer goes to the payment selection (indicated by the `selection_page_url` field) and selects his or her payment method
 * Customer gets back to the checkout view, and by calling this endpoint the selection will then be available in the `payment.selected` field

### Request

`GET /api/checkouts/{checkout_id}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. The cart originally used to create the checkout will have to belong to the same sales channel | Yes
X-Forwarded-For | Header | IP of the customer device issuing the request. If not present, it will be taken from the request that our components get. If the request IP does not match the geographical area where the customer is making his or her order, then there is the possibility that risk assessment will raise some red flags and cancel the order | No
OAuth token | Header | Customer token used to authenticate the request. The token also helps validate that the requested checkout belongs to the user making the request | Yes
checkout_id | Path | ID of the checkout | Yes

## Order operations

We are now entering the last step of the checkout process. The customer already selected everything that was required to create an order, confirmed everything, and pressed the button to create his or her order.

Below we have the operations that allow for this last step, and also some support for it. This translates into creation and read operations.

## Creating an order

> Sample Request

```shell
curl -X POST
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Content-Type: application/x.zalando.customer.order.create+json;charset=UTF-8"
    -d '{
            "checkout_id" : "VOJX0xgdfGbuziS6W9i1FUCeBkRWzaH6CAL5T0dEabc"
        }'
    "https://{Checkout API URL}/api/orders"
```

> Sample Response

```json
{
  "order_number": "10",
  "customer_number": "1",
  "billing_address": {
    "gender": "FEMALE",
    "first_name": "Jim",
    "last_name": "Bean",
    "street": "Tamara-Danz-Str. 1",
    "zip": "10178",
    "city": "München",
    "country_code": "DE"
  },
  "shipping_address": {
    "gender": "FEMALE",
    "first_name": "Jim",
    "last_name": "Bean",
    "street": "Mollstr 1",
    "zip": "10178",
    "city": "Berlin",
    "country_code": "DE"
  },
  "gross_total": {
    "amount": 272.85,
    "currency": "EUR"
  },
  "tax_total": {
    "amount": 43.56,
    "currency": "EUR"
  },
  "created": "2016-08-24T13:29:10.062Z",
  "detail_url": "https://www.zalado.de/benutzerkonto/bestellung-detail/10",
  "external_payment_url": "https://www.example.paypal.com/342hk3jnbsdf"
}
```

There is a lot that is required to create an order. Fortunately, everything is included in the checkout object of the customer, so that is the only parameter that is required to create an order.

Order creation is done after the payment step ([Long Path](#long-path-aka-complete-checkout-process)), or directly after checkout creation ([Happy Path](#happy-path)) if the default payment method exists and is the one that the customer would like to use.

After order creation there might be a final step for payment. Clients of the API already got an hint with the `payment.external_payment` flag, in the checkout response. When that extra step is required, clients get the URL where customers should be redirected to in the order creation response under the `external_payment_url` field. Please note that despite the order has been created, it will remain in a preliminary state until this last payment step is completed.

### Request

`POST /api/orders`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. The checkout passed in the body will have to belong to the same sales channel | Yes
X-Forwarded-For | Header | IP of the customer device issuing the request. If not present, it will be taken from the request that our components get. If the request IP does not match the geographical area where the customer is making his or her order, then there is the possibility that risk assessment will raise some red flags and cancel the order | No
OAuth token | Header | Customer token used to authenticate the request. The token also helps validate that the checkout provided to create the order belongs to the user making the request | Yes
Order creation request object | Body | Input that clients can/need to provide for order creation | Yes

## Getting a list of orders

> Sample Request

```shell
curl -X GET
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Accept: application/x.zalando.customer.orders+json;charset=UTF-8, application/x.problem+json;charset=UTF-8"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    "https://{Checkout API URL}/api/orders?limit=4&created_since=2016-04-16T00:00:00.000Z"
```

> Sample Response

```json
[
  {
    "order_number": "10",
    "customer_number": "1",
    "sales_channel": "16b43f36-5ef9-0a25-3f4b-b00b5007b3de",
    "gross_total": {
      "amount": 272.85,
      "currency": "EUR"
    },
    "created": "2016-08-08T16:23:40.295Z",
    "detail_url": "https://www.zalando.de/benutzerkonto/bestellung-detail/10"
  },
  {
      "order_number": "11",
      "customer_number": "1",
      "sales_channel": "16b43f36-5ef9-0a25-3f4b-b00b5007b3de",
      "gross_total": {
        "amount": 90.95,
        "currency": "EUR"
      },
      "created": "2016-07-08T16:23:40.295Z",
      "detail_url": "https://www.zalando.de/benutzerkonto/bestellung-detail/11"
    }
]
```

Here is another example of an endpoint added to our component with user experience in mind. To allow customers to track what orders they make over the partner's platform we added an endpoint that lists those orders. The endpoint has some filters which can be used by either the partner directly, or by the customer should the partner app extend that option to its users. Those filters are: order creation date (interval start); order creation date (interval end); number of orders returned (limit). The details of each order are kept limited because the idea is just to give an overview of the orders placed by the customer. For a more detailed information on a given order, [there is an endpoint for that](#getting-the-order-details).

Orders returned are limited to those that were made by the client issuing the request. That means that customers using the app from partner X won't see in the return list orders made in the app from partner Y. Or even orders made in Zalando's shop. Orders returned include orders from all the sales channels belonging to the partner.

### Request

`GET /api/orders?limit={limit}&created_since={created_since}&created_until={created_until}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. Only orders created under the same sales channel will be returned | Yes
OAuth token | Header | Customer token used to authenticate the request. The token also helps validate that the requested order belongs to the user making the request. The token also gives information about the API client so that the result is filtered to return the data only if the order was placed via a sales channel of that Zalando partner | Yes
limit | Query | Maximum number of orders returned | No
created_since | Query | Minimum creation date for orders | No
created_until | Query | Maximum creation date for orders | No

## Getting the order details

> Sample Request

```shell
curl -X GET
    -H "Authorization: Bearer eyJraWQiOiJ0ZXN0a2V5LWVzMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9OMlZsWXpZIiwiYXpwIjoic3R5bGlnaHRfTjJWbFl6WSIsInNjb3BlIjpbImF0bGFzLWNhdGFsb2ctYXBpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0NzEwMjQ3NjEsImlhdCI6MTQ3MDk5NTk2MX0.5yrnLBffEHml2nJ1JooQBb08R2MOD-WKGw70ov5zhJpYVZF9jmZh-W45yOohZpq_f-VVb8eMxPC524nZzHkdUg"
    -H "Accept: application/x.zalando.customer.order+json;charset=UTF-8, application/x.problem+json;charset=UTF-8"
    -H "X-Sales-Channel: 16b43f36-5ef9-0a25-3f4b-b00b5007b3de"
    "https://{Checkout API URL}/api/orders/10"
```

> Sample Response

```json
{
  "order_number": "10",
  "customer_number": "1",
  "sales_channel": "16b43f36-5ef9-0a25-3f4b-b00b5007b3de",
  "shipping_address": {
    "gender": "FEMALE",
    "first_name": "Jim",
    "last_name": "Bean",
    "street": "Mollstr 1",
    "zip": "10178",
    "city": "Berlin",
    "country_code": "DE"
  },
  "billing_address": {
    "gender": "FEMALE",
    "first_name": "Jim",
    "last_name": "Bean",
    "street": "Tamara-Danz-Str. 1",
    "zip": "10178",
    "city": "München",
    "country_code": "DE"
  },
  "gross_total": {
    "amount": 90.95,
    "currency": "EUR"
  },
  "tax_total": {
    "amount": 14.52,
    "currency": "EUR"
  },
  "created": "2016-08-08T16:23:40.295Z",
  "detail_url": "https://www.zalando.de/benutzerkonto/bestellung-detail/10"
}
```

The customer can get an overview of orders in the partner platform, which, assuming it only displays the information provided in the [above endpoint](#getting-a-list-of-orders), will be a limited set of information. To get a more detailed view, clients of our API can use this endpoint. The main purpose is to serve as a complimentary endpoint to the order list.

The last two endpoints are read only operations, which means that operations like Returns or Cancellation are not supported by our API. For a full featured post-sales experience, then the customer will have to go to Zalando shop, and manage his or her orders there.

### Request

`GET /api/orders/{order_number}`

Parameter | Type | Description | Required
--------- | ---- | ----------- | --------
X-Sales-Channel | Header | Used to specify under which sales channel the request belongs to. Only if the order was created under the same sales channel will it be returned | Yes
OAuth token | Header | Customer token used to authenticate the request. The token also helps validate that the requested order belongs to the user making the request. The token also gives information about the API client so that the result is filtered to return the data only if the order was placed via a sales channel of that Zalando partner | Yes
order_number | Path | The order number | Yes

## Payment

Payment is the last step in the checkout process. After the customer confirmed everything (articles and addresses), then it's time to specify the payment method. Here you'll find some information about the payment step. Because payment is not owned by Atlas, nor do we act as a gateway to it, information here is limited, but enough to know how to interact with our component during the payment step. For any questions feel free to reach out to team Atlas.

As mentioned above, payment is done in an individual component that does not belong to team Atlas. One of the reasons for this is PCI compliance. This means that customers will be directed to a separate website where they will select the payment method, and input whatever information is necessary to purchase with that method. Fortunately, Zalando's payment component allows customization, which enables a seamless transition. From our customers' perspective it can all have the same look and feel.

Zalando offers its customers a wide selection of payment methods in the different countries that Zalando operates in. That selection includes some classics like Credit Card, Bank Transfer, PayPal, Invoice, and a few others. The real offering depends on quite a few factors, such as, but not limited to:

* Country - some methods are exclusive for a country, or are not available in a country.
* Amount - depending on the total amount some methods may not be available.
* Address type - the addresses the customer submits may have a hand in the list of possible payment methods (you cannot choose Cash-On-Delivery if you deliver to a Packstation, for example).

After selecting the method that the customer wants to pay his or her order with, will most likely come a step to input some information. For credit cards, for example, you will have to submit the number, CVV, and expiration date. Some payment methods, however, require an extra step. PayPal data is not provided during the payment selection step, for example. And some Credit Card payments will add an extra authentication step, 3-D Secure (3DS). The two examples given will take place after order creation. Let's look at the flow with PayPal:

* Customer picks PayPal.
* Customer is redirected back to the checkout overview.
* Customer confirms everything, and presses the 'Buy' button.
* Order is created, but in a preliminary state.
* Customer gets redirected to the PayPal site, to input his or her PayPal account details.
* The success of failure is communicated through the redirect link provided by the API clients.

Regarding the state of the order, in case of success it will be complete. In case of failure, the order will remain as preliminary for some time (the time period is not controlled by team Atlas). During that time customers can still try to pay for their order. After that, the order will be canceled.

To know how all of this takes place in the checkout process using our component, please read the section to [create a checkout](#creating-a-checkout).
