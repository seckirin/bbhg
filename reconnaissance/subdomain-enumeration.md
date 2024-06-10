# Subdomain Enum

[Contents](../readme/table-of-contents.md) > [Based on Domain](broken-reference) > [**Subdomain Enumeration**](subdomain-enumeration.md)

## Preparations

<details>

<summary>Environment / Config</summary>

```bash
# Miscellaneous
DOMAIN="vulnweb.com"
TOOLS="$HOME/hack/tools"
HTTP_PROXY="http://127.0.0.1:7890"
ALL_PROXY="socks5://127.0.0.1:7891"

# Resolvers
RESOLVERS=~/hacking/wordlists/subdomain/resolvers.txt
RESOLVERS_URL=https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt
RESOLVERS_TRUSTED=~/hacking/wordlists/subdomain/resolvers-trusted.txt
RESOLVERS_TRUSTED_URL=https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt

# Wordlists
SUBDOMAINS=~/hacking/wordlists/subdomain/subdomains.txt
SUBDOMAINS_URL=https://raw.githubusercontent.com/y00k1sec/hacking-wordlists/main/subdomain/subdomains.txt
SUBDOMAINS_HUGE=~/hacking/wordlists/subdomain/subdomains-huge.txt
SUBDOMAINS_HUGE_URL=https://raw.githubusercontent.com/y00k1sec/hacking-wordlists/main/subdomain/subdomains-huge.txt
PERMUTATIONS=~/hacking/wordlists/subdomain/permutations.txt
PERMUTATIONS_URL=https://raw.githubusercontent.com/y00k1sec/hacking-wordlists/main/subdomain/permutations.txt

# Config File
AMASS_CONFIG=~/.config/amass/config.ini
BBOT_CONFIG=~/.config/bbot/secrets.yml
GITHUB_TOKENS=~/hacking/configs/github-tokens.txt
GITLAB_TOKENS=~/hacking/configs/gitlab-tokens.txt
SUBFINDER_CONFIG=~/.config/subfinder/provider-config.yaml
```

</details>

<details>

<summary>Required Folders</summary>

```bash
folders=(
    "~/hacking/tools"
    "~/hacking/configs"
    "~/hacking/wordlists"
)

for folder in "${folders[@]}"; do
    mkdir -p "$folder"
done
```

</details>

<details>

<summary>Subdomain Wordlists</summary>

```bash
wget -q -O - "$SUBDOMAINS_URL" >"$SUBDOMAINS"
wget -q -O - "$SUBDOMAINS_HUGE_URL" >"$SUBDOMAINS_HUGE"
wget -q -O - "$PERMUTATIONS_URL" >"$PERMUTATIONS"
```

</details>

<details>

<summary>DNS Resolvers</summary>

```bash
# Update the resolver if one day has passed since the last update
if [[ ! -f $RESOLVERS ]] || [[ $(find "$RESOLVERS" -mtime +0) ]]; then
    echo "[INF] It’s been over a day since the last update of the DNS resolver, it will be updated soon."
    [[ -f $RESOLVERS ]] && rm $RESOLVERS
    iresolver --target $RESOLVERS_URL --threads 2000 --output $RESOLVERS &>/dev/null
fi

# Update the trusted resolver if seven days have passed since the last update
if [[ ! -f $RESOLVERS_TRUSTED ]] || [[ $(find "$RESOLVERS_TRUSTED" -mtime +6) ]]; then
    echo "[INF] It’s been over seven day since the last update of the trusted DNS resolver, it will be updated soon."
    [[ -f $RESOLVERS_TRUSTED ]] && rm $RESOLVERS_TRUSTED
    iresolver --target $RESOLVERS_TRUSTED_URL --threads 2000 --output $RESOLVERS_TRUSTED &>/dev/null
fi
```

</details>

<details>

<summary>Other utilities</summary>

