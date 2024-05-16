# MongoDB

<details>

<summary>Introduction</summary>

MongoDB 是一个开源的面向文档的 NoSQL 数据库，采用了 JSON-like 的 BSON 格式来存储数据。它提供了灵活的数据模型，可以存储各种类型和格式的数据，而不需要预先定义表结构。MongoDB 支持丰富的查询操作、索引和聚合功能，同时具有高性能、可扩展性和高可用性。它被广泛应用于 Web 应用程序、大数据分析、内容管理系统、实时分析和物联网等领域，特别适用于需要处理大量半结构化或非结构化数据的场景。MongoDB 的特点包括横向扩展、自动分片、副本集和地理分布式存储等，使其成为现代应用开发的重要组件之一。

</details>

<table><thead><tr><th width="178">Name</th><th>MongoDB</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>27017</td></tr><tr><td><strong>RCE</strong></td><td>Feasible</td></tr><tr><td><strong>Vulnerability</strong></td><td>Unauthorized Access (Leak Information)</td></tr></tbody></table>

## Unauthorized Access

If the MongoDB service allows unauthorized access, you can use the [Navicate Premium](https://www.navicat.com/en/products/navicat-premium) to connect.

### Batch Testing

{% code title="Use Nuclei" %}
```bash
nuclei -t ~/nuclei-templates/network/misconfig/mongodb-unauth.yaml -l hosts.txt
```
{% endcode %}

{% code title="Reference Docs" %}
```
https://github.com/projectdiscovery/nuclei-templates/blob/main/network/misconfig/mongodb-unauth.yaml
```
{% endcode %}
