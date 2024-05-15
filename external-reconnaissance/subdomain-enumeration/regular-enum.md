# Regular Enum

## Passive Enum

```bash
# https://github.com/owasp-amass/amass/tree/v3.23.3
amass enum -passive -d $DOMAIN -silent -timeout 15 -o amass.txt

# https://github.com/blacklanternsecurity/bbot
bbot -t $DOMAIN -f subdomain-enum -rf passive -em massdns -y -s -om json |
    jq -r 'select(.scope_distance==0) | select(.type=="DNS_NAME") | .data' >bbot.txt

# https://github.com/cemulus/crt
crt -s -l 999999 -json $DOMAIN |
    jq -r '.[].subdomain' | sed -e 's/^\*\.//' > crt.txt

crt -s -l 999999 -json -o crt.json $DOMAIN >/dev/null
[[ -s crt.json ]] && jq -r '.[].subdomain' crt.json | sed -e 's/^\*\.//' >crt.txt && rm crt.json

# https://github.com/gwen001/github-subdomains
github-subdomains -d $DOMAIN -t $GITHUB_TOKENS \
    -k -q -o github-subdomains.txt &>/dev/null

# https://github.com/gwen001/gitlab-subdomains
gitlab-subdomains -d $DOMAIN -t "$(cat $GITLAB_TOKENS)" &>/dev/null && mv $DOMAIN.txt gitlab-subdomains.txt

# https://github.com/projectdiscovery/subfinder
subfinder -d $DOMAIN -all -o subfinder.txt -es github \
    "$([[ -n "$HTTP_PROXY" ]] && echo " -proxy $HTTP_PROXY")" &>/dev/null
```

```bash
# Summarize passive enum results
results=(
    "amass.txt"
    "bbot.txt"
    "crt.txt"
    "github-subdomains.txt"
    "gitlab-subdomains.txt"
    "subfinder.txt"
)

for result in "${results[@]}"; do
    # If the file exists
    if [[ -e "$result" ]]; then
        # Extract results in scope | Unique line to passive.txt
        grep -E "^$DOMAIN$|\.$DOMAIN$" "$result" | anew -q passive.txt
        # Delete the source file
        # rm $result
    fi
done
```

```bash
# python3 summarizer.py --source passive --resolved 0
```

## Active Resolve

```bash
puredns resolve passive.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write active.txt \
    &>/dev/null

# Firmly Resolve
puredns resolve active.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write active.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --source active --resolved 1
# isubsum --source active --resolved 1
```

## Brute Force

```bash
puredns bruteforce $SUBDOMAINS $DOMAIN \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write brute.txt \
    &>/dev/null
sort -u brute.txt -o brute.txt

# Firmly Resolve
puredns resolve brute.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write brute.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --source brute --resolved 1
# isubsum --source brute --resolved 1
```

## Alter Combo

