---
layout: default
permalink: /api-v1/quote_channels/
title: Quote Channels
---
Quote channels provide a method to receive and potentially convert cryptocurrency payments of variable amounts just as with regular channels. The difference is that quote channels have a fixed exchange rate that is used for the cryptocurrency conversion within the validity time of the quote but at the same time still convert incoming funds to the requested *receiver_currency* at the spot price if they are paid after the validity time.
Quote channels have to be created for a certain expected amount in any of the supported fiat currencies. They have a fixed payment address, described by potentially several address formats, and support callbacks. Cryptocurrency payments for less than the smallest possible unit for the chosen fiat currency will not be converted but transferred as-is to the Cubits Wallet.
Quote channels also provide a way of tracking individual payments made to them. A list of transactions is recorded with each quote channel and returned upon querying its transaction endpoint as well as with each transaction callback.

## GET /api/v1/quote_channels/{channel_id}

Get information about an existing quote channel.

### Request

None

### Response

On success, a response with HTTP status `200 OK` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
id          | string(32)  | Unique hex-string identifier of the quote channel
address     | string(34)  | Cryptocurrency address in the default format associated with this quote channel
alt_addresses     | array     | Array of *address* objects
sender_currency | string(3)   | Code of the cryptocurrency accepted by the quote channel
sender_amount | string(16)  | Amount of currency the quote channel expects
receiver_currency | string(3) | Code of the currency that you want to receive
receiver_amount | string(16)  | Amount of currency that should be received (used to calculate the quote) as a decimal floating point number, converted to string (e.g. "123.05")
name        | string(256) | Name of the quote channel
description | string(512) | Description of the quote channel
reference   | string(512) | Individual free-text field stored in the quote channel as-is
callback_url| string(512) | URL that is called on quote channel updates
txs_callback_url| string(512) | URL that is called on quote channel transaction updates
success_url | string(512) | URL to redirect the user to after a successful payment
share_to_keep_in_btc | number | Percent of the each transaction to receive in `sender_currency`, as a number from 0 to 100
created_at  | number      | (float) Unix-epoch timestamp of the quote channel creation
updated_at  | number      | (float) Unix-epoch timestamp when the quote channel data was last updated
valid_until | number | (float) Unix-epoch timestamp after which the quote will expire

### Address objects

Attribute | Data type  | Description
----------|------------|--------------
address   | string(64) | A formatted crypto-address
type      | string(64) | The name of the format of the address (e.g. `base58` or `cash_addr`)
default   | bool       | Whether this address is the default address format, i.e. the `address` attribute of the *channel* object

### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses).

### Example

Request:
```
GET /api/v1/quote_channels/ae9641776a94456a8067970b9765e396 HTTP/1.1
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
  "sender_currency": "BTC",
  "sender_amount": "0.40623221",
  "receiver_currency": "EUR",
  "receiver_amount": "164.25",
  "name": "Order XYZ",
  "description": null,
  "reference": "xyz",
  "callback_url": "https://example.com/callback",
  "txs_callback_url": "https://example.com/txs_callback",
  "success_url": "https://example.com/thank_you.html",
  "created_at": 1427234017.0,
  "updated_at": 1427235200.0,
  "id": "ae9641776a94456a8067970b9765e396",
  "address": "35xv5napaKi4sLuYZWT9xcp7aaj6YZxpkC",
  "alt_addresses": [
    {
      address: "35xv5napaKi4sLuYZWT9xcp7aaj6YZxpkC",
      type: "base58",
      default: true
    }
  ],
  "share_to_keep_in_btc": "0",
  "valid_until": 1427234917.0
}
```

## GET /api/v1/quote_channels/{channel_id}/txs

Get information about all transactions of an existing quote channel.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
page        | number      | *(optional)* Page number to return (default: 1)
per_page    | number      | *(optional)* Number of transactions to return per page (max: 1000, default: 100)

### Response

On success, a response with HTTP status `200 OK` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
pagination  | object      | *Pagination* object for the returned txs array
txs         | array       | Array of *transaction* objects of this quote channel sorted by descending *updated_at* field

#### Pagination Object

Attribute   | Data type   | Description
------------|-------------|--------------
page        | number      | Index of the current page (1 <= page <= page_count)
page_count  | number      | Number of total pages in the result
per_page    | number      | Number of entries per page (1 <= per_page <= 1000)
total_count | number      | Number of total entries in the result

#### Transaction Object

