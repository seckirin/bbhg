# 27017 - MongoDB

## Potential Risks

### Unauthorized Access

```bash
# https://github.com/yuukisec/iPoCs
nuclei -t ~/ipocs -id mongodb-unauth -u localhost:27017

# Bulk testing
nuclei -t ~/ipocs -id mongodb-unauth -l hosts.txt -c 100 -bs 100
```

## Exploiting

### Information Disclosure

```bash
# Navicate Premium
https://www.navicat.com/en/products/navicat-premium
```
