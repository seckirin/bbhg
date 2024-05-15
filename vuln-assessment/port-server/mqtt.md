# MQTT

<details>

<summary>Introduction</summary>

MQTT（Message Queuing Telemetry Transport）是一种轻量级、灵活的消息传输协议，专门设计用于物联网（IoT）和机器到机器（M2M）通信。它采用发布-订阅模式，允许设备和应用程序以低带宽和低能耗的方式进行实时通信。MQTT 具有简单的客户端-服务器架构，支持基于主题的消息发布和订阅，实现了高效的消息传递和可靠性。由于其轻量级和可扩展性，MQTT 被广泛应用于传感器网络、智能家居、工业自动化和物联网解决方案中。

</details>

<table><thead><tr><th width="178">Name</th><th>MQTT (Message Queuing Telemetry Transport)</th></tr></thead><tbody><tr><td><strong>Default Port</strong></td><td>1883</td></tr><tr><td><strong>RCE</strong></td><td>N/A</td></tr><tr><td><strong>Vulnerability</strong></td><td><ul><li>Unauthorized Access (Leak Information)</li><li>Brute Force</li></ul></td></tr></tbody></table>

## Unauthorized Access

If the MQTT service allows unauthorized access, you can use the [MQTT Explorer](https://github.com/thomasnordquist/MQTT-Explorer) to connect.

## Brute Force

If the MQTT service has been authenticated, you can use tools like [legba](https://github.com/evilsocket/legba) and [mqtt-pwn](https://github.com/akamai-threat-research/mqtt-pwn) for brute force.

{% code title="Environment" %}
```bash
HOST=; PORT=1883; # TARGET HOST INFO
DICT_PATH=~/hack/dicts/hack-dict; # LOCAL DICT PATH
```
{% endcode %}

{% code title="Use legba" %}
```bash
docker run \
  -v $DICT_PATH:/hack-dict \
  -v $(PWD):/out \
  --network host -it \
  evilsocket/legba:latest \
  mqtt --target $HOST:$PORT \
  --username /hack-dict/username/top10.txt \
  --password /hack-dict/password/darkweb2017-top100.txt \
  -O /out/legba-mqtt-outptu.txt
```
{% endcode %}
