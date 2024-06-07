# ASN / CIDR Enum

```
sort -n -t . -k 1,1 -k 2,2 -k 3,3 -k 4,4 ipaddresses.txt | icidr -sa-cidr -json > cidr.json

cat cidr.json | jq -r '.[] | select(.count>200)' > cidr_200ormore.json
cat cidr.json | jq -r '.[] | select(.count>150)' > cidr_150ormore.json
cat cidr.json | jq -r '.[] | select(.count>100)' > cidr_100ormore.json
cat cidr.json | jq -r '.[] | select(.count>50)' > cidr_50ormore.json
cat cidr.json | jq -r '.[] | select(.count>10)' > cidr_10ormore.json
```
