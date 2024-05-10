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
```

## Permutation

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
jq -r '.[]|select(.resolved==1)|.subdomain' subdomains.json >resolved.txt
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
 
# Firmly Resolve
sort -u permutation_altdns.txt -o permutation_altdns.txt
puredns resolve permutation_altdns.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write permutation_altdns.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --source permutation_altdns --resolved 1
```

</details>

<details>

<summary>alterx</summary>

```bash
jq -r '.[]|select(.resolved==1)|.subdomain' subdomains.json >resolved.txt

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

# Firmly Resolve
sort -u permutation_alterx.txt -o permutation_alterx.txt
puredns resolve permutation_alterx.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write permutation_alterx.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --source permutation_alterx --resolved 1
```

</details>

<details>

<summary>gotator</summary>

```bash
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

# Firmly Resolve
sort -u permutation_gotator.txt -o permutation_gotator.txt
puredns resolve permutation_gotator.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write permutation_gotator.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --source permutation_gotator --resolved 1
```

</details>

<details>

<summary>ripgen</summary>

```bash
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

# Firmly Resolve
sort -u permutation_ripgen.txt -o permutation_ripgen.txt
puredns resolve permutation_ripgen.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write permutation_ripgen.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --source permutation_ripgen --resolved 1
```

</details>

<details>

<summary>Comparison Results (jq)</summary>

```bash
# Only one tool
cat subdomains.json | jq '.[] | select(.sources == ["permutation_altdns"])'
cat subdomains.json | jq '.[] | select(.sources == ["permutation_alterx"])'
cat subdomains.json | jq '.[] | select(.sources == ["permutation_gotator"])'
cat subdomains.json | jq '.[] | select(.sources == ["permutation_ripgen"])'

# Count the number of a tool
cat subdomains.json | jq '.[] | select(.sources[] == "permutation_altdns") | .subdomain' | wc -l
cat subdomains.json | jq '.[] | select(.sources[] == "permutation_alterx") | .subdomain' | wc -l
cat subdomains.json | jq '.[] | select(.sources[] == "permutation_gotator") | .subdomain' | wc -l
cat subdomains.json | jq '.[] | select(.sources[] == "permutation_ripgen") | .subdomain' | wc -l

# Only Permutation
cat subdomains.json | jq -r '.[] | select(.sources | all(. | contains("permutation")))'
```

</details>

## AI Generate

```bash
# https://github.com/cramppet/regulator
cat subdomains.json | jq -r '.[] | select(.resolved==1) | .subdomain' > resolved.txt
scan_pwd=$(pwd) && pushd $TOOLS/regulator && python3 main.py -t $DOMAIN -f $scan_pwd/resolved.txt -o $scan_pwd/regulator.txt && popd

puredns resolve regulator.txt \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write ai.txt \
    &>/dev/null &&
    rm regulator.txt

# Firmly Resolve
puredns resolve ai.txt \
    --rate-limit 150 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write ai.txt \
    --resolvers $RESOLVERS_TRUSTED \
    &>/dev/null
```

```bash
python3 summarizer.py --sources ai --resolved 1
```

## Web Scraping

```bash
cat subdomains.json | jq -r '.[] | select(.resolved==1) | .subdomain' > resolved.txt

cat resolved | httpx -follow-host-redirects -random-agent -status-code -silent -retries 2 -title -web-server -tech-detect -location -o webs_info.txt
```

## Google Analytics
