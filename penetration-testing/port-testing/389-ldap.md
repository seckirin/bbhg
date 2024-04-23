# 389 LDAP

## Rsync 未授权访问

如果 LDAP 服务存在未授权访问，可以直接使用 [LDAP Admin Tool](https://www.ldapsoft.com/download.html) 等工具进行连接。

### 批量检测 Rsync 未授权访问

```bash
nuclei -t ~/nuclei-templates/http/misconfiguration/unauth-ldap-account-manager.yaml -l ip.txt
```

* [\~/nuclei-templates/http/misconfiguration/unauth-ldap-account-manager.yaml](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/misconfiguration/unauth-ldap-account-manager.yaml)
