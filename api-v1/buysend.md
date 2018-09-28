---
layout: default
permalink: /api-v1/buysend/
title: Buysend
---
## POST /api/v1/buysend

Creates a transaction to buy cryptocurrency using funds from your Cubits account. Bought cryptocurrency will be sent to address provided.

The exact exchange rate will be calculated at the transaction execution time.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
sender      | object      | *Sender* object defining the spending part of the transaction
receiver    | object      | *Receiver* object defining the spending part of the transaction
address     | string(64)   | Address of the cryptocurrency the amount is to be sent to
reference   | string(512) | *(optional)* Individual free-text field stored in the transaction as-is

#### Sender Object

Attribute              | Data type   | Description
-----------------------|-------------|--------------
currency               | string(3)   | Code of the currency that you want to spend (see [List of supported currencies](/appendices/#supported_fiat_currencies))
amount                 | string(17)  | Amount in specified currency to be spent, decimal number as a string (e.g. "12.50")
amount_minus_fees      | string(17)  | Amount after fees in specified currency to be spent, decimal number as a string (e.g. "12.50")

#### Receiver Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the cryptocurrency that you want to spend (see [List of supported cryptocurrencies](/appendices/#supported_cryptocurrencies))
amount            | string(17)  | Amount in specified cryptocurrency to be spent, decimal number as a string (e.g. "0.01250000")
amount_plus_fees  | string(17)  | Amount in specified cryptocurrency to be spent, without fees deducted (e.g. "0.01250000")

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
currency    | string(3)   | Code of the currency that was spent (e.g. "EUR")
amount      | string(17)  | Amount that was spent (e.g. "12.50")
amount_minus_fees      | string(17)  | Total amount that was send in chosen currency

#### Receiver Object

Attribute             | Data type   | Description
----------------------|-------------|--------------
currency              | string(3)   | Code of the currency that was received (e.g. "BTC")
amount                | string(17)  | Amount that was received (e.g. "0.05750000")
address               | string(64)  | Address where money were transferred


### Errors

On error, the API responds with standard [error responses](/request_response/#error_responses) and with some specific to this request:

#### Client errors (4xx)

Situation                 | HTTP status code  | message
--------------------------|-------------------|-------------
Amount has invalid format | 400 Bad Request   | Invalid amount
Requested amount exceeds balance | 400 Bad Request   | Insufficient funds
Requested currency is not supported | 400 Bad Request   | Unsupported currency
Bitcoin address is invalid | 400 Bad Request   | Invalid address

### Example

Request:
```
POST /api/v1/buysend HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****

{
  "sender": {
    "currency": "EUR",
    "amount": "12.50"
  },
  "address": "1AeMbkpHia8FVuKczQKUrv9uMzv7uC1HZi"
}
```

Response:
```
HTTP/1.1 201 Created
Content-Type: application/vnd.api+json

{
  "tx_ref_code": "6NFXB",
  "sender": {
    "currency": "EUR",
    "amount": "12.50",
    "amount_minus_fees": "12.40",
  },
  "receiver": {
    "currency": "BTC",
    "amount": "0.05750000",
    "address": "1AeMbkpHia8FVuKczQKUrv9uMzv7uC1HZi"
  }
}
```
