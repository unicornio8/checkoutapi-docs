# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl -X GET -H "Authorization: Bearer $(oauth2 token)" \
--header "Accept: application/json" "api_endpoint_here"
```

Atlas' APIs use Oauth2 tokens for authentication and authorization. Different endpoints can require tokens from different realms, such as <code>/service</code> or <code>/customers</code>, and with different scopes.
You'll find which tokens are valid in the details of each endpoint.

Atlas API expects Oauth2 tokens to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer {token}`

## How to get access to our components?

Getting access to our components is currently a manual process, so the best course of action is to contact us (team-atlas@zalando.de).

Be sure, though, that 3 questions will be asked, so you might as well have an answer for them:

* Is it in Staging? Or Live?
 * There is a difference in the URLs for staging and live, as one would expect.
 * Also it is important to know the expected extra load on the services.
* Is it for service-to-service communication, or for customers?
 * Service-to-service uses the Resource Owner Password Credentials Grant, so it requires credentials for a Service User (username and password) and Client credentials (Client ID and Client Secret).
 * Customers will user their own login, but to request the token Client credentials are still required.
 * Only a few endpoints are accessible for service-to-service, so be ready to have a customer token ready.
* Is it for internal Zalando developers, or Zalando partners (i.e. the intended clients of the APIs).
 * For Zalando partners, there are more things to take care of than just authentication and authorization.

For further questions (such as "I am not quite sure what is my use case") feel free to contact team Atlas (team-atlas@zalando.de).

## Scopes

Most operations are secured by OAuth2 and we reserve the ability to control the access to the different operations via the use of scopes. Each operation will be managed by a given scope, and if that scope is not included in the token, then access to that operation is denied. However, you do not need to specify the individual scopes when requesting a token as these will be assigned to all valid tokens as a default set. This prevents any break in talking to our APIs should a new scope be created for a new or old feature.

## Service authentication

> Sample Request

```shell
curl -v -X POST
  -H "Authorization: Basic cSDF923sdWERweerZZWFG9348dTdBN=="
  -H "Content-Type: application/x-www-form-urlencoded"
  -H "Accept: application/json"
  -d 'grant_type=password&username=client_HS23eDa2&password=somepassword&realm=/services'
  "https://{Token Service URL}/oauth2/access_token"
```

> Sample Response

```shell
{
    "access_token":"{eyJraWQiOiJMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9ODErNYwiYXpwIjoic3R5bGlnaHRfmaSDggHHjDIsInNjb3BlIjpbIm5g6HH7k88BpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0Njg4NzgwNTksImlhdCI6MTQ2ODg0OTI1OX0.Uxm-sOif2HN7z3Mam3--TEpkpRaCxP9gFIYmSDSVmR07b8lqOVtsSPMTslhcmRfF2HVGG1Wjw}",
    "token_type":"Bearer",
    "expires_in":28800,
    "scope":"scope1 scope2 scope3",
    "realm":"/services"
}
```

Service-to-service authentication is done with the Resource Owner Password Credentials Grant. This means you need a Service User, with a username and a password, as well as a Client ID and Client Secret. This type of tokens is used for requests that do not require consent by a customer, so communication can be exclusively between two servers. This, of course, includes getting a token, validating it, and renewing it (where token renewing is an option).

Service tokens are valid for the Catalog API and for cart operations on the Checkout API, so we can have anonymous carts. The rest of the checkout operations require a customer token.

Service Users and Client credentials are provided by team Greendale, but before getting your credentials talk to us in order to discuss the required use cases.

## Customer authentication

> Login URL for staging

```
https://{Authentication Provider URL}/oauth2/authorize?realm=/customers&response_type=token&client_id=client_HS23eDa2&redirect_uri=http://com.example.app/redirect
```

Customer authentication means that a Zalando customer will input his or her username and password into a login form. Those credentials authenticate the customer, but are not enough to obtain a token. Depending on the OAuth2 flow type more information is required. Currently we support the Implicit Flow and the Authorization Code grant types. For the first the extra information is the Client ID, and the redirect URI, and for the latter the Client Secret is also required. You can see the URL for the login form on the right.

After the customer was successfully authenticated the next step will depend on whether or not is this the first time the customer logs in via the client app or site. If it is the first time, then the customer will have to give consent to share his Zalando account with the client in order to perform the checkout steps. This step will not be necessary in subsequent logins, unless the customer revokes access to the client app or website.

Customer tokens are used for requests that require consent from the customer. Those requests will perform some kind of operation on behalf of the customer (managing addresses, carts, and checkouts, and creating orders). Using our components, a Service token would not work (even with the correct scopes) because the customer is identified by his or her token, and a service token does not identify any customer.

Like Service Authentication, the Client ID is provided by team Greendale. The redirect URI, on the other hand, is provided by the client of our API, as it should be a valid URL for the client's website or mobile app.

## Token Info

> Sample Request

```shell
curl -v --request GET
  -H "Authorization: Bearer eyJraWQiOiJMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9ODErNYwiYXpwIjoic3R5bGlnaHRfmaSDggHHjDIsInNjb3BlIjpbIm5g6HH7k88BpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0Njg4NzgwNTksImlhdCI6MTQ2ODg0OTI1OX0.Uxm-sOif2HN7z3Mam3--TEpkpRaCxP9gFIYmSDSVmR07b8lqOVtsSPMTslhcmRfF2HVGG1Wjw"
  "https://{Token Service URL}/oauth2/tokeninfo"
```

> Sample Response

```shell
{  
   "access_token":"eyJraWQiOiJMjU2IiwiYWxnIjoiRVMyNTYifQ.eyJzdWIiOiJzdHlsaWdodF9ODErNYwiYXpwIjoic3R5bGlnaHRfmaSDggHHjDIsInNjb3BlIjpbIm5g6HH7k88BpLmFydGljbGUucmVhZCIsImF0bGFzLWNhdGFsb2ctYXBpLmZlZWQucmVhZCIsImF0bGFzLWNoZWNrb3V0LWFwaS5jYXJ0LmFsbCIsImF6cCIsInVpZCJdLCJpc3MiOiJCIiwicmVhbG0iOiIvc2VydmljZXMiLCJleHAiOjE0Njg4NzgwNTksImlhdCI6MTQ2ODg0OTI1OX0.Uxm-sOif2HN7z3Mam3--TEpkpRaCxP9gFIYmSDSVmR07b8lqOVtsSPMTslhcmRfF2HVGG1Wjw",
   "scope1":true,
   "scope2":true,
   "scope3":true,
   "client_id":"client_HS23eDa2",
   "expires_in":2014980,
   "grant_type":"password",
   "realm":"/customers",
   "scope":[  
      "scope1",
      "scope2",
      "scope3",
   ],
   "token_type":"Bearer",
   "uid":"0987654321"
}
```

Every call to our endpoints will check for a valid token. However, a client might also want to check whether a token is valid or not. To do that, the Token Info endpoint is the proper way to go. Besides knowing the validity of the token, one can also check the scopes assigned to that token, the customer ID (in the `uid` field for a customer token), the expiry time of the token (`expires_in` field), and the client that requested the token (`client_id`) among other properties.
