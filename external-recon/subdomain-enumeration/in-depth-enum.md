# In-depth Enum

## Permutation

* Best when the number of subdomains is 500 to 1000
* **Features of different tools**
  * [altdns](https://github.com/infosec-au/altdns): large, comprehensive
  * [alterx](https://github.com/projectdiscovery/alterx): stdin, customizable
  * [gotator](https://github.com/Josue87/gotator): depth, customizable
  * [ripgen](https://github.com/resyncgg/ripgen): high-performance

```bash
jq -r '.[]|select(.resolved==1)|.subdomain' subdomains.json >resolved.txt
```

```bash
# https://github.com/infosec-au/altdns
altdns -i resolved.txt -w $PERMUTATIONS -o altdns.txt
puredns resolve altdns.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> permutation_altdns.txt &&
    rm altdns.txt

# Round 2
altdns -i permutation_altdns.txt -w $PERMUTATIONS -o altdns.txt
puredns resolve altdns.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> permutation_altdns.txt &&
    rm altdns.txt
 
# Final Resolve
sort -u permutation_altdns.txt -o permutation_altdns.txt
puredns resolve permutation_altdns.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write permutation_altdns.txt \
    &>/dev/null
```

```bash
# https://github.com/projectdiscovery/alterx
alterx -l resolved.txt -enrich -silent | anew -q alterx.txt
puredns resolve alterx.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> permutation_alterx.txt &&
    rm alterx.txt

# Round 2
alterx -l permutation_alterx.txt -enrich -silent | anew -q alterx.txt
puredns resolve alterx.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> permutation_alterx.txt &&
    rm alterx.txt

# Final Resolve
sort -u permutation_alterx.txt -o permutation_alterx.txt
puredns resolve permutation_alterx.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write permutation_alterx.txt \
    &>/dev/null
```

```bash
# https://github.com/Josue87/gotator
gotator -sub resolved.txt -perm $PERMUTATIONS -depth 1 -numbers 3 -mindup -adv -md -silent | head -c 1G | anew -q gotator.txt
puredns resolve gotator.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> permutation_gotator.txt &&
    rm gotator.txt

# Round 2
gotator -sub permutation_gotator.txt -perm $PERMUTATIONS -depth 1 -numbers 3 -mindup -adv -md -silent | head -c 1G | anew -q gotator.txt
puredns resolve gotator.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> permutation_gotator.txt &&
    rm gotator.txt

# Final Resolve
sort -u permutation_gotator.txt -o permutation_gotator.txt
puredns resolve permutation_gotator.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write permutation_gotator.txt \
    &>/dev/null
```

```bash
# https://github.com/resyncgg/ripgen
ripgen -d resolved.txt -w $PERMUTATIONS | head -c 1G | anew -q ripgen.txt
puredns resolve ripgen.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> permutation_ripgen.txt &&
    rm ripgen.txt

# Round 2
ripgen -d permutation_ripgen.txt -w $PERMUTATIONS | head -c 1G | anew -q ripgen.txt
puredns resolve ripgen.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --quiet >> permutation_ripgen.txt &&
    rm ripgen.txt

# Final Resolve
sort -u permutation_ripgen.txt -o permutation_ripgen.txt
puredns resolve permutation_ripgen.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write permutation_ripgen.txt \
    &>/dev/null
```

```bash
# rm resolved.txt

python3 summarizer.py --source permutation_altdns --resolved 1
python3 summarizer.py --source permutation_alterx --resolved 1
python3 summarizer.py --source permutation_gotator --resolved 1
python3 summarizer.py --source permutation_ripgen --resolved 1

# cat subdomains.json | jq '.[] | select(.sources == ["permutation_altdns"])'
# cat subdomains.json | jq '.[] | select(.sources == ["permutation_alterx"])'
# cat subdomains.json | jq '.[] | select(.sources == ["permutation_gotator"])'
# cat subdomains.json | jq '.[] | select(.sources == ["permutation_ripgen"])'
```

## AI Generate

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

## Recursive Passive

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

## Recursive Brute

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

## Google Analytics

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

## DNS zone transfer

{% code title="https://github.com/darkoperator/dnsrecon" %}
```bash
dnsrecon -t axfr -d target.com
```
{% endcode %}

## NoError

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
