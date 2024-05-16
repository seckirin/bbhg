# Domain-based Recon

## Domain Basics Information

> This method can be used for asset ownership verification

```bash
# Domain Analysis
# Query company name through domain
https://beian.miit.gov.cn/

# Query registrant information through domain
https://www.whois.com/whois/
https://who.is/
https://www.ip138.com/

# Query ICP license information through domain
https://icp.chinaz.com/bytedance.com
https://site.ip138.com/bytedance.com/beian.htm
https://www.beianx.cn/search/bytedance.com

# ICP License Analysis
# Query company name and registration time through ICP license
https://icp.chinaz.com/粤B2-20090059-1
https://www.tianyancha.com/search?key=黑ICP备10000001号
https://www.beianx.cn/search/黑ICP备10000001号

# IP Analysis
# Query registrant information through ip
https://www.whois.com/whois/
https://who.is/
https://www.ip138.com/
```

## Subdomain Enum

See the [Subdomain Enum](subdomain-enumeration.md) page.

## Get DNS Record

```bash
cat subdomains.txt | dnsx -recon -silent -json -o dns.json &>/dev/null
```

## Extract IP Address

```bash
# Exclude CDN and internal IP
jq -r '. | select(.has_internal_ips|not) | try .a[]' dns.json | sort -u | 
    cdncheck -silent -exclude -o ips.txt &>/dev/null
```

## Port Scanning

```bash
# Exclude virtual hosts and scan nmap Top 3000 port
nmap -iL ips.txt -vv -T4 --top-ports 3000 -n --open -oX nmap.xml
```

## Website Probing

```bash
tew -x nmap.xml -dnsx dnsx.json --vhost | httpx -json -o websites.json

jq -r '.url' websites.json | sed -e 's/:80$//g' -e 's/:443$//g' | anew -q websites.txt

jq -r 'try. | "\(.url) [\(.status_code)] [\(.title)] [\(.webserver)] \(.tech)"' \
    websites.json | anew -q websites_plain.txt
```

### Screenshot

```bash
cat websites.txt | cut -d ' ' -f 1 |
    nuclei -headless -id screenshot -V -dir='screenshots' -silent
```

## Service Identification

```bash
nuclei -l websites.txt \
    -t ~/nuclei-templates/http/technologies/fingerprinthub-web-fingerprints.yaml \
    -silent -jsonl
```
