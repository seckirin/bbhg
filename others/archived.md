---
description: 一些过时或是不完全可靠的信息
---

# Archived

```
START 2024/04/22 08:37 SOURCE: 信息收集 > 子域名枚举 > 获取 DNS 记录
# 筛选 CDN 获取 IP 地址 (不稳定)
# https://github.com/projectdiscovery/cdncheck
grep -v -f <(cat scan/dnsx-resolved.json | jq -r '.a[]' | cdncheck -silent) \
    <(cat scan/dnsx-resolved.json | jq -r '.a[]') | sort -u > ips.txt
END
```
