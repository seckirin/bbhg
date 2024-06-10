# Versatile Resources

## Certificate

```bash
# Websites
https://developers.facebook.com/tools/ct/
https://crt.sh/
```

## Google Analytics ID

```bash
# Reverse
https://builtwith.com/relationships/tag/UA-33427076
https://api.hackertarget.com/analyticslookup/?q=UA-33427076
https://publicwww.com/websites/ua-33427076/
http://site-overview.com/website-report-search/analytics-account-id/33427076
https://spyonweb.com/UA-33427076
https://analyzeid.com/id/UA-33427076
https://osint.sh/analytics
```

## ASN / CIDR

```bash
# Websites
https://bgp.he.net/
https://bgp.tools/
https://bgpview.io/
https://asnlookup.com/
https://ipwhois.cnnic.net.cn/

# https://github.com/dhn/spk
spk -s <keyword> -silent -json

# https://github.com/projectdiscovery/asnmap
asnmap [-org <keyword>] [-asn <asn_number>] [-ip <ip_address>] [-domain <domain>] \
    -json -verbose | jq -r '.'
```

## Cyberspace Mapping

```bash
# https://0.zone/
(type=微信小程序&&company==<keyword>)
(type=微信小程序&&company=<company_name>)
(type=安卓APK&&company==<keyword>)
(type=安卓APK&&company=<company_name>)

# https://fofa.info/
# CIDR & Realted domains
cert.subject.org="<company_keyword>" && is_domain=true && cert.is_valid=true
cert.subject.cn="<domain>" && is_domain=true && cert.is_valid=true
# Related Domain
domain="<domain.com>" && header="Content-Security-Policy-Report-Only"
domain="<domain.com>" && header="Content-Security-Policy"
domain="<domain.com>" && header="Set-Cookie"

# https://quake.360.net/
cert:"<domain.com>" AND NOT domain:<domain.com> AND is_domain:true

# https://www.shodan.io/
http.favicon.hash:<favicon_hash>
```
