---
layout: default
permalink: /request_response/
title: Request and Response Format
---
All API requests are performed over HTTPS and follow [JSON API](http://jsonapi.org/) conventions. All data is sent and received as JSON with the content type `application/vnd.api+json`.

## Authentication Headers

All requests to the API *MUST* include the following headers for authentication:
```
X-Cubits-Key: ...
X-Cubits-Nonce: ...
X-Cubits-Signature: ...
```

Using curl:
```
curl -H "X-Cubits-Key: ..." -H "X-Cubits-Nonce: ..." -H "X-Cubits-Signature: ..." ...
```

See [Authentication](/authentication) section for a detailed description.

## GET Requests

It is recommended to set the "Accept" header as follows:
```
Accept: application/vnd.api+json
```

Using curl:
```
curl -H "Accept: application/vnd.api+json" ...
```

## POST Requests

POST requests *MUST* send data in JSON format within request body and have a header `Content-Type: application/vnd.api+json`:
```
POST /api/v1/test
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "foobar": "1x Foo + 1x Bar",
  "foo": 12345,
  "bar": 0.75
}
```

Using curl:
```
curl -H "Content-Type: ..." -H "Accept: ..." -X POST \
     -d '{"foobar": "1x Foo + 1x Bar", "foo": 12345, "bar": 0.75}' ...
```

## Responses

All API server responses are in JSON format with a Content-Type `application/vnd.api+json`:
```
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "status": "success"
}
```

Appropriate HTTP status codes are used both for successful processing and in case of errors.

### Success Responses

Successful responses have one of the following HTTP status codes:

Situation                                          | HTTP status code
---------------------------------------------------|------------------
Request was accepted, validated and processed      | 200 OK
As above, plus a resource was created as a result  | 201 Created

### Error Responses <a name='error_responses'></a>

In case of any error the server responds with an appropriate HTTP status code and a JSON body:
```
HTTP/1.1 403 Forbidden
Content-Type: application/vnd.api+json

{
  "message": "Invalid signature"
}
```

All error responses have the following format:

key     | type   | description
--------|--------|------------
message | String | Human readable description of the error


#### Client Errors (4xx)

Situation                 | HTTP status code  | message
--------------------------|-------------------|-------------
Key is not present        | 400 Bad Request   | Key is missing
Signature is not present  | 400 Bad Request   | Signature is missing
Nonce is not present      | 400 Bad Request   | Nonce is missing
Nonce is already used     | 400 Bad Request   | Invalid nonce
Invalid request parameters| 400 Bad Request   | Invalid parameters
for some requests, more specific messages  | 400 Bad Request   | *message*
Key or signature is invalid | 403 Forbidden   | Invalid signature
Invalid/unknown path      | 404 Not Found     | Not found
Unsupported HTTP method   | 404 Not Found     | Not found
Content-Type not application/vnd.api+json | 415 Unsupported Media Type | Invalid Content-Type


#### Server Errors (5xx)

Situation                    | HTTP status code          | message
-----------------------------|---------------------------|-------------
Any internal error/exception | 500 Internal Server Error | Internal Server Error
