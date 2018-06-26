---
layout: default
permalink: /introduction/
title: Introduction
---
This document describes the Cubits REST API and everything that is necessary to access its resources. In order to use the API, you first need to have a verified Cubits merchant account.

## Configuration

The base URL host for all API requests documented below is:
```
https://api.cubits.com
```

All API requests are performed over HTTPS and follow [JSON API](http://jsonapi.org/) conventions. All data is sent and received as JSON with the content type `application/vnd.api+json`.

## Versioning
The Cubits REST API follows a major.minor versioning scheme. All request formats, headers and semantics are backwards compatible within the same major version, which can also be identified by looking at the request path.

Newer minor versions may add additional headers or fields in the responses or allow for more valid parameter combinations in the requests.

It is safe to use implementations for a given major version of the API with any future minor versions.
The major version of the REST API (also visible in the request path) is reserved for non backwards-compatible changes and in general you cannot use an implementation of the API for different major versions.

## Changelog

| Version | Date       | Changes                                                      |
| ------- |:----------:| ------------------------------------------------------------ |
| v1.0    | 2014-11-03 | Initial version                                              |
| v1.1    | 2015-01-09 | Added `send_money` call                                      |
| v1.2    | 2015-02-09 | Added `accounts`, `quotes`, `buy` and `sell` calls           |
| v1.3    | 2015-05-20 | Added `channels` and signed callbacks                        |
| v1.4    | 2015-06-03 | Introduce new request header format                          |
| v1.5    | 2015-10-26 | Additional `quotes` functionality, restructure documentation |
| v1.6    | 2016-01-20 | Added `quote_channels`                                       |
| v1.7    | 2016-11-07 | Added `buysend` APIs. Extendes channels options with `share_to_keep_in_btc` and `amount_before_fees` |
| v1.8    | 2017-10-12 | Changed invoices defaults, added language selection section  |
| v1.9    | 2018-06-21 | Bitcoin Cash (BCH) support, `prices` endpoint, historical balances for `accounts` endpoint |
