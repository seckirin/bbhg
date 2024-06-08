# Root Domain Enum

## ICP License

* 通过公司名称收集 ICP 备案和根域名
  * 网站
  * 工具
* 最新 ICP 数据:&#x20;
* 历史 ICP 记录:&#x20;

```bash
# Company Name >> ICP License and Root Domain
https://beian.miit.gov.cn/
https://www.beianx.cn/search/
https://0.zone/
https://www.qcc.com/
https://aiqicha.baidu.com/
https://shuidi.cn/
https://www.tianyancha.com/
https://icp.chinaz.com/

# https://github.com/wgpsec/ENScan_GO
./enscan-<version> -n <company_name> -type all -field icp
./enscan-<version> -f <company.txt> -type all -field icp

# 最新 ICP 数据
https://shangjibao.baidu.com

# 历史 ICP 记录
https://icp.chinaz.com/record
```

## Internal DNS <a href="#internal-dns" id="internal-dns"></a>

* **先决条件**：你需要先收集一个或多个与目标关联的根域名。
* **建议**：使用明确属于目标组织的域名作为 NS 记录查询的基础。
* **资产归属验证**：通过此方法收集到的资产可能需要进行资产归属验证。

```bash
# https://github.com/projectdiscovery/dnsx
echo <domain> | dnsx -ns -resp
cat rootdomains.txt | dnsx -ns -resp

https://hackertarget.com/find-shared-dns-servers/
https://api.hackertarget.com/findshareddns/?q=<ns1.domain.com>
```

## HTTP Header

```bash
# https://fofa.info/
domain="<domain>" && header="Content-Security-Policy-Report-Only"
domain="<domain>" && header="Content-Security-Policy"
domain="<domain>" && header="Set-Cookie"

# https://github.com/projectdiscovery/httpx
echo <website> | httpx -csp-probe -silent -json | jq -r 'try .csp.domains[]' | unfurl --unique apexes
```

### Certification

* [https://crt.sh/](https://crt.sh/)
* [https://developers.facebook.com/tools/ct/](https://developers.facebook.com/tools/ct/)
*   [https://fofa.info/](https://fofa.info/)

    ```
    (cert.subject.cn=="<domain>" || cert.subject.cn=="<*.domain>") && domain!="<domain>" && is_domain=true && cert.is_valid=true
    ```

```bash
# https://fofa.info/
(cert.subject.cn=="<domain>" || cert.subject.cn=="<*.domain>") && domain!="<domain>" && is_domain=true && cert.is_valid=true

https://crt.sh/
```

## Via Website

### Favicon Hash

```bash
# # https://github.com/yuukisec/ifavicon
ifavicon -url https://<domain.com>/favicon.ico
ifavicon -file /path/to/favicon.ico

# https://www.shodan.io/
http.favicon.hash:<favicon_hash>

# https://fofa.info/
icon_hash=<favicon_hash>

# https://github.com/xiecat/fofax
echo icon_hash=<favicon_hash> | fofax -ff domain -silent -fs 999999
```

### Google Analytics ID

此技术也适用于子域名枚举 (Subdomain Enumeration)

```
```
