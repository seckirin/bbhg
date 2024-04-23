# Host 碰撞

Host 碰撞是一种常见的网络安全技术，它通过尝试多个 IP 或主机名的组合来发现存在但未公开的服务器或设备。这通常涉及到使用域名和 IP 列表进行配对，然后遍历发松请求，通过响应结果来发现隐蔽的网络资产。这种方式适合用于发现那些禁止直接通过 IP 访问，需要绑定 Host 才能正常请求访问的系统。

进行 host 碰撞测试时，需要收集和分析目标企业的相关数据，通常需要包括以下内容：

* 域名列表：包括目标**存在但没有解析记录的域名**，以及**指向内网 IP 地址的域名**。
* IP 列表：包括目标**域名 A 记录解析到的 IP 地址**，这些地址通常对应反向代理服务器。

## 获取存在但没有解析记录的域名

```bash
cat subs.txt | dnsx -json -retry 3 -rc NXDOMAIN -o hack/host-collision/noxdomain.txt
cat hack/host-collision/noxdomain.txt | jq '.host' -r | anew hack/host-collision/hostList.txt
```

```bash
# REFERENCE DOCS
https://github.com/projectdiscovery/dnsx
https://jqlang.github.io/jq/manual
```

## 获取指向内网 IP 地址的域名

内网 IP 地址通常包括以下三个范围：

* 10.0.0.0 - 10.255.255.255
* 172.16.0.0 - 172.31.255.255
* 192.168.0.0 - 192.168.255.255

为了方便处理数据，我们可以先将 dnsx 扫描结果保存为新的 json 对象。

```bash
cat scan/dnsx/subs-dnsx.json | jq '. | {host: .host, a: .a}' > hack/host-collision/subs-dnsx.json
```

之后可以使用简单的 Python 脚本，筛选出 a 对象包含内网地址对应的域名。

```bash
python3 ExtractPrivateHosts.py hack/host-collision/subs-dnsx.json hack/host-collision/privateHosts.txt
cat hack/host-collision/privateHosts.txt | anew hack/host-collision/hostList.txt
```

```bash
# REFERENCE DOCS
https://gist.github.com/Yuukisec/cfef05a435f6bfbc97fd2fab34995e34
```

## 获取域名 A 记录解析到的 IP 地址

```bash
cat scan/dnsx/subs-dnsx.json | jq '.a[]' -r > hack/host-collision/ipList.txt
```

```bash
# REFERENCE DOCS
https://jqlang.github.io/jq/manual
```

## 使用工具进行 Host 碰撞测试

```bash
# 将上面步骤中获取到的 hostList 和 ipList 复制到程序的 dataSource 目录
java -jar HostCollision.jar
```

```bash
# REFERENCE DOCS
https://github.com/pmiaowu/HostCollision
```

**已知问题**

* 2024/04/17 10:40 获取域名 A 记录解析到的 IP 地址不一定能准确的获取到目标目标的反向代理服务器，CDN 可能会导致的解析的 IP 地址错误，可能需要考虑换一种方式获取目标服务器地址。另外，除了直接通过 DNS 解析获取反向代理主机，还可用扫描 C 段或是扫描其他端口开放的 HTTP 服务器来获取反向代理主机。END

