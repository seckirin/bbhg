# 网络扫描

## 主机发现

```bash
# fping
fping -aqg 192.168.7.0/24

# bash
for i in {1..254} ;do (ping -c 1 192.168.7.$i | grep "from" &) ;done
```
