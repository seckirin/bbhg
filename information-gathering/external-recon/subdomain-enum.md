# Subdomain Enum

## Passive

Use [bbot](https://github.com/blacklanternsecurity/bbot), [subfinder](https://github.com/projectdiscovery/subfinder), [github-subdomain](https://github.com/gwen001/github-subdomains), [gitlab-subdomain](https://github.com/gwen001/gitlab-subdomains) & [crt](https://github.com/cemulus/crt).

{% code title="bbot" %}
```bash
# Single Target
bbot -t $DOMAIN -f subdomain-enum -rf passive -em massdns \
    -y $( [[ $PROXY == true ]] && echo "-c http_proxy=$PROXY_ADDRESS" || echo "") \
    -om json -n bbot -o $BBOT_OUT

# View Data
cat ${BBOT_OUT}/bbot/output.ndjson \
    | jq -r 'select(.scope_distance==0) | select(.type=="DNS_NAME") | .data' \
    | sort -u
```
{% endcode %}

{% code title="crt" %}
```bash
# Single Target
crt -s -l 999999 -json $DOMAIN | tee $CRT_OUT

# View Data
cat $CRT_OUT | jq -r '.[].subdomain' | sed -e 's/^\*\.//' | sort -u
```
{% endcode %}

{% code title="github-subdomains" %}
```sh
# Single Target
github-subdomains -d $DOMAIN -t $GITHUB_TOKENS \
    $( [[ $DEEP == true ]] && echo "" || echo "-k -q") \
    -o $GITHUB_SUBDOMAINS_OUT

# View Data
cat $GITHUB_SUBDOMAINS_OUT
```
{% endcode %}

{% code title="gitlab-subdomains" %}
```bash
# Single Target
gitlab-subdomains -d $DOMAIN -t $GITLAB_TOKEN | tee $GITLAB_SUBDOMAINS_OUT

# View Data
cat $GITLAB_SUBDOMAINS_OUT
```
{% endcode %}

{% code title="subfinder" %}
```bash
# Single Target
subfinder -d ${DOMAIN} -all -v -es github \
    $( [[ $PROXY == true ]] && echo "-proxy $PROXY_ADDRESS" || echo "") \
    -o ${SUBFINDER_OUT}

# Multiple Targets
subfinder -l ${DOMAINS} -all -v -es github $( [[ $PROXY == true ]] && echo \
    "-proxy $PROXY_ADDRESS" || echo "") \
    -o ${SUBFINDER_OUT}

# View Data
cat ${SUBFINDER_OUT} | jq -r '.host'
```
{% endcode %}

<details>

<summary>Abandoned</summary>

{% code title="https://github.com/owasp-amass/amass" overflow="wrap" %}
```bash
DOMAIN=target.com
NOW=$(date +"%F")
NOWT=$(date +"%T")

LOGFILE=".logs/amass-${NOW}_${NOWT}.txt"

AMASS_ENUM_TIMEOUT=180
AMASS_CONFIG=~/.config/amass/config.ini

amass enum -passive -d $DOMAIN -config $AMASS_CONFIG -timeout $AMASS_ENUM_TIMEOUT -silent -json .scan/amass_json.json 2>>"$LOGFILE" >>/dev/null
```
{% endcode %}

</details>

## Active

Use [puredns](https://github.com/d3mondev/puredns) & [tlsx](https://github.com/projectdiscovery/tlsx)

{% code title="puredns resolve" %}
```bash
# Puredns resolve
puredns resolve ${CLEANER_ROOT}/no_resolved.txt \
    -r $RESOLVERS --resolvers-trusted $RESOLVERS_TRUSTED \
    -l 1000 \
    --rate-limit-trusted 100 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w ${PUREDNS_RESOLVE_OUT}

# View Data
cat ${PUREDNS_RESOLVE_OUT}
```
{% endcode %}

{% code title="tlsx" %}
```bash
cat ./scans/subs/resolved.txt \
    | tlsx -san -cn -ro -c 1000 -silent \
    | grep \.${DOMAIN}$ > ${TLSX_RESOLVE_OUT}

# View Data
cat ${TLSX_RESOLVE_OUT}
```
{% endcode %}

## NOERROR

{% hint style="success" %}
If the domain name does not have a wildcard
{% endhint %}

Use [dnsx](https://github.com/projectdiscovery/dnsx).

{% code title="Use dnsx noerror" %}
```bash
# Identify Wildcard
echo error.abc.xyz.target.com | dnsx -r resolvers.txt -rcode noerror,nxdomain -retry 3 -silent

# NOERROR Enumeration
dnsx -d target.com -rcode noerror -silent -r resolves.txt -w subdomains.txt | cur -d ' ' -f 1
```
{% endcode %}

## Brute

Use [puredns](https://github.com/d3mondev/puredns).

{% code title="Use puredns brute" %}
```bash
DEEP=true # If your machine works well

puredns bruteforce \
    $([[ $DEEP == true ]] && echo "$SUBS_WORDLISTS_HUGE" || echo "$SUBS_WORDLISTS") \
    $DOMAIN \
    -r $RESOLVERS --resolvers-trusted $RESOLVERS_TRUSTED \
    -l 1000 \
    --rate-limit-trusted 100 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -w $PUREDNS_BRUTE_OUT

DEEP=false

# View Data
cat $PUREDNS_BRUTE_OUT
```
{% endcode %}

## Permutation

{% hint style="success" %}
If the number of subdomains is less than 500
{% endhint %}

```bash
resolvers_update # recommendations
resolvers_trusted_update

cat .scans/subs/resolved.txt | wc -l
gotator -sub .scans/subs/resolved.txt -perm $PERMUTATION_DICT_FILE \
    -depth 1 -numbers 3 -mindup -adv -md -silent > .dicts/gotator-outptu.txt
cat .dicts/gotator-outptu.txt | wc -l
puredns resolve .dicts/gotator-outptu.txt \
    -r $RESOLVER_FILE --resolvers-trusted $RESOLVER_TRUSTED_FILE \
    -l $PUREDNS_PUBLIC_LIMIT \
    --rate-limit-trusted $PUREDNS_TRUSTED_LIMIT \
    --wildcard-tests $PUREDNS_WILDCARDTEST_LIMIT \
    --wildcard-batch $PUREDNS_WILDCARDBATCH_LIMIT \
    --write .scans/outs/puredns-permutation/valid_domains.txt \
    --write-wildcards .scans/outs/puredns-permutation/wildcards.txt \
    --write-massdns .scans/outs/puredns-permutation/massdns.txt
cat .scans/outs/puredns-permutation/valid_domains.txt
cat .scans/outs/puredns-permutation/valid_domains.txt | anew .scans/subs/resolved.txt | wc -l

# Recursion once
cat .scans/outs/puredns-permutation/valid_domains.txt | wc -l
gotator -sub .scans/outs/puredns-permutation/valid_domains.txt -perm $PERMUTATION_DICT_FILE \
    -depth 1 -numbers 3 -mindup -adv -md -silent > .dicts/gotator-outptu-2.txt
cat .dicts/gotator-outptu-2.txt | wc -l
puredns resolve .dicts/gotator-outptu-2.txt \
    -r $RESOLVER_FILE --resolvers-trusted $RESOLVER_TRUSTED_FILE \
    -l $PUREDNS_PUBLIC_LIMIT \
    --rate-limit-trusted $PUREDNS_TRUSTED_LIMIT \
    --wildcard-tests $PUREDNS_WILDCARDTEST_LIMIT \
    --wildcard-batch $PUREDNS_WILDCARDBATCH_LIMIT \
    --write .scans/outs/puredns-permutation/valid_domains-2.txt \
    --write-wildcards .scans/outs/puredns-permutation/wildcards-2.txt \
    --write-massdns .scans/outs/puredns-permutation/massdns-2.txt
```

## Regex Permutations

## Recursive

### Recursive Passive

### Recursive Brute

## DNS Record

## Scarping

## Google Analytics ID

## Other Technology

### DNS zone transfer

{% code title="https://github.com/darkoperator/dnsrecon" %}
```bash
dnsrecon -t axfr -d target.com
```
{% endcode %}

## Preparations

### RESOLVER

```bash
iresolver -threads 200 -retry 1 -count 10000 \
    -target $RESOLVER_URL -output $RESOLVER_FILE
iresolver -threads 200 -retry 3 \
    -target $RESOLVER_TRUSTED_URL -output $RESOLVER_TRUSTED_FILE

cat ${RESOLVER_FILE} | wc -l
cat ${RESOLVER_TRUSED_FILE} | wc -l
```

### DICTIONARY

```bash
wget -q -O - $SUBDOMAIN_DICT_URL > $SUBDOMAIN_DICT_FILE
wget -q -O - $SUBDOMAIN_DICT_BIG_URL > $SUBDOMAIN_DICT_BIG_FILE
wget -q -O - $PERMUTATION_DICT_URL > $PERMUTATION_DICT_FILE
```
