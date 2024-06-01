# 27017 - MongoDB

## Potential Risks

### Unauthorized Access

```bash
# https://github.com/yuukisec/iPoCs
# RHOST=remoteHost; RPORT=27017
nuclei -t ~/ipocs -id mongodb-unauth -u $RHOST:$RPORT

# Bulk testing
# RHOST_LIST=hosts.txt
nuclei -t ~/ipocs -id mongodb-unauth -l $RHOST_LIST -c 100 -bs 100
```

## **Exploitation**

### Information Disclosure

```bash
# Navicate Premium
https://www.navicat.com/en/products/navicat-premium
```
