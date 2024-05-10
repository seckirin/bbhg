# Summarizing

```python
# summarizing.py
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