* You could refer to [https://pypi.org/project/isubsum/](https://pypi.org/project/isubsum/) to installation [`isubsum`](https://pypi.org/project/isubsum/)
  * `pipx install isubsum`

</details>

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

# Extract results and save to JSON
isubsum --source brute --resolved 1
```

## NoError Enum

```bash
noerror_enum() {
    # Identify Wildcard
    if [[ $(echo "error.abc.xyz.$DOMAIN" | dnsx -r "$RESOLVERS" -rcode noerror,nxdomain -retry 3 -silent | cut -d ' ' -f 2) == "[NXDOMAIN]" ]]; then
        # Continue if the domain name does not have a wildcard
        dnsx -d "$DOMAIN" -r "$RESOLVERS" -w "$SUBDOMAINS" -silent -rcode noerror | cut -d ' ' -f 1 | anew -q noerror.txt
    else
        echo "Wildcard detected, skipping NoError Enum"
    fi
} && noerror_enum

# Extract results and save to JSON
isubsum --source noerror --resolved 1
```

## Passive Source

```bash
# https://github.com/owasp-amass/amass/tree/v3.23.3
amass enum -passive -d $DOMAIN -silent -timeout 15 -o amass.txt

# https://github.com/blacklanternsecurity/bbot
bbot -t $DOMAIN -f subdomain-enum -rf passive -em massdns -y -s -om json |
    jq -r 'select(.scope_distance==0) | select(.type=="DNS_NAME") | .data' >bbot.txt

# https://github.com/cemulus/crt
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
        rm $result
    fi
done

# Extract results and save to JSON
# isubsum --source passive --resolved 0
```

### Alive Verification

```bash
puredns resolve passive.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write alive.txt \
    &>/dev/null

# Firmly Resolve
puredns resolve alive.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write alive.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null

# Extract results and save to JSON
isubsum --source alive --resolved 1
```

## Alter Combo

> This method requires initial [Alive Subdomain Verification](subdomain-enumeration.md#alive-verification).

> Choose one of them. (I usually use alterx).

> Best when the number of subdomains is 500 to 1000.

<details>

<summary><a href="https://github.com/infosec-au/altdns">altdns</a> #large #comprehensive</summary>

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

# Extract results and save to JSON
isubsum --source alter --resolved 1
```

</details>

<details>

<summary><a href="https://github.com/projectdiscovery/alterx">alterx</a> #stdin #customizable</summary>

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

# Extract results and save to JSON
isubsum --source alter --resolved 1
```

</details>

<details>

<summary><a href="https://github.com/Josue87/gotator">gotator</a> #depth #customizable</summary>

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

# Extract results and save to JSON
isubsum --source alter --resolved 1
```

</details>

<details>

<summary><a href="https://github.com/resyncgg/ripgen">ripgen</a> #high-performance</summary>

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

# Extract results and save to JSON
isubsum --source alter --resolved 1
```

</details>

## AI Generate

> This method requires initial [Alive Subdomain Verification](subdomain-enumeration.md#alive-verification).

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

# Extract results and save to JSON
isubsum --source intelli --resolved 1
```

## DNS Enum

> This method requires [Get DNS Record](domain-based.md#get-dns-record), If deep subdomain enumeration is not required, proceed directly to the [final alive verification](subdomain-enumeration.md#final-alive-verification).

```bash
dnsx -r resolvers_trusted.txt -a -aaaa -cname -ns -ptr -mx -soa \
    -silent -retry 3 -json \
    -o dnsx.json \
    >>dnsx.log

cat dnsx.json | jq -r 'try .a[], try .aaaa[], try .cname[], try .ns[], try .ptr[], try .mx[], try .soa[]' 2>/dev/null \
    | grep "\.${DOMAIN}$" \
    | grep -E '^((http|https):\/\/)?([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{1,}(\/.*)?$' \
    >dnsx.txt
cat dnsx.json | jq -r '.a[]' 2>/dev/null | sort -u \
    | hakip2host \
    | cut -d ' ' -f 3 | unfurl -u domains \
    | sed -e 's/*\.//' -e 's/\.$//' -e '/\./!d' \
    | grep "\.${DOMAIN}$" \
    >hakip2host.txt

cat dnsx.txt hakip2host.txt >dnsx_and_hakip2host.txt
puredns resolve dnsx_and_hakip2host.txt \
    -r resolvers.txt --resolvers-trusted resolvers_trusted.txt \
    -l 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch  1500000 \
    -w puredns.txt \
    &>puredns.log 2>&1

sort -u puredns.txt -o puredns.txt

# View Data
cat puredns.txt
```

## Web Scraping

> This method requires [Website Probing](domain-based.md#website-probing). If deep subdomain enumeration is not required, proceed directly to the [final alive verification](subdomain-enumeration.md#final-alive-verification).

> This method requires a large number of HTTP requests to the target’s live subdomain list.

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

# Extract results and save to JSON
isubsum --sources scrap --resolved 1
```

## Google Analytics

> This method requires [Website Probing](domain-based.md#website-probing). If deep subdomain enumeration is not required, proceed directly to the [final alive verification](subdomain-enumeration.md#final-alive-verification).

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
* **Domain Gathering:** This method is also applicable to [domain enumeration](broken-reference).

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

# Extract results and save to JSON
isubsum --sources analy --resolved 1
```

## Recursive

> A more in-depth and comprehensive subdomain enumeration requires a significant amount of API quotas and disk performance resources.
>
> If you’ve got the resources and need deep subdomain enumeration, go ahead and turn these options on. If not, keep them as your plan B for in-depth enum.

### Recursive Passive

```bash
# https://github.com/trickest/dsieve
dsieve -if subdomains.txt -f 3 -top 10 > dsieve.txt

# https://github.com/owasp-amass/amass/tree/v3.23.3
amass enum -passive -df dsieve.txt -nf subdomains.txt -timeout 30 -o amass.txt

puredns resolve amass.txt \
    --rate-limit 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w recursive_passive.txt &&
    rm dsieve.txt
```

### Recursive Brute

> When the number of alive subdomain names lists is 500, the estimated time is 50 minutes

```bash
# https://github.com/trickest/dsieve
dsieve -if subdomains.txt -f 3 -top 10 >dsieve.txt

# https://github.com/resyncgg/ripgen
ripgen -d dsieve.txt -w $PERMUTATIONS >ripgen.txt

puredns resolve ripgen.txt \
    --rate-limit 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w recursive_brute_1.txt \
    &> /dev/null &&
    rm ripgen.txt

# https://github.com/Josue87/gotator
gotator -sub recursive_brute-1.txt -perm $PERMUTATIONS \
    -depth 1 -numbers 3 -mindup -adv -md -silent |
    anew gotator.txt

puredns resolve gotator.txt \
    --rate-limit 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w recursive_brute_2.txt \
    &> /dev/null &&
    rm gotator.txt

# View Data
cat recursive_brute_1.txt recursive_brute_2.txt | sort-u
```

## Final Alive Verification

```bash
cat subdomains.json | jq -r '.[] | select(.resolved==1) | .subdomain' > subdomains.txt

puredns resolve subdomains.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write final.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null

# Extract results and save to JSON
isubsum --sources final --resolved 1
```

