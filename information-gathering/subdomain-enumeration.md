---
description: 💡 该页面主要提供了子域名枚举建议和注意事项。
---

# 子域名枚举

如果对操作中的目录或文件存在疑问，请查询页面底部的 "目录及文件说明"。

收集子域名的两种主要方式包括主动爆破子域名和被动收集子域名。被动收集为主要的收集方式，其他方式作为可选项，辅助子域名收集的全面性。

```bash
# Initialize environment
ROOT_PATH=;TARGET_NAME=;DATETIME=$(date +%Y%m%d%H%M%S)
mkdir -p $ROOT_PATH/$TARGET_NAME \
    $ROOT_PATH/$TARGET_NAME-$DATETIME/dict \
    $ROOT_PATH/$TARGET_NAME-$DATETIME/scan \
#     $ROOT_PATH/$TARGET_NAME-$DATETIME/scan/subfinder \
#     $ROOT_PATH/$TARGET_NAME-$DATETIME/scan/shuffledns \
#     $ROOT_PATH/$TARGET_NAME-$DATETIME/scan/puredns/brute-force \
#     $ROOT_PATH/$TARGET_NAME-$DATETIME/scan/puredns/resolve \
#     $ROOT_PATH/$TARGET_NAME-$DATETIME/scan/dnsx
cd $ROOT_PATH/$TARGET_NAME/$DATETIME

# Fill root domains
touch roots.txt
echo ROOTS... | tee roots.txt

# DNS resolver - https://github.com/trickest/resolvers
wget https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt -O dict/resolvers.txt

# Subdomain dict - https://github.com/yuukisec/hackdict
wget https://raw.githubusercontent.com/Yuukisec/HackDict/main/subdomains/subdomains.txt -O dict/subdomains.txt

# ❯ tree
# .
# ├── dict
# │   ├── resolvers.txt
# │   └── subdomains.txt
# ├── roots.txt
# └── scan
#     ├── dnsx
#     ├── puredns
#     │   ├── brute-force
#     │   └── resolve
#     ├── shuffledns
#     └── subfinder
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
cat scan/subfinder.json | jq '.host' -r | anew subs.txt
```

## 主动爆破子域名

```bash
# 爆破子域名
# https://github.com/blechschmidt/massdns
# https://github.com/projectdiscovery/shuffledns
cat roots.txt | shuffledns -w dict/subdomains.txt -r dict/resolvers.txt -j -o scan/shuffledns.json
cat scan/shuffledns.json | jq '.hostname' -r | anew subs.txt
# or
# https://github.com/robertdavidgraham/masscan
# https://github.com/d3mondev/puredns
# Single domain
puredns bruteforce dict/subdomains.txt target.com -r dict/resolvers.txt -w scan/puredns-bruteforce.txt
# Multiple domain
puredns bruteforce dict/subdomains.txt -d roots.txt -r dict/resolvers.txt -w scan/puredns-bruteforce.txt
cat scan/puredns-bruteforce.txt | anew subs.txt
```

### 获取解析域名

```bash
# https://github.com/d3mondev/puredns
puredns resolve subs.txt -r dict/resolvers.txt -w resolved.txt
```

## 获取 DNS 记录

```bash
# https://github.com/projectdiscovery/dnsx
cat resolved.txt | dnsx -resp -retry 3 -json -o scan/dnsx-resolved.json
cat scan/dnsx-resolved.json | jq '.a[]' -r | sort -u > ips-resolved.txt
# or
cat subs.txt | dnsx -resp -retry 3 -json -o scan/dnsx-subs.json
cat scan/dnsx-subs.json | jq '.a[]' -r | sort -u > ips-subs.txt

# 筛选 CDN 获取 IP 地址
# https://github.com/projectdiscovery/cdncheck
grep -v -f <(cat scan/dnsx-resolved.json | jq '.a[]' -r | cdncheck -silent) \
    <(cat scan/dnsx-resolved.json | jq '.a[]' -r) | sort -u > ips.txt
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
│   ├── dnsx
│   │   ├── resolved-dnsx.json
│   │   └── subs-dnsx.json
│   ├── puredns
│   │   └── puredns-bruteforce.json
│   ├── shuffledns
│   │   └── shuffledns.json
│   └── subfinder
│       ├── subfinder-proxy.json
│       └── subfinder.json
├── resolved.txt # 存在解析记录的域名
├── roots.txt # 目标根域名
└── subs.txt # 所有主动和被动方式发现的子域名
```
