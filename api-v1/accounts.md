---
layout: default
permalink: /api-v1/accounts/
title: Accounts
---
## GET /api/v1/accounts

Retrieves a list of your Cubits wallet accounts. Each wallet can have accounts in different currencies. With this call you can get a complete overview of all your balances on Cubits at any date.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
date        | string(20)  | *(optional)* the date in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format to get the accounts at (default: none, which means current balances)

### Response

On success, the API responds with HTTP status `200 OK` and the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
date        | string(20)  | The ISO-8601-formatted date of the reported balances
accounts    | array       | Array of *account* objects

### Account Objects

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the account currency (e.g. "EUR")
balance     | string(32)  | Current balance of the account, decimal number as a string (e.g. "12.50")

### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses).

### Example

Request:
```
GET /api/v1/accounts?date=2016-02-08 HTTP/1.1
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
  "date": "2016-02-08T00:00:00Z",
  "accounts": [
    {
      "currency": "EUR",
      "balance": "128.45"
    },
    {
      "currency": "BTC",
      "balance": "0.35090347"
    }
  ]
}
```
