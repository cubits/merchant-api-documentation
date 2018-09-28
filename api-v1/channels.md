---
layout: default
permalink: /api-v1/channels/
title: Channels
---
Cubits channels allow you to receive and convert cryptocurrency payments without setting a validity period. Cryptocurrency sent to a channel can be automatically exchanged into any of Cubits’ supported fiat currencies at the spot rate at the time of the payment. Channels have a fixed payment address and support callbacks. A payment screen can also be generated so users are able to conveniently send funds to your address. Cryptocurrency payments that amount to less than the smallest possible unit for the chosen fiat currency will not be converted. Instead, they will be transferred to your Cubits cryptocurrency wallet. You are also able to view a channel’s list of recorded transactions to track individual payments. Possible use cases for Cubits channels are accepting donations on a website or simply keeping track of payments to a particular address.

## GET /api/v1/channels/{channel_id}

Get information about an existing channel.

### Request

None

### Response

On success, a response with HTTP status `200 OK` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
id          | string(32)  | Unique hex-string identifier of the channel
address     | string(34)  | Default address associated with this channel (address currency is defined by `sender_currency` attribute)
alt_addresses     | array     | Array of *address* objects
sender_currency | string(3)   | Code of the cryptocurrency accepted by the channel
receiver_currency | string(3) | Code of the currency that you want to receive
name        | string(256) | Name of the channel, displayed to the customer on the payment screen
description | string(512) | Description of the item displayed to the customer on the payment screen
reference   | string(512) | Individual free-text field stored in the channel as-is
channel_url | string(512) | Unique URL of the channel payment screen on Cubits
callback_url| string(512) | URL that is called on channel updates
txs_callback_url| string(512) | URL that is called on channel transaction updates
success_url | string(512) | URL to redirect the user to after a successful payment
share_to_keep_in_btc | number | Percentage of the each transaction to receive in `sender_currency`, as a number from 0 to 100.
created_at  | number      | (float) Unix-epoch timestamp of the channel creation
updated_at  | number      | (float) Unix-epoch timestamp when the channel data was last updated

### Address objects

Attribute | Data type  | Description
----------|------------|--------------
address   | string(64) | A formatted crypto-address
type      | string(64) | The name of the format of the address (e.g. `base58` or `cash_addr`)
default   | bool       | Whether this address is the default address format, i.e. the `address` attribute of the *channel* object

### Errors

