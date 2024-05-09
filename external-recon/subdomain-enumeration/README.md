# Subdomain Enumeration

## Passive Enum

Use [amass (v3)](https://github.com/owasp-amass/amass/tree/v3.23.3), [bbot](https://github.com/blacklanternsecurity/bbot), [crt](https://github.com/cemulus/crt), [github-subdomain](https://github.com/gwen001/github-subdomains), [gitlab-subdomain](https://github.com/gwen001/gitlab-subdomains) and [subfinder](https://github.com/projectdiscovery/subfinder).

```bash
# limit time
timeout -k 30s "${AMASS_TIMEOUT}m" \
    amass enum -passive -d vulnweb.com -config "$AMASS_CONFIG" -json amass.json -timeout "${AMASS_TIMEOUT}" &>/dev/null

# unlimited time
amass enum -passive -d vulnweb.com -config "AMASS_CONFIG" -json amass.json &>/dev/null

# Extract subdomains from the amass scan results
jq -r '.name' amass.json |
    grep -E "^vulnweb.com$|\.vulnweb.com$" |
    anew -q amass.txt &&
    rm amass.json
```

```bash
# Use bbot to enum vulnweb.com
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
    anew -q bbot.txt &&
    rm bbot.json
```

```bash
# Use crt to enum vulnweb.com
crt \
    -s \
    -l 999999 \
    -json \
    -o crt.json \
    vulnweb.com \
    &>/dev/null

# Extract subdomains from the crt scan results
[[ -s crt.json ]] && jq -r '.[].subdomain' crt.json |
    sed -e 's/^\*\.//' |
    grep -E "^vulnweb.com$|\.vulnweb.com$" |
    anew -q crt.txt &&
    rm crt.json
```

```bash
# Use github-subdomains to enum vulnweb.com (quick)
github-subdomains \
    -d vulnweb.com \
    -t "$TOOLS/github_tokens.txt" \
    -k \
    -q \
    &>/dev/null

# Use github-subdomains to enum vulnweb.com (slow)
github-subdomains \
    -d vulnweb.com \
    -t "$TOOLS/github_tokens.txt" \
    &>/dev/null

# Extract subdomains from the github-subdomains scan results
sort -u vulnweb.com.txt -o github-subdomains.txt &&
    rm vulnweb.com.txt &&
```

```bash
# Use gitlab-subdomains to enum vulnweb.com
cp $TOOLS/gitlab_tokens.txt .tokens

gitlab-subdomains \
    -d vulnweb.com \
    &>/dev/null

# Extract subdomains from the gitlab-subdomains scan results
sort -u vulnweb.com.txt -o gitlab-subdomains.txt &&
    rm vulnweb.com.txt &&
    rm .tokens &&
```

```bash
# Use subfinder to enum vulnweb.com (all sources)
subfinder \
    -domain vulnweb.com \
    -provider-config "$HOME/.config/subfinder/provider-config.yaml" \
    -all \
    -v \
    $([[ -n "${HTTP_PROXY}" ]] && echo " -proxy ${HTTP_PROXY}") \
    -json \
    -o subfinder.json \
    &>/dev/null

# Use subfinder to enum vulnweb.com (single sources)
subfinder \
    -domain vulnweb.com \
    -provider-config "$HOME/.config/subfinder/provider-config.yaml" \
    -s chaos \
    -v \
    $([[ -n "${HTTP_PROXY}" ]] && echo " -proxy ${HTTP_PROXY}") \
    -json \
    -o subfinder.json \
    &>/dev/null

# Extract subdomains from the subfinder scan results
jq -r '.host' subfinder.json |
     grep -E "^vulnweb.com$|\.vulnweb.com" |
     anew -q subfinder.txt &&
     rm subfinder.json
```

```bash
# Summarize all passive enum results
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
    --write puredns-resolve-passive-valid.txt \
    --write-wildcards puredns-resolve-passive-wildcards.txt \
    &>/dev/null

# Extract and summarize subdomains from the puredns resolve results
anew -q active.txt <puredns-resolve-passive-valid.txt && rm puredns-resolve-passive-valid.txt
anew -q active.txt <puredns-resolve-passive-wildcards.txt && rm puredns-resolve-passive-wildcards.txt
```
{% endcode %}

## Brute Force

Use [puredns](https://github.com/d3mondev/puredns).

```bash
# Use puredns to brute force vulnweb.com (quick)
puredns bruteforce "$TOOLS/wordlists/subdomains.txt" vulnweb.com \
    --resolvers "$TOOLS/resolvers/resolvers.txt" \
    --resolvers-trusted "$TOOLS/resolvers/resolvers_trusted.txt" \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write puredns-bruteforce-valid.txt \
    --write-wildcards puredns-bruteforce-wildcards.txt \
    &>/dev/null

# Use puredns to brute force vulnweb.com (slow)
puredns bruteforce "$TOOLS/wordlists/subdomains_huge.txt" vulnweb.com \
    --resolvers "$TOOLS/resolvers/resolvers.txt" \
    --resolvers-trusted "$TOOLS/resolvers/resolvers_trusted.txt" \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write puredns-bruteforce-valid.txt \
    --write-wildcards puredns-bruteforce-wildcards.txt \
    &>/dev/null

# Extract and summarize subdomains from the puredns brute results
anew -q brute.txt <puredns-bruteforce-valid.txt && rm puredns-bruteforce-valid.txt
anew -q brute.txt <puredns-bruteforce-wildcards.txt && rm puredns-bruteforce-wildcards.txt
```

## Summarizing

```bash
# active_added_count=$(anew -d subdomains_temp.txt <active.txt | wc -l | tr -d ' ')
# brute_added_count=$(anew -d subdomains_temp.txt <brute.txt | wc -l | tr -d ' ')
anew -q subdomains_temp.txt <active.txt
anew -q subdomains_temp.txt <brute.txt

# Use puredns to resolve subdomains_temp.txt
puredns resolve subdomains_temp.txt \
    --resolvers "$TOOLS/resolvers/resolvers.txt" \
    --resolvers-trusted "$TOOLS/resolvers/resolvers_trusted.txt" \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write puredns-resolve-subdomains_temp-valid.txt \
    --write-wildcards puredns-resolve-subdomains_temp-wildcards.txt \
    &>/dev/null && rm subdomains_temp.txt

# Extract and summarize first round results
anew -q subdomains.txt <puredns-resolve-subdomains_temp-valid.txt && rm puredns-resolve-subdomains_temp-valid.txt
anew -q subdomains.txt <puredns-resolve-subdomains_temp-wildcards.txt && rm puredns-resolve-subdomains_temp-wildcards.txt
```
