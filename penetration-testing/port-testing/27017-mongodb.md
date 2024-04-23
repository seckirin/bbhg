# 27017 MongoDB

## 未授权访问

如果 MongoDB 服务存在未授权访问，可以直接使用 Navicate Premium 等工具进行连接。

### 批量检测未授权访问

```bash
nuclei -t ~/nuclei-templates/network/misconfig/mongodb-unauth.yaml -l hosts.txt
```

* [\~/nuclei-templates/network/misconfig/mongodb-unauth.yaml](https://github.com/projectdiscovery/nuclei-templates/blob/main/network/misconfig/mongodb-unauth.yaml)
