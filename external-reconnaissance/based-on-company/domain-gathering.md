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

## Reverse DNS

* ~~通过反向 DNS 的方式收集域名具有很大的不确定性~~
* 通过反向 DNS 方式收集到的域名务必要再次进行[资产归属验证](./#verify-assets-attribution)
* 建议使用绝对归属于目标组织的域名作为基础进行 NS 查询

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
