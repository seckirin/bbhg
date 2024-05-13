# Linux

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