* Best when the number of subdomains is 500 to 1000
* **Features of different tools**
  * [altdns](https://github.com/infosec-au/altdns): large, comprehensive
  * [alterx](https://github.com/projectdiscovery/alterx): stdin, customizable
  * [gotator](https://github.com/Josue87/gotator): depth, customizable
  * [ripgen](https://github.com/resyncgg/ripgen): high-performance
* **Hint:** Choose one of the four. (I usually use alterx)

<details>

<summary>altdns</summary>

```bash
jq -r '.[]|select(.resolved==1)|.subdomain' subdomains.json >subdomains.txt
altdns -i subdomains.txt -w $PERMUTATIONS -o altdns.txt
puredns resolve altdns.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> alter.txt &&
    rm altdns.txt

# Round 2
altdns -i alter.txt -w $PERMUTATIONS -o altdns.txt
puredns resolve altdns.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> alter.txt &&
    rm altdns.txt
 
# Firmly Resolve
sort -u alter.txt -o alter.txt
puredns resolve alter.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write alter.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --source alter --resolved 1
# isubsum --source alter --resolved 1
```

</details>

<details>

<summary>alterx</summary>

```bash
jq -r '.[]|select(.resolved==1)|.subdomain' subdomains.json >subdomains.txt
alterx -l subdomains.txt -enrich -silent | anew -q alterx.txt
puredns resolve alterx.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> alter.txt &&
    rm alterx.txt

# Round 2
alterx -l alter.txt -enrich -silent | anew -q alterx.txt
puredns resolve alterx.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> alter.txt &&
    rm alterx.txt

# Firmly Resolve
sort -u alter.txt -o alter.txt
puredns resolve alter.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write alter.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --source alter --resolved 1
# isubsum --source alter --resolved 1
```

</details>

<details>

<summary>gotator</summary>

```bash
jq -r '.[]|select(.resolved==1)|.subdomain' subdomains.json >subdomains.txt
gotator -sub subdomains.txt -perm $PERMUTATIONS -depth 1 -numbers 3 -mindup -adv -md -silent | head -c 1G | anew -q gotator.txt
puredns resolve gotator.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> alter.txt &&
    rm gotator.txt

# Round 2
gotator -sub alter.txt -perm $PERMUTATIONS -depth 1 -numbers 3 -mindup -adv -md -silent | head -c 1G | anew -q gotator.txt
puredns resolve gotator.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> alter.txt &&
    rm gotator.txt

# Firmly Resolve
sort -u alter.txt -o alter.txt
puredns resolve alter.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write alter.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --source alter --resolved 1
# isubsum --source alter --resolved 1
```

</details>

<details>

<summary>ripgen</summary>

```bash
jq -r '.[]|select(.resolved==1)|.subdomain' subdomains.json >subdomains.txt
ripgen -d subdomains.txt -w $PERMUTATIONS | head -c 1G | anew -q ripgen.txt
puredns resolve ripgen.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> alter.txt &&
    rm ripgen.txt

# Round 2
ripgen -d alter.txt -w $PERMUTATIONS | head -c 1G | anew -q ripgen.txt
puredns resolve ripgen.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> alter.txt &&
    rm ripgen.txt

# Firmly Resolve
sort -u alter.txt -o alter.txt
puredns resolve alter.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write alter.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --source alter --resolved 1
# isubsum --source alter --resolved 1
```

</details>

<details>

<summary>Comparison Results (jq)</summary>

```bash
# Only one tool
# cat subdomains.json | jq '.[] | select(.sources == ["alter_altdns"])'
# cat subdomains.json | jq '.[] | select(.sources == ["alter_alterx"])'
# cat subdomains.json | jq '.[] | select(.sources == ["alter_gotator"])'
# cat subdomains.json | jq '.[] | select(.sources == ["alter_ripgen"])'

# Count the number of a alter combo
cat subdomains.json | jq '.[] | select(.sources[] == "alter") | .subdomain' | wc -l

# Only Alter results
cat subdomains.json | jq -r '.[] | select(.sources | all(. | contains("alter")))'
```

</details>

## AI Generate

```bash
# https://github.com/cramppet/regulator
cat subdomains.json | jq -r '.[] | select(.resolved==1) | .subdomain' > subdomains.txt
scan_pwd=$(pwd) && pushd $TOOLS/regulator && python3 main.py -t $DOMAIN -f $scan_pwd/subdomains.txt -o $scan_pwd/regulator.txt && popd

puredns resolve regulator.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write intelli.txt \
    &>/dev/null &&
    rm regulator.txt

# Firmly Resolve
puredns resolve intelli.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write intelli.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --sources intelli --resolved 1
# isubsum --source intelli --resolved 1
```

## Web Scraping

```bash
cat subdomains.json | jq -r '.[] | select(.resolved==1) | .subdomain' > subdomains.txt

# Scrap of URLs
httpx -l subdomains.txt -status-code -title -web-server -tech-detect -location -follow-host-redirects \
    -threads 50 -rate-limit 150 -timeout 10 -retries 2 \
    -silent -no-color -json -o web_full_info1.json \
    &>/dev/null

jq -r 'try .url' web_full_info1.json 2>/dev/null |
    grep $DOMAIN |
    grep -E '^((http|https):\/\/)?([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{1,}(\/.*)?$' |
    sed "s/*.//" |
    anew scrap_urls.txt | unfurl -u domains 2>/dev/null | anew -q scrap.txt &&
    rm web_full_info1.json

# Scrap of TLS and CSP
httpx -l scrap_urls.txt \
    -tls-grab -tls-probe -csp-probe -status-code \
    -threads 50 --rate-limit 150 -timeout 10 -retries 2 \
    -silent -no-color -json -o web_full_info2.json \
    &>/dev/null

jq -r 'try ."tls-grab"."dns_names"[],try .csp.domains[],try .url' web_full_info2.json 2>/dev/null |
    grep $DOMAIN |
    grep -E '^((http|https):\/\/)?([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{1,}(\/.*)?$' |
    sed "s/*.//" |
    sort -u |
    httpx -silent |
    anew scrap_urls.txt | unfurl -u domains 2>/dev/null | anew -q scrap.txt &&
    rm web_full_info2.json

# Scrap of Web Crawling
katana -list scrap_urls.txt \
    -js-crawl -known-files all \
    -concurrency 20 -depth 2 -field-scope rdn \
    -silent -o katana.txt \
    &>/dev/null &&
    # rm scrap_urls.txt

sed -i '/^.\{2048\}./d' katana.txt
cat katana.txt | unfurl -u domains 2>/dev/null | grep -E "\.$DOMAIN$" |
    grep -E '^((http|https):\/\/)?([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{1,}(\/.*)?$' |
    anew -q scrap.txt &&
    rm katana.txt

# Firmly Resolve
puredns resolve scrap.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write scrap.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --sources scrap --resolved 1
# isubsum --sources scrap --resolved 1
```

## Google Analytics

* **Reverse Google Analytics ID Search WebSites:**
  * [https://intelx.io/tools?tab=analytics](https://intelx.io/tools?tab=analytics)
    * [https://search.dnslytics.com/search?q=html.tag:ua-33427076\&d=domains](https://search.dnslytics.com/search?q=html.tag:ua-33427076\&d=domains)
    * [https://hackertarget.com/reverse-analytics-search/](https://hackertarget.com/reverse-analytics-search/)
    * [https://publicwww.com/websites/%22ua-33427076-%22/](https://publicwww.com/websites/%22ua-33427076-%22/)
    * [https://analyzeid.com/id/UA-33427076](https://analyzeid.com/id/UA-33427076)
  * [https://osint.sh/analytics](https://osint.sh/analytics)
  * [https://builtwith.com/relationships/tag/UA-33427076](https://builtwith.com/relationships/tag/UA-33427076)
  * [http://site-overview.com/website-report-search/analytics-account-id/33427076](http://site-overview.com/website-report-search/analytics-account-id/33427076)
  * [https://spyonweb.com/UA-33427076](https://spyonweb.com/UA-33427076)
* **Domain Gathering:** This method is also applicable to [domain gathering](../based-on-company/domain-gathering.md#google-analytics).

```bash
# https://github.com/projectdiscovery/nuclei
# https://gist.github.com/yuukisec/f6b4659b5d4c2d825a6d41a8ae3d73b7
nuclei -t $TOOLS/nuclei-templates/google-analytics-id-detection.yaml -l scrap_urls.txt -silent -no-color |
    cut -d '"' -f2 > google_analytics_id.txt

# https://github.com/dhn/udon
while read id
do
    udon -s $id -silent |
        grep -E "^$DOMAIN$|\.$DOMAIN$" | anew analy.txt
done < google_analytics_id.txt && rm google_analytics_id.txt

# Firmly Resolve
puredns resolve analy.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write analy.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --sources analy --resolved 1
# isubsum --sources analy --resolved 1
```
