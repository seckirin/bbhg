# Website-based Recon

## Fingerprint Web Application (Framework) <a href="#fingerprint-web-application-framework" id="fingerprint-web-application-framework"></a>

* [https://github.com/urbanadventurer/WhatWeb](https://github.com/urbanadventurer/WhatWeb)
* [https://github.com/EdgeSecurityTeam/EHole](https://github.com/EdgeSecurityTeam/EHole)
* [https://github.com/0x727/ObserverWard](https://github.com/0x727/ObserverWard)
* [https://github.com/Tuhinshubhra/CMSeeK](https://github.com/Tuhinshubhra/CMSeeK)
* [https://github.com/zhzyker/dismap](https://github.com/zhzyker/dismap)

## Mapping the Application

**Crawling**: Use burp to listen on port `8081`

```
url=[website_full_url]
katana -u "$url" -known-files all -retry 2 -timeout 20 -proxy hyyp://127.0.0.1:8081
```
