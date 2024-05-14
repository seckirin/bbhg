# Domain Gathering

ICP License

```bash
# Company to ICP
https://www.qcc.com/
https://aiqicha.baidu.com/
https://shuidi.cn/
https://www.tianyancha.com/

# Latest ICP
https://shangjibao.baidu.com/businessRecommand/dynamicRecommand?type=1&source=aqcicp

# https://github.com/wgpsec/ENScan_GO
./enscan -n <company_name> -type all -field icp
# Recursive gathering
./enscan -n <company_name> -type all \
    [-invest <50>] [-deep <7>] \
    [-field {icp,weibo,wechat,app,job,wx_app,copyright,subpplier}] \
```

## Internal Name Server

<details>

<summary>The main function of Internal Name Server</summary>

It is mainly used to improve the DNS resolution efficiency of the internal network, provide more reliable resolution services, and implement specific domain name resolution for internal systems.

</details>

* **Prerequisite:** You need to first collect one or more root domains that belong to the target company.
* **Suggestion:** Use the domain name that clearly belongs to the target organization as the basis for NS record query.
* **Asset Ownership Verification:** The domain gathered through this method need to be [verified for asset ownership](../based-on-company.md#asset-ownership-verification).

```bash
# Query the Name Server of domain
# https://github.com/projectdiscovery/dnsx
echo <domain.com> | dnsx -ns -resp

# If the response result server belongs to the target organization
https://api.hackertarget.com/findshareddns/?q=<ns1.domain.com>
https://hackertarget.com/find-shared-dns-servers/
```

## HTTP Header

<details>

<summary><strong>The main function of the HTTP header Content-Security-Policy</strong></summary>

It allows website administrators to control which resources the user agent can load for a specified page to prevent cross-site scripting attacks (XSS).

</details>

<details>

<summary>The main function of the HTTP header Content-Security-Policy-Report-Only</summary>

It allows web developers to try policies by monitoring (but not enforcing) the impact of the policy. These violation reports use JSON and are sent to the specified URI via HTTP POST request.

</details>

* **Prerequisite:** You need to first collect one or more root domains that belong to the target company.
* **Asset Ownership Verification:** The domain gathered through this method need to be [verified for asset ownership](../based-on-company.md#asset-ownership-verification).

```bash
# https://fofa.info/
domain="<domain.com>" && header="Content-Security-Policy-Report-Only"
domain="<domain.com>" && header="Content-Security-Policy"
domain="<domain.com>" && header="Set-Cookie"

# https://github.com/projectdiscovery/httpx
echo <domain.com> | httpx -csp-probe -silent -json |
    jq -r 'try .csp.domains[]' | unfurl --unique apexes
```

## Certification

* **Prerequisite:** You need to first collect one or more root domains that belong to the target company.
* **Asset Ownership Verification:** The domain gathered through this method need to be [verified for asset ownership](../based-on-company.md#asset-ownership-verification).

```bash
# https://fofa.info/
cert.subject.org="<company_keyword>" && is_domain=true && cert.is_valid=true
cert.subject.cn="<domain>" && is_domain=true && cert.is_valid=true

# https://quake.360.net/
cert:"<domain.com>" AND NOT domain:<domain.com> AND is_domain:true

# https://github.com/cemulus/crt
crt -e -l 999999 -json <domain.com> |
    jq -r '.[].common_name' | unfurl -u apexes
```
