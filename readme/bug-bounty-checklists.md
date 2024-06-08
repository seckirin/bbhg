# Bug Bounty Checklists

[Bug Bounty Hunter Guide](https://app.gitbook.com/o/EOc6S49gNX0wky8nj5si/s/dIwZJMkFd4Zza9vjuuJ7/) • [Contents](table-of-contents.md) • [**Checklists**](bug-bounty-checklists.md)

`Recon Phase`

## Business Information Gathering

* [ ] 检查[**安全响应中心 (SRC)**](../awesome-bugbounty/security-response-center.md) 最新通告，识别并记录目标组织[**主营业务**](../reconnaissance/business-info-gathering.md)
* [ ] 选择一个归属于目标组织的业务、产品或公司名称转到互联网资产发现

## Internet Asset Discovery

* 对归属于目标组织的**公司名称**实体，转到[**根域名枚举 (Root Domain Enum)**](../reconnaissance/root-domain-enum.md)
* 对归属于目标组织的**域名**实体，转到子域名枚举 (Subdomain Enum)
* 对归属于目标组织的**子域名**实体，转到网站探测 (Website Probing)
* 对归属于目标组织的**网站应用**实体，转到信息收集 (Information Gathering)

### Root Domain Enumeration

* [ ] 通过企业名称 / ICP 备案号收集根域
* [ ] 通过企业名称搜索证书透明度日志
* [ ] 检查根域的内部 DNS 服务器
* [ ] 检查根域关的 DNS 证书透明度日志
* [ ] 通过证书通用名称反查根域

## Information Gathering









### Based on Company

* [ ] [**收集企业基本信息**](../reconnaissance/business-info-gathering.md)：收集联系电话、电子邮箱及域名，以及员工姓名和法定代表人姓名。同时，不要忘了查看公司地址及公司主营业务。
* [ ] [**枚举网络资产**](../reconnaissance/internet-asset-discovery.md)：收集 ASN / CIDR、根域名、移动应用程序，以及微信小程序。需要注意的是，并不是所有公司都拥有属于自己的 ASN / CIDR。
* [ ] [**收集企业收购信息**](broken-reference)：收集控股子公司，以及在 SRC 通告中出现过的资产对应的公司。
* [ ] **筛选出归属于目标企业的公司**：从控股子公司中筛选出归属于目标企业的公司，递归执行基于公司的侦查流程。或者，你也可以在合适的时机选定一个企业主体，对归属于主体企业的所有的根域名执行基于域名的侦查流程。

### Based on Domain

* [ ] [**分析域名信息**](../reconnaissance/pending-pages/vulnerability-assessment-greater-than-web-greater-than-information-gathering/domain-analysis.md)：查询域名的注册信息、ICP 备案信息，以及 IP 解析及相应的历史记录，判断其是否归属于目标企业。
* [ ] [**枚举子域名**](../reconnaissance/subdomain-enumeration.md)：使用各种技术来枚举尽可能多的子域名。
* [ ] [**处理子域名**](../reconnaissance/pending-pages/domain-based.md)：对枚举的子域名进行处理，包括验证存活子域名、收集子域名 DNS 记录、归类 IP 地址、对 IP 进行端口扫描和网站探测，以及对探测到的网站进行指纹识别。

### Based on Website

* [ ] Fingerprint Web Server
* [ ] Review Metafiles
* [ ] Enumerate Applications on Webserver
* [ ] Review comments on source code
* [ ] Identify Application Entry Points
* [ ] Map Execution Paths Through Application
* [ ] Fingerprint Web Application Framework
* [ ] Map Application Architecture
