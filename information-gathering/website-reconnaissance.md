# 网站侦查

获取域名当前解析的 IP (直接请求返回的 host 结果)

```bash
cat resolved.txt | httpx -follow-redirects -random-agent -ip -silent -retries 3 -json \
| jq '. | {url: .url, host: .host}' > scan/httpx-url-hosts.json
```

## 历史解析主机记录

* SecurityTrails [https://securitytrails.com/domain/vulweb.com/history/a](https://securitytrails.com/domain/vulweb.com/history/a)
* ViewDNS [https://viewdns.info/iphistory/?domain=vulnweb.com](https://viewdns.info/iphistory/?domain=vulnweb.com)
* NameLimit [https://namelimit.com/iphistory/lookup/domain/amazon\_dot\_com.name](https://namelimit.com/iphistory/lookup/domain/amazon\_dot\_com.name)
* 微步在线 [https://x.threatbook.com/v5/domain/amazon.com](https://x.threatbook.com/v5/domain/amazon.com)
* DNSDumpster [https://dnsdumpster.com/](https://dnsdumpster.com/)

## 快速获取主机解析信息

* IP138 [https://site.ip138.com/amazon.com/](https://site.ip138.com/amazon.com/) + 历史解析 A 记录
* NetCraft [https://sitereport.netcraft.com/?url=vulweb.com](https://sitereport.netcraft.com/?url=vulweb.com)
* HackerTarget [https://hackertarget.com/find-dns-host-records/](https://hackertarget.com/find-dns-host-records/)

## 域名快速分析
