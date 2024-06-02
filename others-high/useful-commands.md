# Useful Commands

根据 IP 地址进行排序，而不是单个数字

```bash
sort -n -t . -k 1,1 -k 2,2 -k 3,3 -k 4,4 ips.txt
```

筛选文件内容中以 `:` 作为分隔符，第一个位置的值

```bash
awk -F ':' '{print $1}' ports.txt

cut -d ':' -f 1 ports.txt
```
