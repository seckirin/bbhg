# Preparations

## Variables

```bash
# General
DOMAIN="vulnweb.com"
TOOLS="$HOME/hack/tools"
HTTP_PROXY="http://127.0.0.1:6152"
ALL_PROXY="socks5://127.0.0.1:6153"

# Resolvers
RESOLVERS="$HOME/.config/puredns/resolvers.txt"
RESOLVERS_URL="https://public-dns.info/nameservers.txt"
RESOLVERS_TRUSTED="$HOME/.config/puredns/resolvers-trusted.txt"
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

## Folders

```bash
# Check all the folders needed
local folders=(
    "${TOOLS}"
    "${TOOLS}/wordlists"
    "${TOOLS}/resolvers"
    "${HOME}/.config/amass/"
    "${HOME}/.config/subfinder/"
    "${HOME}/.config/puredns/"
)

for folder in "${folders[@]}"; do
    mkdir -p "$folder"
done
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

## summarizer.py

```python
import json
import os
import argparse
from datetime import datetime


class SubdomainSummarizer:
    def __init__(self, sources, resolved, delete):
        self.sources = sources.split(",")
        self.resolved = resolved
        self.delete = delete
        self.subdomains = self.load_existing_data()

    @staticmethod
    def load_existing_data():
        if os.path.exists("subdomains.json"):
            with open("subdomains.json", "r") as file:
                existing_data = json.load(file)
                return {item["subdomain"]: item for item in existing_data}
        else:
            return {}

    def process_sources(self):
        for source in self.sources:
            with open(f"{source}.txt", "r") as file:
                for line in file:
                    self.process_subdomain(source, line.strip())

    def process_subdomain(self, source, subdomain):
        if subdomain not in self.subdomains:
            self.subdomains[subdomain] = {
                "subdomain": subdomain,
                "resolved": self.resolved,
                "sources": [],
                "created_at": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                "updated_at": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            }
        else:
            # Update the resolved and updated_at fields
            self.subdomains[subdomain]["resolved"] = self.resolved
            self.subdomains[subdomain]["updated_at"] = datetime.now().strftime(
                "%Y-%m-%d %H:%M:%S"
            )

        # Add the source to the sources list
        if source not in self.subdomains[subdomain]["sources"]:
            self.subdomains[subdomain]["sources"].append(source)
            # Update the updated_at field if sources is updated
            self.subdomains[subdomain]["updated_at"] = datetime.now().strftime(
                "%Y-%m-%d %H:%M:%S"
            )

    def delete_sources(self):
        if self.delete:
            for subdomain in self.subdomains.values():
                for source in self.sources:
                    if source in subdomain["sources"]:
                        subdomain["sources"].remove(source)

    def remove_empty_sources(self):
        self.subdomains = {k: v for k, v in self.subdomains.items() if v["sources"]}

    def write_to_file(self):
        with open("subdomains.json", "w") as file:
            json.dump(list(self.subdomains.values()), file, indent=4)

    def run(self):
        self.process_sources()
        self.delete_sources()
        self.remove_empty_sources()
        self.write_to_file()


def main():
    parser = argparse.ArgumentParser(description="Process some integers.")
    parser.add_argument(
        "-s",
        "--sources",
        type=str,
        required=True,
        help="The sources to be processed, separated by commas",
    )
    parser.add_argument(
        "-r",
        "--resolved",
        type=int,
        required=True,
        choices=[0, 1],
        help="The resolved status",
    )
    parser.add_argument(
        "-d",
        "--delete",
        action="store_true",
        help="Delete the specified sources from the JSON file",
    )

    args = parser.parse_args()

    summarizer = SubdomainSummarizer(args.sources, args.resolved, args.delete)
    summarizer.run()


if __name__ == "__main__":
    main()

```
