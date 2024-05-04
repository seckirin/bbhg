# Public Info Gathering

## ASNs

{% code title="Multipurpose Websites" %}
```bash
https://asnlookup.com/
https://search.dnslytics.com/
https://bgp.he.net/
```
{% endcode %}

{% code title="Mainland China ASN" %}
```
https://bgp.he.net/country/CN
https://search.dnslytics.com/bgp/cn
https://www.pdflibr.com/countries/cn/1
```
{% endcode %}

## Email

```bash
# https://github.com/Josue87/EmailFinder
emailfinder -d $DOMAIN > emailfinder.txt \
    2>emailfinder.log

# https://github.com/JoelGMSec/LeakSearch
python3 LeakSearch.py -k $DOMAIN -o leaksearch.txt
```
