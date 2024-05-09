# Preparations

## Folders

```bash
# Check all the folders needed
local folders=(
    "${TOOLS}"
    "${TOOLS}/wordlists"
    "${TOOLS}/resolvers"
    "${HOME}/.config/amass/"
    "${HOME}/.config/subfinder/"
)

for folder in "${folders[@]}"; do
    mkdir -p "$folder"
done
```

## Variables

```bash
# General
DOMAIN="vulnweb.com"
TOOLS="$HOME/hack/tools"

# Resolvers
RESOLVERS="$TOOLS/resolvers/resolvers.txt"
RESOLVERS_URL="https://public-dns.info/nameservers.txt"
RESOLVERS_TRUSTED="$TOOLS/resolvers/resolvers_trusted.txt"
RESOLVERS_TRUSTED_URL="https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt"

# Wordlists
SUBDOMAINS="${TOOLS}/wordlists/subdomains.txt"
SUBDOMAINS_URL="https://raw.githubusercontent.com/yuukisec/ReconLists/main/subdomains/subdomains.txt"
SUBDOMAINS_HUGE="${TOOLS}/wordlists/subdomains_huge.txt"
SUBDOMAINS_HUGE_URL="https://raw.githubusercontent.com/yuukisec/ReconLists/main/subdomains/subdomains_huge.txt"
PERMUTATIONS="${TOOLS}/wordlists/permutations.txt"
PERMUTATIONS_URL="https://raw.githubusercontent.com/yuukisec/ReconLists/main/subdomains/permutations.txt"

# Config File
AMASS_CONFIG="$HOME/.config/amass/config.ini"
BBOT_CONFIG="${HOME}/.config/bbot/secrets.yml"
GITHUB_TOKENS="${TOOLS}/github_tokens.txt"
GITLAB_TOKENS="${TOOLS}/gitlab_tokens.txt"
SUBFINDER_CONFIG="${HOME}/.config/subfinder/provider-config.yaml"

# Tools Variables
AMASS_TIMEOUT=30 # Minutes
```

## Wordlists

```bash
wget -q -O - "$SUBDOMAINS_URL" >"$SUBDOMAINS"
wget -q -O - "$SUBDOMAINS_HUGE_URL" >"$SUBDOMAINS_HUGE"
wget -q -O - "$PERMUTATIONS_URL" >"$PERMUTATIONS"
```

## Resolvers

```bash
iresolver \
    -threads 2000 \
    -count 20000
    -target "$RESOLVERS_URL" \
    -outptu "$RESOLVERS" \
    &>/dev/null

iresolver \
    -threads 2000 \
    -target "$RESOLVERS_TRUSTED_URL" \
    -outptu "$RESOLVERS_TRUSTED" \
    &>/dev/null
```
