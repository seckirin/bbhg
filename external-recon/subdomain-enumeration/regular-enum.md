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

## Regular Resolve

```bash
cat active.txt brute.txt | sort -u | puredns resolve \
    --rate-limit 3000 \
    --rate-limit-trusted 150 \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    --write regular.txt \
    &>/dev/null
```

## Summarizing

```python
# summarizing.py
import json
import os
from datetime import datetime

# Define source types
sources = ["passive", "active", "brute", "regular"]

# Initialize a dictionary to store subdomain information
subdomains = {}

# Check if the subdomains.json file exists
if os.path.exists("subdomains.json"):
    # If it exists, load the existing data
    with open("subdomains.json", "r") as file:
        existing_data = json.load(file)
        for item in existing_data:
            subdomains[item["subdomain"]] = item

# Read all subdomains and types
for source in sources:
    with open(f"{source}.txt", "r") as file:
        for line in file:
            subdomain = line.strip()
            if subdomain not in subdomains:
                # If the subdomain is new, create a new dictionary object
                subdomains[subdomain] = {
                    "subdomain": subdomain,
                    "resolved": 0,
                    "sources": [],
                    "created_at": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                    "updated_at": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                }
            else:
                # Update the updated_at field
                subdomains[subdomain]["updated_at"] = datetime.now().strftime(
                    "%Y-%m-%d %H:%M:%S"
                )

            # Add the source to the sources list
            if source not in subdomains[subdomain]["sources"]:
                subdomains[subdomain]["sources"].append(source)
                # Update the updated_at field if sources is updated
                subdomains[subdomain]["updated_at"] = datetime.now().strftime(
                    "%Y-%m-%d %H:%M:%S"
                )

            # If the source is regular, set resolved to 1
            if source == "regular" and subdomains[subdomain]["resolved"] != 1:
                subdomains[subdomain]["resolved"] = 1
                # Update the updated_at field if resolved is updated
                subdomains[subdomain]["updated_at"] = datetime.now().strftime(
                    "%Y-%m-%d %H:%M:%S"
                )

# Write the result to the JSON file
with open("subdomains.json", "w") as file:
    json.dump(list(subdomains.values()), file, indent=4)

```
