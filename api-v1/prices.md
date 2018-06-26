---
layout: default
permalink: /api-v1/prices/
title: Prices
---
The following endpoints provide current bid, ask and mid prices for the [supported cryptocurrencies](/merchant-api-documentation/appendices/#supported_cryptocurrencies).

## GET /api/v1/prices/{currency_base}

Lists the fiat prices for the specified cryptocurrency (`currency_base` parameter).

### Request

None

### Response

On success, the API responds with HTTP status `200 OK` and the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
prices      | array       | Array of *price* objects


#### Price Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency (e.g. "EUR")
bid         | string(17)  | The highest buy price
ask         | string(17)  | The lowest sell price
mid         | string(17)  | The price between the best buy and the best sell prices


### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses)

### Example

Request to get BTC price in all supported fiat currencies:

Request:
```
GET /api/v1/prices/BTC HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****
```

Response:
```
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "prices": [
    {
      "currency": "INR",
      "bid": "638311.81",
      "ask": "664986.18",
      "mid": "651649.00"
    },
    {
      "currency": "AUD",
      "bid": "12809.13",
      "ask": "13147.93",
      "mid": "12978.53"
    },
    {
      "currency": "CAD",
      "bid": "12303.92",
      "ask": "12818.09",
      "mid": "12561.01"
    },
    ...
  ]
}
```

## GET /api/v1/prices/{currency_base}/{currency_quote}

Get the price of the specified cryptocurrency (`currency_base` parameter) in the specified fiat currency (`currency_quote` parameter).

### Request

None

### Response

On success, the API responds with HTTP status `200 OK` and the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency (e.g. "EUR")
bid         | string(17)  | The highest buy price
ask         | string(17)  | The lowest sell price
mid         | string(17)  | The price between the best buy and the best sell prices


### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses)

### Example

Request to get BTC price in EUR:

Request:
```
GET /api/v1/prices/BTC/EUR HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****
```

Response:
```
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "currency": "EUR",
  "bid": "8100.57",
  "ask": "8185.67",
  "mid": "8143.12"
}
```


Request to get BCH price in USD:

Request:
```
GET /api/v1/prices/BCH/USD HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****
```

Response:
```
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "currency": "USD",
  "bid": "1312.31",
  "ask": "1384.11",
  "mid": "1348.21"
}
```
