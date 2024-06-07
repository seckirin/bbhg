# Useful Commands

根据 IP 地址进行排序，而不是单个数字

```bash
sort -n -t . -k 1,1 -k 2,2 -k 3,3 -k 4,4 ips.txt
```

提取以 `:` 作为分隔符，第一个位置的值

```bash
# awk
awk -F ':' '{print $1}' ports.txt

# cut
cut -d ':' -f 1 ports.txt
```

使用 bash 数组保存命令参数并执行带有参数命令

```bash
params=(-l -t -r -h)
ls "${params[@]}"

httpx_params=(
    -follow-host-redirects
    -title -web-server -tech-detect -location -status-code
    -threads 50 -rl 150 -timeout 10 -retries 2
)

echo bytedance.com | httpx $httpx_params
```

提取根域名

```bash
# https://github.com/tomnomnom/unfurl
cat test.txt | unfurl -u apexes
```

使用 fofax 通过根域名列表的子域名查询 IP 地址

```bash
python3 ip_via_subdomain.py | fofax -ff ip -fs 10000 | anew ipaddresses.txt
```

使用 fofax 通过根域名列表的证书 CN 查询 IP 地址

```bash
python3 ip_via_cert.py | fofax -ff ip -fs 10000 | anew ipaddresses.txt
```
