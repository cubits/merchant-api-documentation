---
layout: default
permalink: /language_selection/
title: User language selection
---
English is the default language for the payment screen. If you would like to change the language that is displayed to customers, you can modify the redirect URL with the lang parameter, as below:

```
# Notice '?lang=es' part
https://pay.cubits.com/invoice/378d8ec6e305f469b009cb4e2deedf93?lang=es
```

Value of `lang` parameter should be from [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) list.

Currently supported following languages

|Parameter value | Language |
|:--------:|------|
| `en` | English|
| `ja` | Japanese|
| `ko` | Korean|
| `zh` | Chinese|
| `es` | Spanish|
| `ru` | Russian|
| `no` | Norwegian|
| `sv` | Swedish|
| `fi` | Finnish|
| `tr` | Turkish|

