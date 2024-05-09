# Regular

## Passive Enum

Use [amass (v3)](https://github.com/owasp-amass/amass/tree/v3.23.3), [bbot](https://github.com/blacklanternsecurity/bbot), [crt](https://github.com/cemulus/crt), [github-subdomain](https://github.com/gwen001/github-subdomains), [gitlab-subdomain](https://github.com/gwen001/gitlab-subdomains) and [subfinder](https://github.com/projectdiscovery/subfinder).

```bash
# https://github.com/owasp-amass/amass/tree/v3.23.3
amass enum -passive -d vulnweb.com -silent -timeout 30 -o amass.txt

# https://github.com/blacklanternsecurity/bbot
bbot -t vulnweb.com -f subdomain-enum -rf passive -em massdns -y -s -om json |
    jq -r 'select(.scope_distance==0) | select(.type=="DNS_NAME") | .data' >bbot.txt

# https://github.com/cemulus/crt
crt -s -l 999999 -json vulnweb.com |
    jq -r '.[].subdomain' | sed -e 's/^\*\.//' > crt.txt

# https://github.com/gwen001/github-subdomains
github-subdomains -d vulnweb.com -t $GITHUB_TOKENS \
    -k -q -o github-subdomains.txt &>/dev/null

# https://github.com/gwen001/gitlab-subdomains
gitlab-subdomains -d vulnweb.com -t $(cat $GITLAB_TOKENS) &>/dev/null &&
    mv vulnweb.com.txt gitlab-subdomains.txt

# https://github.com/projectdiscovery/subfinder
subfinder -d vulnweb.com -all -o subfinder.txt &>/dev/null
```

### Summarize passive enum results

```bash
local domain=vulnweb.com
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
        # Extract results in scope | Unique line to passive.txt && Delete the source file
        grep -E "^$domain$|\.$domain$" <$result | anew -q passive.txt && rm $result
    fi
done
```

## Active Resolve

```bash
```

## Brute Force
