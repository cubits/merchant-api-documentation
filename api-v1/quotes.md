---
layout: default
permalink: /api-v1/quotes/
title: Quotes
---
Quotes provide an estimation of the exchange rate for a given amount and operation type. They can be used to get information about the result of a subsequent *buy* or *sell* operation.

## POST /api/v1/quotes

Requests a quote for a *buy* or *sell* operation.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
operation   | string(32)  | Type of the transaction: `buy` or `sell`
sender      | object      | *Sender* object specifying the spending part of the transaction
receiver    | object      | *Receiver* object specifying the receiving part of the transaction

#### Sender Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that you want to spend (see [List of supported currencies](/merchant-api-documentation/appendices/#supported_fiat_currencies) and [List of supported cryptocurrencies](/merchant-api-documentation/appendices/#supported_cryptocurrencies))
amount      | string(32)  | *(optional)* Amount in specified currency to be spent, decimal number as a string (e.g. "12.50")

#### Receiver Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency you want to receive (see [List of supported currencies](/merchant-api-documentation/appendices/#supported_fiat_currencies) and [List of supported cryptocurrencies](/merchant-api-documentation/appendices/#supported_cryptocurrencies))
amount      | string(32)  | *(optional)* Amount in specified currency to be received, decimal number as a string (e.g. "12.50")

#### Required Attributes

Exactly one amount, either `sender.amount` or `receiver.amount` must be specified.

### Response

On success, the API responds with HTTP status `201 Created` and the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
operation   | string(32)  | Type of the transaction: `buy` or `sell`
sender      | object      | *Sender* object specifying the spending part of the transaction
receiver    | object      | *Receiver* object specifying the receiving part of the transaction

#### Sender Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the fiat currency to spend (e.g. "EUR")
amount      | string(32)   | Amount in specified currency to be spent, decimal number as a string (e.g. "12.50")

#### Receiver Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency to receive (e.g. "BTC")
amount      | string(32)   | Amount in specified currency to be received, decimal number as a string (e.g. "0.15000000")

### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses) and with some specific to this request:

#### Client errors (4xx)

Situation                 | HTTP status code  | message
--------------------------|-------------------|-------------
Requested operation is not supported | 400 Bad Request | Invalid operation
Amount has invalid format | 400 Bad Request   | Invalid amount
Requested currency is not supported | 400 Bad Request   | Unsupported currency

### Example

Request:
```
POST /api/v1/quotes HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****

{
  "operation": "buy",
  "sender": {
    "currency": "EUR"
  },
  "receiver": {
    "amount": "0.15000000",
    "currency": "BTC"
  }
}
```

Response:
```
HTTP/1.1 201 Created
Content-Type: application/vnd.api+json

{
  "operation": "buy",
  "sender": {
    "amount": "29.89",
    "currency": "EUR"
  },
  "receiver": {
    "amount": "0.15000000",
    "currency": "BTC"
  }
}
```
