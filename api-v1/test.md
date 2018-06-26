---
layout: default
permalink: /api-v1/test/
title: Test
---
## GET /api/v1/test

This request is intended to be used to test if your application is configured properly and can access the Cubits API using GET requests.

### Request

Any

### Response

On success API responds with HTTP status `200 OK` and following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
status      | string(7)   | "success"

### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses).

### Example

Request:
```
GET /api/v1/test HTTP/1.1
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
  "status": "success"
}
```

---
## POST /api/v1/test

This request is intended to be used to test if your application is configured properly and can access the Cubits API using POST requests.

### Request

There are no required parameters, but you can pass any parameters to test if your signature calculation is correct.

### Response

On success API responds with HTTP status `200 OK` and following attributes:

Attribute   | Data type   | Description
------------|-------------|--------------
status      | string(7)   | "success"

### Errors

On error, the API responds with standard [error responses](/merchant-api-documentation/request_response/#error_responses).

### Example

Request:
```
POST /api/v1/test HTTP/1.1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json
X-Cubits-Key: *****
X-Cubits-Nonce: *****
X-Cubits-Signature: *****

{
  "foobar": 123.0,
  "foo": null,
  "bar": true
}
```

Response:
```
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "status": "success"
}
```
