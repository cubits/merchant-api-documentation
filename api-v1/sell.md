---
layout: default
permalink: /api-v1/sell/
title: Sell
---
## POST /api/v1/sell

Creates a transaction to sell cryptocurrency from your Cubits wallet and receive amount in specified fiat currency. Fiat funds will be credited to your Cubits account.

The exact exchange rate will be calculated at the transaction execution time.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
sender      | object      | *Sender* object defining the spending part of the transaction
receiver    | object      | *Receiver* object defining the receiving part of the transaction

#### Sender Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the cryptocurrency to sell(see [List of supported cryptocurrencies](/appendices/#supported_cryptocurrencies))
amount      | string(17)  | Amount in specified cryptocurrency to be spent, decimal number as a string (e.g. "0.01250000")
amount_minus_fees | string(17)  | Amount in specified cryptocurrency to be spent after all fees paid, decimal number as a string (e.g. "0.01250000")

#### Receiver Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that you want to receive (see [List of supported currencies](/appendices/#supported_fiat_currencies))
amount            | string(17)  | Amount in specified currency to be received, decimal number as a string (e.g. "12.50")
amount_plus_fees  | string(17)  | Amount in specified currency to be received, without fees deducted (e.g. "12.50")

Note: only one out of four `amount` parameters above should be passed. Error will be returned otherwise.

### Response

On success, the API responds with HTTP status `201 Created` and the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
tx_ref_code | string(32)  | Reference code of the created transaction
sender      | object      | Information about the *sender* part of this transaction
receiver    | object      | Information about the *receiver* part of this transaction

#### Sender Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that was spent (e.g. "BTC")
amount      | string(17)  | Amount that was spent (e.g. "0.15000000")

#### Receiver Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that was received (e.g. "EUR")
amount      | string(17)  | Amount that was received (e.g. "859.26")


### Errors

On error, the API responds with standard [error responses](/request_response/#error_responses) and with some specific to this request:

#### Client errors (4xx)

Situation                 | HTTP status code  | message
--------------------------|-------------------|-------------
Amount has invalid format | 400 Bad Request   | Invalid amount
Requested amount exceeds balance | 400 Bad Request   | Insufficient funds
Requested currency is not supported | 400 Bad Request   | Unsupported currency

### Example

Request:
```
POST /api/v1/sell HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****

{
  "sender": {
    "amount": "0.15000000",
    "amount": "BCH"
  },
  "receiver": {
    "currency": "EUR"
  }
}
```

Response:
```
HTTP/1.1 201 Created
Content-Type: application/vnd.api+json

{
  "tx_ref_code": "6NFXB",
  "sender": {
    "currency": "BCH",
    "amount": "0.15000000"
  },
  "receiver": {
    "currency": "EUR",
    "amount": "32.63"
  }
}
```
