# Draft notes

```markdown
# 批量识别 Web 服务高危组件的工具和方法

- 使用如 Banli 等现有的工具实现批量识别高危Web服务组件
- 对现有工具进行修改以提高兼容性与准确性

# 渗透测试中 Shiro 组件的识别方法

- 检查响应数据包中是否包含 rememberMe=deleteMe 字段
- 检查响应数据包中是否包含 deleteMe 字段
- 类似的方法也可应用于其他组件的识别

# WebLogic 组件的漏洞测试

- 通过网络搜索 WebLogic 漏洞合集
- 使用工具或手动测试 WebLogic 组件是否存在漏洞

# Spring 组件 heapdump 敏感信息泄露处理方法

- 发现 Spring 组件的 heapdump 泄露时，可能泄露 AK/SK、shiro 密钥等敏感信息
- 使用 heapdump_tool 工具发现敏感信息
- 对于Shiro组件，全局搜索 org.apache.shiro.web.mgt.CookieRememberMeManager 以获取密钥信息
- 搜索关键字如 oss、腾讯云存储 SK、阿里云存储 AK 等关键字，进一步发现云服务密钥泄露
- 当工具难以发现敏感信息时，使用 VisualVM 进行图形化搜索，注意手动 base64 编码发现的 Shiro 密钥

# Spring 组件接口未授权访问处理方法

- 发现Spring组件接口存在未授权访问时，可能泄露Redis等敏感信息
- 若信息加密，可尝试 SpringBootVulnExploit 提到的测试手法

# Spring 组件 CVE-2022-22947 漏洞测试步骤

- 参考 vulhub 的相关文档中所指示的步骤进行测试。
```

```markdown
# Linux 计时任务文件和目录的区别和作用

/etc/cron.d/FILENAME < * * * * * USERNAME COMMAND
- 全局 crontab，存放所有计时任务
- 每一个文件都是单独的计划任务
- 文件可以是任意名称
- 需要指定运行任务的用户名称

/etc/crontab < * * * * * USERNAME COMMAND
- 全局 crontab，可以被所有用户读取，但只有 root 用户可以编辑
- 需要指定运行任务的用户名称

/var/spoof/cron/USERNAME < * * * * * COMMAND
- 用户级 crontab
- 文件名称需要指定为运行任务的用户名称
- 只有相应的用户和 root 用户可以编辑和读取
- 不需要指定运行任务的用户名称
```
