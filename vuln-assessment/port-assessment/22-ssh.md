# 22 - SSH

## Potential Risks

### Brute Force

```bash
# https://github.com/evilsocket/legba
legba ssh --target localhost:22 \
    --username usernames.txt \
    --password passwords.txt \
    --output-format jsonl --output ssh_brute.json

# Bulk testing
legba ssh --target @hosts.txt \
    --username usernames.txt \
    --password passwords.txt \
    --output-format jsonl --output ssh_brute.json
```
