# Regular

## Passive Enum

```bash
# https://github.com/owasp-amass/amass/tree/v3.23.3
amass enum -passive -d $DOMAIN -silent -timeout 15 -o amass.txt &

# https://github.com/blacklanternsecurity/bbot
bbot -t $DOMAIN -f subdomain-enum -rf passive -em massdns -y -s -om json |
    jq -r 'select(.scope_distance==0) | select(.type=="DNS_NAME") | .data' >bbot.txt &

# https://github.com/cemulus/crt
crt -s -l 999999 -json $DOMAIN |
    jq -r '.[].subdomain' | sed -e 's/^\*\.//' > crt.txt &

# https://github.com/gwen001/github-subdomains
github-subdomains -d $DOMAIN -t $GITHUB_TOKENS \
    -k -q -o github-subdomains.txt &>/dev/null &

# https://github.com/gwen001/gitlab-subdomains
gitlab-subdomains -d $DOMAIN -t $(cat $GITLAB_TOKENS) &>/dev/null && mv $DOMAIN.txt gitlab-subdomains.txt

# https://github.com/projectdiscovery/subfinder
subfinder -d $DOMAIN -all -o subfinder.txt -es github \
    $([[ -n "$HTTP_PROXY" ]] && echo " -proxy $HTTP_PROXY") &>/dev/null &
```

```bash
# Summarize passive enum results
local results=(
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
        grep -E "^$DOMAIN$|\.$DOMAIN$" $result | anew -q passive.txt
        # Delete the source file
        # rm $result
    fi
done
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
```
