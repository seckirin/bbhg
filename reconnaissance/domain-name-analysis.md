# Domain Analysis

[Contents](../readme/table-of-contents.md) > [Based on Domain](broken-reference) > [**Domain Name Analysis**](domain-name-analysis.md)

```sh
# Multi-purpose website
https://dnsdblookup.com/
https://tool.chinaz.com/

# Domain >> registrant information (whois)
https://www.whois.com/whois/
https://who.is/
https://www.ip138.com/

# Domain >> ICP licenses and company names
https://beian.miit.gov.cn/ # Official
https://icp.chinaz.com/bytedance.com
https://site.ip138.com/bytedance.com/beian.htm
https://www.beianx.cn/search/bytedance.com
# Bulk query
https://www.link113.com/icp/
https://uutool.cn/icp-batch/
# https://github.com/HG-ha/ICP_Query
RHOST=ICP_QueryServerHost
dtp() {
    while read domain; do
	    while true; do
	        result=$(curl -s "http://$RHOST:16181/query/web?search=$domain")
	        if [ $(echo ${result} | jq -r '.code') -eq 200 ]; then
		        echo $result | jq -r 'try .params.list' -c
		        break
			fi
			echo ${result} | jq -r '.'
			sleep 5
		done
    done
}

# Domain >> IP resolution history
https://viewdns.info/iphistory/?domain=bytedance.com
https://ipchaxun.com/bytedance.com/

# Domain >> IP address
https://www.nslookup.io/domains/bytedance.com/webservers/
```

## ICP License Analysis

```bash
# ICP >> company name and registration time
https://icp.chinaz.com/粤B2-20090059-1
https://www.tianyancha.com/search?key=粤B2-20090059-1
https://www.beianx.cn/search/粤B2-20090059-1
```

## IP Address Analysis

```bash
# IP >> registrant information (whois)
https://www.whois.com/whois/8.8.8.8
https://who.is/whois-ip/ip-address/8.8.8.8

# IP >> location
https://tool.lu/ip/

# IP >> domain name
https://reverseip.domaintools.com/search/?q=122.14.229.7
https://dnsdblookup.com/122.14.229.7/
https://tool.chinaz.com/same/122.14.229.7
```
