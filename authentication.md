---
layout: default
permalink: /authentication/
title: Authentication
---
To use the Cubits API you have to first create an API token (key + secret) in your
[Cubits merchant account](https://cubits.com/merchant#integration_tools). All requests to the API have to be authenticated using this token information in the headers.

Each valid authenticated request has to include the following HTTP headers:
```
X-Cubits-Key: ...
X-Cubits-Nonce: ...
X-Cubits-Signature: ...
```

## Cubits API key: X-Cubits-Key
`X-Cubits-Key` is the API access key which you receive when you generate an API access token. It is a sequence of hex-digits represented as a string, randomly generated when a new key is created in the web interface.

> API access keys are case-sensitive.

### Example
```HTML
X-Cubits-Key: 549653887407b9b8ad66d4b47093eb9f
```

## Nonce: X-Cubits-Nonce
`X-Cubits-Nonce` is a 64-bit **integer** number, which you must generate for every request you make to the API. This *nonce* ("number used once") has to meet the following two requirements:

> *nonce* must be unique for every request you make with the same API access key, ever. If you make a request with the same API access key and nonce again, it will be rejected.

> Every *nonce* that you generate for the request, has to be greater than any of the previous nonces that you used to make requests to the API. There is no way to reset the nonce value for a given API key but you can always just generate a new API key.

One way to generate nonces is to use the UNIX epoch timestamp of the request. Be sure, though, that you use enough precision: if you use only the *seconds* part of the timestamp and you send two requests to the API within the same second, one of them will be rejected. It is recommended that you use the UNIX epoch to microsecond resolution. The nonce value must be representable as an unsigned 64-bit integer therefore it has to be within the range [0..18446744073709551615]

### Example
```HTML
X-Cubits-Nonce: 1411754081462609
```

## Cubits API secret: CUBITS_SECRET

The secret is a randomly generated 64 character long string from the Base62 character set:
`ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789`.

Upon creation of a new API access token, the secret is displayed in the web-interface. Since it is only displayed once, you should write the secret down or copy it into your application code immediately. You can however create new API keys at every point in time.

### Example
```HTTP
CUBITS_SECRET: OJ7C3DBkMScVk89fJllpKFujspsP9aa4KVnGa3DGVQXUA5lTaBK4eWtONQEg5pAX
```

## Signature: X-Cubits-Signature

The signature is derived from the API access secret (CUBITS_SECRET), the nonce (X-Cubits-Nonce) and the request body. Its format is a **hex-string** representation of the result of the following hash calculation:

```
HMAC-SHA512(k, msg)
```

Where:

`k` is your API access secret (CUBITS_SECRET) as UTF-8 string

`msg` is a UTF-8 string, constructed by string concatenation of `uri_path`+`nonce_s`+`SHA256(request_data)`

`uri_path` is the [*path*](http://en.wikipedia.org/wiki/URI_scheme) part of the request URI as UTF-8 string

`nonce_s` is the *nonce* (X-Cubits-Nonce) of the request, converted to a UTF-8 string

`SHA256(request_data)` is the hex-encoded SHA256 digest of `request_data`, as a UTF-8 string, **downcased**

`request_data` is either the [JSON](http://tools.ietf.org/html/rfc7159) encoded request body in case of POST requests, or the URL-encoded query in case of a GET request as specified in [RFC3986](http://tools.ietf.org/html/rfc3986)

### Example 1
> Your API access key is `7287ba0902461025b01d5b99e4679018`

> Your API access secret is `93yJJ8LBDe3zNSewHBdX1XIQDjCMDIn0EKNnXrd3kfzL72fvLz99uKnXFLYuCfkt`

> You are making a POST request to `/api/v1/test`, the nonce is `123` and the POST request body is `{"attr1": 123, "attr2": "hello"}`

> *msg* is then `/api/v1/test123947753ba472927154c534cf2e4e11de27ed7a9560dc033e77d6cc24ee950ea56`

> *X-Cubits-Signature* is `d3cb2a18b754994ea7dcdc4d46cb89cb538d6533155a48f6953296680a1dc2cf7476ce7c194b2cb38231fe75afa14799b976ea61b0190afadaffe53434ea56bf`

### Example 2
> Your API access key is `3cd7a0db76ff9dca48979e24c39b408c`

> Your API access secret is `M2NkN2EwZGI3NmZmOWRjYTQ4OTc5ZTI0YzM5YjQwOGMgIC0KM2NkN2EwZGI3NmZm`

> You are making a GET request to `/api/v1/info`, the nonce is `4711` and the URL-encoded query part of the GET request URI is `first=this+is+a+field&second=was+it+clear+%28already%29%3F`

> *msg* is then `/api/v1/info471121638dfe9dd465f4eb5e31be96cebc0e1baf0966b6378949cf3653c04ad8de00`

> *X-Cubits-Signature* is `24c2a83c15581c85de5b180716bd8e86467c089665d6ab51bd6e979815e9e740a74a265d9b2aaee3db9146766583254d64280b1fbdf1e8cf91bf98ef09aff114`

### Reference Implementations

You can find open source code samples as well as full library implementations on our [Cubits github page](https://github.com/cubits).

#### Ruby 1.9+
```ruby
require 'openssl'

CUBITS_SECRET = '93yJJ8LBDe3zNSewHBdX1XIQDjCMDIn0EKNnXrd3kfzL72fvLz99uKnXFLYuCfkt'

def generate_signature(path, nonce, request_data)
  msg = path + nonce.to_s + OpenSSL::Digest::SHA256.hexdigest(request_data)
  OpenSSL::HMAC.hexdigest('sha512', CUBITS_SECRET, msg)
end

generate_signature('/api/v1/test', 123, '{"attr1": 123, "attr2": "hello"}') # => d3cb2a18b7...
```

#### Python 2.7
```python
import hmac, hashlib

CUBITS_SECRET = '93yJJ8LBDe3zNSewHBdX1XIQDjCMDIn0EKNnXrd3kfzL72fvLz99uKnXFLYuCfkt'

def generate_signature(path, nonce, request_data=''):
   msg = path + str(nonce) + hashlib.sha256(request_data).hexdigest()
   return hmac.new(CUBITS_SECRET, msg=msg, digestmod=hashlib.sha512).hexdigest()

generate_signature('/api/v1/test', 123, '{"attr1": 123, "attr2": "hello"}') # => d3cb2a18b7...
```