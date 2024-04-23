# 子域名枚举

收集子域名的两种主要方式包括主动爆破子域名和被动收集子域名。被动收集为主要的收集方式，其他方式作为可选项，辅助子域名收集的全面性。

在此之前你可以适当的进行一些准备工作，以帮助你更好的进行子域名枚举

```bash
COMPANY=;TARGET=;DATETIME=$(date +%Y%m%d%H%M%S)
mkdir -p $COMPANY/$TARGET \
    $COMPANY/$TARGET/dict \
    $COMPANY/$TARGET/scan \
cd $COMPANY/$TARGET

# DNS resolver - https://github.com/trickest/resolvers
wget https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt -O dict/resolvers.txt

# Subdomain dict - https://github.com/yuukisec/hack-dict
wget https://raw.githubusercontent.com/Yuukisec/hack-dict/main/subdomains/subdomains.txt -O dict/subdomains.txt
```

## 被动收集子域名

```bash
# https://github.com/projectdiscovery/subfinder
# https://docs.projectdiscovery.io/tools/subfinder/install#post-install-configuration
# https://github.com/jqlang/jq
# https://github.com/tomnomnom/anew
# 下面是需要代理的信息源, 可以跳过这一条, 下面的条命令中直接加上 -proxy 参数
# github,censys,commoncrawl,bufferover,hackertarget,waybackarchive,facebook,anubis,digitorus
subfinder -dL roots.txt -s github,censys,commoncrawl,bufferover,\
hackertarget,waybackarchive,facebook,anubis,digitorus \
-proxy socks5://$HOST:$PORT -v -oJ -cs -o subfinder-proxy.json

# Single domain
subfinder -d target.com -all -v -oJ -cs -o scan/subfinder.json <-proxy socks5://$HOST:$PORT>
# Multiple domain
subfinder -dL roots.txt -all -v -oJ -cs -o scan/subfinder.json <-proxy socks5://$HOST:$PORT>

# Cleanup
cat scan/subfinder.json | jq -r '.host' | anew subs.txt | wc -l
```

## 主动爆破子域名

```bash
# 爆破子域名
# https://github.com/blechschmidt/massdns
# https://github.com/projectdiscovery/shuffledns
cat roots.txt | shuffledns -w dict/subdomains.txt -r dict/resolvers.txt -j -o scan/shuffledns.json
cat scan/shuffledns.json | jq -r '.hostname' | anew subs.txt | wc -l
# or
# https://github.com/robertdavidgraham/masscan
# https://github.com/d3mondev/puredns
# Single domain
puredns bruteforce dict/subdomains.txt target.com -r dict/resolvers.txt -w scan/puredns-bruteforce.txt
# Multiple domain
puredns bruteforce dict/subdomains.txt -d roots.txt -r dict/resolvers.txt -w scan/puredns-bruteforce.txt
cat scan/puredns-bruteforce.txt | anew subs.txt | wc -l
```

## 获取正常解析的域名

```bash
# https://github.com/d3mondev/puredns
puredns resolve subs.txt -r dict/resolvers.txt -w resolved.txt
```

## 获取 DNS 记录

```bash
# 获取解析子域名 DNS 记录
cat resolved.txt | dnsx -resp -retry 3 -recon -json -silent -o scan/dnsx-resolved-recon.json
# 筛选解析子域名 A 记录
cat scan/dnsx-resolved-recon.json | jq -r 'select(.a != null) | .a[]' | sort -u \
| anew scan/dnsx-resolved-a.txt | wc -l
# 筛选解析子域名 AAAA 记录
cat scan/dnsx-resolved-recon.json | jq -r 'select(.aaaa != null) | .aaaa[]' | sort -u \
| anew scan/dnsx-resolved-aaaa.txt | wc -l

# 获取所有子域名 DNS 记录
cat subs.txt | dnsx -resp -retry 3 -recon -json -o scan/dnsx-subs-recon.json
# 筛选所有子域名 A 记录
cat scan/dnsx-subs-recon.json | jq -r 'select(.a != null) | .a[]' | sort -u \
| anew scan/dnsx-subs-a.txt | wc -l
# 筛选所有子域名 AAAA 记录
cat scan/dnsx-subs-recon.json | jq -r 'select(.aaaa != null) | .aaaa[]' | sort -u \
| anew scan/dnsx-subs-aaaa.txt | wc -l
# 获取所有子域名 NXDOMAIN 记录 (用于 Host 碰撞测试)
cat subs.txt | dnsx -json -retry 3 -rc NXDOMAIN -silent | jq -r '.host' \
> scan/dnsx-subs-nxdomain.txt
```

## 生成 C 段地址

```bash
# https://github.com/yuukisec/gcidr
cat scan/dnsx-resolved-a.txt | gcidr -sa -j -o ccidr.json
```

## 反编译程序收集子域名

### 反编译 APK

```bash
# https://github.com/iBotPeaches/Apktool
apktool d application.apk
```

### 反编译小程序

```bash
# Windows 小程序存放路径
C:\Users\Administrator\Documents\WeChat Files\Applet\

# MacOS 小程序存放路径
~/Library/Containers/com.tencent.xinWeChat/Data/.wxapplet/packages
```

### 匹配域名和 IP 的正则表达式

```bash
# 域名正则表达式
(http|https)?://[a-zA-Z0-9\.\/_&=@$%?~#-]*target
(http|https)?://[a-zA-Z0-9\.\/_&=@$%?~#-]*target.com

# IP 地址正则表达式
^((2((5[0-5])|([0-4]\d)))|([0-1]?\d{1,2}))(\.((2((5[0-5])|([0-4]\d)))|([0-1]?\d{1,2}))){3}$

# 匹配语法
grep -E "{正则表达式}" -r "{反编译后的文件目录}" --color=auto
```

## 爬取站点收集子域名

ON THE WAY

## Burp 流量收集子域名

ON THE WAY
