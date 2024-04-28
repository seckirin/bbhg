# Subdomain Enum

## Passive Enum

Use [bbot](https://github.com/blacklanternsecurity/bbot), [subfinder](https://github.com/projectdiscovery/subfinder), [github-subdomain](https://github.com/gwen001/github-subdomains), [gitlab-subdomain](https://github.com/gwen001/gitlab-subdomains) & [crt](https://github.com/cemulus/crt).

{% code title="Use bbot passive" %}
```bash
bbot -t $DOMAIN -f subdomain-enum -rf passive -em massdns -o .scans/outs \
  -n bbot -om json -y \
  $( [[ $PROXY == true ]] && echo "-c http_proxy=$PROXY_ADDRESS" || echo "" )

cat .scans/outs/bbot/output.ndjson \
  | jq -r 'select(.scope_distance==0) | select(.type=="DNS_NAME") | .data' \
  > .scans/subs/bbot.txt; cat .scans/subs/bbot.txt | wc -l
```
{% endcode %}

{% code title="Use subfinder passive" %}
```bash
subfinder -d $DOMAIN -all -v -oJ -cs -es github \
  -o .scans/subfinder/subfinder.json \
  $( [[ $PROXY == true ]] && echo "-proxy $PROXY_ADDRESS" || echo "" )

cat .scans/outs/subfinder/subfinder.json | jq -r '.host' \
  > .scans/subs_subfinder.txt; \
  cat .scans/subs/subfinder.txt | wc -l
```
{% endcode %}

{% code title="Use github-subdomain passive" %}
```bash
github-subdomains -d $DOMAIN -t $GITHUB_TOKENS \
  -o .scans/subs/github-subdomain.txt \
  $( [[ $DEEP == true ]] && echo "" || echo "-k -q")

cat .scans/subs/github-subdomain.txt | wc -l
```
{% endcode %}

{% code title="Use gitlab-subdomain passive" %}
```bash
gitlab-subdomains -d $DOMAIN -t $GITLAB_TOKEN \
  | anew .scans/subs/gitlab-subdomain.txt; \
  rm ./${DOMAIN}.txt

cat .scans/subs/gitlab-subdomain.txt | wc -l
```
{% endcode %}

{% code title="Use crt passive" %}
```bash
crt -s -json -l 999999 $DOMAIN | jq -r '.[].subdomain' \
    | sed -e 's/^\*\.//' | sort -u \
    > .scans/subs/crt.txt

cat .scans/subs/crt.txt | wc -l
```
{% endcode %}

{% code title="Clean passive" %}
```bash
cat .scans/subs/* | sort -u > .scans/subs/no_resolved.txt
cat .scans/subs/no_resolved.txt | wc -l
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

## Active DNS resolution

Use [puredns](https://github.com/d3mondev/puredns) & [tlsx](https://github.com/projectdiscovery/tlsx)

{% code title="Use puredns active" %}
```bash
PUREDNS_PUBLIC_LIMIT=0
PUREDNS_TRUSTED_LIMIT=400
PUREDNS_WILDCARDTEST_LIMIT=30
PUREDNS_WILDCARDBATCH_LIMIT=1500000

resolvers_update # recommendations
resolvers_trusted_update

puredns resolve .scans/subs/no_resolved.txt \
    -r $RESOLVER_PATH --resolvers-trusted $RESOLVER_TRUSED_PATH \
    -l $PUREDNS_PUBLIC_LIMIT \
    --rate-limit-trusted $PUREDNS_TRUSTED_LIMIT \
    --wildcard-tests $PUREDNS_WILDCARDTEST_LIMIT \
    --wildcard-batch $PUREDNS_WILDCARDBATCH_LIMIT \
    --write .scans/outs/puredns-resolve/valid_domains.txt \
    --write-wildcards .scans/outs/puredns-resolve/wildcards.txt \
    --write-massdns .scans/outs/puredns-resolve/massdns.txt
cat .scans/outs/puredns-resolve/valid_domains.txt | anew .scans/subs/resolved.txt | wc -l
```
{% endcode %}

{% code title="Use tlsx active" %}
```bash
cat .scans/subs/resolved.txt | tlsx -san -cn -ro -c 1000 -silent \
    $( [[ $DEEP == false ]] && echo "" || echo "-p \"21,22,25,80,110,135,143,261,271,324,443,448,465,563,614,631,636,664,684,695,832,853,854,990,993,989,992,994,995,1129,1131,1184,2083,2087,2089,2096,2221,2252,2376,2381,2478,2479,2482,2484,2679,2762,3077,3078,3183,3191,3220,3269,3306,3410,3424,3471,3496,3509,3529,3539,3535,3660,36611,3713,3747,3766,3864,3885,3995,3896,4031,4036,4062,4064,4081,4083,4116,4335,4336,4536,4590,4740,4843,4849,5443,5007,5061,5321,5349,5671,5783,5868,5986,5989,5990,6209,6251,6443,6513,6514,6619,6697,6771,7202,7443,7673,7674,7677,7775,8243,8443,8991,8989,9089,9295,9318,9443,9444,9614,9802,10161,10162,11751,12013,12109,14143,15002,16995,41230,16993,20003\"" ) \
    | grep \.${DOMAIN}$ | anew .scans/subs/resolved.txt | wc -l
```
{% endcode %}

{% code title="Clean active" %}
```bash
cat .scans/subs/resolved.txt | anew subdomains/subdomains.txt | wc -l
```
{% endcode %}

## NOERROR Enum

Use [dnsx](https://github.com/projectdiscovery/dnsx).

{% code title="Use dnsx noerror" %}
```bash
if [[ $(echo "${RANDOM}.${RANDOM}.${RANDOM}.$DOMAIN" | dnsx -r $RESOLVER_FILE -rcode noerror,nxdomain -retry 3 -silent | cut -d ' ' -f2) == "[NXDOMAIN]" ]];
then
  echo "NXDOMAIN"
  $([[ $DEEP == true ]] && echo "-w $SUBDOMAIN_DICT_BIG_FILE" || echo "-w $SUBDOMAIN_DICT_FILE")
  dnsx -d $DOMAIN -rcode noerror -silent \
    -r $RESOLVER_FILE \
    $([[ $DEEP == true ]] && echo "-w $SUBDOMAIN_DICT_BIG_FILE" || echo "-w $SUBDOMAIN_DICT_FILE") \
    | cut -d' ' -f1 | anew .scans/subs/resolved_noerror.txt
  cat .scans/subs/resolved_noerror.txt | wc -l
fi
```
{% endcode %}

## Brute Force

Use [puredns](https://github.com/d3mondev/puredns).

{% code title="Use puredns brute" %}
```bash
```
{% endcode %}

## Other Technology

### DNS zone transfer

{% code title="https://github.com/darkoperator/dnsrecon" %}
```bash
dnsrecon -t axfr -d target.com
```
{% endcode %}

## Preparations

{% code title="ENVIRONMENT" %}
```bash
SRC_NAME=src_name
TARGET_NAME=target_name
DOMAIN=target.com

GITHUB_TOKENS=token1,token2,token3
GITLAB_TOKEN=token1

TIMEPURE=$(date +"%T")
TIMEFULL=$(date +"%F %T")
TIMELOG=$(date +"%Y%m%d_%H%M%S")

PROXY=true
PROXY_ADDRESS=socks5://127.0.0.1:6153
[[ $PROXY == true ]] \
    && alias PON="export ALL_PROXY=${PROXY_ADDRESS}" \
    && alias POFF="unset ALL_PROXY" \
    && PON \
    || unset ALL_PROXY
echo $PROXY $ALL_PROXY

DEEP=false

RESOLVER_FILE=.dicts/resolvers.txt
RESOLVER_URL=https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt
RESOLVER_TRUSTED_FILE=.dicts/resolvers-trusted.txt
RESOLVER_TRUSTED_URL=https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt

SUBDOMAIN_DICT_FILE=.dicts/subdomain_dict.txt
SUBDOMAIN_DICT_URL=https://raw.githubusercontent.com/yuukisec/hack-dict/main/subdomains/subdomains.txt
SUBDOMAIN_DICT_BIG_FILE=.dicts/subdomain_dict_big.txt
SUBDOMAIN_DICT_BIG_URL=https://raw.githubusercontent.com/n0kovo/n0kovo_subdomains/main/n0kovo_subdomains_huge.txt
```
{% endcode %}

{% code title="DIRECTORY" %}
```bash
mkdir -p ~/hack/srcs/${SRC_NAME}/${TARGET_NAME}
cd ~/hack/srcs/${SRC_NAME}/${TARGET_NAME}
mkdir -p domains subdomains websites \
    .scans .dicts .logs \
        .scans/subs \
        .scans/outs \
            .scans/outs/bbot \
            .scans/outs/subfinder \
            .scans/outs/puredns-resolve
echo $DOMAIN >> domains/domains.txt
```
{% endcode %}

{% code title="RESOLVER" %}
```bash
iresolver -threads 200 -retry 1 -count 10000 \
    -target $RESOLVER_URL -output $RESOLVER_FILE
iresolver -threads 200 -retry 3 \
    -target $RESOLVER_TRUSTED_URL -output $RESOLVER_TRUSTED_FILE

cat ${RESOLVER_FILE} | wc -l
cat ${RESOLVER_TRUSED_FILE} | wc -l
```
{% endcode %}

{% code title="DICTIONARY" %}
```bash
wget -q -O - $SUBDOMAIN_DICT_URL > $SUBDOMAIN_DICT_FILE
wget -q -O - $SUBDOMAIN_DICT_BIG_URL > $SUBDOMAIN_DICT_BIG_FILE
```
{% endcode %}

{% code title="FUNCTION" %}
```bash
function resolvers_update() {
  rm $RESOLVER_FILE
  iresolver -threads 200 -retry 1 -count 7777 \
    -target $RESOLVER_URL -output $RESOLVER_FILE
  cat $RESOLVER_FILE | wc -l
}

function resolvers_trusted_update() {
  rm $RESOLVER_TRUSTED_FILE
  iresolver -threads 200 -retry 3 \
    -target $RESOLVER_TRUSTED_URL -output $RESOLVER_TRUSTED_FILE
  cat $RESOLVER_TRUSTED_FILE | wc -l
}
```
{% endcode %}
