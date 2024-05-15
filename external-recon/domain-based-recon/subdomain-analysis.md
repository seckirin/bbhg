# Website Probing

## Alive Website Probing

<pre class="language-bash"><code class="lang-bash">cat subdomains.json | jq -r '.[] | select(.resolved==1) | .subdomain' >subdomains.txt

WEB_UNCOMMON_PORTS="81,300,591,593,832,981,1010,1311,1099,2082,2095,2096,2480,3000,3001,3002,3003,3128,3333,4243,4567,4711,4712,4993,5000,5104,5108,5280,5281,5601,5800,6543,7000,7001,7396,7474,8000,8001,8008,8014,8042,8060,8069,8080,8081,8083,8088,8090,8091,8095,8118,8123,8172,8181,8222,8243,8280,8281,8333,8337,8443,8500,8834,8880,8888,8983,9000,9001,9043,9060,9080,9090,9091,9092,9200,9443,9502,9800,9981,10000,10250,11371,12443,15672,16080,17778,18091,18092,20720,32000,55440,55672"

# Common Ports
httpx -l subdomains.txt \
    -follow-redirects -random-agent \
    -status-code -title -web-server -tech-detect -location -content-length \
    -threads 50 -rate-limit 150 -timeout 10 -retries 2 \
    -silent -no-color -json -o websites_temp.json
cat websites_temp.json |
    jq -s 'try .' |
    jq 'try unique_by(.input)' |
    jq 'try .[]' >websites_common.json &#x26;&#x26;
    rm websites_temp.json

cat websites_common.json |
    jq -r 'try .url' |
    grep -E "$DOMAIN$|\.$DOMAIN$" |
    grep -E '^((http|https):\/\/)?([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{1,}(\/.*)?$' |
    sed "s/*.//" |
    anew -q websites_common.txt

cat websites_common.json |
    jq -r 'try . |"\(.url) [\(.status_code)] [\(.title)] [\(.webserver)] \(.tech)"' |
    grep "$DOMAIN" |
    anew -q websites_common_plain.txt

# Uncommon Ports
httpx -l subdomains.txt -p $WEB_UNCOMMON_PORTS \
    -follow-host-redirects -random-agent \
    -status-code -title -web-server -tech-detect -location \
    -threads 50 -timeout 10 -retries 2 \
    -silent -no-color -json -o <a data-footnote-ref href="#user-content-fn-1">websites_uncommon_temp</a>.json

cat websites_uncommon_temp.json |
    jq -r 'try .url' |
    grep "$DOMAIN" |
    grep -E '^((http|https):\/\/)?([a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{1,}(\/.*)?' |
    sed "s/*.//" |
    anew -q websites_uncommon.txt

cat websites_uncommon_temp.json |
    jq -r 'try . |"\(.url) [\(.status_code)] [\(.title)] [\(.webserver)] \(.tech)"' |
    grep "$DOMAIN" |
    anew -q websites_uncommon_plain.txt

if [[ -s ".tmp/web_full_info_uncommon.txt" ]]; then
    if [[ $domain =~ ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9] ]]; then
        cat .tmp/web_full_info_uncommon.txt 2>>"$LOGFILE" | anew -q webs/web_full_info_uncommon.txt
    else
        cat .tmp/web_full_info_uncommon.txt 2>>"$LOGFILE" | grep "$domain" | anew -q webs/web_full_info_uncommon.txt
    fi
fi
</code></pre>

## Website Screenshot

```bash
nuclei -l websites_common.txt -headless -id screenshot -V dir='screenshots'
```

[^1]: Delete the "\_temp" character
