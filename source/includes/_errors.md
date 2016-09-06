# Errors

Error cases are handled in the same way across our services. We strive to use proper HTTP codes for the different cases, and use the same object format for errors in every endpoint. As can be seen in our API specs, the content type `x.application/problem+json` is specified for said error cases. Please refer to the API spec for the specification of that content type.

Our components use the following error codes. Specific meanings are described in the section for the individual operations. Here you'll find a more common definition for the different error codes.

Error Code | Description
---------- | -----------
400 - Bad Request | The input provided with the request is invalid. Information can be missing, invalid, or in a wrong format.
401 - Unauthorized | Your token is not valid or wrong
403 - Forbidden | The resource requested is hidden for administrators only
404 - Not Found | The specified violation could not be found
406 - Not Acceptable | You requested a format that isn't json
429 - Too Many Requests | The rate of request is too high. Some requests will be dropped.
500 - Internal Server Error | We had a problem with our server. Try again later.
503 - Service Unavailable | We're temporarily offline for maintenance. Please try again later.