Attribute   | Data type   | Description
------------|-------------|--------------
tx_ref_code | string(6)   | Unique character string to reference this transaction
quote_channel_id  | string(32)  | Id of the quote channel this transaction belongs to
state       | string(9)   | `pending`, `completed` or `cancelled`
created_at  | number      | (float) Unix-epoch timestamp of the transaction creation
updated_at  | number      | (float) Unix-epoch timestamp when the transaction data was last updated
sender      | object      | Information about the *sender* part of this transaction
receiver    | object      | Information about the *receiver* part of this transaction

Unconfirmed transactions will stay in the `pending` state until Cubits deems the risk to accept them as low enough for them to go in the `completed` state.
A transaction that was a double-spend or one that will not confirm for other reasons (e.g. because of the Bitcoin dust limit), will eventually go to `cancelled` state.

#### Sender Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that was sent
amount      | string(17)  | Amount that was sent
bitcoin_txid | string(64) | Blockchain transaction id of this transaction or `null` in case of a Cubits internal transfer

#### Receiver Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that was received.
amount      | string(17)  | Amount that was received, with fees deducted. Note that this amount is preliminary in case of `pending` transactions and might change once the transaction goes to `completed` state.
amount_plus_fees | string(17)  | Amount that was received, without fees deducted. Note that this amount is preliminary in case of `pending` transactions and might change once the transaction goes to `completed` state.

### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses).

### Example

Request:
```
GET /api/v1/quote_channels/ae9641776a94456a8067970b9765e396/txs HTTP/1.1
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
  "pagination": {
    "page": 1,
    "page_count": 1,
    "per_page": 100,
    "total_count": 3
  },
  "txs": [
    {
    "tx_ref_code": "G3H4W",
    "quote_channel_id": "ae9641776a94456a8067970b9765e396",
    "state": "completed",
    "created_at": 1427234023.0,
    "updated_at": 1427234023.0,
    "sender": {
      "currency": "BTC",
      "amount": "0.00020000",
      "bitcoin_txid": "dd0b3099a7bb981d6ac08ec80d805d74260c8d83649a9466671018144408765c"
    },
    "receiver": {
      "currency": "EUR",
      "amount": "0.04",
      "amount_plus_fees": "0.05"
    }
  },
  {
    "tx_ref_code": "284W9",
    "quote_channel_id": "ae9641776a94456a8067970b9765e396",
    "state": "pending",
    "created_at": 1427234123.0,
    "updated_at": 1427235200.0,
    "sender": {
      "currency": "BTC",
      "amount": "0.00050000",
      "bitcoin_txid": "67d2de1c07129c482c88db51563c252bc52267976d4b060f8f78ceff1bb419b3"
    },
    "receiver": {
      "currency": "EUR",
      "amount": "0.12",
      "amount_plus_fees": "0.13"
    }
  }]
}
```

---
## GET /api/v1/quote_channels/{channel_id}/txs/{tx_ref_code}

Get information about an individual transactions of a quote channel.

### Request

None

### Response

On success, a response with HTTP status `200 OK` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
tx_ref_code | string(6)   | Unique character string to reference this transaction
quote_channel_id  | string(32)  | Id of the quote channel this transaction belongs to
state       | string(9)   | `pending`, `completed` or `cancelled`
created_at  | number      | (float) Unix-epoch timestamp of the transaction creation
updated_at  | number      | (float) Unix-epoch timestamp when the transaction data was last updated
sender      | object      | Information about the *sender* part of this transaction
receiver    | object      | Information about the *receiver* part of this transaction

Unconfirmed transactions will stay in the `pending` state until Cubits deems the risk to accept them as low enough for them to go in the `completed` state.
A transaction that was a double-spend or one that will not confirm for other reasons (e.g. because of the Bitcoin dust limit), will eventually go to `cancelled` state.

#### Sender Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that was sent
amount      | string(17)  | Amount that was sent
bitcoin_txid | string(64) | Blockchain transaction id of this transaction or `null` in case of a Cubits internal transfer

#### Receiver Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that was received
amount      | string(17)  | Amount that was received, with fees deducted Note that this amount is preliminary in case of `pending` transactions and might change once the transaction goes to `completed` state
amount_plus_fees | string(17)  | Amount that was received, without fees deducted. Note that this amount is preliminary in case of `pending` transactions and might change once the transaction goes to `completed` state

### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses).

### Example

Request:
```
GET /api/v1/quote_channels/ae9641776a94456a8067970b9765e396/txs/XGHWW HTTP/1.1
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
  "tx_ref_code": "XGHWW",
  "quote_channel_id": "ae9641776a94456a8067970b9765e396",
  "state": "completed",
  "created_at": 1427137517.0,
  "updated_at": 1427137623.0,
  "valid_until": 1427138417.0,
  "sender": {
    "currency": "BTC",
    "amount": "0.00010000",
    "bitcoin_txid": "9ae311aae58c9150ca22f5b3fa7b64e32bc4e25cf9db11ba6bd28de50bcd1f73"
  },
  "receiver": {
    "currency": "EUR",
    "amount": "0.02",
    "amount_plus_fees": "0.03"
  }
}
```

