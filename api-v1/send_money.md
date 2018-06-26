---
layout: default
permalink: /api-v1/send_money/
title: Send Money
---
## POST /api/v1/send_money

Creates a transaction to send cryptocurrency from your Cubits wallet to an external address.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
amount      | string(32)  | Amount of cryptocurrency to be sent, decimal number as a string (e.g. "0.12500000")
address     | string(64)  | Address the amount is to be sent to
currency    | string(3)   | *(optional)* A code of cryptocurrency to be sent (see [List of supported cryptocurrencies](/merchant-api-documentation/appendices/#supported_cryptocurrencies), default: `BTC`)
reference   | string(512) | *(optional)* Individual free-text field stored in the tx as-is

### Response

On success, the API responds with HTTP status `201 Created` and the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
tx_ref_code | string(32)  | Reference code of the created transaction

### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses) and with some specific to this request:


#### Client errors (4xx)

Situation                 | HTTP status code  | message
--------------------------|-------------------|-------------
Amount has invalid format | 400 Bad Request   | Invalid amount
Requested amount exceeds balance | 400 Bad Request   | Insufficient funds
Bitcoin address is invalid | 400 Bad Request   | Invalid address

### Example

Request:
```
POST /api/v1/send_money HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****

{
  "amount": "0.12340000",
  "currency": "BTC",
  "address": "1AeMbkpHia8FVuKczQKUrv9uMzv7uC1HZi"
}
```

Response:
```
HTTP/1.1 201 Created
Content-Type: application/vnd.api+json

{
  "tx_ref_code": "6NFXB"
}
```
