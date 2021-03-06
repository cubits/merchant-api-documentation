---
layout: default
permalink: /api-v1/invoices/
title: Invoices
---
Cubits invoices are a convenient way to accept cryptocurrency payments. An invoice is created by specifying a certain amount and currency you want to receive. You can choose any of Cubits’ supported crypto or fiat currencies. A quote is created and the price in cryptocurrency is calculated. Each invoice has a unique payment address and a payment screen URL with a QR code. The Cubits payment screen also supports directly logging in to a Cubits account to facilitate single-click payments for Cubits users. Each invoice has a limited validity period (15 minutes for fiat currency invoices) during which they can be paid and the (optional) conversion to fiat will be performed. Cryptocurrency payments received after the validity period will be treated as regular deposits.

Cubits operates with two types of invoices: refundable and non-refundable. Non-refundable invoices deposit any funds that are left after conversion. This includes underpaid, unsuccessful invoices as well as overpaid amounts. Refundable invoices enable the customer who received the invoice link to claim excess funds. Since version 1.8 of our API, newly created merchant accounts will have invoices refundable by default. If your account was created before and you wish to have refundable invoices, please contact your account manager.

## Invoice status

Status    | Explanation
:--------:|-------------------------
pending   | Initial state, invoice has been created but no payment was received yet
completed | Success state, invoice has been fully paid
overpaid  | Success state, invoice has been fully paid but additional funds were received
underpaid | Intermediary state, insufficient payment was received
aborted   | Failure state, payment was cancelled by the customer
timeout   | Failure state, no sufficient payment was received before the invoice expired

All invoices with `underpaid` status can be completed, overpaid or timeout depending on user actions.

### State transitions

