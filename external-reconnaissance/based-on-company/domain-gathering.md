# Domain Gathering

## ICP License

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

* The domain gathered through the internal name server method must be [verified for asset ownership](../based-on-company.md#asset-ownership-verification).
* It is recommended to use the domain name that clearly belongs to the target organization as the basis for NS record query.

```bash
# https://github.com/projectdiscovery/dnsx
echo <target.com> | dnsx -ns -resp

# If the response result server belongs to the target organization
https://hackertarget.com/find-shared-dns-servers/
# Or
https://api.hackertarget.com/findshareddns/?q=<ns1.target.com>
```

## HTTP Header

## Favicon Hash

In-house Name Server
