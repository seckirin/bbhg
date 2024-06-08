# Business Info Gathering

## Identify Main Business

<details>

<summary>The <code>biz_recon</code> function of shell script.</summary>

```sh
biz_recon() {

  mode=$1
  search=$2

  wiki_urls=(
    # Encyclopedias
    "https://zh.wikipedia.org/w/index.php?search=%search"
    "https://en.wikipedia.org/w/index.php?search=%search"
    "https://baike.baidu.com/search?word=%search"
    "https://wiki.mbalib.com/wiki/Special:Search?search=%search"
  )
  
  company_urls=(
    "https://aiqicha.baidu.com/s?q=%search"
    "https://www.tianyancha.com/search?key=%search"
    "https://www.qcc.com/web/search?key=%search"
    "https://shuidi.cn/pc-search?key=%search"
    "https://www.crunchbase.com/" # Please access manually
  )
  
  app_urls=(
    "https://app.diandian.com/search/all-24-%search"
    "https://app.diandian.com/search/all-75-%search"
    "https://www.taptap.cn/search/%search"
    "https://sou.xiaolanben.com/search?key=%search"
    "https://www.qimai.cn/search/index/country/cn/version/ios14/search/%search"
    "https://www.qimai.cn/search/android/market/6/search/%search"
    "https://www.google.com/search?q=site:apps.apple.com+\"mac+app\"+%search"
  )
  
  website_urls=(
    "https://icp.chinaz.com/%search"
    "https://www.beianx.cn/search/%search"
    "https://icplishi.com/%search/"
  )
  
  misc_urls=(
    # Search engines
    "https://www.baidu.com/s?wd=%search"
    "https://www.so.com/s?q=%search"
    "https://so.toutiao.com/search?dvpf=pc&keyword=%search"
    "https://www.google.com/search?q=%search"
    "https://www.bing.com/search?q=%search"
    "https://duckduckgo.com/?q=%search"
    "https://yandex.com/search/?text=%search"
    "https://search.yahoo.com/search?p=%search"
    "https://www.mbalib.com/s?q=%search"
    "https://www.sogou.com/web?query=%search"
    "https://weixin.sogou.com/weixin?type=1&query=%search"
    "https://weixin.sogou.com/weixin?type=2&query=%search"
    "https://www.gitlogs.com/most_popular?topic=%search"
  )
  
    case $mode in
    "wiki")
      urls=("${wiki_urls[@]}")
      echo "查询百科信息："
      ;;
    "company")
      urls=("${company_urls[@]}")
      echo "查询公司信息："
      ;;
    "app")
      urls=("${app_urls[@]}")
      echo "查询应用信息："
      ;;
    "website")
      urls=("${website_urls[@]}")
      echo "查询网站信息："
      ;;
    "misc")
      urls=("${misc_urls[@]}")
      echo "查询其他信息："
      ;;
    "all")
      urls=("${wiki_urls[@]}" "${company_urls[@]}" "${app_urls[@]}" "${website_urls[@]}" "${misc_urls[@]}")
      echo "查询所有信息："
      ;;
    *)
      echo "无效的模式：$mode"
      return 1
      ;;
  esac

  for url in "${urls[@]}"; do
    echo "$(echo $url | sed "s/%search/$search/g")"
  done
}
```



</details>

```sh
# Ambiguous
biz_recon wiki <keyword>
biz_recon misc <keyword>

# Website / Domain
biz_recon website <domain_name>

# Application
biz_recon app <application_name>

# Company
biz_recon company <company_name>
```

## Products

```bash
./enscan-v1.0.0-darwin-arm64 -f <companies.txt> \
     -type {all, aqc, tyc} -field icp,copyright,app
```

## Investment or Subsidiary

```bash
# Websites
https://www.qcc.com/
https://aiqicha.baidu.com/
https://shuidi.cn/
https://www.tianyancha.com/

# https://github.com/wgpsec/ENScan_GO
./enscan-<version> -n <company_name> \
    [-invest {49, 50}] [-deep {3, 7}] \
    [-type {aqc,tyc,all,qimai}] \
    [-field {icp,weibo,wechat,app,job,wx_app,copyright,subpplier}] \
```
