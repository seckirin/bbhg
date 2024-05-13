# Website Probing and Analysis

## Web 站点发现

```bash
cat resolved.txt | httpx -follow-redirects -random-agent -status-code \
-silent -retries 3 -title -web-server -tech-detect -location -no-color \
-j -o scan/httpx-webs.json
```

## 域名历史解析记录

* SecurityTrails [https://securitytrails.com/domain/vulweb.com/history/a](https://securitytrails.com/domain/vulweb.com/history/a)
* ViewDNS [https://viewdns.info/iphistory/?domain=vulnweb.com](https://viewdns.info/iphistory/?domain=vulnweb.com)
* NameLimit [https://namelimit.com/iphistory/lookup/domain/amazon\_dot\_com.name](https://namelimit.com/iphistory/lookup/domain/amazon\_dot\_com.name)
* 微步在线 [https://x.threatbook.com/v5/domain/amazon.com](https://x.threatbook.com/v5/domain/amazon.com)
* DNSDumpster [https://dnsdumpster.com/](https://dnsdumpster.com/)

## 快速获取域名解析信息

* IP138 [https://site.ip138.com/amazon.com/](https://site.ip138.com/amazon.com/) + 历史解析 A 记录
* NetCraft [https://sitereport.netcraft.com/?url=vulweb.com](https://sitereport.netcraft.com/?url=vulweb.com)
* HackerTarget [https://hackertarget.com/find-dns-host-records/](https://hackertarget.com/find-dns-host-records/)

