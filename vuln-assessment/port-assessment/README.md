# Port Assessment

## [<mark style="color:red;">22  - SSH</mark>](22-ssh.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/Secure\_Shell](https://en.wikipedia.org/wiki/Secure\_Shell)
* **Potential Risks**
  * [Brute Force](22-ssh.md#brute-force)

## [<mark style="color:red;">389, 636 - LDAP</mark>](389-636-ldap.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/Lightweight\_Directory\_Access\_Protocol](https://en.wikipedia.org/wiki/Lightweight\_Directory\_Access\_Protocol)
* **Potential Risks**
  * [Unauthenticated Access](389-636-ldap.md#unauthenticated-access)
* **Exploitation**
  * [Information Disclosure](389-636-ldap.md#information-disclosure)

## [<mark style="color:red;">873 - Rsync</mark>](873-rsync.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/Rsync](https://en.wikipedia.org/wiki/Rsync)
* **Potential Risks**
  * [Unauthorized Access](873-rsync.md#unauthorized-access)
* **Exploitation**
  * **Upload File**
    * [Method 1: Regular upload files](873-rsync.md#method-1-regular-upload-files)
    * [Method 2: Upload crontab to Get Shell](873-rsync.md#method-2-upload-crontab-to-get-shell)
    * [Method 3: Upload the executable file and add crontab to execute](873-rsync.md#method-3-upload-the-executable-file-and-add-crontab-to-execute)
  * [Download File](873-rsync.md#download-file)

## 1433 - MSSQL

* **Wikipedia**: [https://en.wikipedia.org/wiki/Microsoft\_SQL\_Server](https://en.wikipedia.org/wiki/Microsoft\_SQL\_Server)
* **Exploitation**
  * [CLI Command Execution](1433-mssql.md#cli-command-execution)

## [<mark style="color:red;">1883 - MQTT</mark>](1833-mqtt.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/MQTT](https://en.wikipedia.org/wiki/MQTT)
* **Potential Risks**
  * [Unauthenticated Access](1833-mqtt.md#unauthenticated-access)
  * [Brute Force](1833-mqtt.md#brute-force)
* **Exploitation**
  * [Information Disclosure](1833-mqtt.md#information-disclosure)

## [<mark style="color:red;">2375 - Docker</mark>](2375-docker.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/Docker](https://en.wikipedia.org/wiki/Docker)
* **Potential Risks**
  * [API Unauthenticated Access](2375-docker.md#api-unauthenticated-access)
* **Exploitation**
  * [Container Escape](2375-docker.md#container-escape)

## [<mark style="color:red;">2379 - etcd</mark>](etcd.md)

* **Wikipedia**: [https://zh.wikipedia.org/wiki/Kubernetes#etcd](https://zh.wikipedia.org/wiki/Kubernetes#etcd)
* **Potential Risks**
  * Unauthenticated Access
* **Exploitation**
  * Information Disclosure

## [<mark style="color:red;">3306 - MySQL</mark>](3306-mysql.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/MySQL](https://en.wikipedia.org/wiki/MySQL)
* **Exploitation**
  * **MySQL Command Line Interface**
    * [CLI Write File](3306-mysql.md#cli-write-file)
    * [CLI Load File](3306-mysql.md#cli-load-file)
    * [CLI Command Execution](3306-mysql.md#cli-command-execution)
  * **MySQL SQL Injection**
    * [sqlmap Write File](3306-mysql.md#sqlmap-write-file)
    * [sqlmap Get Shell](3306-mysql.md#sqlmap-get-shell)

## [<mark style="color:red;">6379 - Redis</mark>](6379-redis.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/Redis](https://en.wikipedia.org/wiki/Redis)
* **Potential Risks**
  * [Unauthenticated Access](6379-redis.md#unauthorized-access)
* **Exploitation**
  * [Write File](6379-redis.md#write-file)
  * [GUI Tools](6379-redis.md#gui-tools)

## [<mark style="color:red;">8086 - InfluxDB</mark>](8086-influxdb.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/InfluxDB](https://en.wikipedia.org/wiki/InfluxDB)
* **Potential Risks**
  * [Unauthorized Access](8086-influxdb.md#cve-2019-20933-unauthorized-access)
  * [CVE-2019-20933 (Unauthorized Access)](8086-influxdb.md#cve-2019-20933-unauthorized-access)
* **Exploitation**
  * [Information Disclosure](8086-influxdb.md#information-disclosure)

## [<mark style="color:red;">27017 - MongoDB</mark>](27017-mongodb.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/MongoDB](https://en.wikipedia.org/wiki/MongoDB)
* **Potential Risks**
  * [Unauthenticated Access](27017-mongodb.md#unauthorized-access)
* **Exploitation**
  * [Information Disclosure](27017-mongodb.md#information-disclosure)

## OverView

<table><thead><tr><th>Default Port / Server</th><th data-type="checkbox">RCE</th></tr></thead><tbody><tr><td><a href="etcd.md">2379 - etcd</a></td><td>false</td></tr></tbody></table>

