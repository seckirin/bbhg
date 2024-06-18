# Main domains

## Gathering

### 1) Based company name

#### ICP License

```bash
# https://github.com/y00k1sec/hacking-gadgets/blob/main/icpquery.py
cat companies.txt | icpquery | tee domains_icp.json
cat domains_icp.json | jq -s 'map(.params.list[]) | group_by(.unitName) | map({unitName: .[0].unitName, domain: map(.domain)})'


# Company name to ICP licensed domains websites
https://www.beianx.cn/search/
https://0.zone/
https://www.qcc.com/
https://aiqicha.baidu.com/
https://shuidi.cn/
https://www.tianyancha.com/
https://icp.chinaz.com/
https://beian.miit.gov.cn/

# Latest ICP
https://shangjibao.baidu.com

# ICP history
https://icp.chinaz.com/record

# https://github.com/wgpsec/ENScan_GO
./enscan-<version> -n <company_name> -type all -field icp
./enscan-<version> -f <company.txt> -type all -field icp
```

### 2) Based on existing domain

#### Internal NameServer

```bash
# 创建结束会话后自动删除的临时文件，用于存储发现的名称服务器及关联主域名
tmp_nameservers=$(mktemp); trap "rm -rf $tmp_nameservers" EXIT
tmp_related_domains=$(mktemp); trap "rm -rf $tmp_related_domains" EXIT

# 查询现有主域名所使用的名称服务器
cat domains.txt | dnsx -ns -resp -silent -json | jq -r '.ns[]' | anew $tmp_nameservers
vim $tmp_nameservers # 剔除无趣的关联主域名

# 查询名称服务器相关联的其他主域名
# proyx on
while IFS= read -r nameserver; do
    curl -s "https://api.hackertarget.com/findshareddns/?q=$nameserver" | unfurl -u domains | anew $tmp_related_domains
done < $tmp_nameservers

vim $tmp_related_domains # 剔除无趣的关联主域名
```

### 3) Based on existing website

#### SSL/TLS Certificate

```bash
# 创建结束会话后自动删除的临时文件，用于存储发现的关联主域名
tmp_related_domains=$(mktemp); trap "rm -rf $tmp_related_domains" EXIT

cat websites.txt | httpx -tls-probe -tls-grab -silent -json | jq -r 'try .tls.subject_cn, .tls.subject_an[]' | unfurl -u apexes | anew $tmp_related_domains
vim $tmp_related_domains # 剔除无趣的关联主域名
```

#### CSP (Content Security Policies)

```bash
# 创建结束会话后自动删除的临时文件，用于存储发现的关联主域名
tmp_related_domains=$(mktemp); trap "rm -rf $tmp_related_domains" EXIT

cat websites.txt | httpx -csp-probe -silent -json | jq -r 'try .csp.domains[]' | unfurl -u apexes | anew $tmp_related_domains
vim $tmp_related_domains # 剔除无趣的关联主域名
```

#### Favicon Hash

```bash
# 创建结束会话后自动删除的临时文件，用于存储发现的关联主域名
tmp_related_domains=$(mktemp); trap "rm -rf $tmp_related_domains" EXIT

cat websites.txt | httpx -favicon -silent -json | jq -r 'select(.favicon!=null) | "icon_hash=\"" + .favicon + "\""' | fofax -silent -fs 10000 -ff domain | anew $tmp_related_domains
vim $tmp_related_domains # 剔除无趣的关联主域名

# https://github.com/yuukisec/ifavicon
ifavicon -url https://<domain.com>/favicon.ico
ifavicon -file /path/to/favicon.ico
```

#### Set-Cookie Headers

```bash
# 创建结束会话后自动删除的临时文件，用于存储发现的关联主域名
tmp_related_domains=$(mktemp); trap "rm -rf $tmp_related_domains" EXIT

cat websites.txt | httpx -include-response-header -json -silent | jq -r '.header.set_cookie' | tr ";, " "\n" | grep domain= | cut -d'=' -f2 | unfurl -u apexes | anew $tmp_related_domains
vim $tmp_related_domains # 剔除无趣的关联主域名
```

#### Google Analytics ID

```sh
# Regex: UA-[0-9]+(-[0-9]+)

# https://gist.github.com/y00k1sec/f6b4659b5d4c2d825a6d41a8ae3d73b7
cat websites.txt | nuclei -t $TOOLS/nuclei-templates/google-analytics-id-detection.yaml -silent -no-color | cut -d '"' -f2 > google_analytics_id.txt

# https://github.com/dhn/udon
while read id
do
    udon -s $id -json -silent | jq -r '.domain'
done < google_analytics_id.txt
```

#### Location Headers

```bash
# In the future
```

## Analysis

```bash
# 提取 santa.txt 文件中的主域名到 tmp_domains.txt
cat santa.txt | unfurl -u apexes | tee tmp_domains.txt

# 批量查询 tmp_domains.txt 文件中主域名的备案信息
# https://github.com/y00k1sec/hacking-gadgets/blob/main/domainrecon.sh
# while IFS= read -r domain; do
#    domainrecon.sh domain "$domain" | jq -r '.params.list[]'
# done < tmp_domains.txt
cat tmp_domains.txt | icpquery | jq -c '[.params.list[]] | sort_by(.updateRecordTime) | sort_by(.unitName) | .[] | [.domain, .natureName, .unitName, .mainLicence, .serviceLicence, .updateRecordTime]'

# 手动在网页上查询域名的备案、公司及注册时间等信息
# https://github.com/y00k1sec/hacking-gadgets/blob/main/bizrecon.sh
bizrecon.sh domain <domain.txt>
```
