# Domain Gathering Method

## ICP Filing

This content has been merged into: [general-technology.md](../../external-reconnaissance/general-technology.md "mention") and [based-on-company](../../external-reconnaissance/based-on-company/ "mention").

## Whois

```bash
# Reverse Whois
https://tools.whoisxmlapi.com/reverse-whois-search
```

## Google Analytics ID

```bash
# Reverse Google Analytics ID
https://intelx.io/tools?tab=analytics
https://spyonweb.com/
```

## Favicon

```bash
# Evaluate Favicon Hash
ifavicon -url https://example.com/favicon.ico

# Gathering Domain via Favicon Hash
# # suspense
```

## Other Technologies

```bash
# Gathering Domain via Certificate
cert.subject.org="target" && is_domain=true

# Gathering Domain via HTTP Header
host="target.com" && header="Content-Security-Policy-Report-Only"
host="target.com" && header="Content-Security-Policy"
host="target.com" && header="Set-Cookie"

# Gathering Domain via In-house Name Server
https://hackertarget.com/find-shared-dns-servers/
```
