# Based on Company

## Basic Information

```bash
https://aiqicha.baidu.com/
https://www.tianyancha.com/
https://www.qcc.com/
https://shuidi.cn/
https://www.crunchbase.com/
```

## Network Assets

### ASN and CIDR

**Network Censorship:** The network environment in China is subject to strict regulation and censorship, which may affect the querying and use of ASNs.

**IP Allocation and Management:** IP addresses in China may be managed by different network service providers, which could complicate ASN queries.

```bash
# Keyword to ASN
https://asnlookup.com/
https://bgp.he.net/
https://bgp.tools/
https://bgpview.io/

# ASN to CIDR
https://bgpview.io/

# Keyword to CIDR
https://apps.db.ripe.net/db-web-ui/
https://bgpview.io/

# https://github.com/dhn/spk
spk -s <keyword> -silent -json

# https://github.com/projectdiscovery/asnmap
asnmap [-org <keyword>] [-asn <asn_number>] [-ip <ip_address>] [-domain <domain>] \
    -json -v6 -verbose | jq -r '.'
```

### Root Domain

See the [Domain Gathering](based-on-company/domain-gathering.md) page.

### Mobile Application

```bash
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

## Acquisition Information

```bash
https://www.qcc.com/
https://aiqicha.baidu.com/
https://shuidi.cn/
https://www.tianyancha.com/

# https://github.com/wgpsec/ENScan_GO
./enscan -n <company_name> \
    [-invest <49>] [-deep <7>] \
    [-field {icp,weibo,wechat,app,job,wx_app,copyright,subpplier}] \
    [-type {aqc,tyc,all,qimai}]
```

## Asset Ownership Verification

Return to the First Item.
