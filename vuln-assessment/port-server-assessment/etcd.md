# etcd

<details>

<summary>Introduction</summary>

etcd 是一个开源的分布式键值存储系统，用于可靠地存储和管理分布式系统的配置数据、元数据和状态信息。它使用 Raft 一致性算法确保数据的一致性和可靠性，并提供了简单的 HTTP API 用于读写数据。etcd 通常用于服务发现、配置管理和分布式锁等场景，其轻量级和高可用性使其成为构建云原生应用和微服务架构的重要组件之一。

</details>

<table><thead><tr><th width="178">Name</th><th>etcd</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>2379</td></tr><tr><td><strong>RCE</strong></td><td>N/A</td></tr><tr><td><strong>Vulnerability</strong></td><td>Unauthorized Access (Leak Information)</td></tr></tbody></table>

## Unauthorized Access

If the etcd service allows unauthorized access, you can use the [ETCD Manager](https://etcdmanager.io/) to connect.

{% hint style="info" %}
If there is an etcd service on a host, it is possible to scan the etcd service in the C segment, and it is very likely to discover other hosts that are using the etcd service

```
Example: nmap -p 2379 IP-CIRD -T4 -n -Pn -oG - | grep open
```
{% endhint %}