On error, the API responds with standard [error responses](/request_response/#error_responses).

### Example

Request:
```
GET /api/v1/channels/af9c1f5ae2c51e105e05e75ab716a2067 HTTP/1.1
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
  "receiver_currency": "EUR",
  "name": "Donation payment channel",
  "description": null,
  "reference": "xyz",
  "callback_url": "https://example.com/callback",
  "txs_callback_url": "https://example.com/txs_callback",
  "success_url": "https://example.com/thank_you.html",
  "created_at": 1427137017.0,
  "updated_at": 1427138200.0,
  "id": "af9c1f5ae2c51e105e05e75ab716a206",
  "channel_url": "https://pay.cubits.com/channels/af9c1f5ae2c51e105e05e75ab716a206",
  "address": "34MDjoUDkK35DKKwy6Lk5zkLf84B88TPzv",
  "alt_addresses": [
    {
      "address": "34MDjoUDkK35DKKwy6Lk5zkLf84B88TPzv",
      "type": "base58",
      "default": true
    }
  ],
  "share_to_keep_in_btc": "0"
}
```

## GET /api/v1/channels/{channel_id}/txs

Get information about all transactions of an existing channel.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
page        | number      | *(optional)* Page number to return (default: `1`)
per_page    | number      | *(optional)* Number of transactions to return per page (max: `1000`, default: `100`)

### Response

On success, a response with HTTP status `200 OK` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
pagination  | object      | *Pagination* object for the returned txs array
txs         | array       | Array of *transaction* objects of this channel sorted by descending *updated_at* field

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
channel_id  | string(32)  | Id of the channel this transaction belongs to
state       | string(9)   | `pending`, `completed` or `cancelled`
created_at  | number      | (float) Unix-epoch timestamp of the transaction creation
updated_at  | number      | (float) Unix-epoch timestamp when the transaction data was last updated
sender      | object      | Information about the *sender* part of this transaction
receiver    | object      | Information about the *receiver* part of this transaction

Unconfirmed transactions will stay in the `pending` state until Cubits deems the risk to accept them as low enough for them to go in the `completed` state.

A transaction that was a double-spend or one that will not confirm for other reasons (e.g. because of the dust limit), will eventually go to `cancelled` state.

#### Sender Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that was sent
amount      | string(17)  | Amount that was sent
bitcoin_txid | string(64) | Transaction id of this transaction or `null` in case of a Cubits internal transfer

#### Receiver Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that was received. Note that this may be crypto for fiat channels in case less than the equivalent of the smallest fiat currency unit was sent.
amount      | string(17)  | Amount that was received. Note that this amount is preliminary in case of `pending` transactions and might change once the transaction goes to `completed` state.
amount_plus_fees      | string(17)  | Amount that was received plus fees paid. Note that this amount is preliminary in case of `pending` transactions and might change once the transaction goes to `completed` state.

### Errors

On error, the API responds with standard [error responses](/request_response/#error_responses).

### Example

Request:
```
GET /api/v1/channels/af9c1f5ae2c51e105e05e75ab716a2067/txs HTTP/1.1
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
    "tx_ref_code": "MH5AD",
    "channel_id": "af9c1f5ae2c51e105e05e75ab716a2067",
    "state": "completed",
    "created_at": 1427137517.0,
    "updated_at": 1427137623.0,
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
  },
  {
    "tx_ref_code": "375X4",
    "channel_id": "af9c1f5ae2c51e105e05e75ab716a2067",
    "state": "pending",
    "created_at": 1427138127.0,
    "updated_at": 1427138127.0,
    "sender": {
      "currency": "BTC",
      "amount": "0.00010000",
      "bitcoin_txid": "d25100ff30af7ad87f5da8a509c4b6a51df22a27b02bcb4a5567b66d7913d47f"
    },
    "receiver": {
      "currency": "EUR",
      "amount": "0.02",
      "amount_plus_fees": "0.03"
    }
  },
  {
    "tx_ref_code": "B5CU5",
    "channel_id": "af9c1f5ae2c51e105e05e75ab716a2067",
    "state": "pending",
    "created_at": 1427138200.0,
    "updated_at": 1427138200.0,
    "sender": {
      "currency": "BTC",
      "amount": "0.00005430",
      "bitcoin_txid": "a4521e87cb2e610f79e2a4e55bdb07d416a5844d0b58ad02be57bf2fdee06778"
    },
    "receiver": {
      "currency": "BTC",
      "amount": "0.00005430",
      "amount_plus_fees": "00005431"
    }
  }
  ]
}
```

---
## GET /api/v1/channels/{channel_id}/txs/{tx_ref_code}

Get information about an individual transactions of a channel.

### Request

None

### Response

On success, a response with HTTP status `200 OK` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
tx_ref_code | string(6)   | Unique character string to reference this transaction
channel_id  | string(32)  | Id of the channel this transaction belongs to
state       | string(9)   | `pending`, `completed` or `cancelled`
created_at  | number      | (float) Unix-epoch timestamp of the transaction creation
updated_at  | number      | (float) Unix-epoch timestamp when the transaction data was last updated
sender      | object      | Information about the *sender* part of this transaction
receiver    | object      | Information about the *receiver* part of this transaction

Unconfirmed transactions will stay in the `pending` state until Cubits deems the risk to accept them as low enough for them to go in the `completed` state.
A transaction that was a double-spend or one that will not confirm for other reasons (e.g. because of the dust limit), will eventually go to `cancelled` state.

#### Sender Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that was sent
amount      | string(17)  | Amount that was sent
bitcoin_txid | string(64) | Transaction id of this transaction or `null` in case of a Cubits internal transfer

#### Receiver Object

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that was received. Note that this will be crypto for fiat channels in case less than the equivalent of the smallest fiat currency unit of the fiat currency was sent.
amount      | string(17)  | Amount that was received. Note that this amount is preliminary in case of `pending` transactions and might change once the transaction goes to `completed` state.

### Errors

On error, the API responds with standard [error responses](/request_response/#error_responses).

### Example

Request:
```
GET /api/v1/channels/af9c1f5ae2c51e105e05e75ab716a2067/txs/MH5AD HTTP/1.1
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
  "tx_ref_code": "MH5AD",
  "channel_id": "af9c1f5ae2c51e105e05e75ab716a2067",
  "state": "completed",
  "created_at": 1427137517.0,
  "updated_at": 1427137623.0,
  "sender": {
    "currency": "BTC",
    "amount": "0.00010000",
    "bitcoin_txid": "9ae311aae58c9150ca22f5b3fa7b64e32bc4e25cf9db11ba6bd28de50bcd1f73"
  },
  "receiver": {
    "currency": "EUR",
    "amount": "0.02"
  }
}
```

---
## POST /api/v1/channels

Creates a new channel.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
sender_currency | string(3) | *(optional)* A code of the cryptocurrency that is accepted in the channel (see [List of supported cryptocurrencies](/appendices/#supported_cryptocurrencies), default: `BTC`)
receiver_currency | string(3) | Code of the currency that you want to receive (see [List of supported currencies](/appendices/#supported_fiat_currencies) and [List of supported cryptocurrencies](/appendices/#supported_cryptocurrencies))
name        | string(256) | *(optional)* Name of the channel, displayed to the customer on the payment screen
description | string(512) | *(optional)* Description of the item displayed to the customer on the payment screen
reference   | string(512) | *(optional)* Individual free-text field stored in the channel as-is
callback_url| string(512) | *(optional)* URL that is called on channel status updates
txs_callback_url| string(512) | *(optional)* URL that is called on channel transaction updates
success_url | string(512) | *(optional)* URL to redirect the user to after a successful payment
share_to_keep_in_btc | string(16) | *(optional)* Percentage of the each transaction to receive in `sender_currency`, as a decimal number, converted to string (e.g. `20`).

Note that conversion between cryptocurrencies aren't supported yet, meaning if `receiver_currency` is set to `BCH`, `sender_currency` **must** also be explicitly set to `BCH`.

### Response

On success, a response with HTTP status `201 Created` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
id          | string(32)  | Unique hex-string identifier of the channel
address     | string(34)  | Address of the cryptocurrency associated with this channel
alt_addresses     | array     | Array of *address* objects
sender_currency   | string(3) | Code of the cryptocurrency that is accepted in the channel (e.g. `BTC`)
receiver_currency | string(3) | Code of the currency that you want to receive (e.g. `EUR`)
name        | string(256) | Name of the channel, displayed to the customer on the payment screen
description | string(512) | Description of the item displayed to the customer on the payment screen
reference   | string(512) | Individual free-text field stored in the channel as-is
channel_url | string(512) | Unique URL of the channel payment screen on Cubits
callback_url| string(512) | URL that is called on channel status updates
txs_callback_url| string(512) | URL that is called on channel transaction updates
success_url | string(512) | URL to redirect the user to after a successful payment
share_to_keep_in_btc | string(16) | Percentage of the each transaction to receive in `sender_currency`, as a decimal number, converted to string (e.g. `20`).
created_at  | number      | (float) Unix-epoch timestamp of the channel creation
updated_at  | number      | (float) Unix-epoch timestamp when the channel data was last updated

### Address objects

Attribute | Data type  | Description
----------|------------|--------------
address   | string(64) | A formatted crypto-address
type      | string(64) | the name of the format of the address (e.g. `base58` or `cash_addr`)
default   | bool       | whether this address is the default address format, i.e. the `address` attribute of the *channel* object

### Errors

On error, the API responds with standard [error responses](/request_response/#error_responses) and with one specific to this request:

#### Client errors (4xx)

Situation                 | HTTP status code  | message
--------------------------|-------------------|-------------
Requested currency is not supported | 400 Bad Request   | Unsupported currency


### Example

Request:
```
POST /api/v1/channels HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****

{
  "sender_currency": "BCH",
  "receiver_currency": "EUR",
  "name": "Donation payment channel",
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
  "receiver_currency": "EUR",
  "name": "Donation payment channel",
  "description": null,
  "reference": "xyz",
  "callback_url": "https://example.com/callback",
  "txs_callback_url": null,
  "success_url": "https://example.com/thank_you.html",
  "id": "f5790652fb26b62bb48e65941bec06f8",
  "channel_url": "https://pay.cubits.com/channels/f5790652fb26b62bb48e65941bec06f8",
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
  "created_at": 1427217218.0,
  "updated_at": 1427217218.0
}
```

## POST /api/v1/channels/{channel_id}

Updates an existing channel.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
receiver_currency | string(3) | *(optional)* Code of the new currency that you want to receive (see [List of supported currencies](/appendices/#supported_fiat_currencies) and [List of supported cryptocurrencies](/appendices/#supported_cryptocurrencies))
name        | string(256) | *(optional)* New name of the channel, displayed to the customer on the payment screen
description | string(512) | *(optional)* New description of the item displayed to the customer on the payment screen
reference   | string(512) | *(optional)* New free-text field stored in the channel as-is
callback_url| string(512) | *(optional)* New URL that is called on channel status updates
txs_callback_url| string(512) | *(optional)* URL that is called on channel transaction updates
success_url | string(512) | *(optional)* New URL to redirect the user to after a successful payment
share_to_keep_in_btc | string(16) | *(optional)* Percent of the each transaction to receive in `sender_currency`, as a decimal number, converted to string (e.g. "20")

### Response

On success, a response with HTTP status `200 OK` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
id          | string(32)  | Unique hex-string identifier of the channel
address     | string(34)  | Address of the cryptocurrency associated with this channel
alt_addresses     | array     | Array of *address* objects
receiver_currency | string(3) | [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217#Active_codes) code of the currency that you want to receive (e.g. "EUR")
name        | string(256) | Name of the channel, displayed to the customer on the payment screen
description | string(512) | Description of the item displayed to the customer on the payment screen
reference   | string(512) | Individual free-text field stored in the channel as-is
channel_url | string(512) | Unique URL of the channel payment screen on Cubits
callback_url| string(512) | URL that is called on channel status updates
txs_callback_url| string(512) | URL that is called on channel transaction updates
success_url | string(512) | URL to redirect the user to after a successful payment
share_to_keep_in_btc | string(16) | Percent of the each transaction to receive in `sender_currency`, as a decimal number, converted to string (e.g. "20")
created_at  | number      | (float) Unix-epoch timestamp of the channel creation
updated_at  | number      | (float) Unix-epoch timestamp when the channel data was last updated

### Address objects

Attribute | Data type  | Description
----------|------------|--------------
address   | string(64) | A formatted crypto-address
type      | string(64) | the name of the format of the address (e.g. `base58` or `cash_addr`)
default   | bool       | whether this address is the default address format, i.e. the `address` attribute of the *channel* object

### Errors

On error, the API responds with standard [error responses](/request_response/#error_responses) and with one specific to this request:

#### Client errors (4xx)

Situation                 | HTTP status code  | message
--------------------------|-------------------|-------------
Requested currency is not supported | 400 Bad Request   | Unsupported currency

### Example

Request:
```
POST /api/v1/channels/f5790652fb26b62bb48e65941bec06f8 HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****

{
  "receiver_currency": "BTC",
  "share_to_keep_in_btc": "10"
}
```

Response:
```
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "receiver_currency": "BTC",
  "name": "Donation payment channel",
  "description": null,
  "reference": "xyz",
  "callback_url": "https://example.com/callback",
  "txs_callback_url": null,
  "success_url": "https://example.com/thank_you.html",
  "id": "f5790652fb26b62bb48e65941bec06f8",
  "channel_url": "https://pay.cubits.com/channels/f5790652fb26b62bb48e65941bec06f8",
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
  "updated_at": 1427217219.0
}
```

## Callbacks

When specifying a `callback_url`, the server will send callbacks to inform you about changes to any of the channel fields. The POSTed request body will correspond to the result of a GET request to /api/v1/channels/{channel_id}.

When specifying a `txs_callback_url`, the server will send callbacks to inform you about changes to any of the channel's transactions. The POSTed request body will correspond to the result of a GET request to /api/v1/channels/{channel_id}/txs/{tx_ref_code}.

See [Callbacks](/callback/) for a general description of the Cubits callback mechanism and format.

Usually we will send a callback within a few seconds of a transaction being propagated on the blockchain even though it is not yet confirmed and you will see those transactions as `pending`.
