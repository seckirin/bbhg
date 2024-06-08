# Internet Asset Discovery

[Contents](../readme/table-of-contents.md) > [Based on Company](broken-reference) > [**Internet Asset Enumeration**](internet-asset-enum-bak.md)

## ASN / CIDR Collection

```bash
https://bgp.he.net/
https://bgp.tools/
https://bgpview.io/
https://asnlookup.com/
https://ipwhois.cnnic.net.cn/

# https://github.com/dhn/spk
spk -s <keyword> -silent -json

# https://github.com/projectdiscovery/asnmap
asnmap [-org <keyword>] [-asn <asn_number>] [-ip <ip_address>] [-domain <domain>] \
    -json -v6 -verbose | jq -r '.'
```

## Domain Enumeration

### ICP License

```bash
# Company >> ICP license
https://www.qcc.com/
https://aiqicha.baidu.com/
https://shuidi.cn/
https://www.tianyancha.com/

# Latest ICP
https://shangjibao.baidu.com/businessRecommand/dynamicRecommand?type=1&source=aqcicp

# ICP history
https://icp.chinaz.com/record/

# https://github.com/wgpsec/ENScan_GO
# ./enscan -n <company_name> -type all -field icp
./enscan -n <company_name> -type all \
    [-invest <50>] [-deep <7>] \
    [-field {icp,weibo,wechat,app,job,wx_app,copyright,subpplier}]
```

### Internal DNS

* **Prerequisite:** You need to first collect one or more root domains that belong to the target company.
* **Suggestion:** Use the domain name that clearly belongs to the target organization as the basis for NS record query.
* **Asset Ownership Verification:** The domain gathered through this method need to be [verified for asset ownership](broken-reference).

```bash
# Query the Name Server of domain
# https://github.com/projectdiscovery/dnsx
echo <domain.com> | dnsx -ns -resp

# If the response result server belongs to the target organization
https://api.hackertarget.com/findshareddns/?q=<ns1.domain.com>
https://hackertarget.com/find-shared-dns-servers/
```

### HTTP Header

<details>

<summary><strong>The main function of the HTTP header Content-Security-Policy</strong></summary>

It allows website administrators to control which resources the user agent can load for a specified page to prevent cross-site scripting attacks (XSS).

</details>

<details>

<summary>The main function of the HTTP header Content-Security-Policy-Report-Only</summary>

It allows web developers to try policies by monitoring (but not enforcing) the impact of the policy. These violation reports use JSON and are sent to the specified URI via HTTP POST request.

</details>

* **Prerequisite:** You need to first collect one or more root domains that belong to the target company.
* **Asset Ownership Verification:** The domain gathered through this method need to be [verified for asset ownership](broken-reference).

```bash
# https://fofa.info/
domain="<domain.com>" && header="Content-Security-Policy-Report-Only"
domain="<domain.com>" && header="Content-Security-Policy"
domain="<domain.com>" && header="Set-Cookie"

# https://github.com/projectdiscovery/httpx
echo <domain.com> | httpx -csp-probe -silent -json |
    jq -r 'try .csp.domains[]' | unfurl --unique apexes
```

### Certification

* **Prerequisite:** You need to first collect one or more root domains that belong to the target company.
* **Asset Ownership Verification:** The domain gathered through this method need to be [verified for asset ownership](broken-reference).

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

### Favicon Hash

* **Prerequisite:** You need to first collect one or more root domains that belong to the target company. And the accessible website needs to have a favicon icon
* **Asset Ownership Verification:** The domain gathered through this method need to be [verified for asset ownership](broken-reference).

```bash
# Computing the hash value of the URL icon or file
# https://github.com/yuukisec/ifavicon
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

* **Reverse Google Analytics ID Search WebSites:**
  * [https://intelx.io/tools?tab=analytics](https://intelx.io/tools?tab=analytics)
    * [https://search.dnslytics.com/search?q=html.tag:ua-33427076\&d=domains](https://search.dnslytics.com/search?q=html.tag:ua-33427076\&d=domains)
    * [https://hackertarget.com/reverse-analytics-search/](https://hackertarget.com/reverse-analytics-search/)
    * [https://publicwww.com/websites/%22ua-33427076-%22/](https://publicwww.com/websites/%22ua-33427076-%22/)
    * [https://analyzeid.com/id/UA-33427076](https://analyzeid.com/id/UA-33427076)
  * [https://osint.sh/analytics](https://osint.sh/analytics)
  * [https://builtwith.com/relationships/tag/UA-33427076](https://builtwith.com/relationships/tag/UA-33427076)
  * [http://site-overview.com/website-report-search/analytics-account-id/33427076](http://site-overview.com/website-report-search/analytics-account-id/33427076)
  * [https://spyonweb.com/UA-33427076](https://spyonweb.com/UA-33427076)
* **Prerequisite:** You need to first collect one or more root domains that belong to the target company. And the Website that can be accessed is needed.
* **Asset Ownership Verification:** The domain gathered through this method need to be [verified for asset ownership](broken-reference).
* **Subdomain Enumeration:** This method is also applicable to [subdomain enumeration](broken-reference).

```bash
# https://github.com/projectdiscovery/nuclei
# https://gist.github.com/yuukisec/f6b4659b5d4c2d825a6d41a8ae3d73b7
nuclei -t $TOOLS/nuclei-templates/google-analytics-id-detection.yaml \
    -l roots.txt -silent -no-color | cut -d '"' -f2 > google_analytics_id.txt

# https://github.com/dhn/udon
while read id
do
    udon -s $id -json -silent | jq -r '.domain'
done < google_analytics_id.txt
```

## Mobile applications

### Android / iOS

```bash
# Websites
https://www.qimai.cn/
https://www.diandian.com/

# https://0.zone/
(type=安卓APK&&company==<keyword>)
(type=安卓APK&&company=<company_name>)
```

### WeChat Applet

```bash
# https://0.zone/
(type=微信小程序&&company==<keyword>)
(type=微信小程序&&company=<company_name>)
```
