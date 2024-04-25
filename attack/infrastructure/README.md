# Infrastructure

## Table of Contents

* [Database](database/)
* [Framework](framework.md)
* [Protocol](protocol/)
* [Utility](utility/)
* [Virtualization](virtualization/)

## Default Port Overview

<table><thead><tr><th width="157">Name</th><th width="128">Default Port</th><th width="350">Type &#x26; Main application areas</th><th data-type="checkbox">GetShell?</th></tr></thead><tbody><tr><td><a href="protocol/ssh.md">SSH</a></td><td>22</td><td><a href="protocol/">Protocol</a></td><td>true</td></tr><tr><td><a href="protocol/ldap.md">LDAP</a></td><td>389</td><td><a href="protocol/">Protocol</a></td><td>false</td></tr><tr><td><a href="utility/rsync.md">Rsync</a></td><td>873</td><td><a href="utility/">Utility</a> / Distributed</td><td>true</td></tr><tr><td><a href="database/mssql.md">MSSQL</a></td><td>1433</td><td><a href="database/">Database</a></td><td>true</td></tr><tr><td><a href="protocol/mqtt.md">MQTT</a></td><td>1833</td><td><a href="protocol/">Protocol</a> / IoT</td><td>false</td></tr><tr><td><a href="virtualization/docker.md">Docker</a></td><td>2375</td><td><a href="virtualization/">Virtualization</a> / Distributed</td><td>true</td></tr><tr><td><a href="database/etcd.md">etcd</a></td><td>2379</td><td><a href="database/">Database</a> / Distributed</td><td>false</td></tr><tr><td><a href="database/mysql.md">MySQL</a></td><td>3306</td><td><a href="database/">Database</a></td><td>true</td></tr><tr><td><a href="database/redis.md">Redis</a></td><td>6379</td><td><a href="database/">Database</a> / Distributed</td><td>true</td></tr><tr><td><a href="database/influxdb.md">InfluxDB</a></td><td>8086</td><td><a href="database/">Database</a></td><td>false</td></tr><tr><td><a href="database/mongodb.md">MongoDB</a></td><td>27017</td><td><a href="database/">Database</a></td><td>false</td></tr></tbody></table>

## Infrastructure Summary Template

<table><thead><tr><th width="178">Name</th><th>SSH (Secure Shell Protocol)</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>22</td></tr><tr><td><strong>GetShell?</strong></td><td>Yes</td></tr><tr><td><strong>Vulnerability</strong></td><td>Brute Force</td></tr></tbody></table>
