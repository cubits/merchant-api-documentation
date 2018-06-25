---
layout: default
permalink: /callback/
title: Callbacks
---
## General

Specifying a `callback_url` when creating a resource supporting callbacks (e.g. an `invoice`, `channel` or `quote_channel`), will trigger a POST request to this URL upon any change of an attribute of the respective resource. The request body of the callback will contain the same output that a GET request to the updated resource would return right after the event that triggered the update.

### Example

Invoice creation:
```
POST /api/v1/invoices HTTP/1.1

{
  "currency": "EUR",
  "price": "266.45",
  "name": "Guatemala Joe",
  "description": "Twelve ounces of high quality, organic Guatemalan coffee",
  "reference": "d0cef2",
  "callback_url": "https://my.shop.online/callback"
}
```
Callback after payment:
```
{
  "id": "378d8ec6e305f469b009cb4e2deedf93",
  "status": "completed",
  "address": "34MajoUDkKL5DKKwy6Lk5zkLf84B88TPzv",
  "alt_addresses": [
    {
      "address": "34MajoUDkKL5DKKwy6Lk5zkLf84B88TPzv",
      "type": "base58",
      "default": true
    }
  ],
  "merchant_currency": "EUR",
  "merchant_amount": "266.45",
  "invoice_currency": "BTC",
  "invoice_amount": "0.88613839",
  "paid_currency": "BTC",
  "paid_amount": "0.88613839",
  "pending_currency": "BTC",
  "pending_amount": "0.00000000",
  "name": "Guatemala Joe",
  "description": "Twelve ounces of high quality, organic Guatemalan coffee",
  "reference": "d0cef2",
  "invoice_url": "https://pay.cubits.com/invoice/378d8ec6e305f469b009cb4e2deedf93",
  "callback_url": "https://my.shop.online/callback",
  "success_url": "https://my.shop.online/success",
  "cancel_url": "https://my.shop.online/cancel",
  "notify_email": "payments@my.shop.online",
  "create_time": 1398871897.0,
  "valid_until_time": 1398872497.0
}
```

## Authenticating Callbacks

To provide authentication for the callback, the Cubits API signs the POST request body with the API key and secret, which were used to create or last update the resource. The signature and API key is then passed in the callback headers, together with a callback ID.

### POST Request Headers

Header               | Data type   | Description
---------------------|-------------|--------------------
X-Cubits-Callback-Id | string(8)   | Unique identifier of the callback request
X-Cubits-Key         | string(32)  | *(optional)* API key which was used to create the resource
X-Cubits-Signature   | string(128) | *(optional)* Signature of the request body

If the resource was not created using an API key (for example invoices created by *payment buttons*), the Cubits API does not sign the callback and therefore the headers `X-Cubits-Key` and `X-Cubits-Signature` are omitted.

### X-Cubits-Signature

The signature is derived from the API access secret (CUBITS_SECRET) that was used to create the resource, a unique identifier of the callback (X-Cubits-Callback-Id) and the request body. It is represented as a **hex-string** of the result of the following hash calculation:

```
HMAC-SHA512(k, msg)
```

Where:

`k` is your API access secret (CUBITS_SECRET) as a UTF-8 string

`msg` is a UTF-8 string, constructed by string concatenation of `X-Cubits-Callback-Id`+`SHA256(request-data)`

`X-Cubits-Callback-Id` is a UTF-8 string value of the callback unique identifier passed in the request headers

`SHA256(request-data)` is the hex-encoded SHA256 digest of `request-data`, as a UTF-8 string, **downcased**

`request-data` is the [JSON](http://tools.ietf.org/html/rfc7159) encoded request body

### Example
> You have created an invoice using your API access key `7287ba0902461025b01d5b99e4679018`

> API access secret for this key is `93yJJ8LBDe3zNSewHBdX1XIQDjCMDIn0EKNnXrd3kfzL72fvLz99uKnXFLYuCfkt`

> You have received the callback with `X-Cubits-Callback-Id` being `ABCDEFGH` and the POST request body is `{"attr1": 123, "attr2": "hello"}`

> *msg* is then `ABCDEFGH947753ba472927154c534cf2e4e11de27ed7a9560dc033e77d6cc24ee950ea56`

> *X-Cubits-Signature* for this callback should then be `7d89c35c2e0840867f63b77ea575050db21a134b674d4a38f1e255518efb5b81383442cd9a888dca86dfe3e43a0769525088aac3efed3102a6b14bd1446f14a1`


> It is very important that you verify the callback authenticity and ensure that the callback was actually posted by Cubits. Otherwise it is quite simple for some attacker to forge a callback and to trick you into believing that the payment was made, whereas it was not.

There are two ways to achieve this:

1. Check the signature of the callback, provided in the headers
1. After receiving a callback, get the `id` of the resource and retrieve the actual resource details with a `GET` request.


### Example

```
POST /your/callback/path HTTP/1.1
Content-Type: application/vnd.api+json
X-Cubits-Callback-Id: A7PPGKYM
X-Cubits-Key: 0086bf7149ac69e05ec1808b9f187a10
X-Cubits-Signature: 941923cb09ec43b7b2453ccd4d7196794b9c699157a4c69166d6476108e75bf3...

{
  "merchant_currency": "EUR",
  "merchant_amount": "12.34",
  "invoice_currency": "BTC",
  "invoice_amount": "0.05348733",
  "paid_currency": "BTC",
  "paid_amount": "0.00000000",
  "pending_currency": "BTC",
  "pending_amount": "0.00000000",
  "share_to_keep_in_btc": 0,
  "name": null,
  "description": null,
  "reference": null,
  "callback_url": "https://my.shop.online/your/callback/path",
  "success_url": null,
  "cancel_url": null,
  "notify_email": null,
  "id": "a5fe404f40d628e65ff2d739a63a80f7",
  "status": "pending",
  "invoice_url": "https://pay.cubits.com/invoices/a5fe404f40d628e65ff2d739a63a80f7",
  "address": "34MDjoUDkKL5DKKwy6Lk5zkLf84B88TPzv",
  "alt_addresses": [
    {
      "address": "34MDjoUDkKL5DKKwy6Lk5zkLf84B88TPzv",
      "type": "base58",
      "default": true
    }
  ],
  "valid_until_time": 1426520056.0,
  "create_time": 1426519456.0
}
```

## Callback Retry

The Cubits callback API will only consider a callback as successfully sent when it receives a 2xx HTTP status code (200-299) from your application that is processing the callback POST request.
In case your application is currently unable to process the callback or if there is some error, it should return with a different status code, letting Cubits know that it should retry the callback.
By that we make sure that updates to the status of a resource is not lost even in case of some downtime of your application.

The callback retries will happen in the following increasing time intervals:

Retry attempt | Interval   | Seconds after initial callback
--------------|------------|-------------------------------
1             | 1 second   | 1
2             | 5 seconds  | 6
3             | 10 seconds | 16
4             | 30 seconds | 46
5             | 2 minutes  | 166
6             | 15 minutes | 1066
7             | 1 hour     | 4666
8             | 2 hours    | 11866
9             | 12 hours   | 55066
10            | 24 hours   | 141466
11            | 7 days     | 746266
12            | 14 days    | 1955866
