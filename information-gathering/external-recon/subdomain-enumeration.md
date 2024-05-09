# Subdomain Enumeration

{% hint style="info" %}
There are certain tasks that can enhance the efficiency of your subdomain enumeration. Please refer to the section titled [#preparations](subdomain-enumeration.md#preparations "mention") for more details. For variables used in the commands, please refer to the section titled [#environment-variables](subdomain-enumeration.md#environment-variables "mention").
{% endhint %}

## Passive Scan

Use [amass (v3)](https://github.com/owasp-amass/amass/tree/v3.23.3), [bbot](https://github.com/blacklanternsecurity/bbot), [crt](https://github.com/cemulus/crt), [github-subdomain](https://github.com/gwen001/github-subdomains), [gitlab-subdomain](https://github.com/gwen001/gitlab-subdomains) and [subfinder](https://github.com/projectdiscovery/subfinder).

```bash
# Use amass to scan vulnweb.com (limit time)
timeout -k 30s 30m \
    amass enum \
    -passive \
    -d vulnweb.com \
    -config "$HOME/.config/amass/config.ini" \
    -timeout 30 \
    -json amass.json \
    &>/dev/null

# Use amass to scan vulnweb.com (unlimited time)
amass enum \
    -passive \
    -d vulnweb.com \
    -config "$HOME/.config/amass/config.ini" \
    -json amass.json \
    &>/dev/null

# Extract subdomains from the amass scan results
jq -r '.name' amass.json |
    grep -E "^vulnweb.com$|\.vulnweb.com$" |
    anew -q amass.txt
```

```bash
# Use bbot to scan vulnweb.com
bbot \
    -t vulnweb.com \
    --flags subdomain-enum \
    --require-flags passive \
    --exclude-modules massdns \
    --yes \
    --silent \
    -om json \
    >bbot.json

# Extract subdomains from the bbot scan results
jq -r 'select(.type=="DNS_NAME") | select(.scope_distance==0) | .data' bbot.json |
    grep -E "^vulnweb.com$|\.vulnweb.com$" |
    anew -q bbot.txt
```

```bash
# Use crt to scan vulnweb.com
crt \
    -s \
    -l 999999 \
    -json \
    -o crt.json \
    vulnweb.com \
    &>/dev/null

# Extract subdomains from the crt scan results
jq -r '.[].subdomain' crt.json |
    sed -e 's/^\*\.//' |
    grep -E "^vulnweb.com$|\.vulnweb.com$" |
    anew -q crt.txt
```

```bash
# Use github-subdomains to scan vulnweb.com (quick)
github-subdomains \
    -d vulnweb.com \
    -t "$TOOLS/github_tokens.txt" \
    -k \
    -q \
    &>github-subdomains.log

# Use github-subdomains to scan vulnweb.com (slow)
github-subdomains \
    -d vulnweb.com \
    -t "$TOOLS/github_tokens.txt" \
    &>github-subdomains.log

# Extract subdomains from the github-subdomains scan results
sort -u vulnweb.com.txt -o github-subdomains.txt && rm vulnweb.com.txt
```

```bash
# Use gitlab-subdomains to scan vulnweb.com
cp $TOOLS/gitlab_tokens.txt .tokens

gitlab-subdomains \
    -d vulnweb.com \
    &>gitlab-subdomains.log

# Extract subdomains from the gitlab-subdomains scan results
sort -u vulnweb.com.txt -o gitlab-subdomains.txt && rm vulnweb.com.txt && rm .tokens
```

```bash
# Use subfinder to scan vulnweb.com (all sources)
subfinder \
    -domain vulnweb.com \
    -provider-config "$HOME/.config/subfinder/provider-config.yaml" \
    -s chaos \
    -v \
    $([[ -n "${HTTP_PROXY}" ]] && echo " -proxy ${HTTP_PROXY}") \
    -json \
    -o subfinder.json \
    &>subfinder.log

# Use subfinder to scan vulnweb.com (single sources)
subfinder \
    -domain vulnweb.com \
    -provider-config "$HOME/.config/subfinder/provider-config.yaml" \
    -all \
    -v \
    $([[ -n "${HTTP_PROXY}" ]] && echo " -proxy ${HTTP_PROXY}") \
    -json \
    -o subfinder.json \
    &>subfinder.log

# Extract subdomains from the subfinder scan results
jq -r '.host' subfinder.json |
     grep -E "^vulnweb.com$|\.vulnweb.com" |
     anew -q subfinder.txt
```

```bash
# Summarize all results
local results=(
    "amass.txt"
    "bbot.txt"
    "crt.txt"
    "github-subdomains.txt"
    "gitlab-subdomains.txt"
    "subfinder.txt"
)

for result in "${results[@]}"; do
    if [[ -s "$result" ]]; then
         anew -q passive.txt <"$result"
    fi
done
```

## Active Resolve

Use [puredns](https://github.com/d3mondev/puredns) & [tlsx](https://github.com/projectdiscovery/tlsx)

{% code title="Use puredns active" %}
```bash
# Use puredns to resolve passive.txt
puredns resolve passive.txt \
    --resolvers "$TOOLS/resolvers/resolvers.txt" \
    --resolvers-trusted "$TOOLS/resolvers/resolvers_trusted.txt" \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write puredns-resolve-valid.txt \
    --write-wildcards puredns-resolve-wildcards.txt \
    &>/puredns-resolve.log

# Summarize the puredns resolve results
anew -q active.txt <puredns-resolve-valid.txt
anew -q active.txt <puredns-resolve-wildcards.txt
```
{% endcode %}

## Brute

Use [puredns](https://github.com/d3mondev/puredns).

{% code title="Use puredns brute" %}
```bash
DEEP=true # If your machine works well

puredns bruteforce subs_wordlists.txt $DOMAIN \
    -r resolvers.txt --resolvers-trusted resolvers_trusted.txt \
    -l 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w puredns.txt

DEEP=false
```
{% endcode %}

## Permutation

{% hint style="info" %}
If the number of subdomains is less than 500
{% endhint %}

Use [gotator](https://github.com/Josue87/gotator) and [puredns](https://github.com/d3mondev/puredns)

```bash
gotator -sub resolved.txt -perm permutations.txt \
    -depth 1 -numbers 3 -mindup -adv -md -silent > gotator-1.txt

puredns resolve gotator-1.txt \
    -r $RESOLVERS --resolvers-trusted $RESOLVERS_TRUSTED \
    -l 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w puredns-1.txt

# Do it again
gotator -sub resolved.txt -perm puredns-1.txt \
    -depth 1 -numbers 3 -mindup -adv -md -silent > gotator-2.txt

puredns resolve gotator-2.txt \
    -r resolvers.txt --resolvers-trusted resolvers_trusted.txt \
    -l 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w puredns-2.txt

# View Data
cat puredns-1.txt puredns-2.txt | sort -u
```

## Machine Learning

use [regulator](https://github.com/cramppet/regulator) and [puredns](https://github.com/d3mondev/puredns).

```bash
python3 main.py -t $DOMAIN -f subdomains.txt -o regulator.txt

puredns resolve regulator.txt \
    -r resolvers.txt --resolvers-trusted resolvers_trusted.txt \
    -l 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w puredns.txt
```

## Recursive

### Recursive Passive

Use [dsieve](https://github.com/trickest/dsieve), [amass (v3)](https://github.com/owasp-amass/amass/tree/v3.23.3) and [puredns](https://github.com/d3mondev/puredns)

```bash
dsieve -if subdomains.txt -f 3 -top 10 > dsieve.txt

amass enum -passive -df dsieve.txt -nf subdomains.txt -timeout 30 -o amass.txt

puredns resolve amass.txt \
    -r resolvers.txt --resolvers-trusted resolvers_trusted.txt \
    -l 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w puredns.txt
```

### Recursive Brute

{% hint style="info" %}
If the number of subdomains is less than 500
{% endhint %}

{% hint style="info" %}
It may take more than 50 minutes
{% endhint %}

Use [dsieve](https://github.com/trickest/dsieve),  [ripgen](https://github.com/resyncgg/ripgen), [gotator](https://github.com/Josue87/gotator) and [puredns](https://github.com/d3mondev/puredns)

```bash
# First
dsieve -if subdomains.txt -f 3 -top 10 >./dsieve.txt

ripgen -d ./dsieve.txt -w subs_wordlists.txt >./ripgen.txt

puredns resolve ripgen.txt \
    -r resolvers.txt --resolvers-trusted resolvers_trusted.txt \
    -l 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w puredns-1.txt

# Do it again
gotator -sub resolved.txt -perm puredns-1.txt \
    -depth 1 -numbers 3 -mindup -adv -md -silent > gotator.txt

puredns resolve gotator.txt \
    -r resolvers.txt --resolvers-trusted resolvers_trusted.txt \
    -l 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w puredns-2.txt

# View Data
cat puredns-1.txt puredns-2.txt | sort -u
```

## DNS Record

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

## Web Scarping

{% hint style="info" %}
If the number of subdomains is less than 500
{% endhint %}

```bash
# First request for website data
cat subdomains.txt | httpx -follow-host-redirects -status-code \
    -threads 50 -rl 150 -timeout 10 -silent \
    -retries 2 -title -web-server -tech-detect \
    -location -no-color -json -o httpx-1.json \
    2>>"httpx-1.log" >/dev/null

# Filter domains and urls in the data
cat httpx-1.json | jq -r 'try .url' 2>/dev/null \
    | grep "\.${DOMAIN}$" \
    | grep -E '^((http|https):\/\/)?([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{1,}(\/.*)?$' \
    | sed "s/*.//" \
    | anew httpx-urls-1.txt \
    | unfurl -u domains \
    2>>/dev/null \
    | anew -q httpx-domains-1.txt

# Second request for website data
httpx -l httpx-urls-1.txt \
    -tls-grab -tls-probe -csp-probe -status-code \
    -threads 50 -rl 150 -timeout 10 -silent \
    -retries 2 -no-color -json \
    -o httpx-2.json \
    2>>"httpx-2.log" \
    >/dev/null

# Filter the domains and urls in the data and request the site data again
cat httpx-2.json | jq -r 'try ."tls-grab"."dns_names"[],try .csp.domains[],try .url' 2>/dev/null \
    | grep "$DOMAIN" 
    | grep -E '^((http|https):\/\/)?([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{1,}(\/.*)?$' \
    | sed "s/*.//" | sort -u \
    | httpx -silent \
    | anew httpx-urls-2.txt \
    | unfurl -u domains \
    2>>/dev/null \
    | anew -q httpx-domains-2.txt

# Clean httpx outptu
cat httpx-urls-1.txt httpx-urls-2.txt >httpx-urls-all.txt
cat httpx-domains-1.txt httpx-domains-2.txt > httpx-domains-all.txt

# Run katana
katana -silent -list httpx-urls-all.txt -jc -kf all -c 20 -d 2 -fs rdn \
    -o katana.txt \
    2>>katana.log \
    >/dev/null

# Clean katana outptu
sed -i '/^.\{2048\}./d' ${WRITE_PATH}/subdomains/scrap/katana.txt
cat katana.txt | unfurl -u domains 2>/dev/null \
    | grep "\.${DOMAIN}$" \
    | grep -E '^((http|https):\/\/)?([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{1,}(\/.*)?$' \
    >katana-domains.txt

# Clean all domains
cat httpx-domains-all.txt katana-domains.txt | sort -u \
    >httpx_and_katana_domains.txt

# Resolution
puredns resolve httpx_and_katana_domains.txt \
    -r resolvers.txt --resolvers-trusted resolvers_trusted.txt \
    -l 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w puredns.txt \
    &>puredns.log

sort -u puredns.txt -o puredns.txt

# View Data
cat puredns.txt
```

## Google Analytics ID

```bash
# Request for website data
analyticsrelationships -ch < httpx-urls-all.txt \
    >analyticsrelationships.txt \
    2>analyticsrelationships.log

# Clean outptu
cat analyticsrelationships.txt \
    | grep "\.$DOMAIN$\|^$DOMAIN$" \
    | grep -E '^((http|https):\/\/)?([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{1,}(\/.*)?$' \
    | sed "s/|__ //" \
    | anew -q analyticsrelationships-domains.txt

# Resolution
puredns resolve ${WRITE_PATH}/subdomains/analytics/analyticsrelationships-domains.txt \
    -r resolvers.txt --resolvers-trusted resolvers_trusted.txt \
    -l 0 \
    --rate-limit-trusted 200 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w puredns.txt \
    &>puredns.log

sort -u puredns.txt -o puredns.txt

# View Data
cat puredns.txt
```

## Other Technology

### DNS zone transfer

{% code title="https://github.com/darkoperator/dnsrecon" %}
```bash
dnsrecon -t axfr -d target.com
```
{% endcode %}

### NoError

{% hint style="info" %}
If the domain name does not have a wildcard
{% endhint %}

Use [dnsx](https://github.com/projectdiscovery/dnsx).

{% code title="Use dnsx noerror" %}
```bash
# Identify Wildcard
echo error.abc.xyz.target.com \
    | dnsx -r resolvers.txt -rcode noerror,nxdomain -retry 3 -silent

# NOERROR Enumeration
dnsx -d $DOMAIN -r resolvers.txt -silent -rcode noerror \
    -w subs_wordlists.txt
    | cut -d ' ' -f 1 | tee ./dnsx.txt

# View Data
cat ./dnsx.txt
```
{% endcode %}

## Variables

```bash
# GENERAL
export "$TOOLS"="$HOME/tools"
export "$OUTPUT"="$DOMAIN"
```

## Folders

```bash
# Check all the directories needed for script
local dirs=(
    "${TOOLS}"
    "${TOOLS}/wordlists"
    "${TOOLS}/resolvers"
    "${HOME}/.config/amass/"
    "${HOME}/.config/subfinder/"
    "${OUTPUT}/subdomains"
    "${OUTPUT}/subdomains/passive"
    "${OUTPUT}/subdomains/active"
)

for dir in "${dirs[@]}"; do
    mkdir -p "$dir"
done
```

## Resolvers

```bash
RESOLVERS="$TOOLS/resolvers/resolvers.txt"
RESOLVERS_URL="https://public-dns.info/nameservers.txt"
RESOLVERS_TRUSTED="$TOOLS/resolvers/resolvers_trusted.txt"
RESOLVERS_TRUSTED_URL="https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt"

iresolver -threads 2000 \
    -target "$RESOLVERS_URL" \
    -outptu "$RESOLVERS" \
    &>/dev/null

iresolver -threads 2000 \
    -target "$RESOLVERS_TRUSTED_URL" \
    -outptu "$RESOLVERS_TRUSTED" \
    &>/dev/null
```

## Wordlists

```bash
SUBDOMAINS="${TOOLS}/wordlists/subdomains.txt"
SUBDOMAINS_URL="https://raw.githubusercontent.com/yuukisec/ReconLists/main/subdomains/subdomains.txt"
SUBDOMAINS_HUGE="${TOOLS}/wordlists/subdomains_huge.txt"
SUBDOMAINS_HUGE_URL="https://raw.githubusercontent.com/yuukisec/ReconLists/main/subdomains/subdomains_huge.txt"
PERMUTATIONS="${TOOLS}/wordlists/permutations.txt"
PERMUTATIONS_URL="https://raw.githubusercontent.com/yuukisec/ReconLists/main/subdomains/permutations.txt"

wget -q -O - "$SUBDOMAINS_URL" >"$SUBDOMAINS"
wget -q -O - "$SUBDOMAINS_HUGE_URL" >"$SUBDOMAINS_HUGE"
wget -q -O - "$PERMUTATIONS_URL" >"$PERMUTATIONS"
```
