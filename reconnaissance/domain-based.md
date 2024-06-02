# Subdomain Handling

## DNS Record Collection

```bash
cat subdomains.txt | dnsx -recon -silent -json -o dns.json &>/dev/null
```

## IP Address Extraction

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

## Services Identification

```bash
cat websites.txt | cut -d ' ' -f 1 | 
    nuclei -id fingerprinthub-web-fingerprints -silent -jsonl
```

## Fingerprint Website
