# 子域名枚举

如果对操作中的目录或文件存在疑问，请查询页面底部的 "目录及文件说明"。

收集子域名的两种主要方式包括主动爆破子域名和被动收集子域名。被动收集为主要的收集方式，其他方式作为可选项，辅助子域名收集的全面性。

在此之前你可以适当的进行一些准备工作，以帮助你更好的进行子域名枚举。

```bash
# Initialize environment
COMPANY=;TARGET=;DATETIME=$(date +%Y%m%d%H%M%S)
mkdir -p $COMPANY/$TARGET \
    $COMPANY/$TARGET/dict \
    $COMPANY/$TARGET/scan \
cd $COMPANY/$TARGET

# DNS resolver - https://github.com/trickest/resolvers
wget https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt -O dict/resolvers.txt

# Subdomain dict - https://github.com/yuukisec/hackdict
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
-proxy socks5://ip:port -v -oJ -cs -o subfinder-proxy.json

# Single domain
subfinder -d target.com -all -v -oJ -cs -o scan/subfinder.json
# Multiple domain
subfinder -dL roots.txt -all -v -oJ -cs -o scan/subfinder.json

# Cleanup
cat scan/subfinder.json | jq -r '.host' | anew subs.txt
```

## 主动爆破子域名

```bash
# 爆破子域名
# https://github.com/blechschmidt/massdns
# https://github.com/projectdiscovery/shuffledns
cat roots.txt | shuffledns -w dict/subdomains.txt -r dict/resolvers.txt -j -o scan/shuffledns.json
cat scan/shuffledns.json | jq -r '.hostname' | anew subs.txt
# or
# https://github.com/robertdavidgraham/masscan
# https://github.com/d3mondev/puredns
# Single domain
puredns bruteforce dict/subdomains.txt target.com -r dict/resolvers.txt -w scan/puredns-bruteforce.txt
# Multiple domain
puredns bruteforce dict/subdomains.txt -d roots.txt -r dict/resolvers.txt -w scan/puredns-bruteforce.txt
cat scan/puredns-bruteforce.txt | anew subs.txt
```

## 获取解析域名

```bash
# https://github.com/d3mondev/puredns
puredns resolve subs.txt -r dict/resolvers.txt -w resolved.txt
```

## 获取 DNS 记录

```bash
# 获取解析子域名 DNS 记录
cat resolved.txt | dnsx -resp -retry 3 -json -o scan/dnsx-resolved.json

# 获取所有子域名 DNS 记录 (用于 Host 碰撞测试)
cat subs.txt | dnsx -resp -retry 3 -json -o scan/dns-subs.json
# 筛选所有子域名 A 记录的 IP
cat scan/dns-subs.json | jq -r 'select(.a != null) | .a[]' > subs-dns-a.txt
# 获取所有子域名中解析结果为 NXDOMAIN 的域名
cat subs.txt | dnsx -json -retry 3 -rc NXDOMAIN -silent | jq -r '.host' > subs-dns-nxdomain.txt

cp subs-dns-* ../penetration-testing/host-collision/


# 筛选 CDN 获取 IP 地址 (不稳定)
# https://github.com/projectdiscovery/cdncheck
grep -v -f <(cat scan/dnsx-resolved.json | jq -r '.a[]' | cdncheck -silent) \
    <(cat scan/dnsx-resolved.json | jq -r '.a[]') | sort -u > ips.txt
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

*

## Burp 流量收集子域名

*

**目录及文件说明**

```bash
➜ tree --dirsfirst
.
├── dict # 扫描字典
│   ├── resolvers.txt
│   └── subdomains.txt
├── scan # 输出结果
│   ├── dnsx-resolved.json
│   ├── dnsx-subs.json
│   ├── puredns-bruteforce.json
│   ├── shuffledns.json
│   ├── subfinder-proxy.json
│   └── subfinder.json
├── resolved.txt # 存在解析记录的域名
├── roots.txt # 目标根域名
└── subs.txt # 所有主动和被动方式发现的子域名
```
