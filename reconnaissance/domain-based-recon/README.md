# Domain-based Recon

## Domain Basics Information

This technique also applies to asset ownership verification.

```markdown
# 1. Domain Analysis

## 1.1 Query company name through domain
https://beian.miit.gov.cn/

## 1.2 Query registrant information through domain
https://www.whois.com/whois/
https://who.is/
https://www.ip138.com/

## 1.3 Query ICP license information through domain
https://icp.chinaz.com/bytedance.com
https://site.ip138.com/bytedance.com/beian.htm
https://www.beianx.cn/search/bytedance.com

## 1.4 Query historical IP resolution records through domain
https://ipchaxun.com/

# 2. ICP License Analysis

## 2.1 Query company name and registration time through ICP license
https://icp.chinaz.com/粤B2-20090059-1
https://www.tianyancha.com/search?key=黑ICP备10000001号
https://www.beianx.cn/search/黑ICP备10000001号

# 3. IP Analysis

# 3.1 Query registrant information through ip
https://www.whois.com/whois/
https://who.is/
https://www.ip138.com/
```

## Subdomain Enum

For more information, please refer to page [Subdomain Enum](subdomain-enumeration.md).

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
tew -x nmap.xml -dnsx dns.json --vhost -o hostport.txt |
    httpx -random-agent -silent -json -o websites.json &>/dev/null

jq -r 'try . | "\(.url) [\(.status_code)] [\(.title)] [\(.content_length) byte]"' websites.json |
    sed -e 's/:80$//g' -e 's/:443$//g' | anew -q websites.txt
```

### Screenshot

```bash
cat websites.txt | cut -d ' ' -f 1 |
    nuclei -headless -id screenshot -V -dir='screenshots' -silent
```

## Service Identification

```bash
cat websites.txt | cut -d ' ' -f 1 | 
    nuclei -id fingerprinthub-web-fingerprints -silent -jsonl
```
