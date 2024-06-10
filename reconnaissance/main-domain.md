# Main Domain

<details>

<summary>Question &#x26; Answer</summary>

1. **为什么是主域 (Main Domain) 而不是根域 (Root Domain)？**根域 (Root Domain) 又称顶级域 (Top-Level Domain)，例如 .com，.net，.org 等，为了避免混淆，这里对主域 (Main Domain) 的定义是**业务运行的主域名**，而不是顶级域。
2. **我应该选择哪种技术？**在初期阶段，使用**基于公司**的收集技术，通过公司名称和 SRC 最新通告确定业务主域名；在收集了一定的域名后，使用基于域名的收集技术，收集更多的域名资产，在完成了一次基本的信息收集流程之后（域名收集、子域名收集，以及站点探测），如果有需求，再使用**基于网站**的收集技术，以此来拓充资产数量。

</details>

## Gathering

### Based on Company

```bash
# Company Name >> ICP Licensed domains
https://www.beianx.cn/search/
https://0.zone/
https://www.qcc.com/
https://aiqicha.baidu.com/
https://shuidi.cn/
https://www.tianyancha.com/
https://icp.chinaz.com/
https://beian.miit.gov.cn/

# https://github.com/wgpsec/ENScan_GO
./enscan-<version> -n <company_name> -type all -field icp
./enscan-<version> -f <company.txt> -type all -field icp

# 最新 ICP 数据
https://shangjibao.baidu.com

# 历史 ICP 记录
https://icp.chinaz.com/record
```

```bash
# https://github.com/y00k1sec/hacking-gadgets/blob/main/icpquery.py
echo <company> | icpquery | jq -r 'try .params.list[].domain'
cat companies.txt | icpquery | jq -r 'try .params.list[].domain' | anew domains.txt
```

### Based on Domain (Through Internal NameServer)

```bash
# 创建结束会话后自动删除的临时文件，用于存储发现的名称服务器及关联主域名
tmp_nameservers=$(mktemp); trap "rm -rf $tmp_nameservers" EXIT
tmp_related_roots=$(mktemp); trap "rm -rf $tmp_related_roots" EXIT

# 查询现有主域名所使用的名称服务器
cat domains.txt | dnsx -ns -resp -silent -json | jq -r '.ns[]' | sort -u | tee $tmp_nameservers
vim $tmp_nameservers # 剔除无趣的关联主域名

# 查询名称服务器相关联的其他主域名
while IFS= read -r nameserver; do
    curl -s "https://api.hackertarget.com/findshareddns/?q=$nameserver" | unfurl -u domains | anew $tmp_related_roots
done < $tmp_nameservers

vim $tmp_related_roots # 剔除无趣的关联主域名
```

### Based on Website (Scraping)

```bash
# SSL/TLS Certificate

# 创建结束会话后自动删除的临时文件，用于存储发现的关联主域名
tmp_related_roots=$(mktemp); trap "rm -rf $tmp_related_roots" EXIT

cat websites.txt | httpx -tls-probe -tls-grab -silent -json | jq -r 'try .tls.subject_cn, .tls.subject_an[]' | unfurl -u apexes | anew $tmp_related_roots
vim $tmp_related_roots # 剔除无趣的关联主域名
```

```bash
# CSP (Content Security Policies)

# 创建结束会话后自动删除的临时文件，用于存储发现的关联主域名
tmp_related_roots=$(mktemp); trap "rm -rf $tmp_related_roots" EXIT

cat websites.txt | httpx -csp-probe -silent -json | jq -r 'try .csp.domains[]' | unfurl -u apexes | anew $tmp_related_roots
vim $tmp_related_roots # 剔除无趣的关联主域名
```

```bash
# Favicon Hash

# 创建结束会话后自动删除的临时文件，用于存储发现的关联主域名
tmp_related_roots=$(mktemp); trap "rm -rf $tmp_related_roots" EXIT

cat websites.txt | httpx -favicon -silent -json | jq -r 'select(.favicon!=null) | "icon_hash=\"" + .favicon + "\""' | fofax -silent -fs 10000 -ff domain | anew $tmp_related_roots
vim $tmp_related_roots # 剔除无趣的关联主域名

# https://github.com/yuukisec/ifavicon
ifavicon -url https://<domain.com>/favicon.ico
ifavicon -file /path/to/favicon.ico
```

```bash
# Set-Cookie Headers

# 创建结束会话后自动删除的临时文件，用于存储发现的关联主域名
tmp_related_roots=$(mktemp); trap "rm -rf $tmp_related_roots" EXIT

cat websites.txt | httpx -include-response-header -json -silent | jq -r '.header.set_cookie' | tr ";, " "\n" | grep domain= | cut -d'=' -f2 | unfurl -u apexes | anew $tmp_related_roots
vim $tmp_related_roots # 剔除无趣的关联主域名
```

```sh
# Google Analytics ID

# Regex: UA-[0-9]+(-[0-9]+)

# https://gist.github.com/y00k1sec/f6b4659b5d4c2d825a6d41a8ae3d73b7
cat websites.txt | nuclei -t $TOOLS/nuclei-templates/google-analytics-id-detection.yaml -silent -no-color | cut -d '"' -f2 > google_analytics_id.txt

# https://github.com/dhn/udon
while read id
do
    udon -s $id -json -silent | jq -r '.domain'
done < google_analytics_id.txt
```

```bash
# Location Headers (In the future)
```

## Analysis

```bash
# 提取 santa.txt 文件中的主域名到 domains.txt
cat santa.txt | unfurl -u apexes | tee domains.txt

# 批量查询 domains.txt 文件中主域名的备案信息
# https://github.com/y00k1sec/hacking-gadgets/blob/main/domainrecon.sh
# while IFS= read -r domain; do
#    domainrecon.sh domain "$domain" | jq -r '.params.list[]'
# done < domains.txt
cat domains.txt | icpquery | jq -r '.params.list[] | "[\(.updateRecordTime)] [\(.domain)] [\(.unitName)] [\(.serviceLicence)]"'

# 手动在网页上查询域名的备案、公司及注册时间等信息
# https://github.com/y00k1sec/hacking-gadgets/blob/main/bizrecon.sh
bizrecon.sh domain <domain.txt>
```
