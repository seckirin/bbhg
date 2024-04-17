---
description: 💡 该页面主要提供了子域名枚举建议和注意事项。
---

# 子域名枚举建议

如果对操作中的目录或文件存在疑问，请查询页面底部的 "目录及文件说明"。

## 被动收集子域名

```bash
# https://github.com/projectdiscovery/subfinder
# https://docs.projectdiscovery.io/tools/subfinder/install#post-install-configuration
# BeVigil, BinaryEdge, BufferOver, C99, Censys, CertSpotter, Chaos, Chinaz, DNSDB, Fofa, FullHunt, GitHub, Intelx, PassiveTotal, quake, Robtex, SecurityTrails, Shodan, ThreatBook, VirusTotal, WhoisXML API, ZoomEye API china - worldwide, dnsrepo, Hunter, Facebook, BuiltWith
subfinder -d target.com -all -v -oJ -cs -o subfinder.json
# or
subfinder -dL roots.txt -all -v -oJ -cs -o subfinder.json
cat subfinder.json | jq '.host'

# 需要代理的信息源 (也可以在上面两条命令中直接加上 -proxy 参数)
# github,censys,commoncrawl,bufferover,hackertarget,waybackarchive,facebook,anubis,digitorus
subfinder -dL roots.txt -s github,censys,commoncrawl,bufferover,\
hackertarget,waybackarchive,facebook,anubis,digitorus \
-proxy socks5://ip:port -v -oJ -cs -o subfinder-proxy.json
```

## 主动爆破子域名

```bash
# 获取 DNS 解析器
# https://github.com/trickest/resolvers
wget https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt -O dict/resolvers.txt

# 获取子域名字典
# https://github.com/yuukisec/hackdict
wget https://raw.githubusercontent.com/Yuukisec/HackDict/main/subdomains/subdomains.txt -O dict/subdomains.txt

# 爆破子域名
# https://github.com/projectdiscovery/shuffledns
cat roots.txt | shuffledns -w dict/subdomains.txt -r dict/resolvers.txt -j -o scan/shuffledns/shuffledns.json
cat scan/shuffledns/shuffledns.json | jq '.hostname' -r
# or
# https://github.com/d3mondev/puredns
puredns bruteforce dict/subdomains.txt target.com -r dict/resolvers.txt -w scan/puredns/puredns-bruteforce.txt
# or
puredns bruteforce dict/subdomains.txt -d roots.txt -r dict/resolvers.txt -w scan/puredns/puredns-bruteforce.txt
```

### 获取解析域名

```bash
# https://github.com/d3mondev/puredns
puredns resolve subs.txt -r dict/resolvers.txt -w resolved.txt
```

### 获取 DNS 记录

```bash
# https://github.com/projectdiscovery/dnsx
cat subs.txt | dnsx -resp -retry 3 -json -o scan/dnsx/subs-dnsx.json
cat scan/dnsx/subs-dnsx.json | jq '.' -r
# or
cat resolved.txt | dnsx -resp -retry 3 -json -o scan/dnsx/resolved-dnsx.json
cat scan/dnsx/resolved-dnsx.json | jq '.' -r
```

## 爬取站点收集子域名

*

## Burp 流量收集子域名

*

## 反编译 APK 收集子域名

*

## 反编译小程序收集子域名

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
