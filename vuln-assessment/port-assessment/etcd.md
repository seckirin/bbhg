# 2379 - etcd

## Potential Risks

### Unauthorized Access

```bash
# https://github.com/yuukisec/iPoCs
# RHOST=remoteHost; RPORT=2379
nuclei -t ~/ipocs -id etcd-unauth -u $RHOST:$RPORT

# Bulk testing
```

## Exploitation

### Information Disclosure

* ETCD Manager. <[https://etcdmanager.io/](https://etcdmanager.io/)>

{% hint style="info" %}
If there is an etcd service on a host, it is possible to scan the etcd service in the C segment, and it is very likely to discover other hosts that are using the etcd service

```
Example: nmap -p 2379 <IP-CIRD> -T4 -n -Pn -oG - | grep open
```
{% endhint %}
