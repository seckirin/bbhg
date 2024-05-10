# In-depth Enum

## Recursive

* A more in-depth and comprehensive subdomain enumeration requires a significant amount of API quotas and disk performance resources.
* If you’ve got the resources and need deep subdomain enumeration, go ahead and turn these options on. If not, keep them as your plan B for in-depth enum.

### Recursive Passive

```bash
# https://github.com/trickest/dsieve
dsieve -if resolved.txt -f 3 -top 10 > dsieve.txt

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

* If the number of subdomains is less than 500
* It may take more than 50 minutes

```bash
# https://github.com/trickest/dsieve
dsieve -if resolved.txt -f 3 -top 10 >dsieve.txt

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

## DNS zone transfer

{% code title="https://github.com/darkoperator/dnsrecon" %}
```bash
dnsrecon -t axfr -d target.com
```
{% endcode %}

## NoError

* If the domain name does not have a wildcard

```bash
# https://github.com/projectdiscovery/dnsx
# Identify Wildcard
echo error.abc.xyz.target.com \
    | dnsx -r resolvers.txt -rcode noerror,nxdomain -retry 3 -silent

# NOERROR Enumeration
dnsx -d $DOMAIN -r $RESOLVERS -w $SUBDOMAINS -silent -rcode noerror \
    | cut -d ' ' -f 1 | tee dnsx.txt

# View Data
cat dnsx.txt
```