---
## POST /api/v1/quote_channels

Creates a new quote channel.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
sender_currency | string(3)   | *(optional)* Code of the cryptocurrency accepted by the quote channel (see [List of supported cryptocurrencies](/merchant-api-documentation/appendices/#supported_cryptocurrencies))
receiver_currency | string(3) | Code of the currency that you want to receive (see [List of supported currencies](/merchant-api-documentation/appendices/#supported_fiat_currencies) and [List of supported cryptocurrencies](/merchant-api-documentation/appendices/#supported_cryptocurrencies))
receiver_amount | string(16)  | Amount of currency that should be received (used to calculate the quote) as a decimal floating point number, converted to string (e.g. "123.05")
name        | string(256) | *(optional)* Name of the quote channel
description | string(512) | *(optional)* Description of the quote channel
reference   | string(512) | *(optional)* Individual free-text field stored in the quote channel as-is
callback_url| string(512) | *(optional)* URL that is called on quote channel status updates
txs_callback_url| string(512) | *(optional)* URL that is called on quote channel transaction updates
success_url | string(512) | *(optional)* URL to redirect the user to after a successful payment
share_to_keep_in_btc | string(16) | *(optional)* Percent of the each transaction to receive in `sender_currency`, as a decimal number, converted to string (e.g. "20")

Note that conversion between cryptocurrencies aren't supported yet, meaning if `receiver_currency` is set to `BCH`, `sender_currency` **must** also be explicitly set to `BCH`.

### Response

On success, a response with HTTP status `201 Created` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
id          | string(32)  | Unique hex-string identifier of the quote channel
address     | string(34)  | Cryptocurrency address associated with this quote channel
alt_addresses     | array     | Array of *address* objects
sender_currency | string(3)   | Code of the cryptocurrency accepted by the quote channel
sender_amount | string(16)  | Amount of cryptocurrency the quote channel expects
receiver_currency | string(3) | Code of the currency that you want to receive
receiver_amount | string(16)  | Amount of currency that should be received (used to calculate the quote) as a decimal floating point number, converted to string (e.g. "123.05")
name        | string(256) | Name of the quote channel
description | string(512) | Description of the quote channel
reference   | string(512) | Individual free-text field stored in the quote channel as-is
callback_url| string(512) | URL that is called on quote channel status updates
txs_callback_url| string(512) | URL that is called on quote channel transaction updates
success_url | string(512) | URL to redirect the user to after a successful payment
share_to_keep_in_btc | string(16) | Percent of the each transaction to receive in `sender_currency`, as a decimal number, converted to string (e.g. "20")
created_at  | number      | (float) Unix-epoch timestamp of the quote channel creation
updated_at  | number      | (float) Unix-epoch timestamp when the quote channel data was last updated
valid_until | number      | (float) Unix-epoch timestamp after which the quote will expire

### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses) and with one specific to this request:

#### Client errors (4xx)

Situation                 | HTTP status code  | message
--------------------------|-------------------|-------------
Requested currency is not supported | 400 Bad Request   | Unsupported currency


### Example

Request:
```
POST /api/v1/quote_channels HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****

{
  "sender_currency": "BCH",
  "receiver_currency": "EUR",
  "receiver_amount": "164.25",
  "name": "name": "Order XYZ",
  "reference": "xyz",
  "callback_url": "https://example.com/callback",
  "success_url": "https://example.com/thank_you.html",
  "share_to_keep_in_btc": "0"
}
```

Response:
```
HTTP/1.1 201 Created
Content-Type: application/vnd.api+json

{
  "sender_currency": "BCH",
  "sender_amount": "0.40623221",
  "receiver_currency": "EUR",
  "receiver_amount": "164.25",
  "name": "Order XYZ",
  "description": null,
  "reference": "xyz",
  "callback_url": "https://example.com/callback",
  "txs_callback_url": null,
  "success_url": "https://example.com/thank_you.html",
  "id": "f5790652fb26b62bb48e65941bec06f8",
  "address": "bitcoincash:qpmtetdtqpy5yhflnmmv8s35gkqfdnfdtywdqvue4p",
  "alt_addresses": [
    {
      "address": "bitcoincash:qpmtetdtqpy5yhflnmmv8s35gkqfdnfdtywdqvue4p",
      "type": "cash_addr",
      "default": true
    },
    {
      "address": "1BppmEwfuWCB3mbGqah2YuQZEZQGK3MfWc",
      "type": "base58",
      "default": false
    }
  ],
  "share_to_keep_in_btc": "0",
  "created_at": 1427342218.0,
  "updated_at": 1427342218.0,
  "valid_until": 1427343118.0
  "share_to_keep_in_btc": "0"
}
```

## POST /api/v1/quote_channels/{channel_id}

Updates an existing quote channel.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
description | string(512) | *(optional)* New description of the quote channel
reference   | string(512) | *(optional)* New free-text field stored in the quote channel as-is
name        | string(256) | *(optional)* New name of the quote channel
callback_url| string(512) | *(optional)* New URL that is called on quote channel status updates
txs_callback_url| string(512) | *(optional)* URL that is called on quote channel transaction updates
success_url | string(512) | *(optional)* New URL to redirect the user to after a successful payment
share_to_keep_in_btc | string(16) | Percent of the each transaction to receive in `sender_currency`, as a decimal number, converted to string (e.g. "20")

### Response

On success, a response with HTTP status `200 OK` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
id          | string(32)  | Unique hex-string identifier of the quote channel
address     | string(34)  | Bitcoin address associated with this quote channel
alt_addresses     | array     | Array of *address* objects
sender_currency | string(3)   | Code of the currency that the quote channel can accept (e.g. "BTC")
sender_amount | string(16)  | Amount of currency the quote channel expects
receiver_currency | string(3)   | Code of the currency that you want to receive (e.g. "EUR")
receiver_amount | string(16)  | Amount of currency that should be received (used to calculate the quote)
name        | string(256) | Name of the quote channel
description | string(512) | Description of the quote channel
reference   | string(512) | Individual free-text field stored in the quote channel as-is
callback_url| string(512) | URL that is called on quote channel status updates
txs_callback_url| string(512) | URL that is called on quote channel transaction updates
success_url | string(512) | URL to redirect the user to after a successful payment
share_to_keep_in_btc | string(16) | Percent of the each transaction to receive in `sender_currency`, as a decimal number, converted to string (e.g. "20")
created_at  | number      | (float) Unix-epoch timestamp of the quote channel creation
updated_at  | number      | (float) Unix-epoch timestamp when the quote channel data was last updated
valid_until | number      | (float) Unix-epoch timestamp after which the quote will expire

### Address objects

Attribute | Data type  | Description
----------|------------|--------------
address   | string(64) | A formatted crypto-address
type      | string(64) | The name of the format of the address (e.g. `base58` or `cash_addr`)
default   | bool       | Whether this address is the default address format, i.e. the `address` attribute of the *channel* object

### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses).

### Example

Request:
```
POST /api/v1/quote_channels/f5790652fb26b62bb48e65941bec06f8 HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****

{
  "success_url": "https://example.com/thank_you2.html",
  "share_to_keep_in_btc": "10"
}
```

Response:
```
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "sender_currency": "BTC",
  "sender_amount": "0.40623221",
  "receiver_currency": "EUR",
  "receiver_amount": "164.25",
  "name": "Order XYZ",
  "description": null,
  "reference": "xyz",
  "callback_url": "https://example.com/callback",
  "txs_callback_url": null,
  "success_url": "https://example.com/thank_you2.html",
  "id": "f5790652fb26b62bb48e65941bec06f8",
  "address": "3AvsD1FSJwUwrnXweHVJJ2Av6P4JP1sVyxa",
  "alt_addresses": [
    {
      "address": "3AvsD1FSJwUwrnXweHVJJ2Av6P4JP1sVyxa",
      "type": "base58",
      "default": true
    }
  ],
  "share_to_keep_in_btc": "10",
  "created_at": 1427217218.0,
  "updated_at": 1427217219.0,
  "valid_until": 1427218118.0
}
```

## Callbacks

When specifying a `callback_url`, the server will send callbacks to inform you about changes to any of the quote channel fields. The POSTed request body will correspond to the result of a GET request to /api/v1/quote_channels/{channel_id}.

When specifying a `txs_callback_url`, the server will send callbacks to inform you about changes to any of the quote channel's transactions. The POSTed request body will correspond to the result of a GET request to /api/v1/quote_channels/{channel_id}/txs/{tx_ref_code}.

See [Callbacks](/merchant-api-documentation/callback/) for a general description of the Cubits callback mechanism and format.

Usually we will send a callback within a few seconds of a Blockchain transaction being propagated on the cryptocurrency network even though it is not yet confirmed and you will see those transactions as `pending`.