![State transitions](https://static.cubits.com/state_transition.svg "All possible state transitions of an invoice")

## GET /api/v1/invoices/{invoice_id}

Get information about an existing invoice.

### Request

None

### Response

On success, a response with HTTP status `200 OK` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
id          | string(32)  | Unique hex-string identifier of the invoice
status      | string(11)  | "pending", "completed", "overpaid", "aborted", "timeout" or "underpaid"
address     | string(34)  | Address of the cryptocurrency displayed to the customer
alt_addresses     | array     | Array of *address* objects
merchant_currency | string(3)   | Code of the currency that the merchant wants to receive (e.g. "EUR")
merchant_amount   | string(16)  | Amount of currency the merchant wants to receive, as a decimal floating point number, converted to string (e.g. "123.05")
invoice_currency  | string(3)   | Code of the currency that the customer was requested to pay (e.g. "BTC")
invoice_amount    | string(16)  | Amount of the invoice currency that the customer was requested to pay
paid_currency     | string(3)   | Code of the currency that was paid by the customer
paid_amount       | string(16)  | Amount of currency that was paid by the customer
pending_currency     | string(3)   | Code of the pending amount currency
pending_amount       | string(16)  | Total amount of the unconfirmed incoming payments that were sent by the customer
share_to_keep_in_btc | number | Percent of the invoice amount to be kept in `invoice_currency`, as a number from 0 to 100.
name        | string(256) | Name of the item displayed to the customer
description | string(512) | Description of the item displayed to the customer
reference   | string(512) | Individual free-text field stored in the invoice as-is
invoice_url | string(512) | Unique URL of the invoice payment page on Cubits
callback_url| string(512) | URL that is called on invoice status updates
success_url | string(512) | URL to redirect the customer to after a successful payment
cancel_url  | string(512) | URL to redirect the customer to if the invoice expires
notify_email| string(256) | e-mail address to be notified about payments
create_time | number      | (float) Unix-epoch time of the invoice creation
valid_until_time | number | (float) Unix-epoch timestamp after which the invoice will expire

### Address objects

Attribute | Data type  | Description
----------|------------|--------------
address   | string(64) | A formatted crypto-address
type      | string(64) | The name of the format of the address (e.g. `base58` or `cash_addr`)
default   | bool       | Whether this address is the default address format, i.e. the `address` attribute of the *invoice* object

### Errors

On error, the API responds with standard [error responses](/request_response/#error_responses).

### Example

Request:
```
GET /api/v1/invoices/a7722658ed65c8920c16ac20757b307b HTTP/1.1
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
  "merchant_currency": "EUR",
  "merchant_amount": "266.45",
  "invoice_currency": "BTC",
  "invoice_amount": "0.04529500",
  "paid_currency": "BTC",
  "paid_amount": "0.00000000",
  "pending_currency": "BTC",
  "pending_amount": "0.00000000",
  "share_to_keep_in_btc": 0,
  "name": "Guatemala Joe",
  "description": "Twelve ounces of high quality, organic Guatemalan coffee",
  "reference": "d0cef2",
  "callback_url": "https://my.shop.online/callback",
  "success_url": "https://my.shop.online/success",
  "cancel_url": "https://my.shop.online/cancel",
  "notify_email": "payments@my.shop.online",
  "id": "a7722658ed65c8920c16ac20757b307b",
  "status": "pending",
  "invoice_url": "https://pay.example.org/invoice/a7722658ed65c8920c16ac20757b307b",
  "address": "1AeMbkpHia8FVuKczQKUrv9uMzv7uC1HZi",
  "alt_addresses": [
    {
      "address": "1AeMbkpHia8FVuKczQKUrv9uMzv7uC1HZi",
      "type": "base58",
      "default": true
    }
  ],
  "valid_until_time": 1527865073.0,
  "create_time": 1527864172.0
}
```

---
## POST /api/v1/invoices

Creates a new invoice.

### Request

Attribute   | Data type   | Description
------------|-------------|--------------
currency    | string(3)   | Code of the currency that you want to receive (see [List of supported currencies](/appendices/#supported_fiat_currencies) and [List of supported cryptocurrencies](/appendices/#supported_cryptocurrencies))
price       | string(16)  | Price of the invoice that the merchant wants to receive, as a decimal floating point number, converted to string (e.g. "123.05")
invoice_currency  | string(3)   | *(optional)* Code of the cryptocurrency that the customer will be requested to pay (see [List of supported cryptocurrencies](/appendices/#supported_cryptocurrencies), default: `BTC`)
share_to_keep_in_btc        | number | *(optional)* Percentage of the invoice total to receive in `invoice_currency`, as an integer number from 0 to 100. If not specified, a default value is used from the Cubits Pay / Payouts / Percentage Kept in BTC
name        | string(256) | *(optional)* Name of the item displayed to the customer
description | string(512) | *(optional)* Description of the item displayed to the customer
reference   | string(512) | *(optional)* Individual free-text field stored in the invoice as-is
callback_url| string(512) | *(optional)* URL that is called on invoice status updates
success_url | string(512) | *(optional)* URL to redirect the customer to after a successful payment
cancel_url  | string(512) | *(optional)* URL to redirect the customer to if the invoice expires
notify_email| string(256) | *(optional)* e-mail address to be notified about payments


Note that conversion between cryptocurrencies aren't supported yet, meaning if `currency` is set to `BCH`, `invoice_currency` **must** also be explicitly set to `BCH`.

If any of the optional URLs are omitted in the request, the values set in the merchant profile will be used as defaults.

### Response

On success, a response with HTTP status `201 Created` is returned containing the following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
id          | string(32)  | Unique hex-string identifier of the created invoice
status      | string(11)  | "pending"
address     | string(34)  | Crypto address displayed to the customer
alt_addresses     | array     | Array of *address* objects
merchant_currency | string(3)   | Code of the currency that the merchant wants to receive (e.g. "EUR")
merchant_amount   | string(16)  | Amount of currency the merchant wants to receive, as a decimal floating point number, converted to string (e.g. "123.05")
invoice_currency  | string(3)   | Code of the currency that the customer was requested to pay (e.g. "BTC")
invoice_amount    | string(16)  | Amount of the invoice currency that the customer was requested to pay
paid_currency     | string(3)   | Code of the currency that was paid by the customer
paid_amount       | string(16)  | Amount of currency that was paid by the customer
pending_currency     | string(3)   | Code of the pending amount currency
pending_amount       | string(16)  | Total amount of the unconfirmed incoming payments that were sent by the customer
share_to_keep_in_btc | number | Percentage of the invoice total to receive in `invoice_currency`, as an integer number from 0 to 100
name        | string(256) | Name of the item displayed to the customer
description | string(512) | Description of the item displayed to the customer
reference   | string(512) | Individual free-text field stored in the invoice as-is
invoice_url | string(512) | Unique URL of the invoice payment page on Cubits
callback_url| string(512) | URL that is called on invoice status updates
success_url | string(512) | URL to redirect the customer to after a successful payment
cancel_url  | string(512) | URL to redirect the customer to if the invoice expires
notify_email| string(256) | e-mail address to be notified about payments
create_time | number      | (float) Unix-epoch time of the invoice creation
valid_until_time | number | (float) Unix-epoch timestamp after which the invoice will expire

### Address objects

Attribute | Data type  | Description
----------|------------|--------------
address   | string(64) | A formatted crypto-address
type      | string(64) | The name of the format of the address (e.g. `base58` or `cash_addr`)
default   | bool       | Whether this address is the default address format, i.e. the `address` attribute of the *invoice* object

### Errors

On error API responds with standard [error responses](/request_response/#error_responses)
and with one specific to this request:

#### Client errors (4xx)

Situation                 | HTTP status code  | message
--------------------------|-------------------|-------------
Requested currency is not supported | 400 Bad Request   | Unsupported currency


### Example

Request:
```
POST /api/v1/invoices HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****

{
  "currency": "EUR",
  "price": "266.45",
  "name": "Guatemala Joe",
  "description": "Twelve ounces of high quality, organic Guatemalan coffee",
  "reference": "d0cef2"
}
```

Response:
```
HTTP/1.1 201 Created
Content-Type: application/vnd.api+json

{
  "id": "378d8ec6e305f469b009cb4e2deedf93",
  "status": "pending",
  "address": "1AeMbkpHia8FVuKczQKUrv9uMzv7uC1HZi",
  "alt_addresses": [
    {
      "address": "1AeMbkpHia8FVuKczQKUrv9uMzv7uC1HZi",
      "type": "base58",
      "default": true
    }
  ],
  "merchant_currency": "EUR",
  "merchant_amount": "266.45",
  "invoice_currency": "BTC",
  "invoice_amount": "0.04529500",
  "paid_currency": "BTC",
  "paid_amount": "0.00000000",
  "pending_currency": "BTC",
  "pending_amount": "0.00000000",
  "share_to_keep_in_btc": 0,
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

## Callbacks

When specifying a `callback_url`, invoices will send callbacks to inform you about changes to any of the invoice fields. See [Callbacks](/callback/) for a general description of the Cubits callback mechanism and format.

Usually we will send a callback within a few seconds of a crypto transaction being propagated on the network even though it is not yet confirmed. In some rare cases we might determine that the risk for a double spend attack is higher and wait until we accept a payment transaction. In those cases you will receive the callback with some delay of the actual payment.
