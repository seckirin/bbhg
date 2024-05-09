# Subdomain Enumeration

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
