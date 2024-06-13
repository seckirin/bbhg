# Subdomain

注意：

1. 必须声明[通用函数](subdomain.md#universal-function)中的函数，否则则需要在命令中替换为函数的实际内容
2. 执行命令前先检查工具是否已经安装，否则将无法正常工作

## Preparations

### Manually

```bash
# https://github.com/y00k1sec/hacking-gadgets/blob/main/subrecon.env
source "${HOME}/hacking/gadgets/subrecon.env"
domain=<domain>

# Resolver
wget -q --show-progress -O - "$RESOLVERS_URL" >"$RESOLVERS"
wget -q --show-progress -O - "$RESOLVERS_TRUSTED_URL" >"$RESOLVERS_TRUSTED"

# Wordlist
wget -q --show-progress -O - "$SUBDOMAINS_TINY_URL" >"$SUBDOMAINS_TINY"
wget -q --show-progress -O - "$SUBDOMAINS_MEDIUM_URL" >"$SUBDOMAINS_MEDIUM"
wget -q --show-progress -O - "$SUBDOMAINS_HUGE_URL" >"$SUBDOMAINS_HUGE"
wget -q --show-progress -O - "$SUBDOMAINS_FULL_URL1" "$SUBDOMAINS_FULL_URL2" | sort -u >"$SUBDOMAINS_FULL"
wget -q --show-progress -O - "$PERMUTATIONS_URL" >"$PERMUTATIONS"
```

### subRecon

```bash
# https://github.com/y00k1sec/hacking-gadgets/blob/main/subrecon.sh
subrecon -update-resolvers <quick|reliable>
subrecon -update-wordlists
subrecon -check-tools
```

### Universal Function\*

```bash
run_shuffledns_resolve() {
    shuffledns -d "$domain" -r "$RESOLVERS" -tr "$RESOLVERS_TRUSTED" -mode resolve -silent
}

extract_in_scope_domain() {
    sed '/^.\{2048\}./d' | unfurl -u domains | sed -e 's/^\*\.//' | grep -E ^$domain$\|\.$domain$
}
```

## Enumeration

### 1) Bruteforce

```bash
# Fast: use $SUBDOMAINS_TINY wordlists
# Norm: use $SUBDOMAINS_HUGE wordlists
# Deep: use $SUBDOMAINS_FULL wordlists

# https://github.com/projectdiscovery/shuffledns
shuffledns -d "$domain" -w "$SUBDOMAINS_HUGE" -r "$RESOLVERS" -tr "$RESOLVERS_TRUSTED" -mode bruteforce -silent | anew subdomains_resolved.txt
```

```bash
# Alternative

# https://github.com/d3mondev/puredns
puredns bruteforce "$SUBDOMAINS_HUGE" -d "$domain" -r "$RESOLVERS" --resolvers-trusted "$RESOLVERS_TRUSTED" --quiet | anew -q subdomins_resolved.txt
```

### 2) Passive

```bash
# https://crt.sh
curl -s "https://crt.sh/?q=${domain}&output=json" | jq -r '.[] | .common_name, .name_value' | extract_in_scope_domain | anew subdomains_unresolved.txt | run_shuffledns_resolve | anew subdomains_resolved.txt

# https://github.com/blacklanternsecurity/bbot
bbot -t $domain -f subdomain-enum -rf passive -em massdns -y --config $BBOT_CONFIG --silent -om json | jq -r 'select(.scope_distance==0) | select(.type=="DNS_NAME") | .data' | anew subdomains_unresolved.txt | run_shuffledns_resolve | anew subdomains_resolved.txt

# https://github.com/projectdiscovery/subfinder
subfinder -d $domain -all -es github -provider-config $SUBFINDER_CONFIG -silent -duc | anew subdomains_unresolved.txt | run_shuffledns_resolve | anew subdomains_resolved.txt

# https://github.com/owasp-amass/amass/tree/v3.23.3
amass enum -passive -d $domain -timeout 10 -config $AMASS_CONFIG -silent | anew subdomains_unresolved.txt | run_shuffledns_resolve | anew subdomains_resolved.txt
```

### 3) Altering

```bash
# https://github.com/projectdiscovery/alterx
# https://github.com/projectdiscovery/shuffledns

# Norm: if [[ -s subdomains_resolved.txt ]] && [[ $(wc -l <subdomains_resolved.txt) -le 500 ]]; then
# Deep: No requirement

cat subdomains_resolved.txt | alterx -enrich -silent -duc | run_shuffledns_resolve | anew subdomains_resolved.txt
cat subdomains_resolved.txt | alterx -enrich -silent -duc | run_shuffledns_resolve | anew subdomains_resolved.txt
```

```bash
# Alternative

# https://github.com/infosec-au/altdns
altdns -i subdomains_resolved.txt -w $PERMUTATIONS -o altdns.txt

# https://github.com/Josue87/gotator
gotator -sub subdomains_resolved.txt -perm $PERMUTATIONS -depth 1 -numbers 3 -mindup -adv -md -silent | head -c 1G | anew -q gotator.txt

# https://github.com/resyncgg/ripgen
ripgen -d subdomains_resolved.txt -w $PERMUTATIONS | head -c 1G | anew -q ripgen.txt
```

### 4) AI Regex

```bash
# https://github.com/cramppet/regulator
# https://github.com/projectdiscovery/shuffledns

ai_regex=$(mktemp); trap "rm -rf $ai_regex" EXIT

scan_path=$(pwd)

pushd "${TOOLS}/regulator" 1>/dev/null

python3 main.py -t "$domain" -f "$scan_path/subdomains_resolved.txt" -o "$ai_regex"

popd 1>/dev/null

cat $ai_regex | run_shuffledns_resolve | anew subdomains_resolved.txt
```

### 5) NoError

```bash
run_noerror_enum_fast() {
    if [[ $(echo "absolutely.positively.impossible.$domain" | dnsx -r "$RESOLVERS" -rcode noerror,nxdomain -retry 3 -silent | cut -d ' ' -f 2) == "[NXDOMAIN]" ]]; then
        dnsx -d "$domain" -w "$SUBDOMAINS_TINY" -r "$RESOLVERS" -rcode noerror -silent | cut -d ' ' -f 1 | anew subdomains_noerror.txt
    else
        echo "[-] Wildcard detected, skipping NoError Enum"
    fi
}

run_noerror_enum_norm() {
    if [[ $(echo "absolutely.positively.impossible.$domain" | dnsx -r "$RESOLVERS" -rcode noerror,nxdomain -retry 3 -silent | cut -d ' ' -f 2) == "[NXDOMAIN]" ]]; then
        dnsx -d "$domain" -w "$SUBDOMAINS_MEDIUM" -r "$RESOLVERS" -rcode noerror -silent | cut -d ' ' -f 1 | anew subdomains_noerror.txt
        cat subdomains_unresolved.txt | dnsx -rcode noerror -silent | cut -d ' ' -f 1 | anew subdomains_noerror.txt
    else
        echo "[-] Wildcard detected, skipping NoError Enum"
    fi
}

run_noerror_enum_deep() {
    if [[ $(echo "absolutely.positively.impossible.$domain" | dnsx -r "$RESOLVERS" -rcode noerror,nxdomain -retry 3 -silent | cut -d ' ' -f 2) == "[NXDOMAIN]" ]]; then
        dnsx -d "$domain" -w "$SUBDOMAINS_FULL" -r "$RESOLVERS" -rcode noerror -silent | cut -d ' ' -f 1 | anew subdomains_noerror.txt
        cat subdomains_unresolved.txt | dnsx -rcode noerror -silent | cut -d ' ' -f 1 | anew subdomains_noerror.txt
    else
        echo "[-] Wildcard detected, skipping NoError Enum"
    fi
}

run_noerror_enum_fast
run_noerror_enum_norm
run_noerror_enum_deep
```

### 6) Scraping

```bash
# Website probing simple

# https://github.com/projectdiscovery/httpx

tmp_websites=$(mktemp); trap "rm -rf $tmp_websites" EXIT
cat subdomains_resolved.txt | httpx -silent | anew -q $tmp_websites
```

```bash
# TLS/SSL and CSP

# https://github.com/projectdiscovery/httpx

run_httpx_norm() { httpx -tls-grab -json -silent }
run_httpx_deep() { httpx -tls-grab -tls-probe -csp-probe -tls-grab -json -silent }

cat $tmp_websites | run_httpx_norm | jq -r 'try .tls.subject_cn, try .tls.subject_an[], try .csp.domains[]' | extract_in_scope_domain | anew subdomains_unresolved.txt | run_shuffledns_resolve | anew subdomains_resolved.txt
cat $tmp_websites | run_httpx_deep | jq -r 'try .tls.subject_cn, try .tls.subject_an[], try .csp.domains[]' | extract_in_scope_domain | anew subdomains_unresolved.txt | run_shuffledns_resolve | anew subdomains_resolved.txt
```

```bash
# Website Crawling

# https://github.com/projectdiscovery/katana

run_katana_norm() { katana -js-crawl -depth 1 -silent }
run_katana_deep() { katana -js-crawl -depth 3 -known-files all -silent }

cat "$tmp_websites" | run_katana_norm | extract_in_scope_domain | anew subdomains_unresolved.txt | run_shuffledns_resolve | anew subdomains_resolved.txt
cat "$tmp_websites" | run_katana_deep | extract_in_scope_domain | anew subdomains_unresolved.txt | run_shuffledns_resolve | anew subdomains_resolved.txt
```

```bash
# Google Analytics ID

# https://github.com/projectdiscovery/nuclei-templates
# https://github.com/y00k1sec/iPoCs
# https://github.com/dhn/udon
# https://github.com/Josue87/AnalyticsRelationships

tmp_google_analytics_id=$(mktemp); trap "rm -rf $tmp_google_analytics_id" EXIT
cat "$tmp_websites" | nuclei -t ~/ipocs -id google-analytics-id-detection -silent | cut -d '"' -f 2 | anew -q $tmp_google_analytics_id
while read id
do
    udon -s $id -silent | extract_in_scope_domain | run_shuffledns_resolve | anew subdomains_resolved.txt
done < $tmp_google_analytics_id

while read website
do
    analyticsrelationships -u "$website" -ch | extract_in_scope_domain | run_shuffledns_resolve | anew subdomains_resolved.txt
done < $tmp_websites
```

### 8) DNS Enum

```bash
tmp_dnsrecord=$(mktemp); trap "rm -rf $tmp_dns" EXIT

# https://github.com/projectdiscovery/dnsx
# https://github.com/hakluke/hakip2host

cat subdomains_resolved.txt | dnsx -recon -json -silent > $tmp_dnsrecord

cat $tmp_dnsrecord | jq -r 'try .a[], try .aaaa[], try .cname[], try .ns[], try .ptr[], try .mx[], try .soa[].name, try .soa[].ns, try .soa[].mailbox' | extract_in_scope_domain | run_shuffledns_resolve | anew subdomains_resolved.txt

cat $tmp_dnsrecord | jq -r 'try .a[]' | sort -u | hakip2host | cut -d ' ' -f 3 | extract_in_scope_domain | run_shuffledns_resolve | anew subdomains_resolved.txt
```

## Processing
