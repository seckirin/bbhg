# Port Assessment

## [<mark style="color:red;">22  - SSH</mark>](22-ssh.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/Secure\_Shell](https://en.wikipedia.org/wiki/Secure\_Shell)
* **Potential Risks**
  * [Brute Force](22-ssh.md#brute-force)

## [<mark style="color:red;">389 - LDAP</mark>](389-ldap.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/Lightweight\_Directory\_Access\_Protocol](https://en.wikipedia.org/wiki/Lightweight\_Directory\_Access\_Protocol)
* **Potential Risks**
  * [Unauthenticated Access](389-ldap.md#unauthenticated-access)
* **Exploiting**
  * [Information Disclosure](389-ldap.md#information-disclosure)

## [<mark style="color:red;">1883 - MQTT</mark>](1833-mqtt.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/MQTT](https://en.wikipedia.org/wiki/MQTT)
* **Potential Risks**
  * [Unauthenticated Access](1833-mqtt.md#unauthenticated-access)
  * [Brute Force](1833-mqtt.md#brute-force)
* **Exploiting**
  * [Information Disclosure](1833-mqtt.md#information-disclosure)

## [<mark style="color:red;">3306 - MySQL</mark>](3306-mysql.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/MySQL](https://en.wikipedia.org/wiki/MySQL)
* **Exploiting**
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
  * Unauthenticated Access
* **Exploiting**
  * Write File
  * Get Shell

## [<mark style="color:red;">8086 - InfluxDB</mark>](8086-influxdb.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/InfluxDB](https://en.wikipedia.org/wiki/InfluxDB)
* **Potential Risks**
  * [CVE-2019-20933 (Unauthorized Access)](8086-influxdb.md#cve-2019-20933-unauthorized-access)
* **Exploiting**
  * [Information Disclosure](8086-influxdb.md#information-disclosure)

## [<mark style="color:red;">27017 - MongoDB</mark>](27017-mongodb.md)

* **Wikipedia**: [https://en.wikipedia.org/wiki/MongoDB](https://en.wikipedia.org/wiki/MongoDB)
* **Potential Risks**
  * [Unauthenticated Access](27017-mongodb.md#unauthorized-access)
* **Exploiting**
  * [Information Disclosure](27017-mongodb.md#information-disclosure)

## OverView

<table><thead><tr><th>Default Port / Server</th><th data-type="checkbox">RCE</th></tr></thead><tbody><tr><td><a href="873-rsync.md">873 - Rsync</a></td><td>true</td></tr><tr><td><a href="2375-docker.md">2375 - Docker</a></td><td>true</td></tr><tr><td><a href="etcd.md">2379 - etcd</a></td><td>false</td></tr><tr><td>1433 - MSSQL</td><td>true</td></tr></tbody></table>

