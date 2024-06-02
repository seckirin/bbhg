# Subdomain Takeover Testing

子域名接管指的是攻击者通过接管已弃用但仍指向有效内容的子域名，从而获取对该子域名的控制权。

当一个子域名被指向某些服务，例如云服务或者网站托管服务，而这些服务后来被取消或者删除，但子域名的 DNS 记录没有被相应地更新，那么这个子域名就处于一种可以被“接管”的状态。攻击者可以利用这个漏洞，重新注册注册子域名，然后他们就能控制这个子域名，甚至可以发布恶意内容，进行钓鱼攻击等。

{% embed url="https://github.com/EdOverflow/can-i-take-over-xyz" %}

```bash
python3 sub404.py -f ~/bytedance/fanqienovel/information-gathering/subs.txt
```

```bash
# REFERENCE DOCS
https://github.com/r3curs1v3-pr0xy/sub404
```
