# 389 - LDAP

## Potential Risks

### Unauthenticated Access

```bash
# Bulk Detecting
nuclei -t <BaseUrl> \
     -t ~/nuclei-templates/http/misconfiguration/unauth-ldap-account-manager.yaml

nuclei -l url_lists.txt \
    -t ~/nuclei-templates/http/misconfiguration/unauth-ldap-account-manager.yaml
```

## Exploiting

### Information Disclosure

```bash
# LDAP Admin Tool
https://www.ldapsoft.com/download.html
```

<figure><img src="../../.gitbook/assets/20240425_222357.png" alt=""><figcaption></figcaption></figure>
